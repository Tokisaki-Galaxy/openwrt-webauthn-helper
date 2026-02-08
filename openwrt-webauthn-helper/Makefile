include $(TOPDIR)/rules.mk

PKG_NAME:=webauthn-helper
PKG_VERSION:=1.0.1
PKG_RELEASE:=1

PKG_LICENSE:=Apache-2.0
PKG_MAINTAINER:=Tokisaki-Galaxy

include $(INCLUDE_DIR)/package.mk

# 使用OpenWrt的Rust映射机制
include $(TOPDIR)/feeds/packages/lang/rust/rust-values.mk

# 二进制文件的文件名和URL
RUST_WEBAUTHN_HELPER_VERSION:=1.0.1
RUST_BINARY_FILENAME:=webauthn-helper-$(RUST_WEBAUTHN_HELPER_VERSION)-$(RUSTC_TARGET_ARCH).tar.gz


# 修正下载地址定义方式
PKG_SOURCE:=$(RUST_BINARY_FILENAME)
PKG_SOURCE_URL:=https://github.com/Tokisaki-Galaxy/webauthn-helper/releases/download/v$(RUST_WEBAUTHN_HELPER_VERSION)

# 修正二进制文件校验值
ifeq ($(ARCH),aarch64)
	PKG_HASH:=8449ad6688c0a912e135a5dda43af8d38325c851f3549a51bf953ccc77b567b5
else ifeq ($(ARCH),arm) # update.sh skip this tag, do not delete this comment
	# Ref https://github.com/openwrt/packages/blob/master/lang/rust/rust-values.mk

	ARM_CPU_TYPE:=arm-armel
	ifeq ($(CONFIG_arm_v6)$(CONFIG_arm_v7),)
		ARM_CPU_TYPE:=$(subst arm-,armv5te-,$(ARM_CPU_TYPE))
	else ifeq ($(CONFIG_arm_v7),y)
		ARM_CPU_TYPE:=$(subst arm-,armv7-,$(ARM_CPU_TYPE))
	endif

	ifeq ($(CONFIG_HAS_FPU),y)
		ARM_CPU_TYPE:=$(subst -armel,-armhf,$(ARM_CPU_TYPE))
	endif

	ifeq ($(ARM_CPU_TYPE),arm-armel)
		PKG_HASH:=b907837288f447768ba88d3fcf3dd729c05dd3932b4228ff7c08d649f52cfaec
	else ifeq ($(ARM_CPU_TYPE),arm-armhf)
		PKG_HASH:=4cea2fc10da886412ac239fee061589e4e4a9936f767974b66a5c6450b5fc3aa
	else ifeq ($(ARM_CPU_TYPE),armv5te-armel)
		PKG_HASH:=ba593cdfced72e20df02f4c41df38a016d19fd27b3c210dc2dbd492711df9e79
	else ifeq ($(ARM_CPU_TYPE),armv7-armel)
		PKG_HASH:=da812f5a94e6115b9c7023cf8a54be3622c3bbb9f976484f9dbe189cbde16b1d
	else ifeq ($(ARM_CPU_TYPE),armv7-armhf)
		PKG_HASH:=5b7319a562eb81668e00394b1209a3141f3b62aa3c953c66e7e145b016a8c9cd
	else
		PKG_HASH:=dummy
	endif
else ifeq ($(ARCH),mips)
	PKG_HASH:=297090b28dd309cb9930abb86df7c3754f2a9e4410e5a36fc0ba651d1c8a06dd
else ifeq ($(ARCH),mipsel)
	PKG_HASH:=60f6ef5642286463de17522f0d0b8d6c30c589296b22dc5e0265b3953df87794
else ifeq ($(ARCH),riscv64)
	PKG_HASH:=0f054618a27465d3c7b4a04589a97aee166d54b385cbd0a8647753b62e02610b
else ifeq ($(ARCH),x86_64)
	PKG_HASH:=123f989fd05ff0294053d9dfe843aa2cb50c8f59ba39dc16dc9ce6deaa95b9d6
else
	PKG_HASH:=dummy
endif

define Package/$(PKG_NAME)
	SECTION:=net
	CATEGORY:=Network
	TITLE:=WebAuthn/FIDO2 CLI helper for OpenWrt
endef

define Package/$(PKG_NAME)/description
	WebAuthn/FIDO2 CLI helper for OpenWrt routers
endef

define Build/Prepare
	mkdir -p $(PKG_BUILD_DIR)/bin
	gzip -dc "$(DL_DIR)/$(PKG_SOURCE)" | tar -C $(PKG_BUILD_DIR)/bin --strip-components=1 -xf -
	$(Build/Patch)
endef

define Build/Compile
	# nothing to do
endef

define Package/$(PKG_NAME)/install
	$(INSTALL_DIR) $(1)/usr/libexec
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/bin/webauthn-helper $(1)/usr/libexec/
endef

$(eval $(call BuildPackage,webauthn-helper))
