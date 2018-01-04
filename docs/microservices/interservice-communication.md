---
title: "Mikroszolgáltatások értekezleteire kommunikáció"
description: "Mikroszolgáltatások értekezleteire kommunikáció"
author: MikeWasson
ms.date: 12/08/2017
ms.openlocfilehash: aff2fb7b2be25ca32d6224cee15363880cfb1488
ms.sourcegitcommit: a8453c4bc7c870fa1a12bb3c02e3b310db87530c
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/29/2017
---
# <a name="designing-microservices-interservice-communication"></a>Mikroszolgáltatások tervezése: Interservice kommunikáció

Mikroszolgáltatások közötti kommunikáció hatékony és nagy teljesítményű kell lennie. A sok kisméretű szolgáltatások közötti kommunikáció során egyetlen tranzakció végrehajtásához kihívást is lehet. Ebben a fejezetben úgy tekintünk között szinkron API-k és aszinkron üzenetkezelési mellékhatásokkal is. Ezután úgy tekintünk, néhány a kihívásokat tervezésének rugalmas értekezleteire kommunikációra és a szerepkör, amely a szolgáltatás rácsvonal játszhatja le.

![](./images/interservice-communication.png)

## <a name="challenges"></a>Kihívásai 

Az alábbiakban néhány fő kihívása szolgáltatások közötti kommunikációs eredő. Szolgáltatás rácsvonalak, ez a fejezet ismerteti úgy tervezték, hogy ezek a kihívások számos kezelni.

**Rugalmasság.** Lehetséges, hogy több tucatnyi vagy akár több száz bármely adott mikroszolgáltatási példányai. Egy példányát minden számos oka lehet sikertelen. Csomópont-szintű hiba, például hardverhiba vagy egy virtuális gép újraindítása lehet. Egy példány lehet, hogy összeomlás, vagy egyszerre az által, és nem lehet új kérelmeket feldolgozni. Ezek az események bármelyike okozhatja hálózati hívás sikertelen lesz. Két tervezési mintáról olvashat, amelyek rugalmasabb teheti a hálózati szolgáltatások közötti hívások van:

- **[Próbálja meg újra](../patterns/retry.md)**. Hálózati hívása sikertelen lehet, hogy megbizonyosodjon önmagában átmeneti hiba miatt. Helyett nem végleges, az a hívó érdemes újra megpróbálnia a művelet bizonyos számú alkalommal, vagy amíg a konfigurált időkorlát lejárta általában. Azonban ha egy művelet nem idempotent, újrapróbálkozások okozhat, nem kívánt hatásai. Az eredeti hívás sikeres lehet, de a hívó soha nem kap választ. Ha a hívó újrapróbálkozik, a művelet lehet, hogy hívható meg kétszer. Általában nincs biztonságos, majd ismételje meg a POST vagy javítás módszerek, mert ezek nem garantált, hogy az idempotent lehet.

- **[Áramköri megszakító](../patterns/circuit-breaker.md)**. Túl sok sikertelen kérelem a szűk keresztmetszetek, függőben lévő kérelmek gyűlik össze a várólista okozhat. Ezek a blokkolt kérelmek olyan kritikus rendszererőforrásokat akadályozhatnak, mint a memória, szálak, adatbázis-kapcsolatok stb., ezáltal pedig kaszkádolt meghibásodásokhoz vezethetnek. Az áramköri megszakító mintát megakadályozhatja, hogy a szolgáltatás ismételten közben, valószínűleg sikertelen művelet. 

**Terheléselosztás**. Ha a szolgáltatás "A" hívja service "B", a kérelem egy futó példány szolgáltatás "B" kell elérnie. A Kubernetes a `Service` erőforrástípust biztosít egy stabil IP-címet három munkaállomás-csoporttal csoportja. A szolgáltatás IP-címre a hálózati forgalom iptable szabályok segítségével lekérdezi egy pod továbbítja. Alapértelmezés szerint egy véletlenszerű pod van kiválasztva. (Lásd alább) szolgáltatás rácsvonal biztosíthat intelligensebb terheléselosztási algoritmusok megfigyelt késést és más metrikák alapján.

**Nyomkövetés elosztott**. Egy tranzakció lehet, hogy span több szolgáltatásra. Amely teheti rögzített általános teljesítménye és a rendszer állapotának figyelésére. Akkor is, ha minden szolgáltatások készítsenek-naplók és a metrikák nélkül alkalmazássá őket, hogy bármilyen módon korlátozott használat vannak. A fejezet [naplózás és figyelés](./logging-monitoring.md) megbeszélések arról elosztott nyomkövetés, de azt említse meg itt a kérdés.

**Service versioning**. Egy csoport szolgáltatás új verzióját telepíti, ha el kell kerülni, szolgáltatások vagy külső ügyfelek esetében, függnek tőle. Emellett érdemes egy adott verzióra különböző verzióinak a szolgáltatás-párhuzamos és útvonal-kérelmek futtatásához. Lásd: [API Versioning](./api-design.md#api-versioning) ennek a problémának a további leírását.

**TLS-titkosítás és a kölcsönös TLS hitelesítés**. Biztonsági okokból érdemes lehet a TLS-szolgáltatások közötti forgalom titkosításához és TLS kölcsönös hitelesítés hitelesítéshez használni kívánt hívókat.

## <a name="synchronous-versus-asynchronous-messaging"></a>Szinkron és aszinkron üzenetkezelési

Nincsenek két alapvető üzenetkezelési mintát, mikroszolgáltatások létrehozására használhat más mikroszolgáltatások folytatott kommunikációhoz. 

1. Szinkron kommunikációt. Ebben a mintában a szolgáltatás hívja az API-t, amely egy másik szolgáltatás elérhetővé teszi, például a http- vagy gRPC protokoll használatát. Ez a beállítás nem egy aszinkron üzenettovábbítási mintának, mert a hívó a fogadó válaszára várakozik. 

2. Aszinkron üzenettovábbítási. Ebben a mintában a szolgáltatás üzenetet küld a válaszra való várakozás nélkül, és egy vagy több szolgáltatás aszinkron módon feldolgozni az üzenetet.

Fontos aszinkron i/o- és egy aszinkron protokoll megkülönböztetésére. Aszinkron I/O azt jelenti, hogy a hívó szál nincs letiltva, amíg az i/o-végzi. Amely fontos a teljesítmény, de egy megvalósítási részletes architektúrája tekintetében. Egy aszinkron protokoll azt jelenti, hogy a küldő nem várja meg a választ. HTTP egy szinkron protokoll, annak ellenére, hogy a HTTP-ügyfelek használhatják aszinkron i/o kérelmet küld. 

Nincsenek mellékhatásokkal minden típus. Kérelem-válasz egy jól érthető összeállítást,, így a több mint egy üzenetkezelési rendszerek tervezése természetes tervezése az API-k érezhetik. Azonban aszinkron üzenetkezelési előnye is van bizonyos mikroszolgáltatások architektúra nagyon hasznosak lehetnek:

- **Kapcsolási csökkenteni**. Az üzenet küldője nem kell tudnia a fogyasztó. 

- **Több előfizető**. Egy pub/sub modellt használ, több felhasználóból is kérheti események. Lásd: [eseményvezérelt architektúra stílus](/azure/architecture/guide/architecture-styles/event-driven).

- **Hiba elkülönítési**. Ha a fogyasztó nem sikerül, a küldő továbbra is küldhet üzeneteket. Az üzenetek fog felvenni, ha a fogyasztó állítja helyre. Ez a lehetőség akkor különösen hasznos mikroszolgáltatások architektúra esetén, mert minden egyes szolgáltatás saját életciklus. A szolgáltatás elérhetetlenné válik, vagy egy adott időpontban lecserél egy újabb verzióra. Aszinkron üzenetkezelési időszakos állásidő képes kezelni. Szinkron API-k, másrészt megkövetelése az alárendelt szolgáltatás elérhető legyen, vagy a művelet sikertelen lesz. 
 
- **Válaszkészsége**. Egy fölérendelt szolgáltatás gyorsabb válaszolhatnak, ha várja meg az alárendelt szolgáltatásokkal. Ez különösen fontos mikroszolgáltatások architektúra. Ha (A B kiszolgálóra, amely meghívja a C, és így tovább) hívásokról függőségei láncolata, vár a szinkron hívások adhat hozzá késés mennyiségű nem fogadható el.

- **Terheléskiegyenlítés**. A várólista működhet, és a munkaterhelés szinten puffert, hogy a fogadók a saját díj üzenetek feldolgozásához. 

- **Munkafolyamatok**. Várólisták segítségével kezelheti a munkafolyamat jelölőnégyzet-mutat, az üzenet a munkafolyamat minden lépése után.

Van azonban is használatával hatékonyan aszinkron üzenetkezelési némi kihívást.

- **Az üzenetkezelési infrastruktúra kapcsoló**. Egy adott üzenetkezelési infrastruktúra használatával, hogy az infrastrukturális szoros kapcsoló okozhat. Akkor nehezebben fogok tudni váltani egy másik üzenetkezelési infrastruktúra később lesz.

- **Késés**. Végpontok közötti késés művelet válhat, ha az üzenet-várólistákból feltöltve.  

- **Költség**. Magas teljesítmények, jelentős hatással lehet a az üzenetkezelési infrastruktúra költségét.

- **Összetettség**. Aszinkron üzenetkezelési kezelése értéke nem egy jelentéktelen feladat. Duplikált üzenetek, például való másolásával vagy azáltal, hogy a műveletek az idempotent kell kezelni. Egyúttal rögzített kérés-válasz szemantika használatával aszinkron üzenetkezelési végrehajtásához. Visszajelzés küldése szüksége van egy másik várólistához, valamint olyan módon, és választ üzeneteket összefüggéseket.

- **Átviteli sebesség**. Ha üzenetek szükséges *szemantikáját várólistára*, a várólista szűk keresztmetszetet jelenthet a rendszerben. Minden üzenet megköveteli, hogy legalább egy várólista működését, és egy created műveletet. Ezenkívül várólista szemantikáját általában valamilyen zárolás belül az üzenetkezelési infrastruktúra szükséges. Ha a várólista egy felügyelt szolgáltatás, előfordulhat további késést, mert a várólista külső a fürt virtuális hálózathoz. Ezek a problémák mérséklésére vonatkozó útmutatások kötegelés üzenetek által, de, amely növeli a kódot. Ha az üzenetek nem feltétlenül szükséges várólista szemantikáját, valószínűleg létre tudja esemény használandó *adatfolyam* helyett a sor. További információkért lásd: [architekturális stílus eseményvezérelt](../guide/architecture-styles/event-driven.md).  

## <a name="drone-delivery-choosing-the-messaging-patterns"></a>Dron kézbesítési: Az üzenetkezelési mintát kiválasztása

E szempontok figyelembe vételével a fejlesztői csapat végzett a következő tervezési döntések ütköznek azokkal a dron továbbítási alkalmazást hozhat létre a

- Az adatfeldolgozást szolgáltatás közzétesz egy nyilvános REST API ütemezése, frissítésére, vagy szakítsa meg a kézbesítések használó ügyfélalkalmazások.

- Az adatfeldolgozást szolgáltatás Event Hubs aszinkron üzeneteket küldhet a Feladatütemező szolgáltatás használja. Aszinkron üzenetek is végre kell hajtani a terhelés-simítás adatfeldolgozást szükséges. Hogyan működnek együtt a adatfeldolgozást és a Feladatütemező szolgáltatás a részletekért lásd: [adatfeldolgozást és a munkafolyamat][ingestion-workflow].

- Összes fiókot, a kézbesítési, a csomag, a dron és a külső átviteli szolgáltatások teszi közzé a belső REST API-k. A Feladatütemező szolgáltatás meghívja az ezen API-k egy felhasználói kérelem végrehajtásához. Egy szinkron API-k használatára szükség, hogy a Feladatütemező kell az alárendelt szolgáltatások kapott választ. Az alsóbb rétegbeli szolgáltatásai bármelyikének hibája miatt azt jelenti, hogy a teljes művelet sikertelen volt. Azonban lehetséges probléma a mértékű késés, a háttér-szolgáltatások meghívásával bevezetett. 

- Ha bármely alárendelt szolgáltatás nem átmeneti hiba, a teljes tranzakció sikertelenként kell megjelölni. Ebben az esetben kezelni, a Feladatütemező szolgáltatás aszinkron üzenetet küld a felügyelő, hogy a felügyelő ütemezhet kompenzációs tranzakciók a fejezetben leírtak [adatfeldolgozást és a munkafolyamat] [ ingestion-workflow].   

- A kézbesítési szolgáltatás közzétesz egy nyilvános API, amellyel az ügyfelek a szállítási állapotának beolvasása. A fejezet [API átjáró](./gateway.md), hogyan egy API-átjáró az ügyféltől a mögöttes szolgáltatás elrejtése is tárgyaljuk, így az ügyfél nem szükséges tudni, hogy mely szolgáltatások elérhetővé mely API-k. 

- Míg egy dron útban, akkor a dron szolgáltatás események, amelyek tartalmazzák a dron aktuális hely és az állapot küld. A kézbesítési szolgáltatás figyeli a ezeket az eseményeket egy kézbesítési állapotának nyomon követése érdekében.

- A kézbesítési állapotának megváltozásakor a kézbesítési szolgáltatás például küld-e a kézbesítési állapot eseményt `DeliveryCreated` vagy `DeliveryCompleted`. Minden szolgáltatás fizethet ezeket az eseményeket. Az aktuális terv a kézbesítési szolgáltatás csak előfizető, de lehet más előfizetők később. Az események kódhiba például egy valós idejű elemzési szolgáltatás. És a válaszra való várakozás a Feladatütemező nem kell, mert további előfizetők hozzáadása nem befolyásolja a fő munkafolyamat elérési útját.

![](./images/drone-communication.png)

Figyelje meg, hogy kézbesítési állapoteseményeit származó dron hely események. Például ha egy csomag csökken egy dron eléri a kézbesítési hely, a kézbesítési szolgáltatás fordítja le ez a DeliveryCompleted esemény. Ez az tekintetében tartomány modellek számbavétele példát. A fentebb leírt módon dron felügyeleti külön kötött környezetben tartozik. A dron események átadja egy dron fizikai helyét. Kézbesítési eseményeket, másrészt a kézbesítési, amely a különböző üzleti egységek bekövetkező változások jelölik.

## <a name="using-a-service-mesh"></a>A szolgáltatás háló használatával

A *háló szolgáltatás* van, amely kezeli a szolgáltatások közötti kommunikáció software réteg. Az előző szakaszban felsorolt fontos szempont számos megoldására, és helyezze át a problémák elhagyja a mikroszolgáltatások magukat, és egy megosztott rétegbe felelősséget szolgáltatás rácsvonalak úgy lettek kialakítva. A szolgáltatás háló, amely elfogja a fürt mikroszolgáltatások közötti hálózati kommunikációhoz proxyként funkcionál. 

> [!NOTE]
> Szolgáltatás háló látható egy példa a [diplomata mintát](../patterns/ambassador.md) &mdash; egy segítő szolgáltatás által a hálózati kérelmek, az alkalmazás nevében. 

Most, a fő lehetőség közül választhat a Kubernetes szolgáltatás rácsvonal [linkerd](https://linkerd.io/) és [Istio](https://istio.io/). Mindkét technológiát vannak fejlesztik gyorsan. Jelenleg a korábban megírt ebben az útmutatóban Istio legújabb kiadásának 0,2, így továbbra is nagyon új. Azonban az linkerd és Istio is rendelkezik közös bizonyos funkciókat tartalmazza: 

- Terheléselosztás munkamenet szinten, a megfigyelt késések vagy a függőben lévő kérelmek száma alapján. A réteg-4 terheléselosztás Kubernetes által biztosított keresztül javíthatja a teljesítményt. 

- Réteg-7 útválasztási URL-címe, állomásfejléc, API-verzió vagy más alkalmazásszintű szabályok alapján.

- Próbálja meg újra a sikertelen kérelmek. Egy szolgáltatás háló HTTP hibakódok megértette, és automatikusan újra a sikertelen kérelmek. Beállíthatja, hogy a maximális számú újrapróbálkozást, a határidőn ahhoz, hogy a maximális késleltetés kötött együtt. 

- Kör legfrissebb. Ha példány következetesen sikertelen kérelmek, a szolgáltatás háló fog ideiglenesen megjelölése nem érhető el. A leállási idő után újra megpróbálja a a példányt. Konfigurálhatja az áramköri megszakító különböző feltételek, például az egymást követő hibák száma alapján  

- Szolgáltatás háló értekezleteire indított, például a kérést, a késés, a hiba és annak sikeressége díjszabás és a válasz méretek kapcsolatos metrikákat rögzíti. Szolgáltatás háló is lehetővé teszi, hogy elosztott nyomkövetés korrelációs adatokat az egyes ugrások hozzáadásával a kérelemben.

- Szolgáltatások közötti hívások kölcsönös TLS hitelesítés.

Kell-e a szolgáltatás háló? Elosztott rendszer megnövelik értéke alapértékekkel kényszerítő. Ha nem rendelkezik a szolgáltatás rácsvonal, szüksége lesz figyelembe kell venni a fejezet elején említett kihívásai mindegyikének. Megoldható problémák, mint például az újra gombra, áramköri megszakító és elosztott nyomkövetési szolgáltatás rácsvonal nélkül, de a szolgáltatás háló helyezi át a problémák kívül az egyes szolgáltatások és egy dedikált rétegbe. Szolgáltatás rácsvonalak, másrészt viszonylag új technológia, amely még lejáró. A telepítő és a fürt konfigurációjának telepítése egy szolgáltatás háló hozzáadja összetettségét. Előfordulhatnak teljesítményre gyakorolt hatása, mivel a kérelmek most beolvasása keresztül történik a háló proxy, és mivel a felesleges szolgáltatások futnak a fürt minden csomópontján. Hajtsa végre az alapos teljesítmény kell, és terheléses tesztelés szolgáltatás rácsvonal éles környezetben üzembe helyezése előtt.

> [!div class="nextstepaction"]
> [API-Tervező](./api-design.md)

<!-- links -->

[ingestion-workflow]: ./ingestion-workflow.md
