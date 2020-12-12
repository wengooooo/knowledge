---


---

<h3 id="ssim">SSIM</h3>
<pre class=" language-python"><code class="prism  language-python"><span class="token keyword">from</span> skimage<span class="token punctuation">.</span>measure <span class="token keyword">import</span> compare_ssim
<span class="token keyword">import</span> cv2

<span class="token keyword">def</span> <span class="token function">compare_image_ssim</span><span class="token punctuation">(</span>path_image1<span class="token punctuation">,</span> path_image2<span class="token punctuation">)</span><span class="token punctuation">:</span>  
  
  imageA <span class="token operator">=</span> cv2<span class="token punctuation">.</span>imread<span class="token punctuation">(</span>path_image1<span class="token punctuation">)</span>  
  imageB <span class="token operator">=</span> cv2<span class="token punctuation">.</span>imread<span class="token punctuation">(</span>path_image2<span class="token punctuation">)</span>  
  
  imageB <span class="token operator">=</span> np<span class="token punctuation">.</span>resize<span class="token punctuation">(</span>imageB<span class="token punctuation">,</span> <span class="token punctuation">(</span>imageA<span class="token punctuation">.</span>shape<span class="token punctuation">[</span><span class="token number">0</span><span class="token punctuation">]</span><span class="token punctuation">,</span> imageA<span class="token punctuation">.</span>shape<span class="token punctuation">[</span><span class="token number">1</span><span class="token punctuation">]</span><span class="token punctuation">,</span> imageA<span class="token punctuation">.</span>shape<span class="token punctuation">[</span><span class="token number">2</span><span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">)</span>  
  
  ssim <span class="token operator">=</span> compare_ssim<span class="token punctuation">(</span>imageA<span class="token punctuation">,</span> imageB<span class="token punctuation">,</span> multichannel<span class="token operator">=</span><span class="token boolean">True</span><span class="token punctuation">)</span>  
  
  <span class="token keyword">return</span> ssim
  
compare_image_ssim<span class="token punctuation">(</span>r<span class="token string">'D:\test_image\predict\16-1.png'</span><span class="token punctuation">,</span> r<span class="token string">'D:\test_image\predict\16-2.png'</span><span class="token punctuation">)</span>
</code></pre>
<h3 id="相关性-卡方-巴氏距离">相关性, 卡方, 巴氏距离</h3>
<pre class=" language-python"><code class="prism  language-python"><span class="token keyword">def</span> <span class="token function">compare_img_hist</span><span class="token punctuation">(</span>img1<span class="token punctuation">,</span> img2<span class="token punctuation">)</span><span class="token punctuation">:</span>  
  OPENCV_METHODS <span class="token operator">=</span> <span class="token punctuation">(</span>  
  <span class="token comment"># 相关性比较  </span>
  <span class="token punctuation">(</span><span class="token string">"Correlation"</span><span class="token punctuation">,</span> cv<span class="token punctuation">.</span>HISTCMP_CORREL<span class="token punctuation">)</span><span class="token punctuation">,</span>  
  <span class="token comment"># 卡方比较  </span>
  <span class="token punctuation">(</span><span class="token string">"Chi-Squared"</span><span class="token punctuation">,</span> cv<span class="token punctuation">.</span>HISTCMP_CHISQR<span class="token punctuation">)</span><span class="token punctuation">,</span>  
  <span class="token comment"># 相交性比较  </span>
  <span class="token punctuation">(</span><span class="token string">"Intersection"</span><span class="token punctuation">,</span> cv<span class="token punctuation">.</span>HISTCMP_INTERSECT<span class="token punctuation">)</span><span class="token punctuation">,</span>  
  <span class="token comment"># 巴氏距离比较  </span>
  <span class="token punctuation">(</span><span class="token string">"Bhattacharyya"</span><span class="token punctuation">,</span> cv<span class="token punctuation">.</span>HISTCMP_BHATTACHARYYA<span class="token punctuation">)</span><span class="token punctuation">)</span>  
  
  <span class="token triple-quoted-string string">"""  
 Compare the similarity of two pictures using histogram(直方图)  
 Attention: this is a comparision of similarity, using histogram to calculate ​ For example: 1. img1 and img2 are both 720P .PNG file, and if compare with img1, img2 only add a black dot(about 9*9px), the result will be 0.999999999953 ​ :param img1: img1 in MAT format(img1 = cv2.imread(image1))  
 :param img2: img2 in MAT format(img2 = cv2.imread(image2))  
 :return: the similarity of two pictures """</span>  <span class="token comment"># Get the histogram data of image 1, then using normalize the picture for better compare  </span>
  img1_hist <span class="token operator">=</span> cv<span class="token punctuation">.</span>calcHist<span class="token punctuation">(</span><span class="token punctuation">[</span>img1<span class="token punctuation">]</span><span class="token punctuation">,</span> <span class="token punctuation">[</span><span class="token number">1</span><span class="token punctuation">]</span><span class="token punctuation">,</span> <span class="token boolean">None</span><span class="token punctuation">,</span> <span class="token punctuation">[</span><span class="token number">256</span><span class="token punctuation">]</span><span class="token punctuation">,</span> <span class="token punctuation">[</span><span class="token number">0</span><span class="token punctuation">,</span> <span class="token number">256</span><span class="token punctuation">]</span><span class="token punctuation">)</span>  
  img1_hist <span class="token operator">=</span> cv<span class="token punctuation">.</span>normalize<span class="token punctuation">(</span>img1_hist<span class="token punctuation">,</span> img1_hist<span class="token punctuation">,</span> <span class="token number">0</span><span class="token punctuation">,</span> <span class="token number">1</span><span class="token punctuation">,</span> cv<span class="token punctuation">.</span>NORM_MINMAX<span class="token punctuation">,</span> <span class="token operator">-</span><span class="token number">1</span><span class="token punctuation">)</span>  
  
  img2_hist <span class="token operator">=</span> cv<span class="token punctuation">.</span>calcHist<span class="token punctuation">(</span><span class="token punctuation">[</span>img2<span class="token punctuation">]</span><span class="token punctuation">,</span> <span class="token punctuation">[</span><span class="token number">1</span><span class="token punctuation">]</span><span class="token punctuation">,</span> <span class="token boolean">None</span><span class="token punctuation">,</span> <span class="token punctuation">[</span><span class="token number">256</span><span class="token punctuation">]</span><span class="token punctuation">,</span> <span class="token punctuation">[</span><span class="token number">0</span><span class="token punctuation">,</span> <span class="token number">256</span><span class="token punctuation">]</span><span class="token punctuation">)</span>  
  img2_hist <span class="token operator">=</span> cv<span class="token punctuation">.</span>normalize<span class="token punctuation">(</span>img2_hist<span class="token punctuation">,</span> img2_hist<span class="token punctuation">,</span> <span class="token number">0</span><span class="token punctuation">,</span> <span class="token number">1</span><span class="token punctuation">,</span> cv<span class="token punctuation">.</span>NORM_MINMAX<span class="token punctuation">,</span> <span class="token operator">-</span><span class="token number">1</span><span class="token punctuation">)</span>  
  
  similarity <span class="token operator">=</span> cv<span class="token punctuation">.</span>compareHist<span class="token punctuation">(</span>img1_hist<span class="token punctuation">,</span> img2_hist<span class="token punctuation">,</span> cv<span class="token punctuation">.</span>HISTCMP_CORREL<span class="token punctuation">)</span>  
  similarity2 <span class="token operator">=</span> cv<span class="token punctuation">.</span>compareHist<span class="token punctuation">(</span>img1_hist<span class="token punctuation">,</span> img2_hist<span class="token punctuation">,</span> cv<span class="token punctuation">.</span>HISTCMP_CHISQR<span class="token punctuation">)</span>  
  similarity3 <span class="token operator">=</span> cv<span class="token punctuation">.</span>compareHist<span class="token punctuation">(</span>img1_hist<span class="token punctuation">,</span> img2_hist<span class="token punctuation">,</span> cv<span class="token punctuation">.</span>HISTCMP_BHATTACHARYYA<span class="token punctuation">)</span>  
  <span class="token keyword">print</span><span class="token punctuation">(</span><span class="token string">"相关性：%s, 卡方：%s, 巴氏距离：%s"</span> <span class="token operator">%</span> <span class="token punctuation">(</span>similarity<span class="token punctuation">,</span> similarity2<span class="token punctuation">,</span> similarity3<span class="token punctuation">)</span><span class="token punctuation">)</span>  
  <span class="token keyword">return</span> similarity
<span class="token keyword">print</span><span class="token punctuation">(</span>compare_img_hist<span class="token punctuation">(</span>cv<span class="token punctuation">.</span>imread<span class="token punctuation">(</span>r<span class="token string">'D:\test_image\predict\1.jpg'</span><span class="token punctuation">)</span><span class="token punctuation">,</span> cv<span class="token punctuation">.</span>imread<span class="token punctuation">(</span>r<span class="token string">'D:\test_image\predict\2.jpg'</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">)</span>  
<span class="token keyword">print</span><span class="token punctuation">(</span>compare_img_hist<span class="token punctuation">(</span>cv<span class="token punctuation">.</span>imread<span class="token punctuation">(</span>r<span class="token string">'D:\test_image\predict\3.jpg'</span><span class="token punctuation">)</span><span class="token punctuation">,</span> cv<span class="token punctuation">.</span>imread<span class="token punctuation">(</span>r<span class="token string">'D:\test_image\predict\4.jpg'</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">)</span>  
<span class="token keyword">print</span><span class="token punctuation">(</span>compare_img_hist<span class="token punctuation">(</span>cv<span class="token punctuation">.</span>imread<span class="token punctuation">(</span>r<span class="token string">'D:\test_image\predict\5.jpg'</span><span class="token punctuation">)</span><span class="token punctuation">,</span> cv<span class="token punctuation">.</span>imread<span class="token punctuation">(</span>r<span class="token string">'D:\test_image\predict\6.jpg'</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">)</span>  
<span class="token keyword">print</span><span class="token punctuation">(</span>compare_img_hist<span class="token punctuation">(</span>cv<span class="token punctuation">.</span>imread<span class="token punctuation">(</span>r<span class="token string">'D:\test_image\predict\7.jpg'</span><span class="token punctuation">)</span><span class="token punctuation">,</span> cv<span class="token punctuation">.</span>imread<span class="token punctuation">(</span>r<span class="token string">'D:\test_image\predict\8.jpg'</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">)</span>
</code></pre>
<h3 id="直方图">直方图</h3>
<pre><code># 计算单通道的直方图的相似值
def calculate(image1, image2):
    hist1 = cv2.calcHist([image1], [0], None, [256], [0.0, 255.0])
    hist2 = cv2.calcHist([image2], [0], None, [256], [0.0, 255.0])
    # 计算直方图的重合度
    degree = 0
    for i in range(len(hist1)):
        if hist1[i] != hist2[i]:
            degree = degree + (1 - abs(hist1[i] - hist2[i]) / max(hist1[i], hist2[i]))
        else:
            degree = degree + 1
    degree = degree / len(hist1)
    return degree
</code></pre>
<h3 id="otsu算法">Otsu算法</h3>
<pre class=" language-python"><code class="prism  language-python"><span class="token keyword">import</span> cv2  
<span class="token keyword">from</span> matplotlib <span class="token keyword">import</span> pyplot <span class="token keyword">as</span> plt  
img <span class="token operator">=</span> cv2<span class="token punctuation">.</span>imread<span class="token punctuation">(</span><span class="token string">'1.jpg'</span><span class="token punctuation">,</span> <span class="token number">0</span><span class="token punctuation">)</span>  
ret2<span class="token punctuation">,</span> th2 <span class="token operator">=</span> cv2<span class="token punctuation">.</span>threshold<span class="token punctuation">(</span>img<span class="token punctuation">,</span> <span class="token number">0</span><span class="token punctuation">,</span> <span class="token number">255</span><span class="token punctuation">,</span> cv2<span class="token punctuation">.</span>THRESH_BINARY <span class="token operator">+</span> cv2<span class="token punctuation">.</span>THRESH_OTSU<span class="token punctuation">)</span>  

res <span class="token operator">=</span> regenerate_img<span class="token punctuation">(</span>img<span class="token punctuation">,</span> ret2<span class="token punctuation">)</span>  
plt<span class="token punctuation">.</span>imshow<span class="token punctuation">(</span>res<span class="token punctuation">)</span>  
plt<span class="token punctuation">.</span>show<span class="token punctuation">(</span><span class="token punctuation">)</span>
</code></pre>

