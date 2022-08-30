---
layout: default
title: Cloudflare
parent: Configuració de la xarxa
nav_order: 2
---

# Dominis, registres DNS, i com podem rebre requests del món

En aquesta secció veurem com ens poden arribar peticions del món cap al nostre servidor web, sigui el que sigui el servei que estem servint. Realment, seria molt senzill, ja que per a estar disponibles com a recurs a Internet, només bastarà que configurem un Port Forwarding al nostre firewall, i mitjançant la combinació IP pública + Port, el nostre recurs pot ésser accedit des de qualsevol part del món.

Però, no ens convé anar per lo fàcil. Un port públic al nostre firewall és sinònim d'una porta d'entrada a la nostra xarxa per a tota casta de peticions, siguin vàlides o malicioses. Per això ens convé configurar unes certes regles d'entrada de tràfic per mirar de posar-ho complicat a qui vulgui entrar amb males intencions.

Començarem configurant un nom de domini. Així, podrem filtrar totes les peticions que busquin una IP pública, en comptes d'un domini del tipus ```*.domini.tld.```

En el nostre cas, vam comprar el domini al registrar Namecheap amb bonos de Github Edu.

<img src="..\assets\images\cloudflare\namecheap1.png" alt="Entrada espardenya.me al registrar del domini." width="700"/>

Ara, éssent propietaris del domini ```.espardenya.me.```, podríem redirigir el tràfic desde el domini fins a la nostra IP pública, i amb un reverse proxy filtrar totes les peticions que no vinguin al domini.

Però, nosaltres emprarem els serveis de Cloudflare.

# Cloudflare: més que un registrar

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

> ## Apunt: DDNS
> Normalment, els ISP assignen direccions de manera dinàmica emprant el protocol DHCP als seus clients. Si la nostra IP canvia, haurem d'estar atents a aquest canvi que és pràcticament imprevisible per a canviar el registre al nostre panell de control del domini.
>
> O bé, podem emprar un client DDNS que el que farà és monitoritzar la direcció IP pública de la nostra xarxa per a, en el moment que canvii, assignar-la al nostre gestor de noms de domini, i així automatitzar aquesta tasca i assegurar-nos una major disponbilitat.

Primer de tot, crearem un registre DNS a l'apartat DNS de la [plataforma](dash.cloudflare.com)

Aquest registre ha de tenir les seguents característiques:
* Registre de tipus A
* Domini o subdomini
* Una IP falsa.
* Cloudflare Proxy DESACTIVAT

Per exemple, crearem un registre A per al subdomini test, que apunti a la direcció 1.1.1.1

<img src="..\assets\images\cloudflare\cloudflare1.png" alt="Alta subdomini" width="700"/>

Ara, necessitarem treure un Token per donar accés al nostre PFSense per a canviar el registre que acabam de fer amb les dades vàlides

## **FALTA DOCUMENTAR**

## Anem a crear algunes regles de Firewall.

Podem veure a les analítiques que hem començat a rebre peticions de tot el món al nostre domini. Aquí és on hem de pensar si realment el nostre servei web necessita estar disponible a tot el món, o si basta algun lloc en concret. En el meu cas, dubt que sigui necessari fer la pàgina web accessible des de fora d'Espanya, així que aquesta serà la primera regla del nostre primer Firewall.