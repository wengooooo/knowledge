# bian
## 系统要求

-   至少8GB RAM的64位系统。 强烈建议超过16GB。
-  至少100G的空闲NTFS格式的硬盘，SSD更佳  
-   适当的VS版本
-   至少win7系统以上, 2019Server也可以

## 测试机器
- 阿里云通用性
- 4核CPU
- 16G内存
- 100G SSD云盘 
- windows server 2019 数据中心英文版本

## 设置Windows

### Visual Studio
Chromium需要使用Visual Studio 2017（> = 15.7.2）或2019（> = 16.0.0）进行构建。 必须安装NativeDesktop和C++，MFC选项的安装。 可以从命令行通过将以下参数传递给Visual Studio安装程序来完成。

[vs_enterprise_2019.EXE](https://pan.baidu.com/s/1GNFwPqnXjdLlIhPL5Ui4XA)  提取码  zunh
[vs_enterprise_2017.EXE](https://pan.baidu.com/s/1I1wipbz-Nazqm7YRGCogyQ) 提取码  d3j3
```
vs_enterprise_2019.EXE --add Microsoft.VisualStudio.Workload.NativeDesktop  --add Microsoft.VisualStudio.Component.VC.ATLMFC --includeRecommended
```

您必须安装版本**10.0.18362**或更高版本的Windows 10 SDK。 可以单独安装它，也可以在Visual Studio安装程序中选中相应的框。


安装成功之后还需要安装：**SDK Debugging Tools**，英文版操作系统按照这个方法修改，打开：Control Panel → Programs → Programs and Features → 选择 “Windows Software Development Kit” → Change → Change → Check “Debugging Tools For Windows” → Change


## 配置`depot_tools`
下载 [depot_tools bundle](https://storage.googleapis.com/chrome-infra/depot_tools.zip) 然后解压到c盘

添加环境变量：

把depot_tools的路径加到 PATH  环境变量的最前面，如果你电脑有安装python的环境，建议先卸载

再添加如下环境变量：
GYP_MSVS_VERSION=2019
DEPOT_TOOLS_WIN_TOOLCHAIN=0  
GYP_MSVS_OVERRIDE_PATH="C:\Program Files (x86)\Microsoft Visual Studio\2019\Enterprise"

## 安装PYTHON和GIT

进入depot_tools目录进行安装

    c:/> cd depot_tools
    c:/depot_tools/> gclient
    
## 获取Chromium代码

配置GIT

    C:/depot_tools> git config --global user.name "My Name" 
    C:/depot_tools> git config --global user.email "my-name@chromium.org" 
    C:/depot_tools> git config --global core.autocrlf false 
    C:/depot_tools> git config --global core.filemode false 
    C:/depot_tools> git config --global branch.autosetuprebase always

创建chromium

    C:/> mkdir chromium && cd chromium

获取代码

    C:/chromium> fetch chromium
    
不想要git历史记录的话

    C:/chromium> fetch --no-history chromium

获取所有的代码分支信息?

    C:/chromium> cd src  
    C:/chromium/src> git fetch --all  
    C:/chromium/src> git fetch --tags  # 如果使用no-history，就不要用这句
    C:/chromium/src> git pull

切换编译Chromium版本

    C:/chromium/src/>git checkout tags/80.0.3987.163 -b 80.0.3987.163
    C:/chromium/src/>git branch
    C:/chromium/src/>cd ..
    C:/chromium/>gclient sync -D --force --reset

## 编译

生成编译输出

生成的vs解决方案将包含数千个项目，并且加载速度非常慢。 使用--filters参数可限制仅对您感兴趣的代码生成项目文件。尽管这也将限制项目浏览器中显示的文件，但是调试仍然可以进行，您可以在手动打开的文件中设置断点。 一个可以让您在IDE中编译和运行Chrome但不显示任何源文件的最小解决方案是：

    C:/chromium/src/>gn gen out/Debug --ide=vs2019 --filters=//chrome  
    
  或者

    C:/chromium/src/>gn gen --ide=vs --filters=//chrome --no-deps out\Default


如果想生成更多的项目文件

    C:/chromium/src/>gn gen out/Debug --ide=vs2019 --filters=//chrome;//third_party/WebKit/*;//gpu/*

设置编译参数,会弹出一个文本文件，输入编译参数

    C:/chromium/src/>gn args out/Debug 

   
参数

    target_os="win"
    target_cpu="x86"
    is_component_build=true
    is_debug=true
    is_official_build=false
    google_api_key=false
    google_default_client_id=false
    google_default_client_secret=false
    proprietary_codecs=true
    media_use_ffmpeg=true
    ffmpeg_branding="Chrome"
    enable_nacl=false
    enable_mse_mpeg2ts_stream_parser=true
    enable_hls_sample_aes=true
    blink_symbol_level=0

安装pywin32

```
C:/depot_tools/>python -m pip install pywin32
```

开始编译

```
C:/chromium/src/>ninja -C out\Debug chrome
```

## 参考链接

[官方安装文档](https://chromium.googlesource.com/chromium/src/+/80.0.3987.163/docs/windows_build_instructions.md)

[如何安装旧版本](https://chromium.googlesource.com/chromium/src.git/+/master/docs/building_old_revisions.md)

[中文版说明编译68版本](https://my.oschina.net/u/3175552/blog/3001316)
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEwOTIyMDI4NDJdfQ==
-->