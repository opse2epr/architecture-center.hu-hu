---
layout: LandingPage
ms.topic: landing-page
ms.openlocfilehash: 80cb7fde0694257a5c413b702505e27f18aed8d3
ms.sourcegitcommit: d702b4d27e96e7a5a248dc4f2f0e25cf6e82c134
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/23/2018
---
# <a name="azure-application-architecture-guide"></a>Útmutató az Azure alkalmazásarchitektúrájához

Ez az útmutató egy strukturált megközelítést mutat be az Azure-on futó skálázható, rugalmas, magas rendelkezésre állású alkalmazások tervezéséhez. Az útmutatót az ügyfélesetekből származó tapasztalatok és az ajánlott eljárások alapján állítottunk össze.

<br/>

<img src="./images/guide-steps.svg" style="max-width:800px;"/>

## <a name="introduction"></a>Bevezetés

A felhő megváltoztatja az alkalmazások tervezésének módját. Az egybefüggő kódtömbök helyett az alkalmazások kisebb, decentralizált szolgáltatásokra vannak bontva. Ezek a szolgáltatások API-kkal, vagy aszinkron üzenetküldéssel és eseménykezeléssel kommunikálnak egymással. Az alkalmazások horizontálisan skálázhatók és igény szerint új példányokkal bővíthetők. 

Ezek a trendek új kihívásokat támasztanak felénk. Az alkalmazás elosztott állapotú. A műveletek párhuzamosan és aszinkron módon zajlanak. Hibák esetén az egész rendszernek rugalmasnak kell lennie. Az üzemelő példányoknak automatizáltnak és kiszámíthatónak kell lenniük. A monitorozás és a telemetria kritikus fontosságú a rendszer működésébe való betekintés szempontjából. Az Útmutató az Azure alkalmazásarchitektúrájához célja, hogy segítsen Önnek eligazodni ezen változások között. 

<table>
<thead>
    <tr><th>Hagyományos helyszíni</th><th>Modern felhő</th></tr>
</thead>
<tbody>
<tr><td>Monolitikus, központosított<br/>
Kiszámítható skálázhatóságra tervezve<br/>
Relációs adatbázis<br/>
Erős konzisztencia<br/>
Soros és szinkronizált feldolgozás<br/>
A hibák elkerülését segítő kialakítás (MTBF)<br/>
Időnkénti nagy frissítések<br/>
Kézi felügyelet<br/>
Hópehely-kiszolgálók</td>
<td>
Szétbontott, decentralizált<br/>
Rugalmas skálázásra tervezve<br/>
Polyglot-adatmegőrzés (többféle tárolási technológia vegyítése)<br/>
Végleges konzisztencia<br/>
Párhuzamos és aszinkron feldolgozás<br/>
Hibákra tervezve (MTTR)<br/>
Gyakori kisebb frissítések<br/>
Automatikus önfelügyelet<br/>
Nem módosítható infrastruktúra<br/>
</td>
</tbody>
</table>

A jelen útmutató alkalmazásmérnökök, fejlesztők és üzemeltetési csapatok számára készült. Nem az egyes Azure-szolgáltatások használati útmutatója. Az útmutató elolvasásával megismerheti az alkalmazások Azure-felhőplatformon való létrehozásakor érvényes architektúramintákat és ajánlott eljárásokat. Az [útmutató e-könyv verzióját][ebook] is letöltheti.

## <a name="how-this-guide-is-structured"></a>Az útmutató felépítése

Az Útmutató az Azure alkalmazásarchitektúrájához több lépésből áll az architektúrától és tervezéstől az implementálásig. Minden lépés mellé támogató útmutatást is nyújtunk, amely segítséget nyújt az alkalmazásarchitektúra megtervezésében.

### <a name="architecture-styles"></a>Architektúrastílusok

A legelső döntés a legfontosabb. Milyen típusú architektúrát szeretnénk használni? Lehet mikroszolgáltatási architektúra, hagyományosabb N rétegű alkalmazás, vagy big data-megoldás is. Számos különféle architektúrastílust határoztunk meg. Mindegyiknek megvannak a maga előnyei és kihívásai.

További információ:

- [Architektúrastílusok][arch-styles]
- [Azure-referenciaarchitektúrák][ref-archs]

### <a name="technology-choices"></a>Technológiai lehetőségek

Két technológiai döntést már a legelején meg kell hozni, mivel ezek a teljes architektúrára hatással vannak. Ez a két döntés pedig nem más, mint a számítási szolgáltatás és az adattárak kiválasztása. A *számítás* azon számítási erőforrások üzemeltetési modelljére utal, amelyeken az alkalmazások futnak. Az *adattárak* közé tartoznak az adatbázisok, valamint az üzenetsorokhoz, a gyorsítótárakhoz, a naplókhoz és az alkalmazás által megőrizni kívánt bármely egyéb adathoz használt tárolók is. 

További információ:

- [Számítási szolgáltatás kiválasztása](./technology-choices/compute-overview.md)
- [Adattár kiválasztása](./technology-choices/data-store-overview.md)

### <a name="design-principles"></a>Tervezési alapelvek

Meghatároztunk tíz magas szintű tervezési alapelvet, amelyeket követve skálázhatóbbá, rugalmasabbá és felügyelhetőbbé teheti alkalmazását. Ezek a tervezési alapelvek minden architektúrastílusra érvényesek. A tervezés folyamata során a következő tíz magas szintű tervezési alapelvet kell betartani. Ezután vegye figyelembe az ajánlott eljárásokat az architektúra különféle területeivel kapcsolatban, többek között például az automatikus skálázással, a gyorsítótárazással, az adatparticionálással és az API-tervezéssel.

További információ:

- [Tervezési alapelvek Azure-alkalmazásokhoz][design-principles]
- [Ajánlott eljárások felhőalkalmazások készítésekor][best-practices]

### <a name="quality-pillars"></a>A minőség alappillérei

Egy sikeres felhőalkalmazás a szoftverminőség következő öt alappillérére koncentrál: skálázhatóság, rendelkezésre állás, rugalmasság, felügyelet és biztonság. Tervezés-áttekintési ellenőrzőlistáink segítségével ellenőrizheti, hogy az architektúra megfelel-e ezeknek a minőségi alappilléreknek.

További információ:

- [A szoftverminőség alappillérei][pillars]
- [Tervezésáttekintési ellenőrzőlisták][checklists] 

### <a name="cloud-design-patterns"></a>Tervezési minták felhőkhöz

A tervezési minták általános megoldást nyújtanak a gyakori szoftvertervezési problémákra. Összeállítottunk néhány tervezési mintát, amelyek különösen hasznosak elosztott felhőalkalmazások tervezésekor.

További információ:

- [Felhőalkalmazások tervezési mintáinak katalógusa](../patterns/index.md)


[arch-styles]: ./architecture-styles/index.md
[best-practices]: ../best-practices/index.md
[checklists]: ../checklist/index.md
[compute-options]: ./technology-choices/compute-comparison.md
[design-principles]: ./design-principles/index.md
[ebook]: https://azure.microsoft.com/campaigns/cloud-application-architecture-guide/
[patterns]: ../patterns/index.md?toc=/azure/architecture/guide/toc.json
[pillars]: ./pillars.md
[ref-archs]: ../reference-architectures/index.md
[storage-options]: ./technology-choices/data-store-comparison.md
[technology-choices]: ./technology-choices/index.md

