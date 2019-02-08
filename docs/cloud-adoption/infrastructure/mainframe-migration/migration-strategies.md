---
title: 'Nagyszámítógépek migrálása: Győződjön meg arról, a kapcsoló Nagyszámítógépek az Azure-bA'
description: A nagyszámítógépes környezetek alkalmazásokat át az Azure-ba, a bevált, magas rendelkezésre állású és méretezhető infrastruktúrát Nagyszámítógépek a jelenleg futó rendszerek.
author: njray
ms.date: 12/26/2018
ms.openlocfilehash: 9243f757182f95cc227fd6cd7a5374f9c1371ccc
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899474"
---
# <a name="make-the-switch-from-mainframes-to-azure"></a>Győződjön meg arról, a kapcsoló Nagyszámítógépek az Azure-bA

A hagyományos nagyszámítógépes alkalmazások futtatása egy alternatív platform Azure kínál a rendkívül jól méretezhető számítási és tárolási magas rendelkezésre állású környezetben. Az érték és a egy modern, felhőalapú platform rugalmasságának kap egy nagyszámítógépes környezethez társított költségek nélkül.

Ez a témakör a műszaki útmutatást, hogy a kapcsoló egy nagyszámítógépes platform az Azure-bA.

![A nagyszámítógépes és az Azure](../../_images/mainframe-migration/make-the-switch.png)

## <a name="mips-vs-vcpus"></a>MIPS és vcpu-k

Nincs leképezés univerzális képlet, amely létezik a következő virtuális központi feldolgozóegység (Vcpu) nagyszámítógépes számítási feladatok futtatásához szükséges összegének meghatározása van. A metrika egy millió másodpercenként (MIPS) azonban gyakran leképezve vcpu-k az Azure-ban. MIPS ciklusok egy adott gép másodpercenkénti száma állandó érték megadásával a nagyszámítógépes általános számítási teljesítményt méri.

Egy kis méretű szervezet szükség lehet kisebb, mint 500 MIPS, amíg egy nagy szervezet általában több mint 5000 MIPS használ. 1000 dollárnál egyetlen MIPS, jelenleg egy nagy szervezet várólistában körülbelül $5 millió dollár 5000-MIPS infrastruktúra üzembe helyezése. A tipikus Azure üzemelő példánya a skála éves költségbecslést a körülbelül tized költségét MIPS infrastruktúra. További információkért lásd: 4. a táblázat a [Demystifying nagyszámítógépes – Azure Migrálási](https://azure.microsoft.com/resources/demystifying-mainframe-to-azure-migration) (tanulmány).

MIPS vcpu-k az Azure-ral való pontos kiszámításához vCPU, és futtatja a pontos munkaterhelés típusától függ. Azonban a teljesítményteszt tanulmányok jó alapot nyújtanak száma és típusa kell Vcpu-becslésére. Egy nemrég [HPE zREF teljesítményteszt](https://h20195.www2.hpe.com/v2/getpdf.aspx/4aa4-2452enw.pdf) az alábbi becslések biztosít:

- Online (CICS) feladatok HP Proliant kiszolgálókon futó Intel-alapú magonként 288 MIPS.

- 170 MIPS Intel magonként COBOL batch-feladatokhoz.

Ez az útmutató a kötegelt feldolgozáshoz vCPU / online feldolgozásra és 100 MIPS vCPU / 200 MIPS becslése.

> [!NOTE]
> Ezek a becslések hozatalakor megváltozhatnak, amint új virtuális gép (VM) sorozat az Azure-ban elérhetővé válnak.

## <a name="high-availability-and-failover"></a>Magas rendelkezésre állás és feladatátvétel

A nagyszámítógépes rendszerek gyakran kínálnak öt 9-es (99,999 %-os) nagyszámítógépes összekapcsolással és párhuzamos Sysplex használt rendelkezésre állási. Még üzemeltetők továbbra is szeretné ütemezni a karbantartási állásidő és a program kezdeti (IPLs) tölti be. Az aktuális rendelkezésre megközelítések két vagy három 9 hatékonysága eléri a csúcskategóriás, Intel-alapú kiszolgáló-es.

Ezzel az Azure kínál a kötelezettségvállalás-alapú szolgáltatásiszint-szerződések (SLA), ahol több 9-es rendelkezésre áll az alapértelmezett, optimalizált és a szolgáltatások helyi vagy a földrajzi alapú replikáció.

Az Azure biztosítja, hogy a további rendelkezésre állási replikálja az adatokat több tárolóeszközökről, helyi vagy egyéb földrajzi régiókban. A számítási erőforrások egy Azure-alapú hiba esetén hozzáférhet a replikált adatok vagy a helyi és regionális szinten.

Használhatja az Azure platform, a platformszolgáltatás (PaaS) erőforrások, például [Azure SQL Database](/azure/sql-database/sql-database-technical-overview) és [Azure Cosmos Database](/azure/cosmos-db/introduction), az Azure automatikusan képes kezelni a feladatátvételeket. Azure-infrastruktúra (IaaS) szolgáltatás használatakor feladatátvételi adott rendszer működőképességét, például az SQL Server AlwaysOn funkciókat, a feladatátvételi fürtözési példányok és a rendelkezésre állási csoportok támaszkodik.

## <a name="scalability"></a>Méretezhetőség

Nagyszámítógépek általában vertikális felskálázás, felhőalapú környezetekben horizontális felskálázás közben. Nagyszámítógépek horizontális felskálázása lehetséges egy kapcsoló létesítmény (CF) használatával, de a magas költségeknek hardver-és tárolási teszi Nagyszámítógépek horizontális felskálázási rendkívül drága lenne.

Emellett az a CF-hez szorosan összekapcsolt számítási biztosít, mivel az Azure kibővített funkcióit lazán vannak összekapcsolva. A felhő méretezhetők felfelé vagy lefelé egyezés pontos felhasználói előírások, a számítási, tárolási és szolgáltatások, használatalapú számlázási modellben igény szerinti méretezés.

## <a name="backup-and-recovery"></a>Biztonsági másolat és helyreállítás

A nagyszámítógépes ügyfelek általában vész-helyreállítási hely karbantartása, vagy győződjön meg arról, használatra vagy egy katasztrófa utáni ágakba független nagygépes-szolgáltatót. A vész-helyreállítási webhelyként való szinkronizálás általában fügvényekkel adatok offline példányait. Mindkét lehetőség nagy költségekkel.

Automatizált georedundancia is keresztül érhető el a nagyszámítógépes kapcsolási létesítmény, jóllehet nagyszerű költségére, és általában fenntartott kritikus fontosságú rendszereken. Ezzel szemben az Azure rendelkezik a könnyen megvalósítható és költséghatékony lehetőségei [biztonsági mentési](/azure/backup/backup-introduction-to-azure-backup), [helyreállítási](/azure/site-recovery/site-recovery-overview), és [redundancia](/azure/storage/common/storage-redundancy) helyi és regionális szinten, vagy a geo-redundancia használatával.

## <a name="storage"></a>Storage

Nagyszámítógépek működésének megértése része magában foglalja a különféle egymást átfedő kifejezéseket dekódolása. Például központi tárolására, tényleges memória, valós tárolási és fő storage minden általánosan tekintse meg a nagyszámítógépes processzor közvetlenül csatlakoztatott tároló.

A nagyszámítógépes hardver processzorok és sok más eszközök, például a közvetlen hozzáférés tárolóeszközök (DASDs), a mágnesszalag meghajtók és számos különböző típusú felhasználói konzolok tartalmazza. Szalagok és DASDs rendszer függvények és a felhasználói programok használt.

Nagyszámítógépek fizikai tárolási típusok a következők:

- Központi tároló: Közvetlenül a nagyszámítógépes processzor található, ez a néven is ismert processzor- és tárolási valós.

- Kiegészítő tárolás: Található külön-külön, a nagyszámítógépes, ez a típus magában foglalja a tárolási DASDs a, és más néven lapozási tárolási.

A felhő rugalmas, méretezhető beállítások számos szolgáltatást kínál, és ki kell fizetnie csak ezek a lehetőségek, amelyek van szüksége. [Az Azure Storage](/azure/storage/common/storage-introduction) az adatobjektumok nagymértékben skálázható objektumtároló, a fájlrendszer-szolgáltatását a felhőben, egy megbízható üzenetküldési tárolóban és a NoSQL-alapú tárolót biztosít. Felügyelt és nem felügyelt lemezek virtuális gépekhez, állandó és biztonságos lemezes tárolás adja meg.

## <a name="mainframe-development-and-testing"></a>A nagyszámítógépes fejlesztési és tesztelési célra

A nagyszámítógépes migrálási projekt fő illesztőprogram egy olyan alkalmazásfejlesztés változó felületére. Vállalat szeretne fejlesztői környezetükben, hatékony és rugalmas, üzleti igényeknek megfelelően.

Nagyszámítógépek külön logikai partíciót (LPARs) fejlesztési és tesztelési, például a QA és átmeneti LPARs általában rendelkeznek. Nagyszámítógépes alkalmazásfejlesztési megoldások közé tartozik a fordítóprogramot (COBOL, PL / I Assembler) és szerkesztőktől kezdve. A leggyakoribb eset az interaktív rendszer termelékenység létesítmény (ISPF), amely IBM Nagyszámítógépek futtat az operációs rendszer z/operációs rendszer. Közé tartozik például az ROSCOE programozási létesítmény (RPF) és a számítógép társítja eszközök, például hitelesítésszolgáltató Librarian és a CA-Panvalet.

Emulációs környezetek és fordítóprogramot érhetők el a x86 platformon, így a fejlesztési és tesztelési célra általában lehet egy nagyszámítógépes migrálhat az Azure-bA az első munkaterhelések között. A rendelkezésre állás és a széles körű használatát [DevOps-eszközök az Azure-ban](https://azure.microsoft.com/solutions/devops/) fejlesztési áttelepítésének felgyorsítása és tesztelési környezetben.

Megoldások fejlesztett és tesztelt Azure-ra és a nagyszámítógépes való üzembe helyezéshez készen áll, amikor szüksége lesz a nagyszámítógépes a kód másolásához és fordítsa le van.

## <a name="next-steps"></a>További lépések

> [!div class="nextstepaction"]
> [A nagyszámítógépes alkalmazások áttelepítése](application-strategies.md)
