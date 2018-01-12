---
title: "Rugalmasságra vonatkozó ellenőrzőlista"
description: "Feladatlista, amely útmutatást hibatűrési szempontok a tervezés közben."
author: petertaylor9999
ms.date: 01/10/2018
ms.custom: resiliency, checklist
ms.openlocfilehash: 66ff802c1f7b35db147ffe4279982c827570c3c1
ms.sourcegitcommit: 3d6dba524cc7661740bdbaf43870de7728d60a01
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/11/2018
---
# <a name="resiliency-checklist"></a>Rugalmasságra vonatkozó ellenőrzőlista

Rugalmasság azon képessége, a rendszer helyreállni hibák után, és továbbra is működik, és az egyik a [szoftver minőségű oszlopok](../guide/pillars.md). Tervezése, és esetleg előforduló függően különböző kiküszöböléséhez tervezése a rugalmasságot az alkalmazás szükséges. Az alábbi ellenőrzőlista segítségével ellenőrizze az alkalmazás architektúrák rugalmassági szempontból. 

## <a name="requirements"></a>Követelmények

**Az ügyfél rendelkezésre állási követelményeinek meghatározását.** Az ügyfél összetevők rendelkezésre állását követelményekkel rendelkezik az alkalmazásban, és ez milyen hatással van az alkalmazás tervét. Az ügyfél megállapodás lekérése az alkalmazás minden egyes adatra rendelkezésre állási céljai, ellenkező esetben a Tervező előfordulhat, hogy nem felelnek meg az ügyfél követelményeinek. További információkért lásd: [a rugalmasság követelmények meghatározását](../resiliency/index.md#defining-your-resiliency-requirements).

## <a name="application-design"></a>Alkalmazás tervezése

**Hajtsa végre egy mód hibaelemzés (FMA) az alkalmazáshoz.** FMA az a folyamat egy alkalmazásba, a tervezési fázis korai szakaszában rugalmassági készítéséhez. További információkért lásd: [mód hibaelemzés][fma]. Egy FMA céljai a következők:  

* Határozza meg, milyen típusú hibákat tapasztalhat az alkalmazás.
* Rögzítse a lehetséges hatások és hiba a kérelem a különböző típusú hatását.
* Helyreállítási stratégiák meghatározása.
  

**Szolgáltatások több példányának telepítése.** Ha az alkalmazás attól függ, hogy a szolgáltatás egyetlen példányát, a hibaérzékeny pontot hoz létre. Kiépítés több példány növeli a rugalmasságot és a méretezhetőséget. A [Azure App Service](/azure/app-service/app-service-value-prop-what-is/), jelölje be egy [App Service-csomag](/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview/) , amely több példányt biztosít. Azure-szolgáltatásokhoz, konfigurálhatja a szerepkörök használandó [több példány](/azure/cloud-services/cloud-services-choose-me/#scaling-and-management). A [Azure virtuális gépek (VM)](/azure/virtual-machines/virtual-machines-windows-about/?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json), győződjön meg arról, hogy a virtuális gép architektúrája tartalmazza az egynél több virtuális gép és az egyes virtuális gépek szerepel egy [rendelkezésre állási csoport][availability-sets].   

**Használja az automatikus skálázás válaszolni a terhelés növekszik.** Az alkalmazás automatikusan, a betöltés növekedése horizontális nincs konfigurálva, akkor lehetséges, hogy az alkalmazás szolgáltatások sikertelen lesz, ha azok felhasználó által érintett válnak telített. További részletekért olvassa el a következőket:

* Általános: [méretezhetőség ellenőrzőlista](./scalability.md)
* Az Azure App Service: [méretezése példányszám manuális vagy automatikus][app-service-autoscale]
* Felhőszolgáltatások: [hogyan automatikus méretezési egy felhőalapú szolgáltatás][cloud-service-autoscale]
* Virtuális gépek: [automatikus skálázás és a virtuális gép skálázása beállítása][vmss-autoscale]

**Terheléselosztás használatával terjesztheti a kérelmeket.** Szétosztja az alkalmazás kérelmek kifogástalan szolgáltatáspéldány Elforgatás sérült példányok eltávolításával. Ha a szolgáltatás az Azure App Service- vagy Azure Cloud Services használ, már terhelésű meg. Azonban ha az alkalmazás Azure virtuális gépeken, szüksége lesz egy terhelés-kiegyenlítő kiépítéséhez. Tekintse meg a [Azure Load Balancer](/azure/load-balancer/load-balancer-overview/) áttekintése További részleteket.

**Azure Alkalmazásátjárót több példány használatára konfigurálja.** Az alkalmazás követelményeitől függően egy [Azure Application Gateway](/azure/application-gateway/application-gateway-introduction/) is lehet jobban megfelelnek az alkalmazás szolgáltatások kérések elosztása. Azonban a Application Gateway szolgáltatás egyetlen példánya nem igazolt szolgáltatásiszint-szerződésben garantált ezért lehetséges, hogy az alkalmazás meghiúsulhat, ha az alkalmazás átjárópéldány sikertelen. Egynél több közepes vagy nagyobb alkalmazás átjárópéldányt be kell állítani az értelmében a szolgáltatás rendelkezésre állásának biztosítása kiépíteni a [SLA](https://azure.microsoft.com/support/legal/sla/application-gateway/v1_0/).

**Rendelkezésre állási készletek használata minden egyes alkalmazás szinten.** Helyezi el a példány egy [rendelkezésre állási csoport] [ availability-sets] biztosít egy magasabb [SLA](https://azure.microsoft.com/support/legal/sla/virtual-machines/). 

**Vegye figyelembe, hogy az alkalmazás telepítéséhez a különféle régiókban.** Ha az alkalmazás központi telepítése egy régiót, a ritka esetben az egész régió nem érhető el, az alkalmazás még nem lesz elérhető. Ennek az lehet az alkalmazás szolgáltatásiszint-szerződés feltételei szerint nem fogadható el. Ha igen, fontolja meg az alkalmazás és a szolgáltatások központi telepítése a különféle régiókban. A több területi központi telepítés a egy aktív-aktív mintát (a kérelmek osztja a több aktív példányok között) vagy egy aktív-passzív mintát (megőrzi a "meleg" példánya található tartalék, abban az esetben, ha az elsődleges példány nem). Azt javasoljuk, hogy az alkalmazás szolgáltatások több példányt telepít regionális párok között. További információkért lásd: [üzleti folytonossági és vészhelyreállítási helyreállítási (BCDR): Azure-régiókat párosítva](/azure/best-practices-availability-paired-regions).

**Azure Traffic Manager segítségével különböző régiókban az alkalmazás forgalmat továbbítja.**  [Az Azure Traffic Manager] [ traffic-manager] hajtja végre a DNS-szinten terheléselosztási és más régiókba alapján irányítja a forgalmat a [a forgalom útválasztásához] [ traffic-manager-routing] Adja meg, hogy metódus, és az alkalmazás-végpontok állapotát. Nélkül Forgalomkezelő egy régiót a központi telepítés, amely behatárolja méretezési, növeli az egyes felhasználók késést, és hatására az alkalmazás leállás esetén a régió kiterjedő szolgáltatás szüneteltetése.

**A terheléselosztókkal és a forgalom kezelők állapotfigyelő mintavételt tesztelése és konfigurálása.** Győződjön meg arról, hogy a health logika ellenőrzi a rendszer kritikus részét, és állapotteljesítmény megfelelően válaszol.

* Az állapot-mintavételi csomagjai a [Azure Traffic Manager] [ traffic-manager] és [Azure Load Balancer] [ load-balancer] egy adott funkció szolgál. A Traffic Manager, a állapotmintáihoz határozza meg, hogy egy másik régióban feladatátvételt. A terheléselosztóhoz azt határozza meg a virtuális gép eltávolítása elforgatási.      
* A rendszerállapot-végpontot egy Traffic Manager mintavétel ellenőrizze a kritikus függőségeket, amely ugyanabban a régióban belül vannak telepítve, és amelyek meghibásodása indíthat el egy feladatátvételt egy másik régióban.  
* A terheléselosztóhoz az egészségügyi végpont jelenteniük kell a virtuális gép állapotát. Más rétegekkel vagy a külső szolgáltatások nem tartalmazzák. Ellenkező esetben kívül a virtuális gép előforduló hibát okoz, elforgatás a virtuális gép eltávolítása a terheléselosztó.
* Útmutató a végrehajtási állapot az alkalmazás figyelése: [állapotfigyelő végpont figyelési mintát](https://msdn.microsoft.com/library/dn589789.aspx).

**Külső szolgáltatások figyelésére.** Ha az alkalmazás függőségeit a harmadik féltől származó szolgáltatással, határozza meg, hol, hogyan külső szolgáltatásokról sikertelen lehet, és milyen hatással, ezek a hibák az alkalmazás rendelkezik majd. Egy külső szolgáltatás nem tartalmazhatnak figyelése és diagnosztikai, ezért fontos, hogy azok a indítások jelentkezzen és kivizsgált azokat az alkalmazás állapotának és diagnosztikai naplózás egyedi azonosító segítségével. A figyelési és diagnosztika bevált gyakorlatok további információkért lásd: [megfigyelési és diagnosztikai útmutatást][monitoring-and-diagnostics-guidance].

**Győződjön meg arról, hogy bármely külső szolgáltatás felhasznált biztosít egy SLA-t.** Ha az alkalmazás olyan külső szolgáltatástól függ, de a harmadik féltől származó biztosít a rendelkezésre állási szolgáltatásiszint-szerződésben garantált formájában nem garantálja, az alkalmazás is nem lehet biztosítani. Az SLA-t csak akkor az alkalmazás a legkevesebb szabad összetevő megegyezik.

**Rugalmasság minták távoli műveletek végrehajtása, megfelelő.** Ha az alkalmazás távoli szolgáltatások közötti kommunikáció függ, hajtsa végre az átmeneti hibák kapcsolatos kialakítási minta, mint [újra mintát][retry-pattern], és [áramköri megszakító Minta][circuit-breaker]. További információkért lásd: [rugalmassági stratégiák](../resiliency/index.md#resiliency-strategies).

**Amikor csak lehetséges aszinkron műveletek végrehajtására.** Szinkron műveletek se sajátíthassa ki az erőforrásokat, és más műveletek blokkolható, amíg a hívó megvárja-e a folyamat befejezéséhez. Tervezze meg az alkalmazás számára, amikor csak lehetséges aszinkron műveletek engedélyezése részét képező összes. Aszinkron programozás megvalósításához a C# további információkért lásd: [aszinkron programozás az async és await][asynchronous-c-sharp].

## <a name="data-management"></a>Adatkezelés

**Ismerje meg az alkalmazás-adatforrások replikációs módszereket.** Az alkalmazásadatok különböző forrásokból tárolódnak, és másik rendelkezésre állási követelmények vonatkoznak. Értékelje ki a replikációs módszereket az egyes Azure, az adattárolás beleértve [Azure Storage replikációs](/azure/storage/storage-redundancy/) és [SQL adatbázis aktív Georeplikáció](/azure/sql-database/sql-database-geo-replication-overview/) annak érdekében, hogy az alkalmazás adatokat követelmények teljesülnek.

**Győződjön meg arról, hogy nincs egy felhasználói fiók rendelkezik-e éles és a biztonsági mentés adatokhoz való hozzáférés.** A biztonsági mentések akkor biztonsága sérül, ha egy egyetlen felhasználói fiókkal is a termelési, és a biztonsági mentési források írási engedéllyel rendelkezik. Egy rosszindulatú felhasználó szándékosan sikerült törölni az adatokat, míg az átlagos felhasználók tudta véletlenül törölni. Tervezze meg az alkalmazás számára korlátozza az engedélyeket az egyes felhasználói fiókok, hogy csak az írási hozzáférést igénylő felhasználók írási hozzáféréssel rendelkezik, és csak a termelési vagy biztonsági mentés, de soha sem egyszerre mindkettőre.

**A dokumentum az adatforrás feladatátvétele és a feladat-visszavételt feldolgozni, és tesztelik azt.** Abban az esetben, ha az adatforrás nem catastrophically emberi beavatkozást kell kövesse az utasításokat a feladatátvételt egy új adatforrás készlete. Ha dokumentált lépéseiben hibák, operátor nem lesz képes sikeresen betartásuk és az erőforrás a feladatátvétel. Rendszeresen tesztelniük a utasítás lépéseket annak ellenőrzéséhez, hogy azokat a következő operátor sikeres feladatátvételt, és a feladat-visszavételt a adatforrás képes.

**Ellenőrizze az adatok biztonsági mentése.** Rendszeresen győződjön meg arról, hogy a biztonsági mentési adatokat várt parancsfájl futtatásával érvényesíteni az adatok integritását, a séma és a lekérdezések. Nincs rendelkezik egy biztonsági mentés, ha még nincs visszaállítása az adatforrások hasznos pont. Naplófájl, és jelentést esetlegesen bekövetkező inkonzisztencia, így javítható a biztonsági mentési szolgáltatás.

**Érdemes lehet georedundáns tárolás fiók típusa.** Egy Azure Storage-fiókban tárolt adatok helyben mindig replikálódik. Van azonban több replikálási gyakorlata választhat, ha a Storage-fiók ki van építve. Válassza ki [Azure-írásvédett Georedundáns redundáns tárolás (RA-GRS)](/azure/storage/storage-redundancy/#read-access-geo-redundant-storage) szemben a ritka esetben alkalmazás adatok védelme, amikor egy teljes terület nem érhető el.

> [!NOTE]
> Virtuális gépekhez ne használja az RA-GRS replikációs visszaállítani a virtuális gépek lemezei (VHD-fájlokat). Ehelyett használjon [Azure biztonsági mentés][azure-backup].   
>
>

## <a name="security"></a>Biztonság

**Alkalmazás szintű védelmet biztosít a elosztott szolgáltatásmegtagadásos (DDoS-) támadások megvalósításához.** Azure-szolgáltatások a hálózati rétegben DDos-támadások elleni védelmét. Azonban Azure nem lánctámadások elleni védelem érdekében alkalmazásréteg, nehéz a rosszindulatú felhasználók a kérelmek igaz felhasználói kérelmek megkülönböztetésére. Hogyan alkalmazásszintű DDoS-támadások elleni védelem érdekében további információkért lásd: a "DDoS elleni védelme" szakaszában [Microsoft Azure hálózati biztonság](http://download.microsoft.com/download/C/A/3/CA3FC5C0-ECE0-4F87-BF4B-D74064A00846/AzureNetworkSecurity_v3_Feb2015.pdf) (PDF-fájl letöltése).

**Valósítja meg az alkalmazás erőforrások elérése érdekében a legalacsonyabb jogosultsági szint elvét.** Az alapértelmezett, az alkalmazás erőforrások elérése érdekében a lehető legjobban korlátozza kell lennie. Engedélyek magasabb szintű jóváhagyási alapon. Alapértelmezés szerint az alkalmazás erőforrásokhoz túlságosan megengedő hozzáférést engedélyező eredményezhet valaki véletlenül vagy szándékosan erőforrások törlése. Azure biztosít [szerepköralapú hozzáférés-vezérlés](/azure/active-directory/role-based-access-built-in-roles/) kezelheti a felhasználói jogosultságok, azonban fontos, hogy más erőforrások, amelyek a saját engedélyek rendszerek például az SQL Server legalacsonyabb jogosultsági engedélyeinek ellenőrzése.

## <a name="testing"></a>Tesztelés

**Feladatátvevő és az alkalmazás-tesztelési feladat-visszavétel végrehajtására.** Ha még nem teljes körűen tesztelve feladatátvétel és a feladat-visszavétel, nem lehet biztos, hogy az alkalmazás a függő szolgáltatások helyreállnak szinkronizált módon katasztrófa utáni helyreállítás során. Győződjön meg arról, hogy az alkalmazás függő szolgáltatások feladatátvétel és a megfelelő sorrendben sikertelen.

**Hajtsa végre a hiba-injektálás az alkalmazás tesztelése.** Az alkalmazás számos különféle oka, például a tanúsítvány lejárta, vagy a virtuális gépek tárhelyének a hibáit rendszererőforrás kimerülése meghiúsulhat. Az alkalmazás teszteléséhez szimulálva vagy kiváltó hibák valós üzemi környezetben, a lehető legközelebb környezetben. Például tanúsítványok törlése, mesterségesen erőforrást a rendszer vagy egy tárolási forrás törlése. Ellenőrizze az alkalmazás képes tudja elhárítani az összes típusú hibák, önmagában, és együtt. Ellenőrizze, hogy a hibák nem propagálása vagy egymásra épülő keresztül a rendszer.

**Futtasson teszteket üzemben mindkét szintetikus és valós felhasználói adatok használatával.** Vizsgálat és éles azonosak ritkán, ezért fontos kék/zöld vagy Kanári központi és az alkalmazás tesztelése éles üzemben. Ez lehetővé teszi az alkalmazás tesztelése éles üzemben valós terhelés, és teljesen telepítésekor várt módon működik.

## <a name="deployment"></a>Környezet

**A dokumentum a kiadási folyamat az alkalmazáshoz.** Részletes kibocsátási folyamat dokumentációja, nélkül operátor lehet, hogy rossz frissítés központi telepítéséhez vagy az alkalmazás nem megfelelően konfigurálja. Egyértelműen határozza meg és dokumentálja a kiadási folyamat, és győződjön meg arról, hogy a teljes operations csapat rendelkezésére. 

**Az alkalmazás központi telepítési folyamat automatizálása.** Ha a műveleti személyzet kell manuálisan telepíteni az alkalmazást, emberi tévedések a központi telepítés meghiúsulását okozhatják. 

**Tervezze meg a kiadási folyamat rendelkezésre állásának maximalizálását.** A kiadási folyamathoz szükséges szolgáltatások üzembe helyezése során offline állapotba, ha az alkalmazás nem lehet ismét online állapotú lesz. Használja a [kék/zöld](http://martinfowler.com/bliki/BlueGreenDeployment.html) vagy [Kanári kiadás](http://martinfowler.com/bliki/CanaryRelease.html) telepítési technika az éles az alkalmazás telepítése. Az alábbi eljárások is tartalmaz, amely telepítése éles kódban mellett a kiadás kódot, így a felhasználók kiadás kód átirányíthatók éles kódban meghibásodása.

**Naplófájl, és az alkalmazások központi telepítésének naplózására.** Ha előkészített központi telepítési módszerek, például a kék/zöld vagy Kanári kiadások éles környezetben az alkalmazást egynél több verziója lesz. Probléma jöjjön létre, ha fontos meghatározni az alkalmazás melyik verzióját okozza a problémát. A lehető legtöbb verzióspecifikus információkat robusztus naplózási stratégia megvalósításához.

**A visszaállítási terven központi telepítés rendelkezik.** Akkor lehet, hogy az alkalmazások központi telepítésének sikertelen volt, és miatt az alkalmazás már nem érhető el. Tervezze meg a visszaállítási folyamat lépjen vissza a legutóbbi ismert helyes verzióját, és minimalizálják az állásidőt. 

## <a name="operations"></a>Műveletek

**Gyakorlati tanácsok a figyelés és riasztás az alkalmazás megvalósításához.** Nem megfelelő figyelési, diagnosztikai és riasztások nincs mód, az alkalmazás észlelésére, és javítsa ki az operátor értesítést. További információkért lásd: [megfigyelési és diagnosztikai útmutatást][monitoring-and-diagnostics-guidance].

**Távoli hívás a statisztikai adatok és információk számára elérhetővé tenni az alkalmazás csapatához.**  Ha nem nyomon követése és távoli hívás statisztikát valós időben, és tekintse át ezt az információt könnyű megoldást biztosítson a, a műveleti csapata nem fog egy azonnali képet kaphat az alkalmazás állapotát. Ha csak mérték átlagos távoli híváskor, akkor nem kell elegendő információt, hogy láthatóvá váljon állít ki a szolgáltatásban. Például a késés, az átviteli sebesség és a hibák 99 és 95 százalékos érték a távoli hívás metrikák foglalják össze. Hajtsa végre statisztikai elemzések a mérni kívánt fedik le az egyes PERCENTILIS során előforduló hibákról.

**Nyomon követheti a átmeneti kivételeket és az újrapróbálkozások számát egy megfelelő időkeretre keresztül.** Ha nem nyomon követése és átmeneti kivételek figyelése és az újrapróbálkozások adott idő alatt, akkor lehetséges, hogy egy probléma vagy sikertelen elrejthető, az alkalmazás újrapróbálkozási logika. Ez azt jelenti, hogy a figyelés és naplózás csak sikeres vagy sikertelen volt egy művelet látható, ha a tényt, hogy a művelet több alkalommal ismételhető kivételek miatt volt-e rejtett. A tendencia növekvő kivételek időbeli azt jelzi, hogy a szolgáltatás hibájáról sikertelenek lehetnek. További információkért lásd: [ismételje meg a szolgáltatás konkrét útmutatást][retry-service-guidance].

**Egy korai figyelmeztető rendszert, amely riasztást küld a kezelőnek megvalósításához.** Azonosítsa a teljesítmény a az alkalmazás állapotát, például az átmeneti kivételeket és a távoli utaló késés hívja, és állítsa be a megfelelő küszöbérték esetében. Riasztás küldése műveletek a küszöbérték elérésekor. Állítsa be ezeket a küszöbértékeket, hogy ahhoz, hogy kritikus válik, és válaszolni helyreállítási problémák azonosításához.

**Győződjön meg arról, hogy a csoport több személy be van tanítva, az alkalmazás figyelése, és végezze el a manuális helyreállítási lépéseket.** Ha csak egyetlen operátor a figyelheti az alkalmazást, és helyreállítási lépések indítsa a csoport, az adott személy a hibaérzékeny pontok kialakulását válik. Több személyt is alkalmaznia észlelése és a helyreállítási betanítása, és győződjön meg arról, hogy mindig van legalább egy aktív bármikor.

**Győződjön meg arról, hogy az alkalmazás nem futtatható mentése elleni [Azure-előfizetésre vonatkozó korlátok](/azure/azure-subscription-service-limits/).** Azure-előfizetések korlátokkal rendelkeznek az adott erőforrástípusokra, például az erőforrás-csoportok száma, a magok száma és a storage-fiókok száma.  Ha az alkalmazás követelményeinek haladhatja meg az Azure-előfizetésre vonatkozó korlátok, hozzon létre egy másik Azure előfizetés és a kiépítés elegendő erőforrással van.

**Győződjön meg arról, hogy az alkalmazás nem futtatható mentése elleni [/ szolgáltatásra vonatkozó korlátozások](/azure/azure-subscription-service-limits/).** Az egyes Azure-szolgáltatások fogyasztás korlátokkal rendelkeznek &mdash; például korlátozza a tárolási, átviteli, kapcsolatok, a kérelmek másodpercenkénti és más metrikákkal száma. A kérelem sikertelen lesz, ha megkísérli meghaladó erőforrások használatára. Ennek eredményeképpen az érintett felhasználók sávszélesség-szabályozási és lehetséges állásideje. Attól függően, hogy az adott szolgáltatás és alkalmazás igényeinek, gyakran elkerülhető, ezek a korlátozások (például egy másik tarifacsomag kiválasztása) vertikális felskálázásával vagy kiterjesztése (új példányok hozzáadása).  

**Tervezze meg a kiszolgálóalkalmazás tárolási követelményei az Azure storage méretezhetőségi és Teljesítménycélok alá.** Az Azure storage célja, hogy az előre definiált méretezhetőségi és Teljesítménycélok belül működik, így tervezése az alkalmazás-tárolók ezen célok belül. Ha ezeken a célokon túllépi az alkalmazáshoz fog tapasztalni tárolási sávszélesség-szabályozás. A javítás érdekében további Tárfiókok kiépítéséhez. Ha a Tárfiók korlátot találkoznak futtatja, további Azure-előfizetések kiépíteni, és azt követően a további Tárfiókok nem. További információkért lásd: [Azure Storage méretezhetőségi és teljesítménycéloknak](/azure/storage/storage-scalability-targets/).

**Válassza ki az alkalmazást a megfelelő Virtuálisgép-méretet.** A tényleges Processzor, memória, lemez és a virtuális gépek éles környezetben i/o-mérjük, és ellenőrizze, hogy elegendő-e a kijelölt Virtuálisgép-méretet. Ha nem, az alkalmazás problémákba ütközhetnek, kapacitás, a virtuális gépek megközelíti a határértékeket. Részletesen ismerteti a Virtuálisgép-méretek [az Azure virtuális gépek méretei](/azure/virtual-machines/virtual-machines-windows-sizes/?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

**Annak megállapítása, hogy az alkalmazás terhelés állandó vagy ingadozó adott idő alatt.** Ha a terhelést időbeli ingadozik, használja az Azure Virtuálisgép-méretezési állítja be automatikus skálázási Virtuálisgép-példányok száma. Ellenkező esetben kell manuálisan növeli vagy csökkenti a virtuális gépek számát. További információkért lásd: a [virtuális gépek méretezési készletek áttekintése](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview/).

**Válassza ki a megfelelő szolgáltatásréteg az Azure SQL Database.** Ha az alkalmazás az Azure SQL Database, győződjön meg arról, hogy megadta-e a megfelelő szolgáltatási rétegben. Ha olyan réteghez, amely nem tudja kezelni az alkalmazás adatbázis-tranzakciós egységek (DTU) követelmények választja, az adatok felhasználásával halmozódni fog. A megfelelő service-csomag kiválasztásával további információkért lásd: [SQL Database beállításai és teljesítménye: az egyes szolgáltatásszinteken elérhető](/azure/sql-database/sql-database-service-tiers/).

**Hozzon létre egy folyamatot, az Azure támogatási való interakció.** Ha a folyamat való kapcsolódáshoz [az Azure támogatási](https://azure.microsoft.com/support/plans/) van nincs beállítva ahhoz, hogy forduljon az ügyfélszolgálathoz igény szerint növelhesse, állásidő lesz kell hosszabbítani, a támogatási folyamat valójában az első alkalommal. A folyamat, lépjen kapcsolatba az ügyfélszolgálattal, és problémák növekvő kezdettől fogva az alkalmazás rugalmassági részeként tartalmazza.

**Győződjön meg arról, hogy az alkalmazás nem használja a több mint előfizetésenként storage-fiókok maximális számát.** Azure lehetővé teszi, hogy legfeljebb 200 tárfiókok előfizetésenként. Ha az alkalmazás további tárfiókokat, mint amennyi az előfizetésben jelenleg elérhető, akkor hozzon létre egy új előfizetést, és nincs további storage-fiókok létrehozása. További információkért lásd: [Azure-előfizetés és szolgáltatási korlátok, kvóták és megkötések](/azure/azure-subscription-service-limits/#storage-limits).

**Győződjön meg arról, hogy az alkalmazás nem haladja meg a virtuális gépek lemezeit a méretezhetőségi célok.** Az Azure infrastruktúra-szolgáltatási virtuális gép támogatja az adatlemezek számos tényező, többek között a Virtuálisgép-méretet és a tárfiók típusától függően számos csatolni. Ha az alkalmazás meghaladja a virtuálisgép-lemezeket a méretezhetőségi célok, további tárfiókok kiépíteni, és hozzon létre a virtuális gép lemezeinek van. További információkért lásd: [Azure Storage méretezhetőségi és teljesítménycéloknak](/azure/storage/storage-scalability-targets/#scalability-targets-for-virtual-machine-disks)

## <a name="telemetry"></a>Telemetria

**Jelentkezzen telemetriai adatokat, az alkalmazás az éles környezetben futása közben.** Robusztus telemetriai adatainak rögzítésére során az alkalmazás az éles környezetben fut, vagy Ön nem rendelkezik elegendő információt diagnosztizálásához problémák okait, amíg az aktívan van szolgál a felhasználók. További információkért lásd: [megfigyelési és diagnosztikai][monitoring-and-diagnostics-guidance].

**Egy aszinkron mintát használ naplózási megvalósításához.** Ha a naplózási műveletek szinkron, előfordulhat, hogy letiltják a az alkalmazás kódjában. Győződjön meg arról, hogy a naplózási műveletek aszinkron műveletek megvalósítása.

**Naplóadatok összefüggéseket szolgáltatás határain túlnyúló.** A felhasználó kérésére egy tipikus n szintű alkalmazást, előfordulhat, hogy haladnak át a több szolgáltatás határokat. Például a felhasználó kérésére általában származik, a webes réteg átadott az üzleti szint és végül őrzi meg az adatréteg. Az összetettebb forgatókönyveket a felhasználó kérésére számos különböző szolgáltatásaikat és adataikat áruházak osztható ki. Győződjön meg arról, hogy a rendszer ad eredményül hívások szolgáltatás határokon keresztül történő, a kérelem követheti nyomon az alkalmazás teljes.

## <a name="azure-resources"></a>Azure-erőforrások

**Erőforrások kiépítése Azure Resource Manager-sablonok használatára.** Resource Manager-sablonok megkönnyítik a PowerShell vagy az Azure parancssori felület, amely egy megbízható folyamatot központi telepítések automatizálásához. További információkért lásd: [Azure Resource Manager áttekintése][resource-manager].

**Adjon meg erőforrások beszédes nevet.** Jogosultságot ad az erőforrások kifejező nevet megkönnyíti egy adott erőforrás található, és megismerheti a szerepkör. További információkért lásd: [elnevezési konvenciói Azure-erőforrások](../best-practices/naming-conventions.md)

**Szerepköralapú hozzáférés-vezérlést (RBAC) használata.** Az RBAC segítségével az üzembe helyezett Azure erőforrásokhoz való hozzáférés szabályozása. Az RBAC lehetővé teszi az engedélyezési szerepkörök hozzárendelése a DevOps csapat tagjai ahhoz, hogy a véletlen törlés vagy a módosításokat a telepített erőforrások. További információkért lásd: [hozzáféréskezelést az Azure-portálon az első lépései](/azure/active-directory/role-based-access-control-what-is/)

**Kritikus erőforrásokat, például virtuális gépek erőforrás zárolások használja.** Erőforrás zárolások megakadályozhatják, hogy a kezelőnek véletlenül töröl egy erőforrást. További információkért lásd: [erőforrások az Azure Resource Manager zárolása](/azure/azure-resource-manager/resource-group-lock-resources/)

**Válassza ki a regionális párokat.** Két régió telepítésekor választhat ugyanarra a regionális két régióban. Széles körű leállás esetén minden párból az egyik régió helyreállítása előnyt élvez. Egyes szolgáltatások, például a Georedundáns tárolás párosított régió automatikus replikációs adja meg. További információkért lásd: [üzleti folytonossági és vészhelyreállítási helyreállítási (BCDR): Azure párosítva régiók](/azure/best-practices-availability-paired-regions)

**Erőforráscsoportok függvény és életciklus rendezhetők.**  Általában egy erőforráscsoportot az azonos életciklussal rendelkező erőforrásokat kell tartalmaznia. Így könnyebben központi telepítések felügyeletéhez szükséges, törölje a próbatelepítések, és rendelje hozzá a hozzáférési jogok, így csökkentve az esélye, éles környezet véletlenül törölték vagy módosították. Termelési környezetben, fejlesztési, külön erőforráscsoportokat létrehozni, és tesztelési környezetben. Több régióban üzembe helyezése esetén külön erőforráscsoportok helyezték erőforrások mindegyik régióhoz. Így könnyebben egy régió tartozik újratelepíteni az nem befolyásolja a többi régió(k) esetében.

## <a name="azure-services"></a>Azure-szolgáltatások
A következő ellenőrzőlista elemek az Azure-ban meghatározott szolgáltatásokat vonatkoznak.

- [APP SERVICE](#app-service)
- [Application Gateway](#application-gateway)
- [Cosmos DB](#cosmos-db)
- [Redis Cache](#redis-cache)
- [Search](#search)
- [Storage](#storage)
- [SQL Database](#sql-database)
- [A virtuális gépen futó SQL Server](#sql-server-running-in-a-vm)
- [Traffic Manager](#traffic-manager)
- [Virtuális gépek](#virtual-machines)
- [Virtual Network](#virtual-network)

### <a name="app-service"></a>App Service

**Standard vagy prémium szintű csomagot használja.** Ezek a rétegek átmeneti üzembe helyezési ponti támogatja, és automatikus biztonsági mentés. További információkért lásd: [Azure App Service-csomagok részletes áttekintése](/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview/)

**Kerülje a skálázás felfelé vagy lefelé.** Ehelyett válassza ki a réteg és a példány mérete, hogy alkalmazkodjanak a teljesítménykövetelményekhez szokásos terhelés alatt, majd [horizontális felskálázás](/azure/app-service-web/web-sites-scale/) a példányok adatforgalma változásainak kezeléséhez. Skálázás felfelé és lefelé indíthatnak újra kell indítani az alkalmazást.  

**Adattár konfigurálása alkalmazás-beállításként.** Alkalmazásbeállítások segítségével Alkalmazásbeállítások konfigurációs beállításokat tartalmaz. Adja meg a beállításokat a Resource Manager-sablonok vagy a powershellel, így azokat az automatikus központi telepítésének részeként, / folyamat, amely megbízhatóbb frissítése. További információkért lásd: [webes alkalmazások konfigurálása az Azure App Service](/azure/app-service-web/web-sites-configure/).

**Hozzon létre külön App Service-csomagokról éles és tesztelési.** Ne használjon üzembe helyezési ponti az éles telepítési a teszteléshez.  Minden alkalmazás belül az azonos App Service-csomag az ugyanazon Virtuálisgép-példány megosztása. Éles üzemi pontjának és próbatelepítést be a csomagot, negatívan befolyásolhatja az éles környezet. Például terhelés tesztek csökkentheti a élő munkakörnyezeti helyet. Tegyen próbatelepítést egy külön tervbe, hogy ezeket elszigeteli éles verzióját.  

**A webes API-k külön webalkalmazások.** Ha a megoldás egy előtér-webkiszolgáló és a webes API-k is rendelkezik, fontolja meg, decomposing őket külön App Service-alkalmazásokba. Ez a kialakítás megkönnyíti a megoldás felbontani a munkaterhelés szerint. Futtathatja a web app és az API-t az önálló App Service-csomagokról, így ezek egymástól függetlenül lehet méretezni. Ha nincs szüksége ekkora szintű méretezhetőséget, először telepítheti az alkalmazások azokat a csomagot, és áthelyezi őket külön tervek később, ha szükséges.

**Ne használja az alkalmazás biztonsági mentési szolgáltatás Azure SQL-adatbázisok biztonsági mentése.** Ehelyett használjon [SQL-adatbázis automatikus biztonsági mentés][sql-backup]. App Service biztonsági mentés az adatbázist egy SQL .bacpac fájlba, amely költségek dtu-k exportálja.  

**Telepíthet egy átmeneti helyet.** Az átmeneti egy üzembe helyezési tárhely létrehozása. Alkalmazás frissítések telepítése az átmeneti helyet, és a telepítés előtt csere az éles környezetben. Ez csökkenti az éles környezetben egy hibás frissítés esélyét. Emellett biztosítja, hogy a példányainak Mielőtt éles környezetben felcserélés folyamatban vannak tárolóhelyspecifikus. Számos alkalmazás jelentős melegítési és cold-a kezdési idő rendelkeznek. További információkért lásd: [átmeneti környezet az Azure App Service web Apps beállítása](/azure/app-service-web/web-sites-staged-publishing/).

**Egy üzembe helyezési tárhely ahhoz, hogy a legutóbbi helyes (LKG) központi telepítés létrehozása.** Üzemi környezetben telepít egy frissítést, ha az előző éles környezet áthelyezi a LKG tárolóhelye. Így könnyebben rossz üzemelő példány visszaállítása. Ha később felderíteni a probléma, gyorsan visszatérhet a LKG verziót. További információkért lásd: [alapvető webalkalmazás](../reference-architectures/app-service-web-app/basic-web-app.md).

**Diagnosztikai naplózás engedélyezése**, beleértve a alkalmazásnaplózás és web server naplózása. Naplózási figyelése és diagnosztika fontos. Lásd: [az Azure App Service web Apps diagnosztikai naplózás engedélyezése](/azure/app-service-web/web-sites-enable-diagnostic-log/)

**A blob-tároló napló.** Így könnyebben, amely összegyűjti és elemzi az adatokat.

**Hozzon létre egy külön tárfiókot a naplókat.** Ne használja ugyanazt a tárfiókot naplókat és az alkalmazásadatokat. Ez segít megakadályozni a naplózás alkalmazásteljesítmény csökkentése.

**Figyelemmel kísérni a teljesítményét.** A Teljesítményfigyelő szolgáltatás például [New Relic](http://newrelic.com/) vagy [Application Insights](/azure/application-insights/app-insights-overview/) alkalmazás teljesítményének figyelése és a terhelés viselkedését.  Teljesítményfigyelés lehetővé teszi az alkalmazás valós idejű betekintést. Lehetővé teszi eseményadatokat, és végezze el a probléma alapvető okát a hibákat.

### <a name="application-gateway"></a>Application Gateway

**Legalább két példányt kiépítéséhez.** Alkalmazás-átjáró üzembe helyezéséhez legalább két példánnyal. Egyetlen példány a hibaérzékeny pontok kialakulását. Két vagy több példányát használja a redundancia és a méretezhetőséget. Ahhoz, hogy érvényes a [SLA](https://azure.microsoft.com/support/legal/sla/application-gateway/v1_0/), el kell juttatnia legalább két közepes vagy nagyobb példányát.

### <a name="cosmos-db"></a>Cosmos DB

**Az adatbázis replikálása régiók között.** Cosmos DB lehetővé teszi tetszőleges számú Azure-régiók társítandó egy Cosmos-adatbázis adatbázis-fiók. Egy Cosmos DB adatbázisban régió egy írási és olvasási több régióba is rendelkezik. Ha hiba történik a írási régióban, egy másik replikából olvashatja. Az ügyfél SDK kezeli ezt automatikusan. Az írási régió, egy másik régióban is átadásra. További információkért lásd: [miként ossza el a globális adatok Azure Cosmos DB?](/azure/documentdb/documentdb-distribute-data-globally)

### <a name="redis-cache"></a>Redis Cache

**A georeplikáció konfigurálása**. A georeplikáció lehetővé teszi a csatolás két Premium szint Azure Redis Cache példányt. Az elsődleges gyorsítótár írt adatokat a másodlagos csak olvasható gyorsítótárának replikálódik. További információkért lásd: [georeplikáció konfigurálása az Azure Redis Cache](/azure/redis-cache/cache-how-to-geo-replication)

**Adatmegőrzés konfigurálásához.** A redis-adatmegőrzés lehetővé teszi a Redis tárolt adatok megőrzéséhez. Pillanatképek és hardver meghibásodása esetén is betölthet adatainak biztonsági mentését is. További információkért lásd: [adatok megőrzését egy prémium szintű Azure Redis Cache konfigurálása](/azure/redis-cache/cache-how-to-premium-persistence)

Ha a Redis Cache-gyorsítótár egy ideiglenes, nem pedig egy állandó tároló használ, ezek a javaslatok nem feltétlenül vonatkozik. 

### <a name="search"></a>Keresés

**Több replika kiépítéséhez.** Legalább két replikákat használ a olvasási magas rendelkezésre állású, vagy az olvasási és írási magas rendelkezésre állású három.

**Indexelő több területi telepítések konfigurálása.** Ha egy több területi központi telepítéssel rendelkezik, fontolja meg az indexelő folytonosságát lehetőségeknek.

  * Ha az adatforrás georeplikált, általában kell mutatnia minden egyes regionális Azure Search szolgáltatás indexelő fogja a helyi adatok adatforrás replikáját. Azt a módszert azonban nem ajánlott nagy adatmennyiségek Azure SQL adatbázis tárolja. A hiba oka, hogy Azure Search nem végezhető el a másodlagos SQL-adatbázis-replikákat, csak az elsődleges replikára változott növekményes indexelő. Összes indexelő helyette, mutasson az elsődleges másodpéldány. A feladatátvétel után az új elsődleges replika az Azure Search indexelők helyen.  
  * Ha az adatforrás nincs georeplikált, pont több indexelők ugyanarra az adatforrásra, így több régióba Azure Search szolgáltatás folyamatosan és egymástól függetlenül felületindexét az adatforrás. További információkért lásd: [Azure Search teljesítmény- és optimalizálási szempontok][search-optimization].

### <a name="storage"></a>Tárolás

**Az alkalmazásadatok használja az írásvédett georedundáns tárolás (RA-GRS).** RA-GRS tárolási másodlagos régióba replikálja az adatokat, és a másodlagos régióban olvasási hozzáférést biztosít. Ha egy tárolási leállás az elsődleges régióban, az alkalmazást a másodlagos régióba lehet beolvasni az adatokat. További információkért lásd: [Azure Storage replikációs](/azure/storage/storage-redundancy/).

**A virtuális gépek lemezei felügyelt lemezek használhatók.** [Által kezelt lemezeken] [ managed-disks] megbízhatóságnak virtuális gépek rendelkezésre állási csoportba, mert a lemezek megfelelően különítve egymástól elkerülése érdekében a hibaérzékeny pontokat. Felügyelt lemezek is, nem a VHD-k egy tárfiókot létre IOPS határértékeken. További információkért lásd: [a Windows Azure virtuális gépek rendelkezésre állásának kezelése][vm-manage-availability].

**A várólista-tároló hozzon létre egy biztonsági mentési várólistát egy másik régióban.** Várólista tárolására csak olvasható replika korlátozott használatát, mert nem a várólistára vagy created elemek. Ehelyett hozzon létre egy biztonsági mentési várólistát egy másik régióban tárfiókokban. Egy tároló leállás esetén az alkalmazás használhat a biztonsági mentési várólista, míg az elsődleges régióban elérhető újra. Így az alkalmazás továbbra is az új kérelmek tud feldolgozni.  

### <a name="sql-database"></a>SQL Database

**Standard vagy prémium szintű csomagot használja.** Ezek a rétegek adjon meg egy már pont időponthoz kötött visszaállítás időszak (35 nap). További információkért lásd: [SQL Database beállításai és teljesítménye](/azure/sql-database/sql-database-service-tiers/).

**SQL-adatbázis naplózásának engedélyezése.** Naplózás rosszindulatú támadások és emberi tévedések diagnosztizálásához használható. További információkért lásd: [Ismerkedés az SQL-adatbázis naplózásának](/azure/sql-database/sql-database-auditing-get-started/).

**Aktív Georeplikáció használja** használatát aktív Georeplikáció olvasható másodlagos létrehozásához egy másik régióban.  Ha az elsődleges adatbázis, vagy egyszerűen csak offline állapotba helyezte, hajtsa végre a másodlagos adatbázis manuális feladatátvétele van szüksége.  Feladatátvételt, amíg a másodlagos adatbázis csak olvasható maradjon.  További információkért lásd: [SQL adatbázis aktív Georeplikáció](/azure/sql-database/sql-database-geo-replication-overview/).

**Horizontális használja.** Fontolja meg az adatbázis vízszintesen particionálásához horizontális használatát. Horizontális tartalék elkülönítési biztosít. További információkért lásd: [az Azure SQL Database kiterjesztése](/azure/sql-database/sql-database-elastic-scale-introduction/).

**Pont időponthoz kötött visszaállítás segítségével emberi tévedések helyreállíthatók.**  Pont időponthoz kötött visszaállítás adja vissza, az idő az adatbázis egy korábbi állapotba. További információkért lásd: [használatával automatikus adatbázis biztonsági mentése Azure SQL-adatbázis helyreállítása][sql-restore].

**Georedundáns helyreállítás használni a helyreállításhoz a szolgáltatáskimaradás.** Georedundáns helyreállítás egy adatbázisát állítja vissza a georedundáns biztonsági másolatból.  További információkért lásd: [használatával automatikus adatbázis biztonsági mentése Azure SQL-adatbázis helyreállítása][sql-restore].

### <a name="sql-server-running-in-a-vm"></a>A virtuális gépen futó SQL Server

**Az adatbázis replikálása.** SQL Server Always On rendelkezésre állási csoportok használja replikálja az adatbázist. Magas rendelkezésre állást biztosít, ha egy SQL Server-példány sikertelen. További információkért lásd: [futtassa a Windows virtuális gépek N szintű alkalmazások](../reference-architectures/virtual-machines-windows/n-tier.md)

**Az adatbázis biztonsági mentése**. Ha már használja a [Azure biztonsági mentés](https://azure.microsoft.com/documentation/services/backup/) biztonsági mentése a virtuális gépek, érdemes lehet [Azure biztonsági mentés az SQL Server számítási feladatait a DPM használatának](/azure/backup/backup-azure-backup-sql/). Ezt a módszert van egy biztonsági mentési rendszergazda szerepkör a szervezet és a virtuális gépek és az SQL Server-egyesített helyreállítási folyamattal. Ellenkező esetben használja [SQL Server felügyelt Microsoft Azure Backup](https://msdn.microsoft.com/library/dn449496.aspx).

### <a name="traffic-manager"></a>Traffic Manager

**Manuális feladat-visszavétel végrehajtására.** A Traffic Manager feladatátvételt követően végrehajtása manuális feladat-visszavétel ahelyett, hogy automatikusan sikertelen vissza. Mielőtt ismét sikertelen, ellenőrizze, hogy minden alkalmazás alrendszer kifogástalan.  Ellenkező esetben hozhat létre olyan helyzet, amelyben az alkalmazás tükrözése oda-vissza adatközpontok között. További információkért lásd: [virtuális gépek futtatása a magas rendelkezésre állású több régióba](../reference-architectures/virtual-machines-windows/multi-region-application.md).

**Állapot-mintavételi végpont létrehozása.** Hozzon létre egy egyéni végpontot, amelyhez a jelentést az alkalmazás általános állapotát. Ez lehetővé teszi, hogy áthelyezze a feladatokat, ha bármely kritikus elérési sikertelen, nem csak az előtér Traffic Manager. A végpont egy HTTP-hibakódot kell visszaadnia, ha bármely kritikus függőségi viszonyban a nem megfelelő vagy nem érhető el. Azonban jelentést nem a nem kritikus szolgáltatások által jelzett hibákat. Ellenkező esetben a állapotmintáihoz feladatátvételt válthat nincs szükség esetén, a vakriasztások létrehozása. További információkért lásd: [Traffic Manager-végpont figyelése és a feladatátvételi](/azure/traffic-manager/traffic-manager-monitoring/).

### <a name="virtual-machines"></a>Virtuális gépek

**Kerülje a termelési munkaterhelés futó egyetlen virtuális gép.** Egyetlen Virtuálisgép-telepítéshez nem is lehetséges legyen tervezett vagy nem tervezett karbantartás. Ehelyett több virtuális gép elhelyezése egy rendelkezésre állási csoportot vagy [Virtuálisgép-méretezési készlet](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview/), az előtérben terheléselosztót.

**Adja meg a rendelkezésre állási készlet, amikor a virtuális Gépet.** Jelenleg nincs semmilyen módon nem adja hozzá a virtuális gépek rendelkezésre állási készlet a virtuális gép kiépítése után. Hozzáadásakor egy új virtuális gép rendelkezésre állási állítsa, hozzon létre egy hálózati Adaptert a virtuális géphez, majd adja hozzá a hálózati adapter a háttér-címkészlet a terheléselosztón. Ellenkező esetben a load balancer nem irányítható a hálózati forgalom, hogy a virtuális géphez.

**Minden alkalmazás réteg üzembe egy külön rendelkezésre állási csoportban.** Az N szintű alkalmazásban nem helyezték virtuális gépek a különböző rétegeket az azonos rendelkezésre állási csoportot. Virtuális gépek rendelkezésre állási csoportba kerülnek, (FDs) tartalék tartományokban, és frissítse a tartományok (UD). Azonban ahhoz, hogy a redundancia előnye, hogy FDs és UDs, a rendelkezésre állási csoport minden virtuális gép képesnek kell lennie a azonos ügyfélkérelmet kezelni.

**Válassza ki a megfelelő Virtuálisgép-méretet, a teljesítményre vonatkozó követelmények alapján.** Amikor egy meglévő alkalmazások és szolgáltatások Azure helyezi át, a Virtuálisgép-méretet, amely a leginkább megegyezik a helyszíni kiszolgálók kezdődik. Ezután a munkaterhelések tényleges Processzor, memória és IOPS lemez teljesítménye mérhető, és állítsa be a méretet, ha szükséges. Ezzel biztosíthatja az alkalmazások viselkedése a várt módon egy felhőalapú környezetben. Is ha több hálózati adapter van szüksége, vegye figyelembe az egyes hálózati adapter határérték.

**Használja a felügyelt lemezeit a VHD-k.** [Által kezelt lemezeken] [ managed-disks] megbízhatóságnak virtuális gépek rendelkezésre állási csoportba, mert a lemezek megfelelően különítve egymástól elkerülése érdekében a hibaérzékeny pontokat. Felügyelt lemezek is, nem a VHD-k egy tárfiókot létre IOPS határértékeken. További információkért lásd: [a Windows Azure virtuális gépek rendelkezésre állásának kezelése][vm-manage-availability].

**Alkalmazásokat telepít a adatlemezt, nem az operációsrendszer-lemezképet.** Ellenkező esetben előfordulhat, hogy elérte a lemez méretének korlátját.

**Azure Backup használatával biztonsági másolatot készíteni a virtuális gépek.** Biztonsági mentés véletlen adatvesztést elleni védelem érdekében. További információkért lásd: [Azure virtuális gépek védelme a recovery services-tároló](/azure/backup/backup-azure-vms-first-look-arm/).

**Diagnosztikai naplók engedélyezése**, többek között az alapszintű rendszerállapot metrika, infrastruktúra naplókat, és [rendszerindítási diagnosztika][boot-diagnostics]. Rendszerindítási diagnosztika segíthetnek diagnosztizálni rendszerindítási hiba, ha a virtuális gép nem indítható állapotának beolvasása. További információkért lásd: [áttekintés az Azure diagnosztikai naplók][diagnostics-logs].

**A AzureLogCollector a kiterjesztést használni.** (Windows virtuális gépek csak.) A bővítmény összesíti az Azure platformon naplókat, és feltölti az Azure storage az operátort, a virtuális Géphez való távoli bejelentkezés nélkül. További információkért lásd: [AzureLogCollector bővítmény](/azure/virtual-machines/virtual-machines-windows-log-collector-extension/?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

### <a name="virtual-network"></a>Virtual Network

**Engedélyezési vagy tiltólista nyilvános IP-címek, vegye fel az NSG az alhálózatot.** Engedélyezze a hozzáférést a rosszindulatú felhasználóktól, vagy a hozzáférés engedélyezése csak a felhasználók, akik jogosult az alkalmazás eléréséhez.  

**Hozzon létre egy egyéni állapotmintáihoz.** HTTP vagy TCP tesztelheti a Load Balancer állapot-mintavételi csomagjai. Ha egy virtuális Gépet egy HTTP-kiszolgáló fut, a HTTP-vizsgálatot mint egy TCP-Hálózatfigyelővel a állapotadatainak jobb kijelző. HTTP-vizsgálatot használjon egy egyéni végpontot, amelyhez a jelentések általános állapotát, az alkalmazásnak, beleértve az összes kritikus függőségeket. További információkért lásd: [Azure Load Balancer áttekintése](/azure/load-balancer/load-balancer-overview/).

**A állapotmintáihoz nincs blokkolás.** A Load Balancer állapotmintáihoz egy ismert IP-címet, 168.63.129.16 küldi. Bejövő és kimenő forgalmat az IP-bármely tűzfal házirendek és a hálózati biztonsági csoport (NSG) szabályok nincs blokkolás. Blokkolja a állapotmintáihoz okozna a terheléselosztóra, majd távolítsa el a virtuális gép elforgatás.

**Terheléselosztó naplózását.** A naplók megjelenítése a háttér-hány virtuális gépek nem fordulnak elő a hálózati forgalom miatt sikertelen mintavételi válaszokat. További információkért lásd: [analytics keresse meg a Azure Load Balancer](/azure/load-balancer/load-balancer-monitor-log/).


<!-- links -->
[app-service-autoscale]: /azure/monitoring-and-diagnostics/insights-how-to-scale/
[asynchronous-c-sharp]: /dotnet/articles/csharp/async
[availability-sets]:/azure/virtual-machines/virtual-machines-windows-manage-availability/
[azure-backup]: https://azure.microsoft.com/documentation/services/backup/
[boot-diagnostics]: https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/
[circuit-breaker]: ../patterns/circuit-breaker.md
[cloud-service-autoscale]: /azure/cloud-services/cloud-services-how-to-scale/
[diagnostics-logs]: /azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs/
[fma]: ../resiliency/failure-mode-analysis.md
[resilient-deployment]: ../resiliency/index.md#resilient-deployment
[load-balancer]: /azure/load-balancer/load-balancer-overview/
[managed-disks]: /azure/storage/storage-managed-disks-overview
[monitoring-and-diagnostics-guidance]: ../best-practices/monitoring.md
[resource-manager]: /azure/azure-resource-manager/resource-group-overview/
[retry-pattern]: ../patterns/retry.md
[retry-service-guidance]: ../best-practices/retry-service-specific.md
[search-optimization]: /azure/search/search-performance-optimization/
[sql-backup]: /azure/sql-database/sql-database-automated-backups/
[sql-restore]: /azure/sql-database/sql-database-recovery-using-backups/
[traffic-manager]: /azure/traffic-manager/traffic-manager-overview/
[traffic-manager-routing]: /azure/traffic-manager/traffic-manager-routing-methods/
[vm-manage-availability]: /azure/virtual-machines/windows/manage-availability#use-managed-disks-for-vms-in-an-availability-set
[vmss-autoscale]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview/
