---
title: Az adattárházak és adatpiacok
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.openlocfilehash: 6679ff620ca9e64036c02fce38608de38c57df93
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54482171"
---
# <a name="data-warehousing-and-data-marts"></a>Az adattárházak és adatpiacok

A data warehouse-bA integrált adatok egy vagy több különféle forrásból, egy központi, szervezeti, relációs adattár számos vagy minden területekről esetében. A korábbi adattárházak aktuális és előzménynézeteket adatokat tárolhatnak, és különböző módon használják a jelentéskészítés és az adatok elemzését.

![Az adattárház az Azure-ban](../images/data-warehousing.png)

Adatok áthelyezése a data warehouse-ba, akkor ki kell olvasni fontos üzleti adatokat tartalmazó különböző forrásokból származó rendszeres időközönként. Helyez át az adatokat, mert azt is formázott, tisztítani, érvényesítve, foglalja össze, és újra. Azt is megteheti az adatok tárolhatók a legalacsonyabb szintű részletességi összesített nézetekkel, az adatraktár jelentéskészítési megadott. Mindkét esetben az adatraktárban használt jelentések, elemzés és üzleti intelligenciára épülő (BI) eszközökkel fontos üzleti döntéseket képező adatokat állandó tároló adható válik.

## <a name="data-marts-and-operational-data-stores"></a>Adatpiacainak és működési adatokat tárolja.

Nagy mennyiségű adat kezelése bonyolult, és kevésbé gyakori, hogy rendelkezik egy adattárházzal, amely a teljes cégen belül minden adatot jelöli válik. Ehelyett hozzon létre a szervezetek a kisebb, célzottabb data warehouse-adattárházak, nevű *adatpiacainak*, tegye elérhetővé a kívánt adatokat, elemzési célokra. Egy vezénylési folyamat feltölti a egy működési adattárban tárolt adatok az adatpiac jelenti. A működésiadat-tároló a forrásrendszerben tranzakciós és az Adatközpont közötti közvetítőként működik. A működésiadat-store által kezelt adatok a forrásrendszerben tranzakciós adatok megtisztított verziója, és általában a korábbi, a data warehouse-bA vagy az Adatközpont által kezelt adatok egy részét.

## <a name="when-to-use-this-solution"></a>Ez a megoldás használata

Válasszon egy adattárházat, amikor a be kell kapcsolnia a nagy mennyiségű adat az operációs rendszerek olyan formátumra, amely könnyen áttekinthető, az aktuális és pontos. Adattárházakkal, amelyeknél nem kell ugyanazon tömör adatok szerkezetének az lehet, hogy használja az operatív/OLTP-adatbázisokban. Oszlopnevek, érthető legyen az üzleti felhasználók és az elemzők, adatkapcsolatok leegyszerűsítése érdekében a séma-erdővé és kialakíthattunk egy több táblát is használhatja. Ezek a lépések elősegítik az útmutató a felhasználók, akik kell ad hoc jelentések létrehozása vagy jelentéseket hozhat létre, és az adatok elemzése BI rendszerekben, egy adatbázis-rendszergazda (DBA) vagy az adatok fejlesztői segítsége nélkül.

Fontolja meg, amikor szüksége van a megfelelő teljesítmény biztosítása érdekében tranzakció forrásrendszerekből különválasztásában előzményadatok adattárház segítségével. Adattárházak megkönnyítik a korábbi adatok eléréséhez több helyről azáltal, hogy egy központi helyen közös formátumok, a közös kulcsok, a közös adatmodellt és a közös hozzáférési módszerek használatával.

Adattárházak jelentéseket futtat a forrásrendszerben tranzakció képest gyorsabb jelentéskészítés eredményez, olvasási hozzáférés vannak optimalizálva. Emellett a data warehouse-adattárházak a következő előnyöket nyújtják:

- Több forrásból származó összes előzményadatok is tárolása és elérése a data warehouse-ból, az egyetlen hiteles forrásaként.
- Adatminőség is javítható az adatok törlése, mivel a data warehouse-ba való importálás, pontosabb adatokat, valamint konzisztens kódok és leírások alapján.
- Jelentéskészítő eszközökkel ne befolyásolják a tranzakciós forrásrendszerektől lekérdezés-feldolgozási ciklusokat. A data warehouse lehetővé teszi, hogy a tranzakciós rendszer írási kezelése során az adatraktár eleget tesz az olvasási kérések többségét túlnyomórészt összpontosíthat.
- Egy adatraktár összevonni az adatokat a másik szoftver segítségével.
- Adatok adatbányászati eszközök segíthetnek rejtett minták használatával automatikus módszerek az warehouse-ban tárolt adatokon.
- Adattárházak megkönnyítik a biztonságos hozzáférést nyújtanak a jogosult felhasználók számára, míg másoknak való hozzáférés korlátozása. Hiba esetén nem kell hozzáférést az üzleti felhasználók a forrásadatokat, kiküszöbölve egy éles tranzakció egy vagy több rendszert érő potenciális támadásként értelmezhetők.
- Adattárházak könnyebben hozhassanak létre, üzleti intelligencia megoldásokat építhetnek az adatokat, például [OLAP-kockák](online-analytical-processing.md).

## <a name="challenges"></a>Problémák

A data warehouse-adattárházat az üzleti igényeinek megfelelően konfigurálása használata is néhány a következő kihívásoknak:

- Véglegesítő megfelelően modellezheti az üzleti fogalmak szükséges időt. Ez a fontos lépés, mivel data warehouse-adattárházak készített, ahol a koncepció leképezés a projekt a többi meghajtók információkat. Ebbe beletartozik az szabványosításával üzleti feltételeket és a közös formátumban (például a pénznem és dátum), és úgy, hogy az üzleti felhasználók is könnyen rendszerezhessen, de továbbra is biztosítja az adatok összesítések és a kapcsolatok pontosságát a séma átalakítását.

- Tervezés és az adatok előkészítése beállítása. Szempontok közé tartoznak, hogyan másolhat adatokat a forrás tranzakciós rendszerből, a data warehouse-ba, és a korábbi adatok áthelyezése az operatív adattárak és a warehouse-bA.

- Fenntartása, illetve adatokat minőség javítása tisztítási, mert az adatok az adattárház importálni.

## <a name="data-warehousing-in-azure"></a>Az adattárház az Azure-ban

Az Azure-ban előfordulhat, hogy egy vagy több adatforráshoz, az ügyfél tranzakciók, vagy a különböző részlegek által használt különböző üzleti alkalmazásokat. Egy vagy több hagyományosan tárolja ezeket az adatokat [OLTP](online-transaction-processing.md) adatbázisok. Az egyéb tárolási adathordozókon, például a hálózati megosztások, az Azure Storage-Blobokkal vagy a data lake sikerült őrizhető meg, az adatokat. Az adatraktár saját maga által, illetve például az Azure SQL Database egy relációs adatbázisban is kell tárolni az adatokat. Az analitikus adatok store réteg célja elemzési és jelentéskészítési eszközök ellen a data warehouse-bA vagy az Adatközpont által kiadott lekérdezések kielégítéséhez. Az Azure-ban ezt a képességet analitikai store érheti el az Azure SQL Data Warehouse, vagy az Azure HDInsight Hive vagy az interaktív lekérdezések használata. Emellett kell bizonyos szintű vezénylési rendszeres időközönként áthelyezése vagy adatok másolása az adattárolás a data warehouse, amely teheti meg az Azure HDInsight az Azure Data Factory vagy Oozie használatával.

Való megvalósítása a data warehouse-bA az Azure-ban, igényeitől függően több lehetőség áll rendelkezésre. A következő listákban megszakadnak a két kategóriába sorolhatók [szimmetrikus többprocesszoros feldolgozás](https://en.wikipedia.org/wiki/Symmetric_multiprocessing) (SMP) és [nagymértékben párhuzamos feldolgozási](https://en.wikipedia.org/wiki/Massively_parallel) (MPP).

SMP:

- [Azure SQL Database](/azure/sql-database/)
- [SQL Server virtuális gépen](/sql/sql-server/sql-server-technical-documentation)

MPP:

- [Az Azure Data warehouse-bA](/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is)
- [Apache Hive on HDInsight](/azure/hdinsight/hadoop/hdinsight-use-hive)
- [HDInsight interaktív lekérdezés (Hive LLAP)](/azure/hdinsight/interactive-query/apache-interactive-query-get-started)

Általános szabály, SMP-alapú adattárházakhoz vannak leginkább megfelelő kis, közepes méretű adatkészletek (akár a 4 – 100 TB-ot), amíg MPP big Data típusú adatok gyakran használják. Kis és közepes méretű és big Data típusú adatok között körülhatárolásához részben rendelkezik, hogy a szervezet definíció és -kompatibilis infrastruktúrával. (Lásd: [egy OLTP-adattár kiválasztásával](online-transaction-processing.md#scalability-capabilities).)

Túl az adatok mérete a számítási feladatok mintájának típusa nem valószínű, hogy nagyobb meghatározó tényező. Például összetett lekérdezések előfordulhat, hogy egy állapotáttelepítési pont megoldás túl lassú, és inkább egy MPP megoldásra van szüksége. MPP-alapú rendszerek valószínűleg írnak elő a rendszer teljesítményét adatok kis méretű lehet a feladatokat, elosztott és konszolidált csomópontok között. Ha a adatméretek már meghaladja az 1 TB-ot, és várhatóan folyamatosan nő, fontolja meg egy MPP-megoldás kiválasztása. Előfordulhat azonban, ha az adatok mérete kisebb, mint ez, de a számítási feladatok meghaladják a rendelkezésre álló erőforrások a SMP-megoldás, majd az MPP a legjobb megoldás.

Érhető el, vagy az adattárház tárolja az adatokat több adatforráshoz, beleértve például a data lake, sikerült származnak [Azure Data Lake Store](/azure/data-lake-store/). Egy videó-munkamenetet, amely összehasonlítja a különböző szintek MPP-szolgáltatások által használható az Azure Data Lake, lásd: [az Azure Data Lake és Azure Data warehouse-bA: Modern eljárásokat alkalmaz az alkalmazás](https://azure.microsoft.com/resources/videos/build-2016-azure-data-lake-and-azure-data-warehouse-applying-modern-practices-to-your-app/).

Állapotáttelepítési pont rendszerek egyetlen példánya egy relációs adatbázis-kezelő rendszer (CPU/memória/lemez) az összes erőforrások megosztása jellemzi. Az állapotáttelepítési pont rendszer vertikálisan felskálázhatja. A virtuális gépeken futó SQL Serverhez vertikálisan felskálázhatja a Virtuálisgép-méretet. Az Azure SQL Database skálázhatja a különböző szolgáltatási szint kiválasztásával.

MPP-rendszerek egyre nagyobb számítási csomópontok (amely saját Processzor, memória és i/o-alrendszerre) hozzáadásával kiterjeszthető. A kiszolgáló ekkor horizontális felskálázás szükség a több attól függően, a munkaterhelés vertikális felskálázása a fizikai korlátai vannak. MPP-megoldások azonban egy másik indexmezők miatt eltérések az lekérdezése, modellezés, particionálás, az adatok és egyéb tényezők egyedi, a párhuzamos feldolgozásra van szükség.

Amikor eldönti, mely SMP megoldást használja, lásd: [Azure SQL Database és SQL Server Azure virtuális gépeken közelebbről](/azure/sql-database/sql-database-paas-vs-sql-server-iaas#a-closer-look-at-azure-sql-database-and-sql-server-on-azure-vms).

Az Azure SQL Data Warehouse is használható a kis- és adatkészleteket, ahol a számítási feladat a számítási és memória-intenzív. További információ az SQL Data Warehouse-minták és gyakori alkalmazási esetei:

- [Az SQL Data Warehouse mintákat és kizárási mintákat](https://blogs.msdn.microsoft.com/sqlcat/2017/09/05/azure-sql-data-warehouse-workload-patterns-and-anti-patterns/)

- [Az SQL Data Warehouse betöltési minták és stratégiák](https://blogs.msdn.microsoft.com/sqlcat/2017/05/17/azure-sql-data-warehouse-loading-patterns-and-strategies/)

- [Adatok áttelepítése az Azure SQL Data Warehouse-ba](https://blogs.msdn.microsoft.com/sqlcat/2016/08/18/migrating-data-to-azure-sql-data-warehouse-in-practice/)

- [Az Azure SQL Data Warehouse közös ISV Alkalmazásminták](https://blogs.msdn.microsoft.com/sqlcat/2017/09/05/common-isv-application-patterns-using-azure-sql-data-warehouse/)

## <a name="key-selection-criteria"></a>Fontosabb kritériumok

Így szűkítheti, első lépésként a kérdések megválaszolása:

- Szeretne egy felügyelt szolgáltatás, hanem a saját kiszolgálók kezelése?

- Dolgozunk a rendkívül nagy méretű adatkészleteket vagy nagy mértékben összetett, hosszú ideig futó lekérdezéseket? Ha igen, érdemes lehet egy MPP lehetőséget.

- Egy nagy méretű adatkészlet je zdrojem dat strukturált vagy strukturálatlan? Strukturálatlan adatok szükség lehet a dolgozhatók fel a big data-környezetekben, például a Spark on HDInsight, az Azure Databricks, a Hive LLAP, a HDInsight vagy Azure Data Lake Analytics. Ezek mindegyikét szolgálhatnak ELT (kinyerési, betöltési, átalakítási) és az ETL (kinyerés, átalakítás, betöltés) motorokkal. A feldolgozott adatokat azok is kimeneti strukturált adatok, így könnyebben betöltése az SQL Data warehouse-bA vagy az egyéb lehetőségek közül. Strukturált adatok az SQL Data Warehouse egy optimalizált meghívta ultramagas szintű teljesítményt igénylő, nagy számítási igényű számítási feladatok a számítási teljesítményszint rendelkezik.

- Biztosan a korábbi adatok elkülönítése a jelenlegi, operatív adatok? Ha igen, válassza ki a beállítások ahol [vezénylési](../technology-choices/pipeline-orchestration-data-movement.md) megadása kötelező. Ezek (nagy erőforrásigényű) olvasásra optimalizált önálló adattárházakhoz, és külön múltbeli adatok tárolására a legalkalmasabbak.

- Szükség van integrálhatók több forrásból is, meghaladja az OLTP-adattárhoz adatok? Ha igen, fontolja meg a beállításokat, amelyek könnyen integrálhatók több adatforrást.

- Rendelkezik egy több-bérlős követelménynek? Ha igen, az SQL Data warehouse-ba, nem ideális ehhez a követelményhez. További információkért lásd: [SQL Data Warehouse mintákat és kizárási mintákat](https://blogs.msdn.microsoft.com/sqlcat/2017/09/05/azure-sql-data-warehouse-workload-patterns-and-anti-patterns/).

- Inkább relációs adattároló? Ha igen, szűkítenie a lehetőségeit a relációs adattároló rendelkező, de azt is vegye figyelembe, hogy egy eszköz, például a PolyBase nem relációs adattárak szükség esetén lekérdezni használhatja. Ha úgy dönt, hogy a polybase szolgáltatást akkor használja, azonban teljesítmény tesztek futtatásához a strukturálatlan adatok csoportjai a számítási feladatok számára.

- Van a valós idejű jelentéskészítési követelmények? Ha gyors lekérdezési válaszidőn az egyszeres beszúrásokat keskeny nagy mennyiségű a lehetőségek, amelyek valós idejű jelentéskészítést is támogatja.

- Nagyszámú egyidejű felhasználót és a kapcsolatok támogatása szükséges? Támogatja a több egyidejű felhasználók/kapcsolat lehetővé teszi számos tényezőtől függ.

  - Az Azure SQL Database, tekintse meg a [erőforráskorlátok dokumentált](/azure/sql-database/sql-database-resource-limits) szolgáltatásszint alapján.
  
  - Az SQL Server lehetővé teszi, hogy egy legfeljebb 32 767 felhasználói kapcsolatok. Ha egy virtuális gépen futó teljesítmény függ a virtuális gép méretének és egyéb tényezők.

  - Az SQL Data Warehouse esetében az egyidejű lekérdezéseket és egyidejű kapcsolatok vonatkozó korlátozások. További információkért lásd: [Egyidejűséggel és a számítási feladatok kezelésével az SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-develop-concurrency). Érdemes lehet az egymást kiegészítő szolgáltatásokat, például [Azure Analysis Services](/azure/analysis-services/analysis-services-overview), az SQL Data Warehouse korlátok adattömbökként való lekérését.

- Milyen típusú számítási feladatok rendelkezik? MPP-alapú adattárház-megoldások általában olyan analitikai, a batch-orientált munkaterhelések számára a leginkább megfelelő. Tranzakciós természetéből sok kis olvasási és írási műveletek vagy több sor soronként műveletet, a számítási feladatok esetén fontolja meg az SMP közül. Ez az útmutató egy kivétel, a feldolgozása egy HDInsight-fürtön, például a Spark Streaming, és a egy Hive-táblán belül az adatok tárolása az adatfolyam használata esetén.

## <a name="capability-matrix"></a>Képességmátrix

A következő táblázat összefoglalja a fő különbségeket, a képességek.

### <a name="general-capabilities"></a>Általános képességek

<!-- markdownlint-disable MD033 -->

| | Azure SQL Database | SQL Server (VM) | SQL Data Warehouse | Apache Hive hdinsight | Hive LLAP, a HDInsight |
| --- | --- | --- | --- | --- | --- | -- |
| A felügyelt szolgáltatás | Igen | Nem | Igen | Igen <sup>1</sup> | Igen <sup>1</sup> |
| Adatok előkészítése (data/előzményadatok másolatát tárolja) igényel | Nincs | Nem | Igen | Igen | Igen |
| Egyszerű integrálás több adatforráshoz | Nincs | Nem | Igen | Igen | Igen |
| Támogatja a számítás felfüggesztése | Nincs | Nem | Igen | Nem <sup>2</sup> | Nem <sup>2</sup> |
| Relációs adattároló | Igen | Igen |  Igen | Nem | Nincs |
| Valós idejű jelentéskészítést | Igen | Igen | Nem | Nem | Igen |
| Rugalmas biztonsági mentés visszaállítási pontok | Igen | Igen | Nem <sup>3</sup> | Igen <sup>4</sup> | Igen <sup>4</sup> |
| SMP/MPP | SMP | SMP | MPP | MPP | MPP |

<!-- markdownlint-enable MD033 -->

[1] manuális konfigurálás és a méretezés.

: [2] Ha nem szükséges, és újból létrehozza majd a HDInsight-fürtök lehet törölni. Egy külső adattár csatolása a fürt, az adatok megmaradnak, törölje a fürtöt. Azure Data Factory segítségével a fürt életciklusa automatizálása a számítási feladat feldolgozásához egy igény szerinti HDInsight-fürt létrehozásával, majd törli a feldolgozás befejeződése után.

[3] az SQL Data Warehouse egy adatbázist visszaállíthatja az összes elérhető visszaállítási pont az elmúlt 7 napon belül. A pillanatképek indítsa el a négy és nyolc óránként, és hét napig állnak rendelkezésre. Ha egy pillanatkép 7 napnál régebbi, lejárata előtt, és a visszaállítási pont már nem érhető el.

[4] fontolja meg egy [külső Hive-metaadattár](/azure/hdinsight/hdinsight-hadoop-provision-linux-clusters#use-hiveoozie-metastore) , amelyek biztonsági mentése és visszaállítása, igény szerint. Standard biztonsági mentési és helyreállítási lehetőségek, amelyek a alkalmazni a Blob Storage vagy a Data Lake Store az adatok, vagy harmadik féltől származó HDInsight biztonsági mentésre használható, és állítsa vissza a megoldásokat, mint például [Imanis adatok](https://azure.microsoft.com/blog/imanis-data-cloud-migration-backup-for-your-big-data-applications-on-azure-hdinsight/) nagyobb rugalmasságot és a könnyű használatra is használható.

### <a name="scalability-capabilities"></a>Skálázhatósági képességeket.

<!-- markdownlint-disable MD033 -->

| | Azure SQL Database | SQL Server (VM) |  SQL Data Warehouse | Apache Hive hdinsight | Hive LLAP, a HDInsight |
| --- | --- | --- | --- | --- | --- | -- |
| Redundáns regionális kiszolgálók magas rendelkezésre állás érdekében  | Igen | Igen | Igen | Nem | Nincs |
| Támogatja a lekérdezés horizontális felskálázás (elosztott lekérdezések)  | Nincs | Nem | Igen | Igen | Igen |
| A dinamikus méretezhetőség | Igen | Nincs | Igen <sup>1</sup> | Nincs | Nincs |
| Támogatja a memórián belüli gyorsítótárazáshoz, az adatok | Igen |  Igen | Nem | Igen | Igen |

[1] az SQL Data Warehouse lehetővé teszi az adattárházegységek (dwu-k) száma fokozottabban kisebbre vagy nagyobbra méretezhetők. Lásd: [kezelés számítási teljesítményt az Azure SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-manage-compute-overview).

<!-- markdownlint-enable MD033 -->

### <a name="security-capabilities"></a>Biztonsági képességek

<!-- markdownlint-disable MD033 -->

|                         |           Azure SQL Database            |  SQL Server virtuális gépen  | SQL Data Warehouse |   Apache Hive hdinsight    |    Hive LLAP, a HDInsight     |
|-------------------------|-----------------------------------------|-----------------------------------|--------------------|-------------------------------|-------------------------------|
|     Hitelesítés      | SQL / Azure Active Directory (Azure AD) | SQL / Azure AD / Active Directory |   SQL / Azure AD   | helyi / Azure AD <sup>1</sup> | helyi / Azure AD <sup>1</sup> |
|      Jogosultság      |                   Igen                   |                Igen                |        Igen         |              Igen              |       Igen <sup>1</sup>        |
|        Naplózás         |                   Igen                   |                Igen                |        Igen         |              Igen              |       Igen <sup>1</sup>        |
| Adat-titkosítás inaktív állapotban |            Igen <sup>2</sup>             |         Igen <sup>2</sup>          |  Igen <sup>2</sup>  |       Igen <sup>2</sup>        |       Igen <sup>1</sup>        |
|   Sorszintű biztonság    |                   Igen                   |                Igen                |        Igen         |              Nincs               |       Igen <sup>1</sup>        |
|   Támogatja a tűzfalak    |                   Igen                   |                Igen                |        Igen         |              Igen              |       Igen <sup>3</sup>        |
|  Dinamikus adatmaszkolás   |                   Igen                   |                Igen                |        Igen         |              Nincs               |       Igen <sup>1</sup>        |

<!-- markdownlint-enable MD033 -->

[1] használata szükséges egy [tartományhoz csatlakoztatott HDInsight-fürt](/azure/hdinsight/domain-joined/apache-domain-joined-introduction).

[2] transzparens adattitkosítási (TDE) segítségével az inaktív adatok titkosításához és visszafejtéséhez szükséges.

[3] támogatással [egy Azure virtuális hálózaton belüli](/azure/hdinsight/hdinsight-extend-hadoop-virtual-network).

További információ az data warehouse adatraktár biztonságossá tétele:

- [Az SQL Database-adatbázis védelme](/azure/sql-database/sql-database-security-overview#connection-security)

- [Egy SQL Data warehouse-adatbázis védelme](/azure/sql-data-warehouse/sql-data-warehouse-overview-manage-security)

- [Azure virtuális hálózat használatával Azure HDInsight kiterjesztése](/azure/hdinsight/hdinsight-extend-hadoop-virtual-network)

- [Vállalati szintű Hadoop-biztonság használatába a tartományhoz csatlakoztatott HDInsight-fürtök](/azure/hdinsight/domain-joined/apache-domain-joined-introduction)
