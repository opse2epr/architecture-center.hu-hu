---
title: Online tranzakciófeldolgozás (OLTP)
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.openlocfilehash: be24bc173359539785385de4a188e7536f6d2ffe
ms.sourcegitcommit: e7e0e0282fa93f0063da3b57128ade395a9c1ef9
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/05/2018
ms.locfileid: "52902756"
---
# <a name="online-transaction-processing-oltp"></a>Online tranzakciófeldolgozás (OLTP)

Tranzakciós adatok használ a számítógéprendszerek kezelését a neve, Online tranzakciófeldolgozási (OLTP). OLTP rendszerek regisztrálhatna üzleti fordulnak elő a szervezet napi működését, és ezeket az adatokat a következtetések levonásához lekérdezését támogatja.

## <a name="transactional-data"></a>Tranzakciós adatok

Tranzakciós adatok, amely nyomon követi a kapcsolati, a szervezet tevékenységekkel kapcsolatos információk. Ezek a kapcsolati jellemzően üzleti tranzakciók, például az ügyfelektől, a szállítók, termékek, szolgáltatások fogadott vagy a leltár-, végrehajtott rendelések Mozgatott kifizetéseket kapott. Tranzakciós események, amelyek magukat a tranzakciók képviselnek, általában tartalmaznak egy time dimenzió, bizonyos számértékeket és egyéb adatok mutató hivatkozásokat. 

Tranzakciók általában kell lennie *atomi* és *konzisztens*. Atomitást azt jelenti, hogy a teljes tranzakció mindig sikeres vagy sikertelen lesz a munka egy egységként, és soha nem egy félig kitöltött állapotban marad. A tranzakció nem hajtható végre, ha az adatbázis-rendszer vissza kell görgetni a tranzakció részeként már végrehajtott lépéseket. A hagyományos RDBMS a visszaállítás automatikusan megtörténik, ha egy tranzakció nem hajtható végre. Konzisztencia azt jelenti, hogy tranzakciók mindig hagyja meg az adatok érvényes állapotban. (Ezek a atomitást és konzisztencia nagyon informális leírását. Nincsenek formális definíciók ezeket a tulajdonságokat, mint például [ACID](https://en.wikipedia.org/wiki/ACID).)

Tranzakciós adatbázisok támogathat erős konzisztencia használatával különféle zárolási pesszimista zárolással, például annak biztosításához, hogy az összes adat a vállalati kontextusában erősen konzisztens tranzakciók, minden felhasználó és folyamat számára. 

A leggyakoribb üzembe helyezési architektúra által használt tranzakciós adatait az adatréteg store a 3 szintű architektúrában. A 3-rétegű architektúra általában egy bemutatási, üzleti logikai rétegből és adatokat tároló szint áll. Egy kapcsolódó üzembe helyezési architektúra a [N szintű](/azure/architecture/guide/architecture-styles/n-tier) architektúra, amely előfordulhat, hogy több közel-csomagok kezelése üzleti logikát.

## <a name="typical-traits-of-transactional-data"></a>Tipikus képességekre vonatkozó tranzakciós adatok

Tranzakciós adatok általában a következő jellemzőkkel rendelkezik:

| Követelmény | Leírás |
| --- | --- |
| Normalizálási | Magas normalized |
| Séma | Séma, erősen kényszerítése|
| Konzisztencia | Garantálja a konzisztenciát, ACID |
| Integritás | Nagy fokú integritása |
| Használja a tranzakciók | Igen |
| Zárolási stratégia | A pesszimista vagy optimista|
| Frissíthető | Igen |
| Appendable | Igen |
| Számítási feladat | Mérsékelt nagy írási beolvasása |
| Indexelés | Az elsődleges és másodlagos indexek |
| Datum mérete | Kis és közepes méretű |
| Modell | Relációs |
| Adatalakzat | Táblázatos |
| Rugalmas lekérdezés | Rendkívül rugalmas |
| Méretezés | Kis (MB) és nagy (több TB-osra bővül) | 

## <a name="when-to-use-this-solution"></a>Ez a megoldás használata

Válassza ki a OLTP, ha hatékony feldolgozására és tárolhatja az üzleti tranzakciók, és azonnal elérhetővé tétele az ügyfélalkalmazások konzisztens módon kell. Akkor használja ezt az architektúrát, ha bármilyen képzés résztvevői hasznos képességekkel feldolgozási késedelem negatív hatással lenne a mindennapos az üzleti.

OLTP rendszerek hatékonyan dolgozza fel, és a tranzakciók, valamint lekérdezési tranzakciós adatok tárolására tervezték. A cél az, hogy hatékonyan feldolgozása, és tárolja az egyes tranzakciók szerint az OLTP rendszerek részben kapcsolódással adatok normalizálási &mdash; , felosztani az adatokat, amelyek kisebb redundáns szeletekre. Támogatja a hatékonyságot az OLTP rendszerek nagy számú tranzakció egymástól függetlenül feldolgozni, mert, és ezzel elkerülheti a további feldolgozás szükséges zajok mellett a redundáns adatok adatok integritásának fenntartása.

## <a name="challenges"></a>Problémák
Végrehajtási és az OLTP rendszerek használatával hozhat létre néhány kihívásokat:

- OLTP rendszerek általában nem mindig jó az összesítések kezelése keresztül nagy mennyiségű adatot, bár vannak kivételek, például egy jól megtervezett SQL Server-alapú megoldás. Elemzés, adatokra támaszkodó összesített számítások az egyes tranzakciók több mint egy millió, olyan rendkívül erőforrás-igényes az OLTP rendszerek. Lassú végrehajtásához, és képes lehet egy slow-down okozhat más tranzakciók az adatbázis blokkolásával.
- Ha analytics vezető, és az adatok nagymértékben normalizáltak reporting, a lekérdezések általában összetett, mert irányuló legtöbb lekérdezésnek kell megszüntetéséhez normalizálása összekapcsolások használatával az adatokat. Ezenkívül elnevezési konvenciói az adatbázis-objektumok OLTP-rendszerekben általában tömör és állapotára. A nagyobb normalizálási tömör elnevezési konvenciók szolgáltatással párosítva az üzleti felhasználók lekérdezéséhez, a DBA vagy adatok fejlesztő segítsége nélkül megnehezíti OLTP rendszerek.
- Lassú lekérdezések teljesítményét, a tárolt tranzakciók számától függően a korábbi tranzakciók határozatlan ideig tárolja, és a táblában túl sok adat tárolására vezethet. A leggyakoribb megoldás az, hogy egy megfelelő idő (például az aktuális pénzügyi év) ablakban az OLTP rendszerek karbantartása és más rendszerekre, például egy data martba korábbi adatok kiszervezése vagy [adatraktár](./data-warehousing.md).

## <a name="oltp-in-azure"></a>Az Azure-ban tárolt OLTP-k

Például a webhelyeken üzemeltetett alkalmazások [App Service Web Apps](/azure/app-service/app-service-web-overview), az App Service-ben futó REST API-k vagy mobil- vagy asztali alkalmazások az OLTP rendszerek általában egy REST API-közvetítő segítségével kommunikálnak.

A gyakorlatban a legtöbb számítási feladatok nem állnak tisztán OLTP. Nincs általában egy analitikai összetevőnek. Emellett nincs valós idejű jelentéskészítést, például a jelentések futtatása ellen a műveleti rendszerfiókok iránti kereslet. Ez is nevezik HTAP (hibrid tranzakciós és analitikai feldolgozása). További információkért lásd: [Online Analytical Processing (OLAP)](./online-analytical-processing.md).

Az Azure-ban mind a következő adattárakat felel meg az OLTP alapvető követelményei és a tranzakciós adatok kezelésére:

- [Azure SQL Database](/azure/sql-database/)
- [Az SQL Server Azure virtuális gépként](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-server-iaas-overview?toc=%2Fazure%2Fvirtual-machines%2Fwindows%2Ftoc.json)
- [Azure Database for MySQL](/azure/mysql/)
- [Azure Database for PostgreSQL](/azure/postgresql/)

## <a name="key-selection-criteria"></a>Fontosabb kritériumok

Így szűkítheti, első lépésként a kérdések megválaszolása:

- Szeretne egy felügyelt szolgáltatás, hanem a saját kiszolgálók kezelése?

- Rendelkezik a megoldás a Microsoft SQL Server, MySQL, illetve PostgreSQL kompatibilitás adott függőségek? Az alkalmazás korlátozhatják az illesztőprogramokat támogatja az adattárban való kommunikációhoz, illetve arról, hogy mely adatbázissal rendelkezik teszi feltételezések alapján is tárolja az adatokat.

- Az írási átviteli sebességet megkövetelő különösen magas vannak? Ha igen, válasszon olyan beállítás, amely a táblák biztosít. 

- A megoldás több-bérlős rendszer? Ha igen, fontolja meg a beállításokat, amelyek támogatják a kapacitás-készletek, ahol több adatbázis-példány felhasználhatja a rugalmas készletek az erőforrások fix erőforrások adatbázisonként helyett. Ez segít jobban minden adatbázis-példány között oszthatja el a kapacitás, és lehetővé teszi a megoldás költséghatékonyabb.

- Az adatok, így több régióban is alacsony késleltetéssel nem kell? Ha igen, válasszon egy beállítást, amely támogatja az olvasható másodlagos replikával.

- Nem az adatbázis kell magas rendelkezésre állású geo-kép régióban? Ha igen, válasszon egy beállítást, amely támogatja a földrajzi replikáció. Is figyelembe kell venni a beállításokat, amelyek támogatják az Automatikus feladatátvétel az elsődleges replikáról egy másodlagos replikára.

- Rendelkezik-e az adatbázis konkrét igényeinek? Ha igen, vizsgálja meg a beállításokat, amelyek biztosítanak, mint például a sorszintű biztonság, adatmaszkolást és átlátható adattitkosítást.

## <a name="capability-matrix"></a>Képességmátrix

A következő táblázat összefoglalja a fő különbségeket, a képességek.

### <a name="general-capabilities"></a>Általános képességek 

|                              | Azure SQL Database | Az SQL Server Azure virtuális gépként | Azure Database for MySQL | Azure Database for PostgreSQL |
|------------------------------|--------------------|----------------------------------------|--------------------------|-------------------------------|
|      A felügyelt szolgáltatás      |        Igen         |                   Nem                   |           Igen            |              Igen              |
|       A platformon fut       |        –         |         Windows, Linux, Docker         |           –            |              –              |
| Programozhatóság <sup>1</sup> |   T-SQL HASZNÁLATÁVAL, LEGYEN AZ .NET, R   |         T-SQL használatával, legyen az .NET, R, Python         |  T-SQL használatával, legyen az .NET, R, Python  |              SQL              |

[1] nem többek között az ügyfelek illesztőprogram támogatása, amely lehetővé teszi számos programozási nyelvet való csatlakozáshoz, és az OLTP-adattárban.

### <a name="scalability-capabilities"></a>Skálázhatósági képességeket.

| | Azure SQL Database | Az SQL Server Azure virtuális gépként| Azure Database for MySQL | Azure Database for PostgreSQL|
| --- | --- | --- | --- | --- | --- |
| Maximális példányméret | [4 TB](/azure/sql-database/sql-database-resource-limits) | 256 TB | [1 TB](/azure/mysql/concepts-limits) | [1 TB](/azure/postgresql/concepts-limits) |
| Kapacitás-készleteket támogat  | Igen | Igen | Nem | Nem |
| Támogatja a fürtök horizontális felskálázás  | Nem | Igen | Nem | Nem |
| A dinamikus méretezhetőség (vertikális felskálázási)  | Igen | Nem | Igen | Igen |

### <a name="analytic-workload-capabilities"></a>Elemzési számítási feladatok képességek

| | Azure SQL Database | Az SQL Server Azure virtuális gépként| Azure Database for MySQL | Azure Database for PostgreSQL|
| --- | --- | --- | --- | --- | --- | 
| Historikus táblák | Igen | Igen | Nem | Nem |
| A memóriában (memóriaoptimalizált) táblák | Igen | Igen | Nem | Nem |
| Oszlopcentrikus támogatása | Igen | Igen | Nem | Nem |
| Adaptív lekérdezés-feldolgozás | Igen | Igen | Nem | Nem |

### <a name="availability-capabilities"></a>Rendelkezésre állás

| | Azure SQL Database | Az SQL Server Azure virtuális gépként| Azure Database for MySQL | Azure Database for PostgreSQL|
| --- | --- | --- | --- | --- | --- | 
| Olvasható másodlagos példánnyal | Igen | Igen | Nem | Nem | 
| Földrajzi replikáció | Igen | Igen | Nem | Nem | 
| Másodlagos automatikus feladatátvétel | Igen | Nem | Nem | Nem|
| Adott időpontnak megfelelő helyreállítás | Igen | Igen | Igen | Igen |

### <a name="security-capabilities"></a>Biztonsági képességei

|                                                                                                             | Azure SQL Database | Az SQL Server Azure virtuális gépként | Azure Database for MySQL | Azure Database for PostgreSQL |
|-------------------------------------------------------------------------------------------------------------|--------------------|----------------------------------------|--------------------------|-------------------------------|
|                                             Sorszintű biztonság                                              |        Igen         |                  Igen                   |           Igen            |              Igen              |
|                                                Adatmaszkolás                                                 |        Igen         |                  Igen                   |            Nem            |              Nem               |
|                                         Transzparens adattitkosítás                                         |        Igen         |                  Igen                   |           Igen            |              Igen              |
|                                  Adott IP-címek való hozzáférés korlátozása                                   |        Igen         |                  Igen                   |           Igen            |              Igen              |
|                                  Korlátozhatja a hozzáférést, csak a virtuális hálózatok közötti hozzáférés engedélyezése                                  |        Igen         |                  Igen                   |            Nem            |              Nem               |
|                                    Hitelesítés Azure Active Directory-fiókkal                                    |        Igen         |                  Igen                   |            Nem            |              Nem               |
|                                       Active Directory-alapú hitelesítés                                       |         Nem         |                  Igen                   |            Nem            |              Nem               |
|                                         Multi-Factor Authentication                                         |        Igen         |                  Igen                   |            Nem            |              Nem               |
| Támogatja a [Always Encrypted](/sql/relational-databases/security/encryption/always-encrypted-database-engine) |        Igen         |                  Igen                   |           Igen            |              Nem               |
|                                                 Magánhálózati IP                                                  |         Nem         |                  Igen                   |           Igen            |              Nem               |

