---
title: Az Apache Cassandra használatával N szintű alkalmazás
description: Annak az ismertetése, hogyan kell Linux rendszerű virtuális gépeket futtatni N szintű architektúrához a Microsoft Azure-ban.
author: MikeWasson
ms.date: 09/13/2018
ms.openlocfilehash: 2eceb0b5d939c0aa2cc9fc3209d0f86449fdd72b
ms.sourcegitcommit: dbbf914757b03cdee7a274204f9579fa63d7eed2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/02/2018
ms.locfileid: "50916566"
---
# <a name="n-tier-application-with-apache-cassandra"></a>Az Apache Cassandra használatával N szintű alkalmazás

Ez a referenciaarchitektúra bemutatja, hogyan helyezhet üzembe a virtuális gépek és a egy N szintű alkalmazáshoz, az Apache Cassandra használatával Linux rendszeren az adatréteg számára konfigurált virtuális hálózatot. [**A megoldás üzembe helyezése**.](#deploy-the-solution) 

![[0]][0]

*Töltse le az architektúra [Visio-fájlját][visio-download].*

## <a name="architecture"></a>Architektúra 

Az architektúra a következő összetevőkből áll:

* **Erőforráscsoport.** Az [erőforráscsoportok][resource-manager-overview] az erőforrások csoportosítására használhatók, így élettartamuk, tulajdonosuk vagy egyéb jellemzőik alapján kezelhetők.

* **Virtuális hálózat (VNet) és alhálózatok.** Az Azure-ban minden virtuális gép egy alhálózatokra osztható virtuális hálózatban van üzembe helyezve. Hozzon létre egy külön alhálózatot minden egyes szinthez. 

* **NSG-k**. Használjon [hálózati biztonsági csoportokat][nsg] (NSG-ket) a hálózati forgalom korlátozására a virtuális hálózaton belül. Az itt látható 3 szintű architektúrában például az adatbázisszint csak az üzleti szintről és a felügyeleti alhálózatról érkező forgalmat fogadja, a webes kezelőfelület felől érkező forgalmat nem.

* **Virtuális gépek**. Javaslatok a virtuális gépek konfigurálása, lásd: [Windows virtuális gépek futtatása az Azure-beli](./windows-vm.md) és [Linux rendszerű virtuális gép futtatása az Azure-ban](./linux-vm.md).

* **Rendelkezésre állási csoportok.** Hozzon létre egy [rendelkezésre állási csoportot][azure-availability-sets] minden szinthez, majd minden szinten építsen ki legalább két virtuális gépet. Így a virtuális gépek magasabb szintű [szolgáltatói szerződésre (SLA-ra)][vm-sla] jogosultak. 

* **Virtuálisgép-méretezési csoportot** (nem látható). A [Virtuálisgép-méretezési csoportot] [ vmss] van egy rendelkezésre állási csoport használata helyett. A méretezési csoportok teszi egyszerűvé a horizontális felskálázás az egy szinten lévő virtuális gépek, vagy manuálisan vagy automatikusan előre meghatározott szabályok alapján.

* **Az Azure Load Balancer terheléselosztók.** A [terheléselosztók] [ load-balancer] beérkező internetes kérelmeket a Virtuálisgép-példányok elosztása. Használja a [nyilvános load balancer] [ load-balancer-external] terjeszteni a bejövő internetes forgalom webes szinten és a egy [belső load balancer] [ load-balancer-internal] , elosztja a hálózati forgalmat a webes szintről az üzleti szint.

* **Nyilvános IP-cím**. A nyilvános load balancer az internetes forgalmat fogadó nyilvános IP-cím szükséges.

* **Jumpbox.** Más néven [bástyagazdagép]. A hálózaton található biztonságos virtuális gép, amelyet a rendszergazdák a többi virtuális géphez való kapcsolódásra használnak. A jumpbox olyan NSG-vel rendelkezik, amely csak a biztonságos elemek listáján szereplő nyilvános IP-címekről érkező távoli forgalmat engedélyezi. Az NSG-t kell engedélyezniük ssh-forgalmat.

* **Apache Cassandra-adatbázis**. Magas rendelkezésre állást biztosít az adatszinten a replikáció és a feladatátvétel engedélyezésével.

* **Azure DNS**. Az [Azure DNS][azure-dns] egy üzemeltetési szolgáltatás, amely a Microsoft Azure infrastruktúráját használja a DNS-tartományok névfeloldásához. Ha tartományait az Azure-ban üzemelteti, DNS-rekordjait a többi Azure-szolgáltatáshoz is használt hitelesítő adatokkal, API-kkal, eszközökkel és számlázási információkkal kezelheti.

## <a name="recommendations"></a>Javaslatok

Az Ön követelményei eltérhetnek az itt leírt architektúrától. Ezeket a javaslatokat tekintse kiindulópontnak. 

### <a name="vnet--subnets"></a>Virtuális hálózat/alhálózatok

A virtuális hálózat létrehozásakor határozza meg az egyes alhálózatok erőforrásai által igényelt IP-címek számát. [CIDR] jelöléssel adjon meg egy alhálózati maszkot és egy virtuális hálózati címtartományt, amely elég nagy a szükséges IP-címekhez. Használjon a szabványos [magánhálózati IP-címblokkokba][private-ip-space] eső címterületet. Ezek az IP-címblokkok a következők: 10.0.0.0/8, 172.16.0.0/12 és 192.168.0.0/16.

Válasszon olyan címtartományt, amely nincs átfedésben a helyszíni hálózattal, arra az esetre, ha később átjárót kell beállítania a virtuális hálózat és a helyszíni hálózat között. A virtuális hálózat létrehozása után a címtartomány nem módosítható.

Az alhálózatokat a funkciók és a biztonsági követelmények szem előtt tartásával tervezze meg. Minden ugyanazon szinthez vagy szerepkörhöz tartozó virtuális gépnek egyazon alhálózaton kell lennie. Ez az alhálózat biztonsági korlátot is képezhet. A virtuális hálózatok és alhálózatok megtervezésével kapcsolatos további információkért lásd az [Azure-beli virtuális hálózatok tervezésével és kialakításával][plan-network] foglalkozó témakört.

### <a name="load-balancers"></a>Terheléselosztók

Ne engedélyezze a virtuális gépek elérését közvetlenül az internetről – ehelyett adjon mindegyiknek privát IP-címet. Az IP-cím a nyilvános load Balancer használatával az ügyfelek csatlakoznak.

Adja meg a terheléselosztó a virtuális gépek felé irányuló közvetlen hálózati forgalomra vonatkozó szabályait. Például a HTTP-forgalom engedélyezéséhez hozzon létre egy szabályt, amely hozzárendeli a 80-as portot az előtérbeli konfigurációból a háttércímkészletben található 80-as porthoz. Amikor egy ügyfél HTTP-kérelmet küld a 80-as port felé, a terheléselosztó kiválaszt egy háttérbeli IP-címet egy [kivonatoló algoritmus][load-balancer-hashing] használatával, amely tartalmazza a forrás IP-címét. Így a kérelmek megoszlanak az összes virtuális gép között.

### <a name="network-security-groups"></a>Network security groups (Hálózati biztonsági csoportok)

A szintek közötti forgalmat NSG-szabályokkal korlátozhatja. A fenti 3 szintes architektúra esetén például a webes szint nem kommunikál közvetlenül az adatbázisszinttel. Ennek kényszerítése érdekében az adatbázisszintnek blokkolnia kell a webes szint alhálózatáról érkező bejövő forgalmat.  

1. Megtagadási az összes bejövő forgalmat a virtuális hálózatról. (A szabályban használja a `VIRTUAL_NETWORK` címkét.) 
2. Az üzleti szint alhálózatáról érkező forgalom engedélyezéséhez.  
3. Az adatbázisszint alhálózatán érkező forgalom engedélyezéséhez. Ez a szabály lehetővé teszi, hogy szükség van az adatbázis-replikáció és feladatátvételi adatbázis-beli virtuális gépek közötti kommunikációt.
4. Forgalom ssh (22-es port) a jumpbox alhálózatáról származó. Ez lehetővé teszi, hogy a rendszergazdák csatlakozni tudjanak az adatbázisszinthez a jumpboxból.

2-szabályok létrehozása &ndash; magasabb prioritású, mint az első szabály, így azt felülbírálják a 4.

### <a name="cassandra"></a>Cassandra

Éles környezetben ajánlott a [DataStax Enterprise][datastax] használata, de ezek a javaslatok bármely Cassandra-kiadásra is vonatkoznak. További információk a DataStax futtatásáról az Azure-ban: [A DataStax Enterprise üzembehelyezési útmutatója az Azure-hoz][cassandra-in-azure]. 

Helyezze a virtuális gépeket egy rendelkezésre állási csoport Cassandra-fürtjére annak érdekében, hogy a Cassandra-replikák több tartalék és frissítési tartományon legyenek elosztva. A tartalék és a frissítési tartományokról további információkért lásd a [virtuális gépek rendelkezésre állásának kezelésével][azure-availability-sets] foglalkozó témakört. 

Rendelkezésre állási csoportonként konfiguráljon három tartalék tartományt (ez a maximum) és 18 frissítési tartományt. Így a legmagasabb számú olyan frissítési tartománya lehet, amelyet a tartalék tartományokon még egyenletesen lehet elosztani.   

Konfigurálja a csomópontokat állvány-kompatibilis üzemmódban. Képezze le a tartalék tartományokat állványokra a `cassandra-rackdc.properties` fájlban.

A fürt előtt nincs szükség terheléselosztóra. Az ügyfél közvetlenül csatlakozik a fürt egyik csomópontjához.

A magas rendelkezésre állás érdekében a több Azure-régióban a Cassandra üzembe helyezése. A csomópontok minden régióban állvány-kompatibilis üzemmódban vannak konfigurálva tartalék és frissítési tartományokkal a régión belüli rugalmasság érdekében.


### <a name="jumpbox"></a>Jumpbox

Ssh hozzáférés a nyilvános internetről való engedélyezze az alkalmazás számítási feladatait futtató virtuális gépek. Ehelyett minden ssh hozzáférési ezeken a virtuális gépeken a jumpboxon keresztül kell származnia. A rendszergazda először bejelentkezik a jumpboxba, majd azon keresztül bejelentkezik a többi virtuális gépbe. A jumpbox engedélyezi ssh-forgalmat az internetről, de csak az ismert, biztonságos IP-címeket.

A jumpbox rendelkezik a minimális teljesítménykövetelményei, ezért kattintson egy kisméretű Virtuálisgép-méretet. Hozzon létre egy [nyilvános IP-címet] a jumpbox számára. Helyezze a jumpboxot a többi virtuális géppel megegyező virtuális hálózatba, de egy külön felügyeleti alhálózaton legyen.

A jumpbox védelmére, adja hozzá egy NSG-szabályt, amely engedélyezi az ssh-kapcsolatokat csak a nyilvános IP-címkészletekről. Konfigurálja az ssh forgalom engedélyezésére a felügyeleti alhálózatról a többi alhálózathoz NSG-t.

## <a name="scalability-considerations"></a>Méretezési szempontok

[Virtuálisgép-méretezési csoportok] [ vmss] üzembe helyezése és kezelése, amelyek azonos virtuális gépek segítségével. A méretezési csoportok támogatják a teljesítménymetrikák alapján történő automatikus skálázást. Ahogy a terhelés növekszik a virtuális gépeken, a rendszer további virtuális gépeket ad a terheléselosztóhoz. Fontolja meg a méretezési csoportok használatát, ha virtuális gépek gyors horizontális felskálázására vagy automatikus méretezésre van szüksége.

A méretezési csoportokban üzembe helyezett virtuális gépek konfigurálásának két alapvető módja van:

- Bővítmények segítségével konfigurálhatja a virtuális gépet annak kiépítése után. Ezzel a módszerrel az új virtuálisgép-példányok indítása több időt vehet igénybe, mint a bővítmények nélküli virtuális gépeké.

- Üzembe helyezhet egy [felügyelt lemezt](/azure/storage/storage-managed-disks-overview) egy egyéni rendszerképpel. Ez a módszer gyorsabban kivitelezhető. Azonban ebben az esetben naprakészen kel tartania a rendszerképet.

További szempontokért lásd: [Kialakítási szempontok a méretezési csoportokhoz][vmss-design].

> [!TIP]
> Bármilyen automatikus méretezési megoldás használata esetén jóval előre tesztelje azt az éles környezetre jellemző mennyiségű számítási feladattal.

Minden Azure-előfizetésre alapértelmezett korlátozások vonatkoznak. Ilyen például a régiónként alkalmazható virtuális gépek maximális száma. Támogatási kérelem kitöltésével megnövelheti ezt a korlátot. További információk: [Az Azure-előfizetések és -szolgáltatások korlátozásai, kvótái és megkötései][subscription-limits].

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

Ha nem használ a Virtuálisgép-méretezési csoportok virtuális gépek rendelkezésre állási csoport azonos szinten üzembe. Hozzon létre legalább két virtuális gépet a rendelkezésre állási csoportban az [Azure-beli virtuális gépekre vonatkozó rendelkezésre állási SLA][vm-sla] támogatásához. További információk: [Virtuális gépek rendelkezésre állásának kezelése][availability-set]. 

A terheléselosztó [állapotmintákat][health-probes] használ a virtuálisgép-példányok rendelkezésre állásának monitorozásához. Ha a mintavétel nem ér el egy példányt egy bizonyos időkorláton belül, a terheléselosztó nem irányít több forgalmat az adott virtuális gép felé. A terheléselosztó ezután is folytatja a mintavételezést, és amint a virtuális gép újra elérhetővé válik, a terheléselosztó ismét elkezd forgalmat irányítani felé.

Az alábbiakban néhány javaslat olvasható a terheléselosztó állapot-mintavételére vonatkozóan:

* A mintavételek a HTTP-t vagy a TCP-t tesztelhetik. Ha a virtuális gépek HTTP-kiszolgálót futtatnak, HTTP-mintavételt hozzon létre. Egyéb esetben TCP-mintavételt hozzon létre.
* A HTTP-mintavétel esetében adjon meg elérési utat egy HTTP-végponthoz. A mintavétel 200-as HTTP-választ vár erről a megadott elérési útról. Ez lehet a gyökérútvonal („/”) vagy egy állapotmonitorozó végpont, amely valamilyen egyéni logikát alkalmaz az alkalmazás állapotának ellenőrzésére. A végpontnak engedélyeznie kell a névtelen HTTP-kérelmeket.
* A mintavétel [ismert IP-címről érkezik:][health-probe-ip] 168.63.129.16. Győződjön meg arról, hogy nem tiltja a bejövő és kimenő forgalmat erről az IP-címről valamelyik tűzfalszabályzatban vagy NSG-szabályban.
* Az állapotminták állapotának megtekintéséhez használjon [állapotminta-naplókat][health-probe-log]. Engedélyezze a naplózást az Azure Portalon minden egyes terheléselosztóra vonatkozóan. A naplókat a rendszer az Azure Blob Storage-ba írja. A naplók megmutatják, hány háttérbeli virtuális gép nem fogad hálózati forgalmat a meghiúsult mintavételi válaszok miatt.

A Cassandra-fürt esetében a megfontolandó feladatátvételi forgatókönyvek az alkalmazás által használt konzisztenciaszintektől és a használt replikák számától függenek. További információk a konzisztenciaszintekről és azok használatáról a Cassandrában: [Adatkonzisztencia konfigurálása][cassandra-consistency] és [Cassandra: Hány csomóponttal folyik kommunikáció a Quorum esetében?][cassandra-consistency-usage] Az adatok elérhetősége a Cassandrában az alkalmazás által használt konzisztenciaszinttől és a replikációs mechanizmustól függ. További információ a replikációról a Cassandrában: [Adatok replikációja a NoSQL-adatbázisokban – Ismertetés][cassandra-replication].

## <a name="security-considerations"></a>Biztonsági szempontok

A virtuális hálózatok forgalomelkülönítési határok az Azure-ban. Egy adott virtuális hálózatban található virtuális gépek nem képesek közvetlenül kommunikálni egy másik virtuális hálózat gépeivel. Az azonos virtuális hálózatban elhelyezkedő virtuális gépek képesek a kommunikációra, hacsak [hálózati biztonsági csoportokat] [ nsg] (NSG-ket) nem hoz létre a forgalom korlátozására. További információ: [A Microsoft felhőszolgáltatásai és hálózati biztonság][network-security].

A bejövő internetes forgalom esetében a terheléselosztó szabályai határozzák meg, milyen forgalom érheti el a háttérrendszert. A terheléselosztó szabályai azonban nem támogatják a biztonságos IP-címek listázását, ezért ha fel kíván venni bizonyos nyilvános IP-címeket a biztonságos címek listájára, adjon hálózati biztonsági csoportot az alhálózathoz.

Érdemes lehet hozzáadnia egy hálózati virtuális berendezést (network virtual appliance, NVA), hogy DMZ-t lehessen létrehozni az internet és az Azure-beli virtuális hálózat között. Az NVA egy általános kifejezés egy olyan virtuális berendezésre, amely hálózatokhoz kapcsolódó feladatokat lát el, például gondoskodik a tűzfalról, a csomagvizsgálatról, a naplózásról és az egyéni útválasztásról. További információkért lásd a [DMZ az Azure és az internet közötti implementálásával][dmz] foglalkozó témakört.

Titkosíthatja az inaktív bizalmas adatokat, és az [Azure Key Vaulttal][azure-key-vault] kezelheti az adatbázis titkosítási kulcsait. A Key Vault képes a hardveres biztonsági modulok (HSM-ek) titkosítási kulcsainak tárolására. Emellett ajánlott alkalmazások titkos adatait, például az adatbázis kapcsolati karakterláncainak tárolása a Key Vaultban.

Ajánlott engedélyezni az [DDoS Protection Standard](/azure/virtual-network/ddos-protection-overview), amely biztosítja, hogy egy virtuális hálózatban található erőforrások további DDoS-kockázatcsökkentést. Alapszintű DDoS elleni védelem részeként az Azure platform automatikusan engedélyezve van, noha a DDoS Protection Standard kockázatcsökkentési képességeket biztosít, amelyek kifejezetten az Azure virtuális hálózati erőforrások, amelyek ideálisak.  

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

Ennek a referenciaarchitektúrának egy üzemelő példánya elérhető a [GitHubon][github-folder]. 

### <a name="prerequisites"></a>Előfeltételek

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-the-solution-using-azbb"></a>A megoldás üzembe helyezése az azbb használatával

Ha Linux rendszerű virtuális gépeket szeretne üzembe helyezni egy N szintű alkalmazás referenciaarchitektúrájához, kövesse az alábbi lépéseket:

1. Navigáljon a fenti előfeltételek 1. lépése során klónozott adattár `virtual-machines\n-tier-linux` mappájához.

2. A paraméterfájl egy alapértelmezett rendszergazdai felhasználónevet és jelszót határozza a központi telepítésben lévő mindegyik virtuális gép. Ezeket módosítania kell, mielőtt üzembe helyezné a referenciaarchitektúrát. Nyissa meg az `n-tier-linux.json` fájlt, és cserélje le az egyes **adminUsername** és **adminPassword** mezők tartalmát az új beállításokra.   Mentse a fájlt.

3. Helyezze üzembe a referenciaarchitektúrát az **azbb** parancssori eszköz használatával az alább látható módon.

   ```bash
   azbb -s <your subscription_id> -g <your resource_group_name> -l <azure region> -p n-tier-linux.json --deploy
   ```

A mintául szolgáló referenciaarchitektúra Azure-építőelemekkel történő üzembe helyezéséről további információkat a [GitHub-adattárban][git] talál.

<!-- links -->
[dmz]: ../dmz/secure-vnet-dmz.md
[multi-vm]: ./multi-vm.md
[naming conventions]: /azure/guidance/guidance-naming-conventions
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azure-administration]: /azure/automation/automation-intro
[azure-availability-sets]: /azure/virtual-machines/virtual-machines-linux-manage-availability
[azure-cli-2]: https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest
[azure-dns]: /azure/dns/dns-overview
[azure-key-vault]: https://azure.microsoft.com/services/key-vault

[bástyagazdagép]: https://en.wikipedia.org/wiki/Bastion_host
[cassandra-in-azure]: https://academy.datastax.com/resources/deployment-guide-azure
[cassandra-consistency]: https://docs.datastax.com/en/cassandra/2.0/cassandra/dml/dml_config_consistency_c.html
[cassandra-replication]: https://academy.datastax.com/planet-cassandra/data-replication-in-nosql-databases-explained
[cassandra-consistency-usage]: https://medium.com/@foundev/cassandra-how-many-nodes-are-talked-to-with-quorum-also-should-i-use-it-98074e75d7d5#.b4pb4alb2

[cidr]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[chef]: https://www.chef.io/solutions/azure/
[datastax]: https://www.datastax.com/products/datastax-enterprise
[git]: https://github.com/mspnp/template-building-blocks
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/n-tier-linux
[lb-external-create]: /azure/load-balancer/load-balancer-get-started-internet-portal
[lb-internal-create]: /azure/load-balancer/load-balancer-get-started-ilb-arm-portal
[load-balancer-external]: /azure/load-balancer/load-balancer-internet-overview
[load-balancer-internal]: /azure/load-balancer/load-balancer-internal-overview
[nsg]: /azure/virtual-network/virtual-networks-nsg
[nsg-rules]: /azure/azure-resource-manager/best-practices-resource-manager-security#network-security-groups
[operations-management-suite]: https://www.microsoft.com/server-cloud/operations-management-suite/overview.aspx
[plan-network]: /azure/virtual-network/virtual-network-vnet-plan-design-arm
[private-ip-space]: https://en.wikipedia.org/wiki/Private_network#Private_IPv4_address_spaces
[nyilvános IP-címet]: /azure/virtual-network/virtual-network-ip-addresses-overview-arm
[puppet]: https://puppetlabs.com/blog/managing-azure-virtual-machines-puppet
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[vnet faq]: /azure/virtual-network/virtual-networks-faq
[visio-download]: https://archcenter.blob.core.windows.net/cdn/vm-reference-architectures.vsdx
[Nagios]: https://www.nagios.org/
[Zabbix]: https://www.zabbix.com/
[Icinga]: https://www.icinga.org/
[0]: ./images/n-tier-cassandra.png "N szintű architektúra a Microsoft Azure használatával"

[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview 
[vmss]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
[load-balancer]: /azure/load-balancer/load-balancer-get-started-internet-arm-cli
[load-balancer-hashing]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[vmss-design]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-design-overview
[subscription-limits]: /azure/azure-subscription-service-limits
[availability-set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[health-probes]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[health-probe-log]: /azure/load-balancer/load-balancer-monitor-log
[health-probe-ip]: /azure/virtual-network/virtual-networks-nsg#special-rules
[network-security]: /azure/best-practices-network-security
