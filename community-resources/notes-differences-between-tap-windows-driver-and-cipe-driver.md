# TAP-Windows驱动程序和CIPE驱动程序之间的区别

随OpenVPN 1.5-beta5和更高版本分发的TAP-Windows驱动程序源自Cipe-Win32 2.0-pre15，并进行了一些重大更改：

使用Michael Clarke的修补程序将驱动程序升级到NDIS5，正确实现了睡眠/恢复OID，并修复了“ AdapterTransmit”和“ IRP_MJ_READ”之间的竞争条件，稳定性得到了很大改善，尤其是在睡眠/恢复时。

添加了Christof Meerwald的“媒体状态”修补程序，该修补程序显示给定的TAP-Win32适配器当前未由OpenVPN实例打开时被“拔出”。

修改了MAC生成代码，以遵循Linux生成MAC的算法，使用“ 0：FF：XX：XX：XX：XX：XX”，其中“ XX：XX：XX：XX”是随机的。

添加了用于锁定TAP设备的代码，以便一次只能打开一个OpenVPN实例。

添加了一个MTU参数，其作用类似于Linux下的ifconfig mtu参数。MTU的默认值为“ 1500”，可以通过适配器高级属性对话框进行更改。

设置驱动程序以跟踪其Rx / Tx统计信息，而不是依靠用户空间来设置它们。

在启用所有测试模式（包括低资源模拟模式）的情况下，通过Windows驱动程序验证器运行驱动程序。根据产生的错误检查，我能够解决许多问题，包括使用“ MmGetSystemAddressForMdlSafe”代替“ MmGetSystemAddressForMdl”，修复代码中未检查“ NdisAllocateMemory”返回状态的多个位置以及使标志匹配在“ NdisAllocateMemory”和“ NdisFreeMemory”调用之间。

重命名驱动程序，使其在网络控制面板中显示为“ TAP-Windows”适配器，并且与CIPE驱动程序不冲突。

使驱动程序达到SMP标准（beta8），将数据包排队子例程重做为循环队列，以提高效率并在SMP下更直观地锁定语义。

修复了悬空的IRP错误，如果驱动程序在用户空间进程（beta8）仍处于打开状态时被卸载或禁用，则可能导致错误检查。

修复了以下错误：如果用户空间进程尝试读取数据包但提供的缓冲区太小而无法完全返回该包（beta8），则该适配器实例将无法使用。

添加了几个新的ioctl，以将有趣的状态信息返回给用户空间，例如当前配置的MTU值，驱动程序版本号和扩展的错误状态信息（beta8）。

添加了“ tun”设备仿真（beta8）。

现在可以使用“ TAP_IOCTL_SET_MEDIA_STATUS” ioctl从用户空间直接控制适配器的介质状态。

一个选项已添加到TAP-Windows驱动程序的高级属性页面，该选项使您可以控制适配器在Windows中显示为“始终处于连接状态”，还是由OpenVPN动态启动和关闭连接状态（“应用程序控制”）。

在某种程度上，为了更好地使用Win2K / XP的可用性和稳定性，已经牺牲了与NT 4的向后兼容性。