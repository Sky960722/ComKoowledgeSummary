# Linux基本系统设置

1. 网络设置（手动设置与DHCP自动获取）

   1. /etc/hostname：修改主机名
   2. /etc/hosts：主机名映射文件
   3. /etc/sysconfig/network-scripts/ifcfg-ens33 ：修改网卡协议

   ~~~
   TYPE="Ethernet"    #网络类型（通常是Ethemet）
   PROXY_METHOD="none"
   BROWSER_ONLY="no"
   BOOTPROTO="static"   #IP的配置方法[none|static|bootp|dhcp]（引导时不使用协议|静态分配IP|BOOTP协议|DHCP协议）
   DEFROUTE="yes"
   IPV4_FAILURE_FATAL="no"
   IPV6INIT="yes"
   IPV6_AUTOCONF="yes"
   IPV6_DEFROUTE="yes"
   IPV6_FAILURE_FATAL="no"
   IPV6_ADDR_GEN_MODE="stable-privacy"
   NAME="ens33"   
   UUID="e83804c1-3257-4584-81bb-660665ac22f6"   #随机id
   DEVICE="ens33"   #接口名（设备,网卡）
   ONBOOT="yes"   #系统启动的时候网络接口是否有效（yes/no）
   #IP地址
   IPADDR=192.168.10.100  
   #网关  
   GATEWAY=192.168.10.2      
   #域名解析器
   DNS1=192.168.10.2
   ~~~

   2.Linux当前系统运行状态

   1. df,top,ps
   2. ps：将某个时间点的进程运行情况截取下来 
   3. top：动态查看进程的变化
   4. kill：进程管理

   | 代号 | 名称    | 内容                   |
   | ---- | ------- | ---------------------- |
   | 1    | SIGHUP  | 启动被终止的进程       |
   | 2    | SIGNT   | 中断一个进程的运行     |
   | 9    | SIGKILL | 强制中断一个进程的运行 |
   | 15   | SIGTERM | 以正常的方式结束该进程 |
   | 19   | SIGSTOP | 暂停一个进程的运行     |

   3. 服务（service）管理：
      1. systemctl 服务名 [start | stop | restart | reload | status ] (临时)
         1. service network restart：重启网络服务
         2. systemctl restart systemd-hostnamed：重启主机名
      2. netstat：显示网络统计信息和端口占用情况  
         1.  -anp：查看进程网络信息  
         2. -nlp：查看网络端口占用情况



