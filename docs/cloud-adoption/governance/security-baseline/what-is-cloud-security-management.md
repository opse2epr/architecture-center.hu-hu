---
title: 'CAF: Mit jelent a felhő biztonsági alapterv'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 10/10/2018
description: Mi a Felhőbeli biztonsági alapterv?
author: BrianBlanchard
ms.openlocfilehash: cb6e3372fd76486e5c4467ef822163eac0499573
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899230"
---
<!-- markdownlint-disable MD026 -->

# <a name="what-is-the-cloud-security-baseline"></a>Mi a Felhőbeli biztonsági alapterv?

Ez a felhő biztonsági alapterv, amely épül, amely az általános témakör bevezető cikk a [öt oktatnak, felhőalapú Cégirányítási](../governance-disciplines.md) cégirányítási keretet. További információk a Cloud Security érhető el [az Azure megbízható felhőalapú](https://azure.microsoft.com/overview/trusted-cloud/). Megközelítések szervezetek biztonsági helyzetét javításához megtalálható a [Felhőbeli biztonsági Szolgáltatáskatalógusának](https://www.microsoft.com/security/information-protection)

> [!NOTE]
> Ez a cikk várhatóan nem elég kontextusba, hogy az olvasó biztonsági stratégia megvalósításához. Csak az általános tájékoztatási feladata.

## <a name="cloud-security"></a>Biztonság a felhőben

A cloud security az kiterjesztése, a hagyományos információk biztonsági eljárásokat. A hagyományos informatikai biztonsági szabályzatok és a számítógép biztonsági, hálózati biztonság, adatvédelem, információkat használati és így tovább szabályozó vezérlők tartalmazhat. Ugyanazokat a szabályzatokat és vezérlők ezeket a felhőben van szükség. Bármely Felhőbeli átalakítás során feltétlenül szükséges, hogy aktívan kell-e vonni az adatvédelmi és megérteni a felhőbeli kérdését, és annak biztosítása érdekében a régi informatikai házirendek térkép a megfelelő szintű vezérlését a felhőben.

Bármely felhőbeli biztonsági stratégia minimális, érdemes megfontolni a következő témaköröket:

* Adatok besorolása: Megfelelő adatok besorolása minden olyan adatforrások, amelyek a saját, a védett vagy a szigorúan bizalmas megértéséhez. Ez segít a tervezés során kezelni a kockázatokat.
* Tervezze meg a hibrid felhő használata: Ismertetése hogyan örökölt, a helyszíni hálózatok fog csatlakozni a felhő segítségével a kockázatok feltárása és elhárítása CISO.
* Tervezze meg a támadások és a hibák: Az első néhány hónap során az átalakítás hibákkal történik, a csapat tanul. Alacsony kockázatot jelentő és rendkívül korlátozott központi telepítések további biztonságosan kezdődik.
* Adatvédelmi rangsorolásában: Bármely átalakítás során a teljes csoport ügyfelek és alkalmazottak bizalmas adatvédelmi felső részén szem előtt kell tartania. Az adatai mindig biztonságban vannak a felhőben, de a csapat célszerű tisztában és extra nagy körültekintéssel végezze el a bizalmas adatok esetén.

## <a name="protecting-data-and-privacy"></a>Adatok és a magánszféra védelme

A szervezetek a világ számos részén &mdash; e kormányok, nonprofit vagy vállalkozások &mdash; a felhő-számítástechnika vált a folyamatban lévő IT-stratégia kulcsfontosságú része. A cloud services biztosít minden méretek hozzáférési szervezetek gyakorlatilag korlátlan tárolási szabadítson fel őket a kell vásárolni, karbantartása és a saját hálózatokon és számítógépes rendszerek frissítése közben. A Microsoft és egyéb felhőszolgáltatók kínálnak informatikai infrastruktúrát, platformot és szoftverszolgáltatás (SaaS), így az ügyfelek gyorsan méretezése felfelé és lefelé, szükséges, és csak akkor kell fizetnie a számítási teljesítmény és tárolás használata.

Azonban mivel kiemelési hatékonyság és informatikai költségek csökkentése a felhőszolgáltatások, például a megnövekedett, agilitást, rugalmasságának és szabadságának előnyeit kihasználásához szervezetek, akkor figyelembe kell vennie hogyan services, a felhő bevezetése hatással van a adatvédelmi, biztonsági és megfelelőségi állapotáról. A Microsoft felismerte, hogy a felhőszolgáltatási ajánlatokhoz nem csupán méretezhető, megbízható és felügyelhető, de is biztosításához ügyfeleink adatok védelme és átlátható módon használt.

Biztonsági adatok erős védelmet az összes online számítási környezetben lényegi eleme. De biztonsági önmagában nem elegendő. Fogyasztók és vállalkozások hajlandó vállalni a bukás egy adott felhő-számítástechnikai termék is függ, hogy az adatvédelmi tarthassák adataik fogja védeni, és, hogy az adatokat csak akkor fogjuk használni vásárlói elvárásokkal konzisztens módon megbízható képesek használni . További információ a Microsoft megközelítése [az adatok és a felhőben védelme](https://go.microsoft.com/fwlink/?LinkId=808242&clcid=0x409)

## <a name="risk-mitigation"></a>Kockázatcsökkentés

Minden adatközpontban két legnagyobb kockázatát két forrásból sorolhatók: Elévülési rendszerek és az emberi hibák. Legalább két kockázatok elleni védelme akkor, ha egy IT-biztonsági stratégia meghatározása. Ugyanez igaz a felhőben. Az alábbiakban néhány példa a kockázatok csökkentése és a Felhőbeli biztonsági stratégia megerősítése elhelyezett vezérlők.

* Az örökölt rendszereket: A helyszíni adatközpont megoldások számos összetevőből állnak szoftverekről, hardverekről és folyamatokat, amelyek az aktuális biztonsági kockázatokat mintáink. Ha lehetséges, javíthatja, csere vagy ezekben a rendszerekben kivonása egy felhőalapú átalakítás során. Természetesen ez nem mindig lehetséges. Bármely régebbi rendszerekkel egy hibrid megoldásban éles környezetben marad, esetén fontos, hogy ezek a rendszerek leltározott és értelmezni, Virtual Datacenter kialakítása során. Ezáltal lehetővé teszi a tervezési csapat megszüntetésére, vagy a hozzáférést ezekhez a rendszerekhez, a felhőből.
* IT-biztonsági folyamatok és vezérlőelemek: Legalább felhőalapú tervezési csapat kell frissíteni a meglévő IT-biztonsági folyamatok és szabályozza azok előre, a felhőbe. Ideális esetben az IT-biztonsági csoportja tagjának kell kiberbiztonsági szaktudást és dedikált, amelynek a tervezési és megvalósítási csapatok.
* Figyelés és naplózás: Ha a tervezési cégirányítási dolgoz fel, és eszközöket, biztosítása érdekében, hogy figyelést és felülvizsgálatot megoldások tartalmazzák biztonsági kockázatokat vagy veszélyeket.

> [!NOTE]
> Műszaki adósságot felfedése: Ez a témakör következő felkínál hiányzik. Ez a témakör emellett cikkek ki idővel. További információk a Cloud Security érhető el [az Azure megbízható felhőalapú](https://azure.microsoft.com/overview/trusted-cloud/). Megközelítések szervezetek biztonsági helyzetét javításához megtalálható a [Felhőbeli biztonsági Szolgáltatáskatalógusának](https://www.microsoft.com/security/information-protection)