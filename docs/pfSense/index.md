---
layout: default
title: Firewall
has_children: true
nav_order: 2
---
# PFSense: Com ens connectem a Internet?

PFSense es un software dedicat a complir les funcions de router i firewall. El 


# Router-On-A-Stick

El nostre PFSense estarà connectat a la nostra xarxa en una configuració que anomenarem Router-On-A-Stick. 

Necessitarem crear dues VLAN al nostre switch, la vlan 10 de LAN i gestió i la vlan 99 de WAN.

<img src="..\assets\images\pfSense\vlancreation1.png" alt="Creació de VLANs al switch TP-Link TL-SG3210" width="700"/>

Seleccionarem els ports 1 i 2 del switch El primer, cap al router i el segon cap al módem.

<img src="..\assets\images\pfSense\vlancreation2.png" alt="Creació de VLANs al switch TP-Link TL-SG3210" width="700"/>

A la pestanya de Port Config, hem d'etiquetar el port 2 com a port ACCESS amb PVID 99. Així, tot el tràfic que arribi desde el módem quedarà etiquetat com a tràfic de la VLAN 99 de cara a la nostra xarxa.

El port 1 l'haurem d'etiquetar com a mode TRUNK. El mode TRUNK permetrà passar múltiples VLANs tagged, però només una VLAN untagged.

<img src="..\assets\images\pfSense\vlancreation3.png" alt="Creació de VLANs al switch TP-Link TL-SG3210" width="700"/>

Després, anirem a la direcció de PFSense. A la secció d'Interfaces > Assignments, crearem les VLAN pertinents. Les hem de crear totes a l'interfaç física em0, ja que es l'única del dispositiu.

<img src="..\assets\images\pfSense\vlancreation4.png" alt="Creació de VLANs al switch TP-Link TL-SG3210" width="700"/>

<img src="..\assets\images\pfSense\vlancreation5.png" alt="Creació de VLANs al switch TP-Link TL-SG3210" width="700"/>

Un cop les VLAN han estat creades, podrem crear una interfaç virtual per cada VLAN disponible al port. Anirem a l'apartat Interface Assignments.

<img src="..\assets\images\pfSense\vlancreation6.png" alt="Creació de VLANs al switch TP-Link TL-SG3210" width="700"/>

Ara haurem de configurar la interfície WAN com a una connexió PPPoE.

> ## Apunt: PPPoE
>  PPPoE és una aplicació del protocol punt-a-punt sobre el protocol Ethernet. És el protocol encarregat de proveïr-nos d'identitat en front del nostre ISP i que ens pugui oferir una connexió de banda ampla.

<img src="..\assets\images\pfSense\pppoe1.png" alt="Creació de VLANs al switch TP-Link TL-SG3210" width="700"/>

Marcant PPPoE a l'apartat de l'interfície WAN, i inserint l'usuari i contrassenya que ens proporciona telefònica, tindrem connexió a Internet.

<img src="..\assets\images\pfSense\pppoe2.png" alt="Creació de VLANs al switch TP-Link TL-SG3210" width="700"/>

# Xarxa local - LAN 1

El nostre Router està connectat a Internet, però, i els nostres dispositius? Mai trobaràn cap xarxa disponible ja que encara no l'hem configurat.

Ens posicionarem a sobre l'apartat Interfícies / LAN, activarem l'interfície i li assignarem una subxarxa.

<img src="..\assets\images\pfSense\lan1.png" alt="Creació de VLANs al switch TP-Link TL-SG3210" width="700"/>

Hem aplicat una IP 10.0.74.1 a l'interfície  amb una màscara /24 al nostre PFSense, que ens dona direccions per 255 dispositius.

> ## Apunt: Non-routable address space
> Les direccions IP ipv4 escassegen. Ja no basta una direcció IP per persona, encara menys si necessitàssim una direcció per cada dispositiu. Per això, es van designar uns rangs de direccions no enrutables per poder crear sistemes autònoms interns. Aquestes direccions són les seguents
> * 10.0.0.0/8
> * 172.16.0.0/12
> * 192.168.0.0/24
>
> Aquestes direccions no son enrutades pel router cap a fora de la nostra xarxa.

Anem a configurar el servidor DHCP per a que els nostres dispositius puguin connectar-se a la xarxa de manera automàtica.

> ## Apunt: DHCP
> DHCP és un protocol que ens permet assignar de manera dinàmica una IP a cada dispositiu de la nostra xarxa, emprant una arquitectura semblant a una client-servidor, així eliminant la necessitat d'anar assignant-los un a un. 
>
> El dispositiu client enviarà un paquet broadcast (a tots els dispositius de la xarxa) avisant de que està connectat i disponible per a rebre una direcció.
> Si el servidor DHCP reb aquest missatge, li reservarà una direcció IP i li enviarà una oferta per a la "DHCP Lease". Aquesta oferta conté la IP del router, la IP del servidor DHCP, la màscara de subxarxa, el temps de *lease* i la direcció del client.
> El dispositiu client enviarà només al servidor DHCP un datagrama *Request* per a acceptar l'adreça oferida, i el servidor contestarà amb un datagrama *Ack* per a confirmar que la direcció ha estat assignada de manera correcta.
>
><img src="..\assets\images\pfSense\DHCP_session.png" alt="Creació de VLANs al switch TP-Link TL-SG3210" width="300"/>

Per a configurar DHCP a l'interfície LAN, anirem al menú Services > DHCP Server, i seleccionarem LAN.

<img src="..\assets\images\pfSense\dhcp1.png" alt="Creació de VLANs al switch TP-Link TL-SG3210" width="700"/>

Podem veure el rang d'IP's que el servidor DHCP ens pot servir, i seleccionar un grapat de direccions, o tot el rang.

En el meu cas, he escollit el rang .128 > .192 per a assignar dispositius mitjançant DHCP, ja que he reservat totes les demés direccions cap a DHCP static mappings (és a dir, assignar una IP estàtica a un dispositiu emprant el protocol DHCP.).

# Xarxa local - OPT 1

La VLAN 1 (nativa) estarà dedicada als dispositius IoT i a dispositius foranis que es connectin mitjançant Wi-Fi. 
Podrà connectar-se a Internet, però no podrà iniciar connexions a la xarxa local ni a la xarxa de serveis.

<img src="..\assets\images\pfSense\XarxaOPT1_1.png" alt="Creació de VLANs al switch TP-Link TL-SG3210" width="700"/>

No està implantada encara

# Xarxa local - OPT 2

La VLAN 20 és la xarxa que contindrà els nostres serveis que oferim a Internet.

- La xarxa OPT 2 té la direcció 172.16.0.0
- Gateway - 172.16.0.1
- Regles firewall 
    - Permeten tràfic a fora (Internet)
    - No permeten tràfic originant de OPT2 cap a LAN1, ni cap a OPT1
    - Permeten connexions als ports 22,80 i 443 originàries de LAN1
    - Permeten connexions als ports 80 i 443 originàries dels servidors de Cloudflare.
    <img src="..\assets\images\pfSense\XarxaOPT2_1.png" alt="Creació de VLANs al switch TP-Link TL-SG3210" width="500"/>
    - No permet cap tipus de connexió que no provingui de Cloudflare
    <img src="..\assets\images\pfSense\XarxaOPT2_2.png" alt="Creació de VLANs al switch TP-Link TL-SG3210" width="500"/>
    <img src="..\assets\images\pfSense\XarxaOPT2_3.png" alt="Creació de VLANs al switch TP-Link TL-SG3210" width="500"/>
    <img src="..\assets\images\pfSense\XarxaOPT2_4.png" alt="Creació de VLANs al switch TP-Link TL-SG3210" width="500"/>

# Xarxa local - OPT 3

La xarxa OPT3 és una xarxa interna entre les meves màquines virtuals i el reverse proxy.
