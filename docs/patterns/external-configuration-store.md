---
title: Külső konfigurációs tár mintája
titleSuffix: Cloud Design Patterns
description: A konfigurációs adatokat áthelyezheti az alkalmazás üzembehelyezési csomagjából egy központi helyre.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
ms.custom: seodec18
ms.openlocfilehash: 7e37e5bc052a9d8e8747a3a4ac3d79a311185ea4
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/04/2019
ms.locfileid: "54011310"
---
# <a name="external-configuration-store-pattern"></a>Külső konfigurációs tár mintája

[!INCLUDE [header](../_includes/header.md)]

A konfigurációs adatokat áthelyezheti az alkalmazás üzembehelyezési csomagjából egy központi helyre. Ezzel lehetőség nyílik a konfigurációs adatok egyszerűbb kezelésére és felügyeletére, valamint a konfigurációs adatok alkalmazások és alkalmazáspéldányok közötti megosztására.

## <a name="context-and-problem"></a>Kontextus és probléma

A legtöbb alkalmazásfuttató környezetben találhatók az alkalmazással együtt üzembe helyezett fájlokban tárolt konfigurációs adatok. Bizonyos esetekben ezek a fájlok szerkeszthetők, így az üzembe helyezés után is módosítható az alkalmazás viselkedése. Az ilyen konfigurációmódosítások után viszont újból üzembe kell helyezni az alkalmazást, ez pedig gyakran elfogadhatatlan állásidőt és egyéb adminisztrációs terhelést jelenthet.

A helyi konfigurációs fájlok ráadásul egy alkalmazásra korlátozzák a konfigurációt, pedig néha hasznos lenne megosztani több alkalmazás között is a konfigurációs beállításokat. Ilyenek például az adatbázis-kapcsolati sztringek, a felhasználói felület témájára vonatkozó információk vagy az alkalmazások kapcsolódó készlete által használt üzenetsorok és tárolók URL-címei.

A helyi konfigurációk változásainak az alkalmazás több futó példányán történő kezelése kihívást jelenthet, különösen egy felhőben üzembe helyezett forgatókönyv esetén. Ez különböző konfigurációs beállításokat használó példányokat eredményezhet a frissítés üzembe helyezése során.

Ezenfelül az alkalmazások és összetevők frissítéséhez szükség lehet a konfigurációs sémák módosítására. Számos konfigurációs rendszer nem támogatja a különböző konfigurációsadat-verziókat.

## <a name="solution"></a>Megoldás

A konfigurációs adatokat tárolja egy külső tárban, és biztosítson egy felületet, amely segítségével gyorsan és hatékonyan olvashat és frissíthet konfigurációs beállításokat. A külső tároló típusa az alkalmazás üzemeltetési és futtatókörnyezetétől függ. A felhőben üzembe helyezett forgatókönyv esetében ez általában egy felhőalapú tárolási szolgáltatás, de lehet üzemeltetett adatbázis vagy más rendszer is.

A konfigurációs adatokhoz választott háttértárhoz szükség van egy felületre, amely stabil és könnyen használható hozzáférést biztosít. Az információkat típussal helyesen ellátott és strukturált formátumban kell elérhetővé tennie. Arra is szükség lehet, hogy a konfigurációs adatok védelme érdekében az implementáció engedélyezze a felhasználók hozzáférését, és elég rugalmas legyen ahhoz, hogy a konfiguráció több verzióját is tárolja (például fejlesztési, átmeneti, éles üzemi, mindegyiknél akár több kiadásverziót beleértve).

> Számos beépített konfigurációs rendszer beolvassa az adatokat az alkalmazás indulásakor, majd gyorsítótárazza az adatokat a memóriában, hogy gyors hozzáférést biztosítson, és a lehető legkisebbre csökkentse az alkalmazás teljesítményére gyakorolt hatást. A használt háttértár típusától és a tár késésétől függően hasznos lehet implementálni egy gyorsítótárazási mechanizmust a külső konfigurációs táron belül. További információt a [gyorsítótárazási útmutatóban](https://msdn.microsoft.com/library/dn589802.aspx) találhat. Az ábra a külső konfigurációs tár mintájának és az opcionális helyi gyorsítótárnak az áttekintését teszi lehetővé.

![A külső konfigurációs tár mintájának és az opcionális helyi gyorsítótárnak az áttekintése](./_images/external-configuration-store-overview.png)

## <a name="issues-and-considerations"></a>Problémák és megfontolandó szempontok

A minta megvalósítása során az alábbi pontokat vegye figyelembe:

Válasszon olyan háttértárat, amely elfogadható teljesítményt, magas rendelkezésre állást és robusztus működést biztosít, valamint készíthető róla biztonsági másolat az alkalmazás karbantartási és adminisztrációs folyamatának részeként. Egy felhőben üzemeltetett alkalmazásban egy felhőalapú tárolási mechanizmus használata általában jó választás a követelmények teljesítéséhez.

Úgy tervezze meg a háttértár sémáját, hogy rugalmasan tudja kezelni a tárolt adatok típusait. Győződjön meg arról, hogy minden konfigurációs követelménynek megfeleljen (például típussal ellátott adatok, beállításgyűjtemények, több beállításverzió és bármely más olyan funkció, amelyre a háttértárt használó alkalmazásoknak szükségük lehet). Fontos, hogy a séma könnyen bővíthető legyen, hogy további beállításokat is támogasson, ha megváltoznak a követelmények.

Vegye figyelembe a háttértár fizikai képességeit, illetve azt, hogyan viszonyul a konfigurációs adatok tárolási módjához, továbbá a teljesítményre gyakorolt hatását. Egy konfigurációs adatokat tartalmazó XML-dokumentum tárolásához például feltétel, hogy az egyes beállítások olvasásához a konfigurációs felület vagy az alkalmazás elemezze a dokumentumot. Ettől egy beállítás frissítése bonyolultabbá válik, bár a beállítások gyorsítótárazása segíthet ellensúlyozni a lassabb olvasási teljesítmény.

Gondolja át, hogyan fogja a konfigurációs felület lehetővé tenni a hatókör felügyeletét és a konfigurációs beállítások öröklését. Követelmény lehet például a konfigurációs beállítások hatókörének korlátozása a cég, az alkalmazás vagy a gépek szintjén. Előfordulhat, hogy támogatnia kell a különböző hatókörökhöz való hozzáférés fölötti felügyelet delegálását, valamint megakadályozni vagy engedélyezni az egyes alkalmazások számára a beállítások felülbírálását.

Győződjön meg arról, hogy a konfigurációs felület képes a szükséges formátumban elérhetővé tenni a konfigurációs adatokat, például típussal ellátott értékekként, gyűjteményekként, a kulcs/érték-párokként vagy tulajdonságcsomagok formájában.

Gondolja át, hogyan fog viselkedni a konfigurációs tár felülete, amikor a beállítások hibásak, vagy nem is léteznek a háttértárban. Érdemes lehet visszatérni az alapértelmezett beállításokhoz és naplózni a hibákat. Az olyan aspektusokat is gondolja végig, mint hogy a konfigurációs beállítások kulcsai vagy nevei megkülönböztetik-e a kis- és nagybetűket, hogyan fogja tárolni és kezelni a bináris adatokat, valamint milyen módon fogja kezelni a nullértékű vagy üres értékeket.

Gondolja át, hogyan védheti meg a konfigurációs adatokat, hogy csak a megfelelő felhasználók és alkalmazások kapjanak hozzáférést. A konfigurációs tár felülete valószínűleg nyújt ilyen szolgáltatást, de azt is biztosítani kell, hogy a háttértárban lévő adatokhoz ne lehessen közvetlenül hozzáférni a megfelelő engedély nélkül. Ügyeljen arra, hogy szigorúan elválassza a konfigurációs adatok olvasásához és írásához szükséges engedélyeket. Azt is vegye figyelembe, hogy titkosítania kell-e a konfigurációs beállítások egy részét vagy egészét, és hogy ez hogyan lesz implementálva a konfigurációs tár felületén.

A központilag tárolt, az alkalmazások viselkedését a futtatás közben módosító konfigurációk különösen fontosak, és ugyanazokkal a mechanizmusokkal kell üzembe helyezni, frissíteni és felügyelni, mint az alkalmazás kódját. Az egynél több alkalmazást érintő módosításokat például teljes teszteléssel és szakaszos üzembe helyezéses megközelítéssel kell végrehajtani. Így biztosítható, hogy a módosítás minden, a konfigurációt használó alkalmazás számára megfelelő legyen. Ha egy rendszergazda szerkeszt egy beállítást egy alkalmazás frissítéséhez, az negatív hatással lehet más, ugyanazt a beállítást használó alkalmazásokra.

Ha egy alkalmazás gyorsítótárazza a konfigurációs adatokat, az alkalmazást riasztani kell, ha módosul a konfiguráció. Meg lehet esetleg valósítani egy, a gyorsítótárazott konfigurációs adatokra vonatkozó elévülési szabályzatot, hogy ez az információ rendszeres időközönként automatikusan frissüljön, és érvénybe lépjenek az esetleges változások.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes ezt a mintát használni?

Ez a minta az alábbi esetekben hasznos:

- Több alkalmazás és alkalmazáspéldány között megosztott konfigurációs beállításoknál, vagy ahol a szabványos konfigurációt kell kényszeríteni több alkalmazásra és alkalmazáspéldányra.

- Olyan szabványos konfigurációs rendszer esetében, amely nem támogatja az összes szükséges konfigurációs beállítást, például a képek tárolását vagy az összetett adattípusokat.

- Kiegészítő tárolóként az alkalmazások bizonyos beállításaihoz, amely esetleg lehetővé teszi az alkalmazások számára néhány vagy az összes központilag tárolt beállítás felülbírálását.

- Több alkalmazás adminisztrációjának egyszerűsítésére, valamint opcionálisan a konfigurációs beállítások használatának monitorozására a konfigurációs tár néhány vagy összes hozzáféréstípusának naplózása révén.

## <a name="example"></a>Példa

Egy Microsoft Azure által üzemeltetett alkalmazásban a konfigurációs adatok külső tárolására ideális választás az Azure Storage. Rugalmas, és nagy teljesítményt tesz elérhetővé, ráadásul a magas rendelkezésre állás érdekében háromszor van replikálva, automatikus feladatátvétellel. Az Azure Table Storage egy kulcs/érték-tárt biztosít, amely képes rugalmas sémát használni az értékekhez. Az Azure Blob Storage hierarchikus, tárolóalapú tárt biztosít, amely egyénileg elnevezett blobokban bármilyen típusú adatot képes tárolni.

Az alábbi példa bemutatja, hogyan lehet implementálni egy konfigurációs tárt a Blob Storage-on konfigurációs adatok tárolására és elérhetővé tételére. A `BlobSettingsStore` osztály absztrahálja a Blob Storage-ot a konfigurációs adatok tárolására, és implementálja az alábbi kódban látható `ISettingsStore` interfészt.

> Ez a kód a [GitHubon](https://github.com/mspnp/cloud-design-patterns/tree/master/external-configuration-store) elérhető _ExternalConfigurationStore_ megoldás _ExternalConfigurationStore.Cloud_ projektjében található meg.

```csharp
public interface ISettingsStore
{
    Task<string> GetVersionAsync();

    Task<Dictionary<string, string>> FindAllAsync();
}
```

Ez az interfész a konfigurációs tárban tárolt konfigurációs beállítások lekéréséhez és frissítéséhez határoz meg metódusokat, és tartalmaz egy verziószámot, amely segítségével észlelhető, hogy módosult-e a közelmúltban bármely konfigurációs beállítás. A `BlobSettingsStore` osztály a blob `ETag` tulajdonságát használja a verziókövetés megvalósítására. Az `ETag` tulajdonság a blob minden írása után frissül.

> Ez az egyszerű megoldás úgy lett kialakítva, hogy minden konfigurációs beállítást típussal ellátott érték helyett sztringértékként tesz elérhetővé.

Az `ExternalConfigurationManager` osztály burkolót biztosít egy `BlobSettingsStore` objektum körül. Egy alkalmazás használhatja ezt az osztályt konfigurációs adatok tárolására és lekérésére. Ez az osztály a Microsoft [Reactive Extensions](https://msdn.microsoft.com/library/hh242985.aspx) kódtára segítségével teszi közzé a konfiguráció módosításait az `IObservable` interfész implementációján keresztül. Ha egy beállítás a `SetAppSetting` metódus meghívása révén módosul, létrejön a `Changed` esemény, és az esemény minden előfizetője értesítést kap.

Vegye figyelembe, hogy a rendszer a gyors hozzáférés érdekében minden beállítást gyorsítótáraz is egy `Dictionary` objektumban az `ExternalConfigurationManager` osztályon belül. A konfigurációs beállítások lekéréséhez használt `GetSetting` metódus beolvassa a gyorsítótárból az adatokat. Ha a beállítás nem található a gyorsítótárban, az alkalmazás a `BlobSettingsStore` objektumból kéri le helyette.

A `GetSettings` metódus meghívja a `CheckForConfigurationChanges` metódust, hogy az felderítse, hogy a Blob Storage-ban lévő konfigurációs adatok módosultak-e. Ehhez megvizsgálja a verziószámot, és összehasonlítja az `ExternalConfigurationManager` objektumban tárolt aktuális verziószámmal. Ha történt egy vagy több módosítás, létrejön a `Changed` esemény, és a rendszer frissíti a `Dictionary` objektumban gyorsítótárazott konfigurációs beállításokat. Ez a [gyorsítótár-feltöltési minta](./cache-aside.md) egy alkalmazása.

A következő kódminta a `Changed` esemény, a `GetSettings` metódus és a `CheckForConfigurationChanges` metódus implementálását mutatja be:

```csharp
public class ExternalConfigurationManager : IDisposable
{
  // An abstraction of the configuration store.
  private readonly ISettingsStore settings;
  private readonly ISubject<KeyValuePair<string, string>> changed;
  ...
  private readonly ReaderWriterLockSlim settingsCacheLock = new ReaderWriterLockSlim();
  private readonly SemaphoreSlim syncCacheSemaphore = new SemaphoreSlim(1);  
  ...
  private Dictionary<string, string> settingsCache;
  private string currentVersion;
  ...
  public ExternalConfigurationManager(ISettingsStore settings, ...)
  {
    this.settings = settings;
    ...
  }
  ...
  public IObservable<KeyValuePair<string, string>> Changed => this.changed.AsObservable();
  ...

  public string GetAppSetting(string key)
  {
    ...
    // Try to get the value from the settings cache.
    // If there's a cache miss, get the setting from the settings store and refresh the settings cache.

    string value;
    try
    {
        this.settingsCacheLock.EnterReadLock();

        this.settingsCache.TryGetValue(key, out value);
    }
    finally
    {
        this.settingsCacheLock.ExitReadLock();
    }

    return value;
  }
  ...
  private void CheckForConfigurationChanges()
  {
    try
    {
        // It is assumed that updates are infrequent.
        // To avoid race conditions in refreshing the cache, synchronize access to the in-memory cache.
        await this.syncCacheSemaphore.WaitAsync();

        var latestVersion = await this.settings.GetVersionAsync();

        // If the versions are the same, nothing has changed in the configuration.
        if (this.currentVersion == latestVersion) return;

        // Get the latest settings from the settings store and publish changes.
        var latestSettings = await this.settings.FindAllAsync();

        // Refresh the settings cache.
        try
        {
            this.settingsCacheLock.EnterWriteLock();

            if (this.settingsCache != null)
            {
                //Notify settings changed
                latestSettings.Except(this.settingsCache).ToList().ForEach(kv => this.changed.OnNext(kv));
            }
            this.settingsCache = latestSettings;
        }
        finally
        {
            this.settingsCacheLock.ExitWriteLock();
        }

        // Update the current version.
        this.currentVersion = latestVersion;
    }
    catch (Exception ex)
    {
        this.changed.OnError(ex);
    }
    finally
    {
        this.syncCacheSemaphore.Release();
    }
  }
}
```

> Az `ExternalConfigurationManager` osztály egy `Environment` nevű tulajdonságot is elérhetővé tesz. Ez a tulajdonság támogatja a különböző konfigurációkat eltérő, például átmeneti és éles környezetben futó alkalmazásoknál.

Egy `ExternalConfigurationManager` objektum a `BlobSettingsStore` objektumot is lekérdezheti rendszeres időközönként, hogy történt-e módosítás. A következő kódrészletben a `StartMonitor` metódus meghatározott időközönként meghívja a `CheckForConfigurationChanges` metódust, hogy észlelje az esetleges módosításokat, és a korábban leírt módon létrehozza a `Changed` eseményt.

```csharp
public class ExternalConfigurationManager : IDisposable
{
  ...
  private readonly ISubject<KeyValuePair<string, string>> changed;
  private Dictionary<string, string> settingsCache;
  private readonly CancellationTokenSource cts = new CancellationTokenSource();
  private Task monitoringTask;
  private readonly TimeSpan interval;

  private readonly SemaphoreSlim timerSemaphore = new SemaphoreSlim(1);
  ...
  public ExternalConfigurationManager(string environment) : this(new BlobSettingsStore(environment), TimeSpan.FromSeconds(15), environment)
  {
  }
  
  public ExternalConfigurationManager(ISettingsStore settings, TimeSpan interval, string environment)
  {
      this.settings = settings;
      this.interval = interval;
      this.CheckForConfigurationChangesAsync().Wait();
      this.changed = new Subject<KeyValuePair<string, string>>();
      this.Environment = environment;
  }
  ...
  /// <summary>
  /// Check to see if the current instance is monitoring for changes
  /// </summary>
  public bool IsMonitoring => this.monitoringTask != null && !this.monitoringTask.IsCompleted;

  /// <summary>
  /// Start the background monitoring for configuration changes in the central store
  /// </summary>
  public void StartMonitor()
  {
      if (this.IsMonitoring)
          return;

      try
      {
          this.timerSemaphore.Wait();

          // Check again to make sure we are not already running.
          if (this.IsMonitoring)
              return;

          // Start running our task loop.
          this.monitoringTask = ConfigChangeMonitor();
      }
      finally
      {
          this.timerSemaphore.Release();
      }
  }

  /// <summary>
  /// Loop that monitors for configuration changes
  /// </summary>
  /// <returns></returns>
  public async Task ConfigChangeMonitor()
  {
      while (!cts.Token.IsCancellationRequested)
      {
          await this.CheckForConfigurationChangesAsync();
          await Task.Delay(this.interval, cts.Token);
      }
  }

  /// <summary>
  /// Stop monitoring for configuration changes
  /// </summary>
  public void StopMonitor()
  {
      try
      {
          this.timerSemaphore.Wait();

          // Signal the task to stop.
          this.cts.Cancel();

          // Wait for the loop to stop.
          this.monitoringTask.Wait();

          this.monitoringTask = null;
      }
      finally
      {
          this.timerSemaphore.Release();
      }
  }

  public void Dispose()
  {
      this.cts.Cancel();
  }
  ...
}
```

Az `ExternalConfigurationManager` osztályt a rendszer egyszeri példányként példányosítja az `ExternalConfiguration` osztállyal, ahogy az alább is látható.

```csharp
public static class ExternalConfiguration
{
    private static readonly Lazy<ExternalConfigurationManager> configuredInstance = new Lazy<ExternalConfigurationManager>(
        () =>
        {
            var environment = CloudConfigurationManager.GetSetting("environment");
            return new ExternalConfigurationManager(environment);
        });

    public static ExternalConfigurationManager Instance => configuredInstance.Value;
}
```

Az alábbi kód az _ExternalConfigurationStore.Cloud_ projekt `WorkerRole` osztályából származik. Bemutatja, hogyan használja az alkalmazás az `ExternalConfiguration` osztályt egy beállítás beolvasására.

```csharp
public override void Run()
{
  // Start monitoring configuration changes.
  ExternalConfiguration.Instance.StartMonitor();

  // Get a setting.
  var setting = ExternalConfiguration.Instance.GetAppSetting("setting1");
  Trace.TraceInformation("Worker Role: Get setting1, value: " + setting);

  this.completeEvent.WaitOne();
}
```

A következő, szintén a `WorkerRole` osztályból származó kód bemutatja, hogyan iratkozik fel egy alkalmazás konfigurációs eseményekre.

```csharp
public override bool OnStart()
{
  ...
  // Subscribe to the event.
  ExternalConfiguration.Instance.Changed.Subscribe(
     m => Trace.TraceInformation("Configuration has changed. Key:{0} Value:{1}",
          m.Key, m.Value),
     ex => Trace.TraceError("Error detected: " + ex.Message));
  ...
}
```

## <a name="related-patterns-and-guidance"></a>Kapcsolódó minták és útmutatók

- A [GitHubon](https://github.com/mspnp/cloud-design-patterns/tree/master/external-configuration-store) talál egy, a minta bemutatására szolgáló példát.
