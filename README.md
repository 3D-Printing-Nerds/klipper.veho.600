# klipper.veho-600

# Installation on the embedded system
In this example a Raspberry Pi will be used, however you can use other embedded systems which are supported by mainsail.

Install the Pi with 32 bit os lite. I used rpi-imager (for the first time) and also setup ssh access without passwords (using my rsa-id.pub). Setup your wifi if you need it.

Following the guide from [here](https://docs-os.mainsail.xyz/getting-started/raspberry-pi-os-based)
I installed mainsail from the os-other 3d printing options. I also setup my wifi and changed my hostname to :
```
veho600.local
```

Go to http://veho600.local and then the machine link give it some time to refresh then upgrade what you can by clicking the upgrade buttons.

# Setup the printer config files

Copy the config files over :
```
cd klipper.veho-600/config
scp *  pi@veho600.local:printer_data/config/
```

Now in the webapp, go to "MACHINE" and restart klipper.

# Install the firmware

Copy the update directory to an sdcard. Turn off the printer power, put in the sdcard and turn it on again. The boot splash shows and it reports "Burn: FNW TR 1.BIN" or somethign similar in the lower left hand corner.
Turn off the printer and remove the sdcard. Turn the printer on again. Now it should reports "Initializing..." and stays that way.

Reload your webapp and you should see the mcu reporting something like the following :
```
mcu(stm32f446xx)
Version: v0.11.0-197-ga3eebab4
Load: 0.00, Awake: 0.00, Freq: 180 MHz,
```

# PID tuning

In the console on the dashboard run the follwing - one at a time :
```
PID_CALIBRATE HEATER=heater_bed TARGET=85
PID_CALIBRATE HEATER=extruder TARGET=210
SAVE_CONFIG
```

Home axis :
```
G28
```
Perform an probe calibration. Raise the bed with a piece of A4 paper between the bed and the nozzle until the paper is a little resistive and then accept.
```
PROBE_CALIBRATE
SAVE_CONFIG
```
Home again :
```
G28
```
At this point you can carefully raise the bed to say 70 in the Z direction to save a little time.
Now z stop calibration :
```
Z_ENDSTOP_CALIBRATE
SAVE_CONFIG
```
# rotation distance

Use the webapp interface to specify your extruder heat - say 200 degrees C for PLA.
From a known edge point, mark out 120 mm on the pla line. Now extrude 100mm of filament (using the UI or the following terminal entry) :
```
M91
G1 E100 F60
```
I noted that after extrusion 17mm was left from 120mm. Got to this link : https://www.service-uplink.de/esteps_cal/calculator.php
I would put in "actual extrude distance" of 103, "requested extrude distance" of 100. Looking in printer.cfg, I can see that the original rotation distance was 22.172 mm. The calculator tells me to put 22.837 mm as the new rotation distamnce.

# Slicer settings

## Cura

Add a new Custom FFF printer. Set X, Y, Z sizes to 600 mm. Select "Heated Bed".

Copy the slicer.cura/Start__End_Code.txt start code to the "Start G-Code" box. Copy the end code to the "End G-Code" box.

In the extruder 1 tab, set the nozzle size (veho-600 comes with stock 0.6 mm and a 0.8 mm in the bag). Set the compatible material diameter to 1.75 mm.

Click "Profiles" and "Import Profiles".

### Changing nozzle sizes

* Manage printers -> machine settings -> Extruder 1 -> Nozzle Size = 0.8 mm

# Calibrating and problem solving 
## Finding the serial device
This was written for a Linux system, but may apply to Mac systems as well.

On this system, the serial device is connected with USB. The standard mac and Linux commands to look at the usb devices is lsusb. When the device is plugged in it will appear on the system in the path /dev/ttyUSB* where * is the number. If I long list all ttyUSB* devices on my system I get the one serial USB device which is on my printer (using the ```ls -l /dev/ttyUSB*``` command) and this is what it looks like :
```
pi@veho600:~ $ ls -l /dev/ttyUSB*
crw-rw---- 1 root dialout 188, 0 May 23 22:14 /dev/ttyUSB0
```
I can see that my serial device is ttyUSB0 on the path : /dev/ttyUSB0. On modern Linux the USB serial devices are linked by name and by id in the /dev/serial directory. I can see them by listing (again using the ```ls /dev/serial/*``` command to see subdirectories of /dev/serial) :
```
pi@veho600:~ $ ls /dev/serial/*
/dev/serial/by-id:
usb-1a86_USB_Serial-if00-port0

/dev/serial/by-path:
platform-3f980000.usb-usb-0:1.2:1.0-port0
```
But if I look with the long list, I can see that these devices point to /dev/ttyUSB0 :
```
pi@veho600:~ $ ls -l /dev/serial/*
/dev/serial/by-id:
total 0
lrwxrwxrwx 1 root root 13 May 23 22:12 usb-1a86_USB_Serial-if00-port0 -> ../../ttyUSB0

/dev/serial/by-path:
total 0
lrwxrwxrwx 1 root root 13 May 23 22:12 platform-3f980000.usb-usb-0:1.2:1.0-port0 -> ../../ttyUSB0
```
### Working out if the 3d printer device is present
Apart from using the above commands to see if the device is present, I can use the lsusb command to see what USB devices are on the system. First I will look at the devices with the USB unplugged :
```
pi@veho600:~ $ lsusb
Bus 001 Device 003: ID 0424:ec00 Microchip Technology, Inc. (formerly SMSC) SMSC9512/9514 Fast Ethernet Adapter
Bus 001 Device 002: ID 0424:9514 Microchip Technology, Inc. (formerly SMSC) SMC9514 Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```
Now I want to see whether the printer device registers once it is plugged in. I plug in the printer and lsusb again :
```
pi@veho600:~ $ lsusb
Bus 001 Device 005: ID 1a86:7523 QinHeng Electronics CH340 serial converter
Bus 001 Device 003: ID 0424:ec00 Microchip Technology, Inc. (formerly SMSC) SMSC9512/9514 Fast Ethernet Adapter
Bus 001 Device 002: ID 0424:9514 Microchip Technology, Inc. (formerly SMSC) SMC9514 Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```
The difference in output is the first line ```Bus 001 Device 005: ID 1a86:7523 QinHeng Electronics CH340 serial converter``` and now I know that the printer serial device is registering with the system.

### USB connections, looking at the Linux kenrel log (the kernel ring buffer) and intermittent connections

To see a log of when the usb device plugs in and out we can print out the kernel ring buffer. This can be helpful to look for intermittent connections. Start by clearing out the ring buffer before you plug in the USB device using the command ```dmesg -c``` where "-c" stands for clear history and you may have to add the ```sudo``` command to invoke super user permissions :
```
dmesg -c
```
If I unplug the USB device it looks like so :
```
pi@veho600:~ $ sudo dmesg -c
[  550.279869] ch341-uart ttyUSB0: usb_serial_generic_read_bulk_callback - urb stopped: -32
[  550.279989] ch341-uart ttyUSB0: usb_serial_generic_read_bulk_callback - urb stopped: -32
[  550.333153] usb 1-1.2: USB disconnect, device number 4
[  550.335337] ch341-uart ttyUSB0: ch341-uart converter now disconnected from ttyUSB0
[  550.335451] ch341 1-1.2:1.0: device disconnected
```
The ```ttyUSB0``` device was disconnected from the system. Whenever the serial USB device disconnects it will print out this message in the kernel ring buffer. As I used the "-c" flag this log was cleared form the ring buffer.

Now I plug in my USB device and it looks like this :
```
pi@veho600:~ $ sudo dmesg -c
[  616.416522] usb 1-1.2: new full-speed USB device number 5 using dwc_otg
[  616.549636] usb 1-1.2: New USB device found, idVendor=1a86, idProduct=7523, bcdDevice= 2.64
[  616.549675] usb 1-1.2: New USB device strings: Mfr=0, Product=2, SerialNumber=0
[  616.549691] usb 1-1.2: Product: USB Serial
[  616.550703] ch341 1-1.2:1.0: ch341-uart converter detected
[  616.553033] usb 1-1.2: ch341-uart converter now attached to ttyUSB1
```
I can see that the ch341 uart USB device was plugged in and attached to the device ```/dev/ttyUSB1```.

If you have intermittent USB connections, you will see these connect and disconnection messages in the kernel ring buffer using the ```dmesg``` command.

## Getting your first layer right

Get a standard cube from the cura "calibration shapes" plugin. Make the following properties :
* scaling x, y, z = 300%, 300%, 1.5%
* initial layer height of 0.3

# for the developers
The reference the original marlin code is available here : https://github.com/tronxy3d/F4xx-SIM480x320
We will build the firmware with information from this guide for the F446 (V10 and V6-191121) motherboards - assuming this is installed on your system :
https://tronxy.fandom.com/wiki/Installing_Klipper#Installing_Klipper

## Building the firmware
```
ssh pi@veho600.local
cd klipper
make menuconfig
```
Using the menu, select the following :
* STMicroelectronics STM32
* STM32F446
* 64KiB bootloader
* 8 MHz crystal
* USB (on PA10/PA9)
* 2500000 baud rate
You're now ready to build. Press 'Q' and 'Y' to exit and save. Now :
```
make
```
