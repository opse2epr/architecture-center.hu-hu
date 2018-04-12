---
title: Windows rendszerű virtuális gépek futtatása N szintű architektúrákon
description: Többszintű architektúrák megvalósítása az Azure-ban, különös tekintettel a rendelkezésre állásra, a biztonságra, a méretezhetőségre és a kezelhetőségre.
author: MikeWasson
ms.date: 11/22/2016
pnp.series.title: Windows VM workloads
pnp.series.next: multi-region-application
pnp.series.prev: multi-vm
ms.openlocfilehash: 5ed94eb9ab8203d35d9597336e367d54e03944d7
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/06/2018
---
# <a name="run-windows-vms-for-an-n-tier-application"></a>Windows rendszerű virtuális gépek futtatása N szintű alkalmazáshoz

Ez a referencia-architektúra bevált eljárásokat mutat be arra az esetre, ha egy N szintű alkalmazáshoz szeretne Windows virtuális gépeket (VM-eket) futtatni. [**A megoldás üzembe helyezése**.](#deploy-the-solution) 

![[0]][0]

*Töltse le az architektúra [Visio-fájlját][visio-download].*

## <a name="architecture"></a>Architektúra 

Egy N szintű architektúra számos módon implementálható. Az ábrán egy tipikusnak mondható 3 szintű webalkalmazás látható. Ez az architektúra a következőre épül: [Elosztott terhelésű virtuális gépek futtatása a méretezhetőség és a rendelkezésre állás érdekében][multi-vm]. A webes és üzleti szintek elosztott terhelésű virtuális gépeket használnak.

* **Rendelkezésre állási csoportok.** Hozzon létre egy [rendelkezésre állási csoportot][azure-availability-sets] minden szinthez, majd minden szinten építsen ki legalább két virtuális gépet. Így a virtuális gépek magasabb szintű [szolgáltatói szerződésre (SLA-ra)][vm-sla] jogosultak. Azt is megteheti, hogy egyetlen virtuális gépet helyez üzembe a rendelkezésre állási csoportban, de egy virtuális gép nem jogosult SLA-garanciára, kivéve, ha Azure Premium Storage-t használ minden operációsrendszer- és adatlemezhez.  
* **Alhálózatok.** Hozzon létre egy külön alhálózatot minden egyes szinthez. A címtartományt és az alhálózati maszkot a [CIDR] jelöléssel határozza meg. 
* **Terheléselosztók**. Használjon egy [internetkapcsolattal rendelkező terheléselosztót][load-balancer-external] a bejövő internetes forgalom webes szinten történő elosztására, és egy [belső terheléselosztót][load-balancer-internal] a webes szintről az üzleti szintre irányuló hálózati forgalom elosztására.
* **Jumpbox.** Más néven [bástyagazdagép]. A hálózaton található biztonságos virtuális gép, amelyet a rendszergazdák a többi virtuális géphez való kapcsolódásra használnak. A jumpbox olyan NSG-vel rendelkezik, amely csak a biztonságos elemek listáján szereplő nyilvános IP-címekről érkező távoli forgalmat engedélyezi. Az NSG-nek engedélyeznie kell a távoli asztali (RDP) forgalmat.
* **Monitorozás.** A figyelőszoftverek, mint a [Nagios], a [Zabbix] vagy az [Icinga], információval szolgálhatnak a válaszidőről, a virtuális gép hasznos üzemidejéről és a rendszer általános állapotáról. Olyan virtuális gépre telepítse a figyelőszoftvert, amely egy különálló felügyeleti alhálózaton található.
* <strong>NSG-k</strong>. Használjon [hálózati biztonsági csoportokat][nsg] (NSG-ket) a hálózati forgalom korlátozására a virtuális hálózaton belül. Az itt látható 3 szintű architektúrában például az adatbázisszint csak az üzleti szintről és a felügyeleti alhálózatról érkező forgalmat fogadja, a webes kezelőfelület felől érkező forgalmat nem.
* **SQL Server Always On rendelkezésre állási csoport.** Magas rendelkezésre állást biztosít az adatszinten a replikáció és a feladatátvétel engedélyezésével.
* **(AD DS) Active Directory Domain Services-kiszolgálók** A Windows Server 2016 előtti verziók esetén az SQL Server Always On rendelkezésre állási csoportjait tartományhoz kell csatlakoztatni. Ennek az az oka, hogy a rendelkezésre állási csoportok a Windows Server feladatátvételi fürt (WSFC) technológián alapulnak. A Windows Server 2016-os verziójában a feladatátvételi fürtök már az Active Directory nélkül is létrehozhatók, amely esetben az architektúrához nincs szükség az AD DS-kiszolgálókra. További információ: [A Windows Server 2016 feladatátvételi fürtszolgáltatásával kapcsolatos újdonságok][wsfc-whats-new].
* **Azure DNS**. Az [Azure DNS][azure-dns] egy üzemeltetési szolgáltatás, amely a Microsoft Azure infrastruktúráját használja a DNS-tartományok névfeloldásához. Ha tartományait az Azure-ban üzemelteti, DNS-rekordjait a többi Azure-szolgáltatáshoz is használt hitelesítő adatokkal, API-kkal, eszközökkel és számlázási információkkal kezelheti.

## <a name="recommendations"></a>Javaslatok

Az Ön követelményei eltérhetnek az itt leírt architektúrától. Ezeket a javaslatokat tekintse kiindulópontnak. 

### <a name="vnet--subnets"></a>Virtuális hálózat/alhálózatok

A virtuális hálózat létrehozásakor határozza meg az egyes alhálózatok erőforrásai által igényelt IP-címek számát. [CIDR] jelöléssel adjon meg egy alhálózati maszkot és egy virtuális hálózati címtartományt, amely elég nagy a szükséges IP-címekhez. Használjon a szabványos [magánhálózati IP-címblokkokba][private-ip-space] eső címterületet. Ezek az IP-címblokkok a következők: 10.0.0.0/8, 172.16.0.0/12 és 192.168.0.0/16.

Válasszon olyan címtartományt, amely nincs átfedésben a helyszíni hálózattal, arra az esetre, ha később átjárót kell beállítania a virtuális hálózat és a helyszíni hálózat között. A virtuális hálózat létrehozása után a címtartomány nem módosítható.

Az alhálózatokat a funkciók és a biztonsági követelmények szem előtt tartásával tervezze meg. Minden ugyanazon szinthez vagy szerepkörhöz tartozó virtuális gépnek egyazon alhálózaton kell lennie. Ez az alhálózat biztonsági korlátot is képezhet. A virtuális hálózatok és alhálózatok megtervezésével kapcsolatos további információkért lásd az [Azure-beli virtuális hálózatok tervezésével és kialakításával][plan-network] foglalkozó témakört.

Az egyes alhálózatokon adja meg az alhálózat címterületét CIDR-jelölés formájában. A „10.0.0.0/24” például egy 256 IP-címből álló tartományt hoz létre. A virtuális gépek ezek közül 251-et használhatnak; öt foglalt. Győződjön meg arról, hogy a címtartományok nem fedik át egymást a különböző alhálózatokon. Lásd a [virtuális hálózatra vonatkozó gyakori kérdéseket][vnet faq].

### <a name="network-security-groups"></a>Network security groups (Hálózati biztonsági csoportok)

A szintek közötti forgalmat NSG-szabályokkal korlátozhatja. A fenti 3 szintes architektúra esetén például a webes szint nem kommunikál közvetlenül az adatbázisszinttel. Ennek kényszerítése érdekében az adatbázisszintnek blokkolnia kell a webes szint alhálózatáról érkező bejövő forgalmat.  

1. Hozzon létre egy NSG-t, és társítsa az adatbázisszint alhálózatához.
2. Adjon meg egy szabályt, amely minden, a virtuális hálózatról bejövő forgalmat blokkol. (A szabályban használja a `VIRTUAL_NETWORK` címkét.) 
3. Adjon meg egy magasabb prioritású szabályt, amely engedélyezi az üzleti szint alhálózatáról bejövő forgalmat. Ez a szabály felülírja az előzőt, és lehetővé teszi az üzleti szint számára az adatbázisszinttel folytatott kommunikációt.
4. Adjon meg egy szabályt, amely engedélyezi az adatbázisszint alhálózatán belülről érkező forgalmat. Ez lehetővé teszi a virtuális gépek közötti kommunikációt az adatbázisszinten, amelyre az adatbázis replikációja és feladatátvétele miatt van szükség.
5. Adjon hozzá egy szabályt, amely engedélyezi a jumpbox alhálózatról származó RDP-forgalmat. Ez lehetővé teszi, hogy a rendszergazdák csatlakozni tudjanak az adatbázisszinthez a jumpboxból.
   
   > [!NOTE]
   > A hálózati biztonsági csoportok olyan alapértelmezett szabályokkal rendelkeznek, amelyek engedélyezik a VNeten belülről érkező bejövő forgalmat. Ezek a szabályok nem törölhetők, de magasabb prioritású szabályok létrehozásával felülbírálhatók.
   > 
   > 

### <a name="load-balancers"></a>Terheléselosztók

A külső terheléselosztó osztja el az internetes forgalmat a webes szintre. Hozzon létre egy nyilvános IP-címet ehhez a terheléselosztóhoz. Lásd: [Internetkapcsolattal rendelkező terheléselosztó létrehozása][lb-external-create].

A belső terheléselosztó osztja el a webes szintről az üzleti szint felé irányuló hálózati forgalmat. Ahhoz, hogy ez a terheléselosztó magánhálózati IP-címre tehessen szert, hozzon létre egy előtérbeli IP-konfigurációt, és rendelje hozzá az üzleti szint alhálózatához. Lásd: [Belső terheléselosztó létrehozása][lb-internal-create].

### <a name="sql-server-always-on-availability-groups"></a>SQL Server Always On rendelkezésre állási csoportok

Magas rendelkezésre állású SQL Server esetén az [Always On rendelkezésre állási csoportok] [ sql-alwayson] használatát javasoljuk. A Windows Server 2016-nál régebbi kiadásokban az Always On rendelkezésre állási csoportok tartományvezérlőt igényelnek, és a rendelkezésre állási csoport összes csomópontjának azonos AD-tartományban kell lennie.

A többi szint egy [rendelkezésre állási csoport figyelőjén][sql-alwayson-listeners] keresztül csatlakozik az adatbázishoz. A figyelő lehetővé teszi, hogy az SQL-ügyfél anélkül csatlakozzon, hogy ismerné az SQL-kiszolgáló fizikai példányának nevét. Az adatbázishoz hozzáférő virtuális gépeket csatlakozni kell a tartományhoz. Az ügyfél (ebben az esetben egy másik szint) DNS használatával oldja fel a figyelő virtuális hálózati nevét IP-címekké.

A SQL Server Always On rendelkezésre állási csoportot az alábbiak szerint konfigurálja:

1. Hozzon létre egy Windows Server feladatátvételi fürtszolgáltatási (WSFC-) fürtöt, egy SQL Server Always On rendelkezésre állási csoportot és egy elsődleges replikát. További információ: [Ismerkedés az Always On rendelkezésre állási csoportokkal][sql-alwayson-getting-started]. 
2. Hozzon létre egy belső terheléselosztót statikus magánhálózati IP-címmel.
3. Hozzon létre egy figyelőt a rendelkezésre állási csoporthoz, és rendelje hozzá a figyelő DNS-nevét a belső terheléselosztó IP-címéhez. 
4. Hozzon létre egy terheléselosztó szabályt az SQL Server figyelőportjához (alapértelmezés szerint az 1433-as TCP-port). A terheléselosztó szabálynak engedélyeznie kell a *nem fix IP-címek* használatát, más néven a közvetlen kiszolgálói választ. Ezáltal a virtuális gép közvetlenül válaszol az ügyfélnek, és közvetlen kapcsolatot hozható létre az elsődleges replikával.
  
   > [!NOTE]
   > Ha a nem fix IP engedélyezve van, az előtérbeli portszámnak egyeznie kell a terheléselosztó szabály háttérbeli portszámával.
   > 
   > 

Ha egy SQL-ügyfél csatlakozni próbál, a terheléselosztó továbbítja az elsődleges replikának a kapcsolódási kérelmet. Ha feladatátvitel történik egy másik replikára, akkor a terheléselosztó automatikusan egy új elsődleges replikához továbbítja a további kéréseket. További információ: [ILB-figyelő konfigurálása SQL Server Always On rendelkezésre állási csoportokhoz][sql-alwayson-ilb].

A feladatátvétel során a meglévő ügyfélkapcsolatok lezárulnak. A feladatátvétel befejezése után a rendszer az új elsődleges replikára irányítja az új kapcsolatokat.

Ha az alkalmazás olvasási műveleteinek száma lényegesen több, mint az írási műveleteteké, a csak olvasási lekérdezések egy részét kiszervezheti egy másodlagos replikára. Lásd: [Csak olvasási másodlagos replikához való csatlakozás figyelővel (csak olvasási útválasztás)][sql-alwayson-read-only-routing].

[Kényszerítse][sql-alwayson-force-failover] a rendelkezésre állási csoport manuális feladatátvételét az üzemelő példány teszteléséhez.

### <a name="jumpbox"></a>Jumpbox

A jumpboxnak minimális teljesítménykövetelményei vannak, ezért kis virtuálisgép-méretet jelöljön ki a számára, például egy standard A1-et. 

Hozzon létre egy [nyilvános IP-címet] a jumpbox számára. Helyezze a jumpboxot a többi virtuális géppel megegyező virtuális hálózatba, de egy külön felügyeleti alhálózaton legyen.

Az alkalmazás számítási feladatait futtató virtuális gépek nyilvános internetről való RDP-hozzáférését ne engedélyezze. Az ilyen virtuális gépek RDP-hozzáférésének ehelyett a jumpboxon keresztül kell történnie. A rendszergazda először bejelentkezik a jumpboxba, majd azon keresztül bejelentkezik a többi virtuális gépbe. A jumpbox engedélyezi az internetről érkező RDP-forgalmat, de csak az ismert, biztonságos IP-címekről.

A jumpbox védelmére hozzon létre egy NSG-t, és alkalmazza azt a jumpbox alhálózatára. Vegyen fel egy NSG-szabályt, amely csak a biztonságos nyilvános IP-címkészletekről engedélyezi az RDP-kapcsolatokat. Az NSG az alhálózathoz vagy a jumpbox hálózati adapteréhez is csatolható. Ebben az esetben ajánlott a hálózati adapterhez való csatlakoztatás, így az RDP-forgalom akkor is csak a jumpbox irányába van engedélyezve, ha más virtuális gépeket ad hozzá ugyanahhoz az alhálózathoz.

Állítsa be az NSG-t a többi alhálózathoz is úgy, hogy engedélyezzék a felügyeleti alhálózatból érkező RDP-forgalmat.

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

Hiába van több virtuális gépe, ez az adatbázisszinten nem jelent automatikusan magasabb rendelkezésre állást. Relációs adatbázis esetén rendszerint replikációval és feladatátvétellel biztosítható a magas rendelkezésre állás. Az SQL Server esetében javasoljuk az [Always On rendelkezésre állási csoportok][sql-alwayson] használatát. 

Ha magasabb rendelkezésre állás szükséges, mint amelyet a [virtuális gépekhez készült Azure SLA nyújt][vm-sla], replikálja az alkalmazást két régióban, a feladatátvételre pedig használja az Azure Traffic Managert. További információ: [Windows rendszerű virtuális gépek futtatása több régióban a magas rendelkezésre állás érdekében][multi-dc].   

## <a name="security-considerations"></a>Biztonsági szempontok

Titkosíthatja az inaktív bizalmas adatokat, és az [Azure Key Vaulttal][azure-key-vault] kezelheti az adatbázis titkosítási kulcsait. A Key Vault képes a hardveres biztonsági modulok (HSM-ek) titkosítási kulcsainak tárolására. További információ: [Az Azure Key Vault-integráció SQL Serverhez való integrálása Azure-beli virtuális gépeken][sql-keyvault]. Emellett javasoljuk, hogy az alkalmazások titkos adatait, például az adatbázis-kapcsolati karakterláncokat tárolja a Key Vaultban.

Érdemes lehet hozzáadnia egy hálózati virtuális berendezést (network virtual appliance, NVA), hogy DMZ-t lehessen létrehozni az internet és az Azure-beli virtuális hálózat között. Az NVA egy általános kifejezés egy olyan virtuális berendezésre, amely hálózatokhoz kapcsolódó feladatokat lát el, például gondoskodik a tűzfalról, a csomagvizsgálatról, a naplózásról és az egyéni útválasztásról. További információkért lásd a [DMZ az Azure és az internet közötti implementálásával][dmz] foglalkozó témakört.

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

Ha Windows rendszerű virtuális gépeket szeretne üzembe helyezni egy N szintű alkalmazás referenciaarchitektúrájához, kövesse az alábbi lépéseket:

1. Navigáljon a fenti előfeltételek 1. lépése során klónozott adattár `virtual-machines\n-tier-windows` mappájához.

2. A paraméterfájl egy alapértelmezett rendszergazdai felhasználónevet és jelszót határoz meg az üzemelő példány minden egyes virtuális gépéhez. Ezeket módosítania kell, mielőtt üzembe helyezné a referenciaarchitektúrát. Nyissa meg az `n-tier-windows.json` fájlt, és cserélje le az egyes **adminUsername** és **adminPassword** mezők tartalmát az új beállításokra.
  
   > [!NOTE]
   > Az üzembe helyezés alatt több szkript is fut a **VirtualMachineExtension** objektumokban és egyes **VirtualMachine** objektumok **extensions** beállításaiban. Ezen szkriptek némelyike az imént módosított rendszergazdai felhasználónevet és jelszót kéri. Javasoljuk, hogy tekintse át ezeket a szkripteket, és győződjön meg arról, hogy megfelelő hitelesítő adatokat adott meg. A telepítés sikertelen lehet, ha a megfelelő hitelesítő adatokat használt.
   > 
   > 

Mentse a fájlt.

3. Helyezze üzembe a referenciaarchitektúrát az **azbb** parancssori eszköz használatával az alább látható módon.

   ```bash
   azbb -s <your subscription_id> -g <your resource_group_name> -l <azure region> -p n-tier-windows.json --deploy
   ```

A mintául szolgáló referenciaarchitektúra Azure-építőelemekkel történő üzembe helyezéséről további információkat a [GitHub-adattárban][git] talál.


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
[azure-dns]: /azure/dns/dns-overview
[azure-key-vault]: https://azure.microsoft.com/services/key-vault
[bástyagazdagép]: https://en.wikipedia.org/wiki/Bastion_host
[cidr]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
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
[nyilvános IP-címet]: /azure/virtual-network/virtual-network-ip-addresses-overview-arm
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
[visio-download]: https://archcenter.blob.core.windows.net/cdn/vm-reference-architectures.vsdx
[0]: ./images/n-tier-diagram.png "N szintű architektúra a Microsoft Azure használatával"