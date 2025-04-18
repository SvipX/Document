# 支付接口说明

- 请先联系商务，添加白名单。

## 支付提单

- 请求Api：https://域名/api/pay  
- 请求方式：get/post （返回url）

### 请求参数

|           参数         | 是否必填 | 参与签名 | 描述           |
|:----------------------|:------:|:------:|:-------------|
| app_id               | 必填 | 是 | 商户appid，后台可见 |
| order_no             | 必填 | 是 | 商户订单，原样回传    |
| pay_type             | 必填 | 是 | 参照支付类型       |
| pay_amt              | 必填 | 是 | 支付金额         |
| pay_cur              | 必填 | 是 | 固定参数：CNY     |
| goods_name           | 必填 | 否 | 商品名字         |
| return_url           | 必填 | 是 | 同步通知url      |
| notify_url           | 必填 | 是 | 异步通知url      |
| application_username | 选填 | 否 | 用户姓名，不填影响成功率。heepay 必填  |
| application_user_id  | 必填 | 否 | 用户唯一标识,必填，我方仅需要识别订单来自同一个用户  |
| state                | 选填 | 否 | 商户自定义信息，原值回传 |
| ts                   | 必填 | 是 | unix格式时间戳    |
| sign                 | 必填 | 否 | MD5签名           |

### 签名方式

参与签名参数根据键名以 ascii 升序排序，并剔除值为空(null 或空字符串)的参数及sign参数。  
处理后的参数以 "{$key}={$value}" 的方式拼接，{$ts}与{$secret_key}只拼接值，无键名。

- 签名字符串示例:
  
```
app_id={$app_id}notify_url={$notify_url}order_no={$order_no}pay_amt={$pay_amt}pay_cur={$pay_cur}pay_type={$pay_type}return_url={$return_url}{$ts}{$secret_key}
```
对以上内容进行md5，结果为英文为小写，{$secret_key}为商户秘钥（见商户后台）

### 支付返回参数

如果没有报错，响应支付连接（明文）。如果报错响应json，例：
```
[
  'errcode' => '5000',
  'errmsg' => '系统错误'
]
```
常见错误码说明在最下

## 支付通知回调

- 请求方式：get
- 同步方式（return）不可靠，请使用异步方式（notify）作为充值判断依据
- 
### 场景说明

用户支付完成（成功或失败）后，return通知为直接返回商户给到url（一般是用户点击了返回商户界面操作，默认不触发）  
用户支付完成（成功或失败）后，notify通知为异步通知，肯定会触发，支付平台后台调用该接口，不会在用户界面体现。  
以上两个通知方式的参数都是一样的，注意不要重复处理订单！  

### 通知参数

|      参数      |  是否必填  |  参与签名  | 描述         |
|:---------------|:----------:|:----------:|:-----------|
| app_id         |   一定有   |   是   | 商户appid    |
| is_success     |   一定有   |   是   | 1：成功，0：失败  |
| fail_msg       |   失败有   |   否   | 失败原因       |
| order_no       |   一定有   |   是   | 商户订单号       |
| pay_actual_amt |   一定有   |   是   | 实际支付金额     |
| pay_type       |   一定有   |   否   | 支付类型 |
| transaction_id |   一定有   |   否   | 平台订单编号 |
| state          |   一定有   |   否   | 自定义信息，原值回传 |
| ts             |   一定有   |   是   | unix格式时间戳  |
| sign           |   一定有   |   否   | md5签名      |

### 通知验参

- 签名字符串示例:
  
```
app_id={$app_id}is_success={$is_success}order_no={$order_no}pay_actual_amt={$pay_actual_amt}{$ts}{$secret_key}
```
签名规则参照提单签名方式

### notify通知返回

处理成功后请返回字符串：ok。则视为通知成功，如果没有返回或返回其它内容，则视为通知失败。  
通知失败会间隔60秒再次通知，连续5次。注意不要重复处理订单！  

## 支付查询

- 查询Api：https://域名/api/pay/query
- 请求方式：post

### 查询参数

|   参数   |  是否必填  |    描述    |
|:---------|:----:|:-----------|
| app_id   | 必填 | 商户appid  |
| order_no | 必填 | 商户订单编号 |
| ts       | 必填 | unix时间戳 |
| sign     | 必填 | md5签名    |

### 查询签名

- 签名字符串示例:
  
```
app_id={$app_id}order_no={$order_no}{$ts}{$secret_key}
```
签名规则参照提单签名方式

### 查询返回参数

- 示例：json()
  
```
[
    'errcode' => '0',
    'errmsg' => '查询成功',
    'order_no' => 商户订单,
    'transaction_id' => 平台订单编号,
    'amount' => 支付金额,
    'create_time' => 下单时间,
    'success_time' => 成功时间,
    'status' => 1
]
```
errcode不为 0 时，仅有errcode 与 errmsg两个参数，errmsg为错误原因。  
status为订单状态 1成功 0未支付 -1失败。  

## 支付类型：

- pay_type = e_cny *数字人民币*
- pay_type = union *银联快捷*
- pay_type = union_scan *银联扫码*
- pay_type = union_cloud *云闪付*
- pay_type = alipay *支付宝转账*
- pay_type = alipay_h5 *支付宝H5*
- pay_type = alipay_web *支付宝伪原生*
- pay_type = alipay_scan *支付宝扫码*
- pay_type = alipay_fixed *支付宝固额*
- pay_type = wechat *微信*
- pay_type = wechat_h5 *微信H5*
- pay_type = wechat_scan *微信扫码*
- pay_type = usdt * USDT TRC20*

## 常见错误码说明
| errcode | errmsg    |
|:-------:|:--------|
|  1001   | 参数错误    |
|  1002   | 签名错误    |
|  1003   | 商户不存在   |
|  1004   | 金额限额    |
|  1022   | 参数格式错误  |
|  2001   | 订单号已存在  |
|  2002   | 订单创建失败  |
|  2003   | 没有权限    |
|  2004   | 通道错误   |
|  2005   | 验证码发送失败 |
|  2006   | 验证码验证失败 |
