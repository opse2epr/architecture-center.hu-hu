---
title: Intelligens termékkereső motor az e-kereskedelem számára
titleSuffix: Azure Example Scenarios
description: Világszínvonalú keresési élményt nyújthat egy e-kereskedelmi alkalmazásban.
author: jelledruyts
ms.date: 09/14/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.custom: fasttrack
ms.openlocfilehash: fe67c891a9e42d5216fe6fd81de6ea1333d5bd37
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54488239"
---
# <a name="intelligent-product-search-engine-for-e-commerce"></a>Intelligens termékkereső motor az e-kereskedelem számára

A példa forgatókönyv bemutatja, hogyan egy dedikált keresési szolgáltatás használatával így jelentősen növelhető a relevancia alapján végzett keresés eredményeit az e-kereskedelmi ügyfelei számára.

Keresés az elsődleges mechanizmusa keresztül, amely ügyfelek keresése és így alapvető fontosságú, hogy a keresési eredmények relevánsak, végső soron vásárolt termékek, a _szándékot_ , a keresési lekérdezést, és hogy a teljes körű felhasználói élményt keresése amelyek keresési eldobást azáltal, hogy közel azonnali eredmények, nyelvi elemzés, megfelelő földrajzi hely, szűrés, értékkorlátozás, automatikus kiegészítés, a találatok kiemelése, stb.

Egy tipikus e-kereskedelmi webalkalmazás Imagine termék adatok egy relációs adatbázisban, például az SQL Server vagy az Azure SQL Database tárolja. Keresési lekérdezések gyakran kezeli az adatbázis parancsával `LIKE` lekérdezések vagy [teljes szöveges keresés] [ docs-sql-fts] funkciókat. Használatával [Azure Search] [ docs-search] ehelyett szabadítsa fel a lekérdezés feldolgozása az operatív adatbázis, és könnyen megkezdheti kihasználhatja ezeket a rögzített megvalósítása funkciókat biztosítanak a a lehetséges legjobb keresése rendelkező ügyfelek tapasztalható. Is mivel az Azure Search egy platformszolgáltatás (PaaS) összetevő platformként, nem kell aggódnia az infrastruktúra felügyeletével vagy egy keresési szakértői váljon.

## <a name="relevant-use-cases"></a>Alkalmazási helyzetek

Egyéb alkalmazási helyzetek a következők:

- Ingatlan listaelemek vagy tárolja a felhasználói fizikai hely közelében található.
- Hírek helyen cikkek keresése, vagy további magasabb prioritást jelent a sportközvetítések eredményeket, keres _legutóbbi_ információkat.
- A nagyméretű adattárak való kereséssel _Dokumentumközpontú_ cégek vagy szervezetek házirend döntéshozók és közjegyzőkkel.

Végső soron _bármely_ keresési funkciók valamilyen alkalmazás egy dedikált keresési szolgáltatás is kihasználhatják.

## <a name="architecture"></a>Architektúra

![Az Azure-összetevőket részt vesz-e-kereskedelmi termék intelligens keresőmotort architektúrájának áttekintése][architecture]

Ebben a forgatókönyvben egy elektronikus kereskedelmi megoldás, ahol a termékkatalógus ügyfelek lehet keresést ismerteti.

1. Váltson ügyfelei a **elektronikus kereskedelmi webalkalmazás** bármilyen eszközről.
2. A termékkatalógus megőrződik a egy **Azure SQL Database** a tranzakciós feldolgozást kínál.
3. Használja az Azure Search egy **keresési indexelő** , automatikusan elkészítheti a search-index naprakész keresztül integrált változáskövetés.
4. Az ügyfél keresési lekérdezések kiszervezett a **Azure Search** szolgáltatás, amely feldolgozza a lekérdezést, és a legrelevánsabb eredményeket ad vissza.
5. Ahelyett, hogy a webes keresési funkciókat, mint a felhasználók is használhatják a **robot természetes nyelvi** a közösségi médiában vagy közvetlenül az digitális asszisztensek keresés a termékek és a növekményes pontosítsa a keresési lekérdezést és az eredményeket.
6. Szükség esetén a **Cognitive Search** szolgáltatás használható a mesterséges intelligencia még intelligensebb feldolgozásra a alkalmazni.

### <a name="components"></a>Összetevők

- [App Services szolgáltatásban – a Web Apps] [ docs-webapps] gazdagépek webes alkalmazások, lehetővé téve az automatikus méretezés és magas rendelkezésre állás nélkül az infrastruktúrával kellene foglalkoznia.
- [Az SQL Database] [ docs-sql-database] van egy általános célú relációs adatbázis által felügyelt szolgáltatás, amely a relációs, JSON-, térbeli és XML-struktúrákat is támogat a Microsoft Azure-ban.
- [Az Azure Search] [ docs-search] egy keresési--szolgáltatásként felhőalapú megoldás, amely fejlett keresési funkciókat személyes tartalmakhoz webes, mobil és vállalati alkalmazások keresztül.
- [Bot Service] [ docs-botservice] elkészítheti, tesztelheti, telepítheti és felügyelheti a robotokat intelligens eszközöket biztosít.
- [A cognitive Services] [ docs-cognitive] lehetővé teszi, hogy intelligens algoritmusok segítségével jelenik meg, hallgassa meg beszél, megértésében, valamint a felhasználói szükségletek értelmezése a kommunikáció természetes módszereivel keresztül.

### <a name="alternatives"></a>Alternatív megoldások

- Használhat **adatbázis-keresés** képességek, például az SQL Server teljes szöveges keresés, de a tranzakciós üzlet keresztül is feldolgozza a lekérdezések (megnőtt az igény a feldolgozási teljesítményt) és a keresési funkciókkal az adatbázisban korlátozott.
- Sikerült üzemelteti a nyílt forráskódú [Apache Lucene] [ apache-lucene] (amely az Azure Search épül) az Azure-beli virtuális gépeken, majd ismét, hogy infrastruktúra--szolgáltatásként (IaaS) kezelése, és nem előnyeit, de az Azure Search Lucene felett biztosít számos funkciót.
- Emellett érdemes lehet telepítése [rugalmas keresés] [ elastic-marketplace] az Azure piactéren, amely egy alternatív és képes keresési termék külső szállítótól származó, de is ebben az esetben futtat egy IaaS számítási feladatok.

Egyéb lehetőségek az adatréteg számára a következők:

- [A cosmos DB](/azure/cosmos-db/introduction) – a Microsoft globálisan elosztott, többmodelles adatbázis. Costmos DB futtatásához például Cassandra, Mongodb, más adatmodellek platformot biztosít a grafikon adatainak, vagy az egyszerű. Az Azure Search is támogatja a Cosmos DB-ből az adatokat közvetlenül az indexelés.

## <a name="considerations"></a>Megfontolandó szempontok

### <a name="scalability"></a>Méretezhetőség

A [tarifacsomag] [ search-tier] az Azure Search szolgáltatás a rendelkezésre álló funkciók nem határozza meg, de segítségével főleg [kapacitástervezés] [ search-capacity]meghatározza a maximális tárolási kap, és hány particionálja és replikákat is kioszthatja. **A partíciók** lehetővé teszik több dokumentumok indexelése és a nagyobb írási termékváltozatokat, mivel a **replikák** további lekérdezések másodpercenkénti lekérdezési (QPS) és a magas rendelkezésre állást biztosít.

Dinamikusan módosíthatja a partíciókat és -replikákat a számát, de nem módosíthatja a tarifacsomagot, akkor alaposan gondolja át a megfelelő szint célfeladathoz. Ha mégis módosítja a réteg van szüksége, szüksége lesz az új szolgáltatást egymás mellett, és töltse be újra az indexek van, ekkor az alkalmazásokat az új szolgáltatás mutathat.

### <a name="availability"></a>Rendelkezésre állás

Biztosít az Azure Search egy [99,9 %-os rendelkezésre állási SLA] [ search-sla] a _beolvassa_ (vagyis a lekérdezés) Ha legalább két replika van, valamint a _frissítések_(azaz frissítése a keresési indexek) Ha van legalább három replika készül. Ezért kell kiépítenie legalább két replika, ha azt szeretné, hogy az ügyfelek tudni _keresési_ megbízhatóan, és a tényleges esetén a 3 _módosítja az index_ figyelembe kell venni magas rendelkezésre állású műveletek .

Ha van szükség, hogy a kompatibilitástörő változásokat az indexhez (például adattípusok módosítása, törlése vagy mezők átnevezése) üzemkimaradás nélkül, az index újraépítését kell. Hasonló szolgáltatási szint módosítása, ez azt jelenti, egy új index létrehozását, elszaporításával, az adatokat és majd frissítése az alkalmazásokat az új indexre mutassanak.

### <a name="security"></a>Biztonsági

Az Azure Search egy megfelelő, a legtöbb [biztonsági és az adatvédelmi szabványok][search-security], amely lehetővé teszi a legtöbb iparágban használható.

A szolgáltatáshoz való hozzáférés védelmét, az Azure Search kulcsok két típusú használ: **rendszergazdai kulcsok**, ez lehetővé teszi annak végrehajtása _bármely_ feladatot a szolgáltatásban, és **lekérdezési kulcsokkal**, amely is csak a csak olvasási műveletek, mint a lekérdezési használni. Az alkalmazás, amely végrehajtja a keresést általában nem frissíti az indexet, azt csak kell konfigurálni a lekérdezési kulcs és a nem rendszergazdai kulcsot (különösen akkor, ha a keresés például böngészőben futó parancsfájl-végfelhasználói eszközről történik).

### <a name="search-relevance"></a>Keresés relevancia alapján végzett

Hogyan sikeres van az e-kereskedelmi alkalmazás nagyban függ az ügyfelek számára a keresési eredmények relevancia alapján. Gondosan hangolása felhasználói kutatási alapján optimális eredményeket a keresési szolgáltatás, vagy a beépített funkciók például támaszkodva [forgalomelemzés keresés] [ search-analysis] tudni, hogy az ügyfél keresési minták lehetővé teszi az adatokon alapuló döntéseket hozhat.

Hallgassa meg a keresési szolgáltatás szokásos módjai a következők:

- Használatával [pontozási profilok] [ search-scoring] befolyásolhatja a relevancia alapján végzett, a keresési eredmények között, például melyik mező egyezik a lekérdezést, milyen friss az adatok van, a felhasználónak a földrajzi távolságtól alapján...
- Használatával [Microsoft megadott nyelvi elemzők] [ search-languages] egy fejlett, természetes nyelvi feldolgozási (NLP) stack használatával, amely lekérdezések jobban értelmezése
- Használatával [egyéni elemzőket] [ search-analyzers] annak biztosítására, a termékek találhatók megfelelően, különösen akkor, ha nem nyelvi a keresett információkat, például egy termék gyártmány és modell alapján.

## <a name="deploy-the-scenario"></a>A forgatókönyv megvalósításához

Ebben a forgatókönyvben egy teljes körű e-kereskedelmi verziójának üzembe helyezéséhez kövesse ezt [részletes oktatóanyag] [ end-to-end-walkthrough] , amely biztosítja, hogy egy alkalmazás egy egyszerű jegyvásárlási futó .NET-mintaalkalmazást. Tartalmazza az Azure Search és a tárgyalt funkciók nagy része használja. Emellett nincs automatizálható az Azure-erőforrások legnagyobb része a Resource Manager-sablonnal.

## <a name="pricing"></a>Díjszabás

Ebben a forgatókönyvben költségének megismeréséhez, minden, a fent említett szolgáltatások előre konfigurálva, a költség díjkalkulátorban. Tekintse meg, hogyan díjszabását szeretné módosítani az adott használatra esetben módosítsa a megfelelő változók a várható használat megfelelően.

Adtunk meg beolvasni a várt forgalom mennyisége alapján három példa költség profilt:

- [Kis][small-pricing]: Ebben a profilban használjuk egy `Standard S1` webalkalmazás üzemeltetéséhez a webhelyén, az Azure Bot service, egy ingyenes szintjéhez `Basic` Azure Search szolgáltatást, és a egy `Standard S2` SQL-adatbázis.
- [Közepes][medium-pricing]: Itt azt a webalkalmazást a két példánya méretezése vannak a `Standard S3` szint, a keresési szolgáltatás frissítése egy `Standard S1` szinthez, majd a használatával egy `Standard S6` SQL-adatbázis.
- [Nagy][large-pricing]: Négy példányát használjuk a legnagyobb profil egy `Premium P2V2` Web App frissítése az Azure Bot service az a `Standard S1` szint (1.000.000 az üzenetek a prémium szintű csatornák), 2 egységek használata a `Standard S3` Azure Search szolgáltatást, és a egy `Premium P6` SQL Az adatbázis.

## <a name="related-resources"></a>Kapcsolódó erőforrások

Azure Search kapcsolatos további információkért látogasson el a [dokumentációs központban][docs-search], tekintse meg a [minták][search-samples], vagy tekintse meg a teljes körű [– bemutató webhely] [ search-demo] működés közben.

<!-- links -->
[architecture]: ./media/architecture-ecommerce-search.png
[docs-sql-fts]: /sql/relational-databases/search/query-with-full-text-search
[docs-search]: /azure/search/search-what-is-azure-search
[docs-sql-database]: /azure/sql-database/sql-database-technical-overview
[docs-webapps]: /azure/app-service/app-service-web-overview
[docs-botservice]: /azure/bot-service/
[docs-cognitive]: /azure/cognitive-services/
[apache-lucene]: https://lucene.apache.org/
[elastic-marketplace]: https://azuremarketplace.microsoft.com/marketplace/apps/elastic.elasticsearch
[end-to-end-walkthrough]: https://github.com/Azure/fta-customerfacingapps/tree/master/ecommerce/articles
[search-sla]: https://go.microsoft.com/fwlink/?LinkId=716855
[search-tier]: /azure/search/search-sku-tier
[search-capacity]: /azure/search/search-capacity-planning
[search-security]: /azure/search/search-security-overview
[search-analysis]: /azure/search/search-traffic-analytics
[search-languages]: /rest/api/searchservice/language-support
[search-analyzers]: /rest/api/searchservice/custom-analyzers-in-azure-search
[search-scoring]: /rest/api/searchservice/add-scoring-profiles-to-a-search-index
[search-samples]: https://azure.microsoft.com/resources/samples/?service=search&sort=0
[search-demo]: https://azjobsdemo.azurewebsites.net/
[small-pricing]: https://azure.com/e/db2672a55b6b4d768ef0060a8d9759bd
[medium-pricing]: https://azure.com/e/a5ad0706c9e74add811e83ef83766a1c
[large-pricing]: https://azure.com/e/57f95a898daa487795bd305599973ee6
