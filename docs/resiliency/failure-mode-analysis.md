---
title: A hibaállapot elemzése
description: Útmutató az Azure-alapú megoldások mód elemzés végrehajtásához.
author: MikeWasson
ms.date: 03/24/2017
ms.custom: resiliency
pnp.series.title: Design for Resiliency
ms.openlocfilehash: 95068bf8b1f5b559255e27819aaddb454d3427bc
ms.sourcegitcommit: d08f6ee27e1e8a623aeee32d298e616bc9bb87ff
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 05/07/2018
---
# <a name="failure-mode-analysis"></a>A hibaállapot elemzése
[!INCLUDE [header](../_includes/header.md)]

Hibaelemzés mód (FMA) az a folyamat rugalmassági készítéséhez rendszerben, a rendszer lehetséges hiba pontok azonosításával. A FMA felépítéséről és kialakításáról fázisok részének kell lennie, úgy, hogy a rendszer az elejéről történő hibaelhárítás hozhat létre.

Íme egy FMA elvégzésére általános folyamata:

1. Azonosítsa az összes összetevőt a rendszerben. Foglaljon magába függ külső tényezőktől, például az identitás-szolgáltatóktól, harmadik féltől származó szolgáltatással, stb.   
2. Az egyes összetevők esetleg előforduló lehetséges hibák azonosítására. Előfordulhat, hogy az adott összetevőt egynél több hibaállapotba. Például kell figyelembe venni az olvasási hibák, és hibák külön-külön kiírni, mert a hatás és a lehetséges megoldást eltérőek lesznek.
3. Az általános kockázati megfelelően minden hibaállapotba aránya. Vegye figyelembe a következőket:  

   * Mi az a hibák valószínűségét. Az viszonylag közös? Ritka Extrememly? Nem kell pontos számok; a célja a prioritás rangsorolja segítségével.
   * Mi az az alkalmazás rendelkezésre állásának, adatvesztés, a költség és üzleti megszűnésének gyakorolt hatás?
4. Minden egyes hibaállapotba határozza meg a hogyan az alkalmazás válaszol, és helyreállítani. Figyelembe kell venni mellékhatásokkal költség- és alkalmazás összetettségét.   

A FMA folyamat kezdőpontját Ez a cikk lehetséges függően katalógust és az azok mérséklési tartalmazza. A katalógus technológia vagy Azure-szolgáltatás, valamint egy általános alkalmazás szintű tervezési kategória szerint van rendezve. A katalógus nem teljes, de hozzá van rendelve az alapszintű Azure számos szolgáltatások.

## <a name="app-service"></a>App Service
### <a name="app-service-app-shuts-down"></a>App Service alkalmazás leáll.
**Észlelési**. Lehetséges okok:

* Várt leállás

  * Az alkalmazás leállása operátor például az Azure portál használatával.
  * Az alkalmazás lett távolítva a memóriából, mert üresjáratban van. (Csak akkor, ha a `Always On` beállítás le van tiltva.)
* Nem tervezett leállás

  * Az alkalmazás összeomlik.
  * Az App Service Virtuálisgép-példány nem érhető el.

Application_End naplózási fog dolgozza fel az alkalmazás tartomány leállítási (soft folyamat összeomlási), és csak úgy dolgozza fel az alkalmazás tartomány kiváltott leállítás elkerüléséhez.

**Helyreállítás**

* Ha a leállítási várta, használja az alkalmazás leállítás esemény leállítása. Például az ASP.NET, használja a `Application_End` metódust.
* Ha az alkalmazás memóriából közben üresjárati, automatikusan újraindul a következő kérelemnél. Azonban a "hidegindítás" költség gyakorisága.
* Az alkalmazás közben üresjárati eltávolítása folyamatban érdekében engedélyezze a `Always On` a webalkalmazás-ban történő beállítását. Lásd: [webes alkalmazások konfigurálása az Azure App Service][app-service-configure].
* Az alkalmazás leállítása érdekében operátor beállítása egy erőforrás zárolást a következő `ReadOnly` szintjét. Lásd: [erőforrások az Azure Resource Manager zárolása][rm-locks].
* Ha az alkalmazás összeomlik vagy az App Service virtuális gép nem érhető el, az App Service automatikusan újraindul az alkalmazás.

**Diagnosztika**. Alkalmazás naplók és a webkiszolgáló naplóinak. Lásd: [az Azure App Service web Apps diagnosztikai naplózás engedélyezése][app-service-logging].

### <a name="a-particular-user-repeatedly-makes-bad-requests-or-overloads-the-system"></a>Egy adott felhasználó ismételten hibás kérésekből indít telefonhívásokat, és a rendszer túlterhelések.
**Észlelési**. Hitelesíti a felhasználókat, és tartalmazza a felhasználói Azonosítót alkalmazásnaplók.

**Helyreállítás**

* Használjon [Azure API Management] [ api-management] adatátviteli kérésekre a felhasználótól. Lásd: [speciális kérelmet az Azure API Management-szabályozás][api-management-throttling]
* A felhasználó letiltásához.

**Diagnosztika**. A naplófájl minden hitelesítési kérelemre.

### <a name="a-bad-update-was-deployed"></a>A hibás frissítés telepítése megtörtént.
**Észlelési**. Figyelheti az alkalmazás állapotát az Azure portálon keresztül (lásd: [figyelő Azure web app teljesítmény][app-insights-web-apps]) vagy valósítja meg a [mintát figyelési állapotfigyelő végpont] [health-endpoint-monitoring-pattern].

**Helyreállítási**. Használjon több [üzembe helyezési] [ app-service-slots] és állítsa vissza a legutóbbi helyes központi telepítéshez. További információkért lásd: [alapvető webalkalmazás][ra-web-apps-basic].

## <a name="azure-active-directory"></a>Azure Active Directory
### <a name="openid-connect-oidc-authentication-fails"></a>OpenID Connect (OIDC) hitelesítés nem sikerül.
**Észlelési**. Lehetséges probléma módok a következők:

1. Az Azure AD nem érhető el, vagy a hálózati hiba miatt nem érhető el. A hitelesítési végpont a következő URL nem sikerül, és a OIDC köztes kivételt.
2. Az Azure AD-bérlő nem létezik. A hitelesítési végpont a következő URL HTTP-hibakódot ad vissza, és a OIDC köztes kivételt jelez.
3. Felhasználói nem tudják hitelesíteni magukat. Nincs észlelő stratégia szükség; Az Azure AD bejelentkezési hibák kezeli.

**Helyreállítás**

1. A köztes a nem kezelt kivételeket.
2. Kezelni `AuthenticationFailed` események.
3. Egy hibalap átirányítja a felhasználót.
4. Felhasználói újrapróbálkozások.

## <a name="azure-search"></a>Azure Search
### <a name="writing-data-to-azure-search-fails"></a>Írás az Azure Search nem sikerül.
**Észlelési**. A tényleges `Microsoft.Rest.Azure.CloudException` hibák.

**Helyreállítás**

A [Search .NET SDK] [ search-sdk] átmeneti hiba után automatikusan újrapróbálkozik. Az ügyfél SDK által okozott kivételek nem átmeneti hibák kell kezelni.

Az alapértelmezett újrapróbálkozási házirendet használja exponenciális vissza ki. Használjon másik újrapróbálkozási házirendje, hívja meg a `SetRetryPolicy` a a `SearchIndexClient` vagy `SearchServiceClient` osztály. További információkért lásd: [automatikus újrapróbálkozások][auto-rest-client-retry].

**Diagnosztika**. Használjon [keresési forgalom Analytics][search-analytics].

### <a name="reading-data-from-azure-search-fails"></a>Azure Search adatok beolvasása sikertelen.
**Észlelési**. A tényleges `Microsoft.Rest.Azure.CloudException` hibák.

**Helyreállítás**

A [Search .NET SDK] [ search-sdk] átmeneti hiba után automatikusan újrapróbálkozik. Az ügyfél SDK által okozott kivételek nem átmeneti hibák kell kezelni.

Az alapértelmezett újrapróbálkozási házirendet használja exponenciális vissza ki. Használjon másik újrapróbálkozási házirendje, hívja meg a `SetRetryPolicy` a a `SearchIndexClient` vagy `SearchServiceClient` osztály. További információkért lásd: [automatikus újrapróbálkozások][auto-rest-client-retry].

**Diagnosztika**. Használjon [keresési forgalom Analytics][search-analytics].

## <a name="cassandra"></a>Cassandra
### <a name="reading-or-writing-to-a-node-fails"></a>A csomópont írása vagy olvasása sikertelen.
**Észlelési**. A findlogin. A .NET-ügyfelek esetében ez lesz `System.Web.HttpException`. Előfordulhat, hogy más ügyfél más kivétel típusa.  További információkért lásd: [megfelelők legyenek Cassandra hibakezelés](http://www.datastax.com/dev/blog/cassandra-error-handling-done-right).

**Helyreállítás**

* Minden egyes [Cassandra ügyfél](https://wiki.apache.org/cassandra/ClientOptions) saját újrapróbálkozási házirendek és képességeit. További információkért lásd: [megfelelők legyenek Cassandra hibakezelés][cassandra-error-handling].
* Állvány-kompatibilis központi telepítés, a tartalék tartományok pontjain adatok csomópontokkal használata.
* Telepíthet több régióba helyi kvórum konzisztencia. Nem átmeneti hiba lép fel, ha átadja egy másik régióban.

**Diagnosztika**. Alkalmazásnaplók

## <a name="cloud-service"></a>Felhőszolgáltatás
### <a name="web-or-worker-roles-are-unexpectedlybeing-shut-down"></a>Webes vagy feldolgozói szerepköröket vannak váratlanul leáll.
**Észlelési**. A [RoleEnvironment.Stopping] [ RoleEnvironment.Stopping] az esemény.

<strong>Helyreállítási</strong>. Bírálja felül a [RoleEntryPoint.OnStop] [ RoleEntryPoint.OnStop] szabályosan karbantartása metódust. További információkért lásd: [Azure OnStop események kezelésére úgy a jobb oldali] [ onstop-events] (blogbejegyzés).

## <a name="cosmos-db"></a>Cosmos DB 
### <a name="reading-data-fails"></a>Adatok beolvasása sikertelen.
**Észlelési**. A tényleges `System.Net.Http.HttpRequestException` vagy `Microsoft.Azure.Documents.DocumentClientException`.

**Helyreállítás**

* Az SDK automatikusan újrapróbálkozik a sikertelen kísérletek. Az újrapróbálkozások száma és a maximális várakozási idő beállításához konfigurálása `ConnectionPolicy.RetryOptions`. Az ügyfél által okozott kivételek vagy túlmutatnak az újrapróbálkozási szabályzaton, vagy nem átmeneti hibák.
* Ha a Cosmos DB korlátozza az ügyfél kísérleteit, a 429-es HTTP-hibaüzenetet adja vissza. Ellenőrizze a `DocumentClientException` állapotkódját. Ha a hiba 429 következetesen, fontolja meg a gyűjtemény átviteli sebesség értékének növelésével.
    * A MongoDB API-t használ, ha a szolgáltatás a esetén a sávszélesség-szabályozás hibakód 16500 ad vissza.
* A Cosmos DB adatbázis két vagy több régió replikálásához. Összes replikát is olvasható. Az ügyfél SDK-k használata, adja meg a `PreferredLocations` paraméter. Ez az Azure-régiók rendezett listáját. Az összes olvasási kapnak a lista első rendelkezésre álló terület. A kérés nem teljesíthető, ha az ügyfél megpróbál a más régiókból, a listában, sorrendben. További információkért lásd: [telepítője az SQL API-val Azure Cosmos DB globális terjesztési hogyan][cosmosdb-multi-region].

**Diagnosztika**. Ügyféloldali összes hibák naplózására.

### <a name="writing-data-fails"></a>Sikertelen az adatok írásakor.
**Észlelési**. A tényleges `System.Net.Http.HttpRequestException` vagy `Microsoft.Azure.Documents.DocumentClientException`.

**Helyreállítás**

* Az SDK automatikusan újrapróbálkozik a sikertelen kísérletek. Az újrapróbálkozások száma és a maximális várakozási idő beállításához konfigurálása `ConnectionPolicy.RetryOptions`. Az ügyfél által okozott kivételek vagy túlmutatnak az újrapróbálkozási szabályzaton, vagy nem átmeneti hibák.
* Ha a Cosmos DB korlátozza az ügyfél kísérleteit, a 429-es HTTP-hibaüzenetet adja vissza. Ellenőrizze a `DocumentClientException` állapotkódját. Ha a hiba 429 következetesen, fontolja meg a gyűjtemény átviteli sebesség értékének növelésével.
* A Cosmos DB adatbázis két vagy több régió replikálásához. Ha az elsődleges régióban meghibásodik, egy másik régióban lesz előléptetve írni. A feladatátvétel manuálisan is elindítható. Az SDK automatikus felderítést és az útválasztást, végzi, az alkalmazás kódjában folytatja a működést egy feladatátvétel után. Időszakban a feladatátvétel (általában percben) az írási műveletek, az SDK megkeresi az új írási terület lesz nagyobb késleltetéssel járhat.
  További információkért lásd: [telepítője az SQL API-val Azure Cosmos DB globális terjesztési hogyan][cosmosdb-multi-region].
* Tartalékként továbbra is fennáll a dokumentum a biztonsági mentési üzenetsorba, és a várólista később feldolgozni.

**Diagnosztika**. Ügyféloldali összes hibák naplózására.

## <a name="elasticsearch"></a>Elasticsearch
### <a name="reading-data-from-elasticsearch-fails"></a>Adatok beolvasása Elasticsearch sikertelen lesz.
**Észlelési**. Az adott megfelelő findlogin [Elasticsearch ügyfél] [ elasticsearch-client] használja.

**Helyreállítás**

* Egy újrapróbálkozási mechanizmus használata. Minden ügyfél-nak a saját újrapróbálkozási házirendje.
* Telepítse több Elasticsearch csomópontokat, és a magas rendelkezésre állású replikációs használja.

További információkért lásd: [az Azure-on futó Elasticsearch][elasticsearch-azure].

**Diagnosztika**. Figyelési eszközök segítségével Elasticsearch, vagy jelentkezzen az ügyféloldalon összes hiba a tartalom. A "Figyelés" című [az Azure-on futó Elasticsearch][elasticsearch-azure].

### <a name="writing-data-to-elasticsearch-fails"></a>Írás a Elasticsearch meghiúsul.
**Észlelési**. Az adott megfelelő findlogin [Elasticsearch ügyfél] [ elasticsearch-client] használja.  

**Helyreállítás**

* Egy újrapróbálkozási mechanizmus használata. Minden ügyfél-nak a saját újrapróbálkozási házirendje.
* Ha az alkalmazás működését csökkentett konzisztenciaszint, fontolja meg az írás `write_consistency` beállítása `quorum`.

További információkért lásd: [az Azure-on futó Elasticsearch][elasticsearch-azure].

**Diagnosztika**. Figyelési eszközök segítségével Elasticsearch, vagy jelentkezzen az ügyféloldalon összes hiba a tartalom. A "Figyelés" című [az Azure-on futó Elasticsearch][elasticsearch-azure].

## <a name="queue-storage"></a>Queue Storage
### <a name="writing-a-message-to-azure-queue-storage-fails-consistently"></a>Egy üzenet következetesen Azure Queue storage sikertelen írásakor.
**Észlelési**. Miután *N* újrapróbálkozási kísérletek, továbbra is sikertelen lesz, az írási művelet.

**Helyreállítás**

* Az adatok tárolása a helyi gyorsítótárba, és továbbítja a írási műveleteket ad ki tárolási később, amikor a szolgáltatás elérhetővé válik.
* Hozzon létre egy másodlagos várólistát, és írni erre a várólistára, ha az elsődleges queue nem érhető el.

**Diagnosztika**. Használjon [storage mérőszámainak][storage-metrics].

### <a name="the-application-cannot-process-a-particular-message-from-the-queue"></a>Az alkalmazás nem tudja feldolgozni az egy adott üzenetet az üzenetsorból.
**Észlelési**. Adott alkalmazás. Például az üzenet érvénytelen adatokat tartalmaz, vagy az üzleti logika valamilyen okból nem sikerül.

**Helyreállítás**

Az üzenet áthelyezése egy külön várólista. Egy adott várólistán lévő üzenetek vizsgálata külön folyamatban fut.

Érdemes lehet Azure Service Bus üzenetkezelés várólisták, amely biztosítja a [kézbesítetlen levelek várólistájára] [ sb-dead-letter-queue] funkció erre a célra.

> [!NOTE]
> A webjobs-feladatok tárüzenetsort használ, a WebJobs SDK beépített elhalt üzenetek kezelésének biztosít. Lásd: [Azure a queue storage használata a WebJobs SDK][sb-poison-message].

**Diagnosztika**. Alkalmazásnaplózás használja.

## <a name="redis-cache"></a>Redis Cache
### <a name="reading-from-the-cache-fails"></a>A gyorsítótár olvasása sikertelen lesz.
**Észlelési**. A tényleges `StackExchange.Redis.RedisConnectionException`.

**Helyreállítás**

1. Ismételje meg az átmeneti hibák. Az Azure Redis cache automatikusan újrapróbálkozik, lásd: keresztül támogatja [Redis Cache újrapróbálkozási irányelvek][redis-retry].
2. A gyorsítótár-tévesztései nem átmeneti hibák tekinti, és térhet vissza az eredeti adatforrás.

**Diagnosztika**. Használjon [Redis Cache diagnosztika][redis-monitor].

### <a name="writing-to-the-cache-fails"></a>A gyorsítótárba írása sikertelen.
**Észlelési**. A tényleges `StackExchange.Redis.RedisConnectionException`.

**Helyreállítás**

1. Ismételje meg az átmeneti hibák. Az Azure Redis cache automatikusan újrapróbálkozik, lásd: keresztül támogatja [Redis Cache újrapróbálkozási irányelvek][redis-retry].
2. Ha a hiba nem átmeneti, figyelmen kívül hagyhatja, és hogy más tranzakciók később a gyorsítótárba írni.

**Diagnosztika**. Használjon [Redis Cache diagnosztika][redis-monitor].

## <a name="sql-database"></a>SQL Database
### <a name="cannot-connect-to-the-database-in-the-primary-region"></a>Nem lehet kapcsolódni az adatbázishoz az elsődleges régióban.
**Észlelési**. Csatlakozás sikertelen lesz.

**Helyreállítás**

Előfeltétel: Az adatbázis aktív georeplikációt a kell konfigurálni. Lásd: [SQL adatbázis aktív Georeplikáció][sql-db-replication].

* A lekérdezések egy másodlagos másodpéldány olvasni.
* Beszúrások és a frissítések manuális feladatátvétellel egy másodlagos replikára. Lásd: [kezdeményezzen tervezett vagy nem tervezett feladatátvételt az Azure SQL Database][sql-db-failover].

A replika használ egy másik kapcsolati karakterláncot, ezért szüksége lesz a kapcsolati karakterlánc az alkalmazás frissítésére.

### <a name="client-runs-out-of-connections-in-the-connection-pool"></a>Ügyfél elfogy a kapcsolatkészletben található kapcsolatok.
**Észlelési**. A tényleges `System.InvalidOperationException` hibák.

**Helyreállítás**

* Próbálja megismételni a műveletet.
* A kezelési terv, különítse el a kapcsolatkészletek minden használati eset az, hogy egy használati eset nem dominálja az összes kapcsolat.
* Növelje meg a maximális kapcsolatkészletek.

**Diagnosztika**. Alkalmazás naplókat.

### <a name="database-connection-limit-is-reached"></a>Adatbázis-kapcsolathoz megadott korlátot elérte.
**Észlelési**. Az Azure SQL Database egyidejű dolgozók, a bejelentkezési adatok, valamint a munkamenetek számának korlátozása. A korlátozások attól függnek, hogy a szolgáltatási rétegben. További információkért lásd: [erőforrás-korlátozások az Azure SQL Database][sql-db-limits].

Ezek a hibák észlelésére, catch `System.Data.SqlClient.SqlException` , és ellenőrizze a értékének `SqlException.Number` az SQL-hibakódhoz. Kapcsolódó hibakódok listáját lásd: [az SQL Database-ügyfélalkalmazások SQL hibakódjai: adatbázis-kapcsolati hiba és más olyan problémák][sql-db-errors].

**Helyreállítási**. Ezek a hibák átmeneti, minősülnek, így újrapróbálkozás megoldhatja a problémát. Hogy következetesen érte ezeket a hibákat, fontolja meg az adatbázis méretezése.

**Diagnosztika**. -A [sys.event_log] [ sys.event_log] lekérdezés sikeres adatbázis-kapcsolatok, a csatlakozási hibák és a holtpontok adja vissza.

* Hozzon létre egy [riasztási szabály] [ azure-alerts] nem sikerült a következő kapcsolatok.
* Engedélyezése [SQL Database auditing] [ sql-db-audit] és sikertelen bejelentkezések ellenőrzése.

## <a name="service-bus-messaging"></a>Service Bus-üzenetkezelés
### <a name="reading-a-message-from-a-service-bus-queue-fails"></a>A Service Bus-üzenetsorba üzenet olvasása sikertelen lesz.
**Észlelési**. Az ügyfél SDK kivételeket. A Service Bus kivételek alaposztálya [MessagingException][sb-messagingexception-class]. Ha a hiba átmeneti, a `IsTransient` tulajdonság értéke igaz.

További információkért lásd: [Service Bus üzenetküldési kivételek][sb-messaging-exceptions].

**Helyreállítás**

1. Ismételje meg az átmeneti hibák. Lásd: [Service Bus újrapróbálkozási irányelvek][sb-retry].
2. Üzeneteket egyetlen fogadó nem kézbesíthetők helyezi a *kézbesítetlen levelek várólistájára*. A várólista segítségével tekintse meg az üzenetet nem lehet fogadni. Nincs nincs automatikus karbantartása a kézbesítetlen levelek várólistájára. Üzenetek nem maradnak, amíg explicit módon kérheti le azokat. Lásd: [áttekintése a Service Bus kézbesítetlen levelek várólistájának][sb-dead-letter-queue].

### <a name="writing-a-message-to-a-service-bus-queue-fails"></a>Egy üzenet írása egy Service Bus várólista sikertelen lesz.
**Észlelési**. Az ügyfél SDK kivételeket. A Service Bus kivételek alaposztálya [MessagingException][sb-messagingexception-class]. Ha a hiba átmeneti, a `IsTransient` tulajdonság értéke igaz.

További információkért lásd: [Service Bus üzenetküldési kivételek][sb-messaging-exceptions].

**Helyreállítás**

1. A Service Bus-ügyfél automatikusan újrapróbálkozik átmeneti hibák után. Alapértelmezés szerint azt használja az exponenciális vissza ki. Az újrapróbálkozások maximális számát vagy a maximális engedélyezett idő után az ügyfél kivételt jelez. További információkért lásd: [Service Bus újrapróbálkozási irányelvek][sb-retry].
2. A várólista kvóta túllépése, az ügyfél jelez [QuotaExceededException][QuotaExceededException]. A kivétel üzenetének további részleteket nyújt. A várólistában lévő egyes üzenetek kiürítésére újrapróbálkozás előtt, és vegye figyelembe, folyamatos újrapróbálkozások elkerülése érdekében, miközben a rendszer túllépte az áramköri megszakító minta használatával. Győződjön meg arról is, a [BrokeredMessage.TimeToLive] tulajdonság értéke túl magas.
3. A régión belül használatával növelhető a rugalmasság [üzenetsorok és témakörök particionálva][sb-partition]. Nem particionált üzenetsor vagy témakör egyetlen üzenetküldési tárolóra van hozzárendelve. Az üzenetküldési tárolóban nem érhető el, ha az adott üzenetsor vagy témakör összes művelet sikertelen lesz. A particionált üzenetsor vagy témakör particionálása több üzenetküldési tároló között.
4. A további rugalmasságot különböző régiókban két Service Bus-névtér létrehozása, és az üzenet replikálása. A replikálás aktív vagy passzív replikációs is használhatja.

   * Aktív replikáció: az ügyfél minden üzenetet küld mindkét várólisták. A fogadó mindkét várólisták figyel. Egy egyedi azonosítóval rendelkező üzenetek címkét, így az ügyfél elvetheti a duplikált üzenetek.
   * Passzív replikáció: az ügyfél az üzenet elküldése egy olyan sort. Ha nem sikerül, az ügyfél visszavált a várakozási sorba. A fogadó mindkét várólisták figyel. Ezt a módszert csökkenthető az ismétlődő küldött üzenetek száma. Azonban a fogadó kell továbbra is duplikált üzenetek kezeléséhez.

     További információkért lásd: [georeplication beállítását minta] [ sb-georeplication-sample] és [ajánlott eljárások az alkalmazások a Service Bus kimaradások és vészhelyzetek szigetelő](/azure/service-bus-messaging/service-bus-outages-disasters/).

### <a name="duplicate-message"></a>Ismétlődő üzenet.
**Észlelési**. Vizsgálja meg a `MessageId` és `DeliveryCount` az üzenet tulajdonságai.

**Helyreállítás**

* Ha lehetséges tervezze meg az üzenetet, feldolgozási műveletek idempotent kell. Ellenkező esetben tárolására üzenet azonosítói üzeneteket már dolgoznak fel, és ellenőrizze az azonosító egy üzenet feldolgozása előtt.
* Kettős észlelés, engedélyezze a várólista létrehozása `RequiresDuplicateDetection` igaz értékre állítva. Ezzel a beállítással a Service Bus automatikusan törli a ugyanazzal a küldött üzenet `MessageId` előző üzenetben.  Vegye figyelembe a következőket:

  * Ez a beállítás megakadályozza, hogy a duplikált üzenetek a várólista üzembe. Ez nem akadályozza a fogadó ugyanaz az üzenet feldolgozása egynél többször.
  * Kettős észlelés rendelkezik egy olyan időkeretet. Duplikált küldött meghaladja ezt az ablakot, ha nem észlelhető.

**Diagnosztika**. Duplikált üzenetek naplózása.

### <a name="the-application-cannot-process-a-particular-message-from-the-queue"></a>Az alkalmazás nem tudja feldolgozni az egy adott üzenetet az üzenetsorból.
**Észlelési**. Adott alkalmazás. Például az üzenet érvénytelen adatokat tartalmaz, vagy az üzleti logika valamilyen okból nem sikerül.

**Helyreállítás**

Nincsenek két hiba módot kell figyelembe venni.

* A fogadó észleli a hibát. Ebben az esetben az üzenet áthelyezése a kézbesítetlen levelek várólistájára. Később futtassa a kézbesítetlen levelek várólistában lévő üzenetek vizsgálata külön folyamatban.
* A fogadó nem sikerül, az üzenet feldolgozása közepén &mdash; például egy nem kezelt kivétel miatt. Ebben az esetben kezelésére, használja a `PeekLock` mód. Ebben a módban Ha a lejár a zárolást, az üzenet elérhetővé válik többi fogadó számára. Ha az üzenet túllépi a maximális szám vagy az idő élettartamát, az üzenet automatikusan átkerül a kézbesítetlen levelek várólistájára.

További információkért lásd: [áttekintése a Service Bus kézbesítetlen levelek várólistájának][sb-dead-letter-queue].

**Diagnosztika**. Amikor az alkalmazás egy üzenetet a kézbesítetlen levelek várólistájára helyezi, írni egy eseményt az alkalmazás naplóiban.

## <a name="service-fabric"></a>Service Fabric
### <a name="a-request-to-a-service-fails"></a>A szolgáltatásnak küldött kérelemben sikertelen lesz.
**Észlelési**. A szolgáltatás hibát ad vissza.

**Helyreállítás**

* Keresse meg a proxy újra (`ServiceProxy` vagy `ActorProxy`) és a szolgáltatás-vagy aktormetódus hívja meg ismét.
* **Az állapotalapú szolgáltatás**. A megbízható gyűjtemények műveletek burkolása szerepel egy tranzakcióban. Ha nem sikerül, a tranzakció vissza lesz állítva. A kérést, ha a várólista lekért dolgoz fel újra.
* **Állapotmentes szolgáltatások**. Ha a szolgáltatás továbbra is fennáll, külső áruházban adatokat, minden műveletet kell lennie az idempotent.

**Diagnosztika**. Napló

### <a name="service-fabric-node-is-shut-down"></a>Service Fabric-csomópont le van állítva.
**Észlelési**. Megszakítási jogkivonat továbbítódik a szolgáltatás `RunAsync` metódust. A Service Fabric a feladat törli a csomópont leállítása előtt.

**Helyreállítási**. A megszakítási token leállítási észleléséhez használni. Ha a Service Fabric törlését kéri, Befejezés munka, és zárja be `RunAsync` lehető leggyorsabban tegye.

**Diagnosztika**. Alkalmazásnaplók

## <a name="storage"></a>Storage
### <a name="writing-data-to-azure-storage-fails"></a>Sikertelen az adatok Azure Storage való írásakor
**Észlelési**. Az ügyfél megkapja a hibák írása közben.

**Helyreállítás**

1. Próbálja megismételni a műveletet, helyreállítása az átmeneti hibák. A [újrapróbálkozási házirendet] [ Storage.RetryPolicies] az ügyfél SDK kezeli ezt automatikusan.
2. Az áramköri megszakító mintát mellőzése túlságosan megvalósításához.
3. Ha N újrapróbálkozási kísérletek sikertelenek, hajtsa végre a szabályos fallback. Példa:

   * Az adatok tárolása a helyi gyorsítótárba, és továbbítja a írási műveleteket ad ki tárolási később, amikor a szolgáltatás elérhetővé válik.
   * Ha az írási művelet volt, a tranzakciós hatókörű, kártalanítja a tranzakciót.

**Diagnosztika**. Használjon [storage mérőszámainak][storage-metrics].

### <a name="reading-data-from-azure-storage-fails"></a>Azure Storage adatok beolvasása sikertelen.
**Észlelési**. Az ügyfél megkapja a hibák olvasásakor.

**Helyreállítás**

1. Próbálja megismételni a műveletet, helyreállítása az átmeneti hibák. A [újrapróbálkozási házirendet] [ Storage.RetryPolicies] az ügyfél SDK kezeli ezt automatikusan.
2. RA-GRS tárolási Ha az elsődleges végpont olvasása sikertelen, próbálja meg a másodlagos végponti olvasásakor. Az ügyfél SDK oldhatja meg automatikusan. Lásd: [Azure Storage replikációs][storage-replication].
3. Ha *N* újrapróbálkozási sikertelen kísérlet után, a tartalék tenni szabályosan kezd. Ha nem lehet beolvasni a termék lemezkép tárolási, például egy általános helyőrző kép megjelenítése.

**Diagnosztika**. Használjon [storage mérőszámainak][storage-metrics].

## <a name="virtual-machine"></a>Virtuális gép
### <a name="connection-to-a-backend-vm-fails"></a>A háttér virtuális gép nem sikerül kapcsolódni.
**Észlelési**. A hálózati csatlakozási hibák.

**Helyreállítás**

* Telepítse legalább két háttér virtuális gépek rendelkezésre állási csoportba, a terheléselosztó mögött.
* Ha a kapcsolat hiba átmeneti, néha TCP sikeresen újra fog az üzenet elküldése.
* Újrapróbálkozási házirendje megvalósítása az alkalmazásban.
* Állandó vagy nem átmeneti hiba, megvalósítani a [áramköri megszakító] [ circuit-breaker] mintát.
* Ha a hívó VM túllépi ezt a hálózati kilépő, a kimenő várólista betelik. Ha a kimenő várólista következetesen teljes, fontolja meg a kiterjesztése.

**Diagnosztika**. Naplózza a szolgáltatáshatárokon történő eseményeket.

### <a name="vm-instance-becomes-unavailable-or-unhealthy"></a>Virtuálisgép-példány nem érhető el vagy nem megfelelő állapotú lesz.
**Észlelési**. Egy terheléskiegyenlítő [állapotmintáihoz] [ lb-probe] , amely jelzi, hogy a Virtuálisgép-példány állapota kifogástalan. A mintavétel ellenőriznie kell, hogy a kritikus fontosságú funkciók helytelenül válaszol-e.

**Helyreállítási**. Minden alkalmazás szinten több Virtuálisgép-példány üzembe az azonos rendelkezésre állási csoport, és helyezze el a virtuális gépek előtt terheléselosztó. Ha a állapotmintáihoz sikertelen, a terheléselosztó leállítja az új kapcsolatok küld a nem megfelelő állapotú példány.

**Diagnosztika**. -Használni kívánt terheléselosztó [analytics jelentkezzen][lb-monitor].

* A felügyeleti rendszer figyelheti az összes a állapotfigyelési végpontok konfigurálása.

### <a name="operator-accidentally-shuts-down-a-vm"></a>Operátor véletlenül egy virtuális gép leáll.
**Észlelési**. –

**Helyreállítási**. Állítsa be az erőforrás-zárolást a következő `ReadOnly` szintjét. Lásd: [erőforrások az Azure Resource Manager zárolása][rm-locks].

**Diagnosztika**. Használjon [Azure tevékenységi naplóit][azure-activity-logs].

## <a name="webjobs"></a>WebJobs
### <a name="continuous-job-stops-running-when-the-scm-host-is-idle"></a>Folyamatos feladatot leállítja, ha az SCM állomás üresjáratban.
**Észlelési**. A webjobs-feladat függvény cancellation tokent átadni. További információkért lásd: [szabályos leállítást][web-jobs-shutdown].

**Helyreállítási**. Engedélyezze a `Always On` a webalkalmazás-ban történő beállítását. További információkért lásd: [háttérfeladatok futtatása a webjobs-feladatok][web-jobs].

## <a name="application-design"></a>Alkalmazás tervezése
### <a name="application-cant-handle-a-spike-in-incoming-requests"></a>Alkalmazás nem tudja kezelni a csúcs az igények bejövő kérelmeket.
**Észlelési**. Az alkalmazás függ. Tipikus jelenségek:

* A webhely elindul, HTTP 5xx hibakódok ad vissza.
* Függő szolgáltatások, például az adatbázis vagy a tárolási, adatátviteli kérések megkezdése. Keresse meg a HTTP-hibák HTTP 429 (túl sok kérelem), például attól függően, hogy a szolgáltatás.
* HTTP-várólista hossza nő.

**Helyreállítás**

* Terjessze ki a megnövekedett terhelés kezelése érdekében.
* Kerülje az egymásra épülő hibák megzavarhatják a teljes alkalmazás sikertelen mérsékelni. Megoldás stratégiák a következők:

  * Alkalmazzon a [sávszélesség-szabályozás mintát] [ throttling-pattern] túlságosan háttérrendszerek elkerülése érdekében.
  * Használja [várólista alapú terhelés simítás] [ queue-based-load-leveling] puffer kérelmek és egy megfelelő ütemben dolgozza fel őket.
  * Egyes ügyfelek sorrendet. Például ha az alkalmazás ingyenes és fizetős rétegek, sávszélesség-szabályozási ügyfelek a ingyenes szint, de nem fizetős ügyfelek. Lásd: [prioritású várólista mintát][priority-queue-pattern].

**Diagnosztika**. Használjon [App Service diagnosztikai naplózás][app-service-logging]. Használjon például egy szolgáltatás [Azure Naplóelemzés][azure-log-analytics], [Application Insights][app-insights], vagy [New Relic] [ new-relic] a diagnosztikai naplók megismeréséhez.

### <a name="one-of-the-operations-in-a-workflow-or-distributed-transaction-fails"></a>A műveletek a munkafolyamat vagy az elosztott tranzakciók nem sikerül.
**Észlelési**. Miután *N* újrapróbálkozási kísérletek, továbbra is sikertelen.

**Helyreállítás**

* A kezelési terv hajtja végre a [Feladatütemező ügynök felügyelő] [ scheduler-agent-supervisor] kezelése a teljes munkafolyamat mintát.
* Az időtúllépések nem próbálja újra. Ezt a hibát egy alacsony sikerességi arányát van.
* Besorolni a munkaelemet, ahhoz, hogy később próbálja meg újra.

**Diagnosztika**. Minden művelet (sikeres és sikertelen), beleértve a compensating műveletek naplózása. Korrelációs azonosítót használja, így ugyanazon a tranzakción belül minden műveletet nyomon követheti.

### <a name="a-call-to-a-remote-service-fails"></a>A távoli szolgáltatás hívása sikertelen lesz.
**Észlelési**. HTTP-hibakódot.

**Helyreállítás**

1. Ismételje meg az átmeneti hibák.
2. Ha a hívás sikertelen után *N* kísérleteket, egy tartalék művelet végrehajtása. (Alkalmazás adott.)
3. Alkalmazzon a [áramköri megszakító mintát] [ circuit-breaker] kaszkádolt hibák elkerülése érdekében.

**Diagnosztika**. A naplófájl minden távoli hívás sikertelen.

## <a name="next-steps"></a>További lépések
A FMA folyamattal kapcsolatos további információkért lásd: [rugalmas kialakítás a felhőszolgáltatások] [ resilience-by-design-pdf] (PDF-fájl letöltése).

<!-- links -->

[api-management]: https://azure.microsoft.com/documentation/services/api-management/
[api-management-throttling]: /azure/api-management/api-management-sample-flexible-throttling/
[app-insights]: /azure/application-insights/app-insights-overview/
[app-insights-web-apps]: /azure/application-insights/app-insights-azure-web-apps/
[app-service-configure]: /azure/app-service-web/web-sites-configure/
[app-service-logging]: /azure/app-service-web/web-sites-enable-diagnostic-log/
[app-service-slots]: /azure/app-service-web/web-sites-staged-publishing/
[auto-rest-client-retry]: https://github.com/Azure/autorest/tree/master/docs
[azure-activity-logs]: /azure/monitoring-and-diagnostics/monitoring-overview-activity-logs/
[azure-alerts]: /azure/monitoring-and-diagnostics/insights-alerts-portal/
[azure-log-analytics]: /azure/log-analytics/log-analytics-overview/
[BrokeredMessage.TimeToLive]: https://msdn.microsoft.com/library/microsoft.servicebus.messaging.brokeredmessage.timetolive.aspx
[cassandra-error-handling]: http://www.datastax.com/dev/blog/cassandra-error-handling-done-right
[circuit-breaker]: https://msdn.microsoft.com/library/dn589784.aspx
[cosmosdb-multi-region]: /azure/cosmos-db/tutorial-global-distribution-sql-api
[elasticsearch-azure]: ../elasticsearch/index.md
[elasticsearch-client]: https://www.elastic.co/guide/en/elasticsearch/client/index.html
[health-endpoint-monitoring-pattern]: https://msdn.microsoft.com/library/dn589789.aspx
[onstop-events]: https://azure.microsoft.com/blog/the-right-way-to-handle-azure-onstop-events/
[lb-monitor]: /azure/load-balancer/load-balancer-monitor-log/
[lb-probe]: /azure/load-balancer/load-balancer-custom-probe-overview/#learn-about-the-types-of-probes
[new-relic]: https://newrelic.com/
[priority-queue-pattern]: https://msdn.microsoft.com/library/dn589794.aspx
[queue-based-load-leveling]: https://msdn.microsoft.com/library/dn589783.aspx
[QuotaExceededException]: https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.quotaexceededexception.aspx
[ra-web-apps-basic]: ../reference-architectures/app-service-web-app/basic-web-app.md
[redis-monitor]: /azure/redis-cache/cache-how-to-monitor/
[redis-retry]: ../best-practices/retry-service-specific.md#azure-redis-cache
[resilience-by-design-pdf]: http://download.microsoft.com/download/D/8/C/D8C599A4-4E8A-49BF-80EE-FE35F49B914D/Resilience_by_Design_for_Cloud_Services_White_Paper.pdf
[RoleEntryPoint.OnStop]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleentrypoint.onstop.aspx
[RoleEnvironment.Stopping]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.serviceruntime.roleenvironment.stopping.aspx
[rm-locks]: /azure/azure-resource-manager/resource-group-lock-resources/
[sb-dead-letter-queue]: /azure/service-bus-messaging/service-bus-dead-letter-queues/
[sb-georeplication-sample]: https://github.com/Azure-Samples/azure-servicebus-messaging-samples/tree/master/GeoReplication
[sb-messagingexception-class]: https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.messagingexception.aspx
[sb-messaging-exceptions]: /azure/service-bus-messaging/service-bus-messaging-exceptions/
[sb-outages]: /azure/service-bus-messaging/service-bus-outages-disasters/#protecting-queues-and-topics-against-datacenter-outages-or-disasters
[sb-partition]: /azure/service-bus-messaging/service-bus-partitioning/
[sb-poison-message]: /azure/app-service-web/websites-dotnet-webjobs-sdk-storage-queues-how-to/#poison
[sb-retry]: ../best-practices/retry-service-specific.md#service-bus
[search-sdk]: https://msdn.microsoft.com/library/dn951165.aspx
[scheduler-agent-supervisor]: https://msdn.microsoft.com/library/dn589780.aspx
[search-analytics]: /azure/search/search-traffic-analytics/
[sql-db-audit]: /azure/sql-database/sql-database-auditing-get-started/
[sql-db-errors]: /azure/sql-database/sql-database-develop-error-messages/#resource-governance-errors
[sql-db-failover]: /azure/sql-database/sql-database-geo-replication-failover-portal/
[sql-db-limits]: /azure/sql-database/sql-database-resource-limits/
[sql-db-replication]: /azure/sql-database/sql-database-geo-replication-overview/
[storage-metrics]: https://msdn.microsoft.com/library/dn782843.aspx
[storage-replication]: /azure/storage/storage-redundancy/
[Storage.RetryPolicies]: https://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.aspx
[sys.event_log]: https://msdn.microsoft.com/library/dn270018.aspx
[throttling-pattern]: https://msdn.microsoft.com/library/dn589798.aspx
[web-jobs]: /azure/app-service-web/web-sites-create-web-jobs/
[web-jobs-shutdown]: /azure/app-service-web/websites-dotnet-webjobs-sdk-storage-queues-how-to/#graceful
