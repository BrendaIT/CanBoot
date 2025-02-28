# CanBoot
 Bootloader for STM32F0/1 MCUs

 This bootloader is designed for CAN nodes to be used with
 [Klipper](https://github.com/Klipper3d/klipper).  The bootloader
 itself makes use of Klipper's hardware abstraction layer, stripped
 down to keep the footprint minimal.  Currently the bootloader
 reserves 8 KiB of space.

In addition to CANBUS, CanBoot now supports USB and UART interfaces.

Currently STM32F103, STM32F042, and STM32F072 boards are supported.

## Building

CanBoot also uses Klipper's build system.  The build is configured
with menuconfig.  The steps to fetch and build are as follows:
```
git clone https://github.com/Arksine/CanBoot
cd CanBoot
make menuconfig
make
```

The menuconfig will present the following options:
- `Processor model`: Currently STM32F042, STMF072, and STM32F103 models are
 supported.
- `Disable SWD at startup`:  This is an option for GigaDevice STM32F103
  clones.  Note that this is untested for the bootloader, GigaDevice clones
  may not work as expected.
- `Clock Reference`: Choose the appropriate setting for your board's crystal
- `Communication interface`:  Choose between CAN, USB, and UART.  Be sure
  to choose the interface with the appropriate pins for your hardware.
- For CAN Interfaces:
  - `CAN bus speed`: Select the appropriate speed for your canbus.
- For Serial (USART) Interfaces:
  - `Baud rate for serial port`:  Select the appropriate baud rate for your
    serial device.
- For USB Interfaces:
  - `USB ids`:  Allows the user to define the USB Vendor ID, Product ID,
    and/or Serial Number.
- `Support bootloader entry on rapid double click of reset button`:  When
  enabled it is possible to enter the bootloader by pressing the reset button
  twice within a 500ms window.
- `Enable bootloader entry on button (or gpio) state`:  Enable to use a gpio
  to enter the booloader.
  - `Button GPIO Pin`:  The Pin Name of te
- `Enable Status Led`: Enables the option to select a status LED gpio.
  - `Status LED GPIO Pin`:  The pin name for your LED.  The Pin can be inverted
    if the LED is on when the pin is low.  For example, the status LED Pin for a
    "blue pill" is !PC13.

Once configured and built flash with a programmer such as an ST-Link.  If you
don't have a programmer available, it should be possible to flash STM32F103
devices via UART and STM32F042/72 devices over DFU.  ST's STM32CubeProgrammer
software can facilitate all of these methods, however there are also other
tools such as `stm32flash` (UART) and `dfu-util` (USB DFU).

NOTE:  Prior to flashing CanBoot it is recommended to do a full chip erase.
Doing so allows CanBoot to detect that no application is present and enter
the bootloader.  This is required to enter the bootloader if you have not
configured an alternative method of entry.

## Uploading Klipper
1) Make sure the `klipper` service stopped.
2) Build Klipper with CAN support and with the "8KiB" bootloader setting enabled.
3) Enter the bootloader.  This will occur automatically if no program is detected.
   If you built CanBoot with an alternative method of entry you may use that.
   If upgrading from a currently flashed version of Klipper the `flash_can.py`
   script will command the device to enter the bootloader (currently for CAN
   devices only).
3) Run the flash script:
   For CAN Devices:
   ```
   cd ~/CanBoot
   python3 flash_can.py -i can0 -f ~/klipper/out/klipper.bin -u <uuid>
   ```
   Replace <uuid> with the appropriate uuid for your can device.  If
   the device has not been previouisly flashed with Klipper, it is possible
   to query the bootloader for the UUID:

   ```
   flash_can.py -i can0 -q
   ```

   For USB/UART devices:
   Before flashing, make sure pyserial is installed.  This step only needs to
   be performed once:
   ```
   pip install serial
   ```
   ```
   python3 flash_can.py -d <serial device> -b <baud_rate>
   ```
   Replace `<serial_device>` the the path to the USB/UART device you wish to
   flash.  The `<baud_rate>` is only necessary for UART devices, and defaults
   to 250000 baud if omitted.

## FlashCan usage

Running `flash_can.py -h` to display help:

```
usage: flash_can.py [-h] [-d <serial device>] [-b <baud rate>]
                    [-i <can interface>] [-f <klipper.bin>] [-u <uuid>] [-q]
                    [-v]

Can Bootloader Flash Utility

optional arguments:
  -h, --help            show this help message and exit
  -d <serial device>, --device <serial device>
                        Serial Device
  -b <baud rate>, --baud <baud rate>
                        Serial baud rate
  -i <can interface>, --interface <can interface>
                        Can Interface
  -f <klipper.bin>, --firmware <klipper.bin>
                        Path to Klipper firmware file
  -u <uuid>, --uuid <uuid>
                        Can device uuid
  -q, --query           Query Bootloader Device IDs
  -v, --verbose         Enable verbose responses
```

The `interface` option defaults to `can0` if omitted.  The `firmware` option
defaults to `~/klipper/out/klipper.bin`.  The `uuid` must be specified unless
the user is running a query with `-q`.

## Notes
- It is recommended to USB CAN device flashed with
  [candlelight](https://github.com/candle-usb/candleLight_fw), such as a
  [Canable](https://canable.io/).
- If using a MCP2515 CAN Device (ie: Waveshare RS485 CAN HAT) it is possible
  that packets will be dropped when reading flash back from the node during
  the verification process.  That said, I have successfully tested the 12 MHz
  Crystal variant with the
  [recommended settings](https://www.waveshare.com/wiki/RS485_CAN_HAT).
- Details on the protocol used to communicate with the bootloader may
  be found in [protocol.md](protocol.md).
