---
title: Átjáróösszesítési minta
titleSuffix: Cloud Design Patterns
description: Több egyéni kérést összesíthet egyetlen kérésbe egy átjáró segítségével.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
ms.custom: seodec18
ms.openlocfilehash: 8d929b1b3937d8f9ef50c1b08e8aea0b5c1f92c1
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/04/2019
ms.locfileid: "54009457"
---
# <a name="gateway-aggregation-pattern"></a>Átjáróösszesítési minta

Több egyéni kérést összesíthet egyetlen kérésbe egy átjáró segítségével. Ez a minta akkor lehet hasznos, ha az ügyfélnek különböző háttérrendszereket kell több alkalommal hívnia egy művelet végrehajtásához.

## <a name="context-and-problem"></a>Kontextus és probléma

Előfordulhat, hogy az ügyfélnek különböző háttérszolgáltatásokat kell többször hívnia egy feladat végrehajtásához. Ha egy alkalmazás számos szolgáltatásra támaszkodik egy feladat végrehajtásakor, minden kéréshez erőforrásokat kell felhasználnia. Ha bármilyen új funkciót vagy szolgáltatást adnak hozzá az alkalmazáshoz, további kérések válnak szükségessé, ami még több erőforrást és hálózati hívást igényel. Az ügyfél és a háttérrendszerek közötti forgalom csökkentheti az alkalmazás teljesítményét és skálázhatóságát.  A mikroszolgáltatás-architektúrák még gyakoribbá tették ezt a problémát, mivel a számos kisebb szolgáltatásra épülő alkalmazásokban több szolgáltatások közötti hívás fordul elő.

A következő ábrán az ügyfél minden egyes szolgáltatásnak (1,2,3) kéréseket küld. A szolgáltatások feldolgozzák a kéréseket, és válaszolnak az alkalmazásnak (4,5,6). A jellemzően nagy hálózati késésű mobilhálózatokon az egyedi kérések ilyen módon történő használata nem hatékony, és ilyenkor előfordulhat, hogy megszakad a kapcsolat, vagy a kérések csak részlegesen teljesülnek. Bár az egyes kérések feldolgozása párhuzamosan történik, az alkalmazásnak minden kérés adatait el kell küldenie, megvárni a választ, és feldolgoznia az egyes kérések adatait. Mindez önálló kapcsolatokkal történik, ami növeli a hiba esélyét.

![Az átjáró Átjáróösszesítés minta probléma diagramja](./_images/gateway-aggregation-problem.png)

## <a name="solution"></a>Megoldás

Egy átjáró használatával csökkentheti az ügyfél és a szolgáltatások közötti forgalmat. Az átjáró fogadja az ügyfél kéréseit, továbbítja a kéréseket a különböző háttérrendszerekhez, majd összesíti az eredményeket, és visszaküldi őket a kérést küldő ügyfélnek.

Ez a minta csökkentheti az alkalmazás által a háttérszolgáltatásoknak küldött kérések számát, ami javítja az alkalmazás teljesítményét a nagy késésű hálózatokban.

A következő ábrán az alkalmazás egy kérést küld az átjárónak (1). A kérésben egy további kéréseket tartalmazó csomag található. Az átjáró kibontja a csomagot, és feldolgozza az egyes kéréseket úgy, hogy a megfelelő szolgáltatáshoz irányítja őket (2). Minden egyes szolgáltatás elküldi a választ az átjárónak (3). Az átjáró egyesíti az egyes szolgáltatásoktól érkező válaszokat, és azokat egyetlen válaszként továbbítja az alkalmazásnak (4). Az alkalmazás egyetlen kérést küld, és egyetlen választ kap az átjárótól.

![Az átjáró Átjáróösszesítés minta megoldás diagramja](./_images/gateway-aggregation.png)

## <a name="issues-and-considerations"></a>Problémák és megfontolandó szempontok

- Érdemes ügyelni rá, hogy az átjáró ne kapcsoljon össze háttérszolgáltatásokat.
- Az átjárót a háttérszolgáltatások közelében kell elhelyezni, hogy a lehető legkisebb késés jelentkezzen.
- Az átjárószolgáltatás kritikus meghibásodási pont lehet a rendszeren belül. Győződjön meg arról, hogy az átjáró megfelelően ki tudja szolgálni az alkalmazás rendelkezésre állási követelményeit.
- Az átjáró szűk keresztmetszetté válhat. Ellenőrizze, hogy az átjáró teljesítménye megfelelő-e a várható terhelés kezeléséhez, és skálázható-e az esetleges későbbi növekedésnek megfelelően.
- Végezzen terhelésteszteket az átjárón annak ellenőrzéséhez, hogy az nem indít-e el hibasorozatokat a szolgáltatásokban.
- Implementáljon egy rugalmas rendszert [válaszfalak][bulkhead], [áramkör-megszakítás][circuit-breaker], [újrapróbálkozások][retry], időtúllépés és hasonló technikák használatával.
- Ha egy vagy több szolgáltatás hívása túl sok időt vesz igénybe, akkor bizonyos esetekben elfogadható lehet, ha időtúllépés miatt megszakad a kapcsolat, és a válasz részleges adatokat tartalmaz. Gondolja át, hogyan fogja kezelni az alkalmazása ezt a forgatókönyvet.
- Alkalmazzon aszinkron I/O-t, hogy a háttérrendszer késése ne legyen hatással az alkalmazás teljesítményére.
- Implementáljon elosztott nyomkövetést korrelációs azonosítók használatával az egyes hívások nyomon követéséhez.
- Monitorozza a kérésmetrikákat és a válaszok méretét.
- Fontolja meg gyorsítótárazott adatok visszaadását a feladatátvételi stratégia részeként a hibák kezelésére.
- Megteheti, hogy nem az átjáróba építi be az összesítést, hanem egy összesítési szolgáltatást helyez el az átjáró mögött. A kérések összesítésének valószínűleg eltérő erőforrásigénye van, mint az átjáró többi szolgáltatásának, ami befolyásolhatja az átjáró útválasztási és kiszervezési funkcióit.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes ezt a mintát használni?

Használja ezt a mintát, ha:

- Az ügyfélnek több háttérszolgáltatással kell kommunikálnia az adott művelet végrehajtásához.
- Előfordulhat, hogy az ügyfél csak nagy késésű hálózatokhoz, például mobilhálózatokhoz fér hozzá.

Nem érdemes ezt a mintát használni, ha:

- Csökkenteni szeretné az ügyfél és egy szolgáltatás közötti hívások számát több műveletben. Ebben az esetben fontolja meg egy kötegelt művelet hozzáadását a szolgáltatáshoz.
- Az ügyfél vagy az alkalmazás a háttérszolgáltatások közelében található, és a késés elhanyagolható.

## <a name="example"></a>Példa

A következő példa azt mutatja be, hogyan hozhat létre egyszerű átjáró-összesítési NGINX szolgáltatást a Lua használatával.

```lua
worker_processes  4;

events {
  worker_connections 1024;
}

http {
  server {
    listen 80;

    location = /batch {
      content_by_lua '
        ngx.req.read_body()

        -- read json body content
        local cjson = require "cjson"
        local batch = cjson.decode(ngx.req.get_body_data())["batch"]

        -- create capture_multi table
        local requests = {}
        for i, item in ipairs(batch) do
          table.insert(requests, {item.relative_url, { method = ngx.HTTP_GET}})
        end

        -- execute batch requests in parallel
        local results = {}
        local resps = { ngx.location.capture_multi(requests) }
        for i, res in ipairs(resps) do
          table.insert(results, {status = res.status, body = cjson.decode(res.body), header = res.header})
        end

        ngx.say(cjson.encode({results = results}))
      ';
    }

    location = /service1 {
      default_type application/json;
      echo '{"attr1":"val1"}';
    }

    location = /service2 {
      default_type application/json;
      echo '{"attr2":"val2"}';
    }
  }
}
```

## <a name="related-guidance"></a>Kapcsolódó útmutatók

- [Háttérrendszerek és előtérrendszerek minta](./backends-for-frontends.md)
- [Átjárókiürítés minta](./gateway-offloading.md)
- [Átjáró-útválasztási minta](./gateway-routing.md)

[bulkhead]: ./bulkhead.md
[circuit-breaker]: ./circuit-breaker.md
[retry]: ./retry.md