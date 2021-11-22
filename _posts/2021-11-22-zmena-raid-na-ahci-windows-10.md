---
layout: post
author: bobac
title: Změna SATA RAID mode na AHCI a nebootující Windows 10
---
Potřeboval jsem u svého stárnoucího **Dell OptiPLex 3050** změnit v BIOSu režim SATA disku z RAID na AHCI. Potřeboval jsem to proto, že primární disk je NVME, na něm jsou **Windows 10**, a na druhém (SATA) mám **Ubuntu 20.04**. Potřebuji z Linuxu vidět na Windowsí disk a gůglením jsem zjistil, že důvod, proč žádné `/dev/nvme*` nevidím není v tom, že by můj kernel 5.4.xx NVME nepodporoval, NVME je podporováno už od kernelu 3.3, ale v tom, že kernel nevidí NVME disk, když je v BIOSu nastavený SATA RAID mode.

Bohužel po přepnutí do AHCI režimu Windows odmítaly botovat, vždy skončily v režimu opravy. Málem jsem začal Windows opravovat z instalačního média, ale představa ukládání 5GB image na pomalou flešku mě donutila ještě chvíli gůglit a zde je, co jsem vygůglil.

Problém je v podstatě jen v tom, že Windows nemají zavedený AHCI ovladač disku, stačí je nabootovat do Safe mode, ovladač se sám načte a po dalším rebootu do normálního režimu už je vše OK.

## Postup

1. V běžících Windows se SATA mode RAID pusťit `cmd` jako Administrator.
2. `bcdedit /set {current} safeboot minimal`
3. Reboot do BIOSu a změna na SATA mode na AHCI
4. Nabootovat do Windows (díky bodu 2 nastartují vždy v safe mode).
5. `bcdedit /deletevalue {current} safeboot`
6. Reboot - už to najede normálně v AHCI modu a Windows se spustí

### Poznámky

1. V níže uvedeném postu se píše, že je třeba znát heslo uživatele (ne PIN, pokud používáte AzureAD). Uvedené používám, přesto mým Windows 10 21H2 pin stačil, žádné heslo jsem nepořeboval. Ale pozor na to, je dobré jej znát, co kdyby.
2. Stejně tak se tam píše, že budete potřebovat *Recovery key*, pokud používáte **Bitlocker**. Používám, měl jsem jej po ruce, nebyl třeba.

## Credits

Čerpal jsem ve fóru na [Superuser](https://superuser.com/questions/1280141/switch-raid-to-ahci-without-reinstalling-windows-10#:~:text=56-,There,-is%20actually%20another). Nepoužil jsem preferred řešení, ale to níže uvedené (link ukazuje přímo do textu, pokud používáte poslední Chrome, já mám *96.0.4664.55*).