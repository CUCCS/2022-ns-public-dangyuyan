# 基于 VirtualBox 的网络攻防基础环境搭建

## 一.实验目的

* 掌握 VirtualBox 虚拟机的安装与使用；
* 掌握 VirtualBox 的虚拟网络类型和按需配置；
* 掌握 VirtualBox 的虚拟硬盘多重加载；

## 二.实验环境

* VirtualBox 虚拟机
* 攻击者主机（Attacker）：Kali 
* 网关（Gateway, GW）：Debian 
* 靶机（Victim）：Debian / xp-sp3 / Kali

## 三、实验要求

### 1、虚拟硬盘配置成多重加载

![多重加载](https://user-images.githubusercontent.com/74172793/188535888-5c6c7661-6c2b-46d7-bc8f-99e07ed6fa35.png)


### 2、搭建以下网络拓扑

![拓扑](https://user-images.githubusercontent.com/74172793/188537201-073d5ca6-7b5c-429b-89cd-9b150b990624.png)


### 3、 完成以下网络连通性测试

* 靶机可以直接访问攻击者主机

* 攻击者主机无法直接访问靶机

* 网关可以直接访问靶机和攻击者主机

* 靶机的所有上过流量必须经过网关

* 所有节点都可以访问互联网

## 四.实验步骤

#### （一）搭建虚拟机网络拓扑
根据拓扑图配置网络：

共需要六个虚拟机，其中kali，Debian，xp系统各两个

![虚拟机](https://user-images.githubusercontent.com/74172793/188536186-c18911a1-1a41-4a3a-9780-214b5e359e7c.png)

- **网关Debian配置**
![gw网络](https://user-images.githubusercontent.com/74172793/188536215-4b37dd1d-45ba-49ca-8a66-2e885416dd8c.png)

    * 此时的各网络地址
    ![gw网络地址](https://user-images.githubusercontent.com/74172793/188536090-603dbeaf-3754-4c52-b10c-1a259bb8c7ed.png)

- **网络1靶机-xp配置**
![xp1配置](https://user-images.githubusercontent.com/74172793/188536236-3cf00565-d3fc-4b95-baa7-3c1de5e890a1.png)

    * 该虚拟机ip地址
    ![xp1地址](https://user-images.githubusercontent.com/74172793/188536249-cd7c60be-8a6c-4bc9-9912-c9531afaba7a.png)

- **网络1靶机-kali配置**
![kali1配置](https://user-images.githubusercontent.com/74172793/188536363-9f18225e-635b-4c9e-a40b-2bed0f2323d1.png)

    * 该虚拟机ip地址
    ![kali1地址](https://user-images.githubusercontent.com/74172793/188536383-ea7a32a3-f1be-4de6-97ec-3a4d1d06a5a1.png)

- **网络2靶机-xp配置**
![xp2配置](https://user-images.githubusercontent.com/74172793/188536421-87fe2013-6a1b-471c-83c7-05c4406f7062.png)

    * 该虚拟机ip地址
   ![xp2地址](https://user-images.githubusercontent.com/74172793/188536431-47bd5646-77b8-44df-b7da-6102462ad656.png)

- **网络2靶机-Debian配置**
![dibian2配置](https://user-images.githubusercontent.com/74172793/188536450-54859fd5-d26d-4b0a-92ac-1af0fa5dc3fe.png)

    * 该虚拟机ip地址
    ![Debian2地址](https://user-images.githubusercontent.com/74172793/188536488-76d51171-1cb1-425c-a3f8-d58b0cb1877d.png)

- **网络攻击者-kali配置**
![攻击者配置](https://user-images.githubusercontent.com/74172793/188536548-bcde282d-f622-4fb3-8804-cca317ab7dda.png)

    * 该虚拟机ip地址
    ![攻击者地址](https://user-images.githubusercontent.com/74172793/188536563-30faf71f-ac86-4ce8-a1fe-d654c39b601e.png)

- **虚拟机IP地址（总）**

  | 虚拟机的安装与使用      | ip地址         |
  | --------------- | -------------- |
  | xp-victim1     | 172.16.111.100 |
  | kali-victim1   | 172.16.111.147 |
  | xp-victim2     | 172.16.222.129 |
  | debain-victim2 | 172.16.222.120 |
  | kali-attacker  | 10.0.2.4       |

#### （二）连通性测试

1. 靶机可以直接访问攻击者主机 &

2. 攻击者主机无法直接访问靶机

![攻击者靶机](https://user-images.githubusercontent.com/74172793/188536591-7ff59b9c-292a-4129-80b1-4b16c21c9c3b.png)

3. 网关可以直接访问靶机和攻击者主机

![网关访问1](https://user-images.githubusercontent.com/74172793/188536614-23128b5a-6384-4044-9c18-ed0edfa00b67.png)
![gw访问攻击者](https://user-images.githubusercontent.com/74172793/188536633-6be86d41-7c21-4c02-a311-66dd524ec83a.png)

4. 靶机的所有上过流量必须经过网关

首先在网关中安装tcpdump
```
sudo apt update && apt install tcpdump
```
然后使用命令进行抓包，并在对应网络的虚拟机中进行网络访问
```
sudo tcpdump -i [希望抓取的网卡名称] -n -w [数据保存为的文件名称]
```
![抓包](https://user-images.githubusercontent.com/74172793/188536680-0f41bda8-889a-48e2-8839-25e46707979e.png)
抓包后的文件通过scp传送到本地
![](%E4%BC%A0%E9%80%811.png)
在wireshark中进行分析，发现对应的ip数据均符合靶机和目标网址等信息，证明靶机的所有上过流量必须经过网关
![流量](https://user-images.githubusercontent.com/74172793/188536696-bb8c1c2f-5937-4aae-adef-1e21402f1c37.png)
5. 所有节点都可以访问互联网（以百度为例）
![ping1xp](https://user-images.githubusercontent.com/74172793/188536718-a162f9c7-7044-4313-9f45-d3b614ad0bf9.png)
![ping1kali](https://user-images.githubusercontent.com/74172793/188536738-c8265708-3ecc-4951-8f76-5ac89e2b1819.png)
![ping2xp](https://user-images.githubusercontent.com/74172793/188536758-a1798268-b890-4fcc-a6c6-f2c6f8bff59f.png)
![ping2db](https://user-images.githubusercontent.com/74172793/188536773-08524276-9598-493b-8ccc-8801833f9a4e.png)


## 五.问题及解决方案
1. ssh进行免密登录显示permission denied
原因：可能是文件中禁止root登录
解决方法：```sudo vi /etc/ssh/sshd_config```,在文件中添加```PermitRootLogin yes```后保存restart即可。

2. 网关ping不通xp系统的IP地址
解决方法：关闭xp系统中的防火墙后再进行ping命令。

## 六.参考资料
* [解决ssh permission denied](https://blog.csdn.net/u013007181/article/details/121109027?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ESEARCHCACHE%7ERate-1-121109027-blog-106174829.pc_relevant_vip_default&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ESEARCHCACHE%7ERate-1-121109027-blog-106174829.pc_relevant_vip_default&utm_relevant_index=2)
* [tcp命令传输文件](https://blog.csdn.net/qq_41102371/article/details/125183312)
