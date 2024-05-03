# 支付接口说明

- 支付类型heepay 请求Api：https://域名/api/heepay
- 支付类型为其他 请求Api：https://域名//api/pay  
- 请求方式：get/post （返回url）

### 参数：

|           参数         | 是否必填 | 描述           |
|:----------------------|:------:|:-------------|
| app_id               | 必填 | 商户appid，后台可见 |
| order_no             | 必填 | 商户订单，原样回传    |
| pay_type             | 必填 | 参照支付类型       |
| pay_amt              | 必填 | 支付金额         |
| pay_cur              | 必填 | 固定参数：CNY     |
| goods_name           | 必填 | 商品名字         |
| return_url           | 必填 | 同步通知url      |
| notify_url           | 必填 | 异步通知url      |
| application_username | 必填 | 用户姓名         |
| application_user_id  | 必填 | 身份证号，用户唯一标识  |
| state                | 选填 | 商户自定义信息，原值回传 |
| ts                   | 必填 | unix格式时间戳    |
| sign                 | 必填 | 签名           |

### 签名方式(注意顺序)：

参数根据键名以 ascii 升序排序，并剔除值为空(null 或空字符串)的参数及sign参数。  
处理后的参数以 "{$key}={$value}" 的方式拼接，{ts}与[secret_key]无键值。

示例:
```
app_id={$app_id}notify_url={$notify_url}order_no={$order_no}pay_amt={$pay_amt}pay_cur={$pay_cur}pay_type={$pay_type}return_url={$return_url}{$ts}[secret_key]
```
对以上内容进行md5，结果为英文为小写，[secret_key]为商户秘钥（见商户后台）

## return/notify通知回调

### 注意事项
同步方式（return）不可靠，请使用异步方式（notify）作为充值判断依据

### 请求方式
get

### 场景说明
用户支付完成（成功或失败）后，return通知为直接返回商户给到url（一般是用户点击了返回商户界面操作，会执行跳转，也可能不触发）

用户支付完成（成功或失败）后，notify通知为异步通知，肯定会触发，支付平台后台调用该接口，不会在用户界面体现。

以上两个通知方式的参数都是一样的，注意不要重复处理订单！

### 参数：

|      参数      |  是否必填  | 描述         |
|:---------------|:----------:|:-----------|
| app_id         |   一定有   | 商户appid    |
| is_success     |   一定有   | 1：成功，0：失败  |
| fail_msg       |   失败有   | 失败消息       |
| order_no       |   一定有   | 商户订单       |
| pay_actual_amt |   一定有   | 实际支付金额     |
| state          |   一定有   | 自定义信息，原值回传 |
| ts             |   一定有   | unix格式时间戳  |
| sign           |   一定有   | md5签名      |

### 通知验参示例：
```
app_id={$app_id}is_success={$is_success}order_no={$order_no}pay_actual_amt={$pay_actual_amt}{$ts}[secret_key]
```
签名规则参照支付接口

### notify通知返回：
处理成功后请返回字符串：ok。则视为通知成功，如果没有返回或返回其它内容，则视为通知失败。

### 支付类型：
- pay_type = heepay *汇付宝快捷支付(网关签约)*
- pay_type = bank *网银支付*
- pay_type = bank_swift *网关快捷*
- pay_type = bank_card *银行卡转账*
- pay_type = union_swift *银联快捷*
- pay_type = union_scan *银联扫码*
- pay_type = alipay *支付宝*
- pay_type = wechat *微信*
- pay_type = e_cny *数字人民币*

## 常见错误码说明
| errcode | 错误信息    |
|:-------:|:--------|
|  1001   | 参数错误    |
|  1002   | 签名错误    |
|  1003   | 商户不存在   |
|  1004   | 金额限额    |
|  1022   | 参数格式错误  |
|  2001   | 订单号已存在  |
|  2002   | 订单创建失败  |
|  2003   | 没有权限    |
|  2004   | 城市不匹配   |
|  2005   | 验证码发送失败 |
|  2006   | 验证码验证失败 |
