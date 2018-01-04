---
title: "Naplózás és figyelés a mikroszolgáltatások"
description: "Naplózás és figyelés a mikroszolgáltatások"
author: MikeWasson
ms.date: 12/08/2017
ms.openlocfilehash: 1da67047daa9ae87cda5dd7dd581d6081183c428
ms.sourcegitcommit: 786bafefc731245414c3c1510fc21027afe303dc
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/12/2017
---
# <a name="designing-microservices-logging-and-monitoring"></a>Mikroszolgáltatások tervezése: naplózás és figyelés

Bármely összetett alkalmazásban bármikor valami fog hiba lép fel. Egy mikroszolgáltatások alkalmazásban kell nyomon követnie, több tucatnyi vagy akár több száz szolgáltatások történéseiről. Naplózás és figyelés különösen fontos, hogy a rendszer átfogó képet biztosítson. 

![](./images/monitoring.png)

Mikroszolgáltatások architektúra, akkor különösen kihívást jelenthet rögzítési ponthoz hibák vagy a teljesítmény szűk pontos okát. Egyetlen felhasználót span előfordulhat, hogy több szolgáltatásra. Szolgáltatások előfordulhat, hogy elérte a hálózati i/o-korlátok a fürtben található. Szolgáltatások közötti hívások láncolata ellennyomás okozhat a rendszer, ami nagy késleltetésű vagy egymásra épülő hibák. Továbbá általában nem tudja melyik csomópontján egy adott konténerben fog futni. Ugyanazon a csomóponton elhelyezett tárolók is lehet használják, a korlátozott CPU és memória. 

Győződjön meg arról, mi történik az értelemben, hogy az alkalmazás kell kibocsátás telemetriai események. Ezek a metrikák és a szöveges naplók rendezheti. 

*Metrikák* numerikus érték, amely elemzése. Használhatja őket megfigyelheti, a rendszer a valós idejű (vagy a közel valós időben), illetve időbeli teljesítménytrendek elemzéséhez. Mérőszámok közé tartozik:

- Csomópont-szintű rendszer metrika, beleértve a CPU, a memória, a hálózati, a lemez és a rendszer használata. Rendszer mérőszámok segítenek megérteni a fürt minden csomópontja erőforrás-elosztás és hibáinak elhárításában kiugró.
 
- Kubernetes metrikákat. Szolgáltatások futnak-tárolókban, mert kell gyűjteni a processzorhasználatról, a tároló szintjén, nem csak a virtuális gép szinten. A Kubernetes cAdvisor (tároló Advisor) az ügynök által összegyűjtött statisztikai adatok a CPU, a memória, a fájlrendszer és a használt hálózati erőforrások minden egyes tároló. A kubelet démon cAdvisor erőforrás statisztikákat gyűjt, és elérhetővé teszi azokat a REST API-n keresztül.
   
- Alkalmazás metrikákat. Ez magában foglalja a metrikákat, amely kapcsolódik a szolgáltatás működésének megértése. Például a várólistán lévő bejövő HTTP-kérelmekre, a kérés várakozási ideje, az üzenet-várólista hossza vagy a másodpercenként feldolgozott tranzakciók száma száma.

- Függő szolgáltatások metrikákat. A fürtben található szolgáltatások kérhet például kezelt PaaS-szolgáltatások, a fürtön kívüli külső szolgáltatásokat. Az Azure-szolgáltatásokat is figyelhet használatával [Azure figyelő](/azure/monitoring-and-diagnostics/monitoring-overview). Harmadik féltől származó szolgáltatással is, vagy nem rendelkezhetnek bármely metrikákat. Ha nem, akkor konfigurálnia kell a saját alkalmazás metrikák késleltetés és a Hibaarány statisztikáinak nyomon követéséhez támaszkodnak.

*Naplók* bejegyzései az alkalmazás futása közben bekövetkező eseményekről. Ezek közé tartozik többek között a alkalmazásnaplók (nyomkövetési utasítások) vagy a webkiszolgáló naplóinak. Naplók esetében elsősorban akkor hasznos, a vizsgálatokhoz és a legfelső szintű alapprobléma elemzéséhez. 

## <a name="considerations"></a>Megfontolandó szempontok

A cikk [megfigyelési és diagnosztikai](../best-practices/monitoring.md) egy alkalmazás általános bevált gyakorlat ismertetése. A következőkben adott mikroszolgáltatások architektúrájának környezetében gondolniuk.

**Konfigurációs és felügyeleti**. A naplózás és figyelés felügyelt szolgáltatást használ, vagy hoz naplózásának és figyelésének összetevők a fürtben található tárolóként telepíteni? Ezek a beállítások több tárgyalását című témakör [technológia beállítások](#technology-options) alatt.

**Adatfeldolgozást arány**. Mi az az átviteli sebesség, amellyel a rendszer fogadására képes, telemetriai események? Mi történik, ha adott aránya túllépi? Például a rendszer lehetséges, hogy sávszélesség-szabályozási ügyfelek, ebben az esetben telemetriai adatok nem vesztek el, vagy előfordulhat, hogy az adatok felbontásának. Ez a probléma néha csökkentheti a begyűjtött adatok mennyiségét csökkenti:

  - Metrikák összesítése statisztikákat, pl.: átlagos és a szórást, kiszámításával és statisztikai adatokat küldeni a felügyeleti rendszer.  

  - Az adatok felbontásának &mdash; Ez azt jelenti, hogy az csak az események százalékát feldolgozni.

  - Kötegelt az adatokat az állapotfigyelő szolgáltatás hálózati hívások számának csökkentése érdekében.

**Költség**. Lehet, hogy magas, különösen a nagy mennyiségük választásával dolgozhat fel, és tárolja a telemetriai adatokat. Bizonyos esetekben sikerült is lehet az alkalmazás futtatásának. Ebben az esetben előfordulhat, hogy csökkenteni kell a telemetriai adatok mennyiségének összesítése, egyszerűsítés, vagy az adatok kötegelés fent leírt módon. 
        
**Adatok rögzített**. Hogyan pontos vannak a metrikák? Átlagok kiugró, főleg nagyobb méretezésnél elrejthetők. Is ha a mintavételi ráta értéke túl alacsony, azt is simítja ingadozásai az adatokat. Tűnhet, hogy a összes kérelem kapcsolatos végpont ugyanolyan késéssel, lehet, ha ténylegesen végzése sokkal hosszabb kérelmek jelentős része. 

**Késés**. Engedélyezi a valós idejű figyelés és a riasztások, telemetriai adatokat kell biztosítani gyorsan. Hogyan "valós idejű" van a figyelési irányítópulton megjelenő adatokat? Néhány másodperc régi? Tovább egy percnél?

**Tárolás.** A naplókhoz lehet leghatékonyabb írni a naplóeseményeket a fürt rövid élettartamú tárhelyre, és küldje el a naplófájlok több állandó tároló az ügynök konfigurálása.  Adatok végül kívánja helyezni a hosszú távú, hogy az utólagos elemzés érhető el. Egy mikroszolgáltatások architektúra hozhat létre a telemetriai adatok nagy mennyiségű, így az adott adattárolás díja fontos szempont. Is gondolja át, hogyan fogja kérdezni az adatokat. 

**Irányítópult és a képi megjelenítés.** Kapunk egy átfogó képet, a rendszer az összes szolgáltatását, mind a fürt és a külső szolgáltatások között? Ha telemetriai adatok és a naplók egynél több helyre, az irányítópult megjelenítése az összes is összefüggéseket? A figyelési irányítópult megjelenítése kell legalább a következő információkat:

- Általános erőforrás-elosztás kapacitás és a növekedési. Ez magában foglalja a tárolók, fájl rendszer metrikákat, hálózati és core foglalási száma.
- Tároló metrikák korrelált a szolgáltatás szintjén.
- Rendszer metrikák tárolók tartozzanak.
- Hibákat és kiugró.
    

## <a name="distributed-tracing"></a>Elosztott nyomkövetése

Ahogy azt korábban említettük, az egyik kihívás a mikroszolgáltatások események áramló van szolgáltatásban ismertetése. Egy egyetlen művelet, vagy a tranzakció magában foglalhatja hívások vonatkozik több szolgáltatásra. Hozza létre újból a teljes feladatütemezési lépéseket, hogy egyes szolgáltatások propagálása kell egy *korrelációs azonosító* , amely úgy működik, mint ennek a műveletnek egyedi azonosítója. A korrelációs azonosító lehetővé teszi, hogy [nyomkövetés elosztott](http://microservices.io/patterns/observability/distributed-tracing.html) szolgáltatásban.

Az első szolgáltatás, amely fogad egy ügyfél kérelmet kell létrehozni a korrelációs azonosítója. Ha a szolgáltatás hívást egy HTTP egy másik szolgáltatás, a korrelációs azonosító fejléc az helyezi el. Hasonlóképpen ha a szolgáltatás aszinkron üzenetet küld, helyezi el a korrelációs azonosító az üzenetbe. Alárendelt szolgáltatásokkal továbbra is a korrelációs azonosító propagálására, hogy azt a teljes rendszer áthaladó. Emellett minden kódot írja az alkalmazás metrikák vagy a napló események tartalmaznia kell a korrelációs azonosítója.

Hívások közötti kapcsolatot, ha például a végpontok közötti késés a teljes tranzakció, a sikeres tranzakciók másodpercenkénti száma és a sikertelen tranzakciók százaléka működési metrikák is kiszámíthatja. Többek között a korrelációs azonosító alkalmazásnaplók lehetővé teszi kiváltó okának elemzése végrehajtásához. Egy művelet meghiúsul, ha a log utasítások található összes a szolgáltatáshoz intézett hívások szereplő ugyanazt a műveletet. 

Elosztott nyomkövetés végrehajtása során az alábbiakban néhány szempontok:

- Jelenleg nincs korrelációs azonosító szabványos HTTP-fejléc. A csoport értéket meg kell szabványosítására. A választott határozzák meg a naplózási/figyelési keretrendszer vagy a választott szolgáltatás háló.

- Aszinkron üzenetek Ha az üzenetkezelési infrastruktúra támogatja az üzenetek, hozzáadását metaadatok kell foglalni a korrelációs azonosító, a metaadatok. Ellenkező esetben adja hozzá az üzenet-séma részeként.

- Ahelyett, hogy egyetlen átlátszatlan azonosítót, előfordulhat, hogy küldjön egy *korrelációs környezet* , amely részletesebb információkat, például a hívó által hívott kapcsolatok tartalmazza. 

- Az Azure Application Insights SDK automatikusan be HTTP-fejlécek korrelációs környezet esetében, és az Application Insights-logs magában foglalja a korrelációs azonosító. Ha úgy dönt, hogy az Application Insights épített korrelációs használatát, bizonyos szolgáltatások lehet szükség a korrelációs fejlécek, attól függően, hogy a használt könyvtárak explicit módon terjesztése. További információkért lásd: [az Application Insights Telemetria korrelációs](/azure/application-insights/application-insights-correlation).
   
- Ha használunk szolgáltatás rácsvonal Istio vagy linkerd, ezek a technológiák automatikusan generáljon korrelációs fejlécek HTTP-hívások révén a szolgáltatás háló proxyk legyenek átirányítva. Szolgáltatások továbbítsa a megfelelő fejléceket. 

    - Istio: [elosztott kérelmek nyomkövetése](https://istio-releases.github.io/v0.1/docs/tasks/zipkin-tracing.html)
    
    - linkerd: [környezeti fejléceket](https://linkerd.io/config/1.3.0/linkerd/index.html#http-headers)
    
- Vegye figyelembe, hogyan fogja naplók összevonásakor. Érdemes lehet a naplókban korrelációs azonosító hozzáadásának a módját a csapatok szabványosítására. Strukturált vagy félig strukturált formátumban, például a JSON-NÁ, és adja meg egy közös mező ahhoz, hogy a korrelációs azonosítója.

## <a name="technology-options"></a>Technológia beállítások

**Az Application Insights** egy felügyelt szolgáltatás, amely ingests és tárolja a telemetriai adatokat, és elemzésével, és az adatok kereséséhez eszközöket biztosít az Azure-ban. Az Application Insights használatához egy instrumentation csomag az alkalmazás telepítése. Ez a csomag az alkalmazás figyeli, és telemetriai adatokat küld az Application Insights szolgáltatással. Azt is le, hogy a gazdagép-környezetben telemetriai adatait. Az Application Insights biztosít, beépített korrelációs és függőségi nyomon követését. Lehetővé teszi a rendszer metrikákat, alkalmazás metrikák és az Azure szolgáltatás metrika, egyetlen helyen követik.

Vegye figyelembe, hogy az Application Insights azelőtt gyorsítja fel, ha a adatok meghaladja a maximálisan megengedettet; További információkért lásd: [korlátozza az Application Insights](/azure/azure-subscription-service-limits#application-insights-limits). Egyetlen művelettel hozhat létre több telemetriai események, ezért, ha az alkalmazás a forgalom nagy mennyiségű valószínű halmozódni beolvasása. Ez a probléma mérséklése érdekében a telemetriai adatok forgalom csökkentése érdekében mintavételi végezheti el. Mi a fontosabb: az, hogy a metrikákat kevésbé lesznek pontosak. További információkért lásd: [az Application Insightsban mintavételi](/azure/application-insights/app-insights-sampling). Az adatmennyiség csökkentése érdekében által előre összesítése metrikák &mdash; Ez azt jelenti, például átlag és szórás statisztikai értékek kiszámítása, és ezeket az értékeket a nyers telemetriaadatok helyett küld. A következő blogbejegyzés ismerteti egy Application Insights segítségével léptékű módjáról: [Azure figyelése és az elemzések léptékű](https://blogs.msdn.microsoft.com/azurecat/2017/05/11/azure-monitoring-and-analytics-at-scale/).

Ezenkívül győződjön meg arról, hogy tudomásul veszi a árképzési modellt az Application Insights, mert adatmennyiség alapján van szó. További információkért lásd: [kezelése az Application Insightsban tarifa- és adatok kötet](/azure/application-insights/app-insights-pricing). Ha az alkalmazás telemetriai nagy mennyiségű hoz létre, és nem kíván végrehajtani mintavételi vagy az adatok összesítését, majd az Application Insights nem lehet a megfelelő választás. 

Az Application Insights nem felel meg a követelményeknek, ha az alábbiakban néhány javasolt különböző módszer népszerű nyílt forráskódú technológiák használata.

A rendszer és a tároló metrika, fontolja meg a metrikák exportálása egy idősorozat adatbázis, mint **Prometheus** vagy **InfluxDB** a fürtön.

- InfluxDB egy leküldéses rendszerbe. A metrikák leküldéses kell egy ügynököt. Használhat [Heapster][heapster], amely egy szolgáltatás, amely kubelet, a fürtre vonatkozó metrikákat gyűjtő összesíti az adatokat, és InfluxDB vagy más idősorozat tárolási megoldás leküldi. Az Azure Tárolószolgáltatás Heapster a fürt telepítés részeként telepíti. Másik lehetőségként [Telegraf](https://www.influxdata.com/time-series-platform/telegraf/), amely egy olyan ügynök összegyűjtésére és metrikákat reporting. 

- Prometheus egy olyan lekérésalapú rendszer. A konfigurált helyeket metrikák rendszeresen scrapes. Prometheus is scrape metrikák cAdvisor vagy kube-állapot-metrikák állítja elő. [kube-állapot-metrikák] [ kube-state-metrics] metrikák gyűjti össze a Kubernetes API-kiszolgálóhoz, és elérhetővé teszi azokat Prometheus (vagy egy Prometheus ügyfél végpont kompatibilis kaparóként) szolgáltatás. Heapster összefoglalja, hogy Kubernetes hoz létre, és továbbítja őket a fogadó metrikákat, mivel a kube-állapot-metrikák hoz létre a saját metrikákat, és elérhetővé válnak végpont lekaparást keresztül. A rendszer metrika, használjon [csomópont exportáló](https://github.com/prometheus/node_exporter), vagyis a rendszer metrikáihoz Prometheus exportáló. Lebegőpontos pont adatait, de nem karakterlánc típusú adatok Prometheus támogatja, ezért indokolt rendszer metrikákat, de nem naplók.

- Például egy irányítópult eszközzel **Kibana** vagy **Grafana** megjelenítheti és áttekintheti az adatokat. Az irányítópult szolgáltatás is futtatható a fürt egy tároló belül.

Az alkalmazás naplóiban, érdemes lehet **Fluentd** és **Elasticsearch**. Fluentd egy nyílt forráskódú adatgyűjtő, Elasticsearch pedig egy dokumentum-adatbázis egy keresőmotor segítségével optimalizált. Ezzel a megközelítéssel minden szolgáltatás által a naplófájlok `stdout` és `stderr`, és Kubernetes ezekbe az adatfolyamokba ír a helyi fájlrendszerben. Fluentd gyűjti a naplókat, opcionálisan további metaadatokkal Kubernetes a következőképpen színesíti őket, és elküldi a naplók Elasticsearch. Kibana, Grafana vagy hasonló eszköz használatával hozzon létre egy irányítópultot Elasticsearch. A fürt, amely biztosítja, hogy egy Fluentd pod hozzá van rendelve minden csomóponton egy daemonset Fluentd futtatja. Beállíthatja, hogy Fluentd kubelet naplókat, valamint a tároló naplók gyűjtéséhez. A nagy mennyiségük naplók írása a helyi fájlrendszerben válhat a teljesítménybeli szűk keresztmetszetek, különösen akkor, ha több szolgáltatás ugyanazon a csomóponton futnak. Figyelje a késleltetés és a fájl rendszer lemezhasználat éles környezetben.

Egy Fluentd Elasticsearch a naplók előnye, hogy a szolgáltatások nem szükséges további könyvtár függőségeit. Minden szolgáltatás csak ír `stdout` és `stderr`, és a naplók exportálását történő Elasticsearch Fluentd kezeli. Megtudhatja, hogyan konfigurálhatja a naplózás infrastruktúrát is, a csoportok szolgáltatások írása nem szükséges. Egy feladat az éles környezet, a Elasticsearch fürt konfigurálásához, hogy a forgalom kezelésére méretezés. 

Egy másik lehetőség egy elküldeni a naplókat az Operations Management Suite (OMS) szolgáltatáshoz. A [Naplóelemzési] [ log-analytics] naplóadatokat gyűjti be egy központi tárházban, és a szolgáltatás képes egyesíteni más Azure-szolgáltatásokkal, az alkalmazás által használt adatokat. További információkért lásd: [figyelése az Azure Tárolószolgáltatás-fürtöt a Microsoft Operations Management Suite (OMS)][k8s-to-oms].

## <a name="example-logging-with-correlation-ids"></a>Példa: Naplózás korrelációs azonosító

Mutatja be az egyes ebben a fejezetben bemutatott pontok, ez hogyan a csomag szolgáltatás megvalósítja naplózási kiterjesztett példát. A csomag szolgáltatáshoz készült géppel és használja a [Koa](http://koajs.com/) a Node.js webes keretrendszer. Nincsenek több Node.js naplózási szalagtárak választhat. A Microsoft kivételezett [Winston](https://github.com/winstonjs/winston), olyan népszerű naplózási könyvtár, amely megfelel a teljesítményének tesztelésekor azt.

Foglalják magukban a megvalósítás részletei, hogy egy absztrakt meghatározott `ILogger` felület:

```ts
export interface ILogger {
    log(level: string, msg: string, meta?: any)
    debug(msg: string, meta?: any)
    info(msg: string, meta?: any)
    warn(msg: string, meta?: any)
    error(msg: string, meta?: any)
}
```

Íme egy `ILogger` a Winston könyvtár nagyságúra végrehajtására. A korrelációs azonosító konstruktor paramétert fogad, és az azonosító esetében minden napló üzenetbe. 

```ts
class WinstonLogger implements ILogger {
    constructor(private correlationId: string) {}
    log(level: string, msg: string, payload?: any) {
        var meta : any = {};
        if (payload) { meta.payload = payload };
        if (this.correlationId) { meta.correlationId = this.correlationId }
        winston.log(level, msg, meta)
    }
  
    info(msg: string, payload?: any) {
        this.log('info', msg, payload);
    }
    debug(msg: string, payload?: any) {
        this.log('debug', msg, payload);
    }
    warn(msg: string, payload?: any) {
        this.log('warn', msg, payload);
    }
    error(msg: string, payload?: any) {
        this.log('error', msg, payload);
    }
}
```

A csomag szolgáltatásnak kell a korrelációs azonosító kinyerése a HTTP-kérelem. Például ha linkerd használ, a korrelációs azonosító található a `l5d-ctx-trace` fejléc. A HTTP-kérelem Koa, egy környezeti objektumot, amely lekérdezi a csővezeték-kérelemfeldolgozáshoz keresztül történő továbbítása van tárolva. A korrelációs azonosító beszerzése a környezetben, és a tranzakciónaplókat tartalmazó inicializálása egy köztes függvény azt adhatja meg. (Egy köztes Koa a függvény egyszerűen hajtsa végre az egyes kérelmek függvényből.)

```ts
export type CorrelationIdFn = (ctx: Context) => string;

export function logger(level: string, getCorrelationId: CorrelationIdFn) {
    winston.configure({ 
        level: level,
        transports: [new (winston.transports.Console)()]
        });
    return async function(ctx: any, next: any) {
        ctx.state.logger = new WinstonLogger(getCorrelationId(ctx));
        await next();
    }
}
```

A köztes meghívja a hívó által definiált függvény `getCorrelationId`, beolvasni a korrelációs. Ezután a naplózó példányát, és azt belül stashes `ctx.state`, amelyen egy kulcs-érték szótára Koa át az információkat a feldolgozási folyamaton keresztül. 

A naplózó köztes kerül a folyamat indításakor:

```ts
app.use(logger(Settings.logLevel(), function (ctx) {
    return ctx.headers[Settings.correlationHeader()];  
}));
```

Miután minden be van állítva, is könnyen naplózási utasítás hozzá a kódot. Például itt a lehetőség, amely megkeresi a csomagot. Két hív a `ILogger.info` metódust.

```ts
async getById(ctx: any, next: any) {
  var logger : ILogger = ctx.state.logger;
  var packageId = ctx.params.packageId;
  logger.info('Entering getById, packageId = %s', packageId);

  await next();

  let pkg = await this.repository.findPackage(ctx.params.packageId)

  if (pkg == null) {
    logger.info(`getById: %s not found`, packageId);
    ctx.response.status= 404;
    return;
  }

  ctx.response.status = 200;
  ctx.response.body = this.mapPackageDbToApi(pkg);
}
```

Nem szükséges tartalmazza a korrelációs Azonosítót a naplózás utasításokat, mert a köztes függvény által automatikusan ezt. Ez megkönnyíti a naplózás kód tisztító, és csökkenti annak esélyét, hogy a fejlesztő felejtse el tartalmazza a korrelációs azonosítót. Ezért a naplózás utasítások használják az absztrakt `ILogger` felületet, lenne könnyen cserélje le a tranzakciónaplókat tartalmazó megvalósítása újabb.

> [!div class="nextstepaction"]
> [Folyamatos integrációt és kézbesítés](./ci-cd.md)

<!-- links -->

[app-insights]: /azure/application-insights/app-insights-overview
[heapster]: https://github.com/kubernetes/heapster
[kube-state-metrics]: https://github.com/kubernetes/kube-state-metrics
[k8s-to-oms]: /azure/container-service/kubernetes/container-service-kubernetes-oms
[log-analytics]: /azure/log-analytics/