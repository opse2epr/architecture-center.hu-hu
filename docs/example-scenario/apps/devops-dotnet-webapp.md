---
title: CI-/CD-folyamat az Azure DevOps-szal
description: .NET-alkalmazást hozhat létre, és közzéteheti az Azure Web Appsben az Azure DevOps használatával.
author: christianreddington
ms.date: 07/11/18
ms.openlocfilehash: 97f16b2d3d9c15bc6f5db6fad4c9d8097243ad3d
ms.sourcegitcommit: 0a31fad9b68d54e2858314ca5fe6cba6c6b95ae4
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/13/2018
ms.locfileid: "51610787"
---
# <a name="cicd-pipeline-with-azure-devops"></a>CI-/CD-folyamat az Azure DevOps-szal

Fejlesztési és üzemeltetési a fejlesztési, minőségbiztosítási és IT-üzemeltetési integrációt. Fejlesztési és üzemeltetési egységes kulturális környezet és a folyamatok rengeteg szükséges szoftvereket.

A példaforgatókönyv azt szemlélteti, hogyan fejlesztői csapatok az Azure DevOps az Azure App Service .NET kétrétegű webes alkalmazás üzembe helyezése. A webalkalmazás platformszolgáltatási (PaaS) szolgáltatásokra alsóbb rétegbeli Azure platformjától függ. Ez a dokumentum is mutat meg néhány szempontot, hogy meg kell győződnie, ilyen esetben Azure Platformszolgáltatás használatának tervezésekor.

Folyamatos integráció és Készregyártás (CI/CD) használó alkalmazások fejlesztését modern megközelítését bevezetése, megismerheti, hogyan érték egy robusztus építés, tesztelés, üzembe helyezési és figyelési szolgáltatás segítségével a felhasználók számára a kézbesítésének gyorsítását. Szervezetek által a platform használatával, mint például az Azure DevOps és az Azure App Service szolgáltatásokban, összpontosíthat támogató infrastruktúra felügyeletét, hanem a forgatókönyv fejlesztését.

## <a name="relevant-use-cases"></a>Alkalmazási helyzetek

Vegye figyelembe a DevOps esetében a következő esetekben használja:

* Gyorsabb alkalmazásfejlesztés és fejlesztési életciklusa
* Épület minőségét és a egy automatizált összeállítási és a kibocsátási folyamat konzisztencia

## <a name="architecture"></a>Architektúra

![Az Azure-összetevőket egy fejlesztési és üzemeltetési a forgatókönyvet, az Azure DevOps és az Azure App Service-ben részt vevő architektúrájának áttekintése][architecture]

Ebben a forgatókönyvben egy .NET-webalkalmazás Azure DevOps használatával CI/CD-folyamat magában foglalja. A áramlanak keresztül az adatok a forgatókönyv a következő:

1. Alkalmazás forráskódjának módosítása.
2. Az alkalmazás kódja és a Web Apps web.config fájl véglegesítése.
3. A folyamatos integráció indítja az alkalmazás fordítását és az egységteszteket.
4. Folyamatos készregyártás eseményindítója vezényli az alkalmazás-összetevők *a környezetspecifikus paraméterezni a konfigurációs értékeket*.
5. Üzembe helyezés az Azure App Service-ben.
6. Az Azure Application Insights állapot-, teljesítmény- és használati adatokat gyűjt és elemez.
7. Állapot-, teljesítmény- és használati információk áttekintése.

### <a name="components"></a>Összetevők

* [Az Azure DevOps] [ vsts] egy szolgáltatás a fejlesztési életciklus-végpontok kezelésére szolgáló &mdash; felügyeleti kódot, és továbbra is készítése és kiadása a tervezési és a projekt felügyelet alól.
* [Az Azure Web Apps] [ web-apps] mobil háttérrendszer és a egy PaaS szolgáltatás REST API-k, a webalkalmazások üzemeltetéséhez. Amíg ez a cikk .NET összpontosít, többféle módon további szükséges fejlesztési platformon támogatott.
* [Az Application Insights] [ application-insights] van egy belső, bővíthető alkalmazásteljesítmény-felügyeleti (APM) szolgáltatás webfejlesztőknek, több platformon.

### <a name="alternative-devops-tooling-options"></a>Alternatív fejlesztési és üzemeltetési eszközök beállításai

Ez a cikk foglalkozik az Azure DevOps, miközben [Team Foundation Server] [ team-foundation-server] helyszíni helyettesítő használható. Másik megoldásként is használhat technológiák csoportja a nyílt forráskódú fejlesztési folyamat [Jenkins][jenkins-on-azure].

Az infrastruktúra mint kód szempontjából [Azure Resource Manager-sablonok] [ arm-templates] kerültek, célszerűbb a részeként az Azure DevOps-projekt, de [Terraform] [ terraform] vagy [Chef][chef]. Ha inkább egy infrastruktúra--szolgáltatásként (IaaS)-alapú üzembe helyezés és a szükséges konfiguráció kezelése, érdemes lehet akár [Azure Automation Állapotkonfiguráció][desired-state-configuration], [ Az Ansible][ansible], vagy [Chef][chef].

### <a name="alternatives-to-azure-web-apps"></a>Alternatív megoldások az Azure Web Appshez

Érdemes lehet üzemeltetése az Azure Web Apps az alábbi lehetőségeket:

* [Az Azure Virtual Machines] [ compare-vm-hosting] &mdash; számítási feladatokhoz, vagy vezérlő egy magas szintű igénylő, operációsrendszer-összetevők és szolgáltatások, amelyek nem lehetséges a Web Apps (például a Windows GAC, vagy a COM) függ.
* [A Service Fabric] [ service-fabric] &mdash; egy jó választás, ha a munkaterhelés-architektúra irányul, hogy körül elosztott összetevőket, amelyek üzembe helyezve, és futtassa a vezérlő magas fokú fürtök között. A Service Fabric is használható tárolók üzemeltetéséhez.
* [Az Azure Functions] [ azure-functions] – Ha a munkaterhelés-architektúra részletes köré épülő hatékony kiszolgáló nélküli megközelítés elosztott, hol találhatók az egyes összetevők csak minimális függőségek igénylő összetevői részletesebben igény szerinti (nem folyamatos) futtatásához szükséges, és a vezénylési összetevők, nem szükséges.

Ez [döntési fa](/azure/architecture/guide/technology-choices/compute-decision-tree) segíthet a helyes elérési útjával kiválasztásakor figyelembe az áttelepítéshez.

### <a name="devops"></a>DevOps

**[Folyamatos integrációs (CI)] [ continuous-integration]**  fenntart egy stabil build rendszeresen kicsi, gyakran módosítások végrehajtása a megosztott kódbázis több fejlesztőkkel. A folyamatos integrációs folyamat részeként a következőket kell elvégeznie:
* Kisebb kódmódosítások gyakran véglegesítése. Kerülje a kötegelés nagyobb és összetettebb módosítások, amelyek sikeresen egyesíteni nehezebb lehet.
* Egység magatartási tesztelését, az alkalmazás-összetevők a megfelelő kód lefedettség, beleértve a vizsgálatot a véleménypontszáma elérési utak.
* Győződjön meg arról, a build a megosztott master (vagy trönk) ág vonatkozóan fut le. Ez az ág stabil és telepítésként"kész" kell lennie. Hiányos vagy folyamatban lévő módosításokat kell elkülöníteni egy külön ágban végzett gyakori "integrációs továbbítsa" összevonása később ütközések elkerülése érdekében.

**[Folyamatos kézbesítési (CD)] [ continuous-delivery]**  mutatja be, nem csak egy stabil hozhat létre, de egy stabil központi telepítést. Ez lehetővé teszi, hogy környezetspecifikus konfigurációs és a egy mechanizmust igénylő megfelelően állítja ezeket az értékeket egy kicsit összetettebb, CD megvalósításához. Más CD szempontok közé tartoznak a következők:
* Elegendő integrációs tesztelési lefedettséget szükség, hogy vannak-e konfigurálva a különböző összetevők ellenőrzése és működő megfelelően – teljes körű.
* CD-t is szükségessé, beállítása és környezetre jellemző adatok alaphelyzetbe állítása és felügyelete az adatbázis-séma verziója.
* A folyamatos teljesítés is ki kell betölteni a teszteléshez és a felhasználói elfogadás tesztelési környezetekben.
* Folyamatos készregyártás számos előnyt biztosít folyamatos figyelését, ideális esetben minden környezetben keresztül.
* A konzisztencia és a központi telepítések és az integráció tesztelésre környezetekben megbízhatóságát megkönnyíti létrehozása és konfigurálása, a üzemeltetési infrastruktúrájának alkalmazott. Ez a lényegesen megkönnyíti a felhőalapú számítási feladatokhoz. További információkért lásd: [infrastruktúra mint kód][infra-as-code].
* Folyamatos teljesítés megkezdése érdekében a projekt életciklusának lehető legkorábbi észleléséhez. Később a megkezdése, annál nehéz lesz építhetnek be.
* Integráció és az egységteszteket alkalmazás szolgáltatásaival mint az azonos prioritású kell adni.
* Környezet-agnosztikus központi telepítési csomagok használata, és a kibocsátási folyamat környezetspecifikus konfiguráció kezelése.
* A release Management-eszközöket, vagy egy hardveres-security-modulra (HSM) meghívásával-és nagybetűket konfigurációs védelme vagy [Azure Key Vault] [ azure-key-vault] a kibocsátási folyamat során. Ne tároljon bizalmas konfigurációs verziókövetés belül.

**Folyamatos tanulás**. Alkalmazás alkalmazásteljesítmény-figyelés (APM) eszközök, mint például a CD-környezet a leghatékonyabb figyelést kerül a [Application Insights][application-insights]. Alkalmazás számítási feladat figyelése elegendő mélysége fontos megérteni a hibák vagy a teljesítmény terhelés alatt. Az Application Insights engedélyezése VSTS integrálható [folyamatos figyelés a CD-folyamat][app-insights-cd-monitoring]. Ez használható a következő szintre, emberi beavatkozás vagy visszaállítási nélkül automatikus metódushívásainak engedélyezze, ha a riasztás észlelhető.

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

Vegye fontolóra a [jellemző tervezési minták a rugalmassághoz] [ design-patterns-resiliency] ahol lehetséges.

Számos annak [ajánlott eljárások az App Service] [ resiliency-app-service] a az Azure Architecture Centert.

Rugalmas megoldások tervezésével kapcsolatos általános útmutatásért lásd: [rugalmas alkalmazások tervezése az Azure][resiliency].

## <a name="deploy-the-scenario"></a>A forgatókönyv megvalósításához

### <a name="prerequisites"></a>Előfeltételek

* Meglévő Azure-fiókkal kell rendelkeznie. Ha nem rendelkezik Azure-előfizetéssel, első lépésként létrehozhat egy [ingyenes][azure-free-account] fiókot.
* Jelentkezzen az Azure DevOps szervezet számára. További információkért lásd: [a rövid útmutató: a szervezete létrehozása][vsts-account-create].

### <a name="walk-through"></a>Az útmutatóban

Ebben a forgatókönyvben a CI/CD-folyamatok létrehozására az Azure DevOps-projekt fogja használni.

Az Azure DevOps-projekt fogja üzembe helyezése az App Service-csomag, az App Service-ben és a egy App Insights-erőforrást az Ön számára, valamint az Azure DevOps-projekt konfigurálása az Ön számára.

Az Azure DevOps-projekt üzembe helyezte, és a létrehozás befejezése után tekintse át a társított kódmódosítás szükséges, munkaelemek, és a vizsgálati eredmények. Megfigyelheti, hogy nincs teszt eredményei jelennek meg, mert a kód nem tartalmaz semmilyen tesztek futtatásához.

Tekintse át a kiadás definíciókat. Figyelje meg, hogy a kibocsátási folyamat be lett állítva, ad ki az alkalmazást a fejlesztői környezetbe. Figyelje meg, hogy egy **a folyamatos készregyártás eseményindítója** állítható be a **Drop** összetevő, az automatikus kiadást a fejlesztési környezet létrehozása. A folyamatos üzembe helyezési folyamat részeként jelenhet meg, amelyek több környezetet kiadások. Egy kiadási is kiterjedhet mindkét infrastruktúra (például az infrastruktúra mint kód technikák használatával), és az alkalmazáscsomagok szükséges utáni konfigurációs feladatok együtt is üzembe helyezheti.

## <a name="additional-considerations"></a>Néhány fontos megjegyzés

* Fontolja meg az egyik kihasználva a [a jogkivonatok feladatok] [ vsts-tokenization] a VSTS-piactéren elérhető.
* Fontolja meg a [üzembe helyezés: Azure Key Vault] [ download-keyvault-secrets] VSTS-feladat, a kiadás az Azure Key Vault titkos kódok töltheti le. Ezután használhatja ezeket a titkos kulcsok változókként a kiadási definíció a tárolása verziókövetési az elkerülése érdekében.
* Fontolja meg [változók kiadási] [ vsts-release-variables] meghajtó konfigurációjának a módosításához a környezetek a kiadási-definíciókban. Kiadási változók hatóköre beállítható egy teljes kiadása, vagy egy adott környezetben. Változók használata a titkos adatok, győződjön meg arról, hogy bejelöli-e a lakat ikonra.
* Fontolja meg [üzembe helyezési kapuk] [ vsts-deployment-gates] a kibocsátási folyamat. Ez lehetővé teszi a figyelési adatok (például az incidenskezelés vagy egyéb bespoke rendszerekhez) külső rendszerekkel való társítás e kiadás támogatni kell meghatározni.
* Amennyiben a kibocsátási folyamat manuális beavatkozásra szükség, fontolja meg a [jóváhagyások] [ vsts-approvals] funkciót.
* Fontolja meg [Application Insights] [ application-insights] és a kibocsátási folyamat a lehető leghamarabb további figyelési eszközöket. Sok szervezet csak megfigyelésének saját éles környezetben; a más környezetek figyelésével, a fejlesztési folyamatot a korábbi hibák azonosítása és a problémák elkerülése érdekében az éles környezetben.

## <a name="pricing"></a>Díjszabás

Azure-fejlesztési és üzemeltetési költségek a felhasználók a szervezet más tényezők, például a számát egyidejű build/verziókban szükséges és tesztelési felhasználók száma, valamint a hozzáférést igénylő függenek. További információkért lásd: [Azure DevOps-díjszabás][vsts-pricing-page].

* [Az Azure DevOps] [ vsts-pricing-calculator] egy szolgáltatás, amely lehetővé teszi, hogy a fejlesztési életciklus kezelése. Azt a fizetett havi felhasználónkénti alapon. További díjakat lehet szükség, kívül semmilyen további tesztfelhasználók vagy alapszintű felhasználói licencek egyidejű folyamatok függ.

## <a name="related-resources"></a>Kapcsolódó források (lehet, hogy a cikkek angol nyelvűek)

* [Mit jelent a DevOps?][devops-whatis]
* [A Microsoft - fejlesztési és üzemeltetési munka az Azure DevOps][devops-microsoft]
* [Lépésről lépésre haladó oktatóanyagok: A DevOps és az Azure DevOps][devops-with-vsts]
* [Fejlesztési és üzemeltetési ellenőrzőlista][devops-checklist]
* [CI/CD-folyamat létrehozása a .NET-hez az Azure DevOps-projekt][devops-project-create]

<!-- links -->
[ansible]: /azure/ansible/
[application-insights]: /azure/application-insights/app-insights-overview
[app-service-reference-architecture]: ../../reference-architectures/app-service-web-app/basic-web-app.md
[azure-free-account]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
[arm-templates]: /azure/azure-resource-manager/resource-group-overview#template-deployment
[architecture]: ./media/architecture-devops-dotnet-webapp.png
[availability]: /azure/architecture/checklist/availability
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
[vsts-pricing-page]: https://azure.microsoft.com/pricing/details/visual-studio-team-services/
[vsts-release-variables]: /vsts/pipelines/release/variables?view=vsts&tabs=batch
[vsts-tokenization]: https://marketplace.visualstudio.com/search?term=token&target=VSTS&category=All%20categories&sortBy=Relevance
[azure-key-vault]: /azure/key-vault/key-vault-overview
[infra-as-code]: https://blogs.msdn.microsoft.com/mvpawardprogram/2018/02/13/infrastructure-as-code/
[team-foundation-server]: https://visualstudio.microsoft.com/tfs/
[infra-as-code]: https://blogs.msdn.microsoft.com/mvpawardprogram/2018/02/13/infrastructure-as-code/
[service-fabric]: /azure/service-fabric/
[azure-functions]: /azure/azure-functions/
[azure-containers]: https://azure.microsoft.com/overview/containers/
[compare-vm-hosting]: /azure/app-service/choose-web-site-cloud-service-vm
[app-insights-cd-monitoring]: /azure/application-insights/app-insights-vsts-continuous-monitoring
[azure-region-pair-bcdr]: /azure/best-practices-availability-paired-regions
[devops-project-create]: /azure/devops-project/azure-devops-project-aspnet-core
[security]: /azure/security/