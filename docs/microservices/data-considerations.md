---
title: Mikroszolgáltatások adatok szempontjai
description: Mikroszolgáltatások adatok szempontjai
author: MikeWasson
ms.date: 12/08/2017
ms.openlocfilehash: 9bd7a8424309b5753b42cfb70559836288a3ce9d
ms.sourcegitcommit: c7f46b14ad7d55cf553b2d0b01045c9c25aefcdb
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/09/2017
ms.locfileid: "26587756"
---
# <a name="designing-microservices-data-considerations"></a>Mikroszolgáltatások tervezése: adatok kapcsolatos szempontok

Ez a fejezet mikroszolgáltatások architektúra adatok kezelésével kapcsolatos szempontokat ismerteti. Minden mikroszolgáltatási kezeli a saját adatok, mert adatintegritást és az adatok konzisztenciájának áttekinthet kritikus.

![](./images/data-considerations.png)

Egy alapszintű mikroszolgáltatások lényege, hogy minden egyes szolgáltatás kezeli a saját adatok. Két szolgáltatást nem kell ugyanazt a tárolóban. Ehelyett minden a szolgáltatás felelős a saját személyes adattároló, amely más szolgáltatások nem tud közvetlenül hozzáférni.

Ez a szabály oka közötti szolgáltatásokat, amelyek okozhat, ha a szolgáltatások megosztani az azonos alapjául szolgáló adatok sémák nem szándékos kapcsoló elkerülése érdekében. Ha az adatok séma módosítva lett, a módosítás minden szolgáltatásban, amely támaszkodik, hogy az adatbázis össze kell hangolni. Minden szolgáltatás adattár elkülönítésével azt is korlátozza a módosítás következményeivel, és valóban független központi telepítések agilitást megőrzése. Egy másik oka az, hogy minden egyes mikroszolgáltatási előfordulhat, hogy a saját adatok modellek, a lekérdezések, vagy olvasási/írási kombinációját. Minden team képességét az adott szolgáltatáshoz adattárolás optimalizálása érdekében a megosztott tároló használata korlátozza. 

![](../guide/architecture-styles/images/cqrs-microservices-wrong.png)

Ezt a megközelítést vezet természetes [polyglot adatmegőrzési](https://martinfowler.com/bliki/PolyglotPersistence.html) &mdash; több adatok tárolási technológiákat belül egyetlen alkalmazás használatát. Egy szolgáltatási dokumentum-adatbázis a séma-a-olvasási képességek lehet szükség. Egy másik módosítania kell a hivatkozási integritás egy RDBMS által biztosított. Minden csoport, így a szolgáltatás a legjobb választás. Az általános elvet polyglot adatmegőrzési kapcsolatban bővebben lásd: [használjon a legjobb adatok tárolásához a feladat](../guide/design-principles/use-the-best-data-store.md). 

> [!NOTE]
> Azt megosztása ugyanazon fizikai adatbázis-kiszolgáló szolgáltatások rendben. A probléma akkor fordul elő, amikor szolgáltatások megosztása ugyanazon séma, vagy olvasási és írási ugyanazokat a adatbázistáblák.


## <a name="challenges"></a>Kihívásai

Ez a megközelítés elosztott némi kihívást adatok kezelésének merülhetnek fel. Első lépésként lehet redundancia az adattároló, az azonos elem több helyen jelennek meg adatok között. Például adatok előfordulhat, hogy lehet egy tranzakció részeként tárolja, majd az elemzés, reporting, vagy a máshol tárolt. Ismétlődik vagy particionált adatok adatintegritást és konzisztencia vezethet. Adatok kapcsolatok span több szolgáltatásra, amikor a hagyományos technikák kényszeríteni a kapcsolatok nem használhat.

Hagyományos modellezési használja a szabály az "egy kapcsolattény egyetlen helyen." Minden entitás pontosan egyszer jelenik meg a sémában. Egyéb entitások hivatkoznak rá tárolására, de nem ismétlődő. A nyilvánvaló a hagyományos megközelítés előnye, hogy frissítések válnak, egyetlen helyen, és elkerülhetők a problémák az adatok konzisztenciájának. Mikroszolgáltatások architektúra esetén meg kell figyelembe venni, hogyan frissítések továbbítódnak szolgáltatásban, és kezelése a végleges konzisztencia, amikor adatok nélkül az erős konzisztencia több helyen jelennek meg. 

## <a name="approaches-to-managing-data"></a>Az adatkezelés megközelítése

Nincs egyetlen módszert alkalmaz, amely minden esetben helyes van, de az alábbiakban néhány általános irányelveket mikroszolgáltatások architektúra adatok kezeléséhez.

- Vezessék be a végleges konzisztencia, ahol csak lehetséges. Ismerje meg a helyek, a rendszer, ahol az erős konzisztencia vagy ACID-tranzakciókat és a helyeken, ahol a végleges konzisztencia elfogadható kell.

- Ha módosítania kell az erős konzisztencia biztosítja, egy szolgáltatás jelöl egy adott entitás, amely fel van fedve egy API-n keresztül igazság forrását. Egyéb szolgáltatások előfordulhat, hogy rendelkezik, saját az adatok másolatát, vagy az adatok, idővel konzisztenssé váljanak a törzsadatok, de nem tekinthető igazság forrását egy részét. Tegyük fel az elektronikus kereskedelmi rendszer rendelés ügyfélszolgálata és a javaslási szolgáltatása. A javaslat szolgáltatás események sorrendben szolgáltatásból előfordulhat, hogy figyelésére, de az ügyfél a vételár kérelmek esetén a rendelés szolgáltatás, nem a javaslat szolgáltatást, a teljes tranzakció előzmények.

- Az egyes tranzakciókra vonatkozóan, használjon minták például [Feladatütemező ügynök felügyelő](../patterns/scheduler-agent-supervisor.md) és [Compensating tranzakció](../patterns/compensating-transaction.md) és konzisztens adatok több szolgáltatásban.  Előfordulhat, hogy kell tárolni egy további adat, amely több szolgáltatásra, több szolgáltatás között részleges hiba elkerülése érdekében kiterjedő munkaegység állapotát rögzíti. Munkaelem például tárolása tartós várólista, amíg folyamatban van egy több tranzakció. 

- Csak a szolgáltatás igénylő adatainak tárolására. A szolgáltatás csak módosítania kell a tartomány entitás információt egy részét. Például a szállítási, amelyet a környezetben, azt kell tudni, hogy mely ügyfelek egy adott kézbesítési társítva. Nem szükséges az ügyfél számlázási címét, de &mdash; , amely kezeli a fiók, amelyet a környezetben. Itt gondosan gondolat a tartományhoz, és egy DDD megközelítéssel segítségével. 

- Vegye figyelembe, hogy a szolgáltatások következetes és lazán összekapcsolt-e. Ha két szolgáltatást folyamatosan cserél információt egymással, chatty API-k, ami esetleg a szolgáltatás határok újbóli egyesítése két szolgáltatás vagy funkció újrabontása.

- Használjon egy [eseményvezérelt architektúra stílus](../guide/architecture-styles/event-driven.md). A architektúra stílusát, a szolgáltatás közzéteszi a egy eseményt, amikor az nyilvános modellek és entitások módosultak. Érintett szolgáltatások ezeket az eseményeket is kérheti. Azt jelzi, például egy másik szolgáltatás használhatja-e az adatok lekérdezése több megfelelő materializált nézet létrehozásához az eseményeket. 

- Egy szolgáltatás események birtokló közzé kell tennie egy séma, amely segítségével automatizálhatja a szerializálása és deszerializálása események, gyártók és előfizetők közötti szoros kapcsoló elkerülése érdekében. Távolítsa el a JSON-séma vagy a keretrendszer például [Microsoft Bond](https://github.com/Microsoft/bond), Protobuf vagy Avro.  
 
- Nagy adatmennyiség esetén események keresztmetszetet jelenthet a rendszer, ezért fontolja meg az összesítés használatával, vagy a teljes terhelés csökkentésére kötegelés. 

## <a name="drone-delivery-choosing-the-data-stores"></a>Dron kézbesítési: Az adattároló kiválasztása 

Csak néhány szolgáltatások használatát, még a szállítás, amelyet a környezetben mutatja be, az ebben a szakaszban tárgyalt pontok számos. 

Amikor egy felhasználó egy új kézbesítési ütemezi, az ügyfél kérésében tartalmazza a mindkét a kézbesítési, például a felvétel és dropoff helyeket, és a csomag, például a méret és a súlyozást. Ez az információ munkaegység, amely a adatfeldolgozást szolgáltatás küld az Event Hubs határozza meg. Fontos, hogy a munkaegység marad az állandó tároló közben a Feladatütemező szolgáltatás végrehajtja a munkafolyamatot, hogy a kézbesítési kérelmek nem elvesznek. A munkafolyamat több leírását lásd: [adatfeldolgozást és a munkafolyamat](./ingestion-workflow.md). 

A különböző háttér-szolgáltatások különböző részei a kérelemben szereplő információk érdeklik és is rendelkezik különböző olvasási és írási profilok. 

### <a name="delivery-service"></a>Kézbesítési szolgáltatás

A kézbesítési szolgáltatás kapcsolatos minden jelenleg ütemezett kézbesítését vagy a folyamatban lévő adatait tárolja. A dronok származó eseményeket figyeli, és nyomon követi a folyamatban lévő kézbesítések állapotát. Is küld a tartományhoz események kézbesítési állapotának frissítése.

Valószínű, hogy a felhasználók gyakran abban az esetben ellenőrizze a kézbesítési állapotát, amíg a csomag vár. A kézbesítési szolgáltatás, ezért a tárolóban, amely emeli ki átviteli (olvasási és írási) keresztül hosszú távú tárolás szükséges. Emellett a kézbesítési szolgáltatás nem végez semmilyen összetett lekérdezéseknél, illetve elemzési, egyszerűen kéri le a legfrissebb állapotát egy adott szállítási. A szállítási csoport számára a nagy írási és olvasási teljesítményt Azure Redis Cache választott. A Redis tárolt információk viszonylag rövid élettartamú. A szállítási végrehajtása után a kézbesítési előzmények szolgáltatás a rendszer rekord.

### <a name="delivery-history-service"></a>Kézbesítési előzmények szolgáltatás

A szállítási előzmények szolgáltatás figyeli a kézbesítési állapoteseményeit a kézbesítési szolgáltatásból. Ezek az adatok hosszú távú tárolás tárolja. Nincsenek két különböző használati esetek a korábbi adatok, amelyek különböző adattárolási követelmények. 

Az első forgatókönyv összesítése van az adatok az üzleti optimalizálás, vagy a szolgáltatások minőségének javítása érdekében a adatelemzés céljából. Vegye figyelembe, hogy a kézbesítési előzmények szolgáltatás nem hajtja végre a tényleges adatok elemzése. Csak felelős adatfeldolgozást és tárolására. Ebben a forgatókönyvben a tárolót kell optimalizálni adatelemzés adatokat, a séma-a-olvasási megközelítés adatforrások számos olyan számos keresztül. [Azure Data Lake Store](/azure/data-lake-store/) jó alkalmas ebben a forgatókönyvben. Data Lake Store a Hadoop elosztott fájlrendszerrel (HDFS) kompatibilis Apache Hadoop fájl rendszer, és a teljesítmény adatok analytics forgatókönyvek van beállítva. 

A más forgatókönyv van így a felhasználók egy kézbesítési előzményeinek keresse meg a szállítási befejezése után. Azure Data Lake különösen nincs optimalizálva ehhez a forgatókönyvhöz. Az optimális teljesítmény érdekében a Microsoft azt javasolja, Data Lake dátum szerint particionálva mappákban idősorozat adatok tárolására. (Lásd: [hangolása Azure Data Lake Store a teljesítmény](/azure/data-lake-store/data-lake-store-performance-tuning-guidance)). Struktúra azonban nem optimális gyűjtéséhez az egyes rekordok azonosítóját. Csak is tudja a timestamp,-azonosító szerinti keresés igényel, a teljes gyűjteményt vizsgálatát. Emiatt a küldési előzményeinek szolgáltatás is tárolja a korábbi adatok egy részét Cosmos DB gyorsabb kereséshez. A rekordok nem kell ahhoz, hogy határozatlan ideig maradjanak az Cosmos-Adatbázisba. Régebbi kézbesítések archiválhatók &mdash; tegyük fel például, egy hónap után. Ezt megteheti az alkalmi kötegelt folyamat futtatásával.

### <a name="package-service"></a>Csomag szolgáltatás

A csomag szolgáltatás csomagokat kapcsolatos információkat tárolja. A csomag tárolási követelményei a következők: 

- Hosszú távú tároláshoz.
- Nagy írási teljesítményt igénylő, a csomagok nagy mennyiségű kezelésére képes.
- Támogatja az egyszerű lekérdezések csomagazonosító. Nincs bonyolult illesztésekre vagy a hivatkozási integritás követelményei.

Mivel a csomagban található adat nem relációs, egy Dokumentumközpontú adatbázis megfelelő, és Cosmos DB a szilánkos gyűjtemények segítségével nagyon nagy átviteli sebesség érhető el. A csoport, amely a csomag szolgáltatásban működik, és melyik ismeri a átlagos verem (MongoDB, Express.js, AngularJS és Node.js), a a [MongoDB API](/azure/cosmos-db/mongodb-introduction) Cosmos-adatbázis számára. Amely lehetővé teszi, hogy kihasználja a meglévő tapasztalataikat a MongoDB, amely egy olyan felügyelt Azure szolgáltatás Cosmos DB előnyei lekérése közben.

> [!div class="nextstepaction"]
> [Értekezleteire kommunikáció](./interservice-communication.md)