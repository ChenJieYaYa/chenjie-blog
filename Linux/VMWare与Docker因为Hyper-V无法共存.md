# VMWare与Docker因为Hyper-V无法共存

## 一、使用Docker

* 以管理员身份运行命令提示符
* 输入`bcdedit /set hypervisorlaunchtype auto`
* 重启电脑

## 二、使用VMWare

* 以管理员身份运行命令提示符
* 输入`bcdedit /set hypervisorlaunchtype off`
* 重启电脑

目前没有找到更好的解决办法







