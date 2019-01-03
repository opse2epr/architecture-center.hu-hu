---
title: Többrétegű webalkalmazás beépített magas rendelkezésre ÁLLÁS/vészhelyreállítás
titleSuffix: Azure Example Scenarios
description: Magas rendelkezésre állásra és vészhelyreállításra tervezett többrétegű webalkalmazást hozhat létre az Azure-ban Azure-beli virtuális gépek, rendelkezésre állási csoportok, rendelkezésre állási zónák, az Azure Site Recovery és az Azure Traffic Manager használatával
author: sujayt
ms.date: 11/16/2018
ms.custom: product-team
ms.openlocfilehash: baa468697b4a72975e3b192efc9bdf1861a0c0da
ms.sourcegitcommit: bb7fcffbb41e2c26a26f8781df32825eb60df70c
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/20/2018
ms.locfileid: "53644045"
---
# <a name="multitier-web-application-built-for-high-availability-and-disaster-recovery-on-azure"></a>A helyreállításhoz magas rendelkezésre állás és vészhelyreállítás az Azure-ban létrehozott többrétegű webalkalmazást

Ebben a példaforgatókönyvben olyan rugalmas, magas rendelkezésre állású és vész-helyreállítási Többrétegű alkalmazások telepítéséhez szükséges bármely iparági alkalmazható. Ebben a forgatókönyvben az alkalmazás három rétegből áll.

- Webes réteg: A legfelső rétege, beleértve a felhasználói felületen. Ez a réteg felhasználói interakció érdekében elemzi, és átadja a műveleteket következő rétege a feldolgozáshoz.
- Üzleti szint: A felhasználói interakció érdekében dolgoz fel, és lehetővé teszi a logikai döntéseket hozhat a következő lépésekről. Ez a réteg a webes szint és az adatszint kapcsolódik.
- Adatszint: Az alkalmazás adatait tárolja. Egy adatbázis, objektumtár vagy a file storage általában szolgál.

Gyakori alkalmazási helyzet közé tartozik minden olyan Windows vagy Linux rendszeren futó üzleti szempontból alapvető létfontosságú alkalmazás. Ez olyan megoldásszolgáltatóknál alkalmazásadatokat, például SAP és a SharePoint és a egy egyéni üzleti alkalmazás is lehet.

## <a name="relevant-use-cases"></a>Alkalmazási helyzetek

Egyéb alkalmazási helyzetek a következők:

- Rendkívül rugalmas alkalmazások, például az SAP- és SharePoint üzembe helyezése
- A üzletmenet-folytonossági és Vészhelyreállítási terv esetében – üzletági alkalmazások tervezése
- Vészhelyreállítás konfigurálása és megfelelőségi célokra kapcsolódó gyakorlatok végrehajtása

## <a name="architecture"></a>Architektúra

Ez a forgatókönyv azt mutatja be egy ASP.NET- és Microsoft SQL Servert használó többrétegű alkalmazás. A [Azure-régiók rendelkezésre állási zónákat támogató](/azure/availability-zones/az-overview#regions-that-support-availability-zones), a virtuális gépek (VM) üzembe helyezése a forrásrégióban rendelkezésre állási zónák között, és a virtuális gépek replikálása a vészhelyreállításhoz használt célrégió. Az Azure-régióban, amelyek nem támogatják a rendelkezésre állási zónák üzembe helyezése a virtuális gépek rendelkezésre állási csoportban, és a virtuális gépek replikálása a célrégióban.

![Egy rendkívül rugalmas többrétegű webes alkalmazás architektúrájának áttekintése][architecture]

- A virtuális gépeket az egyes szintek elosztása a zónákat támogató régiók két rendelkezésre állási zónák között. Más régióban üzembe helyezi a virtuális gépeket egy rendelkezésre állási csoporton belül az egyes csomagokban.
- Az adatbázisszint Always On rendelkezésre állási csoportok használatára konfigurálható. Ez az SQL Server-konfigurációval a fürtön belül több elsődleges adatbázis nyolc másodlagos adatbázisok van konfigurálva. Ha a probléma akkor fordul elő, az elsődleges adatbázissal, a fürt átadja a feladatokat egy másodlagos adatbázis, így az alkalmazása továbbra is elérhető. További információkért lásd: [áttekintése az Always On rendelkezésre állási csoportokat az SQL Server][docs-sql-always-on].
- Vész-helyreállítási helyzetekben konfigurálhatja az SQL Always On aszinkron natív replikációt a vészhelyreállításhoz használt célrégió. Beállíthatja az Azure Site Recovery replikálás a célrégióban, ha az adatváltozási sebessége az Azure Site Recovery támogatott határain belül van.
- Felhasználók férhetnek hozzá az előtérbeli ASP.NET webes réteg a traffic manager végpont.
- A traffic manager átirányítja a forgalmat a forrás elsődleges régió elsődleges nyilvános IP-címvégponthoz.
- A nyilvános IP-címet átirányítja a hívást egy, a webes szint Virtuálisgép-példányok egy nyilvános terheléselosztón keresztül. Az összes webes réteg Virtuálisgép-példányok egy alhálózaton vannak.
- A webes szintről VM minden hívás irányítja a rendszer feldolgozási egy belső terheléselosztón keresztül az üzleti szinten lévő Virtuálisgép-példányok közül. Egy külön alhálózatot minden üzleti szint virtuális gépek találhatók.
- A művelet feldolgozása az üzleti szinten lévő, és az ASP.NET-alkalmazás csatlakozik a Microsoft SQL Server-fürtöt a háttér-szinten az Azure belső terheléselosztó keresztül. A háttér-az SQL Server példányai egy külön alhálózatra.
- A traffic manager a másodlagos végpontra a vészhelyreállításhoz használt célrégió nyilvános IP-cím van beállítva.
- Egy elsődleges régióban bekövetkező szolgáltatáskimaradás esetén az Azure Site Recovery feladatátvételi indít el, és az alkalmazás aktívvá válik a célrégióban.
- A traffic manager-végpontot automatikusan átirányítja az ügyfél forgalmát a célrégióban a nyilvános IP-cím.

### <a name="components"></a>Összetevők

- [A rendelkezésre állási csoportok] [ docs-availability-sets] győződjön meg arról, hogy az Azure-ban üzembe helyezett virtuális gépek egy fürtben több elkülönített hardvercsomópont között legyenek elosztva. Ha hardveres vagy szoftveres hiba lép fel az Azure-ban, a virtuális gépeknek csak egy részhalmazát érinti, és a teljes megoldás elérhető és működőképes maradjon.
- [A rendelkezésre állási zónák] [ docs-availability-zones] az alkalmazások és adatok védelme az Adatközpont meghibásodása. A rendelkezésre állási zónák egy Azure-régión belüli különböző fizikai helyeken. Minden zóna egy vagy több adatközpont független áramellátással, hűtéssel és hálózati található áll.
- [Az Azure Site Recovery (ASR)] [ docs-azure-site-recovery] lehetővé teszi, hogy a virtuális gépek replikálása másik Azure-régióba az üzletmenet-folytonossági és vészhelyreállítási igények. A megfelelőségi igények teljesítése érdekében rendszeres vészhelyreállítási próbák végezhet. A rendszer a megadott beállításokkal replikálja a virtuális gépet a kiválasztott régióba, így Ön visszaállíthatja alkalmazásait, ha a forrásrégióban leállás történne.
- [Az Azure Traffic Manager] [ docs-traffic-manager] művelet során gondoskodik a magas rendelkezésre állásának és válaszkészségének globális Azure-régióban szolgáltatásokat egy DNS-alapú forgalom terheléselosztó, amely elosztja a forgalmat az optimális.
- [Az Azure Load Balancer] [ docs-load-balancer] osztja el a bejövő forgalom meghatározott szabályok és az állapotadat-mintavételek alapján. Egy terheléselosztót biztosít alacsony késéssel és nagy teljesítményű, akár több milliónyi összes TCP és UDP-alkalmazás méretezése. Nyilvános load balancer segítségével ebben a forgatókönyvben a webes szintre bejövő fürtforgalom elosztására. Belső terheléselosztó szolgál ebben a forgatókönyvben a háttér SQL Server-fürt az üzleti szintről való forgalom elosztása.

### <a name="alternatives"></a>Alternatív megoldások

- Windows kicserélhető más operációs rendszereken, mert a az infrastruktúra nem függ az operációs rendszer.
- [Az SQL Server for Linux] [ docs-sql-server-linux] lecserélheti a háttér-tárolót.
- Az adatbázis bármilyen elérhető standard szintű adatbázis-alkalmazás is kell helyettesíteni.

## <a name="other-considerations"></a>Egyéb szempontok

### <a name="scalability"></a>Méretezhetőség

Adja hozzá, vagy távolítsa el a virtuális gépeket az egyes szintek, a méretezési követelmények alapján. Mivel ebben a forgatókönyvben a terheléselosztók használ, adhat hozzá további virtuális gépeket egy szintű működésének megzavarása nélkül megtesztelheti az alkalmazások hasznos üzemideje.

Méretezhetőség témaköröket talál a [méretezési ellenőrzőlista] [ scalability] a az Azure Architecture Centert.

### <a name="security"></a>Biztonság

Az előtér-alkalmazás szinten az összes virtuális hálózati forgalmat hálózati biztonsági csoportok védelmét. A szabályok korlátozzák a forgalmat, hogy csak az előtér-alkalmazás szintű Virtuálisgép-példányok férhessenek hozzá a háttér adatbázis szint. Nincs kimenő internetes forgalmat az üzleti szint vagy az adatbázisszint engedélyezett. A támadás által elfoglalt tárterület csökkentéséhez nincs közvetlen Távoli szolgáltatásfelügyelet portjai nyitva. További információkért lásd: [Azure-beli hálózati biztonsági csoportok][docs-nsg].

Biztonságos forgatókönyvek tervezésével kapcsolatos általános útmutatásért tekintse meg a [Azure Security dokumentációja][security].

## <a name="pricing"></a>Díjszabás

Vészhelyreállítás konfigurálása az Azure virtuális gépek Azure Site Recovery használatával a következő díjat számolunk folyamatosan.

- Az Azure Site Recovery virtuális gépenkénti licencköltségeit.
- Hálózati adatkimeneti költségek adatváltozásokat replikálni a forrás Virtuálisgép-lemezek egy másik Azure-régióba. Az Azure Site Recovery beépített tömörítést használ a körülbelül 50 %-kal az adatok átvitel követelmények csökkentése érdekében.
- Tárolási költségek a helyreállítási helyre. Ez a általában ugyanaz, mint a régió forrástároló és a helyreállítási pontok fenntartásához pillanatképként a helyreállításhoz szükséges további tárolási.

Biztosítunk egy [minta költségkalkulátor] [ calculator] vész-helyreállítási hat virtual machines használatával háromszintű alkalmazás konfigurálásához. A szolgáltatások mindegyike a költségkalkulátor az előre konfigurált. Jelenik meg, hogyan díjszabását szeretné módosítani az adott használati esetekhez, módosítsa a megfelelő változókat megbecsülheti költségeit.

<!-- links -->
[architecture]: ./media/arhitecture-disaster-recovery-multi-tier-app.png
[autoscaling]: /azure/architecture/best-practices/auto-scaling
[availability]: ../../checklist/availability.md
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[docs-availability-zones]: /azure/availability-zones/az-overview
[docs-load-balancer]: /azure/load-balancer/load-balancer-overview
[docs-nsg]: /azure/virtual-network/security-overview
[docs-vmss]: /azure/virtual-machine-scale-sets/overview
[docs-sql-always-on]: /sql/database-engine/availability-groups/windows/overview-of-always-on-availability-groups-sql-server
[docs-vmss-autoscale]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview
[docs-vnet]: /azure/virtual-network/virtual-networks-overview
[docs-sql-server-linux]: /sql/linux/sql-server-linux-overview?view=sql-server-linux-2017
[docs-traffic-manager]: /azure/traffic-manager/
[docs-azure-site-recovery]: /azure/site-recovery/azure-to-azure-quickstart/
[docs-availability-sets]: /azure/virtual-machines/windows/manage-availability/
[calculator]: https://azure.com/e/6835332265044d6d931d68c917979e6d/
