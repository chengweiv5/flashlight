# Mate 60 Pro 厂商私有手电筒调光能力核查

- 日期：2026-10-08
- 目标：华为 Mate 60 Pro（ALN-AL00），HarmonyOS 4.2.0，Android 12 / API 31
- 范围：普通、未 root、非系统签名的第三方 Android APK
- 方法：官方资料与 SDK 检查；通过已授权 ADB 只读检查目标手机的厂商配置和系统组件
- 项目状态：保持停止开发；本记录不授权安装、点灯测试或权限绕过

## 结论

**厂商配置中存在后置手电筒亮度节点的线索，但没有找到这台设备上普通第三方 Android APK 可用、可交付的多档 LED 调光接口。**

不能把“标准接口不存在”扩大为“硬件绝对不支持调光”。同样，存在名为 `brightness` 的节点也不能直接证明硬件提供多个有效亮度档位。

后置 LED 调光是用户明确的核心需求。当前未证明有可靠实现路径，继续保持停止开发，不用屏幕柔光或其他功能替代该需求。

## 1. 真机只读证据

### 1.1 标准 Android 接口

目标手机实际的 `framework.jar` 中：

- 有 `CameraManager.setTorchMode(String, boolean)` 和 `registerTorchCallback`。
- 没有 `turnOnTorchWithStrengthLevel`、`getTorchStrengthLevel`。
- 未发现 `CameraCharacteristics` 的标准强度档位字段。

以上来自目标手机字节码检查，不是仅根据系统版本推断。

### 1.2 驱动配置存在亮度节点，但写入受限

实际读取 `/vendor/etc/ueventd.rc` 第 181 行：

```text
/sys/class/leds/torch/brightness  0664  system  system
```

`/vendor/etc/selinux/vendor_file_contexts` 还包含后置 `torch/brightness` 和 `torch/max_brightness` 的 `sysfs_brightness` 标签规则。

这说明固件为该类控制节点配置了权限与安全标签，但不证明节点当前的可用档位或驱动行为：

- `0664 system system` 的配置只给系统用户和系统组写权限，其他用户没有写权限。
- 实测 ADB 身份为 `uid=2000(shell)`，SELinux 为 `Enforcing`。
- 只读列举 `/sys/class/leds` 返回 `Permission denied`。
- 没有读取到实际 `max_brightness` 值，没有尝试写入节点。

因此，这条线索不能作为普通 APK 可直接使用的调光方案。也没有验证 root 或系统签名环境下的可用性。

### 1.3 自带手电筒仍调用开关接口

对目标手机的 `SystemUI.apk` 进行静态反汇编，确认：

```text
HwFlashlightControllerImpl.setFlashlight(boolean)
    -> CameraManager.setTorchMode(String, boolean)
```

所查控制路径只有开关参数，未发现多档亮度参数。此结论限定于所检查的路径，不声称穷尽所有系统组件。

### 1.4 厂商框架与 CameraKit 线索

检查了实际设备上的：

- `hwframework.jar`
- `hwPartCamera.jar`
- `hwServices.jar`
- `services.jar`
- `hwpostcamera.jar`
- `HwCameraKit.apk`（1.1.6）
- `SystemUI.apk`

在所扫描的相关方法与字符串中，没有找到可确认的第三方手电筒亮度设置入口。

CameraKit 内部有 `PostCamera2.switchOnTorchMode(...)` 兼容包装，但其调用通过反射指向 `com.huawei.hwpostcamera.HwPostCamera`。目标手机实际的 `hwpostcamera.jar` 没有对应的 `switchOnTorchMode` / `switchOffTorchMode` 方法，不能把包装代码当作本机可用能力。

## 2. 官方公开资料与 SDK

### Android 标准调光

Android 官方将 `turnOnTorchWithStrengthLevel` 和 `FLASH_INFO_STRENGTH_MAXIMUM_LEVEL` 标为 API 33 引入；最大强度值大于 1 才表示支持多档调节。

- [CameraManager](https://developer.android.com/reference/android/hardware/camera2/CameraManager)
- [CameraCharacteristics](https://developer.android.com/reference/android/hardware/camera2/CameraCharacteristics)

### 华为 Android Camera Engine

华为官方示例提供 `getSupportedFlashMode()` 和 `setFlashMode(int)`。检查官方 Maven 的 CameraKit 1.1.5 SDK，相关整数值是闪光模式枚举：

```text
HW_FLASH_AUTO = 0
HW_FLASH_CLOSE = 1
HW_FLASH_OPEN = 2
HW_FLASH_ALWAYS_OPEN = 3
```

它们不是亮度档位。该 SDK 的公开相关 API 和 RequestKey 中未找到 torch brightness/intensity 设置参数。所查 SDK 版本不代表穷尽华为所有历史、未来或合作伙伴版本。

- [华为 CameraKit 官方示例](https://developer.huawei.com/consumer/en/codelab/CameraKit-SuperSlow/index.html)
- [华为官方 CameraKit 1.1.5 SDK](https://developer.huawei.com/repo/com/huawei/multimedia/camerakit/1.1.5/camerakit-1.1.5.aar)
- [华为官方 Camera Engine 示例仓库](https://github.com/AppGalleryConnect/explore-hms-demos/tree/master/feature_cameraengine)

较新 HarmonyOS 原生 Camera Kit 的文档不能直接作为 HarmonyOS 4.2 Android 兼容 APK 的可调用性证据；本次未把原生状态回调中的亮度字段视为 Android 调光设置接口。

## 3. 仍未确定的内容

- 驱动是否支持多个有效常亮强度，以及范围、温控和硬件保护行为。
- 华为系统内部或合作伙伴未公开 SDK 是否另有入口。
- 获得系统签名、特殊预装权限或 root 后能否控制底层节点。

这些可能性均未验证，不能作为继续普通 APK 开发的可行性结论。本次没有尝试反射调用、Binder 私有命令、驱动写入、修改权限或关闭 SELinux。

## 4. 执行与交付边界

本次未安装测试包、未拍照、未点灯、未修改手机设置，也未恢复产品开发。只读导出的固件文件保留在宿主临时目录，不提交到公开仓库；公开仓库仅保存本研究记录。

已有产品设计不在本次改写。若未来找到可用接口，需要重新核对目标固件、调用权限及安全验证方案，再决定是否恢复开发。
