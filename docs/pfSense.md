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

El nostre Router està connectat a Internet, però, i els nostres dispositius? Mai trobaràn cap xarxa disponible ja que encara no l'hem configurat.

Ens posicionarem a sobre l'apartat Interfícies / LAN, activarem l'interfície i li assignarem una subxarxa.

<img src="assets\images\lan1.png" alt="Creació de VLANs al switch TP-Link TL-SG3210" width="700"/>

Hem aplicat la direcció IP 10.0.74.1/24 al nostre PFSense, així que estem emprant la subxarxa 10.0.74.0 amb màscara 255.255.255.0.

Anem a configurar el servidor DHCP per a que els nostres dispositius puguin connectar-se a la xarxa de manera automàtica.

> ## Apunt: DHCP
> DHCP és un protocol que ens permet assignar de manera dinàmica una IP a cada dispositiu de la nostra xarxa, emprant una arquitectura semblant a una client-servidor, així eliminant la necessitat d'anar assignant-los un a un. 
>
> El dispositiu client enviarà un paquet broadcast (a tots els dispositius de la xarxa) avisant de que està connectat i disponible per a rebre una direcció.
> Si el servidor DHCP reb aquest missatge, li reservarà una direcció IP i li enviarà una oferta per a la "DHCP Lease". Aquesta oferta conté la IP del router, la IP del servidor DHCP, la màscara de subxarxa, el temps de *lease* i la direcció del client.
> El dispositiu client enviarà només al servidor DHCP un datagrama *Request* per a acceptar l'adreça oferida, i el servidor contestarà amb un datagrama *Ack* per a confirmar que la direcció ha estat assignada de manera correcta.
>
><img src="assets\images\DHCP_session.png" alt="Creació de VLANs al switch TP-Link TL-SG3210" width="300"/>

Per a configurar DHCP a l'interfície LAN, anirem al menú Services > DHCP Server, i seleccionarem LAN.

<img src="assets\images\dhcp1.png" alt="Creació de VLANs al switch TP-Link TL-SG3210" width="700"/>

Podem veure el rang d'IP's que el servidor DHCP ens pot servir, i seleccionar un grapat de direccions, o tot el rang.

En el meu cas, he escollit un rang des del 10.0.74.128 fins al 10.0.74.192 ja que he reservat totes les demés direccions cap a DHCP static mappings (és a dir, assignar una IP estàtica a un dispositiu emprant el protocol DHCP.).