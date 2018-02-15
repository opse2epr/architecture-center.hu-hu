---
title: "Az OLTP adattár kiválasztása"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 1c27d7d5f3b78f40822de6b77664dbf49b1367f6
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-an-oltp-data-store-in-azure"></a>Az Azure-ban egy OLTP adattároló kiválasztása

Online tranzakció-feldolgozási (OLTP) a tranzakciós adatok és a tranzakció-feldolgozást kezelése. Ez a témakör az Azure-ban OLTP megoldásokra lehetőségeket hasonlítja össze.

> [!NOTE]
> Az OLTP adattár használatával kapcsolatos további információkért lásd: [Online tranzakció-feldolgozási](../scenarios/online-analytical-processing.md).

## <a name="what-are-your-options-when-choosing-an-oltp-data-store"></a>Az OLTP adatok kiválasztásakor Mik a beállítások tárolásához?

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
| | Azure SQL Database | SQL Server egy Azure virtuális gépen | Azure Database for MySQL | Azure Database for PostgreSQL |
| --- | --- | --- | --- | --- | --- |
| Van a felügyelt | Igen | Nem | Igen | Igen |
| A platformon fut | – | Windows, Linux, Docker | – | – |
| Programozható <sup>1</sup> | T-SQL, .NET, R | T-SQL, .NET, R, Python | T-SQL, .NET, R, Python | SQL | SQL |

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
| | Azure SQL Database | SQL Server egy Azure virtuális gépen| Azure Database for MySQL | Azure Database for PostgreSQL|
| --- | --- | --- | --- | --- | --- | 
| Sorszintű biztonság | Igen | Igen | Igen | Igen |
| Adatmaszkolás | Igen | Igen | Nem | Nem |
| Transzparens adattitkosítás | Igen | Igen | Igen | Igen |
| Hozzáférés korlátozása IP-címek | Igen | Igen | Igen | Igen |
| Hozzáférés korlátozása, hogy csak a virtuális hálózat hozzáférés engedélyezése | Igen | Igen | Nem | Nem |
| Hitelesítés Azure Active Directory-fiókkal | Igen | Igen | Nem | Nem |
| Active Directory-alapú hitelesítés | Nem | Igen | Nem | Nem |
| Multi-Factor Authentication | Igen | Igen | Nem | Nem |
| Támogatja a [mindig titkosítja.](/sql/relational-databases/security/encryption/always-encrypted-database-engine) | Igen | Igen | Igen | Nem | Nem |
| Magánhálózati IP | Nem | Igen | Igen | Nem | Nem |

