## This file is part of the sigrok-util project.
## Copyright (C) 2026 Uwe Norinq <qq984487046@qq.com>
##
## This program is free software; you can redistribute it and/or modify
## it under the terms of the GNU General Public License as published by
## the Free Software Foundation; either version 2 of the License, or
## (at your option) any later version.
##
## This program is distributed in the hope that it will be useful,
## but WITHOUT ANY WARRANTY; without even the implied warranty of
## MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
## GNU General Public License for more details.
##
## You should have received a copy of the GNU General Public License
## along with this program; if not, see <http://www.gnu.org/licenses/>.


# 使用 GNU Make：make -C cross-compile/mingw TARGET=i686 all
# 64 位调试版：make -C cross-compile/mingw TARGET=x86_64 DEBUG=1 all
# PARALLEL 控制组件内部并行度；设为空时继承外层 make -jN。
.DEFAULT_GOAL := all
.DELETE_ON_ERROR:
SHELL := /bin/sh

PATH_MAKEFILE := $(abspath $(dir $(lastword $(MAKEFILE_LIST))))
BUILD_ROOT ?= $(abspath $(PATH_MAKEFILE)/../..)/build

# Build target: "i686" (32bit) or "x86_64" (64bit).
TARGET ?= x86_64

# The base path prefix where the cross-compiled packages will be installed.
PREFIXBASE ?= $(BUILD_ROOT)/mingw_$(FLAVOR)_$(SUFFIX)/sr_ming

# The base path prefix where to build packages.
BUILDBASE ?= $(BUILD_ROOT)/mingw_$(FLAVOR)_$(SUFFIX)/obj

# Edit this to control verbose build output.
# V = V=1 VERBOSE=1
V ?=

# Edit this to enable/disable/modify parallel compiles.
PARALLEL ?= -j4

# Edit this to enable/disable debug builds.
DEBUG ?= 0

# Optionally show some progress as the script executes.
# ECHO = true
ECHO ?= echo

# You usually don't need to change anything below this line.

MXE_REPO ?= git@github.com:mxe/mxe.git
SOURCES ?= $(abspath $(PATH_MAKEFILE)/../../dependencies)

ifeq ($(filter $(TARGET),i686 x86_64),)
$(error TARGET 必须是 i686 或 x86_64)
endif
ifeq ($(filter $(DEBUG),0 1),)
$(error DEBUG 必须是 0 或 1)
endif
BUILD_TYPE := $(if $(filter 1,$(DEBUG)),Debug,Release)
FLAVOR := $(if $(filter 1,$(DEBUG)),debug,release)
SUFFIX := $(if $(filter i686,$(TARGET)),32,64)
PLATFORMDIR := $(abspath $(BUILD_ROOT)/mingw_$(FLAVOR)_$(SUFFIX))
BUILDDIR := $(abspath $(BUILDBASE))
PREFIX := $(abspath $(PREFIXBASE))
DOWNLOADDIR ?= $(abspath $(PREFIX)/../downloads)
DISTDIR ?= $(abspath $(PREFIX)/../contrib)
PORTABLEDIR ?= $(BUILDDIR)/portable
# MXE 保留原有默认存放位置，与其他构建产物分开管理。
MXE_BUILDDIR := $(PATH_MAKEFILE)/build_$(FLAVOR)_$(SUFFIX)/mxe

# The path where your MXE directory is located.
# 优先检测 MXE_BUILDDIR，再检测 HOME/mxe；都不存在时浅克隆到 MXE_BUILDDIR。
# MXE ?= $(HOME)/mxe-git
MXE ?= $(if $(wildcard $(MXE_BUILDDIR)/.),$(MXE_BUILDDIR),$(if $(wildcard $(HOME)/mxe/.),$(HOME)/mxe,$(MXE_BUILDDIR)))


TRIPLET := $(TARGET)-w64-mingw32.static.posix
export PATH := $(MXE)/usr/bin:$(PATH)
export PKG_CONFIG_PATH_$(subst .,_,$(subst -,_,$(TRIPLET))) := $(PREFIX)/lib/pkgconfig:$(MXE)/usr/$(TRIPLET)/lib/pkgconfig
TOOLCHAIN_FLAGS := -B$(MXE)/usr/bin/$(TRIPLET)-
CONFIGURE = CC="$(TRIPLET)-gcc $(TOOLCHAIN_FLAGS)" CXX="$(TRIPLET)-g++ $(TOOLCHAIN_FLAGS)" --host=$(TRIPLET) --prefix="$(PREFIX)" CPPFLAGS=-D__printf__=__gnu_printf__
STATIC := --disable-shared --enable-static

WGET ?= wget -c --quiet
# git clone --depth 1 用于首次获取仓库；已有目录直接复用，不自动更新本地工具链。
GIT_CLONE ?= git clone --depth 1

PYTHON_VERSION := 3.13.17
PYTHON_ARCH := $(if $(filter x86_64,$(TARGET)),amd64,win32)
PYTHON_NUGET := $(if $(filter x86_64,$(TARGET)),python,pythonx86)
PYTHON_URL ?= https://www.python.org/ftp/python/$(PYTHON_VERSION)

.PHONY: all
all: pulseview sigrok-cli

# 已初始化的子模块直接复用，避免普通构建切换本地提交或改动。
COMPONENTS := libserialport libsigrok libsigrokdecode sigrok-firmware sigrok-firmware-fx2lafw sigrok-dumps sigrok-cli pulseview
.PHONY: submodules-init pull-sources $(addsuffix -source,$(COMPONENTS)) $(addsuffix -pull,$(COMPONENTS))
pull-sources: $(addsuffix -pull,$(COMPONENTS))

# 先统一注册缺失子模块，避免并行初始化争用父仓库配置锁。
submodules-init:
	@set --; for component in $(COMPONENTS); do \
		if [ ! -e "$(SOURCES)/$$component/.git" ]; then \
			set -- "$$@" "$(notdir $(SOURCES))/$$component"; \
		fi; \
	done; \
	if [ "$$#" -gt 0 ]; then \
		git -C "$(abspath $(SOURCES)/..)" submodule init -- "$$@"; \
	fi

$(addsuffix -source,$(COMPONENTS)): %-source: | submodules-init
	@if [ ! -e "$(SOURCES)/$*/.git" ]; then \
		git -C "$(abspath $(SOURCES)/..)" submodule update --init --depth=1 -- "$(notdir $(SOURCES))/$*"; \
	fi

# 显式更新到 origin 默认分支；只允许快进，保留本地提交和工作区改动。
$(addsuffix -pull,$(COMPONENTS)): %-pull: %-source
	git -C "$(SOURCES)/$*" fetch --depth=1 origin HEAD
	@cd "$(SOURCES)/$*" && \
		while ! git merge-base --is-ancestor HEAD FETCH_HEAD && \
			[ "$$(git rev-parse --is-shallow-repository)" = true ]; do \
			git fetch --deepen=32 origin HEAD || exit 1; \
		done && git merge --ff-only FETCH_HEAD

libserialport: libserialport-source
libsigrok: libsigrok-source
libsigrokdecode: libsigrokdecode-source
sigrok-firmware: sigrok-firmware-source
sigrok-firmware-fx2lafw: sigrok-firmware-fx2lafw-source
sigrok-dumps: sigrok-dumps-source
sigrok-cli-build: sigrok-cli-source
pulseview-build: pulseview-source

# 准备 MXE 源码并检查交叉编译器；已有目录直接复用。
.PHONY: mxe
mxe:
	@$(ECHO) "preparing MXE source ..."
	@if [ ! -d "$(MXE)" ]; then \
		mkdir -p "$(dir $(MXE))" && $(GIT_CLONE) "$(MXE_REPO)" "$(MXE)"; \
	fi

	@for tool in $(TRIPLET)-gcc; do \
		command -v "$$tool" >/dev/null || { \
			echo "缺少 MXE 工具：$$tool（TARGET=$(TARGET)，MXE=$(MXE)）" >&2; \
			echo "MXE 源码目录存在不代表工具链已编译，请先安装对应架构的 MXE 工具链，或用 TARGET 指定已安装的架构。" >&2; \
			exit 1; \
		}; \
	done
	@mkdir -p "$(PREFIX)/lib/pkgconfig"
	@if [ -f "$(MXE)/usr/$(TRIPLET)/lib/pkgconfig/hidapi.pc" ]; then \
		sed '/^Libs:/s/$$/ -lsetupapi/' "$(MXE)/usr/$(TRIPLET)/lib/pkgconfig/hidapi.pc" > "$(PREFIX)/lib/pkgconfig/hidapi.pc"; \
	fi

# 源码由子模块提供，组件目录仅存放源码外构建产物。
define prepare_build
	@$(ECHO) "component $(1) ..."
	mkdir -p "$(BUILDDIR)/$(1)"
endef

# Autotools 引导文件在源码目录生成，配置和编译在构建目录执行。
define configure_source
	$(call prepare_build,$(1))
	cd "$(SOURCES)/$(1)" && ./autogen.sh
	cd "$(BUILDDIR)/$(1)" && "$(SOURCES)/$(1)/configure" $(2)
endef

.PHONY: python
python: mxe $(DOWNLOADDIR)/python-$(PYTHON_VERSION)-embed-$(PYTHON_ARCH).zip $(DOWNLOADDIR)/$(PYTHON_NUGET)-$(PYTHON_VERSION).nupkg
	@$(ECHO) "preparing Python $(PYTHON_VERSION) dependency ..."
	rm -rf "$(PREFIX)/Python/include" "$(PREFIX)/Python/libs" "$(PREFIX)/Python/runtime" "$(BUILDDIR)/python-sdk"
	mkdir -p "$(PREFIX)/Python/runtime" "$(PREFIX)/Python/libs" "$(BUILDDIR)/python-sdk"
	unzip -oq "$(DOWNLOADDIR)/$(PYTHON_NUGET)-$(PYTHON_VERSION).nupkg" 'tools/include/*' -d "$(BUILDDIR)/python-sdk"
	cp -R "$(BUILDDIR)/python-sdk/tools/include" "$(PREFIX)/Python/"
ifeq ($(TARGET),x86_64)
	patch -p1 "$(PREFIX)/Python/include/pyconfig.h" < "$(PATH_MAKEFILE)/pyconfig.patch"
endif
	unzip -oq "$(DOWNLOADDIR)/python-$(PYTHON_VERSION)-embed-$(PYTHON_ARCH).zip" '*.dll' '*.pyd' 'python313.zip' 'python313._pth' 'LICENSE.txt' -d "$(PREFIX)/Python/runtime"
	mv "$(PREFIX)/Python/runtime/LICENSE.txt" "$(PREFIX)/Python/runtime/Python-LICENSE.txt"
	cd "$(PREFIX)/Python/runtime" && "$(MXE)/usr/$(TRIPLET)/bin/gendef" python313.dll
	$(TRIPLET)-dlltool --dllname python313.dll --def "$(PREFIX)/Python/runtime/python313.def" --output-lib "$(PREFIX)/Python/libs/libpython313.a"
	rm "$(PREFIX)/Python/runtime/python313.def"
	mkdir -p "$(PREFIX)/lib/pkgconfig"
	printf '%s\n' 'prefix=$(PREFIX)/Python' 'libdir=$${prefix}/libs' 'includedir=$${prefix}/include' 'Name: Python' 'Description: Python embedding library' 'Version: $(PYTHON_VERSION)' 'Libs: -L$${libdir} -lpython313' 'Cflags: -I$${includedir}' > "$(PREFIX)/lib/pkgconfig/python3.pc"
	cp "$(PREFIX)/lib/pkgconfig/python3.pc" "$(PREFIX)/lib/pkgconfig/python3-embed.pc"
	@for runtime_file in "$(PREFIX)/Python/runtime/"*; do \
		printf 'Delete "$$INSTDIR\\%s"\n' "$${runtime_file##*/}"; \
	done > "$(PREFIX)/Python/runtime-uninstall.nsh"


$(DOWNLOADDIR)/python-$(PYTHON_VERSION)-embed-$(PYTHON_ARCH).zip: DOWNLOAD_URL = $(PYTHON_URL)/$(@F)
$(DOWNLOADDIR)/$(PYTHON_NUGET)-$(PYTHON_VERSION).nupkg: DOWNLOAD_URL = https://www.nuget.org/api/v2/package/$(PYTHON_NUGET)/$(PYTHON_VERSION)
$(DOWNLOADDIR)/zadig.exe: DOWNLOAD_URL := https://github.com/pbatard/libwdi/releases/download/b721/zadig-2.4.exe
$(DOWNLOADDIR)/zadig_xp.exe: DOWNLOAD_URL := https://github.com/pbatard/libwdi/releases/download/v1.2.5/zadig_xp-2.2.exe

$(DOWNLOADDIR)/python-$(PYTHON_VERSION)-embed-$(PYTHON_ARCH).zip $(DOWNLOADDIR)/$(PYTHON_NUGET)-$(PYTHON_VERSION).nupkg $(DOWNLOADDIR)/zadig.exe $(DOWNLOADDIR)/zadig_xp.exe:
	mkdir -p "$(@D)"
	$(WGET) "$(DOWNLOAD_URL)" -O "$@.part" && mv "$@.part" "$@"

.PHONY: zadig firmware runtime
zadig: $(DOWNLOADDIR)/zadig.exe $(DOWNLOADDIR)/zadig_xp.exe

firmware: sigrok-firmware sigrok-firmware-fx2lafw
runtime: libsigrokdecode zadig firmware

# 编译目标仅依赖所需库；固件和示例数据由打包目标准备。
libserialport: mxe
libsigrok: libserialport mxe
libsigrokdecode: mxe python
sigrok-cli-build: libsigrok libsigrokdecode mxe
pulseview-build: libsigrok libsigrokdecode mxe


# 固件使用宿主机工具链，不能传入 Windows 的 --host 参数。

# private 防止配置参数沿组件依赖链传递。
libserialport libsigrok libsigrokdecode: private CONFIGURE_ARGS = $(CONFIGURE) $(STATIC)
sigrok-firmware sigrok-firmware-fx2lafw: private CONFIGURE_ARGS = --prefix="$(PREFIX)"

.PHONY: libserialport libsigrok libsigrokdecode sigrok-firmware sigrok-firmware-fx2lafw
libserialport libsigrok libsigrokdecode sigrok-firmware sigrok-firmware-fx2lafw:
	$(call configure_source,$@,$(CONFIGURE_ARGS))
	$(MAKE) -C "$(BUILDDIR)/$@" $(PARALLEL) $(V)
	$(MAKE) -C "$(BUILDDIR)/$@" install $(V)


.PHONY: sigrok-dumps
sigrok-dumps:
	$(MAKE) -C "$(SOURCES)/$@" install PREFIX="$(PREFIX)" $(V)


.PHONY: sigrok-cli-build
sigrok-cli-build:
	$(call configure_source,sigrok-cli,$(CONFIGURE))
	$(MAKE) -C "$(BUILDDIR)/sigrok-cli" $(PARALLEL) $(V)
	$(MAKE) -C "$(BUILDDIR)/sigrok-cli" install $(V)


.PHONY: pulseview-build
pulseview-build:
	$(call prepare_build,pulseview)
	mkdir -p "$(BUILDDIR)/pulseview/contrib"
	cp "$(PATH_MAKEFILE)/FileAssociation.nsh" "$(BUILDDIR)/pulseview/contrib/"
	cd "$(BUILDDIR)/pulseview" && $(TRIPLET)-cmake -U '__pkg_config_checked_PKGDEPS' -DCMAKE_INSTALL_PREFIX:PATH="$(PREFIX)" -DCMAKE_BUILD_TYPE=$(BUILD_TYPE) -DCMAKE_C_FLAGS="$(TOOLCHAIN_FLAGS)" -DCMAKE_CXX_FLAGS="$(TOOLCHAIN_FLAGS) -fext-numeric-literals" -DASCIIDOCTOR_PDF_EXECUTABLE:FILEPATH= -DSTATIC_PKGDEPS_LIBS=ON -DDISABLE_WERROR=y -DENABLE_TESTS=y "$(SOURCES)/pulseview"
	$(MAKE) -C "$(BUILDDIR)/pulseview" $(PARALLEL) $(V)
	$(MAKE) -C "$(BUILDDIR)/pulseview" manual
	$(MAKE) -C "$(BUILDDIR)/pulseview" $(if $(filter 1,$(DEBUG)),install,install/strip) $(V)

# NSIS 安装包输出到发布目录；64 位版本传入 PE64 宏。

.PHONY: pulseview sigrok-cli pulseview-installer sigrok-cli-installer
pulseview: pulseview-installer pulseview-portable
sigrok-cli: sigrok-cli-installer sigrok-cli-portable

sigrok-cli-installer: sigrok-cli-build runtime sigrok-dumps
pulseview-installer: pulseview-build runtime sigrok-dumps
sigrok-cli-installer pulseview-installer:
	mkdir -p "$(DISTDIR)"
	sed -i -e 's|$${CROSS}/zadig|$${ZADIG_DIR}/zadig|g' \
		-e '/^[[:space:]]*File .*python34\.dll/c\\tFile "$${CROSS}/Python/runtime/*"' \
		-e '/^[[:space:]]*File .*python34\.zip/d' \
		-e '/^[[:space:]]*File .*\*\.pyd/d' \
		-e '/^[[:space:]]*Delete .*python34\.dll/c\\t!include "$${CROSS}/Python/runtime-uninstall.nsh"' \
		-e '/^[[:space:]]*Delete .*python34\.zip/d' \
		-e '/^[[:space:]]*Delete .*\*\.pyd/d' \
		"$(BUILDDIR)/$(patsubst %-installer,%,$@)/contrib/$(patsubst %-installer,%,$@)_cross.nsi"
	cd "$(BUILDDIR)/$(patsubst %-installer,%,$@)" && \
		installer=$$(sed -n 's/^OutFile "\(.*\)"$$/\1/p' "contrib/$(patsubst %-installer,%,$@)_cross.nsi") && \
		test -n "$$installer" && \
		makensis $(if $(filter x86_64,$(TARGET)),-DPE64=1) "-DZADIG_DIR=$(abspath $(DOWNLOADDIR))" "contrib/$(patsubst %-installer,%,$@)_cross.nsi" \
			"-XOutFile \"$(abspath $(DISTDIR))/$$installer\""

.PHONY: pulseview-portable sigrok-cli-portable
pulseview-portable: pulseview-build runtime
sigrok-cli-portable: sigrok-cli-build runtime
# 免安装包直接使用安装产物，不包含 SDK、静态库和编译文件。
pulseview-portable sigrok-cli-portable:
	@$(ECHO) "creating $(patsubst %-portable,%,$@) portable archive ..."
	rm -rf "$(PORTABLEDIR)/$(patsubst %-portable,%,$@)-$(FLAVOR)-$(SUFFIX)-portable"
	mkdir -p "$(PORTABLEDIR)/$(patsubst %-portable,%,$@)-$(FLAVOR)-$(SUFFIX)-portable/share" "$(DISTDIR)"
	cp "$(PREFIX)/bin/$(patsubst %-portable,%,$@).exe" "$(SOURCES)/$(patsubst %-portable,%,$@)/COPYING" "$(PORTABLEDIR)/$(patsubst %-portable,%,$@)-$(FLAVOR)-$(SUFFIX)-portable/"
	cp "$(PREFIX)/Python/runtime/"* "$(DOWNLOADDIR)/zadig.exe" "$(DOWNLOADDIR)/zadig_xp.exe" "$(PORTABLEDIR)/$(patsubst %-portable,%,$@)-$(FLAVOR)-$(SUFFIX)-portable/"
	cp -R "$(PREFIX)/share/libsigrokdecode" "$(PREFIX)/share/sigrok-firmware" "$(PORTABLEDIR)/$(patsubst %-portable,%,$@)-$(FLAVOR)-$(SUFFIX)-portable/share/"
	find "$(PORTABLEDIR)/$(patsubst %-portable,%,$@)-$(FLAVOR)-$(SUFFIX)-portable" -type d -name __pycache__ -prune -exec rm -rf -- {} +
	find "$(PORTABLEDIR)/$(patsubst %-portable,%,$@)-$(FLAVOR)-$(SUFFIX)-portable" -type f -name '*.pyc' -delete
	rm -f "$(DISTDIR)/$(patsubst %-portable,%,$@)-$(FLAVOR)-$(SUFFIX)-portable.zip"
	cd "$(PORTABLEDIR)" && zip -qr "$(abspath $(DISTDIR))/$(patsubst %-portable,%,$@)-$(FLAVOR)-$(SUFFIX)-portable.zip" "$(patsubst %-portable,%,$@)-$(FLAVOR)-$(SUFFIX)-portable"


.PHONY: clean
clean:
	@$(ECHO) "cleaning platform directory: $(PLATFORMDIR)"
	rm -rf -- "$(PLATFORMDIR)"
	@for component in $(COMPONENTS); do \
		src="$(SOURCES)/$$component"; \
		[ -e "$$src/.git" ] || continue; \
		$(ECHO) "cleaning generated files in submodule: $$component"; \
		git -C "$$src" clean -dfX >/dev/null; \
	done


# 仅清理原有默认位置中的 MXE，保留 HOME/mxe 和显式指定的外部工具链。
.PHONY: clean-mxe
clean-mxe:
	@$(ECHO) "removing MXE from build directory: $(MXE_BUILDDIR)"
	rm -rf "$(MXE_BUILDDIR)"
