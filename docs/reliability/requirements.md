---
title: Az Azure rugalmas alkalmazások követelményeinek meghatározása
description: Az Azure-alkalmazások rendelkezésre állását és rugalmasságát követelményei összegyűjtéséhez áttekintése
author: MikeWasson
ms.date: 04/10/2019
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-design-principles
ms.openlocfilehash: 2af2ce4200c285abfda1395862d21efc3c17e577
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/17/2019
ms.locfileid: "59646549"
---
# <a name="developing-requirements-for-resilient-azure-applications"></a>Fejlesztés az Azure rugalmas alkalmazások követelményei

Épület *rugalmasság* (helyreálljon a hibák) és *rendelkezésre állási* (jelentős állásidő nélkül kifogástalan állapotban fut) alkalmazásaiba kezdődik követelmények összegyűjtése. Például hogy mennyi állásidő fogadható el? A lehetséges állásidő mennyibe vállalkozása? Mik azok az ügyfelek rendelkezésre állási követelmények vonatkoznak? IP-címek fenntartási tegye befektetnie az alkalmazás magas rendelkezésre állású? Mi az a kockázat és a költségek?

## <a name="identify-distinct-workloads"></a>Azonosíthatja a különböző számítási feladatokhoz

Felhőalapú megoldások jellemzően több alkalmazás *számítási feladatok*. Egy számítási feladat olyan különálló képességet vagy feladatot, amely logikailag elkülönítve más feladatok szempontjából üzleti logikát és adatokat a tárolási követelményeket. Előfordulhat például, hogy egy e-kereskedelmi alkalmazás a következő számítási feladatokkal:

- Termékkatalógus tallózása és keresés.
- Rendelések létrehozása és nyomon követése.
- Javaslatok megtekintése.

Minden számítási feladathoz tartozik rendelkezésre állási, skálázhatósági, az adatkonzisztencia és vész-helyreállítási különböző követelmények vonatkoznak. A költségek és a kockázatot az egyes munkaterhelésekhez tartozó terheléselosztási az üzleti döntéseket hozhat.

Szolgáltatásiszint-célkitűzés által számítási feladatok lebontása is. Ha egy szolgáltatás áll, amelyek kritikus fontosságú és kevésbé kritikus fontosságú számítási feladatokhoz, eltérően kezelheti őket, és adja meg a service-szolgáltatások és a rendelkezésre állási követelmények teljesítése érdekében példányok száma.

## <a name="plan-for-usage-patterns"></a>Használati minták tervezése

Különbségek a követelmények azonosítása kritikus és a nem kritikus időszakok alatt. Vannak bizonyos kritikus időszakok, amikor a rendszernek elérhetőnek kell lennie? Például egy készítő alkalmazás nem utasíthat el egy bejelentési határidő során, és egy videóstream-szolgáltatás nem lag élő esemény során. Ebben az esetben a költség, a kockázatok ellen mérjük.

- Üzemidő biztosítása, és a kritikus időszakok megfelelnek a szolgáltatásiszint-szerződések (SLA), tervezze meg a redundancia több régióban abban az esetben, ha nem sikerül, akkor is, ha további költségei.
- Ezzel szemben nem kritikus időszakok alatt futtassa az alkalmazást a költségek csökkentése érdekében egyetlen régióban.
- Bizonyos esetekben a fogyasztás alapú számlázáshoz rendelkező modern kiszolgáló nélküli ugyanazzal a módszerrel csökkentheti a további költségek.

## <a name="establish-availability-and-recovery-metrics"></a>Rendelkezésre állási és helyreállítási metrikák létrehozása

Hozzon létre a követelmények folyamat részeként metrikák két készletnyi alapkonfiguráció-számot. Első segít meghatározni, hogy a helyét, amelyek a redundancia a cloud serviceshez, és mely szolgáltatásiszint-szerződései az ügyfeleknek. A második tervezi segít a vész-helyreállítási állítsa be.

### <a name="availability-metrics"></a>Rendelkezésre állási metrikák

Ezek a mértékek segítségével redundancia tervezése és ügyfél SLA-k meghatározása.

- **Átlagos idő (MTTR) helyreállításához** van az átlagos idő egy összetevő visszaállítása meghibásodás után vesz igénybe.
- **(MTBF) hibák közötti átlagos idő** a következőképpen hosszú összetevő ésszerűen számíthatnak az utolsó között valamilyen okból kimaradás lép.

### <a name="recovery-metrics"></a>Helyreállítási metrikák

A kockázatfelmérés a ezeket az értékeket származtatás, és győződjön meg arról, hogy megértette a költségek és az üzemkimaradás és az adatvesztés kockázatát. Ezek a rendszer nem funkcionális követelmények és üzleti követelmények során kell meghatározni.

- **A helyreállítási időre vonatkozó célkitűzés (RTO)** van egy incidens után az alkalmazás nem érhető el a maximális elfogadható idő.
- **Helyreállítási időkorlát (RPO)** katasztrófa során elfogadható-adatveszteség maximális időtartama.

Ha az MTTR értékét *bármely* egy magas rendelkezésre állású beállításra a kritikus fontosságú összetevőnek meghaladja a rendszer RTO, a rendszer hibát okozhat az üzletmenet elfogadhatatlan mértékű. Ez azt jelenti, hogy a rendszer a megadott RTO belül nem állítható vissza.

## <a name="determine-workload-availability-targets"></a>Számítási feladatok kijelölt elérhetőségi célok meghatározása

Adja meg saját cél SLA minden számítási feladathoz tartozó megoldását, így ellenőrizheti, hogy az architektúra megfelel-e az üzleti követelmények.

### <a name="consider-cost-and-complexity"></a>Fontolja meg a költségek és munkaráfordítás alól

Minden hagyhatunk, magasabb rendelkezésre állás a jobb. De minél több nines, a költséges és bonyolult növekszik. 99,99 %-os üzemidő a rendszer lefordítja arra az összesített állásidő havonta nagyjából öt perc alatt. Megéri a további összetettséget és költségeket az öt nines? A válasz az üzleti követelményektől függ.

Az alábbiakban néhány további szempontot találhat az SLA meghatározásához:

- Négy nines (99,99 %) eléréséhez nem támaszkodhat manuális beavatkozásra a hibákból való helyreállításhoz. Az alkalmazásnak képesnek kell lennie az öndiagnózisra és az önjavításra.
- Négy nines túl kihívást jelent, elég gyors észlelése az SLA feltételeinek kielégítéséhez.
- Gondoljon az SLA-ban megszabott, a mérés alapját képező időtartományra. Minél kisebb a tartomány, annál szigorúbbak a feltételek. Az SLA szempontjából óránkénti vagy napi üzemidő meghatározásához nincs értelme.
- Fontolja meg az MTBF- és az MTTR-mérőszám használatát. A magasabb az SLA-t, a kevésbé gyakran a szolgáltatás is leáll, és a gyorsabb a szolgáltatás helyre kell állítania.
- Szerződés kérhet le az ügyfelek a kijelölt elérhetőségi célok teljesítését az alkalmazás minden darab, és ez a dokumentum. Ellenkező esetben a a tervező nem felel meg az ügyfelek elvárásainak.

### <a name="identify-dependencies"></a>Függőségek azonosításához

Hajtsa végre a függőségi-leképezés gyakorlatok belső és külső függőségek azonosításához. Példák biztonsági vagy az identitást, például az Active Directory vagy a külső szolgáltatások, például egy fizetési szolgáltató vonatkozó függőségeket tartalmaznak, vagy e-mail üzenetküldő szolgáltatás.

Fordítson különös figyelmet külső függőségeit, amelyek egy meghibásodási pont lehet vagy szűk keresztmetszeteket okozhat. Ha egy számítási feladat 99,99 %-os üzemidőt igényel, de egy 99,9 %-os SLA-val olyan szolgáltatástól függ, a a szolgáltatás nem a rendszer egy meghibásodási pont lehet. Egy megoldás, hogy rendelkezik egy tartalék elérési úttal, abban az esetben, ha a szolgáltatás meghiúsul. Azt is megteheti más intézkedéseket a szolgáltatás hiba helyreállíthatók.

Az alábbi táblázat a különböző SLA-szintek lehetséges halmozott állásidejét mutatja be.

| **SLA** | **Állásidő hetente** | **Állásidő havonta** | **Állásidő évente** |
|---------|-----------------------|------------------------|-----------------------|
| 99%     | 1,68 óra            | 7,2 óra              | 3,65 nap             |
| 99.9%   | 10,1 perc          | 43,2 perc           | 8,76 óra            |
| 99.95%  | 5 perc             | 21,6 perc           | 4,38 óra            |
| 99.99%  | 1,01 perc          | 4,32 perc           | 52,56 perc         |
| 99.999% | 6 másodperc             | 25,9 másodperc           | 5,26 perc          |

## <a name="understand-service-level-agreements"></a>Szolgáltatásiszint-szerződések ismertetése

Az Azure-ban a [szolgáltatásiszint-szerződés](https://azure.microsoft.com/en-us/support/legal/sla/) a Microsoft elkötelezettségét üzemidejével és elérhetőségével számára. Ha egy adott szolgáltatás SLA-ja 99,9 %, a szolgáltatás elérhető legyen az idő 99,9 %-számíthat. Különböző szolgáltatásokhoz különböző SLA-kkal rendelkezik.

Az Azure SLA is magában foglalja a jóváírást, ha az SLA nem teljesül, az adott definícióit és rendelkezések *rendelkezésre állási* minden egyes szolgáltatás. Az SLA ezen eleme kényszerítési házirendként működik.

### <a name="composite-slas"></a>Összetett SLA-k

*Összetett SLA-k* több szolgáltatást támogató alkalmazás, minden egyes rendelkezésre állási szintű részletességű járnak. Vegyük példaként egy App Service-webalkalmazás, amely az Azure SQL Database ír. Az oktatóanyag összeállításakor ezek az Azure-szolgáltatások a következő SLA-kkal rendelkeztek:

- Az App Service web apps = 99,95 %
- SQL Database = 99,99%

Mi az alkalmazás esetében elvárt maximális állásidő? Ha valamelyik szolgáltatás meghibásodik, akkor a teljes alkalmazás meghibásodik. Az egyes szolgáltatások meghibásodásának valószínűsége független, tehát a az alkalmazás összetett SLA 99,95 %-os x 99,99 % = 99,94 %. Ez alacsonyabb az egyes SLA-k, ami nem meglepő, mert egy több szolgáltatásra támaszkodó alkalmazások több lehetséges meghibásodási pontok.

Az összetett SLA javítható független tartalék útvonalak létrehozásával. Például ha az SQL-adatbázis nem érhető el, helyezze a tranzakciókat későbbi feldolgozás céljából egy üzenetsorba.

![Összetett SLA](_images/composite-sla.png)

E kialakítás révén az alkalmazás akkor is elérhető, ha épp nem tud csatlakozni az adatbázishoz. Az alkalmazás azonban leáll, ha az adatbázis és az üzenetsor egyidejűleg hibásodik meg. Egyidejű hiba várt százalékos 0,0001 × 0,001,, ezért az ehhez a kombinált útvonalhoz tartozó összetett SLA:

- Adatbázis *vagy* üzenetsor = 1,0 − (0,0001 × 0,001) = 99,99999 %

A teljes összetett SLA:

- Webalkalmazás *és* (adatbázis *vagy* üzenetsor) = 99,95 %-os x 99,99999 % = \~99,95 %-os

Ennek a módszernek hátrányai is vannak. Az alkalmazáslogika összetettebb, licencdíjat kell fizetni, hogy a várólista, és figyelembe kell vennie a adatkonzisztenciával kapcsolatos problémák.

### <a name="slas-for-multiregion-deployments"></a>SLA többrégiós üzemelő példányok esetében

*SLA többrégiós üzemelő példányok esetében* magában foglalja a üzembe helyezése több régióban, és az Azure Traffic Manager átadja a feladatokat, ha az alkalmazás meghibásodik egy régióban a magas rendelkezésre állású technika.

Az összetett SLA egy többrégiós üzembe helyezéshez a következőképpen alakul:

- *N* van az egy régióban üzembe helyezett alkalmazás összetett SLA-t.
- *R* száma, régió, ahol az alkalmazás üzemel.

A várt esélye, hogy az alkalmazás nem tudja az összes régióban egyszerre ((1 − N) \^ R). Ha például egyetlen régióban SLA-ja 99,95 %-os:

- A két régió kombinált SLA = ((1 − 0.9995) 1 − \^ 2) = 99.999975 %
- A négy régióban kombinált SLA = ((1 − 0.9995) 1 − \^ 4) 99.999999 % =

A [Traffic Managerre vonatkozó SLA](https://azure.microsoft.com/en-us/support/legal/sla/traffic-manager/v1_0/) is fontos tényező. Feladatátadás nem azonnali aktív-passzív konfigurációk, ami állásidőt eredményezhet a feladatátvétel alatt áll. Lásd: [Traffic Manager végpont figyelése és feladatátvétele](/azure/traffic-manager/traffic-manager-monitoring).

## <a name="next-steps"></a>További lépések

> [!div class="nextstepaction"]
> [A rugalmasság és a rendelkezésre állási mérnök](./architect.md)
