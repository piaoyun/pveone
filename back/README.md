
### 存档的脚本 - 勿要使用 - 全是BUG

#### pve6.4升级为最新的pve7.x

自测中，勿要使用，未完成

```
curl -L https://raw.githubusercontent.com/spiritLHLS/pve/main/pve6_to_pve7.sh -o pve6_to_pve7.sh && chmod +x pve6_to_pve7.sh && bash pve6_to_pve7.sh
```

### 加载系统模板

- 加载KVM或LXC模板到PVE的ISO/CT列表中(debian11，ubuntu20)
- 加载完成后请web端查看 pve > local(pve) > ISO Images/CT Templates 刷新一下记录，直接去创建虚拟机是可能看不到已加载的

```
curl -L https://raw.githubusercontent.com/spiritLHLS/pve/main/install_iso.sh -o install_iso.sh && chmod +x install_iso.sh && bash install_iso.sh
```

### 替换qcow2

```
curl -L https://raw.githubusercontent.com/spiritLHLS/pve/main/back/rebuild.sh -o rebuild.sh && chmod +x rebuild.sh && bash rebuild.sh
```

```
guestfish -a xxx -i -c "cat /etc/ssh/sshd_config"
```

```
qm exec 虚拟机ID /bin/bash
```

```
qemu-img convert -p -O qcow2 /var/lib/vz/images/152/vm-152-disk-0.raw /root/xxx
```

```
dd if=/dev/zero of=/null.dat
rm -f /null.dat
```

```
cp --sparse=always /var/lib/vz/images/152/vm-152-disk-0.raw /var/lib/vz/images/152/vm-152-disk-0-new.raw
qemu-img convert -p -O qcow2 /var/lib/vz/images/152/vm-152-disk-0-new.raw /root/xxx
qemu-img convert -c -O qcow2 /root/xxxxx /root/xxxxx
```

### 卸载所有虚拟机

```
for vmid in $(qm list | awk '{if(NR>1) print $1}'); do qm stop $vmid; qm destroy $vmid; rm -rf /var/lib/vz/images/$vmid*; done
iptables -t nat -F
iptables -t filter -F
service networking restart
systemctl restart networking.service
rm -rf vm*
```

### 卸载PVE整体环境

尝试失败，因为已自动替换过为PVE的内核，如需卸载需要先替换为原生内核

```
curl -L https://raw.githubusercontent.com/spiritLHLS/pve/main/back/uninstallpve.sh -o uninstallpve.sh && chmod +x uninstallpve.sh && bash uninstallpve.sh
```

### PVE基础安装说明

- 配置完毕需要重启系统加载新内核
- 重启系统前推荐挂上[nezha探针](https://github.com/naiba/nezha)方便在后台不通过SSH使用命令行，避免SSH可能因为商家奇葩的预设导致重启后root密码丢失

# 目录

* [系统要求与配置](#系统要求与配置)
    * [各种要求](#各种要求)
    * [检测硬件环境](#检测硬件环境)
    * [PVE基础安装说明](#PVE基础安装说明)
    * [一键安装PVE](#一键安装PVE)
    * [预配置环境](#预配置环境)
    * [自动配置宿主机的网关](#自动配置宿主机的网关)
* [一键生成KVM虚拟化的NAT服务器](#一键生成KVM虚拟化的NAT服务器)
    * [单独生成KVM虚拟化的VM](#单独生成KVM虚拟化的VM)
    * [单个生成的使用方法](#单个生成的使用方法)
    * [示例](#示例)
    * [删除示例](#删除示例)
    * [相关qcow2镜像](#相关qcow2镜像)
* [批量开设NAT的KVM虚拟化的VM](#批量开设NAT的KVM虚拟化的VM)
    * [使用方法](#使用方法)
    * [删除所有虚拟机](#删除所有虚拟机)
    * [注意事项](#注意事项)
* [一键开设独立IPV4地址的虚拟机](#一键开设独立IPV4地址的虚拟机)
    * [方法](#方法)
    * [删除独立IPV4的虚拟机](#删除独立IPV4的虚拟机)
* [一键生成单个CT也就是LXC虚拟化的NAT容器](#一键生成单个CT也就是LXC虚拟化的NAT容器)
    * [如何使用](#如何使用)
    * [CT示例](#CT示例)
    * [删除示例CT](#删除示例CT)
* [批量开设NAT的LXC虚拟化的CT容器](#批量开设NAT的LXC虚拟化的CT容器)
    * [一键命令](#一键命令)
    * [删除所有CT](#删除所有CT)
* [致谢](#致谢)

### 系统要求与配置

#### 各种要求

建议debian在使用前尽量使用最新的系统

非debian11可使用 [debian一键升级](https://github.com/spiritLHLS/one-click-installation-script#%E4%B8%80%E9%94%AE%E5%8D%87%E7%BA%A7%E4%BD%8E%E7%89%88%E6%9C%ACdebian%E4%B8%BAdebian11) 来升级系统

当然不使用最新的debian系统也没问题，只不过得不到官方支持。只适配Debian系统(非Debian无法通过APT源安装，官方只给了Debian的镜像，其他系统只能使用ISO安装)

- 系统要求：Debian 8+ (建议debian11而不是debian12，因为后者是beta版本，debian11安装的才是稳定版)
- 最低的硬件要求：2核2G内存x86_64架构服务器硬盘至少20G，内存开点swap免得机器炸了[开SWAP点我跳转](https://github.com/spiritLHLS/addswap)
- 可开KVM的硬件要求：VM-X或AMD-V支持-(部分VPS和全部独服支持)
- 如果硬件需求不满足，可使用LXD批量开LXC的[跳转](https://github.com/spiritLHLS/lxc)

**遇到选项不会选的可无脑回车安装，所有脚本内置国内外IP自动判断，使用的是不同的安装源与配置文件，有使用CDN加速镜像下载**

#### 检测硬件环境

- 本仓库脚本执行前务必执行本脚本检测环境，如果不符合安装PVE的要求则无法使用后续的脚本
- 检测硬件配置是否满足最低要求
- 检测硬件环境是否可嵌套虚拟化KVM类型的服务器
- 检测系统环境是否可嵌套虚拟化KVM类型的服务器
- 不可嵌套虚拟化KVM类型的服务器也可以开LXC虚拟化的服务器，但不推荐安装PVE，不如使用[LXD](https://github.com/spiritLHLS/lxc)

```
bash <(wget -qO- --no-check-certificate https://raw.githubusercontent.com/spiritLHLS/pve/main/scripts/check_kernal.sh)
```

国内：

```
bash <(wget -qO- --no-check-certificate https://ghproxy.com/https://raw.githubusercontent.com/spiritLHLS/pve/main/scripts/check_kernal.sh)
```

#### PVE基础安装说明

- 安装的是当下apt源最新的PVE
- 比如debian10则是pve6.4，debian11则是pve7.x，debian12则是pve8.x(建议debian11而不是debian12，因为后者是beta版本，debian11安装的才是稳定版)
- /etc/hosts文件修改(修正商家hostname设置错误以及新增PVE所需的内容)
- 已设置```/etc/hosts```为只读模式，避免重启后文件被覆写，如需修改请使用```chattr -i /etc/hosts```取消只读锁定，修改完毕请执行```chattr +i /etc/hosts```只读锁定
- 检测```/etc/cloud/cloud.cfg```如果发现```preserve_hostname```是```false```，则改为```true```，同上，也用chattr命令进行了文件锁定避免重启覆盖设置
- 检测是否为中国IP，如果为中国IP使用清华镜像源，否则使用官方源
- 安装PVE开虚拟机需要的必备工具包
- 替换apt源中的企业订阅为社区源
- 打印查询Linux系统内核和PVE内核是否已安装
- 检测```/etc/resolv.conf```是否为空，为空则设置检测```8.8.8.8```的开机自启添加DNS的systemd服务
- 新增PVE的APT源链接后，下载PVE并打印输出登陆信息

#### 一键安装PVE

```
curl -L https://raw.githubusercontent.com/spiritLHLS/pve/main/scripts/install_pve.sh -o install_pve.sh && chmod +x install_pve.sh && bash install_pve.sh
```

国内：

```
curl -L https://ghproxy.com/https://raw.githubusercontent.com/spiritLHLS/pve/main/scripts/install_pve.sh -o install_pve.sh && chmod +x install_pve.sh && bash install_pve.sh
```

### 预配置环境

- 创建资源池mypool
- 移除订阅弹窗
- 尝试开启硬件直通
- 检测AppArmor模块并试图安装
- 重启系统前推荐挂上[nezha探针](https://github.com/naiba/nezha)方便在后台不通过SSH使用命令行，避免SSH可能因为商家奇葩的预设导致重启后root密码丢失
- **执行完毕建议等待几分钟后再重启服务器**，执行```reboot```前需要等待后台任务执行完毕，一些宿主机的系统apt命令执行很慢，得等一会才能执行完毕

```
bash <(wget -qO- --no-check-certificate https://raw.githubusercontent.com/spiritLHLS/pve/main/scripts/build_backend.sh)
```

国内

```
bash <(wget -qO- --no-check-certificate https://ghproxy.com/https://raw.githubusercontent.com/spiritLHLS/pve/main/scripts/build_backend.sh)
```

### 自动配置宿主机的网关

- **使用前请保证重启过服务器且此时PVE能正常使用WEB端再执行，重启机器后不要立即执行此命令，待WEB端启动成功后至少等5分钟再执行本命令**
- **这一步是最容易造成SSH断开的，原因是未等待PVE内核启动就修改网络会造成设置冲突，所以至少等几分钟待内核启动也就是WEB端启动成功后再执行**
- 创建vmbr0，母鸡允许addr和gateway为内网IP或外网IP，已自动识别
- vmbr0创建支持纯IPV4或双栈服务器，自动识别IPV4地址和IPV6地址，自动识别对应的IP区间
- 创建vmbr1(NAT网关)
- 开NAT虚拟机时网关（IPV4）使用```172.16.1.1```，IPV4/CIDR使用```172.16.1.x/24```，这里的x不能是1，当然如果后续使用本套脚本无需关注这点细枝末节的东西
- 想查看完整设置可以执行```cat /etc/network/interfaces```查看
- 加载iptables并设置回源且允许NAT端口转发

```
bash <(wget -qO- --no-check-certificate https://raw.githubusercontent.com/spiritLHLS/pve/main/scripts/build_nat_network.sh)
```

国内

```
bash <(wget -qO- --no-check-certificate https://ghproxy.com/https://raw.githubusercontent.com/spiritLHLS/pve/main/scripts/build_nat_network.sh)
```

## 一键生成KVM虚拟化的NAT服务器

使用前记得**执行本仓库的第一个个命令，那个检测硬件环境的命令**，展示如下

![图片](https://user-images.githubusercontent.com/103393591/231160050-79945d07-b3d0-4e8d-9315-74e4fbb24f9d.png)

查询如上的只需使用下面的一键脚本自动创建虚拟机即可，无需手动再修改WEB端设置

![图片](https://user-images.githubusercontent.com/103393591/231160070-c317607c-8b0c-4aa4-bfa2-e75ec6626b24.png)

查询如上的在使用后续脚本创建了虚拟机后，**可能**需要手动修改WEB端设置，需要关闭对应每个虚拟机的硬件嵌套虚拟化，如下图

![图片](https://user-images.githubusercontent.com/103393591/231160449-82911a57-4b49-47ec-8fad-2100c6059017.png)

先停止虚拟机再修改，修改完后再开机才能使用NOVNC，不关闭**可能**导致这个虚拟机有BUG无法使用

如果强行安装PVE开KVM，启动不了的也可以关闭这个选项试试能不能启动虚拟机

### 单独生成KVM虚拟化的VM

- 自动开设NAT服务器，默认使用Debian10镜像，因为该镜像占用最小
- 可在命令中自定义需要使用的镜像，这里有给出配置好的镜像，镜像自带空间设置是2~10G硬盘，日常使用**至少10G以上**即可，除非某些镜像开不起来再增加硬盘大小
- 可在命令中指定存储盘位置，默认不指定时为local盘即系统盘，可指定为PVE中显示的挂载盘
- 自定义内存大小推荐512MB内存，需要注意的是母鸡内存记得开点swap免得机器炸了[开SWAP点我跳转](https://github.com/spiritLHLS/addswap)
- 自动进行内外网端口映射，含22，80，443端口以及其他25个内外网端口号一样的端口
- 生成后需要等待一段时间虚拟机内部的cloudinit配置好网络以及登陆信息，大概需要5分钟
- 虚拟机的相关信息将会存储到对应的虚拟机的NOTE中，可在WEB端查看

```
curl -L https://raw.githubusercontent.com/spiritLHLS/pve/main/scripts/buildvm.sh -o buildvm.sh && chmod +x buildvm.sh
```

国内

```
curl -L https://ghproxy.com/https://raw.githubusercontent.com/spiritLHLS/pve/main/scripts/buildvm.sh -o buildvm.sh && chmod +x buildvm.sh
```

#### 单个生成的使用方法

- 系统支持：详见 [跳转](https://github.com/spiritLHLS/Images/releases/tag/v1.0) 中列出的系统，使用时只需写文件名字，不需要.qcow2尾缀
- **注意这里的用户名不能是纯数字，会造成cloudinit出问题，最好是纯英文或英文开头**

```
./buildvm.sh VMID 用户名 密码 CPU核数 内存 硬盘 SSH端口 80端口 443端口 外网端口起 外网端口止 系统 存储盘
```

#### 示例

测试开一个NAT服务器

以下示例开设VMID为102的虚拟机，用户名是test1，密码是1234567，CPU是1核，内存是512MB，硬盘是10G，SSH端口是40001，80端口是40002，443端口是40003

同时内外网映射端口一致的区间是50000到50025，系统使用的是ubuntu20，使用宿主机的存储盘是local盘

```
./buildvm.sh 102 test1 1234567 1 512 10 40001 40002 40003 50000 50025 ubuntu20 local
```

开设完毕可执行

```
cat vm102
```

登陆后可使用```sudo -i```切换到root用户

查看信息

#### 删除示例

删除端口映射，删除测试的虚拟机和log文件

```
qm stop 102
qm destroy 102
iptables -t nat -F
iptables -t filter -F
service networking restart
systemctl restart networking.service
rm -rf vm102
```

#### 相关qcow2镜像

- 已预安装开启cloudinit，开启SSH登陆，预设值SSH监听V4和V6的22端口，开启允许密码验证登陆，开启允许ROOT登陆

https://github.com/spiritLHLS/Images/releases/tag/v1.0

### 批量开设NAT的KVM虚拟化的VM

- **初次使用前需要保证当前PVE未有任何虚拟机未有进行任何端口映射，否则可能出现BUG**
- **开设前请使用screen挂起执行，避免批量开设时间过长，SSH不稳定导致中间执行中断，推荐使用PVE自带的Shell操作母鸡**
- 可多次运行批量生成VM，但需要注意的是母鸡内存记得开点swap免得机器炸了[开SWAP点我跳转](https://github.com/spiritLHLS/addswap)
- 自动开设NAT服务器，选项留空默认使用debian11镜像，可自定义使用镜像名字，支持的系统名字详见上文的镜像仓库
- 自动进行内外网端口映射，含22，80，443端口以及其他25个内外网端口号一样的端口
- 生成后需要等待一段时间虚拟机内部的cloudinit配置好网络以及登陆信息，大概需要5分钟，每个虚拟机创建之间有间隔等待60秒避免突发性能不足
- 默认批量开设的虚拟机网络配置为：22，80，443端口及一个25个端口区间的内外网映射
- 可自定义批量开设的核心数，内存大小，硬盘大小，使用宿主机哪个存储盘，记得自己计算好空闲资源开设
- 虚拟机的相关信息将会存储到对应的虚拟机的NOTE中，可在WEB端查看

#### 使用方法

```
curl -L https://raw.githubusercontent.com/spiritLHLS/pve/main/scripts/create_vm.sh -o create_vm.sh && chmod +x create_vm.sh && bash create_vm.sh
```

国内

```
curl -L https://ghproxy.com/https://raw.githubusercontent.com/spiritLHLS/pve/main/scripts/create_vm.sh -o create_vm.sh && chmod +x create_vm.sh && bash create_vm.sh
```

开设完毕可执行

```
cat vmlog
```

查看信息

登陆后可使用```sudo -i```切换到root用户

#### 删除所有虚拟机

删除所有nat的端口映射并重启网络，删除所有虚拟机和log文件

```
for vmid in $(qm list | awk '{if(NR>1) print $1}'); do qm stop $vmid; qm destroy $vmid; rm -rf /var/lib/vz/images/$vmid*; done
iptables -t nat -F
iptables -t filter -F
service networking restart
systemctl restart networking.service
rm -rf vmlog
```

### 注意事项

PVE修改虚拟机配置前都得停机先，再修改配置，修改完再启动，免得出现配置重载错误

## 一键开设独立IPV4地址的虚拟机

- **使用前需要保证当前宿主机的IP段带了至少2个IP，且有空余的IP未配置，该空余的IP未绑定宿主机**
- **开设前请使用screen挂起执行，避免开设时间过长，SSH不稳定导致中间执行中断**
- 自动检测可用的IP区间，通过ping检测空余可使用的IP，选取其中之一绑定到虚拟机上
- 系统的相关信息将会存储到对应的虚拟机的NOTE中，可在WEB端查看

### 方法

下载脚本

```
curl -L https://raw.githubusercontent.com/spiritLHLS/pve/main/scripts/buildvm_extraip.sh -o buildvm_extraip.sh && chmod +x buildvm_extraip.sh
```

国内下载

```
curl -L https://ghproxy.com/https://raw.githubusercontent.com/spiritLHLS/pve/main/scripts/buildvm_extraip.sh -o buildvm_extraip.sh && chmod +x buildvm_extraip.sh
```

示例创建

```
./buildvm_extraip.sh VMID 用户名 密码 CPU核数 内存大小以MB计算 硬盘大小以GB计算 系统 存储盘
```

```
./buildvm_extraip.sh 152 test1 1234567 1 1024 10 ubuntu20 local
```

上述命令意义为开设一个带独立IPV4地址的虚拟机，VMID是152，用户名是test1，密码是1234567，CPU是1核，内存是1024MB，硬盘是10G，系统是Ubuntu20，存储盘是local盘也就是系统盘

### 删除独立IPV4的虚拟机

删除示例

```
qm stop 152
qm destroy 152
rm -rf vm152
```

## 一键生成单个CT也就是LXC虚拟化的NAT容器

LXC虚拟化的容器-自带内外网映射

- **初次使用前需要保证当前PVE未有任何虚拟机未有进行任何端口映射，否则可能出现BUG**
- **开设前请使用screen挂起执行，避免批量开设时间过长，SSH不稳定导致中间执行中断，推荐使用PVE自带的Shell操作母鸡**
- 自动开设NAT服务器，默认使用Debian11镜像，也可自定义系统
- 自动进行内外网端口映射，含22，80，443端口以及其他25个内外网端口号一样的端口
- 生成后需要等待一段时间虚拟机内部配置好网络以及登陆信息，大概需要3分钟
- 默认开设的虚拟机网络配置为：22，80，443端口及一个25个端口区间的内外网映射
- 可自定义开设的核心数，内存大小，硬盘大小，使用宿主机哪个存储盘，记得自己计算好空闲资源开设
- 可在命令中指定存储盘位置，默认不指定时为local盘即系统盘，可指定为PVE中显示的挂载盘
- 开设的CT默认已启用SSH且允许root登陆，且已设置支持使用docker的嵌套虚拟化
- 容器的相关信息将会存储到对应的容器的NOTE中，可在WEB端查看

```
curl -L https://raw.githubusercontent.com/spiritLHLS/pve/main/scripts/buildct.sh -o buildct.sh && chmod +x buildct.sh
```

国内

```
curl -L https://ghproxy.com/https://raw.githubusercontent.com/spiritLHLS/pve/main/scripts/buildct.sh -o buildct.sh && chmod +x buildct.sh
```

#### 如何使用

- 系统支持：debian10，debian11，ubuntu18，ubuntu20，ubuntu22，centos8，almalinux9等
- 系统参数一律是小写的系统名字拼接版本号，具体可执行```pveam available --section system```查看可用的系统名字和版本号(注意脚本使用的参数只有小写的英文系统名字拼接版本号)
- 其他系统可能支持可能不支持，自行测试
- 默认用户名是root

```
./buildct.sh CTID 密码 CPU核数 内存 硬盘 SSH端口 80端口 443端口 外网端口起 外网端口止 系统 存储盘
```

#### CT示例

测试开一个NAT的LXC虚拟化的容器

以下示例开设CTID为102的容器，用户名是root，密码是1234567，CPU是1核，内存是512MB，硬盘是5G，SSH端口是20001，80端口是20002，443端口是20003

同时内外网映射端口一致的区间是30000到30025，系统使用的是debian11，使用宿主机的存储盘是local盘

```
./buildct.sh 102 1234567 1 512 5 20001 20002 20003 30000 30025 debian11 local
```

开设完毕可执行

```
cat ct102
```

查看信息，或在web端的note查看

#### 删除示例CT

以下命令将删除示例CT和log文件，删除所有的nat的端口映射并重启网络

```
pct stop 102
pct destroy 102
rm -rf ct102
iptables -t nat -F
iptables -t filter -F
service networking restart
systemctl restart networking.service
```

## 批量开设NAT的LXC虚拟化的CT容器

- 自带内外网映射
- 可重复运行继承配置

#### 一键命令

- **初次使用前需要保证当前PVE未有任何CT容器未有进行任何端口映射，否则可能出现BUG**
- **开设前请使用screen挂起执行，避免批量开设时间过长，SSH不稳定导致中间执行中断，推荐使用PVE自带的Shell操作母鸡**
- 可多次运行批量生成CT容器，但需要注意的是母鸡内存记得开点swap免得机器炸了[开SWAP点我跳转](https://github.com/spiritLHLS/addswap)
- 每个容器创建之间有间隔等待60秒避免突发性能不足
- 可自定义批量开设的核心数，内存大小，硬盘大小，使用宿主机哪个存储盘，记得自己计算好空闲资源开设
- 开设的CT默认已启用SSH且允许root登陆，且已设置支持使用docker的嵌套虚拟化
- 容器的相关信息将会存储到对应的容器的NOTE中，可在WEB端查看

```
curl -L https://raw.githubusercontent.com/spiritLHLS/pve/main/scripts/create_ct.sh -o create_ct.sh && chmod +x create_ct.sh && bash create_ct.sh
```

国内

```
curl -L https://ghproxy.com/https://raw.githubusercontent.com/spiritLHLS/pve/main/scripts/create_ct.sh -o create_ct.sh && chmod +x create_ct.sh && bash create_ct.sh
```

开设完毕可执行

```
cat ctlog
```

查看信息


#### 删除所有CT

以下命令将删除所有CT和所有的log文件，删除所有nat的端口映射并重启网络

```
pct list | awk 'NR>1{print $1}' | xargs -I {} sh -c 'pct stop {}; pct destroy {}'
rm -rf ct*
iptables -t nat -F
iptables -t filter -F
service networking restart
systemctl restart networking.service
```

## 致谢

https://blog.ilolicon.com/archives/615

https://github.com/Ella-Alinda/somescripts/blob/main/nat.sh

https://pve.proxmox.com/pve-docs/qm.1.html

https://down.idc.wiki/Image/realServer-Template/

https://mirrors.tuna.tsinghua.edu.cn/proxmox/

https://github.com/roacn/pve/blob/main/pve.sh

https://github.com/spiritLHLS/lxc

感谢 [@Ella-Alinda](https://github.com/Ella-Alinda) 提供的PVE指导


