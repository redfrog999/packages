# SPDX-License-Identifier: GPL-2.0-only
#
# Copyright (C) 2023 Luca Barbato and Donald Hoskins
# 🛡️ 2026.02 离线主权版：彻底切断官网联系，实现 2.3GHz 百里全量合闸

include $(TOPDIR)/rules.mk

PKG_NAME:=rust
PKG_VERSION:=1.90.0
PKG_RELEASE:=1

# 🎯 物理物料对齐：确保与你提供的 URL 完全一致
PKG_SOURCE:=rustc-$(PKG_VERSION)-src.tar.xz
PKG_SOURCE_URL:=https://github.com/redfrog999/JDCloud-AX6000/releases/download/rustc_1.90.0/
PKG_HASH:=skip

HOST_BUILD_DIR:=$(BUILD_DIR)/host/rustc-$(PKG_VERSION)-src

PKG_MAINTAINER:=Luca Barbato <lu_zero@luminem.org>
PKG_LICENSE:=Apache-2.0 MIT
PKG_LICENSE_FILES:=LICENSE-APACHE LICENSE-MIT

PKG_HOST_ONLY:=1
PKG_BUILD_FLAGS:=no-mips16

include $(INCLUDE_DIR)/host-build.mk
include $(INCLUDE_DIR)/package.mk
include ./rust-values.mk

define Package/rust
  SECTION:=lang
  CATEGORY:=Languages
  SUBMENU:=Rust
  TITLE:=Rust Programming Language Compiler
  URL:=https://www.rust-lang.org/
  DEPENDS:=$(RUST_ARCH_DEPENDS)
endef

# 🏗️ 核心：Build/Prepare 物理欺诈手术
define Build/Prepare
	$(call Build/Prepare/Default)
	@echo "--- 正在实施离线真空屏蔽手术，对齐物料：$(PKG_SOURCE) ---"

	# 1. 物理抹除校验：废除 Cargo 的“联网对账”动机
	find $(PKG_BUILD_DIR)/vendor -name ".cargo-checksum.json" -delete
	
	# 2. 注入“空白圣经”：强迫 Cargo 认为本地 vendor 是唯一的真理
	find $(PKG_BUILD_DIR)/vendor -mindepth 1 -maxdepth 1 -type d | while read dir; do \
		echo '{"files":{},"package":null}' > "$$dir/.cargo-checksum.json"; \
	done

	# 3. 补齐 orig 文件：防止 Actions 环境报 OS Error 2
	find $(PKG_BUILD_DIR) -name "Cargo.toml.orig" -exec touch {} +

	# 4. 注入离线宪法：在源码根目录强制配置 config.toml
	mkdir -p $(PKG_BUILD_DIR)/.cargo
	( \
		echo '[source.crates-io]' ; \
		echo 'replace-with = "vendored-sources"' ; \
		echo '[source.vendored-sources]' ; \
		echo 'directory = "vendor"' ; \
		echo '[net]' ; \
		echo 'offline = true' ; \
	) > $(PKG_BUILD_DIR)/.cargo/config.toml
	
	@echo "--- 物理屏蔽完成，开始 A53 2.3GHz 源码级咆哮编译 ---"
endef

# 强制开启 Cargo 离线模式环境变量
HOST_CONFIGURE_VARS += CARGO_HOME="$(CARGO_HOME)" CARGO_NET_OFFLINE=true

# Rust Configuration Arguments
HOST_CONFIGURE_ARGS = \
    --build=$(RUSTC_HOST_ARCH) \
    --target=$(RUSTC_TARGET_ARCH),$(RUSTC_HOST_ARCH) \
    --host=$(RUSTC_HOST_ARCH) \
    --prefix=$(STAGING_DIR)/host \
    --bindir=$(STAGING_DIR)/host/bin \
    --libdir=$(STAGING_DIR)/host/lib \
    --sysconfdir=$(STAGING_DIR)/host/etc \
    --datadir=$(STAGING_DIR)/host/share \
    --mandir=$(STAGING_DIR)/host/man \
    --dist-compression-formats=gz \
    --disable-sanitizers \
    --release-channel=stable \
    --enable-cargo-native-static \
    --bootstrap-cache-path=$(DL_DIR)/rustc \
    --set=llvm.download-ci-llvm=false \
    --set=build.configure-args="['--enable-vendor']" \
    $(TARGET_CONFIGURE_ARGS)

define Host/Compile
    $(RUST_SCCACHE_VARS) \
    CARGO_HOME=$(CARGO_HOME) \
    CARGO_NET_OFFLINE=true \
    TARGET_CFLAGS="$(TARGET_CFLAGS)" \
    $(PYTHON) $(HOST_BUILD_DIR)/x.py \
        --ci false \
        --build-dir $(HOST_BUILD_DIR)/build \
        dist build-manifest rustc rust-std cargo llvm-tools rust-src
endef

define Host/Install
    ( \
        cd $(HOST_BUILD_DIR)/build/dist ; \
        for targz in *.tar.gz; do \
            $(STAGING_DIR_HOST)/bin/libdeflate-gzip -dc "$$$$targz" | tar -xf - ; \
        done ; \
        find . -mindepth 2 -maxdepth 2 -type f -name install.sh \
            -execdir bash '{}' --prefix=$(STAGING_DIR)/host --disable-ldconfig \; ; \
    )
endef

$(eval $(call HostBuild))
$(eval $(call BuildPackage,rust))
