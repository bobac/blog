---
layout: post
author: bobac
title: Navidrome - self hosted music streaming
tags: [linux,docker,mp3,audioknihy]
---
V minulém [blogu](/2021/11/17/Tagy-na-mp3-z-cli.html) jsem psal o **mid3v2** k editování MP3 Tagů z CLI. Potřeboval jsem si rozběhnout vlastní streamování MP3, které bude mimo iTunes. Jde o to, že mám pár audioknih, které jsem koupil u [Mariana Kechlibara](https://kechlibar.net/obchod/), a které jsem potřeboval oddělit od iTunes. Důvod je poměrně jednoduchý - chci, abych měl rozposlouchanou knihu někde jinde než muziku, z toho prostého důvodu, že iTunes nemá žádné záložky, takže jakmile pustím nějakou hudbu, prd vím, kde jsem přestal poslouchat knížku :-)

## Požadavky
Celkem jednoduché:
1. Musí to běžet na některém z mých serverů, takže *Linux*.
2. Potřebuji k tomu nějakého klienta pro iOS, kde to budu reálně poslouchat (i když je samozřejmě hezké, že si můžu pustit cokoliv odkudkoliv jen z browseru), ale není to můj hlavní use-case.
3. Bylo by hezký, kdyby to mělo hotový `docker-compose.yml`, aby zprovoznění nesežralo příliš lidského sádla.

Moje řešení splňuje vše.

## Instalace
Do nějakého adresáře uložte následující `docker-compose.yml`:
```
version: "3.3"
services:
  navidrome:
    image: deluan/navidrome:latest
    restart: unless-stopped
    ports:
      - "4533:4533"
    environment:
      ND_SCANINTERVAL: 30m
      ND_LOGLEVEL: info
      ND_BASEURL: ""
    volumes:
      - "./data:/data"
      - "/path/to/your/music/here:/music:ro"
```
`/path/to/your/music/here` je v mém případě na zfs svazku, který se jmenuje `tank/audiobooks`, což s sebou přineslo nepříjemný efekt, že kontejner nešlo spustit s obskurní chybovou hláškou, že nemůže vytvořit volume, které je read-only, či něco podobného. Strejda Gůgl mi řekl, že je to proto, že mám docker nainstalovaný (Ubuntu 20.04) pomocí **snap** balíčků, až že se to pak u zfs stává. Řešením bylo `sudo snap remove docker` a instalace pomocí `apt`, návodů je plný internet. Nepříjemností je, že budete muset re-creatnout (buildnout/pullnout) všechny existující kontejnery, což znamenalo puoze čtvrthodinové zdržení. **Miluju Docker!!!** Instalovat všechno postaru, to by mě jeblo.

## Spuštění
Easy: `docker-compose up -d`. Pak se připojit na `http://ip_serveru:4533`, vytvořit si uživatele a je to. Já jsem si ještě vyrobil site na reverzní proxy s HTTPS pomocí [proxy-manageru](/2021/11/05/reverzni-proxy-s-ntlm.html), abych mohl na svůj **Navidrome** zvenku.

Vypadá to takhle:
![Screenshot z Navidrome](/assets/images/navidrome.png)

## Přístup z iOS
Zatím jsem nainstaloval [substreamer](https://apps.apple.com/us/app/substreamer/id1012991665) a zdá se, že dělá přesně to, co chci. Stačí po instalci jen zadat url svého serveru, username a password a je to. Umí to i offline obsah a dokonce i bookmarky, takže je možné si explicitně uložit, kde jsem přestal. Super!

## Credits
Asi by nebylo fér nezmínit, kde jsem na **Navidrome** přišel. Na skvělém Youtube kanálu [Awsome Open Source](https://www.youtube.com/channel/UCwFpzG5MK5Shg_ncAhrgr9g), který provozuje **Brian McGonagill**, `docker-compose.yml` je i s chlupama sebraný z jeho [shownotes](https://shownotes.opensourceisawesome.com/navidrome-music-streaming/), já jsem přidal jen, aby se po rebootu serveru zase pustil. Díky!
