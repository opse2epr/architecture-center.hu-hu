---
title: A rugalmasság és a rendelkezésre állási tesztelési Azure-alkalmazások
description: Módszerek az alkalmazások megbízhatóságát tesztelését az Azure-ban
author: MikeWasson
ms.date: 04/10/2019
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-design-principles
ms.openlocfilehash: cf33a859540810279b1cb85be5a6fb2ebe4500dc
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/17/2019
ms.locfileid: "59646550"
---
# <a name="testing-azure-applications-for-resiliency-and-availability"></a>A rugalmasság és a rendelkezésre állási tesztelési Azure-alkalmazások

Rugalmasság tesztelése, ellenőrizze, hogyan hajtja végre a végpontok közötti munkaterhelés időszakos hibafeltételek mellett.

Futtasson teszteket használata szintetikus és a valós felhasználói adatainak, mind éles környezetben. Tesztelési és éles azonosak ritkán, ezért fontos, hogy az alkalmazás az éles-ellenőrzése egy [kék-zöld](https://martinfowler.com/bliki/BlueGreenDeployment.html) vagy [tesztcsoportos telepítési](https://martinfowler.com/bliki/CanaryRelease.html). Ezzel a módszerrel tesztelt az alkalmazás valós körülmények között, így biztosítható, hogy teljes mértékben telepítésekor várt módon működik.

A tesztelési terv részeként a következők:

- Automatikus üzembe helyezés előtti tesztelése
- Tesztelés hibabeszúrással
- Csúcsidőszak terheléses tesztelés
- Vészhelyreállítás tesztelését
- Külső szolgáltatások tesztelése

A tesztelés iteratív folyamat. Tesztelje az alkalmazást, mérje meg az eredményt, elemezze és kezelje az esetleges kapott hibákat, majd ismételje meg a folyamatot.

## <a name="perform-fault-injection-testing"></a>Tesztelés hibabeszúrással végrehajtása

Tesztelés hibabeszúrással, ellenőrizze a rugalmasság a rendszer, meghibásodások valódi hibák elindításával vagy azok szimulálása. Az alábbiakban néhány stratégiát a idéz elő hibákat:

- Állítsa le a virtuális gép (VM) instances.
- Váltsa ki egyes folyamatok összeomlását.
- Alkalmazzon lejárt tanúsítványokat.
- Módosítson hozzáférési kulcsokat.
- Állítsa le a DNS-szolgáltatást a tartományvezérlőkön.
- Korlátozza az elérhető rendszererőforrásokat, mint például a RAM-ot vagy a szálak számát.
- Válasszon le lemezeket.
- Helyezzen ismételten üzembe egy virtuális gépet.

A tesztelési terv tartalmazniuk kell a lehetséges meghibásodási pontok azonosítani a tervezési fázisban mellett gyakori meghibásodási helyzet:

- Az alkalmazás tesztelése éles minél közelebb környezetben.
- Tesztelési hibák együtt.
- Mérje meg a helyreállítási időtartamokat, és győződjön meg arról, hogy az üzleti követelmények teljesülnek-e.
- Győződjön meg arról, hogy hibák terjedésük kiküszöbölésére, és a egy elkülönített módon kezeli.

Hiba forgatókönyvekkel kapcsolatos további információkért lásd: [hiba és a katasztrófa utáni helyreállítás az Azure-alkalmazásokhoz](./disaster-recovery.md).

## <a name="test-under-peak-loads"></a>Csúcsterhelés alatt tesztelése

Terheléses tesztelés elengedhetetlen, hogy csak a háttér-adatbázis példányairól például a terhelés alatt bekövetkező hibák azonosításáért, vagy a társzolgáltatás szabályozása. Tesztelje a csúcsterhelés között és a csúcsterhelés között, termelési adatok vagy a termelési adatokhoz lehető legközelebb álló szintetikus adatok használatával várható növekedése. A cél, hogy, hogyan viselkedik az alkalmazás valós körülmények között.

## <a name="conduct-disaster-recovery-drills"></a>Vészhelyreállítási próba magatartási

Egy jó vész-helyreállítási terv kezdődnie, és tesztelje, rendszeres időközönként ellenőrizze, hogy működik-e.

Használhatja az Azure Virtual Machines [Azure Site Recovery](/azure/site-recovery/azure-to-azure-quickstart/) replikálja, és hajtsa végre [vészhelyreállítási próbák](/azure/site-recovery/azure-to-azure-tutorial-dr-drill/) éles üzemi alkalmazások pedig, vagy folyamatban lévő replikáció befolyásolása nélkül.

### <a name="failover-and-failback-testing"></a>Feladatátvétel és feladat-visszavétel tesztelése

Teszt feladatátvétel és feladat-visszavételt, győződjön meg arról, hogy az alkalmazás függő szolgáltatások térjen vissza szinkronizált módon a vészhelyreállítás során. Változások rendszerek és a műveletek hatással lehetnek a feladatátvétel és feladat-visszavétel funkciók, de a hatás nem észlelhető, mindaddig, amíg a fő rendszer meghibásodik, vagy túlterhelt válik. A teszt feladatátvételt képességek *előtt* élő probléma kompenzálják szüksége van rájuk. Is, hogy a függő szolgáltatások átadja a feladatokat, és a megfelelő sorrendben sikertelen.

Ha használ [Azure Site Recovery](/azure/site-recovery/) virtuális gépek replikálásához vészhelyreállítási próbákat rendszeres időközönként végrehajtásával tesztek segítségével ellenőrizze a replikációs stratégiát. Feladatátvételi teszt nem érinti a folyamatban lévő virtuális gépek replikációja vagy az éles környezetben. További információkért lásd: [vészhelyreállítási gyakorlatának futtatása az Azure-bA](/azure/site-recovery/site-recovery-test-failover-to-azure).

### <a name="simulation-testing"></a>Szimuláció tesztelése

Szimuláció tesztelés magában foglalja a kis méretű, valósághű helyzetek létrehozását. Szimuláció hatékonyságát a megoldások bemutatása a helyreállítási tervben szereplő, és jelölje ki az esetleges problémákat, amelyek nem győződünk.

Szimuláció tesztelés hajt végre, kövesse az ajánlott eljárásokat:

- Olyan módon, amely nem zavarja a tényleges üzleti, de úgy érzi, egy valódi helyzetben szimulációkat végez.
- Győződjön meg arról, hogy szimulált forgatókönyvek teljesen vezérelhető. A helyreállítási terv úgy tűnik, hogy sikertelen lesz, ha anélkül, hogy a károk Ez a helyzet állíthatja vissza a normál.
- Tájékoztatja arról, hogy mikor és hogyan fogja végezni a szimuláció gyakorlatok. A terv kell részletes információkat talál, az időtartamot és a során a szimuláció érintett erőforrások.

## <a name="test-health-probes-for-load-balancers-and-traffic-managers"></a>A terheléselosztókkal és a traffic Manager-példányok állapotadat-mintavételek tesztelése

A terheléselosztókkal és a traffic Manager-példányok állapotadat-mintavételek tesztelése és konfigurálása. Győződjön meg arról, hogy az állapot végponti ellenőrzi a rendszer kritikus fontosságú részei, és megfelelően válaszol-e.

- A [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview/), az állapotminta határozza meg, a feladatátvételt egy másik régióba. Az állapot végponti ellenőrizni kell az azonos régióban üzembe helyezett kritikus fontosságú függőségek.
- A [Azure Load Balancer](/azure/load-balancer/load-balancer-overview/), az állapotminta határozza meg, távolítsa el a virtuális Gépet a rotációból. Az állapot végponti jelentse a virtuális gép állapotát. Más rétegek vagy a külső szolgáltatások nem tartalmazzák a. Ellenkező esetben hiba, amely a virtuális gép kívül történik, hogy távolítsa el a virtuális Gépet a rotációból a terheléselosztó miatt.

Végrehajtási egészségügyi figyelés az alkalmazásban, tekintse át [állapot végponti Monitorozását végző minta](../patterns/health-endpoint-monitoring.md).

## <a name="test-monitoring-systems"></a>Teszt rendszerek figyelése

Például a tesztelési terv rendszerek figyelése. Automatikus feladatátvétel és feladat-visszavétel rendszerek attól függenek, monitorozási és rendszerállapot megfelelő működésére. Irányítópultokat a rendszer állapotának és operátor riasztások megjelenítését is függ, hogy pontos monitorozási és rendszerállapot. Ha ezek az elemek meghibásodik, a kritikus információk tévesztés, vagy pontatlan adatokat, az operátornak előfordulhat, hogy nem valósíthat meg, hogy a rendszer sérült vagy meghibásodott.

## <a name="include-third-party-services-in-test-scenarios"></a>Harmadik féltől származó szolgáltatásokkal felvétel tesztelési helyzetek

Ha az alkalmazás függőségeit a harmadik féltől származó szolgáltatásokkal, határozza meg, hol és hogyan meghiúsulhat, ezek a szolgáltatások, és milyen hatással ezekre a hibákra lesz az alkalmazás. Vegye figyelembe a szolgáltatásiszint-szerződés (SLA a külső szolgáltatáshoz és a hatás, előfordulhat, hogy rendelkezik a vészhelyreállítási tervet).

Egy külső szolgáltatás nem előfordulhat, hogy monitorozási és diagnosztikai képességeket biztosítanak, így a fontos jelentkezhetnek a meghívásához, és összefüggésbe hozva azokat az az alkalmazás állapotáról és a diagnosztikai naplózás egy egyedi azonosító segítségével. A monitorozási és diagnosztikai bevált gyakorlatokat további információkért lásd: [figyelési és diagnosztikai útmutató](../best-practices/monitoring.md).

## <a name="next-steps"></a>További lépések

> [!div class="nextstepaction"]
> [A rugalmasság és a rendelkezésre állási üzembe helyezése](./deploy.md)
