---
title: Újrapróbálkozásokra vonatkozó általános útmutató
titleSuffix: Best practices for cloud applications
description: Útmutatás az újrapróbálkozáshoz az átmeneti hibák kezelésénél.
author: dragon119
ms.date: 07/13/2016
ms.custom: seodec18
ms.openlocfilehash: fe07364e1a6846f9b7b47b2b79ce8031122edbbd
ms.sourcegitcommit: 4ba3304eebaa8c493c3e5307bdd9d723cd90b655
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/12/2018
ms.locfileid: "53307112"
---
# <a name="transient-fault-handling"></a>Átmeneti hibák kezelése

A távoli szolgáltatásokkal és erőforrásokkal kommunikáló alkalmazásoknak érzékenynek kell lenniük az átmeneti hibákra. Ez különösen igaz a felhőben futó alkalmazások esetében, ahol a környezet és az internetes kapcsolat természetéből adódóan gyakrabban lehet találkozni ezzel a hibatípussal. Az átmeneti hibák közé tartozik az összetevők és szolgáltatások hálózati kapcsolatának pillanatnyi megszakadása, a szolgáltatások átmeneti elérhetetlensége, valamint a foglalt szolgáltatás miatt jelentkező időtúllépés. Ezek a hibák gyakran maguktól megoldódnak, és ha később megismételik a műveletet, az nagy eséllyel sikeres lesz.

Ez a dokumentum az átmeneti hibák kezeléséhez nyújt általános útmutatást. További információt az átmeneti hibák a Microsoft Azure-szolgáltatások használatakor történő kezeléséről az [Azure-szolgáltatásokra vonatkozó újrapróbálkozási irányelvekben](./retry-service-specific.md) talál.

<!-- markdownlint-disable MD026 -->

## <a name="why-do-transient-faults-occur-in-the-cloud"></a>Miért fordulnak elő átmeneti hibák a felhőben?

<!-- markdownlint-enable MD026 -->

Az átmeneti hibák bármilyen környezet, platform vagy operációs rendszer esetén, illetve bármilyen típusú alkalmazásban előfordulhatnak. A helyszíni infrastruktúrákon futó megoldásoknál az alkalmazás, valamint az összetevőinek teljesítményét és rendelkezésre állását általában költséges, és gyakran kihasználatlan hardverredundancia révén tartják fenn, az összetevők és az erőforrások pedig egymás közelében helyezkednek el. Bár ennek köszönhetően ritkábban fordulnak elő hibák, ez nem zárja ki az átmeneti hibákat – sőt, az olyan előre nem látható események miatti kimaradásokat sem, mint a külső tápegységgel vagy a hálózattal kapcsolatos problémák, vagy egyéb katasztrófák.

A felhőalapú üzemeltetés, a magánfelhő-alapú rendszereket is beleértve, magasabb általános rendelkezésre állást tesz lehetővé azáltal, hogy nagyon sok hagyományos számítási csomóponton használ megosztott erőforrásokat, redundanciát, automatikus feladatátvételt és dinamikus erőforrás-kiosztást. Az ilyen típusú környezeteknél azonban gyakrabban fordulhatnak elő átmeneti hibák. Ennek több oka is van:

- A felhőalapú környezetekben sok a megosztott erőforrás, és ezek a védelmük érdekében szabályozott hozzáféréssel rendelkeznek. Egyes szolgáltatások el fogják utasítani a kapcsolatokat, ha a terhelés egy bizonyos szintre emelkedik, vagy ha elérik a maximális átviteli sebességet, hogy lehetővé tegyék a meglévő kérések feldolgozását, és fenntartsák a szolgáltatás teljesítményét az összes felhasználónál. A szabályozás segít fenntartani a szolgáltatás minőségét a szomszédoknál és a többi olyan bérlőnél, amelyek a megosztott erőforrást használják.

- A felhőalapú környezetek létrehozásához hatalmas mennyiségű hagyományos hardveregységre van szükség. Ezek az egységek biztosítják a teljesítményt a terhelés több számítási egység és infrastruktúra-összetevő közötti dinamikus elosztásával, valamint a megbízhatóságot a hibás egységek automatikus újraindításával vagy lecserélésével. Ennek a dinamikus jellegnek köszönhetően időnként előfordulhatnak átmeneti hibák és ideiglenes csatlakozási hibák.

- Az alkalmazás és az általa használt erőforrások és szolgáltatások között gyakran több hardverösszetevő is található, többek között olyan hálózati infrastruktúrák, mint az útválasztók és a terheléselosztók. Ez a további infrastruktúra alkalmanként további kapcsolódási késést és átmeneti kapcsolati hibákat okozhat.

- Az ügyfél és a kiszolgáló közötti hálózati körülmények változékonyak lehetnek, különösen akkor, amikor az interneten keresztül folyik a kommunikáció. A nagy forgalmi terhelés még a helyszíni helyeken is lelassíthatja a kommunikációt, és időszakos csatlakozási hibákat okozhat.

## <a name="challenges"></a>Problémák

Az átmeneti hibák nagymértékben befolyásolhatják az alkalmazás észlelhető rendelkezésre állását, még akkor is, ha az alkalmazást alapos tesztelésnek vetették alá, az összes előrelátható körülmény figyelembevételével. Ahhoz, hogy a felhőben üzemeltetett alkalmazások megbízhatóan működjenek, meg kell tudniuk felelni a következő kihívásoknak:

- Az alkalmazásnak észlelnie kell a hibákat, amikor azok bekövetkeznek, és meg kell tudnia határozni, hogy ezek a hibák várhatóan átmenetiek, annál tartósabbak, vagy végzetesek. A különböző erőforrások hiba esetén nagy eséllyel eltérő válaszokat adnak vissza, és ezek a válaszok a művelet kontextusától függően is eltérhetnek. Például az erőforrások más választ adhatnak a tárolóból való olvasásakor jelentkező hibára, mint a tárolóba való íráskor jelentkező hibára. Sok erőforrás és szolgáltatás részletesen dokumentált átmeneti meghibásodási szerződésekkel rendelkezik. Azonban ha nem állnak rendelkezésre ilyen adatok, nehéz lehet kideríteni a hiba jellegét és tartósságát.

- Az alkalmazásnak képesnek kell lennie újra végrehajtani a műveletet, ha ezzel meg tudja határozni a hiba tartósságát, továbbá nyomon kell követnie, hogy a rendszer hányszor kísérelte meg újból a műveletet.

- Az alkalmazásnak megfelelő stratégiát kell alkalmaznia az újrapróbálkozásokhoz. Ez a stratégia határozza meg az újrapróbálkozások számát, a kísérletek közötti késleltetést, valamint a sikertelen próbálkozások után elvégzendő műveleteket. A próbálkozások megfelelő számát és a köztük lévő időt gyakran nehéz meghatározni, sőt, ezek az erőforrás típusától, valamint az erőforrás és az alkalmazás aktuális működési feltételeitől függően változnak.

## <a name="general-guidelines"></a>Általános irányelvek

Az alábbi irányelvek segítenek megtervezni egy megfelelő átmeneti hibakezelési mechanizmust az alkalmazásai számára:

- **Állapítsa meg, hogy van-e beépített újrapróbálkozási mechanizmus:**

  - Számos szolgáltatás biztosít átmeneti hibakezelési mechanizmust tartalmazó SDK-t vagy ügyfélkódtárat. A mechanizmus által használt újrapróbálkozási szabályzat általában a célszolgáltatás természetéhez és követelményeihez igazodik. Másik lehetőségként a szolgáltatások REST-felületei is visszaadhatnak olyan információkat, amelyek segítenek meghatározni, hogy érdemes-e újrapróbálkozni, és mennyi időt kell várni a következő újrapróbálkozási kísérletig.

  - Ha elérhető, használja a beépített újrapróbálkozási mechanizmust, hacsak nincsenek konkrét és jól érthető követelmények, amelyek más újrapróbálkozási viselkedést igényelnek.

- **Döntse el, hogy a művelet alkalmas-e az újrapróbálkozásra**:

  - Csak azokat a műveleteket érdemes újrapróbálni, ahol a hibák átmenetiek (ezt általában a hiba természetéből lehet megállapítani), és legalább némi esély van arra, hogy a művelet megismétlése sikerrel fog járni. Nincs értelme újrapróbálkozni azoknál a műveleteknél, amelyek érvénytelen műveletet jeleznek, például egy nem létező elemhez tartozó adatbázis-frissítést, vagy kérések küldését egy olyan szolgáltatásnak vagy erőforrásnak, amelyben végzetes hiba történt

  - Általában csak akkor érdemes implementálni az újrapróbálkozásokat, ha meg lehet határozni a próbálkozások teljes hatását, valamint jól értelmezhetők és érvényesíthetők a feltételek. Ha ez nem így van, hagyja, hogy a hívó kód implementálja az újrapróbálkozásokat. Ne feledje, hogy a hatáskörén kívül eső erőforrások és szolgáltatások által visszaadott hibák idővel változhatnak. Előfordulhat, hogy újra kell gondolnia az átmeneti hibák észlelési logikáját.

  - A szolgáltatások vagy összetevők létrehozásakor vegye fontolóra a hibakódok és -üzenetek alkalmazását, amelyek segítenek az ügyfeleknek eldönteni, hogy megpróbálják-e újra végrehajtani a sikertelen műveleteket. Jelezze, ha az ügyfélnek érdemes újrapróbálkoznia a művelettel (például az **isTransient** érték visszaadásával), és tegyen javaslatot a következő próbálkozás előtti késleltetés idejére. Ha webszolgáltatást hoz létre, fontolja meg a szolgáltatási szerződésekben meghatározott egyéni hibaüzenetek visszaadását. Az általános ügyfelek lehet, hogy nem fogják tudni beolvasni ezeket, viszont jól fognak jönni majd az egyéni ügyfelek létrehozásakor.

- **Határozzon meg egy megfelelő újrapróbálkozási számot és időközt:**

  - Rendkívül fontos, hogy a használat típusához optimalizálja az újrapróbálkozások számát és időközét. Ha nem próbálkozik elégszer, az alkalmazás nem fogja tudni végrehajtani a műveletet, és hiba léphet fel. Ha túl sokszor próbálkozik újra, vagy túl kevés ideig vár a próbálkozások között, előfordulhat, hogy az alkalmazás hosszú ideig tartja az erőforrásokat (például a szálakat, a kapcsolatokat és a memóriát), amely kedvezőtlen hatással lesz az alkalmazás állapotára.

  - Az időközök megfelelő értéke és az újrapróbálkozási kísérletek száma a megkísérelt művelet típusától függ. Például ha a művelet egy felhasználói beavatkozás része, ajánlott rövid időközöket beállítani, és csak néhányszor újrapróbálkozni, hogy a felhasználóknak ne kelljen várniuk a válaszra (ami nyitott kapcsolatokat tart fenn, és csökkentheti a rendelkezésre állást a többi felhasználónál). Ha a művelet egy hosszan tartó vagy kritikus fontosságú munkafolyamat része, és a folyamat megszakítása és újraindítása költséges vagy időigényes, célszerű hosszabb ideig várni a kísérletek között, és többször újrapróbálkozni.

  - Az újrapróbálkozások közötti megfelelő időközök meghatározása a sikeres stratégiák megtervezésének legnehezebb része. A stratégiák általában az alábbi újrapróbálkozási időköztípusokat használják:

    - **Exponenciális visszatartás**. Az alkalmazás rövid ideig várakozik az első újrapróbálkozás előtt, majd exponenciálisan növeli a próbálkozások közötti időt. Például megpróbálhatja újra végrehajtani a műveletet 3 másodperc, 12 másodperc, 30 másodperc múlva és így tovább.

    - **Növekményes időközök**. Az alkalmazás rövid ideig várakozik az első újrapróbálkozás előtt, majd fokozatosan növeli a próbálkozások közötti időt. Például megpróbálhatja újra végrehajtani a műveletet 3 másodperc, 7 másodperc, 13 másodperc múlva és így tovább.

    - **Rendszeres időközök**. Az alkalmazás ugyanannyi ideig várakozik a próbálkozások között. Például megpróbálhatja újra végrehajtani a műveletet 3 másodpercenként.

    - **Azonnali újrapróbálkozás**. Az átmeneti hibák néha nagyon rövid ideig tartanak, például ha olyan esemény váltja ki őket, mint a hálózati csomagok ütközése, vagy egy megnövekedett terhelés az egyik hardverösszetevőben. Ebben az esetben célszerű azonnal megismételni a műveletet, mivel az sikerrel járhat, ha a hiba még az idő alatt megszűnik, hogy az alkalmazás összeállítja és elküldi a következő kérést. Soha ne hajtson végre egynél több azonnali újrapróbálkozási kísérletet, és ha az meghiúsul, váltson át alternatív stratégiákra, például exponenciális visszatartásra vagy helyettesítő műveletekre.

    - **Véletlenszerűsítés**. A fent említett újrapróbálkozási stratégiák bármelyike tartalmazhat véletlenszerűsítést, amely megakadályozza, hogy az ügyfél több példánya egyszerre küldjön újrapróbálkozási kísérleteket. Például egy példány megpróbálhatja újra végrehajtani a műveletet 3 másodperc, 11 másodperc, 28 másodperc stb. múlva, egy másik példány pedig 4 másodperc, 12 másodperc, 26 másodperc múlva és így tovább. A véletlenszerűsítés hasznos módszer, amelyet más stratégiákkal együtt is alkalmazni lehet.

  - Általános irányelvként a háttérműveletekhez használja az exponenciális visszatartást, az interaktív műveletekhez pedig az azonnali vagy a rendszeres időközű újrapróbálkozási stratégiát. Mindkét esetben úgy kell kiválasztania a késleltetést és az újrapróbálkozások számát, hogy az újrapróbálkozási kísérletek maximális késleltetése megfeleljen a végpontok közötti késés szükséges követelményeinek.

  - Vegye figyelembe az összes tényezőt, amely hozzájárul az újra megkísérelt művelet teljes maximális időtúllépéséhez. E tényezők közé tartozik a sikertelen kapcsolat válaszadási ideje (általában az ügyfél egyik időtúllépési értéke állítja be), valamint az újrapróbálkozási kísérletek közötti késleltetés és az újrapróbálkozások maximális száma. Ezek együtt nagyon hosszú teljes működési időt eredményezhetnek, főleg az exponenciális késleltetési stratégia használatánál, ahol a meghiúsult kísérletek után gyorsan növekszik az újrapróbálkozások közötti idő. Ha egy folyamatnak meg kell felelnie egy adott szolgáltatói szerződés (SLA), a teljes működési időnek, beleértve az összes időtúllépéseket és késéseket belül, amely definiálni kell az SLA-ban.

  - A túl rövid időközökkel vagy túl sok újrapróbálkozással rendelkező agresszív újrapróbálkozási stratégiák kedvezőtlen hatással lehetnek a célerőforrásra vagy -szolgáltatásra. Ez megakadályozhatja, hogy az erőforrás vagy szolgáltatás helyreálljon a túlterhelt állapotból, és továbbra is blokkolni fogja vagy el fogja utasítani a kérelmeket. Ez egy ördögi kört eredményez, ahol az erőforrás vagy szolgáltatás egyre több kérést kap, és ennek köszönhetően tovább romlik a helyreállási képessége.

  - Az újrapróbálkozási időközök megadásakor vegye figyelembe a műveletek időtúllépését, hogy ne indítsa el azonnal a következő kísérletet (például abban az esetben, ha hasonló az időkorlát és az újrapróbálkozási időköz). Azt is vegye figyelembe, ha a teljes lehetséges időtartamot (az időtúllépések és az újrapróbálkozási időközök együttesét) a megadott teljes idő alatt kell tartani. A szokatlanul rövid vagy hosszú időtúllépéssel rendelkező műveletek befolyásolhatják a várakozási időt és a művelet újbóli megkísérlésének gyakoriságát.

  - Az újrapróbálkozások időközének és számának optimalizálásához használja a kivétel típusát és a benne szereplő adatokat, vagy a szolgáltatástól visszakapott hibakódokat és -üzeneteket. Néhány kivétel vagy hibakód (mint az 503-as „A szolgáltatás nem érhető el” HTTP-kód, egy Retry-After fejléccel a válaszban) például jelezheti, hogy mennyi ideig tarthat a hiba, vagy hogy a szolgáltatás leállt, és nem fog reagálni a további kísérletekre.

- **Kerülje el a kizárási mintákat**:

  - A legtöbb esetben érdemes elkerülni azokat az implementálásokat, amelyek duplikált rétegű újrapróbálkozási kódot tartalmaznak. Kerülje továbbá azokat a kialakításokat, amelyek lépcsőzetes újrapróbálkozási mechanizmusokat tartalmaznak, illetve azokat, amelyek a kéréshierarchiával rendelkező műveletek összes szakaszában implementálják az újrapróbálkozást, hacsak a konkrét követelmények ezt nem igénylik. Ezekben a kivételes esetekben használjon olyan szabályzatokat, amelyek megakadályozzák a túl sok újrapróbálkozást és a túl nagy késleltetési időközöket, és győződjön meg róla, hogy tisztában van a következményekkel. Például ha az egyik összetevő kérést küld a másiknak, amely ezután hozzáfér a célszolgáltatáshoz, és Ön mindkét hívásnál implementál három újrapróbálkozást, akkor a rendszer összesen kilenc újrapróbálkozási kísérletet hajt végre a szolgáltatás irányába. Sok szolgáltatás és erőforrás használ beépített újrapróbálkozási mechanizmust, ezért ha magasabb szinten kell újrapróbálkozásokat implementálnia, nézzen utána, hogyan tilthatja le vagy módosíthatja ezeket.

  - Soha ne implementáljon végtelen újrapróbálkozási mechanizmust. Ez nagy eséllyel megakadályozza, hogy az erőforrás vagy szolgáltatás helyreálljon a túlterhelési helyzetekből, és a hatására tovább fog tartani a szabályozás és a kapcsolatok elutasítása. Ha azt szeretné, hogy a szolgáltatás helyre tudjon állni, használjon véges számú újrapróbálkozást, vagy implementáljon egy mintát, például az [áramkör-megszakítót](../patterns/circuit-breaker.md).

  - Soha ne végezzen azonnali újrapróbálkozást egynél több alkalommal.

  - Kerülje a rendszeres újrapróbálkozási időköz használatát, különösen akkor, ha az Azure-beli szolgáltatások és erőforrások elérésekor sok az újrapróbálkozási kísérlet. Ebben a forgatókönyvben az optimális megközelítés az áramkör-megszakító képességgel kombinált exponenciális visszatartási stratégia.

  - Akadályozza meg, hogy egy adott ügyfél több példánya, vagy különböző ügyfelek több példánya egyszerre küldjön újrapróbálkozásokat. Ha nagy az esély erre, vezessen be véletlenszerűsítést az újrapróbálkozási időközökbe.

- **Tesztelje az újrapróbálkozási stratégiát és az implementálást:**

  - Ügyeljen arra, hogy teljes körűen tesztelje az újrapróbálkozási stratégia implementálását, a lehető legkülönfélébb körülmények mellett, különösen akkor, ha az alkalmazás, és az általa használt célerőforrások vagy -szolgáltatások is nagy terhelés alatt állnak. A következőképpen ellenőrizheti a viselkedést a tesztelés közben:

    - Szúrjon be átmeneti és nem átmeneti hibákat a szolgáltatásba. Például küldjön érvénytelen kéréseket, vagy adjon hozzá egy kódot, amely észleli a tesztelési kéréseket, és különböző típusú hibákat ad vissza válaszként. TestApi használatával egy példa: [tesztelést hibabeszúrással](https://msdn.microsoft.com/magazine/ff898404.aspx) és [TestApi – 5. rész – Bevezetés: Felügyelt kód tartalék injektálási API-k](https://blogs.msdn.microsoft.com/ivo_manolov/2009/11/25/introduction-to-testapi-part-5-managed-code-fault-injection-apis/).

    - Hozza létre az erőforrás vagy szolgáltatás utánzatát, amely ugyanazt a hibatartományt adja vissza, mint a valódi szolgáltatás. Győződjön meg róla, hogy az összes hibatípust lefedi, amelyet az újrapróbálkozási stratégia észlelni tud.

    - Kényszerítse az átmeneti hibák előfordulását a szolgáltatás ideiglenes letiltásával vagy túlterhelésével, de csak abban az esetben, ha az egy egyéni szolgáltatás, amelyet Ön hozott létre és helyezett üzembe (természetesen ne próbálja meg túlterhelni az Azure-ban megosztott erőforrásokat vagy szolgáltatásokat).

    - A HTTP-alapú API-k esetén fontolja meg a FiddlerCore kódtár használatát az automatikus tesztekben a HTTP-kérések eredményének megváltoztatásához. Ezt további adatváltási idők hozzáadásával, vagy a válasz (például a HTTP-állapotkód, fejlécek, törzs vagy egyéb tényezők) módosításával érheti el. Ez lehetővé teszi a hibafeltételek egy részének determinisztikus tesztelését, legyen szó akár átmeneti hibákról, akár más hibatípusokról. További információért lásd a [FiddlerCore](https://www.telerik.com/fiddler/fiddlercore) ismertetését. A kódtár, de különösen a **HttpMangler** osztály használatát bemutató példákért tekintse meg [az Azure Storage SDK forráskódját](https://github.com/Azure/azure-storage-net/tree/master/Test).

    - Végezzen magas terhelési tényezős és párhuzamos teszteket annak ellenőrzéséhez, hogy az újrapróbálkozási mechanizmus és a stratégia megfelelően működik-e ilyen feltételek mellett, nincsenek-e kedvezőtlen hatással az ügyfél működésére, valamint nem okoznak-e keresztkontaminációt a kérések között.

- **Kezelje az újrapróbálkozási szabályzat-konfigurációkat:**

  - Az *újrapróbálkozási szabályzat* az újrapróbálkozási stratégia elemeiből tevődik össze. Ez határozza meg az észlelési mechanizmust, amely megadja a hibák feltételezett típusát, a használandó időközt (például rendszeres, exponenciális visszatartás és véletlenszerűsítés), a tényleges időközi érték(ek)et és az újrapróbálkozások számát.

  - Az újrapróbálkozásokat még a legegyszerűbb alkalmazásokban is sok helyen implementálni kell, az összetettebb alkalmazásokban pedig minden rétegben. A szabályzatok elemeinek több helyen való fix kódolása helyett fontolja meg egy központi hely használatát, ahol az összes szabályzatot tárolhatja. Például tárolja az időközt és az újrapróbálkozások számát az alkalmazás konfigurációs fájljaiban, olvassa be őket a futtatáskor, és hozza létre az újrapróbálkozási szabályzatokat a szoftverből. Ez megkönnyíti a beállítások kezelését, valamint az értékek módosítását és finomhangolását, hogy azok megfeleljenek a változó követelményeknek és forgatókönyveknek. A rendszert azonban úgy tervezze meg, hogy tárolja az értékeket ahelyett, hogy minden alkalommal újra beolvasná a konfigurációs fájlt, és biztosítsa a megfelelő alapértelmezett értékek használatát, ha az értékeket nem lehet lekérni a konfigurációból.

  - Az Azure Cloud Services-alkalmazásokban érdemes az újrapróbálkozási szabályzatok futásidőben való létrehozásához használt értékeket a szolgáltatáskonfigurációs fájlban tárolni, hogy az alkalmazás újraindítása nélkül meg lehessen változtatni őket.

  - Használja ki az Ön által használt ügyfél API-kban elérhető beépített vagy alapértelmezett újrapróbálkozási stratégiák előnyeit, de csak akkor, ha azok megfelelnek a forgatókönyvének. Ezek a stratégiák általában általános célúak. Egyes forgatókönyveknél ezek is elegendőek lehetnek, viszont más forgatókönyvek esetében előfordulhat, hogy nem tartalmaznak elég beállítást ahhoz, hogy megfeleljenek a követelményeknek. A legmegfelelőbb értékek meghatározásához tesztelés útján ki kell tapasztalnia, hogy milyen hatással vannak a beállítások az alkalmazásra.

- **Naplózza és kövesse nyomon az átmeneti és nem átmeneti hibákat:**

  - Az újrapróbálkozási stratégiába foglaljon bele kivételkezelést, és olyan rendszerállapotokat, amelyek naplózzák az újrapróbálkozási kísérleteket. Míg az esetenként előforduló átmeneti hibákra és újrapróbálkozásokra számítani lehet, és nem utalnak hibára, a rendszeresen és egyre nagyobb számban előforduló újrapróbálkozások gyakran olyan problémát jeleznek, amely meghibásodást okozhat, vagy jelenleg is kihat az alkalmazás teljesítményére és rendelkezésre állására.

  - Az átmeneti hibákat „Hiba” bejegyzések helyett naplózza „Figyelmeztetés” bejegyzésként, nehogy a monitorozó rendszerek alkalmazáshibaként észleljék őket, és téves riasztásokat küldjenek.

  - Érdemes egy olyan értéket tárolni a naplóbejegyzésekben, amely jelzi, ha az újrapróbálkozásokat a szolgáltatás szabályozása, vagy más típusú hibák (például csatlakozási hibák) okozták, hogy meg tudja őket különböztetni az adatok elemzése során. A szabályozási hibák számának növekedése gyakran egy tervezési hibát jelez az alkalmazásban, vagy azt, hogy át kell váltani egy dedikált hardvereket kínáló prémium szolgáltatásra.

  - Fontolja meg az újrapróbálkozási mechanizmust tartalmazó műveletek teljes időtartamának mérését és naplózását. Ez remekül szemlélteti az átmeneti hibák általános hatását a felhasználók válaszidejére, a folyamat késleltetésére és az alkalmazás használati eseteinek hatékonyságára. Emellett az újrapróbálkozások számát is naplózza, hogy megismerje a válaszidőhöz hozzájáruló tényezőket.

  - Fontolja meg egy telemetrikus és monitorozó rendszer használatát, amely riasztást ad ki, ha növekedni kezd a hibák száma és mértéke, az újrapróbálkozások átlagos száma, vagy a műveletek sikeres elvégzésének teljes időtartama.

- **Kezelje a folyamatosan meghiúsuló műveleteket:**
  
  - Elő fognak fordulni olyan esetek, amikor a művelet minden kísérletnél folyamatosan meghiúsul. Rendkívül fontos végiggondolni, hogyan fogja kezelni ezt a helyzetet:

    - Az újrapróbálkozási stratégia meg fogja határozni a művelet újrapróbálkozásának maximális számát, viszont nem akadályozza meg, hogy az alkalmazás még egyszer megismételje a műveletet ugyanannyi újrapróbálkozással. Például ha egy megrendelésfeldolgozó szolgáltatás meghiúsul egy végzetes hibával, és emiatt működésképtelenné válik, az újrapróbálkozási stratégia kapcsolati időtúllépést észlelhet, és átmeneti hibának tekintheti azt. A kód meg fogja próbálni újra végrehajtani a műveletet a megadott számú alkalommal, majd feladja. Amikor azonban egy másik ügyfél lead egy megrendelést, a rendszer ismét meg fogja kísérelni a műveletet – annak ellenére, hogy az minden alkalommal garantáltan meghiúsul.

    - Ha el szeretné kerülni a folyamatosan meghiúsuló műveletek újbóli végrehajtását, vegye fontolóra az [áramkör-megszakító minta](../patterns/circuit-breaker.md) alkalmazását. Ebben a mintában, ha a hibák száma meghaladja a határértéket a megadott időtartományon belül, a rendszer azonnal visszaküldi a kéréseket hibaként a meghívónak, és meg sem próbál hozzáférni a hibás erőforráshoz vagy szolgáltatáshoz.

    - Az alkalmazás időről időre tesztelheti a szolgáltatást (időszakos jelleggel és nagyon hosszú kérések közötti időközökkel), hogy észlelje, mikor válik elérhetővé. A megfelelő időköz az adott forgatókönyvtől függ, például a művelet kritikusságától és a szolgáltatás jellegétől, és a hossza néhány perc vagy akár több óra is lehet. Amikor a teszt sikeresen befejeződik, az alkalmazás folytathatja a normál működést, és kéréseket továbbíthat az újonnan helyreállított szolgáltatásnak.

    - Addig is lehetősége van visszaváltani a szolgáltatás egy másik (például egy másik adatközpontban vagy alkalmazásban található) példányára, használhat egy kompatibilis (talán egyszerűbb) funkciókat kínáló, hasonló szolgáltatást, vagy elvégezhet néhány alternatív műveletet annak reményében, hogy a szolgáltatás hamarosan elérhetővé válik. Például célszerű lehet egy üzenetsorban vagy egy adattárban tárolni a szolgáltatásra vonatkozó kéréseket, és később visszajátszani őket. Egyéb esetben lehet, hogy át tudja irányítani a felhasználót az alkalmazás egy másik példányára, és csökkenteni tudja az alkalmazás teljesítményét úgy, hogy az még elfogadható funkciókat kínáljon, vagy szimplán visszaadhat egy üzenetet a felhasználónak, amely jelzi, hogy az alkalmazás jelenleg nem érhető el.

- **Egyéb szempontok**
  
  - Amikor meghatározza az újrapróbálkozások számát és a szabályzatok újrapróbálkozási időközeit, gondolja át, hogy a szolgáltatás vagy az erőforrás művelete egy hosszú lefutású vagy többlépéses művelet része-e. Ha az egyik műveleti lépés meghiúsul, nehéz vagy költséges lehet kompenzálni a többi sikeresen végrehajtott lépést. Ebben az esetben a nagyon hosszú időköz és a sok újrapróbálkozás elfogadható lehet, amíg nem blokkolja a többi műveletet a korlátozott erőforrások tartásával vagy zárolásával.

  - Gondolja át, hogy okozhat-e adatinkonzisztenciát, ha megpróbálja újra végrehajtani ugyanazt a műveletet. Ha a rendszer megismétli egy többlépéses folyamat néhány részét, és a műveletek nem idempotensek, az inkonzisztenciához vezethet. Például egy értéknövelő művelet érvénytelen eredményt fog megadni, ha megismétlik. Ha a rendszer megismétel egy műveletet, amely üzenetet küld egy üzenetsorba, és az üzenetfogyasztó nem tudja észlelni a duplikált üzeneteket, az inkonzisztenciát okozhat a fogyasztóban. Ennek megelőzése érdekében biztosítsa, hogy minden lépést idempotens műveletként tervez meg. Idempotens kapcsolatos további információkért lásd: [Idempotens minták][idempotency-patterns].

  - Vegye figyelembe azoknak a műveleteknek hatókörét, amelyeket a rendszer újra fog próbálni. Például egyszerűbb lehet olyan szinten megvalósítani az újrapróbálkozási kódot, amely több műveletre is kiterjed, és az összeset újból megkísérelni, ha az egyik meghiúsul. Ez azonban idempotencia-problémákat vagy szükségtelen visszaállítási műveleteket eredményezhet.

  - Ha több műveletre kiterjedő újrapróbálkozási hatókört választ, az újrapróbálkozási időközök meghatározásánál, a szükséges idő megfigyelésénél, valamint hibák esetén a riasztások kiadása előtt vegye figyelembe az összes művelet teljes késését.

  - Gondolja végig, hogyan érintheti az újrapróbálkozási stratégiája a szomszédokat és a többi bérlőt egy megosztott alkalmazásban, vagy amikor megosztott erőforrásokat és szolgáltatásokat használ. Az agresszív újrapróbálkozási szabályzatok egyre több átmeneti hibát okozhatnak a többi felhasználónál, és az olyan alkalmazásokban, amelyek közösen használják ezeket az erőforrásokat és szolgáltatásokat. Hasonlóképpen, az erőforrások és szolgáltatások többi felhasználója által alkalmazott újrapróbálkozási szabályzatok is hatással lehetnek az Ön alkalmazására. A kritikus fontosságú alkalmazások esetén prémium szolgáltatásokat is használhat, amelyek nincsenek megosztva. Ezek sokkal nagyobb irányítást tesznek lehetővé az erőforrások és szolgáltatások terhelése és szabályozása felett, ami segíthet megindokolni a többletköltséget.

## <a name="more-information"></a>További információ

- [Azure-szolgáltatásokra vonatkozó újrapróbálkozási irányelvek](./retry-service-specific.md)
- [Áramkör-megszakító minta](../patterns/circuit-breaker.md)
- [Kompenzáló tranzakció mintája](../patterns/compensating-transaction.md)
- [Idempotens minták][idempotency-patterns]

<!-- links -->

[idempotency-patterns]: https://blog.jonathanoliver.com/idempotency-patterns/
