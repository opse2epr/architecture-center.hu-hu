---
title: "Végpont állapotfigyelés"
description: "Funkcionális ellenőrzések megvalósítását a külső eszközök rendszeres időközönként kitett végpontok keresztül elérhető alkalmazást."
keywords: "Kialakítási mintája"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- availability
- management-monitoring
- resiliency
ms.openlocfilehash: 36171d568b9b5bfbbd48ee762b16adea695cf0e9
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="health-endpoint-monitoring-pattern"></a>Állapotfigyelő Végpontmonitoring kijelző minta

[!INCLUDE [header](../_includes/header.md)]

Funkcionális ellenőrzések megvalósítását a külső eszközök rendszeres időközönként kitett végpontok keresztül elérhető alkalmazást. Ez segít annak ellenőrzése, hogy alkalmazások és szolgáltatások megfelelően.

## <a name="context-and-problem"></a>A környezetben, és probléma

Akkor célszerű, és gyakran üzleti igény, a figyelő webes alkalmazások és a háttér-szolgáltatásaihoz, annak érdekében, hogy fontosságúak elérhető és teljesítő megfelelően. Azonban célszerű bonyolultabb, mint a helyszíni szolgáltatások megfigyelése a felhőben futó szolgáltatások figyelésére. Például nem rendelkezik teljes körű hozzáférést engedélyezzenek az üzemeltetési környezetben, és a szolgáltatások általában más platform szállítók és mások által nyújtott szolgáltatások függnek.

Nincsenek számos tényező befolyásolja a felhőben üzemeltetett alkalmazások, például a hálózati késés, a teljesítmény és rendelkezésre állását, valamint a mögöttes számítási és tárolási rendszerek közötti hálózati sávszélesség. A szolgáltatás részben vagy teljesen meghiúsulhatnak ezek a tényezők miatt. Ezért ellenőrizze rendszeres időközönként, hogy a szolgáltatás megfelelően működik-e a szükséges szintű rendelkezésre állását, amelyet a szolgáltatásiszint-szerződéssel (SLA) része lehet.

## <a name="solution"></a>Megoldás

Megvalósítása állapotfigyelés kérelmeket küld az alkalmazás egy végpontot. Az alkalmazás kell a szükséges ellenőrzések elvégzéséhez, és térjen vissza a utalhat, hogy annak állapotát.

Egy állapotfigyelési ellenőrzése általában egyesíti a két tényező:

- (Ha van ilyen) ellenőrzi az az alkalmazás vagy szolgáltatás állapotának ellenőrzése végpontjához a irányuló kérelemre adott válasz.
- Ellenőrizze az eszköz vagy keretrendszer, amely végrehajtja a rendszerállapot-ellenőrzés az eredmények elemzéséhez.

A válaszkód azt jelzi, az alkalmazást, és szükség esetén összetevők állapotát, vagy a szolgáltatások azt használja. A késleltetés vagy válasz idő ellenőrzése történik a figyelési eszközzel vagy a keretrendszer. Az ábra áttekintést nyújt a minta.

![A minta áttekintése](./_images/health-endpoint-monitoring-pattern.png)

Más ellenőrzi, hogy előfordulhat, hogy végzi a állapotfigyelési kódot az alkalmazás belefoglalása:
- Ellenőrzését felhőalapú tároló- és adatbázis rendelkezésre állásával és válaszidejével.
- Más erőforrásokhoz vagy szolgáltatásokhoz ellenőrzése található az alkalmazásban vagy máshol található, de az alkalmazás által használt.

Szolgáltatások és eszközök érhetők el, amely a webalkalmazások figyeléséhez a végpontok csoportja kérelem elküldése, és az eredmények konfigurálható szabályok készlete alapján értékelik. Már viszonylag egyszerű, amelynek egyetlen célja, hogy a rendszer néhány működési tesztek kerülnek végrehajtásra szolgáltatás-végpont létrehozása.

Tipikus ellenőrzi, hogy a végzi el a felügyeleti eszközök közé tartoznak:

- Ez a válaszkód ellenőrzése. Például egy HTTP-válasz 200-as (OK) azt jelzi, hogy az alkalmazás hiba nélkül válaszolt. A felügyeleti rendszer is megkereshet többi válaszkódnál való átfogóbb eredményt ad.
- Észleli a hibákat válasz tartalmának ellenőrzése, akkor is, ha a 200-as (OK) állapotkód adja vissza. Ez csak a visszaadott weblapon vagy a szolgáltatás válasza szakasza érintő hibák észlelését. Például egy lap címe ellenőrzése, vagy csak adott kifejezések, amely jelzi a megfelelő lapon adott vissza.
- Méri a válaszideje, amely megadja, hogy a hálózati késés kombinációja és az alkalmazás a kérelem végrehajtásának idejét. Az érték növelése azt jelzi, hogy az alkalmazás vagy a hálózati feltörekvő hibát.
- Ellenőrzési erőforrásokhoz vagy szolgáltatásokhoz, az alkalmazások, például egy tartalomkézbesítési hálózat tartalmat továbbít a globális gyorsítótárak az alkalmazás által használt kívül található.
- SSL-tanúsítvány lejáratának ellenőrzése.
- A DNS késés és a DNS-hibák az alkalmazás URL-címének DNS-címkeresést válaszideje méri.
- Annak biztosítása érdekében a megfelelő bejegyzéseket a DNS-címkeresés által visszaadott az URL-CÍMEK érvényesítése. Ez segít rosszindulatú kérelem átirányításához a DNS-kiszolgálón egy sikeres támogatás keresztül elkerülése érdekében.

Azt is hasznos, ha lehetséges, ezek futtatásához ellenőrzi a különböző helyszíni vagy üzemeltetett helyek mérni, és hasonlítsa össze a válaszidők. Ideális esetben célszerű figyelemmel kísérni az ügyfelek az egyes helyeken egy pontos képet a teljesítmény eléréséhez a hamarosan helyekről alkalmazások. Csupán egy sokkal hatékonyabban ellenőrzési mechanizmus, az eredmények segíthetnek eldönteni, az alkalmazás központi telepítési helyéről&mdash;és a telepítés több adatközpont-e.

Tesztek is futtatni kell a szolgáltatás példányainak használó ügyfelek annak érdekében, hogy az alkalmazás megfelelően működjön az összes ügyfél számára. Például ha ügyfél tárolási egynél több tárfiókot van elosztva, a megfigyelési folyamat ellenőrizze mindegyik.

## <a name="issues-and-considerations"></a>Problémákat és szempontok

Ebben a mintában megvalósításához meghatározásakor, vegye figyelembe a következő szempontokat:

Hogyan lehet érvényesíteni a választ. Például az ellenőrzése, hogy az alkalmazás megfelelően működik-e elegendő csupán egyetlen 200 (OK) állapotkódot? A legalapvetőbb mértéke rendelkezésre állását biztosítja, és ebben a mintában a minimális általi implementációja, a műveletek, a trendek és a lehetséges jövőbeli problémák az alkalmazásban kevés információt biztosít.

   >  Győződjön meg arról, hogy az alkalmazás megfelelően adja vissza egy 200 (OK) csak akkor, ha a cél erőforráson található, és feldolgozni. Bizonyos esetekben például ha mesterlapra segítségével tárolhatja a célként megadott weblapot, a kiszolgáló visszaküld egy 200 (OK) állapotkód helyett a 404-es (nem található) kód, még akkor is, ha a cél lap tartalom nem található.

A végpontok teszi közzé az alkalmazás száma. Egy megoldás, az alkalmazás által használt alapvető szolgáltatások legalább egy végpontot, és egy másik alacsonyabb prioritású virtuális gép szolgáltatások, így minden figyelési eredmény hozzárendelését fontosságú teszi közzé. Is érdemes lehet további végpontok, például minden alapvető szolgáltatásához tartozó további figyelési granularitási teszi ki. Például egy állapotának ellenőrzése az adatbázis, tárolási és egy külső geokódolás szolgáltatás, amely egy alkalmazást használ, a különböző szintű üzemidő és válasz ideje igénylő ellenőrizze. Az alkalmazás továbbra is lesz kifogástalan, ha a geokódolás szolgáltatás, vagy egy másik háttérfeladat nem érhető el néhány percig.

Használjon-e az azonos végpont figyelés általános hozzáféréshez használt, de egy adott helyre tervezett állapotának ellenőrzése, például /HealthCheck/ {GUID} / az általános hozzáférési végponton. Ez lehetővé teszi, hogy néhány működési teszt a Hálózatfigyelő eszközök, például a hozzáadása egy új felhasználói regisztráció, bejelentkezés és helyezi el egy teszt sorrendben is annak ellenőrzésekor, hogy rendelkezésre áll-e az általános hozzáférési végpont által futtatandó az alkalmazásban.

Milyen típusú információk gyűjtéséhez válaszul figyelési kéréseket, és hogyan adott vissza az adatokat a szolgáltatásban. A legtöbb meglévő eszközöket és keretrendszerek tekintse meg csak a HTTP-állapotkód, amely a végpont adja vissza. Térjen vissza, és további adatok érvényesítéséhez, lehetséges, hogy egy egyéni felügyeleti segédprogram, illetve a szolgáltatás létrehozásához.

Mennyi adat összegyűjtése. Túlzott feldolgozása során az ellenőrzés végrehajtása az alkalmazás túlterhelés, és hatással lehet más felhasználók. Szükséges idő meghaladhatja az időtúllépés a figyelési rendszer, így azt jelöli, hogy az alkalmazás nem érhető el. A legtöbb alkalmazás instrumentation közé tartoznak például hibakezelők és a teljesítmény és a hibával kapcsolatos részletes információk naplózása teljesítményszámlálókat, ez is elegendő lehet további információt visszatérése ellenőrzési állapotellenőrzése helyett.

A végpont állapota gyorsítótárazását. Lehet, olcsóbbá teszi az állapot-ellenőrzés futtatása túl gyakran. Az állapot ellenőrzése egy irányítópulton keresztül történő készként, például szeretné az irányítópulton való állapotellenőrzése kérelmek. Ehelyett rendszeres időközönként ellenőrizze a rendszerállapot és a gyorsítótár állapotát. A gyorsítótárazott állapotát visszaadó végpont tenni.

Nyilvános hozzáférés, amelyek esetleg felfedi a rosszindulatú támadások az alkalmazást, elleni védelem érdekében a figyelési végpontok biztonságának konfigurálása kockáztatja a bizalmas adatok veszélyeztetettségének, vagy vonzerőt szolgáltatásmegtagadási (DoS-) támadásokat. Általában ezt a az alkalmazás konfigurációját, hogy könnyen lehet frissíteni az alkalmazás újraindítása nélkül. Vegye figyelembe a következő módszerek közül:

- A végpont biztonságos hitelesítési kérésével. Ehhez a kérelem fejlécében hitelesítési biztonsági kulcs használata vagy a kérelem sikeres hitelesítési adatok feltéve, hogy a felügyeleti szolgáltatás, vagy az eszköz támogatja a hitelesítést.

 - Homályos vagy rejtett végpont használja. Például teszi közzé az-végpont egy másik IP-címet, az alapértelmezett alkalmazás URL-Címének használatával, konfigurálja a végpontot egy nem szabványos HTTP-portot, és/vagy egy összetett elérési útját a tesztlap használja. Általában további végpont címek és portok adja meg az alkalmazás konfigurációját, majd adja hozzá ezeket a végpontokat bejegyzést a DNS-kiszolgáló, ha ne kelljen közvetlenül adja meg az IP-cím szükséges.

 - A metódus, amely egy paraméter, például a kulcs értéke vagy egy művelet mód értékét fogadja a végpont tenni. Attól függően, hogy ennek a paraméternek adott érték egy kérelem fogadásakor. a kódot is egy adott vizsgálat vagy a teszt végrehajtása, vagy a 404-es (nem található) hibaüzenetet adja vissza, ha a paraméter értéke nem ismerhető fel. Az alkalmazás konfigurációjának sikerült beállítani a felismert paraméterértékeket.

     >  Szolgáltatásmegtagadási támadások várhatóan kisebb mértékű befolyásolása mellett egy külön végponton, amely az alapvető működési teszteket hajt végre a műveletet, az alkalmazás veszélyeztetése nélkül. Ideális esetben ne egy tesztet, amely esetleg felfedi a bizalmas adatokat. Ha egy támadó hasznos információk kell visszaadnia, vegye figyelembe, hogyan fogja védeni a végpont és az adatok a jogosulatlan hozzáférés. Ebben az esetben csak hagyatkoznia információelrejtési módszer nem elegendőek. Azt is figyelembe kell venni HTTPS-kapcsolaton keresztül, és a bizalmas adatok titkosítása bár ez a kiszolgáló terhelését növeli.

- Hogyan érhetők el olyan végponttal, amely hitelesítés használatával lett biztonságossá téve. Nem minden eszközök és keretrendszerek beállítható úgy, hogy a rendszerállapot-ellenőrző kérést hitelesítő adatokat tartalmazza. Például Microsoft Azure beépített állapotfigyelő ellenőrzési funkciók nem adhatók meg a hitelesítő adatok. Egyes külső alternatív megoldások [Pingdom](https://www.pingdom.com/), [Panopta](http://www.panopta.com/), [NewRelic](https://newrelic.com/), és [Statuscake](https://www.statuscake.com/).

- Győződjön meg arról, hogy megfelelően működik-e a figyelési ügynök módjáról. Egyik módszer – ezek általában olyan végponttal, amely egyszerűen értéket ad vissza, az alkalmazás vagy egy véletlenszerű értéket, az ügynök teszteléséhez használható.

   >  Bizonyosodjon meg arról, hogy a felügyeleti rendszer ellenőrzi, például egy önálló tesztelése és beépített teszt azt kiadó téves pozitív eredmények elkerülése érdekében.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes használni ezt a mintát

Ebben a mintában a következőkhöz hasznos:
- Figyelési webhelyek és webalkalmazások rendelkezésre állásának ellenőrzése.
- Figyelés megfelelő működéséhez ellenőrizni webhelyekhez és webes alkalmazásokhoz.
- Észleli és kiszűri a jelzett, amely zavart okozhat a többi alkalmazás középső rétegbeli vagy a megosztott szolgáltatások figyelése.
- Hozzájárul az alkalmazásban, például a teljesítményszámlálók és hibakezelők meglévő instrumentation. Állapotának ellenőrzése ellenőrzése nem cserélje le a naplózást, és az alkalmazás naplózási követelmény. Instrumentation egy meglévő keretrendszer értékes információkat nyújtanak a figyelők számlálók és a hibák vagy más olyan problémák észlelése hibanaplókat. Azonban ez nem információkkal, ha az alkalmazás nem érhető el.

## <a name="example"></a>Példa

A következő kódot példák venni a `HealthCheckController` osztály (minta bemutatja, ebben a mintában érhető el a [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/health-endpoint-monitoring)), azt mutatja be, az ilyen számos különböző Állapotellenőrzések végrehajtásához a végpont.

A `CoreServices` metódus, a C#, az alább látható végez több ellenőrzést az alkalmazásban használt szolgáltatások. Összes vizsgálat futtatása hiba nélkül, ha a metódus egy 200 (OK) állapotkód adja vissza. Kivétel vizsgálat riasztást, ha a módszer (belső hiba) 500 állapotkódot adja vissza. A metódus opcionálisan visszaadhatja további információt során hiba lép fel, ha a felügyeleti eszköz vagy keretrendszer tenni, hogy azt használja.

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
A `ObscurePath` módszer bemutatja, hogyan olvasni egy elérési utat az alkalmazás konfigurációját, és a tesztek használni a végpont. Ebben a példában a C# is bemutatja, hogyan is fogadja el paraméterként Azonosítóval és segítségével keressen érvényes kérelmeket.

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

A `TestResponseFromConfig` módszer bemutatja, hogyan fedhet olyan végponttal, amely ellenőrzi a megadott konfigurációs beállítás értéke.

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
## <a name="monitoring-endpoints-in-azure-hosted-applications"></a>Alkalmazások figyelése az Azure-végpontok üzemeltetett

Végpontok Azure-alkalmazások figyelése az egyes lehetőségek közül választhat:

- Az Azure beépített figyelési funkcióit használja.

- Egy külső szolgáltatás vagy a Microsoft System Center Operations Manager keretrendszer használja.

- Hozzon létre egy egyéni segédprogram vagy szolgáltatás, amely alapján saját vagy egy központigyorsítótár-kiszolgálón.

   >  Annak ellenére, hogy az Azure biztosít ésszerűen átfogó figyelési lehetőségek, további szolgáltatások és eszközök segítségével vonatkozó további információ. Az Azure szolgáltatások lehetővé teszi a beépített figyelési riasztási szabályok. A riasztások szakaszban az Azure portálon, a felügyeleti szolgáltatások lap lehetővé teszi a szolgáltatások előfizetésenként legfeljebb tíz riasztási szabályok konfigurálása. Ezek a szabályok egy feltételt és egy szolgáltatás, például a CPU-terhelést, vagy a kéréseket, és a hibák másodpercenkénti száma a küszöbérték megadása, és a szolgáltatás automatikusan küldhetnek értesítő e-mailek adhat meg az egyes szabályokban található.

A figyelheti feltételek eltérők lehetnek, attól függően, hogy a üzemeltetési módszert választja, az alkalmazás (például a webhelyek, a Cloud Services, a virtuális gépek vagy a Mobile Services), de ezek mindegyikének megadhatják a által használt egy webes végpont riasztási szabályt létrehozni, Adja meg a szolgáltatás beállításait. Ezt a végpontot kell kellő időben válaszolni, úgy, hogy a riasztási rendszer észleli, hogy az alkalmazás megfelelően működik-e.

>  További információt olvashat [riasztási értesítések létrehozása][portal-alerts].

Ha az alkalmazás Azure Cloud Services webes és feldolgozói szerepkörök vagy virtuális gépek működteti, kihasználhatja az egy beépített szolgáltatás az Azure hívott Traffic Manager. A TRAFFIC Manager útválasztási és a terheléselosztás szolgáltatás, amely a kérelmeket az üzemeltetett felhőalapú szolgáltatások alkalmazás alapján a szabályok és beállítások adott példányához történő elosztását.

Útválasztási kérelmek mellett a Traffic Manager pingeli egy URL-cím, egy portszám és egy relatív elérési utat, amely rendszeresen annak meghatározására, hogy melyik példányát szabályzatban meghatározott aktív adja meg, és kérésekre válaszol. Ha azt észleli, hogy egy állapotkód 200 (OK), az alkalmazás rendelkezésre jelöli meg. Bármely más állapotkód Traffic Manager segítségével jelölje meg az alkalmazás kapcsolat nélküli okoz. Az állapot megtekintése a Traffic Manager-konzolon, és konfigurálja a szabályt, hogy az alkalmazás más példányát válaszoló kérelmek átirányítása.

Traffic Manager azonban csak Várjon 10 másodpercet válasz fogadása a figyelési URL-címet. Emiatt biztosítania kell, hogy végrehajtja-e a állapotát ellenőrző kód ebbe az időszakba lehetővé teszi a hálózati késés a oda-vissza a Traffic Manager az alkalmazás és vissza újra.

>  További információt szeretne használatával [Traffic Manager segítségével figyelheti az alkalmazások](https://azure.microsoft.com/documentation/services/traffic-manager/). A TRAFFIC Manager is ismertet [több adatközpont üzembe helyezési útmutatót](https://msdn.microsoft.com/library/dn589779.aspx).

## <a name="related-guidance"></a>Kapcsolódó útmutató

Az alábbi útmutatót az ebben a mintában végrehajtása során lehet hasznos:
- [Telemetria útmutató és Instrumentation](https://msdn.microsoft.com/library/dn589775.aspx). Szolgáltatások és az összetevők állapotának ellenőrzése általában végezhető el probing, de az is, hogy az alkalmazások teljesítményének figyelésére és észleli, hogy a futási időben bekövetkező események hasznos információkat. Vissza az eszközök "figyelés" További információk a állapotfigyelés továbbítani lehet ezeket az adatokat. Telemetria útmutató és Instrumentation felderíti a instrumentation alkalmazások által gyűjtött távoli diagnosztikai adatainak összegyűjtése.
- [Riasztási értesítések fogadásának][portal-alerts].
- Ebben a mintában tartalmaz egy letölthető [mintaalkalmazás](https://github.com/mspnp/cloud-design-patterns/tree/master/health-endpoint-monitoring).

[portal-alerts]: https://azure.microsoft.com/documentation/articles/insights-receive-alert-notifications/
