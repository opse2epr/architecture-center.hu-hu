---
title: 'CAF: A digitális tulajdon észszerűsítése'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
description: Értékelje ki a digitális eszközök segítségével meghatározhatja, hogy az hogyan legjobban üzemeltetéséhez, őket a felhőben.
author: BrianBlanchard
ms.date: 12/10/2018
ms.topic: guide
ms.openlocfilehash: 02189c9edcbfea0a55fe69a53bf610e85470a4d0
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55897201"
---
# <a name="rationalize-the-digital-estate"></a>A digitális tulajdon észszerűsítése

Felhőbeli ésszerűsítés az a folyamat kiértékelése adategységeket az üzemeltető őket a felhőben a legjobb módja határozza meg. Egyszer egy [megközelítés](approach.md) meghatározása és [készlet](inventory.md) lett összesítve, felhőalapú ésszerűsítés megkezdheti. A [ésszerűsítés az 5 Rs](5-rs-of-rationalization.md) a leggyakoribb ésszerűsítés lehetőségeket ismerteti.

## <a name="traditional-view-of-rationalization"></a>Ésszerűsítés hagyományos nézete

Legyen könnyen érthető ésszerűsítés, amikor jelenítenek meg, például egy összetett döntési fa ésszerűsítés hagyományos folyamatán. Az egyes eszközök a digitális hagyatéki adatkéréseket egy folyamatot, amely öt válaszok (az 5 Rs) egyik eredményez. Kis ingatlanadót Ez a folyamat jól működik. A nagyobb ingatlanadót még nem nagyon hatékony és jelentős késés vezethet. Most vizsgálja meg a folyamat infrastruktúrákról. Ezután azt fogja hatékonyabb modell jelenthet.

**Készlet**. Az eszközök, alkalmazások, szoftverek, hardverek, operációs rendszerek és rendszer teljesítmény-mérőszámok többek között egy alapos leltár egy hagyományos modellek használata teljes ésszerűsítés végrehajtásához szükséges.

**A mennyiségi elemzés**. A döntési fában mennyiségi kérdések meghajtó döntések első réteget. Gyakori kérdések: Jelenleg használatban van az eszköz? Ha igen, az azt optimalizált és megfelelően méretezve? Milyen függőség létezik eszközök között? Ezeket a kérdéseket létfontosságúak a besoroláshoz, a leltárba.

**Minőségi elemzési**. A következő készletét döntések emberi intelligenciával minőségi elemzésével formájában van szükség. Ezeket a kérdéseket gyakran egyediek a megoldáshoz, és üzleti résztvevőkkel, és a kiemelt felhasználók csak választ. Ezek a döntések vannak, a folyamat általában késik, lelassítanunk dolgot jelentősen. Ez az elemzés általában felhasznál a 40&ndash;80 FTE órát alkalmazásonként. Az alkalmazásfejlesztés minőségi elemzési kérdések listája útmutatásért lásd: [megközelítések a digitális hagyatéki tervezési](approach.md).

**Ésszerűsítés döntési**. Egy tapasztalt ésszerűsítés csapat tagoknál a mennyiségi és minőségi adatok törlése döntéseket hoz létre. Sajnos a magas fokú ésszerűsítés élmény csapatok drágák hire vagy hónap betanításához.

## <a name="rationalization-at-enterprise-scale"></a>Nagyvállalati szintű ésszerűsítés

Ha ebből a törekvésből időigényes és a egy 50 – virtuális gép digitális hagyatéki tűnhet, imagine meghajtó vállalatok átalakulását a több ezer virtuális gép és az alkalmazások több száz olyan környezetben, szükséges munkát. 1500 FTE óra és 9 hónapig tervezése könnyen lépheti túl a szükséges emberi beavatkozást.

Bár teljes ésszerűsítés befejezési állapota és a egy nagyszerű iránya a digitalizáció felé, ritkán küld egy magas megtérülés az időt és energiát viszonyítva.

Pénzügyi döntések ésszerűsítés elengedhetetlen, ha egy szakmai szervezet, amely a folyamat felgyorsítása felhőalapú ésszerűsítés figyelembe vételével érdemes. Akkor is ezt követően teljes ésszerűsítés lehet költséges és időigényes erőfeszítés sikerült késleltetés, átalakítási és az üzleti eredményeket.

Ez a cikk további része egy másik módszert, más néven növekményes ésszerűsítés ismerteti.

## <a name="incremental-rationalization"></a>Növekményes ésszerűsítés

A teljes ésszerűsítés az egy nagyméretű digitális hagyatéki hajlamos arra, hogy fennáll a veszélye, és az késleltetések összetettséget társított esetén csökkenhet a. A növekményes megközelítés mögött feltételezése, hogy késleltetett döntések fog eltolja abban az esetben a terhelés az üzleti akadályát kockázatának csökkentése érdekében. Idővel Ez a megközelítés a folyamatok és minősített ésszerűsítés döntéseket hozhat, és ehhez hatékony szükséges élmény fejlesztéséhez egy organikus modellt hoz létre.

### <a name="inventory-reduce-discovery-data-points"></a>Készlet: Csökkentheti a felderítés adatpontok

Nagyon néhány szervezetnek az idő, az energia és a költségek, hogy pontos, valós idejű, a teljes digitális hagyatéki befektetni. Elvesztése, ellopása, adatfrissítési ciklusok és az előkészítési gyakran alkalmazott adja meg az eszközintelligencia részletes követési végfelhasználói eszközök. Fordított beruházások (ROI) egy pontos kiszolgáló és a egy hagyományos, helyszíni adatközpontban leltár karbantartása azonban gyakran értéke alacsony. A legtöbb informatikai szervezetek rendelkezik más további legégetőbb felmerülő problémákkal foglalkoznak, mint az egy kínai adatközpont rögzített adategységek használatának követéséről.

Egy felhőalapú átalakításában szerepel a készlet közvetlenül ad eredményül, amelyek üzemeltetési költségek. Pontos leltáradatok kötelező megadni a megfelelő tervezést. Aktuális környezet vizsgálati beállítások sajnos késleltetheti döntések hetek vagy hónapok, vagy a katalógus a teljes készlet. Szerencsére a nincsenek néhány trükköket, felgyorsítja az adatgyűjtést.

Az ügynök-alapú ellenőrzés be kapcsolva a leggyakrabban idézett késleltetés. A robusztus adat, gyakran hagyományos ésszerűsítés szükséges minden eszköz futó ügynök csak gyűjtendő adatok függ. Ez a függőségi ügynökök gyakran lelassul folyamatban, a, ha azt biztonság, műveleteket és felügyeleti funkciók visszajelzései lehet szükség.

Egy növekményes ésszerűsítés folyamatban az ügynök nélküli megoldás használható egy kezdeti felderítés felgyorsítása korai döntéseket hozhat. A környezet összetettsége szintjétől függően egy ügynök-alapú megoldás továbbra is szükség lehet, de el kell távolítani a kritikus útvonalat az üzleti módosítása.

### <a name="quantitative-analysis-streamline-decisions"></a>A mennyiségi elemzés: Egyszerűsíthetők a döntéseket hozhat

A felderítési leltár megközelítés, függetlenül a mennyiségi elemzés ösztönözheti a kezdeti döntések és feltevések. Ez különösen igaz az első számítási feladatok azonosítása tett kísérlet során, vagy ha ésszerűsítés célja a magas szintű költség összehasonlítása. Egy növekményes ésszerűsítés folyamatban a Felhőstratégia és a felhőre való áttérés csapatok korlátozza a [ésszerűsítés 5 Rs](5-rs-of-rationalization.md) két tömör döntésekhez, és csak a alkalmazni ezeket mennyiségi tényezők, egyszerűsítheti az elemzés és csökkentése a meghajtó módosítása szükséges kezdeti adatok mennyisége.

Például ha egy szervezet in the midst of IaaS-migrálás a felhőbe, feltételezhető, hogy a legtöbb számítási feladatok fog elavult, vagy áthelyezett.

### <a name="qualitative-analysis-temporary-assumptions"></a>Minőségi-elemzés: Ideiglenes feltételezések

A lehetséges kimenetek számának csökkentésével egyszerűbb legyen a jövőbeli az eszköz állapotával kapcsolatos kezdeti döntést. Ha csökkentik a beállításokat, az üzleti ez korai szakaszában feltett kérdések számát is csökken.

A fenti példa folytatása, ha a lehetőségek korlátozottak áthelyezési vagy teljes, valójában csak egy kérdést tehet fel és üzleti ésszerűsítés kezdeti során &mdash; kivonása,-e.

"Analysis javasolja, hogy nincsenek-e az eszköz aktívan kihasználva felhasználók. Hogy pontos vagy kell hogy kihagyott valami? " Bináris kérdése általában sokkal könnyebb haladjon végig a minőségi elemzésre.

Ez a megközelítés egyszerű alapkonfigurációkat, pénzügyi tervek, stratégia és iránya eredményez. Minden eszköz későbbi tevékenységek további ésszerűsítés és egyéb beállítások kiértékeléséhez minőségi elemzés révén keresse meg lenne. A kezdeti ésszerűsítés az összes kalkulált végrehajtása előtt szeretne vizsgálni.

## <a name="challenging-assumptions"></a>Nagy kihívást jelentő feltételezések

Az előző szakaszban eredményét egy hozzávetőleges ésszerűsítés feltételezések betöltését. Következő lépésként, ideje fight ezen feltételezések közül.

### <a name="retiring-assets"></a>Eszközök kivonása

A hagyományos helyszíni környezetben kis méretű, használaton kívüli eszközök ritkán üzemeltető hatására jelentős hatással az éves költség. Néhány kivételtől eltekintve a költségmegtakarítást, törlési és azok az eszközök kivonásáról társított ellensúlyozza elemzésére és a tényleges eszköz kivonása szükséges FTE munkát.

Azonban a felhő számlázási modell áthelyezésekor eszközök kivonása hozhat létre az éves működési költségek és erőfeszítések kezdeti áttelepítés jelentős megtakarítást.

Már nem ritka, hogy a szervezetek számára, hogy 20 %-os vagy több saját digitális hagyatéki kivonása egy mennyiségi elemzés befejezése után. Meghatározza az ilyen művelet előtt ajánlott további minőségi elemzéseket végezhet. Miután megerősítette, ezek használatból való kivonást egyaránt az első ROI győzelem a felhőbe migrálás hozhat létre. Sok esetben ez az egyik legnagyobb költségcsökkentő tényezők. Emiatt a javasolt, hogy a Felhőstratégia csapat felügyeli az érvényesítési és a használatból való kivonást egyaránt az eszközök, a build fázisa az áttelepítési folyamat párhuzamos egy korai pénzügyi Win engedélyezéséhez.

### <a name="program-adjustments"></a>Program beállításai

A csak egy átalakulásunkhoz ritkán embarks egy vállalat. Költségek csökkentéséhez, a piaci növekedési és új bevételi forrásokat teremtve között, amely ritkán egy bináris döntés. Ezért javasoljuk, hogy a Felhőstratégia csapat dolgozhat, hogy a párhuzamos átalakítás folyamatában, az elsődleges Átalakulásunkhoz hatókörén kívüli eszközök azonosítása.

Az IaaS-Migrálás példában a cikk ezt használja:

- Kérje meg a fejlesztési és üzemeltetési csapat már üzembe helyezésének automatizálása egy részét képezik, és távolítsa el azokat az alapvető migrálási terv eszközök azonosításához.

- Kérje meg az adatokat és az R & D csapatok, amelyek támogatják a új bevételi források feltárását, és azokat eltávolítsa a core migrálási terv eszközök azonosításához.

A program témájú minőségi elemzési gyorsan hajthatók végre, és létrehozza a igazítás között több áttelepítési sorait.

Egyes eszközök előfordulhat, hogy továbbra is szeretné-e kezelni áthelyezési eszközök, például egy ideig, kimerítőnek újabb ésszerűsítés, a kezdeti áttelepítés után.

## <a name="selecting-the-first-workload"></a>Az első számítási feladat kiválasztása

Az első számítási feladatok végrehajtására key tesztelés és tanulás. Az első lehetőséget mutatja be, és állítsa össze a növekedési így.

### <a name="business-criteria"></a>Üzleti feltételek

A Felhőstratégia csapata üzleti egységet biztosítása átlátszóság üzleti tagja által támogatott számítási feladatok azonosítása. Lehetőleg úgy döntött, egyet, amelyben a csapat rendelkezik szerzett kockán és a felhőre való átállásra erős motiváció.

### <a name="technical-criteria"></a>Műszaki feltételeket

Válassza ki a számítási feladatok, amelyek minimális függőségekkel rendelkezik, és eszközök kis csoportja, áthelyezhetők. Javasoljuk, hogy egy meghatározott tesztelési elérési úttal rendelkező számítási feladatok megkönnyítése érdekében érvényesítési választani.

Az első számítási feladatok gyakran helyezik üzembe egy kísérleti környezet nélkül működési vagy cégirányítási kapacitása. Nagyon fontos válassza ki azt a munkaterhelést, amely nem működik együtt a védett adatok.

### <a name="qualitative-analysis"></a>Minőségi elemzése

A felhőre való áttérés és Felhőstratégia teams együttműködése hogyan e kis méretű számítási feladatok elemzése. Ez létrehoz egy hozhat létre és tesztelhet a minőségi elemzés során ellenőrzött lehetőséget. A kisebb népesség hoz létre a felmérési az érintett felhasználók minőségi részletes elemzés végrehajtásához, egy hét vagy még kevesebb lehetőséget. Közös minőségi elemzési tényező befolyásolja, lásd: a konkrét ésszerűsítés célhelyet a [ésszerűsítés 5 Rs](5-rs-of-rationalization.md).

### <a name="migration"></a>Migrálás

Folyamatos ésszerűsítés párhuzamosan a felhőre való áttérés csapat megkezdheti a kis munkaterhelés bontsa ki a következő kulcsterületeket learning áttelepítése:

- Erősítse meg a felhőszolgáltató platform szakismereteit.
- Határozza meg, a core szolgáltatások (és az Azure megfelelőségi) megfelelően a hosszú távú stratégiai szükséges.
- Jobban megismerheti, hogyan előfordulhat, hogy a műveletek később az átalakítás ki kell.
- Ismerje meg bármely járó üzleti kockázatot és az üzleti szempontból tűrés az említett kockázatok.
- Az üzleti szempontból kockázattűrése alapján cégirányítási létrehozni egy alapkonfigurációt, vagy a termék minimális működőképes (MVP).

## <a name="release-planning"></a>Tartalomkiadás tervezése

A felhőre való áttérés csapat végrehajtása a migrálás vagy az első számítási feladatok végrehajtását, közben a Felhőstratégia csapat megkezdheti a fennmaradó alkalmazások és számítási feladatok fontossági sorrend.

### <a name="power-of-ten"></a>Hatványa

A hagyományos megközelítés ésszerűsítés próbál az óceán forraljuk. Szerencsére a minden alkalmazás tervezése gyakran nem szükséges egy átalakulásunkhoz elindításához. Egy növekményes modellben tíz Power jó kiindulási pontot biztosít. Ebben a modellben a felhő stratégia csapat választja ki az első tíz alkalmazások telepíthetők át. Tíz számítási feladatok egyszerű és összetett számítási feladatok vegyesen kell tartalmaznia.

### <a name="building-the-first-backlogs"></a>Az első sorait készítése

A felhőre való áttérés és Felhőstratégia csapatok is együttműködhetnek a minőségi elemzése az első tíz számítási feladatokhoz. Ez az első rangsorolt áttelepítési várakozó fájlok számát és az első rangsorolt kiadási várakozó hoz létre. Ez a megközelítés lehetővé teszi a csapatok újrafuttathatja a megközelítést, és egy megfelelő folyamat minőségi elemzéshez létrehozásához elegendő időt biztosít.

### <a name="maturing-the-process"></a>A folyamat lejáró

Ha a két csapatok megegyeztünk a minőségi követelményeit, értékelés belül minden egyes ismétléskor feladat válhat. Értékelési feltételek caiq elérése általában szükség van a 2&ndash;3-kiadások.

Az értékelés az áttelepítés növekményes végrehajtási folyamatokba helyezik, miután a felhőre való áttérés csapata is iterálása értékelési és architektúra gyorsabban. Ezen a ponton a Felhőstratégia csapata is kiveszik, csökkentve a kiürítési időpontban. Ez lehetővé teszi a a Felhőstratégia csapat prioritás beállítása az alkalmazások, amelyek a rendszer nem még egy adott kiadás, ezáltal biztosítható piaci feltételek módosítása szoros ciklustól összpontosíthat.

A kiemelt alkalmazások nem minden áttelepítés készen áll. Alkalmazás-előkészítés valószínű, hogy a csapat minőségi mélyebb elemzésre hajt végre, és felderíti az üzleti eseményeket és függőségekkel foglalkozik, szeretne kérni a Hátralék újbóli rangsorolási módosításához. Néhány kiadásai előfordulhat, hogy csoportba a számítási feladatok kis számú. Mások csak tartalmazhat egyetlen számítási feladat.

A felhőre való áttérés csapat valószínű oka az, amely egy teljes számítási feladatok migrálása nem eredményeznek az ismétlések futtatásához. Minél kisebb a számítási feladatok, és kevesebb függőségeket, annál nagyobb eséllyel egy számítási feladatot egyetlen sprint vagy iteráció megfelelően. Ebből kifolyólag javasoljuk, hogy az első néhány alkalmazás kiadási várakozó fájlok számát a kis méretű lehet, és néhány külső függőséget tartalmaz.

## <a name="end-state"></a>Teljes állapot

Az idő múlásával a felhőre való áttérés és a Felhőstratégia csapatok kombinációja a készlet teljes ésszerűsítése fog befejeződni. Azonban ez növekményes lehetővé teszi a csapatok folyamatosan gyorsabban beolvasni a ésszerűsítés folyamat. Lehetővé teszi az eddig is számtalan előnyét képzés résztvevői hasznos képességekkel üzleti eredményeit a Átalakulásunkhoz hamarabb, anélkül is nagy, előzetes költségek elemzése annak érdekében.

Bizonyos esetekben a pénzügyi minta túl szoros a döntéshozáshoz való működésre, további ésszerűsítés nélkül lehet. Ezekben az esetekben a hagyományos megközelítés ésszerűsítés lehet szükség.

## <a name="next-steps"></a>További lépések

Egy ésszerűsítés erőfeszítés kimenete egy rangsorolt mappájában várakozó fájlok számát az összes olyan eszköz, amely a kiválasztott átalakítási érinti. A várakozó fájlok számát a cloud Services költségszámítási modellek alapját egyikükön készen áll.

> [!div class="nextstepaction"]
> [A digitális hagyatéki a költségmodellek igazítása](calculate.md)