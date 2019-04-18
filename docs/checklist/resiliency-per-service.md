---
title: Rugalmasságra vonatkozó ellenőrzőlista az Azure-szolgáltatásokhoz
titleSuffix: Azure Design Review Framework
description: Rugalmasságra vonatkozó útmutatás különböző Azure-szolgáltatásokat biztosító ellenőrzőlista.
author: petertaylor9999
ms.date: 11/26/2018
ms.topic: checklist
ms.service: architecture-center
ms.subservice: cloud-design-principles
ms.custom: resiliency, checklist
ms.openlocfilehash: db42bd259bf71ef2ffa3e9efc5e4cd6ba2078e6b
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/17/2019
ms.locfileid: "59639699"
---
# <a name="resiliency-checklist-for-specific-azure-services"></a>Rugalmasságra vonatkozó ellenőrzőlista az adott Azure-szolgáltatásokhoz

Rugalmasság rendszer azon képessége, hogy helyreálljon a hibák után, és továbbra is működnek. Minden technológiai rendelkezik a saját adott hibaállapot, amely, amikor figyelembe kell vennie, amelyek segítenek az alkalmazás. Az alábbi ellenőrzőlista használatával tekintse át a rugalmassággal kapcsolatos szempontok az adott Azure-szolgáltatásokhoz. Rugalmas alkalmazások tervezése kapcsolatos további információkért lásd: [megbízható Azure-alkalmazások tervezése](../reliability/index.md).

## <a name="app-service"></a>App Service

**Használjon Standard vagy prémium csomagra.** Ezek a csomagok támogatja az átmeneti tárhelyek és automatikus biztonsági mentéseket. További információkért lásd: [Azure App Service díjcsomagjainak részletes áttekintése](/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview/)

**Kerülje a felfelé és lefelé skálázást.** Ehelyett válasszon egy olyan szintet és a példányméret, amelyek megfelelnek a teljesítményigényeinek tipikus terhelés mellett, majd [horizontális felskálázása](/azure/app-service-web/web-sites-scale/) a példányokat a megváltozott forgalommennyiség adatforgalma. Skálázás felfelé és lefelé indíthat újra kell indítani az alkalmazást.

**Konfigurációs Store alkalmazásbeállításokként.** Alkalmazásbeállítások használata, amely a konfigurációs beállítások alkalmazásbeállításokként tárolja. Adja meg a beállításokat a Resource Manager-sablonokat, vagy a PowerShell-lel, így alkalmazza őket egy automatikus központi telepítésének részeként, / folyamat, amely több frissítése. További információkért lásd: [webalkalmazások konfigurálása az Azure App Service](/azure/app-service-web/web-sites-configure/).

**Hozzon létre külön App Service-csomagok az üzemi és tesztkörnyezetben.** Ne használjon tárhelyek az éles környezet teszteléséhez.  Az azonos App Service-csomagban található alkalmazások megosztása a VM-példányokon. Ha termelési és tesztelési célú telepítések ugyanabban a csomagban helyezi, az éles üzembe helyezés negatív hatással lehet. Ha például terheléstesztek csökkentheti az éles helyet. Egy külön tervbe tesztkörnyezetek helyezésével, elkülönítse azokat a változatát tartalmazza.

**A web API-k külön web Apps-alkalmazások.** Ha a megoldás is egy webes előtérrendszert és egy webes API-t, érdemes decomposing őket külön App Service-alkalmazásokba. Ez a kialakítás megkönnyíti a megoldás felbontás számítási feladat alapján. Futtathatja a web app és az API-t külön App Service-csomagok, így ezek egymástól függetlenül skálázhatók. Ha nincs szükség ilyen szintű méretezhetőségre, először telepítheti az alkalmazások ugyanabba a csomagba, és helyezze át őket szét később, ha szükséges.

**Kerülje az App Service biztonsági mentési szolgáltatás Azure SQL-adatbázisok biztonsági mentését.** Ehelyett használjon [SQL-adatbázis automatikus biztonsági mentések][sql-backup]. Az App Service biztonsági mentési exportálja az adatbázist egy SQL .bacpac fájlt, amely költségek a dtu-k.

**Az előkészítési pont telepítése.** Az átmeneti üzembe helyezési tárhely létrehozása. Alkalmazás-frissítéseket telepítheti az előkészítési ponton, és a telepítés előtt sikeressége az éles környezetbe. Ez csökkenti az esélyét, hogy egy rossz frissítés éles környezetben. Emellett biztosítja, hogy a összes példányát, mielőtt éles környezetben a felcserélés folyamatban vannak bemelegíteni. Számos alkalmazás jelentős melegítési és hidegindítási idő van. További információkért lásd: [állítsa be átmeneti környezeteket az Azure App Service web apps](/azure/app-service-web/web-sites-staged-publishing/).

**Hozzon létre egy üzembe helyezési pont, amely tárolja a legutóbbi megfelelően működő (LKG) központi telepítés.** Ha éles környezetben telepít egy frissítést, a LKG tárolóhely helyezze át a korábbi éles környezetet. Ez megkönnyíti az visszavonásához működő üzemelő példányt. Ha probléma merül fel újabb, gyorsan visszaállíthatja a LKG verzióra. További információkért lásd: [alapszintű webalkalmazás](../reference-architectures/app-service-web-app/basic-web-app.md).

**Diagnosztikai naplózás engedélyezése**, beleértve az alkalmazásnaplózást és a webkiszolgáló-naplózást. A naplózás nem fontos a monitorozás és diagnosztika. Lásd: [az Azure App Service web Apps-alkalmazások diagnosztikai célú naplózásának engedélyezése](/azure/app-service-web/web-sites-enable-diagnostic-log/)

**Jelentkezzen be a blob storage-bA.** Ez megkönnyíti a összegyűjteni és elemezni az adatokat.

**Hozzon létre egy külön tárfiókot a naplók számára.** Ne használja ugyanazt a tárfiókot a naplók és alkalmazásadatok. Ez segít megakadályozni a naplózás az alkalmazások teljesítményét csökkenti.

**Teljesítmény figyelése.** A Teljesítményfigyelő szolgáltatás például [New relic-bővítménnyel](https://newrelic.com/) vagy [Application Insights](/azure/application-insights/app-insights-overview/) alkalmazások teljesítményének figyelése és a viselkedés terhelés alatt.  Alkalmazásteljesítmény-figyelő lehetővé teszi az alkalmazás valós idejű betekintést. Lehetővé teszi, hogy diagnosztizálhatja a problémákat, és a hibák alapvető okok elemzését.

## <a name="application-gateway"></a>Application Gateway

**Legalább két példányt üzembe helyezhető.** Helyezze üzembe az Application Gateway legalább két példánnyal. Egyetlen példány egy meghibásodási pont. Két vagy több példány üzemeltetéséhez a redundancia és a méretezhetőség. Ahhoz, hogy megfeleljen a [SLA](https://azure.microsoft.com/support/legal/sla/application-gateway), legalább két közepes vagy nagy méretű példánnyal kell kiépítenie.

## <a name="cosmos-db"></a>Cosmos DB

**Az adatbázis replikálása régiók között elosztva.** A cosmos DB segítségével tetszőleges számú Azure-régiók társítása egy Cosmos DB-adatbázisfiók. Cosmos DB-adatbázis egy írási régiót és a több olvasási régióval is rendelkezhetnek. Ha hiba van az írási régió, egy másik replikára képes olvasni. Az ügyfél-SDK ezt automatikusan megteszi. Az írási régió egy másik régióban is feladatátvételt. További információkért lásd: [hogyan globális adatterjesztés az Azure Cosmos DB](/azure/cosmos-db/distribute-data-globally).

## <a name="event-hubs"></a>Event Hubs

**Az ellenőrzőpontok használata**.  Egy eseményközpontból, eseményfelhasználó kell írni az aktuális pozícióját adattárolásra néhány előre meghatározott időközönként. Így ha az ügyfél egy tartalék (például a fogyasztói szoftverleállások vagy a gazdagép nem), majd egy új példányt folytathatja a utolsó feljegyzett beosztás érkező adatfolyam olvasása. További információkért lásd: [eseményfelhasználó](/azure/event-hubs/event-hubs-features#event-consumers).

**Ismétlődő üzenetek kezeléséhez.** Ha nem sikerül egy eseményközpontból, eseményfelhasználó, üzenetfeldolgozás rögzített az utolsó ellenőrzőponttól folytatja a működését. Az üzenetek, feldolgozása már megtörtént az utolsó ellenőrzőpont után újra lesz feldolgozva. Ezért a feldolgozó logika üzenet idempotensnek kell lenniük, vagy az alkalmazás üzeneteket deduplikálása képesnek kell lennie.

**Kivételek kezelésére.** . Egy eseményközpontból, eseményfelhasználó általában ismétlődő üzenetek kötegelt dolgozza fel. Kezelje a kivételeket, ha egy adott üzenet miatt a kivétel teljes üzenetköteget az adatvesztés elkerülése érdekében a feldolgozási ciklus belül.

**Használja a kézbesítetlen levelek várólistájára vonatkozik.** Egy üzenet feldolgozása eredményeként nem átmeneti hibát, ha helyezze az üzenet a kézbesítetlen levelek várólistája alakzatot, így nyomon követheti az állapotát. A forgatókönyvtől függően előfordulhat, hogy egy másik műveletet, a alkalmazni a kompenzáló tranzakció vagy próbálkozzon újra később az üzenet el. Vegye figyelembe, hogy az Event Hubs nem rendelkezik olyan beépített kézbesítetlen levelek várólistájára funkciót. Azure Queue Storage vagy a Service Bus használatával végrehajtja a kézbesítetlen levelek várólistájára vonatkozik, vagy használja az Azure Functions vagy valamilyen más eseménykezelési mechanizmus.

**Vész-helyreállítási a másodlagos Event Hubs-névtér-ba irányuló feladatátvétel végrehajtására.** További információkért lásd: [Azure Event Hubs Geo-disaster recovery](/azure/event-hubs/event-hubs-geo-dr).

## <a name="redis-cache"></a>Redis Cache

**Georeplikáció konfigurálása**. A georeplikáció lehetővé teszi a két prémium szintű Azure Redis Cache-példány csatolása. Az elsődleges gyorsítótár írt adatok replikálódnak a másodlagos csak olvasható gyorsítótáraként. További információkért lásd: [georeplikáció konfigurálása az Azure Redis Cache-gyorsítótárhoz](/azure/redis-cache/cache-how-to-geo-replication)

**Adatmegőrzés konfigurálása.** A redis megőrzési funkciója segítségével megőrizheti a Redis-ban tárolt adatokkal. Pillanatfelvételt és biztonsági mentése az adatok betölthetők egy hardverhiba. További információkért lásd: [prémium szintű Azure Redis Cache-gyorsítótárhoz adatmegőrzés konfigurálása](/azure/redis-cache/cache-how-to-premium-persistence)

Ha a Redis Cache ideiglenes gyorsítótárként, és nem állandó tárolóként használ, ezek a javaslatok nem alkalmazható.

## <a name="search"></a>Keresés

**Több replika üzembe helyezhető.** Legalább két replika használata olvasási magas rendelkezésre állású, vagy három az olvasási és írási magas rendelkezésre állású.

**Konfigurálhatja az indexelőket többrégiós üzemelő példányokhoz.** Ha több régióból álló üzemelő, fontolja meg a beállításokat az indexelő folytonosságának.

- Ha az adatforrás georeplikált, általában kell mutatnia minden indexelő minden regionális Azure Search szolgáltatás a helyi adatok forrásreplikára. Ezt a megközelítést az Azure SQL Database-ben tárolt nagyméretű adathalmazok azonban nem ajánlott. A hiba oka, hogy az Azure Search nem hajtható végre a másodlagos SQL-adatbázis-replikákat, csak az elsődleges replika indexelő növekményes. Ehelyett minden indexelő mutasson az elsődleges replika. A feladatátvétel után az új elsődleges replika az Azure Search-indexelők mutassanak.

- Ha az adatforrás nem áll, georeplikált, pont, ugyanazt az adatforrást több indexelő, hogy több régióban az Azure Search-szolgáltatás folyamatosan és egymástól függetlenül index az adatforrásból. További információkért lásd: [Azure Search-teljesítmény és optimalizálás szempontok][search-optimization].

## <a name="service-bus"></a>Service Bus

**Prémium szintű használata a termelési számítási feladatokhoz**. [Service Bus prémium szintű üzenetkezelés](/azure/service-bus-messaging/service-bus-premium-messaging) biztosít dedikált és fenntartott feldolgozási erőforrásokat, és memóriakapacitásának kiszámítható teljesítményt és az átviteli sebesség támogatására. Prémium szintű üzenetkezelés csomag is hozzáférést biztosít, amely csak a prémium szintű ügyfelek először számára elérhető új szolgáltatásokkal. Eldöntheti, hogy a várható számítási feladatok alapján üzenetkezelési egységek száma.

**Ismétlődő üzenetek kezeléséhez**. Ha a kiadó üzenet elküldése után azonnal meghiúsul, vagy hálózati vagy a rendszer meghibásodik, hibásan előfordulhat, hogy jegyezze fel, hogy az üzenet lett kézbesítve, és előfordulhat, hogy az azonos üzenetet küld a rendszer kétszer. A Service Bus duplikált elemek észlelésének engedélyezésével úgy tudja kezelni a probléma. További információkért lásd: [észlelési ismétlődő](/azure/service-bus-messaging/duplicate-detection).

**Kivételek kezelése**. Üzenetkezelési API-k készítése a kivételeket, ha egy felhasználói hiba, konfigurációs hiba vagy más hiba lép fel. Az Ügyfélkód (küldők és fogadók) kezelje ezeket a kivételeket a kódra összpontosítsanak. Ez különösen fontos a kötegelt feldolgozás, ahol a kivételkezelés is használható, teljes üzenetköteget az adatvesztés elkerülése érdekében. További információkért lásd: [Service Bus-üzenetkezelés kivételei](/azure/service-bus-messaging/service-bus-messaging-exceptions).

**Újrapróbálkozási szabályzat**. A Service Bus lehetővé teszi az alkalmazások számára legjobb újrapróbálkozási szabályzat kiválasztása. Az alapértelmezett házirendet, hogy a 9 maximális újrapróbálkozások száma, és várjon 30 másodpercet, de ez további módosítható. További információkért lásd: [újrapróbálkozási szabályzat – a Service Bus](/azure/architecture/best-practices/retry-service-specific#service-bus).

**Használja a kézbesítetlen levelek várólistájára**. Ha az üzenet nem lehet feldolgozni, vagy több próbálkozás után címzettnek sem lett elküldve, a kézbesítetlen levelek várólistája kerülnek. Állítson be egy folyamatot az üzenetek olvasásakor a feldolgozatlan üzenetek sorhoz, vizsgálja meg őket és háríthatja el a problémát. A forgatókönyvtől függően előfordulhat, hogy ismételje meg az üzenetet,-, a módosításokat, és próbálkozzon újra, vagy vesse el az üzenetet. További információkért lásd: [a Service Bus – áttekintés kézbesíthetetlen levelek sorai](/azure/service-bus-messaging/service-bus-dead-letter-queues).

**Használja a Geo-Disaster Recovery**. GEO-vészhelyreállítás biztosítja, hogy adatfeldolgozási továbbra is megfelelően működjenek, eltérő régióban vagy datacenter, ha egy teljes Azure-régiót vagy datacenter katasztrófa miatt elérhetetlenné válik. További információkért lásd: [Azure Service Bus Geo-disaster recovery](/azure/service-bus-messaging/service-bus-geo-dr).

## <a name="storage"></a>Storage

**Az alkalmazásadatok esetében használja az írásvédett georedundáns tárolás (RA-GRS).** RA-GRS tároló egy másodlagos régióba replikálja az adatokat, és a másodlagos régióból a csak olvasási hozzáférést biztosít. Ha a társzolgáltatás kimaradása az elsődleges régióban, az alkalmazás olvashatja az adatokat a másodlagos régióból. További információkért lásd: [Azure Storage replikáció](/azure/storage/storage-redundancy/).

**Virtuálisgép-lemezek a Managed Disks használata.** [A Managed Disks] [ managed-disks] megbízhatóságnak-beli virtuális gépek rendelkezésre állási csoportban, mert a lemezei kellőképpen különítve egymástól a kritikus hibapontok elkerülése érdekében. Felügyelt lemezek is, nem a storage-fiókban létrehozott virtuális merevlemezek, az IOPS-korlátok vonatkoznak. További információkért lásd: [Azure-beli Windows virtuális gépek rendelkezésre állásának kezelése][vm-manage-availability].

**A Queue storage hozzon létre egy biztonsági várólistát egy másik régióban.** A Queue storage, a csak olvasható replika korlátozott használja, mert nem lehet üzenetsor vagy elemek eltávolítása a sorból. Ehelyett hozzon létre egy biztonsági várólistát a storage-fiókban egy másik régióban. Ha a társzolgáltatás kimaradása, az alkalmazás a biztonsági várólistát használhatja, az elsődleges régió amíg elérhetővé nem válik újra. Ezzel a módszerrel az alkalmazás továbbra is az új kérelmek tud feldolgozni.

## <a name="sql-database"></a>SQL Database

**Használjon Standard vagy prémium csomagra.** Ezek a rétegek adjon meg egy már időponthoz visszaállítás ideje (35 nap). További információkért lásd: [SQL Database beállításai és teljesítménye](/azure/sql-database/sql-database-service-tiers/).

**Az SQL Database naplózási funkciójának engedélyezése.** Naplózás rosszindulatú támadások vagy az emberi hibák diagnosztizálásához használható. További információkért lásd: [első lépései az SQL database naplózási szolgáltatásával](/azure/sql-database/sql-database-auditing-get-started/).

**Aktív Georeplikáció használatához** használatát aktív Georeplikáció egy olvasható másodlagos létrehozásához egy másik régióban.  Ha az elsődleges adatbázis meghibásodik, vagy egyszerűen csak meg kell offline állapotba kell, a másodlagos adatbázis manuális feladatátvételt hajt végre.  Amíg a rendszer átadja a feladatokat, a másodlagos adatbázis csak olvasható marad.  További információkért lásd: [SQL adatbázis aktív Georeplikációt](/azure/sql-database/sql-database-geo-replication-overview/).

**A horizontális skálázás használja.** Érdemes a horizontális skálázás az adatbázis horizontális particionálásához. A horizontális skálázás is biztosítják a hibák elszigetelését. További információkért lásd: [horizontális felskálázás az Azure SQL Database](/azure/sql-database/sql-database-elastic-scale-introduction/).

**Időponthoz visszaállítás segítségével visszaállíthatja az emberi hibák.**  Időponthoz visszaállítási adja vissza az adatbázist egy korábbi időpontra idő. További információkért lásd: [helyreállítani egy Azure SQL database automatikus biztonsági adatbázismentés használatával][sql-restore].

**A geo-visszaállítás segítségével visszaállíthatja a szolgáltatás-meghibásodás után.** A GEO-visszaállítás visszaállít egy adatbázist egy georedundáns biztonsági másolatból.  További információkért lásd: [helyreállítani egy Azure SQL database automatikus biztonsági adatbázismentés használatával][sql-restore].

## <a name="sql-data-warehouse"></a>SQL Data Warehouse

**Tiltsa le a georedundáns biztonsági másolat.** Alapértelmezés szerint az SQL Data Warehouse egy teljes biztonsági másolatának elkészítése adatait vész-helyreállítási 24 óránként. Ez a funkció kikapcsolása nem ajánlott. További információkért lásd: [Geo-biztonsági mentések](/azure/sql-data-warehouse/backup-and-restore#geo-backups).

## <a name="sql-server-running-in-a-vm"></a>Egy virtuális Gépen futó SQL Server

**Az adatbázis replikálása.** Az SQL Server Always On rendelkezésre állási csoportok használatával replikálja az adatbázist. Magas rendelkezésre állást biztosít, ha egy SQL Server-példány sikertelen. További információkért lásd: [Windows rendszerű virtuális gépek futtatása N szintű alkalmazáshoz](../reference-architectures/virtual-machines-windows/n-tier.md)

**Az adatbázis biztonsági mentése**. Ha már használja a [Azure Backup](/azure/backup/) érdemes használni a virtuális gépek biztonsági mentése, [Azure Backup az SQL Server-tevékenységprofilokhoz DPM használatával](/azure/backup/backup-azure-backup-sql/). Ezzel a módszerrel nincs a szervezet egy biztonsági mentési rendszergazda szerepkört, és a egy egységes helyreállítási eljárás virtuális gépek és az SQL Server. Ellenkező esetben használjon [SQL Server Managed Backup a Microsoft Azure-bA](https://msdn.microsoft.com/library/dn449496.aspx).

## <a name="traffic-manager"></a>Traffic Manager

**Manuális feladat-visszavételt végrehajtani.** A Traffic Manager feladatátvételt végrehajtani a manuális feladat-visszavételt, nem pedig automatikusan visszavétel. Visszavétel előtt győződjön meg arról, hogy alkalmazás minden alrendszere megfelelő állapotú.  Ellenkező esetben létrehozhat egy olyan helyzetet, ahol az alkalmazás oda-vissza az adatközpontok között tükrözi. További információkért lásd: [virtuális gépek futtatása több régióban a magas rendelkezésre állású](../reference-architectures/virtual-machines-windows/multi-region-application.md).

**Hozzon létre egy állapotminta-végpontot.** Hozzon létre egy egyéni végpontot, hogy az alkalmazás általános állapotát. Ez lehetővé teszi a Traffic Manager átadja a feladatokat, ha bármely kritikus útvonalat nem sikerül, nem csak a kezelőfelület segítségével. A végpont HTTP-hibakódot kell visszaadnia, ha bármely kritikus fontosságú függőség nem megfelelő vagy nem érhető el. Nem hibajelentés az üzleti szempontból nem létfontosságú szolgáltatások azonban. Ellenkező esetben az állapotminta is kiválthatják feladatátvétel, ha nincs rá szükség, téves létrehozása. További információkért lásd: [Traffic Manager végpont figyelése és feladatátvétele](/azure/traffic-manager/traffic-manager-monitoring/).

## <a name="virtual-machines"></a>Virtuális gépek

**Kerülje a egy éles számítási feladatok futtatása egyetlen virtuális Gépet.** Egy egyetlen virtuális gép üzembe helyezésének nem képes legyen ellenállni a tervezett vagy nem tervezett karbantartás. Ehelyett helyezzen több virtuális gépet egy rendelkezésre állási csoportban vagy [Virtuálisgép-méretezési csoportot](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview/), az a load balancerrel elöl.

**Adjon meg egy rendelkezésre állási csoportot a virtuális gép üzembe helyezésekor.** Jelenleg nincs lehetőség a virtuális gép hozzáadása egy rendelkezésre állási csoportot a virtuális gép kiépítése után. Hozzáadásakor egy új virtuális gép olyan meglévő rendelkezésre állási állítsa be, ügyeljen arra, hogy hozzon létre egy hálózati Adaptert a virtuális gép, és a hálózati adapter hozzáadása a háttér-címkészletet a terheléselosztón. Ellenkező esetben a terheléselosztó nem irányítható a hálózati forgalom a virtuális Gépre.

**Mindössze az egyes alkalmazásrétegek külön rendelkezésre állási csoportban.** Az N szintű alkalmazáshoz ne helyezzen virtuális gépet a különböző szintek az azonos rendelkezésre állási csoportba. Virtuális gépet egy rendelkezésre állási csoportban vannak elhelyezve a tartalék tartományok között, és frissítési tartományok (UD). Azonban a redundancia élvezheti a tartalék és frissítési tartománnyal, a rendelkezésre állási csoportban lévő összes virtuális Géphez kell tudni az azonos ügyfélkérelmek kezelését.

**Az Azure Site Recovery virtuális gépeket replikálni.** Amikor replikálhat Azure virtuális gépek [Site Recovery][site-recovery], a Virtuálisgép-lemezek folyamatosan replikálja a rendszer a célként megadott régióban aszinkron módon történik. A helyreállítási pontok jönnek létre néhány perces időközönként. Ez lehetővé teszi a helyreállítási időkorlátot (RPO) sorrendjében perc. Vészhelyreállítási próba annyiszor azt szeretné, anélkül, hogy ez hatással lenne az éles alkalmazás vagy a folyamatban lévő replikáció végezhet. További információkért lásd: [vészhelyreállítási gyakorlatának futtatása az Azure-bA][site-recovery-test].

**Válassza ki a megfelelő Virtuálisgép-méret, teljesítmény-követelmények alapján.** Amikor helyez át egy meglévő számítási feladatok Azure-ba, indítsa el a Virtuálisgép-méretet, amely a leginkább egyezik a helyszíni kiszolgálókhoz. Ezután mérheti a CPU, memória és a lemez iops-t a valós számítási feladat teljesítményét, és módosítsa a méretet, ha szükséges. Ezzel biztosíthatja az alkalmazás egy felhőalapú környezetben a várt módon viselkedik. Is ha több hálózati adapter van szüksége, vegye figyelembe a hálózati adapter korlát méreteire vonatkoztatva.

**A felügyelt lemezek használata a virtuális merevlemezeket.** [A Managed Disks] [ managed-disks] megbízhatóságnak-beli virtuális gépek rendelkezésre állási csoportban, mert a lemezei kellőképpen különítve egymástól a kritikus hibapontok elkerülése érdekében. Felügyelt lemezek is, nem a storage-fiókban létrehozott virtuális merevlemezek, az IOPS-korlátok vonatkoznak. További információkért lásd: [Azure-beli Windows virtuális gépek rendelkezésre állásának kezelése][vm-manage-availability].

**Alkalmazások telepítése az adatlemezt, nem az operációsrendszer-lemez.** Ellenkező esetben előfordulhat, hogy eléri a lemez mérete korlátot.

**Az Azure Backup használatával virtuális gépek biztonsági mentése.** Biztonsági másolatok véletlen adatvesztés ellen. További információkért lásd: [Azure virtuális gépek védelme recovery services-tároló](/azure/backup/backup-azure-vms-first-look-arm/).

**Diagnosztikai naplók engedélyezése**, beleértve az alapvető állapotmetrikákat, infrastruktúra naplófájljait és [rendszerindítási diagnosztika][boot-diagnostics]. A rendszerindítási diagnosztika segít diagnosztizálni a rendszerindítási hibát, ha a virtuális gép rendszerindításra képtelen állapotba. További információkért lásd: [áttekintése az Azure diagnosztikai naplók][diagnostics-logs].

**AzureLogCollector bővítményt használja.** (Windows virtuális gépek csak.) Ez a bővítmény az Azure platform naplók összesíti, és anélkül, hogy a virtuális gép távolról bejelentkezik az operátor feltölti azokat az Azure storage. További információkért lásd: [AzureLogCollector bővítmény](/azure/virtual-machines/virtual-machines-windows-log-collector-extension/?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

## <a name="virtual-network"></a>Virtual Network

**Engedélyezési és blokkolási nyilvános IP-címeket adja hozzá egy NSG-t az alhálózathoz.** Letiltja a hozzáférést a rosszindulatú felhasználóktól, vagy a hozzáférés engedélyezése csak a felhasználók, akik rendelkeznek jogosultsággal az alkalmazás eléréséhez.

**Hozzon létre egy egyéni állapotmintát.** Load Balancer állapot-Mintavételei tesztelheti a HTTP vagy TCP. Ha egy virtuális Gépet egy HTTP-kiszolgáló fut, a HTTP-mintavétel, mint a TCP-mintavételt állapotadatainak jobb kijelző. HTTP-mintavétel használata egy egyéni végpontot, amely az alkalmazás általános állapotáról ad a, beleértve az összes kritikus fontosságú függőség jelentést. További információkért lásd: [Azure Load Balancer áttekintése](/azure/load-balancer/load-balancer-overview/).

**Az állapotminta nem blokkolja.** A Load Balancer állapot-mintavétel egy ismert IP-címet, a 168.63.129.16 címről érkezik. Nem blokkolja a bejövő és kimenő forgalmat lévő valamelyik tűzfalszabályzatban vagy a hálózati biztonsági csoport (NSG) szabályai ezen IP-címet. Az állapotminta blokkolja, távolítsa el a virtuális Gépet a rotációból, hogy a terheléselosztó okozna.

**Load Balancer-naplózás engedélyezése.** A naplók megmutatják, hány virtuális gépet a háttér-a rendszer nem fogad hálózati forgalmat a meghiúsult mintavételi válaszok miatt. További információkért lásd: [Log analytics az Azure Load Balancerhez](/azure/load-balancer/load-balancer-monitor-log/).

<!-- links -->
[boot-diagnostics]: https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/
[diagnostics-logs]: /azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs/
[managed-disks]: /azure/storage/storage-managed-disks-overview
[search-optimization]: /azure/search/search-performance-optimization/
[site-recovery]: /azure/site-recovery/
[site-recovery-test]: /azure/site-recovery/site-recovery-test-failover-to-azure
[sql-backup]: /azure/sql-database/sql-database-automated-backups/
[sql-restore]: /azure/sql-database/sql-database-recovery-using-backups/
[vm-manage-availability]: /azure/virtual-machines/windows/manage-availability#use-managed-disks-for-vms-in-an-availability-set
