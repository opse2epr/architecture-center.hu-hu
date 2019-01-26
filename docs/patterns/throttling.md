---
title: Szabályozási minta
titleSuffix: Cloud Design Patterns
description: Szabályozhatja egy alkalmazáspéldány, egyéni bérlő vagy teljes szolgáltatás által használt erőforrások felhasználását.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 0bbe8177abe708cf41c1b5a8d117c05fd280c948
ms.sourcegitcommit: 3b15d65e7c35a19506e562c444343f8467b6a073
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/25/2019
ms.locfileid: "54908596"
---
# <a name="throttling-pattern"></a>Szabályozási minta

[!INCLUDE [header](../_includes/header.md)]

Szabályozhatja egy alkalmazáspéldány, egyéni bérlő vagy teljes szolgáltatás által használt erőforrások felhasználását. Ez a minta lehetővé teszi, hogy a rendszer tovább működjön és teljesítse a szolgáltatói szerződések feltételeit akkor is, ha az erőforrásigény növekedése rendkívüli terhelést jelent az erőforrások számára.

## <a name="context-and-problem"></a>Kontextus és probléma

A felhőalapú alkalmazások terhelése jellemzően az aktív felhasználók számától vagy az általuk végrehajtott tevékenységek típusától függően idővel változik. Például valószínűleg több felhasználó lesz aktív munkaidőben, vagy lehet, hogy a rendszernek nagy számítási igényű elemzéseket kell végeznie minden hónap végén. Hirtelen és váratlan tevékenységcsúcsok is előfordulhatnak. Ha a rendszer feldolgozási követelményei meghaladják a rendelkezésre álló erőforrások kapacitását, az a rendszer gyenge teljesítményét vagy meghibásodását okozhatja. Ha a rendszernek egy megállapodott szolgáltatási szintet kell teljesítenie, az ilyen meghibásodás nem fogadható el.

Számos különféle stratégia létezik a változó terhelés kezelésére a felhőben az alkalmazás üzleti céljaitól függően. Az egyik stratégia a kiépített erőforrások a felhasználók igényeihez igazítása automatikus skálázás alkalmazásával. Ez lehetővé teszi a felhasználói igényeknek való folyamatos megfelelést és a futtatási költségek optimalizálását. Azonban bár az automatikus skálázás elindíthatja további erőforrások kiépítését, a kiépítés nem azonnal történik meg. Ha az igények gyorsan növekednek, előfordulhat, hogy bizonyos ideig erőforráshiány lép fel.

## <a name="solution"></a>Megoldás

Az automatikus skálázás alternatívája, ha az alkalmazások egy adott határig használhatják az erőforrásokat, és ha elérték ezt a határt, a rendszer szabályozza őket. A rendszernek monitoroznia kell az erőforrások felhasználását, így amikor az erőforrás-használat túllépi a küszöbértéket, szabályozhatja az egy vagy több felhasználótól érkező kérelmeket. Így a rendszer továbbra is működőképes lesz, és teljesíteni tudja az érvényben levő szolgáltatói szerződések (SLA-k) feltételeit. Az erőforrás-használat monitorozásával kapcsolatos további információkért tekintse meg a [rendszerállapottal és a telemetriával kapcsolatos útmutatást](https://msdn.microsoft.com/library/dn589775.aspx).

A rendszer számos szabályozási stratégiát implementálhat, többek között a következőket:

- Olyan egyéni felhasználóktól érkező kérelmek visszautasítása, akik másodpercenként több mint n-szer fértek hozzá a rendszer API-jaihoz egy adott időtartamban. Ehhez a rendszernek mérnie az egy adott alkalmazást futtató összes bérlő vagy felhasználó erőforrás-felhasználását. További információkért tekintse meg a [szolgáltatások mérésével kapcsolatos útmutatót](https://msdn.microsoft.com/library/dn589796.aspx).

- Bizonyos nem létfontosságú szolgáltatások működésének letiltása vagy csökkentése annak érdekében, hogy a létfontosságú szolgáltatások akadálytalanul, elegendő erőforrással futhassanak. Ha például az alkalmazás videókimenetet streamel, alacsonyabb felbontásra válthat.

- A tevékenységek mennyiségének egyenletessé tétele terheléskiegyenlítés használatával (erről a megközelítésről bővebben az [üzenetsor-alapú terheléskiegyenlítési mintával](./queue-based-load-leveling.md) kapcsolatos részben olvashat). Ez a megközelítés több-bérlős környezetben az összes bérlő esetében teljesítménycsökkenést jelent. Ha a rendszernek különböző SLA-kkal rendelkező bérlőket kell kiszolgálnia, az értékes bérlők műveletei azonnal végrehajthatóak. Más bérlők kérelmei visszatarthatóak addig, amíg a várólista ki nem ürült. Ez a megközelítés az [elsőbbségi üzenetsor mintája](./priority-queue.md) alapján implementálható.

- Az alacsonyabb prioritású alkalmazások vagy bérlők nevében végrehajtott műveletek elhalasztása. Ezek a műveletek felfüggeszthetőek vagy korlátozhatóak, a bérlő pedig az előállított kivétel formájában kap tájékoztatást arról, hogy a rendszer foglalt, és a műveletet később újból meg kell kísérelni.

Az ábrán az erőforráshasználat területdiagramja látható (a memória, CPU, sávszélesség és más tényezők kombinációja) az időben olyan alkalmazásokra vonatkozóan, amelyek három funkciót használnak. A funkció egy működési terület, például egy összetevő, amely adott feladatokat végez, egy kódrészlet, amely egy összetett számítást hajt végre, vagy egy elem, amely valamilyen szolgáltatást – például memórián belüli gyorsítótárat – biztosít. Ezeket a funkciókat az A, B és C jelöli.

![1. ábra – Grafikon, amely három felhasználó nevében futó alkalmazások erőforrás-használatát ábrázolja az idő függvényében](./_images/throttling-resource-utilization.png)

> A közvetlenül az egyes funkciók vonalai alatti terület az alkalmazások által ezen funkciók igénybevételekor felhasznált erőforrásokat jelöli. Például az A funkció vonala alatti terület az A funkciót használó alkalmazások erőforráshasználatát, az A funkció vonala és a B funkció vonala közötti terület pedig a B funkciót használó alkalmazások erőforráshasználatát mutatja. Az egyes funkciókhoz tartozó területek összesítésével megkapjuk a rendszer teljes erőforráshasználatát.

Az előző ábra a műveletek késleltetésének hatását szemlélteti. Közvetlenül a T1 időpont előtt az ezen funkciókat használó alkalmazásokhoz lefoglalt erőforrások teljes mennyisége elér egy küszöbértéket (az erőforráshasználat határát). Ezen a ponton fennáll a veszély, hogy az alkalmazások kimerítik a rendelkezésre álló erőforrásokat. Ebben a rendszerben a B funkció kevésbé kritikus, mint az A vagy a C funkció, ezért a rendszer ideiglenesen letiltja, és az eddig általa használt erőforrások felszabadulnak. A T1 és a T2 időpont között az A és a C funkciót használó alkalmazások továbbra is normál módon futnak. Idővel az ezen két funkció által igénybe vett erőforrás-mennyiség eléggé lecsökken ahhoz, hogy a T2 időpontban ismét elegendő kapacitás váljon elérhetővé a B funkció engedélyezéséhez.

Az automatikus skálázás és a szabályozás kombinálható is az alkalmazások válaszkészségének megőrzése és az SLA-knak való megfelelés fenntartása érdekében. Ha az erőforrásigény várhatóan magas marad, a szabályozás ideiglenes megoldást biztosít, amíg a rendszer elvégzi a horizontális felskálázást. Ezen a ponton a rendszer teljes működőkészsége visszaállítható.

A következő ábrán a rendszeren futó összes alkalmazás teljes erőforrás-használatának grafikonja látható az idő függvényében, illusztrálva a szabályozás és az automatikus skálázás kombinálását.

![2. ábra – A szabályozás és az automatikus skálázás kombinálásának hatásait ábrázoló grafikon](./_images/throttling-autoscaling.png)

A T1 időpontban a rendszer eléri az erőforráshasználat enyhe korlátjának megfelelő küszöbértéket. A rendszer ezen a ponton megkezdheti a horizontális felskálázást. Ha azonban az új erőforrások nem válnak elég gyorsan elérhetővé, a meglévő erőforrások kimerülhetnek, és a rendszer meghibásodhat. Ennek elkerülése érdekében a rendszer ideiglenesen szabályozva lesz a korábban leírtaknak megfelelően. Ha az automatikus skálázás befejeződött, és a további erőforrások elérhetővé váltak, a szabályozás mértéke csökkenthető.

## <a name="issues-and-considerations"></a>Problémák és megfontolandó szempontok

A minta megvalósítása során az alábbi pontokat vegye figyelembe:

- Az alkalmazások szabályozása és a felhasznált stratégia az architektúrával kapcsolatos döntés, amely a rendszer teljes kialakítására hatással van. A szabályozást az alkalmazástervezési folyamat korai szakaszában kell megfontolni, mert ezt a stratégiát nem könnyű már implementált rendszerhez hozzáadni.

- A szabályozást gyorsan kell végrehajtani. A rendszernek képesnek kell lennie érzékelni a tevékenységek mennyiségének növekedését és megfelelően reagálni. Továbbá az is fontos, hogy a rendszer képes legyen gyorsan visszaállni az eredeti állapotába, miután csökkent a terhelés. Ennek feltétele a megfelelő teljesítményadatok folyamatos rögzítése és monitorozása.

- Ha egy szolgáltatásnak ideiglenesen meg kell tagadnia egy felhasználói kérelmet, konkrét hibakódot kell visszaadnia, amely tájékoztatja az ügyfélalkalmazást arról, hogy a művelet végrehajtása szabályozás miatt lett visszautasítva. Az ügyfélalkalmazás várhat egy ideig a kérelem újbóli megkísérlése előtt.

- A szabályozás ideiglenes intézkedésként használható, amíg a rendszer elvégzi az automatikus skálázást. Egyes esetekben jobb csak szabályozást alkalmazni skálázás helyett, ha a tevékenységcsúcs hirtelen jelentkezik, és várhatóan nem tart sokáig, ugyanis a skálázás jelentősen megnövelheti a futtatási költségeket.

- Ha a szabályozás ideiglenes intézkedésként van alkalmazva, mialatt a rendszer automatikus skálázást alkalmat, és ha az erőforrásigény nagyon gyorsan növekszik, előfordulhat, hogy&mdash;a rendszer nem tud tovább működni akkor sem, ha szabályozott üzemmódban van. Ha ez nem fogadható el, vegye fontolóra nagyobb kapacitástartalék fenntartását és az erőteljesebb automatikus skálázást.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes ezt a mintát használni?

Használja a következő mintát a következő helyzetekben:

- Annak biztosítása érdekében, hogy a rendszer folyamatosan megfeleljen a szolgáltatói szerződések feltételeinek.

- Annak elkerülése érdekében, hogy egyetlen bérlő kisajátíthassa az alkalmazások által biztosított erőforrásokat.

- A tevékenységcsúcsok kezelése érdekében.

- A rendszerköltségek optimalizálása érdekében a működéshez szükséges maximális erőforrásszintek korlátozása révén.

## <a name="example"></a>Példa

Az utolsó ábrán a szabályozás több-bérlős rendszerben történő implementálása látható. A bérlővállalatok felhasználói egy felhőben futtatott alkalmazáshoz férnek hozzá, amelyben kérdőíveket töltenek ki és küldenek el. Az alkalmazás rendszerállapota monitorozza a felhasználóktól az alkalmazáshoz érkező kérelmek küldési gyakoriságát.

Annak érdekében, hogy egy bérlő felhasználói miatt ne csökkenjen az alkalmazás válaszkészsége és rendelkezésre állása a többi felhasználó számára, a rendszer az egyes bérlők felhasználói által egy másodperc alatt elküldhető kérelmek számára vonatkozó korlátot vezet be. Az ezen korlátot meghaladó kérelmeket az alkalmazás blokkolja.

![3. ábra – Szabályozás implementálása több-bérlős alkalmazásban](./_images/throttling-multi-tenant.png)

## <a name="related-patterns-and-guidance"></a>Kapcsolódó minták és útmutatók

Az alábbi minták és útmutatók szintén hasznosak lehetnek a minta megvalósításakor:

- [Rendszerállapot és telemetria – útmutató](https://msdn.microsoft.com/library/dn589775.aspx). A szabályozás alapját a szolgáltatások igénybevételének mértékére vonatkozó adatok gyűjtése képezi. Ismerteti, hogyan állíthat elő és rögzíthet egyéni monitorozási adatokat.
- [Szolgáltatások mérésével kapcsolatos útmutató](https://msdn.microsoft.com/library/dn589796.aspx). Ismerteti, hogyan mérheti a szolgáltatások használatát a használatuk jellegének megismerése céljából. Ez az információ hasznos lehet a szolgáltatás szabályozásának megtervezéséhez.
- [Útmutató az automatikus skálázáshoz](https://msdn.microsoft.com/library/dn589774.aspx). A szabályozás használható ideiglenes intézkedésként, amíg a rendszer elvégzi az automatikus skálázást, vagy azért, hogy ne legyen szükség automatikus skálázásra. Az automatikus skálázási stratégiákra vonatkozó információkat tartalmaz.
- [Üzenetsor-alapú terheléskiegyenlítési minta](./queue-based-load-leveling.md). Az üzenetsor-alapú terheléskiegyenlítés a szabályozás implementálásához gyakran használt mechanizmus. Az üzenetsor képes pufferként működni, amely segít egyenletesebbé tenni a szolgáltatásokhoz érkező, alkalmazások által küldött kérelmek kézbesítési gyakoriságát.
- [Elsőbbségi üzenetsor mintája](./priority-queue.md). Az elsőbbségi üzenetsor szabályozási stratégiába való beépítésével a rendszer képes fenntartani a teljesítményszintet a kritikus fontosságú vagy nagyértékű alkalmazásokhoz, míg a kevésbé fontos alkalmazások teljesítményét csökkenti.
