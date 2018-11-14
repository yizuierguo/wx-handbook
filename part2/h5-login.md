# h5微信登录

> [官方文档](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421140842)

微信授权登录可以获取用户信息，是后续很多数据交互的前提。

### 业务场景

需要获取用户的信息，弹出微信微信授权登录页面，让用户授权登录

### 准备工作

在微信公众号请求用户网页授权之前，开发者需要先到微信公众平台官网中左侧菜单栏的“开发 - 接口权限 - 网页服务 - 网页授权获取用户基本信息”的配置选项中，修改授权回调域名。请注意，这里填写的是域名（是一个字符串），而不是URL，因此请勿加 http:// 等协议头；

### 开始开发

在需要用户授权登录的地方直接使用window.location.href跳转到微信的授权登录页面
	
	window.location.href = https://open.weixin.qq.com/connect/oauth2/authorize?appid=APPID&redirect_uri=REDIRECT_URI&response_type=code&scope=SCOPE&state=STATE#wechat_redirect

若提示“该链接无法访问”，请检查参数是否填写错误，是否拥有scope参数对应的授权作用域权限。

**尤其注意：**由于授权操作安全等级较高，所以在发起授权请求时，微信会对授权链接做正则强匹配校验，**如果链接的参数顺序不对，授权页面将无法正常访问**

参数说明：

|    参数     | 必填 |              说明               |
| ------     |  --  |              --------           |
|    appid   |  是  |       公众号的唯一标识        |
|redirect_uri|  是  | 授权后重定向的回调链接地址， 请使用 urlEncode 对链接进行处理|
|response_type|  是 |  返回类型，请填写code        |
|   scope    |  是  |应用授权作用域，snsapi_base （不弹出授权页面，直接跳转，只能获取用户openid），snsapi_userinfo （弹出授权页面，可通过openid拿到昵称、性别、所在地。并且， 即使在未关注的情况下，只要用户授权，也能获取其信息 ）|
|   state   |  否  | 重定向后会带上state参数，开发者可以填写a-zA-Z0-9的参数值，最多128字节|
|#wechat_redirect| 是 |无论直接打开还是做页面302重定向时候，必须带此参数|

下图为scope等于snsapi_userinfo时的授权页面：

![](https://wakeup-images.oss-cn-hangzhou.aliyuncs.com/assets/2018/daily-work/markdown/h5-login1.jpg)

如果用户同意授权，页面将跳转至 redirect_uri/?code=CODE&state=STATE。

    code说明 ： code作为换取access_token的票据，每次用户授权带上的code将不一样，code只能使用一次，5分钟未被使用自动过期。

现在我们已经拿到了用户授权登录的**code**了，然后就可以通过请求后台数据接口将**code**发给后端，在后端进行后续的登录逻辑处理，然后后端给前端返回用户**token**，后续的接口请求带上这个**token**，登录就ok了。

> 说白了，上述所有操作就是为了**拿code去换取token**