---
title: E-kereskedelmi kezelőfelület
titleSuffix: Azure Example Scenarios
description: E-kereskedelmi webhelyet üzemeltethet az Azure-ban.
author: masonch
ms.date: 07/13/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.custom: fasttrack
social_image_url: /azure/architecture/example-scenario/apps/media/architecture-ecommerce-scenario.png
ms.openlocfilehash: b10ffaafb6479fef89e6691a686c018ac6ce5632
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/17/2019
ms.locfileid: "59640753"
---
# <a name="an-e-commerce-front-end-on-azure"></a>Az Azure-ban egy e-kereskedelmi előtér

A példaforgatókönyv végigvezeti implementálhat egy e-kereskedelmi előtér egy platformszolgáltatás (PaaS) eszközök, Azure-platform használatával. Sok e-kereskedelmi webhelyekről szezonalitás és a forgalom változékonyságát idővel között. Esetén a termékek vagy szolgáltatások iránti igény lép, e kiszámítható módon vagy kiszámíthatatlanul, PaaS-eszközök használatával lehetővé teszi, hogy több vásárló és több tranzakció automatikus kezelésére. Emellett ebben a forgatókönyvben kihasználja a felhő gazdaságosságát szerint kell fizetnie kapacitásért fizet.

Ez a dokumentum segítségével különböző Azure PaaS-összetevőt, és ahhoz, hogy használt szempontokat ismerteti a együtt üzembe helyezünk egy mintaalkalmazást e-kereskedelmi *Relecloud koncertek*, egy online concert jegyeket kezelő platformot.

## <a name="relevant-use-cases"></a>Alkalmazási helyzetek

Egyéb alkalmazási helyzetek a következők:

- Olyan alkalmazást épít ki, amely rugalmas méretezési szolgáltatás a felhasználók kezeléséhez a különböző időpontokban kell rendelkeznie.
- Olyan alkalmazást épít ki, amelyek célja, hogy a világ különböző pontjain különböző Azure-régióban a magas rendelkezésre állású kezeljen.

## <a name="architecture"></a>Architektúra

![Minta forgatókönyv-architektúrát egy e-kereskedelmi alkalmazásban][architecture]

Ebben a forgatókönyvben a következő a vásárlási jegyek egy e-kereskedelmi webhelyről, az adatfolyam-gyűjteményre, a forgatókönyv segítségével mutatja be:

1. Az Azure Traffic Manager egy felhasználói kérelem irányítja az Azure App Service-ben üzemeltetett elektronikus kereskedelmi webhelyen.
2. Az Azure CDN szolgál a felhasználó statikus képeket és a tartalom.
3. Felhasználó jelentkezik be az alkalmazást az Azure Active Directory B2C-bérlő.
4. Felhasználó használja az Azure Search koncertek keres.
5. Webhely concert részletek lekéri az Azure SQL Database-ből.
6. Webhely hivatkozik a megvásárolt jegy lemezképek Blob Storage-ban.
7. Adatbázis-lekérdezés eredményeit az Azure Redis Cache-ben lettek gyorsítótárazva, a jobb teljesítmény érdekében.
8. Felhasználó elküldi a jegyet rendeléseket és concert értékelések, amelyre kerülnek, az üzenetsorban.
9. Azure Functions által feldolgozott order fizetési és concert ellenőrzi.
10. A cognitive services-elemzés a concert tekintse át a vélemény (pozitív vagy negatív) meghatározásához adjon meg.
11. Az Application Insights teljesítménymértékeket biztosít a webalkalmazás állapotát.

### <a name="components"></a>Összetevők

- [Az Azure CDN] [ docs-cdn] statikus, gyorsítótárazott tartalmat továbbít a késés csökkentése érdekében a felhasználók közeli helyről.
- [Az Azure Traffic Manager] [ docs-traffic-manager] szabályozza a különböző Azure-régiókban szolgáltatásvégpontokra érkező felhasználói forgalom elosztása.
- [App Services szolgáltatásban – a Web Apps] [ docs-webapps] gazdagépek webes alkalmazások, lehetővé téve az automatikus méretezés és magas rendelkezésre állás nélkül az infrastruktúrával kellene foglalkoznia.
- [Az Azure Active Directory - B2C] [ docs-b2c] egy identitáskezelő szolgáltatás, amely lehetővé teszi a testreszabás, és hogyan ügyfelek regisztráció, bejelentkezés és az alkalmazások profiljuk kezelését felett van.
- [Tárolási üzenetsorok] [ docs-storage-queues] üzenetsorbeli üzenetek esetén az alkalmazások által elérhető nagy számú tárolja.
- [Függvények] [ docs-functions] kiszolgáló nélküli számítási lehetőség, amelyek lehetővé teszik az alkalmazások igény szerinti futtatása nélkül az infrastruktúrával kellene foglalkoznia.
- [Cognitive Services – Hangulatelemzés] [ docs-sentiment-analysis] használja a machine learning API-kat, és lehetővé teszi a fejlesztők könnyedén adhat alkalmazásaihoz intelligens funkciókat, például érzelem- és video észlelési; videofelismerést, arc, beszédfelismerési és vizuális elismerést; és beszéd- és beszédfelismerés – alkalmazásokba.
- [Az Azure Search] [ docs-search] egy keresési--szolgáltatásként felhőalapú megoldás, amely fejlett keresési funkciókat személyes tartalmakhoz webes, mobil és vállalati alkalmazások keresztül.
- [Storage-blobokat] [ docs-storage-blobs] nagy mennyiségű strukturálatlan adat, például szövegek vagy bináris adatok tárolására vannak optimalizálva.
- [A redis Cache] [ docs-redis-cache] fokozza a teljesítményét és méretezhetőségét, rendszerek, amelyek az erősen támaszkodnak háttér-adattárak rendszer ideiglenesen átmásolja a gyakran használt adatok gyors tárolóba, az alkalmazás közelében helyezkednek el.
- [Az SQL Database] [ docs-sql-database] egy általános célú relációs adatbázis felügyelt szolgáltatás, amely a relációs, JSON-, térbeli és XML-struktúrákat is támogat a Microsoft Azure-ban.
- [Az Application Insights] [ docs-application-insights] célja, hogy folyamatosan javíthassa a teljesítményt és a használhatóságot által automatikusan észlelni a teljesítményanomáliákat keresztül beépített elemzési eszközök segítségével könnyedén áttekinthető a felhasználói tevékenység az alkalmazás.

### <a name="alternatives"></a>Alternatív megoldások

Sok más technológiákkal, amellyel összpontosít az e-kereskedelmi nagy mennyiségű alkalmazás rendelkező ügyfél érhetők el. Ezek a mind az előtér az alkalmazás, valamint az adatréteg terjed ki.

A webes szint és a funkciók más lehetőségek a következők:

- [A Service Fabric] [ docs-service-fabric] -platform, amelynek elsődleges célja elosztott összetevőket, amelyek telepítése és a egy fürtbe, a vezérlő egy magas szintű futtathatja létrehozásához. A Service Fabric is használható tárolók üzemeltetéséhez.
- [Az Azure Kubernetes Service] [ docs-kubernetes-service] – A platform és helyezik üzembe tárolóalapú megoldásokat, amelyek használhatók a mikroszolgáltatási architektúra egy megvalósítását. Ez lehetővé teszi különböző összetevőihez tartozó független méretezését, igény szerinti tudják az alkalmazás rugalmasságát.
- [Az Azure Container Instances] [ docs-container-instances] – gyors üzembe helyezése és a futó tárolók egy rövid életciklusával módon. Itt a tárolók üzembe futtathat például egy üzenet feldolgozása vagy egy számítás elvégzése gyors feldolgozás feladatokat és – figyelmeztetés megszüntetésről, amint ezek teljesülnek.
- [A Service Bus] [ service-bus] helyett az üzenetsor létrehozása sikerült használni.

Egyéb lehetőségek az adatréteg számára a következők:

- [A cosmos DB](/azure/cosmos-db/introduction): A Microsoft globálisan elosztott, többmodelles adatbázis. Ez a szolgáltatás futtatásához például Cassandra, Mongodb, más adatmodellek platformot biztosít a grafikon adatainak, vagy az egyszerű.

## <a name="considerations"></a>Megfontolandó szempontok

### <a name="availability"></a>Rendelkezésre állás

- Vegye figyelembe, kihasználva a [jellemző tervezési minták a rendelkezésre állási] [ design-patterns-availability] a felhőbeli alkalmazások készítése során.
- Tekintse át a rendelkezésre állási szempontok a megfelelő [App Service webalkalmazás referenciaarchitektúrája][app-service-reference-architecture]
- Rendelkezésre állási kapcsolatos további szempontokért tekintse meg a [rendelkezésre állási ellenőrzőlista] [ availability] a az Azure Architecture Centert.

### <a name="scalability"></a>Méretezhetőség

- Ha a cloud application készítése vegye figyelembe a [jellemző tervezési minták a méretezhetőség][design-patterns-scalability].
- Megfontolandó szempontok a megfelelő méretezhetőség [App Service webalkalmazás referenciaarchitektúrája][app-service-reference-architecture]
- Méretezhetőség témaköröket talál a [méretezési ellenőrzőlista] [ scalability] elérhető az Azure Architecture Centert.

### <a name="security"></a>Biztonság

- Vegye figyelembe, kihasználva a [jellemző tervezési minták a biztonsági] [ design-patterns-security] ahol lehetséges.
- Tekintse át a biztonsági szempontok a megfelelő [App Service webalkalmazás referenciaarchitektúrája][app-service-reference-architecture].
- Fontolja meg következő egy [biztonságos fejlesztési életciklus] [ secure-development] folyamat segítségével a fejlesztők és hozhat létre biztonságosabb szoftver címet biztonsági megfelelőségi követelményeknek fejlesztési költségek csökkentése mellett.
- Tervrajz architektúrájának áttekintése [Azure PCI DSS megfelelőségi][pci-dss-blueprint].

### <a name="resiliency"></a>Rugalmasság

- Vegye figyelembe, kihasználva a [áramkör-megszakító minta] [ circuit-breaker] biztosít hibakezelés normális kell az alkalmazás egy része nem érhető el.
- Tekintse át a [jellemző tervezési minták a rugalmassághoz] [ design-patterns-resiliency] , és vegye fontolóra a megfelelő helyen.
- Számos annak [ajánlott eljárások az App Service] [ resiliency-app-service] a az Azure Architecture Centert.
- Érdemes lehet aktív [georeplikációs] [ sql-geo-replication] az adatréteg számára, és [georedundáns] [ storage-geo-redundancy] képekhez és üzenetsorok tárolására.
- A részletesebb leírásáért lásd [rugalmasság][resiliency], az Azure Architecture Centert megfelelő témakörében talál.

## <a name="deploy-the-scenario"></a>A forgatókönyv megvalósításához

Ez a forgatókönyv üzembe helyezéséhez kövesse ezt [részletes oktatóanyag] [ end-to-end-walkthrough] bemutatja, hogyan lehet manuálisan telepíti minden egyes összetevő. Ebben az oktatóanyagban egy alkalmazást egy egyszerű jegyvásárlási futó .NET-mintaalkalmazást is biztosít. Emellett nincs automatizálható az Azure-erőforrások legnagyobb része a Resource Manager-sablonnal.

## <a name="pricing"></a>Díjszabás

Ismerje meg a forgatókönyv futtatásával járó költségeket, a szolgáltatások mindegyike a költségkalkulátor az előre konfigurált. Tekintse meg, hogyan díjszabását szeretné módosítani az adott használatra esetben módosítsa a megfelelő változókat egyezik a várt forgalomhoz.

Adtunk meg beolvasni a várt forgalom mennyisége alapján három példa költség profilt:

- [Kis][small-pricing]: A díjszabási példa az elévült minimális termelési szintű példány létrehozásához szükséges összetevőket jelöli. Itt azt feltételezzük felhasználók, csak a több ezer / hó számozása kis számú. Az alkalmazás egy standard webalkalmazást, amely lesz ahhoz, hogy az automatikus skálázás egyetlen példányát használja. Egyéb összetevők vannak méretezve, hogy egy alapszintű csomag, amelyek lehetővé teszik a költségek minimális mennyisége lesz, de továbbra is győződjön meg arról, hogy nincs-e SLA-t támogatja, és elegendő kapacitás a termelési szintű számítási feladatok.
- [Közepes][medium-pricing]: Ebben a példában díjszabása előzetes központi telepítés ennél kisebb méretű összetevőket jelöli. Itt azt becsülje meg a rendszert használ a hónap folyamán körülbelül 100 000 felhasználót. A várt forgalom az mérsékelt standard csomagot egyetlen alkalmazás szolgáltatáspéldány történik. Ezenkívül mérsékelt rétegből álló cognitive, és keresse meg a díjkalkulátorban feltüntetett szolgáltatással bővül.
- [Nagy][large-pricing]: A díjszabási példa egy alkalmazás nagy adatmennyiség esetén: több millió felhasználó / hó terabájtnyi adatot áthelyezni sorrendje webszolgáltatásokban jelöli. Ezen a szinten használati magas teljesítményt a prémium szint fronted traffic Managerrel több régióban üzembe helyezett webalkalmazások szükség. Adatok a következőket tartalmazza: tárolás, adatbázisok és a CDN, megtörténik az terabájtnyi adatot.

## <a name="related-resources"></a>Kapcsolódó források (lehet, hogy a cikkek angol nyelvűek)

- [Többrégiós webalkalmazás a referencia-architektúra][multi-region-web-app]
- [a tárolók hivatkozás példája eShop][microservices-ecommerce]

<!-- links -->
[architecture]: ./media/architecture-ecommerce-scenario.png
[small-pricing]: https://azure.com/e/90fbb6a661a04888a57322985f9b34ac
[medium-pricing]: https://azure.com/e/38d5d387e3234537b6859660db1c9973
[large-pricing]: https://azure.com/e/f07f99b6c3134803a14c9b43fcba3e2f
[app-service-reference-architecture]: ../../reference-architectures/app-service-web-app/basic-web-app.md
[availability]: /azure/architecture/checklist/availability
[circuit-breaker]: /azure/architecture/patterns/circuit-breaker
[design-patterns-availability]: /azure/architecture/patterns/category/availability
[design-patterns-resiliency]: /azure/architecture/patterns/category/resiliency
[design-patterns-scalability]: /azure/architecture/patterns/category/performance-scalability
[design-patterns-security]: /azure/architecture/patterns/category/security
[docs-application-insights]: /azure/application-insights/app-insights-overview
[docs-b2c]: /azure/active-directory-b2c/active-directory-b2c-overview
[docs-cdn]: /azure/cdn/cdn-overview
[docs-container-instances]: /azure/container-instances/
[docs-kubernetes-service]: /azure/aks/
[docs-functions]: /azure/azure-functions/functions-overview
[docs-redis-cache]: /azure/redis-cache/cache-overview
[docs-search]: /azure/search/search-what-is-azure-search
[docs-service-fabric]: /azure/service-fabric/
[docs-sentiment-analysis]: /azure/cognitive-services/welcome
[docs-sql-database]: /azure/sql-database/sql-database-technical-overview
[docs-storage-blobs]: /azure/storage/blobs/storage-blobs-introduction
[docs-storage-queues]: /azure/storage/queues/storage-queues-introduction
[docs-traffic-manager]: /azure/traffic-manager/traffic-manager-overview
[docs-webapps]: /azure/app-service/app-service-web-overview
[end-to-end-walkthrough]: https://github.com/Azure/fta-customerfacingapps/tree/master/ecommerce/articles
[microservices-ecommerce]: https://github.com/dotnet-architecture/eShopOnContainers
[multi-region-web-app]: /azure/architecture/reference-architectures/app-service-web-app/multi-region
[pci-dss-blueprint]: /azure/security/blueprints/payment-processing-blueprint
[resiliency-app-service]: /azure/architecture/checklist/resiliency-per-service#app-service
[resiliency]: /azure/architecture/checklist/resiliency
[scalability]: /azure/architecture/checklist/scalability
[secure-development]: https://www.microsoft.com/SDL/process/design.aspx
[service-bus]: /azure/service-bus-messaging/
[sql-geo-replication]: /azure/sql-database/sql-database-geo-replication-overview
[storage-geo-redundancy]: /azure/storage/common/storage-redundancy-grs
