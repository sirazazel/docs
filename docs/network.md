# Configuració de la xarxa

Abans de començar, hem de planificar una mica com serà el disseny de la xarxa sobre la qual montarem el nostre homelab.

En un món ideal, ens vindrà un cable del nostre ISP (Proveïdor d'internet), que haurà d'estar connectat a una ONT de la companyia telefònica per a poder obtenir una connexió a Internet. Aquest dispositiu de la companyia de telèfons, al que malanomenarem Router a partir d'ara, complirà les 4 funcions bàsiques que necessitem d'un router:

- Módem: Demodularà la senyal enviada pel medi de la companyia en una senyal intel·ligible pel router.
- Router: Gestionar el tràfic entre la nostra xarxa privada i la xarxa pública.
- Switch: Connectar alguns dispositius de manera física al router.
- AP: Punt d'accés inal·làmbric per a tots els nostres dispositius fora fils.

Per una xarxa estàndard, aquest dispositiu tot-en-un és capaç de dur a terme la seva feina de manera decent. Però, per a nosaltres, no ens servirà, ja que un router estàndard no té les característiques que necessitarem nosaltres. Per començar, necessitarem un switch gestionat que suporti el protocol 802.11Q, un Firewall més potent per a protegir els nostres serveis, un client DDNS, capacitat per a gestionar més d'una xarxa...

