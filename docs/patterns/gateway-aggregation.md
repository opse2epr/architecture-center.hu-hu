---
title: Átjáró összesítési minta
description: Segítségével egy átjáró több egyes kérelmeket az egy kérelemhez.
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: f59c8b8b02c6db28024d13621b782997e63a4e9e
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
ms.locfileid: "24541273"
---
# <a name="gateway-aggregation-pattern"></a>Átjáró összesítési minta

Segítségével egy átjáró több egyes kérelmeket az egy kérelemhez. Ebben a mintában akkor hasznos, ha egy ügyfél kell végrehajtania egy műveletet több különböző háttérrendszerek hívások.

## <a name="context-and-problem"></a>A környezetben, és probléma

Egy feladat végrehajtásához egy ügyfél lehet több hívások különböző háttér-szolgáltatásokra. Olyan alkalmazás, amely a feladat végrehajtása számos szolgáltatás alapul erőforrások minden kérelemnél meg kell kiadás. Bármely új funkció vagy szolgáltatás való hozzáadásakor az alkalmazást, szükség van-e a további kérelmeket, további növekvő erőforrás-követelmények és hálózati hívások. Az ügyfél és a háttérkiszolgáló közötti chattiness negatívan befolyásolhatja a teljesítményt és a skála az alkalmazás.  Mikroszolgáltatási architektúra végzett probléma gyakori, számos kisebb szolgáltatás természetes épül alkalmazások a kereszt-hívások nagyobb mennyiségű van. 

Az alábbi ábra az ügyfél kérelmeket küld arra az egyes szolgáltatások (1,2,3). Minden szolgáltatás feldolgozza a kérést, és visszaküldi a választ az alkalmazás (4,5,6). Mobilhálózati egy és általában nagy késésű az egyes kérelmek használatával az ilyen módon nem hatékony, és megszakadt kapcsolat vagy hiányos kérelmeket eredményezhet. Minden egyes kérelem végezhető párhuzamos, amíg az alkalmazás kell küldeni, várjon, amíg és feldolgozni az adatokat az egyes kérelmek összes külön kapcsolatok, növekvő valószínűsége. a hiba a.

![](./_images/gateway-aggregation-problem.png) 

## <a name="solution"></a>Megoldás

Átjáró használatával az ügyfél és a szolgáltatások közötti chattiness csökkenthető. Az átjáró ügyfél kéréseket fogad, a különböző rendszerek háttér kérelmek kiszállítja aggregálja az eredményeket és elküldi azokat a kérelmező ügyfélnek.

Ebben a mintában csökkenti az alkalmazás által az háttér-services-kérelmek számát, és az alkalmazások teljesítményének javítása a nagy késleltetésű hálózaton keresztül.

A következő ábrán az az alkalmazás egy kérést küld az átjáró (1). A kérelem tartalmazza a további kérelmeket tartalmazó csomagot. Az átjáró bontja a ezeket, és feldolgozza a minden kérelmet elküldheti a megfelelő szolgáltatást (2). Minden szolgáltatás visszaküld egy választ az átjáró (3). Az átjáró egyesíti az egyes szolgáltatások válaszát, és visszaküldi a választ az alkalmazáshoz (4). Az alkalmazás egy egyetlen kérést, és csak egyetlen választ kap az átjárót.

![](./_images/gateway-aggregation.png)

## <a name="issues-and-considerations"></a>Problémákat és szempontok

- Az átjáró nem kell vezetnie a szolgáltatás kapcsolódási háttér szolgáltatásban.
- Az átjáró közelében a háttér-szolgáltatások késés lehető legnagyobb mértékben csökkenteni kell elhelyezni.
- Az átjárószolgáltatás vezethet be a hibaérzékeny pontok kialakulását. Gondoskodjon arról, hogy az átjáró megfelelően célja, hogy az alkalmazás rendelkezésre állási követelményeinek.
- Az átjáró a szűk keresztmetszetek vezethetnek. Győződjön meg arról, az átjáró rendelkezik a megfelelő teljesítmény terhelés kezelésére, és megfelelnek a várható növekedési is méretezhető.
- Hajtsa végre a terhelés tesztelése az átjárón nem vezetnek be kaszkádolt probléma van a szolgáltatások biztosításához.
- Egy rugalmas, például a módszereket kialakításának megvalósítása [válaszfalak][bulkhead], [áramkör legfrissebb][circuit-breaker], [újra] [ retry], és időtúllépéseket okoz.
- Ha egy vagy több hívások túl sokáig tart, előfordulhat, hogy elfogadható időtúllépési és adatok részhalmaza vissza. Vegye figyelembe, hogy az alkalmazás ebben a forgatókönyvben fogja kezelni.
- Aszinkron I/O segítségével győződjön meg arról, hogy a háttérkiszolgálón a késés nem teljesítményproblémákat okozhatnak az alkalmazásban.
- Korrelációs azonosító segítségével nyomon követheti a minden egyes hívás elosztott nyomkövetés megvalósításához.
- A figyelő kérelem metrikák és a válasz méretét.
- Fontolja meg a gyorsítótárazott adatokat ad vissza, a hibák kezelésének feladatátvételi stratégiát.
- Helyett a átjáróra kerülő összesítés létrehozása, vegye figyelembe, ha kialakít egy összesítési szolgáltatás az átjáró mögötti. A kérelem összesítési valószínűleg lesz a különböző erőforrás követelményei, mint más szolgáltatások az átjáró, és hatással lehet az átjáró az Útválasztás és kiszervezésével funkciót.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes használni ezt a mintát

Ez mintát, mikor használja:

- Egy ügyfélnek kell végrehajtania egy műveletet több háttérbeli szolgáltatásokkal kommunikálni.
- Az ügyfél használhatja hálózatok jelentős késés, például cellás hálózatokon.

Ez a minta nem alkalmasak lehetnek esetén:

- Több műveletek között az ügyfél és az egyetlen szolgáltatás közötti hívások számát csökkenteni szeretné. A forgatókönyv, a jobb megoldás lehet egy kötegelt művelet hozzáadása a szolgáltatás.
- Az ügyfél vagy alkalmazás található a háttér-szolgáltatások és a késés nem lényeges szempontja.

## <a name="example"></a>Példa

A következő példa bemutatja, hogyan hozhat létre egy egyszerű átjáró összesítési NGINX szolgáltatás Lua.

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

## <a name="related-guidance"></a>Kapcsolódó útmutató

- [Háttérkiszolgálókon Frontends minta](./backends-for-frontends.md)
- [Átjáró Offloading minta](./gateway-offloading.md)
- [Átjáró útválasztás minta](./gateway-routing.md)

[bulkhead]: ./bulkhead.md
[circuit-breaker]: ./circuit-breaker.md
[retry]: ./retry.md