# SPDX-FileCopyrightText: 2020 Efabless Corporation
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
# SPDX-License-Identifier: Apache-2.0

## Caravel Pointers
CARAVEL_ROOT ?= ../../../caravel
CARAVEL_PATH ?= $(CARAVEL_ROOT)
CARAVEL_FIRMWARE_PATH = $(CARAVEL_PATH)/verilog/dv/caravel
CARAVEL_VERILOG_PATH  = $(CARAVEL_PATH)/verilog
CARAVEL_RTL_PATH = $(CARAVEL_VERILOG_PATH)/rtl
CARAVEL_BEHAVIOURAL_MODELS = $(CARAVEL_VERILOG_PATH)/dv/caravel

## User Project Pointers
UPRJ_VERILOG_PATH ?= ../../../verilog
UPRJ_RTL_PATH = $(UPRJ_VERILOG_PATH)/rtl
UPRJ_GL_PATH  = $(UPRJ_VERILOG_PATH)/gl
UPRJ_BEHAVIOURAL_MODELS = ../

## RISCV GCC 
GCC_PATH?=/ef/apps/bin
GCC_PREFIX?=riscv32-unknown-elf
PDK_PATH?=/ef/tech/SW/sky130A

## Simulation mode: RTL/GL
SIM?=RTL

# include the modules that cocotb needs for test
export COCOTB_REDUCED_LOG_FMT=1
export PYTHONPATH := $(UPRJ_RTL_PATH)/wrapped_silife/silife/test
export LIBPYTHON_LOC=$(shell cocotb-config --libpython)

.SUFFIXES:

PATTERN = silife_test

all:  coco_test

hex:  ${PATTERN:=.hex}

show:
	gtkwave $(CARAVEL_ROOT)/../verilog/dv/caravel_silife_test/silife.vcd silife.gtkw

coco_test: ${PATTERN}.hex
	rm -rf sim_build/
	mkdir sim_build/
	iverilog -o sim_build/sim.vvp -s silife_tb silife_tb.v -DSIM -DFUNCTIONAL \
	-I $(PDK_PATH) \
	-I $(CARAVEL_BEHAVIOURAL_MODELS) -I $(CARAVEL_RTL_PATH) \
	-I $(UPRJ_BEHAVIOURAL_MODELS)    -I $(UPRJ_RTL_PATH)
	
	MODULE=test_silife vvp -M $$(cocotb-config --prefix)/cocotb/libs -m libcocotbvpi_icarus sim_build/sim.vvp
	! grep failure results.xml

coco_test_gl: ${PATTERN}.hex
	rm -rf sim_build/
	mkdir sim_build/
	iverilog -o sim_build/sim.vvp -s silife_tb silife_tb.v -DSIM -DFUNCTIONAL \
	-I $(PDK_PATH) \
	-I $(CARAVEL_BEHAVIOURAL_MODELS) -I $(CARAVEL_RTL_PATH) \
	-I $(UPRJ_BEHAVIOURAL_MODELS)    -I $(UPRJ_RTL_PATH) -I $(UPRJ_GL_PATH)
	
	MODULE=test_silife vvp -M $$(cocotb-config --prefix)/cocotb/libs -m libcocotbvpi_icarus sim_build/sim.vvp
	! grep failure results.xml

%.vvp: %_tb.v %.hex
ifeq ($(SIM),RTL)
	iverilog -DFUNCTIONAL -DSIM -I $(PDK_PATH) \
	-I $(CARAVEL_BEHAVIOURAL_MODELS) -I $(CARAVEL_RTL_PATH) \
	-I $(UPRJ_BEHAVIOURAL_MODELS)    -I $(UPRJ_RTL_PATH) \
	$< -o $@ 
else  
	iverilog -DFUNCTIONAL -DSIM -DGL -I $(PDK_PATH) \
	-I $(CARAVEL_BEHAVIOURAL_MODELS) -I $(CARAVEL_RTL_PATH) -I $(CARAVEL_VERILOG_PATH) \
	-I $(UPRJ_BEHAVIOURAL_MODELS) -I$(UPRJ_RTL_PATH)   -I $(UPRJ_VERILOG_PATH) \
	$< -o $@ 
endif

%.vcd: %.vvp
	vvp $<

test:
	echo ${GCC_PATH}/${GCC_PREFIX}-gcc -I $(CARAVEL_PATH) -march=rv32imc -mabi=ilp32 -Wl,-Bstatic,-T,$(CARAVEL_FIRMWARE_PATH)/sections.lds,--strip-debug -ffreestanding -nostdlib -o $@ $(CARAVEL_FIRMWARE_PATH)/start.s $<

%.elf: %.c $(CARAVEL_FIRMWARE_PATH)/sections.lds $(CARAVEL_FIRMWARE_PATH)/start.s
	${GCC_PATH}/${GCC_PREFIX}-gcc -I $(CARAVEL_PATH) -march=rv32imc -mabi=ilp32 -Wl,-Bstatic,-T,$(CARAVEL_FIRMWARE_PATH)/sections.lds,--strip-debug -ffreestanding -nostdlib -o $@ $(CARAVEL_FIRMWARE_PATH)/start.s $<

%.hex: %.elf
	${GCC_PATH}/${GCC_PREFIX}-objcopy -O verilog $< $@ 
	# to fix flash base address
	sed -i 's/@10000000/@00000000/g' $@

%.bin: %.elf
	${GCC_PATH}/${GCC_PREFIX}-objcopy -O binary $< /dev/stdout | tail -c +1048577 > $@

# ---- Clean ----

clean:
	rm -f *.elf *.hex *.bin *.vvp *.vcd *.log

.PHONY: clean hex all
