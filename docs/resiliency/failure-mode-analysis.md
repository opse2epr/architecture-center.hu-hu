---
title: A hibaállapot elemzése
description: Útmutató a hibaállapot-elemzést végez az Azure-on alapuló felhőalapú megoldásokat.
author: MikeWasson
ms.date: 05/07/2018
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-design-principles
ms.custom: resiliency
ms.openlocfilehash: 0d89570ca42aa087a9c18148b5a4019b6f348e6b
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/17/2019
ms.locfileid: "59640974"
---
# <a name="failure-mode-analysis-for-azure-applications"></a>A hibaállapot elemzése az Azure-alkalmazásokhoz

A hibaállapot elemzése (FMA) egy olyan folyamat, amellyel a rugalmasság rendszerbe, a rendszer lehetséges meghibásodási pontok azonosításával. Az FMA architektúra és kialakítás fázisai részének kell lennie, úgy, hogy a hiba utáni helyreállítás hozhat létre a rendszer az elejétől.

A következő általános folyamata FMA elvégzésére:

1. Azonosítsa a rendszer minden összetevője. Például a külső függőségei, mint például az identitás-szolgáltatóktól, harmadik féltől származó szolgáltatásokkal, és így tovább.
2. Az egyes összetevők azonosíthatja a lehetséges hibák, amely akkor fordulhat elő. Előfordulhat, hogy az adott összetevőt egynél több hibaállapot. Például kell olvasási hibák fontolja meg, és külön-külön írja a hibák, mert a hatás és a lehetséges megoldások eltérő lesz.
3. Minden egyes hiba mód szerint a teljes kockázatot sebessége. Vegye figyelembe a következőket:

   - Mi az a hiba esélyét. Az viszonylag közös? Ritka Extrememly? Nem kell a pontos szám; erre a célra azt a prioritási rangsor.
   - Mi az a az alkalmazás rendelkezésre állását, az adatvesztést, a költség és üzletmenet szempontjából gyakorolt hatást?

4. Minden egyes hiba üzemmódban határozza meg az alkalmazás hogyan válaszol és helyreállítani. Érdemes kompromisszumot kínál a költség- és alkalmazás összetettségét.

Az FMA-folyamat kiindulási pontként a cikkben egy katalógus, lehetséges hibaállapotokat és azok. A katalógus technológia vagy Azure-szolgáltatás, valamint az alkalmazásszintű Tervező általános kategória szerint vannak rendszerezve. A katalógus nem teljes, de ismerteti az alapvető Azure számos szolgáltatások.

## <a name="app-service"></a>App Service

<!-- markdownlint-disable MD026 -->

### <a name="app-service-app-shuts-down"></a>App Service-alkalmazás leáll.

**Észlelési**. Lehetséges okok:

- Várt leállás

  - Az operátornak leállítja az alkalmazás; Ha például az Azure portal használatával.
  - Az alkalmazás lett távolítva a memóriából, mert az inaktív volt. (Csak akkor, ha a `Always On` beállítás le van tiltva.)

- Nem tervezett leállás

  - Az alkalmazás összeomlik.
  - Az App Service Virtuálisgép-példány nem érhető el.

Application_End naplózási fog feltárhatja az alkalmazás tartomány leállítása (helyreállítható folyamatösszeomlás), és az egyetlen módja az alkalmazás tartomány leállások olvasásra.

**Helyreállítás:**

- Ha a Leállítás várt az alkalmazás leállítási esemény segítségével szabályosan leállítani. Ha például az ASP.NET, használja a `Application_End` metódust.
- Ha az alkalmazás volt távolítva a memóriából, míg az inaktív, automatikusan újraindul a következő kérelemnél. Azonban az "hidegindítás" költség számolunk.
- Az alkalmazás eltávolítása folyamatban, míg az inaktív megelőzése érdekében engedélyezze a `Always On` a webalkalmazás beállítását. Lásd: [webalkalmazások konfigurálása az Azure App Service][app-service-configure].
- Az operátornak megakadályozása az alkalmazás leállítása érdekében állítsa be az erőforrás-kezelővel rendelkező `ReadOnly` szintjét. Lásd: [zárolhat erőforrásokat az Azure Resource Manager][rm-locks].
- Ha az alkalmazás összeomlik, vagy egy App Service virtuális gép elérhetetlenné válik, az App Service automatikusan újraindítja az alkalmazást.

**Diagnosztikai**. Az alkalmazásnaplók, és a webkiszolgáló-naplókkal. Lásd: [az Azure App Service web Apps-alkalmazások diagnosztikai célú naplózásának engedélyezése][app-service-logging].

### <a name="a-particular-user-repeatedly-makes-bad-requests-or-overloads-the-system"></a>Egy adott felhasználó többször is lehetővé teszi a hibás kérelmek vagy beáll a rendszer.

**Észlelési**. Felhasználók hitelesítése, és tartalmazza a felhasználói azonosító az alkalmazásnaplókat.

**Helyreállítás:**

- Használat [Azure API Management] [ api-management] késleltetési kérelmekre a felhasználó elől. Lásd: [speciális kérésszabályzás az Azure API Management szolgáltatással][api-management-throttling]
- A felhasználó letiltása.

**Diagnosztikai**. A naplófájl minden hitelesítési kérelemre.

### <a name="a-bad-update-was-deployed"></a>Egy rossz frissítés telepítve lett.

**Észlelési**. Alkalmazás állapotát az Azure portálon keresztül (lásd: [WebApp teljesítményének figyelése Azure][app-insights-web-apps]), illetve végrehajtja a [állapot végponti monitorozását végző minta] [health-endpoint-monitoring-pattern].

**Helyreállítás:**. Több [üzembe helyezési pontok] [ app-service-slots] , és állítsa vissza a legutóbbi megfelelően működő üzemelő példányhoz. További információkért lásd: [alapszintű webalkalmazás][ra-web-apps-basic].

## <a name="azure-active-directory"></a>Azure Active Directory

### <a name="openid-connect-oidc-authentication-fails"></a>OpenID Connect (oidc) Megoldást a hitelesítés sikertelen.

**Észlelési**. Lehetséges meghibásodási módok a következők:

1. Azure ad-ben nem érhető el, vagy hálózati problémák miatt nem érhető el. Átirányítás a hitelesítési végpontra meghiúsul, és a OIDC közbenső kivételt jelez.
2. Az Azure AD-bérlő nem létezik. A hitelesítési végpontot való átirányítást a HTTP-hibakódot adja vissza, és a OIDC közbenső kivételt jelez.
3. Felhasználói nem tudják hitelesíteni magukat. Nincsenek észlelési stratégia nem szükséges; Az Azure AD bejelentkezési hibák kezeli.

**Helyreállítás:**

1. Nem kezelt kivételeket az a közbenső szoftverek.
2. Kezelni `AuthenticationFailed` eseményeket.
3. Hibalap átirányítja a felhasználót.
4. Felhasználói újrapróbálkozásokat.

## <a name="azure-search"></a>Azure Search

### <a name="writing-data-to-azure-search-fails"></a>Azure Search adatok írása sikertelen lesz.

**Észlelési**. A tényleges `Microsoft.Rest.Azure.CloudException` hibákat.

**Helyreállítás:**

A [Search .NET SDK] [ search-sdk] átmeneti hibák után automatikusan újrapróbálkozik. Az ügyfél-SDK által okozott kivételek nem átmeneti hibák kell kezelni.

Az alapértelmezett újrapróbálkozási szabályzatát használja az exponenciális visszatartás. Eltérő újrapróbálkozási szabályzatok használja, hívja meg `SetRetryPolicy` a a `SearchIndexClient` vagy `SearchServiceClient` osztály. További információkért lásd: [automatikus újrapróbálkozások][auto-rest-client-retry].

**Diagnosztikai**. Használat [forgalmi elemzések keresése][search-analytics].

### <a name="reading-data-from-azure-search-fails"></a>Az Azure Search adatok olvasása sikertelen lesz.

**Észlelési**. A tényleges `Microsoft.Rest.Azure.CloudException` hibákat.

**Helyreállítás:**

A [Search .NET SDK] [ search-sdk] átmeneti hibák után automatikusan újrapróbálkozik. Az ügyfél-SDK által okozott kivételek nem átmeneti hibák kell kezelni.

Az alapértelmezett újrapróbálkozási szabályzatát használja az exponenciális visszatartás. Eltérő újrapróbálkozási szabályzatok használja, hívja meg `SetRetryPolicy` a a `SearchIndexClient` vagy `SearchServiceClient` osztály. További információkért lásd: [automatikus újrapróbálkozások][auto-rest-client-retry].

**Diagnosztikai**. Használat [forgalmi elemzések keresése][search-analytics].

## <a name="cassandra"></a>Cassandra

### <a name="reading-or-writing-to-a-node-fails"></a>Csomópont írása és olvasása sikertelen lesz.

**Észlelési**. A findlogin. A .NET-ügyfelek esetében ez általában lesz `System.Web.HttpException`. Előfordulhat, hogy a többi ügyfél más kivételtípusok.  További információkért lásd: [Cassandra hibakezelés jobb kész](https://www.datastax.com/dev/blog/cassandra-error-handling-done-right).

**Helyreállítás:**

- Minden egyes [Cassandra ügyfél](https://wiki.apache.org/cassandra/ClientOptions) saját újrapróbálkozási szabályzatok és képességekkel rendelkezik. További információkért lásd: [Cassandra hibakezelés jobb kész][cassandra-error-handling].
- Állvány-kompatibilis telepítése, a tartalék tartományok között elosztott adatcsomópontokkal használható.
- Üzembe helyezés több régióban helyi kvórum konzisztencia. Nem átmeneti hiba történik, ha feladatátvételt egy másik régióba.

**Diagnosztikai**. Alkalmazásnaplók

## <a name="cloud-service"></a>Felhőszolgáltatás

### <a name="web-or-worker-roles-are-unexpectedlybeing-shut-down"></a>Webes és feldolgozói szerepkörökre is váratlanul leáll.

**Észlelési**. A [RoleEnvironment.Stopping] [ RoleEnvironment.Stopping] esemény lesz elindítva.

**Helyreállítási**. Bírálja felül a [RoleEntryPoint.OnStop] [ RoleEntryPoint.OnStop] metódus szabályosan karbantartása. További információkért lásd: [Azure OnStop események kezeléséhez a jobb megoldást] [ onstop-events] (blog).

## <a name="cosmos-db"></a>Cosmos DB

### <a name="reading-data-fails"></a>Adatok beolvasása sikertelen.

**Észlelési**. A tényleges `System.Net.Http.HttpRequestException` vagy `Microsoft.Azure.Documents.DocumentClientException`.

**Helyreállítás:**

- Az SDK automatikusan újrapróbálkozik a sikertelen kísérletekkel. Konfigurálja az újrapróbálkozások számának és a maximális várakozási idő `ConnectionPolicy.RetryOptions`. Az ügyfél által okozott kivételek vagy túlmutatnak az újrapróbálkozási szabályzaton, vagy nem átmeneti hibák.
- Ha a Cosmos DB korlátozza az ügyfél kísérleteit, a 429-es HTTP-hibaüzenetet adja vissza. Ellenőrizze a `DocumentClientException` állapotkódját. Ha a 429-es hiba rendszeresen, érdemes megfontolni a gyűjtemény átviteli sebesség értékét.
  - Ha a MongoDB API-t használ, a szolgáltatás hibakódot ad vissza hibát 16500 szabályozás.
- Replikálja a Cosmos DB-adatbázis legalább két régióban. Összes replika olvashatók legyenek. Az ügyfél SDK-k használatával, adja meg a `PreferredLocations` paraméter. Ez az Azure-régiók rendezett listája. A lista első elérhető régiójába olvasások kapnak. A kérelem meghiúsul, az ügyfél megpróbálja a más régiókban a listában, sorrendben. További információkért lásd: [beállítása az Azure Cosmos DB globális terjesztését az SQL API használatával][cosmosdb-multi-region].

**Diagnosztikai**. Jelentkezzen az ügyféloldalon hibákat.

### <a name="writing-data-fails"></a>Adatok írása sikertelen lesz.

**Észlelési**. A tényleges `System.Net.Http.HttpRequestException` vagy `Microsoft.Azure.Documents.DocumentClientException`.

**Helyreállítás:**

- Az SDK automatikusan újrapróbálkozik a sikertelen kísérletekkel. Konfigurálja az újrapróbálkozások számának és a maximális várakozási idő `ConnectionPolicy.RetryOptions`. Az ügyfél által okozott kivételek vagy túlmutatnak az újrapróbálkozási szabályzaton, vagy nem átmeneti hibák.
- Ha a Cosmos DB korlátozza az ügyfél kísérleteit, a 429-es HTTP-hibaüzenetet adja vissza. Ellenőrizze a `DocumentClientException` állapotkódját. Ha a 429-es hiba rendszeresen, érdemes megfontolni a gyűjtemény átviteli sebesség értékét.
- Replikálja a Cosmos DB-adatbázis legalább két régióban. Ha az elsődleges régió nem sikerül, egy másik régióba állásúvá lesz előléptetve írni. A feladatátvétel manuálisan is indíthat. Az SDK azért automatikus felderítés és az útválasztást, így az alkalmazás kódja továbbra is működik a feladatátvétel után. Feladatátvételi ideje (általában perc) alatt az írási műveletek lesz nagyobb késést, az SDK megkeresi az új írási régiót. További információkért lásd: [beállítása az Azure Cosmos DB globális terjesztését az SQL API használatával][cosmosdb-multi-region].
- Tartalékként egy biztonsági várólistát a dokumentum tárolására és feldolgozására később a az üzenetsorba.

**Diagnosztikai**. Jelentkezzen az ügyféloldalon hibákat.

## <a name="elasticsearch"></a>Elasticsearch

### <a name="reading-data-from-elasticsearch-fails"></a>Az Elasticsearch történő adatkiolvasás sikertelen lesz.

**Észlelési**. Az adott megfelelő findlogin [Elasticsearch ügyfél] [ elasticsearch-client] használja.

**Helyreállítás:**

- Újrapróbálkozási mechanizmus használata. Minden ügyfél saját újrapróbálkozási szabályzatok rendelkezik.
- Több Elasticsearch csomópont telepítéséhez és használatához a replikációt a magas rendelkezésre állás érdekében.

További információkért lásd: [Elasticsearch futtatása az Azure-ban][elasticsearch-azure].

**Diagnosztikai**. Használjon az elasticsearch számára, vagy jelentkezzen az ügyféloldalon hibákat az adattartalomban. A "Figyelés" című [Elasticsearch futtatása az Azure-ban][elasticsearch-azure].

### <a name="writing-data-to-elasticsearch-fails"></a>Adatok írása az Elasticsearch meghiúsul.

**Észlelési**. Az adott megfelelő findlogin [Elasticsearch ügyfél] [ elasticsearch-client] használja.

**Helyreállítás:**

- Újrapróbálkozási mechanizmus használata. Minden ügyfél saját újrapróbálkozási szabályzatok rendelkezik.
- Ha az alkalmazás működését csökkentett konzisztenciaszint, fontolja meg az írás `write_consistency` beállításaként `quorum`.

További információkért lásd: [Elasticsearch futtatása az Azure-ban][elasticsearch-azure].

**Diagnosztikai**. Használjon az elasticsearch számára, vagy jelentkezzen az ügyféloldalon hibákat az adattartalomban. A "Figyelés" című [Elasticsearch futtatása az Azure-ban][elasticsearch-azure].

## <a name="queue-storage"></a>Queue Storage

### <a name="writing-a-message-to-azure-queue-storage-fails-consistently"></a>Egy üzenet következetesen Azure Queue storage sikertelen írja.

**Észlelési**. Miután *N* megpróbálja próbálkozzon újra, továbbra is sikertelen lesz, az írási művelet.

**Helyreállítás:**

- A data Store a helyi gyorsítótárban, és továbbítsa az írások később, a storage, amikor elérhetővé válik a szolgáltatás.
- Másodlagos várólista létrehozása, és írása az üzenetsornak, ha az elsődleges üzenetsornak nem érhető el.

**Diagnosztikai**. Használat [tármetrikák][storage-metrics].

### <a name="the-application-cannot-process-a-particular-message-from-the-queue"></a>Az alkalmazás nem tudja feldolgozni egy adott üzenetet az üzenetsorból.

**Észlelési**. Adott alkalmazás. Ha például az üzenet érvénytelen adatokat tartalmaz, vagy valamilyen okból meghiúsul az üzleti logika.

**Helyreállítás:**

Az üzenetet egy különálló sorban helyezi át. Megvizsgálhatja az adott üzenetsorban lévő üzenetek külön folyamatban fut.

Fontolja meg az Azure Service Bus-üzenetkezelés várólisták, amely biztosítja a [kézbesítetlen levelek várólistájára] [ sb-dead-letter-queue] funkció erre a célra.

> [!NOTE]
> Ha a tároló-üzenetsorok használata a WebJobs, a WebJobs SDK-t biztosít beépített ártalmas üzenetek kezelése. Lásd: [Azure queue storage használata a WebJobs SDK-val][sb-poison-message].

**Diagnosztikai**. Alkalmazásnaplózás használja.

## <a name="redis-cache"></a>Redis Cache

### <a name="reading-from-the-cache-fails"></a>A gyorsítótárban való olvasás sikertelen lesz.

**Észlelési**. A tényleges `StackExchange.Redis.RedisConnectionException`.

**Helyreállítás:**

1. Ismételje meg az átmeneti hibák. Az Azure Redis cache támogatja a beépített újrapróbálkozási keresztül lásd [Redis Cache újrapróbálkozási irányelvei][redis-retry].
2. Gyorsítótár-tévesztés nem átmeneti hibák kezeli, és visszatér az eredeti adatforrásból.

**Diagnosztikai**. Használat [Redis gyorsítótár-diagnosztikát][redis-monitor].

### <a name="writing-to-the-cache-fails"></a>A gyorsítótárba való írás sikertelen lesz.

**Észlelési**. A tényleges `StackExchange.Redis.RedisConnectionException`.

**Helyreállítás:**

1. Ismételje meg az átmeneti hibák. Az Azure Redis cache támogatja a beépített újrapróbálkozási keresztül lásd [Redis Cache újrapróbálkozási irányelvei][redis-retry].
2. Ha a hiba nem átmeneti, figyelmen kívül hagyásához, és lehetővé teszik a később írni a gyorsítótár egyéb tranzakció.

**Diagnosztikai**. Használat [Redis gyorsítótár-diagnosztikát][redis-monitor].

## <a name="sql-database"></a>SQL Database

### <a name="cannot-connect-to-the-database-in-the-primary-region"></a>Nem lehet csatlakozni az adatbázis az elsődleges régióba.

**Észlelési**. Csatlakozás sikertelen lesz.

**Helyreállítás:**

Előfeltétel: Az adatbázis aktív georeplikációt kell konfigurálni. Lásd: [SQL adatbázis aktív Georeplikációt][sql-db-replication].

- Lekérdezések olvassa el a másodlagos replikáról.
- Beszúrások és frissítések, a manuális feladatátvételt egy másodlagos replikára. Lásd: [kezdeményezzen egy tervezett vagy nem tervezett feladatátvétel az Azure SQL Database][sql-db-failover].

A replika egy másik kapcsolati karakterláncot használja, így a kell az alkalmazás a kapcsolati karakterlánc frissítése.

### <a name="client-runs-out-of-connections-in-the-connection-pool"></a>Ügyfél elfogy a kapcsolatkészletben található kapcsolatok.

**Észlelési**. A tényleges `System.InvalidOperationException` hibákat.

**Helyreállítás:**

- Próbálja megismételni a műveletet.
- Egy kezelési terv, különítse el az egyes használati esetekhez kapcsolatkészletek úgy, hogy egy használati eset nem uralja összes kapcsolatot.
- Növelje a maximális kapcsolódási készleteket.

**Diagnosztikai**. Az alkalmazásnaplók.

### <a name="database-connection-limit-is-reached"></a>Adatbázis kapcsolati korlátot.

**Észlelési**. Az Azure SQL Database egyidejű feldolgozók, bejelentkezések és munkamenetek számát korlátozza. A korlátok a szolgáltatási rétegben függenek. További információkért lásd: [Azure SQL Database erőforrás-korlátozások][sql-db-limits].

Ezek a hibák észlelése, a tényleges `System.Data.SqlClient.SqlException` , és ellenőrizze a értékét `SqlException.Number` számára az SQL-hibakód. Kapcsolódó hibakódok listáját lásd: [az SQL Database-ügyfélalkalmazások SQL-hibakódok: Adatbázis-kapcsolódási hiba és más jellegű problémákat][sql-db-errors].

**Helyreállítási**. Ezek a hibák átmeneti, minősülnek, így újra próbálkozik, azzal megoldhatja a problémát. Ha eléri a hibák folyamatosan, fontolja meg az adatbázison.

**Diagnosztikai**. -A [sys.event_log] [ sys.event_log] lekérdezés sikeres adatbázis-kapcsolatok, a csatlakozási hibák és a holtpontok adja vissza.

- Hozzon létre egy [riasztási szabály] [ azure-alerts] kapcsolatok nem sikerült.
- Engedélyezése [SQL Database naplózási funkciójának] [ sql-db-audit] , és ellenőrizze a sikertelen bejelentkezéseket.

## <a name="service-bus-messaging"></a>Service Bus-üzenetkezelés

### <a name="reading-a-message-from-a-service-bus-queue-fails"></a>A Service Bus-üzenetsorba egy üzenet olvasása sikertelen lesz.

**Észlelési**. Az ügyfél SDK-t a kivételeket. A Service Bus-kivétel alaposztálya [Istransient][sb-messagingexception-class]. Ha a hiba átmeneti, a `IsTransient` tulajdonság igaz értékű.

További információkért lásd: [Service Bus-üzenetkezelés kivételei][sb-messaging-exceptions].

**Helyreállítás:**

1. Ismételje meg az átmeneti hibák. Lásd: [a Service Bus újrapróbálkozási irányelvei][sb-retry].
2. Nem lehet kézbesíteni címzettnek sem üzenetek kerülnek egy *kézbesítetlen levelek várólistájára*. Ez a várólista segítségével megtekintheti, mely üzeneteket mely nem fogadható. Nincs automatikus tisztítására nincs a kézbesítetlen levelek várólistájára vonatkozik. Az üzenetek van maradnak, amíg explicit módon kérheti le azokat. Lásd: [a Service Bus – áttekintés kézbesíthetetlen levelek sorai][sb-dead-letter-queue].

### <a name="writing-a-message-to-a-service-bus-queue-fails"></a>Üzenet írásának egy Service Bus queue sikertelen lesz.

**Észlelési**. Az ügyfél SDK-t a kivételeket. A Service Bus-kivétel alaposztálya [Istransient][sb-messagingexception-class]. Ha a hiba átmeneti, a `IsTransient` tulajdonság igaz értékű.

További információkért lásd: [Service Bus-üzenetkezelés kivételei][sb-messaging-exceptions].

**Helyreállítás:**

1. A Service Bus-ügyfél automatikusan újrapróbálkozik átmeneti hibák után. Alapértelmezés szerint használja az exponenciális visszatartás. Az újrapróbálkozások maximális számát vagy a maximális időkorlát után az ügyfél kivételt jelez. További információkért lásd: [a Service Bus újrapróbálkozási irányelvei][sb-retry].
2. A várólista kvóta túllépése esetén az ügyfél jelez [QuotaExceededException][QuotaExceededException]. A kivételre vonatkozó üzenet a további részleteket nyújt. A várólistában lévő egyes üzenetek kiürítési újrapróbálkozás előtt, és fontolja meg az áramkör-megszakító minta elkerülése érdekében folyamatos próbálkozások során a kvóta túllépése. Győződjön meg arról is, a [BrokeredMessage.TimeToLive] tulajdonság értéke túl magas.
3. Egy adott régión belül használatával növelhető a rugalmasság [particionált üzenetsorok és témakörök][sb-partition]. Egy nem particionált üzenetsor vagy témakör egyetlen üzenetküldési tárolóra van hozzárendelve. Az üzenetküldési tárolóban nem érhető el, ha az üzenetsor vagy témakör összes művelet sikertelen lesz. A particionált üzenetsor vagy témakör particionálása több üzenetküldési tároló között.
4. A további rugalmasság érdekében különböző régiókban lévő két a Service Bus-névtér létrehozása, és replikálja az üzeneteket. A replikálás aktív vagy passzív replikációs használhat.

    - Aktív replikáció: Az ügyfél mindkét üzenetsorok minden üzenetet küld. A fogadó mindkét üzenetsorok figyel. Egy egyedi azonosítóval rendelkező üzenetek címkézése, így az ügyfél képes vesse el az ismétlődő üzeneteket.
    - Passzív replikáció: Az ügyfél az üzenetet küld egy üzenetsorba. Ha hiba történik, az ügyfél visszavált az egyéb üzenetsorba. A fogadó mindkét üzenetsorok figyel. Ez a megközelítés csökkenti a duplikált küldött üzenetek számát. A fogadó azonban továbbra is kell kezelnie ismétlődő üzeneteket.

    További információkért lásd: [gyorsítótárakban minta] [ sb-georeplication-sample] és [ajánlott eljárásai az alkalmazások a Service Bus leállásainak és katasztrófákkal szembeni szigetelő](/azure/service-bus-messaging/service-bus-outages-disasters/).

### <a name="duplicate-message"></a>Ismétlődő üzenetek.

**Észlelési**. Vizsgálja meg a `MessageId` és `DeliveryCount` az üzenet tulajdonságait.

**Helyreállítás:**

- Ha lehetséges tervezze meg az üzenetet feldolgozási műveletekhez, hogy idempotensek legyenek. Ellenkező esetben tárolására Üzenetazonosítók üzenetek feldolgozása már, és ellenőrizze az azonosító egy üzenet feldolgozása előtt.
- Ismétlődések észlelésének engedélyezése az üzenetsorba való létrehozásával `RequiresDuplicateDetection` igaz értékre kell állítani. Ezzel a beállítással a Service Bus automatikusan töröl bármely ugyanazzal a küldött üzenet `MessageId` előző üzenetnek számít.  Vegye figyelembe a következőket:

  - Ez a beállítás megakadályozza, hogy az ismétlődő üzeneteket az üzenetsorba való. A fogadó nem gátolja az ugyanazon üzenet feldolgozását egynél többször.
  - Duplikáltelem-észlelési rendelkezik. Duplicitní küldése meghaladja ezt az ablakot, ha nem észlelhető.

**Diagnosztikai**. Ismétlődő üzenetek naplózása.

### <a name="the-application-cant-process-a-particular-message-from-the-queue"></a>Az alkalmazás nem tudja feldolgozni egy adott üzenetet az üzenetsorból.

**Észlelési**. Adott alkalmazás. Ha például az üzenet érvénytelen adatokat tartalmaz, vagy valamilyen okból meghiúsul az üzleti logika.

**Helyreállítás:**

Nincsenek két hibaállapot kell figyelembe venni.

- A fogadó észleli a hibát. Ebben az esetben az üzenet áthelyezése a kézbesítetlen levelek várólistájára vonatkozik. Később futtatásához egy külön folyamatot kell vizsgálja meg az üzeneteket a kézbesítetlen levelek várólistájára vonatkozik.
- A fogadó meghiúsul az üzenet feldolgozásával közepén &mdash; például egy nem kezelt kivétel miatt. Ebben az esetben kezelése érdekében használjon `PeekLock` mód. Ebben a módban a zárolás lejárta, ha az üzenet elérhetővé válik a többi fogadó számára. Ha az üzenet túllépi a kézbesítések maximális száma vagy a time-to-live, az üzenet automatikusan átkerül a kézbesítetlen levelek várólistájára vonatkozik.

További információkért lásd: [a Service Bus – áttekintés kézbesíthetetlen levelek sorai][sb-dead-letter-queue].

**Diagnosztikai**. Minden alkalommal, amikor az alkalmazás egy üzenetet helyezi át a kézbesítetlen levelek várólistájára vonatkozik, az esemény írni az alkalmazásnaplókat.

## <a name="service-fabric"></a>Service Fabric

### <a name="a-request-to-a-service-fails"></a>Egy szolgáltatás irányuló kérelem sikertelen lesz.

**Észlelési**. A szolgáltatás hibát ad vissza.

**Helyreállítás:**

- Keresse meg a proxy újra (`ServiceProxy` vagy `ActorProxy`) és a szolgáltatás/aktor metódust hívja meg újra.
- **Állapotalapú szolgáltatás**. A reliable collections műveletek egy tranzakció wrap funkciót. Ha hiba történik, a tranzakció vissza lesz állítva. A kérést, ha lekérte az üzenetsorból, lesz újból fel kell dolgozni.
- **Az állapotmentes szolgáltatás**. Ha a szolgáltatás továbbra is fennáll, az adatokat egy külső tároló, minden művelet kell, hogy idempotensek legyenek.

**Diagnosztikai**. Napló

### <a name="service-fabric-node-is-shut-down"></a>Service Fabric-csomópont leáll.

**Észlelési**. A szolgáltatás megszakítás jogkivonat átadott `RunAsync` metódust. A Service Fabric megszakítja a feladatot a csomópont leállítása előtt.

**Helyreállítási**. A megszakítás jogkivonat segítségével észlelheti a Leállítás. Amikor a Service Fabric visszavonás, Befejezés minden olyan munkahelyi, és lépjen ki `RunAsync` lehető leggyorsabb.

**Diagnosztikai**. Alkalmazásnaplók

## <a name="storage"></a>Storage

### <a name="writing-data-to-azure-storage-fails"></a>Írás adatokat az Azure Storage sikertelen

**Észlelési**. Az ügyfél hibát kap, írásakor.

**Helyreállítás:**

1. Próbálja megismételni a műveletet, helyreállítása az átmeneti hibák. A [újrapróbálkozási szabályzat] [ Storage.RetryPolicies] az ügyfél SDK ezt automatikusan megteszi.
2. Bemutatjuk az áramkör-megszakító minta elsöprő tárolási elkerülése érdekében.
3. Ha N újrapróbálkozási kísérletek sikertelenek, végezze el a szabályos tartalék. Példa:

   - A data Store a helyi gyorsítótárban, és továbbítsa az írások később, a storage, amikor elérhetővé válik a szolgáltatás.
   - Ha az írási művelet volt egy tranzakciós hatókörben, kompenzálni a tranzakciót.

**Diagnosztikai**. Használat [tármetrikák][storage-metrics].

### <a name="reading-data-from-azure-storage-fails"></a>Az Azure Storage-ból adatokat olvasó sikertelen lesz.

**Észlelési**. Az ügyfél hibát kap, olvasásakor.

**Helyreállítás:**

1. Próbálja megismételni a műveletet, helyreállítása az átmeneti hibák. A [újrapróbálkozási szabályzat] [ Storage.RetryPolicies] az ügyfél SDK ezt automatikusan megteszi.
2. Az RA-GRS tárolóval Ha az elsődleges végpontból való olvasás sikertelen, próbálja meg a másodlagos végpontból való olvasás. Az ügyfél-SDK segítségével ezt automatikusan kezeli. Lásd: [Azure Storage replikáció][storage-replication].
3. Ha *N* újrapróbálkozási kísérletek sikertelenek, leállás nélküli teljesítménycsökkenés tartalék semmit. Ha például egy termék lemezképet nem lehet beolvasni a storage-ból, ha általános helyőrző képet jeleníthet meg.

**Diagnosztikai**. Használat [tármetrikák][storage-metrics].

## <a name="virtual-machine"></a>Virtuális gép

### <a name="connection-to-a-backend-vm-fails"></a>A háttérrendszernek Virtuálisgép-kapcsolat sikertelen.

**Észlelési**. Hálózati kapcsolati hibák.

**Helyreállítás:**

- Legalább két háttérbeli virtuális gép egy rendelkezésre állási csoportba egy terheléselosztó mögött üzembe helyezése.
- Ha a csatlakozási hiba átmeneti, néha TCP sikeresen újra próbálkozik az üzenetet küldő.
- Újrapróbálkozási szabályzat megvalósítása az alkalmazásban.
- Állandó és nem átmeneti hibák, végrehajtja a [áramkör-megszakító] [ circuit-breaker] mintát.
- Ha a hívó VM meghaladja a hálózati kimenő forgalomra vonatkozó korlát, a kimenő várólista fognak telni. Ha a kimenő várólista következetesen teljes, fontolja meg a horizontális felskálázás.

**Diagnosztikai**. Naplózza a szolgáltatáshatárokon történő eseményeket.

### <a name="vm-instance-becomes-unavailable-or-unhealthy"></a>Virtuálisgép-példány nem érhető el vagy nem megfelelő állapotú lesz.

**Észlelési**. A Load Balancer konfigurálása [állapotadat-mintavétel] [ lb-probe] , amely azt jelzi-e a Virtuálisgép-példány állapota kifogástalan. A mintavétel ellenőrizni kell, hogy a kritikus fontosságú funkciók megfelelően válaszol-e.

**Helyreállítási**. Az egyes alkalmazásrétegek több Virtuálisgép-példányok üzembe ugyanahhoz a rendelkezésre állási csoporthoz, és helyezze el egy terheléselosztót a virtuális gépek elé. Ha az állapotadat-mintavétel meghiúsul, a terheléselosztó nem irányít küld új kapcsolatokat a nem megfelelő állapotú példány.

**Diagnosztikai**. – Használja a Load Balancer [log analytics][lb-monitor].

- A monitorozási rendszer figyelheti az összes figyelési végpontok állapotát konfigurálása.

### <a name="operator-accidentally-shuts-down-a-vm"></a>Operátor véletlenül a virtuális gép leáll.

**Észlelési**. –

**Helyreállítási**. Állítsa be az erőforrás-kezelővel rendelkező `ReadOnly` szintjét. Lásd: [zárolhat erőforrásokat az Azure Resource Manager][rm-locks].

**Diagnosztikai**. Használat [Azure-Tevékenységnaplók][azure-activity-logs].

## <a name="webjobs"></a>WebJobs

### <a name="continuous-job-stops-running-when-the-scm-host-is-idle"></a>Folyamatos feladat nem fut, ha az SCM állomást inaktív.

**Észlelési**. A webjobs-feladat függvény megszakítás tokent átadni. További információkért lásd: [szabályos leállítást][web-jobs-shutdown].

**Helyreállítási**. Engedélyezze a `Always On` beállítása a webalkalmazásban. További információkért lásd: [háttérfeladatok futtatása WebJobs-feladatokkal][web-jobs].

## <a name="application-design"></a>Alkalmazás-tervezés

### <a name="application-cant-handle-a-spike-in-incoming-requests"></a>Alkalmazás nem tudja kezelni a bejövő kérések ugrásszerű.

**Észlelési**. Az alkalmazás függ. Általános jelenség:

- A webhely elindul, HTTP 5xx hibakódok visszaadása.
- Függő szolgáltatások, például az adatbázis vagy tárolócsoport, indítsa el a kérelmek szabályozása. Keresse meg a szolgáltatástól függően HTTP 429 (túl sok kérés), például HTTP-hibák.
- HTTP-várólista hossza nő.

**Helyreállítás:**

- Horizontális felskálázás megnövekedett terhelés kezelése érdekében.
- Megszakítja a teljes alkalmazás hiba egész hibasorozatot indíthat ne kelljen hibák elhárítása érdekében. Kockázatcsökkentési stratégia a következők:

  - Alkalmazzon a [sávszélesség-szabályozási minta] [ throttling-pattern] elsöprő háttérrendszerre elkerülése érdekében.
  - Használat [üzenetsor-alapú terheléskiegyenlítési] [ queue-based-load-leveling] puffereli a kérelmeket, és a egy megfelelő ütemben feldolgozni azokat.
  - Egyes ügyfelek sorrendet. Például ha az alkalmazás ingyenes és fizetős szintek, szabályozás az ingyenes szintet használó ügyfeleknek, de nem fizetős ügyfeleinek. Lásd: [elsőbbségi üzenetsor mintája][priority-queue-pattern].

**Diagnosztikai**. Használat [App Service-ben a diagnosztikai naplózás][app-service-logging]. A szolgáltatás használatához például [Azure Log Analytics][azure-log-analytics], [Application Insights][app-insights], vagy [New relic-bővítménnyel] [ new-relic] a diagnosztikai naplók értelmezése érdekében.

### <a name="one-of-the-operations-in-a-workflow-or-distributed-transaction-fails"></a>A munkafolyamat vagy az elosztott tranzakciók műveletek egyike sikertelen lesz.

**Észlelési**. Miután *N* újra megpróbál, továbbra is sikertelen.

**Helyreállítás:**

- A kezelési terv hajtja végre a [Feladatütemező ügynök felügyeleti] [ scheduler-agent-supervisor] minta kezelheti a teljes munkafolyamatot.
- Az időtúllépések ne próbálkozzon újra. Egy alacsony sikerességi aránya a hiba van.
- Várólista munka, annak érdekében, hogy próbálkozzon újra később.

**Diagnosztikai**. Minden művelet (sikeres és sikertelen), beleértve a kompenzáló műveletek naplózása. Korrelációs azonosítók, használja, így nyomon követheti egy tranzakcióba tartozó összes műveletet.

### <a name="a-call-to-a-remote-service-fails"></a>Egy távoli szolgáltatás hívása sikertelen lesz.

**Észlelési**. HTTP-hibakóddal.

**Helyreállítás:**

1. Ismételje meg az átmeneti hibák.
2. Ha a hívás sikertelen lesz, miután *N* történt kísérlet egy tartalék művelet végrehajtása. (Az alkalmazás adott.)
3. Alkalmazzon a [áramkör-megszakító minta] [ circuit-breaker] egymásra épülő hibák elkerülése érdekében.

**Diagnosztikai**. Jelentkezzen az összes távoli hívás sikertelen.

## <a name="next-steps"></a>További lépések

Az FMA-folyamat kapcsolatos további információkért lásd: [rugalmasság a cloud services elvárt] [ resilience-by-design-pdf] (letölthető PDF-fájl).

<!-- markdownlint-enable MD026 -->

<!-- links -->

[api-management]: /azure/api-management/
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
[cassandra-error-handling]: https://www.datastax.com/dev/blog/cassandra-error-handling-done-right
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
[resilience-by-design-pdf]: https://download.microsoft.com/download/D/8/C/D8C599A4-4E8A-49BF-80EE-FE35F49B914D/Resilience_by_Design_for_Cloud_Services_White_Paper.pdf
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
