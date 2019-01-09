---
title: Szolgáltatások közötti kommunikáció a mikroszolgáltatások
description: Szolgáltatások közötti kommunikáció a mikroszolgáltatások
author: MikeWasson
ms.date: 10/23/2018
ms.openlocfilehash: 4760cd54c494fb8fded4b396ac772d2c9c82cafa
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/08/2019
ms.locfileid: "54113499"
---
# <a name="designing-microservices-interservice-communication"></a>Mikroszolgáltatások tervezése: Szolgáltatások közötti kommunikáció

Mikroszolgáltatások közötti kommunikáció hatékony és robusztus kell lennie. Sok kis szolgáltatások egyetlen tranzakció végrehajtásához használja ez kihívást jelenthet. Ebben a fejezetben áttekintjük az aszinkron üzenetkezelés és szinkron API-k érvényesíthető kompromisszumokat. Ezután hogy tekintsen meg néhányat a rugalmas közötti kommunikációt eredményezhet, és a szerepkör, amely a szolgáltatás rácsvonal lejátszható kialakításának is kihívást jelent.

![Szolgáltatások közötti kommunikációt eredményezhet ábrája](./images/interservice-communication.png)

## <a name="challenges"></a>Problémák

Íme néhány fő kihívása merül fel a szolgáltatások közötti kommunikációt. Szolgáltatás rácsvonalak, ebben a fejezetben ismertetett számos, ezek a kihívások kezelésére tervezték.

**Rugalmasság**. Lehetséges, hogy több tucat vagy akár több száz bármely adott mikroszolgáltatás-példányok. Egy példány nem sikerül, számtalan. Csomópont-szintű hiba, például egy hardverhiba vagy a virtuális gép újraindítását is lehet. Egy példány lehet, hogy az összeomlási, vagy kihasznált, a kérelmek és az új kérések feldolgozása nem sikerült. Ezek az események bármelyikét okozhatja hálózati hívása sikertelen. Nincsenek elősegítheti a hálózati szolgáltatások közötti hívások rugalmasabb két kialakítási minta:

- **[Ismételje meg](../patterns/retry.md)**. Egy hálózati hívások meghiúsulhatnak, ami újbóli próbálkozással megszűnik önmagában átmeneti hiba miatt. Helyett tudnának sikertelen, az a hívó kell próbálja megismételni a műveletet egy bizonyos szám, ahányszor, vagy amíg a beállított időkorlát lejárta általában. Azonban ha egy művelet nem idempotens, az újrapróbálkozások okozhat, nem kívánt mellékhatásokkal. Az eredeti hívás sikeres lehet, de a hívó soha nem kap választ. A hívó újrapróbálkozik, ha a művelet kétszer is érvényesíthetők. Általában nem biztonságos az újrapróbálkozás POST és PATCH metódusokat, mert ezek nem garantált, hogy idempotensek legyenek.

- **[Áramkör-megszakító](../patterns/circuit-breaker.md)**. Túl sok sikertelen kérelem okozhat a szűk keresztmetszet, mert függőben lévő kérelmek halmozódnak az üzenetsorban. Ezek a blokkolt kérelmek olyan kritikus rendszererőforrásokat akadályozhatnak, mint a memória, szálak, adatbázis-kapcsolatok stb., ezáltal pedig kaszkádolt meghibásodásokhoz vezethetnek. Az áramkör-megszakítási minta megakadályozhatja, hogy a szolgáltatás próbáljon többször végrehajtani egy műveletet, amely nagy eséllyel lesz sikertelen.

**Terheléselosztás**. Service "A" "B" szolgáltatást hív meg, ha a kérelem egy futó példány a "B" szolgáltatás kell elérnie. A Kubernetes a `Service` erőforrástípus egy stabil IP-címet biztosít a podok csoportja. A szolgáltatás IP-címet a hálózati forgalom iptable szabályok segítségével lekérdezi továbbítani podot. Alapértelmezés szerint egy véletlenszerű pod van kiválasztva. A szolgáltatás rácsvonal (lásd alább) biztosíthat intelligens terheléselosztási algoritmusok megfigyelt várakozási ideje vagy egyéb mérőszámok alapján.

**Elosztott nyomkövetési**. Egy tranzakció több szolgáltatást magában foglalhat. Amely megnehezítheti annak visszakövetését általános teljesítményt és a rendszer állapotának figyeléséhez. Akkor is, ha minden szolgáltatás állít elő, naplókat és mérőszámokat, anélkül, hogy bármilyen módon összekapcsolása őket, csak korlátozottan használhatók legyenek. A fejezet [naplózást és figyelést](./logging-monitoring.md) előadások többet is megtudhat az elosztott nyomkövetést, de azt díjszabásunkban itt, kihívást.

**Service versioning**. Ha egy csapat helyez üzembe egy szolgáltatás egy új verzióját, el kell kerülni, használhatatlanná tévő szolgáltatások vagy külső a tőle függő ügyfeleket. Emellett érdemes egy szolgáltatást egymás mellett, és átirányíthatja a kéréseket több verziójának futtatására egy meghatározott verzióra. Lásd: [API-k verziókezelése](./api-design.md#api-versioning) további beszélgetéshez, a problémát.

**A TLS-titkosítás és a kölcsönös TLS hitelesítés**. Biztonsági okokból érdemes a TLS-szolgáltatások közötti forgalom titkosításához, és használja a TLS kölcsönös hitelesítés hívóit hitelesítéséhez.

## <a name="synchronous-versus-asynchronous-messaging"></a>A szinkron és aszinkron üzenetkezelés

Nincsenek két alapvető üzenetkezelési mintát, amely mikroszolgáltatások más mikroszolgáltatások folytatott kommunikációhoz használható.

1. Szinkron kommunikációt. Ebben a mintában egy szolgáltatás meghív egy API, amely egy másik szolgáltatás, például a HTTP- vagy gRPC protokoll használatát. Ez a beállítás egy szinkron üzenetkezelési minta azért a hívó megvárja a címzett válaszát.

2. Aszinkron üzenettovábbítási. Ebben a mintában egy szolgáltatás üzenetet küld a válaszra való várakozás nélkül, és a egy vagy több szolgáltatás aszinkron módon feldolgozni az üzenetet.

Fontos, aszinkron i/o- és a egy aszinkron protokoll megkülönböztetésére. Aszinkron I/O azt jelenti, hogy a hívó szálat az I/O végrehajtása során nincs letiltva. Amely fontos a teljesítmény, de egy implementálási részlete az architektúra szempontjából. Egy aszinkron protokoll azt jelenti, hogy a küldő nem várniuk a válaszra. HTTP egy szinkron protokoll, annak ellenére, hogy egy HTTP-alapú aszinkron i/o használhatja a kérelem elküldésekor.

Mindegyik minta hátrányai is vannak. Kérés/válasz nem egy jól érthető paradigm, ezért a API-k tervezése úgy természetesebb, mint a üzenetkezelési rendszerek tervezése során. Aszinkron üzenetkezelés azonban van néhány előnyeit, amelyek a mikroszolgáltatási architektúrákban nagyon hasznosak lehetnek:

- **Kapcsolási csökkentett**. Az üzenet küldője nem kell tudnia a fogyasztó.

- **Több előfizető**. A közzétételi és előfizetési modell használatával, több fogyasztó előfizethetnek események fogadására. Lásd: [eseményvezérelt architektúra stílusának](/azure/architecture/guide/architecture-styles/event-driven).

- **Hiba elkülönítési**. Ha a feldolgozó meghibásodik, a küldő továbbra is küldhet üzeneteket. Az üzenetek fog felvenni, amikor helyreállítja a felhasználói. Ez a lehetőség különösen hasznos a mikroszolgáltatási architektúrákban, mert minden szolgáltatásnak van saját életciklusát. Egy szolgáltatás elérhetetlenné válnak, vagy egy adott időpontban egy újabb verzióra kell cserélni. Aszinkron üzenetkezelés időszakos állásidő képes kezelni. Szinkron API-k, másrészt megkövetelése az alárendelt szolgáltatás elérhető legyen, vagy a művelet sikertelen lesz.

- **Válaszképességét**. Egy felsőbb szintű szolgáltatás gyorsabban választ is, ha várja meg az alárendelt szolgáltatásokkal. Ez különösen hasznos a mikroszolgáltatási architektúrákban. Ha (szolgáltatás egy hívás B, amely meghívja a C, és így tovább) szolgáltatásfüggőségek láncolata, a szinkron hívások vár adhat hozzá késés elfogadhatatlan mennyiségű.

- **Terheléskiegyenlítés**. Egy üzenetsor képes pufferként a terhelés kiegyenlítése érdekében, hogy a fogadók a saját ütemben üzenetek feldolgozásához.

- **A munkafolyamatok**. Üzenetsorok ellenőrzőpontos az üzenetet a munkafolyamat minden lépése után egy munkafolyamatot kezeléséhez használható.

Vannak azonban is áttekinthet néhány problémát, gyakorlatilag az aszinkron üzenetkezelés használatával.

- **Az üzenetkezelési infrastruktúra a csatolási**. Egy adott üzenetkezelési infrastruktúrát használó szoros összekapcsolódást okozhat az infrastruktúrát. Később átválthat egy másik üzenetküldési infrastruktúra nehéz lesz.

- **Késés**. Egy művelet végpontok közötti késés magas, ha az üzenet-várólistákból megtelnek válhat.

- **Költség**. Nagy sebességű jelentős hatással lehet a az üzenetkezelési infrastruktúra költségét.

- **Összetettség**. Aszinkron üzenetkezelés kezelése nem egy jelentéktelen feladat. Ismétlődő üzenetek, például megszüntetéséhez másolásával vagy azáltal, hogy a művelet idempotens kell kezelnie. Emellett akkor is nehezen megvalósíthatók kérés-válasz szemantika aszinkron üzenetküldés segítségével. Visszajelzés küldése szüksége van egy másik üzenetsornak, és vesse össze a kérés-és válaszüzenetek lehetővé.

- **Átviteli sebesség**. Ha üzeneteket kell *szemantika várólistára*, az üzenetsor szűk keresztmetszetté válhat a rendszerben. A szükséges minden üzenetet legalább egy várólista működését és a egy sorból. Továbbá várólista szemantika általában valamilyen zárolás belül az üzenetkezelési infrastruktúra szükséges. Ha egy felügyelt szolgáltatás, az üzenetsor előfordulhat késést, mert a várólista külső a fürt virtuális hálózathoz. A kötegelés üzenetek csökkentheti ezeket a problémákat, de, amely bonyolultabbá teszi a kód. Ha az üzenetek üzenetsorba szemantika nem igényelnek, fogja tudni használni az esemény *stream* helyett egy üzenetsorba. További információkért lásd: [architekturális stílus eseményvezérelt](../guide/architecture-styles/event-driven.md).

## <a name="drone-delivery-choosing-the-messaging-patterns"></a>Drone Delivery: Az üzenetkezelési minták kiválasztása

Ezeket a szempontokat szem előtt a fejlesztői csapat hozni a következő tervezési szempontokat a Drone Delivery alkalmazás

- A szolgáltatás nyilvános REST API-t használó ügyfélalkalmazások ütemezése, frissítenie vagy megszakítja a szállítások tesz elérhetővé.

- A szolgáltatás az Event Hubs aszinkron üzeneteket küldeni a Scheduler szolgáltatás használ. Aszinkron üzenetekkel is végre kell hajtani a terhelés-kiegyenlítés szükséges támogatunk. A betöltési és a Scheduler szolgáltatás együttműködését a részletekért lásd: [adatfeldolgozás és munkafolyamatok][ingestion-workflow].

- Minden fiók, a kézbesítési, a csomag, a Drónos és a külső átviteli szolgáltatások belső REST API-k elérhetővé tehet. A Scheduler szolgáltatás ezen API-k számára, egy felhasználói kérelem hív meg. Egy szinkron API-k oka, hogy az ütemező kell az alsóbb rétegbeli minden kapott választ. Hiba történt az alsóbb rétegbeli szolgáltatások azt jelenti, hogy a teljes művelet sikertelen volt. Azonban egy potenciális problémát, a késés, a háttérszolgáltatások meghívásával bevezetett mennyiségét.

- Ha bármely alárendelt szolgáltatás nem átmeneti hiba, a teljes tranzakció kell megjelölve lennie nem sikerült. Ebben az esetben kezelése érdekében a Scheduler szolgáltatás aszinkron üzenetet küld a felügyelő, hogy a felügyelő ütemezheti a kompenzáló tranzakciók, a fejezetben leírtak szerint [adatfeldolgozás és munkafolyamatok] [ ingestion-workflow].

- A kézbesítési szolgáltatás elérhetővé teszi a nyilvános API-t, amellyel az ügyfelek egy kézbesítési állapotának lekéréséhez. A fejezet [API-átjáró](./gateway.md), bemutatjuk, hogyan API-átjáró elrejtheti az ügyféltől az alapul szolgáló szolgáltatások, így az ügyfél nem kell tudnia a mely szolgáltatások mely API-k elérhetővé.

- Bár egy drónt útban, a Drone szolgáltatás küld eseményeket, amelyek tartalmazzák a drone aktuális helyét és állapotát. A kézbesítési szolgáltatás egy kézbesítési állapotának nyomon követése érdekében ezeket az eseményeket figyeli.

- Ha a szállítási állapota, a kézbesítési szolgáltatás például küld-e egy kézbesítési állapot esemény `DeliveryCreated` vagy `DeliveryCompleted`. Bármely szolgáltatás előfizethetnek ezeket az eseményeket. A jelenlegi kialakítás a kézbesítési szolgáltatás nem az egyetlen előfizető, de lehet más előfizetőkkel később. Például egy valós idejű elemzési szolgáltatás az események előfordulhat, hogy lépjen. És az ütemező nem kell várniuk a válaszra, mert az több előfizetők hozzáadása nem befolyásolja a munkafolyamat fő elérési útja.

![Drónos kommunikációs ábrája](./images/drone-communication.png)

Figyelje meg, hogy a kézbesítési állapotesemények drón helyét események származnak. Például ha csökken a csomag egy drónt eléri egy kézbesítési hely, a kézbesítési szolgáltatás fordítja le ez egy DeliveryCompleted esemény. Itt látható egy példa vélekedést tartománymodellek tekintetében. A fentebb leírt módon Drónos felügyeleti körülhatárolt kontextus egy külön tartozik. A drone eseményeket továbbítja egy drónt fizikai helyét. A kézbesítési esemény, másrészt egy kézbesítési, amely egy másik üzleti entitás állapotának változása képviseli.

## <a name="using-a-service-mesh"></a>A szolgáltatás rácsvonal használatával

A *háló szolgáltatás* szoftver rétege, amely kezeli a szolgáltatások közötti kommunikációt. Szolgáltatás rácsvonalak úgy tervezték, számos, az az előző szakaszban felsorolt problémák megoldása érdekében, és ezek a problémák távolabbi magukat a mikroszolgáltatás-alapú és a egy megosztott rétegre felelősséget áthelyezni. A szolgáltatás háló proxyként működik, amely elfogja a mikroszolgáltatások a fürt közötti hálózati kommunikáció.

> [!NOTE]
> Szolgáltatás háló egyik példája a [Nagykövet minta](../patterns/ambassador.md) &mdash; segítő szolgáltatása, amely az alkalmazás nevében hálózati kéréseket küld.

Most, a fő lehetőség áll rendelkezésre a Kubernetes szolgáltatás rácsvonal [linkerd](https://linkerd.io/) és [Istio](https://istio.io/). Mindkét technológiát gyorsan fejlődő. Azonban néhány linkerd és Istio is rendelkező közös funkciók:

- Terheléselosztás a munkamenet szintjén, a megfigyelt késéseket vagy a szálankénti függőben lévő kérelmek száma alapján. Ezzel javítható a teljesítmény a 4. réteg terheléselosztását Kubernetes által biztosított keresztül.

- 7. rétegbeli útválasztási URL-cím, állomásfejlécet, API-verzió vagy egyéb alkalmazásszintű szabályok alapján.

- Próbálkozzon újra a sikertelen kérelmek. A szolgáltatás rácsvonal tisztában van azzal a HTTP-hibakódok, és automatikusan újra a sikertelen kérelmek. Beállíthatja a maximális újrapróbálkozások számát, és a egy bizonyos időkorláton annak érdekében, hogy a maximális várakozási kötve.

- Áramkör-megszakítás. Ha egy példány folyamatosan sikertelen kérelmek, a szolgáltatás háló fogja ideiglenesen megjelölése nem érhető el. A leállási idő elteltével újra megpróbálja a a példányt. Beállíthatja, hogy az áramkör-megszakító különféle feltételek, például az egymást követő hibák száma alapján  

- Szolgáltatás háló eredményezhet hívássorozatot indított, például a kérés kötet, a késés, a hiba és a sikeres és a válaszok mérete metrikákat rögzíti. A szolgáltatás háló is lehetővé teszi az elosztott nyomkövetést korrelációs adatokat az egyes ugrások hozzáadásával a kérelemben.

- Szolgáltatások közötti hívások kölcsönös TLS hitelesítés.

Szükség van egy szolgáltatás háló? Adnak hozzá egy elosztott rendszer értéke természetesen meggyőző. Ha egy szolgáltatás háló nem rendelkezik, vegye figyelembe, egyes fejezet elején említett kihívást kell. Megoldható problémák, mint például az újrapróbálkozási, áramkör-megszakító és a egy szolgáltatás háló nélkül elosztott nyomkövetést, de a szolgáltatás rácsvonal helyezi át ezen kívül az egyes szolgáltatásokat és a egy dedikált rétegre vonatkozik. Másrészről szolgáltatás rácsvonalak olyan viszonylag új technológia, amely továbbra is a lejáró. A szolgáltatás-háló üzembe helyezése bonyolultabbá teszi a telepítés és a fürt konfigurációját. Előfordulhatnak teljesítményre gyakorolt hatása, mivel a kérelmek most már van irányítva a szolgáltatás háló proxyn keresztül, és mivel a felesleges szolgáltatások futnak a fürt minden csomópontján. Érdemes tegye alapos teljesítmény és a terheléses tesztelés éles környezetben a szolgáltatás-háló üzembe helyezése előtt.

> [!div class="nextstepaction"]
> [API-tervezés](./api-design.md)

<!-- links -->

[ingestion-workflow]: ./ingestion-workflow.md
