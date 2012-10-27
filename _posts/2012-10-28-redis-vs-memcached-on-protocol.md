---
layout: post
title: "Redis与Memcached的协议比较"
description: ""
category: Key Value Store
tags: [redis, memcached, NoSQL]
---
{% include JB/setup %}

<h1>Redis Protocol</h1>
<h2>Request</h2>
	*<参数的个数> CR LF
	$<第一个参数的byte数> CR LF
	<参数的数据> CR LF
	...
	$<第N个参数的byte数> CR LF
	<参数的数据> CR LF

例子如下:

	*3
	$3
	SET
	$5
	mykey
	$7
	myvalue

其字符串的格式如下, 可以看出该查询的每个字节的值都是人可以直接读出的.
	*3\r\n$3\r\nSET\r\n$5\r\nmykey\r\n$7\r\nmyvalue\r\n


<h2>Response</h2>
Redis对请求命令有多种响应回复. Redis的客户端可以通过检查响应回复的第一个byte来判断回复的类型:

	* "+"代表单行回复
	* "-"代表错误消息, 也是单行
	* ":"代表回复的消息为整数
	* "$"代表回复的内容是块回复
	* "*"代表多块回复的前导
