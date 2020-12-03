---


---

<h2 id="安装">安装</h2>
<pre class=" language-python"><code class="prism  language-python">cd <span class="token operator">/</span>usr<span class="token operator">/</span>local<span class="token operator">/</span>src
wget http<span class="token punctuation">:</span><span class="token operator">//</span>lear<span class="token punctuation">.</span>inrialpes<span class="token punctuation">.</span>fr<span class="token operator">/</span>src<span class="token operator">/</span>deepmatching<span class="token operator">/</span>code<span class="token operator">/</span>deepmatching_1<span class="token number">.2</span><span class="token punctuation">.</span><span class="token number">2</span><span class="token punctuation">.</span><span class="token builtin">zip</span>

unzip deepmatching_1<span class="token number">.2</span><span class="token punctuation">.</span><span class="token number">2</span><span class="token punctuation">.</span><span class="token builtin">zip</span>
cd deepmatching_1<span class="token number">.2</span><span class="token punctuation">.</span>2_c<span class="token operator">+</span><span class="token operator">+</span>

yum install atlas<span class="token operator">-</span>devel
<span class="token comment">#查找numpy的arrayobject路径</span>
find <span class="token operator">/</span> <span class="token operator">-</span>name arrayobject<span class="token punctuation">.</span>h

ln <span class="token operator">-</span>s  <span class="token operator">/</span>usr<span class="token operator">/</span>lib64<span class="token operator">/</span>python2<span class="token number">.7</span><span class="token operator">/</span>site<span class="token operator">-</span>packages<span class="token operator">/</span>numpy<span class="token operator">/</span>core<span class="token operator">/</span>include<span class="token operator">/</span>numpy <span class="token operator">/</span>usr<span class="token operator">/</span>include<span class="token operator">/</span>numpy

make clean <span class="token builtin">all</span>
make python

</code></pre>
<h2 id="使用">使用</h2>
<pre><code>./deepmatching climb1.png liberty2.png -nt 0

./deepmatching climb1.png liberty2.png | python rescore.py climb1.png liberty2.png
</code></pre>
<h2 id="可视化">可视化</h2>
<p>由于centos没有桌面，所以先在centos生成文件，然后下载回来本地，再在本地可视化</p>
<pre><code>cd /usr/local/src/deepmatching_1.2.2_c++
./deepmatching 10-1.png 10-2.png -nt 0 &gt; out.txt
</code></pre>
<p><a href="http://xn--viz-5n0el78g.py">修改viz.py</a><br>
适用python3.7<br>
和本地可视化</p>
<pre><code>import sys  
from PIL import Image  
from numpy import *  
from matplotlib.pyplot import *  
  
  
def show_correspondences( img0, img1, corr ):  
  assert corr.shape[-1]==6  
  corr = corr[corr[:,4]&gt;0,:]  
  # make beautiful colors  
  center = corr[:,[1,0]].mean(axis=0) # array(img0.shape[:2])/2 #  
  corr[:,5] = arctan2(*(corr[:,[1,0]] - center).T)  
  corr[:,5] = int32(64*corr[:,5]/pi) % 128  
  set_max = set(corr[:,5])  
  colors = {m:i for i,m in enumerate(set_max)}  
  colors = {m:cm.hsv(i/float(len(colors))) for m,i in colors.items()}  
  def motion_notify_callback(event):  
  if event.inaxes==None: return  
  numaxis = event.inaxes.numaxis  
      if numaxis&lt;0: return  
  x,y = event.xdata, event.ydata  
      ax1.lines = []  
  ax2.lines = []  
  n = sum((corr[:,2*numaxis:2*(numaxis+1)] - [x,y])**2,1).argmin() # find nearest point  
  print ("\rdisplaying #%d (%d,%d) --&gt; (%d,%d), score=%g from maxima %d" % (n, corr[n,0],corr[n,1],corr[n,2],corr[n,3],corr[n,4],corr[n,5]))  
  sys.stdout.flush()  
  
  x,y = corr[n,0:2]  
  ax1.plot(x,y,'+',ms=10,mew=2,color='blue',scalex=False,scaley=False)  
  x,y = corr[n,2:4]  
  ax2.plot(x,y,'+',ms=10,mew=2,color='red',scalex=False,scaley=False)  
  # we redraw only the concerned axes  
  renderer = fig.canvas.get_renderer()  
  ax1.draw(renderer)    
 ax2.draw(renderer)  
  fig.canvas.blit(ax1.bbox)  
  fig.canvas.blit(ax2.bbox)  
  def noticks():  
  xticks([])  
  yticks([])  
  clf()  
  ax1 = subplot(221)  
  ax1.numaxis = 0  
  imshow(img0,interpolation='nearest')  
  noticks()  
  ax2 = subplot(222)  
  ax2.numaxis = 1  
  imshow(img1,interpolation='nearest')  
  noticks()  
  ax = subplot(223)  
  ax.numaxis = -1  
  imshow(img0/2+64,interpolation='nearest')  
  for m in set_max:  
  plot(corr[corr[:,5]==m,0],corr[corr[:,5]==m,1],'+',ms=10,mew=2,color=colors[m],scalex=0,scaley=0)  
  noticks()  
  ax = subplot(224)  
  ax.numaxis = -1  
  imshow(img1/2+64,interpolation='nearest')  
  for m in set_max:  
  plot(corr[corr[:,5]==m,2],corr[corr[:,5]==m,3],'+',ms=10,mew=2,color=colors[m],scalex=0,scaley=0)  
  noticks()  
  subplots_adjust(left=0.01, bottom=0.01, right=0.99, top=0.99,  
  wspace=0.02, hspace=0.02)  
  fig = get_current_fig_manager().canvas.figure  
    cid_move = fig.canvas.mpl_connect('motion_notify_event',motion_notify_callback)  
  print ("Move your mouse over the top images to visualize individual matches")  
  show()  
  fig.canvas.mpl_disconnect(cid_move)  
  
  
  
if __name__=='__main__':  
  # args = sys.argv[1:]  
  img0 = array(Image.open('D:/test_image/predict/10-1.png').convert('RGB'))  
  img1 = array(Image.open('D:/test_image/predict/10-2.png').convert('RGB'))  
  
  # ./deepmatching 10-1.png 10-2.png -nt 0 &gt; out.txt  
  
  retained_matches = []  
  with open('out.txt', 'r') as file:  
  result = file.read(-1)  
  
  for line in result.split("\n"):  
  line = line.split()  
  if not line or len(line)!=6 or not line[0][0].isdigit(): continue  
  x0, y0, x1, y1, score, index = line  
    retained_matches.append((float(x0),float(y0),float(x1),float(y1),float(score),float(index)))  
  assert retained_matches, 'error: no matches piped to this program'  
  show_correspondences(img0, img1, array(retained_matches))
</code></pre>

