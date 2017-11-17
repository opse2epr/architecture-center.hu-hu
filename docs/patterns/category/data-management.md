---
title: "Adatok felügyeleti minták"
description: "Adatkezelés a fő eleme a felhőalapú alkalmazásokhoz, és hogyan befolyásolja a minőségi attribútumok többségét. Adatok általában üzemeltetett különböző helyeken és a teljesítmény, méretezhetőség és a rendelkezésre állási okokból több kiszolgáló között, és ez számos kihívást jelenthet. Például adatkonzisztencia fenn kell tartani, és adatokat általában kell különböző helyek közötti szinkronizálását."
keywords: "Kialakítási mintája"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: a009a06268f114ab7be4544dd81710612dabd8f4
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="data-management-patterns"></a>Adatok felügyeleti minták

[!INCLUDE [header](../../_includes/header.md)]

Adatkezelés a fő eleme a felhőalapú alkalmazásokhoz, és hogyan befolyásolja a minőségi attribútumok többségét. Adatok általában üzemeltetett különböző helyeken és a teljesítmény, méretezhetőség és a rendelkezésre állási okokból több kiszolgáló között, és ez számos kihívást jelenthet. Például adatkonzisztencia fenn kell tartani, és adatokat általában kell különböző helyek közötti szinkronizálását.

| Minta | Összefoglalás |
| ------- | ------- |
| [Gyorsítótár-Tartalékoljon](../cache-aside.md) | Adatok betöltése az igény szerinti egy adattárból a gyorsítótárba |
| [CQRS](../cqrs.md) | Elkülönítse műveletekkel kapcsolatos adatokat olvasni az operatív adatokat frissítő külön-felületek használatával. |
| [Esemény forrás](../event-sourcing.md) | Egy csak append tároló segítségével rögzítheti a teljes eseménysorozatokat, amelyek egy tartományban lévő adatokon műveleteit tartalmazzák. |
| [Index táblázat](../index-table.md) | Indexek létrehozása gyakran lekérdezések által hivatkozott adattárolókhoz mezőinek keresztül. |
| [Materializált nézet](../materialized-view.md) | Az adatokat egy vagy több adattárolókhoz keresztül előfeltöltött nézetek létrehozása az adatok nem ideális formázott kötelező lekérdezési műveletek. |
| [Horizontális](../sharding.md) | Adattároló felosztani vízszintes partíciók vagy szilánkok készlete. |
| [Statikus tartalom üzemeltetéséhez](../static-content-hosting.md) | Statikus tartalom központi telepítése a felhőalapú tároló szolgáltatás által biztosított közvetlenül az ügyfél számára. |
| [Kulcs valet](../valet-key.md) | A token vagy a kulcs, amely egy meghatározott erőforrás vagy a szolgáltatás korlátozott közvetlen hozzáférést biztosít az ügyfelek használja. |