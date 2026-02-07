include $(TOPDIR)/rules.mk

PKG_NAME:=webauthn-helper
PKG_VERSION:=1.0.0
PKG_RELEASE:=1

PKG_LICENSE:=MIT
PKG_MAINTAINER:=Tokisaki-Galaxy

include $(INCLUDE_DIR)/package.mk

# 使用OpenWrt的Rust映射机制
include $(TOPDIR)/feeds/packages/lang/rust/rust-values.mk

# 二进制文件的文件名和URL
RUST_WEBAUTHN_HELPER_VERSION:=1.0.0
RUST_BINARY_FILENAME:=webauthn-helper-$(RUST_WEBAUTHN_HELPER_VERSION)-$(RUSTC_TARGET_ARCH).tar.gz


# 修正下载地址定义方式
PKG_SOURCE:=$(RUST_BINARY_FILENAME)
PKG_SOURCE_URL:=https://github.com/Tokisaki-Galaxy/webauthn-helper/releases/download/v$(RUST_WEBAUTHN_HELPER_VERSION)

# 修正二进制文件校验值
ifeq ($(ARCH),aarch64)
	PKG_HASH:=99915c0623a5c4f2907acb20b48edcd0ce66916dc8e236e515bf45c5427fd4fe
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
		PKG_HASH:=eb5f8b053901c6e2cf3f7bd1b00add544d5105747751cf32473f8a821aa1b187
	else ifeq ($(ARM_CPU_TYPE),arm-armhf)
		PKG_HASH:=661ef670ac50dcf92574fe6813677afba29766e5d5ce80b612b358ce6d40d63f
	else ifeq ($(ARM_CPU_TYPE),armv5te-armel)
		PKG_HASH:=ac52bd6b42f2d06103b5fdd03b627e17b28f8660b994058692162d105f83f7d9
	else ifeq ($(ARM_CPU_TYPE),armv7-armel)
		PKG_HASH:=f04ef3a8656f070091df40a10ba420707b6f53e09790aca3db0ed50bad6fc508
	else ifeq ($(ARM_CPU_TYPE),armv7-armhf)
		PKG_HASH:=021a01ff9bac0a9865c73dcc50f6239e2d3b3be4491c82f96365e84c4e0c222b
	else
		PKG_HASH:=dummy
	endif
else ifeq ($(ARCH),mips)
	PKG_HASH:=e8eb7b73cde68d423297a9f73a25989f872bc72714941e516f818afb8fca5375
else ifeq ($(ARCH),mipsel)
	PKG_HASH:=65c59b7e5f00ce8594c7f791900b2b26765b2c65510d124d2529f30d8a1b7624
else ifeq ($(ARCH),riscv64)
	PKG_HASH:=b5e599368fe660a8451b9347326b74c882642b4b425c761d00d6e27607a0d645
else ifeq ($(ARCH),x86_64)
	PKG_HASH:=29972d27014df1c21ef206506c93eae6300279fb1309f6b2cc2f9636be32e93e
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
