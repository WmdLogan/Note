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
-  容器之间希望有可能共享数据
-  docker容器产生的数据，如果不通过commit，那么容器删除后，数据也没有了
-  为了能保存数据在docker中我们使用卷

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

   