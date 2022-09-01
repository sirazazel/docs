---
layout: default
title: Reverse Proxy
parent: Hipervisor
nav_order: 4
---

# Reverse Proxy per accedir als nostres serveis

Un pic montada tota aquesta infraestructura, com accedirem als nostres serveis? Tant sigui per a rebre peticions de dins com de fora, un reverse proxy, en essencia, és un servidor que es situa al punt d'entrada d'aquestes peticions, i reemet les peticions cap al destí. Aquest "middle man", ens proporciona uns quants beneficis. Com hem pogut veure amb el proxy invers de Cloudflare, un d'ells és que ens ajuda a augmentar la seguretat del nostre servei dirigint-nos totes les peticions des d'un únic punt.

Un altre benefici és la capacitat de redirigir peticions des d'un punt cap a diferents serveis web. Això ens permet configurar el nostre proxy invers per a redirigir el tràfic a un o varis nodes en funció de la càrrega de treball o del servei que busca el client.

Anem a veure com configurar un proxy invers emprant un servidor web anomenat Caddy.

# Caddy com a servidor web

En essència, Caddy no és més que un servidor web. Té unes quantes ventatges a sobre de Apache o Nginx, la més important per a nosaltres és que és molt simple de configurar emprant els anomenats Caddyfiles, que són els arxius de configuració dels sites que desplegarem, la capacitat per a funcionar com a reverse proxy i balancejador de càrrega, i per últim pero no menys important, gestió automàtica de certificats TLS mitjançant Let's Encrypt.

## Implementació del servidor web
Anem a emprar la màquina virtual que hem creat amb Xen Orchestra, que té Debian 11 instal·lat i l'adreça IP 172.16.0.100 com a punt d'entrada als nostres serveis públics. Emprarem SSH per a la gestió de la màquina.

<img src="..\assets\images\caddy\ssh.png" alt="Esquema tipus hipervisors, a la dreta tius " width="700" style="display: block; margin-left: auto; margin-right: auto;"/>

Instal·larem el servidor web emprant el gestor de paquets APT. Primer de tot, hem d'afegir els repositoris oficials de Caddy.

```bash
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update
sudo apt install caddy
```

Un pic instal·lat, podem arrancar el servei si volguéssim i Caddy ja escoltaria als ports :80 i :443 de la IP del nostre ordinador, servint-nos una pàgina d'exemple.

```bash
# Per a arrancar el servei
systemctl start caddy
# Per a que el servei arranqui al arrancar la màquina
systemctl enable caddy
```
Podem comprovar que servim contingut ràpidament emprant curl

```
curl 172.16.0.100
```
En pantalla veurem el fitxer HTML, similar al seguent

<img src="..\assets\images\caddy\resposta.png" alt="Esquema tipus hipervisors, a la dreta tius " width="700" style="display: block; margin-left: auto; margin-right: auto;"/>

Anem a configurar un Site bàsic emprant el fitxer de configuracions Caddyfile.
## Caddyfile
Començarem servint un missatge estàtic al port 80. Ho farem simple, un "Hello, World!" bastarà.

Crearem un fitxer al directori actual anomenat "Caddyfile", sense extensió.

```bash
mkdir CaddyApp
cd ./CaddyApp
vim Caddyfile
```

Al fitxer nou, escriurem el següent:

```json
:80 {
    respond "Hello, World!"
}
```
Guardarem el fitxer, i, al mateix directori, executarem
```bash
caddy adapt
```

Si el daemon de Caddy està en marxa, automàticament s'aplicarà la connexió. Si no, podem arrancar el procés amb ```caddy run```

Podem veure com ens serveix la pàgina si feim ```curl 172.16.0.100```
<img src="..\assets\images\caddy\resposta2.png" alt="Esquema tipus hipervisors, a la dreta tius " width="300" style="display: block; margin-left: auto; margin-right: auto;"/>

### TLS
Anem a configurar un TLS bàsic mitjançant la configuració automàtica de Let's Encrypt. Per a fer-ho, amb aquesta pàgina web tan simple, bastarà que apuntem el nostre domini cap al servidor amb Caddy, i especifiquem el domini a les configuracions del site del Caddyfile.

```json
www.espardenya.me {
    respond "Hello, World!"
}
```

Així ja tendríem un site amb SSL funcionant. 

<img src="..\assets\images\caddy\resposta3.png" alt="Esquema tipus hipervisors, a la dreta tius " width="300" style="display: block; margin-left: auto; margin-right: auto;"/>

Però aquest tipus de certificats només funcionen en un domini estàndard, no funcionarà amb el nostre domini, ja que ens envia el tràfic a través de Cloudflare. Aquesta limitació fa que el dns challenge que empra Let's Encrypt per a verificar la propietat del domini falli.

Cloudflare ens proporciona un parell de claus, i serà tan simple com descarregar-les al servidor i afegir el seguent al Caddyfile per a emprar-les.

```json
www.espardenya.me {
    tls /ruta/certificate.pem /ruta/key.pem
    respond "Hello, World! We're using Full Strict Encryption"
}
```

Ara sí! Ja podem activar la encriptació Full Strict al panell de Cloudflare i revisar els certificats.

<img src="..\assets\images\caddy\fullstrict.png" alt="Esquema tipus hipervisors, a la dreta tius " width="400" style="display: block; margin-left: auto; margin-right: auto;"/>

<img src="..\assets\images\caddy\cert.png" alt="Esquema tipus hipervisors, a la dreta tius " width="700" style="display: block; margin-left: auto; margin-right: auto;"/>


### Reverse Proxy

Anem a redirigir les peticions que ens arriben a diferents serveis que s'estàn executant als nostres nodes. Començarem creant un site amb el subdomini plex.espardenya.me, i el redirigirem a un únic node que està executant una instància d'aquest servei. 

Modificarem el Caddyfile amb les seguents opcions.

```json
espardenya.me {
    tls /ruta/certificate.pem /ruta/key.pem
}
plex.espardenya.me {
    reverse_proxy 172.16.0.200:32400
}
```
El primer bloc marcarà les opcions pel subdomini arrel. Hi podem posar el certificat, ja que també aplicarà a tots els sites del domini espardenya.me.

Assumint que una instància de Plex està funcionant a la IP sel·leccionada...

<img src="..\assets\images\caddy\plex.png" alt="Esquema tipus hipervisors, a la dreta tius " width="400" style="display: block; margin-left: auto; margin-right: auto;"/>

Voilà! Ja tenim un servidor Plex disponible al món. (bé, a Espanya només).

Anem a aplicar-li algunes opcions de seguretat.

```json
espardenya.me {
    tls /ruta/certificate.pem /ruta/key.pem
    header {
		# Amaga "Server: Caddy"
		-Server
		
		# Mitigam atacs Cross-Site-Scripting (XSS)
		Content-Security-Policy default-src 'self' *.espardenya.me
		
		# Activam bloqueig contra atacs XSS del navegador
		X-XSS-Protection 1; mode=block

		# Obligam a establir una connexió mitjançant HSTS 
		Strict-Transport-Security max-age=31536000; includeSubDomains; preload

		# Protecció contra atacs que emprin mètodes de clickjacking
		X-Frame-Options DENY

		# Deshabilitar iframes
		X-Frame-Options: SAMEORIGIN

		# Deshabilitam mostrar metadades sensibles sobre el contingut
		X-Content-Type-Options nosniff
	}
}
plex.espardenya.me {
    reverse_proxy 172.16.0.200:32400
}
```

### Load Balancer.

Per a intentar millorar la qualitat del nostre servei, anem a crear un petit balancejador de càrrega que ens reparteixi el treball entre dos nodes. 

Modificarem el Caddyfile amb les seguents opcions.

```json
espardenya.me {
    tls /ruta/certificate.pem /ruta/key.pem
    ...
}
plex.espardenya.me {
    reverse_proxy * {
        to 172.16.0.200:32400
        to 172.16.0.201:32400

        lb_policy round_robin
        lb_try_duration 1s
        lb_try_interval 250ms
}
```
Aquesta configuració anirà repartint la càrrega entre els dos nodes de manera igualitària, canviant entre node cada 250ms i esperant resposta del node com a màxim un segon. 

Anem a comprovar-ho.

Apagarem ambdós contenidors a cada host, emprant ```docker-compose down```

Podem veure com plex.espardenya.me no resol enlloc.

<img src="..\assets\images\caddy\reverse_proxy_prova.png" alt="Esquema tipus hipervisors, a la dreta tius " width="700" style="display: block; margin-left: auto; margin-right: auto;"/>

Anem a encendre el primer servei. Funciona!

<img src="..\assets\images\caddy\reverse_proxy_prova1.png" alt="Esquema tipus hipervisors, a la dreta tius " width="700" style="display: block; margin-left: auto; margin-right: auto;"/>

Ara encendrem el segon servei i apagarem el primer.

<img src="..\assets\images\caddy\reverse_proxy_prova2.png" alt="Esquema tipus hipervisors, a la dreta tius " width="700" style="display: block; margin-left: auto; margin-right: auto;"/>

El servei segueix resolent.