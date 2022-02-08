# 一、SDK工具包
<br/>

网址:https://pay.weixin.qq.com/wiki/doc/apiv3/wechatpay/wechatpay6_0.shtml

- <ins>[wechatpay-apache-httpclient](https://github.com/wechatpay-apiv3/wechatpay-apache-httpclient)，适用于使用Apache HttpClient处理HTTP的Java开发者。</ins>
- [wechatpay-php](https://github.com/wechatpay-apiv3/wechatpay-php)（推荐）、[wechatpay-guzzle-middleware](https://github.com/wechatpay-apiv3/wechatpay-guzzle-middleware)，适用于PHP开发者。
- [wechatpay-go](https://github.com/wechatpay-apiv3/wechatpay-go)，适用于Go开发者

流程:

- GitHub 下载源码, 抽取(install)jar包
- 依赖

```xml
<dependency>
    <groupId>com.github.wechatpay-apiv3</groupId>
    <artifactId>wechatpay-apache-httpclient</artifactId>
    <version>0.3.0</version>
</dependency>
```

</br>

# 二、Native调起支付API
<br/>
最新更新时间：2020.04.29 \[版本说明\]

商户后台系统先调用微信支付的 **\[Native下单\]** 接口，微信后台系统返回链接参数code\_url，商户后台系统将code\_url值生成二维码图片，用户使用微信客户端扫码后发起支付。

#### 注意：

code_url有效期为2小时，过期后扫码不能再发起支付。

## 01 业务流程时序图

![[c6ba93b6896c47d08e9e5bc6c2d44bca.png]]

原生支付时序图

## 02 业务流程说明：

（1）商户后台系统根据用户选购的商品生成订单。 =>  OrderController.class\_submit()\_

（2）用户确认支付后调用微信支付 **【Native下单API】** 生成预支付交易；

（3）微信支付系统收到请求后生成预支付交易单，并返回交易会话的二维码链接code_url。

（4）商户后台系统根据返回的code_url生成二维码。

（5）用户打开微信“扫一扫”扫描二维码，微信客户端将扫码内容发送到微信支付系统。

（6）微信支付系统收到客户端请求，验证链接有效性后发起用户支付，要求用户授权。

（7）用户在微信客户端输入密码，确认支付后，微信客户端提交授权。

（8）微信支付系统根据用户授权完成支付交易。

（9）微信支付系统完成支付交易后给微信客户端返回交易结果，并将交易结果通过短信、微信消息提示用户。微信客户端展示支付交易结果页面。

（10）微信支付系统通过发送异步消息通知商户后台系统支付结果。商户后台系统需回复接收情况，通知微信后台系统不再发送该单的支付通知。

（11）未收到支付通知的情况，商户后台系统调用【**查询订单API**】。

（12）商户确认订单已支付后给用户发货。

## **03 流程分析**

- 提交微信生成支付二维码 详见Native调起支付API https://pay.weixin.qq.com/wiki/doc/apiv3/apis/chapter3\_4\_4.shtml
- 由流程图可知后台需要
- 1、生成订单
- 2、调统一下单API()
- 4、将链接生成二维码
- 10、异步通知商户支付结果() -> 告知支付通知接受情况
- 11、调用查询订单API(已在订单列表形成,可不做)
- 12、发货操作(略)

## 04 生成二维码规则

对应链接格式：weixin：//weixin://pay.weixin.qq.com/bizpayurl/up?pr=NwY5Mz9&groupid=00。请商户调用第三方库将code_url生成二维码图片。该模式链接较短，生成的二维码打印到结账小票上的识别率较高。

例如，将weixin：//weixin://pay.weixin.qq.com/bizpayurl/up?pr=NwY5Mz9&groupid=00 生成二维码见下图

![[b9cd2b6d8da94c62b4a6b2f8ed2a6cd6.png]]







