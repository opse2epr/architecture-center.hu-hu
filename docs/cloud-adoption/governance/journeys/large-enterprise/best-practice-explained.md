---
title: 'CAF: Nagyvállalati – ajánlott eljárás ismertetése'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Nagyvállalati – ajánlott eljárás ismertetése
author: BrianBlanchard
ms.openlocfilehash: db630b93cd9d5118a8fc29b424ef78e0c1d8848a
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298446"
---
# <a name="large-enterprise-best-practice-explained"></a>Nagyvállalati: Ajánlott eljárás magyarázata

A cégirányítási utazás elindítja a kezdeti vannak beállítva [vállalati szabályzatok](./initial-corporate-policy.md). Ezek a szabályzatok cégirányítási tükröző minimális működőképes termékek (MVP) létrehozásához használt [ajánlott eljárások](./overview.md).

Ebben a cikkben bemutatjuk a magas szintű stratégia, hozzon létre egy cégirányítási MVP szükséges. A cégirányítási MVP középpontjában a [üzembe helyezési gyorsítás](../../deployment-acceleration/overview.md) fegyelem. Az eszközöket és a alkalmazni ezen a ponton minták lehetővé teszi a növekményes fejlesztések, bontsa ki a jövőben cégirányítási szükséges.

## <a name="governance-mvp-cloud-adoption-foundation"></a>Cégirányítási MVP (a felhő bevezetését Foundation)

Gyors bevezetésére vonatkozó cégirányítási és a vállalati házirend elérhető, Köszönjük néhány egyszerű alapelvek és felhőalapú cégirányítási azokat az eszközöket. Ezek a az első megközelítés bármilyen irányítási folyamat három cégirányítási szakterületen. Minden egyes kibontásra váró ebben a cikkben.

Létesíteni a kiindulási pont, ez a cikk bemutatja, hogyan lehet a magas szintű stratégiákat identitásra építve, biztonsági Alapértékeken és üzembe helyezési gyorsítás mögött kell létrehoznia egy cégirányítási MVP, amely az összes bevezetési alapját erre a célra.

![Egy növekményes cégirányítási MVP – példa](../../../_images/governance/governance-mvp.png)

## <a name="implementation-process"></a>Implementálási folyamatáról

A cégirányítási MVP megvalósítása a identitás, biztonsági és hálózatkezelési függőségekkel rendelkezik. Ha elhárulnak a függőségeket, a felhő Cégirányítási csapat meghatározza cégirányítási néhány aspektusait. A felhő Cégirányítási csapat és a támogató csapatok a döntések kényszerítési eszközök egyetlen csomagban keresztül hajtja végre.

![Egy növekményes cégirányítási MVP – példa](../../../_images/governance/governance-mvp-implementation-flow.png)

Ez a megvalósítás használatával egy egyszerű feladatlista is ismerteti:

1. Core függőségek kapcsolatos döntéseket hozzájárulásával: Identitás, a hálózat és a titkosítást.
2. Határozza meg a vállalati szabályzat kényszerítése során használandó mintát.
3. Határozza meg a megfelelő cégirányítási minták az erőforrás konzisztencia, erőforrás-címkézési és Loging és jelentéskészítési szakterületen.
4. A cégirányítási eszközök illeszkedjenek a alkalmazni a függő döntéseket hozhat, és a szabályozási döntések a kiválasztott házirend kényszerítési minta megvalósításához.

[!INCLUDE [implementation-process](../../../../../includes/cloud-adoption/governance/implementation-process.md)]

## <a name="application-of-governance-defined-patterns"></a>Cégirányítási által meghatározott minták alkalmazása

A felhő Cégirányítási csapat felelőssége a következő döntések és megvalósítását. Számos más csapatokkal az bemenetei között meg lesz szükség, de a felhő Cégirányítási csapat valószínű, hogy saját a döntést és a megvalósítás. Az alábbiakban azoknak a használati esetekhez és döntés minden részletét a döntések.

### <a name="subscription-model"></a>Előfizetés-modell

A **Mixed** minta Azure-előfizetés számára lett kiválasztva.

- Az Azure-erőforrások új kérések merülnek fel, mint "Részleg" minden egyes jelentős üzleti egység minden működési földrajzi kell létrehozni. Belül minden egyes szervezeti egységek minden egyes alkalmazás archetype "Előfizetések" kell létrehozni.
- Egy alkalmazás archetype hasonló igényekkel rendelkező alkalmazások csoportosítását. Gyakori példák: Védett adatok, szabályozottabbá alkalmazások (például a HIPAA vagy a FedRAMP), alacsony kockázatú alkalmazások, helyszíni függőségek, a SAP vagy a más Nagyszámítógépek az Azure-beli alkalmazások vagy az alkalmazásokat, amelyek kiterjesztése a helyszíni SAP vagy Nagyszámítógépek alkalmazások. Minden egyes szervezet rendelkezik az adatbesorolásokat és az üzleti támogató alkalmazások típusai alapján egyedi igényeinek. A digitális hagyatéki, függőségi leképezés segítségével határozza meg az alkalmazás archetypes szervezetekben.
- Egy közös elnevezési konvenciót kell egyeztetett az előfizetések kialakítása, a fenti két felsorolás alapján részeként.

### <a name="resource-consistency"></a>Erőforrás-konzisztencia

**Hierarchikus konzisztencia** erőforrás konzisztencia mintaként van kiválasztva.

- Egyes alkalmazásokhoz tartozó erőforráscsoportok kell létrehozni. Minden alkalmazás archetype felügyeleti csoportot kell létrehozni. A társított felügyeleti csoportban lévő összes előfizetés az Azure Policy kell alkalmazni.
- A telepítési folyamat részeként a konzisztencia erőforrás-sablonok az összes eszköz verziókövetési rendszerben kell tárolni.
- Minden erőforráscsoport, egy adott számítási feladatok vagy az alkalmazás kell igazítani.
- Az Azure felügyeleti csoport hierarchia meghatározott felel a beágyazott csoportok alkalmazás tulajdonosa és számlázási feladata.
- Széles körű megvalósítása az Azure Policy túllépheti a csapat vállalt, és előfordulhat, hogy nem sok értéket ezen a ponton. Azonban egy egyszerű alapértelmezett szabályzatot kell létrehozni és egyes erőforráscsoportokhoz kényszerítése az első néhány felhőalapú cégirányítási házirend-utasítások a alkalmazni. Ez határozza meg az adott cégirányítási követelmények végrehajtására szolgál. Ezen megvalósításokhoz alkalmazható az összes üzembe helyezett eszközök között.

### <a name="resource-tagging"></a>Erőforrások címkézése

A **nyilvántartás** minta címkézésre erőforrás van kiválasztva.

- Üzembe helyezett eszközök kell címkézni a következő értékeket: Szervezeti egység/számlázási egység, földrajzi hely, az Adatbesorolás, kritikusság, SLA-t, környezet, alkalmazás Archetype, alkalmazások és alkalmazás tulajdonosa.
- Ezeket az értékeket az Azure felügyeleti csoport és a egy üzembe helyezett eszközhöz társított előfizetés együtt fog döntései cégirányítási műveleteket és biztonsági.

### <a name="logging-and-reporting"></a>Naplózás és jelentéskészítés

Ezen a ponton egy **hibrid** naplózási és jelentéskészítési mintájának javasolt, de nem szükséges a fejlesztői csapatnak sem.

- A cégirányítási követelmények nem jelenleg vannak beállítva az adott adatpontok kapcsolatban kell gyűjteni a naplózás vagy jelentéskészítési célból. A képzeletbeli narratíva jellemző, és a egy kizárási minta kell tekinteni. Naplózási szabványokat határozza meg, és érvényesíti a lehető leghamarabb kell.
- További elemzés szükséges előtt az összes védett adatok vagy az alapvető fontosságú számítási feladatokhoz.
- Előtt támogat a védett adatok vagy a kritikus fontosságú számítási feladatokat, a meglévő, helyszíni üzemeltetési figyelési megoldást a munkaterületen, a naplózáshoz használt hozzáférést kell biztosítani. Alkalmazások szükségesek felel meg a biztonsági és naplózási követelmények kapcsolódik a bérlőt, ha az alkalmazás egy meghatározott SLA-val is támogatja.

## <a name="evolution-of-governance-processes"></a>A cégirányítási folyamatok a fejlődést szem előtt tartva

A házirend-utasítások némelyike nem vagy automatizált azokat az eszközöket nem lehet szabályoz. Egyéb házirendek a számítástechnikai biztonsági és a helyszíni identitás-alapkonfiguráció csapatok rendszeres erőfeszítésekre van szükség. A felhő Cégirányítási csapat felügyeli a legutóbbi nyolc házirend-utasítások végrehajtásához a következő eljárásokat kell:

**Vállalati irányelvek változásai**: A felhő Cégirányítási csapat az MVP megtervezése és elfogadják az új szabályzatok cégirányítási fog módosításokat. A cégirányítási MVP értéke, hogy engedélyezi az automatikus annak érdekében, az új házirendeket.

**Bevezetési gyorsítás**: A felhő Cégirányítási csapat üzembehelyezési szkriptek több csapatok rendelkezik lett áttekintése. Ezek már karbantartása, melyektől a központi telepítési sablonok parancsfájlokkal. A felhő bevezetésének csapatok használhatják ezeket a sablonokat, és több fejlesztési és üzemeltetési csapat gyorsan adhatja meg a központi telepítéseit. Minden parancsprogramhoz tartalmazza a cégirányítási szabályzatok kényszerítése követelményei, és nincs szükség további erőfeszítésekre cloud bevezetési mérnökeitől. Ezek a szkriptek a curators, mint azok valósítható meg az irányelvek változásai időnél gyorsabban van szüksége. Ezenkívül néznek, megoldásgyorsítók elfogadása. Ez biztosítja, egységes központi telepítések szigorúan a megfelelést kényszerítése nélkül.

**Képzési mérnök**: A felhő Cégirányítási csapat hozott létre a két videó mérnökök számára, és a bi-havi képzés kínál. Mindkét erőforrás lekérése gyorsan cégirányítási kulturális környezetet, és hogyan történik a központi telepítések a mérnökök segítségével. A csapat a bővíti a betanítási eszközök használatával mutatja be az új szabályzatok hatása a felhasználókra bevezetési mérnökök segít éles környezetben, és nem éles üzemelő példányok közötti különbség. Ez biztosítja, egységes központi telepítések szigorúan a megfelelést kényszerítése nélkül.

**Üzembe helyezés megtervezése**: Bármely eszköz tartalmazó telepítése védett adatokat, mielőtt a felhő Cégirányítási csapat felelőssége üzembehelyezési szkriptek ellenőrzése cégirányítási igazítás áttekintése. Meglévő csoportokat, korábban jóváhagyott központi telepítések naplózva lesz szoftveres eszközök használatával.

**Havi naplózási és jelentéskészítési**: Havonta, a felhő Cégirányítási csapat futtatja a naplózási házirend folyamatos igazítás ellenőrzése központi telepítések cloud. Eltérések észlelésekor dokumentált, és a felhő bevezetésének csapatok megosztva. Amikor a végrehajtás nem fennáll a veszélye, egy üzleti megszakítás és adatvesztés, a házirendek automatikusan érvényben vannak. Az ellenőrzés végén a felhő Cégirányítási csapat lefordítja az Felhőstratégia és minden felhő bevezetését való kommunikációhoz házirend általános betartása a jelentést. A jelentés is tárolja a naplózási és jogi célú.

**Negyedéves házirend áttekintése**: Negyedévente, a felhő Cégirányítási és Felhőstratégia ellenőrzés eredményeinek áttekintése és a vállalati szabályzat módosítási javaslatokat tehet. Ezek a javaslatok számos folyamatos fejlesztéseket eredményét és a használati minták megfigyelésére. Hagyja jóvá a módosításokat integrálva vannak az ezt követő naplózási ciklus során eszköztámogatás cégirányítási házirend.

## <a name="alternative-patterns"></a>Alternatív minták

A cégirányítási tartson a folyamatban a kiválasztott minták bármelyike nem összhangba kerüljenek az olvasó a követelmények, ha minden minta alternatívái érhetők el:

- [Titkosítási minták](../../../decision-guides/encryption/overview.md)
- [Identitás-minták](../../../decision-guides/identity/overview.md)
- [Naplózás és jelentéskészítés minták](../../../decision-guides/log-and-report/overview.md)
- [A házirend-kényszerítési minták](../../../decision-guides/policy-enforcement/overview.md)
- [Erőforrás konzisztencia minták](../../../decision-guides/resource-consistency/overview.md)
- [Erőforrás címkézést minták](../../../decision-guides/resource-tagging/overview.md)
- [Szoftver szoftveralapú hálózati minták](../../../decision-guides/software-defined-network/overview.md)
- [Előfizetés a tervezési minták](../../../decision-guides/subscriptions/overview.md)

## <a name="next-steps"></a>További lépések

Ez az útmutató valósul meg, miután minden egyes cloud bevezetési csapat folytassa szilárd cégirányítási épül. A felhő Cégirányítási csapata folyamatosan frissítenie kell a vállalati szabályzatok és cégirányítási oktatnak párhuzamosan fog működni.

Mindkét csapatok a tolerancia mutatók segítségével azonosíthatja a következő fejlődést szem előtt tartva továbbra is támogatja a felhőre való áttérés szükséges. A következő lépés a vállalat tartson a folyamatban, hogy fejlesztése a csatlakoztathatóság érdekében a cégirányítási alapkonfiguráció örökölt vagy harmadik felek többtényezős hitelesítés (MFA) követelményekkel rendelkező alkalmazások támogatásához.

> [!div class="nextstepaction"]
> [Identitás alapkonfiguráció fejlődést szem előtt tartva](./identity-baseline-evolution.md)
