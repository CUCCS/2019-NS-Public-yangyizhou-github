---
typora-root-url: chapter1
---

# chapter1 基于 VirtualBox 的网络攻防基础环境搭建

## 实验目的

- 掌握 VirtualBox 虚拟机的安装与使用；

- 掌握 VirtualBox 的虚拟网络类型和按需配置；

- 掌握 VirtualBox 的虚拟硬盘多重加载；

## 实验环境

- VirtualBox虚拟机 6.0.12

- 攻击方主机（Attacker）:Kali

- 网关（Gateway）:Debian

- 靶机（Victim）:Xp-sp3/Kali/Debian 

- 系统版本：debian-10.1.0-amd64-netinst/kali-linux-2019.3-amd64/zh-hans_windows_xp_home_with_service_pack_3_x86_cd_x14-92408

## 实验过程

### 一、创建虚拟机

**多重加载**

在Virtualbox中新建完一台虚拟机，并且装好系统。打开管理->虚拟介质管理器，选择刚才创建的.vdi文件，右键释放后，再在下方类型中选择多重加载，即完成的虚拟硬盘的多重加载。

需要新建一台虚拟机时，选择已有虚拟硬盘，选择刚才创建的多重加载虚拟硬盘，即可直接进入系统，不需要重复安装系统。

### 二、配置网络

网络配置如下：

```
kali-1:
	ip 192.168.3.3 gateway 192.168.3.1 dns 192.168.1.1
xp-1:
	ip 192.168.3.2 gateway 192.168.3.1 dns 192.168.1.1
debian-2:
	ip 192.168.2.2 gateway 192.168.2.1  dns 192.168.1.1
xp-2:
	ip 192.168.2.3 gateway 192.168.2.1 dns 192.168.1.1
gateway: 
	eth0:ip 10.0.2.5 dns 192.168.1.1
	eth1:ip 192.168.3.1
	eth2:ip 192.168.2.1
attacker:
	ip 10.0.2.4 dns 192.168.1.1
```

网络配置完成如下：

Kali为root用户登录，可以直接修改网络配置；

Xp可以直接在网络连接-ipv4设置中直接修改；

Debian需要输入`su -`登录root用户后，修改/etc/network/interfaces配置文件，Gateway配置如下

<img src="/9.png" alt="9" style="zoom:75%;" />

网络配置完成如下

- Gateway

<img src="/gateway.png" style="zoom: 80%;" />

- Attacker

<img src="/attacker.png" alt="attacker" style="zoom:80%;" />

- Victim-KALI-1

<img src="/kali-v-1.png" alt="kali-v-1" style="zoom:80%;" />

- Victim-XP-1

<img src="/xp32-v-1.png" alt="xp32-v-1" style="zoom:80%;" />

- Victim-XP-2

<img src="/xp-v-2.png" alt="xp-v-2" style="zoom:80%;" />

- Victim-Debian-2

<img src="/debian-v-2.png" alt="debian-v-2" style="zoom:80%;" />

### 三、开启路由转发功能

**打开linux下的三层转发功能**

输入命令`vi /etc/sysctl.conf`打开配置文件，去掉"net.ipv4.ip_forward=1"的注释，保存退出。输入`sysctl -p`是配置生效。

**设置iptables规则**

输入命令`iptables -t nat -A POSTROUTING -s 192.168.2.0/24 -o enp0s3 -j MASQUERADE`、`iptables -t nat -A POSTROUTING -s 192.168.3.0/24 -o enp0s3 -j MASQUERADE`配置iptables规则

输入命令`iptables -t nat -L`查看规则

<img src="/iptables.png" alt="iptables" style="zoom:80%;" />

查看靶机，成功上网

<img src="/8.png" alt="8" style="zoom:80%;" />

**设置iptables规则开机自动加载**

输入命令`apt-get install iptables-persistent`安装iptables-persistent，安装后会在/etc目录下创建iptables目录，输入`iptables-save > /etc/iptables/rules.v4`保存规则，重启则会自动加载iptables规则。

## 连通性测试

- 靶机可以直接访问攻击者主机√

  <img src="/10.png" alt="10" style="zoom:75%;" />

- 攻击者主机无法直接访问靶机√

  <img src="/11.png" alt="11" style="zoom:75%;" />

- 网关可以直接访问攻击者主机和靶机√

  <img src="/12.png" alt="12" style="zoom:75%;" />

- 靶机的所有对外上下行流量必须经过网关√

  <img src="/13.png" alt="13" style="zoom:75%;" />

- 所有节点均可以访问互联网√

## 遇到的问题

**DebianIP配置完成后配置信息会失效。**

原因：非root用户只能临时更改ip信息配置。

解决办法：使用命令`su -`进入root用户模式，修改/etc/network/interfeaces文件配置信息。或者可使用`nmtui`命令进入文本化界面配置网络连接信息。

**Debian中开启路由转发时提示用户没有权限**

解决方法：输入`su -`命令登入root用户，再输入`adduser <username> sudo`将用户添加到sudo组中。

![1](/1.png)

**XP在出现ip冲突后无法ping通**。

解决方法：xp检测到ip冲突时会自动断线，关闭防火墙后成功ping通。

**Debian配置网卡启动失败，提示`cannot find device eth0`。**

![](/2.png)

解决方法：使用`nmcli d`命令查看网卡名称，更改配置文件，启动成功。

**Debian重启机器后，网络连接失败，提示“有线 未托管”。使用命令`nmcli d`查看显示state为未托管。**

![](/3.png)

![4](/4.png)

解决方法：修改/etc/NetworkManager/NetworkManager.conf配置文件，修改`managed=false`为`managed=true`。

**Debian网络配置完成后，连接图标显示为"?"，除了"enp0s3"外有一个连接为"Wired connection1"，我左键点击"Wired connection1"后就会断开连接，并且显示无网络可选连接，使用命令`nmcli d`查看显示state为连接中。**

<img src="/5.png" style="zoom: 67%;" />

<img src="/7.png" alt="7" style="zoom: 67%;" />

![6](/6.png)

未解决：重启网卡。不点击，就不断开。

**网关Gateway配置完成后无法上网。**

解决办法：使用`cat /etc/resolv.conf`查看dns为192.168.1.1与eth1ip地址冲突，修改eth1ip地址解决。

**iptables规则重启后失效。**

原因：直接使用`iptables`命令修改防火墙配置的时候，防火墙规则只是保存在内存中，重启后就会失效。

解决方法：尝试众多方法均无效后，最后通过**iptables-persistent**解决了。

> 从 Ubuntu 10.04 LTS (Lucid) 和 Debian 6.0 (Squeeze) 版本开始，可以通过安装一个名为 “iptables-persistent” 的包，安装后它以守护进程的方式来运行，系统重启后可以自动将保存的内容加载到iptables中。

输入命令`apt-get install iptables-persistent`安装iptables-persistent，安装后会在/etc目录下创建iptables目录，输入`iptables-save > /etc/iptables/rules.v4`保存规则，重启则会自动加载iptables规则。