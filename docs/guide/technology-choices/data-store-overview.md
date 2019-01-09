---
title: A megfelelő adattároló kiválasztása
titleSuffix: Azure Application Architecture Guide
description: Áttekintés az Azure-beli adattárak kiválasztása.
author: MikeWasson
ms.date: 06/01/2018
ms.custom: seojan19
ms.openlocfilehash: 472b01a3316edd92f069e9395e8058fef3ef1fa6
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/08/2019
ms.locfileid: "54114266"
---
# <a name="choose-the-right-data-store"></a>A megfelelő adattároló kiválasztása

A modern üzleti rendszerek egyre nagyobb adatmennyiségeket kezelnek. Az adatok külső szolgáltatásokból olvashatók be, vagy maga a rendszer, esetleg a felhasználók is létrehozhatják azokat. Ezek az adatkészletek rendkívül változatos jellemzőkkel és feldolgozási követelményekkel rendelkeznek. A vállalatok az adatok segítségével értékelik a tendenciákat, aktiválják az üzleti folyamatokat, naplózzák az elvégzett műveleteket, elemzik a vásárlói magatartást, stb.

Ez a sokféleség azt jelenti, hogy egyetlen adattár használata a legtöbb esetben nem megfelelő megoldás. Ehelyett gyakran célszerűbb a különféle adattípusokat különböző adattárakban tárolni, amelyek mindegyike az adott számítási feladathoz vagy használati mintázathoz van igazítva. A *többnyelvű perzisztencia* kifejezés az adattárolási technológiákat vegyesen használó megoldások leírására használatos.

Az igényeinek megfelelő adattároló kiválasztása a tervezés egyik kulcsfontosságú döntése. Az SQL- és NoSQL-adatbázisok esetén több száz megvalósítási mód lehetséges. Az adattárakat általában az alapján kategorizáljuk, hogyan strukturálják az adatokat, illetve milyen műveleteket támogatnak. Ebben a cikkben a leggyakoribb tárolási modellek közül többet is ismertetünk. Vegye figyelembe, hogy egy adott adattároló technológia több tárolómodellt is támogathat. Például egy relációsadatbázis-kezelő rendszer (RDBMS) a kulcs/érték vagy a gráf típusú tárolókat is támogathatja. Általánosan megfigyelhető tendencia a *többmodelles* támogatás, amely esetben egy adatbázisrendszer egyszerre több modellt is támogat. Ennek ellenére hasznos a különböző modellek magas szintű működésének megismerése.

Egy adott kategóriában nem minden adattár biztosítja ugyanazt a szolgáltatáskészletet. A legtöbb adattár kiszolgálóoldali funkciókat nyújt az adatok lekérdezéséhez és feldolgozásához. Egyes esetekben ez a funkció be van építve az adattár motorjába. Más esetekben az adattárolási és feldolgozási képességek elkülönülnek egymástól, és a feldolgozás és elemzés terén számos lehetőség közül választhat. Az adattárak különböző programozási és felügyeleti felületet is támogatnak.

Általánosan érvényes szabály, hogy elsőként azt kell eldöntenie, melyik tárolási modell felel meg legjobban az igényeinek. Ezután az adott kategórián belül válasszon ki egy adattárat a szolgáltatáskészlet, a költségek és az egyszerű kezelhetőséghez hasonló szempontok alapján.

## <a name="relational-database-management-systems"></a>Relációsadatbázis-kezelő rendszerek

A relációs adatbázisok kétdimenziós, sorokat és oszlopokat tartalmazó táblák sorozataként rendszerezik az adatokat. Minden tábla saját oszlopokat tartalmaz, és egy tábla minden sora ugyanazon oszlopkészletből áll. Ez a modell matematikai alapú, és a legtöbb szállító a Structured Query Language (SQL – Strukturált lekérdező nyelv) egyik dialektusát használja az adatok lekérdezéséhez és kezeléséhez. Egy RDBMS általában egy egységes tranzakciójú mechanizmust használ, amely az információfrissítés szempontjából megfelel az ACID-modellnek (Atomic, Consistent, Isolated, Durable – atomitás, konzisztencia, izoláció, tartósság).

Egy RDBMS általában egy írásiséma-modellt támogat, amelyben az adatok struktúrája előre meghatározott, és minden olvasási vagy írási műveletnek a sémát kell használnia. Ezzel ellentétes a legtöbb NoSQL-adattár működése, különösen a kulcs/érték típusúaké, amelyek esetén az írásiséma-modell feltételezi, hogy az ügyfél saját értelmező sémáját fogja használni az adatbázisból érkező adatokon, és független az éppen írt adatok formátumától.

Egy RDBMS akkor igazán hasznos, ha fontos a garantáltan erős konzisztencia, ahol minden módosítás atomi jellegű, a tranzakciók pedig mindig konzisztens állapotban tartják az adatokat. Az alapul szolgáló struktúrák horizontális felskálázása azonban a tárterület felosztásával és a több gépen végzett feldolgozással nem lehetséges. Emellett az RDBMS-ben tárolt információkat relációs struktúrába kell rendezni az alábbi normalizálási folyamat lépéseit követve. Bár a folyamat jól ismert, mégis a hatékonyság csökkenéséhez vezethet, mivel a logikai entitásokat először külön táblákban szereplő sorokra kell szétbontani, majd a lekérdezések futtatásakor az adatokat újból össze kell állítani.

Kapcsolódó Azure-szolgáltatás:

- [Azure SQL Database][sql-db]
- [Azure Database for MySQL][mysql]
- [Azure Database for PostgreSQL][postgres]

## <a name="keyvalue-stores"></a>Kulcs/érték tárolók

A kulcs/érték tároló lényegében egy nagy kivonattábla. Minden egyes adatértékhez egy egyedi kulcs társul, a kulcs/érték tároló pedig ezt a kulcsot használja az adatok egy megfelelő kivonatoló függvénnyel való tárolásához. A kivonatoló algoritmus a kivonatolt kulcsok az adattárban való egyenletes elosztására van kiválasztva.

A legtöbb kulcs/érték tároló csak az egyszerű lekérdezési, beszúrási és törlési műveleteket támogatja. Egy érték (akár részleges, akár teljes) módosításához az alkalmazásnak a teljes értékre vonatkozóan felül kell írnia a meglévő adatokat. A legtöbb megvalósításban egyetlen érték olvasása vagy írása atomi műveletnek számít. Ha az érték túl nagy, az írás hosszabb időt is igénybe vehet.

Az alkalmazás tetszőleges adatokat tárolhat egy értékekből álló készletként, bár egyes kulcs/érték tárolók korlátozzák az értékek maximális méretét. A tárolt értékek a tárolórendszer szoftvere számára nem átlátszók. A sémaadatok biztosítása és értelmezése az alkalmazás feladata. Az értékek alapvetően blobok, a kulcs/érték tároló pedig egyszerűen kulcsonként olvassa be vagy tárolja az értéket.

![Egy kulcs-érték tároló ábrája](./images/key-value.png)

A kulcs/érték tárolók kifejezetten az egyszerű kereséseket végző alkalmazásokhoz vannak optimalizálva, azonban kevésbé megfelelőek az olyan rendszerek számára, amelyek az adatokat különböző kulcs/érték tárolókból kérik le. A kulcs/érték tárolók emellett nincsenek optimalizálva olyan forgatókönyvekre, amelyekben az értékenkénti lekérdezés a fontos, nem pedig a kizárólag kulcsokon alapuló keresések végrehajtása. Például egy relációs adatbázisban a WHERE záradék használatával megtalálhat egy rekordot, a kulcs/érték tárolók azonban nem rendelkeznek ilyen típusú, értékekre vonatkozó keresési képességekkel.

Egyetlen kulcs/érték tároló lehet rendkívüli mértékben skálázható, mivel az adattár könnyedén feloszthatja az adatokat több, külön gépeken található csomópont között.

Kapcsolódó Azure-szolgáltatások:

- [Cosmos DB][cosmosdb]
- [Azure Redis Cache][redis-cache]

## <a name="document-databases"></a>Dokumentum-adatbázisok

A dokumentum-adatbázisok koncepciója a kulcs/érték tárolókéhoz hasonló azzal a kivétellel, hogy egy nevesített mezőkből és adatokból (azaz dokumentumokból) álló egy gyűjteményt tárolnak, amelynek mindegyike lehet egyszerű skaláris elem vagy összetett elem, például lista vagy gyermek gyűjtemény. Egy dokumentum mezőiben szereplő adatok többféle módon is kódolhatók, például XML, YAML, JSON, BSON, de tárolásuk akár egyszerű szövegként is történhet. A kulcs/érték tárolókkal ellentétben a dokumentumokban szereplő mezők elérhetők a tárhelykezelő rendszer számára, ezáltal az alkalmazások a mezőkben szereplő értékek segítségével lekérdezhetik és szűrhetik az adatokat.

Általában egy dokumentum egy entitás összes adatát tartalmazza. Az entitást alkotó elemek az alkalmazástól függenek. Például egy entitás tartalmazhatja egy ügyfél vagy egy rendelés adatait, vagy akár a kettő kombinációját is. Egyetlen dokumentum olyan információkat is tartalmazhat, amely RDBMS esetén több relációs tábla között lenne elosztva.

A dokumentumtároló nem igényli, hogy minden dokumentum ugyanazzal a struktúrával rendelkezzen. Ez a szabad formátumú megközelítés nagy fokú rugalmasságot biztosít. Az alkalmazások az üzleti igények változása alapján különböző adatokat tárolhatnak a dokumentumokban.

![A dokumentumtároló ábrája](./images/document.png)

Az alkalmazás a dokumentumkulcs segítségével kérhet le dokumentumokat. Ez a dokumentum egyedi azonosítója, amely az adatok egyenletes elosztása érdekében gyakran kivonatolt. Néhány dokumentum-adatbázis automatikusan létrehozza a dokumentumkulcsot. Mások lehetővé teszik, hogy a dokumentum egyik attribútumát adja meg kulcsként. Az alkalmazás emellett dokumentumokat is lekérdezhet egy vagy több mező alapján. Néhány dokumentum-adatbázis támogatja az indexelést a dokumentumok egy vagy több indexelt mező alapján történő gyors keresése érdekében.

Számos dokumentum-adatbázis támogatja a helyi frissítést, amely révén az alkalmazások a teljes dokumentum átírása nélkül módosíthatják az adott mezők értékeit egy dokumentumban. Az egy dokumentum több mezőjén végrehajtott olvasási és írási műveletek általában atomi jellegűek.

Kapcsolódó Azure-szolgáltatás: [Cosmos DB][cosmosdb]

## <a name="graph-databases"></a>Gráfadatbázisok

A gráfadatbázisok kétféle típusú információt tárolnak: csomópontokat és éleket. A csomópontok entitásokként is felfoghatók. Az élek a csomópontok közötti kapcsolatokat határozzák meg. Mind a csomópontok, mind az élek rendelkezhetnek olyan tulajdonságokkal, amelyek a táblák oszlopaihoz hasonlóan információt nyújtanak az adott csomópontról vagy élről. A szegélyeknek is lehet iránya, amely a kapcsolat természetét jelöli.

A gráfadatbázisok célja az, hogy az alkalmazások számára lehetővé tegyék a csomópontok és élek hálózatát bejáró lekérdezések végrehajtását, illetve hogy elemezzék az entitások közötti kapcsolatokat. Az alábbi ábrán látható gráfként strukturálva egy vállalat személyzeti adatbázisa. Az entitások az alkalmazottak és a részlegek, az élek pedig a jelentéskészítési kapcsolatokat jelzik, valamint azt, hogy melyik alkalmazott melyik részlegen dolgozik. Ezen a gráfon az éleken látható nyilak a kapcsolatok irányát jelzik.

![A dokumentum-adatbázis ábrája](./images/graph.png)

Ezzel a struktúrával egyértelműen hajthatók végre a következőkhöz hasonló lekérdezések: „Minden alkalmazott megkeresése, aki közvetlenül vagy közvetetten Sárának jelent” vagy „Ki dolgozik Jánossal egy részlegen?” A számos entitást és kapcsolatot tartalmazó, nagy méretű gráfok esetén nagyon gyorsan hajthat végre rendkívül összetett elemzéseket. Sok gráfadatbázis használ olyan lekérdezési nyelvet, amellyel egy kapcsolatokból álló hálózat hatékonyan bejárható.

Kapcsolódó Azure-szolgáltatás: [Cosmos DB][cosmosdb]

## <a name="column-family-databases"></a>Oszlopcsalád-adatbázisok

Az oszlopcsalád-adatbázisok sorokba és oszlopokba rendezik az adatokat. Legegyszerűbb formájában az oszlopcsalád-adatbázis a relációs adatbázishoz nagyon hasonlónak tűnhet, legalábbis a koncepciót tekintve. Az oszlopcsalád-adatbázisok igazi előnye a ritka adatok strukturálásának denormalizált megközelítésében rejlik.

Az oszlopcsalád-adatbázisok felfoghatók úgy, mintha a sorokat és oszlopokat tartalmazó táblázatba foglalt adatokról lenne szó, azonban az oszlopok *oszlopcsaládoknak* nevezett csoportokra vannak felosztva. Minden oszlopcsalád olyan oszlopokból álló készletet tartalmaz, amelyek logikailag kapcsolódnak egymáshoz, és beolvasásuk vagy módosításuk általában egy egységként történik. Az egyéb, külön elérhető adatok külön oszlopcsaládokban tárolhatók. Egy oszlopcsaládon belül az új oszlopok dinamikusan hozzáadhatók, a sorok pedig ritkák is lehetnek (ez azt jelenti, hogy a soroknak nem kell minden oszlophoz értéket tartalmazniuk).

Az alábbi ábrán látható példában két oszlopcsalád szerepel: `Identity` és `Contact Info`. Az egyetlen entitás adatai minden egyes oszlopcsaládban ugyanazt a sorkulcsot tartalmazzák. Ez a struktúra, amelyben az oszlopcsaládban szereplő egyes objektumokhoz tartozó sorok dinamikusan változhatnak, az oszlopcsalád-megközelítés egyik fontos előnye. Ezáltal ez az adattárolási forma kifejezetten alkalmas a strukturált, ideiglenes adatok tárolására.

![Oszlopcsalád adatbázisok ábrája](./images/column-family.png)

A kulcs/érték tárolókkal vagy a dokumentum-adatbázisokkal ellentétben a legtöbb oszlopcsalád-adatbázis kulcs szerinti sorrendben tárolja az adatokat, nem pedig kivonatok számítása alapján. Számos megvalósítás esetén létrehozhat indexeket egy oszlopcsalád adott oszlopain. Az indexek révén az oszlop értéke szerint kérheti le az adatokat a sorkulcs helyett.

Az egy soron végrehajtott olvasási és írási műveletek általában atomi jellegűek egyetlen oszlopcsalád esetén, azonban egyes megvalósítások az egész, több oszlopcsaládon átívelő sorban biztosítják az atomitást.

Kapcsolódó Azure-szolgáltatás: [A HDInsight HBase][hbase]

## <a name="data-analytics"></a>Adatelemzés

Az adatelemzési tárolók nagymértékben párhuzamos megoldásokat biztosítanak az adatok betöltéséhez, tárolásához és elemzéséhez. Ezek az adatok egy megosztást mellőző architektúra használatával több kiszolgáló között vannak szétosztva, a maximális skálázhatóság és a függőségek minimalizálása érdekében. Az adatok nagy valószínűséggel nem statikusak, így ezeknek az adattáraknak képesnek kell lenniük nagy mennyiségű információk kezelésére, amelyek számos különböző formátumban érkeznek több streamből, miközben az új lekérdezések feldolgozása továbbra is folyik.

Kapcsolódó Azure-szolgáltatások:

- [SQL Data Warehouse][sql-dw]
- [Azure Data Lake][data-lake]

## <a name="search-engine-databases"></a>Keresőmotor-adatbázisok  

A keresőmotor-adatbázisok támogatják a külső adattárakban és szolgáltatásokban tárolt adatok keresését. A keresőmotor-adatbázisok nagy méretű adatkötetek indexeléséhez használhatók, valamint közel valós idejű hozzáférést biztosítanak az indexekhez. Bár a keresőmotor-adatbázisokra gyakran a web szinonimájaként gondolnak, számos nagy méretű rendszer használja azokat saját adatbázisain is a strukturált, ad-hoc jellegű keresési képességek miatt.

A keresőmotor-adatbázisok alapvető jellemzője az információk gyors tárolásának és indexelésének képessége, valamint hogy rövid válaszidővel végzik a keresési kéréseket. Az indexek lehetnek többdimenziósak, és támogathatják a szabad szöveges keresést a nagy kötegekből álló szöveges adatok esetén. Az indexelés elvégezhető egy lekérési modell használatával, amelyet a keresőmotor-adatbázis aktivál, vagy egy leküldéses modell használatával, amely külső alkalmazáskóddal indítható.

A keresés lehet pontos vagy intelligens. Az intelligens keresés olyan dokumentumokat keres, amelyek megfelelnek egy adott feltételkészletnek, és az egyezés mértékét is kiszámítja. Néhány keresőmotor támogatja az olyan nyelvi elemzéseket is, amelyek szinonimák, kategóriakiterjesztések (például `dogs` megfeleltetése `pets`-nek), szótővizsgálat (azonos szótővel rendelkező szavak megfeleltetése) alapján adnak vissza találatokat.

Kapcsolódó Azure-szolgáltatás: [Az Azure Search][search]

## <a name="time-series-databases"></a>Idősorozat-adatbázisok

Az idősorozat-adatok időpont alapján rendszerezett értékekből álló készletek, az idősorozat-adatbázisok pedig ehhez az adattípushoz vannak optimalizálva. Az idősorozat-adatbázisok nagyon nagy számú írást támogatnak, mivel általában nagy mennyiségű adatokat gyűjtenek sok forrásból és valós időben. A frissítések ritkák, a törlések pedig a legtöbbször tömeges műveletként történnek. Annak ellenére, hogy az idősorozat-adatbázisokba írt rekordok mérete általában kicsi, gyakran előfordulnak nagy méretű rekordok, valamint az adatok összmérete is gyors ütemben növekedhet.

Az idősorozat-adatbázisok kiválóan alkalmasak telemetriaadatok tárolására. A forgatókönyvek IoT-érzékelőket vagy alkalmazás-/rendszerszámlálókat is tartalmaznak.

Kapcsolódó Azure-szolgáltatás: [A Time Series Insights][time-series]

## <a name="object-storage"></a>Objektumtár  

Az objektumtároló nagy méretű bináris objektumok (képek, fájlok, video- és audiostreamek, nagy méretű alkalmazás-adatobjektumok és dokumentumok, virtuális gépek lemezképei) tárolására van optimalizálva. Az ezen tárolótípusokban szereplő objektumokat a tárolt adatok, metaadatok és az objektum eléréséhez szükséges egyedi azonosítók alkotják. Az objektum lehetővé teszi a strukturálatlan adatok rendkívüli mennyiségeinek kezelését.

Kapcsolódó Azure-szolgáltatás: [A BLOB Storage][blob]

## <a name="shared-files"></a>Megosztott fájlok

Időnként az egyszerű, egybesimított fájlok használata az információk tárolásának és lekérésének leghatékonyabb módja. A fájlmegosztások használatával a fájlok egy egész hálózaton elérhetők. A megfelelő biztonsági és egyidejű hozzáférési vezérlőmechanizmusok jelenlétében az adatok ilyen módon történő megosztása révén a megosztott szolgáltatások nagymértékben skálázható adathozzáférést biztosíthatnak az alapvető, alacsony szintű műveletek (például egyszerű olvasási és írási kérések) végrehajtásához.

Kapcsolódó Azure-szolgáltatás: [A File Storage][file-storage]

<!-- links -->

[blob]: https://azure.microsoft.com/services/storage/blobs/
[cosmosdb]: https://azure.microsoft.com/services/cosmos-db/
[data-lake]: https://azure.microsoft.com/solutions/data-lake/
[file-storage]: https://azure.microsoft.com/services/storage/files/
[hbase]: /azure/hdinsight/hdinsight-hbase-overview
[mysql]: https://azure.microsoft.com/services/mysql/
[postgres]: https://azure.microsoft.com/services/postgresql/
[redis-cache]: https://azure.microsoft.com/services/cache/
[search]: https://azure.microsoft.com/services/search/
[sql-db]: https://azure.microsoft.com/services/sql-database
[sql-dw]: https://azure.microsoft.com/services/sql-data-warehouse/
[time-series]: https://azure.microsoft.com/services/time-series-insights/
