---
title: Adattárházak és elemzések értékesítési és marketingterületen
description: Egyesítheti több forrásból származó adatait, és optimalizálhatja az adatelemzést.
author: alexbuckgit
ms.date: 09/15/2018
ms.openlocfilehash: e4c0a37f61f3edfb1f29d26df546f02d31fd40f7
ms.sourcegitcommit: 0a31fad9b68d54e2858314ca5fe6cba6c6b95ae4
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/13/2018
ms.locfileid: "51610702"
---
# <a name="data-warehousing-and-analytics-for-sales-and-marketing"></a>Adattárházak és elemzések értékesítési és marketingterületen

A példaforgatókönyv azt szemlélteti, hogy nagy mennyiségű, több forrásból származó adatok integrálása egy egységes elemzési platform az Azure-ban. Ebben a forgatókönyvben egy értékesítési és marketingmegoldásainak alapul, de a kialakítási minták számos iparágban a nagyméretű adathalmazok, például az e-kereskedelmi, kereskedelmi és egészségügyi fejlett analitikai igénylő szempontjából releváns.

Ez a példa bemutatja egy értékesítési és marketing cég, amely létrehozza az ösztönző programok. Ezeket a programokat díjazza ügyfelek, a szállítók, a értékesítők és a munkatársak számára. Adatok esetében alapvető fontosságú, ezeket a programokat, és javíthatja az üzletvitelt az Azure data-analitika segítségével a vállalat úgy kívánja.

A vállalat kell elemzési adatokat, modern megközelítését, így döntés a megfelelő időben a megfelelő adatok használatával. A vállalati célok a következők:
* Különböző típusú olyan adatforrások kombinálásával felhőméretű platformban.
* Forrás adatok átalakítása egy közös besorolás és struktúra, hogy az adatok egységes és egyszerű képest.
* Több ezer, az ösztönző programok, a magas költségeknek, üzembe helyezése és karbantartása a helyszíni infrastruktúra nélkül is támogató nagymértékben párhuzamos módszerével adatok betöltéséhez.
* Jelentősen csökkenti a gyűjtő és az adatok átalakításához szükséges idő, így összpontosíthat az adatok elemzéséhez.

## <a name="relevant-use-cases"></a>Alkalmazási helyzetek

Ez a módszer is használható:

* A data warehouse-adattárházat egy egyetlen hiteles az adatok forrásaként kell létesíteni.
* Relációs adatforrások integrálható más strukturálatlan adatkészletek.
* Egyszerűbb adatelemzés Szemantikus modellezési és hatékony vizualizációs eszközök használja.

## <a name="architecture"></a>Architektúra

![Az Azure data warehousing és elemzési esetén architektúra][architecture]

Az adatok a következőképpen folyamatok a megoldáson keresztül:

1. Mindegyik adatforrás esetében a frissítések exportált rendszeres időközönként egy átmeneti területre az Azure Blob storage-bA.
2. A Data Factory növekményes módon betölti az adatokat Blob storage-ból előkészítési táblák az SQL Data Warehouse. Az adatok tisztított és átalakított a folyamat során. A Polybase párhuzamosítható a nagy adatkészletek esetében a folyamatot.
3. Új adatok kötegelt betöltése a warehouse-ba, miután egy korábban létrehozott Analysis Servicesbeli táblázatos modellhez frissülnek. Ebben a szemantikai modellben egyszerűbbé teszi a vállalati adatok és kapcsolatok elemzése.
4. Üzleti elemzők a Microsoft Power BI segítségével elemezheti az Analysis Services Szemantikus modell raktározott adatokat.

### <a name="components"></a>Összetevők

A vállalati adatforrások számos különböző platformokon rendelkezik:
* A helyszíni SQL Server
* Helyszíni Oracle
* Azure SQL Database
* Azure Table Storage
* Cosmos DB

Ezek segítségével számos Azure-összetevők különböző adatforrásokból származó adatok betöltése:
* [A BLOB storage-](/azure/storage/blobs/storage-blobs-introduction) előtt az SQL Data Warehouse-bA be van töltve, adatok készíthetők elő forrás lesz használva.
* [A Data Factory](/azure/data-factory) koordinálja az előkészített adatok átalakítása a leggyakoribb struktúra az SQL Data Warehouse-bA. A Data Factory [amikor az adatok betöltése az SQL Data Warehouse-bA a polybase](/azure/data-factory/connector-azure-sql-data-warehouse#use-polybase-to-load-data-into-azure-sql-data-warehouse) átviteli sebesség maximalizálása érdekében. 
* [Az SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is) egy elosztott rendszer tárolására és elemzésére a nagyméretű adathalmazok. A masszív párhuzamos feldolgozási (MPP) is teszi megfelelő a futó, nagy teljesítményű elemzési. Az SQL Data Warehouse segítségével [PolyBase](/sql/relational-databases/polybase/polybase-guide) gyorsan betölteni az adatokat Blob storage-ból.
* [Analysis Services](/azure/analysis-services) szemantikai modellt biztosít az adatok számára. A rendszer teljesítménye is azt növekedhet, ha az adatok elemzésének. 
* [Power bi-ban](/power-bi) adatok elemzése és elemzéseket oszthat meg üzleti elemzési eszközök együttese. Power BI lekérdezhesse az Analysis Servicesben tárolt szemantikai modellt, vagy közvetlenül lekérdezheti azt az SQL Data warehouse-bA.
* [Az Azure Active Directory (Azure AD)](/azure/active-directory) hitelesíti a felhasználókat, akik Power bi-ban az Analysis Services-kiszolgálóhoz csatlakozhat. A Data Factory használatával is az Azure AD egyszerű szolgáltatás-on keresztül hitelesítendő az SQL Data Warehouse vagy [-identitás az Azure-erőforrások](/azure/active-directory/managed-identities-azure-resources/overview).

### <a name="alternatives"></a>Alternatív megoldások

* A példa folyamat számos különböző típusú adatforrásokat tartalmaz. Ez az architektúra számos különböző relációs és nem relációs adatforrások képes kezelni.
* A Data Factory hangolja össze az folyamatot a munkafolyamatokat. Ha szeretné betölteni az adatokat csak egyszer, vagy igény szerinti, használhatja az SQL Server tömeges másolási (bcp) és az AzCopy és hasonló eszközökkel való adatok másolása Blob storage-bA. Majd betöltheti az adatokat közvetlenül az SQL Data Warehouse a Polybase használatával.
* Ha nagy méretű adatkészleteken, érdemes [Data Lake Storage](/azure/storage/data-lake-storage/introduction), amely biztosítja a korlátlan tárolási elemzési adatok.
* Egy helyszíni [SQL Server Parallel Data Warehouse](/sql/analytics-platform-system) berendezés használata big data-feldolgozáshoz is használható. Azonban működési költségek mellett általában sokkal alacsonyabb, mint az SQL Data Warehouse felügyelt felhőalapú megoldással. 
* Az SQL Data Warehouse nem jó megoldás lehet OLTP számítási feladatokat vagy 250GB-nál kisebb adatkészletek. Ezekben az esetekben az Azure SQL Database vagy SQL Server kell használnia.
* Az egyéb alternatívák összehasonlítások az alábbi témakörben talál:
    * [Egy data pipeline vezénylési technológia kiválasztása az Azure-ban](/azure/architecture/data-guide/technology-choices/pipeline-orchestration-data-movement)
    * [Egy köteg feldolgozása az Azure-ban technológia kiválasztása](/azure/architecture/data-guide/technology-choices/batch-processing)
    * [Az Azure-ban egy analitikai adattár kiválasztása](/azure/architecture/data-guide/technology-choices/analytical-data-stores)
    * [Egy data analytics technológia kiválasztása az Azure-ban](/azure/architecture/data-guide/technology-choices/analysis-visualizations-reporting)

## <a name="considerations"></a>Megfontolandó szempontok

Ebben az architektúrában a technológiákat, a skálázhatóság és rendelkezésre állás, a vállalat követelményei ezáltal szabályozhatja a költségeket, miközben telepítve lettek kiválasztva.

* A [nagymértékben párhuzamos feldolgozási architektúra](/azure/sql-data-warehouse/massively-parallel-processing-mpp-architecture) az SQL Data Warehouse a méretezhetőséget és teljesítményt biztosít.
* Az SQL Data Warehouse rendelkezik [garantált SLA-kkal](https://azure.microsoft.com/support/legal/sla/sql-data-warehouse) és [ajánlott eljárások a magas rendelkezésre állás elérésének](/azure/sql-data-warehouse/sql-data-warehouse-best-practices).
* Elemzés a tevékenységet, alacsony, ha a vállalat is [igény szerinti méretezés az SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-manage-compute-overview), csökkentése, vagy akár a költségek csökkentése a számítás felfüggesztése.
* Az Azure Analysis Services lehet [horizontálisan felskálázott](/azure/analysis-services/analysis-services-scale-out) feladatok esetén a válaszidők csökkentése érdekében. Is elkülönítheti a feldolgozása a lekérdezési készletből, így az ügyfél lekérdezések nem lassítsa le a feldolgozási műveletek. 
* Az Azure Analysis Services is rendelkezik [garantált SLA-kkal](https://azure.microsoft.com/support/legal/sla/analysis-services) és [ajánlott eljárások a magas rendelkezésre állás elérésének](/azure/analysis-services/analysis-services-bcdr).
* A [biztonsági modellt az SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-overview-manage-security) kapcsolatbiztonság, biztosít [hitelesítési és engedélyezési](/azure/sql-data-warehouse/sql-data-warehouse-authentication) az Azure AD-n keresztül vagy az SQL Server-hitelesítés és titkosítás. [Az Azure Analysis Services](/azure/analysis-services/analysis-services-manage-users) használja az Azure AD identity management és a felhasználók hitelesítéséhez. 

## <a name="pricing"></a>Díjszabás

Tekintse át a [díjszabása a minta egy adatraktározási forgatókönyv] [ calculator] keresztül az Azure díjkalkulátorát. Állítsa be az értékek megtekintéséhez, hogyan érinti az igényeinek a költségeit.

* [Az SQL Data Warehouse](https://azure.microsoft.com/pricing/details/sql-data-warehouse/gen2) lehetővé teszi, hogy egymástól függetlenül méretezheti a számítási és tárolási szintek. A számítási erőforrások számlázása óránként történik, és igény szerint ezeket az erőforrásokat szüneteltetheti vagy méretezhetők. Tárolási erőforrások számlázása terabájt, így növeli a költségeket, akkor több adatot képes feldolgozni.
* [A Data Factory](https://azure.microsoft.com/pricing/details/data-factory) költségeket olvasási és írási műveletek, a megfigyelési műveleteket és a egy számítási végrehajtott vezénylési tevékenységek száma. A Data Factory költségeket minden további adatfolyam és mindegyikhez által feldolgozott adatok mennyisége növekszik.
* [Analysis Services](https://azure.microsoft.com/pricing/details/analysis-services) fejlesztői, alapszintű és standard szinten érhető el. Példányok díjszabása alapján lekérdezésfeldolgozó egységek (qpu-ra) és a rendelkezésre álló memória. Hogy a költség alacsonyabb, a lekérdezések feldolgozásához, hogy mennyi adatot futtatja, a lehető legkevesebb, és milyen gyakran futnak.
* [Power bi-ban](https://powerbi.microsoft.com/pricing) különböző termék lehetőséget eltérő követelmények vonatkoznak. [Power BI Embedded](https://azure.microsoft.com/pricing/details/power-bi-embedded) beágyazásához Power BI funkcióit belül az alkalmazások Azure-alapú lehetőséget biztosít. A Power BI Embedded-példány a fenti díjszabási mintát tartalmazza.

## <a name="next-steps"></a>További lépések

* Tekintse át a [automatizált vállalati bi-ban az Azure-referenciaarchitektúra](/azure/architecture/reference-architectures/data/enterprise-bi-adf), amely ismerteti, hogyan telepíthető egy példányát az Azure-ban ez az architektúra tartalmazza.
* Olvassa el a [Maritz motiváció megoldások vásárlói beszámolónk][source-document]. Történet ügyféladatok kezeléséhez hasonló megközelítést ismerteti.
* Az adatfolyamatok, adattárházak, online elemzésfeldolgozási (OLAP) és big Data típusú adatok átfogó architekturális útmutatást találhat a [Azure-Adatarchitektúrához](/azure/architecture/data-guide).

<!-- links -->
[source-document]: https://customers.microsoft.com/story/maritz
[calculator]: https://azure.com/e/b798fb70c53e4dd19fdeacea4db78276
[architecture]: ./media/architecture-data-warehouse.png
