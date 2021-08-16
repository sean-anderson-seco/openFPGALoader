# Xilinx notes

To simplify further explanations, we consider the project is generated in the
current directory.

**Note:**
1. Spartan Edge Accelerator Board has only pinheader, so the cable must be provided
2. a *JTAG* <-> *SPI* bridge (used to write bitstream in FLASH) is available for some device, see
[spiOverJtag](https://github.com/trabucayre/openFPGALoader/tree/master/spiOverJtag) to check if your model is supported
3. board provides the device/package model, but if the targeted board is not
   officially supported but the FPGA yes, you can use --fpga-part to provides
   model

<span style="color:red">**Warning** *.bin* may be loaded in memory or in flash, but this extension is a classic extension
for CPU firmware and, by default, *openFPGALoader* load file in memory, double check
*-m* / *-f* when you want to use a firmware for a softcore
(or anything, other than a bitstream) to write somewhere in the FLASH device).</span>

*.bit* file is the default format generated by *vivado*, so nothing special
task must be done to generates this bitstream.

*.bin* is not, by default, produces. To have access to this file you need to configure the tool:
- **GUI**: *Tools* -> *Settings* -> *Bitstreams* -> check *-bin_file*
- **TCL**: append your *TCL* file with `set_property STEPS.WRITE_BITSTREAM.ARGS.BIN_FILE true [get_runs impl_1]`

<span style="color:red">**Warning: for alchitry board the bitstream must be configured with a buswidth of 1 or 2. Quad mode can't be used with alchitry's FLASH**</span>

## Loading a bitstream

<span style="text-decoration:underline">*.bit* and *.bin* are allowed to be loaded in memory.</span>

__file load:__
```bash
openFPGALoader [-m] -b arty *.runs/impl_1/*.bit (or *.bin)
```
or
```bash
openFPGALoader [-m] -b spartanEdgeAccelBoard -c digilent_hs2 *.runs/impl_1/*.bit (or *.bin)
```

### SPI flash:

<span style="text-decoration:underline">*.bit*, *.bin*, and *.mcs* are supported for FLASH.</span>

.mcs must be generates through vivado with a tcl script like
```tcl
set project [lindex $argv 0]

set bitfile "${project}.runs/impl_1/${project}.bit"
set mcsfile "${project}.runs/impl_1/${project}.mcs"

write_cfgmem -format mcs -interface spix4 -size 16 \
    -loadbit "up 0x0 $bitfile" -loaddata "" \
    -file $mcsfile -force

```
**Note:
*-interface spix4* and *-size 16* depends on SPI flash capability and size.**

The tcl script is used with:
```bash
vivado -nolog -nojournal -mode batch -source script.tcl -tclargs myproject
```

__file load:__
```bash
openFPGALoader [--fpga-part xxxx] -f -b arty *.runs/impl_1/*.mcs (or .bit / .bin)
```
**Note: *-f* is required to write bitstream (without them *.bit* and *.bin* are loaded in memory)**

Note: "--fpga-part" is only required if this information is not provided at
board.hpp level or if the board is not officially supported. device/packagee
format is something like xc7a35tcsg324 (arty model). See src/board.hpp, or
spiOverJtag directory for examples.