---
title: Skálázható rendelésfeldolgozás
titleSuffix: Azure Example Scenarios
description: Nagymértékben skálázható rendelésfeldolgozó folyamatot hozhat létre az Azure Cosmos DB használatával.
author: alexbuckgit
ms.date: 07/10/2018
ms.openlocfilehash: fe4e9d64e96d0be66534198bc60e2a73dad43e84
ms.sourcegitcommit: bb7fcffbb41e2c26a26f8781df32825eb60df70c
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/20/2018
ms.locfileid: "53644189"
---
# <a name="scalable-order-processing-on-azure"></a>Skálázható rendelésfeldolgozás az Azure-ban

Ebben a példaforgatókönyvben fontos szervezetek számára szükséges egy hatékonyan méretezhető és rugalmas architektúra online megrendelések feldolgozása. Lehetséges alkalmazások közé tartozik az e-kereskedelmi, és a pénztári, megrendelések teljesítése, és leltár-foglalási és követési kiskereskedelmi.

Ebben a forgatókönyvben egy event sourcing megközelítés implementálása mikroszolgáltatások funkcionális programozási modell használatával vesz igénybe. Mindegyik mikroszolgáltatás egy adatfolyam-feldolgozó számít, és az összes üzleti logika mikroszolgáltatások implementálása. Ez a megközelítés lehetővé teszi a magas rendelkezésre állás és rugalmasság, a georeplikáció és gyors teljesítményt.

Azure-szolgáltatások például a Cosmos DB és a HDInsight használata segíthet a Microsoft globálisan elosztott felhőméretű adatok tárolásához és lekéréséhez szakértelmet kihasználva csökkentheti a költségeket. Ebben a forgatókönyvben kifejezetten címek egy e-kereskedelmi vagy kereskedelmi forgatókönyvben; Ha más igényeinek megfelelően a data services, a rendelkezésre álló lista tekintse át [teljes körűen felügyelt intelligens adatbázis-szolgáltatások az Azure-ban][product-category].

## <a name="relevant-use-cases"></a>Alkalmazási helyzetek

Egyéb alkalmazási helyzetek a következők:

- E-kereskedelmi vagy kiskereskedelem pénztári háttérrendszerekhez.
- Leltárkezelő rendszerei.
- Rendelés teljesítése rendszerek.
- Egyéb integrációs forgatókönyvek egy rendelés feldolgozási folyamat kapcsolódik.

## <a name="architecture"></a>Architektúra

![Példa architektúra méretezhető megrendelések feldolgozása folyamatok][architecture]

Ez az architektúra egy rendelés feldolgozási folyamat főbb összetevőit ismerteti. A áramlanak keresztül az adatok a forgatókönyv a következő:

1. Eseményt üzenetek érintkező alkalmazások számára (szinkron módon HTTP protokollon keresztül) és a különböző háttérrendszerekhez (aszinkron módon útján az Apache Kafka) keresztül adja meg a rendszer. Ezeket az üzeneteket a rendszer átadja a parancs feldolgozási folyamatba.
2. Minden egyes eseményüzenet betöltött és mikroszolgáltatások parancs processzor által leképezett egy meghatározott készletével parancsok egyikét. A parancs processzor bármely aktuális állapotát a parancs végrehajtása egy esemény-adatfolyam pillanatkép adatbázisból kérdezi le. A parancs végrehajtásakor majd, és a parancs kimenete új esemény áll rendelkezésre.
3. Minden egyes esemény egy parancs kimenete többszöröseként számára fontos, hogy egy esemény stream database, Cosmos DB használatával.
4. Minden adatbázis Beszúrás vagy frissítés, az esemény-adatfolyam adatbázisban véglegesített egy által a Cosmos DB módosítási hírcsatorna-esemény jelenik meg. Generáló rendszerek üzenetkézbesítési igényeit is témákra iratkozhat fel minden olyan esemény, amely a rendszerben.
5. A Cosmos DB módosítási hírcsatorna összes eseménye is kapnak egy pillanatkép esemény stream mikroszolgáltatás, amely kiszámítja minden állapotváltozások történt események okozzák. Az új állapot majd véglegesítése Cosmos DB-ben tárolt esemény-adatfolyam pillanatkép adatbázishoz. A pillanatkép-adatbázis összes adatelemek aktuális állapotát egy globálisan elosztott, alacsony késleltetésű adatforrást biztosít. Az esemény-adatfolyam adatbázis biztosít, amelyek megfeleltek az architektúrához, amely lehetővé teszi a hatékony tesztelési, hibaelhárítási és vészhelyreállítási forgatókönyvek eseményt üzenetek egy teljes rekord.

### <a name="components"></a>Összetevők

- [A cosmos DB](/azure/cosmos-db/introduction) a Microsoft globálisan elosztott, többmodelles adatbázis, amely a megoldásokat, rugalmasan és egymástól függetlenül méretezhető átviteli sebesség és tárterület tetszőleges számú földrajzi régió között. Átviteli sebesség, a késés, a rendelkezésre állást kínál, és a konzisztenciára vonatkozó garanciákat biztosít átfogó szolgáltatási szerződésekre (SLA). Ebben a forgatókönyvben használ a Cosmos DB esemény stream-tárhely és a pillanatkép-tároló, és kihasználja [Cosmos DB-módosítási hírcsatorna] [ docs-cosmos-db-change-feed] szolgáltatások adat-helyreállítás konzisztencia és a hibatűrés biztosításához.
- [Az Apache Kafka on HDInsight](/azure/hdinsight/kafka/apache-kafka-introduction) egy felügyelt szolgáltatás megvalósítása az Apache Kafka, egy nyílt forráskódú elosztott streamelési platform streamadatfolyamatok és -alkalmazások létrehozásához. Kafka, üzenetsorokhoz hasonló üzenetközvetítő funkciót is biztosít a közzététel és feliratkozás adatstreameket. Ebben a forgatókönyvben a Kafka használatával feldolgozni a bejövő, valamint a folyamat feldolgozása sorrendben alsóbb rétegbeli események.

## <a name="considerations"></a>Megfontolandó szempontok

A valós idejű üzenetbetöltés, az adattárolás, az adatfolyam-feldolgozás, storage, az analitikus adatok és elemzési és jelentéskészítési számos technológiai lehetőségek érhetők el. Ezek a beállítások, képességek és fontosabb kritériumok áttekintését lásd: [Big data-architektúrák: Valós idejű feldolgozás](/azure/architecture/data-guide/technology-choices/real-time-ingestion) a a [Azure-Adatarchitektúrához](/azure/architecture/data-guide).

A mikroszolgáltatás egy népszerű architekturális stílussá vált a rugalmas, hatékonyan méretezhető, függetlenül üzembe helyezhető és gyorsan fejleszthető felhőalkalmazások létrehozásához. Mikroszolgáltatás-alapú alkalmazások tervezésekor és létrehozásakor eltérő megközelítést igényelnek. Például a Docker, Kubernetes, az Azure Service Fabric és Nomad eszközei lehetővé teszik a mikroszolgáltatás-alapú architektúrák fejlesztését. Egy mikroszolgáltatás-alapú architektúra készítése és útmutatásért lásd: [az Azure mikroszolgáltatások tervezése] (/ azure/architektúrával/mikroszolgáltatások az Azure Architecture Centert.

### <a name="availability"></a>Rendelkezésre állás

Ebben a forgatókönyvben események forráskezelése megközelítés lehetővé teszi, hogy a rendszer összetevőinek lazán összekapcsolva, és egymástól függetlenül üzembe helyezett. A cosmos DB-ajánlatok [magas rendelkezésre állású] [ docs-cosmos-db-regional-failover] , és segít a kompromisszumot kínál a konzisztencia, a rendelkezésre állás és a teljesítmény, a kapcsolódó kezelésében [megfelelő garanciákkal ][docs-cosmos-db-guarantees]. Az Apache Kafka on HDInsight is képes [magas rendelkezésre állású][docs-kafka-high-availability].

Az Azure Monitor egységes felhasználói felületet biztosít a különböző Azure-szolgáltatások figyelésére. További információkért lásd: [a Microsoft Azure figyelés](/azure/monitoring-and-diagnostics/monitoring-overview). Az Event Hubs és Stream Analytics is integrálva van az Azure Monitor szolgáltatással.

Más rendelkezésre állási lehetőségekért lásd: a [rendelkezésre állási ellenőrzőlista][availability].

### <a name="scalability"></a>Méretezhetőség

A HDInsight alatt futó Kafka lehetővé teszi, hogy [tárolójának és skálázhatóságának konfigurálása](/azure/hdinsight/kafka/apache-kafka-scalability) Kafka-fürtök. A cosmos DB gyors és kiszámítható teljesítményt nyújt és [zökkenőmentesen skálázható](/azure/cosmos-db/partition-data) az alkalmazás növekedésével.
A mikroszolgáltatás-alapú architektúra ebben a forgatókönyvben is sourcing esemény megkönnyíti a rendszer méretezése, és bontsa ki a működését.

Más méretezhetőség szempontokért lásd: a [méretezési ellenőrzőlista] [ scalability] elérhető az Azure Architecture Centert.

### <a name="security"></a>Biztonság

A [Cosmos DB biztonsági modell](/azure/cosmos-db/secure-access-to-data) hitelesíti a felhasználókat, és lehetővé teszi az adatok és erőforrások elérését. További információkért lásd: [Cosmos DB-adatbázis biztonsági](/azure/cosmos-db/database-security).

Általános megoldások biztonságos, tekintse át a [Azure Security dokumentációja][security].

### <a name="resiliency"></a>Rugalmasság

Az események forráskezelése minta alkalmazása, architektúra és a kapcsolódó technológiák révén, a példában ennek az alkalmazási helyzetnek rendkívül rugalmas hibák esetén. Rugalmas megoldások tervezésével kapcsolatos általános útmutatásért lásd: [rugalmas alkalmazások tervezése az Azure][resiliency].

## <a name="pricing"></a>Díjszabás

Ez a forgatókönyv futtatásával járó költségeket vizsgálatához a szolgáltatások mindegyike a költségkalkulátor az előre konfigurált. Tekintse meg, hogyan díjszabását szeretné módosítani az adott forgatókönyvhöz, módosítsa a megfelelő változók várható adatmennyiség megfelelően. Ebben a forgatókönyvben a példa díjak magukban foglalják az csak a Cosmos DB és a Kafka-fürt esetében a Cosmos DB módosítási hírcsatorna a kódhibák miatt keletkeznek eseményeket dolgoz fel. Esemény processzorok és a mikroszolgáltatások a tartományvezérlőkről és egyéb alsóbb rétegbeli rendszerek nem tartoznak, és a költségek nagyban függ a mennyiség és a skála ezeket a szolgáltatásokat, valamint az őket végrehajtási választott technológiákat.

Az Azure Cosmos DB pénzneme a kérelemegység (RU). Kérelemegységgel nem kell lefoglalni az olvasási/írási kapacitások vagy kiépítése CPU, memória és iops-t. Az Azure Cosmos DB támogatja a különböző API-k, amelyek különböző műveleteket, és a simple beolvassa és az összetett graph-lekérdezéseket. Mivel nem minden kérelmek egyenlő, a kérelmek kérelemegység alapján a számítást, ami szükséges a kérés kiszolgálása összegének normalizált mennyisége vannak hozzárendelve. A megoldás által igényelt kérelemegységek számát függ, adatméret elem és, adatbázis olvasási és írási műveletek száma másodpercenként. További információkért lásd: [Azure Cosmos DB-Kérésegységeiről](/azure/cosmos-db/request-units). Ezeket a Díjszámítás alapját a Cosmos DB két Azure-régióban működő becsült.

Adtunk meg három példa költség profilok tervez tevékenységek mennyisége alapján:

- [Kis][small-pricing]: a díjszabási példa az 1 TB-os adattár a Cosmos DB és a egy kisméretű (D3 v2) történő 5 Kérelemegységet fenntartott Kafka-fürt.
- [Közepes][medium-pricing]: a díjszabási példa egy 10 TB-os data store a Cosmos DB és a egy vállalkozás (D4 v2) történő 50 RUs fenntartott Kafka-fürt.
- [Nagy][large-pricing]: 500 RUs történő fenntartott a Cosmos DB és a egy nagy méretű (D5 v2) 30 TB adattár Kafka-fürt a díjszabási példa.

## <a name="related-resources"></a>Kapcsolódó források (lehet, hogy a cikkek angol nyelvűek)

Ebben a példaforgatókönyvben alapján ez az architektúra által készített szélesebb körű verziója [Jet.com adatfeldolgozási főigazgatója](https://jet.com) a teljes körű rendelés feldolgozási folyamat számára. További információkért lásd: a [Jet.com adatfeldolgozási főigazgatója műszaki felhasználói profil] [ source-document] és [jet.com a bemutató Build 2018][source-presentation].

Egyéb kapcsolódó erőforrások a következők:

- *[Adatigényes alkalmazások tervezése](https://dataintensive.net)*  Martin Kleppmann (O'Reilly média, 2017.) szerint.
- *[Tartomány modellezési végzett működési: Szoftver köszönhetően Tartományvezérelt tervezési kiszolgálókon és F# ](https://pragprog.com/book/swdddf/domain-modeling-made-functional)*  Scott Wlaschin (pragmatikus programozók LLC, 2018.) szerint.
- Más [a használati esetek Cosmos DB][docs-cosmos-db-use-cases]
- [Valós idejű feldolgozása architektúra](/azure/architecture/data-guide/big-data/real-time-processing) a a [Azure-Adatarchitektúrához](/azure/architecture/data-guide).

<!-- links -->

[architecture]: ./media/architecture-ecommerce-order-processing.png
[product-category]: https://azure.microsoft.com/product-categories/databases/
[source-document]: https://customers.microsoft.com/story/jet-com-powers-innovative-e-commerce-engine-on-azure-in-less-than-12-months
[source-presentation]: https://channel9.msdn.com/events/Build/2018/BRK3602
[small-pricing]: https://azure.com/e/3d43949ffbb945a88cc0a126dc3a0e6e
[medium-pricing]: https://azure.com/e/1f1e7bf2a6ad4f7799581211f4369b9b
[large-pricing]: https://azure.com/e/75207172ece94cf6b5fb354a2252b333
[docs-cosmos-db-change-feed]: /azure/cosmos-db/change-feed
[docs-cosmos-db-regional-failover]: /azure/cosmos-db/regional-failover
[docs-cosmos-db-guarantees]: /azure/cosmos-db/distribute-data-globally#AvailabilityGuarantees
[docs-cosmos-db-use-cases]: /azure/cosmos-db/use-cases
[docs-kafka-high-availability]: /azure/hdinsight/kafka/apache-kafka-high-availability
[docs-event-hubs]: /azure/event-hubs/event-hubs-what-is-event-hubs
[docs-stream-analytics]: /azure/stream-analytics/stream-analytics-introduction
[availability]: /azure/architecture/checklist/availability
[scalability]: /azure/architecture/checklist/scalability
[resiliency]: /azure/architecture/patterns/category/resiliency/
[security]: /azure/security/
