---
layout: post
title:  GBA Development setup in macOS
date:   2020-01-10 15:00:00 -0600
description: GBA Development setup in macOS
categories: GameDev
---

To develop in GBA we need the tools to do it, and installing it in macOS is simple even if they take a few steps.

## Install pacman

The first thing we must do is install pacmac, which will allow us to download the devkitpro for the GBA,
 to install it we must go to the following URL:

 [https://github.com/devkitPro/pacman/releases/tag/v1.0.2](https://github.com/devkitPro/pacman/releases/tag/v1.0.2)

 And we will download **devkitpro-pacman-installer.pkg**. Once it has been downloaded, we install the package. 
 The following will appear:

 ![devkitpronopermission](/assets/img/gamedev/devkitpronopermission.png)

 So, we need to go to **System Preferences -> Security & Privacy**  and allow the permissions:

![Screenshot-2020-01-10-12.45.08](/assets/img/gamedev/Screenshot-2020-01-10-12.45.08.png)

Follow the steps to install the package.

Once that the installation was finished, we need to update our **.bashrc** or **.zshrc** file.

## Update our environments vars

With the installation of pacman, we need to tell to our system where the files are located, so we need to 
update our **.bashrc** or **.zshrc** file.

Open your file and copy the next exports

```shell
export DEVKITPRO=/opt/devkitpro
export DEVKITARM=${DEVKITPRO}/devkitARM
export DEVKITPPC=${DEVKITPRO}/devkitPPC
export PATH=${DEVKITPRO}/tools/bin:$PATH
```

## Install Xcode Tools

Now we need to install the Xcode tools to use the compiler and other utilities. In your terminal write the follow command:

```
xcode-select --install
```

## Install devkitpro

Now we have all the tools to install our GBA compiler, so is time to write in the terminal.

```
sudo dkp-pacman -S gba-dev
```

Enter your password. You will prompt with options to get more control about the things that your going to install, 
for now we are going to install all of them. So, hit Enter and then Enter to confirm.

If you have Catalina you will get an error.

```
error: Partition / is mounted read only
error: not enough free disk space
error: failed to commit transaction (not enough free disk space)
```

This is because Catalina root partition is only for read, and you cannot install anything here, 
but you can have a workaround installing with this command:

```
sudo dkp-pacman -S gba-dev -r /System/Volumes/Data
```

Again, hit Enter, and Enter.

With devkitpro installed, your are not able to create a GBA game, so we need to test if our tools was installed correctly.

## Hello World GBA

In your Desktop or whenever you want, create a folder named **hellogba**, inside create a 
new file named **Makefile**, this file will help us to compile our code and generate a rom

Copy the following

```cmake
#---------------------------------------------------------------------------------
.SUFFIXES:
#---------------------------------------------------------------------------------

ifeq ($(strip $(DEVKITARM)),)
$(error "Please set DEVKITARM in your environment. export DEVKITARM=<path to>devkitARM")
endif

include $(DEVKITARM)/gba_rules

#---------------------------------------------------------------------------------
# TARGET is the name of the output
# BUILD is the directory where object files & intermediate files will be placed
# SOURCES is a list of directories containing source code
# INCLUDES is a list of directories containing extra header files
# DATA is a list of directories containing binary data
# GRAPHICS is a list of directories containing files to be processed by grit
#
# All directories are specified relative to the project directory where
# the makefile is found
#
#---------------------------------------------------------------------------------
TARGET		:= rom/$(notdir $(CURDIR))
BUILD		:= build
SOURCES		:= src
INCLUDES	:= include
DATA		:=
MUSIC		:=

#---------------------------------------------------------------------------------
# options for code generation
#---------------------------------------------------------------------------------
ARCH	:=	-mthumb -mthumb-interwork

CFLAGS	:=	-g -Wall -O2\
		-mcpu=arm7tdmi -mtune=arm7tdmi\
		$(ARCH)

CFLAGS	+=	$(INCLUDE)

CXXFLAGS	:=	$(CFLAGS) -fno-rtti -fno-exceptions

ASFLAGS	:=	-g $(ARCH)
LDFLAGS	=	-g $(ARCH) -Wl,-Map,$(notdir $*.map)

#---------------------------------------------------------------------------------
# any extra libraries we wish to link with the project
#---------------------------------------------------------------------------------
LIBS	:= -lmm -lgba


#---------------------------------------------------------------------------------
# list of directories containing libraries, this must be the top level containing
# include and lib
#---------------------------------------------------------------------------------
LIBDIRS	:=	$(LIBGBA)

#---------------------------------------------------------------------------------
# no real need to edit anything past this point unless you need to add additional
# rules for different file extensions
#---------------------------------------------------------------------------------


ifneq ($(BUILD),$(notdir $(CURDIR)))
#---------------------------------------------------------------------------------

export OUTPUT	:=	$(CURDIR)/$(TARGET)

export VPATH	:=	$(foreach dir,$(SOURCES),$(CURDIR)/$(dir)) \
			$(foreach dir,$(DATA),$(CURDIR)/$(dir)) \
			$(foreach dir,$(GRAPHICS),$(CURDIR)/$(dir))

export DEPSDIR	:=	$(CURDIR)/$(BUILD)

CFILES		:=	$(foreach dir,$(SOURCES),$(notdir $(wildcard $(dir)/*.c)))
CPPFILES	:=	$(foreach dir,$(SOURCES),$(notdir $(wildcard $(dir)/*.cpp)))
SFILES		:=	$(foreach dir,$(SOURCES),$(notdir $(wildcard $(dir)/*.s)))
BINFILES	:=	$(foreach dir,$(DATA),$(notdir $(wildcard $(dir)/*.*)))

ifneq ($(strip $(MUSIC)),)
	export AUDIOFILES	:=	$(foreach dir,$(notdir $(wildcard $(MUSIC)/*.*)),$(CURDIR)/$(MUSIC)/$(dir))
	BINFILES += soundbank.bin
endif

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

export OFILES_BIN := $(addsuffix .o,$(BINFILES))

export OFILES_SOURCES := $(CPPFILES:.cpp=.o) $(CFILES:.c=.o) $(SFILES:.s=.o)

export OFILES := $(OFILES_BIN) $(OFILES_SOURCES)

export HFILES := $(addsuffix .h,$(subst .,_,$(BINFILES)))

export INCLUDE	:=	$(foreach dir,$(INCLUDES),-iquote $(CURDIR)/$(dir)) \
					$(foreach dir,$(LIBDIRS),-I$(dir)/include) \
					-I$(CURDIR)/$(BUILD)

export LIBPATHS	:=	$(foreach dir,$(LIBDIRS),-L$(dir)/lib)

.PHONY: $(BUILD) clean

#---------------------------------------------------------------------------------
$(BUILD):
	@[ -d $@ ] || mkdir -p $@
	@$(MAKE) --no-print-directory -C $(BUILD) -f $(CURDIR)/Makefile

#---------------------------------------------------------------------------------
clean:
	@echo clean ...
	@rm -fr $(BUILD) $(TARGET).elf $(TARGET).gba


#---------------------------------------------------------------------------------
else

#---------------------------------------------------------------------------------
# main targets
#---------------------------------------------------------------------------------

$(OUTPUT).gba	:	$(OUTPUT).elf

$(OUTPUT).elf	:	$(OFILES)

$(OFILES_SOURCES) : $(HFILES)

#---------------------------------------------------------------------------------
# The bin2o rule should be copied and modified
# for each extension used in the data directories
#---------------------------------------------------------------------------------

#---------------------------------------------------------------------------------
# rule to build soundbank from music files
#---------------------------------------------------------------------------------
soundbank.bin soundbank.h : $(AUDIOFILES)
#---------------------------------------------------------------------------------
	@mmutil $^ -osoundbank.bin -hsoundbank.h

#---------------------------------------------------------------------------------
# This rule links in binary data with the .bin extension
#---------------------------------------------------------------------------------
%.bin.o	%_bin.h :	%.bin
#---------------------------------------------------------------------------------
	@echo $(notdir $<)
	@$(bin2o)


-include $(DEPSDIR)/*.d
#---------------------------------------------------------------------------------------
endif
#---------------------------------------------------------------------------------------
```
This template include all necessary things to compile and generate a rom, for now, the most important lines are:

```cmake
TARGET		:= rom/$(notdir $(CURDIR))
BUILD		:= build
SOURCES		:= src
INCLUDES	:= include
DATA		:=
MUSIC		:=
```

That are the directories where you’re going to write your code and generate the output. Create the **rom**, **build**, **src** and  **include** folders.

Inside src directory, create a new file named **hello.c** and write the next code

```cpp
int main()
{
    *(unsigned int*)0x04000000 = 0x0403;

    ((unsigned short*)0x06000000)[120+80*240] = 0x001F;
    ((unsigned short*)0x06000000)[136+80*240] = 0x03E0;
    ((unsigned short*)0x06000000)[120+96*240] = 0x7C00;

    while(1);

    return 0;
}
```

This code show three points in the screen, we are going to use only to test our environment.

Now is time to compile!

In your terminal, write this

```
make
```

And if you see this message

```
linking cartridge
built ... hellogba.gba
ROM fixed!
```

Your environment is ready to work with. In the **rom** folder, you are going to see two files, **hello.gba** and **hello.elf**, the last one is util for debugging.

Now we need an emulator to test our “game”.

I am using mGBA and you can download it here [https://mgba.io/](https://mgba.io/)

Load the hello.gba in the emulator and you should see this screen

![hellogba](/assets/img/gamedev/hellogba.png)

Now, you are ready to create GBA games!

## Resources

Some resources to start GBA development

[https://www.youtube.com/channel/UCQvQbVuWMM7OYTVrmGQq0hA](https://www.youtube.com/channel/UCQvQbVuWMM7OYTVrmGQq0hA)

[http://www.coranac.com/tonc/text/toc.htm](http://www.coranac.com/tonc/text/toc.htm)

[http://www.loirak.com/gameboy/gbatutor.php](http://www.loirak.com/gameboy/gbatutor.php)

[https://github.com/gbdev/awesome-gbadev](https://github.com/gbdev/awesome-gbadev)

[https://www.reinterpretcast.com/writing-a-game-boy-advance-game](https://www.reinterpretcast.com/writing-a-game-boy-advance-game)
