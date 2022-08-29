---
layout: default
title: Hipervisor
has_children: true
nav_order: 3
---

# Hipervisor.

Un hipervisor és un conjunt d'eines, tant software com podríen ser de hardware, que ens permeten particionar els recursos d'una única màquina física en una o un conjunt de programes o sistemes independents. 

## Breu introducció als hipervisors.

En la història, han estat comuns dos tipus d'hipervisors, depenent de la seva arquitectura.

<img src="..\assets\images\xcp\hyperviseur.png" alt="resultat wget" width="300" style="display: block; margin-left: auto; margin-right: auto;"/>

Coneguts com Hipervisors de tipus 1, són aquells que el "Virtual Machine Monitor" s'executa directament a sobre del hardware, fent així tots els programes i sistemes executats en aquell dispositiu siguin en un entorn virtualitzat, maximitzant els recursos que ens pot oferir la màquina. 

Són els mes emprats en datacenters i per grans empreses ja que són més eficients en la gestió dels recursos.

En el nostre cas, estarem emprant un hipervisor de tipus 1 (XCP-ng/Citrix Hypervisor) que correrà bare-metal en el nostre servidor de casa. També existeixen altres hipervisors, com KVM o VMWare ESXi que ofereixen una tecnologia semblant.

En canvi, els Hipervisors de tipus 2, són aquells que s'executen damunt un sistema operatiu, és a dir, són com un programa més en l'ordinador. Hipervisor tals com VMWare Workstation o QEMU en són exemples.

## Xen Server i XCP-ng

He elegit XCP-ng és una plataforma basada en XenSource, la versió OpenSource de Citrix Hypervisor. És una solució pràctica, open source, i bastant senzilla d'implementar.

Una de les raons més importants ha estat que els desenvolupadors de XCP-ng també donen suport a hardware que no té per què ser server-class, i ja que el meu servidor serà un ordinador normal de fa uns anys, necessitam la màxima compatibilitat.

## Instal·lació XCP-ng

La instal·lació de XCP-ng es fa com si fos un sistema operatiu normal. Hem seguit aquest enllaç:

[Guia d'instal·lació oficial ](https://xcp-ng.org/docs/install.html#iso-installation)

Hem configurat al servidor dos NIC's connectats a la mateixa subxarxa, per així conseguir redundància i capacitat per a gestionar més tràfic, amb IP 172.16.0.10.

Un pic feta la instal·lació, el servidor és utilitzable tant vía SSH emprant xe cli, com mitjançant una consola d'administració bàsica anomenada ```xsconsole```.

<img src="..\assets\images\xcp\welcome.png" alt="resultat wget" width="700" style="display: block; margin-left: auto; margin-right: auto;"/>

<img src="..\assets\images\xcp\sshaccess.png" alt="resultat wget" width="700" style="display: block; margin-left: auto; margin-right: auto;"/>

Com podem veure, mitjançant SSH podrem fer absolutament totes les gestions necessàries per a començar a crear màquines virtuals i gestionar les mateixes. A l'apartat [Creem una màquina virtual de Xen Orchestra](./Gesti%C3%B3%20amb%20Xen%20Orchestra.md#creació-primera-vm) veurem com crear i arrancar una màquina virtual mitjançant la consola SSH.

Amb ```xsconsole``` entrarem a una eina de gestió GUI per a fer configuracions bàsiques al servidor. Ja sigui canviar la direcció IP del servidor, com crear una nova "Storage Pool" per a màquines virtuals ho podrem fer des d'aquí.

<img src="..\assets\images\xcp\xsconsole.png" alt="resultat wget" width="700" style="display: block; margin-left: auto; margin-right: auto;"/>

Totes aquestes opcions per a gestionar el servidor són vàlides, però el més senzill es emprar l'eina de gestió oficial [Xen Orchestra](./Gesti%C3%B3%20amb%20Xen%20Orchestra.md), o eines de tercers com xcp-ng Center, que és un programa de gestió de Windows que es connecta al servidor XCP.
