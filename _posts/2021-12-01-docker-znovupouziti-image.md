---
layout: post
author: bobac
title: Docker Compose - znovupoužití image, kterou buildovala jiná service
---
Potřeboval jsem `docker-compose.yml`, který buildil image z `Dockerfile` místo toho, aby ji hotovou odněkud stahoval, nebo abych ji nemusel buildovat předtím já ručně pomocí `docker build...`. To je docela šikovná věc, zejména, pokud ta image je to, co zrovna vyvýjíte. **Docker Compose** na to má hezkou direktivu `build` místo tradiční `image`.

Problém je v tom, že jsem potřebova 2 různé service, které ale budou vždy zbuilděné (krásný český slovo) ze stejné image. Nejdřív jsem to nahulváta dělal tak, že jsem prostě v každé service znova buildil, ale pak mě okolnosti donutily chvíli buildit s direktivou `--no-cache` a celé se to začelo nějak vléct. Nezbývalo než začít gůglit, jak to udělat, aby build proběhl jen jednou a druhá service jej jen použila.

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

## Credits
Celé jsem to ukradl na [Stack Overflow](https://stackoverflow.com/questions/50019948/reuse-image-built-by-one-service-in-another-service/50025029#:~:text=28-,Docker%20Compose,-builds%20your%20image).
