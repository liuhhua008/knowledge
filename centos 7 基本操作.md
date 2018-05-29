



[TOC]



# centos 7 基本操作

## 1. 网络操作

+ ### **查看网络信息**

```shell
[root@bogon ~]# ifconfig
ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.110.101  netmask 255.255.255.0  broadcast 192.168.110.255
        inet6 fe80::b79b:f17e:571b:dce7  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:7c:3c:f9  txqueuelen 1000  (Ethernet)
        RX packets 265  bytes 149460 (145.9 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 347  bytes 32151 (31.3 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 68  bytes 5920 (5.7 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 68  bytes 5920 (5.7 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

virbr0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 192.168.122.1  netmask 255.255.255.0  broadcast 192.168.122.255
        ether 52:54:00:64:48:cc  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@bogon ~]# 

```

### 

+ ### 开启网络

  ​linux默认网络是关闭的,所以你一定要先开启。命令是临时开启，而修改配置是重启后可以开机自动开启。

```shell
root@bogon ~]# ifup ens33
[root@bogon ~]# 

```

```shell
[root@bogon ~]# cd /etc/sysconfig/network-scripts/
[root@bogon network-scripts]# ls
ifcfg-ens33       ifdown-post      ifup-eth     ifup-sit
ifcfg-lo          ifdown-ppp       ifup-ib      ifup-Team
ifcfg-有线连接_1  ifdown-routes    ifup-ippp    ifup-TeamPort
ifdown            ifdown-sit       ifup-ipv6    ifup-tunnel
ifdown-bnep       ifdown-Team      ifup-isdn    ifup-wireless
ifdown-eth        ifdown-TeamPort  ifup-plip    init.ipv6-global
ifdown-ib         ifdown-tunnel    ifup-plusb   network-functions
ifdown-ippp       ifup             ifup-post    network-functions-ipv6
ifdown-ipv6       ifup-aliases     ifup-ppp
ifdown-isdn       ifup-bnep        ifup-routes
[root@bogon network-scripts]# vim ifcfg-ens33 

```

修改IP

```shell
HWADDR=00:E0:69:01:6A:96
TYPE=Ethernet
#BOOTPROTO=dhcp
DEFROUTE=yes
PEERDNS=yes
PEERROUTES=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
IPV6_FAILURE_FATAL=no
NAME=ens33
UUID=5b0a7d76-1602-4e19-aee6-29f57618ca01
ONBOOT=yes
IPADDR0=172.8.1.211
PREFIX0=24
GATEWAY0=172.8.1.1
DNS1=172.8.1.1
```





+ ### 查看物理网卡与网卡配置文件的对应关系

,如下“ens33”是网卡，“有线连接 1”是/etc/sysconfig/network-scripts/ifcfg-有线连接_1 配置文件。现在推荐使用这个工具。CentOS7之前的网络管理是通过ifcfg文件配置管理接口(device)，而现在是通过NetworkManager服务管理连接(connection)。一个接口(device)可以有多个连接(connection)，但是同时只允许一个连接(connection)处于激活（active）状态。



```shell


//查看有哪几个网卡
[root@bogon network-scripts]# nmcli device status
DEVICE      TYPE      STATE   CONNECTION 
ens33       ethernet  连接的  有线连接 1 
virbr0      bridge    连接的  virbr0     
lo          loopback  未托管  --         
virbr0-nic  tun       未托管  -- 

//查看有哪几连接信息
[root@bogon network-scripts]# nmcli connection show
NAME        UUID                                  TYPE      DEVICE 
virbr0      a988a958-6c17-49b6-bdc3-81c5deeb155d  bridge    virbr0 
有线连接 1  b9067fb6-3b4b-4bb1-9727-676b64fb18c1  ethernet  ens33  
ens33       b26485f6-8f94-46a6-9205-bdcfe6d11688  ethernet  --     

//启动和停止接口,如果有2个的话要一个一个停止。激活的话，后面的会代替前面的。
[root@bogon network-scripts]# nmcli connection down 有线连接\ 1 
成功取消激活连接 '有线连接 1'（D-Bus 活动路径：/org/freedesktop/NetworkManager/ ActiveConnection/8）
[root@bogon network-scripts]# nmcli connection up 有线连接\ 1
连接已成功激活（D-Bus 活动路径：/org/freedesktop/NetworkManager/ActiveConnection/10）
[root@bogon network-scripts]# 



[root@bogon network-scripts]# nmcli
ens33: 连接的 to 有线连接 1
        "Intel 82545EM Gigabit Ethernet Controller (Copper) (PRO/1000 MT Single Port Adapter)"
        ethernet (e1000), 00:0C:29:7C:3C:F9, hw, mtu 1500
        ip4 default
        inet4 192.168.110.101/24
        route4 0.0.0.0/0
        route4 192.168.110.0/24
        inet6 fe80::b79b:f17e:571b:dce7/64
        route6 ff00::/8
        route6 fe80::/64
        route6 fe80::/64

virbr0: 连接的 to virbr0
        "virbr0"
        bridge, 52:54:00:64:48:CC, sw, mtu 1500
        inet4 192.168.122.1/24
        route4 192.168.122.0/24

lo: 未托管
        "lo"
        loopback (unknown), 00:00:00:00:00:00, sw, mtu 65536

virbr0-nic: 未托管
        "virbr0-nic"
        tun, 52:54:00:64:48:CC, sw, mtu 1500

DNS configuration:
        servers: 192.168.110.254
        interface: ens33

Use "nmcli device show" to get complete information about known devices and
"nmcli connection show" to get an overview on active connection profiles.


```



==注意现在LINUX已经换命令了，老命令继续保留。==

```shell
ip [选项] 操作对象{link|addr|route...}</p> <p># ip link show # 显示网络接口信息
# ip link set eth0 upi # 开启网卡
# ip link set eth0 down # 关闭网卡
# ip link set eth0 promisc on # 开启网卡的混合模式
# ip link set eth0 promisc offi # 关闭网卡的混个模式
# ip link set eth0 txqueuelen 1200 # 设置网卡队列长度
# ip link set eth0 mtu 1400 # 设置网卡最大传输单元
# ip addr show # 显示网卡IP信息
# ip addr add 192.168.0.1/24 dev eth0 # 设置eth0网卡IP地址192.168.0.1
# ip addr del 192.168.0.1/24 dev eth0 # 删除eth0网卡IP地址</p> <p># ip route list # 查看路由信息
# ip route add 192.168.4.0/24 via 192.168.0.254 dev eth0 # 设置192.168.4.0网段的网关为192.168.0.254,数据走eth0接口
# ip route add default via 192.168.0.254 dev eth0 # 设置默认网关为192.168.0.254
# ip route del 192.168.4.0/24 # 删除192.168.4.0网段的网关
# ip route del default # 删除默认路
```





## 2. SSH配置

+ **查看是否打开 22端口**

```shell
[root@bogon network-scripts]# netstat -tnl
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN     
tcp        0      0 192.168.122.1:53        0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN     
tcp6       0      0 :::111                  :::*                    LISTEN     
tcp6       0      0 :::22                   :::*                    LISTEN     
tcp6       0      0 ::1:631                 :::*                    LISTEN     
tcp6       0      0 ::1:25                  :::*                    LISTEN     
[root@bogon network-scripts]# 

```

+ **查看ssh服务是否启动**

```
[root@bogon network-scripts]# systemctl status sshd.service 
● sshd.service - OpenSSH server daemon
   Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; vendor preset: enabled)
   Active: active (running) since 六 2018-05-19 11:28:23 CST; 55min ago
     Docs: man:sshd(8)
           man:sshd_config(5)
 Main PID: 1411 (sshd)
    Tasks: 1
   CGroup: /system.slice/sshd.service
           └─1411 /usr/sbin/sshd -D

5月 19 11:28:22 bogon systemd[1]: Starting OpenSSH server daemon...
5月 19 11:28:23 bogon sshd[1411]: Server listening on 0.0.0.0 port 22.
5月 19 11:28:23 bogon sshd[1411]: Server listening on :: port 22.
5月 19 11:28:23 bogon systemd[1]: Started OpenSSH server daemon.
[root@bogon network-scripts]# 

```



## 3. yum 操作

+ **yum源操作，换成国内的源**

  ```shell
  //先备份
  [root@bogon network-scripts]# mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/ CentOS-Base.repo.bak

  //下载配置文件
  [root@bogon yum.repos.d]# wget http://mirrors.163.com/.help/CentOS7-Base-163.repo
  --2018-05-19 14:18:45--  http://mirrors.163.com/.help/CentOS7-Base-163.repo
  正在解析主机 mirrors.163.com (mirrors.163.com)... 59.111.0.251
  正在连接 mirrors.163.com (mirrors.163.com)|59.111.0.251|:80... 已连接。
  已发出 HTTP 请求，正在等待回应... 200 OK
  长度：1572 (1.5K) [application/octet-stream]
  正在保存至: “CentOS7-Base-163.repo”

  100%[==============================================================>] 1,572       --.-K/s 用时 0.002s  

  2018-05-19 14:18:45 (903 KB/s) - 已保存 “CentOS7-Base-163.repo” [1572/1572])

  //yum clean all #清除缓存
  [root@bogon yum.repos.d]# yum clean all
  已加载插件：fastestmirror, langpacks
  正在清理软件源： base extras updates
  Cleaning up everything
  Maybe you want: rm -rf /var/cache/yum, to also free up space taken by orphaned data from disabled or removed repos
  Cleaning up list of fastest mirrors

  //建立本地缓存信息
  [root@bogon yum.repos.d]# yum makecache 
  已加载插件：fastestmirror, langpacks
  Determining fastest mirrors
  base                                                                             | 3.6 kB  00:00:00     
  extras                                                                           | 3.4 kB  00:00:00     
  updates                                                                          | 3.4 kB  00:00:00     
  (1/12): base/7/x86_64/group_gz                                                   | 166 kB  00:00:03     
  (2/12): base/7/x86_64/primary_db                                                 | 5.9 MB  00:02:13     
  (3/12): extras/7/x86_64/prestodelta                                              |  47 kB  00:00:01     
  (4/12): extras/7/x86_64/primary_db                                               | 143 kB  00:00:03     
  (5/12): extras/7/x86_64/other_db                                                 |  91 kB  00:00:02     
  (6/12): updates/7/x86_64/prestodelta                                             | 179 kB  00:00:05     
  (7/12): base/7/x86_64/filelists_db                                               | 6.9 MB  00:02:32     
  (8/12): extras/7/x86_64/filelists_db                                             | 517 kB  00:00:17     
  (9/12): updates/7/x86_64/filelists_db                                            | 873 kB  00:00:20     
  (10/12): updates/7/x86_64/other_db                                               | 201 kB  00:00:04     
  (11/12): updates/7/x86_64/primary_db                                             | 1.2 MB  00:00:27     
  (12/12): base/7/x86_64/other_db                                                  | 2.5 MB  00:00:56     
  元数据缓存已建立
  ```



+ **yum搜索软件**

  ```SHELL
  [root@bogon yum.repos.d]# yum search vnc-server
  已加载插件：fastestmirror, langpacks
  Loading mirror speeds from cached hostfile
  ======================================= N/S matched: vnc-server ========================================
  tigervnc-server.x86_64 : A TigerVNC server
  tigervnc-server-applet.noarch : Java TigerVNC viewer applet for TigerVNC server
  tigervnc-server-minimal.x86_64 : A minimal installation of TigerVNC server
  tigervnc-server-module.x86_64 : TigerVNC module to Xorg

    名称和简介匹配 only，使用“search all”试试。
  [root@bogon yum.repos.d]# 
  ```

+ **yum安装软件**

  使用yum安装和卸载软件，有个前提是yum安装的软件包都是rpm格式的。

  ```shell
  [root@bogon yum.repos.d]# yum install tigervnc-server.x86_64
  已加载插件：fastestmirror, langpacks
  Loading mirror speeds from cached hostfile
  正在解决依赖关系
  --> 正在检查事务
  ---> 软件包 tigervnc-server.x86_64.0.1.8.0-5.el7 将被 安装
  --> 解决依赖关系完成

  依赖关系解决

  ========================================================================================================
   Package                       架构                 版本                       源                  大小
  ========================================================================================================
  正在安装:
   tigervnc-server               x86_64               1.8.0-5.el7                base               214 k

  事务概要
  ========================================================================================================
  安装  1 软件包

  总下载量：214 k
  安装大小：508 k
  Is this ok [y/d/N]: y
  Downloading packages:
  警告：/var/cache/yum/x86_64/7/base/packages/tigervnc-server-1.8.0-5.el7.x86_64.rpm: 头V3 RSA/SHA256 Signature, 密钥 ID f4a80eb5: NOKEY
  tigervnc-server-1.8.0-5.el7.x86_64.rpm 的公钥尚未安装
  tigervnc-server-1.8.0-5.el7.x86_64.rpm                                           | 214 kB  00:00:04     
  从 http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7 检索密钥
  导入 GPG key 0xF4A80EB5:
   用户ID     : "CentOS-7 Key (CentOS 7 Official Signing Key) <security@centos.org>"
   指纹       : 6341 ab27 53d7 8a78 a7c2 7bb1 24c6 a8a7 f4a8 0eb5
   来自       : http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7
  是否继续？[y/N]：y
  Running transaction check
  Running transaction test
  Transaction test succeeded
  Running transaction
  警告：RPM 数据库已被非 yum 程序修改。
    正在安装    : tigervnc-server-1.8.0-5.el7.x86_64                                                  1/1 
    验证中      : tigervnc-server-1.8.0-5.el7.x86_64                                                  1/1 

  已安装:
    tigervnc-server.x86_64 0:1.8.0-5.el7                                                                  

  完毕！

  [root@bogon yum.repos.d]# rpm -aq |grep vnc
  gvnc-0.7.0-3.el7.x86_64
  tigervnc-server-1.8.0-5.el7.x86_64
  tigervnc-server-minimal-1.8.0-5.el7.x86_64
  tigervnc-license-1.8.0-5.el7.noarch
  gtk-vnc2-0.7.0-3.el7.x86_64
  [root@bogon yum.repos.d]# 
  ```


​	

+ **列出查找已经安装的软件和卸载**

  ```shell
  [root@bogon ~]# yum list java-1.8*
  已安装插件:fastestmirror, langpacks
  Loading mirror speeds from cached hostfile
  已安装的软件包:
  java-1.8.0-openjdk.x86_64                      1:1.8.0.161-2.b14.el7   @anaconda
  java-1.8.0-openjdk-headless.x86_64             1:1.8.0.161-2.b14.el7   @anaconda
  可安装的软件包：
  java-1.8.0-openjdk.i686                        1:1.8.0.171-7.b10.el7   updates  
  java-1.8.0-openjdk.x86_64                      1:1.8.0.171-7.b10.el7   updates  
  java-1.8.0-openjdk-accessibility.i686          1:1.8.0.171-7.b10.el7   updates  
  java-1.8.0-openjdk-accessibility.x86_64        1:1.8.0.171-7.b10.el7   updates  
  java-1.8.0-openjdk-accessibility-debug.i686    1:1.8.0.171-7.b10.el7   updates  
  java-1.8.0-openjdk-accessibility-debug.x86_64  1:1.8.0.171-7.b10.el7   updates  

  [root@bogon ~]# yum remove java-1.7.0-openjdk-headless.x86_64

  ```

  ​

## 4.rpm操作

+ 查找相关rpm软件

```shell
# 列出所有安装的jdk
[root@bogon ~]# rpm -qa |grep jdk
copy-jdk-configs-3.3-2.el7.noarch
java-1.8.0-openjdk-headless-1.8.0.161-2.b14.el7.x86_64
java-1.8.0-openjdk-1.8.0.161-2.b14.el7.x86_64
# 列出软件包安装的文件
[root@bogon ~]# rpm -ql java-1.8.0-openjdk-1.8.0.161-2.b14.el7.x86_64
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.161-2.b14.el7.x86_64/jre/bin/policytool
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.161-2.b14.el7.x86_64/jre/lib/amd64/libawt_xawt.so
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.161-2.b14.el7.x86_64/jre/lib/amd64/libjawt.so
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.161-2.b14.el7.x86_64/jre/lib/amd64/libjsoundalsa.so
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.161-2.b14.el7.x86_64/jre/lib/amd64/ libsplashscreen.so
/usr/share/applications/java-1.8.0-openjdk-1.8.0.161-2.b14.el7.x86_64-policytool.desktop
/usr/share/icons/hicolor/16x16/apps/java-1.8.0.png
/usr/share/icons/hicolor/24x24/apps/java-1.8.0.png
/usr/share/icons/hicolor/32x32/apps/java-1.8.0.png
/usr/share/icons/hicolor/48x48/apps/java-1.8.0.png
[root@bogon ~]# 

```






## 5. 防火墙操作

+ **通过TCP端口**

  CentOS 7中防火墙是一个非常的强大的功能，在CentOS 6.5中在iptables防火墙中进行了升级了。

  这里以vnc服务5901端口为例

```shell
[root@bogon yum.repos.d]# firewall
firewall-cmd          firewall-config       firewalld             firewall-offline-cmd
[root@bogon yum.repos.d]# firewall-cmd --zone=public --add-port=5901/tcp --permanent
success
[root@bogon yum.repos.d]# firewall-cmd --reload
success
[root@bogon yum.repos.d]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens33
  sources: 
  services: ssh dhcpv6-client
  ports: 5901/tcp
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
	
[root@bogon yum.repos.d]# 
```

+ 查看firewall服务状态,以及关闭和开启

  ```shell
  [root@bogon yum.repos.d]# firewall-cmd --state
  running
  [root@bogon yum.repos.d]#  service firewalld stop 
  Redirecting to /bin/systemctl stop firewalld.service
  [root@bogon yum.repos.d]# service firewalld start 
  Redirecting to /bin/systemctl start firewalld.service
  [root@bogon yum.repos.d]# 
  ```



## 6. 系统服务管理

**1.查看当前所有系统服务的状态**

```
1 systemctl
```

**2.查看指定系统服务的状态**

```
systemctl | grep "服务名称"
```

**3.更改系统服务的状态**

```
# 启动指定服务(重启后无效)
systemctl start 服务名
# 停止指定服务(重启后无效)
systemctl stop 服务名
# 重启指定服务(重启后无效)
systemctl restart 服务名
# 开启指定服务
systemctl enable 服务名
# 关闭指定服务
systemctl disable 服务名
```

**4.查找已经运行的服务，执行文件在哪里**

```shell
[root@bogon bin]# ll /proc/3521
\u603b\u7528\u91cf 0
dr-xr-xr-x. 2 root root 0 5\u6708  19 15:47 attr
-rw-r--r--. 1 root root 0 5\u6708  19 15:47 autogroup
-r--------. 1 root root 0 5\u6708  19 15:47 auxv
-r--r--r--. 1 root root 0 5\u6708  19 15:47 cgroup
--w-------. 1 root root 0 5\u6708  19 15:47 clear_refs
-r--r--r--. 1 root root 0 5\u6708  19 15:36 cmdline
-rw-r--r--. 1 root root 0 5\u6708  19 15:47 comm
-rw-r--r--. 1 root root 0 5\u6708  19 15:47 coredump_filter
-r--r--r--. 1 root root 0 5\u6708  19 15:47 cpuset
lrwxrwxrwx. 1 root root 0 5\u6708  19 15:47 cwd -> /root
-r--------. 1 root root 0 5\u6708  19 15:47 environ
## lrwxrwxrwx. 1 root root 0 5\u6708  19 15:37 exe -> /usr/bin/Xvnc
..........................

[root@bogon bin]# 
```



**5.将已安装的服务设置为系统服务，并且设置为开机启动**

+ 建立服务文件，这有在这个目录下的文件才会被systemctl所识别。以754的权限保存在目录中：

  vim  /usr/lib/systemd/system/redis.service

  ```
  [Unit]
  Description=Redis
  After=network.target remote-fs.target nss-lookup.target
  
  [Service]
  Type=forking
  ExecStart=/usr/local/bin/redis-server /etc/redis.conf
  ExecStop=kill -INT `cat /tmp/redis.pid`
  User=www  //用户名要改啊
  Group=www
  
  [Install]
  WantedBy=multi-user.target
  ```

  ```
  我们新建一个shadowsocks.service然后编辑：
  
  [Unit]
  Description=shadowsocks
  After=this is a shadowsocks service
  
  [Service]
  Type=forking
  PIDFile=/run/shadowsocks.pid
  ExecStart=/usr/bin/ssserver -c /etc/shadowsocks.json -d start
  ExecReload=/usr/bin/ssserver -c /etc/shadowsocks.json -d restart
  ExecStop=/usr/bin/ssserver -c /etc/shadowsocks.json -d stop
  PrivateTmp=true
  
  [Install]
  WantedBy=multi-user.target
  ```

  

  文件内容解释

  ```
  [Unit]:服务的说明
  Description:描述服务
  After:描述服务类别
  
  [Service]服务运行参数的设置
  Type=forking是后台运行的形式
  ExecStart为服务的具体运行命令
  ExecReload为重启命令
  ExecStop为停止命令
  PrivateTmp=True表示给服务分配独立的临时空间
  注意：启动、重启、停止命令全部要求使用绝对路径
  
  [Install]服务安装的相关设置，可设置为多用户
  
  [Unit]部分很简单，是对服务的说明，Description用于描述服务，After用于描述服务类别，这部分其实怎么写都行。
  
  　　[Service]是要注意的地方，PIDFile为程序PID的路径，其实没定义的服务程序运行后会自动在/run目录生成一个同名的pid文件。ExecStart，ExecReload，ExecStop分别对应程序的启动、重启与停止，这个也很好理解，需要注意的是，此处应该写文件的绝对路径。比如本来可以直接运行ssserver -c /etc/shadowsocks.json -d start来启动服务，在此处要写ssserver命令的绝对路径/usr/bin/ssserver。
  
  　　PrivateTmp=True表示给服务分配独立的临时空间
  
  　　[Install]部分是服务安装的相关，可设置为多用户。
  
  　　以754的权限保存后，即可通过systemctl start/stop/restart/enable/disable shadowsocks 来启动或停止服务以及配置开机启动。
  ```

  ​

  + **设置开机自启动,和取消**

    也可以用ntsysv 打在前面打星号来设置开机启动

  ```shell
  [root@bogon ~]#  systemctl daemon-reload //先重启一下管理系统
  
  [root@bogon system]# systemctl enable vncserver
  Created symlink from /etc/systemd/system/multi-user.target.wants/vncserver@:1.sevice.service to /usr/lib/systemd/system/vncserver@.service.
  
  [root@bogon system]# 
  [root@bogon bin]# systemctl disable vncserver
  Removed symlink /etc/systemd/system/sockets.target.wants/xvnc.socket.
  ```

  ​

  + **启动、停止、查看服务**

  ```shell
  #查看已经启动的服务
  [root@bogon bin]# systemctl list-units --type=service
    UNIT                               LOAD   ACTIVE SUB     DESCRIPTION
    abrt-ccpp.service                  loaded active exited  Install ABRT coredump hook
    abrt-oops.service                  loaded active running ABRT kernel log watcher
    abrt-xorg.service                  loaded active running ABRT Xorg log watcher
    abrtd.service                      loaded active running ABRT Automated Bug 
    。。。。。。。。。。。。。。。。。。。。
  [root@bogon system]# systemctl start vncserver
  
   
  [root@bogon system]#systemctl restart vncserver
    
  ```




## 7. 环境变量

+ 环境变量加载顺序

/etc/environment -----> /etc/profile ---->~/.profile 再启动下面的shell环境

/etc/bashrc:为每一个运行bash shell ------> ~/.bashrc e

+ 当前用户shell下的总体环境变量

```shell
[root@bogon ~]# env
XDG_VTNR=1
SSH_AGENT_PID=6794
XDG_SESSION_ID=23
HOSTNAME=bogon
IMSETTINGS_INTEGRATE_DESKTOP=yes
VTE_VERSION=4602
TERM=xterm-256color
SHELL=/bin/bash
XDG_MENU_PREFIX=gnome-
HISTSIZE=1000
WINDOWID=48234502
IMSETTINGS_MODULE=IBus
USER=root
LS_COLORS=rs=0:di=38;5;27:ln=38;5;51:mh=44;38;5;15:pi=40;38;5;11:so=38;5;13:do=38;5;5:bd=48;5;232;38;5;11:cd=48;5;232;38;5;3:or=48;5;232;38;5;9:mi=05;48;5;232;38;5;15:su=48;5;196;38;5;15:sg=48;5;11;38;5;16:ca=48;5;196;38;5;226:tw=48;5;10;38;5;16:ow=48;5;10;38;5;21:st=48;5;21;38;5;15:ex=38;5;34:*.tar=38;5;9:*.tgz=38;5;9:*.arc=38;5;9:*.arj=38;5;9:*.taz=38;5;9:*.lha=38;5;9:*.lz4=38;5;9:*.lzh=38;5;9:*.lzma=38;5;9:*.tlz=38;5;9:*.txz=38;5;9:*.tzo=38;5;9:*.t7z=38;5;9:*.zip=38;5;9:*.z=38;5;9:*.Z=38;5;9:*.dz=38;5;9:*.gz=38;5;9:*.lrz=38;5;9:*.lz=38;5;9:*.lzo=38;5;9:*.xz=38;5;9:*.bz2=38;5;9:*.bz=38;5;9:*.tbz=38;5;9:*.tbz2=38;5;9:*.tz=38;5;9:*.deb=38;5;9:*.rpm=38;5;9:*.jar=38;5;9:*.war=38;5;9:*.ear=38;5;9:*.sar=38;5;9:*.rar=38;5;9:*.alz=38;5;9:*.ace=38;5;9:*.zoo=38;5;9:*.cpio=38;5;9:*.7z=38;5;9:*.rz=38;5;9:*.cab=38;5;9:*.jpg=38;5;13:*.jpeg=38;5;13:*.gif=38;5;13:*.bmp=38;5;13:*.pbm=38;5;13:*.pgm=38;5;13:*.ppm=38;5;13:*.tga=38;5;13:*.xbm=38;5;13:*.xpm=38;5;13:*.tif=38;5;13:*.tiff=38;5;13:*.png=38;5;13:*.svg=38;5;13:*.svgz=38;5;13:*.mng=38;5;13:*.pcx=38;5;13:*.mov=38;5;13:*.mpg=38;5;13:*.mpeg=38;5;13:*.m2v=38;5;13:*.mkv=38;5;13:*.webm=38;5;13:*.ogm=38;5;13:*.mp4=38;5;13:*.m4v=38;5;13:*.mp4v=38;5;13:*.vob=38;5;13:*.qt=38;5;13:*.nuv=38;5;13:*.wmv=38;5;13:*.asf=38;5;13:*.rm=38;5;13:*.rmvb=38;5;13:*.flc=38;5;13:*.avi=38;5;13:*.fli=38;5;13:*.flv=38;5;13:*.gl=38;5;13:*.dl=38;5;13:*.xcf=38;5;13:*.xwd=38;5;13:*.yuv=38;5;13:*.cgm=38;5;13:*.emf=38;5;13:*.axv=38;5;13:*.anx=38;5;13:*.ogv=38;5;13:*.ogx=38;5;13:*.aac=38;5;45:*.au=38;5;45:*.flac=38;5;45:*.mid=38;5;45:*.midi=38;5;45:*.mka=38;5;45:*.mp3=38;5;45:*.mpc=38;5;45:*.ogg=38;5;45:*.ra=38;5;45:*.wav=38;5;45:*.axa=38;5;45:*.oga=38;5;45:*.spx=38;5;45:*.xspf=38;5;45:
SSH_AUTH_SOCK=/run/user/0/keyring/ssh
SESSION_MANAGER=local/unix:@/tmp/.ICE-unix/6630,unix/unix:/tmp/.ICE-unix/6630
USERNAME=root
GNOME_SHELL_SESSION_MODE=classic
PATH=/usr/local/bin:/usr/local/sbin:/usr/bin:/usr/sbin:/bin:/sbin:/root/bin
MAIL=/var/spool/mail/root
DESKTOP_SESSION=gnome-classic
QT_IM_MODULE=xim
XDG_SESSION_TYPE=x11
PWD=/root
XMODIFIERS=@im=ibus
LANG=zh_CN.UTF-8
GDM_LANG=zh_CN.UTF-8
GDMSESSION=gnome-classic
HISTCONTROL=ignoredups
XDG_SEAT=seat0
HOME=/root
SHLVL=2
GNOME_DESKTOP_SESSION_ID=this-is-deprecated
XDG_SESSION_DESKTOP=gnome-classic
LOGNAME=root
XDG_DATA_DIRS=/root/.local/share/flatpak/exports/share/:/var/lib/flatpak/exports/share/:/usr/local/share/:/usr/share/
DBUS_SESSION_BUS_ADDRESS=unix:abstract=/tmp/dbus-pG04WHdoGO,guid=853d5a8a0e4f057bddbc68885b001f4e
LESSOPEN=||/usr/bin/lesspipe.sh %s
WINDOWPATH=1
XDG_RUNTIME_DIR=/run/user/0
DISPLAY=:0
XDG_CURRENT_DESKTOP=GNOME-Classic:GNOME
COLORTERM=truecolor
XAUTHORITY=/run/gdm/auth-for-root-KxE892/database
_=/usr/bin/env
[root@bogon ~]# 
```



+ **用于当前终端**

```
export PATH=/usr/local/jdk/bin:$PAT
```

+ **用于当前用户（推荐）** 

  用户主目录（~目录）下有.bash_profile或.bashrc隐藏文件，在其末行加入

```
export PATH= <your path 1>:<your path 2>:$PATH
source ~/.bashrc (or .profile) #让环境变量立即生效
```

+ **用于所有用户（谨慎）** 

  系统目录下有environment或profile文件，在其末行加入*

```
vi /etc/profile (or environment)
export PATH= <your path 1>:<your path 2>:$PATH
source /etc/profile (or environment) # 让环境变量立即生效
```

+ **查看环境变量**

```
echo $PATH 可以查看环境变量
```

+ **配置JAVA环境变量的案例**

  ==下面的是错的！会害死你的！导致系统启不动。不能在environment中写东西== 正确的方法是在/etc/profile的最后加上这些。

```shell
[root@bogon local]# vim /etc/environment 

JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.161-2.b14.el7.x86_64
JRE_HOME=$JAVA_HOME/jre
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib/dt.jar
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
#注意!!在environment文件中不能写export命令
export JAVA_HOME JRE_HOME CLASSPATH PATH
#让其设置马上生效
[root@bogon local]# source /etc/environment 
#验证一下
[root@bogon local]# echo $CLASSPATH
:/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.161-2.b14.el7.x86_64//lib/dt.jar: /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.161-2.b14.el7.x86_64//lib/tools.jar:/usr/lib/ jvm/java-1.8.0-openjdk-1.8.0.161-2.b14.el7.x86_64//jre/lib/dt.jar

```



## 8.上传文件

+ PUTTY  pscp上传文件   psftp用法都一样，数据传输使用ssh1。因此只要有ssh就可以用

```po
PS C:\Users\008\Desktop\学习> .\pscp.exe C:\Users\008\Desktop\新建文本文档.txt root@192.168.110.101:/root/
root@192.168.110.101's password:
```



# springboot 在centos7上部署

## 1.centos7 jar包变成服务 

+ **创建文件**

在`/etc/systemd/system`目录下新建一个文件，名称叫：`my-app.service`，文件内容如下：

```
[Unit]
Description=sourcethink web api service
After=syslog.target

[Service]
User=root
ExecStart=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.131-3.b12.el7_3.x86_64/bin/java -jar /home/centos/my-app.jar --spring.profiles.active=prod
SuccessExitStatus=143

[Install]
WantedBy=multi-user.target1234567891011
```

`Description`是该文件的一个描述。 
`ExecStart`是真实的执行命令，`java`命令我们用完全的路径来定位执行程序，我们加上`--spring.profiles.active=prod`来指定我们的运行环境对应的配置文件，也就是`application-prod.yml`配置文件。

执行`systemctl enable my-app.service`，让我们的脚本文件变成系统服务，这样系统启动后会自动执行我们的应用。

+ **自定义shell作为服务的执行文件**

		有时候想把`ExecStart=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.131-3.b12.el7_3.x86_64/bin/java -jar /home/centos/my-app.jar --spring.profiles.active=prod`这段命令放到一个shell里，然后让`ExecStart`去执行这个shell，这样做的好处是当我们改变`java -jar xxx`这段命令的时候不用更新`service`文件，只需修改我们的shell即可。

新建一个shell文件：/home/myservice.sh，内容如下：

```
#!/bin/bash -
java -jar /home/centos/my-app.jar --spring.profiles.active=prod12
```

记得把`myservice.sh`的权限修改一下，`chmod 755 myservice.sh`。

把我们的`my-app.service`中的`ExecStart`修改一下：

```
...
ExecStart=/home/myservice.sh
...123
```

改`my-app.service`之前不要忘记先关闭服务。 
`systemctl stop my-app` 
`systemctl disable my-app.service` 
改完后再启用服务 
`systemctl enable my-app.service`





##2. springboot jar包多环境部署

+ **可以给jar包启动指定tcp端口**

`java -jar xxx.jar --server.port=8888`，通过使用--server.port属性来设置xxx.jar应用的端口为8888。 在命令行运行时，连续的两个减号`--`就是对`application.properties`中的属性值进行赋值的标识。所以，`java -jar xxx.jar --server.port=8888`命令，等价于我们在`application.properties`中添加属性`server.port=8888`，该设置在样例工程中可见，读者可通过删除该值或使用命令行来设置该值来验证。 

+ **设置多个配置文件**

  以上都不是重点，这才是重点，这才是重点，这才是重点，重要的事情说3遍。我们在开发Spring Boot应用时，通常同一套程序会被应用和安装到几个不同的环境，比如：开发、测试、生产等。其中每个环境的数据库地址、服务器端口等等配置都会不同，如果在为不同环境打包时都要频繁修改配置文件的话，那必将是个非常繁琐且容易发生错误的事。

​    对于多环境的配置，各种项目构建工具或是框架的基本思路是一致的，通过配置多份不同环境的配置文件，再通过打包命令指定需要打包的内容之后进行区分打包，Spring Boot也不例外，或者说更加简单。

  在Spring Boot中多环境配置文件名需要满足application-{profile}.properties的格式，其中{profile}对应你的环境标识，比如：

 ```
application-dev.properties：开发环境
application-test.properties：测试环境
application-prod.properties：生产环境

 ```

  至于哪个具体的配置文件会被加载，需要在`application.properties `文件中通过`spring.profiles.active`

属性来设置，其值对应{profile}值。如：`spring.profiles.active=test`就会加载application-test.properties

配置文件内容下面，以不同环境配置不同的服务端口为例，进行样例实验。

针对各环境新建不同的配置文件

```
application-dev.properties
application-test.properties
application-prod.properties
```

  在这三个文件均都设置不同的server.port属性，如：dev环境设置为8080，test环境设置为9090，prod环境设置为80 `application.properties`中设置`spring.profiles.active=dev`，就是说默认以dev环境设置测试不同配置的加载：

 ` 执行java -jar xxx.jar，可以观察到服务端口被设置为8080，也就是默认的开发环境（dev）`

 `执行java -jar xxx.jar --spring.profiles.active=test，可以观察到服务端口被设置为9090，也就是测试环境的配置（test）`

`执行java -jar xxx.jar --spring.profiles.active=prod，可以观察到服务端口被设置为80，也就是生产环境的配置（prod）`

 按照上面的实验，可以如下总结多环境的配置思路：  application.properties中配置通用内容，并设置spring. profiles.active=dev，以开发环境为默认配置  application-{profile}.properties中配置各个环境不同的内容 通过命令行方式去激活不同环境的配置。



