---
title: "Kezelési és figyelési minták"
description: "Felhő alkalmazások futnak a távoli adatközpontban Ha nem rendelkezik teljes körű vezérlés az infrastruktúra vagy bizonyos esetekben az operációs rendszer. Ez tehet kezelési és figyelési bonyolultabb, mint a helyszíni telepítéshez. Alkalmazások fel kell fednie futásidejű rendszergazdák és a kezelők kezelni és megfigyelni a rendszer, valamint változó üzleti követelmények és a Testreszabás anélkül, hogy a leállítható vagy újratelepíteni az alkalmazás által használt adatok."
keywords: "Kialakítási mintája"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: 6281f7e5c62593cc60727c994d5dba2c52976b75
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="management-and-monitoring-patterns"></a>Kezelési és figyelési minták

Felhő alkalmazások futnak a távoli adatközpontban Ha nem rendelkezik teljes körű vezérlés az infrastruktúra vagy bizonyos esetekben az operációs rendszer. Ez tehet kezelési és figyelési bonyolultabb, mint a helyszíni telepítéshez. Alkalmazások fel kell fednie futásidejű rendszergazdák és a kezelők kezelni és megfigyelni a rendszer, valamint változó üzleti követelmények és a Testreszabás anélkül, hogy a leállítható vagy újratelepíteni az alkalmazás által használt adatok.

| Minta | Összefoglalás |
| ------- | ------- |
| [Diplomata](../ambassador.md) | Segítő szolgáltatások által küldött kérelmek nevében fogyasztói szolgáltatás vagy alkalmazás létrehozására. |
| [Víruskereső sérülés réteg](../anti-corruption-layer.md) | A modern alkalmazás- és egy örökölt közötti homlokzat vagy adapter rétegben megvalósításához. |
| [Külső Beállítástárolóval](../external-configuration-store.md) | Az alkalmazás központi telepítési csomag kívül a konfigurációs adatok áthelyezése egy központi helyen. |
| [Átjáró aggregáció](../gateway-aggregation.md) | Segítségével egy átjáró több egyes kérelmeket az egy kérelemhez. |
| [Átjáró-Feladatkiszervezést](../gateway-offloading.md) | Megosztott vagy speciális szolgáltatás funkcióit egy átjáró proxyként kiürítése. |
| [Átjáró-útválasztó](../gateway-routing.md) | Útvonal kérelmek több szolgáltatások használatával egy végpontot. |
| [Végpont állapotfigyelés](../health-endpoint-monitoring.md) | Funkcionális ellenőrzések megvalósítását a külső eszközök rendszeres időközönként kitett végpontok keresztül elérhető alkalmazást. |
| [Oldalkocsi](../sidecar.md) | Egy alkalmazás elkülönítési és beágyazás egy külön folyamatban vagy a tároló összetevőinek telepítéséhez. |
| [Strangler](../strangler.md) | Növekményesen áttelepítése egy korábbi rendszer fokozatosan cseréje nehezen funkció az új alkalmazásokkal és szolgáltatásokkal. |
