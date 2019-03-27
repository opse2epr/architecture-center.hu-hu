---
title: Üzenetsor-alapú terheléskiegyenlítési minta
titleSuffix: Cloud Design Patterns
description: Használhat egy pufferként szolgáló üzenetsort egy feladat és az általa meghívott szolgáltatás között, hogy kiegyenlítse az időszakos nagy terheléseket.
keywords: tervezési minta
author: dragon119
ms.date: 01/02/2019
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: c736afced1b0478e8eb1a2694acc4d6a6f0c62fc
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299530"
---
# <a name="queue-based-load-leveling-pattern"></a>Üzenetsor-alapú terheléskiegyenlítési minta

Használhat egy pufferként szolgáló üzenetsort egy feladat és az általa meghívott szolgáltatás között, hogy kiegyenlítse az időszakos nagy terheléseket, amelyek miatt a szolgáltatás meghiúsulhat vagy a feladaton időtúllépés történhet. Ez segít minimálisra csökkenteni a rendelkezésre állásra és a válaszképességre irányuló igényekben jelentkező csúcsok hatását mind a feladat, mind a szolgáltatás esetében.

## <a name="context-and-problem"></a>Kontextus és probléma

A felhőben számos megoldás futtat szolgáltatásokat meghívó feladatokat. Ebben a környezetben ha egy szolgáltatás időszakos nagy terheléseknek van kitéve, az a teljesítménnyel és a megbízhatósággal kapcsolatos problémákat okozhat.

A szolgáltatások ugyanazon megoldás részei lehetnek, mint az azt használó feladatok, vagy külső szolgáltatások lehetnek, amelyek hozzáférést nyújtanak a gyakran használt erőforrásokhoz, például egy gyorsítótárhoz vagy egy társzolgáltatáshoz. Ha több egyidejűleg futó feladat ugyanazt a szolgáltatást használja, egy adott időpillanatban nehéz lehet megjósolni a szolgáltatás felé irányuló kérések mennyiségét.

A szolgáltatásban időszakonként csúcsok jelentkezhetnek az igényekben, amelyek túlterhelést okozhatnak, így a szolgáltatás nem tud időben válaszolni a kérésekre. A szolgáltatások számos egyidejű kéréssel való elárasztása is a szolgáltatás meghiúsulását eredményezheti, ha nem tudja kezelni a kérések okozta versenyt.

## <a name="solution"></a>Megoldás

Bontsa újra a megoldást, és vezessen be egy üzenetsort a feladat és a szolgáltatás között. A feladatok és a szolgáltatás aszinkron módon fut. A feladat közzéteszi a szolgáltatás számára szükséges adatokat tartalmazó üzenetet egy üzenetsorba. Az üzenetsor pufferként működik, addig tárolja az üzenetet, amíg azt a szolgáltatás le nem kéri. A szolgáltatás lekéri az üzeneteket az üzenetsorból, és feldolgozza őket. Egy adott üzenetsorban számos különféle feladatból származó kérés adható át a szolgáltatásnak, amelyek rendkívül változó sebességgel jöhetnek létre. Ez az ábra egy szolgáltatás terhelésének kiegyenlítésére szolgáló üzenetsor használatát mutatja be.

![1. ábra – Szolgáltatás terhelésének kiegyenlítésére szolgáló üzenetsor használata](./_images/queue-based-load-leveling-pattern.png)

Az üzenetsor leválasztja a feladatokat a szolgáltatásról, és a szolgáltatás a saját tempójában tudja kezelni az üzeneteket az egyidejű feladatoktól érkező kérések mennyiségétől függetlenül. Emellett a feladatok nem késnek, ha a szolgáltatás nem érhető el, amikor üzenetet tesznek közzé az üzenetsorba.

Ez a minta az alábbi előnyökkel jár:

- Maximálisra növeli a rendelkezésre állást, mert a szolgáltatásokban felmerülő késések nincsenek azonnali és közvetlen hatással az alkalmazásra, amely akkor is folytathatja az üzenetek üzenetsorba való közzétételét, amikor a szolgáltatás nem érhető el vagy épp nem dolgoz fel üzeneteket.
- Segít a skálázhatóság maximalizálásában, mert az üzenetsorok száma és a szolgáltatások száma is módosítható igény szerint.
- A segítségével kézben tarthatók a költségek, mert az üzembe helyezett szolgáltatáspéldányok számának csak az átlagos terheléshez kell elegendőnek lennie, a csúcsterheléshez nem.

    >  Egyes szolgáltatások szabályozást alkalmaznak, amikor az igények elérnek egy olyan küszöbértéket, amelyet követően a rendszer meghibásodhat. A szabályozás csökkentheti az elérhető funkciókat. Ezekkel a szolgáltatásokkal terheléskiegyenlítést valósíthat meg annak érdekében, hogy ne érje el ezt a küszöbértéket.

## <a name="issues-and-considerations"></a>Problémák és megfontolandó szempontok

A minta megvalósítása során az alábbi pontokat vegye figyelembe:

- Olyan alkalmazáslogikát kell megvalósítani, amely szabályozza a szolgáltatások általi üzenetkezelés sebességét, hogy a cél erőforrás ne legyen túlterhelve. Ne adjon át igénybevételi csúcsokat a rendszer más részeinek. Tesztelje a terhelés alatti rendszert annak ellenőrzéséhez, hogy az biztosítja-e a megfelelő terheléskiegyenlítést, és ennek megfelelően módosítsa az üzeneteket kezelő üzenetsorok és szolgáltatáspéldányok számát.
- Az üzenetsorok egyirányú kommunikációs mechanizmusok. Ha egy feladat válaszra vár egy szolgáltatástól, előfordulhat, hogy olyan mechanizmust kell implementálni, amellyel a szolgáltatás választ tud küldeni. További információkért lásd [az aszinkron üzenetkezelés ismertetését](https://msdn.microsoft.com/library/dn589781.aspx).
- Legyen körültekintő, ha az üzenetsor kéréseit figyelő szolgáltatásokra automatikus skálázást alkalmaz. Ez megnövekedett versenyt eredményezhet az ezen szolgáltatások által megosztott erőforrásokért, és csökkenti az üzenetsor hatékonyságát a terhelés kiegyenlítése terén.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes ezt a mintát használni?

Ez a minta olyan alkalmazásokhoz hasznos, amelyek gyakran túlterhelt szolgáltatásokat használnak.

Ez a minta nem használható jól, ha az alkalmazás minimális késéssel vár választ a szolgáltatástól.

## <a name="example"></a>Példa

Webes alkalmazás írja az adatokat egy külső adattárba. A nagy mennyiségű példánnyal a webalkalmazás fut egyidejűleg, ha az adattár nem elég gyorsan válaszolni a kérésekre, kérelmek okoz időtúllépést, szabályozva, vagy ellenkező esetben sikertelen lehet. Az alábbi ábrán látható egy adattár példányairól nagyszámú egyidejű kéréseit alkalmazáspéldányok által.

![2. ábra – példányairól nagyszámú egyidejű kéréseit webalkalmazás példánya által szolgáltatásként](./_images/queue-based-load-leveling-overwhelmed.png)

A probléma megoldásához használhatja az alkalmazáspéldányok és az adattároló közötti terhelés kiegyenlítése érdekében egy üzenetsorba. Az Azure Functions-alkalmazás beolvassa az üzeneteket az üzenetsorból, és hajtja végre az olvasási/írási kérések az adattárba. A függvényalkalmazás az alkalmazáslogikát szabályozhatja a sebesség, amellyel kéréseket továbbít a tárolót, a tároló legyenek kihasznált elkerülése érdekében. (Ellenkező esetben a függvényalkalmazás csak újra vezet be, a háttéralkalmazás ugyanez a probléma.)

![3. ábra – üzenetsor és a egy függvényalkalmazás használata a terhelés kiegyenlítése érdekében](./_images/queue-based-load-leveling-function.png)



## <a name="related-patterns-and-guidance"></a>Kapcsolódó minták és útmutatók

Az alábbi minták és útmutatók szintén hasznosak lehetnek a minta megvalósításakor:

- [Az aszinkron üzenetkezelés ismertetése](https://msdn.microsoft.com/library/dn589781.aspx). Az üzenetsorok eredendően aszinkron típusúak. Előfordulhat, hogy egy feladatban újra kell tervezni az alkalmazáslogikát a szolgáltatással való közvetlen kommunikációról üzenetsor használatára. Hasonlóképpen szükség lehet a szolgáltatások újrabontására, hogy kéréseket fogadjanak el egy üzenetsortól. Másik lehetőségként meg lehet valósítani egy proxyszolgáltatást a példában bemutatott módon.

- [Versengő felhasználókat ismertető minta](./competing-consumers.md). Egy szolgáltatás több példánya is futtatható lehet, amelyek mindegyike üzenetfogyasztóként viselkedik a terheléskiegyenlítő üzenetsorból. Ezzel a módszerrel beállíthatja az üzenetek szolgáltatásból való fogadásának és a szolgáltatásba küldésének sebességét.

- [Szabályozási minta](./throttling.md). A szolgáltatás általi szabályozás megvalósításának egyszerű módja az üzenetsoralapú terheléskiegyenlítés és a szolgáltatásokra érkező összes kérés átirányítása egy üzenetsorba. A szolgáltatás olyan sebességgel dolgozza fel a kéréseket, amely biztosítja, hogy a szolgáltatás számára szükséges erőforrások ne merüljenek ki, és hogy csökkenjen az esetleges verseny mennyisége.

- [Válassza ki Azure-üzenetkezelési szolgáltatások közötti](/azure/event-grid/compare-messaging-services). Információkat biztosít az üzenetkezelési és sorkezelési mechanizmusok kiválasztásáról Azure-alkalmazásokban.

- [Azure-webalkalmazások méretezhetőség javítása](../reference-architectures/app-service-web-app/scalable-web-app.md). Ez a referenciaarchitektúra magában foglalja a üzenetsor-alapú terheléskiegyenlítési az architektúra részeként.
