此文档包括了淘在路上平台对接商品和订单接口参数和流程的详细定义，以及常见QA(见第九节)。

在开始开发以前建议认真阅读常见QA!

在开始开发以前建议认真阅读常见QA!

在开始开发以前建议认真阅读常见QA!

##目录

- [一、系统级请求格式](#一系统级请求格式)
- [　　1.1 服务请求地址](#11-服务请求地址)
- [　　1.2 请求格式](#12-请求格式)
- [二、应用级接口](#二应用级接口)
- [三、商品相关接口](#三商品相关接口)
- [四、订单相关接口](#四订单相关接口)
- [五、订单业务操作流程](#五订单业务操作流程)
- [六、订单相关接口参数详解](#六订单相关接口参数详解)
- [　　6.1 创建订单](#61-创建订单)
- [　　6.2 付款取消申请](#62-付款取消申请)
- [　　6.3 确认取消订单](#63-确认取消订单)
- [　　6.4 核销订单](#64-核销订单)
- [　　6.5 使用订单](#65-使用订单)
- [　　6.6 获取商户待处理列表](#66-获取商户待处理列表)
- [　　6.7 获取订单详情](#67-获取订单详情)
- [　　6.8 修改订单信息](#68-修改订单信息)
- [　　6.9 设置商户订单标记](#69-设置商户订单标记)
- [　　6.10 设置商户订单商户券](#610-设置商户订单商户券)
- [七、商品相关api接口详情](#七商品相关api接口详情)
- [　　7.1 商品相关实体信息概览](#71商品相关实体信息概览)
- [　　7.2 获取商品列表](#72-获取商品列表)
- [　　7.3 获取商品库存班期价格](#73-获取商品库存班期价格)
- [　　7.4 获取商品库存价格班期](#74-获取商品库存价格班期)
- [　　7.5 更新商品库存价格班期](#75-更新商品库存价格班期)
- [　　7.6 关闭打开班期](#76-关闭打开班期)
- [　　7.7 操作套餐](#77-操作套餐)
- [八、系统级别错误代码](#八系统级别错误代码)
- [九、常见QA](#九常见qa)


##一、系统级请求格式
###1.1 服务请求地址


订单服务地址

- 测试环境http://180.168.78.10/order/order.svc
- 正式环境http://api.117lego.com/order/order.svc

商品服务地址:

- 测试环境 http://180.168.78.10:9900/product/product.svc
- 正式环境 http://api.117lego.com/product/product.svc

###1.2 请求格式
淘在路上所有请求和响应格式为JSON, 并且可以理解成所有的请求只有下面4个参数。

- AppId:string 商户Id
- Time:long unix时间戳: 是从1970年1月1日（UTC/GMT的午夜）开始所经过的秒数（当前时间300秒后过期)
- Sign:string 签名 MD5(AppId+PrivateKey+Time）小写
- RequestData:string 请求的其它参数

前3个参数是为了完成验证和授权，商户开发人员在开始对接之前需要找淘在路上开发人员提供AppId 和 PrivateKey，有时候这两个值在正式环境与测试环境可能不同。关于Sign的获取方式可以参数附件提供的代码。

RequestData是每个不同接口请求参数的Json格式，下面我们拿商户取消订单这个接口来举个例子：

测试环境请求地址：http://180.168.78.10:9900/order/order.svc/CancelAppOrder

Reqeust
<pre>
{
 "AppId": "1128",
 "Time": ""2434028090",
 "Sign": ""fcb626dab1ad02abd0f4ea7ff86b2f63",
 "RequestData": "{\"GroupId\":\"1128\",\"OrderVoucherList\":[{\"OrderId\":\"85306\",\"AppVoucherNo\":[\"12345678\"]},{\""OrderId\":\"85320\",\"AppVoucherNo\":[\"32165498\"]}]}"
}					
</pre>				


Response
<pre>
{
 "Code": 1,
 "IsSuccess": true,
 "Message": "订单商户券更新成功",
 "OrderResultList": [{
  "Code"": 1,
  "IsSuccess": true,
  "Message": null,
  "OrderId": ""85306""
 }, {
  "Code": 1,
  "IsSuccess": true,
  "Message": null,
  "OrderId": "85320"
 }],
 "Guid": "c0bf60e7-be83-4e50-be72-30789c9291ef"
}
</pre>

不同的接口可能返回值会不同，但是下面几个参数是通用的：

- Code:int 返回的处理代码，成功为1，具体的错误代码请参考错误代码列表
- IsSuccess:bool 是否成功
- Message:string 处理消息，处理失败的时候会返回具体的错误消息
- Guid:string 当前请求的唯一标识，根据这个唯一标识可以在淘在路上对接平台的后台查看日志


##二、应用级接口
淘在路上平台对接API从业务上来分类可以分为两种

- 订单
- 商品

商品下又可以分为以下四小类：

- 库存、价格、班期
- 基本信息
- 行程段
- 套餐

从技术对接的角度来可以分为查询与更新两种；更新按照调用方向又可以分为商户主动通知（淘在路上提供API）和淘在路上平台推送（商户提供API）。

##三、商品相关接口
与商品相关的接口相对来说会比较容易理解，因为全部是淘在路上提供，商户只需要负责调用。传入或者接收对应的参数，把商户的参数和淘在路上的参数匹配上即可，后面我们会对这些参数进行详细说明。
<table class="gridtable">
   <tr>
      <th>接口名称</th>
      <th>说明</th>
      <th>业务分类</th>
      <th>技术分类</th>
   </tr>
   <tr>
      <td> - </td>
      <td>获取指定时间的产品库存,价格，库存</td>
      <td> 库存</td>
      <td>查询</td>
   </tr>
   <tr>
      <td> - </td>
      <td>更新商品库存、价格、班期</td>
      <td> 班期</td>
      <td>更新</td>
   </tr>
   <tr>
      <td>OperateScheduleDate</td>
      <td>关闭/打开指定套餐班期</td>
      <td> 班期</td>
      <td>更新</td>
   </tr>
   <tr>
      <td>GetProductList</td>
      <td>输出商品列表</td>
      <td>产品列表</td>
      <td>查询</td>
   </tr>
   <tr>
      <td>GetProductSegmentList</td>
      <td>商品行程段Schema列表</td>
      <td> 行程段</td>
      <td>查询</td>
   </tr>
   <tr>
      <td>OperateProductSegment</td>
      <td>新增/更新商品行程段信息</td>
      <td> 行程段</td>
      <td>更新</td>
   </tr>
   <tr>
      <td> OperateProductItem</td>
      <td>新增/更新套餐</td>
      <td> 套餐</td>
      <td>更新</td>
   </tr>
</table>

##四、订单相关接口
与订单相关的接口会比商品相关的接口复杂一些，因为涉及到具体的业务流程：比如退款，核销等。并且不完全是淘在路上提供商户调用的这种单一模式。有些操作可能需要互相调用三次才能完成，所以我们会对订单操作的流程和接口做更详细的介绍。

<table class="gridtable">
   <tr>
      <th>接口名称</th>
      <th>说明</th>
      <th>技术分类</th>
      <th>提供方</th>
   </tr>
   <tr>
      <td>GetAppOrderList</td>
      <td>商户获取待处理订单列表</td>
      <td>查询</td>
      <td>淘在路上</td>
   </tr>
   <tr>
      <td>GetAppOrderDetail</td>
      <td>商户获取订单详情</td>
      <td>查询</td>
      <td>淘在路上</td>
   </tr>
   <tr>
      <td>UpdateAppOrder</td>
      <td>商户订单核销</td>
      <td>更新</td>
      <td>淘在路上</td>
   </tr>
   <tr>
      <td>UpdateOrderInfo</td>
      <td>供应商修改订单信息</td>
      <td>更新</td>
      <td>淘在路上</td>
   </tr>
   <tr>
      <td>SetAppOrderColor</td>
      <td>设置商户订单标记</td>
      <td>更新</td>
      <td>淘在路上</td>
   </tr>
   <tr>
      <td>SetAppOrderVoucher</td>
      <td>设置商户订单商户券</td>
      <td>更新</td>
      <td>淘在路上</td>
   </tr>
   <tr>
      <td>创建订单</td>
      <td>去商户系统下单</td>
      <td>更新</td>
      <td>商户 </td>
   </tr>
   <tr>
      <td>CancelAppOrder</td>
      <td>商户取消订单（已付款）</td>
      <td>更新</td>
      <td>淘在路上</td>
   </tr>
   <tr>
      <td>订单付款/取消推送</td>
      <td>将订单付款/取消的Schema推送给商户</td>
      <td>更新</td>
      <td>商户 </td>
   </tr>
<tr>
      <td>UseAppOrder</td>
      <td>使用订单</td>
      <td>更新</td>
      <td>商户 </td>
   </tr>
</table>

与订单相关的接口一共有10个，其中有2个查询（GetAppOrderList，GetAppOrderDetail) 以及3个简单的更新操作(UpdateOrderInfo, SetAppOrderColor, SetAppOrderVoucher)都是不被包含在订单的业务操作流程当中的。那么除去这5个接口以外，还剩下5个与订单业务操作流程相关的。

##五、订单业务操作流程
与订单业务流程相关的操作包括：

- 创建订单
- 订单付款
- 已付款取消（包括取消申请和确认取消两个操作)
- 核销（包括预约和使用两个操作)
- 使用 (免预约，直接将订单设置为已使用)

###5.1 创建订单
商户需要为创建订单提供一个接口给淘在路上平台调用，虽然只需要一个接口，但是流程上却可以分为两个不同的流程： 付款前推送下单和付款后推送下单。

**付款前下单**
![](https://github.com/HelloJesse/TravoAPI2.0/raw/master/images/createOrder.png)

**付款后下单**
![](https://github.com/HelloJesse/TravoAPI2.0/raw/master/images/createOrderafterpay.png)

- 接口提供方： 商户
- 接口调用发起方：淘在路上
- 接口名称： 创建订单（商户自定义, 需要提供接口地址给淘在路上)


对于付款前下单和付款后下单，接口的参数定义是一样的。付款前下单即用户在淘在路上app创建订单的时候会立即将订单推送到商户（商户在自己的系统中创建订单)，这是一个**同步**的过程。 而付款后下单即用户在淘在路上app上下单并且完成付款之后，才会将下单信息推送给商户下单。

这里需要商户提供一个下单的接口，淘在路上会将订单的参数封装好调用商户提供的这个下单接口，也就是上面订单相关接口列表中的<**创建订单**>，名称可以由商户来定义，只需要保证输入参数和输出参数和淘在路上接口中定义的保持一致即可。不管是付款前下单还是付款后下单，这个接口的参数对于商户来说都是一样的。

**差异**

- 付款前下单是同步调用，失败或者成功一次即知，如果失败了会告诉APP用户下单失败
- 付款后下单是异步调用，该订单在淘在路上已经创建成功，淘在路上异步通知商户下单，会有相关的重试机制、预警邮件通知到相关人员手动处理，以及立即取消淘在路上订单（因为客户已经付款，如果商户下单失败，则该笔订单不可用，为避免对用户产生更恶劣的后果，在没有其它处理办法的情况下，淘在路上会取消该订单，将费用退还给用户)


###5.2 订单付款与付款后取消申请

首先需要说明的是：订单付款这个操作，只会发生在走“付款前下单”流程的订单上。之所以将这两个接口放在一起是因为它们都是对于订单状态变更的通知。淘在路上会将变之后新的状态推送给商户，商户针对新的订单状态来做相应处理。现在会推给商户4个状态：

- 5：已付款待配送
- 6: 已成交 (不需要配送的订单)
- 10: 取消申请
- 7: 已取消

![](https://github.com/HelloJesse/TravoAPI2.0/raw/master/images/orderstatuschange.png)
- 接口提供方： 商户
- 接口调用发起方：淘在路上
- 接口名称： 付款/申请取消（商户自定义名称，需要提供接口地址给淘在路上)

###5.3 订单确认取消
用户在淘在路上APP取消订单订单之后，订单的状态变为：10(取消申请)，商户可以调用淘在路上接口CancelAppOrder 来完成**取消订单**的操作，在调用这个接口的时候，可以决定具体的退款金额，下面是用户取消订单的整个流程。

![](https://github.com/HelloJesse/TravoAPI2.0/raw/master/images/cancelorder.png)

这个流程里面会涉及到2个接口：

**取消申请**

- 接口提供方： 商户
- 接口调用发起方：淘在路上
- 接口名称： 付款/申请取消（商户自定义名称，需要提供接口地址给淘在路上)

**商户确认取消**

- 接口提供方： 淘在路上
- 接口调用发起方：商户
- 接口名称： CancelAppOrder

###5.4 订单核销
订单核销分为预约和使用，但都是调用同一个接口。

![](https://github.com/HelloJesse/TravoAPI2.0/raw/master/images/updateAppOrder.png)

- 接口提供方： 淘在路上
- 接口调用发起方：商户
- 接口名称： UpdateAppOrder


###5.5 订单使用
在3.4 的订单核销接口中，订单需要先预约之后才能使用，而这个订单使用的接口则不需要预约即可以将订单的状态设置为已使用。

![](https://github.com/HelloJesse/TravoAPI2.0/raw/master/images/useOrder.png)


- 接口提供方： 淘在路上
- 接口调用发起方：商户
- 接口名称： UseAppOrder

###5.6 订单业务流程操作相关接口总览###

<table class="gridtable">
   <tr>
      <th>接口名称 </td>
      <th>说明</th>
      <th>提供方</th>
      <th>调用方</th>
   </tr>
   <tr>
      <td>创建订单</td>
      <td>商户下单</td>
      <td>商户 </td>
      <td>淘在路上</td>
   </tr>
   <tr>
      <td>付款/取消申请</td>
      <td>商户更新订单状态 </td>
      <td>商户 </td>
      <td>淘在路上</td>
   </tr>
   <tr>
      <td>CancelAppOrder</td>
      <td>确认取消订单</td>
      <td>淘在路上</td>
      <td>商户 </td>
   </tr>
   <tr>
      <td>UpdateAppOrder</td>
      <td>核销订单</td>
      <td>淘在路上</td>
      <td>商户 </td>
   </tr>
   <tr>
      <td>UseAppOrder</td>
      <td>使用订单</td>
      <td>淘在路上</td>
      <td>商户 </td>
   </tr>
</table>


##六、订单相关接口参数详解##

###6.1 创建订单

该服务由商户提供，商户开发人员需要告知淘在路上开发人员服务的地址，以便进行联调

####请求消息
<table class="gridtable">
   <tr>
      <th>RequestData</th>
      <td>OrderBasicDTO</td>
   </tr>
   <tr>
      <th>Response</th>
      <td>ResponseOrderDTO</td>
   </tr>
</table>

####OrderBasicDTO

<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>AppId</td>
      <td>string</td>
      <td>商户Id（平台分配）</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>Time</td>
      <td>long</td>
      <td>unix时间</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>Sign</td>
      <td>string</td>
      <td>签名（MD5）</td>
      <td>Y</td>
      <td>MD5（AppId+PrivateKey+Time）小写</td>
   </tr>
   <tr>
      <td>OrderId</td>
      <td>long</td>
      <td>订单号</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>GroupProductId</td>
      <td>string</td>
      <td>商户商品ID</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>AppItemId</td>
      <td>string</td>
      <td>商户套餐ID</td>
      <td>N</td>
      <td></td>
   </tr>
   <tr>
      <td>OrderPrice</td>
      <td>decimal</td>
      <td>订单总金额</td>
      <td>Y</td>
      <td>订单总金额 = 订单应付金额 + 优惠促销金额</td>
   </tr>
   <tr>
      <td>TotalPrice</td>
      <td>decimal</td>
      <td>订单应付金额</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>CouponAmount</td>
      <td>decimal</td>
      <td>优惠促销金额</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>OrderDate</td>
      <td>datetime</td>
      <td>订单日期</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>DepartureDate</td>
      <td>datetime</td>
      <td>出发\使用日期</td>
      <td>N</td>
      <td>可以为空；如果为空，则取有效起始日期至有效截止日期这个时间段</td>
   </tr>
   <tr>
      <td>ReturnDate</td>
      <td>datetime</td>
      <td>返回日期</td>
      <td>N</td>
      <td></td>
   </tr>
   <tr>
      <td>TotalQuantity</td>
      <td>int</td>
      <td>商品总数（总份数）</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>SaleMode</td>
      <td>int</td>
      <td>商品售卖模式</td>
      <td>Y</td>
      <td>不为空（1:直售（指定班期）、2：预售（线下预约）、3：预售（在线预约））</td>
   </tr>
   <tr>
      <td>TotalCount</td>
      <td>int</td>
      <td>总人数</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>TotalAdultQuantity</td>
      <td>int</td>
      <td>成人数</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>TotalChildQuantity</td>
      <td>int</td>
      <td>儿童数</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>Price</td>
      <td>decimal</td>
      <td>商品卖价</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>PresaleStartDate</td>
      <td>datetime</td>
      <td>使用开始时间</td>
      <td>N</td>
      <td>可以为空（预售时传入）</td>
   </tr>
   <tr>
      <td>PresaleEndDate</td>
      <td>datetime</td>
      <td>使用结束时间</td>
      <td>N</td>
      <td>可以为空（预售时传入）</td>
   </tr>
   <tr>
      <td>DepartCarInfo</td>
      <td>string</td>
      <td>接送信息</td>
      <td>N</td>
      <td></td>
   </tr>
   <tr>
      <td>ContactInfo</td>
      <td>ContactDTO</td>
      <td>订单联系人信息</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>PassengerList</td>
      <td>List<PassengerDTO></td>
      <td>订单旅客信息列表</td>
      <td>N</td>
      <td></td>
   </tr>
   <tr>
      <td>ExpandInfo</td>
      <td>string</td>
      <td>OrderExpandDTO对象的序列化</td>
      <td>N</td>
      <td>可以不传，作为备用扩展字段</td>
   </tr>
</table>

####ContactDTO####

<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>ContactName</td>
      <td>string</td>
      <td>联系人名称</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>ContactMobile</td>
      <td>string</td>
      <td>联系人手机</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>ContactEmail</td>
      <td>string</td>
      <td>联系人邮箱</td>
      <td>Y</td>
      <td></td>
   </tr>
</table>

####PassengerDTO####

<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>Name</td>
      <td>string</td>
      <td>旅客姓名</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>Gender</td>
      <td>string</td>
      <td>旅客性别</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>Type</td>
      <td>int</td>
      <td>旅客类型</td>
      <td>Y</td>
      <td>不为空；1-成人;2-儿童</td>
   </tr>
   <tr>
      <td>BirthDate</td>
      <td>datetime</td>
      <td>生日</td>
      <td>N</td>
      <td></td>
   </tr>
   <tr>
      <td>CertificateType</td>
      <td>int</td>
      <td>证件类型</td>
      <td>Y</td>
      <td>不为空；1-身份证;2-护照;3-军官证;4-回乡证;5-台胞证;6-港澳通行证;7-台湾通行证</td>
   </tr>
   <tr>
      <td>CertificateNumber</td>
      <td>string</td>
      <td>证件号码</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>Nationality</td>
      <td>string</td>
      <td>国籍</td>
      <td>N</td>
      <td></td>
   </tr>
   <tr>
      <td>IssueCity</td>
      <td>string</td>
      <td>签发地</td>
      <td>N</td>
      <td></td>
   </tr>
   <tr>
      <td>IssueDate</td>
      <td>datetime</td>
      <td>签发日期</td>
      <td>N</td>
      <td></td>
   </tr>
   <tr>
      <td>PassportType</td>
      <td>int</td>
      <td>护照类型</td>
      <td>N</td>
      <td>可以为空；0-普通因私护照;1-外交护照;2-公务护照</td>
   </tr>
   <tr>
      <td>PassportDatelimit</td>
      <td>datetime</td>
      <td>护照有效期</td>
      <td>N</td>
      <td></td>
   </tr>
   <tr>
      <td>ContactAddress</td>
      <td>string</td>
      <td>联系地址</td>
      <td>N</td>
      <td></td>
   </tr>
</table>

####OrderExpandDTO####

<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>ProductId</td>
      <td>int</td>
      <td>商品ID</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>ProductName</td>
      <td>string</td>
      <td>商品名称</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>ProductItemId</td>
      <td>int</td>
      <td>商品套餐ID</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>ProductItemName</td>
      <td>string</td>
      <td>商品套餐名称</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>OriginalPrice</td>
      <td>decimal</td>
      <td>商品原价</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>VendorCouponAmount</td>
      <td>decimal</td>
      <td>商户承担优惠金额</td>
      <td>N</td>
      <td></td>
   </tr>
</table>

####ResponseOrderDTO####

<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>IsSuccess</td>
      <td>bool</td>
      <td>是否成功</td>
      <td>Y</td>
      <td>不为空；提示调用返回是否成功</td>
   </tr>
   <tr>
      <td>Code</td>
      <td>bool</td>
      <td>编码</td>
      <td>Y</td>
      <td>不为空；返回的调用结果编码</td>
   </tr>
   <tr>
      <td>Message</td>
      <td>string</td>
      <td>提示信息</td>
      <td>Y</td>
      <td>不为空；返回的调用结果信息</td>
   </tr>
   <tr>
      <td>OrderInfo</td>
      <td>OrderInfoDTO</td>
      <td>订单信息</td>
      <td>Y</td>
      <td>不为空；</td>
   </tr>
</table>

####OrderInfoDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>OrderId</td>
      <td>long</td>
      <td>订单号</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>AppOrderId</td>
      <td>string</td>
      <td>商户订单号 </td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>WaitPaymentTime</td>
      <td>datetime</td>
      <td>最晚支付时间</td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
</table>


###6.2 付款/取消申请###

该服务由商户提供，商户开发人员需要告知淘在路上开发人员服务的地址，以便进行联调

####请求消息####
<table class="gridtable">
   <tr>
      <th>RequestData</th>
      <td>OrderPayDTO</td>
   </tr>
   <tr>
      <th>Response</th>
      <td>ResponseDTO</td>
   </tr>
</table>

####OrderPayDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>AppId</td>
      <td>string</td>
      <td>商户Id（平台分配）</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>Time</td>
      <td>long</td>
      <td>unix时间</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>Sign</td>
      <td>string</td>
      <td>签名（MD5）</td>
      <td>Y</td>
      <td>MD5（AppId + PrivateKey + Time）小写</td>
   </tr>
   <tr>
      <td>OrderId</td>
      <td>long</td>
      <td>订单号</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>AppOrderId</td>
      <td>string</td>
      <td>商户订单ID </td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
   <tr>
      <td>OrderStatus</td>
      <td>int</td>
      <td>订单状态</td>
      <td>Y</td>
      <td> （需要配送的订单付款后为5）5：已付款，（不需要配送的订单付款后为6）6：已成交（用户已付款发起取消申请后的状态为10）10：取消申请，（用户未付款发起取消订单后的状态为7）7：已取消"</td>
   </tr>
   <tr>
      <td>ReMark</td>
      <td>string</td>
      <td>订单备注</td>
      <td>Y</td>
      <td>说明</td>
   </tr>
   <tr>
      <td>Guid</td>
      <td>string</td>
      <td>唯一标识</td>
      <td>Y</td>
      <td></td>
   </tr>
</table>

####ResponseDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>IsSuccess</td>
      <td>bool</td>
      <td>是否成功</td>
      <td>Y</td>
      <td>不为空；提示调用返回是否成功</td>
   </tr>
   <tr>
      <td>Code</td>
      <td>bool</td>
      <td>编码</td>
      <td>Y</td>
      <td>不为空；返回的调用结果编码</td>
   </tr>
   <tr>
      <td>Message</td>
      <td>string</td>
      <td>提示信息</td>
      <td>Y</td>
      <td>不为空；返回的调用结果信息</td>
   </tr>
</table>

###6.3 确认取消订单###

- 测试地址：http://10.1.25.61:9900/order/order.svc/UpdateAppOrder
- 正式地址：http://api.117lego.com/order/order.svc/UpdateAppOrder
- 调用方式: POST

####请求消息####
<table class="gridtable">
   <tr>
      <th>RequestData</th>
      <td>CancelOrderDTO</td>
   </tr>
   <tr>
      <th>Response</th>
      <td>OrderProcessResponseDTO</td>
   </tr>
</table>

####CancelOrderDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>GroupId</td>
      <td>string</td>
      <td>商户ID</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>OrderId</td>
      <td>string</td>
      <td>订单号</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>IsAppOrder</td>
      <td>bool</td>
      <td>是否商户订单</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>Uid</td>
      <td>string</td>
      <td>操作人Id</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>Reason</td>
      <td>string</td>
      <td>取消原因</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>OperateType</td>
      <td>int</td>
      <td>操作类型</td>
      <td>Y</td>
      <td>操作类型，0：确认取消；1：取消回滚</td>
   </tr>
   <tr>
      <td>RefundAmount</td>
      <td>decimal</td>
      <td>退款金额</td>
      <td>Y</td>
      <td>全额退款 = 订单总金额 ,部份退款 = 部份退款的金额(部份退款的金额必须大于等于该订单所使用优惠券的金额)</td>
   </tr>
</table>

####OrderProcessResponseDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>IsSuccess</td>
      <td>bool</td>
      <td>是否成功</td>
      <td>Y</td>
      <td>不为空；提示调用返回是否成功</td>
   </tr>
   <tr>
      <td>Code</td>
      <td>int</td>
      <td>编码</td>
      <td>Y</td>
      <td>不为空；返回的调用结果编码</td>
   </tr>
   <tr>
      <td>Message</td>
      <td>string</td>
      <td>提示信息</td>
      <td>Y</td>
      <td>不为空；返回的调用结果信息</td>
   </tr>
   <tr>
      <td>Guid</td>
      <td>string</td>
      <td>唯一标识</td>
      <td>Y</td>
      <td>不为空；返回的调用唯一标识</td>
   </tr>
</table>

###6.4 核销订单###

- 测试地址：http://10.1.25.61:9900/order/order.svc/UpdateAppOrder
- 正式地址：http://api.117lego.com/order/order.svc/UpdateAppOrder
- 调用方式: POST

####请求消息####
<table class="gridtable">
   <tr>
      <th>RequestData</th>
      <td>UpdateAppOrderParaDTO</td>
   </tr>
   <tr>
      <th>Response</th>
      <td>OrderResultDTO</td>
   </tr>
</table>

####UpdateAppOrderParaDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>GroupId</td>
      <td>string</td>
      <td>商户ID</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>AppOrderData</td>
      <td>List< AppOrderDTO > </td>
      <td>订单信息实体列表</td>
      <td>Y</td>
      <td></td>
   </tr>
</table>

####AppOrderDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>OrderId</td>
      <td>long</td>
      <td>平台订单Id</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>AppOrderId</td>
      <td>string</td>
      <td>商户订单Id</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>AppOrderStatus</td>
      <td>int</td>
      <td>平台订单状态</td>
      <td>Y</td>
      <td>6：已预约；9：已使用；7：已取消</td>
   </tr>
   <tr>
      <td>AppVoucher</td>
      <td>List< AppVoucherDTO ></td>
      <td>商户券号</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>AppDelivery</td>
      <td>AppDeliveryDTO</td>
      <td>配送信息</td>
      <td>N</td>
      <td>如需配送,不能为空.</td>
   </tr>
</table>

####AppVoucherDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>AppVoucherNo</td>
      <td>string</td>
      <td>商户券号</td>
      <td>Y</td>
      <td>商户券号</td>
   </tr>
   <tr>
      <td>AppVoucherUseTime</td>
      <td>DateTime</td>
      <td>商户券号使用时间</td>
      <td>N</td>
      <td>商户券号预约使用时间（票券类的产品一定要传入）</td>
   </tr>
   <tr>
      <td>AppVoucherStatus</td>
      <td>int</td>
      <td>商户券状态</td>
      <td>Y</td>
      <td>1：已预约；2：已使用；0：已取消</td>
   </tr>
</table>

####AppDeliveryDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>DeliveryType</td>
      <td>int</td>
      <td>配送类型</td>
      <td>Y</td>
      <td>0,虚拟配送.1,实物配送</td>
   </tr>
   <tr>
      <td>CourierNumber</td>
      <td>string</td>
      <td>快递单号</td>
      <td>N</td>
      <td>虚拟配送时,可为空.</td>
   </tr>
   <tr>
      <td>CourierCorpName</td>
      <td>string</td>
      <td>快递公司</td>
      <td>N</td>
      <td>虚拟配送时,可为空.如实物配送请至少选择下列快递公司中的一个 [中铁快运、EMS快递、申通快递、顺丰速递、圆通快递、中通速递、宅急送、韵达快递、全峰快递、淘在路上]</td>
   </tr>
</table>

####OrderResultDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>OrderId</td>
      <td>long</td>
      <td>订单号</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>IsSuccess</td>
      <td>bool</td>
      <td>是否成功</td>
      <td>Y</td>
      <td>不为空；提示调用返回是否成功</td>
   </tr>
   <tr>
      <td>Code</td>
      <td>int</td>
      <td>编码</td>
      <td>Y</td>
      <td>不为空；返回的调用结果编码</td>
   </tr>
   <tr>
      <td>Message</td>
      <td>string</td>
      <td>提示信息</td>
      <td>Y</td>
      <td>不为空；返回的调用结果信息</td>
   </tr>
</table>


###6.5 使用订单###

- 测试地址：http://10.1.25.61:9900/order/order.svc/UseAppOrder
- 正式地址：http://api.117lego.com/order/order.svc/UseAppOrder
- 调用方式: POST

####请求消息####
<table class="gridtable">
   <tr>
      <th>RequestData</th>
      <td>UseAppOrderRequest</td>
   </tr>
   <tr>
      <th>Response</th>
      <td>ResponseDTO</td>
   </tr>
</table>

####UseAppOrderRequest####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>GroupId</td>
      <td>string</td>
      <td>商户ID</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>OrderId</td>
      <td>long</td>
      <td>平台订单ID</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>TotalQuantity</td>
      <td>int</td>
      <td>订单份数</td>
      <td>Y</td>
      <td></td>
   </tr>
</table>

####ResponseDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>IsSuccess</td>
      <td>bool</td>
      <td>是否成功</td>
      <td>Y</td>
      <td>不为空；提示调用返回是否成功</td>
   </tr>
   <tr>
      <td>Code</td>
      <td>int</td>
      <td>编码</td>
      <td>Y</td>
      <td>不为空；返回的调用结果编码</td>
   </tr>
   <tr>
      <td>Message</td>
      <td>string</td>
      <td>提示信息</td>
      <td>Y</td>
      <td>不为空；返回的调用结果信息</td>
   </tr>
  <tr>
      <td>Guid</td>
      <td>string</td>
      <td>订单唯一标识</td>
      <td>Y</td>
      <td>不为空;返回请求唯一标识</td>
   </tr>
</table>

###6.6 获取商户待处理列表###

- 测试地址：http://10.1.25.61:9900/order/order.svc/GetAppOrderList
- 正式地址：http://api.11
- 7lego.com/order/order.svc/GetAppOrderList
- 调用方式: POST

####请求消息####
<table class="gridtable">
   <tr>
      <th>RequestData</th>
      <td>AppOrderListDate</td>
   </tr>
   <tr>
      <th>Response</th>
      <td>AppOrderListResponseDTO</td>
   </tr>
</table>

####AppOrderListDate####

<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>GroupId</td>
      <td>int</td>
      <td>商户ID</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>OrderDateFrom</td>
      <td>string</td>
      <td>请求开始时间</td>
      <td>Y</td>
      <td>"如订单未付款，则此时间为下单时间、如订单已付款，则此时间为付款时间、如订单已取消，则此事件为商户取消订单的时间"</td>
   </tr>
   <tr>
      <td>OrderDateTo</td>
      <td>string</td>
      <td>请求结束时间</td>
      <td>N</td>
      <td>如为空则默认为当前时间，其逻辑参考上述请求开始时间栏位</td>
   </tr>
</table>

####AppOrderListResponseDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>IsSuccess</td>
      <td>bool</td>
      <td>是否成功</td>
      <td>Y</td>
      <td>不为空；提示调用返回是否成功</td>
   </tr>
   <tr>
      <td>Code</td>
      <td>int</td>
      <td>编码</td>
      <td>Y</td>
      <td>不为空；返回的调用结果编码</td>
   </tr>
   <tr>
      <td>Message</td>
      <td>string</td>
      <td>提示信息</td>
      <td>Y</td>
      <td>不为空；返回的调用结果信息</td>
   </tr>
   <tr>
      <td>Guid</td>
      <td>string</td>
      <td>唯一标识</td>
      <td>Y</td>
      <td>不为空；返回的调用唯一标识</td>
   </tr>
   <tr>
      <td>OrderList</td>
      <td>List< OrderBasicDTO ></td>
      <td>订单列表</td>
      <td>N</td>
      <td>如果没有可以为空</td>
   </tr>
</table>

####ContactDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>ContactName</td>
      <td>string</td>
      <td>联系人名称</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>ContactMobile</td>
      <td>string</td>
      <td>联系人手机</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>ContactEmail</td>
      <td>string</td>
      <td>联系人邮箱</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
</table>

####PassengerDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>Name</td>
      <td>string</td>
      <td>旅客姓名</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>Gender</td>
      <td>string</td>
      <td>旅客性别</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>Type</td>
      <td>int</td>
      <td>旅客类型</td>
      <td>Y</td>
      <td>不为空；1-成人;2-儿童</td>
   </tr>
   <tr>
      <td>BirthDate</td>
      <td>DateTime</td>
      <td>生日</td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
   <tr>
      <td>CertificateType</td>
      <td>int</td>
      <td>证件类型</td>
      <td>Y</td>
      <td>不为空；1-身份证;2-护照;3-军官证;4-回乡证;5-台胞证;6-国际海员证;7-其他</td>
   </tr>
   <tr>
      <td>CertificateNumber</td>
      <td>string</td>
      <td>证件号码</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>Nationality</td>
      <td>string</td>
      <td>国籍</td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
   <tr>
      <td>IssueCity</td>
      <td>string</td>
      <td>签发地</td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
   <tr>
      <td>IssueDate</td>
      <td>DateTime</td>
      <td>签发日期</td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
   <tr>
      <td>PassportType</td>
      <td>int</td>
      <td>护照类型</td>
      <td>N</td>
      <td>可以为空；0-普通因私护照;1-外交护照;2-公务护照</td>
   </tr>
   <tr>
      <td>PassportDatelimit</td>
      <td>DateTime</td>
      <td>护照有效期</td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
   <tr>
      <td>ContactAddress</td>
      <td>string</td>
      <td>联系地址</td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
</table>


####OrderDeliveryDTO####


<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>Receiver</td>
      <td>string</td>
      <td>收件人</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>ContactPhone</td>
      <td>string</td>
      <td>联系电话</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>ZipCode</td>
      <td>string</td>
      <td>邮编</td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
   <tr>
      <td>Address</td>
      <td>string</td>
      <td>配送地址</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>CourierCorpName</td>
      <td>string</td>
      <td>快递公司</td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
   <tr>
      <td>CourierNumber</td>
      <td>string</td>
      <td>快递单号</td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
</table>

####OrderBasicDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>OrderId</td>
      <td>long</td>
      <td>订单号</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>AppOrderId</td>
      <td>string</td>
      <td>关联商户订单号</td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
   <tr>
      <td>GroupProductId</td>
      <td>string</td>
      <td>商户商品名</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>ProductId</td>
      <td>int</td>
      <td>订单商品ID</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>ProductName</td>
      <td>string</td>
      <td>订单商品名称</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>ProductItemId</td>
      <td>int</td>
      <td>订单商品套餐ID</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>ProductItemName</td>
      <td>string</td>
      <td>订单商品套餐名称</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>AppItemId</td>
      <td>string</td>
      <td>商户套餐ID</td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
   <tr>
      <td>TotalPrice</td>
      <td>decimal</td>
      <td>订单总卖价</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>CouponAmount</td>
      <td>decimal</td>
      <td>优惠促销金额</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>RealGet</td>
      <td>decimal</td>
      <td>订单实收金额</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>OrderDate</td>
      <td>DateTime</td>
      <td>订单日期</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>DepartureDate</td>
      <td>DateTime</td>
      <td>出发\使用日期</td>
      <td>N</td>
      <td>可以为空；如果为空，则取有效起始日期至有效截止日期这个时间段</td>
   </tr>
   <tr>
      <td>OrderStatus</td>
      <td>int</td>
      <td>订单状态</td>
      <td>Y</td>
      <td>2-待确认、3-已确认、5-已付款、6-已成交、7-已取消(已付款)</td>
   </tr>
   <tr>
      <td>AppointStatus</td>
      <td>int</td>
      <td>预约状态</td>
      <td>Y</td>
      <td>不为空（1：未预约；2：待确认预约；3：已确认预约；4：已拒绝预约）</td>
   </tr>
   <tr>
      <td>ReturnDate</td>
      <td>DateTime</td>
      <td>返回日期</td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
   <tr>
      <td>TotalQuantity</td>
      <td>int</td>
      <td>产品总数（总份数）</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>TotalCount</td>
      <td>int</td>
      <td>总人数</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>TotalAdultQuantity</td>
      <td>int</td>
      <td>成人数</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>TotalChildQuantity</td>
      <td>int</td>
      <td>儿童数</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>Price</td>
      <td>decimal</td>
      <td>产品卖价</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>ProcessTime</td>
      <td>DateTime</td>
      <td>付款时间</td>
      <td>Y</td>
      <td>可以为空</td>
   </tr>
   <tr>
      <td>PriceAdjust</td>
      <td>decimal</td>
      <td>调整金额</td>
      <td>Y</td>
      <td>可以为空</td>
   </tr>
   <tr>
      <td>PresaleStartDate</td>
      <td>DateTime</td>
      <td>预售开始时间</td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
   <tr>
      <td>PresaleEndDate</td>
      <td>DateTime</td>
      <td>预售截止时间</td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
   <tr>
      <td>DepartCarInfo</td>
      <td>string</td>
      <td>接送信息</td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
   <tr>
      <td>ContactInfo</td>
      <td>ContactDTO</td>
      <td>订单联系人信息</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>PassengerList</td>
      <td>List<PassengerDTO></td>
      <td>订单旅客信息列表</td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
   <tr>
      <td>DeliveryInfo</td>
      <td>OrderDeliveryDTO</td>
      <td>配送信息</td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
   <tr>
      <td>AppItemId</td>
      <td>string</td>
      <td>商户套餐ID</td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
</table>

###6.7 获取订单详情###

- 测试地址：http://10.1.25.61:9900/order/order.svc/GetAppOrderDetail
- 正式地址：http://api.117lego.com/order/order.svc/GetAppOrderDetail
- 调用方式: POST

####请求消息####
<table class="gridtable">
   <tr>
      <th>RequestData</th>
      <td>AppOrderDetailRequestDTO</td>
   </tr>
   <tr>
      <th>Response</th>
      <td>AppOrderDetailResponseDTO</td>
   </tr>
</table>

####AppOrderDetailRequestDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>OrderId</td>
      <td>long</td>
      <td>订单ID</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>IsAppOrderId</td>
      <td>bool</td>
      <td>是否app订单id</td>
      <td>Y</td>
      <td></td>
   </tr>
</table>

####AppOrderDetailResponseDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>IsSuccess</td>
      <td>bool</td>
      <td>是否成功</td>
      <td>Y</td>
      <td>不为空；提示调用返回是否成功</td>
   </tr>
   <tr>
      <td>Code</td>
      <td>bool</td>
      <td>编码</td>
      <td>Y</td>
      <td>不为空；返回的调用结果编码</td>
   </tr>
   <tr>
      <td>Message</td>
      <td>string</td>
      <td>提示信息</td>
      <td>Y</td>
      <td>不为空；返回的调用结果信息</td>
   </tr>
   <tr>
      <td>Guid</td>
      <td>string</td>
      <td>唯一标识</td>
      <td>Y</td>
      <td>不为空；返回的调用唯一标识</td>
   </tr>
   <tr>
      <td>OrderDetail</td>
      <td>OrderBasicDTO</td>
      <td>订单详情</td>
      <td>N</td>
      <td>如果没有可以为空</td>
   </tr>
</table>

####OrderBasicDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>OrderId</td>
      <td>long</td>
      <td>订单号</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>GroupProductId</td>
      <td>string</td>
      <td>商户的产品id</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>ProductName</td>
      <td>string</td>
      <td>订单产品名称</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>ProductItemName</td>
      <td>string</td>
      <td>订单产品套餐名称</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>TotalPrice</td>
      <td>decimal</td>
      <td>订单应付金额</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>CouponAmount</td>
      <td>decimal</td>
      <td>优惠促销金额</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>RealGet</td>
      <td>decimal</td>
      <td>订单实收金额</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>OrderDate</td>
      <td>DateTime</td>
      <td>订单日期</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>DepartureDate</td>
      <td>DateTime</td>
      <td>出发\使用日期</td>
      <td>N</td>
      <td>可以为空；如果为空，则取有效起始日期至有效截止日期这个时间段</td>
   </tr>
   <tr>
      <td>OrderStatus</td>
      <td>int</td>
      <td>订单状态</td>
      <td>Y</td>
      <td>2-待确认、3-已确认、5-已付款、6-已成交</td>
   </tr>
   <tr>
      <td>AppointStatus</td>
      <td>int</td>
      <td>预约状态</td>
      <td>Y</td>
      <td>不为空（1：未预约；2：待确认预约；3：已确认预约；4：已拒绝预约）</td>
   </tr>
   <tr>
      <td>ReturnDate</td>
      <td>DateTime</td>
      <td>返回日期</td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
   <tr>
      <td>TotalQuantity</td>
      <td>int</td>
      <td>产品总数（总份数）</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>TotalCount</td>
      <td>int</td>
      <td>总人数</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>TotalAdultQuantity</td>
      <td>int</td>
      <td>成人数</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>TotalChildQuantity</td>
      <td>int</td>
      <td>儿童数</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>Price</td>
      <td>decimal</td>
      <td>产品卖价</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>ProcessTime</td>
      <td>DateTime</td>
      <td>付款时间</td>
      <td>Y</td>
      <td>可以为空</td>
   </tr>
   <tr>
      <td>PriceAdjust</td>
      <td>decimal</td>
      <td>调整金额</td>
      <td>Y</td>
      <td>可以为空</td>
   </tr>
   <tr>
      <td>PresaleStartDate</td>
      <td>DateTime</td>
      <td>使用开始时间</td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
   <tr>
      <td>PresaleEndDate</td>
      <td>DateTime</td>
      <td>使用结束时间</td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
   <tr>
      <td>DepartCarInfo</td>
      <td>string</td>
      <td>接送信息</td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
   <tr>
      <td>ContactInfo</td>
      <td>ContactDTO</td>
      <td>订单联系人信息</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>PassengerList</td>
      <td>List < PassengerDTO></td>
      <td>订单旅客信息列表</td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
   <tr>
      <td>DeliveryInfo</td>
      <td>OrderDeliveryDTO</td>
      <td>配送信息</td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
   <tr>
      <td>AppItemId</td>
      <td>string</td>
      <td>商户套餐ID</td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
   <tr>
      <td>VendorCouponAmount</td>
      <td>Decimal</td>
      <td>商户承担优惠金额</td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
</table>


####ContactDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>ContactName</td>
      <td>string</td>
      <td>联系人名称</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>ContactMobile</td>
      <td>string</td>
      <td>联系人手机</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>ContactEmail</td>
      <td>string</td>
      <td>联系人邮箱</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
</table>

####PassengerDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>Name</td>
      <td>string</td>
      <td>旅客姓名</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>Gender</td>
      <td>string</td>
      <td>旅客性别</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>Type</td>
      <td>int</td>
      <td>旅客类型</td>
      <td>Y</td>
      <td>不为空；1-成人;2-儿童</td>
   </tr>
   <tr>
      <td>BirthDate</td>
      <td>DateTime</td>
      <td>生日</td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
   <tr>
      <td>CertificateType</td>
      <td>int</td>
      <td>证件类型</td>
      <td>Y</td>
      <td>不为空；1-身份证;2-护照;3-军官证;4-回乡证;5-台胞证;6-国际海员证;7-其他</td>
   </tr>
   <tr>
      <td>CertificateNumber</td>
      <td>string</td>
      <td>证件号码</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>Nationality</td>
      <td>string</td>
      <td>国籍</td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
   <tr>
      <td>IssueCity</td>
      <td>string</td>
      <td>签发地</td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
   <tr>
      <td>IssueDate</td>
      <td>DateTime</td>
      <td>签发日期</td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
   <tr>
      <td>PassportType</td>
      <td>int</td>
      <td>护照类型</td>
      <td>N</td>
      <td>可以为空；0-普通因私护照;1-外交护照;2-公务护照</td>
   </tr>
   <tr>
      <td>PassportDatelimit</td>
      <td>DateTime</td>
      <td>护照有效期</td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
   <tr>
      <td>ContactAddress</td>
      <td>string</td>
      <td>联系地址</td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
</table>


####OrderDeliveryDTO####


<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>Receiver</td>
      <td>string</td>
      <td>收件人</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>ContactPhone</td>
      <td>string</td>
      <td>联系电话</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>ZipCode</td>
      <td>string</td>
      <td>邮编</td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
   <tr>
      <td>Address</td>
      <td>string</td>
      <td>配送地址</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>CourierCorpName</td>
      <td>string</td>
      <td>快递公司</td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
   <tr>
      <td>CourierNumber</td>
      <td>string</td>
      <td>快递单号</td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
</table>

###6.8 修改订单信息###

- 测试地址：http://10.1.25.61:9900/order/order.svc/UpdateOrderInfo
- 正式地址：http://api.117lego.com/order/order.svc/UpdateOrderInfo
- 调用方式: POST

####请求消息####
<table class="gridtable">
   <tr>
      <th>RequestData</th>
      <td>UpdateOrderInfoRequestDTO</td>
   </tr>
   <tr>
      <th>Response</th>
      <td>ResponseDTO</td>
   </tr>
</table>


####UpdateOrderInfoRequestDTO####

<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>GroupId</td>
      <td>int</td>
      <td>商户ID</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>OrderId</td>
      <td>long</td>
      <td>订单ID</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>ContactInfo</td>
      <td>ContactDTO</td>
      <td>订单联系人信息</td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
   <tr>
      <td>InvoiceDInfo</td>
      <td>InvoiceDTO</td>
      <td>订单发票信息</td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
   <tr>
      <td>PassengerInfo</td>
      <td>List< PassengerDTO></td>
      <td>订单出行人信息</td>
      <td>N</td>
      <td>可以为空,如果更新一个出行人，则要把所有的可用出行人传过来</td>
   </tr>
</table>

####ContactDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>ContactName</td>
      <td>string</td>
      <td>联系人名称</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>ContactMobile</td>
      <td>string</td>
      <td>联系人手机</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>ContactEmail</td>
      <td>string</td>
      <td>联系人邮箱</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
</table>

####InvoiceDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>InvInfoTitle</td>
      <td>string</td>
      <td>发票抬头</td>
      <td>N</td>
      <td>可为空</td>
   </tr>
   <tr>
      <td>InvInfoType</td>
      <td>int?</td>
      <td>发票类别</td>
      <td>N</td>
      <td>1-餐饮服务费，2-交通服务费，3-旅游服务费</td>
   </tr>
   <tr>
      <td>Receiver</td>
      <td>string</td>
      <td>收件人</td>
      <td>N</td>
      <td>可为空</td>
   </tr>
   <tr>
      <td>ContactPhone</td>
      <td>string</td>
      <td>联系电话</td>
      <td>N</td>
      <td>可为空</td>
   </tr>
   <tr>
      <td>ProvinceName</td>
      <td>string</td>
      <td>邮寄省名称</td>
      <td>N</td>
      <td>可为空</td>
   </tr>
   <tr>
      <td>CityName</td>
      <td>string</td>
      <td>邮寄市名称</td>
      <td>N</td>
      <td>可为空</td>
   </tr>
   <tr>
      <td>Remark</td>
      <td>string</td>
      <td>邮寄地址备注</td>
      <td>N</td>
      <td>可为空</td>
   </tr>
   <tr>
      <td>ZipCode</td>
      <td>string</td>
      <td>邮编</td>
      <td>N</td>
      <td>可为空</td>
   </tr>
   <tr>
      <td>IsDelivery</td>
      <td>bool?</td>
      <td>是否配送</td>
      <td>N</td>
      <td>可为空</td>
   </tr>
</table>

####PassengerDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>Name</td>
      <td>string</td>
      <td>旅客姓名</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>Gender</td>
      <td>string</td>
      <td>旅客性别</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>Type</td>
      <td>int</td>
      <td>旅客类型</td>
      <td>Y</td>
      <td>不为空；1-成人;2-儿童</td>
   </tr>
   <tr>
      <td>BirthDate</td>
      <td>DateTime</td>
      <td>生日</td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
   <tr>
      <td>CertificateType</td>
      <td>int</td>
      <td>证件类型</td>
      <td>Y</td>
      <td>不为空；1-身份证;2-护照;3-军官证;4-回乡证;5-台胞证;6-国际海员证;7-其他</td>
   </tr>
   <tr>
      <td>CertificateNumber</td>
      <td>string</td>
      <td>证件号码</td>
      <td>Y</td>
      <td>不为空</td>
   </tr>
   <tr>
      <td>Nationality</td>
      <td>string</td>
      <td>国籍</td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
   <tr>
      <td>IssueCity</td>
      <td>string</td>
      <td>签发地</td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
   <tr>
      <td>IssueDate</td>
      <td>DateTime</td>
      <td>签发日期</td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
   <tr>
      <td>PassportType</td>
      <td>int</td>
      <td>护照类型</td>
      <td>N</td>
      <td>可以为空；0-普通因私护照;1-外交护照;2-公务护照</td>
   </tr>
   <tr>
      <td>PassportDatelimit</td>
      <td>DateTime</td>
      <td>护照有效期</td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
   <tr>
      <td>ContactAddress</td>
      <td>string</td>
      <td>联系地址</td>
      <td>N</td>
      <td>可以为空</td>
   </tr>
</table>

###6.9 设置商户订单标记###

- 测试地址：http://10.1.25.61:9900/order/order.svc/SetAppOrderColor
- 正式地址：http://api.117lego.com/order/order.svc/SetAppOrderColor
- 调用方式: POST

####请求消息####
<table class="gridtable">
   <tr>
      <th>RequestData</th>
      <td>SetAppOrderColorRequestDTO</td>
   </tr>
   <tr>
      <th>Response</th>
      <td>ResponseDTO</td>
   </tr>
</table>

####SetAppOrderColorRequestDTO####

<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>GroupId</td>
      <td>int</td>
      <td>商户ID</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>OrderColorList</td>
      <td>List< OrderColorDTO ></td>
      <td>订单颜色实体列表</td>
      <td>Y</td>
      <td></td>
   </tr>
</table>

####OrderColorDTO####

<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>OrderId</td>
      <td>long</td>
      <td>订单ID</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>IsAppOrderId</td>
      <td>bool</td>
      <td>是否为商户订单Id</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>Flag</td>
      <td>int</td>
      <td>颜色枚举</td>
      <td>Y</td>
      <td>颜色枚举（1：红色、2：绿色、3：蓝色、4：紫色、5：黄色、0：无色）</td>
   </tr>
</table>

####OrderProcessResponseDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>IsSuccess</td>
      <td>bool</td>
      <td>是否成功</td>
      <td>Y</td>
      <td>不为空；提示调用返回是否成功</td>
   </tr>
   <tr>
      <td>Code</td>
      <td>int</td>
      <td>编码</td>
      <td>Y</td>
      <td>" 1：更新订单颜色成功；
      -1：AppId不能为空；
      -2：签名错误；
      -3：请求超时；
      -4：订单不属于该商户；
      -5：订单部分更新成功；
      -6：订单颜色实体列表参数错误；
      -99：系统错误；</td>
  <tr>
      <td>Message</td>
      <td>string</td>
      <td>提示信息</td>
      <td>Y</td>
      <td>不为空；返回的调用结果信息</td>
   </tr>
   <tr>
      <td>Guid</td>
      <td>string</td>
      <td>唯一标识</td>
      <td>Y</td>
      <td>不为空；返回的调用唯一标识</td>
   </tr>
   <tr>
      <td>OrderResultList</td>
      <td>List< OrderResultDTO ></td>
      <td>订单处理结果列表</td>
      <td>Y</td>
      <td>不为空；返回的调用结果信息</td>
   </tr>
</table>

####OrderResultDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>IsSuccess</td>
      <td>bool</td>
      <td>是否成功</td>
      <td>Y</td>
      <td>不为空；提示调用返回是否成功</td>
   </tr>
   <tr>
      <td>Code</td>
      <td>int</td>
      <td>编码</td>
      <td>Y</td>
      <td>不为空；返回的调用结果编码</td>
   </tr>
   <tr>
      <td>Message</td>
      <td>string</td>
      <td>提示信息</td>
      <td>Y</td>
      <td>不为空；返回的调用结果信息</td>
   </tr>
   <tr>
      <td>OrderId</td>
      <td>string</td>
      <td>订单号（平台订单号或者商户订单号，和输入参数对应）</td>
      <td>Y</td>
      <td>不为空；返回的调用结果信息</td>
   </tr>
</table>

###6.10 设置商户订单商户券###

- 测试地址：http://10.1.25.61:9900/order/order.svc/SetAppOrderVoucher
- 正式地址：http://api.117lego.com/order/order.svc/SetAppOrderVoucher
- 调用方式: POST

####请求消息####
<table class="gridtable">
   <tr>
      <th>RequestData</th>
      <td>SetAppOrderVoucherRequestDTO</td>
   </tr>
   <tr>
      <th>Response</th>
      <td>ResponseDTO</td>
   </tr>
</table>

####SetAppOrderVoucherRequestDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>GroupId</td>
      <td>string</td>
      <td>商户ID</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>OrderVoucherList</td>
      <td>List< OrderVoucherDTO ></td>
      <td>订单商户券列表</td>
      <td>Y</td>
      <td></td>
   </tr>
</table>

####OrderVoucherDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>OrderId</td>
      <td>long</td>
      <td>订单ID</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>AppVoucherNo</td>
      <td>List< string ></td>
      <td>商户券列表</td>
      <td>Y</td>
      <td></td>
   </tr>
</table>


####OrderProcessResponseDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>IsSuccess</td>
      <td>bool</td>
      <td>是否成功</td>
      <td>Y</td>
      <td>不为空；提示调用返回是否成功</td>
   </tr>
   <tr>
      <td>Code</td>
      <td>int</td>
      <td>编码</td>
      <td>Y</td>
      <td></td>
  <tr>
      <td>Message</td>
      <td>string</td>
      <td>提示信息</td>
      <td>Y</td>
      <td>不为空；返回的调用结果信息</td>
   </tr>
   <tr>
      <td>Guid</td>
      <td>string</td>
      <td>唯一标识</td>
      <td>Y</td>
      <td>不为空；返回的调用唯一标识</td>
   </tr>
   <tr>
      <td>OrderResultList</td>
      <td>List< OrderResultDTO ></td>
      <td>订单处理结果列表</td>
      <td>Y</td>
      <td>不为空；返回的调用结果信息</td>
   </tr>
</table>

####OrderResultDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>IsSuccess</td>
      <td>bool</td>
      <td>是否成功</td>
      <td>Y</td>
      <td>不为空；提示调用返回是否成功</td>
   </tr>
   <tr>
      <td>Code</td>
      <td>int</td>
      <td>编码</td>
      <td>Y</td>
      <td>不为空；返回的调用结果编码</td>
   </tr>
   <tr>
      <td>Message</td>
      <td>string</td>
      <td>提示信息</td>
      <td>Y</td>
      <td>不为空；返回的调用结果信息</td>
   </tr>
   <tr>
      <td>OrderId</td>
      <td>string</td>
      <td>订单号（平台订单号或者商户订单号，和输入参数对应）</td>
      <td>Y</td>
      <td>不为空；返回的调用结果信息</td>
   </tr>
</table>

##七、商品相关API接口详情##

###7.1商品相关实体信息概览###
与订单相比，商品没有复杂的流程。对于商品来说，比较重要的是各个实体之间的关系，与商品相关的接口分为以下5类

- 商品套餐
- 商品列表
- 基本信息
- 库存、价格、班期
- 商品行程段


####商品套餐实体信息####

更新商品套餐的实体信息
通过 操作套餐(OperateProductItem)接口即可以完成对商品套餐的新增和更新操作。


![](http://10.1.25.61:9990/images/api/operateProductItem.png)


####商品列表实体信息####

获取商品列表接口(GetProductList) 返回的信息

![](http://10.1.25.61:9990/images/api/GetProductList.png)


####库存、价格、班期实体信息####
下面的信息可以用于新增和更新商品库存、价格，以及班期信息。 对于库存以及价格信息只能获取或者更新。而班期信息可以新增，更新以及获取。

![](http://10.1.25.61:9990/images/api/ProductSaleScheduleDTO.png)

####查看行程段参数实体信息####
获取行程段信息(GetProductSegmentList)返回的数据会比新增/更新的时候信息丰富一些。

![](http://10.1.25.61:9990/images/api/getProductSegment.png)

####操作行程段参数实体信息####
下面的实体信息可以用于新增和更新商品行程段信息,可以通过接口操作商品行程段(OperateProductSegment)来完成

![](http://10.1.25.61:9990/images/api/operateProductSegment.png)


###7.2 获取商品列表###

####请求消息####
<table class="gridtable">
   <tr>
      <th>RequestData</th>
      <td>GetProductListRequestDTO</td>
   </tr>
   <tr>
      <th>Response</th>
      <td>ResponseGetProductListDTO</td>
   </tr>
</table>

####GetProductListRequestDTO####

<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>Guid</td>
      <td>string</td>
      <td>唯一标识</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>IsSuccess</td>
      <td>bool</td>
      <td>是否成功</td>
      <td>Y</td>
      <td>不为空；提示调用返回是否成功</td>
   </tr>
   <tr>
      <td>Code</td>
      <td>int</td>
      <td>编码</td>
      <td>Y</td>
      <td>不为空；返回的调用结果编码</td>
   </tr>
   <tr>
      <td>Message</td>
      <td>string</td>
      <td>提示信息</td>
      <td>Y</td>
      <td>不为空；返回的调用结果信息</td>
   </tr>
   <tr>
      <td>ProductList</td>
      <td>List< ProductInfoResponseDTO ></td>
      <td>ProductInfoResponseDTO对象实例列表</td>
      <td>Y</td>
      <td></td>
   </tr>
</table>

####ProductInfoResponseDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>AuditState</td>
      <td>int</td>
      <td>审核状态</td>
      <td>N</td>
      <td></td>
   </tr>
   <tr>
      <td>ProductId</td>
      <td>int</td>
      <td>商品ID</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>GroupProductId</td>
      <td>string</td>
      <td>商户产品名</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>ProductName</td>
      <td>string</td>
      <td>产品名称</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>ItemList</td>
      <td>List< ProductItemBasicDTO ></td>
      <td>ProductItemBasicDTO对象实例列表</td>
      <td>Y</td>
      <td></td>
   </tr>
</table>
####ProductItemBasicDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>ItemId</td>
      <td>int</td>
      <td>套餐ID</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>ItemName</td>
      <td>string</td>
      <td>套餐名称</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>AppItemId</td>
      <td>string</td>
      <td>关联商户套餐ID</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>State</td>
      <td>int</td>
      <td>套餐有效性（1：是，0：否）</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>IsDefault</td>
      <td>bool</td>
      <td>是否默认套餐</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>IsCalculateMinPrice</td>
      <td>int</td>
      <td>是否计入商品起价 null:计入(默认) ,0:不计入, 1:计入</td>
      <td>Y</td>
      <td></td>
   </tr>
</table>

###7.3 获取商品库存、班期、价格 ###

####API方法####

- 获取库存 GetProductInventory
- 获取班期 GetProductScheduleList
- 获取价格 GetProductPrice

- 测试地址：http://10.1.25.61:9900/order/product.svc/API方法
- 正式地址：http://api.117lego.com/order/product.svc/API方法
- 调用方式: POST


####请求消息####
<table class="gridtable">
   <tr>
      <th>RequestData</th>
      <td>GetSaleScheduleRequestDTO</td>
   </tr>
   <tr>
      <th>Response</th>
      <td>ResponseGetSaleScheduleDTO</td>
   </tr>
</table>


####GetSaleScheduleRequestDTO####

<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>GroupId</td>
      <td>string</td>
      <td>商户ID</td>
      <td>Y</td>
      <td>商户ID，必填</td>
   </tr>
   <tr>
      <td>ProductId</td>
      <td>int</td>
      <td>商品Id</td>
      <td>Y</td>
      <td>商品ID，必填</td>
   </tr>
   <tr>
      <td>OperateType</td>
      <td>string</td>
      <td>操作类型</td>
      <td>Y</td>
      <td>Schedule:班期；Price：价格；Inventory：库存</td>
   </tr>
   <tr>
      <td>ItemId</td>
      <td>int</td>
      <td>套餐ID</td>
      <td>Y</td>
      <td>必填</td>
   </tr>
   <tr>
      <td>ScheduleDates</td>
      <td>List< DateTime ></td>
      <td>日期列表</td>
      <td>N</td>
      <td>要查询的具体日期列表（与起始日期， 截止日期互斥）</td>
   </tr>
   <tr>
      <td>StartDate</td>
      <td>DateTime?</td>
      <td>起始日期</td>
      <td>N</td>
      <td>查询日期范围开始（与ScheduleDates互斥）</td>
   </tr>
   <tr>
      <td>EndDate</td>
      <td>DateTime?</td>
      <td>截止日期</td>
      <td>N</td>
      <td>查询日期范围结束（与ScheduleDates互斥）</td>
   </tr>
</table>

####ResponseGetSaleScheduleDTO####

<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>Guid</td>
      <td>string</td>
      <td>唯一标识</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>SaleSchedule</td>
      <td>ProductSaleScheduleDTO</td>
      <td>商品班期信息</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>Code</td>
      <td>int</td>
      <td>编码</td>
      <td>Y</td>
      <td>不为空；返回的调用结果编码</td>
   </tr>
   <tr>
      <td>IsSuccess</td>
      <td>bool</td>
      <td>是否成功</td>
      <td>Y</td>
      <td>不为空；提示调用返回是否成功</td>
   </tr>
   <tr>
      <td>Message</td>
      <td>string</td>
      <td>提示信息</td>
      <td>Y</td>
      <td>不为空；返回的调用结果信息</td>
   </tr>
</table>

####ProductSaleScheduleDTO####

<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>ProductId</td>
      <td>int</td>
      <td>商品Id</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>ProductName</td>
      <td>string</td>
      <td>商品名称</td>
      <td>N</td>
      <td></td>
   </tr>
   <tr>
      <td>GroupProductId</td>
      <td>string</td>
      <td>商户商品Id</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>ItemSchedules</td>
      <td>List< ItemSaleScheduleDTO ></td>
      <td>套餐班期信息</td>
      <td>Y</td>
      <td></td>
   </tr>
</table>

####ItemSaleScheduleDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>Item</td>
      <td>ProductItemBasicDTO</td>
      <td>套餐信息</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>SaleSchedules</td>
      <td>List< SaleScheduleDTO ></td>
      <td>价格、库存</td>
      <td>Y</td>
      <td></td>
   </tr>
</table>

####ProductItemBasicDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>ItemId</td>
      <td>int</td>
      <td>套餐ID</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>ItemName</td>
      <td>string</td>
      <td>套餐名称</td>
      <td></td>
      <td></td>
   </tr>
   <tr>
      <td>AppItemId</td>
      <td>string</td>
      <td>关联商户套餐ID</td>
      <td></td>
      <td></td>
   </tr>
   <tr>
      <td>State</td>
      <td>int</td>
      <td>套餐有效性（1：是，0：否）</td>
      <td></td>
      <td></td>
   </tr>
   <tr>
      <td>IsDefault</td>
      <td>bool</td>
      <td>是否默认套餐（1：是，0：否）</td>
      <td></td>
      <td></td>
   </tr>
   <tr>
      <td>IsCalculateMinPrice</td>
      <td>int?</td>
      <td>是否是否计入商品起价 null，计入(默认)； 0，不计入; 1,计入</td>
      <td></td>
      <td></td>
   </tr>
</table>

####SaleScheduleDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>ScheduleDate</td>
      <td>DateTime?</td>
      <td>班期日期</td>
      <td>Y</td>
      <td>要查询的具体日期列表（与起始日期， 截止日期互斥）</td>
   </tr>
   <tr>
      <td>ScheduleStartDate</td>
      <td>DateTime?</td>
      <td>班期起始日期（与ScheduleDate互斥）</td>
      <td>N</td>
      <td>查询日期范围开始（与ScheduleDates互斥）</td>
   </tr>
   <tr>
      <td>ScheduleEndDate</td>
      <td>DateTime?</td>
      <td>班期结束日期（与ScheduleDate互斥）</td>
      <td>N</td>
      <td>查询日期范围结束（与ScheduleDates互斥）</td>
   </tr>
   <tr>
      <td>State</td>
      <td>int</td>
      <td>0：无效、1：有效</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>Prices</td>
      <td>List< ProductPriceDTO ></td>
      <td>价格信息</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>Inventory</td>
      <td>ProductInventoryDTO</td>
      <td>库存信息</td>
      <td>Y</td>
      <td></td>
   </tr>
</table>

####ProductPriceDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>PriceModeId</td>
      <td>int</td>
      <td>价种</td>
      <td>Y</td>
      <td>1:成人价， 2：儿童价，3：单房差，4：单价</td>
   </tr>
   <tr>
      <td>OriginalPrice</td>
      <td>int</td>
      <td>原价</td>
      <td>Y</td>
      <td>卖价不得高于原价</td>
   </tr>
   <tr>
      <td>SalesPrice</td>
      <td>int</td>
      <td>卖价</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>State</td>
      <td>int</td>
      <td>价种有效性</td>
      <td>Y</td>
      <td>0：无效、1：有效</td>
   </tr>
</table>

####ProductInventoryDTO

<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>Total</td>
      <td>int</td>
      <td>总库存</td>
      <td>N</td>
      <td>可为空</td>
   </tr>
   <tr>
      <td>Used</td>
      <td>int</td>
      <td>使用库存</td>
      <td>N</td>
      <td></td>
   </tr>
   <tr>
      <td>InventoryMode</td>
      <td>int</td>
      <td>库存模式</td>
      <td>Y</td>
      <td>不为空（2：无限库存；4：库存不可超；）</td>
   </tr>
   <tr>
      <td>InventoryType</td>
      <td>int</td>
      <td>库存类型</td>
      <td>N</td>
      <td>0：总库存、1：剩余库存</td>
   </tr>
</table>

###7.4 获取商品库存、价格、班期

- 更新库存 GetProductInventory
- 更新班期 GetProductSchedule
- 更新价格 GetProductPrice

- 测试地址：http://10.1.25.61:9900/order/product.svc/API方法
- 正式地址：http://api.117lego.com/order/product.svc/API方法
- 调用方式: POST


####请求消息####
<table class="gridtable">
   <tr>
      <th>RequestData</th>
      <td>GetSaleScheduleRequestDTO</td>
   </tr>
   <tr>
      <th>Response</th>
      <td>ResponseGetSaleScheduleDTO</td>
   </tr>
</table>


####GetSaleScheduleRequestDTO

<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>GroupId</td>
      <td>string</td>
      <td>商户ID</td>
      <td>Y</td>
      <td>商户ID，必填</td>
   </tr>
   <tr>
      <td>ProductId</td>
      <td>int</td>
      <td>商品Id</td>
      <td>Y</td>
      <td>商品ID，必填</td>
   </tr>
   <tr>
      <td>OperateType</td>
      <td>string</td>
      <td>操作类型</td>
      <td>Y</td>
      <td> CloseSchedule：关闭班期；OpenSchedule：打开班期)</td>
   </tr>
   <tr>
      <td>ItemId</td>
      <td>int</td>
      <td>套餐ID</td>
      <td>Y</td>
      <td>必填</td>
   </tr>
   <tr>
      <td>ScheduleDates</td>
      <td>List< DateTime ></td>
      <td>日期列表</td>
      <td>N</td>
      <td>要查询的具体日期列表（与起始日期， 截止日期互斥）</td>
   </tr>
   <tr>
      <td>StartDate</td>
      <td>DateTime?</td>
      <td>起始日期</td>
      <td>N</td>
      <td>查询日期范围开始（与ScheduleDates互斥）</td>
   </tr>
   <tr>
      <td>EndDate</td>
      <td>DateTime?</td>
      <td>截止日期</td>
      <td>N</td>
      <td>查询日期范围结束（与ScheduleDates互斥）</td>
   </tr>
</table>


####ResponseGetSaleScheduleDTO
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>Guid</td>
      <td>string</td>
      <td>唯一标识</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>SaleSchedule</td>
      <td>ProductSaleScheduleDTO</td>
      <td>商品班期信息</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>Code</td>
      <td>int</td>
      <td>编码</td>
      <td>Y</td>
      <td></td>
   </tr>
 <tr>
      <td>IsSuccess</td>
      <td>bool</td>
      <td>是否成功</td>
      <td>Y</td>
      <td></td>
   </tr>
 <tr>
      <td>Message</td>
      <td>string</td>
      <td>提示信息</td>
      <td>Y</td>
      <td></td>
   </tr>
</table>


####ProductSaleScheduleDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>ProductId</td>
      <td>int</td>
      <td>商户Id</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>ProductName</td>
      <td>string</td>
      <td>商品名称</td>
      <td>N</td>
      <td></td>
   </tr>
   <tr>
      <td>GroupProductId</td>
      <td>string</td>
      <td>商户商品Id（商户商品名）</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>ItemSchedules</td>
      <td>List < ItemSaleScheduleDTO ></td>
      <td>套餐班期信息</td>
      <td>Y</td>
      <td></td>
   </tr>
</table>

####ItemSaleScheduleDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>Item</td>
      <td>ProductItemBasicDTO</td>
      <td>套餐信息</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>SaleSchedules</td>
      <td>List< SaleScheduleDTO ></td>
      <td>价格、库存</td>
      <td>Y</td>
      <td></td>
   </tr>
</table>

####ProductItemBasicDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>ItemId</td>
      <td>int</td>
      <td>套餐ID</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>ItemName</td>
      <td>string</td>
      <td>套餐名称</td>
      <td>N</td>
      <td></td>
   </tr>
   <tr>
      <td>AppItemId</td>
      <td>string</td>
      <td>关联商户套餐ID</td>
      <td>N</td>
      <td></td>
   </tr>
   <tr>
      <td>State</td>
      <td>int</td>
      <td>套餐有效性（1：是，0：否）</td>
      <td>N</td>
      <td></td>
   </tr>
   <tr>
      <td>IsDefault</td>
      <td>bool</td>
      <td>是否默认套餐（1：是，0：否）</td>
      <td>N</td>
      <td></td>
   </tr>
   <tr>
      <td>IsCalculateMinPrice</td>
      <td>int?</td>
      <td>是否是否计入商品起价 null，计入(默认)； 0，不计入; 1,计入</td>
      <td>N</td>
      <td></td>
   </tr>
</table>

####SaleScheduleDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>ScheduleDate</td>
      <td>DateTime?</td>
      <td>班期日期</td>
      <td>Y</td>
      <td>要查询的具体日期列表（与起始日期， 截止日期互斥）</td>
   </tr>
   <tr>
      <td>ScheduleStartDate</td>
      <td>DateTime?</td>
      <td>班期起始日期（与ScheduleDate互斥）</td>
      <td>N</td>
      <td>查询日期范围开始（与ScheduleDates互斥）</td>
   </tr>
   <tr>
      <td>ScheduleEndDate</td>
      <td>DateTime?</td>
      <td>班期结束日期（与ScheduleDate互斥）</td>
      <td>N</td>
      <td>查询日期范围结束（与ScheduleDates互斥）</td>
   </tr>
   <tr>
      <td>State</td>
      <td>int</td>
      <td>0：无效、1：有效</td>
      <td></td>
      <td>可为空或忽略该字段</td>
   </tr>
   <tr>
      <td>Prices</td>
      <td>List< ProductPriceDTO ></td>
      <td>价格信息</td>
      <td></td>
      <td>更新班期和价格的时候必需,更新库存的时候可忽略该字段</td>
   </tr>
   <tr>
      <td>Inventory</td>
      <td>ProductInventoryDTO</td>
      <td>库存信息</td>
      <td></td>
      <td>更新班期和库存的时候必需,更新价格的时候可忽略该字段</td>
   </tr>
</table>

####ProductPriceDTO####
更新班期和价格时此实体信息必需
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>PriceModeId</td>
      <td>int</td>
      <td>价种</td>
      <td>Y</td>
      <td>1:成人价， 2：儿童价，3：单房差，4：单价</td>
   </tr>
   <tr>
      <td>OriginalPrice</td>
      <td>int</td>
      <td>原价</td>
      <td>Y</td>
      <td>卖价不得高于原价</td>
   </tr>
   <tr>
      <td>SalesPrice</td>
      <td>int</td>
      <td>卖价</td>
      <td>Y</td>
      <td></td>
   </tr>
</table>

####ProductInventoryDTO####
更新班期和库存时此实体必填
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>Total</td>
      <td>int</td>
      <td>总库存</td>
      <td>N</td>
      <td>可为空</td>
   </tr>
   <tr>
      <td>Used</td>
      <td>int</td>
      <td>使用库存</td>
      <td>N</td>
      <td></td>
   </tr>
   <tr>
      <td>InventoryMode</td>
      <td>int</td>
      <td>库存模式</td>
      <td>Y</td>
      <td>不为空（2：无限库存；4：库存不可超；）</td>
   </tr>
   <tr>
      <td>InventoryType</td>
      <td>int</td>
      <td>库存类型</td>
      <td>N</td>
      <td>0：总库存、1：剩余库存</td>
   </tr>
</table>


###7.5 更新商品库存、价格、班期

####API方法
- 更新库存 UpdateProductInventory
- 更新班期 UpdateProductSchedule
- 更新价格 UpdateProductPrice

- 测试地址：http://10.1.25.61:9900/order/product.svc/API方法
- 正式地址：http://api.117lego.com/order/product.svc/API方法
- 调用方式: POST

####请求消息####
<table class="gridtable">
   <tr>
      <th>RequestData</th>
      <td>OperateSaleScheduleRequestDTO</td>
   </tr>
   <tr>
      <th>Response</th>
      <td>ResponseOperateProductSaleScheduleDTO</td>
   </tr>
</table>

####OperateSaleScheduleRequestDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>GroupId</td>
      <td>string</td>
      <td>商户ID</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>OperateType</td>
      <td>string</td>
      <td>操作类型</td>
      <td>Y</td>
      <td>操作类型(Schedule:班期；Price：价格；Inventory：库存)</td>
   </tr>
   <tr>
      <td>SaleSchedule</td>
      <td>ProductSaleScheduleDTO</td>
      <td>商品班期信息</td>
      <td>Y</td>
      <td></td>
   </tr>
</table>

####ProductSaleScheduleDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>ProductId</td>
      <td>int</td>
      <td>商户Id</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>ProductName</td>
      <td>string</td>
      <td>商品名称</td>
      <td>N</td>
      <td></td>
   </tr>
   <tr>
      <td>GroupProductId</td>
      <td>string</td>
      <td>商户商品Id（商户商品名）</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>ItemSchedules</td>
      <td>List < ItemSaleScheduleDTO ></td>
      <td>套餐班期信息</td>
      <td>Y</td>
      <td></td>
   </tr>
</table>

####ItemSaleScheduleDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>Item</td>
      <td>ProductItemBasicDTO</td>
      <td>套餐信息</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>SaleSchedules</td>
      <td>List< SaleScheduleDTO ></td>
      <td>价格、库存</td>
      <td>Y</td>
      <td></td>
   </tr>
</table>

####ProductItemBasicDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>ItemId</td>
      <td>int</td>
      <td>套餐ID</td>
      <td>Y</td>
      <td></td>
   </tr>
   <tr>
      <td>ItemName</td>
      <td>string</td>
      <td>套餐名称</td>
      <td>N</td>
      <td></td>
   </tr>
   <tr>
      <td>AppItemId</td>
      <td>string</td>
      <td>关联商户套餐ID</td>
      <td>N</td>
      <td></td>
   </tr>
   <tr>
      <td>State</td>
      <td>int</td>
      <td>套餐有效性（1：是，0：否）</td>
      <td>N</td>
      <td></td>
   </tr>
   <tr>
      <td>IsDefault</td>
      <td>bool</td>
      <td>是否默认套餐（1：是，0：否）</td>
      <td>N</td>
      <td></td>
   </tr>
   <tr>
      <td>IsCalculateMinPrice</td>
      <td>int?</td>
      <td>是否是否计入商品起价 null，计入(默认)； 0，不计入; 1,计入</td>
      <td>N</td>
      <td></td>
   </tr>
</table>

####SaleScheduleDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>ScheduleDate</td>
      <td>DateTime?</td>
      <td>班期日期</td>
      <td>Y</td>
      <td>要查询的具体日期列表（与起始日期， 截止日期互斥）</td>
   </tr>
   <tr>
      <td>ScheduleStartDate</td>
      <td>DateTime?</td>
      <td>班期起始日期（与ScheduleDate互斥）</td>
      <td>N</td>
      <td>查询日期范围开始（与ScheduleDates互斥）</td>
   </tr>
   <tr>
      <td>ScheduleEndDate</td>
      <td>DateTime?</td>
      <td>班期结束日期（与ScheduleDate互斥）</td>
      <td>N</td>
      <td>查询日期范围结束（与ScheduleDates互斥）</td>
   </tr>
   <tr>
      <td>State</td>
      <td>int</td>
      <td>0：无效、1：有效</td>
      <td></td>
      <td>可为空或忽略该字段</td>
   </tr>
   <tr>
      <td>Prices</td>
      <td>List< ProductPriceDTO ></td>
      <td>价格信息</td>
      <td></td>
      <td>更新班期和价格的时候必需,更新库存的时候可忽略该字段</td>
   </tr>
   <tr>
      <td>Inventory</td>
      <td>ProductInventoryDTO</td>
      <td>库存信息</td>
      <td></td>
      <td>更新班期和库存的时候必需,更新价格的时候可忽略该字段</td>
   </tr>
</table>

####ProductPriceDTO####
更新班期和价格时此实体信息必需
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>PriceModeId</td>
      <td>int</td>
      <td>价种</td>
      <td>Y</td>
      <td>1:成人价， 2：儿童价，3：单房差，4：单价</td>
   </tr>
   <tr>
      <td>OriginalPrice</td>
      <td>int</td>
      <td>原价</td>
      <td>Y</td>
      <td>卖价不得高于原价</td>
   </tr>
   <tr>
      <td>SalesPrice</td>
      <td>int</td>
      <td>卖价</td>
      <td>Y</td>
      <td></td>
   </tr>
</table>

####ProductInventoryDTO####
更新班期和库存时此实体必填
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>Total</td>
      <td>int</td>
      <td>总库存</td>
      <td>N</td>
      <td>可为空</td>
   </tr>
   <tr>
      <td>Used</td>
      <td>int</td>
      <td>使用库存</td>
      <td>N</td>
      <td></td>
   </tr>
   <tr>
      <td>InventoryMode</td>
      <td>int</td>
      <td>库存模式</td>
      <td>Y</td>
      <td>不为空（2：无限库存；4：库存不可超；）</td>
   </tr>
   <tr>
      <td>InventoryType</td>
      <td>int</td>
      <td>库存类型</td>
      <td>N</td>
      <td>0：总库存、1：剩余库存</td>
   </tr>
</table>

####ResponseOperateProductSaleScheduleDTO####

<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>IsSuccess</td>
      <td>bool</td>
      <td>是否成功</td>
      <td>Y</td>
      <td>不为空；提示调用返回是否成功</td>
   </tr>
   <tr>
      <td>Code</td>
      <td>int</td>
      <td>编码</td>
      <td>Y</td>
      <td>发生错误时，不为空；返回的调用结果编码</td>
   </tr>
   <tr>
      <td>Message</td>
      <td>string</td>
      <td>提示信息</td>
      <td>Y</td>
      <td>发生错误时，不为空；返回的调用结果信息</td>
   </tr>
   <tr>
      <td>Guid</td>
      <td>string</td>
      <td>唯一标识</td>
      <td>Y</td>
      <td></td>
   </tr>
</table>

###7.6 关闭/打开班期###

- 测试地址：http://10.1.25.61:9900/order/product.svc/OperateScheduleDate
- 正式地址：http://api.117lego.com/order/product.svc/OperateScheduleDate
- 调用方式: POST

####请求消息####
<table class="gridtable">
   <tr>
      <th>RequestData</th>
      <td>GetSaleScheduleRequestDTO</td>
   </tr>
   <tr>
      <th>Response</th>
      <td>ResponseProductDTO</td>
   </tr>
</table>

####GetSaleScheduleRequestDTO####

<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>GroupId</td>
      <td>string</td>
      <td>商户ID</td>
      <td>Y</td>
      <td>商户ID，必填</td>
   </tr>
   <tr>
      <td>ProductId</td>
      <td>int</td>
      <td>商品Id</td>
      <td>Y</td>
      <td>商品ID，必填</td>
   </tr>
   <tr>
      <td>OperateType</td>
      <td>string</td>
      <td>操作类型</td>
      <td>Y</td>
      <td> CloseSchedule：关闭班期；OpenSchedule：打开班期)</td>
   </tr>
   <tr>
      <td>ItemId</td>
      <td>int</td>
      <td>套餐ID</td>
      <td>Y</td>
      <td>必填</td>
   </tr>
   <tr>
      <td>ScheduleDates</td>
      <td>List< DateTime ></td>
      <td>日期列表</td>
      <td>N</td>
      <td>要查询的具体日期列表（与起始日期， 截止日期互斥）</td>
   </tr>
   <tr>
      <td>StartDate</td>
      <td>DateTime?</td>
      <td>起始日期</td>
      <td>N</td>
      <td>查询日期范围开始（与ScheduleDates互斥）</td>
   </tr>
   <tr>
      <td>EndDate</td>
      <td>DateTime?</td>
      <td>截止日期</td>
      <td>N</td>
      <td>查询日期范围结束（与ScheduleDates互斥）</td>
   </tr>
</table>

###7.7 操作套餐###
OperateProductItem是一个通用服务，既可以用来新增套餐又可以用来修改套餐。新增时ProductItemBasicDTO的ItemId(套餐Id)为空即可。同时针对新增还可以调用AddProductItem, 修改可以调用UpdateProductItem, 该三个服务的参数是一样的。

- 测试地址：http://10.1.25.61:9900/product/product.svc/OperateProductItem
- 正式地址：http://api.117lego.com/product/product.svc/OperateProductItem
- 调用方式: POST

####请求消息####
<table class="gridtable">
   <tr>
      <th>RequestData</th>
      <td>ProductItemRequestDTO</td>
   </tr>
   <tr>
      <th>Response</th>
      <td>ResponseProductDT</td>
   </tr>
</table>

####ProductItemRequestDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>GroupId</td>
      <td>string</td>
      <td>商户ID</td>
      <td>Y</td>
      <td>商户ID，必填</td>
   </tr>
   <tr>
      <td>ProductId</td>
      <td>int</td>
      <td>商品Id</td>
      <td>Y</td>
      <td>商品ID，必填</td>
   </tr>
   <tr>
      <td>ItemList</td>
      <td>List<ProductItemBasicDTO></td>
      <td>套餐信息</td>
      <td>Y</td>
      <td>套餐基本信息</td>
   </tr>
</table>

####ProductItemBasicDTO####
<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>ItemId</td>
      <td>int</td>
      <td>套餐Id</td>
      <td>N</td>
      <td>新增的时候为空，修改的时候必填</td>
   </tr>
   <tr>
      <td>ItemName</td>
      <td>string</td>
      <td>套餐名称</td>
      <td>Y</td>
      <td>套餐名称</td>
   </tr>
   <tr>
      <td>AppItemId</td>
      <td>string</td>
      <td>商户关联套餐Id</td>
      <td>Y</td>
      <td>商户关联套餐Id</td>
   </tr>
   <tr>
      <td>State</td>
      <td>int</td>
      <td>状态</td>
      <td>N</td>
      <td>套餐有效性（1：是，0：否）</td>
   </tr>
   <tr>
      <td>IsDefault</td>
      <td>bool</td>
      <td>是否默认套餐</td>
      <td>N</td>
      <td>是否默认套餐（1：是，0：否）</td>
   </tr>
   <tr>
      <td>IsCalculateMinPrice</td>
      <td>int?</td>
      <td>是否计入商品起价</td>
      <td>N</td>
      <td> null，计入(默认)； 0，不计入; 1,计入</td>
   </tr>
</table>

####ResponseProductDTO####

<table class="gridtable">
   <tr>
      <th>属性名称</th>
      <th>属性类型</th>
      <th>属性描述</th>
      <th>是否必填</th>
      <th>属性备注</th>
   </tr>
   <tr>
      <td>IsSuccess</td>
      <td>bool</td>
      <td>是否成功</td>
      <td>Y</td>
      <td>不为空；提示调用返回是否成功</td>
   </tr>
   <tr>
      <td>Code</td>
      <td>int</td>
      <td>编码</td>
      <td>Y</td>
      <td>不为空；返回的调用结果编码</td>
   </tr>
   <tr>
      <td>Message</td>
      <td>string</td>
      <td>提示信息</td>
      <td>Y</td>
      <td>不为空；返回的调用结果信息</td>
   </tr>
   <tr>
      <td>Guid</td>
      <td>string</td>
      <td>唯一标识</td>
      <td>Y</td>
      <td></td>
   </tr>
</table>

##八、系统级别错误代码##

<table class="gridtable">
   <tr>
      <th>Code</th>
      <th>Message</th>
   </tr>
   <tr>
      <td>-1</td>
      <td>系统意外错误</td>
   </tr>
   <tr>
      <td>-2</td>
      <td>AppId不能为空</td>
   </tr>
   <tr>
      <td>-3</td>
      <td>AppId必须是数字</td>
   </tr>
   <tr>
      <td>-4</td>
      <td>该请求已过时</td>
   </tr>
   <tr>
      <td>-5</td>
      <td>商户配置错误</td>
   </tr>
   <tr>
      <td>-6</td>
      <td>签名错误</td>
   </tr>
   <tr>
      <td>-7</td>
      <td>已经被列入黑名单</td>
   </tr>
   <tr>
      <td>-8</td>
      <td>输入参数为空</td>
   </tr>
 <tr>
      <td>-14</td>
      <td>当前商户没有使用当前模块权限</td>
   </tr>
 <tr>
      <td>-110</td>
      <td>商户号前后不统一</td>
   </tr>
</table>

##九、常见QA##

###重复操作兼容
不管是商户发起的调用（预约，使用等)还是淘在路上发起的调用（创建订单、付款、申请取消) 。如果由于网络或者其它原因没有收到返回结果（实际上已经操作成功），后续不管由任何一方发起的重复操作，另外一方都需要实现兼容，即返回同样的信息，也就是说双方接口都允许重复的调用操作。

###参数Schema中标识为非必填的字段
由于所有淘在路上提供给商户调用的接口参数都以JSON字符串的形式封装在 RequestData中，对于非必填的字段，在这个JSON字符串中可以直接忽略（填了也没有关系)。

举个例子：

下面是操作套餐的Request
Reqeust
<pre>
{
    "AppId":"[String]{商户Id}",
    "Time":"[long]{Unix Time Stamp}",
    "Sign":"[String]{签名}",
    "RequestData":"{
        \"ItemList\":[{
            \"ItemId\":[Int32]{套餐ID(如不传，则操作默认套餐)},
            \"ItemName\":\"[String]{套餐名称：不为空}\",
            \"AppItemId\":\"[String]{关联商户套餐ID：不为空}\",
            \"State\":[Int32]{套餐有效性（1：是，0：否）},
            \"IsDefault\":\"[Boolean]{是否默认套餐（1：是，0：否）}\",
            \"IsCalculateMinPrice\":\"[Int32]{是否是否计入商品起价 null，计入(默认)； 0，不计入; 1,计入}\"
        }],
        \"ProductId\":[Int32]{产品ID：不为空},
        \"GroupId\":\"[String]{商户ID：不为空}\"
    }"
}			
</pre>				
对于State, IsDefault, IsCalculateMinprice三个非必填的字段，我们可以直接忽略，就像下面这样。
<pre>
{
    "AppId":"[String]{商户Id}",
    "Time":"[long]{Unix Time Stamp}",
    "Sign":"[String]{签名}",
    "RequestData":"{
        \"ItemList\":[{
            \"ItemId\":[Int32]{套餐ID(如不传，则操作默认套餐)},
            \"ItemName\":\"[String]{套餐名称：不为空}\",
            \"AppItemId\":\"[String]{关联商户套餐ID：不为空}\"
        }],
        \"ProductId\":[Int32]{产品ID：不为空},
        \"GroupId\":\"[String]{商户ID：不为空}\"
    }"
}			
</pre>
