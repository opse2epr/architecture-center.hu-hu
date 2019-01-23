---
title: Alapszintű webalkalmazás
titleSuffix: Azure Reference Architectures
description: Azure-ban futó alapszintű webalkalmazásokhoz javasolt architektúra.
author: MikeWasson
ms.date: 12/12/2017
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seodec18
ms.openlocfilehash: 95f634284fe821386704174894a85a4dbca815f7
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54485077"
---
# <a name="run-a-basic-web-application-in-azure"></a>Alapszintű webalkalmazás futtatása az Azure-ban

Ez a referenciaarchitektúra bemutatja a bevált eljárásokat használó webalkalmazás [Azure App Service] [ app-service] és [Azure SQL Database] [ sql-db]. [**A megoldás üzembe helyezése.**](#deploy-the-solution)

![Egy alapszintű webalkalmazást az Azure-ban a referencia-architektúra](./images/basic-web-app.png)

*Töltse le az architektúra [Visio-fájlját][visio-download].*

## <a name="architecture"></a>Architektúra

> [!NOTE]
> Ez az architektúra nem az alkalmazásfejlesztésre összpontosít, és nem feltételezi semmilyen konkrét alkalmazás-keretrendszer használatát. A célja, hogy ismertesse a különböző Azure-szolgáltatások együttes használatát.
>

Az architektúra a következő összetevőkből áll:

- **Erőforráscsoport**. Az [erőforráscsoport](/azure/azure-resource-manager/resource-group-overview) az Azure-erőforrások logikai tárolója.

- **App Service-alkalmazás**. Az [Azure App Service][app-service] egy teljes körűen felügyelt platform felhőalapú alkalmazások létrehozásához és üzembe helyezéséhez.

- **App Service-csomag**. Az [App Service-csomag][app-service-plans] biztosítja az alkalmazást futtató felügyelt virtuális gépeket (VM). Az egy adott csomaghoz tartozó alkalmazások mindig ugyanazokon a virtuálisgép-példányokon futnak.

- **Üzembehelyezési pontok**.  Az [üzembehelyezési pontok][deployment-slots] lehetővé teszik egy üzemelő példány előkészítését, majd áthelyezését az éles környezetbe. Így elkerülhető a közvetlenül az éles környezetbe való üzembe helyezés. Konkrét javaslatokért tekintse meg a [Felügyeleti szempontok](#manageability-considerations) szakaszt.

- **IP-cím**. Az App Service-alkalmazás rendelkezik egy nyilvános IP-címmel és egy tartománynévvel. A tartománynév az `azurewebsites.net` altartománya, például `contoso.azurewebsites.net`.

- **Azure DNS**. Az [Azure DNS][azure-dns] egy üzemeltetési szolgáltatás, amely a Microsoft Azure infrastruktúráját használja a DNS-tartományok névfeloldásához. Ha tartományait az Azure-ban üzemelteti, DNS-rekordjait a többi Azure-szolgáltatáshoz is használt hitelesítő adatokkal, API-kkal, eszközökkel és számlázási információkkal kezelheti. Egyéni tartománynév (például `contoso.com`) használatához hozzon létre DNS-rekordokat, amelyek leképezik az egyéni tartománynevet az IP-címre. További információkat az [egyéni tartománynevek az Azure App Service-ben való konfigurálásával][custom-domain-name] kapcsolatos cikkben olvashat.

- **Azure SQL Database** Az [SQL Database][sql-db] egy felhőben futó, szolgáltatásként nyújtott relációs adatbázis. Az SQL Database kódbázisa a Microsoft SQL Server adatbázismotorral. Az alkalmazás követelményeitől függően is használhatja [, Azure Database for MySQL](/azure/mysql) vagy [, Azure Database for PostgreSQL](/azure/postgresql). Ezek a teljes körűen felügyelt adatbázis-szolgáltatások, a nyílt forráskódú MySQL-kiszolgáló és a Postgres adatbázismotorokat, illetve alapján.

- **Logikai kiszolgáló**. Az Azure SQL Database-ben egy logikai kiszolgáló üzemelteti az adatbázisokat. Logikai kiszolgálónként több adatbázis is létrehozható.

- **Azure Storage**. Hozzon létre egy Azure Storage-fiókot és egy blobtárolót a diagnosztikai naplók tárolásához.

- **Azure Active Directory (Azure AD)**. A hitelesítéshez használja az Azure AD-t vagy egy másik identitásszolgáltatót.

## <a name="recommendations"></a>Javaslatok

Az Ön követelményei eltérhetnek az itt leírt architektúrától. A jelen szakaszban leírt javaslatokat tekintse kiindulópontnak.

### <a name="app-service-plan"></a>App Service-csomag

Használjon standard vagy prémium szintű csomagokat, amelyek támogatják a horizontális felskálázást, az automatikus skálázást és az SSL-t. Az egyes szintek több különféle *példányméretet* támogatnak, amelyek eltérő mennyiségű processzormagot és memóriát biztosítanak. A csomag létrehozása után módosíthatja a szintet és a példányméretet. További információk az App Service-csomagokról: [App Service – díjszabás][app-service-plans-tiers].

A számlázás az App Service-csomagban található példányok alapján történik, még akkor is, ha az alkalmazás leállt. Ügyeljen arra, hogy a nem használt csomagokat (például a tesztelési célú telepítéseket) törölje.

### <a name="sql-database"></a>SQL Database

Használja az SQL Database [12-es verzióját][sql-db-v12]. Az SQL Database az alapszintű, a standard és a prémium [szolgáltatásszintet][sql-db-service-tiers] támogatja. Mindegyik szinten több teljesítményszint érhető el, amelyek mérése [adatbázis-tranzakciós egységben (DTU-ban)][sql-dtu] történik. Tervezze meg a kapacitást, és válasszon az igényeinek megfelelő szolgáltatásszintet és teljesítményszintet.

### <a name="region"></a>Régió

Telepítse az App Service-csomagot és az SQL Database-t ugyanabban a régióban a hálózati késés minimalizálása érdekében. Általában érdemes a felhasználókhoz legközelebbi régiót választani.

Az erőforráscsoportnak szintén van régiója, amely azt adja meg, hogy a telepítés metaadatai hol vannak tárolva. Az erőforráscsoportot és a hozzá tartozó erőforrásokat helyezze ugyanabba a régióba. Ez javíthatja a rendelkezésre állást az üzembe helyezés során.

## <a name="scalability-considerations"></a>Méretezési szempontok

Az Azure App Service egyik fő előnye az alkalmazások skálázásának lehetősége a terhelés alapján. A következő szempontokat kell szem előtt tartani az alkalmazás skálázásának tervezésekor.

### <a name="scaling-the-app-service-app"></a>Az App Service-alkalmazás skálázása

Az App Service-alkalmazások skálázásának két módja van:

- *Vertikális felskálázás*, azaz a példány méretének módosítása. A példány mérete határozza meg a memória mennyiségét, a magok számát és az egyes virtuálisgép-példányokon rendelkezésre álló tároló mennyiségét. Vertikálisan felskálázni manuálisan, a példányméret vagy a csomagszint módosításával lehet.

- *Horizontális felskálázás*, azaz további példányok hozzáadása a megnövekedett terhelés kezeléséhez. Minden tarifacsomaghoz meg van adva a példányok maximális száma.

  A horizontális felskálázás elvégezhető a példányszám manuális módosításával vagy [automatikus skálázással][web-app-autoscale], amikor az Azure egy ütemezés és/vagy teljesítménymetrikák alapján automatikusan hozzáad vagy eltávolít példányokat a csomagból. Az egyes skálázási műveletek gyorsan, általában másodpercek alatt lezajlanak.

  Az automatikus skálázás engedélyezéséhez hozzon létre egy automatikus skálázási *profilt*, amely meghatározza a példányok minimális és maximális számát. A profilok ütemezhetőek. Létrehozhat például külön profilokat munkanapokra és hétvégére. A profilok arra vonatkozó szabályokat is tartalmazhatnak, hogy mikor kell példányokat hozzáadni vagy eltávolítani. (Példa: Két példány hozzáadása, ha CPU-használat 70 % fölé emelkedik 5 perce.)

Javaslatok webalkalmazások skálázásához:

- Ha lehet, kerülje a vertikális fel- és leskálázást, mert ez az alkalmazás újraindítását okozhatja. Ehelyett válasszon egy olyan szintet és méretet, amely megfelel a teljesítményigényeinek tipikus terhelés mellett, majd horizontálisan skálázza fel a példányokat a megváltozott forgalommennyiség kezeléséhez.
- Engedélyezze az automatikus skálázást. Ha az alkalmazás kiszámítható és rendszeres számítási feladatokat lát el, profilok létrehozásával előre ütemezheti a példányok számát. Ha a számítási feladatok mennyisége nem jelezhető előre, használjon szabályalapú automatikus skálázást, amely azonnal reagál a bekövetkező változásokra. A két módszer kombinálható.
- A CPU-használat általában jól használható metrikaként az automatikus skálázási szabályokhoz. Azonban az alkalmazáson terhelésteszteket kell végezni, azonosítani kell a potenciális szűk keresztmetszeteket, és ezek alapján kell megadni az automatikus skálázási szabályokat.
- Az automatikus skálázási szabályok tartalmaznak egy *nyugalomba kerülési* időt. Ez az az időtartam, amennyit a rendszer vár egy skálázási művelet befejezése után, mielőtt elindítana egy újabb skálázási műveletet. A nyugalomba kerülési idő lehetővé teszi a rendszer stabilizálását az újabb skálázási műveletek előtt. Példányok hozzáadásához állítson be rövidebb nyugalomba kerülési időt, az eltávolításukhoz pedig hosszabbat. Például állítson be 5 perc nyugalomba kerülési időt példányok hozzáadásához, példányok eltávolításához azonban 60 percet. Érdemesebb a nagy terhelés alatt gyorsan új példányokat hozzáadni a megnövekedett forgalom kezelésére, majd fokozatosan csökkenteni az erőforrások számát.

### <a name="scaling-sql-database"></a>Az SQL Database skálázása

Ha magasabb szolgáltatásszintre vagy teljesítményszintre van szüksége az SQL Database-hez, az egyes adatbázisokat egyenként vertikálisan felskálázhatja az alkalmazás leállása nélkül. További információkért lásd: [SQL Database beállításai és teljesítménye: Az egyes szolgáltatásszinteken elérhető ismertetése][sql-db-scale].

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

Az oktatóanyag összeállításakor az App Service SLA-ja 99,95%, az SQL Database alap, standard és prémium szintjeinek SKA-ja pedig 99,99%.

> [!NOTE]
> Az App Service SLA-ja önálló példányokra és több példányra is vonatkozik.
>

### <a name="backups"></a>Biztonsági másolatok

Adatvesztés esetén az SQL Database időponthoz kötött visszaállítást és georedundáns visszaállítást biztosít. Ezek a funkciók minden szinten elérhetőek, és automatikusan engedélyezve vannak. A biztonsági mentéseket nem kell ütemezni vagy kezelni.

- Az időponthoz kötött visszaállítással [az emberi hibák állíthatók helyre][sql-human-error], mivel ez a funkció visszaállítja az adatbázist egy korábbi időpont szerinti állapotára.
- A georedundáns visszaállítás [szolgáltatáskimaradás utáni helyreállítást biztosít][sql-outage-recovery], mivel az adatbázist egy georedundáns biztonsági másolatból állítja vissza.

További információkért tekintse meg [a felhőalapú üzletmenet-folytonossággal és az SQL Database-zel végzett adatbázis-vészhelyreállítással][sql-backup] kapcsolatos cikket.

Az App Service [biztonsági mentési és visszaállítási][web-app-backup] funkciót biztosít az alkalmazásfájlokhoz. Azonban ügyeljen arra, hogy a biztonságimásolat-fájlok egyszerű szövegként tartalmazzák az alkalmazásbeállításokat, amelyek titkos adatokat is tartalmazhatnak, például kapcsolati sztringekat. Kerülje az App Service biztonsági mentési funkciójának használatát az SQL-adatbázisok biztonsági mentéséhez, mert a funkció exportálja az adatbázist egy SQL .bacpac fájlba, és ezzel [DTU-kat][sql-dtu] fogyaszt. Ehelyett használja az SQL Database fentebb leírt, időponthoz kötött visszaállítását.

## <a name="manageability-considerations"></a>Felügyeleti szempontok

Hozzon létre külön erőforráscsoportok éles környezetben, fejlesztési, és tesztelési környezetek. Ez megkönnyíti az üzemelő példányok felügyeletét, a tesztkörnyezetek törlését és a hozzáférési jogok kiosztását.

Amikor erőforrásokat rendel az erőforráscsoportokhoz, vegye figyelembe a következőket:

- Életciklus. Általában érdemes az azonos életciklusú erőforrásokat ugyanabba az erőforráscsoportba helyezni.
- Hozzáférés. A csoportokban található erőforrások hozzáférési szabályzatainak megadásához [szerepköralapú hozzáférés-vezérlést][rbac] (RBAC-t) használhat.
- Számlázás. Megtekintheti az erőforráscsoport összegzett költségeit.

További információk: [Azure Resource Manager overview](/azure/azure-resource-manager/resource-group-overview) (Az Azure Resource Manager áttekintése).

### <a name="deployment"></a>Környezet

Az üzembe helyezés két lépésből áll:

1. Az Azure-erőforrások kiépítése. Javasoljuk, hogy használjon [Azure Resource Manager-sablonok] [ arm-template] ehhez a lépéshez. A sablonok megkönnyítik az üzembe helyezések automatizálását a PowerShell vagy az Azure CLI használatával.
2. Az alkalmazás (a kód, a bináris fájlok és a tartalomfájlok) üzembe helyezése. Több lehetőség közül választhat, például üzembe helyezést végezhet a helyi Git-adattárból, használhatja a Visual Studiót, vagy folyamatos üzembe helyezést végezhet egy felhőalapú forrásvezérlőből. További információkat az [alkalmazások az Azure App Service szolgáltatásban való üzembe helyezését][deploy] ismertető cikkben olvashat.

Az App Service-alkalmazások mindig rendelkeznek egy `production` nevű üzembehelyezési ponttal, amely az éles helyet jelöli. Javasoljuk, hogy a frissítések telepítéséhez hozzon létre egy előkészítési pontot. Az előkészítési pont használatának előnyei a következők:

- Ellenőrizhető az üzembe helyezés sikeressége az éles környezetbe váltás előtt.
- Az előkészítési pontra való üzembe helyezéssel biztosítható, hogy minden példány üzemkészen kerüljön az éles környezetbe. Számos alkalmazás jelentős melegítési és hidegindítási idő van.

Javasoljuk továbbá egy harmadik pont létrehozását a legutóbbi megfelelően működő üzemelő példány tárolásához. Az előkészítési és az üzembehelyezési pontok közötti váltáskor a korábbi éles környezetet (amely most az előkészítési ponton található) helyezze át a legutóbbi megfelelően működő példányt tároló pontra. Így ha később probléma merül fel, gyorsan vissza lehet váltani a legutóbbi megfelelően működő verzióra.

![[1]][1]

Korábbi verzióra való visszaváltáskor győződjön meg arról, hogy az adatbázisok sémamódosításai visszamenőlegesen kompatibilisek.

Ne használja az éles környezet pontjait tesztelésre, mivel az egy adott App Service-csomagban található alkalmazások ugyanazokat a virtuálisgép-példányokat használják. Ha például terheléstesztek csökkentheti az éles helyet. Ehelyett hozzon létre külön App Service-csomagot az éles és a tesztelési környezethez. Egy külön tervbe tesztkörnyezetek helyezésével, elkülönítse azokat a változatát tartalmazza.

### <a name="configuration"></a>Konfiguráció

Tárolja a konfigurációs beállításokat [alkalmazásbeállításokként][app-settings]. Adja meg az alkalmazás beállításait Resource Manager-sablonokkal, vagy a PowerShell-lel. Futtatáskor az alkalmazásbeállítások környezeti változókként elérhetők az alkalmazás számára.

Soha ne tároljon jelszavakat, hozzáférési kulcsokat és kapcsolati sztringekat a forrásvezérlőben. Ehelyett adja át ezeket az adatokat paraméterként egy üzembehelyezési szkriptnek, amely alkalmazásbeállításokként tárolja ezeket az értékeket.

Az üzembehelyezési pontok közötti váltáskor a rendszer alapértelmezés szerint vált az alkalmazásbeállítások között is. Ha különböző beállítások szükségesek az éles és az előkészítési környezethez, létrehozhat olyan alkalmazásbeállításokat, amelyek adott ponthoz vannak rögzítve, és nem változnak.

### <a name="diagnostics-and-monitoring"></a>Diagnosztika és figyelés

Engedélyezze a [diagnosztikai naplózást][diagnostic-logs], beleértve az alkalmazásnaplózást és a webkiszolgáló-naplózást. Konfigurálja a naplózást a Blob Storage használatára. A jobb teljesítmény érdekében hozzon létre egy önálló tárfiókot a diagnosztikai naplók számára. Ne használja ugyanazt a tárfiókot a naplók és alkalmazásadatok. A naplózással kapcsolatos részletesebb útmutatásért tekintse meg a [monitorozási és diagnosztikai útmutatót][monitoring-guidance].

Az alkalmazások terhelés alatti teljesítményének és viselkedésének monitorozásához használjon olyan szolgáltatásokat, mint a [New Relic][new-relic] vagy az [Application Insights][app-insights]. Vegye figyelembe az Application Insights [adatátviteli sebességre vonatkozó korlátozásait][app-insights-data-rate].

Végezzen terheléstesztet, például egy eszközzel [Azure DevOps] [ azure-devops] vagy [Visual Studio Team Foundation Server][tfs]. A felhőalapú alkalmazások teljesítményelemzésének általános áttekintéséért tekintse meg a [teljesítményelemzési ismertetőt][perf-analysis].

Tippek az alkalmazás hibaelhárításához:

- Az Azure Portal [hibaelhárítási paneljén][troubleshoot-blade] megoldást találhat a leggyakoribb problémákra.
- A [naplóstreamelés][web-app-log-stream] engedélyezésével közel valós idejű naplóinformációkat tekinthet meg.
- A [Kudu irányítópultján][kudu] számos eszközt találhat az alkalmazások monitorozásához és hibaelhárításához. További információkért tekintse meg az [Azure-webhelyek online eszközeit ismertető][kudu] blogbejegyzést. A Kudu irányítópultja az Azure Portalról érhető el. Nyissa meg az alkalmazása paneljét, és kattintson az **Eszközök**, majd a **Kudu** elemre.
- Visual Studio használata esetén olvassa el a [webalkalmazások az Azure App Service-ben a Visual Studióval végzett hibaelhárítását][troubleshoot-web-app] ismertető cikket hibakeresési és hibaelhárítási tippekért.

## <a name="security-considerations"></a>Biztonsági szempontok

Ez a szakasz a cikkben leírt Azure-szolgáltatásokra vonatkozó biztonsági szempontokat ismerteti. Nem tartalmazza az összes ajánlott biztonsági eljárást. A további biztonsági szempontokért lásd: [Alkalmazás védelme az Azure App Service-ben][app-service-security].

### <a name="sql-database-auditing"></a>Az SQL Database naplózási funkciója

A naplózás segíthet a jogszabályi megfelelőség fenntartásában és az olyan ellentmondások és rendellenességek feltárásában, amelyek üzleti veszélyekre vagy esetleges biztonsági problémákra utalhatnak. További információk: [Ismerkedés az SQL-adatbázis naplózási szolgáltatásával][sql-audit].

### <a name="deployment-slots"></a>Üzembehelyezési pontok

Minden üzembehelyezési pont nyilvános IP-címmel rendelkezik. A nem éles pontok védelme az [Azure Active Directory-bejelentkezéssel][aad-auth] biztosítható, hogy az adott végpontokhoz csak a fejlesztési és üzemeltetési csapatok tagjai férhessenek hozzá.

### <a name="logging"></a>Naplózás

A naplókban soha nem szabad rögzíteni a felhasználók jelszavait és a személyazonossági csalások elkövetéséhez felhasználható egyéb adatait. Tárolás előtt az adatok közül el kell távolítani az ilyen részleteket.

### <a name="ssl"></a>SSL

Az App Service-alkalmazásokhoz elérhető egy SSL-végpont az `azurewebsites.net` egy résztartományán további költségek nélkül. Az SSL-végpont tartalmaz egy helyettesítő tanúsítványt az `*.azurewebsites.net` tartományhoz. Ha egyéni tartománynevet használ, meg kell adnia egy, az egyéni tartománynak megfelelő tanúsítványt. A legegyszerűbb megoldás az, ha közvetlenül az Azure Portalon vásárol egy tanúsítványt. Más hitelesítésszolgáltatóktól származó tanúsítványokat is importálhat. További információkért lásd: [SSL-tanúsítvány vásárlása és konfigurálása saját Azure App Service szolgáltatások számára][ssl-cert].

Biztonsági szempontból ajánlott, hogy az alkalmazás kikényszerítse a HTTPS használatát a HTTP-kérelmek átirányításával. Ez megvalósítható az alkalmazáson belül, vagy egy URL-átírási szabály használatával, amelynek leírását a [HTTPS az Azure App Service-ben alkalmazásokhoz való engedélyezésével][ssl-redirect] kapcsolatos cikkben olvashatja.

### <a name="authentication"></a>Hitelesítés

Hitelesítéshez javasoljuk egy identitásszolgáltató (IDP), például az Azure AD, a Facebook, a Google vagy a Twitter használatát. A hitelesítési folyamathoz használja az OAuth 2 vagy az OpenID Connect (OIDC) megoldást. Az Azure AD funkciókat biztosít felhasználók és csoportok kezeléséhez, alkalmazások szerepköreinek létrehozásához, a helyszíni identitások integrálásához és az olyan háttérszolgáltatások használatához, mint az Office 365 és a Skype Vállalati verzió.

Lehetőség szerint ne közvetlenül az alkalmazás kezelje a felhasználói bejelentkezéseket és a hitelesítő adatokat, mivel ez potenciális támadási felületet jelenthet.  Mindenképp használjon e-mail-visszaigazolást, jelszó-helyreállítást és többtényezős hitelesítést, ellenőrizze a jelszavak erősségét és tárolja a jelszavak kivonatait biztonságos helyen. A nagyobb identitásszolgáltatók ezek mindegyikét elvégzik Ön helyett, emellett folyamatosan monitorozzák és fejlesztik biztonsági eljárásaikat.

Érdemes lehet [App Service-hitelesítést][app-service-auth] használni az OAuth/OIDC hitelesítési folyamat implementálásához. Az App Service-hitelesítés előnyei a következők:

- Könnyen konfigurálható.
- Az egyszerű hitelesítési forgatókönyvekhez nem szükséges kód.
- Támogatja a delegált hitelesítést: az OAuth hozzáférési tokenek lehetővé teszik az erőforrások használatát egy felhasználó nevében.
- Rendelkezik beépített tokengyorsítótárral.

Az App Service-hitelesítés néhány korlátja:

- Korlátozottak a testreszabási lehetőségek.
- A delegált engedélyezés bejelentkezési munkamenetenként egy háttérerőforrásra korlátozódik.
- Több IDP használata esetén nincs beépített mechanizmus a hitelesítő tartomány észleléséhez.
- Több-bérlős forgatókönyvek esetén az alkalmazásnak kell implementálnia a tokenkiállító ellenőrzését végző logikát.

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

Az architektúra egy példa Resource Manager-sablon [elérhető a Githubon][paas-basic-arm-template].

A sablon PowerShell-lel történő üzembe helyezéséhez futtassa a következő parancsokat:

```powershell
New-AzureRmResourceGroup -Name <resource-group-name> -Location "West US"

$parameters = @{"appName"="<app-name>";"environment"="dev";"locationShort"="uw";"databaseName"="app-db";"administratorLogin"="<admin>";"administratorLoginPassword"="<password>"}

New-AzureRmResourceGroupDeployment -Name <deployment-name> -ResourceGroupName <resource-group-name> -TemplateFile .\PaaS-Basic.json -TemplateParameterObject  $parameters
```

További információkért lásd az [erőforrások Azure Resource Manager-sablonokkal végzett üzembe helyezését][deploy-arm-template] ismertető cikket.

<!-- links -->

[aad-auth]: /azure/app-service-mobile/app-service-mobile-how-to-configure-active-directory-authentication
[app-insights]: /azure/application-insights/app-insights-overview
[app-insights-data-rate]: /azure/application-insights/app-insights-pricing
[app-service]: /azure/app-service/
[app-service-auth]: /azure/app-service-api/app-service-api-authentication
[app-service-plans]: /azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview
[app-service-plans-tiers]: https://azure.microsoft.com/pricing/details/app-service/
[app-service-security]: /azure/app-service-web/web-sites-security
[app-settings]: /azure/app-service-web/web-sites-configure
[arm-template]: /azure/azure-resource-manager/resource-group-overview#resource-groups
[azure-devops]: /azure/devops/
[azure-dns]: /azure/dns/dns-overview
[custom-domain-name]: /azure/app-service-web/web-sites-custom-domain-name
[deploy]: /azure/app-service-web/web-sites-deploy
[deploy-arm-template]: /azure/resource-group-template-deploy
[deployment-slots]: /azure/app-service-web/web-sites-staged-publishing
[diagnostic-logs]: /azure/app-service-web/web-sites-enable-diagnostic-log
[kudu]: https://azure.microsoft.com/blog/windows-azure-websites-online-tools-you-should-know-about/
[monitoring-guidance]: ../../best-practices/monitoring.md
[new-relic]: https://newrelic.com/
[paas-basic-arm-template]: https://github.com/mspnp/reference-architectures/tree/master/managed-web-app/basic-web-app/Paas-Basic/Templates
[perf-analysis]: https://github.com/mspnp/performance-optimization/blob/master/Performance-Analysis-Primer.md
[rbac]: /azure/active-directory/role-based-access-control-what-is
[resource-group]: /azure/azure-resource-manager/resource-group-overview
[sla]: https://azure.microsoft.com/support/legal/sla/
[sql-audit]: /azure/sql-database/sql-database-auditing-get-started
[sql-backup]: /azure/sql-database/sql-database-business-continuity
[sql-db]: /azure/sql-database/
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
[tfs]: /tfs/index
[troubleshoot-web-app]: /azure/app-service-web/web-sites-dotnet-troubleshoot-visual-studio
[visio-download]: https://archcenter.blob.core.windows.net/cdn/app-service-reference-architectures.vsdx
[web-app-autoscale]: /azure/app-service-web/web-sites-scale
[web-app-backup]: /azure/app-service-web/web-sites-backup
[web-app-log-stream]: /azure/app-service-web/web-sites-enable-diagnostic-log#streamlogs
[1]: ./images/paas-basic-web-app-staging-slots.png "Váltás az éles és az előkészítési környezetek pontjai között"
