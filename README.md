### shell脚本：
一键给PVE添加CPU频率和各种温度显示，NVME硬盘，机械硬盘，固态硬盘信息：

- [代码链接](https://github.com/7568168/PVE-manager-status/tree/stcf)

![image](https://github.com/7568168/PVE-manager-status/blob/main/PVE效果图.png)


## pve用root登陆运行

```json5
(curl -Lf -o /tmp/temp.sh https://raw.githubusercontent.com/7568168/PVE-manager-status/stcf/stcf || curl -Lf -o /tmp/temp.sh https://cgraw.pages.dev/https://raw.githubusercontent.com/7568168/PVE-manager-status/stcf/stcf) && chmod +x /tmp/temp.sh && /tmp/temp.sh remod
```

## 个人修改

仅sata固态

1.硬盘通电时间计数，年月日

2.硬盘信息和状态分行显示，便于查看



##  来源恩山无线

来源仅供参考

- [来源链接](https://www.right.com.cn/forum/thread-6754687-1-1.html)

脚本自动检测：一键给PVE增加温度和cpu频率显示，NVME，机械固态硬盘信息
1.理论上适合任何设备
2.自动适配传感器数据
3.自动检测NVME硬盘数量
4.自动检测机械，固态硬盘数量
5.自动检测CPU核心数量
  
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

## （更新后顶部显示可通过搜索链接，然后注释去除）

更新：
2023.1.10
理论上适合任意设备！
添加nvme硬盘信息显示
2023.1.11
效果优化
加入固态硬盘和机械硬盘信息
CPU频率加入调速器，最大最小频率
NVME加入读写数据量显示
修复bug
2023.1.12
细节优化
2023.1.14
细节优化
2023.1.15
修正右栏高度和左栏一致，解决双栏浮动布局错位问题
2023.1.18
优化布局，修正温度获取bug
2023.4.14
添加NVME健康度和0E显示
2023.9.4
修复一些设备因为温度问题无限转圈圈
2023.9.5
增加CPU功耗显示
2023.12.13
机械硬盘休眠状态不去获取SMART
