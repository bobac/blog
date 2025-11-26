---
layout: post
author: bobac
title: Jak Azure AD Connect pracuje s účty ADSyncMSA a MSOL – stručné a praktické vysvětlení
tags: [azure,ad,chatgpt]
---
V praxi není nic neobvyklého, když po instalaci Azure AD Connect najdete v AD pár podivných účtů, které na první pohled nedávají smysl. Tato konverzace objasnila, co je co, a proč někdy může nastat problém při přidávání těchto účtů do skupin podle interních politik.

## Co vlastně Azure AD Connect vytváří
### 1. **ADSyncMSAxxxx** – definice gMSA
Tohle je objekt typu *Group Managed Service Account*. Není to běžný uživatelský účet a nemůže se přihlásit. Je to jen „hlavička“ účtu, kterou Active Directory používá k řízení hesel a atributů.

### 2. **ADSyncMSAxxxx$** – skutečný bezpečnostní principal
Služba „Microsoft Azure AD Sync“ běží právě pod tímto účtem.  
Je to reálný logon účet a má `$` stejně jako počítačové účty.  
V konzoli MMC ho ale často vůbec neuvidíte, což mate správce.

### 3. **MSOL_xxxxxxx** – starý service account
Tohle je účet ze starší verze AAD Connect. Moderní instalace už tento účet nepoužívají, ale účet v AD zůstává jako pozůstatek. Nemá vztah k současnému gMSA účtu.

## Proč nejde účet najít v AD
MMC (dsa.msc) neumí zobrazit gMSA tak, jak by člověk čekal.  
Vidíte prázdný kontejner, nevidíte `$` účet, ale Azure AD Sync služba pod ním běží.

Řešením je:
- použít PowerShell,
- nebo použít ADAC (který gMSA zobrazuje správně).

## Jak přidat ADSync gMSA do skupiny
Interní politika vyžaduje, aby všechny účty služeb byly členem určité skupiny — například *grpservice*.  
Proto je potřeba přidat **ADSyncMSAxxxx$**.

PowerShell řeší situaci okamžitě:

```powershell
Add-ADGroupMember -Identity "grpservice" -Members "ADSyncMSAxxxx$"
```

Pokud by vyhledání selhalo, použije se DN definice gMSA, AD si to sám namapuje.

## Závěr
- Azure AD Connect používá gMSA, ne starý MSOL účet.  
- `$` varianta je ta, kterou používá služba.  
- Pro přidání do skupiny je nutné pracovat právě s ní.  
- Starý účet MSOL můžeš s klidem smazat, pokud už není v provozu.

Celá situace vzniká jen díky tomu, že MMC nepodává úplný obraz.  
Jakmile člověk ví, co který objekt je, dává to celé perfektní smysl.
