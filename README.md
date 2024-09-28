### shell脚本：
一键给PVE添加CPU频率和各种温度显示，NVME硬盘，机械硬盘，固态硬盘信息：

- [代码链接](https://github.com/7568168/PVE-manager-status/tree/stcf)

![image](https://github.com/7568168/PVE-manager-status/blob/main/PVE效果图.png)

## 个人修改web网页界面

仅sata固态

1.硬盘通电时间计数，年月日

2.硬盘信息和状态分行显示，便于查看

## shh连接pve，root登陆运行

```json5
(curl -Lf -o /tmp/temp.sh https://raw.githubusercontent.com/7568168/PVE-manager-status/stcf/stcf || curl -Lf -o /tmp/temp.sh https://cgraw.pages.dev/https://raw.githubusercontent.com/7568168/PVE-manager-status/stcf/stcf) && chmod +x /tmp/temp.sh && /tmp/temp.sh remod
```


## 定位PVE虚拟机磁盘位置
qm config <vmid> 查看虚拟所拥有的磁盘
```json0
root@pve:~# qm config 101
...省略
scsi0: local-lvm:vm-101-disk-0,size=128M
scsi1: P4510:103/vm-102-disk-0.qcow2,discard=on,size=80G,ssd=1 
scsi2: NVME1:vm-103-disk-0,size=32G
...省略
```
格式 ： <vmdisk>: <storageid>:<vmid>/<diskid>,<disk option>
### 使用命令pvesm path 来定位
```json0
pvesm path local-lvm:vm-101-disk-0
pvesm path P4510:103/vm-102-disk-0.qcow2
pvesm path NVME1:vm-103-disk-0
输出举例：
root@zxgg:~# pvesm path local-lvm:vm-101-disk-0
/dev/pve/vm-101-disk-0
```

```json0
一些常见路径
这些路径在后续虚拟机迁移备份时用
存储配置文件:
/etc/pve/storage.cfg
存储路径local：
iso存放路径： /var/lib/vz/template/iso/​
虚拟机的备份路径： /var/lib/vz/dump/​
zfs的磁盘路径是：/dev/rpool/data/​
存储路径local-lvm，包括挂载的NFS、SMB等其它存储设备：/mnt/pve/
```
## PVE虚拟机 安装arm4 aarch64系统
- [来源1](https://blog.cfornas.casa/165/)
- - [来源2](https://blog.csdn.net/xumenghe1989/article/details/133382970)
  - - - [来源3](https://foxi.buduanwang.vip/virtualization/pve/2036.html/)
```json0
bios 选择OVMF（UEFI）
修改配置文件/连接到pve ssh/生成EFI

/etc/pve/qemu-server/your-server-id.conf
把cpu: x86-64-v2-AES删除，注释掉vmgenid（ # 注释），添加arch: aarch64，将scsihw改成virtio-scsi-pci，保存并退出。
![image](https://github.com/user-attachments/assets/b17f9757-6131-45dd-a161-4c3df00d97b8)
```

```json0
#查找QEMU_EFI.fd
find / -name QEMU_EFI.fd
#文件的存储路径根据实际情况进行修改
dd if=/usr/share/qemu-efi-aarch64/QEMU_EFI.fd of=/root/vm-101-disk-0.raw conv=notrunc

#扩展为64M
truncate -s 64M /root/vm-101-disk-0.raw
```

## pve上传证书后web网页打不开

- [来源链接](https://hostloc.com/forum.php?mod=redirect&goto=findpost&ptid=1141984&pid=13890625)

ssh上去删掉

/etc/pve/nodes/【节点名称】/pve-ssl-***.pem、pve-ssl-***.key
```json0
cd /etc/pve/nodes/pve
```
重启服务或reboot
```json0
systemctl restart pve-cluster
```

## 能耗控制

### 查看cpu频率，耗电等信息
```json0
apt install powertop
powertop
```

## 查看当前电源策略

```json0
cat /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
performance
```

## 查看可用的电源策略
```json0
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_available_governors
```
# conservative ondemand userspace powersave performance schedutil
```json0
performance	性能模式，将 CPU 频率固定工作在其支持的较高运行频率上，而不动态调节。
userspace	系统将变频策略的决策权交给了用户态应用程序，较为灵活。
powersave	省电模式，CPU 会固定工作在其支持的最低运行频率上。
ondemand	按需快速动态调整 CPU 频率，没有负载的时候就运行在低频，有负载就高频运行。
conservative	与 ondemand 不同，平滑地调整 CPU 频率，频率的升降是渐变式的，稍微缓和一点。
schedutil	负载变化回调机制，后面新引入的机制，通过触发 schedutil sugov_update 进行调频动作。

lscpu 查看 CPU 的频率

修改配置文件
文件位置/etc/init.d/cpufrequtils
#控制台进行修改
nano /etc/init.d/cpufrequtils
ENABLE="true"   
GOVERNOR="conservative"  #运行模式,依照需求调整
MAX_SPEED="0"         #自定义模式下设置cpu频率上限   ，非自定义模式不要填写，否则导致频率锁死最低频率
MIN_SPEED="0"         #下限
#脚本修改
apt install cpufrequtils
cat << 'EOF' > /etc/default/cpufrequtils
GOVERNOR="powersave"
EOF

#PVE一键优化脚本
首先是建议使用PVE一键优化脚本来做一些简单的优化和辅助设置，非常节省时间，教程参考：https://github.com/ivanhao/pvetools
先删除企业源：
rm /etc/apt/sources.list.d/pve-enterprise.list
安装
export LC_ALL=en_US.UTF-8
apt update && apt -y install git && git clone https://github.com/ivanhao/pvetools.git
启动工具（cd到目录，启动工具）
cd ~/pvetools​
./pvetools.sh
卸载
删除下载的pvetools目录即可
基本的设置，非常方便，如配置邮箱通知等

```
## 开启IOMMU
 此步骤几乎为必须
 ```json0
启动内核IOMMU支持
vim /etc/default/grub
GRUB_CMDLINE_LINUX_DEFAULT="quiet"
改为
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt pcie_acs_override=downstream video=efifb:off"

## 优化显卡待机

PVE 宿主机对显卡的待机很不友好，所以在 grub 添加如下 4 个关闭显卡的参数：

video=vesafb:off	禁用 vesa 启动显示设备
video=efifb:off	禁用 efi 启动显示设备
video=simplefb:off	5.15 内核开始直通可能需要这个参数
initcall_blacklist=sysfb_init	部分 A 卡如 RX580 直通异常可能需要这个参数

GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt textonly nomodeset nofb pci=noaer pcie_acs_override=downstream,multifunction video=vesafb:off video=efifb:off video=simplefb:off initcall_blacklist=sysfb_init"
```
update-grub
更新一下 grub 配置文件再重启


#  网友web界面修改-by恩山

- [来源链接](https://www.right.com.cn/forum/thread-6754687-1-1.html)

脚本自动检测：一键给PVE增加温度和cpu频率显示，NVME，机械固态硬盘信息

使用方法：
可以一键执行下面：
```json5
(curl -Lf -o /tmp/temp.sh https://raw.githubusercontent.com/a904055262/PVE-manager-status/main/showtempcpufreq.sh || curl -Lf -o /tmp/temp.sh https://mirror.ghproxy.com/https://raw.githubusercontent.com/a904055262/PVE-manager-status/main/showtempcpufreq.sh) && chmod +x /tmp/temp.sh && /tmp/temp.sh remod
```

没有显示功耗的，请执行下面的命令安装依赖，请确保安装成功，就是最后的一行的输出，必须为 “成功!” 才表示安装成功了。
```json5
apt update ; apt install linux-cpupower && modprobe msr && echo msr > /etc/modules-load.d/turbostat-msr.conf && chmod +s /usr/sbin/turbostat && echo 成功！
```

如果你已经用别人的脚本之类的修改过页面，请先用下面命令先回复官方设置之后，才可以运行本脚本：

```json5
apt update
apt install pve-manager  proxmox-widget-toolkit  --reinstall
rm -f /usr/share/perl5/PVE/API2/Nodes.pm*bak
rm -f /usr/share/pve-manager/js/pvemanagerlib.js*bak
rm -f /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js*bak
```
另外：每次pve升级之后都需要执行一次脚本，因为升级后PVE会自己还原文件
