

entity-enclosing methods

对于子自包含实体 只能用HttpEntity#writeTo(OutputStream)  不能用HttpEntity#getContent()


EntityUtils.toString(myEntity)


对于流式实体读取方法：
1.
``` java
HttpResponse response;
HttpEntity entity = response.getEntity();
if (entity != null) {
    InputStream instream = entity.getContent();
    try {
        // do something useful
    } finally {
        instream.close();
    }
}
```
2.
```java
// ensure that the entity content has been fully consumed and the underlying stream has been closed.
EntityUtils#consume(HttpEntity)
```

`InputStreamEntity` 支持输入流和长度两个参数，使用长度可以限制读取的数量，当长度为负值时可以读取所有


`HttpProcessor` 用于执行HTTP请求前和请求后的预处理以及后处理操作

> *HTTP最初的设想是无状态协议，仅仅面向请求和应答，但是实际使用时必须保留一个状态，HttpContext保留了多个逻辑相关请求的一个上下文 ？ *


阻塞HTTP连接使用方法：
  * 客户端：

``` java
Socket socket = <...>

DefaultBHttpClientConnection conn = new DefaultBHttpClientConnection(8 * 1024);
conn.bind(socket);
System.out.println(conn.isOpen());
HttpConnectionMetrics metrics = conn.getMetrics();
System.out.println(metrics.getRequestCount());
System.out.println(metrics.getResponseCount());
System.out.println(metrics.getReceivedBytesCount());
System.out.println(metrics.getSentBytesCount());
```

* 服务端：

```java
Socket socket = <...>

DefaultBHttpServerConnection conn = new DefaultBHttpServerConnection(8 * 1024);
conn.bind(socket);
HttpRequest request = conn.receiveRequestHeader();
if (request instanceof HttpEntityEnclosingRequest) {
    conn.receiveRequestEntity((HttpEntityEnclosingRequest) request);
    HttpEntity entity = ((HttpEntityEnclosingRequest) request)
            .getEntity();
    if (entity != null) {
        // Do something useful with the entity and, when done, ensure all
        // content has been consumed, so that the underlying connection
        // could be re-used
        EntityUtils.consume(entity);
    }
}
HttpResponse response = new BasicHttpResponse(HttpVersion.HTTP_1_1,
        200, "OK") ;
response.setEntity(new StringEntity("Got it") );
conn.sendResponseHeader(response);
conn.sendResponseEntity(response);
```

> 浏览HTTP协议   HTTP/1.1 specification ?

> trivial pattern matching algorithm ?

> TLS/SSL 安全协议具体内容是什么？ [SSL/TLS协议运行机制的概述](http://www.ruanyifeng.com/blog/2014/02/ssl_tls.html)


> java Socket 简单原理理解

> 同步IO和异步IO有什么区别？

> java 异步IO

> TCP/IP 四层协议： 链路层(ARP协议： 通过在可信网络中将IP地址转换成mac地址)->网络层（ICMP、IP 路由）->传输层（TCP、UDP）->应用层(HTTP)



keytool -genkey -alias serverkey -keystore kserver.keystore

keytool -export -alias serverkey -keystore kserver.keystore -file server.crt

keytool -import -alias serverkey -file server.crt -keystore tclient.keystore

119.75.217.109:80
