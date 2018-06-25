---
title: SQL Servert használó N szintű alkalmazás
description: Hogyan kell beállítani egy többrétegű architektúra Azure, a rendelkezésre állási, a biztonság, a méretezhetőség és kezelhetőség.
author: MikeWasson
ms.date: 06/23/2018
ms.openlocfilehash: 050ea9b3104a2dc9af4cdaad3b4540cd75434e9d
ms.sourcegitcommit: 767c8570d7ab85551c2686c095b39a56d813664b
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 06/24/2018
ms.locfileid: "36746672"
---
# <a name="n-tier-application-with-sql-server"></a>SQL Servert használó N szintű alkalmazás

A referencia-architektúrában bemutatja, hogyan telepítheti a virtuális gépek és az N szintű alkalmazáshoz, SQL Server használata Windows rendszeren az adatréteg egy virtuális hálózatot. [**A megoldás üzembe helyezése**.](#deploy-the-solution) 

![[0]][0]

*Töltse le az architektúra [Visio-fájlját][visio-download].*

## <a name="architecture"></a>Architektúra 

Az architektúra a következő összetevőkből áll:

* **Erőforráscsoport.** Az [erőforráscsoportok][resource-manager-overview] az erőforrások csoportosítására használhatók, így élettartamuk, tulajdonosuk vagy egyéb jellemzőik alapján kezelhetők.

* **Virtuális hálózathoz (VNet) és alhálózatok.** Az Azure-ban minden virtuális gép egy alhálózatokra osztható virtuális hálózatban van üzembe helyezve. Hozzon létre egy külön alhálózatot minden egyes szinthez. 

* **NSG-k**. Használjon [hálózati biztonsági csoportokat][nsg] (NSG-ket) a hálózati forgalom korlátozására a virtuális hálózaton belül. Az itt látható 3 szintű architektúrában például az adatbázisszint csak az üzleti szintről és a felügyeleti alhálózatról érkező forgalmat fogadja, a webes kezelőfelület felől érkező forgalmat nem.

* **Virtuális gépek**. Javaslatok a virtuális gépek konfigurálása, lásd: [futtassa egy Windows virtuális gép Azure](./windows-vm.md) és [a Azure Linux virtuális Gépet futtató](./linux-vm.md).

* **Rendelkezésre állási csoportok.** Hozzon létre egy [rendelkezésre állási csoportot][azure-availability-sets] minden szinthez, majd minden szinten építsen ki legalább két virtuális gépet. Így a virtuális gépek magasabb szintű [szolgáltatói szerződésre (SLA-ra)][vm-sla] jogosultak. 

* **Virtuálisgép-méretezési készlet** (nincs ábrázolva). A [Virtuálisgép-méretezési készlet] [ vmss] van a rendelkezésre állási csoportok használata helyett. A skálázási készletekben teszi megkönnyítik a réteg a virtuális gépek bővíteni, vagy manuálisan vagy automatikusan előre meghatározott szabályok alapján.

* **Az Azure Load Balancer terheléselosztók.** A [terheléselosztók] [ load-balancer] terjesztése a Virtuálisgép-példányok bejövő Internet kéréseket. Használja a [nyilvános terheléselosztó] [ load-balancer-external] bejövő internetes forgalmat a webes réteg terjesztése és egy [belső terheléselosztó] [ load-balancer-internal] való az üzleti szint a webes réteg érkező hálózati forgalom terjesztése.

* **Nyilvános IP-cím**. Nyilvános IP-címnek a nyilvános terheléselosztót internetes forgalom fogadására van szükség.

* **Jumpbox.** Más néven [bástyagazdagép]. A hálózaton található biztonságos virtuális gép, amelyet a rendszergazdák a többi virtuális géphez való kapcsolódásra használnak. A jumpbox olyan NSG-vel rendelkezik, amely csak a biztonságos elemek listáján szereplő nyilvános IP-címekről érkező távoli forgalmat engedélyezi. Az NSG-nek engedélyeznie kell a távoli asztali (RDP) forgalmat.

* **SQL Server Always On rendelkezésre állási csoport.** Magas rendelkezésre állást biztosít az adatszinten a replikáció és a feladatátvétel engedélyezésével. Feladatátvétel a Windows Server feladatátvételi fürt (WSFC) technológiát használ.

* **(AD DS) Active Directory Domain Services-kiszolgálók** A feladatátvevő fürt és a kapcsolódó fürtözött szerepkörök számítógép-objektumokat az Active Directory tartományi szolgáltatások (AD DS) jönnek létre.

* **A felhő tanúsító**. A feladatátvevő fürt csomópontjainak futnia kell, hogy kvórum néven ismert több mint fele szükséges. Ha a fürt csak két csomóponttal rendelkezik, a hálózati partíció okozhat minden csomópont gondolja, hogy a fő csomópont. Ebben az esetben van szüksége egy *tanúsító* bontsa ki, és létrehozza a kvórum. Tanúsító egy olyan erőforrás, például olyan megosztott lemezzel, amelyet a működhet, és egy holtversenyben levő kvórum létrehozásához. Felhő tanúsító tanúsító által használt Azure Blob Storage típusú. A kvórum fogalmát kapcsolatos további információkért lásd: [ismertetése fürt és a készlet kvórum](/windows-server/storage/storage-spaces/understand-quorum). Felhő tanúsító kapcsolatos további információkért lásd: [felhő tanúsító feladatátvevő fürt telepítése](/windows-server/failover-clustering/deploy-cloud-witness). 

* **Azure DNS**. Az [Azure DNS][azure-dns] egy üzemeltetési szolgáltatás, amely a Microsoft Azure infrastruktúráját használja a DNS-tartományok névfeloldásához. Ha tartományait az Azure-ban üzemelteti, DNS-rekordjait a többi Azure-szolgáltatáshoz is használt hitelesítő adatokkal, API-kkal, eszközökkel és számlázási információkkal kezelheti.

## <a name="recommendations"></a>Javaslatok

Az Ön követelményei eltérhetnek az itt leírt architektúrától. Ezeket a javaslatokat tekintse kiindulópontnak. 

### <a name="vnet--subnets"></a>Virtuális hálózat/alhálózatok

A virtuális hálózat létrehozásakor határozza meg az egyes alhálózatok erőforrásai által igényelt IP-címek számát. [CIDR] jelöléssel adjon meg egy alhálózati maszkot és egy virtuális hálózati címtartományt, amely elég nagy a szükséges IP-címekhez. Használjon a szabványos [magánhálózati IP-címblokkokba][private-ip-space] eső címterületet. Ezek az IP-címblokkok a következők: 10.0.0.0/8, 172.16.0.0/12 és 192.168.0.0/16.

Válasszon olyan címtartományt, amely nincs átfedésben a helyszíni hálózattal, arra az esetre, ha később átjárót kell beállítania a virtuális hálózat és a helyszíni hálózat között. A virtuális hálózat létrehozása után a címtartomány nem módosítható.

Az alhálózatokat a funkciók és a biztonsági követelmények szem előtt tartásával tervezze meg. Minden ugyanazon szinthez vagy szerepkörhöz tartozó virtuális gépnek egyazon alhálózaton kell lennie. Ez az alhálózat biztonsági korlátot is képezhet. A virtuális hálózatok és alhálózatok megtervezésével kapcsolatos további információkért lásd az [Azure-beli virtuális hálózatok tervezésével és kialakításával][plan-network] foglalkozó témakört.

### <a name="load-balancers"></a>Terheléselosztók

Ne engedélyezze a virtuális gépek elérését közvetlenül az internetről – ehelyett adjon mindegyiknek privát IP-címet. Az IP-cím nyilvános terheléselosztó használatával az ügyfelek kapcsolódnak.

Adja meg a terheléselosztó a virtuális gépek felé irányuló közvetlen hálózati forgalomra vonatkozó szabályait. Például a HTTP-forgalom engedélyezéséhez hozzon létre egy szabályt, amely hozzárendeli a 80-as portot az előtérbeli konfigurációból a háttércímkészletben található 80-as porthoz. Amikor egy ügyfél HTTP-kérelmet küld a 80-as port felé, a terheléselosztó kiválaszt egy háttérbeli IP-címet egy [kivonatoló algoritmus][load-balancer-hashing] használatával, amely tartalmazza a forrás IP-címét. Így a kérelmek megoszlanak az összes virtuális gép között.

### <a name="network-security-groups"></a>Network security groups (Hálózati biztonsági csoportok)

A szintek közötti forgalmat NSG-szabályokkal korlátozhatja. A fenti 3 szintes architektúra esetén például a webes szint nem kommunikál közvetlenül az adatbázisszinttel. Ennek kényszerítése érdekében az adatbázisszintnek blokkolnia kell a webes szint alhálózatáról érkező bejövő forgalmat.  

1. A virtuális hálózat megtagadása az összes bejövő forgalmat. (A szabályban használja a `VIRTUAL_NETWORK` címkét.) 
2. Az üzleti szint alhálózatból bejövő forgalom engedélyezése.  
3. Maga az adatbázis réteg alhálózat érkező bejövő forgalom engedélyezése. Ez a szabály lehetővé teszi, hogy az adatbázis virtuális gépeket, szükség van az adatbázis-replikációt és feladatátvételt közötti kommunikációt.
4. A jumpbox alhálózatból (3389-es port) RDP-forgalmát engedélyezi. Ez lehetővé teszi, hogy a rendszergazdák csatlakozni tudjanak az adatbázisszinthez a jumpboxból.

Létrehozhat szabályokat 2 &ndash; magasabb prioritású, mint az első szabály, akkor bírálja felül, a 4.


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

Az alkalmazás számítási feladatait futtató virtuális gépek nyilvános internetről való RDP-hozzáférését ne engedélyezze. Az ilyen virtuális gépek RDP-hozzáférésének ehelyett a jumpboxon keresztül kell történnie. A rendszergazda először bejelentkezik a jumpboxba, majd azon keresztül bejelentkezik a többi virtuális gépbe. A jumpbox engedélyezi az internetről érkező RDP-forgalmat, de csak az ismert, biztonságos IP-címekről.

A jumpbox minimális teljesítmény követelményekkel rendelkezik, ezért válassza ki a kisméretű Virtuálisgép-méretet. Hozzon létre egy [nyilvános IP-cím] a jumpbox számára. Helyezze a jumpboxot a többi virtuális géppel megegyező virtuális hálózatba, de egy külön felügyeleti alhálózaton legyen.

A jumpbox védelme érdekében vegyen fel egy NSG-t, amely lehetővé teszi az RDP-kapcsolatok csak egy készletből biztonságos nyilvános IP-címek. Állítsa be az NSG-t a többi alhálózathoz is úgy, hogy engedélyezzék a felügyeleti alhálózatból érkező RDP-forgalmat.

## <a name="scalability-considerations"></a>Méretezési szempontok

[Virtuálisgép-méretezési készlet] [ vmss] szeretné telepíteni, és az azonos virtuális gépek kezelésére. A méretezési csoportok támogatják a teljesítménymetrikák alapján történő automatikus skálázást. Ahogy a terhelés növekszik a virtuális gépeken, a rendszer további virtuális gépeket ad a terheléselosztóhoz. Fontolja meg a méretezési csoportok használatát, ha virtuális gépek gyors horizontális felskálázására vagy automatikus méretezésre van szüksége.

A méretezési csoportokban üzembe helyezett virtuális gépek konfigurálásának két alapvető módja van:

- Bővítmények segítségével konfigurálhatja a virtuális gépet annak kiépítése után. Ezzel a módszerrel az új virtuálisgép-példányok indítása több időt vehet igénybe, mint a bővítmények nélküli virtuális gépeké.

- Üzembe helyezhet egy [felügyelt lemezt](/azure/storage/storage-managed-disks-overview) egy egyéni rendszerképpel. Ez a módszer gyorsabban kivitelezhető. Azonban ebben az esetben naprakészen kel tartania a rendszerképet.

További szempontokért lásd: [Kialakítási szempontok a méretezési csoportokhoz][vmss-design].

> [!TIP]
> Bármilyen automatikus méretezési megoldás használata esetén jóval előre tesztelje azt az éles környezetre jellemző mennyiségű számítási feladattal.

Minden Azure-előfizetésre alapértelmezett korlátozások vonatkoznak. Ilyen például a régiónként alkalmazható virtuális gépek maximális száma. Támogatási kérelem kitöltésével megnövelheti ezt a korlátot. További információk: [Az Azure-előfizetések és -szolgáltatások korlátozásai, kvótái és megkötései][subscription-limits].

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

Ha nem használ Virtuálisgép-méretezési készlet azonos tartozó virtuális gépek üzembe egy rendelkezésre állási csoportot. Hozzon létre legalább két virtuális gépet a rendelkezésre állási csoportban az [Azure-beli virtuális gépekre vonatkozó rendelkezésre állási SLA][vm-sla] támogatásához. További információk: [Virtuális gépek rendelkezésre állásának kezelése][availability-set]. 

A terheléselosztó [állapotmintákat][health-probes] használ a virtuálisgép-példányok rendelkezésre állásának monitorozásához. Ha a mintavétel nem ér el egy példányt egy bizonyos időkorláton belül, a terheléselosztó nem irányít több forgalmat az adott virtuális gép felé. A terheléselosztó ezután is folytatja a mintavételezést, és amint a virtuális gép újra elérhetővé válik, a terheléselosztó ismét elkezd forgalmat irányítani felé.

Az alábbiakban néhány javaslat olvasható a terheléselosztó állapot-mintavételére vonatkozóan:

* A mintavételek a HTTP-t vagy a TCP-t tesztelhetik. Ha a virtuális gépek HTTP-kiszolgálót futtatnak, HTTP-mintavételt hozzon létre. Egyéb esetben TCP-mintavételt hozzon létre.
* A HTTP-mintavétel esetében adjon meg elérési utat egy HTTP-végponthoz. A mintavétel 200-as HTTP-választ vár erről a megadott elérési útról. Ez lehet a gyökérútvonal („/”) vagy egy állapotmonitorozó végpont, amely valamilyen egyéni logikát alkalmaz az alkalmazás állapotának ellenőrzésére. A végpontnak engedélyeznie kell a névtelen HTTP-kérelmeket.
* A mintavétel [ismert IP-címről érkezik:][health-probe-ip] 168.63.129.16. Győződjön meg arról, hogy nem tiltja a bejövő és kimenő forgalmat erről az IP-címről valamelyik tűzfalszabályzatban vagy NSG-szabályban.
* Az állapotminták állapotának megtekintéséhez használjon [állapotminta-naplókat][health-probe-log]. Engedélyezze a naplózást az Azure Portalon minden egyes terheléselosztóra vonatkozóan. A naplókat a rendszer az Azure Blob Storage-ba írja. A naplók megmutatják, hány háttérbeli virtuális gép nem fogad hálózati forgalmat a meghiúsult mintavételi válaszok miatt.

Ha magas rendelkezésre állás érdekében, mint a [Azure SLA-t a virtuális gépek] [ vm-sla] biztosít, érdemes replikációs az alkalmazás két régióban Azure Traffic Manager feladatátvételhez. További információkért lásd: [több területi N szintű alkalmazás a magas rendelkezésre állású][multi-dc].  

## <a name="security-considerations"></a>Biztonsági szempontok

A virtuális hálózatok forgalomelkülönítési határok az Azure-ban. Egy adott virtuális hálózatban található virtuális gépek nem képesek közvetlenül kommunikálni egy másik virtuális hálózat gépeivel. Az azonos virtuális hálózatban elhelyezkedő virtuális gépek képesek a kommunikációra, hacsak [hálózati biztonsági csoportokat] [ nsg] (NSG-ket) nem hoz létre a forgalom korlátozására. További információ: [A Microsoft felhőszolgáltatásai és hálózati biztonság][network-security].

A bejövő internetes forgalom esetében a terheléselosztó szabályai határozzák meg, milyen forgalom érheti el a háttérrendszert. A terheléselosztó szabályai azonban nem támogatják a biztonságos IP-címek listázását, ezért ha fel kíván venni bizonyos nyilvános IP-címeket a biztonságos címek listájára, adjon hálózati biztonsági csoportot az alhálózathoz.

Érdemes lehet hozzáadnia egy hálózati virtuális berendezést (network virtual appliance, NVA), hogy DMZ-t lehessen létrehozni az internet és az Azure-beli virtuális hálózat között. Az NVA egy általános kifejezés egy olyan virtuális berendezésre, amely hálózatokhoz kapcsolódó feladatokat lát el, például gondoskodik a tűzfalról, a csomagvizsgálatról, a naplózásról és az egyéni útválasztásról. További információkért lásd a [DMZ az Azure és az internet közötti implementálásával][dmz] foglalkozó témakört.

Titkosíthatja az inaktív bizalmas adatokat, és az [Azure Key Vaulttal][azure-key-vault] kezelheti az adatbázis titkosítási kulcsait. A Key Vault képes a hardveres biztonsági modulok (HSM-ek) titkosítási kulcsainak tárolására. További információkért lásd: [konfigurálása Azure Key Vault-integráció az SQL Server Azure virtuális gépeken][sql-keyvault]. Alkalmazás titkos adatait, például adatbázis-kapcsolati karakterláncok tárolni a Key Vault is javasoljuk.

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

Ennek a referenciaarchitektúrának egy üzemelő példánya elérhető a [GitHubon][github-folder]. Vegye figyelembe, hogy a teljes telepítési órát is igénybe vehet legfeljebb két, mely tartalmazza az Active Directory tartományi szolgáltatások, a Windows Server feladatátvevő fürt és az SQL Server rendelkezésre állási csoport konfigurálásához a parancsfájlokat futtasson.

### <a name="prerequisites"></a>Előfeltételek

1. Klónozza, ágaztassa vagy a zip-fájl letöltése a [architektúrák hivatkozhat] [ ref-arch-repo] GitHub-tárházban.

2. Telepítés [Azure CLI 2.0][azure-cli-2].

3. Telepítse [az Azure építőelemei][azbb] npm-csomagot.

   ```bash
   npm install -g @mspnp/azure-building-blocks
   ```

4. A parancssorból bash, vagy PowerShell kérdés, jelentkezzen be az Azure-fiókjával az alábbi parancs segítségével.

   ```bash
   az login
   ```

### <a name="deploy-the-solution"></a>A megoldás üzembe helyezése 

1. Futtassa a következő parancs futtatásával hozzon létre egy erőforráscsoportot.

    ```bash
    az group create --location <location> --name <resource-group-name>
    ```

2. A következő parancsot a felhő tanúsító hozzon létre egy tárfiókot.

    ```bash
    az storage account create --location <location> \
      --name <storage-account-name> \
      --resource-group <resource-group-name> \
      --sku Standard_LRS
    ```

3. Keresse meg a `virtual-machines\n-tier-windows` mappába a referencia architektúra GitHub tárház.

4. Nyissa meg az `n-tier-windows.json` fájlt. 

5. Keresse meg a "witnessStorageBlobEndPoint" az összes példányát, és cserélje le a helyőrzőket a 2. lépésben a tárfiók nevét.

    ```json
    "witnessStorageBlobEndPoint": "https://[replace-with-storageaccountname].blob.core.windows.net",
    ```

6. Futtassa a következő paranccsal listát készíthet a tárfiók kulcsait.

    ```bash
    az storage account keys list \
      --account-name <storage-account-name> \
      --resource-group <resource-group-name>
    ```

    A kimenet a következő hasonlóan kell kinéznie. Másolja a `key1` értékét.

    ```json
    [
    {
        "keyName": "key1",
        "permissions": "Full",
        "value": "..."
    },
    {
        "keyName": "key2",
        "permissions": "Full",
        "value": "..."
    }
    ]
    ```

7. Az a `n-tier-windows.json` fájlt, keresse meg az "witnessStorageAccountKey" összes példányát, és illessze be a fiókkulcs.

    ```json
    "witnessStorageAccountKey": "[replace-with-storagekey]"
    ```

8. Az a `n-tier-windows.json` fájlt, keresse meg az összes példányt `testPassw0rd!23`, `test$!Passw0rd111`, és `AweS0me@SQLServicePW`. Cserélje le azokat a saját jelszavát, és mentse a fájlt.

    > [!NOTE]
    > Ha módosítja a adminsztrátori felhasználónév, frissíteni kell a `extensions` blokkolja a JSON-fájlban. 

9. A következő parancsot architektúra telepítéséhez.

    ```bash
    azbb -s <your subscription_id> -g <resource_group_name> -l <location> -p n-tier-windows.json --deploy
    ```

A mintául szolgáló referenciaarchitektúra Azure-építőelemekkel történő üzembe helyezéséről további információkat a [GitHub-adattárban][git] talál.


<!-- links -->
[dmz]: ../dmz/secure-vnet-dmz.md
[multi-dc]: multi-region-sql-server.md
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
[visio-download]: https://archcenter.blob.core.windows.net/cdn/vm-reference-architectures.vsdx
[0]: ./images/n-tier-sql-server.png "N szintű architektúra a Microsoft Azure használatával"
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
