---
title: Megbízható Azure-alkalmazások tervezése
description: Megbízható és magas rendelkezésre állású Azure-alkalmazások létrehozásának bemutatása
author: MikeWasson
ms.date: 04/10/2019
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-design-principles
ms.custom: ''
ms.openlocfilehash: 16c1daeed9b1d79f33051eb69571f1574bf9be4d
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/17/2019
ms.locfileid: "59644014"
---
# <a name="designing-reliable-azure-applications"></a>Megbízható Azure-alkalmazások tervezése

A megbízható alkalmazások felhőben történő létrehozása különbözik a hagyományos alkalmazásfejlesztéstől. Míg régebben megoldást jelenthetett, ha jobb minőségű hardvert vett a vertikális felskálázás érdekében, a felhőalapú környezetekben ehelyett horizontálisan kell felskáláznia. Az összes hiba megakadályozása helyett a cél egyetlen hibás összetevő hatásának minimálisra csökkentése.

A megbízható alkalmazások a következő tulajdonságokkal rendelkeznek:

- **Rugalmasak**, hibaelhárításuk zökkenőmentes, illetve minimális állásidő és adatveszteség mellett működnek a teljes helyreállításig.
- **Magas rendelkezésre állásúak (HA)** és terv szerint, kifogástalan állapotban futnak jelentős állásidő nélkül.

A megbízható alkalmazások létrehozásához feltétlenül ismerni kell, hogy ezen elemek hogyan működnek együtt &mdash;, illetve hogyan befolyásolják a költségeket&mdash;. Ez segíthet meghatározni az elfogadható állásidő mennyiségét, a vállalkozására rótt potenciális költségeket és a helyreállítás során szükséges funkciókat.

Ez a cikk tömör áttekintést nyújt arról, hogyan építheti be a megbízhatóságot az Azure-alkalmazások tervezési folyamatának egyes lépéseibe. Minden szakasz tartalmaz egy részletes cikkre mutató hivatkozást, amely azt ismerteti, hogyan integrálhatja a megbízhatóságot a folyamat adott lépésébe. Az egyes Azure-szolgáltatások megbízhatósági megfontolásaiért tekintse át [az egyes Azure-szolgáltatások rugalmasságára vonatkozó ellenőrzési listát](../checklist/resiliency-per-service.md).

## <a name="build-for-reliability"></a>Megbízhatóságra tervezve

Ez a szakasz a megbízható Azure-alkalmazások létrehozásának hat lépését ismerteti. A lépések olyan szakaszokra hivatkoznak, amelyek még inkább meghatározzák a folyamatot és a feltételeket.

1. [**A követelmények meghatározása:**](#define-requirements) Szétbontott számítási feladatok és üzleti igények alapján fejlesztheti a rendelkezésreállási és helyreállítási követelményeket.
1. [**Az ajánlott architekturális eljárások használata:**](#use-architectural-best-practices) Követheti a bizonyított eljárásokat, azonosíthatja a lehetséges hibapontokat az architektúrában, és meghatározhatja, hogyan fog válaszolni az alkalmazás egy hibára.
1. [**Tesztelés szimulációkkal és kényszerített feladatátvételekkel:**](#test-with-simulations-and-forced-failovers) Hibákat szimulálhat, kényszerített feladatátvételeket aktiválhat, és tesztelheti az észlelést és a helyreállítást a hibák után.
1. [**Az alkalmazás konzisztens üzembe helyezése:**](#deploy-the-application-consistently) Megbízható és megismételhető folyamatok használatával bocsáthatja éles rendszerbe az alkalmazást.
1. [**Az alkalmazás állapotának monitorozása:**](#monitor-application-health) Észlelheti a hibákat, monitorozhatja a lehetséges hibák jelzőit, illetve felmérheti alkalmazásainak állapotát.
1. [**Válasz hibák és vészhelyzetek esetén:**](#respond-to-failures-and-disasters) Azonosíthatja a hibák fellépésének időpontját, és bevált stratégiák alapján határozhatja meg a hiba elhárításának módját.

## <a name="define-requirements"></a>Követelmények meghatározása

Határozza meg üzleti igényeit, és hozza létre megbízhatósági tervét azoknak megfelelően. A következőket ajánljuk figyelmébe:

- **Számítási feladatok és kihasználtság meghatározása:** A *számítási feladat* olyan különálló képességet vagy feladatot jelent, amely logikusan elválasztható más feladatoktól az üzleti logika és az adattárolási követelmények szerint. Az egyes számítási feladatok különböző követelményekkel rendelkezhetnek a rendelkezésre állás, a skálázhatóság, az adatkonzisztencia és a vészhelyreállítás alapján.
- **A használati minták megtervezése:** A követelményeket a *használati minták* is befolyásolhatják. Azonosítsa a követelmények közötti különbségeket kritikus és nem kritikus időszakokban. Egy adóbevallást készítő alkalmazás például nem hibásodhat meg a bevallás elküldésének határideje előtt. Az üzemidő biztosításának érdekében tervezze meg a redundanciát több régión keresztül annak esetére, ha valamelyik meghibásodik. Ezzel szemben a nem kritikus időszakok alatti költségcsökkentés érdekében futtathatja alkalmazását egyetlen régióban is.
- **A rendelkezésreállási metrikák meghatározása – *a helyreállítás átlagos ideje* (MTTR) és *a hibák közötti átlagos idő* (MTBF):** A helyreállítás átlagos ideje (MTTR) az az átlagos idő, amely az összetevő egy hiba utáni helyreállításához szükséges. A hibák közötti átlagos idő (MTBF) az az idő, amelyre az összetevő számíthat a szolgáltatáskimaradások között. Ezekkel a mérőszámokkal meghatározhatja a redundancia hozzáadásának helyét és a szolgáltatásiszint-szerződéseket (SLA-kat) az ügyfelek számára.  
- **A helyreállítási metrikák meghatározása – helyreállítási időre vonatkozó célkitűzés és helyreállítási időkorlát (RPO):** A *helyreállítási időre vonatkozó célkitűzés* (RTO) az a maximális elfogadható idő, ameddig egy alkalmazás elérhetetlen maradhat egy incidens után. A *helyreállítási időkorlát* (RPO) a katasztrófa során elfogadható adatveszteség maximális ideje. Az értékek kiszámításához végezzen kockázatfelmérést, és győződjön meg arról, hogy ismeri a szervezetben esetlegesen bekövetkező állásidő és adatvesztés járulékos költségeit és kockázatát.
    > [!NOTE]
    > Ha a kritikus összetevők *bármely* MTTR-értéke egy magas rendelkezésreállási környezetben meghaladja a rendszer RTO-értékét, a rendszer hibája az üzletmenet elfogadhatatlan mértékű szüneteltetését okozhatja. Tehát a rendszert nem lehet majd visszaállítani a megadott RTO-n belül.
- **A számítási feladatok rendelkezésreállási céljainak meghatározása:** Annak érdekében, hogy az alkalmazás architektúrája megfeleljen üzleti igényeinek, határozza meg az egyes számítási feladatok cél SLA-it. Az alkalmazásfüggőségek mellett vegye számításba a rendelkezésreállási követelményeknek történő megfelelés költségeit és összetettségét is.
- **A szolgáltatásiszint-szerződések megismerése:** Az Azure-ban a szolgáltatásiszint-szerződés (SLA) ismerteti a Microsoft az üzemidővel és hálózati elérhetőséggel kapcsolatos vállalásait. Ha egy adott szolgáltatás SLA-ja 99,9%-os, az azt jelenti, hogy a szolgáltatás várhatóan az idő 99,9%-ában elérhető.  

    Határozza meg saját cél SLA-it a megoldás összes számítási feladatához annak érdekében, hogy meghatározhassa, hogy az architektúra megfelel-e az üzleti követelményeknek. Ha például egy számítási feladat 99,99%-os üzemidőt igényel, de egy 99,9%-os SLA-val rendelkező szolgáltatásra támaszkodik, akkor a szolgáltatás nem lehet rendszerkritikus meghibásodási pont.

További információt a megbízható alkalmazások tervezési követelményeivel kapcsolatban a [rugalmas Azure-alkalmazások fejlesztési követelményeit](./requirements.md) ismertető részben talál.

## <a name="use-architectural-best-practices"></a>Ajánlott architekturális eljárások használata

Az architekturális fázisban összpontosítson arra, hogy olyan eljárásokat vezessen be, amelyek megfelelnek üzleti igényeinek, azonosítják a hibapontokat, és minimalizálják a hibák hatókörét.

- **Hibaállapot-elemzés (FMA) elvégzése:** A hibaállapot-elemzés (FMA) beépíti a rugalmasságot az alkalmazásba a tervezési fázis korai szakaszában. Az FMA segít azonosítani az alkalmazásában esetlegesen felmerülő hibák típusát, azok lehetséges hatását és a lehetséges helyreállítási stratégiákat.
- **Redundanciaterv létrehozása:** Az egyes számítási feladatokhoz szükséges redundancia szintje az Ön üzleti igényeitől függ, és beleszámít az alkalmazás teljes költségébe. 
- **Tervezés a skálázhatóság szem előtt tartásával:** A felhőalapú alkalmazásoknak képesnek kell lenniük a változásokhoz történő igazodásra a skálázhatóság érdekében. Kezdjen különálló összetevőkkel, és tervezze az alkalmazást úgy, hogy automatikusan válaszoljon a terhelés változásaira, amikor csak lehetséges. A tervezés alatt tartsa szem előtt a skálázási korlátokat annak érdekében, hogy a jövőben egyszerűen bővíthesse alkalmazását.
- **Az előfizetési és szolgáltatáskövetelmények meghatározása:** Szükség lehet további előfizetésekre annak érdekében, hogy elegendő erőforrást építhessen ki, amelyek megfelelnek a tárterületre, a kapcsolatokra, az átviteli sebességre és egyéb szempontokra vonatkozó üzleti igényeinek. 
- **A kérések elosztása terheléselosztás használatával:** A terheléselosztás elosztja az alkalmazás kéréseit a kifogástalan állapotú szolgáltatáspéldányok között a nem megfelelő állapotú példányok rotációból történő eltávolításával.
- **Rugalmassági stratégiák meghatározása:** A *rugalmasság* a rendszer azon képessége, hogy helyreálljon a hibák után, és folytassa a működést. Valósítson meg olyan, [rugalmassággal kapcsolatos tervezési mintákat](../patterns/category/resiliency.md), mint például a kritikus erőforrások elkülönítése, kompenzáló tranzakciók használata, illetve aszinkron műveletek végrehajtása, amikor csak lehetséges.
- **A rendelkezésreállási követelmények beépítése a tervezésbe:** A *rendelkezésre állás* az az időarány, amíg a rendszer működik és üzemel. Tegyen lépéseket annak érdekében, hogy az alkalmazás rendelkezésre állása megfeleljen a szolgáltatásiszint-szerződésnek. Például kerülje el a kritikus hibapontokat, bontsa fel a számítási feladatokat szolgáltatásiszint-célkitűzések szerint, illetve szabályozza a nagy forgalmú felhasználókat.
- **Az adatok kezelése:** Az adatok tárolásának, biztonsági mentésének és replikálásának módja kulcsfontosságú.

  - **Az alkalmazásadatok replikálási módszereinek kiválasztása:** Az alkalmazásadatok különböző adattárakban vannak tárolva, és eltérő rendelkezésreállási követelményekkel rendelkezhetnek. Értékelje a replikációs módszereket és helyeket az egyes adattártípusok esetében, hogy meggyőződjön arról, hogy azok kielégítik az igényeit.
  - **A feladatátvételi és feladat-visszavételi folyamatok dokumentálása és tesztelése:** Egyértelműen dokumentálja az utasításokat egy új adattárba történő feladatátvétel esetére, és tesztelje őket rendszeresen, hogy meggyőződjön arról, hogy pontosak és könnyen követhetők.
  - **Az adatok védelme:** Rendszeresen készítsen biztonsági mentést adatairól, és ellenőrizze őket, illetve győződjön meg arról, hogy egyetlen felhasználói fiók sem fér hozzá az éles adatokhoz és biztonsági mentési adataihoz.
  - **Az adat-helyreállítás tervezése:** Győződjön meg arról, hogy a biztonsági mentési és replikációs stratégia a szolgáltatásiszint-követelményeknek megfelelő adat-helyreállítási időt biztosít. Vegye tekintetbe az alkalmazás által használt összes adattípust, a referenciaadatokat és az adatbázisokat is beleértve.

A megbízható alkalmazások architekturális tervezésével kapcsolatban további információt a [Rugalmas és magas rendelkezésre állású Azure-alkalmazások architekturális tervezését](./architect.md) ismertető szakaszban találhat.

## <a name="test-with-simulations-and-forced-failovers"></a>Tesztelés szimulációkkal és kényszerített feladatátvétellel

A megbízhatósági teszteléshez azt kell megvizsgálni, hogyan teljesít a végpontok közötti munkaterhelés csak időnként jelentkező hibafeltételek mellett.

- **A rendszer viselkedésének tesztelése gyakori meghibásodási helyzetekben valódi hibák kiváltása vagy azok szimulálása révén:** Hibabeszúrás használata gyakori forgatókönyvek (ideértve több hiba kombinációját is) és a helyreállítási idő tesztelésére.
- **A csak terhelés alatt bekövetkező hibák azonosítása:** Ahhoz, hogy lássa, az alkalmazás hogyan viselkedik valós feltételek között, tesztelje a csúcsterhelést éles adatok vagy az éles adatokhoz lehető legközelebb álló szintetikus adatok használatával.
- **Vészhelyreállítási próbák végrehajtása:** Rendelkezzen vészhelyreállítási tervvel, és rendszeresen tesztelje működését.
- **Feladatátvételi és feladat-visszavételi tesztek végrehajtása:** Bizonyosodjon meg arról, hogy az alkalmazás függő szolgáltatásai a feladatátvételt és -visszavételt helyes sorrendben hajtják végre.
- **Szimulációs tesztek futtatása:** A valós életből vett forgatókönyvek alapján történő tesztelés segíthet azonosítani a kezelésre szoruló problémákat. A forgatókönyveknek vezérelhetőknek kell lenniük, és nem szabad az üzletmenetben zavart okozniuk. Tájékoztassa a vezetőséget a szimulációs tesztelési tervekről.
- **Állapotadat-mintavételek tesztelése:** Konfiguráljon állapotadat-mintavételeket a terheléselosztó és forgalomkezelő alrendszerekhez a kritikus rendszerösszetevők ellenőrzése érdekében. Tesztelje, hogy megfelelő módon válaszolnak-e.
- **A monitorozó rendszerek tesztelése:** A lehetséges hibák észlelése érdekében bizonyosodjon meg arról, hogy a monitorozó rendszerek megbízhatóan jelentik a kritikus információkat és pontos adatokat adnak vissza.
- **A harmadik féltől származó szolgáltatások bevonása a tesztelési forgatókönyvekbe:** A helyreállítás mellett tesztelje a külső szolgáltatáskimaradás által érintett lehetséges meghibásodási pontokat is.

A tesztelés iteratív folyamat. Tesztelje az alkalmazást, mérje meg az eredményt, elemezze és kezelje a kapott hibákat, majd ismételje meg a folyamatot.

Az alkalmazások megbízhatósági tesztelésével kapcsolatban további információt az [Azure-alkalmazások rugalmasságának és rendelkezésre állásának tesztelését](./testing.md) ismertető szakaszban találhat.

## <a name="deploy-the-application-consistently"></a>Az alkalmazás konzisztens üzembe helyezése

Az *üzembe helyezés* magában foglalja az Azure-erőforrások kiépítését, az alkalmazáskód üzembe helyezését és a konfigurációs beállítások alkalmazását. A frissítések ezek némelyikére vagy mindegyikére terjedhetnek ki.

Miután egy alkalmazás éles telepítése megtörtént, a frissítések hibaforrások lehetnek. A hibalehetőségek minimálisra csökkentése érdekében az üzembehelyezési folyamatnak kiszámíthatónak és megismételhetőnek kell lennie.

- **Az alkalmazás-üzembehelyezési folyamat automatizálása:** Automatizálja a lehető legtöbb folyamatot.
- **A kiadási folyamat maximális rendelkezésre állással való megtervezése:** Ha a kiadási folyamat során néhány szolgáltatást le kell választani, az alkalmazás nem lesz elérhető, amíg ezek újra el nem indulnak. Használja ki a platform-előkészítés és az éles környezet funkcióit. Használjon kék-zöld vagy tesztcsoportos eljárásokat a frissítések kiadására, hogy abban az esetben, ha hiba lép fel, gyorsan vissza lehessen vonni a frissítést.
- **Visszavonási terv létrehozása az üzembe helyezéshez:** Tervezzen visszavonási folyamatot, hogy visszaállíthassa a legutóbbi ismert jól működő verziót, és minimalizálhassa az állásidőt.
- **Az üzembe helyezések naplózása és vizsgálata:** Ha szakaszos üzembehelyezési eljárásokat használ, az alkalmazásnak egyszerre több verziója fog futni az éles környezetben. Egy megbízható naplózási stratégiával rögzítse a lehető legtöbb verzióspecifikus információt.
- **Az alkalmazás kiadási folyamatának dokumentálása:** Egyértelműen határozza meg és dokumentálja a kiadási folyamatot, és győződjön meg arról, hogy az a teljes üzemeltetési csapat rendelkezésére áll.

További információt az alkalmazás megbízhatóságáról és üzembe helyezéséről a[rugalmas és magas rendelkezésre állású Azure-alkalmazások üzembe helyezését](./deploy.md) ismertető szakaszban talál.

## <a name="monitor-application-health"></a>Alkalmazásállapot monitorozása

Alkalmazzon ajánlott eljárásokat az alkalmazás monitorozásához és figyelmeztetések küldéséhez, hogy képes legyen észlelni a hibákat és figyelmeztetni az operátort, aki ki tudja azokat javítani.

- **Állapotadat-mintavételek és ellenőrző funkciók alkalmazása:** Futtassa ezeket rendszeresen az alkalmazáson kívül, hogy észlelhesse az alkalmazás állapotának és teljesítményének romlását.
- **Hosszan futó munkafolyamatok ellenőrzése:** A hibák korai észlelésével elkerülheti, hogy szükség legyen az egész munkafolyamat visszavonására vagy több kompenzáló tranzakció lefuttatására.
- **Alkalmazásnaplók használata:**

  - Naplózza az éles alkalmazásokat és a szolgáltatáshatárokon történő eseményeket.
  - Használjon szemantikus és aszinkron naplózást.
  - Különítse el az alkalmazásnaplókat az auditnaplóktól.

- **Távoli hívási statisztikák gyűjtése és az adatok megosztása az alkalmazás csapatával:** Azonnali áttekintést nyújthat az üzemeltetési csapatnak az alkalmazás állapotáról a távoli hívások metrikáinak, például a késés, az átviteli sebesség és a 99 és 95 százalékos értékbe eső hibák összegzésével. Végezzen statisztikai elemzést a metrikákon az egyes százalékos értékekben előforduló hibák felismeréséhez.
- **Az átmeneti kivételek és az újrapróbálkozások nyomon követése egy megadott időkeretben:** Ha növekszik a kivételek száma egy adott időszakban, a szolgáltatással problémák lehetnek, ami leálláshoz vezethet.
- **Korai figyelmeztetési rendszer létrehozása:** Azonosítsa az alkalmazás állapotának fő teljesítménymutatóit (KPI), mint például az átmeneti kivételek száma vagy a távoli hívások késése, és állítson be megfelelő küszöbértéket mindegyikhez. Az adott küszöbérték elérésekor küldjön figyelmeztetést az operátoroknak.
- **Üzemeltetés az Azure-előfizetés által meghatározott korlátok között:** Az Azure-előfizetések korlátozzák bizonyos erőforrástípusok használatát. Korlátozások vonatkozhatnak például az erőforráscsoportok, a processzormagok és a tárfiókok számára. Kísérje figyelemmel az erőforrástípusok használatát.
- **Külső szolgáltatások monitorozása:** Naplózza a hívásokat és kapcsolja őket össze alkalmazásának állapot- és diagnosztikai naplózási adataival egyedi azonosítók használatával.
- **Több operátor kiképzése az alkalmazás monitorozására és manuális helyreállítási lépések végrehajtására:** Bizonyosodjon meg arról, hogy legalább egy hozzáértő operátor mindig aktív legyen.

Az alkalmazásmegbízhatóság monitorozásával kapcsolatban további információt az [Azure-alkalmazások állapotának monitorozását](./monitoring.md) ismertető szakaszban kaphat.

## <a name="respond-to-failures-and-disasters"></a>Válasz a hibákra és a vészhelyzetekre

Hozzon létre egy helyreállítási tervet, és győződjön meg arról, hogy az kezeli az adatok helyreállítására, a hálózati kimaradásokra, a függő szolgáltatás kiesésére és a régióra kiterjedő szolgáltatáskimaradásra vonatkozó helyzeteket. Vegye számításba a virtuális gépeket, tárakat, adatbázisokat és az egyéb Azure-platformszolgáltatásokat a helyreállítási stratégia létrehozásakor.

- **Az Azure támogatási szolgálatával való kapcsolatfelvétel megtervezése:** Határozzon meg eljárást az Azure támogatási szolgálatával való kapcsolatfelvételre még azelőtt, hogy szükség lenne rá.
- **A vészhelyreállítási terv dokumentálása és tesztelése:** Olyan vészhelyreállítási tervet írjon, amely tükrözi az alkalmazáshibák üzletmenetre gyakorolt hatását. A lehető legnagyobb mértékben automatizálja a visszaállítási folyamatot, és dokumentálja az összes manuális lépést. A terv ellenőrzéséhez és továbbfejlesztéséhez rendszeresen tesztelje a vészhelyreállítási folyamatot.
- **Manuális feladatátvétel szükség esetén:** Egyes rendszerek nem tudják automatikusan átadni a feladataikat. Az ilyen esetekben manuális feladatátvételre van szükség. Ha az alkalmazás egy másodlagos régióba adja át a feladatait, tesztelje annak működési készenlétét. A visszavétel előtt ellenőrizze, hogy az elsődleges régió állapota megfelelő-e, és készen áll-e a forgalom újbóli fogadására. Határozza meg az alkalmazás csökkentett funkciókészletét, és hogy az alkalmazás hogyan tájékoztatja a felhasználókat az ideiglenes problémákról.
- **Előzetes felkészülés az alkalmazáshibákra:** Készüljön fel többféle hibára is egyszerre, beleértve az automatikusan kezelt hibákat, a csökkentett funkcionalitást eredményező hibákat és azokat a hibákat, amelyek miatt az alkalmazás rendelkezésre állása megszűnik. Az alkalmazásnak tájékoztatnia kell a felhasználókat az ideiglenes problémákról.
- **Helyreállítás adatsérülés esetén:** Ha egy adattár meghibásodik, ellenőrizze, hogy konzisztensek-e az adatok, amikor a tároló ismét elérhetővé válik. Ez különösen fontos ha a rendszer replikálta az adatokat. A sérült adatokat állítsa vissza a biztonsági mentésből.
- **Helyreállítás hálózatkiesés után:** Lehet, hogy az alkalmazás képes gyorsítótárazott adatok segítségével helyileg futni, csökkentett funkcionalitással. Ha ez nem lehetséges, akkor leállíthatja az alkalmazást vagy átadhatja a feladatait egy másik régiónak. Amíg a kapcsolat vissza nem áll, tárolja az adatokat egy másik helyen.
- **Helyreállítás függő szolgáltatás kiesése után:** Határozza meg, melyik funkció érhető el továbbra is, és hogy az alkalmazás hogyan válaszoljon.
- **Helyreállítás egész régióra kiterjedő szolgáltatáskimaradás esetén:** Régióra kiterjedő szolgáltatáskimaradások ritkán fordulnak elő, de különösen a kritikus fontosságú alkalmazások esetében szükség van olyan stratégiára, amellyel kezelheti ezeket. Lehet, hogy képes átadni az alkalmazást egy másik régióba vagy átterelni a forgalmat.

A hibakezeléssel és vészhelyreállítással kapcsolatban további információt az [Azure-alkalmazások hibaelhárítása és vészhelyreállítása](./disaster-recovery.md) című szakaszban találhat.

## <a name="next-steps"></a>További lépések

> [!div class="nextstepaction"]
> [Rugalmas alkalmazásokra vonatkozó követelmények kidolgozása:](./requirements.md)
