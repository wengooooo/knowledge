---


---

<p>安装python 3.7.2</p>
<pre><code>yum -y groupinstall "Development tools" yum -y  install zlib-devel  bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel yum install -y libffi-devel zlib1g-dev yum install zlib*  -y

cd /usr/local/src
wget wget https://www.python.org/ftp/python/3.7.2/Python-3.7.2.tar.xz
tar -xvJf  Python-3.7.2.tar.xz
cd Python-3.7.2
./configure --prefix=/usr/local/python3 --enable-optimizations --with-ssl 
#第一个指定安装的路径,不指定的话,安装过程中可能软件所需要的文件复制到其他不同目录,删除软件很不方便,复制软件也不方便.
#第二个可以提高python10%-20%代码运行速度.
#第三个是为了安装pip需要用到ssl,后面报错会有提到.
make &amp;&amp; make install

ln -s /usr/local/python3/bin/python3 /usr/local/bin/python3
ln -s /usr/local/python3/bin/pip3 /usr/local/bin/pip3

yum -y install zlib1g-dev
yum -y install libffi-devel 
</code></pre>
<p>安装glibc-2.23</p>
<pre><code>wget http://ftp.gnu.org/gnu/glibc/glibc-2.23.tar.gz
tar xf glibc-2.23.tar.gz 
cd glibc-2.23/
mkdir build 
cd build/
../configure --prefix=/usr --disable-profile --disable-werror --enable-add-ons --with-headers=/usr/include --with-binutils=/usr/bin
make 
make install
#报错之后运行，不然会出现找不到libm-2.23.so
unlink /lib64/libm.so.6
ln -s libm-2.23.so libm.so.6
再make一次
make install

ldd --version

</code></pre>
<p>安装过程报语法错误就修改源代码</p>
<pre><code>“”“
make的错误两个文件缺少一对 {}
如何快速找到需要添加的位置, 进入vim后  直接输入 / 符号 后面接你要搜索的内容,类似浏览器的Ctrl+F查找
”“”
/ *loc != NULL  快速匹配 *loc != NULL 

# 错误1
vim /home/glibc-2.23/nis/nis_call.c
   if (*loc != NULL)
+  {  #这里添加一个{号
     for (i = 1; i &lt; 16; ++i)
       if (nis_server_cache[i] == NULL)
 	{
@@ -690,6 +691,7 @@ nis_server_cache_add (const_nis_name nam
 	       || ((*loc)-&gt;uses == nis_server_cache[i]-&gt;uses
 		   &amp;&amp; (*loc)-&gt;expires &gt; nis_server_cache[i]-&gt;expires))
 	loc = &amp;nis_server_cache[i];
+  }  #这里添加一个} 号
   old = *loc;

# 错误2 
vim  /home/glibc-2.23/stdlib/setenv.c   
   
   ep = __environ;
   if (ep != NULL)
+  { #这里添加一个{号
     while (*ep != NULL)
       if (!strncmp (*ep, name, len) &amp;&amp; (*ep)[len] == '=')
 	{
@@ -290,6 +291,7 @@ unsetenv (const char *name)
 	}
       else
 	++ep;
+  }  #这里添加一个}号
 
   UNLOCK;   
</code></pre>
<p>安装tensorflow</p>
<pre><code>yum install mesa-libGL.x86_64
pip3 install opencv-python
pip3 install keras==2.3.1
pip3 install h5py==2.10.0
pip3 install tensorflow==1.14.0
</code></pre>

