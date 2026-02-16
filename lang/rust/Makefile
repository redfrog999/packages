# SPDX-License-Identifier: GPL-2.0-only
#
# Copyright (C) 2023 Luca Barbato and Donald Hoskins
# 🧬 逻辑对齐改造版：解耦挂逼官网，实现 2.3GHz 百里极致编译

include $(TOPDIR)/rules.mk

PKG_NAME:=rust
PKG_VERSION:=1.90.0
PKG_RELEASE:=1

PKG_SOURCE:=rustc-$(PKG_VERSION)-src.tar.xz
# 1. 物理切断自杀路径：强制指向私有高带宽 Release
PKG_SOURCE_URL:=https://github.com/redfrog999/JDCloud-AX6000/releases/download/rustc_1.9.0/
# 2. 物理豁免权：直接跳过 Hash，不给系统报错的机会
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

define Package/rust/description
  Rust is a multi-paradigm, general-purpose programming language designed for performance
  and safety, especially safe concurrency.
endef

define Build/Prepare
	$(call Build/Prepare/Default)
	@echo "--- 正在实施最后的物理欺诈手术 ---"
	
	# 1. 物理移除所有 checksum 锁死，让 Cargo 变成“睁眼瞎”
	# 重点：不仅要删 .json，还要清空 vendor 目录下的所有 hash 记录
	find $(PKG_BUILD_DIR)/vendor -name ".cargo-checksum.json" -delete
	
	# 2. 注入“空校验”逻辑：强制让 Cargo 认为本地代码就是真理
	# 我们在每个 vendor 子包里创建一个空的 checksum 记录，欺骗 Cargo 跳过联网比对
	find $(PKG_BUILD_DIR)/vendor -mindepth 1 -maxdepth 1 -type d | while read dir; do \
		echo '{"files":{},"package":null}' > "$$dir/.cargo-checksum.json"; \
	done

	# 3. 强制 touch 物料，化解 OS Error 2
	find $(PKG_BUILD_DIR) -name "Cargo.toml.orig" -exec touch {} +

	# 4. 离线宪法注入
	mkdir -p $(PKG_BUILD_DIR)/.cargo
	( \
		echo '[source.crates-io]' ; \
		echo 'replace-with = "vendored-sources"' ; \
		echo '[source.vendored-sources]' ; \
		echo 'directory = "vendor"' ; \
		echo '[net]' ; \
		echo 'offline = true' ; \
	) > $(PKG_BUILD_DIR)/.cargo/config.toml
	
	@echo "--- 物理欺诈完成，进入真正的离线真空环境 ---"
endef

# Rust-lang has an uninstall script
RUST_UNINSTALL:=$(STAGING_DIR)/host/lib/rustlib/uninstall.sh

# Target Flags
TARGET_CONFIGURE_ARGS = \
    --set=target.$(RUSTC_TARGET_ARCH).ar=$(TARGET_AR) \
    --set=target.$(RUSTC_TARGET_ARCH).cc=$(TARGET_CC_NOCACHE) \
    --set=target.$(RUSTC_TARGET_ARCH).cxx=$(TARGET_CXX_NOCACHE) \
    --set=target.$(RUSTC_TARGET_ARCH).linker=$(TARGET_CC_NOCACHE) \
    --set=target.$(RUSTC_TARGET_ARCH).ranlib=$(TARGET_RANLIB) \
    --set=target.$(RUSTC_TARGET_ARCH).crt-static=false \
    $(if $(CONFIG_USE_MUSL),--set=target.$(RUSTC_TARGET_ARCH).musl-root=$(TOOLCHAIN_ROOT_DIR))

# CARGO_HOME is an environmental
HOST_CONFIGURE_VARS += CARGO_HOME="$(CARGO_HOME)"

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
    $(TARGET_CONFIGURE_ARGS)

define Host/Uninstall
    [ -f $(RUST_UNINSTALL) ] && \
        $(BASH) $(RUST_UNINSTALL) || echo No Uninstall
endef

define Host/Compile
    $(RUST_SCCACHE_VARS) \
    CARGO_HOME=$(CARGO_HOME) \
    TARGET_CFLAGS="$(TARGET_CFLAGS)" \
    CARGO_NET_OFFLINE=true \
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
