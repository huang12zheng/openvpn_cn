# 配置特定于客户端的规则和访问策略

假设我们正在建立公司VPN，并且我们想为3种不同类别的用户建立单独的访问策略：

* 系统管理员 -完全访问网络中的所有计算机
* 员工 -仅访问Samba /电子邮件服务器
* 承包商 -仅访问特殊服务器
我们将采用的基本方法是（a）将每个用户类别划分到其自己的虚拟IP地址范围内，并且（b）通过设置防火墙规则来关闭客户端的虚拟IP地址，从而控制对计算机的访问。

在我们的示例中，假设我们有可变数量的员工，但只有一名系统管理员和两名承包商。我们的IP分配方法是将所有员工放入一个IP地址池，然后为系统管理员和承包商分配固定的IP地址。

请注意，此示例的先决条件之一是您在OpenVPN服务器计算机上运行了软件防火墙，该防火墙使您能够定义特定的防火墙规则。对于我们的示例，我们将假定防火墙为Linux  iptables。

首先，让我们根据用户类别创建一个虚拟IP地址映射：

<table border="1" cellspacing="0" cellpadding="8">
<tbody>
<tr>
<td><strong>Class</strong></td>
<td><strong>Virtual IP Range</strong></td>
<td><strong>Allowed LAN Access</strong></td>
<td><strong>Common Names</strong></td>
</tr>
<tr>
<td>Employees</td>
<td>10.8.0.0/24</td>
<td>Samba/email server at 10.66.4.4</td>
<td>[variable]</td>
</tr>
<tr>
<td>System Administrators</td>
<td>10.8.1.0/24</td>
<td>Entire 10.66.4.0/24 subnet</td>
<td>sysadmin1</td>
</tr>
<tr>
<td>Contractors</td>
<td>10.8.2.0/24</td>
<td>Contractor server at 10.66.4.12</td>
<td>contractor1, contracter2</td>
</tr>
</tbody>
</table>

接下来，让我们将此映射转换为OpenVPN服务器配置。首先，确保你已经采取的步骤 上述 用于制作适用于所有客户（而我们将配置路由，以允许对整个10.66.4.0/24子网的客户端访问的10.66.4.0/24子网，我们会再施以使用防火墙规则实施上述策略表的访问限制）。

首先，为我们的tun 接口定义一个静态单元号 ，以便稍后我们可以在防火墙规则中引用它：

```
dev tun0
```

在服务器配置文件中，定义员工IP地址池
```
server 10.8.0.0 255.255.255.0
```

添加系统管理员和承包商IP范围的路由：
```
route 10.8.1.0 255.255.255.0
route 10.8.2.0 255.255.255.0
```
因为我们将为特定的系统管理员和承包商分配固定的IP地址，所以我们将使用客户端配置目录：

```
client-config-dir ccd
```

然后，将特殊的配置文件放在 ccd 子目录中，以为每个非雇员VPN客户端定义固定的IP地址。

ccd/sysadmin1
```
ifconfig-push 10.8.1.1 10.8.1.2
```
CCD/contractor1
```
ifconfig-push 10.8.2.1 10.8.2.2
```
ccd/contractor2
```
ifconfig-push 10.8.2.5 10.8.2.6
```
每对 ifconfig-push 地址代表虚拟客户端和服务器IP端点。必须从连续的/ 30子网中获取它们，以便与Windows客户端和TAP-Windows驱动程序兼容。具体来说，每个端点对的IP地址中的最后一个八位位组必须来自以下集合：
```
[1，2] [5，6] [9，10] [13，14] [17，18] 
[21，22] [25，26] [29，30] [33，34] [37，38] 
[41，42] [45，46] [49，50] [53，54] [57，58] 
[61，62] [65，66] [69，70] [73，74] [77，78] 
[81，82] [85，86] [89，90] [93，94] [97，98] 
[101,102] [105,106] [109,110] [113,114] [117,118] 
[121,122] [125,126] [129,130​​] [ 133,134] [137,138] 
[141,142] [145,146] [149,150] [153,154] [157,158] 
[161,162] [165,166] [169,170] [173,174] [177,178] 
[181,182] [185,186] [189,190] [193,194] [197,198] 
[201,202] [205,206] [209,210] [213,214] [217,218] 
[221,222] [225,226] [229,230] [233,234] [237,238] 
[241,242] [245,246] [249,250] [253,254]
```
这样就完成了OpenVPN配置。最后一步是添加防火墙规则，以最终确定访问策略。对于此示例，我们将在Linux iptables 语法中使用防火墙规则 ：

```
# Employee rule
iptables -A FORWARD -i tun0 -s 10.8.0.0/24 -d 10.66.4.4 -j ACCEPT

# Sysadmin rule
iptables -A FORWARD -i tun0 -s 10.8.1.0/24 -d 10.66.4.0/24 -j ACCEPT

# Contractor rule
iptables -A FORWARD -i tun0 -s 10.8.2.0/24 -d 10.66.4.12 -j ACCEPT
```