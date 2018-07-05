---
title: DMZ implementálása az Azure és az internet között
description: Interneteléréssel rendelkező, biztonságos hibrid hálózati architektúra megvalósítása az Azure-ban.
author: telmosampaio
ms.date: 07/02/2018
pnp.series.title: Network DMZ
pnp.series.next: nva-ha
pnp.series.prev: secure-vnet-hybrid
cardTitle: DMZ between Azure and the Internet
ms.openlocfilehash: 7a062d2394ae8b3bd1b17c19cbdf512327f9a766
ms.sourcegitcommit: 9b459f75254d97617e16eddd0d411d1f80b7fe90
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 07/03/2018
ms.locfileid: "37403147"
---
# <a name="dmz-between-azure-and-the-internet"></a>Szegélyhálózat (DMZ) az Azure és az internet között

Ez a referenciaarchitektúra egy biztonságos hibrid hálózatot mutat be, amely kiterjeszti a helyszíni hálózatot az Azure-ba, és internetes forgalmat is fogad. [**A megoldás üzembe helyezése**.](#deploy-the-solution)

[![0]][0] 

*Töltse le az architektúra [Visio-fájlját][visio-download].*

Ez a referenciaarchitektúra a [DMZ az Azure és a helyszíni adatközpont közötti implementálásával kapcsolatos][implementing-a-secure-hybrid-network-architecture] cikkben ismertetett architektúrát bővíti ki. Hozzáad egy, az internetes forgalmat kezelő nyilvános DMZ-t a helyszíni hálózatról érkező forgalmat kezelő privát DMZ mellett. 

Az architektúra gyakori használati módjai többek között a következők:

* Hibrid alkalmazások, amelyekben a számítási feladatok részben a helyszínen, részben pedig az Azure-ban futnak.
* Azure-infrastruktúra, amely átirányítja a helyszínről és az internetről bejövő forgalmat.

## <a name="architecture"></a>Architektúra

Az architektúra a következőkben leírt összetevőkből áll.

* **Nyilvános IP-cím (PIP)**. A nyilvános végpont IP-címe. Az internethez csatlakozó külső felhasználók ezen a címen keresztül férhetnek hozzá a rendszerhez.
* **Hálózati virtuális berendezés (NVA)**. Ez az architektúra tartalmaz egy különálló NVA-készletet az internetről érkező forgalom számára.
* **Azure Load Balancer**. Az internetről érkező összes kérés a terheléselosztón halad keresztül, amely elosztja azokat a nyilvános DMZ-n található NVA-k között.
* **Nyilvános DMZ bejövő alhálózata**. Ez az alhálózat fogadja az Azure Load Balancer kéréseit. Ezután átadja a bejövő kéréseket a nyilvános DMZ egyik NVA-jának.
* **Nyilvános DMZ kimenő alhálózata**. Az NVA által jóváhagyott kérések ezen az alhálózaton haladnak át a webes réteg belső terheléselosztója felé.

## <a name="recommendations"></a>Javaslatok

Az alábbi javaslatok a legtöbb forgatókönyvre vonatkoznak. Kövesse ezeket a javaslatokat, ha nincsenek ezeket felülíró követelményei. 

### <a name="nva-recommendations"></a>Hálózati virtuális készülékre vonatkozó javaslatok

Használjon egy NVA-készletet az internetről érkező forgalomhoz, és egy másikat a helyszínről érkezőhöz. Ha egyetlen NVA-készlet használ mindkettőhöz, az biztonsági kockázatot jelent, mivel nincs biztonsági határ a két fajta hálózati forgalom között. A különálló NVA-k használata csökkenti az ellenőrző biztonsági szabályok összetettségét, és egyértelművé teszi, hogy melyik szabály vonatkozik az egyes bejövő hálózati kérésekre. Az egyik NVA-készlet csak az internetes forgalomra, a másik pedig csak a helyszínről érkező forgalomra vonatkozó szabályokat implementál.

Alkalmazzon egy 7. rétegbeli NVA-t, amely megszakítja az alkalmazások kapcsolatát az NVA szintjén, és fenntartja a kompatibilitást a háttérszintekkel. Ez garantálja a szimmetrikus kapcsolatokat, ahol a háttérbeli rétegekből érkező válaszforgalom az NVA-n halad keresztül.  

### <a name="public-load-balancer-recommendations"></a>Nyilvános terheléselosztóra vonatkozó javaslatok

A skálázhatóság és rendelkezésre állás érdekében a nyilvános DMZ NVA-it egy [rendelkezésre állási csoportban][availability-set] helyezze üzembe, és egy [internetkapcsolattal rendelkező terheléselosztó][load-balancer] használatával ossza el az internetes kéréseket a rendelkezésre állási csoport NVA-i között.  

Konfigurálja úgy a terheléselosztót, hogy az csak az internetes forgalomhoz szükséges portokból fogadjon el kéréseket. Korlátozza például a 80-as portra a beérkező HTTP-, valamint a 443-as portra a beérkező HTTPS-kéréseket.

## <a name="scalability-considerations"></a>Méretezési szempontok

Javasoljuk, hogy helyezzen el egy terheléselosztót a nyilvános DMZ előtt a kezdetekkor abban az esetben is, ha az architektúrában kezdetben egyetlen NVA szükséges a nyilvános DMZ-ben. Ez megkönnyíti a több NVA-ra való skálázást a jövőben, ha szükséges.

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

Az internetkapcsolattal rendelkező terheléselosztó megköveteli, hogy a nyilvános DMZ bejövő alhálózatán található minden NVA implementáljon egy [állapotmintát][lb-probe]. A terheléselosztó nem elérhetőnek tekinti azokat az állapotmintákat, amelyek nem válaszolnak ezen a végponton, és a kéréseket a rendelkezésre állási csoport más NVA-i számára továbbítja. Vegye figyelembe, hogy ha egyik NVA sem válaszol, az alkalmazás összeomlik, ezért fontos a monitorozás konfigurálása, amely értesíti a fejlesztési és üzemeltetési csapat tagjait, ha a kifogástalan NVA-k száma egy meghatározott küszöbérték alá esik.

## <a name="manageability-considerations"></a>Felügyeleti szempontok

Monitorozása és felügyelete az nva-k a nyilvános DMZ-ben a a felügyeleti alhálózaton található jumpbox kell elvégeznie. A [DMZ az Azure és a helyszíni adatközpont közötti implementálásával kapcsolatos][implementing-a-secure-hybrid-network-architecture] cikkben ismertetett módon definiáljon egyetlen hálózati útvonalat a helyszíni hálózattól az átjárón keresztül a jumpboxig a hozzáférés korlátozása érdekében.

Ha a helyszíni hálózat és az Azure közötti átjárókapcsolat nem érhető el, a jumpboxot elérheti egy nyilvános IP-cím üzembe helyezésével és a jumpboxhoz való hozzáadásával, majd az internetről való bejelentkezéssel.

## <a name="security-considerations"></a>Biztonsági szempontok

Ez a referenciaarchitektúra többszintű biztonságot valósít meg:

* Az internetkapcsolattal rendelkező terheléselosztó átirányítja a kéréseket a nyilvános DMZ bejövő alhálózatán található NVA-khoz, kizárólag az alkalmazás számára szükséges portokon.
* A nyilvános DMZ bejövő és kimenő alhálózatára vonatkozó NSG-szabályok megakadályozzák az NVA-k feltörését az NSG-szabályokon kívül eső kérések blokkolásával.
* Az NVA-k NAT-útválasztási konfigurációja átirányítja a 80-as és 443-as portokon bejövő kéréseket a webes rétegben lévő terheléselosztóhoz, de figyelmen hagyja az összes többi porton érkező kéréseket.

Érdemes az összes porton érkező bejövő kérésekről naplót készíteni. Rendszeresen ellenőrizze a naplókat, és fordítson különös figyelmet a várt paramétereken kívül eső kérésekre, mert ezek behatolási kísérleteket jelezhetnek.


## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

Az ezeknek a javaslatoknak a figyelembe vételével megvalósított referenciaarchitektúra egy üzemelő példánya elérhető a [GitHubon][github-folder]. 

### <a name="prerequisites"></a>Előfeltételek

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-resources"></a>Erőforrások üzembe helyezése

1. Keresse meg a `/dmz/secure-vnet-hybrid` referencia architektúrák GitHub-adattár mappát.

2. Futtassa az alábbi parancsot:

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <region> -p onprem.json --deploy
    ```

3. Futtassa az alábbi parancsot:

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <region> -p secure-vnet-hybrid.json --deploy
    ```

### <a name="connect-the-on-premises-and-azure-gateways"></a>A helyszíni és az Azure-átjárók csatlakoztatása

Ebben a lépésben két helyi hálózati átjáró csatlakozik.

1. Az Azure Portalon keresse meg a létrehozott erőforráscsoportot. 

2. Keresse meg az erőforrást nevű `ra-vpn-vgw-pip` , és másolja az IP-cím látható a **áttekintése** panelen.

3. Keresse meg az erőforrást nevű `onprem-vpn-lgw`.

4. Kattintson a **konfigurációs** panelen. A **IP-cím**, illessze be a 2. lépésben felírt IP-címet.

    ![](./images/local-net-gw.png)

5. Kattintson a **mentése** és várjon, amíg a művelet befejeződik. Körülbelül 5 percet is igénybe vehet.

6. Keresse meg az erőforrást nevű `onprem-vpn-gateway1-pip`. Másolja ki az IP-cím látható a **áttekintése** panelen.

7. Keresse meg az erőforrást nevű `ra-vpn-lgw`. 

8. Kattintson a **konfigurációs** panelen. A **IP-cím**, illessze be a 6. lépésben felírt IP-címet.

9. Kattintson a **mentése** és várjon, amíg a művelet befejeződik.

10. A kapcsolat ellenőrzéséhez nyissa meg a **kapcsolatok** minden átjáró paneljét. A állapotúnak kell lennie **csatlakoztatva**.

### <a name="verify-that-network-traffic-reaches-the-web-tier"></a>Győződjön meg arról, hogy a hálózati forgalom eléri a webes szint

1. Az Azure Portalon keresse meg a létrehozott erőforráscsoportot. 

2. Keresse meg az erőforrást nevű `pub-dmz-lb`, amely a terheléselosztó a nyilvános DMZ előtt. 

3. Másolja a nyilvános IP-addess származó a **áttekintése** panelen, és nyissa meg ezt a címet egy webböngészőben. Megtekintheti az Apache2 kiszolgáló alapértelmezett kezdőlapját.

4. Keresse meg az erőforrást nevű `int-dmz-lb`, azaz előtti terheléselosztó tartománynévcímkéje a privát DMZ-t. Másolja a magánhálózati IP-címet a **áttekintése** panelen.

5. Keresse meg a virtuális gép nevű `jb-vm1`. Kattintson a **Connect** és a távoli asztal használata a virtuális Géphez való csatlakozáshoz. A felhasználónév és jelszó a onprem.json fájlban vannak megadva.

6. A távoli asztali munkamenetet a nyisson meg egy webböngészőt, és keresse meg az IP-címet a 4. lépéssel. Megtekintheti az Apache2 kiszolgáló alapértelmezett kezdőlapját.

[availability-set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/dmz/secure-vnet-dmz

[implementing-a-secure-hybrid-network-architecture]: ./secure-vnet-hybrid.md
[iptables]: https://help.ubuntu.com/community/IptablesHowTo
[lb-probe]: /azure/load-balancer/load-balancer-custom-probe-overview
[load-balancer]: /azure/load-balancer/load-balancer-Internet-overview
[network-security-group]: /azure/virtual-network/virtual-networks-nsg

[visio-download]: https://archcenter.blob.core.windows.net/cdn/dmz-reference-architectures.vsdx


[0]: ./images/dmz-public.png "Biztonságos hibrid hálózati architektúra"
