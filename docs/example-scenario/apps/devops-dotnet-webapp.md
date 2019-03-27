---
title: CI-/CD-folyamat megtervezése az Azure DevOps segítségével
titleSuffix: Azure Example Scenarios
description: .NET-alkalmazást hozhat létre, és közzéteheti az Azure Web Appsben az Azure DevOps használatával.
author: christianreddington
ms.date: 12/06/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.custom: fasttrack, seodec18
social_image_url: /azure/architecture/example-scenario/apps/media/architecture-devops-dotnet-webapp.svg
ms.openlocfilehash: 22a714a49c3e722fc11f2498e9a3abd1bec46d22
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299216"
---
# <a name="design-a-cicd-pipeline-using-azure-devops"></a>CI-/CD-folyamat megtervezése az Azure DevOps segítségével

Ebben a forgatókönyvben a folyamatos integrációs (CI) és a folyamatos készregyártás (CD) architektúra és kialakítás útmutatást nyújt. Ebben a példában a CI/CD-folyamat egy kétrétegű .NET webalkalmazás az Azure App Service üzembe helyezi.

A modern CI/CD-folyamatok-ba való migrálás üzenetcsere számos előnnyel jár, az alkalmazás létrejött, a központi telepítések, tesztelése és figyelése. Az Azure DevOps felügyelniük együtt más szolgáltatásokkal, például az App Service-ben, szervezetek koncentrálhat kezelését támogató infrastruktúra helyett az alkalmazások fejlesztését.

## <a name="relevant-use-cases"></a>Alkalmazási helyzetek

Fontolja meg az Azure DevOps és a CI/CD-folyamatok:

- Gyorsabb alkalmazásfejlesztés és fejlesztési életciklusa
- Épület minőségét és a egy automatizált összeállítási és a kibocsátási folyamat konzisztencia
- Alkalmazás stabilitását és rendelkezésre állását növelése

## <a name="architecture"></a>Architektúra

![Az Azure-összetevőket egy fejlesztési és üzemeltetési a forgatókönyvet, az Azure DevOps és az Azure App Service-ben részt vevő architektúra ábrája][architecture]

A áramlanak keresztül az adatok a forgatókönyv a következő:

1. A fejlesztő módosítja az alkalmazás forráskódjának.
2. Alkalmazáskód, beleértve a web.config fájl számára fontos, hogy a forráskód adattára Azure tárházakban.
3. Folyamatos integráció aktiválása az alkalmazás fordítását és az egységteszteket Azure vizsgálati csomagok használatával.
4. Folyamatos üzembe helyezés Azure folyamatok belül aktivál egy automatikus központi telepítési az alkalmazás-összetevők *környezetspecifikus konfigurációs értékeket*.
5. Az összetevők az Azure App Service-ben vannak telepítve.
6. Az Azure Application Insights állapot-, teljesítmény- és használati adatokat gyűjt és elemez.
7. A fejlesztők figyelő és mange állapotával, teljesítményével és használati adatokat.
8. Várakozó adatok rangsorolására új szolgáltatások és hibajavítások az Azure-táblák használatával.

### <a name="components"></a>Összetevők

- [Az Azure DevOps] [ vsts] egy szolgáltatás a fejlesztési életciklus-végpontok kezelésére szolgáló &mdash; felügyeleti kódot, és továbbra is készítése és kiadása a tervezési és a projekt felügyelet alól.

- [Az Azure Web Apps] [ web-apps] mobil háttérrendszer és a egy PaaS szolgáltatás REST API-k, a webalkalmazások üzemeltetéséhez. Amíg ez a cikk .NET összpontosít, többféle módon további szükséges fejlesztési platformon támogatott.

- [Az Application Insights] [ application-insights] van egy belső, bővíthető alkalmazásteljesítmény-felügyeleti (APM) szolgáltatás webfejlesztőknek, több platformon.

### <a name="alternatives"></a>Alternatív megoldások

Amíg ez a cikk foglalkozik az Azure DevOps, [Azure DevOps-kiszolgáló] [ azure-devops-server] (korábbi nevén a Team Foundation Server), egy helyszíni helyettesítő használható. Másik megoldásként is használhat technológiák csoportja egy nyílt forráskódú fejlesztési folyamat a [Jenkins][jenkins-on-azure].

Az infrastruktúra mint kód szempontjából [Resource Manager-sablonok] [ arm-templates] használták, mint az Azure DevOps-projekt, de részét sikerült ajánlott egyéb felügyeleti technológiáinak vonatkozásában, mint például [ A Terraform] [ terraform] vagy [Chef][chef]. Ha inkább egy infrastruktúra--szolgáltatásként (IaaS)-alapú üzembe helyezés és a szükséges konfiguráció kezelése, érdemes lehet akár [Azure Automation Állapotkonfiguráció][desired-state-configuration], [ Az Ansible][ansible], vagy [Chef][chef].

Érdemes lehet üzemeltetése az Azure Web Apps az alábbi lehetőségeket:

- [Az Azure Virtual Machines] [ compare-vm-hosting] kezeli a számítási feladatok vagy vezérlő egy magas szintű igénylő, operációsrendszer-összetevők és szolgáltatások, amelyek nem lehetséges a Web Apps (például a Windows GAC, vagy a COM) függ.

- [A Service Fabric] [ service-fabric] van jó választás, ha a munkaterhelés-architektúra irányul, hogy körül elosztott összetevőket, amelyek üzembe helyezve, és futtassa a vezérlő magas fokú fürtök között. A Service Fabric is használható tárolók üzemeltetéséhez.

- [Az Azure Functions] [ azure-functions] biztosít egy érvényes, ha a munkaterhelés-architektúra részletes eltérése a kiszolgáló nélküli megközelítés elosztott, minimális függőségeket, hol tárolja a rendszer egyes összetevőinek igénylő összetevői részletesebben csak az igény szerinti (nem folyamatosan) futtatásához szükséges, és a vezénylési összetevők, nem szükséges.

Ez [döntési fát az Azure számítási szolgáltatások](/azure/architecture/guide/technology-choices/compute-decision-tree) segíthet a helyes elérési útjával kiválasztásakor figyelembe az áttelepítéshez.

## <a name="management-and-security-considerations"></a>Felügyeleti és biztonsági megfontolások

- Fontolja meg az egyik kihasználva a [a jogkivonatok feladatok] [ vsts-tokenization] a VSTS-piactéren elérhető.

- [Az Azure Key Vault] [ download-keyvault-secrets] tevékenységek képesek letölteni titkos kulcsok az Azure Key vault, a kiadás. Ezután használhatja ezeket a titkos kulcsok változókként a kiadási definíciót, tárolja őket a verziókövetési rendszerben elkerülhető a.

- Használja [változók kiadási] [ vsts-release-variables] meghajtó konfigurációjának a módosításához a környezetek a kiadási-definíciókban. Kiadási változók hatóköre beállítható egy teljes kiadása, vagy egy adott környezetben. Változók használata a titkos adatok, győződjön meg arról, hogy bejelöli-e a lakat ikonra.

- [Üzembe helyezés kapuk] [ vsts-deployment-gates] a kibocsátási folyamat célszerű használni. Ez lehetővé teszi a figyelési adatok (például az incidenskezelés vagy egyéb bespoke rendszerekhez) külső rendszerekkel való társítás e kiadás támogatni kell meghatározni.

- A kibocsátási folyamat manuális beavatkozásra szükség, ahol a [jóváhagyások] [ vsts-approvals] funkciót.

- Fontolja meg [Application Insights] [ application-insights] és a kibocsátási folyamat a lehető leghamarabb további figyelési eszközöket. Sok szervezet csak saját éles környezetben figyelésének megkezdése előtt. A más környezetek figyelésével, a fejlesztési folyamatot a korábbi hibák azonosítása és a problémák elkerülése érdekében az éles környezetben.

## <a name="deploy-the-scenario"></a>A forgatókönyv megvalósításához

### <a name="prerequisites"></a>Előfeltételek

- Meglévő Azure-fiókkal kell rendelkeznie. Ha nem rendelkezik Azure-előfizetéssel, mindössze néhány perc alatt létrehozhat egy [ingyenes fiókot](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) a virtuális gép létrehozásának megkezdése előtt.

- Jelentkezzen az Azure DevOps szervezet számára. További információkért lásd: [a rövid útmutató: Hozzon létre a szervezet][vsts-account-create].

### <a name="walk-through"></a>Az útmutatóban

A [Azure DevOps-projekt](/azure/devops-project/azure-devops-project-github) fog üzembe helyezése az App Service-csomag, az App Service-ben és a egy App Insights-erőforrást az Ön számára, valamint az Azure DevOps-projekt konfigurálása az Ön számára.

Az Azure DevOps-projekt üzembe helyezte, és a létrehozás befejezése után tekintse át a társított kódmódosítás szükséges, munkaelemek, és a vizsgálati eredmények. Megfigyelheti, hogy nincs teszt eredményei jelennek meg, mert a kód nem tartalmaz semmilyen tesztek futtatásához.

A projekt létrehoz egy kibocsátási folyamatok és a folyamatos készregyártás eseményindítója, a fejlesztési környezetbe az alkalmazás üzembe helyezése. A folyamatos üzembe helyezési folyamat részeként jelenhet meg, amelyek több környezetet kiadások. Egy kiadási is kiterjedhet mindkét infrastruktúra (például az infrastruktúra mint kód technikák használatával), és az alkalmazáscsomagok szükséges utáni konfigurációs feladatok együtt is üzembe helyezheti.

## <a name="pricing"></a>Díjszabás

Azure-fejlesztési és üzemeltetési költségek a felhasználók a szervezet más tényezők, például a számát egyidejű build/verziókban szükséges és tesztelési felhasználók száma, valamint a hozzáférést igénylő függenek. További információkért lásd: [Azure DevOps-díjszabás][vsts-pricing-page].

Ez [díjkalkulátor] [ vsts-pricing-calculator] becsült biztosít a futó Azure DevOps-20 felhasználó.

Az Azure DevOps felhasználónkénti havi rendszerességgel történik. További díjakat lehet szükség, kívül semmilyen további tesztfelhasználók vagy alapszintű felhasználói licencek egyidejű folyamatok függ.

## <a name="related-resources"></a>Kapcsolódó források (lehet, hogy a cikkek angol nyelvűek)

Tekintse át a következő források további információ a CI/CD-ről és az Azure DevOps:

- [Mit jelent a DevOps?][devops-whatis]
- [A Microsoft - fejlesztési és üzemeltetési munka az Azure DevOps][devops-microsoft]
- [Lépésről lépésre haladó oktatóanyagok: A DevOps és az Azure DevOps][devops-with-vsts]
- [Fejlesztési és üzemeltetési ellenőrzőlista][devops-checklist]
- [CI/CD-folyamat létrehozása a .NET-hez az Azure DevOps-projekt][devops-project-create]

<!-- links -->

[ansible]: /azure/ansible/
[application-insights]: /azure/application-insights/app-insights-overview
[app-service-reference-architecture]: ../../reference-architectures/app-service-web-app/basic-web-app.md
[arm-templates]: /azure/azure-resource-manager/resource-group-overview#template-deployment
[architecture]: ./media/architecture-devops-dotnet-webapp.svg
[chef]: /azure/chef/
[design-patterns-availability]: /azure/architecture/patterns/category/availability
[design-patterns-resiliency]: /azure/architecture/patterns/category/resiliency
[design-patterns-scalability]: /azure/architecture/patterns/category/performance-scalability
[design-patterns-security]: /azure/architecture/patterns/category/security
[desired-state-configuration]: /azure/automation/automation-dsc-overview
[devops-microsoft]: /azure/devops/devops-at-microsoft/
[devops-with-vsts]: https://almvm.azurewebsites.net/labs/vsts/
[devops-checklist]: /azure/architecture/checklist/dev-ops
[application-insights]: https://azure.microsoft.com/services/application-insights/
[cloud-based-load-testing]: https://visualstudio.microsoft.com/team-services/cloud-load-testing/
[cloud-based-load-testing-on-premises]: /vsts/test/load-test/clt-with-private-machines?view=vsts
[jenkins-on-azure]: /azure/jenkins/
[devops-whatis]: /azure/devops/what-is-devops
[download-keyvault-secrets]: /vsts/pipelines/tasks/deploy/azure-key-vault?view=vsts
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[resiliency-app-service]: /azure/architecture/checklist/resiliency-per-service#app-service
[vsts]: /vsts/?view=vsts#pivot=services
[continuous-integration]: /azure/devops/what-is-continuous-integration
[continuous-delivery]: /azure/devops/what-is-continuous-delivery
[web-apps]: /azure/app-service/app-service-web-overview
[vsts-account-create]: /azure/devops/organizations/accounts/create-organization-msa-or-work-student?view=vsts
[vsts-approvals]: /vsts/pipelines/release/approvals/approvals?view=vsts
[devops-project]: https://portal.azure.com/?feature.customportal=false#create/Microsoft.AzureProject
[vsts-deployment-gates]: /vsts/pipelines/release/approvals/gates?view=vsts
[vsts-pricing-calculator]: https://azure.com/e/498aa024454445a8a352e75724f900b1
[vsts-pricing-page]: https://azure.microsoft.com/pricing/details/visual-studio-team-services/
[vsts-release-variables]: /vsts/pipelines/release/variables?view=vsts&tabs=batch
[vsts-tokenization]: https://marketplace.visualstudio.com/search?term=token&target=VSTS&category=All%20categories&sortBy=Relevance
[azure-key-vault]: /azure/key-vault/key-vault-overview
[infra-as-code]: https://blogs.msdn.microsoft.com/mvpawardprogram/2018/02/13/infrastructure-as-code/
[azure-devops-server]: https://visualstudio.microsoft.com/tfs/
[infra-as-code]: https://blogs.msdn.microsoft.com/mvpawardprogram/2018/02/13/infrastructure-as-code/
[service-fabric]: /azure/service-fabric/
[azure-functions]: /azure/azure-functions/
[azure-containers]: https://azure.microsoft.com/overview/containers/
[compare-vm-hosting]: /azure/app-service/choose-web-site-cloud-service-vm
[app-insights-cd-monitoring]: /azure/application-insights/app-insights-vsts-continuous-monitoring
[azure-region-pair-bcdr]: /azure/best-practices-availability-paired-regions
[devops-project-create]: /azure/devops-project/azure-devops-project-aspnet-core
[terraform]: /azure/terraform/
