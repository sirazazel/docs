---
layout: default
title: Exposem serveis a Internet
has_children: false
nav_order: 1
---

#  Exposant serveis a Internet de manera segura.

Aquest projecte està dedicat a oferir alguns serveis web públics i evaluar la seva seguretat.

## Què comporta exposar un servei a Internet?

Hem d'ésser conscients de que **qualsevol** servei que intentem exposar a Internet, comportarà notables perills per a la integritat de la nostra xarxa, i per això, exposem el que exposem, haurem d'assegurar-lo de manera adient.

També cal notar que **no hi ha servei exposat segur**, encara que ens centrem en maximitzar la seguretat del mateix, sempre estarem exposts, en major o menor escala.

## Sobre la infraestructura del projecte

Per ara aquest projecte consta de la seguent infraestructura.

- 2x Switch TP-Link TL-SG3210 V1
  - 8x Ports Ethernet 1Gbps
  - 2x Ports SFP 1Gbps
- 1x Dell Optiplex 790 SFF
    - Intel Core2 Duo 8500
    - 6Gb RAM 1600Mtps Kingston ValueRAM
- 1x Servidor de màquines virtuals
  - Intel Core i5 3310 
  - 16Gb RAM 1866Mtps Kingston HyperX
  - 120Gb SSD Kingston 400
  - 2x Realtek 1Gbps NIC
  - 1x GeForce GTX 1050 Ti

## Característiques i serveis

### Serveis actuals
- [x] PFSense
- [x] XCP-NG
- [X] Xen Orchestra virtual Appliance
- [X] pfBlockerNG
- [X] Caddy
- [ ] Plex

- [X] Suricata

### Serveis futurs
- [ ] Prometheus
- [ ] Grafana
- [ ]  TrueNAS

## Stack de tecnologies

|   | Servei                  | Descripció                                                                                      |
|---|-------------------------|-------------------------------------------------------------------------------------------------|
|<img src="assets\logos\pfSenselogo.png" alt="drawing" width="100"/>| pfSense| Conjunt de serveis Router i Firewall de codi obert.|
|<img src="assets\logos\tplinklogo.png" alt="drawing" width="100"/>| tp-link web ui| WebUI emprada per a configurar ambdós switch.|
|<img src="assets\logos\xcplogo.png" alt="drawing" width="30"/>| XCP-NG| Versió Open Source del Citrix Hypervisor. Hosteja tots els nostres serveis excepte el Firewall.|
|<img src="assets\logos\xologo.png" alt="drawing" width="30"/>| Xen Orchestra Community | Eina de gestió web del servidor de màquines virtuals.|
|<img src="assets\logos\pfBlockerlogo.png" alt="drawing" width="30"/>| pfBlockerNG| Eina de bloqueig de llistes IP i DNS.|
|<img src="assets\logos\namecheaplogo.png" alt="drawing" width="100"/>| Namecheap| Registrar del domini.|
|<img src="assets\logos\cloudflarelogo.png" alt="drawing" width="90"/>| Cloudflare| Protecció en front atacs de denegació de servei, túnel, reverse proxy i Servidor DNS.|
|<img src="assets\logos\fedora-coreos-logo.png" alt="drawing" width="80"/> | Fedora CoreOS| Sistema operatiu base per les màquines virtuals de contenidors.|
|<img src="assets\logos\debianlogo.png" alt="drawing" width="100"/>| Debian | Sistema operatiu per la VM de Xen Orchestra Community.|
|<img src="assets\logos\caddylogo.png" alt="drawing" width="100"/>| Caddy | Servidor web i Proxy invers.|
|<img src="assets\logos\letsencrypt.png" alt="drawing" width="100"/>| Let's Encrypt | CA per TLS|

