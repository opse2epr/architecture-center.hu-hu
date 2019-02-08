---
title: 'CAF: Nagyvállalati – irányítás nagyvállalatoknak az többrétegű'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Nagyvállalati – irányítás nagyvállalatoknak az többrétegű
author: BrianBlanchard
ms.openlocfilehash: 42f4159ca0701c6a798f239359509a3e3b246c38
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899314"
---
# <a name="multiple-layers-of-governance-in-large-enterprises"></a>A nagyobb vállalatok a cégirányítási rétege

A nagyobb vállalatok cégirányítási többrétegű van szükség, ha nincsenek nagyobb összetettségi szintjével, amely a cégirányítási MVP és a későbbi cégirányítási fejlesztések kell megosztani.

Néhány gyakori példa az ilyen kiküszöböli a következők:

- Elosztott cégirányítást funkciók.
- Vállalati informatikai igazoló üzleti egység informatikai szervezeteknek.
- Vállalati informatikai támogató földrajzilag elosztott informatikai szervezetek.

Ez a cikk bemutatja, keresse meg az ilyen típusú összetettséget néhány módszer.

## <a name="large-enterprise-governance-is-a-team-sport"></a>Nagyvállalati cégirányítási egy csapat sport

Létrehozott nagyvállalatoknak gyakran a csapatok vagy alkalmazottak, akik a teljes folyamaton említett állapítják rendelkeznek. Tartson a folyamatban cégirányítási egy megközelítését egy csapat sport mutatja be.

Számos nagy méretű vállalatok számára felhőalapú cégirányítási állapítják lehet blockers való bevezetését. Az identitás, biztonság, műveletek, központi telepítések és konfigurációs tapasztalatra a felhőszolgáltatások kezelésében fejlesztése a vállalaton belül időt vesz igénybe. Informatikai cégirányítási házirend- és informatikai biztonsági azok alkalmazásfüggőségeit végrehajtási is innováció, hónapok szerint lassú vagy akár év. Az üzleti igény az innováció és a meglévő erőforrások védelmét kell irányítási Balancing olyan kényes.

A felhőben rejlő képességeit is akadályok az innovációt, de növeli a kockázatokat. A cégirányítási utazás megmutattuk, a példában szereplő vállalat hogyan hozott létre guardrails a kockázatok csökkentése érdekében. Ahelyett, amelyek mindegyike a környezet védelméhez szükséges állapítják, a felhő Cégirányítási csapatvezetők a kockázatalapú megközelítés annak a szabályozására, hogy mi is telepíthető, amíg a többi csapatban szükséges felhőbeli futamidejét hozhat létre. Ennél is fontosabb minden egyes csapat eléri a felhőalapú lejárat, cégirányítási alkalmazza a megoldásaik azok alkalmazásfüggőségeit. Mivel minden egyes csapat kiforrottabbá válik, és a teljes megoldás ad hozzá, a felhő Cégirányítási csapat fázis kapuk, lehetővé téve a további innovációs és gyarapodhatnak elfogadása is megnyithatja.

Ez a modell a felhő Cégirányítási csapat és a meglévő vállalati közötti partneri növekedését mutatja be csapatok (biztonsági, az informatikai szabályozással, hálózatkezelés, identitáskezelési és mások). Az út a cégirányítási MVP kezdődik, és a egy átfogó befejezési állapotának cégirányítási fejlesztések révén növekszik.

## <a name="requirements-to-supporting-such-a-team-sport"></a>Az ilyen csoport támogatása mellett, követelmények processzormagonként

Egy többrétegű cégirányítási modell első követelmény, hogy a cégirányítási hierarchia ismertetése. A follownig kérdés megválaszolásával segít megérteni az általános cégirányítási hierarchia:

- Felhő számlázási (a cloud services díjszabása) hogyan van lefoglalva a részlegek között?
- Cégirányítási feladatok kiosztás különböző vállalati informatikai és üzleti egységenként?
- Milyen típusú környezeteket hajtsa végre az egyes informatikai kezelése?

## <a name="central-governance-of-a-distributed-governance-hierarchy"></a>Az elosztott cégirányítást hierarchia központi irányítás

Eszközök, például felügyeleti csoportok lehetővé teszik a vállalati IT-hierarchia struktúra, amely megfelel az irányítási hierarchia létrehozása. Eszközök, mint például az Azure-tervek eszközöket alkalmazhat az adott hierarchia különböző rétegek. Az Azure tervezetek rendszerverzióval ellátott és különböző verzióit is alkalmazható a felügyeleti csoportokhoz, előfizetések vagy erőforráscsoportok. Az egyes ezen koncepciók közül a cégirányítási MVP részletesebben ismertetjük.

A fontos elemét alkotják, ezek az eszközök az egyes rendszer azon képessége, egy hierarchia több tervezetek alkalmazandó. Ez lehetővé teszi egy rétegzett folyamat kell irányítási. Az alábbiakban látható egy példa a cégirányítási hierarchikus alkalmazása:

- Vállalati IT: Vállalati informatikai szabványok és minden felhőre való áttérés a alkalmazni házirendeket hoz létre. Ez egy "Baseline" tervezetben materializálva van. Vállalati informatikai majd tulajdonosa az a felügyeleti csoport hierarchia, biztosítva, hogy egy verzióját az alapkonfiguráció a hierarchiában lévő összes előfizetés van hozzárendelve.
- Regionális vagy üzleti egység informatikai: Különböző informatikai csapata hozza létre a saját tervrajz létrehozása egy további rétegét cégirányítási is alkalmazhat. Ezeket a tervek additív szabályzatok és szabványok hozna létre. Fejlett, miután vállalati informatikai azokat szemléltető sikerült alkalmazni a felügyeleti csoport hierarchiában a alkalmazni csomópontok.
- A felhő bevezetésének csapatok: A cloud bevezetési csoportok cégirányítási követelmények kontextusában lehet kapcsolódni a részletes döntések és az alkalmazások és számítási feladatok végrehajtására. Esetenként a csapat is kérhet további Azure-erőforrás konzisztencia sablonok bevezetési erőfeszítések felgyorsításához.

A részleteket, minden szinten cégirányítási végrehajtásával kapcsolatos minden egyes csapat közötti koordinációt lesz szükség. A cégirányítási MVP és cégirányítási fejlesztések tartson a folyamatban leírt segítheti az adott koordináció igazítása.
