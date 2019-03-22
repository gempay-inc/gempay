# GEM 充提系统接口V2.0  2019-03-21

---
### 充提接口域名

#### V1 旧版接口地址
##### 测试地址：`https://{DOMAIN}/stg`
##### 生产地址：`https://{DOMAIN}`

#### V2 新版接口地址，新版接口地址需要配合商户后台使用
##### 测试地址：`https://{DOMAIN}/stg/v2`
##### 生产地址：`https://{DOMAIN}/v2`

##### 实际地址请联系接口人获取


#### V2 新功能简述：
* 加入标签功能。在充值流程中，将原有的固定用户层级模式，转化成可任意配置的标签功能。一个用户可以支持单个或多个标签。支持商户后台自行设置标签。
* 在标签的基础上，支持商户后台自行开启，关闭充值渠道，以及调整某一渠道开启的网关。
* 在标签的基础上，支持商户后台在总后台限额的基础上，自行调整充值限额。
* 提现分流。加入提现分流操作，平台发起提现后，统一进入商户后台，在商户后台进行，自动或者人工的分流
* 提现优先级。商户后台可以自行设置提现优先级，可以设置对某几个渠道进行优先出款

#### 如何从V1升级到V2：
* 修改全局请求地址，统一加入 v2
* 在商户后台自行设置标签
* 充值接口请求参数中，`rank`参数变为必填，并且`rank`参数支持拼接，例如变为`rank=new-1-high`，则同时传入三个标签 ，`new`，`1`，`high`（注意，支持单标签，例如 `rank=1`）
* 操作完以上三步后，即可正常使用接口，商户后台新增的其他功能可以后续研究使用

### 常量介绍
##### `{merchants}`为商户名
##### `{KEY}`为加密密钥,32位，如果收到的不是32位，实际使用保留前32位
##### `{VECTOR}`为加密向量，16位，如果收到的不是16位，实际使用保留前16位
##### 实际地址请联系接口人获取

---

## V1接口

---

### 充值接口

#### 查询当前可用充值方式

方法

`GET`

路径

`/deposit/{merchants}/methods`

请求数据

| 参数名称 | 描述 | 是否必填 | 案例 |
| --- | --- | --- | --- |
| `rank` | 用户分层 | 否 | 2 |

返回数据，例子

```json
{
    "ok": true,
    "data": [
        {
            "gateway": "banks",
            "name": "网银快捷",
            "limits": {
                "min": 2,
                "max": 100
            },
            "banks": [
                {
                    "code": "ABOC",
                    "name": "农业银行"
                }
            ]
        },
        {
            "gateway": "unionpay",
            "name": "银联快捷",
            "limits": {
                "min": 2,
                "max": 100
            }
        },
        {
            "gateway": "mobile_weixin_fixed",
            "name": "移动微信定额",
            "options": [100, 200, 300]
        }
    ]
}
```

***提示***
1. 当 `gateway` 是 `banks` 时有 `banks` 字段列出当前支持的所有可充值银行
2. 当 `gateway` 是 `*_fixed` 时有 `options` 字段列出当前支持的所有固定可充值金额

---

#### 查询当前可用充值方式(移动端)

方法

`GET`

路径

`/deposit/{merchants}/mobile/methods`

请求数据

| 参数名称 | 描述 | 是否必填 | 案例 |
| --- | --- | --- | --- |
| `rank` | 用户分层 | 否 | 2 |


返回数据

```json
{
    "ok": true,
    "data": [
        {
            "gateway": "mobile_unionpay",
            "name": "银联移动",
            "limits": [], // 如果为空，请联系后台管理员配置
            "tips": null
        },
        {
            "gateway": "mobile_qq",
            "name": "QQ移动",
            "limits": {
                "min": 10,
                "max": 10000
            },
            "tips": null
        },
        {
            "gateway": "mobile_weixin",
            "name": "微信移动",
            "limits": {
                "min": 10,
                "max": 10000
            },
            "tips": null
        }
    ]
}
```

---

#### 发起充值请求

方法

`POST`

路径

`/deposit/{merchants}/forward`

请求数据

| 参数名称 | 描述 | 是否必填 | 案列 |
| --- | --- | --- |--- |
| `order_no` | 商户订单号 | 是 |BBMM1111113286 |
| `gateway` | 充值网关，如 `alipay`, `banks` 等 | 是 |banks |
| `amount` | 充值金额，单位元保留两位小数，如 100.00，使用支付宝网关时小数点后两位必须为 .00 | 是 |100.00 |
| `bank` | 银行的 SWIFT 编码，参考 `methods` 返回数据 | `gateway` 为 `banks` 时必填 |COMM |
| `ip` | 用户发起充值请求时的客户端 IP 地址，注：非服务器地址 | 否 | 127.0.0.1 |
| `return_url` | 支付完成后的同步跳转页面，可用于给用户提示进度 | 是 |http://xxx/deposit/return |
| `notify_url` | 支付完成后的异步通知地址 | 是 |http://xxx/deposit/notify  |
| `extra` | 可传任意数据，通知时原样返回 | 否 |dq |
| `rank` | 用户分层 | 否 |2 |
| `user_id` | 用户 ID | 否 |BM_1123 |
| `sign` | 签名 | 是 |XXXXX |


返回数据

1. 前端 HTML 表单跳转的情况

    ```json
    {
        "method": "post",
        "url": "...",
        "form": {}
    }
    ```

    `method` 为 "post" 时在前端页面将 `form` 数据渲染成 HTML 表单并自动提交，
    或者为 "get" 时前端将 `form` 数据渲染为 URL 参数自动跳转

2. 直接显示二维码图片的情况

    ```json
    {
        "method": "qrcode",
        "url": "..."
    }
    ```

    `url` 为二维码图片的 URL 地址或者 base64 编码数据，均可被 `<img src="...">` 直接显示

3. 链接跳转
    ```json
    {
         "method" : "get",
         "url" : "..."
    }
    ```
    
4. 网关异常

    ```json
    {
        "ok": false
    }
    ```

异常数据

- HTTP 409，订单号重复
- HTTP 422，参数错误
- HTTP 403，验签失败

***提示***
1. sign的加密方式为，将待上传参数（除sign）按照升序排序，val 按照 `a&b&c` 进行拼接后，进行 `openssl_encrypt`  `AES-256-CBC` 加密，结果进行`base64`操作

---

#### 发起充值请求(移动端)

方法

`POST`

路径

`/deposit/{merchants}/mobile/forward`

请求数据

| 参数名称 | 描述 | 是否必填 |
| --- | --- | --- |
| `order_no` | 商户订单号 | 是 |
| `gateway` | 充值网关，如 `alipay`, `banks` 等 | 是 |
| `amount` | 充值金额，单位元保留两位小数，如 100.00，使用支付宝网关时小数点后两位必须为 .00 | 是 |
| `notify_url` | 支付完成后的异步通知地址 | 是 |
| `ip` | 用户发起充值请求时的客户端 IP 地址，注：非服务器地址 | 否 |
| `extra` | 可传任意数据，通知时原样返回 | 否 |
| `rank` | 用户分层 | 否 |2 |
| `sign` | 签名（请看最后的【发起充值完整加密案列】） | 是 |

返回数据

*参考 PC 端*

异常数据

- HTTP 409，订单号重复
- HTTP 422，参数错误
- HTTP 403，验签失败

***提示***
1. sign的加密方式为，将待上传参数（除sign）按照升序排序，val 按照 `a&b&c` 进行拼接后，进行 `openssl_encrypt`  `AES-256-CBC` 加密，结果进行`base64`操作

---

#### 充值结果同步通知

*请忽略同步通知的数据不做任何处理，以异步通知结果数据为准*

---

#### 充值结果异步通知

方法

`POST`

请求数据

| 参数名称 | 描述 |
| --- | --- |
| `content` | 加密的通知结果数据，加密方式为 `AES-256-CBC` |

结果数据包

| 参数名称 | 描述 |
| --- | --- |
| `order_no` | 商户订单号 |
| `tx_no` | 充值流水号 |
| `amount` | 充值金额，单位元保留两位小数，如 100.00 |
| `actual_amount` | 实际充值金额，用于银行卡转账等用户可以改变充值金额的情况 |
| `status` | 充值状态 |
| `extra` | 发起充值请求时的任意数据 |

返回数据

异步通知会重试 3 次，直到 HTTP 状态为 2xx

***提示***
1. sign的加密方式为，将收到的数组（无需拼接，无需拼接，无需拼接），直接进行 `openssl_encrypt`  `AES-256-CBC` 加密，结果进行`base64`操作

---

### 查询接口

#### 查询充值订单状态

方法

`POST`

路径

`/deposit/{merchants}/order/{no}`

请求数据

| 参数名称 | 描述 | 是否必填 | 案例 |
| --- | --- | --- | --- |
| `merchants` | 商户名（在地址中，post不传此参数） | 是 | bomao |
| `no` | 商户订单号（在地址中，post不传此参数） | 是 | XXXXXX |
| `sign` | 加密结果（注意，post的参数只有一个sign） | 是 | XXX |

返回数据，例子

```json
{
    "tx_no": "201903081045040474650860",      // 支付系统订单号
    "channel": "qc--onepay_4115",             // 渠道名称
    "status": "issued",                       // 状态         
    "amount": "50.00",                        // 金额
    "no": "1111111720"                        // 商户系统订单号
}
```

***提示***
1. sign的加密方式为，将字符串 `{merchants}&{no}` 进行openssl  AES-256-CBC 加密，结果进行base64操作

---

---

#### 查询代付订单状态

方法

`POST`

路径

`/withdrawal/{merchants}/order/{no}`

请求数据

| 参数名称 | 描述 | 是否必填 | 案例 |
| --- | --- | --- | --- |
| `merchants` | 商户名（在地址中，post不传此参数） | 是 | bomao |
| `no` | 商户订单号（在地址中，post不传此参数） | 是 | XXXXXX |
| `sign` | 加密结果（注意，post的参数只有一个sign） | 是 | XXX |

返回数据，例子

```json
{
    "status": "issued",                       // 状态         
    "amount": "50.00",                        // 金额
    "no": "1111111720"                        // 商户系统订单号
}
```

***提示***
1. sign的加密方式为，将字符串 `{merchants}&{no}` 进行openssl  AES-256-CBC 加密，结果进行base64操作

---

---

### 提现接口

---

#### 查询当前可用提现方式（取消）

方法

`GET`

路径

`/withdrawal/{merchants}/methods`

返回数据

*暂未实现*

---

#### 发起提现请求

方法

`POST`

路径

`/withdrawal/{merchants}/forward`

请求数据

| 参数名称 | 描述 | 是否必填 |
| --- | --- | --- |
| `order_no` | 商户订单号 | 是 |
| `amount` | 提现金额，单位元保留两位小数，如 100.00 | 是 |
| `bank` | 提现到银行的 SWIFT 编码 | 是 |
| `bank_province` | 开户银行所在省 | 是 |
| `bank_city` | 开户银行所在城市 | 是 |
| `bank_branch` | 开户银行支行名称 | 是 |
| `card_holder` | 持卡人名字 | 是 |
| `card_no` | 银行卡号 | 是 |
| `holder_phone` | 持卡人在银行开户填写的电话号码 | 否 |
| `holder_id` | 持卡人身份证号码 | 否 |
| `notify_url` | 代付完成后的异步通知地址 | 是 |
| `cnaps` | 开户银行的联行号 | 根据当前可用提现方式决定是否必填 |
| `rank` | 用户分层 | 否 |
| `extra` | 可传任意数据，通知时原样返回 | 否 |
| `sign` | 签名（请看最后的【发起充值完整加密案列】），与充值一致，所有参数按照key的升序排列，将值拼接字符串，进行加密 | 是 |

返回数据

```json
{
    "ok": true
}
```

异常数据

- HTTP 409，订单号重复
- HTTP 422，参数错误
- HTTP 403，验签失败

***提示***
1. sign的加密方式为，将待上传参数（除sign）按照升序排序，val 按照 `a&b&c` 进行拼接后，进行 `openssl_encrypt`  `AES-256-CBC` 加密，结果进行`base64`操作

---

#### 提现结果异步通知

方法

`POST`

请求数据

| 参数名称 | 描述 |
| --- | --- |
| `content` | 加密的通知结果数据，加密方式为 `AES-256-CBC` |

结果数据包

| 参数名称 | 描述 |
| --- | --- |
| `order_no` | 商户订单号 |
| `tx_no` | 提现流水号 |
| `amount` | 提现金额，单位元保留两位小数，如 100.00 |
| `status` | 提现状态 |
| `extra` | 发起提现请求时的任意数据 |

返回数据

异步通知会重试 3 次，直到 HTTP 状态为 2xx

***提示***
1. sign的加密方式为，将收到的数组（无需拼接，无需拼接，无需拼接），直接进行 `openssl_encrypt`  `AES-256-CBC` 加密，结果进行`base64`操作

---

---

---



## V2接口

---
### 充值接口

#### 查询当前可用充值方式

方法

`GET`

路径

`/deposit/{merchants}/methods`

请求数据

| 参数名称 | 描述 | 是否必填 | 案例 |
| --- | --- | --- | --- |
| `rank` | 标签，此处引入标签概念，需要配合商户后台使用。请现在商户后台创建标签。多标签请用`-`拼接 | 是（注意，必填） | new-1-high （此处表示3个标签，`new`,`1`,`high`）|

返回数据，例子

```json
{
    "ok": true,
    "data": [
        {
            "gateway": "banks",
            "name": "网银快捷",
            "limits": {
                "min": 2,
                "max": 100
            },
            "banks": [
                {
                    "code": "ABOC",
                    "name": "农业银行"
                }
            ]
        },
        {
            "gateway": "unionpay",
            "name": "银联快捷",
            "limits": {
                "min": 2,
                "max": 100
            }
        },
        {
            "gateway": "mobile_weixin_fixed",
            "name": "移动微信定额",
            "options": [100, 200, 300]
        }
    ]
}
```

***提示***
1. 当 `gateway` 是 `banks` 时有 `banks` 字段列出当前支持的所有可充值银行
2. 当 `gateway` 是 `*_fixed` 时有 `options` 字段列出当前支持的所有固定可充值金额

---

#### 查询当前可用充值方式(移动端)

方法

`GET`

路径

`/deposit/{merchants}/mobile/methods`

请求数据

| 参数名称 | 描述 | 是否必填 | 案例 |
| --- | --- | --- | --- |
| `rank` | 标签，此处引入标签概念，需要配合商户后台使用。请现在商户后台创建标签。多标签请用`-`拼接 | 是（注意，必填） | new-1-high （此处表示3个标签，`new`,`1`,`high`）|


返回数据

```json
{
    "ok": true,
    "data": [
        {
            "gateway": "mobile_unionpay",
            "name": "银联移动",
            "limits": [], // 如果为空，请联系后台管理员配置
            "tips": null
        },
        {
            "gateway": "mobile_qq",
            "name": "QQ移动",
            "limits": {
                "min": 10,
                "max": 10000
            },
            "tips": null
        },
        {
            "gateway": "mobile_weixin",
            "name": "微信移动",
            "limits": {
                "min": 10,
                "max": 10000
            },
            "tips": null
        }
    ]
}
```

---

#### 发起充值请求

方法

`POST`

路径

`/deposit/{merchants}/forward`

请求数据

| 参数名称 | 描述 | 是否必填 | 案列 |
| --- | --- | --- |--- |
| `order_no` | 商户订单号 | 是 |BBMM1111113286 |
| `gateway` | 充值网关，如 `alipay`, `banks` 等 | 是 |banks |
| `amount` | 充值金额，单位元保留两位小数，如 100.00，使用支付宝网关时小数点后两位必须为 .00 | 是 |100.00 |
| `bank` | 银行的 SWIFT 编码，参考 `methods` 返回数据 | `gateway` 为 `banks` 时必填 |COMM |
| `ip` | 用户发起充值请求时的客户端 IP 地址，注：非服务器地址 | 否 | 127.0.0.1 |
| `return_url` | 支付完成后的同步跳转页面，可用于给用户提示进度 | 是 |http://xxx/deposit/return |
| `notify_url` | 支付完成后的异步通知地址 | 是 |http://xxx/deposit/notify  |
| `extra` | 可传任意数据，通知时原样返回 | 否 |dq |
| `rank` | 标签，此处引入标签概念，需要配合商户后台使用。请现在商户后台创建标签。多标签请用`-`拼接 | 是（注意，必填） | new-1-high （此处表示3个标签，`new`,`1`,`high`）|
| `user_id` | 用户 ID | 否 |BM_1123 |
| `sign` | 签名（请看最后的【发起充值完整加密案列】） | 是 |XXXXX |


返回数据

1. 前端 HTML 表单跳转的情况

    ```json
    {
        "method": "post",
        "url": "...",
        "form": {}
    }
    ```

    `method` 为 "post" 时在前端页面将 `form` 数据渲染成 HTML 表单并自动提交，
    或者为 "get" 时前端将 `form` 数据渲染为 URL 参数自动跳转

2. 直接显示二维码图片的情况

    ```json
    {
        "method": "qrcode",
        "url": "..."
    }
    ```

    `url` 为二维码图片的 URL 地址或者 base64 编码数据，均可被 `<img src="...">` 直接显示

3. 链接跳转
    ```json
    {
         "method" : "get",
         "url" : "..."
    }
    ```
    
4. 网关异常

    ```json
    {
        "ok": false
    }
    ```

异常数据

- HTTP 409，订单号重复
- HTTP 422，参数错误
- HTTP 403，验签失败

***提示***
1. sign的加密方式为，将待上传参数（除sign）按照升序排序，val 按照 `a&b&c` 进行拼接后，进行 `openssl_encrypt`  `AES-256-CBC` 加密，结果进行`base64`操作

---

#### 发起充值请求(移动端)

方法

`POST`

路径

`/deposit/{merchants}/mobile/forward`

请求数据

| 参数名称 | 描述 | 是否必填 |
| --- | --- | --- |
| `order_no` | 商户订单号 | 是 |
| `gateway` | 充值网关，如 `alipay`, `banks` 等 | 是 |
| `amount` | 充值金额，单位元保留两位小数，如 100.00，使用支付宝网关时小数点后两位必须为 .00 | 是 |
| `notify_url` | 支付完成后的异步通知地址 | 是 |
| `ip` | 用户发起充值请求时的客户端 IP 地址，注：非服务器地址 | 否 |
| `extra` | 可传任意数据，通知时原样返回 | 否 |
| `rank` | 标签，此处引入标签概念，需要配合商户后台使用。请现在商户后台创建标签。多标签请用`-`拼接 | 是（注意，必填） | new-1-high （此处表示3个标签，`new`,`1`,`high`）|
| `sign` | 签名（请看最后的【发起充值完整加密案列】） | 是 |

返回数据

*参考 PC 端*

异常数据

- HTTP 409，订单号重复
- HTTP 422，参数错误
- HTTP 403，验签失败

***提示***
1. sign的加密方式为，将待上传参数（除sign）按照升序排序，val 按照 `a&b&c` 进行拼接后，进行 `openssl_encrypt`  `AES-256-CBC` 加密，结果进行`base64`操作

---

#### 充值结果同步通知

*请忽略同步通知的数据不做任何处理，以异步通知结果数据为准*

---

#### 充值结果异步通知

方法

`POST`

请求数据

| 参数名称 | 描述 |
| --- | --- |
| `content` | 加密的通知结果数据，加密方式为 `AES-256-CBC` |

结果数据包

| 参数名称 | 描述 |
| --- | --- |
| `order_no` | 商户订单号 |
| `tx_no` | 充值流水号 |
| `amount` | 充值金额，单位元保留两位小数，如 100.00 |
| `actual_amount` | 实际充值金额，用于银行卡转账等用户可以改变充值金额的情况 |
| `status` | 充值状态 |
| `extra` | 发起充值请求时的任意数据 |

返回数据

异步通知会重试 3 次，直到 HTTP 状态为 2xx

***提示***
1. sign的加密方式为，将收到的数组（无需拼接，无需拼接，无需拼接），直接进行 `openssl_encrypt`  `AES-256-CBC` 加密，结果进行`base64`操作

---
---

### 查询接口

#### 查询充值订单状态

方法

`POST`

路径

`/deposit/{merchants}/order/{no}`

请求数据

| 参数名称 | 描述 | 是否必填 | 案例 |
| --- | --- | --- | --- |
| `merchants` | 商户名（在地址中，post不传此参数） | 是 | bomao |
| `no` | 商户订单号（在地址中，post不传此参数） | 是 | XXXXXX |
| `sign` | 加密结果（注意，post的参数只有一个sign） | 是 | XXX |

返回数据，例子

```json
{
    "tx_no": "201903081045040474650860",      // 支付系统订单号
    "channel": "qc--onepay_4115",             // 渠道名称
    "status": "issued",                       // 状态         
    "amount": "50.00",                        // 金额
    "no": "1111111720"                        // 商户系统订单号
}
```

***提示***
1. sign的加密方式为，将字符串 `{merchants}&{no}` 进行openssl  AES-256-CBC 加密，结果进行base64操作

---

---

#### 查询代付订单状态

方法

`POST`

路径

`/withdrawal/{merchants}/order/{no}`

请求数据

| 参数名称 | 描述 | 是否必填 | 案例 |
| --- | --- | --- | --- |
| `merchants` | 商户名（在地址中，post不传此参数） | 是 | bomao |
| `no` | 商户订单号（在地址中，post不传此参数） | 是 | XXXXXX |
| `sign` | 加密结果（注意，post的参数只有一个sign） | 是 | XXX |

返回数据，例子

```json
{
    "status": "issued",                       // 状态         
    "amount": "50.00",                        // 金额
    "no": "1111111720"                        // 商户系统订单号
}
```

***提示***
1. sign的加密方式为，将字符串 `{merchants}&{no}` 进行openssl  AES-256-CBC 加密，结果进行base64操作

---

---

### 提现接口

---

#### 查询当前可用提现方式（取消）

方法

`GET`

路径

`/withdrawal/{merchants}/methods`

返回数据

*暂未实现*

---

#### 发起提现请求

方法

`POST`

路径

`/withdrawal/{merchants}/forward`

请求数据

| 参数名称 | 描述 | 是否必填 |
| --- | --- | --- |
| `order_no` | 商户订单号 | 是 |
| `amount` | 提现金额，单位元保留两位小数，如 100.00 | 是 |
| `bank` | 提现到银行的 SWIFT 编码 | 是 |
| `bank_province` | 开户银行所在省 | 是 |
| `bank_city` | 开户银行所在城市 | 是 |
| `bank_branch` | 开户银行支行名称 | 是 |
| `card_holder` | 持卡人名字 | 是 |
| `card_no` | 银行卡号 | 是 |
| `holder_phone` | 持卡人在银行开户填写的电话号码 | 否 |
| `holder_id` | 持卡人身份证号码 | 否 |
| `notify_url` | 代付完成后的异步通知地址 | 是 |
| `cnaps` | 开户银行的联行号 | 根据当前可用提现方式决定是否必填 |
| `rank` | 用户分层 | 否 |
| `extra` | 可传任意数据，通知时原样返回 | 否 |
| `sign` | 签名（请看最后的【发起充值完整加密案列】），与充值一致，所有参数按照key的升序排列，将值拼接字符串，进行加密 | 是 |

返回数据

```json
{
    "ok": true
}
```

异常数据

- HTTP 409，订单号重复
- HTTP 422，参数错误
- HTTP 403，验签失败

***提示***
1. sign的加密方式为，将待上传参数（除sign）按照升序排序，val 按照 `a&b&c` 进行拼接后，进行 `openssl_encrypt`  `AES-256-CBC` 加密，结果进行`base64`操作

---

#### 提现结果异步通知

方法

`POST`

请求数据

| 参数名称 | 描述 |
| --- | --- |
| `content` | 加密的通知结果数据，加密方式为 `AES-256-CBC` |

结果数据包

| 参数名称 | 描述 |
| --- | --- |
| `order_no` | 商户订单号 |
| `tx_no` | 提现流水号 |
| `amount` | 提现金额，单位元保留两位小数，如 100.00 |
| `status` | 提现状态 |
| `extra` | 发起提现请求时的任意数据 |

返回数据

异步通知会重试 3 次，直到 HTTP 状态为 2xx

***提示***
1. sign的加密方式为，将收到的数组（无需拼接，无需拼接，无需拼接），直接进行 `openssl_encrypt`  `AES-256-CBC` 加密，结果进行`base64`操作

---

---

---
---

## 附录

#### 可用充值网关 PC 端

| 字段 | 描述 |
| --- | --- |
| `banks` | 网银快捷 |
| `banks_3rd` | 网银快捷2 |
| `banks_c2c` | 网银转账 |
| `unionpay` | 银联快捷 |
| `unionpay_qr` | 银联扫码 |
| `alipay` | 支付宝 |
| `alipay_fixed` | 支付宝定额 |
| `alipay_c2c`  | 支付宝转账 |
| `weixin` | 微信钱包 |
| `qq` | QQ 钱包 |
| `jd` | 京东钱包 |

---

#### 可用充值网关移动端

| 字段 | 描述 |
| --- | --- |
| `mobile_unionpay` | 银联移动 |
| `mobile_alipay` | 支付宝移动 |
| `mobile_alipay_c2c` | 支付宝移动转账 |
| `mobile_weixin` | 微信移动 |
| `mobile_weixin_fixed` | 移动微信定额 |
| `mobile_qq` | QQ 移动 |
| `mobile_jd` | 京东移动 |
| `mobile_alipay_fixed` |支付宝移动定额|

---

#### 充提状态

| 状态值 | 描述 |
| --- | --- |
| `created` | 流水单据已生成 |
| `issued` | 充值表单或二维码数据已返回、提现网关请求已发起 |
| `succeed` | 成功 |
| `failed` | 失败 |
| `in-progress` | 处理中 |
| `unconfirmable` | 异常情况，需人工参与确认 |
| `expired` | 超时，暂未使用 |


---

#### 异步通知数据加密示例

PHP 版本

```php
$data = [
    'content' => base64_encode(
        openssl_encrypt(
            // 注意，此为异步通知的数据处理方式；请求sign的处理方式是字符串拼接，字符串拼接，字符串拼接
            json_encode(
                [
                    'order_no' => 'goods20171212001',
                    'tx_no' => '201712120951160922999382',
                    'amount' => '100.00',
                    'status' => 'succeed',
                    'extra' => '...',
                ]
            ),
            'AES-256-CBC',
            'key...', // 由 GEM 签发的 key
            OPENSSL_RAW_DATA,
            'vector...' // 由 GEM 签发的 vector
        )
    )
];
```

---

#### 银行码表参考 SWIFT 规范

部分范例，以 `methods` 接口返回数据为准

| 代码 | 银行名称 |
| --- | --- |
| `BKCH` | 中国银行 |
| `CMBC` | 招商银行 |
| `COMM` | 交通银行 |
| `PCBC` | 建设银行 |
| `ICBK` | 工商银行 |
| `ABOC` | 农业银行 |
| `PSBC` | 邮储银行 |
| `EVER` | 光大银行 |
| `MSBC` | 民生银行 |
| `CIBK` | 中信银行 |
| `SPDB` | 浦发银行 |
| `GDBK` | 广发银行 |
| `HXBK` | 华夏银行 |
| `FJIB` | 兴业银行 |
| `SZDB` | 平安银行 |
| `BJCN` | 北京银行 |
| `NJCB` | 南京银行 |
| `HZCB` | 杭州银行 |
| `BKNB` | 宁波银行 |
| `BOSH` | 上海银行 |
| `CHBH` | 渤海银行 |
| `ZJCB` | 浙商银行 |
| `SHRC` | 上海农商银行 |
| `SRCC` | 深圳农商银行 |
| `RCCS` | 顺德农商银行 |
| `BRCB` | 北京农商银行 |
| `BEAS` | 东亚银行 |
| `TCCB` | 天津银行 |
| `CHCC` | 长沙银行 |
| `HFBA` | 恒丰银行 |
| `HFCB` | 徽商银行 |
| `CZCB` | 浙江稠州商业银行 |
| `ZJTL` | 浙江泰隆商业银行 |

---

### 已有加密案例，请联系接口人获取
#### 加密测试网址 ` http://tool.chacuo.net/cryptaes `
* PHP
* JAVA

