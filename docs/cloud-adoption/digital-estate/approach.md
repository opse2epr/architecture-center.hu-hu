---
title: 'CAF: A digitális tulajdonnal kapcsolatos tervezés megközelítései'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
description: Néhány olyan módszert a digitális hagyatéki tervezési ismerteti
author: BrianBlanchard
ms.date: 12/10/2018
ms.topic: guide
ms.openlocfilehash: fc30c9034e14b451b7e9ec9dfc023d72f248b37c
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299297"
---
# <a name="approaches-to-digital-estate-planning"></a>A digitális tulajdonnal kapcsolatos tervezés megközelítései

Digitális hagyatéki tervezési alakzatokat, a kívánt eredmények és a meglévő hagyatéki méretétől függően több is igénybe vehet. Nincsenek is számos lehetőséget a megközelítést kapcsolatban. Fontos közölheti az elvárásokat kapcsolatban a megközelítés a korai tervezési ciklusokat. Nem egyértelmű elvárások gyakran az késleltetések további Hardverleltár összegyűjtéséhez gyakorlatok társított vezethet. Ez a cikk három megközelítése elemzési vázolja fel.

## <a name="workload-driven-approach"></a>Munkaterhelés-képzéstől eltérve

Felülről lefelé értékelés megközelítés kiértékeli a biztonsági szempontokra, például a kategorizálási adatok (magas, közepes vagy alacsony üzletmenetre gyakorolt hatás), megfelelőségi, adatszuverenitási és biztonsági kockázatot követelményeinek. Ez a megközelítés majd felméri kiértékelése foglalkoznak, mint a hitelesítés, adatszerkezet, késésre vonatkozó követelmény, függőségeket és alkalmazás várható élettartama közötti magas szintű architekturális összetettségét. Ezután a fentről lefelé megközelítés méri az alkalmazások, például a szolgáltatási szintek, integráció, karbantartási időszakokat, figyelés és insight működési követelményeinek. E szempontok mindegyikének lett elemzése, és figyelembe veszi, ha az eredmény lesz a pontszám, amely tükrözi a relatív nehézség az alkalmazás a cloud platform minden egyes áttelepíteni: IaaS, PaaS és SaaS.

Második a fentről lefelé értékelés kiértékeli az alkalmazást, a pénzügyi előnyeit, például a működési hatékonyságot, a teljes bekerülési Költséget, befektetés, vagy bármely egyéb megfelelő pénzügyi mérőszámok adja vissza. Emellett az értékelést is megvizsgálja a szezonalitás értékének az alkalmazás (vannak az év igény hirtelen megugró kihasználtság esetén) és a teljes számítási terhelés. Emellett úgy tűnik, a típusa (alkalmi/szakértő, mindig/alkalmanként bejelentkezve), támogatja a felhasználók, és ennek következtében a szükséges skálázhatóságot és a rugalmasságot. Végül azt állapítja meg az értékelés megvizsgálásával üzletmenet-folytonossági és rugalmassági követelményeket, amelyeket az alkalmazás, valamint függőségek a szolgáltatás megszakadását történjen, ha az alkalmazás futtatásához.

> [!TIP]
> Ez a megközelítés interjúk és üzleti és technikai közreműködők anecdotal visszajelzései igényel. Rendelkezésre álló kulcs időzítési a legnagyobb veszélyt. Az adatforrások anecdotal jellege megnehezíti pontos díj és az Időzítéstől becslések előállításához. Előre tervezze ütemezéseket, és ellenőrizze a gyűjtött adatokat.

## <a name="asset-driven-approach"></a>Az eszközintelligencia-képzéstől eltérve

Az eszközintelligencia-képzéstől eltérve az eszközöket, amelyek támogatják az alkalmazások áttelepítése alapján csomag biztosít. Ebben a megközelítésben statisztikai használati adatok egy konfigurációs felügyeleti adatbázis (CMDB) vagy más infrastruktúra-értékelési eszközök kéri le. Ez a megközelítés általában feltételezi, hogy egy IaaS-modell a központi telepítés kiindulópontként. Ebben a folyamatban az elemzés kiértékeli minden eszköz attribútumai: memória, processzor (CPU-magok), operációs rendszer tárolóhely, adatmeghajtókhoz száma a hálózati adapterek (NIC), IPv6, hálózati terheléselosztás, a fürtözést, az operációs verzió rendszer, az adatbázis (ha szükséges), tartományok támogatott, és a külső összetevők vagy szoftvercsomagokat, többek között. Az eszközök leltározott ebben a megközelítésben majd összhangban legyenek számítási feladatok vagy kérelmek csoportosítása és függőségi leképezés célokra.

> [!TIP]
> E módszerhez szükség van egy gazdag statisztikai használati adatok forrását. Az idő, a Hardverleltár vizsgálata, és adatokat gyűjteni az időzítési a legnagyobb veszélyt. Az alacsony szintű adatforrások lemarad eszközök vagy alkalmazások közötti függőségek. Tervezze meg legalább egy hónapot, a Hardverleltár vizsgálata. Függőségek üzembe helyezés előtt érvényesíteni.

## <a name="incremental-approach"></a>Növekményes módszer

CAF számos, hasonló egy növekményes megközelítés magas ajánlott. Esetén digitális hagyatéki tervezési, amely-adatbázisnak felel meg a többfázisú folyamat a következő:

- Kezdeti költségek elemzése: Pénzügyi érvényesítésre szükség, ha egy eszköz adatvezérelt megközelítéssel, beolvasni egy kezdeti költségek kiszámítása a teljes digitális hagyatéki nincs ésszerűsítés a fent ismertetett elindításához. Ez létrehozza a legrosszabb referenciaalap.

- Áttelepítés tervezéséhez: Miután hozzá lett rendelve egy Felhőstratégia csapat, hozhat létre egy kezdeti áttelepítés munkaterhelés-központú módszerével, kizárólag a kollektív ismeretek és korlátozott érdekelt interjúk alapján várakozó fájlok számát. Ez a megközelítés gyorsan összeállít egy kis igényű számítási feladatok felmérést, ezáltal elősegíti az együttműködést.

- A kiadás tervezési: Az áttelepítés várakozó fájlok számát, minden kiadás törli és újra Priorizált a leginkább releváns hatás az üzletmenetre koncentrálhat. A folyamat során meg a következő 5&ndash;10 számítási feladatok rangsorolt kiadások választotta volna. Ezen a ponton a Felhőstratégia csapat lenne fektet az idő, teljes körű munkaterhelés adatvezérelt megközelítés végrehajtásában. Az értékelés késleltetése, amíg egy kiadási feliratán, jobban figyelembe veszi a érintettek idején. Is késlelteti a befektetés teljes elemzés, addig az üzleti elindítja az származó erőfeszítéseket korábbi eredményeinek megtekintéséhez.

- Végrehajtási elemzése: Telepít át, mielőtt modernizálhatja vagy replikálásához bármely eszközre minden egyes fel kell mérni külön-külön és csoportos kiadás részeként. Ezen a ponton a kezdeti eszköz adatvezérelt megközelítés származó adatokat is mindkettőnek pontos méretezése és üzemeltetési korlátozások biztosításához.

> [!TIP]
> A növekményes megközelítés lehetővé teszi a zökkenőmentes tervezési, és gyorsított eredményeket. Nagyon fontos, hogy a résztvevő felek a módszer késleltetett döntéshozatal érdekében megismeréséhez. Ugyanilyen fontos, hogy minden egyes fázisában előzetesen kalkulált dokumentálni kell részletek adatvesztés elkerülése érdekében.

## <a name="next-steps"></a>További lépések

Miután egy módszert választja, a készlet lehessen gyűjteni.

> [!div class="nextstepaction"]
> [Szoftverleltározási adatok összegyűjtése](inventory.md)