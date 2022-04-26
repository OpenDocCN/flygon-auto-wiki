<!--yml
category: 视频
date: 2022-04-26 11:46:01
-->

# 手把手教你使用 MoviePy 自制仿照百度AI图文一键转视频，全网文章可用 - 知乎

> 来源：[https://zhuanlan.zhihu.com/p/351087452](https://zhuanlan.zhihu.com/p/351087452)

﻿全部MoviePy文章目录

[](https://link.zhihu.com/?target=https%3A//blog.csdn.net/qq_20288327/article/details/113758352)

「Mr数据杨的图文一键转视频 V1.0」

## 图文生成视频专栏

[](https://www.zhihu.com/people/Escaflowne1985/zvideos)[](https://link.zhihu.com/?target=https%3A//baijiahao.baidu.com/u%3Fapp_id%3D1663365284421245)[](https://link.zhihu.com/?target=https%3A//www.toutiao.com/c/user/token/MS4wLjABAAAA1qi-anZFAvKqqh7wboo7ZBSStNT1K3dGYUJjQ8TLS_A/)

文中流程介绍中的作品成品

[](https://www.zhihu.com/zvideo/1290238134337892352)

## 百度AI图文生成功能

先了解以下百度AI的图文一键生成视频产品 [图文生成视频](https://link.zhihu.com/?target=https%3A//ai.baidu.com/creation/main/createlab) 点击跳转过去就可以了。

<figure data-size="normal">

<noscript><img src="img/7a796f58e0d2b7c67f65202b2b9acc82.png" data-caption="" data-size="normal" data-rawwidth="1920" data-rawheight="914" class="origin_image zh-lightbox-thumb" data-original="https://pic3.zhimg.com/v2-061ae1959643c1617392ee70b974bd56_r.jpg" data-original-src="https://pic3.zhimg.com/v2-061ae1959643c1617392ee70b974bd56_b.jpg"/></noscript>

</figure>

<figure data-size="normal">

<noscript><img src="img/2de7d3e83782cd0cb0b00fa84d0cd6f5.png" data-caption="" data-size="normal" data-rawwidth="1920" data-rawheight="914" class="origin_image zh-lightbox-thumb" data-original="https://pic1.zhimg.com/v2-49bb4c58aeb0236420c7f0341ce53480_r.jpg" data-original-src="https://pic1.zhimg.com/v2-49bb4c58aeb0236420c7f0341ce53480_b.jpg"/></noscript>

</figure>

这个API接口放出来大致2年了吧，还是这个熊样没变，不过好在里面的内容变化了许多，至少字体比以前看着舒服多了并且广告成分太重了。

然后重点来了。

这个功能需要申请，很是麻烦，我申请了1个月才开通成功，然后抱着好奇的心理试了试开始效果还不错，接下来坑的地方就开始了。

这个功能有点限制不好就是必须是百家号的文章，其他比如说头条、企鹅号、搜狐号、网易号、大风号、大鱼号这里面的文章压根就不让你用。想要做这些文章的图文视频智能转载到你的百家号然后用AI伪创作改写发表，一定概率上会影响你这些账号的权重（听运营的朋友说的，具体真假我也不知道）。

然后作为Python爱好者的我不甘心啊，反复研究看了几百遍百度这个图文一键生成图文视频是如何做到的，大概研究了1个多礼拜看明白了，也研究出来了1.0版本的代码。

## 任务拆解&开发流程

这种视频三要素，素材、讲解、字幕3部分。

**1.素材部分** 1\. 片头、片尾动画不多说，自己用AE或者PR去在各种素材网站上下一个然后自己改下就可以了，如果懒得弄也无所谓。 2\. 图片素材，通过爬虫的方式将文章抓下来提取图片然后保存本地，图片较少的在有搜索引擎的图片地方设置好搜索内容，搜索后下载。

**2.讲解部分** 1\. 文稿是文章中通过NLP提取的内容摘要，大致300字左右就可以了。 2\. 解说通过百度AI的文字转语音接口用主播的声音播放。

**3.字幕部分** 1\. 通过代码开发添加倒视频，后面在制作教程中有教。

**开发流程目录** ​​​​

<figure data-size="normal">

<noscript><img src="img/79805964d936ba3dde00281e862e65a6.png" data-caption="" data-size="normal" data-rawwidth="903" data-rawheight="504" class="origin_image zh-lightbox-thumb" data-original="https://pic2.zhimg.com/v2-b758b2f2efbdc9df0c1331ae8e06ce6d_r.jpg" data-original-src="https://pic2.zhimg.com/v2-b758b2f2efbdc9df0c1331ae8e06ce6d_b.jpg"/></noscript>

</figure>

基于是python的技术，由于是通用简单版本，所以会比较单调，仅仅是实现了功能。比如特效、转场这些会在未来更新。

## 开发环境

演示流程使用Anaconda的Juypter Notebook。

可以自行将内容移植成Py文件。

## 使用流程

**1.数据日志**

<figure data-size="normal">

<noscript><img src="img/03f3dbe4f8f4d0a8123c87f37af5e7cf.png" data-caption="" data-size="normal" data-rawwidth="1364" data-rawheight="276" class="origin_image zh-lightbox-thumb" data-original="https://pic4.zhimg.com/v2-54d159032524dc1711659407bea9eb27_r.jpg" data-original-src="https://pic4.zhimg.com/v2-54d159032524dc1711659407bea9eb27_b.jpg"/></noscript>

</figure>

记录抓取过的内容数据，把自己抓取过的内容进行一个自动化记录，方便未来避免处理重复的内容。

**2.数据抓取**

这里使用的是Python的requests方法抓取到你设置目标的文章的正文内容以及图片信息，为了让生成的视频不单调，建议多找些图片多的文章，或者在其中自己补充一些相关的图片，由于百度AI图文自动生成功能，他自身有强大的图片、视频数据库所以比相对局限的自制的内容就差一些，所以将就一下。

目前制作了的平台数据采集脚本不多，未来会更新。

**3.文字、图片数据处理** 为了视频审核通过，这里的Title和正文内容自己要修正一下。

<figure data-size="normal">

<noscript><img src="img/37ff88b928567ea692efa51f93d27ca0.png" data-caption="" data-size="normal" data-rawwidth="886" data-rawheight="458" class="origin_image zh-lightbox-thumb" data-original="https://pic4.zhimg.com/v2-7747ee3485ac3da1612ddbd5d695b1e7_r.jpg" data-original-src="https://pic4.zhimg.com/v2-7747ee3485ac3da1612ddbd5d695b1e7_b.jpg"/></noscript>

</figure>

为了视频审核通过，这里的图片也要看好了避免一些不必要的麻烦，这里还可以进行一个去水印处理，使用opencv方法就可以了。

<figure data-size="normal">

<noscript><img src="img/bcebdee2da518ec4da4b06a72433359c.png" data-caption="" data-size="normal" data-rawwidth="888" data-rawheight="343" class="origin_image zh-lightbox-thumb" data-original="https://pic2.zhimg.com/v2-eb68ea6656859c91b62fcf72fd458a5d_r.jpg" data-original-src="https://pic2.zhimg.com/v2-eb68ea6656859c91b62fcf72fd458a5d_b.jpg"/></noscript>

</figure>

**4.音频和字幕处理** 语音解说和字幕播放是如何做到同步的？比如你文章里有100个字先要用API接口的方式将这字幕转换成MP3文件作为解说音频，然后将字幕进行切分，这种切分方式我设置的一次显示20个作为字幕，然后进行循环切换。接下来记录MP3的时长。

<figure data-size="normal">

<noscript><img src="img/f429ba9f4ef107e96f20a1f8442164c6.png" data-caption="" data-size="normal" data-rawwidth="1198" data-rawheight="293" class="origin_image zh-lightbox-thumb" data-original="https://pic3.zhimg.com/v2-5d67bc4c0c78829b574a5f40bac74d1e_r.jpg" data-original-src="https://pic3.zhimg.com/v2-5d67bc4c0c78829b574a5f40bac74d1e_b.jpg"/></noscript>

</figure>

**5.图片裁剪** 由于互联网文章里的图片尺寸不一，因此要根据一定规则进行裁剪，这种裁剪方式根据你的视频素材模板进行设置，比如横版视频，竖版图片高度比不能超过设置的横板尺寸。反之竖版图片也是同样的方式进行处理。这样就不会发生图片出现在视频中很别扭的情况。

<figure data-size="normal">

<noscript><img src="img/2498d9dea4b5945852a2925796af0fd4.png" data-caption="" data-size="normal" data-rawwidth="894" data-rawheight="406" class="origin_image zh-lightbox-thumb" data-original="https://pic1.zhimg.com/v2-6ab93bb9ae62b90f78218d6189348cf8_r.jpg" data-original-src="https://pic1.zhimg.com/v2-6ab93bb9ae62b90f78218d6189348cf8_b.jpg"/></noscript>

</figure>

裁剪后的图片，貌似没啥变化是吧？其实图片的像素尺寸已经变了。

<figure data-size="normal">

<noscript><img src="img/6635565c741316baa9097c7de460dfcb.png" data-caption="" data-size="normal" data-rawwidth="890" data-rawheight="381" class="origin_image zh-lightbox-thumb" data-original="https://pic1.zhimg.com/v2-1745da440b812472b36123aaeba40314_r.jpg" data-original-src="https://pic1.zhimg.com/v2-1745da440b812472b36123aaeba40314_b.jpg"/></noscript>

</figure>

**6.制作封面** 选择一个抠图封面人物图像PNG做视频的封面，某些视频网站不能自定义视频封面的情况下是以第一帧作为视频封面的，所以我们要做一个，省去很多自定义的麻烦。

<figure data-size="normal">

<noscript><img src="img/d89e533574db09ee4248f05edc3e5b21.png" data-caption="" data-size="normal" data-rawwidth="1211" data-rawheight="293" class="origin_image zh-lightbox-thumb" data-original="https://pic2.zhimg.com/v2-cb60d63712f99153b32b399e10283ef5_r.jpg" data-original-src="https://pic2.zhimg.com/v2-cb60d63712f99153b32b399e10283ef5_b.jpg"/></noscript>

</figure>

然后将生成的png图片放到的封面图片中，这里我不会PS 就用PPT代替了。

<figure data-size="normal">

<noscript><img src="img/2b3d875cc7c1cadc8932de4c26093229.png" data-caption="" data-size="normal" data-rawwidth="899" data-rawheight="488" class="origin_image zh-lightbox-thumb" data-original="https://pic4.zhimg.com/v2-591554cadb2f64edadb054e096718d8f_r.jpg" data-original-src="https://pic4.zhimg.com/v2-591554cadb2f64edadb054e096718d8f_b.jpg"/></noscript>

</figure>

看下整体的基础素材

<figure data-size="normal">

<noscript><img src="img/1f3cc4fc3535cd07bde8985481405edc.png" data-caption="" data-size="normal" data-rawwidth="896" data-rawheight="512" class="origin_image zh-lightbox-thumb" data-original="https://pic3.zhimg.com/v2-a32dbc3f2ad3f164eeee0670f0f56cf2_r.jpg" data-original-src="https://pic3.zhimg.com/v2-a32dbc3f2ad3f164eeee0670f0f56cf2_b.jpg"/></noscript>

</figure>

**7.视频合成**

图片整理按顺序合成

<figure data-size="normal">

<noscript><img src="img/8073a2c27eb8aa06cb7a28ec5881a4a7.png" data-caption="" data-size="normal" data-rawwidth="1175" data-rawheight="341" class="origin_image zh-lightbox-thumb" data-original="https://pic4.zhimg.com/v2-2f616dbde8692101addc21ccdaa9976b_r.jpg" data-original-src="https://pic4.zhimg.com/v2-2f616dbde8692101addc21ccdaa9976b_b.jpg"/></noscript>

</figure>

<figure data-size="normal">

<noscript><img src="img/5271a4ad6c15af6bc69451a073ec1341.png" data-caption="" data-size="normal" data-rawwidth="889" data-rawheight="609" class="origin_image zh-lightbox-thumb" data-original="https://pic4.zhimg.com/v2-6317c1662e01eae7fe7c7a719be9b43b_r.jpg" data-original-src="https://pic4.zhimg.com/v2-6317c1662e01eae7fe7c7a719be9b43b_b.jpg"/></noscript>

</figure>

将字幕按照顺序与上面生成的视频进行合成，在根据字幕的播放顺序进行匹对即可。

<figure data-size="normal">

<noscript><img src="img/8293c27783e547923c27b3d0d6ac52df.png" data-caption="" data-size="normal" data-rawwidth="1194" data-rawheight="301" class="origin_image zh-lightbox-thumb" data-original="https://pic2.zhimg.com/v2-e6de3b20494c07cebcb4c2540b1cfd75_r.jpg" data-original-src="https://pic2.zhimg.com/v2-e6de3b20494c07cebcb4c2540b1cfd75_b.jpg"/></noscript>

</figure>

<figure data-size="normal">

<noscript><img src="img/f5e6e48a7d0babc9c3bc6f890092284a.png" data-caption="" data-size="normal" data-rawwidth="896" data-rawheight="605" class="origin_image zh-lightbox-thumb" data-original="https://pic2.zhimg.com/v2-752cf1f9d114654e17da1c9abed410c5_r.jpg" data-original-src="https://pic2.zhimg.com/v2-752cf1f9d114654e17da1c9abed410c5_b.jpg"/></noscript>

</figure>

将制作好的封面生成1秒不到的视频进行后续合成用顺便把title字幕也加进去。

<figure data-size="normal">

<noscript><img src="img/9878374a0e073a6f766af78127b8ea9a.png" data-caption="" data-size="normal" data-rawwidth="935" data-rawheight="479" class="origin_image zh-lightbox-thumb" data-original="https://pic4.zhimg.com/v2-8c285da55427aa87f5d727dece64d0af_r.jpg" data-original-src="https://pic4.zhimg.com/v2-8c285da55427aa87f5d727dece64d0af_b.jpg"/></noscript>

</figure>

<figure data-size="normal">

<noscript><img src="img/85c936ef7d05b21042b94d6124a4387c.png" data-caption="" data-size="normal" data-rawwidth="890" data-rawheight="608" class="origin_image zh-lightbox-thumb" data-original="https://pic4.zhimg.com/v2-8bcc6b7e5ecd4a6acf7e17bb2535d42f_r.jpg" data-original-src="https://pic4.zhimg.com/v2-8bcc6b7e5ecd4a6acf7e17bb2535d42f_b.jpg"/></noscript>

</figure>

拼接整个素材，这里准备好预先自己做好的开篇介绍和结尾推广的视频，如果没有也无所谓。

<figure data-size="normal">

<noscript><img src="img/291f76e7b7f640619c689e088570120b.png" data-caption="" data-size="normal" data-rawwidth="1204" data-rawheight="487" class="origin_image zh-lightbox-thumb" data-original="https://pic2.zhimg.com/v2-b15465a615231887dbf1365e7e0720bd_r.jpg" data-original-src="https://pic2.zhimg.com/v2-b15465a615231887dbf1365e7e0720bd_b.jpg"/></noscript>

</figure>

<figure data-size="normal">

<noscript><img src="img/91b2fa8b478f6c85dee098d82d49557d.png" data-caption="" data-size="normal" data-rawwidth="896" data-rawheight="605" class="origin_image zh-lightbox-thumb" data-original="https://pic4.zhimg.com/v2-8f2a9d56274ddad0fcbf52eb311f345f_r.jpg" data-original-src="https://pic4.zhimg.com/v2-8f2a9d56274ddad0fcbf52eb311f345f_b.jpg"/></noscript>

</figure>

生成视频的文件内容。

<figure data-size="normal">

<noscript><img src="img/bb8b646dce3c28750f3bb55c6b55c15b.png" data-caption="" data-size="normal" data-rawwidth="892" data-rawheight="351" class="origin_image zh-lightbox-thumb" data-original="https://pic1.zhimg.com/v2-0a3e108466006c7451adc1a71cea0690_r.jpg" data-original-src="https://pic1.zhimg.com/v2-0a3e108466006c7451adc1a71cea0690_b.jpg"/></noscript>

</figure>

logo水印合成。

<figure data-size="normal">

<noscript><img src="img/8edb19d1aa6cb84ee69c4a35194414b5.png" data-caption="" data-size="normal" data-rawwidth="1205" data-rawheight="462" class="origin_image zh-lightbox-thumb" data-original="https://pic3.zhimg.com/v2-1be7b9f0235c47e17f4c130fec4ba97e_r.jpg" data-original-src="https://pic3.zhimg.com/v2-1be7b9f0235c47e17f4c130fec4ba97e_b.jpg"/></noscript>

</figure>

OK 最后大功告成，完成了一键生成视频的操作。

可以关注一波然后点我的主页去查看各种技术流生成的视频啦。

## Git源码

欢迎大家点星星，支持就是我前进的动力。

[Github源码：ArticleGenerationVideo-V1.0](https://link.zhihu.com/?target=https%3A//github.com/Escaflowne1985/ArticleGenerationVideo-V1.0)