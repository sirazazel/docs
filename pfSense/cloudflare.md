---
layout: default
title: Cloudflare
parent: Firewall
grand_parent: Exposem serveis a Internet
nav_order: 3
---

# Dominis, registres DNS, i com podem rebre requests del món

En aquesta secció veurem com ens poden arribar peticions del món cap al nostre servidor web, sigui el que sigui el servei que estem servint. Realment, seria molt senzill, ja que per a estar disponibles com a recurs a Internet, només bastarà que configurem un Port Forwarding al nostre firewall, i mitjançant la combinació IP pública + Port, el nostre recurs pot ésser accedit des de qualsevol part del món.

Però, no ens convé anar per lo fàcil. Un port públic al nostre firewall és sinònim d'una porta d'entrada a la nostra xarxa per a tota casta de peticions, siguin vàlides o malicioses. Per això ens convé configurar unes certes regles d'entrada de tràfic per mirar de posar-ho complicat a qui vulgui entrar amb males intencions.

Començarem configurant un nom de domini. Així, podrem filtrar totes les peticions que busquin una IP pública, en comptes d'un domini del tipus ```*.domini.tld.```

En el nostre cas, vam comprar el domini al registrar Namecheap amb bonos de Github Edu.

<img src="..\assets\images\cloudflare\namecheap1.png" alt="Entrada espardenya.me al registrar del domini." width="700"/>

Ara, éssent propietaris del domini ```.espardenya.me.```, podríem redirigir el tràfic desde el domini fins a la nostra IP pública, i amb un reverse proxy filtrar totes les peticions que no vinguin al domini.

Però, nosaltres emprarem els serveis de Cloudflare.

## Cloudflare: més que un registrar

Així com Namecheap ens proporciona només resolució de noms, DNSSEC i poca cosa més, Cloudflare ens proporciona molts de serveis que ens permeten assegurar una mica més les peticions que ens arriben a la nostra xarxa.

Per començar, ens proporcionarà 5 regles de Firewall personalitzables de manera gratuïta, protecció contra DDoS, protecció contra peticions de crawlers i bots, i, el més important, un proxy que ens assegurarà que totes les peticions al nostre domini tindràn la meva IP pública enmascarada.

## Com transferim el domini a Cloudflare?

Per que Cloudflare operi el domini escollit, haurem d'indicar a la plataforma de Namecheap que els Nameservers han canviat. 

Primer de tot, anirem a la [plataforma de Cloudflare](dash.cloudflare.com) i afegirem un nou Site. Escrivint el nostre domini ens proporcionaràn els nameservers més propers i més ràpids per el nostre cas.

<img src="..\assets\images\cloudflare\transfer1.png" alt="Afegir nou site a cloudflare." width="700"/>

A nosaltres ens han donat aquests dos nameservers:

```
ignacio.ns.cloudflare.com
melinda.ns.cloudflare.com
```

Anirem al nostre registrar original i els aplicarem com a Nameservers externs

<img src="..\assets\images\cloudflare\transfer2.png" alt="Canvi NS a Namecheap." width="700"/>

Ara només es questió d'esperar unes hores que es faci la transferència entre els dos, i ja podrem gestionar el domini desde la nova plataforma.

## Anem a crear registres DDNS a través del proxy de Cloudflare.

Volem mantenir la nostra IP pública privada. Això és possible gràcies al proxy que tenim disponible a Cloudflare, que  farà que la nostra direcció no la vegi ningú més que nosaltres. Aquest proxy té altres beneficis associats, com que ens assegura que totes les peticions legítimes provinguin de la direcció IP dels seus servidors, i una caché de la nostra pàgina web que ens alliberarà cert nivell de càrrega al nostre host, que no és gaire bo.

> ### Apunt: DDNS
> Normalment, els ISP assignen direccions de manera dinàmica emprant el protocol DHCP als seus clients. Si la nostra IP canvia, haurem d'estar atents a aquest canvi que és pràcticament imprevisible per a canviar el registre al nostre panell de control del domini.
>
> O bé, podem emprar un client DDNS que el que farà és monitoritzar la direcció IP pública de la nostra xarxa per a, en el moment que canvii, assignar-la al nostre gestor de noms de domini, i així automatitzar aquesta tasca i assegurar-nos una major disponbilitat.

### Generam un Token i una Zona DNS que modificarem dinàmicament

Primer de tot, crearem un registre DNS a l'apartat DNS de la [plataforma](dash.cloudflare.com)

Aquest registre ha de tenir les seguents característiques:
* Registre de tipus A
* Domini o subdomini
* Una IP falsa.
* Cloudflare Proxy DESACTIVAT

Per exemple, crearem un registre A per al subdomini test, que apunti a la direcció 1.1.1.1

<img src="..\assets\images\cloudflare\cloudflare1.png" alt="Alta subdomini" width="700"/>

Ara, necessitarem treure un Token per donar accés al nostre PFSense per a canviar el registre que acabam de fer amb les dades vàlides. Anirem a My Profile > API Tokens

<img src="..\assets\images\cloudflare\createToken.png" alt="Alta subdomini" width="700"/>

Crearem un nou Token amb els permisos per a editar les Zones DNS que nosaltres li indiquem.
<img src="..\assets\images\cloudflare\createToken2.png" alt="Alta subdomini" width="700"/>

Crearem el Token i confirmarem les dades, i se'ns brindarà un codi que no es podrà veure més que un pic, per seguretat, així que ens hem d'assegurar de guardar a una keyring o bé emprar-lo directament per la seva funció i ja està.

### Apliquem el client DDNS a PFSense

Ara configurarem el nostre Firewall, per que monitoritzi la direcció IP que ens brinda la ISP i la vagi actualitzant periòdicament.

Anirem a Services > Dynamic DNS

<img src="..\assets\images\cloudflare\ddnsConfig1.png" alt="Alta subdomini" width="700"/>
Clicarem a Add per a afegir un nou recurs

<img src="..\assets\images\cloudflare\ddnsConfig2.png" alt="Alta subdomini" width="700"/>

Afegirem les dades del nostre compte de Cloudflare, i ens assegurarem de deixar marcat 

- [X] Enable Proxy

per a poder emprar els beneficis del firewall i protecció contra atacs DDoS de Cloudflare.

En breus hauriem de veure com s'ha actualitzat la direcció IP a la plataforma de Cloudflare, i està activat el proxy.

<img src="..\assets\images\cloudflare\ddnsConfig3.png" alt="Alta subdomini" width="700"/>

## Anem a crear algunes regles de Firewall.

### Permetre el tràfic només de clients espanyols.
Podem veure a les analítiques que hem començat a rebre peticions de tot el món al nostre domini. Aquí és on hem de pensar si realment el nostre servei web necessita estar disponible a tot el món, o si basta algun lloc en concret. En el meu cas, dubt que sigui necessari fer la pàgina web accessible des de fora d'Espanya, així que aquesta serà la primera regla del nostre primer Firewall.

És una regla bastant senzilla d'aplicar, ja que només necessitam que el tràfic passi si el tag GeoIP del datagrama és igual al codi "ES", i si és distint, es denegarà el tràfic.

<img src="..\assets\images\cloudflare\españita.png" alt="Cream una regla que només permet trafic originari a Espanya." width="700"/>

```
(ip.geoip.country ne "ES")
```

### Bloquejar peticions que provinguin d'User Agents no vàlids coneguts.

Molt de tràfic d'Internet es genera de manera automàtica mitjançant crawlers i bots que van escanejant la xarxa fins a trobar alguna porta oberta. Anem a prohibir el pas d'aquests, almenys dels més coneguts.

Per exemple, Google i tot té crawlers per indexar pàgines web al seu cercador, però no tots aquests programes tenen intencions tan "inocents"

<img src="..\assets\images\cloudflare\badbots.png" alt="Cream una regla que només permet trafic des d'User Agents vàlids." width="700"/>

```
(http.user_agent contains "Yandex") or (http.user_agent contains "muckrack") or (http.user_agent contains "Qwantify") or (http.user_agent contains "Sogou") or (http.user_agent contains "BUbiNG") or (http.user_agent contains "knowledge") or (http.user_agent contains "CFNetwork") or (http.user_agent contains "Scrapy") or (http.user_agent contains "SemrushBot") or (http.user_agent contains "AhrefsBot") or (http.user_agent contains "Baiduspider") or (http.user_agent contains "python-requests") or (http.user_agent contains "crawl" and not cf.client.bot) or (http.user_agent contains "Crawl" and not cf.client.bot) or (http.user_agent contains "bot" and not http.user_agent contains "bingbot" and not http.user_agent contains "Google" and not http.user_agent contains "Twitter" and not cf.client.bot) or (http.user_agent contains "Bot" and not http.user_agent contains "Google" and not cf.client.bot) or (http.user_agent contains "Spider" and not cf.client.bot) or (http.user_agent contains "spider" and not cf.client.bot)
```

## Activam la protecció DDoS.

Per defecte, ja ve activada la protecció en front atacs DDoS de Cloudflare, però podem anar a revisar les accions i regles que desencadenaràn diferents atacs. Anirem a Security > DDoS.

<img src="..\assets\images\cloudflare\ddos.png" alt="Verificam protecció DDOS." width="700"/>

Podem veure el Ruleset que s'aplica a totes les comptes gratuïtes i modificar quina acció es farà en trobar-ne un cas que apliqui.

<img src="..\assets\images\cloudflare\ddosruleset.png" alt="Regles DDOS L7." width="700"/>

Per defecte, quasi tot el que està aquí es bloqueja. Si necessitàssim aplicar un canvi per una situació especial, podríem buscar la regla i deshabilitar-la.

## Mode Bot Fight

Aquest mode, molt similar a la regla de Firewall anterior, té un llistat de patrons de comportament dels bots més coneguts, i quan detecta peticions similars a ells els aplica un JavaScript Challenge.

<img src="..\assets\images\cloudflare\botfight.png" alt="Activam el mode Botfight." width="700"/>

> ### Apunt: JavaScript Challenge i Captchas
> Un Captcha és una operació que l'usuari ha de resoldre per a verificar que no es un bot. Per exemple, escollir entre fotos per a trobar el que ens demanen, operacions matemàtiques, llegir imatges, etc. A aquestes altures de la película, pràcticament tots hem resolt algún captcha en la nostra vida. Són bastant efectius en destriar el tràfic legítim dels Bots, però són molt irritants.
> En canvi, els JS Challenges són uns processos que el JavaScript que s'executi al teu navegador ha de resoldre per a verificar que és un User Agent legítim. Son menys fiables, però així i tot són molt útils, i només ralentitzen una mica la càrrega de la pàgina web.