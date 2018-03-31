---
title: "Adatraktár kiválasztása"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 9cb3d4d0196b02da76d85c7f7f0e4a2a69d531e9
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-a-data-warehouse-in-azure"></a>Adatraktár kiválasztása az Azure-ban

Adatraktár tárháza központi, szervezeti, relációs egy vagy több különböző forrásokból származó integrált adatok. Ez a témakör összehasonlítja az adatraktárak az Azure-ban.

> [!NOTE]
> Adatraktár használatával kapcsolatos további információkért lásd: [adatraktározással és adatpiacainak](../scenarios/data-warehousing.md).

## <a name="what-are-your-options-when-choosing-a-data-warehouse"></a>Mik azok a beállítások egy adatraktár kiválasztásakor?

Adatraktár megvalósításának Azure igényeitől függően több lehetőség áll rendelkezésre. A következő listákban bontottuk két kategóriába sorolhatók [szimmetrikus többprocesszoros feldolgozás](https://en.wikipedia.org/wiki/Symmetric_multiprocessing) (SMP) és [nagymértékben párhuzamos feldolgozási](https://en.wikipedia.org/wiki/Massively_parallel) (MPP). 

SMP:

- [Azure SQL Database](/azure/sql-database/)
- [SQL Server virtuális gépen](/sql/sql-server/sql-server-technical-documentation)

MPP:

- [Az Azure Data warehouse-bA](/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is)
- [Apache Hive hdinsight](/azure/hdinsight/hadoop/hdinsight-use-hive)
- [A HDInsight (Hive LLAP) interaktív lekérdezés](/azure/hdinsight/interactive-query/apache-interactive-query-get-started)

Általános szabályként SMP-alapú adatraktárakra vannak leginkább megfelelő a kis közepes méretű adatkészletek (akár a 4-100 TB), míg az MPP gyakran a big data. Kis/közepes és nagy adatok körülhatárolásához részben a szervezet definition és támogató infrastruktúra rendelkezik. (Lásd: [kiválasztása egy OLTP adattároló](oltp-data-stores.md#scalability-capabilities).) 

Adatok mérete meghaladja a munkaterhelés minta típusa valószínű, hogy nagyobb meghatározó tényező lehet. Például összetett lekérdezések előfordulhat, hogy túl lassú SMP megoldás, és ehelyett MPP megoldásra van szüksége. Az MPP-alapú rendszerek valószínűleg maximálásához kis adatméretek feladatok elosztott és -csomópontokon keresztüli konszolidált köszönhetően a rendszer teljesítményét. Ha az adatok mérete már haladhatja meg az 1 TB-os és várhatóan folyamatosan nő, fontolja meg egy MPP megoldás kiválasztása. Előfordulhat azonban, ha az adatok mérete kisebb, mint ez, de a munkaterhelések túllépte a rendelkezésre álló erőforrások a SMP megoldás, majd MPP a legjobb lehetőség.

Az adatok érhető el, vagy az adatraktár által tárolt adatforrások, data lake, például számos származhat [Azure Data Lake Store](/azure/data-lake-store/). Videó munkamenetet, amely összehasonlítja a különböző szintjeiről MPP szolgáltatások, Azure Data Lake használhat, lásd: [Azure Data Lake és az Azure Data Warehouse: alkalmazása Modern gyakorlatokat, amelyekkel az alkalmazás](https://azure.microsoft.com/resources/videos/build-2016-azure-data-lake-and-azure-data-warehouse-applying-modern-practices-to-your-app/).

Egyetlen példány futhat egy relációs adatbázis-kezelő rendszer (Processzor/memória/lemez) erőforrások megosztása SMP rendszerek jellemző. Az állapotáttelepítési pont rendszer költenie. Az SQL Server egy virtuális gépen költenie a Virtuálisgép-méretet. Az Azure SQL Database esetén legfeljebb egy másik szolgáltatási réteg kiválasztásával. 

Az MPP rendszerek kiterjeszthető további számítási csomópontok (amelyek a saját Processzor, memória- és i/o-alrendszereket) hozzáadásával. A kiszolgáló, amikor kiterjesztése szükség a több attól függően, hogy a munkaterhelés vertikális felskálázásával fizikai korlátozások is. MPP megoldások azonban egy másik skillset miatt eltérések az lekérdezése, modellezési, particionálás, az adatok és más tényezők kizárólag a párhuzamos feldolgozást igényel. 

Amikor arról dönt, hogy melyik SMP megoldást használni, lásd: [Azure SQL Database és az Azure virtuális gépeken futó SQL Server részletes bemutatása](/azure/sql-database/sql-database-paas-vs-sql-server-iaas#a-closer-look-at-azure-sql-database-and-sql-server-on-azure-vms). 

Az SQL Data Warehouse is használható kis és közepes méretű adatkészletekhez, ahol a munkaterhelés-e a számítási és memóriaigényes. További tájékoztatást talál az SQL Data Warehouse mintákat és a gyakori helyzetek:

- [SQL Data Warehouse mintákat és víruskereső minták](https://blogs.msdn.microsoft.com/sqlcat/2017/09/05/azure-sql-data-warehouse-workload-patterns-and-anti-patterns/)
- [Az SQL Data Warehouse mintákat és stratégiák betöltése](https://blogs.msdn.microsoft.com/sqlcat/2017/05/17/azure-sql-data-warehouse-loading-patterns-and-strategies/)
- [Adatok áttelepítése az Azure SQL Data Warehouse-ba](https://blogs.msdn.microsoft.com/sqlcat/2016/08/18/migrating-data-to-azure-sql-data-warehouse-in-practice/)
- [Az Azure SQL Data Warehouse közös ISV alkalmazás minták](https://blogs.msdn.microsoft.com/sqlcat/2017/09/05/common-isv-application-patterns-using-azure-sql-data-warehouse/)

## <a name="key-selection-criteria"></a>Kulcs kiválasztási feltételek

A választási lehetőségek szűkítéséhez indítása ezen kérdések megválaszolásával:

- A saját kiszolgálóit kezelése helyett egy felügyelt szolgáltatás van szüksége?

- Dolgozunk a rendkívül nagy adatkészletek vagy rendkívül összetett, hosszan futó lekérdezések? Ha igen, érdemes lehet egy MPP lehetőséget. 

- A nagy adatkészletet, a rendszer az adatforrás strukturált vagy strukturálatlan? Strukturálatlan adatok kell feldolgozni a Spark on HDInsight, Azure Databricks, LLAP Hive HDInsight, vagy az Azure Data Lake Analytics big Data típusú adatok környezetben. Mindegyik ELT (Extract, Load,-átalakítási) és az ETL (kinyerés, átalakítás, betöltés) motorok szolgálhatnak. Azok a kimenetre küldheti a feldolgozott adatok strukturált adatokká, így könnyebben betöltése az SQL Data Warehouse vagy az egyéb lehetőségek közül. A strukturált adatok az SQL Data Warehouse egy optimalizált nevű számítás, igen nagy teljesítményt igénylő számítási-igényes munkaterhelések teljesítményszinttel rendelkezik.

- Szeretné a jelenlegi, operatív adatok előzményadatait külön? Ha igen, a beállítások valamelyikét kell választani ahol [vezénylési](pipeline-orchestration-data-movement.md) szükséges. Azok önálló során nagy mennyiségű olvasási hozzáférés optimalizálva, és olyan környezethez a legalkalmasabb adattárként külön korábbi.

- Az OLTP adattároló túl, több forrásból származó adatok integrálni kell? Ha igen, vegye figyelembe a beállításokat, amelyek könnyen integrálható a több adatforrást. 

- Több vállalat kiszolgálása követelmény van? Ha igen, az SQL Data Warehouse ideális nem ehhez a követelményhez. További információkért lásd: [SQL Data Warehouse mintákat és víruskereső minták](https://blogs.msdn.microsoft.com/sqlcat/2017/09/05/azure-sql-data-warehouse-workload-patterns-and-anti-patterns/).

- Inkább relációs adattároló? Ha igen, szűkítenie a lehetőségeit a relációs adattároló rendelkező, de vegye figyelembe azt is, hogy, egy eszköz, például a PolyBase segítségével szükség esetén nem relációs adatokat kérdez. Ha úgy dönt, hogy a polybase szolgáltatást akkor használja, azonban teljesítmény tesztek futtatásához a strukturálatlan adatok beállítása a munkaterhelés számára.

- Rendelkezik a valós idejű jelentéskészítési követelmények? Ha szüksége van gyors lekérdezés válaszidőn a singleton Beszúrások, keskeny jelentős mennyiségű beállításairól a valós idejű jelentéskészítést támogató.

- Szüksége nagyszámú egyidejű felhasználók és kapcsolatok támogatásához? Az egyidejű felhasználók/csatlakozások száma támogatására képesek számos tényezőtől függ. 

    - Az Azure SQL Database, tekintse meg a [erőforrás korlátok dokumentált](/azure/sql-database/sql-database-resource-limits) a szolgáltatási réteg alapján. 
    
    - SQL Server lehetővé teszi, hogy egy legfeljebb 32 767 felhasználói kapcsolatok. Ha a virtuális gép fut, teljesítmény függ a Virtuálisgép-méretet és egyéb tényezőkhöz igazodva. 
    
    - Az SQL Data Warehouse vonatkozó korlátokat a párhuzamos lekérdezések és egyidejű kapcsolat van. További információkért lásd: [egyidejűségi és munkaterhelés-kezelés az SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-develop-concurrency). Érdemes például a kiegészítő szolgáltatásokat [Azure Analysis Services](/azure/analysis-services/analysis-services-overview), az SQL Data Warehouse korlátok megoldásához.

- Milyen típusú alkalmazások és szolgáltatások rendelkezik? Adatraktár MPP-alapú megoldások általában legmegfelelőbb analitikai, kötegelt alapú munkaterhelések. Ha a munkaterhelések tranzakciós természetüknél a sok kisméretű olvasási/írási műveletek vagy több soronként-műveletek, érdemes lehet az SMP közül. Egyetlen kivétel ez iránymutatás használata esetén adatfolyam feldolgozás esetén a HDInsight-fürtöt, például a Spark Streaming, és a Hive tábla adatainak tárolásához.

## <a name="capability-matrix"></a>Képesség mátrix

A következő táblázat összefoglalja a főbb változásai képességeit.

### <a name="general-capabilities"></a>Általános képességek

| | Azure SQL Database | SQL Server (VM) | SQL Data Warehouse | Apache Hive hdinsight | A HDInsight LLAP struktúra |
| --- | --- | --- | --- | --- | --- | -- |
| Van a felügyelt | Igen | Nem | Igen | Igen <sup>1</sup> | Igen <sup>1</sup> |
| Adatok előkészítése (tartalmazza a adatok/előzményadatokat másolatát) igényel | Nem | Nem | Igen | Igen | Igen |
| Könnyen integrálható a több adatforráshoz | Nem | Nem | Igen | Igen | Igen |
| Támogatja a számítási felfüggesztése | Nem | Nem | Igen | Nem <sup>2</sup> | Nem <sup>2</sup> |
| Relációs adattároló | Igen | Igen |  Igen | Nem | Nem |
| Valós idejű jelentéskészítést | Igen | Igen | Nem | Nem | Igen |
| Rugalmas biztonsági mentés visszaállítási pontok | Igen | Igen | Nem <sup>3</sup> | Igen <sup>4</sup> | Igen <sup>4</sup> |
| SMP/MPP | SMP | SMP | MPP | MPP | MPP |

[1] kézi konfigurálás és a méretezésről.

[2] HDInsight-fürtök törölheti, ha nincs szükség, és újból létrehozza majd. Egy külső adattároló csatlakoztatni a fürthöz, az adatok őrződnek meg a fürt törlésekor. Azure Data Factory használatával automatizálhatja a fürt életciklus feldolgozni az alkalmazások és szolgáltatások igény szerinti HDInsight-fürtök létrehozásával, majd törölje a feldolgozás befejezése után.

[3] az SQL Data Warehouse visszaállíthatja egy adatbázis bármely elérhető visszaállítási pont az utóbbi hét napban. A pillanatképek négy és nyolc óránként start és hét napja. Ha pillanatkép régebbi, mint hét nap, jár le, és a helyreállítási pont már nem érhető el.

[4] érdemes egy [külső Hive metaadattárhoz](/azure/hdinsight/hdinsight-hadoop-provision-linux-clusters#use-hiveoozie-metastore) , amelyek biztonsági mentése és visszaállítása szükség szerint. Szabványos biztonsági mentési és visszaállítási lehetőségekről a Blob Storage vagy a Data Lake Store az adatok vagy harmadik féltől származó HDInsight biztonsági mentés használható, és állítsa vissza a megoldások, például a [Imanis adatok](https://azure.microsoft.com/blog/imanis-data-cloud-migration-backup-for-your-big-data-applications-on-azure-hdinsight/) nagyobb rugalmasságot és könnyen használható is használható.

### <a name="scalability-capabilities"></a>Méretezhetőség képességek

| | Azure SQL Database | SQL Server (VM) |  SQL Data Warehouse | Apache Hive hdinsight | A HDInsight LLAP struktúra |
| --- | --- | --- | --- | --- | --- | -- |
| A magas rendelkezésre állás érdekében redundáns területi kiszolgálók  | Igen | Igen | Igen | Nem | Nem |
| Támogatja a lekérdezés kibővítési (elosztott lekérdezések)  | Nem | Nem | Igen | Igen | Igen |
| Dinamikus méretezhetőség (felskálázott)  | Igen | Nem | Igen <sup>1</sup> | Nem | Nem |
| Támogatja a memórián belüli gyorsítótárazáshoz, az adatok | Igen |  Igen | Nem | Igen | Igen |

[1] az SQL Data Warehouse lehetővé válik, hogy méretezést kívánó doanw módosításával adattárházegységek (dwu-k) száma. Lásd: [kezelése számítási teljesítményt az Azure SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-manage-compute-overview).

### <a name="security-capabilities"></a>Biztonsági képességei

| | Azure SQL Database | SQL Server virtuális gépen | SQL Data Warehouse | Apache Hive hdinsight | A HDInsight LLAP struktúra |
| --- | --- | --- | --- | --- | --- | -- |
| Hitelesítés  | SQL / Azure Active Directory (Azure AD) | SQL / Azure AD / Active Directory | SQL / Azure AD | helyi és az Azure AD <sup>1</sup> | helyi és az Azure AD <sup>1</sup> |
| Engedélyezés  | Igen | Igen | Igen | Igen | Igen <sup>1</sup> | Igen <sup>1</sup> |
| Naplózás  | Igen | Igen | Igen | Igen | Igen <sup>1</sup> | Igen <sup>1</sup> |
| Adat-titkosítás inaktív állapotban | Igen <sup>2</sup> | Igen <sup>2</sup> | Igen <sup>2</sup> | Igen <sup>2</sup> | Igen <sup>1</sup> | Igen <sup>1</sup> |
| Sorszintű biztonság | Igen | Igen | Igen | Nem | Igen <sup>1</sup> | Igen <sup>1</sup> |
| Támogatja a tűzfalak | Igen | Igen | Igen | Igen | Igen <sup>3</sup> | Igen <sup>3</sup> |
| Dinamikus adatmaszkolás | Igen | Igen | Igen | Nem | Igen <sup>1</sup> | Igen <sup>1</sup> |

[1] igényel a használata egy [HDInsight-fürt tartományhoz](/azure/hdinsight/domain-joined/apache-domain-joined-introduction).

[2] átlátszó Data Encryption (TDE) segítségével az inaktív adatok titkosításához és visszafejtéséhez szükséges.

[3] támogatással [egy Azure virtuális hálózaton belül használt](/azure/hdinsight/hdinsight-extend-hadoop-virtual-network).

További tudnivalók az adatraktár biztonságossá tétele:

* [Az SQL Database-adatbázis védelme](/azure/sql-database/sql-database-security-overview#connection-security)
* [Az SQL Data Warehouse adatbázis védelme](/azure/sql-data-warehouse/sql-data-warehouse-overview-manage-security)
* [Azure virtuális hálózat használatával Azure HDInsight kiterjesztése](/azure/hdinsight/hdinsight-extend-hadoop-virtual-network)
* [Vállalati szintű Hadoop biztonsági HDInsight-fürtökkel a tartományhoz](/azure/hdinsight/domain-joined/apache-domain-joined-introduction)

