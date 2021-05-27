# XPS 7590 Hackinosh黑苹果

## 注意⚠️

本 EFI 适用于 bigsur 11.1-11.2.2  其余版本未经过测试

仅为个人使用，故 README 写的杂乱，欢迎交流推进 XPS-7590 的 Hackintosh 进程

##  1 参考

GitHub:

- https://github.com/gorquan/OC-XPS-7590
- https://github.com/Liyunlock/DELL-XPS-15-7590-4K-Touchscreen_OpenCore_Hackintosh
- https://github.com/romancin/Dell-XPS-7590-OpenCore
- https://github.com/kirainmoe/hasee-tongfang-macos/issues/33
- https://github.com/xxxzc/xps15-9570-macos/issues/40
- https://github.com/Drowningfish223/Xps-7590-BigSur
- https://github.com/stakeout55/Dell-XPS-7590-mac-OS-Big-Sur-11.1
- https://github.com/xxxzc/xps15-9570-macos/issues/69




## 2 准备

### 2.1 硬件概况

| name | introduction      | name | introduction  |
| :--: | ----------------- | :--: | :-----------: |
| 型号 | XPS15-7590        | Cpu  |   i7-9750h    |
| 屏幕 | 4k 夏普触控屏     | 网卡 | Dw1820a(更换) |
| 固态 | SN720 512G        | 声卡 | ALC298(原生)  |
| 内存 | 48G(32+16) 英睿达 | Bios |               |

### 2.2 Bios设置

雷电设置关键

### 2.3 CFG解锁

setup_var_3 0x789 0x00 to disable overclocking lock

setup_var_3 0x6ED 0x00 to disable CFG lock

https://www.imacpc.net/archives/1549



dvmt-preallocted 设置为 96MB

## 3 Bug及解决方法备份



### 3.1 啰嗦模式问题

- OC引导Big Sur卡在`IOConsoleUsers: gIOScreenLockState 3, hs 0, bs 0 now` 注入苹果显示器EDID 48HZ (45 46字节替换为A6A6 再使用128计算最后一位字节 )

- DeviceP.. 删除注入EDID 

- 现在新版WhateverGreen kexts 可以直接内屏 60Hz——>无须注入 EDID

  

### 3.2 4k内屏闪屏



使用`ioreg -lw0 | grep IODisplayEDID | sed "/[^<]*</s///" | xxd -p -r | strings -6` 查看屏幕的生产型号

夏普屏幕安装对应的显示器描述文件, 位置`/Library/ColorSync/Profiles`

外接4k显示器 延迟高 卡顿: 重建缓存解决 `sudo kextchache -i\`

UI 4K设置: dlcd-max 1400000 UIscale—>02



目前type-c 低电压输入下首次链接type-c仍然出现

### 3.3 雷电3设备

使用SSDT-TB3和SSDT-TYPEC和kexts内的IOElectrify

Bios雷电设置最低权限 取消auto相关设置

取消 IOElectrify.kext 和 type-c 的 SSDT 仅开启 SSDT-TB3 仍可



### 3.5 关于0.8ghz锁频

解锁EC�风扇控制位，动态注入风扇控制
目前采用bios关闭DPTF解决此问题



改造散热VRM后有效 降频次数减少 时间缩短为几秒

替换cpufriends 后改善极少出现锁频情况 实现27档变频

### 3.7 LCD亮度调节

改为`Fn+S/B`

karabiner软件实现快捷键

### 3.8 睡眠后蓝牙不可用

Q: 睡眠开机后蓝牙显示打开但不可连接设备 不可搜索新设备

​	1. 将蓝牙内建后+睡眠修复(hackintosh) —>仍然出现


```html
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>PciRoot(0x0)/Pci(0x1c,0x0)/Pci(0x0,0x0)</key>
	<dict>
		<key>AAPL,slot-name</key>
		<string>WLAN</string>
		<key>compatible</key>
		<string>pci14e4,43a3</string> // 也有改为43a0的
		<key>device-type</key>
		<string>Airport Extreme</string>
		<key>model</key>
		<string>DW1820A (BCM4350) 802.11ac Wireless</string>
		<key>name</key>
		<string>Airport</string>
		<key>pci-aspm-default</key>
		<integer>0</integer>
	</dict>
</dict>
</plist>

```

![lL2kFd](https://cdn.jsdelivr.net/gh/flyingchase/Private-Img@master/uPic/lL2kFd.png)



​	2. 屏蔽背面两脚针后生效

### 3.9 替换主板编号后 12 位后触控板二指失效

退出 iCloud 并在其他设备上删除本机，重新修改二码，清除 NVRM 开机重建缓存，设置触控板三指/二指拖 玄学好了。。。





## 4 三码更新

使用`opencorepkg`下载下来的   `macserial`内置程序进行PlatformInfo-Generic-MLB和SystemSeriaNumber填写(先在apple序列号保修查询官网确保未使用过) 

重启后:

`ioreg -d2 -c IOPlatformExpertDevice | awk -F\" '/IOPlatformUUID/{print $(NF-1)}’
   `获得本机主班的UUID填入SystemUUID

⚠️：SystemUUID最后的十二位替换为网卡的mac地址 （mac地址中:去除并将所有小写字母大写再去替换）

​	该法使得 imessage 和 facetime 体验提升巨大









## 5 优化

### 5.1 CPU降压调节

使用`voltageShif`调节，主要参考：

https://www.insanelymac.com/forum/topic/331775-guide-how-to-undervolt-your-haswell-and-above-cpu/

https://github.com/stakeout55/presigned_VoltageShift_Kext_DellXPS7590

感谢！

偏移设置 -110 -92 -110  亦可设置为 

```shell
CPU voltage offset: -125mv
GPU voltage offset: -125mv
CPU Cache voltage offset: -125mv
System Agency offset: -75mv
```

更改设置时先执行remove再launchd

`sudo ./voltageshift buildlaunchd -125 -125 -125 -75 0 0 60`

`sudo ./voltageshift removelaunchd`![TypOYq](https://cdn.jsdelivr.net/gh/flyingchase/Private-Img@master/uPic/TypOYq.png)



### 5.2 电池供电睡眠后耳机杂音

注入ALC守护进程即可





### 5.3 DVMT offset修改为96MB

Bios修改 使用UEFI和Bios提取工具 setup_var_3 0x** 0x03 地址未确定（待定优化） 目前使用WEG也蛮好 闪屏现象基本消失

 

