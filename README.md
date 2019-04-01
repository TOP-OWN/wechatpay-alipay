# wechat_pay-alipay
支付宝支付 、支付宝退款、微信支付、微信企业付款、微信退款

## 一、支付宝支付
* 接口文档：https://docs.open.alipay.com/
* 公共错误码：https://docs.open.alipay.com/common/105806
 ```
 $config = [
     'app_id' => '',
     'rsa_private_key' => '',
     'alipay_public_key' => ''
 ];
 ```
#### 1.当面付
**注意：当面付不用设置应用网关和授权回调地址，只需要设置应用公钥**
``` 
$payData = [
    'order_no' => '', //商户订单号
    'order_price' => '', //订单总金额，单位：元 
    'subject' => '', //订单标题，粗略描述用户的支付目的
    'notify_url' => '', //支付成功后回调地址
    'expire' => '', ////当面付 二维码过期时间，单位：秒
];

$alipay = new Alipay($config);
$info = $alipay->placeQrcode($payData);
$info['code'] != 10000 && json_error((($info['sub_msg'] ?? '') . ' ' . $info['sub_code'] ?? ''), -2);
$info['code_url']  通过qrcode生成二维码后，用户扫码即可
```
#### 2.app支付
```
$payData = [
    'order_no' => '', //商户订单号
    'order_price' => '', //订单总金额，单位：元 
    'subject' => '', //订单标题，粗略描述用户的支付目的
    'notify_url' => '', //支付成功后回调地址
];

$alipay = new Alipay($config);
$info = $alipay->placeApp($payData);
将结果$info传给移动端即可，$info是将参数http_build_query()处理后生成的字符串
```
#### 3.手机网站支付
```
$payData = [
    'order_no' => '', //商户订单号
    'order_price' => '', //订单总金额，单位：元 
    'subject' => '', //订单标题，粗略描述用户的支付目的
    'notify_url' => '', //支付成功后回调地址
    'quit_url' => '', //用户付款中途退出返回商户网站的地址
    'product_code' => '', //销售产品码，商家和支付宝签约的产品码
];

$alipay = new Alipay($config);
$info = $alipay->placeWap($payData);
isset($info['errno']) && $info['errno'] != 0 && json_error($info['error']??'手机网站支付配置错误', -2);
$info['redirect_url']  手机网站调起支付宝链接
```
#### 4.电脑网站支付
* 服务端代码：
```
$payData = [
    'order_no' => '', //商户订单号
    'order_price' => '', //订单总金额，单位：元 
    'subject' => '', //订单标题，粗略描述用户的支付目的
    'notify_url' => '', //支付成功后回调地址
];

$alipay = new Alipay($config);
$info = $alipay->placePage($payData);
$info['html_form']  将结果通过ajax方式写入html body中即可
```

* html代码如下：
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>支付宝电脑网站支付</title>
</head>
<body id="body">

<script src="http://code.jquery.com/jquery-2.2.4.min.js" 
integrity="sha256-BbhdlvQf/xTY9gja0Dq3HiwQF8LaCRTXxZKRutelT44=" crossorigin="anonymous"></script>
<script>
    $.ajax({
        url: "/monitor/test", //接口地址请自行修改
        type: 'POST',
        success: function (data) {
            $("#body").html(data.data.html_form);
        }
    });
</script>
</body>
</html>
```

## 二、支付宝退款
 ```
 $config = [
     'app_id' => '',
     'rsa_private_key' => '',
     'alipay_public_key' => ''
 ];
 ```
#### 1.申请退款
 ```
$param = [
    'out_trade_no' => '', //商户订单号
    'refund_amount' => '', //需要退款的金额，该金额不能大于订单金额,单位为元，支持两位小数
    'refund_reason' => '', //退款的原因说明，256字以内
    'out_request_no' => '', //标识一次退款请求，同一笔交易多次退款需要保证唯一，如需部分退款，则此参数必传
];
 
$alipay = new AlipayRefund($config);
$info = $alipay->doRefund($param);
$info['code'] != 10000 && json_error((($info['sub_msg'] ?? '') . ' ' . $info['sub_code'] ?? ''), -2);
``` 
#### 2.退款查询
```
$payData = [
    'out_trade_no' => '', //商户订单号
    'out_request_no' => '', //标识一次退款请求，同一笔交易多次退款需要保证唯一，如需部分退款，则此参数必传
];
 
$alipay = new AlipayRefund($config);
$info = $alipay->queryRefund($payData);
$info['code'] != 10000 && json_error((($info['sub_msg'] ?? '') . ' ' . $info['sub_code'] ?? ''), -2);
```

## 三、微信支付
* 微信支付文档：https://pay.weixin.qq.com/wiki/doc/api/index.html
* 企业付款文档：https://pay.weixin.qq.com/wiki/doc/api/tools/mch_pay.php?chapter=14_2
```
$config = [
    'appid' => '',
    'mch_id' => '',
    'key' => '',
    'sslcert_path' => '', //支付时不用上传证书
    'sslkey_path' => ''
];

$payData = [
    'body' => '', //订单标题，粗略描述用户的支付目的
    'out_trade_no' => '', //商户订单号
    'total_fee' => '', //订单总金额，单位：分
    'trade_type' => '', //支付类型 JSAPI、APP、NATIVE、MWEB
    'notify_url' => '', //支付成功后回调地址
];

//不同支付方式，需传入参数
if ($payData['trade_type'] == 'JSAPI') {
    $payData['openid'] = ''; //公众号支付 用户openid
}elseif($payData['trade_type'] == 'NATIVE'){
    $payData['expire'] = ''; //当面付 二维码过期时间
}elseif($payData['trade_type'] == 'MWEB'){
    $payData['wap_url'] = ''; //H5支付 WAP网站URL地址
    $payData['wap_name'] = ''; //H5支付 WAP网站名
}

//预下单方法：
$wechatPay = new WechatPay($config);
$info = $wechatPay->unifiedOrder($payData);
$info['return_code'] == 'FAIL' && json_error($info['return_msg'], -2);
$info['result_code'] == 'FAIL' && json_error($info['err_code_des'], -2);
```
#### 1.扫码支付
```
$info['code_url']  此url用于生成支付二维码，然后提供给用户进行扫码支付。
                   注意：code_url的值并非固定，使用时按照URL格式转成二维码即可
```
#### 2.H5支付
```
$info['mweb_url']  mweb_url为拉起微信支付收银台的中间页面，可通过访问该url来拉起微信客户端，完成支付,
                   mweb_url有效期为5分钟可拼接跳转链接：$info['mweb_url'].'&redirect_url='.urlencode($returnUrl);
```
#### 3.App支付
```
$wechatPay->getAppParam($info['prepay_id']);  将结果返回给App端即可
```
#### 4.Jsapi支付
```
$wechatPay->getJsapiParam($info['prepay_id']);  将结果返回给微信内H5支付或小程序端即可
```

## 四、微信退款
 ```
$config = [
    'appid' => '',
    'mch_id' => '',
    'key' => '',
    'sslcert_path' => '', //退款时必须上传证书
    'sslkey_path' => ''
];
 ```
#### 1.申请退款
 ```
$param = [
    'out_trade_no' => '', //商户系统内部订单号
    'out_refund_no' => '', //商户系统内部的退款单号
    'total_fee' => '', //订单总金额，单位为分，只能为整数
    'refund_fee' => '', //退款总金额，订单总金额，单位为分，只能为整数
    'refund_desc' => '', //退款原因，注意：若订单退款金额≤1元且属于部分退款，则不会在退款消息中体现退款原因
    'notify_url' => '', //支付成功后回调地址
];
 
$wechatRefund = new WechatRefund($config);
$info = $wechatRefund->doRefund($param);
$info['return_code'] == 'FAIL' && json_error($info['return_msg'], -2);
$info['result_code'] == 'FAIL' && json_error($info['err_code_des'], -2);
``` 
#### 2.退款查询
```
$outTradeNo = ''; //商户订单号

$wechatRefund = new WechatRefund($config);
$info = $wechatRefund->queryRefund($outTradeNo); 
$info['return_code'] == 'FAIL' && json_error($info['return_msg'], -2);
$info['result_code'] == 'FAIL' && json_error($info['err_code_des'], -2);
```

## 五、微信企业付款
 ```
$config = [
    'appid' => '',
    'mch_id' => '',
    'key' => '',
    'sslcert_path' => '', //企业付款时必须上传证书
    'sslkey_path' => ''
];
 ```
 #### 1.企业付款到零钱
```
$param = [
    'partner_trade_no' => '', //商户订单号，需保持唯一性(只能是字母或者数字，不能包含有其他字符)
    'openid' => '', //商户appid下，某用户的openid
    'amount' => '', //企业付款金额，单位为分
    'desc' => '', //企业付款备注，必填。注意：备注中的敏感词会被转成字符*
    're_user_name' => '', //收款用户真实姓名，NO_CHECK 时可不传，如果check_name设置为FORCE_CHECK，则必填用户真实姓名
];
  
$wechatMchPay = new WechatMchPay($config);
$info = $wechatMchPay->payChange($param);
$info['return_code'] == 'FAIL' && json_error($info['return_msg'], -2);
$info['result_code'] == 'FAIL' && json_error($info['err_code_des'], -2);
``` 
 #### 2.查询企业付款到零钱
 ```
$partnerTradeNo = ''; //商户订单号
 
$wechatRefund = new WechatRefund($config);
$info = $wechatRefund->queryRefund($partnerTradeNo); 
$info['return_code'] == 'FAIL' && json_error($info['return_msg'], -2);
$info['result_code'] == 'FAIL' && json_error($info['err_code_des'], -2);
 ```
 