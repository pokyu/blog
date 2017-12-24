
## CentOS 7搭建VPN服务器

1. 先看看你的主机是否支持pptp，返回结果为yes就表示通过。
```
modprobe ppp-compress-18 && echo yes
```
运行结果：
```
[root@vultr ~]# modprobe ppp-compress-18 && echo yes
yes
```
2. 是否开启了TUN，有的虚拟机主机需要开启，返回结果为cat: /dev/net/tun: File descriptor in bad state。就表示通过。
```
cat /dev/net/tun
```
运行结果：
```
[root@vultr ~]# cat /dev/net/tun
cat: /dev/net/tun: File descriptor in bad state
```
3. 安装ppp , pptpd 和 iptables。
a.先更新一下再安装。
```
yum install update
```
运行结果：
```
[root@vultr ~]# cat /dev/net/tun
cat: /dev/net/tun: File descriptor in bad state
[root@vultr ~]# yum install update
Loaded plugins: fastestmirror
base                                                        | 3.6 kB  00:00:00     
epel/x86_64/metalink                                        |  16 kB  00:00:00     
epel                                                        | 4.7 kB  00:00:00     
extras                                                      | 3.4 kB  00:00:00     
updates                                                     | 3.4 kB  00:00:00     
base/7/x86_64/group_gz         FAILED                                          
ftp://ftp.ussg.iu.edu/linux/centos/7.4.1708/os/x86_64/repodata/9346184be1deb727caf4b1ecf4a7949155da5da74af9b92c172687b290a773df-c7-x86_64-comps.xml.gz: [Errno 12] Timeout on ftp://ftp.ussg.iu.edu/linux/centos/7.4.1708/os/x86_64/repodata/9346184be1deb727caf4b1ecf4a7949155da5da74af9b92c172687b290a773df-c7-x86_64-comps.xml.gz: (28, '')
Trying other mirror.
epel/x86_64/updateinfo         FAILED                                          ETA 
http://mirror.cs.princeton.edu/pub/mirrors/fedora-epel/7/x86_64/repodata/91c2ba027bbd04474bb7a5f22bac6c53f1471510dd09476c281842f9a6042d6a-updateinfo.xml.bz2: [Errno 14] HTTP Error 404 - Not Found
Trying other mirror.
To address this issue please refer to the below knowledge base article 
https://access.redhat.com/articles/1320623
If above article doesn't help to resolve this issue please create a bug on https://bugs.centos.org/
(1/7): epel/x86_64/group_gz                                 | 266 kB  00:00:01     
(2/7): extras/7/x86_64/primary_db                           | 145 kB  00:00:01     
(3/7): base/7/x86_64/group_gz                               | 156 kB  00:00:00     
(4/7): epel/x86_64/updateinfo                               | 855 kB  00:00:00     
(5/7): epel/x86_64/primary_db                               | 6.1 MB  00:00:03     
(6/7): updates/7/x86_64/primary_db                          | 4.5 MB  00:00:01     
(7/7): base/7/x86_64/primary_db                             | 5.7 MB  00:00:05     
Determining fastest mirrors
 *base: mirror.mojohost.com
 *epel: reflector.westga.edu
 *extras: mirror.jax.hugeserver.com
 *updates: mirror.teklinks.com
No package update available.
Error: Nothing to do
```
b.安装ppp和pptpd
```
yum -y install ppp pptpd
```
运行结果：
```
... ...
Installed:
  pptpd.x86_64 0:1.4.0-2.el7                                                                                                                                                                                    
Dependency Installed:
  perl.x86_64 4:5.16.3-292.el7           perl-Carp.noarch 0:1.26-244.el7         perl-Encode.x86_64 0:2.51-7.el7          perl-Exporter.noarch 0:5.68-3.el7     perl-File-Path.noarch 0:2.09-2.el7          
  perl-File-Temp.noarch 0:0.23.01-3.el7  perl-Filter.x86_64 0:1.49-3.el7         perl-Getopt-Long.noarch 0:2.40-2.el7     perl-HTTP-Tiny.noarch 0:0.033-3.el7   perl-PathTools.x86_64 0:3.40-5.el7          
  perl-Pod-Escapes.noarch 1:1.04-292.el7 perl-Pod-Perldoc.noarch 0:3.20-4.el7    perl-Pod-Simple.noarch 1:3.28-4.el7      perl-Pod-Usage.noarch 0:1.63-3.el7    perl-Scalar-List-Utils.x86_64 0:1.27-248.el7
  perl-Socket.x86_64 0:2.010-4.el7       perl-Storable.x86_64 0:2.45-3.el7       perl-Text-ParseWords.noarch 0:3.29-4.el7 perl-Time-HiRes.x86_64 4:1.9725-3.el7 perl-Time-Local.noarch 0:1.2300-2.el7       
  perl-constant.noarch 0:1.27-2.el7      perl-libs.x86_64 4:5.16.3-292.el7       perl-macros.x86_64 4:5.16.3-292.el7      perl-parent.noarch 1:0.225-244.el7    perl-podlators.noarch 0:2.5.1-3.el7         
  perl-threads.x86_64 0:1.87-4.el7       perl-threads-shared.x86_64 0:1.43-6.el7
Complete!
```
c.安装iptables。
这是自带的，如果没有的话就安装。
```
yum install iptables
```
4. 配置pptpd.conf。
```
vi /etc/pptpd.conf    #找到localip，去掉开始的那个#符号
localip 192.168.0.1
remoteip 192.168.0.234-238,192.168.0.245
#这些是默认的，一般不需要去修改，分配给客户端的ip就是234到238之间，你也可以往大了写，看你的客户端有多少。
```
5. 配置options.pptpd
```
vi /etc/ppp/options.pptpd      #在末尾添加dns
ms-dns  8.8.8.8      #这是谷歌的，你也可以改成阿里巴巴的或者其它
ms-dns  8.8.4.4
```
6. 配置连接VPN客户端要用到的帐号密码。
```
vi /etc/ppp/chap-secrets    #格式很通俗易懂。
#  client为帐号，server是pptpd服务，secret是密码，*表示是分配任意的ip
# Secrets for authentication using CHAP
# client        server    secret                  IP addresses
  count        pptpd    771297972            *
```
7. 配置sysctl.conf
```
vi /etc/sysctl.conf
#添加一行    net.ipv4.ip_forward = 1    到末尾即可，然后保存
sysctl -p    #运行这个命令会输出上面添加的那一行信息，意思是使内核修改生效
```
8. 这个时候把iptables关闭的话是可以连接VPN了，之所以要把iptables关闭是因为没有开放VPN的端口，客户如果直接连接的话是不允许的。这里还需要设置iptables的转发规则，让你的客户端连接上之后能访问外网。
```
iptables -t nat -F
iptables -t nat -A POSTROUTING -s 192.168.0.234/24 -j SNAT --to 你的服务器的公网IP
#192.168.0.234/24是你分配给客户的ip，后面的那个是你的服务器的公网IP
```
9. 这个时候还是连接不上的，因为iptables把客户的VPN连接拦截了，不允许连接，所以得开放相应的端口。
具体是哪个端口可以自己查下，我的是默认的，如果你没有更改过的话也会是默认的。
```
iptables -I INPUT -p tcp --dport 1723 -j ACCEPT
iptables -I INPUT -p tcp --dport 47 -j ACCEPT
iptables -I INPUT -p gre -j ACCEPT
```
10. 保险起见，到了这里应该重启一下pptpd服务和iptables服务生效。
```
systemctl restart iptables
systemctl restart pptpd
```
11. VPN服务器搭建完成。