[toc]
### 何谓局部刷新

局部刷新是相对全局刷新而言的。在ajax出现之前，浏览器跟服务器交互一般是通过form表单的形式，或post或get，不管采用哪种方式，将数据提交之后，原页面被替换为新页面。局部刷新是在页面发送一个请求，完成请求只更新页面中的局部内容，原页面依然存在。

通用的做法有两种：ajax和jsonp，接下来将详细介绍着两种方式。
### AJAX基本定义

AJAX即“Asynchronous JavaScript and XML”（异步的JavaScript与XML技术），指的是一套综合了多项技术的浏览器端网页开发技术。Ajax的概念由杰西·詹姆士·贾瑞特所提出。

传统的Web应用允许用户端填写表单（form），当送出表单时就向网页服务器发送一个请求。服务器接收并处理传来的表单，然后送回一个新的网页，但这个做法浪费了许多带宽，因为在前后两个页面中的大部分HTML码往往是相同的。由于每次应用的沟通都需要向服务器发送请求，应用的回应时间依赖于服务器的回应时间。这导致了用户界面的回应比本机应用慢得多。

与此不同，AJAX应用可以仅向服务器发送并取回必须的数据，并在客户端采用JavaScript处理来自服务器的回应。因为在服务器和浏览器之间交换的数据大量减少（大约只有原来的5%）[来源请求],服务器回应更快了。同时，很多的处理工作可以在发出请求的客户端机器上完成，因此Web服务器的负荷也减少了。——维基百科

### XMLHttpRequest
ajax主要是通过XMLHttpRequest对象来控制，XMLHttpRequest定义如下：

XMLHttpRequest 是一个 JavaScript 对象，它最初由微软设计,随后被 Mozilla、Apple 和 Google采纳. 如今,该对象已经被 W3C组织标准化. 通过它,你可以很容易的取回一个URL上的资源数据. 尽管名字里有XML, 但 XMLHttpRequest 可以取回所有类型的数据资源，并不局限于XML。 而且除了HTTP ,它还支持file 和 ftp 协议.

创建一个 XMLHttpRequest 实例, 可以使用如下语句:
var req = new XMLHttpRequest();
方法概述
void abort();
DOMString getAllResponseHeaders();
DOMString? getResponseHeader(DOMString header);

void open(DOMString method, DOMString url, optional boolean async, optional DOMString? user, optional DOMString? password);

void overrideMimeType(DOMString mime);
void send();
void send(ArrayBuffer data);
void send(Blob data);
void send(Document data);
void send(DOMString? data);
void send(FormData data);
void setRequestHeader(DOMString header, DOMString value);
非标准方法

[noscript] void init(in nsIPrincipal principal, in nsIScriptContext scriptContext, in nsPIDOMWindow ownerWindow);

[noscript] void openRequest(in AUTF8String method, in AUTF8String url, in boolean async, in AString user, in AString password);

void sendAsBinary(in DOMString body);
方法
abort() 如果请求已经被发送,则立刻中止请求.

DOMString getAllResponseHeaders(); 返回所有响应头信息(响应头名和值), 如果响应头还没接受,则返回null. 注意: For multipart requests, this returns the headers from the current part of the request, not from the original channel.

DOMString? getResponseHeader(DOMString header); 返回指定的响应头的值, 如果响应头还没被接受,或该响应头不存在,则返回null.

open() 初始化一个请求. 该方法用于JavaScript代码中;如果是本地代码, 使用 openRequest()方法代替.

_注意: Calling this method an already active request (one for which open()or openRequest()has already been called) is the equivalent of calling abort().

void open(
    DOMString method,
    DOMString url,
    optional boolean async,
    optional DOMString user,
    optional DOMString password
);_
参数
method
请求所使用的HTTP方法; 例如 “GET”, “POST”, “PUT”, “DELETE”等. 如果下个参数是非HTTP(S)的URL,则忽略该参数.
url
该请求所要访问的URL
async

一个可选的布尔值参数，默认为true,意味着是否执行异步操作，如果值为false,则send()方法不会返回任何东西，直到接受到了服务器的返回数据。如果为值为true，一个对开发者透明的通知会发送到相关的事件监听者。这个值必须是true,如果multipart 属性是true，否则将会出现一个意外。

user
用户名,可选参数,为授权使用;默认参数为空string.
password
_密码,可选参数,为授权使用;默认参数为空string.
overrideMimeType()_

重写由服务器返回的MIME type。这个可用于, 例如，强制把一个响应流当作“text/xml”来处理和解析,即使服务器没有指明数据是这个类型。注意，这个方法必须在send()之前被调用。

void overrideMimeType(DOMString mimetype);
send()发送请求. 如果该请求是异步模式(默认),该方法会立刻返回. 相反,如果请求是同步模式,则直到请求的响应完全接受以后,该方法才会返回.
注意: 所有相关的事件绑定必须在调用send()方法之前进行.
void send();
void send(ArrayBuffer data);
void send(Blob data);
void send(Document data);
void send(DOMString? data);
void send(FormData data);
*注意*

If the data is a Document, it is serialized before being sent. When sending a Document, versions of Firefox prior to version 3 always send the request using UTF-8 encoding; Firefox 3 properly sends the document using the encoding specified by body.xmlEncoding, or UTF-8 if no encoding is specified.

If it’s an nsIInputStream, it must be compatible with nsIUploadChannel’s setUploadStream()method. In that case, a Content-Length header is added to the request, with its value obtained using nsIInputStream’s available()method. Any headers included at the top of the stream are treated as part of the message body. The stream’s MIMEtype should be specified by setting the Content-Type header using the setRequestHeader()method prior to calling send().

setRequestHeader()

给指定的HTTP请求头赋值.在这之前,你必须确认已经调用 open() 方法打开了一个url.
void setRequestHeader(
    DOMString header,
    DOMString value
);

**参数**
header 将要被赋值的请求头名称.
value 给指定的请求头赋的值.

### ajax样例
var httpRequest = null;
// Old compatibility code, no longer needed.
if (window.XMLHttpRequest) { // Mozilla, Safari, IE7+ ...
httpRequest = new XMLHttpRequest();
} else if (window.ActiveXObject) { // IE 6 and older
httpRequest = new ActiveXObject("Microsoft.XMLHTTP");
}
httpRequest.onreadystatechange = function(){
// process the server response
};

httpRequest.open('GET', '[http://www.exam](http://www.exam/)ßß[ple.org/some.file](http://ple.org/some.file)', true);

httpRequest.send(null);

### 代码分析：
1. 生成一个httpRequest实例，这里需要考虑浏览器的兼容性；
2. 注册onreadstatechange回调。
3. 指定方法（post|get|head等）、url、同步异步方式，并打开一个连接；
4. 发送数据，如果是get，send(null)；如果是post，send(string)，发送一个encode的数据；如果是文件，直接发送一个FormData；

### Event
Ajax的事件包括三个部分，通用、上传以及下载。

### 特殊情况——文件上传

这里单独把文件上传拿出来，是因为它确实ajax方式的特殊应用，传输方式还是原来的方式，内容却不一样，接下来我讲分别使用原生态的方式以及jQuery的方式上传文件。并且由于篇幅限制，后端实现再具文描述。

### 原理介绍
文件上传实际上就是将一些二进制数据通过post的形式上传至后端。上传时必须按照指定的协议，否则后端将无法识别，协议内容如下：
1. post方式提交；
2. 指定post的content-type为mutipart/form-data;
3. 因为指定了content-type，所有内容均按照二进制的形式进行传播。
通过form表单直接上传文件，只需要直接设置enctype属性为multipart/form-data即可。这里我们着重描述通过ajax上传文件。
Ajax上文件使用FormData发送二进制文件，FormData定义如下：

XMLHttpRequest Level 2添加了一个新的接口FormData.利用FormData对象,我们可以通过JavaScript用一些键值对来模拟一系列表单控件,我们还可以使用XMLHttpRequest的send()方法来异步的提交这个”表单”.比起普通的ajax,使用FormData的最大优点就是我们可以异步上传一个二进制文件.

### 构造函数：
new FormData (optional HTMLFormElement form)
参数form： (可选) 一个HTML表单元素,可以包含任何形式的表单控件,包括文件输入框.
其最重要的方法当属append，通过它能够为formData实例添加一系列上传的内容，append定义如下：
void append(DOMString name, Blob value, optional DOMString filename);
void append(DOMString name, DOMString value);
1 name : 字段名称.
2 value : 字段值.可以是,或者一个字符串,如果全都不是,则该值会被自动转换成字符串.

filename: (可选) 指定文件的文件名,当value参数被指定为一个Blob对象或者一个File对象时,该文件名会被发送到服务器上,对于Blob对象来说,这个值默认为”blob”.

### 前端实现
原生态实现
var oMyForm = new FormData();

oMyForm.append("username", "Groucho");
oMyForm.append("accountnum", 123456); // 数字123456被立即转换成字符串"123456"

// fileInputElement中已经包含了用户所选择的文件
oMyForm.append("userfile", fileInputElement.files[0]);

var oFileBody = '<a id="a"><b id="b">hey!</b></a>'; // Blob对象包含的文件内容
var oBlob = new Blob([oFileBody], { type: "text/xml"});

oMyForm.append("webmasterfile", oBlob);

var oReq = new XMLHttpRequest();

oReq.open("POST", "[http://foo.com/submitform.php");](http://foo.com/submitform.php)

oReq.send(oMyForm);

### 代码分析：
1. 创建一个formData实例；
2. 添加string、number、file等内容；
3. 使用XMLHttpRequest实例的send方法提交。

### jQuery上传
HTML页面代码

var fd = new FormData();
// fileInputElement中已经包含了用户所选择的文件
fd.append("userfile", fileInputElement.files[0]);
fd.append("CustomField", "This is some extra data");
$.ajax({
url: "stash.php",
type: "POST",
data: fd,
processData: false, // 告诉jQuery不要去处理发送的数据
contentType: false // 告诉jQuery不要去设置Content-Type请求头
});

### 代码分析
1. 新建一个FormData实例；
2. 加入String/Number/File等内容；

3. 使用jQuery的ajax方法提交文件，注意配置processData和contentType，否则jQuery会根据data自动生成contentType。

###JSONP

jsonp应该是对ajax的一个补充。由于安全考虑，在一般情况下，ajax只支持同源策略（ajax也可以异源，另文讨论），因此需要一个能够通过跨域获取数据的协议——jsonp。

比如，搜狗搜索结果页[http://www.sogou.com](http://www.sogou.com/)，想获取百度[http://www.baidu.com](http://www.baidu.com/)的数据，如果通过ajax肯定是不行的，浏览器会提示Cross Origin。如果通过jsonp的形式就完全可以，前提是具体的接口支持jsonp的协议。

原理介绍

jsonp其实就是在页面中插入一个script标签，并注册一个全局函数，当script标签加载成功时会调用这个全局函数。这个全局函数我们称之为jsonp回调，我们通过自定定义jsonp的回调来完成对数据的处理。一般的，jsonp必须满足以下条件：

1. 因为使用script标签，返回的数据必须满足js语法；
2. 数据格式必须满足functionName([data]);
3. jsonp一般都有数据交换，因此data参数都是有效的数据。

基本实现
var script = document.createElement("script");
var callbackName = jsonp + ( + new Date());
var url = "abc.php?key=123";
script.type="text/javascript";
window[callbackName] = function(data){
// callback define here.
};
script.src = [url, "&callback=", callbackName].join("");
document.getElementsByTagName("head")[0].appendChild(script);

jQuery
$.ajax({
url: "abc.php?key=123",
dataType: "jsonp",
success: function(data){
// callback define here
},
error: function(){
// error callback define here.
}
});

优劣势
对比ajax，jsonp具有以下优点：
1. 跨域支持性比ajax要好；
2. 浏览器兼容性比ajax要好，不同浏览器浏览器对ajax的支持各异，但是对jsonp支持基本都是一致，基本不用考虑浏览器兼容性；
但是，缺点也不少：
1. 只支持get请求；
2. 因为使用script标签，事件回调相对较少；如ajax有onProcess、onload等事件；
3. 目标url的站点必须支持jsonp协议，必须满足functionName(data)的格式；
4. 因为jsonp格式只要引入script标签均可调用，安全性较ajax要差很多；谁家都可用引用该script标签，保证安全的难度太大。

参考文献
XMLHttpRequest https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest

Get start with AJAX: https://developer.mozilla.org/zh-CN/docs/AJAX/Getting_Started

文件上传： http://www.faqs.org/rfcs/rfc1867.html
FormData： https://developer.mozilla.org/zh-CN/docs/Web/API/FormData

FormData文件上传：https://developer.mozilla.org/zh-CN/docs/Web/Guide/Using_FormData_Objects

HTML中使用Ajax进行局部刷新页面

使用Ajax进行用户名动态校验，局部刷新页面1.在HTML页面中使用js脚本将请求数据发送给后台servlet 由按钮触发事件 查询 由js脚本对将数据发送到后台 var req = new ...

---------------------
作者：shushanfx
来源：CSDN
原文：https://blog.csdn.net/shushanfx/article/details/51817565?utm_source=copy
版权声明：本文为博主原创文章，转载请附上博文链接！