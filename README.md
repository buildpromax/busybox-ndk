# BusyBox NDK

用 GitHub Actions 每周自动跟踪 BusyBox 上游最新稳定版,使用官方 Android NDK 交叉编译出各架构的**完全静态**二进制,并自动发布到 Release。

## 产物说明

| 架构 | 说明 |
|------|------|
| `busybox-arm` | ARMv7 (armv7a) — 绝大多数 32 位手机 |
| `busybox-arm64` | AArch64 — 现代 64 位手机 |
| `busybox-x86` | i686 — 老旧 32 位模拟器/平板 |
| `busybox-x86_64` | x86-64 — 64 位模拟器/ChromeOS |

- **NDK r29 + API 24 (Android 7.0+)**,完全静态链接,不依赖任何系统 so
- 基于官方 `android_502_defconfig`,并针对现代 NDK 修掉了若干编译兼容问题
- 每个版本附带 `SHA256SUMS` 校验文件和打包好的 all-arch tar.gz

> ⚠️ MIPS / MIPS64 不再提供:NDK 自 r18 (2018) 起就移除了 MIPS 工具链,任何现代 NDK 都编不了,且 MIPS Android 设备早已绝迹。

## 使用方法

```bash
# 下载后推到设备
adb push busybox-arm64 /data/local/tmp/busybox
adb shell chmod 755 /data/local/tmp/busybox
adb shell /data/local/tmp/busybox --install -s /data/local/tmp/bb
adb shell /data/local/tmp/bb/ls -la
```

或直接在 `adb shell` / Termux 里运行 `/data/local/tmp/busybox <applet> ...`。

## 工作流

[.github/workflows/release.yml](.github/workflows/release.yml) 每周一 (北京时间 11:00) 自动运行:

1. 从上游镜像探测最新稳定版 tag
2. 已构建过的版本自动跳过
3. 4 个架构矩阵并行编译 (arm / arm64 / x86 / x86_64)
4. 汇总产物、生成 SHA256、发布 GitHub Release

也可在 Actions 页面手动触发 (workflow_dispatch)。

## 针对现代 NDK 的编译适配

新版 BusyBox 相对 `android_502_defconfig` 新增的 kconfig 符号默认开启,其中若干在 bionic 下必炸,工作流里已显式关闭并注明原因 (tc、mount、swapon/swapoff、adjtimex、seedrng、yescrypt 等)。详见 workflow 内注释。
