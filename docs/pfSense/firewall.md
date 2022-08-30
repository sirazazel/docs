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


# pfBlocker-ng

PFBlocker-NG és un paquet que ens permetrà bloquejar connexions a direccions IP basant-se en llistes públiques i privades de spammers i botnets malicioses conegudes.

Primer de tot, instal·larem el paquet des de System > Package Manager > Avaliable Packages.
<img src="..\assets\images\firewall\pfBlocker1.png" alt="Instal·lació del paquet PFBlocker-NG" width="700"/>

Un pic instal·lat, ens apareixerà a Firewall > pfBlockerNG

## Bonus: Bloqueig d'anuncis, tracking i spam general a nivell de xarxa