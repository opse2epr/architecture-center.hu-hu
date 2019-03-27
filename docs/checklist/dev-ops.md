---
title: DevOps ellenőrzőlisták
titleSuffix: Azure Design Review Framework
description: Fejlesztési és üzemeltetési kapcsolatos útmutatást nyújtó ellenőrzőlista.
author: dragon119
ms.date: 01/10/2018
ms.topic: checklist
ms.service: architecture-center
ms.subservice: cloud-design-principles
ms.custom: checklist
ms.openlocfilehash: 1a000c811cce57cc9b1fcda84d0eb7e2a1312aca
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298797"
---
# <a name="devops-checklist"></a>DevOps-ellenőrzőlista

Fejlesztési és üzemeltetési integrációja, fejlesztési, minőségbiztosítási és egységes kulturális környezetet az IT-üzemeltetők és az általa kínált szoftverek folyamatok készletének. Ezzel az ellenőrzőlistával kiindulási pontként annak ellenőrzésére fog használni a DevOps-kultúra és a folyamat.

## <a name="culture"></a>Kulturális környezet

**Győződjön meg arról összehangoltságot szervezetek és a csapatok között.** Erőforrások, célú, célok és prioritások vállalaton belüli ütközések lehet sikeres műveletek kockázata. Győződjön meg arról, hogy az üzleti, a fejlesztést és a műveletek közötti összes egy vonalban van-e.

**Győződjön meg arról, a teljes csoport tisztában van azzal a szoftveres életciklusainak.** A csapat által igényelt tudni, hogy az alkalmazás teljes életciklusát, és mely részét életciklusa az alkalmazás jelenleg a. Ez segít az összes csoport tagjai, érdemes lehet mire most, és mi, érdemes lehet tervezése és előkészítése a jövőben.

**Ciklus idejének csökkentése érdekében.** Célja, hogy minimalizálja a használható fejlett szoftver ötleteket áthelyezéséhez szükséges időt. A méretét és az egyes kiadások, hogy a teszt terheket alacsony hatókörének korlátozása. Automatizálja a buildelési, tesztelési, konfigurációs és központi telepítési folyamatokat, amikor csak lehetséges. Törölje a fejlesztők körében, és a fejlesztők és a műveletek közötti kommunikáció bármely akadályai.

**Tekintse át, és javíthatja a folyamatokat.** A folyamatok és eljárások, automatizált és a manuális, soha nem olyan végső. Állítsa be a reguláris felülvizsgálatai aktuális munkafolyamatokat, eljárásokra és dokumentációt,. a cél az folyamatos fejlesztések eszköze.

**Ezt a proaktív tervezés.** Proaktív módon tervezze meg a hiba. Folyamatok kell, hogy gyorsan azonosíthatja a problémákat, ha fordulhat elő, felterjesztheti a javítsa ki a megfelelő csoport tagjai, valamint feloldási megerősítése.

**Ismerje meg, a hibák után.** Hibák elkerülhetetlen, de fontos, hogy ismerje meg, a hibák elkerülhető, ismétlődő őket. -Működési hiba történik, ha a probléma, a dokumentum a OK és megoldás osztályozhatja és oszthat meg bármely tapasztalatokat, amelyek is megtanulta. Amikor csak lehetséges, frissítse a hozhat létre folyamatokat, amelyek automatikus észlelése a jövőben ezt a fajta hiba.

**Optimalizálja a sebességet, és adatokat gyűjteni.** Minden tervezett fokozása egy összefüggésben. Munka a lehető legkisebb lépésekben. Új ötletek gyökérkönyvtárral kísérletek. Alakítsa ki úgy a kísérletek, így begyűjtheti a termelési adatokon, felmérheti azok hatékonyságát. Fel kell készülni sikertelen gyors, ha a feltételezést helytelen.

**Hagyjon időt a tanulást.** Hibák és a sikeres adja meg a helyes a tanulási lehetőségek. Mielőtt az új projektek, gyűjtse össze a fontos tapasztalatokat, és ellenőrizze, hogy ezek a leckék használja fel, a csapat elegendő időt engedélyezése. Is meg kell adni a csapat az idő fejlesztheti, kísérletezhet, és további információ új eszközöket és technikákat.

**A dokumentum-műveletek.** A dokumentum minden eszközök, folyamatok és a termék kódként minőségű azonos szintű automatizált feladatokat. A dokumentum a jelenlegi kialakítás és azok a rendszerek támogatja, valamint helyreállítási folyamatok és egyéb karbantartási eljárások architektúráját. A lépéseket, akkor hajtja végre, elméletileg optimális folyamatok koncentrálhat. Rendszeresen tekintse át, és frissítse a dokumentációt. Kód ügyeljen arra, hogy tartalmas megjegyzések tartalmazza, különösen a nyilvános API-k és eszközök használatával automatikusan létrehozhat egy code dokumentáció, amikor csak lehetséges.

**Tudásbázis megosztása.** Dokumentáció csak akkor hasznos, ha a felhasználók tudják, létezik, és is megtalálhatják azt. Győződjön meg arról, a dokumentáció az rendszerezett és könnyen áttekinthető. Legyen kreatív: Tudásbázis barna csomag (nem hivatalos bemutatók), a videók és a hírlevelek használata.

## <a name="development"></a>Fejlesztés

**Adja meg a fejlesztők az utolsóig üzemi környezetben.** Ha fejlesztési és tesztelési környezetek nem egyezik meg az üzemi környezetben, meglehetősen nehéz teszteléséhez, és diagnosztizálhatja a problémákat. Ezért tartsa a fejlesztési és tesztelési környezetek, az éles környezetben a lehető közelében. Győződjön meg arról, hogy teszt adatai konzisztensek legyenek az éles környezetben használt adatok akkor is, ha a mintaadatokat, és nem valódi termelési adatok (az adatvédelmi és megfelelőségi okok miatt). Tervezze meg létrehozni és mintaadatokat teszt anonimizálása.

**Győződjön meg arról, hogy minden jogosult csoport tagjai infrastruktúra kiépítése, és helyezze üzembe az alkalmazást.** Hasonló erőforrások beállítása és az alkalmazás üzembe helyezése nem terjed ki összetett manuális feladat vagy a rendszer részletes műszaki ismerete. A megfelelő engedélyekkel rendelkező felhasználók létrehozása vagy hasonló erőforrások üzembe helyezése nélkül, a üzemeltetési csapat képesnek kell lennie.

> Ez a javaslat nem jelenti azt, hogy bárki leküldheti az élő frissítés az éles környezetbe. Annak hamarosan csökkentése fennakadások nélkül használható a fejlesztési és a QA csapatok az utolsóig üzemi környezeteket hozhat létre.

**Az elemzés az alkalmazás kialakítása.** Szeretné megtudni, az alkalmazás állapotát, teljesítményét, és hogy kapcsolatos esetleges hibák vagy problémák tapasztalhatók ismernie kell. Mindig olyan kialakítást a tervezési követelmény, és hozhat létre a rendszerállapot a kezdetektől az alkalmazásba. Eseménynaplózás a kiváltó okok elemzése, de is telemetriai és metrikákat kíván monitorozni az átfogó állapot, és az alkalmazás használati Instrumentation tartalmaznia kell.

**Nyomon követheti a műszaki adósságot.** A projektek közül sok kiadási ütemtervekkel is első szándék mértékű vagy egy másik kódot minőségét. Mindig tartsa nyomon követése, ha ez történik. Dokumentum-parancsikonok vagy más nonoptimal megvalósításokhoz, és nyissa meg újra ezeket a problémákat a jövőben ütemezése.

**Fontolja meg a frissítések terjesztése közvetlenül az éles környezetbe.** A teljes kiadási ciklus idő csökkentése érdekében fontolja meg a megfelelő elküldése kód véglegesítés közvetlenül az éles környezetben tesztelt. Használat [be-vagy kikapcsolja a szolgáltatás] [ feature-toggles] szabályozhatja, hogy mely szolgáltatások engedélyezve vannak. Így gyorsan kiadni a fejlesztéstől áthelyezése használatával a be-vagy kikapcsolja a szolgáltatások engedélyezése vagy letiltása. Újraengedélyezi a kapcsolókat akkor is hasznos, ha a tesztek végrehajtása például [tesztcsoportos kiadások][canary-release], ahol egy adott funkció egy részét az éles környezet üzemel.

## <a name="testing"></a>Tesztelés

**Automatizálhatja a teszteléshez.** Manuálisan szoftvertesztelés fárasztó és ki van téve a hiba. Automatizálhatja a gyakori tesztelési feladatokat, és integrálhatja a vizsgálatok az összeállítási folyamatairól. Automatizált tesztelés biztosítja, hogy egységes tesztlefedettség és megismételhetőség. Integrált felhasználói felület tesztek által művelet automatikus eszközzel is kell végezni. Azure fejlesztést tesz lehetővé, és test-erőforrások, amelyek segítségével konfigurálhatja, és hajtsa végre a tesztelés. További információkért lásd: [fejlesztési és tesztelési][dev-test].

**Ellenőrizze a hibákat.** Ha a rendszer a szolgáltatás nem tud csatlakozni, hogy azt válaszol? Lehet, helyreállítása után a szolgáltatás ismét elérhető? Győződjön meg arról, tekintse át a teszt szokásos részét tesztelési és átmeneti környezet hibabeszúrás. Ha a tesztelési folyamattal és eljárások érett, fontolja meg az éles környezetben futó ezeket a teszteket.

**Tesztelés éles környezetben.** A kibocsátási folyamat végén nem éles környezetben történő bevezetéshez. Tesztek kell annak érdekében, hogy az, hogy üzembe helyezett kód megfelelően működik-e. Ritkán frissített telepítések esetén ütemezhet rendszeres karbantartás részeként tesztelés éles környezetben.

**Teljesítménnyel kapcsolatos problémák korai azonosítására teljesítménytesztelési automatizálhatja.** Lehet, hogy komoly teljesítményprobléma hatásának csak olyan szigorúak, mint a kódot tartalmaz hibát. Automatikus funkcionális tesztek megakadályozhatja az alkalmazások hibái, amíg azok nem képes észlelni teljesítményproblémákat okozhat. Adja meg például a késés, a betöltési időt és az erőforrás-használati metrikák elfogadható teljesítménybeli céljainak. Automatizált teljesítménytesztek felvétel a kiadási folyamatot, hogy az alkalmazás megfelel-e ezeket a célokat.

**Kapacitás teszt végrehajtása.** Egy alkalmazás előfordulhat, hogy jól működnek a tesztelési körülmények között, és ezután problémába skálázást vagy az erőforrás-korlátozások miatt éles környezetben. Mindig adja meg a várt kapacitás és a használati korlátozásokat. Ellenőrizze, hogy ellenőrizze, hogy az alkalmazás kezelni ezeket a korlátokat, de is tesztelje, mi történik, ha ezek a korlátok túllépése. A kapacitás tesztelés rendszeres időközönként kell végrehajtani.

A kezdeti kiadása után futtassa a teljesítmény és kapacitás teszteket, amikor az éles kódban végrehajtott frissítéseket. Az összegyűlt korábbi adatok Finomhangolja a tesztek és meghatározni a tesztek típusú kell elvégezni.

**Hajtsa végre, automatikus biztonsági behatolásvizsgálat.** Az alkalmazás biztonságos biztosítva olyan fontos, mint bármely más funkció tesztelése. Győződjön meg arról, automatizált behatolásvizsgálatot kell végezniük a buildelési és üzembe helyezési folyamat alapvető része. Rendszeres biztonsági vizsgálatok és a központilag telepített alkalmazások figyelése a megnyitott portok, végpontok és támadások vizsgálata biztonsági rés ütemezhet. Automatizált tesztelés nem távolítja el a részletes biztonsági ellenőrzések rendszeres időközönként szükségességét.

**Automatizált üzleti folytonossági teszt végrehajtása.** Fejlesztése a nagy méretű az üzletmenet folytonosságának, beleértve a biztonsági másolat helyreállítását a feladatátvételi tesztet. Hajtsa végre ezeket a teszteket rendszeresen automatizált folyamatok beállításával.

## <a name="release"></a>Kiadás

**Központi telepítések automatizálása.** Tesztelje az alkalmazást telepíti, átmeneti és éles környezetekben automatizálhatja. Automation lehetővé teszi, hogy a gyorsabb és megbízhatóbb központi telepítések, és biztosítja az egységes központi telepítések bármely támogatott környezetben. Eltávolítja a manuális telepítés által okozott emberi hibák kockázatát. Azt is egyszerűen kiadások kényelmes többször, minden lehetséges állásidő hatásainak minimalizálása érdekében a ütemezéséhez.

**Folyamatos integráció használja.** Folyamatos integrációs (CI) az eljárás szerint az egyesítés, rendszeres ütemezés szerint kódbázis egy központi összes fejlesztői kódot, és automatikusan végrehajtásával standard buildelési és tesztelési folyamatokat. CI biztosítja, hogy az egész csapat dolgozhat a kódbázis egy időben ütközések nélkül. Emellett biztosítja, hogy a lehető leghamarabb található kód hibák. A CI folyamat lehetőleg, minden alkalommal, amikor a kód véglegesítve, vagy be van jelölve, kell futtatnia. Legalább az naponta egyszer kell futnia.

> Vegye figyelembe a bevezetése egy [alapú trönk fejlesztési modell][trunk-based]. Ebben a modellben a fejlesztők véglegesítse egy önálló fiókirodai (törzs). Nincs olyan követelmény, hogy a véglegesítéseket soha nem felosztása a build. Ez a modell elősegíti a CI, mivel az összes funkció végezhető el a trönk, és az egyesítési ütközések feloldása, amikor a véglegesítés történik.

**Fontolja meg a folyamatos teljesítés.** Folyamatos továbbítás (CD) a kód mindig üzembe helyezéséhez automatikusan fejlesztés, tesztelés, és az utolsóig üzemi környezetben való üzembe helyezése kód készen áll az eljárás annak biztosítására, hogy. Teljes CI/CD-folyamatok létrehozására a folyamatos teljesítés hozzáadása segítségével észlelheti a kód minél hamarabb hibák, és biztosítja, hogy megfelelően tesztelt frissítéseket is kell szabadítani nagyon rövid időn belül.

> Folyamatos *üzembe helyezési* egy folyamat, amely automatikusan a frissítéseket, amelyek megfeleltek a CI/CD-folyamat keresztül, és alkalmazza őket éles környezetbe. Folyamatos üzembe helyezés robusztus automatikus tesztelése és speciális folyamat tervezési igényel, és nem lehet megfelelő összes csapatok számára.

**Győződjön meg arról, kisebb a növekményes változásokat.** Nagy kódmódosítás rendelkezik egy nagyobb lehetséges hibák bevezetni. Amikor csak lehetséges, folyamatosan kis módosításokat. A lehetséges hatásait minden módosítása korlátozza, és megkönnyíti az ismertetése, és elháríthatja a problémákat.

**Változások szabályozza.** Ellenőrizze, hogy a vezérlő, amikor frissítései láthatóak a végfelhasználók számára. Fontolja meg az ellenőrzési funkció újraengedélyezi a kapcsolókat, amikor a végfelhasználók számára engedélyezett-szolgáltatásokat.

**Alkalmazzon kiadási kezelési stratégiák telepítési kockázat csökkentéséhez.** Néhány kockázati üzembe helyezése egy alkalmazás frissítése az éles környezetben mindig jár. A kockázat minimalizálása érdekében használjon stratégiák például [tesztcsoportos kiadások] [ canary-release] vagy [kék-zöld üzembe] [ blue-green] telepíteni a frissítéseket egy részhalmazára a felhasználók. Erősítse meg, a frissítés a várt módon működik, és ezután megkezdik a frissítés a rendszer többi része.

**Dokumentálja az összes módosítást.** Kisebb módosításait és konfigurációs változásokat lehet félreértések és verziókezelését ütközés forrását. Mindig nyilvántartást törölje minden olyan módosításáról, függetlenül attól, milyen kicsi. Mindent, ami megváltozik, többek között az alkalmazott javítások, az irányelvek változásai és konfigurációs változások naplózása. (Ne tartalmazza bizalmas adatok ezeket a naplókat. Például, hogy egy hitelesítő adat frissült, és ki a módosítást, de ne jegyezze fel a frissített hitelesítő adatok log.) A rekord a módosításokat a teljes csapatnak megjelennek.

**Központi telepítések automatizálása.** Központi telepítések automatizálása, és a rendszerek működik a kibocsátás közben problémák észleléséhez. Meg kell a meglévő kód és az adatok üzemben, mielőtt a frissítés felülírja őket az összes éles példányokba kockázatcsökkentési eljárással rendelkezik. Automatikus módszere a visszaállítás előre javítások vagy visszafordítani a változásokat rendelkezik.

**Fontolja meg, így az infrastruktúra nem módosítható.** Nem módosítható infrastruktúra képes legyen, hogy az éles környezetben üzembe helyezését követően nem szabad módosítani infrastruktúra elve szerint. Ellenkező esetben kaphat olyan állapotba, amelyben alkalmi változtatások alkalmazása, így nehéz tudni, hogy pontosan mi változott. Megváltoztathatatlan infrastruktúra működik, és cserélje le a teljes kiszolgáló minden olyan új központi telepítésének részeként. Ez lehetővé teszi a kód és tesztelt és a egy telepített, a üzemeltetési környezet. Mindaddig, amíg a következő létrehozása és üzembe helyezése a ciklus, üzembe helyezését követően a nem módosítani az infrastruktúra-összetevőket.

## <a name="monitoring"></a>Figyelés

**Győződjön meg arról, rendszerek rendszernek megfigyelhetőnek.** Az üzemeltetési csapat mindig betekintéssel kell rendelkezniük egyértelmű állapotának és a rendszer vagy a szolgáltatás állapotát. Külső egészségügyi végpontok beállítása állapotának figyelése, és győződjön meg arról, hogy a műveletek metrikák szoftverfejlesztők alkalmazások kódolása. Lehetővé teszi az események összevetését a rendszerek általános és következetes sémát használják. [Az Azure Diagnostics] [ azure-diagnostics] és [Application Insights] [ app-insights] állapotát és Azure-erőforrások állapotának nyomon követése a szabványos módszere van. A Microsoft [Operation Management Suite] [ oms] központi figyelési és felügyeleti felhőalapú vagy hibrid megoldásokat is biztosít.

**Összesített, és vesse össze a naplókat és mérőszámokat**. Egy megfelelően finomhangolt telemetriai rendszer biztosít nagy mennyiségű nyers feldolgozási adatokat és az eseménynaplókat. Győződjön meg arról, hogy a telemetria és naplózási adatok feldolgozását és korrelált egy rövid időn belül, hogy az üzemeltető személyzetnek mindig naprakész képet, a rendszer állapotát. Rendszerezheti és adatok megjelenítéséhez, amelyek valamilyen probléma javul nézet módokon, hogy, amikor csak lehetséges legyen egyértelmű egymáshoz kapcsolódó események során.

> Tekintse át a vállalati adatmegőrzési követelmények adatok feldolgozásának módja, és hogy mennyi ideig kell tárolni.

**Automatikus riasztások és értesítések valósítanak meg.** Állítsa be a monitorozási eszközökkel, például [Azure Monitor] [ azure-monitor] észlelheti a mintázatokat vagy feltételeket, amelyek a lehetséges vagy aktuális problémákra utalnak, és riasztások küldése a csapattagok számára, aki kezelheti a problémákat. A riasztások elkerülése érdekében a vakriasztások beállítása.

**A figyelő eszközök és források lejárhat.** Egyes források és eszközök, például a tanúsítványok, egy adott idő elteltével lejár. Ellenőrizze, hogy mely eszközök jár le, amikor lejár, és milyen szolgáltatások vagy funkciók függenek azokat nyomon követése. Automatizált folyamatok használatával ezek az eszközök figyelésére. Értesíti a műveletek team-eszköz lejárta előtt, és eszkalációjára, ha a lejárati fenyeget akadályozza az alkalmazás.

## <a name="management"></a>Kezelés

**Műveletek automatizálásához.** Sok hibalehetőséget rejtő manuális kezelése a folyamatok ismétlődő műveletek. Automatizálja ezeket a feladatokat, amikor csak lehetséges, konzisztens végrehajtása és a minőség biztosítása érdekében. Az automation megvalósító kódnak rendelkeznie kell verziószámmal verziókövetési rendszerben. Csakúgy, mint bármilyen más kód automatizálási eszközökkel kell vizsgálni.

**Az infrastruktúra mint kód megközelítés kiépítés igénybe vehet.** Manuális konfigurálás szükséges erőforrások kiépítése minimalizálására. Ehelyett használja a szkriptek és [Azure Resource Manager] [ resource-manager] sablonokat. Verziókövetés, mint bármely más olyan kódot, akkor ne a szkripteket és sablonokat.

**Fontolja meg a tárolók használatával.** Tárolók biztosítják a standard csomag felületen alkalmazások üzembe helyezéséhez. Tárolók használatával, egy alkalmazást helyezünk üzembe minden olyan szoftver, függőségeket és az alkalmazásról, így jelentősen leegyszerűsíti a telepítési folyamat futtatásához szükséges fájlokat tartalmazó önálló csomagok.

Tárolók is létrehozhat egy olyan absztrakciós réteget, az alkalmazás és az alapjául szolgáló operációs rendszerben, ami konzisztenciát biztosít a környezetek között. Ez az absztrakció is elkülönítheti a tároló más folyamatok vagy a gazdagépen futó alkalmazások.

**Rugalmasság és a fürtoperátoroknak valósítanak meg.** Rugalmasság rendszer azon képessége, egy alkalmazás való helyreállításhoz. Rugalmassági stratégiák újrapróbálkozás átmeneti hibák, és átadni a feladatokat egy másodlagos példány vagy akár egy másik régiót tartalmazzák. További információkért lásd: [rugalmas alkalmazások tervezése az Azure][resiliency]. Ezenkívül az alkalmazások, hogy azonnal jelentett problémák, és kezelheti leállások vagy más rendszer hibák.

**A műveletek manuális rendelkezik.** A műveletek manuális vagy *runbook* dokumentumok az eljárások és a felügyeleti információk szükséges műveletek személyzeti rendszer fenntartására. Is, dokumentum, minden olyan forgatókönyvek az operations és kockázatcsökkentési tervek, előfordulhat, hogy méretéből hiba vagy más megszakítás alatt a szolgáltatáshoz. Hozzon létre ebben a dokumentációban a fejlesztési folyamat során, és naprakészen, ezt követően. Ez egy élő dokumentumot és kell kell vizsgálni, tesztelt a rendszeresen.

Megosztott dokumentáció, kritikus fontosságú. Ösztönözze való közreműködés, és az ismeretek megosztása. A teljes csoport kell rendelkeznie a dokumentumokhoz való hozzáférésre. Megkönnyíti annak érdekében, hogy a frissített dokumentumokat a csapat mindenki számára.

**A dokumentum a készenléti eljárásokat.** Ellenőrizze, hogy a készenléti feladatkörök, ütemezését és eljárások dokumentált és az összes csapattagnak megosztott. Ez az információ mindig tartsa naprakészen a.

**A dokumentum a külső függőségekhez eszkalációs eljárások.** Ha az alkalmazás külső külső szolgáltatások, amelyek nem közvetlenül függ, rendelkeznie kell valamilyen okból kimaradás lép kezelésére egy tervet. Dokumentáció a tervezett megoldás-folyamatokhoz történő létrehozása. Támogatási kapcsolattartók és eszkalációs elérési utak közé tartozik.

**Használja a konfigurációkezelés.** Konfigurációmódosítás tervezett, a műveletek számára látható és a rögzített kell lennie. Ez eltarthat konfigurációkezelési adatbázis vagy a konfiguráció a kódot, megközelítést. Konfigurációs rendszeresen naplózni, győződjön meg arról, hogy mi az elvárás ténylegesen helyen.

**Lekérése egy Azure-támogatási csomagra, és megismerheti a folyamatot.** Az Azure számos [támogatási csomagok][azure-support-plans]. Az igényeinek megfelelő csomag határozza meg, és győződjön meg arról, hogy az egész csapat tudni fogja, hogyan használható a. A csapattagok ismerje meg a csomag részleteit, a támogatási folyamat működéséről, és hogyan nyithat egy támogatási jegyet az Azure-ral. A nagy léptékű esemény vannak várhatóan, ha az Azure-támogatás is segítséget nyújtanak a szolgáltatási korlátok növelését. További információkért lásd: a [Azure támogatás – gyakori kérdések](https://azure.microsoft.com/support/faq/).

**Minimális jogosultságon alapuló az alapelveket tartjuk, amikor erőforrásokhoz való hozzáférést.** Gondosan az erőforrásokhoz való hozzáférés kezelésére. Hozzáférés alapértelmezés szerint kell megtagadva, kivéve, ha egy felhasználó explicit módon való hozzáféréssel kap hozzáférést egy erőforráshoz. Csak hozzáférést egy felhasználó a számukra éppen szükséges feladatok elvégzéséhez. Nyomon követheti a felhasználói engedélyek, és hajtsa végre a rendszeres biztonsági eseményeket.

**Szerepköralapú hozzáférés-vezérlés használata.** Felhasználói fiókok és a hozzáférési erőforrások hozzárendelése nem lehet egy manuális folyamat. Használat [szerepköralapú hozzáférés-vezérlés] [ rbac] (RBAC) hozzáférés biztosítása alapján [Azure Active Directory] [ azure-ad] identitások és a csoportok.

**Követési rendszer hibát segítségével nyomon követheti a problémákat.** Anélkül, hogy hatékonyan nyomon követheti a problémákat hagyja ki az elemeket, munkahelyi ismétlődő vagy vezeti be a további problémák könnyebbé vált. Ne hagyatkozzon informális person-to-person kommunikációs hiba javítása állapotának nyomon követését. Problémák kapcsolatos adatokat rögzíti, és azok leküzdési erőforrásokat rendelhet hozzájuk, és adja meg annak előrehaladási és állapotmeghatározási auditnaplót egy hibakövetési eszköz használatával.

**Kezelheti a minden erőforrás egy módosítás rendszer.** A DevOps-folyamat minden szempontját szerepelnie kell egy felügyeleti és a verziókezelés rendszer, hogy a módosítások egyszerűen nyomon követi és naplózza. Ez magában foglalja a kódot, infrastruktúra, konfiguráció, dokumentáció és parancsfájlokat. Kezelje az összes ilyen típusú erőforrások kódokat terjesszen a tesztelési/build/ellenőrzés során.

**Ellenőrző listáinak használata.** Hozzon létre operations ellenőrzőlistákat biztosítása a folyamatok megfelelőségét. Gyakori valami nagy manuális konferenciát, és a következő ellenőrzőlista kényszerítheti a figyelmet a részletek, amelyek ellenkező esetben előfordulhat, hogy kihagyott. A Feladatlisták kezelése, és folyamatosan keresni a feladatok automatizálásához és egyszerűsíthetők a folyamatok.

További információ a Devopsról: [Mi az a DevOps?] [ what-is-devops] a Visual Studio-helyen.

<!-- links -->

[app-insights]: /azure/application-insights/
[azure-ad]: https://azure.microsoft.com/services/active-directory/
[azure-diagnostics]: /azure/monitoring-and-diagnostics/azure-diagnostics
[azure-monitor]: /azure/monitoring-and-diagnostics/monitoring-overview
[azure-support-plans]: https://azure.microsoft.com/support/plans/
[blue-green]: https://martinfowler.com/bliki/BlueGreenDeployment.html
[canary-release]:https://martinfowler.com/bliki/CanaryRelease.html
[dev-test]: https://azure.microsoft.com/solutions/dev-test/
[feature-toggles]: https://www.martinfowler.com/articles/feature-toggles.html
[oms]: https://www.microsoft.com/cloud-platform/operations-management-suite
[rbac]: /azure/active-directory/role-based-access-control-what-is
[resiliency]: ../resiliency/index.md
[resource-manager]: /azure/azure-resource-manager/
[trunk-based]: https://trunkbaseddevelopment.com/
[what-is-devops]: https://www.visualstudio.com/learn/what-is-devops/
