UE4.27是官方支持了FastBuild的

https://blueroses.top/2021/11/04/shi-yong-fastbuild-jia-kuai-unrealengine-bian-yi-su-du/

https://dev.epicgames.com/documentation/zh-cn/unreal-engine/build-configuration-for-unreal-engine?application_version=5.4

https://www.cnblogs.com/sbfhy/p/13046658.html

https://blog.csdn.net/cjw_soledad/article/details/117362397 原理以及修改

自带了FastBuild，在D:\UnrealEngine\Engine\Extras\ThirdPartyNotUE\FASTBuild\Win64目录里

FBuild.exe是主机编辑任务使用的任务分发工具，比如我在一台垃圾电脑上，配置了UnrealBuildTools使用FastBuild，那么在垃圾电脑上编译时，就会执行FBuild.exe。

FBuildWorker.exe在肉鸡上执行，启动后监听了31264端口，等待主机发来任务，启动后可以输入下面命令来看下监听了端口没。

```bash
C:\Users\Administrator>netstat -an |findstr 31264
  TCP    0.0.0.0:31264          0.0.0.0:0              LISTENING
```

FBuildWorker.exe在肉鸡执行后，会查找环境变量的 FASTBUILD_BROKERAGE_PATH 配置的共享文件夹，往里面写入当前的电脑名字，表示我是肉鸡，可以被其他主机使用。

主机开始编译后，就会去环境变量的 FASTBUILD_BROKERAGE_PATH 配置的共享文件夹，查找有哪些文件，就知道了有哪些肉鸡，就会去连接，发送编译任务。



