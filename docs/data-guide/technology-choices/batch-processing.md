---
title: Egy kötegfeldolgozási technológia kiválasztása
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 0117798af82f2caa6704dc86e88be57f09c381ea
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/06/2018
---
# <a name="choosing-a-batch-processing-technology-in-azure"></a>Egy köteg feldolgozása az Azure-ban technológia kiválasztása

Big data-megoldások gyakran használnak a szűrőt, összesített, hosszan futó kötegelt feladatok, és ellenkező esetben az előkészítés elemzés céljából. Általában ezeket a feladatokat tartalmaz, amely forrásfájlokat olvasásakor (például a HDFS, az Azure Data Lake Store és Azure Storage) méretezhető tárolás, őket, és a kimeneti írását méretezhető tárolás új fájlokat. 

Ilyen kötegelt feldolgozáson a végrehajtók kulcsfontosságú követelmény azt a képességet horizontális felskálázás számítások, nagy mennyiségű adatok kezeléséhez. Valós idejű feldolgozással eltérően azonban kötegfeldolgozási várhatóan késések (az adatfeldolgozást és a számítási eredményt közötti idő), amelyek mérik az óra.

## <a name="what-are-your-options-when-choosing-a-batch-processing-technology"></a>Mik azok a beállítások egy kötegfeldolgozási technológia kiválasztásakor?

Az Azure kötegelt feldolgozásra core követelményeknek megfelelő összes a következő adatokat tárolja:

- [Azure Data Lake Analytics](/azure/data-lake-analytics/)
- [Azure SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is)
- [A Spark on HDInsight](/azure/hdinsight/spark/apache-spark-overview)
- [A Hive HDInsight](/azure/hdinsight/hadoop/hdinsight-use-hive)
- [HDInsight Hive LLAP](/azure/hdinsight/interactive-query/apache-interactive-query-get-started)

## <a name="key-selection-criteria"></a>Kulcs kiválasztási feltételek

A választási lehetőségek szűkítéséhez indítása ezen kérdések megválaszolásával:

- A saját kiszolgálóit kezelése helyett egy felügyelt szolgáltatás van szüksége?

- Szeretné kötegfeldolgozási logika deklarációval vagy imperatively szerzői?

- Fog-e végre az felszakadásáig a köteges feldolgozás? Ha igen, vegye figyelembe a lehetőségeket, amelyek lehetővé teszik, hogy felfüggeszti a fürt, vagy amelynek árképzési modellt kötegelt /.

- A kötegelt feldolgozást, például ellenőrizzék a referenciaadatok együtt relációs adatokat kérdez kell? Ha igen, fontolja meg a beállításokat, amelyek lehetővé teszik külső relációs tároló lekérdezését.

## <a name="capability-matrix"></a>Képesség mátrix

A következő táblázat összefoglalja a főbb változásai képességeit. 

### <a name="general-capabilities"></a>Általános képességek

| | Azure Data Lake Analytics | Azure SQL Data Warehouse | A Spark on HDInsight | A Hive HDInsight | HDInsight Hive LLAP |
| --- | --- | --- | --- | --- | --- |
| Van a felügyelt | Igen | Igen | Igen <sup>1</sup> | Igen <sup>1</sup> | Igen <sup>1</sup> |
| Támogatja a számítási felfüggesztése | Nem | Igen | Nem | Nem | Nem |
| Relációs adattároló | Igen | Igen | Nem | Nem | Nem |
| Programozható | U-SQL | T-SQL | Python, Scala, Java, R | HiveQL | HiveQL |
| Programozási paradigma | Deklaratív és imperatív  | Deklaratív | Deklaratív és imperatív | Deklaratív | Deklaratív | 
| Díjszabási modell | A kötegelt száma | Fürt óránként | Fürt óránként | Fürt óránként | Fürt óránként |  

[1], kézi konfigurálás és a méretezés.

### <a name="integration-capabilities"></a>Integrációs funkciók

| | Azure Data Lake Analytics | SQL Data Warehouse | A Spark on HDInsight | A Hive HDInsight | HDInsight Hive LLAP |
| --- | --- | --- | --- | --- | --- |
| Az Azure Data Lake Store elérése | Igen | Igen | Igen | Igen | Igen |
| Az Azure Storage-lekérdezést | Igen | Igen | Igen | Igen | Igen |
| Külső relációs áruházakból lekérdezése | Igen | Nem | Igen | Nem | Nem |

### <a name="scalability-capabilities"></a>Méretezhetőség képességek

| | Azure Data Lake Analytics | SQL Data Warehouse | A Spark on HDInsight | A Hive HDInsight | HDInsight Hive LLAP |
| --- | --- | --- | --- | --- | --- |
| Kibővített granularitási  | Feladatonként | Fürtönként | Fürtönként | Fürtönként | Fürtönként |
| Gyors kibővítési (kevesebb mint 1 perc) | Igen | Igen | Nem | Nem | Nem |
| A memóriában történő gyorsítótárazás adatok | Nem | Igen | Igen | Nem | Igen | 

### <a name="security-capabilities"></a>Biztonsági képességei

| | Azure Data Lake Analytics | SQL Data Warehouse | A Spark on HDInsight | Apache Hive hdinsight | A HDInsight LLAP struktúra |
| --- | --- | --- | --- | --- | --- |
| Hitelesítés  | Azure Active Directory (Azure AD) | SQL / Azure AD | Nem | helyi és az Azure AD <sup>1</sup> | helyi és az Azure AD <sup>1</sup> |
| Engedélyezés  | Igen | Igen| Nem | Igen <sup>1</sup> | Igen <sup>1</sup> |
| Naplózás  | Igen | Igen | Nem | Igen <sup>1</sup> | Igen <sup>1</sup> |
| Adat-titkosítás inaktív állapotban | Igen| Igen <sup>2</sup> | Igen | Igen | Igen |
| Sorszintű biztonság | Nem | Igen | Nem | Igen <sup>1</sup> | Igen <sup>1</sup> |
| Támogatja a tűzfalak | Igen | Igen | Igen | Igen <sup>3</sup> | Igen <sup>3</sup> |
| Dinamikus adatmaszkolás | Nem | Nem | Nem | Igen <sup>1</sup> | Igen <sup>1</sup> |

[1] igényel a használata egy [HDInsight-fürt tartományhoz](/azure/hdinsight/domain-joined/apache-domain-joined-introduction).

[2] átlátszó Data Encryption (TDE) segítségével az inaktív adatok titkosításához és visszafejtéséhez szükséges.

[3] támogatással [egy Azure virtuális hálózaton belül használt](/azure/hdinsight/hdinsight-extend-hadoop-virtual-network).
