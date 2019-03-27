---
title: 'CAF: Biztonsági alapkonfiguráció fegyelem fokozása'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Biztonsági alapkonfiguráció fegyelem fokozása
author: BrianBlanchard
ms.openlocfilehash: 28a971f56c9f8ada1d184bdc1cb3dbb9a17c3507
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299277"
---
# <a name="security-baseline-discipline-improvement"></a>Biztonsági alapkonfiguráció fegyelem fokozása

A biztonsági alapkonfiguráció fegyelem módját a hálózati eszközök és a legfontosabb helyezkednek el egy Felhőszolgáltató megoldás az adatok védelmét szolgáló szabályzatok összpontosít. Belül a felhő irányítási öt állapítják alapvető biztonsági tartalmazza a digitális hagyatéki és az adatok besorolását. Dokumentáció a kockázatokat, üzleti tűréshatár és kockázatcsökkentési stratégia az adatok, eszközök és hálózati biztonsági társított is tartalmaz. Technikai szempontból, ide is bevonása döntésekhez kapcsolatos [titkosítási](../../decision-guides/encryption/overview.md), [hálózati követelmények](../../decision-guides/software-defined-network/overview.md), [hibrid identitáskezelési stratégiák](../../decision-guides/identity/overview.md), és a [folyamatok](compliance-processes.md) felhőbeli fejlesztéséhez használt alapvető biztonsági szabályzatokat.

Ez a cikk ismerteti a vállalat jobban fejlesztéséhez és a biztonsági alapkonfiguráció fegyelem részletes is végezhetnek néhány lehetséges feladatot. Ezeket a feladatokat is kell bontani tervezése, létrehozása, bevezetése és a egy felhőalapú megoldás megvalósítása fázisai működik, amely vannak majd iterálni fejlesztését lehetővé teszi a egy [növekményes megközelítés a felhő cégirányítási](../journeys/overview.md#an-incremental-approach-to-cloud-governance).

![Bevezetési négy fázisa](../../_images/adoption-phases.png)

*1. ábra A növekményes megközelítés a felhő cégirányítási bevezetési fázisait.*

Nem lehet bármely egy dokumentum áthidalhatók az összes üzleti követelményeinek. Ezért ez a cikk ismerteti javasolt minimális és a lehetséges példa tevékenységek a cégirányítási érlelését egyes fázisaiban. Ezek a tevékenységek kezdeti célja segíteni az egy [házirend MVP](../journeys/overview.md#an-incremental-approach-to-cloud-governance) , valamint olyan keretrendszer, a növekményes házirend fejlődést szem előtt tartva. A felhő Cégirányítási csapat mennyi Döntse el, hogy ezeket a tevékenységeket, a biztonsági alapkonfiguráció irányítási lehetőségek javítása érdekében be kell.

> [!CAUTION]
> Sem a minimális vagy potenciális tevékenységek ebben a cikkben leírt adott vállalati szabályzatok vagy harmadik féltől származó megfelelőségi követelmények vannak igazítva. Ez az útmutató célja a beszélgetések, amely mindkét felhőalapú cégirányítási modell követelményekhez igazítását vezetnek megkönnyítése érdekében.

## <a name="planning-and-readiness"></a>Tervezés és a készültségi

Ebben a fázisban cégirányítási azt áthidalja a üzleti eredmények és gyakorlatban hasznosítható stratégiák közötti szakadékot. A folyamat során a a vezetőségi határozza meg az adott mérőszámok, követhesse a digitális hagyatéki vannak leképezve, és megkezdi a teljes áttelepítés sikeresebbé megtervezése.

**Minimális javasolt tevékenységek:**

- Kiértékelheti a [biztonsági alapterv eszközlánc](toolchain.md) beállítások.
- Fejlesztése a draft, architektúra-útmutató a dokumentum, és eljuttatja a kulcsfontosságú érintettekkel.
- Ismertetni, és a csapatok architektúra irányelvek fejlesztésének által érintett felhasználók között.
- Rangsorolt biztonsági tevékenységeket ad hozzá a migrálás mappájában várakozó fájlok számát.

**Lehetséges-tevékenységek:**

- Adja meg a besorolási sémát.
- Egy digitális hagyatéki tervezési folyamat leltározhatók az aktuális informatikai eszközök, az üzleti folyamatok működtetésére, és műveleteket végez.
- Magatartási egy [házirend áttekintése](../../governance/policy-compliance/what-is-a-cloud-policy-review.md) kezdje meg a meglévő vállalati informatikai biztonsági szabályzatok modernizálhatja, és az ismert kockázatok címzés MVP szabályzatokat határozhat meg.
- Tekintse át a felhőalapú platform biztonsági irányelveknek. Az Azure-ban találhatók a a [Microsoft szolgáltatás megbízható Platform](https://www.microsoft.com/trustcenter/stp/default.aspx).
- Határozza meg, hogy tartalmazza-e az alapvető biztonsági házirend egy [biztonsági fejlesztési életciklus](https://www.microsoft.com/securityengineering/sdl/).
- Hálózat, adatok, értékelje ki és üzleti eszközökkel kapcsolatos kockázatokat alapján a következő egy vagy három kiadások, és ezeket a szervezeti szintű mérőműszer.
- Tekintse át a Microsoft [legfontosabb trendeket a kiberbiztonsági](https://www.microsoft.com/security/operations/security-intelligence-report) jelentés az aktuális biztonsági világának áttekintése.
- Érdemes lehet egy [biztonsági fejlesztési és üzemeltetési](https://www.microsoft.com/en-us/securityengineering/devsecops) szervezeti szerepkör számára.

<!-- "en-us" location is required for the URL above. -->

## <a name="build-and-pre-deployment"></a>A build és a központi telepítés előtti

Technikai és felhasználóknak előfeltételnek szükségesek a sikeres környezet migrálása. Ez a folyamat a döntéseket hozhat, készültségi és alapvető infrastruktúra, amely az áttelepítés onnantól összpontosít.

**Minimális javasolt tevékenységek:**

- Alkalmazzon a [biztonsági alapterv eszközlánc](toolchain.md) szerint jelennek meg a központi telepítés előtti fázisban.
- Az architektúra-útmutató a dokumentum frissítése, és eljuttatja a kulcsfontosságú érintettekkel.
- Biztonsági feladatokat alkalmazhat a rangsorolt áttelepítési mappájában várakozó fájlok számát.
- Oktatási anyagok és a dokumentáció, tudatosság kommunikáció, ösztönzők, és egyéb programok meghajtó felhasználói bevezetésére segítségével fejleszthet.

**Lehetséges-tevékenységek:**

- Határozza meg a szervezet [titkosítási](../../decision-guides/encryption/overview.md) adatok felhőben üzemeltetett stratégiát.
- A felhőbeli üzemelő példány kiértékelése [identitás](../../decision-guides/identity/overview.md) stratégia. Határozza meg, hogy a felhőalapú identitáskezelési megoldás hogyan fogja párhuzamos telephelyközi vagy integrálható a helyszíni identitás-szolgáltatóktól.
- Hálózati határ házirendek meghatározása a [szoftveralapú hálózatkezelés (SDN)](../../decision-guides/software-defined-network/overview.md) tervezési annak biztosítására, biztonságos virtualizált hálózati funkciók.
- A szervezet kiértékelése [legalacsonyabb jogosultsági hozzáférés](/azure/active-directory/users-groups-roles/roles-delegate-by-task) szabályzatok, és használja a feladat-alapú szerepkörök az adott erőforráshoz nyújtanak hozzáférést.
- Biztonsággal és figyeléssel az összes cloud services és virtual machines mechanizmusok vonatkoznak.
- Automatizálhatja [biztonsági házirendek](../../decision-guides/policy-enforcement/overview.md) ahol csak lehetséges.
- Tekintse át az alapvető biztonsági házirendet, és határozza meg, ha szeretné-e a csomagokat, például az ajánlott eljárásokkal kapcsolatos megfelelően módosítsa a [biztonsági fejlesztési életciklus](https://www.microsoft.com/securityengineering/sdl/).

## <a name="adopt-and-migrate"></a>Elfogadja és áttelepítése

Áttelepítés az egy növekményes, amely a adatátviteli, a tesztelés és a bevezetési az alkalmazások és számítási feladatok egy meglévő digitális hagyatéki összpontosít.

**Minimális javasolt tevékenységek:**

- Áttelepíteni a [biztonsági alapterv eszközlánc](toolchain.md) éles üzem előtti telepítéséből.
- Az architektúra-útmutató a dokumentum frissítése, és eljuttatja a kulcsfontosságú érintettekkel.
- Fejlesztés az oktatási anyagok és a dokumentáció, tudatosság kommunikáció, ösztönzők, és egyéb programok meghajtó felhasználói elfogadása érdekében

**Lehetséges-tevékenységek:**

- Tekintse át a legújabb biztonsági alapterv, illetve a veszélyforrások elleni adatokat bármely új üzleti kockázatok azonosításához.
- A mérőműszer a szervezet tolerancia kezelni az új biztonsági kockázatok merülhetnek fel.
- Azonosíthatja a házirend eltéréseket, és javításokat kényszerítése.
- Állítsa be a biztonsági és a hozzáférés a verziókövetés automatizálását a maximális szabályzatoknak való megfelelés biztosítása érdekében.  
- Ellenőrizze, hogy az ajánlott eljárásokat az összeállítás során meghatározott / a rendszer megfelelően végrehajtja a központi telepítés előtti fázisait.
- Tekintse át a legalacsonyabb jogosultsági hozzáférési szabályzatokat, és állítsa be a hozzáférés-vezérlést a biztonság maximalizálása érdekében.
- Tesztelje a biztonsági alapkonfiguráció eszközlánc a számítási feladatok azonosítása és megoldása biztonsági rések ellen.

## <a name="operate-and-post-implementation"></a>Üzemeltetése és a megvalósítás utáni

Az átalakítás befejeződése után cégirányítási műveleteket kell élő és egy alkalmazás vagy munkaterhelés természetes élettartama. Ebben a fázisban cégirányítási azt tárgyalja a tevékenységek által gyakran után a megoldás bevezetése, és az átalakítási ciklus kerülési kezdődik.

**Minimális javasolt tevékenységek:**

- Ellenőrizze és/vagy finomíthatja a [biztonsági alapterv eszközlánc](toolchain.md).
- Testre szabhatja az értesítések és jelentések értesítenek a potenciális biztonsági problémákat.
- Az útmutató későbbi bevezetési folyamatok architektúra irányelveket pontosításához.
- Kommunikálni, és tájékoztassa az érintett csapatok rendszeres időközönként folyamatos architektúra irányelvek betartása érdekében.

**Lehetséges-tevékenységek:**

- Minták és a számítási feladatokhoz viselkedés felderítése, és konfigurálja a figyelés és jelentéskészítés a eszközöket, azonosítása és Önt a rendellenes tevékenység, a hozzáférés vagy az erőforrás használat.
- Folyamatosan frissíteni a figyelés és jelentéskészítés a szabályzatok a legfrissebb biztonsági réseivel, a biztonsági rések és a támadások észlelésére.
- Állítsa le a jogosulatlan hozzáférés és az erőforrásokat, előfordulhat, hogy sérült a biztonsága egy támadó letiltása, hogy olyan eljárásokat.
- Rendszeresen tekintse át a legújabb ajánlott biztonsági eljárások, és ahol lehetséges a biztonsági házirend, az automatizálást és monitorozási képességeket ajánlások érvényesek.

## <a name="next-steps"></a>További lépések

Most, hogy megismerkedett a felhőbeli biztonsági cégirányítási fogalmát, helyezze át további részleteket ismerhet meg [milyen biztonsági és ajánlott eljárásokkal kapcsolatos a Microsoft biztosít](azure-security-guidance.md) az Azure-hoz.

> [!div class="nextstepaction"]
> [További információ a biztonsági útmutató az Azure-ban](azure-security-guidance.md)
> [bemutatása az Azure Security](/azure/security/azure-security)
> [naplózási, jelentéskészítési és figyelésének ismertetése](../../decision-guides/log-and-report/overview.md)
