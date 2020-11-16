# 配置OpenVPN在系统启动时自动运行

该领域缺乏标准意味着大多数操作系统在启动时自动配置守护程序/服务以采用不同的方式。默认情况下，配置此功能的最佳方法是将OpenVPN作为软件包安装，例如通过Linux上的RPM或使用Windows安装程序。

* Linux

如果您通过Linux上的RPM或DEB软件包安装OpenVPN，则安装程序将设置 initscript。执行后，初始化脚本将 在/etc/openvpn中扫描 .conf配置文件 如果找到，将为每个文件启动一个单独的OpenVPN守护程序。

* Windows

Windows安装程序将设置服务包装程序，但默认情况下将其关闭。要激活它，请转到控制面板/管理工具/服务，选择OpenVPN服务，右键单击属性，然后将启动类型设置为自动。这会将服务配置为在下次重新启动时自动启动。

启动后，OpenVPN服务包装程序将扫描 \Program Files\OpenVPN\config-auto文件夹中的.ovpn配置文件，并在每个文件上启动单独的OpenVPN进程。

>注意：在较旧版本的OpenVPN GUI上，"config"目录曾经是所有配置的存储，该服务将在此处启动所有配置。从2.5.0版开始，"config"目录用于GUI组件的配置，而"config-auto"目录用于服务包装器以从中自动启动配置。