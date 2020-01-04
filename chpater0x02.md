---

---

# chapter0x02-基于 Scapy 编写端口扫描器

## 实验目的

- 掌握网络扫描之端口状态探测的基本原理

## 实验环境

- python + [scapy](https://scapy.net/)

## 实验要求

- 禁止探测互联网上的 IP ，严格遵守网络安全相关法律法规
- 完成以下扫描技术的编程实现
  - TCP connect scan / TCP stealth scan
  - TCP Xmas scan / TCP fin scan / TCP null scan
  - UDP scan
- 上述每种扫描技术的实现测试均需要测试端口状态为：开放、关闭 和 过滤 状态时的程序执行结果
- 提供每一次扫描测试的抓包结果并分析与课本中的扫描方法原理是否相符？如果不同，试分析原因；
- 在实验报告中详细说明实验网络环境拓扑、被测试 IP 的端口状态是如何模拟的
- （可选）复刻 nmap 的上述扫描技术实现的命令行参数开关

## 实验过程

### 一、网络拓扑结构

将攻击者主机网卡设置为和靶机相同的内部网络internet1

- 攻击者主机

![](chapter0x02/attacker-net.png)

- 靶机

![victim-net](chapter0x02/victim-net.png)

- 拓扑结构

![topology](chapter0x02/topology.png)

- 网卡信息

攻击者主机 192.168.3.10

![attacker-ip](chapter0x02/attacker-ip.png)

靶机 192.168.3.3

![victim-ip](chapter0x02/victim-ip.png)

- 连通性测试

![connectivity-test](chapter0x02/connectivity-test.png)

### 二、扫描技术的编程实现

- 详细代码见[chapter0x02-code](https://github.com/CUCCS/2019-NS-Public-yangyizhou-github/tree/chap0x02chapter0x02-code)

### 三、扫描测试

- TCP Connect Scan

  - 端口关闭状态测试

    - 输入`netstat -anp`查看端口，靶机80端口关闭

      ![victim-port](chapter0x02/victim-port.png)

    - 靶机输入`tcpdump -i eth0 -w connect-close.pcap`监听eth0网卡开始抓包

      ![start-listen](chapter0x02/start-listen.png)

    - 攻击者主机运行扫描文件TCP_connect_scan.py，靶机停止抓包；在虚拟机Attacker界面观察到端口扫描的结果是靶机的80端口为关闭状态

      ![attacker-scan](chapter0x02/attacker-scan.png)

    - 分析靶机抓取到的数据包

      - 攻击者主机给靶机的80端口发送了设置SYN标志的TCP包

      - 靶机发送给攻击者主机的返回包中设置了RST标志

        ![victim-wireshark](chapter0x02/victim-wireshark.png)

        ![connect-close](chapter0x02/connect-close.png)

        - 符合课本中的扫描方法原理，实验成功

  - 端口开启状态测试

    - 输入`nc -lp 80 &`开启80端口，输入`netstat -anp | grep :80`查看端口80成功开启

      ![open-port80](chapter0x02/open-port80.png)

    - 靶机输入`tcpdump -i eth0 -w connect -open.pcap`监听eth0网卡开始抓包
    
      ![start-listen_1](chapter0x02/start-listen_1.png)
    
    - 攻击者主机运行扫描文件TCP_connect_scan.py，靶机停止抓包
    
      ![scan_1](chapter0x02/scan_1.png)
    
    - 分析靶机中抓到的包
    
      - 攻击者和靶机之间进行了完整的3次握手TCP通信（SYN, SYN/ACK, 和RST），并且该连接由靶机在最终握手中发送确认ACK+RST标志来建立，表示端口打开，符合实验原理，实验成功
    
      ![wireshark_1](chapter0x02/wireshark_1.png)
    
      ![connect-open](chapter0x02/connect-open.png)
  
  - 端口过滤状态的测试
  
    - 在靶机中设置80端口被防火墙过滤
  
      ![set_2](chapter0x02/set_2.png)
  
    - 在靶机中设置监听eht0网卡开始抓包
  
      ![listen_2](chapter0x02/listen_2.png)
  
    - 在攻击者主机中运行扫描文件TCP_connect_scan.py，结果显示靶机80端口为过滤状态；靶机停止抓包
  
      ![scan_2](chapter0x02/scan_2.png)
  
    - 分析靶机抓到的包
  
      - 攻击者主机给靶机的80端口发送了设置SYN标志的TCP包
  
      - 靶机未返回TCP数据包给攻击者
  
      - 靶机返回给攻击者一个ICMP数据包，且该包类型为type3
  
        ![wireshark_2](chapter0x02/wireshark_2.png)
  
        符合实验原理，实验成功

- TCP Stealth Scan

  - 端口关闭状态的测试

    - 关闭防火墙

      ![set_3](chapter0x02/set_3.png)

    - 查看靶机端口开启情况，靶机80端口为关闭状态

      ![set_3_1](chapter0x02/set_3_1.png)

    - 靶机中设置监听eth0网卡开始抓包

      ![listen_3](chapter0x02/listen_3.png)

    - 在攻击者主机运行扫描文件TCP_stealth_scan.py，显示端口为关闭状态，靶机停止抓包

      ![scan_3](chapter0x02/scan_3.png)

    - 分析靶机抓到的包

      - 攻击者主机给靶机的80端口发送了设置SYN标志的TCP包

      - 靶机发送给攻击者主机的TCP返回包中设置了RST标志

        ![wireshark_3](chapter0x02/wireshark_3.png)

        ![stealth-close](chapter0x02/stealth-close.png)

        符合实验原理，实验成功

  - 端口开启状态的测试

    - 打开靶机的80端口监听

      ![set_4](chapter0x02/set_4.png)

    - 在靶机设置监听eth0网卡开始抓包

      ![listen_4](chapter0x02/listen_4.png)

    - 在攻击者主机运行扫描文件TCP_stealth_scan.py，结果显示端口开启，靶机停止抓包

      ![scan_4](chapter0x02/scan_4.png)

    - 分析靶机抓到的包

      - 前三个包类似于TCP连接扫描，完成了SYN, SYN/ACK, 和RST

      - 最后在TCP数据包中发送的是RST标志而不是RST + ACK

        ![wireshark_4](chapter0x02/wireshark_4.png)

        ![stealth-open](chapter0x02/stealth-open.png)

        符合实验原理，实验成功

  - 端口过滤状态的测试

    - 在靶机设置80端口被防火墙过滤

      ![set_5](chapter0x02/set_5.png)

    - 在靶机设置监听eth0网卡开始抓包

      ![listen_5](chapter0x02/listen_5.png)

    - 在攻击者主机运行扫描文件TCP_stealth_scan.py，结果显示端口为过滤，靶机停止抓包

      ![scan_5](chapter0x02/scan_5.png)

    - 分析靶机抓到的包

      - 攻击者主机给靶机的80端口发送了设置SYN标志的TCP包

      - 靶机没有返回TCP数据包给攻击者主机

      - 靶机返回给攻击者主机一个ICMP数据包，且该包类型为type3

        ![wireshark_5](chapter0x02/wireshark_5.png)

- TCP XMAS Scan

  - 端口关闭状态的测试

    - ![set_6](chapter0x02/set_6.png)

    - 查看靶机端口开启情况，靶机80端口关闭

      ![set_6_1](chapter0x02/set_6_1.png)

    - 在靶机设置监听eth0网卡开始抓包

      ![listen_6](chapter0x02/listen_6.png)

    - 在攻击者主机运行扫描文件TCP_xmas_scan.py，显示端口为关闭状态，靶机停止抓包

      ![scan_6](chapter0x02/scan_6.png)

    - 分析靶机抓到的包

      - 攻击者主机给靶机的80端口发送了设置了PSH，FIN和URG标志的TCP数据包

      - 靶机发送给攻击者主机的TCP返回包中设置了RST标志

        ![wireshark_6](chapter0x02/wireshark_6.png)

        ![xmas-close](chapter0x02/xmas-close.png)

        符合实验原理，实验成功

  - 端口开启状态的测试

    - 打开靶机的80端口监听

      ![set_7](chapter0x02/set_7.png)

    - 在靶机设置监听eth0网卡开始抓包

      ![listen_7](chapter0x02/listen_7.png)

    - 在攻击者主机运行扫描文件TCP_xmas_scan.py，扫描的结果是：靶机victim的80端口为打开或被过滤状态，靶机停止抓包

      ![scan_7](chapter0x02/scan_7.png)

    - 分析靶机抓到的包

      - 攻击者主机给靶机的80端口发送了设置了PSH，FIN和URG标志的TCP数据包

      - 靶机没有发送TCP包响应，无法区分其80端口打开/被过滤

      - 靶机也没有发送ICMP数据包给攻击者主机，说明端口不是被过滤状态

        ![wireshark_7](chapter0x02/wireshark_7.png)

        ![xmas-open](chapter0x02/xmas-open.png)

        符合实验原理，实验成功

  - 端口过滤状态的测试

    - 在victim靶机设置80端口被防火墙过滤

      ![set_8](chapter0x02/set_8.png)

    - 在靶机设置监听eth0网卡开始抓包

      ![listen_8](chapter0x02/listen_8.png)

    - 在攻击者虚拟机运行扫描文件TCP_xmas_scan.py，扫描的结果是：靶机victim的80端口为过滤状态，靶机停止抓包

      ![scan_8](chapter0x02/scan_8.png)

    - 分析靶机抓到的包

      - 攻击者主机给靶机的80端口发送了设置了PSH，FIN和URG标志的TCP数据包

      - 靶机返回给靶机一个ICMP数据包，且该包类型为type3

        ![wireshark_8](chapter0x02/wireshark_8.png)

        ![xmas-filter](chapter0x02/xmas-filter.png)

        符合实验原理，实验成功

- TCP FIN Scan

  - 端口关闭状态的测试

    - 关闭防火墙

      ![set_9](chapter0x02/set_9.png)

    - 查看靶机端口开启情况，80端口为关闭状态

      ![set_9_1](chapter0x02/set_9_1.png)

    - ![listen_9](chapter0x02/listen_9.png)

    - 在攻击者主机运行扫描文件TCP_fin_scan.py，扫描的结果是：靶机victim的80端口为关闭状态，靶机停止抓包

      ![scan_9](chapter0x02/scan_9.png)

    - 分析靶机抓到的包

      - 攻击者主机给靶机的80端口发送了设置了FIN标志的TCP数据包

      - 靶机发送给攻击者主机的TCP返回包中设置了RST标志

        ![wireshark_9](chapter0x02/wireshark_9.png)

        ![fin-close](chapter0x02/fin-close.png)

        符合实验原理，实验成功

  - 端口开启状态的测试

    - 打开靶机的80端口监听

      ![set_10](chapter0x02/set_10.png)

    - 在靶机设置监听eth0网卡开始抓包

      ![listen_10](chapter0x02/listen_10.png)

    - 在攻击者主机运行扫描文件TCP_fin_scan.py，扫描的结果是：靶机victim的80端口为打开或被过滤状态，靶机停止抓包

      ![scan_10](chapter0x02/scan_10.png)

    - 分析靶机抓到的包

      - 攻击者主机给靶机的80端口发送了设置了FIN标志的TCP数据包

      - 靶机没有发送TCP包响应，无法区分其80端口打开/被过滤

      - 靶机也没有发送ICMP数据包给攻击者主机，说明端口不是被过滤状态

        ![wireshark_10](chapter0x02/wireshark_10.png)

        ![fin-open](chapter0x02/fin-open.png)

        符合实验原理，实验成功

  - 端口过滤状态的测试

    - 在靶机设置80端口被防火墙过滤

      ![set_11](chapter0x02/set_11.png)

    - 在靶机设置监听eth0网卡开始抓包

      ![listen_11](chapter0x02/listen_11.png)

    - 在攻击者主机机运行扫描文件TCP_fin_scan.py，扫描的结果是：靶机victim的80端口为过滤状态，靶机停止抓包

      ![scan_11](chapter0x02/scan_11.png)

    - 分析靶机抓到的包

      - 攻击者主机给靶机的80端口发送了设置了FIN标志的TCP数据包

      - 靶机返回给靶机一个ICMP数据包，且该包类型为type3

        ![wireshark_11](chapter0x02/wireshark_11.png)

        ![fin-filter](chapter0x02/fin-filter.png)

        符合实验原理，实验成功

- TCP NULL Scan

  - 端口关闭状态的测试

    - 关闭防火墙

      ![set_12](chapter0x02/set_12.png)

    - 查看靶机80端口，端口未打开

      ![set_12_1](chapter0x02/set_12_1.png)

    - 在靶机设置监听eth0网卡开始抓包

      ![listen_12](chapter0x02/listen_12.png)

    - 在攻击者主机运行扫描文件TCP_null_scan.py，扫描的结果显示靶机victim的80端口为关闭状态，靶机停止抓包

      ![scan_12](chapter0x02/scan_12.png)

    - 扫描的结果是：靶机victim的80端口为关闭状态

      - 攻击者主机给靶机的80端口发送了一个没有任何标志位的TCP包

      - 靶机发送给攻击者主机的TCP返回包中设置了RST标志

        ![wireshark_12](chapter0x02/wireshark_12.png)

        ![null-close](chapter0x02/null-close.png)

  - 端口开启状态的测试

    - 打开靶机的80端口监听

      ![set_13](chapter0x02/set_13.png)

    - 在靶机设置监听eth0网卡开始抓包

      ![listen_13](chapter0x02/listen_13.png)

    - 在攻击者主机运行扫描文件TCP_null_scan.py，显示靶机victim的80端口为打开或被过滤状态，靶机停止抓包

      ![scan_13](chapter0x02/scan_13.png)

    - 分析靶机抓到的包

      - 攻击者主机给靶机的80端口发送了一个没有任何标志位的TCP包

      - 靶机没有发送TCP包响应，无法区分其80端口打开/被过滤

      - 靶机也没有发送ICMP数据包给攻击者主机，说明端口不是被过滤状态

        ![wireshark_13](chapter0x02/wireshark_13.png)

        ![null-open](chapter0x02/null-open.png)

  - 端口过滤状态的测试

    - 在靶机设置80端口被防火墙过滤

      ![set_14](chapter0x02/set_14.png)

    - 在靶机设置监听eth0网卡开始抓包

      ![listen_14](chapter0x02/listen_14.png)

    - 在攻击者主机运行扫描文件TCP_null_scan.py，结果显示靶机victim的80端口为过滤状态，靶机停止抓包

      ![scan_14](chapter0x02/scan_14.png)

    - 分析靶机抓到的包

      - 攻击者主机给靶机的80端口发送了一个没有任何标志位的TCP包

      - 靶机返回给靶机一个ICMP数据包，且该包类型为type3

        ![wireshark_14](chapter0x02/wireshark_14.png)

        ![null-filter](chapter0x02/null-filter.png)

- UDP Scan

  - 端口关闭状态的测试

    - 关闭防火墙

      ![set_15](chapter0x02/set_15.png)

    - 查看80端口未被打开

      ![set_15_1](chapter0x02/set_15_1.png)

    - 在靶机设置监听eth0网卡开始抓包

      ![listen_15](chapter0x02/listen_15.png)

    - 在攻击者主机运行扫描文件UDP_scan.py，显示80端口关闭，靶机停止抓包

      ![scan_15](chapter0x02/scan_15.png)

    - 分析靶机抓到的包

      - 攻击者主机给靶机的80端口发送了UDP包

      - 靶机响应ICMP端口不可达错误type3和code3

        ![wireshark_15](chapter0x02/wireshark_15.png)

        ![udp-open](chapter0x02/udp-open.png)

  - 端口开启状态的测试

    - 打开靶机的80端口监听

      ![set_16](chapter0x02/set_16.png)

    - 在靶机设置监听eth0网卡开始抓包

      ![listen_16](chapter0x02/listen_16.png)

    - 在攻击者主机运行扫描文件UDP_scan.py，显示80端口开启，靶机停止抓包

      ![scan_16](chapter0x02/scan_16.png)

    - 分析靶机抓到的包

      - 攻击者主机给靶机的80端口发送了UDP包

      - 靶机响应ipv4包

        ![wireshark_16](chapter0x02/wireshark_16.png)

        ![udp-close](chapter0x02/udp-close.png)

  - 端口过滤状态的测试（同端口关闭状态测试）

## 参考资料

- [第五章 网络扫描](https://c4pr1c3.github.io/cuc-ns/chap0x05/main.html)
- [iptables详解及一些常用规则](https://www.jianshu.com/p/ee4ee15d3658)
- [编程实现并讲解TCP connect scan/TCP stealth scan/TCP XMAS scan/UDP scan](https://blog.csdn.net/jackcily/article/details/83117884)



