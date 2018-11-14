# 移动端视频播放白皮书(V0.0.1)

> 
> 随着4G的普遍以及WiFi的广泛使用，手机上的网速已经足够稳定和高速，以视频为主的 HTML5 也越来越普遍了，相比帧动画，视频的表现更加丰富，这里介绍一些实践经验。


## ChangeLog：

| 版本  |  author  |  主要变动 | 备注  |
|:----------:|:-------------:|:------:|:------:|
| 0.0.1 |  一醉 | 创建本白皮书的第一版本 | 2018-07-25 |
| ... | ... | ... | ... |

## 相关连接：

> [HTML5`<video>`标签](http://www.w3school.com.cn/tags/tag_video.asp)

> [iphone-inline-video:解决ios同层播放的问题](https://github.com/bfred-it/iphone-inline-video)

> [腾讯浏览服务](https://x5.tencent.com/tbs/guide/video.html) 

> [HTML5 Video Events and API](https://www.w3.org/2010/05/video/mediaevents.html)

> [chimee播放器](http://chimee.org/)

> [西瓜视频播放器](http://h5player.bytedance.com/)



## 大前提：

1. 由于PC/iOS/Android这些不同平台、不同的浏览器内核、甚至相同内核的不同版本，所实现的`<video>`属性、方法和事件差异较大，解决兼容性问题又给开发造成了很大困扰，所以，如果可能不建议在html5(后简称H5)中实现`video`播放功能，请引导到app中播放。
2. 由于1的存在，所以产品方案（设计，实现等）在规划视频播放的时候，请务必仔细阅读此白皮书
3. 由于浏览器厂商及`html5 video`标签的属性方法事件的支持程度随着时间的推移会逐渐变化，所以，此白皮书会迭代更新，请关注changlog信息
4. 由于在非微信浏览器和微信浏览器下环境差异也很巨大，我们v0.0.1只考虑微信浏览器环境



## 本草案涉及到的测试机型（公司机型有限）：

| 机型   |      系统|  微信版本 | TBS（腾讯浏览服务）版本 |
|:----------:|:-------------:|:------:|:------:|
| ip6 |  iOS | 6.6.7 | - |
| ip7   |  iOS | 6.6.7    | - |
| ip8   |  iOS |  6.6.7   |- |
| ipx   |  iOS |  6.6.7  | - |
| 坚果pro   |  Android | 6.6.7   | |
| 小米   |  Android | 6.6.7   | |
| oppo   |  Android | 6.6.7   | |
| 华为   |  Android | 6.6.7  | 6.2 |




## 此白皮书主要涉及到的内容

1. 现状及造成这些现状的原因
2. `video`播放产品层主要关注的4个问题
3. 最佳实践及统一解决方案
4. 第三方视频播放方案汇总

## 一.现状及造成这些现状的原因

### 1.1`video`播放的现状
移动端HTML5使用原生`<video>`标签播放视频，要做到两个基本原则，速度快和体验佳，先来分析一下这两个问题。
#### 下载速度

以一个8s短视频为例，wifi环境下提供的高清视频达到1000kbps，文件大小大约1MB；非wifi环境下提供的低码率视频是500kbps左右，文件大小大约500KB；参考QzoneTouch多普勒（腾讯内部外网访问不到）测速，2g网络的平均速度是14KB/s，那么下载一个低码率视频耗时35s；那么要想流畅播放视频，就需要一个加载等待的过程，这个过程要有明确的反馈，不能让用户有“坏掉了”的感觉。


| 网络   |      dns(s)|  conn(Connection/s) | rtt([Round-Trip Time](https://blog.csdn.net/jackywangjia/article/details/27643379)/s) |tran(kb/s)|
|:----------:|:-------------:|:------:|:------:|:------:|
| 2g | 3.85785	| 2.33482 | 2.57478 | 14.0374 |
| 3g | 1.60643	| 0.743109 | 0.608047 | 60.1967 |
| wifi | 0.986921 | 0.550208 | 0.444332 | 70.8728 |

#### 用户体验
视频是否可以自动播放，是否能循环播放，是否能显示下载进度，播放的时候如何隐藏控制条，暂停的时候又能显示出来呢。这些问题看上去貌似简单，但是由于PC/iOS/Android这些不同平台、不同的浏览器内核、甚至相同内核的不同版本，所实现的`<video>`属性、方法和事件差异较大，解决兼容性问题又给开发造成了很大困扰。
android下html5的视频播放一直是前端兼容的重灾区，各种体验差，被诟病已久，看之前有人整理的一张图，自己亲测跟这个情况差不多，UI五花八门

![](https://assets.51wakeup.com/assets/2018/daily-work/markdown/06.jpg)

### 1.2造成现状的原因
造成这个现状的主要原因其实就是各大厂商对`video`标签的属性事件等支持标准不一致导致，具体如下：

#### 事件差异

下面是播放一个短视频，在不同平台触发事件和获取属性的差异表现，具体可以访问 [HTML5 Video Events and API](https://www.w3.org/2010/05/video/mediaevents.html)

#### PC

<table>
  <thead>
    <tr>
      <th>#</th>
      <th>event</th>
      <th>readyState</th>
      <th>currentTime (s)</th>
      <th>buffered (s)</th>
      <th>duration (s)</th>
      <th>视频状态</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>1</td>
      <td>loadstart</td>
      <td>NOTHING</td>
      <td>0</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
    </tr>
    <tr>
      <td>2</td>
      <td>suspend</td>
      <td>NOTHING</td>
      <td>0</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
    </tr>
    <tr>
      <td>3</td>
      <td>play</td>
      <td>NOTHING</td>
      <td>0</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
    </tr>
    <tr>
      <td>4</td>
      <td>waiting</td>
      <td>NOTHING</td>
      <td>0</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
    </tr>
    <tr>
      <td>5</td>
      <td>durationchange</td>
      <td>METADATA</td>
      <td>0</td>
      <td>5.35</td>
      <td>7.91</td>
      <td>获取到视频长度</td>
    </tr>
    <tr>
      <td>6</td>
      <td>loadedmetadata</td>
      <td>METADATA</td>
      <td>0</td>
      <td>0.66</td>
      <td>7.91</td>
      <td>获取到元数据</td>
    </tr>
    <tr>
      <td>7</td>
      <td>loadeddata</td>
      <td>ENOUGHDATA</td>
      <td>0</td>
      <td>0.66</td>
      <td>7.91</td>
      <td>-</td>
    </tr>
    <tr>
      <td>8</td>
      <td>canplay</td>
      <td>ENOUGH_DATA</td>
      <td>0</td>
      <td>0.66</td>
      <td>7.91</td>
      <td>-</td>
    </tr>
    <tr>
      <td>9</td>
      <td>playing</td>
      <td>ENOUGH_DATA</td>
      <td>0</td>
      <td>0.66</td>
      <td>7.91</td>
      <td>开始播放</td>
    </tr>
    <tr>
      <td>10</td>
      <td>canplaythrough</td>
      <td>ENOUGH_DATA</td>
      <td>0</td>
      <td>0.66</td>
      <td>7.91</td>
      <td>可以流畅播放</td>
    </tr>
    <tr>
      <td>11</td>
      <td>progress</td>
      <td>ENOUGH_DATA</td>
      <td>0.11</td>
      <td>3.68</td>
      <td>7.91</td>
      <td>持续下载</td>
    </tr>
    <tr>
      <td>12</td>
      <td>timeupdate</td>
      <td>ENOUGH_DATA</td>
      <td>0.14</td>
      <td>4.44</td>
      <td>7.91</td>
      <td>播放进度变化</td>
    </tr>
    <tr>
      <td>…</td>
      <td>…</td>
      <td>…</td>
      <td>…</td>
      <td>…</td>
      <td>…</td>
      <td>…</td>
    </tr>
    <tr>
      <td>23</td>
      <td>progress</td>
      <td>ENOUGH_DATA</td>
      <td>1.77</td>
      <td>7.91</td>
      <td>7.91</td>
      <td>下载完毕</td>
    </tr>
    <tr>
      <td>24</td>
      <td>suspend</td>
      <td>ENOUGH_DATA</td>
      <td>1.77</td>
      <td>7.91</td>
      <td>7.91</td>
      <td>-</td>
    </tr>
    <tr>
      <td>25</td>
      <td>timeupdate</td>
      <td>ENOUGH_DATA</td>
      <td>1.9</td>
      <td>7.91</td>
      <td>7.91</td>
      <td>继续播放中</td>
    </tr>
    <tr>
      <td>…</td>
      <td>…</td>
      <td>…</td>
      <td>…</td>
      <td>…</td>
      <td>…</td>
      <td>…</td>
    </tr>
    <tr>
      <td>48</td>
      <td>timeupdate</td>
      <td>ENOUGH_DATA</td>
      <td>7.7</td>
      <td>7.91</td>
      <td>7.91</td>
      <td>-</td>
    </tr>
    <tr>
      <td>49</td>
      <td>timeupdate</td>
      <td>ENOUGH_DATA</td>
      <td>0</td>
      <td>7.91</td>
      <td>7.91</td>
      <td>-</td>
    </tr>
    <tr>
      <td>50</td>
      <td>seeking</td>
      <td>METADATA</td>
      <td>0</td>
      <td>7.91</td>
      <td>7.91</td>
      <td>-</td>
    </tr>
    <tr>
      <td>51</td>
      <td>waiting</td>
      <td>METADATA</td>
      <td>0</td>
      <td>7.91</td>
      <td>7.91</td>
      <td>-</td>
    </tr>
    <tr>
      <td>52</td>
      <td>timeupdate</td>
      <td>ENOUGH_DATA</td>
      <td>0</td>
      <td>7.91</td>
      <td>7.91</td>
      <td>-</td>
    </tr>
    <tr>
      <td>53</td>
      <td>seeked</td>
      <td>ENOUGH_DATA</td>
      <td>0</td>
      <td>7.91</td>
      <td>7.91</td>
      <td>播放完毕进度回到起点</td>
    </tr>
    <tr>
      <td>54</td>
      <td>canplay</td>
      <td>ENOUGH_DATA</td>
      <td>0</td>
      <td>7.91</td>
      <td>7.91</td>
      <td>-</td>
    </tr>
    <tr>
      <td>55</td>
      <td>playing</td>
      <td>ENOUGH_DATA</td>
      <td>0</td>
      <td>7.91</td>
      <td>7.91</td>
      <td>循环播放</td>
    </tr>
    <tr>
      <td>56</td>
      <td>canplaythrough</td>
      <td>ENOUGH_DATA</td>
      <td>0</td>
      <td>7.91</td>
      <td>7.91</td>
      <td>-</td>
    </tr>
    <tr>
      <td>57</td>
      <td>timeupdate</td>
      <td>ENOUGH_DATA</td>
      <td>0.19</td>
      <td>7.91</td>
      <td>7.91</td>
      <td>-</td>
    </tr>
    <tr>
      <td>…</td>
      <td>…</td>
      <td>…</td>
      <td>…</td>
      <td>…</td>
      <td>…</td>
      <td>…</td>
    </tr>
  </tbody>
</table>


#### iOS

<table>
  <thead>
    <tr>
      <th>#</th>
      <th>event</th>
      <th>readyState</th>
      <th>currentTime (s)</th>
      <th>buffered (s)</th>
      <th>duration (s)</th>
      <th>视频状态</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>1</td>
      <td>loadstart</td>
      <td>NOTHING</td>
      <td>0</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
    </tr>
    <tr>
      <td>2</td>
      <td>suspend</td>
      <td>NOTHING</td>
      <td>0</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
    </tr>
    <tr>
      <td>3</td>
      <td>play</td>
      <td>NOTHING</td>
      <td>0</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
    </tr>
    <tr>
      <td>4</td>
      <td>waiting</td>
      <td>NOTHING</td>
      <td>0</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
    </tr>
    <tr>
      <td>5</td>
      <td>durationchange</td>
      <td>METADATA</td>
      <td>0</td>
      <td>5.35</td>
      <td>7.91</td>
      <td>获取到视频长度</td>
    </tr>
    <tr>
      <td>6</td>
      <td>loadedmetadata</td>
      <td>METADATA</td>
      <td>0</td>
      <td>0.66</td>
      <td>7.91</td>
      <td>获取到元数据</td>
    </tr>
    <tr>
      <td>7</td>
      <td>loadeddata</td>
      <td>ENOUGHDATA</td>
      <td>0</td>
      <td>0.66</td>
      <td>7.91</td>
      <td>-</td>
    </tr>
    <tr>
      <td>8</td>
      <td>canplay</td>
      <td>ENOUGH_DATA</td>
      <td>0</td>
      <td>0.66</td>
      <td>7.91</td>
      <td>-</td>
    </tr>
    <tr>
      <td>9</td>
      <td>playing</td>
      <td>ENOUGH_DATA</td>
      <td>0</td>
      <td>0.66</td>
      <td>7.91</td>
      <td>开始播放</td>
    </tr>
    <tr>
      <td>10</td>
      <td>canplaythrough</td>
      <td>ENOUGH_DATA</td>
      <td>0</td>
      <td>0.66</td>
      <td>7.91</td>
      <td>可以流畅播放</td>
    </tr>
    <tr>
      <td>11</td>
      <td>progress</td>
      <td>ENOUGH_DATA</td>
      <td>0.11</td>
      <td>3.68</td>
      <td>7.91</td>
      <td>持续下载</td>
    </tr>
    <tr>
      <td>12</td>
      <td>timeupdate</td>
      <td>ENOUGH_DATA</td>
      <td>0.14</td>
      <td>4.44</td>
      <td>7.91</td>
      <td>播放进度变化</td>
    </tr>
    <tr>
      <td>…</td>
      <td>…</td>
      <td>…</td>
      <td>…</td>
      <td>…</td>
      <td>…</td>
      <td>…</td>
    </tr>
    <tr>
      <td>23</td>
      <td>progress</td>
      <td>ENOUGH_DATA</td>
      <td>1.77</td>
      <td>7.91</td>
      <td>7.91</td>
      <td>下载完毕</td>
    </tr>
    <tr>
      <td>24</td>
      <td>suspend</td>
      <td>ENOUGH_DATA</td>
      <td>1.77</td>
      <td>7.91</td>
      <td>7.91</td>
      <td>-</td>
    </tr>
    <tr>
      <td>25</td>
      <td>timeupdate</td>
      <td>ENOUGH_DATA</td>
      <td>1.9</td>
      <td>7.91</td>
      <td>7.91</td>
      <td>继续播放中</td>
    </tr>
    <tr>
      <td>…</td>
      <td>…</td>
      <td>…</td>
      <td>…</td>
      <td>…</td>
      <td>…</td>
      <td>…</td>
    </tr>
    <tr>
      <td>48</td>
      <td>timeupdate</td>
      <td>ENOUGH_DATA</td>
      <td>7.7</td>
      <td>7.91</td>
      <td>7.91</td>
      <td>-</td>
    </tr>
    <tr>
      <td>49</td>
      <td>timeupdate</td>
      <td>ENOUGH_DATA</td>
      <td>0</td>
      <td>7.91</td>
      <td>7.91</td>
      <td>-</td>
    </tr>
    <tr>
      <td>50</td>
      <td>seeking</td>
      <td>METADATA</td>
      <td>0</td>
      <td>7.91</td>
      <td>7.91</td>
      <td>-</td>
    </tr>
    <tr>
      <td>51</td>
      <td>waiting</td>
      <td>METADATA</td>
      <td>0</td>
      <td>7.91</td>
      <td>7.91</td>
      <td>-</td>
    </tr>
    <tr>
      <td>52</td>
      <td>timeupdate</td>
      <td>ENOUGH_DATA</td>
      <td>0</td>
      <td>7.91</td>
      <td>7.91</td>
      <td>-</td>
    </tr>
    <tr>
      <td>53</td>
      <td>seeked</td>
      <td>ENOUGH_DATA</td>
      <td>0</td>
      <td>7.91</td>
      <td>7.91</td>
      <td>播放完毕进度回到起点</td>
    </tr>
    <tr>
      <td>54</td>
      <td>canplay</td>
      <td>ENOUGH_DATA</td>
      <td>0</td>
      <td>7.91</td>
      <td>7.91</td>
      <td>-</td>
    </tr>
    <tr>
      <td>55</td>
      <td>playing</td>
      <td>ENOUGH_DATA</td>
      <td>0</td>
      <td>7.91</td>
      <td>7.91</td>
      <td>循环播放</td>
    </tr>
    <tr>
      <td>56</td>
      <td>canplaythrough</td>
      <td>ENOUGH_DATA</td>
      <td>0</td>
      <td>7.91</td>
      <td>7.91</td>
      <td>-</td>
    </tr>
    <tr>
      <td>57</td>
      <td>timeupdate</td>
      <td>ENOUGH_DATA</td>
      <td>0.19</td>
      <td>7.91</td>
      <td>7.91</td>
      <td>-</td>
    </tr>
    <tr>
      <td>…</td>
      <td>…</td>
      <td>…</td>
      <td>…</td>
      <td>…</td>
      <td>…</td>
      <td>…</td>
    </tr>
  </tbody>
</table>

#### Android

<table>
  <thead>
    <tr>
      <th>#</th>
      <th>event</th>
      <th>readyState</th>
      <th>currentTime (s)</th>
      <th>buffered (s)</th>
      <th>duration (s)</th>
      <th>视频状态</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>1</td>
      <td>loadstart</td>
      <td>NOTHING</td>
      <td>0</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
    </tr>
    <tr>
      <td>2</td>
      <td>play</td>
      <td>NOTHING</td>
      <td>0</td>
      <td>-</td>
      <td>-</td>
      <td>-</td>
    </tr>
    <tr>
      <td>3</td>
      <td>waiting</td>
      <td>NOTHING</td>
      <td>0</td>
      <td>0</td>
      <td>-</td>
      <td>-</td>
    </tr>
    <tr>
      <td>4</td>
      <td>durationchange</td>
      <td>ENOUGH_DATA</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-</td>
    </tr>
    <tr>
      <td>5</td>
      <td>durationchange</td>
      <td>ENOUGH_DATA</td>
      <td>0</td>
      <td>0</td>
      <td>7.91</td>
      <td>获取到视频长度</td>
    </tr>
    <tr>
      <td>6</td>
      <td>loadedmetadata</td>
      <td>ENOUGH_DATA</td>
      <td>0</td>
      <td>0</td>
      <td>7.91</td>
      <td>获取到元数据</td>
    </tr>
    <tr>
      <td>7</td>
      <td>loadeddata</td>
      <td>ENOUGHDATA</td>
      <td>0</td>
      <td>0</td>
      <td>7.91</td>
      <td>-</td>
    </tr>
    <tr>
      <td>8</td>
      <td>canplay</td>
      <td>ENOUGH_DATA</td>
      <td>0</td>
      <td>0</td>
      <td>7.91</td>
      <td>-</td>
    </tr>
    <tr>
      <td>9</td>
      <td>canplaythrough</td>
      <td>ENOUGH_DATA</td>
      <td>0</td>
      <td>0</td>
      <td>7.91</td>
      <td>-</td>
    </tr>
    <tr>
      <td>10</td>
      <td>playing</td>
      <td>ENOUGH_DATA</td>
      <td>0</td>
      <td>0</td>
      <td>7.91</td>
      <td>-</td>
    </tr>
    <tr>
      <td>11</td>
      <td>timeupdate</td>
      <td>ENOUGH_DATA</td>
      <td>0</td>
      <td>0</td>
      <td>7.91</td>
      <td>-</td>
    </tr>
    <tr>
      <td>12</td>
      <td>progress</td>
      <td>ENOUGH_DATA</td>
      <td>0</td>
      <td>3.57</td>
      <td>7.91</td>
      <td>下载中</td>
    </tr>
    <tr>
      <td>13</td>
      <td>timeupdate</td>
      <td>ENOUGH_DATA</td>
      <td>0.2</td>
      <td>6.89</td>
      <td>7.91</td>
      <td>开始播放</td>
    </tr>
    <tr>
      <td>14</td>
      <td>progress</td>
      <td>ENOUGH_DATA</td>
      <td>0</td>
      <td>7.91</td>
      <td>7.91</td>
      <td>下载完毕</td>
    </tr>
    <tr>
      <td>…</td>
      <td>…</td>
      <td>…</td>
      <td>…</td>
      <td>…</td>
      <td>…</td>
      <td>…</td>
    </tr>
    <tr>
      <td>49</td>
      <td>timeupdate</td>
      <td>ENOUGH_DATA</td>
      <td>7.79</td>
      <td>7.91</td>
      <td>7.91</td>
      <td>-</td>
    </tr>
    <tr>
      <td>50</td>
      <td>progress</td>
      <td>ENOUGH_DATA</td>
      <td>7.87</td>
      <td>7.91</td>
      <td>7.91</td>
      <td>-</td>
    </tr>
    <tr>
      <td>51</td>
      <td>timeupdate</td>
      <td>ENOUGH_DATA</td>
      <td>0</td>
      <td>7.91</td>
      <td>7.91</td>
      <td>-</td>
    </tr>
    <tr>
      <td>52</td>
      <td>seeking</td>
      <td>ENOUGH_DATA</td>
      <td>0</td>
      <td>7.91</td>
      <td>7.91</td>
      <td>播放完毕进度回到起点</td>
    </tr>
    <tr>
      <td>53</td>
      <td>timeupdate</td>
      <td>ENOUGH_DATA</td>
      <td>0</td>
      <td>7.91</td>
      <td>7.91</td>
      <td>-</td>
    </tr>
    <tr>
      <td>54</td>
      <td>seeked</td>
      <td>ENOUGH_DATA</td>
      <td>0</td>
      <td>7.91</td>
      <td>7.91</td>
      <td>循环播放失败卡住了</td>
    </tr>
    <tr>
      <td>55</td>
      <td>progress</td>
      <td>ENOUGH_DATA</td>
      <td>0</td>
      <td>7.91</td>
      <td>7.91</td>
      <td>-</td>
    </tr>
    <tr>
      <td>56</td>
      <td>stalled</td>
      <td>ENOUGH_DATA</td>
      <td>0</td>
      <td>7.91</td>
      <td>7.91</td>
      <td>-</td>
    </tr>
  </tbody>
</table>


#### 一些常用且需要重点关注的`<video>`事件

<table>
  <thead>
    <tr>
      <th>event</th>
      <th>iOS</th>
      <th>Android</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>play</td>
      <td>只是要播放视频，响应的是video.play()方法，并不代表已经开始播放</td>
      <td>和iOS一样，仅是响应video.play()方法</td>
    </tr>
    <tr>
      <td>durationchange</td>
      <td>会执行一次，一定会获取到视频的duration</td>
      <td>可能会执行多次，只有最后一次才能获取到真实的duration，前面的duration都是0；但低版本Android可能获取到的duration是0或1；（本文提到的低版本Android大部分是4.1以下）</td>
    </tr>
    <tr>
      <td>canplay</td>
      <td>可以认为是视频元素没有问题，可以运行，没有更多含义了，基本用不上</td>
      <td>同iOS</td>
    </tr>
    <tr>
      <td>canplaythrough</td>
      <td>会有明确的缓冲，表示可以流畅播放了；</td>
      <td>没有什么用，视频仍然会卡住，数据可能还没有开始加载；</td>
    </tr>
    <tr>
      <td>playing</td>
      <td>明确表示播放开始了；</td>
      <td>依然没有用，视频可能并没有开始播放；</td>
    </tr>
    <tr>
      <td>progress</td>
      <td>有明确的下载，可以获取到当前的buffer，并且全部下载完毕后不在触发；</td>
      <td>不一定有明确的数据下载，并且全部下载完毕后依然继续触发；</td>
    </tr>
    <tr>
      <td>timeupdate</td>
      <td>会有明确的进度变化，可以获取到currentTime；</td>
      <td>进度不一定变化，currentTime可能总是0，但是第一次有currentTime变化的timeupdate事件一定代表了视频开始播放了；</td>
    </tr>
    <tr>
      <td>error</td>
      <td>iOS中会有明确的错误抛出；</td>
      <td>Android中某些浏览器会莫名其妙的抛出error；</td>
    </tr>
    <tr>
      <td>stalled</td>
      <td>网络状况不佳，导致视频下载中断；</td>
      <td>在没有play之前，也可能会抛出该事件。</td>
    </tr>
  </tbody>
</table>

### 属性差异

<table>
  <thead>
    <tr>
      <th>attributes</th>
      <th>iOS</th>
      <th>android</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>poster<br>封面图片</td>
      <td>支持，但是加载速度明显比在&lt;img&gt;中要慢；</td>
      <td>不一定支持（浏览器厂商的实现标准不统一）；</td>
    </tr>
    <tr>
      <td>preload<br>预加载</td>
      <td>iPhone不支持；</td>
      <td>可能支持；</td>
    </tr>
    <tr>
      <td>autoplay<br>自动播放</td>
      <td>iPhone Safari中不支持，但在webview中可能被开启；iOS开发文档明确说明蜂窝网络下不允许autoplay；</td>
      <td>可能支持；</td>
    </tr>
    <tr>
      <td>loop<br>循环播放</td>
      <td>支持</td>
      <td>可能支持；</td>
    </tr>
    <tr>
      <td>controls<br>控制条</td>
      <td>支持，但是需要开始播放了才显示</td>
      <td>基本都支持显示或者不显示</td>
    </tr>
    <tr>
      <td>width和height</td>
      <td>一定给出明确的属性设置，切不能为0；</td>
      <td>如果不设置，仅仅通过CSS样式去控制视频大小，可能会导致<video>标签失效。</video></td>
    </tr>
  </tbody>
</table>

### 其他怪异bug和不友好表现

<table>
  <thead>
    <tr>
      <th>iOS</th>
      <th>android</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>物理位置覆盖在&lt;video&gt;区域上的元素，click和touch等事件会失效，比如一个&lt;a&gt;链接如果覆盖在&lt;video&gt;上，那么点击后没有任何效果。</td>
      <td>-</td>
    </tr>
    <tr>
      <td>iOS8.0+中，单页面播放视频超过16个，再播放的视频全部MediaError解码异常无法播放。</td>
      <td>-</td>
    </tr>
    <tr>
      <td>iPhone的Safari会弹出一个全屏的播放器来播放视频，iPad则支持内联播放。iOS7+ 如果webview（比如微信）开启了<code class="highlighter-rouge">webview.allowsInlineMediaPlayback = YES;</code>，可以通过设置<code class="highlighter-rouge">webkit-playsinline</code>属性支持内联播放；</td>
      <td>支持内联播放，但某些厂商会用自己的播放器劫持原生的视频播放；</td>
    </tr>
    <tr>
      <td>下载视频时，会先发送一个2字节的请求来获取视频元数据（比如时长），然后再不断的发送分包续传（206）请求来下载视频，抓包显示请求数和请求量至少有一倍的冗余（x2），这个严重的bug在iOS8中有明显的修复，但是分包的206请求仍然会有冗余数据的下载，浪费了流量。</td>
      <td>比iOS的处理方式好，没有第一个2字节请求，没有流量损耗；</td>
    </tr>
    <tr>
      <td>-</td>
      <td>低版本Android（&lt;=4.0.4）中，&lt;video&gt;如果在有相对和决定定位的层中，可能会导致整个页面错位。</td>
    </tr>
    <tr>
      <td>-</td>
      <td>某些浏览器厂商会劫持&lt;video&gt;，用其“自己”的播放器来播放视频，“破坏”了产品本身的播放体验，那么只能case by case的解决了。</td>
    </tr>
    <tr>
      <td>加载视频时没有进度提示，视觉上看不出是播放完了还是卡住了；</td>
      <td>加载视频时，大都会显示一个自带的loading UI（菊花）。</td>
    </tr>
  </tbody>
</table>


## `video`播放产品层主要关注的4个问题

1. 全屏控制；
2. 自动播放；
3. 播放控制；
4. 隐藏播放控件；


### video的全屏控制
#### ios下：
在 iOS 上，APP 都是使用的系统自带的浏览器进行页面渲染，`video`播放视频的效果是统一的，只需要考虑不同的 iOS 版本是否有不一致的地方。在 iOS(11) 上（其他待测试），播放视频默认会弹出一个播放器全屏播放视频，如下效果:

![](https://assets.51wakeup.com/assets/2018/daily-work/markdown/07.jpeg@300w)

播放器上下有的系统默认的控制栏，可以控制视频的播放进度、音量以及暂停或继续播放，播放视频时，视频会 “浮” 在页面上，页面上的所有元素都只能是在视频下面，这种效果显然不是我们想要的。

但好在 iOS 10 Safari 中，video 新增了 `playsinline` 属性，可以使视频内联播放。
iOS 10 之前的版本支持 webkit-playsinline，但是加了这个属性后，在 iOS 9 的上出现只能听到声音不能看到画面的问题，然后再加上这个库 [iphone-inline-video ](https://github.com/bfred-it/iphone-inline-video)一起使用。
最后使用的标签代码

`

     <video id="video" preload="auto" playsinline controls="true" src="//assets.51wakeup.com/assets/2018/pc-homepage/src/assets/home_video.mp4" type="video/mp4"></video>
   
   
   `
#### Android下：
在 Android 上，因为各个软件使用的浏览器渲染引擎不一样，所以视频播放的效果差异也很大，这里主要以微信为主。微信使用的是自带的渲染引擎 TBS，默认的播放效果

![](https://assets.51wakeup.com/assets/2018/daily-work/markdown/08.png@300w)

在播放器的下方也是会有控制栏，视频也会 “浮” 在页面上。而 Android 是不支持 playsinline 属性使视频内联播放的。但是，如果你看过一些腾讯的视频类 HTML5，会发现它们在微信里是可以内联播放的，而这个功能是需要申请加入白名单的。

不过新版的 TBS 内核（>=036849）支持一个叫 同层播放器 的视频播放器，这个不需要申请白名单，只需给 video 设置两个属性 x5-video-player-type="h5" 和 x5-video-player-fullscreen="true"，播放效果如下：

![](https://assets.51wakeup.com/assets/2018/daily-work/markdown/09.jpeg@300w)

加了这个属性后视频上层可以有其他dom元素出现了（非腾讯白名单机制的一种处理措施）。
我们可以通过监测对应的事件知道当前的播放状态

`
       
       //全屏进入
       document.getElementById('video').addEventListener("x5videoenterfullscreen", function(){
		  alert("enter fullscreen")
		});
       
       //全屏退出
		document.getElementById('video').addEventListener("x5videoexitfullscreen", function(){
		  alert("exit fullscreen")
		});

		

`
具体的支持情况：

|   |  TBS < 036849  |  036849 <= TBS < 036900 | 036900 <= TBS  |
|:----------:|:-------------:|:------:|:------:|
| 是否支持同层播放器 |  否 | 是 | 是 |
| 退出全屏播放时触发 | - | x5videoenterfullscreen | x5videoexitfullscreen |
| 进入全屏播放时触发 | - | x5videoexitfullscreen | x5videoenterfullscreen |


如果想在Android下实现小屏幕播放，请使用`x5-playsinline`，效果如下：


![](https://assets.51wakeup.com/assets/2018/daily-work/markdown/10.jpeg@300w)

综上，全屏处理的总结如下：

|            |      默认     | 同层播放(内联到浏览器webview里边) | 小窗 |
|:----------:|:-------------:|:------|:------:|
| >= Ios 10  | 全屏 | playsinline | playsinline + css |
| < Ios 10   | 全屏 | webkit-playsinline + [iphone-inline-video](https://github.com/bfred-it/iphone-inline-video) | webkit-playsinline + [iphone-inline-video](https://github.com/bfred-it/iphone-inline-video) + css |
| Android    | 全屏 | x5-video-player-type="h5"（启用同层H5播放器，就是在视频全屏的时候，div可以呈现在视频层上，也是WeChat安卓版特有的属性。） | x5-playsinline + css |


**webkit-playsinline和playsinline**: 视频播放时局域播放，不脱离文档流 。但是这个属性比较特别， 需要嵌入网页的APP比如WeChat中UIwebview 的allowsInlineMediaPlayback = YES webview.allowsInlineMediaPlayback = YES，才能生效。换句话说，如果APP不设置，你页面中加了这标签也无效，这也就是为什么安卓手机WeChat 播放视频总是全屏，因为APP不支持playsinline，而ISO的WeChat却支持。
这里就要补充下，如果是想做全屏直播或者全屏H5体验的用户，IOS需要设置删除 webkit-playsinline 标签，因为你设置 false 是不支持的 ，安卓则不需要，因为默认全屏。但这时候全屏是有播放控件的，无论你有没有设置control。 做直播的可能用得着播放控件，但是全屏H5是不需要的，那么去除全屏播放时候的控件，需要以下设置：同层播放

**x-webkit-airplay="allow"**: 这个属性应该是使此视频支持ios的AirPlay功能。使用AirPlay可以直接从使用iOS的设备上的不同位置播放视频、音乐还有照片文件，也就是说通过AirPlay功能可以实现影音文件的无线播放，当然前提是播放的终端设备也要支持相应的功能

**x5-video-player-type**: 启用同层H5播放器，就是在视频全屏的时候，div可以呈现在视频层上，也是WeChat安卓版特有的属性。同层播放别名也叫做沉浸式播放，播放的时候看似全屏，但是已经除去了control和微信的导航栏，只留下"X"和"<"两键。目前的同层播放器只在Android（包括微信）上生效，暂时不支持iOS。至于为什么同层播放只对安卓开放，是因为安卓不能像ISO一样局域播放，默认的全屏会使得一些界面操作被阻拦，如果是全屏H5还好，但是做直播的话，诸如弹幕那样的功能就无法实现了，所以这时候同层播放的概念就解决了这个问题。不过在测试的过程中发现，不同版本的IOS和安卓效果略有不同

**x5-video-orientation**: 声明播放器支持的方向，可选值landscape 横屏, portraint竖屏。默认值portraint。无论是直播还是全屏H5一般都是竖屏播放，但是这个属性需要x5-video-player-type开启H5模式
x5­-video­-player­-fullscreen:全屏设置。它又两个属性值，ture和false，true支持全屏播放，false不支持全屏播放。其实，IOS 微信浏览器是Chrome的内核，相关的属性都支持，也是为什么X5同层播放不支持的原因。安卓微信浏览器是X5内核，一些属性标签比如playsinline就不支持，所以始终全屏。

**x5-video-player-fullscreen**：视频播放时将会进入到全屏模式
如果不申明此属性，页面得到视口区域为原始视口大小(视频未播放前)，比如在微信里，会有一个常驻的标题栏，如果不声明此属性，这个标题栏高度不会给页面，播放时会平均分为两块（上下黑块）
注： 声明此属性，需要页面自己重新适配新的视口大小变化。可以通过监听resize 事件来实现

`    <video id="test_video" src="xxx" x5-video-player-type="h5" x5-video-player-fullscreen="true"/> `

需要监听窗口大小变化(resize)实现全屏

`
	
	window.onresize = function(){
		  test_video.style.width = window.innerWidth + "px";
		  test_video.style.height = window.innerHeight + "px";
	}
`
注: 为了让视频真正铺满全屏,可以适当让video的显示区域大于视口区域,这样在显示时在视口外的部截掉后,不会出四周黑边的情况（这是一种全屏的解决方案）

### 自动播放
autoplay的支持依赖内核和网络状况，比如iPhone在蜂窝网络下明确禁用了autoplay；
经过试验，在没有明确的用户操作的情况下，直接通过video.play()也是无法激活播放的；
并且在产品设计上，自动播放也不是一个舒服的用户体验，所以产品设计上尽量避免使用自动播放。
但是，在微信内，可以使用`WeixinJSBridge`来实现自动播放功能：
`

		    if (window.WeixinJSBridge) {
		          WeixinJSBridge.invoke('getNetworkType', {}, (e) => {
		            //播放
		          }, false);
		    } else {
		          document.addEventListener("WeixinJSBridgeReady", () => {
		            WeixinJSBridge.invoke('getNetworkType', {}, (e) => {
		              //播放
		            });
		          }, false);
		    }
        
        `
其他场景还是要引导用户去触发播放。

### 播放控制

对于video或者audio等媒体元素，有一些方法，常用的有`play(),pause();`也有一些事件，如`loadstart,canplay,canplaythrough,ended,timeupdate....`等等。
在移动端有一些坑需要注意，不要轻易使用媒体元素的除`ended,timeupdate`以外event事件，在不同的机子上可能有不同的情况产生，例如：ios下监听canplay和canplaythrough（是否已缓冲了足够的数据可以流畅播放）,当加载时是不会触发的，即使preload="auto"也没用，但在pc的chrome调试器下和android下，是会在加载阶段就触发。ios需要播放后才会触发。总之就是现在的视频标准还不尽完善，有很多坑要注意，要使用前最好自己亲测一遍。就是当第一次播放视频的时候ios端，如果网络慢，视频从开始播到能展现画面会有短暂的黑屏（处理视频源数据的时间），为了避免这个黑屏，可以在视频上加个div浮层（可以一个假的视频第一帧），然后用timeupdate方法监听，视屏播放及有画面的时候再移除浮层。

`     
      
      video.addEventListener('timeupdate',function (){
		    //当视频的currentTime大于0.1时表示黑屏时间已过，已有视频画面，可以移除浮层（.pagestart的div元素）
		    if ( !video.isPlayed && this.currentTime>0.1 ){
		        $('.pagestart').fadeOut(500);
		        video.isPlayed = !0;
		    }
		})

`
还可以结合之前的`WeixinJSBridge`做视频的预加载，因为`preload="auto"`没用
`

		document.addEventListener("WeixinJSBridgeReady", function (){ 
		    video.play();
		    video.pause();
		}, false)

`
### 隐藏播放控件
第一种方案：将视频高度放大到120%，将控制条顶到视窗外面（兼容情况待测试）
具体思路和实现如下：
html中将video标签外包一层
`
    
    <div class="videobox">
        <video id="mainvideo" webkit-playsinline="true" src="http://7xvl2z.com1.z0.glb.clouddn.com/nigg2.mp4"></video>
    </div>
 
`
css:

`

		html,body {
		  padding: 0;
		  margin: 0;
		  width: 100%;
		  height: 100%;
		  -webkit-user-select: none; 
		  user-select: none;
		  overflow: hidden;
		}
		
		.videobox {
		  width: 100%;
		  height: 100%;
		  position: absolute;
		  left: 0;
		  top: 0;
		  overflow: hidden;
		}
		
		video {width: 1px;display: blcok;}
		
		/*
			注：html，body里的overflow：hidden，非常重要，不能省。
			因为微信android的播放器是脱离DOM结构的，也不会响应click、touch等事件。
			如果视频尺寸大了，会产生滚动条，必须在html和body加overflow：hidden，
			在videobox加没用的。
	    */

`
当视频要播放时改变video的宽度（高度会等比缩放，即使自定义高度也是没用的，会被忽略）

`

	  var video = document.querySelector('#mainvideo');
	  var videobox = document.querySelector('.videobox');
	
	  //播放时改变外层包裹的宽度，使video宽度增加，
	  //相应高度也增加了,播放器控件被挤下去，配合overflow：hidden
	  //控件看不见也触摸不到了
	  var setVideoStyle = function (){
	    videobox.style.width = '120%';
	    videobox.style.left = '-10%';
	    video.style.width = '100%';
	  }
  
  
`
当然上面这样写参杂了一些需求逻辑，也可以直接样式表就width:120%,或者更大；这个根据自己的需要来，但基本思路就是将播放器控件顶出视窗之外，达到一种‘去除’、‘消失’的效果。连‘小窗’字样也被顶出去了，用过android或测试过的同学都知道点了小窗后会怎样....（原版的即使去掉了播放器，但小窗字样还在，不能算完全的隐藏播放控件吧）

相应产生的一些问题的解决办法：

1，视频被放大了，画面会被截掉一部分怎么办？

这个可以在视频输出的时候两边和下边留一些留白，即留白可以没用画面黑色底，但又属于视频尺寸的一部分，放视频放大后，将主体画面置满视窗，被挤到外面都是留白的即可。

2，视频播放完毕后会中间自动出现播放控件按钮

如果确实不想在播放完是出现一个按钮，哪怕只有零点几秒，就把视频remove掉，可以使用timeupdate监听视频播放，用video.duration-video.currentTime的差值判断是否快要结束了，在结束前零点几秒提前remove掉。

3，要是不是全屏视频怎么搞？

可以，思路是一样的，将视频控件顶出外层的包裹层，利用overflow：hidden。只是全屏的外层包裹是body而已。

哦了，终于可以搞一个全屏视频伪装的东西了。

第二种方案：使用纯粹的css（兼容情况待测试）
`
		 
		video::-webkit-media-controls {
		    display:none !important;
		}
		video::-webkit-media-controls-overlay-play-button {
		display: none;
		}
`



### 其他需要关注的问题：
#### video的事件那么多，实际可用的只有两个：

`
		
		video.addEventListener('timeupdate', function (e) {
		  console.log(video.currentTime) // 当前播放的进度
		})
		video.addEventListener('ended', function (e) {
		  // 播放结束时触发
		})

`
#### 屏蔽腾讯视频播放之后推荐的视频


`
   
    video.addEventListener('ended', function (e) {
		  currentTime = 0;
	 })


`


## 最佳实践及统一解决方案

[详见基于vue的播放器组件]()


## 第三方视频播放方案汇总

> [chimee播放器](http://chimee.org/)

> [西瓜视频播放器](http://h5player.bytedance.com/)


#### 资料汇总

[视频H5 video标签最佳实践](https://github.com/gnipbao/iblog/issues/11)

[视频H5のVideo标签在微信里的坑和技巧](https://aotu.io/notes/2017/01/11/mobile-video/)

[移动端HTML5`<video>`视频播放优化实践](https://zhaoda.net/2014/10/30/html5-video-optimization/)

[html5--移动端视频video的android兼容，去除播放控件、全屏等](https://segmentfault.com/a/1190000006857675)
