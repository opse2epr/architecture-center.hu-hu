---
title: Naplózás és figyelés a mikroszolgáltatások
description: Naplózás és figyelés a mikroszolgáltatások
author: MikeWasson
ms.date: 10/23/2018
ms.openlocfilehash: 6fe7c9477ac65f98dfc106dc05a82dc2a2c56266
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/08/2019
ms.locfileid: "54113620"
---
# <a name="designing-microservices-logging-and-monitoring"></a>Mikroszolgáltatások tervezése: Naplózás és figyelés

Bármely összetett alkalmazásban valamikor hiba fog hiba lép fel. A mikroszolgáltatás-alkalmazások, a nyomon követendő több tucat vagy akár több száz szolgáltatások történéseiről. Naplózás és figyelés különösen fontosak, hogy a rendszer holisztikus.

![Figyelés a mikroszolgáltatási architektúra ábrája](./images/monitoring.png)

A mikroszolgáltatási architektúrákban, különösen kihívást jelenthet kiszűrheti a hibák vagy a teljesítmény szűk pontos okát. Egyetlen felhasználói művelet több szolgáltatást előfordulhat, hogy kiterjednek. Szolgáltatások tapasztalhat a hálózati i/o-korlátok a fürtben. Szolgáltatások közötti hívások láncolatától ellennyomás a rendszer magas késést eredményez, vagy alapú egymásra épülő hibákat okozhat. Ezen kívül általában nem tudja melyik csomópont egy adott tároló fog futni. Ugyanazon a csomóponton elhelyezett tárolók korlátozott CPU és memória tagja is lehet.

Ahhoz, hogy az értelemben, hogy mi történik, az alkalmazás telemetriai kell gyűjteni.  Telemetria osztható *naplók* és *metrikák*. [Az Azure Monitor](/azure/monitoring-and-diagnostics/monitoring-overview) összegyűjti a naplókat és a metrikák az Azure platformon keresztül.

**Naplók** szöveges rekord az alkalmazás futása közben bekövetkező eseményekről. Ezek közé tartozik többek között az alkalmazásnaplókat (nyomkövetési utasításokat) vagy a webkiszolgáló-naplókkal. Naplók elsősorban hasznosak lehetnek az eseményadatokhoz és a legfelső szintű alapprobléma elemzéséhez.

**Metrikák** numerikus érték, amely elemezhetők. Használhatja őket megfigyelését a rendszer a valós idejű (vagy a közel valós idejű), vagy a teljesítménytrendek elemzéséhez az idő függvényében. Metrikák is subcategorized tovább a következő lehet:

- **Csomópont-szintű** mérőszámokat, például a CPU, memória, hálózati, lemez és fájlrendszerhasználat. Rendszermérőszámokat könnyebben értheti meg a fürt egyes csomópontjaihoz tartozó erőforrás-elosztás és kiugró értékek a hibaelhárítás.

- **Tároló** metrikákat. Ha tárolókon belüli szolgáltatások futnak, kell gyűjteni a tároló szintjén, nem csak a virtuális gép szintjén. A tárolók számítási feladatainak Azure Kubernetes Service (AKS) monitorozása az Azure Monitor is beállítása. További információkért lásd: [tárolók áttekintése az Azure Monitor](/azure/monitoring/monitoring-container-insights-overview). Más tárolóvezénylőt, használja a [Tárolómonitorozási megoldás a Log Analyticsben](/azure/log-analytics/log-analytics-containers).

- **Alkalmazás** metrikákat. Ez magában foglalja a minden olyan metrikák, amely a szolgáltatás működésének megismerése. Ilyenek például a várólistán lévő bejövő HTTP-kérelmekre, a kérés várakozási ideje vagy üzenet-várólista hossza számát. Alkalmazásokat is létrehozhat egyéni metrikákat, amelyek a megadott tartományhoz, például az üzleti tranzakciók percenként feldolgozott számát. Használat [Application Insights](/azure/application-insights/app-insights-overview) , alkalmazásmetrikák engedélyezése.

- **Függő szolgáltatások** metrikákat. Szolgáltatások meghívhatja a külső szolgáltatások vagy a végpontot, például a felügyelt PaaS-szolgáltatások vagy SaaS-szolgáltatásokhoz. Harmadik féltől származó szolgáltatásokkal is, vagy előfordulhat, hogy biztosít bármely metrikák. Ha nem, saját nyomon követheti statisztika a késés és a hibák aránya, alkalmazásmetrikák támaszkodnak kell.

## <a name="considerations"></a>Megfontolandó szempontok

A cikk [Monitorozási és diagnosztikai](../best-practices/monitoring.md) egy általános ajánlott eljárásokat ismerteti. Az alábbiakban néhány adott tudnivaló a mikroszolgáltatási architektúra kontextusában gondolja.

**Konfigurálással és felügyelettel**. Fogja alkalmazni egy felügyelt szolgáltatás, naplózás és figyelés vagy naplózás és figyelés összetevők, a fürtben lévő tárolók üzembe helyezése? További leírását az alábbi lehetőségek közül, tekintse meg a szakasz [technológiákkal](#technology-options) alatt.

**Betöltési arány**. Mi az az átviteli sebességet, amellyel a rendszer betöltheti a telemetria-eseményeinek? Mi történik, hogy aránya túllépi? Ha például a rendszer lehetséges, hogy ügyfelek forgalmának szabályozása, ebben az esetben telemetriai adat elvész, vagy, előfordulhat, hogy az adatok felbontásának. A probléma néha csökkentheti a gyűjtött adatok mennyisége csökkentésével:

- Metrikák összesítés szerint kiszámításának statisztikákat, pl.: átlagos és a szórást, és a statisztikai adatokat küldeni a monitorozási rendszer.
- Az adatok felbontásának &mdash; , az csak az események százalékos feldolgozni.
- Batch az adatokat a figyelési szolgáltatás hálózati hívások számának csökkentése érdekében.

**Költség**. Előfordulhat, hogy magas, különösen nagy mennyiségben feldolgozására, és tárolja a telemetriai adatokat. Bizonyos esetekben ez még akkor is túllépheti az alkalmazás futtatásával járó költségeket. Ebben az esetben előfordulhat, hogy csökkenteni kell a telemetriai adatok mennyisége összesítésével, egyszerűsítés, vagy az adatok kötegelés fent leírtak szerint.

**Adathűséget**. Hogyan pontos vannak a metrikák? Kiugró értékek, különösen nagy mennyiségű átlagokat elrejtéséhez. Is ha a mintavételi ráta túl alacsony, azt is simítja ingadozása az adatokat. Jelenhet meg, hogy minden kérelemhez van kapcsolatban az azonos végpontok közötti késés, ha valójában egy jelentős része kérelmek vannak sokkal tovább tart.

**Késés**. Engedélyezi a valós idejű figyelés és riasztások, a telemetriai adatok elérhetőnek kell lennie gyorsan. Hogyan "valós idejű" az a figyelési irányítópulton megjelenő adatokat? Néhány másodperc régi? Több mint egy perc?

**Storage.** A naplók esetében lehet leghatékonyabb a naplózási eseményeket írni a fürt ideiglenes tároló, és a egy ügynök több állandó tárolókba naplófájlok szállításra való konfigurálásához.  Adatok idővel át hosszú távú tárolásra, hogy legyen elérhető utólagos elemzése. A mikroszolgáltatási architektúra hozhat létre nagy mennyiségű telemetriai adatot, így az adatok költségét fontos szempont. Is vegye figyelembe, hogyan fog kérdezni az adatokat.

**Irányítópult és a Vizualizáció.** Kapunk holisztikus a rendszer minden szolgáltatás, mind a fürt és a külső szolgáltatások között? Ha naplók és a telemetriai adatokat több helyre, is az irányítópult összes megjelenítése és összekapcsolását? A figyelési irányítópult kell megjelennie, legalább a következő információkat:

- Általános erőforrás-elosztás kapacitás és a növekedés. Ez magában foglalja a tárolók, a fájl rendszermérőszámokat, a hálózati és a core foglalási számát.
- Tárolómetrikák korrelált szolgáltatási szintű.
- Rendszermérőszámokat tárolók is vonatkozhatnak.
- Szolgáltatási hibák és a kiugró értékeket.

## <a name="distributed-tracing"></a>Elosztott nyomkövetés

Ahogy már említettük, a mikroszolgáltatások az egyik kihívás van szolgáltatások között az események áramlását ismertetése. Egy egyszeri művelet vagy egy tranzakció több szolgáltatásra meghívásával is járhat. Lépések teljes rekonstruálhassa az egyes szolgáltatások propagálása kell egy *korrelációs azonosító* , amely úgy működik, mint a művelet egyedi azonosítója. A korrelációs azonosító lehetővé teszi, hogy [elosztott nyomkövetési](https://microservices.io/patterns/observability/distributed-tracing.html) szolgáltatások között.

Az első szolgáltatás, amely fogad egy ügyfél kérelmet kell létrehoznia a korrelációs azonosítót. Ha a szolgáltatás lehetővé teszi a HTTP-hívás egy másik szolgáltatás, a korrelációs azonosító a fejléc helyezi el. Hasonlóképpen ha a szolgáltatás aszinkron üzenetet küld, helyezi el a korrelációs Azonosítót az üzenet. Alárendelt szolgáltatásokkal továbbra is propagálása a korrelációs Azonosítót, hogy a teljes rendszeren keresztül biztosítani. Emellett az összes olyan kód, írja az alkalmazás metrikákkal vagy naplózási események tartalmaznia kell a korrelációs azonosítót.

Hívások közötti kapcsolatot, ha például a végpontok közötti késése a teljes tranzakció, a másodpercenkénti sikeres tranzakciók számát és százalékos arányát sikertelen tranzakciók üzemeltetési mérőszámokkal kiszámíthatja. Beleértve a korrelációs azonosítók alkalmazásnaplókat, lehetővé válik a hajtsa végre a problémák eredeti okának feltárását. Ha egy művelet meghiúsul, a log utasítások a szolgáltatás meghívja a műveletben részét képező összes is megtalálhatja.

Elosztott nyomkövetést megvalósításához az alábbiakban néhány szempontot:

- Jelenleg nem korrelációs azonosítók nem szabványos HTTP-fejléc. A csapat egységesen kell ezeket egy egyéni fejléc értéke. A kiválasztott határozzák meg a naplózás és figyelés keretrendszer vagy szolgáltatás háló kiválasztását.

- Aszinkron üzenetek az üzenetkezelési infrastruktúra támogatja a hozzáadásával metaadatok üzenetek, akkor tartalmaznia kell a korrelációs Azonosítót metaadatokként. Ellenkező esetben adja meg az üzenet-séma részeként.

- Ahelyett, hogy egyetlen nem átlátszó azonosítót, előfordulhat, hogy küldjön egy *korrelációs környezet* , amely részletesebb információkat, például a hívó által hívott kapcsolatokat tartalmaz.

- Az Azure Application Insights SDK automatikusan korrelációs környezet kódtárba be HTTP-fejléceket, és az Application Insights-naplók a korrelációs Azonosítót tartalmaz. Ha úgy dönt, az Application Insights beépített korrelációs funkcióinak használatát, bizonyos szolgáltatások előfordulhat, hogy továbbra is explicit módon propagálni kell a korrelációs fejlécek, a használt kódtárak függően. További információkért lásd: [az Application Insights Telemetriai korreláció](/azure/application-insights/application-insights-correlation).

- Ha használunk szolgáltatás rácsvonal Istio vagy linkerd, ezek a technológiák automatikusan generáljon korrelációs fejlécek HTTP-hívások kerülnek a szolgáltatás háló proxyk. Szolgáltatások továbbítsa a megfelelő fejlécek.

  - Istio: [Elosztott kérelmek nyomkövetése](https://istio-releases.github.io/v0.1/docs/tasks/zipkin-tracing.html)
  - Linkerd: [Környezet fejléc](https://linkerd.io/config/1.3.0/linkerd/index.html#http-headers)

- Fontolja meg, hogyan fogja összesíteni a naplókat. Különböző csapatokkal való korrelációazonosítók bele naplók egységesítésére érdemes. Használhat egy strukturált és részben strukturált formátumban, például a JSON-t, és egy közös mezőt, amely tárolja a korrelációs azonosítót.

## <a name="technology-options"></a>Technológiai lehetőségek

**Az Application Insights** egy felügyelt szolgáltatás, az Azure-ban, amely feltölti és tárolja a telemetriai adatokat és eszközöket biztosít az adatok keresése és elemzésére. Az Application Insights használatához telepítenie egy kialakítási csomagot az alkalmazásban. Ez a csomag az alkalmazás figyeli, és telemetriai adatokat küld az Application Insights szolgáltatás. Azt is lekéri, hogy a gazdagép-környezetből a telemetriai adatokat. Az Application Insights biztosít beépített összefüggések keresésére és függőségi nyomkövetés. Segítségével nyomon követheti a rendszer metrikákat, alkalmazásmetrikák és Azure-szolgáltatási metrika, egy helyen.

Vegye figyelembe, hogy az Application Insights szabályozza, ha az adatátviteli sebessége meghaladja a maximális határértéket; További információkért lásd: [korlátozza az Application Insights](/azure/azure-subscription-service-limits#application-insights-limits). Egyetlen művelet több telemetriai események, előfordulhat, hogy létrehozása, ha az alkalmazás nagy mennyiségű forgalmat, valószínű, hogy leszabályozza. A probléma mérséklése érdekében elvégezhet a mintavétel a telemetriai forgalom csökkentése érdekében. A rendszer a kompromisszummal jár, hogy a metrikák kevésbé lesznek pontosak. További információkért lásd: [Application Insights-mintavétel](/azure/application-insights/app-insights-sampling). Is csökkenthető az adatmennyiség előre összesítése metrikák &mdash; azt jelenti, például átlag, és a szórást statisztikai értékek kiszámítása, és ezek az értékek helyett a nyers telemetriát küld. A következő blogbejegyzést egy Application Insights használatával nagy mennyiségű megközelítést ismerteti: [Az Azure figyelési és nagy méretű](https://blogs.msdn.microsoft.com/azurecat/2017/05/11/azure-monitoring-and-analytics-at-scale/).

Emellett ellenőrizze, hogy megértette a díjszabási modell az Application Insights, mert, adatmennyiség alapján lesznek kiszámlázva. További információkért lásd: [az Application Insights árak és adatmennyiségek kezelése](/azure/application-insights/app-insights-pricing). Ha az alkalmazás egy igen nagy mennyiségű telemetriai adatokat hoz létre, és nem kívánja azt végre mintavétel vagy az adatok összesítését, majd az Application Insights nem lehet a megfelelő választás.

Ha az Application Insights nem felel meg a követelményeknek, az alábbiakban néhány olyan népszerű nyílt forráskódú technológiákat használó javasolt módszert.

A rendszer és a tároló mérőszámokat, fontolja meg a metrikák exportálása egy idősorozat-adatbázisba például **Prometheus** vagy **InfluxDB** a fürtben futó.

- InfluxDB egy leküldéses rendszerbe. Küldje le a metrikákat kell egy ügynököt. Használhat [Heapster][heapster], egy szolgáltatás, amely a fürt szintű metrikákat gyűjt kubelet, azaz összesíti az adatokat, majd leküldi azt InfluxDB vagy egyéb idősorozat-tárolási megoldás. Az Azure Container Service Heapster a fürt telepítés részeként telepíti. Másik lehetőségként [Telegraf](https://www.influxdata.com/time-series-platform/telegraf/), ez az ügynök gyűjtéséhez és jelentéskészítési metrikákat.

- Prometheus egy olyan pull-alapú rendszer. A konfigurált helyeket metrikák rendszeres időközönként scrapes. Prometheus is scrape metrikák cAdvisor vagy kube-állapot-metrikák alapján jön létre. [kube-állapot-metrics] [ kube-state-metrics] egy szolgáltatás, amely a mérőszámokat gyűjti össze a Kubernetes API-kiszolgálóhoz, és elérhetővé teszi azokat Prometheus (vagy egy Prometheus ügyfél végpont kompatibilis kaparóként). Heapster összesíti, hogy a Kubernetes állít elő, és továbbítja őket a fogadóba másolt metrikák, mivel kube-állapot-metrikák hoz létre a saját mérőszámokat, és elérhetővé teszi azokat a Pro automatizované získávání dat-végponton keresztül. Használja a rendszer mérőszámokat, [csomópont exportáló](https://github.com/prometheus/node_exporter), azaz a rendszer metrikákhoz Prometheus exportáló. Pont adatait, de nem karakterláncadatokat lebegőpontos Prometheus támogatja, így célszerű a rendszer metrikák naplókat azonban nem.

- Például egy irányítópult eszközzel **Kibana** vagy **Grafana** , az adatok megjelenítése és figyelése. Az irányítópult szolgáltatás is futtathatja a fürt egy tárolóban.

Az alkalmazásnaplók, fontolja meg **Fluentd** és **Elasticsearch**. Fluentd egy nyílt forráskódú adatgyűjtő, és az Elasticsearch a dokumentum-adatbázis, amely arra optimalizáltuk, hogy a keresőmotor-kiszolgálóként. Ezt a módszert használja, minden egyes szolgáltatás naplókat küld `stdout` és `stderr`, és a Kubernetes ezekbe az adatfolyamokba ír a helyi fájlrendszerben. Fluentd összegyűjti a naplókat, igény szerint bővíti azokat a további metaadatokkal a Kubernetes és a naplókat küld az Elasticsearchbe. Kibana, a Grafana vagy egy hasonló eszköz használatával hozzon létre egy irányítópultot az elasticsearch számára. A daemonset a fürtben, amely biztosítja, hogy egy Fluentd pod hozzá van rendelve minden csomóponton fut Fluentd. Kubelet-naplók, valamint a tároló naplóinak gyűjtéséhez Fluentd konfigurálhatja. Nagy mennyiségben naplók írása a helyi fájlrendszerbe válhat teljesítménybeli szűk keresztmetszetek, különösen akkor, ha több szolgáltatás ugyanazon a csomóponton futnak. Figyelheti a késés és a fájl rendszer lemezhasználat éles környezetben.

Egy Fluentd az Elasticsearch a naplók előnye, hogy a szolgáltatások nem szükséges további könyvtár függőségeit. Minden egyes szolgáltatás csak ír `stdout` és `stderr`, és az Elasticsearch a naplók exportálása Fluentd kezeli. Megtudhatja, hogyan konfigurálhatja a naplózás infrastruktúra is, a csapatok szolgáltatások írása nem szükséges. Az egyik kihívás, hogy az Elasticsearch-fürt, éles környezet konfigurálása, úgy, hogy a méretezés a forgalom kezelésére.

Egy másik lehetőség, hogy az Operations Management Suite (OMS) Log Analytics naplók küldése. A [Log Analytics] [ log-analytics] egy központi tárházba gyűjti naplóadatok szolgáltatást, és más Azure-szolgáltatások, az alkalmazás által használt adatokat blokktárolását. További információkért lásd: [monitorozása az Azure Container Service-fürt a Microsoft Operations Management Suite (OMS)][k8s-to-oms].

## <a name="example-logging-with-correlation-ids"></a>Példa: Korrelációs azonosítók való bejelentkezés

Egyes ebben a fejezetben ismertetett pontok megmutatják, Íme egy kiterjesztett példa hogyan a a csomag szolgáltatás biztosítja a naplózás. A csomag szolgáltatás íródott, a TypeScript, és használja a [Koa](https://koajs.com/) Node.js webes keretrendszer. Nincsenek számos Node.js naplózási függvénytárak közül választhat. Hogy kivételezett [Winston](https://github.com/winstonjs/winston), a teljesítményre vonatkozó követelmények teljesítése tesztelt, amikor egy elterjedt naplózási könyvtár.

A gyakorlati kivitelezés részleteinek tárolják, egy absztrakt meghatározott `ILogger` felületen:

```ts
export interface ILogger {
    log(level: string, msg: string, meta?: any)
    debug(msg: string, meta?: any)
    info(msg: string, meta?: any)
    warn(msg: string, meta?: any)
    error(msg: string, meta?: any)
}
```

Íme egy `ILogger` megvalósítása, amely a Winston könyvtárban. Konstruktor paramétereként a korrelációs Azonosítót vesz igénybe, és az azonosító kódtárba minden naplóüzenetre be.

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

A csomag szolgáltatást kell a korrelációs Azonosítót kinyerése a HTTP-kérelem. Például ha linkerd használ, a korrelációs azonosító megtalálható a `l5d-ctx-trace` fejléc. A HTTP-kérelem Koa, a Context objektumot, amely a kérelemfeldolgozási folyamatot keresztülmegy van tárolva. A korrelációs Azonosítót a környezetből, és a naplózó inicializálása egy közbenső függvényt is meghatározzuk. (Egy közbenső Koa függvényt a egyszerűen függvény, amely lekéri az egyes kérések végrehajtása.)

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

A közbenső szoftver meghívja a hívó által megadott függvény `getCorrelationId`beolvasásához a korrelációs azonosítót. Majd létrehoz egy példányt a naplózó és stashes azt `ctx.state`, azaz egy kulcs-érték szótára Koa keresztül adatokat átadni.

A naplózó közbenső kerül a folyamat indításakor:

```ts
app.use(logger(Settings.logLevel(), function (ctx) {
    return ctx.headers[Settings.correlationHeader()];  
}));
```

Miután minden be van állítva, könnyebbé vált a naplózási utasítások hozzá a kódot. Például a itt módszer, amely megkeresi az egy csomagot. Két hív a `ILogger.info` metódust.

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

Nincs szükségünk, a naplózás utasításokat tartalmazza a korrelációs Azonosítót, mert, amely a közbenső szoftver függvény által automatikusan történik. Ez lehetővé teszi a naplózást kód tisztító, és csökkenti annak az esélyét, hogy a fejlesztő felejtse el tartalmazza a korrelációs azonosítót. És mivel minden, a naplózás kimutatások használata az absztrakt `ILogger` felületen, könnyű lenne később naplózó megvalósítása helyett.

> [!div class="nextstepaction"]
> [A folyamatos integrációt és teljesítést](./ci-cd.md)

<!-- links -->

[app-insights]: /azure/application-insights/app-insights-overview
[heapster]: https://github.com/kubernetes/heapster
[kube-state-metrics]: https://github.com/kubernetes/kube-state-metrics
[k8s-to-oms]: /azure/container-service/kubernetes/container-service-kubernetes-oms
[log-analytics]: /azure/log-analytics/
