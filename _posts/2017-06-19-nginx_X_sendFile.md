---
layout:     post
title:      Nginx+X_sendFile下载文件
subtitle:   X_sendFile处理方式
date:       2017-06-19
author:     Zq
header-img: img/post-bg-os-metro.jpg
catalog: true
tags:
    - Nginx
    - PHP
---


## 前言

由于之前在公司收集了用户行为的日志，其他协作部门后续希望对这些日志进行处理分析，需要下载这些日志文件，由PHP来实现下载就给服务器带来了没有必要的开销，所以使用了此方案。


## 正文

因为日志文件体量很大，用PHP搞，意味着我们的脚本必须从磁盘读取文件，该文件通过输出缓冲区，刷新到Web服务器并在发送到客户端之前进行处理，会消耗大量的内存，时间过长会导致php对应的interface超时，还要考虑下载恢复、缓存、线程I/O，种种原因，后实施方案为，由php校验基本的权限，及相关的身份验证逻辑判断，下载的工作交由更加可以胜任的nginx来处理，一举两得。
### X_sendFile

X-Sendfile是一项功能，允许Web应用程序将文件请求重定向到Web服务器，然后Web服务器处理请求，这样就无需执行读取文件和将其发送给用户等任务，Web服务器将加载配置指定的文件并将其发送给用户。
X-Sendfile是一个特殊的标头选项，它告诉Web服务器忽略响应的内容并将其替换为X-Sendfile标头中指定的文件。
当Web服务器遇到此类标头时，它将丢弃所有输出并使用Web服务器内部发送该标头指定的文件，包括缓存标头和下载恢复等所有优化。

X-SendFile的缺点是您无法控制传输机制。例如如果你希望在完成文件下载后执行某些操作，比如只允许用户下载文件一次，这个 X-Sendfile 是没法做到的，因为后台的 php 脚本并不知道下载是否成功。

| SENDFILE 头 | 使用的 WEB 器 |
| ------ | ------ | ------ |
| X-Sendfile | Apache, Lighttpd v1.5, Cherokee |
| X-LIGHTTPD-send-file	| Lighttpd v1.4 |
| X-Accel-Redirect	| Nginx, Cherokee |


此功能在此处以标准格式记录：X-Accel

Nginx 默认支持该特性，不需要加载额外的模块。只是实现有些不同，需要发送的 HTTP 头为 X-Accel-Redirect。
nginx配置：

```
location /protected/ {
 internal;
 root  /some/path;
}
```
*internal表示这个路径只能在 Nginx 内部访问，不能用浏览器直接访问防止未授权的下载。*


### PHP端请求

php中的header（在web server为nginx的情况下）：

```
header('Content-Description: Gzip-File Download');
header('Content-Type: application/octet-stream');
header('Content-Disposition: attachment; filename='.'"'.basename($file_path).'"');
//让Xsendfile发送文件
header('X-Accel-Redirect: /filepath/'.$file_path);
exit;
```
*`X-Accel-Limit-Rate`参数可以控制限速*

当web server检测到这个header头的时候，忽略后端php程序所有的输出，而使用自身的组件（包括 缓存头 和 断点重连 等优化）机制将文件发送给用户。*注：即使文件在 .htaccess 保护下禁止访问，也会被下载。*


### 参考：

- [X-Sendfile - serve large static files efficiently from web applications](https://www.yiiframework.com/wiki/129/x-sendfile-serve-large-static-files-efficiently-from-web-applications)

- [XSendfile nginx](https://www.nginx.com/resources/wiki/start/topics/examples/xsendfile/)


