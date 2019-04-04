---
title: Magas rendelkezésre állású SharePoint Server 2016-farm futtatása az Azure-ban
titleSuffix: Azure Reference Architectures
description: Ajánlott architektúra magas rendelkezésre állású SharePoint Server 2016-farm Azure-ban való üzembe helyezéséhez.
author: njray
ms.date: 07/26/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seodec18
ms.openlocfilehash: 0cce207dedd0eb42e29a152b3bb84acc2dca323a
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58344988"
---
# <a name="run-a-highly-available-sharepoint-server-2016-farm-in-azure"></a>Magas rendelkezésre állású SharePoint Server 2016-farm futtatása az Azure-ban

A jelen referenciaarchitektúra bevált módszereket mutat be a magas rendelkezésre állású SharePoint Server 2016-farmok Azure-ban való üzembe helyezésére a MinRole topológia és az SQL Server Always On rendelkezésre állási csoportok használatával. A SharePoint-farm egy biztonságos virtuális hálózaton lesz üzembe helyezve, amelynek nincs internetes végpontja vagy jelenléte. [**A megoldás üzembe helyezése.**](#deploy-the-solution)

![Magas rendelkezésre állású Azure-beli SharePoint Server 2016-farm referenciaarchitektúrája](./images/sharepoint-ha.png)

*Töltse le az architektúra [Visio-fájlját][visio-download].*

## <a name="architecture"></a>Architektúra

Ez az architektúra a [Windows rendszerű virtuális gépek futtatása N szintű alkalmazáshoz][windows-n-tier] című cikkben bemutatott architektúrára épít. Egy magas rendelkezésre állású SharePoint Server 2016-farmot helyez üzembe egy Azure-beli virtuális hálózaton (VNet) belül. Az architektúra alkalmazható a teszt- és az éles környezetekhez, a SharePoint Office 365-öt tartalmazó hibrid infrastruktúráihoz, vagy mint a vészhelyreállítás alapja.

Az architektúra az alábbi összetevőkből áll:

- **Erőforráscsoportok**. Az [erőforráscsoport][resource-group] egy tároló, amely kapcsolódó Azure-erőforrásokat tárol. A rendszer egy erőforráscsoportot használ a SharePoint-kiszolgálókhoz, és egy másik erőforráscsoportot a virtuális gépektől független infrastruktúra-összetevőkhöz, például a virtuális hálózathoz és a terheléselosztókhoz.

- **Virtuális hálózat (VNet)**. A rendszer a virtuális gépeket egy egyedi intranetes címterülettel rendelkező virtuális hálózaton helyezi üzembe. A virtuális hálózat további alhálózatokra oszlik.

- **Virtuális gépek (VM-ek)**. A rendszer üzembe helyezi a virtuális gépeket a virtuális hálózaton, és statikus magánhálózati IP-címeket rendel az összes virtuális géphez. Az IP-címek gyorsítótárazásával kapcsolatos problémák és a címek újraindítás utáni megváltozásának megelőzéséhez az SQL Servert és a SharePoint Server 2016-ot futtató virtuális gépeken javasolt a statikus IP-címek használata.

- **Rendelkezésre állási csoportok**. Helyezze az egyes SharePoint-szerepkörökhöz tartozó virtuális gépeket külön [rendelkezésre állási csoportokba][availability-set], és helyezzen üzembe legalább két virtuális gépet minden szerepkörhöz. A virtuális gépek így magasabb szintű szolgáltatói szerződésre (SLA-ra) jogosultak.

- **Belső terheléselosztó**. A [terheléselosztó][load-balancer] elosztja a helyszíni hálózatról érkező SharePoint-kérés forgalmát a SharePoint-farm előtér-webkiszolgálói között.

- **Hálózati biztonsági csoportok (NSG-k)**. A virtuális gépeket tartalmazó összes alhálózaton létrejön egy [hálózati biztonsági csoport][nsg]. Az NSG-ket használja a hálózati forgalom korlátozására a virtuális hálózaton belül, hogy el lehessen különíteni az alhálózatokat.

- **Átjáró**. Az átjáró kapcsolatot biztosít a helyszíni hálózat és az Azure-beli virtuális hálózat között. A kapcsolat ExpressRoute- vagy helyek közötti VPN-átjárót használhat. További információ: [Helyszíni hálózat csatlakoztatása az Azure-hoz][hybrid-ra].

- **Windows Server Active Directory- (AD) tartományvezérlők**. Ez a referenciaarchitektúra Windows Server Active Directory- (AD-) tartományvezérlőket helyez üzembe. Ezek a tartományvezérlők az Azure-beli virtuális hálózaton futnak, és megbízhatósági viszonyban vannak a helyszíni Windows Server AD-erdővel. Az ügyfelek SharePoint-farmokon lévő erőforrásokkal kapcsolatos webes kéréseit a rendszer a virtuális hálózaton hitelesíti ahelyett, hogy továbbítaná a hitelesítési forgalmat az átjárókapcsolaton keresztül a helyszíni hálózatra. A DNS-ben intranet A vagy CNAME-rekordok jönnek létre, hogy az intranetes felhasználók fel tudják oldani a SharePoint-farm nevét a belső terheléselosztó magánhálózati IP-címére.

  A SharePoint Server 2016 támogatja az [Azure Active Directory Domain Services](/azure/active-directory-domain-services/) használatát is. Az Azure AD Domain Services felügyelt tartományi szolgáltatásokat biztosít, hogy az Azure-ban ne legyen szükség tartományvezérlők üzembe helyezésére és felügyeletére.

- **SQL Server Always On rendelkezésre állási csoport**. Az SQL Server-adatbázisok magas rendelkezésre állása érdekében javasoljuk a [SQL Server Always On rendelkezésre állási csoportok][sql-always-on] használatát. A rendszer két virtuális gépet használ az SQL Serverhez. Az egyik az elsődleges adatbázis-replikát tartalmazza, a másik pedig a másodlagos replikát.

- **Csomóponttöbbséget használó virtuális gép**. Ez a virtuális gép lehetővé teszi, hogy a feladatátvevő fürt kvórumot hozzon létre. További információ: [A feladatátvevő fürtök kvórumkonfigurációinak ismertetése][sql-quorum].

- **SharePoint-kiszolgálók**. A SharePoint-kiszolgálók töltik be a webes előtér, a gyorsítótárazás, alkalmazás és a keresés szerepköröket.

- **Jumpbox**. Más néven [bástyagazdagép][bastion-host]. A hálózaton található biztonságos virtuális gép, amelyet a rendszergazdák a többi virtuális géphez való kapcsolódásra használnak. A jumpbox olyan NSG-vel rendelkezik, amely csak a biztonságos elemek listáján szereplő nyilvános IP-címekről érkező távoli forgalmat engedélyezi. Az NSG-nek engedélyeznie kell a távoli asztali (RDP) forgalmat.

## <a name="recommendations"></a>Javaslatok

Az Ön követelményei eltérhetnek az itt leírt architektúrától. Ezeket a javaslatokat tekintse kiindulópontnak.

### <a name="resource-group-recommendations"></a>Erőforráscsoportra vonatkozó javaslatok

Javasoljuk, hogy különítse el az erőforráscsoportokat kiszolgálói szerepkör szerint, és hozzon létre egy külön erőforráscsoportot a globális erőforrásnak számító infrastruktúra-összetevők számára. Ebben az architektúrában a SharePoint-erőforrások alkotják az egyik csoportot, az SQL Server és a többi segédeszköz pedig a másikat.

### <a name="virtual-network-and-subnet-recommendations"></a>Virtuális hálózatra és alhálózatra vonatkozó javaslatok

Minden SharePoint-szerepkörhöz használjon egy alhálózatot, plusz egy alhálózatot az átjáróhoz, és egyet a jumpboxhoz.

Az átjáró-alhálózatnak a *GatewaySubnet* névvel kell rendelkeznie. Rendeljen hozzá egy címteret az átjáró-alhálózathoz a virtuális hálózat címterének végéről. További információ: [Helyszíni hálózat csatlakoztatása az Azure-hoz VPN-átjáró használatával][hybrid-vpn-ra].

### <a name="vm-recommendations"></a>Virtuális gépekre vonatkozó javaslatok

Ehhez az architektúrához legalább 44 mag szükséges:

- 8 SharePoint-kiszolgáló, Standard_DS3_v2 méret (4 mag/darab) = 32 mag
- 2 Active Directory-tartományvezérlő, Standard_DS1_v2 méret (1 mag/darab) = 2 mag
- 2 SQL Servert futtató virtuális gép, Standard_DS3_v2 méret = 8 mag
- 1 csomóponttöbbség, Standard_DS1_v2 méret = 1 mag
- 1 felügyeleti kiszolgáló, Standard_DS1_v2 méret = 1 mag

Győződjön meg arról, hogy az Azure-előfizetésnek elegendő virtuálisgép-magkvóta áll rendelkezésére, különben az üzembe helyezés sikertelen lesz. Lásd: [Az Azure-előfizetések és -szolgáltatások korlátozásai, kvótái és megkötései][quotas].

A keresési indexelő kivételével az összes SharePoint-szerepkörhöz javasoljuk a [Standard_DS3_v2][vm-sizes-general] virtuálisgép-méret használatát. A keresési indexelőnek legalább [Standard_DS13_v2][vm-sizes-memory] méretűnek kell lennie. Teszteléshez a referenciaarchitektúra paraméterfájljai a kisebb, DS3_v2 méretet adják meg a keresési indexelő szerepkörhöz. Éles környezetek esetén frissítse a paraméterfájlokat a DS13 vagy nagyobb méret használatára. További információkat a [SharePoint Server 2016 hardver- és szoftverkövetelményeit][sharepoint-reqs] ismertető cikkben találhat.

SQL Servert futtató virtuális gépekhez legalább 4 mag és 8 GB RAM szükséges. A referenciaarchitektúra paraméterfájljai a DS3_v2 méretet adják meg. Előfordulhat, hogy éles környezet esetén nagyobb virtuálisgép-méretet kell megadnia. További információkat [a tárolás és az SQL Server kapacitástervezésével és konfigurálásával (SharePoint Server)](/sharepoint/administration/storage-and-sql-server-capacity-planning-and-configuration#estimate-memory-requirements) kapcsolatos cikkben olvashat.

### <a name="nsg-recommendations"></a>NSG-kre vonatkozó javaslatok

Javasoljuk, hogy legyen egy NSG minden egyes virtuális gépeket tartalmazó alhálózathoz, hogy el lehessen különíteni az alhálózatokat. Ha alhálózat-elkülönítést szeretne konfigurálni, adjon meg olyan NSG-szabályokat, amelyek minden egyes alhálózatnál meghatározzák az engedélyezett vagy tiltott bejövő és kimenő adatforgalmat. További információ: [Hálózati forgalom szűrése hálózati biztonsági csoportokkal][virtual-networks-nsg].

Ne rendeljen NSG-ket az átjáró-alhálózathoz, különben az átjáró nem fog működni.

### <a name="storage-recommendations"></a>Tárolásra vonatkozó javaslatok

A farmon található virtuális gépek tárolókonfigurációjának egyeznie kell a helyszíni üzemelő példányoknál használt megfelelő ajánlott eljárásokkal. A SharePoint-kiszolgálóknak külön lemezzel kell rendelkezniük a naplók számára. A keresési index szerepköröket üzemeltető SharePoint-kiszolgálóknak további lemezterületre van szükségük a keresési index tárolásához. Az SQL Server esetén a szabványos eljárás az adatok és a naplók elkülönítése. Adjon hozzá további lemezeket az adatbázis biztonsági másolatainak tárolásához, és használjon egy külön lemezt a [tempdb][tempdb] számára.

A legjobb megbízhatóság érdekében javasoljuk az [Azure Managed Disks][managed-disks] használatát. A felügyelt lemezek biztosítják, hogy a rendelkezésre állási csoportban elkülönüljenek egymástól a virtuális gépek lemezei a kritikus hibapontok elkerülése érdekében.

> [!NOTE]
> A referenciaarchitektúra Resource Manager-sablonja jelenleg nem használ felügyelt lemezeket. Tervezzük a sablont frissítését a felügyelt lemezek használatára.

A SharePointot és az SQL Servert futtató virtuális gépeknél használjon Prémium felügyelt lemezeket. A csomóponttöbbséget használó kiszolgáló, a tartományvezérlők és a felügyeleti kiszolgáló esetén használhat Standard felügyelt lemezeket.

### <a name="sharepoint-server-recommendations"></a>SharePoint Serverre vonatkozó javaslatok

A SharePoint-farm konfigurálása előtt győződjön meg arról, hogy minden szolgáltatáshoz rendelkezik Windows Server Active Directory-szolgáltatásfiókkal. Ennél az architektúránál a jogosultság szerepkörönként való elkülönítéséhez legalább az alábbi tartományszintű fiókokra van szükség:

- SQL Server-szolgáltatásfiók
- Telepítési felhasználói fiók
- Kiszolgálófarm-fiók
- Keresésiszolgáltatás-fiók
- Tartalom-hozzáférési fiók
- Webalkalmazáskészlet-fiókok
- Szolgáltatási alkalmazáskészlet-fiókok
- Gyorsítótár-felügyelői fiók
- Emelt jogosultsági szintű gyorsítótár-olvasói fiók

Ne felejtse el megtervezni a keresési architektúrát, hogy meg tudjon felelni a lemezteljesítmény minimális támogatási követelményének (200 MB/s). Lásd: [Vállalati keresési architektúra megtervezése a SharePoint Server 2013 rendszerben][sharepoint-search]. Kövesse a [SharePoint Server 2016 bejárásának ajánlott eljárásaira][sharepoint-crawling] vonatkozó útmutatást is.

Továbbá a keresési összetevő adatait tárolja egy külön tárolóköteten vagy nagy teljesítményű partíción. A terhelés csökkentéséhez és a teljesítmény növeléséhez konfigurálja az architektúrához szükséges objektum-gyorsítótár felhasználói fiókjait. A Windows Server operációsrendszer-fájljait, a SharePoint Server 2016 programfájljait és a diagnosztikai naplókat ossza el három külön tárolókötet vagy normál teljesítményű partíció között.

További információk a javaslatokról: [Az első üzembe helyezés rendszergazdai- és szolgáltatásfiókjai a SharePoint Server 2016 rendszerben][sharepoint-accounts].

### <a name="hybrid-workloads"></a>Hibrid számítási feladatok

Ez a referenciaarchitektúra egy SharePoint Server 2016-farmot helyez üzembe, amelyet [SharePoint hibrid környezetként][sharepoint-hybrid] lehet használni &mdash; azaz kiterjeszti a SharePoint Server 2016-ot az Office 365 SharePoint Online-ra. Ha már rendelkezik Office Online Serverrel, lásd: [Az Office Web Apps és az Office Online Server támogatása az Azure-ban][office-web-apps].

A telepítés alapértelmezett szolgáltatásalkalmazásait úgy alakították ki, hogy támogassák a hibrid számítási feladatokat. A SharePoint Server 2016 és az Office 365 bármely hibrid számítási feladatát üzembe lehet helyezni ezen a farmon a SharePoint-infrastruktúra módosítása nélkül, egy kivétellel: A Cloud Hybrid Search szolgáltatásalkalmazást nem lehet üzembe helyezni olyan kiszolgálókon, amelyek meglévő keresési topológiát üzemeltetnek. A farmhoz ezért hozzá kell adni legalább egy keresői szerepköralapú virtuális gépet, hogy támogassa ezt a hibrid forgatókönyvet.

### <a name="sql-server-always-on-availability-groups"></a>SQL Server Always On rendelkezésre állási csoportok

Ez az architektúra SQL Servert futtató virtuális gépeket használ, mivel a SharePoint Server 2016 nem tudja az Azure SQL Database-t használni. A magas rendelkezésre állás SQL Serveren való támogatásához javasoljuk, hogy használja az Always On rendelkezésre állási csoportokat. Ezek megadnak egy adatbáziskészletet, amelynek az adatbázisai együtt hajtják végre a feladatátvételt, így magas rendelkezésre állásúak és helyreállíthatók lesznek. Ebben a referenciaarchitektúrában az adatbázisok létrehozása az üzembe helyezéskor történik, viszont manuálisan engedélyeznie kell az Always On rendelkezésre állási csoportokat, és hozzá kell adnia a SharePoint-adatbázisokat egy rendelkezésre állási csoporthoz. További információ: [A rendelkezésre állási csoport létrehozása és a SharePoint-adatbázisok hozzáadása][create-availability-group].

Emellett azt is javasoljuk, hogy adjon hozzá a fürthöz egy figyelő IP-címet. Ez az SQL Servert futtató virtuális gépek belső terheléselosztójának magánhálózati IP-címe.

Az ajánlott virtuálisgép-méretekért és az Azure-ban futó SQL Server teljesítményével kapcsolatos egyéb javaslatokért lásd: [Az SQL Server teljesítményéhez kapcsolódó ajánlott eljárások Azure-beli virtuális gépeken][sql-performance]. Kövesse az [A SharePoint Server 2016-farmon futó SQL Serverre vonatkozó ajánlott eljárások][sql-sharepoint-best-practices] útmutatását is.

Javasoljuk, hogy a csomóponttöbbséget használó kiszolgálót ne azon a számítógépen helyezze el, amelyen a replikációs partnerek találhatók. A kiszolgáló lehetővé teszi, hogy a másodlagos replikációs partnerkiszolgáló a magas biztonsági üzemmódú munkamenetben felismerje, mikor kell automatikus feladatátvételt kezdeményeznie. A két partnerrel ellentétben a csomóponttöbbséget használó kiszolgáló nem az adatbázist szolgálja ki, hanem az automatikus feladatátvételt támogatja.

## <a name="scalability-considerations"></a>Méretezési szempontok

A meglévő kiszolgálók vertikális felskálázásához egyszerűen módosítsa a virtuális gép méretét.

A SharePoint Server 2016 [MinRoles][minroles] képességével horizontálisan felskálázhatja a kiszolgálókat a kiszolgáló szerepköre alapján, valamint el is távolíthat kiszolgálókat egy adott szerepkörből. Amikor kiszolgálókat ad hozzá egy szerepkörhöz, megadhatja bármelyik önálló szerepkört vagy a kombinált szerepkörök egyikét. Ha azonban a keresési szerepkörhöz ad hozzá kiszolgálókat, a keresési topológiát is újra kell konfigurálnia a PowerShell használatával. A szerepköröket át is alakíthatja a MinRoles használatával. További információ: [MinRole-kiszolgálófarmok kezelése a SharePoint Server 2016-ban][sharepoint-minrole].

Vegye figyelembe, hogy a SharePoint Server 2016 nem támogatja a virtuálisgép-méretezési csoportok automatikus skálázásra történő használatát.

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

Ez a referenciaarchitektúra támogatja a magas rendelkezésre állást az Azure-régióban, mivel minden egyes szerepkörhöz legalább két virtuális gép van üzembe helyezve egy rendelkezésre állási csoportban.

A regionális meghibásodással szembeni védelemhez hozzon létre egy külön vészhelyreállítási farmot egy másik Azure-régióban. A telepítési követelményeket a helyreállításiidő-célkitűzések (RTO-k) és a helyreállításipont-célkitűzések (RPO-k) fogják meghatározni. További részleteké: [Vészhelyreállítási stratégia kiválasztása a SharePoint 2016-hoz][sharepoint-dr]. A másodlagos régiót *párosítani* kell az elsődleges régióval. Széles körű leállás esetén minden párból az egyik régió helyreállítása előnyt élvez. További információ: [Üzletmenet-folytonosság és vészhelyreállítás (BCDR): Az Azure párosított régiói][paired-regions].

## <a name="manageability-considerations"></a>Felügyeleti szempontok

A kiszolgálók, kiszolgálófarmok és helyek működtetéséhez és karbantartásához kövesse a SharePoint-műveletekre vonatkozó ajánlott eljárásokat. További információ: [A SharePoint Server 2016 műveletei][sharepoint-ops].

Az SQL Server SharePoint-környezetben való felügyeletekor megfontolandó feladatok eltérhetnek az adatbázis-alkalmazásoknál jellemzően megfontolt feladatoktól. Az ajánlott eljárás az SQL-adatbázisok hetente történő teljes körű biztonsági mentése, növekményes éjszakai biztonsági mentésekkel. Készítsen biztonsági másolatot a tranzakciónaplókról 15 percenként. Egy másik eljárás az SQL Server karbantartási feladatainak megvalósítása az adatbázisokon, miközben letiltja a beépített SharePoint karbantartási feladatokat. További információ: [A tárolás és az SQL Server kapacitástervezése és konfigurálása][sql-server-capacity-planning].

## <a name="security-considerations"></a>Biztonsági szempontok

A SharePoint Server 2016 futtatásához használt tartományszintű szolgáltatásfiókoknak Windows Server AD-tartományvezérlőkre van szükségük a tartományhoz való csatlakozáshoz és a hitelesítési folyamatokhoz. Az Azure Active Directory Domain Services nem használható erre a célra. Az intraneten már elhelyezett Windows Server AD identitás-infrastruktúra bővítéséhez ez az architektúra egy meglévő helyszíni Windows Server AD-erdő két Windows Server AD replika tartományvezérlőjét használja.

Emellett mindig célszerű előre megtervezni a biztonság megerősítését. Egyéb javaslatok:

- Adjon szabályokat az NSG-khez az alhálózatok és a szerepkörök elkülönítéséhez.
- Ne rendeljen nyilvános IP-címeket a virtuális gépekhez.
- A behatolásészleléshez és a hasznos adatforgalom elemzéséhez fontolja meg egy virtuális hálózati berendezés használatát az előtér-webkiszolgálók előtt a belső Azure Load Balancer helyett.
- Igény szerint IPsec-házirendekkel titkosíthatja a kiszolgálók közötti nem titkosított forgalmat. Ha az alhálózatokat is elkülöníti, frissítse a hálózati biztonsági csoport szabályait, hogy engedélyezzék az IPsec-forgalmat.
- Telepítsen kártevőirtó ügynököket a virtuális gépekre.

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

Ennek a referenciaarchitektúrának egy üzemelő példánya elérhető a [GitHubon][github]. A teljes üzembe helyezés akár több órát is igénybe vehet.

Az üzembe helyezés a következő erőforráscsoportokat hozza létre az előfizetésben:

- ra-onprem-sp2016-rg
- ra-sp2016-network-rg

A sablon paraméterfájljai ezekre a nevekre hivatkoznak, ezért ha módosítja őket, a paraméterfájlokat is frissítse.

A paraméterfájlok több helyen nem módosítható jelszót tartalmaznak. Módosítsa ezeket az értékeket az üzembe helyezés előtt.

### <a name="prerequisites"></a>Előfeltételek

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deployment-steps"></a>A központi telepítés lépései

1. Futtassa az alábbi parancsot egy szimulált helyszíni hálózat üzembe helyezéséhez.

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p onprem.json --deploy
    ```

2. Futtassa az alábbi parancsot az Azure-beli virtuális hálózat és a VPN-átjáró üzembe helyezéséhez.

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p connections.json --deploy
    ```

3. Futtassa az alábbi parancsot a jumpbox, az AD-tartományvezérlők és az SQL Server-virtuálisgépek üzembe helyezéséhez.

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p azure1.json --deploy
    ```

4. Futtassa az alábbi parancsot a feladatátvevő fürt és a rendelkezésreállási csoport létrehozásához.

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p azure2-cluster.json --deploy
    ```

5. Futtassa az alábbi parancsot a fennmaradó virtuális gépek üzembe helyezéséhez.

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p azure3.json --deploy
    ```

Ezen a ponton ellenőrizze, hogy tud-e létesíteni TCP-kapcsolatot az SQL Server Always On rendelkezésreállási csoport számára a webes kezelőfelület és a terheléselosztó között. Ehhez végezze el az alábbi lépéseket:

1. Az Azure Portal használatával keresse meg a `ra-sp-jb-vm1` nevű virtuális gépet az `ra-sp2016-network-rg` erőforráscsoportban. Ez a jumpbox-virtuálisgép.

2. Kattintson a `Connect` parancsra egy, a virtuális gépre irányuló távoli asztali munkamenet megnyitásához. Használja az `azure1.json` paraméterfájlban megadott jelszót.

3. A távoli asztali munkamenetből jelentkezzen be a 10.0.5.4-es IP-címre. Ez az `ra-sp-app-vm1` nevű virtuális gép IP-címe.

4. Nyisson meg egy PowerShell-konzolt a virtuális gépen, és a `Test-NetConnection` parancsmaggal ellenőrizze, hogy csatlakozni tud-e a terheléselosztóhoz.

    ```powershell
    Test-NetConnection 10.0.3.100 -Port 1433
    ```

A kimenetnek a következőképpen kell kinéznie:

```console
ComputerName     : 10.0.3.100
RemoteAddress    : 10.0.3.100
RemotePort       : 1433
InterfaceAlias   : Ethernet 3
SourceAddress    : 10.0.0.132
TcpTestSucceeded : True
```

Ha ez nem sikerül, az Azure Portal segítségével indítsa újra az `ra-sp-sql-vm2` nevű virtuális gépet. Amikor a virtuális gép újraindult, futtassa ismét a `Test-NetConnection` parancsot. A virtuális gép újraindítása után előfordulhat, hogy várnia kell körülbelül egy percet ahhoz, hogy a csatlakozás sikeres legyen.

Most pedig végezze el az üzembe helyezést az alábbi módon.

1. Futtassa az alábbi parancsot a SharePoint-farm elsődleges csomópontjának üzembe helyezéséhez.

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p azure4-sharepoint-server.json --deploy
    ```

2. Futtassa az alábbi parancsot a SharePoint-gyorsítótár, -keresés és -web üzembe helyezéséhez.

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p azure5-sharepoint-farm.json --deploy
    ```

3. Futtassa az alábbi parancsot az NSG-szabályok létrehozásához.

    ```bash
    azbb -s <subscription_id> -g ra-onprem-sp2016-rg -l <location> -p azure6-security.json --deploy
    ```

### <a name="validate-the-deployment"></a>Az üzembe helyezés ellenőrzése

1. Az [Azure Portalon][azure-portal] keresse meg az `ra-onprem-sp2016-rg` erőforráscsoportot.

2. Az erőforrások listájában válassza ki a `ra-onpr-u-vm1` nevű virtuálisgép-erőforrást.

3. Csatlakozzon a virtuális géphez a [Csatlakozás virtuális géphez][connect-to-vm] című részben ismertetett módon. Felhasználónév: `\onpremuser`.

4. Ha létrejött a távoli kapcsolat a virtuális géppel, nyisson meg egy böngészőt a virtuális gépen, és navigáljon a következő helyre: `http://portal.contoso.local`.

5. A **Windows rendszerbiztonság** mezőben jelentkezzen be a SharePoint portálra a `contoso.local\testuser` felhasználónévvel.

Ez a bejelentkezés a helyszíni hálózat által használt Fabrikam.com tartományból nyit alagutat a SharePoint portál által használt contoso.local tartomány felé. Amikor megnyílik a SharePoint-hely, a legfelső szintű bemutatówebhely fog megjelenni.

**_A referenciaarchitektúra készítésében közreműködtek:_** &mdash; Joe Davies, Bob Fox, Neil Hodgkinson, Paul Stork

<!-- links -->

[availability-set]: /azure/virtual-machines/windows/manage-availability
[azure-portal]: https://portal.azure.com
[azure-ps]: /powershell/azure/overview
[azure-pricing]: https://azure.microsoft.com/pricing/calculator/
[bastion-host]: https://en.wikipedia.org/wiki/Bastion_host
[create-availability-group]: /SharePoint/administration/sharepoint-intranet-farm-in-azure-phase-5-create-the-availability-group-and-add
[connect-to-vm]: /azure/virtual-machines/windows/quick-create-portal#connect-to-virtual-machine
[github]: https://github.com/mspnp/reference-architectures
[hybrid-ra]: ../hybrid-networking/index.md
[hybrid-vpn-ra]: ../hybrid-networking/vpn.md
[load-balancer]: /azure/load-balancer/load-balancer-internal-overview
[managed-disks]: /azure/storage/storage-managed-disks-overview
[minroles]: https://technet.microsoft.com/library/mt346114(v=office.16).aspx
[nsg]: /azure/virtual-network/virtual-networks-nsg
[office-web-apps]: https://support.microsoft.com/help/3199955/office-web-apps-and-office-online-server-supportability-in-azure
[paired-regions]: /azure/best-practices-availability-paired-regions
[readme]: https://github.com/mspnp/reference-architectures/tree/master/sharepoint/sharepoint-2016
[resource-group]: /azure/azure-resource-manager/resource-group-overview
[quotas]: /azure/azure-subscription-service-limits
[sharepoint-accounts]: https://technet.microsoft.com/library/ee662513(v=office.16).aspx
[sharepoint-crawling]: https://technet.microsoft.com/library/dn535606(v=office.16).aspx
[sharepoint-dr]: https://technet.microsoft.com/library/ff628971(v=office.16).aspx
[sharepoint-hybrid]: https://aka.ms/sphybrid
[sharepoint-minrole]: https://technet.microsoft.com/library/mt743705(v=office.16).aspx
[sharepoint-ops]: https://technet.microsoft.com/library/cc262289(v=office.16).aspx
[sharepoint-reqs]: https://technet.microsoft.com/library/cc262485(v=office.16).aspx
[sharepoint-search]: https://technet.microsoft.com/library/dn342836.aspx
[sql-always-on]: /sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server
[sql-performance]: /azure/virtual-machines/windows/sql/virtual-machines-windows-sql-performance
[sql-server-capacity-planning]: https://technet.microsoft.com/library/cc298801(v=office.16).aspx
[sql-quorum]: https://technet.microsoft.com/library/cc731739(v=ws.11).aspx
[sql-sharepoint-best-practices]: https://technet.microsoft.com/library/hh292622(v=office.16).aspx
[tempdb]: /sql/relational-databases/databases/tempdb-database
[virtual-networks-nsg]: /azure/virtual-network/virtual-networks-nsg
[visio-download]: https://archcenter.blob.core.windows.net/cdn/Sharepoint-2016.vsdx
[vm-sizes-general]: /azure/virtual-machines/windows/sizes-general
[vm-sizes-memory]: /azure/virtual-machines/windows/sizes-memory
[windows-n-tier]: ../virtual-machines-windows/n-tier.md
