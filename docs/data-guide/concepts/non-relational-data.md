---
title: "A nem relációs adatokat, és a nosql-alapú"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 8dd8f2b9dfef680f99c9c6b32aacf019c13095b0
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/14/2018
---
# <a name="non-relational-data-and-nosql"></a>A nem relációs adatokat, és a nosql-alapú

A *nem relációs adatbázis* egy adatbázis, amely nem használja a sorok és oszlopok található az adatbázis-rendszerek legtöbb hagyományos táblázatos séma. Nem relációs adatbázisok inkább a tárolási modellben, optimalizált a tárolt adatok meghatározott követelményeinek. Például tárolhatjuk az adatok egyszerű kulcs/érték párok, JSON-dokumentumokként, akár szélek és csúcsban grafikon. 

Milyen minden ezen adatokhoz tárolók rendelkezik közös, hogy azok ne használjon egy [relációs modell](./relational-data.md). Ezenkívül ezek általában pontosabb támogatják-e adatokat, és hogyan lehet adatokat lekérdezni a következő típusban. Például, amikor adatsor tárolók vannak optimalizálva, lekérdezések keresztül adatokat, időalapú sorozatát graph adattárolókhoz vannak optimalizálva, entitások közötti kapcsolatok súlyozott felfedezése közben. Egyik formátum volna generalize és a tranzakciós adatok kezelése a feladatát. 

A kifejezés *NoSQL* hivatkozik, amely nem használ SQL lekérdezések, és más programozási nyelveket és szerkezetek helyette használja az adatok lekérdezéséhez adattárolókhoz. A gyakorlatban "NoSQL" azt jelenti, hogy "nem relációs adatbázis," annak ellenére, hogy ezek az adatbázisok számos támogatja az SQL-kompatibilis lekérdezéseket. Az alapul szolgáló lekérdezés végrehajtási stratégia azonban általában nagyon eltérhet a hagyományos RDBMS volna az azonos SQL-lekérdezés végrehajtása.

A következő szakaszok ismertetik a nem relációs nagy csoportra vagy a NoSQL-adatbázis.

## <a name="document-data-stores"></a>A dokumentum adattároló
A dokumentum tárolóban kezeli olyan elnevezett karakterláncmezőket és objektum adatértékek néven entitás egy *dokumentum*. Ezekkel az áruházakkal adatok általában adat tárolása JSON-dokumentumok formájában. Egy skaláris elem, például egy szám vagy egy összetett elem, például a lista vagy egy szülő-gyermek gyűjtemény minden mező értéke lehet. Az egy dokumentum mezők kódolható az sokféleképpen, XML, YAM, JSON, BSON együtt, vagy akár egyszerű szövegként tárolt. A dokumentumok mezők érhetők el a felügyelet tárolórendszer, lekérdezés és a szűrő adatait az alkalmazás engedélyezése a mezők az értékek használatával.  

Általában a dokumentum egy entitás a teljes adatokat tartalmaz. Milyen elemek képeznek egy entitás az adott alkalmazás. Például egy entitás tartalmazhatnak mindkettőt, vagy az ügyfél, a megrendelést részleteit. Egyetlen dokumentum információk is kerülhetnek, lenne egy relációs adatbázis-kezelő rendszerének (RDBMS) a relációs több táblázatot kell elosztva. A dokumentum store nem igényli, hogy az összes dokumentum hasonló struktúrával rendelkezik. Ez a megközelítés szabadkézi nagyfokú rugalmasságot biztosít. Például alkalmazások különböző adatot tárolhat dokumentumok üzleti követelmények megváltozása adott válaszként.  

![Példa a dokumentum adattár](./images/document.png)  

Az alkalmazás dokumentumok lekérheti a dokumentum kulcs használatával. Ez a dokumentum, amely a gyakran kivonatolt, adatok egyenletes elosztása érdekében egyedi azonosítója. Néhány dokumentum-adatbázis automatikusan hozza létre a dokumentum kulcsot. Mások számára lehetővé teszik a dokumentum kívánja használni, mint a key attribútum adja meg. Az alkalmazás egy vagy több mező értéke alapján dokumentumokat is lekérdezheti. Néhány dokumentum-adatbázisokat támogatja, lehetővé teszi egy vagy több indexelt mezők alapján dokumentumok gyors keresési indexelő.  

Sok dokumentum-adatbázis helyi frissítések, egy alkalmazás egy dokumentumot a megadott mező értékének módosítása nélkül írja át a teljes dokumentum engedélyezése támogatja. Olvasási és írási műveletek keresztül egy dokumentumban több mező esetében általában atomi.

Megfelelő Azure szolgáltatás:  

- [Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/)

## <a name="columnar-data-stores"></a>Oszlopos adattároló
Egy oszlopos vagy oszlop-család adattároló oszlopok és sorok alapul. Legegyszerűbb formájukban oszlop-család adattároló jelenhet meg nagyon hasonlít egy relációs adatbázisban, legalább hasonlóak. A valós oszlop-család adatbázis rugalmasságuknak köszönhetően a denormalizált megközelítést szerkezetének kialakítása ritka adatokat, amelyek az oszlop megközelítéssel ered, az adatok tárolására.  

Az eltolásokat tekintheti oszlop-család adattároló mint sorok és oszlopok táblázatos adatokat, de az oszlopok oszlopcsaláddal néven csoportra oszlanak. Minden oszlop termékcsalád oszlopkészleteket, amelyek logikailag kapcsolatos és általában beolvasni vagy egységként választéka tartalmazza. Külön-külön egyéb adatokhoz külön oszlopcsaláddal is tárolható. Egy oszlop családba tartozó új oszlopok is hozzáadhatók dinamikusan, és lehet, hogy a sorok ritka (Ez azt jelenti, hogy a sor nem kell minden oszlop értéke). 

Az alábbi ábrán látható egy példa két oszlopcsaláddal `Identity` és `Contact Info`. Az egyetlen entitás az adatok minden oszlop termékcsalád sor ugyanazzal a kulccsal rendelkezik. Ez a struktúra, ahol dinamikusan változhat a sorok összes adott objektum egy oszlop családba tartozó, egy fontos előnye, hogy az oszlop-család módszert használja, így az űrlap magas alkalmas való adattárolás eltérő sémával adattár.

![Az oszlop-család adatok – példa](../../guide/technology-choices/images/column-family.png)

Egy kulcs-érték tárolóban és a dokumentum-adatbázis legtöbb oszlop-család adatbázis fizikailag tárolja adatait kulcs sorrendben nem úgy kivonatát. A sorkulcs az elsődleges index minősülnek, és lehetővé teszi, hogy a kulcs-alapú hozzáférés egy adott kulcs vagy kulcsok számos keresztül. Egyes megvalósítások használatával hozhat létre másodlagos indexek egy oszlop termékcsalád adott oszlopok keresztül. Másodlagos indexek lehetővé teszik, hogy az oszlopok érték, hanem a sorkulcsa adatainak beolvasása.

A lemezen az oszlopok egy oszlop családba tartozó összes tárolják együtt ugyanabban a fájlban, az egyes fájlok sorok bizonyos számú. Nagy méretű adatkészletekhez Ez a megközelítés hoz létre egy teljesítménynövekedést csökkenti az adatmennyiséget rendszernek végig kell olvasnia a lemezről, ha csak néhány oszlopot a rendszer megkérdezi a együtt egyszerre. 

Egy sor olvasási és írási műveletek esetében általában atomi belül csak egy oszlop címcsaládot, bár egyes megvalósítások atomicity biztosít a teljes sort, több oszlopcsaláddal átfedés.

Megfelelő Azure szolgáltatás:  

- [HBase a hdinsight eszközben](/azure/hdinsight/hdinsight-hbase-overview)

## <a name="keyvalue-data-stores"></a>Kulcs/érték adattároló
A kulcs/érték tároló lényegében egy nagy kivonattábla. Minden egyes adatérték társítandó egyedi kulcs, és a kulcs-érték tároló ezt a kulcsot használja az adatok tárolásához egy megfelelő kivonatoló függvénnyel. A kivonatoló funkció így biztosít az még akkor is, terjesztési kivonatolt kulcsok adatok tárolására van kiválasztva.

Legtöbb kulcs/érték csak támogatási egyszerű lekérdezés tárolja, beszúrási és törlési műveletek. Egy érték módosítása (részben vagy teljesen), az alkalmazás felül kell írnia a meglévő adatok a teljes érték. A legtöbb implementációkban olvasása vagy írása egyetlen értéket egy atomi művelet. Ha az érték túl nagy, írás hosszabb időt vehet igénybe.

Egy alkalmazás tetszőleges adatait tárolhatja értékek csoportjaként, bár egyes kulcs-érték tárolók ugyanazok a korlátozások értékek maximális mérete. A tárolt értékei nem átlátszó, a tárolási rendszer szoftverre. A sémaadatok megadott legyen, és az alkalmazás értelmezi. Alapvetően értékei a blobok és a kulcs-érték tároló egyszerűen kéri le vagy a kulcs értékét tárolja.

![A kulcs/érték adatokat – példa](../../guide/technology-choices/images/key-value.png)

Kulcs-érték tárolók magas a kulcs, vagy számos különböző kulcsok értékének használatával egyszerű keresések alkalmazások vannak optimalizálva, de kevésbé alkalmasak a kulcs/érték, például az adatok csatlakoztatása több különböző táblák között lekérdezni adatokat igénylő rendszerek táblák. 

Kulcs-érték tárolók is nem optimalizált forgatókönyvek, ahol kérdez le, vagy nem kulcs-érték szerinti szűrés fontos, hanem végrehajtott keresési műveletek végrehajtása csak kulcsok alapján. Például egy relációs adatbázisban, is található a rekord nem kulcs oszlopok szűréséhez WHERE záradék használatával, de kulcs/érték tárolók általában nem rendelkezik ilyen típusú értékeket az keresési funkciójának könnyebb működése, vagy ha így tesznek értékek lassú vizsgálatot igényel.

Egy-egy kulcs-érték tároló rendkívül méretezhető, lehet, mert az adattár is könnyen szét adatok több csomópontok külön számítógépeken.

Azure-szolgáltatásokhoz kapcsolódó:  
- [Az Azure Cosmos DB tábla API](/azure/cosmos-db/table-introduction)  
- [Azure Redis Cache](https://azure.microsoft.com/services/cache/)  
- [Azure Table Storage](https://azure.microsoft.com/services/storage/tables/)

## <a name="graph-data-stores"></a>Graph adattároló
Egy grafikonon adattár kétféle típusú adatokat, a csomópontok és a szélek kezeli. Csomópontok jelölik a entitásokat, és szélén, adjon meg e entitások közötti kapcsolatok. Csomópontok és a szélek rendelkezhet, amelyek információval szolgálnak, hogy a csomópont vagy a táblázatban levő oszlopkészleteket hasonló biztonsági tulajdonságai. Szegélyek is lehet a kapcsolat jellege irányt.  

Egy grafikonon adattár az a célja, hogy lehetővé tegye egy alkalmazást, a hálózati csomópont és szélek áthaladó lekérdezések hatékonyan végrehajtásához, és elemezheti az entitások közötti kapcsolatok. Az alábbi ábrán látható, hogy a szervezetek személyzet adatok strukturált, egy grafikonon. Az entitások az alkalmazottak és szervezeti egységek, és széleinek jelzik, hogy jelentéskészítési kapcsolatokat és a részleg, mely az alkalmazottak munka. Az ehhez a diagramhoz a a szélén nyilak a kapcsolat irányát.

![Az adatok egy grafikonon adattár – példa](../../guide/technology-choices/images/graph.png)

Ez egyszerű végrehajtásához lekéri például struktúra teszi "található összes beosztottak közvetlenül vagy közvetve Sarah" vagy "Ki működik John részleghez?" Az entitások és kapcsolatok sok nagy diagramok nagyon gyorsan végezhet nagyon összetett elemzések. Sok diagram adatbázis lekérdezésnyelvet, amely hatékonyan haladnak át a hálózati kapcsolatok segítségével adja meg.  

Megfelelő Azure szolgáltatás:  
- [Az Azure Cosmos DB Graph API](/azure/cosmos-db/graph-introduction)  

## <a name="time-series-data-stores"></a>Adatsorozat időadatok tárolja
Adatsorozat időadatok olyan értékek idő és egy ehhez az adattípushoz adatsorozat adattár optimalizált ideje szerint vannak rendezve. Idő adatsorozat adattárolókhoz támogatnia kell a rendkívül nagy mennyiségű írást,, azokat általában gyűjteni nagy mennyiségű adat valós időben számos forrásból. Idő adatsorozat adattárolókhoz telemetriai adatok tárolására vannak optimalizálva. Forgatókönyvek például a IoT érzékelők vagy alkalmazás/rendszer számlálói. Frissítések ritkán fordul elő, és tömeges műveletek gyakran végzett törli.

![Adatsorozat időadatok – példa](./images/time-series.png)

Annak ellenére, hogy a rekordok egy idő adatsorozat adatbázisba írt általában kicsi, gyakran vannak a rekordok nagy száma, illetve teljes adatméret rohamosan. Adatsorozat időadatok is leíró soron és a késői érkező adatokat tárolja, automatikus adatpontokat és az optimalizálás szempontjából idő windows leírt lekérdezések indexelése. A legutóbbi funkció lehetővé teszi, hogy a lekérdezések futtatásához adatpontok több millió és több adatfolyamokat gyorsan, támogatásához idő adatsorozat képi megjelenítéseket, amelyek egy közös mód adatsorozat felhasznált idő. 

További információkért lásd: [Time series megoldások](../scenarios/time-series.md)

Megfelelő Azure szolgáltatás:  
- [Az Azure idő adatsorozat Insights](https://azure.microsoft.com/services/time-series-insights/)  
- [A hbase a HDInsight eszközzel OpenTSDB](/azure/hdinsight/hdinsight-hbase-overview)

## <a name="object-data-stores"></a>Objektum adattároló
Objektum adattárolókhoz vannak optimalizálva, tárolására és nagyméretű bináris objektumok vagy bináris objektumok, például képek szövegfájlok, video- és adatfolyamok, nagy alkalmazásobjektumok adatok és dokumentumok vagy virtuálisgép-lemezképek beolvasása. Egy objektum áll a tárolt adatok, bizonyos metaadatait és egyedi Azonosítót az objektum eléréséhez. Objektum tárolókat támogató külön-külön nagyon nagy fájlok, valamint adja meg a teljes tárterület kezelése az összes fájl nagy mennyiségű tervezték.  

![Objektum adatai – példa](./images/object.png)

Néhány objektum adattárolókhoz egy adott blob több kiszolgáló-csomópont között, amely lehetővé teszi, hogy a gyors párhuzamos olvasások replikálni. Ez a következő funkciókat engedélyezi a kibővített lekérdezése a nagy fájlokban található adatok, mert több, különböző kiszolgálókon jellemzően futó folyamatok is az összes lekérdezés a nagyméretű adatfájl egyidejűleg.

Az objektum adattárolókhoz egy különleges esetben a hálózati fájlmegosztáson. Fájlmegosztások használata lehetővé teszi, hogy a fájlok szabványos hálózati protokollokon, például a kiszolgálói üzenetblokk (SMB) használatával a hálózaton keresztül érhető el. Adott a megfelelő biztonsági és egyidejű hozzáférés-vezérlési mechanizmusok, ezzel a módszerrel adatok megosztása engedélyezheti a hozzáférést kiválóan méretezhető alapvető, alacsony szintű műveletek, például a egyszerű olvasási és írási kérelmek elosztott szolgáltatásokat.

Megfelelő Azure szolgáltatás:   

- [Azure Blob Storage](https://azure.microsoft.com/services/storage/blobs/)  
- [Azure Data Lake Store](https://azure.microsoft.com/services/data-lake-store/)  
- [Azure File Storage](https://azure.microsoft.com/services/storage/files/)  


## <a name="external-index-data-stores"></a>Külső index adattároló

Külső index adattárolókhoz adatokról és egyéb szolgáltatások tárolt információk keresése adja meg. Külső index egy másodlagos index bármely adattároló funkcionál, és segítségével index nagy mennyiségű adatot, és közel valós idejű ezen indexek hozzáférést nyújtanak. 

Például lehetséges, hogy a fájlrendszer tárolódnak. Fájl keresése a fájl elérési útja gyors, de a fájl tartalma alapján keres igényelnének keresését az összes fájl, amely lassú. Külső index lehetővé teszi a másodlagos keresési indexek létrehozása, és gyorsan megtalálhatja a fájlok, a feltételeknek megfelelő elérési útját. Külső index alkalmazásának van a kulcs/érték egy másik példa a gombot, hogy csak index tárolja. Egy másodlagos index, az adatok levő értékek alapján létrehozhatja, és gyorsan kereshet a kulcsot, amely egyedileg azonosítja az egyes egyező elemek. 

![Keresési adatainak – példa](./images/search.png)

Az indexek egy indexelési folyamat futtatásával jönnek létre. Ez elvégezhető, az adattár által kiváltott, vagy egy leküldéses modellel, az alkalmazás kódjában által kezdeményezett egy lekéréses modell használatával. Indexek többdimenziós és szabad szöveges keresés támogathatja a nagy mennyiségű adatot a szöveg között. 

Külső index adattárolókhoz gyakran használják a teljes szöveg és a web alapú keresési támogatásához. Ezekben az esetekben a keresés is pontosan vagy zavaros lehet. Egy intelligens egyeztetésű keresési feltételek egyeznek-dokumentumok talál, és kiszámítja az egyezés. Egyes külső indexek nyelvi elemzés visszatérési megegyezik a szinonimák, genre bővítések (például egyező "kutya" a "Hobbiállatok") alapján, és származó (például keresés, a "Futtatás" is megfelel "Itt futott" és "fut") által is támogatja. 

Megfelelő Azure szolgáltatás:  

- [Azure Search](https://azure.microsoft.com/services/search/)


## <a name="typical-requirements"></a>Általános követelmények
A nem relációs adattároló, amely a relációs adatbázisok által használt egy másik tárolóhelyre architektúrát gyakran használnak. Pontosabban azokat általában nem rögzített sémájába rendelkező felé. Is általában nem támogatja a tranzakciókat, vagy más tranzakciók hatókörét, és általában nem tartoznak méretezhetőség másodlagos kulcsot.

A következő összehasonlítja a követelmények vonatkoznak a nem relációs adatokat tároló:

| Követelmény | A dokumentum adatok | Oszlop-család adatok | Kulcs/érték | Diagramadatok | 
| --- | --- | --- | --- | --- | 
| Normalizálási | Denormalizált | Denormalizált | Denormalizált | Normalizált | 
| Séma | Olvassa el a séma | Írási, olvasható oszlop sémáját definiált oszlopcsaláddal | Olvassa el a séma | Olvassa el a séma | 
| Egységességének (egyidejű tranzakciókat) | Aprólékosan beállítható konzisztencia, a dokumentum szintű biztosítja, hogy | Oszlop-család&ndash;garanciák szinten | Kulcs szintű garanciák | Graph-szintű garanciák 
| Atomicity (tranzakció hatókörében) | Gyűjtemény | Tábla | Tábla | Graph | 
| Zárolási stratégia | Az optimista (szabad zárolás) | Pesszimista (sor zárolás) | Az optimista (ETag) | 
| Minták | Közvetlen elérésű | A magas/wide adatokkal összesítések | Közvetlen elérésű | Közvetlen elérésű |
| Indexelés | Elsődleges és másodlagos indexek | Elsődleges és másodlagos indexek | Csak az elsődleges index | Elsődleges és másodlagos indexek | 
| Adatok alakzat | Dokumentum | Az oszlopok tartalmazó oszlopcsaláddal táblázatos | Kulcs-érték | Gráf élei számára és a csúcsban tartalmazó | 
| Ritka | Igen | Igen | Igen | Nem | 
| Wide (vagy attribútum rengeteg) | Igen | Igen | Nem | Nem |  
| Datum mérete | Közepes (kis MB) (KB) kicsi | Közepes (MB) nagy (kis GB) | Kis (KB) | Kis (KB) | 
| A teljes maximális skálája | Nagyon nagy méretű (PBs) | Nagyon nagy méretű (PBs) | Nagyon nagy méretű (PBs) | Nagy (több TB-nyi) | 

| Követelmény | Adatsorozat időadatok | Objektum adatai | Külső index adatok |
| --- | --- | --- | --- |
| Normalizálási | Normalizált | Denormalizált | Denormalizált |
| Séma | Olvassa el a séma | Olvassa el a séma | Írás séma | 
| Egységességének (egyidejű tranzakciókat) | – | N/A | – | 
| Atomicity (tranzakció hatókörében) | – | Objektum | – |
| Zárolási stratégia | – | Pesszimista (blob zárolás) | – |
| Minták | Véletlenszerű hozzáférést és összesítési beállítások | Soros hozzáférés | Közvetlen elérésű | 
| Indexelés | Elsődleges és másodlagos indexek | Csak az elsődleges index | – |
| Adatok alakzat | A táblázatos | A BLOB és metaadatok | Dokumentum |
| Ritka | Nem | – | Nem | 
| Wide (vagy attribútum rengeteg) |  Nem | Igen | Igen |  
| Datum mérete | Kis (KB) | Nagy (GB) a nagyon nagy (több TB-nyi) | Kis (KB) |
| A teljes maximális skálája | Nagy (kis több TB-nyi)  | Nagyon nagy méretű (PBs) | Nagy (kis több TB-nyi) | 

