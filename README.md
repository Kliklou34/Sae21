# Sae21
28/03</br>
Plusieurs recherches :</br>
	-Installer un serveur DNS sur une machine , https://www.tutos.eu/3446</br>
	-Installer un serveur web sur sa machine grace a Apache2 , vu dans les anciens TP’s</br>
	-Configurer un serveur DNS </br>


29/03</br>

Mise en place d'un serveur apache2</br>
Comme vu lors du module R203</br>

04/04

Debut de la configuration pare feu
Pour se faire je me suis donc connecter en ssh sur le routeur Mikrotik .
	ssh admin@ip_mikrotik
J'ai maintenant accés au routeur mirkotik en ligne de commande .
Pour configurer le pare feu il faut tout d'abords ajouter deux listes :
Dans 
/interface/firewall/list
	add
		ExternalInterfaces
	add
		InternalInterfaces
Ces listes nous permettrons d'identifier sur quels sortie ethernet nous bloquons les accés ,il faut donc que je définisse 
quelle interface fait partie de InternalInterfaces et laquelle fait partie de ExternalInterfaces.
Pour ce faire :
/intreface/firewall/list/member
	add
		InternalInterfaces
		ether0
	add
		ExternalInterfaces
		ether1
Ainsi je viens donc de donner aux listes les interfaces qui les définissent.

(14;15)/04

Maintenant que les interfaces sont définis il faut maintenant commencer a creer des regles sur le pare feu.
Nous avons pour but d'accepter toutes les connections venant de l'exterieur et qui veulent aller sur le serveur web de la DMZ.
Il nous faut donc que l'on accepte le port 80 en tcp pour les requetes http , le port 53 en udp et tcp pour le DNS.

Pour accepter les connexions depuis l'exterieur en  tcp port 80:
/ip firewall filter 
	add chain=forward protocol=tcp dst-port=80 action=accept in-interface-list=ExternalInterfaces comment="Accept tcp port 80" 

Pour accepter les connexions depuis l'exterieur en udp port 53 :
/ip firewall filter 
	add chain=forward action=accept protocol=udp dst-port=53 in-interface-list=ExternalInterfaces comment="Accept udp prt 53 , DNS"

Pour accepter les connexions depuis l'exterieur en tcp port 53 :
/ip firewall filter
	add chain=forward action=accept protocol=tcp dst-port=53 in-interface-list=ExternalInterfaces comment="Accept tcp port 53"


21/04

Il faut maintenant que j'accepte les connections deja établies , cela veut dire que ces machines ont deja traversés le routeur et sont en phase de retours vers la machine .
Si le retour ce fait sur l'interface externe cela veut donc dire qu'il vient du réseau interne , les membres du réseau interne étant libre , nous devons accepter toutes les connections déja établies sans autres restrictions.

/ip firewall filter
	add chain=forward action=accept connexion-state=established

Cela fait il faut maintenant refuser toutes les autres connections :

/ip firewall filter
	add chain=forward action=drop
	
