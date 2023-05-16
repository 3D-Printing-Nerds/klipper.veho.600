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

# for the developers
The reference marlin code is available here : https://github.com/tronxy3d/F4xx-SIM480x320

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
* USB (on PA11/PA12)
You're now ready to build. Press 'Q' and 'Y' to exit and save. Now :
```
make
```
