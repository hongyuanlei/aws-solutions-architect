#### IPV4地址分类

**传统的IP地址分类**将IP地址分为了A类,B类,C类，D类和E类。其中A类，B类和C类是正常使用的，而D类用来做组播，E类是保留使用的。

```
     ---------      -----      -----      -----
A类 | 0******* | . | Host | . | Host | . | Host |
     ---------      -----      -----      -----
0 ~ 126

127 (01111111) 是一个A类地址，是给loopback testing保留的，不能在network中使用。

     ---------      --------      -----      -----
B类 | 10****** | . | NetWork | . | Host | . | Host |
     ---------      --------      -----      -----
128 ~ 191

     ---------      --------      --------      -----
C类 | 110***** | . | NetWork | . | NetWork | . | Host |
     ---------      --------      --------      -----
192 ~ 223

```

#### 私有IP地址
一般来说内网都用private ip address，而外网都用public ip address。

Class   | Private Address Range
------- | ----------------------------
A       | 10.0.0.0    ~ 10.255.255.255
B       | 172.16.0.0  ~ 172.31.255.255
C       | 192.168.0.0 ~ 192.168.255.255

#### VPC

Amazon Virtual Private Cloud (Amazon VPC) 是AWS云中的自定义虚拟**私有**网络, 它在逻辑上与 AWS 云中的其他虚拟网络隔绝。VPC与你在数据中心中运行的传统网络极其相似，你可以在你定义的VPC内启动 AWS 资源。

当创建一个VPC时，我们需要指定它的VPC CIDR。我们创建的是私有网络，所以我们VPC CIDR可以从私有地址里任意选择。但是如果我们是在设计一个大的组织的网络，那么就需要按需分配你的VPC CIDR，这是因为在大型组织里，我们经常会遇到两个VPC需要不经过共有网络实现互联，这需要两个VPC CIDR没有冲突。

假设，我们现在有一个A类子网地址：`10.0.0.0/16`，这个地址前16位是**网络部分**，后16位是**主机部分**。如果我们不继续对这个子网进行子网划分的话那么这个网络可以容纳2<sup>16</sup>-2（65534）台主机。为了**便于网络管理**、**减少IP地址浪费**和**减少网络广播风暴**，我们需要继续进行子网划分。

让我们进入真实的项目情景中。通常情况下一个项目会用3个部署环境：

- Dev（开发环境）
- Staging（类生产环境）
- Production（生产环境）

如果我们的服务需要支持Production环境Region级别的High Availability(高可用性)，比如需要同时部署在Tokyo Region和Singapore Region以提供高可用性，那么Production环境就包含了Tokyo和Singapore两Region。为了实现环境之间的隔离，我们每个独立的环境都会创建独立的VPC，对这个项目来说，我们需要创建4个VPC:

- Dev Singapore（开发环境）
- Staging Singapore（类生产环境）
- Production Tokyo（生产环境）
- Production Singapore（生产环境）

#### VPC Subnet

VPC Subnet就是对VPC网络子网划分后的子网，我们可以在指定的VPC Subnet内启动AWS 资源。**每个子网必须完全位于一个可用区内，并且不能跨越区域**。

通常，在一个VPC中我们会创建3种类型的VPC Subnet：

- **Public Subnet**
- **NAT Subnet**
- **Private Subnet**

为了提供High Availability(高可用性)，通常我们会把服务部署在多个Availability Zones(可用区)上。

比如，我们现在需要将服务部署在Singapore Region的可用区`ap-southeast-1a`和`ap-southeast-1b`，那么我们的Subnet会是：

- Public Subnet A
- Public Subnet B
- NAT Subnet A
- NAT Subnet B
- Private Subnet A
- Private Subnet B

而将服务部署在Tokyo Region的可用区`ap-northeast-1a`和`ap-northeast-1c`，那么我们的Subnet会是：

- Public Subnet A
- Public Subnet C
- NAT Subnet A
- NAT Subnet C
- Private Subnet A
- Private Subnet C

现在我们来列一下我们一共要创建多少Subent：

Dev Singapore VPC:
- Public Subnet A
- Public Subnet B
- NAT Subnet A
- NAT Subnet B
- Private Subnet A
- Private Subnet B

Staging Singapore VPC:
- Public Subnet A
- Public Subnet B
- NAT Subnet A
- NAT Subnet B
- Private Subnet A
- Private Subnet B

Production Singapore VPC:

- Public Subnet A
- Public Subnet B
- NAT Subnet A
- NAT Subnet B
- Private Subnet A
- Private Subnet B

Production Tokyo VPC:

- Public Subnet A
- Public Subnet C
- NAT Subnet A
- NAT Subnet C
- Private Subnet A
- Private Subnet C


我们现在已经知道，我们需要创建4个VPC，24个VPC Subnet。那么我们要直接将`10.0.0.0/16`划分为四个子网(VPC)，然后再对四个子网继续划分子网(VPC Subnet)吗？回答这个问题之前，我们先想一下如果现在就将其划分为四个子网，那么以后我们有其他的环境要创建怎么办？

最好的办法就是**按需划分网络**，为了按需划分网络我们得提前预估网络需要容纳多少台主机。

如果我们的每个VPC Subnet网络需要容纳100台主机。通过计算：2<sup>7</sup> - **5** = 123 > 100，我们得知主机位需要7位就可以满足我们需求。主机部分7位，则网络部分25位。

至于什么要减5：
> The first four IP addresses and the last IP address in each subnet CIDR block are not available for you to use, and cannot be assigned to an instance. For example, in a subnet with CIDR block `10.0.0.0/24`, the following five IP addresses are reserved:

*   `10.0.0.0`: Network address.

*   `10.0.0.1`: Reserved by AWS for the VPC router.

*   `10.0.0.2`: Reserved by AWS. The IP address of the DNS server is always the base of the VPC network range plus two; however, we also reserve the base of each subnet range plus two. For VPCs with multiple CIDR blocks, the IP address of the DNS server is located in the primary CIDR. For more information, see [Amazon DNS Server](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_DHCP_Options.html#AmazonDNS).

*   `10.0.0.3`: Reserved by AWS for future use.

*   `10.0.0.255`: Network broadcast address. We do not support broadcast in a VPC, therefore we reserve this address.

#### VPC CIDR 划分

由于每个VPC是由6个VPC Subnet组成，通过计算：2<sup>3</sup> = 8 > 6，所以VPC CIDR网络部分需25 - 3 = 22位，主机部分10位：

VPC CIDR                         |  网络部分                                         | 主机部分
---------------------------------- | ---------------------------------------------- |  ----------------
Dev VPC                           | 00001010   00000000 0000**00**   | 00 00000000
Staging VPC                     | 00001010   00000000 0000**01**   | 00 00000000
Production Singpore VPC | 00001010   00000000 0000**10**   | 00 00000000
Production Tokyo VPC      | 00001010   00000000 0000**11**   | 00 00000000

我们将`10.0.0.0/16`划分成了2<sup>6</sup> = 64个`/22`的子网，而我们只使用了其中的4个`/22`的子网。

#### VPC Subnet CIDR 划分

##### Dev VPC Subnet CIDR

现在我们再来将Dev环境VPC CIDR`10.0.0.0/22`划分成VPC Subnet CIDR:

VPC Subnet         |  网络部分                                                     | 主机部分
----------------------- | -------------------------------------------------------- |  -------------
Public Subnet A   | 00001010   00000000 000000**00**  **0** | 0000000
Public Subnet B   | 00001010   00000000 000000**00**  **1** | 0000000
NAT Subnet A      | 00001010   00000000 000000**01**  **0** | 0000000
NAT Subnet B      | 00001010   00000000 000000**01**  **1** | 0000000
Private Subnet A  | 00001010   00000000 000000**10**  **0** | 0000000
Private Subnet B  | 00001010   00000000 000000**10**  **1** | 0000000
未使用                  | 00001010   00000000 000000**11**  **0** | 0000000
未使用                  | 00001010   00000000 000000**11**  **1** | 0000000

转换成VPC Subnet CIDR:

VPC Subnet         |   VPC Subnet CIDR
----------------------- | --------------------------
Public Subnet A   | 10.0.0.0/25
Public Subnet B   | 10.0.0.128/25
NAT Subnet A      | 10.0.1.0/25
NAT Subnet B      | 10.0.1.128/25
Private Subnet A  | 10.0.2.0/25
Private Subnet B  | 10.0.2.128/25

##### Staging VPC Subnet CIDR
现在我们再来将Staging环境VPC CIDR`10.0.4.0/22`划分成VPC Subnet CIDR:

VPC Subnet         |  网络部分                                                     | 主机部分
----------------------- | -------------------------------------------------------- |  -------------
Public Subnet A   | 00001010   00000000 000001**00**  **0** | 0000000
Public Subnet B   | 00001010   00000000 000001**00**  **1** | 0000000
NAT Subnet A      | 00001010   00000000 000001**01**  **0** | 0000000
NAT Subnet B      | 00001010   00000000 000001**01**  **1** | 0000000
Private Subnet A  | 00001010   00000000 000001**10**  **0** | 0000000
Private Subnet B  | 00001010   00000000 000001**10**  **1** | 0000000
未使用                  | 00001010   00000000 000001**11**  **0** | 0000000
未使用                  | 00001010   00000000 000001**11**  **1** | 0000000

转换成VPC Subnet CIDR:

VPC Subnet         |   VPC Subnet CIDR
----------------------- | --------------------------
Public Subnet A   | 10.0.4.0/25
Public Subnet B   | 10.0.4.128/25
NAT Subnet A      | 10.0.5.0/25
NAT Subnet B      | 10.0.5.128/25
Private Subnet A  | 10.0.6.0/25
Private Subnet B  | 10.0.6.128/25

##### Production Singapore VPC Subnet CIDR

现在我们再来将Production Singapore环境VPC CIDR`10.0.8.0/22`划分成VPC Subnet CIDR:

VPC Subnet         |  网络部分                                                     | 主机部分
----------------------- | -------------------------------------------------------- |  -------------
Public Subnet A   | 00001010   00000000 000010**00**  **0** | 0000000
Public Subnet B   | 00001010   00000000 000010**00**  **1** | 0000000
NAT Subnet A      | 00001010   00000000 000010**01**  **0** | 0000000
NAT Subnet B      | 00001010   00000000 000010**01**  **1** | 0000000
Private Subnet A  | 00001010   00000000 000010**10**  **0** | 0000000
Private Subnet B  | 00001010   00000000 000010**10**  **1** | 0000000
未使用                  | 00001010   00000000 000010**11**  **0** | 0000000
未使用                  | 00001010   00000000 000010**11**  **1** | 0000000

转换成VPC Subnet CIDR:

VPC Subnet         |   VPC Subnet CIDR
----------------------- | --------------------------
Public Subnet A   | 10.0.8.0/25
Public Subnet B   | 10.0.8.128/25
NAT Subnet A      | 10.0.9.0/25
NAT Subnet B      | 10.0.9.128/25
Private Subnet A  | 10.0.10.0/25
Private Subnet B  | 10.0.10.128/25

##### Production Tokyo VPC Subnet CIDR

现在我们再来将Production Tokyo环境VPC CIDR`10.0.12.0/22`划分成VPC Subnet CIDR:

VPC Subnet         |  网络部分                                                     | 主机部分
----------------------- | -------------------------------------------------------- |  -------------
Public Subnet A   | 00001010   00000000 000011**00**  **0** | 0000000
Public Subnet C   | 00001010   00000000 000011**00**  **1** | 0000000
NAT Subnet A      | 00001010   00000000 000011**01**  **0** | 0000000
NAT Subnet C      | 00001010   00000000 000011**01**  **1** | 0000000
Private Subnet A  | 00001010   00000000 000011**10**  **0** | 0000000
Private Subnet C  | 00001010   00000000 000011**10**  **1** | 0000000
未使用                  | 00001010   00000000 000011**11**  **0** | 0000000
未使用                  | 00001010   00000000 000011**11**  **1** | 0000000

转换成VPC Subnet CIDR:

VPC Subnet         |   VPC Subnet CIDR
----------------------- | --------------------------
Public Subnet A   | 10.0.12.0/25
Public Subnet B   | 10.0.12.128/25
NAT Subnet A      | 10.0.13.0/25
NAT Subnet B      | 10.0.13.128/25
Private Subnet A  | 10.0.14.0/25
Private Subnet B  | 10.0.14.128/25