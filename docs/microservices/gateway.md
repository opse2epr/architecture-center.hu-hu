---
title: API-átjárók
description: A mikroszolgáltatás-alapú API-átjárókat.
author: MikeWasson
ms.date: 10/23/2018
ms.openlocfilehash: cd13f9a36120bb0093675e1dec683d480e168db5
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/08/2019
ms.locfileid: "54113263"
---
# <a name="designing-microservices-api-gateways"></a>Mikroszolgáltatások tervezése: API-átjárók

A mikroszolgáltatási architektúra, az ügyfél egynél több előtér-szolgáltatás előfordulhat, hogy dolgozhat. Adja meg a tény, hogyan nem ügyfél, hogy milyen végpontok hívása? Mi történik, ha új szolgáltatások jelennek meg, vagy a meglévő szolgáltatások vannak újratervezhetők? Hogyan tegye kezelni a szolgáltatások SSL megszüntetése, hitelesítés és az egyéb problémákat? Egy *API-átjáró* ezek a kihívások megoldására segítségével.

![API-átjáró ábrája](./images/gateway.png)

<!-- markdownlint-disable MD026 -->

## <a name="what-is-an-api-gateway"></a>Mi az API-átjáró?

<!-- markdownlint-enable MD026 -->

API-átjáró helyezkedik el, az ügyfelek és a szolgáltatások között. Fordított proxy, útválasztási szolgáltatások ügyfelektől érkező kérelmek funkcionál. Emellett végrehajthatnak, például a hitelesítést, az SSL-lezárást és a sebességkorlátozás különböző általános feladatok. Ha nem telepít egy átjárót, az ügyfelek kell küldenie kérések közvetlenül előtér-szolgáltatásokat. Vannak azonban bizonyos adatokhoz hozzáférést biztosító szolgáltatások közvetlenül az ügyfelek kapcsolatos lehetséges problémákat:

- Összetett Ügyfélkód eredményezhet. Az ügyfél kell nyomon követheti, több végpontot, és rugalmas módon hibáinak a kezelése.
- Az ügyfél és a háttérkiszolgáló közötti hoz létre. Az ügyfélnek tudnia kell, hogyan vannak bontjuk le az egyes szolgáltatásokat. Így nehezebb karbantartása az ügyfél és a is nehezebb újrabontása szolgáltatásokhoz.
- Egyetlen művelet több hívások lehet szükség. Hogy több hálózati eredményt is kerek lelassítja az ügyfél és a kiszolgáló jelentős késleltetés hozzáadása között.
- Minden egyes nyilvános szolgáltatás például a hitelesítés, az SSL és az ügyfél sebességkorlátozással aggályokat kell kezelnie.
- Szolgáltatások elérhetővé kell tenni például HTTP vagy a WebSocket protokoll ügyfél-barát. Ezzel a megoldással a kiválasztott [kommunikációs protokollok](./interservice-communication.md).
- Nyilvános végpontokkal rendelkező szolgáltatások egy potenciális támadási felületet, és a megerősített kell lennie.

Az átjáró a következő problémák ügyfeleket a szolgáltatásoktól leválasztásával segítségével. Átjárók hajthat végre számos különböző függvényt, és előfordulhat, hogy nem kell azokat. A functions a következő tervezési minták sorolhatók:

[Átjáró útválasztási](../patterns/gateway-routing.md). Az átjáró használatára, irányíthatja a fordított proxy kéri, hogy egy vagy több háttérszolgáltatások 7. rétegbeli útválasztási használatával. Az átjáró egyetlen végpontot biztosít az ügyfelek számára, és különítse el a ügyfeleket a szolgáltatásoktól segítségével.

[Átjáró-összesítési](../patterns/gateway-aggregation.md). Az átjáró használatával egyetlen több egyéni kérést összesíteni. Ez a minta akkor érvényes, ha egyetlen művelet több háttérszolgáltatással hívásainak igényel. Az ügyfél egy kérést küld az átjárónak. Az átjáró továbbítja a kéréseket a különböző háttérrendszerekhez, majd összesíti az eredményeket és elküldi őket az ügyfélnek. Ez segít csökkenteni az ügyfél és a háttérkiszolgáló közötti forgalmat.

[Átjáró-kiürítés](../patterns/gateway-offloading.md). Az átjáró használatával kiszervezheti az átjáróhoz, az egyes szolgáltatások funkcióit különösen az általános problémákat. Ezek a függvények kialakíthattunk egy helyen, ahelyett minden szolgáltatás felelős azok megvalósítását, hasznos lehet. Ez különösen igaz funkciók megvalósításához, például a hitelesítési és engedélyezési speciális ismeretek szükségesek.

Íme néhány példa, amely az átjáró kiszervezhető:

- SSL leállítása
- Hitelesítés
- IP-címei engedélyezési listára helyezhetők
- Ügyfél sebessége (szabályozás) korlátozása
- Naplózás és figyelés
- Válasz-gyorsítótárazás
- Web application firewall (Webalkalmazási tűzfal)
- A GZIP-tömörítés
- Statikus tartalom karbantartása

## <a name="choosing-a-gateway-technology"></a>Egy átjáró technológia kiválasztása

Íme néhány lehetőség a API-átjáró implementálása az alkalmazásban.

- **Fordított proxy server**. Nginx HAProxy népszerű fordított proxy kiszolgálók, amelyek támogatják a terheléselosztást, SSL-t, funkciókat és útválasztási 7. réteg. Mindkettő ingyenes, nyílt forráskódú termékek, további alkalmazásszolgáltatások biztosítása érdekében, és a támogatási lehetőségek fizetős kiadásra. Nginx-et és a HAProxy olyan gazdag szolgáltatáskészleteket és nagy teljesítményű mindkét érett termékhez. Külső modulról, illetve egyéni szkriptek Lua őket bővítheti. Az Nginx NginScript nevű parancsfájl-kezelési JavaScript-alapú modul is támogat.

- **Szolgáltatás háló bejövőforgalom-vezérlőjéhez**. Ha például linkerd vagy Istio szolgáltatás rácsvonal használ, fontolja meg az a Funkciók, az adott szolgáltatás háló a bejövőforgalom-vezérlőjéhez által biztosított. Például a Istio bejövőforgalom-vezérlőjéhez támogatja, 7. rétegbeli útválasztási, HTTP átirányítások, az újrapróbálkozások és egyéb funkciókkal.

- [Az Azure Application Gateway](/azure/application-gateway/). Egy felügyelt terheléselosztási szolgáltatás, amely a 7. rétegbeli útválasztási és SSL-lezárást végezheti el. Webalkalmazási tűzfal (WAF) is tartalmazza.

- [Az Azure API Management](/azure/api-management/). Az API Management egy kulcsrakész közzé API-kat a külső és belső ügyfelek számára az. Egy nyilvánosan elérhető API-t, beleértve a sebességkorlátozást, IP-fehér listázása és hitelesítés az Azure Active Directory vagy az egyéb identitás-szolgáltatóktól felügyelete esetén hasznosak funkciókat biztosít. Az API Management végre bármely terheléselosztást, így például az Application Gateway vagy fordított terheléselosztó együtt kell használni. Az API Management használatával az Application Gateway szolgáltatással kapcsolatos további információkért lásd: [az API Management integrálása belső vnet-en az Application Gatewayen](/azure/api-management/api-management-howto-integrate-internal-vnet-appgateway).

Átjáró technológia kiválasztásakor vegye figyelembe a következőket:

**Szolgáltatások**. A felsorolt lehetőségek felett támogatja a 7. rétegbeli útválasztási, de egyéb funkciói eltérnek támogatása. Attól függően, a szükséges szolgáltatások egynél több átjáró központi telepítését.

**Üzembe helyezés**. Az Azure Application Gateway és az API Management a felügyelt szolgáltatások. Nginx-et és a HAProxy általában a fürtben lévő tárolók fog futni, de is telepíthetők a fürtön kívüli dedikált virtuális gépek. Az átjáró többi részét a munkaterhelésből elkülöníti, de magasabb felügyeleti többletterhelést okoz.

**Felügyelet**. Szolgáltatások frissülnek, vagy új szolgáltatással bővül, az átjáró útválasztási szabályokat lehet kell frissíteni kell. Fontolja meg, hogyan fogja kezelni ezt a folyamatot. SSL-tanúsítványok, IP-listáinak és egyéb aspektusait konfiguráció kezelése hasonló szempontok vonatkoznak.

## <a name="deploying-nginx-or-haproxy-to-kubernetes"></a>Nginx-et vagy a HAProxy Kubernetes üzembe helyezése

Telepítheti az Nginx vagy HAProxy Kubernetes egy [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/) vagy [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) , amely meghatározza, hogy az nginx-et vagy a HAProxy tárolórendszerképet. Egy ConfigMap használatával tárolja a konfigurációs fájlt a proxy, és csatlakoztassa a ConfigMap kötetként. Hozzon létre az átjáró egy Azure Load Balanceren keresztül elérhetővé terheléselosztó típusú szolgáltatást.

Alternatív, hogy hozzon létre egy Bejövőforgalom-vezérlőt. Bejövőforgalom-vezérlőt egy Kubernetes-erőforrás, amely telepít egy terheléselosztót vagy fordított proxykiszolgáló. Számos implementáció létezik, például a HAProxy és az nginx-et. Bejövő forgalmi nevű külön erőforrást Bejövőforgalom-vezérlőt, például az útválasztási szabályokat és a TLS-tanúsítványok beállításait határozza meg. Ezzel a módszerrel nem kell összetett konfigurációs fájlokat, amelyek egy adott proxy server technológiára jellemző kezelése.

Az átjáró nem a potenciális szűk keresztmetszetet vagy rendszerkritikus meghibásodási pontot a rendszer, így mindig a magas rendelkezésre állás érdekében legalább két replika üzembe. Szükség lehet további, a replikák horizontális terhelésétől függően.

Emellett fontolja meg az átjáró fut a fürtben található csomópontok külön készlete. Ez a módszer előnyei a következők:

- Elszigetelés. Minden bejövő forgalom csomópontokhoz, amelyek el lehet különíteni a háttérszolgáltatások készletét kerül.

- Stabil konfiguráció. Ha az átjáró hibásan van konfigurálva, a teljes alkalmazás elérhetetlenné válhat.

- Teljesítmény. Előfordulhat, hogy szeretné egy adott Virtuálisgép-konfigurációt használja a megfelelő teljesítmény biztosítása érdekében az átjáró.

> [!div class="nextstepaction"]
> [Naplózás és figyelés](./logging-monitoring.md)
