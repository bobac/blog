---
layout: post
author: bobac
title: Reverzní proxy s NTLM
---
Už před časem jsem narazil na to, že pokud potřebuju někde udělat reverzní proxy, která potřebuje podporu NTLM authentication, tak mám se svým NGINXem smůlu. NGINX totiž podporuje NTLM pouze v placené verzi. Nějakou dobu registruji existenci web serveru **[Caddy](https://caddyserver.com/)**, ale zatím jsem si vždy vystačil s kombem [NGINX](https://www.nginx.com/) a geniálním, krásným, a vůbec po všech stránkách velmi povedeným [Nginx Proxy Managerem](https://nginxproxymanager.com/). Důvod, proč jsem vždycky používal **NGINX** (třeba namísto Apache) byl v tom, že mi styl konfigurace tak nějak šel lépe do ruky. Nějakou dobu jsem řešil **Let's Encrypt** certifikáty pomocí jejich skriptů a fungovalo to docela hezky, ale **Nginx Proxy Manager** to ještě dost zjednodušil - prostě si zaškrtnete, že k danému webu chcete https, necháte jej *HTTP* traffic automaticky forwardovat na *HTTPS* a o víc se nemusíte starat. Běží to ve dvou kontejnerech, má to hezké klikací webové rozhraní a generujete konfiguráky pro **NGINX** samo. A taky to samo obnovuje certifikáty. Nádhera! Vypadá to asi takhle:

![Screenshot z Nginx Proxy Manageru](/assets/images/proxy-manager.png)

Rozběhnout je to hračka - potřebujete soubor `docker-compose.yml`, který může vypadat třeba takhle:
```
version: '3'
services:
  app:
    image: 'jc21/nginx-proxy-manager:latest'
    restart: unless-stopped
    ports:
      - '80:80'
      - '81:81'
      - '443:443'
    environment:
      DB_MYSQL_HOST: "db"
      DB_MYSQL_PORT: 3306
      DB_MYSQL_USER: "npm"
      DB_MYSQL_PASSWORD: "npm"
      DB_MYSQL_NAME: "npm"
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
  db:
    image: 'jc21/mariadb-aria:latest'
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: 'npm'
      MYSQL_DATABASE: 'npm'
      MYSQL_USER: 'npm'
      MYSQL_PASSWORD: 'npm'
    volumes:
      - ./data/mysql:/var/lib/mysql
```
(Dá se to bez přemýšlení obšlehnout z webu projektu).
Kontejnery pustíte pomocí `docker-compose up -d` a pak stačí jen klikat na `http://lokalni_ip:81`. Prvotní uživatelské jméno je `admin@example.com` a heslo je `changeme`. Myslej sem si, že už to jednodušší být nemůže...

## Seznamte se, tohle je Caddy
Idea webového serveru **Caddy** je následující: Všechno musí fungovat v defaultním nastavení tak nějak samo. **Caddy** je *HTTPS* first, tedy pokud *HTTPS* explicitně nezakážete, pojede pod *HTTPS* a sám si bude točit certifikáty. Konfigurák, který se jmenuje `Caddyfile`, pro reverzní proxy je až směšně jednoduchej:
```
venkovni.server.cz {
  reverse_proxy https://192.168.1.1:443 {
    transport http_ntlm {
      tls_insecure_skip_verify
    }
  }
}
```
Tohle samo vyrobí https://venkovni.server.cz, zařídí si to certifikáty a bude servírovat server na vnitřní https://192.168.1.1:443, který vyžaduje NTLM autentifikaci. V mém případě je vnitřní traffic podepsaný self-signed certifikátem, takže je třeba odstavec `{ tls_insecure_skip_verify }`, kdybych měl vevnitř jen *http*, není třeba ani to.

### Instalace
Jediná drobná zrada je v tom, že si nemůžu jen tak vzít hotový `Dockerfile` nebo `docker-compose.yml`, protože podpora pro NTLM transport není ve standardní binárce zakompilovaná. Ale i tak je to easy - dalo mi to asi hodinku než jsem na to přišel, a to jenom proto, že před tou hodinkou byly moje znalosti o **Caddym** následující:

1. Je to web server/reverzní proxy
2. Je to napsaný v Go

To druhý není pro náš případ vůbec podstatný, ale protože se v *Go* komunitě už nějakou dobu pohybuju, díky tomu jsem se o **Caddy** vůbec dozvěděl :-)

Takže potřebujema dva soubory. `Dockerfile`:
```
FROM caddy:2.4.5-builder AS builder

RUN xcaddy build \
    --with github.com/caddyserver/ntlm-transport

FROM caddy:2.4.5

COPY --from=builder /usr/bin/caddy /usr/bin/caddy
```

A `docker-compose.yml`:
```
version: "3.7"

services:
  caddy:
    build: .
    restart: unless-stopped
    ports:
      - "8080:80"
      - "8443:443"
    volumes:
      - $PWD/Caddyfile:/etc/caddy/Caddyfile
      - $PWD/site:/srv
      - $PWD/data:/data
      - $PWD/config:/config
```

Takže teď už stačí **Caddy** zbuildit pomocí `docker-compose build`. Než ho pustíte, dejte si k těm dvěma souborům do adresáře ještě připravený `Caddyfile` (viz výše).

Spustíte pomocí `docker-compose up -d`.
A je to.

## Závěr
Nevím, jestli se zbavím svého **NGINX**. Ale jenom kvůli tomu, že na něj existuje to krásný klikátko. **Caddy** je totiž tak easy, že snad klikátko ani nepotřebuje. Ale pokud budu zase někdy potřebovat reverzní proxy s NTLM (v mém případě to bylo kvůli Windows Admin Center), určitě po Caddym sáhnu. A dost možná, že na něj přejdu úplně a klikátko oželím. Je to zatím nejjednodušší web server, jaký jsem kdy viděl.
