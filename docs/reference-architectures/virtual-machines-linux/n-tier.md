---
title: Linux rendszerű virtuális gépek futtatása N szintű alkalmazásokhoz az Azure-on
description: Annak az ismertetése, hogyan kell Linux rendszerű virtuális gépeket futtatni N szintű architektúrához a Microsoft Azure-ban.
author: MikeWasson
ms.date: 11/22/2017
pnp.series.title: Linux VM workloads
pnp.series.next: multi-region-application
pnp.series.prev: multi-vm
ms.openlocfilehash: 8d3e6e5124a0abb27a3c72e1ecbd52a1a1da2a33
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/06/2018
---
# <a name="run-linux-vms-for-an-n-tier-application"></a>Linux rendszerű virtuális gépek futtatása N szintű alkalmazáshoz

Ez a referenciaarchitektúra bevált eljárásokat mutat be arra az esetre, ha egy N szintű alkalmazáshoz szeretne Linux rendszerű virtuális gépeket (VM-eket) futtatni. [**A megoldás üzembe helyezése**.](#deploy-the-solution)  

![[0]][0]

*Töltse le az architektúra [Visio-fájlját][visio-download].*

## <a name="architecture"></a>Architektúra

Egy N szintű architektúra számos módon implementálható. Az ábrán egy tipikusnak mondható 3 szintű webalkalmazás látható. Ez az architektúra a következőre épül: [Elosztott terhelésű virtuális gépek futtatása a méretezhetőség és a rendelkezésre állás érdekében][multi-vm]. A webes és üzleti szintek elosztott terhelésű virtuális gépeket használnak.

* **Rendelkezésre állási csoportok.** Hozzon létre egy [rendelkezésre állási csoportot][azure-availability-sets] minden szinthez, majd minden szinten építsen ki legalább két virtuális gépet.  Így a virtuális gépek magasabb szintű [szolgáltatói szerződésre (SLA-ra)][vm-sla] jogosultak. Azt is megteheti, hogy egyetlen virtuális gépet helyez üzembe a rendelkezésre állási csoportban, de egy virtuális gép nem jogosult SLA-garanciára, kivéve, ha Azure Premium Storage-t használ minden operációsrendszer- és adatlemezhez.  
* **Alhálózatok.** Hozzon létre egy külön alhálózatot minden egyes szinthez. A címtartományt és az alhálózati maszkot a [CIDR] jelöléssel határozza meg. 
* **Terheléselosztók**. Használjon egy [internetkapcsolattal rendelkező terheléselosztót][load-balancer-external] a bejövő internetes forgalom webes szinten történő elosztására, és egy [belső terheléselosztót][load-balancer-internal] a webes szintről az üzleti szintre irányuló hálózati forgalom elosztására.
* **Azure DNS**. Az [Azure DNS][azure-dns] egy üzemeltetési szolgáltatás, amely a Microsoft Azure infrastruktúráját használja a DNS-tartományok névfeloldásához. Ha tartományait az Azure-ban üzemelteti, DNS-rekordjait a többi Azure-szolgáltatáshoz is használt hitelesítő adatokkal, API-kkal, eszközökkel és számlázási információkkal kezelheti.
* **Jumpbox.** Más néven [bástyagazdagép]. A hálózaton található biztonságos virtuális gép, amelyet a rendszergazdák a többi virtuális géphez való kapcsolódásra használnak. A jumpbox olyan NSG-vel rendelkezik, amely csak a biztonságos elemek listáján szereplő nyilvános IP-címekről érkező távoli forgalmat engedélyezi. Az NSG-nek lehetővé kell tennie a Secure Shell- (SSH-) forgalmat.
* **Monitorozás.** A figyelőszoftverek, mint a [Nagios], a [Zabbix] vagy az [Icinga], információval szolgálhatnak a válaszidőről, a virtuális gép hasznos üzemidejéről és a rendszer általános állapotáról. Olyan virtuális gépre telepítse a figyelőszoftvert, amely egy különálló felügyeleti alhálózaton található.
* <strong>NSG-k</strong>. Használjon [hálózati biztonsági csoportokat][nsg] (NSG-ket) a hálózati forgalom korlátozására a virtuális hálózaton belül. Az itt látható 3 szintű architektúrában például az adatbázisszint csak az üzleti szintről és a felügyeleti alhálózatról érkező forgalmat fogadja, a webes kezelőfelület felől érkező forgalmat nem.
* **Apache Cassandra-adatbázis**. Magas rendelkezésre állást biztosít az adatszinten a replikáció és a feladatátvétel engedélyezésével.

## <a name="recommendations"></a>Javaslatok

Az Ön követelményei eltérhetnek az itt leírt architektúrától. Ezeket a javaslatokat tekintse kiindulópontnak. 

### <a name="vnet--subnets"></a>Virtuális hálózat/alhálózatok

A virtuális hálózat létrehozásakor határozza meg az egyes alhálózatok erőforrásai által igényelt IP-címek számát. [CIDR] jelöléssel adjon meg egy alhálózati maszkot és egy virtuális hálózati címtartományt, amely elég nagy a szükséges IP-címekhez. Használjon a szabványos [magánhálózati IP-címblokkokba][private-ip-space] eső címterületet. Ezek az IP-címblokkok a következők: 10.0.0.0/8, 172.16.0.0/12 és 192.168.0.0/16.

Válasszon olyan címtartományt, amely nincs átfedésben a helyszíni hálózattal, hogy később átjárót lehessen beállítani a virtuális és a helyszíni hálózat között. A virtuális hálózat létrehozása után a címtartomány nem módosítható.

Az alhálózatokat a funkciók és a biztonsági követelmények szem előtt tartásával tervezze meg. Minden ugyanazon szinthez vagy szerepkörhöz tartozó virtuális gépnek egyazon alhálózaton kell lennie. Ez az alhálózat biztonsági korlátot is képezhet. A virtuális hálózatok és alhálózatok megtervezésével kapcsolatos további információkért lásd az [Azure-beli virtuális hálózatok tervezésével és kialakításával][plan-network] foglalkozó témakört.

Az egyes alhálózatokon adja meg az alhálózat címterületét CIDR-jelölés formájában. A „10.0.0.0/24” például egy 256 IP-címből álló tartományt hoz létre. A virtuális gépek ezek közül 251-et használhatnak; öt foglalt. Győződjön meg arról, hogy a címtartományok nem fedik át egymást a különböző alhálózatokon. Lásd a [virtuális hálózatra vonatkozó gyakori kérdéseket][vnet faq].

### <a name="network-security-groups"></a>Network security groups (Hálózati biztonsági csoportok)

A szintek közötti forgalmat NSG-szabályokkal korlátozhatja. A fenti 3 szintes architektúra esetén például a webes szint nem kommunikál közvetlenül az adatbázisszinttel. Ennek kényszerítése érdekében az adatbázisszintnek blokkolnia kell a webes szint alhálózatáról érkező bejövő forgalmat.  

1. Hozzon létre egy NSG-t, és társítsa az adatbázisszint alhálózatához.
2. Adjon meg egy szabályt, amely minden, a virtuális hálózatról bejövő forgalmat blokkol. (A szabályban használja a `VIRTUAL_NETWORK` címkét.) 
3. Adjon meg egy magasabb prioritású szabályt, amely engedélyezi az üzleti szint alhálózatáról bejövő forgalmat. Ez a szabály felülírja az előzőt, és lehetővé teszi az üzleti szint számára az adatbázisszinttel folytatott kommunikációt.
4. Adjon meg egy szabályt, amely engedélyezi az adatbázisszint alhálózatán belülről érkező forgalmat. Ez lehetővé teszi a virtuális gépek közötti kommunikációt az adatbázisszinten, amelyre az adatbázis replikációja és feladatátvétele miatt van szükség.
5. Adjon meg egy szabályt, amely engedélyezi a jumpbox alhálózatáról származó SSH-forgalmat. Ez lehetővé teszi, hogy a rendszergazdák csatlakozni tudjanak az adatbázisszinthez a jumpboxból.
   
   > [!NOTE]
   > A hálózati biztonsági csoportok olyan [alapértelmezett szabályokkal][nsg-rules] rendelkeznek, amelyek engedélyezik a virtuális hálózaton belülről érkező bejövő forgalmat. Ezek a szabályok nem törölhetők, de magasabb prioritású szabályok létrehozásával felülbírálhatók.
   > 
   > 

### <a name="load-balancers"></a>Terheléselosztók

A külső terheléselosztó osztja el az internetes forgalmat a webes szintre. Hozzon létre egy nyilvános IP-címet ehhez a terheléselosztóhoz. Lásd: [Internetkapcsolattal rendelkező terheléselosztó létrehozása][lb-external-create].

A belső terheléselosztó osztja el a webes szintről az üzleti szint felé irányuló hálózati forgalmat. Ahhoz, hogy ez a terheléselosztó magánhálózati IP-címre tehessen szert, hozzon létre egy előtérbeli IP-konfigurációt, és rendelje hozzá az üzleti szint alhálózatához. Lásd: [Belső terheléselosztó létrehozása][lb-internal-create].

### <a name="cassandra"></a>Cassandra

Éles környezetben ajánlott a [DataStax Enterprise][datastax] használata, de ezek a javaslatok bármely Cassandra-kiadásra is vonatkoznak. További információk a DataStax futtatásáról az Azure-ban: [A DataStax Enterprise üzembehelyezési útmutatója az Azure-hoz][cassandra-in-azure]. 

Helyezze a virtuális gépeket egy rendelkezésre állási csoport Cassandra-fürtjére annak érdekében, hogy a Cassandra-replikák több tartalék és frissítési tartományon legyenek elosztva. A tartalék és a frissítési tartományokról további információkért lásd a [virtuális gépek rendelkezésre állásának kezelésével][azure-availability-sets] foglalkozó témakört. 

Rendelkezésre állási csoportonként konfiguráljon három tartalék tartományt (ez a maximum) és 18 frissítési tartományt. Így a legmagasabb számú olyan frissítési tartománya lehet, amelyet a tartalék tartományokon még egyenletesen lehet elosztani.   

Konfigurálja a csomópontokat állvány-kompatibilis üzemmódban. Képezze le a tartalék tartományokat állványokra a `cassandra-rackdc.properties` fájlban.

A fürt előtt nincs szükség terheléselosztóra. Az ügyfél közvetlenül csatlakozik a fürt egyik csomópontjához.

### <a name="jumpbox"></a>Jumpbox

A jumpboxnak minimális teljesítménykövetelményei vannak, ezért kis virtuálisgép-méretet jelöljön ki a számára, például egy standard A1-et. 

Hozzon létre egy [nyilvános IP-címet] a jumpbox számára. Helyezze a jumpboxot a többi virtuális géppel megegyező virtuális hálózatba, de egy külön felügyeleti alhálózaton legyen.

Az alkalmazás számítási feladatait futtató virtuális gépek nyilvános internetről való SSH-hozzáférését ne engedélyezze. Az ilyen virtuális gépek SSH-hozzáférésének ehelyett a jumpboxon keresztül kell történnie. A rendszergazda először bejelentkezik a jumpboxba, majd azon keresztül bejelentkezik a többi virtuális gépbe. A jumpbox engedélyezi az internetről érkező SSH-forgalmat, de csak az ismert, biztonságos IP-címekről.

A jumpbox védelmére hozzon létre egy NSG-t, és alkalmazza azt a jumpbox alhálózatára. Adjon meg egy NSG-szabályt, amely csak a biztonságos nyilvános IP-címkészletekről engedélyezi az SSH-kapcsolatokat. Az NSG az alhálózathoz vagy a jumpbox hálózati adapteréhez is csatolható. Ebben az esetben ajánlott a hálózati adapterhez való csatlakoztatás, így az SSH-forgalom akkor is csak a jumpbox irányába van engedélyezve, ha más virtuális gépeket ad hozzá ugyanahhoz az alhálózathoz.

Állítsa be az NSG-t a többi alhálózathoz is úgy, hogy engedélyezze a felügyeleti alhálózatból érkező SSH-forgalmat.

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

Helyezzen minden szintet vagy virtuálisgép-szerepkört külön rendelkezésre állási csoportba. 

Hiába van több virtuális gépe, ez az adatbázisszinten nem jelent automatikusan magasabb rendelkezésre állást. Relációs adatbázis esetén rendszerint replikációval és feladatátvétellel biztosítható a magas rendelkezésre állás.  

Ha magasabb rendelkezésre állás szükséges, mint amelyet a [virtuális gépekhez készült Azure SLA nyújt][vm-sla], replikálja az alkalmazást két régióban, a feladatátvételre pedig használja az Azure Traffic Managert. További információkért lásd: [Linux virtuális gépek futtatása több régióban a magas rendelkezésre állás érdekében][multi-dc].  

## <a name="security-considerations"></a>Biztonsági szempontok

Érdemes lehet megadni egy hálózati virtuális berendezést (network virtual appliance, NVA), hogy DMZ-t lehessen létrehozni a nyilvános internet és az Azure-beli virtuális hálózat között. Az NVA egy általános kifejezés egy olyan virtuális berendezésre, amely hálózatokhoz kapcsolódó feladatokat lát el, például gondoskodik a tűzfalról, a csomagvizsgálatról, a naplózásról és az egyéni útválasztásról. További információkért lásd a [DMZ az Azure és az internet közötti implementálásával][dmz] foglalkozó témakört.

## <a name="scalability-considerations"></a>Méretezési szempontok

A terheléselosztók elosztják a hálózati forgalmat a webes és az üzleti szinteken. Új virtuálisgép-példányok hozzáadásával horizontálisan skálázhatja a rendszert. Vegye figyelembe, hogy egymástól függetlenül is méretezheti a webes és az üzleti szinteket, a terheléstől függően. Az ügyfélaffinitás fenntartásának szükségessége okozta lehetséges komplikációk csökkentése érdekében a webes szint virtuális gépeinek állapotmentesnek kell lennie, ahogy az üzleti logikát üzemeltető virtuális gépeknek is.

## <a name="manageability-considerations"></a>Felügyeleti szempontok

A teljes rendszer kezelését egyszerűbbé teheti különböző központosított felügyeleti eszközök használatával, amilyen például az [Azure Automation][azure-administration], a [Microsoft Operations Management Suite][operations-management-suite], a [Chef][chef] vagy a [Puppet][puppet]. Ezek az eszközök konszolidálják több virtuális gép rögzített diagnosztikai és az állapotinformációit, így átfogó képet nyújtanak a rendszerről.

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

Ennek a referenciaarchitektúrának egy üzemelő példánya elérhető a [GitHubon][github-folder]. 

### <a name="prerequisites"></a>Előfeltételek

Mielőtt üzembe helyezhetné saját előfizetésében a referenciaarchitektúrát, az alábbi lépéseket kell elvégeznie.

1. Klónozza, ágaztassa vagy a zip-fájl letöltése a [architektúrák hivatkozhat] [ ref-arch-repo] GitHub-tárházban.

2. Győződjön meg arról, hogy az Azure CLI 2.0 telepítve van a számítógépén. A CLI telepítéséhez kövesse az [Azure CLI 2.0 telepítése][azure-cli-2] című szakaszban leírt utasításokat.

3. Telepítse [az Azure építőelemei][azbb] npm-csomagot.

   ```bash
   npm install -g @mspnp/azure-building-blocks
   ```

4. Jelentkezzen be Azure-fiókjába egy parancssorból, Bash-parancssorból vagy PowerShell-parancssorból az alábbi parancsok egyikével, és kövesse az utasításokat.

   ```bash
   az login
   ```

### <a name="deploy-the-solution-using-azbb"></a>A megoldás üzembe helyezése az azbb használatával

Ha Linux rendszerű virtuális gépeket szeretne üzembe helyezni egy N szintű alkalmazás referenciaarchitektúrájához, kövesse az alábbi lépéseket:

1. Navigáljon a fenti előfeltételek 1. lépése során klónozott adattár `virtual-machines\n-tier-linux` mappájához.

2. A paraméterfájl egy alapértelmezett rendszergazdai felhasználónevet és jelszót határoz meg az üzemelő példány minden egyes virtuális gépéhez. Ezeket módosítania kell, mielőtt üzembe helyezné a referenciaarchitektúrát. Nyissa meg az `n-tier-linux.json` fájlt, és cserélje le az egyes **adminUsername** és **adminPassword** mezők tartalmát az új beállításokra.   Mentse a fájlt.

3. Helyezze üzembe a referenciaarchitektúrát az **azbb** parancssori eszköz használatával az alább látható módon.

   ```bash
   azbb -s <your subscription_id> -g <your resource_group_name> -l <azure region> -p n-tier-linux.json --deploy
   ```

A mintául szolgáló referenciaarchitektúra Azure-építőelemekkel történő üzembe helyezéséről további információkat a [GitHub-adattárban][git] talál.

<!-- links -->
[multi-dc]: multi-region-application.md
[dmz]: ../dmz/secure-vnet-dmz.md
[multi-vm]: ./multi-vm.md
[naming conventions]: /azure/guidance/guidance-naming-conventions
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azure-administration]: /azure/automation/automation-intro
[azure-availability-sets]: /azure/virtual-machines/virtual-machines-linux-manage-availability
[azure-cli-2]: https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest
[azure-dns]: /azure/dns/dns-overview
[bástyagazdagép]: https://en.wikipedia.org/wiki/Bastion_host
[cassandra-in-azure]: https://docs.datastax.com/en/datastax_enterprise/4.5/datastax_enterprise/install/installAzure.html
[cidr]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[chef]: https://www.chef.io/solutions/azure/
[datastax]: http://www.datastax.com/products/datastax-enterprise
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
[Zabbix]: http://www.zabbix.com/
[Icinga]: http://www.icinga.org/
[0]: ./images/n-tier-diagram.png "N szintű architektúra a Microsoft Azure használatával"

