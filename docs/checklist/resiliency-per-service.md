---
title: Rugalmasság ellenőrzőlista az Azure-szolgáltatások
description: Útmutató rugalmasság a különböző Azure-szolgáltatásokat nyújtó ellenőrzőlista.
author: petertaylor9999
ms.date: 03/02/2018
ms.custom: resiliency, checklist
ms.openlocfilehash: 25d961d6bb753b1f515fc073e51bbb912cc59db7
ms.sourcegitcommit: 2123c25b1a0b5501ff1887f98030787191cf6994
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/08/2018
---
# <a name="resiliency-checklist-for-specific-azure-services"></a>Rugalmasság ellenőrzőlista az adott Azure-szolgáltatások

Rugalmasság azon képessége, a rendszer helyreállni hibák után, és továbbra is működik, és az egyik a [szoftver minőségű oszlopok](../guide/pillars.md). Minden technológiára saját adott hiba módokat, amelyek kialakításakor és az alkalmazást kell végiggondolni rendelkezik. Az alábbi ellenőrzőlista segítségével tekintse át az adott Azure-szolgáltatások rugalmassági szempontjai. Tekintse át a [általános rugalmassági ellenőrzőlista](./resiliency.md).

## <a name="app-service"></a>App Service

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

## <a name="application-gateway"></a>Application Gateway

**Legalább két példányt kiépítéséhez.** Alkalmazás-átjáró üzembe helyezéséhez legalább két példánnyal. Egyetlen példány a hibaérzékeny pontok kialakulását. Két vagy több példányát használja a redundancia és a méretezhetőséget. Ahhoz, hogy érvényes a [SLA](https://azure.microsoft.com/support/legal/sla/application-gateway/v1_0/), el kell juttatnia legalább két közepes vagy nagyobb példányát.

## <a name="cosmos-db"></a>Cosmos DB

**Az adatbázis replikálása régiók között.** Cosmos DB lehetővé teszi tetszőleges számú Azure-régiók társítandó egy Cosmos-adatbázis adatbázis-fiók. Egy Cosmos DB adatbázisban régió egy írási és olvasási több régióba is rendelkezik. Ha hiba történik a írási régióban, egy másik replikából olvashatja. Az ügyfél SDK kezeli ezt automatikusan. Az írási régió, egy másik régióban is átadásra. További információkért lásd: [miként ossza el a globális adatok Azure Cosmos DB](/azure/cosmos-db/distribute-data-globally).

## <a name="event-hubs"></a>Event Hubs

**Használjon ellenőrzőpontokat**.  Egy eseményközpontból, eseményfelhasználó az egyes előre meghatározott időközönként állandó tároló kell írni a jelenlegi állapotában. Így ha a fogyasztó (például a fogyasztói összeomlik vagy az állomás nem tud) hibát észlel, majd új példányt folytathatja az adatfolyam olvasásakor az utolsó rögzített hely. További információkért lásd: [eseményfelhasználók](/azure/event-hubs/event-hubs-features#event-consumers).

**Duplikált üzenetek kezeléséhez.** Ha nem sikerül egy eseményközpontból, eseményfelhasználó, üzenet feldolgozása a legutóbbi rögzített ellenőrzőponttól folytatja a működését. Az üzenetek után utolsó ellenőrzőpontja feldolgozása már megtörtént, újra lesz feldolgozva. Ezért az üzenetfeldolgozás logikát kell az idempotent, vagy az alkalmazás fájlok deduplikálása üzenetek képesnek kell lennie.

**Kivétel kezelése.** . Egy eseményközpontból, eseményfelhasználó általában az ismétlődő üzenetkötegek dolgozza fel. Kezelje a kivételek elkerülése érdekében az egész köteget üzenetek elvesztését, ha egyetlen üzenet kivételt okoz a feldolgozási hurkon belül.

**Használja a kézbesítetlen levelek várólistájára.** Ha egy üzenet feldolgozása nem átmeneti hibát, az üzenet lemezre írását a kézbesítetlen levelek várólistájára, hogy a állapotát nyomon követheti. A helyzettől függően előfordulhat, hogy néhány más művelet végrehajtása, olyan kompenzációs tranzakció vagy az üzenet később újra el. Vegye figyelembe, hogy az Event Hubs nem rendelkezik olyan beépített kézbesítetlen levelek várólistájára funkciót. Azure Queue Storage vagy a Service Bus használatával valósítja meg a kézbesítetlen levelek várólistájára, illetve az Azure Functions vagy valamilyen más esemény mód használata.  

**Katasztrófa utáni helyreállítás egy másodlagos Event Hubs névtér feladatátvételét által megvalósítani.** További információkért lásd: [Azure Event Hubs földrajzi-vész-helyreállítási](/azure/event-hubs/event-hubs-geo-dr).

## <a name="redis-cache"></a>Redis Cache

**A georeplikáció konfigurálása**. A georeplikáció lehetővé teszi a csatolás két Premium szint Azure Redis Cache példányt. Az elsődleges gyorsítótár írt adatokat a másodlagos csak olvasható gyorsítótárának replikálódik. További információkért lásd: [georeplikáció konfigurálása az Azure Redis Cache](/azure/redis-cache/cache-how-to-geo-replication)

**Adatmegőrzés konfigurálásához.** A redis-adatmegőrzés lehetővé teszi a Redis tárolt adatok megőrzéséhez. Pillanatképek és hardver meghibásodása esetén is betölthet adatainak biztonsági mentését is. További információkért lásd: [adatok megőrzését egy prémium szintű Azure Redis Cache konfigurálása](/azure/redis-cache/cache-how-to-premium-persistence)

Ha a Redis Cache-gyorsítótár egy ideiglenes, nem pedig egy állandó tároló használ, ezek a javaslatok nem feltétlenül vonatkozik. 

## <a name="search"></a>Keresés

**Több replika kiépítéséhez.** Legalább két replikákat használ a olvasási magas rendelkezésre állású, vagy az olvasási és írási magas rendelkezésre állású három.

**Indexelő több területi telepítések konfigurálása.** Ha egy több területi központi telepítéssel rendelkezik, fontolja meg az indexelő folytonosságát lehetőségeknek.

  * Ha az adatforrás georeplikált, általában kell mutatnia minden egyes regionális Azure Search szolgáltatás indexelő fogja a helyi adatok adatforrás replikáját. Azt a módszert azonban nem ajánlott nagy adatmennyiségek Azure SQL adatbázis tárolja. A hiba oka, hogy Azure Search nem végezhető el a másodlagos SQL-adatbázis-replikákat, csak az elsődleges replikára változott növekményes indexelő. Összes indexelő helyette, mutasson az elsődleges másodpéldány. A feladatátvétel után az új elsődleges replika az Azure Search indexelők helyen.  
  * Ha az adatforrás nincs georeplikált, pont több indexelők ugyanarra az adatforrásra, így több régióba Azure Search szolgáltatás folyamatosan és egymástól függetlenül felületindexét az adatforrás. További információkért lásd: [Azure Search teljesítmény- és optimalizálási szempontok][search-optimization].

## <a name="storage"></a>Tárolás

**Az alkalmazásadatok használja az írásvédett georedundáns tárolás (RA-GRS).** RA-GRS tárolási másodlagos régióba replikálja az adatokat, és a másodlagos régióban olvasási hozzáférést biztosít. Ha egy tárolási leállás az elsődleges régióban, az alkalmazást a másodlagos régióba lehet beolvasni az adatokat. További információkért lásd: [Azure Storage replikációs](/azure/storage/storage-redundancy/).

**A virtuális gépek lemezei felügyelt lemezek használhatók.** [Által kezelt lemezeken] [ managed-disks] megbízhatóságnak virtuális gépek rendelkezésre állási csoportba, mert a lemezek megfelelően különítve egymástól elkerülése érdekében a hibaérzékeny pontokat. Felügyelt lemezek is, nem a VHD-k egy tárfiókot létre IOPS határértékeken. További információkért lásd: [a Windows Azure virtuális gépek rendelkezésre állásának kezelése][vm-manage-availability].

**A várólista-tároló hozzon létre egy biztonsági mentési várólistát egy másik régióban.** Várólista tárolására csak olvasható replika korlátozott használatát, mert nem a várólistára vagy created elemek. Ehelyett hozzon létre egy biztonsági mentési várólistát egy másik régióban tárfiókokban. Egy tároló leállás esetén az alkalmazás használhat a biztonsági mentési várólista, míg az elsődleges régióban elérhető újra. Így az alkalmazás továbbra is az új kérelmek tud feldolgozni.  

## <a name="sql-database"></a>SQL Database

**Standard vagy prémium szintű csomagot használja.** Ezek a rétegek adjon meg egy már pont időponthoz kötött visszaállítás időszak (35 nap). További információkért lásd: [SQL Database beállításai és teljesítménye](/azure/sql-database/sql-database-service-tiers/).

**SQL-adatbázis naplózásának engedélyezése.** Naplózás rosszindulatú támadások és emberi tévedések diagnosztizálásához használható. További információkért lásd: [Ismerkedés az SQL-adatbázis naplózásának](/azure/sql-database/sql-database-auditing-get-started/).

**Aktív Georeplikáció használja** használatát aktív Georeplikáció olvasható másodlagos létrehozásához egy másik régióban.  Ha az elsődleges adatbázis, vagy egyszerűen csak offline állapotba helyezte, hajtsa végre a másodlagos adatbázis manuális feladatátvétele van szüksége.  Feladatátvételt, amíg a másodlagos adatbázis csak olvasható maradjon.  További információkért lásd: [SQL adatbázis aktív Georeplikáció](/azure/sql-database/sql-database-geo-replication-overview/).

**Horizontális használja.** Fontolja meg az adatbázis vízszintesen particionálásához horizontális használatát. Horizontális tartalék elkülönítési biztosít. További információkért lásd: [az Azure SQL Database kiterjesztése](/azure/sql-database/sql-database-elastic-scale-introduction/).

**Pont időponthoz kötött visszaállítás segítségével emberi tévedések helyreállíthatók.**  Pont időponthoz kötött visszaállítás adja vissza, az idő az adatbázis egy korábbi állapotba. További információkért lásd: [használatával automatikus adatbázis biztonsági mentése Azure SQL-adatbázis helyreállítása][sql-restore].

**Georedundáns helyreállítás használni a helyreállításhoz a szolgáltatáskimaradás.** Georedundáns helyreállítás egy adatbázisát állítja vissza a georedundáns biztonsági másolatból.  További információkért lásd: [használatával automatikus adatbázis biztonsági mentése Azure SQL-adatbázis helyreállítása][sql-restore].

## <a name="sql-server-running-in-a-vm"></a>A virtuális gépen futó SQL Server

**Az adatbázis replikálása.** SQL Server Always On rendelkezésre állási csoportok használja replikálja az adatbázist. Magas rendelkezésre állást biztosít, ha egy SQL Server-példány sikertelen. További információkért lásd: [futtassa a Windows virtuális gépek N szintű alkalmazások](../reference-architectures/virtual-machines-windows/n-tier.md)

**Az adatbázis biztonsági mentése**. Ha már használja a [Azure biztonsági mentés](https://azure.microsoft.com/documentation/services/backup/) biztonsági mentése a virtuális gépek, érdemes lehet [Azure biztonsági mentés az SQL Server számítási feladatait a DPM használatának](/azure/backup/backup-azure-backup-sql/). Ezt a módszert van egy biztonsági mentési rendszergazda szerepkör a szervezet és a virtuális gépek és az SQL Server-egyesített helyreállítási folyamattal. Ellenkező esetben használja [SQL Server felügyelt Microsoft Azure Backup](https://msdn.microsoft.com/library/dn449496.aspx).

## <a name="traffic-manager"></a>Traffic Manager

**Manuális feladat-visszavétel végrehajtására.** A Traffic Manager feladatátvételt követően végrehajtása manuális feladat-visszavétel ahelyett, hogy automatikusan sikertelen vissza. Mielőtt ismét sikertelen, ellenőrizze, hogy minden alkalmazás alrendszer kifogástalan.  Ellenkező esetben hozhat létre olyan helyzet, amelyben az alkalmazás tükrözése oda-vissza adatközpontok között. További információkért lásd: [virtuális gépek futtatása a magas rendelkezésre állású több régióba](../reference-architectures/virtual-machines-windows/multi-region-application.md).

**Állapot-mintavételi végpont létrehozása.** Hozzon létre egy egyéni végpontot, amelyhez a jelentést az alkalmazás általános állapotát. Ez lehetővé teszi, hogy áthelyezze a feladatokat, ha bármely kritikus elérési sikertelen, nem csak az előtér Traffic Manager. A végpont egy HTTP-hibakódot kell visszaadnia, ha bármely kritikus függőségi viszonyban a nem megfelelő vagy nem érhető el. Azonban jelentést nem a nem kritikus szolgáltatások által jelzett hibákat. Ellenkező esetben a állapotmintáihoz feladatátvételt válthat nincs szükség esetén, a vakriasztások létrehozása. További információkért lásd: [Traffic Manager-végpont figyelése és a feladatátvételi](/azure/traffic-manager/traffic-manager-monitoring/).

## <a name="virtual-machines"></a>Virtuális gépek

**Kerülje a termelési munkaterhelés futó egyetlen virtuális gép.** Egyetlen Virtuálisgép-telepítéshez nem is lehetséges legyen tervezett vagy nem tervezett karbantartás. Ehelyett több virtuális gép elhelyezése egy rendelkezésre állási csoportot vagy [Virtuálisgép-méretezési készlet](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview/), az előtérben terheléselosztót.

**Adja meg a rendelkezésre állási készlet, amikor a virtuális Gépet.** Jelenleg nincs semmilyen módon nem adja hozzá a virtuális gépek rendelkezésre állási készlet a virtuális gép kiépítése után. Hozzáadásakor egy új virtuális gép rendelkezésre állási állítsa, hozzon létre egy hálózati Adaptert a virtuális géphez, majd adja hozzá a hálózati adapter a háttér-címkészlet a terheléselosztón. Ellenkező esetben a load balancer nem irányítható a hálózati forgalom, hogy a virtuális géphez.

**Minden alkalmazás réteg üzembe egy külön rendelkezésre állási csoportban.** Az N szintű alkalmazásban nem helyezték virtuális gépek a különböző rétegeket az azonos rendelkezésre állási csoportot. Virtuális gépek rendelkezésre állási csoportba kerülnek, (FDs) tartalék tartományokban, és frissítse a tartományok (UD). Azonban ahhoz, hogy a redundancia előnye, hogy FDs és UDs, a rendelkezésre állási csoport minden virtuális gép képesnek kell lennie a azonos ügyfélkérelmet kezelni.

**Válassza ki a megfelelő Virtuálisgép-méretet, a teljesítményre vonatkozó követelmények alapján.** Amikor egy meglévő alkalmazások és szolgáltatások Azure helyezi át, a Virtuálisgép-méretet, amely a leginkább megegyezik a helyszíni kiszolgálók kezdődik. Ezután a munkaterhelések tényleges Processzor, memória és IOPS lemez teljesítménye mérhető, és állítsa be a méretet, ha szükséges. Ezzel biztosíthatja az alkalmazások viselkedése a várt módon egy felhőalapú környezetben. Is ha több hálózati adapter van szüksége, vegye figyelembe az egyes hálózati adapter határérték.

**Használja a felügyelt lemezeit a VHD-k.** [Által kezelt lemezeken] [ managed-disks] megbízhatóságnak virtuális gépek rendelkezésre állási csoportba, mert a lemezek megfelelően különítve egymástól elkerülése érdekében a hibaérzékeny pontokat. Felügyelt lemezek is, nem a VHD-k egy tárfiókot létre IOPS határértékeken. További információkért lásd: [a Windows Azure virtuális gépek rendelkezésre állásának kezelése][vm-manage-availability].

**Alkalmazásokat telepít a adatlemezt, nem az operációsrendszer-lemezképet.** Ellenkező esetben előfordulhat, hogy elérte a lemez méretének korlátját.

**Azure Backup használatával biztonsági másolatot készíteni a virtuális gépek.** Biztonsági mentés véletlen adatvesztést elleni védelem érdekében. További információkért lásd: [Azure virtuális gépek védelme a recovery services-tároló](/azure/backup/backup-azure-vms-first-look-arm/).

**Diagnosztikai naplók engedélyezése**, többek között az alapszintű rendszerállapot metrika, infrastruktúra naplókat, és [rendszerindítási diagnosztika][boot-diagnostics]. Rendszerindítási diagnosztika segíthetnek diagnosztizálni rendszerindítási hiba, ha a virtuális gép nem indítható állapotának beolvasása. További információkért lásd: [áttekintés az Azure diagnosztikai naplók][diagnostics-logs].

**A AzureLogCollector a kiterjesztést használni.** (Windows virtuális gépek csak.) A bővítmény összesíti az Azure platformon naplókat, és feltölti az Azure storage az operátort, a virtuális Géphez való távoli bejelentkezés nélkül. További információkért lásd: [AzureLogCollector bővítmény](/azure/virtual-machines/virtual-machines-windows-log-collector-extension/?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

## <a name="virtual-network"></a>Virtual Network

**Engedélyezési vagy tiltólista nyilvános IP-címek, vegye fel az NSG az alhálózatot.** Engedélyezze a hozzáférést a rosszindulatú felhasználóktól, vagy a hozzáférés engedélyezése csak a felhasználók, akik jogosult az alkalmazás eléréséhez.  

**Hozzon létre egy egyéni állapotmintáihoz.** HTTP vagy TCP tesztelheti a Load Balancer állapot-mintavételi csomagjai. Ha egy virtuális Gépet egy HTTP-kiszolgáló fut, a HTTP-vizsgálatot mint egy TCP-Hálózatfigyelővel a állapotadatainak jobb kijelző. HTTP-vizsgálatot használjon egy egyéni végpontot, amelyhez a jelentések általános állapotát, az alkalmazásnak, beleértve az összes kritikus függőségeket. További információkért lásd: [Azure Load Balancer áttekintése](/azure/load-balancer/load-balancer-overview/).

**A állapotmintáihoz nincs blokkolás.** A Load Balancer állapotmintáihoz egy ismert IP-címet, 168.63.129.16 küldi. Bejövő és kimenő forgalmat az IP-bármely tűzfal házirendek és a hálózati biztonsági csoport (NSG) szabályok nincs blokkolás. Blokkolja a állapotmintáihoz okozna a terheléselosztóra, majd távolítsa el a virtuális gép elforgatás.

**Terheléselosztó naplózását.** A naplók megjelenítése a háttér-hány virtuális gépek nem fordulnak elő a hálózati forgalom miatt sikertelen mintavételi válaszokat. További információkért lásd: [analytics keresse meg a Azure Load Balancer](/azure/load-balancer/load-balancer-monitor-log/).

<!-- links -->
[boot-diagnostics]: https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/
[diagnostics-logs]: /azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs/
[managed-disks]: /azure/storage/storage-managed-disks-overview
[search-optimization]: /azure/search/search-performance-optimization/
[sql-backup]: /azure/sql-database/sql-database-automated-backups/
[sql-restore]: /azure/sql-database/sql-database-recovery-using-backups/
[vm-manage-availability]: /azure/virtual-machines/windows/manage-availability#use-managed-disks-for-vms-in-an-availability-set
