---
title: "Szolgáltatásspecifikus útmutatás az újrapróbálkozási mechanizmushoz"
description: "Szolgáltatás beállítása az újrapróbálkozási mechanizmussal vonatkozó konkrét útmutatást."
author: dragon119
ms.date: 07/13/2016
pnp.series.title: Best Practices
ms.openlocfilehash: 6bb623bd8be89573178f250570407bf83d62c098
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/23/2018
---
# <a name="retry-guidance-for-specific-services"></a>Újrapróbálkozási útmutatás adott szolgáltatásoknál

Az Azure-szolgáltatások és az ügyfél SDK-k tartalmaznak egy újrapróbálkozási mechanizmus. Azonban ezek eltérnek, mert minden egyes szolgáltatás eltérő jellemzőkkel és a követelmények, és így minden újrapróbálkozási mechanizmussal van beállítva, hogy egy adott szolgáltatás. Ez az útmutató az Azure-szolgáltatások többsége a újrapróbálkozási mechanizmus szolgáltatásokat foglalja, és segítséget nyújtanak a használata, igazítja, vagy az újrapróbálkozási mechanizmussal, hogy a szolgáltatás kiterjesztése tartalmaz.

Átmeneti kezel, és újra próbálkozik a kapcsolatokat és a szolgáltatások és erőforrások műveleteket általános útmutatást lásd: [útmutatást újra](./transient-faults.md).

A következő táblázat összefoglalja az ebben az útmutatóban leírt Azure-szolgáltatásokhoz tartozó újrapróbálkozási szolgáltatásokat.

| **Szolgáltatás** | **Ismételje meg a képességek** | **Házirend-konfiguráció** | **Hatókör** | **Telemetria szolgáltatások** |
| --- | --- | --- | --- | --- |
| **[Azure Storage](#azure-storage-retry-guidelines)** |Natív ügyfél |Programozott |Ügyfél és az egyes műveletek |TraceSource |
| **[Az Entity Framework SQL-adatbázis](#sql-database-using-entity-framework-6-retry-guidelines)** |Natív ügyfél |Programozott |Globális egy AppDomain tartományban |None |
| **[Az Entity Framework Core SQL-adatbázis](#sql-database-using-entity-framework-core-retry-guidelines)** |Natív ügyfél |Programozott |Globális egy AppDomain tartományban |Nincs |
| **[Az ADO.NET SQL-adatbázis](#sql-database-using-adonet-retry-guidelines)** |[Polly](#transient-fault-handling-with-polly) |Programozott és deklaratív |Egyetlen utasítások vagy kódblokkokat |Egyéni |
| **[Service Bus](#service-bus-retry-guidelines)** |Natív ügyfél |Programozott |Namespace Manager, az üzenetkezelési gyárból és az ügyfél |ETW |
| **[Azure Redis Cache](#azure-redis-cache-retry-guidelines)** |Natív ügyfél |Programozott |Ügyfél |TextWriter |
| **[Cosmos DB](#cosmos-db-retry-guidelines)** |Natív szolgáltatásban |Non-configurable |Globális |TraceSource |
| **[Azure Search](#azure-storage-retry-guidelines)** |Natív ügyfél |Programozott |Ügyfél |ETW vagy egyéni |
| **[Azure Active Directory](#azure-active-directory-retry-guidelines)** |Natív az ADAL-könyvtár |Az ADAL-könyvtár beágyazása |Belső |None |
| **[Service Fabric](#service-fabric-retry-guidelines)** |Natív ügyfél |Programozott |Ügyfél |Nincs | 
| **[Azure Event Hubs](#azure-event-hubs-retry-guidelines)** |Natív ügyfél |Programozott |Ügyfél |Nincs |

> [!NOTE]
> Az Azure beépített része próbálja meg újból a mechanizmusok, jelenleg nem tudja alkalmazni a különböző típusú hiba különböző újrapróbálkozási házirendje vagy a kivétel a funkciók túl az újrapróbálkozási házirendet. Ezért ajánlott útmutatások érhetők el írásának időpontjában, az optimális átlagos teljesítményt és rendelkezésre állást biztosító házirendet konfigurálhat. Egy a házirend finomhangolását módja elemzése a naplófájlokat, és határozza meg az átmeneti előforduló hibák típusát. Például ha hálózati problémák kapcsolódó hibák többségének, akkor előfordulhat, hogy egy azonnali újrapróbálkozási kísérlet ahelyett Várjon, amíg az első újrapróbálkozásnál hosszú ideig.
>
>

## <a name="azure-storage-retry-guidelines"></a>Az Azure Storage újrapróbálkozási irányelvek
Az Azure storage szolgáltatások közé tartoznak a tábla és a blob storage, fájlok és tárüzenetsort.

### <a name="retry-mechanism"></a>Ismételje meg a mechanizmus
Az újrapróbálkozások egyedi REST művelet szinten történik, és az ügyfél API-implementáció szerves részét képezik. Az SDK az ügyfél tárolási osztályokat, hogy implementálja a [IExtendedRetryPolicy felület](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.aspx).

Nincsenek a felület másik implementációja. Tárolási ügyfelek kifejezetten táblák, blobok és várólisták házirendek közül választhatnak. Minden egyes használ egy másik újrapróbálkozási stratégiát, amely lényegében meghatározza az újrapróbálkozási időköz és az egyéb részleteket.

A beépített osztályok lineáris (állandó késedelem) és a véletlenszerű újrapróbálkozási időközökkel exponenciális támogatásához. Nincs is egy újrapróbálkozási házirend használható, ha egy másik folyamat kezeli a magasabb szintű újrapróbálkozások. Azonban a saját újrapróbálkozási osztályokat is létrehozható, ha nincs megadva a beépített osztályok által meghatározott követelmények.

Alternatív újrapróbálkozások váltás elsődleges és másodlagos tárolótömbök szolgáltatáskeresésre, ha írásvédett georedundáns tárolás (RA-GRS) használ, és a kérés eredménye egy újrapróbálkozást lehetővé tevő hiba. Lásd: [Azure Storage redundancia beállítások](http://msdn.microsoft.com/library/azure/dn727290.aspx) további információt.

### <a name="policy-configuration"></a>Házirend-konfiguráció
Újrapróbálkozási házirendek programozott módon. Egy tipikus eljárás akkor hozhat létre és tölthet egy **TableRequestOptions**, **BlobRequestOptions**, **FileRequestOptions**, vagy **QueueRequestOptions**  példány.

```csharp
TableRequestOptions interactiveRequestOption = new TableRequestOptions()
{
  RetryPolicy = new LinearRetry(TimeSpan.FromMilliseconds(500), 3),
  // For Read-access geo-redundant storage, use PrimaryThenSecondary.
  // Otherwise set this to PrimaryOnly.
  LocationMode = LocationMode.PrimaryThenSecondary,
  // Maximum execution time based on the business use case. 
  MaximumExecutionTime = TimeSpan.FromSeconds(2)
};
```

A kérelem beállítások példány majd beállítható az ügyfélen, és az ügyfél összes művelet a megadott beállításokat fogja használni.

```csharp
client.DefaultRequestOptions = interactiveRequestOption;
var stats = await client.GetServiceStatsAsync();
```

Az ügyfél beállításokat lehet felülbírálni úgy, hogy a kérelem beállítások osztály feltöltött példány paraméterként művelet módszerek.

```csharp
var stats = await client.GetServiceStatsAsync(interactiveRequestOption, operationContext: null);
```

Használhat egy **OperationContext** példány, adja meg a kódot a végrehajtási újbóli kísérlet esetén, és ha egy művelet befejeződött. Ez a kód össze tudják gyűjteni a naplókat és telemetriai működésével kapcsolatos adatokat.

    // Set up notifications for an operation
    var context = new OperationContext();
    context.ClientRequestID = "some request id";
    context.Retrying += (sender, args) =>
    {
      /* Collect retry information */
    };
    context.RequestCompleted += (sender, args) =>
    {
      /* Collect operation completion information */
    };
    var stats = await client.GetServiceStatsAsync(null, context);

Mellett jelző hiba megfelelő-e az újra gombra, a kiterjesztett újrapróbálkozási házirendek vissza egy **RetryContext** objektum, amely jelzi, hogy a legutóbbi kérelem eredményeit, az újrapróbálkozások száma a legközelebbi újrapróbálkozás fordul elő az elsődleges vagy másodlagos helyre (lásd az alábbi táblázatban részletei). A tulajdonságait a **RetryContext** objektum segítségével döntse el, ha, és ha egy újabb kísérletet. További részletekért lásd: [IExtendedRetryPolicy.Evaluate metódus](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.evaluate.aspx).

Az alábbi táblázatok bemutatják az alapértelmezett beállításokat az a beépített házirendek próbálkozzon újra.

**Lehetőségek**

| **Beállítás** | **Alapértelmezett érték** | Jelentése |
| --- | --- | --- |
| MaximumExecutionTime | 120 másodperc | A kérelem, beleértve az összes lehetséges újrapróbálkozási kísérletek maximális végrehajtási ideje. |
| ServerTimeout | Nincs | A kérés a kiszolgáló időkorlátja (kerekített másodperc). Ha nincs megadva, az alapértelmezett érték a kiszolgáló összes kérelem fog használni. Általában a legjobb lehetőség egy hagyja ki ezt a beállítást, hogy a kiszolgáló alapértelmezett szolgál. | 
| LocationMode | Nincs | A tárfiók létrejön az írásvédett georedundáns tárolás (RA-GRS) replikációs beállítás, ha a hely mód segítségével jelzi, hogy melyik helyen kell kapnia a kérelmet. Például ha **PrimaryThenSecondary** van megadva, kérelmek mindig küldi el az elsődleges helyre először. A kérés nem teljesíthető, ha a másodlagos helyre való továbbítás. |
| RetryPolicy | ExponentialPolicy | További információ alább olvasható az egyes lehetőségek. |

**Az exponenciális házirend** 

| **Beállítás** | **Alapértelmezett érték** | Jelentése |
| --- | --- | --- |
| maxAttempt | 3 | Újrapróbálkozások száma. |
| deltaBackoff | 4 másodperc | Vissza az indító időköz újrapróbálkozások között. A TimeSpan érték, egy véletlenszerű elem, beleértve többszörösei későbbi újrapróbálkozás használható. |
| MinBackoff | 3 másodpercenként | Hozzáadandó deltaBackoff alapján kiszámított intervallumokban próbálkozzon újra. Ez az érték nem módosítható.
| MaxBackoff | 120 másodperc | MaxBackoff akkor használatos, ha az számított újrapróbálkozási időköz érték nagyobb, mint MaxBackoff. Ez az érték nem módosítható. |

**Lineáris házirend**

| **Beállítás** | **Alapértelmezett érték** | Jelentése |
| --- | --- | --- |
| maxAttempt | 3 | Újrapróbálkozások száma. |
| deltaBackoff | 30 másodperc | Vissza az indító időköz újrapróbálkozások között. |

### <a name="retry-usage-guidance"></a>Ismételje meg a használati útmutató
A tárolási ügyfél API-ja használatával az Azure storage szolgáltatások elérésekor, vegye figyelembe a következő irányelveket:

* Házirendekkel a automatikusan újrapróbálkozik a Microsoft.WindowsAzure.Storage.RetryPolicies névtér hol a követelményeknek megfelelő. A legtöbb esetben ezek a házirendek elegendő lesz.
* Használja a **ExponentialRetry** kötegműveletek, háttérfeladatok vagy nem interaktív forgatókönyvek házirend. Ezekben az esetekben általában lehetővé teszi a több időt a szolgáltatás helyreállítása – következésképpen megnövekedett arra, a művelet, végül sikeresen lezajlottak.
* Érdemes megadni a **MaximumExecutionTime** tulajdonsága a **RequestOptions** korlátozza a teljes végrehajtási idő, de vegye figyelembe a típusát és méretét, a művelet kiválasztásakor paraméter egy időtúllépési értéket.
* Ha egy egyéni újra megvalósításához szüksége, ne hozzon létre olyan burkolók a tárolási ügyfél osztályok körül. Használja a képességek kiterjesztése a meglévő házirendek segítségével a **IExtendedRetryPolicy** felületet.
* Ha írásvédett georedundáns tárolás (RA-GRS) használ a **LocationMode** adhatja meg, hogy újból megkísérli érik el a másodlagos csak olvasható másolatát az áruházból kell az elsődleges hozzáférési sikertelen. Azonban ez a beállítás használata esetén győződjön meg, hogy az alkalmazás képes sikeresen adatokkal dolgozni, amelyek esetleg elavult, ha az elsődleges áruházból replikációs még nem fejeződött be.

Vegye figyelembe a következő beállításokkal műveletek újrapróbálkozás kezdési. Ezek a beállítások az általános célú, és, felügyelheti a műveleteit, és az értékeket a saját forgatókönyvnek megfelelően konfigurálva finomhangolhatják.  

| **Környezet** | **A minta cél E2E<br />maximális késleltetés** | **Ismételje meg a házirend** | **Beállítások** | **Értékek** | **Működési elv** |
| --- | --- | --- | --- | --- | --- |
| Interaktív, felhasználói felületén<br />előtérben vagy a |2 másodperc |Lineáris |maxAttempt<br />deltaBackoff |3<br />500 ms |Kísérlet történt az 1 - 500 ms késleltetés<br />Kísérlet történt a 2 - 500 ms késleltetés<br />Próbálja meg a 3 - 500 ms késleltetés |
| Háttér<br />vagy kötegelt |30 másodperc |Az exponenciális |maxAttempt<br />deltaBackoff |5<br />4 másodperc |Próbálja meg az 1 – ~ 3 mp késleltetés<br />Kísérlet történt a 2 – ~ 7 mp késleltetés<br />Próbálja meg a 3 - késleltetés ~ 15 másodperc |

### <a name="telemetry"></a>Telemetria
Újrapróbálkozás a rendszer naplózza a **TraceSource**. Konfigurálnia kell egy **TraceListener** az események rögzítése és a megfelelő cél a naplófájlba írja őket. Használhatja a **értékének a TextWriterTraceListener figyelőre** vagy **XmlWriterTraceListener** írni egy naplófájlba, a **EventLogTraceListener** írni a Windows eseménynaplóba, vagy a **EventProviderTraceListener** nyomkövetési adatokat írni az ETW alrendszer. Automatikus kiürítése a puffer, és az eseményeket, amelyek kerül a naplóba (például hiba, figyelmeztetés, tájékoztatás, és részletes) részletességi is konfigurálhatja. További információkért lásd: [ügyféloldali naplózás a .NET a Storage ügyféloldali kódtára](http://msdn.microsoft.com/library/azure/dn782839.aspx).

Műveletek fogadhat egy **OperationContext** példány, mely tesz elérhetővé a **újrapróbálkozás** esemény használható egyéni telemetria logika csatlakoztatni. További információkért lásd: [OperationContext.Retrying esemény](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.operationcontext.retrying.aspx).

### <a name="examples"></a>Példák
Az alábbi példakód bemutatja, hogyan hozható létre kettő **TableRequestOptions** különböző újrapróbálkozásos beállításokkal példányok; interaktív kérelmek egyet, majd egy a háttér-kérelmekre. A példa majd a készlet e két próbálja meg újra az ügyfél-házirendeket, hogy minden olyan kérelem esetében alkalmazni, és az interaktív stratégia is állít be egy adott kérelem, hogy felülírja az ügyfél alapértelmezett beállításait.

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.RetryPolicies;
using Microsoft.WindowsAzure.Storage.Table;

namespace RetryCodeSamples
{
    class AzureStorageCodeSamples
    {
        private const string connectionString = "UseDevelopmentStorage=true";

        public async static Task Samples()
        {
            var storageAccount = CloudStorageAccount.Parse(connectionString);

            TableRequestOptions interactiveRequestOption = new TableRequestOptions()
            {
                RetryPolicy = new LinearRetry(TimeSpan.FromMilliseconds(500), 3),
                // For Read-access geo-redundant storage, use PrimaryThenSecondary.
                // Otherwise set this to PrimaryOnly.
                LocationMode = LocationMode.PrimaryThenSecondary,
                // Maximum execution time based on the business use case. 
                MaximumExecutionTime = TimeSpan.FromSeconds(2)
            };

            TableRequestOptions backgroundRequestOption = new TableRequestOptions()
            {
                // Client has a default exponential retry policy with 4 sec delay and 3 retry attempts
                // Retry delays will be approximately 3 sec, 7 sec, and 15 sec
                MaximumExecutionTime = TimeSpan.FromSeconds(30),
                // PrimaryThenSecondary in case of Read-access geo-redundant storage, else set this to PrimaryOnly
                LocationMode = LocationMode.PrimaryThenSecondary
            };

            var client = storageAccount.CreateCloudTableClient();
            // Client has a default exponential retry policy with 4 sec delay and 3 retry attempts
            // Retry delays will be approximately 3 sec, 7 sec, and 15 sec
            // ServerTimeout and MaximumExecutionTime are not set

            {
                // Set properties for the client (used on all requests unless overridden)
                // Different exponential policy parameters for background scenarios
                client.DefaultRequestOptions = backgroundRequestOption;
                // Linear policy for interactive scenarios
                client.DefaultRequestOptions = interactiveRequestOption;
            }

            {
                // set properties for a specific request
                var stats = await client.GetServiceStatsAsync(interactiveRequestOption, operationContext: null);
            }

            {
                // Set up notifications for an operation
                var context = new OperationContext();
                context.ClientRequestID = "some request id";
                context.Retrying += (sender, args) =>
                {
                    /* Collect retry information */
                };
                context.RequestCompleted += (sender, args) =>
                {
                    /* Collect operation completion information */
                };
                var stats = await client.GetServiceStatsAsync(null, context);
            }
        }
    }
}
```

### <a name="more-information"></a>További információ
* [Az Azure Storage ügyfél könyvtár újrapróbálkozási házirend javaslatok](https://azure.microsoft.com/blog/2014/05/22/azure-storage-client-library-retry-policy-recommendations/)
* [A Storage ügyféloldali kódtára 2.0 – újrapróbálkozási házirendek megvalósítása](http://gauravmantri.com/2012/12/30/storage-client-library-2-0-implementing-retry-policies/)

## <a name="sql-database-using-entity-framework-6-retry-guidelines"></a>SQL-adatbázis Entity Framework 6 újrapróbálkozási szerint
SQL-adatbázis egy olyan üzemeltetett SQL-adatbázis széles méretű és (megosztott) standard és premium (nem megosztott) szolgáltatás is elérhető. Entitás-keretrendszer egy objektum relációs leképező, amely lehetővé teszi a .NET-fejlesztők relációs adatok tartományspecifikus-objektumok segítségével. Szükségtelenné teszi, a legtöbb a fejlesztők általában kell írnia az adat-hozzáférési kód.

### <a name="retry-mechanism"></a>Ismételje meg a mechanizmus
Újrapróbálkozási támogatás áll rendelkezésre, használja az Entity Framework 6.0 SQL-adatbázis elérésekor és magasabb mechanizmuson keresztül nevű [kapcsolati rugalmasságot / újra logika](http://msdn.microsoft.com/data/dn456835.aspx). Az újrapróbálkozási mechanizmus a fő jellemzői a következők:

* Az elsődleges absztrakciós van a **IDbExecutionStrategy** felületet. Ez az interfész:
  * Meghatározza a szinkron és aszinkron **Execute*** módszerek.
  * Meghatározza az osztályokat is használható közvetlenül vagy egy adatbázis-környezet, mint egy alapértelmezett stratégia konfigurált, leképezve a szolgáltató neve, vagy leképezve a szolgáltató nevét és a kiszolgáló nevét. Ha a környezet konfigurálva, próbálkozások egyéni adatbázis-művelet, amelynek lehet több szinten egy adott környezetben a művelethez.
  * Határozza meg, majd ismételje meg a sikertelen kapcsolódás, mikor és hogyan.
* Ez magában foglalja a számos beépített implementációja a **IDbExecutionStrategy** felület:
  * Alapértelmezett – nincs újrapróbálkozás.
  * Alapértelmezett SQL-adatbázis (automatikus) – nincs újrapróbálkozás, de kivételek megvizsgálja, és azokat a javaslatát, hogy az SQL-adatbázis stratégia burkolja.
  * SQL-adatbázis észlelési logika és az alapértelmezett SQL-adatbázis - exponenciális (alaposztály öröklődik).
* Az exponenciális vissza ki stratégiát véletlenszerűsítésének valósítja meg.
* A beépített újrapróbálkozási osztályok állapot-nyilvántartó és nem többszálú futtatásra. Azonban ismét felhasználhatók a jelenlegi művelet befejeződése után.
* Ha a megadott újrapróbálkozások maximális számát, az eredményeket egy új kivétel van csomagolni. Nem a jelenlegi kivétel mentése buborék.

### <a name="policy-configuration"></a>Házirend-konfiguráció
Újrapróbálkozási nyújtott támogatás a használó Entity Framework 6.0 vagy újabb SQL-adatbázis elérésekor. Újrapróbálkozási házirendek programozott módon. A konfiguráció nem módosítható művelet alapon.

A környezet alapértelmezett stratégia konfigurálásakor meg kell adnia egy függvényt, amely létrehoz egy új stratégia igény szerint. A következő kód bemutatja, hogyan hozhat létre egy újrapróbálkozási configuration osztály, amely kiterjeszti a **DbConfiguration** alaposztály.

```csharp
public class BloggingContextConfiguration : DbConfiguration
{
  public BlogConfiguration()
  {
    // Set up the execution strategy for SQL Database (exponential) with 5 retries and 4 sec delay
    this.SetExecutionStrategy(
         "System.Data.SqlClient", () => new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(4)));
  }
}
```

Ezután megadhatja ezt az alapértelmezett újrapróbálkozási stratégia segítségével minden műveletnél a **SetConfiguration** metódusában a **DbConfiguration** példány az alkalmazás indításakor. Alapértelmezés szerint EF automatikusan felderíti és a configuration osztály használata.

    DbConfiguration.SetConfiguration(new BloggingContextConfiguration());

Az újrapróbálkozási configuration osztály kontextus megadhatja a környezeti osztályt ellátása megjegyzésekkel egy **DbConfigurationType** attribútum. Azonban ha csak egy konfigurációs osztály, EF használ, nincs szükség a környezet megjegyzésekkel.

    [DbConfigurationType(typeof(BloggingContextConfiguration))]
    public class BloggingContext : DbContext
    { ...

Ha különböző újrapróbálkozásos stratégiák műveleteket, vagy tiltsa le a konkrét műveletek az újrapróbálkozások van szüksége, a configuration osztály, amely lehetővé teszi, hogy felfüggeszti vagy stratégiák felcserélése jelölő beállításával hozhat létre a **CallContext** . A configuration osztály stratégiák váltáshoz használja ezt a jelzőt, vagy tiltsa le a stratégiát, adja meg, és egy alapértelmezett stratégia használatára. További információkért lásd: [felfüggesztése végrehajtási stratégia](http://msdn.microsoft.com/dn307226#transactions_workarounds) korlátozásai a végrehajtási stratégiák újrapróbálkozás (EF6 és újabb verziók esetében) lapján.

A megadott újrapróbálkozási stratégiák használata egyéni műveletek egy másik módszer akkor létrehozni a szükséges stratégia osztály egy példányát, és adja meg a kívánt beállításokat paramétereknek. Majd meghívása a **ExecuteAsync** metódust.

    var executionStrategy = new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(4));
    var blogs = await executionStrategy.ExecuteAsync(
        async () =>
        {
            using (var db = new BloggingContext("Blogs"))
            {
                // Acquire some values asynchronously and return them
            }
        },
        new CancellationToken()
    );

A legegyszerűbben úgy használja egy **DbConfiguration** ugyanabban a szerelvényben, keresse meg az osztály a **DbContext** osztály. Ez azonban nem megfelelő ugyanabban a környezetben van szükség esetén különböző helyzetekben, például a különböző interaktív és a háttér újrapróbálkozási stratégiát. A különböző környezetek külön alkalmazástartományok hajtható végre, ha a beépített támogatást használja osztályok a konfigurációs fájlban, vagy állítsa be explicit módon a kód használatával. A különböző környezetekben az AppDomain végre kell hajtani, ha egy egyéni megoldás lesz szükség.

További információkért lásd: [kód-alapú konfigurációban (EF6 és újabb verziók esetében)](http://msdn.microsoft.com/data/jj680699.aspx).

A következő táblázat a beépített alapértelmezett beállításainak újrapróbálkozási házirendet EF6 használatakor.

| Beállítás | Alapértelmezett érték | Jelentése |
|---------|---------------|---------|
| Szabályzat | Az exponenciális | Az exponenciális vissza-ki. |
| MaxRetryCount | 5 | Az újrapróbálkozások maximális számát. |
| MaxDelay | 30 másodperc | A maximális késleltetés, újrapróbálkozások között. Ez az érték nem befolyásolja, hogyan késések sorozatát arra az esetre vonatkoznak. Csak egy felső határa határozza meg. |
| DefaultCoefficient | 1 másodperc | Az exponenciális vissza az indító számítási együttható. Ez az érték nem módosítható. |
| DefaultRandomFactor | 1.1 | A többszöröző segítségével vegye fel azokat a véletlenszerű késleltetés minden bejegyzéshez. Ez az érték nem módosítható. |
| DefaultExponentialBase | 2 | A többszöröző, a következő késleltetési kiszámításához használt. Ez az érték nem módosítható. |

### <a name="retry-usage-guidance"></a>Ismételje meg a használati útmutató
EF6 használatával SQL-adatbázis elérésekor, vegye figyelembe a következő irányelveket:

* Válassza ki a megfelelő szolgáltatásra beállítás (megosztott vagy prémium). Egy megosztott példánnyal hosszabb, mint a szokásos csatlakozási késedelem és szabályozási más bérlők megosztott kiszolgáló kihasználtsága miatt romolhat. Ha kiszámítható teljesítményt és a megbízható kis késleltetésű műveletek szükség, vegye figyelembe, premium-lehetőség kiválasztásában.
* A rögzített időközönként stratégia használható az Azure SQL Database nem ajánlott. Ehelyett használja a exponenciális vissza az indító stratégia, mert előfordulhat, hogy túlterheltté vált a szolgáltatást, és hosszabb engedélyezése ahhoz, hogy a helyreállítás több időt.
* Válassza ki a kapcsolat és a parancs időtúllépések megfelelő értéket, kapcsolatok meghatározásakor. Az időtúllépés kiinduló, az üzleti logika tervezésével és a teszteléshez. Ez az érték módosításához idővel a mennyiségű adatot szeretne, vagy módosítsa az üzleti folyamatokat. Túl rövid időtúllépés kapcsolatok korai hibák eredményezhet, amikor az adatbázis foglalt. Túl hosszú időtúllépés előfordulhat, hogy megfelelően működik-e túl hosszú a sikertelen kapcsolódás észlelése előtti várakozási által újrapróbálkozási logika. Az időtúllépés értéke a végpontok közötti késés összetevője bár nem lehet egyszerűen megállapítani hány parancsok hajtja végre, a környezet mentése során. Az alapértelmezett időtúllépési módosíthatja úgy, hogy a **CommandTimeout** tulajdonsága a **DbContext** példány.
* Entitás-keretrendszer támogatja a konfigurációs fájlok meghatározott konfigurációk próbálkozzon újra. Azonban a maximális rugalmasság Azure-érdemes létrehozni programozott módon az alkalmazáson belül a konfigurációs. Az adott paraméterek újrapróbálkozási-házirendek, például az újrapróbálkozások és az újrapróbálkozási intervallumok száma a szolgáltatás konfigurációs fájljában tárolt, és futási időben a megfelelő házirendek létrehozására szolgál. Ez lehetővé teszi, hogy a beállításokat, anélkül, hogy újra kell indítani az alkalmazást módosítani.

Vegye figyelembe a következő beállításokkal műveletek újrapróbálkozás kezdési. A késleltetés, újrapróbálkozások (le, és létrehozott egy exponenciális sorozatot) között nem adható meg. Csak a maximális értékeket adhat meg, ahogy Itt; Ha létrehoz egy egyéni újrapróbálkozási stratégiát. Ezek a beállítások az általános célú, és, felügyelheti a műveleteit, és az értékeket a saját forgatókönyvnek megfelelően konfigurálva finomhangolhatják.

| **Környezet** | **A minta cél E2E<br />maximális késleltetés** | **Ismételje meg a házirend** | **Beállítások** | **Értékek** | **Működési elv** |
| --- | --- | --- | --- | --- | --- |
| Interaktív, felhasználói felületén<br />előtérben vagy a |2 másodperc |Az exponenciális |MaxRetryCount<br />MaxDelay |3<br />750 ms |Kísérlet történt az 1 – 0 mp késleltetés<br />Kísérlet történt a 2 – 750 ms késleltetés<br />Próbálja meg a 3 – 750 ms késleltetés |
| Háttér<br /> vagy kötegelt |30 másodperc |Az exponenciális |MaxRetryCount<br />MaxDelay |5<br />12 másodperc |Kísérlet történt az 1 – 0 mp késleltetés<br />Kísérlet történt a 2 - ~ 1 másodperc késleltetés<br />Próbálja meg a 3 - ~ 3 mp késleltetés<br />Kísérlet történt a 4 – ~ 7 mp késleltetés<br />Kísérlet történt az 5 – 12 mp késleltetés |

> [!NOTE]
> A végpontok közötti késés célokat azt feltételezik, hogy a szolgáltatáshoz való csatlakozásokat az alapértelmezett időkorlátját. Ha hosszabb ideig kapcsolat időtúllépése ad meg, a végpontok közötti késés minden újrapróbálkozási kísérlethez további időpontig bővíthető.
>
>

### <a name="examples"></a>Példák
Az alábbi példakód egy egyszerű adat-hozzáférési megoldás, amely használja az Entity Framework határozza meg. Egy adott újrapróbálkozási stratégiát állítja a nevű osztály egy példányát meghatározásával **BlogConfiguration** , amely kiterjeszti **DbConfiguration**.

```csharp
using System;
using System.Collections.Generic;
using System.Data.Entity;
using System.Data.Entity.SqlServer;
using System.Threading.Tasks;

namespace RetryCodeSamples
{
    public class BlogConfiguration : DbConfiguration
    {
        public BlogConfiguration()
        {
            // Set up the execution strategy for SQL Database (exponential) with 5 retries and 12 sec delay.
            // These values could be loaded from configuration rather than being hard-coded.
            this.SetExecutionStrategy(
                    "System.Data.SqlClient", () => new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(12)));
        }
    }

    // Specify the configuration type if more than one has been defined.
    // [DbConfigurationType(typeof(BlogConfiguration))]
    public class BloggingContext : DbContext
    {
        // Definition of content goes here.
    }

    class EF6CodeSamples
    {
        public async static Task Samples()
        {
            // Execution strategy configured by DbConfiguration subclass, discovered automatically or
            // or explicitly indicated through configuration or with an attribute. Default is no retries.
            using (var db = new BloggingContext("Blogs"))
            {
                // Add, edit, delete blog items here, then:
                await db.SaveChangesAsync();
            }
        }
    }
}
```

Az Entity Framework újrapróbálkozási mechanizmussal további példái megtalálhatók [kapcsolati rugalmasságot / újra logika](http://msdn.microsoft.com/data/dn456835.aspx).

### <a name="more-information"></a>További információ
* [Az Azure SQL adatbázis-teljesítményt és a rugalmasság útmutatója](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx)

## <a name="sql-database-using-entity-framework-core-retry-guidelines"></a>SQL-adatbázis Entity Framework Core újrapróbálkozási szerint
[Entity Framework Core](/ef/core/) van egy objektum relációs leképező, amely lehetővé teszi a .NET Core fejlesztők objektumok, tartomány-specifikus adatait. Szükségtelenné teszi, a legtöbb a fejlesztők általában kell írnia az adat-hozzáférési kód. Az Entity Framework jelen verziójában alapoktól készült, és nem automatikusan örökli a szolgáltatások EF6.x.

### <a name="retry-mechanism"></a>Ismételje meg a mechanizmus
Újrapróbálkozási támogatást biztosított használatával Entity Framework Core mechanizmus segítségével SQL-adatbázis nevű [kapcsolati rugalmasságot](/ef/core/miscellaneous/connection-resiliency). Kapcsolat rugalmassága az EF Core 1.1.0-ás lett bevezetve.

Az elsődleges absztrakciós van a `IExecutionStrategy` felületet. A végrehajtási stratégiát az SQL Server, beleértve az SQL Azure, vegye figyelembe a kivételt típusú követően újra megkísérelhető és ésszerű alapértelmezett értéket az újrapróbálkozások maximális számát, vonatkozó felszólítások közötti idő újrapróbálkozások, és így tovább.

### <a name="examples"></a>Példák

Az alábbi kód lehetővé teszi, hogy automatikus újrapróbálkozások a DbContext objektum, amely képviseli egy munkamenetet az adatbázis konfigurálásakor. 

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .UseSqlServer(
            @"Server=(localdb)\mssqllocaldb;Database=EFMiscellanous.ConnectionResiliency;Trusted_Connection=True;",
            options => options.EnableRetryOnFailure());
}
```

A következő kód bemutatja, hogyan hajtható végre, az automatikus újrapróbálkozásokat tranzakciót egy végrehajtási stratégia használatával. A tranzakció egy delegált van meghatározva. Akkor következik be, hogy átmeneti hiba, ha a végrehajtás stratégia újra a delegált lép működésbe.

```csharp
using (var db = new BloggingContext())
{
    var strategy = db.Database.CreateExecutionStrategy();

    strategy.Execute(() =>
    {
        using (var transaction = db.Database.BeginTransaction())
        {
            db.Blogs.Add(new Blog { Url = "http://blogs.msdn.com/dotnet" });
            db.SaveChanges();

            db.Blogs.Add(new Blog { Url = "http://blogs.msdn.com/visualstudio" });
            db.SaveChanges();

            transaction.Commit();
        }
    });
}
```

### <a name="more-information"></a>További információ
* [Kapcsolat rugalmassága](/ef/core/miscellaneous/connection-resiliency)
* [Adatpont - EF Core 1.1](https://msdn.microsoft.com/magazine/mt745093.aspx)

## <a name="sql-database-using-adonet-retry-guidelines"></a>Az ADO.NET újrapróbálkozási szerint SQL-adatbázis
SQL-adatbázis egy olyan üzemeltetett SQL-adatbázis széles méretű és (megosztott) standard és premium (nem megosztott) szolgáltatás is elérhető.

### <a name="retry-mechanism"></a>Ismételje meg a mechanizmus
SQL-adatbázis nincs újrapróbálkozások ADO.NET használatával elérésekor beépített támogatása rendelkezik. Azonban a visszatérési kódok a kérelmek segítségével határozza meg, egy kérelem sikertelenségének okát. SQL-adatbázis forgalomszabályozásáról további információkért lásd: [erőforrás-korlátozások az Azure SQL Database](/azure/sql-database/sql-database-resource-limits). A megfelelő hibakódok listájáért lásd: [SQL Database-ügyfélalkalmazások SQL hibakódok](/azure/sql-database/sql-database-develop-error-messages).

A Polly könyvtár segítségével valósítja meg az újrapróbálkozások SQL-adatbázis. Lásd: [Polly kezelését átmeneti hiba](#transient-fault-handling-with-polly).

### <a name="retry-usage-guidance"></a>Ismételje meg a használati útmutató
Az ADO.NET használatával SQL-adatbázis elérésekor, vegye figyelembe a következő irányelveket:

* Válassza ki a megfelelő szolgáltatásra beállítás (megosztott vagy prémium). Egy megosztott példánnyal hosszabb, mint a szokásos csatlakozási késedelem és szabályozási más bérlők megosztott kiszolgáló kihasználtsága miatt romolhat. Ha több kiszámítható teljesítményt és a megbízható kis késleltetésű műveletek szükség, vegye figyelembe, premium-lehetőség kiválasztásában.
* Győződjön meg arról, hogy elvégezhető-e a megfelelő szint vagy a hatókör inkonzisztenciát mutat, amely az adatok az idempotent műveletek elkerülése érdekében az újrapróbálkozások. Ideális esetben minden műveletet kell idempotent, így anélkül, hogy ez inkonzisztenciát megismételhető. Ha nem ez a helyzet, az újra gombra vagy a hatókör, amely lehetővé teszi, hogy az összes kapcsolódó is vonható vissza, ha egy művelet nem sikerül; módosításai hajtható végre például a tranzakciós hatókörén belül. További információkért lásd: [Cloud Service Fundamentals az adatelérési réteg – átmeneti hiba kezelése](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx#Idempotent_Guarantee).
* Egy rögzített időközönként stratégia nem ajánlott a az Azure SQL Database interaktív forgatókönyvek kivéve ahol csak néhány újrapróbálkozások nagyon rövid időközönként. Ehelyett érdemes exponenciális vissza az indító stratégia az forgatókönyv a legtöbb.
* Válassza ki a kapcsolat és a parancs időtúllépések megfelelő értéket, kapcsolatok meghatározásakor. Túl rövid időtúllépés kapcsolatok korai hibák eredményezhet, amikor az adatbázis foglalt. Túl hosszú időtúllépés előfordulhat, hogy megfelelően működik-e túl hosszú a sikertelen kapcsolódás észlelése előtti várakozási által újrapróbálkozási logika. Az időtúllépés értéke a végpontok közötti késés; összetevője az újrapróbálkozási késleltetést az újrapróbálkozási házirendet minden újrapróbálkozási kísérlethez megadott hatékonyan kerül.
* Zárja be a kapcsolat egy bizonyos újrapróbálkozások száma, még akkor is, ha vissza ki egy exponenciális használatával újrapróbálkozási logika, és próbálja megismételni a műveletet az új kapcsolat után. Többször meg ugyanazt a kapcsolatot a azonos művelet lehet egy tényező való kapcsolódási problémák léptek fel. Ez a módszer példáért lásd: [Cloud Service Fundamentals az adatelérési réteg – átmeneti hiba kezelése](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx).
* Ha kapcsolatkészletezést használatban (alapértelmezett), amely ugyanazt a kapcsolatot választ ki a készletből bezárása és a kapcsolat újbóli megnyitása után is esély van. Ha ez a helyzet, megoldási technika hívására-e a **ClearPool** metódusában a **SqlConnection** osztály megjelölni a kapcsolat nem használható fel újra. Azonban akkor tegye ezt csak azt követően több kapcsolat sikertelenül, és csak akkor, ha az adott osztály hibás kapcsolatok kapcsolódó átmeneti hibák például az SQL-időtúllépések (hibakód: -2) észlelése.
* Ha az adatok hozzáférési használ a tranzakciók által kezdeményezett **TransactionScope** példányok, az újrapróbálkozási logika nyissa meg újra a kapcsolat, és meg kell kezdeményezni egy új tranzakció hatókörében. Emiatt a Újrapróbálkozást lehetővé tevő kódblokk kell foglalnia a tranzakció teljes körét.

Vegye figyelembe a következő beállításokkal műveletek újrapróbálkozás kezdési. Ezek a beállítások az általános célú, és, felügyelheti a műveleteit, és az értékeket a saját forgatókönyvnek megfelelően konfigurálva finomhangolhatják.

| **Környezet** | **A minta cél E2E<br />maximális késleltetés** | **Ismételje meg a stratégia** | **Beállítások** | **Értékek** | **Működési elv** |
| --- | --- | --- | --- | --- | --- |
| Interaktív, felhasználói felületén<br />előtérben vagy a |2 mp |FixedInterval |Ismétlések száma<br />Újrapróbálkozás<br />Első gyors újrapróbálkozási |3<br />500 ms<br />igaz |Kísérlet történt az 1 – 0 mp késleltetés<br />Kísérlet történt a 2 - 500 ms késleltetés<br />Próbálja meg a 3 - 500 ms késleltetés |
| Háttér<br />vagy kötegelt |30 sec |ExponentialBackoff |Ismétlések száma<br />Minimális biztonsági kikapcsolása<br />Maximális vissza kikapcsolása<br />Különbözeti biztonsági kikapcsolása<br />Első gyors újrapróbálkozási |5<br />0 (mp)<br />60 másodperc<br />2 mp<br />hamis |Kísérlet történt az 1 – 0 mp késleltetés<br />Kísérlet történt a 2 - ~ 2 mp késleltetés<br />Próbálja meg a 3 - ~ 6 mp késleltetés<br />Kísérlet történt a 4 – ~ 14 mp késleltetés<br />Kísérlet történt az 5 – kb. 30 másodperc késleltetés |

> [!NOTE]
> A végpontok közötti késés célokat azt feltételezik, hogy a szolgáltatáshoz való csatlakozásokat az alapértelmezett időkorlátját. Ha hosszabb ideig kapcsolat időtúllépése ad meg, a végpontok közötti késés minden újrapróbálkozási kísérlethez további időpontig bővíthető.
>
>

### <a name="examples"></a>Példák
Ez a szakasz azt mutatja, hogyan használják a Polly Azure SQL Database szolgáltatáshoz konfigurált újrapróbálkozási házirendjei eléréséhez a `Policy` osztály.

A következő kód egy kiterjesztésmetódus jeleníti meg a `SqlCommand` osztály, amely meghívja a `ExecuteAsync` az exponenciális leállítási.

```csharp
public async static Task<SqlDataReader> ExecuteReaderWithRetryAsync(this SqlCommand command)
{
    GuardConnectionIsNotNull(command);

    var policy = Policy.Handle<Exception>().WaitAndRetryAsync(
        retryCount: 3, // Retry 3 times
        sleepDurationProvider: attempt => TimeSpan.FromMilliseconds(200 * Math.Pow(2, attempt - 1)), // Exponential backoff based on an initial 200ms delay.
        onRetry: (exception, attempt) => 
        {
            // Capture some info for logging/telemetry.  
            logger.LogWarn($"ExecuteReaderWithRetryAsync: Retry {attempt} due to {exception}.");
        });

    // Retry the following call according to the policy.
    await policy.ExecuteAsync<SqlDataReader>(async token =>
    {
        // This code is executed within the Policy 

        if (conn.State != System.Data.ConnectionState.Open) await conn.OpenAsync(token);
        return await command.ExecuteReaderAsync(System.Data.CommandBehavior.Default, token);

    }, cancellationToken);
}

```

Az aszinkron kiterjesztésmetódus a következőképpen használható.

```csharp
var sqlCommand = sqlConnection.CreateCommand();
sqlCommand.CommandText = "[some query]";

using (var reader = await sqlCommand.ExecuteReaderWithRetryAsync())
{
    // Do something with the values
}
```

### <a name="more-information"></a>További információ
* [Cloud Service Fundamentals az adatelérési réteg – átmeneti hiba kezelése](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx)

A legtöbb lekérése SQL-adatbázis általános útmutatást lásd: [Azure SQL adatbázis-teljesítményt és a rugalmasság útmutató](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx).


## <a name="service-bus-retry-guidelines"></a>A Service Bus újrapróbálkozási irányelvek
A Service Bus akkor felhő üzenetkezelési platform, amely nagyobb méretezés és rugalmasság lazán összekapcsolt üzenet exchange biztosít az alkalmazások összetevői a felhőben, vagy a helyszínen üzemeltetett is.

### <a name="retry-mechanism"></a>Ismételje meg a mechanizmus
A Service Bus használatával implementációja újrapróbálkozások valósítja meg a [RetryPolicy](http://msdn.microsoft.com/library/microsoft.servicebus.retrypolicy.aspx) alaposztály. Összes Service Bus ügyfél teszi közzé a **RetryPolicy** tulajdonság, amely a implementációja egyikére állítható be a **RetryPolicy** alaposztály. A beépített hitelesítés megvalósításához a következők:

* A [RetryExponential osztály](http://msdn.microsoft.com/library/microsoft.servicebus.retryexponential.aspx). Ez a biztonsági ki időköz, az újrapróbálkozások maximális számát, szabályozó tulajdonságok közzététele és a **TerminationTimeBuffer** tulajdonság, amely a teljes idő, a művelet elvégzéséhez korlátozására szolgál.
* A [NoRetry osztály](http://msdn.microsoft.com/library/microsoft.servicebus.noretry.aspx). Ez használatos, amikor a Service Bus API-szintű ismételt próbálkozás esetén nincs szükség, például ha az újrapróbálkozások által felügyelt egy másik folyamat köteg vagy több lépés művelet részeként.

A Service Bus műveletek visszatérhet a kivételek, számos, a [függelék: üzenetküldési kivételek](http://msdn.microsoft.com/library/hh418082.aspx). A listában a további információk arról, hogy mely Ha ezek határozzák meg, majd próbálja megismételni a műveletet nem megfelelő. Például egy [ServerBusyException](http://msdn.microsoft.com/library/microsoft.servicebus.messaging.serverbusyexception.aspx) azt jelzi, hogy az ügyfél kell Várjon egy ideig, majd próbálja megismételni a műveletet. A előfordulása egy **ServerBusyException** is hatására a Service Bus egy másik mód, amelyben egy extra 10 másodperces késleltetési hozzáadódik a számított újrapróbálkozási késést. Ebben a módban a rövid ideig alaphelyzetbe áll.

A kivételek teszi közzé a Service Bus által visszaadott a **IsTransient** tulajdonságot, amely jelzi, ha az ügyfél érdemes újra megpróbálnia a művelet. A beépített **RetryExponential** házirend támaszkodik a **IsTransient** tulajdonságot a **MessagingException** osztály, amely az összes Service Bus-kivételek alaposztálya. Egyéni implementációja létrehozásakor a **RetryPolicy** használhatja a kivétel típusát kombinációja alaposztály és az **IsTransient** tulajdonság adja meg a részletesebb vezérlés újrapróbálkozási keresztül műveletek. Például találta a **QuotaExceededException** és a szükséges műveletek a várólista kiürítése előtt üzenetet küld.

### <a name="policy-configuration"></a>Házirend-konfiguráció
Újrapróbálkozási házirendek programozott módon van beállítva, és az alapértelmezett házirend állítható be egy **NamespaceManager** és egy **MessagingFactory**, vagy külön-külön az egyes üzenetküldési ügyfelek. Az alapértelmezett újrapróbálkozási házirend beállítását egy üzenetkezelési munkamenet beállíthatja a **RetryPolicy** , a **NamespaceManager**.

    namespaceManager.Settings.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                                 maxBackoff: TimeSpan.FromSeconds(30),
                                                                 maxRetryCount: 3);

Az üzenetkezelési gyár alapján létrehozott összes ügyfél alapértelmezett újrapróbálkozási házirend beállításához állítsa be a **RetryPolicy** , a **MessagingFactory**.

    messagingFactory.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                        maxBackoff: TimeSpan.FromSeconds(30),
                                                        maxRetryCount: 3);

Az üzenetküldési ügyfél újrapróbálkozási házirend beállítását, vagy az alapértelmezett házirend felülbírálása, beállíthatja a **RetryPolicy** tulajdonságon keresztül a kívánt házirend osztály egy példányát:

```csharp
client.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                            maxBackoff: TimeSpan.FromSeconds(30),
                                            maxRetryCount: 3);
```

Az újrapróbálkozási házirendje nem állítható be, az egyedi művelet szintjén. Az üzenetküldési ügyfél minden műveletet vonatkozik.
A következő táblázat a beépített alapértelmezett beállításainak újrapróbálkozási házirendet.

| Beállítás | Alapértelmezett érték | Jelentése |
|---------|---------------|---------|
| Szabályzat | Az exponenciális | Az exponenciális vissza-ki. |
| MinimalBackoff | 0 | Minimális biztonsági ki időköz. Ez az újrapróbálkozási időköz deltaBackoff alapján kiszámított kerül. |
| MaximumBackoff | 30 másodperc | Maximális vissza az indító időköz. MaximumBackoff akkor használatos, ha az számított újrapróbálkozási időköz érték nagyobb, mint MaxBackoff. |
| DeltaBackoff | 3 másodpercenként | Vissza az indító időköz újrapróbálkozások között. A timespan többszörösei későbbi újrapróbálkozás használható. |
| TimeBuffer | 5 másodperc | A megszakítási idő puffer társított az újra gombra. Ha a hátralévő idő TimeBuffer kisebb újrapróbálkozások elhagyásra kerül. |
| MaxRetryCount | 10 | Az újrapróbálkozások maximális számát. |
| ServerBusyBaseSleepTime | 10 másodperc | Ha az utolsó kivétel történt a **ServerBusyException**, ez az érték nem kerülnek be a számított újrapróbálkozási időközt. Ez az érték nem módosítható. |

### <a name="retry-usage-guidance"></a>Ismételje meg a használati útmutató
Vegye figyelembe az alábbi iránymutatásokat, amikor a Service Bus használatával:

* A beépített használatakor **RetryExponential** végrehajtására, tegye implementálja egy tartalék műveletet, mert a házirend reagál a kiszolgáló elfoglalt kivételeket, és automatikusan egy megfelelő újrapróbálkozási módra vált.
* A Service Bus névterek megfeleltetni, amely a biztonsági mentési üzenetsorba automatikus feladatátvétel különálló névtérben levő elsődleges névtér várólista meghibásodásakor nevezett szolgáltatással támogatja. A másodlagos várólistában lévő üzenetek küldhető vissza az elsődleges queue amikor azt állítja helyre. Ez a szolgáltatás segítségével történő címet átmeneti hibák. További információkért lásd: [aszinkron üzenetkezelési mintákat és magas rendelkezésre állású](http://msdn.microsoft.com/library/azure/dn292562.aspx).

Vegye figyelembe a következő beállításokkal műveletek újrapróbálkozás kezdési. Ezek a beállítások az általános célú, és, felügyelheti a műveleteit, és az értékeket a saját forgatókönyvnek megfelelően konfigurálva finomhangolhatják.

| Környezet | Példa maximális késleltetés | Újrapróbálkozási házirend | Beállítások | Működés |
|---------|---------|---------|---------|---------|
| Interaktív, a felhasználói felület vagy a előtér | 2 másodperc *  | Az exponenciális | MinimumBackoff = 0 <br/> MaximumBackoff = 30 másodperc. <br/> DeltaBackoff = 300 msec. <br/> TimeBuffer = 300 ms. <br/> MaxRetryCount = 2 | 1. kísérlet: Késleltetés 0 másodperc. <br/> 2. kísérlet: Késleltetés ~ 300 ms. <br/> Attempt 3: Delay ~900 msec. |
| Háttér vagy kötegelt | 30 másodperc | Az exponenciális | MinimumBackoff = 1 <br/> MaximumBackoff = 30 másodperc. <br/> DeltaBackoff 1,75 mp =. <br/> TimeBuffer = 5 másodperc. <br/> MaxRetryCount = 3 | 1. kísérlet: Késleltetés ~ 1 másodperc. <br/> 2. kísérlet: ~ 3 mp késleltetés. <br/> 3. kísérlet: Késleltetés ~ 6 MS. <br/> Attempt 4: Delay ~13 msec. |

\* További késleltetés, ha a kiszolgáló elfoglalt választ hozzáadott nem beleértve.

### <a name="telemetry"></a>Telemetria
A Service Bus újrapróbálkozások naplózza az ETW-eseményként is használ egy **EventSource**. Hozzá kell rendelni egy **eseményfigyelő Visszahívásán** az események rögzítése és megtekinthetők a teljesítmény-megjelenítőben, vagy azokat a megfelelő cél naplóba bejegyezni esemény forrását. Használhatja a [szemantikai naplózás alkalmazás blokk](http://msdn.microsoft.com/library/dn775006.aspx) ehhez. A kísérletek naplózása a következő formátumot követi a következők:

```text
Microsoft-ServiceBus-Client/RetryPolicyIteration
ThreadID="14,500"
FormattedMessage="[TrackingId:] RetryExponential: Operation Get:https://retry-tests.servicebus.windows.net/TestQueue/?api-version=2014-05 at iteration 0 is retrying after 00:00:00.1000000 sleep because of Microsoft.ServiceBus.Messaging.MessagingCommunicationException: The remote name could not be resolved: 'retry-tests.servicebus.windows.net'.TrackingId:6a26f99c-dc6d-422e-8565-f89fdd0d4fe3, TimeStamp:9/5/2014 10:00:13 PM."
trackingId=""
policyType="RetryExponential"
operation="Get:https://retry-tests.servicebus.windows.net/TestQueue/?api-version=2014-05"
iteration="0"
iterationSleep="00:00:00.1000000"
lastExceptionType="Microsoft.ServiceBus.Messaging.MessagingCommunicationException"
exceptionMessage="The remote name could not be resolved: 'retry-tests.servicebus.windows.net'.TrackingId:6a26f99c-dc6d-422e-8565-f89fdd0d4fe3,TimeStamp:9/5/2014 10:00:13 PM"
```

### <a name="examples"></a>Példák
Az alábbi példakód bemutatja, hogyan állítsa be a műveletek újrapróbálkozási házirendje esetében:

* Egy névtér-kezelőt. A házirend vonatkozik, hogy a kezelő az összes műveletet, és nem bírálható felül az egyes műveletek.
* Egy üzenetkezelési gyárból. A házirend alapján, hogy a gyári létrehozott összes ügyfelére érvényes, és nem bírálható felül az egyes ügyfelek létrehozásakor.
* Az egyéni üzenetküldési ügyfél. Ügyfél létrehozása után az újrapróbálkozási házirendet beállíthatja, hogy az ügyfélszámítógépek számára. A házirend vonatkozik, hogy az ügyfél összes művelete.

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.ServiceBus;
using Microsoft.ServiceBus.Messaging;

namespace RetryCodeSamples
{
    class ServiceBusCodeSamples
    {
        private const string connectionString =
            @"Endpoint=sb://[my-namespace].servicebus.windows.net/;
                SharedAccessKeyName=RootManageSharedAccessKey;
                SharedAccessKey=C99..........Mk=";

        public async static Task Samples()
        {
            const string QueueName = "TestQueue";

            ServiceBusEnvironment.SystemConnectivity.Mode = ConnectivityMode.Http;

            var namespaceManager = NamespaceManager.CreateFromConnectionString(connectionString);

            // The namespace manager will have a default exponential policy with 10 retry attempts
            // and a 3 second delay delta.
            // Retry delays will be approximately 0 sec, 3 sec, 9 sec, 25 sec and the fixed 30 sec,
            // with an extra 10 sec added when receiving a ServiceBusyException.

            {
                // Set different values for the retry policy, used for all operations on the namespace manager.
                namespaceManager.Settings.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(0),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);

                // Policies cannot be specified on a per-operation basis.
                if (!await namespaceManager.QueueExistsAsync(QueueName))
                {
                    await namespaceManager.CreateQueueAsync(QueueName);
                }
            }


            var messagingFactory = MessagingFactory.Create(
                namespaceManager.Address, namespaceManager.Settings.TokenProvider);
            // The messaging factory will have a default exponential policy with 10 retry attempts
            // and a 3 second delay delta.
            // Retry delays will be approximately 0 sec, 3 sec, 9 sec, 25 sec and the fixed 30 sec,
            // with an extra 10 sec added when receiving a ServiceBusyException.

            {
                // Set different values for the retry policy, used for clients created from it.
                messagingFactory.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(1),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);


                // Policies cannot be specified on a per-operation basis.
                var session = await messagingFactory.AcceptMessageSessionAsync();
            }


            {
                var client = messagingFactory.CreateQueueClient(QueueName);
                // The client inherits the policy from the factory that created it.


                // Set different values for the retry policy on the client.
                client.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(0.1),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);


                // Policies cannot be specified on a per-operation basis.
                var session = await client.AcceptMessageSessionAsync();
            }
        }
    }
}
```

### <a name="more-information"></a>További információ
* [Aszinkron üzenetkezelési mintát és magas rendelkezésre állás](http://msdn.microsoft.com/library/azure/dn292562.aspx)

## <a name="azure-redis-cache-retry-guidelines"></a>Azure Redis Cache újrapróbálkozási irányelvek
Azure Redis Cache egy gyors adatok elérése és alacsony késési igényű cache szolgáltatás a népszerű nyílt forráskódú Redis Cache alapján. Biztonságos, Microsoft által felügyelt és az Azure-ban minden alkalmazás elérhető-e.

Ez a szakasz az útmutató alapján a StackExchange.Redis-ügyfelet használó gyorsítótár-hozzáféréshez. Megfelelő ügyfelek listáját megtalálja az a [Redis webhely](http://redis.io/clients), és ezek különböző újrapróbálkozásos mechanizmusok.

Vegye figyelembe, hogy a StackExchange.Redis ügyfél használ-e az multiplexáló egyetlen kapcsolaton keresztül. Az ajánlott használati, hogy az ügyfél példányt létrehozni az alkalmazás indításakor, és ezt a példányt használja a gyorsítótár minden műveletnél. Emiatt a gyorsítótárba kapcsolat csak egyszer, és így az összes ebben a szakaszban található útmutatót az újrapróbálkozási házirendet a kezdeti kapcsolat kapcsolatos – és nem az egyes műveletek, aki hozzáfér a gyorsítótárban.

### <a name="retry-mechanism"></a>Ismételje meg a mechanizmus
A StackExchange.Redis ügyfél használja egy kapcsolat manager osztály, amely olyan beállítási lehetőségekkel, incuding konfigurálva:

- **ConnectRetry**. A száma a gyorsítótárban nem sikerült kapcsolódni a rendszer megpróbálja újból.
- **ReconnectRetryPolicy**. Az újrapróbálkozási stratégiát kíván használni.
- **ConnectTimeout**. A maximális várakozási idő milliszekundumban.

### <a name="policy-configuration"></a>Házirend-konfiguráció
Újrapróbálkozási házirendek programozott módon a beállításainak az ügyfél a gyorsítótár való csatlakozás előtt. Hozzon létre egy példányát erre a **ConfigurationOptions** osztály, feltöltése a tulajdonságait, és úgy, hogy átadja a **Connect** metódust.

A beépített osztályok lineáris (állandó) késleltetés és exponenciális leállítási újrapróbálkozási véletlenszerű időközökkel támogatja. Egyéni újrapróbálkozási házirendje implementálásával is létrehozhat a **IReconnectRetryPolicy** felületet.

Az alábbi példa használatával exponenciális leállítási újrapróbálkozási stratégiát konfigurálja.

```csharp
var deltaBackOffInMilliseconds = TimeSpan.FromSeconds(5).Milliseconds;
var maxDeltaBackOffInMilliseconds = TimeSpan.FromSeconds(20).Milliseconds;
var options = new ConfigurationOptions
{
    EndPoints = {"localhost"},
    ConnectRetry = 3,
    ReconnectRetryPolicy = new ExponentialRetry(deltaBackOffInMilliseconds, maxDeltaBackOffInMilliseconds),
    ConnectTimeout = 2000
};
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

Másik lehetőségként adja meg a beállításokat egy karakterlánc, és adja át a a **Connect** metódust. Vegye figyelembe, hogy a **ReconnectRetryPolicy** tulajdonság nem állítható be ezzel a módszerrel csak a kódot.

```csharp
    var options = "localhost,connectRetry=3,connectTimeout=2000";
    ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

Közvetlenül a gyorsítótárba csatlakoztatásakor is megadható beállítások.

```csharp
var conn = ConnectionMultiplexer.Connect("redis0:6380,redis1:6380,connectRetry=3");
```

További információkért lásd: [verem Exchange Redis konfigurációja](https://stackexchange.github.io/StackExchange.Redis/Configuration) a StackExchange.Redis dokumentációjában.

A következő táblázat a beépített alapértelmezett beállításainak újrapróbálkozási házirendet.

| **Környezet** | **Beállítás** | **Alapértelmezett érték**<br />(v 1.2.2) | Jelentése |
| --- | --- | --- | --- |
| ConfigurationOptions |ConnectRetry<br /><br />ConnectTimeout<br /><br />SyncTimeout<br /><br />ReconnectRetryPolicy |3<br /><br />Legfeljebb 5000 ms plus SyncTimeout<br />1000<br /><br />5000 ms LinearRetry |Ismétlődő hányszor kísérletek csatlakozzon a kezdeti kapcsolat művelet során.<br />Időkorlát (ms) a műveletek csatlakozzon. Nem a késleltetés újrapróbálkozási kísérletek között.<br />Idő (ms) szinkron műveletek engedélyezése.<br /><br />Ismételje meg minden 5000 ms.|

> [!NOTE]
> A szinkron műveletek `SyncTimeout` adhat hozzá a végpontok közötti késés, de ha az értéket túl alacsonyra jelentős időtúllépéseket okozhat. Lásd: [hibaelhárítása az Azure Redis Cache][redis-cache-troubleshoot]. Általánosságban elmondható kerülje a szinkron műveletek, és használja helyette az aszinkron műveletek. További információ: [folyamatok és Multiplexers](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/PipelinesMultiplexers.md).
>
>

### <a name="retry-usage-guidance"></a>Ismételje meg a használati útmutató
Azure Redis Cache használata esetén, vegye figyelembe a következő irányelveket:

* A StackExchange Redis ügyfél felügyeli a saját újrapróbálkozások, de csak ha a kapcsolat létrehozása a gyorsítótárba az alkalmazás első indításakor. Konfigurálhatja a kapcsolat időkorlátja, újrapróbálkozási kísérleteinek száma és a kapcsolat létrehozása az újrapróbálkozások között eltelt idő, de az újrapróbálkozási házirendje nem vonatkozik a gyorsítótár műveleteket.
* Számos újrapróbálkozás helyett fontolja meg, de visszatér az eredeti adatforrás helyette elérésével.

### <a name="telemetry"></a>Telemetria
Akkor is gyűjthet információt használatával kapcsolatok (de nem más műveletek) egy **TextWriter**.

```csharp
var writer = new StringWriter();
...
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

Ezt követően a kimeneti példát alább láthatók.

```text
localhost:6379,connectTimeout=2000,connectRetry=3
1 unique nodes specified
Requesting tie-break from localhost:6379 > __Booksleeve_TieBreak...
Allowing endpoints 00:00:02 to respond...
localhost:6379 faulted: SocketFailure on PING
localhost:6379 failed to nominate (Faulted)
> UnableToResolvePhysicalConnection on GET
No masters detected
localhost:6379: Standalone v2.0.0, master; keep-alive: 00:01:00; int: Connecting; sub: Connecting; not in use: DidNotRespond
localhost:6379: int ops=0, qu=0, qs=0, qc=1, wr=0, sync=1, socks=2; sub ops=0, qu=0, qs=0, qc=0, wr=0, socks=2
Circular op-count snapshot; int: 0 (0.00 ops/s; spans 10s); sub: 0 (0.00 ops/s; spans 10s)
Sync timeouts: 0; fire and forget: 0; last heartbeat: -1s ago
resetting failing connections to retry...
retrying; attempts left: 2...
...
```

### <a name="examples"></a>Példák
Az alábbi példakód újrapróbálkozások, amikor a StackExchange.Redis-ügyfél inicializálása állandó (lineáris) késleltetés konfigurálja. Ez a példa bemutatja, hogyan telepíthet a használt konfiguráció egy **ConfigurationOptions** példány.

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using StackExchange.Redis;

namespace RetryCodeSamples
{
    class CacheRedisCodeSamples
    {
        public async static Task Samples()
        {
            var writer = new StringWriter();

            {
                try
                {
                    var retryTimeInMilliseconds = TimeSpan.FromSeconds(4).Milliseconds; // delay between retries
                    
                    // Using object-based configuration.
                    var options = new ConfigurationOptions
                                        {
                                            EndPoints = { "localhost" },
                                            ConnectRetry = 3,
                                            ReconnectRetryPolicy = new LinearRetry(retryTimeInMilliseconds)
                                        };
                    ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);

                    // Store a reference to the multiplexer for use in the application.
                }
                catch
                {
                    Console.WriteLine(writer.ToString());
                    throw;
                }
            }
        }
    }
}
```

A következő példában a konfigurációs beállítások megadásával karakterláncként állítja be. A kapcsolat időtúllépési érték a várakozási idő a kapcsolatot a gyorsítótárba, nem újrapróbálkozási kísérletek között eltelt maximális időtartam. Vegye figyelembe, hogy a **ReconnectRetryPolicy** tulajdonság csak kód által állítható be.

```csharp
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using StackExchange.Redis;

namespace RetryCodeSamples
{
    class CacheRedisCodeSamples
    {
        public async static Task Samples()
        {
            var writer = new StringWriter();

            {
                try
                {
                    // Using string-based configuration.
                    var options = "localhost,connectRetry=3,connectTimeout=2000";
                    ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);

                    // Store a reference to the multiplexer for use in the application.
                }
                catch
                {
                    Console.WriteLine(writer.ToString());
                    throw;
                }
            }
        }
    }
}
```

További példákért lásd [konfigurációs](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Configuration.md#configuration) a projekt webhelyen.

### <a name="more-information"></a>További információ
* [Webhely redis](http://redis.io/)

## <a name="cosmos-db-retry-guidelines"></a>Cosmos DB újrapróbálkozási irányelvek

Cosmos DB egy olyan teljes körűen felügyelt több modellre adatbázis, amely támogatja a séma nélküli JSON-adatokat. Teljesítménye konfigurálható és megbízható, natív JavaScript-tranzakciófeldolgozást kínál, és mivel felhőbeli felhasználásra készült, rugalmasan méretezhető.

### <a name="retry-mechanism"></a>Ismételje meg a mechanizmus
A `DocumentClient` osztály automatikusan újrapróbálkozik a sikertelen kísérletek. Az újrapróbálkozások száma és a maximális várakozási idő beállításához konfigurálása [ConnectionPolicy.RetryOptions]. Kivételek, amely kiváltja az ügyfél újrapróbálkozási házirend túl vagy nem átmeneti hibák.

Ha Cosmos DB azelőtt gyorsítja fel, az ügyfél, HTTP 429 hibát ad vissza. Az állapotkód: Ellenőrizze a `DocumentClientException`.

### <a name="policy-configuration"></a>Házirend-konfiguráció
A következő táblázat alapértelmezett beállításait a `RetryOptions` osztály.

| Beállítás | Alapértelmezett érték | Leírás |
| --- | --- | --- |
| MaxRetryAttemptsOnThrottledRequests |9 |Ha a kérés nem teljesíthető, mivel a Cosmos DB sebessége korlátozza az ügyfél az újrapróbálkozások maximális száma. |
| MaxRetryWaitTimeInSeconds |30 |A maximális idő másodpercben. Próbálkozzon újra. |

### <a name="example"></a>Példa
```csharp
DocumentClient client = new DocumentClient(new Uri(endpoint), authKey); ;
var options = client.ConnectionPolicy.RetryOptions;
options.MaxRetryAttemptsOnThrottledRequests = 5;
options.MaxRetryWaitTimeInSeconds = 15;
```

### <a name="telemetry"></a>Telemetria
Újrapróbálkozások bejelentkezett egy .NET keresztül strukturálatlan nyomkövetési üzenetekként **TraceSource**. Konfigurálnia kell egy **TraceListener** az események rögzítése és a megfelelő cél a naplófájlba írja őket.

Például ha az App.config fájlban adja hozzá a következő, nyomkövetési adatokat a rendszer létrehoz és a végrehajtható fájl ugyanazon a helyen a fájlt:

```
<configuration>
  <system.diagnostics>
    <switches>
      <add name="SourceSwitch" value="Verbose"/>
    </switches>
    <sources>
      <source name="DocDBTrace" switchName="SourceSwitch" switchType="System.Diagnostics.SourceSwitch" >
        <listeners>
          <add name="MyTextListener" type="System.Diagnostics.TextWriterTraceListener" traceOutputOptions="DateTime,ProcessId,ThreadId" initializeData="CosmosDBTrace.txt"></add>
        </listeners>
      </source>
    </sources>
  </system.diagnostics>
</configuration>
```


## <a name="azure-search-retry-guidelines"></a>Az Azure Search újrapróbálkozási irányelvek
Az Azure Search segítségével hatékony és kifinomult keresési-képességeket adhat a webhelyhez vagy alkalmazáshoz, gyorsan és egyszerűen finomhangolhatják a keresési eredmények között, és gazdag és finomhangolható rangsorolási modellek létrehozásához.

### <a name="retry-mechanism"></a>Ismételje meg a mechanizmus
Az Azure Search SDK-ban való újrapróbálkozásra vezérli a `SetRetryPolicy` metódust a [SearchServiceClient] és [SearchIndexClient] osztályok. Az alapértelmezett házirend ismétlések exponenciális leállítási, ha Azure Search egy 5xx vagy 408 (kérése időtúllépés) választ ad vissza.

### <a name="telemetry"></a>Telemetria
Nyomkövetési ETW vagy egy egyéni nyomkövetési szolgáltató regisztrálja. További információkért lásd: a [AutoRest dokumentáció][autorest].

## <a name="azure-active-directory-retry-guidelines"></a>Az Azure Active Directory újrapróbálkozási irányelvek
Azure Active Directory (Azure AD) egy olyan átfogó identitás- és hozzáférés felügyeleti felhőalapú megoldás, hogy a címtárszolgáltatások mag, speciális identitás irányítás, biztonsági és alkalmazáshozzáférés-kezeléshez. Az Azure AD ezenkívül identitáskezelő platformot kínál a fejlesztőknek, hogy központi házirendeken és szabályokon alapuló hozzáférés-szabályozással bővíthessék az alkalmazásaikat.

### <a name="retry-mechanism"></a>Ismételje meg a mechanizmus
Az Azure Active Directory a az Active Directory Authentication Library (ADAL) beépített újrapróbálkozási mechanizmussal van. Váratlan zárolásokat elkerülése érdekében azt javasoljuk, hogy külső gyártótól származó kódtárak és alkalmazáskód hajthatja végre **nem** próbálja meg újra a hibás kapcsolatok, de lehetővé teszi az ADAL újrapróbálkozások kezelésére. 

### <a name="retry-usage-guidance"></a>Ismételje meg a használati útmutató
Amikor az Azure Active Directoryval, vegye figyelembe az alábbi irányelveket:

* Ha lehetséges, használja az ADAL-könyvtár és a beépített támogatása az ismételt próbálkozás.
* Ha az Azure Active Directory a REST API-t használ, érdemes újra megpróbálnia a művelet csak akkor, ha a 5xx tartomány (például 500 belső kiszolgálóhiba 502 Hibás átjáró, 503-as szolgáltatás nem érhető el vagy 504-es számú átjáró időtúllépése) hiba eredménye. Nem próbálja meg újra más hibákat.
* Az exponenciális vissza az indító házirend ajánlott kötegelt olyan esetekben, az Azure Active Directoryban.

Vegye figyelembe a következő beállításokkal műveletek újrapróbálkozás kezdési. Ezek a beállítások az általános célú, és, felügyelheti a műveleteit, és az értékeket a saját forgatókönyvnek megfelelően konfigurálva finomhangolhatják.

| **Környezet** | **A minta cél E2E<br />maximális késleltetés** | **Ismételje meg a stratégia** | **Beállítások** | **Értékek** | **Működési elv** |
| --- | --- | --- | --- | --- | --- |
| Interaktív, felhasználói felületén<br />előtérben vagy a |2 mp |FixedInterval |Ismétlések száma<br />Újrapróbálkozás<br />Első gyors újrapróbálkozási |3<br />500 ms<br />igaz |Kísérlet történt az 1 – 0 mp késleltetés<br />Kísérlet történt a 2 - 500 ms késleltetés<br />Próbálja meg a 3 - 500 ms késleltetés |
| Háttér vagy<br />batch |60 másodperc |ExponentialBackoff |Ismétlések száma<br />Minimális biztonsági kikapcsolása<br />Maximális vissza kikapcsolása<br />Különbözeti biztonsági kikapcsolása<br />Első gyors újrapróbálkozási |5<br />0 (mp)<br />60 másodperc<br />2 mp<br />hamis |Kísérlet történt az 1 – 0 mp késleltetés<br />Kísérlet történt a 2 - ~ 2 mp késleltetés<br />Próbálja meg a 3 - ~ 6 mp késleltetés<br />Kísérlet történt a 4 – ~ 14 mp késleltetés<br />Kísérlet történt az 5 – kb. 30 másodperc késleltetés |

### <a name="more-information"></a>További információ
* [Az Azure Active Directory hitelesítési Kódtárai][adal]

## <a name="service-fabric-retry-guidelines"></a>A Service Fabric újrapróbálkozási irányelvek

A Service Fabric megbízható szolgáltatások terjesztése fürt védi a cikkben szereplő lehetséges átmeneti hibák többségét ellen. Átmeneti hibákat azonban továbbra is lehetséges, amelyek. Például a naming service közepén útválasztási módosítása onnan kapta, hogy a kérelmet, ha okozza, hogy kivételt jelez. Ha a kérésben 100 milliomod másodperc később, valószínűleg sikeres lesz.

A Service Fabric belsőleg, az ilyen átmeneti hiba kezeli. Egyes beállítások segítségével konfigurálhatja a `OperationRetrySettings` osztály a szolgáltatások beállítása közben.  A következő kód egy példát mutat be. A legtöbb esetben ne legyen szükséges, és az alapértelmezett beállításokat sikeres lesz.

```csharp
    FabricTransportRemotingSettings transportSettings = new FabricTransportRemotingSettings
    {
        OperationTimeout = TimeSpan.FromSeconds(30)
    };

    var retrySettings = new OperationRetrySettings(TimeSpan.FromSeconds(15), TimeSpan.FromSeconds(1), 5);

    var clientFactory = new FabricTransportServiceRemotingClientFactory(transportSettings);

    var serviceProxyFactory = new ServiceProxyFactory((c) => clientFactory, retrySettings);

    var client = serviceProxyFactory.CreateServiceProxy<ISomeService>(
        new Uri("fabric:/SomeApp/SomeStatefulReliableService"),
        new ServicePartitionKey(0));
```

## <a name="more-information"></a>További információ

* [Távoli kivételkezelést](https://github.com/Microsoft/azure-docs/blob/master/articles/service-fabric/service-fabric-reliable-services-communication-remoting.md#remoting-exception-handling)


## <a name="azure-event-hubs-retry-guidelines"></a>Az Azure Event Hubs újra irányelvek

Az Azure Event Hubs egy rendkívül nagy kapacitású, telemetriai adatokat betöltő szolgáltatás, amely események millióinak adatait képes összegyűjteni, átalakítani és tárolni.

### <a name="retry-mechanism"></a>Ismételje meg a mechanizmus
Az Azure Event Hubs ügyféloldali kódtár a újrapróbálkozásra vezérli a `RetryPolicy` tulajdonságát a `EventHubClient` osztály. Az alapértelmezett exponenciális leállítási, amikor az Azure Event Hubs adja vissza egy átmeneti házirend ismétlések `EventHubsException` vagy egy `OperationCanceledException`.

### <a name="example"></a>Példa
```csharp
EventHubClient client = EventHubClient.CreateFromConnectionString("[event_hub_connection_string]");
client.RetryPolicy = RetryPolicy.Default;
```

### <a name="more-information"></a>További információ
[ Az Azure Event Hubs szabványos .NET ügyféloldali kódtár](https://github.com/Azure/azure-event-hubs-dotnet)

## <a name="general-rest-and-retry-guidelines"></a>Általános REST, és próbálkozzon újra irányelveket
Azure vagy harmadik féltől származó szolgáltatások eléréséhez vegye figyelembe a következőket:

* Egy rendszeres megközelítés, az újrapróbálkozásokat lehet, hogy kódú újrafelhasználható, akkor használja, így a egységes módszer alkalmazhatók minden ügyfél és az összes megoldás.
* Fontolja meg az átmeneti hiba kezelési alkalmazás-blokk újrapróbálkozási keretrendszer használatát újrapróbálkozások kezelése, ha a célként megadott szolgáltatás vagy az ügyfél nem beépített újrapróbálkozási mechanizmussal rendelkezik. Ez segítséget nyújt egy egységes újrapróbálkozási viselkedés, és azt a célként megadott szolgáltatás egy megfelelő alapértelmezett újrapróbálkozási stratégiát rendelkezhetnek. Azonban szükség lehet, hogy létrehozzon egyéni újrapróbálkozási kódot azon szolgáltatások, amelyek nem szabványos viselkedését, ne használja a kivételeket úgy, hogy átmeneti hiba azt jelzi, vagy ha szeretné használni egy **újrapróbálkozási-válasz** válasz újrapróbálkozásra kezeléséhez.
* Az átmeneti észlelési logika az ügyfél tényleges API meghívása a REST-hívások segítségével függ. Egyes ügyfelek, például a újabb **HttpClient** osztály, nem egy nem sikeres HTTP-állapotkód: teljesített kérelmek kivételeinek kivételhibát. Ez javítja a teljesítményt, de megakadályozza, hogy az átmeneti hiba kezelési alkalmazás-blokk használata. Ebben az esetben burkolása sikerült kóddal, amely nem sikeres HTTP-állapotkódok, amely a blokk majd tudja feldolgozni a kivételeket a REST API-hívás. Egy másik mechanizmus segítségével azt is megteheti, az újbóli próbálkozások meghajtó.
* A szolgáltatás által visszaadott HTTP-állapotkód: jelzi, hogy a hiba átmeneti segítséget. Előfordulhat, hogy egy ügyfél vagy a újrapróbálkozási keretrendszer állapotkód eléréséhez, vagy a megfelelő kivétel típusának meghatározására által létrehozott kivételek vizsgálata szükséges. A következő HTTP-kódok általában azt jelzi, hogy egy újabb megfelelő:
  * 408 kérelmi időkorlátot.
  * 500 Internal Server Error
  * 502 Hibás átjáró
  * 503 Service Unavailable
  * 504-es számú átjáró időtúllépése
* Ha a kivételek az újrapróbálkozási logika, a következő általában jelzi, hogy egy átmeneti hiba, ha nem sikerült kapcsolatot:
  * WebExceptionStatus.ConnectionClosed
  * WebExceptionStatus.ConnectFailure
  * WebExceptionStatus.Timeout
  * WebExceptionStatus.RequestCanceled
* A szolgáltatás nem érhető el állapota esetén a szolgáltatás a megfelelő késleltetés jelezhetik az újrapróbálkozás előtt a **újrapróbálkozási után** válaszfejléc vagy egy másik egyéni fejlécet. Szolgáltatások további információkat is előfordulhat, hogy egyéni fejlécek küldeni, vagy a válasz tartalmának ágyazva. Az átmeneti hiba kezelési alkalmazás-blokk nem használhatja a szabványos vagy egyéni "újrapróbálkozási-után" fejléc.
* Nem próbálja meg újra az ügyfél-hibák (a 4xx tartomány hibák) jelölő állapotkódok 408 kérelem időtúllépés kivételével.
* Alaposan tesztelje a újra azokat a stratégiákat és a feltételek, például a különböző hálózati állapotok és a különböző rendszer terhelések számos szerinti.

### <a name="retry-strategies"></a>Ismételje meg a stratégiák
A tipikus objektumtípusokra újrapróbálkozási stratégia intervallumok a következők:

* **Az exponenciális**: a megadott számú ki megközelítés véletlenszerű exponenciális biztonsági használatával az intervallum újrapróbálkozások között, az újrapróbálkozásokat végző újrapróbálkozási házirendje. Példa:

        var random = new Random();

        var delta = (int)((Math.Pow(2.0, currentRetryCount) - 1.0) *
                    random.Next((int)(this.deltaBackoff.TotalMilliseconds * 0.8),
                    (int)(this.deltaBackoff.TotalMilliseconds * 1.2)));
        var interval = (int)Math.Min(checked(this.minBackoff.TotalMilliseconds + delta),
                       this.maxBackoff.TotalMilliseconds);
        retryInterval = TimeSpan.FromMilliseconds(interval);
* **Növekményes**: a megadott számú újrapróbálkozások és a próbálkozások növekményes időközét újrapróbálkozási stratégiát. Példa:

        retryInterval = TimeSpan.FromMilliseconds(this.initialInterval.TotalMilliseconds +
                       (this.increment.TotalMilliseconds * currentRetryCount));
* **LinearRetry**: használja a megadott időközönként újrapróbálkozások között, az újrapróbálkozásokat megadott számú végző újrapróbálkozási házirendje. Példa:

        retryInterval = this.deltaBackoff;

### <a name="more-information"></a>További információ
* [Áramköri megszakító stratégiák](http://msdn.microsoft.com/library/dn589784.aspx)

## <a name="transient-fault-handling-with-polly"></a>Átmeneti hiba Polly kezelését
[Polly](http://www.thepollyproject.org) egy függvénytár programozottan kezelni az újrapróbálkozások és [áramköri megszakító] [ circuit-breaker] stratégiák. A Polly projekt tagja a [.NET Foundation][dotnet-foundation]. Ha az ügyfél nem natív módon támogatja az újrapróbálkozások szolgáltatások esetén Polly érvényes alternatíva, és kiküszöböli, egyéni újrapróbálkozási kód, amely implementálja megfelelően nehéz lehet írni. Polly is lehetőséget biztosít nyomkövetési hibák azok bekövetkezésekor, így újrapróbálkozások bejelentkezhet.

<!-- links -->

[adal]: /azure/active-directory/develop/active-directory-authentication-libraries
[autorest]: https://github.com/Azure/autorest/tree/master/docs
[circuit-breaker]: ../patterns/circuit-breaker.md
[ConnectionPolicy.RetryOptions]: https://msdn.microsoft.com/library/azure/microsoft.azure.documents.client.connectionpolicy.retryoptions.aspx
[dotnet-foundation]: https://dotnetfoundation.org/
[polly]: http://www.thepollyproject.org
[redis-cache-troubleshoot]: /azure/redis-cache/cache-how-to-troubleshoot
[SearchIndexClient]: https://msdn.microsoft.com/library/azure/microsoft.azure.search.searchindexclient.aspx
[SearchServiceClient]: https://msdn.microsoft.com/library/microsoft.azure.search.searchserviceclient.aspx
