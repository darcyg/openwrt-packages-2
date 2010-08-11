# Makefile for OpenWRT for cross-compiling GForth
#
# Copyright (C) 1995,1996,1997,1998,2000,2003,2006,2007,2008,2009 Free
# Software Foundation, Inc.
#
# Copyright (C) 2010 David Kühling.  License: GPLv3+

## Note for tuning: In theory you could strip down the resulting packages size
## a lot, if you removed gforth and gforth-fast executables and only included
## the (somewhat slower) gforth-itc and/or gforth-ditc interpreter binary that
## rely on classic indirect threaded code without the (code-bloating) engine
## optimizations.
##
## Also we could create multiple packages here 'gforth-minimal' and 'gforth'?

include $(TOPDIR)/rules.mk

PKG_NAME:=gforth
PKG_SNAPSHOT_DATE=20100807
PKG_VERSION=0.7.0-$(PKG_SNAPSHOT_DATE)
PKG_RELEASE:=4


PKG_BUILD_DEPENDS:= gforth/host libltdl/host
PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)-$(PKG_VERSION)
PKG_FIXUP:=libtool

PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:=\
	http://user.cs.tu-berlin.de/~dvdkhlng/ \
	http://mosquito.dyndns.tv/~spock/
#PKG_SOURCE_URL:= file://~/forth/gforth/
PKG_MD5SUM:=fb1d0dee4836dab57fd3eadbd5fc1922

# Alternate download #1 via CVS: this doesn't work, as CVS is missing the
# kernel.fi images needed for boot-strapping :(
# PKG_SOURCE_PROTO:=cvs
# PKG_SOURCE_VERSION="-D$(PKG_SNAPSHOT_DATE) 23:59"
# PKG_SOURCE_URL=:pserver:anonymous@c1.complang.tuwien.ac.at:/nfs/unsafe/cvs-repository/src-master
# PKG_SOURCE_SUBDIR:=gforth
# PKG_SOURCE:=gforth-$(PKG_SNAPSHOT_DATE).tar.gz

include $(INCLUDE_DIR)/host-build.mk
include $(INCLUDE_DIR)/package.mk

define Package/gforth
  SECTION:=lang
  CATEGORY:=Languages
  TITLE:=GForth
  DEPENDS:= +libltdl
  URL:=http://www.gnu.org/software/gforth/
endef

define Package/gforth/description
  Gforth is a fast and portable implementation of the ANS Forth
  language.
endef

HOST_CONFIGURE_VARS += LTDL_LIBRARY_PATH=$(STAGING_DIR_HOST)/lib

## Engine's dispatch would be crippled when compiling with -Os, also dynamic
## superinstructions don't seem to work in that case
EXTRA_CFLAGS += -O2

## The host-GForth uses -ltdl to link against the staging dir version of
## libltdl.  However, when the host-GForth runs, it won't find that library,
## as no library path is encoded into the ltdl dependency (why?).  So here we
## override LD_LIBRARY_PATH for all the build steps that might run the
## host-GForth.

define Host/Configure
	export LD_LIBRARY_PATH=$(STAGING_DIR_HOST)/lib; \
	$(call Host/Configure/Default)
endef

define Host/Compile
	export LD_LIBRARY_PATH=$(STAGING_DIR_HOST)/lib; \
	$(call Host/Compile/Default)
endef

define Host/Install
	export LD_LIBRARY_PATH=$(STAGING_DIR_HOST)/lib; \
	$(call Host/Install/Default)
endef

## Configuration of the target gforth
FORTHSIZES=--dictionary-size=1M \
	--data-stack-size=16k \
	--fp-stack-size=16k \
	--return-stack-size=16k \
	--locals-stack-size=16k

CONFIGURE_VARS += FORTHSIZES="$(FORTHSIZES)"

CROSS_PREFORTH = $(PKG_BUILD_DIR)/preforth

## Here we call configure, then patch the cross-GForth source tree to replace
## the 'preforth' script with a script that calls our host-compiled GForth.
## We also extract the name of the GForth kernel image used for the target
## architecture, and the source files used by GForth for implementing the
## assembler/disassembler for the target architecture.
define Build/Configure
	$(call Build/Configure/Default,)
	echo "@kernel_fi@" > $(PKG_BUILD_DIR)/kernel_fi.in
	cd $(PKG_BUILD_DIR) && ./config.status --file kernel_fi
	echo "@asm_fs@ @disasm_fs@" > $(PKG_BUILD_DIR)/asm_fs.in
	cd $(PKG_BUILD_DIR) && ./config.status --file asm_fs
	echo "#!/bin/sh" > $(CROSS_PREFORTH)
	echo "export LD_LIBRARY_PATH=$(STAGING_DIR_HOST)/lib;" >> $(CROSS_PREFORTH)
	echo '$(STAGING_DIR_HOST)/bin/gforth -i $(STAGING_DIR_HOST)/lib/gforth/$(PKG_VERSION)/gforth.fi "$$$$@"' >> $(CROSS_PREFORTH)
	chmod a+x $(PKG_BUILD_DIR)/preforth
endef

## Compilation is pretty manual to only build the parts we need.  By default
## the GForth Makefile attempts to auto-tune by recursively calling itself for
## compilation, then running the GForth binary through unit-tests.  This won't
## work with a cross-compile environment.
##
## Todo: we currently always build the -ll-reg version of the engine.  On
## 64-bit architectures this might not work?  Damn it, why is the check for
## 'long long' in the Gforth Makefile, not the configure script?
## Todo: develop a clean upstream patch to configure/Makefile
define Build/Compile
	$(MAKE) -C $(PKG_BUILD_DIR) kernel/version.fs gforth-ditc \
		engine/prim-fast.i engine/prim_lab-fast.i engine/prim_names-fast.i \
		engine/prim_superend-fast.i engine/profile-fast.i \
		engine/prim_num-fast.i engine/prim_grp-fast.i \
		engine/costs-fast.i engine/super2-fast.i
	$(MAKE) -C $(PKG_BUILD_DIR)/engine gforth-fast-ll-reg gforth-ll-reg \
		OPT=-ll-reg OPTDEFINES="-DFORCE_LL -DFORCE_REG" OPTOBJECTS=
	cd $(PKG_BUILD_DIR) && \
            cp engine/gforth-ll-reg ./gforth && \
	    cp engine/gforth-fast-ll-reg ./gforth-fast
endef

##
## define lists of GForth's sources to package for loading in the target system
##

GFORTH_FI_SRC = \
	assert.fs \
	backtrac.fs \
	blocked.fb \
	blocks.fs \
	bufio.fs \
	code.fs \
	debug.fs \
	debugs.fs \
	dis-gdb.fs \
	ekey.fs \
	envos.fs \
	savesys.fs \
	environ.fs \
	errors.fs \
	exboot.fs \
	except.fs \
	extend.fs \
	float.fs \
	glocals.fs \
	hash.fs \
	history.fs \
	intcomp.fs \
	mkdir.fs \
	libcc.fs \
	locals.fs \
	look.fs \
	mkdir.fs \
	prelude.fs \
	quotes.fs \
	search.fs \
	see.fs \
	see-ext.fs \
	simp-see.fs \
	source.fs \
	startup.fs \
	struct.fs \
	struct0x.fs \
	stuff.fs \
	tasker.fs \
	termsize.fs \
	utf-8.fs \
	vt100.fs \
	vt100key.fs \
	wordinfo.fs \
	arch/386/asm.fs arch/386/disasm.fs \
	arch/amd64/asm.fs arch/amd64/disasm.fs \
	arch/alpha/asm.fs arch/alpha/disasm.fs arch/alpha/testasm.fs\
	arch/arm/asm.fs arch/arm/disasm.fs \
	arch/arm/testdisasm.fs arch/arm/testdisasm.out arch/arm/Makefile \
	arch/mips/asm.fs arch/mips/disasm.fs arch/mips/insts.fs \
	arch/mips/testasm.fs arch/mips/testdisasm.fs \
	arch/power/asm.fs arch/power/disasm.fs arch/power/inst.fs

LIBCC_SRC = cstr.fs unix/socket.fs
LIBCC_DIST_SRC = libffi.fs fflib.fs $(LIBCC_SRC)

# todo: strip down (really?)
GFORTH_SRC = $(GFORTH_FI_SRC) $(LIBCC_DIST_SRC) \
	ans-report.fs ansi.fs answords.fs \
	colorize.fs comp-i.fs complex.fs \
	depth-changes.fs dosekey.fs doskey.fs ds2texi.fs \
	envos.dos envos.os2 etags.fs fft.fs filedump.fs fi2c.fs \
	fsl-util.4th glosgen.fs gray.fs httpd.fs install-tags.fs \
	make-app.fs doc/makedoc.fs locate.fs more.fs onebench.fs \
	other.fs prims2x.fs prims2x0.6.2.fs proxy.fs random.fs \
	regexp.fs sokoban.fs string.fs table.fs tags.fs \
	tt.fs \
	unbuffer.fs wordsets.fs xwords.fs \
	test/tester.fs test/ttester.fs \
	test/coretest.fs test/postpone.fs test/dbltest.fs \
	test/string.fs test/float.fs test/search.fs test/gforth.fs \
	test/other.fs test/signals.fs test/checkans.fs \
	test/primtest.fs test/coreext.fs test/deferred.fs \
	test/coremore.fs test/gforth-nofast.fs test/libcc.fs \
	test/macros.fs \
	compat/strcomp.fs \
	bubble.fs siev.fs matrix.fs fib.fs \
	oof.fs oofsampl.fs objects.fs objexamp.fs mini-oof.fs moof-exm.fs \
	moofglos.fs fixpath.fs \
	add.fs lib.fs oldlib.fs sieve.fs \
	endtry-iferror.fs recover-endtry.fs

GFORTH_BIN = gforth gforth-fast gforthmi

GFORTH_SHARE_DIR = /usr/share/gforth/$(PKG_VERSION)
GFORTH_LIB_DIR = /usr/lib/gforth/$(PKG_VERSION)
GFORTH_BIN_DIR = /usr/bin

## Select files for package.  Note how we rename the GForth kernel image to
## 'kernel.fi' here, so that 'postinst' can refer to it without depending on
## architecture-specific naming.
define Package/gforth/install
	$(INSTALL_DIR) $(1)/$(GFORTH_BIN_DIR)
	$(INSTALL_DIR) $(1)/$(GFORTH_SHARE_DIR)
	$(INSTALL_DIR) $(1)/$(GFORTH_LIB_DIR)
	$(INSTALL_DIR) $(1)/$(GFORTH_SHARE_DIR)/../site-forth
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/$$$$(cat $(PKG_BUILD_DIR)/kernel_fi) $(1)/$(GFORTH_SHARE_DIR)/kernel.fi
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/gforth-ditc $(1)/$(GFORTH_LIB_DIR)/
	$(INSTALL_DATA) $(PKG_BUILD_DIR)/siteinit.fs $(1)/$(GFORTH_SHARE_DIR)/../site-forth/
	$(call pkg_install_bin,$(GFORTH_BIN),$(PKG_BUILD_DIR),$(1)/$(GFORTH_BIN_DIR))
	$(call pkg_install_files,$(GFORTH_SRC),$(PKG_BUILD_DIR),$(1)/$(GFORTH_SHARE_DIR))
endef

FORTHKFLAGS= --die-on-signal -i kernel.fi

## make sure this is never evaluated before the configure step ran, else
## $(shell) below is going to fail!  We use the $(shell) to use the correct
## Gforth assembler implementation that was chosen by gforth's configure
## script, when building the forth image.
STARTUP	= exboot.fs startup.fs $(shell cat $(PKG_BUILD_DIR)/asm_fs)

## Directly after installation load the source once and generate a
## corresponding interpreter image.  GForth needs that for quick startup.
define Package/gforth/postinst
#! /bin/sh
echo "Creating GForth interpreter image..."
export GFORTH="$(GFORTH_BIN_DIR)/gforth $(FORTHSIZES) $(FORTHKFLAGS) $(STARTUP)"
gforthmi $(GFORTH_LIB_DIR)/gforth.fi $(FORTHSIZES) $(FORTHKFLAGS) $(STARTUP)
endef

define Package/gforth/prerm
#! /bin/sh
rm -f $(GFORTH_LIB_DIR)/gforth.fi
endef

$(eval $(call HostBuild))
$(eval $(call BuildPackage,gforth))

# The following comments configure the Emacs editor.  Just ignore them.
# Local Variables:
# compile-command: "make -C ~/h/src/qi/openwrt-xburst package/gforth/compile -j2 V=99"
# End: