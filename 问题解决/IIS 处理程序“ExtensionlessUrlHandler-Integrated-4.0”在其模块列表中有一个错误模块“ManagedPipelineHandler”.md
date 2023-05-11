## IIS 处理程序“ExtensionlessUrlHandler-Integrated-4.0”在其模块列表中有一个错误模块“ManagedPipelineHandler”

出现这个错误是因为 IIS 7 采用了更安全的 web.config 管理机制，默认情况下会锁住配置项不允许更改。要取消锁定可以以管理员身份运行命令行 %windir%\system32\inetsrv\appcmd unlock config -section:system.webServer/handlers，其中的 handlers 是错误信息中红字显示的节点名称。

如果modules也被锁定，可以运行%windir%\system32\inetsrv\appcmd unlock config -section:system.webServer/modules

win10下IIS站点访问不了，原因是因为IIS 没有.net 4.5,使用网上的aspnet_regiis.exe -i命令有问题，直接提示：

> C:\WINDOWS\system32>c:\windows\microsoft.net\framework64\v4.0.30319\aspnet_regiis.exe -i
> Microsoft ® ASP.NET RegIIS 版本 4.0.30319.0
> 用于在本地计算机上安装和卸载 ASP.NET 的管理实用工具。
> 版权所有© Microsoft Corporation。保留所有权利。
> 开始安装 ASP.NET (4.0.30319.0)。
> 此操作系统版本不支持此选项。管理员应使用“打开或关闭 Windows 功能”对话框、“服务器管理器”管理工具或 dism.exe 命令行工 具安装/卸载包含 IIS8 的 ASP.NET 4.5。
> 有关更多详细信息，请参见 http://go.microsoft.com/fwlink/?LinkID=216771。
> ASP.NET (4.0.30319.0)安装完毕。

**终极解决办法：
利用dism工具依次执行下面命令**

````cmdd
dism /online /enable-feature /featurename:IIS-ISAPIFilter
dism /online /enable-feature /featurename:IIS-ISAPIExtensions
dism /online /enable-feature /featurename:IIS-NetFxExtensibility45
dism /online /enable-feature /featurename:IIS-ASPNET45
````

