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
# <a name="design-a-cicd-pipeline-using-azure-devops"></a><span data-ttu-id="fb8f1-103">CI-/CD-folyamat megtervezése az Azure DevOps segítségével</span><span class="sxs-lookup"><span data-stu-id="fb8f1-103">Design a CI/CD pipeline using Azure DevOps</span></span>

<span data-ttu-id="fb8f1-104">Ebben a forgatókönyvben a folyamatos integrációs (CI) és a folyamatos készregyártás (CD) architektúra és kialakítás útmutatást nyújt.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-104">This scenario provides architecture and design guidance for building a continuous integration (CI) and continuous deployment (CD) pipeline.</span></span> <span data-ttu-id="fb8f1-105">Ebben a példában a CI/CD-folyamat egy kétrétegű .NET webalkalmazás az Azure App Service üzembe helyezi.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-105">In this example, the CI/CD pipeline deploys a two-tier .NET web application to the Azure App Service.</span></span>

<span data-ttu-id="fb8f1-106">A modern CI/CD-folyamatok-ba való migrálás üzenetcsere számos előnnyel jár, az alkalmazás létrejött, a központi telepítések, tesztelése és figyelése.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-106">Migrating to modern CI/CD processes provides many benefits for application builds, deployments, testing, and monitoring.</span></span> <span data-ttu-id="fb8f1-107">Az Azure DevOps felügyelniük együtt más szolgáltatásokkal, például az App Service-ben, szervezetek koncentrálhat kezelését támogató infrastruktúra helyett az alkalmazások fejlesztését.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-107">By utilizing Azure DevOps along with other services such as App Service, organizations can focus on the development of their apps rather than the management of the supporting infrastructure.</span></span>

## <a name="relevant-use-cases"></a><span data-ttu-id="fb8f1-108">Alkalmazási helyzetek</span><span class="sxs-lookup"><span data-stu-id="fb8f1-108">Relevant use cases</span></span>

<span data-ttu-id="fb8f1-109">Fontolja meg az Azure DevOps és a CI/CD-folyamatok:</span><span class="sxs-lookup"><span data-stu-id="fb8f1-109">Consider Azure DevOps and CI/CD processes for:</span></span>

- <span data-ttu-id="fb8f1-110">Gyorsabb alkalmazásfejlesztés és fejlesztési életciklusa</span><span class="sxs-lookup"><span data-stu-id="fb8f1-110">Accelerating application development and development life cycles</span></span>
- <span data-ttu-id="fb8f1-111">Épület minőségét és a egy automatizált összeállítási és a kibocsátási folyamat konzisztencia</span><span class="sxs-lookup"><span data-stu-id="fb8f1-111">Building quality and consistency into an automated build and release process</span></span>
- <span data-ttu-id="fb8f1-112">Alkalmazás stabilitását és rendelkezésre állását növelése</span><span class="sxs-lookup"><span data-stu-id="fb8f1-112">Increasing application stability and uptime</span></span>

## <a name="architecture"></a><span data-ttu-id="fb8f1-113">Architektúra</span><span class="sxs-lookup"><span data-stu-id="fb8f1-113">Architecture</span></span>

![Az Azure-összetevőket egy fejlesztési és üzemeltetési a forgatókönyvet, az Azure DevOps és az Azure App Service-ben részt vevő architektúra ábrája][architecture]

<span data-ttu-id="fb8f1-115">A áramlanak keresztül az adatok a forgatókönyv a következő:</span><span class="sxs-lookup"><span data-stu-id="fb8f1-115">The data flows through the scenario as follows:</span></span>

1. <span data-ttu-id="fb8f1-116">A fejlesztő módosítja az alkalmazás forráskódjának.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-116">A developer changes application source code.</span></span>
2. <span data-ttu-id="fb8f1-117">Alkalmazáskód, beleértve a web.config fájl számára fontos, hogy a forráskód adattára Azure tárházakban.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-117">Application code including the web.config file is committed to the source code repository in Azure Repos.</span></span>
3. <span data-ttu-id="fb8f1-118">Folyamatos integráció aktiválása az alkalmazás fordítását és az egységteszteket Azure vizsgálati csomagok használatával.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-118">Continuous integration triggers application build and unit tests using Azure Test Plans.</span></span>
4. <span data-ttu-id="fb8f1-119">Folyamatos üzembe helyezés Azure folyamatok belül aktivál egy automatikus központi telepítési az alkalmazás-összetevők *környezetspecifikus konfigurációs értékeket*.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-119">Continuous deployment within Azure Pipelines triggers an automated deployment of application artifacts *with environment-specific configuration values*.</span></span>
5. <span data-ttu-id="fb8f1-120">Az összetevők az Azure App Service-ben vannak telepítve.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-120">The artifacts are deployed to Azure App Service.</span></span>
6. <span data-ttu-id="fb8f1-121">Az Azure Application Insights állapot-, teljesítmény- és használati adatokat gyűjt és elemez.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-121">Azure Application Insights collects and analyzes health, performance, and usage data.</span></span>
7. <span data-ttu-id="fb8f1-122">A fejlesztők figyelő és mange állapotával, teljesítményével és használati adatokat.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-122">Developers monitor and mange health, performance, and usage information.</span></span>
8. <span data-ttu-id="fb8f1-123">Várakozó adatok rangsorolására új szolgáltatások és hibajavítások az Azure-táblák használatával.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-123">Backlog information is used to prioritize new features and bug fixes using Azure Boards.</span></span>

### <a name="components"></a><span data-ttu-id="fb8f1-124">Összetevők</span><span class="sxs-lookup"><span data-stu-id="fb8f1-124">Components</span></span>

- <span data-ttu-id="fb8f1-125">[Az Azure DevOps] [ vsts] egy szolgáltatás a fejlesztési életciklus-végpontok kezelésére szolgáló &mdash; felügyeleti kódot, és továbbra is készítése és kiadása a tervezési és a projekt felügyelet alól.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-125">[Azure DevOps][vsts] is a service for managing your development life cycle end-to-end &mdash; from planning and project management, to code management, and continuing to build and release.</span></span>

- <span data-ttu-id="fb8f1-126">[Az Azure Web Apps] [ web-apps] mobil háttérrendszer és a egy PaaS szolgáltatás REST API-k, a webalkalmazások üzemeltetéséhez.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-126">[Azure Web Apps][web-apps] is a PaaS service for hosting web applications, REST APIs, and mobile back ends.</span></span> <span data-ttu-id="fb8f1-127">Amíg ez a cikk .NET összpontosít, többféle módon további szükséges fejlesztési platformon támogatott.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-127">While this article focuses on .NET, there are several additional development platform options supported.</span></span>

- <span data-ttu-id="fb8f1-128">[Az Application Insights] [ application-insights] van egy belső, bővíthető alkalmazásteljesítmény-felügyeleti (APM) szolgáltatás webfejlesztőknek, több platformon.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-128">[Application Insights][application-insights] is a first-party, extensible Application Performance Management (APM) service for web developers on multiple platforms.</span></span>

### <a name="alternatives"></a><span data-ttu-id="fb8f1-129">Alternatív megoldások</span><span class="sxs-lookup"><span data-stu-id="fb8f1-129">Alternatives</span></span>

<span data-ttu-id="fb8f1-130">Amíg ez a cikk foglalkozik az Azure DevOps, [Azure DevOps-kiszolgáló] [ azure-devops-server] (korábbi nevén a Team Foundation Server), egy helyszíni helyettesítő használható.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-130">While this article focuses on Azure DevOps, [Azure DevOps Server][azure-devops-server] (previously known as Team Foundation Server) could be used as an on-premises substitute.</span></span> <span data-ttu-id="fb8f1-131">Másik megoldásként is használhat technológiák csoportja egy nyílt forráskódú fejlesztési folyamat a [Jenkins][jenkins-on-azure].</span><span class="sxs-lookup"><span data-stu-id="fb8f1-131">Alternatively, you could also use a set of technologies for an open-source development pipeline using [Jenkins][jenkins-on-azure].</span></span>

<span data-ttu-id="fb8f1-132">Az infrastruktúra mint kód szempontjából [Resource Manager-sablonok] [ arm-templates] használták, mint az Azure DevOps-projekt, de részét sikerült ajánlott egyéb felügyeleti technológiáinak vonatkozásában, mint például [ A Terraform] [ terraform] vagy [Chef][chef].</span><span class="sxs-lookup"><span data-stu-id="fb8f1-132">From an infrastructure-as-code perspective, [Resource Manager templates][arm-templates] were used as part of the Azure DevOps project, but you could consider other management technologies such as [Terraform][terraform] or [Chef][chef].</span></span> <span data-ttu-id="fb8f1-133">Ha inkább egy infrastruktúra--szolgáltatásként (IaaS)-alapú üzembe helyezés és a szükséges konfiguráció kezelése, érdemes lehet akár [Azure Automation Állapotkonfiguráció][desired-state-configuration], [ Az Ansible][ansible], vagy [Chef][chef].</span><span class="sxs-lookup"><span data-stu-id="fb8f1-133">If you prefer an infrastructure-as-a-service (IaaS)-based deployment and require configuration management, you could consider either [Azure Automation State Configuration][desired-state-configuration], [Ansible][ansible], or [Chef][chef].</span></span>

<span data-ttu-id="fb8f1-134">Érdemes lehet üzemeltetése az Azure Web Apps az alábbi lehetőségeket:</span><span class="sxs-lookup"><span data-stu-id="fb8f1-134">You could consider these alternatives to hosting in Azure Web Apps:</span></span>

- <span data-ttu-id="fb8f1-135">[Az Azure Virtual Machines] [ compare-vm-hosting] kezeli a számítási feladatok vagy vezérlő egy magas szintű igénylő, operációsrendszer-összetevők és szolgáltatások, amelyek nem lehetséges a Web Apps (például a Windows GAC, vagy a COM) függ.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-135">[Azure Virtual Machines][compare-vm-hosting] handles workloads that require a high degree of control, or depend on OS components and services that are not possible with Web Apps (for example, the Windows GAC, or COM).</span></span>

- <span data-ttu-id="fb8f1-136">[A Service Fabric] [ service-fabric] van jó választás, ha a munkaterhelés-architektúra irányul, hogy körül elosztott összetevőket, amelyek üzembe helyezve, és futtassa a vezérlő magas fokú fürtök között.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-136">[Service Fabric][service-fabric] is a good option if the workload architecture is focused around distributed components that benefit from being deployed and run across a cluster with a high degree of control.</span></span> <span data-ttu-id="fb8f1-137">A Service Fabric is használható tárolók üzemeltetéséhez.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-137">Service Fabric can also be used to host containers.</span></span>

- <span data-ttu-id="fb8f1-138">[Az Azure Functions] [ azure-functions] biztosít egy érvényes, ha a munkaterhelés-architektúra részletes eltérése a kiszolgáló nélküli megközelítés elosztott, minimális függőségeket, hol tárolja a rendszer egyes összetevőinek igénylő összetevői részletesebben csak az igény szerinti (nem folyamatosan) futtatásához szükséges, és a vezénylési összetevők, nem szükséges.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-138">[Azure Functions][azure-functions] provides an effective serverless approach if the workload architecture is centered around fine grained distributed components, requiring minimal dependencies, where individual components are only required to run on demand (not continuously) and orchestration of components is not required.</span></span>

<span data-ttu-id="fb8f1-139">Ez [döntési fát az Azure számítási szolgáltatások](/azure/architecture/guide/technology-choices/compute-decision-tree) segíthet a helyes elérési útjával kiválasztásakor figyelembe az áttelepítéshez.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-139">This [decision tree for Azure compute services](/azure/architecture/guide/technology-choices/compute-decision-tree) may help when choosing the right path to take for a migration.</span></span>

## <a name="management-and-security-considerations"></a><span data-ttu-id="fb8f1-140">Felügyeleti és biztonsági megfontolások</span><span class="sxs-lookup"><span data-stu-id="fb8f1-140">Management and Security Considerations</span></span>

- <span data-ttu-id="fb8f1-141">Fontolja meg az egyik kihasználva a [a jogkivonatok feladatok] [ vsts-tokenization] a VSTS-piactéren elérhető.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-141">Consider leveraging one of the [tokenization tasks][vsts-tokenization] available in the VSTS marketplace.</span></span>

- <span data-ttu-id="fb8f1-142">[Az Azure Key Vault] [ download-keyvault-secrets] tevékenységek képesek letölteni titkos kulcsok az Azure Key vault, a kiadás.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-142">[Azure Key Vault][download-keyvault-secrets] tasks can download secrets from an Azure Key Vault into your release.</span></span> <span data-ttu-id="fb8f1-143">Ezután használhatja ezeket a titkos kulcsok változókként a kiadási definíciót, tárolja őket a verziókövetési rendszerben elkerülhető a.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-143">You can then use those secrets as variables in your release definition, which avoids storing them in source control.</span></span>

- <span data-ttu-id="fb8f1-144">Használja [változók kiadási] [ vsts-release-variables] meghajtó konfigurációjának a módosításához a környezetek a kiadási-definíciókban.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-144">Use [release variables][vsts-release-variables] in your release definitions to drive configuration changes of your environments.</span></span> <span data-ttu-id="fb8f1-145">Kiadási változók hatóköre beállítható egy teljes kiadása, vagy egy adott környezetben.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-145">Release variables can be scoped to an entire release or a given environment.</span></span> <span data-ttu-id="fb8f1-146">Változók használata a titkos adatok, győződjön meg arról, hogy bejelöli-e a lakat ikonra.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-146">When using variables for secret information, ensure that you select the padlock icon.</span></span>

- <span data-ttu-id="fb8f1-147">[Üzembe helyezés kapuk] [ vsts-deployment-gates] a kibocsátási folyamat célszerű használni.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-147">[Deployment gates][vsts-deployment-gates] should be used in your release pipeline.</span></span> <span data-ttu-id="fb8f1-148">Ez lehetővé teszi a figyelési adatok (például az incidenskezelés vagy egyéb bespoke rendszerekhez) külső rendszerekkel való társítás e kiadás támogatni kell meghatározni.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-148">This lets you leverage monitoring data in association with external systems (for example, incident management or additional bespoke systems) to determine whether a release should be promoted.</span></span>

- <span data-ttu-id="fb8f1-149">A kibocsátási folyamat manuális beavatkozásra szükség, ahol a [jóváhagyások] [ vsts-approvals] funkciót.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-149">Where manual intervention in a release pipeline is required, use the [approvals][vsts-approvals] functionality.</span></span>

- <span data-ttu-id="fb8f1-150">Fontolja meg [Application Insights] [ application-insights] és a kibocsátási folyamat a lehető leghamarabb további figyelési eszközöket.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-150">Consider using [Application Insights][application-insights] and additional monitoring tools as early as possible in your release pipeline.</span></span> <span data-ttu-id="fb8f1-151">Sok szervezet csak saját éles környezetben figyelésének megkezdése előtt.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-151">Many organizations only begin monitoring in their production environment.</span></span> <span data-ttu-id="fb8f1-152">A más környezetek figyelésével, a fejlesztési folyamatot a korábbi hibák azonosítása és a problémák elkerülése érdekében az éles környezetben.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-152">By monitoring your other environments, you can identify bugs earlier in the development process and avoid issues in your production environment.</span></span>

## <a name="deploy-the-scenario"></a><span data-ttu-id="fb8f1-153">A forgatókönyv megvalósításához</span><span class="sxs-lookup"><span data-stu-id="fb8f1-153">Deploy the scenario</span></span>

### <a name="prerequisites"></a><span data-ttu-id="fb8f1-154">Előfeltételek</span><span class="sxs-lookup"><span data-stu-id="fb8f1-154">Prerequisites</span></span>

- <span data-ttu-id="fb8f1-155">Meglévő Azure-fiókkal kell rendelkeznie.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-155">You must have an existing Azure account.</span></span> <span data-ttu-id="fb8f1-156">Ha nem rendelkezik Azure-előfizetéssel, mindössze néhány perc alatt létrehozhat egy [ingyenes fiókot](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) a virtuális gép létrehozásának megkezdése előtt.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-156">If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.</span></span>

- <span data-ttu-id="fb8f1-157">Jelentkezzen az Azure DevOps szervezet számára.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-157">You must sign up for an Azure DevOps organization.</span></span> <span data-ttu-id="fb8f1-158">További információkért lásd: [a rövid útmutató: Hozzon létre a szervezet][vsts-account-create].</span><span class="sxs-lookup"><span data-stu-id="fb8f1-158">For more information, see [Quickstart: Create your organization][vsts-account-create].</span></span>

### <a name="walk-through"></a><span data-ttu-id="fb8f1-159">Az útmutatóban</span><span class="sxs-lookup"><span data-stu-id="fb8f1-159">Walk-through</span></span>

<span data-ttu-id="fb8f1-160">A [Azure DevOps-projekt](/azure/devops-project/azure-devops-project-github) fog üzembe helyezése az App Service-csomag, az App Service-ben és a egy App Insights-erőforrást az Ön számára, valamint az Azure DevOps-projekt konfigurálása az Ön számára.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-160">The [Azure DevOps project](/azure/devops-project/azure-devops-project-github) will deploy an App Service Plan, App Service, and an App Insights resource for you, as well as configure the Azure DevOps project for you.</span></span>

<span data-ttu-id="fb8f1-161">Az Azure DevOps-projekt üzembe helyezte, és a létrehozás befejezése után tekintse át a társított kódmódosítás szükséges, munkaelemek, és a vizsgálati eredmények.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-161">Once you've deployed the Azure DevOps project and the build is completed, review the associated code changes, work items, and test results.</span></span> <span data-ttu-id="fb8f1-162">Megfigyelheti, hogy nincs teszt eredményei jelennek meg, mert a kód nem tartalmaz semmilyen tesztek futtatásához.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-162">You will notice that no test results are displayed, because the code does not contain any tests to run.</span></span>

<span data-ttu-id="fb8f1-163">A projekt létrehoz egy kibocsátási folyamatok és a folyamatos készregyártás eseményindítója, a fejlesztési környezetbe az alkalmazás üzembe helyezése.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-163">The project creates a release pipeline and continuous deployment trigger, deploying our application into the Dev environment.</span></span> <span data-ttu-id="fb8f1-164">A folyamatos üzembe helyezési folyamat részeként jelenhet meg, amelyek több környezetet kiadások.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-164">As part of a continuous deployment process, you may see releases that span multiple environments.</span></span> <span data-ttu-id="fb8f1-165">Egy kiadási is kiterjedhet mindkét infrastruktúra (például az infrastruktúra mint kód technikák használatával), és az alkalmazáscsomagok szükséges utáni konfigurációs feladatok együtt is üzembe helyezheti.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-165">A release can span both infrastructure (using techniques such as infrastructure-as-code), and can also deploy the application packages required along with any post-configuration tasks.</span></span>

## <a name="pricing"></a><span data-ttu-id="fb8f1-166">Díjszabás</span><span class="sxs-lookup"><span data-stu-id="fb8f1-166">Pricing</span></span>

<span data-ttu-id="fb8f1-167">Azure-fejlesztési és üzemeltetési költségek a felhasználók a szervezet más tényezők, például a számát egyidejű build/verziókban szükséges és tesztelési felhasználók száma, valamint a hozzáférést igénylő függenek.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-167">Azure DevOps costs depend on the number of users in your organization that require access, along with other factors like the number of concurrent build/releases required and number of test users.</span></span> <span data-ttu-id="fb8f1-168">További információkért lásd: [Azure DevOps-díjszabás][vsts-pricing-page].</span><span class="sxs-lookup"><span data-stu-id="fb8f1-168">For more information, see [Azure DevOps pricing][vsts-pricing-page].</span></span>

<span data-ttu-id="fb8f1-169">Ez [díjkalkulátor] [ vsts-pricing-calculator] becsült biztosít a futó Azure DevOps-20 felhasználó.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-169">This [pricing calculator][vsts-pricing-calculator] provides an estimate for running Azure DevOps with 20 users.</span></span>

<span data-ttu-id="fb8f1-170">Az Azure DevOps felhasználónkénti havi rendszerességgel történik.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-170">Azure DevOps is billed on a per-user per-month basis.</span></span> <span data-ttu-id="fb8f1-171">További díjakat lehet szükség, kívül semmilyen további tesztfelhasználók vagy alapszintű felhasználói licencek egyidejű folyamatok függ.</span><span class="sxs-lookup"><span data-stu-id="fb8f1-171">There may be additional charges dependent upon concurrent pipelines needed, in addition to any additional test users or user basic licenses.</span></span>

## <a name="related-resources"></a><span data-ttu-id="fb8f1-172">Kapcsolódó források (lehet, hogy a cikkek angol nyelvűek)</span><span class="sxs-lookup"><span data-stu-id="fb8f1-172">Related resources</span></span>

<span data-ttu-id="fb8f1-173">Tekintse át a következő források további információ a CI/CD-ről és az Azure DevOps:</span><span class="sxs-lookup"><span data-stu-id="fb8f1-173">Review the following resources to learn more about CI/CD and Azure DevOps:</span></span>

- <span data-ttu-id="fb8f1-174">[Mit jelent a DevOps?][devops-whatis]</span><span class="sxs-lookup"><span data-stu-id="fb8f1-174">[What is DevOps?][devops-whatis]</span></span>
- <span data-ttu-id="fb8f1-175">[A Microsoft - fejlesztési és üzemeltetési munka az Azure DevOps][devops-microsoft]</span><span class="sxs-lookup"><span data-stu-id="fb8f1-175">[DevOps at Microsoft - How we work with Azure DevOps][devops-microsoft]</span></span>
- <span data-ttu-id="fb8f1-176">[Lépésről lépésre haladó oktatóanyagok: A DevOps és az Azure DevOps][devops-with-vsts]</span><span class="sxs-lookup"><span data-stu-id="fb8f1-176">[Step-by-step Tutorials: DevOps with Azure DevOps][devops-with-vsts]</span></span>
- <span data-ttu-id="fb8f1-177">[Fejlesztési és üzemeltetési ellenőrzőlista][devops-checklist]</span><span class="sxs-lookup"><span data-stu-id="fb8f1-177">[DevOps Checklist][devops-checklist]</span></span>
- <span data-ttu-id="fb8f1-178">[CI/CD-folyamat létrehozása a .NET-hez az Azure DevOps-projekt][devops-project-create]</span><span class="sxs-lookup"><span data-stu-id="fb8f1-178">[Create a CI/CD pipeline for .NET with the Azure DevOps project][devops-project-create]</span></span>

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
