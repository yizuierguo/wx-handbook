# 分享

分享功能是产品设计中重要的一环，旨在吸引更多的潜在用户。微信小程序运行在微信环境中，具有天然的优势。

因为现在小程序不支持直接分享到朋友圈功能，所以分享目前的通用逻辑如下：
![](https://assets.51wakeup.com/assets/2018/daily-work/markdown/04.jpg)

## 分享到会话

在页面中，定义 onShareAppMessage(options) 方法，来设置该页面的转发信息。如果想要在页面内发起转发，需要通过给 button 组件设置属性 open-type="share"，需要注意的是只能是 button 组件，更多API可以参考 [转发](https://developers.weixin.qq.com/miniprogram/dev/api/share.html)。
示例：

`

	Page({
	  onShareAppMessage: function (res) {
	    // 区分页面右上角分享和页面内 button 点击
	    if (res.from === 'button') {
	      // 来自页面内 button 转发按钮
	      console.log(res.target)
	    }
	    return {
	      title: '开小课',
	      path: '/page/***/index',
	      imageUrl: 'https://assets.51wakeup.com/***.png'
	    }
	  }
	})


`

参数说明：

> title: 分享文案

> path: 分享页面路径

> imageUrl: 分享图片，可以是本地文件路径或者网络图片路径，建议使用网络图片。当不传入 imageUrl 时，会默认使用截图，显示图片的长宽比是 5:4。

### 获取群标识

假如有个需求要求统计分享到不同群的个数，那我们如何拿到群标识呢？主要是通过 wx.getShareInfo(OBJECT) 方法获取，需要传入 shareTicket。

1. 定义携带 shareTicket 的转发
2. 在分享时，获取 shareTicket 
3. 根据 shareTicket 获取群标识...

`
	 
	  Page({
	  onLoad: function (options) {
	    // 定义携带 shareTicket 的转发
	    wx.showShareMenu({ withShareTicket: true })
	    ...
	  },
	  onShareAppMessage: function (res) {
	    let that = this
	    return {
	      title: '开小课',
	      path: `/pages/***/index`,
	      imageUrl: '***',
	      success: function (res) {
	        // 获取 shareTicket
	        var shareTickets = res.shareTickets
	        if (shareTickets && shareTickets.length !== 0) {
	          wx.getShareInfo({
	            shareTicket: shareTickets[0],
	            success: function(res) {
	              console.log('加密之后的群标识：', res.iv)
	            })
	          }
	        }
	      }
	    }
	  }
	})...

`
### 模版消息

假如有这样一个功能点：当用户收到待领取的红包时，会在微信服务通知 中，收到一条模版消息。下面来看一下如何发送一条模版消息。
1、 登录 [微信公众平台|小程序](https://mp.weixin.qq.com/wxopen/home?lang=zh_CN&token=1793018663) 获取或者新定义模版，参考下图（来自摩拜单车红包逻辑）定义字段。
![](https://assets.51wakeup.com/assets/2018/daily-work/markdown/05.jpg)

2、用户行为，触发模版消息
模版消息的触发条件有两个，一个是支付，另一个是提交表单。我们的触发场景不是用户支付，那只能把分享按钮做成表单。

`
	  
	<form report-submit="true" bindsubmit="shareBtnTap">
	  <button formType="submit" open-type="share">发红包 赚赏金</button>
	</form>
	
	Page({
	  shareBtnTap (e) {
	    // 获取formId
	    console.log('用于发送模版消息的formId: ', e.detail.formId)
	  },
	})...

`

其中，report-submit 属性，表示是否返回 formId。最后在 shareBtnTap 方法中，获取到 formId。

3、 调用接口下发模版消息

在该需求中，模版消息的发送，是在某个特定时机触发，这部分由后台来执行。调用微信接口，传递 模版id、formId、模版内容等。

`

		{
		  "touser": "OPENID",  // 接收用户的openid
		  "template_id": "TEMPLATE_ID",  // 模版id
		  "page": "index",  // 用户点击后的跳转页面
		  "form_id": "FORMID",
		  "data": {
		      "keyword1": {
		          "value": "339208499"
		      },
		      "keyword2": {
		          "value": "2015年01月05日 12:30"
		      },
		      "keyword3": {
		          "value": "粤海喜来登酒店"
		      } ,
		      "keyword4": {
		          "value": "广州市天河区天河路208号"
		      }
		  },
		  "emphasis_keyword": "keyword1.DATA"  // 模板需要放大的关键词，不填则默认无放大
		}
  
`

说明：

* keyword ：配置的文案，如上图所示
* emphasis_keyword ： 模板需要放大的关键词，上图中的“一份现金奖励”文案就是通过此方法放大显示的。


### 分享到朋友圈
















