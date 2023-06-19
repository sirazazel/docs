---
layout: default
title: IDS/IPS
parent: Firewall
grand_parent: Configuram PFSense com a Router / Firewall principal.
nav_order: 3
---

# Detecció d'intrusos, la primera línia de defensa contra les connexions que han passat el firewall.

Els sistemes IDS, de l'anglès Intrusion Detection System, serà qui ens notificarà si ha entrat tràfic sospitós a la nostra xarxa. La diferència amb un firewall és clara, el firewall només actua no permetent l'entrada, però un pic l'amenaça és a dins no pot actuar, en canvi, un IDS sí ja que monitoritza el tràfic intern.Tenim diferents tipus d'IDS, els passius i els reactius.

Un IDS passiu detecta les anomalies comparant el tràfic de la xarxa amb la seva base de dades de firmes malicioses, o bé analitzant el comportament del tràfic en cerca d'anomalies i, en el cas de detectar un request maliciós que compleixi alguna de les seves condicions, marca el tràfic com a perillós i avisa a l'administrador per mitjà d'una senyal d'alerta.

Els IDS reactius, també anomenats IDS/IPS, deixaràn constància de quins paquets han activat l'alerta i bloquejaràn el tràfic amb l'agent que les hagi originat.

Nosaltres instal·larem el paquet Suricata per PFSense, i comprovarem la seva funcionalitat vegent quant temps tarda en bloquejar-nos.

<img src="..\assets\images\suricata\logoSuricata.jpeg" alt="Logo Suricata" width="400" style="display: block; margin-left: auto; margin-right: auto;"/>

## Instal·lació de Suricata.

El paquet s'instal·larà de la mateixa manera que hem instal·lat el paquet pfBlocker-NG. Anirem a System > Packet Manager > Avaliable Packages, buscarem Suricata i l'instal·larem.

El servei el podrem trobar a Services > Suricata.

<img src="..\assets\images\suricata\suricata1.png" alt="Interfícies" width="700" style="display: block; margin-left: auto; margin-right: auto;"/>

Aquí veurem un llistat amb les interfícies que ja hem configurat. Jo he configurat Suricata per que avisi en cas de trobar tràfic sospitós a la LAN, i que avisi i bloquegi en cas de trobar tràfic maliciós a la OPT2, que serà la xarxa pública.

Però, anem pas a pas. Començarem afegint les regles que haurà de seguir Suricata.

## Afegint regles al joc.

Anirem a l'opció de Global Settings, i afegirem les regles que creguem necessàries. Jo he aplicat les regles d'ETOpen, les regles Registered de Talos-Snort, les regles de Feodo Botnet Tracker i d'ABUSE.ch.

<img src="..\assets\images\suricata\rulesets.png" alt="Regles" width="700" style="display: block; margin-left: auto; margin-right: auto;"/>

Per les regles de Snort, ens hem de crear un compte seguint [aquest enllaç](https://www.snort.org/) i treure el codi Oinkmaster, que està disponible al nostre nou perfil.

També és recomanable crear-nos un compte a [Maxmind](https://www.maxmind.com/en/geolite2/signup), i treure la llicència de GeoLite2 gratuita. És una base de dades que ens ajudarà a detectar tràfic desde llocs inhòspits, connexions satèl·lit o mitjançant Tor que el més segur és que no siguin bon senyal.

Sel·leccionarem també cada quan s'actualitzen totes les llistes d'amenaces. Jo ho he deixat com per defecte, cada 12 hores.

<img src="..\assets\images\suricata\suricata2.png" alt="Regles" width="700" style="display: block; margin-left: auto; margin-right: auto;"/>

Per últim, configurarem com actuarà Suricata en referent al temps que estaràn bloquejats els infractors. Donat que les direccions IP van canviant, no és gens pràctic bloquejar els infractors per sempre, ja que aniràn canviant la seva direcció i podem acabar bloquejant peticions de clients legítimes. 

Aquí, hem elegit que els hosts bloquejats no es permeten fins que ha passat un dia des del darrer intent d'atac.

<img src="..\assets\images\suricata\suricata3.png" alt="Configuram el Timeout dels infractors" width="700" style="display: block; margin-left: auto; margin-right: auto;"/>

Un pic feta la configuració, podrem veure a la pestanya Updates tots els nostres rulesets.

<img src="..\assets\images\suricata\suricata4.png" alt="Regles" width="700" style="display: block; margin-left: auto; margin-right: auto;"/>

## Aplicant les regles a una interfície.

Suricata ja sap què escoltar, el problema és que no sap a ón ha d'aplicar-ho. Començarem marcant on **mai ha d'escoltar**.

### Pass lists.

Una pass list és una llista de hosts que Suricata tindrà activat que no ha de bloquejar mai, facin el que facin. Entre d'altres, seràn el hosts de les xarxes locals, els servidors DNS que tinguem configurats, les VPN que puguem tenir actives i les Gateways de la xarxa externa.

<img src="..\assets\images\suricata\passlist.png" alt="Dispositius que Suricata ha d'ignorar." width="700" style="display: block; margin-left: auto; margin-right: auto;"/>


### Activarem una nova interfície.

Per a activar Suricata en una interfície, anirem a la primera pestanya Interfaces > Add.

<img src="..\assets\images\suricata\novaint1.png" alt="menú nova interfície." width="700" style="display: block; margin-left: auto; margin-right: auto;"/>

Per defecte s'aplica a la WAN. Però, realment, és necessari? És el mateix si investigam WAN, però tenim a tothom deshabilitat a la PassList excepte OPT2, que si només investigam OPT2, i ens estalviem moltíssims processaments de paquets que s'acceptaràn igual.

Per tant, les regles de Suricata s'aplicaràn a la interfície OPT2, i podem començar a configurar com actuarà Suricata amb cada alerta.

<img src="..\assets\images\suricata\options1.png" alt="menú opcions interfície." width="700" style="display: block; margin-left: auto; margin-right: auto;"/>

Aquí podem elegir quins logs quedaràn guardats al Syslog per a consultar més tard. Hem de tenir en compte que necessitarem un disc o un NFS molt gran si sel·leccionam guardar més logs dels necessaris.

### Bloqueig dels infractors.

Podrem elegir si bloquejar tots els infractors o només notificar dels atacs. En el cas d'OPT2, volem bloquejar-los. Estarem emprant el mode Legacy ja que amb Inline Mode hem tingut problemes de rendiment degut a la quantitat de paquets a bloquejar.

<img src="..\assets\images\suricata\blockoffenders.png" alt="bloqueig dels infractors." width="700" style="display: block; margin-left: auto; margin-right: auto;"/>

Per últim, aplicarem la nostra Pass List per a prevenir el bloqueig de direccions intern.

<img src="..\assets\images\suricata\passlist1.png" alt="Aplicació passlist." width="700" style="display: block; margin-left: auto; margin-right: auto;"/>

Ja tindrem la interfície configurada a Suricata. Confirmarem i guardarem els canvis, i l'IPS ja s'haurà posat a escoltar.

En pocs minuts, podem començar a comprovar com estan entrant peticions i Suricata les està notificant o bloquejant depenent de la gravetat. Els primers dies, haurem d'anar en cura i revisant les regles, ja que moltes donaràn falsos positius, com les que veurem ara.

<img src="..\assets\images\suricata\falspositiu.png" alt="Falsos positius." width="700" style="display: block; margin-left: auto; margin-right: auto;"/>
