---
title: "Adatsérülés vagy véletlen törlés helyreállítása"
description: "A következő cikket: az adatok sérülésének adatok vagy a véletlen adattörlés és rugalmas, magas rendelkezésre állású, hiba hibatűrő alkalmazásokhoz tervezéséhez, valamint vészhelyreállítás tervezése alapos ismerete"
author: adamglick
ms.date: 08/18/2016
ms.openlocfilehash: b75c774f85c42f64472167897f08a7302ab50a3f
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
[!INCLUDE [header](../_includes/header.md)]
# <a name="azure-resiliency-technical-guidance-recovery-from-data-corruption-or-accidental-deletion"></a>Az Azure rugalmasságát műszaki útmutatót: helyreállítási adatok sérülése vagy véletlen törlés
Ha az adatok beolvasása sérült, vagy véletlenül törölt egy robusztus üzletmenet folytonosságát biztosító terve részeként nehézségekkel egy tervet. A következő helyreállítási információt után adatok sérült, vagy véletlenül törölt alkalmazáshibák vagy a kezelő hibája miatt.

## <a name="virtual-machines"></a>Virtuális gépek
Az Azure virtuális gépek (más néven infrastruktúra,--szolgáltatás virtuális gépek) alkalmazáshibák vagy véletlen törlés elleni védelem, használjon [Azure biztonsági mentés](https://azure.microsoft.com/services/backup/). Azure biztonsági mentés lehetővé teszi a több virtuális gép lemezre összhangban legyenek biztonsági mentések létrehozását. Emellett a mentési tárolóhoz az biztosítása érdekében származó régió régiók replikálható.

## <a name="storage"></a>Storage
Figyelje meg, míg az Azure Storage automatikus replikák keresztül adatrugalmasság kínál, ez nem akadályozza a alkalmazáskód (vagy fejlesztők/felhasználók) származó adatok rongál keresztül nem szándékos vagy véletlen törlés, frissítés, és így tovább. Speciális módszerek, például másolja az adatokat a másodlagos tárolóját az audit napló állásuk alkalmazás- vagy hiba adatok hűség karbantartása szükséges. A fejlesztők kihasználhatják a BLOB [pillanatkép-funkció](https://msdn.microsoft.com/library/azure/ee691971.aspx), amely a blob tartalmát csak olvasható időpontban pillanatképeket hozhat létre. Ez használható alapjául szolgáló adatok minőségű megoldást az Azure Storage blobs szolgáltatásban.

### <a name="blob-and-table-storage-backup"></a>A BLOB és Table Storage biztonsági mentése
Míg a blobok és táblák magas tartós, mindig képviselnek az adatok aktuális állapotát. A nem kívánt módosításának és törlésének adatok helyreállítási lehet szükség, adat-visszaállítást a korábbi állapotába. Ez a tárolására és időpontban másolatok Azure által biztosított képességek kihasználásával elérhető.

Az Azure BLOB, időpontban biztonsági mentések használatával végezheti el a [blob pillanatkép funkció](https://msdn.microsoft.com/library/ee691971.aspx). Minden egyes pillanatkép van csak szó, a szükséges a különbségek belül a blob tárolására, mert az utolsó pillanatkép-állapot tárolásához. A pillanatképek függnek a létezik-e az eredeti blob, alapulnak, ezért egy másik blob vagy akár egy másik tárfiókhoz a másolási műveletek használata javasolt. Ez biztosítja, hogy a biztonsági mentési adatok megfelelően védett véletlen törlése ellen. Az Azure táblázatban időpontban másolatok egy másik táblához vagy az Azure BLOB teheti meg. Részletes útmutatás és példák bemutatják táblák alkalmazásszintű biztonsági másolatait és blobok itt található:

* [A táblák alkalmazáshibák elleni védelme](https://blogs.msdn.microsoft.com/windowsazurestorage/2010/05/03/protecting-your-tables-against-application-errors/)
* [A Blobok alkalmazáshibák elleni védelme](https://blogs.msdn.microsoft.com/windowsazurestorage/2010/04/29/protecting-your-blobs-against-application-errors/)

## <a name="database"></a>Adatbázis
Több [az üzletmenet folytonossága](/azure/sql-database/sql-database-business-continuity/) (biztonsági másolat, visszaállítási) az Azure SQL Database elérhető beállítások. Adatbázisok segítségével másolhatók a [adatbázis-másolat](/azure/sql-database/sql-database-copy/) funkciót, vagy a [exportálása](/azure/sql-database/sql-database-export/) és [importálása](https://msdn.microsoft.com/library/hh710052.aspx) egy SQL Server bacpac-fájl. Adatbázis-másolat biztosítja a tranzakciós úton konzisztens eredmények (keresztül az import/export szolgáltatás) egy bacpac azonban nem. Mindkét lehetőség az adatközponton belül várólista alapú szolgáltatások futnak, és jelenleg nem biztosít egy idő létrehozása után SLA-t.

> [!NOTE]
> Az adatbázis másolása és importálási/exportálási beállítások terhelés jelentős mértékben a forrásadatbázis helyezze el. Indíthatnak Erőforrásverseny vagy a sávszélesség-szabályozás események.
> 
> 

### <a name="sql-database-backup"></a>SQL-adatbázis biztonsági mentése
Microsoft Azure SQL Database időpontban biztonsági mentés által érhetők el [az Azure SQL-adatbázis másolása](/azure/sql-database/sql-database-copy/). Ez a parancs segítségével adatbázis tranzakciós úton konzisztens másolat létrehozása ugyanazon a logikai adatbázis-kiszolgálón vagy egy másik kiszolgálóra. Mindkét esetben az adatbázis-példány teljesen működőképes és a source adatbázis teljesen független. Minden egyes példányra hoz létre egy időpontban helyreállítási lehetőség jelöli. A forrás-adatbázis nevét új adatbázis átnevezésével teljesen helyreállíthatja az adatbázis állapotát. Azt is megteheti helyreállíthatja az adatokat egy adott részének az új adatbázis Transact-SQL-lekérdezések használatával. SQL-adatbázis kapcsolatos további részletekért lásd: [az Azure SQL Database üzletmenet áttekintése](/azure/sql-database/sql-database-business-continuity/).

### <a name="sql-server-on-virtual-machines-backup"></a>A virtuális gépek biztonsági mentése SQL-kiszolgáló
Az SQL Server használja az Azure-infrastruktúra a szolgáltatási célú virtuális gépek (IaaS és infrastruktúra-szolgáltatási virtuális gépeken gyakran nevezik), két módon: hagyományos biztonsági mentések és a naplóküldésben. Hagyományos biztonsági mentések használata lehetővé teszi adott visszaállítása időpontra, de a helyreállítási folyamat lassú. Hagyományos biztonsági másolatok visszaállítása szükséges kezdődő, és egy kezdeti teljes biztonsági mentést, és minden biztonsági másolatának után, majd alkalmazza. A második lehetőség a Naplók biztonsági másolatainak helyreállítását (például úgy, hogy két órával) késleltetése munkamenet naplóküldő konfigurálása. Ez lehetővé teszi egy ablakot, ahol az elsődleges végrehajtott hibák elhárítása.

## <a name="other-azure-platform-services"></a>Egyéb Azure platform szolgáltatásaiból
Bizonyos Azure platform szolgáltatásaiból információk egy felhasználó által szabályozott tárfiókhoz vagy Azure SQL-adatbázis tárolja. Ha a fiók vagy a storage erőforrás nem található vagy sérült, ez a szolgáltatás a súlyos hibákat okozhat. Ebben az esetben fontos karbantartása biztonsági mentések, amely lehetővé tenné, hogy hozza létre ezeket az erőforrásokat, mintha törölt vagy sérült.

Azure-webhelyek és az Azure Mobile Services, a biztonsági mentéshez és a társított adatbázisokban karbantartása. Az Azure Media Services és a virtuális gépek kezelnie kell a társított Azure Storage-fiók és a fiókhoz tartozó összes erőforrást. Például a virtuális gépek kell biztonsági mentése és felügyelnie a virtuális gép lemezeivel az Azure blob Storage tárolóban.

## <a name="checklists-for-data-corruption-or-accidental-deletion"></a>A program sérült adatokat vagy véletlen törlés Ellenőrzőlisták
## <a name="virtual-machines-checklist"></a>Virtuális gépek ellenőrzőlista
1. Tekintse át a virtuális gépek szakasz ebben a dokumentumban.
2. Készítsen biztonsági másolatot, és karbantartása a virtuális gépek lemezei Azure biztonsági mentés (vagy a saját biztonsági mentési rendszer, az Azure blob storage és a VHD-pillanatképek használatával).

## <a name="storage-checklist"></a>Tárolási ellenőrzőlista
1. Tekintse át a dokumentum a tárolási szakaszát.
2. Rendszeresen biztonsági másolatot készíteni a kritikus tároló-erőforrások.
3. Vegye figyelembe, hogy a pillanatkép funkció használata a blobok.

## <a name="database-checklist"></a>Adatbázis ellenőrzőlista
1. Tekintse át a dokumentum az adatbázis szakaszát.
2. Hozzon létre időpontban biztonsági mentések az adatbázis-másolat parancs használatával.

## <a name="sql-server-on-virtual-machines-backup-checklist"></a>SQL Server, a virtuális gépek biztonsági mentése ellenőrzőlista
1. Tekintse át az SQL Server virtuális gépek biztonsági mentése szakasz ebben a dokumentumban.
2. Hagyományos biztonságimásolat-készítő és technikák visszaállítása.
3. A késleltetett naplószolgáltató munkamenet létrehozása.

## <a name="web-apps-checklist"></a>Web Apps ellenőrzőlista
1. Készítsen biztonsági másolatot, és a kapcsolódó adatbázis karbantartása, ha van ilyen.

## <a name="media-services-checklist"></a>A Media Services ellenőrzőlista
1. Készítsen biztonsági másolatot, és a társított tárolási erőforrások kezelése.

## <a name="more-information"></a>További információ
Az Azure biztonsági mentési és visszaállítási funkciókkal kapcsolatos további információkért lásd: [tárolási, a biztonsági mentési és helyreállítási forgatókönyvek](https://azure.microsoft.com/documentation/scenarios/storage-backup-recovery/).


