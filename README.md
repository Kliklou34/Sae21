# Sae21
28/03</br>
Plusieurs recherches :</br>
	-Installer un serveur DNS sur une machine , https://www.tutos.eu/3446</br>
	-Installer un serveur web sur sa machine grace a Apache2 , vu dans les anciens TP’s</br>
	-Configurer un serveur DNS </br>


29/03</br>

Mise en place d'un serveur apache2</br>
Comme vu lors du module R203</br>

04/04</br>

Debut de la configuration pare feu</br>
Pour se faire je me suis donc connecter en ssh sur le routeur Mikrotik .</br>

	ssh admin@ip_mikrotik
	
J'ai maintenant accés au routeur mirkotik en ligne de commande .</br>
Pour configurer le pare feu il faut tout d'abords ajouter deux listes :</br>
</br>
/interface/firewall/list</br>

	add
		ExternalInterfaces
	add
		InternalInterfaces
		
Ces listes nous permettrons d'identifier sur quels sortie ethernet nous bloquons les accés ,il faut donc que je définisse </br>
quelle interface fait partie de InternalInterfaces et laquelle fait partie de ExternalInterfaces.</br>
Pour ce faire :</br>
/intreface/firewall/list/member</br>

	add
		InternalInterfaces
		ether0
	add
		ExternalInterfaces
		ether1
		
Ainsi je viens donc de donner aux listes les interfaces qui les définissent.</br>
</br>
(14;15)/04</br>
</br>
Maintenant que les interfaces sont définis il faut maintenant commencer a creer des regles sur le pare feu.</br>
Nous avons pour but d'accepter toutes les connections venant de l'exterieur et qui veulent aller sur le serveur web de la DMZ.</br>
Il nous faut donc que l'on accepte le port 80 en tcp pour les requetes http , le port 53 en udp le DNS.</br>
</br>
Pour accepter les connexions depuis l'exterieur en  tcp port 80:</br>
/ip firewall filter </br>

	add chain=forward protocol=tcp dst-port=80 action=accept in-interface-list=ExternalInterfaces comment="Accept tcp port 80" 
	
</br>
Pour accepter les connexions depuis l'exterieur en udp port 53 :</br>
/ip firewall filter </br>

	add chain=forward action=accept protocol=udp dst-port=53 in-interface-list=ExternalInterfaces comment="Accept udp prt 53 , DNS"
	
</br>
Pour accepter les connexions depuis l'exterieur en tcp port 53 :</br>
/ip firewall filter</br>

	add chain=forward action=accept protocol=tcp dst-port=53 in-interface-list=ExternalInterfaces comment="Accept tcp port 53"


21/04</br>
</br>
Il faut maintenant que j'accepte les connections deja établies , cela veut dire que ces machines ont deja traversés le routeur et sont en phase de retours vers la machine .
Si le retour ce fait sur l'interface externe cela veut donc dire qu'il vient du réseau interne , les membres du réseau interne étant libre , nous devons accepter toutes les connections déja établies sans autres restrictions.</br>
</br>
/ip firewall filter</br>

	add chain=forward action=accept connexion-state=established
	
</br>
Cela fait il faut maintenant refuser toutes les autres connections :</br>
</br>
/ip firewall filter</br>

	add chain=forward action=drop
</br>
Le pare feu étant configurer il faut maintenant faire du nat sur la DMZ :</br>
Il faut donc que lorsque une personne exterieure tente d'accéder au serveur web il puisse y accéder , il faut donc faire une redirection</br>
 , je fais donc un redirection vers le serveur web lorsque que quelqu'un tente d'accéder depuis l'exterieur depuis le port 80 en tcp.</br>
/ip firewall nat</br>

	add action=dst-nat chain=dstnat dst-port=80 in-interface-list=ExternalInterfaces protocol=tcp to-addresses=192.168.100.1 to-ports=80 connexion-state=new 		comment="Redirection des paquets venant de l'exterieur en port 80 tcp " 
	
Mais il faut aussi que si une personne tente d'accéder au serveur dns il y ait une redirection vers celui-ci.</br>
Pour ce faire je fais donc une redirection lorsque quelqu'un de l'exterieur tente d'y accéder depuis le port 53 en udp.</br>
/ip firewall nat</br>

	add action=dst-nat in-interface-list=ExternalInterfaces chain=dstnat dst-port=53protocol=udp to-addresses=192.168.100.2 to-ports=53


