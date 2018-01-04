---
title: "Alapszintű webalkalmazás"
description: "A Microsoft Azure-ban futó webalkalmazás alapvető ajánlott architektúra."
author: MikeWasson
ms.date: 12/12/2017
cardTitle: Basic web application
ms.openlocfilehash: 598eb547f0e96ae334af391183a792637caa8631
ms.sourcegitcommit: 1c0465cea4ceb9ba9bb5e8f1a8a04d3ba2fa5acd
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/02/2018
---
# <a name="basic-web-application"></a>Alapszintű webalkalmazás
[!INCLUDE [header](../../_includes/header.md)]

A referencia-architektúrában jeleníti meg a webes alkalmazás esetén bevált gyakorlatok csoportja [Azure App Service] [ app-service] és [Azure SQL Database] [ sql-db]. [**A megoldás üzembe helyezéséhez.**](#deploy-the-solution)

![[0]][0]

*Töltse le az architektúra [Visio-fájlját][visio-download].*

## <a name="architecture"></a>Architektúra 

> [!NOTE]
> Ez az architektúra nem összpontosítani alkalmazásfejlesztés, és nem feltételezi azt minden egyes alkalmazás-keretrendszer. Az célja annak megértése, hogyan különböző Azure-szolgáltatások működnek együtt.
>
>

Az architektúra a következő részből áll:

* **Erőforráscsoport**. A [erőforráscsoport](/azure/azure-resource-manager/resource-group-overview) Azure-erőforrások logikai tárolója.

* **App Service alkalmazás**. [Az Azure App Service] [ app-service] egy teljes körűen felügyelt platform létrehozására és központi telepítésére a felhőalapú alkalmazásokhoz.     

* **App Service-csomag**. Egy [App Service-csomag] [ app-service-plans] biztosít a felügyelt üzemeltető virtuális gépek (VM) az alkalmazás. Futtassa az ugyanazon Virtuálisgép-példányok a csomagot hozzárendelt összes alkalmazáshoz.

* **Üzembe helyezési**.  A [üzembe helyezési pont] [ deployment-slots] lehetővé teszi, hogy a központi telepítés tesztelése és felcserélni az éles üzemelő példányhoz. Így elkerülheti a közvetlenül éles környezetben üzembe helyezése. Tekintse meg a [kezelhetőségi](#manageability-considerations) szakasz konkrét javaslatokért.

* **IP-cím**. Az App Service alkalmazás rendelkezik, egy nyilvános IP-cím és tartománynév. A tartománynév altartománya `azurewebsites.net`, például a `contoso.azurewebsites.net`.  

* **Az Azure DNS**. [Az Azure DNS] [ azure-dns] üzemeltetési szolgáltatás DNS-tartományok biztosítani a névfeloldást a Microsoft Azure-infrastruktúra használatával. Ha tartományait az Azure-ban üzemelteti, DNS-rekordjait a többi Azure-szolgáltatáshoz is használt hitelesítő adatokkal, API-kkal, eszközökkel és számlázási információkkal kezelheti. Egy egyéni tartománynevet használni (például `contoso.com`), az egyéni tartománynév leképezése az IP-cím DNS-rekordok létrehozása. További információkért lásd: [egyéni tartománynév beállítása az Azure App Service][custom-domain-name].  

* **Azure SQL Database** [SQL-adatbázis] [ sql-db] egy relációs adatbázis-a-szolgáltatás a felhőben van.

* **A logikai kiszolgáló**. Az Azure SQL Database logikai kiszolgáló üzemelteti az adatbázisokat. Egy logikai kiszolgálón több adatbázist hozhat létre.

* **Azure Storage**. Hozzon létre egy Azure storage-fiók egy blob-tároló diagnosztikai naplók tárolásához.

* **Az Azure Active Directory** (az Azure AD). Használja az Azure AD vagy egy másik identitásszolgáltató-hitelesítéshez.

## <a name="recommendations"></a>Ajánlatok

Az Ön követelményei eltérhetnek az itt leírt architektúrától. A javaslatok használja ebben a szakaszban kiindulási pontként.

### <a name="app-service-plan"></a>App Service-csomagot
A Standard vagy prémium rétegek akkor használható, mert támogatják a bővített kapacitású, automatikus skálázás, és a secure sockets layer (SSL). Minden egyes réteg támogatja több *példány mérete* magok száma és memória, amely eltérő. Terv létrehozása után módosíthatja a réteg vagy a példány mérete. App Service-csomagokról kapcsolatos további információkért lásd: [App Service szolgáltatás díjszabása][app-service-plans-tiers].

Van szó, a példányok az App Service-csomag, még akkor is, ha az alkalmazás leállt. Ügyeljen arra, hogy terveket, amelyeket nem használ (például a próbatelepítések) törlése.

### <a name="sql-database"></a>SQL Database
Használja a [12-es verzióra] [ sql-db-v12] SQL-adatbázis. SQL-adatbázis támogatja a Basic, Standard és Premium [szolgáltatásszintek][sql-db-service-tiers], több teljesítménnyel belül az egyes rétegek szintek mért [adatbázis-tranzakciós egységek (dtu-k)][sql-dtu]. Kapacitástervezés, és válassza ki azt, amelyik megfelel a követelményeknek és teljesítményszintet szintet.

### <a name="region"></a>Régió
Az App Service-csomag és az SQL-adatbázis ugyanabban a régióban, hálózati késés csökkentése érdekében érdemes telepíteni. Általában válassza ki a felhasználók a legközelebb eső régiót.

Az erőforráscsoport egy régiót, amely megadja a központi telepítési metaadatok tárolására is rendelkezik. Helyezze el az erőforráscsoportot és az erőforrások ugyanabban a régióban. Ez javíthatja a rendelkezésre állási üzembe helyezése során. 

## <a name="scalability-considerations"></a>Méretezési szempontok

Egy Azure App Service fő előnye az alkalmazás terhelés alapján méretezési képességét. Az alábbiakban néhány szempontot szem előtt tartani az alkalmazás horizontális tervezése során.

### <a name="scaling-the-app-service-app"></a>Az App Service alkalmazás skálázás

Egy App Service-alkalmazást méretezési két módja van:

* *Vertikális felskálázás*, ami azt jelenti, hogy a példány méretének módosítása. A példány mérete határozza meg a memória, a magok száma, és minden egyes Virtuálisgép-példány tárolója. Legfeljebb manuálisan példányméretének vagy a csomag réteg módosításával.  

* *Horizontális felskálázás*, ami azt jelenti, hogy a megnövekedett terhelés kezelése hozzáadása. Minden tarifacsomag rendelkezik példányok maximális száma. 

  Horizontális felskálázás kézi a példányszámot módosításával, vagy használjon [automatikus skálázás] [ web-app-autoscale] való automatikusan és a egy ütemezést és/vagy a teljesítmény metrikák példányainak törlése. Minden egyes skálázási művelet gyorsan&mdash;általában másodpercen belül. 

  Ahhoz, hogy az automatikus skálázást, hozzon létre egy automatikus skálázás *profil* , amely meghatározza, hogy a példányok minimális és maximális számát. Profilok ütemezhető. Létrehozhat például különálló profilok létrehozását és tartozik. Szükség esetén a profil hozzáadásához vagy eltávolításához példányok a szabályokat tartalmaz. (Példa: két példányt hozzáadása, ha CPU-használat 70 % feletti 5 perc.)
  
A webes alkalmazás méretezéshez ajánlásokat:

* Amennyire csak lehet ne használjon skálázás felfelé és lefelé, mert azt indíthatnak újra kell indítani az alkalmazást. Ehelyett válassza ki a réteg és méretét, hogy alkalmazkodjanak a teljesítménykövetelményekhez szokásos terhelés és majd terjessze ki a példányok adatforgalma változásainak kezeléséhez.    
* Automatikus skálázás engedélyezéséhez. Ha az alkalmazás egy előre jelezhető, rendszeres munkaterhelés, hozzon létre a profilt ütemezése előre a példányok számát. Ha a munkaterhelés nem előre jelezhető, akkor a szabály-alapú automatikus skálázás reagálni a terhelés módosításokat, azok bekövetkezésekor. Mindkét megközelítés kombinálhatja.
* CPU-használat általában jó metrikáját automatikus skálázási szabályok. Azonban be kell tölteni az alkalmazás teszteléséhez, potenciális szűk keresztmetszetek azonosítása és az automatikus skálázási szabályok alapjául az adatokat.  
* Automatikus skálázási szabályok a következők egy *várakozási* időszak, amely egy új skálázási művelet indítása előtt a méretezési művelet befejeződése után várakozási időt. A várakozási időszak lehetővé teszi, hogy a rendszer stabilizálását előtt skálázás megismételni. Állítsa be a példány és egy már várakozási időszak a példányok eltávolításához hozzáadásához rövidebb várakozási időszakot. Állítsa be például a 5 perc – adjon hozzá egy, de a példány eltávolítása 60 perc. Érdemes gyorsan számára a további forgalom kezelésére, és fokozatosan méretezését vissza túl nagy terhelés alatt új példányok felvételéhez.

### <a name="scaling-sql-database"></a>SQL-adatbázis méretezése
Ha egy magasabb réteg vagy a teljesítmény szolgáltatásszint SQL-adatbázis, legfeljebb alkalmazásleállás elkerülése az egyes adatbázisokat. További információkért lásd: [SQL Database beállításai és teljesítménye: az egyes szolgáltatásszinteken elérhető][sql-db-scale].

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok
Írásának időpontjában a szolgáltatásiszint-szerződés (SLA) App Service 99,95 % pedig az SLA-t az SQL Database esetén a Basic, Standard és Premium szintjeinek 99,99 %. 

> [!NOTE]
> Az App Service SLA-t is egyetlen és több példány vonatkozik.  
>
>

### <a name="backups"></a>A hivatkozás a Safari böngészőben nem nyitható meg. Lehet, hogy a Safari használata korlátozva van a beállítások között, vagy a rendszergazda letiltotta a hozzáférést.
Adatvesztés esetén az SQL-adatbázis biztosít pont időponthoz kötött visszaállítás és georedundáns helyreállítás. Ezeket a funkciókat minden csomagban elérhető, és automatikusan engedélyezve van. Nem kell ütemezni, vagy a biztonsági mentéseit. 

- Használja a visszaállításhoz időpontban [emberi tévedések helyreállíthatók] [ sql-human-error] vissza az adatbázist egy korábbi időpontbeli időben. 
- Használja a georedundáns helyreállítás [szolgáltatáskimaradás helyreállíthatók] [ sql-outage-recovery] állítja vissza egy adatbázist egy georedundáns biztonsági másolatból. 

További információkért lásd: [üzleti folytonossági és az adatbázis katasztrófa utáni helyreállítás az SQL Database Felhőbeli][sql-backup].

App Service biztosít egy [biztonsági mentése és visszaállítása] [ web-app-backup] szolgáltatást ahhoz, hogy az alkalmazásfájlokat. Azonban ügyeljen arra, hogy a biztonságimásolat-fájlok tartalmazzák alkalmazásbeállítások egyszerű szövegként, és ezek közé tartozhatnak a titkos kulcsokat, például a kapcsolati karakterláncokat. Kerülje az alkalmazás biztonsági mentési szolgáltatás biztonsági mentése az SQL-adatbázisok, mert azt az adatbázis egy SQL .bacpac fájlba exportálja fel [dtu-k][sql-dtu]. Ehelyett használja a fenti SQL-adatbázis időpontban visszaállítás.

## <a name="manageability-considerations"></a>Felügyeleti szempontok
Termelési környezetben, fejlesztési, külön erőforráscsoportokat létrehozni, és tesztelési környezetben. Ez megkönnyíti a központi telepítések felügyeletéhez szükséges, törölje a próbatelepítések és hozzáférési engedélyek hozzárendelése.

Erőforrások erőforráscsoportok rendelése, vegye figyelembe a következőket:

* Életciklusát. Az azonos életciklussal rendelkező erőforrásokat általában üzembe ugyanabban az erőforráscsoportban.
* A hozzáférés. Használhat [szerepköralapú hozzáférés-vezérlés] [ rbac] (RBAC) hozzáférési házirendek alkalmazása az erőforrások csoportba.
* Számlázási. Megtekintheti az összegzett költségeinek ahhoz az erőforráscsoporthoz.  

További információk: [Azure Resource Manager overview](/azure/azure-resource-manager/resource-group-overview) (Az Azure Resource Manager áttekintése).

### <a name="deployment"></a>Üzembe helyezés
Telepítés két lépésből áll:

1. Az Azure-erőforrások kiépítése. Azt javasoljuk, hogy használjon [Azure Resoure Manager-sablonok] [ arm-template] ehhez a lépéshez. Sablonok megkönnyítik a PowerShell vagy az Azure parancssori felület (CLI) használatával központi telepítések automatizálásához.
2. (A kódot, a bináris fájljait és a tartalomfájlok) az alkalmazás telepítéséhez. Több lehetőség közül választhat, beleértve a helyi Git-tárház telepítése a Visual Studio vagy felhőalapú verziókövetésből folyamatos üzembe helyezést. Lásd: [az alkalmazás telepítése az Azure App Service][deploy].  

Egy App Service-alkalmazást mindenhol nevű egy üzembe helyezési pont `production`, amely jelöli az élő munkakörnyezeti helyet. Ajánlott létrehozni egy átmeneti helyet frissítéseinek telepítéséhez. Az előkészítési pont használatának előnyei a következők:

* A telepítés sikeres volt, mielőtt az éles környezetben csere ellenőrizheti.
* Átmeneti pont telepítése biztosítja, hogy a példányainak Mielőtt éles környezetben felcserélés folyamatban vannak tárolóhelyspecifikus. Számos alkalmazás jelentős melegítési és cold-a kezdési idő rendelkeznek.

Javasoljuk továbbá, egy harmadik tárolóhely ahhoz, hogy a legutóbbi helyes központi telepítés létrehozása. Átmeneti és üzemi cserél, miután az utolsó helyes tárhely helyezze át a korábbi éles környezetben (amely most a tesztelési). Így ha később, a probléma felderítéséhez gyorsan visszatérhet a legutóbbi helyes verzióját.

![[1]][1]

Visszaállítás egy korábbi verzióját esetén győződjön meg róla adatbázis séma módosításait visszamenőlegesen kompatibilis.

Ne használjon üzembe helyezési ponti a éles telepítés tesztelése, mert belül az azonos App Service-csomag összes alkalmazás ugyanazon Virtuálisgép-példányára. Például terhelés tesztek csökkentheti a élő munkakörnyezeti helyet. Ehelyett hozzon létre külön App Service-csomagokról éles és tesztelési. Tegyen próbatelepítést egy külön tervbe, hogy ezeket elszigeteli éles verzióját.

### <a name="configuration"></a>Konfiguráció
Tárolja a konfigurációs beállításokat [Alkalmazásbeállítások][app-settings]. Adja meg az alkalmazás beállításaiban a Resource Manager-sablonok vagy a PowerShell használatával. Alkalmazása futásidőben Alkalmazásbeállítások érhetők el az alkalmazásra, amely a környezeti változók.

Soha ne ellenőrizze jelszavakat, a tárelérési kulcsok és a kapcsolati karakterláncok forrás vezérlőbe. Ehelyett át paraméterekként ezeket a telepítési parancsfájlt, amely tárolja ezeket az értékeket Alkalmazásbeállítások.

Felcserélés egy üzembe helyezési pont, amikor az alkalmazás beállításaiban alapértelmezés szerint van cserélve. Ha eltérő beállításokat az üzemi és átmeneti, tárhely anyagot, és nem get felcserélve Alkalmazásbeállítások is létrehozhat.

### <a name="diagnostics-and-monitoring"></a>Diagnosztika és monitorozás
Engedélyezése [diagnosztikai naplózás][diagnostic-logs]alkalmazásnaplózás és a webkiszolgáló naplózásának is beleértve. A Blob storage használata naplózásának konfigurálása. A megfelelő teljesítmény érdekében hozzon létre egy külön tárfiókot a diagnosztikai naplókat. Ne használja ugyanazt a tárfiókot naplókat és az alkalmazásadatokat. Részletes útmutatás a naplózást, lásd: [megfigyelési és diagnosztikai útmutatást][monitoring-guidance].

Használjon, mint a szolgáltatást [New Relic] [ new-relic] vagy [Application Insights] [ app-insights] alkalmazás teljesítményének figyelése és a terhelés viselkedését. Vegye figyelembe a [adatok sebességhatárok] [ app-insights-data-rate] az Application insights szolgáltatással.

Tesztelheti az terhelés, például egy eszközzel [Visual Studio Team Services][vsts]. A felhőalapú alkalmazások teljesítményét elemző általános áttekintést, lásd: [teljesítményét elemző ismertetése][perf-analysis].

Az alkalmazás hibaelhárítási tippeket:

* Használja a [panel hibaelhárítása] [ troubleshoot-blade] az Azure-portálon keresse meg a megoldást a problémára.
* Engedélyezése [napló streaming] [ web-app-log-stream] közel valós idejű naplóinformációk megjelenítéséhez.
* A [Kudu irányítópult] [ kudu] rendelkezik figyelési és az alkalmazás hibakeresése eszközöket. További információkért lásd: [Azure Websitesra online eszközök kell tudni] [ kudu] (blogbejegyzés). A Kudu irányítópult Azure-portálról érhető el. Nyissa meg az alkalmazáshoz, majd kattintson a panel **eszközök**, majd kattintson a **Kudu**.
* Ha a Visual Studio használata esetén olvassa el [hibaelhárítása a webes alkalmazás az Azure App Service szolgáltatásban a Visual Studio használatával] [ troubleshoot-web-app] Hibakeresés és a hibaelhárítási tippek.

## <a name="security-considerations"></a>Biztonsági szempontok
Ez a szakasz ebben a cikkben leírt Azure-szolgáltatásokhoz való vonatkozó biztonsági szempontokat sorolja fel. Ajánlott biztonsági eljárások teljes listáját nincs. További biztonsági szempontokat lásd: [az Azure App Service alkalmazás biztonságos][app-service-security].

### <a name="sql-database-auditing"></a>SQL Database auditing
Naplózás segít törvényi megfelelőség fenntartásában és az eltérések és rendellenességek, amelyek üzleti problémát jelenthetnek, vagy a biztonság megsértésére megismerése. Lásd: [Ismerkedés az SQL-adatbázis naplózásának][sql-audit].

### <a name="deployment-slots"></a>Üzembe helyezési tárhelyek
Minden egyes üzembe helyezési pont egy nyilvános IP-címmel rendelkezik. Az éles üzembe helyezési ponti használatával biztonságos [Azure Active Directory bejelentkezési] [ aad-auth] , hogy csak a fejlesztési és DevOps csapat tagjai ezekre a végpontokra érhető el.

### <a name="logging"></a>Naplózás
Naplók soha nem rögzíteni kell a felhasználói jelszavakat és egyéb adatait, amelyik alkalmas lehet identitás csalás véglegesítéséhez. Megtisztítás a adatokból adatai előtt tárolja.   

### <a name="ssl"></a>SSL
Egy App Service-alkalmazást tartalmaz olyan altartomány, SSL végpontja `azurewebsites.net` minden további költség nélkül. Az SSL-végpontot tartalmaz, a helyettesítő tanúsítvány a `*.azurewebsites.net` tartomány. Ha egy egyéni tartománynevet használ, meg kell adnia az egyéni tartomány megfelelő tanúsítványt. A legegyszerűbb vásárolhat egy tanúsítványt közvetlenül az Azure portálon keresztül. Tanúsítványokat más hitelesítésszolgáltatótól is importálhat. További információkért lásd: [vásárolnia és SSL-tanúsítvány konfigurálása az Azure App Service][ssl-cert].

Biztonsági szempontból ajánlott az alkalmazás érvényesítenie kell a HTTPS irányítják át HTTP-kérelmekre. Az alkalmazáson belüli megvalósításának, vagy egy URL-cím átdolgozás szabály használata a [HTTPS engedélyezése az Azure App Service alkalmazás][ssl-redirect].

### <a name="authentication"></a>Hitelesítés
Javasoljuk, hogy hitelesítése keresztül az identitásszolgáltató (IDP), például Azure AD Facebook, Google, vagy Twitter. OAuth 2 vagy OpenID Connect (OIDC) használ a hitelesítési folyamat. Az Azure AD felhasználók és csoportok kezelése, alkalmazás szerepköröket hozhat létre, a helyszíni identitások integrálása és háttér szolgáltatásait például az Office 365 és a Skype vállalati lehetőségeket kínál.

Kerülje az alkalmazás felhasználói bejelentkezést és a hitelesítő adatok kezelése közvetlenül, mivel az hozza létre a potenciális támadási felületet.  Ajánlott legalább kellene e-mailes megerősítés, a jelszó-helyreállítás és a többtényezős hitelesítés; jelszó erőssége; érvényesítése és azok kivonatai tárolja biztonságos helyen. A nagy identitás-szolgáltatóktól kezelni tudja a ezeket a beállításokat, és folyamatosan figyelése és javítása a biztonsági eljárásokat.

Érdemes lehet [App Service hitelesítés] [ app-service-auth] az OAuth/OIDC hitelesítési folyamat végrehajtásához. Az App Service hitelesítés előnyei a következők:

* Könnyen konfigurálható.
* Nincs kód egyszerű hitelesítési forgatókönyvek szükség.
* Támogatja az OAuth jogkivonatot használja a felhasználó nevében erőforrást engedélyezési meghatalmazott.
* Itt egy beépített jogkivonatok gyorsítótárát.

App Service hitelesítés korlátozásai:  

* Korlátozott testreszabási beállításait.
* Delegált engedélyezési korlátozódik bejelentkezési munkamenetenként egy háttér-erőforrást.
* Ha egynél több IDP használ, nincs olyan beépített mechanizmus a hitelesítőtartomány feltárásához.
* Több-bérlős forgatókönyvek esetén az alkalmazás ellenőrzése a jogkivonat-kibocsátó programot kell megvalósítania.

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése
Ez az architektúra Resoure Manager sablon egy példa [elérhető a Githubon][paas-basic-arm-template].

A sablon PowerShell használatával történő üzembe helyezéséhez futtassa a következő parancsokat:

```
New-AzureRmResourceGroup -Name <resource-group-name> -Location "West US"

$parameters = @{"appName"="<app-name>";"environment"="dev";"locationShort"="uw";"databaseName"="app-db";"administratorLogin"="<admin>";"administratorLoginPassword"="<password>"}

New-AzureRmResourceGroupDeployment -Name <deployment-name> -ResourceGroupName <resource-group-name> -TemplateFile .\PaaS-Basic.json -TemplateParameterObject  $parameters
```

További információkért lásd: [telepítése Azure Resource Manager-sablonok erőforrások][deploy-arm-template].

<!-- links -->

[aad-auth]: /azure/app-service-mobile/app-service-mobile-how-to-configure-active-directory-authentication
[app-insights]: /azure/application-insights/app-insights-overview
[app-insights-data-rate]: /azure/application-insights/app-insights-pricing
[app-service]: https://azure.microsoft.com/documentation/services/app-service/
[app-service-auth]: /azure/app-service-api/app-service-api-authentication
[app-service-plans]: /azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview
[app-service-plans-tiers]: https://azure.microsoft.com/pricing/details/app-service/
[app-service-security]: /azure/app-service-web/web-sites-security
[app-settings]: /azure/app-service-web/web-sites-configure
[arm-template]: /azure/azure-resource-manager/resource-group-overview#resource-groups
[azure-dns]: /azure/dns/dns-overview
[custom-domain-name]: /azure/app-service-web/web-sites-custom-domain-name
[deploy]: /azure/app-service-web/web-sites-deploy
[deploy-arm-template]: /azure/resource-group-template-deploy
[deployment-slots]: /azure/app-service-web/web-sites-staged-publishing
[diagnostic-logs]: /azure/app-service-web/web-sites-enable-diagnostic-log
[kudu]: https://azure.microsoft.com/blog/windows-azure-websites-online-tools-you-should-know-about/
[monitoring-guidance]: ../../best-practices/monitoring.md
[new-relic]: http://newrelic.com/
[paas-basic-arm-template]: https://github.com/mspnp/reference-architectures/tree/master/managed-web-app/basic-web-app/Paas-Basic/Templates
[perf-analysis]: https://github.com/mspnp/performance-optimization/blob/master/Performance-Analysis-Primer.md
[rbac]: /azure/active-directory/role-based-access-control-what-is
[resource-group]: /azure/azure-resource-manager/resource-group-overview
[sla]: https://azure.microsoft.com/support/legal/sla/
[sql-audit]: /azure/sql-database/sql-database-auditing-get-started
[sql-backup]: /azure/sql-database/sql-database-business-continuity
[sql-db]: https://azure.microsoft.com/documentation/services/sql-database/
[sql-db-overview]: /azure/sql-database/sql-database-technical-overview
[sql-db-scale]: /azure/sql-database/sql-database-service-tiers#scaling-up-or-scaling-down-a-single-database
[sql-db-service-tiers]: /azure/sql-database/sql-database-service-tiers
[sql-db-v12]: /azure/sql-database/sql-database-features
[sql-dtu]: /azure/sql-database/sql-database-service-tiers
[sql-human-error]: /azure/sql-database/sql-database-business-continuity#recover-a-database-after-a-user-or-application-error
[sql-outage-recovery]: /azure/sql-database/sql-database-business-continuity#recover-a-database-to-another-region-from-an-azure-regional-data-center-outage
[ssl-redirect]: /azure/app-service-web/web-sites-configure-ssl-certificate#bkmk_enforce
[sql-resource-limits]: /azure/sql-database/sql-database-resource-limits
[ssl-cert]: /azure/app-service-web/web-sites-purchase-ssl-web-site
[troubleshoot-blade]: https://azure.microsoft.com/updates/self-service-troubleshooting-for-app-service-web-apps-customers/
[troubleshoot-web-app]: /azure/app-service-web/web-sites-dotnet-troubleshoot-visual-studio
[visio-download]: https://archcenter.azureedge.net/cdn/app-service-reference-architectures.vsdx
[vsts]: https://www.visualstudio.com/features/vso-cloud-load-testing-vs.aspx
[web-app-autoscale]: /azure/app-service-web/web-sites-scale
[web-app-backup]: /azure/app-service-web/web-sites-backup
[web-app-log-stream]: /azure/app-service-web/web-sites-enable-diagnostic-log#streamlogs
[0]: ./images/basic-web-app.png "Alapszintű Azure webalkalmazás architektúrája"
[1]: ./images/paas-basic-web-app-staging-slots.png "Üzembe helyezési ponti csere üzemi és átmeneti példányai"
