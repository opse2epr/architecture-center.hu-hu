---
title: 'CAF: Identitás alapkonfiguráció fegyelem fokozása'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Identitás alapkonfiguráció fegyelem fokozása
author: BrianBlanchard
ms.openlocfilehash: c96a638af549782fec22b2068c9b4943df4b943a
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899049"
---
# <a name="identity-baseline-discipline-improvement"></a>Identitás alapkonfiguráció fegyelem fokozása

Az identitásra építve fegyelem módját a házirendekben, amelyek a konzisztencia és a felhasználói identitásokat, függetlenül a felhőszolgáltató folytonosságának biztosítása üzemeltető összpontosít, az alkalmazást vagy munkaterhelést futtatják. Az öt oktatnak, felhőalapú irányítási belül identitásra építve döntések tartalmaz kapcsolatos a [hibrid identitás stratégia](../../decision-guides/identity/overview.md), értékelési és identitás-adattárak megvalósítását bővítmény egyszeri bejelentkezés (azonos bejelentkezés) , naplózhatja és monitorozhatja az illetéktelen vagy rosszindulatú. Bizonyos esetekben is jelenthet modernizálása, konszolidálhatja vagy több identitásszolgáltató integrálása döntéseket hozhat.

Ez a cikk ismerteti a vállalat jobban fejlesztéséhez és az identitásra építve fegyelem részletes is végezhetnek néhány lehetséges feladatot. Ezeket a feladatokat is kell bontani tervezése, létrehozása, bevezetése és a egy felhőalapú megoldás megvalósítása fázisai működik, amely vannak majd iterálni fejlesztését lehetővé teszi a egy [növekményes megközelítés a felhő cégirányítási](../journeys/overview.md#an-incremental-approach-to-cloud-governance).

![Bevezetési négy fázisa](../../_images/adoption-phases.png)

*1. ábra A növekményes megközelítés a felhő cégirányítási bevezetési fázisait.*

Nem lehet bármely egy dokumentum áthidalhatók az összes üzleti követelményeinek. Ezért ez a cikk ismerteti javasolt minimális és a lehetséges példa tevékenységek a cégirányítási érlelését egyes fázisaiban. Ezek a tevékenységek kezdeti célja segíteni az egy [házirend MVP](../journeys/overview.md#an-incremental-approach-to-cloud-governance) , valamint olyan keretrendszer, a növekményes házirend fejlődést szem előtt tartva. A felhő Cégirányítási csapat kell mennyi Döntse el, hogy be ezeket a tevékenységeket, az identitásra építve irányítási lehetőségek javítása érdekében.

> [!CAUTION]
> Sem a minimális vagy potenciális tevékenységek ebben a cikkben leírt adott vállalati szabályzatok vagy harmadik féltől származó megfelelőségi követelmények vannak igazítva. Ez az útmutató célja a beszélgetések, amely mindkét felhőalapú cégirányítási modell követelményekhez igazítását vezetnek megkönnyítése érdekében.

## <a name="planning-and-readiness"></a>Tervezés és a készültségi

Ebben a fázisban cégirányítási azt áthidalja a üzleti eredmények és gyakorlatban hasznosítható stratégiák közötti szakadékot. A folyamat során a a vezetőségi határozza meg az adott mérőszámok, követhesse a digitális hagyatéki vannak leképezve, és megkezdi a teljes áttelepítés sikeresebbé megtervezése.

**Minimális javasolt tevékenységek:**

* Kiértékelheti a [identitás eszközlánc](toolchain.md) lehetőségeket és a egy a szervezet számára megfelelő hibrid stratégia megvalósításához.
* Fejlesztése a draft, architektúra-útmutató a dokumentum, és eljuttatja a kulcsfontosságú érintettekkel.
* Ismertetni, és a csapatok architektúra irányelvek fejlesztésének által érintett felhasználók között.

**Lehetséges-tevékenységek:**

* Adja meg a szerepkörök és -hozzárendelések, szabályozzák az identitás és hozzáférés-kezelés a felhőben.
* A helyszíni csoportjait határozzák meg, és megfelelő felhőalapú szerepkörök hozzárendelése.
* Készlet Identitásszolgáltatók (beleértve az egyéni alkalmazások által használt adatbázis-vezérelt identitások).
* Fontolja meg a konszolidálási és az identitás-szolgáltatóktól, ahol duplikáció van, egyszerűsítése érdekében az általános identitáskezelési megoldás integrációs lehetőségeit.
* Hibrid identitás-szolgáltatóktól kompatibilitásának kiértékelése.
* Az identitás-szolgáltatóktól, amelyek nem kompatibilis a hibrid összevonás vagy helyettesítő lehetőségeinek kiértékeléséhez.

## <a name="build-and-pre-deployment"></a>A build és a központi telepítés előtti

Technikai és felhasználóknak előfeltételnek környezet sikeres áttelepítéséhez szükségesek. Ez a folyamat a döntéseket hozhat, készültségi és alapvető infrastruktúra, amely az áttelepítés onnantól összpontosít.

**Minimális javasolt tevékenységek:**

* Egy tesztelési implementálása előtt fontolja meg a [identitás eszközlánc](toolchain.md), így meg arról, hogy leegyszerűsíti a lehető legnagyobb mértékben a felhasználói élményt.
* Visszajelzés alkalmazása kísérleti tesztek a központi telepítés előtti be. Ismételje meg addig, amíg az eredmények elfogadhatóak.
* Frissítse a architektúra irányelvek dokumentum közé tartoznak az üzembe helyezés és a felhasználó bevezetési terveket és terjeszteni kívánt főbb érintettek.
* Fontolja meg egy korai felhasználója program létrehozó és a felhasználók egy korlátozott számú elérhetőek.
* Tájékoztassa a felhasználókkal és leginkább az architektúra irányelvek által érintett csoportok továbbra is.

**Lehetséges-tevékenységek:**

* A logikai és fizikai architektúra kiértékeléséhez, és döntse el, egy [hibrid identitás stratégia](../../decision-guides/identity/overview.md).
* Térkép identitás-hozzáférési felügyeleti házirendek, például a bejelentkezési azonosító hozzárendeléseket, és válassza ki a megfelelő hitelesítési módszert az Azure ad.
  * Összevont, engedélyezze a rendszergazdai fiókok bérlői korlátozások.
* Integrálhatja a helyszíni és felhőbeli címtárakban.
* Fontolja meg az alábbi hozzáférési minták:
  * [Legkisebb jogosultság hozzáférés](/windows-server/identity/ad-ds/plan/security-best-practices/implementing-least-privilege-administrative-models) modell
  * [Az emelt szintű identitásra építve](/azure/active-directory/privileged-identity-management/pim-configure) hozzáférés-modell
* Részletekről előtti integrációs véglegesítéséhez, és tekintse át [identitás ajánlott eljárások](/azure/security/azure-security-identity-management-best-practices).
  * Egyetlen identitás, egyszeri bejelentkezés (SSO) és a közvetlen egyszeri bejelentkezés engedélyezése
  * Konfigurálja a többtényezős hitelesítés (MFA) a rendszergazdák számára
  * Konszolidálhatja vagy integrálhatja az Identitásszolgáltatók, szükség esetén
  * Megvalósításához szükséges központosíthatja az identitások felügyeleti eszközök
  * – Igény (szerinti JIT) hozzáférési és a szerepkör megváltoztatása riasztás engedélyezése
  * A fő rendszergazdai tevékenységek beépített szerepkörök hozzárendelése kockázatelemzés magatartási
  * Fontolja meg egy erősebb hitelesítést minden felhasználó számára a frissített bevezetését
  * Privileged Identity Baseline (PIM) engedélyezéséhez a JIT (időben korlátozott aktiválás használata) további felügyeleti szerepkörök
  * Globális rendszergazda fiókok (Ellenőrizze, hogy véletlenül nem-e e-mailek nyissa meg a rendszergazdák, vagy futtathatja a programokat, a globális rendszergazdai fiókokhoz társított) külön felhasználói fiókok

## <a name="adopt-and-migrate"></a>Elfogadja és áttelepítése

Áttelepítés az egy növekményes, amely a adatátviteli, a tesztelés és a bevezetési az alkalmazások és számítási feladatok egy meglévő digitális hagyatéki összpontosít.

**Minimális javasolt tevékenységek:**

* Áttelepíteni a [identitás eszközlánc](toolchain.md) a fejlesztéstől az éles környezetbe.
* Az architektúra-útmutató a dokumentum frissítése, és eljuttatja a kulcsfontosságú érintettekkel.
* Oktatási anyagok és a dokumentáció, tudatosság kommunikáció, ösztönzők, és egyéb programok meghajtó felhasználói bevezetésére segítségével fejleszthet.

**Lehetséges-tevékenységek:**

* Ellenőrizze, hogy az ajánlott eljárásokat az összeállítás során meghatározott / a rendszer megfelelően végrehajtja a központi telepítés előtti fázisait.
* Ellenőrizze és/vagy finomíthatja a [hibrid identitás stratégia](../../decision-guides/identity/overview.md).
* Győződjön meg arról, hogy egyes alkalmazások vagy munkaterhelések futnak továbbra is, amelyek a kiadás előtt identitás stratégia összhangban vannak-e.
* Ellenőrizze, hogy egyszeri bejelentkezés (SSO), és közvetlen egyszeri bejelentkezés a várt módon működik az alkalmazások számára.
* Csökkentheti vagy kiküszöbölheti a másodlagos identitástárolók, amikor csak lehetséges száma.
* Minden olyan alkalmazáson belüli vagy adatbázis-identitástárolók szükség ellenőrzését. Egy megfelelő identitásszolgáltató (belső vagy külső) kívül eső identitások kockázati hozhat létre, az alkalmazás és a felhasználó számára.
* Feltételes hozzáférés engedélyezése [helyszíni összevont alkalmazások](/azure/active-directory/active-directory-device-registration-on-premises-setup).
* Identitás elosztása a régiók közötti szinkronizálás több hubs globális szinten.
* Központi szerepköralapú hozzáférés-vezérlés (RBAC) összevonás létesítéséhez.

## <a name="operate-and-post-implementation"></a>Üzemeltetése és a megvalósítás utáni

Az átalakítás befejeződése után cégirányítási műveleteket kell élő és egy alkalmazás vagy munkaterhelés természetes élettartama. Ebben a fázisban cégirányítási azt tárgyalja a tevékenységek által gyakran után a megoldás bevezetése, és az átalakítási ciklus kerülési kezdődik.

**Minimális javasolt tevékenységek:**

* Testre szabhatja a [identitásra építve eszközlánc](toolchain.md) a szervezet változó identitás igények szerint végbement változások alapján.
* Automatizálhatja az értesítések és jelentések értesítenek a potenciális rosszindulatú fenyegetések.
* A figyelő és a jelentést a rendszer az Alkalmazáshasználat és bevezetési folyamat állapotát.
* Üzembe helyezés utáni metrikák jelentés és az érdekelt felek terjesztése.
* Az útmutató későbbi bevezetési folyamatok architektúra irányelveket pontosításához.
* Kommunikálni, és folyamatosan oktatása az érintett csapatok rendszeres időközönként a folyamatban lévő architektúra irányelvek betartása érdekében.

**Lehetséges-tevékenységek:**

* Az identitás szabályok és eljárások betartása rendszeres ellenőrzéseket végez.
* Vizsgálati rosszindulatú és adatok rendszeresen, különösen a személyazonossági csalások, például a lehetséges rendszergazdai fiók átvétel kapcsolatos.
* Egy figyelési és jelentéskészítési eszköz konfigurálása.
* Vegye figyelembe, hogy jobban integrálása a biztonsági és -csalás-megelőző rendszerek.
* Rendszeresen tekintse át az emelt szintű felhasználók vagy szerepkörök hozzáférési jogosultságokat.
  * Minden felhasználó, aki rendszergazdai jogosultság aktiválására jogosulttá azonosításához.
* Tekintse át az előkészítési kivezetésével járó és hitelesítő adat folyamatait.
* Egyre nagyobb szintű automatizálási és identitás hozzáférés-kezelés (IAM) moduljai közötti kommunikációt, és vizsgálja meg.
* Vegye fontolóra egy fejlesztési biztonsági műveletek (DevSecOps) módszere.
* A mérőműszer-a költségek, a biztonsági és a felhasználói bevezetésére eredmények egy hatáselemzés végez.
* Rendszeres időközönként, hogy a rendszer által létrehozott mérőszámok változásait jeleníti meg hatás jelentést készít, és kiszámíthatja, üzleti hatásai a [hibrid identitás stratégia](../../decision-guides/identity/overview.md).
* Integrált figyelés által javasolt létrehozni a [az Azure Security Center](/azure/security-center/security-center-intro).

## <a name="next-steps"></a>További lépések

Most, hogy megismerkedett a felhőalapú identitáskezelést fogalmát, vizsgálja meg a [identitásra építve eszközlánc](toolchain.md) azonosításához az Azure-eszközök és funkciók, amelyek az identitásra építve cégirányítási tantárgy fejlesztése során lesz szüksége a Az Azure platformon.

> [!div class="nextstepaction"]
> [Identitás alapkonfiguráció eszközláncban az Azure-hoz](toolchain.md)