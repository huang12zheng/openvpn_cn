# 将DHCP选项推送给客户端

OpenVPN服务器可以将DHCP选项（例如DNS和WINS服务器地址）推送给客户端（需要注意一些 警告 ）。Windows客户端可以接受本地推DHCP选项，而非Windows客户端可以使用客户端接受他们 了 脚本，分析 foreign_option_ ñ环境变量列表。有关 非Windows  foreign_option_ n 文档和脚本示例的信息，请参见 手册页。

例如，假设您希望连接客户端以使用内部DNS服务器（位于10.66.0.4或10.66.0.5）和WINS服务器（位于10.66.0.8）。将此添加到OpenVPN服务器配置中：
```
push "dhcp-option DNS 10.66.0.4"
push "dhcp-option DNS 10.66.0.5"
push "dhcp-option WINS 10.66.0.8"
```

要在Windows上测试此功能，请在计算机连接到OpenVPN服务器后从命令提示符窗口运行以下命令：
```
ipconfig /all
```
TAP-Windows适配器的条目应显示服务器推送的DHCP选项。