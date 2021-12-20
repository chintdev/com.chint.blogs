---
title: css3 特性之大屏制作
date: 2021-12-20 09:31:57
author: xms # 文章作者
img: # 文章特征图，推荐使用图床(腾讯云、七牛云、又拍云等)来做图片的路径.如: http://xxx.com/xxx.jpg
top: false # 推荐文章（文章是否置顶）
hide: false # 隐藏文章，如果hide值为true，则文章不会在首页显示
cover: false # 文章是否需要加入到首页轮播封面中
coverImg: # 文章在首页轮播封面需要显示的图片路径，如果没有，则默认使用文章的特色图片
password:  # 文章阅读密码
toc: false # 是否开启 TOC，可以针对某篇文章单独关闭 TOC 的功能 `暂未开放`
mathjax: false # 是否开启数学公式支持 
summary: # 文章摘要
categories: # 文章分类
tags: css3 # 文章标签
keywords: css3 # 文章关键字，SEO 时需要
reprintPolicy: # 文章转载规则 `暂时不支持`
---

# 可视化大屏制作 如何适配不同的分辨率和比例？
=================  
* 大屏的需求:  
  1. 需要满足市场上不同比例的大屏(5:4、4:3（较少）、16:10、16:9（新品）)
  2. 需要满足不同分辨率的屏幕 (1080px 2K 4K)
   <!-- ![例子](3.png) -->

*  问题:如果按照设计稿1920*1080开发 ,就不适用于2k 4K,展示效果不好
或者是需要用% 或者 vh vw等比例去写css, 这样的方法不仅笨拙 而且代码量增加了不少,而且计算复杂,存在兼容性问题,字体大小不对等问题
<!-- ![例子](lz.png) -->

**于是我就去看看大厂是怎么做大屏的,结果就发现他们都用了一个比较巧妙的方法,只是用了css3的一个新特性:transform:scale(n);用这个缩放的**

参考百度云的 SugerBi可视化
1. <https://sugar.aipage.com/dashboard/5e81f0ec04f74164a1d0bae94cd386dc>
<!-- ![RUNOOB 图标](1.png)
![代码3](1_1.png) -->

2. <https://sugar.aipage.com/dashboard/9f5dd5b3166c2c832e75cd3db65df460?conditions=%7B%22select%22%3A%225%22%2C%22multiSelect%22%3A%5B%221%22%2C%222%22%2C%223%22%2C%224%22%2C%225%22%2C%22%22%5D%7D>
<!-- ![大屏2](2.png)
![代码3](2_2.png) -->

以及阿里的datav可视化

3. https://datav.aliyuncs.com/share/464d4b7dfb15e9af2f8dfaa8447af9dd?spm=5176.15036128.0.0.6b6f1f40Or2k99
   
<!-- ![大屏3](4.png)
![代码3](4_1.png) -->


<!-- *(再看我们预付费的大屏,用了这样的缩放小技巧)* -->

具体解决代码-----

    function resize() {
        if (document.documentElement && document.documentElement.clientHeight && document.documentElement.clientWidth) {
            var winHeight = document.documentElement.clientHeight; //浏览器的高度
            var winWidth = document.documentElement.clientWidth; //浏览器的宽度
            var main = document.getElementById("main")
            main.style.transform = "scale(" + winWidth / 1920 + ", " + winHeight / 1080 + ")";
        }
    }
    window.onresize = function () { //当窗口大小发生变化时，重新自适应
        resize();
    }

实践一下!
最后再了解一下
*  css3其他新特性
1. 新增选择器 p:nth-child（n）{color: rgba（255, 0, 0, 0.75）}
2. 弹性盒模型 display: flex;
3. 多列布局 column-count: 5;
4. 媒体查询 @media （max-width: 480px） {.box: {column-count: 1;}}
5. 个性化字体 @font-face{font-family:BorderWeb;src:url（BORDERW0.eot）；}
6. 颜色透明度 color: rgba（255, 0, 0, 0.75）；
7. 圆角 border-radius: 5px;
8. 渐变 background:linear-gradient（red, green, blue）；
9. 阴影 box-shadow:3px 3px 3px rgba（0, 64, 128, 0.3）；
10. 倒影 box-reflect: below 2px;
11. 文字装饰 text-stroke-color: red;
12. 文字溢出 text-overflow:ellipsis;
13. 背景效果 background-size: 100px 100px;
14. 边框效果 border-image:url（bt_blue.png） 0 10;
**转换**
1. 旋转 transform: rotate（20deg）；
2. 倾斜 transform: skew（150deg, -10deg）；
3. 位移 transform:translate（20px, 20px）；
4. 缩放 transform: scale（.5）；
5. 平滑过渡 transition: all .3s ease-in .1s;
6. 动画 @keyframes anim-1 {50% {border-radius: 50%;}} animation: anim-1 1s;



