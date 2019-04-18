---
title: A megbízhatóság az Azure-beli alkalmazások állapotának monitorozása
description: Figyelés használata alkalmazások megbízhatóságát, az Azure-ban
author: MikeWasson
ms.date: 04/10/2019
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-design-principles
ms.openlocfilehash: 46f9f3fb623a2facf4b9f9c6ec57376ae59e41dc
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/17/2019
ms.locfileid: "59646569"
---
# <a name="monitoring-application-health-for-reliability-in-azure"></a>A megbízhatóság az Azure-beli alkalmazások állapotának monitorozása

A megfigyelés és a diagnosztika létfontosságú a rugalmasság szempontjából. Ha hiba lép fel, akkor ismernie kell *, amely* sikertelen volt, *amikor* nem sikerült &mdash; és *miért*.

*Figyelés* nem ugyanaz, mint *hibaészlelés*. Hogy az alkalmazás például észlelhet átmeneti hibát, majd próbálkozzon újra, állásidő elkerülése. De naplóznia is kell a próbálja megismételni a műveletet úgy, hogy figyelemmel kísérheti a hibák aránya az átfogó képet az alkalmazás állapotáról.

Úgy gondolja, hogy a monitorozási és diagnosztikai folyamat mint négy különböző szakaszból rendelkező folyamatot: Kialakítási, gyűjtemény és a tárolási, elemzési és elemzés céljából, és a Vizualizáció és riasztások.

## <a name="instrumentation"></a>Rendszerállapot

Nem célszerű, monitorozhatók az alkalmazások közvetlenül, így kialakítási kulcsot. Virtuális gépeken (VM), amely hozzáadása és eltávolítása idővel több tucatnyi nagy méretű elosztott rendszerek előfordulhat, hogy futtassa. Hasonlóképpen felhőalapú alkalmazások előfordulhat, hogy használjon adattárak számos, és egyetlen felhasználói művelet előfordulhat, hogy több alrendszerre is kiterjedhet.

Adja meg a részletes rendszerállapot:

- Hibák az valószínűleg, de nem rendelkezik még történt, hogy a hiba okának megállapításához, csökkentheti a helyzet elegendő adatot biztosítson, és győződjön meg arról, hogy a rendszer továbbra is elérhető.
- Már bekövetkezett hibák az alkalmazás a felhasználó a megfelelő hibaüzenetet adja vissza, de érdemes próbálja folytatni, fut, jóllehet az korlátozott funkciókkal.

Megfigyelő rendszere rögzíteni kell átfogó részleteit, hogy az alkalmazások hatékonyan tudja állítani, és ha szükséges, tervezők és fejlesztők módosíthatja a rendszer ismétlődő az a helyzet elkerülése érdekében.

Figyelés a nyers adatok különböző forrásokból, beleértve a származhatnak:

- Alkalmazásnaplók, mint például amelyek által előállított a [Azure Application Insights](/azure/azure-monitor/app/app-insights-overview) szolgáltatás.
- Teljesítmény-mérőszámok operációs rendszer által gyűjtött [Azure monitorozási ügynökök](/azure/azure-monitor/platform/agents-overview).
- [Azure-erőforrások](/azure/azure-monitor/platform/metrics-supported), az Azure Monitor által gyűjtött metrikák többek között.
- [Az Azure Service Health](/azure/service-health/service-health-overview), amely aktív események nyomon követéséhez irányítópult kínál.
- [Az Azure AD-naplók](/azure/active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics) az Azure platformba beépítve.

A legtöbb Azure-szolgáltatások rendelkezik mérőszámai és diagnosztikái konfigurálható elemzésére és a probléma okának megállapításához. További tudnivalókért lásd: [figyelési adatokat gyűjtött az Azure Monitor](/azure/azure-monitor/platform/data-collection).

## <a name="collection-and-storage"></a>Gyűjtés és tárolás

Különböző helyeken és formátumokban, beleértve a nyers rendszerállapot-adatok tárolhatók:

- Alkalmazások nyomkövetési naplói
- IIS-naplók
- Teljesítményszámlálók

E különálló forrásokat gyűjtött, konszolidált, és az Azure-ban, mint például az Application Insights, az Azure Monitor metrikák, a Service Health, tárfiókok és Azure Log Analytics megbízható adattárak helyezi el.

## <a name="analysis-and-diagnosis"></a>Elemzés és diagnosztika

Ezek adattárak problémák elhárításához és az alkalmazás állapotának átfogó képet kaphat az összesített adatok elemzéséhez. Általában a is [keresse meg és elemezheti](/azure/azure-monitor/log-query/log-query-overview) az Application Insights és a Log Analytics használatával Kusto-lekérdezés és a nézet az előre konfigurált ábrák használatával az adatok [felügyeleti megoldások](/azure/azure-monitor/insights/solutions-inventory). Vagy egy az aktuális javaslatok megtekintése az Azure Advisor használatával [rugalmasság](/azure/advisor/advisor-high-availability-recommendations) és [teljesítmény](/azure/advisor/advisor-performance-recommendations).

## <a name="visualization-and-alerts"></a>Megjelenítés és riasztások

Jelen a telemetriai adatokat, amely megkönnyíti az olyan operátorra, figyelje meg, hogy problémákat vagy tendenciákat, például egy irányítópulton vagy e-mail-értesítés formátuma.

Alkalmazásállapot teljes képet kaphat a [Azure-irányítópultok](/azure/azure-portal/azure-portal-dashboards) nézetet jelenít meg a figyelés az Application Insights, a Log Analytics, az Azure Monitor-metrikák és a Service Health gráfok létrehozásához. És [Azure Monitor riasztások](/azure/azure-monitor/platform/alerts-overview) értesítések létrehozása a Service Health, a resource health, az Azure Monitor-metrikák, a rendszer a Log Analytics és az Application Insights.

Figyelési és diagnosztikai funkciókkal kapcsolatos további információkért lásd: [Monitorozási és diagnosztikai](../best-practices/monitoring.md).

## <a name="health-probes-and-check-functions"></a>Állapot-mintavételei és ellenőrzési funkciók

Az állapotát és a egy alkalmazás teljesítményének idővel romoljon, és a teljesítménycsökkenés lehet, hogy nem észlelhető mindaddig, amíg az alkalmazás nem tudja.

Mintavételezők megvalósítása vagy ellenőrizze a functions, és az alkalmazáson kívüli rendszeresen a futtathatók. Az ellenőrzések egyszerű: az alkalmazás egészére, az alkalmazás egyes részei, az alkalmazás által használt egyes szolgáltatások számára, vagy külön összetevők válaszidő mérése is lehet.

Ellenőrizze a functions futtathatja a folyamatokat, így biztosíthatja, hogy azok érvényes eredményeket, mérheti a késés és rendelkezésre állásának ellenőrzése és információk nyerhetők a rendszer.

## <a name="long-running-workflow-failures"></a>Hosszan futó munkafolyamatok hibája

Hosszan futó munkafolyamatok tartozhat több lépést, amelyek mindegyike függetlennek kell lenniük.

Hosszú futású folyamatok, azzal csökkenthető a, hogy a teljes munkafolyamatot kell vissza lesz állítva, illetve, hogy több kompenzáló tranzakciót kell hajtható végre, az előrehaladását úgy követheti nyomon.

>[!TIP]
> Figyelheti és kezelheti a hosszan futó munkafolyamatok előrehaladását egy minta megvalósításával [Feladatütemező ügynök felügyeleti](../patterns/scheduler-agent-supervisor.md).

## <a name="application-logs"></a>Alkalmazásnaplók

Az alkalmazásnaplók a diagnosztikai adatok fontos forrását képezik. Betekintést nyerhet szükség esetén a legtöbb, kövesse az ajánlott eljárások az alkalmazásnaplózáshoz.

### <a name="log-data-in-the-production-environment"></a>Naplóadatok az éles környezetben

Robusztus telemetriai adatok rögzítése, amíg az alkalmazás az éles környezetben fut, így diagnosztizálhatja a problémákat a termelési állapotban okának elegendő információt kell.

### <a name="log-events-at-service-boundaries"></a>Alkalmazásnapló-események szolgáltatáshatárokon

Használjon korrelációs azonosítót, amely átnyúlik a szolgáltatáshatárokon. Ha egy tranzakció több szolgáltatást és az egyik keresztüláramló meghiúsul, a korrelációs azonosító segítségével nyomon követheti a kéréseket az alkalmazás- és miért nem sikerült a tranzakció pinpoints között.

### <a name="use-semantic-structured-logging"></a>Szemantikai (strukturált) naplózás

Strukturált naplók és különösen fontos felhőszinten naplóadatok elemzésének automatizálását könnyebbé válik. Általában javasoljuk, hogy Azure-erőforrások metrikák és diagnosztikai adatok tárolására, a Log Analytics-munkaterületen, nem pedig a storage-fiókban. Ezzel a módszerrel Kusto-lekérdezés használhatja a gyors és strukturált formátumban szeretne adatokat lekérdezni. Az Azure Monitor API-k és az Azure Log Analytics API-k is használhatja.

### <a name="use-asynchronous-logging"></a>Használjon aszinkron naplózást

Szinkron naplózási műveletek néha letiltása az alkalmazás kódjában, biztonsági mentése, a naplók írt kérelmeket okozza. Használjon aszinkron naplózást alkalmazásnaplózás során rendelkezésre állás megőrzéséhez.

### <a name="separate-application-logging-from-auditing"></a>A naplózás külön alkalmazásnaplózás

Naplórekordok gyakran karbantartása megfelelőségi vagy jogszabályi követelményeknek, és be kell fejeződnie. Eldobott tranzakciók elkerülése érdekében külön, a diagnosztikai naplókat a naplók kezelése.

## <a name="remote-call-statistics"></a>Távoli hívás statisztika

Követése és jelentése távoli valós idejű statisztikák hívja, és adjon meg egy egyszerű módja annak, hogy tekintse át ezt az információt, így az üzemeltetési csapat egy azonnali betekintést az alkalmazás állapotát. Távoli hívás mérőszámokat, például a késés, átviteli sebesség és a 99 és a 95. percentilisei hibáinak összegzése.

## <a name="transient-exceptions-and-retries"></a>Átmeneti kivételek és az újrapróbálkozások

Nyomon követheti az átmeneti kivételek és az újrapróbálkozások számát elvégzésével nyújt betekintést a problémák és hibák az alkalmazás újrapróbálkozási logikát az idő függvényében. Kivételek növekszik az idő múlásával trendjének arra utalhat, hogy a szolgáltatás, hogy a probléma, és sikertelen lehet. További információkért lásd: [az újrapróbálkozási mechanizmushoz adott](../best-practices/retry-service-specific.md).

## <a name="early-warning-system"></a>Korai figyelmeztető rendszert

Figyelje az alkalmazása olyan figyelmeztető jeleit, amelyek proaktív beavatkozást igényelnek. Felmérheti az általános állapotát, az alkalmazást és annak függőségeit, eszközöket segítenek gyorsan felismerni, amikor a rendszer vagy összetevőinek hirtelen elérhetetlenné válik. Ezek segítségével megvalósítani egy korai figyelmeztető rendszert.

1. Az alkalmazás állapotának, például átmeneti kivételeket és a távoli hívás késés a fő teljesítménymutatók azonosításához.
1. Állítsa be a küszöbértékeket, hogy az azonosíthatja a problémákat, mielőtt válik a kritikus fontosságú, és a egy helyreállítási válasz igényelnek.
1. Riasztást küld a műveletek a küszöbérték elérésekor.

Érdemes lehet [Microsoft System Center 2016](https://www.microsoft.com/cloud-platform/system-center) vagy harmadik felektől származó eszközök figyelési képességeket biztosít. A legtöbb figyelési megoldások nyomon követheti a kulcsfontosságú teljesítményszámlálók és szolgáltatás rendelkezésre állása. [Az Azure resource health](/azure/service-health/resource-health-checks-resource-types) tartalmaz néhány beépített állapot állapot-ellenőrzések, amely megkönnyíti a szabályozás, az Azure-szolgáltatások diagnosztizálása.

## <a name="subscription-and-service-limitations"></a>Előfizetés és a szolgáltatás korlátozásai

Az Azure-előfizetések korlátokkal rendelkeznek az egyes erőforrástípusok, például az erőforráscsoportok, a magok és a storage-fiókok száma. Győződjön meg arról, hogy az alkalmazás nem fut a személyzetet Azure-előfizetés korlátai, hozzon létre riasztásokat, amelyek a szolgáltatás hamarosan megszűnik a korlátok és kvóták lekérdezéséhez.

Oldja meg a riasztások a következő előfizetési korlátozásait.

### <a name="individual-services"></a>Egyes szolgáltatások

Egyes Azure-szolgáltatások a storage, átviteli sebesség, kapcsolatok, a kérések száma másodpercenként és egyéb mérőszámok száma használati korlátokkal rendelkeznek. Az alkalmazás sikertelen lesz, ha megkísérli a erőforrások túli ezeket a korlátokat, szolgáltatás-szabályozási és esetleges leállást eredményez.

Az adott szolgáltatás és a az alkalmazás követelményeitől függően gyakran maradhat, ha ezek a korlátok alapján (egy másik tarifacsomagra, például kiválasztása) vertikális vagy horizontális felskálázás (új erőforráspéldányok hozzáadását jelenti).

### <a name="azure-storage-scalability-and-performance-targets"></a>Az Azure storage méretezhetőségi és teljesítménycéljai

Az Azure lehetővé teszi, hogy előfizetésenként storage-fiókok maximális számát. Ha az alkalmazás további tárfiókok, mint amennyi az előfizetésben jelenleg elérhető igényel, további tárfiókok hozzon létre egy új előfizetést. További információkért lásd: [Azure-előfizetés és a szolgáltatások korlátozásai, kvótái és megkötései](/azure/azure-subscription-service-limits/#storage-limits).

Ha az Azure storage méretezhetőségi és teljesítménycéljai túllépi, az alkalmazás storage szabályozás fog tapasztalni. További információkért lásd: [Azure Storage méretezhetőségi és teljesítménycéljai](/azure/storage/storage-scalability-targets/).

### <a name="scalability-targets-for-virtual-machine-disks"></a>Virtuálisgép-lemezek skálázási célértékei

Az Azure infrastruktúra-szolgáltatás (IaaS) virtuális gép egy több adatlemez, attól függően számos tényező befolyásolja, többek között a virtuális gép méretétől és a tárfiók típusát csatolását támogatja. Ha az alkalmazás meghaladja a virtuálisgép-lemezek skálázási célértékei, további tárfiókok üzembe, és ott a virtuálisgép-lemezek létrehozása. További információkért lásd: [Azure Storage méretezhetőségi és teljesítménycéljai](/azure/storage/storage-scalability-targets/#scalability-targets-for-virtual-machine-disks).

### <a name="vm-size"></a>Virtuális gép mérete

A tényleges CPU, memória, lemez és a virtuális gépek i/o megközelítés korlátairól a Virtuálisgép-méretet, ha az alkalmazás felhasználói problémákat tapasztalhatnak a kapacitást. Javítsa ki a hibákat, növelje meg a Virtuálisgép-méretet. Virtuálisgép-méretek ismertetett [az Azure-beli virtuális gépek méretei](/azure/virtual-machines/virtual-machines-windows-sizes/?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

Ha a munkaterhelés időbeli ingadozik, fontolja meg a virtuálisgép-méretezési csoport használatával beállítja a Virtuálisgép-példányok számának automatikus skálázása. Ellenkező esetben kell manuálisan növelheti vagy csökkentheti a virtuális gépek számát. További információkért lásd: a [virtuálisgép-méretezési csoportok – áttekintés](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview/).

### <a name="azure-sql-database"></a>Azure SQL Database

Ha az Azure SQL Database-szintre nem megfelelő az alkalmazás-adatbázis tranzakciós egységek (DTU) megbízhatóan kezelni, az adatok használata szabályozva lesz. A megfelelő service-csomag kiválasztásával további információkért lásd: [vásárlási modellek az Azure SQL Database](/azure/sql-database/sql-database-service-tiers/).

## <a name="third-party-services"></a>Külső szolgáltatások

Ha az alkalmazás függőségeit a harmadik féltől származó szolgáltatásokkal, hogyan meghiúsulhat, ezek a szolgáltatások, és milyen hatása sikertelen lesz az alkalmazás a azonosítása.

Egy külső szolgáltatás nem tartalmazhat, monitorozásához és diagnosztizálásához. Jelentkezzen be a hívások ezeket a szolgáltatásokat, és összefüggésbe hozva azokat az alkalmazás állapotát és a egy egyedi azonosító segítségével diagnosztikai naplózás az. A monitorozási és diagnosztikai bevált gyakorlatokat további információkért lásd: [figyelési és diagnosztikai útmutató](../best-practices/monitoring.md).

## <a name="next-steps"></a>További lépések

> [!div class="nextstepaction"]
> [Vész-helyreállítási terv](./disaster-recovery.md)
