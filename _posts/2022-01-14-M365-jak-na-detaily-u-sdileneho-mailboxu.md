---
layout: post
author: bobac
title: M365 - Jak na detaily u sdíleného resource mailboxu
---
Pokud na M365 vyrobíte sdílený resource mailbox (třeba na rezervace zasedačky, nebo auta), je nepříjemné, že ve výchozím stavu není vidět, kdo má daný prostředek rezervovaný, pokud se do kalendáře podíváte z Outlooku - je tam jen, že je Busy.

Zajistit, aby **Exchange Online** do vytvořené události laskavě připsala, kdo je za ní zodpovědný, je možné pomocí následujících příkazů v *PowerShellu*:

```
Import-Module ExchangeOnlineManagement
Connect-ExchangeOnline -UserPrincipalName admin@firma.onmicrosoft.com
Set-MailboxFolderPermission -Identity sdilenymailbox:\Calendar -User default -AccessRights LimitedDetail
Set-CalendarProcessing -Identity sdilenymailbox -AddOrganizerToSubject $true -DeleteComments $false -DeleteSubject $false
```

### Poznámka
Je nepříjemné, že jsem nepřišel na to, jak udělat to samé, ale být přihlášen jako já, a vyrobit to pro jiného tenanta. Jinými slovy, musím být přihlášen jako admin organizace, pro kterou to nastavuju, ne jako admin organizace, která tuhle organizaci spravuje. Grrrr.!!!

## Credits
Jako tradičně jsem to vykradl někde jinde - jmenovitě [tady](https://lazyadmin.nl/office-365/show-details-room-mailbox-meetings-in-its-calendar-in-office-365/). 