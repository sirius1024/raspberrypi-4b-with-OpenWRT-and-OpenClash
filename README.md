# 树莓派4B部署OpenWRT作为旁路由并安装开放Open猫Clash

众所周知，本人的教程特别适合小白用户。例如“iTerm2 + Oh My Zsh 打造舒适终端体验”，基本可以做到Copy Paste即用的程度。

本文遵循同样的原则，通过查阅多方资料与实践，为各位献上使用Raspberry Pi 4B通过部署OpenWRT作为旁路由使用，并安装OpenClash作为旁路由代理的教程，在使用时，还请遵守当地法律法规，本着技术探寻的初衷，开阔技术视野，保持独立思考。

话不多说，干中学！**一切都使用官方package和源**，以下为最终效果图：

![image-20250819120215979.png](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819120215979.png)



## 一、准备工作

### 1.1 硬件清单
- **树莓派4B（8G内存版）**：确保散热良好，建议搭配散热片或风扇
  - 我所使用的为8G内存版本，官方已经有编译好的firmware，这会方便很多。如果使用Raspberry Pi 5，还需要自行找可用的OpenWRT固件或自行编译。

- **TF卡（至少16GB，Class 10及以上）**：推荐32GB以保证充足空间
- **读卡器**：用于刷写固件
- **网线**：至少1根，用于连接树莓派与主路由
  - 刚开始，需要使用网线连接树莓派和电脑，以便于做最初的网络配置调整，开启WiFI，修改OpenWRT接口IP地址。
  - 在后续步骤中，网线只会用于与主路由进行连接，把它变成旁路由，最终的拓扑可见下图。

- **电源适配器**：树莓派4B专用5V/3A Type-C电源

### 1.2 软件工具

| 工具用途         | 推荐软件                               | 下载链接                                                     |
| ---------------- | -------------------------------------- | ------------------------------------------------------------ |
| 固件刷写         | BalenaEtcher                           | [https://www.balena.io/etcher/](https://www.balena.io/etcher/) |
| SSH远程访问      | Terminal（Windows）/ Terminal（macOS） | 原生应用商店安装即可                                         |
| 文件传输（可选） | WinSCP                                 | [https://winscp.net/eng/download.php](https://winscp.net/eng/download.php) |

### 1.3 固件与插件准备

- **OpenWRT固件**：`openwrt-24.10.2-bcm27xx-bcm2711-rpi-4-ext4-factory.img.gz`  
  下载地址：[https://downloads.openwrt.org/releases/24.10.2/targets/bcm27xx/bcm2711/](https://downloads.openwrt.org/releases/24.10.2/targets/bcm27xx/bcm2711/)
- **OpenClash插件**：`luci-app-openclash_0.46.137-beta_all.ipk`  
  下载地址：[https://github.com/vernesong/OpenClash/releases/download/v0.46.137/luci-app-openclash_0.46.137_all.ipk](https://github.com/vernesong/OpenClash/releases/download/v0.46.137/luci-app-openclash_0.46.137_all.ipk)



*在选择OpenWRT固件时，可根据自己的硬件选择对应版本，官方Selector网站为：https://firmware-selector.openwrt.org/?version=SNAPSHOT&target=bcm27xx%2Fbcm2711&id=rpi-4*




## 二、刷写OpenWRT固件

### 2.1 打开BalenaEtcher

打开BalenaEtcher，点击“从文件烧录”，选择下载的OpenWRT固件（`.img.gz`格式）。

![image-20250819123650398](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819123650398.png)

### 2.2 刷写固件至TF卡

1. 点击**选择目标磁盘***（Select Target）*，选择TF卡对应的盘符（注意：确保选择正确，避免格式化其他磁盘）  

   ![image-20250819123819861](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819123819861.png)

2. 点击**现在烧录！** *（Flash!）*，等待刷写完成（约3-5分钟，期间请勿拔出读卡器）  

3. *刷写完成后，如果Windows可能提示“需要格式化磁盘”，**点击取消**，不要格式化！*

4. 烧录完成：

   ![image-20250819123951007](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819123951007.png)

   ![image-20250819123957692](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819123957692.png)

### 2.3 首次启动树莓派

1. 将烧录好的TF卡插入树莓派4B的TF卡槽  

   1. **默认的OpenWRT固件中，此时需要连接网线（树莓派 RJ45 LAN口 → 电脑 RJ45 LAN口）**  
   2. 如果使用的是Immortalwrt固件，则可以直接连接名为“immortalwrt”的免密WIFI，然后进行后续动作。但经过实测，暂不推荐使用Immortalwrt固件。

2. 已经连接好树莓派和电脑后，接通电源，树莓派自动启动（指示灯闪烁，启动完成后绿灯常亮）

   ![image-20250819134322468](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819134322468.png)

3. 关闭电脑的WIFI，此时电脑会显示正在连接以太网

   ![image-20250819134048638](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819134048638.png)

   ![image-20250819134058019](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819134058019.png)

   

4. 等大概10s左右（可以手动check是否已经被分配IP），不出意外，你的电脑本地IP应该变成了192.168.1.xxx，此时去浏览器访问192.168.1.1，即可看到OpenWRT的可视化界面，无需密码，直接点击登录即可。登陆后如图所示，小功告成！我们已经拥有了一部搭载OpenWRT的树莓派了。

   ![image-20250819134231487](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819134231487.png)

   ![image-20250819134451526](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819134451526.png)

5. 接下来是一个非常重要的步骤，也是和其它教程不太一样的。找到Network--Wireless，会看到有一个被disabled的名为OpenWrt的WiFi，启用它，不需要修改任何配置，然后保存应用即可。确定OpenWrt的WIFI被启用后，网线就可以从你的电脑上移除了，后面我们会通过WIFI连接到OpenWrt的界面，来执行所有操作。

   ![image-20250819135552677](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819135552677.png)

6. 这时你可以打开电脑的WiFi，并且连接上刚才我们启用的OpenWrt WiFi，仍然可以直接访问OpenWRT的web界面。

   ![image-20250819140047053](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819140047053.png)

7. **接下来是另一个无比重要、也是全篇最为复杂的配置修改，请集中精力，看我的步骤：**

   1. **找到Network--Interface**，会看到一个已经被创建好的lan，它有很多已经配置好的信息，比如它会有一个192.168.1.1/24的IPv4地址，和一个后面会被禁用的IPv6地址

      ![image-20250819140454165](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819140454165.png)

   2. **点击编辑（Edit）**，在弹出的窗口中，首先修改IPv4网关（IPv4 gateway），网关的地址跟你的上联路由器有关，比如我使用的一款路由器，机身上会写着管理地址为“192.168.8.1”。所以我会把IPv4网关修改成与这个地址相同的“192.168.8.1”，IPv4地址（IPv4 Address）则可以修改成192.168.8.xxx，xxx≠1，255，254等冲突地址，在这里我修改成了**192.168.8.101，记住这个地址**，可以根据主路由可使用的地址自行判断。

      ![image-20250819140815408](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819140815408.png)

      *再比如，小米路由器的地址应该是192.168.31.1，如果你使用的是小米路由器，则需要改成对应的网关并找一个对应网段的IP作为树莓派的IPv4 Address；TP-LINK/华为/ASUS等等同理。*

      **此时不需要点保存“Save”，后面还有很多要改。**

   3. **在Advanced Settings中**，勾选“Force link”、"Use default gateway"，Use custom DNS servers填写刚才IPv4 gateway的地址，比如上图中我的网关改成了192.168.8.1，那这里填写的内容是一样的。

      同时，在IPv6 assignment length中选择禁用disabled，因为全球网络环境复杂，且IPv6的支持至今仍然是个谜。

      ![image-20250819141316759](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819141316759.png)

   4. **在DHCP Server--General Setup中**，勾选忽略此接口“Ignore interface”

      ![image-20250819141444258](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819141444258.png)

   5. **在IPv6 Settings中**，禁用RA-Service和DHCPv6-Service，此时我们终于可以**点击Save保存**了

      ![image-20250819141543349](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819141543349.png)

   6. 这时我们会看到在Web界面的右上角，显示有刚才我们所作的变更数量，他们尚未被应用到OpenWrt系统中，点击Save & Apply，会弹出Connectivity change。如果没有信心，可以点击蓝色的按钮，这时需要在90秒内在此完成接入确认，否则刚才所作的变更会回滚。

      但真男人做事果决，这次我选红色的线。

      ![image-20250819141856951](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819141856951.png)

      ![image-20250819141903707](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819141903707.png)

   7. 不出意外的话，已经出意外了，你会发现192.168.1.1连不上了。但真男人不投降，**在操作系统里（我使用的是Windows 11）忘记刚才“OpenWrt”的WIFI，并再次连接它**。这一步主要是为了刷新WiFi所分配的IP地址，因为现在你要访问的不是192.168.1.1，记得你刚才修改过的gateway和IPv4 address吗？现在要访问IPv4 Address了，比如我刚才修改成了192.168.8.101，则访问这个地址，一切都回来了：

      ![image-20250819142316895](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819142316895.png)

8. 在Network--DHCP/DNS--Filter（过滤器）中，勾选过滤IPv6 AAAA记录，保存并应用。

9. 如果你的主路由本来是可以访问网络的，那么此时你连接着OpenWrt，也可以访问公网了。百度一下试试吧！


## 三、OpenWRT系统扩容（不建议跳过）

默认情况下，OpenWRT仅使用TF卡的部分空间（约200MB），需手动扩容以利用全部容量。毕竟后面我们还要装一堆package和Open小猫咪。*BTW，你甚至可以用来做一个在线下载或者NAS。*

这一步需要用到`ssh`工具，如果你是Windows 11/10，可以在Microsoft Store安装“Terminal”，MacOS的话自己会带一个。

   ```bash
   # root@上面改的IPv4地址
   ssh root@192.168.8.101
   
   The authenticity of host '192.168.8.101 (192.168.8.101)' can't be established.
   ED25519 key fingerprint is SHA256:balabalabala.
   This key is not known by any other names.
   Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
   #这里手动敲个yes
   
   Warning: Permanently added '192.168.8.101' (ED25519) to the list of known hosts.
   
   BusyBox v1.36.1 (2025-06-23 20:40:36 UTC) built-in shell (ash)
   
     _______                     ________        __
    |       |.-----.-----.-----.|  |  |  |.----.|  |_
    |   -   ||  _  |  -__|     ||  |  |  ||   _||   _|
    |_______||   __|_____|__|__||________||__|  |____|
             |__| W I R E L E S S   F R E E D O M
    -----------------------------------------------------
    OpenWrt 24.10.2, r28739-d9340319c6
    -----------------------------------------------------
   === WARNING! =====================================
   There is no root password defined on this device!
   Use the "passwd" command to set up a new password
   in order to prevent unauthorized SSH logins.
   --------------------------------------------------
   root@OpenWrt:~#
   ```

*PS. 可以用命令提示符吗？可以，主要是确保你可以执行ssh命令即可。*

### 3.1 安装扩容工具

1. 执行以下命令安装工具：

   ```bash
   # 更新package list
   opkg update
   # 安装扩容所需的cmd
   opkg install cfdisk fdisk block-mount
   # 一大段输出后，最后大概会是这些：
   Configuring terminfo.
   Configuring libmount1.
   Configuring libncurses6.
   Configuring lsblk.
   Configuring block-mount.
   Configuring libfdisk1.
   Configuring cfdisk.
   Configuring fdisk.
   ```

如果在这一步你碰到网络问题（downloads.openwrt.org），有可能是因为无法访问opkg的官方源，建议换成清华的源，速度和质量都不错。点击System--Software，会看到软件包，点击“Configure opkg”：

![image-20250819144225075](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819144225075.png)

opkg.conf和customfeeds.conf都不要动，一个没啥用，另一个还得hack，直接修改最下面框框（“**/etc/opkg/distfeeds.conf**”）中的内容：

```bash
# 这是修改前的内容，把里边的https://downloads.openwrt.org/，都替换成https://mirrors.tuna.tsinghua.edu.cn/openwrt/就行了。
# 如果出现6.6.93-1-5642ee3ae5a6da3ce336f51cf968083c相关的错误，是因为镜像源没有完全同步对应的package，自己找到镜像源里得package，修改成对应名字的包即可。

src/gz openwrt_core https://downloads.openwrt.org/releases/24.10.2/targets/bcm27xx/bcm2711/packages
src/gz openwrt_base https://downloads.openwrt.org/releases/24.10.2/packages/aarch64_cortex-a72/base
src/gz openwrt_kmods https://downloads.openwrt.org/releases/24.10.2/targets/bcm27xx/bcm2711/kmods/6.6.93-1-5642ee3ae5a6da3ce336f51cf968083c
src/gz openwrt_luci https://downloads.openwrt.org/releases/24.10.2/packages/aarch64_cortex-a72/luci
src/gz openwrt_packages https://downloads.openwrt.org/releases/24.10.2/packages/aarch64_cortex-a72/packages
src/gz openwrt_routing https://downloads.openwrt.org/releases/24.10.2/packages/aarch64_cortex-a72/routing
src/gz openwrt_telephony https://downloads.openwrt.org/releases/24.10.2/packages/aarch64_cortex-a72/telephony
```



### 3.2 查看磁盘信息

执行命令确认TF卡设备名称：

```bash
lsblk
```

**预期输出**（树莓派TF卡通常为`mmcblk0`）：

```
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
mmcblk0     179:0    0 29.7G  0 disk 
├─mmcblk0p1 179:1    0   50M  0 part /boot
└─mmcblk0p2 179:2    0  190M  0 part /overlay
```

会看到TF卡的名字，比如上面的输出中，TF卡的名字是`mmcblk0`，盘它。

### 3.3 调整分区大小

1. 启动分区工具：

   ```bash
   cfdisk /dev/mmcblk0
   ```

2. 找到Size最大的那一个，一般是最后一个，以我使用的32G TF卡为例：

   ![image-20250819145154476](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819145154476.png)

3. 在交互界面操作：

   - **选择分区**：用**上下键**选中最后一行空间最大的那个`Free space`（根分区，类型为`Linux`）  
   - **调整大小**：按**左右键**选择`New`，按**Enter**确认  
   - **使用最大空间**：我这里是29.6G，直接按**Enter**（默认使用全部可用空间）  
   - **选择primary**：Enter确认
   - **保存更改**：按**左右键**选择`Write`，回车，输入`yes`并按**Enter**确认 。怕了吗？补个图让你心安一些：

     ![image-20250819145454941](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819145454941.png)
   - **退出工具**：选择`Quit`并按**Enter**  

### 3.4 分区格式化

此时执行`fdisk -l`命令，应该会看到名为“/dev/mmcblk0p3”的新分区：

```bash
Device         Boot  Start      End  Sectors  Size Id Type
/dev/mmcblk0p1 *      8192   139263   131072   64M  c W95 FAT32 (LBA)
/dev/mmcblk0p2      147456   360447   212992  104M 83 Linux
/dev/mmcblk0p3      360448 62333951 61973504 29.6G 83 Linux
```

**对新分区进行格式化**：

```bash
root@OpenWrt:~# mkfs.ext4 /dev/mmcblk0p3

mke2fs 1.47.0 (5-Feb-2023)
Discarding device blocks: done
Creating filesystem with 7746688 4k blocks and 1937712 inodes
Filesystem UUID: e2c4b3c0-32cd-4fd1-92b2-454ed4c36198
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000

Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): #回车即可
done
Writing superblocks and filesystem accounting information: done
```

### 3.5 扩容

这里会碰到OpenWrt的一个bug，神奇的是，你需要改成中文界面，就不存在这个bug。

在System--Software中，搜索`luci-i18n-base-zh-cn`，点击安装Install，安装完成后刷新界面即可：

![image-20250819151229544](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819151229544.png)

刷新界面后，系统会变成中文，点击系统--挂载点（System--Mounts），点击“生成配置”后，下拉到“挂载点”部分，会看到我们刚才创建和格式化的`/mnt/mmcblk0p3`，点击“编辑Edit”：

![image-20250819151655961](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819151655961.png)在挂载点中，选择“作为根文件系统使用”：

![image-20250819151805975](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819151805975.png)

**注意，此时下面会给你一堆系统生成的命令，这部分命令中有一个不能称之为错误的错误：**

```bash
mkdir -p /tmp/introot
mkdir -p /tmp/extroot
mount --bind / /tmp/introot
# mount /dev/sda1 /tmp/extroot 这一行替换成我们刚才创建的mmcblk0p3，如下：
mount /dev/mmcblk0p3 /tmp/extroot
tar -C /tmp/introot -cvf - . | tar -C /tmp/extroot -xf -
umount /tmp/introot
umount /tmp/extroot
```

**逐行粘贴到命令行中执行：**

```bash
root@OpenWrt:~# mkdir -p /tmp/introot
root@OpenWrt:~# mkdir -p /tmp/extroot
root@OpenWrt:~# mount --bind / /tmp/introot
root@OpenWrt:~# mount /dev/mmcblk0p3 /tmp/extroot
root@OpenWrt:~# tar -C /tmp/introot -cvf - . | tar -C /tmp/extroot -xf -
# 一顿输出猛如虎...
root@OpenWrt:~# umount /tmp/introot
root@OpenWrt:~# umount /tmp/extroot
```

**然后点击保存按钮--点击保存并应用按钮：**

![image-20250819152126473](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819152126473.png)

![image-20250819152501869](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819152501869.png)

**回到ssh，执行`reboot`大法**，完成后，可以在首页看到已经扩容完成：

```bash
reboot
```

*首页表现：*

![image-20250819153107671](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819153107671.png)

*软件包页表现：*

![image-20250819153318845](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819153318845.png)



## 四、安装开放小猫咪

截至到目前为止，你拥有了一台以树莓派为硬件基础，以OpenWRT为软件基础的旁路由器。

接下来是安装Open小猫咪的Magic time，再次提醒，合法合规，做个好人。

### 4.1 安装依赖和开放猫

1. 更换dnsmasq为dnsmasq-full，点击系统--软件包--更新列表，更新完之后，搜索`dnsmasq`，卸载`dnsmasq`，安装`dnsmasq-full`

   ![image-20250819154711783](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819154711783.png)

   ![image-20250819154726836](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819154726836.png)

2. 从某个版本开始，OpenWrt放弃iptables，改用nftables，我们首先要安装开放猫所需要的nftables依赖

   ```bash
   opkg update
   opkg install bash dnsmasq-full curl ca-bundle ip-full ruby ruby-yaml kmod-tun kmod-inet-diag unzip kmod-nft-tproxy luci-compat luci luci-base
   # 这一步出问题可能会有两个原因，一个是使用官方源的网络原因，另一个是因为换了镜像源导致的package不匹配的原因……
   ```

3. 安装完依赖之后，上传下载好的开放猫的ipk包：

   ![image-20250819155027717](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819155027717.png)

   点击安装：

   ![image-20250819155049341](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819155049341.png)

4. 如果出现XHR错误，不用理它。点击更新列表，在已安装中可以看到已经安装了开放猫：

   ![image-20250819155223064](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819155223064.png)

5. 点击最上面的退出按钮，再登录，可以看到多出一个“服务”标签，下面有我们安装好的开放猫：

   ![image-20250819155318899](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819155318899.png)

6. 点击它，进入开放猫的dashboard，会提示你是否安装内核，点取消即可，后面会自动安装。

   ![image-20250819155451818](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819155451818.png)

7. 点击配置订阅，输入一系列不可描述订阅地址，Agent一般默认Clash，保存，切换成刚才你不可描述的配置文件。此时会自动返回首页，切换到运行日志，能看到它在下载刚才提示你的内核了：

   ![image-20250819155905064](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819155905064.png)

### 4.2 配置开放猫

1. 找到覆写设置--DNS设置，勾选图片中的选项：

   ![image-20250819160341405](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819160341405.png)

2. 拉到下面的DNS Server中，添加OpenWRT的地址，我的是192.16.8.101，记得吗？

   然后点击保存配置、应用配置。

   ![image-20250819160905741](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819160905741.png)

3. 如果你的内核下载一直停留在某个进度，可以尝试在覆写设置--常规设置--Github地址修改中，改为CDN

   ![image-20250819161029730](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819161029730.png)

4. 在开放猫的首页中，内核下载后会自动启动代理服务，如果在下方访问检查中看到Github/YouTube连接正常，即使它们可能时延比较高，说明已经配置好代理了。

   ![image-20250819161254729](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819161254729.png)

5. **重要步骤，**此时万里长征仅需最后一步，修改本地IP地址和DNS。IP地址为主路由和OpenWRT同一网段的地址，DNS设置为OpenWRT的管理地址，旁路由正式发挥作用：

   ![image-20250819161702515](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819161702515.png)

   6. ❀❀❀**Enjoy，正文完**❀❀❀


## 五、开放猫的一些可选配置

### 5.1 配置Open克莱屎/开放Clash/开放猫

1. 刷新Web界面，进入 **服务 → OpenClash**  
2. 首次使用需初始化，点击**核心管理**，选择**Clash Premium**核心，点击**下载核心**  
3. 进入**配置订阅**，点击**添加订阅**：
   - **订阅名称**：自定义（如“我的订阅”）  
   - **订阅地址**：输入你的Clash订阅链接（需包含节点信息）  
   - **自动更新**：勾选“启用自动更新”，更新间隔设为“24小时”  
4. 点击**保存**，等待订阅更新完成（约10秒）  
5. 进入**全局设置**，将**运行模式**设为**规则模式**  
6. 点击**启动OpenClash**，状态显示“运行中”即成功。


## 六、主题美化

在这种繁杂的教程中，你一定看到过某些漂亮的紫色界面，而我们截图和你实际看到的，都是古老的bootstrap风格。这其实是因为主题设置的原因。

那款花哨的主题是这个：https://github.com/jerrykuku/luci-theme-argon



### 6.1 安装luci-theme-argon主题

回到ssh界面，执行以下命令：

```bash
opkg update
opkg install luci-compat
opkg install luci-lib-ipkg
cd /tmp/
# 会下载主题包
wget --no-check-certificate https://github.com/jerrykuku/luci-theme-argon/releases/download/v2.3.2/luci-theme-argon_2.3.2-r20250207_all.ipk

# 做一下rename，有时会有bug，下载下来的文件叫xxx?sp=r
mv b4abb55f-7d79-4c42-b1de-95626c950071?sp=r luci-theme-argon_2.3.2-r20250207_all.ipk

opkg install luci-theme-argon*.ipk

```

刷新界面，是你想要的样子了吧：

![image-20250819162443373](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819162443373.png)

### 6.2 切换主题

1. 进入 **系统 → 系统 → 语言和界面**  
2. 在**设计主题**下拉菜单中选择已安装的主题（如`Argon`）  
3. 点击**保存并应用**，页面自动刷新，主题生效。


## 七、常见问题排查

### 7.1 无法访问OpenWRT Web界面

- **检查IP**：确认电脑IP与旁路由同网段（如`192.168.x.y`）  
- **重启网络**：拔插树莓派网线，或执行`/etc/init.d/network restart`  

### 7.2 开放猫启动失败

开放猫本身软件质量和界面逻辑比较混乱，毕竟core才是精华，但也算是V/S等框架中相对好用的了

- **检查核心**：进入**服务 → OpenClash → 核心管理**，确认核心已下载且版本匹配  
- **日志排查**：查看**运行日志**，若提示“端口被占用”，关闭冲突服务（如`dnsmasq`）  
- **依赖检查**：重新安装依赖包：`opkg install --force-reinstall dnsmasq-full`


## 八、总结

1. 树莓派的作为主路由和旁路由都是大材小用，希望作为主路由的朋友，不妨自己研究一下可刷梅林固件的路由器。
2. 对于其它对标开放猫的代理，需要自行寻找已经编译好的包，或自行根据自身平台进行编译。
3. 我依然会选择被Freeman视为垃圾的商业路由器作为主路由，省心，稳定。
4. 还在考虑是否接受赞助。

