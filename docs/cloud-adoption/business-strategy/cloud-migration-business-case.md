---
title: 'CAF: Egy felhőbeli migrálás üzleti eset létrehozása'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
description: Szempontok a felhőmigrálás üzleti indoklásának létrehozásához.
author: BrianBlanchard
ms.date: 12/10/2018
ms.topic: guide
ms.openlocfilehash: 21c2b877a0f329711027f020fd0047479e8c7bfb
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55898306"
---
# <a name="build-a-business-justification-for-cloud-migration"></a>Migrálás a felhőbe üzleti indoklásának létrehozása

A migrálás a felhőbe korai az elvárt megtérülési rátát (ROI) hozhat létre a felhő átalakítás folyamatában. Azonban a materiális egyértelmű üzleti indoklásának fejlesztésével, kapcsolódó költségeket, és adja vissza lehet összetett folyamat. Ez a cikk segítséget nyújt a úgy gondolja, hogy milyen adatokkal kapcsolatos létrehozásához szükséges egy pénzügyi modellt, amely igazodik a felhőbe migrálás eredményekkel. Először is hozzunk kihajtsuk kapcsolatos típusú felhőbeli migrálás, néhány tévhitet, így a szervezet néhány gyakori hibák elkerülése érdekében.

## <a name="dispelling-cloud-migration-myths"></a>Felhőbeli migrálás tévhitet dispelling

**Miközben: Olcsóbb a felhőben.** Közös határvonalak, amely egy adatközpontban, a felhőben működő mindig olcsóbb a helyszíni. Bár ez lehet igaz, akkor sem az abszolút. Bizonyos esetekben felhőalapú üzemeltetési költségek is valójában magasabb lehet. Gyakran ez gyenge költség cégirányítási, illesztését alapjául szolgáló, a párhuzamos folyamatok, a rendszer szokatlan konfigurációk esetén az okozza, vagy személyzeti költségek nőtt. Szerencsére ezeket a problémákat számos mérsékelhető a korai ROI létrehozásához. Található útmutatást [létrehozását az üzleti indoklás](#building-the-business-justification) , és elkerülheti ezeket ők segítséget. Az egyéb, az itt felsorolt tévhitet dispelling is segít.

**Miközben: Minden kell, végezze a felhőbe.** Sőt számos üzleti illesztőprogramokat, amelyek meghiúsíthatják hibrid megoldás kiválasztását. Az üzleti modell többszöri célszerű előre megtervezni egy első kerek a mennyiségi elemzés végrehajtásához, leírtak szerint a [digitális hagyatéki cikkek](../digital-estate/5-rs-of-rationalization.md). Az egyes mennyiségi illesztőprogramok ésszerűsítés részt kapcsolatos kiegészítő tudnivalókat lásd: [ésszerűsítés, az 5-Rs](../digital-estate/5-rs-of-rationalization.md). Mindkét módszerrel használatával egyszerűen kapott leltáradatok és a egy rövid mennyiségi elemzési számítási feladatok vagy az alkalmazásokat, amelyek a felhőben magasabb költségeket eredményezhet. Ezek a módszerek is azonosíthatóak, függőségek vagy forgalmi mintázatait, amely egy hibrid megoldás lenne szükség.

**Miközben: A helyszíni környezet tükrözés segítséget kérek a pénzt takaríthat meg a felhőben.** Során digitális hagyatéki tervezi, akkor sem átláthatóságról kívüli ügyfelek 50 %-a kiépített környezet biztosította a fel nem használt kapacitás észleléséhez. Ha az eszközök kiosztása a felhőben is egyezik az aktuális kiépítés költségmegtakarítást nehezen hajlandónak lesz. Vegye figyelembe, hogy az üzembe helyezett eszközök használati mintákat, nem kialakítási minták igazodva méretének csökkentését.

**Miközben: A meghajtó felhőmigrálás üzleti helyzetei Server költségeit.** Egyes esetekben ez is igaz. Egyes vállalatok számára fontos kiszolgálókhoz kapcsolódó folyamatos beruházási költségek csökkentése érdekében. Azonban ez több tényezőtől függ. Egy öt nyolc éves hardverek frissítési ciklusát rendelkező vállalatok általában nem kategorizálhatók tekintse meg a gyors értéket ad vissza a felhőbe való migrálást. Szabványosított vagy kényszerített adatfrissítési ciklusok rendelkező vállalatok gyorsan elérjék a megtérülési pont. Egyéb kiadások mindkét esetben lehet, adja meg az áttelepítés pénzügyi eseményindítói. Néhány példa a költségeket, gyakran kihagyott a költségek csak kiszolgáló, vagy csak a virtuális gép nézetét állapotba helyezésekor a következők:

- A virtualizálás, a kiszolgálók és a közbenső szoftverek díjait széles körű lehet. A felhőszolgáltatók kiküszöbölése néhány ezeket a díjakat. Két példa virtualizálási költségek csökkentése a felhőszolgáltatók a [Azure Hybrid Benefits](https://azure.microsoft.com/pricing/hybrid-benefit/#services) és [foglalások](https://azure.microsoft.com/reservations/) programok.
- Üzleti veszteségek leállások miatt gyorsan haladhatja meg a hardver vagy szoftver költségeket. Ha az aktuális adatközponti instabillá válhat, ahol az üzleti lehetőség költségek és a tényleges üzleti költségek tekintetében leállások hatásának számszerűsítése.
- A környezeti költségek hatásos is. Az átlagos American család a kezdőlapja a legnagyobb befektetési és a legmagasabb díj költségvetés. Ugyanez igaz gyakran adatközpontok számára. Ingatlan, létesítmények és segédprogram költségek egy a helyszíni üzemeltetési költség valós részét képviseli. Adatközpontok kivezettük, ha ezekben a létesítményekben újrafelhasználásra kerülnek az üzlet szerint, vagy potenciálisan üzleti sikerült mentesülhet a költségek teljes egészében.

**Miközben: Üzemeltetési költségek (OpEx) jobb, mint beruházási költségek (CapEx).** Csatornára. További információ a [pénzügyi eredményekkel](business-outcomes/fiscal-outcomes.md) OpEx a cikkben egy jó dolog is lehet. Azonban számos iparágban hasznosak, mint a negatív OpEx számára látható. Az alábbiakban néhány példát, amely aktiválja szorosabban együttműködnek az OpEx beszélgetés vonatkozó számlázási és az üzleti egységek:

- Amikor a vállalkozás üzleti értékelés illesztőprogramként tőke eszközök látja, CapEx árképzésre fordított lehet egy negatív eredmény. Univerzális standard, miközben a róluk szóló véleményeket leggyakrabban akkor fordul elő, kiskereskedelmi, gyártási és konstrukció iparágban.
- OpEx növekedésével is látható, a negatív eredmény vállalkozásokban egy privát saját cég vagy pozicionálási, befektetés álljon a tulajdonosa.
- Ha az üzleti összpontosít az erősen értékesítési margók javítása, vagy csökkentve a költségeket a termékek eladott (COGS), a OpEx lehet egy negatív serkenti az eredményt.

OpEx értéke nem mindig egy hibás lépés. Vállalkozások valószínűleg több OpEx látja, mint a kedvezőbb CapEx-nál. Például ez a módszer jól fogadhatók vállalatok, amelyek próbál javítása pénzforgalmát, tőkeberuházás csökkentése, vagy csökkentse a eszköz üzemek szerint.

Mielőtt egy üzleti indoklás, amely a konvertálást CapEx költségekké alakítását összpontosít, hogy melyik jobb üzleti. Számlázási és beszerzési gyakran segíthet legjobb igazítása pénzügyi célok az üzenetet.

**Miközben: A felhőbe való áthelyezés olyan, mintha egy kapcsoló tükrözés.** Áttelepítés olyan manuálisan az erős technikai átalakítást. Üzleti indoklásának fejlesztésével, különösen azok a indoklások, idő-és nagybetűket, amelyek megnőhet az objektumok áttelepítéséhez szükséges idő a következő szempontokat vegye figyelembe:

- Sávszélesség-korlátozások: Az aktuális adatközpontban és a felhőszolgáltató közötti sávszélesség vonják ütemtervek áttelepítés során.
- Üzleti tesztelési ütemterveket: Az üzleti tesztelési alkalmazások tanúsítása készültségi és a teljesítmény időigényes lehet. Igazítás a kiemelt felhasználók, és tesztelési folyamatokat, kritikus fontosságú.
- Migrálás végrehajtása ütemterveket: A hátralévő időt és az áttelepítés végrehajtásához szükséges emberi beavatkozást növelheti a költségeket és a késleltetés ütemterveket. Alkalmazottak lefoglalása vagy partnerek szerződő is késleltetheti-e a folyamat, és a csomag elszámolni.

Technikai és kulturális akadályait lelassíthatja a felhőre való áttérés. Ha fontos szempont a az üzleti indoklás idő, a legjobb megoldás a megfelelő megtervezése. Nincsenek két javasolt módszer, amely segíthet csökkenteni az ütemterv kockázatok tervezése során.

- Első lépésként fektet az időt és energiát a műszaki bevezetési megkötések ismertetése. Bár lehet, hogy gyorsan nyomás magas, fontos, valósághű végrehajtási idősor-fiókjához.
- A második kulturális környezet vagy személyek akadályait merülnek fel, ha rendelkeznek, mint a technikai korlátok miatt több súlyos hatással van. Felhőre való áttérés hoz létre a változás, amely a kívánt átalakítási műveletet hoz létre. Sajnos emberek néha lehetőségének változás, és előfordulhat, hogy további támogatási csomaggal rendelkező igazítása kell. Módosíthatja, és bevonhatja őket korai kommunikálniuk vannak, a csapat kulcs személyek azonosítása.

Maximális készültségi és ütemterv kockázatok mérséklése érdekében előkészítése végrehajtó érintettek egymáshoz igazításával nyitóbeszélgetést üzleti értéket és az üzleti eredményeket. Ezeket a módosításokat az átalakítás mellékelt ismertetése érintettek segítségével. Az elejétől valósághű elvárások törölje és állítsa is be lehet. Személyek vagy technológia lassú a folyamatot, amikor vezetői támogatás besorolás könnyebb lesz.

## <a name="building-the-business-justification"></a>Az üzleti indoklás készítése

A következő folyamat megközelítést definiál a fejlődő üzleti indoklásának a migrálás a felhőbe. Ez a tartalom olvasása közben a számítások vagy pénzügyi feltételek igényel további magyarázatot, olvassa el a [pénzügyi modelleket](financial-models.md) a további tisztázása.

A legmagasabb szintű üzleti indoklás a képlet használata egyszerű. Azonban a szükséges adatokkal való feltöltéséhez a képlet finom Adatpontok igazítása nehézkes lehet. A legmagasabb szintű üzleti indoklásának fordított beruházások (ROI) a javasolt technikai változáskezelési társított összpontosít. A megtérülési ráta általános képlete:

Megtérülés = (szerezhet, befektetés &minus; kezdeti, befektetés) / kezdeti, befektetés

Ezt a képletet kicsomagolása áttelepítési-specifikus nézet létrehozása a képletek meghajtón minden a bemeneti változók a egyenlőségi jobb oldalán. Ez a cikk fennmaradó részéből olyan szempontokat kell figyelembe vennie.

![Megtérülés egyenlő (nyereség, befektetés – költség, befektetés), befektetés költség /](../_images/formula-roi.png)

## <a name="migration-specific-initial-investment"></a>Áttelepítési-specifikus kezdeti befektetések

- A felhőszolgáltatók, például az Azure megbecsülni a felhőbefektetések számító kínálnak. Például ilyen egy Számológép a [Azure díjkalkulátor](https://azure.microsoft.com/en-in/pricing/).
- Egyes felhőszolgáltatók költség különbözeti számító is támogatják. A különbözeti költségkalkulátor például a [Azure teljes költség a tulajdonjog (TCO) Számológép](https://azure.com/tco).
- A kifinomultabb költség struktúrák, fontolja meg egy [digitális hagyatéki tervezési](../digital-estate/overview.md) gyakorolják.
- Becslés az áttelepítés.
- Minden olyan várható képzési lehetőségek kiszámítása. [A Microsoft Learn](https://docs.microsoft.com/learn/) képes e költségek csökkentése érdekében.
- Egyes vállalatok fektetett meglévő munkatársai által az idő tagok előfordulhat, hogy kell szerepelnie a kezdeti költségek. Útmutatásért tekintse meg a pénzügyi office.
- Beszélje meg minden olyan további költségek és a terhet költségeket a pénzügyi Office-ellenőrzés céljából.

## <a name="migration-specific-revenue-deltas"></a>Áttelepítési-specifikus bevétel eltérések

Ez a helyzet gyakran van kihagyott, egy migrálási üzleti indoklás létrehozásakor. Egyes területeken a felhőben csökkentheti költségeit. Azonban a végső bármely átalakítási célja, hogy jobb eredmények felfüggesztési idő függvényében. Érdemes tudni, hogy hosszú távú bevétel fejlesztései alsóbb rétegbeli következményeinek. Milyen új technológiák az áttelepítés, amely jelenleg nem javítható után lesz elérhető üzleti? Milyen projektek vagy üzleti célokhoz függőségek a régi technológiák által blokkolt? Milyen programok várakozó, magas cap-ex technológiai költségeiket függőben?

A felhő zárolása lehetőségek figyelembe vételével, munkahelyi alapján számítja ki, a bevétel, amely növeli az üzleti ezeket a lehetőségeket sikerült származnak.

## <a name="migration-specific-cost-deltas"></a>Áttelepítési-külön eltérések

Módosításokat a javasolt áttelepítési származnak, a költségek kiszámításához. Lásd: [pénzügyi modelleket](financial-models.md) költség eltérések a különböző típusú részleteit. A felhőszolgáltatók milyen gyakran biztosítja a különbözeti költségszámításokhoz. A különbözeti költségkalkulátor például a [Azure teljes költség a tulajdonjog (TCO) Számológép](https://azure.com/tco).

További példák a, amely a felhőbe történő Migrálás csökkentheti költségeit:

- Data Center felmondása vagy csökkentése (környezeti költségek)
- A felhasznált energia (környezeti költségek) csökkentése
- Állvány lezárást (fizikai eszköz helyreállítási)
- Hardver frissítés (költségek elkerülését) megakadályozása
- Kerülje a szoftver megújítást (működési költségek csökkentéséhez vagy költségek elkerülését)
- Szállítói összevonás (működési költségek csökkentéséhez és a lehetséges szoftveres költségek csökkentéséhez)

## <a name="when-roi-results-are-surprising"></a>Amikor ROI eredményeket is meglepő

Ha a megtérülési RÁTÁRA, a felhőbe történő migrálás nem az elvárásoknak megfelelően, a közös tévhitet, ez a cikk elején felsorolt gondolnia hasznosak lehetnek.

Fontos azonban megérteni, hogy a költség megtakarítás serkenti az eredményt értéke nem mindig lehetséges. Nincsenek további megfelelően működjenek a felhőben, mint a helyszíni alkalmazásokat, amelyek a költségek. Ezek az alkalmazások is jelentősen döntés az elemzés eredménye.

A megtérülési RÁTÁRA 20 % alatt van, vegye figyelembe a [digitális hagyatéki tervezési](../digital-estate/overview.md) a gyakorlatban a figyelmet adott [ésszerűsítés](../digital-estate/rationalize.md). A mennyiségi elemzés, során a számítási feladatok, amelyek az eredmények tevékenységdiagramon található mindegyik alkalmazás felülvizsgálat végrehajtása. Elképzelhető, hogy távolítsa el azokat a munkaterheléseket a terv célszerű előre megtervezni. Használati adatok érhető el, ha fontolja meg a használatot a virtuális gépek méretének csökkentését.

Ha továbbra is a helytelen igazítású a megtérülési RÁTÁRA, segítségért a Microsoft értékesítési képviselőjével vagy [tapasztalt partnerek vesznek](https://azure.microsoft.com/en-us/migration/partners/).

## <a name="next-steps"></a>További lépések

> [!div class="nextstepaction"]
> [Felhőre vonatkozó pénzügyi minta létrehozása](./financial-models.md)