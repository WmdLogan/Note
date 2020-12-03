# OVS(Open vSwitch)

- ovs-vsctl add-br ovs1
- ovs-vsctl show 
- ovs-vsctl add-port 网桥名 端口名
  -  -- set interface 端口名 type=vxlan options:remote_ip=10.0.2.12 options:key=100 
- ovs-docker add-port 网桥名 接口名 容器ID 
  - --ipaddress=192.168.1.1/24
- ovs-vsctl list interface 端口名

流表

- ovs-ofctl dump-flows 网桥名
- ovs-ofctl del-flows 网桥名
- ovs-ofctl add-flow 网桥名 "priority = 1, in_port = 3, actions=output:4"
- ovs-ofctl add-flow 网桥名 "priority = 2,in_port =4, actions = drop"

docker run -it  --name centnginx --net=none --privileged centostcpdump /bin/bash

 cd /usr/local/nginx/sbin/

./nginx