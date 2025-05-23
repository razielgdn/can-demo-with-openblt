#****************************************************************************************
#|  Description: Makefile for GNU ARM Embedded toolchain.
#|    File Name: makefile
#|
#|---------------------------------------------------------------------------------------
#|                          C O P Y R I G H T
#|---------------------------------------------------------------------------------------
#|   Copyright (c) 2017  by Feaser    http://www.feaser.com    All rights reserved
#|
#|---------------------------------------------------------------------------------------
#|                            L I C E N S E
#|---------------------------------------------------------------------------------------
#| This file is part of OpenBLT. OpenBLT is free software: you can redistribute it and/or
#| modify it under the terms of the GNU General Public License as published by the Free
#| Software Foundation, either version 3 of the License, or (at your option) any later
#| version.
#|
#| OpenBLT is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY;
#| without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR
#| PURPOSE. See the GNU General Public License for more details.
#|
#| You have received a copy of the GNU General Public License along with OpenBLT. It 
#| should be located in ".\Doc\license.html". If not, contact Feaser to obtain a copy.
#|
#****************************************************************************************

SHELL = sh

#|--------------------------------------------------------------------------------------|
#| Configure project name                                                               |
#|--------------------------------------------------------------------------------------|
PROJ_NAME=openblt_stm32f103

#|--------------------------------------------------------------------------------------|
#| Configure tool path                                                                  |
#|--------------------------------------------------------------------------------------|
# Configure the path to where the arm-none-eabi-gcc program is located. If the program
# is available on the path, then the TOOL_PATH variable can be left empty.
# Make sure to add a fordward slash at the end. Note that on Windows it should be in the
# 8.3 short pathname format with forward slashes. To obtain the pathname in the 8.3
# format, open the directory in the Windows command prompt and run the following command:
#  cmd /c for %A in ("%cd%") do @echo %~sA 
#TOOL_PATH=C:/PROGRA~2/GNUTOO~1/82018-~1/bin/
#TOOL_PATH=~/arm-tools/gcc-arm-none-eabi-10.3-2021.10/bin/
TOOL_PATH=~/arm-tools/arm-gnu-toolchain-14.2.rel1-x86_64-arm-none-eabi/bin/

#|--------------------------------------------------------------------------------------|
#| Collect project files                                                                |
#|--------------------------------------------------------------------------------------|
# Recursive wildcard function implementation. Example usages:
#   $(call rwildcard, , *.c *.h)   
#     --> Returns all *.c and *.h files in the current directory and below
#   $(call rwildcard, /lib/, *.c)
#     --> Returns all *.c files in the /lib directory and below
rwildcard = $(strip $(foreach d,$(wildcard $1*),$(call rwildcard,$d/,$2) $(filter $(subst *,%,$2),$d)))

# Collect all application files in the current directory and its subdirectories, but 
# exclude flash-layout.c as this one is directly included in a source file, when used.
PROJ_FILES  = $(filter-out flash_layout.c, $(call rwildcard, , *.c *.h *.s))
# Collect bootloader core files
PROJ_FILES += $(wildcard ../Source-openBLT/*.c)
PROJ_FILES += $(wildcard ../Source-openBLT/*.h)
# Collect bootloader port files
PROJ_FILES += $(wildcard ../Source-openBLT/ARMCM3_STM32F1/*.c)
PROJ_FILES += $(wildcard ../Source-openBLT/ARMCM3_STM32F1/*.h)
# Collect bootloader port compiler specific files
PROJ_FILES += $(wildcard ../Source-openBLT/ARMCM3_STM32F1/GCC/*.c)
PROJ_FILES += $(wildcard ../Source-openBLT/ARMCM3_STM32F1/GCC/*.h)


#|--------------------------------------------------------------------------------------|
#| Toolchain binaries                                                                   |
#|--------------------------------------------------------------------------------------|
RM = rm
CC = $(TOOL_PATH)arm-none-eabi-gcc
LN = $(TOOL_PATH)arm-none-eabi-gcc
OC = $(TOOL_PATH)arm-none-eabi-objcopy
OD = $(TOOL_PATH)arm-none-eabi-objdump
AS = $(TOOL_PATH)arm-none-eabi-gcc
SZ = $(TOOL_PATH)arm-none-eabi-size

#|--------------------------------------------------------------------------------------|
#| Filter project files
#|--------------------------------------------------------------------------------------|
PROJ_ASRCS  = $(filter %.s,$(foreach file,$(PROJ_FILES),$(notdir $(file))))
PROJ_CSRCS  = $(filter %.c,$(foreach file,$(PROJ_FILES),$(notdir $(file))))

#|--------------------------------------------------------------------------------------|
#| Set important path variables                                                         |
#|--------------------------------------------------------------------------------------|
VPATH    = $(foreach path,$(sort $(foreach file,$(PROJ_FILES),$(dir $(file)))) $(subst \,/,$(OBJ_PATH)),$(path) :)
OBJ_PATH = obj
BIN_PATH = bin
INC_PATH = $(patsubst %/,%,$(patsubst %,-I%,$(sort $(foreach file,$(filter %.h,$(PROJ_FILES)),$(dir $(file))))))
LIB_PATH  = 

#|--------------------------------------------------------------------------------------|
#| Options for toolchain binaries                                                       |
#|--------------------------------------------------------------------------------------|

STDFLAGS    = -mcpu=cortex-m3 -mthumb -std=gnu11 -fstack-usage -Wall -specs=nano.specs
STDFLAGS   += -fdata-sections -ffunction-sections -Wall -g -Wno-strict-aliasing
OPTFLAGS    = -Os -flto
DEPFLAGS  = -MT $@ -MMD -MP -MF $(OBJ_PATH)/$*.d
CFLAGS      = $(STDFLAGS) $(OPTFLAGS)
CFLAGS     += -DUSE_FULL_LL_DRIVER -DUSE_HAL_DRIVER -DSTM32F103xB
CFLAGS     += -D__weak="__attribute__((weak))" -D__packed="__attribute__((__packed__))"
CFLAGS     += $(INC_PATH)
AFLAGS      = $(CFLAGS)
LFLAGS      = $(STDFLAGS) $(OPTFLAGS)
LFLAGS     += -T"STM32F103C8TX_FLASH.ld" -Wl,-Map=$(BIN_PATH)/$(PROJ_NAME).map
LFLAGS     += -Wl,--gc-sections $(LIB_PATH)
OFLAGS      = -O srec
ODFLAGS     = -x
SZFLAGS     = -B -d
RMFLAGS     = -f

#|--------------------------------------------------------------------------------------|
#| Specify library files                                                                |
#|--------------------------------------------------------------------------------------|
LIBS = 

#|--------------------------------------------------------------------------------------|
#| Define targets                                                                       |
#|--------------------------------------------------------------------------------------|
AOBJS = $(patsubst %.s,%.o,$(PROJ_ASRCS))
COBJS = $(patsubst %.c,%.o,$(PROJ_CSRCS))

#|--------------------------------------------------------------------------------------|
#| Make ALL                                                                             |
#|--------------------------------------------------------------------------------------|
.PHONY: prepare_dirs
prepare_dirs:
	mkdir -p $(OBJ_PATH)
	mkdir -p $(BIN_PATH)
	
.PHONY: all
all: prepare_dirs $(BIN_PATH)/$(PROJ_NAME).srec 

$(BIN_PATH)/$(PROJ_NAME).srec : $(BIN_PATH)/$(PROJ_NAME).elf
	@$(OC) $< $(OFLAGS) $@
	@$(OD) $(ODFLAGS) $< > $(BIN_PATH)/$(PROJ_NAME).map
	@echo +++ Summary of memory consumption:
	@$(SZ) $(SZFLAGS) $<
	@echo +++ Build complete [$(notdir $@)]

$(BIN_PATH)/$(PROJ_NAME).elf : $(AOBJS) $(COBJS) 
	@echo +++ Linking [$(notdir $@)]
	@$(LN) $(LFLAGS) -o $@ $(patsubst %.o,$(OBJ_PATH)/%.o,$(^F)) $(LIBS)
	
.PHONY: $(BIN_PATH)/$(PROJ_NAME).bin
$(BIN_PATH)/$(PROJ_NAME).bin:
	 @$(OC)  -O binary $(BIN_PATH)/$(PROJ_NAME).elf $@
  
#|--------------------------------------------------------------------------------------|
#| Compile and assemble                                                                 |
#|--------------------------------------------------------------------------------------|
$(AOBJS): %.o: %.s
	@echo +++ Assembling [$(notdir $<)]
	@$(AS) $(AFLAGS) -c $< -o $(OBJ_PATH)/$(@F)

$(COBJS): %.o: %.c $(OBJ_PATH)/%.d
	@echo +++ Compiling [$(notdir $<)]
	@$(CC) $(DEPFLAGS) $(CFLAGS) -c $< -o $(OBJ_PATH)/$(@F)


#|--------------------------------------------------------------------------------------|
#| Make CLEAN                                                                           |
#|--------------------------------------------------------------------------------------|
.PHONY: clean
clean: 
	@echo +++ Cleaning build environment
	@$(RM) $(RMFLAGS) $(foreach file,$(AOBJS),$(OBJ_PATH)/$(file))
	@$(RM) $(RMFLAGS) $(foreach file,$(COBJS),$(OBJ_PATH)/$(file))
	@$(RM) $(RMFLAGS) $(patsubst %.o,%.lst,$(foreach file,$(AOBJS),$(OBJ_PATH)/$(file)))
	@$(RM) $(RMFLAGS) $(patsubst %.o,%.lst,$(foreach file,$(COBJS),$(OBJ_PATH)/$(file)))
	@$(RM) $(RMFLAGS) $(patsubst %.o,%.d,$(foreach file,$(COBJS),$(OBJ_PATH)/$(file)))
	@$(RM) $(RMFLAGS) $(patsubst %.o,%.su,$(foreach file,$(COBJS),$(OBJ_PATH)/$(file)))
	@$(RM) $(RMFLAGS) $(BIN_PATH)/$(PROJ_NAME).elf $(BIN_PATH)/$(PROJ_NAME).*.su $(BIN_PATH)/$(PROJ_NAME).bin $(BIN_PATH)/$(PROJ_NAME).map
	@$(RM) $(RMFLAGS) $(BIN_PATH)/$(PROJ_NAME).srec
	@echo +++ Clean complete
	
#|--------------------------------------------------------------------------------------|
#| Dependency generation                                                                |
#|--------------------------------------------------------------------------------------|
DEPFILES := $(PROJ_CSRCS:%.c=$(OBJ_PATH)/%.d)
$(DEPFILES):
include $(wildcard $(DEPFILES))	

