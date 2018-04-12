---
title: Linux rendszerű virtuális gépek futtatása több Azure-régióban a magas rendelkezésre állás érdekében
description: Virtuális gépek üzembe helyezése több régióban az Azure-on a magas rendelkezésre állás és rugalmasság érdekében.
author: MikeWasson
ms.date: 11/22/2016
pnp.series.title: Linux VM workloads
pnp.series.prev: n-tier
ms.openlocfilehash: 07ccf44f28203e6d5001475b47adce01437e9600
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/30/2018
---
# <a name="run-linux-vms-in-multiple-regions-for-high-availability"></a>Linux rendszerű virtuális gépek futtatása több régióban a magas rendelkezésre állás érdekében

Ez a referenciaarchitektúra néhány bevált eljárást mutat be egy N szintű alkalmazás több Azure-régióban való futtatásához a magas rendelkezésre állás eléréséhez és egy robusztus vészhelyreállítási infrastruktúra kiépítéséhez. 

![[0]][0]

*Töltse le az architektúra [Visio-fájlját][visio-download].*

## <a name="architecture"></a>Architektúra 

Ez az architektúra a [Linux rendszerű virtuális gépek futtatása N szintű alkalmazáshoz](n-tier.md) című cikkben bemutatott architektúrára épít. 

* **Elsődleges és másodlagos régiók**. Használjon két régiót a magas rendelkezésre állás eléréséhez. Az egyik ebben az esetben az elsődleges régió, a másik pedig a feladatátvétel során használt régió.
* **Azure DNS**. Az [Azure DNS][azure-dns] egy üzemeltetési szolgáltatás, amely a Microsoft Azure infrastruktúráját használja a DNS-tartományok névfeloldásához. Ha tartományait az Azure-ban üzemelteti, DNS-rekordjait a többi Azure-szolgáltatáshoz is használt hitelesítő adatokkal, API-kkal, eszközökkel és számlázási információkkal kezelheti.
* **Azure Traffic Manager**. A [Traffic Manager][traffic-manager] az egyik régió felé irányítja a beérkező kérelmeket. A normál működés során a rendszer az elsődleges régió felé irányítja a kérelmeket. Ha az adott régió nem érhető el, a Traffic Manager átadj a feladatokat a másodlagos régiónak. További információt a következő szakaszban talál: [A Traffic Manager konfigurációja](#traffic-manager-configuration).
* **Erőforráscsoportok**. Hozzon létre külön [erőforráscsoportokat][resource groups] az elsődleges régióhoz, a másodlagos régióhoz, valamint a Traffic Managerhez. Ennek köszönhetően rugalmasan, egyetlen erőforráskészletként kezelheti az egyes régiókat, például ismételten üzembe helyezhet egy régiót a másik eltávolítása nélkül. [Kapcsolja össze az erőforráscsoportokat][resource-group-links], hogy egy lekérdezés futtatásával listázhassa az alkalmazás összes erőforrását.
* **Virtuális hálózatok**. Hozzon létre külön virtuális hálózatot mindegyik régióhoz. Ügyeljen arra, hogy a címterek ne legyenek átfedésben.
* **Apache Cassandra**. A Cassandra üzembe helyezése több Azure-régió adatközpontjaiban a magas rendelkezésre állás elérésének érdekében. A csomópontok minden régióban állvány-kompatibilis üzemmódban vannak konfigurálva tartalék és frissítési tartományokkal a régión belüli rugalmasság érdekében.

## <a name="recommendations"></a>Javaslatok

A többrégiós architektúrák magasabb rendelkezésre állást biztosíthatnak, mint az egyetlen régióban történő üzembe helyezés. Ha egy régió üzemkimaradása hatással van az elsődleges régióra, a [Traffic Manager][traffic-manager] szolgáltatással feladatátvételt hajthat végre a másodlagos régióba. Ez az architektúra az alkalmazás egy adott alrendszerének meghibásodása esetén is segíthet.

A régiók közötti magas rendelkezésre állás elérésének több általános megközelítése van:   

* Aktív/passzív készenléti kiszolgálóval. A forgalom az egyik régióra irányul, míg a másik készenléti kiszolgálón várakozik. Készenléti kiszolgáló esetében a másodlagos régió virtuális gépei folyamatosan le vannak foglalva és futnak.
* Aktív/passzív hidegtartalékkal. A forgalom az egyik régióra irányul, míg a másik hidegtartalékként áll készenlétben. Hidegtartalék esetében a másodlagos régió virtuális gépei nincsenek lefoglalva, amíg nem szükségessé nem válnak feladatátvétel céljából. Ezen megközelítés futtatása kevesebb költséggel jár, de a meghibásodáskor általában hosszabb időt vesz igénybe az üzembe állás.
* Aktív/aktív. Mindkét régió aktív, és a kérelmek terheléselosztással oszlanak meg közöttük. Ha egy régió elérhetetlenné válik, kikerül a rotációból. 

Ez az architektúra a készenléti kiszolgálóval konfigurált aktív/passzív üzemmódra összpontosít, a Traffic Managert használva a feladatátvételhez. Vegye figyelembe, hogy ha kis számú virtuális gépet helyez üzembe készenléti kiszolgálóként, igény szerint később is elvégezheti a horizontális felskálázásukat.


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

A következő [Azure CLI][install-azure-cli]-parancsot frissíti a prioritást:

```bat
azure network traffic-manager  endpoint set --resource-group <resource-group> --profile-name <profile>
    --name <traffic-manager-name> --type AzureEndpoints --priority 3
```    

Másik megoldásként ideiglenesen letilthatja a végpontot, amíg készen nem áll a feladat-visszavételre:

```bat
azure network traffic-manager  endpoint set --resource-group <resource-group> --profile-name <profile>
    --name <traffic-manager-name> --type AzureEndpoints --status Disabled
```    

A feladatátvétel okától függően előfordulhat, hogy újra üzembe kell helyeznie az erőforrásokat a régión belül. A feladat-visszavétel előtt tesztelje a működési készenlétet. A teszt többek között a következőket ellenőrzi:

* A virtuális gépek megfelelően vannak-e konfigurálva. (Az összes szükséges szoftver telepítve van, az IIS fut stb.)
* Az alkalmazás alrendszerei működőképesek-e.
* Funkciótesztelés. (Pl. az adatbázisszint elérhető a webes szintről.)

### <a name="cassandra-deployment-across-multiple-regions"></a>A Cassandra üzembe helyezése több régióban

A Cassandra-adatközpontok egymáshoz kapcsolódó adatcsomópontok csoportjai, amelyek együtt vannak konfigurálva egy fürtön belül a replikációhoz és a feladatok elkülönítéséhez.

Éles környezetben ajánlott a [DataStax Enterprise][datastax] használata. További információk a DataStax futtatásáról az Azure-ban: [A DataStax Enterprise üzembehelyezési útmutatója az Azure-hoz][cassandra-in-azure]. A következő általános javaslatok bármely Cassandra-kiadás esetében érvényesek: 

* Rendeljen nyilvános IP-címet minden egyes csomóponthoz. Ez lehetővé teszi, hogy a fürtök az Azure gerincinfrastruktúráját használva kommunikáljanak egymással a régiókon át, ezzel magas teljesítményt biztosítva alacsony költségek mellett.
* Védje a csomópontokat a megfelelő tűzfal- és hálózati biztonsági csoport (NSG-) konfigurációval, amely kizárólag az ismert forrásokból – például ügyfelektől és más csomópontfürtöktől – érkező, valamint az azok felé irányuló forgalmat engedi át. Vegye figyelembe, hogy a Cassandra eltérő portokat használ a kommunikációhoz, az OpsCenterhez, a Sparkhoz és egyebekhez. További információk a Cassandra porthasználatáról: [Tűzfalak porthozzáférésének konfigurálása][cassandra-ports].
* Használjon SSL-titkosítást az összes [ügyfél és csomópont][ssl-client-node], valamint [csomópont és csomópont közötti][ssl-node-node] kommunikációhoz.
* Egy adott régión belül kövesse a [Cassandra – javaslatok](n-tier.md#cassandra) című cikk útmutatását.

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

Komplex N szintű alkalmazás esetén előfordulhat, hogy nem kell replikálnia a teljes alkalmazást a másodlagos régióba. Ehelyett elég lehet replikálni egy kritikus fontosságú alrendszert, amely az üzletmenet folytonosságának fenntartásához szükséges.

A Traffic Manager a rendszer egyik lehetséges meghibásodási pontja. Ha a Traffic Manager szolgáltatás meghibásodik, az ügyfelek a leállás ideje alatt nem érhetik el az alkalmazást. Tekintse át a [Traffic Manager szolgáltatói szerződését][tm-sla], és döntse el, hogy a Traffic Manager egyedüli használata elegendő-e vállalata magas rendelkezésre állásra vonatkozó követelményeihez. Ha nem, akkor a biztonság érdekében érdemes lehet hozzáadni egy másik forgalomkezelési szolgáltatást a feladat-visszavételhez. Ha az Azure Traffic Manager szolgáltatás meghibásodik, módosítsa a CNAME rekordját a DNS-ben úgy, hogy a többi forgalomkezelő szolgáltatásra mutasson. (Ezt a lépést manuálisan kell elvégezni. Amíg a DNS-módosítások érvénybe nem lépnek, az alkalmazás nem lesz elérhető.)

A Cassandra-fürt esetében a megfontolandó feladatátvételi forgatókönyvek az alkalmazás által használt konzisztenciaszintektől és a használt replikák számától függenek. További információk a konzisztenciaszintekről és azok használatáról a Cassandrában: [Adatkonzisztencia konfigurálása][cassandra-consistency] és [Cassandra: Hány csomóponttal folyik kommunikáció a Quorum esetében?][cassandra-consistency-usage] Az adatok elérhetősége a Cassandrában az alkalmazás által használt konzisztenciaszinttől és a replikációs mechanizmustól függ. További információ a replikációról a Cassandrában: [Adatok replikációja a NoSQL-adatbázisokban – Ismertetés][cassandra-replication].

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
[cassandra-in-azure]: https://academy.datastax.com/resources/deployment-guide-azure
[cassandra-consistency]: http://docs.datastax.com/en/cassandra/2.0/cassandra/dml/dml_config_consistency_c.html
[cassandra-replication]: http://www.planetcassandra.org/data-replication-in-nosql-databases-explained/
[cassandra-consistency-usage]: https://medium.com/@foundev/cassandra-how-many-nodes-are-talked-to-with-quorum-also-should-i-use-it-98074e75d7d5#.b4pb4alb2
[cassandra-ports]: https://docs.datastax.com/en/datastax_enterprise/5.0/datastax_enterprise/sec/configFirewallPorts.html
[datastax]: https://www.datastax.com/products/datastax-enterprise
[health-endpoint-monitoring-pattern]: https://msdn.microsoft.com/library/dn589789.aspx
[install-azure-cli]: /azure/xplat-cli-install
[regional-pairs]: /azure/best-practices-availability-paired-regions
[resource groups]: /azure/azure-resource-manager/resource-group-overview
[resource-group-links]: /azure/resource-group-link-resources
[services-by-region]: https://azure.microsoft.com/regions/#services
[ssl-client-node]: http://docs.datastax.com/en/cassandra/2.0/cassandra/security/secureSSLClientToNode_t.html
[ssl-node-node]: http://docs.datastax.com/en/cassandra/2.0/cassandra/security/secureSSLNodeToNode_t.html
[tablediff]: https://msdn.microsoft.com/library/ms162843.aspx
[tm-configure-failover]: /azure/traffic-manager/traffic-manager-configure-failover-routing-method
[tm-monitoring]: /azure/traffic-manager/traffic-manager-monitoring
[tm-routing]: /azure/traffic-manager/traffic-manager-routing-methods
[tm-sla]: https://azure.microsoft.com/support/legal/sla/traffic-manager/v1_0/
[traffic-manager]: https://azure.microsoft.com/services/traffic-manager/
[visio-download]: https://archcenter.blob.core.windows.net/cdn/vm-reference-architectures.vsdx
[wsfc]: https://msdn.microsoft.com/library/hh270278.aspx
[0]: ./images/multi-region-application-diagram.png "Magas rendelkezésre állású hálózati architektúra N szintű Azure-alkalmazásokhoz"
