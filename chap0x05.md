## 实验目的
- 掌握网络扫描之端口状态探测的基本原理

## 实验要求

- 禁止探测互联网上的 IP ，严格遵守网络安全相关法律法规
- 完成以下扫描技术的编程实现
  - [x] [TCP connect scan](#tcp-connect-scan) / [TCP stealth scan](#tcp-stealth-scan)
  - [x] [TCP Xmas scan](#tcp-xmas-scan) / [TCP fin scan](#tcp-fin-scan) / [TCP null scan](#tcp-null-scan)
  - [x] [UDP scan](#udp-scan)
  - [x] 上述每种扫描技术的实现测试均需要测试端口状态为：开放、关闭 和 过滤 状态时的程序执行结果
  - [x] 提供每一次扫描测试的抓包结果并分析与课本中的扫描方法原理是否相符？如果不同，试分析原因；
  - [x] 在实验报告中详细说明实验网络环境拓扑、被测试 IP 的端口状态是如何模拟的
  - [x] 复刻 nmap 的上述扫描技术实现的命令行参数开关

## 实验过程
### 一.实验前准备
#### 1.网络拓扑

![拓扑](https://user-images.githubusercontent.com/74172793/194222602-0b6680c0-04e6-47e0-805d-89c4bedb09a3.png)

#### 2.端口状态设置
- 防火墙关闭状态：对应端口没有开启监听, 防火墙没有开启。
```
ufw disable
```
- 开启状态：对应端口开启监听，防火墙处于关闭状态。
  - apache2基于TCP, 在80端口提供服务; 
  - DNS服务基于UDP,在53端口提供服务。
```
systemctl start apache2 
systemctl start dnsmasq 
```
- 过滤状态：对应端口开启监听, 防火墙开启。
```
ufw enable && ufw deny 80/tcp
ufw enable && ufw deny 53/udp
```

### 二.过程
#### 1.TCP connect scan
> 先发送一个S，然后等待回应。如果有回应且标识为RA，说明目标端口处于关闭状态；如果有回应且标识为SA，说明目标端口处于开放状态。这时TCP connect scan会回复一个RA，在完成三次握手的同时断开连接.

**code**

- [TCP_connect_scan.py](https://github.com/CUCCS/2022-ns-public-dangyuyan/blob/chap0x05/code/TCP_connect_scan.py)

**结果**

（1）close：
```
sudo ufw disable
```
![tcp-connect-close](https://user-images.githubusercontent.com/74172793/194222659-295cd401-61cf-4a70-960d-20aa46e2e423.png)
![tcp-connect-close-wireshark](https://user-images.githubusercontent.com/74172793/194222667-23000c7d-efb9-4a2c-94ba-55dc12f8ea61.png)


- nmap复刻

![tcp-connect-close-nmap](https://user-images.githubusercontent.com/74172793/194222686-8152afe9-577c-4232-9906-40f72bfa7a15.png)
（2）filter：
```
sudo ufw enable && sudo ufw deny 8083/tcp
```
![tcp-connect-filter](https://user-images.githubusercontent.com/74172793/194222706-8bc54aaa-fa8b-4156-9772-a0295564b9a3.png)
![tcp-connect-fiter-wireshark](https://user-images.githubusercontent.com/74172793/194222723-a7649b5a-084f-45e4-8918-9789aacb399f.png)

- nmap复刻

![tcp-connect-filter-nmap](https://user-images.githubusercontent.com/74172793/194222734-5a6f1b8c-8b33-46c8-8e5e-26f9f503f058.png)

（3）open：
```
sudo ufw enable && sudo ufw allow 8083/tcp
```
![tcp-connect-open](https://user-images.githubusercontent.com/74172793/194222755-b1bc66c7-567b-4035-8afb-da2328d9f0d5.png)
![tcp-connect-open-wireshark](https://user-images.githubusercontent.com/74172793/194222769-35a0f6f5-f9df-4b01-89ca-59825173d889.png)

- nmap复刻

![tcp-connect-open-nmap](https://user-images.githubusercontent.com/74172793/194222782-87d5547b-0f57-4024-bdae-e8df1c2e7a9c.png)

#### 2.TCP stealth scan
> 先发送一个S，然后等待回应。如果有回应且标识为RA，说明目标端口处于关闭状态；如果有回应且标识为SA，说明目标端口处于开放状态。这时TCP stealth scan只回复一个R，不完成三次握手，直接取消建立连接。

**code**

- [TCP_stealth_scan.py](https://github.com/CUCCS/2022-ns-public-dangyuyan/blob/chap0x05/code/TCP_stealth_scan.py)

**结果**

（1）close：
```
sudo ufw disable
```

![tcp-stealth-close](https://user-images.githubusercontent.com/74172793/194222831-436895df-f46d-4d1a-bb1e-828e53630e2b.png)

- nmap复刻

![tcp-stealth-close-nmap](https://user-images.githubusercontent.com/74172793/194222854-fd80afa8-9208-4b14-b744-dc5170c2ccbb.png)

（2）filter：
```
sudo ufw enable && sudo ufw deny 8083/tcp
```

![tcp-stealth-filter](https://user-images.githubusercontent.com/74172793/194222887-a8c6ad55-64c6-42bc-85b9-2631d1ddb89f.png)

- nmap复刻

![tcp-stealth-filter-nmap](https://user-images.githubusercontent.com/74172793/194222905-be57a5f9-03d8-47fc-aded-db2a16153616.png)

（3）open：
```
sudo ufw enable && sudo ufw allow 8083/tcp
```
![tcp-stealth-open](https://user-images.githubusercontent.com/74172793/194222928-0f9a9ddd-0fa8-4458-ab3d-6dd0e9fddb8b.png)

- nmap复刻

![tcp-stealth-open-nmap](https://user-images.githubusercontent.com/74172793/194222955-56a388da-9076-4491-b849-8bda6c0c2da2.png)

#### 3.TCP Xmas scan
> 一种隐蔽性扫描，当处于端口处于关闭状态时，会回复一个RST包；其余所有状态都将不回复。

**code**

- [TCP_Xmas_scan.py](https://github.com/CUCCS/2022-ns-public-dangyuyan/blob/chap0x05/code/TCP_Xmas_scan.py)

**结果**

（1）close：
```
sudo ufw disable
```

![tcp-xmas-close](https://user-images.githubusercontent.com/74172793/194222982-be9f6457-cf9a-4eb9-93a5-74b1a43d8c13.png)

- nmap复刻

![tcp-xmas-close-nmap](https://user-images.githubusercontent.com/74172793/194223007-4142cde8-8b08-4021-8738-962ccee4fe2f.png)

（2）filter：
```
sudo ufw enable && sudo ufw deny 8083/tcp
```
![tcp-xmas-filter](https://user-images.githubusercontent.com/74172793/194223017-d93143d5-704a-49c7-b507-0cb0a2c98db2.png)

- nmap复刻

![tcp-xmas-filter-nmap](https://user-images.githubusercontent.com/74172793/194223040-935dc8ce-8f5f-4d5c-aff6-942bebb3876a.png)

（3）open：
```
sudo ufw enable && sudo ufw allow 8083/tcp
```
![tcp-xmas-open](https://user-images.githubusercontent.com/74172793/194223075-15af3f60-2304-46cc-8265-4762bc503a36.png)
![tcp-xmas-open-wireshark](https://user-images.githubusercontent.com/74172793/194223092-fb09ea9f-c19a-4d91-82c9-3c0691399006.png)

- nmap复刻

![tcp-xmas-open-nmap](https://user-images.githubusercontent.com/74172793/194223117-782f73ff-9284-4cb9-be06-f178cae3497f.png)

#### 4.TCP FIN scan
> 仅发送FIN包，FIN数据包能够通过只监测SYN包的包过滤器，隐蔽性较SYN扫描更⾼，此扫描与Xmas扫描也较为相似，只是发送的包未FIN包，同理，收到RST包说明端口处于关闭状态；反之说明为开启/过滤状态。

**code**

- [TCP_fin_scan.py](https://github.com/CUCCS/2022-ns-public-dangyuyan/blob/chap0x05/code/TCP_fin_scan.py)

**结果**

（1）close：
```
sudo ufw disable
```

![tcp-fin-close](https://user-images.githubusercontent.com/74172793/194223134-9aee2011-5c61-4392-aed3-c08aefa3c778.png)
![tcp-fin-close-wireshark](https://user-images.githubusercontent.com/74172793/194223155-d2f694a6-39c1-49c3-add7-a683a74389b2.png)
- nmap复刻

![tcp-fin-close-nmap](https://user-images.githubusercontent.com/74172793/194223170-f5127fda-6a06-412b-91bc-505f6e4da06c.png)

（2）filter：
```
sudo ufw enable && sudo ufw deny 8083/tcp
```
![tcp-fin-filter](https://user-images.githubusercontent.com/74172793/194223188-e02907ab-af4e-4d51-8087-207bb182ca8a.png)
![tcp-fin-filter-wireshark](https://user-images.githubusercontent.com/74172793/194223203-eeaefb09-359c-4740-9da8-b3f1a34986c2.png)

- nmap复刻

![tcp-fin-filter-nmap](https://user-images.githubusercontent.com/74172793/194223215-9e540588-417b-4adf-9f58-0c822f9f84c9.png)

（3）open：
```
sudo ufw enable && sudo ufw allow 8083/tcp
```
![tcp-fin-open](https://user-images.githubusercontent.com/74172793/194223223-e7cb1217-40e4-4c05-808e-c28f82f3638c.png)
![tcp-fin-open-wireshark](https://user-images.githubusercontent.com/74172793/194223235-b64e5e1c-cdb5-4d13-8b33-8496f473470c.png)

- nmap复刻
![tcp-fin-open-nmap](https://user-images.githubusercontent.com/74172793/194223255-096ec286-1520-464f-8273-e6bcf0c96d9e.png)

#### 5.TCP NULL scan
> 发送的包中关闭所有TCP报⽂头标记，实验结果预期还是同理：收到RST包说明端口为关闭状态，未收到包即为开启/过滤状态.

**code**

- [TCP_null_scan.py](https://github.com/CUCCS/2022-ns-public-dangyuyan/blob/chap0x05/code/TCP_null_scan.py)

**结果**

（1）close：
```
sudo ufw disable
```

![tcp-null-close](https://user-images.githubusercontent.com/74172793/194223267-e49f0979-ea28-4d18-bfb1-52b29524483a.png)
![tcp-null-close-wireshark](https://user-images.githubusercontent.com/74172793/194223286-0e646c6f-daa1-4b44-bb86-558ae74ebb92.png)

- nmap复刻

![tcp-null-close-nmap](https://user-images.githubusercontent.com/74172793/194223299-b311b397-eb43-4d50-9883-7ccbf0c947c9.png)

（2）filter：
```
sudo ufw enable && sudo ufw deny 8083/tcp
```
![tcp-null-filter-wireshark](https://user-images.githubusercontent.com/74172793/194223317-96483ec1-4c8d-47cc-a257-856505a45857.png)


- nmap复刻

![tcp-null-filter-nmap](https://user-images.githubusercontent.com/74172793/194223334-1a774f93-0867-4273-ac8f-dd68e8f848c5.png)

（3）open：
```
sudo ufw enable && sudo ufw allow 8083/tcp
```
![tcp-null-open-wireshark](https://user-images.githubusercontent.com/74172793/194223362-3e8f6f65-772f-4929-bbae-e6c6f2ee50e5.png)

- nmap复刻

![tcp-null-open-nmap](https://user-images.githubusercontent.com/74172793/194223374-bf81810d-bb12-4f6f-bf79-9e2733cad7bd.png)


#### 6.UDP scan
>一种开放式扫描，通过发送UDP包进行扫描。当收到UDP回复时，该端口为开启状态；否则即为关闭/过滤状态.

**code**

- [UDP_scan.py](https://github.com/CUCCS/2022-ns-public-dangyuyan/blob/chap0x05/code/UDP_scan.py)

**结果**

（1）close：
```
sudo ufw disable
```

![udp-close](https://user-images.githubusercontent.com/74172793/194223408-820aed3b-6d4b-4ab9-ab55-a8ced9543956.png)


- nmap复刻

![udp-close-nmap](https://user-images.githubusercontent.com/74172793/194223416-0b9188dc-2851-40c7-95ad-5180eba16869.png)

（2）filter：
```
sudo ufw enable && sudo ufw deny 8083/tcp
```
![udp-filter](https://user-images.githubusercontent.com/74172793/194223431-dbd2c17f-fca6-4495-a8dd-2a96b180ef8e.png)

- nmap复刻

![udp-filter-nmap](https://user-images.githubusercontent.com/74172793/194223448-aca52ad3-b6c1-451c-960c-4be4e7f96536.png)

（3）open：
```
sudo ufw enable && sudo ufw allow 8083/tcp
```
![udp-open](https://user-images.githubusercontent.com/74172793/194223461-adf9ecd0-d1ea-4fa8-89df-10f2a7055a21.png)


- nmap复刻

![udp-open-nmap](https://user-images.githubusercontent.com/74172793/194223472-8fb8cd07-47ab-4973-baf5-cdabc9fe60e4.png)

## 实验总结
1.扫描方式与端口状态的对应关系

  |     扫描方式/端口状态     |              开放               |      关闭       |      过滤       |
  | :-----------------------: | :-----------------------------: | :-------------: | :-------------: |
  |  TCP-connect / TCP-stealth  | 能抓到ACK&RST包 | 收到一个RST包 | 收不到TCP包 |
  | TCP-Xmas / TCP-FIN / TCP-NULL |         收不到TCP包         |  收到一个RST包  | 收不到TCP包 |
  |            UDP            |          收到UDP包          | 收不到UDP包 | 收不到UDP包 |

2. 提供每一次扫描测试的抓包结果并分析与课本中的扫描方法原理是否相符？如果不同，试分析原因。
> 相符。

## 参考
- [2021-ns-public-Lychee00](https://github.com/CUCCS/2021-ns-public-Lychee00/blob/chap0x05/chap0x05/report05.md?plain=1)
- [课件](https://github.com/c4pr1c3/cuc-ns-ppt/blob/master/chap0x05.md)
- [2020-ns-public-Crrrln](https://github.com/CUCCS/2020-ns-public-Crrrln/blob/chap0x05/chap0x05/exp5.md)
