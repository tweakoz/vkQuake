# GNU Makefile for vkQuake unix targets, Sep. 10, 2014
# You need the SDL library fully installed.
# "make DEBUG=1" to build a debug client.
# "make SDL_CONFIG=/path/to/sdl-config" for unusual SDL installations.
# "make DO_USERDIRS=1" to enable user directories support
# "make VULKAN_SDK=/path/to/sdk" if it is not already in path

# Enable/Disable user directories support
DO_USERDIRS=0

### Enable/Disable codecs for streaming music support
USE_CODEC_WAVE=1
USE_CODEC_FLAC=0
USE_CODEC_MP3=1
USE_CODEC_VORBIS=1
USE_CODEC_OPUS=0
# either mikmod, or xmp
USE_CODEC_MIKMOD=0
USE_CODEC_XMP=0
USE_CODEC_UMX=0

# which library to use for mp3 decoding: mad or mpg123
MP3LIB=mad
# which library to use for ogg decoding: vorbis or tremor
VORBISLIB=vorbis

# ---------------------------
# Helper functions
# ---------------------------

check_gcc = $(shell if echo | $(CXX) $(1) -Werror -S -o /dev/null -xc - > /dev/null 2>&1; then echo "$(1)"; else echo "$(2)"; fi;)

# ---------------------------

HOST_OS := $(shell uname|sed -e s/_.*//|tr '[:upper:]' '[:lower:]')

DEBUG   ?= 0

# ---------------------------
# build variables
# ---------------------------

CXX ?= g++
LINKER = $(CXX)

#CPUFLAGS= -mtune=i686
#CPUFLAGS= -march=pentium4
#CPUFLAGS= -mtune=k8
#CPUFLAGS= -march=atom
CPUFLAGS=
DFLAGS ?=
CCFLAGS ?= -Wall -Wno-trigraphs
CCFLAGS += $(CPUFLAGS)
ifneq ($(DEBUG),0)
DFLAGS += -D_DEBUG
CCFLAGS += -g
do_strip=
DFLAGS += -DNDEBUG
CCFLAGS += -O2
CCFLAGS += $(call check_gcc,-fweb,)
CCFLAGS += $(call check_gcc,-frename-registers,)
cmd_strip=$(STRIP) $(1)
endif

ifneq ($(VULKAN_SDK),)
LDFLAGS += -L$(VULKAN_SDK)/lib
CCFLAGS += -I$(VULKAN_SDK)/include
endif

ifeq ($(DO_USERDIRS),1)
CCFLAGS += -DDO_USERDIRS=1
endif

### X11BASE only gets used if its in an unusual place

X11DIRS := /usr/X11R7 /usr/local/X11R7 /usr/X11R6 /usr/local/X11R6
X11BASE_GUESS := $(shell \
	if [ -e /usr/include/X11/Xlib.h ] && \
		[ -e /usr/lib/libX11.a ]; then exit 0; fi; \
	if [ -e /usr/local/include/X11/Xlib.h ] && \
		[ -e /usr/local/lib/libX11.a ]; then exit 0; fi; \
	for DIR in $(X11DIRS); do \
            if [ -e $$DIR/include/X11/Xlib.h ] && \
		[ -e $$DIR/lib/libX11.a ]; then echo $$DIR; break; fi; \
	done )

X11BASE	?= $(X11BASE_GUESS)

ifneq ($(X11BASE),)
LDFLAGS+= -L$(X11BASE)/lib
CCFLAGS += -I$(X11BASE)/include
endif

CCFLAGS += -DUSE_SDL2

SDL_CONFIG ?= sdl2-config
SDL_CCFLAGS := $(shell $(SDL_CONFIG) --cflags)
SDL_LIBS   := $(shell $(SDL_CONFIG) --libs)

ifeq ($(HOST_OS),sunos)
NET_LIBS   :=-lsocket -lnsl -lresolv
else
NET_LIBS   :=
endif

ifneq ($(VORBISLIB),vorbis)
ifneq ($(VORBISLIB),tremor)
$(error Invalid VORBISLIB setting)
endif
endif
ifneq ($(MP3LIB),mpg123)
ifneq ($(MP3LIB),mad)
$(error Invalid MP3LIB setting)
endif
endif
ifeq ($(MP3LIB),mad)
mp3_obj=snd_mp3
lib_mp3dec=-lmad
endif
ifeq ($(MP3LIB),mpg123)
mp3_obj=snd_mpg123
lib_mp3dec=-lmpg123
endif
ifeq ($(VORBISLIB),vorbis)
cpp_vorbisdec=
lib_vorbisdec=-lvorbisfile -lvorbis -logg
endif
ifeq ($(VORBISLIB),tremor)
cpp_vorbisdec=-DVORBIS_USE_TREMOR
lib_vorbisdec=-lvorbisidec -logg
endif

CODECLIBS  :=
ifeq ($(USE_CODEC_WAVE),1)
CCFLAGS+= -DUSE_CODEC_WAVE
endif
ifeq ($(USE_CODEC_FLAC),1)
CCFLAGS+= -DUSE_CODEC_FLAC
CODECLIBS+= -lFLAC
endif
ifeq ($(USE_CODEC_OPUS),1)
# opus and opusfile put their *.h under <includedir>/opus,
# but they include the headers without the opus directory
# prefix and rely on pkg-config. ewww...
CCFLAGS+= -DUSE_CODEC_OPUS
CCFLAGS+= $(shell pkg-config --cflags opusfile)
CODECLIBS+= $(shell pkg-config --libs   opusfile)
endif
ifeq ($(USE_CODEC_VORBIS),1)
CCFLAGS+= -DUSE_CODEC_VORBIS $(cpp_vorbisdec)
CODECLIBS+= $(lib_vorbisdec)
endif
ifeq ($(USE_CODEC_MP3),1)
CCFLAGS+= -DUSE_CODEC_MP3
CODECLIBS+= $(lib_mp3dec)
endif
ifeq ($(USE_CODEC_MIKMOD),1)
CCFLAGS+= -DUSE_CODEC_MIKMOD
CODECLIBS+= -lmikmod
endif
ifeq ($(USE_CODEC_XMP),1)
CCFLAGS+= -DUSE_CODEC_XMP
CODECLIBS+= -lxmp
endif
ifeq ($(USE_CODEC_UMX),1)
CCFLAGS+= -DUSE_CODEC_UMX
endif

COMMON_LIBS:= -lm

ifneq ($(HOST_OS),darwin)
COMMON_LIBS+= -lX11-xcb -lvulkan
CCFLAGS+= -DLINUX
else
	COMMON_LIBS+= -lMoltenVK
endif

LIBS := $(COMMON_LIBS) $(NET_LIBS) $(CODECLIBS)

# ---------------------------

CXXFLAGS = $(CCFLAGS)
CXXFLAGS += -std=c++17

# ---------------------------
# targets
# ---------------------------

.PHONY:	clean debug release

DEFAULT_TARGET := vkquake

# ----------------------------------------------------------------------------
# objects
# ----------------------------------------------------------------------------

MUSIC_OBJS:= bgmusic.o \
	snd_codec.o \
	snd_flac.o \
	snd_wave.o \
	snd_vorbis.o \
	snd_opus.o \
	$(mp3_obj).o \
	snd_mp3tag.o \
	snd_mikmod.o \
	snd_xmp.o \
	snd_umx.o
COMOBJ_SND := snd_dma.o snd_mix.o snd_mem.o $(MUSIC_OBJS)
SYSOBJ_SND := snd_sdl.o
SYSOBJ_CDA := cd_sdl.o
SYSOBJ_INPUT := in_sdl.o
SYSOBJ_GL_VID:= gl_vidsdl.o
SYSOBJ_NET := net_bsd.o net_udp.o
SYSOBJ_SYS := pl_linux.o sys_sdl_unix.o
SYSOBJ_MAIN:= main_sdl.o
SYSOBJ_RES :=

SHADER_OBJS = \
	alias_frag.o \
	alias_alphatest_frag.o \
	alias_vert.o \
	basic_alphatest_frag.o \
	screen_warp_comp.o \
	screen_warp_rgba8_comp.o \
	cs_tex_warp_comp.o \
	basic_frag.o \
	basic_notex_frag.o \
	basic_vert.o \
	sky_layer_frag.o \
	sky_layer_vert.o \
	postprocess_frag.o \
	postprocess_vert.o \
	world_frag.o \
	world_vert.o \
	showtris_frag.o \
	showtris_vert.o

GLOBJS = \
	gl_refrag.o \
	gl_rlight.o \
	gl_rmain.o \
	gl_fog.o \
	gl_rmisc.o \
	r_part.o \
	r_world.o \
	gl_screen.o \
	gl_sky.o \
	gl_warp.o \
	$(SYSOBJ_GL_VID) \
	gl_draw.o \
	image.o \
	gl_texmgr.o \
	gl_mesh.o \
	gl_heap.o \
	r_sprite.o \
	r_alias.o \
	r_brush.o \
	gl_model.o

OBJS := strlcat.o \
	strlcpy.o \
	$(GLOBJS) \
	$(SYSOBJ_INPUT) \
	$(COMOBJ_SND) \
	$(SYSOBJ_SND) \
	$(SYSOBJ_CDA) \
	$(SYSOBJ_NET) \
	net_dgrm.o \
	net_loop.o \
	net_main.o \
	chase.o \
	cl_demo.o \
	cl_input.o \
	cl_main.o \
	cl_parse.o \
	cl_tent.o \
	console.o \
	keys.o \
	menu.o \
	sbar.o \
	view.o \
	wad.o \
	cmd.o \
	common.o \
	crc.o \
	cvar.o \
	cfgfile.o \
	host.o \
	host_cmd.o \
	mathlib.o \
	pr_cmds.o \
	pr_edict.o \
	pr_exec.o \
	sv_main.o \
	sv_move.o \
	sv_phys.o \
	sv_user.o \
	world.o \
	zone.o \
	$(SYSOBJ_SYS) $(SYSOBJ_MAIN) $(SYSOBJ_RES)

# ---------------------------
# pattern rules
# ---------------------------

%.o:	%.cpp
	$(CXX) $(DFLAGS) -c $(CXXFLAGS) $(SDL_CCFLAGS) -o $@ $<
%.o:	../Shaders/Compiled/%.c
	$(CC) $(DFLAGS) -c $(CCFLAGS) $(SDL_CCFLAGS) -o $@ $^

# ------------------------
# Linux build rules
# ------------------------

vkquake:	$(OBJS) $(SHADER_OBJS)
	$(LINKER) $(OBJS) $(SHADER_OBJS) $(LDFLAGS) $(LIBS) $(SDL_LIBS) -o $@

image.o: lodepng.cpp lodepng.h stb_image_write.h

release:	vkquake
debug:
	$(error Use "make DEBUG=1")

clean:
	rm -f $(shell find . \( -name '*~' -o -name '#*#' -o -name '*.o' -o -name '*.res' -o -name $(DEFAULT_TARGET) \) -print)

prefix ?= /usr
exec_prefix ?= $(prefix)
bindir ?= $(exec_prefix)/games
sbindir ?= $(exec_prefix)/sbin
INSTALL ?= install
INSTALL_PROGRAM ?= $(INSTALL)
INSTALL_DATA ?= ${INSTALL} -m 644
install: vkquake
	$(INSTALL_PROGRAM) $(CURDIR)/vkquake $(DESTDIR)$(bindir)/vkquake
