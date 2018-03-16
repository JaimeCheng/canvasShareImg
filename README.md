# canvasShareImg
微信小程序 canvas生成分享图 保存到相册发票圈 Demo

## 思路分解
**1.** 二维码如果带有参数 就有服务端发送过来 我们需要的只是图片的url；

**2.** 把用到的图片和文字绘制到画布上 `wx.createCanvasContext()`；

**3.** 用户点击生成分享图的时候。把画布转成图片`wx.canvasToTempFilePath()`一直到这里，以上操作对用户都是不可见的,此时生成图片后API会返回图片的路径url，这时可以把拿到的图片url放到一个`<image>`里展示给用户了；

**4.** 再来一个按钮把展示的图片调用`wx.saveImageToPhotosAlbum()`保存到相册中，完结撒花；

**更详细的解释见 <a href="https://www.jianshu.com/p/01f526a4f948" target="_blank">微信小程序 canvas生成分享图</a>**

**任何问题欢迎留言**
