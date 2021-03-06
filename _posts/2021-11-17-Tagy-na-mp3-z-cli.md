---
layout: post
author: bobac
title: Tagy na MP3 z CLI
---
Před chvílí jsem zjitil, že potřebuji z command lajny na linuxovém serveru upravovat tagy pro MP3 soubory. Po chvíli googlení jsem přistál u `mid3v2`. Hlavní důvod je, že by měl podporovat **utf-8**.

## Instalace
Potřebuje **PIP**. Takže pokud **PIP** ještě nemáte, nainstalujte jej:

```
sudo apt install pip
```
Po instalaci **PIP**u nainstalujte **MUTAGEN**:
```
sudo pip install mutagen
```

## Použití
Ověřte, že `mid3v2` funguje:
```
# mid3v2
Usage: mid3v2 [OPTION] [FILE]...

Mutagen-based replacement for id3lib's id3v2.

Options:
  --version             show program's version number and exit
  -h, --help            show this help message and exit
  -v, --verbose         be verbose
  -q, --quiet           be quiet (the default)
  -e, --escape          enable interpretation of backslash escapes
  -f, --list-frames     Display all possible frames for ID3v2.3 / ID3v2.4
  --list-frames-v2.2    Display all possible frames for ID3v2.2
  -L, --list-genres     Lists all ID3v1 genres
  -l, --list            Lists the tag(s) on the open(s)
  --list-raw            Lists the tag(s) on the open(s) in Python format
  -d, --delete-v2       Deletes ID3v2 tags
  -s, --delete-v1       Deletes ID3v1 tags
  -D, --delete-all      Deletes ID3v1 and ID3v2 tags
  --delete-frames=FID1,FID2,...
                        Delete the given frames
  -C, --convert         Convert tags to ID3v2.4 (any editing will do this)
  -a "ARTIST", --artist="ARTIST"
                        Set the artist information
  -A "ALBUM", --album="ALBUM"
                        Set the album title information
  -t "SONG", --song="SONG"
                        Set the song title information
  -c "DESCRIPTION":"COMMENT":"LANGUAGE", --comment="DESCRIPTION":"COMMENT":"LANGUAGE"
                        Set the comment information
  -p "FILENAME":"DESCRIPTION":"IMAGE-TYPE":"MIME-TYPE", --picture="FILENAME":"DESCRIPTION":"IMAGE-TYPE":"MIME-TYPE"
                        Set the picture
  -g "GENRE", --genre="GENRE"
                        Set the genre or genre number
  -y YYYY[-MM-DD], --year=YYYY[-MM-DD], --date=YYYY[-MM-DD]
                        Set the year/date
  -T "num/num", --track="num/num"
                        Set the track number/(optional) total tracks
You can set the value for any ID3v2 frame by using '--' and then a frame ID.
For example:
        mid3v2 --TIT3 "Monkey!" file.mp3
would set the "Subtitle/Description" frame to "Monkey!".

Any editing operation will cause the ID3 tag to be upgraded to ID3v2.4.
```

## Příklady
Přečtení tagů ze souboru:
```
mid3v2 -l soubor.mp3
```

Změna čísla stopy určitého souboru (v mém případě tam byl nesmysl):
```
mid3v2 --TRCK=5 05_soubor.mp3
```

Nastavení konzistentního jména alba pro všechny mp3 v adresáři (aby iTunes nebo třeba [Navidrome](https://www.navidrome.org/)) pro jedno album v adresáři nevyrobil tři samostatná alba, jenom kvůli překlepům/nekonzistencím v tagu:
```
mid3v2 --TALB="Jméno alba" *.mp3
```

Třeba to někomu pomůže, já si to sem píšu hlavně proto, že za dav roky, až to budu zase potřebovat, tak nebudu vědět, co jsem použil a kde to leží :-)
