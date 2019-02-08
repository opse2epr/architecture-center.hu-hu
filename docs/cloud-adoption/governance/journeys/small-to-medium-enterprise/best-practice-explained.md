---
title: 'CAF: Kis és közepes méretű vállalat – a cégirányítási MVP kapcsolatos további technikai részleteket'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: MAGYARÁZAT kis és közepes méretű vállalat – a cégirányítási MVP kapcsolatos további technikai részleteket
author: BrianBlanchard
ms.openlocfilehash: e726213459c8bee63e3cc77ab54868fe7196b3ac
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899171"
---
# <a name="small-to-medium-enterprise-best-practice-explained"></a>Small-to-medium enterprise: Bevált gyakorlat ismertetése

A cégirányítási utazás elindítja a kezdeti vannak beállítva [vállalati szabályzatok](./initial-corporate-policy.md). Ezek a szabályzatok egy cégirányítási, amely tartalmazza az MVP létrehozására használt [ajánlott eljárások](./overview.md).

Ebben a cikkben bemutatjuk a magas szintű stratégia, hozzon létre egy cégirányítási MVP szükséges. A cégirányítási MVP középpontjában a [üzembe helyezési gyorsítás](../../deployment-acceleration/overview.md) fegyelem. Az eszközöket és a alkalmazni ezen a ponton minták lehetővé teszi a növekményes fejlesztések, bontsa ki a jövőben cégirányítási szükséges.

## <a name="governance-mvp-cloud-adoption-foundation"></a>Cégirányítási MVP (a felhő bevezetését Foundation)

Gyors bevezetésére vonatkozó cégirányítási és a vállalati házirend elérhető, Köszönjük néhány egyszerű alapelvek és felhőalapú cégirányítási azokat az eszközöket. Ezek a az első megközelítés bármilyen irányítási folyamat három felhőalapú Cégirányítási szakterületen. Minden egyes kibontásra váró ebben a cikkben.

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

A felhő Cégirányítási csapat felelős a következő döntések és megvalósítását. Számos más csapatok a bemenetek szükséges, de a felhő Cégirányítási csapat valószínű, hogy saját mind a döntést, és a megvalósítás. Az alábbiakban azoknak a használati esetekhez és döntés minden részletét a döntések.

### <a name="subscription-model"></a>Előfizetés-modell

A **alkalmazás kategóriához** minta Azure-előfizetés számára lett kiválasztva.

- Egy alkalmazás archetype módja hasonló igényekkel rendelkező csoport alkalmazásokhoz. Gyakori példák: Védett adatok, szabályozottabbá alkalmazások (például a HIPAA vagy a FedRAMP), alacsony kockázatú alkalmazások, helyszíni függőségek, a SAP vagy a más Nagyszámítógépek az Azure-beli alkalmazások vagy az alkalmazásokat, amelyek kiterjesztése a helyszíni SAP vagy Nagyszámítógépek alkalmazások. Ezek archetypes egyediek szervezetenként adatbesorolások és az üzleti hatékonyságnövelő alkalmazások típusai alapján. A digitális hagyatéki, függőségi leképezés segítheti az alkalmazás archetypes meghatározásakor a szervezeten belüli.
- Részlegek valószínűleg nem lehet szükség az aktuális fókuszt. Üzemelő példányok várt lehet behatárolja az egyetlen elszámolási egységet. A bevezetés szakaszban nem is lehet nagyvállalati szerződés számlázási központosítása. Valószínű, hogy ez a szint elfogadásának egyetlen használatalapú fizetéses Azure-előfizetés által van kezelve.
- A nagyvállalati szerződések portáljának használata vagy a nagyvállalati szerződés létezik-e, függetlenül a előfizetésen továbbra is lehet definiálni, és teljesítésével minimalizálása érdekében a rendszergazda csak a számlázási túli overheard.
- Az a **alkalmazás kategóriához** minta, előfizetések jön létre minden egyes alkalmazás archetype. Az egyes előfizetésekhez tartozik egy fiók környezetenként (fejlesztési, tesztelési és éles környezetben).
- Egy közös elnevezési konvenciót kell jóvá kell hagynia az előfizetés tervezése, az előző két pont alapján részeként.

### <a name="resource-consistency"></a>Erőforrás-konzisztencia

A **üzembe helyezési konzisztencia** minta egy erőforrás konzisztencia lett kiválasztva.

- Erőforráscsoportok jönnek létre az egyes alkalmazásokhoz. Felügyeleti csoportok minden alkalmazás archetype jön létre. Az Azure Policy kell alkalmazni az összes előfizetést a társított felügyeleti csoportból.
- A telepítési folyamat részeként az Azure-erőforrás konzisztencia sablonok, az erőforráscsoport verziókövetési rendszerben kell tárolni.
- Minden egyes erőforráscsoportot egy adott számítási feladatok vagy az alkalmazás társítva.
- Az Azure felügyeleti csoportok vállalati szabályzat kiforrottá cégirányítási frissítési tervek engedélyezze.
- Széles körű megvalósítása az Azure Policy túllépheti a csapat vállalt, és nem nyújthat az érték nagy fokú jelenleg. Azonban egy egyszerű alapértelmezett szabályzatot kell létrehozni és alkalmazni az aktuális felhő cégirányítási házirend-utasítások kis számú kényszerítéséhez minden egyes felügyeleti csoporthoz. Ez a szabályzat adott cégirányítási követelmények végrehajtására fogja meghatározni. Ezen megvalósításokhoz alkalmazható az összes üzembe helyezett eszközök között.

### <a name="resource-tagging"></a>Erőforrások címkézése

A **besorolási** címkézés, a minta egy modellt az erőforrás-címkézési lett kiválasztva.

- Üzembe helyezett eszközök kell címkézni a következő értékeket: Adatok besorolása, kritikusság, SLA-t és környezetben.
- Ezeket az értékeket négy cégirányítási műveleteket és biztonsági döntések vezérlik.
- Üzleti egység vagy csoportos belül egy nagyobb méretű vállalat végrehajtása alatt cégirányítási tartson a folyamatban, címkézést is tartalmaznia kell metaadatok a számlázási egység.

### <a name="logging-and-reporting"></a>Naplózás és jelentéskészítés

Ezen a ponton egy **Felhőbeli natív** naplózás és jelentéskészítés a minta javasolt, de nem szükséges a fejlesztői csapatnak sem.

- A cégirányítási követelmények nem kell gyűjteni a naplózás vagy jelentéskészítési célból vonatkozó az adatok állítani.
- További elemzést a védett adatokat vagy alapvető fontosságú számítási feladatokhoz kibocsátása előtt szükség lesz.

## <a name="evolution-of-governance-processes"></a>A cégirányítási folyamatok a fejlődést szem előtt tartva

Cégirányítási fejlődésének egyes házirend-utasítások nem, vagy nem határozza meg a automatizált azokat az eszközöket. Egyéb házirendek eredményez erőfeszítés az IT-biztonsági csoportja és a helyszíni identitás-kezelési csapatunk idővel. Információforrásait a új kockázatok csökkentése érdekében a Cloud Cégirányítási csapat az alábbi eljárások fog felügyeli.

**Bevezetési gyorsítás**: A felhő Cégirányítási csapat üzembehelyezési szkriptek több csapatok rendelkezik lett áttekintése. Tartják a parancsprogramokat, amelyek a központi telepítési sablonok szolgálhat egy készletét. Ezeket a sablonokat a felhőre való áttérés és a fejlesztési és üzemeltetési csapatok által használt gyorsabban adhatja meg a központi telepítéseit. Ezeket a szkripteket mindegyike tartalmaz a cégirányítási szabályzatokat, nincs további erőfeszítéssel cloud bevezetési mérnökeitől számos kikényszeríteni a szükséges követelményeknek. Ezek a szkriptek a curators, mint a Felhőbeli Cégirányítási csapata gyorsabban valósíthat meg az irányelvek változásai. A felhő Cégirányítási csapat eredményeként parancsfájl válogatott, bevezetési gyorsítás forrásaként látható. Ez többek között a központi telepítések, konzisztencia szigorúan a megfelelést kényszerítése nélkül hoz létre.

**Képzési mérnök**: A felhő Cégirányítási csapat hozott létre a két videó mérnökök számára, és a bi-havi képzés kínál. Ezen anyagok gyorsan megismerheti a cégirányítási kulturális környezet és a dolog hogyan végzett központi telepítések során mérnökei segítenek. A csapat léptet képzési eszközök azt mutatják be, éles környezetben, és nem éles üzemelő példányok közötti különbség, hogy a mérnökökkel tisztában van az új házirendek hogyan befolyásolják bevezetését. Ez többek között a központi telepítések, konzisztencia szigorúan a megfelelést kényszerítése nélkül hoz létre.

**Üzembe helyezés megtervezése**: Üzembe helyezése bármilyen eszköz tartalmazó védett adatokat, mielőtt a felhő Cégirányítási csapat áttekinti az üzembehelyezési szkriptek ellenőrzése cégirányítási igazítását. Meglévő csoportokat, korábban jóváhagyott központi telepítések naplózva lesz szoftveres eszközök használatával.

**Havi naplózási és jelentéskészítési**: Havonta, a felhő Cégirányítási csapat futtatja a naplózási házirend folyamatos igazítás ellenőrzése központi telepítések cloud. Eltérések észlelésekor dokumentált, és a felhő bevezetésének csapatok megosztva. Amikor a végrehajtás nem fennáll a veszélye, egy üzleti megszakítás és adatvesztés, a házirendek automatikusan érvényben vannak. Az ellenőrzés végén a felhő Cégirányítási csapat lefordítja az Felhőstratégia és minden felhő bevezetését való kommunikációhoz házirend általános betartása a jelentést. A jelentés is tárolja a naplózási és jogi célú.

**Negyedéves házirend áttekintése**: Negyedévente, a felhő Cégirányítási és Felhőstratégia tekintse át a vizsgálati eredmények, és vállalati szabályzat módosítási javaslatokat tehet. Ezek a javaslatok számos folyamatos fejlesztéseket eredményét és a használati minták megfigyelésére. Hagyja jóvá a módosításokat integrálva vannak az ezt követő naplózási ciklus során eszköztámogatás cégirányítási házirend.

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

Ha ez az útmutató van megvalósítva, minden egyes cloud bevezetési csapat meg oda-eredményes cégirányítási épül. A felhő Cégirányítási csapata folyamatosan frissíteni a vállalati szabályzatok és cégirányítási oktatnak párhuzamosan fog működni.

A két csapatok a tolerancia mutatók segítségével azonosíthatja a következő fejlődést szem előtt tartva továbbra is támogatja a felhőre való áttérés szükséges. Tartson a folyamatban lévő fiktív vállalat a következő lépés a biztonsági alapkonfiguráció támogatja a védett adatokról a felhőbe való áthelyezés folyamatosan fejlődik.

> [!div class="nextstepaction"]
> [Biztonsági alapkonfiguráció fejlődést szem előtt tartva](./security-baseline-evolution.md)