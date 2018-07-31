---
title: Rugalmasságra vonatkozó ellenőrzőlista
description: A feladatlista, amely rugalmasság aggályokat során tervezési útmutatást nyújt.
author: petertaylor9999
ms.date: 01/10/2018
ms.custom: resiliency, checklist
ms.openlocfilehash: 883424d5d3535f822cdba61ecb9520ce05f75ec7
ms.sourcegitcommit: 2154e93a0a075e1f7425a6eb11fc3f03c1300c23
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 07/30/2018
ms.locfileid: "39352644"
---
# <a name="resiliency-checklist"></a>Rugalmasságra vonatkozó ellenőrzőlista

Rugalmasság rendszer azon képessége, hogy helyreálljon a hibák után, és továbbra is működnek, és az egyik a [szoftverminőség alappillérei](../guide/pillars.md). Az alkalmazást a rugalmassághoz tervezése tervezése, és a hibaállapot esetleg előforduló különféle csökkentése igényel. Az alábbi ellenőrzőlista használatával tekintse át az alkalmazásarchitektúra rugalmassági szempontból. Emellett tekintse át a [rugalmassági ellenőrzőlista az adott Azure-szolgáltatások](./resiliency-per-service.md).

## <a name="requirements"></a>Követelmények

**Adja meg az ügyfél rendelkezésre állási követelmények vonatkoznak.** Az ügyfél lesz a rendelkezésre állási követelmények vonatkoznak, az összetevők az alkalmazásban, és ez hatással lesz az alkalmazás-tervezés. Szerződés kérhet le az ügyfél a kijelölt elérhetőségi célok teljesítését az alkalmazás minden darab, ellenkező esetben a Tervező előfordulhat, hogy nem felel meg az ügyfelek elvárásainak. További információkért lásd: [rugalmas alkalmazások tervezése az Azure](../resiliency/index.md).

## <a name="application-design"></a>Alkalmazás-tervezés

**Hajtsa végre a hibaállapot-elemzést (FMA) az alkalmazáshoz.** FMA egy olyan folyamat, amellyel a rugalmasság korai tervezési szakaszában az alkalmazásba. További információkért lásd: [hibaállapot elemzése][fma]. Az FMA célja a következők:  

* Azonosítsa, hogy milyen típusú hibákat tapasztalhat az alkalmazás.
* Rögzítse a potenciális hatásokat és nem sikerült az alkalmazás a különböző típusú hatását.
* Stratégiák azonosítása.
  

**Telepítse a szolgáltatások több példányát.** Ha az alkalmazás egy szolgáltatás egyetlen példánya függ, a meghibásodási pontot hoz létre. Üzembe helyezett példányban több növeli a rugalmasságot és méretezhetőséget is. A [Azure App Service](/azure/app-service/app-service-value-prop-what-is/), jelölje be egy [App Service-csomag](/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview/) , amelynek keretében több példányt. Az Azure Cloud Services, konfigurálja az összes, használja a szerepkörök [több példány](/azure/cloud-services/cloud-services-choose-me/#scaling-and-management). A [Azure Virtual Machines (VM)](/azure/virtual-machines/virtual-machines-windows-about/?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json), győződjön meg arról, hogy a virtuális gép architektúra tartalmaz-e egynél több virtuális gép és minden virtuális gép szerepel egy [rendelkezésre állási csoport][availability-sets].   

**Az automatikus skálázás használatával a terhelés növekedése esetén válaszolhat.** Ha az alkalmazás nincs konfigurálva a terhelés növekedése automatikus skálázáshoz, lehetséges, hogy az alkalmazás szolgáltatások sikertelen lesz, ha azok a felhasználói kérelmek legyen telített. További részletekért tekintse meg a következőket:

* Általános: [méretezési ellenőrzőlista](./scalability.md)
* Az Azure App Service: [példányszám manuális vagy automatikus méretezése][app-service-autoscale]
* A cloud Services: [automatikus méretezése egy felhőalapú szolgáltatás][cloud-service-autoscale]
* Virtuális gépek: [automatikus méretezés és virtuálisgép-méretezési csoportokban][vmss-autoscale]

**Használja a terheléselosztási kérések elosztása.** Terheléselosztás osztja el a kérelmeket az alkalmazás kifogástalan állapotú szolgáltatási példányai nem megfelelő állapotú példányokat eltávolításával a rotációból. Ha a szolgáltatás az Azure App Service vagy az Azure Cloud Services használ, már terhelésű az Ön számára. Azonban ha az alkalmazás Azure virtuális gépek, szüksége lesz a terheléselosztó üzembe helyezése. Tekintse meg a [Azure Load Balancer](/azure/load-balancer/load-balancer-overview/) kapcsolatos további részletek áttekintése.

**Az Azure Application Gateway-átjárók több példány használatára konfigurálja.** Az alkalmazás követelményeitől függően egy [Azure Application Gateway](/azure/application-gateway/application-gateway-introduction/) előfordulhat, hogy lehet jobban megfelelnek az alkalmazás szolgáltatásokra irányuló kérések elosztása. Azonban az Application Gateway szolgáltatás egyetlen példánya nem által garantált SLA-t, hogy lehetséges legyen, hogy az alkalmazás sikertelen lehet, ha az Application Gateway-példány sikertelen lesz. Több közepes vagy nagy méretű Alkalmazásátjáró példány feltételei a szolgáltatás rendelkezésre állásának garantálása érdekében kiépítése a [SLA](https://azure.microsoft.com/support/legal/sla/application-gateway/v1_0/).

**Rendelkezésre állási csoportok használata az alkalmazásrétegek esetében.** Helyezi el a-példányokat egy [rendelkezésre állási csoport] [ availability-sets] biztosít egy újabb [SLA](https://azure.microsoft.com/support/legal/sla/virtual-machines/). 

**Vegye figyelembe, hogy az alkalmazás üzembe helyezése több régióban.** Ha az alkalmazás egyetlen régióban üzembe, az esemény ritkán fordul elő a teljes régió elérhetetlenné válik, az alkalmazás még nem lesz elérhető. Lehet, hogy ez elfogadhatatlan az alkalmazás szolgáltatásiszint-szerződés feltételei szerint. Ha igen, fontolja meg az alkalmazás és a hozzá tartozó szolgáltatások telepítése több régióban. Egy többrégiós üzembe helyezés az egy aktív-aktív minta (kérelmek elosztásával aktív példányok) vagy egy aktív-passzív minta (tartja "meleg" példány számára fenntartott, abban az esetben, ha az elsődleges példány sikertelen). Azt javasoljuk, hogy több-példányok üzembe helyezésekor az alkalmazásszolgáltatások között regionális párokról. További információkért lásd: [üzleti folytonossági és vészhelyreállítási helyreállítási (BCDR): Azure párosított régiói](/azure/best-practices-availability-paired-regions).

**Az Azure Traffic Manager használatával irányíthatja a forgalmat az alkalmazás különböző régiókban.**  [Az Azure Traffic Manager] [ traffic-manager] terheléselosztás, a DNS szintjén végzi, és átirányítja a forgalmat alapján különböző régiók a [forgalom-útválasztást] [ traffic-manager-routing] metódus azt adja meg, és az alkalmazás végpontok állapotát. Nélkül Traffic Manager, pedig csak egyetlen régióban, amely korlátozza a skálázási, növeli az egyes felhasználók közel valós idejű és régióra kiterjedő szolgáltatáskimaradás esetén alkalmazás állásidőt okoz az üzembe helyezéshez.

**A terheléselosztókkal és a traffic Manager-példányok állapotadat-mintavételek tesztelése és konfigurálása.** Győződjön meg arról, hogy az állapotfigyelő logikát ellenőrzi a rendszer kritikus fontosságú részei, és megfelelően válaszol-e az állapot-mintavételei.

* Az állapot-mintavételei a [Azure Traffic Manager] [ traffic-manager] és [Azure Load Balancer] [ load-balancer] kiszolgálása egy adott funkciót. A Traffic Manager, az állapotminta határozza meg, feladatátvételt egy másik régióba. A load Balancer meghatározza, hogy e virtuális gép eltávolítása a rotációból.      
* A Traffic Manager mintavétel az állapot végponti ellenőrizni kell a kritikus fontosságú függőségek, amelyek egy régión belül vannak telepítve, és amelyek meghibásodása indíthat el egy feladatátvételt egy másik régióba.  
* A load Balancer üzemállapoti végpontját jelentse a virtuális gép állapotát. Más rétegek vagy a külső szolgáltatások nem tartalmazzák a. Ellenkező esetben hiba, amely a virtuális gép kívül történik, hogy távolítsa el a virtuális Gépet a rotációból a terheléselosztó miatt.
* Végrehajtási egészségügyi figyelés az alkalmazásban, tekintse át [állapot végponti Monitorozását végző minta](https://msdn.microsoft.com/library/dn589789.aspx).

**Külső szolgáltatások figyelésére.** Ha az alkalmazás függőségeit a harmadik féltől származó szolgáltatásokkal, határozza meg, hol és hogyan meghiúsulhat, ezek a külső szolgáltatások és milyen hatással ezekre a hibákra lesz az alkalmazás a. Egy külső szolgáltatás nem tartalmazhatnak monitorozást és diagnosztikát, ezért fontos, hogy ezek közül a meghívásához és összefüggésbe hozva azokat az alkalmazás állapotát az és a diagnosztikai naplózás egy egyedi azonosító segítségével. A monitorozási és diagnosztikai bevált gyakorlatokat további információkért lásd: [figyelési és diagnosztikai útmutató][monitoring-and-diagnostics-guidance].

**Győződjön meg arról, hogy bármely harmadik fél szolgáltatásának felhasznált biztosít SLA-t.** Ha az alkalmazás külső szolgáltatásra támaszkodik, de a harmadik féltől származó formájában, szolgáltatásiszint-szerződésben garantált rendelkezésre állási garancia biztosítja, az alkalmazás rendelkezésre állás is nem garantálható. Az SLA csak olyan jól, a legkevesebb szabad az alkalmazás-összetevő.

**Rugalmassági minták távoli műveletek végrehajtása a megfelelő.** Ha az alkalmazása függ a távoli szolgáltatások közötti kommunikációt, hajtsa végre a [tervezési minták](../patterns/category/resiliency.md) számára, mint például az átmeneti hibák kezelése [újrapróbálkozási minta][retry-pattern], és [Áramkör-megszakító minta][circuit-breaker]. 

**Amikor csak lehetséges aszinkron műveletek végrehajtására.** A szinkron műveletek is tudják kisajátítani az erőforrásokat, és egyéb műveletek blokkolható, amíg a hívó megvárja, amíg a folyamat befejezéséhez. Tervezze meg, amikor csak lehetséges aszinkron műveletek lehetővé teszik az alkalmazás egyes részei. Az aszinkron programozásba megvalósítása a C# további információkért lásd: [aszinkron programozás az async és await][asynchronous-c-sharp].

## <a name="data-management"></a>Adatkezelés

**Ismerje meg az alkalmazás-adatforrások replikációs módszereket.** Az alkalmazásadatok különböző adatforrások lesz tárolva, és a különböző rendelkezésre állási követelmények vonatkoznak. A replikációs módszereket az egyes Azure-ban, az adattárolás kiértékelése többek között [Azure Tárreplikáció](/azure/storage/storage-redundancy/) és [SQL adatbázis aktív Georeplikációt](/azure/sql-database/sql-database-geo-replication-overview/) annak biztosítására, hogy az alkalmazás adatokat követelmények teljesülnek.

**Győződjön meg arról, hogy nincs egyetlen felhasználói fiók rendelkezik-e éles és a biztonsági mentési adatokhoz való hozzáférés.** A biztonsági mentések integritása sérül, ha egy egyetlen felhasználói fiók éles és a biztonsági mentési források írási engedéllyel rendelkezik. Egy rosszindulatú felhasználó szándékosan sikerült törölni a teljes adatmennyiséget, míg az átlagos felhasználók sikerült véletlenül törlést kezdeményezni. Tervezze alkalmazását úgy korlátozhatja az egyes felhasználói fiókok engedélyeket, így csak azok a felhasználók írási hozzáférést igénylő írási hozzáféréssel rendelkezik, és csak olyan éles vagy a biztonsági mentés, de nem mindkettőt.

**A dokumentum az adatforrás feladatátvételét és feladat-visszavételt feldolgozni, és a tesztelés közben.** Abban az esetben, ha az adatforrás nem ami katasztrofálisan emberi beavatkozást kell kövesse az utasításokat a átadja a feladatokat egy új adatforrás készletét. Ha a dokumentált lépések hibákat, az operátornak nem képes sikeresen kövesse azokat, és átadja a feladatokat az erőforrás. Rendszeresen tesztelje a utasítás lépéseket annak ellenőrzéséhez, hogy azokat a következő operátor képes sikeres feladatátvétel és feladat-visszavételt az adatforrás.

**Ellenőrizze az adatok biztonsági mentése.** Rendszeresen ellenőrizze, hogy a biztonsági mentési adatok várt parancsfájl futtatásával érvényesíteni az adatok integritásának megőrzése, a sémát és a lekérdezések. Nincs kellene a biztonsági mentést, ha nem hasznos, ha szeretné visszaállítani az adatforrások pont. Jelentkezzen, és bármely inkonzisztenciákról tesz jelentést, a biztonsági mentési szolgáltatás nem állítható helyre.

**Fontolja meg egy, a georedundáns tárfiók típusa.** Egy Azure Storage-fiókban tárolt adatok mindig replikálódik helyileg. Vannak azonban van egy Tárfiók üzembe helyezésekor választhat replikációs néhány stratégiát. Válassza ki [Azure írásvédett Georedundáns Társzolgáltatási (RA-GRS)](/azure/storage/storage-redundancy/#read-access-geo-redundant-storage) védelme érdekében az alkalmazás adatait a ritka esetben, ha egy teljes régió elérhetetlenné válik.

> [!NOTE]
> A virtuális gépek esetében ne támaszkodjon kizárólag RA-GRS replikáció visszaállítása a Virtuálisgép-lemezek (VHD-fájlok). Ehelyett használjon [Azure Backup][azure-backup].   
>
>

## <a name="security"></a>Biztonság

**Az elosztott szolgáltatásmegtagadási (DDoS-) támadások elleni alkalmazásszintű védelem megvalósításához.** Azure-szolgáltatások a hálózati rétegen DDos-támadásokkal szembeni védettek. Azonban az Azure nem alkalmazásréteg támadásokkal szembeni, mert nehéz megkülönböztetni a valós felhasználói kérések a rosszindulatú felhasználók-kérelmek. "DDoS elleni védelem" című szakaszában talál további tájékoztatást az alkalmazásréteg DDoS-támadásokkal szembeni védelme [a Microsoft Azure hálózati biztonság](http://download.microsoft.com/download/C/A/3/CA3FC5C0-ECE0-4F87-BF4B-D74064A00846/AzureNetworkSecurity_v3_Feb2015.pdf) (letölthető PDF-fájl).

**Az alkalmazás-erőforrásokhoz való hozzáférést a minimális jogosultság elvének megvalósítása.** Az alkalmazás-erőforrások eléréséhez az alapértelmezett a lehető legjobban kell lennie. Adja meg egy jóváhagyási alapon magasabb szintű engedélyeket. Alapértelmezés szerint az alkalmazás-erőforrások túlzottan megengedő hozzáférést adna eredményezhet valaki véletlenül vagy szándékosan erőforrások törlése. Az Azure biztosít [szerepköralapú hozzáférés-vezérlés](/azure/active-directory/role-based-access-built-in-roles/) kezelheti a felhasználói jogosultságok, de fontos, hogy más erőforrások, például az SQL Server a saját engedélyek rendszerekkel rendelkező legalacsonyabb jogosultsági engedélyeinek ellenőrzése.

## <a name="testing"></a>Tesztelés

**Hajtsa végre a feladatátvétel és feladat-visszavétel az alkalmazás tesztelése.** Ha még nem lett teljes körűen tesztelve feladatátvétel és feladat-visszavételt, nem lehet biztos, hogy az alkalmazás a függő szolgáltatások térjen vissza szinkronizált módon a vészhelyreállítás során. Győződjön meg arról, hogy az alkalmazás függő szolgáltatások feladatátvétel és a megfelelő sorrendben sikertelen.

**Hajtsa végre a hibabeszúrás az alkalmazás tesztelése.** Az alkalmazás nem sikerül, számos különféle oka lehet, például a lejárt tanúsítványok, az erőforrásokat a virtuális gép és a tárolási hibák Erőforrásfogyás. Tesztelheti alkalmazását szimuláló vagy valódi hibák elindítása az éles környezetben, a lehető legközelebb környezetben. Például tanúsítványok törlése, mesterségesen használják a rendszer erőforrásait, vagy egy tárolási forrás törlése. Ellenőrizze az alkalmazás minden típusú hibák, önálló és kombinált helyreállítás lehetőségét is. Ellenőrizze, hogy hibák nem propagálása vagy kaszkádolás révén a rendszer.

**Futtasson teszteket használata szintetikus és a valós felhasználói adatainak, mind éles környezetben.** Tesztelési és éles megegyeznek a ritkán, ezért fontos, kék vagy zöld vagy a tesztcsoportos központi telepítés és az alkalmazás tesztelése éles környezetben. Ez lehetővé teszi, hogy az alkalmazás tesztelése a valódi terhelés alatt éles környezetben, és győződjön meg arról, teljes körűen telepítésekor várt módon működik.

## <a name="deployment"></a>Környezet

**A dokumentum a kibocsátási folyamat az alkalmazáshoz.** Nélkül részletes kibocsátási folyamat dokumentációja az operátornak előfordulhat, hogy egy rossz frissítés üzembe helyezése vagy az alkalmazás nem megfelelően konfigurálja. Egyértelműen határozza meg és dokumentálja a kibocsátási folyamat, és győződjön meg arról, hogy azt a teljes üzemeltetési csapat rendelkezésére. 

**Az alkalmazás üzembe helyezési folyamat automatizálható.** Ha a munkatársak szükség az alkalmazás manuális telepítése, emberi hiba okozhat az üzembe helyezés sikertelen lesz. 

**Tervezze meg a kibocsátási folyamat rendelkezésre állásának maximalizálása érdekében.** A kiadási folyamathoz szükséges szolgáltatások üzembe helyezése során offline állapotba, ha az alkalmazás nem lehet ismét online állapotú lesz. Használja a [kék vagy zöld](http://martinfowler.com/bliki/BlueGreenDeployment.html) vagy [canary kiadás](http://martinfowler.com/bliki/CanaryRelease.html) üzembe helyezési módszer alkalmazását az éles környezetben való üzembe helyezéséhez. Az alábbi eljárások mindkettőben éles kódban mellett a kiadás kód üzembe helyezése, így a felhasználók kiadási kód átirányíthatóak éles kódban, hogy hiba esetén.

**Jelentkezzen be, és az alkalmazás-KözpontiTelepítés naplózása.** Ha szakaszos üzembe helyezési technikák, például a kék és zöld vagy a tesztcsoportos kiadások, az alkalmazás az éles környezetben futó egynél több verziója lesz. Probléma történjen, ha fontos meghatározni, hogy az alkalmazás melyik verzióját okozza a problémát. A lehető legtöbb verzióspecifikus adatok rögzítése egy hatékony naplózás stratégia megvalósításához.

**A visszaállítási terven üzembe helyezéshez rendelkeznie.** Akkor lehet, hogy az alkalmazás központi telepítésének sikertelen volt, és miatt az alkalmazás már nem érhető el. Tervezze meg a visszaállítási folyamat lépjen vissza a legutóbbi ismert helyes verzióját, és minimalizálják az állásidőt. 

## <a name="operations"></a>Műveletek

**Gyakorlati tanácsok a monitorozási és riasztási az alkalmazásban megvalósításához.** Nem megfelelő monitoring, diagnosztika és riasztások nincs lehetőség a hibák észlelése az alkalmazásban, és riasztást küldjön olyan operátorra, javítsa ki őket. További információkért lásd: [figyelési és diagnosztikai útmutató][monitoring-and-diagnostics-guidance].

**Távoli hívás a statisztikai adatok, és az alkalmazás csapatához elérhetővé az adatokat.**  Ha nem nyomon követése és távoli hívás statisztikát valós időben, és tekintse át ezeket az adatokat egyszerű módot biztosítanak a, a üzemeltetési csapat nem fog egy azonnali betekintést az alkalmazás állapotát. És ha csak a mérték távoli átlagos hívási idő, nem fog tudni elég információ felfedése problémák által a szolgáltatásokkal. Távoli hívás mérőszámokat, például a késés, átviteli sebesség és a 99 és a 95. percentilisei hibáinak összegzése. Hajtsa végre a statisztikai elemzések elvégzésével nyújt betekintést az egyes PERCENTILIS során előforduló hibákról a metrikákra vonatkozóan.

**Nyomon követheti az átmeneti kivételek és az újrapróbálkozások számát egy megfelelő időtartamon keresztül.** Ha nem nyomon követheti és figyelheti az átmeneti kivételek és újrapróbálkozások száma idővel, lehetőség, hogy egy probléma vagy hiba sikerült szerint rejtve van az alkalmazás újrapróbálkozási logikát. Azt jelenti Ha a figyelés és naplózás csak jeleníti meg a sikeres vagy sikertelen művelet, a tény, hogy a művelet többször próbálkozik kivételek miatt kellett rejtett. A kivételek növekszik az idő múlásával trendjének azt jelzi, hogy a szolgáltatás, hogy a probléma sikertelenek lehetnek. További információkért lásd: [az újrapróbálkozási mechanizmushoz adott][retry-service-guidance].

**Bemutatjuk egy korai figyelmeztető rendszert, amely riaszt egy operátort.** Azonosítsa a fő teljesítménymutatók mutatók, az alkalmazás állapotáról, például az átmeneti kivételek és a távoli hívás késést, és állítsa be a megfelelő küszöbérték mindegyikük számára. Riasztást küld a műveletek a küszöbérték elérésekor. Állítsa be ezeket a küszöbértékeket, hogy az azonosíthatja a problémákat, mielőtt válik a kritikus fontosságú, és a egy helyreállítási válasz igényelnek.

**Győződjön meg arról, hogy a csapat több személy be van tanítva, az alkalmazás figyelése és végrehajthatja a manuális helyreállítási lépéseket.** Ha csak egyetlen operátor a csapat, akik az alkalmazás figyelése és helyreállítási lépések indíthat, a személy lesz egy meghibásodási pont. Betanítása észlelése és javítása a több egyéni felhasználók számára, és ellenőrizze, hogy mindig van legalább egy aktív, bármikor.

**Győződjön meg arról, hogy az alkalmazás nem fut be elleni [az Azure-előfizetés korlátaival](/azure/azure-subscription-service-limits/).** Az Azure-előfizetések korlátokkal rendelkeznek az egyes erőforrástípusok, például az erőforráscsoportok száma, a magok számát és a storage-fiókok száma.  Ha az alkalmazás követelményeinek meghaladja az Azure-előfizetés korlátai, hozzon létre egy másik Azure előfizetéssel, és üzembe helyezése elegendő erőforrás létezik.

**Győződjön meg arról, hogy az alkalmazás nem fut be elleni [szolgáltatási korlátok](/azure/azure-subscription-service-limits/).** Egyes Azure-szolgáltatások használati korlátokkal rendelkeznek &mdash; például a storage, a átviteli sebesség, a kapcsolatok, a kérések száma másodpercenként és egyéb mérőszámok száma korlátozza. Az alkalmazás sikertelen lesz, ha megkísérli a erőforrások meghaladják a beállított ezeket a korlátokat. Ez a szolgáltatás sávszélesség-szabályozási és a lehetséges állásidő az érintett felhasználók eredményez. Az adott szolgáltatás és a az alkalmazás követelményeitől függően gyakran elkerülheti ezeket a korlátokat (például egy másik tarifacsomag kiválasztása) vertikális felskálázásával vagy horizontális felskálázását (új erőforráspéldányok hozzáadását jelenti).  

**A kiszolgálóalkalmazás tárolási követelményei az Azure storage méretezhetőségi és teljesítménycéljai úgy tervezze meg.** Az Azure storage célja előre meghatározott méretezhetőségi és teljesítménycéljai belül működik, így tervezése az alkalmazás felhasználja az ezen célok belül. Ha ezeken a célokon túllépi az alkalmazás storage szabályozás fog tapasztalni. A probléma megoldásához, további Tárfiókok kiépítése. Ha a Tárfiók korlát együtt futtatja, további Azure-előfizetések kiépítése, majd további Tárfiókok van. További információkért lásd: [Azure Storage méretezhetőségi és Teljesítménycéljai](/azure/storage/storage-scalability-targets/).

**Válassza ki a megfelelő Virtuálisgép-méret az alkalmazáshoz.** A tényleges CPU, memória, lemez és a virtuális gépeket, éles környezetben i/o mérjük, és ellenőrizze, hogy elegendő-e a kiválasztott Virtuálisgép-méretet. Ha nem, akkor az alkalmazás felhasználói problémákat tapasztalhatnak a kapacitás, a virtuális gépek korlátaikról megközelítést. Virtuálisgép-méretek részletesen ismertetett [az Azure-beli virtuális gépek méretei](/azure/virtual-machines/virtual-machines-windows-sizes/?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

**Állapítsa meg, az alkalmazás adott számítási stabil vagy ingadozó idővel.** Ha a munkaterhelés időbeli ingadozik, használja az Azure Virtuálisgép-méretezési csoportok automatikus méretezése Virtuálisgép-példányok számát. Ellenkező esetben meg kell manuálisan növelheti vagy csökkentheti a virtuális gépek számát. További információkért lásd: a [virtuálisgép-méretezési csoportok – áttekintés](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview/).

**Válassza ki a megfelelő szolgáltatási rétegben az Azure SQL Database.** Ha az alkalmazás az Azure SQL Database, győződjön meg arról, hogy kiválasztotta a megfelelő szolgáltatási rétegben. Ha egy csomagot, amely nem tudja kezelni az alkalmazás adatbázis-tranzakciós egységek (DTU) követelmények választ, az adatok használata szabályozva lesz. A megfelelő service-csomag kiválasztásával további információkért lásd: [SQL Database beállításai és teljesítménye: Mi érhető el az egyes szolgáltatásszinteken](/azure/sql-database/sql-database-service-tiers/).

**Hozzon létre egy Azure-támogatási folytatott interakcióra szolgáló folyamat.** Ha a folyamat részleggel [az Azure-támogatás](https://azure.microsoft.com/support/plans/) van előtt, forduljon az ügyfélszolgálathoz az igények növekedésével – nincs beállítva, állásidő lesz kell hosszan tartó, a támogatási folyamat akkor nyit meg először. A folyamat ügyfélszolgálaton és a problémák továbbítása problémák kezdettől fogva az alkalmazás rugalmasság részeként tartalmazza.

**Győződjön meg arról, hogy az alkalmazás nem használja a több mint előfizetésenként storage-fiókok maximális számát.** Az Azure lehetővé teszi, hogy legfeljebb 200 tárfiók előfizetésenként. Ha az alkalmazás további tárfiókok, mint amennyi az előfizetésben jelenleg elérhető, akkor hozzon létre egy új előfizetést, és nincs további tárfiókokat létrehozni. További információkért lásd: [Azure-előfizetés és a szolgáltatások korlátozásai, kvótái és megkötései](/azure/azure-subscription-service-limits/#storage-limits).

**Győződjön meg arról, hogy az alkalmazás nem haladja meg a virtuálisgép-lemezek skálázási célértékei.** Azure IaaS virtuális gép egy több adatlemez függően számos tényező befolyásolja, többek között a virtuális gép mérete és a tárfiók típusát csatolását támogatja. Ha az alkalmazás meghaladja a virtuálisgép-lemezek skálázási célértékei, további tárfiókok üzembe, és ott a virtuálisgép-lemezek létrehozása. További információkért lásd: [Azure Storage méretezhetőségi és Teljesítménycéljai](/azure/storage/storage-scalability-targets/#scalability-targets-for-virtual-machine-disks)

## <a name="telemetry"></a>Telemetria

**Telemetriai adatok naplózása, az alkalmazás az éles környezetben való futtatásakor.** Rögzítése robusztus telemetrikus adatokat, amíg az alkalmazás az éles környezetben fut, vagy nem fog tudni diagnosztizálása a hibák okát, bár aktívan szolgálja ki, felhasználók elegendő információt. További információkért lásd: [Monitoring and Diagnostics][monitoring-and-diagnostics-guidance].

**Naplózás az aszinkron minta használatával hajtja végre.** Ha naplózási műveletek szinkron, előfordulhat, hogy blokkolja az alkalmazás kódjában. Győződjön meg arról, hogy a naplózási műveletek aszinkron műveletek vannak implementálva.

**A szolgáltatás határain túlnyúló összevetését naplóadatokat.** Egy tipikus n szintű alkalmazás esetében a felhasználó kérésére útválasztópárokon több szolgáltatás határain. Például egy felhasználó kérelme általában a webes rétegből származik, és az üzleti szint átadott és végül megőrzi az adatszinten lévő. Az összetettebb esetekhez egy felhasználói kérelem terjeszthetők a számos különböző szolgáltatásokat és adattárakat. Győződjön meg arról, hogy a naplózás rendszer utal. hívások a szolgáltatás határain túlnyúló úgy követheti nyomon a kérelmet az alkalmazásban.

## <a name="azure-resources"></a>Azure-erőforrások

**Erőforrások üzembe helyezése Azure Resource Manager-sablonok használata.** Resource Manager-sablonok megkönnyítik a PowerShell vagy az Azure CLI, ami egy további megbízható folyamatot üzembe helyezések automatizálását. További információkért lásd: [Azure Resource Manager áttekintése][resource-manager].

**Erőforrásoknak adjon kifejező nevet.** Jogosultságot ad az erőforrások adjon kifejező nevet könnyebben megtalálni egy adott erőforrást és megérteni a szerepét. További információkért lásd: [Azure-erőforrások elnevezési konvenciói](../best-practices/naming-conventions.md)

**Szerepköralapú hozzáférés-vezérlés (RBAC) használja.** Az RBAC használatával üzembe helyezett Azure-erőforrásokhoz való hozzáférés szabályozása. RBAC lehetővé teszi, hogy engedélyezési szerepköröket rendeljen a fejlesztési és üzemeltetési csapat tagjai a telepített erőforrások módosítások vagy véletlen törlés megelőzése érdekében. További információkért lásd: [hozzáférés-kezelés az Azure portal – első lépések](/azure/active-directory/role-based-access-control-what-is/)

**Használjon erőforrászárat a kritikus fontosságú erőforrások, például virtuális gépeket.** Erőforrás-zárolások megakadályozza, hogy az operátor véletlenül töröl egy erőforrást. További információkért lásd: [zárolhat erőforrásokat az Azure Resource Managerrel](/azure/azure-resource-manager/resource-group-lock-resources/)

**Válassza ki a regionális párokról.** Ha két régióban is üzembe helyeznek, régiókat azonos regionális párokból érdemes választani. Széles körű leállás esetén minden párból az egyik régió helyreállítása előnyt élvez. Egyes szolgáltatások, például a Georedundáns tárolás adja meg a replikálás automatikus, a párosított régióba. További információkért lásd: [üzleti folytonossági és vészhelyreállítási helyreállítási (BCDR): Azure párosított régiói](/azure/best-practices-availability-paired-regions)

**Erőforráscsoportok függvény és életciklus rendszerezheti.**  Általában egy erőforráscsoport tartalmaznia kell az azonos életciklussal rendelkező erőforrások. Ez megkönnyíti a központi telepítések felügyeletéhez szükséges, a tesztkörnyezetek törlését és a adhatnak hozzáférési jogokat csökkenti annak az esélyét, hogy éles környezet véletlenül törölték vagy módosították. Hozzon létre külön erőforráscsoportok éles környezetben, fejlesztési, és tesztelési környezetek. A több régióból álló üzemelő külön erőforráscsoportok minden olyan régió esetében az erőforrások üzembe. Ez megkönnyíti az ismételt üzembe helyezése több régióban anélkül, hogy befolyásolná a többi régió(k).

## <a name="next-steps"></a>További lépések

- [Rugalmasságra vonatkozó ellenőrzőlista az adott Azure-szolgáltatásokhoz](./resiliency-per-service.md)
- [Hibaállapot elemzése](../resiliency/failure-mode-analysis.md)


<!-- links -->
[app-service-autoscale]: /azure/monitoring-and-diagnostics/insights-how-to-scale/
[asynchronous-c-sharp]: /dotnet/articles/csharp/async
[availability-sets]:/azure/virtual-machines/virtual-machines-windows-manage-availability/
[azure-backup]: https://azure.microsoft.com/documentation/services/backup/
[circuit-breaker]: ../patterns/circuit-breaker.md
[cloud-service-autoscale]: /azure/cloud-services/cloud-services-how-to-scale/
[fma]: ../resiliency/failure-mode-analysis.md
[load-balancer]: /azure/load-balancer/load-balancer-overview/
[monitoring-and-diagnostics-guidance]: ../best-practices/monitoring.md
[resource-manager]: /azure/azure-resource-manager/resource-group-overview/
[retry-pattern]: ../patterns/retry.md
[retry-service-guidance]: ../best-practices/retry-service-specific.md
[traffic-manager]: /azure/traffic-manager/traffic-manager-overview/
[traffic-manager-routing]: /azure/traffic-manager/traffic-manager-routing-methods/
[vmss-autoscale]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview/
