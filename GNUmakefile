lsb_dist     := $(shell if [ -f /etc/redhat-release ] ; then \
                  cat /etc/redhat-release | sed 's/ .*//' ; \
                  elif [ -f /etc/os-release ] ; then \
                  grep '^NAME=' /etc/os-release | sed 's/.*=\"//' | sed 's/ .*//' ; \
                  elif [ -x /usr/bin/lsb_release ] ; then \
                  lsb_release -is ; else echo Linux ; fi)
lsb_dist_ver := $(shell if [ -f /etc/redhat-release ] ; then \
                  cat /etc/redhat-release | sed s/.*release\ // | sed s/\ .*// ; \
                  elif [ -f /etc/os-release ] ; then \
                  grep '^VERSION=' /etc/os-release | sed 's/.*=\"//' | sed 's/ .*//' ; \
                  elif [ -x /usr/bin/lsb_release ] ; then \
                  lsb_release -rs | sed 's/[.].*//' ; else uname -r | sed 's/[-].*//' ; fi)
#lsb_dist     := $(shell if [ -x /usr/bin/lsb_release ] ; then lsb_release -is ; else uname -s ; fi)
#lsb_dist_ver := $(shell if [ -x /usr/bin/lsb_release ] ; then lsb_release -rs | sed 's/[.].*//' ; else uname -r | sed 's/[-].*//' ; fi)
uname_m      := $(shell uname -m)

short_dist_lc := $(patsubst CentOS,rh,$(patsubst RedHatEnterprise,rh,\
                   $(patsubst RedHat,rh,\
                     $(patsubst Fedora,fc,$(patsubst Ubuntu,ub,\
                       $(patsubst Debian,deb,$(patsubst SUSE,ss,$(lsb_dist))))))))
short_dist    := $(shell echo $(short_dist_lc) | tr a-z A-Z)
pwd           := $(shell pwd)
rpm_os        := $(short_dist_lc)$(lsb_dist_ver).$(uname_m)

# this is where the targets are compiled
build_dir ?= $(short_dist)$(lsb_dist_ver)_$(uname_m)$(port_extra)
bind      := $(build_dir)/bin
libd      := $(build_dir)/lib64
objd      := $(build_dir)/obj
dependd   := $(build_dir)/dep

default_cflags := -ggdb -O3
# use 'make port_extra=-g' for debug build
ifeq (-g,$(findstring -g,$(port_extra)))
  default_cflags := -ggdb
endif
ifeq (-a,$(findstring -a,$(port_extra)))
  default_cflags := -fsanitize=address -ggdb -O3
endif

CC          ?= gcc -std=c11
cc          := $(CC)
arch_cflags := -fno-omit-frame-pointer
gcc_wflags  := -Wall -Wextra -Werror
fpicflags   := -fPIC
soflag      := -shared
rpath       := -Wl,-rpath,$(pwd)/$(libd)

# rpmbuild uses RPM_OPT_FLAGS
ifeq ($(RPM_OPT_FLAGS),)
CFLAGS ?= $(default_cflags)
else
CFLAGS ?= $(RPM_OPT_FLAGS)
endif
cflags := $(gcc_wflags) $(CFLAGS) $(arch_cflags)

INCLUDES ?=
DEFINES  ?=
includes := -Iinclude $(INCLUDES)
defines  :=

.PHONY: everything
everything: all

# copr/fedora build (with version env vars)
# copr uses this to generate a source rpm with the srpm target
-include .copr/Makefile

# debian build (debuild)
# target for building installable deb: dist_dpkg
-include deb/Makefile

liblzf_files := lzf_c lzf_d
liblzf_cfile := src/lzf_c.c src/lzf_d.c
liblzf_objs  := $(addprefix $(objd)/, $(addsuffix .o, $(liblzf_files)))
liblzf_dbjs  := $(addprefix $(objd)/, $(addsuffix .fpic.o, $(liblzf_files)))
liblzf_deps  := $(addprefix $(dependd)/, $(addsuffix .d, $(liblzf_files))) \
                     $(addprefix $(dependd)/, $(addsuffix .fpic.d, $(liblzf_files)))
liblzf_spec  := $(version)-$(build_num)
#liblzf_ver   := $(major_num).$(minor_num)
liblzf_ver   := $(major_num)

$(libd)/liblzf.a: $(liblzf_objs)

$(libd)/liblzf.so: $(liblzf_dbjs)

lzf_files := lzf
lzf_cfile := src/lzf.c
lzf_objs  := $(addprefix $(objd)/, $(addsuffix .o, $(lzf_files)))
lzf_deps  := $(addprefix $(dependd)/, $(addsuffix .d, $(lzf_files)))
lzf_libs  := $(libd)/liblzf.so
lzf_lnk   := -llzf

$(bind)/lzf: $(lzf_objs) $(lzf_libs)

unlzf_objs  := $(lzf_objs)
unlzf_cfile := src/lzf.c
unlzf_deps  := $(lzf_deps)
unlzf_libs  := $(lzf_libs)
unlzf_lnk   := $(lzf_lnk)

# so small, no need for symlink
$(bind)/unlzf: $(unlzf_objs) $(unlzf_libs)

all_exes    += $(bind)/lzf $(bind)/unlzf
all_depends += $(liblzf_deps) $(lzf_deps)
all_dirs    += $(bind) $(libd) $(objd) $(dependd)
all_libs    += $(libd)/liblzf.a $(libd)/liblzf.so

all: $(all_libs) $(all_exes) cmake

.PHONY: cmake
cmake: CMakeLists.txt

.ONESHELL: CMakeLists.txt
CMakeLists.txt: .copr/Makefile
	@cat <<'EOF' > $@
	cmake_minimum_required (VERSION 3.9.0)
	if (POLICY CMP0111)
	  cmake_policy(SET CMP0111 OLD)
	endif ()
	project (lzf)
	include_directories (
	  include
	)
	if (CMAKE_SYSTEM_NAME STREQUAL "Windows")
	  if ($$<CONFIG:Release>)
	    add_compile_options (/arch:AVX2 /GL /std:c11 /wd4244)
	  else ()
	    add_compile_options (/arch:AVX2 /std:c11 /wd4244)
	  endif ()
	else ()
	  add_compile_options ($(cflags))
	endif ()
	add_library (lzf STATIC $(liblzf_cfile))
	EOF

# create directories
$(dependd):
	@mkdir -p $(all_dirs)

# remove target bins, objs, depends
.PHONY: clean
clean:
	rm -r -f $(bind) $(libd) $(objd) $(dependd)
	if [ "$(build_dir)" != "." ] ; then rmdir $(build_dir) ; fi

.PHONY: clean_dist
clean_dist:
	rm -rf dpkgbuild rpmbuild

.PHONY: clean_all
clean_all: clean clean_dist

$(dependd)/depend.make: $(dependd) $(all_depends)
	@echo "# depend file" > $(dependd)/depend.make
	@cat $(all_depends) >> $(dependd)/depend.make

ifeq (SunOS,$(lsb_dist))
remove_rpath = rpath -r
else
remove_rpath = chrpath -d
endif
# target used by rpmbuild
.PHONY: dist_bins
dist_bins: $(all_libs) $(all_exes)
	$(remove_rpath) $(libd)/liblzf.so
	$(remove_rpath) $(bind)/lzf
	$(remove_rpath) $(bind)/unlzf

# target for building installable rpm
.PHONY: dist_rpm
dist_rpm: srpm
	( cd rpmbuild && rpmbuild --define "-topdir `pwd`" -ba SPECS/lzf.spec )

ifeq ($(DESTDIR),)
# 'sudo make install' puts things in /usr/local/lib, /usr/local/include
install_prefix ?= /usr/local
else
# debuild uses DESTDIR to put things into debian/lzf/usr
install_prefix = $(DESTDIR)/usr
endif
install_lib_suffix ?=

install: everything
	install -d $(install_prefix)/lib$(install_lib_suffix)
	install -d $(install_prefix)/bin $(install_prefix)/include
	for f in $(libd)/liblzf.* ; do \
	if [ -h $$f ] ; then \
	cp -a $$f $(install_prefix)/lib$(install_lib_suffix) ; \
	else \
	install $$f $(install_prefix)/lib$(install_lib_suffix) ; \
	fi ; \
	done
	install $(bind)/lzf $(bind)/unlzf $(install_prefix)/bin
	install -m 644 include/*.h $(install_prefix)/include

# force a remake of depend using 'make -B depend'
.PHONY: depend
depend: $(dependd)/depend.make

# dependencies made by 'make depend'
-include $(dependd)/depend.make

$(objd)/%.o: src/%.c
	$(cc) $(cflags) $(includes) $(defines) $($(notdir $*)_includes) $($(notdir $*)_defines) -c $< -o $@

$(objd)/%.fpic.o: src/%.c
	$(cc) $(cflags) $(fpicflags) $(includes) $(defines) $($(notdir $*)_includes) $($(notdir $*)_defines) -c $< -o $@

$(libd)/%.a:
	ar rc $@ $($(*)_objs)

$(libd)/%.so:
	$(cc) $(soflag) $(cflags) -o $@.$($(*)_spec) -Wl,-soname=$(@F).$($(*)_ver) $($(*)_dbjs) $($(*)_dlnk) $(cpp_dll_lnk) $(sock_lib) $(math_lib) $(thread_lib) $(malloc_lib) $(dynlink_lib) && \
	cd $(libd) && ln -f -s $(@F).$($(*)_spec) $(@F).$($(*)_ver) && ln -f -s $(@F).$($(*)_ver) $(@F)

$(bind)/%:
	$(cc) $(cflags) $(rpath) -o $@ $($(*)_objs) -L$(libd) $($(*)_lnk) $(cpp_lnk) $(sock_lib) $(math_lib) $(thread_lib) $(malloc_lib) $(dynlink_lib)

$(dependd)/%.d: src/%.c
	$(cc) $(arch_cflags) $(defines) $(includes) $($(notdir $*)_includes) $($(notdir $*)_defines) -MM $< -MT $(objd)/$(*).o -MF $@

$(dependd)/%.fpic.d: src/%.c
	$(cc) $(arch_cflags) $(defines) $(includes) $($(notdir $*)_includes) $($(notdir $*)_defines) -MM $< -MT $(objd)/$(*).fpic.o -MF $@

