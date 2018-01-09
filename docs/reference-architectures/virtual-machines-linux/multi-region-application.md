---
title: "Linux virtuális gépek futtatása a több Azure-régiók a magas rendelkezésre állás"
description: "Ügyfélszoftverek központi telepítése a virtuális gépek magas rendelkezésre állást és a rugalmasság Azure több régióba."
author: MikeWasson
ms.date: 11/22/2016
pnp.series.title: Linux VM workloads
pnp.series.prev: n-tier
ms.openlocfilehash: 7d720a004d21edbffc0ddeba54e291aa817550e0
ms.sourcegitcommit: c9e6d8edb069b8c513de748ce8114c879bad5f49
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/08/2018
---
# <a name="run-linux-vms-in-multiple-regions-for-high-availability"></a>Linux virtuális gépek futtatása a magas rendelkezésre állású több régióba

A referencia-architektúrában futó N szintű alkalmazás több Azure-régiók, rendelkezésre állást és a hatékony vész-helyreállítási infrastruktúra elérése érdekében bevált gyakorlatok csoportja jeleníti meg. 

![[0]][0]

*Töltse le az architektúra [Visio-fájlját][visio-download].*

## <a name="architecture"></a>Architektúra 

Ez az architektúra látható egy épít [futtatása a Linux virtuális gépek N szintű alkalmazások](n-tier.md). 

* **Elsődleges és másodlagos régiók**. Két régió segítségével magasabb rendelkezésre állásának eléréséhez. Az elsődleges régióban egyik. A más régióban van, a feladatátvételre.
* **Az Azure DNS**. [Az Azure DNS] [ azure-dns] üzemeltetési szolgáltatás DNS-tartományok biztosítani a névfeloldást a Microsoft Azure-infrastruktúra használatával. Ha tartományait az Azure-ban üzemelteti, DNS-rekordjait a többi Azure-szolgáltatáshoz is használt hitelesítő adatokkal, API-kkal, eszközökkel és számlázási információkkal kezelheti.
* **Az Azure Traffic Manager**. [A TRAFFIC Manager] [ traffic-manager] irányítja a bejövő kéréseket a régiók egyikéhez sem. A normál működés során az elsődleges régióban irányítja a kérelmeket. Ha az adott régióban, a Traffic Manager feladatátadás a másodlagos régióba. További információkért tekintse meg a szakasz [Traffic Manager konfigurációs](#traffic-manager-configuration).
* **Erőforráscsoportok**. Hozzon létre külön [erőforráscsoportok] [ resource groups] az elsődleges régióban, a másodlagos régióba, és a Traffic Manager. Ez lehetővé teszi a rugalmasság, minden egyes régió szerinti egyetlen gyűjtemény-erőforrások kezelésére. Például egy régió tartozik, sikerült újratelepítése nélkül a másik egy le. [Az erőforráscsoportok hivatkozás][resource-group-links], így az alkalmazás az erőforrások listájának lekérdezés futtatása.
* **Vnetek**. Hozzon létre egy külön hálózatok mindegyik régióhoz. Ellenőrizze, hogy a címterek nem lehetnek átfedésben.
* **Apache Cassandra**. Adatközpontokban Cassandra telepíteni a magas rendelkezésre állású Azure-régiók között. Minden régióban a csomópontok a hibája állvány-kompatibilis módban van konfigurálva, és frissítési tartományt, a rugalmasságot belül a régió.

## <a name="recommendations"></a>Javaslatok

Egy több területi architektúra biztosít magas rendelkezésre állás érdekében, mint egyetlen régióban üzembe helyezni. Ha egy regionális kimaradás érinti az elsődleges régióban, [Traffic Manager] [ traffic-manager] hogy áthelyezze a feladatokat a másodlagos régióba. Ez az architektúra segíthet is, ha az alkalmazás egy egyedi alrendszer nem tud.

Magas rendelkezésre állás elérése érdekében a különböző régiókban néhány általános módszer van:   

* A melegtartalék aktív/passzív. Forgalom kerül egy régió tartozik, a más vár melegtartalék közben. Meleg készenléti azt jelenti, hogy a virtuális gépeket, a másodlagos régióban lefoglalt és fut mindig.
* A meleg készenléti állapotban lévő aktív/passzív. Forgalom kerül egy régió tartozik, a más vár meleg készenléti közben. Meleg készenléti állapot azt jelenti, hogy a virtuális gépeket, a másodlagos régióban nem foglal le addig, amíg a feladatátvételi szükséges. Ez a megközelítés költségek kisebb futtatásához, de általában adott meghibásodás során online állapotba hosszabb ideig tart.
* Aktív. Mindkét régió aktív, és kérés kerül közöttük. Ha nem érhető el egy régió tartozik, az kívül Elforgatás lesz végrehajtva. 

Ez az architektúra melegtartalék feladatátvételi Traffic Manager használata az aktív/passzív összpontosít. Vegye figyelembe, hogy sikerült központi telepítése egy kis számú virtuális gépek melegtartalék és majd horizontális felskálázás igény szerint.


### <a name="regional-pairing"></a>Regionális párosítás

Minden Azure-régió, egy másik ugyanazon a földrajzi régióját párosított. Régiók általában választhat a azonos regionális pár (például USA keleti régiója 2 és Velünk központi). Ennek során a következő előnyöket nyújtja:

* A széles körű kimaradás esetén legalább egy régió kívül minden pár helyreállítási prioritása van.
* Tervezett Azure rendszerfrissítések előkészítése már megkezdődött párosított régiók egymás után, az lehetséges állásidő minimalizálása érdekében.
* Párok találhatók belül az azonos földrajzi adatok rezidens követelményeinek megfelelően.

Győződjön meg arról, hogy mindkét régió támogatja az összes Azure-szolgáltatás szükséges az alkalmazás (lásd: [régiói][services-by-region]). Regionális párok kapcsolatos további információkért lásd: [üzleti folytonossági és vészhelyreállítási helyreállítási (BCDR): Azure-régiókat párosítva][regional-pairs].

### <a name="traffic-manager-configuration"></a>A TRAFFIC Manager-konfiguráció

A Traffic Manager konfigurálásakor, vegye figyelembe a következő szempontokat:

* **Útválasztás**. A TRAFFIC Manager támogatja több [útválasztási algoritmusok][tm-routing]. A jelen cikkben ismertetett esetben használjon *prioritás* útválasztási (korábbi nevén *feladatátvételi* útválasztási). Ezzel a beállítással a Traffic Manager minden kérést küld az elsődleges régióban, hacsak az elsődleges régióban el nem érhető el. Ezen a ponton akkor automatikusan átkerül a másodlagos régióba. Lásd: [konfigurálhatja feladatátvételi útválasztási módszer][tm-configure-failover].
* **Állapotmintáihoz**. HTTP (vagy HTTPS) használ a TRAFFIC Manager [mintavételi] [ tm-monitoring] minden egyes régió rendelkezésre állásának figyelésére. A mintavétel egy 200-as HTTP-választ a megadott URL-cím elérési keresi. Ajánlott eljárásként hozzon létre egy végpontot, amely jelent a alkalmazás általános állapotát, és használja ezt a végpontot a állapotmintáihoz. Ellenkező esetben a mintavétel előfordulhat, hogy jelentés a kifogástalan állapotú végpontok, ha az alkalmazás kritikus részei ténylegesen nem működnek. További információkért lásd: [állapotfigyelő végpont figyelési mintát][health-endpoint-monitoring-pattern].

Amikor a Traffic Manager átadja a feladatokat a rendszer egy adott időn belül, ha ügyfelek nem tudják elérni az alkalmazást. A duration hatással van a következő tényezőket:

* A állapotmintáihoz észlelni kell, hogy az elsődleges régióban vált nem érhető el.
* DNS-kiszolgálók frissítenie kell a gyorsítótárban lévő DNS-rekordok az IP-címhez, amely a DNS--élettartama (TTL) függ. Az alapértelmezett élettartam 300 másodpercen (5 percen), de ezt az értéket a Traffic Manager-profil létrehozásakor konfigurálhatja.

További információkért lásd: [kapcsolatos Traffic Manager figyelési][tm-monitoring].

Ha a Traffic Manager átadja, ajánlott végrehajtása manuális feladat-visszavétel, nem pedig egy automatikus feladat-visszavétel végrehajtására. Ellenkező esetben hozhat létre olyan helyzet, amelyben az alkalmazás oda-vissza régiók közötti tükrözi. Ellenőrizze, hogy minden alkalmazás alrendszer visszavétele előtt kifogástalan.

Vegye figyelembe, hogy a Traffic Manager vissza alapértelmezés szerint automatikusan nem. Ennek megelőzése érdekében manuálisan a Prioritás csökkentése az elsődleges régió egy feladatátvételi esemény után. Tegyük fel például, hogy az elsődleges régióban 1 prioritású virtuális gép, és a másodlagos prioritású 2. A feladatátvétel után állítsa be az elsődleges régióban prioritás 3, hogy megakadályozza az automatikus feladatátvételi. Ha készen áll vissza, frissítse a prioritás 1.

A következő [Azure CLI] [ install-azure-cli] parancs frissíti a prioritás:

```bat
azure network traffic-manager  endpoint set --resource-group <resource-group> --profile-name <profile>
    --name <traffic-manager-name> --type AzureEndpoints --priority 3
```    

Egy másik megoldás, hogy ideiglenesen letilthatja a végpont, amíg készen feladat-visszavételt:

```bat
azure network traffic-manager  endpoint set --resource-group <resource-group> --profile-name <profile>
    --name <traffic-manager-name> --type AzureEndpoints --status Disabled
```    

A feladatátvételi függően szükség lehet újratelepíteni az erőforrások régión belül. Mielőtt ismét sikertelen, egy működőképesség tesztjének elvégzéséhez. A vizsgálat ellenőrizni kell, többek között:

* Virtuális gépek helyesen vannak konfigurálva. (Az összes szükséges szoftverek telepítve van, az IIS fut, és így tovább.)
* Alkalmazás alrendszereket is megfelelő.
* Funkcionális tesztelése. (Például az adatbázis-rétegből érhető el a webes réteg alapján.)

### <a name="cassandra-deployment-across-multiple-regions"></a>Különféle régiókban Cassandra telepítése

Cassandra adatközpontok a replikáció és a munkaterhelések elkülönítése a fürtön belül együtt konfigurált kapcsolódó adatok csomópontok csoportja.

Ajánlott [alapszintű DataStax vállalati] [ datastax] éles használja. Az Azure-beli alapszintű DataStax további információkért lásd: [alapszintű DataStax vállalati üzembe helyezési útmutató az Azure][cassandra-in-azure]. Az alábbi általános javaslatokat bármely Cassandra edition vonatkoznak: 

* A nyilvános IP-cím hozzárendelése minden csomóponton. Ez lehetővé teszi a fürtök Azure gerincét infrastruktúrája, alacsony költséggel nagyobb teljesítményt nyújtó régiók kommunikálhassanak.
* Biztonságos-csomópontokat használja a megfelelő tűzfal és a hálózati biztonsági csoport (NSG) beállítások, így forgalmat csak ismert gazdagépek, köztük az ügyfelek és a többi fürtcsomóponton. Vegye figyelembe, hogy a Cassandra más portokat használ a kommunikációs, OpsCenter, Spark-, és így tovább. Cassandra portokkal, lásd: [tűzfalhozzáférés port konfigurálása][cassandra-ports].
* SSL-titkosítást használni az összes [ügyfél-csomópont] [ ssl-client-node] és [csomópontok] [ ssl-node-node] kommunikáció.
* A régión belül útmutatását [Cassandra javaslatok](n-tier.md#cassandra).

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

Egy összetett N szintű alkalmazást, nem kell replikálni a másodlagos régióban a teljes alkalmazás. Ehelyett csak lehet, hogy replikálja a kritikus alrendszer, az üzletmenet folytonossága támogatásához szükséges.

A TRAFFIC Manager ponttá lehetséges hiba a rendszerben. A Traffic Manager szolgáltatás nem sikerül, ha az ügyfelek nem tudnak hozzáférni az alkalmazás a leállások során. Tekintse át a [Traffic Manager SLA][tm-sla], és döntse el, hogy kizárólag a Traffic Manager segítségével megfelel-e az üzleti követelményeinek, a magas rendelkezésre állás érdekében. Ha nem, akkor vegyen fel egy másik forgalom felügyeleti megoldás, a feladat-visszavételre. Ha az Azure Traffic Manager szolgáltatás nem sikerül, módosítsa a CNAME rekordot a DNS-ben, hogy a többi felügyeleti szolgálat mutasson. (Ebben a lépésben kézzel kell elvégezni, és az alkalmazás nem lehet a DNS-módosítások továbbítódnak.)

Figyelembe kell venni a feladatátvételi forgatókönyvek Cassandra fürt használják az alkalmazást, valamint a használt replikák száma konzisztenciaszintek függ. Konzisztenciaszintek és Cassandra használata: [konfigurálása az adatok konzisztenciájának] [ cassandra-consistency] és [Cassandra: hány csomópontok vannak volt szó, a kvórummal?][cassandra-consistency-usage] Az alkalmazás és a replikációs eljárás által használt konzisztenciaszint Cassandra az adatok rendelkezésre állását határozza meg. Az Cassandra replikáció, lásd: [a NoSQL-adatbázisok alapján adatreplikáció][cassandra-replication].

## <a name="manageability-considerations"></a>Felügyeleti szempontok

Ha a telepítés frissítéséhez frissíteni egy régió tartozik csökkenti annak esélyét, a globális hibák egy helytelen konfiguráció vagy az alkalmazás hiba egyszerre.

Tesztelje a rugalmasság, a rendszer a hibák. Alább található néhány gyakori meghibásodási helyzet:

* Állítson le virtuálisgép-példányokat.
* Nyomás erőforrások, például CPU és memória.
* Hálózati kapcsolat bontása/késleltetés.
* Váltsa ki egyes folyamatok összeomlását.
* Alkalmazzon lejárt tanúsítványokat.
* Hardver hibák szimulálásához.
* Állítsa le a DNS-szolgáltatás a tartományvezérlőkön.

A helyreállítási idő mérését, és győződjön meg arról, hogy megfelelnek-e az üzleti igényeknek. Teszt sikertelen üzemmódot, valamint kombinációi.


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
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[wsfc]: https://msdn.microsoft.com/library/hh270278.aspx
[0]: ./images/multi-region-application-diagram.png "Magas rendelkezésre állású hálózati architektúra Azure N szintű alkalmazásokhoz"
