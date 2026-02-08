# openwrt-webauthn-helper

English | [简体中文](README.zh.md)


[![License](https://img.shields.io/badge/License-Apache--2.0-blue.svg)](LICENSE)

## Introduction

openwrt-webauthn-helper is a dependency package for luci-app-webauthn, providing WebAuthn/FIDO2 CLI helper functionality for OpenWrt routers.

This repository automatically monitors the upstream [webauthn-helper](https://github.com/Tokisaki-Galaxy/webauthn-helper) repository for new releases and builds OpenWrt packages (ipk/apk format) for multiple architectures.

## Installation

### Install from Release

1. Visit the [Release page](https://github.com/Tokisaki-Galaxy/openwrt-webauthn-helper/releases) to download the ipk file suitable for your device architecture

2. Upload the downloaded ipk file to your OpenWrt device

3. Install:

```bash
opkg install /path/to/webauthn-helper_xxx.ipk
```

## Automated Release Process

This repository uses GitHub Actions to automatically:

1. **Monitor upstream releases**: Checks for new releases from [webauthn-helper](https://github.com/Tokisaki-Galaxy/webauthn-helper) every 6 hours
2. **Update version**: Updates the Makefile with the latest version and checksums
3. **Build packages**: Compiles packages for multiple OpenWrt architectures (both ipk and apk formats)
4. **Create release**: Publishes a new release with the same description as the upstream release

The release descriptions are synchronized from the upstream repository, ensuring consistency across both projects.

## License

This project is licensed under the Apache-2.0 License.

## Maintainer

- [Tokisaki-Galaxy](https://github.com/Tokisaki-Galaxy)