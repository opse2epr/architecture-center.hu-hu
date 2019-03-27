---
title: 'CAF: A Cost Management metrikák, a mutatók és a szervezet kockázattűrési határát'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Felhőalapú cégirányítási viszonyítva Cost Management ismertetése
author: BrianBlanchard
ms.openlocfilehash: 76e6b1b32dd862322f6cafd9aa63c6c4f79f4f5d
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298466"
---
# <a name="cost-management-metrics-indicators-and-risk-tolerance"></a>A Cost Management metrikák, a mutatók és a szervezet kockázattűrési határát

Ebből a cikkből segítséget nyújt az üzleti szervezet kockázattűrési határát számszerűsítik a Cost Management vonatkozik. Metrikák és a mutatók segítségével hozhat létre egy üzleti esetet, befektetés végzett az adatbázisplatform működését is a Cost Management fegyelem.

## <a name="metrics"></a>Mérőszámok

A Cost Management általában a költségekkel kapcsolatos metrikák összpontosít. A kockázati elemzés részeként szeretné a jelenlegi és tervezett kiadások felhőalapú számítási feladatok között mekkora kockázatot, és mennyire fontos beruházási költség cégirányítási, hogy a bevezetési stratégia meghatározása a kapcsolódó adatok összegyűjtése.

A következő példák hasznos metrikák mérik a kockázattűrése az alapvető biztonsági területen belül összegyűjtendő:

- Éves költségeit: A teljes éves költségét a felhőszolgáltató által nyújtott szolgáltatások
- Havi kiadások: A teljes havi költségét a felhőszolgáltató által nyújtott szolgáltatások
- Előrejelzett és a tényleges arány: Az arány összehasonlítása előrejelzett és a tényleges kiadásokat (havi vagy éves)
- Ütemben elfogadása (MOM) arány: Százalékos aránya a különbözeti a felhőbeli költségek hónapra
- Összesített költségeit: Teljes elhatárolt naponta kiadások, a hónap kezdetétől fogva
- Költési trendek: A költségvetés költési trend

## <a name="risk-tolerance-indicators"></a>Kockázati tolerancia mutatók

Korai központi telepítések, például fejlesztési, tesztelési vagy kísérleti első számítási feladatok során Cost Management várhatóan viszonylag alacsony kockázat. További eszközök vannak telepítve, növekszik a kockázatot és kockázati szintű az üzleti szempontból valószínű, hogy elutasítja. Ezenkívül, több felhő bevezetési csapat adta konfigurálása vagy az eszközök üzembe helyezése a felhőben nő a kockázata, és csökkenti a hibatűrést. Ezzel szemben a Cost Management fegyelem növekvő lépnek további innovatív új technológiákat üzembe helyezéséhez a cloud Bevezetési fázis a személyek.

A felhő bevezetésének korai szakaszában az üzleti kockázat tolerancia alapterv meghatározásához fog működni. Miután alapterv, szüksége lesz a feltételt, amely akkor lép működésbe, befektetés a Cost Management tantárgy meghatározásához. Ezek a feltételek valószínűleg különböző minden szervezet számára.

Ha azonosította [üzleti kockázatai](./business-risks.md), az üzleti eseményindítókat, amelyek vélhetően megnövelheti ezek a kockázatok azonosítása alkalmazó azonosítására fog dolgozni. Az alábbiakban néhány példa a hogyan mérőszámokat, például a fent említett lehet összehasonlítani a referenciakonfiguráció kockázattűrése az üzleti igények tovább a Cost Management beruházni jelzi.

- Kötelezettségvállalás adatvezérelt (a legtöbb gyakori): A vállalat számára fontos, hogy a költségkeret-beállítási $X, 000 000 felhőalapú szállító ebben az évben. Győződjön meg arról, hogy az üzleti nem haladja meg a költségkeret célok több mint 20 %-kal, és hogy 90 %-ában ezen kötelezettségvállalás jegyében fogják használni, a Cost Management fegyelem van szükségük.
- Eseményindító százalékos aránya: A felhő költségeit, amely egy vállalati stabil azok éles rendszerek esetén. Ha, amely módosítja a nagyobb, hogy X a(z) %, a Cost Management fegyelem célszerű előre megtervezni befektetési lesz.
- Szükségesnél több erőforrással ellátott eseményindító: Egy olyan cég, úgy véli, hogy üzembe helyezett megoldások vannak szükségesnél több erőforrással ellátott. A Cost Management egy prioritású befektetési, amíg a bizonyítani megfelelő igazítás kiépítés és az eszközintelligencia-felhasználás mintavételezését.
- Havi költségkeret eseményindító: Egy cég, amely több mint $x várólistában, 000 / hó eloszlatása költség számít. Ha a költségkeret-beállítási meghaladja, hogy az adott hónapban, azok kell beruházni a Cost Management.
- Éves költségkeret eseményindító: Az informatikai R & D költségvetés, amely lehetővé teszi, hogy a költségkeret-beállítási $X, a felhő Kísérletezési évenként 000 vállalattal. Előfordulhat, hogy éles számítási feladatok futtatása a felhőben, de továbbra is minősülnek kísérleti megoldások, ha a költségvetés nem haladja meg az adott mennyiség. Miután emelkedik, azok kell a költségvetést, például egy éles befektetési kezelik, és szorosan összegű költségkeretet kezeléséhez.
- Káros OpEx (ritka): Vállalati ezeket nagyon OpEx káros és fejlesztési és tesztelési számítási feladatok üzembe helyezése előtt kell a Cost Management vezérlők beállítása.

## <a name="next-steps"></a>További lépések

Használatával a [Cloud Management-sablon](./template.md), dokumentum-mérőszámok és hibatűrő mutatók, amelyek a jelenlegi cloud bevezetési tervet.

Cost Management szabályzat betartása vonatkozó és egy folyamat létrehozása a kockázatokat és a hibatűrést, hoz létre.

> [!div class="nextstepaction"]
> [Szabályzat megfelelőségi folyamat létrehozása](compliance-processes.md)
