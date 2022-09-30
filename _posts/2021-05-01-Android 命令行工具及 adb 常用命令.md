---
title: Android 命令行工具及 adb 常用命令
tag: [Android, adb]
key: 2021-05-01-Android 命令行工具及 adb 常用命令
---



Refer:

这个 github repo 很详细：[mzlogin/awesome-adb: ADB Usage Complete / ADB 用法大全 (github.com)](https://github.com/mzlogin/awesome-adb)

[Android 调试桥 (adb)  |  Android 开发者  |  Android Developers (google.cn)](https://developer.android.google.cn/studio/command-line/adb?hl=zh-cn#devicestatus)

[ADB 常用命令及其用法大全_醒不了的星期八的博客-CSDN 博客_adb 命令](https://blog.csdn.net/qq_39969226/article/details/87897863)

# 无线连接

## 方式一

1. 先用 usb 连接手机：`adb tcpip 5555`
2. 断开 usb 连接，连接手机：`adb connect <device-ip-address>`
3. 断开 adb 连接：`adb disconnect <device-ip-address>`

## 方式二（无需 usb 线）

1. 手机上下载终端模拟器 [Terminal Emulator for Android](https://apkpure.com/tw/terminal-emulator-for-android/jackpal.androidterm)
2. 在其中运行命令：

```bash
su

setprop service.adb.tcp.port 5555

```

# adb

即 Android Debug Bridge，Android 调试桥。安装在 `android_sdk/platform-tools/` 下。

C/S 架构，分为以下三个组件

- **客户端**：用于发送命令。客户端在开发机器上运行。您可以通过发出 adb 命令从命令行终端调用客户端。
- **守护程序 (adbd)**：用于在设备上运行命令。守护程序在每个设备上作为后台进程运行。
- **服务器**：用于管理客户端与守护程序之间的通信。服务器在开发机器上作为后台进程运行。

## 添加工具环境变量

1. 新建系统环境变量 `ANDROID`，值为 android_sdk/platform-tools/;android_sdk/tools/
2. 将 %ADNROID% 添加到系统环境变量 `Path`

## 常用命令

- 查询设备 `adb devices -l`
- 选择特定设备
- `adb -s`+ 设备序列号
- `adb -e` 选择 TCP/IP 设备
- `adb -d` 选择 usb 设备
- 安装应用 `adb install`
- 卸载 app 但保留数据和缓存文件 `adb uninstall -k`
- 端口转发
- e.g.,设置主机端口 6100 到设备端口 7100 的转发 `adb forward tcp:6100 tcp:7100`
- 主机端口 6100 到 local:logd 的转发 `adb forward tcp:6100 local:logd`
- 相互复制文件 `adb pull/push`
- 查看日志 `logcat`，`-c` 清除 log
- 查看 bug 报告：`adb bugreport`
- 停止 adb 服务器 `adb kill-server`

### shell 命令

许多 shell 命令都由 [toybox](http://landley.net/toybox/) 提供

- 调用 Activity 管理器 `am`，如启动 Activity、强行停止进程、广播 intent、修改设备屏幕属性等等
- 强制停止应用 `force-stop package`
- 调用软件包管理器 `pm`
- `list packages -3` 列出除了系统应用的第三方应用包名
- `list permissions/features/libraries/users`
- 输出给定 package 的 APK 路径 `path package`
- 删除与软件包关联的所有数据 `clear package`
- 向应用授予/撤销权限 `grant/revoke package_name permission`
- 调用设备政策管理器 `dpm`
- 截取屏幕截图/录像 `screencap/screenrecord`，按 Ctrl + C 键可停止屏幕录制；如果不手动停止，到三分钟或 --time-limit 设置的时间限制时，录制将会自动停止。
- 生成文本格式的配置文件信息 `cmd package dump-profiles package`

  - > - Android Runtime (ART) 会收集已安装应用的执行配置文件，这些配置文件用于优化应用性能。

- 重置测试设备 `cmd testharness enable`
- 启动用于检查 sqlite 数据库的 sqlite 命令行程序 `sqlite3 /data/data/com.example.app/databases/rssitems.db`
- 系统信息 `dumpsys `
- `package` 包信息
- `meminfo` 内存使用情况
- `window displays` 显示屏参数
- 查看正在运行的 Services：`dumpsys activity services [<packagename>] `
- 查看内存占用前 10 的 app `top -s 10`
- `adb shell input `
- `tap 500 1450` 模拟屏幕点击事件
- `swipe 100 500 100 1450 100` 模拟手势滑动事件
- `text` 焦点处于某文本框时输入文本
- `keyevent <keycode>` 模拟按键输入

![](https://xdo0.github.io/imgsrc/boxcnPqWcMLF4QudC7wBtWcOGjg.png)

- 查看设备信息 `getprop `
- `ro.product.cpu.abi` 查看 CPU 架构
- `ro.build.version.release` Android 系统版本
- `ro.product.model` 型号

![](https://xdo0.github.io/imgsrc/boxcnPuYAqnZncUskIaMopN1YQf.png)

- 获得 android_id：`settings get secure android_id`
- 查看屏幕分辨率/密度：`wm size/density`
- 获取 MAC 地址：`cat /sys/class/net/wlan0/address` , 根据系统版本参数可能不同
- 查看连接过的 WiFi 密码：`cat /data/misc/wifi/*.conf`
- `date -s 20190531.131600`  将系统日期和时间更改为 2019 年 05 月 31 日 13 点 16 分 00 秒。
- 内核日志：`dmesg`
- 使用 [Monkey](https://developer.android.com/studio/test/monkey) 进行随机压力测试：`monkey -p <packagename> -v 500` 发送 500 个伪随机事件

#### 反编译相关

- 查看当前进程的内存的加载映射情况：`cat /proc/7654/maps`
- 看当前应用使用的端口号信息：`cat /proc/[pid]/net/tcp`
- 查看进程的状态信息：`cat /proc/[pid]/status` 通过该命令获取到当前进程的包
- 查看一个 dex 文件的详细信息：`dexdump <dex_path>`

# 命令行工具

> 在 android studio 中使用更方便

- AAPT2，Android Asset Packaging Tool，Android 资源打包工具
- 获取 apk 的清单文件：`aapt dump xmltree <apk_path> AndroidMainfest.xml > demo.txt`
- apkanalyzer，在构建流程完成后立即了解 APK 的组成，并且可以比较两个 APK 之间的差异
- 输出应用 ID、版本代码和名称：`apk summary apk-file`
- 比较 apk：`apk compare [options] apk-file apk-file2`
- apksigner，为 apk 签名
- avdmanager，创建和管理 Android 虚拟设备 (AVD)
- bmgr，Backup Manager，备份管理器
- bundletool，[Android App Bundle](https://developer.android.google.cn/guide/app-bundle%3Fhl%3Dzh-cn%23other_considerations) 是一种发布格式，其中包含您应用的所有经过编译的代码和资源，它会将 APK 生成及签名交由 Google Play 来完成。
- d8，将项目的 Java 字节码编译为在 Android 设备上运行的 DEX 字节码
- dmtracedump，将项目的 Java 字节码编译为在 Android 设备上运行的 DEX 字节码
- [dumpsys](https://developer.android.google.cn/studio/command-line/dumpsys%3Fhl%3Dzh-cn)，在 Android 设备上运行的工具，可提供有关系统服务的信息。
- etc1tool，命令行实用程序，可用于将 PNG 图片编码为 ETC1 压缩标准格式，并将 ETC1 压缩图片解码回 PNG。
- jobb，构建不透明二进制 Blob (OBB) 格式的已加密和未加密 APK 扩展文件。使用 StorageManager 在应用中下载和装载这些扩展文件。OBB 文件用于为 Android 应用提供额外文件资源（例如图形、音频和视频），这些文件资源与应用的 APK 文件是分开的。
- jetifier，将依赖于支持库的库迁移为依赖于等效的 AndroidX 软件包
- [logcat](https://developer.android.google.cn/studio/command-line/logcat%3Fhl%3Dzh-cn)，命令行工具，用于转储系统消息日志，包括设备抛出错误时的堆栈轨迹，以及从应用使用 Log 类写入的消息。
- mkscard，在多个虚拟设备之间创建共享的 FAT32 磁盘映像，以模拟多个设备中存在相同 SD 卡的情形
- retrace，用于从经过混淆处理的堆栈轨迹获取原始堆栈轨迹。系统会通过在映射文件中对类名和方法名与其原始定义进行匹配来重构堆栈轨迹。
- sdkmanager，查看、安装、更新和卸载 Android SDK 的软件包
- perfetto，通过 adb 在 Android 设备上收集性能信息。多种来源，例如：
- 使用 `ftrace` 收集内核信息
- 使用 `atrace` 收集服务和应用中的用户空间注释
- 使用 `heapprofd` 收集服务和应用的本地内存使用情况信息
- zipalign，zip 归档文件对齐工具。在将 APK 文件分发给最终用户之前，应该先使用 zipalign 进行优化。
