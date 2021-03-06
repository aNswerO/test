# DHCP服务：
## DHCP（Dynamic Host Configuration Protocol）：动态主机配置协议
+ 主要用途：
    + 用于内部网络和网络服务供应商给用户自动分配IP地址 

    + 用于内部网络管理员对所有主机做集中管理
+ 使用场景：
    + 解决IPv4资源不足的问题

    + 自动化安装系统
## DHCP服务简介：
+ 同网段多DHCP服务：
    + DHCP服务必须基于本地
    + 分配IP地址时遵循先到先得的原则
+ 跨网段DHCP服务：
    + 借助于dhcrelay（DHCP中继代理）
+ 相关协议：
    + ARP
    + RARP
+ DHCP服务器相关：
    + 服务器程序：/usr/sbin/dhcpd
        + 服务器默认端口号：67/udp
    + DHCP中继代理程序：/usr/sbin/dhcrelay

+ DHCP客户端相关：
    + 客户端默认端口号：68/udp
    + 客户端自动获取的IP信息存放于：/var/lib/dhclient
## DHCP报文：
+ DHCP Discover：客户端请求获取IP地址时，并不知道DHCP服务器的位置；因此客户端会在本地网络以广播方式发送请求报文（***Discover***），来发现网络中的DHCP服务器；所有收到 ***Discover*** 报文的DHCP服务器都会发送回应报文（***Offer***），这样客户端就会得知网络中存在的DHCP服务器的位置

+ DHCP Offer：DHCP服务器收到 ***Discover*** 报文后，就会在配置文件中设定的地址池中查找一个合适的IP地址，再加上相应的租约信息和其他配置信息（如网关、DNS服务器等）来构建一个 ***Offer*** 报文，发送给客户端，告知客户端本服务器可以为其提供IP地址
    >此过程为**预分配**，只是告知客户端IP地址可提供，后续还需要客户端通过**ARP**检测该IP是否重复
+ DHCP Request：客户端会收到很多 ***Offer*** 报文，所以要从这些 ***Offer*** 中选择一个（通常客户端会选择第一个发送 ***Offer*** 报文的服务器作为自己的目标服务器），并回应一个**广播**的 ***Request*** 报文，告知选择的服务器
    >客户端获取IP地址后，在地址使用租期过去1/2时，会向服务器发送**单播**的 ***Request*** 报文来延续租期；若在这之后没有收到DHCP服务器的 ***Ack*** 报文，则会在租期过去3/4时发送**广播**的 ***Request*** 报文试图延续租期
+ DHCP ACK：DHCP服务器收到 ***Request*** 报文后，会根据报文中携带的用户MAC来查找有没有对应的租约记录，若有则发送 ***ACK*** 报文作为回应，通知用户可以使用的IP地址
+ DHCP NAK：若DHCP收到 ***Reques*** 报文后，没有发现相应的租约记录或因为某些原因无法正常分配IP地址，则会发送一个 ***NAK*** 报文作为回应，告知客户端无法分配合适的IP地址
+ DHCP Release：当客户端不再需要使用分配的IP地址时，可以主动向DHCP服务器发送一个 ***Release*** 报文，告知服务器自己不再需要这个IP地址，DHCP会释放此IP地址以及被绑定的租约信息
+ DHCP Decline：客户端收到 ***ACK*** 报文后，通过**ARP**检测发现服务器分配的IP地址冲突或因其他原因导致IP地址不可用，就会发送一个 ***Decline*** 报文，通知服务器这个分配的IP地址不可用
+ DHCP Inform（此报文极少用到）：客户端如果需要从DHCP服务器获取详细的配置信息，可以发送一个 ***Inform*** 报文向服务器进行请求；服务器收到该报文后，根据租约信息进行查找，找到相应的配置信息后发送 ***ACK*** 报文回应客户端
## DHCP配置文件：
>dhcpd.conf：安装后默认没有配置，可以拷贝/usr/share/doc/dhcp-4.2.5/dhcpd.conf.example文件并重命名为/etc/dhcp/dhcpd.conf，在此文件基础上做配置
+ 一些基础配置：
```shell
    option domain-name;    #指定DNS服务器的名字
    option domain-namel;    #指定DNS服务器的IP地址
    default 600;    #默认租期，单位为s
    max-lease-time 7200;    #最大租期，单位为s
    subnet    #对哪个子网提供DHCP服务
    range    #由起始IP和终止IP组成，用来描述DHCP提供动态分配IP的范围
    option routers    #为客户端设定默认网关
```
+ 一些其他配置选项：
```shell
    filename    #指明引导文件名称
    next-server    #提供引导文件的服务器IP地址
```

# TFTP服务：
## TFTP（Trivial File Transfer Protocol）：用于传输文件的简单高级协议
>TFTP是FTP的简化版本，用来传输比FTP更易于使用但功能较少的文件
+ FTP与TFTP的区别：

||FTP|TFTP|
|-|-|-|
|安全性|具有适当的身份验证和加密协议|开放协议，没有加密机制||
|传输层协议|使用TCP作为传输层协议|使用UDP作为传输层协议|
|使用端口|使用2个端口|使用一个端口|
|RFC|基于RFC 959文档|基于RFC 1350文档|
|执行命令|执行命令很多|可执行命令只有5个（rrq、wrq、data、ack、error）|
>FTP使用2个端口：TCP端口21，是个侦听端口；TCP端口20或更高TCP端口1024以上用于源连接  
TFTP仅使用一个具有停止和等待模式的端口：69/udp

# PXE（Preboot Excution Environment）：预启动执行环境
## 介绍：
+ 基于C/S网络模式，支持远程主机通过网络下载远端服务器的映像，并由此通过网络启动操作系统

+ PXE可以引导和安装windows、Linux等操作系统
## 工作原理：
+ 客户端向PXE服务器上的DHCP发送IP地址请求信息，DHCP检测客户端网卡的MAC地址是否合法，若合法则返回IP地址，同时将启动文件**pxelinux.0**的位置信息一起发送给客户端

+ 客户端向PXE服务器上的TFTP发送获取**pxelinux.0**的请求信息，TFTP接受到信息后再向客户端发送**pxelinux.0**的大小信息，试探客户端是否满意；当TFTP收到客户端返回的同意信息后，正式向客户端发送**pxelinux.0**
+ 客户端执行接收到的**pxelinux.0**文件
+ 客户端向TFTP服务器发送针对本机的配置信息文件（位于TFTP服务器的**pxelinux.cfg**目录下），TFTP将配置文件发挥客户端，然后客户端依据配置文件内容执行后续操作
+ 客户端向TFTP发送Linux内核请求信息，TFTP接受到信息后将内核文件发送给客户端
+ 客户端向TFTP发送根文件请求信息，TFTP接收到消息后返回根文件系统
+ 客户端启动Linux内核
+ 客户端下载安装源文件，读取自动化安装脚本