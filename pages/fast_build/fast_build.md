## FastBuild分布式编译

参考了以下教程：

  https://blueroses.top/2021/11/04/shi-yong-fastbuild-jia-kuai-unrealengine-bian-yi-su-du/

  https://dev.epicgames.com/documentation/zh-cn/unreal-engine/build-configuration-for-unreal-engine?application_version=5.4

  https://www.cnblogs.com/sbfhy/p/13046658.html

  https://blog.csdn.net/cjw_soledad/article/details/117362397

UE4.27是官方集成了FastBuild的，并且提供了FastBuild的可执行文件，在D:\UnrealEngine\Engine\Extras\ThirdPartyNotUE\FASTBuild\Win64目录里。

我们无需再去对FastBuild做任何修改。

本篇在Windows上来实战使用FastBuild分布式编译UE4。

下面是机器配置：

|主机   | 肉鸡  |
|---|---|
|GPD Pocket3   | 台式机  |
|N6000@1.1GHz 4Core 4Thread |E5-2680 v3@2.5Ghz x2 24Core 48Thread|
|8GB|32GB|


### 1. FastBuild介绍

![](../../imgs/fast_build/win_dir_list_file.jpg)

FastBuild就只有这么几个文件，只需要使用`FBuildWorker.exe`和`FBuild.exe`就可以开始分布式编译。

下面介绍其作用。

#### 1.1 FBuildWorker

FBuildWorker.exe在肉鸡上执行，运行如图。

![](../../imgs/fast_build/fbuildworker.jpg)

它启动后监听了31264端口，等待主机发来任务，启动后可以输入下面命令来看下监听了端口没。

```bash
C:\Users\Administrator>netstat -an |findstr 31264
  TCP    0.0.0.0:31264          0.0.0.0:0              LISTENING
```

FBuildWorker.exe在肉鸡执行后，会查找环境变量的 FASTBUILD_BROKERAGE_PATH 配置的共享文件夹，往里面写入当前的电脑名字，表示我是肉鸡，可以被其他主机使用。

![](../../imgs/fast_build/worker_computer_name_in_share_folder.jpg)

#### 1.2 FBuild

FBuild.exe是主机编辑任务使用的任务分发工具.

比如我在一台垃圾电脑上，配置了UnrealBuildTools使用FastBuild，那么在垃圾电脑上编译时，就会执行FBuild.exe。

主机开始编译后，就会去环境变量的 FASTBUILD_BROKERAGE_PATH 配置的共享文件夹，查找有哪些文件，就知道了有哪些肉鸡，就会去连接，发送源码到肉鸡去编译。


### 2. 实战分布式编译

从上面的介绍知道，需要创建一个共享文件夹，让工作机和肉鸡都可以访问到。

然后在工作机和肉鸡中都配置系统环境变量`FASTBUILD_BROKERAGE_PATH`指向这个共享文件夹。
