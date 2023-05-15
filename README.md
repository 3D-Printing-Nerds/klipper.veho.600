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

