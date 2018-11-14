---
title: IoT és adatelemzés az építőiparban
description: IoT-eszközök és adatelemzés használatával biztosíthatja az építési projektek átfogó felügyeletét és működését.
author: alexbuckgit
ms.date: 08/29/2018
ms.openlocfilehash: 74868191687e63a54a69fdacb7276983d98faf74
ms.sourcegitcommit: 0a31fad9b68d54e2858314ca5fe6cba6c6b95ae4
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/13/2018
ms.locfileid: "51610923"
---
# <a name="iot-and-data-analytics-in-the-construction-industry"></a>IoT és adatelemzés az építőiparban

Ebben a példaforgatókönyvben megoldások, amelyek számos IoT-eszközökről származó adatok integrálása egy átfogó adatok elemzési architektúrát automatizálhatja a döntéshozatalhoz és a szervezetek számára fontos. Lehetséges alkalmazások konstrukció, adatbányászati, gyártási vagy egyéb iparági megoldások használata esetén a nagy mennyiségű IoT-alapú adatok számos bemeneti adatait tartalmazza.

Ebben a forgatókönyvben egy konstrukció gyártó járművek, mérőszámok és drónok, amely telemetriai adatokat küldik IoT és GPS-technológiák használatával hoz létre. A vállalat szeretne azok jobban a működési feltételek és berendezések állapotának monitorozása a data-architektúra korszerűsítéséhez. Cserélje le a vállalati örökölt megoldás használatával a helyszíni infrastruktúrát is idő és munkaerő-igényes lenne, és nem tudná kezelni a várható adatmennyiség megfelelően skálázhatja.

A vállalat egy felhőalapú "intelligens konstrukció" megoldás kiépítését. Kell egy átfogó építkezés-adatok összegyűjtése és a művelet és a különböző elemek, a hely karbantartási automatizálása. A vállalati célok a következők:

* Integrálásához, és minden konstrukció elemzése a hely eszközöket és az adatokat a berendezések állásidő minimálisra csökkentése érdekében, és a lopás csökkentése érdekében.
* Távolról, és automatikusan szabályozása konstrukció berendezések munkaerő hiány, végső soron kevesebb feldolgozó valamint engedélyezése sikeres alacsonyabb magasan képzett dolgozók által okozott hatások mérsékléséhez.
* A működési költségeket és munkaerő követelményei támogató infrastruktúra, minimalizálja a termelékenység és a biztonság növelése mellett.
* Egyszerű méretezés növeli a telemetriai adatok támogatásához az infrastruktúra.
* Megfelel az összes releváns jogi követelmények által belföldi erőforrások kiépítése a rendszer rendelkezésre állását veszélyeztetése nélkül.
* Nyílt forráskódú szoftvert használ a dolgozók eddigi tapasztalatai fordított befektetésének maximalizálása érdekében.

Például az IoT Hub és a HDInsight a felügyelt Azure-szolgáltatások használata lehetővé teszi az ügyfél rövid idő alatt készíthet, és helyezhet üzembe átfogó megoldást egy alacsonyabb üzemeltetési költség. Ha további adatokat elemzést, a rendelkezésre álló lista tekintse át [teljes körűen felügyelt adatok elemzési szolgáltatások az Azure-ban][product-category].

## <a name="relevant-use-cases"></a>Alkalmazási helyzetek

Egyéb alkalmazási helyzetek a következők:

* Fejlesztés, adatbányászati vagy berendezés gyártási forgatókönyvek
* Eszköz adatok tárolására és elemzésére szolgáló nagyméretű gyűjteménye
* Adatbetöltési és -elemzés a nagyméretű adathalmazok

## <a name="architecture"></a>Architektúra

![IoT és data analytics konstrukció iparág architektúrája][architecture]

Az adatok a következőképpen folyamatok a megoldáson keresztül:

1. Érzékelői adatokat gyűjti és elküldi a konstrukció eredmények adatok betöltése az Azure virtuálisgép-fürt által futtatott elosztott terhelésű webszolgáltatások rendszeres időközönként berendezés.
2. Az egyéni webszolgáltatások konstrukció eredmények adatbetöltést és -tárolja is az Azure virtuális gépeken futó Apache Cassandra-fürtben.
3. Egy másik adatkészlet gyűjti az adatokat a különböző konstrukció berendezések IoT-érzékelőket, és az IoT hubnak küldött.
4. Nyers adatok gyűjtött van az Azure blob storage-ba közvetlenül az IoT hubról küldött, és azonnal elérhető megtekintése és elemzése.
5. Az IoT Hub-n keresztül gyűjtött adatok közel valós időben az Azure Stream Analytics-feladat által feldolgozott és egy Azure SQL database tárolja.
6. A fejlesztés az intelligens felhő webalkalmazás elemzők és a végfelhasználók megtekintése és elemzése, érzékelőadatok és képanyag érhető el. 
7. Batch-feladatok a webalkalmazás felhasználók által kezdeményezett igény szerinti. A kötegelt feldolgozást a HDInsight Apache Spark fut, és elemzi az új adatokat tárolja a Cassandra-fürt. 

### <a name="components"></a>Összetevők

* [Az IoT Hub](/azure/iot-hub/about-iot-hub) eszközönkénti identitás- és más hely elemek a felhőalapú platform és a berendezés közötti biztonságos kétirányú kommunikációt központi üzenet kiszolgálókezelési. Az IoT Hub gyorsan gyűjtheti támogatunk minden egyes eszközhöz a data analytics folyamatba. 
* [Az Azure Stream Analytics](/azure/stream-analytics/stream-analytics-introduction) olyan eseményfeldolgozó motor, amely a nagy mennyiségű adat a eszközök és más adatforrásokhoz is elemezhet. Támogatja a kinyerésekor információk adatfolyamokból, minták és kapcsolatok is. Ebben a forgatókönyvben a Stream Analytics fogadnak, és IoT-eszközökről származó adatokat elemzi, és az Azure SQL Database tárolja az eredményeket. 
* [Az Azure SQL Database](/azure/sql-database/sql-database-technical-overview) az eredményeket az IoT-eszközök és a mérőszámok, amely tekinthet meg elemzők és a egy Azure-alapú webes alkalmazás felhasználók által elemzett adatait tartalmazza. 
* [A BLOB storage-](/azure/storage/blobs/storage-blobs-introduction) tárolók kép összegyűjtött adatok alapján az IoT hub-eszközöket. A rendszerkép-adatok a webalkalmazás-n keresztül is megtekinthetők.
* [A TRAFFIC Manager](/azure/traffic-manager/traffic-manager-overview) szabályozza a különböző Azure-régiókban szolgáltatásvégpontokra érkező felhasználói forgalom elosztása.
* [Load Balancer](/azure/load-balancer/load-balancer-overview) adatok jelentkezés konstrukció berendezések eszközökről elosztja a Virtuálisgép-alapú webes szolgáltatások magas rendelkezésre állást biztosít.
* [Az Azure Virtual Machines](/azure/virtual-machines) futtatja a webes szolgáltatásokat, amelyek fogadják és adatbetöltést konstrukció eredményeit az Apache Cassandra-adatbázisba.
* [Az Apache Cassandra](https://cassandra.apache.org) későbbi feldolgozásra az Apache Spark a fejlesztés adatok tárolására használt elosztott NoSQL-adatbázis.
* [Web Apps](/azure/app-service/app-service-web-overview) lekérdezését, és megtekintheti a forrásadatok és képek segítségével végfelhasználói webes alkalmazást. Felhasználó is kezdeményezheti a kötegelt feladatok Apache Spark az alkalmazáson keresztül.
* [Az Apache Spark on HDInsight](/azure/hdinsight/spark/apache-spark-overview) támogatja a memórián belüli feldolgozást a big data elemző alkalmazások teljesítményének növelése érdekében. Ebben a forgatókönyvben a Spark futtatására szolgál összetett algoritmusokat az Apache Cassandra tárolt adatokon.


### <a name="alternatives"></a>Alternatív megoldások

* [A cosmos DB](/azure/cosmos-db/introduction) egy alternatív NoSQL adatbázis-technológia. A cosmos DB biztosít [globális méretekben több főkiszolgálós támogatást](/azure/cosmos-db/multi-region-writers) a [több jól definiált konzisztenciaszintet](/azure/cosmos-db/consistency-levels) különböző ügyfél-követelmények teljesítéséhez. Ezen kívül támogatja az [Cassandra API](/azure/cosmos-db/cassandra-introduction). 
* [Az Azure Databricks](/azure/azure-databricks/what-is-azure-databricks) egy Azure-ra optimalizált Apache Spark-alapú elemzési platform. Integrálva van az Azure-ral, egyetlen kattintással beállítható, zökkenőmentes munkafolyamatokat és egy együttműködésen alapuló interaktív munkaterületet biztosít.
* [Data Lake Storage](/azure/storage/data-lake-storage) alternatívája a Blob storage. Ebben az esetben a Data Lake Storage nem volt elérhető a célként megadott régióban.
* [Web Apps](/azure/app-service) a webes szolgáltatások üzemeltetésére, az adatok feldolgozására konstrukció eredményeket is használható.
* A valós idejű üzenetbetöltés, az adattárolás, az adatfolyam-feldolgozás, storage, az analitikus adatok és elemzési és jelentéskészítési számos technológiai lehetőségek érhetők el. Ezek a beállítások, képességek és fontosabb kritériumok áttekintését lásd: [Big data-architektúrák: valós idejű feldolgozás](/azure/architecture/data-guide/technology-choices/real-time-ingestion) a a [Azure-Adatarchitektúrához](/azure/architecture/data-guide).

## <a name="considerations"></a>Megfontolandó szempontok

Az Azure-régiók rendelkezésre állásának széles körű ebben a forgatókönyvben egy fontos tényező. Több régióban rendelkezik egy ország biztosíthat a vész-helyreállítási szerződéses kötelezettségek és a törvény kényszerítési követelményeinek való megfelelés is engedélyezése közben. Nagy sebességű kommunikációt az Azure-régiók között is ebben a forgatókönyvben egy fontos tényező.

Azure-támogatás a nyílt forráskódú technológiákat az ügyfél kihasználhatja meglévő szaktudásukkal munkaerő számára engedélyezett. Az ügyfél is gyorsítható fel kisebb költséggel és üzemi számítási feladatokat a helyszíni megoldásokhoz képest az új technológiák bevezetése. 

## <a name="pricing"></a>Díjszabás

Az alábbi szempontokat a hatékony felhasználhatóságot ebben a megoldásban a költségek jelentős részét.

* Azure virtuális gépek költségeit költségráfordításokkal egyenes arányban növekszik, további példányokat felhasznált. Virtuális gépek felszabadított állapotban vannak, csak a tárolási költségekkel, és nem számítási költségeit. Ezek a gépek felszabadítva majd kell újra kiosztja, ha igény szerint magas.
* [Az IoT Hub](https://azure.microsoft.com/pricing/details/iot-hub) költségek alakítják üzembe helyezett IoT-egységet, valamint a szolgáltatási rétegben választása esetén a számát, amely meghatározza, hogy az engedélyezett egységenként naponta üzenetek száma. 
* [Stream Analytics](https://azure.microsoft.com/pricing/details/stream-analytics) díjszabása a szolgáltatásba az adatok feldolgozásához szükséges folyamatos átviteli egységek száma alapján.

## <a name="related-resources"></a>Kapcsolódó források (lehet, hogy a cikkek angol nyelvűek)

Szeretné implementálhat hasonló architektúrával, olvassa el a [Komatsu vásárlói beszámolónk][customer-story].

Útmutató a big data-architektúrák érhető el a [Azure-Adatarchitektúrához](/azure/architecture/data-guide).

<!-- links -->
[product-category]: https://azure.microsoft.com/product-categories/analytics/
[customer-site]: https://home.komatsu/en/
[customer-story]: https://customers.microsoft.com/story/komatsu-manufacturing-azure-iot-hub-japan
[architecture]: ./media/architecture-big-data-with-iot.png
