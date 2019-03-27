---
title: Az adatok mikroszolgáltatások kapcsolatos szempontok
description: Mikroszolgáltatás-alapú adatok szempontjai.
author: MikeWasson
ms.date: 02/25/2019
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: microservices
ms.openlocfilehash: dd620cdcb6cb3a06d4fae4fb34eeb086ec17b4c5
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298425"
---
# <a name="data-considerations-for-microservices"></a>Az adatok mikroszolgáltatások kapcsolatos szempontok

Ez a cikk ismerteti a mikroszolgáltatási architektúrákban adatkezelési szempontok. Mivel minden mikroszolgáltatás kezeli a saját adatai, a adatintegritásának és adatkonzisztencia kritikus kihívásokat is.

A mikroszolgáltatások egyik alapelve, hogy mindegyik szolgáltatás a saját adatait felügyeli. Két szolgáltatást kell ossza meg a tárolóban. Ehelyett minden egyes szolgáltatás a saját személyes adattár, amely más szolgáltatások közvetlenül nem férnek hozzá felelős.

Ez a szabály az az oka, hogy kerülje a szolgáltatások, amelyek okozhat, ha a szolgáltatások megosztása az azonos mögöttes adatsémák véletlen összekapcsolását. Az adatséma változás történik, ha a módosítást kell koordinált összes támaszkodik, hogy az adatbázis-szolgáltatás között. Minden egyes szolgáltatás adattárát elkülönítésével azt is módosítása hatókörének korlátozása, és valóban független üzembe helyezések rugalmasságának megőrzése. Egy másik oka, hogy mindegyik mikroszolgáltatás előfordulhat, hogy rendelkezik a saját adatmodellt, a lekérdezések, vagy olvasási/írási mintákat. A megosztott tárolóban használata korlátozza az egyes csapatok lehetővé teszi az adott szolgáltatáshoz adatok a tárolási költségek optimalizálására.

![A CQRS nem megfelelő megközelítést bemutató ábra](../../guide/architecture-styles/images/cqrs-microservices-wrong.png)

Ezt a megközelítést vezet természetes módon [polyglot-adatmegőrzés](https://martinfowler.com/bliki/PolyglotPersistence.html) &mdash; több egy alkalmazáson belül adattárolási technológiák használatát. Egy szolgáltatás szükség lehet a dokumentum-adatbázis az olvasási séma képességeit. Egy másik szükség lehet a RDBMS által biztosított hivatkozási integritást. Minden egyes csapat díjmentes, hogy a szolgáltatás a legjobb választás. Az általános elvet polyglot-adatmegőrzés kapcsolatos további információkért lásd: [a feladathoz legmegfelelőbb adattárat használja](../../guide/design-principles/use-the-best-data-store.md).

> [!NOTE]
> Ez nem okoz gondot szolgáltatások ugyanazon fizikai adatbázis-kiszolgáló megosztásához. A probléma akkor fordul elő, amikor a szolgáltatások megosztani ugyanazzal a sémával vagy olvasási és írási ugyanazokat az adatbázistáblák.

## <a name="challenges"></a>Problémák

Ez a megközelítés elosztott áttekinthet néhány problémát, adatkezelési merülnek fel. Először is előfordulhatnak redundancia különböző adattárakban, és az azonos elem több helyen jelennek meg adatok. Például adatok előfordulhat, hogy lehet egy tranzakció részeként tárolja, majd Analytics, a jelentésben, vagy archiválása máshol tárolt. Ismétlődik vagy particionált adatok is az adatok integritásának megőrzése, és a konzisztencia problémákat okozhat. Adatkapcsolatok span több szolgáltatást, amikor kényszeríteni a kapcsolatokat a hagyományos felügyeleti technikák nem használható.

A hagyományos modellezési használja a szabály az "egy tény egy helyen." Minden entitás pontosan egyszer jelenik meg a sémában. Más entitások hivatkozásokat tartalmaznak, de nem másolhatja. A nyilvánvaló hagyományos megközelítés előnye, hogy egyetlen helyen, és elkerülhetők a problémák az adatok konzisztenciájának végrehajtott frissítések. A mikroszolgáltatási architektúrákban akkor fontolja meg, hogyan frissítések szolgáltatás lépnek, és hogyan kezelje az erős konzisztencia nélkül több helyen jelenik meg adat végleges konzisztencia.

## <a name="approaches-to-managing-data"></a>Módszerek az adatok kezelése

Nincs egyetlen módszer, amely minden esetben helyes van, de Íme néhány általános irányelveket a mikroszolgáltatási architektúrákban adatok kezeléséhez.

- Ahol lehetséges, támogassa a végső konzisztenciát. Ismerje meg a helyek, a rendszer, ahol erős konzisztencia vagy ACID-tranzakciókat, és a helyeket, ahol a végleges konzisztencia fogadható el kell.

- Erős konzisztencia megvalósulásának van szüksége, amikor egy szolgáltatás jelöl az adott entitások, amelyek egy API-n keresztül közvetlenül hiteles forráshoz. Egyéb szolgáltatások olyan saját maguk az adatok másolatát, vagy az adatok, idővel konzisztenssé váljanak a törzsadatok, de nem veszi figyelembe a hiteles forrásaként egy részét. Képzeljünk el például egy e-kereskedelmi rendszerben egy rendelés ügyfélszolgálat és a egy szolgáltatás. A szolgáltatás-események előfordulhat, hogy figyelése a rendelés szolgáltatásból, de az ügyfél kéri a visszatérítés, hogy a rendelés szolgáltatás, nem a javaslat szolgáltatás, amely rendelkezik a teljes tranzakció előzmények.

- Tranzakció, használjon minták például [Feladatütemező ügynök felügyeleti](../../patterns/scheduler-agent-supervisor.md) és [kompenzáló tranzakció](../../patterns/compensating-transaction.md) és konzisztens adatokat több szolgáltatás között.  Szükség lehet tárolásához egy további adat, amely egy, több szolgáltatást, többek között több szolgáltatást részleges hiba elkerülése érdekében kiterjedő munkaegység állapotát rögzíti. Például, hogy egy munkaelemet az tartós üzenetsor amíg folyamatban van egy több lépésből álló tranzakció.

- Store csak a szolgáltatás szükséges adatokat. Szolgáltatás előfordulhat, hogy elegendő információ egy tartományi entitás egy része. Például a szállítási, amelyet a környezetben, azt kell tudni, hogy melyik ügyfél egy adott kézbesítési társítva. Az ügyfél számlázási cím nincs szükségünk, de &mdash; , amely a fiók, amelyet környezet kezeli. A tartomány gondosan mértékegységeként és DDD módszerével, itt is segítségével.

- Vegye figyelembe, hogy a szolgáltatások-e következetes és lazán összekapcsoltak. Ha a két szolgáltatás folyamatosan cserél információt egymással, forgalmas API-kat, így szükség lehet újrarajzolja a szolgáltatások határait egyesítése a két szolgáltatás vagy funkció újrabontás.

- Használja az [eseményvezérelt architektúra stílusának](../../guide/architecture-styles/event-driven.md). Ezt az architektúrastílust, a szolgáltatás egy eseményt, amikor a nyilvános modellek és entitások mértékben tesz közzé. Az érintett szolgáltatások előfizethetnek ezeket az eseményeket. Ha például egy másik szolgáltatás használatával az események hozhatnak létre, amely több lehetővé teszi az adatok materializált nézet.

- Egy szolgáltatás, amely a tulajdonában lévő események tegyen közzé egy sémát, amely segítségével automatizálhatja a szerializálásához és deszerializálásához az események között a kiadók és -előfizetők szoros összekapcsolódást elkerülése érdekében. Gondolja át, JSON-sémájában vagy keretrendszert, például [Microsoft Bond](https://github.com/Microsoft/bond), Protopuf, vagy avro-hoz.

- Nagy adatmennyiség esetén események a rendszer szűk keresztmetszetté válik, ezért érdemes az összesítést használ, vagy a teljes terhelés csökkentésére, kötegelés.

## <a name="example-choosing-data-stores-for-the-drone-delivery-application"></a>Példa: A Drone Delivery alkalmazás lehetőséget választva adattárak

A sorozat előző cikkek egy drónos szállítási szolgáltatást egy futó példaként tárgyalják. Talál további információt a forgatókönyv és a megfelelő megvalósításának hivatkozhat [Itt](./index.md).

Recap, hogy az alkalmazás több mikroszolgáltatások a kézbesítések ütemezését, a drone határozza meg. Amikor egy felhasználó egy új szállítási ütemezi, az ügyfél kérése tartalmaz, például a méret és a súlyozást a csomag és a szállítási begyűjtést és dropoff helyeket, például vonatkozó információkat. Ezt az információt adott munkaegység határozza meg.

A különböző háttérrendszerekhez érdeklik a különböző részeit az adatokat a kérésben és is rendelkezik különböző olvasási és írási profilok.

![Az adatok kapcsolatos szempontok ábrája](../images/data-considerations.png)

### <a name="delivery-service"></a>Kézbesítési szolgáltatás

A kézbesítési szolgáltatás minden teljesítés jelenleg ütemezett kapcsolatos és folyamatban lévő információkat tárolja. Figyeli a drónok-eseményfolyam megtekintéséhez a, és nyomon követi a folyamatban lévő kézbesítések állapotát. Tartományi események kézbesítési állapotának frissítése is elküldi.

Valószínű, hogy a felhasználók gyakran abban az esetben ellenőrizze a kézbesítési állapotát, amíg a csomag várnak. Ezért a kézbesítési szolgáltatás igényel, amelyek kiemelik átviteli (olvasási és írási) keresztül hosszú távú tárolás a tárolóban. Emellett a kézbesítési szolgáltatás nem hajt végre semmilyen összetettebb lekérdezések vagy az analysis, egyszerűen lekéri a legfrissebb állapotát egy adott szállítási. A kézbesítés service csapatának Azure Redis Cache választotta, a magas olvasási és írási teljesítmény. A redis tárolt adatok viszonylag rövid életű. A kézbesítési befejezése után a a szállítás előzményei szolgáltatás a rendszerben.

### <a name="delivery-history-service"></a>Előzmények szolgáltatásra

A szállítás előzményei szolgáltatás figyeli a kézbesítési szolgáltatás eseményeit, szállítási állapot. A hosszú távú tárolás tárolja ezeket az adatokat. Nincsenek két különböző használati esetek korábbi adatok, amelyek eltérő tárolási követelményekkel rendelkeznek.

Az első forgatókönyv összesíti az adatokat a data-elemzések, optimalizálhatja az üzleti vagy a szolgáltatás minőségének javítása érdekében. Vegye figyelembe, hogy a szállítás előzményei szolgáltatás nem hajtja végre a tényleges adatok elemzése. Csak feladata a feldolgozási és tárolási. A jelen esetben a storage kell optimalizálni adatelemzés keresztül adatokat, egy olvasási séma megközelítést befogadására, adatforrások különböző nagy készletét. [Az Azure Data Lake Store](/azure/data-lake-store/) van egy jó választás a ebben a forgatókönyvben. Data Lake Store az Apache Hadoop-fájlrendszer az a Hadoop elosztott fájlrendszer (HDFS) kompatibilis, és a nagy teljesítményt nyújtson az adatelemzési forgatókönyvekben.

A bármilyen más forgatókönyvhöz van így a felhasználók a kézbesítési befejeződése után keresse ki egy kézbesítési előzményeit. Az Azure Data Lake különösen nincs optimalizálva az ebben a forgatókönyvben. Az optimális teljesítmény érdekében a Microsoft azt javasolja, a Data Lake dátum alapján particionált mappákban idősorozat-adatok tárolására. (Lásd: [a teljesítmény hangolása az Azure Data Lake Store](/azure/data-lake-store/data-lake-store-performance-tuning-guidance)). Struktúra viszont nem optimális által az egyes rekordok keresése Ha is tudja a timestamp, a keresés azonosító alapján szükséges, vizsgálata a teljes gyűjteményt. Ezért a szállítás előzményei szolgáltatás is tárolja a korábbi adatok egy részét Cosmos DB-ben a gyorsabb keresés. A rekordok nem kell a Cosmos DB-ben határozatlan ideig marad. Régebbi kézbesítések archiválhatók &mdash; egy hónap után mondja ki. Ezt megteheti egy alkalmi kötegfolyamat futtatásával.

### <a name="package-service"></a>Szolgáltatási csomag

A csomag szolgáltatás az összes olyan csomagot adatait tárolja. A csomag tárolási követelményei a következők:

- Hosszú távú tároláshoz.
- Nagy mennyiségű, csomagok, nagy írási teljesítményt igénylő kezelésére képes.
- Támogatja az egyszerű lekérdezések csomagazonosítót. Nem bonyolult illesztésekre vagy a hivatkozásintegritás.

Mivel a csomag adatai nem relációs, dokumentum-orientált adatbázis megfelelő, és Cosmos DB nagyon nagy átviteli sebességet érhet el horizontálisan skálázott gyűjtemények használatával. A csoport, amely a csomag szolgáltatás működik tisztában van-e a MEAN-verem (MongoDB, szolgáló Express.js, AngularJS és Node.js), így azok válassza ki a [MongoDB API-val](/azure/cosmos-db/mongodb-introduction) Cosmos DB-hez készült. Amely lehetővé teszi a MongoDB, ez egy felügyelt Azure-szolgáltatás a Cosmos DB előnyeit beolvasása közben meglévő élményt kiaknázását.
