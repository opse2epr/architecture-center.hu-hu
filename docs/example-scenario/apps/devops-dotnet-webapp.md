---
title: CI/CD-folyamat a vsts-sel
description: Egy példa létrehozása és közzététele az Azure Web Apps .NET-alkalmazás
author: christianreddington
ms.date: 07/11/18
ms.openlocfilehash: 6fb8f00f91ebb5ce6ca873d711d6919cf935ec55
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 09/28/2018
ms.locfileid: "47428550"
---
# <a name="cicd-pipeline-with-vsts"></a>CI/CD-folyamat a vsts-sel

Fejlesztési és üzemeltetési a fejlesztési, minőségbiztosítási és IT-üzemeltetési integrációt. Fejlesztési és üzemeltetési egységes kulturális környezet és a folyamatok rengeteg szükséges szoftvereket.

A példaforgatókönyv azt szemlélteti, hogyan fejlesztői csapatok az Visual Studio Team Services az Azure App Service .NET kétrétegű webes alkalmazás üzembe helyezése. A webalkalmazás Platformszolgáltatási (PaaS) szolgáltatásokra alsóbb rétegbeli Azure Platform függ. Ez a dokumentum is mutat meg néhány szempontot, hogy meg kell győződnie, ilyen esetben az Azure Platform (PaaS) szolgáltatás használatával való tervezésekor.

Alkalmazásfejlesztés folyamatos integrációs (CI) és folyamatos üzembe helyezés (CD) segítségével modern megközelítését bevezetése, megismerheti, hogyan érték egy robusztus építés, tesztelés, üzembe helyezési és figyelési szolgáltatás segítségével a felhasználók számára a kézbesítésének gyorsítását. Platform használatával, mint például a Visual Studio Team Services mellett az Azure App Service szolgáltatásokban, a szervezetek biztosítható, a forgatókönyv fejlesztésének ahelyett, hogy az infrastruktúra engedélyezze azt a felügyeleti összpontosítanak maradnak.

## <a name="related-use-cases"></a>Kapcsolódó alkalmazási helyzetek

Vegye figyelembe a DevOps esetében a következő esetekben használja:

* Felgyorsítható az alkalmazások fejlesztése és fejlesztési életciklusa
* Épület minőségét és a egy automatizált összeállítási és a kibocsátási folyamat konzisztencia

## <a name="architecture"></a>Architektúra

![Az Azure-összetevőket részt vesz egy fejlesztési és üzemeltetési környezetek a Visual Studio Team Services és az Azure App Service-architektúra áttekintése][architecture]

Ebben a forgatókönyvben egy .NET-webalkalmazás a Visual Studio Team Services (VSTS) használata a fejlesztési és üzemeltetési folyamatot ismerteti. A áramlanak keresztül az adatok a forgatókönyv a következő:

1. Alkalmazás forráskódjának módosítása.
2. Az alkalmazás kódja és a Web Apps web.config fájl véglegesítése.
3. A folyamatos integráció indítja az alkalmazás fordítását és az egységteszteket.
4. Folyamatos készregyártás eseményindítója vezényli az alkalmazás-összetevők *a környezetspecifikus paraméterezni a konfigurációs értékeket*.
5. Üzembe helyezés az Azure App Service-ben.
6. Az Azure Application Insights állapot-, teljesítmény- és használati adatokat gyűjt és elemez.
7. Állapot-, teljesítmény- és használati információk áttekintése.

### <a name="components"></a>Összetevők

* [Erőforráscsoportok] [ resource-groups] vannak az Azure-erőforrások logikai tárolója is adja meg a felügyeleti sík hozzáférés-vezérlő határához – gondolja át, hogy "központi telepítési egység" jelző egy erőforráscsoportot.
* [A Visual Studio Team Services (VSTS)] [ vsts] egy szolgáltatás, amely lehetővé teszi a fejlesztési életciklus-végpontok; kezelését a tervezési és projektmenedzsment, a kód felügyelete készítése és kiadása a keresztül.
* [Az Azure Web Apps] [ web-apps] üzemeltetéséhez, webalkalmazások, a REST API-k és a mobilalkalmazások háttérkomponensei Platformszolgáltatási (PaaS) szolgáltatás, a Platform. Amíg ez a cikk .NET összpontosít, többféle módon további szükséges fejlesztési platformon támogatott.
* [Az Application Insights] [ application-insights] van egy belső, bővíthető alkalmazásteljesítmény-felügyeleti (APM) szolgáltatás webfejlesztőknek, több platformon.

### <a name="alternative-devops-tooling-options"></a>Alternatív fejlesztési és üzemeltetési eszközök beállításai

Bár ez a cikk elsősorban a Visual Studio Team Services [Team Foundation Server] [ team-foundation-server] helyi helyére írja be a használható. Másik megoldásként is találhat egy nyílt forráskódú fejlesztési folyamat kihasználva együtt használt technológiák gyűjteménye [Jenkins][jenkins-on-azure].

A kód perspektíva, az infrastruktúra- [Azure Resource Manager-sablonok] [ arm-templates] kerültek, célszerűbb a részeként az Azure DevOps-projekt, de [Terraform] [ terraform] vagy [Chef] [ chef] Ha itt nincs befektetéseit. Ha inkább olyan infrastruktúra-szolgáltatás (IaaS) alapuló telepítési és a szükséges konfiguráció kezelése, akkor érdemes lehet akár [Azure Automation Állapotkonfiguráció][desired-state-configuration], [ Az Ansible] [ ansible] vagy [Chef][chef].

### <a name="alternatives-to-web-app-hosting"></a>Alternatívák webalkalmazás üzemeltetéséhez

Alternatívák üzemeltetése az Azure Web Apps:

* [Virtuális gép] [ compare-vm-hosting] - a számítási feladatok, amelyek a szükséges vezérlőt, magas szintű vagy operációsrendszer-összetevők függenek / szolgáltatások, amelyek nem engedélyezettek a Web Apps (például a Windows GAC, vagy a COM)
* [Tároló üzemeltetési] [ azure-containers] – ahol függőségek az operációs rendszer és hordozhatóság üzemeltető, vagy sűrűségű üzemeltetés, egyúttal követelményeinek.
* [A Service Fabric] [ service-fabric] -jó választás, ha a munkaterhelés-architektúra irányul, hogy körül elosztott összetevőket, amelyek üzembe helyezve, és futtassa a vezérlő magas fokú fürtök között. A Service Fabric is használható tárolók üzemeltetéséhez.
* [Az Azure functions kiszolgáló nélküli] [ azure-functions] -jó választás, ha a munkaterhelés-architektúra részletes eltérése elosztott, minimális függőségeket, ahol az egyes összetevők csak szükséges igénylő összetevői részletesebben igény szerinti (nem folyamatos) futtatása, és vezénylési összetevők, nem szükséges.

### <a name="devops"></a>DevOps

**[Folyamatos integrációs (CI)] [ continuous-integration]**  mutassa be egy stabil build, egynél több egyéni fejlesztői vagy csapat számára a megosztott kódbázis folyamatosan kicsi, gyakran módosítások végrehajtása a következőkhöz.
A folyamatos integrációs folyamat részeként kell;

* Ellenőrizze a kód kis mennyiségű gyakran (elkerülése érdekében, ezek lehetnek nehezebben egyesítése sikerült nagyobb és összetettebb módosításainak kötegelés)
* (Beleértve a véleménypontszáma elérési útjait) megfelelő kódot lefedettséggel rendelkező vetünk az alkalmazás összetevői
* A megosztott, master (vagy trönk) ág vonatkozóan fut le a build biztosítása. Ez az ág stabil és telepítésként"kész" kell lennie. Hiányos vagy folyamatban lévő módosításokat kell elkülöníteni egy külön ágban végzett gyakori "integrációs továbbítsa" összevonása később ütközések elkerülése érdekében.

**[Folyamatos kézbesítési (CD)] [ continuous-delivery]**  tárhelyeken, nem csak egy stabil hozhat létre, de egy stabil üzembe helyezési bemutatásához. Ez megkönnyíti egy kicsit összetettebb CD megvalósításához, a környezetspecifikus konfigurációra szükség, és megfelelően állítja ezeket az értékeket egy mechanizmust.

Emellett elegendő lefedettségét integráció teszteléséhez szükséges győződjön meg arról, hogy vannak-e konfigurálva a különböző összetevők és megfelelően – teljes körű működik-e.

Ez is szükségessé, beállítása és környezetre jellemző adatok alaphelyzetbe állítása és felügyelete az adatbázis-séma verziója.

A folyamatos teljesítés is kiterjesztheti terheléses tesztelés és a felhasználói elfogadás tesztelési környezeteket.

Folyamatos Készregyártás számos előnyt biztosít az folyamatos figyelés ideális esetben minden környezetben keresztül.
A konzisztencia és a központi telepítések és az integráció tesztelésre környezetekben megbízhatóságát könnyebbé vált a parancsfájl-kezelési létrehozását és a konfiguráció vagy a (olyanra, amely lényegesen megkönnyíti a felhőbeli számítási feladatok esetén tekintse meg az Azure üzemeltetési infrastruktúrájának infrastruktúra mint kód) – Ez más néven a ["infrastruktúra-as-kód"][infra-as-code].

* Indítsa el a lehető leghamarabb a folyamatos Készregyártás a projekt életciklusában. Hagyja a később, annál bonyolultabb lesz.
* Integráció és az egységteszteket kell fordítani a projekt lehetőségként azonos prioritású
* Környezet független központi telepítési csomagok használata, és a kibocsátási folyamat környezetspecifikus konfigurációjának kezelése.
* A kiadási kezelőeszközök belül, vagy hívja hardver-security-modulra (HSM), bizalmas konfigurációs védelme vagy [Key Vault][azure-key-vault], a kibocsátási folyamat során. Ne tároljon bizalmas konfigurációs verziókövetés belül.

**Folyamatos tanulás** – a leghatékonyabb figyelést egy CD-környezet által biztosított eszközök alkalmazások teljesítményének figyelése (APM röviden), például a Microsoft [Application Insights] [ application-insights]. Alkalmazás számítási feladat figyelése elegendő mélysége, kritikus fontosságú, hibák, a terhelés megértése. [Az App Insights integrálhatók a VSTS folyamatos figyelés a CD-folyamat engedélyezéséhez][app-insights-cd-monitoring]. Ez használható a következő szintre, emberi beavatkozás vagy visszaállítási nélkül automatikus metódushívásainak engedélyezze, ha a riasztás észlelhető.

## <a name="considerations"></a>Megfontolandó szempontok

### <a name="availability"></a>Rendelkezésre állás

Vegye figyelembe, kihasználva a [jellemző tervezési minták a rendelkezésre állási] [ design-patterns-availability] a felhőbeli alkalmazások készítése során.

Tekintse át a rendelkezésre állási szempontok a megfelelő [App Service webalkalmazás referenciaarchitektúrája][app-service-reference-architecture]

Rendelkezésre állási témaköröket talál a [rendelkezésre állási ellenőrzőlista] [ availability] a az Azure Architecture Centert.

### <a name="scalability"></a>Méretezhetőség

Ha a cloud application készítése vegye figyelembe a [jellemző tervezési minták a méretezhetőség][design-patterns-scalability].

Megfontolandó szempontok a megfelelő méretezhetőség [App Service webalkalmazás referenciaarchitektúrája][app-service-reference-architecture]

Méretezhetőség témaköröket talál a [méretezési ellenőrzőlista] [ scalability] a az Azure Architecture Centert.

### <a name="security"></a>Biztonság

Vegye figyelembe, kihasználva a [jellemző tervezési minták a biztonsági] [ design-patterns-security] ahol lehetséges.

Tekintse át a biztonsági szempontok a megfelelő [App Service webalkalmazás referenciaarchitektúrája][app-service-reference-architecture].

Általános megoldások biztonságos, tekintse át a [Azure Security dokumentációja][security].

### <a name="resiliency"></a>Rugalmasság

Tekintse át a [jellemző tervezési minták a rugalmassághoz] [ design-patterns-resiliency] , és vegye fontolóra a megfelelő helyen.

Számos annak [ajánlott eljárások az App Service] [ resiliency-app-service] a az Azure Architecture Centert.

Rugalmas megoldások tervezésével kapcsolatos általános útmutatásért lásd: [rugalmas alkalmazások tervezése az Azure][resiliency].

## <a name="deploy-the-scenario"></a>A forgatókönyv megvalósításához

### <a name="prerequisites"></a>Előfeltételek

* Meglévő Azure-fiókkal kell rendelkeznie. Ha nem rendelkezik Azure-előfizetéssel, első lépésként létrehozhat egy [ingyenes][azure-free-account] fiókot.
* Jelentkezzen az Azure DevOps szervezet számára. További információkért lásd: [a rövid útmutató: a szervezete létrehozása][vsts-account-create].

### <a name="walk-through"></a>Útmutató

Ebben a forgatókönyvben a CI/CD-folyamatok létrehozására az Azure DevOps-projekt fogja használni.

A DevOps-projekt fogja üzembe helyezése az App Service-csomag, az App Service-ben és a egy App Insights-erőforrást az Ön számára, valamint a Visual Studio Team Services-projekt konfigurálása az Ön számára.

A DevOps-projekt üzembe helyezte és a létrehozás befejezése után tekintse át a társított kódmódosítás szükséges, munkaelemek, és a vizsgálati eredmények. Megfigyelheti, nincs teszt eredményei jelennek meg, a kódot nem tartalmazó bármely tesztek futtatásához.

Tekintse át a kiadás definíciókat. Figyelje meg, hogy a kibocsátási folyamat telepítése, helyőrzőkivétel fejlesztői az alkalmazás közzététele befejeződött Figyelje meg, hogy egy **a folyamatos készregyártás eseményindítója** állítható be a **Drop** összetevő, a Dev környezetekbe történő automatikus kiadásaival hozhat létre. A folyamatos üzembe helyezési folyamat részeként a kiadások látni span több környezetben. Kiadás span mindkét infrastruktúra (például infrastruktúra technikák használatával kód), és az alkalmazáscsomagokat, valamint minden olyan konfigurációt követő feladatok szükséges is telepíthet.

**További szempontok.**

* Fontolja meg az egyik kihasználva a [a jogkivonatok feladatok] [ vsts-tokenization] , amelyek is elérhetők a VSTS-piactéren.
* Fontolja meg a [üzembe helyezés: Azure Key Vault] [ download-keyvault-secrets] VSTS-feladat, a kiadás az Azure KeyVault titkos kódok töltheti le. Ezután használhatja ezeket a titkos kulcsokat a kiadási definíció részeként változókként, és kell nem tárolja őket verziókövetési rendszerben.
* Fontolja meg [változók kiadási] [ vsts-release-variables] meghajtó konfigurációjának a módosításához a környezetek a kiadási-definíciókban. Kiadási változók hatóköre beállítható egy teljes kiadása, vagy egy adott környezetben. Változók használata a titkos adatok, győződjön meg arról, hogy bejelöli-e a lakat ikonra.
* Fontolja meg [üzembe helyezési kapuk] [ vsts-deployment-gates] a kibocsátási folyamat. Ez lehetővé teszi a figyelési adatok (például az incidenskezelés vagy egyéb bespoke rendszerekhez) külső rendszerekkel való társítás e kiadás támogatni kell meghatározni.
* Amennyiben a kibocsátási folyamat manuális beavatkozásra szükség, fontolja meg a [jóváhagyások] [ vsts-approvals] funkciót.
* Fontolja meg [Application Insights] [ application-insights] és a kiadás a lehető leghamarabb folyamat további figyelési eszközök. A legtöbb szervezet csak megfigyelésének saját éles környezetben azonban a folyamat korábbi szakaszában a lehetséges hibák azonosítása és a felhasználók számára, éles környezetben gyakorolt hatás megakadályozása sikerült.

## <a name="pricing"></a>Díjszabás

A Visual Studio Team Services ára a szervezet tényezők, például az egyidejű build/verziókban szükséges, száma mellett a hozzáférést igénylő felhasználók számát és tesztfelhasználók számát függ. Ezek részletes leírást talál a további a [díjszabását ismertető lapon VSTS][vsts-pricing-page].

* [A Visual Studio Team Services (VSTS)] [ vsts-pricing-calculator] egy szolgáltatás, amely lehetővé teszi, hogy a fejlesztési életciklus kezelése és a fizetett van egy felhasználónkénti / hónap alapon történik. További díjakat lehet szükség, kívül semmilyen további tesztfelhasználók vagy alapszintű felhasználói licencek egyidejű folyamatok függ.

## <a name="related-resources"></a>Kapcsolódó erőforrások

* [Mit jelent a DevOps?][devops-whatis]
* [A Microsoft - fejlesztés és üzemeltetés a Visual Studio Team Services munka][devops-microsoft]
* [Lépésről lépésre haladó oktatóanyagok: Fejlesztés és üzemeltetés a Visual Studio Team Services használatával][devops-with-vsts]
* [CI/CD-folyamat létrehozása a .NET-hez az Azure DevOps-projekt][devops-project-create]

<!-- links -->
[ansible]: /azure/ansible/
[application-insights]: /azure/application-insights/app-insights-overview
[app-service-reference-architecture]: ../../reference-architectures/app-service-web-app/basic-web-app.md
[azure-free-account]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
[arm-templates]: /azure/azure-resource-manager/resource-group-overview#template-deployment
[architecture]: ./media/devops-dotnet-webapp/architecture-devops-dotnet-webapp.png
[availability]: /azure/architecture/checklist/availability
[chef]: /azure/chef/
[design-patterns-availability]: /azure/architecture/patterns/category/availability
[design-patterns-resiliency]: /azure/architecture/patterns/category/resiliency
[design-patterns-scalability]: /azure/architecture/patterns/category/performance-scalability
[design-patterns-security]: /azure/architecture/patterns/category/security
[desired-state-configuration]: /azure/automation/automation-dsc-overview
[devops-microsoft]: /azure/devops/devops-at-microsoft/
[devops-with-vsts]: https://almvm.azurewebsites.net/labs/vsts/
[application-insights]: https://azure.microsoft.com/en-gb/services/application-insights/
[cloud-based-load-testing]: https://visualstudio.microsoft.com/team-services/cloud-load-testing/
[cloud-based-load-testing-on-premises]: /vsts/test/load-test/clt-with-private-machines?view=vsts
[jenkins-on-azure]: /azure/jenkins/
[devops-whatis]: /azure/devops/what-is-devops
[download-keyvault-secrets]: /vsts/pipelines/tasks/deploy/azure-key-vault?view=vsts
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[resiliency-app-service]: /azure/architecture/checklist/resiliency-per-service#app-service
[resiliency]: /azure/architecture/checklist/resiliency
[scalability]: /azure/architecture/checklist/scalability
[vsts]: /vsts/?view=vsts#pivot=services
[continuous-integration]: /azure/devops/what-is-continuous-integration
[continuous-delivery]: /azure/devops/what-is-continuous-delivery
[web-apps]: /azure/app-service/app-service-web-overview
[terraform]: /azure/terraform/
[vsts-account-create]: /azure/devops/organizations/accounts/create-organization-msa-or-work-student?view=vsts
[vsts-approvals]: /vsts/pipelines/release/approvals/approvals?view=vsts
[devops-project]: https://portal.azure.com/?feature.customportal=false#create/Microsoft.AzureProject
[vsts-deployment-gates]: /vsts/pipelines/release/approvals/gates?view=vsts
[vsts-pricing-calculator]: https://azure.com/e/498aa024454445a8a352e75724f900b1
[vsts-pricing-page]: https://azure.microsoft.com/en-us/pricing/details/visual-studio-team-services/
[vsts-release-variables]: /vsts/pipelines/release/variables?view=vsts&tabs=batch
[vsts-tokenization]: https://marketplace.visualstudio.com/search?term=token&target=VSTS&category=All%20categories&sortBy=Relevance
[azure-key-vault]: /azure/key-vault/key-vault-overview
[infra-as-code]: https://blogs.msdn.microsoft.com/mvpawardprogram/2018/02/13/infrastructure-as-code/
[team-foundation-server]: https://visualstudio.microsoft.com/tfs/
[infra-as-code]: https://blogs.msdn.microsoft.com/mvpawardprogram/2018/02/13/infrastructure-as-code/
[service-fabric]:/azure/service-fabric/
[azure-functions]:/azure/azure-functions/
[azure-containers]:https://azure.microsoft.com/en-us/overview/containers/
[compare-vm-hosting]:/azure/app-service/choose-web-site-cloud-service-vm
[app-insights-cd-monitoring]:/azure/application-insights/app-insights-vsts-continuous-monitoring
[azure-region-pair-bcdr]:/azure/best-practices-availability-paired-regions
[devops-project-create]: /azure/devops-project/azure-devops-project-aspnet-core
[security]: /azure/security/