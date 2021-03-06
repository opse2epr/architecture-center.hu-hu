---
title: Kezelési és figyelési minták
titleSuffix: Cloud Design Patterns
description: A felhőalapú alkalmazások egy távoli adatközpontban futnak, ahol nem lehetséges teljesen uralni az infrastruktúrát, vagy bizonyos esetekben, az operációs rendszert. Emiatt a kezelés és a figyelés bonyolultabb, mint egy helyszíni üzemelő példánynál. Az alkalmazásoknak elérhetővé kell tenniük a futásidőre vonatkozó adatokat, hogy a rendszergazdák és a kezelők ezek segítségével kezeljék és figyeljék a rendszert, valamint támogathassák a változó üzleti feltételeket és a testreszabást anélkül, hogy le kellene állítani vagy újra üzembe kellene helyezni az alkalmazást.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 2b141a39b17f97ece011e93ca2f05cf5d2baeeea
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/17/2019
ms.locfileid: "59641093"
---
# <a name="management-and-monitoring-patterns"></a>Kezelési és figyelési minták

A felhőalapú alkalmazások egy távoli adatközpontban futnak, ahol nem lehetséges teljesen uralni az infrastruktúrát, vagy bizonyos esetekben, az operációs rendszert. Emiatt a kezelés és a figyelés bonyolultabb, mint egy helyszíni üzemelő példánynál. Az alkalmazásoknak elérhetővé kell tenniük a futásidőre vonatkozó adatokat, hogy a rendszergazdák és a kezelők ezek segítségével kezeljék és figyeljék a rendszert, valamint támogathassák a változó üzleti feltételeket és a testreszabást anélkül, hogy le kellene állítani vagy újra üzembe kellene helyezni az alkalmazást.

|                              Mintázat                               |                                                              Összegzés                                                              |
|--------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------|
|                   [Ambassador](../ambassador.md)                   |                 Olyan segítő szolgáltatásokat hozhat létre, amelyek egy otthoni használatra szánt szolgáltatás vagy alkalmazás nevében küldenek hálózati kéréseket.                 |
|        [Anti-Corruption Layer](../anti-corruption-layer.md)        |                       Egy előtér- vagy adapterréteget implementálhat egy korszerű alkalmazás és egy korábbi rendszer között.                       |
| [External Configuration Store](../external-configuration-store.md) |                A konfigurációs adatokat áthelyezheti az alkalmazás üzembehelyezési csomagjából egy központi helyre.                |
|          [Gateway Aggregation](../gateway-aggregation.md)          |                          Több egyéni kérést összesíthet egyetlen kérésbe egy átjáró segítségével.                           |
|           [Gateway Offloading](../gateway-offloading.md)           |                              A megosztott vagy specializált szolgáltatásműködést kiszervezheti egy átjáró proxyra.                              |
|              [Gateway Routing](../gateway-routing.md)              |                                   Átirányíthatja a kéréseket több szolgáltatásra egyetlen végpont használatával.                                    |
|   [Health Endpoint Monitoring](../health-endpoint-monitoring.md)   |   Rendszeres időközönként működés-ellenőrzéseket implementálhat egy alkalmazásban, amelyhez az elérhetővé tett végpontokon keresztül hozzáférhetnek külső eszközök.    |
|                      [Sidecar](../sidecar.md)                      |         Egy alkalmazás összetevőit külön folyamatban vagy tárolóban helyezheti üzembe, így elkülönítést és beágyazást biztosíthat.          |
|                    [Strangler](../strangler.md)                    | Növekményesen migrálhat egy korábbi rendszert oly módon, hogy egyes funkciódarabokat fokozatosan új alkalmazásokra és szolgáltatásokra cserél. |
