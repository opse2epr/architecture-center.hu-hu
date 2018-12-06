---
title: Egy adattárolási módszerektől kiválasztása
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.openlocfilehash: c97249228ca45a7a17822b6dd55acad6360c6f6b
ms.sourcegitcommit: e7e0e0282fa93f0063da3b57128ade395a9c1ef9
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/05/2018
ms.locfileid: "52902646"
---
# <a name="choosing-a-big-data-storage-technology-in-azure"></a>A big Data típusú adatok tárolási technológia kiválasztása az Azure-ban

Ez a témakör a big data-megoldások adattárolás lehetőségeit hasonlítja össze &mdash; pontosabban tömeges adatbetöltés és a kötegelt feldolgozásra, és nem a adattárolás [analitikus adattárak](./analytical-data-stores.md) vagy [valós idejű adatfolyam-feldolgozó](./real-time-ingestion.md).

## <a name="what-are-your-options-when-choosing-data-storage-in-azure"></a>Mik azok a beállítások az adattárolás kiválasztásakor az Azure-ban?

Az adatok feldolgozása Azure-ba, igényeitől függően több lehetőség áll rendelkezésre:

**A File storage**

- [Az Azure Storage-blobok](/azure/storage/blobs/storage-blobs-introduction)
- [Azure Data Lake Store](/azure/data-lake-store/)

**NoSQL-adatbázisok**

- [Azure Cosmos DB](/azure/cosmos-db/)
- [A HDInsight-alapú HBase](https://hbase.apache.org/)

## <a name="azure-storage-blobs"></a>Az Azure Storage-blobok

Az Azure Storage szolgáltatás jelenleg felügyelt tárolási szolgáltatás, amely magas rendelkezésre álló, biztonságos, tartós, méretezhető és redundáns. A karbantartást és a kritikus problémák kezelését a Microsoft végzi el Önnek. Az Azure Storage szolgáltatás a legtöbb széles körben használt tárolási megoldásnak az Azure biztosít, szolgáltatásokat és eszközöket, amelyekkel lehet vele száma miatt.

Nincsenek különböző Azure Storage szolgáltatás adatok tárolására használható. A legrugalmasabb lehetőség az olyan adatforrások közül blobok tárolására [a Blob storage-](/azure/storage/blobs/storage-blobs-introduction). Blobok lényegében ugyanolyan fájlok. Képek, dokumentumok, HTML-fájlok, virtuális merevlemezeket (VHD), például naplók big Data típusú adatok tárolására, az adatbázisok biztonsági mentések &mdash; pretty szinte bármit. A blobok tárolása tárolókban történik, amelyek a mappákhoz hasonlatosak. Egy tároló blobokat áll csoportosítását biztosítja. Egy tárfiók korlátlan számú tárolót tartalmazhat, egy tároló pedig korlátlan számú blob tárolására használható.

Az Azure Storage megfelelő választás az olyan big data és analitikai megoldások, a rugalmasság, magas rendelkezésre állású és alacsony költségek miatt. Itt a gyakori és ritka elérésű és archív tárolási szintek különböző használati esetek. További információkért lásd: [Azure Blob Storage: gyakori, ritka és archív tárolási szintek](/azure/storage/blobs/storage-blob-storage-tiers).

Az Azure Blob storage elérhető lesz a Hadoop (HDInsight keresztül érhető el). A HDInsight egy blobtárolót használhat az Azure Storage-ben a fürt alapértelmezett fájlrendszereként. Egy Hadoop elosztott fájlrendszer (HDFS) felületen WASB illesztőprogram által biztosított keresztül a HDInsight összetevők teljes készlete működhet közvetlenül a strukturált vagy strukturálatlan adatokon a BLOB. Az Azure Blob storage-n keresztül az Azure SQL Data Warehouse a PolyBase szolgáltatás használatával is elérhető.

Egyéb szolgáltatásokat, amelyek az Azure Storage a jó választás a következők:

- [Több egyidejű stratégiák](/azure/storage/common/storage-concurrency?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).
- [Vészhelyreállítás és magas rendelkezésre állási lehetőségek](/azure/storage/common/storage-disaster-recovery-guidance?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).
- [Titkosítás inaktív állapotban](/azure/storage/common/storage-service-encryption?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).
- [Szerepköralapú hozzáférés-vezérlés (RBAC)](/azure/storage/common/storage-security-guide?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#management-plane-security) való hozzáférés az Azure Active Directory – felhasználók és csoportok használata.

## <a name="azure-data-lake-store"></a>Azure Data Lake Store

[Az Azure Data Lake Store](/azure/data-lake-store/) van egy vállalati szintű kapacitású adattár a big Data típusú adatok adatelemzési számítási feladatokhoz. A Data Lake lehetővé teszi, hogy egy egyfelhasználós bármilyen méretű, típusú és feldolgozási sebességű adatok [biztonságos](/azure/data-lake-store/data-lake-store-overview#DataLakeStoreSecurity) műveleti és felderítési jellegű helyét.

Data Lake Store nincsenek megkötések a fiókok méretének, a fájlok méretét vagy a data lake-ben tárolt adatok mennyisége. Adatok azáltal, hogy több példányt tartósan tárolja, és az időtartam, ameddig az adatok tárolhatók a Data Lake a nem korlátozott. Amellett, hogy a fájlok védelme érdekében váratlan meghibásodások esetén több példányát, a Data lake több egyéni tárolókiszolgáló kártevőcsaládba egy fájl részeit. Ez javítja az olvasás átviteli sebességét a fájl adatelemzés céljából történő párhuzamos beolvasásakor.

Data Lake Store (HDInsight keresztül érhető el) hadoopból érhető el a WebHDFS-kompatibilis REST API-k használatával. Akkor fontolja meg az egyedi vagy összevont fájlméret-nál nagyobb, mint amelyet az Azure Storage által támogatott használatával Ez az Azure Storage-megoldásként. Vannak azonban [teljesítmény-finomhangolási útmutató](/azure/data-lake-store/data-lake-store-performance-tuning-guidance#optimizing-io-intensive-jobs-on-hadoop-and-spark-workloads-on-hdinsight) kell követnie, ha az elsődleges tárolóként használ a Data Lake Store egy HDInsight-fürtöt, az adott irányelvek [Spark](/azure/data-lake-store/data-lake-store-performance-tuning-spark), [ Hive-](/azure/data-lake-store/data-lake-store-performance-tuning-hive), [MapReduce](/azure/data-lake-store/data-lake-store-performance-tuning-mapreduce), és [Storm](/azure/data-lake-store/data-lake-store-performance-tuning-storm). Emellett mindenképpen ellenőrizze a Data Lake Store [régiónkénti rendelkezésre állás](https://azure.microsoft.com/regions/#services), mert nem érhető el a legtöbb régióban az Azure-tároló, és ugyanabban a régióban a HDInsight-fürt által működnie kell.

Az Azure Data Lake Analytics szolgáltatással párosítva, Data Lake Store kifejezetten lehetővé teszi, hogy a tárolt adatok elemzése és a nagy teljesítményt nyújtson az adatelemzési forgatókönyvekben. Data Lake Store használatával az Azure SQL Data Warehouse a PolyBase szolgáltatás használatával is elérhető.

## <a name="azure-cosmos-db"></a>Azure Cosmos DB

[Az Azure Cosmos DB](/azure/cosmos-db/) van a Microsoft globálisan elosztott többmodelles adatbázisa. Cosmos DB számjegy-ezredmásodperces a világ bármely pontján található 99 százalékon segíti a teljesítmény finomhangolását több jól definiált konzisztenciamodelleket kínál, és többkiszolgálós képességekkel rendelkező magas rendelkezésre állást garantál.

Azure Cosmos DB a sémafüggetlen. Automatikusan indexeli az összes adatot, nem kell sémákat és indexeket felügyeleti foglalkozik. Ezenkívül többmodelles, tehát natívan támogatja a dokumentum, kulcs-érték, gráf és oszlopcsalád alapú adatmodelleket egyaránt. 

Az Azure Cosmos DB-funkcióit:

- [Georeplikáció](/azure/cosmos-db/distribute-data-globally)
- [A teljesítmény és a tárterület rugalmas méretezése](/azure/cosmos-db/partition-data) világszerte
- [Öt jól definiált konzisztenciaszint](/azure/cosmos-db/consistency-levels)

## <a name="hbase-on-hdinsight"></a>HBase on HDInsight

[Az Apache HBase](https://hbase.apache.org/) egy nyílt forráskódú NoSQL-adatbázis, amely Hadoop épül, és modellezni Google BigTable után van. A HBase véletlenszerű hozzáférést és erős konzisztenciát biztosít a nagy mennyiségű strukturálatlan és részben strukturált adatok séma nélküli adatbázisban oszlopcsaládok szerint rendezve biztosít.

Az adatok a táblasorokban vannak tárolva, és a sorokon belüli adatok oszlopcsalád szerint vannak csoportosítva. A HBase a séma nélküli, abban az értelemben, hogy az oszlopok és a bennük tárolt adatok típusa nem kell használat előtt meghatározni. A nyílt forráskód lineáris módon méreteződik át a több ezer csomópontnyi adat petabájtjainak kezelése érdekében. Az adatredundanciára, a kötegelt feldolgozásra és más olyan szolgáltatásokra támaszkodhat, amelyeket elosztott alkalmazások nyújtanak a Hadoop rendszerben.

A [HDInsight-implementáció](/azure/hdinsight/hbase/apache-hbase-overview) kihasználja a HBase táblák automatikus árnyalását, erős konzisztencia az olvasási és írási és automatikus feladatátvételt biztosít horizontális felskálázást architektúrájának. A teljesítményt a memóriába való gyorsítótárazás növeli az olvasáshoz, és a nagy streaming-kapacitás az írásokhoz. A legtöbb esetben érdemes [hozzon létre egy virtuális hálózaton belül, a HBase-fürt](/azure/hdinsight/hbase/apache-hbase-provision-vnet) így más HDInsight-fürtök és alkalmazások közvetlenül hozzáférhet a táblákat.

## <a name="key-selection-criteria"></a>Fontosabb kritériumok

Így szűkítheti, első lépésként a kérdések megválaszolása:

- Felügyelt, nagy sebességű, felhőalapú tárolás bármilyen szöveget vagy bináris adatot kell? Ha igen, majd válassza ki a fájl tárolási lehetőségek közül.

- Optimalizált fájlok tárolásához szüksége párhuzamos elemzési és magas átviteli sebesség vagy iops-t? Ha igen, majd válassza ki az olyan beállítás, amely van beállítva, hogy elemzési számítási feladatok teljesítményére.

- Szükség van egy séma nélküli adatbázisban strukturálatlan és részben strukturált adatok tárolására? Ha igen, válassza ki a nem relációs lehetőségek közül. Lehetőségek az indexelés és adatbázis-modellek összehasonlítása. Az elsődleges adatbázis modellek tárolni kívánt adatok típusától függően a legnagyobb tényező lehet.

- Az Ön régiójában is használhatja a szolgáltatást? Ellenőrizze az egyes Azure-szolgáltatás regionális elérhetősége. Lásd: [elérhető termékek régiók szerint](https://azure.microsoft.com/regions/services/).

## <a name="capability-matrix"></a>Képességmátrix

A következő táblázat összefoglalja a fő különbségeket, a képességek.

### <a name="file-storage-capabilities"></a>Fájl tárolási lehetőségek

|  | Azure Data Lake Store | Az Azure Blob Storage-tárolók |
| --- | --- | --- |
| Cél | A big data-elemzési célokra tárolás |Általános célú objektum tárolni a storage-forgatókönyvek széles választéka |
| Használati esetek | Kötegelt, például a streaming analytics és a machine learning adat naplófájlokból, IoT-adatokat, kattintson az adatfolyamot, a nagyméretű adathalmazok | Bármilyen típusú szöveget vagy bináris adatot, például az alkalmazás biztonsági célból, biztonsági mentési adatokat, a streamelési és az általános célú adatok médiatárolónkba |
| struktúra | Hierarchikus fájlrendszer | Objektumtároló strukturálatlan névtér esetében |
| Hitelesítés | Alapján [az Azure Active Directory-identitásokkal](/azure/active-directory/active-directory-authentication-scenarios) | A közös titkos kulcsot alapján [hozzáférési kulcsainak](/azure/storage/common/storage-create-storage-account#manage-your-storage-account) és [megosztott hozzáférési aláírást kulcsok](/azure/storage/common/storage-dotnet-shared-access-signature-part-1), és [szerepköralapú hozzáférés-vezérlés (RBAC)](/azure/security/security-storage-overview) |
| Hitelesítési protokoll | OAuth 2.0. Hívások tartalmaznia kell egy Azure Active Directory által kiadott érvénytelen JWT (JSON webes jogkivonat) | Üzenet kivonat-alapú hitelesítési kód (HMAC). Hívások a Base64-kódolású SHA-256 kivonatoló tartalmaznia kell a HTTP-kérelem egy része felett. |
| Engedélyezés | A POSIX hozzáférés-vezérlési listák (ACL). Fájl- és szintű hozzáférés-vezérlési listák az Azure Active Directory-identitások alapján állítható be. | A fiókszintű engedélyezési használja [hozzáférési kulcsainak](/azure/storage/common/storage-create-storage-account#manage-your-storage-account). A fiók, tárolót vagy blobot engedélyezési használja [közös hozzáférési aláírási kulcsokat](/azure/storage/common/storage-dotnet-shared-access-signature-part-1). |
| Naplózás | Érhető el.  |Elérhető |
| Titkosítás inaktív állapotban | Transzparens, kiszolgálóoldali | Átlátszó, kiszolgálóoldali; Ügyféloldali titkosítás |
| Fejlesztői SDK-k | .NET, Java, Python, Node.js | .NET, Java, Python, Node.js, a C++, a Ruby |
| Elemzési számítási feladat teljesítményére | Optimalizált teljesítménygyűjtés párhuzamos elemzési számítási feladatokhoz, nagy átviteli sebességű és IOPS | Nem optimalizált elemzési számítási feladatokhoz |
| Blobméretének korlátjai | A fiókok méretének, a fájlok méretét vagy a fájlok száma korlátlan | Dokumentált konkrét korlátozások [Itt](/azure/azure-subscription-service-limits#storage-limits) |
| Georedundancia | Helyileg redundáns (az adatok egy Azure-régióban. több példány) | Helyileg redundáns (LRS), globálisan georedundáns (GRS), az írásvédett globálisan redundáns (RA-GRS). Lásd: [Itt](/azure/storage/common/storage-redundancy) további információ |

### <a name="nosql-database-capabilities"></a>Nosql-alapú adatbázis-funkcióiról

|                                    |                                           Azure Cosmos DB                                           |                                                             HBase on HDInsight                                                             |
|------------------------------------|-----------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------|
|       Elsődleges adatbázismodell       |                      Dokumentum-tároló, gráf, kulcs-érték tároló, széles oszloptár                      |                                                             Széles körű oszloptár                                                              |
|         A másodlagos indexek          |                                                 Igen                                                 |                                                                     Nem                                                                     |
|        SQL nyelvi támogatás        |                                                 Igen                                                 |                                     Igen (használatával a [Phoenix](https://phoenix.apache.org/) JDBC-illesztőprogram)                                      |
|            Konzisztencia             |                   Erős, kötött elavulás, munkamenet, konzisztens előtag, végleges                   |                                                                   Erős                                                                   |
| Natív integráció az Azure Functions |                        [Igen](/azure/cosmos-db/serverless-computing-database)                        |                                                                     Nem                                                                     |
|   Automatikus globális terjesztés    |                          [Igen](/azure/cosmos-db/distribute-data-globally)                           | Nem [HBase-fürt replikációja konfigurálható](/azure/hdinsight/hbase/apache-hbase-replication) régióban, végleges konzisztencia |
|           Díjszabási modell            | Másodpercalapú díjat számítunk fel, ha szükséges, rugalmasan méretezhető tárhelyért rugalmasan méretezhető kérelemegységet (RU) |                              Percenkénti díjszabás HDInsight-fürt (vízszintes méretezés a csomópontok), tárolás                               |

