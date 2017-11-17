---
title: "Útmutató az automatikus méretezéshez"
description: "Útmutatás az alkalmazáshoz szükséges erőforrásokat dinamikusan lefoglalni automatikus skálázást."
author: dragon119
ms.date: 05/17/2017
pnp.series.title: Best Practices
ms.openlocfilehash: f2d42e9d6f4baa2da111c61fe12b48fdec785b92
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="autoscaling"></a>Automatikus skálázás
[!INCLUDE [header](../_includes/header.md)]

Automatikus skálázás dinamikusan a teljesítményre vonatkozó követelmények megfelelő erőforrások lefoglalása során a rendszer. Munka mennyiségének növekedésével az alkalmazás esetleg további erőforrásokat a kívánt teljesítményt szintek és megfelelnek a szolgáltatásiszint-szerződések (SLA). Igény szerinti slackens, és már nem szükséges további erőforrások, ezek lehetnek való hozzárendelése a költségek minimalizálása érdekében.

Automatikus skálázás kihasználja a felhő által működtetett környezetek rugalmassága közben enyhítése kezelési terhelés mellett. Csökkenti a szükséges folyamatosan figyelni a rendszer teljesítményét és megalapozott döntéseket hozzáadásával vagy eltávolításával erőforrások operátor.

Két módon fő alkalmazás is méretezhető: 

* **Függőleges skálázás**is néven skálázás felfelé és lefelé olyan erőforrás kapacitásának módosítása. Például mozgathatja az alkalmazás egy nagyobb Virtuálisgép-méretet. Függőleges skálázás gyakran szükséges a rendszer átmenetileg nem érhető el elvégzése közben újratelepítés alatt van. Ezért is ritkább függőleges skálázás automatizálásához.
* **Horizontális skálázás**is néven méretezés és, azt jelenti, hogy hozzáadásával vagy eltávolításával erőforrás példányai. Az alkalmazás továbbra is, hogy megszakítás nélkül fut, új erőforrások törlődnek. Ha a kiépítési folyamat befejeződött, a megoldást már telepítették, az további erőforrásokra. Igény szerinti csökken, ha a további erőforrások szabályszerűen leállítása és felszabadítása. 

Számos felhőalapú rendszer, beleértve a Microsoft Azure-ban a horizontális skálázás támogatja. Ez a cikk többi összpontosít horizontális skálázást.

> [!NOTE]
> Automatikus skálázás legtöbbször a számítási erőforrásokat vonatkozik. Bár egy adatbázis vagy az üzenet-várólista vízszintesen méretezése, ez általában jár [adatparticionálás][data-partitioning], amelyek általában nem automatikus.
>

## <a name="overview"></a>Áttekintés

Az automatikus skálázás stratégia általában az alábbi adatokra foglal magába:

* Instrumentation és az alkalmazás, szolgáltatás és az infrastruktúra szintjén rendszerek figyelése. Ezek a rendszerek rögzítése alapvető metrikákat, például a válaszidőt, várólista-hosszúságok meggátolják, CPU-felhasználás és memória használata.
* Döntéshozással logika, amely előre meghatározott küszöbértéket, vagy az ütemezések elleni metrikákat, és úgy dönt, hogy méretezhető-e.
* A rendszer méretezhető összetevőket.
* Tesztelési, figyelés, és annak érdekében, hogy a várt módon működik az automatikus skálázás stratégia hangolása.

Azure biztosít beépített automatikus skálázás mechanizmusok a gyakori forgatókönyvek kezeléséhez. Ha egy adott szolgáltatás vagy technológia nem rendelkezik beépített automatikus skálázás funkciót, vagy ha meghatározott automatikus skálázás követelmények meghaladja a kapacitását, érdemes lehet egy egyéni végrehajtására. Egyéni implementációja szeretné gyűjteni működési és a rendszer metrikák elemzése a metrikák és majd ennek megfelelően az erőforrások méretezése.

## <a name="configure-autoscaling-for-an-azure-solution"></a>Egy Azure megoldás az automatikus skálázás konfigurálása

Azure beépített automatikus skálázás a legtöbb számítási lehetőségeket biztosít.

* **Virtuális gépek** támogatja az automatikus skálázás használatával [Virtuálisgép-méretezési készlet][vm-scale-sets], amelyeket egy úgy, hogy egy Azure virtuális gépek halmaza csoportként kezelheti. Lásd: [hogyan használhatja az automatikus skálázás és a virtuálisgép-méretezési csoportok][vm-scale-sets-autoscale].

* **A Service Fabric** is támogatja az automatikus skálázás keresztül Virtuálisgép-méretezési készlet. A Service Fabric-fürt minden csomópont-típus egy külön Virtuálisgép-méretezési készlet lett beállítva. Így az egyes csomóponttípusok is méretezhető bejövő vagy kimenő egymástól függetlenül. Lásd: [bejövő vagy kimenő automatikus méretezése szabályok használatával a Service Fabric-fürt méretezése][service-fabric-autoscale].

* **Az Azure App Service** rendelkezik beépített automatikus skálázást. Automatikus skálázási beállításokat az összes belül egy App Service apps vonatkoznak. Lásd: [méretezése példányszám manuális vagy automatikus][app-service-autoscale].

* **Azure Cloud Services** beépített automatikus skálázás szerepkörök szintjén van. Lásd: [konfigurálása az automatikus skálázás egy felhőalapú szolgáltatás, a portál][cloud-services-autoscale].

Ezek a beállítások minden használja számítási [Azure figyelő automatikus skálázás] [ monitoring] való közös automatikus skálázás funkciókat biztosítanak.

* **Az Azure Functions** eltér az előző számítási beállítás, mert nem kell minden automatikus skálázási szabályok konfigurálása. Ehelyett az Azure Functions automatikusan lefoglalja számára számítási teljesítményt a kódja fut, amikor terhelés kezelése érdekében szükség szerint kiterjesztése. További információkért lásd: [válassza ki a megfelelő üzemeltetési terv az Azure Functions][functions-scale].

Végül egy egyéni automatikus skálázás megoldás egyes esetekben hasznos lehet. Használhatja például az Azure diagnostics és az alkalmazás-alapú metrika, figyeléséhez és exportálni az alkalmazás metrikák egyéni kód együtt. Ezután sikerült meghatározása egyéni szabályok alapján a következő metrikák tekintetében, és Resource Manager REST API-k segítségével indítható el az automatikus skálázást. Nincs egyszerű megvalósítani, azonban egyéni megoldás és kell tekinteni, csak ha az előző módszerek egyike is megfelel a követelményeknek.

A platform beépített automatikus skálázás részeit, akkor használja, ha megfelelnek a követelményeinek. Ha nem, alaposan gondolja át, hogy valóban szüksége méretezési összetettebb funkciók. További követelmények is például több granularitási vezérlő, különböző módokon eseményindító események méretezés, a különböző előfizetéseknél skálázás és a skálázás más típusú erőforrások észlelését.

## <a name="use-azure-monitor-autoscale"></a>Használja az Azure a figyelő automatikus skálázás

[Az Azure a figyelő automatikus skálázás] [ monitoring] Virtuálisgép-méretezési készlet, az Azure App Service és az Azure Cloud Service közös automatikus skálázás funkciókat biztosítanak. Skálázás is ütemezés szerint, vagy végrehajtandó futásidejű metrika, például CPU és memória-használat alapján. Példák:

- 10-példányának létrehozását a kibővítési, és a skála 4 példányokhoz szombat és vasárnap. 
- Ha átlagos CPU-használat 70 % feletti egy példány által végzett méretezés és a skála egy példánya alá való esésekor CPU-használat 50 %.
- A horizontális felskálázáshoz egy példány, ha a várólistán lévő üzenetek száma meghaladja a meghatározott küszöbérték.

A beépített mérőszámok listájáért lásd: [Azure figyelő automatikus skálázás közös metrikák][autoscale-metrics]. Egyéni metrikákat az Application Insights segítségével is megvalósíthatja. 

Konfigurálhatja az automatikus skálázás PowerShell, az Azure parancssori felület, Azure Resource Manager-sablonok vagy az Azure-portál használatával. Részletesebb vezérlés, használja a [Azure Resource Manager REST API](https://msdn.microsoft.com//library/azure/dn790568.aspx). A [Azure figyelési szolgáltatás könyvtár](http://www.nuget.org/packages/Microsoft.WindowsAzure.Management.Monitoring) és a [Microsoft Insights könyvtár](https://www.nuget.org/packages/Microsoft.Azure.Insights/) (előzetes verzió) SDK-k, amelyek lehetővé teszik a különböző erőforrások metrikák begyűjtése, és hajtsa végre az automatikus skálázás azáltal, hogy a REST API-k használatát. Az erőforrások adott Azure Resource Manager támogatása nem érhető el, vagy Azure Cloud Services használata a Service Management REST API használható az automatikus skálázást. Minden egyéb esetben az Azure Resource Manager használatára.

Az Azure automatikus skálázás használatakor, vegye figyelembe a következő szempontokat:

* Vegye figyelembe, hogy meg is előre jelezni az alkalmazás terhelésének elégséges ütemezett automatikus skálázás, használandó hozzáadása és eltávolítása, hogy a kereslet várható csúcsait megfelelnek. Ha ez nem lehetséges, használjon futásidejű mérőszámok alapján reaktív automatikus skálázás igény szerinti előre nem látható változásainak kezeléséhez. Általában ezek a módszerek kombinálásával. Hozzon létre például egy stratégiát, amely az időpontokat, amikor tudja, hogy az alkalmazás leginkább foglalt ütemezés alapján. Ezzel biztosíthatja, hogy kapacitás áll rendelkezésre, ha szükséges, új példányok elindítását késedelem nélkül. Az egyes ütemezett szabályokat, amelyek lehetővé teszik a reaktív automatikus skálázás annak érdekében, hogy az alkalmazás kezeli a tartós azonban előre nem látható csúcsait igény szerint az adott időszakban-metrikáinak definiálása.
* Legtöbbször a metrikák és a kapacitás követelményeit, különösen akkor, ha egy alkalmazás először lett telepítve közötti kapcsolat megértése nehéz feladat. Kiépítése elején, egy kis további kapacitást és majd figyelhető meg és hangolható a kapacitás közelebb kerüljön a tényleges betöltést az automatikus skálázás szabályokat.
* Konfigurálja az automatikus skálázás szabályokat, és ezután az adott idő alatt az alkalmazás teljesítményének figyeléséhez. Ez az ellenőrzés eredményeit segítségével állítsa be, amelyben a rendszer méretezi szükség módja. Vegye azonban figyelembe, hogy az automatikus skálázás nincs egy azonnali folyamat. Átlagos CPU kihasználtsága meghaladja a (vagy alá) például metrika reagálni időt vesz igénybe a megadott küszöbértéket.
* Egy mért eseményindító attribútum (például a Processzor kihasználtsága vagy a várólista hossza) alapuló észlelés mechanizmus használó szabályok automatikus skálázás keresztül használja a összesített értéket idő, nem pedig azonnali értékek automatikus skálázás művelet indításához. Alapértelmezés szerint az összesített értékek átlagát. Ez megakadályozza, hogy a rendszer túl gyorsan reagál, vagy gyors rezgési okozza. Lehetővé teszi új-példányok automatikus-indított rendezésére mód bekövetkezését, amíg az új példányok indul további automatikus skálázás műveletek akadályozza meg, hogy fut az idő. Az Azure felhőalapú szolgáltatásairól és az Azure virtuális gépek, az alapértelmezett időszak összesítést 45 percben, így is igénybe vehet a metrika az igény szerinti igényeiben jelentkező válaszul automatikus skálázás indításához ennyi ideig. Az összesítési időszak módosíthatja az SDK használatával, de vegye figyelembe, hogy kevesebb mint 25 percig időszakok előre nem látható eredményekhez vezethet (további információkért lásd: [automatikus skálázás Felhőszolgáltatások CPU százalékán Azure figyelési szolgáltatásokkal Könyvtár](http://rickrainey.com/2013/12/15/auto-scaling-cloud-services-on-cpu-percentage-with-the-windows-azure-monitoring-services-management-library/)). A Web Apps az átlagos időtartam sokkal rövidebb, így öt perc után módosítva lett a átlagos eseményindító mérték rendelkezésre álló új példányok.
* Ha konfigurálja automatikus skálázás helyett az SDK-t a portálhoz, során, ami a szabályok aktívak részletesebb ütemezés is megadhat. Hozzon létre egy saját metrikák is, és vagy anélkül, hogy a meglévőket az automatikus skálázás szabályokban bármelyikét használhatja őket. Például előfordulhat, hogy kívánja alternatív számlálók, például a kérelmek / másodperc vagy a memória átlagos rendelkezésre állás érdekében a használja vagy egyéni adott üzleti folyamatok mérő számlálókat.
* Amikor a Service Fabric automatikus skálázást, a fürt a csomóponttípusok a Virtuálisgép-méretezési beállítja a háttér, így az egyes csomóponttípusok automatikus méretezése szabályokat kell. Vegye figyelembe a csomópontok számát, amelyekkel rendelkeznie automatikus skálázás beállítása előtt. A megbízhatósági szint választott célja a rendelkeznie kell az elsődleges csomóponttípusok a csomópontok minimális száma. További információk: [bejövő vagy kimenő automatikus méretezése szabályok használatával a Service Fabric-fürt méretezése](https://docs.microsoft.com/en-us/azure/service-fabric/service-fabric-cluster-scale-up-down).
* A portál segítségével erőforrások, például SQL adatbázis-példány és a várólisták összekapcsolása egy Cloud Service-példány. Ez lehetővé teszi a manuális és automatikus méretezési konfigurációs beállításokat az egyes a kapcsolt erőforrásokban külön könnyebben eléréséhez. További információkért lásd: [hogyan: erőforrás összekapcsolása egy felhőalapú szolgáltatás](/azure/cloud-services/cloud-services-how-to-manage).
* Több házirendek és szabályok konfigurálásakor sikerült ütköznek egymással. Automatikus skálázás az alábbi ütközés feloldása szabályok használatával győződjön meg arról, hogy van-e mindig a megfelelő mennyiségű példánnyal fut:
  * Kiterjesztési műveletek mindig a elsőbbséget élveznek a skálázási műveletek.
  * Ha horizontális felskálázás műveletek ütközést, a szabály a példányok száma a legnagyobb növekedése kezdeményező lép érvénybe.
  * Ha vertikális műveletek ütközik, a szabály a példányok száma a legkisebb csökkenése kezdeményező lép érvénybe.
* Egy App Service Environment-környezetben a feldolgozókészleten vagy előtér-metrikák használható automatikus skálázási szabályok meghatározásához. További információkért lásd: [automatikus skálázás és az App Service Environment-környezet](/azure/app-service/app-service-environment-auto-scale).

## <a name="application-design-considerations"></a>Alkalmazás kialakítási szempontok
Automatikus skálázás nem egy azonnali megoldás. Egyszerűen erőforrások hozzáadása a rendszert, vagy a folyamatnak több példánya fut nem garantálja, hogy javítja a rendszer teljesítményét. Az automatikus skálázás stratégia tervezésekor, vegye figyelembe a következő szempontokat:

* A rendszer úgy kell megtervezni, hogy vízszintesen méretezhető. Példány affinitás; feltételezéseket elvégzése nem tervezze meg, hogy a kód állandó futásának egy folyamat adott példányt igénylő megoldások. Amikor skálázás vízszintesen egy felhőszolgáltatás vagy webhely, nem érdemes feltételezni, hogy több kérelem ugyanarról a forrásról mindig továbbítja a ugyanezen példányában. Ugyanezen okból tervezze meg több szolgáltatás ugyanazon példányára mindig átirányítás alkalmazásból kérelem igénylő elkerülése érdekében állapotmentes szolgáltatások. Olvassa be az üzenetek várólistából való várólistából, és feldolgozza azokat szolgáltatás tervezésekor nem ellenőrizze a szolgáltatás leíróinak példányok bármely feltételezéseket egy adott üzenet. A várólista hossza növekedésével automatikus skálázás elindítása volt a szolgáltatás további példányát. A [versengő fogyasztók mintát] [ competing-consumers] ebben a forgatókönyvben kezelésének módját ismerteti.
* Ha a megoldás megvalósítja a hosszan futó feladatot, tervezze meg ezt a feladatot kiterjesztése és a méretezés is támogatja. Határidő nélküli eljárni, egy ilyen feladat megakadályozhatja a folyamat egy példányát szabályszerűen amikor a rendszer méretezi a Leállítás, vagy ha a folyamat kényszerített megszakadt adatok megszakadhat. Ideális esetben refactor a hosszan futó feladatot, és szakítsa meg a feldolgozás, amely kisebb, különálló adattömbökbe végez. A [pipe-ok és a szűrők mintát] [ pipes-and-filters] példával szemlélteti, hogyan lehet ezt elérni.
* Azt is megteheti valósíthatja meg, hogy a rekordok állapot rendszeres időközönként a feladattal kapcsolatos információkat, és ebben az állapotban mentése az a folyamat, a feladatütemezés futtatását bármelyik példánya részéről elérhető tartós storage ellenőrzőpont mechanizmus. Így a folyamat leáll, ha a munkát végző azt is folytatni a legutóbbi ellenőrzőponttól egy másik példánnyal.
* Háttérfeladatok futtatása külön számítási példányokért, például feldolgozói szerepkörök a felhőalapú szolgáltatások üzemeltetett alkalmazás, szükség lehet az alkalmazás különböző méretezési házirendekkel különböző részei méretezése. Például szükség lehet további felhasználói felület (UI) számítási telepítendő példányok nélkül növelje meg a háttérben számítási példányok, vagy ez ellentéte. Ha (például az alapszintű és a prémium szintű szolgáltatáscsomagok) szolgáltatás különböző szintjei, szükség lehet a számítási erőforrásokat premium szolgáltatás csomagok agresszívabb, mint az alapszintű service csomagokat kiterjesztése érdekében végzett eszközfigyelés SLA-k.
* Fontolja meg a várólista, amelyben felhasználói felület és a háttér számítási példányok kommunikálnak az automatikus skálázás stratégia feltételként hosszát. Ez az a legjobb jelzi, hogy egy egyensúlyhiány vagy a jelenlegi terhelés és a háttér-feladat a feldolgozási kapacitás különbsége.
* Ha az automatikus skálázás stratégia alapjául üzleti folyamatokat, például az egyes óra vagy egy összetett tranzakció átlagos végrehajtási idő leadott megrendelések száma mérő számlálók győződjön meg arról, hogy megértette a ezekből az eredmények közötti kapcsolat számlálók és a tényleges számítási kapacitásigények típusú. Elképzelhető, hogy egynél több méretezhető, vagy az üzleti folyamatok számlálói változásai egység számítási szükséges.  
* Kísérlet túlzottan horizontális, és példányok ezreit társított költségek elkerülése érdekében a rendszer elkerülése érdekében fontolja meg, automatikusan hozzáadhatók-példányok maximális számának korlátozása. A legtöbb automatikus skálázás mechanizmusok engedélyezi, hogy adja meg a szabály a példányok minimális és maximális száma. Emellett érdemes lehet szabályosan terhelése a szolgáltatásokat, a rendszer biztosít, ha a példányok maximális száma telepítették, és a rendszer továbbra is túl van terhelve.
* Tartsa szem előtt, hogy az automatikus skálázás nem feltétlenül meg a legmegfelelőbb mechanizmus egy hirtelen kapacitásnövelés a munkaterhelés kezeléséhez. Kiépíteni, és indítsa el a szolgáltatás új példányát, vagy erőforrások hozzáadása a rendszer időt vesz igénybe, és a maximális igény szerint is telt ezen további erőforrások elérhetővé tett ideje szerint. Ebben a forgatókönyvben jobb megoldás lehet a szolgáltatás szabályozása. További információkért lásd: a [sávszélesség-szabályozás mintát][throttling].
* Ezzel szemben a kapacitás dolgozza fel az összes kérelmet, ha a kötet gyorsan ingadozik, és költség nem jelentős meghatározó tényező, javasolt, hogy további példányokat gyorsabban elindul egy agresszív automatikus skálázás stratégia használata. Egy ütemezett házirend felel meg a legnagyobb terhelést, mielőtt terhelés várható, hogy a megfelelő számú elinduló is használható.
* Az automatikus skálázás mechanizmust kell az automatikus skálázás folyamat figyelésére, és minden automatikus skálázás esemény adatainak naplózása (azt, hogy milyen erőforrásokat hozzáadott vagy eltávolított, kiváltó okáról, és ha). Ha létrehoz egy egyéni automatikus skálázás mechanizmus, győződjön meg arról, hogy ez a funkció magában foglalja. Elemezheti az adatait a mérték, az automatikus skálázás stratégiájának hatékonyságát, és szükség esetén igazítható. Észlelheti a rövid távú, mind a használati minták egyre több nyilvánvaló, és több hosszú távon, az üzleti bővíti, illetve fejleszteni az alkalmazás követelményeinek. Ha egy alkalmazás eléri a felső korlátot definiálva az automatikus skálázást, a mechanizmus is előfordulhat, hogy riasztás, egy olyan operátort, akik manuálisan indítsa el a további erőforrások, ha szükséges. Előfordulhat, hogy, ilyen körülmények a következő operátor is manuálisan eltávolítja ezeket az erőforrásokat, után a munkaterhelés megkönnyíti a felelős.

## <a name="related-patterns-and-guidance"></a>Útmutató és a kapcsolódó minták
A következő mintákat és útmutatókat is a forgatókönyvhöz kapcsolódó automatikus skálázás végrehajtása során:

* [Sávszélesség-szabályozás mintát][throttling]. Ebben a mintában ismerteti, hogyan alkalmazás továbbra is működik, és eleget tegyen a szolgáltatói szerződéseknek, ha az igény szerinti növelését egy rendkívüli terhelés erőforrástól függ. Sávszélesség-szabályozás segítségével az automatikus skálázást a rendszer nem egyszerre tiltása, amíg a rendszer kimenő méretezi.
* [Versengő fogyasztók mintát][competing-consumers]. Ebben a mintában ismerteti, hogyan kezelhető bármely alkalmazáspéldány üzeneteit szolgáltatáspéldány készletét végrehajtásához. Automatikus skálázás indítása és leállítása a várható munkaterheléshez szolgáltatáspéldány használható. Ez a megközelítés lehetővé teszi, hogy a rendszer több üzenetet párhuzamosan a teljesítmény optimalizálása, jobb méretezhetőséget és a rendelkezésre állási és munkaterhelésének feldolgozni.
* [Megfigyelési és diagnosztikai](./monitoring.md). Rendszerállapot- és telemetriabevitelt összegyűjtik azokat az információkat, amelyek az automatikus skálázás folyamat is meghajtó fontosságúak.


<!-- links -->

[monitoring]: /azure/monitoring-and-diagnostics/monitoring-overview-autoscale
[app-service-autoscale]: /azure/monitoring-and-diagnostics/insights-how-to-scale?toc=%2fazure%2fapp-service-web%2ftoc.json#scaling-based-on-a-pre-set-metric
[app-service-plan]: /azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview
[autoscale-metrics]: /azure/monitoring-and-diagnostics/insights-autoscale-common-metrics
[cloud-services-autoscale]: /azure/cloud-services/cloud-services-how-to-scale-portal
[competing-consumers]: ../patterns/competing-consumers.md
[data-partitioning]: ./data-partitioning.md
[functions-scale]: /azure/azure-functions/functions-scale
[link-resource-to-cloud-service]: /azure/cloud-services/cloud-services-how-to-manage#how-to-link-a-resource-to-a-cloud-service
[pipes-and-filters]: ../patterns/pipes-and-filters.md
[service-fabric-autoscale]: /azure/service-fabric/service-fabric-cluster-scale-up-down
[throttling]: ../patterns/throttling.md
[vm-scale-sets]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
[vm-scale-sets-autoscale]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview
