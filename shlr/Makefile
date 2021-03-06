include ../config-user.mk
include ../global.mk
# for EXT_EXE
include ../librz/config.mk
include ../mk/platform.mk
_INCLUDE_MK_GCC_=
include ../mk/${COMPILER}.mk
include ../mk/darwin.mk

# Build against last capstone release or next
ifeq ($(shell test -d ../.git && echo 1),1)
CS_RELEASE=0
else
CS_RELEASE=1
endif

CS_NEXT=${CSNEXT}

CAPSTONE_CFLAGS?=-g
# CAPSTONE_MAKEFLAGS?=CAPSTONE_ARCHS="arm aarch64"

GIT?=git
ifeq ($(shell type wget 2> /dev/null && echo 1),1)
WGET?=wget -c --no-check-certificate -O
else
WGET?=curl -o
endif

# NOTE: when you update TS_TIP or TS_BRA, also update them in shlr/meson.build
TS_URL=https://github.com/tree-sitter/tree-sitter.git
TS_BRA=master
TS_TIP=5080de496a87fc2b9609af80e3d893a04e2b868c

ifeq ($(CS_RELEASE),1)
CS_VER=4.0.2
CS_TAR=https://codeload.github.com/aquynh/capstone/tar.gz/$(CS_VER)
#CS_TAR=http://capstone-engine.org/download/$(CS_VER)/capstone-$(CS_VER).tgz
CS_PATCHES=0
else
CS_TAR=
CS_URL_BASE=github.com/aquynh/capstone
CS_URL=$(GIT_PREFIX)$(CS_URL_BASE).git
CS_ARCHIVE=https://$(CS_URL_BASE)/archive
CS_UPD=20190515
# NOTE: when you update CS_TIP or CS_BRA, also update them in shlr/meson.build
ifeq ($(CS_NEXT),1)
CS_TIP=d956ceddabcbe544ee8f074ba75c86c6381b34e6
CS_BRA=next
else
CS_TIP=2edae851d9fee511a57d4da32d5acecedd95d7ed
CS_BRA=v4
endif
ifeq ($(CS_COMMIT_ARCHIVE),1)
CS_ARCHIVE_URL=$(CS_ARCHIVE)/$(CS_TIP).zip
else
CS_ARCHIVE_URL=
endif
# REVERT THIS COMMIT BECAUSE ITS WRONG
CS_REV=
CS_PATCHES=1
endif

.PHONY: capstone-sync capstone-build all clean mrproper libgdbr libwinkd bochs tree-sitter-sync sdbs

ifeq ($(shell gcc -v > /dev/null 2>&1 && echo works),works)
HOST_CC?=gcc
endif
SHLR?=$(shell pwd)
AR?=ar
RANLIB?=ranlib
MODS=sdb zip java tcc mpc yxml
MODS+=gdb qnx ar
# lz4
ifneq ($(CC),cccl)
endif
ifeq (1,$(USE_PTRACE_WRAP))
MODS+=ptrace-wrap
endif
SDB_URL=git://github.com/rizinorg/sdb
#SDB_URL=/Users/pancake/prg/sdb
SPP_URL=git://github.com/rizin/spp
PTRACE_WRAP_URL=git://github.com/thestr4ng3r/ptrace-wrap
PWD=$(shell pwd)
SDB_CONFIG=${PWD}/sdb-config.mk

CFLAGS_SHARED=${PIC_CFLAGS}

ifeq (${EXT_SO},)
all:
	echo EXT_SO not defined
	exit 1
endif

preall: libwinkd capstone-build tree-sitter-build rizin-shell-parser-build bochs
	@for MOD in ${MODS} ; do \
		echo $(MAKE) -C $$MOD ; \
		$(MAKE) -C $$MOD HAVE_VALA= ROOT="${PWD}/../" CC="${CC}" ; \
	done

SDBFILES=sdb/src/.sdb${EXT_EXE} sdb/sdb${EXT_EXE} sdb/src/.sdb sdb/sdb
PREMODS=capstone gdb winkd sdb bochs ar

clean mrproper:
	rm -f ${SDBFILES}
	@for MOD in ${PREMODS} ${MODS} ; do $(MAKE) -C $$MOD clean ; done

ifeq ($(BUILD_OS),windows)
BUILD_EXT_EXE=.exe
else
BUILD_EXT_EXE=
endif

PIC=
ifneq ($(CC),cccl)
PIC=-fPIC
endif

SDB_HOST=sdb/sdb$(BUILD_EXT_EXE)
SDB_LIBA=sdb/src/libsdb.a

$(SDB_HOST):
	$(MAKE) sdb-host HOST_CC=$(HOST_CC) CC=$(HOST_CC)

$(SDB_LIBA):
	$(MAKE) sdb-target

sdbs: $(SDB_HOST)

sdb-host:
	@echo
	@echo ">>>>>>>>"
	@echo "HOST SDB"
	@echo ">>>>>>>>"
	@echo
	$(MAKE) -C sdb clean ; rm -f sdb/src/*.o sdb/src/sdb_version.h
	$(MAKE) -C sdb/src "CC=${HOST_CC}" LDFLAGS='${HOST_LDFLAGS}' CPPFLAGS='' CFLAGS='${HOST_CFLAGS} ${PIC}' bin
	cp -f sdb/src/sdb${BUILD_EXT_EXE} sdb/src/.sdb${BUILD_EXT_EXE}
	cp -f sdb/src/sdb${BUILD_EXT_EXE} sdb/sdb$(BUILD_EXT_EXE)
	-file sdb/sdb$(BUILD_EXT_EXE)

sdb-target:
	@echo
	@echo ">>>>>>>>>>"
	@echo "TARGET SDB"
	@echo ">>>>>>>>>>"
	@echo
	$(MAKE) -C sdb clean ; rm -f sdb/src/*.o sdb/src/sdb_version.h
	$(MAKE) -C sdb/src sdb_version.h
	$(MAKE) -C sdb/src ARCH=xxx RANLIB="${RANLIB}" CFLAGS_SHARED="${CFLAGS_SHARED}" \
		CC="${CC}" AR="${AR}" ARCH=undefined CFLAGS='${CFLAGS}' LDFLAGS='${LDFLAGS}' libsdb.a
	${RANLIB} sdb/src/libsdb.a

.PHONY: sdb-target
SDB_F=README.md config.mk src Makefile meson.build
SDB_SYNCFILES=$(addprefix sdb.vc/,${SDB_F})
I=../librz/include

libgdbr:
	$(MAKE) -C gdb all

libwinkd:
	$(MAKE) -C winkd all

bochs:
	$(MAKE) -C bochs all

sdb-clean clean-sdb:
	$(MAKE) -C sdb clean
	rm -f sdb/sdb.exe
	rm -f sdb/src/libsdb.${EXT_AR}

.PHONY: ptrace-wrap ptrace-wrap-sync sync-ptrace-wrap ptrace-wrap-clean clean-ptrace-wrap
PTRACE_WRAP_F=include src README.md COPYING
PTRACE_WRAP_SYNCFILES=$(addprefix ptrace-wrap.vc/,${PTRACE_WRAP_F})

ptrace-wrap-sync sync-ptrace-wrap:
	rm -rf ptrace-wrap ptrace-wrap.vc
	git clone --depth 1 ${PTRACE_WRAP_URL} ptrace-wrap.vc
	mkdir -p ptrace-wrap
	cp -rf ${PTRACE_WRAP_SYNCFILES} ptrace-wrap
	sed -e "s, library(, static_library(," -e "s,project(,# project(," -e "s,subdir(,# subdir(," ptrace-wrap.vc/meson.build > ptrace-wrap/meson.build
	printf "include ../../librz/config.mk\ninclude ../../mk/platform.mk\ninclude ../../mk/${COMPILER}.mk\n\n" | cat - ptrace-wrap.vc/Makefile > ptrace-wrap/Makefile
	rm -rf ptrace-wrap.vc
	printf "CFLAGS+=-I\$$(STOP)/ptrace-wrap/include\nLINK+=\$$(STOP)/ptrace-wrap/libptrace_wrap.\$$(EXT_AR)\nPTRACEWRAP_OBJS=\$$(STOP)/ptrace-wrap/src/ptrace_wrap.o\n" > ptrace-wrap/deps.mk

ptrace-wrap:
	$(MAKE) -C ptrace-wrap all

ptrace-wrap-clean clean-ptrace-wrap:
	$(MAKE) -C ptrace-wrap clean

$(TCCLIB) librz_tcc/librz_tcc.${EXT_AR}:
	cd librz_tcc ; $(MAKE)

tcc-clean tccclean:
	cd librz_tcc ; $(MAKE) clean

CFILES=i386-asm.c i386-gen.c libtcc.c tccasm.c tccelf.c tccgen.c tccpp.c
HFILES=tcc.h i386-asm.h tcclib.h tcctok.h stab.h
HFILES+=elf.h config.h i386-tok.h

# TODO: use mob branch?

tcc-sync sync-tcc:
	rm -rf _
	$(GIT) clone --depth 1 git://repo.or.cz/tinycc.git _
	cd _ ; ./configure --prefix=${PREFIX}
	mkdir -p tcc
	for FILE in ${CFILES} ${HFILES} ; do cp -f _/$$FILE tcc ; done
	cp -f _/VERSION _/COPYING _/stab.def tcc
	$(GIT) add tcc
	rm -rf _

install: librz_shlr.${EXT_AR}
	mkdir -p ${DESTDIR}/${LIBDIR}
	cp -f librz_shlr.${EXT_AR} ${DESTDIR}/${LIBDIR}

install-symlink symstall:
	mkdir -p "${DESTDIR}${LIBDIR}"
	#cd tcc ; for FILE in librz_tcc* ; do \
	#	ln -fs `pwd`/$$FILE ${DESTDIR}${LIBDIR}/$$FILE ; done

uninstall deinstall:
	rm -f ${DESTDIR}${LIBDIR}/librz_shlr.${EXT_AR}

ifeq ($(USE_CAPSTONE),1)
# Do nothing here
capstone:

capstone-sync:

capstone-build:

else

ifeq ($(CS_RELEASE),1)
capstone-sync: capstone

capstone-clean:
	cd capstone ; $(MAKE) clean

capstone: capstone-$(CS_VER).tar.gz
	tar xzvf capstone-$(CS_VER).tar.gz
	rm -rf capstone
	mv capstone-$(CS_VER) capstone
ifeq ($(CS_PATCHES),1)
	if [ -d capstone/.git ]; then \
		cd capstone ; for PATCH in ../capstone-patches/* ; do patch -p1 < $$PATCH ; done ; \
	fi
endif
	mkdir -p capstone/include/capstone
	cp -rf capstone/include/*.h capstone/include/capstone

capstone-$(CS_VER).tar.gz:
	$(WGET) capstone-$(CS_VER).tar.gz $(CS_TAR)
else

capstone: capstone-sync

capstone-sync:
ifeq ($(WITHOUT_PULL),1)
	@echo "Nothing to sync because of --without-pull"
else
	"$(SHELL)" capstone.sh "${CS_URL}" "${CS_BRA}" "${CS_TIP}" "${CS_REV}" "${CS_ARCHIVE_URL}"
endif

.PHONY: capstone capstone-sync
endif

ifeq (${EXT_AR},)
all:
	echo EXT_AR is not defined
	exit 1
endif

capstone-build: capstone
	cd capstone && CFLAGS="-Dmips=mips ${CAPSTONE_CFLAGS} $(CFLAGS)" LDFLAGS="$(LDFLAGS)" \
		$(MAKE) $(CAPSTONE_MAKEFLAGS) CC="$(CC)" AR_EXT=a IS_CYGWIN=0 CAPSTONE_STATIC=yes CAPSTONE_SHARED=no \
		RANLIB="$(RANLIB)" LIBARCHS="" AR="$(AR)" IS_MINGW=0 libcapstone.a
ifneq (${EXT_AR},a)
	-cp -f capstone/libcapstone.a capstone/libcapstone.${EXT_AR}
endif
endif
../../rizin-webui:
	cd ../.. ; git clone --depth 1 https://github.com/rizinorg/rizin-webui

../../rizin-webui/dist/m: ../../rizin-webui
ifeq ($(WITHOUT_PULL),1)
	cd ../../rizin-webui/www/m && npm i ; $(MAKE) release
else
	cd ../../rizin-webui/www/m && git pull ; npm i ; $(MAKE) release
endif

tree-sitter-build: tree-sitter/libtree-sitter.$(EXT_AR)

tree-sitter/libtree-sitter.$(EXT_AR): tree-sitter/lib/src/lib.o
	$(AR) rvs $@ $<
	$(RANLIB) $@

tree-sitter/lib/src/lib.o:
	$(CC) $(NEWSHELL_CFLAGS) -c tree-sitter/lib/src/lib.c -o $@ -Itree-sitter/lib/include -Itree-sitter/lib/src $(CFLAGS)

tree-sitter-sync:
	rm -rf tree-sitter tree-sitter.vc
	"$(SHELL)" clone_3rd_repo.sh tree-sitter.vc "${TS_URL}" "${TS_BRA}" "${TS_TIP}"
	mkdir -p tree-sitter/lib
	cp -rf ./tree-sitter.vc/lib/src ./tree-sitter/lib
	cp -rf ./tree-sitter.vc/lib/include ./tree-sitter/lib
	rm -rf tree-sitter.vc

rizin-shell-parser-build: rizin-shell-parser/libshell-parser.$(EXT_AR)

rizin-shell-parser/libshell-parser.$(EXT_AR): rizin-shell-parser/src/parser.o rizin-shell-parser/src/scanner.o
	$(AR) rvs $@ rizin-shell-parser/src/parser.o rizin-shell-parser/src/scanner.o
	$(RANLIB) $@

rizin-shell-parser/src/parser.o: rizin-shell-parser/src/parser.c
	$(CC) $(NEWSHELL_CFLAGS) -c rizin-shell-parser/src/parser.c -o $@ -Irizin-shell-parser/src/tree_sitter -Itree-sitter/lib/include $(CFLAGS)

rizin-shell-parser/src/scanner.o: rizin-shell-parser/src/scanner.c
	$(CC) -c rizin-shell-parser/src/scanner.c -o $@ -Irizin-shell-parser/src/tree_sitter -Itree-sitter/lib/include $(CFLAGS)

SHLRS+=tree-sitter/libtree-sitter.a
SHLRS+=rizin-shell-parser/libshell-parser.a

spp-sync sync-spp:
	-git rm -r spp
	rm -rf spp
	git clone --depth 1 ${SPP_URL} spp
	rm -rf spp/.git*
	rm -rf spp/t
	git add spp

spp: spp-sync
	CFLAGS="-DUSE_R2=1 -I../../librz/include -DHAVE_FORK=${HAVE_FORK} -fPIC" $(MAKE) -C spp r2lib

SHLRS+=ar/librz_ar.a
SHLRS+=bochs/lib/libbochs.a
SHLRS+=capstone/libcapstone.a
SHLRS+=gdb/lib/libgdbr.a
SHLRS+=java/librz_java.a
SHLRS+=lz4/liblz4.a
SHLRS+=qnx/lib/libqnxr.a
SHLRS+=yxml/libyxml.a
SHLRS+=winkd/librz_winkd.a
SHLRS+=zip/librz.a

librz_shlr.${EXT_AR}:
	rm -rf .librz
	mkdir .librz
ifeq ($(CC),emcc)
	for LIB in ${SHLRS} ; do \
		if [ -f "$${LIB}" ]; then emcc -emit-llvm -nostdlib -Wl,--whole-archive -o .librz/$$(basename $${LIB}.a).o $${LIB} ; fi ; \
	done
	${AR} rs $@ .librz/*.o
	$(RANLIB) $@
else
	for LIB in ${SHLRS} ; do \
		if [ -f "$${LIB}" ]; then ${PARTIALLD} -o .librz/$$(basename $${LIB}.a).o $${LIB} ; fi ; \
	done
	${AR} rs $@ .librz/*.o
	$(RANLIB) $@
endif

.PHONY: spp

.PHONY: mpc mpc-clean clean-mpc
mpc:
	$(MAKE) -C mpc all

mpc-clean clean-mpc:
	$(MAKE) -C mpc clean
