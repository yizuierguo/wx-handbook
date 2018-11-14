# 小程序开发基础

## 准备工作
如果你已经注册或者认证了一个小程序，那么你已经拿到了appid，现在需要下载[微信开发者工具](https://developers.weixin.qq.com/miniprogram/dev/devtools/download.html)，开发者工具提供一个配置文件 project.config.json，可以在不同的设备中同步开发者工具的配置。官方介绍如下：

> 小程序开发者工具在每个项目的根目录都会生成一个 project.config.json，你在工具上做的任何配置都会写入到这个文件，当你重新安装工具或者换电脑工作时，你只要载入同一个项目的代码包，开发者工具就自动会帮你恢复到当时你开发项目时的个性化配置，其中会包括编辑器的颜色、代码上传时自动压缩等等一系列选项。...

更多配置项细节可以参考官方文档 [开发者工具的配置](https://developers.weixin.qq.com/miniprogram/dev/devtools/edit.html#%E9%A1%B9%E7%9B%AE%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6)。

### 开发语言

小程序采用 WXML + WXSS + JS 三种开发语言组合，其和网页编程采用的 HTML + CSS + JS 类似，WXML 用来描述当前这个页面的结构，WXSS 用来描述页面的样式，JS 用来处理这个页面和用户的交互。

### WXML

WXML（WeXin Markup Language）和 HTML 类似，也有标签和属性，但针对小程序平台做了些优化。

相较 HTML，小程序的标签显得更加简洁，比如 div、section 、header等块级标签统一为 view，p、span、b 等文案类标签统一为 text，同时也新增了很多实用标签，比如 picker 滚动选择器、map 地图、web-view 网页容器等。

可以简单理解为，小程序所有的标签都是[原生组件](https://developers.weixin.qq.com/miniprogram/dev/component/)。

WXML 支持数据绑定：

`<view> {{message}} </view>`


在 JS 中定义 message 变量：

`
Page({
  data: {
    message: 'Hello MOBIKE!'
  }
})`

更多关于 WXML 的介绍请见 [WXML · 小程序](https://developers.weixin.qq.com/miniprogram/dev/framework/view/wxml/)。

### WXSS

WXSS（WeXin Style Sheets）是微信定义的一套样式语言，其具有 CSS 大部分特性，同时为了更适合开发微信小程序，WXSS 对 CSS 进行了扩充以及修改。

小程序使用 rpx（responsive pixel）作为尺寸单位。屏幕宽度固定为 750rpx，设置了 rpx 单位的元素可以根据屏幕宽度进行自适应，所以设计稿统一以 750px 输出（iPhone 6 标准）。

小程序没有 html 、body标签，如果想要设置页面的样式，可以直接使用 page 选择器：

`page{
  background: #FFFFFF;
}
`
WXSS 对选择器的支持并没有 Web 那么丰富，比如属性选择器 [attr]、相邻选择器 h1 + p 等都不被支持，只支持以下几种：
![](https://assets.51wakeup.com/assets/2018/daily-work/markdown/03.png)
同时，我们也可以在 WXML 中写内联样式：

`<view style="color: #FF9900;" />`

也支持变量：

`<view style="color: {{color}};" />`

更多关于 WXSS 的介绍请见 [WXSS · 小程序](https://developers.weixin.qq.com/miniprogram/dev/api/)。

### 生命周期

和大多数前端框架类似，小程序也有生命周期。

### App 生命周期

整个小程序的生命周期如下图所示：

| 属性   |      类型|  描述 | 触发时机 | 
|----------|:-------------:|------:|------:|
| onLaunch |  Function | 生命周期函数--监听小程序初始化 | 当小程序初始化完成时，会触发 onLaunch（全局只触发一次）|
| onShow   |  Function | 生命周期函数--监听小程序显示   | 当小程序启动，或从后台进入前台显示，会触发 onShow | 
| onHide   |  Function | 生命周期函数--监听小程序隐藏   | 当小程序从前台进入后台，会触发 onHide |

App 的生命周期是在小程序启动后全局监控的，不随着页面切换变化而变化。通过 App() 注册小程序，传入对应的生命周期函数，具体文档见[注册程序](https://developers.weixin.qq.com/miniprogram/dev/framework/app-service/app.html)。

### Page 生命周期
除了整个小程序 App 的生命周期，每个页面（Page）也有自己的生命周期：


| 属性   |      类型|  描述 |
|----------|:-------------:|------:|
| onLoad |  Function | 生命周期函数--监听页面加载 |
| onReady   |  Function | 生命周期函数--监听页面初次渲染完成   | 
| onShow   |  Function | 生命周期函数--监听页面显示   | 
| onHide   |  Function | 生命周期函数--监听页面隐藏  | 
| onUnload   |  Function | 生命周期函数--监听页面卸载  |


## 享贞小程序









