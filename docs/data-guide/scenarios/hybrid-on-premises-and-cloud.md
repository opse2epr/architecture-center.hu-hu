---
title: Helyszíni adatmegoldások kiterjesztése a felhőre
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.openlocfilehash: dcffe32883e379dd17c1f54a7eeebda3dab8d808
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298941"
---
# <a name="extending-on-premises-data-solutions-to-the-cloud"></a>Helyszíni adatmegoldások kiterjesztése a felhőre

Amikor szervezetek számítási feladatok és az adatok áthelyezése a felhőbe, a helyszíni adatközpontok gyakran továbbra is fontos szerepet játszanak. Az előfizetési időszak *hibrid felhő* nyilvános felhő kombinációja hivatkozik, és a helyszíni adatközpontok egyaránt kiterjedő integrált informatikai környezet. Egyes szervezetek elérési útként hibrid felhő segítségével áttelepítheti a teljes adatközpontját és a felhő áthelyezése időpontja. Más szervezetek a cloud services használatával kiterjesztheti meglévő helyszíni infrastruktúrán.

Ez a cikk azt ismerteti, néhány megfontolandó szempontok és a egy hibrid felhőmegoldás, az adatok kezelésére vonatkozó ajánlott eljárások

## <a name="when-to-use-a-hybrid-solution"></a>Mikor érdemes használni egy hibrid megoldás

Vegye figyelembe, hogy egy hibrid megoldás használatával a következő esetekben:

- Átmenet stratégiánk hosszabb távú egy teljes mértékben felhőalapú natív megoldás az áttelepítés során.
- Amikor szabályzat vagy szabályzatok nem engedélyezik meghatározott adatok vagy a számítási feladatok áthelyezése a felhőbe.
- A vész helyreállítási és hibatűréssel kezelik, adatait és szolgáltatásait a helyszíni és felhő közötti replikálásával környezetekben.
- A helyszíni adatközpont és a távoli helyeken, a késés csökkentése üzemeltetése az Azure-ban az architektúra egy része.

## <a name="challenges"></a>Problémák

- Konzisztens biztonsági, felügyeleti és fejlesztési környezet létrehozását és work felesleges ismétlésének elkerülése érdekében.

- Hoz létre egy megbízható, alacsony késleltetés és a helyszíni és a felhő között biztonságos adatkapcsolat környezetekben.

- Replikálja az adatokat, és a módosítása, alkalmazások és eszközök a megfelelő adatok használata minden környezetben tárolja.

- Biztonságossá tétele és elérni a helyszíni, vagy fordítva, de a felhőben tárolt adatok titkosítása.

## <a name="on-premises-data-stores"></a>A helyszíni adattárak

A helyszíni adattárak közé tartozik, adatbázisok és a fájlokat. Előfordulhat, hogy megtartja ezeket helyi számos oka lehet. Hiba lehetséges, hogy szabályzat vagy szabályzatok, amely nem engedi meg az adott adat vagy számítási feladatait a felhőbe való áthelyezés. Adatok szuverenitását, adatvédelmi vagy biztonsági aggályokat előnyben részesíti a helyszíni elhelyezését. Áttelepítés során érdemes arra, hogy néhány adat helyi olyan alkalmazás, amely még nem lettek áttelepítve.

Az alkalmazásadatok helyezi a nyilvános felhőben szempontok közé tartoznak:

- **Költség**. Az Azure-beli tárolási költsége jelentősen kisebb, mint egy helyszíni adatközpont a hasonló jellemzőkkel rendelkező tárolási tartanunk lehet. Természetesen számos vállalat rendelkezik meglévő beruházásait a csúcskategóriás San hálózatok, így ezek költségtakarékos nem jut el teljes átszállításokat mindaddig, amíg a meglévő hardver életkorok.

- **Rugalmas skálázás**. Megtervezéséhez és kezeléséhez az adatok kapacitás növekedési a helyszíni környezetben kihívást jelenthet, különösen akkor, ha Adatnövekedés nehéz előre jelezni. Ezeket az alkalmazásokat a felhőben a kapacitás igény szerinti és gyakorlatilag korlátlan kapacitású kihasználhatják. A szempontok fontos kevesebb viszonylag statikus méretű adatkészletek álló alkalmazásokat.

- **Vész-helyreállítási**. Az Azure-ban tárolt adatok automatikusan replikálhatók az Azure régiókon belül és a földrajzi régióban. A hibrid környezetekben ezek a technológiák segítségével replikálja a helyszíni és felhőalapú adattárak között.

## <a name="extending-data-stores-to-the-cloud"></a>A felhőben tárolja az adatok kiterjesztése

Többféle módon is kiterjeszti a felhőbe helyszíni adattárakban. Az egyik lehetőség az, hogy a helyszíni és felhőbeli replikákat. Ez a magas fokú hibatűrést kialakíthatja, de lehet szükség az alkalmazások csatlakozzanak a feladatátvétel esetén a megfelelő adattár módosítása.

Egy másik lehetőség, hogy az adatok egy részét a felhőbeli tárhelyére, miközben gondoskodik a több aktuális vagy a szigorúbban elért adatok a helyszínen. Ez a módszer is lehetőséget nyújtanak olyan több költséghatékony hosszú távú tárolás céljából, valamint növelheti a data access válaszidők csökkenti a működési adatkészlet.

Egy harmadik lehetőség, hogy az összes adat a helyi környezetben tarthatja, de a felhő-számítástechnikai alkalmazások üzemeltetéséhez. Ehhez az alkalmazások felhőbeli üzemeltetéséhez és biztonságos kapcsolaton keresztül csatlakozni a helyszíni adattárban lenne.

## <a name="azure-stack"></a>Azure Stack

Egy teljes körű hibrid felhőalapú megoldás, érdemes lehet használatával [a Microsoft Azure Stack](/azure/azure-stack/). Az Azure Stack egy hibrid felhőplatform, amely lehetővé teszi az adatközpontból Azure-szolgáltatásokat biztosíthat az. Ez segít a helyszíni és az Azure közötti konzisztencia megtartása által azonos eszközök használatával, és megköveteli a programkód módosítása nélkül.

Az alábbiakban néhány használati esetek, Azure és az Azure Stack:

- **Peremhálózati és hálózat nélküli megoldásoknál**. Késéssel és elérhetőséggel kapcsolatos követelmények cím helyileg az Azure Stackben adatok feldolgozását, és majd összesíti az Azure-ban történő közös alkalmazás-logikával további elemzéshez.

- **Felhőbeli alkalmazásokat, amelyek megfelelnek a változatos szabályzat**. Fejlesszen és helyezzen üzembe alkalmazásokat az Azure-ban, és rugalmasan üzembe helyezheti az azonos helyszíni alkalmazásokhoz az Azure Stack használatával szabályozási vagy a házirend követelményeinek.

- **Felhőalapú alkalmazás helyszíni modellje**. Frissítés és a meglévő alkalmazásaik vagy újak létrehozása az Azure segítségével. Használjon egységes fejlesztési-üzemeltetési folyamatokat a felhőben Azure-ban és az Azure Stack a helyszíni.

## <a name="sql-server-data-stores"></a>Az SQL Server-adattárak

Ha a helyszíni SQL Server futtatja, biztonsági mentést a Microsoft Azure Blob Storage-szolgáltatás használata, és állítsa vissza. További információkért lásd: [SQL Server biztonsági mentés és helyreállítás a Microsoft Azure Blob Storage szolgáltatás](/sql/relational-databases/backup-restore/sql-server-backup-and-restore-with-microsoft-azure-blob-storage-service). Ezt a képességet biztosít korlátlan külső helyszínen történő tárolás, és lehetővé teszi, hogy az azonos biztonsági mentések a helyszíni és az Azure-beli virtuális gépeken futó SQL Server rendszert futtató SQL Server között.

[Az Azure SQL Database](/azure/sql-database/) egy felügyelt relációs adatbázis-szolgáltatásként van. Mivel az Azure SQL Database a Microsoft SQL Server motort használja, az alkalmazások férhetnek mindkét technológiáival ugyanúgy adataihoz. Az Azure SQL Database is kombinálható az SQL Server hasznos módon. Ha például a [SQL Server Stretch Database](/sql/sql-server/stretch-database/stretch-database) funkció lehetővé teszi, hogy egy alkalmazás eléréséhez, hogyan néz ki az SQL Server-adatbázis, míg egyetlen tábla, vagy adott tábla összes sora előfordulhat, hogy tárolja, az Azure SQL Database-ben. Ez a technológia automatikusan áthelyezi az adatokat, amely nem érhető el egy meghatározott ideig a felhőbe. Ezek az adatok olvasása alkalmazások deduplikálta, hogy minden adat áthelyezése megtörtént a felhőbe.

Adatokat tárolja a helyszíni és felhőbeli kihívást jelenthet, ha kíván megőrizni az adatokat szinkronizálni. Meg lehet oldani a [SQL Data Sync](/azure/sql-database/sql-database-sync-data), egy szolgáltatás, amely az Azure SQL Database, amely lehetővé teszi, hogy szinkronizálja az adatokat, választja, kétirányúan több Azure SQL Database-adatbázisok és SQL Server-példányokat. Adatszinkronizálás megkönnyíti az adatok naprakészen tartani a különböző adattárak között, amíg azt nem használható vész-helyreállítási vagy a helyszíni SQL Serverről az Azure SQL Database történő áttelepítéséhez.

A vészhelyreállítási és üzletmenet-folytonossági, használhatja [AlwaysOn rendelkezésre állási csoportok](/sql/database-engine/availability-groups/windows/overview-of-always-on-availability-groups-sql-server) adatait replikálhatja az SQL Server két vagy több példányát, amelyek némelyike futhat egy másik földrajzi régióban található Azure-beli virtuális gépek .

## <a name="network-shares-and-file-based-data-stores"></a>Hálózati megosztások és tárolók fájlalapú adatok

A hibrid felhő architektúrában szokás újabb fájlok a helyi környezetben tarthatja, régebbi fájlokat a felhőbe archiválás során a szervezet számára. Ez néha nevű fájl rétegezést, ahol nincs közvetlen elérését fájlok beállítja, a helyszíni és felhőben üzemeltetett. Ez a megközelítés segít a sávszélesség-használat minimalizálása érdekében, és újabb fájlokat, amelyek valószínűleg az elérés időpontját elérhető a legtöbb gyakran. Egy időben a felhőalapú tárolási előnyeit archivált adatok beolvasása.

Szervezetek is Kezdésként érdemes lehet a hálózati megosztások teljes egészében a felhőbe helyezheti át. Ez akkor lehet célszerű, például, ha az alkalmazást, hogy azokat a felhőben is található. Ezzel az eljárással teheti meg a [adatkoordinálás](../technology-choices/pipeline-orchestration-data-movement.md) eszközök.

[Az Azure StorSimple](/azure/storsimple/) kezelése a helyszíni eszközök és az Azure cloud storage közötti tárolási feladatokat a legteljesebb körű integrált tárolási megoldást kínál. StorSimple egy hatékony, költségkímélő és könnyen felügyelhető terület tárolóhálózat (SAN) tárolómegoldás, amely a problémák és a költségeket, a nagyvállalati adattárolás és -védelem számos költsége kiküszöbölhető a. Azt a jogvédett StorSimple 8000 sorozatú eszköz használ, a cloud Services szolgáltatásokkal integrálható, és biztosít egy integrált felügyeleti eszközökkel.

Egy másik módja a helyszíni hálózati megosztások felhőalapú fájltároló együtt használandó [Azure Files](/azure/storage/files/storage-files-introduction). Azure Files-ajánlatok teljes körűen felügyelt fájlmegosztásokhoz, így hozzáférhet a standard [Server Message Block](https://msdn.microsoft.com/library/windows/desktop/aa365233.aspx?f=255&MSPPError=-2147217396) (SMB) protokoll (más néven CIFS). Egy fájl megosztása a helyi számítógépen, vagy ezeket együtt használni a meglévő alkalmazásokat, hogy hozzáférési helyi vagy hálózati megosztás fájlokat akkor is csatlakoztathatja az Azure Files.

Az Azure Files fájlmegosztások szinkronizálja a helyi Windows-kiszolgálók, használja a [Azure File Sync](/azure/storage/files/storage-sync-files-planning). Egyik legnagyobb előnye az Azure File Sync szint fájlokat a helyi fájlkiszolgáló és az Azure Files között lehetősége. Ez lehetővé teszi, hogy csak a legújabb és elért fájlok helyileg.

További információkért lásd: [való használata az Azure Blob storage, az Azure Files és az Azure Disks](/azure/storage/common/storage-decide-blobs-files-disks).

## <a name="hybrid-networking"></a>Hibrid hálózatkezelés

Ez a cikk a megoldások összpontosított hibrid adatokat, de egy másik szempont, hogyan lehet a helyszíni hálózat kiterjesztése az Azure-bA. További információ a hibrid megoldások ezt az alábbi témakörök:

- [Egy a helyszíni hálózathoz való kapcsolódáshoz Azure megoldás kiválasztása](../../reference-architectures/hybrid-networking/considerations.md)
- [Hibrid hálózat referenciaarchitektúrák](../../reference-architectures/hybrid-networking/index.md)
