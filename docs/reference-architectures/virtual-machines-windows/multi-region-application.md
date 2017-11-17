---
title: "Windows virtuális gépek futtatása a több Azure-régiók a magas rendelkezésre állás"
description: "Ügyfélszoftverek központi telepítése a virtuális gépek magas rendelkezésre állást és a rugalmasság Azure több régióba."
author: MikeWasson
ms.date: 11/22/2016
pnp.series.title: Windows VM workloads
pnp.series.prev: n-tier
ms.openlocfilehash: b3f1fcf1403a5199191cb37dfed4fbe86695766d
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="run-windows-vms-in-multiple-regions-for-high-availability"></a>Futtassa a Windows virtuális gépek több régióba a magas rendelkezésre állás

A referencia-architektúrában futó N szintű alkalmazás több Azure-régiók, rendelkezésre állást és a hatékony vész-helyreállítási infrastruktúra elérése érdekében bevált gyakorlatok csoportja jeleníti meg. 

[![0]][0] 

*Töltse le a [Visio fájl] [ visio-download] ezen architektúra.*

## <a name="architecture"></a>Architektúra 

Ez az architektúra látható egy épít [futtassa a Windows virtuális gépek N szintű alkalmazások](n-tier.md). 

* **Elsődleges és másodlagos régiók**. Két régió segítségével magasabb rendelkezésre állásának eléréséhez. Az elsődleges régióban egyik. A más régióban van, a feladatátvételre. 
* **Az Azure Traffic Manager**. [A TRAFFIC Manager] [ traffic-manager] irányítja a bejövő kéréseket a régiók egyikéhez sem. A normál működés során az elsődleges régióban irányítja a kérelmeket. Ha az adott régióban, a Traffic Manager feladatátadás a másodlagos régióba. További információkért tekintse meg a szakasz [Traffic Manager konfigurációs](#traffic-manager-configuration).
* **Erőforráscsoportok**. Hozzon létre külön [erőforráscsoportok] [ resource groups] az elsődleges régióban, a másodlagos régióba, és a Traffic Manager. Ez lehetővé teszi a rugalmasság, minden egyes régió szerinti egyetlen gyűjtemény-erőforrások kezelésére. Például egy régió tartozik, sikerült újratelepítése nélkül a másik egy le. [Az erőforráscsoportok hivatkozás][resource-group-links], így az alkalmazás az erőforrások listájának lekérdezés futtatása.
* **Vnetek**. Hozzon létre egy külön hálózatok mindegyik régióhoz. Ellenőrizze, hogy a címterek nem lehetnek átfedésben. 
* **SQL Server Always On rendelkezésre állási csoport**. Ha az SQL Server rendszer használata esetén ajánlott [SQL Always On rendelkezésre állási csoportok] [ sql-always-on] a magas rendelkezésre állás érdekében. Az SQL Server-példányokat tartalmazó mindkét régiókban egyetlen rendelkezésre állási csoport létrehozása. 

    > [!NOTE]
    > Is érdemes lehet [Azure SQL Database][azure-sql-db], amely a relációs adatbázisok felhőszolgáltatásként biztosítja. Az SQL Database nem kell egy rendelkezésre állási csoport konfigurálásához, vagy feladatátvételi kezelése.  
    > 

* **VPN-átjárók**. Hozzon létre egy [VPN-átjáró] [ vpn-gateway] minden egyes virtuális, és konfigurálja a [VNet – VNet-kapcsolatot][vnet-to-vnet], a két Vnetek közötti hálózati forgalom engedélyezése . Ez elengedhetetlen ahhoz az SQL Always On rendelkezésre állási csoportnak.

## <a name="recommendations"></a>Javaslatok

Egy több területi architektúra biztosít magas rendelkezésre állás érdekében, mint egyetlen régióban üzembe helyezni. Ha egy regionális kimaradás érinti az elsődleges régióban, [Traffic Manager] [ traffic-manager] hogy áthelyezze a feladatokat a másodlagos régióba. Ez az architektúra segíthet is, ha az alkalmazás egy egyedi alrendszer nem tud.

Magas rendelkezésre állás elérése érdekében a különböző régiókban néhány általános módszer van: 

* A melegtartalék aktív/passzív. Forgalom kerül egy régió tartozik, a más vár melegtartalék közben. Meleg készenléti azt jelenti, hogy a virtuális gépeket, a másodlagos régióban lefoglalt és fut mindig.
* A meleg készenléti állapotban lévő aktív/passzív. Forgalom kerül egy régió tartozik, a más vár meleg készenléti közben. Meleg készenléti állapot azt jelenti, hogy a virtuális gépeket, a másodlagos régióban nem foglal le addig, amíg a feladatátvételi szükséges. Ez a megközelítés költségek kisebb futtatásához, de általában adott meghibásodás során online állapotba hosszabb ideig tart.
* Aktív. Mindkét régió aktív, és kérés kerül közöttük. Ha nem érhető el egy régió tartozik, az kívül Elforgatás lesz végrehajtva. 

A referencia-architektúrában melegtartalék feladatátvételi Traffic Manager használata az aktív/passzív összpontosít. Vegye figyelembe, hogy sikerült központi telepítése egy kis számú virtuális gépek melegtartalék és majd horizontális felskálázás igény szerint.

### <a name="regional-pairing"></a>Regionális párosítás

Minden Azure-régió, egy másik ugyanazon a földrajzi régióját párosított. Régiók általában választhat a azonos regionális pár (például USA keleti régiója 2 és Velünk központi). Ennek során a következő előnyöket nyújtja:

* A széles körű kimaradás esetén legalább egy régió kívül minden pár helyreállítási prioritása van.
* Tervezett Azure rendszerfrissítések előkészítése már megkezdődött párosított régiók egymás után, az lehetséges állásidő minimalizálása érdekében.
* Párok találhatók belül az azonos földrajzi adatok rezidens követelményeinek megfelelően. 

Győződjön meg arról, hogy mindkét régió támogatja az összes Azure-szolgáltatás szükséges az alkalmazás (lásd: [régiói][services-by-region]). Regionális párok kapcsolatos további információkért lásd: [üzleti folytonossági és vészhelyreállítási helyreállítási (BCDR): Azure-régiókat párosítva][regional-pairs].

### <a name="traffic-manager-configuration"></a>A TRAFFIC Manager-konfiguráció

A Traffic Manager konfigurálásakor, vegye figyelembe a következő szempontokat:

* **Útválasztás**. A TRAFFIC Manager támogatja több [útválasztási algoritmusok][tm-routing]. A jelen cikkben ismertetett esetben használjon *prioritás* útválasztási (korábbi nevén *feladatátvételi* útválasztási). Ezzel a beállítással a Traffic Manager minden kérést küld az elsődleges régióban, hacsak az elsődleges régióban el nem érhető el. Ezen a ponton akkor automatikusan átkerül a másodlagos régióba. Lásd: [konfigurálhatja feladatátvételi útválasztási módszer][tm-configure-failover].
* **Állapotmintáihoz**. HTTP (vagy HTTPS) használ a TRAFFIC Manager [mintavételi] [ tm-monitoring] minden egyes régió rendelkezésre állásának figyelésére. A mintavétel egy 200-as HTTP-választ a megadott URL-cím elérési keresi. Ajánlott eljárásként hozzon létre egy végpontot, amely jelent a alkalmazás általános állapotát, és használja ezt a végpontot a állapotmintáihoz. Ellenkező esetben a mintavétel előfordulhat, hogy jelentés a kifogástalan állapotú végpontok, ha az alkalmazás kritikus részei ténylegesen nem működnek. További információkért lásd: [állapotfigyelő végpont figyelési mintát][health-endpoint-monitoring-pattern].   

Amikor a Traffic Manager átadja a feladatokat a rendszer egy adott időn belül, ha ügyfelek nem tudják elérni az alkalmazást. A duration hatással van a következő tényezőket:

* A állapotmintáihoz észlelni kell, hogy az elsődleges régióban vált nem érhető el.
* DNS-kiszolgálók frissítenie kell a gyorsítótárban lévő DNS-rekordok az IP-címhez, amely a DNS--élettartama (TTL) függ. Az alapértelmezett élettartam 300 másodpercen (5 percen), de ezt az értéket a Traffic Manager-profil létrehozásakor konfigurálhatja.

További információkért lásd: [kapcsolatos Traffic Manager figyelési][tm-monitoring].

Ha a Traffic Manager átadja, ajánlott végrehajtása manuális feladat-visszavétel, nem pedig egy automatikus feladat-visszavétel végrehajtására. Ellenkező esetben hozhat létre olyan helyzet, amelyben az alkalmazás oda-vissza régiók közötti tükrözi. Ellenőrizze, hogy minden alkalmazás alrendszer visszavétele előtt kifogástalan.

Vegye figyelembe, hogy a Traffic Manager vissza alapértelmezés szerint automatikusan nem. Ennek megelőzése érdekében manuálisan a Prioritás csökkentése az elsődleges régió egy feladatátvételi esemény után. Tegyük fel például, hogy az elsődleges régióban 1 prioritású virtuális gép, és a másodlagos prioritású 2. A feladatátvétel után állítsa be az elsődleges régióban prioritás 3, hogy megakadályozza az automatikus feladatátvételi. Ha készen áll vissza, frissítse a prioritás 1.

A következő [Azure CLI] [ install-azure-cli] parancs frissíti a prioritás:

```bat
azure network traffic-manager  endpoint set --resource-group <resource-group> --profile-name <profile>
    --name <traffic-manager-name> --type AzureEndpoints --priority 3
```    

Egy másik megoldás, hogy ideiglenesen letilthatja a végpont, amíg készen feladat-visszavételt:

```bat
azure network traffic-manager  endpoint set --resource-group <resource-group> --profile-name <profile>
    --name <traffic-manager-name> --type AzureEndpoints --status Disabled
```

A feladatátvételi függően szükség lehet újratelepíteni az erőforrások régión belül. Mielőtt ismét sikertelen, egy működőképesség tesztjének elvégzéséhez. A vizsgálat ellenőrizni kell, többek között:

* Virtuális gépek helyesen vannak konfigurálva. (Az összes szükséges szoftverek telepítve van, az IIS fut, és így tovább.)
* Alkalmazás alrendszereket is megfelelő. 
* Funkcionális tesztelése. (Például az adatbázis-rétegből érhető el a webes réteg alapján.)

### <a name="configure-sql-server-always-on-availability-groups"></a>SQL Server mindig a rendelkezésre állási csoportok konfigurálása

Windows Server 2016 előtti SQL Server Always On rendelkezésre állási csoportok igényli tartományvezérlő, és a rendelkezésre állási csoport összes csomópontjának azonos Active Directory (AD) tartományban kell lennie. 

A rendelkezésre állási csoport konfigurálásához:

* Legalább két tartományvezérlő tegyen mindegyik régióhoz.
* Adjon meg egy statikus IP-cím minden tartományvezérlőn.
* A Vnetek közötti kommunikáció engedélyezésére VNet – VNet kapcsolat létrehozása.
* Minden egyes virtuális hálózat vegye fel az IP-címét a tartományvezérlők (mindkét régió) a DNS-kiszolgáló listájához. A következő parancssori parancsot használhatja. További információkért lásd: [egy virtuális hálózatot (VNet) által használt kezelése DNS-kiszolgálók][vnet-dns].

    ```bat
    azure network vnet set --resource-group dc01-rg --name dc01-vnet --dns-servers "10.0.0.4,10.0.0.6,172.16.0.4,172.16.0.6"
    ```

* Hozzon létre egy [Windows Server feladatátvételi fürtszolgáltatási] [ wsfc] (WSFC) fürt, amely tartalmazza az SQL Server-példányok mindkét régióban. 
* Hozzon létre egy SQL Server mindig a rendelkezésre állási csoportot, amely tartalmazza az SQL Server-példányok az elsődleges és másodlagos régiókban. Lásd: [kiterjesztése mindig a rendelkezésre állási csoport távoli Azure Datacenter (PowerShell)](https://blogs.msdn.microsoft.com/sqlcat/2014/09/22/extending-alwayson-availability-group-to-remote-azure-datacenter-powershell/) lépéseit.

    * Helyezze el az elsődleges másodpéldány az elsődleges régióban.
    * Egy vagy több másodlagos replikák helyezze el az elsődleges régióban. Konfigurálja azokat a szinkron véglegesítési használata automatikus feladatátvételre.
    * Egy vagy több másodlagos replikák helyezze el a másodlagos régióba. Konfigurálja azokat használni a *aszinkron* véglegesítési teljesítményének javítására szolgál. (Ellenkező esetben minden T-SQL-tranzakciót kell várnia az oda-vissza a hálózaton át a másodlagos régióba.)

    > [!NOTE]
    > Az aszinkron véglegesítésű replikák nem támogatják az automatikus feladatátvételre.
    >
    >

További információkért lásd: [fut a Windows virtuális gépek az Azure-N szintű architektúra](n-tier.md).

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

Egy összetett N szintű alkalmazást, nem kell replikálni a másodlagos régióban a teljes alkalmazás. Ehelyett csak lehet, hogy replikálja a kritikus alrendszer, az üzletmenet folytonossága támogatásához szükséges.

A TRAFFIC Manager ponttá lehetséges hiba a rendszerben. A Traffic Manager szolgáltatás nem sikerül, ha az ügyfelek nem tudnak hozzáférni az alkalmazás a leállások során. Tekintse át a [Traffic Manager SLA][tm-sla], és döntse el, hogy kizárólag a Traffic Manager segítségével megfelel-e az üzleti követelményeinek, a magas rendelkezésre állás érdekében. Ha nem, akkor vegyen fel egy másik forgalom felügyeleti megoldás, a feladat-visszavételre. Ha az Azure Traffic Manager szolgáltatás nem sikerül, módosítsa a CNAME rekordot a DNS-ben, hogy a többi felügyeleti szolgálat mutasson. (Ebben a lépésben kézzel kell elvégezni, és az alkalmazás nem lehet a DNS-módosítások továbbítódnak.)

Az SQL Server-fürt van két feladatátvételi forgatókönyvek kell figyelembe venni:

- Az SQL Server adatbázis-replikákat az elsődleges régióban összes sikertelen. Például ez akkor fordulhat regionális kimaradás során. Ebben az esetben kézi feladatátvételt kell a rendelkezésre állási csoportból, annak ellenére, hogy a Traffic Manager automatikusan átkerül a kezelőfelületre. Kövesse a [egy SQL Server rendelkezésre állási csoport kényszerített kézi feladatátvétel végrehajtása](https://msdn.microsoft.com/library/ff877957.aspx), ismertető kényszerített feladatátvételre végrehajtani az SQL Server 2016 SQL Server Management Studio, a Transact-SQL vagy a PowerShell használatával.

   > [!WARNING]
   > A kényszerített feladatátvételi fennáll az adatvesztés. Az elsődleges régióban újra online állapotba kerül, ha az adatbázis, és felhasználása [tablediff] különbségek kereséséhez.
   >
   >
- A TRAFFIC Manager átadja a feladatokat a másodlagos régióba, de az elsődleges SQL Server adatbázis-replika továbbra is elérhető. Például az előtér-réteg sikertelen lehet, az nem befolyásolja az SQL Server virtuális gépen. Ebben az esetben internetes továbbítódik a másodlagos régióban, és adott régióban még lehet kapcsolódni az elsődleges másodpéldány. Azonban nem lesznek nagyobb késéseket, mert az SQL Serverhez való csatlakozást fog régiók között. Ebben a helyzetben végre kell hajtania a manuális feladatátvételt az alábbiak szerint:

   1. Ideiglenes Váltás az SQL Server adatbázis-replikát a másodlagos régióban *szinkron* véglegesítési. Ez biztosítja, nem lesznek adatvesztés a feladatátvétel során.
   2. Feladatok átadása a, hogy a replika.
   3. Ha nem vissza az elsődleges régióban, a aszinkron véglegesítési beállítás visszaállításához.

## <a name="manageability-considerations"></a>Felügyeleti szempontok

Ha a telepítés frissítéséhez frissíteni egy régió tartozik csökkenti annak esélyét, a globális hibák egy helytelen konfiguráció vagy az alkalmazás hiba egyszerre.

Tesztelje a rugalmasság, a rendszer a hibák. Az alábbiakban néhány gyakori hiba helyzetek ellenőrzéséhez:

* Állítsa le a Virtuálisgép-példányok.
* Nyomás erőforrások, például CPU és memória.
* Hálózati kapcsolat bontása/késleltetés.
* Összeomlás-folyamatokat.
* Tanúsítványok lejárnak.
* Hardver hibák szimulálásához.
* Állítsa le a DNS-szolgáltatás a tartományvezérlőkön.

A helyreállítási idő mérését, és győződjön meg arról, hogy megfelelnek-e az üzleti igényeknek. Teszt sikertelen üzemmódot, valamint kombinációi.



<!-- Links -->
[hybrid-vpn]: ../hybrid-networking/vpn.md

[azure-sla]: https://azure.microsoft.com/support/legal/sla/
[azure-sql-db]: https://azure.microsoft.com/documentation/services/sql-database/
[health-endpoint-monitoring-pattern]: https://msdn.microsoft.com/library/dn589789.aspx
[install-azure-cli]: /azure/xplat-cli-install
[regional-pairs]: /azure/best-practices-availability-paired-regions
[resource groups]: /azure/azure-resource-manager/resource-group-overview
[resource-group-links]: /azure/resource-group-link-resources
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[services-by-region]: https://azure.microsoft.com/regions/#services
[sql-always-on]: https://msdn.microsoft.com/library/hh510230.aspx
[tablediff]: https://msdn.microsoft.com/library/ms162843.aspx
[tm-configure-failover]: /azure/traffic-manager/traffic-manager-configure-failover-routing-method
[tm-monitoring]: /azure/traffic-manager/traffic-manager-monitoring
[tm-routing]: /azure/traffic-manager/traffic-manager-routing-methods
[tm-sla]: https://azure.microsoft.com/support/legal/sla/traffic-manager/v1_0/
[traffic-manager]: https://azure.microsoft.com/services/traffic-manager/
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[vnet-dns]: /azure/virtual-network/virtual-networks-manage-dns-in-vnet
[vnet-to-vnet]: /azure/vpn-gateway/vpn-gateway-vnet-vnet-rm-ps
[vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[wsfc]: https://msdn.microsoft.com/library/hh270278.aspx

[0]: ./images/multi-region-application-diagram.png "Azure N szintű alkalmazásokhoz magas rendelkezésre állású hálózati architektúra"
