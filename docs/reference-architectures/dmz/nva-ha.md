---
title: Magas rendelkezésre állású virtuális hálózati berendezések üzembe helyezése
titleSuffix: Azure Reference Architectures
description: Magas rendelkezésre állású virtuális berendezések üzembe helyezéséről.
author: telmosampaio
ms.date: 12/08/2018
ms.custom: seodec18
ms.openlocfilehash: d3f9017db1bbf9741b10db16eb5a3dbab78f1160
ms.sourcegitcommit: 7d21aec9d9de0004ac777c1d1e364f53aac2350d
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/09/2018
ms.locfileid: "53120752"
---
# <a name="deploy-highly-available-network-virtual-appliances"></a>Magas rendelkezésre állású virtuális hálózati berendezések üzembe helyezése

Ez a cikk a magas rendelkezésre állású hálózati virtuális berendezések (network virtual appliance, NVA) Azure-ban való üzembe helyezésének módját ismerteti. Az NVA-kat általában a szegélyhálózatokról, más néven DMZ-kről a más hálózatok és alhálózatok felé irányuló hálózati forgalom szabályozására használják. Az DMZ Azure-ban történő implementálásával kapcsolatos további tudnivalókért lásd a [Microsoft-felhőszolgáltatásokkal és a hálózati biztonsággal][cloud-security] foglalkozó cikket. A cikk csak bejövő, csak kimenő, valamint bejövő és kimenő forgalmú példaarchitektúrákat is tartalmaz.

**Előfeltételek:** A jelen cikk feltételezi, hogy az olvasó alapszinten érti az Azure-hálózatok, az [Azure-terheléselosztók][lb-overview] és a [felhasználó által definiált útvonalak][udr-overview] (UDR-ek) működését.

## <a name="architecture-diagrams"></a>Architektúra-diagramok

Az NVA-k számos különböző architektúrában helyezhetők üzembe egy DMZ-n. A következő ábra például [egyetlen NVA][nva-scenario] bejövő forgalomhoz való használatát illusztrálja.

![[0]][0]

Ebben az architektúrában az NVA biztonságos hálózati határt biztosít az összes bejövő és kimenő hálózati forgalom ellenőrzésével, és csak azt a forgalmat engedi át, amely megfelel a hálózati biztonsági szabályoknak. Az a tény azonban, hogy az összes hálózati forgalom az NVA-n halad keresztül, azt eredményezi, hogy az NVA rendszerkritikus meghibásodási pontot képez a hálózatban. Ha az NVA leáll, akkor nincs más útvonal a hálózati forgalom számára, és a háttérbeli alhálózatok nem lesznek elérhetők.

Az NVA magas rendelkezésre állásúvá tételéhez helyezzen üzembe több NVA-t egy rendelkezésre állási csoport részeként.

A következő architektúrák bemutatják a magas rendelkezésre állású NVA-khoz szükséges erőforrásokat és konfigurációkat:

| Megoldás | Előnyök | Megfontolandó szempontok |
| --- | --- | --- |
| [Bejövő forgalom 7-es rétegű NVA-kkal][ingress-with-layer-7] |Az összes NVA-csomópont aktív |Kapcsolatok leállítására és SNAT használatára képes NVA-t igényel<br/> Külön NVA-készletet igényel az internetről és az Azure-ból érkező forgalomhoz <br/> Csak az Azure-on kívülről származó forgalom kezelésére használható |
| [Kimenő forgalom 7-es rétegű NVA-kkal][egress-with-layer-7] |Az összes NVA-csomópont aktív | Kapcsolatok leállítására és forráshálózati címfordítás (source network address translation, SNAT) implementálására képes NVA-t igényel
| [Bejövő és kimenő forgalom 7-es rétegű NVA-kkal][ingress-egress-with-layer-7] |Az összes csomópont aktív<br/>Képes kezelni az Azure-ból eredő forgalmat |Kapcsolatok leállítására és SNAT használatára képes NVA-t igényel<br/>Külön NVA-készletet igényel az internetről és az Azure-ból érkező forgalomhoz |
| [PIP-UDR kapcsoló][pip-udr-switch] |Egyetlen NVA-készlet az összes forgalomhoz<br/>Az összes forgalom kezelésére képes (nincsenek korlátozva a portszabályok) |Aktív-passzív<br/>Feladatátvételi folyamatot igényel |
| [PIP-UDR SNAT](#pip-udr-nvas-without-snat) | Egyetlen NVA-készlet az összes forgalomhoz<br/>Az összes forgalom kezelésére képes (nincsenek korlátozva a portszabályok)<br/>Nincs szükség a bejövő kéréseket SNAT konfigurálása |Aktív-passzív<br/>Feladatátvételi folyamatot igényel<br/>Tesztelés és a feladatátvételi logika futtatása a virtuális hálózaton kívül |

## <a name="ingress-with-layer-7-nvas"></a>Bejövő forgalom 7-es rétegű NVA-kkal

Az alábbi ábrán egy magas rendelkezésre állású architektúra látható, amely egy internetkapcsolattal rendelkező terheléselosztó mögötti bejövő forgalmi DMZ-t implementál. Ezt az architektúrát arra tervezték, hogy 7-es rétegű forgalmi kapcsolatot, például HTTP-t vagy HTTPS-t biztosítson Azure-beli számítási feladatokhoz:

![[1]][1]

Az ilyen architektúra előnye, hogy minden NVA aktív, és ha az egyik meghibásodik, akkor a terheléselosztó átirányítja a hálózati forgalmat egy másikra. Mindkét NVA a belső terheléselosztóra irányítja a forgalmat, így mindaddig, amíg egy NVA aktív, a forgalom nem akad el. Az NVA-knak le kell zárnia a webes szintű virtuális gépek felé tartó SSL-forgalmat. Ezek az NVA-k nem használhatók a helyszíni forgalom kezelésére, mert a helyszíni forgalom egy másik, saját hálózati útvonalakkal rendelkező dedikált NVA-készletet igényel.

> [!NOTE]
> Ezt az architektúrát az [Azure és a helyszíni adatközpont közötti DMZ][dmz-on-prem] és az [Azure és az internet közötti DMZ][dmz-internet] referenciaarchitektúrákban használják. Ezen referenciaarchitektúrák mindegyike tartalmaz egy felhasználható üzembehelyezési megoldást. További információkért kövesse a hivatkozásokat.

## <a name="egress-with-layer-7-nvas"></a>Kimenő forgalom 7-es rétegű NVA-kkal

Az előző architektúra bővíthető oly módon, hogy egy kimenő forgalmi DMZ-t is biztosítson az Azure-beli számítási feladatokból származó kérelmek kezelésére. A következő architektúrát arra tervezték, hogy magas rendelkezésre állást biztosítson a DMZ-n lévő NVA-k számára 7-es rétegű forgalom (például HTTP vagy HTTPS) esetén:

![[2]][2]

Ebben az architektúrában az Azure-ból származó összes forgalom át van irányítva egy belső terheléselosztóra. A terheléselosztó elosztja a kimenő kérelmeket egy NVA-készlet tagjai között. Ezek az NVA-k az internetre irányítják a forgalmat saját nyilvános IP-címeik használatával.

> [!NOTE]
> Ezt az architektúrát az [Azure és a helyszíni adatközpont közötti DMZ][dmz-on-prem] és az [Azure és az internet közötti DMZ][dmz-internet] referenciaarchitektúrákban használják. Ezen referenciaarchitektúrák mindegyike tartalmaz egy felhasználható üzembehelyezési megoldást. További információkért kövesse a hivatkozásokat.

## <a name="ingress-egress-with-layer-7-nvas"></a>Bejövő és kimenő forgalom 7-es rétegű NVA-kkal

A két előző architektúrában külön DMZ tartozott a bejövő és a kimenő forgalomhoz. A következő architektúra bemutatja, hogyan kell létrehozni olyan DMZ-t, amely egyaránt használható a bejövő és kimenő 7-es rétegű forgalomhoz (például HTTP-hez vagy HTTPS-hez):

![[4]][4]

Ebben az architektúrában az NVA-k az alkalmazásátjáróról érkező kérelmeket dolgozzák fel. Az NVA-k a terheléselosztó háttérkészletében található, számítási feladatokat végző virtuális gépek kimenő kérelmeit is feldolgozzák. Mivel a bejövő forgalmat a rendszer alkalmazásátjáróval, a kimenőt pedig terheléselosztóval irányítja, az NVA-k felelősek a munkamenet affinitásának fenntartásáért. Ez azt jelenti, hogy az alkalmazásátjáró leképezi a bejövő és a kimenő kérelmeket, így továbbíthatja a helyes választ az eredeti kérelmezőnek. Azonban a belső terheléselosztó nem fér hozzá az alkalmazásátjáró leképezéseihez, így a saját logikáját használja, amikor választ küld az NVA-knak. Előfordulhat, hogy a terheléselosztó nem annak az NVA-nak küld választ, amely eredetileg megkapta a kérelmet az alkalmazásátjáróról. Ebben az esetben az NVA-knak kommunikálniuk kell, és továbbítaniuk kell egymás között a választ, hogy a megfelelő NVA küldhesse tovább a választ az alkalmazásátjáróra.

> [!NOTE]
> Úgy is megoldhatja az aszimmetrikus útvonal-választási problémát, ha gondoskodik róla, hogy az NVA-k bejövő forráshálózati címfordítást (SNAT) végezzenek. Ez lecserélné a kérelmező eredeti forrás IP-címét az NVA által a bejövő adatfolyamon használt IP-címeinek egyikével. Így gondoskodni lehet róla, hogy több NVA is használható legyen egyszerre az útvonal szimmetriájának megőrzése mellett.

## <a name="pip-udr-switch-with-layer-4-nvas"></a>PIP-UDR kapcsoló 4-es rétegű NVA-kkal

Az alábbi szakasz egy aktív és egy passzív NVA-val rendelkező architektúrát mutat be. Ez az architektúra mind a bejövő, mind a kimenő 4-es rétegű forgalmat kezeli:

![[3]][3]

> [!TIP]
> Ez az architektúra teljes körű megoldást érhető el a [GitHub][pnp-ha-nva].

Ez az architektúra hasonlít a cikkben elsőként bemutatott architektúrára. Az egyetlen NVA-t tartalmazott, amely a 4-es rétegű beérkező kérelmek fogadását és szűrését végezte. Ez az architektúra mindezt egy második passzív NVA-val egészíti ki a magas rendelkezésre állás érdekében. Ha az aktív NVA meghibásodik, a passzív NVA aktiválódik, és az UDR, valamint a PIP úgy módosul, hogy az imént aktivált NVA hálózati adaptereire mutasson. Az UDR és a PIP ezen módosításai manuálisan vagy egy automatizált folyamattal is elvégezhetők. Az automatizált folyamat jellemzően egy démon, vagy egy másik Azure-beli figyelőszolgáltatás, amely végrehajtja az aktív NVA állapotvizsgálatát, majd átkapcsolja az UDR-t és a PIP-et, amikor az NVA meghibásodását észleli.

A fenti ábrán egy például szolgáló [ZooKeeper][zookeeper]-fürt látható, amely egy magas rendelkezésre állású démont biztosít. A ZooKeeper-fürtön belül egy csomópontkvórum kiválaszt egy vezetőt. Ha a vezető meghibásodik, akkor a többi csomópont választást tart egy új vezető kinevezésére. A jelen architektúra esetében a vezető csomópont végrehajtja a démont, amely lekérdezi az NVA üzemállapoti végpontját. Ha az NVA nem válaszol az állapotvizsgálatra, akkor a démon aktiválja a passzív NVA-t. Ezt követően a démon lehívja az Azure REST API-t a PIP eltávolításához a meghibásodott NVA-ból, majd csatolja azt az újonnan aktivált NVA-hoz. A démon ezután módosítja az UDR-t, hogy az újonnan aktivált NVA belső IP-címére mutasson.

Ne vegyen fel ZooKeeper-csomópontokat olyan alhálózatba, amely csak olyan útvonalon keresztül érhető el, amely tartalmazza az NVA-t. Ha mégis így tesz, a ZooKeeper-csomópontok nem lesznek érhetők az NVA meghibásodásakor. Ha a démon bármilyen okból sikertelen, akkor nem fogja tudni használni a ZooKeeper-csomópontokat a probléma diagnosztizálására.

A teljes megoldás, beleértve a mintakódot, olvassa el a fájlokat a [GitHub-adattár][pnp-ha-nva].

## <a name="pip-udr-nvas-without-snat"></a>A PIP-UDR nva-k SNAT

Ez az architektúra két Azure-beli virtuális gépek futtatásához az NVA tűzfala egy aktív-passzív konfigurációt, amely támogatja az Automatikus feladatátvétel, de nem szükséges a forrás hálózati cím címfordítás (SNAT) használja.

![PIP-UDR nva-k nélkül SNAT-architektúra](./images/nva-ha/pip-udr-without-snat.png)

> [!TIP]
> Ez az architektúra teljes körű megoldást érhető el a [GitHub][ha-nva-fo].

Ez a megoldás Azure ügyfelek, akik SNAT nem lehet konfigurálni a bejövő kérelmekre a saját NVA tűzfalak lett tervezve. SNAT elrejti az eredeti forrás ügyfél IP-címe. Ha be kell jelentkeznie az eredeti IP-címek vagy az nva-k mögött belül más többrétegű biztonsági összetevők használni őket, ez a megoldás egyszerű megközelítést kínál.

A következő ugrás címét az aktív NVA tűzfal virtuális gép hálózati adapteréhez IP-címét állítsa által automatizált UDR táblabejegyzéseket feladatátvétele. Az automatikus feladatátvételi logika használatával létrehozott függvényalkalmazás szolgáltatásban üzemeltetett [Azure Functions](/azure/azure-functions/). A feladatátvételi kódja fut, mint belül az Azure Functions egy kiszolgáló nélküli függvény. Üzembe helyezés kényelmesen, költségkímélő és könnyen kezelése és testreszabása. Emellett a függvényalkalmazás üzemeltetett belül az Azure Functions, így egy csoporttól sem a virtuális hálózaton. Ha a virtuális hálózatot milyen hatással vannak az NVA-tűzfalak, a függvényalkalmazás továbbra is futtatható önállóan. Tesztelés azért pontosabb, valamint az ugyanazon útvonalat használja, mint a bejövő ügyfélkérelmek a virtuális hálózaton kívül történik.

Az nva-t tűzfal rendelkezésre állásának ellenőrzéséhez a függvénykódot alkalmazás mintavételek a két módszer egyikével:

- Az Azure-beli virtuális gépek állapotának figyelésével az nva-t tűzfal üzemeltetéséhez.

- Tesztelésével nyitott portot a tűzfalon a háttér-webkiszolgálóhoz van-e. Ezt a beállítást, a az nva-n egy szoftvercsatorna tesztelése a függvény-alkalmazás kódjához PIP-n keresztül kell közzétennie.

Válassza a mintavétel a függvényalkalmazás konfigurálásakor használni kívánt típusát. A teljes megoldás, beleértve a mintakódot, olvassa el a fájlokat a [GitHub-adattár][ha-nva-fo].

## <a name="next-steps"></a>További lépések

- Megtudhatja, hogyan [implementálhat DMZ-t az Azure és a helyszíni adatközpont között][dmz-on-prem] 7-es rétegű NVA-k használatával.
- Megtudhatja, hogyan [implementálhat DMZ-t az Azure és az internet között][dmz-internet] 7-es rétegű NVA-k használatával.
- [Az Azure-ban a hálózati virtuális berendezés hibák elhárítása](/azure/virtual-network/virtual-network-troubleshoot-nva)

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
[pnp-ha-nva]: https://github.com/mspnp/ha-nva
[ha-nva-fo]: https://aka.ms/ha-nva-fo

<!-- images -->

[0]: ./images/nva-ha/single-nva.png "Egyetlen NVA-ból álló architektúra"
[1]: ./images/nva-ha/l7-ingress.png "7-es rétegű bejövő forgalom"
[2]: ./images/nva-ha/l7-ingress-egress.png "7-es rétegű kimenő forgalom"
[3]: ./images/nva-ha/active-passive.png "Aktív-passzív fürt"
[4]: ./images/nva-ha/l7-ingress-egress-ag.png
