---
title: KVM
date: 2016-12-26 13:37:33
tags:
---
## 一、安装kvm

1. `$ sudo apt-get install qemu-kvm libvirt-bin ubuntu-vm-builder bridge-utils`
2. ``$ sudo adduser `id -un` libvirtd``
3. 验证安装:
`$ virsh -c qemu:///system list`
没有错误信息就表明安装成功！
确认如下两个文件的权限正确：
```
$ sudo ls -la /var/run/libvirt/libvirt-sock
srwxrwx--- 1 root libvirtd 0 2010-08-24 14:54 /var/run/libvirt/libvirt-sock
$ ls -l /dev/kvm
crw-rw----+ 1 root kvm 10, 232 Jul  8 22:04 /dev/kvm
```
4. 安装完毕后，会自动生成虚拟网卡 virbr0，可以通过 ifconfig 查看确认一下
5. 创建网卡桥接
`$ sudo vi /etc/network/interfaces`
增加如下内容：
```
auto br0
iface br0 inet static
address 192.168.0.222
netmask 255.255.255.0
gateway 192.168.0.1
bridge_ports eth0
```
保存，重启网卡设置，`ifconfig` 再查看网卡的设置，这个时候 eth0 已经没有具体的IP地址，IP地址出在了 br0 的虚拟网上面。
安装后，默认的目录为：`/etc/libvirt/qemu/`

## 二、kvm 虚拟机的日常管理

```
 virsh list --all > 查看虚拟机状态
 virsh start [name] > 开机
 virsh shutdown [name] > 关机
 virsh destroy [name] > 强制关闭电源
 virsh reboot[name] > 关机
 virsh suspend [name] > 挂起服务器
 virsh resume [name] > 恢复服务器
 virsh autostart [name] > 配置开机自启动虚拟机 （/etc/libvirt/qemu/autostart/）
 virsh undefine [name] > 删除虚拟机 (只删除配置文件，并不删除虚拟磁盘文件)
 virsh create /etc/libvirt/qemu/[name].xml > 通过配置文件启动虚拟机
 virsh dumpxml [name] > /etc/libvirt/qemu/[name].xml > 导出KVM虚拟机配置文件(可通过这种方式进行备份)
 virsh edit [name] > 编辑虚拟机配置文件(/etc/libvirt/qemu/[name].xml，不建议直接通过vi编辑)
 virsh --version > 查看虚拟工具版本
 virt-install --version > 查看虚拟工具版本
 qemu-kvm -version > 查看虚拟工具版本
```
注：virsh 命令丰富，可以执行各种维护任务。

## 三、安装 Win2003SP2 虚拟机

1. 上传系统 iso 文件到 `/file/tools/win/win2003/WIN_2003_SP2.iso`
2. 开始安装
```
$ virt-install --name=win2003sp2 \
--hvm \
--ram 1024 \
--vcpus=1 \
--os-type=windows \
--os-variant=win2k3 \
--arch=x86_64 \
--disk path=/file/kvm/win2003sp2/win2003sp2.img,size=20 \
--network bridge=br0 \
--accelerate \
--graphics vnc,listen=0.0.0.0,port=5911 \
--cdrom /file/tools/win/win2003/WIN_2003_SP2.iso \
--boot cdrom
```
参数说明:
```
--name      指定虚拟机名称
--hvm        使用全虚拟化（与--paravirt相对）
--ram        分配内存大小，单位为 MB
--vcpus     分配CPU核心数，最大与实体机CPU核心数相同
--disk        指定虚拟机使用的磁盘，path 指定路径，size 指定大小，单位为G
--network  网络设置，使用默认配置时可设为 "--network network:default"
--accelerate 加速
--graphics vnc,listen=0.0.0.0,port=5911   指定VNC监控端口、绑定IP，默认端口为5900，端口不能重复；IP 默认绑定127.0.0.1，这里改为 0.0.0.0
--cdrom     指定安装镜像iso设置光驱获取虚拟光驱文件的路径
--os-type=windows 
--os-variant=win2k3 
--arch=x86_64 
--boot cdrom 指定从 cdrom 启动，从硬盘启动时设为 hd
--autostart      指定主机启动时自动启动此虚拟机
```

3. 将虚拟机的启动方式改为从磁盘启动
`$ virsh edit win2003sp3`

将 `<boot dev='cdrom'/>`改为 `<boot dev='hd'/>`
然后重新启动虚拟机：
`$virsh destroy win2003sp3`
`$virsh start win2003sp3`
之后就可以重新使用 vnc 连接，继续安装过程。
4. 安装完毕后按需要设置IP地址、远程连接等即可

## 四、KVM 克隆

复制已经安装好的虚拟机为另一个全新的虚拟机。如要将上面的 win2003sp2 复制为 win2003sp2-2：
1. 关闭 win2003sp2
`$virsh shutdown win2003sp2`
2. 克隆 win2003sp2-2
`$virt-clone -o win2003sp2 -n win2003sp2-2 -f /file/kvm/win2003sp2/win2003sp2-2.img`
如果虚拟机文件比较大，复制的时间就会比较长。
3. 修改 win2003sp2-2.xml 文件
修改 vnc 端口避免与 win2003sp2 重复。
`$virsh edit win2003sp2-2`
`<graphics type='vnc' port='5912' autoport='no' listen='0.0.0.0'>`
4. 启动 win2003sp2-2
`$virsh start win2003sp2-2`
最后利用 vnc 登录到 win2003sp2-2 之后，修改网络设置，避免 IP 冲突。

## 五、KVM 快照

对虚拟机做快照，防止损坏了可以正常恢复。kvm 快照分两种：
第1种：lvm 快照，如果分区是 lvm，可以利用 lvm 进行 kvm 的快照备份
第2种：由于 raw 格式不支持镜像，所以需要将格式转换为 qcow2 才可以创建快照。
第2种具体操作如下：
kvm 虚拟机默认使用 raw 格式的镜像格式，性能最好，速度最快，它的缺点就是不支持一些新的功能，如镜像、zlib  磁盘压缩、AES加密等。   
要使用镜像功能，磁盘格式必须为 qcow2。
下面开始 kvm 虚拟机快照备份的过程：
1. 查看磁盘格式
`$qemu-img info win2003sp2.img`
2. 关闭 win2003sp2
`$virsh shutdown win2003sp2`
3. 转换磁盘格式
`$qemu-img convert -f raw -O qcow2 win2003sp2.img win2003sp2.img.qcow2`
-f  源镜像的格式   
-O 目标镜像的格式
4. 修改虚拟机配置文件，指向新的磁盘格式文件
`$virsh edit win2003sp2`
```
<devices>
  ...
  <disk type='file' device='disk'>
    <driver name='qemu' type='qcow2' cache='none'/>
    <source file='/file/kvm/win2003sp2/win2003sp2.img.qcow2'/>
    <target dev='hda' bus='ide'/>
    <address type='drive' controller='0' bus='0' target='0' unit='0'/>
  </disk>
  ...
</devices>
```
注：转换后镜像文件占用的空间大大减少了，而且日后为动态自动增加，而不是固定占用开始设置时占用的大小。
5. 创建快照 
`$virsh snapshot-create-as win2003sp2 clean`
6. 查看虚拟机镜像快照的版本
`$virsh snapshot-list win2003sp2`
7. 看当前最新的快照版本
`$virsh snapshot-current win2003sp2`
8. 恢复虚拟机快照
恢复虚拟机快照前必须先关闭虚拟机！
`$virsh snapshot-revert win2003sp2 clean`
9. 删除虚拟机快照
`$virsh snapshot-delete win2003sp2 clean`
