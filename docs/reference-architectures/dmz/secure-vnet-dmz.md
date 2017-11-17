---
title: "Végrehajtási DMZ Azure és az Internet között"
description: "Hogyan egy biztonságos hibrid hálózati architektúra Internet-hozzáféréssel rendelkező végrehajtásához az Azure-ban."
author: telmosampaio
ms.date: 11/23/2016
pnp.series.title: Network DMZ
pnp.series.next: nva-ha
pnp.series.prev: secure-vnet-hybrid
cardTitle: DMZ between Azure and the Internet
ms.openlocfilehash: 372d5bb0fc0e3c272843e062210dec5c15b2b78a
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="dmz-between-azure-and-the-internet"></a>Szegélyhálózat (DMZ) az Azure és az internet között

A referencia-architektúrában olyan biztonságos hibrid hálózatot, amely egy helyi hálózati kibővíti az Azure-ba, és is fogad az internetes forgalmat jeleníti meg. 

[![0]][0] 

*Töltse le a [Visio fájl] [ visio-download] ezen architektúra.*

A referencia-architektúrában kiterjeszti a leírt architektúra [Azure és a helyszíni adatközpont között DMZ végrehajtási][implementing-a-secure-hybrid-network-architecture]. Egy nyilvános DMZ internetes forgalom mellett a saját Szegélyhálózaton, amely kezeli a helyszíni hálózati forgalmat kezelő hozzáadása 

Ez az architektúra a jellemző használati többek között:

* Hibrid alkalmazások ahol munkaterhelések Futtatás részben helyszínen és részben az Azure.
* Azure-infrastruktúra, amely a helyszíni és az internetről érkező bejövő forgalmat.

## <a name="architecture"></a>Architektúra

Az architektúra a következő összetevőkből áll.

* **Nyilvános IP-cím (PIP)**. A nyilvános végpontot IP-címét. Külső felhasználók csatlakozik az internethez a cím segítségével is elérhetik a rendszert.
* **Hálózati virtuális készülék (NVA)**. Ez az architektúra az interneten érkező forgalom NVAs külön készletét tartalmazza.
* **Az Azure terheléselosztó**. A load balancer továbbítja az internetről érkező összes bejövő kérelmet, és továbbítja őket a nyilvános Szegélyhálózaton lévő NVAs.
* **Nyilvános DMZ bejövő alhálózati**. Ez az alhálózat az Azure load balancer érkező kéréseket fogad el. Bejövő kérelmek kerülnek átadásra a nyilvános Szegélyhálózaton lévő NVAs egyikét.
* **Nyilvános DMZ kimenő alhálózati**. Ez az alhálózat a belső terheléselosztóhoz a webes réteg az NVA által engedélyezett kérelmek továbbítása.

## <a name="recommendations"></a>Javaslatok

Az alábbi javaslatok a legtöbb forgatókönyvre vonatkoznak. Kövesse ezeket a javaslatokat, ha nincsenek ezeket felülíró követelményei. 

### <a name="nva-recommendations"></a>NVA javaslatok

Válasszon egy NVAs forgalom az internethez, és a helyszíni származó forgalmat egy másik származó. NVAs csak egy készletét mindkét használata biztonsági kockázatot jelent, mivel nincs biztonsági külső hálózati forgalom két készlet között. Külön NVAs használata csökkenti a biztonsági szabályok keresése összetettségét, és teszi, hogy mely szabályokat felel meg az egyes bejövő kérelmek hálózati törölje. NVAs egy készletét, csak az internetes forgalmat szabályainak valósítja meg, amíg NVAs más engedélykészletre szabályok csak a helyszíni adatforgalomhoz végrehajtására.

A réteg-7 NVA állítsa le az alkalmazás-kapcsolatok az NVA szinten, majd a háttérrendszer rétegek való kompatibilitás tartalmazza. Ez biztosítja, hogy szimmetrikus kapcsolat, ahol a háttérrendszer rétegek válasz forgalmát adja vissza, az NVA keresztül.  

### <a name="public-load-balancer-recommendations"></a>Nyilvános load balancer javaslatok

A méretezhetőség és a rendelkezésre állási, a nyilvános DMZ NVAs a telepítése egy [rendelkezésre állási csoport] [ availability-set] , és egy [internetre irányuló terheléselosztót] [ load-balancer]internetes kérelmek szét a rendelkezésre állási készlet NVAs.  

A load balancer csak internetes forgalomhoz szükséges portokat a kérelmek fogadására konfigurálja. Például korlátozhatja a 80-as portot a bejövő HTTP-kéréseket, és a 443-as portot a bejövő HTTPS-kéréseket.

## <a name="scalability-considerations"></a>Méretezési szempontok

Akkor is, ha az architektúrák kezdetben a nyilvános szegélyhálózaton egy egyetlen NVA van szüksége, javasoljuk a terheléselosztó a nyilvános DMZ elé üzembe az elejétől. Amely megkönnyíti a méretezhető, több NVAs a jövőben, szükség esetén.

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

Az internetre irányuló a terheléselosztót igényel a nyilvános Szegélyhálózaton lévő minden egyes NVA bejövő alhálózati végrehajtásához egy [állapotmintáihoz][lb-probe]. A rendszerállapot-vizsgálatot, amely nem válaszol ezen a végponton tekinthető nem érhető el, és a terheléselosztó az azonos rendelkezésre állási készlet átirányítja a más NVAs kéréseket. Vegye figyelembe, hogy minden NVAs nem válaszolnak, ha az alkalmazás sikertelen lesz, ezért fontos, hogy figyelés riasztási DevOps konfigurált Ha NVA megfelelő példányok száma a meghatározott küszöbérték alá esik.

## <a name="manageability-considerations"></a>Felügyeleti szempontok

Minden ellenőrzése és kezelése a nyilvános Szegélyhálózaton lévő NVAs lehet végzi el a felügyeleti alhálózat jumpbox. A bemutatott [Azure és a helyszíni adatközpont között DMZ végrehajtási][implementing-a-secure-hybrid-network-architecture], határozza meg a helyi hálózatról a jumpbox az átjárón keresztül egyetlen hálózati útvonal korlátozása a hozzáférés.

Ha az Azure-bA a helyi hálózati átjáró kapcsolat nem működik, továbbra is érhető el a jumpbox telepítése egy nyilvános IP-címet, a jumpbox való hozzáadását és az interneten van bejelentkezve.

## <a name="security-considerations"></a>Biztonsági szempontok

A referencia-architektúrában több szintű biztonságot valósítja meg:

* Az internetre irányuló a terheléselosztó irányítja a NVAs kéréseket, a bejövő nyilvános DMZ alhálózati, és csak az alkalmazáshoz szükséges portokat.
* Az NSG-szabályok, bejövő és kimenő nyilvános DMZ alhálózatok megakadályozhatják, hogy a NVAs sérült, az NSG-szabályok kívül eső kérések blokkolásával.
* A NAT a NVAs útválasztási konfigurációja irányítja a bejövő kérelmeket a 80-as és a webes réteg terheléselosztó 443-as porton, de figyelmen kívül hagyja az összes porton kérelmek.

Minden port az összes bejövő kérelmet naplózni kell. Rendszeresen naplók a, fizető figyelmet a kérelmekre, amelyek várt paraméter kívül esik, előfordulhat, hogy ezek határozzák meg behatolás kísérletek.

## <a name="solution-deployment"></a>A megoldás üzembe helyezése

A környezet egy referencia-architektúrában, amely megvalósítja az alábbi javaslatokat érhető el a [GitHub][github-folder]. A referencia-architektúrában telepíthető Windows vagy Linux rendszerű virtuális gépeken az alábbi utasításokat követve:

1. Kattintson a lenti gombra:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fdmz%2Fsecure-vnet-dmz%2FvirtualNetwork.azuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. Ha a hivatkozás megnyílt az Azure Portalon, meg kell adnia néhány beállítás értékét:
   * Az **Erőforráscsoport** neve már meg van adva a paraméterfájlban, ezért válassza az **Új létrehozása** lehetőséget és a szövegmezőbe írja az `ra-public-dmz-network-rg` karakterláncot.
   * Válassza ki a régiót a **Hely** legördülő listából.
   * Ne szerkessze a **Sablon gyökér szintű URI-je** vagy a **Paraméter gyökér szintű URI-je** szövegmezőt.
   * Válassza ki a **operációsrendszer-típus** a legördülő listából, **windows** vagy **linux**.
   * Tekintse át a használati feltételeket, majd kattintson az **Elfogadom a fenti feltételeket** lehetőségre.
   * Kattintson a **beszerzési** gombra.
3. Várjon, amíg az üzembe helyezés befejeződik.
4. Kattintson a lenti gombra:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fdmz%2Fsecure-vnet-dmz%2Fworkload.azuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
5. Ha a hivatkozás megnyílt az Azure Portalon, meg kell adnia néhány beállítás értékét:
   * Az **Erőforráscsoport** neve már meg van adva a paraméterfájlban, ezért válassza az **Új létrehozása** lehetőséget és a szövegmezőbe írja az `ra-public-dmz-wl-rg` karakterláncot.
   * Válassza ki a régiót a **Hely** legördülő listából.
   * Ne szerkessze a **Sablon gyökér szintű URI-je** vagy a **Paraméter gyökér szintű URI-je** szövegmezőt.
   * Tekintse át a használati feltételeket, majd kattintson az **Elfogadom a fenti feltételeket** lehetőségre.
   * Kattintson a **beszerzési** gombra.
6. Várjon, amíg az üzembe helyezés befejeződik.
7. Kattintson a lenti gombra:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fdmz%2Fsecure-vnet-dmz%2Fsecurity.azuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
8. Ha a hivatkozás megnyílt az Azure Portalon, meg kell adnia néhány beállítás értékét:
   * A **erőforráscsoport** neve már definiálva van a paraméterfájl, ezért select **használata meglévő** , és írja be `ra-public-dmz-network-rg` a szövegmezőben.
   * Válassza ki a régiót a **Hely** legördülő listából.
   * Ne szerkessze a **Sablon gyökér szintű URI-je** vagy a **Paraméter gyökér szintű URI-je** szövegmezőt.
   * Tekintse át a használati feltételeket, majd kattintson az **Elfogadom a fenti feltételeket** lehetőségre.
   * Kattintson a **beszerzési** gombra.
9. Várjon, amíg az üzembe helyezés befejeződik.
10. A paraméter fájlok közé tartoznak kódolt rendszergazda felhasználónevét és jelszavát az összes virtuális gépet, és erősen ajánlott, hogy azonnal módosíthatja is. Az egyes virtuális gépek, a központi telepítésben lévő, válassza ki azt az Azure portálon, és kattintson a **jelszó-átállítási** a a **támogatási + hibaelhárítási** panelen. Válassza ki **jelszó-átállítási** a a **mód** legördülő mezőben, majd válasszon egy új **felhasználónév** és **jelszó**. Kattintson a **frissítés** gombra kattintva mentse.


[availability-set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/dmz/secure-vnet-dmz

[implementing-a-secure-hybrid-network-architecture]: ./secure-vnet-hybrid.md
[iptables]: https://help.ubuntu.com/community/IptablesHowTo
[lb-probe]: /azure/load-balancer/load-balancer-custom-probe-overview
[load-balancer]: /azure/load-balancer/load-balancer-Internet-overview
[network-security-group]: /azure/virtual-network/virtual-networks-nsg

[visio-download]: https://archcenter.azureedge.net/cdn/dmz-reference-architectures.vsdx


[0]: ./images/dmz-public.png "biztonságos hibrid hálózati architektúra"