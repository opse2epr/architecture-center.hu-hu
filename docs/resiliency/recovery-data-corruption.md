---
title: Helyreállítás adatsérülés vagy véletlen törlés
description: A helyreállítás adatsérülés adatok vagy véletlen törlés és a tartalék rugalmas, magas rendelkezésre állású, hibatűrő alkalmazások tervezése, valamint a vészhelyreállítási adatbázisból ismertető cikk
author: MikeWasson
ms.date: 11/11/2018
ms.openlocfilehash: 1f3dd448ac6172727481c437fb8a113f25d83464
ms.sourcegitcommit: dbbf914757b03cdee7a274204f9579fa63d7eed2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/02/2018
ms.locfileid: "50916269"
---
# <a name="recover-from-data-corruption-or-accidental-deletion"></a>Helyreállítás adatsérülés vagy véletlen törlés 

Ha az adatok beolvasása sérült vagy véletlenül törölt egy robusztus üzletmenet folytonosságát biztosító terve részeként tapasztalja csomagot. Az alábbiakban látható helyreállítási információk után adatok sérült vagy véletlenül törölt alkalmazáshibák vagy operátor hiba miatt.

## <a name="virtual-machines"></a>Virtuális gépek

Azure Virtual Machines (VM) az alkalmazáshibák vagy véletlen törlés elleni védelme, használja a [Azure Backup](/azure/backup/). Az Azure Backup lehetővé teszi több virtuális gép lemezeinek konzisztensek legyenek biztonsági mentések létrehozását. Emellett a Backup-tárolóban adja meg a helyreállítási régióban elmaradásából-régiók közötti replikálható.

## <a name="storage"></a>Storage

Az Azure Storage biztosítja az adatok rugalmasságát biztosítja az automatikus replikációval. Azonban ez nem akadályozza alkalmazáskód vagy a felhasználók hibás adatokból, hogy véletlenül vagy kártételi. Fejlett technikák, például az adatok másolásának auditálási naplóba kerülnek a másodlagos tárolóhelyre karbantartása adathűséget alkalmazás vagy felhasználó hiba esetén van szükség. 

- **Blokkblobok**. Minden egyes blokkblob időponthoz pillanatkép létrehozása. További információkért lásd: [létrehozása egy pillanatképet egy Blobról](/rest/api/storageservices/creating-a-snapshot-of-a-blob). Minden pillanatkép csak díjkötelesek a különbségek a blobon belüli tárolására, mivel az utolsó pillanatkép állapota szükséges tárhelyet. A pillanatképek függenek létezik-e az eredeti blob azok alapulnak, így célszerű a másolási műveletek egy másik blob vagy akár egy másik tárfiókba. Ez biztosítja, hogy a biztonsági mentési adatokat megfelelően védett véletlen törlése ellen. Használhat [AzCopy](/azure/storage/common/storage-use-azcopy) vagy [Azure PowerShell-lel](/azure/storage/common/storage-powershell-guide-full) a blobok másolása egy másik tárfiókba.

- **Fájlok**. Használjon [megosztási pillanatképek](/azure/storage/files/storage-snapshots-files), vagy az AzCopy és a PowerShell használatával a fájlok másolása egy másik tárfiókba.

- **Táblák**. Az AzCopy segítségével egy másik régióban egy másik tárfiókba történő táblák adatainak exportálása.

## <a name="database"></a>Adatbázis

### <a name="azure-sql-database"></a>Azure SQL Database 

Az SQL Database automatikusan végrehajtja az adatbázis teljes biztonsági mentését hetente, különbözeti adatbázis biztonsági mentését óránként, és a tranzakciós jelentkezzen biztonsági mentések minden öt - tíz percet üzleti adatai védelméről adatvesztéssel szemben. Időponthoz visszaállítás, visszaállítás adatbázist használnak egy korábbi időpontra. További információkért lásd:

- [Automatikus biztonsági adatbázismentés használatával Azure SQL-adatbázis helyreállítása](/azure/sql-database/sql-database-recovery-using-backups)

- [Az Azure SQL Database üzletmenet-folytonossági funkcióinak áttekintése](/azure/sql-database/sql-database-business-continuity)

### <a name="sql-server-on-vms"></a>Az SQL Server virtuális gépeken

A virtuális gépeken futó SQL Server, két lehetőség van: a hagyományos biztonsági mentéseket és a naplóküldésben. A hagyományos biztonsági mentések lehetővé teszi, hogy egy adott időpontra való visszaállítása, de a helyreállítási folyamat lassú. A hagyományos biztonsági másolatok visszaállítását végzni kezdve egy kezdeti biztonsági mentés, és ezután alkalmazása után, amely minden biztonsági szükséges. A második lehetőség egy munkamenet (például úgy, hogy két óra) naplók biztonsági másolatainak a visszaállítás késleltetése naplóküldő konfigurálásához. Ez lehetővé teszi a hibák az elsődleges végzett ablakot.

### <a name="azure-cosmos-db"></a>Azure Cosmos DB

Az Azure Cosmos DB automatikusan fogadja a biztonsági mentések rendszeres időközönként. Biztonsági másolatai külön egy másik tárolási szolgáltatás, és ezeket a biztonsági mentéseket globálisan replikálva vannak a regionális katasztrófa szembeni ellenálló-képesség. Ha véletlenül törli az adatbázis vagy -gyűjteményben, küldjön egy támogatási jegyet, vagy hívja a legutóbbi automatikus biztonsági mentés visszaállíthatja az adatokat az Azure ügyfélszolgálatától. További információkért lásd: [automatikus online biztonsági mentés és visszaállítás az Azure Cosmos DB](/azure/cosmos-db/online-backup-and-restore).

### <a name="azure-database-for-mysql-azure-database-for-postresql"></a>Azure Database for MySQL, Azure Database for Postgresql

Ha használja az Azure Database MySQL vagy az Azure Database for Postgresql, az adatbázis-szolgáltatás automatikusan lehetővé teszi a szolgáltatás biztonsági másolatának öt percenként. Az automatikus biztonsági mentési szolgáltatás használatával állítsa vissza a kiszolgáló és az összes hozzá tartozó adatbázisok be egy új kiszolgálót egy korábbi-időponthoz. További információkért lásd:

- [Hogyan biztonsági mentése és visszaállítása egy kiszolgálót az Azure Database for MySQL-hez az Azure portal használatával](/azure/mysql/howto-restore-server-portal)

- [Egy Azure Database for PostgreSQL-kiszolgáló biztonsági mentése és visszaállítása az Azure Portal használatával](/azure/postgresql/howto-restore-server-portal)

