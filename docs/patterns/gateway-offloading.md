---
title: Átjáró Offloading minta
description: Megosztott vagy speciális szolgáltatás funkcióit egy átjáró proxyként kiürítése.
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: 6b3e4541aae77349ca91c18c788ddb508912361d
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
ms.locfileid: "24540009"
---
# <a name="gateway-offloading-pattern"></a>Átjáró Offloading minta

Megosztott vagy speciális szolgáltatás funkcióit egy átjáró proxyként kiürítése. Ebben a mintában alkalmazásfejlesztés egyszerűsítheti megosztott szolgáltatás funkciókat, például az SSL-tanúsítványok tér át az alkalmazást az átjáró többi részével.

## <a name="context-and-problem"></a>A környezetben, és probléma

Néhány gyakran használják több szolgáltatásban, és ezek a szolgáltatások konfigurációs, felügyeleti és karbantartási szükséges. Egy megosztott vagy speciális szolgáltatás, minden alkalmazás központi telepítése terjesztett növeli az adminisztrációs terhelést, és növeli az elágazás a központi telepítési hiba. A megosztott szolgáltatás frissítéseit is, hogy az összes szolgáltatásban kell telepíteni.

Megfelelően kezeli a biztonsági problémák (jogkivonat érvényesítésére, titkosítás, SSL-tanúsítványok kezelését) és egyéb összetett feladatokat megkövetelheti, hogy a csoport tagjai magas speciális ismeretek rendelkezik. Például egy alkalmazás által igényelt tanúsítvány kell kell konfigurált és telepített összes alkalmazás-példányokon. Az egyes új központi telepítéséhez annak érdekében, hogy azt nem jár le a tanúsítványt kell kezelni. Bármely általános tanúsítvány lejárati dátuma frissíteni kell, tesztelése és ellenőrzése minden alkalmazás üzembe helyezés.

Más közös szolgáltatásokon, például a hitelesítési, engedélyezési, naplózási, figyelés, vagy [sávszélesség-szabályozás](./throttling.md) megvalósítása és kezelése a központi telepítések számos különböző nehéz lehet. Jobb megoldás lehet vonják össze a funkciót, az ilyen típusú munkaterhet és a hibák esélyét csökkentése érdekében.

## <a name="solution"></a>Megoldás

Néhány funkció kiszervezése be egy API-átjárót, különösen több területet vonatkozik, például a tanúsítványkezelés, hitelesítést, SSL lezárást, figyelés, protokollfordításhoz, és sávszélesség-szabályozás. 

Az alábbi ábrán egy API-átjáró, amely megszakítja bejövő SSL-kapcsolatot. Az eredeti kérelmezőnek bármely HTTP-kiszolgáló előtt az API-átjáró nevében adatokat kér.

 ![](./_images/gateway-offload.png)
 
Ez a minta a következő előnyöket nyújtja:

- Leegyszerűsítheti a szolgáltatások fejlesztéséhez kell terjeszteni, és a támogató erőforrások, például a webkiszolgáló-tanúsítványok és a biztonságos webhely konfigurációja karbantartása eltávolításával. Egyszerűbb konfiguráció egyszerűbb kezelés és a méretezhetőség eredményez, és így egyszerűbb frissítése.

- Dedikált csoportok megvalósításához speciális szakértelem, például a biztonsági igénylő szolgáltatások engedélyezése. Ez lehetővé teszi, hogy az alapvető csapat az alkalmazás funkciói, hagyja a speciális, de több területet problémák a megfelelő szakértőknek összpontosíthat.

- Adja meg egyes konzisztencia kérés- és naplózás és figyelés. Akkor is, ha a szolgáltatás nem megfelelően tagolva, az átjáró beállítható úgy, hogy a figyelés és naplózás minimális szintjét.

## <a name="issues-and-considerations"></a>Problémákat és szempontok

- Győződjön meg arról az API-átjáró magas rendelkezésre állású és rugalmas hiba esetén. Kerülje a hibaérzékeny pontokat által az API-átjáró több példányát futtatni. 
- Győződjön meg arról, az átjáró a kapacitás készült, és a méretezésről az alkalmazás- és végpontok követelményeinek. Győződjön meg arról, hogy az átjáró nem keresztmetszetet jelenthet az alkalmazáshoz, és megfelelően méretezhető.
- A teljes alkalmazás, például biztonsági vagy átviteli által használt funkciók csak kiürítése.
- Az API-átjárónak soha nem kell kiszervezett üzleti logikát. 
- Ha tranzakciók nyomon van szüksége, fontolja meg, generálásához. korrelációs azonosító naplózási célokra.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes használni ezt a mintát

Ez mintát, mikor használja:

- Az alkalmazás központi telepítése egy megosztott szempont, például az SSL-tanúsítványok és titkosítás rendelkezik.
- Ez a szolgáltatás által közösen alkalmazások központi telepítéseit, amelyek különböző erőforrás-követelményei, például a memória-erőforrásokat, a tárolási kapacitás vagy a hálózati kapcsolatok.
- Helyezze át az ezzel kapcsolatos problémák, például a hálózati biztonság, a sávszélesség-szabályozás és a más hálózati határ észrevételeivel konkrétabb team kívánja.

Előfordulhat, hogy ez a minta nem megfelelő, ha a kapcsoló szolgáltatásban okozna.

## <a name="example"></a>Példa

A Nginx-t az SSL kiszervezési akkor, a következő konfigurációs bejövő SSL-kapcsolat leállítása, majd továbbítja a három fölérendelt HTTP-kiszolgálóhoz csatlakozni.

```
upstream iis {
        server  10.3.0.10    max_fails=3    fail_timeout=15s;
        server  10.3.0.20    max_fails=3    fail_timeout=15s;
        server  10.3.0.30    max_fails=3    fail_timeout=15s;
}

server {
        listen 443;
        ssl on;
        ssl_certificate /etc/nginx/ssl/domain.cer;
        ssl_certificate_key /etc/nginx/ssl/domain.key;

        location / {
                set $targ iis;
                proxy_pass http://$targ;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto https;
proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header Host $host;
        }
}
```

## <a name="related-guidance"></a>Kapcsolódó útmutató

- [Háttérkiszolgálókon Frontends minta](./backends-for-frontends.md)
- [Átjáró összesítési minta](./gateway-aggregation.md)
- [Átjáró útválasztás minta](./gateway-routing.md)

