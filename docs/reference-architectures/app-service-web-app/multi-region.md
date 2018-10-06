---
title: Többrégiós webalkalmazás
description: A Microsoft Azure szolgáltatásban futó magas rendelkezésre állású webalkalmazásokhoz javasolt architektúra.
author: MikeWasson
ms.date: 11/23/2016
cardTitle: Run in multiple regions
ms.openlocfilehash: 9fae95c0c0d38a756c10f37864495caad5ebda3b
ms.sourcegitcommit: b2a4eb132857afa70201e28d662f18458865a48e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 10/05/2018
ms.locfileid: "48819057"
---
# <a name="run-a-web-application-in-multiple-regions"></a>Webalkalmazás futtatása több régióban
[!INCLUDE [header](../../_includes/header.md)]

Ez a referenciaarchitektúra bemutatja, hogyan futtatható egy Azure App Service-alkalmazás több régióban a magas rendelkezésre állás elérése érdekében. 

![Referenciaarchitektúra: Magas rendelkezésre állású webalkalmazás](./images/multi-region-web-app-diagram.png) 

*Töltse le az architektúra [Visio-fájlját][visio-download].*

## <a name="architecture"></a>Architektúra 

Ez az architektúra [A méretezhetőség javítása webalkalmazásokban][guidance-web-apps-scalability] című cikkben bemutatott architektúrára épít. A legfontosabb különbségek a következők:

* **Elsődleges és másodlagos régiók**. Ez az architektúra két régiót használ a magas rendelkezésre állás eléréséhez. A rendszer az összes régióban telepíti az alkalmazást. A normál működés során a hálózati forgalom az elsődleges régió felé irányul. Ha az elsődleges régió nem érhető el, a rendszer a másodlagos régió felé irányítja a forgalmat. 
* **Azure DNS**. Az [Azure DNS][azure-dns] egy üzemeltetési szolgáltatás, amely a Microsoft Azure infrastruktúráját használja a DNS-tartományok névfeloldásához. Ha tartományait az Azure-ban üzemelteti, DNS-rekordjait a többi Azure-szolgáltatáshoz is használt hitelesítő adatokkal, API-kkal, eszközökkel és számlázási információkkal kezelheti.
* **Azure Traffic Manager**. A [Traffic Manager][traffic-manager] az elsődleges régió felé irányítja a kérelmeket. Ha az adott régiót futtató alkalmazás nem érhető el, a Traffic Manager átirányítja a forgalmat a másodlagos régió felé.
* Az SQL Database és a Cosmos DB **georeplikációja**. 

A többrégiós architektúrák magasabb rendelkezésre állást biztosíthatnak, mint az egyetlen régióban történő üzembe helyezés. Ha egy régió üzemkimaradása hatással van az elsődleges régióra, a [Traffic Manager][traffic-manager] szolgáltatással feladatátvételt hajthat végre a másodlagos régióba. Ez az architektúra az alkalmazás egy adott alrendszerének meghibásodása esetén is segíthet.

A régiók közötti magas rendelkezésre állás elérésének több általános megközelítése van: 

* Aktív/passzív készenléti kiszolgálóval. A forgalom az egyik régióra irányul, míg a másik készenléti kiszolgálón várakozik. Készenléti kiszolgáló esetében a másodlagos régió virtuális gépei folyamatosan le vannak foglalva és futnak.
* Aktív/passzív hidegtartalékkal. A forgalom az egyik régióra irányul, míg a másik hidegtartalékként áll készenlétben. Hidegtartalék esetében a másodlagos régió virtuális gépei nincsenek lefoglalva, amíg nem szükségessé nem válnak feladatátvétel céljából. Ezen megközelítés futtatása kevesebb költséggel jár, de a meghibásodáskor általában hosszabb időt vesz igénybe az üzembe állás.
* Aktív/aktív. Mindkét régió aktív, és a kérelmek terheléselosztással oszlanak meg közöttük. Ha egy régió elérhetetlenné válik, kikerül a rotációból. 

Ez a referenciaarchitektúra a készenléti kiszolgálóval konfigurált aktív/passzív üzemmódra összpontosít, a Traffic Managert használva a feladatátvételhez. 


## <a name="recommendations"></a>Javaslatok

Az Ön követelményei eltérhetnek az itt leírt architektúrától. A jelen szakaszban leírt javaslatokat tekintse kiindulópontnak.

### <a name="regional-pairing"></a>Régiónkénti párosítás
Minden egyes Azure-régió párban áll egy másikkal egy azonos földrajzi területen belül. Régiókat általában azonos regionális párokból érdemes választani (például: USA 2. keleti régiója és USA középső régiója). Ennek előnyei például a következők:

* Széles körű leállás esetén a helyreállításkor minden párból legalább az egyik régió előnyt élvez.
* A tervezett Azure-rendszerfrissítések egyszerre csak a régiópár egyik tagján jelennek meg, ami csökkenti az állásidőt.
* A legtöbb esetben a regionális párok azonos földrajzi helyen belül találhatók, hogy megfeleljenek az adatok tárolási helyére vonatkozó előírásoknak.

Azonban győződjön meg arról, hogy mindkét régió támogatja az összes Azure-szolgáltatást, amely szükséges az alkalmazásához. Lásd: [Szolgáltatások régiónként][services-by-region]. További információ a regionális párokról: [Üzletmenet-folytonosság és vészhelyreállítás (BCDR): Az Azure párosított régiói][regional-pairs].

### <a name="resource-groups"></a>Erőforráscsoportok
Fontolja meg az elsődleges és másodlagos régió, valamint a Traffic Manager külön [erőforráscsoportokban][resource groups] való elhelyezését. Ez lehetővé teszi, hogy az egyes régiókban telepített erőforrásokat egyetlen gyűjteményként kezelje.

### <a name="traffic-manager-configuration"></a>A Traffic Manager konfigurációja 

**Útválasztás**. A Traffic Manager többféle [útválasztási algoritmust][tm-routing] támogat. A cikkben leírt forgatókönyvhöz a *prioritásos* útválasztást (régebbi nevén *feladatátvétel esetén használt* útválasztás) használja. Ezzel a beállítással a Traffic Manager az összes kérelmet az elsődleges régió felé irányítja, kivéve akkor, ha az a régió elérhetetlenné válik. Ebben az esetben automatikusan átadja a feladatokat a másodlagos régiónak. Lásd: [A feladatátvétel esetén használt útválasztás konfigurálása][tm-configure-failover].

**Állapotminta**. A Traffic Manager HTTP- (vagy HTTPS-) mintavételt használ az egyes végpontok rendelkezésre állásának monitorozására. A mintavétel egy megfelelt/nem felelt meg teszttel határozza meg a Traffic Manager számára, hogy át kell-e adnia a feladatokat a másodlagos régiónak. Ehhez egy kérelmet küld egy meghatározott URL-címre. Ha egy bizonyos időkorláton belül nem 200-as értékű választ kap, a mintavétel meghiúsul. Négy sikertelen kérelem után a Traffic Manager csökkentett teljesítményűnek jelöli a végpontot, és átadja a feladatokat a másik végpontnak. Részletek: [Traffic Manager-végpont monitorozása és feladatátvétele][tm-monitoring].

Ajánlott eljárásként hozzon létre egy olyan állapotminta-végpontot, amely az alkalmazás általános állapotáról ad jelentést, és ezt a végpontot használja az állapotmintához. A végpontnak a kritikus fontosságú függőségeket kell ellenőriznie, például az App Service-alkalmazásokat, a tárolási üzenetsort és az SQL Database-t. Ellenkező esetben előfordulhat, hogy a mintavétel megfelelően működő végpontot jelent, miközben az alkalmazás kritikus fontosságú részei valójában hibásak.

Másrészről viszont ne használja az állapotmintát alacsonyabb prioritású szolgáltatások ellenőrzéséhez. Ha például egy e-mail-szolgáltatás áll le, az alkalmazás képes egy második szolgáltatóra váltani, vagy egyszerűen később elküldeni az e-maileket. Ez nem elég magas prioritás ahhoz, hogy az alkalmazás feladatátvételt kezdeményezzen. További információk: [Állapot végponti monitorozását végző minta][health-endpoint-monitoring-pattern].
 
### <a name="sql-database"></a>SQL Database
Használjon [aktív georeplikációt][sql-replication] olvasható másodlagos replika létrehozásához egy másik régióban. Legfeljebb négy olvasható másodlagos replikával rendelkezhet. Alkalmazzon feladatátvételt egy másodlagos adatbázisba, ha az elsődleges adatbázis meghibásodik vagy offline állapotba kell helyezni. Az aktív georeplikáció bármilyen rugalmas adatbáziskészlet bármilyen adatbázishoz konfigurálható.

### <a name="cosmos-db"></a>Cosmos DB
A Cosmos DB támogatja a régiók közötti georeplikációt. Az egyik régió írhatóként van kijelölve, a többi pedig csak olvasható replika.

Regionális kimaradás esetén a feladatátvételhez kijelölhet egy másik régiót írhatóként. Az ügyfél-SDK automatikusan az aktuális írható régióba küldi az írási kérelmeket, így feladatátvétel után nem kell frissítenie az ügyfél konfigurációját. További információ: [Globális adatterjesztés az Azure Cosmos DB-vel][cosmosdb-geo].

> [!NOTE]
> A replikák mindegyike ugyanabba az erőforráscsoportba tartozik.
>
>

### <a name="storage"></a>Storage
Az Azure Storage szolgáltatáshoz [írásvédett georedundáns tárolást][ra-grs] (RA-GRS) használjon. Az RA-GRS típusú tárolással a rendszer az adatokat egy másodlagos régióba replikálja. A másodlagos régióhoz csak olvasható hozzáférése van egy különálló végponton keresztül. Regionális kimaradás vagy vészhelyzet esetén az Azure Storage csapata dönthet úgy, hogy földrajzi feladatátvételt hajt végre a másodlagos régióba. Ehhez a feladatátvételhez nincs szükség az ügyfél beavatkozására.

A Queue Storage szolgáltatáshoz hozzon létre egy biztonsági várólistát a másodlagos régióban. A feladatátvétel során az alkalmazás a biztonsági várólistát használhatja, amíg az elsődleges régió újra elérhetővé nem válik. Ezzel a módszerrel az alkalmazás továbbra is az új kérelmek tud feldolgozni.

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok


### <a name="traffic-manager"></a>Traffic Manager

A Traffic Manager automatikusan elvégzi a feladatok átadását, ha az elsődleges régió elérhetetlenné válik. Amikor a Traffic Manager átadja a feladatokat, az alkalmazás egy ideig nem lesz elérhető a felhasználók számára. Ez az időtartam a következő tényezőktől függ:

* Az állapotmintának észlelnie kell, ha az elsődleges adatközpont elérhetetlenné válik.
* A tartománynév-szolgáltatás (DNS) kiszolgálóinak frissíteniük kell az IP-címhez tartozó gyorsítótárazott DNS-rekordokat. Ez a DNS élettartamától (TTL) függ. Az alapértelmezett TTL 300 másodperc (5 perc), de ezt az értéket a Traffic Manager-profil létrehozásakor konfigurálhatja.

Részletek: [A Traffic Manager monitorozásának ismertetése][tm-monitoring].

A Traffic Manager a rendszer egyik lehetséges meghibásodási pontja. Ha a szolgáltatás meghibásodik, az ügyfelek a leállás ideje alatt nem érhetik el az alkalmazást. Tekintse át a [Traffic Manager szolgáltatói szerződést (SLA)][tm-sla], és döntse el, hogy a Traffic Manager egyedüli használata elegendő-e vállalata magas rendelkezésre állásra vonatkozó követelményeihez. Ha nem, akkor a biztonság érdekében érdemes lehet hozzáadni egy másik forgalomkezelési szolgáltatást a feladat-visszavételhez. Ha az Azure Traffic Manager szolgáltatás meghibásodik, módosítsa a saját kanonikus nevének (CNAME) rekordját a DNS-ben úgy, hogy a többi forgalomkezelő szolgáltatásra mutasson. Ezt a lépést manuálisan kell elvégezni. Amíg a DNS-módosítások érvénybe nem lépnek, az alkalmazás nem lesz elérhető.

### <a name="sql-database"></a>SQL Database
Az SQL Database helyreállítási időkorlátjának (RPO) és becsült helyreállítási idejének (ERT) ismertetése [Az Azure SQL Database üzletmenet-folytonossági funkcióinak áttekintése][sql-rpo] című cikkben található. 

### <a name="storage"></a>Storage
Az RA-GRS tároló tartós tárolást biztosít, de fontos tudni, mi történhet egy esetleges kimaradás során:

* A tárhely leállásakor egy ideig nem lesz írási hozzáférése az adatokhoz. Leállás során a másodlagos végponton keresztül továbbra is olvashatók az adatok.
* Ha egy regionális kimaradás vagy vészhelyzet hatással van az elsődleges helyszínre, és az ott található adatok nem állíthatók helyre, az Azure Storage csapata dönthet úgy, hogy földrajzi feladatátvételt hajt végre a másodlagos régióba.
* Az adatok replikálása a másodlagos régióba aszinkron módon történik. Ezért, amikor földrajzi feladatátvétel megy végbe, adatvesztés történhet, ha az adatok nem állíthatók teljesen helyre az elsődleges régióból.
* Az átmeneti hibák – például a hálózati leállások – nem indítják meg a tároló feladatátvételét. Tervezze alkalmazását úgy, hogy ellenálló legyen az átmeneti hibákkal szemben. Lehetséges megoldások:
  
  * Olvasás a másodlagos régióból.
  * Ideiglenes átváltás másik tárfiókra az új írási műveletekhez (például az üzenetek üzenetsorba való helyezéséhez).
  * Adatok átmásolása a másodlagos régióból egy másik tárfiókba.
  * Csökkentett szolgáltatás nyújtása, amíg a rendszer végre nem hajtja a feladat-visszavételt.

További információk: [Mi a Mi a teendő az Azure Storage leállása esetén][storage-outage].

## <a name="manageability-considerations"></a>Felügyeleti szempontok

### <a name="traffic-manager"></a>Traffic Manager

Ha a Traffic Manager feladatátvételt hajt végre, automatikus feladat-visszavétel megvalósítása helyett a manuális feladat-visszavételt ajánlunk. Ellenkező esetben előfordulhat, hogy egyes esetekben az alkalmazás oda-vissza váltogat a régiók között. A feladat-visszavétel előtt ellenőrizze, hogy az alkalmazás minden alrendszere működik-e.

Vegye figyelembe, hogy a Traffic Manager alapértelmezés szerint automatikusan végrehajtja a feladat-visszavételt. Ennek megelőzéséhez manuálisan csökkentse az elsődleges régió prioritását a feladatátvétel után. Tegyük fel például, hogy az elsődleges régió 1-es prioritású, a második pedig 2-es. A feladatátvétel után az automatikus visszavétel megelőzéséhez állítsa az elsődleges régió prioritását 3-asra. A prioritást akkor állítsa 1-esre, ha már készen áll a visszaváltásra.

A prioritás a következő parancsokkal frissíthető.

**PowerShell**

```bat
$endpoint = Get-AzureRmTrafficManagerEndpoint -Name <endpoint> -ProfileName <profile> -ResourceGroupName <resource-group> -Type AzureEndpoints
$endpoint.Priority = 3
Set-AzureRmTrafficManagerEndpoint -TrafficManagerEndpoint $endpoint
```

További információk: [Az Azure Traffic Manager parancsmagjai][tm-ps].

**Azure parancssori felület (CLI)**

```bat
azure network traffic-manager endpoint set --name <endpoint> --profile-name <profile> --resource-group <resource-group> --type AzureEndpoints --priority 3
```    

### <a name="sql-database"></a>SQL Database
Ha az elsődleges adatbázis meghibásodik, hajtson végre manuális feladatátvételt a másodlagos adatbázisra. Lásd: [Az Azure SQL Database visszaállítása vagy feladatátvétel a másodlagos kiszolgálóra][sql-failover]. A másodlagos adatbázis csak olvasható marad, amíg el nem végzi a feladatátvételt.


<!-- links -->

[azure-sql-db]: https://azure.microsoft.com/documentation/services/sql-database/
[azure-dns]: /azure/dns/dns-overview
[cosmosdb-geo]: /azure/cosmos-db/distribute-data-globally
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
[tm-ps]: /powershell/module/azurerm.trafficmanager
[tm-routing]: /azure/traffic-manager/traffic-manager-routing-methods
[tm-sla]: https://azure.microsoft.com/support/legal/sla/traffic-manager
[traffic-manager]: https://azure.microsoft.com/services/traffic-manager
[visio-download]: https://archcenter.blob.core.windows.net/cdn/app-service-reference-architectures.vsdx
