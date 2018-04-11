---
title: Válassza ki a megfelelő tárolót
description: Adattároló kiválasztása az Azure-ban – áttekintés
author: MikeWasson
ms.openlocfilehash: 3a5780c4a2dbd8a41e9c7bfa7f68d8a7916a7374
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="choose-the-right-data-store"></a>Válassza ki a megfelelő tárolót

A modern üzleti rendszerek kezelése egyre nagy mennyiségű adatot. Adatok előfordulhat, hogy külső szolgáltatásokból okozhatnak, a rendszer által létrehozott vagy felhasználók által létrehozott. Előfordulhat, hogy ezek az adathalmazok nagyon eltérőek jellemzőit és ügyféloldali bővítmények feldolgozási követelményeivel. Vállalatok adatok segítségével felmérheti a trendeket, indítás, üzleti folyamatok, a műveletek naplózási, elemzése a felhasználói viselkedés, és számos egyéb elemet. 

Ez sokfélesége azt jelenti, hogy egyetlen adattárolóban általában nem ajánlott megközelítés. Ehelyett érdemes gyakran különböző típusú adatok tárolhatók különböző adattárolókhoz, minden egyes arra irányul, hogy egy adott munkaterhelés vagy használati minta felé. A kifejezés *polyglot adatmegőrzési* vegyesen adatok tárolási technológiákat használó megoldásokkal leírására használatos.

A tervezési döntés a megfelelő tárolót a követelmények teljesítéséhez kiválasztása. Nincsenek szó százait megvalósítások SQL és a NoSQL-adatbázisok közül választhat. Adattároló gyakran szerint vannak kategóriába által hogyan azok szerkezeti adatokhoz és milyen műveleteket támogatják-e. Ez a cikk ismerteti a leggyakoribb tárolási modellek számos. Vegye figyelembe, hogy egy adott adattároló technológia támogathatja több tároló modellek. Például egy relációs adatbázis-kezelő rendszerek (RDBMS) is rendelkezhetnek kulcs/érték vagy a diagramon. Valójában nincs általános trendje úgynevezett *multimodel* támogatása, ha egy önálló adatbázis rendszer támogatja-e több modellt. De továbbra is hasznos lehet tudni, hogy a különböző modell magas szinten. 

Egy adott kategória nem minden adattárolókhoz adja meg az azonos szolgáltatáskészlet. A legtöbb adattárolókhoz ellátni a kiszolgálóoldali adatok lekérdezés és a folyamat. Egyes esetekben ez a funkció van beépítve az adatok tárolási összetevőt. Más esetekben az tárolás és feldolgozás képességek elkülönül egymástól, és előfordulhat, hogy számos lehetőség közül választhat, feldolgozása és elemzése. Adatok tárolók is támogatják a különböző programozott és felügyeleti felületek. 

Általában, először meg kell válaszolnia figyelembe véve, amelyre a tárolási modell bizonyul a legalkalmasabbnak igényeinek. Fontolja meg egy adott adattár adott kategóriába, például szolgáltatáskészlet, a költség, és a könnyű kezelés tényezők alapján.

## <a name="relational-database-management-systems"></a>Relációs adatbázis-kezelő rendszerek

Relációs adatbázisok adatok rendezése kétdimenziós táblákon, amelyekhez a sorok és oszlopok sorozataként. Minden tábla saját oszlopot tartalmaz, és egy tábla minden sorára be van állítva a azonos az oszlopok. Ez a modell matematikailag alapul, és a legtöbb szállító nyújt a Structured Query Language (SQL) nyelvjárást beolvasása és adatok kezelése. Egy RDBMS általában bevezet egy tranzakciós úton konzisztens mechanizmus, amely megfelel a sav (atomi, Consistent, elkülönítés, tartós) modelljét adatainak frissítésére. 

Egy RDBMS általában egy séma módosításkor modell, ahol a adatszerkezet időben van definiálva, és minden írási vagy olvasási műveletek kell használnia a séma támogatja. Ellentétben a legtöbb NoSQL-adattároló, különösen kulcs/érték típusok, ahol a séma-a-olvasási modell feltételezi, hogy az ügyfél a adatbázisból származó adatokat a saját interpretive séma előíró lesz, és független írt adatok formátuma.

Egy RDBMS akkor nagyon hasznos, ha az erős konzisztencia biztosítja fontos &mdash; ha összes módosítás atomi, és tranzakciók mindig hagyja meg az adatokat konzisztens állapotú. Azonban az alapul szolgáló struktúrák nem alkalmasak az tárolási terjesztése és keresztül gépek kiterjesztése. Emellett egy RDBMS tárolt információ kell állítani relációs struktúrába a normalizálási folyamatot követve. Amíg ez a folyamat magától értetődő, akkor előfordulhat, hogy a hatékonyság, hiánya miatt nem fejtheti vissza a logikai entitások sorokba külön táblázatban, hogy szükség van, és majd később az adatok, lekérdezések futtatásakor. 

Megfelelő Azure szolgáltatás: 

- [Azure SQL Database][sql-db]
- [Azure MySQL-adatbázis][mysql]
- [Azure PostgreSQL-adatbázishoz][postgres]

## <a name="keyvalue-stores"></a>Kulcs-érték tárolók

A kulcs/érték tároló lényegében egy nagy kivonattábla. Minden egyes adatérték társítandó egyedi kulcs, és a kulcs-érték tároló ezt a kulcsot használja az adatok tárolásához egy megfelelő kivonatoló függvénnyel. A kivonatoló funkció így biztosít az még akkor is, terjesztési kivonatolt kulcsok adatok tárolására van kiválasztva. 

Legtöbb kulcs/érték csak támogatási egyszerű lekérdezés tárolja, beszúrási és törlési műveletek. Egy érték módosítása (részben vagy teljesen), az alkalmazás felül kell írnia a meglévő adatok a teljes érték. A legtöbb implementációkban olvasása vagy írása egyetlen értéket egy atomi művelet. Ha az érték túl nagy, írás hosszabb időt vehet igénybe. 

Egy alkalmazás tetszőleges adatait tárolhatja értékek csoportjaként, bár egyes kulcs-érték tárolók ugyanazok a korlátozások értékek maximális mérete. A tárolt értékei nem átlátszó, a tárolási rendszer szoftverre. A sémaadatok megadott legyen, és az alkalmazás értelmezi. Alapvetően értékei a blobok és a kulcs-érték tároló egyszerűen kéri le vagy a kulcs értékét tárolja. 

![](./images/key-value.png)

Kulcs-érték tárolók nagyon egyszerű keresések alkalmazások vannak optimalizálva, de különböző kulcs-érték tárolók között lekérdezni adatokat igénylő rendszerek kevésbé alkalmasak. Kulcs-érték tárolók is nem optimalizált forgatókönyvek, ahol értékkel lekérdezése fontos, hanem végrehajtott keresési műveletek végrehajtása csak kulcsok alapján. Például egy relációs adatbázisban, WHERE záradék használatával is található a rekord, de kulcs/érték tárolók általában nem rendelkezik ilyen típusú értékeket az keresési funkciójának könnyebb működése.

Egy-egy kulcs-érték tároló rendkívül méretezhető, lehet, mert az adattár is könnyen szét adatok több csomópontok külön számítógépeken. 

Azure-szolgáltatásokhoz kapcsolódó: 

- [A cosmos DB][cosmosdb]
- [Azure Redis gyorsítótár][redis-cache]

## <a name="document-databases"></a>Dokumentum-adatbázisokat

A dokumentum-adatbázis egy kulcs-érték tároló fogalmilag hasonló, azzal a különbséggel, hogy gyűjteménye tárolja elnevezett mezőkből és adatok (más néven dokumentumok), amelyek mindegyike skaláris elemek egyszerű vagy összetett elemek, mint például a listák és a gyermekgyűjteményeknél. Az egy dokumentum mezők kódolható az sokféleképpen, XML, YAM, JSON, BSON együtt, vagy akár egyszerű szövegként tárolt. Kulcs-érték tárolók, ellentétben a dokumentumokban lévő mezők érhetők el a felügyelet tárolórendszer, lekérdezés és a szűrő adatait az alkalmazás engedélyezése a mezők az értékek használatával. 

Általában a dokumentum egy entitás a teljes adatokat tartalmaz. Milyen elemek képeznek egy entitás az adott alkalmazás. Például egy entitás tartalmazhatnak mindkettőt, vagy az ügyfél, a megrendelést részleteit. Egyetlen dokumentum lenne egy RDBMS több relációs táblák kell elosztva információkat is tartalmaznak. 

A dokumentum store nem igényli, hogy az összes dokumentum hasonló struktúrával rendelkezik. Ez a megközelítés szabadkézi nagyfokú rugalmasságot biztosít. Alkalmazások, üzleti követelmények módosítása különböző adatot tárolhat dokumentumokat.

![](./images/document.png)

Az alkalmazás dokumentumok lekérheti a dokumentum kulcs használatával. Ez a dokumentum, amely a gyakran kivonatolt, adatok egyenletes elosztása érdekében egyedi azonosítója. Néhány dokumentum-adatbázis automatikusan hozza létre a dokumentum kulcsot. Mások számára lehetővé teszik a dokumentum kívánja használni, mint a key attribútum adja meg. Az alkalmazás egy vagy több mező értéke alapján dokumentumokat is lekérdezheti. Néhány dokumentum-adatbázisokat támogatja, lehetővé teszi egy vagy több indexelt mezők alapján dokumentumok gyors keresési indexelő. 

Sok dokumentum-adatbázis helyi frissítések, egy alkalmazás egy dokumentumot a megadott mező értékének módosítása nélkül írja át a teljes dokumentum engedélyezése támogatja. Olvasási és írási műveletek keresztül egy dokumentumban több mező esetében általában atomi.

Megfelelő Azure-szolgáltatások: [Cosmos DB][cosmosdb]

## <a name="graph-databases"></a>Graph-adatbázisok

Egy grafikonon adatbázis tárolja a kétféle típusú adatokat, a csomópontok és a szélén. Az eltolásokat tekintheti csomópontok entitásként. Adja meg a csomópontok közötti kapcsolatokat, amelyek szélén. Csomópontok és a szélek rendelkezhet, amelyek információval szolgálnak, hogy a csomópont vagy a táblázatban levő oszlopkészleteket hasonló biztonsági tulajdonságai. Szegélyek is lehet a kapcsolat jellege irányt.

Egy grafikonon adatbázis az a célja, hogy lehetővé tegye egy alkalmazást, a hálózati csomópont és szélek áthaladó lekérdezések hatékonyan végrehajtásához, és elemezheti az entitások közötti kapcsolatok. A szervezetek személyzeti adatbázis strukturált, mint egy grafikonon kövesse ábrán látható. Az entitások az alkalmazottak és szervezeti egységek, és széleinek jelzik, hogy jelentéskészítési kapcsolatokat és a részleg, mely az alkalmazottak munka. Az ehhez a diagramhoz a a szélén nyilak a kapcsolat irányát.
 
![](./images/graph.png)

Ez egyszerű végrehajtásához lekéri például struktúra teszi "található összes beosztottak közvetlenül vagy közvetve Sarah" vagy "Ki működik John részleghez?" Az entitások és kapcsolatok sok nagy diagramok nagyon gyorsan végezhet nagyon összetett elemzések. Sok diagram adatbázis lekérdezésnyelvet, amely hatékonyan haladnak át a hálózati kapcsolatok segítségével adja meg. 

Megfelelő Azure-szolgáltatások: [Cosmos DB][cosmosdb]

## <a name="column-family-databases"></a>Oszlop-család adatbázisok

Egy oszlop-család adatbázis adatok sorok és oszlopok rendezi. Legegyszerűbb formájukban oszlop-család adatbázis jelenhet meg nagyon hasonlít egy relációs adatbázisban, legalább hasonlóak. A valós oszlop-család adatbázis rugalmasságuknak köszönhetően a denormalizált megközelítéssel szerkezetének kialakítása ritka adatokat is. 

Az eltolásokat tekintheti oszlop-család adatbázis sorok és oszlopok táblázatos adatokat, de az oszlopok néven csoportra oszlanak *oszlopcsaláddal*. Minden oszlop termékcsalád oszlopkészleteket, amelyek logikailag egymáshoz kapcsolódó és általában beolvasni vagy egységként választéka tartalmazza. Külön-külön egyéb adatokhoz külön oszlopcsaláddal is tárolható. Egy oszlop családba tartozó új oszlopok is hozzáadhatók dinamikusan, és lehet, hogy a sorok ritka (Ez azt jelenti, hogy a sor nem kell minden oszlop értéke).

Az alábbi ábrán látható egy példa két oszlopcsaláddal `Identity` és `Contact Info`. Egyetlen entitás az adatok minden oszlop termékcsalád sor ugyanazzal a kulccsal rendelkezik. Ez a struktúra, ahol dinamikusan változhat a sorok összes adott objektum egy oszlop családba tartozó, egy fontos előnye, hogy az oszlop-család módszert használja, így ezt az űrlapot magas olyan alkalmas strukturált, "volatile" adattárolás adattár.

![](./images/column-family.png) 

Egy kulcs-érték tárolóban és a dokumentum-adatbázis legtöbb oszlop-család adatbázis tárolja adatait kulcs sorrendben, nem úgy kivonatát. Számos implementáció eleve indexek létrehozása az oszlop-család adott oszlopok keresztül teszi lehetővé. Indexek lehetővé teszik, hogy az oszlopok érték, hanem a sorkulcsa adatainak beolvasása.

Egy sor olvasási és írási műveletek esetében általában atomic csak egy oszlop-címcsaládot, a, bár egyes megvalósítások atomicity biztosít a teljes sort, több oszlopcsaláddal átfedés.

Megfelelő Azure-szolgáltatások: [HBase a hdinsight eszközben][hbase]

## <a name="data-analytics"></a>Adatelemzés

Adattároló analytics nagymértékben párhuzamos megoldásokat választásával dolgozhat fel, tárolására és elemzéséhez. Ezek az adatok van osztva a megosztás-nothing architektúra segítségével maximalizálhatja a méretezhetőség és a függőségek minimalizálása érdekében több kiszolgáló között. Az adatok nem valószínű, hogy statikus, lehet, ezért ezekkel az áruházakkal tudja kezelni a nagy mennyiségű információ többféle formátumúak érkező több adatfolyam, miközben továbbra is új lekérdezések feldolgozásához meg kell adni. 

Azure-szolgáltatásokhoz kapcsolódó:

- [Az SQL Data Warehouse][sql-dw]
- [Azure Data Lake][data-lake]

## <a name="search-engine-databases"></a>Keresési adatbázisok  

A keresési vezérlő adatbázisának támogatja a keresse meg a külső adatokat tárolja és szolgáltatásokban tárolt adatokat. A keresési vezérlő adatbázisának használható index nagy mennyiségű adatot, és közel valós idejű ezen indexek hozzáférést nyújtanak. Keresési adatbázisok vannak gyakran-re, hogy azonos a webes, bár sok nagy méretű rendszerekhez őket biztosításához használja strukturált és alkalmi keresse a saját adatbázis-képességeket.

A keresési motor adatbázis kulcsfontosságú mutatókat tárolására és nagyon gyorsan index információkat, és a search kérelemmel gyors válaszidők biztosítása. Indexek többdimenziós és szabad szöveges keresés támogathatja a nagy mennyiségű adatot a szöveg között. Indexelő lekéréses modell, a keresési motor adatbázis által kiváltott, vagy egy leküldéses modellel, az külső alkalmazáskód által kezdeményezett segítségével hajtható végre. 

A keresés pontos vagy zavaros lehet. Egy intelligens egyeztetésű keresési feltételek egyeznek-dokumentumok talál, és kiszámítja az egyezés. Néhány a keresőmotorok is támogatja, amelyek alapján szinonimák, genre bővítések visszatérési megegyezik a nyelvi analysis (például megfelelő `dogs` a `pets`), és amelynek (egyező szavak megegyező legfelső szinttel). 

Megfelelő Azure-szolgáltatások: [Azure Search][search]

## <a name="time-series-databases"></a>Idő adatsorozat adatbázisok

Adatsorozat időadatok értékek idő szerint vannak rendezve, idő adatsorozat adatbázis pedig egy adatbázist, amely ehhez az adattípushoz van optimalizálva. Idő adatsorozat adatbázisok nagyon nagy mennyiségű írást, támogatnia kell, általában nagy mennyiségű adat valós idejű gyűjtött számos forrásból származó. Frissítések ritkán fordul elő, és tömeges műveletek gyakran végzett törli. Annak ellenére, hogy a rekordok egy idősorozat adatbázisba írt általában kicsi, gyakran vannak a rekordok nagy száma, illetve teljes adatméret rohamosan.

Idő adatsorozat adatbázisok megfelelőek telemetriai adatainak tárolásához. Forgatókönyvek például a IoT érzékelők vagy alkalmazás/rendszer számlálói.

Megfelelő Azure-szolgáltatások: [idő adatsorozat Insights][time-series]

## <a name="object-storage"></a>Objektumtár  

Objektum-tárolási tárolja és beolvassa a nagyméretű bináris objektumok (képek, fájlok, video- és folyókat, nagy alkalmazásobjektumok adatok és dokumentumok, a virtuálisgép-lemezképeket) van optimalizálva. Ezen típusok álló a tárolt adatok, bizonyos metaadatait és egyedi Azonosítót az objektum eléréséhez tároló objektumok. Objektum tárolja a rendkívül nagy mennyiségű strukturálatlan adatok kezelését teszi lehetővé.  

Megfelelő Azure-szolgáltatások: [Blob-tároló][blob]

## <a name="shared-files"></a>Megosztott fájlok   

Egyes esetekben egyszerű egybesimított fájlok használata a leghatékonyabb eszköze lehet tárolja, és az információk beolvasása. Fájlmegosztások használata lehetővé teszi, hogy a fájlok hálózaton keresztül érhető el. Adott a megfelelő biztonsági és egyidejű hozzáférés-vezérlési mechanizmusok, ezzel a módszerrel adatok megosztása engedélyezheti a kiválóan méretezhető hozzáférést alapvető, alacsony szintű műveleteket, például az egyszerű olvasási és írási kérelmek elosztott szolgáltatásokat.

Megfelelő Azure-szolgáltatások: [File Storage][file-storage]

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