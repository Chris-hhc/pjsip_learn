## 总述



1. 软电话A 向 B 发送一个 SIP消息 INVITE， 邀请B通话
2. 软电话B振铃，向A 回复一个SIP消息 RING, 通知 A 正在振铃中，请A等待
3. 软电话B提机，向A发一个SIP消息 OK, 通知 A 可以通话了
4. 软电话A 向 B 回复一个回应消息 ACK，正式启动通话
5. 接下来，双方通话
6. 软电话B挂机，向 A 发一个SIP消息 BYE, 通知 A 通话结束
7. 软电话A 向 B 回复一个消息 OK, 通话结束

含有代理服务器

- 代理服务器接收到了 INVITE 请求，然后发送了一个 100 (Trying) 响应给 Alice 的软电话。这个 100 (Trying) 响应表示这个 INVITE 已经被收到，代理正在通过路由设置路由这个 INVITE 到其目的地。
- atlanta.com代理服务器定位到这个代理服务器在 biloxi.com，它可能执行一个特别的 DNS 查询来找到服务 biloxi.com 域的 SIP 服务器。获得 biloxi.com 代理服务器的 IP 地址，然后转发或者在这里代理其 INVITE 请求。在转发这个请求之前，这个 atlanta.com 代理服务器添加另外一个 Via 头字段，这个头字段包含自己的地址（这个 INVITE 已经在第一个 Via 包含了 Alice 的地址）
- biloxi.com代理服务器收到这个 INVITE 消息后，然后回复一个带 100 (Trying) 响应消息到
  atlanta.com 代理服务器，表示它已经收到了这个 INVITE 消息，正在处理这个请求。
- 代理服务器会查询一个定位服务器，我们称之为定位服务，定位服务包含当前 Bob 的IP 地址。biloxi.com 代理服务器会添加另外一个 Via header ，并且携带自己的 IP 地址，这个地址是针对这个 INVITE 请求的，代理转发这个请求到 Bob 的 SIP 软电话。
- Bob 决定是否应答这个呼叫，这里 Bob 的软电话会产生振铃提示。Bob 的软电话提示 180 振铃，这个响应消息会路由根据相反的方向回到两个代理服务器。每个代理使用 Via header 域值来决定发送响应的地址方向，并且从顶部路由记录中删除自己的地址。因此，尽管要求 DNS 和定位服务查询 路由这个初始的 INVITE 请求，180（Ringing）响应返回到呼叫方时可以没有查询消息或没有代理服务器中所保持的状态。
- Bob 决定应答这个呼叫。当他拿起电话听筒时，他的 SIP 电话会发送一个 200 (OK) 响应消息来表示这个呼叫已经应答。这个 200 (OK) 包含了一个消息体，这个消息体带了这个呼叫会话的媒体描述类型，这个媒体描述中说明了 Bob 希望和Alice 创建会话。
- 最后，Alice 软电话发送一个确认消息 ACK，这个消息发送到 Bob 软电话来确认最终响应 (200 (OK))已收到。在这个示例中，这个 ACK 是通过 Alice 软电话直接被发送到了 Bob 软电话，发送过程绕开了两个代理服务器。这样处理的原因就是因为两个终端已经通过互相学习知道对方的地址，双方地址是通过 INVITE/200（OK）交互时的 Contact 头获得，当然这个地址在初始时的 INVITE 是双方都不知道的。两个代理服务器的查询服务也不需要，因此，代理服务器则会退出这个呼叫流程。
- Alice 或 Bob 任何一方都可以有权决定修改媒体会话的属性。修改会话属性是通过发送一个 re-INVITE 消息，在此消息中包含一个新的媒体描述来实现。这个 re-INVITE 涉及到了已存在的 dialog,因此其他的参与方知道这个消息是修改了现在的会话，而不是重新建立的新会话。其他方发送一个 200（ok）接受这个修改。请求方对 200（ok）发送一个 ACK。如果其他方不能接受这个修改的话，它会发生一个错误响应，例如 488 (Not Acceptable Here)，同样也接收一个 ACK 确认消息。但是，这个 re-INVITE 失败不会导致目前的呼叫失败-这个会话仍然会继续使用以前协商的属性。
- 在呼叫结束后，Bob 首先挂机(hangs up)，并且生成一个 BYE 消息。这个 BYE 会直接
  路由返回到 Alice 的软电话，这里仍然绕过了代理。 Alice 确认了 BYE 接收，发送一个
  200（ok），结束这个会话和 BYE 消息事务。
- Bob 软电话基于初始化处理，在一定周期内 Bob 软电话对在biloxi.com 的服务器发送 REGISTER 消息，我们称之为 SIP registrar 或者 SIP 注册。REGISTER 消息关联 Bob 的 SIP 软电话或者 SIPS URI (sip:bob@biloxi.com)，这个机器是当前 Bob 写入记录的地址（它在 Contact 头中传输 SIP 或者 SIP URL）。 这个注册会写入此关联，也被称之为在数据库中的绑定或者定位服务，此定位服务可以使用在biloxi.com 域的代理中。经常，对于一个域的注册服务器需要和这个域的代理协同工作。这里一定要注意，区分不同类型的 SIP 服务器功能概念是非常重要的，它们区别是在于逻辑处理的不同，而不是物理上，形体上的不同

## Structure of the Protocol

1. SIP 结构的最低层是语法和解码层。解码是通过增强的 Backus-Naur Form grammar(BNF)语法来实现的。
2. 第二层是传输层。它定义了用户如何发送请求，如何接收响应和服务器如何通过网络接收请求和发送响应。所有 SIP 网元都包含一个传输层。
3. 第三层是事务层。事务是 SIP 的基础核心模块。事务是一个由用户端事务对服务器端事务发送的请求，用户端使用传输层对服务器端发送事务请求，所有的服务器端事务所携带响应消息返回到客户端。事务层处理应用层的重传，对请求响应的匹配和应用层超时管理。 任何由用户代理（UAC）完成的任务通过使用一系列的事务来触发。 用户代理包含了一个事务层，就像是一个状态代理。 无状态代理没有包含事务层。事务层有一个用户端模块（称之为用户事
   务）和一个服务器端事务模块（称之为服务器端模块），每个模块通过各自的有限状态机来呈现，状态机来处理每个特别的请求
4. 在事务层上面的是事务用户（TU）。每个 SIP 实体，除了无状态代理都是一个事务用户。**当一个 TU 希望发送一个请求时，它会创建一个用户事务实例，然后把这个实例传递给这个请求，并且携带目的地 IP 地址，端口和传输请求**。一个创建了用户事务的TU 也可以取消这个用户事务。当用户取消了一个事务时，它会请求服务器停止进一步的处理，变换到退出的状态，这个状态是这个事务初始化前的退出状态，并且生成对这个事务生成错误响应消息。 这个处理过程是通过一个 CANCEL 请求来处理，它构成了属于自己的事务，但是仅针对这个被取消的事务

## 术语

### Outbound Proxy

它是一个代理，负责接收从客户端发出的请求，即使它可能不是一个通过 Request-URI 解析度服务器。 通常情况下，一个 UA 可以通过 outbound proxy手动配置，或通过自动配置协议进行学习。

### Location Service

定位服务用来支持一个 SIP 重定位或代理服务器来获得关于被呼叫方可能存在的地址信息。它包含一个绑定的 address-of-record 列表数值，这些从从零个到多个 contact 地址。这个绑定关系可以通过多种方式来创建或者删除；此协议细节中定义了一个 REGISTER method 来更新绑定关系。

### Redirect Server

 重定向服务器是一个用户代理服务器，它会对接收的请求产生 3xx 响应，重新定向用户，让用户联系其他可选的 URL 列表中的 URI 地址。

### Request

 请求是一个由用户端发送到服务器的 SIP 消息，请求的目的是触发一个特别的操作。

### Response

响应是一个由服务器端发送到用户端的 SIP 消息，其目的是说明请求发送后服务器端回复的状态。

### Route Set

路由集是一组有序 SIP 或者 SIPS URI 的集和，它用来表示当发送 一个特别的请求时所经过的代理列表。 路由集通过路由头，例如 Record-Route 或者经过配置后获得。

### Stateful Proxy

状态代理是一个逻辑实体，它按照规范中请求处理的流程保持用户端和服务器端之间的事务状态机的处理状态，也就是所谓的事务状态代理。状态代理的执行在第 16 章做了进一步的说明。状态代理（事务）和呼叫状态代理是不同的。

### Stateless Proxy

 无状态代理是一个逻辑实体，它不会保持用户端和服务器端之间的事务状态机。无状态代理前转从下游收到的每个请求，前转从上游收到的每一个响应。



## 报文结构

```
start-line
message-header
CRLF   //一个表示头结束的空行
[ message-body ]
```

### Requests

![image-20231227103252240](/Users/huanghaochen/Library/Application Support/typora-user-images/image-20231227103252240.png)

Method: 此规范定义了六个方法: REGISTER 支持注册联系消息，INVITE，ACK，和CANCEL 支持会话创建，BYE 支持结束会话，OPTIONS 支持对服务器的能力查询。SIP 拓展中定义了其他的方法。

### Responses

![image-20231227103448441](/Users/huanghaochen/Library/Application Support/typora-user-images/image-20231227103448441.png)

#### **首行（start-line）**

分请求行(Requests)和状态行(Responses)

- **请求行**: 由**请求类型、请求目的地址和协议版本号**构成。请求类型有：INVITE,ACK,OPTIONS,BYE,CANCEL和REGISTER。
- **状态行**: 是被叫方向主叫方返回的状态信息，如1xx，2xx，3xx，4xx，5xx，6xx。

##### 请求类型

- **INVITE**：用于发起呼叫请求。INVITE消息包括消息头和数据区两部分。INVITE 消息头包含主、被呼叫的地址，呼叫主题和呼叫优先级等信息。数据区则是关于会话媒体的信息，可由会话描述协议SDP 来实现。
- **BYE**：当一个用户决定中止会话时，可以使用BYE 来结束会话。
- **OPTIONS**：用于询问被叫端的能力信息，但OPTIONS 本身并不能发起呼叫。
- **ACK**： 对已收到的消息进行确认应答。
- **REGISTER**：用于用户向SIP服务器传送位置信息或地址信息。
- **CANCEL**：取消当前的请求，但它并不能中止已经建立的连接。

##### 状态类型

![image-20231228150803886](/Users/huanghaochen/Library/Application Support/typora-user-images/image-20231228150803886.png)

- 1xx：临时消息：表示表示请求消息已经收到，后面将继续处理该请求。
- 2xx：成功消息：表示请求已经被成功的理解、接受或执行。
- 3xx：重定向消息：表示为了完成请求还需采取更进一步的动作。
- 4xx：客户机错误：表示该请求含有语法错误或在这个服务器上不能被满足。
- 5xx：服务器错误：表示该服务器不能处理一个明显有效的请求。
- 6xx：全局性故障：表示该请求在任何服务器上都不能被实现。

#### **消息头（message-header）**

- **TO**： 格式：`TO: 显示名<接收者URI>;tag=n`,显示名和tag可选。接收者URI是SIP网络种唯一标识接收终端的标识符。例：`TO: Name<SIP:caller@WORK.COM>;TAG=11111`或 `TO: sip:caller@work.com`
- **FROM**: 给出标识会话发起者的URI。比如：FROM: `sip:caller@work.com;tag=hyh8` `tag`是必需的。
- **CALL-ID**: 用于全局唯一标识正在建立的会话的标识符。 随机数加UAC标识信息。
- **CSeq**: 用于标识同一会话中不同事务的序号，通常由一个用作序号的整型数和消息类型组成。整个会话操作过程由不同的事务组成，每一事务所涉及的消息的CSeq序号必须相同。
- **Via**: 为响应消息提供传输路径，当请求消息经过每一跳节点时，每一跳节点都把自身的IP地址信息放入顶层Via中。响应消息则沿着请求消息记录下的传输路径反向传输，首先移走指明自身IP地址信息的顶层消息头

#### **消息体**

SIP协议一个最主要的作用就是协商媒体信息。媒体信息通过message-body携带，基于SDP会话描述协议。对于PSTN语音编码格式，主要有G711A、G711U、G729等。

SIP协商中主叫方会带上自己支持的所有音频编码列表到被叫方，被叫方一般在回铃时从主叫支持的类型中选出一种或多种自己支持的编码，返回主叫后，双人按顺序选出第一个支持的编码。

### INVITE 消息

#### (1)起始行(start-line)：

```
<Method> <URI> <SIP_VERSION>
INVITE sip:some@192.168.31.131:50027 SIP/2.0
```

- Method是请求方法，本例是INVITE, SIP协议规定的Method有六种: INVITE, ACK, CANCEL用于创建对话，BYE用于结束对话, REGISTER用于登记,OPTIONS用于查询服务器能力
- URI表示所请求的用户或服务器, 也支持 “tel” URI， 本例是sip:some@192.168.31.131:50027,
- SIP_VERSION是 SIP版本号，本例是 SIP/2.0

#### (2)消息头部(header)

一个请求消息头部至少要包含六个字段：Via, To, From, CSeq, Caller-ID, Max-Forwards

 name : value ; value;

##### I. Via字段

```
Via: SIP/2.0/UDP 192.168.31.131:51971;rport;branch=z9hG4bKiYblddPPX
```

- Via头字段保存所经过SIP网元(客户端或Proxy)的主机名或网络地址（可能还有端口号），消息中的所有Via头字段对请求消息而言，从下至上依次表示到当前所在SIP网元为止，请求消息所经过的路径；对响应消息而言，从上至下依次表示从当前网元开始，响应所应遵循的路径。

- Via字段包含SIP协议版本以及消息传输所用的传输协议, 此例为: SIP/2.0/UDP
- branch参数:
  - 在SIP网元（UAC或Proxy）发出或转发请求消息时，在其插入的Via字段中必须包含branch参数，该参数用于标识此请求消息所创建的事务。
  - branch 参数可以用做loop detection，这时参数必须被分成两部分：第一部分符合一般的原则（对于RFC3261，z9hG4bK），第二部分(此例为iYblddPPX)被用来实现loop detection以用来区分loop和spiral。
  - loop和spiral均指Proxy收到一个请求后转发，然后此转发的请求又重新到达该Proxy，区别是loop中请求的Request-URI以及其他影响Proxy处理的头字段均不变，而Spiral请求中这些部分必需有某个发生改变，spiral发生的典型情况是Request-URI发生改变。Proxy在插入Via字段前，其branch 参数的loop.
  - detection部分依据以下元素编码：To Tag，From Tag，Call-ID字段，Request-URI，Topmost Via字段，Cseq的序号部分（即与request method无关），以及proxy-require字段，proxy authorization字段。注意：request method不能用于计算branch参数，比如CANCEL以及非2XX response的ACK与其所cancel的request或对应的INVITE属于同一个事务，即其branch参数相同。见RFC3261 P22 P25 P39 P95 P105

##### II. Max-Forwards 字段

```shell
    Max-Forwards: 70
```

- Max-Forwards 字段表示request到达UAS的跳数的限制。是一个整数，经过每一跳时减去一。如果Max-Forwards已经是零，可是request还没有到达目的地，则就会产生一个483(too many hops)响应

##### III. To字段

```shell
   To: <sip:some@192.168.31.131:50027>
```

- To字段表示消息的接收者
- To 字段可以有一个tag参数，to tag代表dialog的对等参与者（peer）。在UAC发出一个初始Dialog的请求（如INVITE）时，即发出out-of-dialog请求时，由于dialog还没有建立，不含to tag参数。当UAS收到INVITE请求时，在其发出的2xx或101-199响应中设置to tag参数，与UAC设置的From Tag参数以及Call-ID（呼叫唯一标识）一起作为一个Dialog ID（对话唯一标识，包含To tag，From Tag，Call-ID）的一个部分。RFC3261规定只有INVITE请求与2xx或101-199响应可以建立Dialog（由101-199响应创建的Dialog称为early dialog）。见RFC3261 P70

##### IV. From字段

```shell
   From: <sip:null@null>;tag=Prf3c3Xc
```

- From字段表示消息的发送者
- From字段必须包含tag参数，在UAC发出一个out-of-dialog请求（对话建立请求）时，必须设置一个唯一的tag参数，作为Dialog ID的一个部分。

##### V. Call-ID字段

邀请id

```shell
   Call-ID: cenXTa4i-1423587756904@appletekiAir
```

- 是一个邀请(Invitation)或来自同一个UAC用户的所有登记请求(Registeration，包括更新登记，取消登记)以及由此产生的一组响应的唯一标识。**一个邀请可以建立多个Dialog**（当被叫用户有多个联系方式时），这成为Forking，因而Call-ID只是一次呼叫邀请的唯一标识，Call-ID与UAC在发出请求中设置的From Tag字段以及UAS在其相映中设置的To Tag字段三者一起作为一个Dialog-ID。
- 在一个Dialog中，所有的requests和responses的Call-ID必须一致 同一UA的每一个register 的Call-ID必须一致。

##### VI. CSeq 字段

```shell
   CSeq: 1 INVITE
```

- 用于在同一个Dialog中标识及排序事务（transaction）以及区分新的请求 与请求的重发。
- CSeq包括顺序号和方法（method），方法必须和它所对应的request相匹配。对于out-of-dialog的非**register request，取值任意**。
- 对于dialog内的每一个新的request（如BYE,re-INVITE,OPTION），Cseq的序号加1。但是对于CANCEL,ACK除外。对于ACK而言，Cseq的序号必须与其所对应的request相同。对于CANCEL而言，Cseq的序号也必须与其cancel掉的request相同。
- 注意：在同一个对话中的UAC和UAS分别维护自己的CSeq序号，他们发出请求的CSeq序号是不相关的。

##### VII. Contact 字段

```shell
Contact: <sip:null@192.168.31.131:51971;transport=UDP>
```

- 对于非Register事务，Contact header field 主要提供了UAC或UAS的 直接联系SIP URI，UAC在发出的对话建立（out-of-dialog）INVITE请求的Contact字段中提供自己的直接联系SIP URI，在UAS收到该请求后在其发出响应的Contact字段中提供自己的直接联系SIP URI，这样在建立对话后，**UA间可以通过对方的直接联系SIP URI绕过Proxy直接发送请求。**
- 对于Register事务，表示地址绑定中的contact address（vs. address-of-record）
- Contact header field contains IP address and port on which the sender is awaiting further requests sent by callee. Other header fields are not important and will be not described here.

##### VIII. Content-Type字段

```shell'
     Content-Type: application/sdp
```

- 主要表示发给接收器的消息体的媒体类型。如果消息体不是空的，则Content-type header field一定要存在。如果Content-type header field存在，而消息体是空的，表明该类型的媒体流长度是0。

##### VIIII. Content-Length字段

```shell
    Content-Length: 215
```

 表示消息体的长度。是十进制数。

#### (3)消息体(message body)

```
v=0            //版本号为0
o=user1 685988692 621323255 IN IP4 192.168.31.131 //建立者用户名＋会话ID＋版本＋网络类型＋地址类型＋地址 
s=-            //会话名
c=IN IP4 192.168.31.131  //连接信息：网络类型＋地址类型＋地址
t=0 0         //会话活动时间 起始时间＋终止时间
m=audio 49432 RTP/AVP 0 8 101   //媒体描述：媒体＋端口＋传送＋格式列表
																	音频 ＋ 端口49432 ＋ 传输协议RTP ＋ 格式AVP，有效负荷0（u率PCM编码） 
																	
a=rtpmap:0 PCMU/8000  //0或多个会话属性： 属性 ＋ 有效负荷＋ 编码名称 ＋ 抽样频率。

a=rtpmap:8 PCMA/8000  // rtpmap ＋   0型  ＋  PCMU  ＋  8KHz 

a=rtpmap:101 telephone-event/8000

a=sendrecv  //a 可以有多个， 见SDP协议


```

# REGISTER

allows a central server (registrar) to s**tore the location of a SIP User-Agent.**

![Picture showing a typical registrar](https://www.kamailio.org/docs/tutorials/sip-introduction/figures/registrar.png)

![img](https://blog.wildix.com/wp-content/uploads/2018/07/Register-message.jpg)

```
REGISTER sip:10.10.1.99 SIP/2.0
CSeq: 1 REGISTER
Via: SIP/2.0/UDP 10.10.1.13:5060;
 branch=z9hG4bK78946131-99e1-de11-8845-080027608325;rport
User-Agent: MySipClient/4.0.0
From: <sip:13@10.10.1.99>
 ;tag=d60e6131-99e1-de11-8845-080027608325
Call-ID: e4ec6031-99e1
To: <sip:13@10.10.1.99>
Contact: <sip:13@10.10.1.13>;q=1
Allow: INVITE,ACK,OPTIONS,BYE,CANCEL,SUBSCRIBE,NOTIFY,REFER,MESSAGE,
 INFO,PING
Expires: 3600
Content-Length: 0
Max-Forwards: 70
```

- User-Agent: indicates the SIP Client connecting; most devices will indicate here the manufacturer – product name – software version and other information, such as the MAC address
- To: similar to *From*, in the case of the registration this field is usually the same as *From*. The tag is missing here but will be filled up by the SIP Server during the reply to the *REGISTER*
- Contact: usually indicates where the reply should go to, if rport was not set
- Expires: the desired registration duration in seconds

[SIP：松散路由与严格路由-CSDN博客](https://blog.csdn.net/m0_37915666/article/details/115026427)

## OK

```
SIP/2.0 200 OK
Via: SIP/2.0/UDP 192.168.1.30:5060;received=66.87.48.68
From: sip:sip2@iptel.org
To: sip:sip2@iptel.org;tag=794fe65c16edfdf45da4fc39a5d2867c.b713
Call-ID: 2443936363@192.168.1.30
CSeq: 63629 REGISTER
Contact: Msip:sip2@66.87.48.68:5060;transport=udp>;q=0.00;expires=120
Server: Sip EXpress router (0.8.11pre21xrc (i386/linux))
Content-Length: 0
Warning: 392 195.37.77.101:5060 "Noisy feedback tells:  
  pid=5110 req_src_ip=66.87.48.68 req_src_port=5060 in_uri=sip:iptel.org 
  out_uri=sip:iptel.org via_cnt==1"
```

### BYE

```
BYE sip:info@hypotenuse.example.org SIP/2.0
Via: SIP/2.0/TCP port443.hotmail.example.com:54212;branch=z9hG4bK312bc
Max-Forwards:70
To: <sip:info@hypotenuse.example.org>;tag=63124
From: <sip:pythag42@hotmail.example.com>;tag=9341123
Call-ID: 34283291273
CSeq: 47 BYE
Content-Length: 0
```

应答只能由对端UA生成。如果UA收到未知的BYE请求，那么它应答回应481 Dialog/Transaction Does Not Exist

### ACK

```
ACK sip:laplace@mathematica.example.org SIP/2.0
Via: SIP/2.0/TCP 128.5.2.1:5060;branch=z9hG4bK1834
Max-Forwards:70
To: Marquis de Laplace <sip:laplace@mathematica.example.org> ;tag=90210
From: Nathaniel Bowditch <sip:n.bowditch@salem.example.com> ;tag=887865
Call-ID: 152-45-32-N-32-23-47-W
CSeq: 3 ACK
Content-Type: application/sdp
Content-Length: ...
 
v=0
o=bowditch 2590844326 2590944532 IN IP4
s=Bearing
c=IN IP4 salem.example.org t=0 0
m=audio 32852 RTP/AVP 96 0
a=rtpmap:96 SPEEX/8000
a=rtpmap:0 PCMU/8000
```

​    ACK方法用于确认收到INVITE请求的最终应答。其它请求方法不需要确认。

​    ACK的CSeq**序号不变**，但方法描述变成ACK。这有助于UAS匹配对应的INVITE事务。

​    ACK消息可以携带application/sdp消息体。如果初始INVITE没有携带SDP信息，就允许在ACK消息中携带。如果INVITE带了SDP消息体，那么就不应该在ACK消息中携带SDP消息体。不能用ACK方法变更初始INVITE所描述的媒体信息，如果需要变更，必须使用re-INVITE或UPDATE方法。在ACK中携带SDP的方式常用于与其它协议交互的场景，特别是在发初始INVITE时不能获取媒体特征的场景。

​    
