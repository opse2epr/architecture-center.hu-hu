---
title: Átjáró-útválasztási minta
titleSuffix: Cloud Design Patterns
description: Átirányíthatja a kéréseket több szolgáltatásra egyetlen végpont használatával.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: e5c93c98a562e790d547d08fdf312c973cfceed8
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54487228"
---
# <a name="gateway-routing-pattern"></a>Átjáró-útválasztási minta

Átirányíthatja a kéréseket több szolgáltatásra egyetlen végpont használatával. Ez a minta akkor lehet hasznos, ha több szolgáltatást kíván elérhetővé tenni egyetlen végponton, és a kérés alapján szeretné elvégezni az átirányítást a megfelelő szolgáltatáshoz.

## <a name="context-and-problem"></a>Kontextus és probléma

Ha az ügyfél több szolgáltatást is használ, az ügyfél számára nehéz feladat külön végpontot létrehozni minden szolgáltatás számára, és felügyelni azokat. Egy e-kereskedelmi alkalmazás például olyan szolgáltatásokat biztosíthat, mint a keresés, az értékelések, a kosár, a fizetés és a rendelési előzmények. Mindegyik szolgáltatáshoz külön API tartozik. Az ügyfél ezeken keresztül éri el a szolgáltatásokat, és ismernie kell minden egyes végpontot ahhoz, hogy csatlakozhasson a szolgáltatásokhoz. Ha egy API megváltozik, az ügyfelet is frissíteni kell. Ha újrabont egy szolgáltatást két vagy három önálló szolgáltatásra, a szolgáltatás és az ügyfél kódját is módosítani kell.

## <a name="solution"></a>Megoldás

Helyezzen egy átjárót az alkalmazások, szolgáltatások vagy telepítések csoportja elé. Az alkalmazás 7. rétegbeli útválasztását használva irányítsa át a kérést a megfelelő példányokhoz.

Ezzel a mintával az ügyfélalkalmazásnak csupán egyetlen végpontot kell ismernie, és azzal kell kommunikálnia. Szolgáltatások egyesítése vagy szétbontása után az ügyfelet nem feltétlenül kell frissíteni. Az továbbra is küldhet kéréseket az átjárónak, csak az útválasztás változik.

Az átjáró ezenkívül lehetővé teszi a háttérszolgáltatások elkülönítését az ügyfelektől, így az ügyfél hívásai egyszerűek maradnak, de az átjáró mögötti háttérszolgáltatások módosíthatók lesznek. Az ügyfél hívásai átirányíthatók a szolgáltatáshoz vagy szolgáltatásokhoz, amely(ek)nek kezelnie kell a várt ügyfélviselkedést. Ez lehetővé teszi szolgáltatások hozzáadását, felosztását és átszervezését az átjáró mögött az ügyfél módosítása nélkül.

![Az átjáró-útválasztási minta ábrája](./_images/gateway-routing.png)

Ez a minta segíthet a telepítés során is, mivel lehetővé teszi a frissítések bevezetésének kezelését. A szolgáltatás újabb verziójának telepítésekor az az aktuális verzióval egyidejűleg telepíthető. Útválasztás segítségével szabályozhatja, hogy a szolgáltatás melyik verzióját bemutatják az ügyfelektől, így rugalmasan kiadási stratégiát alkalmazhat fokozatos, egyidejű vagy frissítések kibocsátások befejezéséhez. Az új szolgáltatás telepítése után felfedezett hibák gyorsan visszavonhatók az átjáró konfigurációjának módosításával, ami nem befolyásolja az ügyfeleket.

## <a name="issues-and-considerations"></a>Problémák és megfontolandó szempontok

- Az átjárószolgáltatás kritikus meghibásodási pont lehet a rendszeren belül. Győződjön meg arról, hogy az átjáró konfigurációja megfelelően ki tudja szolgálni a rendelkezésre állási követelményeket. Az implementálás során tartsa szem előtt a rugalmassági és hibatűrési képességeket.
- Az átjárószolgáltatás szűk keresztmetszetté válhat. Győződjön meg arról, hogy az átjáró megfelelő teljesítménnyel bír a várható terhelés kezelésére, és skálázható a várható későbbi növekedésnek megfelelően.
- Végezzen terhelésteszteket az átjárón annak ellenőrzéséhez, hogy az nem indít-e el hibasorozatokat a szolgáltatásokban.
- Az átjáró útválasztása 7. szintű. Az útválasztás történhet IP-cím, port, fejléc vagy URL-cím alapján.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes ezt a mintát használni?

Használja ezt a mintát, ha:

- Az ügyfélnek több szolgáltatást kell használnia, amelyek egy átjáró mögött érhetőek el.
- Egyetlen végpont használatával kívánja egyszerűsíteni az ügyfélalkalmazásokat.
- A kérést kívülről címezhető végpontokról belső virtuális végpontokra kell átirányítani (például egy virtuális gép portjainak elérhetővé tétele virtuális IP-címek fürtözéséhez).

Nem érdemes ezt a mintát használni olyan egyszerű alkalmazásokhoz, amelyek csupán egy vagy két szolgáltatást használnak.

## <a name="example"></a>Példa

Az Nginxet útválasztóként használva létrehozhatja ezt az egyszerű konfigurációs fájt, amely a különböző virtuális könyvtárakban található alkalmazások kéréseit a háttérrendszer különböző gépeire irányíthatja át.

```console
server {
    listen 80;
    server_name domain.com;

    location /app1 {
        proxy_pass http://10.0.3.10:80;
    }

    location /app2 {
        proxy_pass http://10.0.3.20:80;
    }

    location /app3 {
        proxy_pass http://10.0.3.30:80;
    }
}
```

## <a name="related-guidance"></a>Kapcsolódó útmutatók

- [Háttérrendszerek és előtérrendszerek minta](./backends-for-frontends.md)
- [Átjáróösszesítés minta](./gateway-aggregation.md)
- [Átjárókiürítés minta](./gateway-offloading.md)
