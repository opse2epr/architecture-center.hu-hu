---
title: Feladatütemező ügynök felügyeleti mintája
titleSuffix: Cloud Design Patterns
description: Koordinálhat egy műveletkészletet egy elosztott szolgáltatáskészleten és más távoli erőforrásokon.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: ebfd949e28443a7554426d51eccdcadfed4179ac
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299482"
---
# <a name="scheduler-agent-supervisor-pattern"></a>Feladatütemező ügynök felügyeleti mintája

[!INCLUDE [header](../_includes/header.md)]

Elosztott műveleteket egyetlen műveletként koordinálhat. Ha a műveletek bármelyike sikertelen, próbálja átlátható módon kezelni a hibákat, különben elképzelhető, hogy az elvégzett munkát vissza kell vonni, így a teljes művelet egészében lesz sikeres vagy sikertelen. Ez rugalmasságot nyújthat az elosztott rendszereknek azáltal, hogy lehetővé teszi a helyreállítást és az újrapróbálkozást az átmeneti kivételek, a hosszan tartó hibák és a folyamathibák következtében sikertelen műveletek esetében.

## <a name="context-and-problem"></a>Kontextus és probléma

Egy alkalmazás több lépésből álló feladatokat hajt végre – a lépések némelyike esetleg távoli szolgáltatásokat hív meg vagy távoli erőforrásokhoz fér hozzá. Az egyes lépések egymástól függetlenek is lehetnek, de azokat a feladatot megvalósító alkalmazáslogika vezényli.

Amikor csak lehetséges, az alkalmazásnak biztosítania kell, hogy a feladat befejeződjön, valamint a távoli szolgáltatásokhoz vagy erőforrásokhoz való hozzáférés során esetlegesen fellépő hibákat el kell hárítania. A hibáknak számos oka lehet. Előfordulhat, hogy a hálózat nem működik, a kommunikáció megszakadt, egy távoli szolgáltatás nem válaszol vagy instabil, vagy egy távoli erőforrás átmenetileg nem érhető el, például erőforrás-korlátozás miatt. A legtöbb esetben a hibák átmenetiek, és használatával kezelhetők a [újrapróbálkozási minta](./retry.md).

Ha az alkalmazás nem könnyen helyreállítható állandó hibát észlel, a teljes művelet integritásának biztosítása érdekében képesnek kell lennie visszaállítani a rendszert egy konzisztens állapotba.

## <a name="solution"></a>Megoldás

A feladatütemező ügynök felügyeleti mintája az alábbi aktorokat határozza meg. Ezek az aktorok szervezik meg a teljes feladat részeként végrehajtandó lépéseket.

- Az **ütemező** gondoskodik a feladat lépéseinek végrehajtásáról, és összehangolja a működésüket. Ezek a lépések folyamattá vagy munkafolyamattá egyesíthetők. Az ütemező felelős azért, hogy a munkafolyamat lépései a helyes sorrendben kövessék egymást. Ahogy az egyes lépések végbemennek, az ütemező rögzíti a munkafolyamat állapotát, például: „a lépés még nem kezdődött el”, „a lépés fut”, vagy a „lépés befejeződött”. Az állapotot leíró információnak tartalmaznia kell a lépés befejezéséhez engedélyezett idő felső határát („befejezési idő”). Ha egy lépéshez szükség van távoli szolgáltatáshoz vagy erőforráshoz való hozzáférésre, az ütemező meghívja a megfelelő ügynököt, és átadja neki az elvégzendő munka részleteit. Az ütemező általában aszinkron kérés/válasz típusú üzenetküldés formájában kommunikál az ügynökökkel. Ez üzenetsorok segítségével valósítható meg, bár egyéb elosztott üzenetkezelési technológiák is használhatók.

    > Az ütemező a [folyamatkezelő mintájában](https://www.enterpriseintegrationpatterns.com/patterns/messaging/ProcessManager.html) található folyamatkezelőhöz hasonló funkciót tölt be. A tényleges munkafolyamatot általában egy munkafolyamat-motor definiálja és implementálja, amelyet az ütemező irányít. Ez a megközelítés leválasztja a munkafolyamat üzleti logikáját az ütemezőről.

- Az **ügynök** olyan logikát tartalmaz, amely magában foglalja a feladat lépéseiben hivatkozott távoli szolgáltatások hívását vagy távoli erőforrásokhoz való hozzáférést. Általában minden ügynök egyetlen szolgáltatáshoz vagy erőforráshoz csomagolja a hívásokat, és alkalmazza a megfelelő hibakezelést és újrapróbálkozási logikát (időtúllépési megkötés függvényében, amelyről a későbbiekben olvashat). Ha az ütemező által futtatott munkafolyamat lépései több szolgáltatást és erőforrást használnak különböző lépéseken keresztül, minden lépés hivatkozhat más ügynökre (ez a minta egy implementálási részlete).

- A **felügyelő** figyeli az ütemező által végrehajtott feladat lépéseinek állapotát. Rendszeres időközönként fut (a gyakorisága rendszerspecifikus), és megvizsgálja az ütemező által fenntartott lépések állapotát. Ha időtúllépést vagy sikertelen lépést észlel, a megfelelő ügynökkel helyreállíttatja a lépést, vagy elvégezteti a megfelelő helyreigazító műveletet (ez egyes lépések állapotának módosításával járhat). Vegye figyelembe, hogy a helyreállítási vagy helyreigazító műveleteket az ütemező és az ügynökök végzik. A felügyelő csupán kérést küld a műveletek elvégzésére.

Az ütemező, az ügynök és a felügyelő logikai összetevők, amelyek fizikai implementálása a használt technológiától függ. Például számos logikai ügynök implementálható egyetlen webszolgáltatás részeként.

Az ütemező az állapottároló elnevezésű tartós adattárban információkat tárol a feladat és az egyes lépések állapotáról. A felügyelő az információk alapján segítséget nyújt annak meghatározásában, meghiúsult-e egy lépés, vagy sem. Az ábra bemutatja az ütemező, az ügynökök, a felügyelő és az állapottároló közötti kapcsolatot.

![1. ábra – Az aktorok a Feladatütemező ügynök felügyeleti mintájában](./_images/scheduler-agent-supervisor-pattern.png)

> [!NOTE]
> Ezen az ábrán a minta egyszerűsített változata látható. Valós alkalmazáskor az ütemező több példánya is futhat párhuzamosan, mindegyik feladatok alegységeként. A rendszer futtathat több példányt minden ügynökből, vagy akár több felügyelőt is. Ebben az esetben a felügyelőknek koordinálniuk kell egymással a munkájukat, hogy ne ugyanazokat a sikertelen lépéseket és feladatokat próbálják helyreállítani. A [vezetőválasztási minta](./leader-election.md) egy lehetséges megoldást jelent erre a problémára.

Amikor az alkalmazás készen áll egy feladat futtatására, kérést küld az ütemezőnek. Az ütemező információkat rögzít az állapottárolóba a feladat és a lépések kezdeti állapotáról (például: meg nem kezdett lépés), majd elkezdi végrehajtani a munkafolyamat által meghatározott műveleteket. Egy-egy lépés megkezdésekor az ütemező frissíti az adattárolóban az adott lépés állapotával kapcsolatos információt (például: a lépés fut).

Ha egy lépés egy távoli szolgáltatásra vagy erőforrásra hivatkozik, az ütemező üzenetet küld a megfelelő ügynöknek. Az üzenet tartalmazza azt az információt, amelyet az ügynöknek továbbítania kell a szolgáltatásnak, vagy amellyel hozzáférhet az erőforráshoz, valamint tartalmazza a művelet befejezési idejét is. Ha az ügynök sikeresen teljesíti a műveletet, választ küld az ütemezőnek. Az ütemező így frissítheti az állapotinformációt az állapottárolóban (például: lépés befejezve), és végrehajthatja a következő lépést. Ez a folyamat folytatódik a teljes feladat befejezéséig.

Egy ügynök bármilyen újrapróbálkozási logikát alkalmazhat, amely a feladatának teljesítéséhez szükséges. Ha azonban az ügynök nem teljesíti a feladatot a befejezési idő lejárta előtt, az ütemező a műveletet sikertelenként fogja kezelni. Ebben az esetben az ügynöknek be kell fejeznie a tevékenységét, és nem szabad visszaküldenie semmit az ütemezőnek (még hibaüzenetet sem), valamint nem szabad helyreállítással próbálkoznia. A korlátozás oka, hogy miután egy lépés túllépte az időkorlátot vagy meghiúsult, az ügynök egy másik példánya ütemezve lehet a sikertelen lépés futtatására (erről a folyamatról a későbbiekben olvashat).

Ha az ügynök nem tudja végrehajtani a feladatát, az ütemező nem kap választ. A minta nem tesz különbséget az időkorlátot túllépő és a valóban sikertelen lépések között.

Ha egy lépés túllépi az időkorlátot vagy meghiúsul, az állapottárolóba bekerül egy rekord arról, hogy a lépés fut, de a befejezési idő lejárt. A felügyelő ehhez hasonló lépéseket keres, és megpróbálja helyreállítani őket. Egy lehetséges stratégia a felügyelő számára a befejezési időérték frissítése a lépés elvégzéséhez szükséges idő meghosszabbításának céljából, majd a lejárt lépést azonosító üzenet küldése az ütemezőnek. Ezután az ütemező megpróbálhatja megismételni a lépést. Ehhez a kialakításhoz azonban szükséges, hogy a feladatok idempotensek legyenek.

Előfordulhat, hogy a felügyelőnek meg kell akadályoznia ugyanazon lépés végrehajtásának újrapróbálását, ha a lépés folyamatosan meghiúsul vagy lejár. Ennek kivitelezéséhez a felügyelő egy ismétlésszámot tarthat fenn az állapottárolóban minden egyes lépéshez, az állapotinformációval együtt. Ha ez a szám túllép egy előre meghatározott küszöbértéket, a felügyelő másik stratégiaként hosszabb ideig várhat, mielőtt értesítené az ütemezőt, hogy az próbálja újra a lépést, annak reményében, hogy a várakozás alatt a probléma megoldódik. A felügyelő másik megoldásként kérést küldhet az ütemezőnek a teljes feladat visszavonására egy [kompenzáló tranzakciós minta](./compensating-transaction.md) alkalmazásával. Ennek a megközelítésnek a sikeressége azon múlik, biztosítják-e az ügynökök és az ütemező a sikeresen befejeződött lépések kompenzáló műveleteinek implementálásához szükséges információkat.

> A felügyelőnek nem célja sem az ütemező és az ügynökök monitorozása, sem hiba esetén az újraindításuk. A rendszer e részét az ezeket az összetevőket futtató infrastruktúrának kell kezelnie. A felügyelőnek szintén nem feladata ismerni a tényleges üzleti műveleteket, amelyeket az ütemező által végzett feladatok futtatnak (beleértve a sikertelen feladatok esetében történő kompenzálás módját). Ez az ütemező által alkalmazott munkafolyamat-logika feladata. A felügyelő egyetlen felelőssége, hogy meghatározza, meghiúsult-e egy lépés, és megszervezze a megismétlését vagy a sikertelen lépést tartalmazó teljes feladat visszavonását.

Ha az ütemező hiba után indult újra, vagy az ütemező által végzett munkafolyamat váratlanul leállt, az ütemező képes meghatározni a hiba beálltakor végzett feladatok állapotát, és kész folytatni őket attól a ponttól. A folyamat implementálásának részletei nagy valószínűséggel rendszerspecifikusak. Ha a feladat nem helyreállítható, szükség lehet a feladat által már elvégzett munka visszavonására. Ehhez szükséges lehet egy [kompenzáló tranzakció](./compensating-transaction.md) implementálása.

Ennek a mintának az egyik legfontosabb előnye a váratlan ideiglenes vagy nem helyreállítható hibák esetén biztosított rugalmasság. A rendszer tervezhető úgy, hogy kijavítsa önmagát. Például ha hiba lép fel egy ügynök vagy az ütemező működésében, elindíthat egy új példányt, és a felügyelő segítségével folytathatja a feladatokat. Ha a felügyelő működésében lép fel hiba, elindíthat egy másik példányt, amellyel onnan folytathatja a munkát, ahol a hiba történt. Ha a felügyelő rendszeres időközönként ütemezve fut, automatikusan indítható új példány egy előre meghatározott időszak után. Az állapottároló replikálható a még nagyobb fokú rugalmasság elérése érdekében.

## <a name="issues-and-considerations"></a>Problémák és megfontolandó szempontok

A minta megvalósítása során az alábbi pontokat vegye figyelembe:

- Ez a minta nehezebben implementálható, és szükséges hozzá a rendszer minden lehetséges hibaállapotának alapos tesztelése.

- Az ütemező által implementált helyreállítási/újrapróbálkozási logika összetett, és függ az állapottárolóban tárolt állapotinformációtól. Előfordulhat, hogy rögzíteni kell a kompenzáló tranzakció implementálásához szükséges információt egy tartós adattárban.

- Fontos, milyen gyakran fut a felügyelő. Elég gyakran kell futnia annak megakadályozásához, hogy a hibás lépések hosszú ideig blokkoljanak egy alkalmazást, de nem olyan gyakran, hogy többletterheléssé váljon.

- Az ügynökök által végzett lépések többször is futtathatók. Az ezeket a lépéseket implementáló logikának idempotensnek kell lennie.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes ezt a mintát használni?

Használja ezt a mintát, ha egy elosztott környezetben, például felhőben futó folyamatnak rugalmasnak kell lennie a kommunikációs és/vagy működési hibákkal szemben.

Nem érdemes ezt a mintát használni olyan feladatok esetében, amelyek nem hívnak meg távoli szolgáltatásokat vagy nem férnek hozzá távoli erőforrásokhoz.

## <a name="example"></a>Példa

Egy elektronikus kereskedelmi rendszert implementáló webalkalmazás lett üzembe helyezve a Microsoft Azure-ban. Az alkalmazás segítségével a felhasználók böngészhetnek az elérhető termékek között, és rendeléseket adhatnak le. A felhasználói felület webes szerepkörként fut, az alkalmazás rendelésfeldolgozó elemei pedig feldolgozói szerepkörökként vannak implementálva. A rendelésfeldolgozó logika egy része magában foglalja a távoli szolgáltatásokhoz való hozzáférést. A rendszer ezen aspektusában gyakrabban előfordulhatnak átmeneti vagy hosszan tartó hibák. Ezért a tervezők a Feladatütemező ügynök felügyeleti mintáját használták a rendszer rendelésfeldolgozó elemeinek implementálásához.

Amikor az ügyfél lead egy rendelést, az alkalmazás összeállít egy üzenetet, amelyben leírja a rendelést, majd egy üzenetsorba küldi. Az üzenetet egy feldolgozói szerepkörben futó külön beküldési folyamat olvassa be, beszúrja a rendelés részleteit a rendelési adatbázisba, és rögzíti a rendelési folyamatot az állapottárolóban. Vegye figyelembe, hogy a rendelési adatbázisba és az állapottárolóba történő beszúrások ugyanazon művelet részeként történnek. A beküldési folyamat célja a két beszúrás együttes befejezésének biztosítása.

A beküldési folyamat által a rendeléshez létrehozott állapotinformáció az alábbiakat tartalmazza:

- **OrderID**. A rendelési adatbázisban található rendelés azonosítója.

- **LockedBy**. A rendelést kezelő feldolgozói szerepkör példányazonosítója. Előfordulhat, hogy a feldolgozói szerepkör több aktuális példánya futtatja az ütemezőt, de az egyes rendeléseket csak egy példánynak kell kezelnie.

- **CompleteBy**. Az az időpont, ameddig a rendelés feldolgozásának meg kell történnie.

- **ProcessState**. A rendelést kezelő feladat jelenlegi állapota. A lehetséges állapotok a következők:

  - **Pending**. A rendelés létrejött, de a feldolgozása még nem kezdődött el.
  - **Processing**. A rendelés feldolgozása folyamatban van.
  - **Processed**. A rendelés feldolgozása sikeresen megtörtént.
  - **Error**. A rendelés feldolgozása nem sikerült.

- **FailureCount**. A rendelés feldolgozására tett kísérletek száma.

Ebben az állapotinformációban az `OrderID` mező az új rendelés rendelésazonosítójából lett kimásolva. A `LockedBy` és `CompleteBy` mező értéke `null`, a `ProcessState` mező értéke `Pending`, a `FailureCount` mező értéke pedig 0.

> [!NOTE]
> Ebben a példában a rendeléskezelő logika viszonylag egyszerű, és csak egyetlen lépése van, amely távoli szolgáltatást hív meg. Egy összetettebb, többlépéses forgatókönyv szerint az a folyamat akkor vélhetően több lépést tartalmaz, és így az állapottárolóban jön &mdash; mindegyike egy-egy adott lépés állapotát leíró.

Az ütemező szintén a feldolgozói szerepkör részeként fut, és implementálja a rendelést kezelő üzleti logikát. Az ütemező egy új rendeléseket kereső példánya olyan rekordokat vizsgál az állapottárolóban, amelyekben a `LockedBy` mező értéke null, a `ProcessState` mezőé pedig „Pending”. Amikor az ütemező új rendelést talál, azonnal kitölti a `LockedBy` mezőt a saját példányazonosítójával, a `CompleteBy` mezőt egy megfelelő időpontra állítja be, a `ProcessState` mezőt pedig „Processing” értékre állítja. A kód kizárólagos és atomi tervezése biztosítja, hogy az ütemező két párhuzamosan futó példánya ne kezelhesse egyszerre ugyanazt a rendelést.

Az ütemező ezután futtatja az üzleti munkafolyamatot a rendelés aszinkron módon történő feldolgozásához, és átadja a rendelésnek az állapottárolóban található `OrderID` mezőben lévő értéket. A rendelést kezelő munkafolyamat lekéri a rendelés részleteit a rendelési adatbázisból, és elvégzi a feladatát. Ha a rendelésfeldolgozási munkafolyamatban egy lépésnek távoli szolgáltatást kell meghívnia, erre ügynököt használ. A munkafolyamat-lépés egy kérés/válasz típusú csatornaként működő Azure Service Bus-üzenetsorpárt használ az ügynökkel folytatott kommunikációhoz. Az ábra a megoldás magas szintű nézetét mutatja be.

![2. ábra – A Feladatütemező ügynök felügyeleti mintájának használata rendelések kezeléséhez egy Azure-megoldásban](./_images/scheduler-agent-supervisor-solution.png)

Az ügynöknek egy munkafolyamat-lépés által küldött üzenet leírja a rendelést, és tartalmazza a befejezési időt. Ha az ügynök választ kap a távoli szolgáltatástól a befejezési idő lejárta előtt, válaszüzenetet küld abba a Service Bus-üzenetsorba, amelyet a munkafolyamat figyel. Amikor a munkafolyamat-lépés megkapja az érvényes válaszüzenetet, befejezi a feldolgozást, majd az ütemező „Processed” állapotra állítja be a rendelési állapot „ProcessState” mezőjének értékét. Ezen a ponton a rendelés feldolgozása sikeresen befejeződött.

Ha a befejezési idő lejár, mielőtt az ügynök választ kap a távoli szolgáltatástól, az ügynök egyszerűen abbahagyja a feldolgozást, és leállítja a rendelés kezelését. Ha a rendelést kezelő munkafolyamat túllépi a befejezési időt, hasonlóképpen leáll. A rendelés állapota az állapottárolóban mindkét esetben „Processing” marad, de a befejezési idő alapján látható, hogy a rendelés feldolgozására szánt idő letelt, és a folyamat sikertelennek lett nyilvánítva. Vegye figyelembe, hogy ha a távoli szolgáltatáshoz hozzáférő ügynök vagy a rendelést kezelő munkafolyamat (vagy mindkettő) váratlanul leáll, az állapottárolóban található információ szintén „Processing” értékre állítva marad, majd idővel lejárt befejezési idővel rendelkezik.

Ha az ügynök helyreállíthatatlan, nem átmeneti hibát észlel a távoli szolgáltatással való kapcsolatfelvételre tett kísérlet közben, hibaüzenetet küldhet vissza a munkafolyamatnak. Az ütemező a rendelési állapotot „Error” értékre állíthatja, és elindíthat egy eseményt, amely riaszt egy operátort. Az operátor ekkor megpróbálja manuálisan elhárítani a hibát, és újraküldeni a sikertelen feldolgozási lépést.

A felügyelő rendszeres időközönként átvizsgálja az állapottárolót lejárt befejezési időértékű rendeléseket keresve. Ha talál ilyen rekordot, növeli a `FailureCount` mező értékét. Ha a hibaszám értéke egy meghatározott küszöbérték alatt van, a felügyelő visszaállítja a `LockedBy` mezőt nullértékre, új lejárati időre frissíti a `CompleteBy` mezőt, és beállítja a `ProcessState` mező állapotát „Pending” értékre. Az ütemező egy példánya átveheti ezt a rendelést, és az eddigiekhez hasonlóan elvégezheti a feldolgozását. Ha a hibaszám értéke meghaladja a meghatározott küszöbértéket, a rendszer a hiba okát nem átmenetinek tekinti. A felügyelő a rendelési állapotot „Error” értékre állítja, és elindít egy eseményt, amely riaszt egy operátort.

> Ebben a példában a felügyelő egy külön feldolgozói szerepkörben van implementálva. Számos stratégiát használhat a felügyelői feladat futtatására, beleértve az Azure Scheduler-szolgáltatás használatát (nem összetévesztendő az ebben a mintában használt ütemező összetevővel). Az Azure Scheduler-szolgáltatással kapcsolatos további információért látogasson el [a Scheduler](https://azure.microsoft.com/services/scheduler/) oldalára.

Bár ebben a példában nem mutattuk be, előfordulhat, hogy az ütemezőnek folyamatosan tájékoztatnia kell a rendelést beküldő alkalmazást a rendelés folyamatáról és állapotáról. Az alkalmazás és az ütemező az egymás közötti függőségek elkerülése érdekében el van különítve egymástól. Az alkalmazás nem rendelkezik információval arról, hogy az ütemező melyik példánya kezeli a rendelést, ahogyan az ütemező sem tud arról, pontosan melyik alkalmazáspéldány adta le a rendelést.

Ahhoz, hogy a rendelés állapota jelenthető legyen, az alkalmazás használhatja a saját személyes válaszvárólistáját. A válaszvárólista részleteit tartalmazza a kérés, amelyet a beküldési folyamat kap meg, amely hozzáadja ezt az információt az állapottárhoz. Ezután az ütemező üzeneteket küld ebbe az üzenetsorba a rendelés állapotáról (kérés megérkezett, rendelés befejezve, rendelés sikertelen stb.). Az üzeneteknek tartalmazniuk kell a rendelési azonosítót, hogy összekapcsolhatók legyenek az alkalmazás által küldött eredeti kéréssel.

## <a name="related-patterns-and-guidance"></a>Kapcsolódó minták és útmutatók

Az alábbi minták és útmutatók szintén hasznosak lehetnek a minta megvalósításakor:

- [Újrapróbálkozási minta](./retry.md). Ügynökök használhatják ezt a mintát egy olyan művelet átlátható módon történő megismétléséhez, amely korábban hibás állapotú távoli szolgáltatáshoz vagy erőforráshoz fér hozzá. Akkor használja, ha a hiba oka valószínűsíthetően átmeneti és elhárítható.
- [Áramkör-megszakítási minta](./circuit-breaker.md). Ügynökök használhatják ezt a mintát olyan hibák kezelésére, amelyek elhárítása távoli szolgáltatáshoz vagy erőforráshoz való csatlakozáskor változó mennyiségű időt vesz igénybe.
- [Kompenzáló tranzakció mintája](./compensating-transaction.md). Ha az ütemező által végzett munkafolyamatot nem lehet sikeresen befejezni, szükséges lehet az addig elvégzett munka visszavonása. A Kompenzáló tranzakció mintája leírja, hogyan valósítható ez meg olyan műveletek esetében, amelyek a végleges konzisztenciájú modellt követik. Az ilyen típusú műveleteket általában egy összetett üzleti folyamatokat és munkafolyamatokat végző ütemező implementálja.
- [Az aszinkron üzenetkezelés ismertetése](https://msdn.microsoft.com/library/dn589781.aspx). A Feladatütemező ügynök felügyeleti mintájának összetevői általában egymástól elválasztva futnak, és aszinkron módon kommunikálnak. Itt néhány, aszinkron módon történő kommunikáció implementálására használt, üzenetsorokon alapuló megközelítés leírását olvashatja.
- [Vezetőválasztási minta](./leader-election.md). Szükséges lehet koordinálni egy felügyelő több példányának műveleteit annak megakadályozása érdekében, hogy a példányok ugyanazt a sikertelen folyamatot próbálják meg helyreállítani. A Vezetőválasztási minta ismerteti, hogyan teheti meg ezt.
- [Felhőarchitektúra: A Feladatütemező ügynök felügyeleti mintája](https://blogs.msdn.microsoft.com/clemensv/2010/09/27/cloud-architecture-the-scheduler-agent-supervisor-pattern/) Clemens Vasters blogján
- [Folyamatkezelő minta](https://www.enterpriseintegrationpatterns.com/patterns/messaging/ProcessManager.html)
- [6. hivatkozás: Egy Saga a Sagákról](https://msdn.microsoft.com/library/jj591569.aspx). Egy példa arra, hogyan használja a CQRS mintája a folyamatkezelőt (a CQRS-út útmutatásának része).
- [Microsoft Azure Scheduler](https://azure.microsoft.com/services/scheduler/)
