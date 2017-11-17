---
title: "Külső Beállítástárolóval"
description: "Az alkalmazás központi telepítési csomag kívül a konfigurációs adatok áthelyezése egy központi helyen."
keywords: "Kialakítási mintája"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- design-implementation
- management-monitoring
ms.openlocfilehash: 733ca979903d1526d3a1a6b281a8903893e19fda
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="external-configuration-store-pattern"></a>Külső a konfigurációs adattároló minta

[!INCLUDE [header](../_includes/header.md)]

Az alkalmazás központi telepítési csomag kívül a konfigurációs adatok áthelyezése egy központi helyen. Ez az egyszerűbb kezelés és konfigurációs adatok feletti ellenőrzési, és az alkalmazások és alkalmazáspéldányok közötti megosztás a konfigurációs adatokat biztosít lehetőségek.

## <a name="context-and-problem"></a>A környezetben, és probléma

A legtöbb alkalmazás futásidejű környezetek tartalmazza az alkalmazással együtt telepített fájlok használatban konfigurációs információkat. Néhány esetben szerkesztése után szeretné módosítani az alkalmazás viselkedését üzembe helyezésüket követően ezeket a fájlokat. A konfiguráció módosításai kérhetnek az alkalmazást újra kell telepíteni, ami gyakran elfogadhatatlan állásidő és egyéb adminisztratív terhelés mellett.

Helyi konfigurációs fájlok is korlátozza a konfiguráció egyetlen alkalmazás, de egyes esetekben célszerű konfigurációs beállítások megosztására több alkalmazás között. Ilyen például az adatbázis-kapcsolati karakterláncok, a felhasználói felület téma információkat vagy a várólisták és a kapcsolódó alkalmazások által használt tároló URL-címei.

Kezeléséhez a helyi beállításokat az alkalmazás, különösen a felhőben üzemeltetett forgatókönyvek több futó példánya kihívást is. Különböző konfigurációs beállítások használatával, a frissítés telepítésekor példányok eredményezhet.

Alkalmazások és összetevők frissítéseit is követelhetnek sémák konfigurációs módosításait. Számos konfigurációs rendszer konfigurációs adatait különböző verziói nem támogatják.

## <a name="solution"></a>Megoldás

A konfigurációs adatok tárolása a külső tárhelyen, és gyors és hatékony olvassa el és konfigurációs beállításainak frissítése használható illesztőfelületet szolgáltasson. Külső áruházban típusa attól függ, hogy a tároló és a futásidejű környezet, az alkalmazás. A felhőben üzemeltetett forgatókönyvek általában a felhőalapú társzolgáltatás, de lehet egy központi adatbázisba vagy más system.

A biztonsági tár úgy dönt, hogy a konfigurációs adatokat egy felület, amely egységes és könnyen használható hozzáféréssel kell rendelkeznie. Akkor kell láthassa helytelenül beírt és strukturált formátumban. Végrehajtására is szükség lehet ahhoz, hogy a konfigurációs adatok védelmét, és lehet vajon elég rugalmas az tároló (például a fejlesztés, átmeneti vagy üzemi, beleértve a több kiadási konfigurációs több verziójának engedélyezése a felhasználói hozzáférés engedélyezésére verziója minden egyes).

> Számos beépített konfigurációs rendszer beolvasni az adatokat, amikor az alkalmazás elindul, és gyors hozzáférést, és az alkalmazás teljesítményére gyakorolt hatásának minimalizálása érdekében a memóriában adatok gyorsítótárazásához. Típusától biztonsági használt tárolási és a Tiltás késése a ezt a tárolót hasznos lehet a külső a konfigurációs adattároló belül gyorsítótárazást végrehajtásához. További információkért lásd: a [gyorsítótárazás útmutatást](https://msdn.microsoft.com/library/dn589802.aspx). Az ábra azt mutatja be, a választható helyi gyorsítótár külső a konfigurációs adattároló mintát áttekintését.

![A választható helyi gyorsítótár külső a konfigurációs adattároló mintát áttekintése](./_images/external-configuration-store-overview.png)


## <a name="issues-and-considerations"></a>Problémákat és szempontok

Ebben a mintában megvalósításához meghatározásakor, vegye figyelembe a következő szempontokat:

Válassza ki a biztonsági tároló, amely elfogadható teljesítményt, a magas rendelkezésre állás, a szolgáltatás megbízhatóságára nyújt, és a biztonsági másolat készíthető a karbantartási és felügyeleti folyamatának részeként. A felhőben üzemeltetett alkalmazásban a felhő tárolási mechanizmus segítségével, akkor a általában ezek a követelmények teljesítéséhez.

Tervezze meg a séma, a biztonsági tároló, hogy lehetővé tegyék a rugalmasságot biztosít a típusú adatok tárolására képes. Győződjön meg arról, hogy az összes konfigurációs követelmények például típusos adatok, beállítások, beállítások különböző verzióinak és más szolgáltatások, az azt használó alkalmazások igénylő biztosít. A séma kell könnyen terjessze ki további beállításokat támogatja, a követelmények változnak.

Vegye figyelembe, hogy a mögöttes fizikai képességeit tárolja, hogyan felügyeletében a konfigurációs adatokat tárolja, és a teljesítményre gyakorolt hatás. Például egy beállításait tartalmazó XML-dokumentum tárolásához szükséges konfigurációs felületen vagy az alkalmazás a dokumentum értelmezhető ahhoz, hogy egyes beállítások beolvasása. Fog létrehozni, akkor egy összetettebb, beállítás frissítése ellenére gyorsítótárazási a beállítások segítséget eltolási lassabb olvasási teljesítmény.

Vegye figyelembe, hogyan ad engedélyt a konfigurációs felület a hatókör-vezérlés és a konfigurációs beállítások öröklési. Például lehet a követelmény a hatókör beállításait a szervezetben, alkalmazás és a gép szint. Módosítania, hogy támogatják a különböző hatóköröket történő hozzáférés delegálására és a beállítások felülírják az egyes alkalmazások engedélyezése vagy tiltása.

Győződjön meg arról, hogy a konfigurációs felület is elérhetővé teheti a konfigurációs adatokat, például gépelt értékek, a gyűjtemények, a kulcs/érték párok vagy a tulajdonságcsomagok szükséges formátumban.

Vegye figyelembe, hogy a konfigurációs tároló felület beállítások hibákat tartalmaz, vagy nem található a biztonsági tárolóban működése. Akkor célszerű térjen vissza az alapértelmezett beállításokat, és a hibák naplózására. A nagybetűk konfigurációs beállítás kulcsok vagy nevek, a tárolási és a bináris adatok kezelése és a módon, hogy kezeli-e a null vagy üres értékeket szempontjai figyelembe venni.

Vegye figyelembe, hogyan védi meg a konfigurációs adatokat, hogy csak a megfelelő felhasználók és az alkalmazások hozzáférjenek. Ez valószínűleg egy olyan beállítás, a konfigurációs tároló illesztőfelület, de az is biztosítani kell, hogy az adatokat a biztonsági tároló nem érhető el közvetlenül a megfelelő engedély nélkül. Győződjön meg arról, szigorú elkülönítése is megvalósuljon olvasni és írni a konfigurációs adatok szükséges engedélyekkel. Is vegye figyelembe, hogy vagy azok egy részét a konfigurációs beállítások titkosítani kell, és hogyan ez fogja kell végrehajtani a konfigurációs tároló felületen.

Központilag tárolt konfigurációk, amelyek alkalmazás megváltozzon futásidőben, különösen fontos kell telepíteni, frissíteni, és felügyelt alkalmazás kód telepítésének ugyanazt a mechanizmust használva. Például több alkalmazást is érintő változások végre kell hajtani egy teljes vizsgálatot, és győződjön meg arról, hogy a módosítás megfelelő minden olyan alkalmazásnál, amely használja ezt a konfigurációt a telepítési módszerre előkészítése. A rendszergazda szerkesztése egy alkalmazás frissítése egy beállítást, ha más alkalmazásokat, amelyek ugyanazt a beállítást használják kedvezőtlen hatással lehetett.

Ha egy alkalmazás gyorsítótárba helyezi azt a konfigurációs adatokat, az alkalmazást kell riasztást kap, ha a konfiguráció módosításait. Előfordulhat, hogy a valósíthat meg egy elévülési szabályzatának keresztül gyorsítótárazott konfigurációs adatokat, hogy ezeket az információkat rendszeres időközönként automatikusan frissülnek, és módosításokat felvételre (és a intézkedni).

## <a name="when-to-use-this-pattern"></a>Mikor érdemes használni ezt a mintát

Ebben a mintában a következőkhöz hasznos:

- Konfigurációs beállítások, amelyek között vannak megosztva, több alkalmazás és alkalmazáspéldányok, vagy ha normál konfigurációban kell kényszeríthető több alkalmazások és alkalmazáspéldányok.

- Egy szabványos konfigurációs rendszer nem támogatja a szükséges konfigurációs beállítások, például képeket vagy összetett adattípusú tárolásához.

- Az egyes alkalmazások beállításai kiegészítő tárolóként lehet, hogy így alkalmazások vagy azok egy részét a központilag tárolt beállításainak felülbírálására.

- Egyszerű kezelés érdekében több alkalmazást, és opcionálisan ellenőrzési úgy használja a konfigurációs beállítások történő naplózásának révén a konfigurációs adattároló néhány vagy az összes típusú hozzáférést.

## <a name="example"></a>Példa

A Microsoft Azure szolgáltatásban futtatott alkalmazás egy tipikus választott külsőleg konfigurációs adatok tárolásához, hogy használja az Azure Storage. Ez rugalmas, magas teljesítményt nyújt, és automatikus feladatátvétel magas rendelkezésre állást biztosítani a rendszer háromszor replikálja. Az Azure Table storage egy kulcs-érték tárolóban biztosít az értékek egy rugalmas séma használhatja. Az Azure Blob storage biztosítja, hogy külön-külön elnevezett blobok bármilyen típusú adat tárolására képes hierarchikus, tároló-alapú tárolását.

A következő példa bemutatja, hogyan a konfigurációs adattároló keresztül tárolja, és tegye elérhetővé a konfigurációs adatokat a Blob-tároló implementálhatók. A `BlobSettingsStore` osztály kivonatolja a Blob-tároló rendelkezik a konfigurációs adatokat, és megvalósítja az `ISettingsStore` felülete az alábbi kódban látható.

> Ezt a kódot a rendszer biztosítja a _ExternalConfigurationStore.Cloud_ a projektre a _ExternalConfigurationStore_ az elérhető megoldás [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/external-configuration-store).

```csharp
public interface ISettingsStore
{
    Task<string> GetVersionAsync();

    Task<Dictionary<string, string>> FindAllAsync();
}
```

Ez az interfész beolvasása és a konfigurációs adattároló tárolt konfigurációs beállításainak frissítése módszerek meghatározása, és egy verziószámot, amely segítségével észleli, hogy nemrég módosították minden olyan konfigurációs beállítást is. A `BlobSettingsStore` osztályt használja a `ETag` tulajdonság a BLOB versioning végrehajtásához. A `ETag` tulajdonság frissítése automatikusan történik minden alkalommal, amikor a blob írása.

> Úgy lett kialakítva a legegyszerűbb megoldás gépelt értékek, hanem karakterlánc-értékek összes konfigurációs beállítás közzétesz.

A `ExternalConfigurationManager` osztály kínál egy egy `BlobSettingsStore` objektum. Az alkalmazás az osztály használatával tárolásához és konfigurációs információk lekéréséhez. Ez az osztály használja a Microsoft [reaktív bővítmények](https://msdn.microsoft.com/library/hh242985.aspx) könyvtár keresztül megvalósítása a konfiguráció módosításai teszi közzé a `IObservable` felületet. Ha egy beállítás meghívásával módosítják a `SetAppSetting` metódus, a `Changed` egy esemény jelenik meg, és ezt az eseményt minden előfizetők értesítést fog kapni.

Vegye figyelembe, hogy az összes beállítást is a gyorsítótárba helyezett egy `Dictionary` objektum belül a `ExternalConfigurationManager` osztály a gyors hozzáférés. A `GetSetting` egy konfigurációs beállítás beolvasásához használt módszer a adatokat olvas a gyorsítótárból. Ha a beállítás nem található a gyorsítótárban, azt veszi át a `BlobSettingsStore` objektum helyett.

A `GetSettings` metódus meghívja a `CheckForConfigurationChanges` metódus észleli, hogy változott-e a konfigurációs adatokat a blob Storage tárolóban. Ennek érdekében a verziószám vizsgálata és összehasonlítva azt az a jelenlegi verziószám birtokában a `ExternalConfigurationManager` objektum. Ha egy vagy több változtatások történtek, a `Changed` egy esemény jelenik meg, és tárolja a rendszer a konfigurációs beállításokat a `Dictionary` objektum frissítése. Ez az az alkalmazás a [gyorsítótár-Tartalékoljon mintát](cache-aside.md).

A következő kódot a minta azt mutatja be az `Changed` esemény, a `GetSettings` metódust, és a `CheckForConfigurationChanges` metódus valósíthatók meg:

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

> A `ExternalConfigurationManager` osztály is biztosít nevű tulajdonság `Environment`. Ez a tulajdonság támogatja a különböző konfigurációkat különböző környezetekben, például az átmeneti és üzemi futó alkalmazásokra vonatkozóan.

Egy `ExternalConfigurationManager` objektum is lekérdezheti a `BlobSettingsStore` objektum rendszeres időközönként a módosításokat. A következő kódrészlet a `StartMonitor` metódushívások `CheckForConfigurationChanges` ismeri fel a módosításokat, és indítson egy időközönként a `Changed` esemény, a fentebb leírt módon.

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

A `ExternalConfigurationManager` osztály által egypéldányos példányként létrejön a `ExternalConfiguration` osztály alább látható.

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

Az alábbi kód származik a `WorkerRole` osztályt a _ExternalConfigurationStore.Cloud_ projekt. Azt mutatja, hogy az alkalmazás használja a `ExternalConfiguration` osztály beolvasni egy beállítását.

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

A következő kódot a is a `WorkerRole` osztály, bemutatja, hogyan számítógépcsoportra fizetett elő az alkalmazás konfigurációs események.

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

## <a name="related-patterns-and-guidance"></a>Útmutató és a kapcsolódó minták

- Minta bemutatja, ebben a mintában érhető el a [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/external-configuration-store).
