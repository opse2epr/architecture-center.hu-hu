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

# <a name="building-a-cicd-pipeline-for-microservices-on-kubernetes"></a>A mikroszolgáltatásokat CI/CD-folyamat felépítésével bajlódnia a kubernetes használatával

Ez a mikroszolgáltatási architektúra megbízható CI/CD-folyamat létrehozásához kihívást jelenthet. Az egyes csapatok felszabadítása nélkül más csapatokkal megszakítása vagy az alkalmazás egésze destabilizing szolgáltatások gyorsan és megbízhatóan, képesnek kell lennie.

Ez a cikk ismerteti egy példa a CI/CD folyamatot a mikroszolgáltatásokat helyez üzembe az Azure Kubernetes Service (AKS). Minden csoport és a projekt nem egyezik, így nem ez a cikk, hard-and-fast szabályai szerint. Ehelyett azt azt jelentette a saját CI/CD-folyamat tervezéséhez kiindulási pontként.

Az itt ismertetett példa folyamat létrehoztunk egy mikroszolgáltatás-alapú referenciaimplementációt, a Drone Delivery alkalmazást, amely a nevű [GitHub][ri]. Az alkalmazás forgatókönyv [Itt](./design/index.md#reference-implementation).

A folyamat célja is a következőképpen épül fel:

- Csapatok létrehozhat és a szolgáltatások egymástól függetlenül üzembe helyezéséhez.
- Kód módosítása, hogy a CI folyamat a rendszer automatikusan telepíti az utolsóig üzemi környezetben.
- Minőségi kapuk a rendszer érvényesíti a folyamat minden szakaszában.
- A szolgáltatás egy új verziója is telepíthető az előző verzió párhuzamosan lesz.

A további háttér-információkért lásd: [CI/CD-mikroszolgáltatás-architektúrákat](./ci-cd.md).

## <a name="assumptions"></a>Előfeltételek

Ebben a példában alkalmazásában az alábbiakban néhány feltételezéseket a fejlesztői csapat és a kód alap:

- A kódtár egy monorepo mappák mikroszolgáltatás szerint vannak rendezve.
- A csapat stratégiáját az elágazási alapján [trönk-alapú fejlesztési](https://trunkbaseddevelopment.com/).
- A csapata által használt [kiadási ágakat](/azure/devops/repos/git/git-branching-guidance?view=azure-devops#manage-releases) kiadások kezeléséhez. Mindegyik mikroszolgáltatás külön kiadások jön létre.
- A CI/CD-folyamat használja [Azure folyamatok](/azure/devops/pipelines/?view=azure-devops) hozhat létre, tesztelheti, és a mikroszolgáltatások üzembe az aks-ben.
- Mindegyik mikroszolgáltatás a tárolórendszerképeket vannak tárolva [Azure Container Registry](/azure/container-registry/).
- A csapat Helm-diagramok használja az egyes mikroszolgáltatások csomagolását.

Ilyen Előfeltevések meghajtó a CI/CD-folyamat az adott részletek nagy része. Azonban az alapszintű megközelítés az itt leírtak szerint más folyamatokat, eszközöket és szolgáltatásokat, például a Jenkins vagy a Docker Hub módosítani kell.

## <a name="validation-builds"></a>Érvényesítési buildek

Tegyük fel, hogy a fejlesztő neve a kézbesítési szolgáltatás mikroszolgáltatások dolgozik. Egy új szolgáltatás fejlesztésekor a fejlesztői egy funkciót a főágban kód ellenőrzi. Szabályok szerint funkció ágak elnevezése `feature/*`.

![CI/CD-munkafolyamat](./images/aks-cicd-1.png)

A build-definíciós fájl tartalmaz egy eseményindítót, amely szűri a kiadásiág-nevet és a forrás elérési útja:

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

&#11162;Tekintse meg a [forrásfájl](https://github.com/mspnp/microservices-reference-implementation/blob/master/src/shipping/delivery/azure-pipelines-validation.yml).

Ezt a módszert használja, minden egyes csapat a saját build folyamat lehet. Csak olyan kódot, amely be van jelölve, a `/src/shipping/delivery` mappa aktiválnak egy buildet, a Tartalomkézbesítési szolgáltatás. A CI-build, amely megfelel a szűrő egyik ágára való leküldés a véglegesítéseket aktivál. Ezen a ponton a munkafolyamatban a CI-build néhány csak minimális mennyiségű kódra ellenőrzés fut:

1. A kód felépítéséhez.
1. Futtatni az egységteszteket.

A célja, hogy a fejlesztési idő rövid, így a fejlesztő gyors visszajelzés. Ha a szolgáltatás készen áll a egyesítése a mesterrel, a fejlesztői megnyitása új Ez elindít egy másik CI build néhány további ellenőrzést végző:

1. A kód felépítéséhez.
1. Futtatni az egységteszteket.
1. A futtatókörnyezet tárolólemezkép létrehozásához.
1. Futtassa a vizsgálatot a biztonsági rések a rendszerképen.

![CI/CD-munkafolyamat](./images/aks-cicd-2.png)

> [!NOTE]
> Az Azure DevOps-Adattárakkal, definiálhat [házirendek](/azure/devops/repos/git/branch-policies) ágak védelme érdekében. Például a szabályzat szükségük CI a build sikeres létrehozása és a egy jóváhagyás a jóváhagyó egyesítése a mesterrel.

## <a name="full-cicd-build"></a>Teljes CI/CD-build

Később a csapat a kézbesítési szolgáltatás új verziójának telepítése készen áll. A kiadáskezelő elnevezési mintázatot a főágból hoz létre egy ágat: `release/<microservice name>/<semver>`. Például: `release/delivery/v1.0.2`.

![CI/CD-munkafolyamat](./images/aks-cicd-3.png)

Ez az ág létrehozása fut, plusz az előző lépések mindegyikét teljes CI build aktiválása:

1. A tárolórendszerkép leküldése az Azure Container Registrybe. A kép a kiadásiág-nevet származó verziószámmal van megjelölve.
2. Futtatás `helm package` csomagolása a Helm-diagramot a szolgáltatáshoz. A diagram is van megjelölve a verziószámmal.
3. A Helm-csomag leküldése a Tárolójegyzékbe.

Ha a build sikeres lesz, elindít egy üzembe helyezés (CD) folyamat használatával egy Azure-folyamatok [kibocsátási folyamatok](/azure/devops/pipelines/release/what-is-release-management). Ez a folyamat rendelkezik az alábbi lépéseket:

1. A Helm-diagram üzembe helyezése QA környezetben.
1. Jóváhagyó jelentkezik, mielőtt a csomagot az éles környezetbe helyezi át. Lásd: [kiadási központi telepítés ellenőrzési jóváhagyások használatával](/azure/devops/pipelines/release/approvals/approvals).
1. A Docker-rendszerképet az Azure Container Registry a termelési névtér újracímkézése. Például, ha az aktuális címke `myrepo.azurecr.io/delivery:v1.0.2`, az éles címke `myrepo.azurecr.io/prod/delivery:v1.0.2`.
1. Helyezze üzembe a Helm-diagram az éles környezetben.

Egy monorepo, még akkor is, ezeket a feladatokat korlátozhatja az egyes mikroszolgáltatások úgy, hogy a teams telepítheti a nagy sebességű. A folyamat néhány manuális lépésből áll: Lekéréses kérelmek jóváhagyása, a kiadási ágak létrehozása és az éles fürt üzemelő jóváhagyása. Ezen lépések végrehajtása manuális házirend &mdash; azok sikerült automatizált, ha a szervezet részesíti előnyben.

## <a name="isolation-of-environments"></a>A környezetek elkülönítése

Kell több környezetet, ahol a szolgáltatások, beleértve a fejlesztési, füst tesztelése, integráció tesztelése, terheléses tesztelés díjaival, és végül éles környezetben telepít. Ezekben a környezetekben kell bizonyos fokú elkülönítés. A Kubernetes választhat, hogy fizikai elkülönítése és logikai elkülönítése között. Fizikai elkülönítése azt jelenti, hogy különálló fürt üzembe helyezése. Logikai elkülönítéssel használ, a névterek és a házirendeket, a fentebb leírt módon.

Azt javasoljuk, hogy hozzon létre egy dedikált fürt és a fejlesztési-tesztelési környezetek esetében külön fürtben. Logikai elkülönítés használatával a fejlesztési-tesztelési fürtön belüli környezeteket. A fejlesztési-tesztelési fürtre telepített szolgáltatások soha nem üzleti adatokat tartalmazó adattárakra hozzáféréssel kell rendelkeznie.

## <a name="build-process"></a>Folyamat létrehozása

Ha lehetséges, a csomag a build-folyamatoknak egy Docker-tárolóba. Amely lehetővé teszi, hogy a kód összetevők, Docker, anélkül, hogy a build-környezet konfigurálása az összes build használatával hozhat létre. Tárolóalapú folyamatának megkönnyíti a horizontális felskálázás a CI folyamat új buildelési ügynökök hozzáadásával. Emellett minden fejlesztő a csapat hozhat létre a kódot a build-tároló futtatásával.

Többlépcsős buildek a Docker használatával meghatározhatja a build környezet és a futtatókörnyezet rendszerkép egy docker-fájlban. Ha például itt látható, amely egy ASP.NET Core-alkalmazást hoz létre egy docker-fájlban:

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

&#11162;Tekintse meg a [forrásfájl](https://github.com/mspnp/microservices-reference-implementation/blob/master/src/shipping/workflow/Dockerfile).

A docker-fájl több összeállítási fázisok határozza meg. Figyelje meg, hogy a szakasz nevű `base` használja az ASP.NET Core-futtatókörnyezet, az nevű szakasz során `build` a teljes ASP.NET Core SDK-t használja. A `build` szakasz segítségével hozhatók létre az ASP.NET Core-projektet. De a végső futásidejű tároló hitelesítésszolgáltatókból `base`, amely csak a modult tartalmaz, és jelentősen kisebb, mint a teljes SDK lemezkép.

### <a name="building-a-test-runner"></a>A tesztfuttató készítése

Egy másik jó gyakorlat, hogy a tárolóban futtatni az egységteszteket. Ha például a következő része, amely egy teszt futtatója összeállít egy Docker-fájlt:

```
FROM build AS testrunner
WORKDIR /src/tests

COPY Fabrikam.Workflow.Service.Tests/*.csproj .
RUN dotnet restore Fabrikam.Workflow.Service.Tests.csproj

COPY Fabrikam.Workflow.Service.Tests/. .
ENTRYPOINT ["dotnet", "test", "--logger:trx"]
```

A fejlesztők a Docker-fájl használatával helyileg futtasson teszteket:

```bash
docker build . -t delivery-test:1 --target=testrunner
docker run -p 8080:8080 delivery-test:1
```

A CI folyamat részeként ellenőrzési lépést is futhat a tesztek.

Vegye figyelembe, hogy ezt a fájlt használja a Docker `ENTRYPOINT` parancsot futtathatja a teszteket, nem a Docker `RUN` parancsot.

- Ha használja a `RUN` parancsot, a tesztek futtatását minden alkalommal, amikor a rendszerkép létrehozásához. Használatával `ENTRYPOINT`, hogy a teszt nem vehetnek részt. Csak ha explicit módon futnak a `testrunner` szakaszban.
- A sikertelen teszt nem okozza a Docker `build` parancs sikertelen lesz. Ily módon, hogy megkülönböztethesse tárolót hozhat létre a sikertelen teszt hibáinak.
- Vizsgálati eredmények menthető egy csatlakoztatott kötetre.

### <a name="container-best-practices"></a>Tároló ajánlott eljárások

Az alábbiakban néhány egyéb ajánlott eljárások a tárolók:

- Adja meg a szervezeti szintű szabályai tároló címkék, verziószámozása és a fürthöz (podok, szolgáltatások és így tovább) üzembe helyezett erőforrások elnevezési szabályainak. Amely megkönnyítheti a üzembe helyezési problémák diagnosztizálásához.

- A fejlesztési és tesztelési ciklus során a CI/CD-folyamat számos tárolórendszerképek fog létrehozni. Csak a kiadásban néhány ilyen rendszerképpel deduplikációra, majd csak néhányat ezen kiadás jelöltek fog első előléptetése éles környezetbe. Rendelkezik egy egyértelmű verziókezelési stratégiát, úgy, hogy már ismeri, mely képek az éles környezetben jelenleg telepítve vannak, és vissza lehet vonni egy korábbi verziójának szükség esetén.

- Mindig üzembe helyezése nem adott tároló verzió címkéket, `latest`.

- Használat [névterek](/azure/container-registry/container-registry-best-practices#repository-namespaces) az Azure Container Registry rendszerképek éles környezetben, amely továbbra is tesztelik rendszerképekből jóváhagyott elkülönítésére. Ne helyezze át a production névtérből kép, mindaddig, amíg készen áll az éles üzembe helyezés. Ha ez az eljárás kombinálva a tárolórendszerképek Szemantikus verziószámozást, csökkentheti az esélye, hogy véletlenül üzembe helyezése egy olyanra, amely a kiadás nem lett jóváhagyva.

- Kövesse a legalacsonyabb jogosultsági szint elvének a futó tárolók replikaadatok felhasználóként. A Kubernetes, létrehozhat egy pod biztonsági szabályzatot, amely megakadályozza, hogy a tárolók nevében *legfelső szintű*. Lásd: [megakadályozza, hogy a Podok legfelső szintű jogosultságokkal fut](https://docs.bitnami.com/kubernetes/how-to/secure-kubernetes-cluster-psp/).

## <a name="helm-charts"></a>Helm-diagramok

Fontolja meg a Helm használatával létrehozásának és üzembe helyezésének a szolgáltatások kezeléséhez. Íme néhány a Helm, amelyek segítenek a CI/CD-funkcióit:

- Gyakran egyetlen mikroszolgáltatást határozza meg több Kubernetes-objektumokat. Helm lehetővé teszi, hogy ezek az objektumok egy Helm-diagram szabálykészletnek.
- A diagram a Helm egyetlen paranccsal ahelyett, hogy a kubectl-parancsokat is telepíthető.
- Diagramok használata explicit módon verziószámmal. Helm használatával egy verziója engedje, kiadásokban megtekintése és visszaállítása egy korábbi verzióra. Követési frissítések és a módosításokat, Szemantikus verziószámozást elkészíthetők visszaállítása egy korábbi verziójának használatával.
- Helm-diagramok elkerülhető a tartalomkiszolgálón információkat, például a címkék és a választók, sablonok használatával.
- Helm diagramok közti függőségeket is kezelheti.
- Diagramok a Helm-adattárhoz, például az Azure Container Registry, tárolható, és a buildelési folyamat integrálva.

További információ a Container Registry használatával, egy Helm-adattárhoz: [használata Azure Container Registry vagy a Helm az alkalmazás diagramok](/azure/container-registry/container-registry-helm-repos).

Egyetlen mikroszolgáltatást több Kubernetes konfigurációs fájlok is igénybe vehet. Szolgáltatás frissítése is jelenti azt, csak az alábbi fájlok frissítéséhez választók, a címkék és a címkéket. Helm kezeli ezeket az adatokat tartalmazó egyetlen csomag neve diagram, és lehetővé teszi, hogy egyszerűen frissítheti a YAML-fájlok változók használatával. Helm írhat paraméteres YAML-konfigurációs fájlok (a Go-sablonok alapján) sablon nyelvet használ.

Ha például a következő egy YAML-fájlt, amely meghatározza egy központi telepítést része:

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

&#11162;Tekintse meg a [forrásfájl](https://github.com/mspnp/microservices-reference-implementation/blob/master/charts/package/templates/package-deploy.yaml).

Láthatja, hogy a központi telepítés nevét, a címkék és a tároló spec összes használata sablon paramétereit, központi telepítéskor is tartalmaz. Ha például a parancssorból:

```bash
helm install $HELM_CHARTS/package/ \
     --set image.tag=0.1.0 \
     --set image.repository=package \
     --set dockerregistry=$ACR_SERVER \
     --namespace backend \
     --name package-v0.1.0
```

Bár a CI/CD-folyamat sikerült telepíteni a diagram közvetlenül a Kubernetes, javasoljuk, hogy egy diagram archív (.tgz fájl) létrehozása és leküldése a diagram például az Azure Container Registry egy Helm-adattárhoz. További információkért lásd: [csomag Docker-alapú alkalmazások az Azure-folyamatokban Helm-diagramok](/azure/devops/pipelines/languages/helm?view=azure-devops).

Fontolja meg a saját névtere való üzembe helyezés a Helm és szerepköralapú hozzáférés-vezérlés (RBAC) használatával való korlátozása, hogy mely névtereket, helyezhet üzembe. További információkért lásd: [szerepköralapú hozzáférés-vezérlés](https://helm.sh/docs/using_helm/#helm-and-role-based-access-control) a Helm dokumentációjában.

### <a name="revisions"></a>Változatok

Helm-diagramok mindig van egy verziószámot, amely kell használnia [Szemantikus verziószámozást](https://semver.org/). Egy diagram is lehet egy `appVersion`. Ez a mező nem kötelező, de kapcsolatos, a diagram verziója nem rendelkezik. Néhány csapat érdemes alkalmazásverziónál külön a frissítések, a diagramokat. Azonban egy egyszerűbb módszert kell használni egy verziószámot, diagram és alkalmazás verziója között egy 1:1-es kapcsolat. Ily módon tárolhatja kiszolgálónként kiadás egy diagram és egyszerű üzembe helyezése a kívánt kiadásban:

```bash
helm install <package-chart-name> --version <desiredVersion>
```

Egy másik jó gyakorlat, hogy a változás okának jegyzet a központi telepítési sablon:

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

Ez lehetővé teszi, hogy megtekintse az egyes változatoknál megjelentek a változás okának mező használatával a `kubectl rollout history` parancsot. Az előző példában a változás okának nyújtja, mint egy Helm-diagram paraméter.

```bash
$ kubectl rollout history deployments/delivery-v010 -n backend
deployment.extensions/delivery-v010
REVISION  CHANGE-CAUSE
1         Initial deployment
```

Is használhatja a `helm list` paranccsal tekintheti meg a változat-nyilvántartása:

```bash
~$ helm list
NAME            REVISION    UPDATED                     STATUS        CHART            APP VERSION     NAMESPACE
delivery-v0.1.0 1           Sun Apr  7 00:25:30 2019    DEPLOYED      delivery-v0.1.0  v0.1.0          backend
```

> [!TIP]
> Használja a `--history-max` jelzőt a Helm inicializálásakor. Ez a beállítás korlátozza az alkalmazáson, amely a tiller valóban menti az előzményeket. A tiller valóban változat-nyilvántartása configmaps tárolja. Frissítések gyakran használt ad ki, ha a configmaps ki is kapcsolhatja, ha az előzmények mérete korlátozza.

## <a name="azure-devops-pipeline"></a>Azure DevOps Pipeline

Az Azure-folyamatok, a folyamatok vannak osztva, *folyamatok alakíthatók ki* és *folyamatok felszabadítása*. A létrehozási folyamat a CI folyamat fut, és létrehozza a build-összetevőket. A mikroszolgáltatási architektúra a Kubernetes ezeket az összetevőket a tárolórendszerképre és a Helm-diagramok, amelyek meghatározzák az egyes mikroszolgáltatások. A kibocsátási folyamat fut a CD-folyamat, amely üzembe helyezi a mikroszolgáltatás egy fürtbe.

A CI folyamat ebben a cikkben korábban leírtak alapján a buildelési folyamat tartalmazhatnak a következő feladatokat:

1. A teszt futtatója tárolót hozhat létre.

    ```yaml
    - task: Docker@1
      inputs:
        azureSubscriptionEndpoint: $(AzureSubscription)
        azureContainerRegistry: $(AzureContainerRegistry)
        arguments: '--pull --target testrunner'
        dockerFile: $(System.DefaultWorkingDirectory)/$(dockerFileName)
        imageName: '$(imageName)-test'
    ```

1. Futtassa a tesztek futtatásához a teszt futtatója tároló docker meghívásával.

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

1. Tegye közzé a vizsgálati eredmények. Lásd: [integrálása a build és a feladatok](/azure/devops/pipelines/languages/docker?view=azure-devops&tabs=yaml#integrate-build-and-test-tasks).

    ```yaml
    - task: PublishTestResults@2
      inputs:
        testResultsFormat: 'VSTest'
        testResultsFiles: 'TestResults/*.trx'
        searchFolder: '$(System.DefaultWorkingDirectory)'
        publishRunAttachments: true
    ```

1. A modul tárolót hozhat létre.

    ```yaml
    - task: Docker@1
      inputs:
        azureSubscriptionEndpoint: $(AzureSubscription)
        azureContainerRegistry: $(AzureContainerRegistry)
        dockerFile: $(System.DefaultWorkingDirectory)/$(dockerFileName)
        includeLatestTag: false
        imageName: '$(imageName)'
    ```

1. Küldje le a tároló Azure Container Registry (vagy egyéb tároló-beállításjegyzék).

    ```yaml
    - task: Docker@1
      inputs:
        azureSubscriptionEndpoint: $(AzureSubscription)
        azureContainerRegistry: $(AzureContainerRegistry)
        command: 'Push an image'
        imageName: '$(imageName)'
        includeSourceTags: false
    ```

1. A Helm-diagramot a csomag.

    ```yaml
    - task: HelmDeploy@0
      inputs:
        command: package
        chartPath: $(chartPath)
        chartVersion: $(Build.SourceBranchName)
        arguments: '--app-version $(Build.SourceBranchName)'
    ```

1. A Helm-csomag leküldése az Azure Container Registry (vagy más Helm-adattárhoz).

    ```yaml
    task: AzureCLI@1
      inputs:
        azureSubscription: $(AzureSubscription)
        scriptLocation: inlineScript
        inlineScript: |
        az acr helm push $(System.ArtifactsDirectory)/$(repositoryName)-$(Build.SourceBranchName).tgz --name $(AzureContainerRegistry);
    ```

&#11162;Tekintse meg a [forrásfájl](https://github.com/mspnp/microservices-reference-implementation/blob/master/src/shipping/delivery/azure-pipelines-ci.yml).

A CI folyamat kimenete egy éles használatra kész tárolórendszerképet, és a egy frissített Helm-diagramot a mikroszolgáltatási. Ezen a ponton a kiadási folyamathoz átvehetik. Ez hajtja végre az alábbi lépéseket:

- Dev/QA és átmeneti környezetek üzembe helyezése.
- Várja meg, jóváhagyhatja vagy elutasíthatja az üzembe helyezés jóváhagyónak.
- A tároló rendszerképét kiadás újracímkézése
- A kiadási címke leküldése a tároló-beállításjegyzékbe.
- Frissítse a Helm-diagramot a termelési fürtben.

Kiadási folyamatok létrehozásával kapcsolatos további információkért lásd: [kiadási folyamatokat, vázlat kiadások és kiadáslehetőségek](/azure/devops/pipelines/release/?view=azure-devops).

Az alábbi ábrán látható az ebben a cikkben ismertetett teljes körű CI/CD folyamatot:

![CD-re/CD-folyamat](./images/aks-cicd-flow.png)

## <a name="next-steps"></a>További lépések

Ez a cikk egy referenciaimplementációt, amely az annak alapjául szolgáló [GitHub][ri].

[ri]: https://github.com/mspnp/microservices-reference-implementation