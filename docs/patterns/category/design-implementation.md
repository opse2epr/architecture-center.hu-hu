---
title: "Tervezési és megvalósítási minták"
description: "Tervezési magában foglalja a konzisztencia és koherencia összetevő tervezési és telepítési, karbantartási követelmények egyszerűsítése érdekében a felügyeleti és fejlesztői és újrahasznosításának ahhoz, hogy összetevők és alrendszereket használható egyéb alkalmazások és más tényezők forgatókönyvek. A tervezési és megvalósítási fázis során hozott döntések hatalmas hatást gyakorolnak a minőségi és a teljes tulajdonosi költség, a felhőben üzemeltetett alkalmazások és szolgáltatások."
keywords: "Kialakítási mintája"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: 3d6e528b8c88c6fcc265d6425dcdae6fae5166fb
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="design-and-implementation-patterns"></a>Tervezési és megvalósítási minták

Tervezési magában foglalja a konzisztencia és koherencia összetevő tervezési és telepítési, karbantartási követelmények egyszerűsítése érdekében a felügyeleti és fejlesztői és újrahasznosításának ahhoz, hogy összetevők és alrendszereket használható egyéb alkalmazások és más tényezők forgatókönyvek. A tervezési és megvalósítási fázis során hozott döntések hatalmas hatást gyakorolnak a minőségi és a teljes tulajdonosi költség, a felhőben üzemeltetett alkalmazások és szolgáltatások.

| Minta | Összefoglalás |
| ------- | ------- |
| [Diplomata](../ambassador.md) | Segítő szolgáltatások által küldött kérelmek nevében fogyasztói szolgáltatás vagy alkalmazás létrehozására. |
| [Víruskereső sérülés réteg](../anti-corruption-layer.md) | A modern alkalmazás- és egy örökölt közötti homlokzat vagy adapter rétegben megvalósításához. |
| [Frontends a háttérkiszolgálókon](../backends-for-frontends.md) | Hozzon létre külön háttér szolgáltatások által meghatározott előtér-alkalmazások vagy felületeket. |
| [CQRS](../cqrs.md) | Elkülönítse műveletekkel kapcsolatos adatokat olvasni az operatív adatokat frissítő külön-felületek használatával. |
| [Számítási erőforrás összevonása](../compute-resource-consolidation.md) | Több feladatokat vagy műveleteket egyetlen számítási egységbe összesítése |
| [Külső Beállítástárolóval](../external-configuration-store.md) | Az alkalmazás központi telepítési csomag kívül a konfigurációs adatok áthelyezése egy központi helyen. |
| [Átjáró aggregáció](../gateway-aggregation.md) | Segítségével egy átjáró több egyes kérelmeket az egy kérelemhez. |
| [Átjáró-Feladatkiszervezést](../gateway-offloading.md) | Megosztott vagy speciális szolgáltatás funkcióit egy átjáró proxyként kiürítése. |
| [Átjáró-útválasztó](../gateway-routing.md) | Útvonal kérelmek több szolgáltatások használatával egy végpontot. |
| [Vezető választás](../leader-election.md) | Koordinálja az elosztott alkalmazásban lévő együttműködés task példányokat gyűjteménye egy példánya, amely azt feltételezi, hogy a többi példány felelős a vezetőjeként megválasztását által végrehajtott műveletekről. |
| [Adatcsatornák és a szűrők](../pipes-and-filters.md) | Egy különálló elemek, amelyek felhasználhatók egy sorozat komplex feldolgozását végző feladat lebontva. |
| [Oldalkocsi](../sidecar.md) | Egy alkalmazás elkülönítési és beágyazás egy külön folyamatban vagy a tároló összetevőinek telepítéséhez. |
| [Statikus tartalom üzemeltetéséhez](../static-content-hosting.md) | Statikus tartalom központi telepítése a felhőalapú tároló szolgáltatás által biztosított közvetlenül az ügyfél számára. |
| [Strangler](../strangler.md) | Növekményesen áttelepítése egy korábbi rendszer fokozatosan cseréje nehezen funkció az új alkalmazásokkal és szolgáltatásokkal. |