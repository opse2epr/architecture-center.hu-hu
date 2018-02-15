---
title: "Olyan adatok tárolási technológia kiválasztása"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: a51c8d990818e0b0efb04dd7686b065e5b98338a
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-a-big-data-storage-technology-in-azure"></a>A big Data típusú adatok tárolási technológia kiválasztása az Azure-ban

Ez a témakör összehasonlítja a big data-megoldások adatok tárolási lehetőségei &mdash; pontosabban adatok tárolási tömeges adatfeldolgozást és kötegfeldolgozási, és nem a [analitikai adattárolókhoz](./analytical-data-stores.md) vagy [valós idejű adatfolyam-továbbítási adatfeldolgozást](./real-time-ingestion.md)

## <a name="what-are-your-options-when-choosing-data-storage-in-azure"></a>Mik azok a beállítások adattárolás kiválasztásakor az Azure-ban?

Az Azure, az adatok bevitele igényeitől függően több lehetőség áll rendelkezésre:

**A File storage**

- [Az Azure Storage blobs szolgáltatásban](/azure/storage/blobs/storage-blobs-introduction)
- [Azure Data Lake Store](/azure/data-lake-store/)

**NoSQL-adatbázisok**

- [Azure Cosmos DB](/azure/cosmos-db/)
- [A HDInsight HBase](http://hbase.apache.org/)

## <a name="azure-storage-blobs"></a>Azure Storage blobs

Az Azure Storage egy magas rendelkezésre álló, biztonságos, tartós, méretezhető és redundáns egy felügyelt tároló szolgáltatás. A karbantartást és a kritikus problémák kezelését a Microsoft végzi el Önnek. Az Azure Storage-e a széles körű tárolási megoldás Azure biztosít, szolgáltatásokat és eszközöket, amelyek azt használható száma miatt.

Nincsenek különböző Azure Storage szolgáltatás használhatja adattároláshoz. Az adatforrások száma blobok tárolására készültek legrugalmasabb beállítás [Blob-tároló](/azure/storage/blobs/storage-blobs-introduction). Blobok alapvetően fájlokat. Képek, dokumentumok, HTML-fájlok, virtuális merevlemezek (VHD), például naplók big Data típusú adatok tárolásához, a biztonsági másolatokat &mdash; pretty sok semmit. A blobok tárolása tárolókban történik, amelyek a mappákhoz hasonlatosak. Egy tároló blobok blobkészletek csoportosítását biztosítja. A storage-fiók korlátlan számú tárolót tartalmazhat, és egy tároló korlátlan számú BLOB tárolhatja.

Az Azure Storage nagy adatok és analitikák megoldások, érdemes választani egy miatt a rugalmasság, a magas rendelkezésre állás és a alacsony költség. Biztosít a gyakran használt adatok, a ritkán használt adatok, és archiválja a tárolási rétegek, a különböző használati eseteket. További információkért lásd: [Azure Blob Storage: közbeni, hűtsük le, és archiválja a tárolási rétegek](/azure/storage/blobs/storage-blob-storage-tiers).

Az Azure Blob storage is (HDInsight keresztül érhető el) hadoopból érhető el. A HDInsight egy blobtárolót használhat az Azure Storage-ben a fürt alapértelmezett fájlrendszereként. A Hadoop elosztott fájlrendszer (HDFS) rendszer által biztosított csatoló WASB illesztőprogram, a hdinsight összetevők teljes készlete működhet közvetlenül a strukturált vagy strukturálatlan adatok blobként tárolja. Az Azure Blob storage Azure SQL Data Warehouse PolyBase szolgáltatással a keresztül is elérhető.

Egyéb Azure Storage-érdemes választani alkotó jellemzői a következők:

- [Több egyidejű stratégiák](/azure/storage/common/storage-concurrency?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).
- [Vész-helyreállítási és magas rendelkezésre állású lehetőség](/azure/storage/common/storage-disaster-recovery-guidance?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).
- [Titkosítását](/azure/storage/common/storage-service-encryption?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).
- [Szerepköralapú hozzáférés-vezérlés (RBAC)](/azure/storage/common/storage-security-guide?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#management-plane-security) elérés Azure Active Directory – felhasználók és csoportok használatával.

## <a name="azure-data-lake-store"></a>Azure Data Lake Store

[Azure Data Lake Store](/azure/data-lake-store/) van egy vállalati szintű kapacitású adattár a big data koncepción alapuló adatelemzési célokra. Data Lake lehetővé teszi egy egyfelhasználós bármilyen méretű, típusú és feldolgozási sebességű adatok rögzítéséhez [biztonságos](/azure/data-lake-store/data-lake-store-overview#DataLakeStoreSecurity) műveleti és felderítési jellegű helyét.

Data Lake Store nincsenek megkötések a fiókok méretének, a fájlok méretét vagy a data lake-ben tárolt adatok mennyisége. Adatait tárolja másolatnak köszönhetően, és nincs korlátozva az az időtartam, az adatok a Data Lake tárolható. Azonkívül, hogy a fájlokat, amelyek védelmet biztosítanak a váratlan meghibásodások több másolatot, a Data lake több egyéni tárolókiszolgáló terjed egy fájl részeit. Ez javítja az olvasás átviteli sebességét a fájl adatelemzés céljából történő párhuzamos beolvasásakor.

Data Lake Store (HDInsight keresztül érhető el) hadoopból érhető el a WebHDFS-kompatibilis REST API-k használatával. Akkor fontolja meg az egyedi vagy összevont méretűek-nál nagyobb, amelyet az Azure Storage támogatja használatát az Azure Storage helyett. Van azonban [teljesítményének hangolása irányelvek](/azure/data-lake-store/data-lake-store-performance-tuning-guidance#optimizing-io-intensive-jobs-on-hadoop-and-spark-workloads-on-hdinsight) kell követnie, ha a Data Lake Store-t elsődleges tárhelyét a HDInsight-fürtök, az adott útmutatókat [Spark](/azure/data-lake-store/data-lake-store-performance-tuning-spark), [ Hive](/azure/data-lake-store/data-lake-store-performance-tuning-hive), [MapReduce](/azure/data-lake-store/data-lake-store-performance-tuning-mapreduce), és [Storm](/azure/data-lake-store/data-lake-store-performance-tuning-storm). Emellett ügyeljen arra, hogy ellenőrizze a Data Lake Store [regionális rendelkezésre állási](https://azure.microsoft.com/regions/#services), mert nem érhető el a lehető legtöbb régióban Azure tárolóként, és így ellenőrzi, hogy a HDInsight-fürt ugyanabban a régióban található.

Az Azure Data Lake Analytics összekapcsolt Data Lake Store kifejezetten lehetővé teszi, hogy a tárolt adatokra vonatkozó elemzés, és van beállítva, a teljesítmény adatok analytics forgatókönyvek esetén. Data Lake Store Azure SQL Data Warehouse PolyBase szolgáltatással a keresztül is elérhető.

## <a name="azure-cosmos-db"></a>Azure Cosmos DB

[Az Azure Cosmos DB](/azure/cosmos-db/) Microsoft globálisan elosztott több modellre adatbázis. Cosmos DB biztosítja, hogy az egyetlen-számjegy-ezredmásodperces késések fordulnak elő, a bárhol a világon 99th PERCENTILIS több jól meghatározott konzisztencia modellek finomhangolása teljesítményt nyújt, és magas rendelkezésre állású többhelyű képességeket biztosítja.

Az Azure Cosmos DB séma-független. Anélkül, hogy a séma- és index felügyeleti foglalkozik az adatok automatikusan elvégzi. Ezenkívül többmodelles, tehát natívan támogatja a dokumentum, kulcs-érték, gráf és oszlopcsalád alapú adatmodelleket egyaránt. 

Azure Cosmos DB-funkció:

- [Georeplikáció](/azure/cosmos-db/distribute-data-globally)
- [Átviteli sebesség és tárterület a rugalmas méretezést](/azure/cosmos-db/partition-data) világszerte
- [Öt jól meghatározott konzisztenciaszintek](/azure/cosmos-db/consistency-levels)

## <a name="hbase-on-hdinsight"></a>HBase on HDInsight

[Apache HBase](http://hbase.apache.org/) egy nyílt forráskódú NoSQL-adatbázis, amely a Hadoop és modellezve Google BigTable után van. A HBase véletlenszerű hozzáférést és erős konzisztenciát biztosít a nagy mennyiségű strukturálatlan és félig strukturált adatok egy séma nélküli adatbázisban oszlopcsaláddal szerint vannak rendszerezve biztosít.

Az adatok a táblasorokban vannak tárolva, és a sorokon belüli adatok oszlopcsalád szerint vannak csoportosítva. A HBase séma nélküli, abban az értelemben, hogy sem az oszlopokat és a bennük tárolt adatok kell meghatározni a használatuk előtt. A nyílt forráskód lineáris módon méreteződik át a több ezer csomópontnyi adat petabájtjainak kezelése érdekében. Az adatredundanciára, a kötegelt feldolgozásra és más olyan szolgáltatásokra támaszkodhat, amelyeket elosztott alkalmazások nyújtanak a Hadoop rendszerben.

A [HDInsight-implementáció](/azure/hdinsight/hbase/apache-hbase-overview) kihasználja a HBase méretezhető automatikus horizontális táblák, erős konzisztenciát biztosít az olvasási és írási műveletek és automatikus feladatátvételt a kibővített architektúrát. A teljesítményt a memóriába való gyorsítótárazás növeli az olvasáshoz, és a nagy streaming-kapacitás az írásokhoz. A legtöbb esetben érdemes [hozzon létre egy virtuális hálózaton belül a HBase-fürtöt](/azure/hdinsight/hbase/apache-hbase-provision-vnet) , más HDInsight-fürtök és alkalmazások közvetlenül hozzáférhetnek a táblázatokat.

## <a name="key-selection-criteria"></a>Kulcs kiválasztási feltételek

A választási lehetőségek szűkítéséhez indítása ezen kérdések megválaszolásával:

- Nem felügyelt, nagy sebességű, bármely típusú szöveges vagy bináris adatok felhőalapú tárolása kell? Ha igen, majd válassza ki a fájl tárolási lehetőségek közül.

- Szükséges optimalizált fájlok tárolására az párhuzamos analytics számítási feladatok és a magas teljesítmény/iops-értéket? Ha igen, majd válassza a olyan beállítás, amely van beállítva, hogy elemzés számítási feladat teljesítményére.

- Strukturálatlan és félig strukturált adat tárolása egy séma nélküli adatbázisban kell? Ha igen, válassza ki a nem relációs lehetőségek közül. Beállítások indexelő és adatbázis-modellek összehasonlítása Attól függően, hogy milyen adatokat kell tárolni az elsődleges adatbázis-modellek a legnagyobb tényező lehet.

- Használhatja a szolgáltatást az adott régióban? Minden egyes Azure szolgáltatás regionális rendelkezésre állásának ellenőrzése. Lásd: [régiónként rendelkezésre álló termékek](https://azure.microsoft.com/regions/services/).

## <a name="capability-matrix"></a>Képesség mátrix

A következő táblázat összefoglalja a főbb változásai képességeit.

### <a name="file-storage-capabilities"></a>Fájl tárolási képességeinek

|  | Azure Data Lake Store | Az Azure Blob Storage tárolók |
| --- | --- | --- |
| Cél | Optimalizált tárolási megoldás big data elemzés munkaterhelések |Általános célú objektum tárolására számos különböző tárolási forgatókönyvek |
| Használati esetek | Kötegelt, streaming analytics és a machine learning adatok például a naplófájlok, IoT-adatokat, kattintson az adatfolyamokat, nagy adatkészletek | Szöveges vagy bináris adatok, például az alkalmazás bármilyen típusú biztonsági end, a biztonsági mentési adatokat, a adatfolyam-továbbításhoz és általános célú adatok tárolása media |
| struktúra | Hierarchikus fájlrendszer | Egyszerű névtér objektum tároló |
| Hitelesítés | Alapján [az Azure Active Directory-identitás](/azure/active-directory/active-directory-authentication-scenarios) | A közös titkokat alapján [Tárelérési kulcsok](/azure/storage/common/storage-create-storage-account#manage-your-storage-account) és [megosztott hozzáférési aláírást kulcsok](/azure/storage/common/storage-dotnet-shared-access-signature-part-1), és [szerepköralapú hozzáférés-vezérlést (RBAC)](/azure/security/security-storage-overview) |
| Hitelesítési protokoll | OAuth 2.0-s. Hívások tartalmaznia kell egy érvényes Azure Active Directory által kibocsátott JWT (JSON web token) | Kivonat-alapú üzenethitelesítési kóddal (HMAC). Hívások tartalmaznia kell egy Base64-kódolású SHA-256 kivonatoló a HTTP-kérelmek egy része felett. |
| Engedélyezés | POSIX hozzáférés-vezérlési lista (ACL). Fájl- és szintű Azure Active Directory identitások alapuló hozzáférés-vezérlési listák állítható be. | A fiók szintű hitelesítéshez használja [Tárelérési kulcsok](/azure/storage/common/storage-create-storage-account#manage-your-storage-account). A fiók, a tároló vagy a blob engedélyezési használjon [megosztott hozzáférési aláírást kulcsok](/azure/storage/common/storage-dotnet-shared-access-signature-part-1). |
| Naplózás | Érhető el.  |Elérhető |
| Titkosítás inaktív állapotban | Átlátszó, kiszolgálóoldali | Átlátszó, kiszolgálóoldali; Ügyféloldali titkosítás |
| Fejlesztői SDK-k | .NET, Java, Python, Node.js | .NET, Java, Python, Node.js, a C++, a Ruby |
| Elemzés számítási feladat teljesítményére | Optimalizált párhuzamos analytics munkaterhelések teljesítményét, nagy mennyiségre és IOPS | Az analytics-feladatok nem optimalizált |
| Méretkorlát | Nem határoz meg a fiókok méretének, a fájl méretét vagy a fájlok száma | Egyes korlátok dokumentált [Itt](/azure/azure-subscription-service-limits#storage-limits) |
| Georedundancia | Helyileg redundáns (adatok többszörös lemásolását, egy Azure-régióban) | Helyileg redundáns (LRS), a globális redundáns (GRS), az írásvédett globálisan redundáns (RA-GRS). Lásd: [Itt](/azure/storage/common/storage-redundancy) további információ |

### <a name="nosql-database-capabilities"></a>NoSQL adatbázis képességek

| | Azure Cosmos DB | HBase on HDInsight |
| --- | --- | --- |
| Elsődleges adatbázis-modell | Dokumentálja a tároló, a graph, a kulcs-érték tároló, a széles oszlop tároló | Széles oszlop tároló |
| Másodlagos indexek | Igen | Nem |
| SQL nyelvi támogatás | Igen | Igen (használja a [Phoenix](http://phoenix.apache.org/) JDBC-illesztőt) |
| Konzisztencia | Az erős, kötött elavulás, munkamenet, egységes előtag, végleges | Erős |
| Natív integráció az Azure Functions | [Igen](/azure/cosmos-db/serverless-computing-database) | Nem |
| Automatikus globális terjesztési | [Igen](/azure/cosmos-db/distribute-data-globally) | Nem [HBase fürt replikációs konfigurálható](/azure/hdinsight/hbase/apache-hbase-replication) végleges konzisztencia régiók között |
| Díjszabási modell | Rugalmasan méretezhető kérelemegység (RUs) másodpercenként felszámított, igény szerint rugalmasan méretezhető tárolás | Perc árképzési HDInsight-fürthöz (horizontális skálázás csomópontok), tárolás |
