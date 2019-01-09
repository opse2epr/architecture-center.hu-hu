---
title: Állapot végponti monitorozását végző minta
titleSuffix: Cloud Design Patterns
description: Rendszeres időközönként működés-ellenőrzéseket implementálhat egy alkalmazásban, amelyhez az elérhetővé tett végpontokon keresztül hozzáférhetnek külső eszközök.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
ms.custom: seodec18
ms.openlocfilehash: 85a1355ff47e6fce80d9b2ed114024651eb994db
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/08/2019
ms.locfileid: "54114249"
---
# <a name="health-endpoint-monitoring-pattern"></a>Állapot végponti monitorozását végző minta

[!INCLUDE [header](../_includes/header.md)]

Rendszeres időközönként működés-ellenőrzéseket implementálhat egy alkalmazásban, amelyhez az elérhetővé tett végpontokon keresztül hozzáférhetnek külső eszközök. Ennek segítségével ellenőrizheti, hogy az alkalmazások és szolgáltatások megfelelően működnek-e.

## <a name="context-and-problem"></a>Kontextus és probléma

A webalkalmazások és háttérszolgáltatások monitorozása bevált gyakorlat, és gyakran üzleti elvárás is. Ezzel biztosítható, hogy elérhetőek legyenek és megfelelően működjenek. A felhőben futó szolgáltatások monitorozása azonban bonyolultabb, mint a helyszíni szolgáltatásoké. Nem rendelkezik például teljes körű hozzáféréssel az üzemeltetési környezethez, emellett a szolgáltatások általában más, platformszolgáltatók és mások által biztosított szolgáltatásoktól függnek.

Számos tényező befolyásolja a felhőben üzemeltetett alkalmazásokat, például a hálózati késés, a mögöttes számítási és tárolórendszerek teljesítménye és rendelkezésre állása, valamint a köztük lévő hálózati sávszélesség. E tényezők miatt a szolgáltatás működése részben vagy teljesen meghiúsulhat. Ezért rendszeres időközönként ellenőriznie kell, hogy a szolgáltatás megfelelően működik-e, hogy biztosítsa a szükséges szintű rendelkezésre állást, amely része lehet a szolgáltatói szerződésnek (SLA).

## <a name="solution"></a>Megoldás

Az alkalmazás végpontjaira való kérések küldésével megvalósíthatja az állapotmonitorozást. Az alkalmazásnak el kell végeznie a szükséges ellenőrzéseket, és vissza kell adnia egy jelzést az állapotra vonatkozóan.

Az állapotmonitorozási ellenőrzés általában két tényezőt ötvöz:

- Az alkalmazás vagy szolgáltatás által, az állapot-ellenőrzési végpontra küldött kérésre válaszul végrehajtott ellenőrzések (ha van ilyen).
- Az állapotellenőrzést végző eszköz vagy keretrendszer által végrehajtott eredményelemzés.

A válaszkód jelzi az alkalmazás és igény szerint az alkalmazás által használt bármely összetevő vagy szolgáltatás állapotát. A késleltetés vagy válaszidő ellenőrzését a monitorozó eszköz vagy keretrendszer végzi. Az ábra áttekintést nyújt a mintáról.

![A minta áttekintése](./_images/health-endpoint-monitoring-pattern.png)

Az egyéb ellenőrzések, amelyeket az állapotmonitorozási kód végrehajthat az alkalmazásban többek között a következők lehetnek:

- Felhőalapú tárolók vagy adatbázisok rendelkezésre állásának és válaszidejének ellenőrzése.
- Az alkalmazásban vagy máshol található (de az alkalmazás által használt) egyéb erőforrások vagy szolgáltatások ellenőrzése.

Elérhetők olyan szolgáltatások és eszközök is, amelyek azáltal monitorozzák a webalkalmazásokat, hogy kérést küldenek egy konfigurálható végpontkészletre, és konfigurálható szabályok alapján kiértékelik az eredményeket. Viszonylag egyszerű olyan szolgáltatásvégpontot létrehozni, amelynek egyetlen célja az, hogy funkcióteszteket hajtson végre a rendszeren.

A monitorozási eszközökkel elvégezhető általános ellenőrzések:

- A válaszkód ellenőrzése. Például a 200-as (OK) kódú HTTP-válasz azt jelzi, hogy az alkalmazás hiba nélkül válaszolt. Elképzelhető, hogy a monitorozási rendszer más válaszkódokat is ellenőriz annak érdekében, hogy átfogóbb eredményeket adjon vissza.
- A válasz tartalmának ellenőrzése a hibák észlelése érdekébe, még akkor is, ha a rendszer 200-as (OK) állapotkódot adott vissza. Ezáltal észlelhetők olyan hibák, amelyek a weblap vagy szolgáltatás visszaadott válaszának csak egy részét érintik. Ilyen például az oldalak címének ellenőrzése, vagy adott kifejezés keresése, amely azt jelzi, hogy a megfelelő oldal lett visszaadva.
- A válaszidő mérése, amely a hálózati késésből és az alkalmazás által, a kérés végrehajtásához felhasznált időből tevődik össze. A növekvő érték egy probléma megjelenését jelezheti az alkalmazásban vagy a hálózatban.
- Az alkalmazáson kívüli erőforrások vagy szolgáltatások ellenőrzése, ilyen például a tartalomkézbesítési hálózat, amelyet az alkalmazás globális gyorsítótárakból származó tartalmak továbbítására használ.
- SSL-tanúsítványok lejáratának ellenőrzése.
- Az alkalmazás URL-címére irányuló DNS-címkeresés válaszidejének mérése, amellyel a DNS-késés és DNS-hibák mérhetők.
- A DNS-címkeresés által visszaadott URL-cím ellenőrzése, amellyel biztosítható, hogy a bejegyzések helyesek. Ezzel elkerülhető a DNS-kiszolgáló elleni sikeres támadással indított rosszindulatú kérelemátirányítás.

Emellett ahol lehetséges, hasznos lehet lefuttatni ezeket az ellenőrzéseket különböző helyszíni vagy üzemeltetési helyeken a válaszidők mérésének és összehasonlításának érdekében. Ha lehetséges, olyan helyekről monitorozza az alkalmazásokat, amelyek az ügyfelek közelében találhatók, hogy pontos képet kapjon az egyes helyek teljesítményéről. Amellett, hogy ez hatékonyabb ellenőrzési mechanizmust biztosít, az eredmények alapján dönthet az alkalmazás üzembehelyezési helyéről, és arról, hogy telepíti-e egynél több adatközpontban.

Érdemes teszteket futtatni az ügyfelek által használt összes szolgáltatáspéldányon, hogy meggyőződjön róla, az alkalmazás minden ügyfél esetében megfelelően működik. Ha például az ügyfél tárfiókja több mint egy tárfiókra kiterjed, a monitorozási folyamatnak az összeset ellenőriznie kell.

## <a name="issues-and-considerations"></a>Problémák és megfontolandó szempontok

A minta megvalósítása során az alábbi pontokat vegye figyelembe:

A válasz érvényesítésének módja. Például elég csupán egyetlen 200-as (OK) állapotkód annak ellenőrzésére, hogy megfelelően működik-e az alkalmazás? Habár ez az alkalmazás rendelkezésre állásának legegyszerűbb mérési módja, illetve a minta legalapvetőbb implementációja, kevés információt biztosít a műveletekről, trendekről és az alkalmazásban lehetségesen felmerülő hibákról.

   > Győződjön meg arról, hogy az alkalmazás csak akkor ad vissza 200 (OK) állapotkódot, amikor a rendszer megtalálta és feldolgozta a célerőforrást. Egyes forgatókönyvekben, például akkor, amikor mesteroldalt használ a célweboldal üzemeltetéséhez, a kiszolgáló a 404-es (Nem található) kód helyett a 200-as (OK) állapotkódot küldi vissza, még akkor is, ha a céltartalomoldal nem található.

Az alkalmazás számára elérhető végpontok száma. Az egyik lehetőség elérhetővé tenni legalább egy végpontot az alkalmazás által használt alapszolgáltatások számára, és egy másikat az alacsonyabb prioritású szolgáltatások számára, hogy különböző fontossági szintek legyenek hozzárendelve az egyes monitorozási eredményekhez. Emellett érdemes lehet elérhetővé tenni több végpontot, például egyet minden alapszolgáltatás számára, hogy a monitorozás még részletesebb legyen. Az állapotellenőrzések például ellenőrizhetik az adatbázist, a tárolót és az alkalmazás által használt külső geokódolási szolgáltatásokat, amelyek mindegyike különböző szintű üzemidőt és válaszidőt igényel. Az alkalmazás akkor is megfelelő állapotban lehet, ha a geokódolási szolgáltatás vagy valamely másik háttérfeladat nem érhető el néhány percig.

Használható-e az általános hozzáférésre használt végpont monitorozásra, de az állapot-ellenőrzésekre tervezett egyedi elérési úton, például /HealthCheck/{GUID}/ az általános hozzáférési végponton? Ez lehetővé teszi, hogy a monitorozási eszközök lefuttassanak néhány funkciótesztet az alkalmazásban (ilyen például az új felhasználói regisztrációk hozzáadása, a bejelentkezés és a tesztmegrendelés felvétele), miközben ellenőrzik, hogy az általános hozzáférési végpont elérhető-e.

A szolgáltatásban a monitorozási kérésekre válaszul gyűjtendő információk típusa, és ezek visszaadásának módja. A legtöbb meglévő eszköz és keretrendszer csak a végpont által visszaadott HTTP-állapotkódot figyeli. További információk visszaadásához és érvényesítéséhez elképzelhető, hogy létre kell hoznia egy egyéni monitorozási segédprogramot vagy szolgáltatást.

A gyűjtendő információ mennyisége. Ha a rendszer túlzott mértékű feldolgozást végez az ellenőrzés alatt, az túlterhelheti az alkalmazást, és hatással lehet más felhasználókra. A szükséges idő meghaladhatja a monitorozási rendszer időkorlátját, így az nem elérhetőnek jelzi az alkalmazást. A legtöbb alkalmazásban találhatók például hibakezelők és teljesítményszámlálók, amelyek naplózzák a teljesítményt és a hibákkal kapcsolatos részletes információkat. Ez is elegendő lehet az állapotellenőrzésre vonatkozó bővebb információk visszaadása helyett.

A végpont állapotának gyorsítótárazása. Az állapot-ellenőrzés túl gyakori futtatása költséges lehet. Ha például az állapotjelentés egy irányítópulton keresztül történik, nem szeretnénk, hogy az irányítópultból érkező minden kérés állapot-ellenőrzést indítson el. Helyette azt szeretnénk, hogy időszakosan ellenőrizze a rendszer állapotát és gyorsítótárazza azt. Tegyen elérhetővé egy végpontot, amely visszaadja a gyorsítótárazott állapotot.

Hogyan konfiguráljuk a monitorozási végpontok biztonságát annak érdekében, hogy védve legyenek a nyilvános hozzáféréstől, amely rosszindulatú támadásoknak teheti ki az alkalmazást, bizalmas információkat szivárogtathat ki vagy szolgáltatásmegtagadási (DoS-) támadásokat vonzhat be. Ezt általában az alkalmazás konfigurációjában kell megadni, hogy könnyen frissíthető legyen az alkalmazás újraindítása nélkül. Fontolja meg az alábbi módszerek alkalmazását:

- Tegye biztonságossá a végpontot azáltal, hogy kötelezővé teszi a hitelesítést. Ezt egy hitelesítési biztonsági kulcs a kérés fejlécében való megadásával teheti meg, vagy úgy, hogy hitelesítő adatokat továbbít a kérelemmel együtt, feltéve, hogy a monitorozási szolgáltatás vagy az eszköz támogatja a hitelesítést.

  - Használjon rejtett végpontot. Például tegye elérhetővé a végpontot egy másik, az alapértelmezett alkalmazás URL-címétől eltérő IP-címen, konfigurálja a végpontot egy nem szabványos HTTP-porton, és/vagy használjon a tesztoldalra mutató összetett elérési utat. Általában megadhat további végpontcímeket és -portokat az alkalmazás konfigurációjában, és hozzáadhatja e végpontok bejegyzéseit a DNS-kiszolgálóhoz, ha nem kívánja közvetlenül megadni az IP-címet.

  - Tegyen elérhetővé olyan metódusokat a végpontokon, amelyek olyan paramétereket fogadnak el, mint a kulcsértékek vagy a működési mód értékei. A paraméterhez megadott értéktől függően a kérés fogadásakor a kód elvégezhet egy adott tesztet vagy több tesztet, vagy ha a paraméter nem ismerhető fel, visszaadhatja a 404-es (Nem található) kódot. A felismert paraméterek értékei beállíthatók az alkalmazás konfigurációjában.

     >  A DoS-támadások valószínűleg kisebb hatással lesznek egy különálló végpontra, amely anélkül végez alapszintű funkcióteszteket, hogy akadályozná az alkalmazás működését. Ha lehetséges, ne használjon olyan teszteket, amelyek kiszivárogtathatnak bizalmas információkat. Ha olyan információt kell visszaadnia, amely hasznos lehet egy támadónak, gondolja át, hogyan fogja megvédeni a végpontot és az adatot a jogosulatlan hozzáféréstől. Ebben az esetben nem elég pusztán az elrejtésre támaszkodni. Emellett érdemes lehet HTTPS-kapcsolatot alkalmazni, és titkosítani minden bizalmas adatot, bár ez meg fogja növelni a kiszolgáló terheltségét.

- Hogyan érhetők el az olyan végpontok, amelyek hitelesítés által védettek? Nem minden eszköz és keretrendszer konfigurálható arra, hogy az állapot-ellenőrzési kérésbe belefoglalja a hitelesítő adatokat. A Microsoft Azure beépített állapot-ellenőrzési funkciói például nem képesek hitelesítő adatokat biztosítani. Néhány alternatív, külső megoldás: [Pingdom](https://www.pingdom.com/), [Panopta](https://www.panopta.com/), [NewRelic](https://newrelic.com/) és [Statuscake](https://www.statuscake.com/).

- Hogyan ellenőrizhető, hogy a figyelőügynök megfelelően működik-e? Az egyik lehetőség elérhetővé tenni egy végpontot, amely egyszerűen visszaad egy értéket az alkalmazás konfigurációjából, vagy egy véletlenszerű értéket, amellyel tesztelhető az ügynök.

   >  Emellett bizonyosodjon meg arról, hogy a monitorozási rendszer önmagát is ellenőrzi (például önteszttel és beépített teszttel) annak érdekében, nehogy téves eredményeket adjon.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes ezt a mintát használni?

Ez a minta az alábbi esetekben hasznos:

- Webhelyek és webalkalmazások monitorozása a rendelkezésre állás ellenőrzése érdekében.
- Webhelyek és webalkalmazások monitorozása a megfelelő működés ellenőrzése érdekében.
- Középső rétegű vagy megosztott szolgáltatások monitorozása azon hibák észlelése és elkülönítése érdekében, amelyek megzavarhatják a többi alkalmazást.
- A meglévő alkalmazás kiegészítése például teljesítményszámlálókkal és hibakezelőkkel. Az állapot-ellenőrzés nem helyettesíti a naplózást az alkalmazásban. A rendszerállapot-megfigyelés értékes információkkal szolgálhat egy meglévő keretrendszerre vonatkozóan, amely monitorozza a számlálókat és a hibanaplókat a hibák vagy más problémák észlelése érdekében. Ha azonban az alkalmazás nem érhető el, nem tud információkkal szolgálni.

## <a name="example"></a>Példa

Az alábbi példakódok a `HealthCheckController` osztályból származnak (egy, ezt a mintát bemutató mintakód elérhető a [GitHubon](https://github.com/mspnp/cloud-design-patterns/tree/master/health-endpoint-monitoring)), és azt mutatják be, hogyan lehet elérhetővé tenni a végpontokat különböző állapot-ellenőrzések elvégzéséhez.

A `CoreServices` metódus, amely lent, C# nyelven látható, egy sor ellenőrzést hajt végre az alkalmazás által használt szolgáltatásokon. Ha mindegyik teszt hiba nélkül lefut, a metódus 200-as (OK) állapotkódot ad vissza. Ha bármelyik teszt kivételt jelez, a metódus 500-as (Belső hiba) állapotkódot ad vissza. Ha hiba történik, a metódus igény szerint további információkat adhat vissza, ha a monitorozási eszköz vagy keretrendszer használni tudja azokat.

```csharp
public ActionResult CoreServices()
{
  try
  {
    // Run a simple check to ensure the database is available.
    DataStore.Instance.CoreHealthCheck();

    // Run a simple check on our external service.
    MyExternalService.Instance.CoreHealthCheck();
  }
  catch (Exception ex)
  {
    Trace.TraceError("Exception in basic health check: {0}", ex.Message);

    // This can optionally return different status codes based on the exception.
    // Optionally it could return more details about the exception.
    // The additional information could be used by administrators who access the
    // endpoint with a browser, or using a ping utility that can display the
    // additional information.
    return new HttpStatusCodeResult((int)HttpStatusCode.InternalServerError);
  }
  return new HttpStatusCodeResult((int)HttpStatusCode.OK);
}
```

Az `ObscurePath` metódus azt mutatja be, hogyan olvashatók az elérési útvonalak az alkalmazás konfigurációjából, és hogyan használhatók végpontként a tesztekhez. Ez a C# nyelven írt példa azt is bemutatja, hogyan fogadhatók el az azonosítók paraméterként és használhatók a kérések érvényességének ellenőrzésére.

```csharp
public ActionResult ObscurePath(string id)
{
  // The id could be used as a simple way to obscure or hide the endpoint.
  // The id to match could be retrieved from configuration and, if matched,
  // perform a specific set of tests and return the result. If not matched it
  // could return a 404 (Not Found) status.

  // The obscure path can be set through configuration to hide the endpoint.
  var hiddenPathKey = CloudConfigurationManager.GetSetting("Test.ObscurePath");

  // If the value passed does not match that in configuration, return 404 (Not Found).
  if (!string.Equals(id, hiddenPathKey))
  {
    return new HttpStatusCodeResult((int)HttpStatusCode.NotFound);
  }

  // Else continue and run the tests...
  // Return results from the core services test.
  return this.CoreServices();
}
```

A `TestResponseFromConfig` metódus azt mutatja be, hogyan tehet elérhetővé olyan végpontot, amely egy adott konfigurációs beállítás értékét ellenőrzi.

```csharp
public ActionResult TestResponseFromConfig()
{
  // Health check that returns a response code set in configuration for testing.
  var returnStatusCodeSetting = CloudConfigurationManager.GetSetting(
                                                          "Test.ReturnStatusCode");

  int returnStatusCode;

  if (!int.TryParse(returnStatusCodeSetting, out returnStatusCode))
  {
    returnStatusCode = (int)HttpStatusCode.OK;
  }

  return new HttpStatusCodeResult(returnStatusCode);
}
```

## <a name="monitoring-endpoints-in-azure-hosted-applications"></a>Végpontok monitorozása az Azure által üzemeltetett alkalmazásokban

Néhány lehetőség a végpontok monitorozására az Azure-alkalmazásokban:

- Használja az Azure beépített monitorozási funkcióit.

- Használjon külső szolgáltatást vagy keretrendszert, például a Microsoft System Center Operations Managert.

- Hozzon létre egyéni segédprogramot vagy szolgáltatást, amely saját vagy üzemeltetett kiszolgálón fut.

   >  Bár az Azure viszonylag átfogó monitorozási lehetőségeket kínál, részletes információkért további szolgáltatásokat és eszközöket is használhat. Az Azure Management Services egy beépített monitorozási mechanizmust biztosít a riasztási szabályokhoz. Az Azure Portal felügyeleti szolgáltatási oldalának Riasztások szakasza előfizetésenként akár tíz riasztási szabály konfigurálását is lehetővé teszi a szolgáltatásokhoz. Ezek a szabályok egy feltételt és egy küszöbértéket adnak meg az olyan szolgáltatásoknak, mint például a processzorhasználat, vagy meghatározzák a másodpercenkénti kérések vagy hibák számát, ezenkívül a szolgáltatás automatikusan e-mailes értesítéseket küldhet a szabályban megadott címekre.

A monitorozható feltételek az alkalmazáshoz választott üzemeltetési mechanizmustól függően eltérőek lehetnek (például Web Sites, Cloud Services, Virtual Machines vagy Mobile Services), de ezek mindegyike képes riasztási szabályokat létrehozni, amelyek a szolgáltatás beállításaiban megadott webes végpontokat használják. Ennek a végpontnak időben kell válaszolnia, hogy a riasztási rendszer észlelhesse, hogy az alkalmazás megfelelően működik.

> További információ a [riasztási értesítések létrehozásáról][portal-alerts].

Ha az Azure Cloud Services webes és feldolgozói szerepkörökben vagy Virtual Machines-környezetben üzemelteti az alkalmazást, kihasználhatja az egyik beépített Azure-szolgáltatás, a Traffic Manager előnyeit. A Traffic Manager egy útválasztó- és terheléselosztási szolgáltatás, amely képes kéréseket szétosztani a Cloud Services által üzemeltetett alkalmazás adott példányai között különböző szabályok és beállítások alapján.

Az útválasztási kérelmek mellett a Traffic Manager pingel egy URL-címet, egy portot és egy relatív elérési útvonalat, amelyeket rendszeres időközönként megadhat, hogy megállapítsa, az alkalmazás mely, a szabályokban megadott példányai aktívak és válaszolnak a kérésekre. Ha a 200-as (OK) állapotkódot észleli, elérhetőnek jelöli az alkalmazást. A Traffic Manager minden más állapotkód esetén offline állapotúként jelöli az alkalmazást. A Traffic Manager konzolján megtekintheti az állapotot, és konfigurálhatja a szabályt, hogy átirányítsa a kéréseket az alkalmazás más példányaira, amelyek válaszolnak.

A Traffic Manager azonban csak tíz másodpercig vár a monitorozó URL-cím válaszára. Éppen ezért győződjön meg róla, hogy a rendszer az időkorláton belül futtatja az állapot-ellenőrzési kódot, ezzel lehetővé téve a hálózaton belüli adatváltási késést a Traffic Managerből az alkalmazásba, majd pedig vissza.

> További információ a [Traffic Manager alkalmazások monitorozására való használatáról](/azure/traffic-manager/). A Traffic Managerről a [több adatközpont üzembe helyezéséről szóló útmutatóban](https://msdn.microsoft.com/library/dn589779.aspx) is olvashat.

## <a name="related-guidance"></a>Kapcsolódó útmutatók

Az alábbi útmutató hasznos lehet ennek a mintának az implementálása során:

- [Rendszerállapot és telemetria – útmutató](https://msdn.microsoft.com/library/dn589775.aspx). A szolgáltatások és összetevők állapotának ellenőrzése általában mintavétel útján történik, de emellett hasznos lehet érvényes információkkal rendelkezni az alkalmazásteljesítmény monitorozásához és a futtatás során bekövetkező események észleléséhez. Ezeket az adatokat az állapotfigyeléssel kapcsolatos tovább információkként vissza lehet küldeni a monitorozási eszközökbe. A rendszerállapotra és telemetriára vonatkozó útmutató bemutatja, hogyan lehet távolról az alkalmazások rendszerállapot-figyelése által gyűjtött diagnosztikai adatokat gyűjteni.
- [Riasztási értesítések fogadása][portal-alerts].
- Ez a minta egy letölthető [mintaalkalmazást](https://github.com/mspnp/cloud-design-patterns/tree/master/health-endpoint-monitoring) tartalmaz.

[portal-alerts]: /azure/azure-monitor/platform/alerts-metric
