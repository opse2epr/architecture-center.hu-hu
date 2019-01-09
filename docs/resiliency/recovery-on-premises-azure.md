---
title: Helyreállítás a helyszínről az Azure-ba
titleSuffix: Azure Resiliency Technical Guidance
description: Ismertetése, és az Azure-bA helyszíni infrastruktúra helyreállítási rendszerek kialakítása.
author: adamglick
ms.date: 08/18/2016
ms.custom: seojan19
ms.openlocfilehash: 5e4c4ea4eede5f11e787b9957b8de47736645672
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/08/2019
ms.locfileid: "54111359"
---
[!INCLUDE [header](../_includes/header.md)]

# <a name="azure-resiliency-technical-guidance-recovery-from-on-premises-to-azure"></a><span data-ttu-id="0f96d-103">Technikai útmutató az Azure rugalmassága: Helyreállítás a helyszínről az Azure-ba</span><span class="sxs-lookup"><span data-stu-id="0f96d-103">Azure resiliency technical guidance: Recovery from on-premises to Azure</span></span>

<span data-ttu-id="0f96d-104">Azure-szolgáltatások engedélyezése az Azure-bA egy helyszíni adatközpont kiterjesztését magas rendelkezésre állás és vészhelyreállítás helyreállítási célból átfogó készletét biztosítja:</span><span class="sxs-lookup"><span data-stu-id="0f96d-104">Azure provides a comprehensive set of services for enabling the extension of an on-premises datacenter to Azure for high availability and disaster recovery purposes:</span></span>

- <span data-ttu-id="0f96d-105">**Hálózatkezelés**: A virtuális magánhálózati, biztonsággal kiterjesztheti a helyszíni hálózatát a felhőbe.</span><span class="sxs-lookup"><span data-stu-id="0f96d-105">**Networking**: With a virtual private network, you securely extend your on-premises network to the cloud.</span></span>
- <span data-ttu-id="0f96d-106">**COMPUTE**: Helyszíni Hyper-V használó ügyfelek is "átemelése" meglévő virtuális gépek (VM) az Azure-bA.</span><span class="sxs-lookup"><span data-stu-id="0f96d-106">**Compute**: Customers using Hyper-V on-premises can “lift and shift” existing virtual machines (VMs) to Azure.</span></span>
- <span data-ttu-id="0f96d-107">**Tárolási**: A StorSimple az Azure Storage-terjeszti ki a fájlrendszer.</span><span class="sxs-lookup"><span data-stu-id="0f96d-107">**Storage**: StorSimple extends your file system to Azure Storage.</span></span> <span data-ttu-id="0f96d-108">Az Azure Backup szolgáltatás biztonsági mentési fájl-és SQL Database-adatbázisok az Azure Storage biztosít.</span><span class="sxs-lookup"><span data-stu-id="0f96d-108">The Azure Backup service provides backup for files and SQL databases to Azure Storage.</span></span>
- <span data-ttu-id="0f96d-109">**Adatbázis-replikáció**: Az SQL Server 2014 (vagy újabb) rendelkezésre állási csoportok a helyszíni adatok magas rendelkezésre állás és vészhelyreállítás helyreállítási valósíthat meg.</span><span class="sxs-lookup"><span data-stu-id="0f96d-109">**Database replication**: With SQL Server 2014 (or later) Availability Groups, you can implement high availability and disaster recovery for your on-premises data.</span></span>

## <a name="networking"></a><span data-ttu-id="0f96d-110">Hálózat</span><span class="sxs-lookup"><span data-stu-id="0f96d-110">Networking</span></span>

<span data-ttu-id="0f96d-111">Azure Virtual Network segítségével egy logikailag elkülönített terület létrehozása az Azure és annak biztonságos csatlakoztatása a helyszíni adatközponthoz vagy egyetlen ügyfélgéphez IPsec-kapcsolat használatával.</span><span class="sxs-lookup"><span data-stu-id="0f96d-111">You can use Azure Virtual Network to create a logically isolated section in Azure and securely connect it to your on-premises datacenter or a single client machine by using an IPsec connection.</span></span> <span data-ttu-id="0f96d-112">A Virtual Network révén, kihasználhatja az igény szerint skálázható infrastruktúra az Azure-ban művelet során gondoskodik az adatokhoz és alkalmazásokhoz a helyszínen, beleértve a Windows Server, Nagyszámítógépek és UNIX rendszerekre való kapcsolódás.</span><span class="sxs-lookup"><span data-stu-id="0f96d-112">With Virtual Network, you can take advantage of the scalable, on-demand infrastructure in Azure while providing connectivity to data and applications on-premises, including systems running on Windows Server, mainframes, and UNIX.</span></span> <span data-ttu-id="0f96d-113">Lásd: [az Azure hálózati dokumentációja](/azure/virtual-network/virtual-networks-overview/) további információt.</span><span class="sxs-lookup"><span data-stu-id="0f96d-113">See [Azure networking documentation](/azure/virtual-network/virtual-networks-overview/) for more information.</span></span>

## <a name="compute"></a><span data-ttu-id="0f96d-114">Compute</span><span class="sxs-lookup"><span data-stu-id="0f96d-114">Compute</span></span>

<span data-ttu-id="0f96d-115">Ha a helyszíni Hyper-V használata esetén meg is "átemelése" meglévő virtuális gépek Azure-ba, és formázza a Windows Server 2012 (vagy újabb rendszerű), a virtuális gép módosítása nélkül, vagy virtuális gép konvertálása szolgáltatók.</span><span class="sxs-lookup"><span data-stu-id="0f96d-115">If you're using Hyper-V on-premises, you can “lift and shift” existing virtual machines to Azure and service providers running Windows Server 2012 (or later), without making changes to the VM or converting VM formats.</span></span> <span data-ttu-id="0f96d-116">További információkért lásd: [lemezek és virtuális merevlemezek, az Azure-beli virtuális gépek](/azure/virtual-machines/virtual-machines-linux-about-disks-vhds/?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).</span><span class="sxs-lookup"><span data-stu-id="0f96d-116">For more information, see [About disks and VHDs for Azure virtual machines](/azure/virtual-machines/virtual-machines-linux-about-disks-vhds/?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).</span></span>

## <a name="azure-site-recovery"></a><span data-ttu-id="0f96d-117">Azure Site Recovery</span><span class="sxs-lookup"><span data-stu-id="0f96d-117">Azure Site Recovery</span></span>

<span data-ttu-id="0f96d-118">Ha azt szeretné, vész-helyreállítási szolgáltatás (DRaaS), az Azure biztosít [Azure Site Recovery](https://azure.microsoft.com/services/site-recovery/).</span><span class="sxs-lookup"><span data-stu-id="0f96d-118">If you want disaster recovery as a service (DRaaS), Azure provides [Azure Site Recovery](https://azure.microsoft.com/services/site-recovery/).</span></span> <span data-ttu-id="0f96d-119">Az Azure Site Recovery VMware-, Hyper-V és fizikai kiszolgálók teljes körű védelmet nyújt.</span><span class="sxs-lookup"><span data-stu-id="0f96d-119">Azure Site Recovery offers comprehensive protection for VMware, Hyper-V, and physical servers.</span></span> <span data-ttu-id="0f96d-120">Az Azure Site Recovery használhatja egy másik helyszíni kiszolgálón vagy az Azure helyreállítási helyként.</span><span class="sxs-lookup"><span data-stu-id="0f96d-120">With Azure Site Recovery, you can use another on-premises server or Azure as your recovery site.</span></span> <span data-ttu-id="0f96d-121">További információ az Azure Site Recovery: a [Azure Site Recovery dokumentációja](https://azure.microsoft.com/documentation/services/site-recovery/).</span><span class="sxs-lookup"><span data-stu-id="0f96d-121">For more information on Azure Site Recovery, see the [Azure Site Recovery documentation](https://azure.microsoft.com/documentation/services/site-recovery/).</span></span>

## <a name="storage"></a><span data-ttu-id="0f96d-122">Storage</span><span class="sxs-lookup"><span data-stu-id="0f96d-122">Storage</span></span>

<span data-ttu-id="0f96d-123">Többféle módon is az Azure-t a helykiszolgáló biztonsági mentése a helyszíni adatok számára.</span><span class="sxs-lookup"><span data-stu-id="0f96d-123">There are several options for using Azure as a backup site for on-premises data.</span></span>

### <a name="storsimple"></a><span data-ttu-id="0f96d-124">StorSimple</span><span class="sxs-lookup"><span data-stu-id="0f96d-124">StorSimple</span></span>

<span data-ttu-id="0f96d-125">A StorSimple biztonságos és átlátható módon integrálja a helyszíni alkalmazások számára a felhőalapú tárolást.</span><span class="sxs-lookup"><span data-stu-id="0f96d-125">StorSimple securely and transparently integrates cloud storage for on-premises applications.</span></span> <span data-ttu-id="0f96d-126">Emellett egy egyetlen berendezés, amely nagy teljesítményű rétegzett helyi és felhőalapú tárolási, élő archiválás, felhőbeli adatok védelmére, és vész-helyreállítási biztosít.</span><span class="sxs-lookup"><span data-stu-id="0f96d-126">It also offers a single appliance that delivers high-performance tiered local and cloud storage, live archiving, cloud-based data protection, and disaster recovery.</span></span> <span data-ttu-id="0f96d-127">További információkért lásd: a [StorSimple termékoldalán](https://azure.microsoft.com/services/storsimple/).</span><span class="sxs-lookup"><span data-stu-id="0f96d-127">For more information, see the [StorSimple product page](https://azure.microsoft.com/services/storsimple/).</span></span>

### <a name="azure-backup"></a><span data-ttu-id="0f96d-128">Azure Backup</span><span class="sxs-lookup"><span data-stu-id="0f96d-128">Azure Backup</span></span>

<span data-ttu-id="0f96d-129">Az Azure Backup lehetővé teszi, hogy a felhőbeli biztonsági mentést a jól ismert biztonsági mentési eszközök a Windows Server 2012 (vagy újabb), Windows Server 2012 Essentials (vagy újabb) és a System Center 2012 Data Protection Manager (vagy újabb).</span><span class="sxs-lookup"><span data-stu-id="0f96d-129">Azure Backup enables cloud backups by using the familiar backup tools in Windows Server 2012 (or later), Windows Server 2012 Essentials (or later), and System Center 2012 Data Protection Manager (or later).</span></span> <span data-ttu-id="0f96d-130">Ezek az eszközök egy munkafolyamatot, amely független a biztonsági mentések tárolási helyét a helyi lemezre vagy az Azure Storage-e biztonsági másolatokat kezelő adja meg.</span><span class="sxs-lookup"><span data-stu-id="0f96d-130">These tools provide a workflow for backup management that is independent of the storage location of the backups, whether a local disk or Azure Storage.</span></span> <span data-ttu-id="0f96d-131">Adatok biztonsági mentése a felhőbe, után engedéllyel rendelkező felhasználók egyszerűen helyreállíthatják az biztonsági mentések minden olyan kiszolgálóra.</span><span class="sxs-lookup"><span data-stu-id="0f96d-131">After data is backed up to the cloud, authorized users can easily recover backups to any server.</span></span>

<span data-ttu-id="0f96d-132">A növekményes biztonsági másolatok fájlok csak módosítások átkerülnek a felhőben.</span><span class="sxs-lookup"><span data-stu-id="0f96d-132">With incremental backups, only changes to files are transferred to the cloud.</span></span> <span data-ttu-id="0f96d-133">Ez segít hatékony tárhelyet használja, sávszélességet takaríthat és időponthoz helyreállítani az adatokat több verzióját.</span><span class="sxs-lookup"><span data-stu-id="0f96d-133">This helps to efficiently use storage space, reduce bandwidth consumption, and support point-in-time recovery of multiple versions of the data.</span></span> <span data-ttu-id="0f96d-134">Választhatja azt is, használja az egyéb hasznos segédanyaghoz, például az adatmegőrzési házirendek, az adattömörítés és adatátvitel szabályozása.</span><span class="sxs-lookup"><span data-stu-id="0f96d-134">You can also choose to use additional features, such as data retention policies, data compression, and data transfer throttling.</span></span> <span data-ttu-id="0f96d-135">Az Azure-t a biztonsági mentési helyre az előnnyel jár nyilvánvaló, hogy a biztonsági mentések egyben a "külső".</span><span class="sxs-lookup"><span data-stu-id="0f96d-135">Using Azure as the backup location has the obvious advantage that the backups are automatically “offsite”.</span></span> <span data-ttu-id="0f96d-136">Ezzel elkerülhető a felesleges követelmények biztosításához és a helyszíni biztonsági mentési adathordozó védelmére.</span><span class="sxs-lookup"><span data-stu-id="0f96d-136">This eliminates the extra requirements to secure and protect on-site backup media.</span></span>

<span data-ttu-id="0f96d-137">További információkért lásd: [Mi az Azure Backup?](/azure/backup/backup-introduction-to-azure-backup/) és [Azure Backup konfigurálása a DPM-adatok](https://technet.microsoft.com/library/jj728752.aspx).</span><span class="sxs-lookup"><span data-stu-id="0f96d-137">For more information, see [What is Azure Backup?](/azure/backup/backup-introduction-to-azure-backup/) and [Configure Azure Backup for DPM data](https://technet.microsoft.com/library/jj728752.aspx).</span></span>

## <a name="database"></a><span data-ttu-id="0f96d-138">Adatbázis</span><span class="sxs-lookup"><span data-stu-id="0f96d-138">Database</span></span>

<span data-ttu-id="0f96d-139">Rendelkezik egy vész-helyreállítási megoldást hibrid-környezetben az SQL Server-adatbázisok AlwaysOn rendelkezésre állási csoportok, az adatbázis-tükrözés, a naplóküldés és a biztonsági mentés használatával, és állítsa vissza az Azure Blob storage.</span><span class="sxs-lookup"><span data-stu-id="0f96d-139">You can have a disaster recovery solution for your SQL Server databases in a hybrid-IT environment by using AlwaysOn Availability Groups, database mirroring, log shipping, and backup and restore with Azure Blob storage.</span></span> <span data-ttu-id="0f96d-140">Ezek a megoldások használják az Azure Virtual machines szolgáltatásban futó SQL Server.</span><span class="sxs-lookup"><span data-stu-id="0f96d-140">All of these solutions use SQL Server running on Azure Virtual Machines.</span></span>

<span data-ttu-id="0f96d-141">AlwaysOn rendelkezésre állási csoportok is használható, ahol adatbázis-replikák létezik-e a helyszíni környezetben hibrid-IT és a felhőben.</span><span class="sxs-lookup"><span data-stu-id="0f96d-141">AlwaysOn Availability Groups can be used in a hybrid-IT environment where database replicas exist both on-premises and in the cloud.</span></span> <span data-ttu-id="0f96d-142">Ez az alábbi ábrán látható.</span><span class="sxs-lookup"><span data-stu-id="0f96d-142">This is shown in the following diagram.</span></span>

![Az SQL Server AlwaysOn rendelkezésre állási csoportok a hibrid felhőalapú architektúra](./images/technical-guidance-recovery-on-premises-azure/SQL_Server_Disaster_Recovery-3.png)

<span data-ttu-id="0f96d-144">Az adatbázis-tükrözés is átnyúlhatnak a helyszíni kiszolgálók és a egy ügyféltanúsítvány-alapú telepítés a felhőszolgáltatás.</span><span class="sxs-lookup"><span data-stu-id="0f96d-144">Database mirroring can also span on-premises servers and the cloud in a certificate-based setup.</span></span> <span data-ttu-id="0f96d-145">A következő ábra szemlélteti a fogalom.</span><span class="sxs-lookup"><span data-stu-id="0f96d-145">The following diagram illustrates this concept.</span></span>

![Az SQL Server adatbázis-tükrözés a hibrid felhőalapú architektúra](./images/technical-guidance-recovery-on-premises-azure/SQL_Server_Disaster_Recovery-4.png)

<span data-ttu-id="0f96d-147">Naplóküldés szinkronizálni egy helyszíni adatbázishoz az SQL Server-adatbázis egy Azure virtuális gépen használható.</span><span class="sxs-lookup"><span data-stu-id="0f96d-147">Log shipping can be used to synchronize an on-premises database with a SQL Server database in an Azure virtual machine.</span></span>

![A hibrid felhőalapú architektúra SQL Server-naplóküldés](./images/technical-guidance-recovery-on-premises-azure/SQL_Server_Disaster_Recovery-5.png)

<span data-ttu-id="0f96d-149">Végül készíthető egy helyi adatbázis közvetlenül az Azure Blob storage-bA.</span><span class="sxs-lookup"><span data-stu-id="0f96d-149">Finally, you can back up an on-premises database directly to Azure Blob storage.</span></span>

![SQL Server biztonsági mentése az Azure Blob storage-ban egy hibrid felhőalapú architektúra](./images/technical-guidance-recovery-on-premises-azure/SQL_Server_Disaster_Recovery-6.png)

<span data-ttu-id="0f96d-151">További információkért lásd: [magas rendelkezésre állás és vészhelyreállítás helyreállítási SQL Serverhez készült Azure-beli virtuális gépeken](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-high-availability-dr/) és [biztonsági mentése és visszaállítása az SQL Server Azure-beli virtuális gépeken](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-backup-recovery/).</span><span class="sxs-lookup"><span data-stu-id="0f96d-151">For more information, see [High availability and disaster recovery for SQL Server in Azure virtual machines](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-high-availability-dr/) and [Backup and restore for SQL Server in Azure virtual machines](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-backup-recovery/).</span></span>

## <a name="checklists-for-on-premises-recovery-in-microsoft-azure"></a><span data-ttu-id="0f96d-152">A Microsoft Azure-ban a helyszíni helyreállítás Ellenőrzőlisták</span><span class="sxs-lookup"><span data-stu-id="0f96d-152">Checklists for on-premises recovery in Microsoft Azure</span></span>

<!-- markdownlint-disable MD024 -->

### <a name="networking"></a><span data-ttu-id="0f96d-153">Hálózat</span><span class="sxs-lookup"><span data-stu-id="0f96d-153">Networking</span></span>

1. <span data-ttu-id="0f96d-154">Tekintse át a jelen dokumentum a hálózatkezelés című szakaszában.</span><span class="sxs-lookup"><span data-stu-id="0f96d-154">Review the Networking section of this document.</span></span>
2. <span data-ttu-id="0f96d-155">Virtuális hálózat használatával a helyszíni biztonságosan csatlakozhat a felhőben.</span><span class="sxs-lookup"><span data-stu-id="0f96d-155">Use Virtual Network to securely connect on-premises to the cloud.</span></span>

### <a name="compute"></a><span data-ttu-id="0f96d-156">Compute</span><span class="sxs-lookup"><span data-stu-id="0f96d-156">Compute</span></span>

1. <span data-ttu-id="0f96d-157">Tekintse át a számítási szakasz ebben a dokumentumban.</span><span class="sxs-lookup"><span data-stu-id="0f96d-157">Review the Compute section of this document.</span></span>
2. <span data-ttu-id="0f96d-158">Virtuális gépek áthelyezése a Hyper-V és az Azure között.</span><span class="sxs-lookup"><span data-stu-id="0f96d-158">Relocate VMs between Hyper-V and Azure.</span></span>

### <a name="storage"></a><span data-ttu-id="0f96d-159">Storage</span><span class="sxs-lookup"><span data-stu-id="0f96d-159">Storage</span></span>

1. <span data-ttu-id="0f96d-160">Tekintse át a Storage szakasz ebben a dokumentumban.</span><span class="sxs-lookup"><span data-stu-id="0f96d-160">Review the Storage section of this document.</span></span>
2. <span data-ttu-id="0f96d-161">A felhőalapú tárolással a StorSimple-szolgáltatások előnyeit.</span><span class="sxs-lookup"><span data-stu-id="0f96d-161">Take advantage of StorSimple services for using cloud storage.</span></span>
3. <span data-ttu-id="0f96d-162">Az Azure Backup szolgáltatás segítségével.</span><span class="sxs-lookup"><span data-stu-id="0f96d-162">Use the Azure Backup service.</span></span>

### <a name="database"></a><span data-ttu-id="0f96d-163">Adatbázis</span><span class="sxs-lookup"><span data-stu-id="0f96d-163">Database</span></span>

1. <span data-ttu-id="0f96d-164">Tekintse át a jelen dokumentum adatbázis szakaszát.</span><span class="sxs-lookup"><span data-stu-id="0f96d-164">Review the Database section of this document.</span></span>
2. <span data-ttu-id="0f96d-165">Fontolja meg az SQL Server Azure virtuális gépeken a biztonsági másolattal.</span><span class="sxs-lookup"><span data-stu-id="0f96d-165">Consider using SQL Server on Azure VMs as the backup.</span></span>
3. <span data-ttu-id="0f96d-166">AlwaysOn rendelkezésre állási csoportok beállításához.</span><span class="sxs-lookup"><span data-stu-id="0f96d-166">Set up AlwaysOn Availability Groups.</span></span>
4. <span data-ttu-id="0f96d-167">Konfigurálja az ügyféltanúsítvány-alapú adatbázis-tükrözés.</span><span class="sxs-lookup"><span data-stu-id="0f96d-167">Configure certificate-based database mirroring.</span></span>
5. <span data-ttu-id="0f96d-168">Használja a naplóküldésben.</span><span class="sxs-lookup"><span data-stu-id="0f96d-168">Use log shipping.</span></span>
6. <span data-ttu-id="0f96d-169">Készítsen biztonsági másolatot a helyszíni adatbázisok az Azure Blob storage.</span><span class="sxs-lookup"><span data-stu-id="0f96d-169">Back up on-premises databases to Azure Blob storage.</span></span>

<!-- markdownlint-enable MD024 -->
