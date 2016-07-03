---
layout: post
title: "HTTP Parser代码解析"
description: ""
category: 开源系统
tags: [HTTP]
---
{% include JB/setup %}

HTTP Parser简介
---

HTTP Parser是一个优秀的HTTP解析器。使用C语言编写，可以使用在对HTTP消息解析有性能要求的应用中。不需要任何的依赖程序，每个HTTP流只需要40字节的存储数据。HTTP Parser可以同时解析HTTP消息中的各个元素，例如：
  
  * Header fields and values
  * Content-Length
  * Request method
  * Response status code
  * Transfer-Encoding
  * HTTP version
  * Request URL
  * Message body


同时也支持Upgrade头的处理。对于URL，HTTP Parser也提供了一个简单的解析工具，http_parser_url，可以在没有内存复制的情况下将URL中的不同部分解析出来，十分的高效。

***

HTTP Parser的是什么，怎么来的，给谁用

HTTP Parser在github的地址
https://github.com/nodejs/http-parser
是nodejs的一个子项目，源自nginx
只有http_parser.h、http_parser.c两个文件。
另外提供了test.c和bench.c，提供简单的测试和程序范例
在contrib中，提供了http_parser_url的使用范例

Python Binding
https://github.com/benoitc/http-parser

HTTP Parser的特点是小、快、没有依赖、使用简单、功能完备

***

如何使用HTTP Parser
---
HTTP Parser有两个核心的结构体http_parser和http_parser_settings，通过定义这两个结构之后，调用

***

HTTP Parser的主要工作流程
---

***

http_parser和http_parser_settings
---
```c
struct http_parser {
  /** PRIVATE **/
  unsigned int type : 2;         /* enum http_parser_type */
  unsigned int flags : 8;        /* F_* values from 'flags' enum; semi-public */
  unsigned int state : 7;        /* enum state from http_parser.c */
  unsigned int header_state : 7; /* enum header_state from http_parser.c */
  unsigned int index : 7;        /* index into current matcher */
  unsigned int lenient_http_headers : 1;

  uint32_t nread;          /* # bytes read in various scenarios */
  uint64_t content_length; /* # bytes in body (0 if no Content-Length header) */

  /** READ-ONLY **/
  unsigned short http_major;
  unsigned short http_minor;
  unsigned int status_code : 16; /* responses only */
  unsigned int method : 8;       /* requests only */
  unsigned int http_errno : 7;

  /* 1 = Upgrade header was present and the parser has exited because of that.
   * 0 = No upgrade header present.
   * Should be checked when http_parser_execute() returns in addition to
   * error checking.
   */
  unsigned int upgrade : 1;

  /** PUBLIC **/
  void *data; /* A pointer to get hook to the "connection" or "socket" object */
};


struct http_parser_settings {
  http_cb      on_message_begin;
  http_data_cb on_url;
  http_data_cb on_status;
  http_data_cb on_header_field;
  http_data_cb on_header_value;
  http_cb      on_headers_complete;
  http_data_cb on_body;
  http_cb      on_message_complete;
  /* When on_chunk_header is called, the current chunk length is stored
   * in parser->content_length.
   */
  http_cb      on_chunk_header;
  http_cb      on_chunk_complete;
};
```

***

http_parser_url
---

```c
struct http_parser_url {
  uint16_t field_set;           /* Bitmask of (1 << UF_*) values */
  uint16_t port;                /* Converted UF_PORT string */

  struct {
    uint16_t off;               /* Offset into buffer in which field starts */
    uint16_t len;               /* Length of run in buffer */
  } field_data[UF_MAX];
};
```

结束语
---
感谢大家，感谢CCTV

http://rootk.com/post/tutorial-for-http-parser.html
http://rootk.com/post/http-parser%E5%AE%9E%E9%99%85%E8%A7%A3%E6%9E%90%E8%BF%87%E7%A8%8B.html

