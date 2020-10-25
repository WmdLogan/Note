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

```
 docker run hello-world
```

## 二、docker帮助命令

```
docker version
docker info
docker --help(推荐)
```

## 三、docker常用命令

```
docker images：列出本地主机上的镜像
				-a：列出本地所有镜像（含中间镜像层）
				-q：只显示镜像ID
				--digests：显示镜像的摘要信息
				--no-trunc：显示完整的镜像信息
docker search 镜像名字，如tomcat等
						--no-trunc：显示完整的镜像信息
						-s：列出收藏数不小于指定值的镜像
						--automated：只列出automated build类型的镜像
//该命令上docker.hub.com上搜索镜像
docker pull 镜像名字
docker rmi 镜像名字ID，可多个
					-f：强制删除
docker rmi -f $<docker images -qa>：删除全部
```

