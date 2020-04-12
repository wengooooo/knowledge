---


---

<h2 id="系统要求">系统要求</h2>
<ul>
<li>至少8GB RAM的64位系统。 强烈建议超过16GB。</li>
<li>至少100G的空闲NTFS格式的硬盘，SSD更佳</li>
<li>适当的VS版本</li>
<li>至少win7系统以上, 2019Server也可以</li>
</ul>
<h2 id="测试机器">测试机器</h2>
<ul>
<li>阿里云通用性</li>
<li>4核CPU</li>
<li>16G内存</li>
<li>100G SSD云盘</li>
<li>windows server 2019 数据中心英文版本</li>
</ul>
<h2 id="设置windows">设置Windows</h2>
<h3 id="visual-studio">Visual Studio</h3>
<p>Chromium需要使用Visual Studio 2017（&gt; = 15.7.2）或2019（&gt; = 16.0.0）进行构建。 必须安装NativeDesktop和C++，MFC选项的安装。 可以从命令行通过将以下参数传递给Visual Studio安装程序来完成。</p>
<p><a href="https://pan.baidu.com/s/1GNFwPqnXjdLlIhPL5Ui4XA">vs_enterprise_2019.EXE</a>  提取码  zunh<br>
<a href="https://pan.baidu.com/s/1I1wipbz-Nazqm7YRGCogyQ">vs_enterprise_2017.EXE</a> 提取码  d3j3</p>
<pre><code>vs_enterprise_2019.EXE --add Microsoft.VisualStudio.Workload.NativeDesktop  --add Microsoft.VisualStudio.Component.VC.ATLMFC --includeRecommended
</code></pre>
<p>您必须安装版本<strong>10.0.18362</strong>或更高版本的Windows 10 SDK。 可以单独安装它，也可以在Visual Studio安装程序中选中相应的框。</p>
<p>安装成功之后还需要安装：<strong>SDK Debugging Tools</strong>，英文版操作系统按照这个方法修改，打开：Control Panel → Programs → Programs and Features → 选择 “Windows Software Development Kit” → Change → Change → Check “Debugging Tools For Windows” → Change</p>
<h2 id="配置depot_tools">配置<code>depot_tools</code></h2>
<p>下载 <a href="https://storage.googleapis.com/chrome-infra/depot_tools.zip">depot_tools bundle</a> 然后解压到c盘</p>
<p>添加环境变量：</p>
<p>把depot_tools的路径加到 PATH  环境变量的最前面，如果你电脑有安装python的环境，建议先卸载</p>
<p>再添加如下环境变量：<br>
GYP_MSVS_VERSION=2019<br>
DEPOT_TOOLS_WIN_TOOLCHAIN=0<br>
GYP_MSVS_OVERRIDE_PATH=“C:\Program Files (x86)\Microsoft Visual Studio\2019\Enterprise”</p>
<h2 id="安装python和git">安装PYTHON和GIT</h2>
<p>进入depot_tools目录进行安装</p>
<pre><code>c:/&gt; cd depot_tools
c:/depot_tools/&gt; gclient
</code></pre>
<h2 id="获取chromium代码">获取Chromium代码</h2>
<p>配置GIT</p>
<pre><code>C:/depot_tools&gt; git config --global user.name "My Name" 
C:/depot_tools&gt; git config --global user.email "my-name@chromium.org" 
C:/depot_tools&gt; git config --global core.autocrlf false 
C:/depot_tools&gt; git config --global core.filemode false 
C:/depot_tools&gt; git config --global branch.autosetuprebase always
</code></pre>
<p>创建chromium</p>
<pre><code>C:/&gt; mkdir chromium &amp;&amp; cd chromium
</code></pre>
<p>获取代码</p>
<pre><code>C:/chromium&gt; fetch chromium
</code></pre>
<p>不想要git历史记录的话</p>
<pre><code>C:/chromium&gt; fetch --no-history chromium
</code></pre>
<p>获取所有的代码分支信息?</p>
<pre><code>C:/chromium&gt; cd src  
C:/chromium/src&gt; git fetch --all  
C:/chromium/src&gt; git fetch --tags  # 如果使用no-history，就不要用这句
C:/chromium/src&gt; git pull
</code></pre>
<p>切换编译Chromium版本</p>
<pre><code>C:/chromium/src/&gt;git checkout tags/80.0.3987.163 -b 80.0.3987.163
C:/chromium/src/&gt;git branch
C:/chromium/src/&gt;cd ..
C:/chromium/&gt;gclient sync -D --force --reset
</code></pre>
<h2 id="编译">编译</h2>
<p>生成编译输出</p>
<p>生成的vs解决方案将包含数千个项目，并且加载速度非常慢。 使用–filters参数可限制仅对您感兴趣的代码生成项目文件。尽管这也将限制项目浏览器中显示的文件，但是调试仍然可以进行，您可以在手动打开的文件中设置断点。 一个可以让您在IDE中编译和运行Chrome但不显示任何源文件的最小解决方案是：</p>
<pre><code>C:/chromium/src/&gt;gn gen out/Debug --ide=vs2019 --filters=//chrome  
</code></pre>
<p>或者</p>
<pre><code>C:/chromium/src/&gt;gn gen --ide=vs --filters=//chrome --no-deps out\Default
</code></pre>
<p>如果想生成更多的项目文件</p>
<pre><code>C:/chromium/src/&gt;gn gen out/Debug --ide=vs2019 --filters=//chrome;//third_party/WebKit/*;//gpu/*
</code></pre>
<p>设置编译参数,会弹出一个文本文件，输入编译参数</p>
<pre><code>C:/chromium/src/&gt;gn args out/Debug 
</code></pre>
<p>参数</p>
<pre><code>target_os="win"
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
</code></pre>
<p>安装pywin32</p>
<pre><code>C:/depot_tools/&gt;python -m pip install pywin32
</code></pre>
<p>开始编译</p>
<pre><code>C:/chromium/src/&gt;ninja -C out\Debug chrome
</code></pre>
<h2 id="参考链接">参考链接</h2>
<p><a href="https://chromium.googlesource.com/chromium/src/+/80.0.3987.163/docs/windows_build_instructions.md">官方安装文档</a></p>
<p><a href="https://chromium.googlesource.com/chromium/src.git/+/master/docs/building_old_revisions.md">如何安装旧版本</a></p>
<p><a href="https://my.oschina.net/u/3175552/blog/3001316">中文版说明编译68版本</a></p>

