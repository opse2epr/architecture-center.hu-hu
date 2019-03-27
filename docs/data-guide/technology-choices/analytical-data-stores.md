---
title: Az analitikai adattár kiválasztása
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.openlocfilehash: 236f5eaffffa8eb1206f13f3eb7fb57828f0a12d
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299249"
---
# <a name="choosing-an-analytical-data-store-in-azure"></a>Az Azure-ban egy analitikai adattár kiválasztása

Az egy [big Data típusú adatok](../big-data/index.md) architecture, gyakran van szükség az olyan analitikai adatokat tárolja a feldolgozott szolgál egy strukturált formában, hogy lekérdezhetők legyenek elemzőeszközökkel. Analitikus adattárak mindkét ritkáról gyakori elérésű útvonal a támogatási lekérdezése és ritka elérésű útvonal adatok együttesen nevezzük a kiszolgálóréteget, vagy az adatok tárolására szolgáló.

Feldolgozott adatok a gyakori elérésű útvonal és a ritka elérésű útvonal a kiszolgálórétegbe foglalkozik. Az a [lambda architektúra](../big-data/index.md#lambda-architecture), a kiszolgálórétegbe oszlik egy _sebesség szolgáló_ rétege, amely tárolja az adatokat növekményes módon feldolgozott, és a egy _batch-kiszolgáló_rétege, amely tartalmazza a kötegelt feldolgozásra kimeneti. A kiszolgálórétegbe véletlenszerű olvasást és kis késésű erős támogatását igényli. A gyors réteg az adattárolás támogatnia kell a is véletlenszerű írásokat, mert batch adatbetöltéshez a store-bA be fogja vezetni a nem várt késedelmeket. Másrészről adattárolás a kötegelt réteg nem kell a véletlenszerű írások támogatására, de ehelyett írja a batch.

Nincs egyetlen data management ideális megoldássá az összes tárolási feladatok van. Más felügyeleti megoldásokkal a különböző feladatok vannak optimalizálva. A legtöbb valós felhőalapú alkalmazások és folyamatok big Data típusú adatok számos különböző adattárolási követelmények, és a gyakran használt adatok tárolási megoldások kombinációját.

## <a name="what-are-your-options-when-choosing-an-analytical-data-store"></a>Mik azok a beállítások egy analitikai adattár kiválasztásakor?

Az adatok tárolási szolgáltató az Azure-ban, igényeitől függően több lehetőség áll rendelkezésre:

- [SQL Data Warehouse](/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is)
- [Azure SQL Database](/azure/sql-database/)
- [Az SQL Server Azure virtuális Gépen](/sql/sql-server/sql-server-technical-documentation)
- [A HDInsight HBase/Phoenix](/azure/hdinsight/hbase/apache-hbase-overview)
- [Hive LLAP, a HDInsight](/azure/hdinsight/interactive-query/apache-interactive-query-get-started)
- [Az Azure Analysis Services](/azure/analysis-services/analysis-services-overview)
- [Azure Cosmos DB](/azure/cosmos-db/)

Ezek a beállítások adja meg a különböző adatbázis-modellek, amelyek a különböző típusú feladatok vannak optimalizálva:

- [Kulcs/érték](https://msdn.microsoft.com/library/dn313285.aspx#sec7) adatbázisok tárolásához egyetlen szerializált objektum minden egyes kulcs érték. Készen nagy mennyiségű adatot, ahol szeretne kapni egy elemet a megadott kulcs értékét, és nincs tárolására szolgáló lekérdezés elem más tulajdonságok alapján.
- [A dokumentum](https://msdn.microsoft.com/library/dn313285.aspx#sec8) adatbázisok találhatók, amelyben az értékek a következők kulcs/érték adatbázis *dokumentumok*. A "dokumentumok" Ebben a környezetben olyan elnevezett mezőkből és értékek gyűjteménye. Általában az adatbázis tárolja az adatokat egy formátumban, például XML, YAML, JSON vagy BSON, de előfordulhat, hogy használja az egyszerű szöveges. A dokumentum adatbázisok nem kulcs mezők lekérdezést, és a másodlagos indexeket, a hatékonyabb lekérdezés meghatározása. Ez lehetővé teszi a dokumentum-adatbázis bonyolultabb, mint a dokumentum-kulcsnak az értéke feltételek alapján adatokat igénylő alkalmazásokhoz megfelelő. Ha például kérdezheti a mezők, például a termék azonosítója, az ügyfél-azonosító vagy az ügyfél neve.
- [Oszlopcsalád-](https://msdn.microsoft.com/library/dn313285.aspx#sec9) adatbázisok az olyan kulcs-érték adattárak, kapcsolódó oszlopok oszlopcsaláddal nevű gyűjteményekbe adattárolás struktúra. Például népszámlálási adatbázis lehet egy csoportot az oszlopok egy személy neve (először középen, utolsó), a személy címe egy csoportot, és a egy csoportot az a személy profiladatokat (születési idő, a Nemet adatok). Az adatbázis tárolhatja minden oszlopcsalád egy külön partícióban, miközben az összes adat egy személy ugyanazzal a kulccsal kapcsolatos. Egy alkalmazás egyetlen oszlopcsalád olvashatja az adatokat egy entitás összes elolvasása nélkül.
- [Graph](https://msdn.microsoft.com/library/dn313285.aspx#sec10) adatbázisok tárolhatók objektumok és kapcsolatok gyűjteménye. A gráfadatbázis hatékonyan hajthat végre lekérdezéseket, amelyek objektumokat és kapcsolatokat hálózatát bejáró közöttük. Például lehet, hogy az objektumok egy emberi erőforrások adatbázisban az alkalmazottak, és érdemes megkönnyítése érdekében lekérdezések például "található összes alkalmazottait közvetlenül vagy közvetve Scott."

## <a name="key-selection-criteria"></a>Fontosabb kritériumok

Így szűkítheti, első lépésként a kérdések megválaszolása:

- Szükség van, amely az adatok egy gyakori elérésű útvonal alapul szolgálhat tárolási szolgáltató? Ha igen, szűkítheti a lehetőségek, amelyek az a sebesség kiszolgálórétegbe vannak optimalizálva.

- Ehhez meg kell nagymértékben párhuzamos feldolgozási (MPP) támogatást, ahol lekérdezéseket automatikusan között oszlanak meg több folyamatok vagy a csomópontok? Ha igen, válasszon egy beállítást, amely támogatja a lekérdezés horizontális felskálázást.

- Inkább használja a relációs adattároló? Ha igen, szűkítheti a lehetőségeit a azokat egy relációs adatbázis-modellel. Azonban vegye figyelembe, hogy néhány nem relációs adattárak támogatja-e az SQL-szintaxis, és az eszközök, például a PolyBase segítségével nem relációs adattárak lekérdezése.

## <a name="capability-matrix"></a>Képességmátrix

A következő táblázat összefoglalja a fő különbségeket, a képességek.

### <a name="general-capabilities"></a>Általános képességek

| | SQL Database | SQL Data Warehouse | A HDInsight HBase/Phoenix | Hive LLAP, a HDInsight | Azure Analysis Services | Cosmos DB |
| --- | --- | --- | --- | --- | --- | --- |
| A felügyelt szolgáltatás | Igen | Igen | Igen <sup>1</sup> | Igen <sup>1</sup> | Igen | Igen |
| Elsődleges adatbázismodell | Relációs (Oszlopalapú formátum az oszlopcentrikus indexek használata esetén) | Oszlopos szerkezetű relációs tábláival | Széles körű oszloptár | Hive és a memóriában | Szemantikai modellek táblázatos/MOLAP | Dokumentum-tároló, gráf, kulcs-érték tároló, széles oszloptár |
| SQL nyelvi támogatás | Igen | Igen | Igen (használatával [Phoenix](https://phoenix.apache.org/) JDBC-illesztőprogram) | Igen | Nem | Igen |
| Réteg szolgáltató gyorsaságot | Igen <sup>2</sup> | Nem | Igen | Igen | Nem | Igen |

[1] a kézi konfigurálás és a méretezést.

A(z) [2] a memóriaoptimalizált táblák és kivonatoló vagy fürtözetlen index.
 
### <a name="scalability-capabilities"></a>Skálázhatósági képességeket.

|                                                  | SQL Database | SQL Data Warehouse | A HDInsight HBase/Phoenix | Hive LLAP, a HDInsight | Azure Analysis Services | Cosmos DB |
|--------------------------------------------------|--------------|--------------------|----------------------------|------------------------|-------------------------|-----------|
| Redundáns regionális kiszolgálók magas rendelkezésre állás érdekében |     Igen      |        Igen         |            Igen             |           Nem           |           Nem            |    Igen    |
|             Támogatja a lekérdezés horizontális felskálázás             |      Nem      |        Igen         |            Igen             |          Igen           |           Igen           |    Igen    |
|          A dinamikus méretezhetőség (vertikális felskálázási)          |     Igen      |        Igen         |             Nem             |           Nem           |           Igen           |    Igen    |
|        Támogatja a memórián belüli gyorsítótárazáshoz, az adatok        |     Igen      |        Igen         |             Nem             |          Igen           |           Igen           |    Nem     |

### <a name="security-capabilities"></a>Biztonsági képességek

| | SQL Database | SQL Data Warehouse | A HDInsight HBase/Phoenix | Hive LLAP, a HDInsight | Azure Analysis Services | Cosmos DB |
| --- | --- | --- | --- | --- | --- | --- |
| Authentication  | SQL / Azure Active Directory (Azure AD) | SQL / Azure AD | helyi / Azure AD <sup>1</sup> | helyi / Azure AD <sup>1</sup> | Azure AD | adatbázis-felhasználók / hozzáférést az Azure AD-vezérlőt (IAM) |
| Adat-titkosítás inaktív állapotban | Igen <sup>2</sup> | Igen <sup>2</sup> | Igen <sup>1</sup> | Igen <sup>1</sup> | Igen | Igen |
| Sorszintű biztonság | Igen | Nem | Igen <sup>1</sup> | Igen <sup>1</sup> | Igen (objektumszintű biztonság a modellben) keresztül | Nem |
| Támogatja a tűzfalak | Igen | Igen | Igen <sup>3</sup> | Igen <sup>3</sup> | Igen | Igen |
| Dinamikus adatmaszkolás | Igen | Nem | Igen <sup>1</sup> | Igen * | Nem | Nem |

[1] használata szükséges egy [tartományhoz csatlakoztatott HDInsight-fürt](/azure/hdinsight/domain-joined/apache-domain-joined-introduction).

[2] transzparens adattitkosítás (TDE) segítségével az inaktív adatok titkosításához és visszafejtéséhez szükséges.

[3] egy Azure virtuális hálózaton belül. Lásd: [kiterjesztése az Azure HDInsight használata az Azure Virtual Network](/azure/hdinsight/hdinsight-extend-hadoop-virtual-network).
