# http-
http传输的一些积累吧


###关于cookie
	> hostOnly
	> httpOnly

###Content-Security-Policy
	> 限定浏览器加载资源
		e.g:   script-src	'self' js.a.com	定义针对 JavaScript 的加载策略。

###一些安全相关的HTTP响应头
	> Strict-Transport-Security
		必须https访问页面
	> X-Frame-Options
		是否允许其他页面作为iframe加载进入
### 状态行、请求头、消息主体

### content-type:
	> Content-Type: application/x-www-form-urlencoded  =》原生form表单提交(enctype未设置)

	> Content-Type: multipart/form-data; => enctype:multipart/form-data

	> Content-Type: application/json

	> Content-Type: text/xml

#### Proxy-Connection: 代理会将connection的请求头转发，是的代理服务器傻等服务器keep-alive的结束。

### 内容协商专用字段
	> 客户端发送的请求:
				Accept	告知服务器发送何种媒体类型	Content-Type
		Accept-Language	告知服务器发送何种语言	Content-Language
		Accept-Charset	告知服务器发送何种字符集	Content-Type
		Accept-Encoding	告知服务器采用何种压缩方式	Content-Encoding

### HTTP/1.0 的持久连接机制是后来才引入的，通过 Connection: keep-alive 这个头部来实现，服务端和客户端都可以使用它告诉对方在发送完数据之后不需要断开 TCP 连接
	e.g 
		require('net').createServer(function(sock) {
		    sock.on('data', function(data) {
		        sock.write('HTTP/1.1 200 OK\r\n');
		        sock.write('\r\n');
		        sock.write('hello world!');
		        sock.destroy();
		    });
		}).listen(9090, '127.0.0.1');
		启动服务后，在浏览器里访问 127.0.0.1:9090，正确输出了指定内容，一切正常。去掉 sock.destroy() 这一行，让它变成持久连接，重启服务后再访问一下。这次的结果就有点奇怪了：迟迟看不到输出，通过 Network 查看请求状态，一直是 pending。
		这是因为，对于非持久连接，浏览器可以通过连接是否关闭来界定请求或响应实体的边界；而对于持久连接，这种方法显然不奏效。上例中，尽管我已经发送完所有数据，但浏览器并不知道这一点，它无法得知这个打开的连接上是否还会有新数据进来，只能傻傻地等了。

### content-length
	e.g (content-length 用于keep-alive告诉什么时候结束)
	require('net').createServer(function(sock) {
	    sock.on('data', function(data) {
	        sock.write('HTTP/1.1 200 OK\r\n');
	        sock.write('Content-Length: 12\r\n');
	        sock.write('\r\n');
	        sock.write('hello world!');
	    });
	}).listen(9090, '127.0.0.1');

### 分块编码
	分块编码相当简单，在头部加入 Transfer-Encoding: chunked 之后，就代表这个报文采用了分块编码。这时，报文中的实体需要改为用一系列分块来传输。每个分块包含十六进制的长度值和数据，长度值独占一行，长度不包括它结尾的 CRLF（\r\n），也不包括分块数据结尾的 CRLF。最后一个分块长度值必须为 0，对应的分块数据没有内容，表示实体结束。按照这个格式改造下之前的代码：
	require('net').createServer(function(sock) {
	    sock.on('data', function(data) {
	        sock.write('HTTP/1.1 200 OK\r\n');
	        sock.write('Transfer-Encoding: chunked\r\n');
	        sock.write('\r\n');

	        sock.write('b\r\n');
	        sock.write('01234567890\r\n');

	        sock.write('5\r\n');
	        sock.write('12345\r\n');

	        sock.write('0\r\n');
	        sock.write('\r\n');
	    });
	}).listen(9090, '127.0.0.1');

### 代理
	var http = require('http');
	var net = require('net');
	var url = require('url');

	function request(cReq, cRes) {
    	var u = url.parse(cReq.url);

	    var options = {
	        hostname : u.hostname, 
	        port     : u.port || 80,
	        path     : u.path,       
	        method     : cReq.method,
	        headers     : cReq.headers
	    };

	    var pReq = http.request(options, function(pRes) {
	        cRes.writeHead(pRes.statusCode, pRes.headers);
	        pRes.pipe(cRes);
	    }).on('error', function(e) {
	        cRes.end();
	    });

	    cReq.pipe(pReq);
	}

### Content-Encoding 是 HTTP 中用来对「采用何种编码格式传输正文」

http.createServer().on('request', request).listen(8888, '0.0.0.0');


### 状态码小记
301：永久移动
303/307: 临时搬离的资源（see other/ temporary redirect）
500：服务器错误
501: 请求方法不认识
502：响应链问题
503：暂时不可用

etag 唯一代表没有更改


###
浏览器解析主机名
浏览器查询这个主机的ip
浏览器获取端口号
浏览器向服务端发送一条http响应报文
浏览器从服务器读取http响应报文
浏览器关闭连接

服务器建立连接
服务器接收请求
服务器处理请求
服务器访问资源
服务器构建响应
服务器发送响应
服务器记录事务处理过程

###
持久连接不需要再次连接时间
keep-alive max=5 允许最大5个事务

###
代理：协议连接器
网关：协议转化器

###代理
代理可以集中访问控制权限
内容路由器，转码器

###缓存
	1.过期标志=> Expires Cache-Control

	authentication http header => 401 Login required

	2.cookie 临时/永久

###HTTPS
	HTTP 应用层
	SSL or TLS 安全层
	TCP 传输层
	IP 网络层
	网络接口 数据链路层

	=> SSL
		.交换协议版本号。
		.选择一个两端都了解的密码。
		.对两端的身份进行认证。
		.生成临时的会话密钥，以便加密信道。


DNS轮转 =》负载均衡