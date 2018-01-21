---
title: Window 10 中 创建 VMware 虚拟机时 Hyper-V 问题
date: 2018-01-20 11:00:00
categories: [Tool]
tags: [WMware, Hyper-V]
---


> 摘要：本文简述了博主在 Window 10 安装用 VMware 创建 CentOS 虚拟机时，遇到的 Hyper-V 问题，以及解决方案。

### 问题

![问题描述](https://1csh1.github.io/img/vmware-install-in-window10-typer-v/01.hyper-v.png)

&emsp;&emsp;安装虚拟机的时候提示上图问题，开启 Hyper-V 也不能解决，找了一些资料，发现一篇文档的操作可以解决这个问题，所以记录下来。

### 解决方案

1. 关闭掉 Virtualization Based Security，如图操作
![关闭 Virtualization Based Security](https://1csh1.github.io/img/vmware-install-in-window10-typer-v/02.close-virtualization-based-security.png)

2. 关闭 Hyper-V，如图操作
![关闭 Hyper-V](https://1csh1.github.io/img/vmware-install-in-window10-typer-v/03.close-hyper-v.png)

3. 执行如下命令，**提示：确认驱盘X是没在使用的，可以修改为你没使用的驱盘**
```
mountvol X: /s
copy %WINDIR%\System32\SecConfig.efi X:\EFI\Microsoft\Boot\SecConfig.efi /Y
bcdedit /create {0cb3b571-2f2e-4343-a879-d86a476d7215} /d "DebugTool" /application osloader
bcdedit /set {0cb3b571-2f2e-4343-a879-d86a476d7215} path "\EFI\Microsoft\Boot\SecConfig.efi"
bcdedit /set {bootmgr} bootsequence {0cb3b571-2f2e-4343-a879-d86a476d7215}
bcdedit /set {0cb3b571-2f2e-4343-a879-d86a476d7215} loadoptions DISABLE-LSA-ISO,DISABLE-VBS
bcdedit /set {0cb3b571-2f2e-4343-a879-d86a476d7215} device partition=X:
mountvol X: /d
```

4. 然后重启电脑，就可以安装虚拟机啦。

希望本文可以帮助到你！

### 参考文章
[Powering on a vm in VMware Workstation on Windows 10 host where Credential Guard/Device Guard is enabled fails with BSOD (2146361)](https://kb.vmware.com/s/article/2146361)