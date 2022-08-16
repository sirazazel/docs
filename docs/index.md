#  Exposant serveis a Internet de manera segura.

Aquest projecte està dedicat a oferir alguns serveis web des d'un "Homelab". 

> ### Què es un "Homelab"?
>És tenir un (o alguns) servidors a casa on es podrà experimentar amb noves tecnologies a petita escala, allotjar llocs o serveis web tant en l'àmbit local com a Internet.

## Què comporta exposar un servei a Internet?

Hem d'ésser conscients de que **qualsevol** servei que intentem exposar a Internet, comportarà notables perills per a la integritat de la nostra xarxa, i per això, exposem el que exposem, haurem d'assegurar-lo de manera adient.

També cal notar que **no hi ha servei exposat segur**, encara que ens centrem en maximitzar la seguretat del mateix, sempre estarem exposts, en major o menor escala.

## Sobre el Homelab

El meu Homelab per ara consta de la seguent infraestructura.

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
- [ ] Caddy
- [ ] Plex
- [ ] Prometheus
- [ ] Grafana
- [ ] Snort
### Serveis futurs
- []  TrueNAS

## Stack de tecnologies

|   | Servei                  | Descripció                                                                                      |
|---|-------------------------|-------------------------------------------------------------------------------------------------|
|   | pfSense                 | Conjunt de serveis Router i Firewall de codi obert.                                             |
|   | tp-link web ui          | WebUI emprada per a configurar ambdós switch.                                                   |
|   | XCP-NG                  | Versió Open Source del Citrix Hypervisor. Hosteja tots els nostres serveis excepte el Firewall. |
|   | Xen Orchestra Community | Eina de gestió web del servidor de màquines virtuals.                                           |
|   | pfBlockerNG             | Eina de bloqueig de llistes IP i DNS.                                                           |
|   | Namecheap               | Registrar del domini.                                                                           |
|   | Cloudflare              | Protecció en front atacs de denegació de servei, túnel, reverse proxy i Servidor DNS.           |
|   | Fedora CoreOS           | Sistema operatiu base per les màquines virtuals de contenidors.                                 |
|   | Debian                  | Sistema operatiu per la VM de Xen Orchestra Community.                                          |
|   | Caddy                   | Servidor web i Proxy invers.                                                                    |
|   | Let's Encrypt           | CA per TLS                                                                                      |

### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [Basic writing and formatting syntax](https://docs.github.com/en/github/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/sirazazel/espardenya.cloud/settings/pages). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.
