# Openocd for multi-WCHLinkE debugging

configuration to connect first enumerated WCH-LinkE device; file: [wch-riscv.cfg](bin/wch-riscv.cfg)
```tcl
#interface wlink
adapter driver wlinke
# use index 0, 1, 2... to select each wchlink
wlink_set_index 0
adapter speed 6000
transport select sdi

wlink_set_address 0x00000000
set _CHIPNAME wch_riscv
sdi newtap $_CHIPNAME cpu -irlen 5 -expected-id 0x00001

set _TARGETNAME $_CHIPNAME.cpu

target create $_TARGETNAME.0 wch_riscv -chain-position $_TARGETNAME
$_TARGETNAME.0 configure  -work-area-phys 0x20000000 -work-area-size 10000 -work-area-backup 1
set _FLASHNAME $_CHIPNAME.flash

flash bank $_FLASHNAME wch_riscv 0x00000000 0 0 0 $_TARGETNAME.0

lappend post_init_commands {echo "Ready for Remote Connections"}
```
configuration to connect second enumerated WCH-LinkE device; file: [wch-1-riscv.cfg](bin/wch-1-riscv.cfg)
```tcl
#interface wlink
adapter driver wlinke
# use index 0, 1, 2... to select each wchlink
wlink_set_index 1
adapter speed 6000
transport select sdi

wlink_set_address 0x00000000
set _CHIPNAME wch_riscv
sdi newtap $_CHIPNAME cpu -irlen 5 -expected-id 0x00001

set _TARGETNAME $_CHIPNAME.cpu

target create $_TARGETNAME.0 wch_riscv -chain-position $_TARGETNAME
$_TARGETNAME.0 configure  -work-area-phys 0x20000000 -work-area-size 10000 -work-area-backup 1
set _FLASHNAME $_CHIPNAME.flash

flash bank $_FLASHNAME wch_riscv 0x00000000 0 0 0 $_TARGETNAME.0

lappend post_init_commands {
    echo "Ready for Remote Connections"
}
```

## openocd configuration setup in Eclipse or similar IDE

*Note: Make sure for each openocd instance use differenct port numbers for telnet/tcl/gdb*
![image](https://github.com/123swk123/openocd-wch-linke-multilink/assets/903389/0c139b1d-18e1-4c95-84e1-1552219947e8)

## To manually read Flash info

*for example on ch32v003, run this from openocd bin dir*

`openocd.exe -f .\wch-riscv.cfg -c 'wch_riscv.cpu.0 configure -event reset-end {flash probe 0}' -c init -c reset -c exit`
```log
Open On-Chip Debugger 0.11.0+dev-snapshot (2023-04-23-01:01)
Licensed under GNU GPL v2
For bug reports, read
        http://openocd.org/doc/doxygen/bugs.html
Info : only one transport option; autoselect 'sdi'
Warn : Transport "sdi" was already selected
{echo "Ready for Remote Connections"}
Info : WCH-LinkE  mode:RV version 2.9
Info : wlink_init ok
Info : clock speed 6000 kHz
Info : [wch_riscv.cpu.0] datacount=2 progbufsize=8
Info : [wch_riscv.cpu.0] Examined RISC-V core; found 1 harts
Info : [wch_riscv.cpu.0]  XLEN=32, misa=0x40800014
[wch_riscv.cpu.0] Target successfully examined.
Info : starting gdb server for wch_riscv.cpu.0 on 3333
Info : Listening on port 3333 for gdb connections
Info : device id = 0x83a7abcd
Info : flash size = 16kbytes
```
