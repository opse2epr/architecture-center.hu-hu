---
title: Magas rendelkezésre állású SharePoint Server 2016-farm futtatása az Azure-ban
description: Bevált módszerek a magas rendelkezésre állású SharePoint Server 2016-farm létrehozására az Azure-ban.
author: njray
ms.date: 08/01/2017
ms.openlocfilehash: d1e3f0b73c94844ac649bf2abb6917809202fdb7
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/30/2018
---
# <a name="run-a-high-availability-sharepoint-server-2016-farm-in-azure"></a>Magas rendelkezésre állású SharePoint Server 2016-farm futtatása az Azure-ban

A jelen referenciaarchitektúra bevált módszereket mutat be a magas rendelkezésre állású SharePoint Server 2016-farmok Azure-ban való létrehozására a MinRole topológia és az SQL Server Always On rendelkezésre állási csoportok használatával. A SharePoint-farm egy biztonságos virtuális hálózaton lesz üzembe helyezve, amelynek nincs internetes végpontja vagy jelenléte. [**A megoldás üzembe helyezése**.](#deploy-the-solution) 

![](./images/sharepoint-ha.png)

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

- **Windows Server Active Directory- (AD) tartományvezérlők**. A SharePoint Server 2016 nem támogatja az Azure Active Directory Domain Services használatát, ezért Windows Server AD-tartományvezérlőket kell telepítenie. Ezek a tartományvezérlők az Azure-beli virtuális hálózaton futnak, és megbízhatósági viszonyban vannak a helyszíni Windows Server AD-erdővel. Az ügyfelek SharePoint-farmokon lévő erőforrásokkal kapcsolatos webes kéréseit a rendszer a virtuális hálózaton hitelesíti ahelyett, hogy továbbítaná a hitelesítési forgalmat az átjárókapcsolaton keresztül a helyszíni hálózatra. A DNS-ben intranet A vagy CNAME-rekordok jönnek létre, hogy az intranetes felhasználók fel tudják oldani a SharePoint-farm nevét a belső terheléselosztó magánhálózati IP-címére.

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

A Standard DSv2 virtuális gépek méretei alapján ehhez az architektúrához legalább 38 magra van szükség:

- 8 SharePoint-kiszolgáló, Standard_DS3_v2 méret (4 mag/darab) = 32 mag
- 2 Active Directory-tartományvezérlő, Standard_DS1_v2 méret (1 mag/darab) = 2 mag
- 2 SQL Servert futtató virtuális gép, Standard_DS1_v2 méret = 2 mag
- 1 csomóponttöbbség, Standard_DS1_v2 méret = 1 mag
- 1 felügyeleti kiszolgáló, Standard_DS1_v2 méret = 1 mag

A magok teljes száma a kiválasztott virtuálisgép-méretektől függ. További információt a [SharePoint Serverre vonatkozó javaslatokat](#sharepoint-server-recommendations) tartalmazó alábbi szakaszban talál.

Győződjön meg arról, hogy az Azure-előfizetésnek elegendő virtuálisgép-magkvóta áll rendelkezésére, különben az üzembe helyezés sikertelen lesz. Lásd: [Az Azure-előfizetések és -szolgáltatások korlátozásai, kvótái és megkötései][quotas]. 
 
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

A keresési indexelő kivételével az összes szerepkörhöz javasoljuk a [Standard_DS3_v2][vm-sizes-general] virtuálisgép-méret használatát. A keresési indexelőnek legalább [Standard_DS13_v2][vm-sizes-memory] méretűnek kell lennie. 

> [!NOTE]
> A referenciaarchitektúra Resource Manager-sablonja a telepítés tesztelésére a kisebb DS3 méretet használja a keresési indexelőhöz. Éles környezetek esetén használja a DS13 méretet, vagy egy annál nagyobbat. 

Az éles környezetben végzett számítási feladatokról lásd: [A SharePoint Server 2016 hardver- és szoftverkövetelményei][sharepoint-reqs]. 

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

A referenciaarchitektúra üzembehelyezési szkriptjei elérhetők a [GitHubon][github]. 

Az architektúra növekményesen vagy egyszerre is telepíthető. Első alkalommal a növekményes üzembe helyezést javasoljuk, hogy megismerhesse, melyik üzembehelyezési lépés mire való. A növekmény az alábbi *mód* paraméterek egyikével határozható meg.

| Mód           | Művelet                                                                                                            |
|----------------|-------------------------------------------------------------------------------------------------------------------------|
| onprem         | (Nem kötelező) Üzembe helyez egy szimulált helyszíni hálózati környezetet tesztelési és értékelési célokra. Ez a lépés nem csatlakozik tényleges helyszíni hálózathoz. |
| infrastruktúra | Üzembe helyezi a SharePoint 2016 hálózati infrastruktúrát és a jumpboxot az Azure-ban.                                                |
| createvpn      | Üzembe helyez egy-egy virtuális hálózati átjárót a SharePoint- és a helyszíni hálózaton, és csatlakoztatja őket. Ezt a lépést csak akkor futtassa, ha futtatta az `onprem` lépést.                |
| számítási feladat       | Üzembe helyezi a SharePoint-kiszolgálókat a SharePoint-hálózaton.                                                               |
| biztonság       | Üzembe helyezi a hálózati biztonsági csoportot a SharePoint-hálózaton.                                                           |
| összes            | Üzembe helyezi az összes fenti környezetet.                            


Az architektúra szimulált helyszíni hálózati környezettel történő növekményes üzembe helyezéséhez futtassa az alábbi lépéseket ebben a sorrendben:

1. onprem
2. infrastruktúra
3. createvpn
4. számítási feladat
5. biztonság

Az architektúra szimulált helyszíni hálózati környezet nélkül történő növekményes üzembe helyezéséhez futtassa az alábbi lépéseket ebben a sorrendben:

1. infrastruktúra
2. számítási feladat
3. biztonság

Ha mindent egy lépésben szeretne üzembe helyezni, használja az `all` paramétert. Vegye figyelembe, hogy a teljes folyamat több órát is igénybe vehet.

### <a name="prerequisites"></a>Előfeltételek

* Telepítse az [Azure PowerShell][azure-ps] legújabb verzióját.

* A referenciaarchitektúra üzembe helyezése előtt ellenőrizze, hogy az előfizetése elegendő kvótával rendelkezik-e (legalább 38 mag). Ha nincs elég mag, küldjön támogatási kérelmet az Azure Portalról további kvótáért.

* Az üzembe helyezés költségeinek meghatározásához lásd: [Azure-díjkalkulátor][azure-pricing].

### <a name="deploy-the-reference-architecture"></a>A referenciaarchitektúra üzembe helyezése

1.  Töltse le vagy klónozza a [GitHub-adattárat][github] a saját helyi számítógépére.

2.  Nyisson meg egy PowerShell-ablakot, és lépjen a `/sharepoint/sharepoint-2016` mappára.

3.  Futtassa az alábbi PowerShell-parancsot. A \<subscription id\> paraméterhez adja meg az Azure-előfizetése azonosítóját. A \<location\> paraméterhez adjon meg Azure-régiót (például `eastus` vagy `westus`). A \<mode\> paraméterhez adja meg az `onprem`, `infrastructure`, `createvpn`, `workload`, `security` vagy az `all` értéket.

    ```powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> <mode>
    ```   
4. Ha a rendszer kéri, jelentkezzen be az Azure-fiókjába. A kiválasztott módtól függően az üzembehelyezési szkriptek befejezése több órát is igénybe vehet.

5. Ha az üzembe helyezés befejeződik, futtassa a szkripteket az SQL Server Always On rendelkezésre állási csoportok konfigurálásához. További részletekért tekintse meg az [információs fájlt][readme].

> [!WARNING]
> A paraméterfájlok több helyen nem módosítható jelszót (`AweS0me@PW`) tartalmaznak. Módosítsa ezeket az értékeket az üzembe helyezés előtt.


## <a name="validate-the-deployment"></a>Az üzembe helyezés ellenőrzése

A referenciaarchitektúra üzembe helyezése után az alábbi erőforráscsoportok fognak megjelenni a használt előfizetés alatt:

| Erőforráscsoport        | Cél                                                                                         |
|-----------------------|-------------------------------------------------------------------------------------------------|
| ra-onprem-sp2016-rg   | Szimulált helyszíni hálózat Active Directoryval, a SharePoint 2016-hálózattal összevonva |
| ra-sp2016-network-rg  | A SharePoint-környezet támogatására szolgáló infrastruktúra                                                 |
| ra-sp2016-workload-rg | A SharePoint és a támogató erőforrások                                                             |

### <a name="validate-access-to-the-sharepoint-site-from-the-on-premises-network"></a>A SharePoint-hely helyszíni hálózatról való elérésének ellenőrzése

1. Az [Azure Portalon][azure-portal] **Erőforráscsoportok** területén válassza ki a `ra-onprem-sp2016-rg` erőforráscsoportot.

2. Az erőforrások listájában válassza ki a `ra-adds-user-vm1` nevű virtuálisgép-erőforrást. 

3. Csatlakozzon a virtuális géphez a [Csatlakozás virtuális géphez][connect-to-vm] című részben ismertetett módon. Felhasználónév: `\onpremuser`.

5.  Ha létrejött a távoli kapcsolat a virtuális géppel, nyisson meg egy böngészőt a virtuális gépen, és navigáljon a következő helyre: `http://portal.contoso.local`.

6.  A **Windows rendszerbiztonság** mezőben jelentkezzen be a SharePoint portálra a `contoso.local\testuser` felhasználónévvel.

Ez a bejelentkezés a helyszíni hálózat által használt Fabrikam.com tartományból nyit alagutat a SharePoint portál által használt contoso.local tartomány felé. Amikor megnyílik a SharePoint-hely, a legfelső szintű bemutatówebhely fog megjelenni.

### <a name="validate-jumpbox-access-to-vms-and-check-configuration-settings"></a>A jumpbox virtuális gépekhez való hozzáférésének ellenőrzése és a konfigurációs beállítások áttekintése

1.  Az [Azure Portalon][azure-portal] **Erőforráscsoportok** területén válassza ki a `ra-sp2016-network-rg` erőforráscsoportot.

2.  Az erőforrások listájában válassza ki a `ra-sp2016-jb-vm1` nevű virtuálisgép-erőforrást, azaz a jumpboxot.

3. Csatlakozzon a virtuális géphez a [Csatlakozás virtuális géphez][connect-to-vm] című részben ismertetett módon. Felhasználónév: `testuser`.

4.  Miután bejelentkezett a jumpboxba, nyisson meg onnan egy RDP-munkamenetet. Csatlakozzon a virtuális hálózaton található bármely másik virtuális géphez. Felhasználónév: `testuser`. A távoli számítógép biztonsági tanúsítványára vonatkozó figyelmeztetést figyelmen kívül hagyhatja.

5.  Amikor megnyílik a virtuális géppel létesített távoli kapcsolat, tekintse át a konfigurációt, és végezzen módosításokat a felügyeleti eszközökkel, például a Kiszolgálókezelővel.

Az alábbi táblázatban az üzembe helyezett virtuális gépek láthatók. 

| Erőforrás neve      | Cél                                   | Erőforráscsoport        | Virtuális gép neve                       |
|--------------------|-------------------------------------------|-----------------------|-------------------------------|
| Ra-sp2016-ad-vm1   | Active Directory + DNS                    | Ra-sp2016-network-rg  | Ad1.contoso.local             |
| Ra-sp2016-ad-vm2   | Active Directory + DNS                    | Ra-sp2016-network-rg  | Ad2.contoso.local             |
| Ra-sp2016-fsw-vm1  | SharePoint                                | Ra-sp2016-network-rg  | Fsw1.contoso.local            |
| Ra-sp2016-jb-vm1   | Jumpbox                                   | Ra-sp2016-network-rg  | Jb (a bejelentkezéshez használja a nyilvános IP-címet) |
| Ra-sp2016-sql-vm1  | SQL Always On – Feladatátvétel                  | Ra-sp2016-network-rg  | Sq1.contoso.local             |
| Ra-sp2016-sql-vm2  | SQL Always On – Elsődleges                   | Ra-sp2016-network-rg  | Sq2.contoso.local             |
| Ra-sp2016-app-vm1  | SharePoint 2016 alkalmazás MinRole       | Ra-sp2016-workload-rg | App1.contoso.local            |
| Ra-sp2016-app-vm2  | SharePoint 2016 alkalmazás MinRole       | Ra-sp2016-workload-rg | App2.contoso.local            |
| Ra-sp2016-dch-vm1  | SharePoint 2016 elosztott gyorsítótárazás MinRole | Ra-sp2016-workload-rg | Dch1.contoso.local            |
| Ra-sp2016-dch-vm2  | SharePoint 2016 elosztott gyorsítótárazás MinRole | Ra-sp2016-workload-rg | Dch2.contoso.local            |
| Ra-sp2016-srch-vm1 | SharePoint 2016 keresés MinRole            | Ra-sp2016-workload-rg | Srch1.contoso.local           |
| Ra-sp2016-srch-vm2 | SharePoint 2016 keresés MinRole            | Ra-sp2016-workload-rg | Srch2.contoso.local           |
| Ra-sp2016-wfe-vm1  | SharePoint 2016 webes kezelőfelület MinRole     | Ra-sp2016-workload-rg | Wfe1.contoso.local            |
| Ra-sp2016-wfe-vm2  | SharePoint 2016 webes kezelőfelület MinRole     | Ra-sp2016-workload-rg | Wfe2.contoso.local            |


**_A referenciaarchitektúra készítésében közreműködtek:_** &mdash;  Joe Davies, Bob Fox, Neil Hodgkinson, Paul Stork

<!-- links -->

[availability-set]: /azure/virtual-machines/windows/manage-availability
[azure-portal]: https://portal.azure.com
[azure-ps]: /powershell/azure/overview
[azure-pricing]: https://azure.microsoft.com/pricing/calculator/
[bastion-host]: https://en.wikipedia.org/wiki/Bastion_host
[create-availability-group]: https://technet.microsoft.com/library/mt793548(v=office.16).aspx
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
[sql-performance]: /virtual-machines/windows/sql/virtual-machines-windows-sql-performance
[sql-server-capacity-planning]: https://technet.microsoft.com/library/cc298801(v=office.16).aspx
[sql-quorum]: https://technet.microsoft.com/library/cc731739(v=ws.11).aspx
[sql-sharepoint-best-practices]: https://technet.microsoft.com/library/hh292622(v=office.16).aspx
[tempdb]: /sql/relational-databases/databases/tempdb-database
[virtual-networks-nsg]: /azure/virtual-network/virtual-networks-nsg
[visio-download]: https://archcenter.blob.core.windows.net/cdn/Sharepoint-2016.vsdx
[vm-sizes-general]: /azure/virtual-machines/windows/sizes-general
[vm-sizes-memory]: /azure/virtual-machines/windows/sizes-memory
[windows-n-tier]: ../virtual-machines-windows/n-tier.md
