# Configuració de la xarxa

Abans de començar, hem de planificar una mica com serà el disseny de la xarxa sobre la qual montarem el nostre homelab.

En un món ideal, ens vindrà un cable del nostre ISP (Proveïdor d'internet), que haurà d'estar connectat a una ONT de la companyia telefònica per a poder obtenir una connexió a Internet. Aquest dispositiu de la companyia de telèfons, al que malanomenarem Router a partir d'ara, complirà les 4 funcions bàsiques que necessitem d'un router:

- Módem: Demodularà la senyal enviada pel medi de la companyia en una senyal intel·ligible pel router.
- Router: Gestionar el tràfic entre la nostra xarxa privada i la xarxa pública.
- Switch: Connectar alguns dispositius de manera física al router.
- AP: Punt d'accés inal·làmbric per a tots els nostres dispositius fora fils.

Per una xarxa estàndard, aquest dispositiu tot-en-un és capaç de dur a terme la seva feina de manera decent. Però, per a nosaltres, no ens servirà, ja que un router estàndard no té les característiques que necessitarem nosaltres. Per començar, necessitarem un switch gestionat que suporti el protocol 802.11Q, un Firewall més potent per a protegir els nostres serveis, un client DDNS, capacitat per a gestionar més d'una xarxa...

Un esquema ideal seria el seguent:

<img src="assets\images\network\network-ideal.png" alt="drawing" width="400"/>

On el router de la companyia és delegat a fer només la funció de módem, i està connectat al nostre router i firewall pfSense. Del mateix, surt una interfaç LAN connectada a un Switch, que ens interconnectarà tots els nostres dispositius (Ordinadors, AP's, Servidors, IoT, etc..)

A la realitat i per limitacions del hardware, el meu esquema es veu així:

<img src="assets\images\network\network-real.png" alt="drawing" width="400"/>

Degut a la limitació de només un NIC del Dell Optiplex 790 que estic emprant com a Router PFSense, necessitarem trobar una manera de tenir una interfaç d'entrada a Internet i una cap a la nostra xarxa privada en un mateix cable. Aquesta configuració es diu Router-On-A-Stick, i s'aconsegueix mitjançant VLANs.

## Què és una VLAN?
Una VLAN és una xarxa virtual, que ens permetrà separar dues xarxes lògiques independents en una mateixa xarxa física.

L'ús de VLANs ens permetrà diferenciar entre el tràfic de sortida cap a Internet, i el tràfic intern de la nostra xarxa. Cada paquet quedarà etiquetat al header de la seva trama d'Ethernet amb un número de VLAN, i així, qualsevol router, switch de capa 2 o switch de capa 3 podrà discernir entre paquets i enviar-los a la seva destinació correcta.

> ## Apunt: IEEE 802.1q
> És el protocol que emprem per a etiquetar els datagrames Ethernet amb la seva tag correcta. Només insereix quatre bytes a la capçalera del datagrama entre la MAC de destí i el tamany de la payload.
> <img src="assets\images\network\datagrama_802.11q.png" alt="Esquema de la capçalera d'una trama Ethernet" width="700"/>
> Tots els dispositius que suportin connexions emprant VLAN han de ser compatibles amb aquest estàndard
