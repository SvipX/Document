# 代付接口说明
## 代付提单
- 代付请求Api：https://域名/api/withdraw
- 请求方式：post

### 请求参数

| 参数              | 是否必填 | 描述             |
|:----------------|:------:|:---------------|
| app_id          | 必填 | 商户appid，后台可见   |
| withdraw_type   | 必填 | 1:安全发 2：支付宝转账 3：银行卡 4：数字人民币 5：USDT  |
| amount          | 必填 | 代付金额，单位：元      |
| order_no        | 必填 | 商户订单，原样回传      |
| bank_id         | 必填 | 支付宝代付填 0  其他参照 banks.json |
| bank_account    | 必填 | 银行账号/支付宝账号     |
| bank_account_type | 必填 | 银行类型 0 对私 1 对公 |
| bank_user_name  | 必填 | 账户姓名/支付宝姓名  |
| bank_user_id    | 必填 | 身份证号或其他游戏id；用户唯一标识         |
| branch_name     | 必填 | 支行名称；完整的名称：中国银行上海市正阳支行 |
| notify_url     | 必填 | 回调地址 |
| city            | 非必填 | 地级城市名或直辖市区名； 如: 杭州市 或 朝阳区。参照 citys.json |
| ts              | 必填 | unix格式时间戳    |
| sign            | 必填 | 签名             |

收款方式若为支付宝 branch_name 固定值 "支付宝"。

###  签名方式

所有请求参数根据键名以 ascii 升序排序，并剔除值为空(null 或空字符串)的参数及sign参数。  
处理后的参数以 "&" 作为间隔，以 "{$key}={$value}" 的方式拼接，最后在结尾拼接上 "key={$secret_key}"，得到签名字符串。  
最后使用 md5 加密签名字符串，得到请求签名 sign。签名结果为小写。

- 签名示例:

```
    protected function sign($params, $secret_key)
    {
        $sign_str = '';
        ksort($params);
        foreach ($params as $index => $param) {
            if ($index === 'sign' || $param === '' || $param === null) {
                continue;
            }
            $sign_str .= "{$index}={$param}&";
        }
        $sign_str .= "key={$secret_key}";
        return md5($sign_str);
    }
```

### 代付返回参数 

- 示例：json()
  
```
[
    'errmsg' => '提交成功',
    'errcode' => 0
]
```
errcode为 0 时说明提交成功，其他值为提交失败，errmsg为失败信息


## 代付异步通知

- 请求方式：get

### 场景说明

平台接收代付请求后，将会生成一笔提现订单并开始执行，并于订单创建完成后异步通知至提现所传入的notify_url。  
根据商户权限和提现金额的不同，提现所需到账时间也会有所不同，故异步通知可能比较久。  
成功或失败的订单都会通知一次，请确认通知中携带的订单状态，如果失败会携带失败原因。

### 通知参数

|      参数      |  是否必填  |  参与签名  | 描述         |
|:---------------|:----------:|:----------:|:-----------|
| app_id         |   一定有   |   是   | 商户appid    |
| status     |   一定有   |   是   | success代付成功 checking处理中 fail代付失败。   |
| fail_msg       |   失败有   |   否   | 失败原因  urlencode(中文)     |
| order_no       |   一定有   |   是   | 商户订单号       |
| transaction_id |   一定有   |   是   | 平台订单号     |
| amount       |   一定有   |   是   | 代付金额 |
| ts             |   一定有   |   是   | unix格式时间戳  |
| sign           |   一定有   |   否   | md5签名      |

### 通知验参

参数根据键名以 ascii 升序排序，并剔除值为空(null 或空字符串)的参数及sign参数。  
处理后的参数以 "{$key}={$value}" 的方式拼接，{$ts}与{$secret_key}只拼接值，无键名。

- 通知签名示例
  
```
$sign = md5("app_id={$app_id}status={$status}order_no={$order_no}transaction_id={$transaction_id}amount={$amount}{$ts}{$secret_key}");
```

### notify通知返回

处理成功后请返回 ok 视为通知成功，如果没有返回或返回其它内容，则视为通知失败。  

## 代付查询

- 代付查询Api：https://域名/api/withdraw/query
- 请求方式：post
- 请代付订单发起1分钟后再查询结果

### 查询参数

|   参数   |  空  |    描述    |
|:---------|:----:|:-----------|
| app_id   | 必填 | 商户appid  |
| order_no | 必填 | 代付订单编号 |
| ts       | 必填 | unix时间戳 |
| sign     | 必填 | md5签名    |

### 查询签名

- 查询签名示例
  
```
$sign = md5("app_id={$app_id}&order_no={$order_no}&ts={$ts}&key={$secret_key}");
```
签名规则参照提单签名方式
### 查询返回参数

- 示例：json()
  
```
[
    'errcode' => '0',
    'errmsg' => '查询成功',
    'status' => 'checking',
    'order_no' => $out_order_no,
    'ts' => time(),
];
```
errcode不为 0 时，仅有errcode 与 errmsg两个参数，errmsg为错误原因。  
errcode为 0 时说明查询成功，errmsg为代付订单备注（失败原因或鉴权）。  
status为代付订单状态 success代付成功 checking处理中 fail代付失败。  


## 常见错误参数说明

| errcode | 错误信息    |
|:-------:|:--------|
|  1001   | 参数错误    |
|  1002   | 签名错误    |
|  1003   | 商户不存在   |
|  1004   | 金额限额    |
|  1005   | 银行编码错误  |
|  1006   | 城市不匹配   |
|  2001   | 订单编号重复  |
|  2002   | 订单创建失败  |
|  2003   | 没有权限    |
|  2004   | 通道异常    |
|  3001   | 账户余额不足  |
|  3002   | 查询订单不存在 |
|  3200   | 代付通道请求失败 |
|  3201   | 代付通道请求超时 |
|  3202   | 代付通道提现失败 |
|  3203   | 不支持该银行 |
|  3204   | 代付查询超时 |
|  3205   | 代付订单失败 |
|  3206   | 鉴权失败，三要素不匹配 |
|  3207   | 鉴权成功，请重新发起支付 |
|  5000   | 系统异常    |
|  5001   | 通道异常    |
