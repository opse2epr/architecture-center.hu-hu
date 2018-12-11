---
title: Felhőre vonatkozó pénzügyi minta létrehozása
titleSuffix: Enterprise Cloud Adoption
description: Felhőre vonatkozó pénzügyi minta létrehozása
author: BrianBlanchard
ms.date: 12/10/2018
ms.openlocfilehash: 9a4d6b7c30d3dc0509a1b26fa8d247ae39ab87ca
ms.sourcegitcommit: e7f8676bbffe500fc4d6deb603b7c0b7ba1884a6
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/10/2018
ms.locfileid: "53179455"
---
# <a name="enterprise-cloud-adoption-how-to-create-a-financial-model-for-cloud-transformation"></a>Enterprise Cloud Adoption: Felhőre vonatkozó pénzügyi minta létrehozása

Egy pénzügyi modellt, amely pontosan tükrözi a teljes üzleti értéket bármely felhőbeli átalakítás létrehozása bonyolult feladatnak bizonyulhat. Pénzügyi modelleket és üzleti a indoklások általában a következő egy szervezet különböző. Ez a cikk néhány képletek és a pontokat, amelyek gyakran kimaradt a pénzügyi minta létrehozásakor néhány dolgot végre hoz létre.

## <a name="return-on-investment-roi"></a>A megtérülési ráta (ROI)

Az elvárt megtérülési rátát (ROI) legtöbbször fontos kritérium, a C-Suite vagy tábla. Megtérülés összehasonlítása módjai, korlátozott, befektetés erőforrást szolgál. A megtérülési ráta képlete viszonylag egyszerű. Előfordulhat, hogy a kell létrehoznia a képletet minden egyes bemenet részletei nem egyszerű. Alapvetően ROI érték a előállított egy kezdeti befektetések megtérülési rátát. Általában azt jelölt százalékaként:

![Lépjen vissza a befektetési (ROI) eredménye (nyereség, befektetés – költség, befektetés), befektetés költség /](../_images/formula-roi.png)

<!-- markdownlint-disable MD036 -->
*Megtérülés = (szerezhet, befektetés &minus; kezdeti, befektetés) / kezdeti, befektetés*
<!-- markdownlint-enable MD036 -->

A következő szakaszokban végigvezetjük a kezdeti befektetések, és a nyereség, befektetés (eredmény) kiszámításához szükséges adatok.

## <a name="calculating-initial-investment"></a>Kezdeti, befektetés kiszámítása

Kezdeti, befektetés a munkavállalónkénti (CapEx) és kiadások üzemi (OpEx) szükséges átalakítás végrehajtásához. A besorolás, a költségek számlázási modellt és a pénzügyi igazgató szabályozó függően változhat. Azonban ez a kategória lenne közé tartozik többek között: Professzionális szolgáltatások átalakításához, az átalakítás során kizárólag az átalakítást, az átalakítás során a cloud services költsége és potenciálisan a fizetett alkalmazottak költsége során használt szoftverlicencekről.

Adja hozzá ezeket a díjakat a kezdeti befektetések becsült létrehozása.

## <a name="calculating-the-gain-from-investment"></a>A nyereség, befektetés kiszámítása

Nyereség, befektetés milyen gyakran van szükség a második képlet számításhoz, amely jellemző az üzleti eredmények és a hozzá kapcsolódó technikai. Bevételek nem, egyszerűen kiszámítása a költségek csökkentését.

Bevételek kiszámításához, két változót szükség:

![Befektetés nyereségét egyenlő bevétel eltérések + költség eltérések](../_images/formula-gain-from-investment.png)

<!-- markdownlint-disable MD036 -->
*Szerezhet, befektetés = bevétel eltérések + költség eltérések*
<!-- markdownlint-enable MD036 -->

Az egyes alább ismertetjük.

## <a name="revenue-delta"></a>Bevétel Változásreplikációjához

Bevétel változásreplikációjához a vállalkozásával együtt kell előrejelzett ellenőriztünk. Miután az üzleti résztvevőkkel elfogadja a bevétel hatás szerint is megjelenítheti, elnyerve pozíciója javítására használható.

## <a name="cost-deltas"></a>A Cost eltérések

Költség eltérések a növekedést vagy csökkenést, amely az átalakítás következtében fognak érkezni mennyisége. Számos független változók, amelyek hatással lehetnek a költségek eltérései nagyobbak. Bevételek beruházna csökkentésének, a költségek elkerülése, a üzemeltetési költségek csökkentése és a amortizációs árképzésre fordított például nehéz költségek nagymértékben alapulnak. Példák a következő szakaszok költség eltérések kell figyelembe venni.

### <a name="depreciation-reductions-or-acceleration"></a>Amortizációs vagy a gyorsítás

Amortizációs útmutatást beszéljen a pénzügyi vezetőnek Követnie a, vagy pénzügyi csapata. A következő hivatott amortizációs témakörről általános referenciaként szolgálnak.

Tőke fektetett a beszerzési egy eszköz, amikor, befektetés lehet használni a pénzügyi vagy adó célra használja, a folyamatban lévő előnyöket előállításához várt gondolhatunk, az eszköz. Egyes vállalatok amortizációs látja, mint egy pozitív adó használja ki. Mások láthassák költségként véglegesítve, folyamatban lévő egyéb ismétlődő kiadások, az éves informatikai költségvetés teljesítménykapacitást hasonló.

Beszéljen a pénzügyi Office-Ha amortizációs eltávolítási lehetőség, és azt szeretne közreműködői tevékenységet végezne pozitív eltérések cost megtekintéséhez.

### <a name="physical-asset-recovery"></a>Fizikai eszközintelligencia-recovery

Bizonyos esetekben a kivont eszközök árusíthatók bevétel forrásaként. Gyakran előfordul a bevétel lumped van, a költségek csökkentéséhez az egyszerűség kedvéért. Azonban valóban az árbevétel növelését, és mint ilyen adózott. Beszéljen a pénzügyi Office megvalósíthatóságának ezt a beállítást, és a fiók számára az eredményül kapott bevétel annak megértéséhez.

### <a name="operational-cost-reductions"></a>Üzemeltetési költségek csökkentése

A vállalkozás működtetéséhez szükséges ismétlődő kiadásokat gyakran nevezzük működési költségeket. OpEx egy nagyon tág kategóriába. Szoftverlicencelés, üzemeltetési költségek, számlák Electric vállalattal, ingatlan bérlését, hűtési költségei, operations, berendezések bérlését, cserealkatrészekre, karbantartási-szerződéseket, javítása szükséges ideiglenes alkalmazottak kellene benne a legtöbb számlázási modellek Üzleti folytonosság és vész-helyreállítási BC/szolgáltatások, és számos egyéb kiadások, amelyek nem igényelnek beruházna jóváhagyásokat.

Ebben a kategóriában a legnagyobb eredmény terület egyikének akkor, ha figyelembe véve a egy működési Átalakulásunkhoz is. Így ezt a listát az alábbi befektettek idő ritkán kihasználatlan. A CIO kérdéseket tehet fel, és annak biztosítása érdekében üzemeltetési költségek minden pénzügyi csapata számolják.

### <a name="cost-avoidance"></a>Költségek elkerülése

Ha az üzemeltetési költségek (OpEx) várt, de még nincs egy jóváhagyott költségvetés, előfordulhat, hogy nem fér el a költségek csökkentését kategória. Például ha a VMWare és a Microsoft licencek főkulcsalapot hoz létre, és a következő évre fizetős kell, nem teljesen minősített költségek még. Azokat a csökkentése várható költségeket kezeli mint különbözeti költségszámításokhoz az üzemeltetési költségeket. Azonban azok kell lennie emlegetik "költségek elkerülését," egyeztetési és a költségvetés jóváhagyási befejezéséig.

### <a name="soft-cost-reductions"></a>Helyreállítható költségek csökkentése

Az egyes vállalatok például az üzemeltetés bonyolult vagy a teljes munkaidejű alkalmazottak megfelelően működjenek az egy kínai adatközpont csökkentését helyreállítható költségek sikerült is tartalmaz. Azonban, beleértve a helyreállítható költségeit lehet nevet adni aliasnévként. Egy nem dokumentált feltételezve, hogy a költségek csökkentése fog feleltetni a képzés résztvevői hasznos képességekkel költségmegtakarítást helyreállítható költségekkel együtt szúr be. Technológiai projektek ritkán helyreállítható tényleges költség-helyreállítási eredményez.

### <a name="headcount-reductions"></a>Személyzet csökkentése

Időmegtakarítás a személyzet gyakran megtalálhatók helyreállítható költségek csökkentése mellett. E tekintve informatikai fizetés vagy volt szükség létszámnövelésre tényleges csökkentése leképezni, ha azt sikerült kiszámítani külön, a létszám csökkentését.

Mindemellett a szükséges ismereteket szükséges a helyszíni általában leképezése egy hasonló (vagy magasabb szintű) a felhőben szükséges ismeretek halmaza. Ez azt jelenti, személyek általában nem get meghatározott ki felhőbe történő migrálás után.

Kivétel akkor, ha a működési kapacitását egy harmadik féltől származó vagy egy felügyelt szolgáltatásokat kínáló (MSP) által biztosított. Egy harmadik fél által felügyelt IT-rendszereit, ha a sikerült helyébe megfelelően működjenek a költségek egy natív felhőalapú megoldást, vagy a natív MSP. Egy felhőbeli natív MSP várhatóan hatékonyan és potenciálisan alacsonyabb költségekkel. Ez a helyzet, ha az üzemeltetési költségek csökkentése a rögzített költségszámításokhoz az tartozik.

### <a name="capital-expense-reductions-or-avoidance"></a>Beruházna vagy a elkerülése

A beruházási költségek (CapEx) némileg eltérőek, hogy működési kiadásait. Általában ez a kategória határozzák meg az adatfrissítési ciklusok vagy az Adatközpont bővítése. Például egy adatközpont bővítése lenne egy új nagy teljesítményű fürt futtatásához egy Big Data megoldás, sem az adattárházra, és általában lenne illik CapEx kategória. Gyakori olyan alapszintű frissítési ciklusok. Egyes vállalatok frissítési ciklusok merev hardver, jelentése eszközök vannak elavult, és rendszeres időközönként (általában minden 3, 5 vagy 8 év) helyettesíti. Ezeket a ciklusok gyakran eszköz bérleti ciklusok egybe vagy berendezések gyűjteményszintű előrejelzett. Ha egy frissítés során rákeres, informatikai Rajzolás CapEx új berendezések beszerezni.

Ha egy frissítés során, és a tervezett, a felhő átalakítási segíti, hogy a költség megszüntetéséhez. Egy frissítési ciklus tervezett, de még nem jóváhagyott, ha a felhő átalakítási létrehozhat egy beruházási költségek elkerülését. A költségek különbözeti mindkét forgatókönyvet kell hozzáadni.
