# InforLink nRF52840 Dongle PA

Collected here is information on a **InforLink nRF52840 Dongle PA** that can still be purchased, but for which there is no support due to the fact that the company's website www.infor-link.com is not available.

![image](https://user-images.githubusercontent.com/56395503/147378102-c36ce858-9c77-4515-b43a-55a189e1393f.png)

## Hardware

This dongle uses **GT840B_02** / **GT832B_02** modules based on the nRF52840 chip with **RFX2401** PA + LNA IC which is activated via GPIO 

## Pinout

![image](https://user-images.githubusercontent.com/56395503/147380177-191fba9e-a21e-4404-8e79-ffd2967b6150.png)

![image](https://user-images.githubusercontent.com/56395503/147378340-e7fd49c8-9ab5-41fa-ae42-2019035ba855.png)

|       |       |    | ============= |    |         |                 |
| ----- | ----- | -- |-------------- | -- | ------- | --------------- |
| DEBUG | GND   |    |               |    |         |                 |
| DEBUG | VCC   |    |               |    |         |                 |
| DEBUG | DIO   |    |               |    |         |                 |
| DEBUG | CLK   |    |               |    |         |                 |
|       |       |    |               |    |         |                 | 
|       | GND   |    |               |    | GND     |                 |
|       | P0.10 | 10 |               | 42 | P1.10   |                 | 
|       | P0.09 |  2 |               | 45 | P1.13   |                 | 
|       | P1.00 | 32 |               | 47 | P1.15   |                 | 
|       | P0.24 | 24 |               |  2 | P0.02   |                 | 
|       | P0.22 | 22 |               | 29 | P0.29   | A0              | 
|       | P0.20 | 20 |               | 31 | P0.31   | A1              | 
|       | P0.17 | 17 |               |    | GND     |                 | 
|       | P0.15 | 15 |               |    | VDD_OUT |                 | 
|       | P0.13 | 13 |               |    | GND     |                 | 
|       |       |    |               |    |         |                 | 
|       |       |    |               | 43 | P1.11   | Enable PA       | 
|       |       |    |               | 44 | P1.12   | Enable LNA      | 
|       |       |    |               |    |         |                 |
|       |       |    |               | 41 | P1.09   | LED_GREEN (RGB) |
|       |       |    |               |  8 | P0.08   | LED_RED   (RGB) |
|       |       |    |               | 12 | P0.12   | LED_BLUE  (RGB) |
|       |       |    |               |  6 | P0.06   | LED_BUILTIN     |
|       |       |    |               | 38 | P1.06   | SW2_BUTTON      |
|       |       |    |               | 18 | P0.18   | SW3_BUTTON      |
|       |       |    |               |    |         |                 |
|       |       |    |               |    |         |                 |
## Circuit
![image](https://user-images.githubusercontent.com/56395503/147378648-b0c026ba-8fb7-42f2-b0d2-2ef575e7f37f.png)

## Connecting to the programmer

| nrf52840 Dongle PA | ST-Link V2	| J-Link (clone) |
| -------- | ---------- | -------------- |
| GND	     | (3) GND    | (4) GND        |
| VDD      | (7) 3.3V   | (2) 3.3V       |
| DIO 	   | (2) SWDIO  | (7) SWDIO      |
| CLK      | (6) SWCLK  | (9) SWCLK      |

Flashing with ST-Link V2
```
└─$ openocd -f interface/stlink.cfg -f target/nrf52.cfg -c "init; halt; nrf5 mass_erase; program ../nrf_sniffer_for_bluetooth_le_4.1.0/hex/sniffer_nrf52840dongle_nrf52840_4.1.0.hex verify reset exit"
```
Flashing with J-Link
```
└─$ JLinkExe -device nRF52 -if SWD -speed 4000 -autoconnect 1
J-Link>erase
J-Link>r
J-Link>loadfile nrf_sniffer_for_bluetooth_le_4.1.0/hex/sniffer_nrf52840dongle_nrf52840_4.1.0.hex
J-Link>r
```
### Enable APPROTECT
```
└─$ JLinkExe -device nRF52 -if SWD -speed 4000 -autoconnect 1
J-Link>Mem 0x10001208, 4
10001208 = FF FF FF FF                                       .…
J-Link>Write4 0x10001208, 0xFFFFFF00
Writing FFFFFF00 -> 10001208
J-Link>Mem 0x10001208, 4
10001208 = 00 FF FF FF
J-Link>r
```
### Disable APPROTECT

![image](https://user-images.githubusercontent.com/56395503/147379889-6cf95273-9aef-4e01-98e7-d54c49183d3c.png)

_memory will be erased_
```
J-Link>unlock Kinetis
Found SWD-DP with ID 0x2BA01477
Unlocking device...RESET (pin 15) high, but should be low. Please check target hardware.
Timeout while unlocking device.
J-Link>r
```
### APPROTECT bypass
- https://limitedresults.com/2020/06/nrf52-debug-resurrection-approtect-bypass
- https://limitedresults.com/2020/06/nrf52-debug-resurrection-approtect-bypass-part-2

| Base address | Peripheral | Instance | Description                    |
| ------------ | ---------- | -------- | ------------------------------ |
| 0x10001000   | UICR       | UICR     | User information configuration |

| Register  | Offset | Description            |
| --------- | ------ | ---------------------- |
| APPROTECT | 0x208  | Access port protection |

| Description | Documentation   |
| -- | -- |
| Access port protection blocks the debugger from read and write access to all CPU registers and memory-mapped addresses. See the UICR register APPROTECT on page 45 for more information on enabling access port protection. | ![image](https://user-images.githubusercontent.com/56395503/147380010-766bf570-192e-46d0-9669-c9dfa3e03729.png) |

0x10001000 + 0x208 = 0x10001208

```
J-Link>?
Mem              Mem  [<Zone>:]<Addr>, <NumBytes> (hex)              Read memory and show corresponding ASCII values.
Mem8             Mem8  [<Zone>:]<Addr>, <NumBytes> (hex)             Read  8-bit items.
Mem16            Mem16 [<Zone>:]<Addr>, <NumItems> (hex)             Read 16-bit items.
Mem32            Mem32 [<Zone>:]<Addr>, <NumItems> (hex)             Read 32-bit items.
Write1           W1 [<Zone>:]<Addr>, <Data> (hex)                    Write  8-bit items.
Write2           W2 [<Zone>:]<Addr>, <Data> (hex)                    Write 16-bit items.
Write4           W4 [<Zone>:]<Addr>, <Data> (hex)                    Write 32-bit items.
J-Link>Mem 0x10001208, 4
10001208 = FF FF FF FF 
└─$ echo "obase=2; ibase=16; FFFFFFFF" | bc
11111111111111111111111111111111
```

## nRF Sniffer for Bluetooth LE
- https://www.nordicsemi.com/Products/Development-tools/nrf-sniffer-for-bluetooth-le/download
- https://infocenter.nordicsemi.com/pdf/nRF_Sniffer_BLE_UG_v4.0.0.pdf
```
└─$ wget https://nsscprodmedia.blob.core.windows.net/prod/software-and-other-downloads/desktop-software/nrf-sniffer/sw/nrf_sniffer_for_bluetooth_le_4.1.0.zip
└─$ mkdir nrf_sniffer_for_bluetooth_le_4.1.0
└─$ 7z x nrf_sniffer_for_bluetooth_le_4.1.0.zip -onrf_sniffer_for_bluetooth_le_4.1.0
└─$ openocd -f interface/stlink.cfg -f target/nrf52.cfg -c "init; halt; nrf5 mass_erase; program ./nrf_sniffer_for_bluetooth_le_4.1.0/hex/sniffer_nrf52840dongle_nrf52840_4.1.0.hex verify reset exit"
└─$ cd nrf_sniffer_for_bluetooth_le_4.1.0/extcap/
└─$ sudo /usr/bin/python3 -m pip install -r requirements.txt
└─$ sudo /usr/bin/python3 -m pip install psutil
└─$ sudo cp -r * /root/.config/wireshark/extcap/
└─$ cd ..
└─$ sudo cp -r Profile_nRF_Sniffer_Bluetooth_LE /root/.config/wireshark/profiles/
└─$ sudo wireshark
```
After flashing, you will need to activate the low-noise amplifier by supplying 3.3V to the P1.12.

```
└─$ dmesg
[ 8273.248538] usb 1-4: new full-speed USB device number 19 using xhci_hcd
[ 8273.400559] usb 1-4: New USB device found, idVendor=1915, idProduct=522a, bcdDevice= 2.04
[ 8273.400570] usb 1-4: New USB device strings: Mfr=1, Product=2, SerialNumber=3
[ 8273.400577] usb 1-4: Product: nRF Sniffer for Bluetooth LE
[ 8273.400581] usb 1-4: Manufacturer: ZEPHYR
[ 8273.400586] usb 1-4: SerialNumber: A9F59A9D07C288C4
[ 8273.403121] cdc_acm 1-4:1.0: ttyACM0: USB ACM device
```
## SweynTooth
- https://asset-group.github.io/disclosures/sweyntooth/
```
└─$ git clone https://github.com/Matheus-Garbelini/sweyntooth_bluetooth_low_energy_attacks
J-Link>loadfile ./sweyntooth_bluetooth_low_energy_attacks/s140_nrf52_6.1.1_softdevice.hex
J-Link>loadfile ./sweyntooth_bluetooth_low_energy_attacks/nRF52_driver_firmware.hex
[31636.826209] usb 1-4: new full-speed USB device number 65 using xhci_hcd
[31636.994099] usb 1-4: New USB device found, idVendor=239a, idProduct=8029, bcdDevice= 1.00
[31636.994109] usb 1-4: New USB device strings: Mfr=1, Product=2, SerialNumber=3
[31636.994116] usb 1-4: Product: Bluefruit nRF52840
[31636.994121] usb 1-4: Manufacturer: Adafruit Industries
[31636.994126] usb 1-4: SerialNumber: A9F59A9D07C288C4
[31636.999242] cdc_acm 1-4:1.0: ttyACM0: USB ACM device
└─$ sudo pip install -r requirements.txt
└─$ sudo ./install_sweyntooth.sh
```
## ArduinoIDE Support
File => Preferences => Additional Boards Manager URLs:
`https://adafruit.github.io/arduino-board-index/package_adafruit_index.json`
Tools => Board: => Boards Manageg => Adafruit nRF52 (1.2.0) => Install
Tools => Board: => Adafruit nRF52 Boards => InforLink nRF52840 Dongle PA

### Useful links:
- https://github.com/lyusupov/SoftRF/tree/master/software/firmware/source#nrf52840
- https://github.com/adafruit/Adafruit_nRF52_Arduino#recommended-adafruit-nrf52-bsp-via-the-arduino-board-manager
- https://learn.adafruit.com/bluefruit-nrf52-feather-learning-guide/using-the-bootloader?view=all
- http://blog.bachi.net/?p=4748
- https://flaviutamas.com/2020/notes-getting-started-nrf52-thread
- https://github.com/makerdiary/nrf52840-mdk-usb-dongle/issues/51
- https://openthread.io/platforms/co-processor/firmware
- https://github.com/openthread/ot-nrf528xx/blob/main/src/nrf52840/README.md
- https://russianblogs.com/article/3372691573/
- https://diyiot.wordpress.com/2016/02/07/ble-application-with-nrf51822-remotely-control-a-led/
- https://diyiot.wordpress.com/2015/11/29/ble-application-with-nrf51822-firmware-flashing/
- https://github.com/openweave/openweave-nrf52840-lock-example/blob/master/start-jlink-gdb-server.sh
- https://www.diytronic.ru/2017/12/05/testing-nrf51822-ble-module/
- https://infocenter.nordicsemi.com/pdf/nRF_Sniffer_UG_v2.2.pdf
- https://infocenter.nordicsemi.com/topic/ug_sniffer_ble/UG/sniffer_ble/intro.html?cp=10_5
- https://infocenter.nordicsemi.com/topic/ug_sniffer_802154/UG/sniffer_802154/intro_802154.html?cp=10_4
- https://github.com/NordicSemiconductor/nRF-Sniffer-for-802.15.4
- https://jimmywongiot.com/2020/07/08/how-to-install-nrf-sniffer-through-nrf52-dk-board/
- https://next-hack.com/index.php/2021/11/13/porting-doom-to-an-nrf52840-based-usb-bluetooth-le-dongle/
- 
