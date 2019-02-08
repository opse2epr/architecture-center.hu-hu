---
title: 'CAF: Erőforrás konzisztencia fegyelem fokozása'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Erőforrás konzisztencia fegyelem fokozása
author: BrianBlanchard
ms.openlocfilehash: bc81b894d46266c52291c53dba5532ab2ab6b860
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899067"
---
# <a name="resource-consistency-discipline-improvement"></a>Erőforrás konzisztencia fegyelem fokozása

Az erőforrás konzisztencia fegyelem módját a működési környezetben, alkalmazás vagy munkaterhelés kezelésével kapcsolatos házirendek összpontosít. Felhőalapú irányítási öt állapítják belül erőforrás konzisztencia tartalmaz, alkalmazások, számítási feladatok és az eszközintelligencia teljesítményének figyelését. A skálázási igények figyelembevételével készült, teljesítménytúllépéseknek szolgáltatói szerződés (SLA), és kerülje el proaktív módon SLA-sértésekhez automatikus szervizelés útján szükséges feladatok is tartalmaz.

Ez a cikk ismerteti a vállalat jobban fejlesztéséhez és erőforrások konzisztencia tantárgy részletes is végezhetnek néhány lehetséges feladatot. Ezeket a feladatokat is kell bontani tervezése, létrehozása, bevezetése és a egy felhőalapú megoldás megvalósítása fázisai működik, amely vannak majd iterálni fejlesztését lehetővé teszi a egy [növekményes megközelítés a felhő cégirányítási](../journeys/overview.md#an-incremental-approach-to-cloud-governance).

![Bevezetési négy fázisa](../../_images/adoption-phases.png)

*1. ábra A növekményes megközelítés a felhő cégirányítási bevezetési fázisait.*

Nem lehet bármely egy dokumentum áthidalhatók az összes üzleti követelményeinek. Ezért ez a cikk ismerteti javasolt minimális és a lehetséges példa tevékenységek a cégirányítási érlelését egyes fázisaiban. Ezek a tevékenységek kezdeti célja segíteni az egy [házirend MVP](../journeys/overview.md#an-incremental-approach-to-cloud-governance) , valamint olyan keretrendszer, a növekményes házirend fejlődést szem előtt tartva. A felhő Cégirányítási csapat kell mennyi Döntse el, hogy be ezeket a tevékenységeket, az erőforrás konzisztencia irányítási lehetőségek javítása érdekében.

> [!CAUTION]
> Sem a minimális vagy potenciális tevékenységek ebben a cikkben leírt adott vállalati szabályzatok vagy harmadik féltől származó megfelelőségi követelmények vannak igazítva. Ez az útmutató célja a beszélgetések, amely mindkét felhőalapú cégirányítási modell követelményekhez igazítását vezetnek megkönnyítése érdekében.

## <a name="planning-and-readiness"></a>Tervezés és a készültségi

Ebben a fázisban cégirányítási azt áthidalja a üzleti eredmények és gyakorlatban hasznosítható stratégiák közötti szakadékot. A folyamat során a a vezetőségi határozza meg az adott mérőszámok, követhesse a digitális hagyatéki vannak leképezve, és megkezdi a teljes áttelepítés sikeresebbé megtervezése.

**Minimális javasolt tevékenységek:**

* Kiértékelheti a [erőforrás konzisztencia eszközlánc](toolchain.md) beállítások.
* Megértette a licencelési követelményeket a stratégia számára.
* Fejlesztése a draft, architektúra-útmutató a dokumentum, és eljuttatja a kulcsfontosságú érintettekkel.
* Ismerkedjen az erőforrás-kezelő használatával üzembe helyezése, kezelése és monitorozása a rendszer egy csoportként megoldás összes erőforrását.
* Ismertetni, és a csapatok architektúra irányelvek fejlesztésének által érintett felhasználók között.
* Rangsorolt erőforrás üzembe helyezési feladatok hozzáadása a migrálás mappájában várakozó fájlok számát.

**Lehetséges-tevékenységek:**

* Az üzleti résztvevőkkel és/vagy a felhőbeli stratégia csapat kívánt felhő számlázási módszert és költségszámítási eljárások az üzleti egységek és a teljes szervezet belül működik.
* Adja meg a [figyelése és a szabályzatok kényszerítése](compliance-processes.md) követelményeinek.
* Vizsgálja meg az üzleti értéket és csökkentési házirend és SLA követelményeinek meghatározása érdekében szolgáltatáskimaradás költsége.
* Határozza meg, hogy helyezünk üzembe egy [egyszerű számítási feladat](./governance-simple-workload.md) vagy [több csapat](./governance-multiple-teams.md) adatirányítási stratégia az erőforrások.
* A tervezett számítási feladatokhoz a skálázhatósági követelmények meghatározása.

## <a name="build-and-pre-deployment"></a>A build és a központi telepítés előtti

Technikai és felhasználóknak előfeltételnek szükségesek a sikeres környezet migrálása. Ez a folyamat a döntéseket hozhat, készültségi és alapvető infrastruktúra, amely az áttelepítés onnantól összpontosít.

**Minimális javasolt tevékenységek:**

* Alkalmazzon a [erőforrás konzisztencia eszközlánc](toolchain.md) szerint jelennek meg a központi telepítés előtti fázisban.
* Az architektúra-útmutató a dokumentum frissítése, és eljuttatja a kulcsfontosságú érintettekkel.
* Erőforrás üzembe helyezési feladatokat alkalmazhat a rangsorolt áttelepítési mappájában várakozó fájlok számát.
* Oktatási anyagok és a dokumentáció, tudatosság kommunikáció, ösztönzők, és egyéb programok meghajtó felhasználói bevezetésére segítségével fejleszthet.

**Lehetséges-tevékenységek:**

* Döntse el, az egy [előfizetés tervezési stratégiát](../../decision-guides/subscriptions/overview.md), kiválasztása, az előfizetés mintáit, amely legjobban az igényeinek szervezet és a számítási feladatok.
* Használja a [erőforrás konzisztencia](../../decision-guides/resource-consistency/overview.md) stratégia idővel architektúra irányelvek kényszerítésére.
* Alkalmazzon [erőforrás-kiosztási és a szabványoknak](../../decision-guides/resource-tagging/overview.md) számviteli és a szervezeti igényeinek megfelelő az erőforrások.
* Időponthoz proaktív cégirányítási létrehozásához használja a központi telepítési sablonok és az automation közös konfigurációkat és a egy egységes csoportosítást struktúra kényszerítéséhez, erőforrások és -erőforráscsoportok üzembe helyezésekor.
* Létesítsen egy legkisebb jogosultság engedélyek modell, ahol felhasználók rendelkeznek alapértelmezés szerint nem rendelkezik engedélyekkel.
* Határozza meg, aki a szervezet tulajdonában lévő minden egyes számítási feladatok és a fiók, és ki kell fenntartani, vagy módosíthatja ezeket az erőforrásokat. Adja meg a felhő-szerepkörökről és feladatokról, ezek igényeinek megfelelően, és ezek a szerepkörök alapjaként használni a hozzáférés-vezérlést.
* Adja meg az erőforrások közti függőségeket.
* A terv szakaszában meghatározott követelményeknek megfelelő méretezés automatizált erőforrás megvalósításához.
* Végezzen hozzáférési teljesítmény mérésére igénybe vett szolgáltatások minőségét.
* Érdemes megfontolni a [házirend](/azure/governance/policy/overview) SLA kényszerítési beállításait és erőforrás-létrehozási szabályok használata kezelheti.

## <a name="adopt-and-migrate"></a>Elfogadja és áttelepítése

Áttelepítés az egy növekményes, amely a adatátviteli, a tesztelés és a bevezetési az alkalmazások és számítási feladatok egy meglévő digitális hagyatéki összpontosít.

**Minimális javasolt tevékenységek:**

* Áttelepíteni a [erőforrás konzisztencia eszközlánc](toolchain.md) éles üzem előtti telepítéséből.
* Az architektúra-útmutató a dokumentum frissítése, és eljuttatja a kulcsfontosságú érintettekkel.
* Oktatási anyagok és a dokumentáció, tudatosság kommunikáció, ösztönzők, és egyéb programok meghajtó felhasználói bevezetésére segítségével fejleszthet.
* Bármely meglévő automatizált parancsfájlok áttelepítése vagy eszközöket, támogatja a megadott SLA követelményeinek.

**Lehetséges-tevékenységek:**

* Végezze el, és tesztelje a figyelési és jelentéskészítési adatok. a kiválasztott helyszíni, felhőbeli-átjáróval vagy hibrid megoldás.
* Határozza meg, ha módosításokat kell végezni az erőforrások SLA-t vagy a felügyeleti házirend.
* Operatív feladatok javíthatja a lekérdezési képességek hatékonyan keresése az erőforrás a felhő szűrőként való alkalmazásával.
* A változó üzleti igények és cégirányítási követelmények erőforrások igazítása
* Győződjön meg arról, hogy a virtuális gépek, virtuális hálózatok és tárfiókok tükrözik a tényleges erőforrás-hozzáférési igényei során minden kiadással, és szükség szerint módosíthatja.
* Ellenőrizze, hogy az erőforrások automatikus méretezése megfelel-e a hozzáférési követelmények.
* Tekintse át az erőforrások, erőforráscsoportok és az Azure-előfizetések a felhasználói hozzáférést, és szükség szerint módosítsa a hozzáférés-vezérlést.
* A figyelő változásai erőforrás csomagok eléréséhez, és ellenőrizheti az érintettekkel való, hogy további bejelentkezési kompromisszumot van szükség.
* Frissítse a módosításokat az architektúra-útmutató a dokumentumot, tükrözik a tényleges költségek.
* Határozza meg, hogy a szervezet szükség van-e a P & Ls szülővezérlők pénzügyi igazítását az üzleti egységek.
* Globális szervezetek számára a szolgáltatásiszint-szerződés megfelelőségi vagy a szuverenitásra vonatkozóan követelmények megvalósítása.
* Felhőalapú összesítés, a megoldás üzembe helyezése átjáró egy felhőszolgáltatónál.
* Olyan eszközök, amelyek nem teszi lehetővé a hibrid vagy átjáró beállításai szorosan összekapcsolhatja a egy működési figyelési eszközzel figyelése.

## <a name="operate-and-post-implementation"></a>Üzemeltetése és a megvalósítás utáni

Az átalakítás befejeződése után cégirányítási műveleteket kell élő és egy alkalmazás vagy munkaterhelés természetes élettartama. Ebben a fázisban cégirányítási azt tárgyalja a tevékenységek által gyakran után a megoldás bevezetése, és az átalakítási ciklus kerülési kezdődik.

**Minimális javasolt tevékenységek:**

* Testre szabhatja a [erőforrás konzisztencia eszközlánc](toolchain.md) változó igényeihez a szervezet a Cost Management frissítések alapján.
* Fontolja meg, hogy a tényleges erőforrás-használat bármely értesítések és jelentések automatizálása.
* Az útmutató későbbi bevezetési folyamatok architektúra irányelvek pontosításához.
* Tájékoztassa az érintett csoportok rendszeres időközönként a folyamatban lévő az architektúra irányelvek betartása érdekében.

**Lehetséges-tevékenységek:**

* Módosítsa a tényleges erőforrások változásainak negyedéves terveket.
* Automatikusan alkalmazza, és a cégirányítási követelmények érvényesítése során a későbbiekben.
* Használaton erőforrások értékelése, és határozza meg, érdemes Folytatás zajlik.
* Ők olyan eltéréseket és rendellenességeket között a tervezett és tényleges erőforrás-használat észlelése.
* Segítséget nyújt a felhőbeli bevezetési csoportok és a Felhőstratégia csapat megismeréséhez és a rendellenességek feloldása.
* Határozza meg, ha módosításokat kell bocsátani erőforrás konzisztencia a számlázási és szolgáltatói szerződésekkel.
* Értékelje ki a naplózást és figyelést eszközök meghatározni, hogy a helyszíni, felhőbeli átjáró vagy hibrid megoldás kell módosítani.
* Üzleti egységek és a földrajzilag elosztott csoportok esetében határozza meg, ha a szervezet fontolja meg további felhőalapú felügyeleti funkciókat (például [az Azure felügyeleti csoportok](/azure/governance/management-groups/)) jobban a alkalmazni központi szabályzatokon és SLA követelményeinek.

## <a name="next-steps"></a>További lépések

Most, hogy megismerkedett a felhőbeli erőforrás-szabályozás fogalmát, helyezze át további részleteket ismerhet meg [erőforrásokhoz való hozzáférés kezelésének módját](azure-resource-access.md) előkészítésekor megtudhatja, hogyan tervezhet egy cégirányítási modellt használ az Azure-ban egy [egyszerű számítási feladat ](governance-simple-workload.md) vagy [több csapat](governance-multiple-teams.md).

> [!div class="nextstepaction"]
> [További információ az Azure-beli erőforrás-hozzáférés](azure-resource-access.md)
> [további információ az Azure SLA-k](https://azure.microsoft.com/support/legal/sla/)
> [naplózási, jelentéskészítési és figyelésének ismertetése](../../decision-guides/log-and-report/overview.md)
