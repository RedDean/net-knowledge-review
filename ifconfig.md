### 关于ipconfig

查看ip地址使用ip addr和ifconfig(Linux环境下)

无法使用这两个命令时，可安装net-tools和iproute2，运行ip addr。

例:

```shell
root@test:~# ip addr 1: lo:
root@test:~# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether fa:16:3e:c7:79:75 brd ff:ff:ff:ff:ff:ff
    inet 10.100.122.2/24 brd 10.100.122.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fec7:7975/64 scope link 
       valid_lft forever preferred_lft forever
```

IP地址和广播域的划分有关系。它是网卡的通讯地址，相当于门牌号。

例子中10.100.122.2是ip地址（ipv4）。以“.”划分四部分，每部分8bit，总共32位。

地址后 scope global,代表该网卡对外，接受各个地方的包。host代表仅本机通信。

lo为回环接口，往往会被分配到 127.0.0.1 这个地址。这个地址用于本机通信，经过内核处理后直接返回，不会在任何网络中出现。

fe80::f816:3eff:fec7:7975/64是ipv6,128位。

#### ip地址划分

如图，ip地址分为5类( 网络号+主机号 )。

![ip-type](./img/ip_type.jpg)

A、B、C 三类地址包含的主机数量:

![ip-cpu-num](./img/ip_cpu_num.jpg)

**为什么会有无类型域间选路（CIDR）?** 就在这儿了，C类地址主机数太少，B类又太多，用CIDR折中。

#### CIDR

上例：10.100.122.2/24，这种格式即是CIDR。

32位中，前24位为网络号，后8位为主机号。

##### 广播地址

10.100.122.255即是广播地址，发送该地址，所有10.100.122网段的机器都可以收到。

##### 子网掩码

上例中255.255.255.0为子网掩码。用来确定网络号。

将子网掩码和IP地址按位计算AND即可得网络号。

##### CIDR计算

16.158.165.91/22

前22位为网络号，16.158占16位，还剩6位。

165换算二进制位<10100101>，前6位为网络号，16.158.<101001>。机器号为<01>.91。

该例第一个地址为16.158.<101001><00>.1，即16.158.164.1。

子网掩码为,255.255.<111111><00>.0，即255.255.252.0。

广播地址为 16.158.<101001><11>.255，即16.158.167.255。

#### MAC地址

例子中 link/ether fa:16:3e:c7:79:75 brd ff:ff:ff:ff:ff:ff，为MAC地址，也为物理地址，16进制，6byte。

全局唯一。硬件角度，保证不同网卡有不同标识。

虽然MAC唯一，但是由于网络包传递中间要通过交换机、路由等设备，光有mac地址是不够的，需要ip地址来定位。

#### 网络设备状态标识

上例中 <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000

- UP: 网卡已启动
- BROADCAST: 该网卡有广播地址，可发送广播包
- MULTICAST: 多播包
- LOWER_UP: 网线
- mtu 1500：最大传输单元
- qdisc pfifo_fast: 数据包排队规则
