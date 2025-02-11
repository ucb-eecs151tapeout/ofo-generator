#########################################################################################
# pre-process 151 core verilog into a single blackbox file
# TODO elam how can we handle multiple `riscv_top`s? do we need to manually unique-ify the names
#########################################################################################


# name of output pre-processed verilog file
vsrc_dir =$(abspath ./)
proj_name = kevin-kore
# TODO: better way  to do this
#sram_dir = "/home/ff/eecs251b/sky130/sram22_sky130_macros"
sram_dir = "/scratch/ee198-20-tab/sram22_sky130_macros" 

  # this is tuning wmask to a wire rather than a 1 width bus in sram22_64x24m4w24 bc it is otherwise incompatible with the lib
	# NOTE  for some reason it looks like it's inserting a space after sram_dir which breaks the paths. not sure why, manually fixing for now
	# note this only works for sim, this causes the sram macros to not be placed if you try to run synthesis (bc it maps the sram modules to some trivial verilog)
sram_include_str = "" # "\`include \"$(sram_dir)/sram22_256x32m4w8/sram22_256x32m4w8.v\"\n\`include \"$(sram_dir)/sram22_64x32m4w8/sram22_64x32m4w8.v\"\n\`include \"$(sram_dir)/sram22_512x64m4w8/sram22_512x64m4w8.v\"\n\`include \"$(sram_dir)/sram22_64x24m4w24/sram22_64x24m4w24.v\"\n"

PREPROC_VERILOG = $(vsrc_dir)/$(proj_name).preprocessed.v

.PHONY: default $(PREPROC_VERILOG)
default: $(PREPROC_VERILOG)

#########################################################################################
# includes and vsrcs
#########################################################################################


lookup_srcs = $(shell find -L $(1)/ -name target -prune -o -iname "*.$(2)" -print 2> /dev/null)

VLOG_DIR = $(vsrc_dir)/$(proj_name)/src

ALL_VSRCS = $(VLOG_DIR)/*.v

INC_DIRS = $(VLOG_DIR)/

#########################################################################################
# pre-process using custom script to replace the includes (but leave rest unaffected)
#########################################################################################

PREPROC_SCRIPT = $(vsrc_dir)/../../../../../../../scripts/insert-includes.py

$(PREPROC_VERILOG): $(ALL_VSRCS)
	rm -rf combined.v
	mkdir -p $(dir $(PREPROC_VERILOG))

	# Don't include testbenches in the verilog because I'm lazy and don't want to fix the msb/lsb thing in them and we won't use them anyways
	touch combined.v
	find $(ALL_VSRCS) | grep -iv test | xargs -I{} sh -c "cat {}; echo ''" >> combined.v

	# Unique-ify ALU module name which colides with rocketchip
	sed -i 's/ALU ALU/custom_ALU custom_ALU/g' combined.v
	sed -i 's/ALU(/custom_ALU(/g' combined.v

	# Make the riscv_top module name match the scala blackbox name
	# TODO there's probably a more robust way to do this too
	sed -i 's/module riscv_top/module OneFiftyOneCoreBlackBox/g' combined.v


	$(PREPROC_SCRIPT) combined.v $@ $(INC_DIRS)

	# Disable the cache for easier initial tilelink interface setup/debuggin
	# TODO this breaks sim, see README
	#printf "\`define no_cache_mem\n" | cat - $@ > temp && mv temp $@
	# TODO someone make these paths not hardcoded, these are the SRAMs that are used in the design
	# include SRAM verilog file at the top of the preprocessed file
	printf $(sram_include_str) | cat - $@ > temp && mv temp $@


	# FIXME FIXME FIXME getting into sketch hack territory here
	# this causes a width mismatch
	sed -i 's/`define CACHE_DEPTH_BITS_IMEM 7/`define CACHE_DEPTH_BITS_IMEM 6/g' $@

	rm -rf combined.v

clean:
	rm -rf $(vsrc_dir)/*.preprocessed.v
