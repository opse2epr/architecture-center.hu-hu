---
title: 'CAF: Előfizetés döntési útmutató'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: További információ a cloud platform-előfizetésekhez Azure áttelepítések core szolgáltatás.
author: rotycenh
ms.openlocfilehash: c0781f6af25150d359395b1b80506dd0cfee8e3c
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899103"
---
# <a name="subscription-decision-guide"></a>Előfizetés döntési útmutató

Minden felhőalapú platform alapvető tulajdonjog modell, amely révén a szervezetek számos erőforrás és számlázási lehetőségek alapulnak. A struktúra, amely Azure eltér a más felhőszolgáltatók, mert tartalmazza a szervezeti hierarchia különböző támogatási lehetőségeket és előfizetés tulajdonjogának csoportosítva. Függetlenül attól nincs általában egy egyéni felelős a számlázási és a egy másik, akinek ki lett osztva a legfelső szintű tulajdonosaként-erőforrások kezeléséhez.

![Küldik az ábrázolást előfizetésekkel legalább a legösszetettebb, hogy az alábbi hivatkozások a jump igazítva](../../_images/discovery-guides/discovery-guide-subscriptions.png)

Ugrás ide: [Előfizetések kialakítása és az Azure Enterprise-megállapodások](#subscriptions-design-and-azure-enterprise-agreements) | [előfizetés tervezési minták](#subscription-design-patterns) | [felügyeleti csoportok](#management-groups)  |  [Szervezet az előfizetés szintjén](#organization-at-the-subscription-level)

Előfizetések kialakítása a vállalat létrehozni a struktúrát, vagy eszközök rendszerezése a felhőre való áttérés során használja a leggyakrabban használt stratégiákat egyike.

**Előfizetés hierarchia**: A *előfizetés* olyan logikai gyűjtemény, az Azure-szolgáltatások (például virtuális gépek, a SQL DB, az App Services vagy a tárolók). Az Azure-ban minden egyes eszköz egyetlen előfizetéshez van telepítve. Az egyes előfizetésekhez majd tulajdonában van egy *fiók*. Ezt a fiókot egy felhasználói fiókot (vagy lehetőleg egy szolgáltatásfiók), amely a számlázási és a rendszergazdai hozzáférést biztosít egy előfizetésből. Azok a vásárlóknak, akik rendelkeznek adatközpontjainak szén-használata az Azure segítségével egy nagyvállalati szerződés (EA) meghatározott mennyiségű, egy másik szinten a nevű egy *részleg* kerül. A nagyvállalati szerződés portál, előfizetési, fiókok és szervezeti egységek használható egy hierarchia, felügyeleti és számlázási célból.

Előfizetés tervek összetettsége változik. Kapcsolatos stratégia tervezési döntéseket, azok általában olyan üzleti és informatikai megkötések is egyedi nincsenek pontok vannak. Technikai döntéseket hozhat, IT-tervezők és döntési elvégzése előtt a döntéshozók működnie kell az üzleti résztvevőkkel és a felhő stratégia csapat a kívánt felhő számlázási megközelítés, belül az üzleti egység, és a globális költségelszámolási eljárások megismeréséhez piaci a szervezet igényeinek.

**Kihasználás**: A fenti képen a szaggatott vonal hivatkozik-kihasználás is egyszerű és bonyolultabb előfizetés tervezési minták között. További technikai döntési digitális hagyatéki mérete és az Azure-előfizetés korlátai, elkülönítési és elkülönítése házirendek és operatív IT-részlegek általában alapján pontokat előfizetések kialakítása jelentős hatással lehet.

**Egyéb szempontok**: A rendszer egy lényeges tudnivaló, hogy egy előfizetés tervezési kiválasztásakor vegye figyelembe, hogy nem csoportosíthatók az erőforrások vagy központi telepítései csak úgy előfizetések. Előfizetések létrejöttek az Azure-ban, így korábbi Azure-megoldások például Azure Service Managerrel kapcsolatos korlátozásokkal rendelkeznek szolgálja.

Üzembe helyezési struktúrája, az automatizálást és új megközelítést kíván erőforrások csoportosítása hatással lehet a struktúra előfizetések kialakítása. Fontolja meg egy előfizetés tervezési többszöri hogyan [erőforrás konzisztencia](../resource-consistency/overview.md) döntések befolyásolhatja a tervezési döntések. Ha például egy nagy, multinacionális szervezet kezdetben érdemes lehet egy bonyolult mintával az előfizetés-kezelési. Azonban, hogy ugyanazzal a vállalati előfordulhat, hogy valósíthat meg nagyobb előnyei és üzleti egység mintát egyszerűbb felügyeleti csoport hierarchia hozzáadásával.

## <a name="subscriptions-design-and-azure-enterprise-agreements"></a>Előfizetések kialakítása és az Azure nagyvállalati szerződések

Minden Azure-előfizetés társítva egy fiókot, amely minden egyes előfizetés számlázási és a legfelső szintű hozzáférés-vezérlés csatlakozik. Egy olyan fiók több előfizetéssel rendelkezhet, és előfizetések szervezet alapszinten tud biztosítani.

Kis Azure üzemelő példánya esetében az előfizetések kis gyűjteménye vagy egy előfizetés előfordulhat, hogy compose a teljes felhőalapú szűrőként. Azonban nagy valószínűséggel az Azure környezetekben span az támogatja a szervezet felépítésének és a bypass több előfizetést kell [előfizetési kvóták és korlátozások](/azure/azure-subscription-service-limits).

Minden egyes Azure nagyvállalati szerződéssel lehetőséget kínál a további előfizetések és fiókok szervezheti, amelyek tükrözik a szervezeti felhasználók prioritásaik hierarchiák. A szervezeti nagyvállalati beléptetés határozza meg, a alakzat és a egy szerződéses szempontból vállalaton belüli Azure-szolgáltatások használatát. Belül minden egyes nagyvállalati szerződés további feloszthatja a részlegek, fiókok és előfizetések a szervezet struktúrája megfelelő környezetét.

![hierarchia](../../_images/infra-subscriptions/agreement.png)

## <a name="subscription-design-patterns"></a>Előfizetés a tervezési minták

Minden vállalati eltér. Ezért a részleg/fiók /-előfizetéséből hierarchiában engedélyezve van a teljes Azure nagyvállalati szerződés lehetővé teszi, hogy jelentős rugalmasságot nyújt, az Azure hogyan vannak rendszerezve. A számlázás, erőforrás-kezelést és erőforrás-hozzáférés a vállalat igényeinek megfelelően a szervezet hierarchiáján modellezési-e a első és legfontosabb, döntés, amely a nyilvános felhőben indítása során.

A következő előfizetés minták általános növekedése tükrözik előfizetés tervezési összetettebbé válnak lehetséges szervezeti prioritások támogatásához:

### <a name="single-subscription"></a>Egy előfizetés

Olyan szervezeteknek, amelyek kell üzembe helyeznie a felhőben üzemeltetett eszközök kis számú elegendő fiókonként egyetlen előfizetéssel. Ez gyakran az az első előfizetés minta megvalósítása során kezdve a felhő bevezetésének folyamatát, így semmibe kísérleti vagy megvalósíthatósági példában egy felhőalapú platform funkcióinak megismerése fogalmat telepítések.

Azonban lehetnek olyan technikai korlátozását, egyetlen előfizetéssel támogató erőforrások száma. A felhő hagyatéki méretének növekedésével valószínűleg érdemes is támogatja a jobb rendszerezése szabályzatok, és hozzáférés-vezérlés oly módon, egyetlen előfizetéssel nem támogatott az erőforrások rendszerezéséhez.

### <a name="application-category-pattern"></a>Kategória alkalmazásminta

A szervezet felhőbeli tárhely méretének növekedésével egyre nagyobb valószínűséggel válik több előfizetés használata. Ebben a forgatókönyvben az előfizetések általában jönnek létre az alkalmazásokat, az üzleti kritikusság, a megfelelőségi követelmények, a hozzáférés-vezérlés vagy az adatvédelmi igényeket alapvető különbség támogatásához. Az előfizetések és -fiókok ezek Alkalmazáskategóriák támogató összes szerveződnek egy egyetlen osztály birtokolt és felügyelt központi informatikai munkatársak alatt.

Minden egyes szervezet alkalmazások eltérően, kategorizálásának fog választani, gyakran elválasztó alapján meghatározott alkalmazások vagy szolgáltatások vagy alkalmazások archetypes a témakörgyűjtemény előfizetések. Előfordulhat, hogy adja meg ezt a mintát egy külön előfizetés számítási feladatok a következők:

- Kísérleti vagy alacsony kockázatú alkalmazások
- Védett adatok rendelkező alkalmazások
- Alapvető fontosságú számítási feladatokhoz
- Alkalmazások (például a HIPAA vagy a FedRAMP) szabályozási követelmények teljesítése
- Batch számítási feladatot
- Például a Hadoop big data számítási feladatok
- Tárolóalapú számítási feladatokra, például a Kubernetes üzembe helyezési vezénylők alkalmazásával
- Elemzési számítási feladatok

Ez a minta több fiókok adott munkaterhelés konkrét felelős tulajdonosok támogatja. Gyakorta nem áll rendelkezésre a szervezeti egység szintjén a nagyvállalati szerződés hierarchia összetettebb struktúra, mivel ez a minta nem igényel Azure nagyvállalati szerződés megvalósítása.

![Kategória alkalmazásminta](../../_images/infra-subscriptions/application.png)

### <a name="functional-pattern"></a>Funkcionális minta

Ez a minta rendezi az előfizetések és fiókok működési mentén, például a Pénzügy, az értékesítés és az informatikai támogatás az Azure nagyvállalati szerződéssel rendelkező ügyfeleink számára biztosított vállalati/szervezeti egység/fiók vagy előfizetés-hierarchia használatával.

![Funkcionális minta](../../_images/infra-subscriptions/functional.png)

### <a name="business-unit-pattern"></a>Üzleti egység minta

Ez a minta előfizetések és fiókok eredmény kategória, üzleti egység, osztás, nyereség center vagy az Azure nagyvállalati szerződés hierarchia használatával hasonló struktúra alapján csoportosítja.

![Üzleti egység minta](../../_images/infra-subscriptions/business.png)

### <a name="geographic-pattern"></a>Földrajzi minta

A szervezet számára a globális műveleteket ez a minta előfizetések és fiókok használata az Azure nagyvállalati szerződéssel hierarchia földrajzi régiók alapján csoportosítja.

![Földrajzi minta](../../_images/infra-subscriptions/geographic.png)

### <a name="mixed-patterns"></a>Vegyes minták

Enterprise részleg/fiók/előfizetések hierarchia. Azonban minták, mint például a földrajzi régióban, és a részleg összetettebb megfelelően kombinálhatja számlázási és szervezeti struktúrák a vállalaton belül. Emellett a [erőforrás konzisztencia tervezési](../resource-consistency/overview.md) kiterjeszthető a cégirányítási és az előfizetések kialakítása szervezeti struktúráját.

Felügyeleti csoportok, a következő szakaszban leírt módon segíthet bonyolultabb szervezeti felépítés támogatja.

Felügyeleti csoportok a következő szakaszban tárgyalt segíthet bonyolultabb szervezeti felépítés támogatja.

## <a name="management-groups"></a>Felügyeleti csoportok

A részleg és a szervezet struktúra keresztül Enterprise-megállapodások mellett [az Azure felügyeleti csoportok](/azure/governance/management-groups/index) további rugalmasságot a több házirendet, a hozzáférés-vezérlés és a megfelelőségi rendszerezéséhez az előfizetések. Lehetővé teszi, hogy ne a számlázási hierarchiában a hierarchia létrehozása a felügyeleti csoportok ágyazhatók legfeljebb hat szintje. Ez lehet, kizárólag az erőforrások hatékony kezelését.

Felügyeleti csoportok is tükrözi a számlázási hierarchiában, és gyakran a vállalatok így indítsa el. Azonban a felügyeleti csoportok a power akkor, ha azokat használja a szervezet modell ahol kapcsolódó előfizetések &mdash; függetlenül attól, hogy hol vannak a számlázási hierarchia &mdash; vannak csoportosítva, és a közös szerepkörrel kell szabályzatok és kezdeményezések.

Példák erre vonatkozóan:

- Production/non-production: Egyes vállalatok hozzon létre felügyeleti csoportot, és azok éles környezetben, és nem éles üzemű előfizetéseket. Felügyeleti csoportok lehetővé teszik, hogy ezeket az ügyfeleket több könnyedén kezelheti a szerepköröket és szabályzatok, például: nem éles előfizetés előfordulhat, hogy lehetővé teszi a fejlesztők "közreműködői" hozzáféréssel, de az éles környezetben csak az "olvasó" hozzáféréssel rendelkeznek.
- Belső szolgáltatások és a külső szolgáltatások: Sokkal éles környezetben, illetve nem éles, például a vállalatok gyakran rendelkeznek eltérő követelmények vonatkoznak, a házirendek és a belső szolgáltatások és a külső ügyfelek által használt szolgáltatások szerepkört.

## <a name="organization-at-the-subscription-level"></a>Az előfizetés szintjén szervezet

Annak meghatározása, a szervezeti fiókok (vagy egy felügyeleti csoportok), elsősorban kell annak eldöntése, hogyan fog osztani az Azure-környezet a szervezet megfelelően. Előfizetések azonban olyan, ahol a tényleges munka történik, és ezek a döntések befolyásolják a biztonsággal, méretezhetőséggel és a számlázás.

Vegye figyelembe a következő minták, útmutatók:

- **Alkalmazás/szolgáltatás**: Előfizetések képviseli egy alkalmazás vagy szolgáltatás (alkalmazások portfólióját).

- **Életciklus**: Előfizetések képviseli egy olyan szolgáltatás, például éles vagy fejlesztői életciklusát.

- **Részleg**: Előfizetések képviselik a szervezetben lévő részlegek számára.

Az első két mintákat a leggyakrabban használt, és mindkettő erősen ajánlott. Az életciklus módszer a legtöbb szervezet számára megfelelő. Ebben az esetben az általános ajánlás az, hogy két alapszintű előfizetés használata: éles környezetben, és nem éles, majd az erőforráscsoportokat a kezdetét vette az a környezetben további.

Az előfizetések és az erőforrás Azure általános leírása csoportok használata a csoportban és -erőforrások kezelése, lásd: [erőforráshozzáférés-kezelés az Azure-ban](../../getting-started/azure-resource-access.md).

## <a name="next-steps"></a>További lépések

Ismerje meg, a hozzáférés-vezérlés és felügyelni a felhőbeli identitás-szolgáltatások használata.

> [!div class="nextstepaction"]
> [Identitáskezelés](../identity/overview.md)
