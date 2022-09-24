#### 拓扑图
![拓扑](https://user-images.githubusercontent.com/74172793/192076097-d5f9ce87-0538-45e7-a620-ef51ec732436.png)

网关、攻击者、受害者相关信息：
* 网关：

![网关ip](https://user-images.githubusercontent.com/74172793/192076107-c902c8ec-474a-490c-81ba-b06e9d6736c5.png)

   172.16.222.1<br>
   08:00:27:2f:ce:65

* 攻击者：

![攻击者ip](https://user-images.githubusercontent.com/74172793/192076116-5c075cdc-bee0-4a0e-b929-2269fcdb1905.png)

 172.16.222.148<br>
 08:00:27:ec:5e:b3

* 受害者：

![受害者ip](https://user-images.githubusercontent.com/74172793/192076124-d0d460ca-8f35-4cab-8159-64d11daef8d7.png)

172.16.222.120<br>
08:00:27:29:67:99

#### 实验前安装scapy
在攻击者主机执行
```
>>sudo apt update && sudo apt install python3 python3-pip

>>pip3 install scapy[complete]
```

#### 实验一：检测局域网中的异常终端
首先检查受害者主机是否开启了混杂模式：
```
ip link show enp0s3
```

![受害者未开启混杂](https://user-images.githubusercontent.com/74172793/192076147-978b4bd9-c9ca-42cb-b47b-d9b67711dbcc.png)

结果中未出现promisc，说明没有开启；


在攻击者主机中进入scapy，并执行
```
pkt = promiscping("172.16.222.120")
```

![攻击者进入scapy](https://user-images.githubusercontent.com/74172793/192076171-c68551b6-1fdf-4750-8efd-eb4df2cce1b1.png)

回到受害者主机，执行
```
sudo ip link set enp0s3 promisc on
```

![受害者开启混杂](https://user-images.githubusercontent.com/74172793/192076192-96c386fc-0644-401f-acfe-0b3cdd1eb5f4.png)

再回到攻击者主机，

![攻击者成功](https://user-images.githubusercontent.com/74172793/192076203-4bf2666c-db05-440b-ab49-61fc3246d7e8.png)

发现攻击者得到了正确的回应，信息均与受害者信息相同；

最后回到受害者主机将混杂模式关闭。

#### 实验二：手工单步“毒化”目标主机的 ARP 缓存
在攻击者主机中执行：

![构造arp请求](https://user-images.githubusercontent.com/74172793/192076230-457ae391-a64d-41e5-82cf-fa88a5d0baf9.png)

发送该广播请求；<br>
通过获取网关的mac地址伪造ARP响应包，并将该响应包发送到受害者主机；

![网关地址+构造响应包](https://user-images.githubusercontent.com/74172793/192076241-ec26ca3a-0eb1-4763-b396-17f14cb6a31f.png)

在受害者主机上查看ARP缓存：

![查看受害者arp缓存](https://user-images.githubusercontent.com/74172793/192076257-85bcca0d-bc00-4fb0-b7cb-0489e99d2e58.png)

但是发现地址并没有改变，将sendp改为send之后再次执行上述操作：

![把sendp改为send](https://user-images.githubusercontent.com/74172793/192076263-6f9fa821-d23e-4246-81ce-360610946f0e.png)

发现地址改为了攻击者地址；

再次回到攻击者主机中执行以下操作：
```
# 恢复受害者主机的 ARP 缓存记录
## 伪装网关给受害者发送 ARP 响应
restorepkt1 = Ether()/ARP(op=2, psrc="172.16.222.1", hwsrc="08:00:27:2f:ce:65", pdst="172.16.111.111", hwdst="08:00:27:29:67:99")
sendp(restorepkt1, count=100, inter=0.2)
```

![伪造网关发送](https://user-images.githubusercontent.com/74172793/192076276-de7621cc-11b8-409d-8a8a-5aff804f3c65.png)

这时再次在受害者主机中查看ARP缓存

![最后恢复](https://user-images.githubusercontent.com/74172793/192076284-3e8f6fcf-14da-4158-8107-d3e6d6e6bfe9.png)


#### 问题与解答
* 在使用sendp发送响应包后，发现受害者主机中的相应地址并未改变，参考上一届师哥师姐的文档，发现把sendp改为send即可 
    * [第四章实验](https://github.com/CUCCS/2021-ns-public-Tbc-tang/blob/chap0x04/0x04.md)
* 在第一次进入scapy时执行pkt语句报错，这里需要用root权限，所以退出后再以root权限进入就好了。
