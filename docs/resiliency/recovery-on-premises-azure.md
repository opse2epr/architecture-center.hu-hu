---
title: 'Műszaki útmutató: az Azure-bA helyszíni helyreállítási'
description: 'A következő cikket: ismertetése és helyreállítási designing Azure-bA helyszíni infrastruktúra rendszerek'
author: adamglick
ms.date: 08/18/2016
ms.openlocfilehash: 6992e27d148074b3d60c282318741f45974d1afd
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/06/2018
---
[!INCLUDE [header](../_includes/header.md)]

# <a name="azure-resiliency-technical-guidance-recovery-from-on-premises-to-azure"></a>Az Azure rugalmasságát műszaki útmutató: az Azure-bA helyszíni helyreállítási
Azure-szolgáltatását egy helyszíni adatközpontot az Azure-ba, a bővítmény engedélyezése a magas rendelkezésre állási és vészhelyreállítási helyreállítási célokra széles választékát nyújtja:

* **Hálózati**: A virtuális magánhálózat, biztonságos kiterjesztése a helyszíni hálózat a felhőbe.
* **Számítási**: ügyfelek helyszíni Hyper-V használatával "növekedési és az eltolás mértékét megadó" meglévő virtuális gépek (VM) az Azure-bA.
* **Tárolási**: StorSimple bővíti a fájlrendszer Azure Storage. Az Azure biztonsági mentési szolgáltatás biztonsági mentést biztosít a fájlokhoz és SQL-adatbázisok Azure Storage.
* **Adatbázis-replikáció**: az SQL Server 2014 (vagy újabb) rendelkezésre állási csoportok, a helyszíni adatok magas rendelkezésre állás és katasztrófa-helyreállítás is létrehozható.

## <a name="networking"></a>Hálózat
Az Azure Virtual Network használatával logikailag elkülönített szakasz létrehozása az Azure-ban, és biztonságosan csatlakoztassa a helyszíni adatközpontját, illetve egyetlen ügyfélszámítógép egy IPsec-kapcsolat használatával. A virtuális hálózattal kihasználhatja a méretezhető, igény szerinti infrastruktúra az Azure-ban ugyanakkor biztosítható a kapcsolat adatokhoz és alkalmazásokhoz a helyszínen, beleértve a Windows Server, Nagyszámítógépek és UNIX rendszerekre. Lásd: [Azure hálózati dokumentációja](/azure/virtual-network/virtual-networks-overview/) további információt.

## <a name="compute"></a>Számítás
Ha helyszíni Hyper-V használ, akkor "növekedési és az eltolás mértékét megadó" meglévő virtuális gépek Azure-ba, és formázza a Windows Server 2012-es (vagy újabb rendszerű), a virtuális gép módosítása nélkül, vagy a virtuális gépeket védetté alakító szolgáltatók. További információkért lásd: [lemezek és az Azure virtuális gépek virtuális merevlemezek](/azure/virtual-machines/virtual-machines-linux-about-disks-vhds/?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).

## <a name="azure-site-recovery"></a>Azure Site Recovery
Ha katasztrófa utáni helyreállítás szolgáltatásként (DRaaS), az Azure biztosít [Azure Site Recovery](https://azure.microsoft.com/services/site-recovery/). Az Azure Site Recovery VMware, Hyper-V és fizikai kiszolgálók átfogó védelmet nyújt. Az Azure Site Recovery használhatja más helyszíni kiszolgálón vagy az Azure helyreállítási helyként. Az Azure Site Recovery további információkért lásd: a [Azure Site Recovery dokumentáció](https://azure.microsoft.com/documentation/services/site-recovery/).

## <a name="storage"></a>Tárolás
Többféle módon is a helyszíni adatok az Azure biztonsági mentési helyeként.

### <a name="storsimple"></a>StorSimple
StorSimple biztonságosan és transzparens módon integrálható a helyszíni alkalmazások felhőtárhelyére. Egy egyetlen készülék letölti a nagy teljesítményű rétegzett helyi és felhőalapú tárolási, élő archiválás, felhőalapú adatvédelem és vész-helyreállítási is biztosít. További információkért lásd: a [StorSimple, termékoldala](https://azure.microsoft.com/services/storsimple/).

### <a name="azure-backup"></a>Azure Backup
Azure biztonsági mentés lehetővé teszi, hogy a felhőbeli biztonsági másolatok a jól ismert biztonsági mentési eszközök a Windows Server 2012 (vagy újabb), Windows Server 2012 Essentials (vagy újabb), és a System Center 2012 Data Protection Manager (vagy újabb). Ezek az eszközök munkafolyamat biztonságimásolat-felügyeleti, amely független a biztonsági mentések tárolási helyét a helyi lemezen vagy az Azure Storage adja meg. Adatok biztonsági mentése a felhőbe, után jogosult felhasználók egyszerűen helyre lehet állítani kiszolgálói biztonsági másolatok.

A növekményes biztonsági mentések csak a fájlokon végrehajtott módosítások átkerülnek a felhő. Ez segít, hatékonyan tárhelyet használja, csökkenteni a sávszélesség-felhasználást vagy időpontban helyreállítási adatok több verziójának támogatása. Választhatja azt is, szolgáltatások, például az adatmegőrzési házirendek az adattömörítés vagy adatátvitel szabályozása. Azure használja a biztonsági mentési helyre, azt a nyilvánvaló előnyt, győződjön meg arról, hogy a biztonsági mentések automatikusan "külső". Ezzel a megoldással a további követelmények biztosításához és a helyszíni biztonsági mentési adathordozó védelmére.

További információkért lásd: [Mi az Azure Backup?](/azure/backup/backup-introduction-to-azure-backup/) és [Azure Backup konfigurálása a DPM-adatok](https://technet.microsoft.com/library/jj728752.aspx).

## <a name="database"></a>Adatbázis
Vész-helyreállítási megoldást rendelkezik az SQL Server-adatbázisok hibrid-környezetben az AlwaysOn rendelkezésre állási csoportok, az adatbázis-tükrözés, a naplóküldési és a biztonsági másolat használatával, és állítsa vissza az Azure Blob storage szolgáltatással. Ezek a megoldások használó Azure virtuális gépeken futó SQL Server.

AlwaysOn rendelkezésre állási csoportok használható egy hibrid-informatikai környezetben, ahol adatbázis-replikák létezik-e a helyszínen és a felhőben. Ezt az alábbi ábra mutatja be.

![SQL Server AlwaysOn rendelkezésre állási csoportok hibrid felhő architektúra](./images/technical-guidance-recovery-on-premises-azure/SQL_Server_Disaster_Recovery-3.png)

Az adatbázis-tükrözés is átnyúlhatnak a helyszíni kiszolgálók és a felhőt, egy tanúsítvány alapú telepítő. A következő ábra szemlélteti a fogalom.

![SQL Server adatbázis-tükrözés hibrid felhő architektúra](./images/technical-guidance-recovery-on-premises-azure/SQL_Server_Disaster_Recovery-4.png)

Naplóküldésben szinkronizálását egy helyi adatbázist az SQL Server-adatbázis egy Azure virtuális gépen használható.

![SQL Server naplóküldési hibrid felhő architektúra](./images/technical-guidance-recovery-on-premises-azure/SQL_Server_Disaster_Recovery-5.png)

Végezetül is biztonsági másolatot készíteni egy helyi adatbázist közvetlenül az Azure Blob storage.

![SQL Server mentésére az Azure Blob storage hibrid felhő architektúra](./images/technical-guidance-recovery-on-premises-azure/SQL_Server_Disaster_Recovery-6.png)

További információkért lásd: [magas rendelkezésre állás és katasztrófa-helyreállítás az SQL Server Azure virtuális gépekben](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-high-availability-dr/) és [biztonsági mentése és visszaállítása az SQL Server Azure virtuális gépekben](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-backup-recovery/).

## <a name="checklists-for-on-premises-recovery-in-microsoft-azure"></a>A Microsoft Azure-ban a helyszíni helyreállítás Ellenőrzőlisták
### <a name="networking"></a>Hálózat
1. Tekintse át a hálózati szakasz ebben a dokumentumban.
2. Virtuális hálózati segítségével biztonságosan csatlakozzon a helyi a felhőbe.

### <a name="compute"></a>Számítás
1. Tekintse át a dokumentum a számítási szakaszát.
2. Helyezze át a virtuális gépek a Hyper-V és az Azure között.

### <a name="storage"></a>Tárolás
1. Tekintse át a dokumentum a tárolási szakaszát.
2. A felhőalapú tároló-StorSimple szolgáltatások előnyeit.
3. Az Azure Backup szolgáltatás segítségével.

### <a name="database"></a>Adatbázis
1. Tekintse át a dokumentum az adatbázis szakaszát.
2. Vegye fontolóra SQL Server Azure virtuális gépeken a biztonsági mentés használatát.
3. AlwaysOn rendelkezésre állási csoportok beállítása.
4. Konfigurálja a tanúsítvány alapú adatbázis-tükrözés.
5. Használja a naplóküldésben.
6. Adatbázisok biztonsági mentése a helyszíni Azure Blob Storage.


