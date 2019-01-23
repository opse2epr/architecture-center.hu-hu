---
title: Átjárókiszervezési minta
titleSuffix: Cloud Design Patterns
description: A megosztott vagy specializált szolgáltatásműködést kiszervezheti egy átjáró proxyra.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 28dbd1798060d1bdd01b1e6bee03337a2239a9ef
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54487661"
---
# <a name="gateway-offloading-pattern"></a>Átjárókiszervezési minta

A megosztott vagy specializált szolgáltatásműködést kiszervezheti egy átjáró proxyra. Ez a minta leegyszerűsítheti az alkalmazásfejlesztést a megosztott szolgáltatásfunkciók, például az SSL-tanúsítványok használatának az átjáróba való áthelyezésével.

## <a name="context-and-problem"></a>Kontextus és probléma

Bizonyos funkciókat több szolgáltatás is használ, és ezek a funkciók beállítást, felügyeletet és karbantartást igényelnek. A minden alkalmazástelepítéskor elosztott megosztott vagy speciális szolgáltatások növelik az adminisztrációs terhelést és a telepítési hibák esélyét. A megosztott funkciók frissítéseit a funkciót használó összes szolgáltatásban telepíteni kell.

A biztonsági problémák megfelelő kezelése (tokenérvényesítés, titkosítás, SSL-tanúsítványkezelés) és egyéb összetett feladatok elvégzése speciális szaktudást kívánhat meg. Például az egy alkalmazás számára szükséges tanúsítványt konfigurálni és telepíteni kell az alkalmazás összes példányán. Minden új telepítés esetében szükség van a tanúsítvány kezelésére, hogy az ne járjon le. Ha közeledik egy közös tanúsítvány lejárati ideje, akkor frissíteni, majd tesztelni és ellenőrizni kell azt minden alkalmazástelepítésen.

Más közös szolgáltatásokat, például a hitelesítést, az engedélyezést, a naplózást, a monitorozást vagy a [szabályozást](./throttling.md) nehéz implementálni és felügyelni nagy számú telepítés esetén. Érdemes lehet konszolidálni az ilyen típusú funkciókat a terhelés és a hibák esélyének csökkentése érdekében.

## <a name="solution"></a>Megoldás

Szervezzen ki egyes funkciókat egy API-átjáróba, különösen az általános feladatokat, mint a tanúsítványkezelés, a hitelesítés, az SSL-lezárás, a monitorozás, a protokollfordítás, valamint a szabályozás.

A következő ábrán egy API-átjáró látható, amely lezárja a bejövő SSL-kapcsolatokat. Az eredeti kérelmező nevében kér adatokat bármely, az API-átjárónál felsőbb rétegbeli HTTP-kiszolgálótól.

 ![Az átjáró-kiürítés minta ábrája](./_images/gateway-offload.png)

A minta használata többek között a következő előnyökkel jár:

- Egyszerűsíti a szolgáltatások fejlesztését, mivel nincs szükség a támogató erőforrások (például a webkiszolgáló-tanúsítványok és a biztonságos webhelyek konfigurációja) elosztására és karbantartására. Az egyszerűbb konfigurálás könnyebb felügyeletet és skálázhatóságot jelent, és így könnyebb a szolgáltatásfrissítés is.

- A speciális szakértelmet igénylő funkciók implementálását (például biztonság) bízza külön erre a célra létrehozott csapatokra. Így a központi csapat az alkalmazás működésére koncentrálhat, és a több szolgáltatást érintő, de speciális kérdések megoldását a szakértőkre hagyhatja.

- Egységesítse a kérések és válaszok naplózását és monitorozását. Beállíthatja, hogy az átjáró akkor is végezzen minimális monitorozási és naplózási tevékenységet, ha a szolgáltatás nem lett megfelelően kialakítva.

## <a name="issues-and-considerations"></a>Problémák és megfontolandó szempontok

- Gondoskodjon arról, hogy az API-átjáró magas rendelkezési állású legyen, és ne legyenek rá hatással a meghibásodások. Kerülje el a kritikus meghibásodási pontok kialakulását: futtassa az API-átjáró több példányát.
- Gondoskodjon arról, hogy az átjáró megfelel az alkalmazás és a végpontok kapacitási és skálázási követelményeinek. Gondoskodjon arról, hogy az átjáró ne váljon szűk keresztmetszetté az alkalmazás számára, és megfelelően skálázható legyen.
- Csak olyan funkciókat szervezzen ki, amelyeket a teljes alkalmazás használ, például a biztonsági és az adatátviteli funkciókat.
- Üzleti logikát sosem szabad kiszervezni az API-átjáróba.
- Ha nyomon kell követnie a tranzakciókat, érdemes korrelációs azonosítókat létrehozni a naplózáshoz.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes ezt a mintát használni?

Használja ezt a mintát, ha:

- Az alkalmazástelepítésben vannak közös feladatok, például az SSL-tanúsítványok vagy a titkosítás.
- Közös funkciója van több alkalmazástelepítésnek, amelyek erőforrásigényei eltérőek (például memória-erőforrások, tárterület vagy hálózati kapcsolatok).
- Bizonyos feladatokat egy speciális csapatra szeretne átruházni (például hálózati biztonság, szabályozás, hálózathatárokkal kapcsolatos kérdések).

Nem érdemes ezt a mintát használni, ha összekapcsol egyes szolgáltatásokat.

## <a name="example"></a>Példa

Ha az Nginxet használja SSL-kiszervezési berendezésként, a következő konfiguráció leállítja az összes beérkező SSL-kapcsolatot, és elosztja azt három felső rétegbeli HTTP-kiszolgáló között.

```console
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

## <a name="related-guidance"></a>Kapcsolódó útmutatók

- [Háttérrendszerek és előtérrendszerek minta](./backends-for-frontends.md)
- [Átjáróösszesítés minta](./gateway-aggregation.md)
- [Átjáró-útválasztási minta](./gateway-routing.md)
