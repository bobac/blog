---
layout: post
author: bobac
title: Jak nastavit Windows, aby braly hodiny v počítači dle UTC
---
Občas je třeba, aby RTC hodiny v počítači byly nastaveny na *UTC*, ale aby Windows ukazovaly čas dle naší časové zóny, tedy *local time*. Hodí se to třeba v případě, že provozujete dual boot Windows a Linux. Obecně je totiž zvykem, že Windows očekávají, že jsou hodiny v počítači nastaveny na místní čas, kdežto Linux očekává, že hodiny v počítači jsou nastaveny na UTC.

To způsobuje následující problémy:

1. liší se časy souborů vytvořených jednotlivými systémy o rozdíl mezi UTC a místním časem (u nás o 1 až 2 hodiny, podle toho, jestli máme zrovna letní nebo zimní čas)
2. systémy se neustále přetahují o to, na kolik budou hodiny nastaveny, protože synchronizace času z internetu nastavuje taky RTC hodiny v počítači
3. následkem čeho není čas nikdy dobře :-)

## Co s tím
Stačí Windows říct, že mají taky počíat s tím, že čas v RTC hodinách v počítači (ten, co si nastavujete v setupu) je UTC. Stačí na to jednoduchá úprava v registrech:

V klíči `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\TimeZoneInformation` vytvořte `QWORD` se jménam `RealTimeIsUniversal` a nastavte jeho hodnotu na `1` a rebootněte systém. To je vše.

Pokud náhodou používáte 32bitová Windows (existuje ještě někdo takový?), místo `QWORD` použijte `DWORD`.
