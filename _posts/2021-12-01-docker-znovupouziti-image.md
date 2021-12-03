---
layout: post
author: bobac
title: Docker Compose - znovupoužití image, kterou buildovala jiná service
---
Potřeboval jsem `docker-compose.yml`, který buildil image z `Dockerfile` místo toho, aby ji hotovou odněkud stahoval, nebo abych ji nemusel buildovat předtím já ručně pomocí `docker build...`. To je docela šikovná věc, zejména, pokud ta image je to, co zrovna vyvýjíte. **Docker Compose** na to má hezkou direktivu `build` místo tradiční `image`.

Problém je v tom, že jsem potřeboval 2 různé service, které ale budou vždy zbuilděné (krásný český slovo) ze stejné image. Nejdřív jsem to nahulváta dělal tak, že jsem prostě v každé service znova buildil, ale pak mě okolnosti donutily chvíli buildit s direktivou `--no-cache` a celé se to začelo nějak vléct. Nezbývalo než začít gůglit, jak to udělat, aby build proběhl jen jednou a druhá service jej jen použila.

## Jak na to
Je to samozřejmě easy. Stačí, aby první service udělala build a zároveň hotovou image nějak pojmenovala, a pak už jen stačí, aby ta druhá použila tuhle image a měla nastavenou dependecy na té první. Asi takhle `docker-compose.yml`:

```
version: '3'

services:

  master:
      build: .
      image: mojeimage

  slave:
      image: mojeimage
      depends_on: master
```

## Použití
Pak už stačí jen image zbuildit pomocí `docker-compose build`. Docker Compose buildne `master` image a pojmenuje ji `mojeimage`.Následný `docker-compose up` už pustí obě services z jediné image.

### Poznámka
Preferuji vytvoření `docker-compose.yml` a následné buildění (pokud to jde) pomocí **Docker Compose** zejména proto, že v podstatě předpis (který tak jako tak potřebuju) mám v `docker-compose.yml` a nemusím přemýšlet (to bolí vždy) nad tím, jak pojmenovat image apod. tak, aby to fungovalo. Samozřejmě, že by fungovalo ručně zbuildit pomocí `docker build -t mojeimage .`, ale tam bych musel jméno image nejdřív stejně vyčíst ze svého `docker-compose.yml`. Takhle je to jednodušší :-)

## Credits
Celé jsem to ukradl na [Stack Overflow](https://stackoverflow.com/questions/50019948/reuse-image-built-by-one-service-in-another-service/50025029#:~:text=28-,Docker%20Compose,-builds%20your%20image).
