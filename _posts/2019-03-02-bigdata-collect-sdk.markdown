---
layout: post
title:  "如何设计一个数据采集的WEBSDK"
date:   2019-03-01 00:18:23 +0700
categories: [bigdata]
---

#### 概述
  上一节我们讲到数据埋点方面的一些工作如何进行，但是由于埋点的成本相对比较高，且容易出
错，我们能不能将部分工作工具化，从而减小在数据采集上面的成本呢？答案是肯定的，那就是我们
运用WEBSDK将上述工作封装起来，并实现相关工作的自动化。
  
#### WEBSDK我们怎么设计  
  那我们具体怎么设计我们的WEBSDK呢？实际上我们上节讲到数据埋点几个关键的点，第一个是
数据采集的时机，数据的协议及数据采集的字段等，那这些我们集成到我们WEBSDK?  
##### 第一步：我们确认我们SDK需要集成那些事件  
  我们确认我们SDK集成那些事件，最为主要的就是了解我们需要采集那些数据，通常对于我们
一个网站或者一个Ｍ站我们需要采集有事件大致上我们可以分解为如下几个重要的事件:  
  
###### 用户浏览类事件  
  用户浏览类事件即用户打一个HTML页面，页面加载完成。表示用户进入到这个页面，通常
我们通过这种事件的数据计算出有多少访问，产生了多少ＰＶ，以及用户的浏览路径等，进行
用户路径的分析等等。
    
###### 用户点击类事件  
  用户点击类事件，通常我们是用户通过触发一个Button或者一个LINK产生一个Click事件
通常我们用于收集一些数据用于观察用于对该类控件的点击情况。CLick大部分为一个用户主动事件
一般结合PV事件或者控件曝光事件计算点击轮化率等指标或者用户改进在线的产品，点击数据是非常
重的数据源因为具有明确的主动意图，所以应用范围相当广。
  
###### 用户控件曝光类事件  
  控件曝光类事件其实是浏览类事件一个分解体，在WEB2.0时代，浏览形式不仅仅只有之前的页面形式
，现在出现更多的展示形态如Tab，如曝布流等或者弹窗。我们不通过ＰＶ来记录该类事件，因为PV表示页面浏览
而我们使用控件曝光来记录这些产品对于用户的展示情况。
  
###### 用户自定义事件  
　　用户自定义事件其实是在上述特定事件无法满足的基础上，为了保证SDK灵活性，而扩展的一类事件
主要特性是不对用户上报的数据及时间不做特定的定义，开发人员可以在任意地点进行调用。以保证SDK灵活性与扩展性。
  
##### 第二步：我们将这些事件分解为数据埋点  
  
###### 用户浏览类事件  
  
上报点：页面加载完成时上报，上报完成是指ＷＥＢ页面的ＤＯＭ结构加载完成，采用实时上报
上报范围：该活动涉及的所有页面  
上报协议：采用HTTP GET方式  
上报数据地址:http://www.viewc.org/logs  
```$xslt
一个字段的定义包含三个部分　　
字段名：具体的名称一般使用简化的字段　　
类型：该字段使用什么类型的数据  
说明：对于该字段的描述用于让别人对该字段取什么的值有可执行理解  

EG:

event   String  固定值"PV"  
cookie  String  获取网站Cookie UUID,用于标识唯一的用户，如果该值不存在时，则生一个UUID的Cookie,种在viewc.org下面，有效期999年　　
url     String  获取当前浏览器地址，用于标识当前页面  
ref     String  获取当前页面的来源页面，如果没有则为空  
lob     String  扩展字段用于业务扩展格式采用 key=value,多个值以＆分隔，并进行urlencode
```
  
###### 用户点击类事件  
  
上报点：用户点击时上报对应具体的Click事件  
上报范围：根据产品业务范围而定  
上报协议：采用HTTP GET方式  
上报数据地址:http://www.viewc.org/logs  
```$xslt
EG:

event   String  固定值"Click"  
cookie  String  获取网站Cookie UUID,用于标识唯一的用户，如果该值不存在时，则生一个UUID的Cookie,种在viewc.org下面，有效期999年　　
url     String  获取当前浏览器地址，用于标识当前页面  
label   String  控件名称  
lob     String  扩展字段用于业务扩展格式采用 key=value,多个值以＆分隔，并进行urlencode
```
  
###### 用户控件曝光类事件  
  
上报点：用户触发控件曝光时上报，具体对应Show事件或者数据加载完成Success事件  
上报范围：根据产品业务范围而定  
上报协议：采用HTTP GET方式  
上报数据地址:http://www.viewc.org/logs  
```$xslt
EG:

event   String  固定值"show"  
cookie  String  获取网站Cookie UUID,用于标识唯一的用户，如果该值不存在时，则生一个UUID的Cookie,种在viewc.org下面，有效期999年　　
url     String  获取当前浏览器地址，用于标识当前页面  
label   String  控件名称  
lob     String  扩展字段用于业务扩展格式采用 key=value,多个值以＆分隔，并进行urlencode
```
  
###### 用户自定义事件  
  
上报点：不做具体的定义，以具体产品代码触发而定  
上报范围：根据产品业务范围而定  
上报协议：采用HTTP GET方式  
上报数据地址:http://www.viewc.org/logs  
```$xslt
EG:

event   String  固定值"custom"  
cookie  String  获取网站Cookie UUID,用于标识唯一的用户，如果该值不存在时，则生一个UUID的Cookie,种在viewc.org下面，有效期999年　　
lob     String  扩展字段用于业务扩展格式采用 key=value,多个值以＆分隔，并进行urlencode
```
  
##### 第三步：我们找到对应的实现方法  
　　根据上面分解的具体的埋点文档，我们进行SDK具体实现，同样我们需要进行一些约定，从而
形成具体的规范比如引入方式等。我们具体实现抽象成如下几部分:  
  
###### 引入方式  

```$xslt
以下是设计一种引入方式，具体根据公司前端框架而定

<script>
(function(G,D,s,c,p){
c={
site_verion:window._M_SITE_VERSION,
server_url:"logs",
sdk:"/sdk-web-sdk.js"
};
function load(t){
  s=D.createElmenet("script");
  s.src=c.sdk;
  t=D.getElementByTagName("script");
  t=t[t.length-1];
  t.parentNode.insertBefore(s,t);
}
})(this.document)
<script>
```  
  
###### 各事件接入规范  
  浏览类自动上报声明方式  
```$xslt
以下是浏览类自动上报一种声明方式，SDK在过滤页面DOM结构时，获取Ｂody相关属性，结合页面相关数据进行上报
<body td-stat-page="demopage1" td-stat-lob="k1=v1&k2=v2"></body>

如下是单页面应用也可以将相关属性定义在DIV中　如
<div class="td-channel" td-stat-page="demopage1" td-stat-lob="k1=v1&k2=v2"></div>
```  
  
  点击类自动上报声明方式  
```$xslt
通常我们并不会收集所有元素的click，需要收集的控件，我们通常通过声明的方式进行定义如：
<div td-stat-mod="index_slider"></div>
表示该模块产生的点击进行收集，当然也可以放在具体的控件上面
<span td-stat-click=""></span>
```  
  同理其它类型的事件也可以采用这种声明的方式进行，从而形成一个WEBSDK,加快数据采集的效率，而自定义的方法则实际是为前端应用提供一个可调用的接口，该
接口不限制定义的参与，由客户端任意进行拼接。

#### WEBSDK如何与其它终端SDK配合使用  
  我们知道WEBSDK主要应用于网页端上面，随着移动时代的到来。我们会发现很多APP里面有
很多的Ｈ５页面，我们Ｈ５页面可以使用WEBSDK实现上报。那当我们Ｈ５被在具ＡＰＰ里面打开时
我们怎么和移动端的数据上报进行打通。这是非常关键的一步，我们知道在网页端我们通过Cookie跟踪用户
移动端可能过用户的imei或者openudid/Idfa等跟踪用户将用户的行为进行串联，但是当用户同时使用Ｍ站与ＡＰＰ里我们怎么将用户的信息进行关联实现数据
价值的最大化。通常我们使用jsbridge，我们知道ＡＰＰ通过webview组件实现页面的加载，本质上就是一个简单的版的浏览器。我们通过将移动端的ＳＤＫ中的
上报方法通过webview注入到webview中，websdk通过ua 字段进行判断以识别该页面正在ＡＰＰ内打开，从而在ＡＰＰ内打开时通过调用注入的方法实现将
网页产生的数据上报通过jsbridge通过app代码走移动sdk上报。具体实现原理后续会讲到。
