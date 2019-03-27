---
title: Adatkezelési minták
titleSuffix: Cloud Design Patterns
description: Az adatkezelés a felhőalapú alkalmazások kulcsfontosságú eleme, és befolyásolja a legtöbb minőségi attribútumot. Az adatok általában különböző helyeken, több kiszolgálón találhatók a teljesítmény, a skálázhatóság vagy a rendelkezésre állás miatt, ez pedig különféle kihívásokat jelenthet. Fenn kell tartani például az adatok konzisztenciáját, és az adatokat jellemzően több különböző hely között kell szinkronizálni.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 25571a431836656856ed3f299455dfdb94ae3477
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298730"
---
# <a name="data-management-patterns"></a>Adatkezelési minták

[!INCLUDE [header](../../_includes/header.md)]

Az adatkezelés a felhőalapú alkalmazások kulcsfontosságú eleme, és befolyásolja a legtöbb minőségi attribútumot. Az adatok általában különböző helyeken, több kiszolgálón találhatók a teljesítmény, a skálázhatóság vagy a rendelkezésre állás miatt, ez pedig különféle kihívásokat jelenthet. Fenn kell tartani például az adatok konzisztenciáját, és az adatokat jellemzően több különböző hely között kell szinkronizálni.

|                        Mintázat                         |                                                                  Összegzés                                                                  |
|--------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------|
|            [Cache-Aside](../cache-aside.md)            |                                            Igény szerint tölthet be adatokat egy gyorsítótárba egy adattárolóból                                             |
|                   [CQRS](../cqrs.md)                   |                    Különböző felületek használatával elkülönítheti az adatolvasó műveleteket az adatfrissítő műveletektől.                     |
|         [Event Sourcing](../event-sourcing.md)         |               Használhat egy csak hozzáfűzéssel bővíthető tárat az egy tartomány adatain elvégzett műveleteket leíró események teljes sorozatának rögzítésére.               |
|            [Index Table](../index-table.md)            |                         Indexeket hozhat létre a lekérdezések által gyakran hivatkozott adattárbeli mezőkről.                          |
|      [Materialized View](../materialized-view.md)      | Létrehozhat előre kitöltött nézeteket egy vagy több adattár adataiból, ha az adatok formázása nem ideális a szükséges lekérdezési műveletekhez. |
|               [Sharding](../sharding.md)               |                                    Egy adattárat horizontális partíció- vagy szilánkkészletté oszthat fel.                                     |
| [Static Content Hosting](../static-content-hosting.md) |                   A statikus tartalmakat egy felhőalapú társzolgáltatásban helyezheti üzembe, amely közvetlenül az ügyfélnek közvetíti azt.                    |
|              [Valet Key](../valet-key.md)              |                 Jogkivonatot vagy kulcsot használhat, amely korlátozott közvetlen hozzáférést biztosít az ügyfelek számára egy adott erőforráshoz vagy szolgáltatáshoz.                 |
