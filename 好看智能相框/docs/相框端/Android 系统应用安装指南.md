# Android 系统应用安装指南

本文档介绍如何将 APK 安装为 Android 系统应用，使其获得系统级权限。

## 前置条件

- 设备已 Root
- 已安装 ADB 工具
- 设备通过 USB 连接并开启调试模式

## 系统应用目录说明

| 目录 | 权限级别 | 说明 |
|------|----------|------|
| `/system/priv-app/` | 特权系统应用 | 可获得所有签名保护权限，推荐 |
| `/system/app/` | 普通系统应用 | 权限较少 |
| `/data/app/` | 用户应用 | 普通权限 |

## 安装步骤

### 1. 连接设备并获取 Root 权限

```bash
# 检查设备连接
adb devices -l

# 获取 root 权限
adb root
```

### 2. 重新挂载 /system 为可写

```bash
adb shell "mount -o rw,remount /system"
```

### 3. 创建应用目录

```bash
# 创建 priv-app 子目录（目录名建议与应用名一致）
adb shell "mkdir -p /system/priv-app/PhotoFrame"
```

### 4. 推送 APK 文件

```bash
adb push /path/to/PhotoFrame.apk /system/priv-app/PhotoFrame/PhotoFrame.apk
```

### 5. 设置正确的权限

```bash
# 目录权限 755
adb shell "chmod 755 /system/priv-app/PhotoFrame"

# APK 文件权限 644
adb shell "chmod 644 /system/priv-app/PhotoFrame/PhotoFrame.apk"

# 所有者设为 root
adb shell "chown root:root /system/priv-app/PhotoFrame/PhotoFrame.apk"
```

### 6. 验证安装

```bash
adb shell "ls -la /system/priv-app/PhotoFrame/"
```

预期输出：
```
-rw-r--r-- root     root     10459714 2025-12-15 07:54 PhotoFrame.apk
```

### 7. 重启设备

```bash
adb reboot
```

## 一键安装脚本

```bash
#!/bin/bash

APK_PATH="$1"
APP_NAME="PhotoFrame"

if [ -z "$APK_PATH" ]; then
    echo "用法: $0 <apk_path>"
    exit 1
fi

echo "=== 安装系统应用 ==="

# 获取 root 权限
adb root
sleep 2

# 重新挂载 /system
adb shell "mount -o rw,remount /system"

# 创建目录
adb shell "mkdir -p /system/priv-app/$APP_NAME"

# 推送 APK
adb push "$APK_PATH" "/system/priv-app/$APP_NAME/$APP_NAME.apk"

# 设置权限
adb shell "chmod 755 /system/priv-app/$APP_NAME"
adb shell "chmod 644 /system/priv-app/$APP_NAME/$APP_NAME.apk"
adb shell "chown root:root /system/priv-app/$APP_NAME/$APP_NAME.apk"

# 验证
echo "=== 验证安装 ==="
adb shell "ls -la /system/priv-app/$APP_NAME/"

echo "=== 重启设备 ==="
adb reboot

echo "安装完成！设备正在重启..."
```

## 验证系统权限

重启后，可以通过 Logcat 查看应用是否获得系统权限：

```bash
adb logcat -s HkApplication
```

预期输出：
```
============ 系统权限检测 ============
当前 UID: 1000
系统 UID: 1000
是否系统应用: true
--------- 权限状态 ---------
✅ 系统设置写入 (android.permission.WRITE_SECURE_SETTINGS)
✅ 精确位置 (android.permission.ACCESS_FINE_LOCATION)
✅ 修改WiFi (android.permission.CHANGE_WIFI_STATE)
--------- 签名检测 ---------
系统签名匹配: ✅ 是
=========================================
```

## 卸载系统应用

```bash
# 获取 root 权限
adb root

# 重新挂载 /system
adb shell "mount -o rw,remount /system"

# 删除应用目录
adb shell "rm -rf /system/priv-app/PhotoFrame"

# 重启
adb reboot
```

## 注意事项

### 1. 签名要求

如果应用在 AndroidManifest.xml 中声明了 `android:sharedUserId="android.uid.system"`，则必须：
- 使用设备的系统签名（platform key）签署 APK
- 否则应用将无法安装或启动

### 2. Android 版本差异

| Android 版本 | 注意事项 |
|--------------|----------|
| 5.0+ | 需要 SELinux 权限，可能需要修改 sepolicy |
| 6.0+ | 运行时权限对系统应用自动授予 |
| 10.0+ | 更严格的分区验证，可能需要禁用 AVB |

### 3. OTA 更新

系统应用安装在 `/system` 分区，OTA 更新可能会覆盖或删除这些应用。

### 4. 权限白名单（Android 8.0+）

某些敏感权限需要在 `/etc/permissions/` 目录下添加白名单文件：

```xml
<!-- /system/etc/permissions/privapp-permissions-photoframe.xml -->
<?xml version="1.0" encoding="utf-8"?>
<permissions>
    <privapp-permissions package="com.hk.phone_album.pad">
        <permission name="android.permission.WRITE_SECURE_SETTINGS"/>
        <!-- 其他需要的权限 -->
    </privapp-permissions>
</permissions>
```

## 常见问题

### Q: 安装后应用无法启动？

A: 检查签名是否与系统签名匹配。可以通过以下命令查看：
```bash
adb shell "dumpsys package com.hk.phone_album.pad | grep -i signature"
```

### Q: 提示 "INSTALL_FAILED_SHARED_USER_INCOMPATIBLE"？

A: 应用使用了 `sharedUserId="android.uid.system"` 但签名不匹配。需要使用正确的系统签名重新签署 APK。

### Q: 如何获取设备的系统签名？

A: 通常需要从设备厂商或 ROM 源码获取 `platform.pk8` 和 `platform.x509.pem` 文件，然后使用 `keytool-importkeypair` 工具生成 keystore。

---

## 参考命令汇总

```bash
# 检查设备
adb devices -l

# 获取 root
adb root

# 挂载 /system 可写
adb shell "mount -o rw,remount /system"

# 查看已安装的系统应用
adb shell "ls -la /system/priv-app/"

# 查看应用信息
adb shell "dumpsys package com.hk.phone_album.pad"

# 查看应用 UID
adb shell "ps | grep phone_album"

# 查看 Logcat
adb logcat -s HkApplication
```

