# Deploying OpenWRT on Raspberry Pi 4B as a Secondary Router with OpenClash

As is well known, my tutorials are particularly suitable for novice users. For example, "iTerm2 + Oh My Zsh for a Comfortable Terminal Experience" can basically be used with Copy-Paste.

Following the same principle, through consulting various materials and practice, this tutorial presents how to use Raspberry Pi 4B by deploying OpenWRT as a secondary router and installing OpenClash as a secondary router proxy. When using it, please abide by local laws and regulations, explore technology with the original intention, broaden technical horizons, and maintain independent thinking.

Without further ado, let's learn by doing! **Everything uses official packages and sources**. Here's the final result:

![](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819120215979.png)

## I. Preparation

### 1.1 Hardware List
- **Raspberry Pi 4B (8GB RAM version)**: Ensure good heat dissipation, preferably with heat sinks or a fan
  - I used the 8GB RAM version, and there is an officially compiled firmware available, which will be much more convenient. If using Raspberry Pi 5, you need to find a usable OpenWRT firmware or compile it yourself.

- **TF Card (at least 16GB, Class 10 or higher)**: 32GB is recommended for sufficient space
- **Card Reader**: For flashing firmware
- **Ethernet Cable**: At least one, for connecting Raspberry Pi to the main router
  - Initially, you need to connect the Raspberry Pi to the computer with an Ethernet cable for initial network configuration adjustments, enabling Wi-Fi, and modifying the OpenWRT interface IP address.
  - In subsequent steps, the Ethernet cable will only be used to connect to the main router, turning it into a secondary router. The final topology can be seen in the diagram below.

- **Power Adapter**: Dedicated 5V/3A Type-C power supply for Raspberry Pi 4B

### 1.2 Software Tools

| Tool Purpose         | Recommended Software                  | Download Link                                                     |
| -------------------- | -------------------------------------- | ---------------------------------------------------------------- |
| Firmware Flashing    | BalenaEtcher                          | [https://www.balena.io/etcher/](https://www.balena.io/etcher/)   |
| SSH Remote Access    | Terminal (Windows) / Terminal (macOS) | Install from native app store                                    |
| File Transfer (Optional) | WinSCP                               | [https://winscp.net/eng/download.php](https://winscp.net/eng/download.php) |

### 1.3 Firmware and Plugin Preparation

- **OpenWRT Firmware**: `openwrt-24.10.2-bcm27xx-bcm2711-rpi-4-ext4-factory.img.gz`  
  Download link: [https://downloads.openwrt.org/releases/24.10.2/targets/bcm27xx/bcm2711/](https://downloads.openwrt.org/releases/24.10.2/targets/bcm27xx/bcm2711/)
- **OpenClash Plugin**: `luci-app-openclash_0.46.137-beta_all.ipk`  
  Download link: [https://github.com/vernesong/OpenClash/releases/download/v0.46.137/luci-app-openclash_0.46.137_all.ipk](https://github.com/vernesong/OpenClash/releases/download/v0.46.137/luci-app-openclash_0.46.137_all.ipk)

*When selecting OpenWRT firmware, choose the version corresponding to your hardware. The official selector website is: https://firmware-selector.openwrt.org/?version=SNAPSHOT&target=bcm27xx%2Fbcm2711&id=rpi-4*

## II. Flashing OpenWRT Firmware

### 2.1 Open BalenaEtcher

Open BalenaEtcher, click "Flash from file", and select the downloaded OpenWRT firmware (`.img.gz` format).

![image-20250819123650398](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819123650398.png)

### 2.2 Flash Firmware to TF Card

1. Click **Select Target**, and choose the drive corresponding to the TF card (Note: Ensure correct selection to avoid formatting other disks)  

   ![image-20250819123819861](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819123819861.png)

2. Click **Flash!** and wait for the flashing to complete (approximately 3-5 minutes; do not remove the card reader during this time)  

3. *After flashing, if Windows prompts "You need to format the disk", **click Cancel**; do not format!*

4. Flashing completed:

   ![image-20250819123951007](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819123951007.png)

   ![image-20250819123957692](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819123957692.png)

### 2.3 First Boot of Raspberry Pi

1. Insert the flashed TF card into the TF card slot of Raspberry Pi 4B  

   1. **In the default OpenWRT firmware, you need to connect an Ethernet cable (Raspberry Pi RJ45 LAN port → Computer RJ45 LAN port) at this time**  
   2. If using Immortalwrt firmware, you can directly connect to the password-free WIFI named "immortalwrt" and then proceed with subsequent actions. However, based on actual testing, Immortalwrt firmware is not recommended for now.

2. After connecting the Raspberry Pi to the computer, power it on. The Raspberry Pi will start automatically (indicator light flashes; green light remains on after startup completion)

   ![image-20250819134322468](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819134322468.png)

3. Turn off the computer's WIFI. The computer will show that it is connecting to Ethernet

   ![image-20250819134048638](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819134048638.png)

   ![image-20250819134058019](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819134058019.png)

4. Wait for about 10 seconds (you can manually check if an IP has been assigned). If all goes well, your computer's local IP should become 192.168.1.xxx. Now visit 192.168.1.1 in the browser to see the OpenWRT web interface. No password is required; just click login. After logging in, as shown in the figure, we have successfully set up a Raspberry Pi running OpenWRT.

   ![image-20250819134231487](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819134231487.png)

   ![image-20250819134451526](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819134451526.png)

5. Next is a very important step, which is different from other tutorials. Find Network--Wireless, and you will see a disabled WiFi named OpenWrt. Enable it without modifying any configuration, then save and apply. After confirming that the OpenWrt WiFi is enabled, you can remove the Ethernet cable from your computer. Later, we will connect to the OpenWrt interface via WiFi to perform all operations.

   ![image-20250819135552677](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819135552677.png)

6. Now you can turn on your computer's WiFi and connect to the OpenWrt WiFi we just enabled. You can still directly access the OpenWRT web interface.

   ![image-20250819140047053](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819140047053.png)

7. **Next is another extremely important and the most complex configuration modification in the entire tutorial. Please focus and follow my steps:**

   1. **Find Network--Interface**, and you will see a pre-created lan interface with various configured information, such as an IPv4 address of 192.168.1.1/24 and an IPv6 address that will be disabled later.

      ![image-20250819140454165](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819140454165.png)

   2. **Click Edit**, and in the pop-up window, first modify the IPv4 gateway. The gateway address is related to your upstream router. For example, the router I use has a management address "192.168.8.1" written on the device. So I will change the IPv4 gateway to "192.168.8.1", and the IPv4 Address can be changed to 192.168.8.xxx, where xxx ≠ 1, 255, 254, etc. to avoid conflicts. Here I changed it to **192.168.8.101. Remember this address**; you can determine it based on the available addresses of your main router.

      ![image-20250819140815408](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819140815408.png)

      *For example, Xiaomi routers typically use 192.168.31.1. If you use a Xiaomi router, you need to change the gateway accordingly and find an IP in the corresponding网段 for the Raspberry Pi's IPv4 Address; the same applies to TP-LINK/Huawei/ASUS, etc.*

      **Do not click "Save" yet; there are more settings to modify.**

   3. **In Advanced Settings**, check "Force link" and "Use default gateway". For Use custom DNS servers, enter the same address as the IPv4 gateway. For example, in the image above, my gateway was changed to 192.168.8.1, so the content filled here is the same.

      At the same time, select disabled in IPv6 assignment length because the global network environment is complex, and IPv6 support is still inconsistent.

      ![image-20250819141316759](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819141316759.png)

   4. **In DHCP Server--General Setup**, check "Ignore interface"

      ![image-20250819141444258](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819141444258.png)

   5. **In IPv6 Settings**, disable RA-Service and DHCPv6-Service. Now we can finally **click Save**.

      ![image-20250819141543349](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819141543349.png)

   6. You will now see the number of changes you made displayed in the upper right corner of the web interface. These changes have not yet been applied to the OpenWrt system. Click Save & Apply, and a Connectivity change prompt will pop up. If you're unsure, you can click the blue button, which requires you to confirm the connection within 90 seconds; otherwise, the changes will be rolled back.

      But real men act decisively. This time, I choose the red line.

      ![image-20250819141856951](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819141856951.png)

      ![image-20250819141903707](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819141903707.png)

   7. If everything goes as expected, something unexpected might have happened: you can no longer connect to 192.168.1.1. But real men don't give up. **Forget the "OpenWrt" WiFi in your operating system (I'm using Windows 11) and reconnect to it**. This step is mainly to refresh the IP address assigned by WiFi because now you need to access the IPv4 Address you just modified instead of 192.168.1.1. For example, I changed mine to 192.168.8.101, so accessing this address brings everything back:

      ![image-20250819142316895](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819142316895.png)

8. In Network--DHCP/DNS--Filter, check "Filter IPv6 AAAA records" and save and apply.

9. If your main router can access the internet, then you should also be able to access the public network when connected to OpenWrt. Try searching on Baidu!

## III. Expanding OpenWRT System Storage (Not Recommended to Skip)

By default, OpenWRT only uses part of the TF card space (approximately 200MB). Manual expansion is required to utilize the full capacity, especially since we will install many packages and OpenClash later. *BTW, you can even use it as an online downloader or NAS.*

This step requires the `ssh` tool. If you're using Windows 11/10, you can install "Terminal" from the Microsoft Store; macOS comes with one pre-installed.

   ```bash
   # root@the IPv4 address you modified above
   ssh root@192.168.8.101
   
   The authenticity of host '192.168.8.101 (192.168.8.101)' can't be established.
   ED25519 key fingerprint is SHA256:balabalabala.
   This key is not known by any other names.
   Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
   # Manually type yes here
   
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

*PS. Can I use Command Prompt? Yes, as long as you can execute ssh commands.*

### 3.1 Install Expansion Tools

1. Execute the following commands to install tools:

   ```bash
   # Update package list
   opkg update
   # Install commands required for expansion
   opkg install cfdisk fdisk block-mount
   # After a large amount of output, the final lines will be something like:
   Configuring terminfo.
   Configuring libmount1.
   Configuring libncurses6.
   Configuring lsblk.
   Configuring block-mount.
   Configuring libfdisk1.
   Configuring cfdisk.
   Configuring fdisk.
   ```

If you encounter network issues (downloads.openwrt.org) at this step, it may be because you cannot access the official opkg source. It is recommended to switch to Tsinghua University's mirror, which offers good speed and quality. Click System--Software, find the software packages, and click "Configure opkg":

![image-20250819144225075](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819144225075.png)

Do not modify opkg.conf or customfeeds.conf; one is useless, and the other requires hacking. Directly modify the content in the bottom box ("**/etc/opkg/distfeeds.conf**"):

```bash
# This is the original content. Replace all instances of https://downloads.openwrt.org/ with https://mirrors.tuna.tsinghua.edu.cn/openwrt/.
# If you encounter errors related to 6.6.93-1-5642ee3ae5a6da3ce336f51cf968083c, it's because the mirror source hasn't fully synced the corresponding package. Find the package in the mirror source and modify it to the corresponding name.

src/gz openwrt_core https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/24.10.2/targets/bcm27xx/bcm2711/
src/gz openwrt_base https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/24.10.2/packages/aarch64_cortex-a72/base/
src/gz openwrt_kmods https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/24.10.2/targets/bcm27xx/bcm2711/kmods/6.6.93-1-5642ee3ae5a6da3ce336f51cf968083c/
src/gz openwrt_luci https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/24.10.2/packages/aarch64_cortex-a72/luci/
src/gz openwrt_packages https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/24.10.2/packages/aarch64_cortex-a72/packages/
src/gz openwrt_routing https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/24.10.2/packages/aarch64_cortex-a72/routing/
src/gz openwrt_telephony https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/24.10.2/packages/aarch64_cortex-a72/telephony/
```

### 3.2 Check Disk Information

Execute the command to confirm the TF card device name:

```bash
lsblk
```

**Expected output** (Raspberry Pi TF card is usually `mmcblk0`):

```
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
mmcblk0     179:0    0 29.7G  0 disk 
├─mmcblk0p1 179:1    0   50M  0 part /boot
└─mmcblk0p2 179:2    0  190M  0 part /overlay
```

You will see the name of the TF card, such as `mmcblk0` in the output above. Let's work with it.

### 3.3 Adjust Partition Size

1. Launch the partition tool:

   ```bash
   cfdisk /dev/mmcblk0
   ```

2. Find the largest Size, usually the last one. Taking my 32GB TF card as an example:

   ![image-20250819145154476](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819145154476.png)

3. Operations in the interactive interface:

   - **Select partition**: Use the **up/down arrow keys** to select the last `Free space` (root partition, type `Linux`)  
   - **Resize**: Use the **left/right arrow keys** to select `New` and press **Enter** to confirm  
   - **Use maximum space**: Mine shows 29.6G; press **Enter** directly (uses all available space by default)  
   - **Select primary**: Press Enter to confirm  
   - **Save changes**: Use left/right arrow keys to select `Write`, press Enter, type `yes`, and press **Enter** to confirm. Worried? Here's a reassuring screenshot:

     ![image-20250819145454941](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819145454941.png)
   - **Exit tool**: Select `Quit` and press **Enter**  

### 3.4 Format Partition

Executing the `fdisk -l` command should now show a new partition named "/dev/mmcblk0p3":

```bash
Device         Boot  Start      End  Sectors  Size Id Type
/dev/mmcblk0p1 *      8192   139263   131072   64M  c W95 FAT32 (LBA)
/dev/mmcblk0p2      147456   360447   212992  104M 83 Linux
/dev/mmcblk0p3      360448 62333951 61973504 29.6G 83 Linux
```

**Format the new partition**:

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
Creating journal (32768 blocks): # Just press Enter
done
Writing superblocks and filesystem accounting information: done
```

### 3.5 Expand Filesystem

Here you will encounter a bug in OpenWrt. Surprisingly, switching to the Chinese interface resolves this bug.

In System--Software, search for `luci-i18n-base-zh-cn`, click Install, and refresh the interface after installation:

![image-20250819151229544](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819151229544.png)

After refreshing, the system will be in Chinese. Click 系统--挂载点 (System--Mounts), click "生成配置" (Generate Config), then scroll down to the "挂载点" (Mount Points) section. You will see the `/mnt/mmcblk0p3` we just created and formatted. Click "编辑" (Edit):

![image-20250819151655961](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819151655961.png)

In the mount point settings, select "作为根文件系统使用" (Use as root filesystem):

![image-20250819151805975](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819151805975.png)

**Note that the system will generate a series of commands, which contain a minor issue that isn't quite a bug:**

```bash
mkdir -p /tmp/introot
mkdir -p /tmp/extroot
mount --bind / /tmp/introot
# mount /dev/sda1 /tmp/extroot  Replace this line with our newly created mmcblk0p3 as follows:
mount /dev/mmcblk0p3 /tmp/extroot
tar -C /tmp/introot -cvf - . | tar -C /tmp/extroot -xf -
umount /tmp/introot
umount /tmp/extroot
```

**Paste and execute these commands line by line in the terminal:**

```bash
root@OpenWrt:~# mkdir -p /tmp/introot
root@OpenWrt:~# mkdir -p /tmp/extroot
root@OpenWrt:~# mount --bind / /tmp/introot
root@OpenWrt:~# mount /dev/mmcblk0p3 /tmp/extroot
root@OpenWrt:~# tar -C /tmp/introot -cvf - . | tar -C /tmp/extroot -xf -
# A lot of output will appear...
root@OpenWrt:~# umount /tmp/introot
root@OpenWrt:~# umount /tmp/extroot
```

**Then click the Save button--click Save & Apply:**

![image-20250819152126473](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819152126473.png)

![image-20250819152501869](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819152501869.png)

**Return to ssh and execute the `reboot` command**. After completion, you can see the expanded storage on the homepage:

```bash
reboot
```

*Homepage display:*

![image-20250819153107671](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819153107671.png)

*Software package page display:*

![image-20250819153318845](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819153318845.png)

## IV. Install OpenClash

Up to this point, you have a secondary router based on Raspberry Pi hardware and OpenWRT software.

Next is the Magic time to install OpenClash. Again, remember to comply with laws and regulations and be a good citizen.

### 4.1 Install Dependencies and OpenClash

1. Replace dnsmasq with dnsmasq-full. Go to System--Software--Update List. After updating, search for `dnsmasq`, uninstall `dnsmasq`, and install `dnsmasq-full`.

   ![image-20250819154711783](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819154711783.png)

   ![image-20250819154726836](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819154726836.png)

2. Starting from a certain version, OpenWrt has replaced iptables with nftables. We first need to install nftables dependencies required by OpenClash.

   ```bash
   opkg update
   opkg install bash dnsmasq-full curl ca-bundle ip-full ruby ruby-yaml kmod-tun kmod-inet-diag unzip kmod-nft-tproxy luci-compat luci luci-base
   # There may be two reasons for issues here: network problems with the official source, or package mismatch due to switching to a mirror source...
   ```

3. After installing dependencies, upload the downloaded OpenClash ipk package:

   ![image-20250819155027717](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819155027717.png)

   Click Install:

   ![image-20250819155049341](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819155049341.png)

4. If an XHR error occurs, ignore it. Click Update List, and you can see OpenClash installed in the Installed section:

   ![image-20250819155223064](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819155223064.png)

5. Click the Logout button at the top, then log in again. You will see a new "Services" tab with the installed OpenClash:

   ![image-20250819155318899](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819155318899.png)

6. Click on it to enter the OpenClash dashboard. It will prompt you to install the core; click Cancel, as it will be installed automatically later.

   ![image-20250819155451818](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819155451818.png)

7. Click "Configuration Subscription", enter a series of indescribable subscription addresses, keep Agent as default Clash, save, and switch to the configuration file you just added. It will automatically return to the homepage, and switching to the running log will show it downloading the core mentioned earlier:

   ![image-20250819155905064](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819155905064.png)

### 4.2 Configure OpenClash

1. Find Override Settings--DNS Settings and check the options in the image:

   ![image-20250819160341405](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819160341405.png)

2. Scroll down to DNS Server and add the OpenWRT address, which is 192.16.8.101 in my case. Remember?

   Then click Save Configuration and Apply Configuration.

   ![image-20250819160905741](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819160905741.png)

3. If your core download is stuck at a certain progress, try changing the Github address to CDN in Override Settings--General Settings--Github Address Modification.

   ![image-20250819161029730](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819161029730.png)

4. On the OpenClash homepage, the proxy service will start automatically after the core is downloaded. If the connectivity check shows that Github/YouTube connections are normal (even if latency is high), it means the proxy is configured correctly.

   ![image-20250819161254729](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819161254729.png)

5. **Important step:** Now the final step of the long march is to modify the local IP address and DNS. The IP address should be in the same网段 as the main router and OpenWRT, and the DNS should be set to OpenWRT's management address for the secondary router to function properly:

   ![image-20250819161702515](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819161702515.png)

   6. ❀❀❀**Enjoy, end of main text**❀❀❀

## V. Optional Configurations for OpenClash

### 5.1 Configure OpenClash

1. Refresh the web interface and go to **Services → OpenClash**  
2. Initialization is required for first use. Click **Core Management**, select **Clash Premium** core, and click **Download Core**  
3. Go to **Configuration Subscription** and click **Add Subscription**:
   - **Subscription Name**: Custom (e.g., "My Subscription")  
   - **Subscription Address**: Enter your Clash subscription link (must contain node information)  
   - **Auto Update**: Check "Enable auto-update" and set update interval to "24 hours"  
4. Click **Save** and wait for subscription update to complete (about 10 seconds)  
5. Go to **Global Settings** and set **Running Mode** to **Rule Mode**  
6. Click **Start OpenClash**. Status showing "Running" indicates success.

## VI. Theme Customization

In such complex tutorials, you must have seen some beautiful purple interfaces, while our screenshots and what you actually see have the old bootstrap style. This is actually due to theme settings.

That fancy theme is this one: https://github.com/jerrykuku/luci-theme-argon

### 6.1 Install luci-theme-argon

Return to the ssh interface and execute the following commands:

```bash
opkg update
opkg install luci-compat
opkg install luci-lib-ipkg
cd /tmp/
# This will download the theme package
wget --no-check-certificate https://github.com/jerrykuku/luci-theme-argon/releases/download/v2.3.2/luci-theme-argon_2.3.2-r20250207_all.ipk

# Rename in case of bugs where the downloaded file is named xxx?sp=r
mv b4abb55f-7d79-4c42-b1de-95626c950071?sp=r luci-theme-argon_2.3.2-r20250207_all.ipk

opkg install luci-theme-argon*.ipk

```

Refresh the interface, and it should look like what you wanted:

![image-20250819162443373](https://raw.githubusercontent.com/sirius1024/pubimgs/master/blogs/openwrt/image-20250819162443373.png)

### 6.2 Switch Theme

1. Go to **System → System → Language and Interface**  
2. Select the installed theme (e.g., `Argon`) from the **Design Theme** dropdown menu  
3. Click **Save & Apply**, and the page will refresh automatically with the new theme.

## VII. Troubleshooting Common Issues

### 7.1 Cannot Access OpenWRT Web Interface

- **Check IP**: Ensure the computer's IP is in the same网段 as the secondary router (e.g., `192.168.x.y`)  
- **Restart Network**: Unplug and replug the Raspberry Pi Ethernet cable, or execute `/etc/init.d/network restart`  

### 7.2 OpenClash Fails to Start

OpenClash itself has inconsistent software quality and interface logic. After all, the core is the essence, but it is relatively user-friendly among frameworks like V/S.

- **Check Core**: Go to **Services → OpenClash → Core Management** and confirm the core is downloaded and version-matched  
- **Log Troubleshooting**: Check **Running Logs**. If prompted "Port occupied", close conflicting services (e.g., `dnsmasq`)  
- **Dependency Check**: Reinstall dependencies with `opkg install --force-reinstall dnsmasq-full`

## VIII. Summary

1. Using Raspberry Pi as a main router or secondary router is overkill. For those considering it as a main router, you might want to research routers that can flash Merlin firmware.
2. For other proxies comparable to OpenClash, you need to find precompiled packages or compile them yourself according to your platform.
3. I still prefer commercial routers that Freeman considers garbage as my main router—convenient and stable.
4. Still considering whether to accept sponsorships.