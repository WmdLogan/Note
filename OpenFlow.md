# OpenFlow协议

## 一、Introduction

​	这篇文章描述了OpenFlow Switch的基本组成，以及Openflow controller如何通过OpenFlow switch protocol来管理OpenFlow Switch。本文基于openflow-switch-v1.5.1版本。

## 二、Switch Components

![](D:\Logan\note\picture\1-1.png)

**图1 OpenFlow switch的主要组成部分**

一个OpenFlow Switch包含一个或多个flow table（流表），一个group table（组表，OpenFlow v1.1.0以上版本才有），以及用于openflow switch和controller之间通信的openflow channel（通道）。openflow switch和controller之间通过openflow switch protocol通信。

一个OpenFlow Switch中的flow tables是按照数字顺序排列的，起始索引号从 0 开始，任何进入到OpenFlow Switch的数据包都会从第一个flow table，即 table 0 开始处理，后续的Flow Table可能会被使用到，而这依赖于table 0中匹配成功的flow entry的instructions。

通过openflow switch protocol，controller可以添加、修改、删除flow table中的flow entry（流表项）。每个flow table中有一组flow entries；每个flow entry包含match fields（匹配域）、counters（计数器）和一组数据包操作的instructions（指令）。

flow entries会指向一个port。这个port通常是physical port，也可能是由switch定义的logical port或者reserved ports。

group table用于定义一组可被多个流表项共同使用的动作

Meter table用于计量和限速

## 三、OpenFlow Ports

OpenFlow ports是openflow switch用来传递数据包的网络接口。OpenFlow Switches之间也通过OpenFlow ports逻辑相连。一个数据包只可以从一个OpenFlow switch的ingress port进入，从OpenFlow switch的output port流出。一个openflow port必须有计数器，状态和配置。

一个OpenFlow switch必须同时支持physical ports, logical ports 和 reserved ports三种端口

- physical ports是由openflow switch定义的与交换机的硬件端口一一对应的端口。
- logical ports是交换机定义的建立在物理端口之上的高层抽象，用于完成某种特定的功能，如隧道、链路等。一个逻辑端口可以映射到多个物理端口。
- reserved ports由OpenFlow switch规范定义，用于通用的转发动作，如发送到控制器、洪泛或采用非openflow方法转发

## 四、FlowTable

![](D:\Logan\note\picture\1-2.png)**图2 包在处理管道的流通过程**

### 4.1、Pipeline Processing

openflow switch有两种类型，一种是OpenFlow-only，另一种是OpenFlow-hybrid。OpenFlow-only switch只支持openflow的操作，即所有的数据包都在openflow pipeline中处理；OpenFlow-hybrid switch既支持openflow的操作，也支持正常以太网交换机的操作。

每条openflow pipeline包含一个或多个flow table，每个flow table包含多条flow entry。openflow pipeline决定了flow tables之间如果交互。一台openflow switch必须至少有一个ingress flow table，可以有多个flow tables。

openflow switch中的flow table按照顺序进行编号，编号从0开始。pipeline processing有两种阶段，ingress processing和egrss processing。两种阶段的flowtable由第一个egress table区分，所有编号比它小的flow table都必须是ingress flow。

### 4.2、Flow Entry

每个Flow Entry由匹配域、优先级、计数器、指令集、计时器、Cookie和Flags组成：

![](D:\Logan\note\picture\1-3.png)

**图3 Flow Entry组成项**

Match Fields：匹配域，可能包含ingress port、数据包头信息以及前继Flow Table指明的Metadata等（Metadata：a maskable register that is used to carry information from one table to the next.）

Priority：匹配优先级，范围是0-65535，数字越大优先级越高

Counters：计数器，统计与该Flow Entry成功匹配的包数量

Instructions：指令集，应用到与该Flow Entry成功匹配的数据包

Timeouts：在该Flow Entry过期前的最大有效时间或者空闲时间

Cookie：被Remote Controller用来筛选Flow Statistics、Flow Modification或者Flow Deletion行为的指示值

flags：用来改变Flow Entry的管理方式

一个Flow Entry在Flow Table中通过Match Fields和Priority两个字段来唯一标识（或称为联合主键）

### 4.3、Flow Removal

　Flow Entry可以通过三种方式从Flow Table里被移除：

1. 通过Remote Controller直接发送移除Flow Entry的消息

2. 通过OpenFlow Switch的过期机制

   每一个Flow Entry有一个与之关联的 idle_timeout 和 hard_timeout。

   如果 hard_timeout 值不为 0 ，那么OpenFlow Switch需要记住这条Flow Entry的生成时间，当其生存时间达到指定的 hard_timeout 时间时，将被逐出Flow Table；

   如果 idle_timeout 值不为 0 ，那么OpenFlow Switch需要记住最后一次与该Flow Entry成功匹配的时间，当超过指定 idle_timeout 时间仍未有能成功匹配的数据包，那么该Flow Entry将会被逐出Flow Table。

3. 通过OpenFlow Switch的逐出机制。当OpenFlow Switch需要回收资源时，Flow Entries 也有可能被逐出Flow Table。该机制对于OpenFlow Switch实现者来说是一个可选的特性，其必须按照规范来决定哪些Flow Entries被逐出，可能会依照Flow Entry参数、资源映射关系或者其他内部约束等。

### 4.4、Matching

 	当一个数据包被一个Flow Table处理时，数据包会依据该流表中每条Flow Entry的Priority值与该Flow Table里的所有Flow Entry进行匹配，当发现匹配成功的Flow Entry时，该Flow Entry相关联的Instructions Set将会被执行，这些Instructions可能会将该数据包显示地直接转发到后续的其他Flow Table里（通过Goto指令）继续处理，在那里继续采取同样的方式来处理数据包。

​	这里需要注意的是，Flow Entry只会将数据包继续往前（往索引号比当前大的Flow Table）转发，而不会倒序转发。

​	假若一个数据包在一个Flow Table里没有发现能够匹配成功的Flow Entry，那么这叫作一次Table Miss。发生Table Miss后的动作取决于这个Flow Table的配置：1）直接丢弃，2）继续转发给后续的Flow Table，3）封装成 packet-in 消息发送给Remote Controller。

![](D:\Logan\note\picture\1-4.png)

**图4 流表匹配过程图**

### 4.5、Instructions

![](D:\Logan\note\picture\1-5.png)

**图5 flow table中的匹配和instruction执行过程**

每个flow entry包含一组instructions，当数据包匹配中时会执行这些instrctions，包含篡改数据包、改变action set、改变pipeline。instruction包含必选和options instruction和required instruction，required instruction是指openflow switch必须支持的instruction。下表列出一些常用instruction：

| instruction              | 类型                 | 说明                                                 |
| ------------------------ | -------------------- | ---------------------------------------------------- |
| Clear-Actions            | Required Instruction | 立即清除报文的action set中所有的动作                 |
| Write-Actions action     | Required Instruction | 它的行为是将自己包含的动作合并到报文的action set中   |
| Goto-Table next-table-id | Required Instruction | 它表示把报文交给后续的哪张流表处理                   |
| Apply-Actions action     | Optional Instruction | 它的行为是对报文应用这些指令，不改变报文的action set |

### 4.6、Action Set

一个action set和一个数据包相关联。一个action set默认是空的。一个Flow Entry可以通过Write-Actions或者Clear-Actions指令来修改Action Set，这个Action Set会在不同的Flow Table之间进行传递，当一个Flow Entry的Instructions里不包含Goto-Table指令时，那么整个Pipeline Processing就会在此Flow Entry停止，然后开始执行与该数据包关联的Action Set里所有的action

同样地，一个Action Set里每种类型的action最多只有一个，action类型大致如下所示：

| 类型              | 说明                                                     |
| ----------------- | -------------------------------------------------------- |
| copy TTL inwards  | 在数据包上执行copy TTL inwards操作                       |
| pop               | pop出数据包里所有的tag                                   |
| push-MPLS         | push MPLS tag到数据包里                                  |
| push-PBB          | push PBB tag到数据包里                                   |
| push-VLAN         | push VLAN tag到数据包里                                  |
| copy TTL outwards | 在数据包上执行copy TTL outwards操作                      |
| decrement TTL     | 降低数据包的TTL                                          |
| set               | 在数据包上执行set-fields操作                             |
| qos               | 执行与qos相关的操作，比如set_queue                       |
| group             | 转到指定的Group Table里继续执行其action(s)。             |
| output            | 如果没有group action指定，那么将数据包转发到指定的port里 |

　　Apply-Actions指令会触发立即执行Action Set里的action，并且这些action必须按照上述列表的顺序来依次执行，而不管这些action加入到Action Set的顺序，另外output action一定要是最后执行的；如果一个Action Set里同时存在output action和group action，那么此时group action将优先被执行，而output action将会被忽略，反之，如果一个Action Set里都不存在output action和group action，那么该数据包将会被丢弃。

### 4.7、Group Table

Group Table给OpenFlow Switch提供了更加高级的数据包转发特性（如select和all），一个group table包含多条group entry。每条group entry的组成如下

![](D:\Logan\note\picture\1-6.png)

**图6 group entry主要组成部分**

- group identifier： 32位无符号整型，作为openflow switch中识别group entry的唯一标识
- group type： 决定了group的语义
- counters： 统计被该group entry处理过的数据包数量
- action buckets：一个action bucket的有序列表，每个action bucket又包含了一组action set及其参数。

一条group entry可以有0个或多个buckets，但是indirect类型的group只有一个bucket。没有bucket的group直接drop数据包。

一个bucket包含actions，这个actions可以篡改数据包，也可以指明output port。

一个OpenFlow Switch不需要支持所有的Group Types，但必须支持下面三个类型：

1）all：Group Table中所有的Action Buckets都会被执行，这种类型的Group Table主要用于数据包的多播或者广播。

2）select：仅仅执行Group Table中的某一个Action Bucket，基于OpenFlow Switch的调度算法。

3）indirect：执行Group Table中已经定义好的Action Bucket，这种类型的Group Table仅仅只支持一个Action Bucket。允许多个Flow Entries或者Groups 指向同一个通用的 Group Identifier，支持更快更高效的聚合。这种类型的Group Table与那些仅有一个Action Bucket的Group Table是一样的。

### 4.8、Meter Table

　Meter Table同样是由多个Meter Enties构成，每个Meter Entry定义每个 Flow 的 meters。基于此结构，OpenFlow Switch可以实现各种简单的QoS功能（Quality of Service），比如速率限制等，再结合每个port的queues，可以实现更加复杂的QoS框架。

　　一个meter可以衡量与它关联的数据包的速率，并进而可以控制其聚合速率。任何一个Flow Entry都可以在其Instructions Set里指定某一个Meter，从而控制与该Flow Entry能够成功匹配的数据包的聚合速率。

## 五、OpenFlow Channel

OpenFlow Channel是控制器和交换机通信的通道。Controller可以通过该通道来配置和管理openflow switch、接收openflow switch发出的事件等。但是OpenFlow Channe的消息必须以OpenFLow switch protocol的格式封装。OpenFlow Channel通常采用TLS加密，但也支持直接使用TCP。常见端口有6633、6634、6653。

### 5.1、OpenFlow Switch Protocol Overview

OpenFLow Switch Protocol支持一下三种类型的消息：

- controller-to-switch： 由controller发出，用来管理或获取switch状态。
- asynchronous：异步类型消息，由openflow switch发出，将网络事件或者其自身状态变化告知controller
- symmetric：对称类型消息，可由openflow switch发出也可由controller发出，也不必通过请求建立

### 5.2、Controller-to-Switch

controller-to-switch类型消息由controller发出，switch回不回复皆可。

| 消息          | 说明                           |
| ------------- | ------------------------------ |
| Features      | 用来获取openflow switch特性    |
| Configuration | 用来配置openflow switch        |
| Modify-State  | 用来修改openflow switch状态    |
| Read-State    | 用来读取openflow switch状态    |
| Packet-out    | controller将数据包发送给switch |

### 5.3、Asynchronous

asynchronous类型消息无需经过controller请求，openflow switch就可发出。

| 消息         | 说明                                            |
| ------------ | ----------------------------------------------- |
| Packet‐in    | openflow switch将数据包发送给controller         |
| Flow‐Removed | openflow switch告知controller其自身流表被删除   |
| Port‐Status  | openflow switch告知controller其自身端口状态更新 |

### 5.4、Symmetric

symmetric类型消息，controller或openflow switch皆可主动发起

| 消息  | 说明                                                |
| ----- | --------------------------------------------------- |
| Hello | 用来建立openflow switch与controller之间的连接       |
| Echo  | 用来保持openflow switch与controller之间的连接状态   |
| Error | openflow switch或者controller告知对方其自身发生错误 |

### 5.5、报文实例

OpenFlow协议报文一般为如下格式：

![](D:\Logan\note\picture\1-7.jpg)

对应的结构体如下：

```c
/* Header on all OpenFlow packets. */
struct ofp_header {
uint8_t version; /* OFP_VERSION. */
uint8_t type; /* One of the OFPT_ constants. */
uint16_t length; /* Length including this ofp_header. */
uint32_t xid; /* Transaction id associated with this packet.
Replies use the same id as was in the request
to facilitate pairing. */
};
```

**图7 openflow协议报文格式**

![](D:\Logan\note\picture\1-8.png)

**图8 openflow协议报文实例**

#### 5.5.1、Hello类型

controller与openflow switch建立连接后，双方发送OFPT_HELLO消息进行版本的协商；若协议版本协商成功，则连接建立；否则。发送ERROR消息描述失败的原因并终止连接。

![](D:\Logan\note\picture\1-9.png)

**图9 switch与controller互相发送HELLO消息**

#### 5.5.2、Features类型

协商完成后，controller发送OFPT_FEATURES_REQUEST消息获取switch的信息，交换机回复OFPT_FEATERS_REPLY消息将交换机的详细参数告知给控制器。

![](D:\Logan\note\picture\1-10.png)

**图10 controller请求switch详细参数，switch告知参数**

Features Reply Message对应的结构体：

```c
struct ofp_switch_features{
    struct ofp_header header;
    uint64_t datapath_id; /*唯一标识 id 号*/
    uint32_t n_buffers; /*交缓冲区可以缓存的最大数据包个数*/
    uint8_t n_tables; /*流表数量*/
    uint8_t auxiliary_id; /* 识别辅助连接 */
    uint8_t pad[2]; /*align to 64 bits*/
    /* Features. */
    uint32_t capabilities; /* Bitmap of support "ofp_capabilities". */
	uint32_t reserved;
};
```

Features Reply Message结构如图11所示：

![](D:\Logan\note\picture\1-11.png)

**图11 switch回复OFPT_FEATERS_REPLY消息内容**

如果request或者reply的Openflow消息超过了64KB，则会将超出的部分封装在Multipart类型的消息中，如下图所示

![](D:\Logan\note\picture\1-11-1.png)

**图12 Multipart类型消息**

#### 5.5.3、Port-Status消息

如果port有添加、修改、删除，openflow switch需要用PORT-STATUS消息通知controller

![](D:\Logan\note\picture\1-12.png)

**图13 PORT-STATUS消息**

对应的结构体：

```c
/* A physical port has changed in the datapath */
struct ofp_port_status {
struct ofp_header header;
uint8_t reason; /* One of OFPPR_*. */
uint8_t pad[7]; /* Align to 64-bits. */
struct ofp_port desc;
};
```

#### 5.5.4、Configuration类型

controller发送OFPT_SET_CONFIG消息向switch下发配置参数,switch不需要回复。controller可以通过发送OFPT_GET_CONFIG_REQUEST消息查询switch的配置信息，switch发送OFPT_GET_CONFIG_REPLY消息进行回复。

对应的结构体如下

```c
/* Switch configuration. */
struct ofp_switch_config {
struct ofp_header header;
uint16_t flags; /* Bitmap of OFPC_* flags. */
uint16_t miss_send_len; /* Max bytes of packet that datapath
should send to the controller. See
ofp_controller_max_len for valid values.
*/
};

/*The flags field is a bitmap that uses a combination of the following configuration flags :*/
enum ofp_config_flags {
/* Handling of IP fragments. */
OFPC_FRAG_NORMAL = 0, /* No special handling for fragments. */
OFPC_FRAG_DROP = 1 << 0, /* Drop fragments. */
OFPC_FRAG_REASM = 1 << 1, /* Reassemble (only if OFPC_IP_REASM set). */
OFPC_FRAG_MASK = 3, /* Bitmask of flags dealing with frag. */
};
```

#### 5.5.5、Packet_In类型

有两种情况会触发switch向controller发送OFPT_PACKET_IN消息：

- 当switch收到一个数据包后，查找流表且没有匹配中时，则switch将数据包封装在 Packet-in 消息中发送给controller处理，这时候数据包会存放在switch缓存区中，不会被丢弃。
- 数据包在流表中有匹配的条目，但是其中所指示的 actions中包含转发给controller的动作，注意这时候数据包不会被放进缓冲区。

对应的结构体：

```c
struct ofp_packet_in {
    struct ofp_header header;
    uint32_t buffer_id; /* ID assigned by datapath. */
    uint16_t total_len; /* Full length of frame. */
    uint8_t reason; /* Reason packet is being sent (one of OFPR_*) */
    uint8_t table_id; /* ID of the table that was looked up */
    uint64_t cookie; /* Cookie of the flow entry that was looked up. */
    struct ofp_match match; /* Packet metadata. Variable size. */
    /* The variable size and padded match is always followed by:
* - Exactly 2 all-zero padding bytes, then
* - An Ethernet frame whose length is inferred from header.length.
* The padding bytes preceding the Ethernet frame ensure that the IP
* header (if any) following the Ethernet header is 32-bit aligned.
*/
    //uint8_t pad[2]; /* Align to 64 bit + 16 bit */
    //uint8_t data[0]; /* Ethernet frame */
};
```

![](D:\Logan\note\picture\1-13.png)

**图14 OFPT_PACKET_IN消息**

#### 5.5.6、Packet_Out类型

controller向switch发送OFPT_PACKET_OUT类型消息。

对应的结构体：

```c
/* Send packet (controller -> datapath). */
struct ofp_packet_out {
    struct ofp_header header;
    uint32_t buffer_id; /* ID assigned by datapath (OFP_NO_BUFFER if none). */
    uint16_t actions_len; /* Size of action array in bytes. */
    uint8_t pad[2]; /* Align to 64 bits. */
    struct ofp_match match; /* Packet pipeline fields. Variable size. */
    /* The variable size and padded match is followed by the list of actions. */
    /* struct ofp_action_header actions[0]; *//* Action list - 0 or more. */
    /* The variable size action list is optionally followed by packet data.
* This data is only present and meaningful if buffer_id == -1. */
    /* uint8_t data[0]; */ /* Packet data. The length is inferred
from the length field in the header. */
};

```

![](D:\Logan\note\picture\1-14.png)

**图14 OFPT_PACKET_OUT消息** 

#### 5.5.6、Modify-State类型

通过OFPT_FLOW_MOD消息向控制器下发流表操作

当交换机收到一个数据包并且交换机中没有与该数据包匹配的流表项时，交换机将此数据包封装到OFPT_PACKET_IN消息中发送给控制器，并且交换机会将该数据包缓存。控制器收到Packet-in消息后，可以发送OFPT_FLOW_MOD消息给交换机，给交换机添加流表项。OFPT_FLOW_MOD消息中的buffer_id字段设置为OFPT_PACKET_IN消息中的buffer_id值。

对应的结构体：

```c
struct ofp_flow_mod {
    struct ofp_header header;
    struct ofp_match match; /*流表的匹配域*/ 
    uint64_t cookie; /*流表项标识符*/
    uint8_t table_id;/*flow table的ID*/
    uint16_t command; /*可以是ADD,DELETE,DELETE-STRICT,MODIFY,MODIFY-STRICT*/
    uint16_t idle_timeout; /*空闲超时时间*/
    uint16_t hard_timeout; /*最大生存时间*/
    uint16_t priority; /*优先级，优先级高的流表项优先匹配*/
    uint32_t buffer_id; /*缓存区ID ,用于指定缓存区中的一个数据包按这个消息的action列表处理*/  
    uint16_t out_port; /*如果这条消息是用于删除流表则需要提供额外的匹配参数*/
    uint16_t flags; /*标志位，可以用来指示流表删除后是否发送flow‐removed消息，添加流表时是否检查流表重复项，添加的流表项是否为应急流表项。*/
    struct ofp_action_header actions[0]; /*action列表*/
};
/*flow table支持以下修改操作*/
enum ofp_flow_mod_command {
    OFPFC_ADD = 0, /* New flow. */
    OFPFC_MODIFY = 1, /* Modify all matching flows. */
    OFPFC_MODIFY_STRICT = 2, /* Modify entry strictly matching wildcards andpriority. */
    OFPFC_DELETE = 3, /* Delete all matching flows. */
    OFPFC_DELETE_STRICT = 4, /* Delete entry strictly matching wildcards andpriority. */
};

```



![](D:\Logan\note\picture\1-15.png)

**图15 OFPT_FLOW_MOD消息**

#### 5.5.7、ECHO类型

当没有其他的数据包进行交换时，交换机会定期给交换机发送OFPT_ECHO_REQUEST类型消息，交换机回复OFPT_ECHO_REPLY类型消息，用来查询连接状态，确保通信通畅。

![](D:\Logan\note\picture\1-16.png)

**图16 ECHO消息**

## 六、官方链接

[OpenFlow官方网站]: https://www.opennetworking.org/sdn-resources/openflow
[OpenFlow协议规范]: https://www.opennetworking.org/technical-communities/areas/specification
