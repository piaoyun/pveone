# PVE

[![Hits](https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https%3A%2F%2Fgithub.com%2FspiritLHLS%2Fpve&count_bg=%2379C83D&title_bg=%23555555&icon=&icon_color=%23E7E7E7&title=hits&edge_flat=false)](https://hits.seeyoufarm.com)

感谢 Proxmox VE 的免费订阅支持

如果有未适配的商家或机器欢迎联系[@spiritlhl_bot](https://t.me/spiritlhl_bot)，有空会尝试支持一下

待开发内容：

- 国内宿主机开设容器/虚拟机时，由于网络与官方的包管理源链接非常不通畅，需要使用镜像链接替换或CDN加速整体链接
- KVM/LXC模板加载部分自定义的限制，避免机器用于滥用发包
- LXC模板构建自定义的模板提前初始化好部分内容，避免原始模板过于干净导致初始化时间过长

## 更新

2023.10.03

- 修复开设带IPV6地址的虚拟机时，网关顺序错配的问题
- 更新开设出的虚拟机的nameserver和searchdomain设置，避免某些机器在解析域名时出错

[更新日志](CHANGELOG.md)

## 说明文档

国内(China)：

[virt.spiritlhl.net](https://virt.spiritlhl.net/)

国际(Global)：

[www.spiritlhl.net](https://www.spiritlhl.net/)

说明文档中 Proxmox VE 分区内容

[https://github.com/oneclickvirt/kvm_images](https://github.com/oneclickvirt/kvm_images) 为对应虚拟机镜像仓库

## Stargazers over time

[![Stargazers over time](https://starchart.cc/spiritLHLS/pve.svg)](https://starchart.cc/spiritLHLS/pve)

## 友链

VPS融合怪测评脚本

https://github.com/spiritLHLS/ecs
