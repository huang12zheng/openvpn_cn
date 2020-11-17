# 控制正在运行的OpenVPN进程

* 在Linux/BSD/Unix上运行
OpenVPN接受以下几种信号：

* SIGUSR1—有条件的重新启动，旨在在没有root特权的情况下重新启动
* SIGHUP-硬重启
* SIGUSR2—将连接统计信息输出到日志文件或syslog
* SIGTERM和 SIGINT—退出

使用 writepid 指令将OpenVPN守护程序的PID写入文件，以便您知道将信号发送到哪里（如果使用initscript启动openvpn，则脚本可能已经 在openvpn 命令行上传递了 –writepid指令 ）。

在Windows上以GUI身份运行
请参阅 OpenVPN GUI页面。

* 在Windows命令提示符窗口中运行

在Windows上，可以通过右键单击OpenVPN配置文件（.ovpn 文件）并选择“在此配置文件上启动OpenVPN”来启动OpenVPN。

以这种方式运行后，可以使用几个键盘命令：
```
F1—有条件的重新启动（不会关闭/重新打开TAP适配器）
F2—显示连接统计信息
F3-硬重启
F4-退出
```
* 作为Windows服务运行

在Windows上将OpenVPN作为服务启动时，控制它的唯一方法是：

通过服务控制管理器（“控制面板”/“管理工具”/“服务”）进行启动/停止控制。
通过管理界面（见下文）。

* 修改实时服务器配置

尽管大多数配置更改都需要您重新启动服务器，但是特别有两个指令是指可以实时动态更新的文件，它们将在服务器上立即生效，而无需重新启动服务器进程。

1. client-config-dir

此指令设置客户端配置目录，OpenVPN服务器将在每个传入连接上扫描该目录，以搜索特定于客户端的配置文件（ 有关更多信息，请参见 手册页）。可以在不重新启动服务器的情况下即时更新此目录中的文件。请注意，此目录中的更改仅对新连接有效，而对现有连接无效。如果您希望更改特定于客户端的配置文件对当前连接的客户端（或已断开连接但服务器尚未使其实例对象超时的客户端）立即生效，请通过使用管理来终止该客户端实例对象界面（如下所述）。这将导致客户端重新连接并使用新的 client-config-dir 文件。

2. crl-verify

此伪指令为证书吊销列表 文件命名 ，如以下“ 吊销证书” 部分所述。可以即时修改CRL文件，并且更改将立即对新连接或正在重新协商其SSL/TLS通道的现有连接生效（默认情况下每小时发生一次）。如果要终止其证书刚刚添加到CRL的当前连接的客户端，请使用管理界面（如下所述）。

* 状态文件

默认的 server.conf 文件有一行
```
status openvpn-status.log
```
它将 每分钟一次将当前客户端连接列表输出到文件 openvpn-status.log。

使用管理界面
该 OpenVPN的管理接口 允许在运行中的OpenVPN过程控制的很大。您可以通过远程登录到管理接口端口直接使用管理接口，也可以使用 本身连接到管理接口的OpenVPN GUI间接使用 管理接口。

要在OpenVPN服务器或客户端上启用管理接口，请将其添加到配置文件中：
```
management localhost 7505
```
这告诉OpenVPN在TCP端口7505上侦听管理接口客户端（端口7505是任意选择-您可以使用任何空闲端口）。

OpenVPN运行之后，您可以使用telnet 客户端连接到管理界面 。例如：

```
ai:~ # telnet localhost 7505
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
>INFO:OpenVPN Management Interface Version 1 -- type 'help' for more info
help
Management Interface for OpenVPN 2.0_rc14 i686-suse-linux [SSL] [LZO] [EPOLL] built on Feb 15 2005
Commands:
echo [on|off] [N|all]  : Like log, but only show messages in echo buffer.
exit|quit              : Close management session.
help                   : Print this message.
hold [on|off|release]  : Set/show hold flag to on/off state, or
                         release current hold and start tunnel.
kill cn                : Kill the client instance(s) having common name cn.
kill IP:port           : Kill the client instance connecting from IP:port.
log [on|off] [N|all]   : Turn on/off realtime log display
                         + show last N lines or 'all' for entire history.
mute [n]               : Set log mute level to n, or show level if n is absent.
net                    : (Windows only) Show network info and routing table.
password type p        : Enter password p for a queried OpenVPN password.
signal s               : Send signal s to daemon,
                         s = SIGHUP|SIGTERM|SIGUSR1|SIGUSR2.
state [on|off] [N|all] : Like log, but show state history.
status [n]             : Show current daemon status info using format #n.
test n                 : Produce n lines of output for testing/debugging.
username type u        : Enter username u for a queried OpenVPN username.
verb [n]               : Set log verbosity level to n, or show if n is absent.
version                : Show current version number.
END
exit
Connection closed by foreign host.
ai:~ #
```