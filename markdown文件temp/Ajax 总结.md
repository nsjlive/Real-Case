### Ajax 总结

1. ##### XMLHttpRequest 对象的相关方法

     1.1  XHR创建对象

   ```javascript
   var xmlhttp;
   if (window.XMLHttpRequest)
     {// code for IE7+, Firefox, Chrome, Opera, Safari
     xmlhttp=new XMLHttpRequest();
     }
   else
     {// code for IE6, IE5
     xmlhttp=new ActiveXObject("Microsoft.XMLHTTP");
     }
   ```

   1.2 向服务端发送请求

   ```javascript
   xmlhttp.open("GET","test1.txt",true);   //规定请求的类型、URL 以及是否异步处理请求。
   xmlhttp.send();    //将请求发送到服务器。
   ```

   ###### 方法参数解释如下：

   open(*method*,*url*,*async*)

   - *method*：请求的类型；GET 或 POST
   - *url*：文件在服务器上的位置
   - *async*：true（异步）或 false（同步）

   send(*string*)

   - *string*：仅用于 POST 请求

   如果需要像 HTML 表单那样 POST 数据，请使用 setRequestHeader() 来添加 HTTP 头。然后在 send() 方法中规定您希望发送的数据：

   ```javascript
   xmlhttp.setRequestHeader("content-type","text/xml;charset=utf-8");    //向请求中添加 HTTP 头
   ```

   ###### 方法参数解释如下：

   setRequestHeader(*header*,*value*)

   - *header*: 规定头的名称
   - *value*: 规定头的值

###### 1.3 服务器响应

如需获得来自服务器的响应，请使用 XMLHttpRequest 对象的 responseText 或 responseXML 属性。

responseText 获得字符串形式的响应数据。
responseXML 获得 XML 形式的响应数据。

###### 1.4 onreadystatechange 事件

当请求被发送到服务器时，我们需要执行一些基于响应的任务。每当 readyState 改变时，就会触发 onreadystatechange 事件。

readyState：存有 XMLHttpRequest 的状态。从 0 到 4 发生变化。
status：响应的HTTP状态码。
通过之前的Ajax调用公网服务的代码可以具体的了解XMLHttpRequest对象的使用。

```javascript
var getXmlHttpRequest = function () {
    try{
     //主流浏览器提供了XMLHttpRequest对象
         return new XMLHttpRequest();
    }catch(e){
     //低版本的IE浏览器没有提供XMLHttpRequest对象，IE6以下
     //所以必须使用IE浏览器的特定实现ActiveXObject
         return new ActiveXObject("Microsoft.XMLHTTP");
    } 
};
//创建XMLHttpRequest对象
var xhr = getXmlHttpRequest();
//打开连接
xhr.open("post","http://ws.webxml.com.cn/WebServices/MobileCodeWS.asmx",true);
//设置数据类型
xhr.setRequestHeader("content-type","text/xml;charset=utf-8");
//设置回调函数
xhr.onreadystatechange=function(){
//判断是否发送成功和判断服务端是否响应成功
if(4 == xhr.readyState && 200 == xhr.status){
    alert(xhr.responseText);
  }
}
//组织SOAP协议数据 篇幅原因，此处信息省略
var soapXML = "<?xml version=\"1.0\" encoding=\"utf-8\"?>····
   ····";        
//发送数据
xhr.send(soapXML);
```

2. #### xmlhttp.readyState==4 && xmlhttp.status==200解析

##### 2.1 readyState（状态值）和status（状态码）的区别

readyState，是指运行AJAX所经历过的几种状态，无论访问是否成功都将响应的步骤，可以理解成为AJAX运行步骤，使用“ajax.readyState”获得
status，是指无论AJAX访问是否成功，由HTTP协议根据所提交的信息，服务器所返回的HTTP头信息代码，使用“ajax.status”获得
总体理解：可以简单的理解为state代表一个整体的状态。而status是这个大的state下面具体的小的状态。

##### 2.2 什么是readyState

readyState是XMLHttpRequest对象的一个属性，用来标识当前XMLHttpRequest对象处于什么状态。
readyState总共有5个状态值，分别为0~4，每个值代表了不同的含义

```javascript
0：初始化，XMLHttpRequest对象还没有完成初始化
1：载入，XMLHttpRequest对象开始发送请求
2：载入完成，XMLHttpRequest对象的请求发送完成
3：解析，XMLHttpRequest对象开始读取服务器的响应
4：完成，XMLHttpRequest对象读取服务器响应结束
```

##### 2.3 什么是status

status是XMLHttpRequest对象的一个属性，表示响应的HTTP状态码
在HTTP1.1协议下，HTTP状态码总共可分为5大类

```
1xx：信息响应类，表示接收到请求并且继续处理
2xx：处理成功响应类，表示动作被成功接收、理解和接受
3xx：重定向响应类，为了完成指定的动作，必须接受进一步处理
4xx：客户端错误，客户请求包含语法错误或者是不能正确执行
5xx：服务端错误，服务器不能正确执行一个正确的请求
```

```
100——客户必须继续发出请求
101——客户要求服务器根据请求转换HTTP协议版本
200——交易成功
201——提示知道新文件的URL
202——接受和处理、但处理未完成
203——返回信息不确定或不完整
204——请求收到，但返回信息为空
205——服务器完成了请求，用户代理必须复位当前已经浏览过的文件
206——服务器已经完成了部分用户的GET请求
300——请求的资源可在多处得到
301——删除请求数据
302——在其他地址发现了请求数据
303——建议客户访问其他URL或访问方式
304——客户端已经执行了GET，但文件未变化
305——请求的资源必须从服务器指定的地址得到
306——前一版本HTTP中使用的代码，现行版本中不再使用
307——申明请求的资源临时性删除
400——错误请求，如语法错误
401——请求授权失败
402——保留有效ChargeTo头响应
403——请求不允许
404——没有发现文件、查询或URl
405——用户在Request-Line字段定义的方法不允许
406——根据用户发送的Accept拖，请求资源不可访问
407——类似401，用户必须首先在代理服务器上得到授权
408——客户端没有在用户指定的饿时间内完成请求
409——对当前资源状态，请求不能完成
410——服务器上不再有此资源且无进一步的参考地址
411——服务器拒绝用户定义的Content-Length属性请求
412——一个或多个请求头字段在当前请求中错误
413——请求的资源大于服务器允许的大小
414——请求的资源URL长于服务器允许的长度
415——请求资源不支持请求项目格式
416——请求中包含Range请求头字段，在当前请求资源范围内没有range指示值，请求也不包含If-Range请求头字段
417——服务器不满足请求Expect头字段指定的期望值，如果是代理服务器，可能是下一级服务器不能满足请求
500——服务器产生内部错误
501——服务器不支持请求的函数
502——服务器暂时不可用，有时是为了防止发生系统过载
503——服务器过载或暂停维修
504——关口过载，服务器使用另一个关口或服务来响应用户，等待时间设定值较长
505——服务器不支持或拒绝支请求头中指定的HTTP版本
```

##### 2.4 思考问题：为什么onreadystatechange的函数实现要同时判断readyState和status呢？

第一种思考方式：只使用readyState

```javascript
var getXmlHttpRequest = function () {
  if (window.XMLHttpRequest) {
    return new XMLHttpRequest();
  }
  else if (window.ActiveXObject) {
    return new ActiveXObject("Microsoft.XMLHTTP");
  }
};
var xhr = getXmlHttpRequest();
xhr.open("get", "1.txt", true);
xhr.send();
xhr.onreadystatechange = function () {
  if (xhr.readyState == 4) {
    alert(xhr.responseText);
  }
};
```

服务响应出错了，但还是**返回了信息**，这并不是我们想要的结果
如果返回不是200，而是404或者500，由于只使用readystate做判断，它不理会放回的结果是200、404还是500，只要响应成功返回了，就**执行**接下来的javascript代码，结果将造成各种不可预料的错误。所以只使用readyState判断是行不通的。

第二种思考方式：只使用status判断

```javascript
var getXmlHttpRequest = function () {
  try{
    return new XMLHttpRequest();
  }catch(e){
    return new ActiveXObject("Microsoft.XMLHTTP");
  }
};
var xhr = getXmlHttpRequest();
xhr.open("get", "1.txt", true);
xhr.send();
xhr.onreadystatechange = function () {
    if (xhr.status == 200) {
        alert("readyState=" + xhr.readyState + xhr.responseText);
    }
};
```

事实上，结果却不像预期那样。响应码确实是返回了200，但是总共弹出了3次窗口！第一次是“readyState=2”的窗口，第二次是“readyState=3”的窗口，第三次是“readyState=4”的窗口。由此，可见**onreadystatechange函数的执行不是只在readyState变为4**的时候触发的，而是readyState（2、3、4）的每次变化都会触发，所以就出现了前面说的那种情况。可见，单独使用status判断也是行不通的。

##### 5. readyState和status的先后判断顺序

由上面的试验，我们可以知道判断的时候readyState和status缺一不可。那么readyState和status的先后判断顺序会不会有影响呢？我们可以将status调到前面先判断，代码如 xhr.status == 200 && xhr.readyState == 4
事实上，这对于最终的结果是**没有影响的**，但是中间的性能就不同了。由试验我们知道，readyState的每次变化都会触发onreadystatechange函数，假如先判断status，那么每次都会多判断一次status的状态。虽然性能上影响甚微，不过还是应该抱着追求极致代码的想法，把readyState的判断放在前面。
xhr.readyState == 4 && xhr.status == 200

















