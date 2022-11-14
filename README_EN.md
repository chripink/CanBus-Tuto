# CanBus-Tutorial-ENGLISH
Installation of a Can Bus PCB on the print head  

[Cliquez ici pour la version Française](https://github.com/chripink/CanBus-Tuto/blob/main/README.md)

## Link to buy the material    
[BTT EBB and U2C boards](https://ali.ski/fYbp4)  
[CRIMP pliers if you don't have one](https://ali.ski/n02iB)  

### Additional material  
[Cutting pliers](https://ali.ski/M58mv)  
[ATC Semitec 104GT-2 Thermistor](https://ali.ski/NlTzoU)  
[50W Heating cartridge](https://ali.ski/YAYFnu)  

# Advantages and disadvantages  
If you've ever set up a printer or changed the wiring, you know how time consuming it can be. Plus, this is not the most funny part...  
If you make a modification to your printhead (change the hotend, add an ABL, add a sensor, etc, etc) it gets annoying to have to add cables in our cahins. This is where a CAN BUS PCB comes into its own.  
In my opinion, the advantages are:  
* Easier wiring. We only have 4 wires going from the motherboard to the print head (24V power supply and CAN communication)  
* Flexibility: it greatly facilitates any modification that we want to make on our X cart  

The disadvantages :  
* Adds complexity: An additional mcu and declare a CAN BUS in Linux (it's relative because it has to be done only once)
* It takes a bit of space but in general we know how to get out of it :)

I have been using a Mellow CAN BUS board on my Voron 2.4 for a few months. It's only happiness, it works very well even when printing quick.  
I'm doing this tutorial with a BTT EBB and U2C board that I bought for my Sidewinder X1. The reason is that these are cheaper :)

# Presentation of the boards  

## Principle of operation   
![Principe Communication: https://github.com/bigtreetech/U2C](/images/CAN_how.png)  

## U2C Board  

:warning:**Before you start, make sure you have an up-to-date installation of Klipper :)** :warning:  
The Raspberry Pi (or any other mini computer) does not have a built-in CAN bus. So we need a USB to CAN converter. This is where the U2C board comes into play.  
![Carte U2C Source: https://github.com/bigtreetech/U2C](/images/U2C_description.png)  
* CAN_IN : Connected to a USB port of the Raspberry Pi, it is the USB communication.  
* Power : To be connected to our 24V power supply, it will be the power supply of our printing head.  
* CAN_OUT : We can use one of these 4 outputs to wire our print head. We will use one of the two lower ones because of the currents that will circulate (40W minimum for the heating cartridge alone)  
* CAN_OUT* : These 2 USB connectors are CAN ports on which we can connect a compatible motherboard. It will not be our case here, our motherboard is connected in USB to the Rpi.  
* CAN 120R : These jumpers connect the terminations of the CAN BUS. They are to be placed.  

## EBB board  
This board will be the one that will be on our print head.  
![Carte U2C Source: https://github.com/bigtreetech/EBB](/images/EBB42_description.png)  
* TYPE-C : USB port that we will use to flash (once only) the MCU with a Bootloader. This will allow to flash the MCU in CAN BUS in the future.  
* In blue, the CAN BUS, to be connected to the U2C board.  
* Probe : Can be used for any ABL. Here we will use a BL Touch.  
* I2C : Connector linked to the I2C of the MCU. Can be used to connect different sensors. We will not use it in this tutorial.  
* RGB : Can be used to connect Neopixel leds. Very useful if used on a StealthBurner. We will not use it in this tutorial.  
* Endstop : Is used to wire the endstops. If your X and Y endstops are on the X cart, it is very convenient :)  
* PT100/PT1000 : Will not be used in this tutorial because we use a standard temperature sensor type Generic 3950 or ATC Semitec 104GT-2.  
* TH0 : Hotend temperature sensor.  
* FAN1 & FAN2 : Will be used for the Hotend & Nozzle fan.  
* Hotend 0 : This terminal block will supply the heating cartridge.  
* E motor : This is where we will connect our extruder motor. (A TMC2209 is integrated to the board)  

# Programming the boards  

## U2C  

:warning:**OPTIONAL !!**:warning:  
Normally the U2C board is programmed from the factory. If this is not the case, we will see how to do it.  

* Download this software https://upyun.pan.zxkxz.cn/Utils/STM32CubeProgrammer and et unzip it to the desired location.  
* Connect the U2C board to your Computer by holding the button on the picture to make it enter DFU mode (the blue "Status" led should light up).  
![Carte U2C Prog](/images/u2c_usb.png)  
* Launch STM32CubeProgrammer\bin\STM32CubeProgrammer.exe.  
* The board should be recognized into STM32CubeProgrammer.   
![Carte U2C Prog](/images/STM32CUBE.png)  
It also appears in the Windows Device Manager.  
![Carte U2C Prog](/images/Task_manager.png)  
* Click on connect.  
* Select the file to be programmed here https://github.com/bigtreetech/U2C/tree/master/firmware  
To know which file to choose, you have to read the annotations on the MCU of the board.   
![Carte U2C Prog](/images/MCU_U2C.png)  
Mine is an STM32F072.  
* Open the file and press the Download button to flash the board.  
![Carte U2C Prog](/images/U2C_Flash.png)  

## EBB  

### Flashing the Bootloader  

#### DFU  mode
In order to flash the bootloader, we will have to put the EBB board in DFU mode (as for the U2C board before).   
To do this, connect the board to the Raspberry Pi via USB and hold down the push button. If the 24V is not yet connected, you will also have to put the VBus Jumper to power the board in USB.  
![Carte EBB source : https://github.com/bigtreetech/EBB](/images/EBB_DFU.png)  
:warning:**WARNING !!!! In DFU mode, the Mosfet of the heater cartridge will be active. Be sure to flash the bootloader before connecting it or disconnect it !**:warning:  
![Carte EBB DFU mode](/images/EBB_DFU_RPI.png)  
#### Download, compilation and flash of CAN_BOOT  

Arksine has developed a SUPER CAN bootloader. It will allow to update Klipper in CAN directly. We will not need to connect the USB cable or switch to DFU mode.  
* Connect to your printer via SSH.  
* Install DFU_UTIL with the following command :  
`sudo apt install dfu-util -y`  
* Be sure your board is detected in DFU mode :  
`lsusb`  
 
![EBB en mode DFU](/images/STM_in_DFU_MODE.png)  
If this one is not, make a new reset.  
We can already see our USB ID that we can note. For me 0483:df11.  
* Now let's download CanBoot  
`git clone https://github.com/Arksine/CanBoot`  
`cd CanBoot`  
* We compile the CanBoot for our MCU.  
`make menuconfig`  
:warning:The U2C board is programmed to operate at a CAN frequency of 250000. Make sure you always use this value and not 500000 by default.  
![makemenuconfig CanBoot](/images/makemenuconfigCanBoot.png)  
`make`  
![CanBoot Success](/images/Canboot_success.png)  
* We can now flash the Bootloader we have compiled  
`sudo dfu-util -a 0 -d 0483:df11 --dfuse-address 0x08000000:force:mass-erase -D ~/CanBoot/out/canboot.bin`  
![CanBoot Flash Success](/images/canbootOK.png)  

## CAN configuration on the Rpi  
* Install nano if not already done  
`sudo apt update && sudo apt install nano -y`  
* Create the configuration file for the CAN port. Copy and paste from a block.  
`sudo /bin/sh -c "cat > /etc/network/interfaces.d/can0" << EOF`  
`allow-hotplug can0`  
`iface can0 can static`  
` bitrate 250000`  
` up ifconfig \$IFACE txqueuelen 1024`  
`EOF`  

* Open the file and check it `sudo nano /etc/network/interfaces.d/can0`  
It is possible that the `$IFACE` has not been copied. Add it if necessary and save with CTRL+X - Y - ENTER  
* Automatically activate CAN at power up  
`sudo wget https://upyun.pan.zxkxz.cn/shell/can-enable -O /usr/bin/can-enable > /dev/null 2>&1 && sudo chmod +x /usr/bin/can-enable || echo "The operation failed"`  
`sudo cat /etc/rc.local | grep "exit 0" > /dev/null || sudo sed -i '$a\exit 0' /etc/rc.local`  
`sudo sed -i '/^exit\ 0$/i \can-enable -d can0 -b 250000 -t 1024' /etc/rc.local`  
* On reboot la Rpi.  
`sudo reboot`  

## CAN Bus Wiring  
For the wiring :  
* 24V : Keep in mind that ALL the power will go through this pair of cables. This includes the heating cartridge, the motor, bl touch, etc.  
You must therefore provide a sufficient cross section to support the current. A cross section of min 0.5mm is recommended.    
* CAN communication : Ccurrent passes through here is very low. You can use the same cables as for the 24V or choose thinner ones.  

My personal choice is to use an Ethernet cable. I use 3 pairs for the 24V and one twisted pair for the CAN communication.  

### U2C  
On the U2C board, there is no pinout indicated and the documentation does not clearly mention it.  
Here is the pinout:  
![U2C CAN Pinout](/images/U2C_CAN_PINOUT.png)  
![U2C CAN POWER](/images/U2C_POWER.png)  

### EBB  
On the EBB board, the Pinout is clearly marked :)  
![EBB CAN Pinout](/images/EBB_Pinout.png)  

## Flash Klipper into the EBB  
The boards are now mounted and wired, it is now time to flash Klipper on the EBB.  
* First we compile Klipper for our EBB   
`cd ~/klipper`  
`make menuconfig`  
![EBB menuconfig](/images/EBB_makemenuconfig.png)  
`make clean`  
`make`  
* Then, we will identify our CAN device. We must know its ID to flash it.  
`sudo service klipper stop`  
`cd ~/klipper/lib/canboot/`  
`python3 flash_can.py -q`  
My CAN ID is `ebf7f23a7d85`  
![EBB CAN ID](/images/CANID.png)  
* And finally we flash the EBB  
`python3 flash_can.py -i can0 -f ~/klipper/out/klipper.bin -u ebf7f23a7d85`  
`sudo service klipper start`  
When an update is required, this step is the only one that needs to be repeated.  
All that remains now is to configure Klipper.  

## Configuration Klipper
In the printer.cfg file, add `[include EBB.cfg]` and add the EBB.cfg file to the klipper configuration.  
Comment :  
* Fan & Nozzle Fan  
* All the extrudeur part  
* [bltouch]  
`sensor_pin:`  
`control_pin:`  
The new pins are defined in the EBB.cfg file  
In the EBB.cfg file adjust the parameters :  
* Rotation distance (according your extruder)  
* [tmc2209 extruder] run_current = `YOUR VALUE`  
* Check that all other settings match your configuration.  

# Références
U2C board BigTreeTech Github https://github.com/bigtreetech/U2C  
EBB board BigTreeTech Github https://github.com/bigtreetech/EBB  
STL mouting plate from Voron French Users Github https://github.com/elpopo-eng/VoronFrenchUsers/tree/main/Mod/Huvud_mounts  
Can Bootloader https://github.com/Arksine/CanBoot  
Mini tutorial from Benoit V2.1277 from Discord VoronFR https://discordapp.com/channels/811518442721116232/967097962582933586/967101437899337738  
Mellow Documentation https://mellow.klipper.cn/#/board/fly_sht36_42/  
