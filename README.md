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


# Références
(Carte U2C BigTreeTech Github)[https://github.com/bigtreetech/U2C]
(Carte EBB BigTreeTech Github)[https://github.com/bigtreetech/EBB]
(STL de Montage des cartes Voron French Users Github)[https://github.com/elpopo-eng/VoronFrenchUsers/tree/main/Mod/Huvud_mounts]
