```c++
/**
 * @brief 更新Trace Writer的连接状态
 * @return 是否成功更新连接
 */
static bool Writer_UpdateConnection()
{
    // 如果主线程中没有成功连接到AsioRecorder，函数直接返回false
    if (!GPendingDataHandle)
    {
        return false;
    }

	// 下面的代码只在游戏启动时执行一次。
	// 因为将主线程连接AsioRecorder的句柄GPendingDataHandle，交给Writer线程的GDataHandle后。
	// 主线程中的GPendingDataHandle会被置为0，所以这段代码只会执行一次。

	// 断线了怎么办呢？
	// 发现Unreal Insights并没有处理断线重连……
	// 在 void Writer_SendDataRaw(const void* Data, uint32 Size) 函数里，如果发送数据失败了，就会直接Close掉GDataHandle，并设置GDataHandle=0.
	// 那么是可以在Writer线程的Worker里判断GDataHandle是否为0，如果为0，那么就重新连接到AsioRecorder。

    // 如果已经存在一个数据句柄，那么它会关闭并丢弃待处理的数据句柄，然后返回false
    if (GDataHandle)
    {
        IoClose(GPendingDataHandle);
        GPendingDataHandle = 0;
        return false;
    }

    // 将主线程连接AsioRecorder的句柄GPendingDataHandle，交给Writer线程的GDataHandle
    GDataHandle = GPendingDataHandle;
    GPendingDataHandle = 0;

    // 进行握手操作，向数据句柄写入一个魔数'Magic'
    const uint32 Magic = 'TRCE';
    bool bOk = IoWrite(GDataHandle, &Magic, sizeof(Magic));

    // 向数据句柄写入一个传输头，包含传输版本和协议版本
    const struct {
        uint8 TransportVersion = ETransport::TidPacket;
        uint8 ProtocolVersion  = EProtocol::Id;
    } TransportHeader;
    bOk &= IoWrite(GDataHandle, &TransportHeader, sizeof(TransportHeader));

    // 如果在握手或写入传输头的过程中发生错误，它会关闭数据句柄并返回false
    if (!bOk)
    {
        IoClose(GDataHandle);
        GDataHandle = 0;
        return false;
    }

    // 发送头事件和事件描述
    TWriteBufferRedirect<512> HeaderEvents;
    Writer_LogHeader();
    Writer_LogTimingHeader();
    HeaderEvents.Close();

    Writer_DescribeEvents();

    Writer_SendData(HeaderEvents.GetData(), HeaderEvents.GetSize());

    // 返回true表示更新连接成功
    return true;
}

```

重连时是不是忘了写入`Trace`和协议导致解析失败？

```c++
// 进行握手操作，向数据句柄写入一个魔数'Magic'
const uint32 Magic = 'TRCE';
bool bOk = IoWrite(GDataHandle, &Magic, sizeof(Magic));

// 向数据句柄写入一个传输头，包含传输版本和协议版本
const struct {
    uint8 TransportVersion = ETransport::TidPacket;
    uint8 ProtocolVersion  = EProtocol::Id;
} TransportHeader;
bOk &= IoWrite(GDataHandle, &TransportHeader, sizeof(TransportHeader));
```