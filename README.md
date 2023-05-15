# klipper.veho-600

CURRENTLY ROUGHING IN STEPS - BELOW ARE TO BE FIXED UP LATER

Install the Pi with 32 bit os lite. I used rpi-imager (for the first time) and also setup ssh access without passwords (using my rsa-id.pub).
Following the giode from [here](https://docs-os.mainsail.xyz/getting-started/raspberry-pi-os-based)
I installed mainsail from the os-other 3d printing options. I also setup my wifi and changed my hostname to :
```
veho600.local
```

Go to http://veho600.local and then the machine link give it some time to refresh then upgrade what you can by clicking the upgrade buttons.

I route myself to  : sftp://pi@veho600.local/home/pi/printer_data/config
Dump all the cfg files to that location.

# setup printer config

Copy the config files over :
```
cd klipper.veho-600/config
scp *  pi@veho600.local:printer_data/config/
```

Now in the webapp, go to "MACHINE" and restart klipper.

# install the fw

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
