---
title: Az adatraktározás terén és adatpiacainak
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: eec883c68cf94637c3061814d0841c73b58d7e52
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/31/2018
---
# <a name="data-warehousing-and-data-marts"></a>Az adatraktározás terén és adatpiacainak

Adatraktár tárháza központi, szervezeti, relációs egy vagy több különböző forrásokból származó integrált adatok sok vagy az összes területek között. Az adatraktárak aktuális és korábbi adatokat tárolhatnak, és különböző módon használható a reporting és az adatok elemzése.

![Az Azure-ban adatraktározási](./images/data-warehousing.png)

Áthelyezni az adatokat az adatraktárban, akkor ki kell olvasni a különböző forrásokból, amely tartalmazza a fontos üzleti adatok rendszeres időközönként. Az adatok áthelyezése, képes kell formázni, tisztítani, érvényesítve, foglalja össze, és az átszervezés. Alternatív megoldásként az adatokat tárolhatja a legalacsonyabb szintű részletesség, a jelentés az adatraktár előírt összesített nézetek. Mindkét esetben az adatraktárban használt jelentések, elemzés és fontos üzleti döntéseket üzleti intelligenciával eszközökkel képező adatok állandó tárhely válik.

## <a name="data-marts-and-operational-data-stores"></a>Adatpiacainak és működési adatokat tároló

Léptékű adatok kezelése bonyolult, és szeretné, hogy minden adatot a teljes vállalat jelölő egyetlen adatraktár ritkább válik. Ehelyett hozzon létre a szervezetek kisebb, célzottabb adatraktárban nevű *adatpiacainak*, amely teszi közzé a kívánt adathoz elemzés céljából. Az orchestration folyamat feltölti az adatpiac jelenti az operatív adatok tárolóban tárolt adatok. A működésiadat-tároló egy közvetítőként a forrásrendszerben tranzakciós és az Adatközpont között. Az operatív adatokat tároló által kezelt tranzakciós forrásrendszerben levő adatok megtisztított verziója, és általában a data warehouse-bA vagy adatpiac által kezelt a korábbi adatok egy részét. 

## <a name="when-to-use-this-solution"></a>Ez a megoldás használatával

Adatraktár akkor válassza, ha be kell kapcsolni a nagy mennyiségű adatot az operációs rendszerek olyan formátumra, amely könnyen értelmezhető, aktuális és pontos. Az adatraktárak nem kell azonos tömör adatok szerkezetének az lehet, hogy használja az a működési/OLTP-adatbázisok. Célszerű üzleti felhasználók és elemzők, a séma bővítésével leegyszerűsítheti a adatok kapcsolatok átalakításának és több táblázatot konszolidálhatják egy oszlopnevek is használhatja. Az alábbi lépések segítségével útmutató felhasználók, akik ad hoc jelentések létrehozása és a BI rendszereken az egy adatbázis-rendszergazda (DBA) vagy a fejlesztővel adatok nélkül az adatok elemzése és jelentések létrehozása.

Adatraktár esetekben érdemes szüksége előzményadatokat különálló, a teljesítményre vonatkozó megfontolásból tranzakció forrásrendszerektől. Az adatraktárak könnyen előzményadatokat férhetnek több helyről, ha egy központi helyen közös formátumok, közös kulcsok, közös adatmodellekben és közös hozzáférési módszer használatával.

Az adatraktárak vannak optimalizálva, olvasási hozzáférést, a jelentések futtatása ellen a forrásrendszerben tranzakció képest gyorsabb jelentéskészítésre eredményez. Az adatraktárakra emellett a következő előnyöket biztosítják:

* A különböző forrásokból származó összes előzményadatokat tárolja, és elérhető az adatraktár igazság egyetlen forrásaként.
* Az adatminőségi javíthatja törli az adatokat a program az importáláskor az adatraktárba, pontosabb adatokat szolgáltató való egységes kódok és leírásokat.
* Jelentéskészítési eszközök ne befolyásolják a tranzakciós forrásrendszerek feldolgozási ciklus lekérdezéshez. Egy data warehouse lehetővé teszi a tranzakciós rendszer írási műveletekről, kezelése során az adatraktár megfelel a legtöbb az olvasási kérések túlnyomórészt összpontosítanak.
* Adatraktár összevonni az adatokat a különböző szoftver segítségével.
* Adatok adatbányászati eszközök segítséget rejtett minták használatával automatikus módszereit, amelyek a raktárban tárolt adatok alapján.
* Az adatraktárak megkönnyítik a biztonságos hozzáférés biztosítása jogosult felhasználókra, míg mások való hozzáférés korlátozása. Nincs szükség az üzleti felhasználónak hozzáférést biztosít a forrásadatok, ezzel kiküszöbölve egy potenciális támadási felület éles tranzakció egy vagy több rendszer ellen.
* Az adatraktárak könnyebben üzleti intelligenciát biztosító megoldások felett az adatokat, például létrehozásához [OLAP-kockák](online-analytical-processing.md).

## <a name="challenges"></a>Problémák

Egy adatraktár az az üzleti igényeinek megfelelően beállítása átvihetők a következő problémákkal egy részénél:

* Véglegesítő megfelelően modellezheti az üzleti fogalmak szükséges időt. Fontos lépés, ez, vagy az adatraktárak sem áll, ahol koncepció leképezési meghajtók a projekt a további információkat. Ez magában foglalja a szabványosítása üzleti-mal kapcsolatos kifejezések és a közös formázza az adathordozót (például a pénznem és dátum), és át lesznek strukturálva a séma oly módon, hogy az üzleti felhasználók számára értelmes állomásnevet, de továbbra is biztosítja az adatok összesítések és kapcsolatok pontosságát.
* Tervezési, valamint beállítja a adatok előkészítése. Szempont például a forrásrendszerből tranzakciós adatok másolása az adatraktár és az előzményadatok áthelyezése kívül a működési adatokat tárolja, és az adatraktárba.
* Fenntartása, illetve az adatminőségi javítása tisztítási, mert az adatok importálása az adatraktárba.

## <a name="data-warehousing-in-azure"></a>Az Azure-ban adatraktározási

Az Azure előfordulhat, hogy egy vagy több adatforráshoz, hogy felhasználói tranzakciókat, vagy a különböző részlegek által használt különböző üzleti alkalmazások. Ez hagyományosan adatai egy vagy több [OLTP](online-transaction-processing.md) adatbázisok. Az adatok a más adathordozók, például a hálózati megosztások, Azure Storage Blobsba vagy data lake állandósítható. Az adatok is tárolhatók, az adatraktár saját magát vagy egy relációs adatbázisban, például az Azure SQL Database. Az analitikai adatok tárolási réteg célja lekérdezések elemzés és a data warehouse-bA vagy adatpiac jelentéskészítési eszközök által kibocsátott kielégítéséhez. Az Azure ezen analitikai tároló képesség érheti el az Azure SQL Data Warehouse szolgáltatással, vagy az Azure HDInsight Hive vagy az interaktív lekérdezés segítségével. Emellett bizonyos fokú rendszeresen áthelyezése vagy adatok másolása az adattárolás az adatraktáron, amely teheti az orchestration kell a Azure HDInsight Azure Data Factory és az Oozie használatával.

Kapcsolódó szolgáltatások:

* [Azure SQL Database](/azure/sql-database/)
* [SQL Server virtuális gépen](/sql/sql-server/sql-server-technical-documentation)
* [Azure Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is)
* [Apache Hive hdinsight](/azure/hdinsight/hadoop/hdinsight-use-hive)
* [A HDInsight (Hive LLAP) interaktív lekérdezés](/azure/hdinsight/interactive-query/apache-interactive-query-get-started)


## <a name="technology-choices"></a>Technológiai lehetőségek

- [Az adatraktárak](../technology-choices/data-warehouses.md)
- [Vezénylési folyamat](../technology-choices/pipeline-orchestration-data-movement.md)

