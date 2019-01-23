---
title: Gépjárművek valós idejű IoT-adatainak betöltése és feldolgozása
titleSuffix: Azure Example Scenarios
description: Valós idejű járműadatokat tölthet be és dolgozhat fel az IoT használatával.
author: msdpalam
ms.date: 09/12/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.custom: fasttrack, IoT
ms.openlocfilehash: 73f01d683b17facde59f9c917f00043773ec474b
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54486896"
---
# <a name="ingestion-and-processing-of-real-time-automotive-iot-data"></a>Gépjárművek valós idejű IoT-adatainak betöltése és feldolgozása

Ebben a példaforgatókönyvben összeállít egy valós idejű adatfeldolgozást és -feldolgozási adatfolyamat kell fogadni és feldolgozni az üzenetet az IoT-eszközök (az általános érzékelők) a big Data típusú adatok elemzési platform az Azure-ban. Járműtelemetria telematika adatfeldolgozást és -feldolgozási rendszerek a következők csatlakoztatott autó-megoldásokat hozhat létre. Ebben a forgatókönyvben az autó-telematika adatfeldolgozást és -feldolgozási rendszerek alapul. Azonban a kialakítási minták számos iparágban érzékelők segítségével felügyelheti és figyelheti a komplex rendszerek például okosépületek, kommunikációs, gyártási, kereskedelmi és egészségügyi iparágak szempontjából is.

Ez a példa bemutatja a valós idejű adatfeldolgozást és -feldolgozási adatfolyamat üzenetek járművek telepített IoT-eszközökről. Több ezer és üzenetek (vagy események) több millió IoT-eszközök és érzékelők által jönnek létre. Rögzítésével és elemzésével ezeket az üzeneteket, hogy visszafejteniük értékes információkat és megfelelő műveleteket. Például telematika eszközökkel felszerelt autók, ha az eszköz (IoT) üzenetek valós időben rögzíthetők a hétköznapi azt tudná járművek élő helyen figyelheti, megtervezhetik az útvonalakat optimalizált, nyújt segítséget az illesztőprogramok és iparágak telematika használatával kapcsolatos támogatási például az automatikus biztosítási.

Ebben a példában a bemutatóban az imagine egy autó gyártóvállalat cég, amely szeretne létrehozni egy valós idejű rendszer kell fogadni és feldolgozni üzenetek telematika eszközökről. A vállalati célok a következők:

- Betöltési, és valós idejű járművek érzékelőkről és eszközökről érkező adatok tárolására.
- Elemezze a jármű helyét és egyéb információkat a kibocsátott keresztül (például motor kapcsolatos érzékelők és a környezettel kapcsolatos érzékelők) érzékelők különböző típusú üzeneteket.
- Az egyéb alárendelt feldolgozáshoz, gyakorlatban hasznosítható elemzéseket biztosít elemzést a data Store (például objektuma forgatókönyvekben, biztosítási szervek is érdekelheti, mi történt során baleset stb.)

## <a name="relevant-use-cases"></a>Alkalmazási helyzetek

Egyéb alkalmazási helyzetek a következők:

- Járműtelemetria karbantartási emlékeztetők és riasztási.
- Helyalapú szolgáltatások a vehicle utasok (azaz SOS).
- Önálló (önálló vezetési) járművek.

## <a name="architecture"></a>Architektúra

![Képernyőfelvétel](media/architecture-realtime-analytics-vehicle-data1.png)

Egy tipikus big data típusú adatfeldolgozást folyamat végrehajtása a adatáramlás balról jobbra. A valós idejű big Data jellegű adatok feldolgozása folyamatban a áramlanak keresztül az adatok a megoldás a következő:

1. Az IoT adatforrásokból generált események, az üzenetek küldése a streamfeldolgozási streamrétegében keresztül az Azure HDInsight Kafka. HDInsight Kafka témakörökben konfigurálható ideig tárolja az adatstreameket.
2. A Kafka-fogyasztók, az Azure Databricks, fogadja az üzenetet a Kafka-témakörbe, valós időben feldolgozni az adatokat az üzleti logika alapján, és ezután elküldheti a szolgáltató a tárolási réteg.
3. Alsóbb rétegbeli tárolási szolgáltatásoktól, például az Azure Cosmos DB, Azure SQL Data warehouse-bA vagy az Azure SQL DB, lesz egy adatforrást a bemutató és műveleti réteg.
4. Üzleti elemzők a Microsoft Power BI segítségével raktározott adatok elemzéséhez. Más alkalmazások épülhetnek a kiszolgálórétegbe. Ha például tehetjük elérhetővé a külső felhasználásra szolgáltatási réteg adatokon alapuló API-k.

### <a name="components"></a>Összetevők

IoT-eszközök által létrehozott események (adatok vagy üzenetek) vannak betöltött, feldolgozása, és ott további elemzés, bemutatót és művelet, a következő Azure-összetevők használatával:

- [Az Apache Kafka on HDInsight](/azure/hdinsight/kafka/apache-kafka-introduction) a feldolgozási réteg van. A Kafka-témakörbe, a Kafka producer API használatával történő írja az adatokat.
- [Az Azure Databricks](/services/databricks) az Adatátalakítási és elemzési rétegben található. Databricks-jegyzetfüzeteket megvalósítása a Kafka-fogyasztók API-t az adatok beolvasása a Kafka-témakört.
- [Az Azure Cosmos DB](/services/cosmos-db), [Azure SQL Database](/azure/sql-database/sql-database-technical-overview), és a storage kiszolgálórétegbe, ahol az Azure Databricks adatösszekötők keresztül adatokat írni az Azure SQL Data Warehouse.
- [Az Azure SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is) egy elosztott rendszer tárolására és elemzésére a nagyméretű adathalmazok. A masszív párhuzamos feldolgozási (MPP) is teszi megfelelő a futó, nagy teljesítményű elemzési.
- [Power bi-ban](https://docs.microsoft.com/power-bi) adatok elemzése és elemzéseket oszthat meg üzleti elemzési eszközök együttese. Power BI lekérdezhesse az Analysis Servicesben tárolt szemantikai modellt, vagy közvetlenül lekérdezheti azt az SQL Data warehouse-bA.
- [Az Azure Active Directory (Azure AD)](/azure/active-directory) hitelesíti a felhasználókat, amikor csatlakozik [Azure Databricks](https://azure.microsoft.com/services/databricks). Ha szeretné készítünk egy adatkockát [Analysis Services](/azure/analysis-services) alapján a modellt az Azure SQL Data Warehouse-adatok alapján, sikerült használjuk aad-ben a Power bi-ban Analysis Services-kiszolgálóhoz való csatlakozáshoz. A Data Factory segítségével is az Azure AD egyszerű szolgáltatás vagy a Felügyeltszolgáltatás-identitás (MSI) az SQL Data Warehouse hitelesítéséhez.
- [Az Azure App Services](/azure/app-service/app-service-web-overview), különösen [API-alkalmazás](/services/app-service/api) adatok külső felek számára, a szolgáltató rétegben tárolt adatok alapján is használható.

## <a name="alternatives"></a>Alternatív megoldások

![Képernyőfelvétel](media/architecture-realtime-analytics-vehicle-data2.png)

Általánosabb big Data típusú adatok folyamat használata más Azure-összetevőket is megvalósítható.

- A stream betöltési réteg sikerült használjuk [az IoT Hub](https://azure.microsoft.com/services/iot-hub) vagy [Eseményközpont](https://azure.microsoft.com/services/event-hubs), helyett [HDInsight Kafka](/azure/hdinsight/kafka/apache-kafka-introduction) betölteni az adatokat.
- Az átalakítási és elemzési réteg sikerült használjuk [HDInsight Storm](/azure/hdinsight/storm/apache-storm-overview), [HDInsight Spark](/azure/hdinsight/spark/apache-spark-overview), vagy [Azure Stream Analytics](https://azure.microsoft.com/services/stream-analytics).
- [Analysis Services](/azure/analysis-services) szemantikai modellt biztosít az adatok számára. A rendszer teljesítménye is azt növekedhet, ha az adatok elemzésének. A modell Azure DW adatok alapján hozhat létre.

## <a name="considerations"></a>Megfontolandó szempontok

Ebben az architektúrában a technológiák a méretezési csoport szükség feldolgozni az eseményeket, az SLA-t a szolgáltatások, a Költségkezelés és a könnyű kezelés összetevők alapján lettek kiválasztva.

- Felügyelt [HDInsight Kafka](/azure/hdinsight/kafka/apache-kafka-introduction) tartalmaz egy 99,9 %-os SLA-t integrálva van az Azure Managed Disks
- [Az Azure Databricks](/azure/azure-databricks/what-is-azure-databricks) a kezdetektől fogva a teljesítmény- és költséghatékonyságot a felhőben van optimalizálva. A Databricks futtatókörnyezete az Apache Spark számítási feladatokat, amelyek a teljesítmény növelése és az Azure-ban, akár 10 – 100 x csökkentheti a költségeket ad hozzá több főbb képességei többek között:
- Az Azure Databricks mélyen integrálható az Azure-adatbázisok és a tárolók: [Az Azure SQL Data Warehouse](/azure/sql-data-warehouse), [az Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db), [az Azure Data Lake Storage](https://azure.microsoft.com/services/storage/data-lake-storage), és [Azure Blob Storage](https://azure.microsoft.com/services/storage/blobs)
  - Automatikus skálázás és autotermination a Spark-fürtök automatikusan csökkentheti minimálisra a költségeket.
  - Teljesítményoptimalizálás, beleértve a gyorsítótárazás, az indexelés és speciális lekérdezés optimalizálása, amelynek a jobb teljesítmény érdekében, akár 10 – 100 x hagyományos Apache Spark-telepítések felhőbeli vagy helyszíni környezetben keresztül.
  - Az integráció az Azure Active Directoryval lehetővé teszi, hogy teljes Azure-alapú megoldásokat futtasson az Azure Databricks használatával.
  - Az Azure Databricks szerepköralapú hozzáférés lehetővé teszi, hogy a notebookok, fürtök, feladatok és részletes felhasználói engedélyeket.
  - Nagyvállalati szintű SLA-kat tartalmaz.
- Az Azure Cosmos DB a Microsoft globálisan elosztott többmodelles adatbázisa. Az Azure Cosmos DB fejlesztésének középpontjában az alapoktól fogva a globális elosztás és a horizontális skálázás áll. Kulcsrakész globális elosztást kínál tetszőleges mennyiségű Azure-régióra, mert bárhol is vannak a felhasználói, gördülékenyen skálázza és replikálja az adatokat. Világszerte rugalmasan skálázhatja a teljesítményt és a tárolást, és csak azért a teljesítményért és tárolásért kell fizetnie, amire szüksége van.
- Az SQL Data Warehouse a nagymértékben párhuzamos feldolgozási architektúra méretezhetőséget és teljesítményt biztosít.
- Az Azure SQL Data Warehouse garantált SLA-kkal rendelkezik, és ajánlott eljárások a magas rendelkezésre állás elérésének.
- Elemzés a tevékenységet, alacsony, amikor a vállalat méretezheti az Azure SQL Data Warehouse igény szerinti, csökkentése, vagy akár az alacsonyabb költségek a számítás felfüggesztése.
- Az Azure SQL Data Warehouse biztonsági modellt biztosít a kapcsolat biztonsága, hitelesítés és engedélyezés az Azure AD-n keresztül vagy az SQL Server-hitelesítés és titkosítás.

## <a name="pricing"></a>Díjszabás

Felülvizsgálat [Azure Databricks díjszabása](https://azure.microsoft.com/pricing/details/databricks), [Azure HDInsight díjszabása](https://azure.microsoft.com/pricing/details/hdinsight), [díjszabása a minta egy adatraktározási forgatókönyv](https://azure.com/e/b798fb70c53e4dd19fdeacea4db78276) keresztül az Azure díjkalkulátorát. Állítsa be az értékek megtekintéséhez, hogyan érinti az igényeinek a költségeit.

- [Az Azure HDInsight](/azure/hdinsight) egy teljes körűen felügyelt felhőszolgáltatás, amely segítségével könnyen, gyorsan és költséghatékonyan folyamat nagy mennyiségű adat
- [Az Azure Databricks](https://azure.microsoft.com/services/databricks) számos két különálló számítási [Virtuálisgép-példányok](https://azure.microsoft.com/pricing/details/databricks/#instances) munkafolyamattal szabott – adatfeldolgozási számítási segítségével hozhatnak létre és futtathatnak feladatokat, és az adatok az adattervezők egyszerűen Analytics munkafolyamatokra megismerését, megjelenítése, módosítására és megoszthatják az adatokat és elemzéseket-interaktív módon adatszakértők számára.
- [Az Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db) számjegy-ezredmásodperces a világ bármely pontján található 99 százalékon garantálja, kínál [többszörös, jól definiált konzisztenciamodellekkel](/azure/cosmos-db/consistency-levels) finomhangolása a teljesítmény és a magas garanciák rendelkezésre állást biztosít többkiszolgálós képességekkel – az iparág vezető átfogó [szolgáltatói szerződések](https://azure.microsoft.com/support/legal/sla/cosmos-db) (SLA).
- [Az Azure SQL Data Warehouse](https://azure.microsoft.com/pricing/details/sql-data-warehouse/gen2) lehetővé teszi, hogy egymástól függetlenül méretezheti a számítási és tárolási szintek. A számítási erőforrások számlázása óránként történik, és igény szerint ezeket az erőforrásokat szüneteltetheti vagy méretezhetők. Tárolási erőforrások számlázása terabájt, így növeli a költségeket, akkor több adatot képes feldolgozni.
- [Analysis Services](https://azure.microsoft.com/pricing/details/analysis-services) fejlesztői, alapszintű és standard szinten érhető el. Példányok díjszabása alapján lekérdezésfeldolgozó egységek (qpu-ra) és a rendelkezésre álló memória. Hogy a költség alacsonyabb, a lekérdezések feldolgozásához, hogy mennyi adatot futtatja, a lehető legkevesebb, és milyen gyakran futnak.
- [Power bi-ban](https://powerbi.microsoft.com/pricing) különböző termék lehetőséget eltérő követelmények vonatkoznak. [Power BI Embedded](https://azure.microsoft.com/pricing/details/power-bi-embedded) beágyazásához Power BI funkcióit belül az alkalmazások Azure-alapú lehetőséget biztosít. A Power BI Embedded-példány a fenti díjszabási mintát tartalmazza.

## <a name="next-steps"></a>Következő lépések

- Tekintse át a [valós idejű elemzési](https://azure.microsoft.com/solutions/architecture/real-time-analytics) hivatkozik, amely tartalmazza a folyamatot a folyamat a big data architektúra.
- Tekintse át a [big data bővített analitikához](https://azure.microsoft.com/solutions/architecture/advanced-analytics-on-big-data) a különböző azure-összetevőkre épülő betekintést nyerhet a referencia-architektúra segíthetnek a big Data típusú adatok folyamat létrehozása.
- Olvassa el a [valós idejű feldolgozás](/azure/architecture/data-guide/big-data/real-time-processing) különböző Azure-összetevőket gyors képet kaphat az Azure-dokumentáció segít valós idejű adatstreamek feldolgozására.
- Az adatfolyamatok, adattárházak, online elemzésfeldolgozási (OLAP) és big Data típusú adatok átfogó architekturális útmutatást találhat a [Azure-Adatarchitektúrához](/azure/architecture/data-guide).
