---
title: Event Sourcing
description: Használhat egy csak hozzáfűzéssel bővíthető tárat az egy tartomány adatain elvégzett műveleteket leíró események teljes sorozatának rögzítésére.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- data-management
- performance-scalability
ms.openlocfilehash: 9a0bf170c9b54c3b2ee9cc91d6dcb5c55a13b96a
ms.sourcegitcommit: ea7108f71dab09175ff69322874d1bcba800a37a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/17/2018
---
# <a name="event-sourcing-pattern"></a>Események forráskezelése minta

[!INCLUDE [header](../_includes/header.md)]

Ahelyett, hogy a tartomány adatainak csak az aktuális állapotát tárolná, egy csak hozzáfűzéssel bővíthető tár használatával rögzítheti az adatokon végzett műveletek teljes sorozatát is.
A tároló rekordrendszerként működik, és a használatával tényleges táblává alakíthatóak tartományi objektumok. Ez egyszerűsítheti a feladatokat az összetett tartományokban, mivel nem szükséges szinkronizálni az adatmodellt és a vállalati tartományt, miközben nő a teljesítmény, a skálázhatóság és a válaszképesség. Emellett konzisztenciát kölcsönöz a tranzakciós adatoknak, és teljeskörű naplókat és előzményeket tárol, ami lehetővé teszi a kompenzáló műveletek végrehajtását.

## <a name="context-and-problem"></a>Kontextus és probléma

A legtöbb alkalmazás adatokkal dolgozik, és a szokásos megközelítés szerint az alkalmazás az adatok aktuális állapotát tárolja, és folyamatosan frissíti, ahogy a felhasználók használják azokat. Például a hagyományos CRUD (create – létrehozás, read – olvasás, update – frissítés, delete – törlés) modellben a tipikus adatfolyamat szerint az alkalmazás beolvassa az adatokat a tárolóból, bizonyos módosításokat végez azokon, majd frissíti az adatok aktuális állapotát az új értékekkel,&mdash;gyakran olyan tranzakciók használatával, amelyek zárolják az adatokat.

A CRUD megközelítésnek vannak bizonyos korlátai:

- A CRUD rendszerek a frissítési műveleteket közvetlenül az adattárolóban hajtják végre, ami ronthatja a teljesítményt és a válaszképességet, valamint a szükséges feldolgozási többletterhelés miatt korlátozhatja a skálázhatóságot.

- A nagyszámú egyidejű felhasználót tartalmazó együttműködési tartományokban az adatfrissítés konfliktusok valószínűbbek, mivel a frissítési műveletek egyetlen adatelemen vannak végrehajtva.

- Hacsak nincs valamilyen további naplózási mechanizmus kiépítve, amely az egyes műveletek részleteit egy külön naplóban rögzíti, az előzmények elvesznek.

> A CRUD megközelítés korlátairól részletesebben a [CRUD – csak akkor, ha megengedheti](https://blogs.msdn.microsoft.com/maarten_mullender/2004/07/23/crud-only-when-you-can-afford-it-revisited/) című cikkben olvashat.

## <a name="solution"></a>Megoldás

Az Események forráskezelése minta az adatokon végrehajtott műveletek kezelésére eseménysorozat-alapú megközelítést definiál, amelyben az események egy csak hozzáfűzéssel bővíthető tárban vannak rögzítve. Az alkalmazáskód ez egyes adatműveleteket imperatív módon leíró események sorát küldi a tárolóba, ahol azok megőrződnek. Mindegyik esemény az adatok módosításainak egy halmazát (például `AddedItemToOrder`) jelöli.

Az eseményeket egy eseménytár őrzi, amely az adatok aktuális állapotát rögzítő rekordrendszerként (mérvadó adatforrásként) működik. Az eseménytár általában közzéteszi ezeket az eseményeket, hogy a felhasználók értesüljenek róluk, és szükség szerint kezelhessék azokat. A fogyasztók például inicializálhatnak olyan feladatokat, amelyek az eseményekben lévő műveleteket más rendszerekre alkalmazzák, vagy egyéb kapcsolódó műveleteket hajtanak végre, amelyek a működéshez szükségesek. Vegyük észre, hogy az eseményeket létrehozó alkalmazáskód elválik az eseményekre feliratkozó rendszerektől.

Az eseménytár által közzétett események tipikus felhasználása az entitások tényleges táblán alapuló nézeteinek karbantartása, ahogy az alkalmazás műveletei módosítják azokat, valamint a külső rendszerekkel való integráció biztosítása. Például a rendszer megtarthatja az összes olyan ügyfélmegrendelés tényleges táblán alapuló nézetét, amelyek a felhasználói felület részein megjelentek. Ahogy az alkalmazás új megrendeléseket ad hozzá, tételeket ad hozzá vagy távolít el a megrendelésekben, szállítási információkat ad hozzá, az ezeket a módosításokat leíró események kezelhetőek, és a használatukkal frissíthetőek a [tényleges táblán alapuló nézetek](materialized-view.md).

Emellett az alkalmazásoknak bármikor lehetősége van olvasni az eseményelőzményeket, és azok használatával tényleges táblává alakítani az entitások aktuális állapotát az adott eseményhez tartozó összes esemény visszajátszásával és feldolgozásával. Ez történhet igény szerint a tartományobjektumok tényleges táblává alakításával a kérések feldolgozása során, vagy pedig ütemezett feladatokon keresztül, hogy az entitás állapota tényleges táblán alapuló nézetként tárolható legyen a megjelenítési réteg kiszolgálására.

Az ábrán a minta áttekintése látható, beleértve az eseménystream használata kínálta egyes lehetőségeket, például a tényleges táblán alapuló nézetek létrehozását, az események külső alkalmazásokkal és rendszerekkel való integrálását, valamint az események visszajátszását az adott entitások aktuális állapotai leképezéseinek létrehozásához.

![Az Események forráskezelése minta áttekintése és példája](./_images/event-sourcing-overview.png)


Az Események forráskezelése minta az alábbi előnyöket biztosítja:

- Az események nem módosíthatók, és egy csak hozzáfűzési művelettel tárolhatók. Az eseményt inicializáló felhasználói felület, munkafolyamat vagy folyamat folytatódhat, és az eseményeket kezelő feladatok a háttérben futhatnak. Kombinálva azzal a ténnyel, hogy a tranzakciók a feldolgozás során nem versengenek, ez rendkívüli mértékben javíthatja az alkalmazások teljesítményét és skálázhatóságét, különösen a megjelenítési szint vagy a felhasználói felület vonatkozásában.

- Az események olyan egyszerű objektumok, amelyek valamely megtörtént műveletet írnak le, az esemény által jelölt művelet leírásához szükséges társított adatokkal együtt. Az események közvetlenül nem frissítik az adattárakat. Egyszerűen csak rögzítve vannak, hogy a megfelelő időben kezelhetőek legyenek. Ez egyszerűbbé teheti az implementálást és a felügyeletet.

- Az események általában jelentéssel bírnak a tartományszakértők számára, miközben az [objektumrelációs impedanciaeltérés](https://en.wikipedia.org/wiki/Object-relational_impedance_mismatch) miatt az összetett adatbázistáblák nehezen érthetőek lehetnek. A táblák olyan mesterséges szerkezetek, amelyek a rendszer aktuális állapotát és nem a bekövetkezett eseményeket mutatják.

- Az Események forráskezelése segítségével megelőzhető, hogy a párhuzamos frissítések ütközést okozzanak, mivel nem szükséges az objektumokat közvetlenül az adattárban frissíteni. Azonban a tartománymodellt továbbra is úgy kell kialakítani, hogy védje magát az olyan kérések ellen, amelyek inkonzisztens állapotot okozhatnak.

- Az események csak hozzáfűzéssel bővíthető tárolása biztosítja, hogy a naplózás használatával monitorozhatóak az adattáron végrehajtott műveletek, az aktuális állapot a tényleges táblán alapuló nézetekként vagy leképezésekként újra létrehozható az események bármely pillanatban való visszajátszásával, valamint támogatja a tesztelést és a hibakeresést is. Emellett azzal, hogy a változásokat kompenzáló eseményekkel kell visszavonni, a visszavont módosítások is szerepelnek az előzményekben, ami nem így lenne, ha a modell egyszerűen csak az aktuális állapotot tárolná. Az események listájának használatával emellett elemezhető az alkalmazás teljesítménye, és észlelhetőek a felhasználók viselkedési trendjei, valamint egyéb, üzleti szempontból hasznos információk is kinyerhetőek.

- Az eseménytár eseményeket indít, és a feladatok ezekre válaszképp hajtanak végre műveleteket. A feladatok ily módú leválasztása az eseményekről biztosítja a rugalmasságot és a bővíthetőséget. A feladatok ismerik az események típusát és adatait, az eseményeket kiváltó műveleteket azonban nem. További előny, hogy egyszerre több feladat is kezelheti az egyes eseményeket. Ez egyszerűsíti az integrációt az olyan szolgáltatásokkal és rendszerekkel, amelyek csak az eseménytár által kiváltott új eseményeket figyelik. Azonban az Események forráskezelése események általában nagyon alacsony szintűek, így szükség lehet inkább adott integrációs események létrehozására.

> Az Események forráskezelése mintát általában a CQRS mintával kombinálva alkalmazzák az adatkezelési feladatoknak az eseményekre való válaszként történő végrehajtásával és a tárolt eseményekből tényleges táblán alapuló nézeteket létrehozásával.

## <a name="issues-and-considerations"></a>Problémák és megfontolandó szempontok

A minta megvalósítása során az alábbi pontokat vegye figyelembe:

A rendszer végül csak a tényleges táblán alapuló nézetek vagy adatleképezések a visszajátszott adatok alapján való létrehozásával válik konzisztenssé. Bizonyos időbeli késedelem van az eseményeknek a kérések kezelése során az eseménytárba való felvétele, az események közzététele és az események a fogyasztók általi feldolgozása között. Ez alatt az idő alatt az entitások további módosításait leíró új események érkezhetnek az eseménytárba.

> [!NOTE]
> A végleges konzisztenciával kapcsolatos információkért lásd: [Adatkonzisztencia – Ismertető](https://msdn.microsoft.com/library/dn589800.aspx).

Az eseménytár szolgál az adatok állandó forrásaként, emiatt az eseményadatokat soha nem szabad frissíteni. Az entitások frissítésének egyetlen módja a módosítások visszavonására, ha egy kompenzáló eseményt vesz fel az eseménytárba. Ha a megőrzött események formátumát (tehát nem magukat az adatokat) kell módosítani, például egy migrálás során, nehézkes lehet a tárolóban lévő meglévő eseményeket kombinálni az új verzióval. Esetleg szükséges lehet a módosításokat végrehajtó összes eseményt végigléptetni, hogy kompatibilisek legyenek az új formátummal, vagy az új formátumot használó új eseményeket hozzáadni. Érdemes lehet az eseményséma minden verzióját verzióbélyeggel ellátni a régi és az új eseményformátumok megtartásához.

Az eseménytárban több szálon vagy példányban futó alkalmazások is tárolhatnak eseményeket. Az eseménytárban tárolt események konzisztenciája létfontosságú, csakúgy, mint az egyes entitásokra ható események sorrendje (az entitásokat érintő változások sorrendje befolyásolja azok aktuális állapotát). Segíthet elkerülni a problémákat, ha az egyes eseményeket időbélyeggel látja el. Egy másik gyakori eljárás, ha a kérésekből eredő minden eseményt egy növekményes azonosítóval jelöl. Ha két művelet egyidőben próbál eseményeket hozzáadni ugyanahhoz az entitáshoz, az eseménytár elutasíthatja az olyan eseményeket, amelyek entitásazonosítója és eseményazonosítója megegyezik egy meglévőével.

Nincs szabványos megközelítés vagy létező mechanizmus (mint például az SQL-lekérdezések) az események olvasására az adatok beszerzéséhez. Adatként kizárólag eseménystreamek olvashatóak ki feltételként egy eseményazonosítót megadva. Az eseményazonosító tipikusan egyedi entitásokra képezhető le. Egy entitás aktuális állapota csak úgy határozható meg, ha visszajátssza az összes hozzá kapcsolódó eseményt az entitás eredeti állapotából kiindulva.

Az egyes eseménystreamek hossza kihat a rendszer kezelésére és frissítésére. Nagyobb streamek esetén érdemes lehet adott időközönként, például adott számú eseményenként pillanatképeket létrehozni. Az entitás aktuális állapota a pillanatképből kiindulva és az annak időpontját követően bekövetkezett eseményeket visszajátszva kérhető le. Az adatpillanatképek létrehozásával kapcsolatos további információkért lásd: [Pillanatkép Martin Fowler Enterprise Application Architecture webhelyén](http://martinfowler.com/eaaDev/Snapshot.html) és [Mester-alárendelt pillanatkép-replikálás](https://msdn.microsoft.com/library/ff650012.aspx).

Bár az Események forráskezelése minimalizálja az adatok ütköző frissítésének esélyét, az alkalmazásnak így is képesnek kell lennie kezelni a végleges konzisztenciából és a tranzakciók hiányából eredő inkonzisztenciát. Például egy, a leltárkészlet csökkenését jelző esemény érkezhet az adattárba, miközben éppen egy, az adott tételre vonatkozó megrendelés érkezik, ami azt eredményezi, hogy a két művelet egyeztetni kell, és vagy az ügyfelet tájékoztatni kell, vagy egy teljesítetlen rendelést létrehozni.

Az esemény-közzététel lehet „legalább egyszer” beállítású, amely esetben az események fogyasztóinak idempotensnek kell lenniük. Az eseményekben leírt frissítéseket nem szabad ismételten alkalmazniuk, ha az esemény többször is kezelve lett. Például ha egy fogyasztó több példánya is karbantartja egy entitás valamely tulajdonságának összesítését, például a leadott megrendelések teljes számát, csak az egyiknek szabad sikeresen növelnie az összesítést a megrendelésleadás esemény bekövetkezésekor. Bár nem kulcsfontosságú jellemzője az Események forráskezelése mintának, ez az általános implementációs döntés.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes ezt a mintát használni?

Az alábbi esetekben alkalmazza ezt a mintát:

- Ha rögzíteni szeretné a szándékokat, a célokat vagy az okokat az adatokban. Például egy ügyfélentitás módosításai rögzíthetők adott eseménytípusok sorozataként, például _Hazaköltözött_, _Lezárt számla_ vagy _Elhalálozott_.

- Ha elengedhetetlen az ütköző adatfrissítések előfordulásának minimalizálása vagy teljes megszüntetése.

- Ha szeretné a bekövetkező eseményeket rögzíteni, majd azok visszajátszásával visszaállítani a rendszer állapotát, visszafordítani a változásokat, vagy megőrizni az előzményeket és naplózni az eseményeket. Például ha egy feladat több lépésből áll, esetleg szükség lehet olyan műveletek végrehajtására, amelyek visszavonják a frissítéseket, majd egyes lépések visszajátszásával visszaállítják az adatokat egy konzisztens állapotba.

- Ha az események használata az alkalmazás működésének természetes funkciója, és további fejlesztést vagy implementálási ráfordítást igényel.

- Ha le szeretné választani az adatok bevitelének vagy frissítésének folyamatát az adott műveletek alkalmazásához szükséges feladatokról. Ez történhet a felhasználói felület teljesítményének javítása vagy az események olyan figyelőkre való terjesztésének érdekében, amelyek az események bekövetkezésekor végeznek műveleteket. Például egy bérlistarendszer integrálása egy költségbeküldési webhellyel azzal a céllal, hogy az eseménytár által a webhelyen megvalósított adatfrissítésekre válaszként indított eseményeket a webhely és a bérlistarendszer egyaránt feldolgozza.

- Ha szeretné tudni rugalmasan módosítani a tényleges táblává alakított modellek és entitásadatok formátumát a követelmények változásakor, vagy&mdash;a CQRS-sel való együttes használatkor&mdash;igazítania kell valamely olvasási modellt vagy az adatokat feltáró nézeteket.

- A CQRS-sel való együttes használatkor, és ha a végleges konzisztencia az olvasási modellek frissítése közben elfogadható, vagy az eseménystreamekből származó entitások és adatok rehidratálása eredményezte teljesítményváltozás elfogadható.

Nem érdemes ezt a mintát használni a következő helyzetekben:

- Kis vagy egyszerű tartományok, rendszerek, kis vagy semmilyen üzleti logikával nem rendelkező rendszerek, valamint a hagyományos CRUD adatkezelési mechanizmusokkal eleve jól együttműködő nem tartományi rendszerek esetén.

- Olyan rendszerek esetén, ahol a konzisztencia és az adatnézetek valós idejű frissítése szükséges.

- Olyan rendszerek esetén, ahol a naplózásra, az előzmények megőrzésére, a visszaállítási képességekre és a műveletek visszajátszására nincs szükség.

- Olyan rendszerek esetén, ahol a mögöttes adatok ütköző frissítései rendkívül kis eséllyel fordulnak elő. Például olyan rendszerek esetén, amelyek túlnyomórészt hozzáadnak, nem pedig frissítenek adatokat.

## <a name="example"></a>Példa

Egy konferenciakezelő rendszerben nyomon kell követni a kész foglalások számát, hogy lehessen tudni, van-e még szabad hely egy adott konferencián, ha egy érdeklődő foglalni szeretne. A rendszer az egyes konferenciákra érkezett foglalások számát legalább kétféleképpen tárolhatja:

- A rendszer az egyes konferenciákra érkezett foglalások számára vonatkozó adatokat tárolhatja külön entitásként a foglalási adatokat tartalmazó adatbázisban. A foglalások vagy a lemondások alkalmával a rendszer megfelelően növelheti vagy csökkentheti ezt a számot. Ez a megközelítés elméletben egyszerű, azonban skálázhatósági problémákat okozhat, ha rövid időn belül nagy számú résztvevő próbál foglalni. Például egy foglalási időszak utolsó vagy utolsó néhány napján.

- A rendszer a foglalásokkal és lemondásokkal kapcsolatos adatokat tárolhatja eseményekként egy eseménytárban. Ezután a szabad helyek számát ezeknek az eseményeknek a visszajátszásával számíthatja ki. Ez a megközelítés az események változtathatatlansága miatt jobban skálázható lehet. A rendszernek csak az eseménytárból kell tudnia adatokat olvasni, vagy adatokat hozzáfűzni ugyanitt. A foglalások és a lemondások eseményadatai soha nem módosulnak.

A következő ábra azt mutatja be, hogy a konferenciakezelő rendszer helyfoglaló alrendszere hogyan implementálható az Események forráskezelése használatával.

![Helyfoglalási adatok rögzítése az Események forráskezelése használatával egy konferenciakezelő rendszerben](./_images/event-sourcing-bounded-context.png)


Két hely foglalása esetén a műveletek sorrendje a következő:

1. A felhasználói felület kiad egy parancsot két hely foglalására két résztvevő számára. A parancsot egy külön parancskezelő dolgozza fel. Egy, a felhasználói felületről leválasztott és a parancsként közzétett kérések kezeléséért felelős logikai rész.

2. A konferencia összes foglalásával kapcsolatos adatokat tartalmazó összesítés a foglalásokat és a lemondásokat leíró események lekérdezésével állatható össze. Az összesítés neve `SeatAvailability`, és egy olyan tartományi modell tartalmazza, amely az összesítésben foglalt adatok lekérdezésére és módosítására szolgáló metódusokat tárja fel.

    > A fontolóra vehető optimalizálási lehetőségek a pillanatképek használata (így az összesítés aktuális állapotának bekéréséhez nem szükséges a teljes eseménylistát lekérdezni és visszajátszani), valamint az összesítés gyorsítótárazott másolatának tárolása a memóriában.

3. A parancskezelő egy, a tartományi modellben feltárt metódust hív meg a foglalások intézésére.

4. A `SeatAvailability` összesítés rögzíti a lefoglalt helyek számát tartalmazó eseményt. A következő alkalommal, amikor az összesítés valamilyen eseményt alkalmaz, a szabad helyek számát a rendszer az összes foglalás használatával számítja ki.

5. A rendszer hozzáfűzi az új eseményt az események sorához az eseménytárban.

Ha egy felhasználó lemond egy helyet, a rendszer ugyanezt a folyamatot követi, azzal a különbséggel, hogy a parancskezelő egy helylemondási eseményt létrehozó parancsot ad ki és fűz hozzá az eseménytárhoz.

Amellett, hogy több lehetőséget kínál a skálázásra, az eseménytár teljes körű előzményeket – vagy naplózást – is biztosít a konferencia foglalásaival és lemondásaival kapcsolatban. Az eseménytárban szereplő események szolgálnak pontos rekordként. Az összesítéseket nem szükséges egyéb módon megőrizni, mivel a rendszer könnyedén visszajátszhatja az eseményeket, és visszaállíthatja az állapotot bármely időpontra.

> Ezzel a példával kapcsolatban további információkat [az Események forráskezelése bemutatásában](https://msdn.microsoft.com/library/jj591559.aspx) talál.

## <a name="related-patterns-and-guidance"></a>Kapcsolódó minták és útmutatók

Az alábbi minták és útmutatók szintén hasznosak lehetnek a minta megvalósításakor:

- [Parancskiadási és lekérdezési felelősségek elkülönítése (CQRS) minta](cqrs.md). A CQRS implementálások állandó adatforrását biztosító írási tároló alapjául gyakran az Események forráskezelése minta egy implementálása szolgál. A szakasz azt ismerteti, hogyan lehet különböző felületek használatával elkülöníteni az alkalmazások adatolvasó műveleteit az adatfrissítő műveletektől.

- [A Materialized View minta](materialized-view.md). Az Események forráskezelése mintán alapuló rendszerekben használt adattárak tipikusan nem nagyon alkalmasak a hatékony lekérdezésre. Ehelyett az általános megközelítés szerint rendszeres időközönként vagy az adatok változásakor szokás előfeltöltött nézeteket létrehozni az adatokról. Ez a szakasz ennek a menetét mutatja be.

- [Kompenzáló tranzakció mintája](compensating-transaction.md). Az Események forráskezelése tárban található meglévő adatok nem frissülnek, hanem új bejegyzések lesznek hozzáadva, amelyek átváltják az entitások állapotát az új értékekre. A módosítások visszavonásához kompenzáló bejegyzéseket kell alkalmazni, mivel a megelőző módosításokat nem lehet egyszerűen visszavonni. A szakasz azt ismerteti, hogyan lehet visszavonni a korábbi műveletek által végrehajtott módosításokat.

- [Adatkonzisztencia – Ismertető](https://msdn.microsoft.com/library/dn589800.aspx). Ha az Események forráskezelése mintát egy külön olvasási tárral vagy tényleges táblán alapuló nézetekkel alkalmazza, a beolvasott adatok nem azonnal, hanem csak végül lesznek konzisztensek. A szakasz az elosztott adatok konzisztenciájának megőrzésével kapcsolatos problémákat foglalja össze.

- [Adatparticionálási útmutató](https://msdn.microsoft.com/library/dn589795.aspx). Az Események forráskezelése minta alkalmazása esetén az adatokat gyakorta szokás particionálni a skálázhatóság javítása, a versengés csökkentése és a teljesítmény optimalizálása érdekében. A szakasz az adatok diszkrét partíciókra való felosztását és az esetlegesen felmerülő problémákat ismerteti.

- Greg Young bejegyzése: [Miért érdemes az Események forráskezelése mintát alkalmazni?](http://codebetter.com/gregyoung/2010/02/20/why-use-event-sourcing/).
