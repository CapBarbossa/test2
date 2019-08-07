## 本文档使用PyCharm打开

0. 向系统中添加一个新的[网络命名空间][1].

    `ip netns add test1`
1. UP起来一个网络命名空间(netns: network namespace)
 
    `ip netns exec test1 ip link set dev lo up`
2. 那么问题来了,`ip a`和`ip link`究竟是什么不同呢?

        大概是ip a 返回的是IP地址,'a' for address;
        ip link返回的是与device相关的硬件的状态
3. 查看docker网络相关的东西

    `docker network ls`
4. 向系统中添加虚拟网络设备---一对*veth* (virtual ethernet)

    `ip link add veth-test1 type veth peer name veth-test2`
    \#Note:peer name之间有空格
5. 网络设备都是需要被添加到网络命名空间中的
    
    `ip link set veth-test1 netns test1`  #将虚拟网络设备veth-test1添加到名为test1的网络命名空间
6. 查看网络命名空间所具有的网络设备

    `ip netns exec test1 ip link`  #在host主机上查看网络命名空间test1的网络设备

7. 执行`ip link`命令所返回的结果

        1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000 
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00

        13: veth-test1@if12: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
        link/ether 8a:1b:94:da:6c:d6 brd ff:ff:ff:ff:ff:ff link-netns test2

    lo表示本地回环口;每行开头的数字表示网络接口设备的编号.冒号后紧跟的是网络设备的名称,如编号为1的lo网络接口和编号为13的veth-test1网络接口.

8. 为网络空间的网络设备添加ip

    `ip netns exec test1 ip addr add 192.168.1.1/24 dev veth-test1`
    
    此时,Run:
     
    `ip netns exec test1 ip a`
    
    就会看到网络空间test1的虚拟网络设备veth-test1的ip地址已经被设置为192.168.1.1,但是它的state仍然为DOWN.

    此时,只需要Up一下即可:
    
    `ip netns exec test1 ip link set dev veth-test1 up`

9. 此时,两个命名空间test1,test2就通过veth-pair连通了
> 互相ping:
> `ip netns exec test1 ping -c5 192.168.1.2` 


[1]: None "A network namespace is logically another copy of the network stack, with its own routes, firewall rules, and network devices."

