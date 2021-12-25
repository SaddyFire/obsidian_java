最新更新时间：2020.05.26 [版本说明](https://pay.weixin.qq.com/wiki/doc/apiv3/release/chapter1_4.shtml)

商户Native支付下单接口，微信后台系统返回链接参数code\_url，商户后台系统将code\_url值生成二维码图片，用户使用微信客户端扫码后发起支付。

## 01 接口说明

**适用对象：** 直连商户

**请求URL：** https://api.mch.weixin.qq.com/v3/pay/transactions/native

**请求方式：** POST

path指该参数为路径参数

query指该参数需在请求URL传参

body指该参数需在请求JSON传参

## 02 请求参数

| 参数名 | 变量  | 类型\[长度限制\] | 必填  | 描述  |
| --- | --- | --- | --- | --- | --- |
| 应用ID | appid | string\[1,32\] | 是   | body 由微信生成的应用ID，全局唯一。请求基础下单接口时请注意APPID的应用属性，例如公众号场景下，需使用应用属性为公众号的APPID 示例值：wxd678efh567hg6787 |
| 直连商户号 | mchid | string\[1,32\] | 是   | body 直连商户的商户号，由微信支付生成并下发。 示例值：1230000109 |
| 商品描述 | description | string\[1,127\] | 是   | body 商品描述 示例值：Image形象店-深圳腾大-QQ公仔 |
| 商户订单号 | out\_trade\_no | string\[6,32\] | 是   | body 商户系统内部订单号，只能是数字、大小写字母_-*且在同一个商户号下唯一 示例值：1217752501201407033233368018 |
| 交易结束时间 | time_expire | string\[1,64\] | 否   | body 订单失效时间，遵循[rfc3339](https://datatracker.ietf.org/doc/html/rfc3339)标准格式，格式为YYYY-MM-DDTHH:mm:ss+TIMEZONE，YYYY-MM-DD表示年月日，T出现在字符串中，表示time元素的开头，HH:mm:ss表示时分秒，TIMEZONE表示时区（+08:00表示东八区时间，领先UTC 8小时，即北京时间）。例如：2015-05-20T13:29:35+08:00表示，北京时间2015年5月20日 13点29分35秒。 订单失效时间是针对订单号而言的，由于在请求支付的时候有一个必传参数prepay\_id只有两小时的有效期，所以在重入时间超过2小时的时候需要重新请求下单接口获取新的prepay\_id。其他详见时间规则。  建议：最短失效时间间隔大于1分钟 示例值：2018-06-08T10:34:56+08:00 |
| 附加数据 | attach | string\[1,128\] | 否   | body 附加数据，在查询API和支付通知中原样返回，可作为自定义参数使用，实际情况下只有支付完成状态才会返回该字段。 示例值：自定义数据 |     |
| 通知地址 | notify_url | string\[1,256\] | 是   | body 通知URL必须为直接可访问的URL，不允许携带查询串，要求必须为https地址。 格式：URL 示例值：\[https://www.weixin.qq.com/wxpay/pay.php |
| 订单优惠标记 | goods_tag | string\[1,32\] | 否   | body\]([https://www.weixin.qq.com/wxpay/pay.php订单优惠标记goods_tagstring\[1,32\]否body](https://www.weixin.qq.com/wxpay/pay.php%E8%AE%A2%E5%8D%95%E4%BC%98%E6%83%A0%E6%A0%87%E8%AE%B0goods_tagstring%5B1,32%5D%E5%90%A6body)) 订单优惠标记 示例值：WXG |
| \- 订单金额 | amount | object | 是   | body 订单金额信息 |

![[4be234a05e6741fc871dace71386b6fb.png]]

|     |     |     |     |     |
| --- | --- | --- | --- | --- |
| +优惠功能 | detail | object | 否   | body 优惠功能 |
| +场景信息 | scene_info | object | 否   | body 支付场景描述 |
| +结算信息 | settle_info | object | 否   | body 结算信息 |

#### 02_2 请求示例

- JSON

```json
 {
    "mchid": "1900006XXX",
    "out_trade_no": "native12177525012014070332333",
    "appid": "wxdace645e0bc2cXXX",
    "description": "Image形象店-深圳腾大-QQ公仔",
    "notify_url": "https://weixin.qq.com/",
    "amount": {
        "total": 1,
        "currency": "CNY"
    }
} 
```

## 03 返回参数

| 参数名 | 变量  | 类型\[长度限制\] | 必填  | 描述  |
| --- | --- | --- | --- | --- |
| 二维码链接 | code_url | string\[1,512\] | 是   | 此URL用于生成支付二维码，然后提供给用户扫码支付。<br>注意：code_url并非固定值，使用时按照URL格式转成二维码即可。<br>示例值：weixin://wxpay/bizpayurl/up?pr=NwY5Mz9&groupid=00 |

#### 03_2 返回示例

- 正常示例

```
 {
    "code_url": "weixin://wxpay/bizpayurl?pr=p4lpSuKzz"
} 
```



- 错误码[公共错误码](https://pay.weixin.qq.com/wiki/doc/apiv3/Share/error_code.shtml)

| 状态码 | 错误码 | 描述  | 解决方案 |
| --- | --- | --- | --- |
| 403 | TRADE_ERROR | 交易错误 | 因业务原因交易失败，请查看接口返回的详细信息 |
| 500 | SYSTEMERROR | 系统错误 | 系统异常，请用相同参数重新调用 |
| 401 | SIGN_ERROR | 签名错误 | 请检查签名参数和方法是否都符合签名算法要求 |
| 403 | RULELIMIT | 业务规则限制 | 因业务规则限制请求频率，请查看接口返回的详细信息 |
| 400 | PARAM_ERROR | 参数错误 | 请根据接口返回的详细信息检查请求参数 |
| 403 | OUT\_TRADE\_NO_USED | 商户订单号重复 | 请核实商户订单号是否重复提交 |
| 404 | ORDERNOTEXIST | 订单不存在 | 请检查订单是否发起过交易 |
| 400 | ORDER_CLOSED | 订单已关闭 | 当前订单已关闭，请重新下单 |
| 500 | OPENID_MISMATCH | openid和appid不匹配 | 请确认openid和appid是否匹配 |
| 403 | NOTENOUGH | 余额不足 | 用户账号余额不足，请用户充值或更换支付卡后再支付 |
| 403 | NOAUTH | 商户无权限 | 请商户前往申请此接口相关权限 |
| 400 | MCH\_NOT\_EXISTS | 商户号不存在 | 请检查商户号是否正确 |
| 500 | INVALID_TRANSACTIONID | 订单号非法 | 请检查微信支付订单号是否正确 |
| 400 | INVALID_REQUEST | 无效请求 | 请根据接口返回的详细信息检查 |
| 429 | FREQUENCY_LIMITED | 频率超限 | 请降低请求接口频率 |
| 500 | BANKERROR | 银行系统异常 | 银行系统异常，请用相同参数重新调用 |
| 400 | APPID\_MCHID\_NOT_MATCH | appid和mch_id不匹配 | 请确认appid和mch_id是否匹配 |
| 403 | ACCOUNTERROR | 账号异常 | 用户账号异常，无需更多操作 |