---
title: Biztonságos webes alkalmazások létrehozása az Azure-beli Windows virtuális gépek
description: Biztonságos, többszintű webalkalmazást hozhat létre a Windows Serverrel az Azure-ban méretezési csoportok, az Application Gateway és terheléselosztók használatával.
author: iainfoulds
ms.date: 12/06/2018
ms.custom: seodec18
ms.openlocfilehash: 4e4d2117fbc46eda46f7ef276a71739e3a79270e
ms.sourcegitcommit: 4ba3304eebaa8c493c3e5307bdd9d723cd90b655
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/12/2018
ms.locfileid: "53307061"
---
# <a name="building-secure-web-applications-with-windows-virtual-machines-on-azure"></a>Biztonságos webes alkalmazások létrehozása a Windows virtual machines az Azure-ban

Ebben a forgatókönyvben a biztonságos és többszintű webalkalmazások a Microsoft Azure-on futó architektúra és kialakítás útmutatást nyújt. Ebben a példában az ASP.NET-alkalmazások biztonságos módon csatlakozik egy védett háttér-Microsoft SQL Server-fürt virtuális gépek használatával.

Hagyományosan szervezetek kellett karbantartása örökölt helyszíni alkalmazások és szolgáltatások biztonságos infrastruktúrát biztosít. Ezek a Windows Server az alkalmazások biztonságos az Azure-beli üzembe helyezésével, szervezetek korszerűsítheti a telepítések és csökkentheti a helyszíni üzemeltetési költségeket és munkaterhelést.

## <a name="relevant-use-cases"></a>Alkalmazási helyzetek

Néhány példa, ahol az ebben a forgatókönyvben alkalmazhatja:

* Alkalmazástelepítések alapú modernizálásánál megfelelő választás a következő biztonságos felhőalapú környezetben.
* Csökkenti a kezelési terhelést örökölt helyszíni alkalmazások és szolgáltatások.
* Betegek healthcare javítása és a élményük az új alkalmazás-platformokat.

## <a name="architecture"></a>Architektúra

![Az Azure-összetevőket Windows Server alkalmazás többrétegű szabályozott iparágakban részt architektúrájának áttekintése][architecture]

Ebből a forgatókönyvből megtudhatja csatlakozik egy háttér-adatbázishoz, mindkét Windows Server 2016-on futó előtér-webalkalmazást. A áramlanak keresztül az adatok a forgatókönyv a következő:

1. Felhasználók keresztül az Azure Application Gateway az előtérbeli ASP.NET-alkalmazás elérésére.
2. Az Application Gateway elosztja a forgalmat egy Azure-beli virtuálisgép-méretezési csoportban lévő Virtuálisgép-példányok között.
3. Az alkalmazás a Microsoft SQL Server-fürtöt a háttér-szinten az Azure load balancer keresztül csatlakozik. A háttér-az SQL Server példányai egy külön Azure virtuális hálózatban, védi a hálózati biztonsági csoport szabályait, amelyek korlátozzák a forgalmat.
4. A terheléselosztó elosztja a forgalmat az SQL Server egy másik virtuálisgép-méretezési csoportban lévő Virtuálisgép-példányok között.
5. Az Azure Blob Storage úgy működik, mint egy [felhőbeli tanúsító] [ cloud-witness] a háttér-szintű SQL Server-fürt számára. A kapcsolat a virtuális hálózaton belül a virtuális hálózati szolgáltatásvégpontot az Azure Storage engedélyezve van.

### <a name="components"></a>Összetevők

* [Az Azure Application Gateway] [ appgateway-docs] egy 7. rétegbeli webes forgalom terheléselosztó, amely alkalmazásbarát és terjeszthetnek a forgalmat az adott útválasztási szabályok alapján. Alkalmazásátjáró is képes kezelni az SSL-kiürítés, a továbbfejlesztett webkiszolgáló teljesítményét.
* [Az Azure Virtual Network] [ vnet-docs] lehetővé teszi az erőforrások, például virtuális gépek biztonságosan kommunikálnak egymással, az internethez, és a helyszíni hálózatokkal. Virtuális hálózatok adja meg az elkülönítés és Szegmentálás, szűrő és útvonal-forgalom és helyek közötti kapcsolat engedélyezését. Két virtuális hálózat kombinálva a megfelelő NSG-k segítségével ebben a forgatókönyvben adja meg egy [demilitarizált zóna] [ dmz] (DMZ) és az alkalmazás-összetevők elkülönítését. A két hálózat virtuális hálózatok közötti társviszony együtt kapcsolódik.
* [Azure-beli virtuálisgép-méretezési csoport] [ scaleset-docs] lehetővé teszi, hogy hozzon létre és manager azonos, load csoportja elosztott terhelésű virtuális gépek. A virtuálisgép-példányok száma automatikusan növelhető vagy csökkenthető a pillanatnyi igényeknek megfelelően vagy egy meghatározott ütemezés szerint. Ez a forgatókönyv - egyet az előtér-ASP.NET alkalmazás példányai, és a egy SQL Server fürt háttérbeli Virtuálisgép-példányokhoz két külön virtuálisgép-méretezési csoportok használhatók. PowerShell kívánt állapot configuration (DSC) vagy az Azure egyéni szkriptek futtatására szolgáló bővítmény a szükséges szoftverekkel és a konfigurációs beállítások VM-példányok kiépítéséhez használható.
* [Azure-beli hálózati biztonsági csoportok] [ nsg-docs] tartalmaznak, amelyek engedélyezik vagy megtagadják a bejövő vagy kimenő hálózati forgalmat a forrás vagy cél IP-cím, port és protokoll alapján biztonsági szabályokból álló listát. Ebben a forgatókönyvben a virtuális hálózatok a hálózati biztonsági csoport szabályait, amelyek korlátozzák a forgalmat az alkalmazás-összetevők közötti biztonságosak.
* [Az Azure load balancer] [ loadbalancer-docs] osztja el a szabályok és az állapotmintákat megfelelően bejövő forgalmat. Egy terheléselosztót biztosít alacsony késéssel és nagy átviteli sebességet, és akár több milliónyi összes TCP és UDP-alkalmazás méretezhető. Belső terheléselosztó szolgál ebben a forgatókönyvben a háttér SQL Server-fürt az előtér-alkalmazás szintről forgalom elosztása.
* [Az Azure Blob Storage] [ cloudwitness-docs] működik az SQL Server-fürt Felhőbeli tanúsító helyét. A tanúsító szolgál a fürt működését és a egy dönthet arról, hogy a kvórum további szavazata szükséges döntéseket. Használatával a Felhőbeli tanúsító nincs szükség az működjön egy hagyományos tanúsító fájlmegosztást, egy további virtuális Gépre.

### <a name="alternatives"></a>Alternatív megoldások

* A Linux és Windows felcserélhetők óta az infrastruktúra nem függ az operációs rendszer.

* [Az SQL Server for Linux] [ sql-linux] lecserélheti a háttér-tárolót.

* [A cosmos DB](/azure/cosmos-db/introduction) egy másik alternatíva az adattároló.

## <a name="considerations"></a>Megfontolandó szempontok

### <a name="availability"></a>Rendelkezésre állás

Ebben a forgatókönyvben a VM-példányokon rendelkezésre állási zónában üzemelnek. Minden zóna egy vagy több adatközpont független áramellátással, hűtéssel és hálózati található tevődik össze. Az összes engedélyezett régiókban érhető el egy minimum három zónából állnak. Ehhez a terjesztéshez zónákban Virtuálisgép-példányok az alkalmazásrétegek magas rendelkezésre állást biztosít. További információkért lásd: [Mik a rendelkezésre állási zónák az Azure-ban?][azureaz-docs]

Az adatbázisszint Always On rendelkezésre állási csoportok használatára konfigurálható. Ez az SQL Server-konfigurációval a fürtön belül több elsődleges adatbázis nyolc másodlagos adatbázisok van konfigurálva. Ha a probléma akkor fordul elő, az elsődleges adatbázissal, a fürt átadja a feladatokat egy másodlagos adatbázis, amely lehetővé teszi az alkalmazás továbbra is elérhetők. További információkért lásd: [áttekintése az Always On rendelkezésre állási csoportokat az SQL Server][sqlalwayson-docs].

Több rendelkezésre állási útmutatóért lásd: a [rendelkezésre állási ellenőrzőlista] [ availability] a az Azure Architecture Centert.

### <a name="scalability"></a>Méretezhetőség

Ebben a forgatókönyvben a virtual machine scale sets az előtér- és összetevőket használ. A méretezési csoportok az előtér-alkalmazás szinten futtató Virtuálisgép-példányok száma automatikusan méretezheti az ügyfelek igényei szerint válaszul, vagy egy meghatározott ütemezés alapján. További információkért lásd: [az automatikus méretezés a virtual machine scale sets áttekintése][vmssautoscale-docs].

Méretezhetőség témaköröket talál a [méretezési ellenőrzőlista] [ scalability] a az Azure Architecture Centert.

### <a name="security"></a>Biztonság

A virtuális hálózat hálózati biztonsági csoportok által védett, és a forgalom az előtér-alkalmazás szinten be. A szabályok korlátozzák a forgalmat, hogy csak az előtér-alkalmazás szintű Virtuálisgép-példányok férhessenek hozzá a háttér adatbázis szint. Nincs kimenő internetes forgalom engedélyezve van az adatbázisszint. A támadás által elfoglalt tárterület csökkentéséhez nincs közvetlen Távoli szolgáltatásfelügyelet portjai nyitva. További információkért lásd: [Azure-beli hálózati biztonsági csoportok][nsg-docs].

Útmutató megtekintése a Payment Card Industry Data Security Standards (PCI DSS 3.2) üzembe helyezése [megfelelő infrastruktúra][pci-dss]. Biztonságos forgatókönyvek tervezésével kapcsolatos általános útmutatásért tekintse meg a [Azure Security dokumentációja][security].

### <a name="resiliency"></a>Rugalmasság

A rendelkezésre állási zónák és a virtual machine scale sets együtt ebben a forgatókönyvben az Azure Application Gateway és a load balancer használ. E két hálózati összetevők csatlakoztatott Virtuálisgép-példányok forgalom elosztását, és biztosíthatja, hogy a forgalom csak megtörténik a kifogástalan állapotú virtuális gépeket az állapot-mintavételei bele. Két Application Gateway-példány egy aktív-passzív konfigurációban vannak konfigurálva, és a egy zónaredundáns load balancert használja. Ez a konfiguráció lehetővé teszi a hálózati erőforrásokhoz, és az alkalmazás rugalmas problémákra utal, amelyek egyébként zavarja a forgalom és végfelhasználói hozzáférése.

Rugalmas forgatókönyvek tervezésével kapcsolatos általános útmutatásért lásd: [rugalmas alkalmazások tervezése az Azure][resiliency].

## <a name="deploy-the-scenario"></a>A forgatókönyv megvalósításához

**Előfeltételek.**

* Meglévő Azure-fiókkal kell rendelkeznie. Ha nem rendelkezik Azure-előfizetéssel, mindössze néhány perc alatt létrehozhat egy [ingyenes fiókot](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) a virtuális gép létrehozásának megkezdése előtt.
* A háttér-méretezési csoportot az SQL Server-fürt üzembe helyezése, meg kell lennie egy tartományhoz az Azure Active Directory (AD) Domain Servicesben.

Az alapvető infrastruktúra ebben a forgatókönyvben egy Azure Resource Manager-sablon üzembe helyezéséhez hajtsa végre az alábbi lépéseket.

1. Válassza ki a **üzembe helyezés az Azure** gombra:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Finfrastructure%2Fregulated-multitier-app%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. Várja meg a sablon üzembe helyezéséhez az Azure Portal megnyitásához, majd kövesse az alábbi lépéseket:
   * Válassza ki a **új létrehozása** erőforrás csoportra, majd adjon meg egy nevet például *myWindowsscenario* a szövegmezőben.
   * Válassza ki a régiót, a **hely** legördülő listából.
   * Adjon meg egy felhasználónevet és a biztonságos jelszó a virtuális gép méretezési csoport példányaihoz.
   * Tekintse át a használati feltételeket, majd ellenőrizze **elfogadom a feltételeket és a fenti feltételeket**.
   * Válassza ki a **beszerzési** gombra.

Az üzembe helyezés befejeződik 15 – 20 percet is igénybe vehet.

## <a name="pricing"></a>Díjszabás

Ebben a forgatókönyvben költségének megismeréséhez, a szolgáltatások mindegyike a költségkalkulátor az előre konfigurált. Tekintse meg, hogyan díjszabását szeretné módosítani az adott használati esetekhez, módosítsa a megfelelő változókat egyezik a várt forgalomhoz.

A méretezési csoport Virtuálisgép-példányain az alkalmazásokat futtató száma alapján három példa költség profilok adtunk meg.

* [Kis][small-pricing]: a díjszabási példa korrelációt keres a két előtér-és két háttérbeli Virtuálisgép-példányok között.
* [Közepes][medium-pricing]: 20 előtér- és 5 háttérbeli Virtuálisgép-példányok a díjszabási példa utal.
* [Nagy][large-pricing]: 100 előtérrendszer, mind a 10 háttérbeli Virtuálisgép-példányok a díjszabási példa utal.

## <a name="related-resources"></a>Kapcsolódó források (lehet, hogy a cikkek angol nyelvűek)

Ebben a forgatókönyvben egy háttérbeli virtuális gép méretezési csoportot, amely a Microsoft SQL Server-fürt használja. A cosmos DB is használható, méretezhető és biztonságos adatbázis-rétegből az alkalmazásadatok számára. Egy [Azure virtuális hálózati szolgáltatásvégpont] [ vnetendpoint-docs] lehetővé teszi, hogy csak a virtuális hálózatot a kritikus fontosságú Azure-szolgáltatási erőforrások védelmét. Ebben a forgatókönyvben a VNet-végpontok engedélyezése az előtér-alkalmazás szint és a Cosmos DB közötti adatforgalom biztonságossá teheti. További információkért lásd: a [Azure Cosmos DB áttekintő](/azure/cosmos-db/introduction).

Részletes megvalósítási útmutatók, tekintse át a [referenciaarchitektúra az SQL Server használatával N szintű alkalmazások][ntiersql-ra].

<!-- links -->
[appgateway-docs]: /azure/application-gateway/overview
[architecture]: ./media/architecture-regulated-multitier-app.png
[autoscaling]: /azure/architecture/best-practices/auto-scaling
[availability]: ../../checklist/availability.md
[azureaz-docs]: /azure/availability-zones/az-overview
[cloudwitness-docs]: /windows-server/failover-clustering/deploy-cloud-witness
[loadbalancer-docs]: /azure/load-balancer/load-balancer-overview
[nsg-docs]: /azure/virtual-network/security-overview
[ntiersql-ra]: /azure/architecture/reference-architectures/n-tier/n-tier-sql-server
[resiliency]: /azure/architecture/resiliency/ 
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability 
[scaleset-docs]: /azure/virtual-machine-scale-sets/overview
[sqlalwayson-docs]: /sql/database-engine/availability-groups/windows/overview-of-always-on-availability-groups-sql-server
[vmssautoscale-docs]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview
[vnet-docs]: /azure/virtual-network/virtual-networks-overview
[vnetendpoint-docs]: /azure/virtual-network/virtual-network-service-endpoints-overview
[pci-dss]: /azure/security/blueprints/pcidss-iaaswa-overview
[dmz]: /azure/virtual-network/virtual-networks-dmz-nsg
[sql-linux]: /sql/linux/sql-server-linux-overview?view=sql-server-linux-2017
[cloud-witness]: /windows-server/failover-clustering/deploy-cloud-witness
[small-pricing]: https://azure.com/e/711bbfcbbc884ef8aa91cdf0f2caff72
[medium-pricing]: https://azure.com/e/b622d82d79b34b8398c4bce35477856f
[large-pricing]: https://azure.com/e/1d99d8b92f90496787abecffa1473a93
