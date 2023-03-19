**English** | [中文](https://p3terx.com/archives/build-openwrt-with-github-actions.html)

# Actions-OpenWrt

[![LICENSE](https://img.shields.io/github/license/mashape/apistatus.svg?style=flat-square&label=LICENSE)](https://github.com/P3TERX/Actions-OpenWrt/blob/master/LICENSE)
![GitHub Stars](https://img.shields.io/github/stars/P3TERX/Actions-OpenWrt.svg?style=flat-square&label=Stars&logo=github)
![GitHub Forks](https://img.shields.io/github/forks/P3TERX/Actions-OpenWrt.svg?style=flat-square&label=Forks&logo=github)

A template for building OpenWrt with GitHub Actions

## Usage

- Click the [Use this template](https://github.com/P3TERX/Actions-OpenWrt/generate) button to create a new repository.
- Generate `.config` files using [Lean's OpenWrt](https://github.com/coolsnowwolf/lede) source code. ( You can change it through environment variables in the workflow file. )
- Push `.config` file to the GitHub repository.
- Select `Build OpenWrt` on the Actions page.
- Click the `Run workflow` button.
- When the build is complete, click the `Artifacts` button in the upper right corner of the Actions page to download the binaries.

## Tips

- It may take a long time to create a `.config` file and build the OpenWrt firmware. Thus, before create repository to build your own firmware, you may check out if others have already built it which meet your needs by simply [search `Actions-Openwrt` in GitHub](https://github.com/search?q=Actions-openwrt).
- Add some meta info of your built firmware (such as firmware architecture and installed packages) to your repository introduction, this will save others' time.

## Credits

- [Microsoft Azure](https://azure.microsoft.com)
- [GitHub Actions](https://github.com/features/actions)
- [OpenWrt](https://github.com/openwrt/openwrt)
- [Lean's OpenWrt](https://github.com/coolsnowwolf/lede)
- [tmate](https://github.com/tmate-io/tmate)
- [mxschmitt/action-tmate](https://github.com/mxschmitt/action-tmate)
- [csexton/debugger-action](https://github.com/csexton/debugger-action)
- [Cowtransfer](https://cowtransfer.com)
- [WeTransfer](https://wetransfer.com/)
- [Mikubill/transfer](https://github.com/Mikubill/transfer)
- [softprops/action-gh-release](https://github.com/softprops/action-gh-release)
- [ActionsRML/delete-workflow-runs](https://github.com/ActionsRML/delete-workflow-runs)
- [dev-drprasad/delete-older-releases](https://github.com/dev-drprasad/delete-older-releases)
- [peter-evans/repository-dispatch](https://github.com/peter-evans/repository-dispatch)

## License

[MIT](https://github.com/P3TERX/Actions-OpenWrt/blob/main/LICENSE) © [**P3TERX**](https://p3terx.com)


1. 前言
这是一个小白教程，适合新手。 OpenWrt对USB设备支持是比较完善的，挂载USB移动硬盘后，可以做nas/bt下载等，同时samba共享都很方便。
下文配置适用于OpenWrt 18.06之后的版本。

2. 编译选项
官方源代码编译OpenWrt，和USB相关的编译选项：

USB挂载：
Base system -->block-mount/blockd,

USB驱动支持：
Kernel modules --> USB Support -->kmod-usb2/kmod-usb3
Kernel modules --> USB Support -->kmod-usb-core/kmod-usb-storage/kmod-usb-storage-extra

文件系统支持：
Kernel modules --> Filesystem -->kmod-fs-ext4/kmod-fs-msdos/kmod-fs-nfs/kmod-fs-ntfs, kmod-usb-storage-uas, kmod-usb2/usb3

其它：
Utilities-->Disc-->cfdisk/hd-idle/hd-param
Utilities --> smartmontools  (smartctl查看硬盘信息及健康等)

3. 设置
添加上述编译选项并编译安装后，在Openwrt的LUCI界面，“系统”-->“挂载点”， 就可以配置自动挂载硬盘了。

挂载采用UUID方式挂载，这样可以保证重启后，总是挂载上正确的硬盘。对于交换分区，采用文件作为交换分区即可。

1） 先手动挂载 (挂载在/mnt/udisk目录下)
  插上移动硬盘后，查看/dev/应该能看到移动硬盘。（下面假设移动硬盘只有一个分区，OpenWrt系统识别为/dev/sda1）
SSH进路由器，输入如下命令：

# ls /dev/sd*
  /dev/sda   /dev/sda1
# mkdir /mnt/udisk
# mount /dev/sda1   /mnt/udisk
复制代码


2)  在OpenWrt页面配置自动挂载
  在OpenWrt管理页面，“系统”-->"挂载点"，“已挂载文件系统”， 可以看到我们挂载好的硬盘：



在“挂载点”下， 选择“添加”：
在"UUID"处，选择我们的移动硬盘(/dev/sda1, 其它分区类似)， "挂载点"选择自定义， 输入挂载路径“/mnt/udisk”,  点击“启用此挂载点”。 保存应用。



3) 启用交换分区
正常挂载移动硬盘后(假设为/mnt/udisk), 我们在移动硬盘上建立一个文件swapfile, 大小512MB作为交换分区。
SSH进路由器，输入如下命令：

# cd /mnt/udisk
# dd  if=/dev/zero  of=/mnt/udisk/swapfile   bs=1M   count=512

# mkswap     /mnt/udisk/swapfile
# swapon    /mnt/udisk/swapfile
复制代码


4) 配置交换分区
在OpenWrt管理页面，“系统”-->"挂载点"， "交换分区"--> "添加"，
“设备” 处，选择“自定义”， 然后输入我们的上面交换文件： /mnt/udisk/swapfile ,   点击“启用”， 保存应用。





4. 总结

1） 采用 UUID方式挂载，重启路由后，总能自动并且正确挂载移动硬盘。
2） 启用移动硬盘的swapfile文件做交换分区，可以不需要对移动硬盘进行多分区格式。
3） 如果挂载多个分区，同样采用UUID进行自动挂载即可。
4)  关于交换分区，如果需要硬盘自动休眠，那么最好不要挂载硬盘上的交换分区（可能影响硬盘休眠）
5） 查看硬盘信息几个命令：(SSH登录进入路由)
#lsblk

(查看uuid)
#blkid   /dev/sda1

(查看smart信息)
#smartctl  -a  -d  sat  /dev/sda
复制代码

