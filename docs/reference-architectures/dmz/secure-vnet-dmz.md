---
title: DMZ implementálása az Azure és az internet között
description: Interneteléréssel rendelkező, biztonságos hibrid hálózati architektúra megvalósítása az Azure-ban.
author: telmosampaio
ms.date: 11/23/2016
pnp.series.title: Network DMZ
pnp.series.next: nva-ha
pnp.series.prev: secure-vnet-hybrid
cardTitle: DMZ between Azure and the Internet
ms.openlocfilehash: c88545b1fcae49b413e7e2b6ac5bd92d3fd3456d
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/30/2018
ms.locfileid: "30270404"
---
# <a name="dmz-between-azure-and-the-internet"></a>Szegélyhálózat (DMZ) az Azure és az internet között

Ez a referenciaarchitektúra egy biztonságos hibrid hálózatot mutat be, amely kiterjeszti a helyszíni hálózatot az Azure-ba, és internetes forgalmat is fogad. 

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

Az összes figyelési és -felügyelet a nyilvános Szegélyhálózaton lévő NVAs által a felügyeleti alhálózat jumpbox hajtható végre. A [DMZ az Azure és a helyszíni adatközpont közötti implementálásával kapcsolatos][implementing-a-secure-hybrid-network-architecture] cikkben ismertetett módon definiáljon egyetlen hálózati útvonalat a helyszíni hálózattól az átjárón keresztül a jumpboxig a hozzáférés korlátozása érdekében.

Ha a helyszíni hálózat és az Azure közötti átjárókapcsolat nem érhető el, a jumpboxot elérheti egy nyilvános IP-cím üzembe helyezésével és a jumpboxhoz való hozzáadásával, majd az internetről való bejelentkezéssel.

## <a name="security-considerations"></a>Biztonsági szempontok

Ez a referenciaarchitektúra többszintű biztonságot valósít meg:

* Az internetkapcsolattal rendelkező terheléselosztó átirányítja a kéréseket a nyilvános DMZ bejövő alhálózatán található NVA-khoz, kizárólag az alkalmazás számára szükséges portokon.
* A nyilvános DMZ bejövő és kimenő alhálózatára vonatkozó NSG-szabályok megakadályozzák az NVA-k feltörését az NSG-szabályokon kívül eső kérések blokkolásával.
* Az NVA-k NAT-útválasztási konfigurációja átirányítja a 80-as és 443-as portokon bejövő kéréseket a webes rétegben lévő terheléselosztóhoz, de figyelmen hagyja az összes többi porton érkező kéréseket.

Érdemes az összes porton érkező bejövő kérésekről naplót készíteni. Rendszeresen ellenőrizze a naplókat, és fordítson különös figyelmet a várt paramétereken kívül eső kérésekre, mert ezek behatolási kísérleteket jelezhetnek.

## <a name="solution-deployment"></a>A megoldás üzembe helyezése

Az ezeknek a javaslatoknak a figyelembe vételével megvalósított referenciaarchitektúra egy üzemelő példánya elérhető a [GitHubon][github-folder]. A referenciaarchitektúra Windows vagy Linux rendszerű virtuális gépeken helyezhető üzembe az alábbi utasításokat követve:

1. Kattintson az alábbi gombra:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fdmz%2Fsecure-vnet-dmz%2FvirtualNetwork.azuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. Ha a hivatkozás megnyílt az Azure Portalon, meg kell adnia néhány beállítás értékét:
   * Az **Erőforráscsoport** neve már meg van adva a paraméterfájlban, ezért válassza az **Új létrehozása** lehetőséget és a szövegmezőbe írja az `ra-public-dmz-network-rg` karakterláncot.
   * Válassza ki a régiót a **Hely** legördülő listából.
   * Ne szerkessze a **Sablon gyökér szintű URI-je** vagy a **Paraméter gyökér szintű URI-je** szövegmezőt.
   * Válassza ki az **operációs rendszer típusát** a legördülő listából: **Windows** vagy **Linux**.
   * Tekintse át a használati feltételeket, majd kattintson az **Elfogadom a fenti feltételeket** lehetőségre.
   * Kattintson a **Vásárlás** gombra.
3. Várjon, amíg az üzembe helyezés befejeződik.
4. Kattintson az alábbi gombra:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fdmz%2Fsecure-vnet-dmz%2Fworkload.azuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
5. Ha a hivatkozás megnyílt az Azure Portalon, meg kell adnia néhány beállítás értékét:
   * Az **Erőforráscsoport** neve már meg van adva a paraméterfájlban, ezért válassza az **Új létrehozása** lehetőséget és a szövegmezőbe írja az `ra-public-dmz-wl-rg` karakterláncot.
   * Válassza ki a régiót a **Hely** legördülő listából.
   * Ne szerkessze a **Sablon gyökér szintű URI-je** vagy a **Paraméter gyökér szintű URI-je** szövegmezőt.
   * Tekintse át a használati feltételeket, majd kattintson az **Elfogadom a fenti feltételeket** lehetőségre.
   * Kattintson a **Vásárlás** gombra.
6. Várjon, amíg az üzembe helyezés befejeződik.
7. Kattintson az alábbi gombra:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fdmz%2Fsecure-vnet-dmz%2Fsecurity.azuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
8. Ha a hivatkozás megnyílt az Azure Portalon, meg kell adnia néhány beállítás értékét:
   * Az **erőforráscsoport** neve már meg van adva a paraméterfájlban, ezért válassza a **Meglévő használata** lehetőséget, és a szövegmezőbe írja be az `ra-public-dmz-network-rg` karakterláncot.
   * Válassza ki a régiót a **Hely** legördülő listából.
   * Ne szerkessze a **Sablon gyökér szintű URI-je** vagy a **Paraméter gyökér szintű URI-je** szövegmezőt.
   * Tekintse át a használati feltételeket, majd kattintson az **Elfogadom a fenti feltételeket** lehetőségre.
   * Kattintson a **Vásárlás** gombra.
9. Várjon, amíg az üzembe helyezés befejeződik.
10. A paraméterfájlokban szerepel egy nem módosítható rendszergazdai felhasználónév és jelszó minden virtuális gép számára. Erősen ajánlott mindkettőt azonnal lecserélni. Az üzemelő példány minden virtuális gépe esetében válassza ki a gépet az Azure Portalon, majd a **Támogatás + hibaelhárítás** panelen kattintson a **Jelszó alaphelyzetbe állítása** lehetőségre. A **Mód** legördülő listában válassza a **Jelszó alaphelyzetbe állítása** lehetőséget, majd adjon meg új értéket a **Felhasználónév** és a **Jelszó** mezőben. Kattintson a **Frissítés** gombra a mentéshez.


[availability-set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/dmz/secure-vnet-dmz

[implementing-a-secure-hybrid-network-architecture]: ./secure-vnet-hybrid.md
[iptables]: https://help.ubuntu.com/community/IptablesHowTo
[lb-probe]: /azure/load-balancer/load-balancer-custom-probe-overview
[load-balancer]: /azure/load-balancer/load-balancer-Internet-overview
[network-security-group]: /azure/virtual-network/virtual-networks-nsg

[visio-download]: https://archcenter.blob.core.windows.net/cdn/dmz-reference-architectures.vsdx


[0]: ./images/dmz-public.png "Biztonságos hibrid hálózati architektúra"
