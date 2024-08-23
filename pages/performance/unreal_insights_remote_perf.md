## 10.2 基于UnrealInsights的远程性能监测系统

UnrealInsights是基于网络来收集传输Trace数据的，并且是C(游戏客户端)-->S(TraceInsightsServer)-->C(Profiler)机制。

基于这套架构进行开发，可以搭建一套远程的性能监测系统。

在测试人员对游戏进行测试时，开发可以实时查看游戏在不同手机上的性能表现数据。

PS：要是每个TraceEvent上，有对应的录屏功能就好了，有点像UWA的Unity GOT工具。

