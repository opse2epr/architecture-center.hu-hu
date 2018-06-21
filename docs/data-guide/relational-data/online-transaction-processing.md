---
title: Online tranzakciófeldolgozás (OLTP)
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 8650b919fc1a59240343015493a1fe41c8729a72
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/06/2018
ms.locfileid: "30848699"
---
# <a name="online-transaction-processing-oltp"></a>Online tranzakciófeldolgozás (OLTP)

A felügyeleti használ a számítógéprendszerek tranzakciós adatok hivatkozik, Online tranzakciófeldolgozási (OLTP). Az OLTP rendszerek regisztrálhatna üzleti fordulnak elő a szervezet napi működését, és támogatja az adatokba, így következtetéseket lekérdezését.

## <a name="transactional-data"></a>Tranzakciós adatok

Tranzakciós adatok, amely nyomon követi a kapcsolati, a szervezetek tevékenységekkel kapcsolatos információkat. A kapcsolati jellemzően üzleti tranzakciók, például az ügyfelektől, szállítók, a leltározást, végrehajtott rendeléseket és a szolgáltatások kézbesíteni való áthaladás termékek kifizetéseket kapott. Tranzakciós események, amelyek megfelelnek a tranzakciók magukat, általában a egy time dimenzió, bizonyos számértékeket és egyéb adatok mutató hivatkozásokat tartalmaz. 

Tranzakciók általában kell lenniük *atomi* és *konzisztens*. Atomicity azt jelenti, hogy a teljes tranzakció mindig sikeres vagy sikertelen munka egy egységként, soha nem egy félig kitöltött állapotban marad. A tranzakció nem hajtható végre, ha az adatbázis-rendszer kell visszaállítása volt meg, hogy a tranzakció részeként műveleteket. A hagyományos RDBMS a visszaállítás automatikusan történik, a tranzakció nem hajtható végre, ha. Konzisztencia azt jelenti, hogy tranzakciók mindig hagyja meg az adatokat állapota érvénytelen. (Ezek a atomicity és konzisztencia nagyon kötetlen leírását. Nincsenek formális definíciója ezeket a tulajdonságokat, például a [sav](https://en.wikipedia.org/wiki/ACID).)

Tranzakciós adatbázisok is támogatja az erős konzisztencia, különböző zárolási, például pesszimista zárolás használatával gondoskodjon arról, hogy minden adatot a vállalati keretén belül kifejezetten konzisztens tranzakciókhoz, a felhasználók és a folyamatok. 

A leggyakoribb üzembe helyezési architektúrája tranzakciós adatait használó az adatréteg tároló 3-rétegű architektúra. A 3-rétegű architektúra jellemzően a bemutatási szint, üzleti logikai rétegből és adatok tárolási réteg áll. A kapcsolódó központi telepítési architektúrája a [N szintű](/azure/architecture/guide/architecture-styles/n-tier) architektúra, amely több közel-Rétegek kezelése üzleti logikát.

## <a name="typical-traits-of-transactional-data"></a>Tipikus jellemzők tranzakciós adatok

Tranzakciós adatok általában a következő jellemzőkkel rendelkezik:

| Követelmény | Leírás |
| --- | --- |
| Normalizálási | Magas normalizált |
| Séma | Írás, erősen kényszerített séma|
| Konzisztencia | Az erős konzisztencia sav biztosítja, hogy |
| Integritás | Integritása magas szintű |
| Használja a tranzakciók | Igen |
| Zárolási stratégia | Pesszimista vagy optimista|
| Updateable | Igen |
| Appendable | Igen |
| Számítási feladat | Nagy mennyiségű írási műveletekről, mérsékelt olvassa be |
| Indexelés | Elsődleges és másodlagos indexek |
| Datum mérete | Kis, közepes méretű |
| Modell | Relációs |
| Adatok alakzat | A táblázatos |
| Lekérdezés rugalmasságot | Rugalmas |
| Méretezés | Kis (MB) és nagy (néhány több TB-nyi) | 

## <a name="when-to-use-this-solution"></a>Ez a megoldás használatával

Ha hatékony feldolgozni, és üzleti tranzakciók tárolja, és azonnal elérhetővé tétele az ügyfélalkalmazások konzisztens módon van szüksége, válassza a OLTP. Használja az ebbe az architektúrába, ha bármilyen anyagi feldolgozási késedelem hatással lenne egy negatív az üzleti mindennapi műveleteinek.

OLTP rendszerek hatékonyan feldolgozni, és a tranzakciók, valamint a lekérdezés tranzakciós adatok tárolására készültek. A cél hatékonyan feldolgozási és tárolása az egyes tranzakciók által az OLTP rendszerek részben úgy érhető el, adatok normalizálási &mdash; Ez azt jelenti, hogy ossza az adatok kisebb csoportjai, amelyek kevesebb redundáns. Támogatja a hatékonyságát, mivel lehetővé teszi, hogy az OLTP rendszerek nagyszámú tranzakciók egymástól függetlenül feldolgozni, és elkerülhető a felesleges feldolgozási szükséges redundáns adatok mellett adatok integritásának fenntartása.

## <a name="challenges"></a>Problémák
Végrehajtási és az OLTP rendszerek használata néhány kihívást hozhat létre:

- Az OLTP rendszerek megfelelőek nem mindig összesítések kezelése keresztül nagy mennyiségű adat, bár kivételek, például egy tervezett jól az SQL Server-alapú megoldás. Az adatok, támaszkodó összesített számítások az egyes tranzakciók több mint több millió, alapján Analytics nagyon erőforrás-igényes az OLTP rendszerek. Lassú hajtható végre, és lehet el egy slow-down következtében letiltásával más tranzakciók az adatbázisban.
- Ha elemzés végrehajtása, és az adatok magas normalizált jelentéskészítés, a lekérdezések általában összetett, mert irányuló legtöbb lekérdezésnek kell deszerializálni optimalizálására illesztések használatával az adatok. Emellett az OLTP rendszerek adatbázis-objektumok elnevezési szabályai általában tömör és állapotára. A nagyobb normalizálási alapján tömör elnevezési konvenciók kialakulhat nehezebben OLTP rendszerek üzleti felhasználók DBA vagy az adatok a fejlesztők az nélkül lekérdezéséhez.
- Lassú a lekérdezési teljesítményt, attól függően, hogy a tárolt tranzakciók száma tranzakciók előzményeinek határozatlan ideig tárolja, és túl sok adat tárolása táblában vezethet. A gyakori megoldás, hogy az OLTP rendszerben (például az aktuális pénzügyi év) megfelelő ablakban karbantartása és egyéb rendszerekre, például egy adatpiacot korábbi adatok továbbítása vagy [adatraktár](./data-warehousing.md).

## <a name="oltp-in-azure"></a>OLTP az Azure-ban

Alkalmazások, mint az üzemeltetett webhelyek [App Service Web Apps](/azure/app-service/app-service-web-overview), az App Service-ben futó REST API-k vagy hordozható vagy asztali alkalmazások az OLTP rendszerek általában a REST API-t a közvetítő keresztül kommunikálnak.

A gyakorlatban a munkaterhelések többségéhez nincsenek tisztán OLTP. Nincs általában egy analitikai összetevője. Emellett nincs valós idejű jelentéskészítést, például a jelentések futtatása ellen a műveleti rendszerfiókok iránti kereslet. Ez más néven a HTAP (hibrid tranzakciós és Analytical Processing). További információkért lásd: [Online Analytical Processing (OLAP)](./online-analytical-processing.md).

Az Azure a következő adatokat tároló összes felel meg a OLTP alapvető követelményei és a tranzakciós adatokat:

- [Azure SQL Database](/azure/sql-database/)
- [SQL Server egy Azure virtuális gépen](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-server-iaas-overview?toc=%2Fazure%2Fvirtual-machines%2Fwindows%2Ftoc.json)
- [Azure Database for MySQL](/azure/mysql/)
- [Azure Database for PostgreSQL](/azure/postgresql/)

## <a name="key-selection-criteria"></a>Kulcs kiválasztási feltételek

A választási lehetőségek szűkítéséhez indítása ezen kérdések megválaszolásával:

- A saját kiszolgálóit kezelése helyett egy felügyelt szolgáltatás van szüksége?

- Rendelkezik a megoldás a Microsoft SQL Server, MySQL vagy PostgreSQL kompatibilitás adott függőségek? Az alkalmazás az adatokat tárolja, dönthet úgy, az illesztőprogramokat az támogatja-e az adattár való kommunikáció vagy adatbázis szolgál arról, hogy mely teszi feltételek alapján korlátozhatja.

- Ezek a írási átviteli követelmények különösen nagy? Ha igen, válassza ki, amely a táblák biztosít. 

- A megoldás több-bérlős van? Ha igen, vegye figyelembe a kapacitás-készletek, ahol több adatbázis-példány tárolt erőforrások helyett adatbázisonként rögzített erőforrások rugalmas készletek támogatott beállításokat. Ez segít jobban kapacitás el az összes adatbázis-példány, és lehetővé teszi a megoldás költséghatékonyabb.

- Az adatokat, így több régióba kis késéssel nem kell? Ha igen, válassza ki, amely támogatja az olvasható másodlagos másodpéldányokra.

- Nem az adatbázis kell magas rendelkezésre állású grafikus földrajzi régiók között? Ha igen, válassza ki, amely támogatja a földrajzi replikációját. Is gondolja át a beállításokat, amelyek támogatják az Automatikus feladatátvétel az elsődleges replikáról egy másodlagos replikára.

- Rendelkezik-e az adatbázis konkrét igényeinek? Ha igen, ellenőrizze a beállításokat, például sorszintű biztonság van, az adatok maszkolása és az átlátható adattitkosítás képességeket biztosítják.

## <a name="capability-matrix"></a>Képesség mátrix

A következő táblázat összefoglalja a főbb változásai képességeit.

### <a name="general-capabilities"></a>Általános képességek 

|                              | Azure SQL Database | SQL Server egy Azure virtuális gépen | Azure Database for MySQL | Azure Database for PostgreSQL |
|------------------------------|--------------------|----------------------------------------|--------------------------|-------------------------------|
|      Van a felügyelt      |        Igen         |                   Nem                   |           Igen            |              Igen              |
|       A platformon fut       |        –         |         Windows, Linux, Docker         |           –            |              –              |
| Programozható <sup>1</sup> |   T-SQL, .NET, R   |         T-SQL, .NET, R, Python         |  T-SQL, .NET, R, Python  |              SQL              |

[1] nem beleértve az ügyfél illesztőprogram használatát, amely lehetővé teszi számos programozási nyelvek csatlakozhat, és az OLTP adattár használata.

### <a name="scalability-capabilities"></a>Méretezhetőség képességek

| | Azure SQL Database | SQL Server egy Azure virtuális gépen| Azure Database for MySQL | Azure Database for PostgreSQL|
| --- | --- | --- | --- | --- | --- |
| Maximális adatbázis példány mérete | [4 TB](/azure/sql-database/sql-database-resource-limits) | 256 TB | [1 TB](/azure/mysql/concepts-limits) | [1 TB](/azure/postgresql/concepts-limits) |
| Támogatja a kapacitás készletek  | Igen | Igen | Nem | Nem |
| Támogatja a fürtök kibővítési  | Nem | Igen | Nem | Nem |
| Dinamikus méretezhetőség (felskálázott)  | Igen | Nem | Igen | Igen |

### <a name="analytic-workload-capabilities"></a>Elemzési munkaterhelés képességek

| | Azure SQL Database | SQL Server egy Azure virtuális gépen| Azure Database for MySQL | Azure Database for PostgreSQL|
| --- | --- | --- | --- | --- | --- | 
| Historikus táblák | Igen | Igen | Nem | Nem |
| A memóriában (memóriaoptimalizált) táblák | Igen | Igen | Nem | Nem |
| Oszlopcentrikus támogatása | Igen | Igen | Nem | Nem |
| Adaptív lekérdezés-feldolgozás | Igen | Igen | Nem | Nem |

### <a name="availability-capabilities"></a>Rendelkezésre állás

| | Azure SQL Database | SQL Server egy Azure virtuális gépen| Azure Database for MySQL | Azure Database for PostgreSQL|
| --- | --- | --- | --- | --- | --- | 
| Olvasható másodlagos adatbázis | Igen | Igen | Nem | Nem | 
| Földrajzi replikáció | Igen | Igen | Nem | Nem | 
| Másodlagos automatikus feladatátvétel | Igen | Nem | Nem | Nem|
| Adott időpontnak megfelelő helyreállítás | Igen | Igen | Igen | Igen |

### <a name="security-capabilities"></a>Biztonsági képességei

|                                                                                                             | Azure SQL Database | SQL Server egy Azure virtuális gépen | Azure Database for MySQL | Azure Database for PostgreSQL |
|-------------------------------------------------------------------------------------------------------------|--------------------|----------------------------------------|--------------------------|-------------------------------|
|                                             Sorszintű biztonság                                              |        Igen         |                  Igen                   |           Igen            |              Igen              |
|                                                Adatmaszkolás                                                 |        Igen         |                  Igen                   |            Nem            |              Nem               |
|                                         Transzparens adattitkosítás                                         |        Igen         |                  Igen                   |           Igen            |              Igen              |
|                                  Hozzáférés korlátozása IP-címek                                   |        Igen         |                  Igen                   |           Igen            |              Igen              |
|                                  Hozzáférés korlátozása, hogy csak a virtuális hálózat hozzáférés engedélyezése                                  |        Igen         |                  Igen                   |            Nem            |              Nem               |
|                                    Hitelesítés Azure Active Directory-fiókkal                                    |        Igen         |                  Igen                   |            Nem            |              Nem               |
|                                       Active Directory-alapú hitelesítés                                       |         Nem         |                  Igen                   |            Nem            |              Nem               |
|                                         Multi-Factor Authentication                                         |        Igen         |                  Igen                   |            Nem            |              Nem               |
| Támogatja a [mindig titkosítja.](/sql/relational-databases/security/encryption/always-encrypted-database-engine) |        Igen         |                  Igen                   |           Igen            |              Nem               |
|                                                 Magánhálózati IP                                                  |         Nem         |                  Igen                   |           Igen            |              Nem               |

