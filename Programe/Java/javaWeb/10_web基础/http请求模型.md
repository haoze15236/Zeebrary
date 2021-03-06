# Request请求

http请求可以划分为如下几个部分:

![img](https://img-blog.csdnimg.cn/20190527111928530.png)

## 请求方法

- **get** : 请求数据可以在url上直接看到,数据不安全,且url大小有限,因此传输的数据也有限,在Rest风格编码中一般用于获取数据
- **post** :根据请求头部中的`Content-Type`来定义多种不同格式的请求数据,Rest风格编码中一般用于新增数据
- **put** :Rest风格编码中一般用于修改数据
- **delete** :Rest风格编码中一般用于删除数据

## 请求头

### **Content-Type**

说明此次请求中请求数据的格式。

- **application/x-www-form-urlencoded**:post请求默认数据格式,使用URLencode转码

- **multipart/form-data**: 一般用于文件上传等长数据量参数的格式,其格式如下:

  ```http
  POST http://www.example.com HTTP/1.1
  Content-Type:multipart/form-data; boundary=----WebKitFormBoundaryyb1zYhTI38xpQxBK
  
  ------WebKitFormBoundaryyb1zYhTI38xpQxBK
  Content-Disposition: form-data; name="city_id"
  
  1
  
  ------WebKitFormBoundaryyb1zYhTI38xpQxBK
  Content-Disposition: form-data; name="company_id"
  
  2
  ------WebKitFormBoundaryyb1zYhTI38xpQxBK
  Content-Disposition: form-data; name="file"; filename="chrome.png"
  Content-Type: image/png
  
  PNG ... content of chrome.png ...
  ------WebKitFormBoundaryyb1zYhTI38xpQxBK--
  ```

- **application/json** ：平常开发采用的最多的格式,用于传送json报文。

### Accpet

告诉服务端,客户端能够处理的数据格式。

### Accept-Encoding

这个属性是用来告诉服务器能接受压缩形式，例如:Accept-Encoding:gzip, deflate。

### Accept-Language

表示客户端支持的语言格式,如中文/英文，通常浏览器直接发起请求时，浏览器会根据被设置的语言环境（默认语言），来附加上该字段。

### Referer

请求的来源，即当前请求是从哪个网址跳转过来的。

### Cache-Control

对缓存进行控制,如一个请求希望响应的内容在客户端缓存一年,或不被缓可以通过这个报文头设置

![这里写图片描述](https://img-blog.csdn.net/20180329184105968?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0JvYm9tYV9kdXQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### Host

指定要请求的资源所在的主机和端口

### User-Agent 

告诉服务器，客户端使用的操作系统、浏览器版本和名称

# Response响应