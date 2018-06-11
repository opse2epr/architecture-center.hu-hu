---
title: Az adattár kiválasztásának kritériumai
description: Az Azure számítási lehetőségeinek áttekintése
author: MikeWasson
ms.openlocfilehash: 70f746f80c29623004620d83eb38747777df7f84
ms.sourcegitcommit: 85334ab0ccb072dac80de78aa82bcfa0f0044d3f
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 06/11/2018
ms.locfileid: "35252873"
---
# <a name="criteria-for-choosing-a-data-store"></a>Az adattár kiválasztásának kritériumai

Az Azure számos adattárolási megoldástípust támogat, amelyek mindegyike más funkciókat és képességeket biztosít. Ez a cikk az adattárak értékelésekor szükséges összehasonlítási kritériumokat ismerteti. A cél az, hogy segítséget nyújtson az aktuális megoldás igényeinek megfelelő adattárolási típus meghatározásában.

## <a name="general-considerations"></a>Általános szempontok

Az összehasonlítás megkezdéséhez gyűjtsön annyit az alábbi, az adatigényeire vonatkozó információkból, amennyit csak tud. Ez az információ segít annak meghatározásában, hogy melyik adattárolási típus felel meg az igényeinek.

### <a name="functional-requirements"></a>Funkcionális követelmények

- **Adatformátum**. Milyen típusú adatokat szándékozik tárolni? A gyakori típusok közé tartoznak a tranzakciós adatok, a JSON-objektumok, a telemetriaadatok, a keresési indexek és az egybesimított fájlok.
- **Az adatok mérete**. Mekkora a tárolni kívánt entitások mérete? Ezeket az entitásokat egyetlen dokumentumként kell fenntartani, vagy azok feloszthatók több dokumentumra, táblára, gyűjteményre stb.?
- **Skálázás és struktúra**. Milyen általános tárolókapacitásra van szüksége? Tervezi az adatok particionálását? 
- **Adatkapcsolatok**. Az adatoknak támogatniuk kell-e az egy-a-többhöz vagy a több-a-többhöz kapcsolatokat? Maguk a kapcsolatok fontos részét képezik az adatoknak? Szükség lesz-e az adatok egyesítésére vagy egyéb típusú kombinálására ugyanazon vagy más külső adatkészletekből származó adatokkal? 
- **Konzisztenciamodell**. Mennyire fontos, hogy az egy csomóponton végzett frissítések más csomópontokon is megjelenjenek, mielőtt további módosításokat lehetne eszközölni? Elfogadható-e a végleges konzisztencia? Szüksége van-e ACID-garanciákra az egyes tranzakciókhoz?
- **A séma rugalmassága**. Milyen sémát fog alkalmazni az adatokon? Rögzített sémát, esetleg írási séma vagy olvasási séma megközelítést fog használni?
- **Egyidejűség**. Milyen egyidejűségi mechanizmust kíván használni az adatok frissítésekor és szinkronizálásakor? Az alkalmazás sok olyan frissítést fog végezni, amelyek ütközhetnek? Ha igen, a rekordok zárolására és pesszimista feldolgozási vezérlő lehet szükség. Vagy támogathatja-e az optimista egyidejűségi vezérlőket? Ha igen, elegendő-e az egyszerű időbélyegző-alapú egyidejűségi vezérlés, vagy szüksége van-e többverziós egyidejűségi vezérlésre?
- **Adatáthelyezés**. A megoldásnak végre kell-e hajtania ETL-feladatokat az adatok másik tárolóba vagy adattárházba történő áthelyezéséhez?
- **Az adatok életciklusa**. Az adatok egyszer írhatók és többször olvashatók? Áthelyezhetők-e ritkán használt vagy offline tárhelyre?
- **Egyéb támogatott funkciók**. Szüksége van-e bármilyen más konkrét funkcióra, például sémaérvényesítésre, összesítésre, indexelésre, teljes szöveges keresésre, esetleg MapReduce-feladatra vagy egyéb lekérdezési képességekre?

### <a name="non-functional-requirements"></a>Nem funkcionális követelmények

- **Teljesítmény és skálázhatóság**. Milyen igényei vannak az adatteljesítményre vonatkozóan? Vannak-e egyedi igényei az adatok betöltése és feldolgozása szempontjából? Milyen elfogadható válaszidők vonatkoznak az adatok lekérdezésére és összesítésére a beolvasás után? Milyen mértékű vertikális felskálázásra lesz szüksége az adattárhoz? A munkaterhelések nagyrészt olvasási vagy nagyrészt írási műveltekből állnak?
- **Megbízhatóság**. Milyen általános SLA-támogatásra van szüksége? Milyen szintű hibatűrést kíván nyújtani az adatfelhasználók számára? Milyen típusú biztonsági mentési és visszaállítási képességekre van szüksége? 
- **Replikáció**. Szükség lesz-e az adatok több replika vagy régió közötti szétosztására? Milyen típusú adatreplikációs képességekre van szüksége? 
- **Korlátok**. Beleférnek-e egy adott adattár korlátaiba a skálázási, a kapcsolatok számára vonatkozó, illetve az átviteli követelményei? 

### <a name="management-and-cost"></a>Felügyelet és költségek

- **Felügyelt szolgáltatás**. Ha lehetséges, használjon felügyelt adatszolgáltatást, hacsak nem kifejezetten olyan képességekre van szüksége, amelyek csak egy IaaS által üzemeltetett adattárban találhatók meg.
- **Régiónkénti rendelkezésre állás**. Felügyelt szolgáltatások esetén a szolgáltatás elérhető-e az összes Azure-régióban? A megoldást bizonyos Azure-régiókban kell-e üzemeltetni?
- **Hordozhatóság**. Szükség lesz-e az adatok helyszíni vagy külső adatközpontokba, esetleg egyéb felhőalapú üzemeltetési környezetekbe történő migrálására?
- **Licencelés**. Előnyben részesíti-e a jogvédett licencet az OSS licenctípussal szemben? Vonatkozik-e egyéb külső korlátozás a használt licenc típusára vonatkozóan?
- **A teljes költség**. Milyen mértékű a megoldásban használt szolgáltatás teljes költsége? Hány példánynak kell futnia az üzemidő- és a teljesítménybeli követelmények kiszolgálásához? Ennél a számításnál az üzemeltetés költségeit is vegye figyelembe. A felügyelt szolgáltatások használatának egyik előnye az alacsonyabb üzemeltetési költség.
- **Költséghatékonyság**. Lehetséges-e az adatok particionálása a költséghatékonyabb tárolás érdekében? Például áthelyezhetők-e a nagy méretű objektumok egy költséges relációs adatbázisból egy objektumtárolóba?

### <a name="security"></a>Biztonság

- **Biztonság**. Milyen típusú titkosításra van szüksége? Szüksége van titkosításra inaktív állapotban? Milyen hitelesítési mechanizmust szeretne használni az adatokhoz való kapcsolódáshoz?
- **Naplózás**. Milyen típusú naplót szeretne létrehozni?
- **Hálózati követelmények**. Szeretné-e korlátozni vagy egyéb módon kezelni a más hálózati forrásokból származó adatokat? Az adatok elérésére csak az Azure-környezeten belülről van szükség? Az adatoknak adott IP-címekről vagy alhálózatokról is elérhetőnek kell lenniük? Elérhetőnek kell-e lenniük a helyszínen üzemeltetett alkalmazások vagy szolgáltatások, esetleg egyéb külső adatközpontok számára?

### <a name="devops"></a>DevOps

- **Képzettség**. Vannak-e olyan programozási nyelvek, operációs rendszerek és egyéb technológiák, amelyek használatában a csapata különösen képzett? Vannak-e olyanok, amelyekkel a csapata nehezen boldogulna?
- **Ügyfelek**. Elérhető-e megfelelő szintű ügyféltámogatás az Ön által használt fejlesztési nyelvekhez?

A következő szakaszokban különböző adattármodelleket hasonlítunk össze a számításifeladat-profil, az adattípusok és az alkalmazási helyzetekre vonatkozó példák alapján.

## <a name="relational-database-management-systems-rdbms"></a>Relációsadatbázis-kezelő rendszerek (RDBMS)

<table>
<tr><td><strong>Számítási feladat</strong></td>
    <td>
        <ul>
            <li>Mind az új rekordok létrehozása, mind a meglévő adatok frissítése rendszeresen megtörténik.</li>
            <li>Több műveletet kell egyetlen tranzakción belül végrehajtani.</li>
            <li>Kereszttáblák létrehozásához összesítő függvényekre van szükség.</li>
            <li>Jól integrálhatónak kell lennie a jelentéskészítő eszközökkel.</li>
            <li>A kapcsolatok kényszerítése az adatbázis korlátozásai révén valósul meg.</li>
            <li>A lekérdezési teljesítmény optimalizálása indexekkel történik.</li>
            <li>Lehetővé teszi az adatok bizonyos részhalmazainak elérését.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Adattípus</strong></td>
    <td>
        <ul>
            <li>Az adatok nagymértékben normalizáltak.</li>
            <li>Az adatbázissémák szükségesek és kényszerítettek.</li>
            <li>Több-a-többhöz kapcsolatok az adatbázisban szereplő adatentitások között.</li>
            <li>A korlátozásokat a séma határozza meg, és alkalmazza az adatbázisban szereplő minden adatra.</li>
            <li>Szükséges az adatok nagy fokú integritása. Az indexeket és kapcsolatokat pontosan karban kell tartani.</li>
            <li>Szükséges az adatok erős konzisztenciája. A tranzakciók működésének módja biztosítja, hogy minden adat minden felhasználó és folyamat számára teljes mértékben megegyezzen.</li>
            <li>Az egyes adatbeviteleknek kis vagy közepes méretűnek kell lenniük.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Példák</strong></td>
    <td>
        <ul>
            <li>Üzletág (emberierőforrás-kezelés, ügyfélkapcsolat-kezelés, vállalati erőforrás-tervezés)</li>
            <li>Leltárkezelés</li>
            <li>Jelentéskészítési adatbázis</li>
            <li>Könyvelés</li>
            <li>Eszközkezelés</li>
            <li>Alaptőke-kezelés</li>
            <li>Rendeléskezelés</li>
        </ul>
    </td>
</tr>
</table>

## <a name="document-databases"></a>Dokumentum-adatbázisok

<table>
<tr><td><strong>Számítási feladat</strong></td>
    <td>
        <ul>
            <li>Általános célú.</li>
            <li>Gyakoriak a beszúrási és frissítési műveletek. Mind az új rekordok létrehozása, mind a meglévő adatok frissítése rendszeresen megtörténik.</li>
            <li>Nincs objektumrelációs impedanica-eltérés. A dokumentumok jobban illeszkednek az alkalmazáskódban használt objektumstruktúrákhoz.</li>
            <li>Gyakrabban használnak optimista konkurenciát.</li>
            <li>Az adatokat a felhasználó alkalmazásnak kell módosítania és feldolgoznia.</li>
            <li>Az adatokhoz többmezős indexelésre van szükség.</li>
            <li>Egyes dokumentumokat egyetlen blokként olvas be és ír a rendszer.</li>
    </td>
</tr>
<tr><td><strong>Adattípus</strong></td>
    <td>
        <ul>
            <li>Az adatok denormalizált módon is kezelhetők.</li>
            <li>Az egyes dokumentumadatok mérete viszonylag kicsi.</li>
            <li>Minden egyes dokumentumtípus használhatja a saját sémáját.</li>
            <li>A dokumentumok nem kötelező mezőket is tartalmazhatnak.</li>
            <li>A dokumentumadatok részben strukturáltak, ami azt jelenti, hogy az egyes mezők adattípusai nincsenek szigorúan meghatározva.</li>
            <li>Az adatösszesítés támogatott.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Példák</strong></td>
    <td>
        <ul>
            <li>Termékkatalógus</li>
            <li>Felhasználói fiókok</li>
            <li>Anyagjegyzék</li>
            <li>Személyre szabás</li>
            <li>Tartalomkezelés</li>
            <li>Műveleti adatok</li>
            <li>Leltárkezelés</li>
            <li>Tranzakció-előzményadatok</li>
            <li>Más NoSQL-tárolók materializált nézete. Fájl cseréje/BLOB-indexelés.</li>
        </ul>
    </td>
</tr>
</table>

## <a name="keyvalue-stores"></a>Kulcs/érték tárolók

<table>
<tr><td><strong>Számítási feladat</strong></td>
    <td>
        <ul>
            <li>Az adatokat egy szótárhoz hasonlóan azonosítja és éri el.</li>
            <li>Nagy mértékben skálázható.</li>
            <li>Nincs szükség illesztésre, zárolásra vagy egyesítésre.</li>
            <li>Nem használ összesítési mechanizmusokat.</li>
            <li>A másodlagos indexeket általában nem használja.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Adattípus</strong></td>
    <td>
        <ul>
            <li>Az adatok mérete általában nagy.</li>
            <li>Minden kulcs egyetlen értékhez van társítva, amely egy nem felügyelt adatblob.</li>
            <li>Nincs sémakényszerítés.</li>
            <li>Nincsenek entitások közötti kapcsolatok.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Példák</strong></td>
    <td>
        <ul>
            <li>Adatok gyorsítótárazása</li>
            <li>Munkamenet-kezelés</li>
            <li>Felhasználói beállítások és profilkezelés</li>
            <li>Termékjavaslat és reklámszolgáltatás</li>
            <li>Szótárak</li>
        </ul>
    </td>
</tr>
</table>

## <a name="graph-databases"></a>Gráfadatbázisok

<table>
<tr><td><strong>Számítási feladat</strong></td>
    <td>
        <ul>
            <li>Az adatelemek közötti kapcsolatok nagyon összetettek, számos ugrással a kapcsolódó adatelemek között.</li>
            <li>Az adatelemek közötti kapcsolat dinamikus, és az idő előrehaladtával változik.</li>
            <li>Az objektumok közötti kapcsolatok kiváló hozzáférést biztosítanak, a bejáráshoz nincs szükségük külső kulcsokra vagy illesztésekre.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Adattípus</strong></td>
    <td>
        <ul>
            <li>Az adatok csomópontokból és kapcsolatokból állnak.</li>
            <li>A csomópontok a táblázatok soraihoz vagy a JSON-dokumentumokhoz hasonlítanak.</li>
            <li>A kapcsolatok épp olyan fontosak, mint a csomópontok, és a lekérdezési nyelvben közvetlenül megjelennek.</li>
            <li>Az összetett objektumokat (például egy több telefonszámmal rendelkező személy) a rendszer általában több kisebb csomópontra osztja szét, amelyeket bejárható kapcsolatok kötnek össze. </li>
        </ul>
    </td>
</tr>
<tr><td><strong>Példák</strong></td>
    <td>
        <ul>
            <li>Szervezeti diagramok</li>
            <li>Közösségi diagramok</li>
            <li>Csalások észlelése</li>
            <li>Elemzés</li>
            <li>Javaslati motorok</li>
        </ul>
    </td>
</tr>
</table>

## <a name="column-family-databases"></a>Oszlopcsalád-adatbázisok

<table>
<tr><td><strong>Számítási feladat</strong></td>
    <td>
        <ul>
            <li>A legtöbb oszlopcsalád-adatbázis rendkívül gyorsan hajtja végre az írási műveleteket.</li>
            <li>A frissítési és törlési műveletek ritkák.</li>
            <li>Nagy teljesítmény és kis késleltetésű hozzáférés biztosítására tervezték.</li>
            <li>Támogatja egy jóval nagyobb méretű rekordon belüli adott mezőkészletre irányuló egyszerű lekérdezési hozzáféréseket.</li>
            <li>Nagy mértékben skálázható.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Adattípus</strong></td>
    <td>
        <ul>
            <li>Az adatok egy kulcsoszlopból és egy vagy több oszlopcsaládból álló táblákban tárolódnak.</li>
            <li>Az adott oszlopok az egyes sorokban eltérőek lehetnek.</li>
            <li>Az egyes cellák „get” és „put” parancsokkal érhetők el.</li>
            <li>Egy vizsgálat paranccsal több sort ad vissza.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Példák</strong></td>
    <td>
        <ul>
            <li>Javaslatok</li>
            <li>Személyre szabás</li>
            <li>Érzékelői adatok</li>
            <li>Telemetria</li>
            <li>Üzenetkezelés</li>
            <li>Közösségi médiaelemzés</li>
            <li>Webes elemzés</li>
            <li>Tevékenység figyelése</li>
            <li>Időjárási és egyéb idősorozat-adatok</li>
        </ul>
    </td>
</tr>
</table>

## <a name="search-engine-databases"></a>Keresőmotor-adatbázisok

<table>
<tr><td><strong>Számítási feladat</strong></td>
    <td>
        <ul>
            <li>Több forrásból és szolgáltatástól származó adatok indexelése.</li>
            <li>A lekérdezések ad-hoc jellegűek, és összetettek lehetnek.</li>
            <li>Összesítést igényel.</li>
            <li>Teljes szöveges keresést igényel.</li>
            <li>Ad-hoc jellegű önkiszolgáló lekérdezéseket igényel.</li>
            <li>Minden mező indexelésével végzett adatelemzésre van szükség.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Adattípus</strong></td>
    <td>
        <ul>
            <li>Részben strukturált, vagy strukturálatlan</li>
            <li>Szöveg</li>
            <li>Strukturált adatokra hivatkozó szöveg</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Példák</strong></td>
    <td>
        <ul>
            <li>Termékkatalógusok</li>
            <li>Keresés webhelyen</li>
            <li>Naplózás</li>
            <li>Elemzés</li>
            <li>Vásárlási webhelyek</li>
        </ul>
    </td>
</tr>
</table>

## <a name="data-warehouse"></a>Adattárház

<table>
<tr><td><strong>Számítási feladat</strong></td>
    <td>
        <ul>
            <li>Adatelemzés</li>
            <li>Enterprise BI   </li>
        </ul>
    </td>
</tr>
<tr><td><strong>Adattípus</strong></td>
    <td>
        <ul>
            <li>Több forrásból származó előzményadatok.</li>
            <li>Általában a denormalizált egy &quot;csillag&quot; vagy &quot;snowflake&quot; séma-, tény-és dimenziótáblák álló.</li>
            <li>Általában ütemezés szerint töltődik fel új adatokkal.</li>
            <li>A dimenziótáblák gyakran egy entitás több korábbi verzióját is tartalmazzák, amit <em>lassan változó dimenziónak</em> nevezünk.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Példák</strong></td>
    <td>Egy vállalati adattárház, amely elemzési modellek, jelentések és irányítópultok számára biztosít adatokat.
    </td>
</tr>
</table>


## <a name="time-series-databases"></a>Idősorozat-adatbázisok

<table>
<tr><td><strong>Számítási feladat</strong></td>
    <td>
        <ul>
            <li>Műveletek (95-99 %) egy túlságosan hányadát írási műveletek.</li>
            <li>A rekordok hozzáfűzése általában időrend szerint történik.</li>
            <li>A frissítések ritkák.</li>
            <li>A törlések tömegesen fordulnak elő, céljaik általában egybefüggő blokkok vagy rekordok.</li>
            <li>Az olvasási kérések lehetnek az elérhető memóriánál nagyobb méretűek.</li>
            <li>Az&#39;s gyakori, hogy mi történjen, párhuzamosan több olvasása.</li>
            <li>A rendszer az adatokat szakaszosan olvassa be időrend szerint növekvő vagy csökkenő sorrendben.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Adattípus</strong></td>
    <td>
        <ul>
            <li>Az elsődleges kulcsként és rendezési mechanizmusként használt időbélyegző.</li>
            <li>A bejegyzésből származó mérések vagy a bejegyzés által képviselt elemek leírása.</li>
            <li>A bejegyzés típusára, eredetére és egyéb információira vonatkozó további adatokat meghatározó címkék.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Példák</strong></td>
    <td>
        <ul>
            <li>Monitorozás és eseménytelemetria.</li>
            <li>Érzékelő- és egyéb IoT-adatok.</li>
        </ul>
    </td>
</tr>
</table>

## <a name="object-storage"></a>Objektumtár

<table>
<tr><td><strong>Számítási feladat</strong></td>
    <td>
        <ul>
            <li>Kulcs azonosítja.</li>
            <li>Az objektumok nyilvánosan vagy privát módon is hozzáférhetők.</li>
            <li>A tartalom általában egy táblázat, kép- vagy videofájl vagy hasonló adategység.</li>
            <li>A tartalomnak tartósnak (állandónak) kell lennie, és bármely alkalmazásszinten vagy virtuális gépen külső tartalomnak kell lennie.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Adattípus</strong></td>
    <td>
        <ul>
            <li>Az adatok mérete nagy.</li>
            <li>Blobadatok.</li>
            <li>Az érték nem átlátszó.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Példák</strong></td>
    <td>
        <ul>
            <li>Képek, videók, Office-dokumentumok, PDF-fájlok</li>
            <li>CSS, szkriptek, CSV</li>
            <li>Statikus HTML, JSON</li>
            <li>Napló- és vizsgálati fájlok</li>
            <li>Adatbázisok biztonsági mentése</li>
        </ul>
    </td>
</tr>
</table>

## <a name="shared-files"></a>Megosztott fájlok

<table>
<tr><td><strong>Számítási feladat</strong></td>
    <td>
        <ul>
            <li>Áttelepítés meglévő alkalmazásokból, amelyek kommunikálnak a fájlrendszerrel.</li>
            <li>SMB-kapcsolatot igényel.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Adattípus</strong></td>
    <td>
        <ul>
            <li>Egy hierarchikus mappakészlet fájljai.</li>
            <li>Szabványos I/O-kódtárakkal elérhető.</li>
        </ul>
    </td>
</tr>
<tr><td><strong>Példák</strong></td>
    <td>
        <ul>
            <li>Örökölt fájlok</li>
            <li>A megosztott tartalom bizonyos számú virtuális gép vagy alkalmazáspéldány számára elérhető</li>
        </ul>
    </td>
</tr>
</table>
