include $(TOPDIR)/rules.mk

PKG_NAME        := telemt
PKG_VERSION     := 3.3.32
PKG_RELEASE     := 1

PKG_SOURCE:=$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:=https://github.com/telemt/telemt/archive/refs/tags
PKG_HASH:=15072620d21149359d53148713af31e65cc031116e5bdc2fd9a982f5df749080

PKG_BUILD_DIR      := $(BUILD_DIR)/$(PKG_NAME)-$(PKG_VERSION)

PKG_MAINTAINER  := evl38784
PKG_LICENSE     := TELEMT-PL-3
PKG_LICENSE_FILES:=LICENSE
PKG_CPE_ID:=cpe:/a:telemt:telemt

PKG_BUILD_DEPENDS := rust/host
PKG_BUILD_PARALLEL := 1


include $(INCLUDE_DIR)/package.mk
include $(TOPDIR)/feeds/packages/lang/rust/rust-package.mk

CARGO_PROFILE_ENV := \
	CARGO_PROFILE_RELEASE_OPT_LEVEL=z \
	CARGO_PROFILE_RELEASE_LTO=true \
	CARGO_PROFILE_RELEASE_CODEGEN_UNITS=1 \
	CARGO_PROFILE_RELEASE_PANIC=abort \
	CARGO_PROFILE_RELEASE_STRIP=true

define Package/telemt
  SECTION   := net
  CATEGORY  := Network
  TITLE     := Telemt — Telegram MTProxy daemon
  URL       := https://github.com/telemt/telemt
  DEPENDS   := $(RUST_ARCH_DEPENDS) @(aarch64||arm||x86_64||i386||mipsel||mips||mips64el||riscv64)
endef

define Package/telemt/description
  Lightweight headless Telegram MTProxy daemon written in Rust + Tokio.
endef

define Package/telemt/conffiles
/etc/config/telemt
endef

define Build/Prepare
	$(call Build/Prepare/Default)
	sed -i 's/\r//;1s/^\xef\xbb\xbf//' $(CURDIR)/files/telemt.init
	sed -i 's/\r//;1s/^\xef\xbb\xbf//' $(CURDIR)/files/telemt.config
endef

define Build/Configure
	@echo "telemt: target arch=$(ARCH), Rust triple=$(RUST_TARGET)"
endef

define Package/telemt/install
	$(INSTALL_DIR)  $(1)/usr/bin
	$(INSTALL_BIN)  $(PKG_INSTALL_DIR)/bin/telemt $(1)/usr/bin/telemt

	$(INSTALL_DIR)  $(1)/etc/init.d
	$(INSTALL_BIN)  $(CURDIR)/files/telemt.init   $(1)/etc/init.d/telemt

	$(INSTALL_DIR)  $(1)/etc/config
	$(INSTALL_CONF) $(CURDIR)/files/telemt.config $(1)/etc/config/telemt
endef

define Package/telemt/postinst
#!/bin/sh
if pidof telemt >/dev/null 2>&1; then
    killall -9 telemt 2>/dev/null || true
fi

if [ -d "/etc/telemt" ]; then
    mv /etc/telemt /etc/telemt.disabled_by_luci 2>/dev/null || true
fi

old_metrics=$$(uci -q get telemt.general.metrics_port 2>/dev/null)
if [ "$$old_metrics" = "9091" ] || [ "$$old_metrics" = "9090" ]; then
    uci set telemt.general.metrics_port='9092'
    uci set telemt.general.api_port='9091'
    uci commit telemt 2>/dev/null || true
fi

[ -x /etc/init.d/telemt ] && /etc/init.d/telemt enable 2>/dev/null || true
exit 0
endef

define Package/telemt/prerm
#!/bin/sh
[ -x /etc/init.d/telemt ] && /etc/init.d/telemt disable 2>/dev/null || true
[ -x /etc/init.d/telemt ] && /etc/init.d/telemt stop   2>/dev/null || true
exit 0
endef

define Package/telemt/postrm
#!/bin/sh
rm -f /var/etc/telemt.toml
rm -f /var/etc/telemt.version
rm -f /tmp/telemt_stats.txt
exit 0
endef

$(eval $(call RustBinPackage,telemt))
$(eval $(call BuildPackage,telemt))
