# Tutorial of Building Bare Metal Debugging & Development Environment for Maix Bit (K210)
> Notice that the Environment here is for C/C++, Assembly, Bare metal, not for MaixPy.  
  
English | [简体中文](README-zh_CN.md)  

## Objective for the Tutorial  
This tutorial is designed to help you build a bare metal debugging and development environment for Sipeed Maix Bit (Kendryte 210). The tutorial will guide you step by step without basic knowledge of embedded system and try to reduce uncontrollable bugs.  
  
## Advices  
Please strictly follow every step of the tutorial since every step of the tutorial has a quite reasonable reason to avoid some crazy mistakes.  
Please ensure that you have basic knowledge of electronic circuit, Linux (operating system), GCC & GDB (C/C++) before starting the tutorial.  

## Hareware Preparation
* [Sipeed Maix Bit (NOT include display and camera)](https://www.seeedstudio.com/Sipeed-MAix-BiT-for-RISC-V-AI-IoT-p-2872.html)  
This board is designed for AI+IOT and has 16MiB flash on board. The 400MHz dual core chip Kendryte 210 on the board is 64-bit RISC-V Architecture.  
Its typical development scenario is the computer vision of IOT by using [MaixPy](https://maixpy.sipeed.com/en/) (Maybe you can do more fun things on it.) However, we mainly use it for bare metal development here.  
Advantage: Relatively reasonable price: $12.9 & Relatively complete documentation: [K210](https://canaan.io/developer), [Maix Bit](https://dl.sipeed.com/shareURL/MAIX) & [Relatively complete support](https://github.com/kendryte) & Relatively good cost–performance ratio ([even can run Linux or overclock](https://www.phoronix.com/scan.php?page=news_item&px=RISC-V-Changes-Linux-5.8)) 
   
* [Sipeed-USB-JTAG-TTL-RISC-V-Debugger (include DuPont Cable)](https://www.seeedstudio.com/Sipeed-USB-JTAG-TTL-RISC-V-Debugger-p-2910.html)  
This Debugger use [FT2232C](https://www.ftdichip.com/Support/Documents/DataSheets/ICs/DS_FT2232C.pdf) from FTDI Chip as its USB UART/FIFO IC.  
J-Link is about $400, J-Link EDU is about $60. Sipeed RV Debugger is only $7.6.  
Also, it can work with [PlatformIO and K210](https://docs.platformio.org/en/latest/boards/kendryte210/sipeed-maix-bit.html).  

### Connection
The pin layout of debugger:  

<img src=dpin.jpg></img>
  
The pin layout of Maix Bit:  
  
<img src=bpin.png></img>
  
Connection table:  

| Debugger | Maix Bit |
|:---:|:---:|
| GND | GND |
| `RXD` | `TX` |
| `TXD` | `RX` |
| 5V  | 5V |
| 3V3 | 3V3 |
| TDI | TDI |
| RST | RST |
| TMS | TMS |
| TDO | TDO |
| TCK | TCK |

> Notice that Debugger's `RX` pin should be connected to Board's `TXD` pin, and Debugger's `TX` pin should be connected to Board's `RXD` pin. This is important for `UART for Serial Communication`.  


## Development Environment
Host: Windows 10 Build 19042.685  
Virtual Machine: VMware Workstation 16 Player (VirtualBox has some tricky things on USB)  
System image: ubuntu-20.04.1-desktop-amd64.iso  (Use desktop version to use multiple terminals and VS Code easily)
  
> We did not choose WSL2 here because of [WSL #2195](https://github.com/microsoft/WSL/issues/2195). WSL2 is using Hyper-V, but Hyper-V does not support USB pass-through.  
    Alternative Solutions:  
    1. OpenOCD on Windows, ToolChain on WSL2  
    2. Use [usbip-win](https://github.com/cezanne/usbip-win), OpenOCD use IP access USB device  
    Just do yourself a favor, use VM :)

## Prepare Development Environment Steps

Recommend Step for creating a working directory (optional):  
`mkdir k210 && cd k210`  

### ToolChain Installation

1. Download [Kendryte ToolChain Release v8.2.0](https://github.com/kendryte/kendryte-gnu-toolchain/releases/tag/v8.2.0-20190213)
2. `tar -xvzf kendryte-toolchain-ubuntu-amd64-8.2.0-20190213.tar.gz`
3. `cd kendryte-toolchain-ubuntu-amd64-8.2.0-20190213/kendryte-toolchain`
4. `sudo cp -a ./. /opt/riscv-toolchain`, the ToolChain is copied to `/opt/riscv-toolchain`, notice that the folder name is `riscv-toolchain` instead of `kendryte-toolchain` !
6. `export k210toolchain=/opt/riscv-toolchain/bin`, `export PATH="$k210toolchain:$PATH"`, add ToolChain to path.  
Now, ToolChain Installation has finished.

About ToolChain: https://metalcode.eu/2019-11-16-gnu-toolchain.html

### SDK Installation

1. Download [Kendryte SDK Release v0.5.6](https://github.com/kendryte/kendryte-standalone-sdk/releases/tag/V0.5.6)
2. `tar -xvzf kendryte-standalone-sdk-0.5.6.tar.gz`
3. `cd kendryte-standalone-sdk-0.5.6`
4. `mkdir build && cd build`
5. `cmake .. -DPROJ=<ProjectName> -DTOOLCHAIN=/opt/riscv-toolchain/bin && make`  
Notice that  `<ProjectName>` needed to be replaced by real Project Name (`hello_world`).  
Now, SDK Installation has finished, and `hello_world` should be compiled successfully in `build`.  

> If `riscv64-unknown-elf-gcc: error trying to exec 'cc1': execvp: No such file or directory` occurs, recheck ToolChain Installation steps to make sure (1) you have `/opt/riscv-toolchain` & (2) you have added ToolChain location to `$PATH`.

### OpenOCD Debug
Notice that Maix Bit uses CH552 chip to implement USB-Serial without JTAG. (K210 supports JTAG, but the CH552 on Maix Bit has no JTAG function) (https://wiki.sipeed.com/soft/maixpy/en/get_started/install_driver/bit.html)  
CH552 emulates FT2232 chip, which is the same as the debugger chip (FT2232C) and causing conflict if they are used at the same time.  
  
> ONLY connect the debugger to your VM to avoid conflict!

1. Download [Kendryte OpenOCD Relaese v0.2.3](https://github.com/kendryte/openocd-kendryte/releases/tag/v0.2.3)
2. `tar -xvzf kendryte-openocd-0.2.3-ubuntu64.tar.gz`
3. `cd kendryte-openocd-0.2.3-ubuntu64/kendryte-openocd`
4. `cd bin`
5. `./openocd -f ../tcl/kendryte.cfg`
> If `./openocd: error while loading shared libraries: libusb-0.1.so.4: cannot open shared object file: No such file or directory` occurs, run `sudo apt-get install libusb-0.1` ;  
If `./openocd: error while loading shared libraries: libftdi.so.1: cannot open shared object file: No such file or directory` occurs, run `sudo apt-get install libftdi-dev` ;  
If `./openocd: error while loading shared libraries: libhidapi-hidraw.so.0: cannot open shared object file: No such file or directory`, run `sudo apt-get install libhidapi-hidraw0` .  
See https://github.com/ntfreak/openocd/blob/0dd3b7fa6c7930446967772832a351e90c426d69/README#L221
6. Then, you should see
    ```
     _  __              _            _     
    | |/ /___ _ __   __| |_ __ _   _| |_ ___  
    | ' // _ \ '_ \ / _` | '__| | | | __/ _ \ 
    | . \  __/ | | | (_| | |  | |_| | ||  __/ 
    |_|\_\___|_| |_|\__,_|_|   \__, |\__\___| 
                               |___/          
    Kendryte Open On-Chip Debugger For RISC-V v0.2.3 (2019-02-21)
    Licensed under GNU GPL v2
    adapter speed: 3000 kHz
    Error: No J-Link device found.
    ```
    If you are not using J-Link Debugger, you will see the error. It is okay, we just check the correctness of OpenOCD installation here. You can `Ctrl+C` to exit.    
7.  By https://steward-fu.github.io/website/mcu/k210/openocd.htm & https://metalcode.eu/2019-11-19-k210-debugging.html & https://mensi.ch/blog/articles/using-the-sipeed-jtag-debugger-with-a-blue-pill,    
    
    first, `cd ../tcl`
    * create file `ft2232c.cfg`
    ```py
    interface ftdi
    ftdi_vid_pid 0x0403 0x6010

    # [3:0] = [TMS 1, TDO 0, TDI 1, TCK 1]
    # [7:4] = GPOPL0-3
    # [11:8] = GPOPH0-3
    # 0xfff8 = 1111 1111 1111 1000
    # 0xfffb = 1111 1111 1111 1011
    ftdi_layout_init 0xfff8 0xfffb

    ftdi_layout_signal nTRST -data 0x0100 -oe 0x0100
    ftdi_layout_signal nSRST -data 0x0200 -oe 0x0200

    ```

    * create file `k210.cfg`
    ```py
    # debug adapter
    source [find ft2232c.cfg]

    transport select jtag
    adapter_khz 10000

    # server port
    gdb_port 3333
    telnet_port 4444

    # add cpu target
    set _CHIPNAME riscv

    jtag newtap $_CHIPNAME cpu -irlen 5 -expected-id 0x04e4796b

    set _TARGETNAME $_CHIPNAME.cpu
    target create $_TARGETNAME riscv -chain-position $_TARGETNAME

    # command
    init
    halt

    ```
8. `cd ../bin`  
    Again, confirm you are in `kendryte-openocd/bin`.  
    Just run `./openocd -f ../tcl/k210.cfg`.  (Debug `-d`)  
    If
    ```
    Error: libusb_open() failed with LIBUSB_ERROR_ACCESS
    Error: libusb_open() failed with LIBUSB_ERROR_ACCESS
    Error: no device found
    Error: unable to open ftdi device with vid 0403, pid 6010, description '*', serial '*' at bus location '*'
    ```
    occurs, run `sudo ./openocd -f ../tcl/k210.cfg`.  
        
    Now, OpenOCD will connect successfully and shows something like:
    ```
     _  __              _            _     
    | |/ /___ _ __   __| |_ __ _   _| |_ ___  
    | ' // _ \ '_ \ / _` | '__| | | | __/ _ \ 
    | . \  __/ | | | (_| | |  | |_| | ||  __/ 
    |_|\_\___|_| |_|\__,_|_|   \__, |\__\___| 
                               |___/          
    Kendryte Open On-Chip Debugger For RISC-V v0.2.3 (2019-02-21)
    Licensed under GNU GPL v2
    adapter speed: 10000 kHz
    Info : ftdi: if you experience problems at higher adapter clocks, try the command   "ftdi_tdo_sample_edge falling"
    Info : clock speed 10000 kHz
    Info : JTAG tap: riscv.cpu tap/device found: 0x04e4796b (mfg: 0x4b5 (<unknown>), part: 0x4e47,  ver: 0x0)
    Core [0] halted at 0x8000b434 due to debug interrupt
    Info : Examined RISCV core; found 2 harts
    Info : Listening on port 3333 for gdb connections
    Core [1] halted at 0x8000bf8e due to debug interrupt
    Core [0] halted at 0x8000b434 due to debug interrupt
    Info : Listening on port 6666 for tcl connections
    Info : Listening on port 4444 for telnet connections
    ```
    be sure there is `Core [0] ` and `Core [1] `. Also, notice that we will use `port 3333` in GDB for debugging later.  

### GDB from ToolChain for Debug
Start a new terminal here and keep the old OpenOCD terminal!  
  
1. `cd kendryte-toolchain/bin`
2. `./riscv64-unknown-elf-gdb ../../kendryte-standalone-sdk-0.5.6/build/hello_world`
3. In (gdb):  
    1. `set print pretty on` (optional)
    2. `target remote localhost:3333`
    3. `monitor reset halt`
    4. `load`  
    Debug: `tb (tbreak) main`  
    5. `c (continue)`  
    Debug: `s (step)` (step into) `n (next)` (step over)  

    (If `hello_world` is debugging, the program will be stuck in `scanf()` and will not print anything by `printf()`. To solve this issue, we need to use `UART for Serial Communication`)

4. Any time, `Ctrl+C` will stop current debugging. Then, `q (quit)` in (gdb) can quit gdb.  

About OpenOCD and GDB debug:  
https://stackoverflow.com/questions/38033130/how-to-use-the-gdb-gnu-debugger-and-openocd-for-microcontroller-debugging-fr

### UART for Serial Communication
Start a new terminal here and keep OpenOCD terminal and GDB terminal alive!  

Arguments are from [k210 UART Doc p118](https://s3.cn-north-1.amazonaws.com.cn/dl.kendryte.com/documents/kendryte_standalone_programming_guide_20190311144158_en.pdf):  
```
Arguments for UART:  
    Baud Rate(Bps): 115200 (9600 no working!)  
    Parity: None  
    Data bits: 8  
    Stop bit: 1
```
  
> If you are in Linux, you do NOT need a driver. If you are in Windows, download `CDM21228_Setup.zip` for Win10, download `FTDI_Win7.zip` for Win7 from https://dl.sipeed.com/MAIX/tools/ftdi_vcp_driver.  
  
If you are following `OpenOCD Debug` and only connecting debugger to your VM, just go to Step 1.
> If you are using Maix Bit's ft2232c chip for USB-UART (on your host):  
Notice that Maix BiT's CH552T chip only use ONE serial interface of K210 to implement USB-Serial.  
Thus, two serial interfaces will be shown in device management, but only one can be used for UART so you need to try both! (https://wiki.sipeed.com/soft/maixpy/en/get_started/upgrade_maixpy_firmware.html)  
  
1. `sudo apt-get install minicom`
2. `ls /dev/ttyUSB* -la`  
    remember the device name: (for example) `ttyUSB1`
    > When the debugger is just connected to VM, there may be two ttyUSBs (like `ttyUSB0` and `ttyUSB1`). If OpenOCD is running, there should be only one ttyUSB available.  
3. `sudo minicom -s -con` (Colorful screen `-con`)
4. select `Serial port setup`
5. press `A`, change device to the device in 1. , for example, `/dev/ttyUSB1`  
   press `F`, set to `No`  (Important!)  
   press `Enter`
6. select `Modem and dialing`  
    press the corresponding keys and set 9 strings `Init string, Reset string, Dialing prefix #1, Dialing suffix #1, Dialing prefix #2, Dialing suffix #2, Dialing prefix #3, Dialing suffix #3, Hang-up string` to nothing.  
    press `Enter`
7. select `Screen and keyboard`  
    press `B`, set to `DEL`  
    press `R`, set to `Yes`  
    press `Enter`  
8. select `Save setup as dfl` (Save the current setup as default)
9. select `Exit from Minicom`
10. `sudo minicom`  
    Now, you should see
    ```
    Welcome to minicom 2.7.1

    OPTIONS: I18n 
    Compiled on Dec 23 2019, 02:06:26.
    Port /dev/ttyUSB1, 08:31:02

    ``` 
    Port `/dev/ttyUSB1` is connected.  
    Notice that press `Ctrl+A` first, then `Z` for help or `any key` for other function, not press them simultaneously.
11. If you are debugging `hello_world` in gdb with OpenOCD, and type `6` in minicom.  
    Then, you should see  
    ```
    Welcome to minicom 2.7.1                                               
                                                                   
    OPTIONS: I18n                                                          
    Compiled on Dec 23 2019, 02:06:26.                                     
    Port /dev/ttyUSB1, 03:12:01                                            

    Press CTRL-A Z for help on special keys                                    

    Core 0 Hello world                                                         
                      Core 1 Hello world                                       
                                        6                                      
    Data is 6                                                              

    ```
    Now, UART should work well.  
    
MaixPy bin should be burned to Maix Bit by default. If Maix Bit's UART is connected correctly, press the `RESET` button on Maix Bit, and you should see something like  

<img src=maixpy.png></img>

About Minicom setup:  
https://maixpy.sipeed.com/en/get_started/serial_tools.html?h=115200  
https://wiki.emacinc.com/wiki/Getting_Started_With_Minicom

## About
This tutorial is written when I only have the experience of using development board but have no knowledge of embedded development. In fact, it is mainly for building an on-chip verification environment for [my compiler project](https://github.com/qingpeng9802/minijava-to-k210-riscv-compiler) about RISC-V assembly.  
  
Now (2020.12), there are not many resources of RISC-V development board, which means we need some exploration during development, especially for non-MaixPy development environment of Maix Bit. I hope this tutorial can help more people enjoy the fun of RISC-V bare metal development :)

## Acknowledgement
[@steward-fu (Steward Fu)](https://github.com/steward-fu):  
https://steward-fu.github.io/website/mcu/k210/openocd.htm  
[@metalcode-eu (Metalcode)](https://github.com/metalcode-eu):  
https://metalcode.eu/2019-11-19-k210-debugging.html  
[@mensi (Manuel Stocker)](https://github.com/mensi):  
https://mensi.ch/blog/articles/using-the-sipeed-jtag-debugger-with-a-blue-pill  

## License  
Copyright (C) 2020-2021 Qingpeng Li  
This work is licensed under a [Attribution-NonCommercial-NoDerivatives 4.0 International (CC BY-NC-ND 4.0) License](https://creativecommons.org/licenses/by-nc-nd/4.0/).  
