#---------------------------------------------------------------------------------
.SUFFIXES:
#---------------------------------------------------------------------------------
ifeq ($(strip $(DEVKITARM)),)
$(error "Please set DEVKITARM in your environment. export DEVKITARM=<path to>devkitARM")
endif

include $(DEVKITARM)/ds_rules

#---------------------------------------------------------------------------------
# BUILD is the directory where object files & intermediate files will be placed
# SOURCES is a list of directories containing source code
# INCLUDES is a list of directories containing extra header files
# DATA is a list of directories containing binary files
# all directories are relative to this makefile
#---------------------------------------------------------------------------------
STORAGE		?=	rpg
BUILD		:=	$(STORAGE)/build
SOURCES		:=	source
INCLUDES	:=	include $(BUILD)
DATA		:=	data

#---------------------------------------------------------------------------------
# options for code generation
#---------------------------------------------------------------------------------
ARCH	:=	-mthumb
#-mthumb-interwork

CFLAGS	:=	-g -Wall -Os\
		-mcpu=arm7tdmi -mtune=arm7tdmi -fomit-frame-pointer\
		-ffast-math \
		$(ARCH)

CFLAGS	+=	$(INCLUDE) -DARM7 -fno-dwarf2-cfi-asm
CXXFLAGS	:=	$(CFLAGS) -fno-rtti -fno-exceptions -fno-rtti --ansi


ASFLAGS	:=	-g $(ARCH)
LDFLAGS	=	-specs=../../specs/ds_arm7.specs -g $(ARCH) -Wl,-Map,$(notdir $*).map

LIBS	:=	-lunds7

#---------------------------------------------------------------------------------
# list of directories containing libraries, this must be the top level containing
# include and lib
#---------------------------------------------------------------------------------
LIBDIRS	:=	$(LIBUNDS)


#---------------------------------------------------------------------------------
# no real need to edit anything past this point unless you need to add additional
# rules for different file extensions
#---------------------------------------------------------------------------------
ifneq ($(notdir $(BUILD)),$(notdir $(CURDIR)))
#---------------------------------------------------------------------------------

export ARM7BIN	:=	$(CURDIR)/$(STORAGE)/$(TARGET).arm7
export ARM7ELF	:=	$(CURDIR)/$(STORAGE)/$(TARGET).arm7.elf
export DEPSDIR	:=	$(CURDIR)/$(BUILD)

export VPATH	:=	$(foreach dir,$(SOURCES),$(CURDIR)/$(dir))

CFILES		:=	$(foreach dir,$(SOURCES),$(notdir $(wildcard $(dir)/*.c)))
CPPFILES	:=	$(foreach dir,$(SOURCES),$(notdir $(wildcard $(dir)/*.cpp)))
SFILES		:=	$(foreach dir,$(SOURCES),$(notdir $(wildcard $(dir)/*.s)))
BINFILES	:=	$(foreach dir,$(DATA),$(notdir $(wildcard $(dir)/*.*)))

export OFILES	:=	$(addsuffix .o,$(BINFILES)) \
			$(CPPFILES:.cpp=.o) $(CFILES:.c=.o) $(SFILES:.s=.o)

export INCLUDE	:=	$(foreach dir,$(INCLUDES),-I$(CURDIR)/$(dir)) \
			$(foreach dir,$(LIBDIRS),-I$(dir)/include) \
			-I$(CURDIR)/$(BUILD)

export LIBPATHS	:=	$(foreach dir,$(LIBDIRS),-L$(dir)/lib)

#---------------------------------------------------------------------------------
# use CXX for linking C++ projects, CC for standard C
#---------------------------------------------------------------------------------
ifeq ($(strip $(CPPFILES)),)
#---------------------------------------------------------------------------------
	export LD	:=	$(CC)
#---------------------------------------------------------------------------------
else
#---------------------------------------------------------------------------------
	export LD	:=	$(CXX)
#---------------------------------------------------------------------------------
endif
#---------------------------------------------------------------------------------

.PHONY: $(BUILD) clean

#---------------------------------------------------------------------------------
$(BUILD):
	@[ -d $@ ] || mkdir -p $@
	@make --no-print-directory -C $(BUILD) -f $(CURDIR)/Makefile

#---------------------------------------------------------------------------------
clean:
	@echo clean ...
	@rm -fr $(STORAGE) *.elf


#---------------------------------------------------------------------------------
else

DEPENDS	:=	$(OFILES:.o=.d)

#---------------------------------------------------------------------------------
# main targets
#---------------------------------------------------------------------------------
$(ARM7BIN)	:	$(ARM7ELF)
	@$(OBJCOPY) -O binary $< $@
	@echo built ... $(notdir $@)


$(ARM7ELF)	:	$(OFILES)
	@echo linking $(notdir $@)
	@$(LD)  $(LDFLAGS) $(OFILES) $(LIBPATHS) $(LIBS) -o $@


#---------------------------------------------------------------------------------
# you need a rule like this for each extension you use as binary data
#---------------------------------------------------------------------------------
%.bin.o	:	%.bin
#---------------------------------------------------------------------------------
	@echo $(notdir $<)
	@$(bin2o)

-include $(DEPENDS)

#---------------------------------------------------------------------------------------
endif
#---------------------------------------------------------------------------------------