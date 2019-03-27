# <a name="using-tactical-ddd-to-design-microservices"></a>Mikroszolgáltatások tervezése taktikai DDD használatával

A stratégiai DDD fázisban ki az üzleti tartomány végzett, és definiálása, amelyet a tartománymodellek környezetét. Taktikai DDD van, a tartomány modellek nagyobb pontossággal meghatározása során. A taktikai minták körülhatárolt kontextus egy egyetlen belül érvényesek. A mikroszolgáltatási architektúrákban tudjuk különösen az entitás- és összesített minták iránt. Ezek a minták alkalmazásának segítségünkre lesz a természetes határokat a szolgáltatások, az alkalmazás azonosításához (lásd a [következő cikkre](./microservice-boundaries.md) sorozat). Általános szabály a mikroszolgáltatások nem kisebb, mint az összesítést, és nem haladhatja meg a körülhatárolt kontextus kell lennie. Első lépésként vizsgáljuk meg a taktikai minták. Ezután azt fogja azokat alkalmazni a korlátozott szállítási környezetben a Drone Delivery alkalmazást.

## <a name="overview-of-the-tactical-patterns"></a>A taktikai minták áttekintése

Ez a témakör röviden ismerteti a taktikai DDD mintáit, így ha már ismeri a DDD, valószínűleg ezt a szakaszt kihagyhatja. A minták fejezet 5 részletesen ismertetett &ndash; 6 Eric Evans címjegyzék-alkalmazásával, valamint a *Implementing Domain-Driven tervezési* Vaughn Vernon szerint.

![Taktikai tartományvezérelt tervezési mintákat bemutató ábra](../images/ddd-patterns.png)

**Entitások**. Egy entitás egy olyan objektum, amely továbbra is fennáll, idővel egyedi azonosítóval rendelkező. Ha például egy banki alkalmazás, ügyfelek és partnerek lenne entitások.

- Egy entitás egy egyedi azonosítót a rendszer, amely segítségével kereshet, vagy kérje le az entitást tartalmaz. Ez nem jelenti azt közvetlenül a felhasználóknak mindig megkapja a azonosítóját. Ez lehet egy GUID Azonosítót vagy egy elsődleges kulcs egy adatbázisban.
- Az identitás több kapcsolódó kontextusokat magában foglalhat, és előfordulhat, hogy rutinszerű túl az alkalmazás teljes élettartama. Például bankszámla számokat vagy a kormányzat által kibocsátott azonosítók vannak kötve egy adott alkalmazás teljes élettartama.
- Egy entitás attribútumai idővel változhat. Például egy személy nevét vagy címét változhat, de azok továbbra is ugyanazt az embert.
- Egy entitás tartalmazhat más entitásokra mutató hivatkozásokat.

**Objektumok érték**. Egy érték objektumnak nincs identitás. Csak a hozzá tartozó attribútumok értékeinek van definiálva. Érték objektumokat is nem módosíthatók. Érték objektum frissítése, mindig hozzon létre egy új példányt a régi helyett. Érték objektumoknak is lehetnek, amely magába foglalja az üzleti logikát módszereket, de azokat a módszereket az objektum állapota nem mellékhatásai legyen. Tipikus érték objektumok például színek, dátumok és időpontok és pénznemek.

**Összesítések**. Összesítés konzisztencia határ egy vagy több entitás körül határozza meg. Az összesítés pontosan egy entitás, a legfelső szintű. Keresési történik a gyökérszintű entitás-azonosítója használatával. Minden más entitás aggregált a legfelső szintű gyermek, és a legfelső szintű következő mutatók hivatkozik.

Az összesítés célja, hogy tranzakciós invariants modell. A való világból dolgot kell összetett webhelyek közötti kapcsolatok. Ügyfelek rendelések létrehozása rendelések tartalmazzák a termékek, termékek szállítók rendelkezik, és így tovább. Ha az alkalmazás módosítja a több kapcsolódó objektumot, hogy nem garantálja a konzisztenciát? Hogyan tudjuk invariants nyomon követheti, és kényszeríteni azokat?  

A hagyományos alkalmazások gyakran használt adatbázis-tranzakciók konzisztencia érvényesítéséhez. Az elosztott alkalmazásokban azonban ez gyakran nem megvalósítható. Egyetlen üzleti tranzakciók több adattárral, magában foglalhat, vagy előfordulhat, hogy hosszú ideig futó, vagy külső szolgáltatásokat is igénybe vehet. Végső soron van az alkalmazáshoz, az adatok réteg nem a szükséges tartomány invariants kényszerítésére. Ez milyen összesítések úgy van kialakítva, hogy a modell.

> [!NOTE]
> Összesítés tartalmazhatnak egyetlen entitást, gyermekentitások nélkül. Ez teszi összesítés a tranzakciós határt.

**Tartomány- és alkalmazásszolgáltatások**. A DDD-terminológiában a szolgáltatás olyan objektum, amely egy logikai bármely állapotban tartása nélkül. Evans különböztet *tartományi szolgáltatások*, amely magába foglalja az üzleti logikát, és *alkalmazásszolgáltatások*, biztosító műszaki funkciókat, például felhasználói hitelesítés és a egy SMS küldése az üzenet. Tartományi szolgáltatások gyakran használják az eszközmodell viselkedésének kiterjedő több entitás.

> [!NOTE]
> Az előfizetési időszak *szolgáltatás* szoftverfejlesztési túl van terhelve. Itt a definíció nem kapcsolódik közvetlenül mikroszolgáltatási.

**Tartományi események**. Tartományi események segítségével a rendszer egyéb részeivel értesítése, ha olyan esemény történik. Ahogy a neve is sugallja, tartományi események valamit a tartományon belül kell jelenti. Például "egy rekordot egy táblában illesztett" nem egy tartományi esemény. "Egy kézbesítési meg lett szakítva" az tartomány esemény. Tartományi események olyan gyakran a mikroszolgáltatási architektúrákban. Mikroszolgáltatás-alapú vannak osztva, és ne osszon adattárak, mert a tartomány események koordinálnak egymással a mikroszolgáltatások lehetőséget biztosíthat. A cikk [szolgáltatások közötti kommunikáció](../design/interservice-communication.md) aszinkron üzenetkezelést részletesebben ismerteti.

Nincsenek néhány egyéb DDD minták nem szerepel itt, beleértve az előállítók, az adattárak és a modulok. Ezek lehet hasznos, amikor azt fontolgatja, mikroszolgáltatások mintákat, de kevésbé fontos a határok között mikroszolgáltatások tervezése során.

## <a name="drone-delivery-applying-the-patterns"></a>Drone delivery: A minták alkalmazásának

Kezdődik meg, amely a szállítási határolt környezetet kell kezelni a forgatókönyvekkel.

- Egy ügyfél egy termék drónos áru, egy drónos szállítási szolgáltatásra regisztrált üzleti kérhetnek.
- A küldő egy címkét (vonalkód vagy RFID) helyezi a csomagot hoz létre.
- Egy drónt folytattuk a munkát, és a egy csomag forráshelyéről továbbítására a célhelyen.
- Amikor az ügyfél ütemezi a kézbesítési, akkor a rendszer egy útvonal-információkat, az időjárási viszonyok és az előzményadatok alapján DRÓN biztosít.
- Ha a drone útban, a felhasználó nyomon követheti az aktuális hely és a legújabb DRÓN.
- Mindaddig, amíg egy drónt rendelkezik mértékének a csomag, az ügyfél egy kézbesítési vonhatja vissza.
- Az ügyfél értesítést kap a kézbesítési befejezésekor.
- A küldő kézbesítési megerősítő használói kérhetnek az ügyfél egy aláírás vagy az ujját nyomtatási formájában.
- Felhasználók kereshet egy befejezett szállítás előzményei.

Az ezekben az esetekben a fejlesztői csapat az alábbi azonosított **entitások**.

- Kézbesítés
- Csomag
- Drónos
- Fiók
- Megerősítés
- Értesítés
- Címke

Az első négy, Tartalomkézbesítési, csomag, Drónos fiók jsou összes **összesítések** , amelyek az tranzakció-konzisztencia határokat. Megerősítések és értesítések gyermekentitások a szállítások, és a címkék olyan gyermekentitások csomagok.

A **érték-objektumok** Ez a kialakítás tartalmazzák a helyet, DRÓN, PackageWeight és PackageSize.

Mutatja be, ez a kézbesítési összesítés UML diagramját. Figyelje meg, hogy rendelkezik-e más összesítéseket, beleértve a fiókot, a csomag és a Drone mutató hivatkozásokat.

![A kézbesítési összesítés UML ábrája](../images/delivery-entity.png)

Nincsenek a két tartomány esemény:

- Bár egy drónt útban, a Drone entitás küld DroneStatus események, amelyek ismertetik a drón helyét és állapotát (átvitel közben, szállítási).

- A kézbesítési entitás DeliveryTracking eseményeket, amikor módosul egy szállítási küld. Ezek közé tartozik a DeliveryCreated, DeliveryRescheduled, DeliveryHeadedToDropoff és DeliveryCompleted.

Figyelje meg, hogy ezek az események leírása dolog, amelyek fontosak lehetnek a tartomány modellen belül. Adja meg valamit a tartományhoz, és nem kapcsolódik egy adott programozási nyelv szerkezet.

A fejlesztői csapat egy további terület, amely nem fér el eligazíthatja bármelyikével az eddig leírt azonosítani. A rendszer egyes részei kell koordinálja az összes ütemezés vagy egy tartalomkézbesítési frissítése lépéseit. Ezért a fejlesztői csapat hozzáadott két **tartományi szolgáltatások** terve: egy *Scheduler* , hogy koordinálja a lépéseket, és a egy *felügyelő* , amely figyeli az egyes állapota lépés annak észleléséhez, hogy olyan lépéseket a sikertelen vagy túllépte az időkorlátot. Ez a kapcsolat egy változata a [Feladatütemező ügynök felügyeleti mintájának](../../patterns/scheduler-agent-supervisor.md).

![A módosított tartományi modellben ábrája](../images/drone-ddd.png)

## <a name="next-steps"></a>További lépések

A következő lépés az egyes mikroszolgáltatások. Ehhez adja meg.

> [!div class="nextstepaction"]
> [Mikroszolgáltatások határainak azonosítása](./microservice-boundaries.md)