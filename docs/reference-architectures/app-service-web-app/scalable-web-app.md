---
title: Méretezhető webalkalmazás
description: A Microsoft Azure-ban futó webalkalmazás méretezhetőségének javítása.
author: MikeWasson
pnp.series.title: Azure App Service
pnp.series.prev: basic-web-app
pnp.series.next: multi-region-web-app
ms.date: 11/23/2016
cardTitle: Improve scalability
ms.openlocfilehash: 6459acebfa25491332e2118b9e8fe51d5fc79ff3
ms.sourcegitcommit: 5d99b195388b7cabba383c49a81390ac48f86e8a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 07/10/2018
ms.locfileid: "37958806"
---
# <a name="improve-scalability-in-a-web-application"></a>Méretezhetőség javítása egy webalkalmazásban

Ez a referenciaarchitektúra az Azure App Service-webalkalmazások méretezhetőségének és teljesítményének javítására szolgáló bevált eljárásokat mutat be.

![[0]][0]

*Töltse le az architektúra [Visio-fájlját][visio-download].*

## <a name="architecture"></a>Architektúra  

Ez az architektúra az [Alapszintű webalkalmazás][basic-web-app] témakörében bemutatott architektúrára épít. A következő összetevőket tartalmazza:

* **Erőforráscsoport**. Az [erőforráscsoport][resource-group] az Azure-erőforrások logikai tárolója.
* **[Webalkalmazás][app-service-web-app]** és **[API-alkalmazás][app-service-api-app]**. A tipikus modern alkalmazás általában tartalmaz egy webhelyet és egy vagy több RESTful webes API-t. A webes API-t a böngészőügyfelek használják az AJAX-on keresztül, valamint a natív ügyfélalkalmazások vagy a kiszolgálóoldali alkalmazások. A webes API-k tervezésének szempontjait lásd: [API-tervezési segédlet][api-guidance].    
* **WebJob**. Az [Azure WebJobs][webjobs] használatával hosszan futó feladatokat futtathat a háttérben. A WebJob-feladat futtatható ütemezés szerint, folyamatosan vagy egy olyan eseményindító hatására, amilyen például egy üzenetet elhelyezése az üzenetsorban. A WebJob-feladat háttérfolyamatként fut az App Service-alkalmazások környezetében.
* **Üzenetsor**. Az itt látható architektúrában az alkalmazás úgy teszi várakozási sorba a feladatokat, hogy üzenetet helyez el egy [Azure Queue Storage][queue-storage]-üzenetsorban. Az üzenet aktivál egy függvényt a WebJob-feladatban. Választhatja a Service Bus-üzenetsorok használatát is. A két lehetőség összehasonlítását lásd: [Azure-üzenetsorok és Service Bus-üzenetsorok összehasonlítása][queues-compared].
* **Gyorsítótár**. A félig statikus adatok tárolására az [Azure Redis Cache][azure-redis] használható.  
* <strong>CDN</strong>. Az [Azure Content Delivery Network][azure-cdn] (CDN) a nyilvánosan elérhető tartalmak gyorsítótárazására használható a kisebb késés és a gyorsabb tartalomkézbesítés biztosítása érdekében.
* **Adattárolás**. A relációs adatok esetében az [Azure SQL Database][sql-db] használata javasolt. A nem relációs adatokhoz érdemes fontolóra venni egy olyan NoSQL-tároló alkalmazását, amilyen például a [Cosmos DB][cosmosdb].
* **Azure Search**. Az [Azure Search][azure-search] használatával olyan keresési funkciók adhatók hozzá, mint például a keresési javaslatok, az intelligens keresés és a nyelvspecifikus keresés. Az Azure Search általában egy másik adattárral együtt használatos, különösen akkor, ha az elsődleges adattár megköveteli a szigorú konzisztenciát. Ennek a megközelítésnek megfelelően a mérvadó adatokat a másik adattárban kell tárolni, a keresési indexet pedig az Azure Search-ben. Az Azure Search továbbá egyetlen, különböző adattárakból összeállított keresési index létrehozására is használható.  
* **E-mail/SMS**. Az e-mailek és SMS-üzenetek küldése esetében érdemes egy külső szolgáltatást használni, amilyen például a SendGrid vagy a Twilio, semmint közvetlenül az alkalmazásba beépíteni ezt a funkciót.
* **Azure DNS**. Az [Azure DNS][azure-dns] egy üzemeltetési szolgáltatás, amely a Microsoft Azure infrastruktúráját használja a DNS-tartományok névfeloldásához. Ha tartományait az Azure-ban üzemelteti, DNS-rekordjait a többi Azure-szolgáltatáshoz is használt hitelesítő adatokkal, API-kkal, eszközökkel és számlázási információkkal kezelheti.

## <a name="recommendations"></a>Javaslatok

Az Ön követelményei eltérhetnek az itt leírt architektúrától. A jelen szakaszban leírt javaslatokat tekintse kiindulópontnak.

### <a name="app-service-apps"></a>App Service-alkalmazások
A webalkalmazást és a webes API-t külön App Service-alkalmazásokként javasolt létrehozni. Ez a kialakítás lehetővé teszi, hogy külön App Service-csomagokban futhassanak, így egymástól függetlenül lehet méretezni őket. Ha kezdetben nincs szükség ilyen szintű méretezhetőségre, az alkalmazások ugyanabba a csomagba is telepíthetők. Szükség esetén később is szét lehet választani őket.

> [!NOTE]
> Az Alapszintű, a Standard és a Prémium csomagok esetében a csomagban szereplő virtuális gépek példányai után kell fizetni, és nem alkalmazásonként. Lásd: [Az App Service árképzése][app-service-pricing]
> 
> 

Ha használni kívánja az App Service Mobile Apps *Easy Tables* vagy *Easy APIs* funkcióit, külön App Service-alkalmazást kell létrehoznia erre a célra.  Ezen funkciók működéséhez egy speciális alkalmazás-keretrendszer szükséges.

### <a name="webjobs"></a>WebJobs
Az erőforrás-igényes WebJob-feladatokat érdemes egy külön App Service-csomag üres App Service-alkalmazásába telepíteni. Ez dedikált példányokat biztosít a WebJob-feladathoz. Lásd: [Útmutató a háttérfeladatokhoz][webjobs-guidance].  

### <a name="cache"></a>Gyorsítótár
A teljesítmény és a méretezhetőség növelhető azzal, ha bizonyos adatok gyorsítótárazását az [Azure Redis Cache][azure-redis] végzi. A következő esetekben érdemes megfontolni a Redis Cache használatát:

* Félig statikus tranzakciós adatok.
* Munkamenet-állapot.
* HTML-kimenet. Ez az összetett HTML-kimenetet megjelenítő alkalmazások esetében lehet hasznos.

A gyorsítótárazási stratégia tervezésével kapcsolatos részletes útmutatót lásd: [Gyorsítótárazási útmutató][caching-guidance].

### <a name="cdn"></a>Tartalomkézbesítési hálózat (CDN)
Az [Azure CDN] [ azure-cdn] a statikus tartalmak gyorsítótárazására használható. A CDN fő előnye, hogy csökkenti a felhasználói oldalon érzékelhető késést, mivel a tartalom gyorsítótárazása olyan peremhálózati kiszolgálón történik, amely földrajzilag a felhasználó közelében található. A CDN emellett az alkalmazás terhelését is csökkenti, mivel a forgalmat nem az alkalmazás kezeli.

Ha az alkalmazás főként statikus oldalakat tartalmaz, érdemes lehet [a CDN-t használni a teljes alkalmazás gyorsítótárazásához][cdn-app-service]. Egyéb esetben helyezze a statikus tartalmakat, például a képeket, illetve a CSS- és HTML-fájlokat az [Azure Storage-ba, a gyorsítótárazásukat pedig a CDN használatával végezze][cdn-storage-account].

> [!NOTE]
> Az Azure CDN nem szolgáltathat olyan tartalmat, amelyhez hitelesítés szükséges.
> 
> 

A részletes útmutatásért lásd: [Content Delivery Network (CDN) útmutató][cdn-guidance].

### <a name="storage"></a>Storage
A modern alkalmazások gyakran nagy mennyiségű adatot dolgoznak fel. A felhőnek megfelelő méretezés érdekében fontos a megfelelő tárolótípus kiválasztása. Íme néhány alapvető javaslat. 

| Amit tárolni szeretne | Példa | Ajánlott tároló |
| --- | --- | --- |
| Fájlok |Képek, dokumentumok, PDF-fájlok |Azure Blob Storage |
| Kulcs-érték párok |Felhasználói azonosító által keresett felhasználói profiladatok |Azure Table Storage |
| Rövid üzenetek a további feldolgozás aktiválásához |Rendelési kérelmek |Azure Queue Storage, Service Bus-üzenetsor vagy Service Bus-témakör |
| Alapszintű lekérdezést igénylő, nem relációs adatok rugalmas sémával |Termékkatalógus |Dokumentum-adatbázis, például Azure Cosmos DB, MongoDB vagy Apache CouchDB |
| Részletesebb lekérdezés támogatását, szigorú sémát és/vagy nagy mértékű következetességet igénylő relációs adatok |Termékleltár |Azure SQL Database |

## <a name="scalability-considerations"></a>Méretezési szempontok

Az Azure App Service egyik fő előnye az alkalmazások skálázásának lehetősége a terhelés alapján. A következő szempontokat kell szem előtt tartani az alkalmazás skálázásának tervezésekor.

### <a name="app-service-app"></a>App Service-alkalmazás
Ha a megoldás több App Service-alkalmazást tartalmaz, érdemes megfontolni a külön App Service-csomagokba történő telepítésüket. Ez a megoldás lehetővé teszi az egymástól függetlenül történő méretezésüket, mivel külön példányokon futnak. 

Hasonlóképpen hasznos lehet egy WebJob-feladatot a saját csomagjában elhelyezni, így a háttérfeladatok nem azokon a példányokon futnak, amelyek a HTTP-kérelmeket kezelik.  

### <a name="sql-database"></a>SQL Database
A *horizontális skálázás* alkalmazásával növelhető az SQL-adatbázis méretezhetősége. A horizontális skálázás az adatbázis horizontális particionálását jelenti. A horizontális skálázás lehetővé teszi az adatbázis [Elastic Database-eszközökkel][sql-elastic] történő horizontális felskálázását. A horizontális skálázás lehetséges előnyei a következők:

- Nagyobb tranzakciós átviteli sebesség.
- A lekérdezések gyorsabban futnak az adatok egy részén.

### <a name="azure-search"></a>Azure Search
Az Azure Search eltávolítja az összetett adatkeresések okozta többletterhelést az elsődleges adattárolóból, továbbá a terhelésnek megfelelően méretezhető. Lásd: [Az erőforrásszintek méretezése a lekérdezéshez és a számítási feladatok indexeléséhez az Azure Search szolgáltatásban][azure-search-scaling].

## <a name="security-considerations"></a>Biztonsági szempontok
Ez a szakasz a cikkben leírt Azure-szolgáltatásokra vonatkozó biztonsági szempontokat ismerteti. Nem tartalmazza az összes ajánlott biztonsági eljárást. A további biztonsági szempontokért lásd: [Alkalmazás védelme az Azure App Service-ben][app-service-security].

### <a name="cross-origin-resource-sharing-cors"></a>Eltérő eredetű erőforrások megosztása (CORS)
Ha külön alkalmazásként hoz létre egy webhelyet és egy webes API-t, a webhely csak akkor hajthat végre ügyféloldali AJAX-hívásokat az API felé, ha a CORS engedélyezve van.

> [!NOTE]
> A böngésző biztonsági beállításai megakadályozzák, hogy egy weblap AJAX-kérelmeket küldjön egy másik tartományba. Ennek a korlátozásnak a neve azonos eredethez kapcsolódó szabályzat, és annak a megakadályozására szolgál, hogy egy rosszindulatú webhely érzékeny adatokat olvashasson be egy másik webhelyről. A CORS egy W3C szabvány, amely lehetővé teszi a kiszolgáló számára az azonos eredethez kapcsolódó szabályzat feloldását és néhány eltérő eredetű kérelem engedélyezését, más kérések elutasítása mellett.
> 
> 

Az App Services beépített támogatást nyújt a CORS használatához anélkül, hogy alkalmazáskódot kellene írni. Lásd: [API-alkalmazások felhasználása JavaScriptből a CORS használatával][cors]. A webhelyet fel kell venni az API által engedélyezett származási helyek listájára.

### <a name="sql-database-encryption"></a>SQL Database-titkosítás
A [transzparens adattitkosítás][sql-encryption] használatával titkosíthatók az inaktív adatok az adatbázisban. Ez a szolgáltatás valós időben végzi a titkosítást és a visszafejtést a teljes adatbázisra vonatkozóan (a biztonsági mentéseket és a tranzakciós naplófájlokat is beleértve), miközben az alkalmazást nem kell módosítani. A titkosítás némi késéssel jár, ezért a védelmet igénylő adatokat érdemes elkülöníteni egy saját adatbázisba, hogy csak ezen kelljen engedélyezni a titkosítást.  
  

<!-- links -->

[api-guidance]: ../../best-practices/api-design.md
[app-service-security]: /azure/app-service-web/web-sites-security
[app-service-web-app]: /azure/app-service-web/app-service-web-overview
[app-service-api-app]: /azure/app-service-api/app-service-api-apps-why-best-platform
[app-service-pricing]: https://azure.microsoft.com/pricing/details/app-service/
[azure-cdn]: https://azure.microsoft.com/services/cdn/
[azure-dns]: /azure/dns/dns-overview
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
[cosmosdb]: /azure/cosmos-db/
[queue-storage]: /azure/storage/storage-dotnet-how-to-use-queues
[queues-compared]: /azure/service-bus-messaging/service-bus-azure-and-service-bus-queues-compared-contrasted
[resource-group]: /azure/azure-resource-manager/resource-group-overview#resource-groups
[sql-db]: https://azure.microsoft.com/documentation/services/sql-database/
[sql-elastic]: /azure/sql-database/sql-database-elastic-scale-introduction
[sql-encryption]: https://msdn.microsoft.com/library/dn948096.aspx
[tm]: https://azure.microsoft.com/services/traffic-manager/
[visio-download]: https://archcenter.blob.core.windows.net/cdn/app-service-reference-architectures.vsdx
[web-app-multi-region]: ./multi-region.md
[webjobs-guidance]: ../../best-practices/background-jobs.md
[webjobs]: /azure/app-service/app-service-webjobs-readme
[0]: ./images/scalable-web-app.png "Azure-beli webalkalmazás továbbfejlesztett méretezhetőséggel"
