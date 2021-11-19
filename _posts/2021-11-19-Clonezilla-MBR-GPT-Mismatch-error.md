---
layout: post
author: bobac
title: Clonezilla - divná chyba MBR a GPT mismatch
---
Experimentoval jsem s [Clonezillou](https://clonezilla.org/). Nainstaloval jsem čisté **Windows 10 Pro** na disk, kde už předtím nějaká Win10 byla. Nicméně jsem v setupu na začátku vykilloval všechny partition a do prázdného místa nainstaloval Windows znovu. Po tradičním zapatchování jsem chtěl Clonezillou udělat image celého disku. Po odbouchání next - next - next víceméně s defaultním nastavením na mě vypadla divná chyba, něco jako *"this disk contains mismatched gpt and mbr"* a imagování crashlo ještě dřív, než začalo.

No, a protože jsem začal gůglit a na první dobrou jsem si pěkně MBR partition zlikvidoval tak, že wokna ani nezačala bootovat, podruhé jsem gůglil pozorněji a tady je postup, co v takovém případě udělat - tohle fungovalo. Pro mě dobrá zpráva je, že jsem na disku neměl žádná data, takže kromě ztráty času se nekonal žádný stress :-)

## Postup

* přepnout se do command line (já jsem použil chvat Alt - šipka vlevo, ale Clonezilla na to má i meníčko)
* `sudo gdisk /dev/sda`
* Dostanete něco jako:

```
Partition table scan:
MBR: MBR only
BSD: not present
APM: not present
GPT: present

Found valid MBR and GPT. Which do you want to use?
1 – MBR
2 – GPT
3 – Create blank GPT

Your answer:
```

* Zvolíte 1 - MBR
* následuje:

```
Command (? for help): x

Expert command (? for help): z
About to wipe out GPT on /dev/sda. Proceed? (Y/N): y
GPT data structures destroyed! You may now partition the disk using fdisk or
other utilities.
Blank out MBR? (Y/N): n
MBR is unchanged. You may need to delete an EFI GPT (0xEE) partition
with fdisk or another tool.
```

* done

## Credits
Našel jsem to původně [tady](https://casits.artsandsciences.fsu.edu/tips/cloning-systems/mismatched-GPT-MBR-error).