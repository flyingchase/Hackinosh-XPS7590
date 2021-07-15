# XPS 7590 Hackinosh黑苹果

## 注意⚠️

本 EFI 适用于 bigsur 11.1-11.2.2  其余版本未经过测试

仅为个人使用，故 README 写的杂乱，欢迎交流推进 XPS-7590 的 Hackintosh 进程

部分内容涉及 BIOS 修改 有硬件损坏的风险

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

- 各大QQ群内大佬的无私分享




## 2 准备

### 2.1 硬件概况

| name | introduction      | name | introduction  |
| :--: | ----------------- | :--: | :-----------: |
| 型号 | XPS15-7590        | Cpu  |   i7-9750h    |
| 屏幕 | 4k 夏普触控屏     | 网卡 | Dw1820a(更换) |
| 固态 | SN720 512G        | 声卡 | ALC298(原生)  |
| 内存 | 48G(32+16) 英睿达 | Bios |    v1.7.0     |

### 2.2 Bios设置

雷电设置关键

- ThunderBolt Adapter Configuration：勾选 Thunder 和 NoSecurity 其他关闭
- ThunderBolt Auto Switch：取消 AutoSwitch 勾选 Native Enumeration

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

BIOS 设置：

​	ThunderBolt Adapter Configuration：勾选 Thunder 和 NoSecurity 其他关闭

​	ThunderBolt Auto Switch：取消 AutoSwitch 勾选 Native Enumeration

​	BIOS修改：

``` shell
setup_var 0x4F0 0x01
setup_var 0x4F6 0x01
```

关闭其他 type-c 和tb3 相关的 ssdt 使用TbtOnPch.aml 并 patch 修改_E42 to XE42 

重置 nvrm 后即可

<img src="https://cdn.jsdelivr.net/gh/flyingchase/Private-Img@master/uPic/6XeA6p.png" alt="6XeA6p" style="zoom:50%;" />

注意第一次睡眠前拔掉雷电设备或者采用开机后即睡眠再唤醒

感谢群内大佬！ 

### 3.5 关于0.8ghz锁频

>解决方案均收集于互联网，侵删

个人尝试：

1. 改造散热VRM后有效 降频次数减少 时间缩短为几秒

2. 替换cpufriends 后改善极少出现锁频情况 实现27档变频

3. 拔掉电池  按开机键释放主板静电 放置 5mins 后插上电池开机——>无效

3. 更改 DVMT 的最大值从 256 到 MAX（BIOS 内pre-allocation 没有 128MB 分配的可选项 最大 64MB） 无效

4. 使用 voltageshift 降压后 偶尔出现

   Intel Power Gradget 动态监测在高负载出现 0.78ghz 锁频时插入原装 dell 充电器cpu 频率恢复正常但五秒后再次锁0.8ghz 此时拔出电源又恢复正常 但五秒后重复锁频

   即拔掉电源线以改善性能——>醉了

5. 修改 BIOS 中的 Bi-dorectional PROCHOT  关闭——>修改 BIOS 有风险 ⚠️

   > 查找有指出可能是主板上温度传感器出错 使用 5.3 中修改 BIOS 方法`setup_var_3 0x724 0x00 `关闭 BD PROCHOT
   >
   > [CPU 风扇停转后发生什么](https://zhuanlan.zhihu.com/p/27624654)
   
   关闭 BD 后仍然出现，但使用电池供电很少出现
   
7. 爬远景评论发现以下几种方式

   1. 拔掉电池 按住开机键 20s 再装上电池 使用电池供电开机后再插上电源
   2. dell 原装圆孔充电器接口内针口损坏——>使用手电筒照射充电器圆孔内部看不大清，但是本机使用 type-c供电出现0.78ghz锁频的次数更多
   
8. 交流群内提供 BIOS 降压方式：

   ``` shell
   0x855 0x01
   0x856 0x01
   0x85B 0x64
   0x85D 0x01
   0xAFF 0x1E
   0xB01 0x01
   ```



现状：

​	重 GPU 需求下仍然出现降频 0.8ghz 现象 拔出电源恢复几秒后再次降频 视负载情况恢复正常睿频 大概在 10s-Long 最长有过持续十分钟

 总结：

​	目前存在可能是主板上元件损坏/~~原装充电器圆孔内针脚损坏~~/BIOS需要更新或者刷低版/电池不合格需要更换



### 3.6 LCD亮度调节

改为`Fn+S/B`

- karabiner软件实现快捷键更改

### 3.6 睡眠后蓝牙不可用

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

### 3.8 替换主板编号后 12 位后触控板二指失效

退出 iCloud 并在其他设备上删除本机，重新修改二码，清除 NVRM 开机重建缓存，设置触控板三指/二指拖 玄学好了。。。

### 3.9 Hackintool 中工具无法查看 CFG 信息

总是在安全性与隐私中 允许签名再重启

https://www.mfpud.com/topics/4303/ 

```shell
# 进入 Hackintool 程序包内Content-resources-kexts
sudo chown -R root:wheel AppleIntelInfo.kext
sudo kextutil AppleIntelInfo.kext
sudo cat /tmp/AppleIntelInfo.dat
# 最后取消注入驱动
sudo kextunload AppleIntelInfo.kext
```

## 4 三码更新

[三码参考链接-写的很详细很完备](https://heipg.cn/tutorial/macserial-and-iservice-opencore.html)

使用`opencorepkg`下载下来的   `macserial`内置程序进行PlatformInfo-Generic-MLB和SystemSeriaNumber填写(先在apple序列号保修查询官网确保未使用过) 

重启后:

`ioreg -d2 -c IOPlatformExpertDevice | awk -F\" '/IOPlatformUUID/{print $(NF-1)}’
   `获得本机主班的UUID填入SystemUUID

⚠️：**SystemUUID**最后的十二位替换为网卡的mac地址 （mac地址中:去除并将所有小写字母大写再去替换）

​	该法使得 imessage 和 facetime 体验提升巨大

config中**ROM：**修改为en0的网卡mac地址 注意删掉: 最后四位与前面有空格

## 5 优化

### 5.1 CPU降压调节

- 使用`voltageShif`调节，主要参考：

  [Guide-undervolt](https://www.insanelymac.com/forum/topic/331775-guide-how-to-undervolt-your-haswell-and-above-cpu/)

  [XPS-7590_VoltageShift](https://github.com/stakeout55/presigned_VoltageShift_Kext_DellXPS7590)

  感谢！


  更改设置时先执行remove再launchd

  `sudo ./voltageshift buildlaunchd <CPU> <GPU> <CPUCache> <SA> <AI/O> <DI/O> <turbo> <pl1> <pl2> <remain> <UpdateMins (0 only apply at bootup)>`

  `sudo ./voltageshift buildlaunchd -135 -92 -125 -75 0 0 1 75 90 1 60`

  打开 turbo 并设置 PL1 56w PL2 90w  将 kexts 留在系统中并 60mins 执行一次

>https://github.com/syscl/CPUTune
>
>https://github.com/SeptemberHX/VoltageShift
>
>```shell
>sudo chown -R root:wheel VoltageShift.kext
>chmod +x voltageshift
>
>
># load kexts
>./voltageshift loadkext
># OR 
>sudo kextutil  -r ./  -b com.sicreative.VoltageShift
>
># unload kexts 
>./voltageshift unloadkext
># OR
>sudo kextunload -b com.sicreative.VoltageShift
>
># 检查是否 load kexts
>kextstat | grep -v com.apple
>
># 下述两条命令使得 voltageshift 在任意地方均可执行
> sudo cp -r VoltageShift.kext /Library/Extensions/
> sudo cp voltageshift /usr/local/bin
>```

![TypOYq](https://cdn.jsdelivr.net/gh/flyingchase/Private-Img@master/uPic/TypOYq.png)

- BIOS降压

  ``` shell
  0x855 0x01
  0x856 0x01
  0x85B 0x64
  0x85D 0x01
  0xAFF 0x1E
  0xB01 0x01
  ```

  

### 5.2 电池供电睡眠后耳机杂音

注入ALC守护进程即可



### 5.3 DVMT offset修改为最大值 MAX

解决4k 内屏/低电压 type-c 输入时外接 4k显示器闪屏

- 在 Window 下使用 DELL_PFS_Extract 工具实现官网对应本机 BIOS.exe 文件提取.bin 文件 （使用方式为将 bios.exe 直接拖到应用图标上打开即可） 感谢远景论坛网友的分享

- 得到文件中`1 -- 1 System BIOS with BIOS Guard.bin `为待提取文件

- 使用 UEFI Tool 打开`.bin`文件 

  - 查找`DVMT`并定位到 PE32![hiBhVr](https://cdn.jsdelivr.net/gh/flyingchase/Private-Img@master/uPic/hiBhVr.png)

  - 将 PE32 处导出为 `Section_PE32_image_Setup.sct  `文件 ![GET145](https://cdn.jsdelivr.net/gh/flyingchase/Private-Img@master/uPic/GET145.png)

    - 使用 `ifrextract`工具将 `.sct`文件转化为 `.txt`格式 再查找 CFG/DVMT 等offset 偏移量

    - 参考 CFG 解锁制作的 UEFI 引导盘 使用命令`setup_var_3 0xA11 0x02/0x03` 设置 再使用`setup_var_3 0xA10 ***`   但是 BIOS 设置无 64MB更大

      ![FXa9XI](https://cdn.jsdelivr.net/gh/flyingchase/Private-Img@master/uPic/FXa9XI.png)

      ![yKs00E](https://cdn.jsdelivr.net/gh/flyingchase/Private-Img@master/uPic/yKs00E.png)

配合 WEG.kext 设置 dvmt 属性可以有效解决上述问题

4k 内屏在使用新版 WhateverGreen.kext 基础上配置相应的 framebuffer补丁后，在低电压45/60w 的PD 充电高负载场景、外接 4k 显示器并睡眠唤醒情况下均未出现闪屏现象



## 其他待整理

### ACPI整理：

> 笔记本背光亮度调节 SSDT-PNLF.aml SSDT-ALS0.aml
> 睡眠秒唤醒 SSDT-GPRW SSDT-UPRW

ACPI

> 电量显示0 SSDT-BATT.aml
> 节能 SSDT-PLUG
> 禁用独显 SSDT-DDGPU.aml
> 解除USB端口限制 SSDT-EC.aml
> 苹果原生电源管理SSDT-PLUG.aml



### 网络接口

wifi 非 en0

1. 进入系统设置-网络，删除左边列表所有项
2. 删除 `/Library/Preferences/SystemConfiguration/NetworkInterfaces.plist`
3. 重启电脑
4. 进入系统设置-网络，点击左侧的 '+'，将 Wi-Fi 添加回来。

