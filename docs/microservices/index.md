---
title: Mikroszolgáltatások tervezése, létrehozása és működtetése az Azure-ban Kubernetes használatával
description: Mikroszolgáltatások tervezése, létrehozása és működtetése az Azure-ban
author: MikeWasson
ms.date: 12/08/2017
ms.openlocfilehash: 857e91a8eeefec18b459f2e66fde9a4f8bbe7b21
ms.sourcegitcommit: 744ad1381e01bbda6a1a7eff4b25e1a337385553
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/08/2018
ms.locfileid: "27701102"
---
# <a name="designing-building-and-operating-microservices-on-azure"></a>Mikroszolgáltatások tervezése, létrehozása és működtetése az Azure-ban

![](./images/drone.svg)

A mikroszolgáltatás egy népszerű architekturális stílussá vált a rugalmas, hatékonyan méretezhető, függetlenül üzembe helyezhető és gyorsan fejleszthető felhőalkalmazások létrehozásához. Azonban ahhoz, hogy ez több legyen, mint egy divatos kifejezés, a mikroszolgáltatások esetén új megközelítés szükséges az alkalmazások tervezésekor és létrehozásakor. 

Ezekben a cikkekben megismerjük, hogyan hozhat létre és futtathat egy mikroszolgáltatási architektúrát az Azure-ban. A témakörök a következők:

- Mikroszolgáltatási architektúra tervezése tartományvezérelt tervezéssel (Domain Driven Design, DDD). 
- A megfelelő Azure-technológiák kiválasztása a kialakítás számítási, tárolási, üzenetküldési és egyéb elemeihez.
- A mikroszolgáltatások tervezési mintáinak ismertetése.
- Rugalmasságot, méretezhetőséget és teljesítményt szem előtt tartó tervezés.
- CI-/CD-folyamatok létrehozása.


Végig egy teljes körű forgatókönyvre koncentrálunk: egy drónos szállítási szolgáltatásra, amely segítségével az ügyfelek csomagok drónos felvételét és kézbesítését ütemezhetik. A referenciaimplementáció kódját a GitHubon találja.

[![GitHub](../_images/github.png) referenciaimplementáció][drone-ri]

Elsőként kezdjük az alapokkal. Mik azok a mikroszolgáltatások, és mik a mikroszolgáltatási architektúra bevezetésének előnyei?

## <a name="why-build-microservices"></a>Miért érdemes mikroszolgáltatásokat létrehozni?

A mikroszolgáltatási architektúrákat kisebb, független szolgáltatások alkotják. Íme néhány a mikroszolgáltatások meghatározó jellemzői közül:

- Mindegyik mikroszolgáltatás egyetlen üzleti képességet valósít meg.
- A mikroszolgáltatások általában elég kis méretűek ahhoz, hogy fejlesztők egy kis csoportja alkalmas legyen a megírásukra és fenntartásukra.
- A mikroszolgáltatások külön folyamatokban futnak, jól meghatározott API-kon vagy üzenetkezelési mintákon keresztül kommunikálva. 
- A mikroszolgáltatások nem osztoznak az adattárolókon vagy adatsémákon. Mindegyik mikroszolgáltatás a saját adatainak kezeléséért felelős. 
- A mikroszolgáltatások külön kódbázissal és forráskóddal rendelkeznek. Használhatnak azonban közös segédprogramkódtárat.
- Mindegyik mikroszolgáltatás a többi szolgáltatástól függetlenül telepíthető és frissíthető. 

Megfelelő felhasználás esetén a mikroszolgáltatások számos hasznos előnyt biztosítanak:

- **Rugalmasság.** Mivel a mikroszolgáltatások egymástól függetlenül üzemelnek, a hibajavítások és a funkciók kiadása egyszerűbben felügyelhető. A szolgáltatásokat a teljes alkalmazás ismételt üzembe helyezése nélkül frissítheti, illetve visszavonhat egy frissítést, ha valami probléma merül fel. Számos hagyományos alkalmazásnál, ha egy hiba merül fel az egyik részében, az megállíthatja a teljes kiadási folyamatot, amely eredményeként az új funkciók bevezetéséhez meg kell várni a hibajavítás integrálását, tesztelését és kiadását.  

- **Kis kód, kis csapatok.** A mikroszolgáltatásoknak elég kis méretűnek kell lenniük ahhoz, hogy egyetlen termékfejlesztő-csapat képes legyen elkészíteni, tesztelni és üzembe helyezni azokat. A kis kódbázisok könnyebben átláthatóak. A nagy, monolitikus alkalmazások esetén idővel gyakran a kódfüggőségek összekuszálódnak, ezért az új funkciók hozzáadásához a kódot számos helyen módosítani kell. Mivel nem osztoznak a kódon vagy adattárakon, a mikroszolgáltatási architektúrák minimalizálják a függőségeket, így az új funkciók hozzáadása egyszerűbben elvégezhető. A kis csapatméret elősegíti a nagyobb fokú rugalmasságot. A „kétpizzás szabály” szerint egy csapatnak elég kicsinek kell lennie ahhoz, hogy két pizzával jóllakjon mindenki. Természetesen ez nem egy pontos metrika, és a csapat étvágyától is függ! A lényeg azonban az, hogy a nagy csoportok általában kevésbé hatékonyak, mivel a kommunikáció lassabb, a vezetési költségek magasabbak és a rugalmasság csökken.  

- **Technológiák vegyítése**. A csapatok kiválaszthatják a szolgáltatásukhoz legmegfelelőbb technológiát, illetve szükség szerint ezek elegyét. 

- **Rugalmasság**. Ha egy adott mikroszolgáltatás elérhetetlenné válik, az nem zavarja meg a teljes alkalmazást, amennyiben a fölérendelt mikroszolgáltatások úgy vannak megtervezve, hogy megfelelően kezeljék a hibákat (például áramköri megszakítás bevezetésével).

- **Méretezhetőség**. A mikroszolgáltatási architektúra lehetővé teszi, hogy az összes mikroszolgáltatást egymástól függetlenül méretezhesse. Így horizontálisan felskálázhatja a több erőforrást igénylő alrendszereket anélkül, hogy a teljes alkalmazást méreteznie kellene. Ha tárolókon belül helyez üzembe szolgáltatásokat, a mikroszolgáltatásokat nagyobb sűrűségben helyezheti egyetlen gazdagépre, így hatékonyabban használhatja fel az erőforrásokat.

- **Adatelkülönítés**. A sémafrissítések végrehajtása sokkal egyszerűbb, mivel csak egyetlen mikroszolgáltatást érintenek. A monolitikus alkalmazásokban a sémafrissítések nagy nehézséget jelenthetnek, mivel az alkalmazás különböző részei ugyanazokat az adatokat használhatják, kockázatossá téve ezzel a séma módosítását.
 
## <a name="no-free-lunch"></a>Nincs ingyenebéd

Ezeket az előnyöket nem adják ingyen. Ennek a cikksorozatnak a célja, hogy segítsen megoldani a rugalmas, méretezhető és felügyelhető mikroszolgáltatások fejlesztésének kihívásait.

- **Szolgáltatáshatárok**. Amikor mikroszolgáltatásokat fejleszt, alaposan meg kell fontolnia, hol húzza meg a határokat a szolgáltatások között. Miután a szolgáltatásokat létrehozta és éles környezetben üzembe helyezte, a határok újratervezése nehézkes lehet. A megfelelő szolgáltatáshatárok kiválasztása az egyik legnagyobb kihívás a mikroszolgáltatási architektúrák tervezésekor. Mekkorák legyenek az egyes szolgáltatások? Mikor érdemes a funkciókat több szolgáltatás között megosztani, és mikor érdemes egy szolgáltatáson belül tartani? Ebben az útmutatóban ismertetünk egy megközelítést, amely tartományvezérelt tervezéssel állapítja meg a szolgáltatások határait. Először [tartományelemzéssel](./domain-analysis.md) azonosítjuk a kapcsolódó kontextusokat, majd [taktikai DDD-minták](./microservice-boundaries.md) készletét alkalmazzuk a funkcionális és nem funkcionális követelmények alapján. 

- **Adatkonzisztencia és -integritás**. A mikroszolgáltatások egyik alapelve, hogy mindegyik szolgáltatás a saját adatait felügyeli. Így a szolgáltatások különállók maradnak, azonban kihívásokkal szembesülhet az adatintegritás vagy a redundancia terén. Ezeket a problémákat az [adatkezelési szempontokkal](./data-considerations.md) foglalkozó témakörben tárgyaljuk.

- **Hálózati torlódás és késleltetés**. A sok kis részletes szolgáltatás használata több szolgáltatások közötti kommunikációt és hosszabb végpontok közötti késleltetést eredményezhet. A [szolgáltatások közötti kommunikációt](./interservice-communication.md) bemutató fejezet ismerteti a szolgáltatások közötti üzenetküldés megfontolandó szempontjait. A szinkron és aszinkron kommunikációnak is van helye a mikroszolgáltatási architektúrákban. Fontos a jó [API-tervezés](./api-design.md), hogy a szolgáltatások lazán összekapcsolva maradjanak, valamint egymástól függetlenül üzembe helyezhetők és frissíthetők legyenek.
 
- **Összetettség**. A mikroszolgáltatás-alkalmazások több mozgó részből állnak. Habár az egyes szolgáltatások egyszerűek lehetnek, együttműködésük elengedhetetlen. Egyetlen felhasználói művelet több szolgáltatást is igénybe vehet. Az [adatfeldolgozást és a munkafolyamatokat](./ingestion-workflow.md) ismertető fejezetben a kérések magasabb átviteli sebességen történő feldolgozásakor, a munkafolyamatok koordinálásakor és a hibák kezelésekor felmerülő problémákat vizsgáljuk meg. 

- **Kommunikáció az ügyfelek és az alkalmazás között.**  Amikor egy alkalmazást több kis szolgáltatásra bont, hogyan kommunikálhatnak az ügyfelek ezekkel a szolgáltatásokkal? Minden egyes szolgáltatást közvetlenül meg kell hívni, vagy a kérések egy [API-átjárón](./gateway.md) keresztül legyenek átirányítva?

- **Figyelés**. Az elosztott alkalmazások figyelése sokkal nehezebb lehet, mint a monolitikus alkalmazásoké, mivel több szolgáltatás telemetriaadatait kell korrelálni. A [naplózást és figyelést](./logging-monitoring.md) ismertető fejezet megoldásokat kínál ezekre a problémákra.

- **Folyamatos integráció és teljesítés (CI/CD)**. A mikroszolgáltatások egyik fő célja a rugalmasság. Ennek érdekében automatizált és robusztus [CI-re/CD-re](./ci-cd.md) van szüksége, hogy gyorsan és megbízhatóan tudjon egyes szolgáltatásokat tesztelési és éles környezetben üzembe helyezni.

## <a name="the-drone-delivery-application"></a>A Drone Delivery alkalmazás

Annak érdekében, hogy körüljárjuk ezeket a problémákat és illusztráljuk a mikroszolgáltatási architektúrák néhány ajánlott eljárását, létrehoztunk egy referenciaimplementációt, a Drone Delivery alkalmazást. A referenciaimplementációt a [GitHubon][drone-ri] találja meg.

A Fabrikam vállalat egy drónos szállítási szolgáltatást indít. A cég egy drónokból álló flottát kezel. A vállalkozások regisztrálnak a szolgáltatásban, és a felhasználók kérhetik egy termék drónos kézbesítését. Amikor az ügyfél ütemezi a felvételt, egy háttérrendszer hozzárendel egy drónt, és értesíti a felhasználót a várható szállítási időről. Amíg a kézbesítés folyamatban van, az ügyfél nyomon követheti a drón helyét, miközben a becsült érkezési időpont folyamatosan frissül.

Ez a forgatókönyv egy viszonylag összetett tartományt foglal magába. A vállalat számára problémát jelenthet a drónok ütemezése, a csomagok nyomon követése, a felhasználói fiókok felügyelete, valamint az előzményadatok tárolása és elemzése. Ezenkívül a Fabrikam gyorsan piacra szeretne lépni, majd gyors iterációkat kíván végezni, új funkciók és képességek hozzáadásával. Az alkalmazásnak felhőszinten kell üzemelnie, magas szolgáltatási szintű célkitűzéssel (service level objective, SLO). A Fabrikam emellett arra számít, hogy a rendszer különböző részeinek nagyon eltérő adattárolási és lekérdezési igényei lesznek. A megfontolandó szempontok alapján a Fabrikam azt a döntést hozta, hogy egy mikroszolgáltatási architektúrát választ a Drone Delivery alkalmazáshoz.

> [!NOTE]
> Amennyiben segítségre van szüksége a mikroszolgáltatási architektúra és egyéb architekturális stílusok közötti választáshoz, tekintse meg az [Azure-alkalmazásarchitektúrákhoz készült útmutatót](../guide/index.md).

A referenciaimplementáció Kubernetest és [Azure Container Service-t (ACS)](/azure/container-service/kubernetes/) használ. A magas szintű architekturális döntések és kihívások azonban minden tárolóvezénylőre érvényesek, beleértve az [Azure Service Fabricet](/azure/service-fabric/) is. 

> [!div class="nextstepaction"]
> [Tartományelemzés](./domain-analysis.md)


<!-- links -->

[drone-ri]: https://github.com/mspnp/microservices-reference-implementation
