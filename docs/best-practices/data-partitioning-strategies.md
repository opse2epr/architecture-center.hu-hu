---
title: Adatok particionálási stratégiái
titleSuffix: Best practices for cloud applications
description: Megadhat, és külön elérhető adatok partíciók útmutatást.
author: dragon119
ms.date: 11/04/2018
ms.topic: best-practice
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: a87972a3901ed9499b5b25831131a79ff5db8f87
ms.sourcegitcommit: eee3a35dd5a5a2f0dc117fa1c30f16d6db213ba2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/06/2019
ms.locfileid: "55782098"
---
# <a name="data-partitioning-strategies"></a>Adatok particionálási stratégiái

Ez a cikk ismerteti a különböző Azure adattárakban lévő adatokat a particionálás néhány stratégiát. Adatok particionálása és ajánlott eljárások mikor kapcsolatos általános útmutatásért lásd [adatparticionálás](./data-partitioning.md)

## <a name="partitioning-azure-sql-database"></a>Az Azure SQL Database particionálási

Az egyes SQL-adatbázisokban tárolható adatok mennyisége korlátozott. A feldolgozási sebességet architekturális tényezők és az adatbázis által támogatott egyidejű kapcsolatok száma korlátozzák.

[Rugalmas készletek](/azure/sql-database/sql-database-elastic-pool) támogatja a horizontális skálázást SQL-adatbázishoz. Rugalmas készletek használata esetén is particionálja az adatait, amely több SQL-adatbázis között szegmensekre. A kezelendő adatok mennyiségének növekedésével vagy csökkenésével hozzá is adhat és el is távolíthat szegmenseket. Rugalmas készletek is segít csökkenteni a versengést elosztja a terhelést az adatbázisok között.

Minden egyes szegmens egy SQL-adatbázisként van megvalósítva. Szegmensek több adatkészletet képes tárolni (nevű egy *shardlet*). Minden egyes adatbázis maga biztosítja a benne található shardleteket leíró metaadatokat. Egy shardlet lehet egyetlen adatelemet vagy shardlet ugyanazzal a kulccsal rendelkező elemek csoportja. Ha például egy több-bérlős alkalmazásban, a shardletkulcs lehet a bérlő Azonosítóját, és egy bérlő összes adatát ugyanaz a shardlet tárolhatja tárolhatja.

Ügyfélalkalmazások adatkészlet társít egy shardletkulcs felelős. Egy külön SQL-adatbázis szolgál globális szegmenstérkép-kezelőként. Az adatbázis összes szegmens és shardlet listáját a rendszer tartalmaz. Az alkalmazás egy példányát a szegmenstérkép beszerzése a szegmenstérkép-kezelő adatbázis csatlakozik. A horizontális skálázási térképet helyben gyorsítótárazza, és használja a térkép kérelmek átirányítása a megfelelő szegmensre. Ez a funkció takarja el egy sor API, amely tartalmazza a [Elastic Database-ügyfélkódtár](/azure/sql-database/sql-database-elastic-database-client-library), amely Java és .NET érhető el.

Rugalmas készletek kapcsolatos további információkért lásd: [horizontális felskálázás az Azure SQL Database](/azure/sql-database/sql-database-elastic-scale-introduction).

Csökkentheti a késéseket és javítható a rendelkezésre állás, a globális szegmenstérkép-kezelő adatbázis lehet replikálni. A prémium szintű díjcsomagok árából, az aktív georeplikáció folyamatosan replikálhatja az adatokat különböző régiókban lévő adatbázisokba konfigurálható.

Másik megoldásként használhatja [Azure SQL Data Sync](/azure/sql-database/sql-database-sync-data) vagy [Azure Data Factory](/azure/data-factory/) a szegmenstérkép-kezelő adatbázis replikálása régiók között elosztva. Ilyen típusú replikáció időközönként fut, és megfelelőbb, ha a szegmenstérkép ritkán módosul, és nem szükséges prémium szintű.

Az Elastic Database két sémát kínál az adatok shardletekre való leképezésére és a szegmenseken való eltárolására:

- A **listás szegmenstérkép** ugyanazt a kulcsot egy shardlet társítja. Például egy több-bérlős rendszerben az egyes bérlők adatai egy egyedi kulccsal lehetnek társítva, majd egy külön shardletben tárolhatók. Elkülönítés biztosítása, minden egyes shardletek saját horizontális megtartható.

    ![Bérlők adatainak tárolása külön szegmensekben listás szegmenstérkép használatával](./images/data-partitioning/PointShardlet.png)

- A **tartomány-szegmenstérkép** társítja a egy shardlet összefüggő key értékek egy halmazát. Ha például csoportosíthatók azon bérlők számára (mindegyik saját kulccsal) belül ugyanaz a shardlet tárolhatja az adatokat. Ez a séma kevésbé költséges, mint az első, mivel a bérlők osztoznak az adattárolási, de kevesebb elszigetelést.

    ![Bérlőtartomány adatainak tárolása tartomány-szegmenstérkép használatával](./images/data-partitioning/RangeShardlet.png)

Egyetlen szegmens több shardlet adatait is tartalmazhatja. Például listás shardletek használatával több nem egymást követő shardlet adatait is tárolhatja egyetlen szegmensben. Bár ezek külön térképeken kibocsátásokban megtörténik a tartományshardleteket és a listás shardleteket ugyanabban a szegmensben is kombinálhatók. Az alábbi ábrán ez a módszer:

![Több szegmenstérkép megvalósítása](./images/data-partitioning/MultipleShardMaps.png)

Rugalmas készletek adjon hozzá és távolíthat szegmenseket az adatmennyiség növekedésének vagy csökkenésének lehetővé teszi. Ügyfél-alkalmazások létrehozása és törölhetnek szegmenseket dinamikusan, és transzparens módon frissíthetik a szegmenstérkép-kezelőt. A szegmensek eltávolítása azonban egy destruktív művelet, amelyet követően a szegmensben lévő összes adatot törölni kell.

Ha egy alkalmazásnak kell osztani két külön szegmensekben vagy összevonás szegmensekre, használja a [felosztási-egyesítési eszközének](/azure/sql-database/sql-database-elastic-scale-overview-split-and-merge). Ez az eszköz egy Azure-webszolgáltatásként fut, és migrálja az adatokat a szegmensek közötti biztonságos.

A particionálási séma jelentős hatással lehet a rendszer teljesítményét. Azt is befolyásolhatja a sebesség, amellyel szegmensek kell hozzáadni vagy eltávolítani, illetve adatokat kell kell particionálni a szegmensek között. Vegye figyelembe a következőket:

- Ugyanabban a szegmensben az együtt használt adatokat csoportosítsa, és kerülje az olyan műveleteket, amelyek több szegmens származó adatok eléréséhez. Egy szegmens egy önálló SQL-adatbázis, és adatbázisok közti összekapcsolásokat az ügyféloldalon kell végrehajtani.

    Bár az SQL Database nem támogatja az adatbázisok közti összekapcsolásokat, a rugalmas Adatbáziseszközök végrehajtásához használhatja [megtalálhatjuk szegmensek közötti lekérdezéseket](/azure/sql-database/sql-database-elastic-scale-multishard-querying). Többszegmenses lekérdezés egyéni lekérdezéseket küld minden egyes adatbázishoz, és egyesíti az eredményeket.

- Olyan rendszerek, szegmensek közötti függőségek rendelkező kialakítását. Hivatkozási integritási megkötéseket, eseményindítók és a egy adatbázisban tárolt eljárások nem hivatkozhat egy másik objektumaira.

- Ha rendelkezik a lekérdezések által gyakran használt referenciaadatok, fontolja meg az adatok replikálása a szegmensek között. Ez a megközelítés is szükségtelenné teszi a data join több adatbázisban. Ideális esetben az ilyen adatok lehetnek statikus vagy lassan mozgó, csökkentheti a replikálás beavatkozást, és csökkentheti a replikációigény elavult.

- Ugyanahhoz a szegmenstérképhez tartozó Shardlet ugyanazzal a sémával kell rendelkeznie. Ez a szabály nem kényszeríti az SQL Database, azonban az adatok kezelése és lekérdezése rendkívül bonyolulttá, ha az egyes shardletek különböző sémákkal rendelkeznek válik. Ehelyett hozzon létre külön szegmenstérképet minden séma. Ne feledje, hogy különböző shardleteken adatokat tárolhatja ugyanabban a szegmensben.

- Tranzakciós műveletek csak az adatok horizontális skálázáson belül, és a szegmensek között nem támogatottak. A tranzakciók több shardletet is érinthetnek, amennyiben azok egyazon szegmens részét képezik. Ezért ha az üzleti logikának kell tranzakciókat végrehajtania, vagy ugyanabban a szegmensben lévő adatok tárolásához, vagy valósítson meg végső konzisztenciát.

- A szegmenseket tárolja az azokban lévő adatokat használó felhasználók közelében helyezze el. Ezzel a stratégiával csökkenthetők a késések.

- Kerülje a nagyon aktív és inaktív viszonylag szegmensek vegyesen kellene. Próbálja a terhelést egyenletesen elosztani a szegmensek között. Előfordulhat, hogy a horizontális skálázás kulcsok kivonatoláshoz. Ha a szegmensek földrajzi elhelyezkedése lényeges, gondoskodjon róla, hogy a kivonatalapú kulcsok az adatokat használó felhasználókhoz közeli szegmensekben tárolt shardletekre mutatnak.

### <a name="partitioning-azure-table-storage"></a>Az Azure Table Storage particionálása

Az Azure Table Storage egy particionálási célokra kifejlesztett kulcs-érték tároló. Minden entitás tárolása egy partícióban történik, és a partíciókat belsőleg az Azure Table Storage kezeli. A táblákban tárolt minden entitás meg kell adnia egy kétrészes kulcsot, amely tartalmazza:

- **A partíciókulcs**. Ha az Azure table storage helyezi az entitást, amely meghatározza a partíció karakterlánc-érték ez. Ugyanazzal a partíciókulccsal rendelkező entitások mind ugyanazon a partíción tárolja.

- **A sorkulcs**. Ez az egy karakterláncérték, amely azonosítja az entitást a partíción belül. A partícióban lévő entitások a sorkulcs alapján vannak növekvő betűrendbe rendezve. Minden egyes entitásnak egyedi partíciókulcs-sorkulcs kombinációval kell rendelkeznie, amelynek a hossza legfeljebb 1 KB lehet.

Ha egy korábban nem használt partíciókulccsal rendelkező entitást vesz fel egy táblába, az Azure Table Storage egy új partíciót hoz létre az entitás számára. Az ezzel a partíciókulccsal rendelkező többi entitás is ugyanezen a partíción lesz tárolva.

Ez az eljárás lényegében egy automatikus horizontális felskálázási stratégiát valósít meg. Mindegyik partíció tárolja az Azure-adatközpontban annak biztosítására, hogy adatokat egyetlen partícióról lekérő lekérdezések gyorsan futhassanak ugyanazon a kiszolgálón.

A Microsoft tett közzé [teljesítménycélokat] Azure Storage-hoz. Ha a rendszer várhatóan meghaladja majd ezeket a korlátokat, érdemes lehet entitások felosztása több táblára felosztva. Vertikális particionálás használatával ossza fel a mezőket csoportokra aszerint, hogy várhatóan melyik mezőkhöz férnek hozzá együtt.

Az alábbi ábrán látható egy példatárfiókban logikai struktúráját. A tárfiók három táblát tartalmaz: Customer Info, Product Info és Order Info.

![Táblák és partíciók egy példatárfiókban](./images/data-partitioning/TableStorage.png)

Mindegyik tábla több partíciót tartalmaz.

- A Customer Info táblában az adatok particionálása az alapján a város, ahol az ügyfél megtalálható. A sorkulcs az ügyfél-azonosítót tartalmaz.
- A Product Info táblában a termékek a termékkategória szerint vannak particionálva, és a sorkulcs a termékszám.
- Az Order Info táblában a megrendelések rendelési dátum szerint vannak particionálva, és a sorkulcs Megadja azt az időtartamot, a megrendelések fogadásának. Az adatok az egyes partíciókban a sorkulcs alapján vannak rendezve.

Az Azure Table Storage-ben használt entitások kialakításakor vegye figyelembe a következő szempontokat:

- Válassza ki egy partíciókulcsot és a sorkulcs szerint hogyan az adatokhoz. Válasszon olyan partíciókulcs-sorkulcs kombinációt, amely a lekérdezések nagyobb részét támogatja. A leghatékonyabb lekérdezések a partíciókulcs és a sorkulcs megadásával kérik le az adatokat. Az egy partíciókulcsot és a sorkulcsok egy tartományát megadó lekérdezések egy partíció vizsgálatával hajthatók végre. Ez viszonylag gyorsan megy, mivel az adatok a sorkulcs szerint vannak rendezve. Lekérdezések nem adja meg, melyik partíciót kell átvizsgálni, ha minden partíciót kell átvizsgálni.

- Ha valamely entitás természetes kulccsal rendelkezik, használja azt partíciókulcsként, és adjon meg egy üres sztringet sorkulcsként. Ha egy entitás két tulajdonság álló összetett kulccsal rendelkezik, válassza ki a ritkábban változó tulajdonságot partíciókulcsként, míg a másik sorkulcsként. Ha egy entitás kettőnél több kulcstulajdonsággal rendelkezik, a tulajdonságok összefűzésével adja meg a partíció- és a sorkulcsot.

- Ha rendszeresen hajt végre lekérdezéseket, amelyek a partíció- és sorkulcsot kulcsok eltérő mezők alapján adatokat kereshet, vegye fontolóra a [Indextábla minta](../patterns/index-table.md), vagy fontolja meg egy másik adattárhoz, amely támogatja az indexelést, például a Cosmos DB használatával .

- Ha a partíciókulcsokat egy monoton sorozat alapján (például "0001", "0002", "0003") használatával hozzon létre, és mindegyik partíció csak korlátozott mennyiségű adatot tartalmaz, az Azure table storage képes fizikailag csoportosítani ezeket a partíciókat együtt ugyanazon a kiszolgálón. Az Azure Storage feltételezi, hogy az alkalmazás nagy valószínűséggel egy egybefüggő tartományát (lekérdezések) partíciók közötti lekérdezések végrehajtásához, és ebben az esetben van optimalizálva. Azonban ez a megközelítés vezethet pontokhoz, mert az új entitások koncentrálódik várhatóan egyik végén lehet koncentrált az egybefüggő tartományát. Emellett a skálázhatóságot is csökkentheti. A terhelés egyenletesebb elosztását, érdemes kivonatolnia a partíciókulcsot.

- Az Azure Table Storage támogatja a tranzakciós műveletek végrehajtását az egyazon partíción lévő entitásokon. Egy alkalmazás elvégezhet több insert, update, delete, cseréje vagy egyesítési műveletek egyetlen elemi egységként, mindaddig, amíg a tranzakció nem tartalmaz 100-nál több entitást és a kérelem hasznos adattartalma nem haladja meg a 4 MB. Nem tranzakciós művelet több partíciót, és előfordulhat, hogy a végleges konzisztencia megvalósításával. A table storage és a tranzakció kapcsolatos további információkért lásd: [Entitáscsoport-tranzakciók végrehajtása].

- Vegye figyelembe a partíciókulcs részletességét:

  - Minden entitás ugyanazzal a partíciókulccsal használatával ugyanazon a partíción egy kiszolgálón tárolt eredménye. Ez megakadályozza, hogy a partíció horizontális felskálázás, és a terhelés összpontosít egyetlen kiszolgálón. Ennek eredményeképpen Ez a módszer csak akkor kis számú entitást tárolására alkalmas. Azonban, győződjön meg arról, hogy valamennyi entitás részt vehet tranzakciók.

  - Minden entitás egyedi partíciókulccsal rendelkezik a table storage szolgáltatás egy külön partíciót az egyes entitásokhoz lévő nagy számú kis partíció létrehozásához. Ez a megközelítés jobban skálázható, mint az, amelyik egyetlen partíciókulcsot használ, de az entitáscsoportokon végzett tranzakciók nem lehetségesek. Emellett a több entitást érintő lekérdezések esetén esetleg több kiszolgálót is olvasni kell. Azonban az alkalmazás által végrehajtott lekérdezések, ha a partíciókulcsokat monoton sorozat mentén hozza majd használatával segíthet optimalizálhatóak.

  - Különböző entitások egy részét a partíciós kulcs megosztása lehetővé teszi, hogy a csoport kapcsolódó entitások ugyanazon a partíción. A kapcsolódó entitásokat érintő műveletek végrehajthatók entitáscsoport-tranzakciók használatával, és a több kapcsolódó entitást olvasó lekérdezések egyetlen kiszolgáló elérésével teljesíthetőek.

További információkért lásd: [Az Azure Storage Table tervezési útmutatója].

## <a name="partitioning-azure-blob-storage"></a>Az Azure Blob Storage particionálása

Az Azure blob storage segítségével nagyméretű bináris objektumok tárolhatóak. Használja a blokkblobok használatát támogatják forgatókönyvekben, ha szeretne feltölteni, vagy töltse le a nagy adatmennyiségek gyors. A lapblobok olyan alkalmazásokban használhatóak, ahol nem sorban, hanem véletlenszerűen kell elérni az adathalmaz egyes részeit.

Minden blob (blokk vagy lap) külön tárolóban található egy Azure-tárfiókban. A tárolók használatával csoportokba rendezhetőek a hasonló biztonsági követelményekkel rendelkező kapcsolódó blobok. A csoportosítás nem fizikai, hanem logikai alapon történik. A tárolókon belül minden egyes blob egyedi névvel rendelkezik.

A blobok partíciókulcsa a fiók neve + a tároló neve + a blob neve. A partíciókulcs partíció tartományok és a tartományok adatok betöltése-között oszlanak el a rendszer segítségével. A blobok leoszthatóak több kiszolgálóra a hozzáférés horizontális felskálázása érdekében, egy blobot azonban csak egyetlen kiszolgáló szolgálhat ki.

Ha az elnevezési sémája időbélyegeket vagy numerikus azonosítókat használ, vezethet túlzott forgalmat egy partíciót, hogy hatékonyan korlátozza a rendszer a terheléselosztás. Például ha egy blob objektumot használó időbélyegzővel ellátott például napi üzemeltetési *éééé-hh-nn*, a művelet az összes forgalom egypartíciós kiszolgálóhoz lép majd. Ehelyett érdemes lehet a név 3 számjegyből kivonattal előtag. További információkért lásd: [Partícióelnevezési konvenciót](/azure/storage/common/storage-performance-checklist#subheading47)

Egy egyetlen blokk vagy lap írására vonatkozó műveletek elemiek, de több blokkok, lapra vagy nem. Ha biztosítania kell a blokkokon, lapokon és blobokon végzett írási műveletek során a konzisztenciát, blobbérlet alkalmazásával zárolhatja az írásukat.

## <a name="partitioning-azure-storage-queues"></a>Az Azure tároló-üzenetsorok particionálása

Az Azure tároló-üzenetsorok használatával folyamatok közötti aszinkron üzenetkezelés valósítható meg. Minden egyes Azure-tárfiók korlátlan számú üzenetsort, és mindegyik üzenetsor korlátlan számú üzenetet tartalmazhat. Az egyetlen korlát a tárfiókban rendelkezésre álló hely mennyisége. Egy egyes üzenetek maximális mérete 64 KB. Ha ennél nagyobb méretű üzenetekre van szüksége, vegye fontolóra az Azure Service Bus-üzenetsorok használatát.

Minden egyes üzenetsor egyedi névvel rendelkezik a tárfiókon belül, amelyben található. Az Azure név alapján particionálja az üzenetsorokat. Az egy üzenetsorba tartozó üzeneteket egy partíció tárolja, amelyet egy kiszolgáló vezérel. A különböző üzenetsorok a terhelés elosztása érdekében futtathatók külön kiszolgálókon. Az üzenetsorok kiszolgálókra való leosztása az alkalmazások és a felhasználók számára transzparens módon történik.

A nagy alkalmazásokban ne ugyanazt az üzenetsort használja az alkalmazás összes példányához, mivel így az üzenetsort futtató kiszolgáló kritikus ponttá válhat. Használjon inkább külön üzenetsorokat az alkalmazás különböző funkcionális területei szerint. Az Azure tároló-üzenetsorok nem támogatják a tranzakciókat, ezért ha az üzeneteket külön üzenetsorokra irányítja, ez nem befolyásolja az üzenetkonzisztenciát.

Az egyes Azure tároló-üzenetsorok másodpercenként 2000 üzenetet képesek feldolgozni. Ha ennél nagyobb sebességre van szüksége, érdemes több üzenetsort létrehoznia. Például egy globális alkalmazás esetében hozzon létre külön tároló-üzenetsorokat külön tárfiókokban az egyes régiókban futó alkalmazáspéldányok kezelésére.

## <a name="partitioning-azure-service-bus"></a>Az Azure Service Bus particionálási

Az Azure Service Bus egy üzenetközvetítő használatával dolgozza fel a Service Bus-üzenetsorokra vagy -témakörökbe küldött üzeneteket. Alapértelmezés szerint az üzenetsorra vagy témakörbe küldött összes üzenetet ugyanaz az üzenetközvetítő folyamat kezeli. Az ilyen architektúra korlátozhatja az üzenetsor teljes feldolgozási sebességét. Az üzenetsorok és témakörök azonban a létrehozásukkor particionálhatóak. Ezt az üzenetsor vagy témakör *EnablePartitioning* tulajdonságának *true* értékre állításával teheti meg.

A particionált üzenetsor vagy témakör több töredékre osztható fel, amelyek mindegyikét egy külön üzenettár és üzenetközvetítő szolgálja ki. A töredékek létrehozása és felügyelete a Service Bus feladata. Amikor egy alkalmazás üzenetet küld egy particionált üzenetsorra vagy témakörbe, a Service Bus hozzárendeli az üzenetet az üzenetsor vagy témakör valamelyik töredékéhez. Amikor egy alkalmazás üzenetet fogad egy üzenetsorról vagy előfizetésről, a Service Bus az összes töredékben ellenőrzi, hogy melyik a következő elérhető üzenet, és azt továbbítja az alkalmazásba feldolgozásra.

Ez a struktúra segít elosztani a terhelést az üzenetközvetítők és üzenettárak közt, így növeli a skálázhatóságot és javítja a rendelkezésre állást. Ha valamelyik töredék üzenetközvetítője vagy üzenettára átmenetileg nem érhető el, a Service Bus a többi elérhető töredék valamelyikéből ki tudja olvasni az üzeneteket.

A Service Bus az üzeneteket az alábbiak szerint osztja le a töredékekre:

- Ha az üzenet egy munkamenethez, ugyanazt az értéket rendelkező üzenetek a *SessionId* tulajdonság ugyanarra a töredékre érkeznek.

- Ha az üzenet nem képezi munkamenet részét, de a feladó megadta a *PartitionKey* tulajdonság értéket, az összes azonos *PartitionKey* értékű üzenet ugyanarra a töredékre lesz küldve.

  > [!NOTE]
  > Ha a *SessionId* és a *PartitionKey* tulajdonságok értéke egyaránt meg van adva, ezeket ugyanarra az értékre kell állítani, vagy az üzenet el lesz utasítva.

- Ha egy üzenet *SessionId* és *PartitionKey* tulajdonságai nincsenek megadva, de a duplikálásészlelés engedélyezve van, a rendszer a *MessageId* tulajdonságot használja. Az azonos *MessageId* értékű üzenetek ugyanarra a töredékre lesznek küldve.

- Ha az üzeneteknek nincs *SessionId, PartitionKey* vagy *MessageId* tulajdonsága, a Service Bus szekvenciálisan osztja le az üzeneteket a töredékekre. Ha egy töredék nem érhető el, a Service Bus továbblép a következőre. Ez azt jelenti, hogy az üzenetkezelési infrastruktúra ideiglenes hibája nem okozza az üzenetküldési művelet meghiúsulását.

A Service Bus-üzenetsorok vagy -témakörök particionálásának tervezése során mérlegelje a következőket:

- A Service Bus-üzenetsorok és -témakörök egy Service Bus-névtér hatókörén belül jönnek létre. A Service Bus névterenként jelenleg 100 particionált üzenetsort vagy témakört képes kezelni.

- Az egyes Service Bus-névterek kvótákat írnak elő a rendelkezésre álló erőforrásokra, például az egyes témakörök előfizetőinek számára, az egyidejű küldési és fogadási kérelmek számára, valamint a létrehozható egyidejű kapcsolatok maximális számára vonatkozóan. Kvóta vannak dokumentálva [Service Bus-kvóták]. Ha várhatóan túllépi majd ezeket az értékeket, hozzon létre további névtereket a maguk üzenetsoraival és témaköreivel, és ossza le a terhelést ezekre a névterekre. Például egy globális alkalmazás esetében hozzon létre külön névtereket mindegyik régióban, és konfigurálja az alkalmazáspéldányokat, hogy a legközelebbi névtér üzenetsorait és témaköreit használják.

- A tranzakciók keretében küldött üzenetekben meg kell adni egy partíciókulcsot. Ez lehet egy *SessionId*, egy *PartitionKey* vagy egy *MessageId* tulajdonság. Az egy tranzakció keretében küldött üzenetekben ugyanazt a partíciókulcsot kell megadni, mivel ugyanannak az üzenetközvetítő folyamatnak kell feldolgoznia azokat. Az egy tranzakcióba tartozó üzeneteket nem küldheti külön üzenetsorokra vagy témakörökbe.

- A particionált üzenetsorok és témakörök nem konfigurálhatóak automatikus törlésre tétlenség esetére.

- A particionált üzenetsorok és témakörök jelenleg nem használhatóak az Advanced Message Queueing Protocol (AMQP) protokollal, ha platformfüggetlen vagy hibrid megoldásokat épít fel.

## <a name="partitioning-cosmos-db"></a>A Cosmos DB particionálási

Az Azure Cosmos DB egy NoSQL-adatbázis, amely az [Azure Cosmos DB SQL API][cosmosdb-sql-api] használatával képes JSON-dokumentumokat tárolni. A Cosmos DB-adatbázisban lévő objektumok vagy egyéb adatok JSON-szerializált ábrázolásai. A rendszer nem tartat be rögzített sémákat, ez alól az egyetlen kivétel az, hogy minden dokumentumnak tartalmaznia kell egy egyedi azonosítót.

A dokumentumok gyűjteményekbe vannak rendezve. A kapcsolódó dokumentumokat gyűjteménybe foglalhatja. Például egy blogbejegyzéseket karbantartó rendszerben mindegyik blogbejegyzés tartalmát külön dokumentumban tárolhatja egy gyűjteményben. Az egyes tématípusok szerint is létrehozhat gyűjteményeket. Egy több-bérlős alkalmazásban, például egy olyan rendszerben, ahol több szerző is kezelheti és irányíthatja a saját blogbejegyzéseit, a blogokat particionálhatja szerző szerint, és mindegyik szerző számára külön gyűjteményeket hozhat létre. A gyűjtemények számára lefoglalt tárhely rugalmas, szükség szerint csökkenthető vagy növelhető.

A Cosmos DB támogatja az adatok automatikus particionálását az alkalmazásban definiált partíciókulcs alapján. A *logikai partíció* egy olyan partíció, amely egy adott partíciókulcs-értékhez tartozó adatokat tárolja. Az adott partíciókulcs-értékkel rendelkező összes dokumentum ugyanabba a logikai partícióba kerül. A Cosmos DB az értékeket a partíciókulcs kivonata alapján osztja el. Az egyes logikai partíciók maximális mérete 10 GB. Így a partíciókulcs megfelelő megválasztása fontos szempont a tervezés során. Válasszon egy olyan tulajdonságot, amelynek az értékei és a hozzáférési mintái széles skálán mozognak. További információkat az [Azure Cosmos DB particionálási és horizontális leskálázási eljárásait](/azure/cosmos-db/partition-data) ismertető cikkben talál.

> [!NOTE]
> Mindegyik Cosmos DB-adatbázis rendelkezik *teljesítményszinttel*, amely az adatbázis számára elérhető erőforrások mennyiségét határozza meg. A teljesítményszinthez egy *kérelemegység* (RU) sebességkorlát tartozik. Az RU-sebességkorlát az adott gyűjtemény számára fenntartott, kizárólag általa használható erőforrások mennyiségét határozza meg. A gyűjtemény költségei az adott gyűjtemény számára kiválasztott teljesítményszinttől függenek. A nagyobb teljesítményszint (és RU-sebességkorlát) nagyobb költségekkel jár. A gyűjtemények teljesítményszintjét az Azure Portalon módosíthatja. További információkat [az Azure Cosmos DB kérelemegységeit ismertető][cosmos-db-ru] cikkben talál.

Ha a particionálási mechanizmust, Cosmos DB által biztosított nem elegendő, szükség lehet a szegmensben az adatokat az alkalmazás szintjén. A dokumentumgyűjtemények egy természetes mechanizmust kínálnak az adatok particionálására egy adott adatbázisban. A szegmensekre particionálás legegyszerűbben úgy valósítható meg, ha mindegyik szegmenshez létrehoz egy gyűjteményt. A tárolók logikai erőforrások, és több kiszolgálóra is kiterjedhetnek. A rögzített méretű tárolók mérete legfeljebb 10 GB, feldolgozási sebessége legfeljebb 10000 RU/s lehet. Korlátlan tárolók nem rendelkeznek maximális tárméret, de meg kell adnia egy partíciókulcsot. Az alkalmazások horizontális particionálása esetén az ügyfélalkalmazásnak a kérelmeket a megfelelő szegmensre kell irányítania, ez általában az adatok a szegmenskulcsot meghatározó egyes attribútumain alapuló leképezési mechanizmussal valósítható meg.

Az összes adatbázis egy Cosmos DB-adatbázis adatbázisfiók környezetében jön létre. Egy fiók több adatbázist is tartalmazhat, és meghatározza, hogy az adatbázisok mely régiókban lesznek létrehozva. Mindegyik fiók betartatja a saját hozzáférés-vezérlését. A Cosmos DB fiókok használatával a szegmenseket (az adatbázisokon belüli gyűjteményeket) az azokat használó felhasználókhoz földrajzilag közel helyezheti el, és kikényszerítheti, hogy csak az adott felhasználók érhessék el azokat.

Ha az adatokat a Cosmos DB SQL API-val tervezi particionálni, vegye figyelembe a következő szempontokat:

- **A Cosmos DB-adatbázis számára elérhető erőforrások mennyisége a fiók kvótakorlátainak függvénye**. Mindegyik adatbázis több gyűjteményt is tárolhat, és mindegyik gyűjteményhez tartozik egy teljesítményszint, amely az adott gyűjtemény RU-sebességkorlátját (a fenntartott feldolgozási sebességet) határozza meg. További információk: [Az Azure-előfizetések és -szolgáltatások korlátozásai, kvótái és megkötései][azure-limits].

- **Mindegyik dokumentumnak rendelkeznie kell egy attribútummal, amely alapján egyedileg azonosítható a gyűjteményen belül**. Ez az attribútum nem azonos a partíciókulccsal, amely azt határozza meg, hogy melyik gyűjteményben található a dokumentum. Egy gyűjtemény nagy számú dokumentumot tartalmazhat. Ezt elméletileg csak a dokumentumazonosító hossza korlátozza. A dokumentumazonosító legfeljebb 255 karakter hosszúságú lehet.

- **A dokumentumokra irányuló műveletek egy tranzakció környezetében lesznek végrehajtva. A tranzakciók hatóköre a dokumentumot tároló gyűjtemény.** Ha egy művelet meghiúsul, az általa végrehajtott feldolgozás vissza lesz vonva. Az épp feldolgozás alatt álló dokumentumokon végrehajtott módosítások pillanatképszinten el vannak különítve. Ez a mechanizmus biztosítja, hogy ha például egy új dokumentum létrehozására irányuló kérelem meghiúsul, az adatbázist egyidejűleg lekérdező többi felhasználó nem látja a félkész, majd eltávolított dokumentumot.

- **Az adatbázis-lekérdezések hatóköre szintén a gyűjtemény**. Egy lekérdezés csak egyetlen gyűjtemény adatait kérdezheti le. Ha több gyűjteményből kell lekérdeznie adatokat, mindegyik gyűjteményt külön-külön kell lekérdeznie, majd az eredményeket az alkalmazás kódjában egyesítenie.

- **A Cosmos DB támogatja a programozható elemek használatát, amelyek a gyűjteményben a dokumentumok mellett tárolhatóak**. Ezek lehetnek tárolt eljárások, felhasználó által definiált függvények és eseményindítók (JavaScript nyelven). Ezek az elemek a gyűjteményen belül az összes dokumentumot elérhetik. Ezen felül ezek az elemek a környezeti tranzakció hatókörében (egy, valamely dokumentumon végrehajtott létrehozás, törlés vagy csere művelet eredményeként aktiválódó trigger esetén) vagy egy új tranzakció indításával (egy, valamely kifejezett ügyfélkérelem eredményeképp futtatott tárolt eljárás esetén) futtatható. Ha egy programozható elem kódja kivételt dob, a tranzakció vissza lesz vonva. A tárolt eljárások és eseményindítók használatával tartható fent az integritás és a konzisztencia a dokumentumok közt, de a dokumentumoknak ugyanabba a gyűjteménybe kell tartoznia.

- **Törekedni kell rá, hogy az adatbázisban tárolni kívánt gyűjtemények ne haladják meg a teljesítményszintjeik által meghatározott feldolgozási sebességeket**. További információkat [az Azure Cosmos DB kérelemegységeit ismertető][cosmos-db-ru] cikkben talál. Ha várhatóan meghaladja majd ezeket a korlátokat, érdemes a gyűjteményeket különböző fiókokban lévő adatbázisokra szétosztani a terhelés csökkentése érdekében.

## <a name="partitioning-azure-search"></a>Az Azure Search particionálási

Az adatkeresési funkcionalitás a webalkalmazások biztosított leggyakoribb navigációs és felfedezési módszer. Segítségével a felhasználók gyorsan találhatják meg az erőforrásokat (például a termékeket egy e-kereskedelmi alkalmazásban) a keresési feltételek kombinációja alapján. Az Azure Search szolgáltatás teljes szöveges keresési funkcionalitást biztosít a webes tartalmakban, és olyan szolgáltatásokat kínál, mint a szövegkiegészítés, a közeli találatokon alapuló lekérdezési javaslatok és a jellemzőalapú navigáció. További információkért lásd: [Mi az az Azure Search?].

Az Azure Search a kereshető tartalmakat JSON-dokumentumokként tárolja egy adatbázisban. Definiálhat indexeket a dokumentumok kereshető mezőinek a meghatározásához, majd ezeket a definíciókat megadhatja az Azure Searchnek. Amikor egy felhasználó beküld egy keresési kérelmet, az Azure Search a megfelelő indexek használatával találja meg az egyező elemeket.

A versengés csökkentése érdekében az Azure Search által használt tárterület 1, 2, 3, 4, 6 vagy 12 partícióra osztható, és mindegyik partíció 6 példányban replikálható. A partíciók és a replikák számának szorzatát *keresési egységnek* (SU) nevezzük. Az Azure Search egyetlen példánya legfeljebb 36 SU-t tartalmazhat (tehát egy 12 partícióval rendelkező adatbázis legfeljebb 3 replikát támogat).

A számlázás a szolgáltatásban lefoglalt SU-k száma alapján történik. A kereshető tartalom mennyiségének vagy a keresési kérelmek számának növekedtével a meglévő Azure Search-példányok SU-inak száma is növelhető a többletterhelés kezelése érdekében. Maga az Azure Search a dokumentumokat egyenletesen osztja el a partíciók közt. A manuális particionálási stratégiák jelenleg nem támogatottak.

Mindegyik partíció legfeljebb 15 millió dokumentumot tartalmazhat vagy 300 GB tárterületet foglalhat el (amelyik érték kisebb). Legfeljebb 50 indexet hozhat létre. A szolgáltatás teljesítménye a dokumentumok összetettségétől, a rendelkezésre álló indexektől és a hálózati késés hatásaitól függően változik. Átlagosan egyetlen replika (1 SU) másodpercenként 15 lekérdezést (QPS) képes feldolgozni, bár javasolt saját adatok használatával teljesítménymérést végezni a feldolgozási sebesség pontosabb mérése érdekében. További információkért lásd: [Az Azure Search szolgáltatási korlátozásai].

> [!NOTE]
> A kereshető dokumentumokban tárolható adatok típusa egy korlátozott halmazba tartozhat: sztringek, logikai értékek, numerikus adatok, dátum és idő adatok és egyes földrajzi adatok. További információkért lásd [Támogatott adattípusok (Azure Search)] foglalkozó oldalt a Microsoft webhelyén.

Csak korlátozottan szabályozható, hogy az Azure Search hogyan particionálja az adatokat a szolgáltatás egyes példányaiban. Azonban egy globális környezetben esetleg növelhető a teljesítmény, valamint csökkenthető a késés és a versengés magának a szolgáltatásnak a particionálásával a következő stratégiák valamelyike mentén:

- Hozzon létre egy Azure Search-példányt minden egyes földrajzi régióban, és gondoskodjon róla, hogy az ügyfélalkalmazások a legközelebbi elérhető példányra legyenek irányítva. Ehhez a stratégiához a kereshető tartalmak frissítéseit időben kell replikálni a szolgáltatás minden példányán.

- Hozzon létre két szintet az Azure Searchben:

  - Egy helyi szolgáltatást minden egyes régióban, amely az adott régióban lévő felhasználók által leggyakrabban lekérdezett adatokat tartalmazza. A felhasználók a kérelmeket ide irányítva gyors, de korlátozott eredményeket kaphatnak.
  - Egy globális szolgáltatást, amely az összes adatot tartalmazza. A felhasználók a kérelmeket ide irányítva lassabb, de teljesebb eredményeket kaphatnak.

Ez a megközelítés akkor a legmegfelelőbb, ha a keresett adatok az egyes régiókban jelentősen eltérnek.

## <a name="partitioning-azure-redis-cache"></a>Az Azure Redis Cache particionálási

Az Azure Redis Cache egy, a Redis kulcs-érték adattáron alapuló megosztott gyorsítótárazási szolgáltatást biztosít a felhőben. Ahogy a neve is mutatja (cache = gyorsítótár), az Azure Redis Cache egy gyorsítótárazási megoldásnak készült. Csak átmeneti adatok tárolására készült, és nem állandó adattárként szolgál. Az Azure Redis Cache-t használó alkalmazásokat úgy érdemes kialakítani, hogy tovább működjenek, ha a gyorsítótár nem érhető el. Az Azure Redis Cache támogatja az elsődleges/másodlagos replikációt a magas rendelkezésre állás biztosítása érdekében, de jelenleg a gyorsítótár maximális mérete 53 GB-ra van korlátozva. Ha ennél több tárhely szükséges, további gyorsítótárakat kell létrehoznia. További információkért lásd: [Azure Redis Cache].

A Redis-adattárak particionálása szétosztja az adatokat a Redis szolgáltatás példányai között. Minden példány egy külön partíciót alkot. Az Azure Redis Cache a Redis-szolgáltatásokat egy előtár mögött kivonatolja, és nem teszi azokat közvetlenül közzé. A particionálás legegyszerűbben úgy valósítható meg, ha létrehoz több Azure Redis Cache-példányt, és az adatokat azok között osztja el.

Ez egyes adatelemekhez hozzárendelhet egy azonosítót (a partíciókulcsot), amely meghatározza, hogy melyik gyorsítótár tárolja az adatelemet. Az ügyfélalkalmazás logikája azután ennek az azonosítónak a használatával irányíthatja a kérelmeket a megfelelő partícióra. Ez a séma nagyon egyszerű, de ha a particionálási séma módosul (például további Azure Redis Cache-példányok létrehozása esetén), az ügyfélalkalmazások újrakonfigurálására lehet szükség.

A natív Redis (nem az Azure Redis Cache) támogatja a Redis-fürtözésen alapuló kiszolgálóoldali particionálást. Ebben a megközelítésben az adatok egyenletesen oszthatók meg a kiszolgálók között egy kivonatoló mechanizmus segítségével. Mindegyik Redis-kiszolgáló tárolja a partíción tárolt kivonatkulcsokat leíró metaadatokat, és arra vonatkozóan is tartalmaz információkat, hogy mely kivonatkulcsok találhatók a más kiszolgálókon lévő partíciókon.

Az ügyfélalkalmazások egyszerűen beküldik a kérelmeket valamelyik résztvevő Redis-kiszolgálóra (valószínűleg a legközelebbire). A Redis-kiszolgáló megvizsgálja az ügyfélkérelmet. Ha az helyileg megoldható, akkor elvégzi a kért műveletet. Ha nem oldható meg, továbbítja a kérést a megfelelő kiszolgálóra.

Erről a Redis-fürtözés használatával megvalósított modellről további részleteket a Redis webhelyén, a [Oktatóanyag Redis-fürtökhöz] tartalmazó oldalon talál. A Redis-fürtözés az ügyfélalkalmazások számára transzparens módon történik. A fürthöz további Redis-kiszolgálók is hozzáadhatók (és az adatok újraparticionálhatók) anélkül, hogy az ügyfeleket újra kellene konfigurálni.

> [!IMPORTANT]
> Az Azure Redis Cache jelenleg támogatja a Redis a Fürtszolgáltatás [prémium](/azure/azure-cache-for-redis/cache-how-to-premium-clustering) réteg csak.

A Redis használatával történő particionálásról további információkat a Redis webhelyén elérhető, [Particionálás: adatok felosztása több Redis-példány között] leíró oldalon talál. A szakasz további része feltételezi, hogy ügyféloldali vagy proxyval támogatott particionálást végez.

Ha az adatokat az Azure Redis Cache-sel tervezi particionálni, vegye figyelembe a következő szempontokat:

- Az Azure Redis Cache nem végleges adattárolásra lett kialakítva, ezért bármilyen particionálási sémát választ is, az alkalmazás kódjának az adatokat egy, a gyorsítótártól különböző helyről kell tudnia lekérdezni.

- A gyakran együtt használt adatokat érdemes egy partíción tartani. A Redis egy hatékony kulcs-érték tároló, amely több magas szinten optimalizált mechanizmust biztosít az adatok rendszerezéséhez. Ezek a mechanizmusok a következők lehetnek:
  - Egyszerű sztringek (legfeljebb 512 MB hosszúságú bináris adatok)
  - Összesített típusok, például listák (amelyek szolgálhatnak üzenetsorként vagy veremként)
  - Halmazok (rendezett és rendezetlen)
  - Kivonatok (amelyekkel csoportosíthatóak a kapcsolódó mezők, például az egy objektum mezőit jelölő elemek)

- Az összesített típusok összerendelhető segítségével több kapcsolódó mező ugyanazzal a kulccsal. A Redis-kulcsok listákat, halmazokat vagy kivonatokat azonosítanak, nem pedig az azokban tárolt adatelemeket. Ezek a típusok az Azure Redis Cache-ben mind elérhetők, és a Redis webhelyén, az [Adattípusok] lapon ismerhetőek meg. Például egy e-kereskedelmi rendszer az ügyfelek által feladott megrendeléseket követő részében az egyes ügyfelek adatai tárolhatóak egy Redis-kivonatban, amelynek a kulcsa lehet a felhasználó azonosítója. Mindegyik kivonat az ügyfél megrendelésazonosítóinak egy gyűjteményét tárolja. Egy külön Redis-halmaz tárolhatja a megrendeléseket, ezeket is kivonatként strukturálva, és kulcsként a megrendelésazonosítót használva. A 8. ábrán ez a struktúra látható. Vegye figyelembe, hogy a Redis nem valósít meg semmilyen hivatkozásintegritást, így a fejlesztő feladata, hogy fenntartsa az ügyfelek és a megrendelések kapcsolatát.

![Javasolt struktúra a Redis-tárolóban a megrendelések és adataik rögzítéséhez](./images/data-partitioning/RedisCustomersandOrders.png)

*8. ábra Javasolt struktúra a Redis-tárolóban a megrendelések és adataik rögzítéséhez.*

> [!NOTE]
> A Redisben a kulcsok bináris adatértékek (mint a Redis-sztringek), és legfeljebb 512 MB adatot tartalmazhatnak. Elméletileg a kulcsok szinte bármilyen információt tartalmazhatnak. Javasoljuk azonban egy egységes kulcselnevezési konvenció bevezetését, amely leírja az adatok típusát és azonosítja az entitásokat, de nem túl hosszú. Általános megközelítésként javasolt „entitástípus:azonosító” formátumú kulcsokat használni. Például a „user:99” névvel jelölheti egy 99-es azonosítójú ügyfél kulcsát.

- A vertikális particionálás megvalósítható, ha a különböző összesítésekben lévő kapcsolódó információkat ugyanabban az adatbázisban tárolja. Például egy e-kereskedelmi alkalmazásban tárolhatja a termékekkel kapcsolatos gyakran használt adatokat az egyik Redis-kivonatban, a kevésbé gyakran használt felhasználóadatokat pedig egy másikban. Mindkét kivonat használhatja ugyanazt a termékazonosítót a kulcs részeként. Használhatja például "termék: *Neurális hálózat*" (ahol *Neurális hálózat* a termékazonosító) a termékinformációkhoz és a "product_details: *Neurális hálózat*" a részletes adatokhoz. Ezzel a stratégiával csökkenthető a legtöbb lekérdezés által várhatóan beolvasott adatok mennyisége.

- A Redis-adattárak újraparticionálhatóak, de érdemes figyelembe venni, hogy ez összetett és időigényes feladat. A Redis-fürtszolgáltatás képes automatikusan újraparticionálni az adatokat, ez a funkció azonban az Azure Redis Cache-ben nem elérhető. Ezért a particionálási séma tervezésekor igyekezzen elegendő szabad helyet hagyni az egyes partíciókon az adatmennyiség idővel várható növekedésének megfelelően. Ne feledje azonban, hogy az Azure Redis Cache az adatok ideiglenes gyorsítótárazására szolgál, valamint hogy a gyorsítótárban tárolt adatok élettartama korlátozható az élettartam (TTL) érték megadásával. A viszonylag rövid életű adatok esetében az élettartam rövidre vehető, a statikus adatok esetében azonban sokkal hosszabbra. Lehetőleg ne tároljon nagy mennyiségű hosszabb élettartamú adatot a gyorsítótárban, amennyiben azok mennyisége feltehetően megtöltené a gyorsítótárat. Meghatározhat egy kiürítési szabályzatot, amely törli az adatokat az Azure Redis Cache-ből, ha a hely kiemelt fontosságú.

  > [!NOTE]
  > Az Azure Redis Cache használata esetén a gyorsítótár maximális mérete (250 MB és 53 GB között) a megfelelő tarifacsomag kiválasztásával határozható meg. Azonban a méret az Azure Redis Cache-gyorsítótár létrehozása után nem növelhető (és nem is csökkenthető).

- A Redis-kötegek és -tranzakciók nem terjedhetnek ki több kapcsolatra, így az egyes kötegek vagy tranzakciók által érintett összes adatot ugyanabban az adatbázisban (szegmensben) kell tárolni.

  > [!NOTE]
  > A Redis-tranzakcióban foglalt műveletszekvencia nem feltétlenül elemi jellegű. A tranzakciókat alkotó parancsokat a rendszer a futtatás előtt ellenőrzi, majd az üzenetsorba küldi azokat. Ha ebben a fázisban hiba történik, a rendszer a teljes üzenetsort elveti. Azonban miután a tranzakció sikeresen el lett küldve, az üzenetsorban lévő parancsok sorrendben lefutnak. Ha bármely parancs meghiúsul, csak az adott parancs futása áll le. A rendszer az üzenetsorban lévő összes megelőző és követő parancsot végrehajtja. További információkat a Redis webhelyén, a [tranzakciók] lapján talál.

- A Redis korlátozott számú elemi műveletet támogat. Az ilyen típusú műveletek közül több kulcs és érték használatát kizárólag az MGET és az MSET művelet támogatja. Az MGET műveletek kulcsok egy megadott listájához tartozó értékek gyűjteményét adják vissza, az MSET műveletek pedig ugyanezeket tárolják. Ha használni szeretné ezeket a műveleteket, az MSET és az MGET parancsok által hivatkozott kulcs-érték párokat ugyanabban az adatbázisban kell tárolnia.

## <a name="partitioning-azure-service-fabric"></a>Az Azure Service Fabric particionálási

Az Azure Service Fabric egy mikroszolgáltatás-platform, amely futtatókörnyezetet biztosít az elosztott alkalmazások felhőben végzett futtatására. A Service Fabric támogatja a .Net vendégrendszer futtatható fájljait, az állapotalapú és az állapotmentes szolgáltatásokat és a tárolókat. Az állapotalapú szolgáltatások egy [megbízható gyűjteményt][service-fabric-reliable-collections] biztosítanak az adatok kulcs-érték gyűjteményben való tartós tárolásához a Service Fabric-fürtben. Particionálási kulcsok megbízható gyűjteményekben vonatkozó stratégiákkal kapcsolatos további információkért lásd: [Irányelvek és javaslatok az Azure Service Fabric megbízható gyűjteményeihez].

### <a name="more-information"></a>További információ

- [Az Azure Service Fabric áttekintése] egy bevezető az Azure Service Fabric szolgáltatáshoz.

- [A Service Fabric Reliable Services particionálása] további információkat szolgáltat az Azure Service Fabric Reliable Services szolgáltatásairól.

## <a name="partitioning-azure-event-hubs"></a>Az Azure Event Hubs particionálási

Az [Azure Event Hubs][event-hubs] nagy léptékű adatstreamelésre lett kifejlesztve, és a particionálás integrált részét képezi a horizontális skálázás lehetővé tétele érdekében. Mindegyik felhasználó az üzenetstreamnek csak egy adott partícióját olvassa.

Az esemény-közzétevő csak a partíciókulcsot ismeri, azt a partíciót nem, amelyre az esemény közzé lesz téve. A kulcs és a partíció szétválasztása révén a küldőnek nem szükséges behatóan ismernie az alárendelt feldolgozási folyamatokat. (Eseményeket közvetlenül is küldhet adott partíciókra, az azonban általában nem ajánlott.)  

A partíciószám megadásakor hosszú távú szempontokat érdemes mérlegelni. Az eseményközpontok létrehozását követően a partíciók száma nem módosítható.

Az Event Hubs-partíciók használatával kapcsolatos további információkat a [Mi az Event Hubs?] című cikk tartalmazza.

A rendelkezésre állás és a konzisztencia közti kompromisszummal kapcsolatos megfontolásokat a [Rendelkezésre állás és konzisztencia az Event Hubsban] című cikk tartalmazza.

[Rendelkezésre állás és konzisztencia az Event Hubsban]: /azure/event-hubs/event-hubs-availability-and-consistency
[azure-limits]: /azure/azure-subscription-service-limits
[Azure Content Delivery Network]: /azure/cdn/cdn-overview
[Azure Redis Cache]: https://azure.microsoft.com/services/cache/
[Azure Storage Scalability and Performance Targets]: /azure/storage/storage-scalability-targets
[Az Azure Storage Table tervezési útmutatója]: /azure/storage/storage-table-design-guide
[Building a Polyglot Solution]: https://msdn.microsoft.com/library/dn313279.aspx
[cosmos-db-ru]: /azure/cosmos-db/request-units
[Data Access for Highly-Scalable Solutions: Using SQL, NoSQL, and Polyglot Persistence]: https://msdn.microsoft.com/library/dn271399.aspx
[Data consistency primer]: https://aka.ms/Data-Consistency-Primer
[Data Partitioning Guidance]: https://msdn.microsoft.com/library/dn589795.aspx
[Adattípusok]: https://redis.io/topics/data-types
[cosmosdb-sql-api]: /azure/cosmos-db/sql-api-introduction
[Elastic Database features overview]: /azure/sql-database/sql-database-elastic-scale-introduction
[event-hubs]: /azure/event-hubs
[Federations Migration Utility]: https://code.msdn.microsoft.com/vstudio/Federations-Migration-ce61e9c1
[Irányelvek és javaslatok az Azure Service Fabric megbízható gyűjteményeihez]: /azure/service-fabric/service-fabric-reliable-services-reliable-collections-guidelines
[Multi-shard querying]: /azure/sql-database/sql-database-elastic-scale-multishard-querying
[Az Azure Service Fabric áttekintése]: /azure/service-fabric/service-fabric-overview
[A Service Fabric Reliable Services particionálása]: /azure/service-fabric/service-fabric-concepts-partitioning
[Particionálás: adatok felosztása több Redis-példány között]: https://redis.io/topics/partitioning
[Entitáscsoport-tranzakciók végrehajtása]: /rest/api/storageservices/Performing-Entity-Group-Transactions
[Oktatóanyag Redis-fürtökhöz]: https://redis.io/topics/cluster-tutorial
[Running Redis on a CentOS Linux VM in Azure]: https://blogs.msdn.microsoft.com/tconte/2012/06/08/running-redis-on-a-centos-linux-vm-in-windows-azure/
[Scaling using the Elastic Database split-merge tool]: /azure/sql-database/sql-database-elastic-scale-overview-split-and-merge
[Using Azure Content Delivery Network]: /azure/cdn/cdn-create-new-endpoint
[Service Bus-kvóták]: /azure/service-bus-messaging/service-bus-quotas
[service-fabric-reliable-collections]: /azure/service-fabric/service-fabric-reliable-services-reliable-collections
[Az Azure Search szolgáltatási korlátozásai]:  /azure/search/search-limits-quotas-capacity
[Sharding pattern]: ../patterns/sharding.md
[Támogatott adattípusok (Azure Search)]:  https://msdn.microsoft.com/library/azure/dn798938.aspx
[Tranzakciók]: https://redis.io/topics/transactions
[Mi az Event Hubs?]: /azure/event-hubs/event-hubs-what-is-event-hubs
[Mi az az Azure Search?]: /azure/search/search-what-is-azure-search
[What is Azure SQL Database?]: /azure/sql-database/sql-database-technical-overview
[teljesítménycélokat]: /azure/storage/common/storage-scalability-targets
