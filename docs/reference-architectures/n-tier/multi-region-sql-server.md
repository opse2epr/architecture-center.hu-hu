---
title: Többrégiós N szintű alkalmazás magas rendelkezésre állás érdekében
description: Virtuális gépek üzembe helyezése több régióban az Azure-on a magas rendelkezésre állás és rugalmasság érdekében.
author: MikeWasson
ms.date: 07/19/2018
pnp.series.title: Windows VM workloads
pnp.series.prev: n-tier
ms.openlocfilehash: 34dd47175e7fd0002cba577ad6c1034968ed4098
ms.sourcegitcommit: b2a4eb132857afa70201e28d662f18458865a48e
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 10/05/2018
ms.locfileid: "48819125"
---
# <a name="multi-region-n-tier-application-for-high-availability"></a>Többrégiós N szintű alkalmazás magas rendelkezésre állás érdekében

Ez a referenciaarchitektúra néhány bevált eljárást mutat be egy N szintű alkalmazás több Azure-régióban való futtatásához a magas rendelkezésre állás eléréséhez és egy robusztus vészhelyreállítási infrastruktúra kiépítéséhez. 

[![0]][0] 

*Töltse le az architektúra [Visio-fájlját][visio-download].*

## <a name="architecture"></a>Architektúra 

Ez az architektúra épít látható [SQL Servert használó N szintű alkalmazás](n-tier-sql-server.md). 

* **Elsődleges és másodlagos régiók**. Használjon két régiót a magas rendelkezésre állás eléréséhez. Az egyik ebben az esetben az elsődleges régió. A másik a feladatátvétel során használt régió.

* **Azure Traffic Manager**. A [Traffic Manager][traffic-manager] az egyik régió felé irányítja a beérkező kérelmeket. A normál működés során a rendszer az elsődleges régió felé irányítja a kérelmeket. Ha az adott régió nem érhető el, a Traffic Manager átadj a feladatokat a másodlagos régiónak. További információt a következő szakaszban talál: [A Traffic Manager konfigurációja](#traffic-manager-configuration).

* **Erőforráscsoportok**. Hozzon létre külön [erőforráscsoportokat][resource groups] az elsődleges régióhoz, a másodlagos régióhoz, valamint a Traffic Managerhez. Ennek köszönhetően rugalmasan, egyetlen erőforráskészletként kezelheti az egyes régiókat, például ismételten üzembe helyezhet egy régiót a másik eltávolítása nélkül. [Kapcsolja össze az erőforráscsoportokat][resource-group-links], hogy egy lekérdezés futtatásával listázhassa az alkalmazás összes erőforrását.

* **Virtuális hálózatok**. Hozzon létre külön virtuális hálózatot mindegyik régióhoz. Ügyeljen arra, hogy a címterek ne legyenek átfedésben. 

* **SQL Server Always On rendelkezésre állási csoport**. Az SQL Server használata esetén az [SQL Server Always On rendelkezésre állási csoportok][sql-always-on] használata javasolt a magas rendelkezésre állás érdekében. Hozzon létre egyetlen rendelkezésre állási csoportot, amely mindkét régióban tartalmazza az SQL Server-példányokat. 

    > [!NOTE]
    > Másik lehetőségként fontolja meg az [Azure SQL Database][azure-sql-db] használatát, amely relációs adatbázist biztosít felhőszolgáltatás formájában. Az SQL Database használatával nem kell rendelkezésre állási csoportot konfigurálnia, vagy kezelnie a feladatátvételt.  
    > 

* **VPN-átjárók**. Hozzon létre egy [VPN-átjárót][vpn-gateway] mindkét virtuális hálózatban, és konfiguráljon egy [virtuális hálózatok közötti kapcsolatot][vnet-to-vnet], amely lehetővé teszi a hálózati forgalmat a kettő között. Ez szükséges az SQL Always On rendelkezésre állási csoport használatához.

## <a name="recommendations"></a>Javaslatok

A többrégiós architektúrák magasabb rendelkezésre állást biztosíthatnak, mint az egyetlen régióban történő üzembe helyezés. Ha egy régió üzemkimaradása hatással van az elsődleges régióra, a [Traffic Manager][traffic-manager] szolgáltatással feladatátvételt hajthat végre a másodlagos régióba. Ez az architektúra az alkalmazás egy adott alrendszerének meghibásodása esetén is segíthet.

A régiók közötti magas rendelkezésre állás elérésének több általános megközelítése van: 

* Aktív/passzív készenléti kiszolgálóval. A forgalom az egyik régióra irányul, míg a másik készenléti kiszolgálón várakozik. Készenléti kiszolgáló esetében a másodlagos régió virtuális gépei folyamatosan le vannak foglalva és futnak.
* Aktív/passzív hidegtartalékkal. A forgalom az egyik régióra irányul, míg a másik hidegtartalékként áll készenlétben. Hidegtartalék esetében a másodlagos régió virtuális gépei nincsenek lefoglalva, amíg nem szükségessé nem válnak feladatátvétel céljából. Ezen megközelítés futtatása kevesebb költséggel jár, de a meghibásodáskor általában hosszabb időt vesz igénybe az üzembe állás.
* Aktív/aktív. Mindkét régió aktív, és a kérelmek terheléselosztással oszlanak meg közöttük. Ha egy régió elérhetetlenné válik, kikerül a rotációból. 

Ez a referenciaarchitektúra a készenléti kiszolgálóval konfigurált aktív/passzív üzemmódra összpontosít, a Traffic Managert használva a feladatátvételhez. Vegye figyelembe, hogy ha kis számú virtuális gépet helyez üzembe készenléti kiszolgálóként, igény szerint később is elvégezheti a horizontális felskálázásukat.

### <a name="regional-pairing"></a>Régiónkénti párosítás

Minden egyes Azure-régió párban áll egy másikkal egy azonos földrajzi területen belül. Régiókat általában azonos regionális párokból érdemes választani (például: USA 2. keleti régiója és USA középső régiója). Ennek előnyei például a következők:

* Széles körű leállás esetén a helyreállításkor minden párból legalább az egyik régió előnyt élvez.
* A tervezett Azure-rendszerfrissítések egyszerre csak a régiópár egyik tagján jelennek meg, ami csökkenti az állásidőt.
* A párok azonos földrajzi helyen belül találhatók, hogy megfeleljenek az adatok tárolási helyére vonatkozó előírásoknak. 

Azonban győződjön meg arról, hogy mindkét régió támogatja az összes Azure-szolgáltatást, amely szükséges az alkalmazásához (lásd: [Szolgáltatások régiónként][services-by-region]). További információ a regionális párokról: [Üzletmenet-folytonosság és vészhelyreállítás (BCDR): Az Azure párosított régiói][regional-pairs].

### <a name="traffic-manager-configuration"></a>A Traffic Manager konfigurációja

A Traffic Manager konfigurálásakor vegye figyelembe a következő szempontokat:

* **Útválasztás**. A Traffic Manager többféle [útválasztási algoritmust][tm-routing] támogat. A cikkben leírt forgatókönyvhöz a *prioritásos* útválasztást (régebbi nevén *feladatátvétel esetén használt* útválasztás) használja. Ezzel a beállítással a Traffic Manager az összes kérelmet az elsődleges régió felé irányítja, kivétel akkor, ha az nem elérhető. Ebben az esetben automatikusan átadja a feladatokat a másodlagos régiónak. Lásd: [A feladatátvétel esetén használt útválasztás konfigurálása][tm-configure-failover].
* **Állapotminta**. A Traffic Manager HTTP- (vagy HTTPS-) [mintavételt][tm-monitoring] használ az egyes régiók rendelkezésre állásának monitorozására. A mintavétel 200-as HTTP-választ vár egy megadott URL-címhez. Ajánlott eljárásként hozzon létre egy olyan végpontot, amely az alkalmazás általános állapotáról ad jelentést, és ezt a végpontot használja az állapotmintához. Ellenkező esetben előfordulhat, hogy a mintavétel megfelelően működő végpontot jelent, miközben az alkalmazás kritikus fontosságú részei valójában hibásak. További információk: [Állapot végponti monitorozását végző minta][health-endpoint-monitoring-pattern].   

Amikor a Traffic Manager átadja a feladatokat, az alkalmazás egy ideig nem lesz elérhető a felhasználók számára. Ez az időtartam a következő tényezőktől függ:

* Az állapotmintának észlelnie kell, ha az elsődleges régió elérhetetlenné válik.
* A DNS-kiszolgálóknak frissíteniük kell az IP-címhez tartozó gyorsítótárazott DNS-rekordokat. Ez a DNS élettartamától (TTL) függ. Az alapértelmezett TTL 300 másodperc (5 perc), de ezt az értéket a Traffic Manager-profil létrehozásakor konfigurálhatja.

Részletek: [A Traffic Manager monitorozásának ismertetése][tm-monitoring].

Ha a Traffic Manager feladatátvételt hajt végre, automatikus feladat-visszavétel megvalósítása helyett a manuális feladat-visszavételt ajánlunk. Ellenkező esetben előfordulhat, hogy egyes esetekben az alkalmazás oda-vissza váltogat a régiók között. A feladat-visszavétel előtt ellenőrizze, hogy az alkalmazás minden alrendszere működik-e.

Vegye figyelembe, hogy a Traffic Manager alapértelmezés szerint automatikusan végrehajtja a feladat-visszavételt. Ennek megelőzéséhez manuálisan csökkentse az elsődleges régió prioritását a feladatátvétel után. Tegyük fel például, hogy az elsődleges régió 1-es prioritású, a második pedig 2-es. A feladatátvétel után az automatikus visszavétel megelőzéséhez állítsa az elsődleges régió prioritását 3-asra. A prioritást akkor állítsa 1-esre, ha már készen áll a visszaváltásra.

A következő [Azure CLI][azure-cli]-parancsot frissíti a prioritást:

```bat
az network traffic-manager endpoint update --resource-group <resource-group> --profile-name <profile>
    --name <endpoint-name> --type azureEndpoints --priority 3
```    

Másik megoldásként ideiglenesen letilthatja a végpontot, amíg készen nem áll a feladat-visszavételre:

```bat
az network traffic-manager endpoint update --resource-group <resource-group> --profile-name <profile>
    --name <endpoint-name> --type azureEndpoints --endpoint-status Disabled
```

A feladatátvétel okától függően előfordulhat, hogy újra üzembe kell helyeznie az erőforrásokat a régión belül. A feladat-visszavétel előtt tesztelje a működési készenlétet. A teszt többek között a következőket ellenőrzi:

* A virtuális gépek megfelelően vannak-e konfigurálva. (Az összes szükséges szoftver telepítve van, az IIS fut stb.)
* Az alkalmazás alrendszerei működőképesek-e. 
* Funkciótesztelés. (Pl. az adatbázisszint elérhető a webes szintről.)

### <a name="configure-sql-server-always-on-availability-groups"></a>Az SQL Server Always On rendelkezésre állási csoportok konfigurálása

A Windows Server 2016-nál régebbi kiadásokban az SQL Server Always On rendelkezésre állási csoportok tartományvezérlőt igényelnek, és a rendelkezésre állási csoport összes csomópontjának azonos Active Directory (AD) tartományban kell lennie. 

Az rendelkezésre állási csoport konfigurálása:

* Legalább két tartományvezérlőt helyezzen mindegyik régióba.
* Minden tartományvezérlőhöz rendeljen egy statikus IP-címet.
* Hozzon létre egy virtuális hálózatok közötti kapcsolatot a virtuális hálózatok közötti kommunikációhoz.
* Minden egyes virtuális hálózat esetében adja hozzá a tartományvezérlők IP-címeit (mindkét régióból) a DNS-kiszolgálók listájához. Ezt az alábbi CLI-paranccsal teheti meg. További információkért lásd: [módosítása DNS-kiszolgálók][vnet-dns].

    ```bat
    az network vnet update --resource-group <resource-group> --name <vnet-name> --dns-servers "10.0.0.4,10.0.0.6,172.16.0.4,172.16.0.6"
    ```

* Hozzon létre egy [Windows Server feladatátvételi fürtszolgáltatást][wsfc] (WSFC) fürtöt, amely mindkét régióban tartalmazza SQL Server-példányokat. 
* Hozzon létre egy SQL Server Always On rendelkezésre állási csoportot, amely tartalmazza az elsődleges és a másodlagos régió SQL Server-példányait. Ennek lépéseit az [Always On rendelkezésre állási csoport kiterjesztése távoli Azure adatközpontra (PowerShell)](https://blogs.msdn.microsoft.com/sqlcat/2014/09/22/extending-alwayson-availability-group-to-remote-azure-datacenter-powershell/) című cikkben találja.

  * Az elsődleges régiót helyezze az elsődleges régióba.
  * Helyezzen egy vagy több másodlagos replikát az elsődleges régióba. Konfigurálja azokat a szinkron véglegesítés használatára, automatikus feladatátvétellel.
  * Helyezzen egy vagy több másodlagos replikát a másodlagos régióba. Ezeket a teljesítmény érdekében *aszinkron* véglegesítés használatára konfigurálja. (Ellenkező esetben az összes T-SQL-tranzakciónak várnia kell, míg az adatok körbeérnek a hálózaton a másodlagos régióig.)

    > [!NOTE]
    > Az aszinkron véglegesítésű replikák nem támogatják az automatikus feladatátvételt.
    >
    >

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

Komplex N szintű alkalmazás esetén előfordulhat, hogy nem kell replikálnia a teljes alkalmazást a másodlagos régióba. Ehelyett elég lehet replikálni egy kritikus fontosságú alrendszert, amely az üzletmenet folytonosságának fenntartásához szükséges.

A Traffic Manager a rendszer egyik lehetséges meghibásodási pontja. Ha a Traffic Manager szolgáltatás meghibásodik, az ügyfelek a leállás ideje alatt nem érhetik el az alkalmazást. Tekintse át a [Traffic Manager szolgáltatói szerződését][tm-sla], és döntse el, hogy a Traffic Manager egyedüli használata elegendő-e vállalata magas rendelkezésre állásra vonatkozó követelményeihez. Ha nem, akkor a biztonság érdekében érdemes lehet hozzáadni egy másik forgalomkezelési szolgáltatást a feladat-visszavételhez. Ha az Azure Traffic Manager szolgáltatás meghibásodik, módosítsa a CNAME rekordját a DNS-ben úgy, hogy a többi forgalomkezelő szolgáltatásra mutasson. (Ezt a lépést manuálisan kell elvégezni. Amíg a DNS-módosítások érvénybe nem lépnek, az alkalmazás nem lesz elérhető.)

Az SQL Server-fürt esetében két feladatátvételi forgatókönyvet kell figyelembe venni:

- Az összes SQL Server-adatbázisreplika meghibásodik az elsődleges régióban. Ez például regionális kimaradás során fordulhat elő. Ebben az esetben manuálisan kell elvégeznie a rendelkezésre állási csoport feladatátvételét, annak ellenére, hogy a Traffic Manager az előtérben automatikusan elvégzi a feladatátvételt. Kövesse a [Kényszerített manuális feladatátvétel elvégzése SQL Server rendelkezésre állási csoporton](https://msdn.microsoft.com/library/ff877957.aspx) című cikk lépéseit. A cikk leírja, hogyan végezhető kényszerített feladatátvétel az SQL Server Management Studio, a Transact-SQL vagy a PowerShell használatával az SQL Server 2016-ban.

   > [!WARNING]
   > A kényszerített feladatátvétel esetében adatvesztés fordulhat elő. Amint az elsődleges régió újra elérhetővé válik, készítsen pillanatfelvételt az adatbázisról, és használja a [tablediff] parancsot a különbségek megkereséséhez.
   >
   >
- A Traffic Manager átadja a feladatokat a másodlagos régiónak, de az elsődleges SQL Server-adatbázisreplika továbbra is elérhető marad. Például előfordulhat, hogy az előtérréteg meghibásodik, de ez nincs hatással az SQL Servert futtató virtuális gépekre. Ebben az esetben a rendszer átirányítja az internetes forgalmat a másodlagos régióba, és ez a régió továbbra is csatlakozhat az elsődleges replikához. Ekkor azonban nagyobb lesz a késés, mivel az SQL Server-kapcsolatoknak több régión kell áthaladniuk. Ebben a helyzetben manuálisan kell végrehajtania a feladatátvételt az alábbiak szerint:

   1. Ideiglenesen állítson át egy SQL Server-adatbázisreplikát a másodlagos régióban *szinkron* véglegesítésre. Ez biztosítja, hogy ne legyen adatvesztés a feladatátvétel során.
   2. Végezze el a feladatátvételt erre a replikára.
   3. Miután megtörtént a feladat-visszavétel az elsődleges régióra, állítsa vissza az aszinkron véglegesítést.

## <a name="manageability-considerations"></a>Felügyeleti szempontok

Amikor frissíti az üzemelő példányt, egyszerre egy régiót frissítsen. Ezzel csökkenthető a nem megfelelő konfiguráció vagy az alkalmazásban felmerülő hiba okozta globális meghibásodás esélye.

Tesztelje a rendszer meghibásodásokkal szembeni rugalmasságát. Alább található néhány gyakori meghibásodási helyzet:

* Állítson le virtuálisgép-példányokat.
* Terhelje az erőforrásokat, például a processzort és a memóriát.
* Bontsa/késleltesse a hálózati kapcsolatot.
* Váltsa ki egyes folyamatok összeomlását.
* Alkalmazzon lejárt tanúsítványokat.
* Szimuláljon hardverhibákat.
* Állítsa le a DNS-szolgáltatást a tartományvezérlőkön.

Mérje meg a helyreállítási időtartamokat, és győződjön meg róla, hogy azok megfelelnek az üzleti követelményeinek. Több hibaállapot kombinációját is tesztelje.



<!-- Links -->
[hybrid-vpn]: ../hybrid-networking/vpn.md
[azure-dns]: /azure/dns/dns-overview
[azure-sla]: https://azure.microsoft.com/support/legal/sla/
[azure-sql-db]: https://azure.microsoft.com/documentation/services/sql-database/
[health-endpoint-monitoring-pattern]: https://msdn.microsoft.com/library/dn589789.aspx
[azure-cli]: /cli/azure/
[regional-pairs]: /azure/best-practices-availability-paired-regions
[resource groups]: /azure/azure-resource-manager/resource-group-overview
[resource-group-links]: /azure/resource-group-link-resources
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[services-by-region]: https://azure.microsoft.com/regions/#services
[sql-always-on]: https://msdn.microsoft.com/library/hh510230.aspx
[tablediff]: https://msdn.microsoft.com/library/ms162843.aspx
[tm-configure-failover]: /azure/traffic-manager/traffic-manager-configure-failover-routing-method
[tm-monitoring]: /azure/traffic-manager/traffic-manager-monitoring
[tm-routing]: /azure/traffic-manager/traffic-manager-routing-methods
[tm-sla]: https://azure.microsoft.com/support/legal/sla/traffic-manager
[traffic-manager]: https://azure.microsoft.com/services/traffic-manager
[visio-download]: https://archcenter.blob.core.windows.net/cdn/vm-reference-architectures.vsdx
[vnet-dns]: /azure/virtual-network/manage-virtual-network#change-dns-servers
[vnet-to-vnet]: /azure/vpn-gateway/vpn-gateway-vnet-vnet-rm-ps
[vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[wsfc]: https://msdn.microsoft.com/library/hh270278.aspx

[0]: ./images/multi-region-sql-server.png "Magas rendelkezésre állású hálózati architektúra N szintű Azure-alkalmazásokhoz"
