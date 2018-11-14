# h5微信支付

> [官方文档](https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=7_7&index=6)

> [微信支付简介](https://wohugb.gitbooks.io/wechat-pay/content/intro.html)

### 业务场景

在微信客户端环境下，H5页面调起微信支付。

### 准备工作

1、需要有一个有**支付功能**的公众号，即[支付账户](https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=3_1)，用户所支付的金额会到支付账户中；

2、在微信商户平台（pay.weixin.qq.com）设置公众号支付的支付授权目录，设置路径：商户平台-->产品中心-->开发配置；（因为公众号支付在请求支付的时候会**校验请求来源是否有在商户平台做了配置**，所以必须确保支付目录已经正确的被配置，否则将验证失败，请求支付不成功）

### 开始开发

微信有一个内置对象WeixinJSBridge（该内置对象在其他浏览器中无效），我们可以通过调用WeixinJSBridge.invoke（'getBrandWCPayRequest',obj）方法调起微信支付，需要配置obj参数和返回值如下：

**参数：**

|名称|变量名|必填|类型|描述|
|  --- 	|   ---	   | -- |    ---   | ----------- |
|公众号id|  appId   | 是 |String(16)|商户注册具有支付权限的公众号成功后即可获得|
|时间戳  | timeStamp| 是 |String(32)|当前的时间|
|随机字符串|nonceStr| 是 |String(32) |随机字符串，不长于32位|
|订单详情扩展字符串|package| 是 |String(128)|统一下单接口返回的prepay_id参数值|
|签名方式| signType  | 是 |String(32)|签名类型，默认为MD5，支持HMAC-SHA256和MD5。注意此处需与统一下单的签名类型一致|
|签名|   paySign   | 是 |String(64)|签名|

    注：appId、timeStamp、nonceStr、package、signType这些参数都可通过请求后端接口获取

**返回值：**

|返回值|描述|
|--|--|
|get_brand_wcpay_request:ok|支付成功|
|get_brand_wcpay_request:cancel|支付过程中用户取消|
|get_brand_wcpay_request:fail|支付失败|
|调用支付JSAPI缺少参数：total_fee|请检查预支付会话标识prepay_id是否已失效|

示例代码如下：

    function onBridgeReady(){
	   WeixinJSBridge.invoke(
	      'getBrandWCPayRequest', {
	         "appId":"wx2421b1c4370ec43b",     //公众号名称，由商户传入     
	         "timeStamp":"1395712654",         //时间戳，自1970年以来的秒数     
	         "nonceStr":"e61463f8efa94090b1f366cccfbbb444", //随机串     
	         "package":"prepay_id=u802345jgfjsdfgsdg888",     
	         "signType":"MD5",         //微信签名方式：     
	         "paySign":"70EA570631E4BB79628FBCA90534C63FF7FADD89" //微信签名 
	      },
	      function(res){
	      if(res.err_msg == "get_brand_wcpay_request:ok" ){
	      // 使用以上方式判断前端返回,微信团队郑重提示：
	            //res.err_msg将在用户支付成功后返回ok，但并不保证它绝对可靠。
	      } 
	   }); 
	}
	if (typeof WeixinJSBridge == "undefined"){
	   if( document.addEventListener ){
	       document.addEventListener('WeixinJSBridgeReady', onBridgeReady, false);
	   }else if (document.attachEvent){
	       document.attachEvent('WeixinJSBridgeReady', onBridgeReady); 
	       document.attachEvent('onWeixinJSBridgeReady', onBridgeReady);
	   }
	}else{
	   onBridgeReady();
	}

如果调支付成功，就会自动弹出微信支付弹窗让用户输入密码进行支付。

错误提示及解决方法：

  “支付场景非法”：若用户未授权登录过该公众号，则在调支付时可能会提示“支付场景非法”，所以需要在调支付前通过请求后端接口来判断用户有没有授权登录过该公众号，若用户未授权登录过，则需要进行[授权登录](part2/h5-login.md)

...
