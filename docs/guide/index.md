---
layout: LandingPage
ms.topic: landing-page
ms.openlocfilehash: 530844a0d3b1256cec807e7bad509a40dca304f6
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/06/2018
---
# <a name="azure-application-architecture-guide"></a>Útmutató az Azure alkalmazásarchitektúrájához

Ez az útmutató egy strukturált megközelítést mutat be az Azure-on futó skálázható, rugalmas, magas rendelkezésre állású alkalmazások tervezéséhez. Az útmutatót az ügyfélesetekből származó tapasztalatok és az ajánlott eljárások alapján állítottunk össze.

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

**[Architektúrastílusok][arch-styles]**. A legelső döntés a legfontosabb. Milyen típusú architektúrát szeretnénk használni? Lehet mikroszolgáltatási architektúra, hagyományosabb N rétegű alkalmazás, vagy big data-megoldás is. Hét különböző architektúrastílust határoztunk meg. Mindegyiknek megvannak a maga előnyei és kihívásai.

> &#10148; Az [Azure-referenciaarchitektúrák][ref-archs] bemutatják az Azure-beli ajánlott telepítéseket, valamint a skálázhatóságra, a rendelkezésre állásra, a kezelhetőségre és a biztonságra vonatkozó megfontolandó szempontokat. A legtöbb esetben üzembe helyezhető Resource Manager-sablonokat is tartalmaznak.

**[Technológiai lehetőségek][technology-choices]**. Két technológiai döntést már a legelején meg kell hozni, mivel ezek a teljes architektúrára hatással vannak. Ez a két döntés pedig nem más, mint a számítási és az adattárolási technológiák kiválasztása. A *számítás* kifejezés azon számítási erőforrások futtatási modelljére utal, amelyeken az alkalmazás fut. A tárolási technológiák közé tartoznak az adatbázisok, valamint az üzenetsorokhoz, a gyorsítótárakhoz, az IoT-adatokhoz, a strukturálatlan naplóadatokhoz, és az alkalmazás által megőrizni kívánt bármely egyéb adathoz használt adattárak is. 

> &#10148; A [Számítási lehetőségek][compute-options] és a [Tárolási lehetőségek][storage-options] szakasz részletes összehasonlításokat tartalmaz a számítási és tárolási szolgáltatások kiválasztásához.

**[Tervezési alapelvek][design-principles]**. A tervezés folyamata során a következő tíz magas szintű tervezési alapelvet kell betartani. 

> &#10148; Az [Ajánlott eljárások][best-practices] cikkek specifikus útmutatást nyújtanak többek között olyan területeken, mint az automatikus skálázás, a gyorsítótárazás, az adatparticionálás vagy az API-tervezés.   

**[Pillérek][pillars]**. Egy sikeres felhőalkalmazás a szoftverminőség következő öt alappillérére koncentrál: skálázhatóság, rendelkezésre állás, rugalmasság, felügyelet és biztonság. 

> &#10148; A [Tervezés-áttekintési ellenőrzőlisták][checklists] segítségével ellenőrizheti, hogy a terve megfelel-e ezeknek a minőségi alappilléreknek. 

**[Tervezési minták felhőkhöz][patterns]**. Ezek a tervezési minták hasznosak megbízható, skálázható és biztonságos Azure-beli alkalmazások létrehozásához. Minden minta egy problémát, egy problémáról szóló mintát és egy Azure-alapú példát ismertet.

> &#10148; Tekintse át a [felhőkhöz készült tervezési minták teljes katalógusát](../patterns/index.md).


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

