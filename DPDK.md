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



