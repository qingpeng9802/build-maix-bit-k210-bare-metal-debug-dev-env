# Maix Bit (K210) 的裸机调试与开发环境的搭建教程
> 注意此环境是指C/C++，汇编，裸机开发的环境，不是MaixPy的环境。
  
[English](README.md) | 简体中文  

## 教程目标  
这个教程旨在帮助你搭建Sipeed Maix Bit (Kendryte 210)的裸机调试和开发环境。本教程会尽可能在不需要嵌入式基础知识的情况下做到逐步指导并尽量减少出现不可控的bug。
  
## 建议  
请严格遵守教程的每个步骤，因为每一步都有相当合理的理由来避免神秘的错误。  
在你开始教程之前，请确保你有关于电子电路，Linux（操作系统），GCC和GDB（C/C++）的基础知识。  
  
## 硬件准备
> 注：SeeedStudio官网不向中国发货，请到淘宝或1688搜索相应开发板，主要注意开发板是否包含显示器和摄像头，调试器是否包含杜邦线。  
  
* [Sipeed 矽速科技 Maix Bit (不包含显示屏和摄像头)](https://www.seeedstudio.com/Sipeed-MAix-BiT-for-RISC-V-AI-IoT-p-2872.html)  
这块开发板是为了AI+IOT而设计，有16MiB的板上flash。板上的400MHz的双核芯片是RISC-V 64位架构。  
它的典型开发环境是使用[MaixPy](https://maixpy.sipeed.com/en/)开发的IOT的计算机视觉。（也许可以在上面开发更多有趣的东西。）但我们这里主要用它来进行裸机开发。  
优势：相对合理的价格：￥84 & 相对完善的文档：[K210](https://canaan.io/developer), [Maix Bit](https://dl.sipeed.com/shareURL/MAIX) & [相对完善的支持](https://github.com/kendryte) & 相对好的性价比 ([甚至可以运行Linux或超频](https://www.phoronix.com/scan.php?page=news_item&px=RISC-V-Changes-Linux-5.8)) 
  
* [Sipeed-USB-JTAG-TTL-RISC-V-Debugger (包含杜邦线)](https://www.seeedstudio.com/Sipeed-USB-JTAG-TTL-RISC-V-Debugger-p-2910.html)  
这个调试器使用来自 FTDI Chip的 [FT2232C](https://www.ftdichip.com/Support/Documents/DataSheets/ICs/DS_FT2232C.pdf) 作为USB UART/FIFO IC。  
J-Link 需要约 $400, J-Link EDU 需要约 $60. Sipeed RV 调试器只需要 ￥65.  
另外它和可以和 [PlatformIO and K210](https://docs.platformio.org/en/latest/boards/kendryte210/sipeed-maix-bit.html) 一起工作。

### 连接
调试器的针脚布局:  

<img src=dpin.jpg></img>
  
Maix Bit的针脚布局:  
  
<img src=bpin.png></img>
  
连接表:  

| 调试器 | Maix Bit |
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

> 注意调试器的`RX`针脚应该连接板子的`TXD`针脚，且调试器的`TX`针脚应该连接板子的`RXD`针脚。这对`UART 串行通信`很重要。  

## 开发环境
宿主机: Windows 10 Build 19042.685  
虚拟机: VMware Workstation 16 Player (VirtualBox 的USB支持有点奇怪)  
系统镜像: ubuntu-20.04.1-desktop-amd64.iso  (使用桌面版能更方便的使用多个终端和VS Code)
  
> 我们未使用WSL是因为 [WSL #2195](https://github.com/microsoft/WSL/issues/2195)。 WSL2正在使用Hyper-V，但Hyper-V不支持USB直通。  
    替代方案：  
    1. OpenOCD on Windows, 工具链 on WSL2  
    2. 用 [usbip-win](https://github.com/cezanne/usbip-win), OpenOCD 使用ip来访问USB设备  
    我建议使用虚拟机来获得一致的开发体验。  

## 开发环境准备步骤

建议步骤：创建工作目录（可选）：  
`mkdir k210 && cd k210`  

### 工具链安装

1. 下载 [Kendryte ToolChain Release v8.2.0](https://github.com/kendryte/kendryte-gnu-toolchain/releases/tag/v8.2.0-20190213)
2. `tar -xvzf kendryte-toolchain-ubuntu-amd64-8.2.0-20190213.tar.gz`
3. `cd kendryte-toolchain-ubuntu-amd64-8.2.0-20190213/kendryte-toolchain`
4. `sudo cp -a ./. /opt/riscv-toolchain`, 工具链复制到 `/opt/riscv-toolchain`, 注意文件夹名字是 `riscv-toolchain` 而不是 `kendryte-toolchain` !
6. `export k210toolchain=/opt/riscv-toolchain/bin`, `export PATH="$k210toolchain:$PATH"`, 来添加工具链到路径.  
现在工具链安装已经完成。  

关于工具链: https://metalcode.eu/2019-11-16-gnu-toolchain.html

### SDK 安装

1. 下载 [Kendryte SDK Release v0.5.6](https://github.com/kendryte/kendryte-standalone-sdk/releases/tag/V0.5.6)
2. `tar -xvzf kendryte-standalone-sdk-0.5.6.tar.gz`
3. `cd kendryte-standalone-sdk-0.5.6`
4. `mkdir build && cd build`
5. `cmake .. -DPROJ=<ProjectName> -DTOOLCHAIN=/opt/riscv-toolchain/bin && make`  
注意  `<ProjectName>` 需要用真实项目名来替换 (`hello_world`).  
现在SDK安装已经完成, 且 `hello_world` 应该在 `build` 中成功编译。  

> 如果出现 `riscv64-unknown-elf-gcc: error trying to exec 'cc1': execvp: No such file or directory`， 检查工具链安装步骤，确保 (1) 你有 `/opt/riscv-toolchain` & (2) 你已经将工具链添加到 `$PATH`.

### OpenOCD 调试
注意Maix Bit使用CH552芯片来实现USB-串行功能，但没有JTAG功能。（K210支持JTAG，但Maix Bit上的CH552没有JTAG功能）(https://wiki.sipeed.com/soft/maixpy/zh/get_started/install_driver/bit.html)  
CH552将模拟FT2232芯片，而调试器使用FT2232C，它们同时使用会导致冲突。  
  
> 只 将调试器连接到虚拟机来避免冲突！

1. 下载 [Kendryte OpenOCD Relaese v0.2.3](https://github.com/kendryte/openocd-kendryte/releases/tag/v0.2.3)
2. `tar -xvzf kendryte-openocd-0.2.3-ubuntu64.tar.gz`
3. `cd kendryte-openocd-0.2.3-ubuntu64/kendryte-openocd`
4. `cd bin`
5. `./openocd -f ../tcl/kendryte.cfg`
> 如果出现 `./openocd: error while loading shared libraries: libusb-0.1.so.4: cannot open shared object file: No such file or directory` , 运行 `sudo apt-get install libusb-0.1` ;  
如果出现 `./openocd: error while loading shared libraries: libftdi.so.1: cannot open shared object file: No such file or directory` , 运行 `sudo apt-get install libftdi-dev` ;  
如果出现 `./openocd: error while loading shared libraries: libhidapi-hidraw.so.0: cannot open shared object file: No such file or directory`, 运行 `sudo apt-get install libhidapi-hidraw0` .  
见 https://github.com/ntfreak/openocd/blob/0dd3b7fa6c7930446967772832a351e90c426d69/README#L221
6. 然后，你应该看到
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
    如果你没有在使用J-Link调试器，你会看到这个错误。可以忽略，我们只在这里检查OpenOCD安装的正确性。你可以按`Ctrl+C`来退出。  
7.  根据 https://steward-fu.github.io/website/mcu/k210/openocd.htm & https://metalcode.eu/2019-11-19-k210-debugging.html & https://mensi.ch/blog/articles/using-the-sipeed-jtag-debugger-with-a-blue-pill,    
    
    首先， `cd ../tcl`
    * 创建文件 `ft2232c.cfg`
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

    * 创建文件 `k210.cfg`
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
    再次确认你在 `kendryte-openocd/bin`.  
    运行 `./openocd -f ../tcl/k210.cfg`.  (Debug `-d`)  
    如果出现
    ```
    Error: libusb_open() failed with LIBUSB_ERROR_ACCESS
    Error: libusb_open() failed with LIBUSB_ERROR_ACCESS
    Error: no device found
    Error: unable to open ftdi device with vid 0403, pid 6010, description '*', serial '*' at bus location '*'
    ```
    运行 `sudo ./openocd -f ../tcl/k210.cfg`.  
        
    现在OpenOCD应该成功连接，且显示类似这样：  
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
    确保有 `Core [0] ` and `Core [1] `. 而且, 注意我们将会在GDB中使用 `port 3333` 来调试.  

### 工具链的GDB调试
开启一个新的终端，且保持旧的OpenOCD终端打开！  
  
1. `cd kendryte-toolchain/bin`
2. `./riscv64-unknown-elf-gdb ../../kendryte-standalone-sdk-0.5.6/build/hello_world`
3. 在 (gdb):  
    1. `set print pretty on` (可选)
    2. `target remote localhost:3333`
    3. `monitor reset halt`
    4. `load`  
    Debug: `tb (tbreak) main`  
    5. `c (continue)`  
    Debug: `s (step)` (step into) `n (next)` (step over)  

    (如果 `hello_world` 正在被调试, 程序将会卡在 `scanf()` 且 `printf()` 不会打印任何东西。为了解决这个问题，我们需要用 `UART 串行通信`)

4. 任何时候， `Ctrl+C` 将会停止当前调试。 然后，(gdb)中 `q (quit)` 可以退出gdb。  

关于 OpenOCD 和 GDB 调试:  
https://stackoverflow.com/questions/38033130/how-to-use-the-gdb-gnu-debugger-and-openocd-for-microcontroller-debugging-fr

### UART 串行通信
开启一个新的终端，且保持旧的OpenOCD终端和GDB终端打开！  

来自 [k210 UART Doc p118](https://s3.cn-north-1.amazonaws.com.cn/dl.kendryte.com/documents/kendryte_standalone_programming_guide_20190311144158_en.pdf) 的参数：  
```
UART 参数:  
    Baud Rate(Bps) 波特率: 115200 (9600 无效！)  
    Parity 奇偶校验位: None  
    Data bits 数据位: 8  
    Stop bit 停止位: 1
```
  
> 如果你在 Linux, 你不需要安装驱动。 如果你在 Windows，从 https://dl.sipeed.com/MAIX/tools/ftdi_vcp_driver)，Win10下载 `CDM21228_Setup.zip`, Win7下载 `FTDI_Win7.zip`。  
  
如果你正在跟随教程的 `OpenOCD 调试` 且只连接了调试器到虚拟机，跳到 Step 1。
> 如果你正在使用Maix Bit的ft2232c芯片（在你的宿主机上）来做USB-UART：  
注意Maix BiT的CH552T芯片只使用了一个K210的串口来实现USB-串口。  
因此，两个串口都会出现在设备管理中，但只有一个可以用做UART，所以你需要两个都试试！(https://wiki.sipeed.com/soft/maixpy/zh/get_started/upgrade_maixpy_firmware.html)  
  
1. `sudo apt-get install minicom`
2. `ls /dev/ttyUSB* -la`  
    记住设备名: 例如 `ttyUSB1`  
    > 当调试器被连接到虚拟机，可能会有两个ttyUSB设备（如 `ttyUSB0` 和 `ttyUSB1`）。如果OpenOCD正在运行，应该只有一个ttyUSB可用。  
3. `sudo minicom -s -con` (彩色屏幕 `-con`)
4. 选择 `Serial port setup`
5. 按 `A` 修改设备到 1. 中的设备，例如 `/dev/ttyUSB1`  
   按 `F` 选 `No`  (重要！)  
   按 `Enter`
6. 选择 `Modem and dialing`  
    按对应的键，设置9个字符串成空字符串 `Init string, Reset string, Dialing prefix #1, Dialing suffix #1, Dialing prefix #2, Dialing suffix #2, Dialing prefix #3, Dialing suffix #3, Hang-up string`    
    按 `Enter`
7. 选择 `Screen and keyboard`  
    按 `B` 选 `DEL`  
    按 `R` 选 `Yes`  
    按 `Enter`  
8. 选择 `Save setup as dfl` (保存当前设置到默认)
9. 选择 `Exit from Minicom`
10. `sudo minicom`  
    现在你应该看到  
    ```
    Welcome to minicom 2.7.1

    OPTIONS: I18n 
    Compiled on Dec 23 2019, 02:06:26.
    Port /dev/ttyUSB1, 08:31:02

    ``` 
    设备 `/dev/ttyUSB1` 已连接。  
    注意先按下 `Ctrl+A` ，再按下 `Z` 来查看help或者 `any key` 来执行，不要同时按下它们。  
11. 如果你正在gdb中和OpenOCD一起调试 `hello_world` ，且再minicom中按下 `6`。  
    然后，你应该看到  
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
    现在UART应该工作良好。  
    
MaixPy的bin文件应该被默认烧录到Maix Bit。如果Maix Bit的UART被成功连接，按下Maix Bit上的`RESET`键，你应该看到类似这样  
  
<img src=maixpy.png></img>

关于 Minicom 设置：  
https://maixpy.sipeed.com/en/get_started/serial_tools.html?h=115200  
https://wiki.emacinc.com/wiki/Getting_Started_With_Minicom

## 关于
这个教程是我在仅有开发板使用经验但没有嵌入式开发知识的情况下写成的，实际上，主要是为了我的一个关于RISC-V汇编的[编译器项目](https://github.com/qingpeng9802/minijava-to-k210-riscv-compiler)的片上验证来搭建环境。  
  
目前 (2020.12) RISC-V开发板的资源相对缺乏，开发中需要一定的探索，特别是对于Maix Bit的非MaixPy开发环境。希望本教程能够帮助更多的人享受到riscv裸机开发的乐趣 :)

## 致谢
[@steward-fu (Steward Fu)](https://github.com/steward-fu):  
https://steward-fu.github.io/website/mcu/k210/openocd.htm  
[@metalcode-eu (Metalcode)](https://github.com/metalcode-eu):  
https://metalcode.eu/2019-11-19-k210-debugging.html  
[@mensi (Manuel Stocker)](https://github.com/mensi):  
https://mensi.ch/blog/articles/using-the-sipeed-jtag-debugger-with-a-blue-pill  

## 许可证  
版权所有 (C) 2020-2021 Qingpeng Li  
本作品采用 [署名-非商业性使用-禁止演绎 4.0 国际 (CC BY-NC-ND 4.0) 许可证](https://creativecommons.org/licenses/by-nc-nd/4.0/) 进行许可。  
