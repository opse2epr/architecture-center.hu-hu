---
title: SQL Servert használó N szintű alkalmazás
description: Egy többrétegű architektúra megvalósítása az Azure-ban, a rendelkezésre állás, biztonság, skálázhatósággal és kezelhetőséggel módja.
author: MikeWasson
ms.date: 11/12/2018
ms.openlocfilehash: 857b666ef8af8fec21d7a8a9756508344aa07acc
ms.sourcegitcommit: 9293350ab66fb5ed042ff363f7a76603bf68f568
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/13/2018
ms.locfileid: "51577123"
---
# <a name="windows-n-tier-application-on-azure-with-sql-server"></a>Az Azure-ban az SQL Server Windows N szintű alkalmazás

Ez a referenciaarchitektúra bemutatja, hogyan helyezhet üzembe a virtuális gépek és a egy N szintű alkalmazáshoz, a Windows SQL Server használata az adatréteg számára konfigurált virtuális hálózatot. [**A megoldás üzembe helyezése**.](#deploy-the-solution) 

![[0]][0]

*Töltse le az architektúra [Visio-fájlját][visio-download].*

## <a name="architecture"></a>Architektúra 

Az architektúra a következő összetevőkből áll:

* **Erőforráscsoport.** Az [erőforráscsoportok][resource-manager-overview] az erőforrások csoportosítására használhatók, így élettartamuk, tulajdonosuk vagy egyéb jellemzőik alapján kezelhetők.

* **Virtuális hálózat (VNet) és alhálózatok.** Minden Azure VM egy alhálózatokra osztható virtuális hálózatban van üzembe helyezve. Hozzon létre egy külön alhálózatot minden egyes szinthez. 

* **Az Application gateway**. [Az Azure Application Gateway](/azure/application-gateway/) egy 7. rétegbeli terheléselosztó. Ebben az architektúrában, a web front end irányítja a HTTP-kérelmekre. Az Application Gateway is biztosít egy [webalkalmazási tűzfal](/azure/application-gateway/waf-overview) (WAF) az alkalmazás, amely védi a gyakori támadások és biztonsági rések. 

* **NSG-k**. Használjon [hálózati biztonsági csoportokat][nsg] (NSG-ket) a hálózati forgalom korlátozására a virtuális hálózaton belül. Például a háromrétegű architektúra itt látható, az adatbázisszinten nem fogadja el a web front end, csak az üzleti szint és a felügyeleti alhálózatról érkező forgalmat.

* **A DDoS Protection**. Bár az Azure platform lehetővé teszi, hogy alapvető védelmet az elosztott szolgáltatásmegtagadási (DDoS-) támadások ellen, javasoljuk a [DDoS Protection Standard][ddos], amely továbbfejlesztett DDoS-támadás kockázatcsökkentése szolgáltatások. Lásd: [biztonsági szempontok](#security-considerations).

* **Virtuális gépek**. Javaslatok a virtuális gépek konfigurálása, lásd: [Windows virtuális gépek futtatása az Azure-beli](./windows-vm.md) és [Linux rendszerű virtuális gép futtatása az Azure-ban](./linux-vm.md).

* **Rendelkezésre állási csoportok.** Hozzon létre egy [rendelkezésre állási csoport] [ azure-availability-sets] mindegyik réteghez, és az egyes szintek, amelyek a virtuális gépek így magasabb szintű támogatásra jogosult legalább két virtuális gép kiépítése [szolgáltatói szerződést (SLA)] [ vm-sla].

* **Terheléselosztók**. Használat [Azure Load Balancer] [ load-balancer] a webes szintről az üzleti szint és az üzleti szint az SQL-kiszolgálóra irányuló hálózati forgalom elosztására.

* **Nyilvános IP-cím**. Az alkalmazás internetes forgalmat fogadó nyilvános IP-cím van szükség.

* **Jumpbox.** Más néven [bástyagazdagép]. A hálózaton található biztonságos virtuális gép, amelyet a rendszergazdák a többi virtuális géphez való kapcsolódásra használnak. A jumpbox olyan NSG-vel rendelkezik, amely csak a biztonságos elemek listáján szereplő nyilvános IP-címekről érkező távoli forgalmat engedélyezi. Az NSG-nek engedélyeznie kell a távoli asztali (RDP) forgalmat.

* **SQL Server Always On rendelkezésre állási csoport.** Magas rendelkezésre állást biztosít az adatszinten a replikáció és a feladatátvétel engedélyezésével. Windows Server feladatátvételi fürt (WSFC) technológiát használ a feladatátvételhez.

* **(AD DS) Active Directory Domain Services-kiszolgálók** A feladatátvevő fürt és a kapcsolódó fürtözött szerepkörök számítógép-objektumai jönnek létre az Active Directory Domain Servicesben (AD DS).

* **Felhőbeli tanúsító**. A feladatátvevő fürtök csomópontjainak futnia kell, amelyről ismert, hogy a kvórum több mint felét van szükség. Ha a fürt csak két csomópontot tartalmaz, egy hálózati partíció okozhat, minden csomóponthoz gondolja, hogy a fő csomóponttal. Ebben az esetben van szüksége egy *tanúsító* ties felosztása és kvórumot hozzon létre. Tanúsító például a megosztott lemezzel működő, egy tie megszakító kvórumot hozzon létre egy erőforrást. Felhőbeli tanúsító egy olyan típusú, amelyet az Azure Blob Storage. A kvórum fogalmát kapcsolatos további információkért lásd: [ismertetése fürt és a készlet kvórum](/windows-server/storage/storage-spaces/understand-quorum). További információ a Felhőbeli tanúsító: [Felhőbeli tanúsító telepítése feladatátvevő fürt](/windows-server/failover-clustering/deploy-cloud-witness). 

* **Azure DNS**. [Az Azure DNS] [ azure-dns] DNS-tartományok egy üzemeltetési szolgáltatás. A Microsoft Azure infrastruktúráját használja a névfeloldást biztosít. Ha tartományait az Azure-ban üzemelteti, DNS-rekordjait a többi Azure-szolgáltatáshoz is használt hitelesítő adatokkal, API-kkal, eszközökkel és számlázási információkkal kezelheti.

## <a name="recommendations"></a>Javaslatok

Az Ön követelményei eltérhetnek az itt leírt architektúrától. Ezeket a javaslatokat tekintse kiindulópontnak. 

### <a name="vnet--subnets"></a>Virtuális hálózat/alhálózatok

A virtuális hálózat létrehozásakor határozza meg az egyes alhálózatok erőforrásai által igényelt IP-címek számát. [CIDR] jelöléssel adjon meg egy alhálózati maszkot és egy virtuális hálózati címtartományt, amely elég nagy a szükséges IP-címekhez. Használjon a szabványos [magánhálózati IP-címblokkokba][private-ip-space] eső címterületet. Ezek az IP-címblokkok a következők: 10.0.0.0/8, 172.16.0.0/12 és 192.168.0.0/16.

Válasszon olyan címtartományt, amely nincs átfedésben a helyszíni hálózattal, arra az esetre, ha később átjárót kell beállítania a virtuális hálózat és a helyszíni hálózat között. A virtuális hálózat létrehozása után a címtartomány nem módosítható.

Az alhálózatokat a funkciók és a biztonsági követelmények szem előtt tartásával tervezze meg. Minden ugyanazon szinthez vagy szerepkörhöz tartozó virtuális gépnek egyazon alhálózaton kell lennie. Ez az alhálózat biztonsági korlátot is képezhet. A virtuális hálózatok és alhálózatok megtervezésével kapcsolatos további információkért lásd az [Azure-beli virtuális hálózatok tervezésével és kialakításával][plan-network] foglalkozó témakört.

### <a name="load-balancers"></a>Terheléselosztók

Nem teszik elérhetővé a virtuális gépek közvetlenül az internethez, de ehelyett adjon mindegyiknek privát IP-címet. Az Application Gateway társított nyilvános IP-cím használatával az ügyfelek csatlakoznak.

Adja meg a terheléselosztó a virtuális gépek felé irányuló közvetlen hálózati forgalomra vonatkozó szabályait. Például HTTP-forgalom engedélyezéséhez képezze le a 80-as porton az előtérbeli konfigurációból a háttér-címkészletet a 80-as porton. Amikor egy ügyfél HTTP-kérelmet küld a 80-as port felé, a terheléselosztó kiválaszt egy háttérbeli IP-címet egy [kivonatoló algoritmus][load-balancer-hashing] használatával, amely tartalmazza a forrás IP-címét. Kérelmek megoszlanak a háttér-címkészletben levő összes virtuális gép között.

### <a name="network-security-groups"></a>Network security groups (Hálózati biztonsági csoportok)

A szintek közötti forgalmat NSG-szabályokkal korlátozhatja. A fenti három rétegű architektúrában a webes szint nem kommunikál közvetlenül az adatbázisszint. Ennek kényszerítése érdekében az adatbázisszintnek blokkolnia kell a webes szint alhálózatáról érkező bejövő forgalmat.  

1. Megtagadási az összes bejövő forgalmat a virtuális hálózatról. (A szabályban használja a `VIRTUAL_NETWORK` címkét.) 
2. Az üzleti szint alhálózatáról érkező forgalom engedélyezéséhez.  
3. Az adatbázisszint alhálózatán érkező forgalom engedélyezéséhez. Ez a szabály lehetővé teszi, hogy szükség van az adatbázis-replikáció és feladatátvételi adatbázis-beli virtuális gépek közötti kommunikációt.
4. Engedélyezze az RDP-forgalom (3389-es port) a jumpbox alhálózatáról származó. Ez lehetővé teszi, hogy a rendszergazdák csatlakozni tudjanak az adatbázisszinthez a jumpboxból.

2-szabályok létrehozása &ndash; magasabb prioritású, mint az első szabály, így azt felülbírálják a 4.


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

Ha egy SQL-ügyfél csatlakozni próbál, a terheléselosztó továbbítja az elsődleges replikának a kapcsolódási kérelmet. Ha feladatátvételt egy másik replikára, a terheléselosztó automatikusan egy új elsődleges replikára irányítja a kérelmeket új. További információ: [ILB-figyelő konfigurálása SQL Server Always On rendelkezésre állási csoportokhoz][sql-alwayson-ilb].

A feladatátvétel során a meglévő ügyfélkapcsolatok lezárulnak. A feladatátvétel befejezése után a rendszer az új elsődleges replikára irányítja az új kapcsolatokat.

Ha az alkalmazás olvasási műveleteinek száma lényegesen több, mint az írási műveleteteké, a csak olvasási lekérdezések egy részét kiszervezheti egy másodlagos replikára. Lásd: [Csak olvasási másodlagos replikához való csatlakozás figyelővel (csak olvasási útválasztás)][sql-alwayson-read-only-routing].

[Kényszerítse][sql-alwayson-force-failover] a rendelkezésre állási csoport manuális feladatátvételét az üzemelő példány teszteléséhez.

### <a name="jumpbox"></a>Jumpbox

Tiltása RDP-hozzáférés a nyilvános internetről való az alkalmazás számítási feladatait futtató virtuális gépek. Az ilyen virtuális gépek RDP-hozzáférésének ehelyett a jumpboxon keresztül kell történnie. A rendszergazda először bejelentkezik a jumpboxba, majd azon keresztül bejelentkezik a többi virtuális gépbe. A jumpbox engedélyezi az internetről érkező RDP-forgalmat, de csak az ismert, biztonságos IP-címekről.

A jumpbox rendelkezik a minimális teljesítménykövetelményei, ezért kattintson egy kisméretű Virtuálisgép-méretet. Hozzon létre egy [nyilvános IP-címet] a jumpbox számára. Helyezze a jumpboxot a többi virtuális géppel megegyező virtuális hálózatba, de egy külön felügyeleti alhálózaton legyen.

A jumpbox védelmére, adja hozzá egy NSG-szabályt, amely RDP-kapcsolatok kizárólag a nyilvános IP-címkészletekről engedélyezi. Állítsa be az NSG-t a többi alhálózathoz is úgy, hogy engedélyezzék a felügyeleti alhálózatból érkező RDP-forgalmat.

## <a name="scalability-considerations"></a>Méretezési szempontok

A web és business szintek, érdemes lehet használatával [a virtual machine scale sets][vmss], állítsa be a különálló virtuális gépek rendelkezésre állási környezetbe való üzembe helyezés helyett. Egy méretezési csoportot egyszerűen üzembe helyezheti és kezelheti az azonos virtuális gépek és az automatikus méretezés a virtuális gépek teljesítmény-mérőszámok alapján. Ahogy a terhelés növekszik a virtuális gépeken, a rendszer további virtuális gépeket ad a terheléselosztóhoz. Fontolja meg a méretezési csoportok használatát, ha virtuális gépek gyors horizontális felskálázására vagy automatikus méretezésre van szüksége.

A méretezési csoportokban üzembe helyezett virtuális gépek konfigurálásának két alapvető módja van:

- Bővítmények segítségével konfigurálhatja a virtuális Gépet a telepítés után. Ezzel a módszerrel az új virtuálisgép-példányok indítása több időt vehet igénybe, mint a bővítmények nélküli virtuális gépeké.

- Üzembe helyezhet egy [felügyelt lemezt](/azure/storage/storage-managed-disks-overview) egy egyéni rendszerképpel. Ez a módszer gyorsabban kivitelezhető. Azonban ez megköveteli, hogy a kép naprakészen.

További információkért lásd: [kialakítási szempontok a méretezési csoportok][vmss-design].

> [!TIP]
> Bármilyen automatikus méretezési megoldás használata esetén jóval előre tesztelje azt az éles környezetre jellemző mennyiségű számítási feladattal.

Minden Azure-előfizetésre alapértelmezett korlátozások vonatkoznak. Ilyen például a régiónként alkalmazható virtuális gépek maximális száma. Támogatási kérelem kitöltésével megnövelheti ezt a korlátot. További információk: [Az Azure-előfizetések és -szolgáltatások korlátozásai, kvótái és megkötései][subscription-limits].

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

Ha nem használja a virtual machine scale sets, helyezzen virtuális gépet az azonos szinten egy rendelkezésre állási csoportba. Hozzon létre legalább két virtuális gépet a rendelkezésre állási csoportban az [Azure-beli virtuális gépekre vonatkozó rendelkezésre állási SLA][vm-sla] támogatásához. További információk: [Virtuális gépek rendelkezésre állásának kezelése][availability-set]. A méretezési csoportok automatikus használata *elhelyezési csoportra*, amely szerepét egy implicit rendelkezésre állási csoportot.

A terheléselosztó [állapotmintákat][health-probes] használ a virtuálisgép-példányok rendelkezésre állásának monitorozásához. Ha a mintavétel nem ér el egy példányt egy bizonyos időkorláton belül, a terheléselosztó nem irányít forgalmat küld a virtuális Gépeket. A terheléselosztó ezután is folytatja a mintavételezést, és amint a virtuális gép újra elérhetővé válik, a terheléselosztó ismét elkezd forgalmat irányítani felé.

Az alábbiakban néhány javaslat olvasható a terheléselosztó állapot-mintavételére vonatkozóan:

* A mintavételek a HTTP-t vagy a TCP-t tesztelhetik. Ha a virtuális gépek HTTP-kiszolgálót futtatnak, HTTP-mintavételt hozzon létre. Egyéb esetben TCP-mintavételt hozzon létre.
* A HTTP-mintavétel esetében adjon meg elérési utat egy HTTP-végponthoz. A mintavétel 200-as HTTP-választ vár erről a megadott elérési útról. Az elérési út lehet a gyökér elérési útja ("/"), vagy a végpont, amely valamilyen egyéni logikát az alkalmazás állapotának ellenőrzéséhez. A végpontnak engedélyeznie kell a névtelen HTTP-kérelmeket.
* A mintavétel [ismert IP-címről érkezik:][health-probe-ip] 168.63.129.16. Nem blokkolja a bejövő és kimenő forgalmat az IP-címről valamelyik tűzfalszabályzatban vagy NSG-szabályok a.
* Az állapotminták állapotának megtekintéséhez használjon [állapotminta-naplókat][health-probe-log]. Engedélyezze a naplózást az Azure Portalon minden egyes terheléselosztóra vonatkozóan. A naplókat a rendszer az Azure Blob Storage-ba írja. A naplók megmutatják, hány virtuális gépet nincsenek rájuk vonatkozó hálózati forgalom meghiúsult mintavételi válaszok miatt.

Ha van szüksége, mint a magasabb rendelkezésre állás a [Azure SLA-beli virtuális gépek] [ vm-sla] biztosít, fontolja meg replikációs az alkalmazást két régióban, az Azure Traffic Managerrel a feladatátvételhez. További információkért lásd: [többrégiós N szintű alkalmazás magas rendelkezésre állású][multi-dc].  

## <a name="security-considerations"></a>Biztonsági szempontok

A virtuális hálózatok forgalomelkülönítési határok az Azure-ban. Egy virtuális hálózaton található virtuális gépek nem tud közvetlenül kommunikálni egy másik virtuális hálózattal a virtuális gépeket. Az azonos virtuális hálózatban elhelyezkedő virtuális gépek képesek a kommunikációra, hacsak [hálózati biztonsági csoportokat] [ nsg] (NSG-ket) nem hoz létre a forgalom korlátozására. További információ: [A Microsoft felhőszolgáltatásai és hálózati biztonság][network-security].

**SZEGÉLYHÁLÓZAT (DMZ)**. Érdemes lehet hozzáadnia egy hálózati virtuális berendezést (network virtual appliance, NVA), hogy DMZ-t lehessen létrehozni az internet és az Azure-beli virtuális hálózat között. Az NVA egy általános kifejezés egy olyan virtuális berendezésre, amely hálózatokhoz kapcsolódó feladatokat lát el, például gondoskodik a tűzfalról, a csomagvizsgálatról, a naplózásról és az egyéni útválasztásról. További információkért lásd a [DMZ az Azure és az internet közötti implementálásával][dmz] foglalkozó témakört.

**Titkosítási**. Titkosíthatja az inaktív bizalmas adatokat, és az [Azure Key Vaulttal][azure-key-vault] kezelheti az adatbázis titkosítási kulcsait. A Key Vault képes a hardveres biztonsági modulok (HSM-ek) titkosítási kulcsainak tárolására. További információkért lásd: [konfigurálása az Azure Key Vault-integráció az SQL Server Azure virtuális gépeken][sql-keyvault]. Emellett ajánlott alkalmazások titkos adatait, például az adatbázis kapcsolati karakterláncainak tárolása a Key Vaultban.

**A DDoS protection**. Az Azure platform alapértelmezés szerint alapszintű DDoS elleni védelmet biztosít. Ez a alapvető védelem célja védelme az Azure-infrastruktúra egészében. Alapszintű DDoS elleni védelem automatikusan engedélyezve van, de javasoljuk a [DDoS Protection Standard][ddos]. Standard szintű védelem használatával adaptív finomhangolási, az alkalmazás hálózati forgalmi minták alapján észlelheti a fenyegetéseket. Ez lehetővé teszi, hogy az infrastruktúra-szintű DDoS szabályzatok egyébként észrevétlenül DDoS-támadásokkal szembeni tulajdonosaival. Standard szintű védelmet is biztosít a riasztás, a telemetria és az Azure Monitor-analitikát. További információkért lásd: [Azure DDoS Protection: ajánlott eljárások és referenciaarchitektúrákat][ddos-best-practices].

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

Ennek a referenciaarchitektúrának egy üzemelő példánya elérhető a [GitHubon][github-folder]. A teljes üzembe helyezés eltarthat akár két órát, amely tartalmazza az Active Directory tartományi szolgáltatások, a Windows Server feladatátvevő fürt és az SQL Server rendelkezésre állási csoport konfigurálása a szkriptek futtatására.

### <a name="prerequisites"></a>Előfeltételek

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

1. A következő paranccsal hozzon létre egy erőforráscsoportot.

    ```bash
    az group create --location <location> --name <resource-group-name>
    ```

2. A következő parancsot a Felhőbeli tanúsító a Storage-fiók létrehozása.

    ```bash
    az storage account create --location <location> \
      --name <storage-account-name> \
      --resource-group <resource-group-name> \
      --sku Standard_LRS
    ```

3. Keresse meg a `virtual-machines\n-tier-windows` referencia architektúrák GitHub-adattár mappát.

4. Nyissa meg az `n-tier-windows.json` fájlt. 

5. Keresse meg az "witnessStorageBlobEndPoint" összes példányát, és cserélje le a helyőrző szöveg a 2. lépés a tárfiók nevére.

    ```json
    "witnessStorageBlobEndPoint": "https://[replace-with-storageaccountname].blob.core.windows.net",
    ```

6. A következő parancsot a tárfiók fiók kulcsainak listázása.

    ```bash
    az storage account keys list \
      --account-name <storage-account-name> \
      --resource-group <resource-group-name>
    ```

    A kimenet a következőhöz hasonlóan kell kinéznie. Másolja a `key1` értékét.

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

7. Az a `n-tier-windows.json` fájlt, keressen a "witnessStorageAccountKey", és illessze be a fiókkulcsot.

    ```json
    "witnessStorageAccountKey": "[replace-with-storagekey]"
    ```

8. Az a `n-tier-windows.json` fájlt, keresse meg az összes példányát `[replace-with-password]` és `[replace-with-sql-password]` egy erős jelszót cserélje le őket. Mentse a fájlt.

    > [!NOTE]
    > Ha módosítja a rendszergazdai felhasználónevet, frissíteni kell a `extensions` letiltja a JSON-fájlban. 

9. Futtassa a következő parancsot az architektúra üzembe helyezéséhez.

    ```bash
    azbb -s <your subscription_id> -g <resource_group_name> -l <location> -p n-tier-windows.json --deploy
    ```

A mintául szolgáló referenciaarchitektúra Azure-építőelemekkel történő üzembe helyezéséről további információkat a [GitHub-adattárban][git] talál.


<!-- links -->
[dmz]: ../dmz/secure-vnet-dmz.md
[multi-dc]: multi-region-sql-server.md
[n-tier]: n-tier.md
[azure-availability-sets]: /azure/virtual-machines/virtual-machines-windows-manage-availability#configure-each-application-tier-into-separate-availability-sets
[azure-dns]: /azure/dns/dns-overview
[azure-key-vault]: https://azure.microsoft.com/services/key-vault
[bástyagazdagép]: https://en.wikipedia.org/wiki/Bastion_host
[cidr]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[ddos]: /azure/virtual-network/ddos-protection-overview
[ddos-best-practices]: /azure/security/azure-ddos-best-practices
[git]: https://github.com/mspnp/template-building-blocks
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/n-tier-windows
[nsg]: /azure/virtual-network/virtual-networks-nsg
[plan-network]: /azure/virtual-network/virtual-network-vnet-plan-design-arm
[private-ip-space]: https://en.wikipedia.org/wiki/Private_network#Private_IPv4_address_spaces
[nyilvános IP-címet]: /azure/virtual-network/virtual-network-ip-addresses-overview-arm
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
[visio-download]: https://archcenter.blob.core.windows.net/cdn/vm-reference-architectures.vsdx
[0]: ./images/n-tier-sql-server.png "N szintű architektúra a Microsoft Azure használatával"
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview 
[vmss]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
[load-balancer]: /azure/load-balancer/
[load-balancer-hashing]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[vmss-design]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-design-overview
[subscription-limits]: /azure/azure-subscription-service-limits
[availability-set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[health-probes]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[health-probe-log]: /azure/load-balancer/load-balancer-monitor-log
[health-probe-ip]: /azure/virtual-network/virtual-networks-nsg#special-rules
[network-security]: /azure/best-practices-network-security
