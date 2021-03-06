## basicauth

## bind

绑定覆盖服务器应该绑定的主机。通常，侦听器绑定到通配符主机。但是，您可以强制侦听器绑定到另一个主机名或IP。该指令只接收主机，而不接受端口。

请注意，绑定站点不一致可能会导致意想不到的后果。例如，如果同一个端口上的两个站点解析为127.0.0.1，其中只有一个站点配置了`bind 127.0.0.1`，那么只有一个站点是可访问的，因为另一个站点将绑定到端口而没有特定的主机;操作系统将选择更具体的匹配套接字。(虚拟主机不会在不同的监听器之间共享)

### 语法

```
bind host
```

- **host** 是要绑定到的主机名（或IP地址）

### 例子

要使您的 socket 只能访问本机，请绑定到IP 127.0.0.1（localhost）：

```
bind 127.0.0.1
```


## browse

## errors

错误允许您设置自定义错误页面并启用错误日志记录。

如果没有这个中间件，错误响应(HTTP状态> = 400)不会被记录，客户端会收到一条明文错误消息。
使用一个错误日志，将记录每个错误的文本，这样您就可以确定哪些错误，而不向客户公开这些细节。使用错误页面，您可以显示自定义错误消息并指示您的访问者该做什么。当您指定自定义错误页面时，将自动启用错误日志记录。

### 语法

```
errors [logfile]
```

- **logfile** 
是相对于当前工作目录创建（或附加到）的错误日志文件的路径。有关如何指定输出位置的更多详细信息，请参阅[日志位置](#destination)。默认是`stderr`。

要指定自定义错误页面，请打开一个代码块：

```
errors [logfile] {
	code     file
	rotate_size     mb
	rotate_age      days
	rotate_keep     count
	rotate_compress
}
```

- **code** 可以是HTTP状态代码（4xx，5xx或`*`默认错误页面）。

- **file** 是错误页面的静态HTML文件（相对于站点根目录）。

- **rotate_size** 是滚动之前日志文件必须达到的大小（以兆字节为单位）。

- **rotate_age** 是保留滚动日志文件的天数。

- **rotate_keep** 是要保留的最大滚动日志文件数; 旧的滚动日志文件被清除。

- **rotate_compress** 是压缩旋转日志文件的选项。gzip是唯一支持的格式。

<a name="destination"></a>
### 日志位置 {#destination}

日志位置可以是以下几种：

- 相对于当前工作目录的文件名

- **stdout** 或 **stderr** 写入控制台

- **visible** 将错误（包括完整堆栈跟踪（如果适用））写入响应中（除了本地调试之外，不推荐）

- **syslog** 写入本地系统日志（Windows下除外）

- **syslog://host[:port]** 通过UDP写入本地或远程syslog服务器

- **syslog+udp://host[:port]** 与上述相同

- **syslog+tcp://host[:port]** 通过TCP写入本地或远程syslog服务器

默认的日志位置是`stderr`

### 滚动日志

日志有可能填满磁盘。为了减轻这种情况，错误日志将根据此默认配置自动滚动：

```
rotate_size 100 # Rotate a log when it reaches 100 MB
rotate_age  14  # Keep rotated log files for 14 days
rotate_keep 10  # Keep at most 10 rotated log files
rotate_compress # Compress rotated log files in gzip format
```

您可以指定这些子目录来自定义滚动日志。

### 例子

将错误记录到error.log中：

```
errors
```

将错误记录到父目录中的自定义文件中：

```
errors ../error.log
```

记录错误并提供自定义错误页面：

```
errors {
	404 404.html # Not Found
	500 500.html # Internal Server Error
}
```

将错误记录到自定义日志文件并提供自定义错误页面：

```
errors ../error.log {
	404 404.html # Not Found
	500 500.html # Internal Server Error
}
```

定义默认，全部错误页面：

```
errors {
	* default_error.html
}
```

使客户端可以看到错误（仅用于调试）：

```
errors visible
```

自定义错误日志滚动：

```
errors {
	rotate_size 50  # Rotate after 50 MB
	rotate_age  90  # Keep rotated files for 90 days
	rotate_keep 20  # Keep at most 20 log files
	rotate_compress # Compress rotated log files in gzip format
}
```


## expvar

## ext

当 URL 的路径部分为空时，ext 可以让您的站点拥有干净的URL。
通过检查一个点(`.`)路径的最后一个元素，可以检测到URL中的扩展。

### 语法

```
ext extensions...
```
- extensions... 

一个空间分隔扩展列表(包括`.`)来尝试。将按所列的顺序尝试扩展。至少需要一个扩展。

### 例子

假设您有一个名为/contact.html的文件。你可以试着用它来服务 .html 

按顺序尝试.html，.htm和.php：

```
ext .html .htm .php
```
## fastcgi

fastcgi 代理请求到一个 FastCGI 服务器。虽然这个指令最常见的用途是为PHP站点提供服务，但它默认是一个通用的 FastCGI 代理。这个指令可以用不同的请求路径多次使用。

### 语法

```
fastcgi path endpoint [preset] {
	root     directory
	ext      extension
	split    splitval
	index    indexfile
	env      key value
	except   ignored_paths...
	upstream endpoint
	connect_timeout duration
	read_timeout    duration
	send_timeout    duration
}
```

- path 

在请求转发之前匹配的基本路径。

- endpoint

FastCGI 服务器的地址或 Unix socket 。

- preset

可选的预设名称（见下文）。使用预设时，不需要重复预设的单独设置

- root 

根指定 FastCGI 服务器使用的根目录，如果它不同于虚拟主机的根目录。这在FastCGI 服务器位于不同的服务器、chroot-jailed 和 容器中，则很有用。

- ext 

指定扩展名，如果请求URL具有该扩展名，则会将请求代理到FastCGI。


- split

指定如何分割URL;当它成为 PATH_INFO CGI 变量的一部分后，分割值就变成了第一部分的末尾和 URL 中的任何内容。

- index

指定文件未由URL指定时要尝试的默认文件

- env

为给定值设置了一个名为 key 的环境变量; ENV 属性可以使用多次，而值可以使用[请求占位符](/http-server/#placeholders)。

- except 

排除请求路径（可分隔），即使它与基本路径匹配，它也不受 fastcgi处理的影响。

- upstream 

指定了一个额外的后端来使用。将执行基本的负载平衡。这可以多次指定。



- connect_timeout 

允许连接到后端的时间。必须是持续时间值(例如: "10s" )。

- read_timeout

允许从后端读取响应的时间。必须是持续时间值。

- send_timeout

允许向后端发送请求的时间。必须是持续时间值。

### Presets 预设

预设是某种快速cgi配置的简写。这些预设是可用的:

php 简写

```
ext   .php
split .php
index index.php
```
您不需要指定预置的配置设置。但是，如果需要手动声明它们，则可以覆盖它的单独设置。


### Examples 例子

在 `127.0.0.1:9000` 处理 FastCGI 的所有请求：

```
fastcgi / 127.0.0.1:9000
```


将/ blog中的所有请求转发到php-fpm提供的PHP站点（如WordPress）：

```
fastcgi /blog/ 127.0.0.1:9000 php
```

使用自定义FastCGI配置：

```
fastcgi / 127.0.0.1:9001 {
	split .html
}
```
使用 PHP 预设，但覆盖ext属性：

```
fastcgi / 127.0.0.1:9001 php {
	ext .html
}
```
使用PHP预设，但 FastCGI 服务器正在基于[官方 Docker 映像](https://hub.docker.com/_/php/)（容器端口 9000 发布到 127.0.0.1:9001）的容器中运行：

```
fastcgi / 127.0.0.1:9001 php {
	root /var/www/html
}
```


## gzip

如果客户端支持，gzip 会启用 gzip 压缩。默认情况下，响应不会被 gzip 压缩。如果启用，默认设置将确保未压缩图像，视频和存档（已压缩）。

请注意，即使没有 gzip 指令，Caddy 也可以在.gz（gzip）或.br（brotli）压缩文件中使用，如果它们已经存在于磁盘上，并且客户机支持该编码。

### 语法

```
gzip
```

普通的gzip配置对于大多数情况都是足够用的，但是如果需要，您可以像这样：

```
gzip {
    ext        extensions...
    not        paths
    level      compression_level
    min_length min_bytes
}
```

- **extensions...** 是压缩文件扩展，用空格分隔的列表。支持通配符 `*` 匹配所有扩展。
- **paths**  是空格分隔的路径列表，不进行压缩.
- **compression_level**  是从1（最佳速度）到9（最佳压缩）的数字。默认为9。
- **min_bytes**  是在压缩发生之前所需的响应中的最小字节数。默认值不是最小长度。

### 例子

启用gzip压缩：

```
gzip
```

启用非常快速最小压缩，除了在 /images 和 /videos 文件夹（注意，但图像和视频不会被 gzip 压缩）

```
gzip {
    level 1
    not   /images /videos
}
```

## header

header 可以操纵响应头。

请注意，如果您希望从代理后端删除响应标头，则必须在[代理](#proxy)指令中执行此操作。

### 语法

```
header path name value
```

- **path** 是匹配的基本路径。

- **name** 是字段的名称。前缀用连字符（`-`）删除标题或加号（`+`）来追加而不是覆盖。

- **value** 是字段的值。也可以使用[占位符](/http-server/#placeholders)插入动态值。

此指令可以多次使用，也可以为同一路径分组多个自定义标题字段：

```
header path {
	name value
}
```

### 例子

所有页面的自定义 header：

```
header / X-Custom-Header "My value"
```

从标题中剥离“隐藏”字段：

```
header / -Hidden
```

特定路径的多个自定义标头，同时删除服务器字段：

```
header /api {
	Access-Control-Allow-Origin  *
	Access-Control-Allow-Methods "GET, POST, OPTIONS"
	-Server
}
```

向所有页面添加一些安全标头：

```
header / {
	# Enable HTTP Strict Transport Security (HSTS) to force clients to always
	# connect via HTTPS (do not use if only testing)
	Strict-Transport-Security "max-age=31536000;"
	# Enable cross-site filter (XSS) and tell browser to block detected attacks
	X-XSS-Protection "1; mode=block"
	# Prevent some browsers from MIME-sniffing a response away from the declared Content-Type
	X-Content-Type-Options "nosniff"
	# Disallow the site to be rendered within a frame (clickjacking protection)
	X-Frame-Options "DENY"
}
```



## import

## index

索引设置用于 “index” 文件的文件名列表。当请求目录路径而不是特定的文件时，将对目录进行检查，以检查现有的索引文件。将提供第一个匹配的文件名。

按照这个顺序，默认的索引文件是:

1. index.html
2. index.htm
3. index.txt
4. default.html
5. default.htm
6. default.txt

当使用 index 指令时，将不会附加此列表。

### 语法

```
index filenames...
```

- filenames...

以空格分隔的文件名作为索引的列表。至少需要一个名称。

### 例子

只使用 goaway.pn g和 easteregg.html 作为索引文件：

```
index goaway.png easteregg.html
```


## internal

## limits

## log

日志启用请求日志。请求日志也从一些白话语中被称为访问日志。

### 语法

没有参数，访问日志将以所有请求的通用日志格式写入access.log：

```
log
```

自定义日志文件位置：

```
log file
```

是相对于当前工作目录创建（或附加到）日志文件的路径。有关如何指定输出位置的更多详细信息，请参阅日志位置。默认是access.log。

要将此日志限制在某些请求或更改日志格式：

```
log path file [format]
```

- **path** 是要匹配的基本请求路径，以便记录。
- **file** 是相对于当前工作目录创建（或附加到）的日志文件。
- **format**  是要使用的日志格式; 默认为通用日志格式。

大型日志文件会自动滚动。您可以通过打开一个块来自定义滚动日志：

```
log path file [format] {
	rotate_size     mb
	rotate_age      days
	rotate_keep     count
	rotate_compress
}
```

- **rotate_size** 是滚动之前日志文件必须达到的大小（以兆字节为单位）。
- **rotate_age** 是保留旋转日志文件的天数。
- **rotate_keep** 是要保留的最大旋转日志文件数; 旧的旋转日志文件被修剪。
- **rotate_compress** 是压缩旋转日志文件的选项。gzip是唯一支持的格式。

### 日志格式

您可以使用任何[占位符](/http-server/#placeholders)值指定自定义日志格式。日志支持请求和响应占位符。


目前有两种预置格式。

**{common}（默认）**

```
{remote} - {user} [{when}] \"{method} {uri} {proto}\" {status} {size}
```

**{combined} - {common}追加**

```
\"{>Referer}\" \"{>User-Agent}\"
```

### 日志位置 

日志位置可以是以下几种：

- 相对于当前工作目录的文件名

- **stdout** 或 **stderr** 写入控制台

- **visible** 将错误（包括完整堆栈跟踪（如果适用））写入响应中（除了本地调试之外，不推荐）

- **syslog** 写入本地系统日志（Windows下除外）

- **syslog://host[:port]** 通过UDP写入本地或远程syslog服务器

- **syslog+udp://host[:port]** 与上述相同

- **syslog+tcp://host[:port]** 通过TCP写入本地或远程syslog服务器

默认的日志位置是`stderr`

### 滚动日志

日志有可能填满磁盘。为了减轻这种情况，错误日志将根据此默认配置自动滚动：

```
rotate_size 100 # Rotate a log when it reaches 100 MB
rotate_age  14  # Keep rotated log files for 14 days
rotate_keep 10  # Keep at most 10 rotated log files
rotate_compress # Compress rotated log files in gzip format
```

您可以指定这些子目录来自定义滚动日志。

### 例子

将所有请求记录到access.log：

```
log
```

将所有请求记录到stdout：

```
log stdout
```

自定义日志格式：

```
log / stdout "{proto} Request: {method} {path}"
```

预定格式：

```
log / stdout "{combined}"
```

滚动日志：

```
log requests.log {
	rotate_size 50  # Rotate after 50 MB
	rotate_age  90  # Keep rotated files for 90 days
	rotate_keep 20  # Keep at most 20 log files
	rotate_compress # Compress rotated log files in gzip format
}
```

## markdown

markdown 可以根据需要将 `markdown文件` 作为 HTML 页面服务。您可以指定整个自定义模板，或者仅在页面上使用 CSS 和 JavaScript 文件，以自定义的外观和行为。

### 语法

```
markdown [basepath] {
	ext         extensions...
	[css|js]    file
	template    [name] path
	templatedir defaultpath
}
```
- basepath 

是匹配的基本路径。如果请求 URL 没有以此路径前缀，则 Markdow n将不会激活。默认是站点的根目录。

- extensions...

以空格分隔的文件扩展名列表，用作 Markdown（默认为.md，.markdown 和.mdown）; 这与使用缺省文件扩展名的ext指令不同。

- css

表明下一个参数是在页面上使用的 css 文件

- js

指出下一个参数是包含在页面上的 JavaScript 文件。

- file

添加到页面的JS或CSS文件。


- template

定义了一个带有给定名称的模板在给定的路径上。要指定默认模板，请省略名称。Markdown 文件可以使用其前面的名称来选择模板。

- templatedir 

设置在列出模板时使用给定的 defaultpath 的默认路径。


您可以多次使用 js 和 css 参数来为默认模板添加更多的文件。如果您想接受默认值，应该完全省略花括号。

### 前端 (文档元数据)

Markdown 文件可能从前面的事情开始，这是一个关于文件的特殊格式的元数据块。例如，它可以描述用于呈现文件的 HTML 模板，或者定义标题标签的内容。前面的内容可以是 YAML、TOML 或 JSON 格式。

TOML 以 `+++` 开始和结束：

```
+++
template = "blog"
title = "Blog Homepage"
sitename = "A Caddy site"
+++
```

YAML 以 `---` 开始和结束：

```
---
template: blog
title: Blog Homepage
sitename: A Caddy site
---
```

JSOn 用 `{}` 包起来

```
{
	"template": "blog",
	"title": "Blog Homepage",
	"sitename": "A Caddy site"
}
```

前面的字段 "author", "copyright", "description" 和 "subject" 将用于`<meta>`在呈现的页面上创建标签。

### Markdown 模板

模板文件只是带有模板标签的 HTML 文件，称为 actions ，可以根据所提供的文件插入动态内容。元数据中定义的变量可以从`{{.Doc.variable}}`这样的模板中访问，其中“variable”是变量的名称。该变量`.Doc.body`保存 markdown 文件的主体。

这是一个简单的示例模板（设计）：

```
<!DOCTYPE html>
<html>
	<head>
		<title>{{.Doc.title}}</title>
	</head>
	<body>
		Welcome to {{.Doc.sitename}}!
		<br><br>
		{{.Doc.body}}
	</body>
</html>
```
除了这些模板操作之外，所有[标准 Caddy 模板操作](/http-server/#template-actions)都可以在 Markdown 模板中使用。一定要将您渲染的任何HTML(使用 HTML、js 和 urlquery 函数)进行过滤!

### 例子

在没有特殊格式的情况下，在/ blog中设置Markdown页面（假设.md是Markdown扩展名）：

```
markdown /blog
```

与上述相同，但使用自定义的 CSS 和 JS 文件：

```
markdown /blog {
	ext .md .txt
	css /css/blog.css
	js  /js/blog.js
}
```

使用自定义模板：

```
markdown /blog {
	template default.html
	template blog  blog.html
	template about about.html
}
```


## mime

mime在请求中基于文件扩展的响应设置内容类型。
通常情况下，通过嗅探内容，可以自动地检测到静态文件，但有时候是不可能的。如果您遇到的响应是错误的内容类型，或者是服务于静态文件以外的内容，您可以使用这个中间件来设置正确的内容类型。

### 语法

```
mime ext type
```


- **ext** 是要匹配的文件扩展名，包括`.`前缀。
- **type** 是Content-Type

如果您有很多MIME类型要设置，请打开一个代码块：

```
mime {
	ext type
}
```

每行定义一个MIME扩展类型对。您可以在mime块中具有所需的行数。

### 例子

自定义Flash文件的内容类型：

```
mime .swf application/x-shockwave-flash
```

对于多个文件：

```
mime {
	.swf application/x-shockwave-flash
	.pdf application/pdf
}
```


## pprof

pprof 在 /debug /pprof 中发布运行时分析数据。您可以在站点上访问 /debug /pprof，以获得可用端点的索引。

{{< note title="注意" >}}
这是一个调试工具。 某些请求（如收集执行跟踪）可能很慢。 如果您在现场网站上使用pprof，请考虑限制访问或仅暂时启用它。
{{< /note >}}

有关更多信息，请参阅[Go的pprof文档](https://golang.org/pkg/net/http/pprof/)并阅读[Profiling Go程序](https://blog.golang.org/profiling-go-programs)。

### 语法

```
pprof
```

### 例子

启用 pprof 

```
pprof
```

## proxy

proxy 方便了基本的反向代理和稳健的负载均衡器。该 proxy 支持多个后端和添加自定义标头。负载均衡功能包括多个策略，运行状况检查和故障转移。Caddy 还可以代理 WebSocket 连接。

该中间件添加了可以以[日志](#log)格式使用的[占位符](/http-server/#placeholders)：{upstream} - 请求被代理的上游主机的名称。

### 语法

在其最基本的形式中，简单的反向代理使用以下语法：

```
proxy from to
```
- from

是要被代理的请求的基本路径

- to

是要代理的目标端点（可能包括端口范围）

**但是，包括负载平衡在内的高级功能可以用扩展语法来使用：**

```
proxy from to... {
	policy name [value]
	fail_timeout duration
	max_fails integer
	max_conns integer
	try_duration duration
	try_interval duration
	health_check path
	health_check_port port
	health_check_interval interval_duration
	health_check_timeout timeout_duration
	header_upstream name value
	header_downstream name value
	keepalive number
	without prefix
	except ignored_paths...
	upstream to
	insecure_skip_verify
	preset
}
```
- from

是要被代理的请求的基本路径。

- to

是要代理的目的端点。需要至少一个，但可以指定多个。如果未指定方案（http / https），则使用http。Unix 套接字也可以通过前缀 “unix：”来使用。

- policy

使用负载均衡策略; 仅适用于多个后端。可以是随机的，least_conn，round_robin，first，ip_hash，uri_hash 或 header。如果选择 header ，还必须提供 header 名称。默认是随机的。

- fail_timeout

指定记录对后端的请求失败最长时间。超时>0 启用了请求失败计数，并且在失败的情况下需要在后台进行负载均衡。如果失败的请求数量累积到 max_fail s值，则主机将被视为已关闭，并且不会将请求路由到该主机，直到失败的请求开始被忘记为止。默认情况下，这是禁用（0s），意味着失败的请求将不会被记住，后端将始终被视为可用。必须是持续时间值（如“10s”或“1m”）。

- max_fails

在考虑后端关闭之前需要的 fail_timeout 中的失败请求数。如果 fail_timeout 为0，则不使用。必须至少为1，默认值为1。

- max_conns

每个后端的最大并发请求数。0表示无限制。达到极限时，其他请求将失败，并显示 Bad Gateway（502）。默认值为0。

- try_duration 

为每个请求选择可用的上游主机多长时间。默认情况下，此重试被禁用（“0s”）。客户端可能会挂起这么长时间，而代理尝试找到可用的上游主机。仅当对初始选择的上游主机的请求失败时才使用此值。

- try_interval

是在选择另一个上游主机来处理请求之间等待多长时间。默认值为250ms。只有在向上游主机的请求失败时才相关。请注意，使用 try_duration 将其设置为0可能会导致非常严格的循环，并且如果所有主机都停留，则可以占满CPU。

- health_check

将使用路径来检查每个后台的运行状况。如果后端返回200-399的状态码，则该后端被认为是健康的。如果没有，后端标记为 health_check_interval 不健康，并且请求不会被路由到它。如果未提供此选项，则禁用健康检查。

- health_check_port

将使用端口来执行运行状况检查，而不是为上游提供的端口。如果您使用内部端口进行调试，则健康检查端点会从公共视图中隐藏，这很有用。

- health_check_interval

指定不健康后端的每个健康检查之间的时间。默认间隔为30秒（“30秒”）。


- health_check_timeout

设置健康检查请求的最后期限。如果健康检查在 health_check_timeout 内没有响应，则健康状况检查被认为是失败的。默认值为60秒（“60s”）。


- header_upstream 

将文件头传递给后端。字段名称是 name，值是 value。多个标题可以多次指定此选项，也可以使用请求占位符插入动态值。默认情况下，现有的字段头将被替换，但您可以通过使用加号（+）前缀字段名称来添加或合并字段值。您可以通过使用减号（ - ）前缀标题名称来删除字段，并将该值留空。


- header_downstream

修改后端返回的响应头。它的工作方式与 header_upstream 相同。

- keepalive 

是保持打开到后端的最大空闲连接数。默认启用;设置为0，禁用 keepalives。在繁忙的服务器上设置更高的值。


- without

在将请求代理向上游之前，没有URL前缀。例如，请求/api /foo /api将会导致代理请求/foo。

- except

一个空格分隔的路径列表，以排除在代理之外。与 ignored_paths 匹配的请求将通过 thrued。

- upstream 

指定另一个后端。如果需要，它可以使用像":8080-8085"这样的端口范围。当有很多后端路由时，它经常被使用多次。

-insecure_skip_verify 

覆盖了后端TLS证书的验证，基本上通过HTTPS禁用安全功能。

- preset 

配置代理以满足某些条件的可选速记方式。请参阅下面的预设。


{{< note title="注意" >}}
为了在失败事件中执行适当的冗余负载均衡，必须设置fail_timeout 和 try_duration 值为>0。
{{< /note >}}

第一个选项之后的所有内容都是可选的，包括由大括号括起来的属性块。


### 预设

以下的预设有:

- websocket

指示此代理正在转发WebSocket连接。这是简写:

```
header_upstream Connection {>Connection}
header_upstream Upgrade {>Upgrade}
```
{{< note title="注意" >}}
HTTP/2 不支持协议升级。
{{< /note >}}

- transparent

从原始请求中传递主机信息，这是大多数后端应用程序所期望的。简写:
 
 ```
 header_upstream Host {host}
header_upstream X-Real-IP {remote}
header_upstream X-Forwarded-For {remote}
header_upstream X-Forwarded-Proto {scheme}
```

### 策略

有几种负载均衡策略可用:

- random (default)

随机选择后端。

- least_conn

选择后端与最少的活动连接。

- round_robin

以循环方式选择后端。

- frist

按照在Caddyfile中定义的顺序选择第一个可用的后端。

- ip_hash 

通过散列请求 IP 来选择后端，根据后端的总数均匀分布在哈希空间上。

- uri_hash

通过散列请求 URI 来选择后端，根据后端的总数均匀分布在哈希空间上

- header 

通过散列由策略名称后面的[value]指定的给定标题的值进行选择，基于总后端数量均匀分布在散列空间

### 例子

将/ api内的所有请求代理到后台系统：

```
proxy /api localhost:9005
```

负载均衡三个后端之间的所有请求（使用随机策略）：

```
proxy / web1.local:80 web2.local:90 web3.local:100
```

与上述相同，具有 header 关联性：

```
proxy / web1.local:80 web2.local:90 web3.local:100 {
	policy header X-My-Header
}
```

循环风格：

```
proxy / web1.local:80 web2.local:90 web3.local:100 {
	policy round_robin
}
```

使用健康检查和代理标头来传递主机名，IP和方案上游：

```
proxy / web1.local:80 web2.local:90 web3.local:100 {
	policy round_robin
	health_check /health
	transparent
}
```

代理WebSocket连接：

```
proxy /stream localhost:8080 {
	websocket
}
```

排除 /static 或 /robots.txt 的代理请求：

```
proxy / backend:1234 {
	except /static /robots.txt
}
```

## push

## redir

如果URL匹配指定的规则，redir 会向客户端发送 HTTP 重定向状态代码。你也可以给它定义条件。

### 语法

```
redir from to [code]
```

- from 

要匹配的请求路径（它必须完全匹配，除了/，这代表匹配全部）。

-to 

重定向到（可使用的路径[请求占位符](/http-server/#placeholders)）。



- code

用于响应的HTTP状态代码; 必须在[300-308]（不包括306.）范围内。也可以使用`meta`为浏览器发布元标记进行重定向。默认状态代码是301（永久重定向）。

要创建永久的“全部”重定向，请忽略 from 值：

```
redir to
```

如果您有很多重定向，请通过创建表来共享重定向代码：

```
redir [code] {
	from to [code]
}
```
每行定义一个重定向，并可以选择覆盖在`表`顶部定义的重定向代码。如果未指定重定向代码，则使用默认值。

一组重定向也可以是有条件的：

```
redir [code] {
	if    a cond b
	if_op [and|or]
	...
}
```

- if

指定重写条件。默认情况下，多个 if 需要同时满足[译者注：`与`] 。a 和 b 可以是任何字符串，可以使用请求占位符。 cond 是条件，可能的值在[rewrite](#rewrite)中可以看到 （也有一个if声明）。

if_op 指定 if 之间如何联系; 默认是and[译者注：and = `与` or = `或`]。


### 被保护的路径

默认情况下，重定向会精确匹配路径到您定义的精确位置。您可以在任何"to"参数中使用[可替换值](/http-server/#placeholders)（例如{uri} 或 {path}）来保留请求URL的路径或其他部分。仅支持请求占位符。

### 例子

当请求进入/resources/images/photo.jpg 时，使用HTTP 307（临时重定向）状态码重定向到/resources/images/drawing.jpg：

```
redir /resources/images/photo.jpg /resources/images/drawing.jpg 307
```

将所有请求重定向到https://newsite.com，同时保留请求 URI：

```
redir https://newsite.com{uri}
```

定义共享 307 状态代码的多个重定向，最后一个除外：

```
redir 307 {
	/foo     /info/foo
	/todo    /notes
	/api-dev /api       meta
}
```

只有转发协议是 HTTP 时重定向：

```
redir 301 {
	if {>X-Forwarded-Proto} is http
	/  https://{host}{uri}
}
```





### 语法


## request_id

## rewrite

rewrite 执行内部 URL 重写。这允许客户端请求一个资源，但实际上是另一个资源，而不需要 HTTP 重定向。rewrite 是客户不可见的。

有简单的 rewrite（快速）和复杂的 rewrite（较慢），但它们足够强大以适应大多数动态后端应用程序。

### 语法

```
rewrite from to
```

- from

匹配的确切路径

-to 

重写（资源与响应）的目标路径

高级用户可能会打开一个块并进行复杂的重写规则：

```
rewrite [basepath] {
	regexp pattern
	ext    extensions...
	if     a cond b
	if_op  [and|or]
	to     destinations...
}
```
- basepath 

与正则表达式改写前匹配的基本路径。默认为/。

- regexp（简写："r"）

将匹配给定正则表达式模式的路径 。 
> 超高负载服务器应避免使用正则表达式。

- extensions... 

包含或忽略的文件扩展名（带`.`）以空格分隔。前缀以扩展名 `!` 排除。正斜杠 `/` 符号匹配没有文件扩展名的路径。

- if 

指定重写条件。多个if之间默认为 and (`与`) 的关系。a 和 b 可以是任何字符串，可以使用[请求占位符](/http-server/#placeholders)。 `cond`是条件，可能的值如下所述。

- if_op 

指定 if 之间如何联系; 默认是and[译者注：and = `与` or = `或`]。

- destinations... 

一个或多个空格分隔的路径来重写，支持 [请求占位符](/http-server/#placeholders)以及编号的正则表达式捕获，如{1}，{2}等。重写将按顺序检查每个目标位置并重写存在的第一个目标规则。每一个都被作为一个文件进行检查，或者作为一个目录结束。如果没有其他目标规则，最后一个目标规则将作为默认值。

### if 条件

if关键字是描述的规则的强大方法。它需要的格式 `a cond b`，其中的值 `a` 和 `b` 被 `cond` 分隔开，形成一个条件。它可以是这样的:

- is = a等于b
- not = a不等于b
- has = a有b作为子串（b是a的子字符串）
- not_has = b不是a的子串
- starts_with = b是a的前缀
- not_starts_with = b不是a的前缀
- ends_with = b是a的后缀
- not_ends_with = b不是a的后缀
- match = `任何`匹配b，其中b是正则表达式
- not_match = a不匹配b，其中b是正则表达式

注意：作为一般规则，您可以cond通过使用前缀来对任何条件进行否定`not_`。

### 例子

将所有内容重写为 /index.php。（`rewrite / /index.php`只匹配/）

```
rewrite / {
	regexp .*
	to /index.php
}
```

当请求进入/mobile 时，实际上是 /mobile/index

```
rewrite /mobile /mobile/index
```

如果文件不是favicon.ico，并且它不是有效的文件或目录，请提供维护页面（如果存在），或者最后重写为index.php。

```
rewrite {
	if {file} not favicon.ico
	to {path} {path}/ /maintenance.html /index.php
}
```
如果用户代理包含"mobile"，且路径不是有效的文件/目录，rewrite 到移动索引页。

```
rewrite {
	if {>User-agent} has mobile
	to {path} {path}/ /mobile/index.php
}
```
用查询字符串 rewrite /app 到 /index 并且带`{1}`匹配`(.*)`。

```
rewrite /app {
	r  (.*)
	to /index?path={1}
}
```

将 /app/example 的请求重写到 /index.php?category=example.

```
rewrite /app {
	r  ^/(\w+)/?$
	to /index?category={1}
}
```


## root

root 指定站点的根目录。这在网站的根目录（/）与 Caddy 执行的目录不同时非常有用。

默认的root是当前的工作目录。相对的根路径是相对于当前工作目录。

### 语法

```
root path
```

- path 

站点根目录

### 例子

从 jake 的p ublic_html 文件夹中运行，而不是当前工作目录:

```
root /home/jake/public_html
```


## shutdown

## startup

启动在服务器开始时执行命令。这对于通过运行脚本或启动后台进程（如php-fpm）来准备服务站点非常有用。（另请参见 [shutdown](#shutdown)）

在启动时执行的每个命令都是阻塞的，除非您使用空格和`&`后缀命令，否则将导致命令在后台运行。命令的输出和错误分别转到`stdout`和`stderr`。没有stdin。

在`Caddyfile`中，命令只执行一次。

### 语法

```
startup command
```

- command

执行的命令; 其后可能是参数


### 例子

在开始监听之前启动php-fpm：

```
startup /etc/init.d/php-fpm start、
```

在 Windows 上，当命令路径包含空格时，可能需要使用引号：

```
startup "\"C:\Program Files\PHP\v7.0\php-cgi.exe\" -b 127.0.0.1:9123" &
```


## status

## templates

## timeouts

## tls

tls配置HTTPS连接。由于 HTTPS 已自动启用，因此该指令只能用于覆盖默认设置。如果有的话，要小心使用。

Caddy 支持 SNI(服务器名指示)，所以您可以在您的机器上的同一个端口上服务多个HTTPS站点。此外，Caddy还为所有合格证书实施 OCSP 封装。Caddy 还会自动轮换所有 TLS 会话密钥。

tls 指令将忽略被显式定义为 http:// 或在端口80上的站点。这允许您在一个与 HTTP 和 HTTPS 站点共享的服务器块中使用 tls 指令。

如果在启动服务器时并不是所有的主机名都不知道，那么您可以使用[按需TLS功能](/http-server/#on-demand)，它在 TLS 握手期间发出证书，而不是在启动时发出证书。

{{< note title="Caddy 对于密码套件，曲线，密钥类型和协议都有合理的默认值。" >}}
他们的选择和排序可能随着新版本而改变。你可能不需要自己改变它们。调整 TLS 配置需要自担风险。
{{< /note >}}

### 语法

```
tls off
```

禁用站点的 TLS 。不推荐，除非你知道自己在做什么。关闭 TLS，自动 HTTPS也被禁用，因此默认端口（2015）将不会更改。

```
tls email
```

- email

用于生成具有受信任CA的证书的电子邮件地址。通过在这里提供电子邮件，您将不会在运行 Caddy 时被提示。

虽然不需要使用上面的语法来启用 TLS，但是它允许您指定用于您的 CA 帐户的电子邮件地址，而不是提示一个或使用以前运行的另一个。

要使用自己的证书和秘钥：

```
tls cert key
```

- cert

证书文件。如果证书是由 CA 签名的，那么这个证书文件应该是一个合成证书:域名证书，然后是 CA 的证书(根证书通常不需要)。

- key

与证书文件匹配的私钥文件。

您可以多次使用此指令来指定多个证书和密钥对。

或者让 Caddy 在内存中生成并使用不受信任的自签名证书：

```
tls self_signed
```

上面的语法使用了 Caddy 的默认 TLS 设置，其中包含了您自己的证书和密钥或者自签名证书，在大多数情况下应该是足够的。
高级用户可以打开一个设置块，以获得更多的控制，可选择指定他们自己的证书和密钥:

```
tls [cert key] {
    ca        uri
    protocols min max
    ciphers   ciphers...
    curves    curves...
    clients   [request|require|verify_if_given] clientcas...
    load      dir
    max_certs limit
    key_type  type
    dns       provider
    alpn      protos...
    must_staple
}
```

- ca

指定与ACME兼容的证书颁发机构端点来请求证书。

- cert 和 key 

证书和密钥与上述相同。

- protocols

规定了支持的最小和最大协议版本。见下面的有效值。如果min和max相同，则只需要指定一次。

- ciphers

 一列空间分隔的密码，将被支持，覆盖默认值。如果您列出了任何一个，只有您指定的那些将被允许并且优先于给定的。请参阅下面的有效值。

- curves

以给定顺序支持的空格分隔曲线的列表，覆盖默认值。有效曲线如下。

- clents

在 TLS 客户端身份验证过程中用于验证的空间分离客户根 CA 的列表。如果使用，客户将被要求通过他们的浏览器提交他们的证书，这将会在客户端证书颁发机构的列表中得到验证。如果客户的证书没有由其中一个root ca 签名，则客户端将不被允许连接。注意，这个设置适用于整个监听器，而不仅仅是一个站点。您可以在客户端 CA 列表前使用一个关键字修改客户端身份验证的严格性:
- * **request**  仅要求客户端提供证书，但如果没有提供或提交无效的，则不会失败。
- * **require**  需要客户端证书，但不验证它。
- * **verify_if_given**  如果没有提交，verify_if_given 将不会失败，但拒绝所有未通过验证的内容。
- * **默认** 默认情况下，如果没有设置标志，但是找到了 CA 文件，则需要同时执行这两种操作:要求客户机证书并验证它们。


- load

从中加载证书和密钥的目录。整个目录及其子文件夹将被搜索到.pem文件。每个.pem 文件必须包含 PEM 编码的证书（链）和关键块，并联在一起。

- max_certs

启用按需的TLS，在第一次握手时获得证书，而不是在启动时。如果在 Caddyfile 中提供完整的主机名并在启动时知道，则不建议使用此方法。这个限制值限制了允许在这个站点上(在TLS握手期间)发出的证书数量。它一定是一个正整数。此值在流程退出后重新设置(通过重新加载保存)。

- key_type

是生成证书密钥时使用的密钥类型（仅适用于托管或TLS或自签名证书）。有效值为 rsa2048，rsa4096，rsa8192，p256和p384。默认是rsa2048。

- dns

是dns提供者的名称;指定它可以实现 [DNS 域名验证](/http-server/#dns-challenge)。请注意，您需要提供凭据才能正常工作。

- alpn

是用于应用层协议协商（ALPN）的空格分隔协议的列表。对于 HTTPS 服务器，可以通过此设置启用/禁用HTTP 版本。默认是 `h2 http/1.1`。

- must_staple

启用管理证书的必备程序。小心使用。

### 协议

按照优先级降序支持以下协议：

> tls1.2（默认最大值）
> tls1.1（默认最小值）
> TLS1.0

请注意，设置最小的协议版本过高将限制能够连接的客户端，但有利于更好的隐私。[译者注：由于安全性原因，不建议使用 tls2.0 和 tls3.0 协议版本]

支持的协议和默认协议版本可以随时更改。

### 密码套件

目前支持以下密码套件：

> ECDHE-ECDSA-AES256-GCM-SHA384
> ECDHE-RSA-AES256-GCM-SHA384
> ECDHE-ECDSA-AES128-GCM-SHA256
> ECDHE-RSA-AES128-GCM-SHA256
> ECDHE-ECDSA-WITH-CHACHA20-POLY1305
> ECDHE-RSA-WITH-CHACHA20-POLY1305
> ECDHE-RSA-AES256-CBC-SHA
> ECDHE-RSA-AES128-CBC-SHA
> ECDHE-ECDSA-AES256-CBC-SHA
> ECDHE-ECDSA-AES128-CBC-SHA
> RSA-AES256-CBC-SHA
> RSA-AES128-CBC-SHA
> ECDHE-RSA-3DES-EDE-CBC-SHA
> RSA-3DES-EDE-CBC-SHA

{{< warning title="注意" >}}
注意：出于安全考虑，HTTP/2 规范将拉黑超过275个密码套件。除非你知道你在做什么，否则最好接受默认的密码套件设置。
{{< /warning >}}

密码套件可以随时添加到Caddy或从Caddy中删除。类似地，默认密码套件可以随时更改。

### 曲线

EC密码套件支持以下曲线：

> X25519
> P256
> P384
> P521

### 例子

记住，默认情况下启用TLS，并且通常不需要此指令！这些示例适用于手动管理证书或需要自定义设置的高级用户。

使用 HTTPS 使用自定义证书和私钥：

```
tls ../cert.pem ../key.pem
```

根据需要在TLS握手期间获得证书，限制10个新证书：

```
tls {
    max_certs 10
}
```

加载所有在 /www/certificates 中发现的pem证书和密钥:

```
tls {
    load /www/certificates
}
```

在内存中提供具有自签名证书的站点（浏览器不受信任，但方便本地开发）：

```
tls self_signed
```


## websocket

websocket 便于 websocket 服务器/代理。当创建新的WebSocket连接时，执行命令，并且Caddy中继客户端与该命令的连接。

只要从stdin输入并写入stdout，任何命令都可以执行，因为它将与WebSocket客户端进行通信。该命令不需要知道它正在与Web Socket客户端通信; 只需使用stdin和stdout。

客户端连接时，Caddy不会保留后端进程的活动。开发人员有责任确保程序在客户端准备关闭连接或准备终止之前终止。

{{< note title="注意" >}}
HTTP/2 不支持协议升级，因此您可能不得不禁用 HTTP/2，以便成功地在安全连接上使用该指令。
{{< /note >}}

### 语法

```
websocket [path] command
```

- path

与请求URL匹配的基本路径

- command

执行的命令


如果省略path，则假定默认路径为 `/`（表示所有请求）。

### 例子

简单的WebSockets echo服务器：

```
websocket /echo cat
```
