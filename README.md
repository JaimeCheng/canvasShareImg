[在线阅读地址](https://www.jianshu.com/p/01f526a4f948)**

小程序的入口主要集中了以下3个：
* 二维码
* 搜索栏
* 聊天转发

然而并没有流量大户——朋友圈；虽然小程序不能分享到朋友圈，但其并没有把长按识别二维码的功能阉割；于是乎在朋友圈疯狂转发的便成了一张带小程序码的图片；
这个功能的实现，在我第一次写小程序的时候困扰了很久；最开始想的重点都在带参数的小程序码上，查资料的时候偏离了轨道，一番搜索之后无疾而终，遂把这锅扔给了服务端，但仍耿耿于怀；第二次又是一番搜索，理清了头绪，就是canvas画一张图的事儿，但canvas我很少写，带着恐惧便没多尝试，一半demo被搁置；N天之后，又想起这事，豁然开朗：) 

## 思路分解
 **源码见文章底部**
**1.** 二维码如果带有参数 就有服务端发送过来 我们需要的只是图片的url；
**2.** 把用到的图片和文字绘制到画布上 `wx.createCanvasContext()`；
**3.** 用户点击生成分享图的时候。把画布转成图片`wx.canvasToTempFilePath()`一直到这里，以上操作对用户都是不可见的,此时生成图片后API会返回图片的路径url，这时可以把拿到的图片url放到一个`<image>`里展示给用户了；
**4.** 再来一个按钮把展示的图片调用`wx.saveImageToPhotosAlbum()`保存到相册中，完结撒花；

![canvas生成分享图](https://upload-images.jianshu.io/upload_images/3981371-37ced7b3fb0baaf0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

真的只是canvas画张图的事儿啊！！！为什么之前脑子像堵塞一般？现在切入正题，一步一步实现生成分享图并保存到相册的功能；

### 1. 准备工作
当然是准备一张画布了 o(*￣︶￣*)o  顺便把页面结构一起画一下···
```html
<!-- canvas.wxml -->
<!-- 画布大小按需定制 这里我按照背景图的尺寸定的  -->
<canvas  canvas-id="shareImg" style="width:545px;height:771px"></canvas>

<!-- 生成分享图 这里的操作是把canvas绘制的图预览出来  -->
<button class='share' type='primary' bindtap='share'>生成分享图</button>

<!-- 预览分享图 这里就是上图展示的效果   -->
<!-- 刚开始是隐藏的 生成分享图之后显示, 用一个布尔变量来控制 这里的样式大家看图就写出来了 -->
<view hidden='{{hidden}}' class='preview'>
  <image src='{{prurl}}' mode='widthFix'></image>
  <button type='primary' size='mini' bindtap='save'>保存分享图</button>
</view>
```

```css
/* pages/canvas/canvas.wxss */
canvas{
  position: fixed;
  top: 0;
  left: 999px;
}
/* 画布绘制过程不能隐藏 但是又不能被用户看见 所以定位在用户看不到地方 */
```

### 2. 绘制画布内容
以图中为例，画布分为三部分： 背景图 、二维码图、描述文字；
绘图和文字的方法 看[小程序开发文档](https://developers.weixin.qq.com/miniprogram/dev/api/canvas/CanvasContext.html)即可 写的很清楚；
这里注意绘制图片到画布之前 要先调用` wx.getImageInfo()`获取图片信息；
废话不多说，直接上代码；
```js
/* promise可以忽略 是用来改善异步回调执行顺序 与本功能没有大的关系 */

let promise1 = new Promise(function (resolve, reject) {

    /* 获得要在画布上绘制的图片 */
    wx.getImageInfo({
        src: '../../public/pics/qrcode.jpg',
        success: function (res) {
          console.log(res)
          resolve(res);
        }
   })
});
let promise2 = new Promise(function (resolve, reject) {
    wx.getImageInfo({
        src: '../../public/pics/qrbg.png',
        success: function (res) {
          console.log(res)
          resolve(res);
        }
   })
 });

/* 图片获取成功才执行后续代码 */
 Promise.all(
   [promise1,promise2]
 ).then(res => {
    console.log(res)

    /* 创建 canvas 画布 */
    const ctx = wx.createCanvasContext('shareImg')

    /* 绘制图像到画布  图片的位置你自己计算好就行 参数的含义看文档 */
    /* ps: 网络图片的话 就不用加../../路径了 反正我这里路径得加 */
    ctx.drawImage('../../' + res[0].path, 158, 190, 210, 210)
    ctx.drawImage('../../'+res[1].path, 0, 0, 545, 771)
  
    /* 绘制文字 位置自己计算 参数自己看文档 */
    ctx.setTextAlign('center')                        //  位置
    ctx.setFillStyle('#ffffff')                       //  颜色
    ctx.setFontSize(22)                               //  字号
    ctx.fillText('分享文字描述', 545 / 2, 130)         //  内容  不会自己换行 需手动换行
    ctx.fillText('分享文字描述', 545 / 2, 160)         //  内容
    
    /* 绘制 */
    ctx.stroke()
    ctx.draw()
    })
```
### 3. 画布生成图片
点击`生成分享图`时候触发该事件；
直接调用API`wx.canvasToTempFilePath()`即可；
附上链接[wx.canvasToTempFilePath()](https://developers.weixin.qq.com/miniprogram/dev/api/canvas/wx.canvasToTempFilePath.html) ;
```js
var that =this
wx.canvasToTempFilePath({
      x: 0,
      y: 0,
      width: 545,
      height: 771,
      destWidth: 545,
      destHeight: 771,
      canvasId: 'shareImg',
      success: function (res) {
        console.log(res.tempFilePath);
        /* 这里 就可以显示之前写的 预览区域了 把生成的图片url给image的src */
        that.setData({
          prurl: res.tempFilePath,
          hidden:false
        })
      },
      fail: function (res) {
        console.log(res)
      }
    })
```
### 4. 保存图片到相册
保存图片到相册，这里记得要获取授权(＾Ｕ＾)ノ~ＹＯ
直接调用API`wx.canvasToTempFilePath()`即可；
附上链接[wx.saveImageToPhotosAlbum()](https://developers.weixin.qq.com/miniprogram/dev/api/media/image/wx.saveImageToPhotosAlbum.html) ;
```js
var that =this
  wx.saveImageToPhotosAlbum({
      filePath: that.data.prurl,
      success(res) {
        wx.showModal({
          content: '图片已保存到相册，赶紧晒一下吧~',
          showCancel: false,
          confirmText: '好的',
          confirmColor: '#333',
          success: function (res) {
           if (res.confirm) {
              console.log('用户点击确定');
              /* 该隐藏的隐藏 */
              that.setData({
                hidden:true
              })
            }
          }
        })
      }
  })
```
###  ^ _ ^ 完结撒花，是不是非常简单
以上代码仅供参考，剩下的全靠freestyle了，有任何问题可与我留言；如果有帮到您，请别吝啬您的star和喜欢~
demo：[canvasShareImg](https://github.com/JaimeCheng/canvasShareImg)

###  P·S 一个重要问题：圆角头像以及带用户头像的小程序码以下代码放在绘制canvas的最后步骤。头像调好大小位置覆盖在二维码中间位置就好了。
```js
      ctx.save() //这句很重要  保存之前的绘制
      var r = 105
      var cx = 158 + r
      var cy = 190 + r
      ctx.arc(cx, cy, r, 0, 2 * Math.PI)
      ctx.clip()
      ctx.drawImage("图片path", 158, 190, 210, 210)
      ctx.restore() 
      ctx.draw()
```

**任何问题欢迎留言**
