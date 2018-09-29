---
title: Útmutató az automatikus skálázás
description: Útmutatás az automatikus skálázáshoz az alkalmazások által igényelt erőforrások dinamikus lefoglalása érdekében.
author: dragon119
ms.date: 05/17/2017
pnp.series.title: Best Practices
ms.openlocfilehash: 1afbb7d48329d84979aa72b7e0707442cacfa45e
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 09/28/2018
ms.locfileid: "47429417"
---
# <a name="autoscaling"></a>Automatikus skálázás
[!INCLUDE [header](../_includes/header.md)]

Az automatikus skálázás az a folyamat, amely dinamikusan lefoglalja az erőforrásokat a teljesítményre vonatkozó követelményeknek megfelelően. Az elvégzendő feladatok mennyiségének növekedésével az alkalmazásoknak további erőforrásokra lehet szükségük ahhoz, hogy fenn tudják tartani a kívánt teljesítményszinteket és megfeleljenek a szolgáltatásiszint-szerződésekben (SLA) leírtaknak. Ahogy az erőforrásigény csökken, a további erőforrások felszabadíthatók a költségek csökkentése érdekében.

Az automatikus skálázás kihasználja a felhőben futtatott környezetek rugalmasságát, ugyanakkor csökkenti a kezelési terheket. Ezzel a megoldással kevésbé van szükség olyan operátorra, aki folyamatosan monitorozza a rendszer teljesítményét, és döntéseket hoz az erőforrások hozzáadásáról vagy eltávolításáról.

Az alkalmazásokat két fő módszerrel lehet skálázni: 

* A **vertikális skálázás**, más néven fel- és leskálázás az erőforrások kapacitásának módosítását jelenti. Például az alkalmazások áthelyezhetők egy nagyobb méretű virtuális gépre. A vertikális skálázáshoz a rendszereket általában ideiglenesen elérhetetlenné kell tenni az újbóli üzembe helyezés előtt. Emiatt a vertikális skálázás automatizálása kevésbé gyakori.
* A **horizontális skálázás**, más néven is néven horizontális fel- és leskálázás erőforráspéldányok hozzáadását vagy eltávolítását jelenti. Az alkalmazás az új erőforrások kiépítése közben továbbra is megszakítás nélkül fut. Ha a kiépítési folyamat befejeződött, a megoldás a további erőforrásokon is üzembe helyezésre kerül. Ha az igény csökken, a további erőforrások probléma nélkül leállíthatók és felszabadíthatók. 

Számos felhőalapú rendszer, köztük a Microsoft Azure is támogatja az automatikus horizontális skálázást. A cikk további része a horizontális skálázással foglalkozik.

> [!NOTE]
> Az automatikus skálázás legfőképp a számítási erőforrásokra vonatkozik. Bár lehetséges az adatbázisok vagy az üzenetsorok horizontális skálázása is, ehhez általában [adatparticionálás][data-partitioning] szükséges, ami legtöbbször nem automatizált.
>

## <a name="overview"></a>Áttekintés

Az automatikus skálázási stratégia általában a következőket foglalja magába:

* Rendszerek kialakítása és monitorozása az alkalmazás, szolgáltatás és infrastruktúra szintjén. Ezek a rendszerek olyan alapvető metrikákat rögzítenek, mint a válaszidők, az üzenetsorok hossza, a CPU-használat és a memóriahasználat.
* Döntéshozási logika, amely e metrikákat az előre meghatározott küszöbértékekkel vagy ütemezésekkel összevetve értékeli, és eldönti, hogy szükség van-e skálázásra.
* A rendszert skálázó összetevők.
* Az automatikus skálázási stratégia tesztelése, monitorozása és finomhangolása annak érdekében, hogy a várt módon működjön.

Az Azure beépített automatikus skálázási mechanizmusokat biztosít, amelyek a gyakori forgatókönyvekhez használhatók. Ha egy adott szolgáltatás vagy technológia nem rendelkezik beépített automatikus skálázási funkcióval, vagy ha az adott automatikus skálázási követelmények meghaladják a kapacitását, érdemes lehet egy egyéni kialakítást használni. Az egyéni kialakítások összegyűjtik a működési és rendszermetrikákat, amelyeket elemeznek, majd ennek megfelelően skálázzák az erőforrásokat.

## <a name="configure-autoscaling-for-an-azure-solution"></a>Egy Azure-megoldás automatikus skálázásának konfigurálása

Az Azure beépített automatikus skálázást nyújt a legtöbb számítási lehetőséghez.

* A **Virtual Machines** az automatikus skálázást [virtuálisgép-méretezési csoportok][vm-scale-sets] használatával támogatja, amelyek segítségével több Azure-beli virtuális gép egy csoportként kezelhető. Lásd: [Az automatikus skálázás és a virtuálisgép-méretezési csoportok használata][vm-scale-sets-autoscale].

* A **Service Fabric** szintén támogatja a virtuálisgép-méretezési csoportok használatával végzett automatikus skálázást. A Service Fabric-fürtök minden csomóponttípusa egy külön virtuálisgép-méretezési csoportként üzemel. Így az egyes csomóponttípusok egymástól függetlenül skálázhatók fel vagy le horizontálisan. Lásd: [Service Fabric-fürt horizontális fel- vagy leskálázása automatikus skálázási szabályok használatával][service-fabric-autoscale].

* Az **Azure App Service** beépített automatikus skálázási funkcióval rendelkezik. Az automatikus skálázási beállítások egy adott App Service-szolgáltatáson belüli összes alkalmazásra vonatkoznak. Lásd: [Példányszám manuális vagy automatikus skálázása][app-service-autoscale].

* Az **Azure Cloud Services** beépített szerepkörszintű automatikus skálázási funkcióval rendelkezik. Lásd: [Felhőszolgáltatás automatikus skálázásának konfigurálása a portálon][cloud-services-autoscale].

Ezek a számítási lehetőségek mind az [Azure Monitor automatikus skálázása][monitoring] segítségével biztosítják a közös automatikus skálázási funkciókat.

* Az **Azure Functions** abban különbözik a korábbi számítási lehetőségektől, hogy nem kell hozzá automatikus skálázási szabályokat konfigurálni. Ehelyett az Azure Functions automatikusan lefoglalja a számítási erőforrásokat a kód futásakor, és szükség szerint végzi el a horizontális felskálázást a terhelés kezeléséhez. További információ: [A megfelelő Azure Functions szolgáltatási csomag kiválasztása][functions-scale].

Végül egyes esetekben egy egyéni automatikus skálázási megoldás használata is hasznos lehet. Például használhat Azure-diagnosztikákat és alkalmazásalapú metrikákat, valamint egyéni kódokat az alkalmazásmetrikák monitorozásához és exportálásához. Ezután a metrikák alapján megadhat egyéni szabályokat, majd a Resource Manager REST API-k segítségével aktiválhatja az automatikus skálázást. Azonban az egyéni megoldásokat nem egyszerű megvalósítani, ezért a használatuk csak abban az esetben ajánlott, ha az előző módszerek egyike sem fel meg a követelményeknek.

Használja a platform beépített automatikus skálázási funkcióit, ha azok megfelelnek a követelményeinek. Ha nem, alaposan gondolja át, hogy valóban szüksége van-e az összetettebb skálázási funkciókra. A további követelmények közé tartozhatnak a részletesebb vezérlés, a méretezést kiváltó események észlelésének különböző módjai, a különböző előfizetéseken keresztüli skálázás és az egyéb típusú erőforrások skálázása.

## <a name="use-azure-monitor-autoscale"></a>Az Azure Monitor automatikus skálázásának használata

Az [Azure Monitor automatikus skálázása][monitoring] közös automatikus skálázási funkciókat biztosít a virtuálisgép-méretezési csoportokhoz, az Azure App Service-szolgáltatásokhoz és az Azure Cloud Service-szolgáltatásokhoz. A skálázás elvégezhető egy ütemezés szerint, vagy futtatókörnyezeti metrikára, például CPU- vagy memóriahasználatra is alapozható. Példák:

- Horizontális felskálázás 10 példányra hétköznapokon és horizontális leskálázás 4 példányra hétvégén. 
- Horizontális felskálázás egy példánnyal, ha az átlagos CPU-használat 70% fölé emelkedik és horizontális leskálázás egy példánnyal, ha a CPU-használat 50% alá esik.
- Horizontális felskálázás egy példánnyal, ha egy üzenetsorban található üzenetek száma meghalad egy adott küszöbértéket.

A beépített metrikák listájáért lásd: [Az Azure Monitor automatikus skálázáshoz használt közös metrikái][autoscale-metrics]. Az Application Insights segítségével megvalósíthat egyéni metrikákat is. 

Az automatikus skálázást a PowerShell, az Azure CLI, egy Azure Resource Manager-sablon vagy az Azure Portal használatával konfigurálhatja. Részletesebb vezérléshez használja az [Azure Resource Manager REST API](https://msdn.microsoft.com//library/azure/dn790568.aspx)-t. Az [Azure Monitoring Service Management Library](https://www.nuget.org/packages/Microsoft.WindowsAzure.Management.Monitoring) és a [Microsoft Insights Library](https://www.nuget.org/packages/Microsoft.Azure.Insights/) (előzetes verzió) olyan SDK-k, amelyek lehetővé teszik a különböző erőforrások metrikáinak begyűjtését és az automatikus skálázás végrehajtását a REST API-k használatával. Az Azure Resource Managert nem támogató erőforrások, illetve az Azure Cloud Services használata esetén a Service Management REST API használható az automatikus skálázáshoz. Minden egyéb esetben használja az Azure Resource Managert.

Az Azure automatikus skálázás használatakor vegye figyelembe a következő szempontokat:

* Vegye figyelembe, hogy az alkalmazás terhelését kellő mértékben előre tudja-e jelezni az ütemezett automatikus skálázás használatához, a példányok hozzáadásához és eltávolításához, illetve az erőforrásigény várható csúcsainak kiszolgálásához. Ha ez nem lehetséges, használjon a futtatókörnyezeti metrikákon alapuló, reaktív automatikus skálázást, amely képes kezelni az erőforrásigény előre nem látható változásait. Általában ezek a módszerek kombinálhatók. Például létrehozhat egy olyan stratégiát, amely az erőforrásokat ütemezés alapján adja hozzá azokban az időpontokban, amikor tudni lehet, hogy az alkalmazás a leginkább le lesz terhelve. Ezzel biztosítható az igény szerinti kapacitás rendelkezésre állása az új példányok elindításából adódó késedelem nélkül. Minden egyes ütemezett szabályhoz megadhatók metrikák, amelyek az adott időszakra lehetővé teszik a reaktív automatikus skálázást annak biztosítása érdekében, hogy az alkalmazás kezelni tudja a tartós, de előre nem látható igénybevételi csúcsokat.
* A metrikák és a kapacitásbeli követelmények közötti kapcsolatot általában nehéz átlátni, különösen egy alkalmazás első üzembe helyezésekor. A legjobb kezdetben kiépíteni egy kis további kapacitást, majd monitorozás alapján az automatikus skálázási szabályok finomhangolásával közelebb állítani a kapacitást a valós terheléshez.
* Konfigurálja az automatikus skálázási szabályokat, majd egy ideig monitorozza az alkalmazás teljesítményét. A monitorozás eredményei alapján beállíthatja, hogy a rendszer szükség esetén hogyan végezze el a skálázást. Azonban vegye figyelembe, hogy az automatikus skálázás nem egy azonnali folyamat. Időbe telik, míg a rendszer reagál az olyan metrikákra, mint például amikor az átlagos CPU-használat meghalad egy bizonyos küszöbértéket (vagy az alá csökken).
* A mért eseményindító attribútumon (például CPU-használaton vagy egy üzenetsor hosszán) alapuló észlelési mechanizmust használó automatikus skálázási szabályok az automatikus skálázási műveleteket nem azonnali értékek, hanem egy adott időszakra vonatkozó összesített érték alapján aktiválják. Alapértelmezés szerint az összesített érték az adott idő alatt mért értékek átlaga. Ez a módszer megakadályozza, hogy a rendszer túl gyorsan reagáljon vagy gyors ingadozást idézzen elő. Emellett időt hagy az automatikusan elindított új példányoknak, hogy beálljanak a futási üzemmódba, és megakadályozza, hogy az új példányok indulása közben további automatikus skálázási műveletek menjenek végbe. Az Azure Cloud Services és az Azure Virtual Machines esetében az alapértelmezett összesítési időszak 45 perc, tehát ennyi időbe telhet, amíg a metrika az igénybevételi csúcsokra válaszul aktivál egy automatikus skálázási műveletet. Módosítsa az összesítési időszak az SDK-val, de vegye figyelembe, hogy a 25 percnél rövidebb időszakok kiszámíthatatlan eredményekhez vezethetnek. A Web Apps esetében az átlagolási időtartam jóval rövidebb, így az új példányok már nagyjából öt perccel a kiváltó átlagolt mérőszám változását követően elérhetők.
* Ha az automatikus skálázást a portál helyett az SDK használatával konfigurálja, akkor részletesebb ütemezést adhat meg az aktív szabályokra vonatkozóan. Emellett létrehozhat saját metrikákat is, és felhasználhatja őket az automatikus skálázási szabályokhoz a meglévő metrikákkal együtt vagy azok nélkül is. Például előfordulhat, hogy alternatív számlálókat szeretne használni, mint a kérelmek másodpercenkénti száma vagy a memória átlagos rendelkezésre állása, esetleg olyan egyéni számlálókat szeretne használni, amelyek bizonyos üzleti folyamatokat mérnek.
* A Service Fabric automatikus skálázásakor a fürtben található csomóponttípusokat a háttérben lévő virtuálisgép-méretezési csoportok alkotják, így az egyes csomóponttípusok mindegyikéhez automatikus skálázási szabályokat kell beállítani. Vegye figyelembe a szükséges csomópontok számát, mielőtt beállítja az automatikus skálázást. Az elsődleges csomóponttípushoz megadott minimálisan szükséges csomópontszámot a kiválasztott megbízhatósági szint határozza meg. További információ: [Service Fabric-fürt fel- vagy leskálázása automatikus skálázási szabályok használatával](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-scale-up-down).
* A portál segítségével erőforrásokat, például SQL Database-példányokat és üzenetsorokat kapcsolhat egy Cloud Service-példányhoz. Ez könnyebben elérhetővé teszi a különböző manuális és automatikus skálázási konfigurációs beállításokat a kapcsolt erőforrások mindegyikéhez. További információ: [Erőforrás összekapcsolása egy felhőalapú szolgáltatással](/azure/cloud-services/cloud-services-how-to-manage).
* Ha több házirend és szabály is konfigurálva van, azok ütközhetnek egymással. Az automatikus skálázás az alábbi ütközésfeloldási szabályok használatával biztosítja, hogy mindig elegendő mennyiségű példány fusson:
  * A horizontális felskálázási műveletek mindig elsőbbséget élveznek a horizontális leskálázási műveletekkel szemben.
  * Ha horizontális felskálázási műveletek között van ütközés, akkor mindig a példányszámok legnagyobb mértékű növelését kezdeményező szabály lép érvénybe.
  * Ha horizontális leskálázási műveletek között van ütközés, akkor mindig a példányszámok legkisebb mértékű csökkentését kezdeményező szabály lép érvénybe.
* Az App Service Environment-környezetekben bármennyi feldolgozókészlet vagy előtérmetrika használható az automatikus skálázási szabályok meghatározásához. További információ: [Automatikus skálázás és az App Service Environment-környezet](/azure/app-service/app-service-environment-auto-scale).

## <a name="application-design-considerations"></a>Alkalmazáskialakítási szempontok
Az automatikus skálázás nem egy azonnali megoldás. Ha egyszerűen hozzáadunk erőforrásokat egy rendszerhez, vagy több példányban futtatunk egy folyamatot, az nem garantálja a rendszer teljesítményének növekedését. Az automatikus skálázási stratégiák tervezésekor vegye figyelembe a következő szempontokat:

* A rendszert úgy kell megtervezni, hogy horizontálisan skálázható legyen. Ne hagyatkozzon feltételezésekre a példányok affinitásával kapcsolatban, és ne tervezzen olyan megoldásokat, amelyeknek az az igénye, hogy a kód mindig egy folyamat adott példányán fusson. Egy felhőszolgáltatás vagy webhely horizontális skálázásakor nem szabad azt feltételezni, hogy az azonos forrásból érkező kérelmek mindig ugyanahhoz a példányhoz kerülnek. Ugyanezen okból a szolgáltatásokat állapot nélkülire kell tervezni, hogy az egy adott alkalmazástól érkező kéréseknek ne kelljen mindig a szolgáltatás ugyanazon példányához kerülnie. Amikor egy olyan szolgáltatást tervez meg, amely egy üzenetsor üzeneteit olvassa be és dolgozza fel, akkor ne hagyatkozzon feltételezésekre azt illetően, hogy egy adott üzenetet a szolgáltatás mely példánya fog kezelni. Az automatikus skálázással a szolgáltatás további példányai indíthatók el az üzenetsor hosszának növekedésével párhuzamosan. E forgatókönyv kezeléséről a [versengő felhasználókat ismertető minta][competing-consumers] tartalmaz további információt.
* Ha a megoldás egy hosszan futó feladatot valósít meg, akkor a feladatot úgy kell megtervezni, hogy a horizontális fel- és leskálázást is támogassa. Megfelelő odafigyelés nélkül a feladatok akadályozhatják azt, hogy a folyamatpéldányok megfelelően leálljanak a rendszer leskálázásakor, vagy adatvesztést okozhatnak a folyamat kényszerített leállításakor. Ideális esetben érdemes a hosszan futó feladatot újrabontani, és az általa végzett feldolgozást kisebb, különálló darabokra osztani. Ennek megvalósításával kapcsolatban a [csöveket és szűrőket ismertető mint][pipes-and-filters] tartalmaz további információt.
* További megoldásként megvalósíthat egy ellenőrzőpontos mechanizmust, amely rendszeres időközönként rögzíti a feladat állapotadatait és elmenti azokat egy tartós tárolóban, amelyhez a feladatot futtató folyamat bármely példánya hozzáférhet. Így a folyamat leállásakor az általa végzett munkát egy másik példány a legutóbbi ellenőrzőponttól tudja folytatni.
* Amikor háttérfeladatok külön számítási példányokon futnak, például egy felhőszolgáltatásban futó alkalmazás különböző feldolgozói szerepkörei esetében, akkor szükség lehet az alkalmazás különböző részeinek különböző skálázási szabályzattal történő skálázására. Például szükséges lehet a felhasználói felületek (UI) további számítási példányainak üzembe helyezése a háttérbeli számítási példányok számának növelése nélkül, vagy ennek a fordítottja. Ha több különböző szolgáltatási szintet is nyújt (például alapszintű és prémium szolgáltatáscsomagokat), akkor a prémium szolgáltatáscsomagok a számítási erőforrások agresszívabb horizontális felskálázását igényelhetik az SLA betartása érdekében, mint az alapszintű szolgáltatáscsomagok.
* Érdemes lehet az automatikus skálázási stratégia feltételeként annak az üzenetsornak a hosszát használni, amelyben a felhasználói felületi és a háttérbeli számítási példányok kommunikálnak. Ez jelzi a legpontosabban az aktuális terhelés és a háttérfeladat feldolgozási kapacitása közti egyenetlenséget vagy különbséget.
* Ha az automatikus skálázási stratégia alapja egy üzleti folyamatokhoz kapcsolódó mérőszám, amely például az óránként leadott megrendelések számát vagy egy összetett tranzakció átlagos végrehajtási idejét méri, akkor mindenképpen legyen tisztában azzal, hogy milyen összefüggés van az ilyen típusú mérőszámok eredményei és a tényleges számítási kapacitásigények között. Elképzelhető, hogy az üzleti folyamatok mérőszámainak változására válaszul egynél több összetevőt vagy számítási egységet is skálázni kell.  
* Annak érdekében, hogy a rendszer ne próbálkozzon túlzott mértékű horizontális felskálázással, illetve a több ezer példány futtatásából adódó költségek elkerüléséhez érdemes korlátozni az automatikusan hozzáadható példányok maximális számát. A legtöbb automatikus skálázási mechanizmus lehetővé teszi az egyes szabályokhoz tartozó minimális és maximális példányszámok megadását. Emellett érdemes lehet megfontolni a rendszer által nyújtott funkciók zökkenőmentes csökkentését abban az esetben, ha az üzembe helyezett példányok száma elérte a maximális értéket, és a rendszer még mindig túl van terhelve.
* Vegye figyelembe, hogy az automatikus skálázás nem feltétlenül a legmegfelelőbb mechanizmus a terhelés hirtelen növekedésének kezeléséhez. Az új szolgáltatáspéldányok kiépítése és indítása, illetve az erőforrások hozzáadása időigényes művelet, és lehet, hogy mire a további erőforrások elérhetővé válnak, addigra az igénycsúcs már el is múlt. Ebben az esetben érdemesebb lehet a szolgáltatás szabályozását választani. Erről a [szabályozási minta][throttling] tartalmaz további információt.
* Ezzel szemben amikor az erőforrásigény hirtelen ingadozásakor szükség van az összes kérelem feldolgozásához elegendő kapacitásra, és a költségek nem jelentenek elsődleges szempontot, akkor érdemes egy agresszív automatikus skálázási stratégiát használni, amely gyorsabban elindítja a további példányokat. Használhat egy ütemezett szabályzatot is, amely a várt maximális terhelés kezeléséhez elegendő számú példányt indít el még a terhelés várt időpontja előtt.
* Az automatikus skálázási mechanizmusnak monitoroznia kell az automatikus skálázási folyamatot, és naplóznia kell az összes automatikus skálázási esemény részleteit (az eseményindítót, illetve hogy mely erőforrások lettek hozzáadva vagy eltávolítva és mikor). Az egyéni automatikus skálázási mechanizmusok létrehozásakor ügyeljen arra, hogy a mechanizmus tartalmazza ezt a funkciót. Elemezze az adatokat, és mérje fel az automatikus skálázási stratégia hatékonyságát, majd szükség esetén finomhangolja a stratégiát. A hangolást elvégezheti rövid távon, a használati minták világosabbá válásával párhuzamosan, vagy hosszú távon, az üzleti bővülését vagy az alkalmazás követelményeinek változását követve. Ha egy alkalmazás eléri az automatikus skálázáshoz megadott felső korlátot, akkor a mechanizmus riasztást küldhet az operátornak, aki szükség esetén manuálisan indíthat el további erőforrásokat. Vegye figyelembe, hogy ilyen körülmények között az operátor felelős lehet az erőforrások manuális eltávolításáért is, miután az erőforrásigény csökkent.

## <a name="related-patterns-and-guidance"></a>Kapcsolódó minták és útmutatók
A következő minták és útmutatók szintén hasznosak lehetnek az automatikus skálázás bevezetésének esetén:

* [Szabályozási minta][throttling]. Ez a minta azt ismerteti, hogy egy alkalmazás hogyan működhet tovább és hogyan teljesítheti az SLA-k feltételeit, amikor az erőforrásigény növekedése rendkívüli módon leterheli az erőforrásokat. A szabályozás és az automatikus skálázás együttes használatával megelőzhető a rendszer túlterhelődése a felskálázás közben.
* [Versengő felhasználókat ismertető minta][competing-consumers]. Ez a minta azt ismerteti, hogy hogyan helyezhető üzembe egy szolgáltatáspéldány-készlet, amely képes bármely alkalmazáspéldány üzeneteit kezelni. Az automatikus skálázás használatával a szolgáltatáspéldányok a várt terhelésnek megfelelően indíthatók el és állíthatók le. Ez a megközelítés lehetővé teszi, hogy a rendszer több üzenetet dolgozzon fel párhuzamosan a teljesítmény optimalizálása, a skálázhatóság és rendelkezésre állás javítása és a terhelés elosztása érdekében.
* [Monitorozás és diagnosztika](./monitoring.md). A kialakítás és a telemetria kulcsfontosságú szerepet játszanak az automatikus skálázást vezérlő adatok összegyűjtésében.


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
