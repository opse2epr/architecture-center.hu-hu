---
title: "Adattároló kiválasztásának szempontjai"
description: "Az Azure compute beállítások áttekintése"
author: MikeWasson
ms.openlocfilehash: 7fb75cd334438c5b985fa04ad8afe3236f2391f8
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="criteria-for-choosing-a-data-store"></a>Adattároló kiválasztásának szempontjai

Azure támogatja számos különböző típusú adatok tárolási megoldások, mindegyik különböző szolgáltatásokat és képességeket biztosít. Ez a cikk ismerteti a összehasonlítási feltétel kiértékelésekor adattárat kell használnia. A cél, hogy a segítségével meghatározhatja, hogy mely adattípusok tároló megfelel a megoldás követelményei.

## <a name="general-considerations"></a>Általános megfontolások

Az összehasonlítás elindításához gyűjtse össze a következő információkat, mint az adatokkal kapcsolatos igények megfelelően. Ez az információ segít meghatározni, mely adattípusok tárolási igényeinek.

### <a name="functional-requirements"></a>Működési követelmények

- **Az adatformátum**. Milyen típusú adatok vannak, tárolni szándékozik? Használt típusok a következők: tranzakciós adatok, JSON objektumok, telemetriai, keresési indexek vagy egybesimított fájlokba.
- **Adatok mérete**. Hogyan nagy kell tárolni entitások vannak? Ezeket az entitásokat kell egyetlen dokumentum fenn kell tartani, vagy azok oszthatók több dokumentumok, táblák, gyűjtemények, és így tovább?
- **Méretezés és struktúra**. Mi az az általános tárolókapacitáson van szüksége? Hajtsa végre, amelyek várhatóan az adatok particionálása? 
- **Adatok kapcsolatok**. Az adatok kell egy-a-többhöz vagy a több-több kapcsolat támogatásához? A kapcsolatok magukat az adatokat egyik fontos része? Szükség lesz való csatlakozásra, és ugyanahhoz az adatkészlethez belül, vagy a külső adatkészletek adatait egyébként felhasználó? 
- **Konzisztencia-modell**. Mennyire fontos a azt egy csomópontban jelennek meg más csomópontokat, mielőtt további módosításokat hajthat végrehajtott frissítések? Fogadhat végleges konzisztencia? Szüksége van az egyes tranzakciókra vonatkozóan ACID garanciák?
- **Séma rugalmasságát**. Milyen sémák rendszer alkalmaz az adatok? Egy rögzített sémájába, a séma módosításkor megközelítés vagy a séma-a-olvasási megközelítést fog használni?
- **Párhuzamossági**. Milyen párhuzamossági mechanizmus kívánja használja, ha a frissítés és az adatok szinkronizálását? Az alkalmazás akkor fogja elvégezni, amely potenciálisan ütközhetnek számos frissítést. Ha igen, előfordulhat, hogy a rekord zárolás és pesszimista feldolgozási vezérlő igénylő. Másik lehetőségként támogathat egyidejű hozzáférések optimista vezérlők? Ha igen, egyszerű időbélyeg-alapú feldolgozási vezérlő elég, vagy van szüksége több verzió párhuzamossági vezérlő hozzáadott új funkció?
- **Adatátvitel**. A megoldás kell áthelyezni az adatokat más tárolók vagy adatraktárak ETL feladatok elvégzéséhez?
- **Adatok életciklus**. Az adatok írási van – egyszer, olvassa el a többhöz? Az mozgathatók ritkán vagy a cold storage?
- **Egyéb támogatott szolgáltatás**. Nem kell semmilyen más szolgáltatást, például a sémaérvényesítés, összesítési, indexelő, teljes szöveges keresés, MapReduce, vagy más lekérdezési képességeket?

### <a name="non-functional-requirements"></a>A nem működési követelmények

- **Teljesítmény és méretezhetőség**. Mik a adatok teljesítményre vonatkozó követelmények? Konkrét követelmények adatforgalmi adatfeldolgozást díjakat és az adatok feldolgozási sebességet van? Mik a elfogadható válaszidejét kérdez le, és az adatok egyszer okozhatnak összesítése? Hogyan nagy szükség van-e az adatokat tároló méretezést kívánó? A számítási feladatok van, további műveleteket olvasási vagy írási műveleteket?
- **Megbízhatóság**. Milyen általános SLA szüksége támogatásához? Milyen szintű hibatűrést van szüksége az adatfelhasználók számára? Milyen típusú biztonsági mentési és visszaállítási képességeket van szüksége? 
- **Replikációs**. Több replikák és régiók közötti elosztását kell az adatokat? Milyen típusú adatok replikációs képességek szükség van-e? 
- **Korlátok**. Egy adott adattár határain támogatja a követelmények a méretezés, a kapcsolatokat, és az átvitel száma? 

### <a name="management-and-cost"></a>Felügyeleti és a költséghatékonyság

- **A felügyelt**. Ha lehetséges, egy felügyelt adatok szolgáltatását használja, ha szüksége van, amely csak egy üzemeltetett IaaS-tárolóban található meghatározott jogosultságokat.
- **Régiónkénti elérhetőség**. Felügyelt szolgáltatások esetén érhető el a szolgáltatás az összes Azure-régiók? Nem a megoldás kell bizonyos Azure-régiók-ban üzemeltetett?
- **Hordozhatósága**. Az adatok kell áttelepíteni a helyszíni, külső adatközpontok vagy egyéb felhőalapú üzemeltetési környezeteket?
- **Licencelési**. Rendelkezik egy védett OSS licenc típusa és előnyben? Vannak-e egyéb külső korlátozások licenc típusa használhatja?
- **A teljes költség**. Mi az a teljes költség szempontjából, a szolgáltatás belül a megoldás használatával? Hány példányban kell futtatni, saját hasznos üzemidő és átviteli igényeinek támogatására? Vegye figyelembe a számítási műveletek költségét. Felügyelt szolgáltatásokat inkább akkor működési költséghatékony.
- **Költség hatékonyságának**. Képes az adatok tárolására, hatékonyan további költség partícióazonosító? Például lehet egy drága relációs adatbázisból nagy objektumok helyez át egy objektum áruházban?

### <a name="security"></a>Biztonság

- **Biztonság**. Milyen típusú titkosítást igényelnek? Kell-e titkosítását? Milyen hitelesítési módszert szeretné használni az adatokhoz történő kapcsolódáshoz?
- **Naplózás**. Milyen típusú naplózást szeretne létrehozni?
- **Hálózati követelmények**. Szüksége korlátozhatja, vagy ellenkező esetben az adatok hozzáférésének más hálózati erőforrások kezelése? Nem szükséges adatokat kell csak érhetők el az Azure-környezeten belül? Nem a szükséges adatokat az adott IP-címek és alhálózatok érhető el? Nem azt kell ki alkalmazásaiban vagy az üzemeltetett szolgáltatások a helyszíni vagy más külső adatközpontokban elérhető?

### <a name="devops"></a>DevOps

- **Szakértelem set**. Vannak-e bizonyos programozási nyelvek, operációs rendszerek és egyéb technológia, hogy a csoport az különösen adept? Vannak-e más, a csapat dolgozhat nehéz lenne?
- **Ügyfelek** helyes ügyfél támogatása a fejlesztési nyelveket van?

A következő szakaszok különböző adatokat tároló modellek munkaterhelés-profilt, az adattípusokat és példák az alkalmazási helyzetekre összehasonlítása.

## <a name="relational-database-management-systems-rdbms"></a>Relációs adatbázis-kezelő rendszerek (RDBMS)

<table>
<tr><td>**Számítási feladat**</td>
    <td>
        <ul>
            <li>Mindkét létrehozása új rekordok és adatok frissítése rendszeresen fordulhat elő.</li>
            <li>Több műveletet kell egy tranzakción belül kell végrehajtani.</li>
            <li>Az aggregátumfüggvényeket típusát végrehajtásához szükséges.</li>
            <li>Jelentéskészítő eszközök erős integrálva szükség.</li>
            <li>Kapcsolatok használatával adatbázis korlátozások érvényesek lesznek.</li>
            <li>Indexek használt lekérdezési teljesítmény optimalizálása érdekében.</li>
            <li>Lehetővé teszi, hogy az adatok meghatározott fájlcsoportokat a hozzáférést.</li>
        </ul>
    </td>
</tr>
<tr><td>**Adattípus**</td>
    <td>
        <ul>
            <li>Adatok magas normalizált.</li>
            <li>Adatbázis-sémák szükségesek, de lépnek érvénybe.</li>
            <li>Az adatbázis entitások közötti kapcsolatok több-a-többhöz.</li>
            <li>Korlátozások a sémában definiált és partíciószeletre vonatkozó adatokat az adatbázisban.</li>
            <li>Adatok integritása magas szintű igényel. Indexek és kapcsolatok kell pontosan kell tartani.</li>
            <li>Adatok az erős konzisztencia igényel. Tranzakciók, amely biztosítja az összes adat 100 %-os összes felhasználójához és folyamatok konzisztens módon fog működni.</li>
            <li>Egyes bejegyzések mérete olyan kis és közepes méretű lehet.</li>
        </ul>
    </td>
</tr>
<tr><td>**Példák**</td>
    <td>
        <ul>
            <li>Üzletági (emberi beruházási felügyeletet, az Ügyfélkapcsolat-kezelés, vállalati erőforrás tervezése)</li>
            <li>Készlet kezelése</li>
            <li>Jelentéskészítő adatbázis</li>
            <li>Könyvelés</li>
            <li>Eszközök kezelése</li>
            <li>Alap kezelése</li>
            <li>Kezelési</li>
        </ul>
    </td>
</tr>
</table>

## <a name="document-databases"></a>Dokumentum-adatbázisokat

<table>
<tr><td>**Számítási feladat**</td>
    <td>
        <ul>
            <li>Általános célú.</li>
            <li>Beszúrási és frissítési műveletek megegyeznek. Mindkét létrehozása új rekordok és adatok frissítése rendszeresen fordulhat elő.</li>
            <li>Nincs objektum relációs impedancia nem megfelelő. Dokumentumok jobban is megegyezhet alkalmazáskód használt objektum struktúrákat.</li>
            <li>Egyidejű hozzáférések optimista több gyakran használják.</li>
            <li>Adatok kell módosíthatók és dolgozza fel az alkalmazás fel.</li>
            <li>Adatok index több mező szükséges.</li>
            <li>Egyes dokumentumok beolvasni, és egyetlen blokkot megírva.</li>
    </td>
</tr>
<tr><td>**Adattípus**</td>
    <td>
        <ul>
            <li>Adatok deszerializálni normalizált módon kezelhetők.</li>
            <li>A dokumentum egyes adatok mérete viszonylag kicsi.</li>
            <li>Minden egyes dokumentumtípus használhatja a saját séma.</li>
            <li>Dokumentumok választható mezők szerepelhetnek.</li>
            <li>A dokumentum adata félig strukturált, ami azt jelenti, hogy adatokat minden mező nem feltétlenül típusait.</li>
            <li>Adatösszesítés esetén támogatott.</li>
        </ul>
    </td>
</tr>
<tr><td>**Példák**</td>
    <td>
        <ul>
            <li>Termékkatalógus</li>
            <li>Felhasználói fiókok</li>
            <li>Anyagjegyzék</li>
            <li>Személyre szabás</li>
            <li>Tartalomkezelés</li>
            <li>Operatív adatok</li>
            <li>Készlet kezelése</li>
            <li>Tranzakció előzményadatok</li>
            <li>A más NoSQL-tárolókon materializált nézet. Cserél fájl/BLOB indexelő.</li>
        </ul>
    </td>
</tr>
</table>

## <a name="keyvalue-stores"></a>Kulcs-érték tárolók

<table>
<tr><td>**Számítási feladat**</td>
    <td>
        <ul>
            <li>Adatok azonosítja, és egy azonosító kulcs, például a dictionary használatával érhető el.</li>
            <li>Nagymértékben méretezhető.</li>
            <li>Nem csatlakozik, lock vagy egyesítésekhez szükség.</li>
            <li>Nincs összesítési mechanizmusokat használják.</li>
            <li>Másodlagos indexek általában nem használják.</li>
        </ul>
    </td>
</tr>
<tr><td>**Adattípus**</td>
    <td>
        <ul>
            <li>Adatok mérete általában nagy.</li>
            <li>Minden kulcs társítva, amely egy nem felügyelt adatok BLOB egyetlen értéket.</li>
            <li>Nincs séma kényszerítési van.</li>
            <li>Nincs entitások közötti kapcsolatok.</li>
        </ul>
    </td>
</tr>
<tr><td>**Példák**</td>
    <td>
        <ul>
            <li>Adatok gyorsítótárazása</li>
            <li>Munkamenet-kezelés</li>
            <li>Felhasználói beállítások és profilok felügyeletének</li>
            <li>A termék javaslat és ad szolgál</li>
            <li>szótárak</li>
        </ul>
    </td>
</tr>
</table>

## <a name="graph-databases"></a>Graph-adatbázisok

<table>
<tr><td>**Számítási feladat**</td>
    <td>
        <ul>
            <li>Adatok elemek között kapcsolatok nagyon bonyolultak, kapcsolódó adatelemek között hány ugrást használata esetén.</li>
            <li>Az adatok elemek közötti kapcsolat dinamikusak, és változnak az idők.</li>
            <li>Objektumok közötti kapcsolatok első osztályú építését, anélkül, hogy a külső kulcsokat és átjárása illesztések.</li>
        </ul>
    </td>
</tr>
<tr><td>**Adattípus**</td>
    <td>
        <ul>
            <li>Adatok csomópontok és a kapcsolatok magában foglalja.</li>
            <li>Csomópontok a táblázat sorainak vagy JSON-dokumentumok hasonlóak.</li>
            <li>Kapcsolatok éppen olyan fontos, mint a csomópontok, és közvetlenül a lekérdezés nyelven érhetők el.</li>
            <li>Összetett objektumok, például egy személy, a telefonszámokat, az egyes különálló, kisebb csomópontok lehet osztani együtt traversable kapcsolatok </li>
        </ul>
    </td>
</tr>
<tr><td>**Példák**</td>
    <td>
        <ul>
            <li>Szervezeti diagramok</li>
            <li>Közösségi diagramjait</li>
            <li>Csalások észlelése</li>
            <li>Elemzés</li>
            <li>A javaslat motorok</li>
        </ul>
    </td>
</tr>
</table>

## <a name="column-family-databases"></a>Oszlop-család adatbázisok

<table>
<tr><td>**Számítási feladat**</td>
    <td>
        <ul>
            <li>A legtöbb oszlop-család adatbázisok rendkívül gyorsan írási műveletek végrehajtására.</li>
            <li>Frissítse és törlési műveletek ritkák.</li>
            <li>Célja, hogy nagyobb teljesítményt és alacsony késésű hozzáférést biztosítanak.</li>
            <li>Támogatja az egyszerű lekérdezés hozzáférésének egy meghatározott sokkal nagyobb rekord mezőit.</li>
            <li>Nagymértékben méretezhető.</li>
        </ul>
    </td>
</tr>
<tr><td>**Adattípus**</td>
    <td>
        <ul>
            <li>Egy kulcsoszlop és egy vagy több oszlopcsaláddal táblák adatokat tárolja.</li>
            <li>Egyes oszlopok az egyes sorok eltérőek lehetnek.</li>
            <li>Az egyes cellák get keresztül elért, és helyezze parancsok</li>
            <li>Több sort visszaadja a vizsgálat paranccsal.</li>
        </ul>
    </td>
</tr>
<tr><td>**Példák**</td>
    <td>
        <ul>
            <li>Javaslatok</li>
            <li>Személyre szabás</li>
            <li>Érzékelői adatok</li>
            <li>Telemetria</li>
            <li>Üzenetküldés</li>
            <li>Közösségi médiaelemzés használatával</li>
            <li>Webes elemzőeszközt</li>
            <li>Figyelése</li>
            <li>Időjárás és egyéb idősorozat adatok</li>
        </ul>
    </td>
</tr>
</table>

## <a name="search-engine-databases"></a>Keresési adatbázisok

<table>
<tr><td>**Számítási feladat**</td>
    <td>
        <ul>
            <li>Az indexelési adatok több forrásokból és szolgáltatásokat.</li>
            <li>Lekérdezések alkalmi és összetett feladat lehet.</li>
            <li>Megköveteli összesítési.</li>
            <li>Teljes szöveges keresés szükség.</li>
            <li>Ad hoc önkiszolgáló lekérdezés megadása kötelező.</li>
            <li>Minden mező indexével Data elemzésre szükség.</li>
        </ul>
    </td>
</tr>
<tr><td>**Adattípus**</td>
    <td>
        <ul>
            <li>Félig strukturált vagy strukturálatlan</li>
            <li>Szöveg</li>
            <li>Szöveg strukturált adatok alapján</li>
        </ul>
    </td>
</tr>
<tr><td>**Példák**</td>
    <td>
        <ul>
            <li>A termék katalógusok</li>
            <li>Hely keresése</li>
            <li>Naplózás</li>
            <li>Elemzés</li>
            <li>Helyek vásárlás</li>
        </ul>
    </td>
</tr>
</table>

## <a name="data-warehouse"></a>Adattárház

<table>
<tr><td>**Számítási feladat**</td>
    <td>
        <ul>
            <li>Adatelemzés</li>
            <li>Vállalati BI   </li>
        </ul>
    </td>
</tr>
<tr><td>**Adattípus**</td>
    <td>
        <ul>
            <li>A különböző forrásokból származó előzményadatok.</li>
            <li>"Csillag" vagy "snowflake" séma, amely tény-és dimenziótáblák általában denormalizált.</li>
            <li>Általában be az új adatokkal ütemezés szerint.</li>
            <li>Dimenziótáblák többek között általában egy entitás néven több történelmi verziója egy *lassan a dimenzió módosításával*.</li>
        </ul>
    </td>
</tr>
<tr><td>**Példák**</td>
    <td>A vállalati adatokat biztosít a elemzési modellek, jelentések és irányítópultok az adatraktár.
    </td>
</tr>
</table>


## <a name="time-series-databases"></a>Idő adatsorozat adatbázisok

<table>
<tr><td>**Számítási feladat**</td>
    <td>
        <ul>
            <li>Egy túlnyomórészt műveletek (95-99 %) hányadát-e írási műveleteket.</li>
            <li>Rekord idő sorrendben általában hozzáfűződik egymás után.</li>
            <li>Frissítések ritkák.</li>
            <li>Törlések tömeges fordul elő, és egymást követő blokkot vagy rekordok történik.</li>
            <li>Olvasási kérések lehet nagyobb, mint a rendelkezésre álló memória.</li>
            <li>Esetében gyakori, hogy mi történjen, párhuzamosan több olvasása.</li>
            <li>Adatolvasás egymás után növekvő vagy csökkenő sorrendben idő.</li>
        </ul>
    </td>
</tr>
<tr><td>**Adattípus**</td>
    <td>
        <ul>
            <li>A primary key és rendezési mechanizmust, amely időbélyegző.</li>
            <li>A belépési vagy a bejegyzést jelöli leírását mérték.</li>
            <li>További információt a típusát, a forrás és az egyéb információkat a bejegyzés megadása címkéket.</li>
        </ul>
    </td>
</tr>
<tr><td>**Példák**</td>
    <td>
        <ul>
            <li>Figyelés és az esemény telemetriai adatokat.</li>
            <li>Érzékelő vagy egyéb IoT-adatokat.</li>
        </ul>
    </td>
</tr>
</table>

## <a name="object-storage"></a>Objektumtár

<table>
<tr><td>**Számítási feladat**</td>
    <td>
        <ul>
            <li>Kulcs azonosítja.</li>
            <li>Lehet, hogy objektumok nyilvánosan vagy egyénileg érhető el.</li>
            <li>Tartalom van általában egy eszköz, például egy táblázat, kép vagy videofájl.</li>
            <li>Lehet, hogy a tartalom tartós (állandó), és a külső alkalmazás réteg vagy virtuális gép.</li>
        </ul>
    </td>
</tr>
<tr><td>**Adattípus**</td>
    <td>
        <ul>
            <li>Adatok mérete nagy.</li>
            <li>Blobadatok.</li>
            <li>Értéke nem átlátszó.</li>
        </ul>
    </td>
</tr>
<tr><td>**Példák**</td>
    <td>
        <ul>
            <li>Képek, videók, office-dokumentumok, PDF-fájlok</li>
            <li>CSS, parancsfájlok, CSV</li>
            <li>Statikus HTML, JSON</li>
            <li>Napló-és naplózás</li>
            <li>Adatbázisok biztonsági mentése</li>
        </ul>
    </td>
</tr>
</table>

## <a name="shared-files"></a>Megosztott fájlok

<table>
<tr><td>**Számítási feladat**</td>
    <td>
        <ul>
            <li>Áttelepítés a meglévő alkalmazás, amely interakciót folytatni a fájlrendszerben.</li>
            <li>SMB-kapcsolat szükséges.</li>
        </ul>
    </td>
</tr>
<tr><td>**Adattípus**</td>
    <td>
        <ul>
            <li>Fájlok, mappák hierarchikus meg.</li>
            <li>Elérhető a szabványos i/o-könyvtárak.</li>
        </ul>
    </td>
</tr>
<tr><td>**Példák**</td>
    <td>
        <ul>
            <li>A hagyományos fájlok</li>
            <li>Megosztott tartalom érhető el a virtuális gépek vagy az app-példányok száma között</li>
        </ul>
    </td>
</tr>
</table>
