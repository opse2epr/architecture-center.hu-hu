---
title: Kompenzáló tranzakció mintája
titleSuffix: Cloud Design Patterns
description: Visszavonhat egy sorozatnyi, együttesen végül konzisztens műveletet meghatározó lépés által végrehajtott munkát.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 5cefef36b7c763324de2e962d47509b1d1b2c968
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298718"
---
# <a name="compensating-transaction-pattern"></a>Kompenzáló tranzakció mintája

[!INCLUDE [header](../_includes/header.md)]

Visszavon egy sor, együtt végül egy konzisztens műveletet alkotó lépésben végrehajtott munkát, ha egy vagy több lépés meghiúsul. A végleges konzisztenciájú modellt követő műveletek általában olyan felhőben üzemeltetett alkalmazásokban találhatók, amelyek összetett üzleti folyamatokat és munkafolyamatokat valósítanak meg.

## <a name="context-and-problem"></a>Kontextus és probléma

A felhőben futó alkalmazások gyakran módosítják az adatokat. Ezek az adatokat számos, különböző földrajzi helyeken lévő adatforrás között lehetnek elosztva. Az elosztott környezetben a versengés elkerülése és a teljesítmény javítása érdekében az alkalmazásnak nem az erős tranzakció-konzisztencia biztosítására kell törekednie. Ehelyett az alkalmazásnak a végleges konzisztenciát kell megvalósítania. Ebben a modellben egy tipikus üzleti tevékenység különálló lépések sorozatából áll. A lépések végrehajtása közben a rendszer állapotának átfogó képe inkonzisztens lehet, amikor azonban a tevékenység befejeződött, és az összes lépés végrehajtása megtörtént, a rendszernek újra konzisztensnek kell lennie.

> Az [Adatkonzisztencia – Ismertető](https://msdn.microsoft.com/library/dn589800.aspx) információkkal szolgál arról, hogy az elosztott tranzakciók miért nem skálázhatók jól, illetve ismerteti a végleges konzisztenciájú modell alapelveit.

A végleges konzisztenciájú modell esetében a kihívást a meghiúsult lépések kezelése jelenti. Ebben az esetben szükség lehet a művelet összes korábbi lépése által elvégzett munka visszavonására. Az adatokat azonban nem lehet egyszerűen visszaállítani, mert a párhuzamos alkalmazáspéldányok esetleg módosíthatták azokat. Még az olyan esetekben is, ahol az adatokat egy párhuzamos alkalmazáspéldány sem módosította, egy lépések visszavonása nem feltétlenül jelenti egyértelműen az eredeti állapot visszaállítását. Előfordulhat, hogy különböző üzletspecifikus szabályok alkalmazása válik szükségessé (lásd a Példa szakaszban bemutatott utazási webhely esetét).

Ha a végleges konzisztenciát megvalósító művelet több heterogén adattárolóra is kiterjed, a lépések visszavonása esetében az összes adattárat végig kell járni. Minden egyes adattárban megfelelően kell visszavonni az elvégzett munkát ahhoz, hogy a rendszer ne maradjon inkonzisztens.

A végleges konzisztenciát megvalósító művelet által érintett adatok közül nem feltétlenül található mindegyik adatbázisban. A szolgáltatásorientált architektúra (SOA) környezetében egy művelet meghívhat egy másik műveletet a szolgáltatásban, változást okozva a szolgáltatás által tárolt állapotban. A művelet visszavonásához ezt az állapotváltozást is vissza kell vonni. Ehhez szükség lehet a szolgáltatás ismételt meghívására, illetve egy másik, az eredeti lépés hatását visszavonó művelet végrehajtására.

## <a name="solution"></a>Megoldás

A megoldást egy kompenzáló tranzakció megvalósítása jelenti. A kompenzáló tranzakció lépéseinek vissza kell vonniuk az eredeti művelet lépéseinek hatásait. Előfordulhat, hogy a kompenzáló tranzakció nem tudja egyszerűen visszaállítani a rendszer aktuális állapotát a művelet megkezdése előtti állapotra, mert ez a megoldás felülírná a párhuzamos alkalmazáspéldányok által végrehajtott esetleges módosításokat. Ehelyett egy intelligens folyamatra van szükség, amely a párhuzamos alkalmazáspéldányok által végzett munkát is figyelembe veszi. Ez a folyamat általában alkalmazásspecifikus, és az eredeti művelet által elvégzett munka jellege határozza meg.

Gyakran alkalmazott megközelítés egy munkafolyamat segítségével megvalósítani egy végül konzisztens műveletet, amely esetében kompenzációra van szükség. Az eredeti műveletet végrehajtása során a rendszer információkat rögzít minden lépésről, illetve arról, hogyan lehet visszavonni az általuk elvégzett munkát. Ha a művelet bármely ponton meghiúsul, a munkafolyamat visszafelé haladva végigveszi az elvégzett lépéseket, és az összes esetében végrehajtja a visszavonáshoz szükséges munkát. Fontos megjegyezni, hogy a kompenzáló tranzakció nem feltétlenül az eredeti műveletnek megfelelő sorrendben vonja vissza az elvégzett munkát, és egyszerre több visszavonási lépést is végrehajthat.

> Ez a gyakorlat nagyban hasonlít a [Clemens Vasters blogjában](https://vasters.com/clemensv/2012/09/01/Sagas.aspx) tárgyalt Sagas-stratégiához.

A kompenzáló tranzakció is egy végül konzisztenssé váló művelet, amely szintén meghiúsulhat. A rendszernek képesnek kell lennie a kompenzáló tranzakció helyreállítására az adott hibánál, majd folytatni a műveletet. Előfordulhat, hogy egy meghiúsult lépést meg kell ismételni, ezért a kompenzáló tranzakcióban szereplő lépéseket idempotens parancsokként kell meghatározni. További információ: [Idempotens minták](https://blog.jonathanoliver.com/idempotency-patterns/) (Jonathan Oliver blogjában).

Bizonyos esetekben előfordulhat, hogy egy meghiúsult lépés helyreállítása csak manuális intézkedés révén lehetséges. Ilyen helyzetben a rendszernek riasztást kell létrehoznia, és a lehető legtöbb információt kell biztosítania a hiba okairól.

## <a name="issues-and-considerations"></a>Problémák és megfontolandó szempontok

A minta megvalósítása során az alábbi pontokat vegye figyelembe:

Nem mindig egyszerű megállapítani, hogy a végleges konzisztenciát megvalósító művelet adott lépése mikor hiúsult meg. Előfordulhat, hogy egy lépés nem azonnal hiúsul meg, de elakadást okoz. Egyes esetekben szükséges lehet valamilyen időtúllépési mechanizmus bevezetésére.

A kompenzáció logikáját nem könnyű általánosítani. A kompenzáló tranzakciók alkalmazásspecifikusak. A meghiúsult művelet lépéseinek hatása akkor vonható vissza, ha az alkalmazás ehhez elegendő információval rendelkezik.

A kompenzáló tranzakción belüli lépéseket idempotens parancsokként kell meghatározni. Ez a kompenzáló tranzakció meghiúsulása esetén lehetővé teszi a lépések megismétlését.

Az eredeti művelet lépéseit és a kompenzáló tranzakciót kezelő infrastruktúrának rugalmasnak kell lennie. Nem veszítheti el a meghiúsult lépések kompenzációjához szükséges információkat, továbbá megbízhatóan kell monitoroznia a kompenzációs logika előrehaladásának állapotát.

A kompenzáló tranzakció nem feltétlenül az eredeti művelet kezdetén fennálló állapotba állítja vissza az adatokat a rendszerben. Ehelyett a művelet meghiúsulása előtt sikeresen végrehajtott lépések által elvégzett munkát kompenzálja.

A kompenzáló tranzakció lépéseinek sorrendje nem feltétlenül az eredeti művelet lépéseinek a fordítottja. Például ha egy adattár a többinél érzékenyebb az inkonzisztenciákra, a kompenzáló tranzakció lépései először ennek a tárolónak a módosításait vonják vissza.

A műveletet végrehajtó erőforrásokra helyezett rövid távú, időkorlát-alapú zárolás és az erőforrások előzetes beszerzése növeli a tevékenység sikeres végrehajtásának esélyét. A munkát csak az összes erőforrás beszerzését követően szabad végrehajtani. A zárolás lejárta előtt az összes tevékenységet be kell fejezni.

A kompenzáló tranzakciót kiváltó hibák minimalizálása érdekében érdemes lehet olyan újrapróbálkozási logikát alkalmazni, amely a szokásosnál megengedőbb. Ha a végleges konzisztenciát megvalósító művelet egyik lépése meghiúsul, próbálja meg átmeneti kivételként kezelni a hibát, és ismételje meg az adott lépést. Csak akkor állítsa le a műveletet és indítsa el a kompenzáló tranzakciót, ha a lépés ismételten vagy visszavonhatatlanul meghiúsul.

> A kompenzáló tranzakció végrehajtásával kapcsolatban számos hasonló kihívás merül fel, mint a végleges konzisztencia esetében. További információért lásd a végleges konzisztencia megvalósításának szempontjaival foglalkozó részt az [Adatkonzisztencia – Ismertető](https://msdn.microsoft.com/library/dn589800.aspx) témakörben.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes ezt a mintát használni?

Ezt a mintát csak olyan műveletek esetében javasolt használni, amelyeket a meghiúsulásuk esetén vissza kell vonni. Ha lehetséges, keressen olyan megoldásokat, amelyekkel elkerülhető a kompenzáló tranzakciók jelentette összetettség.

## <a name="example"></a>Példa

Az utazási webhely lehetővé teszi, hogy az ügyfelek utakat foglaljanak le. Egy útiterv repülőjárat- és szállásfoglalások egész sorát tartalmazhatja. Az ügyfél, aki Seattle-ből Londonba, majd onnan Párizsba utazik, a következő lépéseket hajthatja végre az útiterv létrehozása során:

1. Helyet foglal a Seattle-ből Londonba tartó F1 repülőjáratra.
2. Helyet foglal a Londonból Párizsba tartó F2 repülőjáratra.
3. Helyet foglal a Párizsból Seattle-be tartó F3 repülőjáratra.
4. Szobát foglal a londoni H1 szállodában.
5. Szobát foglal a párizsi H2 szállodában.

Ezek a lépések egy végül konzisztens műveletet alkotnak, bár minden egyes lépés külön műveletet jelent. Ezért a lépések végrehajtása mellett a rendszernek a visszavonásukhoz szükséges műveleteket is rögzítenie kell arra az esetre, ha az ügyfél úgy dönt, hogy törli az útitervet. Az ellenkező irányú műveletek elvégzéséhez szükséges lépések kompenzáló tranzakcióként futtathatók le.

Megjegyzendő, hogy a kompenzáló tranzakció lépései nem feltétlenül az eredeti lépések pontos ellentétét jelentik, továbbá a kompenzáló tranzakció minden egyes lépésének logikája figyelembe kell vegyen minden vonatkozó üzletspecifikus szabályt. Például ha az ügyfél lemondja a repülőjegyre vonatkozó foglalást, korántsem biztos, hogy visszatérítés is jár neki. Az ábra bemutatja, hogyan hozható létre kompenzáló tranzakció egy utazás foglalására irányuló, hosszú ideje tartó tranzakció visszaállítására.

![Kompenzáló tranzakció létrehozása egy utazás foglalására irányuló, hosszú ideje tartó tranzakció visszaállítására](./_images/compensating-transaction-diagram.png)

> [!NOTE]
> Elvileg lehetséges a kompenzáló tranzakció lépéseinek párhuzamos végrehajtása is, attól függően, hogy az egyes lépések kompenzációs logikája hogyan lett kialakítva.

Számos üzleti megoldás esetében egyetlen hibás lépés nem feltétlenül jelenti azt, hogy az egész rendszert vissza kell állítani egy kompenzáló tranzakcióval. Ha például &mdash;az ügyfél lefoglalja az F1, F2 és F3 repülőjáratot az utazási webhely forgatókönyvében,&mdash;de utána nem tud szobát foglalni a H1 szállodában, a járatfoglalások törlése helyett érdemes inkább a város egy másik szállodáját a figyelmébe ajánlani. Az ügyfél továbbra is dönthet a lemondás mellett (ebben az esetben lefut a kompenzáló tranzakció, és visszavonja az F1, F2 és F3 járatra vonatkozó foglalást), de ezt a döntést az ügyfélnek kell meghoznia, nem pedig a rendszernek.

## <a name="related-patterns-and-guidance"></a>Kapcsolódó minták és útmutatók

Az alábbi minták és útmutatók szintén hasznosak lehetnek a minta megvalósításakor:

- [Adatkonzisztencia – Ismertető](https://msdn.microsoft.com/library/dn589800.aspx). A Kompenzáló tranzakció mintája gyakran használják olyan műveletek visszavonásához, amelyek a végleges konzisztenciájú modell megvalósítását szolgálják. Ez az ismertető a végleges konzisztencia előnyeiről és hátrányairól nyújt tájékoztatást.

- [A Feladatütemező ügynök felügyeleti mintája](./scheduler-agent-supervisor.md). Ismerteti, hogyan lehet olyan rugalmas rendszereket megvalósítani, amelyek elosztott szolgáltatásokat és erőforrásokat használó üzleti műveletek végrehajtását végzik. Egyes esetekben szükség lehet egy művelet által elvégzett munka kompenzáló tranzakcióval történő visszavonására.

- [Újrapróbálkozási minta](./retry.md). A kompenzáló tranzakciók végrehajtása költséges lehet, de az Újrapróbálkozási minta szerint megvalósított, a hibás műveletek megismétlésére vonatkozó hatékony szabályzattal minimálisra lehet csökkenteni a használatukat.
