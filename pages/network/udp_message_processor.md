## 处理收到的UDP包

对UDP包的接受处理和发送处理，是在单独的线程中进行消费处理。

```c++
uint32 FUdpMessageProcessor::Run()
{
	while (!bStopping)
	{
		WorkEvent->Wait(CalculateWaitTime());

		do
		{
			FDateTime LastTime = CurrentTime;
			CurrentTime = FDateTime::UtcNow();
			DeltaTime = CurrentTime - LastTime;

			ConsumeInboundSegments();//处理收到的UDP包
			ConsumeOutboundMessages();//处理发出的UDP包
			UpdateKnownNodes();
			UpdateNetworkStatistics();
		} while ((!InboundSegments.IsEmpty() || MoreToSend()) && !bStopping);
	}

	delete Beacon;
	Beacon = nullptr;

	delete SocketSender;
	SocketSender = nullptr;

	return 0;
}
```

```c++
void FUdpMessageProcessor::ConsumeInboundSegments()//消费收到的UDP包
{
	SCOPED_MESSAGING_TRACE(FUdpMessageProcessor_ConsumeInboundSegments);
	FInboundSegment Segment;

	FTimespan MaxWorkTimespan = ComputeMaxWorkTimespan(DeadHelloIntervals, Beacon);
	while (InboundSegments.Dequeue(Segment)) //从队列中弹出一个收到的UDP包
	{
		// quick hack for TTP# 247103
		if (!Segment.Data.IsValid())
		{
			continue;
		}

		FUdpMessageSegment::FHeader Header;
		*Segment.Data << Header; //这里先从Segment.Data提取Header，拿到消息版本和消息类型等

		if (FilterSegment(Header))
		{
			FNodeInfo& NodeInfo = KnownNodes.FindOrAdd(Header.SenderNodeId);

			if (!NodeInfo.NodeId.IsValid())
			{
				NodeInfo.NodeId = Header.SenderNodeId;
				NodeInfo.ProtocolVersion = Header.ProtocolVersion;
				NodeDiscoveredDelegate.ExecuteIfBound(NodeInfo.NodeId);
			}

			NodeInfo.ProtocolVersion = Header.ProtocolVersion;
			NodeInfo.Endpoint = Segment.Sender;
			NodeInfo.LastSegmentReceivedTime = CurrentTime;

			switch (Header.SegmentType)
			{
			case EUdpMessageSegments::Abort:
				ProcessAbortSegment(Segment, NodeInfo);
				break;

			case EUdpMessageSegments::Acknowledge://收到一个ACK包，远端发来这个包，表示确认收到了某个包，里面包含了包ID。
				ProcessAcknowledgeSegment(Segment, NodeInfo);
				break;

			case EUdpMessageSegments::AcknowledgeSegments:
				ProcessAcknowledgeSegmentsSegment(Segment, NodeInfo);
				break;

			case EUdpMessageSegments::Bye:
				ProcessByeSegment(Segment, NodeInfo);
				break;

			case EUdpMessageSegments::Data:
				ProcessDataSegment(Segment, NodeInfo);
				break;

			case EUdpMessageSegments::Hello:
				ProcessHelloSegment(Segment, NodeInfo);
				break;

			case EUdpMessageSegments::Ping:
				ProcessPingSegment(Segment, NodeInfo);
				break;

			case EUdpMessageSegments::Pong:
				ProcessPongSegment(Segment, NodeInfo);
				break;

			case EUdpMessageSegments::Retransmit:
				ProcessRetransmitSegment(Segment, NodeInfo);
				break;

			case EUdpMessageSegments::Timeout:
				ProcessTimeoutSegment(Segment, NodeInfo);
				break;

			default:
				ProcessUnknownSegment(Segment, NodeInfo, (uint8)Header.SegmentType);
			}
		}

		if ((CurrentTime + MaxWorkTimespan) <= FDateTime::UtcNow())
		{
			break;
		}
	}
}
```

看下对Acknowledge包的处理。

```c++
void FUdpMessageProcessor::ProcessAcknowledgeSegment(FInboundSegment& Segment, FNodeInfo& NodeInfo)//收到一个ACK包，远端发来这个包，表示确认收到了某个包，里面包含了包ID。
{
	FUdpMessageSegment::FAcknowledgeChunk AcknowledgeChunk;
	AcknowledgeChunk.Serialize(*Segment.Data, NodeInfo.ProtocolVersion);

	TSharedPtr<FUdpMessageSegmenter>* FoundSegmenter = NodeInfo.Segmenters.Find(AcknowledgeChunk.MessageId);//从发出的UDP包中找到这个ACK包原始包ID对应的Segmenter，AcknowledgeChunk.MessageId是指需要ACK的包ID
	if (FoundSegmenter)
	{
		TSharedPtr<FUdpMessageSegmenter>& Segmenter = *FoundSegmenter;
		if (EnumHasAnyFlags(Segmenter->GetMessageFlags(), EMessageFlags::Reliable))//如果是可靠消息，就标记为已经成功发送
		{
			NodeInfo.MarkComplete(AcknowledgeChunk.MessageId, CurrentTime);
		}
		SendSegmenterStatsToListeners(AcknowledgeChunk.MessageId, NodeInfo.NodeId, Segmenter);//发送一个segmenter的统计信息给监听者
	}

	NodeInfo.Segmenters.Remove(AcknowledgeChunk.MessageId);//从发出的包列表，移除这个已经被ACK的包。

	UE_LOG(LogUdpMessaging, Verbose, TEXT("Received Acknowledge for %d from %s"), AcknowledgeChunk.MessageId , *NodeInfo.NodeId.ToString());//打印日志，表示从远端收到了ACK包。
}
```

`FNodeInfo` 存储一个远端的信息，包括收到的UDP包列表。

`NodeInfo.Segmenters`存储了对这个远端发出的包，现在收到了一个ACK包，ACK包里带了需要ACK的PackageID，找到这个PackageID对应的Segmenter。

如果这个Segmenter是标记为`Reliable`，那么就需要标记它为完成。

同时也需要处理这一段时间间隔内是否有`Reliable`包超时了，如果超时就设定为丢包，标记为需要重传。

```c++
/**
    * Make a given MessageId as complete and recompute the desired window size based
    * on any loss / acks received.
    * 标记一个消息为完成，并处理这一段时间的丢包情况，并根据丢失的数据包和收到的ack重新计算窗口大小
    */
void MarkComplete(int32 MessageId, const FDateTime& InCurrentTime)
{
    uint32 AckSegments = RemoveAllInflightFromMessageId(MessageId);//移除所有与给定MessageId匹配的数据包，从InflightSegments(正在飞行途中的数据包)中移除
    uint32 SegmentLoss = RemoveLostSegments(InCurrentTime);//遍历所有的发出的数据包，如果一个数据包超过2倍的平均往返时间还没有收到ACK。那么认为这个数据包就丢失了，我们必须重发它。返回丢失的数据包数量。
    ComputeWindowSize(AckSegments, SegmentLoss);//计算新的窗口大小
}
```

```c++
/**
    * Remove all segments in the inflight buffer that match the given MessageId.
    * 移除所有与给定MessageId匹配的数据包，从InflightSegments(正在飞行途中的数据包)中移除
    */
uint32 RemoveAllInflightFromMessageId(int32 MessageId)
{
    int32 BeforeRemove = InflightSegments.Num();//移除前的数量
    RemoveInflightIf([MessageId](const FSentData& Data)//移除所有MessageId相同的数据包，会有多个数据包的MessageId相同吗？
    {
        return Data.MessageId == MessageId;
    });
    return BeforeRemove - InflightSegments.Num();//返回移除的数量
}
```

```c++
/**
    * Iterate over all segments and discover any potential segments that may be lost.  A lost segment is determined
    * using 2 times the average round trip time.  If an ack has not been received in that time frame then it is
    * lost and we must resend it.
    * 遍历所有的发出的数据包，如果一个数据包超过2倍的平均往返时间还没有收到ACK。那么认为这个数据包就丢失了，我们必须重发它。
    * 返回丢失的数据包数量
    */
uint32 RemoveLostSegments(const FDateTime& InCurrentTime)
{
    uint32 SegmentsLost = 0;

    // We use 2 times the average round trip time to determine if a segment has been lost.
    // 如果一个标记为Reliable的数据包，在2倍的平均往返时间(发出到收到ACK的时间)内没有收到ack，那么认为这个数据包丢失了
    RemoveInflightIf([&SegmentsLost, InCurrentTime, this](const FSentData& Data)
    {
        const bool bIsLost = ((InCurrentTime - Data.TimeSent) > 2*AvgRoundTripTime) && Data.bIsReliable;
        if (bIsLost)
        {
            MarkSegmenterSegmentLoss(Data);//标记一个数据包为丢失，需要重传
            SegmentsLost++;
        }
        return bIsLost;
    });
    Statistics.SegmentsLost += SegmentsLost;//统计丢失的数据包数量
    return SegmentsLost;//返回丢失的数据包数量
}
```

```c++
/**
    * Mark a segment for retransmission because it was deemed lost.
    * 标记一个数据包为丢失，需要重传
    */
void MarkSegmenterSegmentLoss(const FSentData& Data)
{
    TSharedPtr<FUdpMessageSegmenter> Segmenter = Segmenters.FindRef(Data.MessageId);//在发出的包的队列中，找到这个包
    if (Segmenter.IsValid())
    {
        Segmenter->MarkForRetransmission(Data.SegmentNumber);//标记这个数据包为需要重传
    }
}
```

```c++
void FUdpMessageSegmenter::MarkForRetransmission(uint32 SegmentId)//标记这个数据包为需要重传
{
	if (SegmentId < (uint32)PendingSendSegments.Num() && !PendingSendSegments[SegmentId])//SegmentId这个数据包的序号做下标，PendingSendSegments[SegmentId] = true表示这个数据包还没有发送,!PendingSendSegments[SegmentId]表示这个数据包已经发送过了
	{
		UE_LOG(LogUdpMessaging, Verbose, TEXT("Marking segment %d of %d for retransmission"), SegmentId+1, PendingSendSegments.Num());

		++PendingSendSegmentsCount;//待发送的数据包数量+1
		PendingSendSegments[SegmentId] = true;//重新设置为true，表示这个数据包需要发送。
		// Note - we don't need to clear acknowledgments. If any segment is in transit and acknowledged after this
		// call we'll do that and if possible stop the pending send, and if not we don't need to wait for the ack
		// 注意-我们不需要清除acknowledgments。如果这个因为超时没收到ACK的包，这里被标记为重传，然后又马上收到了ACK，那么后面可能的话就会停止这个包的重传，如果没有收到ACK，我们也不需要等待ACK。
	}
}
```