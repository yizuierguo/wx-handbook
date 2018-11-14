# 跨页面通信

## 业务场景

假设有两个页面：用户列表页、信息编辑页，在列表中点击后某条信息后，进入编辑页面 
![](https://assets.51wakeup.com/assets/2018/daily-work/markdown/01.png)
修改了用户信息后，返回到列表页，列表中需要显示修改后的信息,
例如把 “李四” 改为了 “李六”，那么返回列表页后，第2条记录就应该显示的是 “李六”

### 如何更新？

可以马上想到的方案：

方案一：可以重新加载列表，返回到列表页时，触发的是onShow事件，那么就在 onShow 处理函数中重新请求数据进行加载
但这样做不太好处理用户体验问题，例如修改的是经过多次下拉翻页后的某条用户信息

方案二：也可以不用重新加载，在保存之后设置缓存，指明修改的用户ID、修改后的数据，然后在列表页的onShow处理函数中读取缓存，直接修改现有列表中的数据

总之，上述两种方案都不是很好，解决上面的更新方式都不太优雅，建议使用broadcast广播机制

列表页设置监听，编辑页修改完成后发送广播通知

列表页：


`
const broadcast = require("./utils/broadcast");

..
onLoad(options){
...
   broadcast.on("broadcast_user_modified",(data)=>{
   
   //处理逻辑..
   
   });
...

}

`

编辑页：

`
const broadcast = require("./utils/broadcast");
...

//广播

const data = {
     userid:user_id
}

broadcast.fire("broadcast_user_modified",data);
...

`

[github](https://github.com/binnng/broadcast.js)