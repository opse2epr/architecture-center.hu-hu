---
title: Helyreállítás a helyszínről az Azure-ba
titleSuffix: Azure Resiliency Technical Guidance
description: Ismertetése, és az Azure-bA helyszíni infrastruktúra helyreállítási rendszerek kialakítása.
author: adamglick
ms.date: 08/18/2016
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-design-principles
ms.custom: seojan19, resiliency
ms.openlocfilehash: 768e53e1024533b384c610378385c96d88d8571f
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54487372"
---
[!INCLUDE [header](../_includes/header.md)]

# <a name="azure-resiliency-technical-guidance-recovery-from-on-premises-to-azure"></a>Technikai útmutató az Azure rugalmassága: Helyreállítás a helyszínről az Azure-ba

Azure-szolgáltatások engedélyezése az Azure-bA egy helyszíni adatközpont kiterjesztését magas rendelkezésre állás és vészhelyreállítás helyreállítási célból átfogó készletét biztosítja:

- **Hálózatkezelés**: A virtuális magánhálózati, biztonsággal kiterjesztheti a helyszíni hálózatát a felhőbe.
- **COMPUTE**: Helyszíni Hyper-V használó ügyfelek is "átemelése" meglévő virtuális gépek (VM) az Azure-bA.
- **Tárolási**: A StorSimple az Azure Storage-terjeszti ki a fájlrendszer. Az Azure Backup szolgáltatás biztonsági mentési fájl-és SQL Database-adatbázisok az Azure Storage biztosít.
- **Adatbázis-replikáció**: Az SQL Server 2014 (vagy újabb) rendelkezésre állási csoportok a helyszíni adatok magas rendelkezésre állás és vészhelyreállítás helyreállítási valósíthat meg.

## <a name="networking"></a>Hálózatkezelés

Azure Virtual Network segítségével egy logikailag elkülönített terület létrehozása az Azure és annak biztonságos csatlakoztatása a helyszíni adatközponthoz vagy egyetlen ügyfélgéphez IPsec-kapcsolat használatával. A Virtual Network révén, kihasználhatja az igény szerint skálázható infrastruktúra az Azure-ban művelet során gondoskodik az adatokhoz és alkalmazásokhoz a helyszínen, beleértve a Windows Server, Nagyszámítógépek és UNIX rendszerekre való kapcsolódás. Lásd: [az Azure hálózati dokumentációja](/azure/virtual-network/virtual-networks-overview/) további információt.

## <a name="compute"></a>Számítás

Ha a helyszíni Hyper-V használata esetén meg is "átemelése" meglévő virtuális gépek Azure-ba, és formázza a Windows Server 2012 (vagy újabb rendszerű), a virtuális gép módosítása nélkül, vagy virtuális gép konvertálása szolgáltatók. További információkért lásd: [lemezek és virtuális merevlemezek, az Azure-beli virtuális gépek](/azure/virtual-machines/virtual-machines-linux-about-disks-vhds/?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).

## <a name="azure-site-recovery"></a>Azure Site Recovery

Ha azt szeretné, vész-helyreállítási szolgáltatás (DRaaS), az Azure biztosít [Azure Site Recovery](https://azure.microsoft.com/services/site-recovery/). Az Azure Site Recovery VMware-, Hyper-V és fizikai kiszolgálók teljes körű védelmet nyújt. Az Azure Site Recovery használhatja egy másik helyszíni kiszolgálón vagy az Azure helyreállítási helyként. További információ az Azure Site Recovery: a [Azure Site Recovery dokumentációja](https://azure.microsoft.com/documentation/services/site-recovery/).

## <a name="storage"></a>Tárhely

Többféle módon is az Azure-t a helykiszolgáló biztonsági mentése a helyszíni adatok számára.

### <a name="storsimple"></a>StorSimple

A StorSimple biztonságos és átlátható módon integrálja a helyszíni alkalmazások számára a felhőalapú tárolást. Emellett egy egyetlen berendezés, amely nagy teljesítményű rétegzett helyi és felhőalapú tárolási, élő archiválás, felhőbeli adatok védelmére, és vész-helyreállítási biztosít. További információkért lásd: a [StorSimple termékoldalán](https://azure.microsoft.com/services/storsimple/).

### <a name="azure-backup"></a>Azure Backup

Az Azure Backup lehetővé teszi, hogy a felhőbeli biztonsági mentést a jól ismert biztonsági mentési eszközök a Windows Server 2012 (vagy újabb), Windows Server 2012 Essentials (vagy újabb) és a System Center 2012 Data Protection Manager (vagy újabb). Ezek az eszközök egy munkafolyamatot, amely független a biztonsági mentések tárolási helyét a helyi lemezre vagy az Azure Storage-e biztonsági másolatokat kezelő adja meg. Adatok biztonsági mentése a felhőbe, után engedéllyel rendelkező felhasználók egyszerűen helyreállíthatják az biztonsági mentések minden olyan kiszolgálóra.

A növekményes biztonsági másolatok fájlok csak módosítások átkerülnek a felhőben. Ez segít hatékony tárhelyet használja, sávszélességet takaríthat és időponthoz helyreállítani az adatokat több verzióját. Választhatja azt is, használja az egyéb hasznos segédanyaghoz, például az adatmegőrzési házirendek, az adattömörítés és adatátvitel szabályozása. Az Azure-t a biztonsági mentési helyre az előnnyel jár nyilvánvaló, hogy a biztonsági mentések egyben a "külső". Ezzel elkerülhető a felesleges követelmények biztosításához és a helyszíni biztonsági mentési adathordozó védelmére.

További információkért lásd: [Mi az Azure Backup?](/azure/backup/backup-introduction-to-azure-backup/) és [Azure Backup konfigurálása a DPM-adatok](https://technet.microsoft.com/library/jj728752.aspx).

## <a name="database"></a>Adatbázis

Rendelkezik egy vész-helyreállítási megoldást hibrid-környezetben az SQL Server-adatbázisok AlwaysOn rendelkezésre állási csoportok, az adatbázis-tükrözés, a naplóküldés és a biztonsági mentés használatával, és állítsa vissza az Azure Blob storage. Ezek a megoldások használják az Azure Virtual machines szolgáltatásban futó SQL Server.

AlwaysOn rendelkezésre állási csoportok is használható, ahol adatbázis-replikák létezik-e a helyszíni környezetben hibrid-IT és a felhőben. Ez az alábbi ábrán látható.

![Az SQL Server AlwaysOn rendelkezésre állási csoportok a hibrid felhőalapú architektúra](./images/technical-guidance-recovery-on-premises-azure/SQL_Server_Disaster_Recovery-3.png)

Az adatbázis-tükrözés is átnyúlhatnak a helyszíni kiszolgálók és a egy ügyféltanúsítvány-alapú telepítés a felhőszolgáltatás. A következő ábra szemlélteti a fogalom.

![Az SQL Server adatbázis-tükrözés a hibrid felhőalapú architektúra](./images/technical-guidance-recovery-on-premises-azure/SQL_Server_Disaster_Recovery-4.png)

Naplóküldés szinkronizálni egy helyszíni adatbázishoz az SQL Server-adatbázis egy Azure virtuális gépen használható.

![A hibrid felhőalapú architektúra SQL Server-naplóküldés](./images/technical-guidance-recovery-on-premises-azure/SQL_Server_Disaster_Recovery-5.png)

Végül készíthető egy helyi adatbázis közvetlenül az Azure Blob storage-bA.

![SQL Server biztonsági mentése az Azure Blob storage-ban egy hibrid felhőalapú architektúra](./images/technical-guidance-recovery-on-premises-azure/SQL_Server_Disaster_Recovery-6.png)

További információkért lásd: [magas rendelkezésre állás és vészhelyreállítás helyreállítási SQL Serverhez készült Azure-beli virtuális gépeken](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-high-availability-dr/) és [biztonsági mentése és visszaállítása az SQL Server Azure-beli virtuális gépeken](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-backup-recovery/).

## <a name="checklists-for-on-premises-recovery-in-microsoft-azure"></a>A Microsoft Azure-ban a helyszíni helyreállítás Ellenőrzőlisták

<!-- markdownlint-disable MD024 -->

### <a name="networking"></a>Hálózatkezelés

1. Tekintse át a jelen dokumentum a hálózatkezelés című szakaszában.
2. Virtuális hálózat használatával a helyszíni biztonságosan csatlakozhat a felhőben.

### <a name="compute"></a>Számítás

1. Tekintse át a számítási szakasz ebben a dokumentumban.
2. Virtuális gépek áthelyezése a Hyper-V és az Azure között.

### <a name="storage"></a>Tárhely

1. Tekintse át a Storage szakasz ebben a dokumentumban.
2. A felhőalapú tárolással a StorSimple-szolgáltatások előnyeit.
3. Az Azure Backup szolgáltatás segítségével.

### <a name="database"></a>Adatbázis

1. Tekintse át a jelen dokumentum adatbázis szakaszát.
2. Fontolja meg az SQL Server Azure virtuális gépeken a biztonsági másolattal.
3. AlwaysOn rendelkezésre állási csoportok beállításához.
4. Konfigurálja az ügyféltanúsítvány-alapú adatbázis-tükrözés.
5. Használja a naplóküldésben.
6. Készítsen biztonsági másolatot a helyszíni adatbázisok az Azure Blob storage.

<!-- markdownlint-enable MD024 -->
