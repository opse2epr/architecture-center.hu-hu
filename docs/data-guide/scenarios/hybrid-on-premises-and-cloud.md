---
title: "Kiterjesztése a helyszíni data-megoldásokat a felhőben"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 66fc225dc123202ba587d82f15ea0883e1bbf3b5
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/14/2018
---
# <a name="extending-on-premises-data-solutions-to-the-cloud"></a>Kiterjesztése a helyszíni data-megoldásokat a felhőben

Ha a szervezet számítási feladatok és az adatok a felhőben, a helyszíni adatközpontjaikat gyakran továbbra is fontos szerepet játszanak. A kifejezés *hibrid felhő* nyilvános felhő kombinációja hivatkozik, és a helyszíni adatközpontok egyaránt kiterjedő, integrált informatikai környezet. Egyes szervezetek elérési útnak a hibrid felhő használja a teljes adatközpont áttelepítéséhez a felhő eltávolítása idő. Más felhőalapú szolgáltatások segítségével bővítheti a meglévő helyszíni infrastruktúrát. 

Ez a cikk ismerteti az egyes szempontok és a hibrid felhőalapú megoldás adatok kezelésére vonatkozó ajánlott eljárások

## <a name="when-to-use-a-hybrid-solution"></a>Hibrid megoldás használatával

Vegye figyelembe a hibrid megoldás használatával a következő esetekben:

* Mint áttérési stratégia teljes mértékben felhőalapú natív megoldásokhoz hosszabb távú áttelepítés során.
* Ha szabályzat vagy házirendek nem teszik lehetővé meghatározott adatok vagy a munkaterhelések áthelyezését a felhőbe.
* A vész helyreállítási és a hibatűrés érdekében replikálja az adatokat és szolgáltatásokat között a helyszíni és felhőalapú környezetek.
* A helyszíni adatközpontban és a távoli helyeken közötti késés csökkentése helyez el az Azure-ban a architektúrájának a része.

## <a name="challenges"></a>Kihívásai

* Konzisztens biztonsági, kezelési és fejlesztési környezet létrehozása, és elkerülhető az a párhuzamos munkát.

* Létrehozása egy megbízható, alacsony késést és a helyszíni és a felhő közötti biztonságos adatkapcsolat környezetekben.

* Az adatok replikálása és módosítása, alkalmazások és eszközök használata a megfelelő adatokat tárolja, minden környezetben.

* Biztonságossá tétele és érhető el a helyszíni, vagy fordítva, de a felhőben tárolt adatok titkosítása.

## <a name="on-premises-data-stores"></a>A helyszíni adattárolókhoz.

A helyszíni adattárolókhoz adatbázisok és a fájlok tartalmazzák. Megtartja ezeket helyi számos oka lehet. Nem lehetséges, hogy szabályzat vagy házirendeket, amelyek nem teszik lehetővé meghatározott adatok vagy a munkaterhelések áthelyezését a felhőbe. Adatok közös joghatóság alá, adatvédelmi és biztonsági szempontok a helyszíni elhelyezési előfordulhat, hogy alkalmazást. A áttelepítés során érdemes lehet arra, hogy néhány adat helyi olyan alkalmazás, amely még nem lettek áttelepítve.

Az alkalmazásadatok helyezi el a nyilvános felhők szempontokat figyelembe venni:

* **Költség**. Lehet, hogy az Azure-ban tárolók költségét jelentősen alacsonyabb, mint egy helyszíni adatközpontját a hasonló jellemzőkkel rendelkező tárolási fenntartásának költsége. Természetesen sok vállalat rendelkezik már meglévő befejtetések a csúcskategóriás San hálózatok, ezért ezeket az előnyöket költség nem jut el teljes átszállításokat meglévő hardver elavulnak amíg.
* **Rugalmasan méretezhető**. Tervezési és adatok kezelése a helyszíni környezetben kapacitás növekedési kihívást jelenthet, különösen nehezen becsülhető, az adatmennyiség-növekedés esetén. Ezek az alkalmazások előnyeit élvezheti az igény szerinti kapacitás és gyakorlatilag korlátlan tárterület érhető el a felhőben. A szempont, hogy kevesebb megfelelő alkalmazás esetében, amely viszonylag statikus méretű adatkészletek alkotják.
* **Vész-helyreállítási**. Az Azure-ban tárolt adatok automatikusan replikálható az Azure-régiót belül, mind a földrajzi régiók között. Hibrid környezetekben ezek a technológiák segítségével replikálja a helyszíni és felhőalapú adattárolók között.

## <a name="extending-data-stores-to-the-cloud"></a>Adatok kiterjesztése tárolja a felhőbe

Többféle módon is kiterjesztése a felhőbe a helyszíni adattárolókhoz. Egy beállítás az, hogy a helyszíni és felhőalapú replikák. Ez magas fokú hibatűrést eléréséhez, de előfordulhat, hogy módosítaná alkalmazások csatlakozzanak a feladatátvétel esetén a megfelelő tárolót.

Egy másik lehetőség, hogy az adatok egy részét a felhőbeli tárhelyén, ugyanakkor változatlanul megőrizze a több aktuális vagy több magas elért adatokat a helyszíni. Ez a módszer is adjon meg egy hosszú távú tárolás költséghatékonyabb beállítása, valamint csökkenti a működési adatkészlet javíthatja a data access válaszidejét.

A harmadik lehetőség egy tartani minden adatokról a helyszínen, de a felhőalapú informatika meghatározására alkalmazások üzemeltetését. Ehhez az szükséges, akkor az alkalmazás a felhőben tárolni, és csatlakoztassa a helyszíni adattárolóihoz biztonságos kapcsolaton keresztül. 

## <a name="azure-stack"></a>Azure Stack

A teljes hibrid felhőalapú megoldás, érdemes lehet [Microsoft Azure verem](/azure/azure-stack/). Az Azure verem egy hibrid felhő platform, amely lehetővé teszi az Adatközpont az Azure-szolgáltatásokhoz biztosít. Ezzel a megoldással biztosítja az egységességet a helyszíni és az Azure, közötti azonos eszközök használatával, és nem kód módosítani kellene. 

Az alábbiakban néhány használati esetekben az Azure és az Azure-verem:

* **Edge és a leválasztott megoldások**. -A késés és a hálózati kapcsolati követelményeinek helyileg Azure-veremben lévő adatok feldolgozása és a további elemzés, a közös alkalmazáslogikát keresztül is az Azure-ban majd összesítése. 
* **Felhőalapú alkalmazásokhoz, amelyek megfelelnek a változatos szabályzat**. Fejlesztése alkalmazások és központi telepítésekor az Azure, a rugalmasságot, központi telepítéséhez a azonos alkalmazások helyszíni Azure verem előírásoknak megfelelő vagy a házirend követelményeinek.
* **A felhő alkalmazás modell helyszíni**. Azure használatával frissítse és kiterjesztése a meglévő alkalmazások vagy új címkék létrehozása. Használja a konzisztens DevOps feldolgozza egész Azure a felhőben és helyszíni Azure verem.

## <a name="sql-server-data-stores"></a>SQL Server-adattároló

Ha a helyszíni SQL Server fut, a Microsoft Azure Blob Storage szolgáltatással a biztonsági mentéshez, és állítsa vissza. További információkért lásd: [SQL Server biztonsági másolat és helyreállítás a Microsoft Azure Blob Storage szolgáltatás](/sql/relational-databases/backup-restore/sql-server-backup-and-restore-with-microsoft-azure-blob-storage-service). Ez a funkció lehetővé teszi az korlátlan telephelytől távoli tároló, és lehetővé teszi, hogy az azonos biztonsági mentések a helyszíni és az Azure virtuális gépen futó SQL Server rendszert futtató SQL-kiszolgáló között. 

[Az Azure SQL Database](/azure/sql-database/) egy felügyelt relációs adatbázis-mint – a szolgáltatás. Mivel az Azure SQL-adatbázis a Microsoft SQL Server adatbázismotor használja, az alkalmazások férhetnek a két technológia ugyanúgy adatok. Az Azure SQL adatbázis is kombinálható az SQL Server hasznos módon. Például a [SQL Server Stretch Database](/sql/sql-server/stretch-database/stretch-database) funkció lehetővé teszi egy alkalmazás eléréséhez, míg más SQL Server-adatbázis egyetlen táblájára néz vagy az Azure SQL Database tárolódhat, hogy a tábla összes sorát. Ez a technológia automatikusan mozgatja az adatokat, amely nem érhető el egy meghatározott ideig a felhőbe. Ezek az adatok olvasása alkalmazások nem tudnak, hogy minden adat áthelyezése megtörtént a felhőbe.

Adatok kezelése a helyszínen tárolja, és a felhőben komoly kihívást jelenthet, ha a felügyelni kívánt a szinkronizált adatok megőrzése. Meg lehet oldani a [SQL adatszinkronizálás](/azure/sql-database/sql-database-sync-data), egy szolgáltatás, amely az Azure SQL Database, amely lehetővé teszi, hogy szinkronizálja az adatokat választ ki, két irányban több Azure SQL-adatbázisok és SQL Server-példányokat. Adatszinkronizálás megkönnyíti az adatok naprakészen tarthatók ezek különböző adattárolók között, amíg azt nem használható vész-helyreállítási vagy a helyszíni SQL Server áttelepítéséhez az Azure SQL Database.

A vész helyreállítási és üzleti folytonossági, használhat [AlwaysOn rendelkezésre állási csoportok](/sql/database-engine/availability-groups/windows/overview-of-always-on-availability-groups-sql-server) replikálják az adatokat két vagy több példányát az SQL Server között, ezek közül néhány futhat egy másik földrajzi régióban Azure virtuális gépeken .

## <a name="network-shares-and-file-based-data-stores"></a>Hálózati megosztások és tárolók fájlalapú adatok

A hibrid felhő architektúrák esetében gyakori, egy szervezet tartani az újabb fájlok helyszíni régebbi fájlokat a felhőbe archiválás során. Ez néha hívott fájl rétegezéséhez, ahol mindkét zökkenőmentes hozzáférést fájlok beállítja, a helyszíni és felhőben üzemeltetett. Ez a megközelítés segít minimalizálása érdekében a hálózati sávszélesség használatának, és hozzáférési idejének újabb fájlokat, amelyek valószínűleg a gyakran elért a legtöbb. Egy időben a felhőalapú tárolás előnyeit az archivált adatok beolvasása. 

A szervezetek is kíván, helyezze át a hálózati megosztások teljes egészében a felhőbe. Lenne szükséges, például, ha elérhető azokat az alkalmazásokat is található a felhőben. Ezzel az eljárással teheti használatával [adatok előkészítése](../technology-choices/pipeline-orchestration-data-movement.md) eszközök.


[Az Azure StorSimple](/azure/storsimple/) a helyszíni eszközök és az Azure felhőalapú tárolást közötti tárolási feladatok felügyeletének a legteljesebb körű integrált tárolási megoldást kínál. StorSimple egy hatékony, költséghatékony és könnyen kezelhető tárolási terület tárolóhálózat (SAN) megoldás, amely kiküszöböli a problémák és a vállalati tárolás és az adatvédelem társított is számos. Ez a saját fejlesztésű StorSimple 8000 series eszközt használ, integrálható a felhőszolgáltatásokkal, és integrált felügyeleti eszközöket biztosít.

Egy másik helyi hálózati megosztások mellett felhőalapú a file storage használata módja a [Azure fájlok](/azure/storage/files/storage-files-introduction). Az Azure fájlok ajánlatok teljes körűen felügyelt, így hozzáférhet a normál fájlmegosztásokhoz [Server Message Block](https://msdn.microsoft.com/library/windows/desktop/aa365233.aspx?f=255&MSPPError=-2147217396) (SMB) protokoll (más néven CIFS). Azure-fájlok csatlakoztathatnak egy fájlt a helyi számítógépen található megosztott, vagy használhatja őket a már meglévő alkalmazásokkal, hogy hozzáférési helyi, illetve a hálózati megosztás fájlok.

Azure-fájlokban szereplő fájlmegosztások szinkronizálja a helyi Windows Server kiszolgálókon, használja a [Azure fájlszinkronizálás](/azure/storage/files/storage-sync-files-planning). Egyik fő előnye, hogy Azure fájlszinkronizálás is réteg fájlokat a helyi fájlkiszolgáló és az Azure-fájlok között. Ez lehetővé teszi mindig csak a legújabb és során elért fájlokat helyileg. 

További információkért lásd: [Azure Blob Storage tárolóban, a Azure fájlok vagy az Azure-lemezek használatára való](/azure/storage/common/storage-decide-blobs-files-disks).

## <a name="hybrid-networking"></a>Hibrid hálózatkezelés

Ez a cikk a megoldások összpontosított hibrid adatokat, de meg kell vizsgálni, hogyan terjeszthető ki a helyszíni hálózat az Azure-bA. Hibrid megoldások ezen jellemzője kapcsolatos további információkért lásd a következő témaköröket:

- [Egy a helyszíni hálózathoz való kapcsolódáshoz Azure megoldás kiválasztása](../../reference-architectures/hybrid-networking/considerations.md)
- [Hibrid hálózati referencia architektúra](../../reference-architectures/hybrid-networking/index.md)

