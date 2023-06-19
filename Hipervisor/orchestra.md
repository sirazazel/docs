---
layout: default
title: Xen Orchestra
parent: Hipervisor
nav_order: 2
---

# Xen Orchestra: Com gestionar les màquines virtuals del nostre hipervisor.

Xen Orchestra Appliance, que anomenarem XOA a partir d'ara, és l'eina de gestió oficial de l'hipervisor XCP-ng. Funciona com una màquina virtual dins el nostre servidor, a la que accedirem tant per SSH si necessitàssim fer algun tipus de manteniment al sistema, com pels ports 80 o 443, on mitjançant una eina web, podrem gestionar per complet totes les màquines virtuals del nostre servidor.

XOA és Open Source també, però amb una versió de pagament que s'autoinstal·la desde XCP-ng. En el nostre cas, com que volem emprar la versió gratuita, compilarem XOA des del codi font, que podrem trobar [a Github](https://github.com/vatesfr/xen-orchestra).

Per tant, necessitarem crear una màquina virtual amb el sistema Linux que s'adapti a les nostres necessitats. Jo he elegit Debian 11, ja que es un sistema complet i estable però amb els paquets mínims, no vull afegir dependències innecessàries al sistema. 

## Anem a crear una VM desde xe-cli
### Obtenir el medi d'instal·lació
Primer de tot, necessitarem el medi d'instal·lació del nostre sistema base. XCP-ng ens permetrà crear Storage Resources (SR) que són carpetes amb informació que pot ésser compartida entre el host i les màquines virtuals. Al host, en crearem una anomenada "Base ISO Images" per als medis d'instal·lació. 

```bash
mkdir /var/opt/diskimages
sr_path="/var/opt/diskimages"
sr_name="Base ISO Images"
xe sr-create name-label="$sr_name" type=iso device-config:location="$sr_path" \
device-config:legacy_mode=true content-type=iso
    >el-teu-sr-guid
```

Un cop hem creat la SR, podem descarregar la ISO emprant els programes ```curl``` o ```wget```. A CentOS, que és el sistema base de XCP-ng, podrem emprar wget sense haver d'instal·lar el paquet, ja que ve preinstal·lat.

```bash
cd /var/opt/diskimages
wget "url-descarrega-sistema-preferit"
```
Un cop descarregat, veurem algun missatge similar a aquest al terminal.
<img src="..\assets\images\xoa\wget1.png" alt="resultat wget" width="700"/>

### Maquetar la VM des d'una plantilla 

XCP-ng disposa de plantilles estàndard que ens crearàn una VM amb els requisits mínims per a funcionar amb el sistema que li diguis. Ens és molt útil, ja que així ens asseguram de que no ens deixam res per configurar.

Buscarem i instal·larem una plantilla de Debian amb les seguents comandes:
```bash
xe template-list | grep name-label | grep -i 11
    > name-label ( RW): Debian Bullseye 11
xe vm-install template="Debian Bullseye 11" new-name-label="XOA"
    > el-teu-vm-uuid
```

Anem a personalitzar la plantilla amb els requisits per executar-hi XOA.
Primer de tot, guardarem algunes dades com a variables d'entorn per a fer-nos la vida més senzilla.

```bash
UUID=el-teu-uuid
NAME=la-teva-name-label
```
La comanda ```xe cd-list``` ens llistarà el nom de les ISO que estàn disponibles dins un SR, i la comanda ```xe network-list``` ens donarà la UUID de les xarxes a les que puguem afegir màquines virtuals.
```bash
ISO="nom-arxiu-iso.iso"
NETWORK="uuid-xarxa"
```
Hem d'obtenir el disc virtual bàsic que ens ha creat la plantilla de Debian per a ampliar-lo a 15Gb, que és el que ens recomana Xen Orchestra. El podem cercar de la seguent manera:

```bash
xe vm-disk-list vm="$NAME"
VDI="el-teu-disk-uuid"
```
Amb les anteriors dades recollides, podem modificar la màquina virtual plantilla i adaptar-la al nostre ús.
```bash
xe vm-cd-add uuid="$UUID" cd-name=$ISO device=1
# seleccionam mv amb uuid guardat i inserim el disc d'instal·lació de debian
xe vm-param-set HVM-boot-policy="BIOS order" uuid=$UUID
# modificam el boot order de la mv amb UUID seleccionat.
xe vif-create vm-uuid=$UUID network-uuid=$NETWORK device=0
# cream una interfície de xarxa virtual per a la mv
xe vm-memory-limits-set static-max 2000MiB static-min=512MiB
# assignam ram maquina virtual
xe vdi-resize uuid=$VDI disk-size=15GiB
# ampliam disc a 15 Gb, el mínim per XOA
```

Un cop modificat, la màquina està llesta per a ser arrancada i instal·lada mitjançant VNC o SSH

```bash
xe vm-start uuid=$UUID
```

Jo he emprat el client VNC que ve amb l'eina [XCP-ng Center](https://github.com/xcp-ng/xenadmin/releases/tag/v20.04.01.33). Es connecta directament a cada host i ens permet fer la instal·lació de manera senzilla.

<img src="..\assets\images\xoa\xcp-ngcenter.png" alt="xcp-ng Center mode client vnc" width="700"/>


### Arrancada de la màquina virtual i preparatius

Xen Orchestra necessita certes dependències, que les podem resoldre instal·lant des del packet manager de la nostra distribució escollida.

```bash
apt install curl
curl -fsSL https://deb.nodesource.com/setup_[num darrera versió] | bash -
apt install -y nodejs
apt install -y build-essential
apt install -y redis-server libpng-dev git python3-minimal libvhdi-utils lvm2 cifs-utils
npm install --global yarn
```

Emprarem Git per a descarregar el projecte Orchestra. El millor seria compilar i executar Orchestra en un perfil no privilegiat, però si ho feim d'aquesta manera no podrem accedir als ports 80 ni 443, ni montar cap NFS Share. Podríem emprar un port no conegut, i crear els shares per SSH, però per a fer-ho més simple emprarem el port 443 en un usuari amb privilegis adients.

```bash
cd "/directori-escollit"
git clone -b master https://github.com/vatesfr/xen-orchestra
cd xen-orchestra
yarn
yarn build
```

Un pic executat, ```yarn``` ens ha instal·lat totes les dependències del projecte, i ```yarn build``` ens ha compilat el projecte amb tots els components adients.

## Configuració del servei web

Totes les configuracions bàsiques que vulguem fer al servidor web que allotjarà l'eina XOA, es faràn modificant el seguent fitxer d'exemple.

```bash
cd packages/xo-server
vim sample.config.toml
```

Hem de modificar el fitxer ```sample.config.toml``` amb les configuracions adients i guardar-lo com a ```config.toml```, al directori ```~/.config/xo-server```

```bash
mkdir -p ~/.config/xo-server
cp config.toml ~/.config/xo-server/config.toml
```

Per a la meva configuració, vull que el meu XOA escolti al port 443 i emplei un certificat TLS 1.3 autosignat, per tant, m'asseguraré de que l'opció del port 80 estigui comentada

```toml
# Basic HTTP.
[[http.listen]]
# Address on which the server is listening on.
#
# Sets it to 'localhost' for IP to listen only on the local host.
#
# Default: all IPv6 addresses if available, otherwise all IPv4 addresses.
# hostname = 'localhost'

# Port on which the server is listening on.
#
# Default: undefined
# port = 80
```

I les opcions d'HTTPS bàsic descomentades.

```toml
# # Basic HTTPS.
# # The only difference is the presence of the certificate and the key.
# [[http.listen]]
# #hostname = '127.0.0.1'
port = 443
# # Whether to autogenerate a self signed certificate if the `cert` or `key`
# # files could not be found.
autoCert = true
#
# # File containing the certificate (PEM format).
# #
# # If a chain of certificates authorities is needed, you may bundle them
# # directly in the certificate.
# #
# # Note: the order of certificates does matter, your certificate should come
# # first followed by the certificate of the above
# # certificate authority up to the root.
# #
# # Default: undefined
cert = './certificate.pem'
#
# # File containing the private key (PEM format).
# #
# # If the key is encrypted, the passphrase will be asked at
# # server startup.
# #
# # Default: undefined
key = './key.pem'
```
Ja podem executar el projecte, emprant la comanda ```yarn start```, i anar a la direcció IP que hagem configurat per a la màquina per a accedir al servei.

 <img src="..\assets\images\xoa\xoa1.png" alt="projecte compilat" width="700"/>
 <img src="..\assets\images\xoa\xoa2.png" alt="web" width="700"/>

### Entrada a Unbound

És complicat recordar direccions IP, i més poguent aprofitar les característiques del nostre PFSense. Anem a configurar una entrada al DNS Resolver per a poder entrar a la Virtual Appliance amb una direcció URL estàndard privada.

Entrarem al Router, a Services > DNS Resolver.
 <img src="..\assets\images\xoa\xoa3.png" alt="creació entrada dns" width="700"/>
Baixarem al final del tot, on ens sortirà l'opció de crear una nova entrada al DNS.
 <img src="..\assets\images\xoa\xoa4.png" alt="creació entrada dns" width="700"/>
A partir d'ara les petiicions a orchestra.espardenya.local resoldràn a la IP assignada (172.16.0.20),
 <img src="..\assets\images\xoa\xoa5.png" alt="creació entrada dns" width="700"/>

## Connectar la Virtual Appliance al servidor XCP-ng
Per a poder gestionar els servidors o nodes a XCP-ng, els hem d'afegir. Serà tan senzil com 
clicar Add server i afegir les dades que ens demanen.
<img src="..\assets\images\xoa\conectarServer.png" alt="conexió xcp-ng" width="700"/>
<img src="..\assets\images\xoa\conectarServer3.png" alt="conexió xcp-ng" width="700"/>
Com que el certificat SSL no és públic sinó autosignat, Orchestra ens demanarà si l'acceptem igualment abans de carregar totes les dades.
<img src="..\assets\images\xoa\conectarServer2.png" alt="conexió xcp-ng" width="700"/>
Un pic acceptat ja podrem gestionar el servidor.
<img src="..\assets\images\xoa\orchestra.png" alt="conexió xcp-ng" width="700"/>

## Creació d'una màquina virtual mitjançant Xen Orchestra.

Per a crear una màquina virtual nova des d'Orchestra, anirem al panell principal. Ens apareixerà el llistat de màquines virtuals com hem vist a la captura anterior, i adalt a la dreta clicarem al botó New VM.

<img src="..\assets\images\xoa\vm_nova.png" alt="creacio nova vm" width="700"/>

Talment hem fet amb la versió CLI, ens hem d'assegurar de descarregar la ISO d'instal·lació mitjançant curl a la SR que toca. En aquest cas, seleccionarem la iso de Debian 11 a l'apartat Install Settings, i pitjarem Create.

La nostra nova màquina virtual s'autoarrancarà i apareixerà al llistat. Ja podrem clicar damunt de la màquina per anar a veure l'estat general de la mateixa.

<img src="..\assets\images\xoa\vm_nova1.png" alt="estat nova vm" width="700"/>

Podem anar a la pestanya Console i procedir amb l'instal·lador del sistema.

<img src="..\assets\images\xoa\vm_nova2.png" alt="consola nova vm" width="700"/>

A l'apartat Network, podem crear un NIC nou per si fos necessari canviar la màquina de VLAN.

<img src="..\assets\images\xoa\vm_nova3.png" alt="consola nova vm" width="700"/>

Jo li he assignat la interfície connectada a la VLAN 20, que correspon a la xarxa aïllada OPT2 del meu sistema.

## Conclusió

Xen Orchestra és una eina bastant senzilla d'emprar i molt pràctica per a gestionar el servidor XCP-ng i les màquines virtuals que s'executen a sobre. Ens fa molt més senzilla la gestió que no el sistema xe-cli que hem vist a l'apartat anterior.