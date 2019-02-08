---
title: 'CAF: Üzembe helyezés gyorsítás fegyelem fokozása'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Üzembe helyezés gyorsítás fegyelem fokozása
author: alexbuckgit
ms.openlocfilehash: 98192c777d8866efb01544737e8cabea6354c4d7
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899479"
---
# <a name="deployment-acceleration-discipline-improvement"></a>Üzembe helyezés gyorsítás fegyelem fokozása

A telepítési gyorsítás szakágankénti összpontosít, győződjön meg arról, hogy az erőforrások üzembe helyezése és következetesen és repeatably konfigurált házirendek létrehozása és megfelelőségi őket életciklusuk teljes ideje marad. Belül a öt oktatnak, felhőalapú Cégirányítási, üzembe helyezési gyorsítás tartalmaz döntések kapcsolódó automatizálhatja központi telepítések, forrás-szabályozása üzembe helyezési összetevők, figyelés üzembe helyezett erőforrások karbantartása a kívánt állapotban, és a naplózás minden olyan megfelelőségi probléma.

Ez a cikk ismerteti a vállalat jobban fejlesztéséhez és üzembe helyezési gyorsítás tantárgy részletes is végezhetnek néhány lehetséges feladatot. Ezeket a feladatokat is kell bontani tervezése, létrehozása, bevezetése és a egy felhőalapú megoldás megvalósítása fázisai működik, amely vannak majd iterálni fejlesztését lehetővé teszi a egy [növekményes megközelítés a felhő cégirányítási](../journeys/overview.md#an-incremental-approach-to-cloud-governance).

![Bevezetési négy fázisa](../../_images/adoption-phases.png)

*1. ábra A növekményes megközelítés a felhő cégirányítási bevezetési fázisait.*

Nem lehet bármely egy dokumentum áthidalhatók az összes üzleti követelményeinek. Ezért ez a cikk ismerteti javasolt minimális és a lehetséges példa tevékenységek a cégirányítási érlelését egyes fázisaiban. Ezek a tevékenységek kezdeti célja segíteni az egy [házirend MVP](../journeys/overview.md#an-incremental-approach-to-cloud-governance) , valamint olyan keretrendszer, a növekményes házirend fejlődést szem előtt tartva. A felhő Cégirányítási csapat kell mennyi Döntse el, hogy be ezeket a tevékenységeket, az identitásra építve irányítási lehetőségek javítása érdekében.

> [!CAUTION]
> Sem a minimális vagy potenciális tevékenységek ebben a cikkben leírt adott vállalati szabályzatok vagy harmadik féltől származó megfelelőségi követelmények vannak igazítva. Ez az útmutató célja a beszélgetések, amely mindkét felhőalapú cégirányítási modell követelményekhez igazítását vezetnek megkönnyítése érdekében.

## <a name="planning-and-readiness"></a>Tervezés és a készültségi

Ebben a fázisban cégirányítási azt áthidalja a üzleti eredmények és gyakorlatban hasznosítható stratégiák közötti szakadékot. A folyamat során a a vezetőségi határozza meg az adott mérőszámok, követhesse a digitális hagyatéki vannak leképezve, és megkezdi a teljes áttelepítés sikeresebbé megtervezése.

**Minimális javasolt tevékenységek:**

- Kiértékelheti a [üzembe helyezési gyorsítás eszközlánc](toolchain.md) lehetőségeket és a egy a szervezet számára megfelelő hibrid stratégia megvalósításához.
- Fejlesztése a draft, architektúra-útmutató a dokumentum, és eljuttatja a kulcsfontosságú érintettekkel.
- Ismertetni, és a csapatok architektúra irányelvek fejlesztésének által érintett felhasználók között.
- Fejlesztői csapatok és DevSecOps alapelvek és stratégiák és teljes mértékben automatizált központi telepítések központi telepítési gyorsítás tantárgy fontosságát az informatikai részleg betanításához.

**Lehetséges-tevékenységek:**

- Szerepkörök és a felhőbeli üzembe helyezés gyorsítás szabályozó hozzárendelések megadása.

## <a name="build-and-pre-deployment"></a>A build és a központi telepítés előtti

**Minimális javasolt tevékenységek:**

- Új felhőalapú alkalmazásokat vezessen be teljesen automatikus telepítéseket a fejlesztési folyamat korai szakaszaiban. A befektetés az vizsgálati folyamatok megbízhatóságának javítása, és a konzisztencia érdekében a fejlesztési, minőségbiztosítási és éles környezetek között.
- Store összes üzembe helyezési összetevők például a központi telepítési sablonok vagy konfigurációs parancsfájlokat, például a GitHub vagy az Azure DevOps source-control platform használatával.
- Egy tesztelési implementálása előtt fontolja meg a [üzembe helyezési gyorsítás eszközlánc](toolchain.md), így meg arról, hogy leegyszerűsíti a lehető legnagyobb mértékben környezetekben. Visszajelzés alkalmazni a próbaüzem tesztek végrehajtása során a központi telepítés előtti fázist, ismételje meg igény szerint.
- Az alkalmazások logikai és fizikai architektúrájának vizsgálja, és az alkalmazás-erőforrások üzembe helyezésének automatizálása, vagy az egyéb felhőalapú erőforrások használatával architektúra részeit javíthatja üzleti lehetőségeket fedezhet.
- Frissítse a architektúra irányelvek dokumentum közé tartoznak az üzembe helyezés és a felhasználó bevezetési terveket és terjeszteni kívánt főbb érintettek.
- Tájékoztassa a felhasználókkal és leginkább az architektúra irányelvek által érintett csoportok továbbra is.

**Lehetséges-tevékenységek:**

- Meghatározása a folyamatos integráció és folyamatos üzembe helyezés (CI/CD) folyamat teljes körű kezelését a felszabadítása frissíti az alkalmazás a fejlesztési, minőségbiztosítási és éles környezetben keresztül.

## <a name="adopt-and-migrate"></a>Elfogadja és áttelepítése

Áttelepítés az egy növekményes, amely a adatátviteli, a tesztelés és a bevezetési az alkalmazások és számítási feladatok egy meglévő digitális hagyatéki összpontosít.

**Minimális javasolt tevékenységek:**

- Áttelepíteni a [üzembe helyezési gyorsítás eszközlánc](toolchain.md) a fejlesztéstől az éles környezetbe.
- Az architektúra-útmutató a dokumentum frissítése, és eljuttatja a kulcsfontosságú érintettekkel.
- Oktatási anyagok és a dokumentáció, tudatosság kommunikáció, ösztönzők és más programok meghajtó fejlesztő és informatikai bevezetési fejleszthet.

**Lehetséges-tevékenységek:**

- Ellenőrizze, hogy az ajánlott eljárásokat, a build és a központi telepítés előtti szakaszában meghatározott megfelelően lesznek végrehajtva.
- Győződjön meg arról, hogy egyes alkalmazások vagy munkaterhelések futnak igazítja a gyorsítás központi telepítési stratégiát kiadás előtt.

## <a name="operate-and-post-implementation"></a>Üzemeltetése és a megvalósítás utáni

Az átalakítás befejeződése után cégirányítási műveleteket kell élő és egy alkalmazás vagy munkaterhelés természetes élettartama. Ebben a fázisban cégirányítási azt tárgyalja a tevékenységek által gyakran után a megoldás bevezetése, és az átalakítási ciklus kerülési kezdődik.

**Minimális javasolt tevékenységek:**

- Testre szabhatja a [üzembe helyezési gyorsítás eszközlánc](toolchain.md) a szervezet változó identitás igények szerint végbement változások alapján.
- Automatizálhatja az értesítések és jelentések értesítenek a potenciális konfigurációs problémákat és a rosszindulatú fenyegetések.
- A figyelő és az alkalmazás- és erőforrás-használati jelentés.
- Üzembe helyezés utáni metrikák jelentés és az érdekelt felek terjesztése.
- Vizsgálja felül a architektúra irányelvek későbbi bevezetési folyamatok irányításához.
- Továbbra is kommunikálni, és az érintett felhasználók és a csapatok rendszeres időközönként architektúra irányelvek betartása folyamatos biztosítása érdekében.

**Lehetséges-tevékenységek:**

- Kívánt állapot konfigurációs figyelési és jelentéskészítési eszköz konfigurálása.
- Rendszeresen tekintse át a konfigurációs eszközök és parancsfájlok folyamatok és a gyakori problémák azonosításához.
- Érett DevSecOps eljárások és lebontásában szervezeti vezethet, fejlesztési, műveleteket és biztonsági csoportokkal végzett munka.

## <a name="next-steps"></a>További lépések

Most, hogy megismerkedett a felhőalapú identitáskezelést fogalmát, vizsgálja meg a [identitásra építve eszközlánc](toolchain.md) azonosításához az Azure-eszközök és funkciók, amelyek az identitásra építve cégirányítási tantárgy fejlesztése során lesz szüksége a Az Azure platformon.

> [!div class="nextstepaction"]
> [Identitás alapkonfiguráció eszközláncban az Azure-hoz](toolchain.md)
