如果你想给电视盒子装个 Linux 来玩玩, 你大概只有一种选择 -- Armbian. 这是专为 ARM 设备，基于 Debian 或 Ubuntu 设计的发行版.

如果你只想搭个服务器，这两个发行版已经足够了. 但是如果你想当迷你桌面主机用, 那就不一样了.
Ubuntu 最新的版本在 ARM 平台上的桌面体验可谓一言难尽, 从 22.04 开始, Ubuntu 开始将一些桌面必须软件放入自家的 Snap 商店当中，众所周知这是个十分傻逼的玩意，而且它在 ARM 平台完全没法工作，甚至没法正常安装软件包.
而 Debian(Sid)，它的桌面体验可能要***相对***差一些. 再说，并不是人人都喜欢Deb系的发行版.

所以, 就有了这篇文章. 让我们来安装一个其他的发行版吧！

### 准备阶段
首先，我们需要选择哪个发行版呢？桌面体验优秀这种要求不必多说，我们还需要考虑它对 ARM 平台支持情况如何，为 ARM 平台准备的软件多不多等等.
除了 Debian 系的发行版，Arch 系对 ARM 平台的支持也很不错.

我们能选的就是 Arch 系最好的两个发行版，Arch Linux 本尊和 Manjaro. 它们对 ARM 的支持情况不相上下，都十分不错.
在这里我选择 Arch Linux. 当然, 我也会顺带说一下 Manjaro 的安装.

HK1 Box 是基于 Amlogic 平台, Armbian社区已经不再支持 Amlogic 平台了. 所以首先我们要去 [ophub/amlogic-s9xxx-armbian](https://github.com/ophub/amlogic-s9xxx-armbian/) 下载我们机型的镜像作为底包.
直接前往这个项目的 Release 界面, 找到像这样 `Armbian_Aml_jammy_xx.xx.xxxx` 的 Release 项，下载你机型对应的包. 比如我的 HK1Box 需要选择 `Armbian_22.11.0_Aml_s905x3_jammy_5.15.72_server_xxxx.xx.xx.img.gz`.

找到底包后, 我们还需要找到 Arch Linux ARM 的 rootfs 包. 在中国, 我们需要去[镜像站](https://mirrors.tuna.tsinghua.edu.cn/archlinuxarm/os/)找.
我们选择 `Odroid C2` 的包 `ArchLinuxARM-odroid-c2-latest.tar.gz` , 这款设备的硬件和我们的盒子很相似. 对于其他的 Amlogic 平台设备也可以选择这个包.

Manjaro 提供的是镜像, 直接去 https://manjaro.org/download/, 往下翻一翻, 就能看到 Manjaro ARM.
设备选择 `Odroid C4`, 然后下载一个你喜欢的桌面的镜像, 或者干脆用不带桌面的.

下载上面提到的文件, 这样, 我们需要的东西就准备完毕了.

### 安装系统
然后我们需要先把底包写入到U盘. 如果你有 [balenaEtcher](https://www.balena.io/etcher/), 可以直接用它写入. 否则：
```bash
gunzip Armbian_22.11.0_Aml_s905x3_jammy_5.15.72_server_xxxx.xx.xx.img.gz # 使用 gunzip 解压镜像, 你需要把文件名替换为你自己的
dd if=Armbian_22.11.0_Aml_s905x3_jammy_5.15.72_server_xxxx.xx.xx.img of=/dev/sdx # 使用 dd 写入， 你需要把文件名和设备路径替换为你自己的
```
接着挂载U盘上名为 `ROOTFS` 的分区, 把里面的这些文件和文件夹拷贝出来：
```
/etc/amlogic_model_database.conf
/etc/fstab
/usr/sbin/armbian-install
/usr/lib/u-boot
/usr/lib/modules
/usr/lib/firmware
```
这些文件之后我们还需要. 然后我们需要清空 ROOTFS, 进入挂载目录执行：
```bash
sudo rm -rf *
```
接着，不同的地方就来了.
- Arch
清空之后, 把下载来的 Arch Linux ARM rootfs 包解压到 ROOTFS 分区.

- Manjaro
使用 `kpartx` 之类的软件挂载镜像, 把 `ROOT_MNJRO` 分区里所有内容复制到 ROOTFS 分区. 可以用 `tar` 或 `rsync` 来复制, 直接 `cp` 可能会导致权限问题.

然后, 删除 rootfs 里的以下文件:
```
/boot/*
/etc/fstab
/usr/lib/modules
/usr/lib/firmware
```
`/boot` 让它空着就可以, 另外的三个用我们刚才从 Armbian 中复制出的相应文件或文件夹替换.
然后把其他刚刚拷贝出的文件复制也到相应位置.

接着就是修改启动设置, [ophub/amlogic-s9xxx-armbian](https://github.com/ophub/amlogic-s9xxx-armbian/) 有详细说明, 我就不提了.

### 进入系统

完成上面一系列操作, 配置好启动设置后, 我们就能直接进入系统了.
使用默认用户名 `root` 和密码 `root` 登录.(Manjaro 的密码不是这个)
***这十分不安全, 你进入系统后应该立刻修改**

***如果你安装的是 Manjaro, 不用看下面这一段. Manjaro 的镜像都是配置好的, 如果你选了带桌面的镜像, 那应该会直接进入桌面.**

在我们开始安装各种软件之前, 我们需要先执行一下以下的命令:

***以下操作默认你已经正常连接网络.**

- 初始化 pacman 密钥环
```bash
pacman-key --init
pacman-key --populate
```

之后我们需要换源, 这个操作就不讲了.

- 更新下当前的系统
```bash
pacman -Syyu
```

- 安装一些缺失的重要软件:
```bash
pacman -S sudo uboot-tools wget git
```

如果你想把系统安装到 emmc 的话, 我推荐你现在就安装:
```bash
armbian-install
```

这样, 一个 Arch Linux ARM 基本系统就准备完毕了. 你可以安装一个桌面环境, 或者说其他你需要的软件. 开始享受吧！

### 一些有用的东西

- 更新内核
源里的内核在这肯定是不能用了, 你需要自己编译一个.
怎么编译内核在[这里](https://github.com/ophub/amlogic-s9xxx-armbian/tree/main/compile-kernel)有讲解，我就不说了.
和在一般电脑上的 Arch Linux 一样, 安装内核后, 我们需要执行一下 `mkinitcpio` 来生成 Initrd 镜像, 但在这里, 它还不够. 
我们需要这样做: 
```bash
sudo mkinitcpio -g -k (你的内核版本) #生成 Initrd 镜像, 标准格式. 你需要指定你的内核版本.
sudo cp /boot/uInitrd /boot/uInitrd-bkp #备份当前的 uInitrd 镜像
sudo mkimage -A arm64 -O linux -T ramdisk -C gzip -n uInitrd -d (生成的镜像位置) /boot/uInitrd #把新生成的 Initrd 镜像转换成 u-Boot 格式, 这样才能正常启动.
```
然后修改下启动设置, 把内核那一条指向新内核就行了.

---

- AUR
我们亲爱的 AUR 管理器在 arm 平台也是完全可用的
paru (Rust): ~~这玩意我在盒子上足足编译了32分钟~~
```bash
git clone https://aur.archlinux.org/paru.git
cd paru
makepkg -si
cd ..
rm -rf paru
```
yay (go):
```bash
git clone https://aur.archlinux.org/yay.git
cd git
makepkg -si
cd ..
rm -rf yay
```
AUR 对 arm 的支持基本可以分为3种
- 通用的或者现场下载源码编译的，一般都能用
- 重打包二进制, 有 arm 二进制的, 稍微改一下 PKGBUILD 一般都能用
- 重打包二进制, 没有 arm 二进制的, 那就没办法了
在手动打包时, 可以使用 `makepkg -A`. 这样会无视架构要求, 就能成功打包了.

---

- 浏览器
Edge, Chrome 等都只为 amd64 打包. 你可以选择 Firefox, Chromium 等.


