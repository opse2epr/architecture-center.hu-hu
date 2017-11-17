---
title: "Több területi webalkalmazás"
description: "Ajánlott architektúra magas rendelkezésre állással, a Microsoft Azure-ban futó webalkalmazás."
author: MikeWasson
ms.date: 11/23/2016
cardTitle: Run in multiple regions
ms.openlocfilehash: 2d7d0c38bef3efc73a7ba2dd61e4190d07deb1b5
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="run-a-web-application-in-multiple-regions"></a>Egy webalkalmazás több régióba futtatása
[!INCLUDE [header](../../_includes/header.md)]

A referencia-architektúrában több régióba magas rendelkezésre állás biztosítása érdekében az Azure App Service-alkalmazás futtatását mutatja. 

![Architektúra hivatkozni: magas rendelkezésre állású-webalkalmazás](./images/multi-region-web-app-diagram.png) 

*Töltse le a [Visio fájl] [ visio-download] ezen architektúra.*

## <a name="architecture"></a>Architektúra 

Ez az architektúra látható egy épít [webalkalmazás a méretezhetőség javítása][guidance-web-apps-scalability]. A fő különbségek a következők:

* **Elsődleges és másodlagos régiók**. Ez az architektúra két régió magasabb rendelkezésre állásának eléréséhez használja. Az alkalmazás központi telepítése mindegyik régióhoz. A normál működés során a rendszer a hálózati forgalom irányítja az elsődleges régióban. Ha nem érhető el az elsődleges régióban, továbbítódik a másodlagos régióba. 
* **Az Azure Traffic Manager**. [A TRAFFIC Manager] [ traffic-manager] irányítja a bejövő kéréseket az elsődleges régióban. Ha az adott régióban futó alkalmazás nem érhető el, a Traffic Manager feladatátadás a másodlagos régióba.
* **A georeplikáció** SQL-adatbázis és a Cosmos-adatbázis. 

Egy több területi architektúra biztosít magas rendelkezésre állás érdekében, mint egyetlen régióban üzembe helyezni. Ha egy regionális kimaradás érinti az elsődleges régióban, [Traffic Manager] [ traffic-manager] hogy áthelyezze a feladatokat a másodlagos régióba. Ez az architektúra segíthet is, ha az alkalmazás egy egyedi alrendszer nem tud.

Magas rendelkezésre állás elérése érdekében a különböző régiókban néhány általános módszer van: 

* A melegtartalék aktív/passzív. Forgalom kerül egy régió tartozik, a más vár melegtartalék közben. Meleg készenléti azt jelenti, hogy a virtuális gépeket, a másodlagos régióban lefoglalt és fut mindig.
* A meleg készenléti állapotban lévő aktív/passzív. Forgalom kerül egy régió tartozik, a más vár meleg készenléti közben. Meleg készenléti állapot azt jelenti, hogy a virtuális gépeket, a másodlagos régióban nem foglal le addig, amíg a feladatátvételi szükséges. Ez a megközelítés költségek kisebb futtatásához, de általában adott meghibásodás során online állapotba hosszabb ideig tart.
* Aktív. Mindkét régió aktív, és kérés kerül közöttük. Ha nem érhető el egy régió tartozik, az kívül Elforgatás lesz végrehajtva. 

A referencia-architektúrában melegtartalék feladatátvételi Traffic Manager használata az aktív/passzív összpontosít. 


## <a name="recommendations"></a>Javaslatok

A követelmények eltérhetnek az itt leírt architektúra. A javaslatok használja ebben a szakaszban kiindulási pontként.

### <a name="regional-pairing"></a>Regionális párosítás
Minden Azure-régió, egy másik ugyanazon a földrajzi régióját párosított. Régiók általában választhat a azonos regionális pár (például USA keleti régiója 2. régiója, USA középső RÉGIÓJA). Ennek során a következő előnyöket nyújtja:

* A széles körű kimaradás esetén legalább egy régió kívül minden pár helyreállítási prioritása van.
* Tervezett Azure rendszerfrissítések egymás után, az lehetséges állásidő minimalizálása érdekében párosított régiók már megkezdődött.
* A legtöbb esetben a regionális párok található az ugyanazon földrajzi adatok rezidens követelményeinek megfelelően.

Azonban győződjön meg arról, hogy mindkét régió támogatja az összes Azure-szolgáltatás szükséges az alkalmazáshoz. Lásd: [régiói][services-by-region]. Regionális párok kapcsolatos további információkért lásd: [üzleti folytonossági és vészhelyreállítási helyreállítási (BCDR): Azure-régiókat párosítva][regional-pairs].

### <a name="resource-groups"></a>Erőforráscsoportok
Vegye figyelembe az elsődleges régióban, a másodlagos régióba és a Traffic Manager elhelyezését külön [erőforráscsoportok][resource groups]. Ez lehetővé teszi minden egyes régió egyetlen gyűjteményként telepített erőforrások kezelését.

### <a name="traffic-manager-configuration"></a>A TRAFFIC Manager-konfiguráció 

**Útválasztás**. A TRAFFIC Manager támogatja több [útválasztási algoritmusok][tm-routing]. A jelen cikkben ismertetett esetben használjon *prioritás* útválasztási (korábbi nevén *feladatátvételi* útválasztási). Ezzel a beállítással a Traffic Manager minden kérést küld az elsődleges régióban hacsak az adott régió végpont el nem érhető el. Ezen a ponton akkor automatikusan átkerül a másodlagos régióba. Lásd: [konfigurálhatja feladatátvételi útválasztási módszer][tm-configure-failover].

**Állapotmintáihoz**. A TRAFFIC Manager egy HTTP-(vagy HTTPS) mintavételi használja a végpontok rendelkezésre állásának figyeléséhez. A mintavétel a másodlagos régióba feladatátvételét fázis/sikertelen próba adja meg a Traffic Manager. Működését tekintve a kérést küld a megadott URL-címe. A határidőn belül nem 200 választ kap, ha a mintavételi sikertelen lesz. Négy sikertelen kérelmek, miután a Traffic Manager jelöli meg a végpont állapota csökkentett teljesítményűre vált, és átadja a feladatokat a többi végpont. További információkért lásd: [Traffic Manager-végpont figyelése és a feladatátvételi][tm-monitoring].

Ajánlott eljárásként hozzon létre egy egészségügyi mintavételi végpontot, amely jelent a alkalmazás általános állapotát, és használni ezt a végpontot a állapotmintáihoz. A végpont ellenőrizni kell az App Service alkalmazások, a tároló várólista és az SQL-adatbázis például kritikus fontosságú függőségek. Ellenkező esetben a mintavétel előfordulhat, hogy jelentés a kifogástalan állapotú végpontok, ha az alkalmazás kritikus részei ténylegesen nem működnek.

Másrészről ne használja a állapotmintáihoz alacsonyabb prioritású virtuális gép szolgáltatások kereséséhez. Például ha olyan e-mail szolgáltatás leáll az alkalmazás képes második-szolgáltatóra váltani, vagy csak később az e-mailek küldése. Ez nem egy elég magas prioritású virtuális gép a feladatátvételt az alkalmazás. További információkért lásd: [állapotfigyelő végpont figyelési mintát][health-endpoint-monitoring-pattern].
 
### <a name="sql-database"></a>SQL Database
Használjon [aktív Georeplikáció] [ sql-replication] olvasható másodlagos replika létrehozásához egy másik régióban. Legfeljebb négy olvasható másodlagos másodpéldányokra lehet. Átadja egy másodlagos adatbázist, ha az elsődleges adatbázis meghibásodik vagy offline állapotba kell. Aktív Georeplikáció a rugalmas adatbáziskészlet bármely adatbázis konfigurálható.

### <a name="cosmos-db"></a>Cosmos DB
Cosmos DB georeplikáció különböző régiókban támogat. Egy régió tartozik kijelölt írható és a többi csak olvasható replika.

Regionális kimaradás esetén is utasíthat el át egy másik régióban kell lennie a írási régió kiválasztásával. SDK automatikusan elküldi az ügyfélnek az aktuális írási terület, az írási kérelmeket szolgál, így nem kell frissíteni az ügyfél beállításait egy feladatátvétel után. További információkért lásd: [miként ossza el a globális adatok Azure Cosmos DB?][docdb-geo]

> [!NOTE]
> A replikák mindegyikét tartozik ugyanabban az erőforráscsoportban.
>
>

### <a name="storage"></a>Storage
Az Azure Storage használata [írásvédett georedundáns tárolás] [ ra-grs] (RA-GRS). Az RA-GRS tárolással rendelkező másodlagos régióba replikálja az adatok. Az adatok csak olvasási hozzáféréssel rendelkezik a másodlagos régióban külön végpontot keresztül. Ha egy regionális kimaradás vagy katasztrófa, az Azure Storage csapat dönthet, hogy a másodlagos régióba földrajzi-feladatátvétel végrehajtásához. Nincs nincs feladatátvételi szükség felhasználói beavatkozásra.

A Queue storage a biztonsági mentési várólista létrehozása a másodlagos régióban. A feladatátvétel során az alkalmazás használhatja a biztonsági mentési várólista, míg az elsődleges régióban elérhető újra. Így az alkalmazás továbbra is az új kérelmek tud feldolgozni.

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok


### <a name="traffic-manager"></a>Traffic Manager

A TRAFFIC Manager automatikusan leállítja keresztül, ha az elsődleges régióban nem érhető el. Ha a Traffic Manager átadja, van egy adott időn belül, ha ügyfelek nem tudják elérni az alkalmazást. A duration hatással van a következő tényezőket:

* A állapotmintáihoz észlelni kell, hogy az elsődleges adatközpont vált nem érhető el.
* Tartományi szolgáltatási (DNS) névkiszolgálók frissítenie kell a gyorsítótárban lévő DNS-rekordok az IP-címhez, amely a DNS--élettartama (TTL) függ. Az alapértelmezett élettartam 300 másodpercen (5 percen), de ezt az értéket a Traffic Manager-profil létrehozásakor konfigurálhatja.

További információkért lásd: [kapcsolatos Traffic Manager figyelési][tm-monitoring].

A TRAFFIC Manager ponttá lehetséges hiba a rendszerben. Ha a szolgáltatás nem sikerül, az ügyfelek nem tudnak hozzáférni az alkalmazás a leállások során. Tekintse át a [Traffic Manager szolgáltatásiszint-szerződéssel (SLA)] [ tm-sla] és döntse el, hogy kizárólag a Traffic Manager segítségével megfelel-e az üzleti követelményeinek, a magas rendelkezésre állás érdekében. Ha nem, akkor vegyen fel egy másik forgalom felügyeleti megoldás, a feladat-visszavételre. Ha az Azure Traffic Manager szolgáltatás nem sikerül, módosítsa a kanonikus név (CNAME) rekord a DNS-ben, hogy a többi felügyeleti szolgálat mutasson. Ebben a lépésben kézzel kell elvégezni, és az alkalmazás sem lesznek elérhetők, amíg a DNS-módosításokat továbbítja.

### <a name="sql-database"></a>SQL Database
A helyreállítási időkorlát (RPO) és az SQL-adatbázis becsült helyreállításkor (Beszúrása) című témakörben [az Azure SQL Database üzletmenet áttekintése][sql-rpo]. 

### <a name="storage"></a>Storage
RA-GRS tárolási tartós tárolására szolgál, de fontos tudni, mi történhet kimaradás során:

* A tárolási tervezett kimaradás esetén nem lesznek egy adott időn belül, ha nem rendelkezik írási-férhetnek hozzá az adatokhoz. Ön továbbra is olvashatók be a másodlagos végponti a szolgáltatáskimaradás elhárítása során.
* Ha egy regionális kimaradás vagy katasztrófa hatással van az elsődleges helyre, és nem lehet helyreállítani az adatokat, az Azure Storage csapat dönthet végezzen el egy földrajzi-feladatátvételt a másodlagos régióba.
* A másodlagos régióba replikálása aszinkron módon történik. Ezért a földrajzi feladatátvétel történik, ha adatvesztést, lehetséges, ha az adatokat nem lehet helyreállítani az elsődleges régióban.
* Átmeneti hibák, például egy hálózati kimaradás nem indít el a tárolási feladatátvevő. Tervezze meg az alkalmazás átmeneti hibák rugalmasak lehetnek. Lehetséges megoldást:
  
  * A másodlagos régióba olvasni.
  * Átmenetileg váltson egy másik tárfiókhoz új az írási műveletek (például, hogy a várólista-üzenetek).
  * A másodlagos régióba adatainak másolása másik tárolási fiókot.
  * Adja meg a csökkentett, amíg a rendszer visszaállítása sikertelen.

További információkért lásd: [Mi a teendő, ha egy Azure Storage esetleges leálláskor][storage-outage].

## <a name="manageability-considerations"></a>A kezelhetőséggel kapcsolatos szempontok

### <a name="traffic-manager"></a>Traffic Manager

Ha a Traffic Manager átadja, ajánlott végrehajtása manuális feladat-visszavétel, nem pedig egy automatikus feladat-visszavétel végrehajtására. Ellenkező esetben hozhat létre olyan helyzet, amelyben az alkalmazás oda-vissza régiók közötti tükrözi. Ellenőrizze, hogy minden alkalmazás alrendszer visszavétele előtt kifogástalan.

Vegye figyelembe, hogy a Traffic Manager vissza alapértelmezés szerint automatikusan nem. Ennek megelőzése érdekében manuálisan a Prioritás csökkentése az elsődleges régió egy feladatátvételi esemény után. Tegyük fel például, hogy az elsődleges régióban 1 prioritású virtuális gép, és a másodlagos prioritású 2. A feladatátvétel után állítsa be az elsődleges régióban prioritás 3, hogy megakadályozza az automatikus feladatátvételi. Ha készen áll vissza, frissítse a prioritás 1.

Az alábbi parancsokat a prioritás frissítése.

**PowerShell**

```bat
$endpoint = Get-AzureRmTrafficManagerEndpoint -Name <endpoint> -ProfileName <profile> -ResourceGroupName <resource-group> -Type AzureEndpoints
$endpoint.Priority = 3
Set-AzureRmTrafficManagerEndpoint -TrafficManagerEndpoint $endpoint
```

További információkért lásd: [Azure Traffic Manager parancsmagok][tm-ps].

**Az Azure parancssori felület (CLI)**

```bat
azure network traffic-manager endpoint set --name <endpoint> --profile-name <profile> --resource-group <resource-group> --type AzureEndpoints --priority 3
```    

### <a name="sql-database"></a>SQL Database
Ha az elsődleges adatbázis, hajtsa végre a másodlagos adatbázis manuális feladatátvétele. Lásd: [egy Azure SQL Database vagy feladatátvételi visszaállításához a másodlagos][sql-failover]. A másodlagos adatbázis csak olvasható marad, amíg a rendszer átadja.


<!-- links -->

[azure-sql-db]: https://azure.microsoft.com/documentation/services/sql-database/
[docdb-geo]: /azure/documentdb/documentdb-distribute-data-globally
[guidance-web-apps-scalability]: ./scalable-web-app.md
[health-endpoint-monitoring-pattern]: https://msdn.microsoft.com/library/dn589789.aspx
[ra-grs]: /azure/storage/storage-redundancy#read-access-geo-redundant-storage
[regional-pairs]: /azure/best-practices-availability-paired-regions
[resource groups]: /azure/azure-resource-manager/resource-group-overview#resource-groups
[services-by-region]: https://azure.microsoft.com/regions/#services
[sql-failover]: /azure/sql-database/sql-database-disaster-recovery
[sql-replication]: /azure/sql-database/sql-database-geo-replication-overview
[sql-rpo]: /azure/sql-database/sql-database-business-continuity#sql-database-features-that-you-can-use-to-provide-business-continuity
[storage-outage]: /azure/storage/storage-disaster-recovery-guidance
[tm-configure-failover]: /azure/traffic-manager/traffic-manager-configure-failover-routing-method
[tm-monitoring]: /azure/traffic-manager/traffic-manager-monitoring
[tm-ps]: https://msdn.microsoft.com/library/mt125941.aspx
[tm-routing]: /azure/traffic-manager/traffic-manager-routing-methods
[tm-sla]: https://azure.microsoft.com/support/legal/sla/traffic-manager/v1_0/
[traffic-manager]: https://azure.microsoft.com/services/traffic-manager/
[visio-download]: https://archcenter.azureedge.net/cdn/app-service-reference-architectures.vsdx
