---
layout: default
title: Clúster management
parent: Hipervisor
nav_order: 5
---

# Escalant el nostre servei web.

Posem el cas que el nostre servei web està éssent molt soli·licitat, i, amb tantes peticions, la disponibilitat del nostre servei es veu en perill. Per això, anem a muntar un petit clúster de nodes que executaràn serveis Docker de manera independent.

Emprarem Docker Swarm, que és l'eina d'orquestació de contenidors que està incluida al Docker Engine. 

## Com es configura una Swarm?

### Nodes

Una Swarm consisteix en un grup de nodes que poden executar tasques de manera organitzada. Cada node agafa un rol, tant sigui Manager, Worker, o els dos alhora. Els Managers s'encarreguen d'assignar cada tasca a un node apropiat i de gestionar que en cada moment hi hagi un mínim d'instàncies en marxa per a poder atendre les peticions.

Per altra banda, els nodes s'encarreguen d'executar els serveis assignats pels managers.

### Serveis

Un servei és una instància d'una tasca a executar als nodes. Hi ha dos tipus principals de serveis, els serveis replicats i els serveis globals.

#### Serveis globals

Un servei global és un servei que s'executa un únic cop a cada node disponible del clúster

Per exemple, seguint aquesta definició de desplegament, el servei s'executarà en tots els nodes amb el rol de treballador.

```yaml
    deploy:
        mode: global
        placement:
            constraints:
                - node.role == worker
```

#### Servei replicat

Un servei replicat és un servei que s'executa al node més adient fins a arribar al número de rèpliques que s'espera segons la definició de servei.

```yaml
    deploy:
        mode: replicated
        replicas: 4
        placement:
            constraints:
                - node.role == manager
```

En aquest exemple, a la swarm s'executaràn quatre instàncies del servei definit entre tots els managers.

## Clúster de 4 nodes

Anem a crear un clúster de quatre nodes, amb dos Managers i dos Workers. L'ideal seria repartir els nodes entre servidors, per a aconseguir protecció en front a una caiguda del servidor general, però per restriccions del hardware crearem els quatre nodes al nostre servidor XCP-ng.

Crearem quatre màquines virtuals executant Fedora CoreOS, que és un sistema orientat a l'execució de contenidors amb Docker preinstal·lat.

### Creació i implementació de la VM

El procés de creació de la VM és el mateix que hem vist a l'apartat de creació de VMs a Xen Orchestra. Clonarem 5 VMs iguals i els canviarem el nom de domini.

<img src="..\assets\images\orquestacio\vms.png" alt="llistat vms" width="700"/>

He creat 5 nodes ja que necessitam alguna manera de tenir el mateix directori d'arxius a tots i cada un dels nodes, per tant també crearem un NFS molt simple.

> #### Apunt: NFS
> NFS és un protocol de la capa d'aplicació que ens permetrà montar un sistema de fitxers distribuit com si es tractàs d'un sistema local.
> Es compon d'un o més clients i un servidor de fitxers.

### Muntatge del servidor de fitxers

Ens assegurarem de tenir instal·lats els paquets adients per a fer funcionar un nfs share al nostre sistema. En sistemes basats en RPM, el paquet del servidor s'anomena ```nfs-storage``` i el paquet que proporciona l'eina client s'anomena ```nfs-utils```.

A Fedora CoreOS ja venen preinstal·lats, però si no fos el cas, els instal·laríem amb les seguents comandes.

```bash
# Servidor
sudo rpm-ostree install nfs-storage
# Client
sudo rpm-ostree install nfs-utils
```
#### Servidor

Crearem una carpeta al directori Home, que s'anomenarà plex, la designarem com a carpeta fora usuari i grup i donarem permisos a tothom per a modificar-la.

```bash
cd /home
# Aplicam els permisos a la carpeta compartida
sudo mkdir -p /plex && sudo chown nobody:nogroup /home/plex && sudo chmod 777 /home/plex
# Assignam la carpeta compartida com un export nfs
sudo echo "/home/plex *(rw,sync,no_subtree_check)" >> /etc/exports
# Reiniciam el daemon nfs per a veure aplicats els canvis
sudo exportfs -a && sudo systemctl restart nfs-kernel-server
```

> Nota: Aquest sistema no és gaire segur ja que permetríem a tothom que conegui el share modificar les dades sense preguntar, però per a les proves ens serà suficient.
>
> Tampoc és un sistema lo suficientment ràpid per a passar a entorns de producció més enllà que aquestes proves.
>
> Hauríem de pensar en implementar un servidor de dades iSCSI, o glusterfs, o similars.

#### Client

Crearem la carpeta al directori home i muntarem el NFS.

```bash
sudo mkdir -p /home/plex
# Muntam la carpeta compartida
sudo mount IP_OPT3:/home/plex /home/plex
```

Podem verificar com ens aparéixen les dades a tots els nodes amb la seguent comanda, que ens printarà el directory tree.
```bash
find . | sed -e "s/[^-][^\/]*\// |/g" -e "s/|\([^ ]\)/|-\1/"
```

<img src="..\assets\images\orquestacio\nfs.png" alt="arbre de directoris" width="700"/>

Els nostres nodes ja tenen un sistema de fitxers compartit des d'on crear els volums. Anem a assignar els serveis a cadascun d'ells.

### Creació de la swarm.

#### Manager #1

Farem SSH a la màquina la qual hem designat per a ser el mànager líder del nostre clúster i executarem el seguent:

```bash
docker-swarm init --advertise--addr IP_LÍDER_OPT3
```

Ens generarà una resposta similar a la seguent:

<img src="..\assets\images\orquestacio\swarmtoken.png" alt="arbre de directoris" width="700"/>

Copiarem la comanda i la guardarem.

Ara executarem ```docker swarm join-token manager``` per a treure el token d'entrada a més mánagers, i el guardarem també per a futur ús.

#### Manager #2

Executarem el token d'entrada del node mànager, i si estàn a la mateixa xarxa local el node entrarà a la swarm.

#### Workers #1 i #2

Executarem el token d'entrada a ambdós workers i seràn admesos al clúster.

#### Estat dels nodes

Podem veure l'estat dels nodes executant ```docker node ls```

<img src="..\assets\images\orquestacio\nodes.png" alt="arbre de directoris" width="700"/>

### Execució de serveis.

A partir d'ara, podem gestionar tot des de qualsevol dels dos managers. Hem configurat el Reverse Proxy per que envii les peticions a qualsevol dels dos mànagers, així encara que algun dels dos caigui seguirem tenint accés al servei mentre el node es reinicia.

#### Docker Compose per executar serveis a la swarm

Podem modificar un arxiu Compose amb les especificacions de Deploy per a assignar quantes instàncies i quines restriccions van a cada servei. 

Modificant el compose de l'apartat Plex, quedaria així:

```yaml
version: '3.9'
services:
  plex:
    image: plexinc/pms-docker
    deploy:
    # Deu rèpliques a qualsevol dels nodes
      replicas: 10
    # Aquesta directiva vol dir que només un contenidor s'actualitza alhora, i només cada 5 segons es pot començar a actualitzar un contenidor.
      update_config:
        parallelism: 1
        delay: 5s
        order: start-first
    # S'intentarà arrancar un contenidor que falli per qualsevol motiu cada 10 segons, fins a un màxim de 10 vegades. Després es descarta.
      restart_policy:
        condition: any
        delay: 10s
        max_attempts: 10
        window: 15s
    ports:
      - 32400:32400/tcp
      - 3005:3005/tcp
      - 8324:8324/tcp
      - 32469:32469/tcp
      - 1900:1900/udp
      - 32410:32410/udp
      - 32412:32412/udp
      - 32413:32413/udp
      - 32414:32414/udp
    environment:
      - TZ=Europe/Madrid
      - PLEX_CLAIM=claim-ey6ekAqeQjosd1P
    hostname: plex.espardenya.me
    volumes:
      - ./plex/database:/config
      - ./plex/transcode:/transcode
      - ./plex/media:/data
```

Anem a provar d'executar aquest Compose.

```bash
docker stack deploy -C docker-compose.yml myPlexService
```

Amb la comanda ```docker stack ls``` podem veure que el servei s'ha creat correctament. I amb la comanda ```docker service ls``` podrem veure si els serveis s'han desplegat.

<img src="..\assets\images\orquestacio\servicedeployed.png" alt="arbre de directoris" width="700"/>


La comanda ```docker stack ps nomServei``` ens mostrarà a quin node s'estàn executant els serveis.

<img src="..\assets\images\orquestacio\servicedeployed2.png" alt="arbre de directoris" width="700"/>

Els mateixos nodes manager s'encarregaràn de fer un balanceig de càrrega equitatiu per a sempre mantenir 10 nodes disposats a donar el servei adient.