---
title: Átjáró útválasztás minta
description: Útvonal kérelmek több szolgáltatások használatával egy végpontot.
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: 53239b23cfd98fad1edc38ca37c2274d5a9d7a0f
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="gateway-routing-pattern"></a>Átjáró útválasztás minta

Útvonal kérelmek több szolgáltatások használatával egy végpontot. Ebben a mintában akkor hasznos, ha egy végpontot, és az útvonalat a megfelelő szolgáltatásra, a kérés alapján több szolgáltatásokat nyújtó kíván.

## <a name="context-and-problem"></a>A környezetben, és probléma

Ha egy ügyfél több szolgáltatásait, egy külön végpont minden egyes szolgáltatás beállítását, valamint hogy az ügyfél-végpontok kezelése kihívást jelenthet. Például egy e-kereskedelmi alkalmazás előfordulhat, hogy nevünkben szolgáltatásokat például keresés, értékelést, bevásárlókocsiból, vegye ki és rendelési előzményeit. Minden szolgáltatás van egy másik API, az ügyfél együtt kell működnie, és az ügyfél kell tudnia végpontok a szolgáltatásokhoz való csatlakozás érdekében. Ha módosítja egy API-t, az ügyfél is frissíteni kell. Ha két vagy több különálló szolgáltatás refactor, egy szolgáltatás, a kód a szolgáltatás és az ügyfél is módosítania kell.

## <a name="solution"></a>Megoldás

Egy átjáró alkalmazások, szolgáltatások vagy a központi telepítések elé helyezze el. Alkalmazás réteg 7 útválasztás használatával továbbítja a kérelmet a megfelelő példányt.

Az ebben a mintában az ügyfélalkalmazás csak kell tudni kommunikálni egy végpontot. A szolgáltatás konszolidált vagy kiválasztott, az ügyfél nem feltétlenül igényel frissítése. Az átjáró, és csak a útválasztási kérelmet benyújtó továbbra is azt.

Átjáró emellett lehetővé teszi a háttér-szolgáltatást, az ügyfelek így ügyfélhívásokat egyszerű tarthatja engedélyezze a háttér-szolgáltatások, az átjáró mögötti változásainak absztrakt. Ügyfél hívások átirányítható bármilyen szolgáltatást vagy a szolgáltatások kell kezelni a várható ügyfél viselkedését, így hozzáadásához ossza fel, és az átjáró mögötti szolgáltatások biztosítására átrendezéséhez az ügyfél módosítása nélkül.

![](./_images/gateway-routing.png)
 
Ebben a mintában az üzembe helyezéssel is hozzájárulhat a azáltal, hogy hogyan frissítések már megkezdődött a felhasználók kezeléséhez. Ha a szolgáltatás egy új verziója van telepítve, azt a meglévő verziót párhuzamosan is telepíthető. Útválasztás segítségével szabályozhatja, hogy melyik verzió a szolgáltatás nem jelenik meg az ügyfelek, felkínálva a rugalmasságot különböző kiadás stratégiák használata növekményes, hogy párhuzamosan, vagy végezze el a frissítések végrehajtása. Probléma merül fel az új szolgáltatás telepítése után felderített gyorsan beállításai állíthatók vissza azáltal, hogy olyan konfigurációs módosítást az átjárón, az ügyfelek befolyásolása nélkül.

## <a name="issues-and-considerations"></a>Problémákat és szempontok

- Az átjárószolgáltatás vezethet be a hibaérzékeny pontok kialakulását. Győződjön meg arról, a rendelkezésre állási követelményeinek megfelelően tervezték. Érdemes lehet rugalmasság és a hibatűrés tolerancia képességek végrehajtása során.
- Az átjárószolgáltatás vezethet be a szűk keresztmetszetek. Győződjön meg arról, az átjáró rendelkezik a megfelelő teljesítmény terhelés kezelésére, és könnyedén méretezhető az növekedési igényeinek megfelelően.
- Hajtsa végre a terhelés tesztelése az átjárón nem vezetnek be kaszkádolt probléma van a szolgáltatások biztosításához.
- 7. szint átjáró útválasztási. Ez alapulhat a IP-, port, fejléc vagy URL-CÍMÉT.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes használni ezt a mintát

Ez mintát, mikor használja:

- Egy ügyfél hozzáfér egy átjáró mögötti több szolgáltatásait kell.
- Egyszerűbbé teheti az ügyfélalkalmazások egyetlen végpont használatával kívánja.
- Belső virtuális végpontok, például a fürt virtuális IP-címek egy virtuális gép portját az ilyen külsőleg megcímezhető végpontok érkező kéréseket továbbítani kell.

Ez a minta nem lehet megfelelő, ha csak egy vagy két szolgáltatást használó egyszerű alkalmazással.

## <a name="example"></a>Példa

Az útválasztó Nginx használ, a következő: egy egyszerű példa konfigurációs fájlt a kiszolgáló, amely az alkalmazások különböző virtuális könyvtárak különböző gépek hátsó végén levő kérelmek

```
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

## <a name="related-guidance"></a>Kapcsolódó útmutató

- [Háttérkiszolgálókon Frontends minta](./backends-for-frontends.md)
- [Átjáró összesítési minta](./gateway-aggregation.md)
- [Átjáró Offloading minta](./gateway-offloading.md)



