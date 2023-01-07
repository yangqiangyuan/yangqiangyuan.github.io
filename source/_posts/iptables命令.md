---
title: iptables命令
date: 2023-01-07 10:20:14
tags: iptables
---

### iptables命令

<!--more-->

```
语法
iptables(选项)(参数)

这些选项指定执行明确的动作：若指令行下没有其他规定,该行只能指定一个选项. 对于长格式的命令和选项名,所用字母长度只要保证iptables能从其他选项中区 分出该指令就行了。

-A -append
在所选择的链末添加一条或更多规则。当源（地址）或者/与 目的（地址）转换 为多于一个(多个)地址时，这条规则会加到所有可能的地址(组合)后面。
-D -delete
从所选链中删除一条或更多规则。这条命令可以有两种方法：可以把被删除规则 指定为链中的序号(第一条序号为1),或者指定为要匹配的规则。
-R -replace
从选中的链中取代一条规则。如果源（地址）或者/与 目的（地址）被转换为多地 址，该命令会失败。规则序号从1开始。
-I -insert
根据给出的规则序号向所选链中插入一条或更多规则。所以，如果规则序号为1， 规则会被插入链的头部。这也是不指定规则序号时的默认方式。
-L -list
显示所选链的所有规则。如果没有选择链，所有链将被显示。也可以和z选项一起 使用，这时链会被自动列出和归零。精确输出受其它所给参数影响。
-F -flush
清空所选链。这等于把所有规则一个个的删除。
--Z -zero
把所有链的包及字节的计数器清空。它可以和 -L配合使用，在清空前察看计数器，请参见前文。
-N -new-chain
根据给出的名称建立一个新的用户定义链。这必须保证没有同名的链存在。
-X -delete-chain
删除指定的用户自定义链。这个链必须没有被引用，如果被引用，在删除之前你必须删 除或者替换与之有关的规则。如果没有给出参数，这条命令将试着删除每个非 内建的链。
-P -policy
设置链的目标规则。
-E -rename-chain
根据用户给出的名字对指定链进行重命名，这仅仅是修饰，对整个表的结构没有影响。 TARGETS参数给出一个合法的目标。只有非用户自定义链可以使用规则，而且内建链和用 户自定义链都不能是规则的目标。
-h Help.
帮助。给出当前命令语法非常简短的说明。

p -protocal [!]protocol
规则或者包检查(待检查包)的协议。指定协议可以是tcp、udp、icmp中的一个或 者全部，也可以是数值，代表这些协议中的某一个。当然也可以使用在/etc/pro tocols中定义的协议名。在协议名前加上"!"表示相反的规则。数字0相当于所有 all。Protocol all会匹配所有协议，而且这是缺省时的选项。在和check命令结合 时，all可以不被使用。

-s -source [!] address[/mask]
指定源地址，可以是主机名、网络名和清楚的IP地址。mask说明可以是网络掩码 或清楚的数字，在网络掩码的左边指定网络掩码左边”1”的个数，因此，mask 值为24等于255.255.255.0。在指定地址前加上"!"说明指定了相反的地址段。标志 
 --src 是这个选项的简写。

-d --destination [!] address[/mask]
指定目标地址，要获取详细说明请参见 -s标志的说明。标志 --dst 是这个选项的简写。

-j --jump target
(-j 目标跳转)指定规则的目标；也就是说，如果包匹配应当做什么。目标可以是用 户自定义链（不是这条规则所在的），某个会立即决定包的命运的专用内建目标， 或者一个扩展（参见下面的EXTENSIONS）。如果规则的这个选项被忽略，那么匹 配的过程不会对包产生影响，不过规则的计数器会增加。

-i -in-interface [!] [name]
(i -进入的（网络）接口 [!][名称])这是包经由该接口接收的可选的入口名称，包通过 该接口接收（在链INPUT、FORWORD和PREROUTING中进入的包）。当在接口名 前使用"!"说明后，指的是相反的名称。如果接口名后面加上"+"，则所有以此接口名 开头的接口都会被匹配。如果这个选项被忽略，会假设为"+"，那么将匹配任意接口。

-o --out-interface [!][name]
(-o --输出接口[名称])这是包经由该接口送出的可选的出口名称，包通过该口输出（在 链FORWARD、OUTPUT和POSTROUTING中送出的包）。当在接口名前使用"!"说明 后，指的是相反的名称。如果接口名后面加上"+"，则所有以此接口名开头的接口都会 被匹配。如果这个选项被忽略，会假设为"+"，那么将匹配所有任意接口。

--dport num  匹配目标端口号
--sport num  匹配来源端口号
```

iptables命令顺序

```
iptables -t 表名 <-A/I/D/R> 规则链名 [规则号] <-i/o 网卡名> -p 协议名 <-s 源IP/源子网> --sport 源端口 <-d 目标IP/目标子网> --dport 目标端口 -j 动作
```

**表名包括：**

- **raw**：高级功能，如：网址过滤。
- **mangle**：数据包修改（QOS），用于实现服务质量。
- **net**：地址转换，用于网关路由器。
- **filter**：包过滤，用于防火墙规则。

**规则链名包括：**

- **INPUT链**：处理输入数据包。
- **OUTPUT链**：处理输出数据包。
- **PORWARD链**：处理转发数据包。
- **PREROUTING链**：用于目标地址转换（DNAT）。
- **POSTOUTING链**：用于源地址转换（SNAT）。

**动作包括：**

- **ACCEPT**：接收数据包。
- **DROP**：丢弃数据包。
- **REDIRECT**：重定向、映射、透明代理。
- **SNAT**：源地址转换。
- **DNAT**：目标地址转换。
- **MASQUERADE**：IP伪装（NAT），用于ADSL。
- **LOG**：日志记录。

#### 案例

*1.禁止服务器被ping，服务器拒绝icmp的流量，给与响应*

```
#给INPUT链添加规则，指定icmp协议，指定icmp类型 是8(回显请求)，  -s指定网段范围  -j 跳转的目标，即将做什么
iptables -A INPUT -p icmp --icmp-type 8 -s 0/0 -j REJECT
```

*2.服务器禁ping，请求直接丢弃*

```
[root@localhost ~]# iptables -F
[root@localhost ~]# iptables -A INPUT -p icmp --icmp-type 8 -s 0/0 -j DROP
```

3.检查防火墙规则

```
[root@localhost ~]# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
DROP       icmp --  anywhere             anywhere             icmp echo-request

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```

4.清空所有防火墙规则链

```
[root@localhost ~]# iptables -F
[root@localhost ~]# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```

**5.注意不要轻易在云服务器上设置，默认拒绝的规则，否则ssh流量进不去，直接断开远程连接了**

6.删除**第一条**规则

```
[root@localhost ~]# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
DROP       icmp --  anywhere             anywhere             icmp echo-request

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
[root@localhost ~]#
[root@localhost ~]# iptables -D INPUT 1
```

7.禁止访问本机的80端口

```
#禁止流量进入，指定tcp类型，拒绝的端口是80，动作是拒绝
iptables -A INPUT -p tcp --dport 80 -j DROP
```

8.禁止服务器的FTP端口,也就是禁止21端口

```
#服务器禁止21端口流量
[root@localhost ~]# iptables -A INPUT -p tcp --dport 21 -j DROP
```

9.只允许指定的ip远程连接此服务器，拒绝其他主机22端口流量

```
#iptables自上而下匹配
iptables -A INPUT -s 222.35.242.139/24 -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j REJECT

[root@localhost ~]# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
ACCEPT     tcp  --  222.35.242.0/24      anywhere             tcp dpt:ssh
REJECT     tcp  --  anywhere             anywhere             tcp dpt:ssh reject-with icmp-port-unreachable

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination


#换一台ip的机器，直接被拒绝
[root@web01 ~]# ssh root@123.206.16.61
ssh: connect to host 123.206.16.61 port 22: Connection refused

#只要删除第二条拒绝的规则，即可
[root@localhost ~]# iptables -D INPUT 2

#又可以连接了
[root@web01 ~]# ssh root@123.206.16.61
```

10.禁止指定的机器ip，访问本机的80端口策略，可以封禁某些恶意请求

```
#此时的防火墙规则
[root@localhost ~]# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
ACCEPT     tcp  --  222.35.242.0/24      anywhere             tcp dpt:ssh
REJECT     tcp  --  anywhere             anywhere             tcp dpt:ssh reject-with icmp-port-unreachable

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination


#在规则链开头，追加一个新规则,禁止某个ip地址，访问本机的80端口
[root@localhost ~]# iptables -I INPUT -p tcp -s 222.35.242.139/24 --dport 80 -j REJECT
[root@localhost ~]# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
REJECT     tcp  --  222.35.242.0/24      anywhere             tcp dpt:http reject-with icmp-port-unreachable
ACCEPT     tcp  --  222.35.242.0/24      anywhere             tcp dpt:ssh
REJECT     tcp  --  anywhere             anywhere             tcp dpt:ssh reject-with icmp-port-unreachable

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
[root@localhost ~]#


```

11.禁止所有的主机网段，访问本机的8000~9000的端口

```
[root@localhost ~]# iptables -A INPUT -p tcp -s 0/0 --dport  8000:9000 -j REJECT
[root@localhost ~]#
[root@localhost ~]#
[root@localhost ~]# iptables -A INPUT -p udp -s 0/0 --dport  8000:9000 -j REJECT
```
