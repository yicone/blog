# 充值Q币 #
## 一、充值接口 ##
由充值系统提供给代理商访问。  
**访问地址**：http://*.*.*.*/`qb/pay`
`POST`方式提交以下参数。需采用`UrlEncode`编码(`UTF-8`)。

## 请求参数说明 ##

> <table class="table">
   <tr>
      <td>参数</td>
      <td>说明</td>
      <td>备注</td>
   </tr>
   <tr>
      <td>ClientId</td>
      <td>代理商ID</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>ClientOrderNo</td>
      <td>订单号</td>
      <td>不为空 由数字及字母组成 不大于35位</td>
   </tr>
   <tr>
      <td>PlayerAccount</td>
      <td>QQ号码</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>Amount</td>
      <td>购买数量</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>UserIP</td>
      <td>用户充值IP</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>Sign</td>
      <td>签名</td>
      <td>不为空
	  </td>
   </tr>
</table>

> Sign = MD5(`ClientOrderNo~PlayerAccount~Amount~UserIP~**SignKey**`)  
Hash结果为`大写32位Hex`；SignKey在代理商申请时与ClientId一起分配

## 1. 请求参数构造示例（C#） ##


> string postData = string.Format(“`ClientOrderNo={0}&PlayerAccount={1}&Amount={2}&UserIP ={3}&Sign={4}`”, clientOrderNo, playerAccount, amount, userIP, sign);  
postData = HttpUtility.`UrlEncode`(postData, Encoding.`UTF8`);

## 2. 返回结果示例 ##
1. <`ChargeResult`>  
<`ResultCode`>071<`/ResultCode`>  
<`ResultMessage`>订单重复<`/ResultMessage`>  
<`/ChargeResult`>
2. <`ChargeResult`>  
	<`ResultCode`>000<`/ResultCode`>  
	<`ResultMessage`>成功<`/ResultMessage`>  
<`/ChargeResult`>

## 二、发货确认接口 ##
需由代理商提供。  
接受充值系统以`POST`方式提交的请求。采用`UrlEncode`编码(`UTF-8`)。  
参数：`clientOrderNo`（即代理商调用充值接口时传入的订单号）  
代理商接受到请求后，需用文本方式返回约定的值如下：
> 
<table>
<tr><td>返回码</td><td>备注</td></tr>
<tr><td>000</td><td>发货确认成功</td></tr>
<tr><td>-001</td><td>发货确认失败</td></tr>
<table>
