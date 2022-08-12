# CanBus-Tuto
Mise en place d'un PCB Can Bus sur la tête d'impression

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
* E motor : C'est ici que l'on branchera notre moteur extrudeur.

# Références
Carte U2C BigTreeTech Github https://github.com/bigtreetech/U2C  
Carte EBB BigTreeTech Github https://github.com/bigtreetech/EBB  
STL de Montage des cartes Voron French Users Github https://github.com/elpopo-eng/VoronFrenchUsers/tree/main/Mod/Huvud_mounts  
