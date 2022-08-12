# CanBus-Tuto
Mise en place d'un PCB Can Bus sur la tête d'impression

## Lien pour acheter les cartes :  
[Aliexpress](https://ali.ski/fYbp4)

# Avantages et inconvénients
Si vous avez déjà monté un imprimante ou changé le câblage, vous savez a quel point ça peut prendre du temps. De plus, ce n'est pas la partie la plus drôle...
De plus, si vous faites une modification sur votre tête d'impression (changement de hotend, ajout d'un ABL, ajout d'un capteur, etc, etc) ça devient vite ennuyant de devoir ajouter des câbles dans nos gaines/chaines. C'est la qu'un PCB CAN BUS prend tout son sens.  
Selon moi, les avantages sont :  
* Faciliter le câblage. On a plus que 4 fils qui vont de la carte mère à la tête d'impression (Alimentation 24V et communication CAN)
* Flexibilité : cela facilité grandement toute modification que l'on veut effectuer sur notre chariot X

Les désavantages :  
* Ajoute de la complexité : Un mcu supplémentaire et déclarer un CAN BUS dans Linux (c'est tout relatif car c'est a faire une seule fois)
* Ca prend un peu de place mais en général on sait s'en sortir :)

J'utilise une carte CAN BUS de chez Mellow sur ma Voron 2.4 depuis quelques mois. C'est que du bonheur, ça fonctionne très bien même en imprimant rapidement. 
Je fais ce tuto avec une carte BTT EBB et U2C que j'ai achetée pour ma X1. La raison est que celles-ci sont moins cher :)

# Présentation des cartes 
## Carte U2C

:warning:**Avant de commencer, assurez-vous d'avoir une installation de Klipper à jour :)** :warning:  
La Raspberry Pi (ou tout autre mini ordinateur) n'a pas de bus CAN intégré. Il faut nous faut donc un convertisseur USB vers CAN. C'est la que la carte U2C entre en jeu. 
![Carte U2C Source: https://github.com/bigtreetech/U2C](/images/U2C_description.png)  
* CAN_IN : Rélié à un port USB de la Raspberry Pi, c'est la communication USB.
* Power : A relier à notre alimentation 24V, ce sera l'alimentation de notre tête d'impresison.
* CAN_OUT : On peut utiliser une de ces 4 sorties pour cabler notre tête d'impression. On utilisera une des deux du bas vu les courrants qui vont circuler (40W pour rien que pour la cartouche de chauffe)
* CAN_OUT* : Ces 2 ports USB sont des ports CAN sur lesquels ont peut brancher une carte mère compatible. Ce ne sera pas notre cas ici, notre carte mère est reliée en USB à la Rpi.
* CAN 120R : Ces jumpers connectent les terminaisons du CAN BUS. Ils sont a placer.

## Carte EBB
Cette carte sera celle qui se trouvera sur notre tête d'impression. 
![Carte U2C Source: https://github.com/bigtreetech/EBB](/images/EBB42_description.png)  
* TYPE-C : Port USB que nous utiliserons pour flasher (une seule fois) le MCU avec un Bootloader. Cela pour permetra de pourvoir flasher le MCU en CAN BUS à l'avenir.
* En bleu,le BUS CAN, à relier à la carte U2C.
* Probe : peut être utilisé pour n'importe quel ABL. Ici nous utiliserons un BL Touch.
* I2C : Connecteur rélié à l'I2C su MCU. Peut être utilisé pour brancher différents capteurs. Nous ne l'utiliserons pas dans ce tuto.
* RGB : Peut être utilisé pour relier des leds type Néopixel. Nous ne l'utiliserons pas dans ce tuto.
* Endstop : Est utilisé pour configurer les endstops. Si vos Endstops X et Y sont sur le chariot X, c'est très pratique :)
* PT100/PT1000 : Ne sera pas utiulisé dans ce tuto car on utilise une sonde de température standard type Generic 3950 ou ATC Semitec 104GT-2.
* TH0 : Capteur de température Hotend.
* FAN1 & FAN2 : seront utilisés pour le ventillateur Hotend & Buse.
* Hotend 0 : Ce bornier alimentera la cartouche chauffante.
* E motor : C'est ici que l'on branchera notre moteur extrudeur. (Un TMC2209 est intégré à la carte)

# Programmation des cartes
## U2C  
Normalement la carte U2C est programmée d'usine. Si toutefois ce n'étais pas le cas, nous allons voir comment faire.

* téléchargez le programme https://upyun.pan.zxkxz.cn/Utils/STM32CubeProgrammer et décompressez le à l'endroit souhaité.
* Connectez la carte U2C à votre PC en maintenant le bouton sur la photo pour la faire rentrer en mode DFU (La led "Status" bleue doit s'alumer).
![Carte U2C Prog](/images/u2c_usb.png)  
* Lancer STM32CubeProgrammer\bin\STM32CubeProgrammer.exe.
* La carte doit etre reconnue dans STM32CubeProgrammer. 
![Carte U2C Prog](/images/STM32CUBE.png)  
Elle apparait aussi dans le Gestionnaire de périphériques Windows.
![Carte U2C Prog](/images/Task_manager.png)
* Clickez sur connect.
* Sélectionnez le fichier à programmer ici https://github.com/bigtreetech/U2C/tree/master/firmware  
Pour savoir quel fichier choisir, il faut lire les anotations sur le MCU de la carte. 
![Carte U2C Prog](/images/MCU_U2C.png)
Pour moi il s'agit d'un STM32F072.
* Ouvrez le fichier et appuyez sur le bouton Download pour flasher la carte. 
![Carte U2C Prog](/images/U2C_Flash.png)

## EBB
### Flash du Bootloader  
#### Mode DFU  
Afin de flasher le bootloader, nous allons devoir mettre la carte EBB en mode DFU (comme la carte U2C précédement). 
Pour ce faire, branchez la carte en USB à la Raspberry Pi en maintenant le bouton poussoir enfoncé. Si le 24V n'est pas encore relié, vous devrez aussi mettre le Jumper VBus pour alimenter la carte en USB.  
![Carte EBB source : https://github.com/bigtreetech/EBB](/images/EBB_DFU.png)  
:warning:**ATTENTION !!!! En mode DFU, le Mosfet de la cartouche de chauffe sera actif. Veillez à flasher le bootloader avant de la brancher ou déconnectez la !**:warning:  
![Carte EBB DFU mode](/images/EBB_DFU_RPI.png)  
#### Téléchrgement compilation et flash de CAN_BOOT
Arksine nous a développé un SUPER bootloader CAN. Il permettra de mettre a jour Klipper en CAN directement. Nous n'aurons donc plus besoin de brancher le Cable USB ou de passer en mode DFU.
* Connectez vous à votre Imprimante en SSH.
* Installez DFU_UTIL avec la commande suivante :  
`sudo apt install dfu-util -y`  
* Assurez vous que votre carte est bien détectée en mode DFU :  
`lsusb`
![EBB en mode DFU](/images/STM_in_DFU_MODE.png) 
Si celle-ci ne l'est pas, faites un nouveat reset.
On peut déjà observer notre ID USB que l'on peut noter. Pour moi 0483:df11. 
* Téléchargons maintenant le CanBoot
`git clone https://github.com/Arksine/CanBoot`  
`cd CanBoot`  
* On compile le CanBoot pour notre MCU. 
`make menuconfig`  
![makemenuconfig CanBoot](/images/makemenuconfigCanBoot.png)
`make`  
![CanBoot Success](/images/Canboot_success.png)
* On peut maintenant flasher le Bootloader qu'on a compilé 
`sudo dfu-util -a 0 -d 0483:df11 --dfuse-address 0x08000000:force:mass-erase -D ~/CanBoot/out/canboot.bin`
![CanBoot Flash Success](/images/canbootOK.png)

## Configuration du CAN sur la Rpi  
* Installer nano si ce n'est pas déja fait  
'sudo apt update && sudo apt install nano -y'  
* Créer le fichier de configuration pour le port CAN. Copier Coller d'un bloc.  
`sudo /bin/sh -c "cat > /etc/network/interfaces.d/can0" << EOF`  
`auto can0`  
`iface can0 can static`  
` bitrate 250000`  
` up ifconfig $IFACE txqueuelen 1024`  
`EOF`
* Ouvrez le fichier et vérifiez le `sudo nano /etc/network/interfaces.d/can0`  
Il est possible que le `$IFACE` n'ai pas été copié. Ajoutez le si nécessaire et enregistrez avec CTRL+X - Y - ENTER
* Activer automatiquement le CAN à la mise sous tension
`sudo wget https://upyun.pan.zxkxz.cn/shell/can-enable -O /usr/bin/can-enable > /dev/null 2>&1 && sudo chmod +x /usr/bin/can-enable || echo "The operation failed"`  
`sudo cat /etc/rc.local | grep "exit 0" > /dev/null || sudo sed -i '$a\exit 0' /etc/rc.local`  
`sudo sed -i '/^exit\ 0$/i \can-enable -d can0 -b 250000 -t 1024' /etc/rc.local`  
* On reboot la Rpi.  
`sudo reboot`  

## Cablage du CAN Bus  
Pour le cablage :
* 24V : Il faut garder à l'esprit que TOUTE l'alim passera par cette paire de cables. Cela inclut la cartouche de chauffe, le moteur, bl touch, etc.  
Il faut donc prévoir une section suffisante pour supporter le courrant. Une section de min 0.5mm est recommandée.  
* La Communication CAN. Très peu de courrant passe ici.Vous pouvez utiliser les mêmes cables que pour le 24V ou choisir des plus fins.  

Mon  choix personnel est d'utiliser un cable Ethernet. J'utilise 3 paires pour le 24V et une paire tosadée pour la communication CAN.  

### U2C  
Sur la carte U2C, il n'y a pas de pinout indiqué et la documentation le le mentionne pas clairement. 
Voici le pinout :
![U2C CAN Pinout](/images/U2C_CAN_PINOUT.png)
![U2C CAN POWER](/images/U2C_POWER.png)

### EBB  
Sur la carte EBB, le Pinout est clairement indiqué :)
![EBB CAN Pinout](/images/EBB_Pinout.png)  

## Flash de Klipper sur la EBB  
Les cartes sont maintenant montées et câblées, Il est maintenant temps de flasher Klipper sur la EBB. 
* D'abord on Compile Klipper pour notre EBB
`cd Klipper`  
`make menuconf`  
![EBB menuconfig](/images/EBB_makemenuconfig.png)
`make clean`  
`make`  
* Ensuite, on va identifier notre périphérique CAN. On doit connaitre son ID pour le flasher.
`sudo service klipper stop`  
`cd ~/klipper/lib/canboot/`  
`python3 flash_can.py -q`  
Mon CAN ID est `ebf7f23a7d85`
![EBB CAN ID](/images/CANID.png)
* Et enfin on flash la EBB  
`python3 flash_can.py -i can0 -f ~/klipper/out/klipper.bin -u ebf7f23a7d85`  
`sudo service klipper start`  
Il ne reste maintenant plus qu'a faire la configuration Klipper.

## Configuration Klipper
Dans le fichier printer.cfg
Ajouter `[include EBB.cfg]` et ajouter le fichier EBB.cfg dans la configuration klipper.
Commenter :
* Fan & Nozzle Fan  
* Toute la partie extrudeur  
* [bltouch]
`sensor_pin: ^EBBCan:PB8`
`control_pin: EBBCan:PB9`
Les nouvelles pins sont définies dans le fichier EBB.cfg  
Dans le fichier EBB.cfg ajuster les paramètres :  
* Rotation distance (selon votre extrudeur)  
* [tmc2209 extruder] run_current = `VOTRE VALEUR`

# Références
Carte U2C BigTreeTech Github https://github.com/bigtreetech/U2C  
Carte EBB BigTreeTech Github https://github.com/bigtreetech/EBB  
STL de Montage des cartes Voron French Users Github https://github.com/elpopo-eng/VoronFrenchUsers/tree/main/Mod/Huvud_mounts  
Can Bootloader https://github.com/Arksine/CanBoot
Mini tuto de Benoit V2.1277 Sur le Discord VoronFR https://discordapp.com/channels/811518442721116232/967097962582933586/967101437899337738
