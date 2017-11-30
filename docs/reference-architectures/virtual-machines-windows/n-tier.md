---
title: "Futó Windows virtuális gépeinek N szintű architektúrája"
description: "How Azure, a rendelkezésre állási, biztonsági, méretezhetőség és kezelhetőség biztonsági kifizető különös figyelmet egy többrétegű architektúra végrehajtásához."
author: MikeWasson
ms.date: 11/22/2016
pnp.series.title: Windows VM workloads
pnp.series.next: multi-region-application
pnp.series.prev: multi-vm
ms.openlocfilehash: e25d10d661ac4759f209bd27384303dee2ee454e
ms.sourcegitcommit: 583e54a1047daa708a9b812caafb646af4d7607b
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/28/2017
---
# <a name="run-windows-vms-for-an-n-tier-application"></a>Futtassa a Windows virtuális gépek N szintű alkalmazás

A referencia-architektúrában futó Windows virtuális gépeket (VM) N szintű alkalmazások bevált gyakorlatok csoportja jeleníti meg. [**A megoldás üzembe helyezése**.](#deploy-the-solution) 

![[0]][0]

*Töltse le az architektúra [Visio-fájlját][visio-download].*

## <a name="architecture"></a>Architektúra 

Számos módon valósítja meg az N szintű architektúrát. Az ábrán egy tipikus 3 szintű webalkalmazást. Ez az architektúra épül [futtassa a virtuális gépek elosztott terhelésű a méretezhetőség és a rendelkezésre állási][multi-vm]. A web és business szintek használata virtuális gépek elosztott terhelésű.

* **Rendelkezésre állási készletek.** Hozzon létre egy [rendelkezésre állási csoport] [ azure-availability-sets] minden egyes réteg, és minden egyes rétegben legalább két virtuális gépek telepítéséhez. Ez lehetővé teszi a virtuális gépeket abban az esetben jogosult a magasabb [szolgáltatásiszint-szerződés (SLA) szolgáltatás] [ vm-sla] virtuális gépekhez. Telepíthet egy virtuális rendelkezésre állási csoportba, de a virtuális rendszer nem felel meg egy (SLA) garantált csak egyetlen virtuális gép prémium szintű Azure Storage használ az operációs rendszer és az összes lemez.  
* **Alhálózatok.** Hozzon létre egy külön alhálózatot az egyes rétegekhez. Adja meg a címtartományt és alhálózati maszk használatával [CIDR] jelöléssel. 
* **Terheléselosztók**. Használjon egy [internetre irányuló terheléselosztót] [ load-balancer-external] terjeszteni a webes réteg bejövő internetes forgalmat és egy [belső terheléselosztó] [ load-balancer-internal]hálózati forgalmat a a webes réteg az üzleti szint és terjeszteni.
* **Jumpbox.** Más néven a [megerősített állomás]. Biztonságos virtuális gép, amely a rendszergazdák csatlakozhat a virtuális gépeket a hálózaton. A jumpbox olyan NSG-vel rendelkezik, amely csak a biztonságos elemek listáján szereplő nyilvános IP-címekről érkező távoli forgalmat engedélyezi. Az NSG-nek engedélyeznie kell a távoli asztali (RDP) forgalmat.
* **Figyelés.** Például a figyelés szoftver [Nagios], [Zabbix], vagy [Icinga] biztosíthat, válaszidő, illetve a virtuális gép hasznos üzemidő és a rendszer általános állapotát. Telepíti a figyelési, amely egy különálló felügyeleti alhálózat el van helyezve.
* **Az NSG-k.** Használjon [hálózati biztonsági csoportok] [ nsg] (NSG-ket) korlátozzák a hálózati forgalmat a Vneten belül. Például a 3-rétegű architektúra itt látható, az adatbázis-rétegből nem fogadja el a előtér, csak az üzleti szint és a felügyeleti alhálózati a forgalmat.
* **SQL Server Always On rendelkezésre állási csoporthoz.** Itt az adatréteg: magas rendelkezésre állású replikációs és feladatátvételi engedélyezésével.
* **Active Directory tartományi szolgáltatások (AD DS) kiszolgálók**. Előtt a Windows Server 2016 SQL Server Always On rendelkezésre állási csoportok egy tartományhoz kell csatlakoztatni. Ennek az az oka a rendelkezésre állási csoportok Windows Server feladatátvételi fürt (WSFC) technológia függ. Windows Server 2016 vezet be Active Directory nélküli feladatátvevő fürt létrehozásának ebben az esetben az Active Directory tartományi szolgáltatások-kiszolgálók esetén nincs szükség ebbe az architektúrába. További információkért lásd: [a Windows Server 2016 feladatátvételi fürtszolgáltatásával kapcsolatos újdonságok][wsfc-whats-new].

## <a name="recommendations"></a>Javaslatok

Az Ön követelményei eltérhetnek az itt leírt architektúrától. Ezeket a javaslatokat tekintse kiindulópontnak. 

### <a name="vnet--subnets"></a>Virtuális hálózatot / alhálózatot

A virtuális hálózat létrehozásakor határozza meg a minden alhálózatban kell hány IP-címeket. Adjon meg alhálózati maszkot és egy virtuális hálózat címtartománya elég nagy a szükséges IP-címek, amely az [CIDR] jelöléssel. Amely a szabványos címterület használata [privát IP-címblokkok][private-ip-space], amelyeket 10.0.0.0/8, 172.16.0.0/12 vagy 192.168.0.0/16.

Válassza ki, amely nem fedi át a helyszíni hálózat címtartományt, abban az esetben, akkor be kell állítania egy átjárót a virtuális hálózat és a helyszíni hálózat között később. Miután létrehozta a virtuális hálózat, a címtartomány nem módosítható.

Tervezze meg működésének és biztonsági követelmények figyelembe vételével rendelkező alhálózatokhoz. Minden virtuális gép ugyanabban a réteg vagy szerepkör kell kísérhet ugyanazon az alhálózaton, amelyek lehetnek biztonsági korlátot. Virtuális hálózatokat és alhálózatokat megtervezésével kapcsolatos további információkért lásd: [tervezési és kialakítási Azure virtuális hálózatok][plan-network].

Az egyes alhálózatokon adja meg a címtér az alhálózat CIDR-formátumban. Például a "10.0.0.0/24" 256 IP-címek egy tartománya hoz létre. Virtuális gépek használhatják 251 ezek; öt vannak fenntartva. Győződjön meg arról, hogy a cím címtartományok nem fednek át alhálózatokon keresztül. Tekintse meg a [virtuális hálózat – gyakori kérdések][vnet faq].

### <a name="network-security-groups"></a>Network security groups (Hálózati biztonsági csoportok)

NSG-szabályok segítségével korlátozzák a rétegek közötti forgalmat. Például a fenti 3-rétegű architektúra esetén a webes réteg nem közvetlenül kommunikálnak az adatbázis-rétegből. Az adatbázis-rétegből érvényesítik, a webes réteg alhálózati érkező bejövő forgalmat kell blokkolja.  

1. Hozzon létre egy NSG-t, és társítsa azt az adatbázis réteg alhálózathoz.
2. Adjon hozzá egy szabályt, amely minden bejövő forgalom a vnet megtagadja. (Használja a `VIRTUAL_NETWORK` címke a szabályban.) 
3. A magasabb prioritású, amely lehetővé teszi a bejövő forgalom az üzleti szint alhálózatból rendelkező szabály hozzáadását. Ez a szabály felülírja az előző szabályt, és lehetővé teszi, hogy felvegye a az adatbázis-rétegből az üzleti szint.
4. Adjon hozzá egy szabályt, amely lehetővé teszi, hogy a bejövő forgalom maga az adatbázis réteg alhálózat belül. Ez a szabály lehetővé teszi, hogy az adatbázis-rétegből, szükség van az adatbázis-replikációt és feladatátvételt a virtuális gépek közötti kommunikáció.
5. Adjon hozzá egy szabályt, amely a jumpbox alhálózatból RDP-forgalmát engedélyezi. Ez a szabály lehetővé teszi, hogy a rendszergazdák a jumpbox az adatbázis-rétegből csatlakoztassa.
   
   > [!NOTE]
   > Az NSG tartozik alapértelmezett szabályokat, amelyek lehetővé teszik a Vneten belül minden bejövő forgalom. Ezek a szabályok nem lehet törölni, de magasabb prioritású szabályok létrehozásával felülbírálhatja.
   > 
   > 

### <a name="load-balancers"></a>Terheléselosztók

A külső terheléselosztó osztja el a webes réteg internetes forgalmat. Hozzon létre egy nyilvános IP-címet ehhez a terheléselosztóhoz. Lásd: [egy internetre irányuló Terheléselosztói][lb-external-create].

A belső terheléselosztó osztja el a hálózati forgalmat a a webes réteg és az üzleti szint. Adjon meg egy magánhálózati IP-címet ehhez a terheléselosztóhoz, hozzon létre egy előtérbeli IP-konfigurációja, és rendelje hozzá azt az üzleti szint alhálózatát. Lásd: [létrehozásához a belső terheléselosztók][lb-internal-create].

### <a name="sql-server-always-on-availability-groups"></a>SQL Server Always On rendelkezésre állási csoportok

Ajánlott [Always On rendelkezésre állási csoportok] [ sql-alwayson] SQL Server magas rendelkezésre állásra. Windows Server 2016-ot, mielőtt az Always On rendelkezésre állási csoportok igényli tartományvezérlő, és a rendelkezésre állási csoport összes csomópontjának AD ugyanabban a tartományban kell lennie.

Más szintekre kapcsolódni az adatbázishoz, keresztül egy [rendelkezésre állási csoport figyelőjének][sql-alwayson-listeners]. A figyelő lehetővé teszi, hogy a fizikai SQL Server-példány nevét ismerete nélkül kapcsolódni az SQL-ügyfeleken. A virtuális gépeket, hogy az adatbázis hozzáférés csatlakoznia kell a tartományhoz. Az ügyfél (ebben az esetben egy másik réteg) DNS-Kiszolgálókat használ, a figyelő virtuálishálózat-név feloldani az IP-címeket.

A SQL Server mindig a rendelkezésre állási csoport konfigurálásához az alábbiak szerint:

1. Hozzon létre egy Windows Server feladatátvételi fürtszolgáltatási (WSFC) fürt, egy SQL Server mindig a rendelkezésre állási csoport és egy elsődleges másodpéldány. További információkért lásd: [Ismerkedés az Always On rendelkezésre állási csoportok][sql-alwayson-getting-started]. 
2. Hozzon létre egy belső elosztott terhelésű hozzá statikus magánhálózati IP-cím.
3. Hozzon létre egy rendelkezésre állási csoport figyelőjének, és képezze le a figyelő DNS-nevet a belső terheléselosztók IP-címét. 
4. Hozzon létre olyan terheléselosztó szabályhoz a figyelő az SQL Server-porthoz (TCP 1433-as port alapértelmezés szerint). Engedélyeznie kell a terheléselosztó szabályhoz *fix IP*, más néven a közvetlen kiszolgálói válasz. Ez azt eredményezi, hogy a virtuális gép közvetlenül válaszolni az ügyfelet, ami lehetővé teszi, hogy a közvetlen kapcsolat az elsődleges másodpéldányhoz.
  
  > [!NOTE]
  > Ha fix IP engedélyezve van, a előtér-portszám lehet ugyanaz, mint a háttér-port számát, a terheléselosztó szabályhoz.
  > 
  > 

Ha egy SQL-ügyfél próbál csatlakozni, a terheléselosztó a kapcsolódási kérelmet továbbítja a az elsődleges másodpéldány. Ha a feladatátvételt a replika egy másik, a terheléselosztó automatikusan továbbítja az későbbi kérelmeket egy új elsődleges replika. További információkért lásd: [egy ILB figyelőt az SQL Server Always On rendelkezésre állási csoportok konfigurálása][sql-alwayson-ilb].

A feladatátvétel során a már meglévő ügyfélkapcsolatok be legyen zárva. A feladatátvétel után a rendszer az új kapcsolatok az új elsődleges replika irányítsa.

Ha az alkalmazás lehetővé teszi az írási műveletek mint jóval több olvasási, kiszervezése a csak olvasható másodlagos replika lekérdezések némelyike. Lásd: [használja egy figyelő való csatlakozáshoz a csak olvasható másodlagos replika (csak olvasható Útválasztás)][sql-alwayson-read-only-routing].

Az üzemelő példány által [kényszerített kézi feladatátvétel] [ sql-alwayson-force-failover] a rendelkezésre állási csoport.

### <a name="jumpbox"></a>Jumpbox

A jumpbox fogja teljesítményének minimális követelmények vonatkoznak, így jelölje ki például a Standard A1 jumpbox kis Virtuálisgép-méretet. 

Hozzon létre egy [nyilvános IP-cím] a jumpbox számára. Helyezze a jumpbox, mint a többi virtuális gépe ugyanazon virtuális, de egy különálló felügyeleti alhálózat.

Engedélyezi az RDP a nyilvános interneten keresztül az alkalmazás munkaterhelés futtató virtuális gépeken. Ehelyett ezeket a virtuális gépek RDP-hozzáférést a jumpbox keresztül kell következnie. A rendszergazda bejelentkezik az a jumpbox, és majd bejelentkezik az a jumpbox a többi virtuális gép. A jumpbox RDP-forgalmát engedélyezi az internetről, de csak az ismert, biztonságos IP-címeket.

A jumpbox biztonságos hozzon létre egy NSG-t, és alkalmazza azt a jumpbox alhálózathoz. Vegyen fel egy NSG-t, amely lehetővé teszi az RDP-kapcsolatok csak egy készletből biztonságos nyilvános IP-címek. Az NSG csatolhatók az alhálózatot, vagy a jumpbox hálózati adaptert. Ebben az esetben ajánlott való csatlakoztatás a hálózati adapter, így az RDP-forgalmát engedélyezve van a csak a jumpbox, még akkor is, ha más virtuális gépek hozzá ugyanazon az alhálózaton.

Konfigurálja az NSG-k más alhálózatok felügyeleti alhálózatról RDP-forgalmát engedélyezi.

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

Az adatbázis-rétegből, hogy több virtuális gép automatikusan jelenti azt, hogy a magas rendelkezésre állású adatbázis. Egy relációs adatbázisban, az általában akkor replikációs és feladatátvételi használata magas rendelkezésre állás biztosítása érdekében. SQL Server esetében javasoljuk [Always On rendelkezésre állási csoportok][sql-alwayson]. 

Ha magas rendelkezésre állás érdekében, mint a [Azure SLA-t a virtuális gépek] [ vm-sla] biztosít, az alkalmazás két régiók közötti replikáció és a feladatátvételre használni az Azure Traffic Manager. További információkért lásd: [futtassa a Windows virtuális gépek magas rendelkezésre állásra több régióba][multi-dc].   

## <a name="security-considerations"></a>Biztonsági szempontok

Inaktív bizalmas adatok titkosítására és használata [Azure Key Vault] [ azure-key-vault] az adatbázis titkosítási kulcsok kezeléséhez. Key Vault képes tárolni a titkosítási kulcsokat a hardveres biztonsági modulokkal (HSM). További információkért lásd: [konfigurálása Azure Key Vault-integráció az SQL Server Azure virtuális gépeken] [ sql-keyvault] is javasoljuk Key Vault alkalmazás titkos adatait, például adatbázis-kapcsolati karakterláncok tárolni.

Fontolja meg a hálózati virtuális készülék (NVA) az internetről és az Azure virtuális hálózat között DMZ létrehozásához. NVA a virtuális készülék által végrehajtható műveleteket a hálózattal kapcsolatos feladatokat, mint például a tűzfal, csomag vizsgálati, naplózási és egyéni útválasztás neve. További információkért lásd: [Azure és az Internet között DMZ végrehajtási][dmz].

## <a name="scalability-considerations"></a>Méretezési szempontok

A terheléselosztók hálózati forgalmat és a web és business szintek terjesztése. Új Virtuálisgép-példányok hozzáadásával horizontálisan méretezhető. Vegye figyelembe, hogy méretezheti a web és business szintek egymástól függetlenül, a terhelés alapján. Ügyfélaffinitás fenntartásához szükséges okozta lehetséges komplikációk csökkentése érdekében a webes réteg a virtuális gépek állapotmentes kell lennie. A virtuális gépeket üzemeltető üzleti logikát is kell állapot nélküli.

## <a name="manageability-considerations"></a>Felügyeleti szempontok

Egyszerűbb kezelés a teljes rendszer központosított kiszolgálófelügyelet eszközei többek között a [Azure Automation][azure-administration], [a Microsoft Operations Management Suite] [ operations-management-suite], [Chef][chef], vagy [Puppet][puppet]. Ezek az eszközök konszolidálja rögzített több virtuális gépek és a rendszer általános áttekintést nyújt a diagnosztika és állapotfigyelő információkat.

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

A központi telepítés a referenciaarchitektúra megtalálható [GitHub][github-folder]. 

### <a name="prerequisites"></a>Előfeltételek

Mielőtt a saját előfizetésének telepítése a referencia-architektúrában, a következő lépésekkel kell azt.

1. Klónozza, ágaztassa vagy a zip-fájl letöltése a [AzureCAT referencia architektúra] [ ref-arch-repo] GitHub-tárházban.

2. Győződjön meg arról, hogy az Azure CLI 2.0 telepítve a számítógépre. A parancssori felület telepítéséhez kövesse az utasításokat a [Azure CLI 2.0 telepítése][azure-cli-2].

3. Telepítse a [Azure építőelemeket] [ azbb] npm csomag.

  ```bash
  npm install -g @mspnp/azure-building-blocks
  ```

4. A parancssorból bash, vagy PowerShell kérdés, jelentkezzen be az Azure-fiókjával használja az alábbi parancsok egyikét, és kövesse az utasításokat.

  ```bash
  az login
  ```

### <a name="deploy-the-solution-using-azbb"></a>Az üzembe helyezéséhez azbb használatával

A Windows virtuális gépek egy N szintű alkalmazás referencia architektúra telepítéséhez kövesse az alábbi lépéseket:

1. Keresse meg a `virtual-machines\n-tier-windows` az 1. lépésben a fenti előfeltételek klónozott tárház mappáját.

2. A paraméterfájl egy alapértelmezett adminsztrátori felhasználónevet és jelszót határozza meg a központi telepítésben lévő összes virtuális Géphez. Ezek a referencia-architektúrában központi telepítése előtt kell módosítani. Nyissa meg a `n-tier-windows.json` fájlt, és cserélje le az egyes **adminUsername** és **adminPassword** mező az új beállításokkal.
  
  > [!NOTE]
  > Nincsenek több futtatott parancsfájlok, a telepítés során mind a a **VirtualMachineExtension** objektumok és az a **bővítmények** egyes beállításokat a **VirtualMachine**objektumok. Ezek a parancsfájlok némelyike igényelnek, a rendszergazdai felhasználónevet és jelszót, amely csak módosította. Javasoljuk, hogy tekintse át ezeket a parancsfájlokat győződjön meg arról, hogy a megfelelő hitelesítő adatokat adott meg. A telepítés sikertelen lehet, ha nem adott meg a megfelelő hitelesítő adatokkal.
  > 
  > 

Mentse a fájlt.

3. Telepíthető a referencia architektúra például a **azbb** parancssori eszköz alább látható módon.

  ```bash
  azbb -s <your subscription_id> -g <your resource_group_name> -l <azure region> -p n-tier-windows.json --deploy
  ```

A minta referenciaarchitektúra használata Azure építőelemeket telepítésével kapcsolatos további információkért látogasson el a [GitHub-tárházban][git].


<!-- links -->
[dmz]: ../dmz/secure-vnet-dmz.md
[multi-dc]: multi-region-application.md
[multi-vm]: multi-vm.md
[n-tier]: n-tier.md
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azure-administration]: /azure/automation/automation-intro
[azure-availability-sets]: /azure/virtual-machines/virtual-machines-windows-manage-availability#configure-each-application-tier-into-separate-availability-sets
[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-cli-2]: https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest
[azure-key-vault]: https://azure.microsoft.com/services/key-vault
[megerősített állomás]: https://en.wikipedia.org/wiki/Bastion_host
[CIDR]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[chef]: https://www.chef.io/solutions/azure/
[git]: https://github.com/mspnp/template-building-blocks
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/n-tier-windows
[lb-external-create]: /azure/load-balancer/load-balancer-get-started-internet-portal
[lb-internal-create]: /azure/load-balancer/load-balancer-get-started-ilb-arm-portal
[load-balancer-external]: /azure/load-balancer/load-balancer-internet-overview
[load-balancer-internal]: /azure/load-balancer/load-balancer-internal-overview
[nsg]: /azure/virtual-network/virtual-networks-nsg
[operations-management-suite]: https://www.microsoft.com/server-cloud/operations-management-suite/overview.aspx
[plan-network]: /azure/virtual-network/virtual-network-vnet-plan-design-arm
[private-ip-space]: https://en.wikipedia.org/wiki/Private_network#Private_IPv4_address_spaces
[nyilvános IP-cím]: /azure/virtual-network/virtual-network-ip-addresses-overview-arm
[puppet]: https://puppetlabs.com/blog/managing-azure-virtual-machines-puppet
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[sql-alwayson]: https://msdn.microsoft.com/library/hh510230.aspx
[sql-alwayson-force-failover]: https://msdn.microsoft.com/library/ff877957.aspx
[sql-alwayson-getting-started]: https://msdn.microsoft.com/library/gg509118.aspx
[sql-alwayson-ilb]: /azure/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-alwayson-int-listener
[sql-alwayson-listeners]: https://msdn.microsoft.com/library/hh213417.aspx
[sql-alwayson-read-only-routing]: https://technet.microsoft.com/library/hh213417.aspx#ConnectToSecondary
[sql-keyvault]: /azure/virtual-machines/virtual-machines-windows-ps-sql-keyvault
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines
[vnet faq]: /azure/virtual-network/virtual-networks-faq
[wsfc-whats-new]: https://technet.microsoft.com/windows-server-docs/failover-clustering/whats-new-in-failover-clustering
[Nagios]: https://www.nagios.org/
[Zabbix]: http://www.zabbix.com/
[Icinga]: http://www.icinga.org/
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[0]: ./images/n-tier-diagram.png "Microsoft Azure használatával N szintű architektúra"