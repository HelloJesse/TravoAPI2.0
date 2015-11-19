##一、系统级请求格式##
###1.1 服务请求地址 ###


订单服务地址

- 测试环境http://101.231.223.162:9900/order/order.svc
- 正式环境http://api.117lego.com/order/order.svc

商品服务地址:

- 测试环境 http://101.231.223.162:9900/product/product.svc 
- 正式环境 http://api.117lego.com/product/product.svc 


###1.2 请求格式 ###
淘在路上所有请求和响应格式为JSON, 并且可以理解成所有的请求只有下面4个参数。

- AppId:string 商户Id
- Time:long unix时间戳（当前时间300秒后过期)
- Sign:string 签名 MD5(AppId + PrivateKey + Time）小写
- RequestData:string 请求的其它参数 

前3个参数是为了完成验证和授权，商户开发人员在开始对接之前需要找淘在路上开发人员提供AppId 和 PrivateKey，有时候这两个值在正式环境与测试环境可能不同。关于Sign的获取方式可以参数附件提供的代码。

RequestData是每个不同接口请求参数的Json格式，下面我们拿商户取消订单这个接口来举个例子：

测试环境请求地址：http://api.117lego.com/order/order.svc/CancelAppOrder

Reqeust
>{
 ""AppId"": ""1128"", <br/>
 ""Time"": ""2434028090"",<br/>
 ""Sign"": ""fcb626dab1ad02abd0f4ea7ff86b2f63"",<br/>
 ""RequestData"": ""{\""GroupId\"":\""1128\"",\""OrderVoucherList\"":<br/>[{\""OrderId\"":\""85306\"",\""AppVoucherNo\"":[\""12345678\""]},{\""OrderId\"":\""85320\"",\""AppVoucherNo\"":[\""32165498\""]}]}""
}					
>	
		
Response
>"{
 ""Code"": 1,<br/>
 ""IsSuccess"": true,<br/>
 ""Message"": ""订单商户券更新成功"",<br/>
 ""OrderResultList"": [{<br/>
  ""Code"": 1,<br/>
  ""IsSuccess"": true,<br/>
  ""Message"": null,<br/>
  ""OrderId"": ""85306""<br/>
 }, {<br/>
  ""Code"": 1,<br/>
  ""IsSuccess"": true,<br/>
  ""Message"": null,<br/>
  ""OrderId"": ""85320""<br/>
 }],
 ""Guid"": ""c0bf60e7-be83-4e50-be72-30789c9291ef""<br/>
}"					
>
					
不同的接口可能返回值会不能但是下面几个参数是通用的：

- Code:int 返回的处理代码，成功为1，具体的错误代码请参考错误代码列表
- IsSuccess:bool 是否成功
- Message:string 处理消息，处理失败的时候会返回具体的错误消息
- Guid:string 当前请求的唯一标识，根据这个唯一标识可以在淘在路上对接平台的后台查看日志


