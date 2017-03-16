---
layout: post
title: 使用html5ImgCompress压缩图片并上传服务器
categories: [前端]
description: 使用html5ImgCompress压缩图片并上传服务器
keywords: html5ImgCompress, Blob, base64
---
产品开发中经常会遇到上传图片的场景，最近一个项目中就遇到了这样的需求，开发完成后发现在手机上选择相册照片上传速度非常慢，尤其是照片多张一起上传你的时候。

原因嘛也很简单，现在手机相机像素都很高，随便一张照片动辄4~5M，如果4张照片一起传，就将近20M，手机4G状态下上传也要好几秒，而且太耗流量对用户也是一种不负责的表现。

所以自然而然就有了压缩上传的需求，网上找了一通，发现[html5ImgCompress](https://github.com/mhbseal/html5ImgCompress)这个HTML5图片压缩库不错，支持pc，mobile（ios/android）和multiple。
### 使用html5ImgCompress压缩图片
html5ImgCompress压缩图片使用非常简单，直接以图片文件做为输入，可以得到压缩后图片的base64字符串，后续在对base64值做操作就行，将base64值直接赋予img标签src属性，可以直接在浏览器预览压缩后的图片，或者上传base64图片，或者将base64图片生成类文件的blob再上传。

引用github主页的例子（github上done回调函数两个参数写颠倒了，应该是file, base64）：
```javascript
<input type="file" />
<script>
  document.getElementsByTagName('input')[0].addEventListener("change", function(e) {
    new html5ImgCompress(e.target.files[0], {
      before: function(file) {
        console.log('压缩前...');
        // 这里一般是对file进行filter，例如用file.type.indexOf('image') > -1来检验是否是图片
        // 如果为非图片，则return false放弃压缩（不执行后续done、fail、complete），并相应提示
      },
      done: function (file, base64) {
        console.log('压缩成功...');
        // ajax和服务器通信上传base64图片等操作
      },
      fail: function(file) {
        console.log('压缩失败...');
      },
      complete: function(file) {
        console.log('压缩完成...');
      },
      notSupport: function(file) {
        console.log('浏览器不支持！')
        // 不支持操作，例如PC在这里可以采用swfupload上传
      }
    });
  }, false)
</script>
```
使用img预览压缩后base64的demo可以看[这里](http://mhbseal.com/demo/html5/html5ImgCompress/demo/index.html)，这里不再累述，下面重点介绍压缩后图片如何上传服务器。
### 压缩后上传服务器
使用html5ImgCompress压缩后上传到服务器有两种方案： 
1. 直接将压缩后的base64上传服务器，服务端需要专门订制开发
2. 将base64转化为Blob对象，服务端可以复用原来的文件上传逻辑
显然方法2比1要通用，对已有业务修改最小，所以这里我们只展示了方法2的实现：

<script async src="//jsfiddle.net/gitfree/tgh5gwjm/5/embed/js,html,result/"></script>
