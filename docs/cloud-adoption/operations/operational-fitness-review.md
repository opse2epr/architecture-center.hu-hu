---
title: 'Enterprise Cloud Adoption: Működési – alapok'
description: Útmutatás a működési – alapok
author: petertaylor9999
ms.date: 09/20/2018
ms.openlocfilehash: d5f4c6529e92be387465a6ab9dca55267c584c11
ms.sourcegitcommit: b7e521ba317f4fcd3253c80ac0c0a355eaaa56c5
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 09/21/2018
ms.locfileid: "46534588"
---
# <a name="establishing-an-operational-fitness-review"></a>Működési mentességre felülvizsgálat létrehozása

A vállalati megkezdi a működését az Azure-beli számítási feladatokhoz, mint a következő lépés létrehozására egy **működési mentességre tekintse át** folyamat enumerálásához, megvalósításához és iteratív tekintse át a **nem működőképes** követelmények ezeket a feladatokat. _Nem funkcionális_ a szolgáltatás a várt működési viselkedés kapcsolatos követelményeket. A továbbiakban nem funkcionális követelmények öt alapvető kategóriába sorolhatók a [szoftverminőség alappillérei](../../guide/pillars.md): skálázhatóság, rendelkezésre állás, rugalmasság (beleértve az üzleti folytonossági és vészhelyreállítási helyreállítási), kezeléséhez, és biztonság. A egy működési mentességre elbírálási folyamata célja annak ellenőrzése, hogy az üzletmenet szempontjából kritikus fontosságú számítási feladatok megfelelnek az elvárásainak, a minőség alappillérei garanciát az üzleti.

Ebből kifolyólag a vállalati vállalnia kell a egy működési mentességre elbírálási folyamata teljes mértékben megérteni a számítási feladatok éles környezetben való futtatása eredő problémák, határozza meg, hogyan javíthatja a problémákat, majd oldja meg őket. Ez a cikk ismerteti egy magas szintű működési mentességre felülvizsgálati folyamat, amellyel a vállalat e cél eléréséhez.

## <a name="operational-fitness-at-microsoft"></a>A Microsoft Operational alkalmasságát

Kezdettől fogva az Azure platform fejlődése lett, a folyamatos fejlesztési és a Microsoft számos csapat által végzett adatintegrációs projekt. A megfelelő minőség és konzisztencia az Azure-mérete és összetettsége nélkül egy robusztus folyamat enumerálása, és a nem funkcionális követelmények megvalósításának rendszeresen projekt nagyon nehéz lenne.

A Microsoft követi ezeket a folyamatokat vevőrendelési azok számára, a jelen dokumentumban vázolt.

## <a name="understanding-the-problem"></a>A probléma ismertetése

Amint már tudja, a [bevezetés](../../cloud-adoption/getting-started/overview.md), az első lépés egy vállalati digitális átalakulást az Azure bevezetésével megoldandó üzleti problémák azonosítja. A következő lépés az, hogy egy magas szintű megoldást a problémára, például a számítási feladatok migrálása a felhőbe, vagy egy meglévő gyakorlatainak helyszíni szolgáltatás cloud funkciókat. Végül a megoldás kialakításának és implementálva.

Ez a folyamat során a hangsúly be kapcsolva gyakran a _funkciók_ a szolgáltatás. Vannak, egy kívánt _működési_ végrehajtásához szolgáltatás követelményeit. Például egy termék kézbesítési szolgáltatás a termék, a termék követési kézbesítési, felhasználói értesítések és egyéb során a forrás- és helyek meghatározásához szükséges funkciókat.

Ellentétben a _nem működőképes_ követelmények vonatkoznak a tulajdonságokat, például a szolgáltatás [rendelkezésre állási](../../checklist/availability.md), [rugalmasság](../../resiliency/index.md), és [méretezhetőség](../../checklist/scalability.md). Ezek a tulajdonságok abban tér el a funkcionális követelmények, nem közvetlenül befolyásolják a szolgáltatás bármely adott funkció végső funkcióját. Azonban ezek nem funkcionális követelmények kapcsolódnak a _teljesítmény_ és _folytonossági_ a szolgáltatás.

Néhány nem funkcionális követelmények alapján a szolgáltatói szerződés (SLA) adható meg. Például tekintetében a szolgáltatás folyamatossága érdekében a szolgáltatás egy rendelkezésre állási követelmény jelöl százalékos például **elérhető 99,99 %-ában**. Egyéb nem funkcionális követelmények meghatározásához nehezebb lehet, és éles igényei megváltoznak változhatnak. Például egy fogyasztói szolgáltatás indítása előfordulhat, hogy a nem várt átviteli sebességet megkövetelő irányuló után népszerűség, megnő.

! [MEGJEGYZÉS] A rugalmasság érdekében magyarázatok RPO, RTO, SLA-t és a kapcsolódó fogalmak, beleértve a követelmények meghatározásánál tart teljesítménykezelési részletesebben [rugalmas alkalmazások tervezése az Azure](../../resiliency/index.md#define-your-availability-requirements).

## <a name="operational-fitness-review-process"></a>Működési mentességre elbírálási folyamata

A teljesítmény és a egy vállalati szolgáltatások folyamatosságát fenntartása fontos, hogy végrehajtja- _működési mentességre felülvizsgálati_ folyamat.

![A működési mentességre felülvizsgálati folyamat áttekintése](_images/ofr-flow.png)

Magas szinten a folyamat két fázissal rendelkezik. A követelmények az Előfeltételek szakaszban létrehozott és támogató szolgáltatásai leképezve. Ez a kevésbé gyakran; történik Talán évente, vagy ha új műveletek jelennek meg. Az Előfeltételek szakasz kimenetét a folyamat fázisban használja. A flow fázisban történik gyakrabban; Javasoljuk, hogy havonta.

### <a name="prerequisites-phase"></a>Előfeltételek fázis

Ebben a fázisban a lépések célja, hogy a szükséges követelményeket, a fontos szolgáltatások rendszeres felülvizsgálatát végző rögzítése.

- **Azonosítsa a létfontosságú üzleti tevékenységeket**. Azonosítsa a vállalati **alapvető fontosságú** üzleti műveletek. Üzleti műveletek függetlenek bármilyen kiegészítő szolgáltatást. Más szóval az üzleti műveletek a tényleges tevékenységeket, amelyek az üzleti kell végrehajtania, és az informatikai szolgáltatások által támogatott képviseli. Az előfizetési időszak _alapvető fontosságú_, vagy másik lehetőségként _üzletileg kritikus_, tükrözi egy súlyos hatása a vállalat számára, ha a művelet hátráltatja van. Egy online kereskedő Előfordulhat például, ha egyes üzleti műveletekhez, például "elem hozzáadása egy bevásárlókosárhoz ügyfelek engedélyezése" vagy "feldolgozni a hitelkártyás fizetés". Ha ezen műveletek egyike meghibásodik, egy ügyfél nem lehet végrehajtani a tranzakciót, és a vállalat értékesítési kihasználására – sikertelen lesz.

- **Műveletek leképezése szolgáltatások**. Képezze le az azokat támogató szolgáltatások e üzleti műveletek. A fenti vásárlási bevásárlókocsi példában több olyan szolgáltatást vehet részt: egy készlet tőzsdei felügyeleti szolgáltatás, egy vásárlási Bevásárlókocsi szolgáltatás és mások. Hitelkártyás fizetés a fenti példában egy helyszíni fizetési szolgáltatást egy külső fizetési szolgáltatás is dolgozhat.

- **Elemezheti a szolgáltatásfüggőségek**. A legtöbb üzleti tevékenységekhez szükséges vezénylési több támogató szolgáltatások között. Fontos tudni, hogy a függőségeket a és a folyamat a működés szempontjából kritikus fontosságú tranzakciók a szolgáltatásokon keresztül. A helyszíni szolgáltatásokhoz és az Azure-szolgáltatások közti függőségeket is figyelembe kell venni. A vásárlási bevásárlókocsi példában a készlet tőzsdei felügyeleti szolgáltatás üzemeltethető a helyszínen, és egy fizikai adatraktárból az alkalmazottak által bevitt adatok feldolgozására, de előfordulhat, hogy tárolni adatok Azure-szolgáltatások például [az Azure storage](/azure/storage/common/storage-introduction) vagy adatbázis például [az Azure Cosmos DB](/azure/cosmos-db/introduction).

Ezek a tevékenységek kimenetének egy olyan **stratégiai mutatószámrendszer metrikák** szolgáltatási műveletek. A metrikák szempontjából nem működőképes jellemzői – például a rendelkezésre állás, a méretezhetőség és a vész-helyreállítási telefonokat. Scorecard metrikák expressz a feltételeket, amelyek a szolgáltatás várhatóan illesztő nem felel meg. Ezek a metrikák, a műveletet megfelelő részletességgel bármilyen szinten lehet kifejezni.

A stratégiai mutatószámrendszer egyszerűen fogalmazva a jelentéssel bíró párbeszéd fenntartása az üzleti tulajdonosai és a mérnöki csapathoz között kell megadni. Például méretezhetőség stratégiai mutatószámrendszer metrika sikerült kifejezhető _zöld_ végrehajtásához, a kívánt feltételeket, _sárga_ nem felelnek meg a kívánt feltételeket, de aktívan a tervezett végrehajtási a szervizelés és _piros_ nem felelnek meg a kívánt feltételeket terv és a művelet számára.

Fontos kiemelni, hogy ezek a metrikák közvetlenül kell tükrözniük üzleti igényeknek megfelelően.

### <a name="service-review-phase"></a>Szolgáltatás áttekintése. fázis

A szolgáltatást felülvizsgálat fázisban központi eleme a működési mentességre ellenőrzés során.

- **Szolgáltatási metrikák mérik**. A stratégiai mutatószámrendszer mérőszámok segítségével, a szolgáltatások ellenőrizni kell, hogy, hogy megfeleljenek az üzleti követelményeinek. Ez azt jelenti, hogy szolgáltatás a figyelés létfontosságú. Ha nem képes szolgáltatások garanciát a nem funkcionális követelmények készletének a figyelése, majd a megfelelő stratégiai mutatószámrendszer metrikák kell tekinteni piros. Ebben az esetben az első lépés a szervizelési, hogy a megfelelő szolgáltatás megfigyelésének.
Tételezzük fel, ha az üzleti vár egy szolgáltatás 99,99 %-os rendelkezésre állással működik, de a van nem éles környezetben nem működik a telemetria helyen rendelkezésre állásának mérésére, meg kell, hogy, már nem felel meg a követelménynek.

- **Szervizelés tervezése**. Beállítása, hogy minden egyes szolgáltatás metrikákkal, amely egy elfogadható küszöb alá esnek meghatározhatja a szolgáltatást ahhoz, hogy a művelet egy elfogadható metrika szervizelés költségét. Ha a szervizelés alatt áll a szolgáltatás költsége nagyobb, mint a szolgáltatás a várt bevételnövekedést, folytassa a nem materiális költségeket, például a felhasználói élményt nyújt. Például ügyfelek nehézséget okoz a sikeres sorrendet a szolgáltatás használatával helyezi el, ha azok dönthet egy versenytárs helyette.

- **Szervizelés végrehajtása**. Miután az üzleti tulajdonosai és mérnöki éri el a díjcsomagot, azt kell végrehajtani. A megvalósítás állapotát, amikor stratégiai mutatószámrendszer metrikák nyilvánosan lektorálhatók kell jelenteni.

Ez a folyamat iteratív, és ideális a vállalati csapat célja, hogy a tulajdonos, rendelkeznie kell. Ennek a csapatnak meg kell felelnie rendszeresen tekintse át a meglévő szervizelési projektek, a fundamentals tekintse át az új számítási feladatok indíthat és nyomon követheti a vállalat általános stratégiai mutatószámrendszer. A csapat kell jogosultak biztosítása érdekében szervizelési csapatok ütemezés mögött található, vagy nem felelnek meg a metrikák.

## <a name="structure-of-the-operational-fitness-review-team"></a>A működési mentességre csapatától szerkezete

A működési mentességre csapatától a következő szerepkörök tevődik össze:

1. **Üzleti tulajdonosa**. Ezt a szerepkört biztosít a cég és minden egyes "alapvető fontosságú" üzleti műveletekhez priorizálhatja a ismerete. Ez a szerepkör is hasonlítja össze a megoldás költsége az üzletmenetre gyakorolt hatás, és a végleges döntést meghajtókat a fenyegetés kiküszöbölését.

2. **Üzleti tanácsadó**. Ez a szerepkör felelős bontásához, üzleti műveletek alfeladatra részre, és a részeket hozzárendelése a helyszíni és a cloud services és az infrastruktúra. A szerepkör megköveteli a részletes technológiai tudásukat kihasználva az egyes üzleti műveletekhez tartozó.

3. **Mérnöki csapathoz tulajdonosa**. Ez a szerepkör felelős a szolgáltatásokat az üzleti művelet társított implementating. Ezek a személyek részt vehet a tervezési, végrehajtására és bármely megoldására nem működőképes követelmény, amelyeket a működési mentességre tekintse át a csapat által megoldások üzembe helyezéséhez.

4. **Szolgáltatás tulajdonosa**. Ez a szerepkör felelős működő, az üzleti alkalmazásokat és szolgáltatásokat. Ezek a személyek ezen alkalmazások és szolgáltatások számára a naplózási és használati adatokat gyűjteni. Ezen adatok segítségével is azonosíthatja a problémákat, és ellenőrizze a telepítés után javításokat.

## <a name="operational-fitness-review-meeting"></a>Működési mentességre tekintse át az értekezlet

Azt javasoljuk, hogy megfelelnek-e a működési mentességre csapatától rendszeres időközönként. Ha például a team megfelelnek sikerült egy havi-váltás gyakoriságáról és a jelentés állapota és a metrikák elnyerése negyedévente.

A folyamat, valamint a értekezlet saját igényeinek megfelelően módosítani kell. Javasoljuk, hogy a következő feladatokat kiindulási pontként:

1. A vállalat tulajdonosa és üzleti tanácsadó enumerálása, és meghatározhatja az egyes üzleti műveletekhez a termékgondozó csoportja és a szolgáltatások tulajdonosait a bemenetet a nem funkcionális követelményeket. Az üzleti tevékenységekhez által korábban azonosított a prioritás tekintse át és ellenőrizve van. Az új üzleti tevékenységekhez a meglévő lista prioritást hozzá van rendelve.

2. A mérnöki és a szolgáltatás tulajdonosait leképezése a **aktuális állapota** üzleti műveletek a megfelelő helyszíni és a cloud services. A hozzárendelés az egyes szolgáltatásokhoz, mint egy függőségi fán orientált összetevői listáját tevődik össze. Ha a lista és a függőségi fa jönnek létre, a **kritikus útvonalakat** a fa keresztül történik.

3. A termékgondozó csoportja és a szolgáltatások tulajdonosait tekintse át a műveleti naplózás és figyelés az előző lépésben felsorolt szolgáltatások aktuális állapotát. Hatékony naplózás és figyelés kritikusak, szolgáltatás-összetevők, amelyek hozzájárulnak a nem funkcionális követelmények teljesítéséhez failuring azonosítása érdekében. Ha elegendő naplózás és figyelés nem teljesülnek, egy csomagot kell létrehozni és implementált helyen helyezi őket.

4. Scorecard metrikákat hoz létre az új üzleti művelet. A stratégiai mutatószámrendszer minden szolgáltatáshoz igazítva a nem funkcionális követelmények és a egy metrika jelölő arról, hogy az összetevő megfelel-e a követelményeknek, 2. lépésben azonosított alkotó összetevők listájának tevődik össze.

5. A különböző összetevőket, amelyek nem felelnek meg a nem funkcionális követelmények egy magas szintű megoldások készült, és mérnöki tulajdonos van hozzárendelve. Ezen a ponton a vállalat tulajdonosa és üzleti tanácsadó kell létesítenie a szervizelési munkát, az üzleti művelet várható bevétel alapján költségvetést.

6. Végül felülvizsgálat folyamatban lévő szervizelési munkát végez. Minden folyamatban lévő munka a stratégiai mutatószámrendszer metrikák felülvizsgálata a várt metrikák alapján. Metrikák értekezlet alkotó összetevők a szolgáltatás tulajdonosa Megadja annak ellenőrzéséhez, hogy teljesül-e a metrika a naplózást és figyelést adatok. Alkotó összetevőket, amelyek nem értekezlet-metrikák, az egyes mérnöki tulajdonosa, ezért nem metrikák, hogy a rendszer elérte a problémákat, és egyetlen új tervet szervizelési ismerteti.

## <a name="recommended-resources"></a>Ajánlott erőforrások

- [Szoftverminőség alappillérei](../../guide/pillars.md).
Az Azure architektúra-útmutató ezen szakasza a szoftverminőség öt alappillérét ismerteti: skálázhatóság, rendelkezésre állás, rugalmasság, felügyeleti és biztonsági.
- [10 tervezési elvek, az Azure-alkalmazásokhoz](../../guide/design-principles/index.md).
Az Azure architektúra-útmutató ezen szakasza ismerteti a tervezési alapelveket az alkalmazás méretezhető, rugalmas és kezelhető egy készletét.
- [Rugalmas alkalmazások tervezése az Azure](../../resiliency/index.md).
Ez az útmutató a kifejezés rugalmasság és a kapcsolódó fogalmak ismertetésével kezdődik. Ezután ismerteti a rugalmasság elérésének egy, a tervezéstől és implementálástól a központi telepítésig és üzemeltetésig terjedő folyamatát, amely az alkalmazások élettartama során strukturált megközelítést használ.
- [Tervezési minták felhőkhöz](../../patterns/index.md).
Ezek a tervezési minták hasznosak fejlesztőcsapatai a szoftverminőség alappillérei alkalmazások készítése során.