---
layout: default
title: Regles Firewall
parent: Firewall
nav_order: 2
---
# Exposició de ports al món
Com ja sabem, per ser accessibles des de fora de la nostra xarxa, els serveis han de tenir una redirecció de ports activa. Teòricament, cada servei necessitarà un port obert per a funcionar, però, eines com un Reverse Proxy ens permeten tenir un únic port obert (per exemple, el port 443 per al tràfic HTTP encriptat emprant el protocol TLS), i permetre-n's accedir a tots els nostres serveis ja sigui mitjançant subdominis, dominis diferents, una pàgina de selecció, etc.

Nosaltres, emprarem un Reverse proxy per a filtrar el tràfic depenent del subdomini que nosaltres entrem a la direcció URL. He escollit el servidor web Caddy per a funcionar com a reverse proxy, ja que és bastant simple d'emprar i configurar, i suporta la creació de certificats TLS 1.3 amb entitats certificadores com Let's Encrypt de manera bastant autònoma. A la pàgina [Caddy](https://docs.espardenya.me/Hipervisor/caddy.html) tindrem més informació sobre com estarà configurat aquest servei.

Per començar, només ens bastarà saber quina serà la direcció IP on escoltarà aquest servidor, i crear una regla per a permetre-hi el tràfic.
## Álias de direccions IP
PFSense ens permet crear áliases de direccions IP per a facilitar la gestió de les regles del Firewall. Començarem creant dos álias simples, un de la direcció destí del nostre Caddy, i un altre de totes les direccions IP dels servidors de Cloudflare.

Anirem a Firewall > Aliases, i afegirem un nou álias al final de la  llista.

<img src="..\assets\images\firewall\aliasip.png" alt="Creació d'àlias per a la IP del host" width="700"/>

He creat un álias de nom "Caddy" que apuntarà a la direcció IP del nostre reverse proxy.

<img src="..\assets\images\firewall\aliasip1.png" alt="Creació d'àlias per a la IP del host" width="700"/>

Ara crearem un álias d'un llistat de direccions que es troben a una URL pública. Anirem al menú "URLs", i afegirem una nova entrada.

<img src="..\assets\images\firewall\aliasip1.png" alt="Creació d'àlias per a la llista d'IPs" width="700"/>

<img src="..\assets\images\firewall\aliasip1.png" alt="Creació d'àlias per a la llista d'IPs" width="700"/>

En aquesta llista també tinc creats àlies de llisttes de spam amb pfBlocker.

## Activació del Port Forward
Per a activar la redirecció de ports al port pertinent de la màquina sel·leccionada, anirem a Firewall>NAT, i afegirem un nou port forward a damunt de tot a la llista.

<img src="..\assets\images\firewall\portforward1.png" alt="Creació port forward" width="700"/>

Assignarem la nostra interfície pública, ja que aquest Port Forward és pel tràfic que ve de fora de la nostra xarxa. Sel·leccionarem el protocol TCP, i marcarem el Source com el nostre àlias de les direccions de Cloudflare, des de qualsevol port.

<img src="..\assets\images\firewall\portforward2.png" alt="Creació port forward" width="700"/>

A l'apartat de Destí, aplicarem la nostra adreça de l'interfície WAN, que és on ens arribaran les peticions. Hem d'especificar també que únicament volem habilitar el port 80 i/o el port 443.

També sel·leccionarem a l'apartat Target IP la direcció a la que voldrem redireccionar les peticions que rebrem. Marcarem únicament el port 80 i/o 443.

<img src="..\assets\images\firewall\portforward3.png" alt="Creació port forward" width="700"/>

# Regles de Firewall per a cada interfaç pertinent

Cada regla de Firewall es pot aplicar a una interfície corresponent. Per defecte, un firewall d'una xarxa local no permet absolutament **cap** connexió que comenci de fora de la xarxa cap un equip de dins. Aquest tipus de limitació és perfecta per una xarxa de casa, i serà la que mantindrem a les xarxes OPT1 i LAN.

En canvi, la xarxa OPT2 sí tindrà unes quantes regles necessàries per a permetre el tràfic al port forward que ens hem creat abans.

Per començar, he creat tres regles per a permetre el tràfic que inicia desde la xarxa LAN cap a la xarxa OPT2 amb destinació a qualsevol adreça IP de la xarxa OPT2 als ports 22, 80 i 443, però he denegat el tràfic que comenci desde OPT2 en direcció a la interfície LAN.

Aquestes regles són degudes a que necessito accedir als recursos de gestió interna de les màquines per a configurar-les.
    
<img src="..\assets\images\pfSense\XarxaOPT2_1.png" alt="Creació de VLANs al switch TP-Link TL-SG3210" width="700"/>

> ### Apunt: Lectura de les llistes
> El firewall llegirà sempre les regles de dalt cap abaix, per tant la primera que fa match és la que s'aplica.

També he permès el trafic originari de OPT2 cap a Internet (Tota xarxa que no sigui LAN1, que al ser la primera és automàticament denegada.).

Per últim, l'única regla d'entrada que hem configurat és la que permet que únicament la llista de direccions de Cloudflare accedeixi als recursos del nostre servei.

<img src="..\assets\images\firewall\cloudfirewall.png" alt="Regla de entrada." width="700"/>

La resta de tràfic és automàticament denegada.

# pfBlocker-ng

PFBlocker-NG és un paquet que ens permetrà bloquejar connexions a direccions IP basant-se en llistes públiques i privades de spammers i botnets malicioses conegudes.

Primer de tot, instal·larem el paquet des de System > Package Manager > Avaliable Packages.

<img src="..\assets\images\firewall\pfBlocker1.png" alt="Instal·lació del paquet PFBlocker-NG" width="700"/>

Un pic instal·lat, ens apareixerà a Firewall > pfBlockerNG. Ara, per a que funcioni de manera correcta, hem de configurar el DNS resolver. La idea de PFBlocker és que actui com a primer DNS de la nostra xarxa, eliminant les direccions que trobi en les seves llistes.

Anirem a Services > DNS Resolver > General Settings, i seleccionarem les Network Interfaces que hem d'escoltar, i les Network Interfaces que són Scary Internet.

<img src="..\assets\images\firewall\pfBlocker2.png" alt="Instal·lació del paquet PFBlocker-NG" width="700"/>
<img src="..\assets\images\firewall\pfBlocker3.png" alt="Marcam interfícies d'entrada i sortida" width="700"/>

Ara anirem a Firewall > PFBlockerNG i activarem unes quantes llistes de direccions que no volem que accedeixin al nostre servei. Anirem a DNSBL i activarem el servei.

Buscarem l'opció Virtual IP Address i hi escriurem una direcció que estigui **fora del nostre domini**. Serà la direcció a la que enviarem les peticions DNS que volem rebutjar.

Seleccionarem l'interfície "Localhost" i ens assegurem que l'opció "Permit Firewall Rules" estigui **activa** i escoltant en les tres interfícies a les que volem limitar.

<img src="..\assets\images\firewall\pfBlocker4.png" alt="Marcam interfícies d'entrada i sortida" width="700"/>

Tot seguit, anirem a Feeds i seleccionarem algunes llistes de direccions malicioses que volguem bloquejar. N'hi ha moltes, i quantes més llistes més recursos gastarà el nostre PFSense, així que hem d'anar alerta i seleccionar-les en cura.

Com a exemple, buscarem la llista "Stop Forum Spam", que ens bloqueja direccions de coneguts spammers a forums públics, així no podràn accedir al nostre servei. Recomano les llistes de Spamhaus, Talos, Alienvault, Emerging threats, i MaxMind, m'han donat bon resultat bloquejant anuncis i tracking.

<img src="..\assets\images\firewall\pfBlocker5.png" alt="Llista Stop-Forum-Spam" width="700"/>

Clicarem al símbol + per a afegir-la.

<img src="..\assets\images\firewall\pfBlocker6.png" alt="Llista Stop-Forum-Spam" width="700"/>

Ens portarà a la llista, on baixarem abaix i guardarem. Per a aplicar tots els filtres, anirem a la pestanya Update.

<img src="..\assets\images\firewall\pfBlocker7.png" alt="Actualitzam els filtres" width="700"/>

Podem forçar l'actualització o esperar a que s'actualitzi cada hora.

<img src="..\assets\images\firewall\update.png" alt="Actualitzam els filtres" width="700"/>

Podem veure que la actualització ha estat correcta. Ha trobat canvis a les regles de Firewall ja que hem afegit dues llistes noves, i ha recarregat els filtres.

## Bonus: Bloqueig d'anuncis, tracking i spam general a nivell de xarxa

Com hem pogut veure, a part de llistes d'agents maliciosos, també hi ha llistes de bloqueig d'anuncis. Les hem de aplicar talment hem fet les anteriors, i tot d'una s'actualitzi podrem veure com molts d'anuncis no carreguen, així alliberant la xarxa de tràfic no desitjat. 

<img src="..\assets\images\firewall\msn.png" alt="Pàgina de msn fora anuncis" width="700"/>
