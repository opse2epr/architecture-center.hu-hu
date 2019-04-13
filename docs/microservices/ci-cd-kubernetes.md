---
title: A mikroszolgáltatásokat CI/CD-folyamat felépítésével bajlódnia a kubernetes használatával
description: Egy példa a CI/CD folyamatot a mikroszolgáltatásokat helyez üzembe az Azure Kubernetes Service (AKS) ismerteti.
author: MikeWasson
ms.date: 04/11/2019
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: microservices
ms.openlocfilehash: e7da8a1b4111fb7856b919b40d033833a4e475e0
ms.sourcegitcommit: d58e6b2b891c9c99e951c59f15fce71addcb96b1
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/12/2019
ms.locfileid: "59533196"
---
<!-- markdownlint-disable MD040 -->

# <a name="building-a-cicd-pipeline-for-microservices-on-kubernetes"></a><span data-ttu-id="76d35-103">A mikroszolgáltatásokat CI/CD-folyamat felépítésével bajlódnia a kubernetes használatával</span><span class="sxs-lookup"><span data-stu-id="76d35-103">Building a CI/CD pipeline for microservices on Kubernetes</span></span>

<span data-ttu-id="76d35-104">Ez a mikroszolgáltatási architektúra megbízható CI/CD-folyamat létrehozásához kihívást jelenthet.</span><span class="sxs-lookup"><span data-stu-id="76d35-104">It can be challenging to create a reliable CI/CD process for a microservices architecture.</span></span> <span data-ttu-id="76d35-105">Az egyes csapatok felszabadítása nélkül más csapatokkal megszakítása vagy az alkalmazás egésze destabilizing szolgáltatások gyorsan és megbízhatóan, képesnek kell lennie.</span><span class="sxs-lookup"><span data-stu-id="76d35-105">Individual teams must be able to release services quickly and reliably, without disrupting other teams or destabilizing the application as a whole.</span></span>

<span data-ttu-id="76d35-106">Ez a cikk ismerteti egy példa a CI/CD folyamatot a mikroszolgáltatásokat helyez üzembe az Azure Kubernetes Service (AKS).</span><span class="sxs-lookup"><span data-stu-id="76d35-106">This article describes an example CI/CD pipeline for deploying microservices to Azure Kubernetes Service (AKS).</span></span> <span data-ttu-id="76d35-107">Minden csoport és a projekt nem egyezik, így nem ez a cikk, hard-and-fast szabályai szerint.</span><span class="sxs-lookup"><span data-stu-id="76d35-107">Every team and project is different, so don't take this article as a set of hard-and-fast rules.</span></span> <span data-ttu-id="76d35-108">Ehelyett azt azt jelentette a saját CI/CD-folyamat tervezéséhez kiindulási pontként.</span><span class="sxs-lookup"><span data-stu-id="76d35-108">Instead, it's meant to be a starting point for designing your own CI/CD process.</span></span>

<span data-ttu-id="76d35-109">Az itt ismertetett példa folyamat létrehoztunk egy mikroszolgáltatás-alapú referenciaimplementációt, a Drone Delivery alkalmazást, amely a nevű [GitHub][ri].</span><span class="sxs-lookup"><span data-stu-id="76d35-109">The example pipeline described here was created for a microservices reference implementation called the Drone Delivery application, which you can find on [GitHub][ri].</span></span> <span data-ttu-id="76d35-110">Az alkalmazás forgatókönyv [Itt](./design/index.md#reference-implementation).</span><span class="sxs-lookup"><span data-stu-id="76d35-110">The application scenario is described [here](./design/index.md#reference-implementation).</span></span>

<span data-ttu-id="76d35-111">A folyamat célja is a következőképpen épül fel:</span><span class="sxs-lookup"><span data-stu-id="76d35-111">The goals of the pipeline can be summarized as follows:</span></span>

- <span data-ttu-id="76d35-112">Csapatok létrehozhat és a szolgáltatások egymástól függetlenül üzembe helyezéséhez.</span><span class="sxs-lookup"><span data-stu-id="76d35-112">Teams can build and deploy their services independently.</span></span>
- <span data-ttu-id="76d35-113">Kód módosítása, hogy a CI folyamat a rendszer automatikusan telepíti az utolsóig üzemi környezetben.</span><span class="sxs-lookup"><span data-stu-id="76d35-113">Code changes that pass the CI process are automatically deployed to a production-like environment.</span></span>
- <span data-ttu-id="76d35-114">Minőségi kapuk a rendszer érvényesíti a folyamat minden szakaszában.</span><span class="sxs-lookup"><span data-stu-id="76d35-114">Quality gates are enforced at each stage of the pipeline.</span></span>
- <span data-ttu-id="76d35-115">A szolgáltatás egy új verziója is telepíthető az előző verzió párhuzamosan lesz.</span><span class="sxs-lookup"><span data-stu-id="76d35-115">A new version of a service can be deployed side by side with the previous version.</span></span>

<span data-ttu-id="76d35-116">A további háttér-információkért lásd: [CI/CD-mikroszolgáltatás-architektúrákat](./ci-cd.md).</span><span class="sxs-lookup"><span data-stu-id="76d35-116">For more background, see [CI/CD for microservices architectures](./ci-cd.md).</span></span>

## <a name="assumptions"></a><span data-ttu-id="76d35-117">Előfeltételek</span><span class="sxs-lookup"><span data-stu-id="76d35-117">Assumptions</span></span>

<span data-ttu-id="76d35-118">Ebben a példában alkalmazásában az alábbiakban néhány feltételezéseket a fejlesztői csapat és a kód alap:</span><span class="sxs-lookup"><span data-stu-id="76d35-118">For purposes of this example, here are some assumptions about the development team and the code base:</span></span>

- <span data-ttu-id="76d35-119">A kódtár egy monorepo mappák mikroszolgáltatás szerint vannak rendezve.</span><span class="sxs-lookup"><span data-stu-id="76d35-119">The code repository is a monorepo, with folders organized by microservice.</span></span>
- <span data-ttu-id="76d35-120">A csapat stratégiáját az elágazási alapján [trönk-alapú fejlesztési](https://trunkbaseddevelopment.com/).</span><span class="sxs-lookup"><span data-stu-id="76d35-120">The team's branching strategy is based on [trunk-based development](https://trunkbaseddevelopment.com/).</span></span>
- <span data-ttu-id="76d35-121">A csapata által használt [kiadási ágakat](/azure/devops/repos/git/git-branching-guidance?view=azure-devops#manage-releases) kiadások kezeléséhez.</span><span class="sxs-lookup"><span data-stu-id="76d35-121">The team uses [release branches](/azure/devops/repos/git/git-branching-guidance?view=azure-devops#manage-releases) to manage releases.</span></span> <span data-ttu-id="76d35-122">Mindegyik mikroszolgáltatás külön kiadások jön létre.</span><span class="sxs-lookup"><span data-stu-id="76d35-122">Separate releases are created for each microservice.</span></span>
- <span data-ttu-id="76d35-123">A CI/CD-folyamat használja [Azure folyamatok](/azure/devops/pipelines/?view=azure-devops) hozhat létre, tesztelheti, és a mikroszolgáltatások üzembe az aks-ben.</span><span class="sxs-lookup"><span data-stu-id="76d35-123">The CI/CD process uses [Azure Pipelines](/azure/devops/pipelines/?view=azure-devops) to build, test, and deploy the microservices to AKS.</span></span>
- <span data-ttu-id="76d35-124">Mindegyik mikroszolgáltatás a tárolórendszerképeket vannak tárolva [Azure Container Registry](/azure/container-registry/).</span><span class="sxs-lookup"><span data-stu-id="76d35-124">The container images for each microservice are stored in [Azure Container Registry](/azure/container-registry/).</span></span>
- <span data-ttu-id="76d35-125">A csapat Helm-diagramok használja az egyes mikroszolgáltatások csomagolását.</span><span class="sxs-lookup"><span data-stu-id="76d35-125">The team uses Helm charts to package each microservice.</span></span>

<span data-ttu-id="76d35-126">Ilyen Előfeltevések meghajtó a CI/CD-folyamat az adott részletek nagy része.</span><span class="sxs-lookup"><span data-stu-id="76d35-126">These assumptions drive many of the specific details of the CI/CD pipeline.</span></span> <span data-ttu-id="76d35-127">Azonban az alapszintű megközelítés az itt leírtak szerint más folyamatokat, eszközöket és szolgáltatásokat, például a Jenkins vagy a Docker Hub módosítani kell.</span><span class="sxs-lookup"><span data-stu-id="76d35-127">However, the basic approach described here be adapted for other processes, tools, and services, such as Jenkins or Docker Hub.</span></span>

## <a name="validation-builds"></a><span data-ttu-id="76d35-128">Érvényesítési buildek</span><span class="sxs-lookup"><span data-stu-id="76d35-128">Validation builds</span></span>

<span data-ttu-id="76d35-129">Tegyük fel, hogy a fejlesztő neve a kézbesítési szolgáltatás mikroszolgáltatások dolgozik.</span><span class="sxs-lookup"><span data-stu-id="76d35-129">Suppose that a developer is working on a microservice called the Delivery Service.</span></span> <span data-ttu-id="76d35-130">Egy új szolgáltatás fejlesztésekor a fejlesztői egy funkciót a főágban kód ellenőrzi.</span><span class="sxs-lookup"><span data-stu-id="76d35-130">While developing a new feature, the developer checks code into a feature branch.</span></span> <span data-ttu-id="76d35-131">Szabályok szerint funkció ágak elnevezése `feature/*`.</span><span class="sxs-lookup"><span data-stu-id="76d35-131">By convention, feature branches are named `feature/*`.</span></span>

![CI/CD-munkafolyamat](./images/aks-cicd-1.png)

<span data-ttu-id="76d35-133">A build-definíciós fájl tartalmaz egy eseményindítót, amely szűri a kiadásiág-nevet és a forrás elérési útja:</span><span class="sxs-lookup"><span data-stu-id="76d35-133">The build definition file includes a trigger that filters by the branch name and the source path:</span></span>

```yaml
trigger:
  batch: true
  branches:
    include:
    - master
    - feature/*
    - topic/*

    exclude:
    - feature/experimental/*
    - topic/experimental/*

  paths:
     include:
     - /src/shipping/delivery/
```

<span data-ttu-id="76d35-134">&#11162;Tekintse meg a [forrásfájl](https://github.com/mspnp/microservices-reference-implementation/blob/master/src/shipping/delivery/azure-pipelines-validation.yml).</span><span class="sxs-lookup"><span data-stu-id="76d35-134">&#11162; See the [source file](https://github.com/mspnp/microservices-reference-implementation/blob/master/src/shipping/delivery/azure-pipelines-validation.yml).</span></span>

<span data-ttu-id="76d35-135">Ezt a módszert használja, minden egyes csapat a saját build folyamat lehet.</span><span class="sxs-lookup"><span data-stu-id="76d35-135">Using this approach, each team can have its own build pipeline.</span></span> <span data-ttu-id="76d35-136">Csak olyan kódot, amely be van jelölve, a `/src/shipping/delivery` mappa aktiválnak egy buildet, a Tartalomkézbesítési szolgáltatás.</span><span class="sxs-lookup"><span data-stu-id="76d35-136">Only code that is checked into the `/src/shipping/delivery` folder triggers a build of the Delivery Service.</span></span> <span data-ttu-id="76d35-137">A CI-build, amely megfelel a szűrő egyik ágára való leküldés a véglegesítéseket aktivál.</span><span class="sxs-lookup"><span data-stu-id="76d35-137">Pushing commits to a branch that matches the filter triggers a CI build.</span></span> <span data-ttu-id="76d35-138">Ezen a ponton a munkafolyamatban a CI-build néhány csak minimális mennyiségű kódra ellenőrzés fut:</span><span class="sxs-lookup"><span data-stu-id="76d35-138">At this point in the workflow, the CI build runs some minimal code verification:</span></span>

1. <span data-ttu-id="76d35-139">A kód felépítéséhez.</span><span class="sxs-lookup"><span data-stu-id="76d35-139">Build the code.</span></span>
1. <span data-ttu-id="76d35-140">Futtatni az egységteszteket.</span><span class="sxs-lookup"><span data-stu-id="76d35-140">Run unit tests.</span></span>

<span data-ttu-id="76d35-141">A célja, hogy a fejlesztési idő rövid, így a fejlesztő gyors visszajelzés.</span><span class="sxs-lookup"><span data-stu-id="76d35-141">The goal is to keep build times short, so the developer can get quick feedback.</span></span> <span data-ttu-id="76d35-142">Ha a szolgáltatás készen áll a egyesítése a mesterrel, a fejlesztői megnyitása új</span><span class="sxs-lookup"><span data-stu-id="76d35-142">Once the feature is ready to merge into master, the developer opens a PR.</span></span> <span data-ttu-id="76d35-143">Ez elindít egy másik CI build néhány további ellenőrzést végző:</span><span class="sxs-lookup"><span data-stu-id="76d35-143">This triggers another CI build that performs some additional checks:</span></span>

1. <span data-ttu-id="76d35-144">A kód felépítéséhez.</span><span class="sxs-lookup"><span data-stu-id="76d35-144">Build the code.</span></span>
1. <span data-ttu-id="76d35-145">Futtatni az egységteszteket.</span><span class="sxs-lookup"><span data-stu-id="76d35-145">Run unit tests.</span></span>
1. <span data-ttu-id="76d35-146">A futtatókörnyezet tárolólemezkép létrehozásához.</span><span class="sxs-lookup"><span data-stu-id="76d35-146">Build the runtime container image.</span></span>
1. <span data-ttu-id="76d35-147">Futtassa a vizsgálatot a biztonsági rések a rendszerképen.</span><span class="sxs-lookup"><span data-stu-id="76d35-147">Run vulnerability scans on the image.</span></span>

![CI/CD-munkafolyamat](./images/aks-cicd-2.png)

> [!NOTE]
> <span data-ttu-id="76d35-149">Az Azure DevOps-Adattárakkal, definiálhat [házirendek](/azure/devops/repos/git/branch-policies) ágak védelme érdekében.</span><span class="sxs-lookup"><span data-stu-id="76d35-149">In Azure DevOps Repos, you can define [policies](/azure/devops/repos/git/branch-policies) to protect branches.</span></span> <span data-ttu-id="76d35-150">Például a szabályzat szükségük CI a build sikeres létrehozása és a egy jóváhagyás a jóváhagyó egyesítése a mesterrel.</span><span class="sxs-lookup"><span data-stu-id="76d35-150">For example, the policy could require a successful CI build plus a sign-off from an approver in order to merge into master.</span></span>

## <a name="full-cicd-build"></a><span data-ttu-id="76d35-151">Teljes CI/CD-build</span><span class="sxs-lookup"><span data-stu-id="76d35-151">Full CI/CD build</span></span>

<span data-ttu-id="76d35-152">Később a csapat a kézbesítési szolgáltatás új verziójának telepítése készen áll.</span><span class="sxs-lookup"><span data-stu-id="76d35-152">At some point, the team is ready to deploy a new version of the Delivery service.</span></span> <span data-ttu-id="76d35-153">A kiadáskezelő elnevezési mintázatot a főágból hoz létre egy ágat: `release/<microservice name>/<semver>`.</span><span class="sxs-lookup"><span data-stu-id="76d35-153">The release manager creates a branch from master with this naming pattern: `release/<microservice name>/<semver>`.</span></span> <span data-ttu-id="76d35-154">Például: `release/delivery/v1.0.2`.</span><span class="sxs-lookup"><span data-stu-id="76d35-154">For example, `release/delivery/v1.0.2`.</span></span>

![CI/CD-munkafolyamat](./images/aks-cicd-3.png)

<span data-ttu-id="76d35-156">Ez az ág létrehozása fut, plusz az előző lépések mindegyikét teljes CI build aktiválása:</span><span class="sxs-lookup"><span data-stu-id="76d35-156">Creation of this branch triggers a full CI build that runs all of the previous steps plus:</span></span>

1. <span data-ttu-id="76d35-157">A tárolórendszerkép leküldése az Azure Container Registrybe.</span><span class="sxs-lookup"><span data-stu-id="76d35-157">Push the container image to Azure Container Registry.</span></span> <span data-ttu-id="76d35-158">A kép a kiadásiág-nevet származó verziószámmal van megjelölve.</span><span class="sxs-lookup"><span data-stu-id="76d35-158">The image is tagged with the version number taken from the branch name.</span></span>
2. <span data-ttu-id="76d35-159">Futtatás `helm package` csomagolása a Helm-diagramot a szolgáltatáshoz.</span><span class="sxs-lookup"><span data-stu-id="76d35-159">Run `helm package` to package the Helm chart for the service.</span></span> <span data-ttu-id="76d35-160">A diagram is van megjelölve a verziószámmal.</span><span class="sxs-lookup"><span data-stu-id="76d35-160">The chart is also tagged with a version number.</span></span>
3. <span data-ttu-id="76d35-161">A Helm-csomag leküldése a Tárolójegyzékbe.</span><span class="sxs-lookup"><span data-stu-id="76d35-161">Push the Helm package to Container Registry.</span></span>

<span data-ttu-id="76d35-162">Ha a build sikeres lesz, elindít egy üzembe helyezés (CD) folyamat használatával egy Azure-folyamatok [kibocsátási folyamatok](/azure/devops/pipelines/release/what-is-release-management).</span><span class="sxs-lookup"><span data-stu-id="76d35-162">Assuming this build succeeds, it triggers a deployment (CD) process using an Azure Pipelines [release pipeline](/azure/devops/pipelines/release/what-is-release-management).</span></span> <span data-ttu-id="76d35-163">Ez a folyamat rendelkezik az alábbi lépéseket:</span><span class="sxs-lookup"><span data-stu-id="76d35-163">This pipeline has the following steps:</span></span>

1. <span data-ttu-id="76d35-164">A Helm-diagram üzembe helyezése QA környezetben.</span><span class="sxs-lookup"><span data-stu-id="76d35-164">Deploy the Helm chart to a QA environment.</span></span>
1. <span data-ttu-id="76d35-165">Jóváhagyó jelentkezik, mielőtt a csomagot az éles környezetbe helyezi át.</span><span class="sxs-lookup"><span data-stu-id="76d35-165">An approver signs off before the package moves to production.</span></span> <span data-ttu-id="76d35-166">Lásd: [kiadási központi telepítés ellenőrzési jóváhagyások használatával](/azure/devops/pipelines/release/approvals/approvals).</span><span class="sxs-lookup"><span data-stu-id="76d35-166">See [Release deployment control using approvals](/azure/devops/pipelines/release/approvals/approvals).</span></span>
1. <span data-ttu-id="76d35-167">A Docker-rendszerképet az Azure Container Registry a termelési névtér újracímkézése.</span><span class="sxs-lookup"><span data-stu-id="76d35-167">Retag the Docker image for the production namespace in Azure Container Registry.</span></span> <span data-ttu-id="76d35-168">Például, ha az aktuális címke `myrepo.azurecr.io/delivery:v1.0.2`, az éles címke `myrepo.azurecr.io/prod/delivery:v1.0.2`.</span><span class="sxs-lookup"><span data-stu-id="76d35-168">For example, if the current tag is `myrepo.azurecr.io/delivery:v1.0.2`, the production tag is `myrepo.azurecr.io/prod/delivery:v1.0.2`.</span></span>
1. <span data-ttu-id="76d35-169">Helyezze üzembe a Helm-diagram az éles környezetben.</span><span class="sxs-lookup"><span data-stu-id="76d35-169">Deploy the Helm chart to the production environment.</span></span>

<span data-ttu-id="76d35-170">Egy monorepo, még akkor is, ezeket a feladatokat korlátozhatja az egyes mikroszolgáltatások úgy, hogy a teams telepítheti a nagy sebességű.</span><span class="sxs-lookup"><span data-stu-id="76d35-170">Even in a monorepo, these tasks can be scoped to individual microservices, so that teams can deploy with high velocity.</span></span> <span data-ttu-id="76d35-171">A folyamat néhány manuális lépésből áll: Lekéréses kérelmek jóváhagyása, a kiadási ágak létrehozása és az éles fürt üzemelő jóváhagyása.</span><span class="sxs-lookup"><span data-stu-id="76d35-171">The process has some manual steps: Approving PRs, creating release branches, and approving deployments into the production cluster.</span></span> <span data-ttu-id="76d35-172">Ezen lépések végrehajtása manuális házirend &mdash; azok sikerült automatizált, ha a szervezet részesíti előnyben.</span><span class="sxs-lookup"><span data-stu-id="76d35-172">These steps are manual by policy &mdash; they could be automated if the organization prefers.</span></span>

## <a name="isolation-of-environments"></a><span data-ttu-id="76d35-173">A környezetek elkülönítése</span><span class="sxs-lookup"><span data-stu-id="76d35-173">Isolation of environments</span></span>

<span data-ttu-id="76d35-174">Kell több környezetet, ahol a szolgáltatások, beleértve a fejlesztési, füst tesztelése, integráció tesztelése, terheléses tesztelés díjaival, és végül éles környezetben telepít.</span><span class="sxs-lookup"><span data-stu-id="76d35-174">You will have multiple environments where you deploy services, including environments for development, smoke testing, integration testing, load testing, and finally production.</span></span> <span data-ttu-id="76d35-175">Ezekben a környezetekben kell bizonyos fokú elkülönítés.</span><span class="sxs-lookup"><span data-stu-id="76d35-175">These environments need some level of isolation.</span></span> <span data-ttu-id="76d35-176">A Kubernetes választhat, hogy fizikai elkülönítése és logikai elkülönítése között.</span><span class="sxs-lookup"><span data-stu-id="76d35-176">In Kubernetes, you have a choice between physical isolation and logical isolation.</span></span> <span data-ttu-id="76d35-177">Fizikai elkülönítése azt jelenti, hogy különálló fürt üzembe helyezése.</span><span class="sxs-lookup"><span data-stu-id="76d35-177">Physical isolation means deploying to separate clusters.</span></span> <span data-ttu-id="76d35-178">Logikai elkülönítéssel használ, a névterek és a házirendeket, a fentebb leírt módon.</span><span class="sxs-lookup"><span data-stu-id="76d35-178">Logical isolation makes use of namespaces and policies, as described earlier.</span></span>

<span data-ttu-id="76d35-179">Azt javasoljuk, hogy hozzon létre egy dedikált fürt és a fejlesztési-tesztelési környezetek esetében külön fürtben.</span><span class="sxs-lookup"><span data-stu-id="76d35-179">Our recommendation is to create a dedicated production cluster along with a separate cluster for your dev/test environments.</span></span> <span data-ttu-id="76d35-180">Logikai elkülönítés használatával a fejlesztési-tesztelési fürtön belüli környezeteket.</span><span class="sxs-lookup"><span data-stu-id="76d35-180">Use logical isolation to separate environments within the dev/test cluster.</span></span> <span data-ttu-id="76d35-181">A fejlesztési-tesztelési fürtre telepített szolgáltatások soha nem üzleti adatokat tartalmazó adattárakra hozzáféréssel kell rendelkeznie.</span><span class="sxs-lookup"><span data-stu-id="76d35-181">Services deployed to the dev/test cluster should never have access to data stores that hold business data.</span></span>

## <a name="build-process"></a><span data-ttu-id="76d35-182">Folyamat létrehozása</span><span class="sxs-lookup"><span data-stu-id="76d35-182">Build process</span></span>

<span data-ttu-id="76d35-183">Ha lehetséges, a csomag a build-folyamatoknak egy Docker-tárolóba.</span><span class="sxs-lookup"><span data-stu-id="76d35-183">When possible, package your build process into a Docker container.</span></span> <span data-ttu-id="76d35-184">Amely lehetővé teszi, hogy a kód összetevők, Docker, anélkül, hogy a build-környezet konfigurálása az összes build használatával hozhat létre.</span><span class="sxs-lookup"><span data-stu-id="76d35-184">That allows you to build your code artifacts using Docker, without needing to configure the build environment on each build machine.</span></span> <span data-ttu-id="76d35-185">Tárolóalapú folyamatának megkönnyíti a horizontális felskálázás a CI folyamat új buildelési ügynökök hozzáadásával.</span><span class="sxs-lookup"><span data-stu-id="76d35-185">A containerized build process makes it easy to scale out the CI pipeline by adding new build agents.</span></span> <span data-ttu-id="76d35-186">Emellett minden fejlesztő a csapat hozhat létre a kódot a build-tároló futtatásával.</span><span class="sxs-lookup"><span data-stu-id="76d35-186">Also, any developer on the team can build the code simply by running the build container.</span></span>

<span data-ttu-id="76d35-187">Többlépcsős buildek a Docker használatával meghatározhatja a build környezet és a futtatókörnyezet rendszerkép egy docker-fájlban.</span><span class="sxs-lookup"><span data-stu-id="76d35-187">By using multi-stage builds in Docker, you can define the build environment and the runtime image in a single Dockerfile.</span></span> <span data-ttu-id="76d35-188">Ha például itt látható, amely egy ASP.NET Core-alkalmazást hoz létre egy docker-fájlban:</span><span class="sxs-lookup"><span data-stu-id="76d35-188">For example, here's a Dockerfile that builds an ASP.NET Core application:</span></span>

```
FROM microsoft/dotnet:2.2-runtime AS base
WORKDIR /app

FROM microsoft/dotnet:2.2-sdk AS build
WORKDIR /src/Fabrikam.Workflow.Service

COPY Fabrikam.Workflow.Service/Fabrikam.Workflow.Service.csproj .
RUN dotnet restore Fabrikam.Workflow.Service.csproj

COPY Fabrikam.Workflow.Service/. .
RUN dotnet build Fabrikam.Workflow.Service.csproj -c Release -o /app

FROM build AS publish
RUN dotnet publish Fabrikam.Workflow.Service.csproj -c Release -o /app

FROM base AS final
WORKDIR /app
COPY --from=publish /app .
ENTRYPOINT ["dotnet", "Fabrikam.Workflow.Service.dll"]
```

<span data-ttu-id="76d35-189">&#11162;Tekintse meg a [forrásfájl](https://github.com/mspnp/microservices-reference-implementation/blob/master/src/shipping/workflow/Dockerfile).</span><span class="sxs-lookup"><span data-stu-id="76d35-189">&#11162; See the [source file](https://github.com/mspnp/microservices-reference-implementation/blob/master/src/shipping/workflow/Dockerfile).</span></span>

<span data-ttu-id="76d35-190">A docker-fájl több összeállítási fázisok határozza meg.</span><span class="sxs-lookup"><span data-stu-id="76d35-190">This Dockerfile defines several build stages.</span></span> <span data-ttu-id="76d35-191">Figyelje meg, hogy a szakasz nevű `base` használja az ASP.NET Core-futtatókörnyezet, az nevű szakasz során `build` a teljes ASP.NET Core SDK-t használja.</span><span class="sxs-lookup"><span data-stu-id="76d35-191">Notice that the stage named `base` uses the ASP.NET Core runtime, while the stage named `build` uses the full ASP.NET Core SDK.</span></span> <span data-ttu-id="76d35-192">A `build` szakasz segítségével hozhatók létre az ASP.NET Core-projektet.</span><span class="sxs-lookup"><span data-stu-id="76d35-192">The `build` stage is used to build the ASP.NET Core project.</span></span> <span data-ttu-id="76d35-193">De a végső futásidejű tároló hitelesítésszolgáltatókból `base`, amely csak a modult tartalmaz, és jelentősen kisebb, mint a teljes SDK lemezkép.</span><span class="sxs-lookup"><span data-stu-id="76d35-193">But the final runtime container is built from `base`, which contains just the runtime and is significantly smaller than the full SDK image.</span></span>

### <a name="building-a-test-runner"></a><span data-ttu-id="76d35-194">A tesztfuttató készítése</span><span class="sxs-lookup"><span data-stu-id="76d35-194">Building a test runner</span></span>

<span data-ttu-id="76d35-195">Egy másik jó gyakorlat, hogy a tárolóban futtatni az egységteszteket.</span><span class="sxs-lookup"><span data-stu-id="76d35-195">Another good practice is to run unit tests in the container.</span></span> <span data-ttu-id="76d35-196">Ha például a következő része, amely egy teszt futtatója összeállít egy Docker-fájlt:</span><span class="sxs-lookup"><span data-stu-id="76d35-196">For example, here is part of a Docker file that builds a test runner:</span></span>

```
FROM build AS testrunner
WORKDIR /src/tests

COPY Fabrikam.Workflow.Service.Tests/*.csproj .
RUN dotnet restore Fabrikam.Workflow.Service.Tests.csproj

COPY Fabrikam.Workflow.Service.Tests/. .
ENTRYPOINT ["dotnet", "test", "--logger:trx"]
```

<span data-ttu-id="76d35-197">A fejlesztők a Docker-fájl használatával helyileg futtasson teszteket:</span><span class="sxs-lookup"><span data-stu-id="76d35-197">A developer can use this Docker file to run the tests locally:</span></span>

```bash
docker build . -t delivery-test:1 --target=testrunner
docker run -p 8080:8080 delivery-test:1
```

<span data-ttu-id="76d35-198">A CI folyamat részeként ellenőrzési lépést is futhat a tesztek.</span><span class="sxs-lookup"><span data-stu-id="76d35-198">The CI pipeline should also run the tests as part of the build verification step.</span></span>

<span data-ttu-id="76d35-199">Vegye figyelembe, hogy ezt a fájlt használja a Docker `ENTRYPOINT` parancsot futtathatja a teszteket, nem a Docker `RUN` parancsot.</span><span class="sxs-lookup"><span data-stu-id="76d35-199">Note that this file uses the Docker `ENTRYPOINT` command to run the tests, not the Docker `RUN` command.</span></span>

- <span data-ttu-id="76d35-200">Ha használja a `RUN` parancsot, a tesztek futtatását minden alkalommal, amikor a rendszerkép létrehozásához.</span><span class="sxs-lookup"><span data-stu-id="76d35-200">If you use the `RUN` command, the tests run every time you build the image.</span></span> <span data-ttu-id="76d35-201">Használatával `ENTRYPOINT`, hogy a teszt nem vehetnek részt.</span><span class="sxs-lookup"><span data-stu-id="76d35-201">By using `ENTRYPOINT`, the tests are opt-in.</span></span> <span data-ttu-id="76d35-202">Csak ha explicit módon futnak a `testrunner` szakaszban.</span><span class="sxs-lookup"><span data-stu-id="76d35-202">They run only when you explicitly target the `testrunner` stage.</span></span>
- <span data-ttu-id="76d35-203">A sikertelen teszt nem okozza a Docker `build` parancs sikertelen lesz.</span><span class="sxs-lookup"><span data-stu-id="76d35-203">A failing test doesn't cause the Docker `build` command to fail.</span></span> <span data-ttu-id="76d35-204">Ily módon, hogy megkülönböztethesse tárolót hozhat létre a sikertelen teszt hibáinak.</span><span class="sxs-lookup"><span data-stu-id="76d35-204">That way you can distinguish container build failures from test failures.</span></span>
- <span data-ttu-id="76d35-205">Vizsgálati eredmények menthető egy csatlakoztatott kötetre.</span><span class="sxs-lookup"><span data-stu-id="76d35-205">Test results can be saved to a mounted volume.</span></span>

### <a name="container-best-practices"></a><span data-ttu-id="76d35-206">Tároló ajánlott eljárások</span><span class="sxs-lookup"><span data-stu-id="76d35-206">Container best practices</span></span>

<span data-ttu-id="76d35-207">Az alábbiakban néhány egyéb ajánlott eljárások a tárolók:</span><span class="sxs-lookup"><span data-stu-id="76d35-207">Here are some other best practices to consider for containers:</span></span>

- <span data-ttu-id="76d35-208">Adja meg a szervezeti szintű szabályai tároló címkék, verziószámozása és a fürthöz (podok, szolgáltatások és így tovább) üzembe helyezett erőforrások elnevezési szabályainak.</span><span class="sxs-lookup"><span data-stu-id="76d35-208">Define organization-wide conventions for container tags, versioning, and naming conventions for resources deployed to the cluster (pods, services, and so on).</span></span> <span data-ttu-id="76d35-209">Amely megkönnyítheti a üzembe helyezési problémák diagnosztizálásához.</span><span class="sxs-lookup"><span data-stu-id="76d35-209">That can make it easier to diagnose deployment issues.</span></span>

- <span data-ttu-id="76d35-210">A fejlesztési és tesztelési ciklus során a CI/CD-folyamat számos tárolórendszerképek fog létrehozni.</span><span class="sxs-lookup"><span data-stu-id="76d35-210">During the development and test cycle, the CI/CD process will build many container images.</span></span> <span data-ttu-id="76d35-211">Csak a kiadásban néhány ilyen rendszerképpel deduplikációra, majd csak néhányat ezen kiadás jelöltek fog első előléptetése éles környezetbe.</span><span class="sxs-lookup"><span data-stu-id="76d35-211">Only some of those images are candidates for release, and then only some of those release candidates will get promoted to production.</span></span> <span data-ttu-id="76d35-212">Rendelkezik egy egyértelmű verziókezelési stratégiát, úgy, hogy már ismeri, mely képek az éles környezetben jelenleg telepítve vannak, és vissza lehet vonni egy korábbi verziójának szükség esetén.</span><span class="sxs-lookup"><span data-stu-id="76d35-212">Have a clear versioning strategy, so that you know which images are currently deployed to production, and can roll back to a previous version if necessary.</span></span>

- <span data-ttu-id="76d35-213">Mindig üzembe helyezése nem adott tároló verzió címkéket, `latest`.</span><span class="sxs-lookup"><span data-stu-id="76d35-213">Always deploy specific container version tags, not `latest`.</span></span>

- <span data-ttu-id="76d35-214">Használat [névterek](/azure/container-registry/container-registry-best-practices#repository-namespaces) az Azure Container Registry rendszerképek éles környezetben, amely továbbra is tesztelik rendszerképekből jóváhagyott elkülönítésére.</span><span class="sxs-lookup"><span data-stu-id="76d35-214">Use [namespaces](/azure/container-registry/container-registry-best-practices#repository-namespaces) in Azure Container Registry to isolate images that are approved for production from images that are still being tested.</span></span> <span data-ttu-id="76d35-215">Ne helyezze át a production névtérből kép, mindaddig, amíg készen áll az éles üzembe helyezés.</span><span class="sxs-lookup"><span data-stu-id="76d35-215">Don't move an image into the production namespace until you're ready to deploy it into production.</span></span> <span data-ttu-id="76d35-216">Ha ez az eljárás kombinálva a tárolórendszerképek Szemantikus verziószámozást, csökkentheti az esélye, hogy véletlenül üzembe helyezése egy olyanra, amely a kiadás nem lett jóváhagyva.</span><span class="sxs-lookup"><span data-stu-id="76d35-216">If you combine this practice with semantic versioning of container images, it can reduce the chance of accidentally deploying a version that wasn't approved for release.</span></span>

- <span data-ttu-id="76d35-217">Kövesse a legalacsonyabb jogosultsági szint elvének a futó tárolók replikaadatok felhasználóként.</span><span class="sxs-lookup"><span data-stu-id="76d35-217">Follow the principle of least privilege by running containers as a nonprivileged user.</span></span> <span data-ttu-id="76d35-218">A Kubernetes, létrehozhat egy pod biztonsági szabályzatot, amely megakadályozza, hogy a tárolók nevében *legfelső szintű*.</span><span class="sxs-lookup"><span data-stu-id="76d35-218">In Kubernetes, you can create a pod security policy that prevents containers from running as *root*.</span></span> <span data-ttu-id="76d35-219">Lásd: [megakadályozza, hogy a Podok legfelső szintű jogosultságokkal fut](https://docs.bitnami.com/kubernetes/how-to/secure-kubernetes-cluster-psp/).</span><span class="sxs-lookup"><span data-stu-id="76d35-219">See [Prevent Pods From Running With Root Privileges](https://docs.bitnami.com/kubernetes/how-to/secure-kubernetes-cluster-psp/).</span></span>

## <a name="helm-charts"></a><span data-ttu-id="76d35-220">Helm-diagramok</span><span class="sxs-lookup"><span data-stu-id="76d35-220">Helm charts</span></span>

<span data-ttu-id="76d35-221">Fontolja meg a Helm használatával létrehozásának és üzembe helyezésének a szolgáltatások kezeléséhez.</span><span class="sxs-lookup"><span data-stu-id="76d35-221">Consider using Helm to manage building and deploying services.</span></span> <span data-ttu-id="76d35-222">Íme néhány a Helm, amelyek segítenek a CI/CD-funkcióit:</span><span class="sxs-lookup"><span data-stu-id="76d35-222">Here are some of the features of Helm that help with CI/CD:</span></span>

- <span data-ttu-id="76d35-223">Gyakran egyetlen mikroszolgáltatást határozza meg több Kubernetes-objektumokat.</span><span class="sxs-lookup"><span data-stu-id="76d35-223">Often a single microservice is defined by multiple Kubernetes objects.</span></span> <span data-ttu-id="76d35-224">Helm lehetővé teszi, hogy ezek az objektumok egy Helm-diagram szabálykészletnek.</span><span class="sxs-lookup"><span data-stu-id="76d35-224">Helm allows these objects to be packaged into a single Helm chart.</span></span>
- <span data-ttu-id="76d35-225">A diagram a Helm egyetlen paranccsal ahelyett, hogy a kubectl-parancsokat is telepíthető.</span><span class="sxs-lookup"><span data-stu-id="76d35-225">A chart can be deployed with a single Helm command, rather than a series of kubectl commands.</span></span>
- <span data-ttu-id="76d35-226">Diagramok használata explicit módon verziószámmal.</span><span class="sxs-lookup"><span data-stu-id="76d35-226">Charts are explicitly versioned.</span></span> <span data-ttu-id="76d35-227">Helm használatával egy verziója engedje, kiadásokban megtekintése és visszaállítása egy korábbi verzióra.</span><span class="sxs-lookup"><span data-stu-id="76d35-227">Use Helm to release a version, view releases, and roll back to a previous version.</span></span> <span data-ttu-id="76d35-228">Követési frissítések és a módosításokat, Szemantikus verziószámozást elkészíthetők visszaállítása egy korábbi verziójának használatával.</span><span class="sxs-lookup"><span data-stu-id="76d35-228">Tracking updates and revisions, using semantic versioning, along with the ability to roll back to a previous version.</span></span>
- <span data-ttu-id="76d35-229">Helm-diagramok elkerülhető a tartalomkiszolgálón információkat, például a címkék és a választók, sablonok használatával.</span><span class="sxs-lookup"><span data-stu-id="76d35-229">Helm charts use templates to avoid duplicating information, such as labels and selectors, across many files.</span></span>
- <span data-ttu-id="76d35-230">Helm diagramok közti függőségeket is kezelheti.</span><span class="sxs-lookup"><span data-stu-id="76d35-230">Helm can manage dependencies between charts.</span></span>
- <span data-ttu-id="76d35-231">Diagramok a Helm-adattárhoz, például az Azure Container Registry, tárolható, és a buildelési folyamat integrálva.</span><span class="sxs-lookup"><span data-stu-id="76d35-231">Charts can be stored in a Helm repository, such as Azure Container Registry, and integrated into the build pipeline.</span></span>

<span data-ttu-id="76d35-232">További információ a Container Registry használatával, egy Helm-adattárhoz: [használata Azure Container Registry vagy a Helm az alkalmazás diagramok](/azure/container-registry/container-registry-helm-repos).</span><span class="sxs-lookup"><span data-stu-id="76d35-232">For more information about using Container Registry as a Helm repository, see [Use Azure Container Registry as a Helm repository for your application charts](/azure/container-registry/container-registry-helm-repos).</span></span>

<span data-ttu-id="76d35-233">Egyetlen mikroszolgáltatást több Kubernetes konfigurációs fájlok is igénybe vehet.</span><span class="sxs-lookup"><span data-stu-id="76d35-233">A single microservice may involve multiple Kubernetes configuration files.</span></span> <span data-ttu-id="76d35-234">Szolgáltatás frissítése is jelenti azt, csak az alábbi fájlok frissítéséhez választók, a címkék és a címkéket.</span><span class="sxs-lookup"><span data-stu-id="76d35-234">Updating a service can mean touching all of these files to update selectors, labels, and image tags.</span></span> <span data-ttu-id="76d35-235">Helm kezeli ezeket az adatokat tartalmazó egyetlen csomag neve diagram, és lehetővé teszi, hogy egyszerűen frissítheti a YAML-fájlok változók használatával.</span><span class="sxs-lookup"><span data-stu-id="76d35-235">Helm treats these as a single package called a chart and allows you to easily update the YAML files by using variables.</span></span> <span data-ttu-id="76d35-236">Helm írhat paraméteres YAML-konfigurációs fájlok (a Go-sablonok alapján) sablon nyelvet használ.</span><span class="sxs-lookup"><span data-stu-id="76d35-236">Helm uses a template language (based on Go templates) to let you write parameterized YAML configuration files.</span></span>

<span data-ttu-id="76d35-237">Ha például a következő egy YAML-fájlt, amely meghatározza egy központi telepítést része:</span><span class="sxs-lookup"><span data-stu-id="76d35-237">For example, here's part of a YAML file that defines a deployment:</span></span>

```yaml
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: {{ include "package.fullname" . | replace "." "" }}
  labels:
    app.kubernetes.io/name: {{ include "package.name" . }}
    app.kubernetes.io/instance: {{ .Release.Name }}
  annotations:
    kubernetes.io/change-cause: {{ .Values.reason }}

...

  spec:
      containers:
      - name: &package-container_name fabrikam-package
        image: {{ .Values.dockerregistry }}/{{ .Values.image.repository }}:{{ .Values.image.tag }}
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        env:
        - name: LOG_LEVEL
          value: {{ .Values.log.level }}
```

<span data-ttu-id="76d35-238">&#11162;Tekintse meg a [forrásfájl](https://github.com/mspnp/microservices-reference-implementation/blob/master/charts/package/templates/package-deploy.yaml).</span><span class="sxs-lookup"><span data-stu-id="76d35-238">&#11162; See the [source file](https://github.com/mspnp/microservices-reference-implementation/blob/master/charts/package/templates/package-deploy.yaml).</span></span>

<span data-ttu-id="76d35-239">Láthatja, hogy a központi telepítés nevét, a címkék és a tároló spec összes használata sablon paramétereit, központi telepítéskor is tartalmaz.</span><span class="sxs-lookup"><span data-stu-id="76d35-239">You can see that the deployment name, labels, and container spec all use template parameters, which are provided at deployment time.</span></span> <span data-ttu-id="76d35-240">Ha például a parancssorból:</span><span class="sxs-lookup"><span data-stu-id="76d35-240">For example, from the command line:</span></span>

```bash
helm install $HELM_CHARTS/package/ \
     --set image.tag=0.1.0 \
     --set image.repository=package \
     --set dockerregistry=$ACR_SERVER \
     --namespace backend \
     --name package-v0.1.0
```

<span data-ttu-id="76d35-241">Bár a CI/CD-folyamat sikerült telepíteni a diagram közvetlenül a Kubernetes, javasoljuk, hogy egy diagram archív (.tgz fájl) létrehozása és leküldése a diagram például az Azure Container Registry egy Helm-adattárhoz.</span><span class="sxs-lookup"><span data-stu-id="76d35-241">Although your CI/CD pipeline could install a chart directly to Kubernetes, we recommend creating a chart archive (.tgz file) and pushing the chart to a Helm repository such as Azure Container Registry.</span></span> <span data-ttu-id="76d35-242">További információkért lásd: [csomag Docker-alapú alkalmazások az Azure-folyamatokban Helm-diagramok](/azure/devops/pipelines/languages/helm?view=azure-devops).</span><span class="sxs-lookup"><span data-stu-id="76d35-242">For more information, see [Package Docker-based apps in Helm charts in Azure Pipelines](/azure/devops/pipelines/languages/helm?view=azure-devops).</span></span>

<span data-ttu-id="76d35-243">Fontolja meg a saját névtere való üzembe helyezés a Helm és szerepköralapú hozzáférés-vezérlés (RBAC) használatával való korlátozása, hogy mely névtereket, helyezhet üzembe.</span><span class="sxs-lookup"><span data-stu-id="76d35-243">Consider deploying Helm to its own namespace and using role-based access control (RBAC) to restrict which namespaces it can deploy to.</span></span> <span data-ttu-id="76d35-244">További információkért lásd: [szerepköralapú hozzáférés-vezérlés](https://helm.sh/docs/using_helm/#helm-and-role-based-access-control) a Helm dokumentációjában.</span><span class="sxs-lookup"><span data-stu-id="76d35-244">For more information, see [Role-based Access Control](https://helm.sh/docs/using_helm/#helm-and-role-based-access-control) in the Helm documentation.</span></span>

### <a name="revisions"></a><span data-ttu-id="76d35-245">Változatok</span><span class="sxs-lookup"><span data-stu-id="76d35-245">Revisions</span></span>

<span data-ttu-id="76d35-246">Helm-diagramok mindig van egy verziószámot, amely kell használnia [Szemantikus verziószámozást](https://semver.org/).</span><span class="sxs-lookup"><span data-stu-id="76d35-246">Helm charts always have a version number, which must use [semantic versioning](https://semver.org/).</span></span> <span data-ttu-id="76d35-247">Egy diagram is lehet egy `appVersion`.</span><span class="sxs-lookup"><span data-stu-id="76d35-247">A chart can also have an `appVersion`.</span></span> <span data-ttu-id="76d35-248">Ez a mező nem kötelező, de kapcsolatos, a diagram verziója nem rendelkezik.</span><span class="sxs-lookup"><span data-stu-id="76d35-248">This field is optional, and doesn't have to be related to the chart version.</span></span> <span data-ttu-id="76d35-249">Néhány csapat érdemes alkalmazásverziónál külön a frissítések, a diagramokat.</span><span class="sxs-lookup"><span data-stu-id="76d35-249">Some teams might want to application versions separately from updates to the charts.</span></span> <span data-ttu-id="76d35-250">Azonban egy egyszerűbb módszert kell használni egy verziószámot, diagram és alkalmazás verziója között egy 1:1-es kapcsolat.</span><span class="sxs-lookup"><span data-stu-id="76d35-250">But a simpler approach is to use one version number, so there's a 1:1 relation between chart version and application version.</span></span> <span data-ttu-id="76d35-251">Ily módon tárolhatja kiszolgálónként kiadás egy diagram és egyszerű üzembe helyezése a kívánt kiadásban:</span><span class="sxs-lookup"><span data-stu-id="76d35-251">That way, you can store one chart per release and easily deploy the desired release:</span></span>

```bash
helm install <package-chart-name> --version <desiredVersion>
```

<span data-ttu-id="76d35-252">Egy másik jó gyakorlat, hogy a változás okának jegyzet a központi telepítési sablon:</span><span class="sxs-lookup"><span data-stu-id="76d35-252">Another good practice is to provide a change-cause annotation in the deployment template:</span></span>

```yaml
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: {{ include "delivery.fullname" . | replace "." "" }}
  labels:
     ...
  annotations:
    kubernetes.io/change-cause: {{ .Values.reason }}
```

<span data-ttu-id="76d35-253">Ez lehetővé teszi, hogy megtekintse az egyes változatoknál megjelentek a változás okának mező használatával a `kubectl rollout history` parancsot.</span><span class="sxs-lookup"><span data-stu-id="76d35-253">This lets you view the change-cause field for each revision, using the `kubectl rollout history` command.</span></span> <span data-ttu-id="76d35-254">Az előző példában a változás okának nyújtja, mint egy Helm-diagram paraméter.</span><span class="sxs-lookup"><span data-stu-id="76d35-254">In the previous example, the change-cause is provided as a Helm chart parameter.</span></span>

```bash
$ kubectl rollout history deployments/delivery-v010 -n backend
deployment.extensions/delivery-v010
REVISION  CHANGE-CAUSE
1         Initial deployment
```

<span data-ttu-id="76d35-255">Is használhatja a `helm list` paranccsal tekintheti meg a változat-nyilvántartása:</span><span class="sxs-lookup"><span data-stu-id="76d35-255">You can also use the `helm list` command to view the revision history:</span></span>

```bash
~$ helm list
NAME            REVISION    UPDATED                     STATUS        CHART            APP VERSION     NAMESPACE
delivery-v0.1.0 1           Sun Apr  7 00:25:30 2019    DEPLOYED      delivery-v0.1.0  v0.1.0          backend
```

> [!TIP]
> <span data-ttu-id="76d35-256">Használja a `--history-max` jelzőt a Helm inicializálásakor.</span><span class="sxs-lookup"><span data-stu-id="76d35-256">Use the `--history-max` flag when initializing Helm.</span></span> <span data-ttu-id="76d35-257">Ez a beállítás korlátozza az alkalmazáson, amely a tiller valóban menti az előzményeket.</span><span class="sxs-lookup"><span data-stu-id="76d35-257">This setting limits the number of revisions that Tiller saves in its history.</span></span> <span data-ttu-id="76d35-258">A tiller valóban változat-nyilvántartása configmaps tárolja.</span><span class="sxs-lookup"><span data-stu-id="76d35-258">Tiller stores revision history in configmaps.</span></span> <span data-ttu-id="76d35-259">Frissítések gyakran használt ad ki, ha a configmaps ki is kapcsolhatja, ha az előzmények mérete korlátozza.</span><span class="sxs-lookup"><span data-stu-id="76d35-259">If you're releasing updates frequently, the configmaps can grow very large unless you limit the history size.</span></span>

## <a name="azure-devops-pipeline"></a><span data-ttu-id="76d35-260">Azure DevOps Pipeline</span><span class="sxs-lookup"><span data-stu-id="76d35-260">Azure DevOps Pipeline</span></span>

<span data-ttu-id="76d35-261">Az Azure-folyamatok, a folyamatok vannak osztva, *folyamatok alakíthatók ki* és *folyamatok felszabadítása*.</span><span class="sxs-lookup"><span data-stu-id="76d35-261">In Azure Pipelines, pipelines are divided into *build pipelines* and *release pipelines*.</span></span> <span data-ttu-id="76d35-262">A létrehozási folyamat a CI folyamat fut, és létrehozza a build-összetevőket.</span><span class="sxs-lookup"><span data-stu-id="76d35-262">The build pipeline runs the CI process and creates build artifacts.</span></span> <span data-ttu-id="76d35-263">A mikroszolgáltatási architektúra a Kubernetes ezeket az összetevőket a tárolórendszerképre és a Helm-diagramok, amelyek meghatározzák az egyes mikroszolgáltatások.</span><span class="sxs-lookup"><span data-stu-id="76d35-263">For a microservices architecture on Kubernetes, these artifacts are the container images and Helm charts that define each microservice.</span></span> <span data-ttu-id="76d35-264">A kibocsátási folyamat fut a CD-folyamat, amely üzembe helyezi a mikroszolgáltatás egy fürtbe.</span><span class="sxs-lookup"><span data-stu-id="76d35-264">The release pipeline runs that CD process that deploys a microservice into a cluster.</span></span>

<span data-ttu-id="76d35-265">A CI folyamat ebben a cikkben korábban leírtak alapján a buildelési folyamat tartalmazhatnak a következő feladatokat:</span><span class="sxs-lookup"><span data-stu-id="76d35-265">Based on the CI flow described earlier in this article, a build pipeline might consist of the following tasks:</span></span>

1. <span data-ttu-id="76d35-266">A teszt futtatója tárolót hozhat létre.</span><span class="sxs-lookup"><span data-stu-id="76d35-266">Build the test runner container.</span></span>

    ```yaml
    - task: Docker@1
      inputs:
        azureSubscriptionEndpoint: $(AzureSubscription)
        azureContainerRegistry: $(AzureContainerRegistry)
        arguments: '--pull --target testrunner'
        dockerFile: $(System.DefaultWorkingDirectory)/$(dockerFileName)
        imageName: '$(imageName)-test'
    ```

1. <span data-ttu-id="76d35-267">Futtassa a tesztek futtatásához a teszt futtatója tároló docker meghívásával.</span><span class="sxs-lookup"><span data-stu-id="76d35-267">Run the tests, by invoking docker run against the test runner container.</span></span>

    ```yaml
    - task: Docker@1
      inputs:
        azureSubscriptionEndpoint: $(AzureSubscription)
        azureContainerRegistry: $(AzureContainerRegistry)
        command: 'run'
        containerName: testrunner
        volumes: '$(System.DefaultWorkingDirectory)/TestResults:/app/tests/TestResults'
        imageName: '$(imageName)-test'
        runInBackground: false
    ```

1. <span data-ttu-id="76d35-268">Tegye közzé a vizsgálati eredmények.</span><span class="sxs-lookup"><span data-stu-id="76d35-268">Publish the test results.</span></span> <span data-ttu-id="76d35-269">Lásd: [integrálása a build és a feladatok](/azure/devops/pipelines/languages/docker?view=azure-devops&tabs=yaml#integrate-build-and-test-tasks).</span><span class="sxs-lookup"><span data-stu-id="76d35-269">See [Integrate build and test tasks](/azure/devops/pipelines/languages/docker?view=azure-devops&tabs=yaml#integrate-build-and-test-tasks).</span></span>

    ```yaml
    - task: PublishTestResults@2
      inputs:
        testResultsFormat: 'VSTest'
        testResultsFiles: 'TestResults/*.trx'
        searchFolder: '$(System.DefaultWorkingDirectory)'
        publishRunAttachments: true
    ```

1. <span data-ttu-id="76d35-270">A modul tárolót hozhat létre.</span><span class="sxs-lookup"><span data-stu-id="76d35-270">Build the runtime container.</span></span>

    ```yaml
    - task: Docker@1
      inputs:
        azureSubscriptionEndpoint: $(AzureSubscription)
        azureContainerRegistry: $(AzureContainerRegistry)
        dockerFile: $(System.DefaultWorkingDirectory)/$(dockerFileName)
        includeLatestTag: false
        imageName: '$(imageName)'
    ```

1. <span data-ttu-id="76d35-271">Küldje le a tároló Azure Container Registry (vagy egyéb tároló-beállításjegyzék).</span><span class="sxs-lookup"><span data-stu-id="76d35-271">Push to the container to Azure Container Registry (or other container registry).</span></span>

    ```yaml
    - task: Docker@1
      inputs:
        azureSubscriptionEndpoint: $(AzureSubscription)
        azureContainerRegistry: $(AzureContainerRegistry)
        command: 'Push an image'
        imageName: '$(imageName)'
        includeSourceTags: false
    ```

1. <span data-ttu-id="76d35-272">A Helm-diagramot a csomag.</span><span class="sxs-lookup"><span data-stu-id="76d35-272">Package the Helm chart.</span></span>

    ```yaml
    - task: HelmDeploy@0
      inputs:
        command: package
        chartPath: $(chartPath)
        chartVersion: $(Build.SourceBranchName)
        arguments: '--app-version $(Build.SourceBranchName)'
    ```

1. <span data-ttu-id="76d35-273">A Helm-csomag leküldése az Azure Container Registry (vagy más Helm-adattárhoz).</span><span class="sxs-lookup"><span data-stu-id="76d35-273">Push the Helm package to Azure Container Registry (or other Helm repository).</span></span>

    ```yaml
    task: AzureCLI@1
      inputs:
        azureSubscription: $(AzureSubscription)
        scriptLocation: inlineScript
        inlineScript: |
        az acr helm push $(System.ArtifactsDirectory)/$(repositoryName)-$(Build.SourceBranchName).tgz --name $(AzureContainerRegistry);
    ```

<span data-ttu-id="76d35-274">&#11162;Tekintse meg a [forrásfájl](https://github.com/mspnp/microservices-reference-implementation/blob/master/src/shipping/delivery/azure-pipelines-ci.yml).</span><span class="sxs-lookup"><span data-stu-id="76d35-274">&#11162; See the [source file](https://github.com/mspnp/microservices-reference-implementation/blob/master/src/shipping/delivery/azure-pipelines-ci.yml).</span></span>

<span data-ttu-id="76d35-275">A CI folyamat kimenete egy éles használatra kész tárolórendszerképet, és a egy frissített Helm-diagramot a mikroszolgáltatási.</span><span class="sxs-lookup"><span data-stu-id="76d35-275">The output from the CI pipeline is a production-ready container image and an updated Helm chart for the microservice.</span></span> <span data-ttu-id="76d35-276">Ezen a ponton a kiadási folyamathoz átvehetik.</span><span class="sxs-lookup"><span data-stu-id="76d35-276">At this point, the release pipeline can take over.</span></span> <span data-ttu-id="76d35-277">Ez hajtja végre az alábbi lépéseket:</span><span class="sxs-lookup"><span data-stu-id="76d35-277">It performs the following steps:</span></span>

- <span data-ttu-id="76d35-278">Dev/QA és átmeneti környezetek üzembe helyezése.</span><span class="sxs-lookup"><span data-stu-id="76d35-278">Deploy to dev/QA/staging environments.</span></span>
- <span data-ttu-id="76d35-279">Várja meg, jóváhagyhatja vagy elutasíthatja az üzembe helyezés jóváhagyónak.</span><span class="sxs-lookup"><span data-stu-id="76d35-279">Wait for an approver to approve or reject the deployment.</span></span>
- <span data-ttu-id="76d35-280">A tároló rendszerképét kiadás újracímkézése</span><span class="sxs-lookup"><span data-stu-id="76d35-280">Retag the container image for release</span></span>
- <span data-ttu-id="76d35-281">A kiadási címke leküldése a tároló-beállításjegyzékbe.</span><span class="sxs-lookup"><span data-stu-id="76d35-281">Push the release tag to the container registry.</span></span>
- <span data-ttu-id="76d35-282">Frissítse a Helm-diagramot a termelési fürtben.</span><span class="sxs-lookup"><span data-stu-id="76d35-282">Upgrade the Helm chart in the production cluster.</span></span>

<span data-ttu-id="76d35-283">Kiadási folyamatok létrehozásával kapcsolatos további információkért lásd: [kiadási folyamatokat, vázlat kiadások és kiadáslehetőségek](/azure/devops/pipelines/release/?view=azure-devops).</span><span class="sxs-lookup"><span data-stu-id="76d35-283">For more information about creating a release pipeline, see [Release pipelines, draft releases, and release options](/azure/devops/pipelines/release/?view=azure-devops).</span></span>

<span data-ttu-id="76d35-284">Az alábbi ábrán látható az ebben a cikkben ismertetett teljes körű CI/CD folyamatot:</span><span class="sxs-lookup"><span data-stu-id="76d35-284">The following diagram shows the end-to-end CI/CD process described in this article:</span></span>

![CD-re/CD-folyamat](./images/aks-cicd-flow.png)

## <a name="next-steps"></a><span data-ttu-id="76d35-286">További lépések</span><span class="sxs-lookup"><span data-stu-id="76d35-286">Next steps</span></span>

<span data-ttu-id="76d35-287">Ez a cikk egy referenciaimplementációt, amely az annak alapjául szolgáló [GitHub][ri].</span><span class="sxs-lookup"><span data-stu-id="76d35-287">This article was based on a reference implementation that you can find on [GitHub][ri].</span></span>

[ri]: https://github.com/mspnp/microservices-reference-implementation