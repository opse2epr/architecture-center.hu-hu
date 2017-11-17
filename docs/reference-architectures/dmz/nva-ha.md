---
title: "Egy magas rendelkezésre állású virtuális hálózati berendezések telepítése"
description: "Ügyfélszoftverek központi telepítése a virtuális hálózati készülékek a magas rendelkezésre állású."
author: telmosampaio
ms.date: 12/06/2016
pnp.series.title: Network DMZ
pnp.series.prev: secure-vnet-dmz
cardTitle: Deploy highly available network virtual appliances
ms.openlocfilehash: 844c87f535d2a8cb415489cb2c8e840f8c585d7d
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="deploy-highly-available-network-virtual-appliances"></a>Virtuális készülékek magas rendelkezésre állású hálózati telepítése

Ez a cikk a hálózati virtuális készülékek (NVAs) a magas rendelkezésre álláshoz az Azure-ban készlete telepítésének módját ismerteti. NVA általában használt hálózati forgalmat a szegélyhálózaton, más néven DMZ, és más hálózatok és alhálózatok üzenetáramlásának szabályozására. Az Azure-ban DMZ kapcsolatos további tudnivalókért lásd: [Microsoft cloud services és a hálózati biztonság][cloud-security]. A cikk csak érkező, kimenő csak, és a bemenő és kimenő példa architektúrához tartalmazza. 

**Előfeltételek:** megismerheti az Azure hálózatkezelés, a jelen cikk feltételezi [Azure load Balancer terheléselosztók][lb-overview], és [felhasználó által definiált útvonalak] [ udr-overview] (Udr-EK). 


## <a name="architecture-diagrams"></a>Architektúra

NVA számos különböző architektúrákban DMZ telepíthetők. Például a következőképpen ábrázolható használatát egy [NVA egyetlen] [ nva-scenario] érkező számára. 

![[0]][0]

Ebben az architektúrában az NVA biztosít a biztonságos hálózati határ ellenőrzése az összes bejövő és kimenő hálózati forgalom, és csak a forgalmat, amely megfelel a hálózati biztonsági szabályok átadásakor. Azonban, hogy az összes hálózati forgalom haladnak keresztül az NVA azt jelenti, hogy az NVA a hibaérzékeny pontok kialakulását a hálózaton. Az NVA nem sikerül, ha nincs más útvonal a hálózati forgalmat, és a háttér-alhálózat nem érhetők el.

NVA magas rendelkezésre állásúvá tételéhez telepítsen több NVA be egy rendelkezésre állási csoportot.    

A következő architektúrák azt mutatják be, az erőforrások és a magas rendelkezésre állású NVAs szükséges konfiguráció:

| Megoldás | Előnyök | Megfontolandó szempontok |
| --- | --- | --- |
| [A réteg 7 NVAs érkező][ingress-with-layer-7] |NVA csomópontjaihoz aktívak. |Állítsa le a kapcsolatok és SNAT használja NVA igényel</br> Meg kell adni egy külön NVAs, és az Azure-ból az internetről érkező forgalom </br> Csak akkor használható a kívülről Azure származó forgalmat |
| [A réteg 7 NVAs kilépési][egress-with-layer-7] |NVA csomópontjaihoz aktívak. | Megköveteli, hogy a kapcsolatok és megvalósít forrás hálózati címfordítás (SNAT) is leáll NVA
| [A réteg 7 NVAs érkező-kimenő][ingress-egress-with-layer-7] |Minden csomópont aktívak.<br/>Tudja kezelni a forgalmat az Azure-ban |Állítsa le a kapcsolatok és SNAT használja NVA igényel<br/>Meg kell adni egy külön NVAs, és az Azure-ból az internetről érkező forgalom |
| [A PIP-UDR kapcsoló][pip-udr-switch] |Egyetlen halmazába NVAs összes forgalom<br/>Az összes forgalom (portszabályokat korlátozva) is kezelése |Aktív-passzív<br/>A feladatátvételi folyamat igényel |

## <a name="ingress-with-layer-7-nvas"></a>A réteg 7 NVAs érkező

Az alábbi ábrán egy magas rendelkezésre állású architektúra, amely megvalósítja az érkező DMZ egy internetre irányuló terheléselosztó mögött. Ez az architektúra arra tervezték, hogy csatlakozhasson az Azure munkaterhelések réteg 7-forgalom esetén például a HTTP vagy HTTPS:

![[1]][1]

Az ebbe az architektúrába előnye, hogy minden NVAs aktív, és a terheléselosztó meghibásodásakor hálózati forgalmát átirányítja a más NVA. Irányíthatja a forgalmat a belső terheléselosztóhoz ezért mindaddig, amíg egy NVA aktív, forgalom mindkét NVAs továbbra is a. A NVAs szükséges terheléselosztónál zárja le a webes réteg virtuális gépek szánt SSL-forgalmat. Ezek NVAs nem terjeszthető ki a helyszíni-forgalom kezelésére, mert a helyszíni forgalom egy másik dedikált-készletet igényel NVAs a saját hálózati útvonalak.

> [!NOTE]
> Ez az architektúra használatban van a [DMZ Azure és a helyszíni adatközpont között] [ dmz-on-prem] hivatkozhat architektúra és a [DMZ Azure és az Internet között] [ dmz-internet] architektúra hivatkozik. A referencia architektúra mindegyikének tartalmaz egy központi telepítési megoldás, amely használható. Hivatkozásokon talál további információt.

## <a name="egress-with-layer-7-nvas"></a>A réteg 7 NVAs kilépési

Az előző architektúra bővíthet egy kimenő DMZ biztosít az Azure számítási származó kérelmek. A következő architektúra arra tervezték, hogy a Szegélyhálózaton lévő NVAs magas rendelkezésre állásának biztosítása a réteg 7-forgalmat, például HTTP vagy HTTPS:

![[2]][2]

Ebben az architektúrában az Azure-ban származó összes forgalmat egy belső terheléselosztóhoz történik. A terheléselosztó kimenő kérelmek NVAs közötti elosztása. Ezek NVAs közvetlen forgalom az internethez, az egyes nyilvános IP-címek használatával.

> [!NOTE]
> Ez az architektúra használatban van a [DMZ Azure és a helyszíni adatközpont között] [ dmz-on-prem] hivatkozhat architektúra és a [DMZ Azure és az Internet között] [ dmz-internet] architektúra hivatkozik. A referencia architektúra mindegyikének tartalmaz egy központi telepítési megoldás, amely használható. Hivatkozásokon talál további információt.

## <a name="ingress-egress-with-layer-7-nvas"></a>A réteg 7 NVAs érkező-kimenő

A két előző architektúrákban hiba történt a bemenő és kimenő külön DMZ. A következő architektúra bemutatja, hogyan kell létrehozni, amely egyaránt be- és kilépési réteg 7-forgalom esetén például a HTTP vagy HTTPS használható DMZ: 

![[4]][4]

Ebben az architektúrában az NVAs feldolgozni az Alkalmazásátjáró érkező kéréseket. A NVAs is érkező a munkaterhelési virtuális gépek a háttér-készletben a terheléselosztó kimenő kérések feldolgozását. Bejövő forgalom az Alkalmazásátjáró történik, és a terheléselosztó kimenő forgalmat történik, mert a NVAs munkamenet affinitás karbantartásáért felelős. Ez azt jelenti, hogy az Alkalmazásátjáró fenntart egy táblázatot a bejövő és kimenő kérelmek, így a helyes választ az eredeti kérelmezőnek továbbíthatja. A belső terheléselosztó azonban nincs hozzáférése a alkalmazástársítások átjáró, és használja a saját logikát a NVAs válaszokat küldhet. Akkor lehet a terheléselosztó, amely nem kezdetben kapta meg a kérelmet az Alkalmazásátjáró NVA választ küldhet. Ebben az esetben a NVAs kell kommunikációhoz, és a válasz továbbít köztük, így a helyes NVA továbbíthatja az Alkalmazásátjáró válasz.

> [!NOTE]
> A NVAs hajtsa végre a bemeneti forrás a hálózati címfordítás (SNAT) biztosításával a aszimmetrikus útválasztási probléma is megoldható. Ez az eredeti forrás IP-cím egy IP-címét használja a bejövő adatfolyam NVA azonosságát cserélje. Ez biztosítja, hogy több NVAs használhatja egyszerre útvonal szimmetriája adatainak megőrzése mellett.

## <a name="pip-udr-switch-with-layer-4-nvas"></a>4. rétegbeli NVAs PIP-UDR kapcsoló

A következő architektúra egy aktív és passzív NVA egy architektúrát mutatja be. Ez az architektúra is be- és kilépési a 4. rétegbeli forgalmat kezeli: 

![[3]][3]

Ez az architektúra a cikkben szereplő első architektúra hasonlít. Az architektúra része egy egyetlen NVA fogadása és 4. rétegbeli bejövő kérelmek szűrése. Ez az architektúra ad hozzá egy második passzív NVA magas rendelkezésre állás biztosításához. Az aktív NVA nem sikerül, ha a passzív NVA aktív legyen, és a UDR és a PIP megváltoznak, az aktív NVA hálózati kártyája mutasson. A UDR és a PIP a módosítások is, vagy manuálisan kell elvégezni, vagy egy automatikus folyamat segítségével. Az automatizált folyamat általában démon vagy más Azure-beli figyelőszolgáltatás. Lekérdezi a állapotmintáihoz meg az aktív NVA, és a UDR és PIP kapcsoló hajt végre, amikor azt észleli, hogy az NVA sikertelen. 

A fenti ábra mutat egy példát [ZooKeeper] [ zookeeper] egy magas rendelkezésre állású démon biztosító fürt. A ZooKeeper fürtön belül a megfelelő kvórumú csomópont egy vezető választja. A kitöltés nem sikerül, ha a többi csomópontot tartsa egy új vezető kiválasztják választást. Az ebbe az architektúrába a vezető csomópont a démon a rendszerállapot-végpont az NVA lekérdező hajtja végre. Az NVA nem válaszol a állapotmintáihoz, ha a démon a passzív NVA aktiválva lesz. A démon majd az Azure REST API-t a PIP eltávolítása a sikertelen NVA, és csatolja az újonnan aktivált NVA. A démon ezután módosítja a UDR az újonnan aktivált NVA belső IP-címre mutasson.

> [!NOTE]
> Ne vegyen fel a ZooKeeper csomópontok egy alhálózatot, amely csak egy olyan útvonalat, amely tartalmazza az NVA segítségével érhető el. Ellenkező esetben a ZooKeeper csomópontok nem érhetők az NVA meghibásodásakor. Amennyiben a démon nem bármilyen okból, akkor nem fogja tudni a probléma diagnosztizálása érdekében a ZooKeeper csomópontok egyikét sem használni. 

<!--### Solution Deployment-->

<!-- instructions for deploying this solution here --> 

## <a name="next-steps"></a>Következő lépések
* Megtudhatja, hogyan [valósítja meg az Azure és a helyszíni adatközpont között DMZ] [ dmz-on-prem] réteg-7 NVAs használatával.
* Megtudhatja, hogyan [valósítja meg az Azure és az Internet között DMZ] [ dmz-internet] réteg-7 NVAs használatával.

<!-- links -->
[cloud-security]: /azure/best-practices-network-security
[dmz-on-prem]: ./secure-vnet-hybrid.md
[dmz-internet]: ./secure-vnet-dmz.md
[egress-with-layer-7]: #egress-with-layer-7-nvas
[ingress-with-layer-7]: #ingress-with-layer-7-nvas
[ingress-egress-with-layer-7]: #ingress-egress-with-layer-7-nvas
[lb-overview]: /azure/load-balancer/load-balancer-overview/
[nva-scenario]: /azure/virtual-network/virtual-network-scenario-udr-gw-nva/
[pip-udr-switch]: #pip-udr-switch-with-layer-4-nvas
[udr-overview]: /azure/virtual-network/virtual-networks-udr-overview/
[zookeeper]: https://zookeeper.apache.org/

<!-- images -->
[0]: ./images/nva-ha/single-nva.png "Egyetlen NVA architektúrája"
[1]: ./images/nva-ha/l7-ingress.png "Réteg 7 érkező"
[2]: ./images/nva-ha/l7-ingress-egress.png "Réteg 7 kimenő forgalom"
[3]: ./images/nva-ha/active-passive.png "Aktív-passzív fürt"
[4]: ./images/nva-ha/l7-ingress-egress-ag.png
