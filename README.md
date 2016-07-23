# Adb 用法大全

Adb，即 [Android Debug Bridge](https://developer.android.com/studio/command-line/adb.html)，它是 Android 开发/测试人员不可替代的强大工具，也是 Android 设备玩家的好玩具。

本文档主要参考 adb 的官方文档和网络上的部分网友整理，详情见 [参考链接][#参考链接]。有部分命令的支持情况可能与 Android 系统版本及定制 ROM 的实现有关。

## 目录

* [设备连接管理](#设备连接管理)
	* [查询已连接设备/模拟器](#查询已连接设备模拟器)
	* [无线连接](#无线连接)
* [应用管理](#应用管理)
	* [查看所有已安装应用](#查看所有已安装应用)
	* [安装 APK](#安装-apk)
	* [卸载应用](#卸载应用)
	* [调起应用](#调起应用)
	* [查看前台 Activity](#查看前台-activity)
* [文件管理](#文件管理)
	* [复制设备里的文件到电脑](#复制设备里的文件到电脑)
	* [复制电脑里的文件到设备](#复制电脑里的文件到设备)
* [调试](#调试)
	* [查看/过滤日志](#查看过滤日志)
* [查看设备信息](#查看设备信息)
	* [查看设备型号](#查看设备型号)
	* [查看设备电池状况](#查看设备电池状况)
	* [查看设备分辨率](#查看设备分辨率)
	* [查看 android\_id](#查看-android_id)
* [其它实用功能](#其它实用功能)
	* [录制屏幕](#录制屏幕)
* [参考链接](#参考链接)

## 设备连接管理

### 查询已连接设备/模拟器

命令：

```
adb devices
```

输出示例：

```
List of devices attached
cf264b8f	device
emulator-5554	device
```

输出格式为 `[serialNumber] [state]`，serialNumber 即我们常说的 SN，state 有如下几种：

* `offline` —— 表示设备未连接成功或无响应。

* `device` —— 设备已连接。注意这个状态并不能标识 Android 系统已经完全启动和可操作，在设备启动过程中设备实例就可连接到 adb，但启动完毕后系统才处于可操作状态。

* `no device` —— 没有设备/模拟器连接。

以上输出显示当前已经连接了两台设备/模拟器，`cf264b8f` 与 `emulator-5554` 分别是它们的 SN。从 `emulator-5554` 这个名字可以看出它是一个 Android 模拟器。

常见异常输出：

1. 没有设备/模拟器连接成功。

   ```
   List of devices attached
   ```

2. 设备/模拟器未连接到 adb 或无响应。

   ```
   List of devices attached
   cf264b8f	offline
   ```

### 无线连接

// TODO

## 应用管理

### 查看所有已安装应用

命令：

```
adb shell pm list packages
```

输出示例：

```
package:com.android.smoketest
package:com.example.android.livecubes
package:com.android.providers.telephony
package:com.google.android.googlequicksearchbox
package:com.android.providers.calendar
package:com.android.providers.media
package:com.android.protips
package:com.android.documentsui
package:com.android.gallery
package:com.android.externalstorage
...
// other packages here
...
```

### 安装 APK

命令：

```
adb install <APK 文件路径>
```

参数：

`adb install` 后面可以跟一些参数来控制安装 APK 的行为，常用参数及含义如下：

| 参数 | 含义                  |
|------|-----------------------|
| -r   | 允许覆盖安装。        |
| -s   | 将应用安装到 sdcard。 |
| -d   | 允许降级覆盖安装。    |

完整参数列表及含义可以直接运行 `adb` 命令然后查看 `adb install [-lrtsdg] <file>` 一节。

如果见到类似如下输出（状态为 `Success`）代表安装成功：

```
12040 KB/s (22205609 bytes in 1.801s)
        pkg: /data/local/tmp/SogouInput_android_v8.3_sweb.apk
Success
```

而如果状态为 `Failure` 则表示安装失败。常见安装失败输出代码、含义及可能的解决办法如下：

| 输出                                               | 含义                                                                     | 解决办法                                        |
|----------------------------------------------------|--------------------------------------------------------------------------|-------------------------------------------------|
| INSTALL\_FAILED\_ALREADY\_EXISTS                   | 应用已经存在                                                             | 使用 `-r` 参数                                  |
| INSTALL\_FAILED\_INVALID\_APK                      | 无效的 APK 文件                                                          |                                                 |
| INSTALL\_FAILED\_INVALID\_URI                      | 无效的 APK 文件名                                                        | 确保 APK 文件名里无中文                         |
| INSTALL\_FAILED\_INSUFFICIENT\_STORAGE             | 空间不足                                                                 | 清理空间                                        |
| INSTALL\_FAILED\_DUPLICATE\_PACKAGE                | 已经存在同名程序                                                         |                                                 |
| INSTALL\_FAILED\_NO\_SHARED\_USER                  | 请求的共享用户不存在                                                     |                                                 |
| INSTALL\_FAILED\_UPDATE\_INCOMPATIBLE              | 已经安装过签名不一样的同名应用，且数据没有移除                           |                                                 |
| INSTALL\_FAILED\_SHARED\_USER\_INCOMPATIBLE        | 请求的共享用户存在但签名不一致                                           |                                                 |
| INSTALL\_FAILED\_MISSING\_SHARED\_LIBRARY          | 安装包使用了设备上不可用的共享库                                         |                                                 |
| INSTALL\_FAILED\_REPLACE\_COULDNT\_DELETE          | 替换时无法删除                                                           |                                                 |
| INSTALL\_FAILED\_DEXOPT                            | dex 优化验证失败或空间不足                                               |                                                 |
| INSTALL\_FAILED\_OLDER\_SDK                        | 设备系统版本低于应用要求                                                 |                                                 |
| INSTALL\_FAILED\_CONFLICTING\_PROVIDER             | 设备里已经存在与应用里同名的 content provider                            |                                                 |
| INSTALL\_FAILED\_NEWER\_SDK                        | 设备系统版本高于应用要求                                                 |                                                 |
| INSTALL\_FAILED\_TEST\_ONLY                        | 应用是 test-only 的，但安装时没有指定 `-t` 参数                          |                                                 |
| INSTALL\_FAILED\_CPU\_ABI\_INCOMPATIBLE            | 包含不兼容设备 CPU 应用程序二进制接口的 native code                      |                                                 |
| INSTALL\_FAILED\_MISSING\_FEATURE                  | 应用使用了设备不可用的功能                                               |                                                 |
| INSTALL\_FAILED\_CONTAINER\_ERROR                  | sdcard 访问失败                                                          | 确认 sdcard 可用，或者安装到内置存储            |
| INSTALL\_FAILED\_INVALID\_INSTALL\_LOCATION        | 不能安装到指定位置                                                       | 切换安装位置，添加或删除 `-s` 参数              |
| INSTALL\_FAILED\_MEDIA\_UNAVAILABLE                | 安装位置不可用                                                           | 一般为 sdcard，确认 sdcard 可用或安装到内置存储 |
| INSTALL\_FAILED\_VERIFICATION\_TIMEOUT             | 验证安装包超时                                                           |                                                 |
| INSTALL\_FAILED\_VERIFICATION\_FAILURE             | 验证安装包失败                                                           |                                                 |
| INSTALL\_FAILED\_PACKAGE\_CHANGED                  | 应用与调用程序期望的不一致                                               |                                                 |
| INSTALL\_FAILED\_UID\_CHANGED                      | 以前安装过该应用，与本次分配的 UID 不一致                                | 清除以前安装过的残留文件                        |
| INSTALL\_FAILED\_VERSION\_DOWNGRADE                | 已经安装了该应用更高版本                                                 | 使用 `-d` 参数                                  |
| INSTALL\_FAILED\_PERMISSION\_MODEL\_DOWNGRADE      | 已安装 target SDK 支持运行时权限的同名应用，要安装的版本不支持运行时权限 |                                                 |
| INSTALL\_PARSE\_FAILED\_NOT\_APK                   | 指定路径不是文件，或不是以 `.apk` 结尾                                   |                                                 |
| INSTALL\_PARSE\_FAILED\_BAD\_MANIFEST              | 无法解析的 AndroidManifest.xml 文件                                      |                                                 |
| INSTALL\_PARSE\_FAILED\_UNEXPECTED\_EXCEPTION      | 解析器遇到异常                                                           |                                                 |
| INSTALL\_PARSE\_FAILED\_NO\_CERTIFICATES           | 安装包没有签名                                                           |                                                 |
| INSTALL\_PARSE\_FAILED\_INCONSISTENT\_CERTIFICATES | 已安装该应用，且签名与 APK 文件不一致                                    | 先卸载设备上的该应用，再安装                    |
| INSTALL\_PARSE\_FAILED\_CERTIFICATE\_ENCODING      | 解析 APK 文件时遇到 `CertificateEncodingException`                       |                                                 |
| INSTALL\_PARSE\_FAILED\_BAD\_PACKAGE\_NAME         | manifest 文件里没有或者使用了无效的包名                                  |                                                 |
| INSTALL\_PARSE\_FAILED\_BAD\_SHARED\_USER\_ID      | manifest 文件里指定了无效的共享用户 ID                                   |                                                 |
| INSTALL\_PARSE\_FAILED\_MANIFEST\_MALFORMED        | 解析 manifest 文件时遇到结构性错误                                       |                                                 |
| INSTALL\_PARSE\_FAILED\_MANIFEST\_EMPTY            | 在 manifest 文件里找不到找可操作标签（instrumentation 或 application）   |                                                 |
| INSTALL\_FAILED\_INTERNAL\_ERROR                   | 因系统问题安装失败                                                       |                                                 |
| INSTALL\_FAILED\_USER\_RESTRICTED                  | 用户被限制安装应用                                                       |                                                 |
| INSTALL\_FAILED\_DUPLICATE\_PERMISSION             | 应用尝试定义一个已经存在的权限名称                                       |                                                 |
| INSTALL\_FAILED\_NO\_MATCHING\_ABIS                | 应用包含设备的应用程序二进制接口不支持的 native code                     |                                                 |
| INSTALL\_CANCELED\_BY\_USER                        | 应用安装需要在设备上确认，但未操作设备或点了取消                         | 在设备上同意安装                                |
| INSTALL\_FAILED\_ACWF\_INCOMPATIBLE                | 应用程序与设备不兼容                                                     |                                                 |
| does not contain AndroidManifest.xml               | 无效的 APK 文件                                                          |                                                 |
| is not a valid zip file                            | 无效的 APK 文件                                                          |                                                 |
| Offline                                            | 设备未连接成功                                                           | 先将设备与 adb 连接成功                         |
| unauthorized                                       | 设备未授权允许调试                                                       |                                                 |
| error: device not found                            | 没有连接成功的设备                                                       | 先将设备与 adb 连接成功                         |
| protocol failure                                   | 设备已断开连接                                                           | 先将设备与 adb 连接成功                         |
| Unknown option: -s                                 | Android 2.2 以下不支持安装到 sdcard                                      | 不使用 `-s` 参数                                |
| No space left on devicerm                          | 空间不足                                                                 | 清理空间                                        |
| Permission denied ... sdcard ...                   | sdcard 不可用                                                            |                                                 |

参考：[PackageManager.java](https://github.com/android/platform_frameworks_base/blob/master/core%2Fjava%2Fandroid%2Fcontent%2Fpm%2FPackageManager.java)

### 卸载应用

命令：

```
adb uninstall [-k] <packagename>
```

`<packagename>` 表示应用的包名，`-k` 参数可选，表示卸载应用但保留数据和缓存目录。

命令示例：

```
adb uninstall com.qihoo360.mobilesafe
```

表示卸载 360 手机卫士。

### 调起应用

// TODO

### 查看前台 Activity

命令：

```
adb shell dumpsys activity activities | grep mFocusedActivity
```

输出示例：

```
mFocusedActivity: ActivityRecord{8079d7e u0 com.cyanogenmod.trebuchet/com.android.launcher3.Launcher t42}
```

其中的 `com.cyanogenmod.trebuchet/com.android.launcher3.Launcher` 就是当前处于前台的 Activity。

## 文件管理

### 复制设备里的文件到电脑

命令：

```
adb pull <设备里的文件路径> [电脑上的目录] 
```

其中 `电脑上的目录` 参数可以省略，默认复制到当前目录。

例：

```
adb pull /sdcard/sr.mp4 ~/tmp/
```

*小技巧：*设备上的文件路径可能需要 root 权限才能访问，如果你的设备已经 root 过，可以先使用 `adb shell` 和 `su` 命令在 adb shell 里获取 root 权限后，先 `cp /path/on/device /sdcard/filename` 将文件复制到 sdcard，然后 `adb pull /sdcard/filename /path/on/pc`。

### 复制电脑里的文件到设备

命令：

```
adb push <电脑上的文件路径> <设备里的目录> 
```

例：

```
adb push ~/sr.mp4 /sdcard/
```

*小技巧：*设备上的文件路径普通权限可能无法直接写入，如果你的设备已经 root 过，可以先 `adb push /path/on/pc /sdcard/filename`，然后 `adb shell` 和 `su` 在 adb shell 里获取 root 权限后，`cp /sdcard/filename /path/on/device`。

## 调试

### 查看/过滤日志

// TODO

## 查看设备信息

### 查看设备型号

命令：

```
adb shell getprop ro.product.model
```

输出示例：

```
Nexus 5
```

### 查看设备电池状况

命令：

```
adb shell dumpsys battery
```

输入示例：

```
Current Battery Service state:
  AC powered: false
  USB powered: true
  Wireless powered: false
  status: 2
  health: 2
  present: true
  level: 44
  scale: 100
  voltage: 3872
  temperature: 280
  technology: Li-poly
```

其中 `scale` 代表最大电量，`level` 代表当前电量。上面的输出表示还剩下 44% 的电量。

### 查看设备分辨率

命令：

```
adb shell dumpsys window displays
```

输出示例：

```
WINDOW MANAGER DISPLAY CONTENTS (dumpsys window displays)
  Display: mDisplayId=0
    init=1080x1920 480dpi cur=1080x1920 app=1080x1776 rng=1080x1005-1794x1701
    deferred=false layoutNeeded=false
    ...
    // some other output here
    ...
```

### 查看 android\_id

命令：

```
adb shell settings get secure android_id
```

输出示例：

```
51b6be48bac8c569
```

## 其它实用功能

### 录制屏幕

录制屏幕以 mp4 格式保存到 /sdcard：

```
adb shell screenrecord /sdcard/filename.mp4
```

需要停止时按 <kbd>Ctrl-C</kbd>，默认录制时间和最长录制时间都是 180 秒。

如果需要导出到电脑：

```
adb pull /sdcard/filename.mp4
```

`screenrecord` 命令也支持一些参数，可以使用 `adb shell screenrecord --help` 查看，下面是简介：

| 参数                | 含义                                            |
|---------------------|-------------------------------------------------|
| --size WIDTHxHEIGHT | 视频的尺寸，比如 `1280x720`，默认是屏幕分辨率。 |
| --bit-rate RATE     | 视频的比特率，默认是 4Mbps。                    |
| --time-limit TIME   | 录制时长，单位秒。                              |
| --verbose           | 输出更多信息。                                  |

## 参考链接

* [Android Debug Bridge](https://developer.android.com/studio/command-line/adb.html)
* [ADB Shell Commands](https://developer.android.com/studio/command-line/shell.html)
| INSTALL\_FAILED\_MEDIA\_UNAVAILABLE                | 安装位置不可用                                                           | 一般为 sdcard，确认 sdcard 可用或安装到内存 |
