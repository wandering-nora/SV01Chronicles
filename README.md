# SV01Chronicles
This is a logbook for my SV01 printer thinkering.
If you somehow stumble upon this feel free to look around and reach out.

> Warning: I am a beginner and I definitely make mistakes. I take no responsibility.

The Sovol SV01 is a nice little machine, but the time has come to give it a glow up with Klipper to increase print quality and speed.
My machine is stock apart from a couple upgrades:

### All metal hot end
I installed a TriangleLabs V6 plated copper hot end following [this](https://youtu.be/IrHVTM04Ivc?si=vAyz9BuilHrvBVQR) great tutorial, which now allows me to print up to 290C. (or 500C with a thermistor swap according to the manufacturer)

### Auto bed leveling
I installed a cr touch, which was the best cheap upgrade I did.

### New firmware
I upgraded the printer to Marlin 2.0 building it from [coptertec's](https://www.coptertec.de/blogs/news/marlin-2-0-for-sovol-sv01) source. This is irrelevant for Klipper since it will be replaced.

## Klipper
I am installing klipper on a raspberry pi 3B+. I decided to go with Mainsail, but everything else should work for Fluidd and Octoprint as well.

### Config file
There is a config file available for the SV01 directly in the main repository [here](https://raw.githubusercontent.com/Klipper3d/klipper/refs/heads/master/config/printer-sovol-sv01-2020.cfg) which I used as the starting point.

### Kiauh
Mainsail offers a premade image but I decided to use Kiauh to instal it manually so I can more easily switch to another interface using the same installation procedure.

Setup Raspberry Pi OS Lite on the pi using RPI imager enabling wifi and ssh. Then ssh into it and run
```bash
sudo apt-get update && sudo apt-get install git -y
```
```bash
cd ~ && git clone https://github.com/dw-0/kiauh.git
```
```bash
./kiauh/kiauh.sh
```
Now follow the instructions and install Klipper, Moonraker and Mainsail.

### Flashing the firmware
Now it's time to flash the firmware so connect the pi to the printer with a mini USB ✨ cable then run:
```bash
cd ~/klipper/
make menuconfig
```
Select Atmega AVR and atmega2560 then press Q and Y.
```bash
make
```
Now determine the serial port with
```bash
ls /dev/serial/by-id/*
```
and flash the firmware with
```bash
sudo service klipper stop
make flash FLASH_DEVICE=<serial port>
sudo service klipper start
```
After a power cycle the screen is now blank and the pi is ready to do all the work.

### Klipper config
Copy [printer.cfg](https://github.com/wandering-nora/SV01Chronicles/blob/main/printer.cfg) to ~/printer_data/config/printer.cfg.  
Now access Mainsail in a browser by entering your raspberry pi ip, if it complains about missing printer.cfg restart it.

Now run
```bash
ls /dev/serial/by-id/*
```
and update printer.cfg file
```
[mcu]
serial: <id found>
```
After verifying [basic functionality](https://www.klipper3d.org/Config_checks.html)  pid tune the extruder and bed heater by typing these commands in the console
```
PID_CALIBRATE HEATER=extruder TARGET=170
SAVE_CONFIG
PID_CALIBRATE HEATER=heater_bed TARGET=60
SAVE_CONFIG
```

### Bed leveling
Calibrate the probe offset by running
```
PROBE
GET_POSITION
```
mark on the build plate the probe's position and moving the nozzle over it

```
GET_POSITION
```
subtract the X and Y coordinates of the first from the second and apply it to
```
[bltouch]
x_offset: <x nozzle - x probe>
y_offset: <y nozzle - y probe>
```


