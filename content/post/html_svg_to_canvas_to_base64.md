---
title: "html svg转换成canvas，生成图片base64地址"
slug: "html_svg_to_canvas_to_base64"
description: "这里用到了 canvg.js 这个goole发布的插件，原理是把 svg 装成canvas，再利用canvas的toDataURL，输出图片的base64地址。"
date: "2019-03-30T01:25:36+08:00"
thumbnail: ""
categories:
  - "编程开发"
tags:
  - "svg"
  - "canvas"
  - "base64"
  - "canvg.js"
---

今天做小程序时发现小程序不支持svg标签，网页上的图标是 html5 的 svg 标签，没有字体源文件，通过 search 看到有人给出了解决方案，这里整理一下，方便自己或者有需要的朋友开箱即用。

这里用到了 canvg.js 这个goole发布的插件，原理是把 svg 转成canvas，再利用canvas的toDataURL，输出图片的base64地址。

canva.js 需要依赖于rgbcolor.js。这个应该都比较容易下载到。

附上下载地址：https://github.com/canvg/canvg

下面是demo：

```html
<!DOCTYPE html>
<head>
    <meta charset="UTF-8">
    <title>html svg转换成canvas，生成图片base64地址</title>
    <!--[if IE]>
    <script type="text/javascript" src="flashcanvas.js"></script>
    <![endif]-->
    <script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/canvg/1.4/rgbcolor.min.js"></script>
    <script type="text/javascript" src="https://cdn.jsdelivr.net/npm/canvg/dist/browser/canvg.min.js"></script>
    <script type="text/javascript">
        var context;
        var redraw = false;
    
        function resize() {
            var c = document.getElementById('container'),
                custom = true;

            if (!custom) {
                c.style.width = (window.innerWidth || document.body.clientWidth) + 'px';
                c.style.height = (window.innerHeight || document.body.clientHeight) + 'px';
            } else {
                c.style.width = '32px';
                c.style.height = '32px';
            }

            redraw = true;
        }
    
        function bodyonload() {
            var dataURL,
                canvas = document.getElementById('canvas');
    
            // canvas.style.background = '#de335e';
            if (typeof (FlashCanvas) != 'undefined') context = document.getElementById('canvas').getContext;
            var svg = '<svg xmlns="http://www.w3.org/2000/svg" height="512px" viewBox="0 0 464 464" width="512px"><g><path d="m24 144c-4.414062 0-8 3.585938-8 8v120h16v-120c0-4.414062-3.585938-8-8-8zm0 0" fill="#f60" class="active-path"></path><path d="m104 256h-32c-10.414062 0-19.214844 6.710938-22.527344 16h77.046875c-3.304687-9.289062-12.105469-16-22.519531-16zm0 0" fill="#f60" class="active-path"></path><path d="m56 417.472656v-97.472656h-16v97.472656c-9.304688 3.304688-16 12.09375-16 22.527344 0 13.257812 10.742188 24 24 24s24-10.742188 24-24c0-10.433594-6.695312-19.222656-16-22.527344zm0 0" fill="#f60" class="active-path"></path><path d="m0 288h464v16h-464zm0 0" fill="#f60" class="active-path"></path><path d="m48 248.207031c.089844-.070312.160156-.144531.246094-.207031-.085938-.0625-.15625-.136719-.246094-.207031zm0 0" fill="#f60" class="active-path"></path><path d="m101.121094 192h-29.121094c-13.230469 0-24 10.769531-24 24s10.769531 24 24 24h29.121094c13.230468 0 24-10.769531 24-24s-10.769532-24-24-24zm0 0" fill="#f60" class="active-path"></path><path d="m143.191406 272h304.335938c-3.992188-35.945312-34.527344-64-71.527344-64h-192c-11.128906 0-22.257812 2.625-32.191406 7.601562l-11.214844 5.605469c-1.363281 10.3125-6.609375 19.320313-14.289062 25.609375 8.542968 5.769532 14.757812 14.742188 16.886718 25.183594zm0 0" fill="#f60" class="active-path"></path><path d="m413.273438 80-29.882813-69.734375-33.503906 92.140625-9.605469-14.40625h-52.28125v-72c0-8.824219-7.175781-16-16-16h-80c-8.824219 0-16 7.175781-16 16v40h-69.273438l-18.117187 42.265625-30.488281-83.859375-22.402344 33.59375h-35.71875v16h44.28125l9.597656-14.40625 33.503906 92.140625 29.890626-69.734375h58.726562v40c0 8.824219 7.175781 16 16 16h16v24h16v40h16v-40h16v-24h16c8.824219 0 16-7.175781 16-16v-8h43.71875l22.402344 33.59375 30.496094-83.859375 18.109374 42.265625h61.273438v-16zm-157.273438-16h-16v16h-16v-16h-16v-16h16v-16h16v16h16zm0 0" fill="#f60" class="active-path"></path><path d="m424 417.472656v-97.472656h-16v97.472656c-9.304688 3.304688-16 12.09375-16 22.527344 0 13.257812 10.742188 24 24 24s24-10.742188 24-24c0-10.433594-6.695312-19.222656-16-22.527344zm0 0" fill="#f60" class="active-path"></path></g></svg>';
            resize();
            canvg('canvas', svg, {
                ignoreMouse: true,
                ignoreAnimation: true,
                renderCallback: function () {
                    console.info('svg 转换为 canvas 完成!');
    
                    // 这个便是 base64 的图片地址
                    // 切记图片格式为 png，如果是 jpg 或者 jpeg，那么输出的 base64 的图片背景不是透明的，是黑色的！！！
                    dataURL = canvas.toDataURL('image/png');
                    console.info(dataURL)
                },
                forceRedraw: function () {
                    var update = redraw;
                    redraw = false;
                    return update;
                }
            });
        }
    
        window.onresize = resize;
    </script>
</head>
<body onload="bodyonload();">
<div id="container">
    <canvas id="canvas"></canvas>
</div>
</body>
</html>
```

由于Mac开发环境，没有IE，所以那个JS文件我就懒得找了。

以上文件保存为 test.html，然后执行以下命令，即可在浏览器打开 http://localhost:8888 查看效果，前提是php环境装好了。

```shell
php -S localhost:8888 test.html
```
 
参考文章：[svg转化成canvas以便生成base64位的图片](https://www.cnblogs.com/zsslll/p/6133856.html)


