---
title: Közzétevő-előfizető minta
description: Engedélyezheti egy alkalmazás események aszinkron módon több érdekelt fogyasztók jelentjük be.
keywords: tervezési minta
author: alexbuckgit
ms.date: 12/07/2018
ms.openlocfilehash: f47792dd60e65cac1ff06928d9ea100ed452933a
ms.sourcegitcommit: a0a9981e7586bed8d876a54e055dea1e392118f8
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/11/2018
ms.locfileid: "53233963"
---
# <a name="publisher-subscriber-pattern"></a>Közzétevő-előfizető minta

Engedélyezheti egy alkalmazás jelentjük be az eseményeket az érdekelt fogyasztók aynchronously több, a fogadók a feladói kapcsoló nélkül.

**Más néven**: Közzétevői/előfizetői üzenetkezelés

## <a name="context-and-problem"></a>Kontextus és probléma

A felhőalapú és elosztott alkalmazások a rendszer összetevői gyakran kell tájékoztatni a más összetevők juthatnak.

Aszinkron üzenetkezelés a fogyasztók feladók szétválaszthatók, és elkerülheti a blokkolja a küldőt, hogy várnia kell a válaszra hatékony módszert. Azonban minden felhasználóhoz dedikált üzenetsor használatával nem hatékonyan méretezhetők több fogyasztó számára. Emellett néhány felhasználói számára érdekes lehet a az információkat csak egy részhalmazát. Hogyan is a küldő jelentjük eseményeket az összes érdekelt fogyasztók identitásuk ismerete nélkül?

## <a name="solution"></a>Megoldás

Vezessen be egy aszinkron üzenetkezelési alrendszer, amely a következőket tartalmazza:

- Egy bemeneti üzenetkezelési csatornát, a küldő által használt. A küldő csomagokat eseményt üzenetek, egy ismert formátumban, és ezeket az üzeneteket a bemeneti csatornán keresztül küld. Ebben a mintában a küldő néven is ismert a *közzétevő*.

  > [!NOTE]
  > A *üzenet* adatcsomagot is van. Egy *esemény* egy üzenet, amely értesíti, módosítása vagy egy lezajlott művelettel kapcsolatos egyéb összetevőket.

- Egy kimeneti / fogyasztói az üzenetkezelési csatornán. A fogyasztók nevezzük *előfizetők*.

- A másolási minden üzenetet a bemeneti csatornáról a kimeneti csatornákhoz iránt, hogy az üzenet összes előfizetőknek mechanizmusa. Ez a művelet általában egy közvetítői szerepet betöltő üzenetsínre broker vagy az esemény például kezeli.

Az alábbi ábrán ez a minta logikai összetevőit:

![Közzétételi-feliratkozási üzenetmintát használó minta](./_images/publish-subscribe.png)
 
Közzétevői/előfizetői üzenetkezelés az alábbi előnyökkel jár:

- Elválasztja az alrendszereket, mégis kommunikálniuk kell. Alrendszerek egymástól függetlenül kezelhetők, és üzeneteket megfelelően kezelhetők, akkor is, ha egy vagy több fogadóval offline állapotban van.

- Ez fokozza a méretezhetőséget, és javítja a válaszképesség, a küldő. A küldő is gyorsan egy üzenet küldése a bemeneti csatornára, majd térjen vissza a core-feldolgozási feladataikat. Az üzenetkezelési infrastruktúra feladata üzenetek érdekelt előfizetők lépnek érvénybe.

- Ez növeli a megbízhatóságot. Aszinkron üzenetkezelés segítséget nyújt az alkalmazások továbbra is zökkenőmentes megnövekedett terhelések és hatékonyabban átmeneti hibák kezelésére.

- Lehetővé teszi a késleltetett vagy ütemezett feldolgozás. Előfizetők megvárhatja, amíg csúcsidőn üzenetek csomópontmetrikák, vagy üzeneteket irányítja, vagy egy meghatározott ütemezés szerint feldolgozva.

- Lehetővé teszi az egyszerűbb integráció különböző platformokon, programozási nyelvek, vagy a kommunikációs protokollokat használó rendszerek közötti, valamint a helyszíni rendszerek és a felhőben futó alkalmazások között.

- Ez megkönnyíti az aszinkron munkafolyamatokat vállalaton belül.

- Ez növeli a testability. Csatornák figyelhető és üzenetek megvizsgálni, vagy egy teljes integráció teszt stratégia részeként naplóz.

- Kockázatok elkülönítése az alkalmazások számára biztosít. Minden alkalmazás alapképességeiről, összpontosíthat, amíg az üzenetkezelési infrastruktúra kezel megbízhatóan juthatnak üzenetek több fogyasztó szükséges. 

## <a name="issues-and-considerations"></a>Problémák és megfontolandó szempontok

A minta megvalósítása során az alábbi pontokat vegye figyelembe:

- **Meglévő technológiák.** Erősen ajánlott elérhető üzenetkezelési termékek és szolgáltatások, amelyek támogatják a közzétételi-feliratkozási modell ahelyett, hogy a saját. Az Azure-ban, fontolja meg [a Service Bus](/azure/service-bus-messaging/) vagy [Event Grid](/azure/event-grid/). További is használható közzétevői/előfizetői üzenetkezelési technológiák közé tartozik a Redis, RabbitMQ és az Apache Kafka.

- **Előfizetés kezelésére.** Az üzenetkezelési infrastruktúra meg kell adni a fogyasztók használatával előfizetés vagy mondja le a rendelkezésre álló csatorna-mechanizmust.

- **Biztonság.** Minden olyan csatorna csatlakozik a biztonsági házirendet, hogy megakadályozza a lehallgatást jogosulatlan felhasználók vagy alkalmazások által kell korlátozni.

- **Üzenetek alkészleteiben.** Az előfizetőink, általában csak a közzétevő által elosztott üzenetek részhalmazát iránt. Üzenetkezelési szolgáltatások gyakran előfizetők által fogadott üzenetek szűkítése engedélyezése:

  - **A témakörök.** Minden témakörhöz tartozik egy dedikált kimeneti csatorna, és mindegyik felhasználó előfizethetnek az összes kapcsolódó témakörökre.
  - **Tartalom szűrése.** Az üzenetek megvizsgálni és elosztott minden üzenet tartalma alapján. Minden egyes előfizetőnek adhatja meg a tartalom van érdekelné.

- **Helyettesítő karakteres előfizetők.** Vegye figyelembe, hogy lehetővé teszi az előfizetők helyettesítő karakterek használatával több témákra iratkozhat fel.

- **Kétirányú kommunikációt.** Közzétételi-csatornák-rendszer előfizetés szerint egyirányú kell kezelni. Ha egy adott előfizető nyugtázási vagy a állapotát továbbítja a közzétevőjének van szüksége, fontolja meg a [kérés/válasz minta](http://www.enterpriseintegrationpatterns.com/patterns/messaging/RequestReply.html). Ez a minta egy csatorna, egy üzenet küldéséhez az előfizető és a egy külön választ csatorna használ a közzétevőjének való kommunikációhoz.

- **Üzenetrendezés.** A sorrendet, amelyben a fogyasztói példányok üzenetek fogadása nem garantált, és nem feltétlenül tükrözi az üzenetek létrehozásának sorrendjét. Érdemes úgy megtervezni rendszert annak biztosításához, hogy üzenetfeldolgozás segít kiküszöbölni minden olyan függőséget sorrendjéről üzenetkezelés, idempotens.

- **Üzenet prioritása.** Egyes megoldások megkövetelheti, hogy a meghatározott sorrendben üzenetek feldolgozása. A [elsőbbségi üzenetsor mintája](priority-queue.md) ellenőrizhetik, hogy az adott üzenetek előtt mások lépnek érvénybe.

- **Ártalmas üzenetek.** Ha egy helytelenül formázott üzenet vagy feladat olyan erőforrásokhoz igényel hozzáférést, amelyek nem érhetők el, az a szolgáltatáspéldány sikertelen működéséhez vezethet. A rendszer üzenetek az üzenetsorban, akadályozhatja meg. Ehelyett rögzítése, és ezeket az üzeneteket részleteit máshol tárolhatja az, hogy azok elemezhetők, ha szükséges.

- **Ismétlődő üzenetek.** Az adott üzenet többszöri lehet küldeni. Ha például a küldő után üzenetküldés egy meghiúsulhat. Majd egy új példányát a küldő előfordulhat, hogy indítsa el, és ismételje meg az üzenetet. Az üzenetkezelési infrastruktúra az ismétlődő üzenetek észlelése és eltávolítása (más néven megszüntetéséhez duping) Üzenetazonosítók annak érdekében, hogy a legtöbb-egyszeri kézbesítési üzenetek kell megvalósítania.

- **Üzenetek lejáratkor.** Előfordulhat, hogy az üzenet élettartama korlátozható. A feldolgozása ezen az időn belül nem történik, ha már nem releváns, és el kell távolítani. A küldő megadhatja egy experiation, amikor az üzenetben található adatokat részeként. Egyetlen külső eseményfogyasztó ellenőrizheti ezt az információt, mielőtt eldönti, az üzleti logikát az üzenethez társított végrehajtásához.

- **Üzenet az ütemezés.** Előfordulhat, hogy ideiglenesen embargoed egy üzenet, és nem kell feldolgozni egy adott dátumig és időpontig. Az üzenet nem elérhetőnek kell lennie a fogadó ez időpontig.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes ezt a mintát használni?

Használja ezt a mintát, ha:

- Egy alkalmazás adatokat küldhet a fogyasztók jelentős számú kell.

- Egy alkalmazás kell kommunikálnia egymástól függetlenül fejlesztett alkalmazásokat vagy szolgáltatásokat, amelyek különböző platformokon, programozási nyelvek és kommunikációs protokollt használhat.

- Egy alkalmazás képes információkat küldeni a fogyasztók számára anélkül, hogy a felhasználók valós idejű parancsválaszait.

- Az integráció rendszerek célja a végleges konzisztenciájú modellt támogatja az adatok.

- Egy alkalmazás több fogyasztó, előfordulhat, hogy különböző rendelkezésre állási követelmények vagy üzemidő ütemezéseket, mint a küldő felé történő információtovábbítást kell.

Nem érdemes ezt a mintát használni, ha:

- Egy alkalmazásnak csak néhány felhasználói, akiknek szükség van az előállító alkalmazás jelentősen eltérő adatait.

- Egy alkalmazás közel valós idejű interakciót fogyasztóval rendelkező igényel.

## <a name="example"></a>Példa

Az alábbi ábrán látható egy vállalati integrációs architektúra Service Bust használó munkafolyamatok koordinálása, és az Event Grid értesítheti alrendszerek bekövetkező eseményekről. További információkért lásd: [Azure-beli vállalati integráció üzenetek, várólisták és események](../reference-architectures/enterprise-integration/queues-events.md).

![Vállalati integráció architektúrája](../reference-architectures/enterprise-integration/_images/enterprise-integration-queues-events.png)

## <a name="related-patterns-and-guidance"></a>Kapcsolódó minták és útmutatók

Az alábbi minták és útmutatók hasznosak lehetnek a minta megvalósításakor:

- [Válasszon az Azure-szolgáltatások, amelyek kézbesíti az üzeneteket](/azure/event-grid/compare-messaging-services).

- A [eseményvezérelt architektúra stílusának](../guide/architecture-styles/event-driven.md) olyan architektúrastílus, amely pub/sub üzenetküldési használ.

- [Az aszinkron üzenetkezelés ismertetése](https://msdn.microsoft.com/library/dn589781.aspx). Az üzenetsorok aszinkron típusú kommunikációs mechanizmusok. Ha a feldolgozói szolgáltatásnak választ kell küldenie egy alkalmazásnak, szükség lehet valamilyen válaszüzenet-küldés alkalmazására. Az aszinkron üzenetkezelés ismertetése információkat nyújt a kérelem/válasz típusú üzenetküldés üzenetsorok használatával történő megvalósításával kapcsolatban.

- [A megfigyelő minta](https://en.wikipedia.org/wiki/Observer_pattern). A közzétételi-feliratkozási minta épül, amely a megfigyelő minta az aszinkron üzenetkezelés használatával megfigyelők témák leválasztásával.

- [Üzenet Broker minta](https://en.wikipedia.org/wiki/Message_broker). Számos üzenetkezelő alrendszer, amelyek támogatják a közzétételi-feliratkozási modek vannak megvalósítva egy közvetítő-n keresztül.
