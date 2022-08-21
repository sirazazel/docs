---
layout: default
title: Xen Orchestra
parent: Hipervisor
nav_order: 2
---

# Buttons
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---


Instalar XOA from sources
# que es xoa
XOA is a virtual machine with Xen Orchestra already installed, thus intended to work out-of-the-box. The only dependency is a running Xen/XCP-ng hypervisor host with network and storage configurations. There is a bash script to be executed on the hypervisor shell which will download VM appliance and create a new Virtual Machine from it.
# creació primera vm
## Crear SR amb isos on afegir sa iso de debian

```bash
mkdir /var/opt/diskimages
sr_path="/var/opt/diskimages"
sr_name="Base ISO Images"
xe sr-create name-label="$sr_name" type=iso device-config:location="$sr_path" \
device-config:legacy_mode=true content-type=iso
    >el-teu-sr-guid
```
Afegir ISO a SR

```bash
cd /var/opt/diskimages
wget "url-descarrega-sistema-preferit"
```
expected result
<img src="..\assets\images\xoa\wget1.png" alt="resultat wget" width="700"/>

## crear vm desde terminal 
creació vm
```bash
xe template-list | grep name-label | grep -i 11
    > name-label ( RW): Debian Bullseye 11
xe vm-install template="Debian Bullseye 11" new-name-label="XOA"
    > el-teu-vm-uuid
```
guardar uuid, nom, xarxa i iso en variables per fer tot mes facil

```bash
UUID=el-teu-uuid
NAME=la-teva-name-label
ISO="nom-arxiu-iso.iso"
NETWORK="uuid-xarxa"
```
tip: ```xe cd-list``` printa les isos disponibles actualment
el mateix passa amb ```xe network-list```

cercam uuid del disc que hem creat amb el vm-install i el guardam com a variable també

```bash
xe vm-disk-list vm="$NAME"
VDI="el-teu-disk-uuid"
```
configuració parametres vm

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
un pic fet això podem arrancar la màquina amb 
```bash
xe vm-start uuid=$UUID
```
i connectar-nos a traves de VNC

## descarregar i compilar projecte
un pic a debian instalar node.js com admin

```bash
apt install curl
curl -fsSL https://deb.nodesource.com/setup_[num darrera versió] | bash -
apt install -y nodejs
apt install -y build-essential
```

instalar yarn

```bash
npm install --global yarn
```

instalar dependencies
```bash
apt install -y redis-server libpng-dev git python3-minimal libvhdi-utils lvm2 cifs-utils
```

You need to use the git source code manager to fetch the code. Ideally, you should run XO as a non-root user, and if you choose to, you need to set up sudo to be able to mount NFS remotes. As your chosen non-root (or root) user, run the following:

root té acces a ports 80 i 443, non-root no. lo seu seria algun port random pero for the sake of simplicity port 80 i arreando
```bash
cd /home
git clone -b master https://github.com/vatesfr/xen-orchestra
cd xen-orchestra
yarn
yarn build
```
yarn instala dependencies de node.js i yarn build compila el projecte 

ojo! minim de ram 2Gb, amb estándar 1Gb:
```bash
FATAL ERROR: Ineffective mark-compacts near heap limit Allocation failed - JavaScript heap out of memory 
```
:(
## configuració projecte
per designar que xo escolti port :80:

```bash
cd packages/xo-server
vim sample.config.toml
```
modificam sample.config.toml i el guardam com config.toml amb les nostres configuracions
```bash
mkdir -p ~/.config/xo-server
cp config.toml ~/.config/xo-server/config.toml
```

revisar que port = 80 estigui descomentat
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
port = 80
```

o bé, si volem certificat autofirmat:

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

emprarem ssl autofirmat, per tant comentam la línia port = 80, ```yarn start``` i ja podem anar a https://172.16.0.20
 expected output:

 <img src="..\assets\images\xoa\xoa1.png" alt="projecte compilat" width="700"/>
 <img src="..\assets\images\xoa\xoa2.png" alt="web" width="700"/>

 això es feo, per tant anirem a crear una entrada al dns resolver de pfSense
 <img src="..\assets\images\xoa\xoa3.png" alt="creació entrada dns" width="700"/>
 abaix de tot d'aquest menú podem afegir entrades al nostre dns resolver (unbound)
 <img src="..\assets\images\xoa\xoa4.png" alt="creació entrada dns" width="700"/>
 <img src="..\assets\images\xoa\xoa5.png" alt="creació entrada dns" width="700"/>

ea

## certificats a pfSense pq ja no digui que son autofirmats?

# connexió amb xcp-ng

 <img src="..\assets\images\xoa\conectarServer.png" alt="conexió xcp-ng" width="700"/>

rellenem els camps i apretam Connect, ens donarà error per certificat autofirmat

<img src="..\assets\images\xoa\conectarServer2.png" alt="conexió xcp-ng" width="700"/>

acceptem i listos

<img src="..\assets\images\xoa\conectarServer3.png" alt="conexió xcp-ng" width="700"/>

ja podrem gestionar des d'aquí totes les vm del nostre servidor

<img src="..\assets\images\xoa\orchestra.png" alt="conexió xcp-ng" width="700"/>
