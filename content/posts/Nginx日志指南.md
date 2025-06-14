---
title: 'Nginx日志指南'
date: '2024-11-12T10:24:09+08:00'
draft: false
author: vdong
description: ''
categories:
  - 技术
tags:
  - nginx
typora-root-url: ./..\..\static
---

nginx 与大多数应用程序一样，记录了大量与客户端交互、系统事件和潜在错误相关的数据。然而，只有通过正确的配置、管理和分析才能充分发挥这些数据的潜力。

## 找到日志

与大多数 Web 服务器一样，Nginx 仔细地将其活动记录在两个不同的日志文件中：

- `/var/log/nginx/access.log` ：此文件记录每个传入请求，捕获关键详细信息，例如客户端的 IP 地址、请求的时间戳、请求的资源 (URI)、响应的 HTTP 状态代码以及客户端的用户代理（浏览器和操作）系统）。
- `/var/log/nginx/error.log` :  该文件充当诊断工具，记录请求处理和其他 Nginx 操作期间遇到的错误和问题。它记录时间戳、错误级别、错误消息和任何相关上下文等信息，以帮助进行故障排除。

### 定位不同环境下的Nginx日志

#### Linux 发行版

在大多数 Linux 发行版中，Nginx 日志文件通常位于`/var/log/nginx/`目录中。您会发现它们分别名为`access.log`和`error.log`。

如果您在默认位置找不到日志文件，则需要检查特定的 Nginx 配置。首先确定 Nginx 配置文件的位置（通常是`/etc/nginx/nginx.conf` ）：

```shell
nginx -t
```

 如果配置文件有效，此命令应输出配置文件的位置：

```shell
## output
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

打开配置文件并查找`error_log`和`access_log`指令以查明它们各自的位置。

`/etc/nginx/nginx.conf`

```shell
error_log /var/log/nginx/error.log

http {
  . . .
  access_log  /var/log/nginx/access.log;
  . . .
}
```

默认情况下，Nginx 全局应用`error_log`指令，而`access_log`通常放置在`http`块中。

#### Docker容器

由于 Docker 容器是短暂的，因此直接在容器中存储日志是不切实际的。官方 Nginx Docker 镜像通过创建从`/var/log/nginx/access.log`和`/var/log/nginx/error.log`到容器的标准输出 ( `/dev/stdout` ) 和标准错误 ( `/dev/stderr`的符号链接来解决这个问题`/dev/stderr` ) 分别流。这使得[Docker 的日志记录机制](https://betterstack.com/community/guides/logging/how-to-start-logging-with-docker/)能够收集和管理日志。

您可以[在 Dockerfile 中找到相关行](https://github.com/nginxinc/docker-nginx/blob/a6f7d140744f8b15ff4314b8718b3f022efc7f43/mainline/debian/Dockerfile#L105-L107)：

`mainline/debian/Dockerfile`

```dockerfile
. . .
ln -sf /dev/stdout /var/log/nginx/access.log \
&& ln -sf /dev/stderr /var/log/nginx/error.log \
```

nginx 容器运行后，访问相应的地址会生成一些日志，使用 `docker logs` 命令可以查看相应的日志，其中混合了访问日志和错误日志。

要仅查看访问日志，请将标准错误重定向到`/dev/null` ：

```shell
docker logs -f nginx-server 2>/dev/null
```

同样，要仅查看错误日志，请将标准输出重定向到`/dev/null` ：

```shell
docker logs -f nginx-server 1>/dev/null
```

## 配置 Nginx 访问日志

默认情况下，Nginx 访问日志以 combined 格式生成，无特殊说明时，该格式的定义为：

```nginx
'$remote_addr - $remote_user [$time_local] ' '"$request" $status $body_bytes_sent ' '"$http_referer" "$http_user_agent" "$http_x_forwarded_for"';
```

此配置会生成类似于以下内容的访问日志条目：

```shell
172.17.0.1 - - [06/Aug/2024:16:37:59 +0000] "GET / HTTP/1.1" 200 615 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0" "-"
```

![Nginx access logs combined format explained](/imgs/Nginx/image-20241113143459334.png)

- `172.17.0.1` ：提出请求的客户端的IP地址。
- `-`：如果使用身份验证，则这是经过身份验证的用户名；否则，它是连字符 (-)。
- `[06/Aug/2024:16:37:59 +0000]` ：处理请求的本地时间。
- `"GET / HTTP/1.1"` ：请求方法、路径和HTTP协议版本。
- `200` ：返回给客户端的HTTP状态码。
- `615` ：响应正文的大小（以字节为单位）。
- `"-"` ：引用页面的 URL（如果有）。
- `"Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0"` : 客户端提供的浏览器和操作系统信息。
- `"-"` ：如果请求通过代理，该变量包含原始客户端IP地址。

您可以使用主 Nginx 配置文件 ( `/etc/nginx/nginx.conf` ) 中的 log_format 指令或`/etc/nginx/sites-enabled`中特定于主机的配置中的`log_format`指令来定制访问日志格式。

如果 Nginx 直接在您的主机上运行，您可以相应地编辑相关文件。对于 Docker 实例，执行以下命令从`nginx`映像中提取 Nginx 配置文件并将其保存到主机上的`nginx.conf`文件中：

```shell
docker run --rm --entrypoint=cat nginx /etc/nginx/nginx.conf > nginx.conf
```

准备好再次启动容器后，请确保将修改后的文件从主机挂载到容器内的`/etc/nginx/nginx.conf` 

```shell
docker run --name nginx-server -v ./nginx.conf:/etc/nginx/nginx.conf:ro -d nginx
```

### 自定义访问日志格式

可以使用 `log_format` 指令来自定义访问日志的格式：

```nginx
log_format <name> '<formatting_variables>';
```

你需要做的是设置一个格式名，然后使用提供的[核心变量](https://nginx.org/en/docs/http/ngx_http_core_module.html#variables)和[日志变量](https://nginx.org/en/docs/http/ngx_http_log_module.html#log_format)设置日志的结构，下面是它的示例：

```nginx
. . .
http {
    . . .
    log_format custom '$remote_addr - $remote_user [$time_local]  $status '
                  '"$host" "$request" $body_bytes_sent "$http_referer" '
                  '"$http_user_agent" "$http_x_forwarded_for"';
    . . .
}
```

要应用此格式，只需修改配置文件中的 `access_log` 指令：

```nginx
access_log /var/log/nginx/access.log custom;
```

### 设置记录日志的条件

大流量下，`Nginx` 的访问日志可能变得相当大，条件日志记录允许您根据特定条件有选择地过滤日志条目，以减少日志量并提高性能。

语法如下：

```shell
access_log /path/to/access.log <log_format> if=<condition>;
```

`<condition>`是 Nginx 对每个请求进行计算的布尔表达式。如果计算结果为`true` ，则写入日志条目；否则，它会被跳过。

以下示例演示如何从访问日志中排除成功 (2xx) 和重定向 (3xx) 状态代码的日志：

```nginx
http {
    map $status $loggable {
        ~^[23]  0;  # Match 2xx and 3xx status codes
        default 1;  # Log everything else
    }

    access_log /var/log/nginx/access.log combined if=$loggable;
}
```

一些实际应用：

- 仅记录错误响应（4xx 和 5xx）以进行故障排除。
- 排除已知为机器人的特定用户代理或 IP 地址。
- 仅记录对应用程序特定部分的请求。
- 记录一定百分比的请求以[降低记录成本，](https://betterstack.com/community/guides/logging/log-sampling/)同时仍然捕获代表性样本（[请参阅此处了解一些采样技术](https://www.f5.com/company/blog/nginx/sampling-requests-with-nginx-conditional-logging)）。

### 禁用访问日志

如果您已经通过 Web 应用程序收集请求日志，则可能需要使用特殊的`off`值或重定向到`/dev/null`来禁用 Nginx 访问日志。

```nginx
access_log off;
access_log /dev/null;
```

## 构建Nginx访问日志

在云原生分布式系统和微服务领域，[结构化日志记录](https://betterstack.com/community/guides/logging/structured-logging/)因其相对于传统纯文本日志的众多优势而获得了巨大的关注。

例如， [Caddy](https://betterstack.com/community/guides/web-servers/caddy/) （Nginx 的替代品）会生成如下所示的访问日志：

```json
{
    "level": "info",
    "ts": 1646861401.5241024,
    "logger": "http.log.access",
    "message": "handled request",
    "request": {
        "remote_ip": "127.0.0.1",
        "remote_port": "41342",
        "client_ip": "127.0.0.1",
        "proto": "HTTP/2.0",
        "method": "GET",
        "host": "localhost",
        "uri": "/",
        "headers": {
            "User-Agent": ["curl/7.82.0"],
            "Accept": ["*/*"],
            "Accept-Encoding": ["gzip, deflate, br"],
        },
        "tls": {
            "resumed": false,
            "version": 772,
            "cipher_suite": 4865,
            "proto": "h2",
            "server_name": "example.com"
        }
    },
    "bytes_read": 0,
    "user_id": "",
    "duration": 0.000929675,
    "size": 10900,
    "status": 200,
    "resp_headers": {
        "Server": ["Caddy"],
        "Content-Encoding": ["gzip"],
        "Content-Type": ["text/html; charset=utf-8"],
        "Vary": ["Accept-Encoding"]
    }
}
```

让我们探讨一下如何将 Nginx 访问日志带入这个现代时代。

虽然 Nginx 本身并不生成 JSON 日志，但您可以用`log_format`指令与`escape=json`参数（确保正确转义 JSON 中无效的字符）来实现此目的。

```nginx
. . .
http {
    . . .
    log_format custom_json escape=json '{'
        '"level":"info",'
        '"ts": "$time_iso8601",'
        '"message": "handled request $request_method $request_uri",'
        '"request": {'
            '"id": "$http_x_request_id",'
            '"remote_ip": "$remote_addr",'
            '"remote_port": "$remote_port",'
            '"protocol": "$server_protocol",'
            '"method": "$request_method",'
            '"host": "$host",'
            '"uri": "$request_uri",'
            '"headers": {'
                '"user-agent": "$http_user_agent",'
                '"accept": "$http_accept",'
                '"accept-encoding": "$http_accept_encoding",'
                '"traceparent": "$http_traceparent",'
                '"tracestate": "$http_tracestate"'
            '}'
        '},'
        '"bytes_read": $request_length,'
        '"duration_msecs": $request_time,'
        '"size": $bytes_sent,'
        '"status": $status,'
        '"resp_headers": {'
          '"content_length": "$sent_http_content_length",'
          '"content_type": "$sent_http_content_type"'
        '}'
    '}';

    access_log /var/log/nginx/access.log custom_json;
    . . .
}
```

在此配置中：

- 我们定义了一种名为`custom_json`的新日志格式，并使用`escape=json`启用 JSON 转义

- 在 JSON 结构中，我们捕获各种信息：
  - 基本信息：例如日志级别、时间戳 (ts) 和消息。
  - 请求信息：嵌套在请求对象下的详细请求信息，包括使用`$http_<header_name>`的标头。
  - 指标信息：读取字节数、响应时间和响应大小等指标。
  - 响应信息：如状态、特定响应标头像`$sent_http_<header_name>`

- 最后，将`custom_json`格式应用于访问日志。

保存配置并重新启动 Nginx 后，使用一些虚构的[分布式跟踪标头发](https://betterstack.com/community/guides/observability/distributed-tracing/)出请求：

```shell
curl -v -H "traceparent: 00-0af7651916cd43dd8448eb211c80319c-b7ad6b7169203331-01" \
     -H "tracestate: rojo=00f067aa0ba902b7" \
     -H "X-Request-Id: f45a82a7-7066-40d4-981d-145952c290f8" \
     http://localhost
```

您将观察到干净的 JSON 格式的新访问日志条目：

```json
{
  "level": "info",
  "ts": "2024-08-07T11:57:31+00:00",
  "message": "handled request GET /",
  "request": {
    "id": "f45a82a7-7066-40d4-981d-145952c290f8",
    "remote_ip": "172.17.0.1",
    "remote_port": "39638",
    "protocol": "HTTP/1.1",
    "method": "GET",
    "host": "localhost",
    "uri": "/",
    "headers": {
      "user-agent": "curl/8.6.0",
      "accept": "*/*",
      "accept-encoding": "",
      "traceparent": "00-0af7651916cd43dd8448eb211c80319c-b7ad6b7169203331-01",
      "tracestate": "rojo=00f067aa0ba902b7"
    }
  },
  "bytes_read": 229,
  "duration_msecs": 0.000,
  "size": 853,
  "status": 200,
  "resp_headers": {
    "content_length": "615",
    "content_type": "text/html"
  }
}
```

## 配置Nginx错误日志

Nginx 错误日志是诊断和解决 Web 服务器问题的重要工具。它捕获各种 Nginx 操作期间发生的错误、警告和其他重要事件。让我们探讨如何配置和管理这个宝贵的资源。

`error_log`指令控制 Nginx 的错误日志记录行为。它接受两个参数：日志文件的路径和日志的最低[严重级别](https://betterstack.com/community/guides/logging/log-levels-explained/)。

```shell
error_log /var/log/nginx/error.log <severity_level>;
```

严重级别按照从低到高分为以下几种：

- debug ：高度详细的消息，主要用于故障排除和开发。
- info ：有关服务器操作的一般信息消息。
- notice ：值得注意的事件，但不一定是错误。
- warn ：可能表明潜在问题的意外事件。
- error ：处理过程中遇到的实际错误。
- crit ：需要注意的关键情况。
- alert ：需要立即采取措施的错误。
- emerg ：导致系统无法使用的严重错误。

如果您没有在 Nginx 配置中显式配置错误严重性级别，您将看到`error`级别及其之上的所有级别（ `crit` 、 `alert`和`emerg` ）的消息。然而，官方 Nginx docker 镜像的默认级别设置为`notice` 。

### Nginx 错误日志格式

![Nginx error log format](/imgs/Nginx/image-20241113164234124.png)

Nginx 错误日志遵循为人类可读性和易于工具解析而设计的格式。一般格式为：

```nginx
YYYY/MM/DD HH:MM:SS [<severity_level>] <pid>#<tid>: *<cid> <message>
```

这里：

- `<pid>` ：进程 ID
- `<tid>` : 线程 ID
- `<cid>` ：连接 ID

示例：

```nginx
2024/08/07 17:41:58 [error] 29#29: *1 open() "/usr/share/nginx/html/make" failed (2: No such file or directory), client: 172.17.0.1, server: localhost, request: "GET /make HTTP/1.1", host: "localhost"
```

### 将错误记录到多个文件

与访问日志类似，您可以配置 Nginx 将错误记录到多个文件，即使具有不同的严重级别：

```shell
error_log /var/log/nginx/error.log info;
error_log /var/log/nginx/emerg_error.log emerg;
```

在此设置中，除调试级别消息之外的所有事件都将记录到`error.log`中，而紧急事件将记录到名为`emerg_error.log`的单独文件中。

### 禁用错误日志

如果您需要完全禁用 Nginx 错误日志（尽管通常不推荐），您可以将其重定向到`/dev/null` 。在撰写本文时似乎没有特殊的`off`值。

```shell
error_log /dev/null;
```

