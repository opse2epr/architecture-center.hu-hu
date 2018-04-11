---
title: API-átjárókkal
description: Mikroszolgáltatások átjárói API
author: MikeWasson
ms.date: 12/08/2017
ms.openlocfilehash: 6483d416363e24f4084d6b856847a740bf4054d9
ms.sourcegitcommit: a8453c4bc7c870fa1a12bb3c02e3b310db87530c
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/29/2017
---
# <a name="designing-microservices-api-gateways"></a>Mikroszolgáltatások tervezése: API átjárók

Mikroszolgáltatások architektúra esetén előfordulhat, hogy módosításának ügyfél egynél több előtér-szolgáltatás. Megadott ezt a tényt, hogyan nem ügyfelet, hogy milyen végpontok hívására? Mi történik, ha az új szolgáltatások új vagy meglévő szolgáltatások átkerült vannak? Hogyan tegye kezeli a szolgáltatások SSL lezárást, hitelesítési és más okok? Egy *API átjáró* ezekkel a kihívásokkal megoldására segítségével. 

![](./images/gateway.png)

## <a name="what-is-an-api-gateway"></a>Mi az az API-átjáró?

Az API-átjáró ügyfelek és a szolgáltatások közötti helyezkedik el. Úgy működik, mint egy fordított proxy services ügyfelek útválasztási kérelmeit. Több területet különböző feladatokat, mint a hitelesítés, az SSL-lezárást és sebességkorlátozást azt is végrehajthatja. Ha nem telepít egy átjárót, az ügyfelek kell kérelmet küldeni közvetlenül előtér-szolgáltatásokat. Van azonban néhány jelentkezik, mintha közvetlenül az ügyfelek számára szolgáltatásokat érintő lehetséges problémákat:

- Összetett Ügyfélkód eredményezhet. Az ügyfél kell nyomon követésére több végpontot, és rugalmas módon kijavíthassa a hibákat. 
- Az ügyfél és a háttérkiszolgáló közötti hoz létre. Az ügyfélnek tudnia kell, hogyan vannak kiválasztott az adott szolgáltatást. Amelyről nehezebb megőrzése az ügyfél és is nehezebben azonosítóterületen szolgáltatásokhoz.
- Egy művelettel több szolgáltatás hívásainak lehet szükség. Hogy több hálózati eredmény is kerekíteni között az ügyfél és a kiszolgáló hozzáadása jelentős késés való adatváltások számát. 
- Minden egyes nyilvánosan elérhető szolgáltatás vonatkozik, például a hitelesítés, az SSL és az ügyfél sebesség korlátozása kell kezelni. 
- Szolgáltatások elérhetővé kell tenni egy ügyfél-barát protokoll, például a http- vagy WebSocket. Ez korlátozza, hogy a választott [kommunikációs protokollokat](./interservice-communication.md). 
- Azokat a szolgáltatásokat, nyilvános végpontok egy potenciális támadási felületet, és a megerősített kell lennie.

Átjáró segítségével a következő problémák az ügynökök szolgáltatások leválasztásával. Átjárók számos különböző funkciót hajthat végre, és előfordulhat, hogy nem kell az összes. A függvény a következő kialakítási minta sorolhatók:

[Átjáró útválasztási](../patterns/gateway-routing.md). Az átjáró használatára, egy vagy több háttérbeli szolgáltatások használata réteg 7 útválasztási kérelem továbbításához fordított proxy. Az átjáró egy végpontot biztosít az ügyfelek számára, és segít használata leválasztja az ügyfelek szolgáltatásokból. 

[Átjáró összesítési](../patterns/gateway-aggregation.md). Segítségével az átjáró több egyes kérelmeket az egy kérelemhez. Ebben a mintában érvényes, ha egy művelettel több háttér-Services-hívások szükségesek. Az ügyfél egy kérést küld az átjárót. Az átjáró kiszállítja kéréseket a különböző háttér-szolgáltatások és aggregálja az eredményeket, majd elküldi az ügyfélnek. Ez segít csökkenteni az ügyfél és a háttérkiszolgáló közötti chattiness. 

[Átjáró kiszervezésével](../patterns/gateway-offloading.md). Az átjáró használatához kiszervezéséhez funkció az egyes szolgáltatások az átjáróra, különösen több területet vonatkozik. Egy helyen ahelyett, hogy ezek kivitelezésével felelős így minden szolgáltatás konszolidálhatják ezeket a funkciókat a hasznos lehet. Ez különösen igaz szolgáltatások megfelelően, például a hitelesítési és engedélyezési megvalósításához speciális ismeretek szükségesek. 

Az alábbiakban néhány olyan funkciót, amely egy átjáró kiszervezett sikerült:

- SSL-lezárást
- Hitelesítés
- IP-engedélyezése
- Ügyfél sebessége (sávszélesség-szabályozás) korlátozása
- Naplózás és figyelés
- Gyorsítótár válasz
- Web application firewall (Webalkalmazási tűzfal)
- A GZIP-tömörítés
- Statikus tartalom karbantartása

## <a name="choosing-a-gateway-technology"></a>Egy átjáró technológia kiválasztása

Íme néhány lehetőség egy API-átjáró megvalósítása az alkalmazásban.

- **Fordított proxykiszolgáló**. Nginx és a haproxy esetén támogatja a szolgáltatások, mint a hálózati terheléselosztást, SSL, népszerű fordított proxy-kiszolgálót, és 7 útválasztási réteg. Mindkettő ingyenes, nyílt forráskódú termékek, további lehetőségeket biztosítanak, és a beállítások támogató fizetett verzióval. Nginx és a haproxy esetén is gazdag szolgáltatáskészletek és nagy teljesítményű mindkét érett terméket. A külső modulok vagy egyéni parancsfájlok írásával a Lua őket kiterjesztheti. Nginx NginScript néven parancsfájl-kezelési JavaScript-alapú modul is támogatja.

- **Háló érkező szolgáltatásvezérlővel**. Ha például linkerd vagy Istio szolgáltatás rácsvonal használ, fontolja meg a biztosított szolgáltatásokat is a érkező vezérlő által az adott szolgáltatás háló. Például a Istio érkező vezérlő támogatja a réteg 7 útválasztási, HTTP átirányításokat, az újrapróbálkozások és egyéb szolgáltatásokat. 

- [Az Azure Alkalmazásátjáró](/azure/application-gateway/). Alkalmazásátjáró egy felügyelt terheléselosztási szolgáltatást, amely útválasztási réteg-7- és SSL-lezárást hajthatnak végre. Webalkalmazási tűzfal (WAF) is biztosít.

- [Az Azure API Management](/azure/api-management/). Az API Management az API-k közzétételéhez külső és belső ügyfeleknek olyan azonnal használható megoldás. Jól lehet egy nyilvánosan elérhető API, beleértve a sebességkorlátozást, listázása IP fehér és hitelesítés használata az Azure Active Directory vagy az egyéb identitás-szolgáltatóktól funkciókat biztosít. A terheléselosztást, API Management nem végrehajtani, így például az Alkalmazásátjáró vagy fordított proxy terheléselosztó együtt kell használni.

Átjáró technológia kiválasztásakor vegye figyelembe a következőket:

**Szolgáltatások**. A felsorolt lehetőségek mindenekelőtt réteg 7 útválasztást támogatnak, de egyéb funkciók változhatnak támogatása. Attól függően, hogy a szükséges szolgáltatások egynél több átjáró központi telepítését. 

**Központi telepítési**. Az Azure Application Gateway és API-kezelés a felügyelt szolgáltatások. Nginx és a haproxy esetén általában a fürtben található tárolók fog futni, de a fürtön kívüli dedikált virtuális gépek számára is telepíthető. Ez az átjáró többi részét a munkaterhelésből elkülöníti megoldás, ám magasabb kezelési terhelés mellett.

**Felügyeleti**. Szolgáltatások frissítése, vagy új szolgáltatással bővül, az átjáró útválasztási szabályok esetleg frissítenie kell. Vegye figyelembe, hogyan fogja kezelni ezt a folyamatot. Hasonló feltételek vonatkoznak, SSL-tanúsítványok, IP-whitelists és egyéb konfigurációs aspektusainak kezelése.

## <a name="deployment-considerations"></a>Telepítési szempontok

### <a name="deploying-nginx-or-haproxy-to-kubernetes"></a>Nginx vagy haproxy esetén történő telepítésének Kubernetes

Mint Kubernetes Nginx vagy haproxy esetén telepítheti egy [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/) vagy [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) , amely megadja, hogy a Nginx vagy haproxy esetén tároló-lemezképet. Egy ConfigMap a konfigurációs fájl tárolása a proxy, és csatlakoztassa a ConfigMap kötetként. Hozzon létre teszi közzé az átjárón keresztül az Azure Load Balancer terheléselosztó típusú szolgáltatást. 

<!-- - Configure a readiness probe that serves a static file from the gateway (rather than routing to another service). -->

A másik lehetőség az érkező vezérlőhöz létrehozásához. Az érkező vezérlőhöz, amely telepít egy terheléselosztót vagy fordított proxykiszolgáló Kubernetes erőforrás. Több megvalósítások létezik, például a Nginx és a haproxy esetén. Egy érkező nevű külön erőforrás érkező vezérlő, például az útválasztási szabályokat és a TLS-tanúsítványok beállításokat határoz meg. Ezzel a módszerrel nem kell összetett konfigurációs fájlok egy adott proxykiszolgáló server technológiára jellemző kezelése. Érkező tartományvezérlők továbbra is Kubernetes szolgáltatása beta írásának időpontjában, és a szolgáltatás továbbra is fejleszteni.

Az átjáró egy potenciális szűk vagy az kritikus hibapont a rendszerben, ezért mindig telepíteni a magas rendelkezésre állás érdekében legalább két replika. Szükség lehet további, a replikák horizontális terhelésétől függően. 

Figyelembe venni az átjáró fut a dedikált meg a fürtben lévő csomópontok. Ezt a módszert használja a következő előnyöket nyújtja:

- Elszigetelés. Minden bejövő forgalom kerül el lehet különíteni a háttér-szolgáltatások csomópontok készletét.

- Stabil konfigurációs. Ha az átjáró helytelenül van konfigurálva, a teljes alkalmazás elérhetetlenné válnának. 

- Teljesítmény. Érdemes lehet egy meghatározott Virtuálisgép-konfiguráció használata az átjáró teljesítményének javítására szolgál.

<!-- - Load balancing. You can configure the external load balancer so that requests always go to a gateway node. That can save a network hop, which would otherwise happen whenever a request lands on a node that isn't running a gateway pod. This consideration applies mainly to large clusters, where the gateway runs on a relatively small fraction of the total nodes. In Azure Container Service (ACS), this approach currently requires [ACS Engine](https://github.com/Azure/acs-engine)) which allows you to create multiple agent pools. Then you can deploy the gateway as a DaemonSet to the front-end pool. -->

### <a name="azure-application-gateway"></a>Azure Application Gateway

Csatlakozás az Azure-ban Kubernetes fürtre Alkalmazásátjáró:

1. A fürt virtuális hálózatot hozzon létre egy üres alhálózatot.
2. Alkalmazás-átjáró üzembe helyezéséhez.
3. Hozzon létre egy Kubernetes szolgáltatás típus =[NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport). Ez a szolgáltatás minden egyes csomóponton mutatja meg, hogy az elérhető a fürtön kívül. A terheléselosztó nem hoz létre.
5. A hozzárendelt portszámot a szolgáltatáshoz tartozó lekérdezése.
6. Vegyen fel egy Application Gateway ahol:
    - A háttérkészlet az ügynök virtuális gépeket tartalmaz.
    - A HTTP-beállítással a port számát.
    - 80/443-as porton figyeli a átjáró figyelő
    
Állítsa be a példányszám 2 vagy több, a magas rendelkezésre állás.

### <a name="azure-api-management"></a>Azure API Management 

Az API Management Kubernetes fürt az Azure-ban való csatlakozáshoz:

1. A fürt virtuális hálózatot hozzon létre egy üres alhálózatot.
2. Az adott alhálózat API Management központi telepítését.
3. Terheléselosztó típusú Kubernetes szolgáltatás létrehozása. Használja a [belső terheléselosztó](https://kubernetes.io/docs/concepts/services-networking/service/#internal-load-balancer) létrehozni a belső terheléselosztót, egy internetre irányuló terheléselosztót helyett az alapértelmezett jegyzet.
4. A privát IP-cím keresése a belső terheléselosztó kubectl vagy az Azure parancssori felület használatával.
5. Az API Management segítségével hozzon létre egy API-t, amely a saját IP-címet a terheléselosztó irányítja.

Fontolja meg az API Management kombinálásával fordított proxy, hogy Nginx, haproxy esetén vagy Azure Application Gateway. Az Alkalmazásátjáró API Management használatával kapcsolatos információkért lásd: [API Management integrálható egy belső virtuális Hálózatot az Alkalmazásátjáró](/azure/api-management/api-management-howto-integrate-internal-vnet-appgateway).

> [!div class="nextstepaction"]
> [Naplózás és figyelés](./logging-monitoring.md)
