# <a name="devops-checklist"></a>DevOps ellenőrzőlista

DevOps, az integráció fejlesztési, minőségi megbízhatósági és egy egyesített kulturális környezet az IT-üzemeltetők és a folyamatok szoftver kézbesítéséhez készletét. 

Kiindulási pontként az alábbi ellenőrzőlista segítségével mérje fel a DevOps kulturális környezet és a folyamat.

## <a name="culture"></a>Kulturális környezet

**Győződjön meg arról üzleti pozicionálását szervezetek és csoportjai között.** Erőforrások, célja, célok és prioritások szervezeten belüli ütközések sikeres műveletek kockázatot is lehet. Győződjön meg arról, hogy az üzleti, fejlesztési és műveletek közötti összes igazított.

**Győződjön meg arról, a teljes csoport megértette a szoftver életciklusát.** A csapat kell az alkalmazás a teljes felügyeleti életciklus megismerése, és melyik része a életciklusa az alkalmazás jelenleg a. Ezzel a megoldással akkor érdemes lehet tevékenységeit most, és mit kell kell tervezése és előkészítése a jövőben ismeri az összes csoport tagjai.

**Ciklus idejének csökkentése érdekében.** Cél használható fejlesztésű szoftver ötleteket áthelyezéséhez szükséges idő minimalizálása érdekében. A méret és a hatókör tartani a teszt terheket alacsony egyes kiadásainak korlátozza. Automatizálhatja az összeállítási, tesztelési, konfigurációs és központi telepítési folyamatok, amikor csak lehetséges. Törölje a jelet minden akadályt fejlesztők között, és a fejlesztők és a műveletek közötti kommunikációra. 

**Tekintse át, és növeli a folyamatokat.** A folyamatok és eljárások, akár automatikusan, manuális, amelyek soha nem végleges. Állítsa be aktuális munkafolyamatok, eljárások és dokumentáció, azzal a céllal, folyamatos javítása rendszeres értékelést.

**Tegye proaktív tervezési.** Proaktív tervezze meg a hiba. Gyorsan azonosíthatja a problémákat, ha fordulhat elő, javítsa ki a megfelelő csoport tagjai számára az ilyen, valamint névfeloldási megerősítése folyamatok rendelkezik.

**Ismerje meg a hibákat.** Hibák előbb vagy utóbb elkerülhetetlenné, de fontos, hogy megtudjuk hibák őket ismétlődésének elkerülése érdekében. Működési hiba akkor fordul elő, ha a probléma, a dokumentum a OK és megoldás osztályozhatja, és bármely során tapasztalatokat, hogy megismerte volt osztja. Amikor csak lehetséges, frissítse a build folyamatok automatikus észlelése az adott típusú hiba a jövőben.

**Optimalizálás sebességre és adatainak gyűjtéséről.** Minden tervezett fokozása egy alapul. A lehető legkisebb lépésekben működik. Új ötleteket tekinti kísérletekben. A kísérletek állíthatnak be, hogy a termelési adatok ellenőrzéséhez hatékonyságuk hozhatja létre. Készüljön sikertelen gyors, ha probléma a alapul.

**Learning várja.** Hibák és a sikeres adja meg a tanulási jó lehetőségeit. Mielőtt új projektek, gyűjtsön során a fontos tapasztalatokat, és ellenőrizze, hogy ezeket a megszerzett használja fel, a csapat elegendő időt engedélyezése. Is meg kell adni a csoport létrehozásához képességek, kísérletezhet, és további információk az új eszközöket és technikákat. 

**A dokumentum műveletek.** A dokumentum minden eszközök, folyamatok és automatizált feladatok a termékkódot a minőségi szintjét. Dokumentálja az aktuális terv és azok a rendszerek támogatásához, valamint helyreállítási folyamatokat és az egyéb karbantartási eljárások architektúrájának. A lépéseket, ténylegesen végrehajtaná, elméletileg optimális folyamatok összpontosítanak. Rendszeresen tekintse át, és frissítse a dokumentációt. Kód győződjön meg arról, az ésszerűen megjegyzéseket, különösen a nyilvános API felületeket, és eszközök segítségével amikor csak lehetséges dokumentációja automatikus létrehozása. 

**Ossza meg a Tudásbázis.** Dokumentáció csak akkor hasznos, ha személyek tudja, hogy létezik-e, és találja meg. Ellenőrizze a dokumentációt rendszerezett és könnyen felfedezhetővé. Legyen kreatív: részekre (nem hivatalos bemutatók), videók, és a hírlevelek, megoszthatja a Tudásbázis segítségével.

## <a name="development"></a>Fejlesztés

**Adja meg a fejlesztők hasonló környezetekhez.** Fejlesztési és tesztelési környezetek nem egyeznek az éles környezetben, akkor annak tesztelése, és a problémák diagnosztizálásához. Ezért fejlesztési tartása, és tesztelése az éles környezetben a lehető hamarosan környezetekhez. Győződjön meg arról, hogy vizsgálati adatok konzisztensek legyenek a, éles környezetben használt adatok akkor is, ha a mintaadatok, és nem tényleges termelési adatok (adatvédelmi vagy megfelelőségi okokból). Tervezze meg szeretnék létrehozni és mintaadatokat teszt anonimizálásához.

**Győződjön meg arról, hogy minden jogosult csoport tagjai infrastruktúra kiépítéséhez, és telepítse központilag az alkalmazást.** Hasonló erőforrások beállítása és az alkalmazás telepítése nem terjed ki bonyolult manuális feladatok vagy részletes technikai tudásszintjük, a rendszer. Bárki, aki a megfelelő engedélyekkel kell létrehozni és hasonló erőforrások telepítése nélkül a műveletek csapathoz. 

> Ez a javaslat nem jelenti azt, hogy bárki tolható működés közbeni frissítések az éles környezet. Az ismertető csökkentése súrlódás a fejlesztési és QA csapatok hasonló környezetek létrehozása.

**Az alkalmazás betekintés a állíthatnak be.** Tudni, hogy az alkalmazás állapotát, hogy teljesítményét, és hogy kapcsolatos esetleges hibák vagy problémák tapasztalhatók kell. Mindig kialakítási követelmények instrumentation tartalmazza, és állítsa be az alkalmazásba a kezdetektől instrumentation. Eseménynaplózás a kiváltó okának elemzése, de is telemetriai adatok és metrikák általános állapotát és az alkalmazást figyeléséhez Instrumentation tartalmaznia kell.

**A műszaki hitelviszonyt nyomon.** Sok projektek kiadás ütemezéseket is beolvasása előrébb, mértékű vagy egy másik kódot minőségi keresztül. Mindig kísérje figyelemmel, ha ez történik. Dokumentálja a parancsikonok vagy más nonoptimal esetében, és ütemezés szerinti időt a jövőben le újra ezeket a problémákat.

**Vegye figyelembe, hogy közvetlenül az éles frissítések terjesztése.** A teljes kiadás ciklusidő csökkentése érdekében fontolja meg a kódot véglegesítése közvetlenül a termelési kérdez le megfelelően tesztelte. Használjon [váltógombok funkció] [ feature-toggles] szabályozásához, hogy mely szolgáltatások engedélyezve vannak. Ez lehetővé teszi, hogy fejlesztési áthelyezését gyorsan, a kibocsátási váltógombok segítségével engedélyezheti vagy tilthatja le a szolgáltatásokat. Be-és kikapcsolása is akkor hasznosak, ha például a tesztek végrehajtása [Kanári kiadásokban][canary-release], amelyen üzembe van helyezve egy adott szolgáltatással az éles környezetben egy részét.

## <a name="testing"></a>Tesztelés

**Automatizálható a teszteléshez.** Manuális tesztelési szoftver fárasztó és ki vannak téve a hiba. Gyakori tesztelési feladatok automatizálásához, és a tesztek integrálja a build folyamatokat. Egységes teszt lefedettsége és reprodukálhatósági automatikus tesztelést biztosítja. Integrált felhasználói felület vizsgálatokat is egy automatizált eszköz által hajtható végre. Azure kínál a fejlesztési és a teszt erőforrásokat, amelyek segítségével konfigurálhatja, és végrehajtása a tesztelés. További információkért lásd: [fejlesztési és tesztelési][dev-test].

**Ellenőrizze a hibákat.** Ha a rendszer a szolgáltatás nem tud csatlakozni, hogy azt válaszol? Az helyreállíthatja után a szolgáltatás nem érhető el újra? Ellenőrizze a hiba injektálási tekintse át a teszt szokásos részét tesztelési és átmeneti környezet. Ha a tesztelési folyamattal és eljárások érett, vegye figyelembe, éles környezetben ezek a tesztek futtatása. 

**Éles üzemi környezetben.** A kiadási folyamat éles környezetbe való telepítés nem végződhet. Győződjön meg arról, hogy telepített kód megfelelően működik-e tesztek rendelkezik. Ritkán frissített telepítések esetén a karbantartás során tesztelés üzemi ütemezni.

**Teljesítménnyel kapcsolatos problémák azonosításához korai teljesítménytesztelési automatizálásához.** Lehet, hogy csak a súlyos hiba a kódban, súlyos teljesítményprobléma hatását. Automatikus működési tesztek megakadályozhatja az alkalmazás-hibák, amíg azok nem előfordulhat, hogy észleli teljesítményproblémákat. Adja meg például késés, a betöltési idők és erőforrás-használati metrikák elfogadható teljesítmény célokhoz. Vegye fel a kiadás csővezetéket, győződjön meg arról, hogy az alkalmazás megfelel-e ezeket a célokat automatizált teljesítménytesztek.

**Kapacitás tesztek végrehajtása.** Az alkalmazások előfordulhat, hogy vizsgálati körülmények jól működnek, és majd problémába ütközik méretezési vagy erőforrás-korlátozások miatt éles környezetben. Mindig adja meg a maximális várt kapacitás és a használati korlátok. Ellenőrizze, hogy ellenőrizze, hogy az alkalmazás ezen korlátok kezelni, de is tesztelni, mi történik, ha ezeket a határértékeket. A kapacitás tesztelés rendszeres időközönként kell elvégezni.

Kezdeti megjelenése után futtassa a teljesítmény és kapacitás tesztek amikor éles kódban végrehajtott frissítések. Konfigurálva finomhangolhatják a tesztek és határozza meg a tesztek típusú kell végezni a korábbi adatok.

**Az automatikus biztonsági behatolást vagy a biztonság tesztek végrehajtása.** Annak biztosításában, az alkalmazás biztonságos olyan fontos, mint bármely más funkció tesztelése. Ellenőrizze a automatizált behatolást vagy a biztonság tesztelése a build és a telepítési folyamat a szabványos része. Adja meg a rendszeres biztonsági vizsgálatok és biztonsági rések keresése a központilag telepített alkalmazások figyelése a megnyitott portok, a végpontok és a támadások. Automatizált tesztelése nem távolítja el mélyreható biztonsági ellenőrzések rendszeres időközönként van szükség.

**Automatizált üzleti folytonossági tesztek végrehajtása.** Nagy méretű az üzletmenet folytonossága, beleértve a biztonsági mentési helyreállítási és a feladatátvételi teszt fejlesztéséhez. Állítsa be automatikus folyamatot rendszeresen ezek a tesztek végrehajtásához.

## <a name="release"></a>Kiadás

**Központi telepítések automatizálásához.** Tesztelje az alkalmazás központi telepítését, átmeneti és éles környezetek automatizálásához. Automatizálás lehetővé teszi, hogy gyorsabb és megbízhatóbb központi telepítések, és biztosítja az egységes központi telepítésére bármely támogatott környezetben. Eltávolítja a manuális központi telepítések által okozott emberi tévedések kockázatát. Azt is könnyen alkalmas időpontban, semmilyen hatása várható állásidő minimalizálása érdekében a kiadások ütemezni.

**Használja a folyamatos integrációt.** Folyamatos integrációt (CI) egyesítése egy központi kódot összes fejlesztői kódbázis rendszeres időközönként, és automatikusan végrehajtásával szabványos összeállítását és a test-eljárások. CI biztosítja, hogy egy teljes csoport is dolgozhatnak codebase ugyanakkor anélkül, hogy ütközések. Emellett biztosítja, hogy a kód hibákat a lehető leghamarabb találnak. A CI folyamat lehetőleg, minden alkalommal, amikor kódot véglegesítve lett, vagy be van jelölve, kell futtatnia. Legalább a rendszer naponta egyszer indít lefuttassa.

> Vegye figyelembe a bevezetése egy [trönk alapú fejlesztési modell][trunk-based]. Ebben a modellben fejlesztők véglegesítése egy fiókirodának (törzsének). Követelmény, hogy véglegesíti soha nem törhetik meg a build van. Ez a modell elősegíti a CI, mivel az összes szolgáltatás végezhető el a trönk, és a véglegesítés történik, amikor az egyesítési ütközések feloldása.

**Érdemes lehet folyamatos kézbesítését.** Folyamatos kézbesítési (CD), amely kód mindig automatikusan létrehozása, tesztelése és hasonló környezetekben való központi telepítésének kód üzembe helyezésére ajánlott annak biztosítására, hogy. Hozzon létre egy teljes CI/CD folyamat folyamatos kézbesítése hozzáadása segít a kód amint lehetséges hibák, és biztosítja, hogy megfelelően tesztelt frissítések nagyon rövid időn belül kell szabadítani észlelése.

> Folyamatos *telepítési* egy folyamat, amely automatikusan a frissítéseket a CI/CD-feldolgozási folyamaton keresztül továbbított fogad, és telepíti azokat éles környezetben. Folyamatos üzembe helyezés robusztus automatikus tesztelése és speciális folyamat tervezési igényel, és nem lehet megfelelő összes csoportjai.

**Ügyeljen a kis a növekményes változásokat.** Nagy kódmódosításokat rendelkezik egy nagyobb lehetséges hibák bevezetése. Amikor csak lehetséges – maradjon kicsi módosításokat. Korlátozza a lehetséges hatások minden változás, és megkönnyíti annak megértéséhez, valamint debug probléma merül fel.

**Változások szabályozzák.** Gondoskodjon arról, hogy mikor frissítései láthatóak a végfelhasználók számára feletti. Fontolja meg, a szolgáltatás váltógombok annak vezérléséhez, ha a szolgáltatások engedélyezve vannak a végfelhasználók számára.

**Alkalmazzon kiadásban felügyeleti stratégiák telepítési kockázat csökkentése érdekében.** Néhány kockázat központi telepítése egy alkalmazás frissítése üzemi mindig terjed ki. A kockázat minimalizálása érdekében használjon stratégiák például [Kanári kiadásokban] [ canary-release] vagy [kék-zöld központi telepítések] [ blue-green] telepítheti a frissítéseket egy részét a felhasználók. Erősítse meg, a frissítés megfelelően működik-e, és ezután megkezdik a frissítés a rendszer a többi.

**A dokumentum összes módosítását.** Kisebb frissítéseket és a konfigurációs módosítások félreértések és versioning ütköző forrása lehet. Mindig nyilvántarthatja törölje a jelet a módosításokat, függetlenül attól, milyen kicsi. Minden, ami változik, beleértve az alkalmazott javítások, a házirend módosításai és a konfigurációs módosítások naplózása. (Nem az olyan bizalmas adatok ezekben a naplókban. Például, hogy a hitelesítő adatok frissítve lett, és ki a módosítást, de ne jegyezze fel a frissített hitelesítő adatok napló.) A rekord a változások kell lennie a teljes csoport számára látható. 

**Központi telepítések automatizálásához.** Összes központi telepítésének automatizálása, és a bevezetés alatt talált hibát rendszerek rendelkezik. Ha meg kell őrizni a meglévő kódot és az adatok üzemben, előtt a frissítés cseréli le, azok összes éles példányát megoldás eljárással rendelkezik. Automatikus módszere a továbbítás javítások állíthatja, illetve visszaállítható a változás rendelkezik.

**Vegye figyelembe, hogy infrastruktúra nem módosítható.** Nem módosítható infrastruktúra az elven, hogy ne módosítsa infrastruktúra üzemi telepítés után. Ellenkező esetben kaphat olyan állapotba, ahol az alkalmi módosítások történtek, így nehezen pontosan mit változásairól. Nem módosítható infrastruktúra működését tekintve a teljes kiszolgálók cseréjekor minden új központi telepítésének részeként. Ez lehetővé teszi a kódot és az üzemeltetési környezetben tesztelése és telepítése és a blokk. Amennyiben telepített, infrastruktúra-összetevőihez nem módosította, amíg és ciklus üzembe helyezheti a Tovább gombra. 

## <a name="monitoring"></a>Figyelés

**Ellenőrizze a rendszer megfigyelhető.** A műveleti csapata mindig meg kell adni egyértelmű lássák a rendszer vagy a szolgáltatás állapotát. Külső állapotfigyelő végpontok beállítása állapotának figyelésére, és győződjön meg arról, hogy alkalmazásokat állíthatnak be a műveletek metrikák van kódolva. Egy általános és következetes séma, amely lehetővé teszi az események összefüggéseket rendszeren használható. [Az Azure Diagnostics] [ azure-diagnostics] és [Application Insights] [ app-insights] a szabványos használják az Azure-erőforrások állapotát nyomon követését. Microsoft [művelet felügyeleti csomag] [ oms] is biztosít a központosított figyelése és a felhőalapú vagy hibrid megoldások felügyelete.

**Összesítő és a naplók és a metrikák összefüggéseket**. Egy megfelelően felműszerezett telemetriai rendszer nagy mennyiségű nyers Teljesítményadatok és eseménynaplók ad meg. Győződjön meg arról, hogy telemetriai adatok és naplózási adatok feldolgozása és egy rövid idő alatt, korrelált, hogy a műveleti személyzet mindig naprakész képet a helyrendszer állapotát. Rendszerezése és jeleníti meg az adatokat, melyek egy javul betekintést nyújt probléma merül fel, így amikor csak lehetséges törölje a jelet amikor egymáshoz kapcsolódó események.

> Tekintse meg a vállalati adatmegőrzési követelmények adatok feldolgozásának módja, és mennyi ideig kell tárolni. 

**Automatizált riasztások és értesítések megvalósításához.** Eszközök, például a figyelés beállítása [Azure figyelő] [ azure-monitor] ismeri fel a szabályszerűségeket vagy a feltételeket, amelyek lehetséges vagy aktuális problémákra utalnak, és riasztást küldjön a csapattagok számára, akik az problémákat is. A vakriasztások elkerülése érdekében a riasztások hangolása.

**Eszközök és erőforrások lejáratok a figyelheti.** Bizonyos erőforrások és az eszközök, például a tanúsítványok, a megadott idő elteltével lejár. Győződjön meg arról, hogy mely eszközök jár le, amikor lejár, és milyen szolgáltatások vagy szolgáltatások függő személyektől nyomon. Ezek az eszközök felügyeletére automatizált folyamatok használni. Értesíti a műveletek egy eszköz lejárta előtt vonja össze, és eszkalálása, ha az alkalmazás zavarná fenyeget lejárati.

## <a name="management"></a>Kezelés

**Műveletek automatizálásához.** Ismétlődő műveletek folyamatok manuálisan kezelése hibákhoz vezethet. Mindig konzisztens végrehajtása és a minőségét a feladatok automatizálásához. Az automatizálás megvalósító kódot verziókezelő a rendszerverzióval ellátott kell lennie. Akárcsak bármely más kódot automatizálási eszközökkel kell vizsgálni.

**Kiépítés igénybe vehet egy infrastruktúra-,-kód-módszert is.** A lehető legkevesebb manuális konfiguráció szükséges erőforrások biztosításához. Ehelyett használja a parancsfájlok és [Azure Resource Manager] [ resource-manager] sablonok. A verziókövetési rendszerrel, mint bármely más kódot, akkor ne a parancsfájlok és sablonok. 

**Érdemes lehet tárolók.** Tárolók egy standard csomag-alapú felületet biztosít az alkalmazások központi telepítéséhez. Tárolók használ, egy alkalmazás lett telepítve, amely tartalmazza az összes szoftver, függőségeket és az alkalmazásról, így jelentősen egyszerűbb a telepítési folyamat futtatásához szükséges fájlok önálló csomagok használata. 

Tárolók is létrehozhat egy hardverabsztrakciós réteg között az alkalmazás és az alapjául szolgáló operációs rendszer, így konzisztencia környezetek között. Ez az absztrakció is különítse el a más folyamatok vagy a gazdagépen futó alkalmazások egy tárolót. 

**Rugalmasság és self-healing megvalósításához.** Rugalmasságra az alkalmazás helyreállni hibák után képességét. Rugalmasság kapcsolatos olyan stratégiák közé tartozik a újrapróbálkozás átmeneti hibák és a feladatátadás a másodlagos példány vagy akár egy másik régióban. További információkért lásd: [rugalmas alkalmazások az Azure-kialakítása][resiliency]. Az alkalmazások állíthatnak be, hogy azonnal jelentett problémák és kezelheti a kimaradások vagy más rendszerhiba.

**Egy műveletek kézi rendelkezik.** Az operations manuális vagy *runbook* dokumentumok eljárások és a rendszer fenntartására személyzeti műveletek szükséges felügyeleti adatokat. Bármely műveletek forgatókönyvek és a megoldás terveket, amelyek esetleg play hiba vagy más szüneteltetése során a szolgáltatás is szerepelnek. Hozzon létre ebben a dokumentációban a fejlesztés során, és azt naprakészen tartása ezt követően. Ez élő dokumentumok, és kell kell vizsgálni, tesztelése és továbbfejlesztett rendszeresen. 

Megosztott dokumentáció alapvető fontosságú. Ösztönözze közre és oszthat csoport tagjai. A teljes csoport kell rendelkeznie a dokumentumokhoz való hozzáférést. Könnyen bárki a csapat frissített dokumentumok megőrzése érdekében.

**A dokumentum a készenléti eljárásokat.** Ellenőrizze, hogy a készenléti feladatokat, ütemezéseihez és eljárások dokumentált és megosztott összes csapattagok számára. Ezek az információk naprakészen tartsa mindig.

**A dokumentum a külső függőségekhez eszkalációs eljárások.** Ha az alkalmazás külső külső-szolgáltatások, amelyek nem közvetlenül függ, rendelkeznie kell egy tervet kimaradások kezelésére. A tervezett megoldás folyamatok dokumentáció létrehozása. Támogatási kapcsolattartók és eszkalációs útvonalak lehetnek.

**Konfigurációkezelés használja.** Konfigurációs módosítások tervezett, műveletek számára látható és rögzített kell lennie. Ez a konfigurációkezelési adatbázis, vagy a konfiguráció szerint kód megközelítés formában is beletelhet. Konfigurációs rendszeresen naplózni annak biztosításához, hogy mi az elvárás ténylegesen helyen.

**Az Azure támogatási terv és a folyamatához.** Azure számos [tervek támogatja][azure-support-plans]. Az Ön igényeinek megfelelő csomag határozza meg, és győződjön meg arról, hogy a teljes csoport tudja, hogy miképpen lehet vele. Csoporttagok ismerje meg a csomag részletes adatait, a támogatási folyamat működéséről és az Azure támogatási jegy megnyitása. Ha egy magas szintű esemény vannak előrejelző, az Azure támogatási segíthet a növelését a szolgáltatásra vonatkozó korlátozások. További információkért lásd: a [Azure támogatás – gyakori kérdések](https://azure.microsoft.com/en-us/support/faq/).

**Kövesse a legalacsonyabb jogosultsági alapelvek, amikor erőforrásokhoz való hozzáférés.** Gondosan erőforrásokhoz való hozzáférés kezelése. Hozzáférés alapértelmezés szerint nem kap, kivéve, ha a felhasználó explicit módon kap hozzáférést egy erőforráshoz. Csak hozzáférést egy felhasználó szükséges el tudják végezni feladataikat. A felhasználói engedélyek nyomon, és végezze el a rendszeres biztonsági eseményeket.

**Szerepköralapú hozzáférés-vezérlés használatára.** Felhasználói fiókok és a hozzáférés hozzárendelése erőforrásokhoz nem lehet egy manuális folyamat. Használjon [szerepköralapú hozzáférés-vezérlés] [ rbac] (RBAC) hozzáférést alapján [Azure Active Directory] [ azure-ad] identitások és csoportok. 

**Nyomkövetési rendszer programhiba segítségével nyomon követheti a problémákat.** Egy jó módszer problémák nyomon követésére, nélkül is könnyen teljesíti az elemeket, munkahelyi ismétlődő vagy bevezetni a további problémák. Ne hagyatkozzon hibák állapotának nyomon követése nem hivatalos person-to-person kommunikációt. Egy eszköz követési hiba segítségével problémák kapcsolatos adatokat rögzíti, rendeljen hozzájuk erőforrásokat, és adja meg a folyamatban lévő és állapota egy naplóban. 

**Minden az erőforrások kezelése a változás-kezelési rendszerbe.** A DevOps folyamat minden aspektusát szerepelnie kell egy felügyeleti és verziókezelő rendszer, így a módosításokat is könnyen nyomon követése és naplózása. Ez magában foglalja a kódot, infrastruktúra, konfigurációs, dokumentáció és parancsfájlok. Minden ilyen típusú erőforrások kezelje a teszt/build/felülvizsgálati folyamat során kódot. 

**Ellenőrzőlista használata.** Hozzon létre műveletek ellenőrzőlisták folyamatok követi biztosításához. Általános valamit a nagy kézi, amelyeken a, és a következő ellenőrzőlista kényszerítheti a figyelmet, ellenkező esetben előfordulhat, hogy kihagyott adatokat. A Feladatlisták kezelése, és folyamatosan keressen a feladatok automatizálásához és folyamatok egyszerűsítésére módszereket.

DevOps kapcsolatban bővebben lásd: [DevOps újdonságai?] [ what-is-devops] a Visual Studio helyen.

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
