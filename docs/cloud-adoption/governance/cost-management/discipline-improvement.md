---
title: 'CAF: A Cost Management fegyelem fokozása'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: A Cost Management fegyelem fokozása
author: BrianBlanchard
ms.openlocfilehash: 34975d195a95b1a85ada96efe8c76a6138385ec1
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899463"
---
# <a name="cost-management-discipline-improvement"></a>A Cost Management fegyelem fokozása

A Cost Management fegyelem próbál meg címet alapvető üzleti kockázatai kapcsolatos számítási feladatok felhőbeli üzemeltetése esetén felmerülő költségek. Felhőalapú irányítási öt állapítják belül Cost Management részt vesz a cél az létrehozása és üzemeltetése egy tervezett költség ciklus felhőalapú erőforrások felhasználási és szabályozása.

Ez a cikk ismerteti a lehetséges feladatokat a vállalat fejlesztését, és a Cost Management fegyelem részletes hajtsa végre. Ezeket a feladatokat is kell bontani tervezése, létrehozása, bevezetése és a egy felhőalapú megoldás megvalósítása fázisai működik, amely vannak majd iterálni fejlesztését lehetővé teszi a egy [növekményes megközelítés a felhő cégirányítási](../journeys/overview.md#an-incremental-approach-to-cloud-governance).

![Bevezetési négy fázisa](../../_images/adoption-phases.png)

*1. ábra A növekményes megközelítés a felhő cégirányítási bevezetési fázisait.*

Nincs egyetlen dokumentum összes vállalatok igényeit is figyelembe. Ezért ez a cikk ismerteti javasolt minimális és a lehetséges példa tevékenységek a cégirányítási érlelését egyes fázisaiban. Ezek a tevékenységek kezdeti célja segíteni az egy [házirend MVP](../journeys/overview.md#an-incremental-approach-to-cloud-governance) , valamint olyan keretrendszer, a növekményes házirend fejlődést szem előtt tartva. A felhő Cégirányítási csapat mennyi Döntse el, hogy ezeket a tevékenységeket a Cost Management irányítási lehetőségek javítása érdekében be kell.

> [!CAUTION]
> Sem a minimális vagy potenciális tevékenységek ebben a cikkben leírt adott vállalati szabályzatok vagy harmadik féltől származó megfelelőségi követelmények vannak igazítva. Ez az útmutató célja a beszélgetések, amely mindkét felhőalapú cégirányítási modell követelményekhez igazítását vezetnek megkönnyítése érdekében.

## <a name="planning-and-readiness"></a>Tervezés és a készültségi

Ebben a fázisban cégirányítási azt áthidalja a üzleti eredmények és gyakorlatban hasznosítható stratégiák közötti szakadékot. A folyamat során a a vezetőségi határozza meg az adott mérőszámok, követhesse a digitális hagyatéki vannak leképezve, és megkezdi a teljes áttelepítés sikeresebbé megtervezése.

**Minimális javasolt tevékenységek:**

* Kiértékelheti a [Cost Management eszközlánc](toolchain.md) beállítások.
* Fejlesztése a draft, architektúra-útmutató a dokumentum, és eljuttatja a kulcsfontosságú érintettekkel.
* Ismertetni, és a csapatok architektúra irányelvek fejlesztésének által érintett felhasználók között.

**Lehetséges-tevékenységek:**

* Győződjön meg arról, amelyek az üzleti indoklás támogatása a stratégia költségvetési döntéseket hozhat.
* Ellenőrizze a tanulási metrikák, amellyel a sikeres lefoglalt finanszírozási jelentést.
* Ismerje meg a kívánt felhő számlázási modell, amely befolyásolja, hogyan felhőköltségek elszámolni.
* Megismerkedhet a digitális hagyatéki csomaggal, és ellenőrizze a pontos költségszámítási elvárásainak.
* Állapítsa meg, hogy jobban "utólagos fizetést", illetve, hogy egy precommitment megvásárlása nagyvállalati szerződés vásárlási lehetőségek kiértékelése.
* Üzleti célok összekapcsolása, tervezett költségvetések igazítása és költségvetési tervek szükség szerint módosíthatja.
* Fejlesztés a célok, és költségvetés-jelentéskészítési mechanizmus, amellyel értesíteni technikai és üzleti résztvevőkkel minden költség ciklus végén.

## <a name="build-and-pre-deployment"></a>A build és a központi telepítés előtti

Technikai és felhasználóknak előfeltételnek környezet sikeres áttelepítéséhez szükségesek. Ez a folyamat a döntéseket hozhat, készültségi és alapvető infrastruktúra, amely az áttelepítés onnantól összpontosít.

**Minimális javasolt tevékenységek:**

* Alkalmazzon a [Cost Management eszközlánc](toolchain.md) szerint jelennek meg a központi telepítés előtti fázisban.
* Az architektúra-útmutató a dokumentum frissítése, és eljuttatja a kulcsfontosságú érintettekkel.
* Oktatási anyagok és a dokumentáció, tudatosság kommunikáció, ösztönzők, és egyéb programok meghajtó felhasználói bevezetésére segítségével fejleszthet.
* Határozza meg, ha a vásárlási követelmény igazítása a költségvetés és a célokat.

**Lehetséges-tevékenységek:**

* Igazítás a költségvetési terveket biztosít a [előfizetés stratégia](../../decision-guides/subscriptions/overview.md) , amely meghatározza, hogy a core tulajdonjog modellt.
* Használja ki a [erőforrás konzisztencia stratégia](../../decision-guides/resource-consistency/overview.md) architektúra kényszerítése és irányelveket költségek időbeli alakulása.
* Határozza meg, hogy vannak-e bármilyen költség rendellenességeket, amelyek befolyásolják a bevezetési és áttelepítési terveket.

## <a name="adopt-and-migrate"></a>Elfogadja és áttelepítése

Áttelepítés az egy növekményes, amely a adatátviteli, a tesztelés és a bevezetési az alkalmazások és számítási feladatok egy meglévő digitális hagyatéki összpontosít.

**Minimális javasolt tevékenységek:**

* Áttelepíteni a [Cost Management eszközlánc](toolchain.md) éles üzem előtti telepítéséből.
* Az architektúra-útmutató a dokumentum frissítése, és eljuttatja a kulcsfontosságú érintettekkel.
* Oktatási anyagok és a dokumentáció, tudatosság kommunikáció, ösztönzők, és egyéb programok meghajtó felhasználói bevezetésére segítségével fejleszthet.

**Lehetséges-tevékenységek:**

* A felhő számlázási modell megvalósításához.
* Győződjön meg arról, hogy a költségvetés tükrözik a tényleges kiadásokat során minden kiadással, és szükség szerint módosíthatja.
* Monitorozhatja a változásokat a költségvetési tervek, és ellenőrizze az érintettekkel való, ha további bejelentkezési kompromisszumot van szükség.
* Frissítse a módosításokat az architektúra-útmutató a dokumentumot, tükrözik a tényleges költségek.

## <a name="operate-and-post-implementation"></a>Üzemeltetése és a megvalósítás utáni

Az átalakítás befejeződése után cégirányítási műveleteket kell élő és egy alkalmazás vagy munkaterhelés természetes élettartama. Ebben a fázisban cégirányítási azt tárgyalja a tevékenységek által gyakran után a megoldás bevezetése, és az átalakítási ciklus kerülési kezdődik.

**Minimális javasolt tevékenységek:**

* Testre szabhatja a [Cost Management eszközlánc](toolchain.md) a szervezet cost management igények változása alapján.
* Vegye figyelembe, hogy a tényleges kiadásokat bármely értesítések és jelentések automatizálása.
* Az útmutató későbbi bevezetési folyamatok architektúra irányelvek pontosításához.
* Az architektúra irányelvek betartása folyamatos biztosítása érdekében rendszeresen érintett csapatok oktatása.

**Lehetséges-tevékenységek:**

* Hajtsa végre a negyedéves felhőalapú üzleti felülvizsgálat való kommunikációhoz, az üzleti és az kapcsolódó költségek értéket.
* Módosítsa a tényleges kiadásokat módosításait negyedéves terveket.
* Határozza meg, az üzleti egység előfizetések P & Ls pénzügyi igazítását.
* Elemezheti az érdekelt érték és jelentéskészítési módszerrel havi költséget.
* Használaton eszközök javítása, és határozza meg, érdemes Folytatás zajlik.
* Ők olyan eltéréseket és rendellenességeket a terv és a tényleges kiadásokat közötti észleli.
* Segítséget nyújt a felhő bevezetésének csapatok és a Felhőstratégia csapat megértése és megoldása a rendellenességeket.

## <a name="next-steps"></a>További lépések

Most, hogy megismerkedett a felhőalapú identitáskezelést fogalmát, vizsgálja meg a [Cost Management eszközlánc](toolchain.md) azonosításához az Azure-eszközök és funkciók, amelyek az Azure Cost Management cégirányítási tantárgy fejlesztése során lesz szüksége Platform.

> [!div class="nextstepaction"]
> [A Cost Management eszközláncban az Azure-hoz](toolchain.md)