---
title: Szinkron I/O kizárási minta
titleSuffix: Performance antipatterns for cloud apps
description: A hívó szál blokkolása az I/O végrehajtása során teljesítménycsökkenést okozhat, és hatással lehet a vertikális méretezhetőségre.
author: dragon119
ms.date: 06/05/2017
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 1b53806b2939a7c44a8b48c9146d5e86c84d9e2e
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54486165"
---
# <a name="synchronous-io-antipattern"></a>Szinkron I/O kizárási minta

A hívó szál blokkolása az I/O végrehajtása során teljesítménycsökkenést okozhat, és hatással lehet a vertikális méretezhetőségre.

## <a name="problem-description"></a>A probléma leírása

Egy szinkron I/O művelet blokkolja a hívó szálat az I/O végrehajtása során. A hívó szál várakozó állapotba vált, és ez idő alatt nem tud hasznos munkát végezni, ami pazarolja a feldolgozási erőforrásokat.

Néhány gyakori példa I/O műveletekre:

- Adatok lekérése és tárolása egy adatbázisban vagy bármilyen egyéb állandó tárolóban.
- Kérések küldése webszolgáltatásoknak.
- Üzenetek küldése vagy lekérése az üzenetsorokból.
- Helyi fájlok írása és olvasása.

A kizárási minta jellemzően az alábbi okokból következhet be:

- Ez tűnik egy adott művelet leginkább intuitív megoldási módjának.
- Az alkalmazásnak egy válaszra van szüksége egy adott kérdésre.
- Az alkalmazás egy olyan kódtárat használ, amely csak szinkron módszereket biztosít az I/O műveletekhez.
- Egy külső kódtár belső szinkron I/O műveleteket hajt végre. Egyetlen szinkron I/O hívás képes egy teljes hívásláncot blokkolni.

Az alábbi kód egy fájlt tölt fel az Azure Blob Storage-ba. A kódblokkok két helyen várnak szinkron I/O műveleteket: a `CreateIfNotExists` és az `UploadFromStream` metódusban.

```csharp
var blobClient = storageAccount.CreateCloudBlobClient();
var container = blobClient.GetContainerReference("uploadedfiles");

container.CreateIfNotExists();
var blockBlob = container.GetBlockBlobReference("myblob");

// Create or overwrite the "myblob" blob with contents from a local file.
using (var fileStream = File.OpenRead(HostingEnvironment.MapPath("~/FileToUpload.txt")))
{
    blockBlob.UploadFromStream(fileStream);
}
```

Íme egy példa, amely választ vár egy külső szolgáltatástól. A `GetUserProfile` metódus egy távoli szolgáltatást hív meg, amely egy `UserProfile` profilt ad vissza.

```csharp
public interface IUserProfileService
{
    UserProfile GetUserProfile();
}

public class SyncController : ApiController
{
    private readonly IUserProfileService _userProfileService;

    public SyncController()
    {
        _userProfileService = new FakeUserProfileService();
    }

    // This is a synchronous method that calls the synchronous GetUserProfile method.
    public UserProfile GetUserProfile()
    {
        return _userProfileService.GetUserProfile();
    }
}
```

Mindkét példa teljes kódját megtalálja [itt][sample-app].

## <a name="how-to-fix-the-problem"></a>A probléma megoldása

Cserélje le a szinkron I/O műveleteket aszinkron műveletekre. Ez szabaddá teszi az aktuális szálat, amely így hasznos munkát tud végezni a blokkolás helyett, és ez segít javítani a számítási erőforrások kihasználtságát. Az I/O műveletek aszinkron végrehajtása különösen hatékony megoldás az ügyfélalkalmazásokról érkező kérések hirtelen megszaporodásának lekezelésére.

Számos kódtár szinkron és aszinkron verziókat egyaránt kínál a különféle metódusokhoz. Amikor csak lehetséges, használja az aszinkron verziókat. Íme az előbbi példa aszinkron verziója a fájl feltöltéséhez az Azure Blob Storage-ba.

```csharp
var blobClient = storageAccount.CreateCloudBlobClient();
var container = blobClient.GetContainerReference("uploadedfiles");

await container.CreateIfNotExistsAsync();

var blockBlob = container.GetBlockBlobReference("myblob");

// Create or overwrite the "myblob" blob with contents from a local file.
using (var fileStream = File.OpenRead(HostingEnvironment.MapPath("~/FileToUpload.txt")))
{
    await blockBlob.UploadFromStreamAsync(fileStream);
}
```

Az `await` operátor visszaadja az irányítást a hívó környezetnek az aszinkron művelet végrehajtása közben. Az ezt az utasítást követő kód olyan folytatólagos műveleteket tartalmaz, amelyeknek a végrehajtására az aszinkron művelet befejeztével kerül sor.

A jól megtervezett szolgáltatásoknak aszinkron működést is biztosítaniuk kell. Íme a felhasználói profilokat visszaadó webszolgáltatás aszinkron verziója. A `GetUserProfileAsync` metódus a Felhasználói profil szolgáltatás aszinkron verzióját használja.

```csharp
public interface IUserProfileService
{
    Task<UserProfile> GetUserProfileAsync();
}

public class AsyncController : ApiController
{
    private readonly IUserProfileService _userProfileService;

    public AsyncController()
    {
        _userProfileService = new FakeUserProfileService();
    }

    // This is an synchronous method that calls the Task based GetUserProfileAsync method.
    public Task<UserProfile> GetUserProfileAsync()
    {
        return _userProfileService.GetUserProfileAsync();
    }
}
```

Az olyan kódtárak esetében, amelyek nem biztosítják a műveletek aszinkron verzióit, esetleg létre lehet hozni aszinkron burkolókat egyes szinkron metódusok köré. Ezt a megközelítést csak óvatosan alkalmazza. Növelheti ugyan vele az aszinkron burkolót meghívó szál válaszképességét, azonban több erőforrást használ fel. Előfordulhat, hogy létre kell hozni egy extra szálat, és a szál által végzett munka szinkronizálása többletmunkát jelent. A megoldás előnyeiről és hátrányairól a következő blogbejegyzés értekezik, amely azzal foglalkozik, hogy [érdemes-e aszinkron burkolókba csomagolni a szinkron metódusokat.][async-wrappers]

Íme egy példa egy aszinkron burkolóra egy szinkron metódus körül.

```csharp
// Asynchronous wrapper around synchronous library method
private async Task<int> LibraryIOOperationAsync()
{
    return await Task.Run(() => LibraryIOOperation());
}
```

Így a hívó kód most a burkolóra várakozhat:

```csharp
// Invoke the asynchronous wrapper using a task
await LibraryIOOperationAsync();
```

## <a name="considerations"></a>Megfontolandó szempontok

- A várhatóan nagyon rövid élettartamú és nem versengő I/O műveletek jobb teljesítményt biztosíthatnak szinkron műveletekként. Ilyen lehet például egy SSD meghajtón található kisméretű fájlok olvasása. A feladatok másik szálra küldéséhez, majd a feladat befejeztével a szál szinkronizálásához szükséges többletráfordítás esetleg meghaladja az aszinkron I/O működés előnyeit. Az ilyen esetek azonban meglehetősen ritkák, és a legtöbb I/O műveletet érdemes aszinkron módon végrehajtani.

- Az I/O teljesítmény fokozásával előfordulhat, hogy a rendszer egyéb részei válnak szűk keresztmetszetekké. Például a szálak blokkolásának megszüntetésével nagyobb egyidejű kérésmennyiség érkezhet a megosztott erőforrásokra, ami erőforráshiányhoz vagy leszabályozáshoz vezethet. Ha ez problematikus méreteket ölt, előfordulhat, hogy horizontálisan fel kell skáláznia a webkiszolgálók számát, vagy particionálnia kell az adattárakat a versenyhelyzet feloldása érdekében.

## <a name="how-to-detect-the-problem"></a>A probléma észlelése

A felhasználók számára úgy tűnhet, hogy az alkalmazás nem válaszol vagy rendszeresen lefagy. Az alkalmazás időtúllépési kivételeket produkálhat. Ezek a hibák HTTP 500 (belső kiszolgáló) hibaüzeneteket eredményezhetnek. A kiszolgálón a beérkező ügyfélkérések esetleg blokkolva lesznek, amíg egy szál fel nem szabadul, emiatt pedig túl hosszúra nőhetnek a kérésvárólisták, ami HTTP 503-as (a szolgáltatás nem érhető el) hibákat okozhat.

A következő lépések végrehajtásával azonosíthatja a problémát:

1. Az éles rendszer monitorozásával megállapíthatja, hogy a blokkolt feldolgozószálak korlátozzák-e a teljesítményt.

2. Ha a kérések blokkolásának a szálak hiánya az oka, ellenőrizze az alkalmazást, és állapítsa meg, hogy mely műveletekre jellemző a szinkron I/O működés.

3. Végezze el az egyes szinkron I/O működésű műveletek szabályozott terheléstesztjét annak megállapítása érdekében, hogy az adott műveletek hatással vannak-e a rendszer teljesítményére.

## <a name="example-diagnosis"></a>Diagnosztikai példa

Az alábbi szakaszokban ezeket a lépéseket hajtjuk végre a fentebb leírt mintaalkalmazáson.

### <a name="monitor-web-server-performance"></a>A webkiszolgáló teljesítményének monitorozása

Az Azure webalkalmazások és webes szerepkörök esetében érdemes monitorozni az IIS webkiszolgáló teljesítményét. Különösen a kérésvárólisták hosszára érdemes figyelni annak kiderítése érdekében, hogy a nagyon aktív időszakokban az elérhető szálakra való várakozás blokkolja-e a kéréseket. Ezekre az adatokra az Azure Diagnostics használatának engedélyezésével tehet szert. További információkért lásd:

- [Alkalmazások monitorozása az Azure App Service-ben][web-sites-monitor]
- [Teljesítményszámlálók létrehozása és használata Azure-alkalmazásokban][performance-counters]

Alakítsa ki úgy az alkalmazást, hogy látható legyen, hogyan kezeli a fogadott kéréseket. A kérésfolyam nyomon követése segít azonosítani, hogy lassú lefutású hívásokat hajt-e végre és blokkolja-e az aktuális szálat. A szálakra vonatkozó adatok gyűjtése is rámutathat azokra a kérésekre, amelyek blokkolva vannak.

### <a name="load-test-the-application"></a>Az alkalmazás terheléstesztje

A következő diagramon a korábban bemutatott szinkron `GetUserProfile` metódus teljesítménye látható változó terhelések mellett, 4000 egyidejű felhasználóig. Az alkalmazás egy Azure Cloud Services webes szerepkörben futó ASP.NET-alkalmazás.

![A szinkron I/O műveleteket végző mintaalkalmazás teljesítménydiagramja][sync-performance]

A szinkron működés kötelezően 2 másodpercig alszik a szinkron I/O szimulálása végett, így a minimális válaszidő valamivel több, mint 2 másodperc. Amikor a terhelés eléri a hozzávetőleg 2500 egyidejű felhasználót, az átlagos válaszidő elér egy plafont, bár a kérések másodpercenkénti mennyisége tovább nő. A két mutató logaritmikus skálán van megjelenítve. A kérések másodpercenkénti száma ettől a ponttól a teszt végéig a kétszeresére nő.

Ha így önmagában nézzük, ebből a tesztből nem derül ki egyértelműen, hogy a szinkron I/O működés problémát jelent-e. Nagyobb terhelés esetén az alkalmazás elérhet egy olyan fordulópontot, ahol a webkiszolgáló már nem képes időben feldolgozni a kéréseket, így az ügyfélalkalmazásoknál időtúllépési kivételek jelennek meg.

A beérkező kéréseket az IIS webkiszolgáló várólistára sorolja, és egy, az ASP.NET-szálkészletben futó szálnak osztja ki. Mivel mindegyik művelet szinkron végzi az I/O tevékenységeket, a szál blokkolva van, amíg a művelet véget nem ér. A munkaterhelés növekedésével végül az ASP.NET-szálkészletben található minden egyes szál le lesz foglalva és blokkolva lesz. Ezen a ponton minden további beérkező kérés a várólistában várakozik, amíg egy szál elérhetővé nem válik. Ahogy nő a várólista hossza, a kérések túllépik az időkorlátot.

### <a name="implement-the-solution-and-verify-the-result"></a>A megoldás megvalósítása és az eredmény ellenőrzése

A következő diagram a kód aszinkron verzióján végzett terhelésteszt eredményeit mutatja be.

![Az aszinkron I/O műveleteket végző mintaalkalmazás teljesítménydiagramja][async-performance]

A teljesítmény sokkal magasabb. Az előző teszttel egyező időtartam alatt a rendszer mintegy tízszer akkora teljesítményt mutatott a kérések másodpercenkénti számát tekintve. Emellett az átlagos válaszidő is viszonylag állandó, és hozzávetőleg 25-ször kisebb az előző tesztben mért értékeknél.

[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/SynchronousIO
[async-wrappers]: https://blogs.msdn.microsoft.com/pfxteam/2012/03/24/should-i-expose-asynchronous-wrappers-for-synchronous-methods/
[performance-counters]: /azure/cloud-services/cloud-services-dotnet-diagnostics-performance-counters
[web-sites-monitor]: /azure/app-service-web/web-sites-monitor

[sync-performance]: _images/SyncPerformance.jpg
[async-performance]: _images/AsyncPerformance.jpg
