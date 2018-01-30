---
title: "Nem megfelelő példányosítás kizárási minta"
description: "Elkerülheti, hogy folyamatosan új példányok jöjjenek létre olyan objektumokból, amelyeket elvileg csak egyszer kéne létrehozni, hogy utána meg lehessen őket osztani."
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: 4b217f7fc644901eb5c3e77319d151caed30eef1
ms.sourcegitcommit: cf207fd10110f301f1e05f91eeb9f8dfca129164
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/29/2018
---
# <a name="improper-instantiation-antipattern"></a>Nem megfelelő példányosítás kizárási minta

Ronthatja a teljesítményt, ha folyamatosan új példányokat hoz létre olyan objektumokból, amelyeket elvileg csak egyszer kéne létrehozni, hogy utána meg lehessen őket osztani. 

## <a name="problem-description"></a>A probléma leírása

Számos kódtár biztosít absztrakt entitásokat a külső erőforrásokhoz. Ezek az osztályok általában belsőleg kezelik az adott erőforrással kialakított saját kapcsolataikat, és olyan közvetítőként funkcionálnak, amelyeken keresztül az ügyfelek elérhetik az erőforrásokat. Néhány példa az Azure-alkalmazásoknál használható közvetítőosztályokra:

- `System.Net.Http.HttpClient`. HTTP használatával kommunikál egy webszolgáltatással.
- `Microsoft.ServiceBus.Messaging.QueueClient`. Üzeneteket küld és fogad egy Service Bus-üzenetsorból. 
- `Microsoft.Azure.Documents.Client.DocumentClient`. Kapcsolódik egy Cosmos DB-példányhoz.
- `StackExchange.Redis.ConnectionMultiplexer`. Kapcsolódik a Redishez, beleértve az Azure Redis Cache-t is.

Ezeket az osztályokat csak egyszer kell példányosítani, majd újra felhasználhatók az alkalmazás teljes élettartama során. Azonban gyakori téveszme, hogy ezeket az osztályokat csak akkor kell létrehozni, amikor épp szükség van rájuk, és gyorsan ki kell őket adni. (Az itt felsorolt osztályok .NET-kódtárak, azonban ez a jelenség nem csak a .NET-re jellemző.)

A következő ASP.NET egy `HttpClient` példányt hoz létre egy távoli szolgáltatással folytatott kommunikációhoz. A teljes kódmintát [itt][sample-app] találja.

```csharp
public class NewHttpClientInstancePerRequestController : ApiController
{
    // This method creates a new instance of HttpClient and disposes it for every call to GetProductAsync.
    public async Task<Product> GetProductAsync(string id)
    {
        using (var httpClient = new HttpClient())
        {
            var hostName = HttpContext.Current.Request.Url.Host;
            var result = await httpClient.GetStringAsync(string.Format("http://{0}:8080/api/...", hostName));
            return new Product { Name = result };
        }
    }
}
```

Egy webalkalmazáson belül ez a technika nem méretezhető. A rendszer egy új `HttpClient` objektumot hoz létre minden egyes felhasználói kéréshez. Nagy terhelés mellett a webkiszolgáló kimerítheti az elérhető szoftvercsatornák számát, ami `SocketException` hibákat eredményez.

A probléma nem korlátozódik a `HttpClient` osztályra. Az erőforrásokat burkoló, valamint azok az osztályok is hasonló hibákat okozhatnak, amelyeknek költséges a létrehozása. A következő példa egy `ExpensiveToCreateService` osztály példányait hozza létre. Itt a probléma nem feltétlenül a szoftvercsatornák kimerülése, hanem egyszerűen az egyes példányok létrehozásához szükséges idő hossza. Az osztály példányainak folyamatos létrehozása és megsemmisítése negatívan befolyásolhatja a rendszer méretezhetőségét.

```csharp
public class NewServiceInstancePerRequestController : ApiController
{
    public async Task<Product> GetProductAsync(string id)
    {
        var expensiveToCreateService = new ExpensiveToCreateService();
        return await expensiveToCreateService.GetProductByIdAsync(id);
    }
}

public class ExpensiveToCreateService
{
    public ExpensiveToCreateService()
    {
        // Simulate delay due to setup and configuration of ExpensiveToCreateService
        Thread.SpinWait(Int32.MaxValue / 100);
    }
    ...
}
```

## <a name="how-to-fix-the-problem"></a>A probléma megoldása

Ha a külső erőforrást burkoló osztály megosztható és szálbiztos, hozza létre annak egy megosztott egyszeri példányát vagy újrahasználható példányainak készletét.

A következő példában egy statikus `HttpClient` példány szerepel, amely minden kérésben megosztja a kapcsolatot.

```csharp
public class SingleHttpClientInstanceController : ApiController
{
    private static readonly HttpClient HttpClient;

    static SingleHttpClientInstanceController()
    {
        HttpClient = new HttpClient();
    }

    // This method uses the shared instance of HttpClient for every call to GetProductAsync.
    public async Task<Product> GetProductAsync(string id)
    {
        var hostName = HttpContext.Current.Request.Url.Host;
        var result = await HttpClient.GetStringAsync(string.Format("http://{0}:8080/api/...", hostName));
        return new Product { Name = result };
    }
}
```

## <a name="considerations"></a>Megfontolandó szempontok

- Az ilyen kizárási minta kulcseleme egy *megosztható* objektum példányainak ismétlődő létrehozása és megsemmisítése. Ha az adott osztály nem megosztható (nem szálbiztos), a kizárási minta nem alkalmazható.

- A megosztott erőforrás típusától függhet, hogy egyszeri példányt kell használni, vagy egy készletet kell létrehozni. A `HttpClient` osztályt megosztva és nem készletben kell használni. Más objektumok támogathatják a készletben való használatot, ami lehetővé teszi, hogy a rendszer több példányon ossza szét a számítási feladatokat.

- A több kérés között megosztott objektumoknak szálbiztosnak *kell* lenniük. A `HttpClient` osztályt így kell használni, más osztályok azonban nem feltétlenül támogatják az egyidejű kéréseket, ezért mindenképpen olvassa át a rendelkezésre álló dokumentációt.

- Egyes erőforrástípusok ritkák, és nem szabad lefoglalni őket. Ilyenek például az adatbázis-kapcsolatok. Egy megnyitott adatbázis-kapcsolat szükségtelen lefoglalása azt eredményezheti, hogy más egyidejű felhasználók nem tudják majd elérni az adatbázist.

- A .NET-keretrendszerben számos olyan objektumot, amelyek külső erőforrásokhoz építenek ki kapcsolatot, az ilyen kapcsolatokat kezelő egyéb osztályok statikus példányosító metódusai hoznak létre. Az ilyen objektumokat el kell menteni és újra kell használni, nem pedig elvetni, majd újra létrehozni. Például az Azure Service Busban a `QueueClient` objektum egy `MessagingFactory` objektumon keresztül jön létre. Belül a `MessagingFactory` kezeli a kapcsolatokat. További információkért lásd: [Ajánlott eljárások a teljesítmény javításához a Service Bus-üzenetkezelés használatával][service-bus-messaging].

## <a name="how-to-detect-the-problem"></a>A probléma észlelése

A probléma tünete lehet a teljesítmény hirtelen csökkenése vagy a hibák arányának megnövekedése, amelyet a következők valamelyike kísér: 

- A kivételek számának növekedése, amely arra utal, hogy az erőforrások, például szoftvercsatornák, adatbázis-kapcsolatok, fájlmutatók stb. kimerülőben vannak. 
- Megnövekedett mértékű memóriahasználat és szemétgyűjtés.
- A hálózati, meghajtó- és adatbázis-tevékenységek növekedése.

A következő lépések végrehajtásával azonosíthatja a problémát:

1. Az éles rendszer folyamatmonitorozásával azonosíthatja azokat a pontokat, ahol a válaszidők lelassulnak vagy a rendszer működése az erőforrások hiányai miatt meghiúsul.
2. Az ezeken a pontokon gyűjtött telemetriaadatok vizsgálatával megállapíthatja, hogy mely műveletek hoznak létre és semmisítenek meg erőforrás-igényes objektumokat.
3. Az egyes gyanús műveletek terheléstesztjét egy kontrollált tesztkörnyezetben végezze el, ne az éles rendszeren.
4. Tekintse át a forráskódot, és vizsgálja meg, hogyan kezeli a rendszer a közvetítő kódokat.

Tekintse át a rendszer nagy terheltsége esetén lassan futó vagy kivételeket eredményező műveletek hívásláncait. Ezek az információk segíthetnek megérteni, hogyan használják ezek a műveletek az erőforrásokat. A kivételek segíthetnek megállapítani, hogy a hibákat a megosztott erőforrások kimerülése okozza-e. 

## <a name="example-diagnosis"></a>Diagnosztikai példa

Az alábbi szakaszokban ezeket a lépéseket hajtjuk végre a fentebb leírt mintaalkalmazáson.

### <a name="identify-points-of-slow-down-or-failure"></a>A lassulási vagy meghiúsulási pontok azonosítása

Az alábbi ábra a [New Relic APM][new-relic] használatával létrehozott eredményeket mutatja, ahol is a gyenge válaszidejű műveletek láthatók. Ebben az esetben a `NewHttpClientInstancePerRequest` vezérlő `GetProductAsync` metódusát érdemes közelebbről is megvizsgálni. Vegyük észre, hogy ezeknek a műveleteknek a futtatásakor a hibák aránya is nő. 

![A mintaalkalmazás a New Relic monitorozási irányítópultján, amint egy HttpClient-objektum új példányát hozza létre mindegyik kéréshez][dashboard-new-HTTPClient-instance]

### <a name="examine-telemetry-data-and-find-correlations"></a>A telemetriaadatok vizsgálata és az összefüggések felderítése

A következő képen a szálak profilkészítésével gyűjtött adatok láthatóak ugyanarra az időszakra vonatkozóan, mint az előző képen. A rendszer jelentős mennyiségű időt tölt a szoftvercsatorna-kapcsolatok megnyitásával, és még ennél is többet a bezárásukkal és a szoftvercsatorna-kivételek kezelésével.

![A mintaalkalmazás a New Relic szálprofilkészítőjén, amint egy HttpClient-objektum új példányát hozza létre mindegyik kéréshez][thread-profiler-new-HTTPClient-instance]

### <a name="performing-load-testing"></a>Terhelésteszt végrehajtása

A terhelésteszttel szimulálhatja a felhasználók által végrehajtható jellemző műveleteket. Ez segíthet azonosítani, hogy a rendszer mely részein tapasztalható erőforrásfogyás a különféle terhelések mellett. A teszteket egy kontrollált környezetben hajtsa végre, ne az éles rendszeren. A következő diagramon a `NewHttpClientInstancePerRequest` vezérlő által kezelt kérések átviteli sebessége látható, ahogy az egyidejű felhasználók száma fokozatosan 100-ra nő.

![A mintaalkalmazás átviteli sebessége, amint egy HttpClient-objektum új példányát hozza létre mindegyik kéréshez][throughput-new-HTTPClient-instance]

Eleinte a másodpercenként kezelt kérések mennyisége is nő, ahogy a számítási feladatok száma növekszik. 30 felhasználó környékén azonban a sikeres kérések mennyisége elér egy határt, és a rendszerben kivételek kezdenek jelentkezni. Innentől a kivételek mennyisége fokozatosan növekszik a felhasználóterhelés növekedésével együtt. 

A terhelésteszt ezeket a hibákat HTTP 500-as (belső kiszolgáló) hibákként sorolta be. A telemetria áttekintésével kiderült, hogy a hibákat az okozta, hogy a rendszer szoftvercsatorna-erőforrásai kimerültek, ahogy egyre több és több `HttpClient` objektum jött létre.

A következő diagramon egy hasonló teszt látható egy vezérlővel, amely az egyedi `ExpensiveToCreateService` objektumot hozza létre.

![A mintaalkalmazás átviteli sebessége, amint egy ExpensiveToCreateService új példányát hozza létre mindegyik kéréshez][throughput-new-ExpensiveToCreateService-instance]

Ezúttal a vezérlő nem ad ki kivételeket, az átviteli sebesség azonban még mindig plafonba ütközik, miközben az átlagos válaszidő a 20-szorosára nő. (A diagramon a válaszidő és az átviteli sebesség logaritmikus skálán van megjelenítve.) A telemetria kimutatta, hogy a probléma elsődleges oka az `ExpensiveToCreateService` új példányainak létrehozása volt.

### <a name="implement-the-solution-and-verify-the-result"></a>A megoldás megvalósítása és az eredmény ellenőrzése

Miután beállítottuk, hogy a `GetProductAsync` metódus egyetlen `HttpClient` példányt osszon meg, a második terhelésteszt teljesítményjavulást mutatott. A rendszer nem jelzett hibákat, és képes volt kezelni a másodpercenként akár 500 kérésig növő terhelést is. Az átlagos válaszidő az előző teszthez képest a felére csökkent.

![A mintaalkalmazás átviteli sebessége, amint egy HttpClient-objektum már meglévő példányát használja az egyes kérésekhez][throughput-single-HTTPClient-instance]

Összehasonlításképpen az alábbi ábra a híváslánc-telemetriát mutatja. Ezúttal a rendszer az idő nagy részében valós munkát végez, és nem csupán szoftvercsatornákat nyit meg és zár be.

![A mintaalkalmazás a New Relic szálprofilkészítőjén, amint egy HttpClient-objektum egyetlen példányát hozza létre az összes kéréshez][thread-profiler-single-HTTPClient-instance]

A következő diagramon egy hasonló terhelésteszt eredményei láthatók az `ExpensiveToCreateService` objektum egy megosztott példányának használatával. Újra azt látjuk, hogy a feldolgozott kérések mennyisége a felhasználóterheléssel arányosan növekszik, míg az átlagos válaszidő alacsony marad. 

![A mintaalkalmazás átviteli sebessége, amint egy HttpClient-objektum már meglévő példányát használja az egyes kérésekhez][throughput-single-ExpensiveToCreateService-instance]



[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/ImproperInstantiation
[service-bus-messaging]: /azure/service-bus-messaging/service-bus-performance-improvements
[new-relic]: https://newrelic.com/application-monitoring
[throughput-new-HTTPClient-instance]: _images/HttpClientInstancePerRequest.jpg
[dashboard-new-HTTPClient-instance]: _images/HttpClientInstancePerRequestWebTransactions.jpg
[thread-profiler-new-HTTPClient-instance]: _images/HttpClientInstancePerRequestThreadProfile.jpg
[throughput-new-ExpensiveToCreateService-instance]: _images/ServiceInstancePerRequest.jpg
[throughput-single-HTTPClient-instance]: _images/SingleHttpClientInstance.jpg
[throughput-single-ExpensiveToCreateService-instance]: _images/SingleServiceInstance.jpg
[thread-profiler-single-HTTPClient-instance]: _images/SingleHttpClientInstanceThreadProfile.jpg
