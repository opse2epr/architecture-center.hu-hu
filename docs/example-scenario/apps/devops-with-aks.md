---
title: CI/CD-folyamat a tárolóalapú számítási feladatokhoz
description: Bevált forgatókönyv egy DevOps-folyamattal a Jenkins, a Azure Container Registry, az Azure Kubernetes Service, a Cosmos DB és a Grafana használó Node.js-webalkalmazások készítéséhez.
author: iainfoulds
ms.date: 07/05/2018
ms.openlocfilehash: dceb4ad3c34ec43a54d802772f5817cacdd3929c
ms.sourcegitcommit: 8b5fc0d0d735793b87677610b747f54301dcb014
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 07/29/2018
ms.locfileid: "39334215"
---
# <a name="cicd-pipeline-for-container-based-workloads"></a>CI/CD-folyamat a tárolóalapú számítási feladatokhoz

Ebben a példaforgatókönyvben olyan vállalatok, amelyek fejlesztése korszerűsítheti a tárolók és a DevOps-munkafolyamatokba használatával szeretne alkalmazható. Ebben a forgatókönyvben egy Node.js-webalkalmazás összeállítása és a Jenkins által üzembe helyezett egy Azure Container Registry és Azure Kubernetes Service-ben. Globálisan elosztott adatbázis-rétegből, az Azure Cosmos DB-hez használható. Figyelheti, és az alkalmazás teljesítményproblémáinak elhárítása, az Azure Monitor integrálható a Grafana példány és az irányítópult.

Példaforgatókönyvek alkalmazás például egy automatizált fejlesztési környezetet biztosít, új kódot véglegesítéseket ellenőrzése és hogyan lehet továbbítani rá új üzembe helyezésekhez, átmeneti és éles környezetekbe. Hagyományosan vállalkozások kellett manuálisan hozhat létre és fordítsa le az alkalmazásokat és frissítéseket, és egy nagy, monolitikus kódot alap kezelhet. A folyamatos integrációs (CI) és a folyamatos készregyártás (CD) használó alkalmazások fejlesztését modern megközelítését gyorsabban hozhat létre programot, teszteléséhez, és a szolgáltatások üzembe helyezéséhez. Ez a modern módszer lehetővé teszi, hogy kiadási alkalmazásokat és frissítéseket az ügyfelekhez a gyorsabb és hatékonyabb módon üzleti változásaihoz.

Azure-szolgáltatások, például az Azure Kubernetes Service, Container Registry és Cosmos DB használatával vállalatok használhatja a legújabb, az alkalmazás fejlesztési technikákkal és eszközök segítségével egyszerűsítheti azok végrehajtási magas rendelkezésre állású.

## <a name="related-use-cases"></a>Kapcsolódó alkalmazási helyzetek

Ebben a forgatókönyvben a következő használati esetek, vegye figyelembe:

* Modernizálhatja alkalmazás fejlesztési eljárásaikat mikroszolgáltatásokon és tárolókon alapuló megközelítést.
* Felgyorsítható az alkalmazások fejlesztése és üzembe helyezési életciklusának.
* Központi telepítések tesztelése és érvényesítési elfogadási környezetek automatizálása.

## <a name="architecture"></a>Architektúra

![Az Azure-összetevőket részt vesz egy fejlesztési és üzemeltetési környezetek a Jenkins az Azure Container Registry és Azure Kubernetes Service-architektúra áttekintése][architecture]

Ebben a forgatókönyvben egy Node.js-webalkalmazás és adatbázis-háttérrendszer fejlesztési és üzemeltetési folyamatot ismerteti. A áramlanak keresztül az adatok a forgatókönyv a következő:

1. A fejlesztők a Node.js webes alkalmazás forráskódjának módosítást hajt végre.
2. A kód módosítása számára fontos, hogy a forrás vezérlő adattár, például a GitHub.
3. A folyamatos integrációs (CI) folyamat elindításához egy GitHub-webhook aktiválja a Jenkins projekt build.
4. A Jenkins készítése feladatot használ a dinamikus fordító-ügynökhöz az Azure Kubernetes Service egy tároló-létrehozási folyamat végrehajtásához.
5. Egy tárolórendszerkép jön létre a kódot verziókövetési rendszerben, és ezután a rendszer továbbítja egy Azure Container Registrybe.
6. Jenkins folyamatos üzembe helyezés (CD) keresztül a Kubernetes-fürtöt helyezünk üzembe a a frissített tárolórendszerkép.
7. A Node.js-webalkalmazás Azure Cosmos DB, a háttérbeli használja. Cosmos DB és az Azure Kubernetes Service jelentéseket mérőszámok az Azure Monitor.
8. A Grafana példány nyújt az alkalmazásteljesítmény alapján az Azure Monitor az adatok vizuális irányítópultokkal.

### <a name="components"></a>Összetevők

* [A Jenkins] [ jenkins] egy nyílt forráskódú automatizáló kiszolgáló, amely integrálható az Azure-szolgáltatások engedélyezése a folyamatos integrációs (CI) és a folyamatos készregyártás (CD). Ebben a forgatókönyvben a Jenkins koordinálja véglegesítés verziókövetési alapján új tárolórendszerképek létrehozása, leküldi a rendszerképeket az Azure Container Registrybe, majd frissítések alkalmazáspéldányok Azure Kubernetes Service-ben.
* [Azure-beli Linux rendszerű virtuális gépek] [ azurevm-docs] a Jenkins és a Grafana példányok futtatásához használt.
* [Az Azure Container Registry] [ azureacr-docs] tárolja és kezeli az Azure Kubernetes Service-fürt által használt tárolórendszerképek. Képek a rendszer biztonságosan tárolja, és más régiókban is replikálja az Azure platform üzembe helyezéshez szükséges idő felgyorsítása érdekében.
* [Az Azure Kubernetes Service] [ azureaks-docs] egy felügyelt Kubernetes-platform, amely lehetővé teszi, hogy helyezhet üzembe és kezelhet tárolóalapú alkalmazásokat tárolóvezénylési szakértelem nélkül is. Üzemeltetett Kubernetes-szolgáltatásként az Azure olyan fontos műveleteket bonyolít le, mint az állapotmonitorozás és a karbantartás.
* [Az Azure Cosmos DB] [ azurecosmosdb-docs] egy globálisan elosztott, többmodelles adatbázis, amely lehetővé teszi, hogy illeszkedjen az igényeihez különféle adatbázis és a konzisztencia modellek közül választhat. A Cosmos DB használatával az adatok globális replikálása, és van nem fürt felügyeleti vagy a replikációs összetevő telepítését és konfigurálását.
* [Az Azure Monitor] [ azuremonitor-docs] segít a teljesítmény, a biztonság fenntartására és a trendek azonosítására. Figyelő által előállított metrikákat más erőforrásokat és eszközöket, mint a Grafana használhatják.
* [Grafana] [ grafana] egy nyílt forráskódú megoldás lekérdezésére, megjelenítése, riasztás és metrikák ismertetése. Egy adatforrás beépülő modul az Azure Monitor lehetővé teszi, hogy a Grafana létrehozása az Azure Kubernetes Service-ben futó és a Cosmos DB használatával az alkalmazások teljesítményének figyelése vizuális irányítópultokkal.

### <a name="alternatives"></a>Alternatív megoldások

* [A Visual Studio Team Services] [ vsts] és a egy folyamatos integrációs (CI), tesztelési és üzembe helyezése (CD) megvalósítása a Team Foundation Server Súgó folyamat-bármely alkalmazás.
* [Kubernetes] [ kubernetes] futtatható közvetlenül az Azure-beli virtuális gépek helyett egy felügyelt szolgáltatáson keresztül, ha szeretné jobban szabályozhatja a fürthöz.
* [A Service Fabric] [ service-fabric] van egy másik másodlagos tárolóvezénylő, mely lecserélheti az AKS.

## <a name="considerations"></a>Megfontolandó szempontok

### <a name="availability"></a>Rendelkezésre állás

Az alkalmazásteljesítmény monitorozásához és a jelentés a problémák, ebben a forgatókönyvben egyesíti a Grafana az Azure Monitor-vizuális irányítópultokkal. Ezek az eszközök teszik figyelheti, és előfordulhat, hogy az összes ezután központilag telepítheti a CI/CD-folyamat kódfrissítéseket igénylő teljesítménnyel kapcsolatos problémáinak elhárítása.

Az Azure Kubernetes Service-fürt részeként a terheléselosztó elosztja a forgalmat alkalmazás egy vagy több tárolók (podok) az alkalmazást futtató között. Ez a megközelítés a tárolóalapú alkalmazások Kubernetes-ben futó magas rendelkezésre állású infrastruktúrát biztosít az ügyfelek számára.

Rendelkezésre állási témaköröket talál a [rendelkezésre állási ellenőrzőlista] [ availability] az architektúra-központ érhető el.

### <a name="scalability"></a>Méretezhetőség

Az Azure Kubernetes Service lehetővé teszi a fürtcsomópontok, hogy a videólejátszást az alkalmazások számát. Ahogy az alkalmazás növekszik, ki lehet terjeszteni a szolgáltatás futtatásához Kubernetes-csomópontok száma.

Az Azure Cosmos DB egy globálisan elosztott, többmodelles adatbázis, amely globálisan méretezhető tárolt alkalmazásadatok. A cosmos DB kivonatolja az infrastruktúra mint a hagyományos adatbázis-összetevőket kell, és dönthet úgy, hogy a Cosmos DB globálisan, hogy a videólejátszást az ügyfelek replikálni.

Méretezhetőség témaköröket talál a [méretezési ellenőrzőlista] [ scalability] az architektúra-központ érhető el.

### <a name="security"></a>Biztonság

A támadások bekövetkeztének minimalizálása érdekében ez a forgatókönyv nem fedi fel a Jenkins Virtuálisgép-példány HTTP protokollon keresztül. Minden olyan felügyeleti feladatok, amely a jenkins használatával kezelheti a helyi gépen SSH-alagút használatával biztonságos távoli kapcsolatot hoz létre. A Jenkins és a Grafana Virtuálisgép-példányok csak az SSH nyilvános kulcsos hitelesítés engedélyezett. Jelszóalapú belépési kísérletek le vannak tiltva. További információkért lásd: [Jenkins-kiszolgáló futtatása az Azure-ban](../../reference-architectures/jenkins/index.md).

Ebben a forgatókönyvben hitelesítő adataival és engedélyeivel szétválasztása, használja az Azure Active Directory (AD) dedikált szolgáltatásnév. Az egyszerű szolgáltatás hitelesítő adatai, egy biztonságos hitelesítő objektumot a Jenkins tárolódnak, így, hogy ne legyenek közvetlenül elérhetővé tett és szkriptek vagy a buildelési folyamat látható.

Általános megoldások biztonságos, tekintse át a [Azure Security dokumentációja][security].

### <a name="resiliency"></a>Rugalmasság

Ebben a forgatókönyvben az alkalmazás az Azure Kubernetes Service használja. Kubernetes épített összetevői rugalmasság figyelő, és ha a probléma, indítsa újra a tárolókat (podok). Kombinálva több Kubernetes-csomópontokon fut, az alkalmazás működését egy pod vagy a csomópont nem érhető el.

Rugalmas megoldások tervezésével kapcsolatos általános útmutatásért lásd: [rugalmas alkalmazások tervezése az Azure][resiliency].

## <a name="deploy-the-scenario"></a>A forgatókönyv megvalósításához

**Előfeltételek.**

* Meglévő Azure-fiókkal kell rendelkeznie. Ha nem rendelkezik Azure-előfizetéssel, mindössze néhány perc alatt létrehozhat egy [ingyenes fiókot](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) a virtuális gép létrehozásának megkezdése előtt.
* Nyilvános ssh-kulcs van szüksége. Egy nyilvános kulcspárra létrehozásának lépéseiért lásd: [létrehozása és használata egy SSH-kulcsot a Linux rendszerű virtuális gépek párosítsa][sshkeydocs].
* A hitelesítési szolgáltatás és az erőforrások egy Azure Active Directory (AD) egyszerű szolgáltatást kell rendelkeznie. Szükség esetén is létrehoz egy szolgáltatást, az egyszerű [az ad sp create-for-rbac][createsp]

    ```azurecli-interactive
    az ad sp create-for-rbac --name myDevOpsScenario
    ```

    Jegyezze fel a *appId* és *jelszó* Ez a parancs kimenetében. A forgatókönyv telepítésekor ezeket az értékeket a sablonhoz adnia.

Ebben a forgatókönyvben egy Azure Resource Manager-sablon üzembe helyezéséhez hajtsa végre az alábbi lépéseket.

1. Kattintson a **üzembe helyezés az Azure** gombra:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fdevops-with-aks%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. Várja meg a sablon üzembe helyezéséhez az Azure Portal megnyitásához, majd kövesse az alábbi lépéseket:
   * Válassza ki a **új létrehozása** erőforrás csoportra, majd adjon meg egy nevet például *myAKSDevOpsScenario* a szövegmezőben.
   * Válassza ki a régiót, a **hely** legördülő listából.
   * A szolgáltatás egyszerű alkalmazás Azonosítóját és jelszavát adja meg a `az ad sp create-for-rbac` parancsot.
   * Adjon meg egy felhasználónevet és a biztonságos jelszó a Jenkins-példány és a Grafana konzol.
   * Adjon meg egy SSH-kulcsot a Linuxos virtuális gépekre történő bejelentkezések biztonságossá tételéhez.
   * Tekintse át a használati feltételeket, majd ellenőrizze **elfogadom a feltételeket és a fenti feltételeket**.
   * Válassza ki a **beszerzési** gombra.

Az üzembe helyezés befejeződik 15 – 20 percet is igénybe vehet.

## <a name="pricing"></a>Díjszabás

Ebben a forgatókönyvben költségének megismeréséhez, a szolgáltatások mindegyike a költségkalkulátor az előre konfigurált. Tekintse meg, hogyan díjszabását szeretné módosítani az adott használati esetekhez, módosítsa a megfelelő változókat egyezik a várt forgalomhoz.

Adtunk három példa költség profilok tárolólemezképek tárolására és az alkalmazások futtatása a Kubernetes-csomópontok száma alapján.

* [Kis][small-pricing]: Ez havonta 1000 tároló buildek utal.
* [Közepes][medium-pricing]: Ez havonta 100 000 tároló buildek utal.
* [Nagy][large-pricing]: Ez havonta 1 000 000 tároló buildek utal.

## <a name="related-resources"></a>Kapcsolódó erőforrások

Ebben a forgatókönyvben használt Azure Container Registry és Azure Kubernetes Service-ben tárolja, és a tárolóalapú alkalmazások futtatására. Az Azure Container Instances a tárolóalapú alkalmazások futtatása bármely vezénylési összetevők üzembe helyezése nélkül is használható. További információkért lásd: [Azure Container Instances áttekintésében][azureaci-docs].

<!-- links -->
[architecture]: ./media/devops-with-aks/architecture-devops-with-aks.png
[autoscaling]: ../../best-practices/auto-scaling.md
[availability]: ../../checklist/availability.md
[azureaci-docs]: /azure/container-instances/container-instances-overview
[azureacr-docs]: /azure/container-registry/container-registry-intro
[azurecosmosdb-docs]: /azure/cosmos-db/introduction
[azureaks-docs]: /azure/aks/intro-kubernetes
[azuremonitor-docs]: /azure/monitoring-and-diagnostics/monitoring-overview
[azurevm-docs]: /azure/virtual-machines/linux/overview
[createsp]: /cli/azure/ad/sp#az-ad-sp-create
[grafana]: https://grafana.com/
[jenkins]: https://jenkins.io/
[resiliency]: ../../resiliency/index.md
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[security]: /azure/security/
[scalability]: ../../checklist/scalability.md
[sshkeydocs]: /azure/virtual-machines/linux/mac-create-ssh-keys
[vsts]: /vsts/?view=vsts
[kubernetes]: https://kubernetes.io/
[service-fabric]: /azure/service-fabric/

[small-pricing]: https://azure.com/e/841f0a75b1ea4802ba1ac8f7918a71e7
[medium-pricing]: https://azure.com/e/eea0e6d79b4e45618a96d33383ec77ba
[large-pricing]: https://azure.com/e/3faab662c54c473da55a1e93a27e0e64
