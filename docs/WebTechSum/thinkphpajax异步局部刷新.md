摘要：ThinkPHP是一个小型网站很常用的低端框架，但是不专业的文档和编码导致使用者很容易只知其表不知其里。这里仅就官方文档中未曾提及的在thinkphp中使用jquery实现ajax异步交互略作总结。

环境：ThinkPHP3.2.3，jQuery
阅读目录：
正文：

在一般的网站中，都需要用到jquery或者其他框架（比如angular）来处理前后端数据交互。在thinkphp在后台也内置了一些函数用于数据交互（比如ajaxReturn()）。本文的目的是打通jquery ajax到thinkphp的前后端数据交互过程。

前言：
一、thinkphp关于ajax的介绍
**1.1 ajaxReturn：**

\Think\Controller类提供了ajaxReturn方法用于AJAX调用后**返回**数据给**客户端（视图、模板、js等）**。并且支持**JSON**、JSONP、XML和EVAL四种方式给客户端接受数据（默认JSON）。（链接：http://document.thinkphp.cn/manual_3_2.html#ajax_return）

配置方式：convention.php中定义了默认编码类型为DEFAULT_AJAX_RETURN =>  'JSON',
分析：ajaxReturn()调用了**json_encode()**将数值转换成json数据存储格式，常用的数值是数组。

注意：The value being encoded can be any type except a resource(资源文件).All string data must be UTF-8 encoded.（链接：http://php.net/manual/en/function.json-encode.php）

举例：
$data['status']  = 1;
$data['content'] = 'content';
**$this->ajaxReturn($data);**
**1.2 请求类型：**
系统内置了一些常量用于判断请求类型，比如：
常量 说明
IS_GET 判断是否是GET方式提交
IS_POST 判断是否是POST方式提交
IS_PUT 判断是否是PUT方式提交
IS_DELETE 判断是否是DELETE方式提交
IS_AJAX 判断是否是AJAX提交
REQUEST_METHOD 当前提交类型

目的：一方面可以针对请求类型作出不同的逻辑处理，另外一方面可以过滤不安全的请求。 （链接：http://document.thinkphp.cn/manual_3_2.html#request_method）

使用方法：
class UserController extends Controller{
     public function update(){
         if (IS_POST){
             $User = M('User');
             $User->create();
             $User->save();
             $this->success('保存完成');
         }else{
             $this->error('非法请求');
         }
     }
}
**1.3 跳转和重定向：**

功能比较鸡肋，在ajax异步交互局部刷新中，不需要有文字提示的跳转。（链接：http://document.thinkphp.cn/manual_3_2.html#page_jump_redirect）

二、jQuery Ajax的介绍：
**2.1 官网关于jQuery.ajax()的介绍**：
jQuery.ajax() 方法用于执行 AJAX（异步 HTTP）请求。（链接：http://www.jquery123.com/jQuery.ajax/）
语法：$.ajax({name:value, name:value, ... })，该参数规定 AJAX 请求的一个或多个名称/值对。
常见参数：
**type (默认: 'GET')**
类型: String

请求方式 ("POST" 或 "GET")， 默认为 "GET"。注意:其它 HTTP 请求方法，如 PUT 和 DELETE 也可以使用，但仅部分浏览器支持。

**url (默认: 当前页面地址)**
类型: String
发送请求的地址。
**async (默认: true)（1.8版本已弃用）**
类型: Boolean
默认设置下，所有请求均为异步请求（也就是说这是默认设置为 true ）。如果需要发送同步请求，请将此选项设置为 false 。
**data**
类型: Object, String

发送到服务器的数据。将自动转换为请求字符串格式。GET 请求中将附加在 URL 后面。查看 processData 选项说明，以禁止此自动转换。对象必须为"{键:值}"格式。如果这个参数是一个数组，jQuery会按照traditional 参数的值， 将自动转化为一个同名的多值查询字符串(查看下面的说明)。注：如 {foo:["bar1", "bar2"]} 转换为 '&foo=bar1&foo=bar2'。

**dataType (默认: Intelligent Guess (xml, json, script, or html))**
类型: String

预期服务器返回的数据类型。如果不指定，jQuery 将自动根据 HTTP 包 MIME 信息来智能判断，比如XML MIME类型就被识别为XML。在1.4中，JSON就会生成一个JavaScript对象，而script则会执行这个脚本。随后服务器端返回的数据会根据这个值解析后，传递给回调函数。举例:

"json": 把响应的结果当作 JSON 执行，并返回一个JavaScript对象。在 jQuery 1.4 中，JSON 格式的数据以严格的方式解析，如果格式有错误，jQuery都会被拒绝并抛出一个解析错误的异常。（见[json.org](http://json.org/)的更多信息，正确的JSON格式。）

**error**
类型: Function( jqXHR jqXHR, String textStatus, String errorThrown )

请求失败时调用此函数。有以下三个参数：jqXHR (在 jQuery 1.4.x前为XMLHttpRequest) 对象、描述发生错误类型的一个字符串 和 捕获的异常对象。如果发生了错误，错误信息（第二个参数）除了得到null之外，还可能是"timeout", "error", "abort" ，和 "parsererror"。 当一个HTTP错误发生时，errorThrown 接收HTTP状态的文本部分，比如： "Not Found"（没有找到） 或者 "Internal Server Error."（服务器内部错误）。 从jQuery 1.5开始, 在error设置可以接受函数组成的数组。每个函数将被依次调用。 注意：此处理程序在跨域脚本和JSONP形式的请求时不被调用。这是一个 Ajax Event。

**success**
类型: Function( Object data, String textStatus, jqXHR jqXHR )

请求成功后的回调函数。这个函数传递3个参数：从服务器返回的数据，并根据dataType参数进行处理后的数据，一个描述状态的字符串;还有 jqXHR（在jQuery 1.4.x前为XMLHttpRequest） 对象 。在jQuery 1.5， 成功设置可以接受一个函数数组。每个函数将被依次调用。这是一个 Ajax Event

**其他jQuery-ajax-settings**，详见：http://www.jquery123.com/#jQuery-ajax-settings

举例：
在js中把id作为数据发送到服务器， 保存一些数据到服务器上， 一旦请求完成就通知用户。  如果请求失败，则提醒用户。
var menuId = $("ul.nav").first().attr("id");
var request = $.ajax({
  url: "script.php",
  type: "POST",
  data: {id : menuId},
  dataType: "html"
});
request.done(function(msg) {
  $("#log").html( msg );
});
request.fail(function(jqXHR, textStatus) {
  alert( "Request failed: " + textStatus );
});
注意：此处也可以在ajax()中使用success和error参数判断请求结果成功还是失败，并执行下一步操作。

**2.2 js与json**
2.2.1 json是什么：
JSON：JavaScript 对象表示法（JavaScript Object Notation）。是独立于语言之外的存储和交换文本信息的语法。
2.2.2 json和ajax的关系？

在上面关于jquery.ajax的介绍中提到了，json可以作为一个ajax函数的dataType，这样数据就会通过json语法传输了。（链接：http://www.cnblogs.com/haitao-fan/p/3908973.html）

在jquery的ajax函数中，只能传入3种类型的数据：（链接：http://www.cnblogs.com/haitao-fan/p/3908973.html）

>1.json字符串："uname=alice&mobileIpt=110&birthday=1983-05-12"
>2.json对象：{uanme:'vic',mobileIpt:'110',birthday:'2013-11-11'}
>3.json数组：
[
    {"name":"uname","value":"alice"},
    {"name":"mobileIpt","value":"110"},
    {"name":"birthday","value":"2012-11-11"}
]
2.2.3 json的编解码和数据转换：

2.2.2中提到的json对象是更方便与js数组、js字符串或php数组、php字符串进行数据转化的json类型。下面以json对象为例讲解一下json对象与js和php的数据类型转化。

json对象转化成数组：
<script type="text/javascript">

        var jsonStr = '[{"id":"01","open":false,"pId":"0","name":"A部门"},{"id":"01","open":false,"pId":"0","name":"A部门"},{"id":"011","open":false,"pId":"01","name":"A部门"},{"id":"03","open":false,"pId":"0","name":"A部门"},{"id":"04","open":false,"pId":"0","name":"A部门"}, {"id":"05","open":false,"pId":"0","name":"A部门"}, {"id":"06","open":false,"pId":"0","name":"A部门"}]';

      //  var jsonObj = $.parseJSON(jsonStr);
      var jsonObj =  JSON.parse(jsonStr)
        console.log(jsonObj)
     var jsonStr1 = JSON.stringify(jsonObj)
     console.log(jsonStr1+"jsonStr1")
     var jsonArr = [];
     for(var i =0 ;i < jsonObj.length;i++){
            jsonArr[i] = jsonObj[i];
     }
     console.log(typeof(jsonArr))
    </script>
想要将表单数据提交到后台，需要先从表单获取数据/数据集：

serialize和serializeArray的区别是serialize()获取到序列化的表单值字符串，serializeArray()以数组形式输出序列化表单值。举例：

var serialize_string=$('#form').serialize();
得到：a=1&b=2&c=3&d=4&e=5
var serialize_string_array=$('#form').serializeArray();
得到：
[
  {name: 'firstname', value: 'Hello'},
  {name: 'lastname', value: 'World'},
  {name: 'alias'}, // 值为空
]

相对来说，serializeArray()和最终想要得到的json数组更加相似。只不过需要将包含多个name-value形式json对象的json数组改写成'first_name':'Hello'形式的json对象。

下面有两种转换方法：http://blog.jdk5.com/zh/convert-form-data-to-javascript-object-with-jquery/

这里使用第二种方法举例，可以起名为change_serialize_to_json()：
var data = {};
$("form").serializeArray().map(function(x){
if (data[x.name] !== undefined) {
        if (!data[x.name].push) {
            data[x.name] = [data[x.name]];
        }
        data[x.name].push(x.value || '');
    } else {
        data[x.name] = x.value || '';
    }
});
输出：{"input1":"","textarea":"234","select":"1"}

完整流程：

var serialize=$('#form').serialize()结果（得到一个字符串，用serializeArray()更好，其中中文都会转换成utf8）：

serial_number=SN2&result=%E9%9D%9E%E6%B3%95
var serialize_array=$('#form').serializeArray()结果（结果是json对象数组）：
Array [ Object, Object ]
var data=change_serialize_to_json(serialize_array)的结果是（以第二种转换方法为例，结果是json对象）：
Object {serial_number: "SN2", result: "非法" }
var json_data=JSON.stringify(data)（结果是json字符串）：
{"serial_number":"SN2","result":"非法"}

在js端将表单数据转化为json形式的其他函数：
将json字符串转换为json对象：
eval("(" + status_process+ ")")；
json字符串转化成json对象：
// jquery的方法
var jsonObj = $.parseJSON(jsonStr)
//js 的方法
var jsonObj =  JSON.parse(jsonStr)
json对象转化成json字符串:
//js方法
var jsonStr1 = JSON.stringify(jsonObj)
JSON.parse()用于从一个字符串中解析出json对象。JSON.stringify()相反，用于从一个对象解析出字符串。
（链接：http://blog.csdn.net/wangxiaohu__/article/details/7254598）

str_replace() 函数用于替换掉字符串中的特定字符，比如替换掉数据转换后多余的空格、'/nbsp'等

注意：serialize和serializeArray()函数在处理checkbox时存在无法获取未勾选项的bug，需要自己编写函数改写原函数，举例：
（链接：http://www.cnblogs.com/tangge/p/6554891.html）
//value赋值为off是因为正常的serializeArray()获取到的勾选的checkbox值为on。
$.fn.mySerializeArray = function () {
    var a = this.serializeArray();
    var $radio = $('input[type=radio],input[type=checkbox]', this);
    var temp = {};
    $.each($radio, function () {
        if (!temp.hasOwnProperty(this.name))
        {
            if ($("input[name='" + this.name + "']:checked").length == 0)
            {
                temp[this.name] = "";
                a.push({name: this.name, value: "off"});
            }
        }
    });
    return a;
};

三、使用js操作DOM实现局部刷新：
实现局部刷新的途径：
1、假设页面有查询form和结果table。

2、点击查询form的提交，触 发js自定义的submit事件，在submit函数中对获取的表单数据检测后如果符合要求就传递给控制器，控制器从数据库获取结果数组后返回给ajax的success。对返回给ajax的结果数组，可以创建一个refresh()函数，或直接在success中用jQuery（或其他js）操纵结果table（DOM），比如删除tbody节点下的所有内容，并将结果数组格式化后添加到tbody下面。

举例：
//1、php中的form表单

<div class="modal fade hide" id="add_engineer_modal" tabindex="-1" role="dialog">

......
<form id="add_engineer_modal_form" class="form-horizontal">
<fieldset>
......

<button type="button" class="btn btn-primary" id="add_engineer_modal_submit" onclick="add_engineer_modal_submit()" >提交更改</button>

......
</fieldset>
</form>
</div>

//2、js校验表单并发起ajax
function add_engineer_modal_check_value() {
    //以edit_modal_check_value()为模板

    var serialize_array_object = $("#add_engineer_modal_form").mySerializeArray();

    var data = change_serialize_to_json(serialize_array_object);
    var check_results = [];
    check_results['result'] = [];//保存错误信息
    check_results['data'] = data;//保存input和select对象
    //check_employee_number是自定义判断员工号函数。
    if (check_employee_number(data['employee_number']) == false)
    {
        check_results['result'].push("请输入有效的员工号（可选）");
    }
    return check_results;
}

function add_engineer_modal_submit() {
    var check_results = add_engineer_modal_check_value();
    if (check_results['result'].length == 0)
    {

        var json_data = JSON.stringify(check_results['data']);   //JSON.stringify() 方法将一个JavaScript值转换为一个JSON字符串（ajax要求json对象或json字符串才能传输）

        $.ajax({
            type: 'POST',
            url: add_engineer_url, //在php中全局定义url，方便使用thinkphp的U方法

            data: {"json_data": json_data},            //ajax要求json对象或json字符串才能传输,json_data只是json字符串而已

            dataType: "json",
            success: function (data) {
                console.log("数据交互成功");
            },
            error: function (data) {
                console.log("数据交互失败");
            }
        });
    }
    else
    {
        //弹出错误提示alert
    }
    return 0;
}

3、控制器返回数组给js
public function add_engineer() {
if (IS_AJAX)
{
$posted_json_data = I('post.json_data');
$posted_json_data_replace = str_replace('"', '"', $posted_json_data);

$posted_json_data_replace_array = (Array)json_decode($posted_json_data_replace);

   //处理数据库事务写入，通过判断写入结果来区分ajaxReturn的结果
  //可以将所有想要返回的数据放在一个数组中，比如新增的行id、插入数据库的操作是否成功
  //如果操作数据库成功就返回如下结果。
   $user_table->commit();
$data['result'] = true;
$data['pk_user_id'] = $data_add_user_result;
$this->ajaxReturn($data);
return 0;
}
}
改写js：
在js的ajax中，如果整个ajax正常交互，就会走success函数，否则会走error函数，一般情况下，error出现的原因都是传输的数据不符合要求。
在success中的data就是ajaxReturn中传输的数组，举例：
success: function (data) {
                if (data['result'] == false)
                {
                    alert(data['alert']);
                }
                else
                {
                    $('#add_engineer_modal').modal('hide');
                    $('#user_list_table tr').eq(0).after('<tr></tr>');
                    //这里就可以使用data['pk_user_id']了。

                    $('#user_list_table tr').eq(1).append('<td>'+data['pk_user_id']+'</td>');

                }
            },

四、总结
整个过程是：
在php中编写页面中的表单、提交按钮等；
在js中对php中的按钮事件添加校验和触发函数，在js函数内，如果js对象的格式和内容正确就向控制器url（php中初始化）发起ajax请求；
控制器中的相应操作响应ajax请求，并判断数据后做数据库读写操作，然后对数据库操作结果做出判断，ajaxReturn返回js需要的数组；
当ajax成功返回时，js中ajax的success里面使用js重写（或初始化）需要显示的信息。
这样就完成了ajax异步局部刷新。