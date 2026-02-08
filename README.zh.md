# openwrt-webauthn-helper

[English](README.md) 简体中文

[![许可证](https://img.shields.io/badge/许可证-Apache--2.0-blue.svg)](LICENSE)

## 简介

openwrt-webauthn-helper 是 luci-app-webauthn 的依赖包，为 OpenWrt 路由器提供 WebAuthn/FIDO2 CLI 辅助功能。

本仓库会自动监测上游 [webauthn-helper](https://github.com/Tokisaki-Galaxy/webauthn-helper) 仓库的新版本发布，并为多个架构自动构建 OpenWrt 软件包（ipk/apk 格式）。

## 安装方法

### 从 Release 下载安装

1. 前往 [Release 页面](https://github.com/Tokisaki-Galaxy/openwrt-webauthn-helper/releases) 下载适合您设备架构的 ipk 文件

2. 将下载的 ipk 文件上传到您的 OpenWrt 设备

3. 安装：

```bash
opkg install /path/to/webauthn-helper_xxx.ipk
```

## 自动发布流程

本仓库使用 GitHub Actions 自动完成以下工作：

1. **监测上游发布**：每 6 小时检查一次 [webauthn-helper](https://github.com/Tokisaki-Galaxy/webauthn-helper) 的新版本发布
2. **更新版本**：自动更新 Makefile 中的版本号和校验和
3. **构建软件包**：为多个 OpenWrt 架构编译软件包（包括 ipk 和 apk 格式）
4. **创建发布**：发布新版本，发布说明与上游仓库保持一致

发布说明会从上游仓库同步，确保两个项目之间的一致性。

**注意**：为了让自动触发工作流正常运行，需要配置个人访问令牌（PAT）。详见 `.github/WORKFLOW.md` 中的设置说明。

## 许可证

本项目采用 Apache-2.0 许可证。

## 维护者

- [Tokisaki-Galaxy](https://github.com/Tokisaki-Galaxy)
