---
title: Queue-Based Load Leveling
description: Használhat egy pufferként szolgáló üzenetsort egy feladat és az általa meghívott szolgáltatás között, hogy kiegyenlítse az időszakos nagy terheléseket.
keywords: Kialakítási mintája
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- messaging
- availability
- performance-scalability
- resiliency
ms.openlocfilehash: 99b226511fe14bffdab3cdcf65d4e6cffe89bba6
ms.sourcegitcommit: 8ab30776e0c4cdc16ca0dcc881960e3108ad3e94
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/08/2017
---
# <a name="queue-based-load-leveling-pattern"></a>Betöltési simítás mintát várólista-alapú

[!INCLUDE [header](../_includes/header.md)]

Egy feladatot, és meghívja a ahhoz, hogy a szolgáltatás leáll vagy a feladat időtúllépést okozó időszakos nagy terhelések sima szolgáltatás közötti pufferként a várólista használja. Ez segít az igény a rendelkezésre állási és reakcióidőt, mind a tevékenységhez, és a szolgáltatás csúcsait hatás minimalizálása érdekében.

## <a name="context-and-problem"></a>A környezetben, és probléma

Sok megoldásokat a felhőben tartalmaz, amely éppen futó feladatok, amelyek szolgáltatások meghívni. Ebben a környezetben a szolgáltatás van kitéve időszakos túl nagy terhelés esetén okozhat teljesítményt és megbízhatósági hibákat.

Egy szolgáltatás, az azt használó feladatok azonos megoldás részét képező, vagy egy külső szolgáltatás elérését biztosító gyakran használt erőforrások, például a gyorsítótár vagy egy társzolgáltatás lehet. Ha ugyanazt a szolgáltatást használják a párhuzamosan futó feladatok száma, előre jelezni mennyiségű kérést a szolgáltatás bármikor nehézkes lehet.

Egy szolgáltatás esetleges időszakos csúcsidőkbe az igény szerinti azt túlterhelés, és nem válaszol a kérelmekre időben is szembesülhet. Sok egyidejű kérés szolgáltatás elárasztás is eredményezhet a szolgáltatás hiányában nem tudja kezelni a versengés miatt ezek a kérelmek esetén.

## <a name="solution"></a>Megoldás

A megoldás refactor és között a feladatok és a várólista bevezetése. A feladatok és az aszinkron módon futnak. A feladat várólistára szolgáltatáshoz szükséges adatokat tartalmazó üzenet küldi. A várólista pufferként működik egy, az üzenet tárolja, amíg a szolgáltatás lekéri azt. A szolgáltatás beolvassa a a várólistából, és feldolgozza azokat. Számos feladatot, amely magas változó jönnek létre, érkező kérelmeket is átadható a szolgáltatásnak ugyanazon üzenetsorból. Az ábrán látható, hogy egy szolgáltatás terhelése szinten várólista segítségével.

![1. ábra - várólista segítségével szolgáltatás terhelése szinten](./_images/queue-based-load-leveling-pattern.png)

A várólista leválasztja a feladatokat a szolgáltatásból, és a szolgáltatás képes kezelni a saját tempójában függetlenül mennyiségű kérést az egyidejű feladatok üzenetekben. Emellett nem lesz késleltetés tevékenységhez Ha a szolgáltatás nem érhető el, egy üzenetet az üzenetsornak visszaküldés időpontjában.

Ez a minta a következő előnyöket nyújtja:

- Segíthet rendelkezésre állásának maximalizálását, mert késleltetések felmerülő, Services nem rendelkezik az azonnali és közvetlen hatással lehet az alkalmazásról, így továbbra is küldje a várólista-üzenetek, akkor is, ha a szolgáltatás nem érhető el, vagy jelenleg nem dolgoz fel üzenetet.
- Segíthet maximalizálása a méretezhetőség, mert mind a várólisták számát és a szolgáltatások számát is eltérőek lehetnek a igény kielégítéséhez.
- Költségek szabályozása segítségével, mert a telepített service példányok száma csak kell lennie a csúcsterhelés helyett az átlagos terheléssel teljesítéséhez megfelelő.

    >  Egyes szolgáltatások valósítja meg, amelyek sikertelen lehet, a rendszer a küszöbérték elérésekor igény szerinti szabályozás. Sávszélesség-szabályozás csökkentheti a funkció érhető el. Ezekkel a szolgáltatásokkal győződjön meg arról, hogy a ezt a küszöbértéket nem elérte-e simítás terhelés valósíthatja meg.

## <a name="issues-and-considerations"></a>Problémákat és szempontok

Ebben a mintában megvalósításához meghatározásakor, vegye figyelembe a következő szempontokat:

- Is végre kell hajtani az alkalmazáslogikát, amely meghatározza, mely szolgáltatások leíró üzeneteket elkerülése érdekében a célerőforrás overwhelming aránya. Ne igényeiben jelentkező benyújtása igény szerint a rendszer a következő szakaszára. Tesztelje a rendszer biztosítja a szükséges simítás betöltés alatt, és módosítsa úgy a várólisták száma és a szolgáltatáspéldány, amelyek kezelik ennek érdekében üzenetek számát.
- Üzenetsorok egyirányú kommunikációs mechanizmusra. Ha egy feladat válaszára a szolgáltatás vár, egy olyan mechanizmus, amellyel a szolgáltatás visszajelzés végrehajtásához szükség lehet. További információkért lásd: a [aszinkron üzenetkezelési ismertetése](https://msdn.microsoft.com/library/dn589781.aspx).
- Ügyeljen arra, hogy ha automatikus skálázás alkalmaz a szolgáltatásokról, amelyek a figyelő a várólista-kérésekre. Ez minden olyan erőforrásnál, hogy ezek a szolgáltatások megosztani, és a várólista segítségével a terhelés szinten hatékonyságát megnövekedett versengés eredményezhet.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes használni ezt a mintát

Ebben a mintában akkor hasznos, bármely alkalmazás által használt szolgáltatások lépnek túl van terhelve.

Ez a minta nem lehet hasznos, ha az alkalmazás és a minimális mértékű a szolgáltatás választ vár.

## <a name="example"></a>Példa

A Microsoft Azure webes szerepkör tárolja az adatokat egy különálló tárhelyet szolgáltatással. A webes szerepkör példánya nagy számú egyszerre fut, akkor lehetséges, hogy a tároló szolgáltatás nem válaszol a kérelmekre elég gyorsan megakadályozhatja, hogy ezeket a kéréseket a időzítése vagy meghibásodása lesz. Ezt az értéket kiemeli folyamatban túl sok egyidejű kéréseinek webes szerepkör példánya nagy számú szolgáltatás.

![2. ábra – folyamatban túl sok egyidejű kéréseinek webes szerepkör példánya nagy számú szolgáltatás](./_images/queue-based-load-leveling-overwhelmed.png)


A probléma megoldásához, várólista segítségével szinten a terhelést a webes szerepkörpéldányokat és a tárolási szolgáltatás között. Azonban a társzolgáltatás szinkron kérések fogadására van tervezve, és nem lehet egyszerűen módosítani olvashatja és teljesítmény kezelését. A feldolgozói szerepkör kéréseket fogad a várólistából, és továbbítja őket a társzolgáltatás proxy szolgáltatás ügyfélgépként is vezethet. Az alkalmazás logikája a feldolgozói szerepkör szabályozhatják a sebesség, amellyel kérelmeket továbbítja a társzolgáltatás, hogy megakadályozza a társzolgáltatás nem egyszerre. Az ábra azt mutatja be, várólistát és a feldolgozói szerepkör segítségével szinten a terhelést a webes szerepkör példánya és a szolgáltatás között.

![3. ábra - várólista és a feldolgozói szerepkör segítségével szinten a terhelést a webes szerepkör példánya és a szolgáltatás között](./_images/queue-based-load-leveling-worker-role.png)

## <a name="related-patterns-and-guidance"></a>Útmutató és a kapcsolódó minták

A következő mintákat és útmutatókat is lehet releváns ebben a mintában végrehajtása során:

- [Aszinkron üzenetkezelési ismertetése](https://msdn.microsoft.com/library/dn589781.aspx). Üzenet-várólistákból eredendően aszinkron. Az alkalmazás logikája feladat átírása, ha közvetlenül kommunikálni a szolgáltatás számára egy üzenet-várólista használata alkalmazkodik szükség lehet. Hasonlóképpen szükség lehet a refactor egy szolgáltatást, hogy az üzenet-várólista kéréseket fogad. Másik lehetőségként lehet valósíthat meg a proxy szolgáltatás, a példában bemutatott módon.
- [Versengő fogyasztók mintát](competing-consumers.md). Esetleg a szolgáltatás, minden egyes nevében jár el a terhelés simítás várólistából üzenetfogyasztó több példányát futtatni. Ez a módszer segítségével a sebességének, ahol az üzenetek fogadott és átadhat.
- [Sávszélesség-szabályozási mintát](throttling.md). Egy olyan szabályozás megvalósításához legegyszerűbb várólista alapú terhelés simítás és összes kérelem átirányítása egy szolgáltatást, egy üzenet-várólista keresztül. A szolgáltatás, amely biztosítja, hogy a szolgáltatás működéséhez szükséges erőforrások nem nagyon gyorsan kimerítették sebességgel, és kevesebb a versengés, amely akkor fordulhat kérelmek feldolgozásának.
- [Feldolgozási sor szolgáltatással kapcsolatos fogalmak](https://msdn.microsoft.com/library/azure/dd179353.aspx). Az Azure-alkalmazások egy üzenetkezelési és üzenetsor-kezelési mechanizmust kiválasztására vonatkozó adatokat.
