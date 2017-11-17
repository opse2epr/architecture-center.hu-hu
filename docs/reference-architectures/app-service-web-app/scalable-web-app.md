---
title: "Méretezhető webalkalmazás"
description: "A Microsoft Azure-ban futó webalkalmazás méretezhetőség javítása."
author: MikeWasson
pnp.series.title: Azure App Service
pnp.series.prev: basic-web-app
pnp.series.next: multi-region-web-app
ms.date: 11/23/2016
cardTitle: Improve scalability
ms.openlocfilehash: b875b89b87edd5636d90da8b7f8211f965b39937
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="improve-scalability-in-a-web-application"></a>A webalkalmazás a méretezhetőség javítása

A referencia-architektúrában bevált eljárások a méretezhetőség és teljesítmény az Azure App Service-webalkalmazás fejlesztése jeleníti meg.

![[0]][0]

*Töltse le a [Visio fájl] [ visio-download] ezen architektúra.*

## <a name="architecture"></a>Architektúra  

Ez az architektúra látható egy épít [alapvető webalkalmazás][basic-web-app]. A következő összetevőket tartalmazza:

* **Erőforráscsoport**. A [erőforráscsoport] [ resource-group] Azure-erőforrások logikai tárolója.
* **[A webalkalmazás] [ app-service-web-app]**  és  **[API-alkalmazás][app-service-api-app]**. A modern alkalmazások is egy webhelyet, és egy vagy több RESTful webes API-k vehetők igénybe. A webes API keresztül AJAX azoknál a böngészőügyfeleknél, natív ügyfélalkalmazások, vagy kiszolgálóoldali alkalmazások előfordulhat, hogy használni. Szempontok a designing webes API-k, lásd: [API tervezési útmutató][api-guidance].    
* **Webjobs-feladat**. Használjon [Azure WebJobs] [ webjobs] hosszan futó feladatokat futtatni a háttérben. Webjobs-feladatok ütemezés szerint, változásáig, vagy egy eseményindítót, például egy üzenetet a várólistában üzembe válaszul futtathatja. Webjobs-feladat az App Service alkalmazás környezetében háttérfolyamatként fut.
* **Várólista**. Az architektúra látható itt, az alkalmazás várólisták feladatok háttérben által egy üzenet, amely egy [Azure Queue storage] [ queue-storage] várólista. Az üzenet elindítja a webjobs-feladat egy függvényt. Azt is megteheti Service Bus-üzenetsorok is használhatja. Lásd: [Azure várólisták és a Service Bus üzenetsorok - képest és ellentétben][queues-compared].
* **Gyorsítótár**. A félig statikus adatok tárolásához [Azure Redis Cache][azure-redis].  
* **CDN**. Használjon [Azure tartalom Delivery Network] [ azure-cdn] (CDN) kisebb késést biztosít, és a tartalom gyorsabb kézbesítési nyilvánosan elérhető tartalmak gyorsítótárazására való.
* **Adattárolás**. Használjon [Azure SQL Database] [ sql-db] relációs adatok. A nem relációs adatokhoz, fontolja meg egy NoSQL-tároló, például [Cosmos DB][documentdb].
* **Az Azure Search**. Használjon [Azure Search] [ azure-search] hozzá például a keresési javaslatok, az intelligens egyeztetésű keresési és a nyelvspecifikus keresési keresési funkciót. Az Azure Search általában egy másik adattár együtt használatos, különösen akkor, ha az elsődleges adattár megköveteli a szigorú konzisztencia. Ez a megközelítés tároljuk mérvadó adatok szerepel az adattárban és az Azure Search search-index. Az Azure Search vonják össze egy egyetlen search-index több adat áruházakból is használható.  
* **E-mailek/SMS**. Például a SendGrid vagy Twilio egy külső szolgáltatás segítségével küldjön e-mailek vagy SMS-üzenetek helyett ezt a funkciót felépítése közvetlenül az alkalmazásba.

## <a name="recommendations"></a>Javaslatok

A követelmények eltérhetnek az itt leírt architektúra. A javaslatok használja ebben a szakaszban kiindulási pontként.

### <a name="app-service-apps"></a>App Service-alkalmazások
Azt javasoljuk, hogy a webalkalmazás és a webes API létrehozása, külön App Service-alkalmazásokhoz. Ez a kialakítás teszi futtathatók az önálló App Service-csomagokról, így azokat egymástól függetlenül lehet méretezni. Ha kezdetben méretezhetőség, amely szintnek nincs szüksége, az alkalmazások üzembe helyezés a csomagot, és helyezze azokat különálló tervek az később szükség esetén.

> [!NOTE]
> A Basic, Standard és Premium terveket a terv nem alkalmazásonkénti a Virtuálisgép-példányok kell fizetni. Lásd: [App Service díjszabás][app-service-pricing]
> 
> 

Ha használni kívánja a *könnyen táblák* vagy *egyszerű API-k* App Service Mobile Apps, a szolgáltatások egy külön App Service-alkalmazás létrehozása az erre a célra.  Ezek a funkciók és lehetővé teszi egy adott alkalmazás-keretrendszer támaszkodnak.

### <a name="webjobs"></a>WebJobs
Érdemes megfontolni a erőforrás intenzív WebJobs belül egy külön alkalmazásszolgáltatás üres App Service-alkalmazásokhoz tervezéséhez. Külön példányt biztosít a webjobs-feladat. Lásd: [feladatok útmutatást háttérben][webjobs-guidance].  

### <a name="cache"></a>Gyorsítótár
Teljesítmény és méretezhetőség növelheti az használatával [Azure Redis Cache] [ azure-redis] egyes adatok gyorsítótárazásához. Vegye figyelembe, hogy a Redis Cache segítségével:

* A tranzakció félig statikus adatok.
* Munkamenet-állapot.
* HTML-kimenetében. Ez lehet hasznos, amely összetett HTML-kimenetében leképezési alkalmazások.

Részletes útmutatást a gyorsítótárazási stratégia tervezése, lásd: [útmutatást gyorsítótárazás][caching-guidance].

### <a name="cdn"></a>Tartalomkézbesítési hálózat (CDN)
Használjon [Azure CDN] [ azure-cdn] statikus tartalmak gyorsítótárazására való. A fő előnye, hogy a CDN oka a felhasználók számára, a késés csökkentése érdekében tartalom rendszer gyorsítótárba helyezte egy biztonsági kiszolgálót, amely földrajzilag megközelíti a felhasználó. CDN is csökkentheti terhelése az alkalmazást, mert van az alkalmazás által nem kezelt, hogy a forgalom.

Ha az alkalmazás főként a statikus lapokat tartalmaz, érdemes lehet [a teljes alkalmazás gyorsítótárazza a CDN][cdn-app-service]. Ellenkező esetben helyezze a statikus tartalmak, képek, CSS, például HTML-fájlokat, és a [Azure Storage és a CDN használata gyorsítótárazza azokat a fájlokat][cdn-storage-account].

> [!NOTE]
> Az Azure CDN nem tartalomszolgáltatás, amelyhez hitelesítés szükséges.
> 
> 

Részletes útmutatást, lásd: [Content Delivery Network (CDN) útmutatást][cdn-guidance].

### <a name="storage"></a>Storage
Modern alkalmazások gyakran nagy mennyiségű adat feldolgozására. Ahhoz, hogy méretezni a felhőben, fontos válassza ki a megfelelő tárolási típusát. Az alábbiakban néhány alapterv javaslatokat. 

| Szeretné tárolni | Példa | Ajánlott tárolási |
| --- | --- | --- |
| Fájlok |Képek, dokumentumok, PDF-fájlok |Azure Blob Storage |
| Kulcs/érték párok |Felhasználói azonosító által keresett felhasználói profil adatai |Azure Table Storage |
| Rövid üzenetek kívánják indul el, további feldolgozás |Rendelés kérelmek |Az Azure Queue storage, Service Bus-üzenetsorba, vagy a Service Bus-témakörbe |
| Egy rugalmas séma igénylő alapvető lekérdezése nem relációs adatok |Termékkatalógus |A dokumentum-adatbázisban, például az Azure Cosmos DB, MongoDB vagy Apache CouchDB |
| Gazdagabb lekérdezések támogatását, szigorú séma, és/vagy erős konzisztenciát igénylő relációs adatok |Termék |Azure SQL Database |

## <a name="scalability-considerations"></a>Méretezési szempontok

Egy Azure App Service fő előnye az alkalmazás terhelés alapján méretezési képességét. Az alábbiakban néhány szempontot szem előtt tartani az alkalmazás horizontális tervezése során.

### <a name="app-service-app"></a>App Service alkalmazás
Ha a megoldás több App Service-alkalmazást tartalmaz, érdemes megfontolni a App Service-csomagokról külön őket. Ez a megközelítés lehetővé teszi, hogy egymástól függetlenül méretezheti őket, mert a példány külön számítógépen futnak. 

Hasonlóképpen fontolja meg webjobs-feladat üzembe a saját terv, így háttérfeladatok ne futtassa a példányokon HTTP-kérelmeket kezelnek.  

### <a name="sql-database"></a>SQL Database
SQL-adatbázis által méretezhetőség fokozása *horizontális* az adatbázisban. Az adatbázis particionálási vízszintesen horizontális hivatkozik. Horizontális lehetővé teszi az adatbázishoz vízszintesen kiterjesztése [skálázáshoz rugalmas adatbáziseszközöket][sql-elastic]. Horizontális lehetséges előnyei a következők:

- Tranzakció nagyobb átviteli sebesség.
- Lekérdezések gyorsabban megjelenítheti az adatok egy részét.

### <a name="azure-search"></a>Azure Search
Az Azure Search eltávolítja a terhelést növelni az összetett adatokat keresések végrehajtása az elsődleges adattárolóból, és azt a terhelés kezelésére méretezhetők. Lásd: [erőforrás szintjeinek lekérdezési és indexelési munkaterhelések az Azure Search méretezése][azure-search-scaling].

## <a name="security-considerations"></a>Biztonsági szempontok
Ez a szakasz ebben a cikkben leírt Azure-szolgáltatásokhoz való vonatkozó biztonsági szempontokat sorolja fel. Ajánlott biztonsági eljárások teljes listáját nincs. További biztonsági szempontokat lásd: [az Azure App Service alkalmazás biztonságos][app-service-security].

### <a name="cross-origin-resource-sharing-cors"></a>Eltérő eredetű erőforrások megosztása (CORS)
Ha létrehoz egy webhely és a webes API-t, a különálló alkalmazás, a webhely nem hajtható végre az ügyféloldali AJAX-hívások az API-hoz, kivéve, ha engedélyezi a CORS.

> [!NOTE]
> Böngésző biztonsági megakadályozza, hogy egy weblap AJAX-kérelmek egy másik tartományra. Ez a korlátozás az azonos eredetű házirend nevezik, és megakadályozza, hogy egy rosszindulatú hely sentitive adatolvasás a másik helyről. A CORS W3C szabvány, amely lehetővé teszi a kiszolgáló enyhíteni az azonos eredetű házirend, és néhány eltérő eredetű kérések engedélyezése közben elutasítása mások.
> 
> 

App Service szolgáltatások beépített támogatása CORS, anélkül, hogy az alkalmazás kód írása rendelkezik. Lásd: [javascriptből CORS használatával API-alkalmazások felhasználása][cors]. A webhely hozzáadása az engedélyezett eredetet listája az API-hoz.

### <a name="sql-database-encryption"></a>SQL adatbázis-titkosítás
Használjon [átlátható adattitkosítási] [ sql-encryption] Ha az adatbázis inaktív adatok titkosítása van szüksége. Ez a szolgáltatás hajtja végre a valós idejű titkosítása és visszafejtése egy teljes adatbázisra (egyebek között a biztonsági mentések és a tranzakciós naplófájlok), és nem kell módosítani az alkalmazáshoz. Titkosítási hozzáadása némi késés, ezért ajánlott külön kell biztonságossá tételével kapcsolatosak, a saját adatbázisába, és engedélyezheti a titkosítást csak az, hogy az adatbázis adatai.  
  

<!-- links -->

[api-guidance]: ../../best-practices/api-design.md
[app-service-security]: /azure/app-service-web/web-sites-security
[app-service-web-app]: /azure/app-service-web/app-service-web-overview
[app-service-api-app]: /azure/app-service-api/app-service-api-apps-why-best-platform
[app-service-pricing]: https://azure.microsoft.com/pricing/details/app-service/
[azure-cdn]: https://azure.microsoft.com/services/cdn/
[azure-redis]: https://azure.microsoft.com/services/cache/
[azure-search]: https://azure.microsoft.com/documentation/services/search/
[azure-search-scaling]: /azure/search/search-capacity-planning
[background-jobs]: ../../best-practices/background-jobs.md
[basic-web-app]: basic-web-app.md
[basic-web-app-scalability]: basic-web-app.md#scalability-considerations
[caching-guidance]: ../../best-practices/caching.md
[cdn-app-service]: /azure/app-service-web/cdn-websites-with-cdn
[cdn-storage-account]: /azure/cdn/cdn-create-a-storage-account-with-cdn
[cdn-guidance]: ../../best-practices/cdn.md
[cors]: /azure/app-service-api/app-service-api-cors-consume-javascript
[documentdb]: https://azure.microsoft.com/documentation/services/documentdb/
[queue-storage]: /azure/storage/storage-dotnet-how-to-use-queues
[queues-compared]: /azure/service-bus-messaging/service-bus-azure-and-service-bus-queues-compared-contrasted
[resource-group]: /azure/azure-resource-manager/resource-group-overview#resource-groups
[sql-db]: https://azure.microsoft.com/documentation/services/sql-database/
[sql-elastic]: /azure/sql-database/sql-database-elastic-scale-introduction
[sql-encryption]: https://msdn.microsoft.com/library/dn948096.aspx
[tm]: https://azure.microsoft.com/services/traffic-manager/
[visio-download]: https://archcenter.azureedge.net/cdn/app-service-reference-architectures.vsdx
[web-app-multi-region]: ./multi-region.md
[webjobs-guidance]: ../../best-practices/background-jobs.md
[webjobs]: /azure/app-service/app-service-webjobs-readme
[0]: ./images/scalable-web-app.png "Webalkalmazás az Azure fejlett méretezhetőséget a"
