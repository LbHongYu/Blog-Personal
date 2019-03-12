## Ajax 原理

Ajax 是 Asynchronous JavaScript + XML 的简写，是能够像服务器请求额外数据而无须加载页面的技术。Ajax的核心技术是 XMLHttpRequest 对象（简称 XML），XHR 提供向服务器发送请求和解析服务器响应的借口，能够以异步方式从服务器取得数据，替代了以刷新页面的方式。用户取得数据之后，再通过DOM操作将数据插入页面。

Ajax 的名字中包含了 "XML" 成分，但是Ajax通信技术与数据格式无关，从服务器返回的数据也不一定是XML数据。

我们看看 如何创建一个XHR对象：
```
function createXHR(){
	let xhr = null;
	if(window.XMLHttpRequest){
		xhr = new XMLHttpRequest();
	}else{
        //兼容 IE7-
		xhr = new ActiveXObject('Microsoft.XMLHTTP');
	}
	return xhr;
}
```
接下来我们看看XHR对象实例有哪些属性和方法，和如何使用它来请求和获取数据。
```XMLHttpRequest``` 对象实例方法：

1. ```open()``` 开启一个请求
接受三个参数：请求方法，请求地址，是否异请求

2. ```send()``` 发送请求， 如果是GET方法，send方法需要传入 ```null```

3. ```setRequestHeader()``` 自定义请求头。

4. ```getRequestHeader(header)``` 获取请求头内容。

5. ```getAllRequestHeaders()``` 获取全部请求头内容。


6. 收到服务器响应之后，数据会自动填充到XHR对象属性中：
	+ ```responseText```: 响应主体中的文本。
	+ ```responseXML```: 如果响应的内容是 "text/xml" 或 "application/xml",
							 这个属性将包含响应数据的 XML DOM 文档。
	+ ```status```： 响应状态码
	+ ```statusText```： HTTP 状态的说明。
	+ ```readyState``` : 表示请求/响应过程的当前活动阶段。
		* 0: 未初始化，此时尚未调用 ```open``` 方法。
		* 1： 启动，调用 ```open``` 之后，调用 ```send``` 之后。
		* 2： 发送， 调用```send``` 之后 ，等到响应。
		* 3： 接受。接受了部分数据。
		* 4： 数据完全返回。

7. ```abort()``` 取消异步请求
	调用这个方法后， XHR对象会停止触发 onreadystatechange 事件， 且不在允许访问任何与响应相关的对象属性。调用abort之后应该解除 XHR 对象的引用，由于内存原因不建议重用 XHR 对象。

*我们应该使用 DOM0 级方法添加事件处理程序，因为并非所有的浏览器支持 DOM2 级方法，与其他事件处理程序不同， 没有在 onreadystatechange 事件处理程序中传递 event， 没有引用 ```this``` 来获取 XHR 对象。 原因是 onreadystatechange 事件处理程序的作用域问题，如果使用 this 对象，在有的浏览器中会导致函数执行失败或者错误发生。
```
function createXHRInstance(xhr, resolveCb, rejectCb){
    var xhr = createXHR();
	xhr.onreadystatechange = function(){
		if(xhr.readyState === 4){
			if((xhr.status >= 200 && xhr.status <= 300) || xhr.status === 304){
				resolveCb(xhr.response);
			}else{
				rejectCb()
			}
		}
	}

	return xhr;
}

function fetchRequest(url, method, data, xhr){
    xhr.open(method, url, ture);

    xhr.send(data);

    return xhr;
}

function parseURLParam(url, data){
	if(Object.prototype.toString.call(data) === "[object Object]"){
		url += ~(url.indexOf("?")) ? "?" :'&';
		for(var i in data){
			if(Object.prototype.hasOwnProperty.call(data, i)){
				url += encodeURIComponent(i) + '=' + encodeURIComponent(data[i]);
			}
		}
	}else{
		console.log('data should be a object');
	}
	return url;
}
```

#### XMLHttpRequest 2 级


1. ```FormData``` 用于提供表单数据的序列化。

添加数据的方法： 
	1. FormData对象实例的append方法： ```new FormData().append(key, value);```
	2. 往FormData方法传入一个form元素： ```new FormData(formElement);```

使用FormData 对象可以不用在XHR对象手动设置请求头部， XHR对象能够识别传入的数据类型是 FormData 实例，并配置适当的头部信息

2. IE8+ 新增 ```timeout``` 属性

XHR 对象 timeout 属性表示接受一个毫秒数的数字，表示发送请求后等待响应多少毫秒之后就终止。设置这个属性之后，如果发生超时就会触发 timeout 事件。

xhr.timeout = 30000;
xhr.ontimeout = function(){
	//the request is timeout
}

在请求终止时，会调用 ontimeout 事件处理程序，。 如果在超时终止请求之后再访问 status 属性，就会导致错误。为避免浏览器报告错误，可以将检查 status 属性的语句放在 try-catch 语句当中。


3. ```overrideMimeType()``` 方法
 overrideMimeType 用于重写 XHR 响应的 MIME 类型，在调用 send 之前。传入一个希望得到的文件类型。
 这样就可以重写服务器返回的MIME类型。


#### 进度事件
	定义了与客户端服务器通信有关的事件。

* ```loadstart``` 接收到响应数据的第一个字节时触发。
* ```process``` 接收响应期间被一次或者多次触发
* ```error``` 请求发生错误时错误
* ```abort``` 调用abort方法而终止连接时触发
* ```load``` 接收完整的响应数据时触发,
* ```loadend``` 在通信完成或者触发error、abort 或 load 事件后触发。

	loadstart ---> process ---> error/abort/load ---> loadend

 1. load 事件： 可以使用这个事件代替 onreadystatechange，且不用判断readyState

	load 事件处理程序会接收到一个event对象，event.target 指向XHR实例，但是也不要通过这种方式回去xhr实例对象，并非所有浏览器都提供了这个event对象。

```	
 function getData(xhr){
	xhr.load = function(){
		if((xhr.status >= 200 && xhr.status <= 300) || xhr.status === 304){
			getDateSuccess(xhr.response);
		}else{
			getDateFailed()
		}
	}
	return xhr;
}
```

2. progress 事件

process 接收响应期间周期性触发，onprogress 事件处理程序会接收一个event对象，event.target指向XHR对象,还有三个表示数据返回进度的信息。onprogress最好是在open方法之前添加。

* lengthComputable 表示进度可用的布尔值
* position 已经接受的字节数
* totalSize 根据Content-length响应头部确定的预期字节数


#### 跨源资源共享

