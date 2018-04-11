---
title: Egy a helyszíni hálózathoz való kapcsolódáshoz Azure megoldás kiválasztása
description: Hasonlítja össze egy a helyszíni hálózathoz való kapcsolódáshoz Azure architektúrák hivatkozik.
author: telmosampaio
ms.date: 04/06/2017
ms.openlocfilehash: 274b9df1817632a7f3eaafa8bf02e965fdc3feea
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="choose-a-solution-for-connecting-an-on-premises-network-to-azure"></a>Egy a helyszíni hálózathoz való kapcsolódáshoz Azure megoldás kiválasztása

Ez a cikk egy a helyszíni hálózathoz való kapcsolódáshoz az Azure Virtual Network (VNet) lehetőségeket hasonlítja össze. Az egyes lehetőségek nyújtunk a referencia-architektúrában és a központilag telepíthető megoldás.

## <a name="vpn-connection"></a>VPN-kapcsolat

A virtuális magánhálózat (VPN) segítségével csatlakozzon a helyi hálózaton egy Azure virtuális hálózatot egy IPSec VPN-alagúton keresztül.

Ez az architektúra hibrid alkalmazásoknál, ahol a forgalom között helyszíni hardver és a felhő várhatóan világos, vagy a rendszer a rugalmasság és a feldolgozási kapacitása révén a felhő némileg kiterjesztett késése kereskedelmi.

**Előnyei**

- Konfigurálása egyszerű.

**Kihívásai**

- Szükséges egy helyszíni VPN-eszközön.
- Bár a Microsoft biztosítja, hogy a 99,9 %-os rendelkezésre állás minden VPN-átjáró, a szolgáltatásiszint-szerződés vonatkozik a VPN-átjáró, és nem a hálózati kapcsolat az átjáróhoz.
- VPN-kapcsolaton keresztül Azure VPN Gateway jelenleg legfeljebb 200 MB/s sávszélesség. Szükség lehet az Azure virtual Networkhöz particionálásához több VPN-kapcsolatokon keresztül, ha várhatóan hosszabb legyen, mint a teljesítményt.

**[További tudnivalók...][vpn]**

## <a name="azure-expressroute-connection"></a>Az Azure ExpressRoute-kapcsolat

ExpressRoute-kapcsolatot egy külső kapcsolatot közvetítésével saját, dedikált kapcsolatot használja. A magánhálózati kapcsolat az Azure a helyszíni hálózat kiterjesztése. 

Ez az architektúra futó nagyméretű, kritikus, méretezhetőség magas fokú igénylő munkaterhelések hibrid alkalmazások alkalmas. 

**Előnyei**

- Sokkal nagyobb sávszélesség érhető el; legfeljebb 10 GB/s attól függően, hogy a kapcsolat szolgáltatóját.
- Támogatja a dinamikus méretezés sávszélesség során alacsonyabb igényű időszakainak költségek csökkentése érdekében. Azonban nem minden kapcsolat szolgáltatók kell ezt a beállítást.
- Engedélyezheti a szervezet nemzeti felhők, attól függően, hogy a kapcsolat szolgáltatójánál közvetlen hozzáférést.
- 99,9 %-os SLA-elérhetőséget az egész kapcsolaton keresztül.

**Kihívásai**

- Összetett feladat lehet beállítása. Egy ExpressRoute-kapcsolat létrehozásához szükséges külső kapcsolatot szolgáltatóval működő. A szolgáltató feladata a hálózati kapcsolat történő üzembe helyezéséhez.
- Helyszíni nagy sávszélességű útválasztók igényel.

**[További tudnivalók...][expressroute]**

## <a name="expressroute-with-vpn-failover"></a>A VPN-feladatátvételi ExpressRoute

A beállítások az előző két, ExpressRoute segítségével normál körülmények között, de feladatátadás a VPN-kapcsolatot, ha a kapcsolat az ExpressRoute-kapcsolatcsoport a veszteség egyesíti.

Ebbe az architektúrába, ami a hibrid alkalmazásokat, amelyek az ExpressRoute nagyobb sávszélességet kell, és azt is igénylik, magas rendelkezésre állású hálózati kapcsolat megfelelő. 

**Előnyei**

- Magas rendelkezésre állású Ha ExpressRoute-kapcsolatcsoport nem sikerül, a tartalék kapcsolatot azonban alacsonyabb sávszélességű hálózaton.

**Kihívásai**

- Összetett konfigurálásához. Akkor be kell állítania egy VPN-kapcsolat és a ExpressRoute-kapcsolatcsoportot.
- Redundáns hardveres (VPN-készülék), valamint a redundáns Azure VPN Gateway kapcsolat, amelynek díjakat kell fizetnie kell.

**[További tudnivalók...][expressroute-vpn-failover]**

<!-- links -->
[expressroute]: ./expressroute.md
[expressroute-vpn-failover]: ./expressroute-vpn-failover.md
[vpn]: ./vpn.md