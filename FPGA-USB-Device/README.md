![语言](https://img.shields.io/badge/语言-verilog_(IEEE1364_2001)-9A90FD.svg) ![部署](https://img.shields.io/badge/部署-quartus-blue.svg) ![部署](https://img.shields.io/badge/部署-vivado-FF1010.svg)

[English](#en) | [中文](#cn)

　


<span id="en">FPGA USB-device</span>
===========================

## What and Why?

The usual solution to realize custom USB devices on FPGA is to use USB chips (such as CY7C68013), which leads to high circuit cost. This repo is an FPGA-based USB Full-Speed device core, which only require a simple circuit (just like STM32 microcontrollers) instead of additional USB chips.

Based on this, I further implement USB audio, USB camera, USB disk, USB keyboard and USB-Serial devices on FPGA. They are all standard devices specified by USB, which can be plug and play without installing drivers.

## Features

- Pure Verilog implementation for universal FPGAs such as Xilinx, Altera, etc.
- The circuit is simple, **only need three FPGA pins, one resistor and one USB connector** (see [Circuit connection](#circuit_en)).

If you're not familiar with USB protocol stack, but want to quickly implement USB devices on FPGA, you can use some USB classes I provide:

- **USB audio** is a USB Audio Class (UAC) device. It can let FPGA be a speaker and a microphone (they are both in stereo 2-channel, 48ksps). It provides streaming interfaces for FPGA developers to receive speaker data and send microphone data.
- **USB camera** is a USB Video Class (UVC) device. It can let FPGA be a USB camera, which provides a streaming interface for sending video data to PC.
- **USB flash drive** is a USB Mass Storage (MSC) device. It can let FPGA be a USB flash disk.
- **USB keyboard** is a USB Human Interface (HID) device. It can let FPGA be a USB keyboard, which provides a interface for FPGA developers to send "key press" action.
- **USB-Serial** is a USB Communication Device Class (USB-CDC) device. It can let FPGA be a USB serial port device, which provides interfaces for FPGA developers to receive and send data to PC.
- **USB-Serial-2channel**: is a composite device that includes two USB-CDC devices. It can let FPGA a two-channel serial port device, which provides interfaces for FPGA developers to receive and send data to PC.

If you are familiar with USB protocol stack, you can use the usb device core in this repo to develop more USB devices (see [Development using USB core](#usbcore_en)). The core provides:

- Customize 1 device descriptor, 1 configuration descriptor, and 6 string descriptors.
- 4 IN endpoints (0x81\~0x84), 4 OUT endpoints (0x01\~0x04).
- An optional debug interface for printing debug information to the computer via a UART, you can see the USB communication process through it.

 　

## Demonstration of USB devices

|                    | What you can see in Windows Device Manager |What you can see in softwares|
| :----------------: | :----------------------------: | :----------------------------: |
|    **USB audio**    |![](./figures/ls_audio.png)| ![](./figures/test_audio.png)  |
|   **USB camera**   |![](./figures/ls_camera.png)| ![](./figures/test_camera.png) |
|      **USB disk**      |![](./figures/ls_disk.png)|  ![](./figures/test_disk.png)  |
|    **USB keyboard**    |![](./figures/ls_keyboard.png)|    Press a key every 2 seconds    |
|   **USB Serial**   |![](./figures/ls_serial.png)| ![](./figures/test_serial.png) |
| **USB Serial 2channel** |![](./figures/ls_serial2.png)|              ditto              |

 　

## Compatibility for Operation Systems

I tested the compatibility of these devices on different operating systems, as shown below.

|          |     Windows 10     |Linux Ubuntu 18.04|    macOS 10.15     |
| :----------------: | :----------------: | :----------------------------------: | :----------------: |
|    **USB audio**    | :heavy_check_mark: |:heavy_check_mark:| :heavy_check_mark: |
|   **USB camera**   | :heavy_check_mark: |:heavy_check_mark:          |        :x: *        |
|      **USB disk**      | :heavy_check_mark: |:heavy_check_mark:          | :heavy_check_mark: |
|    **USB keyboard**    | :heavy_check_mark: |:heavy_check_mark:          | :heavy_check_mark: |
|   **USB Serial**   | :heavy_check_mark: |:heavy_check_mark:          | :heavy_check_mark: |
| **USB Serial 2channel** | :heavy_check_mark: |:heavy_check_mark:| :heavy_check_mark: |

> :x:  macOS recognizes my USB camera device, but it can't read the video. The reason is unknown and to be resolved later.

 　

### Special thanks

- Thanks [xiaowuzxc](https://github.com/xiaowuzxc) for submitting the code of a USB speaker device. I later expanded it to current USB audio device (speaker+microphone).
- Thanks *\*\*\*8125@qq.com for solving the problem of Linux cannot recongnize one of the channels of USB-Serial-2channel.

 　

# <span id="circuit_en">Circuit connection</span>

USB has 4 wires: `VBUS`, `GND`, `USB_D-`, and `USB_D+` . Taking USB Type B connector as an example, the 4 wires are defined in the figure below.

|![USBTypeB](./figures/usb_typeb.png)|
| :-----------------------------------: |
|**Figure**: USB connector (square female) and cable.|

Please connect the circuit according to the figure below. Where `usb_dp_pull`, `usb_dp`, `usb_dn` are the three common IO pins of FPGA (must be 3.3 V). Note that:

- If you use flying leads for connection, ensure that the length of flying leads not longer than 10 cm.
- If you use PCB for connection, note that `USB_D-` and `USB_D+` are differential pairs.
- The `usb_dp_pull` should connect to `USB_D+` through a 1.5kΩ resistor.
- Don't forget to connect `GND`
- The `VBUS` pin of USB connector `VBUS` is a 5V power source that comes from USB-host, it can be ignored, or can supply power for FPGA.

```
  _________________
  |               |
  |  usb_dp_pull* |-------|
  |               |       |
  |               |      |-| 1.5k resistor
  |               |      | |
  |               |      |_|            ____________                             __________
  |               |       |             |          |                             |
  |       usb_dp* |-------^-------------| USB_D+   |                             |
  |               |                     |          |    USB cable                |
  |       usb_dn* |---------------------| USB_D-   |<===========================>| Host PC
  |               |                     |          |                             |
  |           GND |---------------------| GND      |                             |
  |               |                     |          |                             |
  -----------------                     ------------                             ----------
        FPGA              USB connector (Type-B, mini, micro, or Type-C)           Host PC
 *: usb_dp_pull, usb_dp, usb_dn are common 3V3 IO pins of FPGA.

                        Figure : Connect FPGA to USB
```

 　

# II List of Design code

[RTL](./RTL) The folder contains all the design source code, which is divided into 3 folders according to the level:

|                  Folder                  |      Level      |Description|
| :--------------------------------------: | :-------------: | :----------------------------------------------------------- |
| [RTL/fpga_examples](./RTL/fpga_examples) |     Application     |Realize specific applications based on USB class. To facilitate testing, the applications here are very simple. You can develop complex applications such as capturing data from a CMOS sensor and sending to the USB UVC core.|
|     [RTL/usb_class](./RTL/usb_class)     |    USB class    |Realize some USB class cores based on the USB device core.|
|    [RTL/usbfs_core](./RTL/usbfs_core)    | USB device core |A universal USB device core, which implements USB protocol stack from signal-level to transaction-level. Developers that know well about the USB protocol can use it to develop more USB devices. See [Development using USB core](#usbcore_en).|

 　

Specifically, each source code file is listed as follows:

|                  Folder                  |          File Name          |Description|
| :--------------------------------------: | :----------------------: | :----------------------------------------------------------- |
| [RTL/fpga_examples](./RTL/fpga_examples) |  fpga_top_usb_audio.v   |Simple demo of using usb_audio_top.v. Let FPGA be a USB speaker & microphone with loopback connection, so that the speaker's voice will recorded by the microphone.|
| [RTL/fpga_examples](./RTL/fpga_examples) |  fpga_top_usb_camera.v  |Simple demo of using usb_camera_top.v. Let FPGA be a USB camera that generates a black and white stripe video.|
| [RTL/fpga_examples](./RTL/fpga_examples) |   fpga_top_usb_disk.v   |Simple demo of using usb_disk_top.v. Let FPGA be a USB flash disk with a 24 KB FAT16 file system.|
| [RTL/fpga_examples](./RTL/fpga_examples) | fpga_top_usb_keyboard.v |Simple demo of using usb_keyboard_top.v. Let FPGA be a USB keyborad that pressing a key every 2 seconds.|
| [RTL/fpga_examples](./RTL/fpga_examples) |  fpga_top_usb_serial.v  |Simple demo of using usb_serial_top.v. Let FPGA be a USB serial port device.|
| [RTL/fpga_examples](./RTL/fpga_examples) | fpga_top_usb_serial2.v  |Simple demo of using usb_serial2_top.v. Let FPGA be a two-channel USB serial port device.|
|     [RTL/usb_class](./RTL/usb_class)     |     usb_audio_top.v     |use **usbfs_core_top.v** to implement USB speaker & microphone|
|     [RTL/usb_class](./RTL/usb_class)     |    usb_camera_top.v     |use **usbfs_core_top.v** to implement USB Video Class (UVC) camera|
|     [RTL/usb_class](./RTL/usb_class)     |     usb_disk_top.v      |use **usbfs_core_top.v** to implement USB Mass Storage Class (MSC) disk|
|     [RTL/usb_class](./RTL/usb_class)     |   usb_keyboard_top.v    |use **usbfs_core_top.v** to implement USB Human Interface Device Class (HID) keyboard|
|     [RTL/usb_class](./RTL/usb_class)     |    usb_serial_top.v     |use **usbfs_core_top.v** to implement USB Communication Device Class (CDC) serial port|
|     [RTL/usb_class](./RTL/usb_class)     |    usb_serial2_top.v    |use **usbfs_core_top.v** to implement a composite device, including 2 USB-CDC serial ports|
|    [RTL/usbfs_core](./RTL/usbfs_core)    |  **usbfs_core_top.v**   |Top module of the USB device core|
|    [RTL/usbfs_core](./RTL/usbfs_core)    |   usbfs_transaction.v   |USB transaction-level controller, called by usbfs_core_top.v.|
|    [RTL/usbfs_core](./RTL/usbfs_core)    |    usbfs_packet_rx.v    |USB packet-level RX controller, called by usbfs_core_top.v.|
|    [RTL/usbfs_core](./RTL/usbfs_core)    |    usbfs_packet_tx.v    |USB packet-level TX controller, called by usbfs_core_top.v.|
|    [RTL/usbfs_core](./RTL/usbfs_core)    |    usbfs_bitlevel.v     |USB bit-level controller (PHY) , called by usbfs_core_top.v.|
|    [RTL/usbfs_core](./RTL/usbfs_core)    |  usbfs_debug_monitor.v  |Collects and send out debug information. only for debug|
|    [RTL/usbfs_core](./RTL/usbfs_core)    |        usbfs_debug_uart_tx.v        |Convert the debug information to a UART signal. only for debug|

The following sections describe how to use these USB classes.

 　

# Ⅲ USB audio

The USB audio device implements speaker + microphone. The file hierarchy is as follows. Please add these files to the FPGA project, compile and program it to the FPGA.

- RTL/fpga_examples/**fpga_top_usb_audio.v**
  - RTL/usb_class/**usb_audio_top.v**
    - RTL/usbfs_core/**usbfs_core_top.v**
      - Other .v files in RTL/usbfs_core/

> :warning: The USB core of this repo requires a 60 MHz driving clock. Since I use Altera Cyclone IV, an `altpll` primitive module is used in my code to convert inputted 50MHz clock to 60MHz clock. If your FPGA is not a Altera Cyclone IV, you should remove `altpll` module and use the corresponding primitive or IP core to generate 60 MHz clock. For example, for Xilinx FPGAs, you can use Clock Wizard IP.

### Test

After the USB is plugged in, open the Windows Device Manager, you would find a device:

![](./figures/ls_audio.png)

Because **fpga_top_usb_audio.v** loopback connects the speaker and the microphone, the played voice of the speaker will be recorded by the microphone.

For testing, first select the device as the audio output device:

![](./figures/select_audio.png)

Then play a music. Meanwhile use any recording software or video recording software (such as obstudio), select FPGA-USB-audio as the input microphone, and record an audio. Finally you will find that the recorded audio is the same as the music you played.

### Application development

You can develop more complex audio applications based on this simple example, for which you need to pay attention to the in/out ports of **usb_audio_top.v** . See the code comments for details.

 　

# IV USB camera

The file hierarchy is as follows. Please add these files to the FPGA project, compile and program it to the FPGA.

- RTL/fpga_examples/**fpga_top_usb_camera.v**
  - RTL/usb_class/**usb_camera_top.v**
    - RTL/usbfs_core/**usbfs_core_top.v**
      - Other .v files in RTL/usbfs_core/

> :warning: The USB core of this repo requires a 60 MHz driving clock. Since I use Altera Cyclone IV, an `altpll` primitive module is used in my code to convert inputted 50MHz clock to 60MHz clock. If your FPGA is not a Altera Cyclone IV, you should remove `altpll` module and use the corresponding primitive or IP core to generate 60 MHz clock. For example, for Xilinx FPGAs, you can use Clock Wizard IP.

### Test

After the USB is plugged in, open the Windows Device Manager, you would find a device:

![](./figures/ls_camera.png)

Open the "camera" software of Windows, you would see a scrolling black and white stripe:

![](./figures/test_camera.png)

### Application development

You can develop more camera applications based on this simple example, for which you need to pay attention to the in/out ports of **usb_camera_top.v** 

#### Parameters of usb_camera_top.v

The key parameters are listed here:


```verilog
// parameters of usb_camera_top.v
parameter        FRAME_TYPE = "YUY2",    // "MONO" or "YUY2"
parameter [13:0] FRAME_W    = 14'd320,   // video-frame width  in pixels, must be a even number
parameter [13:0] FRAME_H    = 14'd240,   // video-frame height in pixels, must be a even number
```

You can set the width and height of the video frame by modifying parameter `FRAME_W` and `FRAME_H`.

As for paramter `FRAME_TYPE` , it can be `"MONO"` or `"YUY2"` . 

##### FRAME_TYPE="MONO"

 `FRAME_TYPE="MONO"` Is grayscale mode. Each pixel is a 1-byte luminance value (0-255).

For example, for a 4x3 video frame, each frame has 12 pixels. While working, the user should send following 12 bytes successively to the module:


```
Y00  Y01  Y02  Y03  Y10  Y11  Y12  Y13  Y20  Y21  Y22  Y23
```

Where Y00 is the luminance value of row0 column0; Y01 is the luminance value of row0, column1; ......

##### FRAME_TYPE="YUY2"

 `FRAME_TYPE="YUY2"` is color mode, also known as YUYV, in which each pixel has an independent 1-byte luminance value (Y), while two adjacent pixels share a 1-byte blue chroma value (U) and a 1-byte red chroma value (V).

For example, for a 4x2 video frame, the user should send following 12 bytes successively to the module:


```
Y00  U00  Y01  V00  Y02  U02  Y03  V02  Y10  U10  Y11  V10  Y12  U12  Y13  V12
```

Where (Y00, U00, V00) is the pixel of row0 column0; (Y01, U00, V00) is the pixel of row0 column1; (Y02, U02, V02) is the pixel of row0 column2; (Y03, U02, V02) is the pixel of row0 column3. (Y12, U10, V10) is the pixel of row1 column0; ...

#### Ports of usb_camera_top.v

The ports of **usb_camera_top.v** used to read the pixel from the outside is:


```verilog
// pixel fetch signals of usb_camera_top.v    start-of-frame |  frame data transmitting   | end-of-frame
output reg         vf_sof,                // 0000000001000000000000000000000000000000000000000000000000000    // vf_sof=1 indicate a start of video-frame
output reg         vf_req,                // 0000000000000000010001000100010001000100010001000000000000000    // when vf_req=1, a byte of pixel data on vf_byte need to be valid
input  wire [ 7:0] vf_byte,               //                                                                  // a byte of pixel data
```

**usb_camera_top.v** will continuously input and sent frames to the Host-PC.

At the beginning of each video frame, `vf_sof=1` pulses for one cycle. Then  `vf_req=1` pulses for **N** cycles, where **N** is the number of bytes of the video frame, for `FRAME_TYPE="MONO"`, **N** = frame width × frame height; For `FRAME_TYPE="YUY2"`, **N** = 2 × frame width × frame height. Each time when `vf_req=1`, the outside world should provide a byte onto `vf_byte` signal, which should be valid at most 4 cycles after `vf_req=1`  and keep until the next time when `vf_req=1`.

### Frame rate and performance

This module sends the video to the Host-PC with a fixed speed. The larger the video frame size, the smaller the frame rate. The theoretical bandwidth of USB Full Speed is 12Mbps. In fact, the module sends 800 bytes of pixel data (400 pixels) every 1ms. Therefore, the approximate calculation formula of the frame rate is:

Frame Rate = 400000/ (Frame Width × Frame Height)

 　

# Ⅴ U disk

The file hierarchy is as follows. Please add these files to the FPGA project, compile and program it to the FPGA.

- RTL/fpga_examples/**fpga_top_usb_disk.v**
  - RTL/usb_class/**usb_disk_top.v**
    - RTL/usbfs_core/**usbfs_core_top.v**
      - Other .v files in RTL/usbfs_core/

> :warning: The USB core of this repo requires a 60 MHz driving clock. Since I use Altera Cyclone IV, an `altpll` primitive module is used in my code to convert inputted 50MHz clock to 60MHz clock. If your FPGA is not a Altera Cyclone IV, you should remove `altpll` module and use the corresponding primitive or IP core to generate 60 MHz clock. For example, for Xilinx FPGAs, you can use Clock Wizard IP.

### Test

After the USB is plugged in, open the Windows Device Manager, you would find a device:

![](./figures/ls_disk.png)

And you would see a new disk in Windows File Explorer, with one file "example,txt" in it.

![](./figures/test_disk.png)

Since it is implemented with the FPGA's on-chip memory (BRAM), all your modifications will disappear when the FPGA is powered off or re-programed. The free space of this hard disk is only 3.5 KB, which is basically useless and only for testing.

### Application development

You can develop larger or more complex USB-disks based on this simple example, for which you need to pay attention to the in/out ports **usb_disk_top.v**  (see code comments for details).

The key ports including:


```verilog
// signals of usb_disk_top.v
output reg  [40:0] mem_addr,      // byte address
output reg         mem_wen,       // 1:write   0:read
output reg  [ 7:0] mem_wdata,     // byte to write
input  wire [ 7:0] mem_rdata,     // byte to read
```

These signals are used for reading and writing the **Storage space of hard disk**. `mem_addr` Is a 41-bit byte address, so the accessable addressing space is 2^41=2TB. 

At each clock cycle:

- If `mem_wen=1`, the Host wants to write one byte of data to the device, the write address is `mem_addr`, and the write data is `mem_wdata`;
- If `mem_wen=0` the device needs to read a byte of data, the read address is `mem_addr`, and the read data should appear on `mem_rdata` on the next cycle.

This interface is very easy to connect to the BRAM of FPGA.

In order for disk to be recognized as a formatted hard disk, you can provide an initial data to the BRAM, which contains a file system.

> :warning:  A file system is a data structure used to organize files and is stored on the hard disk. The production method is quite complicated and will not be describe here. If you need to make a customized U disk device, you can contact me through issue or email.

 　

# VI USB keyboard

The file hierarchy is as follows. Please add these files to the FPGA project, compile and program it to the FPGA.

- RTL/fpga_examples/**fpga_top_usb_keyboard.v**
  - RTL/usb_class/**usb_keyboard_top.v**
    - RTL/usbfs_core/**usbfs_core_top.v**
      - Other .v files in RTL/usbfs_core/

> :warning: The USB core of this repo requires a 60 MHz driving clock. Since I use Altera Cyclone IV, an `altpll` primitive module is used in my code to convert inputted 50MHz clock to 60MHz clock. If your FPGA is not a Altera Cyclone IV, you should remove `altpll` module and use the corresponding primitive or IP core to generate 60 MHz clock. For example, for Xilinx FPGAs, you can use Clock Wizard IP.

### Test

After the USB is plugged in, open the Windows Device Manager, you would find a device:

![](./figures/ls_keyboard.png)

The keyboard will press a English letter key every 2 seconds.

### Application development

You can develop more complex keyboard applications based on this simple example, for which you need to pay attention to the in/out ports of **usb_keyboard_top.v**  (see code comments for details).

Note the following signals:


```verilog
// signals of usb_keyboard_top.v:
input  wire [15:0] key_value,     // Indicates which key to press, NOT ASCII code! see https://www.usb.org/sites/default/files/hut1_21_0.pdf section 10.
input  wire        key_request,   // when key_request=1 pulses, a key is pressed.
```

 `key_request` usually needs to maintain 0, when you need to press a key, let `key_request=1` for a cycle, meanwhile set a key code on `key_value` . Refer to https://www.usb.org/sites/default/files/hut1_21_0.pdf  Section 10 for the definition of key code. For example, keys 'A'-'Z' corresponds to `key_value=16'h0004~16'h001D` 

 　

# Ⅶ USB-Serial

The file hierarchy is as follows. Please add these files to the FPGA project, compile and program it to the FPGA.

- RTL/fpga_examples/**fpga_top_usb_serial.v**
  - RTL/usb_class/**usb_serial_top.v**
    - RTL/usbfs_core/**usbfs_core_top.v**
      - Other .v files in RTL/usbfs_core/

> :warning: The USB core of this repo requires a 60 MHz driving clock. Since I use Altera Cyclone IV, an `altpll` primitive module is used in my code to convert inputted 50MHz clock to 60MHz clock. If your FPGA is not a Altera Cyclone IV, you should remove `altpll` module and use the corresponding primitive or IP core to generate 60 MHz clock. For example, for Xilinx FPGAs, you can use Clock Wizard IP.

### Test

After the USB is plugged in, open the Windows Device Manager, you would find a device:

![](./figures/ls_serial.png)

In this example, the **usb_serial_top.v** received data is converted from lowercase letters to uppercase letters (ASCII code), and then looped back to the sending interface. You can send data to Serial Port using minicom, putty, HyperTerminal, or Serial Assistant software on Host-PC, and the sent data will be echoed (with lowercase letters converted to uppercase letters). As shown in figure below.

![](./figures/test_serial.png)

> :warning:  Because this USB-CDC-based Serial-Port is not a true UART, it is also called **Virtual serial port**. So there will be no any effects when you change baud rate, data bit, check bit, and stop bits.

### Application development

You can develop more complex Serial-Port communication applications based on this simple example, for which you need to pay attention to the in/out ports of **usb_serial_top.v**  (see the code comments for details).

The key signals are described here, including:


```verilog
// signals of usb_serial_top.v
// receive data (host-to-device)
output wire [ 7:0] recv_data,     // received data byte
output wire        recv_valid,    // when recv_valid=1 pulses, a data byte is received on recv_data
// send data (device-to-host)
input  wire [ 7:0] send_data,     // data byte to send
input  wire        send_valid,    // when device want to send a data byte, set send_valid=1. the data byte will be sent successfully when (send_valid=1 && send_ready=1).
output wire        send_ready,    // send_ready handshakes with send_valid. send_ready=1 indicates send-buffer is not full and will accept the byte on send_data. send_ready=0 indicates send-buffer is full and cannot accept a new byte. 
```

The signal of host-to-device is relatively simple. Every time a data byte is received, `recv_valid` will become high of one cycle, and the byte will appear on `recv_data` at the same time.

The device-to-host signal is in the opposite direction, and there is an `send_ready` signal, `send_ready=0` indicating that the send buffer inside the module is full and cannot send new data temporarily. `send_ready` And `send_valid` form a handshake signal, when the user needs to send a byte, he should let `send_valid=1`, at the same time let the byte appear on `send_data`. When `send_valid=1 && send_ready=1` , the byte is successfully sent to the send buffer, and the user can proceed to send the next byte. This handshake mechanism is similar to AXI-stream.

There is a send buffer of 1024 bytes in **usb_serial_top.v** . If the send throughput is not large, the send buffer will never be full, in this case you can ignore `send_ready`. However, when the send throughput is large, which may cause the sending buffer to be full, then `send_ready` shouldn't be ignored.

 　

# Ⅷ Dual-channel USB-Serial

The file hierarchy is as follows. Please add these files to the FPGA project, compile and program it to the FPGA.

- RTL/fpga_examples/**fpga_top_usb_serial2.v**
  - RTL/usb_class/**usb_serial2_top.v**
    - RTL/usbfs_core/**usbfs_core_top.v**
      - Other .v files in RTL/usbfs_core/

> :warning: The USB core of this repo requires a 60 MHz driving clock. Since I use Altera Cyclone IV, an `altpll` primitive module is used in my code to convert inputted 50MHz clock to 60MHz clock. If your FPGA is not a Altera Cyclone IV, you should remove `altpll` module and use the corresponding primitive or IP core to generate 60 MHz clock. For example, for Xilinx FPGAs, you can use Clock Wizard IP.

### Test

After the USB is plugged in, open the Windows Device Manager, you would find two Serial devices:

![](./figures/ls_serial2.png)

It's behavior is as same as the single-channel USB-Serial, which will not be described here.

### Application development

You can develop a more complex Serial-Port communication application based on this simple example. For this, you need to pay attention to the in/out ports of **usb_serial2_top.v** (see the code comment for details). Its usage is as same as the single-channel USB-Serial, except that the sending and receiving interfaces become dual-channel, which will not be repeated here.

 　

 　

# <span id="usbcore_en"> Ⅸ Development using USB device core</span>

You can use my USB device core (**usbfs_core_top.v**) to develop more USB devices. The core provides:

- Customize 1 device descriptor, 1 configuration descriptor, and 6 string descriptors.
- Four IN endpoints (0x81\~0x84) and four OUT endpoints (0x01\~0x04) .
- An optional debug interface for printing debug information to the computer via a UART, you can see the USB communication process through it.

The parameters and in/out ports of **usbfs_core_top.v** are described below.

### Parameters of usbfs_core_top.v

| Parameters | Type                                 |Explain|
| --------------------- | --------------------------------------- | -------------------------------------------- |
| `DESCRIPTOR_DEVICE`   | `[18*8-1:0]` (can contain 18 bytes) |Device descriptor|
| `DESCRIPTOR_STR1`     | `[64*8-1:0]` (can contain 64 bytes) |String descriptor 1|
| `DESCRIPTOR_STR2`     | `[64*8-1:0]` (can contain 64 bytes) |String Descriptor 2|
| `DESCRIPTOR_STR3`     | `[64*8-1:0]` (can contain 64 bytes) |String Descriptor 3|
| `DESCRIPTOR_STR4`     | `[64*8-1:0]` (can contain 64 bytes) |String Descriptor 4|
| `DESCRIPTOR_STR5`     | `[64*8-1:0]` (can contain 64 bytes) |String Descriptor 5|
| `DESCRIPTOR_STR6`     | `[64*8-1:0]` (can contain 64 bytes) |String Descriptor 6|
| `DESCRIPTOR_CONFIG`   | `[512*8-1:0]` (can contain 512 bytes) |Configuration descriptor|
| `EP00_MAXPKTSIZE`     | `[7:0]` | max packet size of control endpoint |
| `EP81_MAXPKTSIZE`     | `[9:0]` | max packet size of IN endpoint 0x81 |
| `EP82_MAXPKTSIZE`     | `[9:0]` | max packet size of IN endpoint 0x82 |
| `EP83_MAXPKTSIZE`     | `[9:0]` | max packet size of IN endpoint 0x83 |
| `EP84_MAXPKTSIZE`     | `[9:0]` | max packet size of IN endpoint 0x84 |
| `EP81_ISOCHRONOUS`    |0 or 1| Is IN endpoint 0x81 isochronous? |
| `EP82_ISOCHRONOUS`    |0 or 1| Is IN endpoint 0x82 isochronous? |
| `EP83_ISOCHRONOUS`    |0 or 1| Is IN endpoint 0x83 isochronous? |
| `EP84_ISOCHRONOUS`    |0 or 1| Is IN endpoint 0x84 isochronous? |
| `EP01_ISOCHRONOUS`    |0 or 1| Is OUT endpoint 0x01 isochronous? |
| `EP02_ISOCHRONOUS`    |0 or 1| Is OUT endpoint 0x02 isochronous? |
| `EP03_ISOCHRONOUS`    |0 or 1| Is OUT endpoint 0x03 isochronous? |
| `EP04_ISOCHRONOUS`    |0 or 1| Is OUT endpoint 0x04 isochronous? |
| `DEBUG`               |TRUE or FALSE| Enable Debug interface ?    |

> :warning:  According to USB 1.1 specification, when an endpoint is in isochronous transfer mode, the maximum packet size can be any value from 8 \ to 1023. When the endpoint is in interrupt or bulk transfer mode, the maximum packet size can only be 8, 16, 32, or 64.

### Ports of usbfs_core_top.v

#### Clock and Reset

The `clk` signal needs to be 60 MHz:


```verilog
// signals of usbfs_core_top.v
input  wire clk,           // 60MHz is required
```

The reset signal `rstn` should be set to high level during normal operation. If it is necessary to stop the operation, set `rstn` to low. At this time, if the USB is plugged into the Host-PC, the Host-PC will detect that the USB is unplugged.


```verilog
// signals of usbfs_core_top.v
input  wire rstn,          // active-low reset, reset when rstn=0 (USB will unplug when reset)
```

#### USB Signal

The following 3 signals need to be connect according to [Circuit connection](#circuit_en).


```verilog
// signals of usbfs_core_top.v
// USB signals
output reg  usb_dp_pull,   // connect to USB D+ by an 1.5k resistor
inout       usb_dp,        // USB D+
inout       usb_dn,        // USB D-
```

#### Signal for indicating whether the USB is plugged.

The `usb_rstn` signal indicates whether the USB is connected, high for connected and low for disconnected. There are two possible reasons for disconnection: either the USB cable is not plugged in the Host, or the FPGA side is in resetting state ( `rstn=0`).


```verilog
// signals of usbfs_core_top.v
output reg  usb_rstn,      // 1: connected , 0: disconnected (when USB cable unplug, or when system reset (rstn=0))
```

#### USB-transfer and USB-frame detection signal

When `sot` and `sof` are high for one cycle, it indicates that a USB-transfer or a USB-frame is detected, respectively.

USB-transfer refers to the whole process of USB transfer, including control transfer, interrupt transfer, bulk transfer and isochronous transfer.

USB-frame means a SOF token is sent from USB-host to USB-device every 1ms, which can be used to guide the isochronous transfer.


```verilog
// signals of usbfs_core_top.v
output reg  sot,           // detect a start of USB-transfer
output reg  sof,           // detect a start of USB-frame
```

#### Response signal of control transfer

The following three signals `ep00_setup_cmd` `ep00_resp_idx` `ep00_resp` provide the interface for control transfer.


```verilog
// signals of usbfs_core_top.v
// endpoint 0 (control endpoint) command response here
output wire [63:0] ep00_setup_cmd,
output wire [ 8:0] ep00_resp_idx,
input  wire [ 7:0] ep00_resp,
```

According to the USB specification, control transfer is performed only on control endpoint (0x00), and the Host will first send an 8-byte SETUP command. The device may respond data packets to the Host. Based on the fields of `bmRequestType[6:5]` of the SETUP command, control transfer can be divided into three categories:

- Standard control transfer ( `bmRequestType[6:5]=0`)
- Class-specific control transfer ( `bmRequestType[6:5]=1`)
- Vendor-specific control transfer ( `bmRequestType[6:5]=2`)

Standard control transfer is used to respond to descriptor and other data, which is processed internally in **usbfs_core_top.v** and does not need to be concerned by the developer. The developer only needs to specify the descriptor with parameter.

Class-specific control transfer and vendor-specific control transfer are related to the implementation of specific devices. Developers can respond to them through these three signals. For example, the UVC device provided by this repo uses it to respond to the UVC Video Probe and Commit Controls. In addition, some simple devices do not need Class-specific control transfer and Vendor-specific control transfer at all, so developers can ignore these three signals.

When a control transfer is performed, the 8-byte SETUP command will appear on  `ep00_setup_cmd`  first. Then `ep00_resp_idx` signal will increase from 0, representing the byte address that the response is currently required. The developer needs to set `ep00_resp` to the corresponding byte at next cycle.

Taking the UVC device as an example, the following code detects whether the SETUP command requires the device to respond to the UVC Video Probe and Commit Controls.


```verilog
// In the UVC device, the host requests UVC Video Probe and Commit Controls, the following code is to respond this request:
always @ (posedge clk)
    if(ep00_setup_cmd[7:0] == 8'hA1 && ep00_setup_cmd[47:16] == 32'h_0001_0100 )
        ep00_resp <= UVC_PROBE_COMMIT[ep00_resp_idx];
    else
        ep00_resp <= '0;
```

For the content of the command and response data of control transfer, please refer to the specific USB class's specification.

#### IN endpoint (0x81\~0x84) signals

The following 4 groups of signals correspond to 4 IN endpoints and are used to send packets from device to host.

For example, if the FPGA needs to send an IN packet on the IN endpoint 0x81, the following is required:

- First, let `ep81_valid=1` and hold, meanwhile holding the 0th byte of the packet on `ep81_data`.
-  `ep81_ready` and `ep81_valid` form a pair of handshake signals. Each time when `ep81_ready=ep81_valid=1` , a byte is successfully transmitted.
- At the next cycle when `ep81_ready=1` , if the entire IN packet transmission is finished, let `ep81_valid=0` . If the IN packet still has bytes to send, keep `ep81_valid=1` and let `ep81_data` to be the next byte to be sent.


```verilog
// signals of usbfs_core_top.v
// endpoint 0x81 data input (device-to-host)
input  wire [ 7:0] ep81_data,     // IN data byte
input  wire        ep81_valid,    // when device want to send a data byte, assert valid=1. the data byte will be sent successfully when valid=1 & ready=1.
output wire        ep81_ready,    // handshakes with valid. ready=1 indicates the data byte can be accept.
// endpoint 0x82 data input (device-to-host)
input  wire [ 7:0] ep82_data,     // IN data byte
input  wire        ep82_valid,    // when device want to send a data byte, assert valid=1. the data byte will be sent successfully when valid=1 & ready=1.
output wire        ep82_ready,    // handshakes with valid. ready=1 indicates the data byte can be accept.
// endpoint 0x83 data input (device-to-host)
input  wire [ 7:0] ep83_data,     // IN data byte
input  wire        ep83_valid,    // when device want to send a data byte, assert valid=1. the data byte will be sent successfully when valid=1 & ready=1.
output wire        ep83_ready,    // handshakes with valid. ready=1 indicates the data byte can be accept.
// endpoint 0x84 data input (device-to-host)
input  wire [ 7:0] ep84_data,     // IN data byte
input  wire        ep84_valid,    // when device want to send a data byte, assert valid=1. the data byte will be sent successfully when valid=1 & ready=1.
output wire        ep84_ready,    // handshakes with valid. ready=1 indicates the data byte can be accept.
```

#### OUT endpoint (0x01\~0x04) signals

The following four groups of signals correspond to four OUT endpoints and are used to receive the OUT packet from host to device.

For example, when `ep01_valid=1` for one cycle, a byte of OUT packet is received, the byte appears on `ep01_data` . In addition, the boundary of different packets can be detected by the `sot` signal mentioned before.


```verilog
// signals of usbfs_core_top.v
// endpoint 0x84 data input (device-to-host)
input  wire [ 7:0] ep84_data,     // IN data byte
input  wire        ep84_valid,    // when device want to send a data byte, assert valid=1. the data byte will be sent successfully when valid=1 & ready=1.
output wire        ep84_ready,    // handshakes with valid. ready=1 indicates the data byte can be accept.
// endpoint 0x01 data output (host-to-device)
output wire [ 7:0] ep01_data,     // OUT data byte
output wire        ep01_valid,    // when out_valid=1 pulses, a data byte is received on out_data
// endpoint 0x02 data output (host-to-device)
output wire [ 7:0] ep02_data,     // OUT data byte
output wire        ep02_valid,    // when out_valid=1 pulses, a data byte is received on out_data
// endpoint 0x03 data output (host-to-device)
output wire [ 7:0] ep03_data,     // OUT data byte
output wire        ep03_valid,    // when out_valid=1 pulses, a data byte is received on out_data
// endpoint 0x04 data output (host-to-device)
output wire [ 7:0] ep04_data,     // OUT data byte
output wire        ep04_valid,    // when out_valid=1 pulses, a data byte is received on out_data
```

#### Debug output interface

To enable Debug interface, set the parameter `DEBUG=1` . Otherwise the debug interface will not output any data.

The following signals are used to print debug information to the outside world. The debug information is a stream of printable ASCII code bytes. When `debug_en=1` , one byte appears on `debug_data` .


```verilog
// signals of usbfs_core_top.v
// debug output info, only for USB developers, can be ignored for normally use
output wire        debug_en,      // when debug_en=1 pulses, a byte of debug info appears on debug_data
output wire [ 7:0] debug_data,    // 
output wire        debug_uart_tx  // debug_uart_tx is the signal after converting {debug_en,debug_data} to UART (format: 115200,8,n,1). If you want to transmit debug info via UART, you can use this signal. If you want to transmit debug info via other custom protocols, please ignore this signal and use {debug_en,debug_data}.
```

In order to facilitate the use, I will also convert the `{debug_en,debug_data}` to a UART output signal `debug_uart_tx` . You can connect the UART to PC, and use a Serial Port software (minicom, HyperTermainal, etc) to monitor the debugging information.

Note that the UART configurations are : baud rate=115200, data bits=8, no parity bits.

The following figure shows the debug data printed on the UART when my USB UVC device is plugged into the computer. As you can see, this is a descriptor enumeration process.

|![debug_info](./figures/debug.png)|
| :----------------------------------------------------: |
|**Figure**: Debugging data printed on the UART when the USB UVC device is plugged into the computer.|



> :warning:  Because the speed of UART is slow, when a large amount of debugging information is generated, some debugging information will be discarded. For example, when UVC transmits video, the information printed by UART is incomplete. If you want to view the full debugging information when large amount of USB data, you should use other high-speed communication to send `{debug_en, debug_data}` to Host-PC instead of UART.

 　

　

# References

* https://www.usbmadesimple.co.uk/ : USB Made Simple
* https://github.com/FengJungle/USB_Protocol_CH  USB Chinese Protocol Manual
* https://www.usb.org/document-library/usb-20-specification : USB 2.0 Specification
* https://www.usb.org/document-library/video-class-v11-document-set : USB Video Class (UVC) 1.1 Specification
* https://www.usb.org/document-library/audio-device-class-spec-basic-audio-devices-v10-and-adopters-agreement : USB Audio Class (UAC) 1.0 Specification
* https://www.usb.org/document-library/mass-storage-class-specification-overview-14 : USB Mass Storage Class (USB-MSC) 1.4 Specification
* https://www.usb.org/document-library/class-definitions-communication-devices-12 : USB Communication Device Class (USB-CDC) 1.2 Specification
* Other FPGA implementations:
  * https://github.com/avakar/usbcorev : a USB-device controller, only supported up to the transaction layer.
  * http://jorisvr.nl/article/usb-serial : One USB-CDC, VHDL implementation, requires additional UTMI PHY.
  * https://github.com/pbing/USB  A USB-HID implementation for Low Speed.
  * https://github.com/ultraembedded/cores : Contains some USB-host and USB-device implementations, requiring UTMI PHY or ULPI PHY.

　

　

　


　

<span id="cn">FPGA USB-device</span>
===========================

USB 1.1 device 控制器。可在 FPGA 上实现各种 USB 设备。比如 USB扬声器和麦克风、USB摄像头、U盘、USB键盘、USB串口 。

为了在 FPGA 上实现 USB 设备，通常的技术路线是使用 USB 芯片 (例如 Cypress CY7C68013)，导致电路和软件的开发成本较高。本库用 FPGA 实现一个通用的 USB 1.1 (Full Speed) device 控制器，可以像 STM32 单片机那样，用非常简单的电路来实现 USB 设备，而不依赖额外的 USB 芯片。

基于此，我还在 FPGA 上实现了 USB音频、USB摄像头、U盘、USB键盘、USB-Serial (串口)，它们是 USB 所规定的标准设备，因此不需要安装驱动就能即插即用。

本库的特点：

- 纯 Verilog 实现，适用于 Xilinx 、Altera 等各种型号的 FPGA 。
- 所需的电路非常简单，除了FPGA外，**只需3个FPGA引脚，1个电阻，1个USB接口座**（见[电路连接](#circuit)）

如果你不熟悉 USB 协议栈，但想快速实现某种 USB 设备，可以使用我封装的一些 USB 功能：

- **USB音频** : 包括扬声器和麦克风 (双声道, 48ksps) ，提供接收扬声器音频数据、发送麦克风音频数据的流式接口。
- **USB摄像头** : 可向电脑传输视频（宽和高可自定义），提供发送视频数据的流式接口。
- **U盘**。
- **USB键盘**：提供"按键按下"的控制接口。
- **USB-Serial** : 实现串口设备，可在电脑上使用 minicom, putty, HyperTerminal, 串口助手等软件与 FPGA 进行数据传输。
- **USB-Serial-2ch** : 是一个复合设备，包括2个独立的 USB-Serial 。

如果你熟悉 USB 协议栈，可以使用本库来开发更多的 USB 设备 (详见[USB-device二次开发](#usbcore))。它提供：

- 自定义1个设备描述符(device descriptor)、1个配置描述符(configuration descriptor)、6个字符串描述符(string descriptor) 。
- 4 个 IN endpoint (0x81\~0x84) 、 4 个 OUT endpoint (0x01\~0x04) 。
- 可选的调试输出接口 (debug interface)，通过一个额外的UART打印调试信息到电脑，可以看到 USB 数据包的通信过程。

 　

## 效果展示

|                    | Windows 设备管理器中看到的设备 |            效果展示            |
| :----------------: | :----------------------------: | :----------------------------: |
|    **USB音频**     |  ![](./figures/ls_audio.png)   | ![](./figures/test_audio.png)  |
|   **USB摄像头**    |  ![](./figures/ls_camera.png)  | ![](./figures/test_camera.png) |
|      **U盘**       |   ![](./figures/ls_disk.png)   |  ![](./figures/test_disk.png)  |
|    **USB键盘**     | ![](./figures/ls_keyboard.png) |    每2秒按下一个英文字母键     |
|   **USB-Serial**   |  ![](./figures/ls_serial.png)  | ![](./figures/test_serial.png) |
| **USB-Serial-2ch** | ![](./figures/ls_serial2.png)  |              同上              |

 　

## 兼容性

我测试了这些设备在不同操作系统上的兼容性，如下表。

|     兼容性测试     |     Windows 10     |          Linux Ubuntu 18.04          |    macOS 10.15     |
| :----------------: | :----------------: | :----------------------------------: | :----------------: |
|    **USB音频**     | :heavy_check_mark: | :heavy_check_mark: (不识别bug已解决) | :heavy_check_mark: |
|   **USB摄像头**    | :heavy_check_mark: |          :heavy_check_mark:          |        :x:         |
|      **U盘**       | :heavy_check_mark: |          :heavy_check_mark:          | :heavy_check_mark: |
|    **USB键盘**     | :heavy_check_mark: |          :heavy_check_mark:          | :heavy_check_mark: |
|   **USB-Serial**   | :heavy_check_mark: |          :heavy_check_mark:          | :heavy_check_mark: |
| **USB-Serial-2ch** | :heavy_check_mark: | :heavy_check_mark: (不识别bug已解决) | :heavy_check_mark: |

> :x: macOS 能识别我的 USB 摄像头设备，但无法读出视频，原因未知，留待后续解决。

 　

### 特别鸣谢

- 感谢 [github.com/xiaowuzxc](https://github.com/xiaowuzxc) 提交的 USB 扬声器的代码。本人后来将其拓展为现在的 USB音频 (扬声器+麦克风)
- 感谢 \*\*\*\*\*8125@qq.com 解决了 USB-Serial-2ch 中有一个通道不识别的问题。

 　

# <span id="circuit">Ⅰ 电路连接</span>

USB 具有 `VBUS`, `GND`, `USB_D-`, `USB_D+` 这4根线。以 USB Type B 连接座（俗称USB方口母座）为例，这4根线定义如下图。

| ![USBTypeB](./figures/usb_typeb.png)  |
| :-----------------------------------: |
| **图**：USB 连接座（方口母座）与线。 |

请进行如下图的电路连接。其中 `usb_dp_pull`, `usb_dp`, `usb_dn` 是 FPGA 的 3 个普通IO引脚（电平必须为 3.3V）。其中：

- FPGA 的 `usb_dn` 接 `USB_D-` 。:warning: 如果是飞线连接，要保证连接线长度在10cm以内。如果是PCB连接，要与 `usb_dp` 差分布线。
- FPGA 的 `usb_dp` 接 `USB_D+` 。:warning: 如果是飞线连接，要保证连接线长度在10cm以内；如果是PCB连接，要与 `usb_dn` 差分布线。
- FPGA 的 `usb_dp_pull` 要通过 1.5kΩ 的电阻接  `USB_D+` 
- FPGA 的 `GND`  接 USB 连接座的 `GND` 
- USB 连接座的 `VBUS` 是一个 5V, 500mA 的电源，可以不连，也可给 FPGA 供电。


```
  _________________
  |               |
  |   usb_dp_pull |-------|
  |               |       |
  |               |      |-| 1.5k resistor
  |               |      | |
  |               |      |_|        ____________                  __________
  |               |       |         |          |                  |
  |        usb_dp |-------^---------| USB_D+   |                  |
  |               |                 |          |    USB cable     |
  |        usb_dn |-----------------| USB_D-   |<================>| Host PC
  |               |                 |          |                  |
  |           GND |-----------------| GND      |                  |
  |               |                 |          |                  |
  -----------------                 ------------                  ----------
        FPGA                          USB 连接座                      电脑
                       图 : FPGA 连接 USB 的方法
```

 　

# Ⅱ 代码文件一览

[RTL](./RTL) 文件夹包含了所有代码，其中按层级分为三个文件夹：

|                  文件夹                  |      层级       | 说明                                                         |
| :--------------------------------------: | :-------------: | :----------------------------------------------------------- |
| [RTL/fpga_examples](./RTL/fpga_examples) |     应用层      | 在 USB class 的基础上实现具体的应用功能。为了方便测试，这里实现的应用功能非常简单，例如回环测试 USB-Serial、生成黑白条纹给 USB 摄像头。你可以开发复杂的应用，例如采集 CMOS 图象传感器的数据给 USB 摄像头。 |
|     [RTL/usb_class](./RTL/usb_class)     |    USB class    | 在 USB device core 的基础上实现了一些 USB class 。例如用 USB Communication Device Class (USB-CDC) 实现 USB-Serial；用 USB Video Class (UVC) 实现摄像头 …… |
|    [RTL/usbfs_core](./RTL/usbfs_core)    | USB device core | 一个通用的 USB device core，实现 USB 底层信号的处理，包括从 bit-level 到 transaction-level 。熟悉 USB 协议栈的开发者可以用它来开发更多的 USB 设备，详见 [USB device 二次开发](#usbcore) 。 |

 　

具体地，对每个代码文件说明如下：

|                  文件夹                  |          文件名          | 说明                                                         |
| :--------------------------------------: | :----------------------: | :----------------------------------------------------------- |
| [RTL/fpga_examples](./RTL/fpga_examples) |  fpga_top_usb_audio.v   | 把 usb_audio_top.v 的扬声器和麦克风回环连接，扬声器的放音会被麦克风录到 |
| [RTL/fpga_examples](./RTL/fpga_examples) |  fpga_top_usb_camera.v  | 生成黑白条纹给 usb_camera_top.v ，用照相机软件能看到这个黑白条纹 |
| [RTL/fpga_examples](./RTL/fpga_examples) |   fpga_top_usb_disk.v   | 用 usb_disk_top.v 实现了 24 KB 的 FAT16 文件系统的 U盘      |
| [RTL/fpga_examples](./RTL/fpga_examples) | fpga_top_usb_keyboard.v | 每2秒生成一个按键信号给 usb_keyboard_top.v                  |
| [RTL/fpga_examples](./RTL/fpga_examples) |  fpga_top_usb_serial.v  | 把 usb_serial_top.v 的发送和接收回环连接，电脑上发送的字符会被回显 |
| [RTL/fpga_examples](./RTL/fpga_examples) | fpga_top_usb_serial2.v  | 把 usb_serial2_top.v 的发送和接收回环连接，电脑上发送的字符会被回显 |
|     [RTL/usb_class](./RTL/usb_class)     |     usb_audio_top.v     | 实现 **USB扬声器+麦克风** |
|     [RTL/usb_class](./RTL/usb_class)     |    usb_camera_top.v     | USB Video Class (UVC) 实现 **USB摄像头**                     |
|     [RTL/usb_class](./RTL/usb_class)     |     usb_disk_top.v      | USB Mass Storage Class (USB-MSC) 实现 **U盘**                |
|     [RTL/usb_class](./RTL/usb_class)     |   usb_keyboard_top.v    | USB Human Interface Device Class (USB-HID) 实现 **USB键盘**  |
|     [RTL/usb_class](./RTL/usb_class)     |    usb_serial_top.v     | USB Communication Device Class (USB-CDC) 实现 **USB-Serial** |
|     [RTL/usb_class](./RTL/usb_class)     |    usb_serial2_top.v    | 复合设备，包含2个 USB-CDC ，实现 **双通道USB-Serial**        |
|    [RTL/usbfs_core](./RTL/usbfs_core)    |  **usbfs_core_top.v**   | USB device core 的顶层模块                                   |
|    [RTL/usbfs_core](./RTL/usbfs_core)    |   usbfs_transaction.v   | 实现 USB transaction-level，被 usbfs_core_top.v 调用        |
|    [RTL/usbfs_core](./RTL/usbfs_core)    |    usbfs_packet_rx.v    | 实现 USB packet-level RX，被 usbfs_core_top.v 调用          |
|    [RTL/usbfs_core](./RTL/usbfs_core)    |    usbfs_packet_tx.v    | 实现 USB packet-level TX，被 usbfs_core_top.v 调用          |
|    [RTL/usbfs_core](./RTL/usbfs_core)    |    usbfs_bitlevel.v     | 实现 USB bit-level，被 usbfs_core_top.v 调用                |
|    [RTL/usbfs_core](./RTL/usbfs_core)    |  usbfs_debug_monitor.v  | 收集调试信息（不干扰 USB 的功能），被 usbfs_core_top.v 调用 |
|    [RTL/usbfs_core](./RTL/usbfs_core)    |        usbfs_debug_uart_tx.v        | 将调试信息转为一个 UART 信号，被 usbfs_core_top.v 调用      |

下文逐一介绍本库提供的每一个 USB class 的使用方法。最后会介绍 [基于 USB device core 的二次开发](#usbcore) 。

 　

# Ⅲ USB 音频

本库的 USB 音频设备实现了扬声器+麦克风。代码文件调用关系如下。请将这些文件加入 FPGA 工程，编译并烧录到 FPGA 。

- RTL/fpga_examples/**fpga_top_usb_audio.v**
  - RTL/usb_class/**usb_audio_top.v**
    - RTL/usbfs_core/**usbfs_core_top.v**
      - RTL/usbfs_core/里的其它.v文件(不逐个列出了)

> :warning: 本库的 USB core 需要 60MHz 的驱动时钟。**fpga_top_usb_audio.v** 里调用了 Altera 的 altpll 原语来把晶振的 50MHz 时钟转为 60MHz 时钟。如果你的 FPGA 不是 Altera Cyclone IV ，请删掉 altpll 部分的代码，然后用对应的原语或 IP 核来生成 60MHz 时钟。例如对于 Xilinx FPGA ，可以使用 Clock Wizard IP 。

### 测试

USB插入后，打开 Windows 设备管理器，应该能看到音频设备：

![](./figures/ls_audio.png) 

由于 **fpga_top_usb_audio.v** 将扬声器和麦克风回环连接，因此扬声器的放音会被麦克风录到。为了测试，首先要选择该设备为音频输出设备：

![](./figures/select_audio.png) 

然后随便放一个音乐。再用任意录音软件或录视频软件 (比如 obstudio) ，选择 FPGA-USB-audio 作为输入麦克风，录一段音或视频。最后你会发现录到的音频和你放的音乐一样。

### 应用开发

你可以基于这个简单的例子开发更复杂的音频应用，为此你需要关注 **usb_audio_top.v** 的模块接口，详见代码注释，这里不做赘述。

 　

# Ⅳ USB 摄像头

本库的 USB 摄像头设备的代码文件调用关系如下。请将这些文件加入 FPGA 工程，编译并烧录到 FPGA 。

- RTL/fpga_examples/**fpga_top_usb_camera.v**
  - RTL/usb_class/**usb_camera_top.v**
    - RTL/usbfs_core/**usbfs_core_top.v**
      - RTL/usbfs_core/里的其它.v 文件(不逐个列出了)

> :warning: 本库的 USB core 需要 60MHz 的驱动时钟。**fpga_top_usb_camera.v** 里调用了 Altera 的 altpll 原语来把晶振的 50MHz 时钟转为 60MHz 时钟。如果你的 FPGA 不是 Altera Cyclone IV ，请删掉 altpll 部分的代码，然后用对应的原语或 IP 核来生成 60MHz 时钟。例如对于 Xilinx FPGA ，可以使用 Clock Wizard IP 。

### 测试

USB插入后，打开 Windows 设备管理器，应该能看到摄像头设备：

![](./figures/ls_camera.png) 

打开 Windows 10 自带的照相机软件，应该能看到一个滚动的黑白条纹：

![](./figures/test_camera.png) 

### 应用开发

你可以基于这个简单的例子开发更复杂的摄像头应用，为此你需要关注 **usb_camera_top.v** 的模块接口（详见代码注释）。

#### usb_camera_top.v 的参数

这里对关键的参数进行说明：

```verilog
// parameters of usb_camera_top.v
parameter        FRAME_TYPE = "YUY2",    // "MONO" or "YUY2"
parameter [13:0] FRAME_W    = 14'd320,   // video-frame width  in pixels, must be a even number
parameter [13:0] FRAME_H    = 14'd240,   // video-frame height in pixels, must be a even number
```

你可以用 `FRAME_W` 和 `FRAME_H` 设置视频帧的宽和高。`FRAME_TYPE` 可取 `"MONO"` 或 `"YUY2"` 。

##### FRAME_TYPE="MONO"

`FRAME_TYPE="MONO"` 是灰度模式，每个像素对应1字节的明度值。例如，对于一个 4x3 的视频帧，该模块会先后从外界读取以下12个字节：

```
Y00  Y01  Y02  Y03  Y10  Y11  Y12  Y13  Y20  Y21  Y22  Y23
```

其中 Y00 是第0行第0列的明度值；Y01 是第0行第1列的明度值；……

##### FRAME_TYPE="YUY2"

`FRAME_TYPE="YUY2"`  是一种彩色模式，又称为 YUYV ，每个像素具有独立的 1字节明度值(Y)，而相邻的两个像素共享1字节的蓝色度(U)和1字节的红色度(V) 。例如，对于一个 4x2 的视频帧，该模块会先后从外界读取以下16个字节：

```
Y00  U00  Y01  V00  Y02  U02  Y03  V02  Y10  U10  Y11  V10  Y12  U12  Y13  V12
```

其中 (Y00, U00, V00) 是第0行第0列的像素；(Y01, U00, V00) 是第0行第1列的像素；(Y02, U02, V02) 是第0行第2列的像素；(Y03, U02, V02) 是第0行第3列的像素；(Y10, U10, V10) 是第1行第0列的像素；……

#### usb_camera_top.v 的信号

 **usb_camera_top.v** 中用来从外界读取像素的信号是：

```verilog
// pixel fetch signals of usb_camera_top.v    start-of-frame |  frame data transmitting   | end-of-frame
output reg         vf_sof,                // 0000000001000000000000000000000000000000000000000000000000000    // vf_sof=1 indicate a start of video-frame
output reg         vf_req,                // 0000000000000000010001000100010001000100010001000000000000000    // when vf_req=1, a byte of pixel data on vf_byte need to be valid
input  wire [ 7:0] vf_byte,               //                                                                  // a byte of pixel data
```

 **usb_camera_top.v** 会通过以上信号不断读取视频帧，并发送给 Host-PC 。在每个视频帧的开始， `vf_sof` 会出现一周期的高电平。然后 `vf_req` 会断续地出现 **N** 次高电平，其中 **N** 是视频帧的字节数，对于 `FRAME_TYPE="MONO"` ，**N**=帧宽度×帧高度；对于 `FRAME_TYPE="YUY2"` ，**N**=2×帧宽度×帧高度 。每当 `vf_req=1` 时，外界应该提供一个字节（像素数据）到 `vf_byte` 信号上，最晚应该在 `vf_req=1` 后的第4个时钟周期让 `vf_byte` 有效，并且保持有效直到下一次 `vf_req=1` 。

### 帧率与性能

本模块以固定带宽发送视频到 Host-PC ，视频帧的尺寸越大，帧率越小。 USB Full Speed 的理论带宽是 12Mbps ，实际上本模块每1ms固定发送 800 字节的像素数据（400个像素），因此帧率的大致计算公式为：

帧率 = 400000 / (帧宽度×帧高度)    *(帧/秒)*

 　

# Ⅴ U盘

本库的 U盘设备的代码文件调用关系如下。请将这些文件加入 FPGA 工程，编译并烧录到 FPGA 。

- RTL/fpga_examples/**fpga_top_usb_disk.v**
  - RTL/usb_class/**usb_disk_top.v**
    - RTL/usbfs_core/**usbfs_core_top.v**
      - RTL/usbfs_core/里的其它.v文件(不逐个列出了)

> :warning: 本库的 USB core 需要 60MHz 的驱动时钟。**fpga_top_usb_disk.v** 里调用了 Altera 的 altpll 原语来把晶振的 50MHz 时钟转为 60MHz 时钟。如果你的 FPGA 不是 Altera Cyclone IV ，请删掉 altpll 部分的代码，然后用对应的原语或 IP 核来生成 60MHz 时钟。例如对于 Xilinx FPGA ，可以使用 Clock Wizard IP 。

### 测试

USB插入后，打开 Windows 设备管理器，应该能看到一个硬盘：

![](./figures/ls_disk.png) 

Windows 文件资源管理器中应该能看到这个硬盘，里面有一个文件 "example.txt" 。你可以在这个硬盘里添加、修改、删除文件。由于它是用 FPGA 的片内存储器 (BRAM) 实现的，因此 FPGA 断电或重新烧录后，你的所有修改都会消失。这个硬盘的可用空间只有 3.5 KB ，基本上没啥用，仅供测试。

![](./figures/test_disk.png) 

### 应用开发

你可以基于这个简单的例子开发更大或更复杂的 USB-disk ，为此你需要关注 **usb_disk_top.v** 的模块接口（详见代码注释）。

这里对其中关键的信号进行说明，包括：

```verilog
// signals of usb_disk_top.v
output reg  [40:0] mem_addr,      // byte address
output reg         mem_wen,       // 1:write   0:read
output reg  [ 7:0] mem_wdata,     // byte to write
input  wire [ 7:0] mem_rdata,     // byte to read
```

这些信号用于读写 **硬盘的存储空间** 。 `mem_addr` 是 41-bit 的字节地址，因此寻址空间为 2^41=2TB 。不过实际的硬盘存储空间肯定小于 2TB ，因此只有从 `mem_addr=0` 到 `mem_addr=硬盘容量` 的地址是有效的。在每个时钟周期：

- 如果 `mem_wen=1` ，说明 Host 想写一字节的数据给设备，写地址为 `mem_addr` ，写数据为 `mem_wdata` ；
- 如果 `mem_wen=0` ，设备需要读一个字节的数据，读地址为 `mem_addr`  ，读数据应该在下一周期送到  `mem_rdata` 上。

该接口非常容易接到 FPGA 的 BRAM 上，这样我们就用 BRAM 实现了硬盘的存储空间。

为了让 disk 能被识别为格式化好的硬盘，你可以给这个 BRAM 提供一个初始数据，里面包含一个文件系统。**fpga_top_usb_disk.v** 里的 BRAM 就实现了一个总大小为 24KB 的 FAT16 文件系统。

> :warning: 文件系统是用来组织文件的数据结构，和文件本身一样也存储在硬盘里。制作方法比较复杂，不在这里赘述。如果需要制作定制的 U盘设备，可以通过 issue 联系本人。

 　

# Ⅵ USB键盘

本库的 USB 键盘设备的代码文件调用关系如下。请将这些文件加入 FPGA 工程，编译并烧录到 FPGA 。

- RTL/fpga_examples/**fpga_top_usb_keyboard.v**
  - RTL/usb_class/**usb_keyboard_top.v**
    - RTL/usbfs_core/**usbfs_core_top.v**
      - RTL/usbfs_core/里的其它.v文件(不逐个列出了)

> :warning: 本库的 USB core 需要 60MHz 的驱动时钟。**fpga_top_usb_keyboard.v** 里调用了 Altera 的 altpll 原语来把晶振的 50MHz 时钟转为 60MHz 时钟。如果你的 FPGA 不是 Altera Cyclone IV ，请删掉 altpll 部分的代码，然后用对应的原语或 IP 核来生成 60MHz 时钟。例如对于 Xilinx FPGA ，可以使用 Clock Wizard IP 。

### 测试

USB插入后，打开 Windows 设备管理器，应该能看到一个键盘设备：

![](./figures/ls_keyboard.png) 

该键盘会每2秒按下英文字母按键。打开一个记事本就能看到效果。

### 应用开发

你可以基于这个简单的例子开发更复杂的键盘应用，为此你需要关注 **usb_keyboard_top.v** 的模块接口（详见代码注释）。

这里的对其中用来发送按键信号的信号进行说明：

```verilog
// signals of usb_keyboard_top.v:
input  wire [15:0] key_value,     // Indicates which key to press, NOT ASCII code! see https://www.usb.org/sites/default/files/hut1_21_0.pdf section 10.
input  wire        key_request,   // when key_request=1 pulses, a key is pressed.
```

`key_request`平时需要保持为0，当你需要按下一个键时，需要让 `key_request=1` 一个周期，同时在 `key_value` 上输入该按键对应的代码。按键代码的定义详见 https://www.usb.org/sites/default/files/hut1_21_0.pdf 的 Section 10 。例如，按键 'a'-'z' 对应 `key_value=16'h0004~16'h001D`  ，按键 '1'-'0' 对应 `key_value=16'h001E~16'h0027`  。

 　

# Ⅶ USB-Serial

本库的 USB-Serial 设备的代码文件调用关系如下。请将这些文件加入 FPGA 工程，编译并烧录到 FPGA 。

- RTL/fpga_examples/**fpga_top_usb_serial.v**
  - RTL/usb_class/**usb_serial_top.v**
    - RTL/usbfs_core/**usbfs_core_top.v**
      - RTL/usbfs_core/里的其它.v文件(不逐个列出了)

> :warning: 本库的 USB core 需要 60MHz 的驱动时钟。**fpga_top_usb_serial.v** 里调用了 Altera 的 altpll 原语来把晶振的 50MHz 时钟转为 60MHz 时钟。如果你的 FPGA 不是 Altera Cyclone IV ，请删掉 altpll 部分的代码，然后用对应的原语或 IP 核来生成 60MHz 时钟。例如对于 Xilinx FPGA ，可以使用 Clock Wizard IP 。

### 测试

USB插入后，打开 Windows 设备管理器，应该能看到一个 USB-serial 设备：

![](./figures/ls_serial.png) 

本例把 **usb_serial_top.v** 收到的数据按照 ASCII 码把小写字母转换为大写字母，然后回环连接到发送接口。你可以在 Host-PC 上用 minicom, putty, HyperTerminal, 串口助手的软件来发送数据给 Serial Port ，发送的数据会被回显（其中小写字母转化为大写字母）。以串口助手为例，如下图。

![](./figures/test_serial.png) 

> :warning: 因为该 Serial-Port 并不是真正的 UART，所以又称为**虚拟串口**。设置虚拟串口的波特率、数据位、校验位、停止位将不会有任何效果。

### 应用开发

你可以基于这个简单的例子开发更复杂的 Serial-Port 通信应用，为此你需要关注 **usb_serial_top.v** 的模块接口（详见代码注释）。

这里对其中关键的信号进行说明，包括：

```verilog
// signals of usb_serial_top.v
// receive data (host-to-device)
output wire [ 7:0] recv_data,     // received data byte
output wire        recv_valid,    // when recv_valid=1 pulses, a data byte is received on recv_data
// send data (device-to-host)
input  wire [ 7:0] send_data,     // data byte to send
input  wire        send_valid,    // when device want to send a data byte, set send_valid=1. the data byte will be sent successfully when (send_valid=1 && send_ready=1).
output wire        send_ready,    // send_ready handshakes with send_valid. send_ready=1 indicates send-buffer is not full and will accept the byte on send_data. send_ready=0 indicates send-buffer is full and cannot accept a new byte. 
```

其中 host-to-device 的信号较为简单，每当收到数据字节时，`recv_valid` 出现一个周期的高电平，同时 `recv_data` 上出现该字节。

而 device-to-host 的信号方向相反，而且多出来了一个 `send_ready` 信号，`send_ready=0` 说明模块内部的发送缓冲区满了，暂时不能发送新的数据。 `send_ready`与 `send_valid` 构成握手信号，当用户需要发送一个字节时，应该让 `send_valid=1` ，同时让字节出现再 `send_data` 上。当 `send_valid=1 && send_ready=1` 时，该字节被成功送入发送缓存，用户就可以继而发送下一字节。该握手机制类似 AXI-stream 。

**usb_serial_top.v** 中有 1024 字节的发送缓存。如果需要发送的数据吞吐率不大，发送缓存永远不会满，也可也无视 `send_ready` 信号。然而，在数据吞吐率较大从而可能导致发送缓存满的情况下，如果无视 `send_ready=0`  的情况，可能导致部分数据丢失。

 　

# Ⅷ 双通道 USB-Serial

本库的双通道 USB-Serial 设备的代码文件调用关系如下。请将这些文件加入 FPGA 工程，编译并烧录到 FPGA 。

- RTL/fpga_examples/**fpga_top_usb_serial2.v**
  - RTL/usb_class/**usb_serial2_top.v**
    - RTL/usbfs_core/**usbfs_core_top.v**
      - RTL/usbfs_core/里的其它.v文件(不逐个列出了)

> :warning: 本库的 USB core 需要 60MHz 的驱动时钟。**fpga_top_usb_serial2.v** 里调用了 Altera 的 altpll 原语来把晶振的 50MHz 时钟转为 60MHz 时钟。如果你的 FPGA 不是 Altera Cyclone IV ，请删掉 altpll 部分的代码，然后用对应的原语或 IP 核来生成 60MHz 时钟。例如对于 Xilinx FPGA ，可以使用 Clock Wizard IP 。

### 测试

USB插入后，打开 Windows 设备管理器，应该能看到2个 USB-serial 设备：

![](./figures/ls_serial2.png) 

测试方法与单通道的 USB-Serial 相同，这里不做赘述。

### 应用开发

你可以基于这个简单的例子开发更复杂的 Serial-Port 通信应用，为此你需要关注 **usb_serial2_top.v** 的模块接口（详见代码注释），其用法与单通道的 USB-Serial 相同，只不过发送和接收接口都变成了双通道，这里不做赘述。

 　

# <span id="usbcore">Ⅸ 基于 USB device core 的二次开发</span>

你可以使用 **usbfs_core_top.v** 开发其它的 USB 设备，它提供：

- 自定义1个设备描述符(device descriptor)、1个配置描述符(configuration descriptor)、6个字符串描述符(string descriptor) 。
- 除了 control endpoint (0x00) 外，提供 4 个 IN endpoint (0x81\~0x84) 、 4 个 OUT endpoint (0x01\~0x04) 。
- 可选的调试输出接口 (debug interface)，通过一个额外的UART打印调试信息到电脑，可以看到 USB 数据包的通信过程。

下文对 **usbfs_core_top.v** 的参数和输入输出信号进行说明。

### usbfs_core_top.v 的参数

**usbfs_core_top.v** 的参数如下表。

| usbfs_core_top 的参数 | 类型                        | 说明                                      |
| --------------------- | --------------------------- | ----------------------------------------- |
| `DESCRIPTOR_DEVICE`   | `[18*8-1:0]` (最长18字节)   | 设备描述符                                |
| `DESCRIPTOR_STR1`     | `[64*8-1:0]` (最长64字节)   | 字符串描述符1                             |
| `DESCRIPTOR_STR2`     | `[64*8-1:0]` (最长64字节)   | 字符串描述符2                             |
| `DESCRIPTOR_STR3`     | `[64*8-1:0]` (最长64字节)   | 字符串描述符3                             |
| `DESCRIPTOR_STR4`     | `[64*8-1:0]` (最长64字节)   | 字符串描述符4                             |
| `DESCRIPTOR_STR5`     | `[64*8-1:0]` (最长64字节)   | 字符串描述符5                             |
| `DESCRIPTOR_STR6`     | `[64*8-1:0]` (最长64字节)   | 字符串描述符6                             |
| `DESCRIPTOR_CONFIG`   | `[512*8-1:0]` (最长512字节) | 配置描述符                                |
| `EP00_MAXPKTSIZE`     | `[7:0]`                     | control endpoint 最大包大小               |
| `EP81_MAXPKTSIZE`     | `[9:0]`                     | IN endpoint 0x81 最大包大小               |
| `EP82_MAXPKTSIZE`     | `[9:0]`                     | IN endpoint 0x82 最大包大小               |
| `EP83_MAXPKTSIZE`     | `[9:0]`                     | IN endpoint 0x83 最大包大小               |
| `EP84_MAXPKTSIZE`     | `[9:0]`                     | IN endpoint 0x84 最大包大小               |
| `EP81_ISOCHRONOUS`    | 0 或 1                      | IN endpoint 0x81 是否是 isochronous 模式  |
| `EP82_ISOCHRONOUS`    | 0 或 1                      | IN endpoint 0x82 是否是 isochronous 模式  |
| `EP83_ISOCHRONOUS`    | 0 或 1                      | IN endpoint 0x83 是否是 isochronous 模式  |
| `EP84_ISOCHRONOUS`    | 0 或 1                      | IN endpoint 0x84 是否是 isochronous 模式  |
| `EP01_ISOCHRONOUS`    | 0 或 1                      | OUT endpoint 0x01 是否是 isochronous 模式 |
| `EP02_ISOCHRONOUS`    | 0 或 1                      | OUT endpoint 0x02 是否是 isochronous 模式 |
| `EP03_ISOCHRONOUS`    | 0 或 1                      | OUT endpoint 0x03 是否是 isochronous 模式 |
| `EP04_ISOCHRONOUS`    | 0 或 1                      | OUT endpoint 0x04 是否是 isochronous 模式 |
| `DEBUG`               | "TRUE" 或 "FALSE"           | 是否启用调试接口                          |

> :warning: 根据 USB 1.1 specification ，当一个 endpoint 是 isochronous 传输模式时，最大包大小可取 8\~1023 的任意值。当 endpoint 是 interrupt 或 bulk 传输模式时，最大包大小只能取 8, 16, 32, 或 64 。

### usbfs_core_top.v 的信号

#### 时钟与复位

需要给 `clk` 信号提供 60MHz 的时钟：

```verilog
// signals of usbfs_core_top.v
input  wire clk,           // 60MHz is required
```

复位信号 `rstn` 在正常工作时应该取高电平，如果需要停止工作，可以让 `rstn` 取低电平，此时如果 USB 插在 Host-PC 上，Host-PC 会检测到 USB 被拔出。

```verilog
// signals of usbfs_core_top.v
input  wire rstn,          // active-low reset, reset when rstn=0 (USB will unplug when reset)
```

#### USB 信号

以下3个信号需要引出到 FPGA 的引脚上，并按照 [电路连接方法](#circuit) 来连接到 USB 接口。

```verilog
// signals of usbfs_core_top.v
// USB signals
output reg  usb_dp_pull,   // connect to USB D+ by an 1.5k resistor
inout       usb_dp,        // USB D+
inout       usb_dn,        // USB D-
```

#### 用来指示USB是否被插入的信号

`usb_rstn` 信号指示了 USB 是否连接，高电平代表已连接；低电平代表未连接。未连接可能是有两种情况：要么USB线被从 Host 上拔出，要么 FPGA 侧主动进行复位（ `rstn=0` ）

```verilog
// signals of usbfs_core_top.v
output reg  usb_rstn,      // 1: connected , 0: disconnected (when USB cable unplug, or when system reset (rstn=0))
```

#### USB-transfer 和 USB-frame 检测信号

当以下两个信号 `sot` 和 `sof` 出现一周期的高电平时，分别指示了 USB-transfer 和 USB-frame 的开始。其中 USB-transfer 是指一个 USB transfer 的全过程，包括 control transfer, interrupt transfer, bulk transfer 和 isochronous transfer 。而 USB-frame 起始于 USB-host 每 1ms 会发送一次的 SOF token ，可以用来指导  isochronous transfer 。

```verilog
// signals of usbfs_core_top.v
output reg  sot,           // detect a start of USB-transfer
output reg  sof,           // detect a start of USB-frame
```

#### control transfer 响应信号

以下三个信号 `ep00_setup_cmd`, `ep00_resp_idx`, `ep00_resp` 提供了响应 control transfer 的接口。

```verilog
// signals of usbfs_core_top.v
// endpoint 0 (control endpoint) command response here
output wire [63:0] ep00_setup_cmd,
output wire [ 8:0] ep00_resp_idx,
input  wire [ 7:0] ep00_resp,
```

根据 USB specification ，control transfer 只在 control endpoint (0x00) 上进行，Host 会先发送 8 字节的 SETUP command ，根据 SETUP command ，device 可能响应数据给 Host 。根据 SETUP command 中的 `bmRequestType[6:5]` 字段，control transfer 可以分为三类：

- Standard control transfer (`bmRequestType[6:5]=0`)
- Class-specific control transfer (`bmRequestType[6:5]=1`)
- Vendor-specific control transfer (`bmRequestType[6:5]=2`)

其中 Standard control transfer 用来响应描述符等数据，在 **usbfs_core_top.v** 内部处理的，不需要开发者关心，开发者只需要用 parameter 指定好描述符即可。而 Class-specific control transfer 和 Vendor-specific control transfer 与具体设备的实现有关，开发者可以通过这三个信号响应它们。例如本库提供的 UVC 设备用它来响应 UVC Video Probe and Commit Controls ，而 HID 设备用它来响应 HID descriptor 。另外，有些简单的设备根本不需要 Class-specific control transfer 和 Vendor-specific control transfer ，则开发者可以无视这三个信号。

当一个 control transfer 进行时，`ep00_setup_cmd` 信号上先出现 8 字节的 SETUP command 。然后 `ep00_resp_idx` 信号从 0 开始递增，代表了当前需要响应第几个字节。开发者需要随时将响应数据的第  `ep00_resp_idx`  个字节打在 `ep00_resp` 信号上。

以 UVC 设备举例，以下代码检测 SETUP command 是否要求设备响应 UVC Video Probe and Commit Controls ，如果是，就响应 `UVC_PROBE_COMMIT` 数组中的第 `ep00_resp_idx`   字节打在  `ep00_resp` 信号上，否则就响应 `0x00` ：

```verilog
// 举例：在 UVC 设备中，当 host 请求 UVC Video Probe and Commit Controls 时，应该使用以下写法进行响应：
always @ (posedge clk)
    if(ep00_setup_cmd[7:0] == 8'hA1 && ep00_setup_cmd[47:16] == 32'h_0001_0100 )
        ep00_resp <= UVC_PROBE_COMMIT[ep00_resp_idx];
    else
        ep00_resp <= '0;
```

关于 control transfer 的命令和响应数据的内容，请参考具体的协议 specification 。

#### IN endpoint (0x81\~0x84) 数据输入信号

以下 4 组信号对应 4 个 IN endpoint ，用于发送 device-to-host 的 IN packet 。

例如，如果 FPGA 需要在 IN endpoint 0x81 上发送一个 IN packet ，需要：

- 首先，令 `ep81_valid=1`  并保持，同时在 `ep81_data` 上保持 packet 的第0个字节。
- `ep81_ready` 与 `ep81_valid` 构成握手信号。每当 `ep81_ready` 出现一个周期的高电平，说明一个字节被成功发送。
- 在 `ep81_ready=1` 的下一个周期，如果 IN packet 发送结束，需要令 `ep81_valid=0` 。如果 IN packet 还有字节每发完，保持  `ep81_valid=1`  ，并将  `ep81_data`  更新为新的待发送的字节。

```verilog
// signals of usbfs_core_top.v
// endpoint 0x81 data input (device-to-host)
input  wire [ 7:0] ep81_data,     // IN data byte
input  wire        ep81_valid,    // when device want to send a data byte, assert valid=1. the data byte will be sent successfully when valid=1 & ready=1.
output wire        ep81_ready,    // handshakes with valid. ready=1 indicates the data byte can be accept.
// endpoint 0x82 data input (device-to-host)
input  wire [ 7:0] ep82_data,     // IN data byte
input  wire        ep82_valid,    // when device want to send a data byte, assert valid=1. the data byte will be sent successfully when valid=1 & ready=1.
output wire        ep82_ready,    // handshakes with valid. ready=1 indicates the data byte can be accept.
// endpoint 0x83 data input (device-to-host)
input  wire [ 7:0] ep83_data,     // IN data byte
input  wire        ep83_valid,    // when device want to send a data byte, assert valid=1. the data byte will be sent successfully when valid=1 & ready=1.
output wire        ep83_ready,    // handshakes with valid. ready=1 indicates the data byte can be accept.
// endpoint 0x84 data input (device-to-host)
input  wire [ 7:0] ep84_data,     // IN data byte
input  wire        ep84_valid,    // when device want to send a data byte, assert valid=1. the data byte will be sent successfully when valid=1 & ready=1.
output wire        ep84_ready,    // handshakes with valid. ready=1 indicates the data byte can be accept.
```

#### OUT endpoint (0x01\~0x04) 数据输出信号

以下 4 组信号对应 4 个 OUT endpoint ，用于接收 host-to-device 的 OUT packet。

例如，当 `ep01_valid` 出现一个周期的高电平时，说明收到了 OUT packet 中的一个字节，该字节出现在 `ep01_data` 上。另外，packet 的边界可以用之前讲过的 `sot` 信号来检测。

```verilog
// signals of usbfs_core_top.v
// endpoint 0x84 data input (device-to-host)
input  wire [ 7:0] ep84_data,     // IN data byte
input  wire        ep84_valid,    // when device want to send a data byte, assert valid=1. the data byte will be sent successfully when valid=1 & ready=1.
output wire        ep84_ready,    // handshakes with valid. ready=1 indicates the data byte can be accept.
// endpoint 0x01 data output (host-to-device)
output wire [ 7:0] ep01_data,     // OUT data byte
output wire        ep01_valid,    // when out_valid=1 pulses, a data byte is received on out_data
// endpoint 0x02 data output (host-to-device)
output wire [ 7:0] ep02_data,     // OUT data byte
output wire        ep02_valid,    // when out_valid=1 pulses, a data byte is received on out_data
// endpoint 0x03 data output (host-to-device)
output wire [ 7:0] ep03_data,     // OUT data byte
output wire        ep03_valid,    // when out_valid=1 pulses, a data byte is received on out_data
// endpoint 0x04 data output (host-to-device)
output wire [ 7:0] ep04_data,     // OUT data byte
output wire        ep04_valid,    // when out_valid=1 pulses, a data byte is received on out_data
```

#### 调试输出接口

要使用调试接口，需要把模块参数 `DEBUG` 设为 1，否则调试接口不会有任何输出。

以下信号用来向外界打印调试信息。调试信息是一个 ASCII 码字节流，是人类可读的。当 `debug_en=1`  时， `debug_data` 上出现一个字节。

```verilog
// signals of usbfs_core_top.v
// debug output info, only for USB developers, can be ignored for normally use
output wire        debug_en,      // when debug_en=1 pulses, a byte of debug info appears on debug_data
output wire [ 7:0] debug_data,    // 
output wire        debug_uart_tx  // debug_uart_tx is the signal after converting {debug_en,debug_data} to UART (format: 115200,8,n,1). If you want to transmit debug info via UART, you can use this signal. If you want to transmit debug info via other custom protocols, please ignore this signal and use {debug_en,debug_data}.
```

为了方便使用，我还将 `{debug_en,debug_data}` 字节流转化位 UART 输出信号 `debug_uart_tx` ，可以将 `debug_uart_tx` 连接到电脑的 UART 上，用串口助手等软件来查看调试信息。

注意： UART 的配置应该为 波特率=115200, 数据位=8，无校验位，停止位=1 。

下图展示了我的 USB UVC 设备插入电脑时 UART 上打印的调试数据。可以看到这是一个描述符枚举的过程。

|           ![debug_info](./figures/debug.png)           |
| :----------------------------------------------------: |
| **图**：USB UVC 设备插入电脑时 UART 上打印的调试数据。 |

> :warning: 因为 UART 的传输速度较慢，当大量的调试信息产生时，会有一部分调试信息被 UART 丢弃。例如当 UVC 进行视频传输时，因为传输的数据量很大，UART 打印的信息是不完整的。因此如果需要在大量数据通信的过程中查看调试信息，就不能用 UART ，而是用其它高速的通信方式将  `{debug_en, debug_data}` 发给 host-PC 来查看。

 　

　

# 参考资料

* https://www.usbmadesimple.co.uk/ : USB Made Simple
* https://github.com/FengJungle/USB_Protocol_CH : USB 中文协议手册
* https://www.usb.org/document-library/usb-20-specification : USB 2.0 Specification
* https://www.usb.org/document-library/video-class-v11-document-set : USB Video Class (UVC) 1.1 Specification
* https://www.usb.org/document-library/audio-device-class-spec-basic-audio-devices-v10-and-adopters-agreement : USB Audio Class (UAC) 1.0 Specification
* https://www.usb.org/document-library/mass-storage-class-specification-overview-14 : USB Mass Storage Class (USB-MSC) 1.4 Specification
* https://www.usb.org/document-library/class-definitions-communication-devices-12 : USB Communication Device Class (USB-CDC) 1.2 Specification
* 其它 FPGA 实现：
  * https://github.com/avakar/usbcorev : 一个 USB-device 控制器，仅支持到了 transaction 层。
  * http://jorisvr.nl/article/usb-serial : 一个 USB-CDC，VHDL实现，需要额外的 UTMI PHY。
  * https://github.com/pbing/USB : 一个 Low Speed 的 USB-HID 实现。
  * https://github.com/ultraembedded/cores : 包含一些 USB-host 和 USB-device 实现，需要 UTMI PHY 或 ULPI PHY。



