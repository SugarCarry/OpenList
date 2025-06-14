#
# Copyright (C) 2015-2016 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v3.
#

include $(TOPDIR)/rules.mk

PKG_NAME:=openlist
PKG_VERSION:=beta
PKG_WEB_VERSION:=rolling
PKG_RELEASE:=1

PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:= \
  https://codeload.github.com/OpenListTeam/OpenList/tar.gz/v$(PKG_VERSION)? \
  https://codeload.github.com/OpenListTeam/OpenList/tar.gz/$(PKG_VERSION)?
PKG_HASH:=f598b191566d9e0c66ccbf37b75ec38a5e203dc92fdd76065d6efec84e9ad7e1

PKG_BUILD_DIR:=$(BUILD_DIR)/OpenList-$(PKG_VERSION)

PKG_LICENSE:=GPL-3.0
PKG_LICENSE_FILE:=LICENSE
PKG_MAINTAINER:=sbwml <admin@cooluc.com>

define Download/$(PKG_NAME)-web
  FILE:=openlist-frontend-dist-v0.0.0-f89d0c8.tar.gz
  URL_FILE:=openlist-frontend-dist-v0.0.0-f89d0c8.tar.gz
  URL:=https://github.com/OpenListTeam/OpenList-Frontend/releases/download/$(PKG_WEB_VERSION)/
  HASH:=92895da17f0bcae74129d5bc60627fc635374828511a341d8773140913545212
endef

PKG_BUILD_DEPENDS:=golang/host
PKG_BUILD_PARALLEL:=1
PKG_USE_MIPS16:=0
PKG_BUILD_FLAGS:=no-mips16

GO_PKG:=github.com/OpenListTeam/OpenList
GO_PKG_LDFLAGS:= \
	-X '$(GO_PKG)/internal/conf.BuiltAt=$(shell date '+%Y-%m-%d %H:%M:%S %z')' \
	-X '$(GO_PKG)/internal/conf.GoVersion=$(shell $(STAGING_DIR_HOSTPKG)/bin/go version | sed 's/go version //')' \
	-X '$(GO_PKG)/internal/conf.GitAuthor=The OpenList Projects Contributors <noreply@openlist.team>' \
	-X '$(GO_PKG)/internal/conf.GitCommit=tarball/$(shell echo $(PKG_HASH) | cut -c 1-7)' \
	-X '$(GO_PKG)/internal/conf.Version=v$(PKG_VERSION) (OpenWrt $(ARCH_PACKAGES))' \
	-X '$(GO_PKG)/internal/conf.WebVersion=$(PKG_WEB_VERSION)'
ifneq ($(CONFIG_ARCH_64BIT),y)
  GO_PKG_EXCLUDES:=drivers/lark
endif

include $(INCLUDE_DIR)/package.mk
include $(TOPDIR)/feeds/packages/lang/golang/golang-package.mk

define Package/$(PKG_NAME)
  SECTION:=net
  CATEGORY:=Network
  SUBMENU:=Web Servers/Proxies
  TITLE:=A file list program that supports multiple storage
  URL:=https://openlist.team/
  DEPENDS:=$(GO_ARCH_DEPENDS) +ca-bundle
endef

define Package/$(PKG_NAME)/conffiles
/etc/openlist
/etc/config/openlist
endef

define Package/$(PKG_NAME)/description
  A file list program that supports multiple storage, powered by Gin and Solidjs.
endef

ifeq ($(ARCH),arm)
  ARM_CPU_FEATURES:=$(word 2,$(subst +,$(space),$(call qstrip,$(CONFIG_CPU_TYPE))))
  ifeq ($(ARM_CPU_FEATURES),)
    TARGET_CFLAGS:=
    TARGET_LDFLAGS:=
  endif
endif

ifneq ($(CONFIG_USE_MUSL),)
  TARGET_CFLAGS += -D_LARGEFILE64_SOURCE
endif

define Build/Prepare
	$(call Build/Prepare/Default)
	$(TAR) --strip-components=1 -C $(PKG_BUILD_DIR)/public/dist -xzf $(DL_DIR)/openlist-frontend-dist-v0.0.0-f89d0c8.tar.gz
	$(SED) 's_https://docs.oplist.org/logo.png_/assets/logo.png_g' $(PKG_BUILD_DIR)/public/dist/index.html
	$(SED) 's_https://docs.oplist.org/logo.svg_/assets/logo.svg_g' $(PKG_BUILD_DIR)/public/dist/index.html
endef

define Package/$(PKG_NAME)/install
	$(call GoPackage/Package/Install/Bin,$(PKG_INSTALL_DIR))
	$(INSTALL_DIR) $(1)/usr/bin
	$(INSTALL_BIN) $(PKG_INSTALL_DIR)/usr/bin/OpenList $(1)/usr/bin/openlist
	$(INSTALL_DIR) $(1)/etc/config $(1)/etc/init.d $(1)/etc/openlist
	$(INSTALL_CONF) $(CURDIR)/files/openlist.config $(1)/etc/config/openlist
	$(INSTALL_BIN) $(CURDIR)/files/openlist.init $(1)/etc/init.d/openlist
	$(INSTALL_DATA) $(CURDIR)/files/data.db $(1)/etc/openlist/data.db
endef

$(eval $(call Download,$(PKG_NAME)-web))
$(eval $(call BuildPackage,$(PKG_NAME)))
