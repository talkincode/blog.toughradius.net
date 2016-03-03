title: ToughRADIUS API v1 发布
date: 2016-03-03 12:18:51
tags: release
---

ToughRADIUS API v1 发布了。

ToughRADIUS API要解决的三个问题：

- **业务功能扩展**：ToughRADIUS V2版本提供了一个精简的管理系统，但是对于一些客户比较复杂的业务需求会显得不够用，利用API可以实现业务功能的扩展，通过已有CRM等管理系统与ToughRADIUS实现对接。

- **自助服务系统定制**：ToughRADIUS V2版本移除了自助服务模块，我们希望把这个模块提供給第三方来实现。

- ** 移动终端APP开发**：对于ToughRADIUS V2版本，无论是管理，自服务都是可以扩展到移动终端的，通过API可以支持IOS，Android，微信公众号开发扩展。

## 协议说明

- 协议采用基于http的接口调用方式。
- 消息发起方：第三方系统。
- 消息接受方：TOUGHRADIUS系统。
- 接口请求方法：HTTP GET 或者 POST，支持标准http1.1协议。
- 消息报文编码：UTF-8，注意对参数进行 urlencode 编码
- 接口鉴权：MD5签名，`md5(共享密钥+排序的参数值组合字符串)`,签名校验是双向的，消息接收方对请求消息进行签名验证，同时需对结果进行签名返回，消息发起方对响应结果的签名进行验证。
- 当前接口版本：v1


完整文档请访问：[《ToughRADIUS 接口文档》](http://docs.toughradius.net/toughradius_v2/api_docs.html)

> 随着ToughRADIUS版本的更新完善，API接口内容也会持续完善。