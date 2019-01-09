---
title: Nem relációs adat- és NoSQL
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.openlocfilehash: e712cf7def9e237257b911c40346027c1529201d
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/08/2019
ms.locfileid: "54114283"
---
# <a name="non-relational-data-and-nosql"></a>Nem relációs adat- és NoSQL

A *nem relációs adatbázis* egy adatbázis, amely nem használja a sorok és oszlopok található a legtöbb hagyományos adatbázisrendszerek táblázatos sémája. Nem relációs adatbázisok inkább egy tárolási modell, amely optimalizált, tárolt adatok típusú az adott igények szerint. Például tárolhatók az adatok egyszerű kulcs/érték párok, JSON-dokumentumok formájában vagy élek és csúcspontok gráfként.

Milyen az összes ilyen tárolók rendelkezik a common data, hogy azok ne használjon egy [relációs modell](../relational-data/index.md). Emellett azok általában pontosabban támogatják-e adatokat, és hogyan kérhetők le adatok típusát. Például, amikor idősorozat-adatok áruházak vannak optimalizálva lekérdezések keresztül feladatütemezések időalapú adatok, amíg graph adattárak entitások közötti kapcsolatok súlyozott felfedezése vannak optimalizálva. Sem formátum lenne generalize jól, a feladat a tranzakciós adatok kezeléséhez.

Az előfizetési időszak *nosql-alapú* hivatkozik, amely nem az SQL lekérdezések használatával, ezért használja inkább más programozási nyelveket és szerkezetek az adatok lekérdezéséhez adattárakban. A gyakorlatban "NoSQL" azt jelenti, hogy "nem relációs adatbázis," annak ellenére, hogy számos, az adatbázisok támogatják az SQL-kompatibilis lekérdezéseket. Azonban az alapul szolgáló lekérdezés végrehajtási stratégia általában jelentősen különbözik a hagyományos RDBMS volna az azonos SQL-lekérdezés végrehajtása módja.

A következő szakaszok ismertetik a fő kategóriára oszthatók nem relációs vagy NoSQL-adatbázis.

## <a name="document-data-stores"></a>A dokumentum-adattárak

Egy dokumentum adattár kezeli az elnevezett karakterlánc-mezők és az adatértékek objektum egy entitásban néven egy *dokumentum*. Ezek adattárak általában adatok tárolása JSON-dokumentumok formájában. Minden mező értéke lehet egy skaláris elem, például egy szám vagy egy összetett elem, például lista vagy a szülő-gyermek gyűjtemény. Egy dokumentum mezőiben szereplő adatok többféle módon, beleértve az XML, YAML, JSON, BSON, de tárolásuk akár egyszerű szövegként is kódolhatók. A storage-kezelési rendszerhez, ezekben a mezőkben szereplő értékek segítségével lekérdezhetik és szűrhetik az adatokat az alkalmazás engedélyezése a dokumentumokon belül mezők érhetők el.

Általában egy dokumentum egy entitás összes adatát tartalmazza. Az entitást alkotó elemek az alkalmazástól függenek. Például egy entitás tartalmazhatja egy ügyfél vagy egy rendelés adatait, vagy akár a kettő kombinációját is. Egyetlen dokumentum, amely egy relációsadatbázis-kezelő rendszerének (RDBMS) a több relációs tábla között lenne elosztva adatokat tartalmazhatnak. A dokumentumtároló nem igényli, hogy minden dokumentum ugyanazzal a struktúrával rendelkezzen. Ez a szabad formátumú megközelítés nagy fokú rugalmasságot biztosít. Például alkalmazások alapján különböző adatokat tárolhatnak a dokumentumokban az üzleti követelmények változásakor a.

![A példában a dokumentum-adattár](./images/document.png)  

Az alkalmazás a dokumentumkulcs segítségével kérhet le dokumentumokat. Ez a dokumentum egyedi azonosítója, amely az adatok egyenletes elosztása érdekében gyakran kivonatolt. Néhány dokumentum-adatbázis automatikusan létrehozza a dokumentumkulcsot. Mások lehetővé teszik, hogy a dokumentum egyik attribútumát adja meg kulcsként. Az alkalmazás emellett dokumentumokat is lekérdezhet egy vagy több mező alapján. Néhány dokumentum-adatbázis támogatja az indexelést a dokumentumok egy vagy több indexelt mező alapján történő gyors keresése érdekében.

Számos dokumentum-adatbázis támogatja a helyi frissítést, amely révén az alkalmazások a teljes dokumentum átírása nélkül módosíthatják az adott mezők értékeit egy dokumentumban. Az egy dokumentum több mezőjén végrehajtott olvasási és írási műveletek általában atomi jellegűek.

Kapcsolódó Azure-szolgáltatás:  

- [Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/)

## <a name="columnar-data-stores"></a>Oszlopalapú adattárak

Oszlopalapú vagy oszlopcsalád-adattár rendezik az adatokat sorok és oszlopok. Legegyszerűbb formájában az oszlopcsalád-adattár is megjelenhetnek nagyon hasonlít egy relációs adatbázisban, legalábbis a koncepciót tekintve. A valódi oszlopcsalád adatbázisok rugalmasságuknak köszönhetően az Oszlopalapú megközelítés ered, adattárolás, amely ritka adatok strukturálásának denormalizált megközelítésében rejlik.

Is felfoghatók tabulált, sorokat és oszlopokat betöltő, oszlopcsalád-adattár, de az oszlopok oszlopcsaláddal néven csoportokra vannak felosztva. Minden oszlopcsalád oszlopkészleteket, amelyek logikailag kapcsolódó és a rendszer általában beolvasott vagy rendelkezik egyetlen egységként kezelhetők. Az egyéb, külön elérhető adatok külön oszlopcsaládokban tárolhatók. Egy oszlopcsaládon belül az új oszlopok dinamikusan hozzáadhatók, a sorok pedig ritkák is lehetnek (ez azt jelenti, hogy a soroknak nem kell minden oszlophoz értéket tartalmazniuk).

Az alábbi ábrán látható példában két oszlopcsalád szerepel: `Identity` és `Contact Info`. Az adatokat egyetlen entitás tartalmaz minden oszlopcsalád ugyanazt a sorkulcsot. Ez a struktúra, ahol a sorok bármely egy oszlopcsalád adott objektumához dinamikusan változhat, egy fontos előnye, hogy az oszlopcsalád-módszert használja, így ezt a kérdőívet, magas az olyan adatok eltérő sémákkal tárolására szolgáló adattár.

![Példa az oszlopcsalád-adatok](../../guide/technology-choices/images/column-family.png)

Egy kulcs/érték tároló és a egy dokumentum-adatbázis eltérően a legtöbb oszlopcsalád-adatbázisok fizikailag adatok tárolása kulcs szerinti sorrendben, nem pedig kivonatok szerint. A sorkulcs számít az elsődleges index, és engedélyezi a kulcsalapú hozzáférést egy adott kulcs vagy kulcsok számos. Egyes megvalósítások lehetővé teszi, hogy hozzon létre másodlagos indexeket egy oszlopcsalád adott oszlopain. Másodlagos indexek lehetővé kérheti le az adatokat a sorkulcs helyett oszlop értéke.

A lemezen az oszlopok egy oszlopcsalád belül az összes tárolódnak együtt ugyanabban a fájlban, az adott számú sor mindkét fájlban. A nagy adatkészletek Ez a megközelítés, amely lemezről beolvasott, ha csak néhány oszlopot a rendszer megkérdezi a együtt egyszerre kell adatmennyiség csökkentésével teljesítménybeli előnyt hoz létre.

Sor olvasási és írási műveletek általában atomi belül egyetlen oszlopcsalád jellegűek, bár egyes megvalósítások az egész sort, több oszlopcsaláddal átfedés biztosítják az atomitást.

Kapcsolódó Azure-szolgáltatás:  

- [A HDInsight HBase](/azure/hdinsight/hdinsight-hbase-overview)

## <a name="keyvalue-data-stores"></a>Kulcs/érték-adattárak

A kulcs/érték tároló lényegében egy nagy kivonattábla. Minden egyes adatértékhez egy egyedi kulcs társul, a kulcs/érték tároló pedig ezt a kulcsot használja az adatok egy megfelelő kivonatoló függvénnyel való tárolásához. A kivonatoló algoritmus a kivonatolt kulcsok az adattárban való egyenletes elosztására van kiválasztva.

A legtöbb kulcs/érték tároló csak az egyszerű lekérdezési, beszúrási és törlési műveleteket támogatja. Egy érték (akár részleges, akár teljes) módosításához az alkalmazásnak a teljes értékre vonatkozóan felül kell írnia a meglévő adatokat. A legtöbb megvalósításban egyetlen érték olvasása vagy írása atomi műveletnek számít. Ha az érték túl nagy, az írás hosszabb időt is igénybe vehet.

Az alkalmazás tetszőleges adatokat tárolhat egy értékekből álló készletként, bár egyes kulcs/érték tárolók korlátozzák az értékek maximális méretét. A tárolt értékek a tárolórendszer szoftvere számára nem átlátszók. A sémaadatok biztosítása és értelmezése az alkalmazás feladata. Az értékek alapvetően blobok, a kulcs/érték tároló pedig egyszerűen kulcsonként olvassa be vagy tárolja az értéket.

![A példában az adatok egy kulcs/érték tároló](../../guide/technology-choices/images/key-value.png)

Kulcs/érték tárolók magas a kulcs, vagy széles körű kulcsok értékének használatával egyszerű kereséseket végző alkalmazásokhoz vannak optimalizálva, azonban kevésbé megfelelőek a rendszerek esetében, amelyek használatával adatokat lekérdezni, kulcs/érték, például az adatok összekötő több különböző táblákon táblák.

Kulcs/érték tárolók emellett nincsenek optimalizálva forgatókönyvekhez, ahol fontos lekérdezése vagy nem kulcs-érték alapján történő szűrést, nem pedig kizárólag kulcsokon alapuló keresések végrehajtása. Például egy relációs adatbázisban, az nem kulcs oszlop szűrése a WHERE záradék használatával megtalálhat egy rekordot, de kulcs/érték tárolók nem rendelkeznek ilyen típusú értékekre vonatkozó keresési képességekkel, vagy ha az összes érték lassú vizsgálatot igényel.

Egyetlen kulcs/érték tároló lehet rendkívüli mértékben skálázható, mivel az adattár könnyedén feloszthatja az adatokat több, külön gépeken található csomópont között.

Kapcsolódó Azure-szolgáltatások:  

- [Az Azure Cosmos DB tábla API](/azure/cosmos-db/table-introduction)
- [Azure Redis Cache](https://azure.microsoft.com/services/cache/)  
- [Azure Table Storage](https://azure.microsoft.com/services/storage/tables/)

## <a name="graph-data-stores"></a>Graph-adattárak

A graph-tárolóban kétféle típusú információt tárolnak: csomópontokat és éleket kezeli. Csomópontok entitást képviselik, és élek meg ezek az entitások közötti kapcsolatokat. Mind a csomópontok, mind az élek rendelkezhetnek olyan tulajdonságokkal, amelyek a táblák oszlopaihoz hasonlóan információt nyújtanak az adott csomópontról vagy élről. A szegélyeknek is lehet iránya, amely a kapcsolat természetét jelöli.

A graph-tárolóban célja, hogy egy alkalmazás, amely a csomópontok és élek hálózatát bejáró lekérdezések, és elemezheti az entitások közötti kapcsolatokat. Az alábbi ábrán látható gráfként strukturálva a szervezet munkatársai adatait. Az entitások az alkalmazottak és a részlegek, az élek pedig a jelentéskészítési kapcsolatokat jelzik, valamint azt, hogy melyik alkalmazott melyik részlegen dolgozik. Ezen a gráfon az éleken látható nyilak a kapcsolatok irányát jelzik.

![A gráf adattárban lévő adatok – példa](../../guide/technology-choices/images/graph.png)

Ezzel a struktúrával egyértelműen hajthatók végre a következőkhöz hasonló lekérdezések: „Minden alkalmazott megkeresése, aki közvetlenül vagy közvetetten Sárának jelent” vagy „Ki dolgozik Jánossal egy részlegen?” A számos entitást és kapcsolatot tartalmazó, nagy méretű gráfok esetén nagyon gyorsan hajthat végre rendkívül összetett elemzéseket. Sok gráfadatbázis használ olyan lekérdezési nyelvet, amellyel egy kapcsolatokból álló hálózat hatékonyan bejárható.

Kapcsolódó Azure-szolgáltatás:  

- [Az Azure Cosmos DB Graph API](/azure/cosmos-db/graph-introduction)  

## <a name="time-series-data-stores"></a>Idősorozat-adatokat tárolja.

Idősorozat-adatok olyan készlete, értékeket, és az ilyen típusú adatok sorozat adattár optimalizált időpontja szerint vannak rendezve. Idősorozat-adattárak támogatnia kell egy nagyon nagy számú írást, mivel általában nagy mennyiségű adat valós idejű gyűjtött sok forrásból származó. Idősorozat-adattárak telemetriaadatok tárolására vannak optimalizálva. A forgatókönyvek IoT-érzékelőket vagy alkalmazás-/rendszerszámlálókat is tartalmaznak. A frissítések ritkák, a törlések pedig a legtöbbször tömeges műveletként történnek.

![Idősorozat-adatok – példa](./images/time-series.png)

Bár az idősorozat-adatbázisok írt rekordok általában kicsi, gyakran előfordulnak nagy számú rekordjait, és az adatok összmérete is gyors ütemben növekedhet. Idősorozat-adatok is leíró out soron kívüli és késői érkező adatait tárolja, az Automatikus indexelés adatpontok, és a windows idő szempontjából leírt lekérdezések optimalizálását. A legutóbbi funkció lehetővé teszi, hogy a lekérdezések gyorsan futhassanak több milliónyi adatpont és több adatfolyamot, annak érdekében, hogy támogatja a time series vizualizációkat, ez gyakran az idősorozat-adatok felhasznált idő.

További információkért lásd: [idő idősorozattal kapcsolatos megoldások](../scenarios/time-series.md)

Kapcsolódó Azure-szolgáltatások:

- [Azure Time Series Insights](https://azure.microsoft.com/services/time-series-insights/)  
- [A HDInsight-alapú HBase-OpenTSDB](/azure/hdinsight/hdinsight-hbase-overview)

## <a name="object-data-stores"></a>Objektum-adattárak

Objektum adattárak tárolására és lekérdezésére a nagy méretű bináris objektumok vagy bináris objektumok, például képek, szövegfájlok, videó- és audio-adatfolyamokat, nagy méretű alkalmazás-adatobjektumok és dokumentumok és virtuális gépek lemezképei vannak optimalizálva. Egy objektum áll a tárolt adatok, a metaadatok és a egy egyedi azonosítója az objektum eléréséhez szükséges. Az objektum támogatja az egyénileg nagyon nagy méretű fájlok, valamint adja meg a teljes tárterület kezelése az összes fájl nagy mennyiségű lett tervezve.

![Objektumadatok – példa](./images/object.png)

Néhány objektum adattár replikálhatja több kiszolgáló-csomópontra, amely lehetővé teszi a párhuzamos gyors olvasást egy adott blob. Ez lehetővé teszi a horizontális felskálázás lekérdezése a nagy fájlokban található adatokat, mert több, különböző kiszolgálókon jellemzően futó folyamatok egyes lekérdezheti a nagy mennyiségű adat fájl egy időben.

Látható objektum egy különleges eset a hálózati fájlmegosztáson. Fájlmegosztások használata lehetővé teszi a fájlok szabványos hálózati protokollokon, például a kiszolgálói üzenetblokk (SMB) használatával a hálózaton keresztül érhető el. Adja meg a megfelelő biztonsági és egyidejű hozzáférési, így az adatok megosztásában biztosíthatnak a rugalmasan méretezhető adathozzáférés alapvető, alacsony szintű műveletek, például egyszerű olvasási és írási kérelmeket elosztott szolgáltatásokat.

Kapcsolódó Azure-szolgáltatások:

- [Azure Blob Storage](https://azure.microsoft.com/services/storage/blobs/)
- [Azure Data Lake Store](https://azure.microsoft.com/services/data-lake-store/)  
- [Azure File Storage](https://azure.microsoft.com/services/storage/files/)

## <a name="external-index-data-stores"></a>Külső index adattárak

Külső index adattárak lehetővé teszi, hogy a más adattárakban és szolgáltatásokban tárolt adatok. Külső index egy másodlagos indexet bármely adattároló funkcionál, és használható nagy mennyiségű adatok indexelése és a közel valós idejű hozzáférést az indexekhez.

Előfordulhat például, hogy a fájlrendszer szövegfájlok. Fájl keresése szerint a fájl elérési útja a gyors, de a keresés alapján a fájl tartalmát kellene keresését, az összes fájl, amely lassú. Külső index teszi lehetővé másodlagos keresési indexet hoz létre, és gyorsan megtalálhatja a feltételeknek a fájlok elérési útját. Külső index alkalmazását, ha kulcs/érték van egy másik példa, hogy csak index által a kulcsot tárolja. Az adatok értékei alapján másodlagos indexet készíthet, és gyorsan kikeresni a kulcsot, amely egyedileg azonosítja az egyes egyező elemeket.

![Keresési adatok – példa](./images/search.png)

Az indexek fut egy indexelési folyamat hoznak létre. Ez elvégezhető, az adattár által aktivált, vagy egy leküldéses modell használatával, alkalmazáskód által kezdeményezett egy lekérési modell használatával. Az indexek többdimenziós és nagy kötegekből álló szöveges adatok támogathatják a szabad szöveges keresést.

Külső index adattárak gyakran használják a teljes szöveges és a webes keresés. Ezekben az esetekben a keresés lehet pontos vagy intelligens lehet. Az intelligens keresés olyan dokumentumokat keres, amelyek megfelelnek egy adott feltételkészletnek, és az egyezés mértékét is kiszámítja. Néhány külső indexet is támogatják a nyelvi elemzés, amelyeket a visszaadandó találatok szinonimák, kategóriakiterjesztések (például egyező "kutyák", "kisállatok") alapján, és, amely szerint (például keresés, "Futtatás" is egyezések "futott" és "fut").

Kapcsolódó Azure-szolgáltatás:  

- [Azure Search](https://azure.microsoft.com/services/search/)

## <a name="typical-requirements"></a>Jellemző követelményei

Nem relációs adattárak gyakran használnak a különböző tárolási architektúrát, amely relációs adatbázisok által használt. Pontosabban, általában nem rögzített sémát kellene felé. Ezenkívül általában nem támogatja a tranzakciókat, vagy pedig hatókörét, a tranzakciók, és általában nem tartoznak másodlagos indexeket, a méretezhetőség.

A következő követelmények mindegyike a nem relációs adattárakat hasonlítja össze:

| Követelmény | Dokumentum-adatokat | Oszlopcsalád-adatok | Kulcs/érték-adatok | A grafikon adatainak |
| --- | --- | --- | --- | --- |
| Normalizálási | Denormalizált | Denormalizált | Denormalizált | Normalizált |
| Séma | Olvassa el a séma | Írás, Olvasás oszlop sémáját definiált oszlopcsaláddal | Olvassa el a séma | Olvassa el a séma |
| (Között több tranzakció) | Hangolható konzisztencia, a dokumentum-szintű garanciák | Oszlopcsalád-&ndash;szintű garanciák | Kulcs-szintű garanciák | Graph-szintű garanciák |
| Atomitást (tranzakció-hatókörben) | Gyűjtemény | Tábla | Tábla | Graph |
| Zárolási stratégia | Az optimista (zárolás ingyenes) | A pesszimista (sor zárolás) | Az optimista (ETag) |
| Elérni | Közvetlen elérésű | A magas/wide adatok összesítések | Közvetlen elérésű | Közvetlen elérésű |
| Indexelés | Az elsődleges és másodlagos indexek | Az elsődleges és másodlagos indexek | Csak az elsődleges index | Az elsődleges és másodlagos indexek |
| Adatalakzat | Dokumentum | Táblázatos oszlopcsaláddal tartalmazó oszlop | Kulcs-érték | Graph-tartalmazó élek és csúcspontok |
| Ritka | Igen | Igen | Igen | Nem |
| Wide (nagy mennyiségű vagy attribútum) | Igen | Igen | Nem | Nem |  
| Datum mérete | (KB) kis-és közepes méretű (alacsony MB) | Közepes (MB) és nagy méretű (alacsony GB) | Kicsi (KB) | Kicsi (KB) |
| A teljes maximális skála | Nagyon nagy (PBs) | Nagyon nagy (PBs) | Nagyon nagy (PBs) | Nagy (TB-osra bővül) |

| Követelmény | Idősorozat-adatok | Objektum adatai | Adatok külső indexelése |
| --- | --- | --- | --- |
| Normalizálási | Normalizált | Denormalizált | Denormalizált |
| Séma | Olvassa el a séma | Olvassa el a séma | Séma |
| (Között több tranzakció) | – | N/A | – |
| Atomitást (tranzakció-hatókörben) | – | Objektum | – |
| Zárolási stratégia | – | A pesszimista (blob zárolás) | – |
| Elérni | Véletlenszerű hozzáférést és összesítés | Soros hozzáférés | Közvetlen elérésű |
| Indexelés | Az elsődleges és másodlagos indexek | Csak az elsődleges index | – |
| Adatalakzat | Táblázatos | Blobok és metaadatok | Dokumentum |
| Ritka | Nem | – | Nem |
| Wide (nagy mennyiségű vagy attribútum) |  Nem | Igen | Igen |  
| Datum mérete | Kicsi (KB) | Nagy (GB) nagyon nagy (TB-osra bővül) | Kicsi (KB) |
| A teljes maximális skála | Nagy (alacsony TB-osra bővül)  | Nagyon nagy (PBs) | Nagy (alacsony TB-osra bővül) |
