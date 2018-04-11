---
title: Adatsérülés vagy véletlen törlés helyreállítása
description: 'A következő cikket: az adatok sérülésének adatok vagy a véletlen adattörlés és rugalmas, magas rendelkezésre állású, hiba hibatűrő alkalmazásokhoz tervezéséhez, valamint vészhelyreállítás tervezése alapos ismerete'
author: MikeWasson
ms.date: 01/10/2018
ms.openlocfilehash: 76d2f996750d5a67b67bd5dc4977580f3b8abbc3
ms.sourcegitcommit: 3d6dba524cc7661740bdbaf43870de7728d60a01
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/11/2018
---
# <a name="recover-from-data-corruption-or-accidental-deletion"></a>Adatsérülés vagy véletlen törlés helyreállítása 

Ha az adatok beolvasása sérült, vagy véletlenül törölt egy robusztus üzletmenet folytonosságát biztosító terve részeként nehézségekkel egy tervet. A következő helyreállítási információt után adatok sérült, vagy véletlenül törölt alkalmazáshibák vagy a kezelő hibája miatt.

## <a name="virtual-machines"></a>Virtuális gépek

Az alkalmazáshibák vagy véletlen törlés elleni védelem Azure virtuális gépek (VM), használjon [Azure biztonsági mentés](/azure/backup/). Azure biztonsági mentés lehetővé teszi a több virtuális gép lemezre összhangban legyenek biztonsági mentések létrehozását. Emellett a mentési tárolóhoz az biztosítása érdekében származó régió régiók replikálható.

## <a name="storage"></a>Tárolás

Az Azure Storage automatikus replikák keresztül adatrugalmasság biztosít. Azonban ez nem akadályozza alkalmazáskód vagy hibás adatokból felhasználók véletlenül vagy szándékosan. Speciális módszerek, például másolja az adatokat a másodlagos tárolóját az audit napló állásuk alkalmazás- vagy hiba adatok hűség karbantartása szükséges. 

- **Blokkblobokat**. Hozzon létre minden egyes blokkblob pont időponthoz kötött pillanatképet. További információkért lásd: [egy pillanatképet készíteni egy Blob](/rest/api/storageservices/creating-a-snapshot-of-a-blob). Minden egyes pillanatkép van csak szó, a szükséges a különbségek belül a blob tárolására, mert az utolsó pillanatkép-állapot tárolásához. A pillanatképek függnek a létezik-e az eredeti blob, alapulnak, ezért egy másik blob vagy akár egy másik tárfiókhoz a másolási műveletek használata javasolt. Ez biztosítja, hogy a biztonsági mentési adatok megfelelően védett véletlen törlése ellen. Használhat [AzCopy](/azure/storage/common/storage-use-azcopy) vagy [Azure PowerShell](/azure/storage/common/storage-powershell-guide-full) a BLOB másolása másik tárolási fiókot.

- **Fájlok**. Használjon [megosztása pillanatképeket (előzetes verzió)](/azure/storage/files/storage-how-to-use-files-snapshots), vagy másolja a fájlokat egy másik tárfiókhoz AzCopy vagy a PowerShell segítségével.

- **Táblák**. AzCopy segítségével a tábla adatainak exportálása egy másik régióban másik tárolási fiókot.

## <a name="database"></a>Adatbázis

### <a name="azure-sql-database"></a>Azure SQL Database 

SQL-adatbázis automatikusan elvégzi az adatbázis teljes biztonsági mentés hetente kombinációja, a különbözeti adatbázis óránkénti, és a tranzakció jelentkezhetnek biztonsági mentések minden öt - tíz perc az üzleti adatvesztés elleni védelméhez. Használja a időpontban visszaállítást a restore egy adatbázis egy korábbi időpontra. További információkért lásd:

- [Automatikus adatbázis biztonsági mentését használó Azure SQL-adatbázis helyreállítása](/azure/sql-database/sql-database-recovery-using-backups)

- [Az Azure SQL Database üzletmenet áttekintése](/azure/sql-database/sql-database-business-continuity)

### <a name="sql-server-on-vms"></a>SQL Server virtuális gépeken

Virtuális gépeken futó SQL Server, a rendszer két lehetőség közül választhat: hagyományos biztonsági mentések és a naplóküldésben. Hagyományos biztonsági mentések lehetővé teszi adott visszaállítása időpontra, de a helyreállítási folyamat lassú. Hagyományos biztonsági másolatok visszaállítása szükséges kezdődő, és egy kezdeti teljes biztonsági mentést, és minden biztonsági másolatának után, majd alkalmazza. A második lehetőség a Naplók biztonsági másolatainak helyreállítását (például úgy, hogy két órával) késleltetése munkamenet naplóküldő konfigurálása. Ez lehetővé teszi egy ablakot, ahol az elsődleges végrehajtott hibák elhárítása.

### <a name="azure-cosmos-db"></a>Azure Cosmos DB

Azure Cosmos-adatbázis rendszeres időközönként automatikusan biztonsági mentések vesz igénybe. Biztonsági mentések külön-külön tárolják egy másik tárhelyre, és azokat a biztonsági mentések globálisan replikálva vannak a rugalmasságot regionális vészhelyzetek ellen. Ha véletlenül törli az adatbázis vagy a gyűjteményt, a támogatási jegy fájlt, vagy visszaállíthatja az adatokat a legutóbbi automatikus biztonsági mentés az Azure támogatási hívás. További információkért lásd: [automatikus online biztonsági mentés és helyreállítás Azure Cosmos DB](/azure/cosmos-db/online-backup-and-restore).

### <a name="azure-database-for-mysql-azure-database-for-postresql"></a>MySQL, Azure-adatbázis PostreSQL az Azure-adatbázis

Ha Azure-adatbázis MySQL vagy az Azure-adatbázishoz a PostreSQL, az adatbázis-szolgáltatás automatikusan révén a szolgáltatás biztonsági másolatot, ötpercenként. Az automatikus biztonsági mentési szolgáltatás használatával állítsa vissza a kiszolgáló és az adatbázisok be egy új kiszolgálót egy korábbi-időpontban. További információkért lásd:

- [Készítsen biztonsági másolatot, és egy kiszolgálóhoz az Azure-adatbázis visszaállítása a MySQL az Azure portál használatával](/azure/mysql/howto-restore-server-portal)

- [Biztonsági mentése és visszaállítása egy kiszolgálóhoz az Azure-adatbázis az Azure portál használatával PostgreSQL](/azure/postgresql/howto-restore-server-portal)

