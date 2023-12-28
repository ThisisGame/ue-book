## 2.2 UE4.27集成opencv4.8.0

下载opencv 4.8.0

https://opencv.org/releases/

双击opencv-4.8.0-windows.exe，解压到项目里的Source\ThirdParty。

![](../../imgs/opencv/integrate_opencv/extract_thirdparty.jpg)


### 一些异常

1. 当使用Debug版本的lib时，`cv::VideoWriter.open` 失败，并且抛出异常。

```c#
PublicAdditionalLibraries.AddRange(
    new string[] {
        // ... add other libraries needed for this module
        Path.Combine(ThirdPartyPath,"opencv/build/x64/vc16/lib/opencv_world480d.lib")
    }
);
```

![[](../../imgs/opencv/integrate_opencv/video_writer_open_failed.jpg)]

```
D:\UnrealEngine\Engine\Binaries\Win64\UE4Editor-Win64-Debug.exe D:\ExploreUE\ThirdPersonDemo\ThirdPersonDemo.uproject -skipcompile
[OVRPlugin][ERROR] ovr_Initialize failed: Unable to load LibOVRRT DLL
 (software\oculussdk\integrations\ovrplugin\main\src\util\compositorcapi.cpp:78)[ WARN:0@157.380] global cap_msmf.cpp:2892 cv::cv_writer_open_with_params MSMF: Unknown C++ exception is raised
[ INFO:0@157.300] global videoio_registry.cpp:244 cv::`anonymous-namespace'::VideoBackendRegistry::VideoBackendRegistry VIDEOIO: Enabled backends(9, sorted by priority): FFMPEG(1000); GSTREAMER(990); INTEL_MFX(980); MSMF(970); DSHOW(960); CV_IMAGES(950); CV_MJPEG(940); UEYE(930); OBSENSOR(920)
[ INFO:0@157.300] global backend_plugin.cpp:383 cv::impl::getPluginCandidates Found 3 plugin(s) for FFMPEG
[ INFO:0@157.300] global plugin_loader.impl.hpp:67 cv::plugin::impl::DynamicLib::libraryLoad load D:\UnrealEngine\Engine\Binaries\Win64\opencv_videoio_ffmpeg480_64d.dll => FAILED
[ INFO:0@157.303] global plugin_loader.impl.hpp:67 cv::plugin::impl::DynamicLib::libraryLoad load opencv_videoio_ffmpeg480_64d.dll => FAILED
[ INFO:0@157.347] global plugin_loader.impl.hpp:67 cv::plugin::impl::DynamicLib::libraryLoad load opencv_videoio_ffmpeg480_64.dll => OK
[ INFO:0@157.347] global backend_plugin.cpp:50 cv::impl::PluginBackend::initCaptureAPI Found entry: 'opencv_videoio_capture_plugin_init_v1'
[ INFO:0@157.347] global backend_plugin.cpp:169 cv::impl::PluginBackend::checkCompatibility Video I/O: initialized 'FFmpeg OpenCV Video I/O Capture plugin': built with OpenCV 4.8 (ABI/API = 1/1), current OpenCV version is '4.8.0' (ABI/API = 1/1)
[ INFO:0@157.347] global backend_plugin.cpp:69 cv::impl::PluginBackend::initCaptureAPI Video I/O: plugin is ready to use 'FFmpeg OpenCV Video I/O Capture plugin'
[ INFO:0@157.347] global backend_plugin.cpp:84 cv::impl::PluginBackend::initWriterAPI Found entry: 'opencv_videoio_writer_plugin_init_v1'
[ INFO:0@157.347] global backend_plugin.cpp:169 cv::impl::PluginBackend::checkCompatibility Video I/O: initialized 'FFmpeg OpenCV Video I/O Writer plugin': built with OpenCV 4.8 (ABI/API = 1/1), current OpenCV version is '4.8.0' (ABI/API = 1/1)
[ INFO:0@157.347] global backend_plugin.cpp:103 cv::impl::PluginBackend::initWriterAPI Video I/O: plugin is ready to use 'FFmpeg OpenCV Video I/O Writer plugin'
[ INFO:0@157.348] global backend_plugin.cpp:383 cv::impl::getPluginCandidates Found 2 plugin(s) for GSTREAMER
[ INFO:0@157.348] global plugin_loader.impl.hpp:67 cv::plugin::impl::DynamicLib::libraryLoad load D:\UnrealEngine\Engine\Binaries\Win64\opencv_videoio_gstreamer480_64d.dll => FAILED
[ INFO:0@157.350] global plugin_loader.impl.hpp:67 cv::plugin::impl::DynamicLib::libraryLoad load opencv_videoio_gstreamer480_64d.dll => FAILED
[ INFO:0@157.350] global backend_plugin.cpp:383 cv::impl::getPluginCandidates Found 2 plugin(s) for INTEL_MFX
[ INFO:0@157.351] global plugin_loader.impl.hpp:67 cv::plugin::impl::DynamicLib::libraryLoad load D:\UnrealEngine\Engine\Binaries\Win64\opencv_videoio_intel_mfx480_64d.dll => FAILED
[ INFO:0@157.352] global plugin_loader.impl.hpp:67 cv::plugin::impl::DynamicLib::libraryLoad load opencv_videoio_intel_mfx480_64d.dll => FAILED
[ INFO:0@157.353] global backend_plugin.cpp:383 cv::impl::getPluginCandidates Found 2 plugin(s) for MSMF
[ INFO:0@157.379] global plugin_loader.impl.hpp:67 cv::plugin::impl::DynamicLib::libraryLoad load D:\UnrealEngine\Engine\Binaries\Win64\opencv_videoio_msmf480_64d.dll => OK
[ INFO:0@157.379] global backend_plugin.cpp:50 cv::impl::PluginBackend::initCaptureAPI Found entry: 'opencv_videoio_capture_plugin_init_v1'
[ INFO:0@157.379] global backend_plugin.cpp:169 cv::impl::PluginBackend::checkCompatibility Video I/O: initialized 'Microsoft Media Foundation OpenCV Video I/O plugin': built with OpenCV 4.8 (ABI/API = 1/1), current OpenCV version is '4.8.0' (ABI/API = 1/1)
[ INFO:0@157.379] global backend_plugin.cpp:69 cv::impl::PluginBackend::initCaptureAPI Video I/O: plugin is ready to use 'Microsoft Media Foundation OpenCV Video I/O plugin'
[ INFO:0@157.379] global backend_plugin.cpp:84 cv::impl::PluginBackend::initWriterAPI Found entry: 'opencv_videoio_writer_plugin_init_v1'
[ INFO:0@157.379] global backend_plugin.cpp:169 cv::impl::PluginBackend::checkCompatibility Video I/O: initialized 'Microsoft Media Foundation OpenCV Video I/O plugin': built with OpenCV 4.8 (ABI/API = 1/1), current OpenCV version is '4.8.0' (ABI/API = 1/1)
[ INFO:0@157.379] global backend_plugin.cpp:103 cv::impl::PluginBackend::initWriterAPI Video I/O: plugin is ready to use 'Microsoft Media Foundation OpenCV Video I/O plugin'
[ERROR:0@157.380] global cap.cpp:656 cv::VideoWriter::open VIDEOIO(CV_IMAGES): raised unknown C++ exception!


[ERROR:0@157.381] global cap.cpp:656 cv::VideoWriter::open VIDEOIO(CV_MJPEG): raised unknown C++ exception!

```

修改为非Debug版本后正常。

```c#
PublicAdditionalLibraries.AddRange(
    new string[] {
        // ... add other libraries needed for this module
        Path.Combine(ThirdPartyPath,"opencv/build/x64/vc16/lib/opencv_world480.lib")
    }
);
```



