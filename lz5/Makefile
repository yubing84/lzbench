# ################################################################
# LZ5 library - Makefile
# Copyright (C) Yann Collet 2011-2015
# All rights reserved.
# 
# BSD license
# Redistribution and use in source and binary forms, with or without modification,
# are permitted provided that the following conditions are met:
# 
# * Redistributions of source code must retain the above copyright notice, this
#   list of conditions and the following disclaimer.
# 
# * Redistributions in binary form must reproduce the above copyright notice, this
#   list of conditions and the following disclaimer in the documentation and/or
#   other materials provided with the distribution.
# 
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
# ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
# WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
# DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR
# ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
# (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
# LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON
# ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
# (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
# SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
# 
# You can contact the author at :
#  - LZ5 source repository : https://github.com/inikep/lz5
#  - LZ5 forum froup : https://groups.google.com/forum/#!forum/lz5c
# ################################################################

# Version numbers
VERSION?= 1.4
LIBVER_MAJOR:=`sed -n '/define LZ5_VERSION_MAJOR/s/.*[[:blank:]]\([0-9][0-9]*\).*/\1/p' < lz5.h`
LIBVER_MINOR:=`sed -n '/define LZ5_VERSION_MINOR/s/.*[[:blank:]]\([0-9][0-9]*\).*/\1/p' < lz5.h`
LIBVER_PATCH:=`sed -n '/define LZ5_VERSION_RELEASE/s/.*[[:blank:]]\([0-9][0-9]*\).*/\1/p' < lz5.h`
LIBVER := $(LIBVER_MAJOR).$(LIBVER_MINOR).$(LIBVER_PATCH)

DESTDIR?=
PREFIX ?= /usr/local
CPPFLAGS= -DXXH_NAMESPACE=LZ5_
CFLAGS ?= -O3
CFLAGS += -std=c99 -Wall -Wextra -Wundef -Wshadow -Wcast-align -Wcast-qual -Wstrict-prototypes -pedantic
FLAGS   = $(CPPFLAGS) $(CFLAGS) $(LDFLAGS)

LIBDIR?= $(PREFIX)/lib
INCLUDEDIR=$(PREFIX)/include


# OS X linker doesn't support -soname, and use different extension
# see : https://developer.apple.com/library/mac/documentation/DeveloperTools/Conceptual/DynamicLibraries/100-Articles/DynamicLibraryDesignGuidelines.html
ifeq ($(shell uname), Darwin)
	SHARED_EXT = dylib
	SHARED_EXT_MAJOR = $(LIBVER_MAJOR).$(SHARED_EXT)
	SHARED_EXT_VER = $(LIBVER).$(SHARED_EXT)
	SONAME_FLAGS = -install_name $(PREFIX)/lib/liblz5.$(SHARED_EXT_MAJOR) -compatibility_version $(LIBVER_MAJOR) -current_version $(LIBVER)
else
	SONAME_FLAGS = -Wl,-soname=liblz5.$(SHARED_EXT).$(LIBVER_MAJOR)
	SHARED_EXT = so
	SHARED_EXT_MAJOR = $(SHARED_EXT).$(LIBVER_MAJOR)
	SHARED_EXT_VER = $(SHARED_EXT).$(LIBVER)
endif

default: liblz5

all: liblz5

liblz5: lz5.c lz5hc.c lz5frame.c xxhash.c   # need to compile once with -fPIC, and once without -fPIC
	@echo compiling static library
	@$(AR) rcs $@.a $^
	@echo compiling dynamic library $(LIBVER)
	@$(CC) $(FLAGS) -shared $^ -fPIC $(SONAME_FLAGS) -o $@.$(SHARED_EXT_VER)
	@echo creating versioned links
	@ln -sf $@.$(SHARED_EXT_VER) $@.$(SHARED_EXT_MAJOR)
	@ln -sf $@.$(SHARED_EXT_VER) $@.$(SHARED_EXT)

clean:
	@rm -f core *.o *.a *.$(SHARED_EXT) *.$(SHARED_EXT).* liblz5.pc
	@echo Cleaning library completed


#------------------------------------------------------------------------
#make install is validated only for Linux, OSX, kFreeBSD and Hurd targets
ifneq (,$(filter $(shell uname),Linux Darwin GNU/kFreeBSD GNU))

liblz5.pc: liblz5.pc.in Makefile
	@echo creating pkgconfig
	@sed -e 's|@PREFIX@|$(PREFIX)|' \
            -e 's|@LIBDIR@|$(LIBDIR)|' \
            -e 's|@INCLUDEDIR@|$(INCLUDEDIR)|' \
            -e 's|@VERSION@|$(VERSION)|' \
             $< >$@

install: liblz5 liblz5.pc
	@install -d -m 755 $(DESTDIR)$(LIBDIR)/pkgconfig/ $(DESTDIR)$(INCLUDEDIR)/
	@install -m 755 liblz5.$(SHARED_EXT_VER) $(DESTDIR)$(LIBDIR)
	@ln -sf liblz5.$(SHARED_EXT_VER) $(DESTDIR)$(LIBDIR)/liblz5.$(SHARED_EXT_MAJOR)
	@ln -sf liblz5.$(SHARED_EXT_VER) $(DESTDIR)$(LIBDIR)/liblz5.$(SHARED_EXT)
	@install -m 644 liblz5.pc $(DESTDIR)$(LIBDIR)/pkgconfig/
	@install -m 644 liblz5.a $(DESTDIR)$(LIBDIR)/liblz5.a
	@install -m 644 lz5.h $(DESTDIR)$(INCLUDEDIR)/lz5.h
	@install -m 644 lz5hc.h $(DESTDIR)$(INCLUDEDIR)/lz5hc.h
	@install -m 644 lz5frame.h $(DESTDIR)$(INCLUDEDIR)/lz5frame.h
	@echo lz5 static and shared libraries installed

uninstall:
	@rm -f $(DESTDIR)$(LIBDIR)/liblz5.$(SHARED_EXT)
	@rm -f $(DESTDIR)$(LIBDIR)/liblz5.$(SHARED_EXT_MAJOR)
	@rm -f $(DESTDIR)$(LIBDIR)/pkgconfig/liblz5.pc
	@[ -x $(DESTDIR)$(LIBDIR)/liblz5.$(SHARED_EXT_VER) ] && rm -f $(DESTDIR)$(LIBDIR)/liblz5.$(SHARED_EXT_VER)
	@[ -f $(DESTDIR)$(LIBDIR)/liblz5.a ] && rm -f $(DESTDIR)$(LIBDIR)/liblz5.a
	@[ -f $(DESTDIR)$(INCLUDEDIR)/lz5.h ] && rm -f $(DESTDIR)$(INCLUDEDIR)/lz5.h
	@[ -f $(DESTDIR)$(INCLUDEDIR)/lz5hc.h ] && rm -f $(DESTDIR)$(INCLUDEDIR)/lz5hc.h
	@[ -f $(DESTDIR)$(INCLUDEDIR)/lz5frame.h ] && rm -f $(DESTDIR)$(INCLUDEDIR)/lz5frame.h
	@echo lz5 libraries successfully uninstalled

endif
