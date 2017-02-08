---
layout: post
title: PHP 发送 HTTP 请求
category: 技术
tags: PHP HTTP REQUEST
description: HTTP
---

### HTTP 请求头

```
GET /index.html HTTP/1.1
Host: first.app
Connection: keep-alive
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.71 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Encoding: gzip, deflate, sdch
Accept-Language: zh-CN,zh;q=0.8
Cookie: PHPSESSID=72r3lr1f3m3nd3qq2ss9p6gel4


```

### socket 方式

```php
<?php
$fp=fsockopen('first.app',80,$errno,$errstr,30); //打开连接
if(!$fp){
	die("连接失败，原因：$errstr ");
}

$http='';
$http.="GET /index.html HTTP/1.1\r\n";   //GET 请求页面
$http.="HOST:first.app\r\n";   //请求域名
$http.="Connection:close\r\n\r\n";  //关闭

fwrite($fp, $http,strlen($http));  //发送协议
$data='';
while (!feof($fp)) { // feof() 测试文件指针是否到了文件结束的位置
	$data.=fread($fp, 4096);  //读取信息 每次读取4K
}
fclose($fp); //关闭连接
var_dump($data);
?>

```

### curl 方式


```php
<?php

$ch = curl_init("first.app/index.html");
$fp = fopen("curl.txt", "w");
curl_setopt($ch, CURLOPT_FILE, $fp);
curl_setopt($ch, CURLOPT_HEADER, 0);

curl_exec($ch);
curl_close($ch);
fclose($fp);
?>

```



### file_get_contents 方式

```php
<?php
$content=file_get_contents('http://first.app/index.html');
var_dump($content);
?>
```

### 其他

```
curl_setopt($ch, CURLOPT_HEADER, false); //去除返回协议头
curl_setopt($ch, CURLOPT_REFERER, 'first.app/index.html'); //模拟

```

### 总结

curl 和 file_get_contents 方式底层原理还是 socket 方式，只是对 http 请求进行了封装。
file_get_contents 最简单， curl 对大量请求性能较好。

