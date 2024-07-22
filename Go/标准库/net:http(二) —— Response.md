## HTTP Response结构

`HTTP Response`（HTTP响应）是在客户端（`Client`）向服务器（`Server`）发送HTTP请求（`Request`）之后，服务器返回给客户端的数据。

同样可以使用 `curl` 或 `http`命令发起`HTTP`请求或者在浏览器端下也可以使用开发者工具查看。

下图为一个`Http Response`的示例：



![img](https://cos.liuqm.cc/v2-f86248856390b36aac06aff206593727_1440w.webp)

image-20230215153738413



从上述图示中可以看出，`Http Response`报文的格式：

```text
响应行
响应头
响应空行
响应体
```

来展开分析下吧。

### 响应行

响应行包括协议版本、状态码和状态码原因短语。他们之间使用空格隔开。

例如上述图示中请求行内容为：`HTTP/1.1 302 Found`，则：

- 协议版本为 `HTTP/1.1`

响应协议版本通常与请求（`request`）的协议版本相同。常用的HTTP协议版本包括HTTP/1.0和HTTP/1.1，其中HTTP/1.1是目前使用最广泛的HTTP协议版本。

- 状态码为 `302`

响应状态码表示服务器对请求的响应结果。在浏览器场景下，浏览器会根据状态码做出相应的处理。常见的状态码如下表：

| 状态码 | 含义                                                 |
| ------ | ---------------------------------------------------- |
| 200    | 请求已经被成功处理                                   |
| 201    | 请求已经被成功处理，并且在服务器上创建了新的资源     |
| 204    | 请求已经被成功处理，但响应报文中不包含实体的主体部分 |
| 301    | 请求的资源已经被永久移动到了新的URI                  |
| 302    | 请求的资源被临时移动到了新的URI                      |
| 304    | 请求的资源未被修改，客户端可以使用本地缓存的版本     |
| 400    | 请求的语法错误或无法被服务器理解                     |
| 401    | 请求需要用户验证，该响应必须包含WWW-Authenticate头部 |
| 403    | 服务器拒绝执行请求，客户端没有权限访问请求的资源     |
| 404    | 请求的资源不存在                                     |
| 500    | 服务器在执行请求时发生了错误                         |
| 502    | 服务器作为网关或代理时从上游服务器接收到无效的响应   |
| 503    | 服务器暂时无法处理请求，通常由于服务器过载或维护     |

完整的状态码可以参考： [https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status](https://link.zhihu.com/?target=https%3A//developer.mozilla.org/zh-CN/docs/Web/HTTP/Status)

- 状态码原因为 `Found`

同上， 状态码原因是跟状态码一一对应的，可以参考： [https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status](https://link.zhihu.com/?target=https%3A//developer.mozilla.org/zh-CN/docs/Web/HTTP/Status)

### 响应头

HTTP **响应头**（response header）是一种 `HTTP`标头，其可以用于 `HTTP`响应，且与响应消息主体无关。它们被用于描述服务器的基本信息，以及数据的描述，服务器通过这些数据的描述信息，可以通知客户端如何处理等一会儿它回送的数据。

HTTP响应头部包含一组属性-值对，用于描述HTTP响应的各种属性，例如响应的内容类型、长度、缓存控制等。这些属性通常以键值对的形式出现，其中键表示属性名，值表示属性的值，以冒号（:）分隔。HTTP响应头部以一个空行作为结束标志。

例如：

```http
Bdpagetype: 3
Connection: keep-alive
Content-Length: 154
Content-Type: text/html
Date: Wed, 15 Feb 2023 07:31:16 GMT
Location: https://www.baidu.com/search/error.html
```

下面来罗列下常见的HTTP 响应头：

| 头部名称                    | 描述                         |
| --------------------------- | ---------------------------- |
| Content-Type                | 指定响应主体的MIME类型       |
| Content-Length              | 指定响应主体的长度           |
| Cache-Control               | 指定缓存控制策略             |
| Server                      | 指定HTTP服务器的名称和版本号 |
| Date                        | 指定响应产生的时间           |
| Last-Modified               | 指定资源的最后修改时间       |
| ETag                        | 指定资源的实体标识           |
| Expires                     | 指定响应的过期时间           |
| Location                    | 重定向的URL地址              |
| Set-Cookie                  | 指定一个或多个HTTP cookie    |
| Access-Control-Allow-Origin | 指定跨域请求的允许域名       |
| Content-Encoding            | 指定响应主体的压缩方式       |
| Transfer-Encoding           | 指定传输编码方式             |
| Connection                  | 指定是否保持连接             |
| Vary                        | 指定缓存的变化因素           |

除了标准的HTTP响应头外，其实响应头部中的属性可以根据需要进行添加、修改或删除，以便客户端和服务器之间进行更加灵活的交互。

完整版本的响应头参考： [https://en.wikipedia.org/wiki/List_of_HTTP_header_fields](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/List_of_HTTP_header_fields)

### 响应体

`HTTP`响应体（`Response Body`）是`HTTP`响应的一部分，包含服务器返回给客户端的数据。它跟随在`HTTP`响应头之后，并以空行分隔。HTTP响应体中的数据通常是`HTML`、`XML`、`JSON`或其他格式的文本、图像、音频或视频等多媒体内容。

`HTTP`响应体的格式通常由`Content-Type`响应头部指定，例如，`Content-Type: text/html`表示响应体是`HTML`格式的文本数据，`Content-Type: image/jpeg`表示响应体是`JPEG`格式的图像数据。常见的响应体`Content-Type`如下：

- text/html：HTML格式的文本数据，用于显示网页。
- text/plain：纯文本格式的数据，适用于显示纯文本文档。
- application/json：JSON格式的数据，用于Web应用程序之间传递数据。
- application/xml：XML格式的数据，适用于Web应用程序之间传递数据或存储结构化文档。
- image/png、image/jpeg：PNG或JPEG格式的图像数据，用于显示图像。
- audio/mpeg、video/mp4：MP3或MP4格式的音频或视频数据，用于播放音频或视频。
- application/octet-stream：二进制数据，适用于传输未知的二进制文件类型。

除了以上列出的Content-Type类型，还有很多其他的Content-Type类型，可以根据实际需求进行指定。另外，`Content-Length`响应头部指定了响应体的字节数。

`HTTP`响应体的内容可以是静态的文件，也可以是动态生成的内容，例如通过`CGI`、`ASP`、`PHP`等技术生成的网页内容。在一些特殊情况下，`HTTP`响应体可以为空，例如`HTTP 204 No Content`状态码。

`HTTP`响应体还可以用来传递服务器端的状态信息、错误信息和调试信息等，方便开发者调试和维护`Web`应用程序。由于HTTP响应体可能包含大量数据，因此在传输过程中需要采用压缩和分块传输等技术，以提高传输效率和降低带宽消耗。

### net/http包中的Response

在`Go`语言的`net/http`包中，`response.go`文件包含了HTTP响应（`Response`）相关的代码实现，我们逐一将重点展开分析。

### Response结构体

`Response`结构体定义了`HTTP`响应的基本属性，如状态码、`HTTP`头部、响应体等。

`Response`结构体也提供了一些便捷的方法，如`Write`、`WriteHeader`等，用于设置响应体和`HTTP`头部等属性。

`Response`结构体定义源码如下：

```go
type Response struct {
    Status           string               //表示HTTP响应的状态行中的状态码和原因短语，例如"200 OK"
    StatusCode       int                  //表示HTTP响应的状态码，例如200
    Proto            string               //表示使用的HTTP协议版本，例如"HTTP/1.1"
    ProtoMajor       int                  //表示使用的HTTP协议版本的主版本号，例如1
    ProtoMinor       int                  //表示使用的HTTP协议版本的副版本号，例如1
    Header           Header               //表示HTTP响应的头部信息，是一个Header类型的映射
    Body             io.ReadCloser        //表示HTTP响应的主体，是一个io.ReadCloser类型的接口，可以读取响应的数据
    ContentLength    int64                //表示HTTP响应的主体的长度，如果长度未知则为-1
    TransferEncoding []string             //表示HTTP响应的传输编码，例如"chunked"，如果未设置则为nil
    Close            bool                 //表示HTTP响应是否需要关闭连接
    Uncompressed     bool                 //表示HTTP响应是否已经解压缩
    Trailer          Header               //表示HTTP响应的头部信息中未确定的部分
    Request          *Request             //表示发送HTTP请求的Request结构体，如果未设置则为nil
    TLS              *tls.ConnectionState //表示HTTP响应的TLS连接信息，如果未使用TLS则为nil
}
```

`Response`结构体的字段表示了`HTTP`响应的各种属性，包括状态码、`HTTP`头部、响应体、`HTTP`协议版本、传输编码等。这些字段是`HTTP`服务器处理`HTTP`请求并生成`HTTP`响应时必不可少的。

### ReadResponse

`ReadResponse`函数用于解析HTTP响应，将响应字节流转换为`Response`结构体类型。该函数会从给定的`io.Reader`对象中读取`HTTP`响应的各个部分，包括状态行、响应头部、响应体等，并将其组合成`Response`类型的结构体，以便后续处理。

```go
/**
参数说明:
    r  指向bufio.Reader类型的指针r，表示HTTP响应的字节流数据来源
    req 指向http.Request类型的指针req，表示与HTTP响应相关的HTTP请求信息
*/
func ReadResponse(r *bufio.Reader, req *Request) (*Response, error) {
    //创建一个textproto.Reader类型的对象tp，并将bufio.Reader对象r传递给它，用于读取HTTP响应的文本部分
    tp := textproto.NewReader(r)
    //传入req创建Response对象
    resp := &Response{
        Request: req,
    }
    //调用tp.ReadLine()方法读取HTTP响应的第一行，即响应行，并将读取的文本内容保存在line变量中
    line, err := tp.ReadLine()
    if err != nil {
        if err == io.EOF {
            err = io.ErrUnexpectedEOF
        }
        return nil, err
    }
    //解析响应行line变量，如果格式正确，则解析变量中的Proto协议信息以及Status状态信息，否责报badStringError类型错误
    if i := strings.IndexByte(line, ' '); i == -1 {
        return nil, badStringError("malformed HTTP response", line)
    } else {
        resp.Proto = line[:i]
        resp.Status = strings.TrimLeft(line[i+1:], " ")
    }
    //解析Status状态信息，如果格式正确，则解析出状态码并赋值给statusCode，否责报badStringError类型错误
    statusCode := resp.Status
    if i := strings.IndexByte(resp.Status, ' '); i != -1 {
        statusCode = resp.Status[:i]
    }
    //校验状态码长度
    if len(statusCode) != 3 {
        return nil, badStringError("malformed HTTP status code", statusCode)
    }

    //校验状态码信息
    resp.StatusCode, err = strconv.Atoi(statusCode)
    if err != nil || resp.StatusCode < 0 {
        return nil, badStringError("malformed HTTP status code", statusCode)
    }
    var ok bool
    //解析校验协议版本信息
    if resp.ProtoMajor, resp.ProtoMinor, ok = ParseHTTPVersion(resp.Proto); !ok {
        return nil, badStringError("malformed HTTP version", resp.Proto)
    }

    //解析HTTP响应头部，并将解析结果存储在resp的Header字段中
    //ReadMIMEHeader方法实际上是先读取到HTTP头部和空行之间的所有文本行，然后将它们解析为键值对，最后返回一个map[string][]string类型的对象，其中键表示HTTP头部的名称，值表示HTTP头部的值
    mimeHeader, err := tp.ReadMIMEHeader()
    if err != nil {
        if err == io.EOF {
            err = io.ErrUnexpectedEOF
        }
        return nil, err
    }
    resp.Header = Header(mimeHeader)
    //调用了fixPragmaCacheControl函数，该函数用于规范化HTTP头部中的Pragma和Cache-Control字段，使得这两个字段只出现一次，并且同时存在时Cache-Control字段的优先级更高。
    //这样做是因为Pragma字段在HTTP/1.0时代用于控制缓存，而Cache-Control字段是HTTP/1.1引入的用于控制缓存的标准字段
    fixPragmaCacheControl(resp.Header)

    //调用readTransfer函数，读取HTTP响应消息体
    err = readTransfer(resp, r)
    if err != nil {
        return nil, err
    }
    return resp, nil
}
```

### Response.Write

`Response.Write`方法用于将`HTTP`响应数据写入到连接的客户端，其具体功能如下：

1. 根据`HTTP`响应头部中的`Content-Type`字段确定`HTTP`响应数据的类型，设置`HTTP`响应数据的`Content-Type`字段；
2. 将`HTTP`响应头部和`HTTP`响应数据写入到连接的客户端。

此方法的函数签名如下：

```go
func (r *Response) Write(w io.Writer) error
```

其中，`r`表示要写入的`HTTP`响应数据，`w`表示连接的客户端。

此方法会先将`HTTP`响应头部写入到连接的客户端，然后再将`HTTP`响应数据写入到连接的客户端。在写入`HTTP`响应数据之前，会先写入`Content-Length`字段，表示`HTTP`响应数据的长度，这样客户端才能正确地读取到`HTTP`响应数据。

写入过程中可能会出现各种错误，因此`Write`方法的返回值为`error`类型，表示写入过程中出现的任何错误。

`Response.Write` 源码如下：

```go
func (r *Response) Write(w io.Writer) error {
    //将状态信息从Status中解析出来
    text := r.Status
    if text == "" {
        var ok bool
        text, ok = statusText[r.StatusCode]
        if !ok {
            text = "status code " + strconv.Itoa(r.StatusCode)
        }
    } else {
        text = strings.TrimPrefix(text, strconv.Itoa(r.StatusCode)+" ")
    }
    //向客户端写入响应行信息，包括协议 状态码 状态信息
    if _, err := fmt.Fprintf(w, "HTTP/%d.%d %03d %s\r\n", r.ProtoMajor, r.ProtoMinor, r.StatusCode, text); err != nil {
        return err
    }

    //创建Response指针对象r1,并将*r赋值给r1
    r1 := new(Response)
    *r1 = *r

    /**
    响应主体不为空但响应主体长度为0的情况，读取响应主体信息以及长度并赋值给n和buf变量：
    1. 如果buf不为空，并且不是io.EOF结束符，则将buf作为错误信息直接返回
    2.
        2-1 如果n等于0，表示主体为空，将Response.Body赋值为noBody空结构
        2-2 如果n不等于0，设置Response.ContentLength长度为-1，表示响应未知长度；
            接着创建了一个结构体，其中包含两个接口类型 io.Reader 和 io.Closer这个结构体被用来替换 Response 对象的 Body 属性，也就是响应体的内容
            新的响应体是一个 io.MultiReader 对象，由两个 Reader 组成，第一个 Reader 是一个长度为1的 bytes.Reader 对象，第二个 Reader 是原始响应对象的 Body。
            io.MultiReader 对象可以将多个 Reader 拼接在一起，用于顺序读取多个 Reader 的内容
    */
    if r1.ContentLength == 0 && r1.Body != nil {
        var buf [1]byte
        n, err := r1.Body.Read(buf[:])
        if err != nil && err != io.EOF {
            return err
        }
        if n == 0 {
            r1.Body = NoBody
        } else {
            r1.ContentLength = -1
            r1.Body = struct {
                io.Reader
                io.Closer
            }{
                io.MultiReader(bytes.NewReader(buf[:1]), r.Body),
                r.Body,
            }
        }
    }

    //根据HTTP协议的规范决定是否需要在响应结束后关闭连接。例如，如果响应体的长度未知，连接可能需要在响应结束后关闭
    if r1.ContentLength == -1 && !r1.Close && r1.ProtoAtLeast(1, 1) && !chunked(r1.TransferEncoding) && !r1.Uncompressed {
        r1.Close = true
    }

    //根据 r1 中的信息，创建一个新的 TransferWriter 对象并返回
    tw, err := newTransferWriter(r1)
    if err != nil {
        return err
    }
    //写入响应头信息
    err = tw.writeHeader(w, nil)
    if err != nil {
        return err
    }

    //排除掉指定的响应头部信息，即respExcludeHeader里面的头部信息
    err = r.Header.WriteSubset(w, respExcludeHeader)
    if err != nil {
        return err
    }
    //调用 tw.shouldSendContentLength() 方法，用于检查是否已经在之前的响应头部中添加了 Content-Length 字段。
    //如果已经添加过，那么无需再次添加。然后，它检查当前响应体的长度是否为 0，是否使用了分块编码，以及当前响应状态码是否允许包含响应体。
    //如果满足这些条件，则需要在响应头部中添加 Content-Length: 0 字段，表示响应体为空。
    contentLengthAlreadySent := tw.shouldSendContentLength()
    if r1.ContentLength == 0 && !chunked(r1.TransferEncoding) && !contentLengthAlreadySent && bodyAllowedForStatus(r.StatusCode) {
        if _, err := io.WriteString(w, "Content-Length: 0\r\n"); err != nil {
            return err
        }
    }

    //写入响应空行
    if _, err := io.WriteString(w, "\r\n"); err != nil {
        return err
    }
    //写入响应体
    err = tw.writeBody(w)
    if err != nil {
        return err
    }
    return nil
}
```

`Response`内容大致如此，后续将对 `Client`和`Server`内容进行分解，以完善`net/http`的脉络。