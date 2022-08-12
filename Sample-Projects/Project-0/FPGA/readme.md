### "Project 0" For Nitefury-II and Litefury

This project implements basic functionality:

- PCIe x4
- DMA to onboard DDR
- Identification registers

### Build Notes:

The first time the project is opened, there are some files missing that Vivado expects. 
In the project window, with the current directory set to the same directory as the .xpr file, run the following command in the tcl window:

```source ../../common/Scripts/rebuild.tcl```

After that, you can simply run

```source ../../common/Scripts/build.tcl```

when making changes.

The build creates an .mcs file in the mcs directory


It is normal to have 3 critical warnings during implementation. The hardware does not pin out the PCIe x4 in the way preferred by Xilinx XDMA block.
In particular, the pinout of the PCIe x4 interface is already selected by the autogenerated constraints file for the XDMA block.
See Top_xdma_0_0_pcie2_ip-PCIE_X0Y0.xdc where it generates:
```
# PCIe Lane 0
set_property LOC GTPE2_CHANNEL_X0Y7 [get_cells {inst/gt_top_i/pipe_wrapper_i/pipe_lane[0].gt_wrapper_i/gtp_channel.gtpe2_channel_i}]
# PCIe Lane 1
set_property LOC GTPE2_CHANNEL_X0Y6 [get_cells {inst/gt_top_i/pipe_wrapper_i/pipe_lane[1].gt_wrapper_i/gtp_channel.gtpe2_channel_i}]
# PCIe Lane 2
set_property LOC GTPE2_CHANNEL_X0Y5 [get_cells {inst/gt_top_i/pipe_wrapper_i/pipe_lane[2].gt_wrapper_i/gtp_channel.gtpe2_channel_i}]
# PCIe Lane 3
set_property LOC GTPE2_CHANNEL_X0Y4 [get_cells {inst/gt_top_i/pipe_wrapper_i/pipe_lane[3].gt_wrapper_i/gtp_channel.gtpe2_channel_i}]
```

The usual way to override autogenerated constraints is to simply put new constraints in a different file, and have have those constraints take effect later
by changing the PROCESSING_ORDER property of the file. See [https://www.xilinx.com/support/answers/52947.html].

However, the LOC property, which is the one we need to change, doesn't work in this fashion, since Vivado can't assign a LOC property to a cell
if the LOC is already in use. And they are all in use. Normally, the reset_property function can fix this, but reset_property doesn't work in constraint files.

This leaves two options:
- Write a tcl script that does synthesis, then does reset_property for the above 4 cells, then does implementation
OR
- Set the correct constraints early (before the IP constraints), and deal with the warnings when Vivado evaluates the (wrong) constraints in the autogenerated IP

This project uses the second option

### Programming the flash

You can add the flash part using the following command ```source ../../common/Scripts/add-flash-s25fl256s-0.tcl```

Then to program thje flash: ```source ../../common/Scripts/program-flash.tcl```


### Register Map

| Address  | Size (bytes) | Function  |
| ---      | ---          | ---       |
| 0        |  4           | "LITE"    |
| 4        |  4           | version   |
| 8-0xFFFF | 64K-8        | unused    |
| 0x10000  | see IP       | AXI-SPI   |

### Memory Map
| Address  | Size  | Function               |
| ---      | ---   | ---                    |
| 0        |  1GiB | Mapped to onboard DRAM |



