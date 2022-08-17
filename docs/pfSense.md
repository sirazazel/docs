# PFSense: Com ens connectem a Internet?

PFSense es un software dedicat a complir les funcions de router i firewall. El 


# Router-On-A-Stick

El nostre PFSense estarà connectat a la nostra xarxa en una configuració que anomenarem Router-On-A-Stick. 

Necessitarem crear dues VLAN al nostre switch, la vlan 10 de LAN i gestió i la vlan 99 de WAN.

<img src="assets\images\vlancreation1.png" alt="Creació de VLANs al switch TP-Link TL-SG3210" width="700"/>

Seleccionarem els ports 1 i 2 del switch El primer, cap al router i el segon cap al módem.

<img src="assets\images\vlancreation2.png" alt="Creació de VLANs al switch TP-Link TL-SG3210" width="700"/>

A la pestanya de Port Config, hem d'etiquetar el port 2 com a port ACCESS amb PVID 99. Així, tot el tràfic que arribi desde el módem quedarà etiquetat com a tràfic de la VLAN 99 de cara a la nostra xarxa.

El port 1 l'haurem d'etiquetar com a mode TRUNK. El mode TRUNK permetrà passar múltiples VLANs tagged, però només una VLAN untagged.

<img src="assets\images\vlancreation3.png" alt="Creació de VLANs al switch TP-Link TL-SG3210" width="700"/>

Després, anirem a la direcció de PFSense. A la secció d'Interfaces > Assignments, crearem les VLAN pertinents. Les hem de crear totes a l'interfaç física em0, ja que es l'única del dispositiu.

<img src="assets\images\vlancreation4.png" alt="Creació de VLANs al switch TP-Link TL-SG3210" width="700"/>

<img src="assets\images\vlancreation5.png" alt="Creació de VLANs al switch TP-Link TL-SG3210" width="700"/>

Un cop les VLAN han estat creades, podrem crear una interfaç virtual per cada VLAN disponible al port. Anirem a l'apartat Interface Assignments.

<img src="assets\images\vlancreation6.png" alt="Creació de VLANs al switch TP-Link TL-SG3210" width="700"/>

Ara haurem de configurar la interfície WAN com a una connexió PPPoE.

> ## Apunt: PPPoE
>  PPPoE és una aplicació del protocol punt-a-punt sobre el protocol Ethernet. És el protocol encarregat de proveïr-nos d'identitat en front del nostre ISP i que ens pugui oferir una connexió de banda ampla.

<img src="assets\images\pppoe1.png" alt="Creació de VLANs al switch TP-Link TL-SG3210" width="700"/>

Marcant PPPoE a l'apartat de l'interfície WAN, i inserint l'usuari i contrassenya que ens proporciona telefònica, tindrem connexió a Internet.

<img src="assets\images\pppoe2.png" alt="Creació de VLANs al switch TP-Link TL-SG3210" width="700"/>

# Xarxa local - LAN1

El nostre Router està connectat a Internet, però, i els nostres dispositius?