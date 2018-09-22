---
title: Adatfeldolgozást és a valós idejű autógyártás IOT-adatok feldolgozása
description: Big Data-feldolgozási folyamat járművek valós idejű IoT-adatok betöltése és feldolgozása.
author: meeral
ms.date: 09/12/2018
ms.openlocfilehash: 5c3a43332a24c9ec60d8257363b9968eb4ce97d4
ms.sourcegitcommit: b7e521ba317f4fcd3253c80ac0c0a355eaaa56c5
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 09/21/2018
ms.locfileid: "46534691"
---
# <a name="ingestion-and-processing-of-real-time-automotive-iot-data"></a>Adatfeldolgozást és a valós idejű autógyártás IOT-adatok feldolgozása

Ebben a példaforgatókönyvben összeállít egy valós idejű adatfeldolgozást és -feldolgozási adatfolyamat kell fogadni és feldolgozni az üzenetet az IoT-eszközök (az általános érzékelők) a big Data típusú adatok elemzési platform az Azure-ban. Járműtelemetria telematika adatfeldolgozást és -feldolgozási rendszerek a következők csatlakoztatott autó-megoldásokat hozhat létre. Ebben a forgatókönyvben az autó-telematika adatfeldolgozást és -feldolgozási rendszerek alapul. Azonban a kialakítási minták számos iparágban érzékelők segítségével felügyelheti és figyelheti a komplex rendszerek például okosépületek, kommunikációs, gyártási, kereskedelmi és egészségügyi iparágak szempontjából is.

Ez a példa bemutatja a valós idejű adatfeldolgozást és -feldolgozási adatfolyamat üzenetek járművek telepített IoT-eszközökről. Több ezer és üzenetek (vagy események) több millió IoT-eszközök és érzékelők által jönnek létre. Rögzítésével és elemzésével ezeket az üzeneteket, hogy visszafejteniük értékes információkat és megfelelő műveleteket. Például telematika eszközökkel felszerelt autók, ha az eszköz (IoT) üzenetek valós időben rögzíthetők a hétköznapi azt tudná járművek élő helyen figyelheti, megtervezhetik az útvonalakat optimalizált, nyújt segítséget az illesztőprogramok és iparágak telematika használatával kapcsolatos támogatási például az automatikus biztosítási.

Ebben a példában a bemutatóban az imagine egy autó gyártóvállalat cég, amely szeretne létrehozni egy valós idejű rendszer kell fogadni és feldolgozni üzenetek telematika eszközökről. A vállalati célok a következők:
* Betöltési, és valós idejű járművek érzékelőkről és eszközökről érkező adatok tárolására.
* Elemezze a jármű helyét és egyéb információkat a kibocsátott keresztül (például motor kapcsolatos érzékelők és a környezettel kapcsolatos érzékelők) érzékelők különböző típusú üzeneteket.
* Az egyéb alárendelt feldolgozáshoz, gyakorlatban hasznosítható elemzéseket biztosít elemzést a data Store (például objektuma forgatókönyvekben, biztosítási szervek is érdekelheti, mi történt során baleset stb.)

## <a name="potential-use-cases"></a>A lehetséges alkalmazási helyzetek

Ebben a forgatókönyvben a következő használati esetek, a fenti célok mellett vegye figyelembe, adatfeldolgozást és -feldolgozási rendszerek telematika létrehozásakor:

* Járműtelemetria karbantartási emlékeztetők és riasztási.
* Helyalapú szolgáltatások, a vehicle utasok (azaz SOS)
* Önálló (önálló vezetési) járművek.

## <a name="architecture"></a>Architektúra

![Képernyőfelvétel](media/architecture-diagram-realtime-analytics-vehicle-data1.png)

Egy tipikus big data típusú adatfeldolgozást folyamat végrehajtása a adatáramlás balról jobbra. A valós idejű big Data jellegű adatok feldolgozása folyamatban a áramlanak keresztül az adatok a megoldás a következő:

1. Az IoT adatforrásokból generált események, az üzenetek küldése a streamfeldolgozási streamrétegében keresztül az Azure HDInsight Kafka.  HDInsight Kafka témakörökben konfigurálható ideig tárolja az adatstreameket.
2. A Kafka-fogyasztók, az Azure Databricks, fogadja az üzenetet a Kafka-témakörbe, valós időben feldolgozni az adatokat az üzleti logika alapján, és ezután elküldheti a szolgáltató a tárolási réteg.
3. Alsóbb rétegbeli tárolási szolgáltatásoktól, például az Azure cosmos DB, Azure SQL Data warehouse-bA vagy az Azure SQL DB, lesz egy adatforrást a bemutató és műveleti réteg.
4. Üzleti elemzők a Microsoft Power BI segítségével raktározott adatok elemzéséhez. Más alkalmazások épülhetnek a kiszolgálórétegbe. Ha például tehetjük elérhetővé a külső felhasználásra szolgáltatási réteg adatokon alapuló API-k.

### <a name="components"></a>Összetevők
Létrehozott események (adatok vagy üzenetek) vannak betöltött, IoT-eszköz feldolgozott, és ott további elemzés, bemutatót és művelet, a következő Azure-összetevők használatával:
* [HDInsight Kafka](https://docs.microsoft.com/en-us/azure/hdinsight/kafka/apache-kafka-introduction) a feldolgozási réteg van. A kafka producer API-t használjuk az adatok írása a Kafka-témakörbe.
* [Az Azure Databricks](https://azure.microsoft.com/en-us/services/databricks/) az Adatátalakítási és elemzési rétegben található. Databricks-jegyzetfüzeteket megvalósítása a kafka-fogyasztók API-t a megadott olvasni a Kafka-témakört.
* [Azure cosmos DB](https://azure.microsoft.com/en-us/services/cosmos-db/), [Azure SQL DB](https://docs.microsoft.com/en-us/azure/sql-database/), és [Azure SQL Data Warehouse](https://review.docs.microsoft.com/en-us/azure/sql-data-warehouse) lévő tárolási kiszolgálórétegbe ahol [Azure Databricks](https://azure.microsoft.com/en-us/services/databricks/) írhat, az adatok használatával Az adatösszekötők.
* [Az Azure SQL Data Warehouse](https://review.docs.microsoft.com/en-us/azure/sql-data-warehouse) egy elosztott rendszer tárolására és elemzésére a nagyméretű adathalmazok. A masszív párhuzamos feldolgozási (MPP) is teszi megfelelő a futó, nagy teljesítményű elemzési.
* [Power bi-ban](https://review.docs.microsoft.com/en-us/power-bi) adatok elemzése és elemzéseket oszthat meg üzleti elemzési eszközök együttese. [Power bi-ban](https://review.docs.microsoft.com/en-us/power-bi) lekérdezheti az Analysis Servicesben tárolt szemantikai modellt, vagy közvetlenül lekérdezheti azt az SQL Data warehouse-bA.
* [Az Azure Active Directory](https://review.docs.microsoft.com/en-us/azure/active-directory) hitelesíti a felhasználókat, amikor csatlakozik [Azure Databricks](https://azure.microsoft.com/en-us/services/databricks/). Ha szeretné készítünk egy adatkockát [Analysis Services](https://review.docs.microsoft.com/en-us/azure/analysis-services) alapján a modellen alapuló [Azure SQL Data Warehouse](https://review.docs.microsoft.com/en-us/azure/sql-data-warehouse) adatok sikerült használjuk aad-ben a Power bi-ban Analysis Services-kiszolgálóhoz való csatlakozáshoz. A Data Factory segítségével is az Azure AD egyszerű szolgáltatás vagy a Felügyeltszolgáltatás-identitás (MSI) az SQL Data Warehouse hitelesítéséhez.
* [Az Azure App Services](https://docs.microsoft.com/en-us/azure/app-service/app-service-web-overview), különösen [API-alkalmazás](https://azure.microsoft.com/en-us/services/app-service/api/) adatok külső felek számára, a szolgáltató rétegben tárolt adatok alapján is használható.

## <a name="alternatives"></a>Alternatív megoldások

![Képernyőfelvétel](media/architecture-diagram-realtime-analytics-vehicle-data2.png)

Általánosabb big Data típusú adatok folyamat végrehajtani, hogy más Azure komponenseket kell elhelyezheti. 
* A Stream betöltési rétegben, sikerült használjuk [az IoT Hub](https://azure.microsoft.com/en-us/services/iot-hub/) vagy [Eseményközpont](https://azure.microsoft.com/en-us/services/event-hubs/), helyett [HDInsight Kafka](https://docs.microsoft.com/en-us/azure/hdinsight/kafka/apache-kafka-introduction) betölteni az adatokat.
* Az átalakítási és analytics réteget, hogy hasznosíthatja [HDInsight Storm](https://docs.microsoft.com/en-us/azure/hdinsight/storm/apache-storm-overview), [HDInsight Spark](https://docs.microsoft.com/en-us/azure/hdinsight/spark/apache-spark-overview), vagy [Azure Stream Analytics](https://azure.microsoft.com/en-us/services/stream-analytics/)
* [Analysis Services](https://review.docs.microsoft.com/en-us/azure/analysis-services) szemantikai modellt biztosít az adatok számára. A rendszer teljesítménye is azt növekedhet, ha az adatok elemzésének. A modell Azure DW adatok alapján hozhat létre.

## <a name="considerations"></a>Megfontolandó szempontok

Ebben az architektúrában a technológiák a méretezési csoport szükség feldolgozni az eseményeket, az SLA-t a szolgáltatások, a Költségkezelés és a könnyű kezelés összetevők alapján lettek kiválasztva.
* Felügyelt [HDInsight Kafka](https://docs.microsoft.com/en-us/azure/hdinsight/kafka/apache-kafka-introduction) tartalmaz egy 99,9 %-os SLA-t integrálva van az Azure Managed Disks
* [Az Azure Databricks](https://azure.microsoft.com/en-us/services/databricks/) a kezdetektől fogva a teljesítmény- és költséghatékonyságot a felhőben van optimalizálva. A Databricks futtatókörnyezete az Apache Spark számítási feladatokat, amelyek a teljesítmény növelése és az Azure-ban, akár 10 – 100 x csökkentheti a költségeket ad hozzá több főbb képességei többek között:
* [Az Azure Databricks](https://azure.microsoft.com/en-us/services/databricks/) szorosan integrálódik az Azure-adatbázisok, és tárolja: [Azure SQL Data Warehouse](https://review.docs.microsoft.com/en-us/azure/sql-data-warehouse), [Azure cosmos DB](https://azure.microsoft.com/en-us/services/cosmos-db/), [Azure Data Lake Storage](https://azure.microsoft.com/en-us/services/storage/data-lake-storage/), és [Azure Blob Storage](https://azure.microsoft.com/en-us/services/storage/blobs/)
    * Automatikus méretezés és automatikus – Bezárás automatikusan a költségek csökkentése érdekében a Spark-fürtök számára.
    * Teljesítményoptimalizálás, beleértve a gyorsítótárazás, az indexelés és speciális lekérdezés optimalizálása, amelynek a jobb teljesítmény érdekében, akár 10 – 100 x hagyományos Apache Spark-telepítések felhőbeli vagy helyszíni környezetben keresztül.
    * Integráció a [Azure Active Directory](https://review.docs.microsoft.com/en-us/azure/active-directory) is futtathatja a teljes Azure-alapú megoldások használatával [Azure Databricks](https://azure.microsoft.com/en-us/services/databricks/).
    * [Az Azure Databricks](https://azure.microsoft.com/en-us/services/databricks/) szerepköralapú hozzáférés lehetővé teszi, hogy a notebookok, fürtök, feladatok és részletes felhasználói engedélyeket.
    * Nagyvállalati szintű SLA-kat is tartalmaz
* [Azure cosmos DB](https://azure.microsoft.com/en-us/services/cosmos-db/) a Microsoft globálisan elosztott, többmodelles adatbázisa. Az Azure Cosmos DB fejlesztésének középpontjában az alapoktól fogva a globális elosztás és a horizontális skálázás áll. Kulcsrakész globális elosztást kínál tetszőleges mennyiségű Azure-régióra, mert bárhol is vannak a felhasználói, gördülékenyen skálázza és replikálja az adatokat. Világszerte rugalmasan skálázhatja a teljesítményt és a tárolást, és csak azért a teljesítményért és tárolásért kell fizetnie, amire szüksége van.
* Az SQL Data Warehouse a nagymértékben párhuzamos feldolgozási architektúra méretezhetőséget és teljesítményt biztosít.
* [Az Azure SQL Data Warehouse](https://review.docs.microsoft.com/en-us/azure/sql-data-warehouse) garantált SLA-kkal és ajánlott eljárások a magas rendelkezésre állás eléréséhez.
* Elemzés a tevékenységet, alacsony, ha a vállalat méretezhetők [Azure SQL Data Warehouse](https://review.docs.microsoft.com/en-us/azure/sql-data-warehouse) igény szerinti, csökkentését, vagy akár a költségek csökkentése a számítás felfüggesztése.
* A [Azure SQL Data Warehouse](https://review.docs.microsoft.com/en-us/azure/sql-data-warehouse) biztonsági modellt biztosít a kapcsolat biztonsága, hitelesítés és engedélyezés az Azure AD-n keresztül vagy az SQL Server-hitelesítés és titkosítás.


## <a name="pricing"></a>Díjszabás

Felülvizsgálat [Azure Databricks díjszabása](https://azure.microsoft.com/en-us/pricing/details/databricks/), [Azure HDInsight díjszabása](https://azure.microsoft.com/en-us/pricing/details/hdinsight/), [díjszabása a minta egy adatraktározási forgatókönyv](https://azure.com/e/b798fb70c53e4dd19fdeacea4db78276) keresztül az Azure díjkalkulátorát. Állítsa be az értékek megtekintéséhez, hogyan érinti az igényeinek a költségeit.
* [Az Azure HDInsight](https://docs.microsoft.com/en-us/azure/hdinsight/) egy teljes körűen felügyelt felhőszolgáltatás, amely segítségével könnyen, gyorsan és költséghatékonyan folyamat nagy mennyiségű adat
* [Az Azure Databricks](https://azure.microsoft.com/en-us/services/databricks/) számos két különálló számítási [Virtuálisgép-példányok](https://azure.microsoft.com/en-us/pricing/details/databricks/#instances) munkafolyamattal szabott – adatfeldolgozási számítási segítségével hozhatnak létre és futtathatnak feladatokat, és az adatok az adattervezők egyszerűen Analytics munkafolyamatokra megismerését, megjelenítése, módosítására és megoszthatják az adatokat és elemzéseket-interaktív módon adatszakértők számára.
* [Azure cosmos DB](https://azure.microsoft.com/en-us/services/cosmos-db/) számjegy-ezredmásodperces a világ bármely pontján található 99 százalékon garantálja, kínál [többszörös, jól definiált konzisztenciamodellekkel](https://docs.microsoft.com/en-us/azure/cosmos-db/consistency-levels) finomhangolása a teljesítmény és a magas garanciák rendelkezésre állást biztosít többkiszolgálós képességekkel – az iparág vezető átfogó [szolgáltatói szerződések](https://azure.microsoft.com/en-us/support/legal/sla/cosmos-db/) (SLA).
* [Az Azure SQL Data Warehouse](https://azure.microsoft.com/pricing/details/sql-data-warehouse/gen2/) lehetővé teszi, hogy egymástól függetlenül méretezheti a számítási és tárolási szintek. A számítási erőforrások számlázása óránként történik, és igény szerint ezeket az erőforrásokat szüneteltetheti vagy méretezhetők. Tárolási erőforrások számlázása terabájt, így növeli a költségeket, akkor több adatot képes feldolgozni.
* [Analysis Services](https://azure.microsoft.com/pricing/details/analysis-services/) fejlesztői, alapszintű és standard szinten érhető el. Példányok díjszabása alapján lekérdezésfeldolgozó egységek (qpu-ra) és a rendelkezésre álló memória. Hogy a költség alacsonyabb, a lekérdezések feldolgozásához, hogy mennyi adatot futtatja, a lehető legkevesebb, és milyen gyakran futnak.
* [Power bi-ban](https://powerbi.microsoft.com/pricing/) különböző termék lehetőséget eltérő követelmények vonatkoznak. [Power BI Embedded](https://azure.microsoft.com/pricing/details/power-bi-embedded/) beágyazásához Power BI funkcióit belül az alkalmazások Azure-alapú lehetőséget biztosít. A Power BI Embedded-példány a fenti díjszabási mintát tartalmazza.

## <a name="next-steps"></a>További lépések
* Tekintse át a [valós idejű elemzési](https://azure.microsoft.com/en-us/solutions/architecture/real-time-analytics/) hivatkozik, amely tartalmazza a folyamatot a folyamat a big data architektúra.
* Tekintse át a [big data bővített analitikához](https://azure.microsoft.com/en-gb/solutions/architecture/advanced-analytics-on-big-data/) a különböző azure-összetevőkre épülő betekintést nyerhet a referencia-architektúra segíthetnek a big Data típusú adatok folyamat létrehozása.
* Olvassa el a [valós idejű feldolgozás](https://docs.microsoft.com/en-us/azure/architecture/data-guide/big-data/real-time-processing) különböző Azure-összetevőket gyors képet kaphat az Azure-dokumentáció segít valós idejű adatstreamek feldolgozására.
* Az adatfolyamatok, adattárházak, online elemzésfeldolgozási (OLAP) és big Data típusú adatok átfogó architekturális útmutatást találhat a [Azure-Adatarchitektúrához](https://review.docs.microsoft.com/en-us/azure/architecture/data-guide/).
* Ismerje meg az Azure-alapú megoldások bevált gyakorlatokat a [Azure Architecture Centert](https://review.docs.microsoft.com/en-us/azure/architecture/).
