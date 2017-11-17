---
title: "Architektúra eseményvezérelt stílus"
description: "Előnyeit, kihívást és ajánlott eljárásai ismerteti eseményvezérelt és Azure IoT-architektúra"
author: MikeWasson
ms.openlocfilehash: ff7f936ceabefe7079a1ebbfa717ff4095bf133b
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="event-driven-architecture-style"></a>Architektúra eseményvezérelt stílus

Egy eseményvezérelt architektúra áll **esemény gyártók** , amely egy az események streamjét, létrehozása és **eseményfelhasználók** , amely az események figyelését. 

![](./images/event-driven.svg)

Az események azonnal közel valós időben, így a felhasználók válaszolhassanak azonnal események azok bekövetkezésekor. Gyártók vannak különválik fogyasztók &mdash; termelő nem ismert, mely a fogyasztók figyelő. A fogyasztók is le vannak egymástól, és minden felhasználó számára megjelenített összes eseményt. Ez eltér a [versengő fogyasztók] [ competing-consumers] mintát, ahol a fogyasztók lekéréses üzenetek várólistából való várólistából, és csak egyszer dolgozza fel üzenet (feltéve, hogy nincs hiba). Az egyes rendszerek, például az IoT események kell fogyasztanak, nagyon nagy kötetek.

Egy esemény architektúra pub/sub modell vagy egy esemény-adatfolyam modell használhatja. 

- **Pub/sub**: az üzenetkezelési infrastruktúra nyomon követi az előfizetések. Az esemény közzétételekor mindegyik előfizető az esemény küld. Egy esemény fogadását követően nem kell újból, és új előfizetők nem jelenik meg az esemény. 

- **Adatfolyam-esemény**: események kerülnek egy naplóban. Események szigorúan csak rendezett (belül partíció) és tartós. Az ügyfelek az adatfolyam nem előfizetni, ehelyett az adatfolyam valamely része egy ügyfél is olvasható. Az ügyfél felelős a pozíciót az adatfolyamban lehetőségre. Ez azt jelenti, hogy egy ügyfél bármikor csatlakozhat, és képes visszajátszásos események.

A fogyasztó oldalon közös változata is:

- **Egyszerű esemény feldolgozása**. Egy esemény azonnal elindítja a végfelhasználói művelet. Például használhatja az Azure Functions az Service Bus eseményindító, úgy, hogy egy függvény hajt végre, amikor egy Service Bus-témakörbe közzétett egy üzenetet.

- **Az összetett eseményfeldolgozást**. A fogyasztó dolgozza fel események sorozatát keres mintákat az adatok, például az Azure Stream Analytics vagy alatt futó Apache Storm technológiával. Például akkor képes összesíteni szivattyútelepek érzékelőinek adatai az embedded-eszköz egy olyan időkeretet keresztül, és generálása egy értesítést, ha a mozgóátlag keverve használ egy bizonyos küszöböt. 

- **Az adatfolyam Eseményfeldolgozási**. Egy adatfolyam platform, például az Azure IoT-központ vagy az Apache Kafka, mint a betöltési események és hírcsatornát őket adatfolyam processzorok folyamat használja. Az adatfolyam processzorok folyamat való működésre, vagy az adatfolyam átalakítása. Előfordulhat, hogy az alkalmazás különböző alrendszerek több adatfolyam processzor. Erre akkor remekül beválik, ha az IoT-munkaterhelések.

Lehet, hogy a rendszer, például egy IoT-megoldás a fizikai eszközök külső az események forrása. A rendszer ebben az esetben az adatok a kötet és az adatforrás által igényelt átviteli képesnek kell lennie.

Logikai a fenti ábrán minden ügyfél típusú egyetlen mezőben jelenik meg. Az eljárás az gyakori, hogy olyan ügyféllel, kívánja kerülni a fogyasztó a hibaérzékeny pontok kialakulását vált a rendszer több példánya is. Lehet, hogy több példánya is szükségesek, hogy a kötet és az események gyakorisága. Emellett egy-egy fogyasztó előfordulhat, hogy feldolgozni az eseményeket az egyszerre használható szálak. Ez kihívást hozhat létre, ha események sorrendben kell végrehajtani, vagy szükség pontosan-egyszer szemantikáját. Lásd: [minimalizálása érdekében koordinációs][minimize-coordination]. 

## <a name="when-to-use-this-architecture"></a>Mikor érdemes használni, ez az architektúra

- Több alrendszereket kell feldolgozni a azonos események. 
- A minimális időeltolódást valós idejű feldolgozással.
- Összetett Eseményfeldolgozási, például mintaegyezéshez vagy Összesítés idő windows keresztül.
- Nagy adatmennyiség és nagy sebességű adatok, például az IoT.

## <a name="benefits"></a>Előnyök

- Létrehozói és felhasználói le.
- Nincs pont közötti pont-integrációja. Akkor is könnyen új felhasználók hozzáadása a rendszer.
- Felhasználók válaszolhassanak események azonnal azok az ügyfélszámítógépekre érkeznek. 
- Rugalmasan méretezhető, terjesztett. 
- A alrendszereket az eseménystream független nézetek.

## <a name="challenges"></a>Kihívásai

- Garantált kézbesítés. Az egyes rendszerek, különösen az IoT-forgatókönyvek esetén rendkívül fontos garantálja, hogy kézbesíti az eseményeket.
- Feldolgozási sorrendben események vagy pontosan egyszer. Minden felhasználói típus általában több példányát, a rugalmasság és a méretezhetőség fut. Ez kihívást hozhat létre, ha az események (tartozó ügyfél) sorrendben kell végrehajtani, vagy ha a feldolgozó logika nem idempotent.

## <a name="iot-architecture"></a>IoT-architektúra

Eseményvezérelt architektúrák olyan központi helyet foglalnak el az IoT-megoldások. Az alábbi ábra a IoT egy lehetséges logikai architektúráját mutatja be. A diagram az esemény-adatfolyam-összetevők a architektúra emeli ki.

![](./images/iot.png)

A **felhőátjáróhoz** ingests eszköz események, a felhő határt, egy megbízható, alacsony késésű rendszer üzenetküldési használatával.

Eszközök el tudja küldeni események közvetlenül az átjáró, vagy egy **mező átjáró**. A mező az átjáró az speciális eszközt vagy szoftvert, az eszközök általában közösen elhelyezett fogadó eseményeket, és továbbítja őket a felhőátjáróhoz. A mező átjáró is előfordulhat, hogy előfeldolgozása nyers eszköz események, funkciókat, például a szűrést, összesítési vagy protokoll átalakítás végrehajtásához.

Bevitel, után események halad át egy vagy több **adatfolyam-processzorok** , amelyek az adatok (például, hogy a tároló) útvonalát, vagy hajtsa végre a kapcsolódó elemzések és egyéb feldolgozási.

Az alábbiakban néhány gyakori hibatípusokat feldolgozása. (A lista létrehozási biztosan nem teljes.)

- Írás a esemény cold tárhely, az archiválás vagy kötegelt elemzés.

- Elérési út analytics közbeni, az esemény-adatfolyam elemzésekor a (mellett) valós idejű, rendellenességek észlelését ismeri fel az minták keresztül működés közbeni idő windows vagy aktiváltak riasztásokat, ha egy adott feltétel lép fel az adatfolyam. 

- Kezeli a különleges típusú eszközök, például az értesítések és riasztások nem telemetriai üzeneteit. 

- Gépi tanulás.

A listák, amelyek az egyes elemek árnyékolt szürke összetevőket az IoT-rendszer nem közvetlenül kapcsolódó adatfolyam-esemény, de ide tartoznak a teljesség megjelenítése.

- A **eszközbeállításjegyzékben** kiépített eszközt, beleértve az eszköz azonosítóját, és általában eszköz metaadatait, például a hely egy adatbázis.

- A **API kiépítés** egy közös külső felület üzembe helyezési és új eszközök regisztrációja.

- Bizonyos IoT-megoldás lehetővé teszi **parancs és a vezérlő üzenetek** eszközökre küldendő.

> Ebben a szakaszban bemutatott IoT nagyon magas szintű nézetét, és sok subtleties és kihívásokkal kell figyelembe venni. Részletesebb referencia-architektúrában és vitafórum, tekintse meg a [Microsoft Azure IoT-Referenciaarchitektúra] [ iot-ref-arch] (PDF-fájl letöltése).

 <!-- links -->

[competing-consumers]: ../../patterns/competing-consumers.md
[iot-ref-arch]: https://azure.microsoft.com/en-us/updates/microsoft-azure-iot-reference-architecture-available/
[minimize-coordination]: ../design-principles/minimize-coordination.md


