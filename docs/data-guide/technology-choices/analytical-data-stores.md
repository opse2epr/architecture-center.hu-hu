---
title: Az analitikai adatokat tároló kiválasztása
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: cdc32c16e30aec5e1c0cb6959182215f99d56b56
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/06/2018
ms.locfileid: "30846882"
---
# <a name="choosing-an-analytical-data-store-in-azure"></a>Az analitikai adatokat tároló kiválasztása az Azure-ban

Az egy [big Data típusú adatok](../big-data/index.md) architektúra, szükség van a gyakran az az analitikus adatok tárolására szolgál a feldolgozott adatok strukturált formátuma nem kérdezhetők le analitikai eszközeivel. Analitikai adatokat tárolja, hogy mindkét közbeni elérési támogatási lekérdezését és cold-path adatok együttesen nevezzük szolgáló réteg, vagy az adatok tárolására szolgál.

A szolgáló réteg a gyakran használt adatok elérési útja és a cold elérési foglalkozik a feldolgozott adatok. Az a [lambda architektúra](../big-data/index.md#lambda-architecture), a szolgáló réteg oszlik egy _sebesség szolgál_ réteg esetében, amely Növekményesen feldolgozott adatokat tárolja, és egy _szolgálkötegelt_szintje, amely tartalmazza a kötegelt feldolgozásra kimenet. A szolgáló réteg erős támogatást igényel az véletlenszerű olvasási és kis késésű. Is támogatnia kell az adattárolás a sebesség réteg véletlenszerű írások, mert adatok betöltése a tárolóba kötegelt okozna nemkívánatos késést. Másrészről adattárolás a kötegelt réteg nem támogatja a véletlenszerű írások kell, de ehelyett kötegelt írási műveletek.

Nincs egyetlen legjobb adatok felügyeleti választás az összes adat tárolási feladatok elvégzéséről van. A különböző feladatok vannak optimalizálva, különböző adatkezelési megoldásokból. A legtöbb valós felhőalapú alkalmazások és folyamatok big Data típusú adatok különböző adatok tárolási követelményekkel rendelkezik, és gyakran használt adatok tárolási megoldások kombinációja.

## <a name="what-are-your-options-when-choosing-an-analytical-data-store"></a>Mik azok a beállítások egy analitikai adattároló kiválasztásakor?

Az adatok tárolási szolgáltató Azure igényeitől függően több lehetőség áll rendelkezésre:

- [SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is)
- [Azure SQL Database](/azure/sql-database/)
- [SQL Server Azure virtuális gépen](/sql/sql-server/sql-server-technical-documentation)
- [A HDInsight HBase/Phoenix](/azure/hdinsight/hbase/apache-hbase-overview)
- [A HDInsight LLAP struktúra](/azure/hdinsight/interactive-query/apache-interactive-query-get-started)
- [Az Azure Analysis Services](/azure/analysis-services/analysis-services-overview)
- [Azure Cosmos DB](/azure/cosmos-db/)

Ezeket a beállításokat, amely különböző típusú feladatok vannak optimalizálva, különböző adatbázis-modellek biztosítják:

- [Kulcs/érték](https://msdn.microsoft.com/library/dn313285.aspx#sec7) adatbázisok tartsa egy szerializált objektum minden egyes kulcs értékre. Nagy mennyiségű adatot, ha le szeretné kérdezni egy elemet egy adott kulcs értéket, és nem kell tárolásához jó fontosságúak a lekérdezés más tulajdonságai, a cikk alapján.
- [A dokumentum](https://msdn.microsoft.com/library/dn313285.aspx#sec8) adatbázisok a következők kulcs/érték-adatbázisok, amelyben az értékek a következők *dokumentumok*. A "dokumentumok" Ebben a környezetben az elnevezett mezők és értékek gyűjteménye. Általában az adatbázis tárolja az adatokat egy formátumban, például XML, YAM, JSON vagy BSON, de használhat egyszerű szöveg. A dokumentum adatbázisok nem kulcs mezőkre lekérdezhetik és hatékonyabbá teszi lekérdezése másodlagos indexek meghatározása. Így a dokumentum-adatbázis megfelelő feltételekkel bonyolultabb, mint a dokumentum kulcs értékének beolvasása igénylő alkalmazásokhoz. Például lekérdezhet olyan mezőkön, például termékazonosító, a felhasználói azonosító vagy az ügyfél neve.
- [Oszlop-család](https://msdn.microsoft.com/library/dn313285.aspx#sec9) adatbázisok a következők adattárolás szerkezeti gyűjteményekbe kapcsolódó oszlopok oszlopcsaláddal nevű kulcs/érték adattárolókhoz. Nyilvántartásba adatbázis Előfordulhat például, a felhasználónév az oszlopok egy csoportja (először középső, legutóbbi), az a személy címét egy csoport és egy csoportot az a személy profil adatait (születési, nemét adatok). Az adatbázis tárolhatja minden oszlop termékcsalád külön partícióra, míg egy személy ugyanazzal a kulccsal kapcsolatos adatokat. Egy alkalmazás egy oszlop családba elolvashatják összes entitásra vonatkozó adatok elolvasása nélkül történik.
- [Graph](https://msdn.microsoft.com/library/dn313285.aspx#sec10) adatbázisok objektumok és kapcsolatok gyűjteményeként információk tárolására. Egy grafikonon adatbázis hatékony hajthat végre az egymás között az objektumok és kapcsolatok a hálózaton áthaladó lekérdezések. Például lehet, hogy az objektumok alkalmazottak HR-adatbázisban, és érdemes lehetővé teszi lekérdezések például a "található összes közvetlenül vagy közvetve dolgozó alkalmazottak Scott."

## <a name="key-selection-criteria"></a>Kulcs kiválasztási feltételek

A választási lehetőségek szűkítéséhez indítása ezen kérdések megválaszolásával:

- Van szüksége, amely ki tud szolgálni az adatok elérési útnak gyakran használt adatok tárolási szolgáltató? Ha igen, szűkítheti a beállítások az adott sebesség szolgáló réteg vannak optimalizálva.

- Tegye meg kell nagymértékben párhuzamos feldolgozási (MPP) támogatás, ahol lekérdezéseket a rendszer automatikusan terjeszt több folyamatok vagy csomóponton keresztül? Ha igen, választhatja ki, amely támogatja a lekérdezés kibővítési.

- Szeretné használni a relációs adattároló? Ha igen, szűkítenie a lehetőségeit a relációs adatbázis-modell rendelkező. Azonban ügyeljen arra, hogy néhány nem relációs áruház lekérdezése SQL-szintaxis támogatásához, és az eszközök, például a PolyBase segítségével nem relációs adatokat kérdez.

## <a name="capability-matrix"></a>Képesség mátrix

A következő táblázat összefoglalja a főbb változásai képességeit.

### <a name="general-capabilities"></a>Általános képességek

| | SQL Database | SQL Data Warehouse | A HDInsight HBase/Phoenix | A HDInsight LLAP struktúra | Azure Analysis Services | Cosmos DB |
| --- | --- | --- | --- | --- | --- | --- |
| Van a felügyelt | Igen | Igen | Igen <sup>1</sup> | Igen <sup>1</sup> | Igen | Igen |
| Elsődleges adatbázis-modell | Relációs (oszlopos formátum oszlopcentrikus indexek használata esetén) | Relációs táblák oszlopos tárolóval | Széles oszlop tároló | Hive/a memóriában | Táblázatos/MOLAP szemantikai modellek | Dokumentálja a tároló, a graph, a kulcs-érték tároló, a széles oszlop tároló |
| SQL nyelvi támogatás | Igen | Igen | Igen (használatával [Phoenix](http://phoenix.apache.org/) JDBC-illesztőt) | Igen | Nem | Igen |
| Gyorsaságot szolgáló réteg | Igen <sup>2</sup> | Nem | Igen | Igen | Nem | Igen |

[1], kézi konfigurálás és a méretezés.

A(z) [2] Using memóriaoptimalizált táblák és a kivonatoló vagy fürtözetlen indexeire.
 
### <a name="scalability-capabilities"></a>Méretezhetőség képességek

|                                                  | SQL Database | SQL Data Warehouse | A HDInsight HBase/Phoenix | A HDInsight LLAP struktúra | Azure Analysis Services | Cosmos DB |
|--------------------------------------------------|--------------|--------------------|----------------------------|------------------------|-------------------------|-----------|
| A magas rendelkezésre állás érdekében redundáns területi kiszolgálók |     Igen      |        Igen         |            Igen             |           Nem           |           Nem            |    Igen    |
|             Támogatja a lekérdezés kibővítési             |      Nem      |        Igen         |            Igen             |          Igen           |           Igen           |    Igen    |
|          Dinamikus méretezhetőség (felskálázott)          |     Igen      |        Igen         |             Nem             |           Nem           |           Igen           |    Igen    |
|        Támogatja a memórián belüli gyorsítótárazáshoz, az adatok        |     Igen      |        Igen         |             Nem             |          Igen           |           Igen           |    Nem     |

### <a name="security-capabilities"></a>Biztonsági képességei

| | SQL Database | SQL Data Warehouse | A HDInsight HBase/Phoenix | A HDInsight LLAP struktúra | Azure Analysis Services | Cosmos DB |
| --- | --- | --- | --- | --- | --- | --- |
| Hitelesítés  | SQL / Azure Active Directory (Azure AD) | SQL / Azure AD | helyi és az Azure AD <sup>1</sup> | helyi és az Azure AD <sup>1</sup> | Azure AD | adatbázis-felhasználók / keresztül hozzáférést az Azure AD-vezérlőt (IAM) |
| Adat-titkosítás inaktív állapotban | Igen <sup>2</sup> | Igen <sup>2</sup> | Igen <sup>1</sup> | Igen <sup>1</sup> | Igen | Igen |
| Sorszintű biztonság | Igen | Nem | Igen <sup>1</sup> | Igen <sup>1</sup> | Igen (keresztül objektumszintű biztonsági modell) | Nem |
| Támogatja a tűzfalak | Igen | Igen | Igen <sup>3</sup> | Igen <sup>3</sup> | Igen | Igen |
| Dinamikus adatmaszkolás | Igen | Nem | Igen <sup>1</sup> | Igen * | Nem | Nem |

[1] igényel a használata egy [HDInsight-fürt tartományhoz](/azure/hdinsight/domain-joined/apache-domain-joined-introduction).

[2] átlátható adattitkosítás (TDE) segítségével az inaktív adatok titkosításához és visszafejtéséhez szükséges.

[3] az Azure Virtual Network használatakor. Lásd: [kiterjesztése Azure HDInsight Azure virtuális hálózat használatával](/azure/hdinsight/hdinsight-extend-hadoop-virtual-network).
