# DPDK

## 一、DPDK基础

- DPDK的出现充分释放了IA（Intel Architure）平台对包处理的吞吐能力。随着吞吐率的上升，中断触发的开销是不能忍受的，DPDK通过一系列**软件优化**的方法，利用IA平台硬件特性，提供完整的底层开发支持库
  - 大页利用
  - cache对齐
  - 线程绑定
  - NUMA感知
  - 内存通道交叉访问
  - 无锁化数据结构
  - 预取
  - SIMD指令利用
- 可以说，由DPDK加速的用户态协议栈将会越来越多地支撑起计算节点上的网络服务

# VPP

## 一、vpp的软件框架构成

| 名称（由外到内） | 作用                                                         |
| ---------------- | ------------------------------------------------------------ |
| Plugins          | 包含越来越丰富的数据平面插件集,可以认为每一个插件是一个小型的应用app |
| VNET             | 与VPP的网络接口（第2,3和4层）协同工作，执行会话和流量管理，并与设备和数据控制平面配合使用 |
| VLIB             | 矢量处理库。vlib层还处理各种应用程序管理功能                 |
| VPP Inf          | VPP基础设施层，包含核心库源代码。该层执行内存函数，与向量和环一起使用， |

## 二、VPP插件开发

每一个插件在vpp里面有不同的node构成，每一个node主要分为以下四种类型：

1. VLIB_NODE_TYPE_INTERNAL
   内部节点，最典型的节点接收缓冲向量，执行操作。vpp大部分节点是这个角色，主要对数据流做内部处理，比如ip4-input-no-checksum/ip4-icmp-input等内部功能节点
2. VLIB_NODE_TYPE_INPUT
   输入节点，通常是设备输入节点。从零开始创建框架并分派到内部节点(internal), 比如dpdk-input/af-packet-input节点，
   input节点收包模式分为轮询和中断两种模式
3. VLIB_NODE_TYPE_PRE_INPUT
   输入节点前处理的节点，暂时在vpp里面没用用到
4. VLIB_NODE_TYPE_PROCESS
   线程节点，和线程一样，可以暂停、等待事件、恢复，不同于pthread，他是基于setjump/longjump实现的线程.
   等待一个事件：always_inline f64 vlib_process_wait_for_event_or_clock (vlib_main_t * vm, f64 dt)
   发送一个事件: always_inline void vlib_process_signal_event (vlib_main_t * vm, uword node_index, uword type_opaque, uword data)

## 三、vpp常用结构体

- vlib_main_t：记录全局信息。比如一些统计数据，Node Graph，注册的functions，是整个VPP的入口

**Node Graph相关结构体：主要用于记录node_graph相关信息**

- vlib_node_main_t：节点图主结构，记录全局节点图的数据信息
- vlib_node_t：单个节点的主结构，包括节点的处理功能函数，名称，节点类型等，主要保存一些相对静态信息
- vlib_node_registration_t：注册node节点时使用，包括节点业务逻辑的函数地址、节点状态、节点名称等
- vlib_node _runtime_t：实际在调度node过程中使用的结构，主要记录在处理过程中的信息变动
- vlib_frame_t：保存每个node对应的要处理的数据的内存地址信息
- vlib_pending_frame_t：记录运行节点索引、数据包索引、下一个数据包索引
- vlib_next_frame_t：获取node要处理的下一跳信息，定位node下一跳信息

**插件结构体 ：插件是结合一些节点信息提供业务功能，在VPP启动时通过加载动态so库来加载**

- Plugin_main_t(在/src/lib/unix目录)：用于记录全局所有静态插件信息，如：插件路径，插件信息，依赖的全局节点图信息等，在VPP启动加载插件时使用
- Plugin_config_t：记录单个插件配置状态信息
- Plugin_info_t：记录插件的信息：包括名字，文件名等，主要用于在VPP启动时加载插件动态库

**Feature结构体：Feature是在VPP启动时通过INIT链表进行加载。在VPP的实现中，每个feature包含一个node，每个node归属于一个ARC的集合。通过feature和插件方式，可以实现不改变源代码，在VPP中动态加载业务节点**

- vnet_feature_main_t：feature的主结构体，包含注册的ARC和feature列表等
- vnet_feature_arc_registration_t：arc中的feature按照代码指定的顺序串接起来。arc结构将记录这组feature的起始node和结束node。系统初始化时完成每个feature的连接
- vnet_feature_registration_t：：feature结构体，一个feature包含一个node，通过该结构体指定node和归属的arc和相对应位置 

# Docker

|            | Docker容器             | 虚拟机                      |
| ---------- | ---------------------- | --------------------------- |
| 操作系统   | 与宿主机共享OS         | 宿主机OS上运行虚拟机OS      |
| 存储大小   | 镜像小，便于存储和传输 | 镜像庞大（vmdk、vdi等）     |
| 运行性能   | 几乎无额外性能损失     | 操作系统额外的CPU、内存消耗 |
| 移植性     | 轻便、灵活             | 笨重，与虚拟化技术耦合度高  |
| 硬件亲和性 | 面向软件开发者         | 面向硬件运维者              |
| 部署速度   | 秒级                   | 10s以上/分钟级              |

**三要素：仓库、镜像、容器**

## 一、运行helloworld镜像

 docker run hello-world

## 二、docker帮助命令

docker version
docker info
docker --help(推荐)

## 三、docker镜像命令

**docker images：列出本地主机上的镜像**
				-a：列出本地所有镜像（含中间镜像层）
				-q：只显示镜像ID
				--digests：显示镜像的摘要信息
				--no-trunc：显示完整的镜像信息
**docker search 镜像名字，如tomcat等**
						--no-trunc：显示完整的镜像信息
						-s：列出收藏数不小于指定值的镜像
						--automated：只列出automated build类型的镜像
//该命令上docker.hub.com上搜索镜像
**docker pull 镜像名字**
**docker rmi 镜像名字ID，可多个**
					-f：强制删除
**docker rmi -f $<docker images -qa>：删除全部**

## 四、docker容器命令

**//启动docker容器**
  **docker run [OPTIONS] IMAGE [COMMAND][ARG...]**
  -d：后台运行容器，并返回容器ID，也即启动守护式容器（不用交互）
  -i：以交互模式运行容器，通常与-t同时使用
  -t：为容器分配一个伪输入终端，通常与-i同时使用
  --name：为容器指定一个名称  
  -p：主机端口：docker容器端口  （docker run -it -p 8080:8080 tomcat）
  -P：随机分配端口  
 --net = none：无网络模式启动（docker使用网桥可以通过ovs-docker实现）

  **//列出所有正在运行的容器**
  **docker ps**
  -a：列出当前所有正在运行的容器+历史运行过的
  -l：显示最近创建的容器
  -n：显示最近n个创建的容器
  -q：静默模式，只显示容器编号
  --no-trunc：不截断退出

  **//退出容器**
  exit：容器停止退出
  ctrl+P+Q：容器不停止退出 

**//启动容器  
docker start 容器ID或者容器名**

**//重启容器  
docker restart**

**//停止容器  
docker stop ID**

**//强制停止容器  
docker kill ID**

**//删除已停止的容器  
docker rm 容器ID**

**//查看容器日志  
docker logs**  
-t：加入时间戳  
-f：跟随最新的日志打印  
--tail 数字：显示最后多少条

**//查看容器内运行的进程  
docker top 容器ID**

**//查看容器内部细节  
  docker inspect**

**//进入正在运行的容器并以命令行交互**  
docker attach 容器ID ：进入容器，不会启动新的进程   
docker exec -t 容器ID：在容器中打开新的终端，启动新的进程

**//从容器内拷贝文件到主机上**  
docker cp 容器ID**:**容器内路径 目的主机路径

## 五、docker镜像

镜像是一种轻量级、可执行的独立软件包，用来打包软件运行环境和基于运行环境开发的软件，它包含运行某个软件所需的所有内容，包括代码、运行时、库、环境变量和配置文件

- 底层原理：UnionFS（联合文件系统）
- 分层的镜像：最大的好处就是共享资源，多个镜像从相同的base镜像构建而来，宿主机只在磁盘上保存一份base镜像

//**提交镜像**  
docker commit -a=" " -m= " " 容器ID 镜像名 

## 六、docker容器数据卷

数据共享+数据持久化

-  将运用与运行的环境打包行程容器运行，运行可以伴随着容器，但是我们对数据的要求希望是持久化的
- 容器之间希望有可能共享数据
- docker容器产生的数据，如果不通过commit，那么容器删除后，数据也没有了
- 为了能保存数据在docker中我们使用卷

1. 直接命令添加容器数据卷启动容器

   docker run -it -v /宿主机绝对路径目录:/容器内目录 镜像名

   docker run -it -v /宿主机绝对路径目录:/容器内目录:ro 镜像名(只读)

2. dockerfile添加  

   ```dockerfile
      #volume test
      FROM contos
      VOLUME["/dataVolumeContainer1","/dataVolumeContainer2"]  
      CMD echo "finished,------success1"
      CMD /bin/bash
   ```

   等价于  
      docker run -it -v /host1:/dataVolumeContainer1 -v /host2:/dataVolumeContainer2 centos /bin/bash  
   因为没有指定宿主机路径，用docker inspect ID命令查看Volume的宿主机默认路径

docker build -f /mydocker/Dockerfile -t zzyy/centos

## 七、dockerfile解析

1. 步骤：
   1. 手动编写dockerfile
   2. docker build执行，获得一个镜像
   3. docker run
   
2. 执行过程

   1. docker从基础镜像运行一个容器
   2. 执行一条指令并对容器作出修改
   3. 提交一个新的镜像层
   4. 基于刚提交的镜像运行一个新容器
   5. 执行dockerfile的下一条指令直到所有指令都执行完成

3. docker体系结构（保留字指令）

   1. FROM:基础镜像，当前新镜像是基于哪个镜像的
   2. MAINTAINER：镜像维护者的姓名和邮箱地址
   3. RUN：容器构建时需要运行的命令
   4. EXPOSE：当前容器对外暴露出的端口
   5. WORKDIR：指定在创建容器后，终端默认登录进来的工作目录库，一个落脚点
   6. ENV：用来在构建镜像过程中设置环境变量
   7. ADD：将宿主机目录下的文件拷贝进镜像且ADD命令会自动处理URL和解压tar压缩包
   8. COPY：类似ADD，拷贝文件和目录到镜像中
   9. VOLUME：容器数据卷，用于数据保存和持久化工作
   10. ==CMD：指定一个容器启动时要运行的命令。==Dockerfile中可以有多个CMD命令，但只有最后一个生效，CMD会被docker run之后的参数替换
   11. ==ENTRYPOINT：指定一个容器启动时要运行的命令。==ENTRYPOINT的目的和CMD一样，都是在指定容器启动程序及参数
   12. ONBUILD：当构建一个被继承的dockerfile时运行命令，父镜像在被子镜像继承后，父镜像的onbuild被触发

   | BUILD         | BOTH    | RUN        |
   | ------------- | ------- | ---------- |
   | FROM          | WORKDIR | CMD        |
   | MAINTAINER    | USER    | ENV        |
   | COPY          |         | EXPOSE     |
   | ADD           |         | VOLUME     |
   | RUN           |         | ENTRYPOINT |
   | ONBUILD       |         |            |
   | .dockerignore |         |            |

   

# OVS(Open vSwitch)

首先创建一个网桥：ovs-vsctl add-br br0;  
绑定某个网卡：ovs-vsctl add-port br0 eth0；

- 数据包流向：
  1. 经过网卡eth0
  2. 到openVswitch的端口vport上进入openVswitch中
  3. 根据key值进行流表的匹配
     - 匹配成功：根据流表（flowtable）中对应的action完成动作
     - 匹配不成功，执行默认操作，有可能是放回内核网络协议栈中去处理。（在创建网桥时就会相应的创建一个端口连接内核协议栈）
- 常用修改代码位置
  1. 在datapath.c中的ovs_dp_process_received_packet(struct vport *p, struct sk_buff *skb)函数内添加相应的代码来达到自己的目的，因为对于每个数据包来说这个函数都是必经之地；
  2. 设计自己的流表；可以根据流表来设计自己的action，完成自己想要的功能

- 术语与概念

  - Bridge：网桥。在OpenvSwitch中每个虚拟交换机（vswitch）都可以认为是一个网桥。OVS在底层的通信是借助了网桥模块来实现的，用brctl也能查看到ovs所创建的网桥设备
  - datapath：负责执行数据交换。在内核空间实现把从接收端口收到的数据包在流表中进行匹配，并执行匹配到的动作。一个datapath可以对应多个vport，关联一个flowtable
  - flowtable：数据流表。包含多个条目，每个条目包含两个内容：一个match/key和一个action
  - vport：端口。指虚拟交换机逻辑上的接口，可以通过ovs-vsctl命令查看各个网桥（即虚拟交换机）上接入的端口
  - patch：连线。网络节点和运算节点之间ovs虚拟交换的联结
  - tun/tap：Linux系统中的虚拟网络设备。tap等同于以太网设备，操作数据帧；tun模拟网络层设备，操作ip数据包

  下图红色箭头为数据流向

![image-20201105151830266](C:\Users\logan\AppData\Roaming\Typora\typora-user-images\image-20201105151830266.png)

# GRE协议

GRE （Generic Routing Encapsulation，通用路由封装协议）对某些网络层协议（如：IP，IPX，AppleTalk等）的数据报文进行封装，使这些被封装的数据报文能够在另一个网络层协议（如IP）中传输。GRE协议实际上是一种==封装协议==，==它提供了将一种协议的报文封装在另一种协议报文中的机制，使报文能够在异种网络中传输==。异种报文传输的通道称为tunnel（隧道）。相关RFC标准，1701，1702，2637，2784，2890。

# VXLAN

## 一、定义

Virtual eXtensible Local Area Network（VXLAN(Virtual Extensible LAN)虚拟可扩展局域网，是一种 overlay 网络技术，将原始2层以太网帧进行UDP封装 (MAC-in-UDP)，增加8字节 VXLAN头部，8字节 UDP头部， 20字节 IP 头部和14字节以太网头部，共50字节。

## 二、术语

**1. VTEP**

VXLAN Tunnel Endpoint (VTEP)。VXLAN使用VTEP设备对VXLAN报文进行封装与解封装，包括ARP请求报文和正常的VXLAN数据报文，VTEP将原始以太网帧通过VXLAN封装后发送至对端 VTEP设备，对端VTEP接收到 VXLAN报文后解封装然后根据原始 MAC进行转发，VTEP可以是物理交换机、物理服务器或者其他支持 VXLAN的硬件设备或软件来实现。

**2. VNI**

Virtual Network ID ( VNI)， VNI封装在 VXLAN头部，共 24-bit ，***支持16,000,000 逻辑网络。

**3. VXLAN 网关**

VXLAN网关用于连接 VXLAN网络和传统 VLAN网络，VXLAN网关实现 VNI和VLAN ID 之间的映射， VXLAN 网关实际上也是一台 VTEP设备。

**4. 组播组**

VTEP设备要加入相同的组播组，主要用于控制平面地址学习。

端口、网桥

- ovs-vsctl add-br ovs1
- ovs-vsctl show 
- ovs-vsctl add-port 网桥名 端口名
  -  -- set interface 端口名 type=vxlan options:remote_ip=10.0.2.12 options:key=100 
- ovs-docker add-port 网桥名 接口名 容器ID 
  - --ipaddress=192.168.1.1/24
- ovs-vsctl list interface 端口名 | grep ofport

流表

- ovs-ofctl dump-flows 网桥名
- ovs-ofctl del-flows 网桥名
- ovs-ofctl add-flow 网桥名 "priority = 1, in_port = 3, actions=output:4"
- ovs-ofctl add-flow 网桥名 "priority = 2,in_port =4, actions = drop"

```dockerfile
FROM centostcpdump
RUN yum install gcc-c++
RUN yum install -y pcre pcre-devel
RUN yum install -y zlib zlib-devel
RUN yum install -y openssl openssl-devel
RUN yum -y install nginx
RUN yum -y install wget
RUN cd /usr/local/
RUN wget http://nginx.org/download/nginx-1.8.0.tar.gz
RUN tar -zxvf nginx-1.8.0.tar.gz
RUN cd nginx-1.8.0
RUN ./configure --user=nobody --group=nobody --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_gzip_static_module --with-http_realip_module --with-http_sub_module --with-http_ssl_module
RUN make
RUN make install
CMD /bin/bash
```

docker run -it  --name centnginx --net=none --privileged centostcpdump /bin/bash

 cd /usr/local/nginx/sbin/

./nginx