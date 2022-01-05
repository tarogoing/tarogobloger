2019年07月10日 14:16:05 [wcy7916](https://me.csdn.net/wcy7916) 阅读数 1206 文章标签： [小程序](https://so.csdn.net/so/search/s.do?q=%E5%B0%8F%E7%A8%8B%E5%BA%8F&t=blog)[webview](https://so.csdn.net/so/search/s.do?q=webview&t=blog)[web](https://so.csdn.net/so/search/s.do?q=web&t=blog) 更多

分类专栏： [vue](https://blog.csdn.net/wcy7916/article/category/7636259) [小程序](https://blog.csdn.net/wcy7916/article/category/7756109)

版权声明：本文为博主原创文章，遵循[ CC 4.0 BY-SA ](http://creativecommons.org/licenses/by-sa/4.0/)版权协议，转载请附上原文出处链接和本声明。

本文链接：https://blog.csdn.net/wcy7916/article/details/90263039
**前言：**

想把app内的精听课本模块的网页内嵌到新开发的微信小程序里面，之前已经放在了公众号里，现在也要支持在小程序里购买，阅读。既然支持购买，小程序就得有注册登录的功能、支付的功能。支付不能用微信自带的支付，因为不支持，（就算在开发者工具里支持，体验版和正式版也不支持），所以需要调用小程序的支付功能。

**实现：**
小程序中内嵌H5网页是这样的：

<template> <div class="detail container"> //courseUrl 就是网页入口地址 <web-view :src="courseUrl" bindmessage="bindmessage"/> </div>

</template>

要在小程序中实现支付，先在在H5网页的项目中设置支付跳转：（以下是**微信公众号支付和跳转微信小程序支付相结合的完整代码**，@ViewBag.wx_signature这种字段是后台提供的值，是c#语法）

@{ Layout = null; } <!DOCTYPE html> <html> <head> <meta charset="UTF-8"> <meta name="viewport" content="width=device-width,initial-scale=1,user-scalable=0">

<script src="https://res.wx.qq.com/open/js/jweixin-1.3.2.js"></script> <link rel="stylesheet" href="https://res.wx.qq.com/open/libs/weui/1.1.2/weui.min.css" />

<script src="~/Scripts/jquery-1.8.2.min.js"></script>
<title>微信支付</title>

<style> #container { padding: 0 15px; } .page__title { text-align: center; padding: 20px 0; color: gray; font-size: 20px; } </style>

</head> <body ontouchstart>

<script type="text/javascript"> wx.config({ debug: false, appId: '@ViewBag.wx_appid', timestamp: '@ViewBag.wx_timestamp', nonceStr: '@ViewBag.wx_nonceStr', signature: '@ViewBag.wx_signature', jsApiList: [ 'checkJsApi', 'closeWindow', 'scanQRCode', 'chooseWXPay' ] }); function isInApplets(){ wx.miniProgram.getEnv(function (res) {//获取当前环境 if(res.miniprogram){ // true 在微信小程序中 return 1; }else{// false 在微信公众号里 return -1; } }); } wx.ready(function () { if (typeof WeixinJSBridge == "undefined") { if (document.addEventListener) { document.addEventListener('WeixinJSBridgeReady', onBridgeReady, false); } else if (document.attachEvent) { document.attachEvent('WeixinJSBridgeReady', onBridgeReady); document.attachEvent('onWeixinJSBridgeReady', onBridgeReady); } } else { wx.miniProgram.getEnv(function (res) {//获取当前环境 if(res.miniprogram){ // true 在微信小程序中 console.log("在小程序中") wxAppletsPay(); }else{// false 在微信公众号里 onBridgeReady(); console.log("不在小程序中") } }); } }); wx.error(function (res) { // config信息验证失败会执行error函数，如签名过期导致验证失败，具体错误信息可以打开config的debug模式查看，也可以在返回的res参数中查看，对于SPA可以在这里更新签名。 //alert("wx.error" + JSON.stringify(res)); }); function onBridgeReady() { WeixinJSBridge.invoke( 'getBrandWCPayRequest', { "appId": "@ViewBag.orderInfo["appId"]", //公众号名称，由商户传入 "timeStamp": "@ViewBag.orderInfo["timeStamp"]", //时间戳，自1970年以来的秒 "nonceStr": "@ViewBag.orderInfo["nonceStr"]", //随机串 "package": "@ViewBag.orderInfo["package"]", "signType": "@ViewBag.orderInfo["signType"]", //微信签名方式： "paySign": "@ViewBag.orderInfo["paySign"]"//微信签名 }, function (res) { if (res.err_msg == "get_brand_wcpay_request:ok") { window.location = '@ViewBag.returnUrl'; } } ); } //微信小程序支付 function wxAppletsPay(){ //点击微信支付后，调取统一下单接口生成微信小程序支付需要的支付参数 var payParam = { "appId": "@ViewBag.orderInfo["appId"]", //外刊小程序appid "timeStamp": "@ViewBag.orderInfo["timeStamp"]", //时间戳，自1970年以来的秒数 "nonceStr": "@ViewBag.orderInfo["nonceStr"]", //随机串 "package": "@ViewBag.orderInfo["package"]", "signType": "@ViewBag.orderInfo["signType"]", //微信签名方式： "paySign": "@ViewBag.orderInfo["paySign"]"//微信签名 }; //定义path 与小程序的支付页面的路径相对应 var path = '/pages/pay/main?payParam=' + encodeURIComponent(JSON.stringify(payParam)); //通过JSSDK的api跳转到指定的小程序页面 wx.miniProgram.navigateTo({url: path}); } $(function () { //微信支付 $("#btn-open-wx-payment").on("click", function () { try { if(isInApplets()){// true 在微信小程序中 console.log("在小程序中") wxAppletsPay(); }else{// false 在微信公众号里 onBridgeReady(); console.log("不在小程序中") } } catch (e) { console.error(e) onBridgeReady(); console.log("不在小程序中") } }); $("#btn-goback").on("click", function () { javascript: history.go(-1); }); }); </script> <div class="container" id="container"> <div class="page__hd"> <h1 class="page__title">使用微信付款</h1> </div> <div class="weui-panel weui-panel_access"> <div class="weui-panel__bd"> <a href="javascript:;" class="weui-btn weui-btn_primary" id="btn-open-wx-payment">打开微信支付</a> <a href="javascript:;" class="weui-btn weui-btn_default" id="btn-goback">返回上一页</a> </div> </div> </div> </body> </html>

这是以上代码对应的付款前的过渡H5页面
![10140346374.jpeg](../_resources/10140346374.jpeg)
在小程序项目中新建一个支付页：pay.vue

<template> <!-- 该页面只是为了收到webview网页的支付参数 --> <div class="detail container"></div> </template> <script> export default { name: "pay", data() { return {}; }, onLoad: function(options) { var _self = this; //页面加载调取微信支付 _self.requestPayment(options); }, mounted() {}, methods: { //根据 obj 的参数请求wx 支付 requestPayment: function(obj) { var objPay = JSON.parse(decodeURIComponent(obj.payParam)); //调起微信支付 wx.requestPayment({ //相关支付参数 appId: objPay.appId, timeStamp: objPay.timeStamp, nonceStr: objPay.nonceStr, package: objPay.package, signType: objPay.signType, paySign: objPay.paySign, //小程序微信支付成功的回调通知 success: function(res) { console.log("付款成功") //成功之后拉起微信支付 微信支付完成之后跳转到微信自带的支付成功页面 点击页面上的 ‘确定’ 按钮 返回到首页 wx.navigateTo({ url: '/pages/index/main' }) }, //小程序支付失败的回调通知 fail: function(res) { console.log("支付失败"); console.log(res); var pages = getCurrentPages(); var currPage = pages[pages.length - 1]; var prevPage = pages[pages.length - 2]; wx.navigateBack(); } }); } } }; </script> <style scoped> </style>

**tip:**
webview网页在小程序里的调试是，将鼠标放在h5网页上，右击，会出现“调试”选项。
webview网页的域名以及网页内部的其他域名要在微信公众平台=》开发设置=》业务域名中添加，且域名是https://的。否则网页打不开，像酱紫：

![](file:///D:/LiuWenbo/YouDaoNote/tpxip@126.com/68a7e69516b049aab2cf4227c046690b/10141417237.jpeg)

**bug:**
如果支付失败，注意检查openid和package的值是否正确，大部分问题就出在这两个参数值上面。

参考地址：[微信小程序开发之webview组件内网页实现微信原生支付](https://juejin.im/post/5a4f5280f265da3e2b16393c)