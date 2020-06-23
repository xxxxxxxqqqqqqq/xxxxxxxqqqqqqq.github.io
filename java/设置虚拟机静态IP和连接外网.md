## CentOS 7

1. 主机设置：网络连接 ``VMware Network Adapter VMnet8`` -> ``IPV4`` 属性设置 -> 如下图

![image-20200623135744519](C:\Users\Administrator\Desktop\blog\java\设置虚拟机静态IP和连接外网.assets\image-20200623135744519.png)

``注：此处的C类IP地址，不一定非要与主机相同，即192.168.XXX.XXX均可。建议IP使用.1,网关使用.2,方便记忆`` 

2. VMware设置：``虚拟网络编辑器``

* 关闭DHCP动态IP协议
* 子网IP与主机保持在同一C类IP之下，子网掩码一致。
* ``NET设置`` 网关IP与主机一致1

![image-20200623141326706](C:\Users\Administrator\Desktop\blog\java\设置虚拟机静态IP和连接外网.assets\image-20200623141326706.png)

3. ``CentOS 7`` 设置

   *  更改 ``/etc/sysconfig/network-scripts/ifcfg-ens33`` 配置。

     ```                                               
     TYPE=Ethernet
     PROXY_METHOD=none
     BROWSER_ONLY=no
     BOOTPROTO=static  # 弃用DHCP协议，使用static静态IP
     DEFROUTE=yes
     IPV4_FAILURE_FATAL=no
     IPV6INIT=yes
     IPV6_AUTOCONF=yes
     IPV6_DEFROUTE=yes
     IPV6_FAILURE_FATAL=no
     IPV6_ADDR_GEN_MODE=stable-privacy
     NAME=ens33
     UUID=87fb6081-6b4e-453d-8667-fdb811b61e4f
     DEVICE=ens33
     ONBOOT=yes  # 系统启动时，使用当下配置的网络
     IPADDR=192.168.64.72  # 自定义静态IP，统一C类IP
     GATEWAY=192.168.64.2  # 网关IP，保持一致
     NETMASK=255.255.255.0  # 子网掩码，保持一致
     DNS1=8.8.8.8  # DNS域名解析 
     DNS2=8.8.4.4  # 尝试过单独配置这两个，均无法生效，需同时配置
     ```

   ```
   参考指令：
   chmod 777 FILENAME  # 文件没有权限
   su  # 默认切换root用户  su -USERNAME
   ll/ls -a # 查看隐藏文件，如遇到vim打开文件异常，删除使用vim指令时生成的.swp中间文件
   ```

   ```
   注：
   
   1. BOOTPROTO、ONBOOT做修改，IPADDR、GATEWAY、NETMASK做添加，其余参数都没用（网上很多都是扯犊子的，配置一堆，吊用没有）。
   
   2. 如果发现ping外网不通，但ping外网的IP是通的，则是DNS配置问题。一种解决方案如上，DNS1和DNS2；另一种方案修改/etc/resolv.conf，添加nameserver。
   ```

![image-20200623143655508](C:\Users\Administrator\Desktop\blog\java\设置虚拟机静态IP和连接外网.assets\image-20200623143655508.png)

* 重启网络服务，查看配置结果。参考指令如下：

  ```
  service network stop/start/restart/status
  systemctl stop/start/restart/status network
  ifconfig
  ip addr
  ping
  ```

## CentOS 8

 1. 主机设置：同CentOS 7。

 2. VMware设置：同CentOS 7。

 3. ``CentOS 8`` 设置：大体思路相同，不同的是使用指令修改和添加参数（网传也可以配置文件，但我没改成功）。重启网络服务的指令也不同，```CentOS 8```默认没有```network```服务。

    ```
    参考指令：nmcli  nmtui
    nmtui  # 界面化配置指令
    
    查看网卡信息
    nmcli connection # 以下所有connection可简写成c
    
    显示具体的网络接口信息
    nmcli connection show xxx
    
    显示所有活动连接
    nmcli connection show --active
    
    删除一个网卡连接
    nmcli connection delete xxx
    
    给xxx添加一个子网掩码（NETMASK）
    nmcli connection modify xxx ipv4.addresses 192.168.0.58/24
    
    IP获取方式设置成手动（BOOTPROTO=static/none）
    nmcli connection modify xxx ipv4.method manual
    
    添加一个网关（GATEWAY）
    nmcli connection modify xxx ipv4.gateway 192.168.64.2
    
    给xxx修改IP（IPADDR）
    nmcli connection modify xxx ipv4.addresses 192.168.0.58
    
    添加一个ipv4
    nmcli connection modify xxx +ipv4.addresses 192.168.0.59/24
    
    删除一个ipv4
    nmcli connection modify xxx -ipv4.addresses 192.168.0.59/24
    
    添加DNS
    nmcli connection modify xxx ipv4.dns 114.114.114.114
    
    删除DNS
    nmcli connection modify xxx -ipv4.dns 114.114.114.114
    
    可一块写入：
    nmcli connection modify xxx ipv4.dns 114.114.114.114 ipv4.gateway 192.168.0.2
    
    使用nmcli重新回载网络配置
    nmcli c reload
    
    如果之前没有xxx的connection，则上一步reload后就已经自动生效了，启动网络配置
    nmcli c up xxx
    
    ```