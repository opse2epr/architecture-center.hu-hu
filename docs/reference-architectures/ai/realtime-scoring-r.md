---
title: Machine Learning-modellek valós idejű pontozása
description: Egy valós idejű előrejelzési szolgáltatás megvalósítása az R, Machine Learning-kiszolgáló futtatása az Azure Kubernetes Service (AKS) használatával.
author: njray
ms.date: 12/12/2018
ms.custom: azcat-ai
ms.openlocfilehash: 6f3447d1dcab801ccdaf4cf88611725cc00eb68d
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/08/2019
ms.locfileid: "54112277"
---
# <a name="real-time-scoring-of-r-machine-learning-models"></a>Machine Learning-modellek valós idejű pontozása

Ez a referenciaarchitektúra bemutatja, hogyan egy valós idejű (szinkron) előrejelzési szolgáltatás megvalósítása az R Microsoft Machine Learning-kiszolgáló futtatása az Azure Kubernetes Service (AKS) használatával. Ez az architektúra célja, hogy általános és bármely valós idejű szolgáltatásként telepíteni kívánt R beépített prediktív modell számára a megfelelő. **[A megoldás üzembe helyezése][github]**.

## <a name="architecture"></a>Architektúra

![Valós idejű pontozás a R machine learning modellek az Azure-ban][0]

Ez a referenciaarchitektúra egy tároló-alapú megközelítést vesz igénybe. Egy Docker-rendszerkép összeállításakor tartalmazó R, valamint a különböző összetevők szükséges új adatok pontozása céljából. Ezek közé tartozik a modellobjektumot, saját maga és a egy pontozószkriptre. Ez a rendszerkép Docker-tárolójegyzék az Azure-ban üzemeltetett, és ezután üzembe helyezett egy Kubernetes-fürthöz emellett az Azure-ban való leküldésekor.

Ez a munkafolyamat architektúrája a következő összetevőket tartalmazza.

- **[Az Azure Container Registry] [ acr]**  a munkafolyamat a képek tárolására szolgál. A beállításjegyzékek tároló-beállításjegyzék létrehozása a standard keresztül kezelhetők [Docker Registry V2 API] [ docker] és az ügyfél.

- **[Az Azure Kubernetes Service] [ aks]**  az üzembe helyezés és a szolgáltatás futtatására szolgál. A standard használatával létrehozott AKS kezelhetők [Kubernetes API-t] [ k-api] és az ügyfél (kubectl).

- **[Microsoft Machine Learning-kiszolgáló] [ mmls]**  a REST API, a szolgáltatás meghatározására szolgál, és tartalmazza a [modell Operacionalizálás][operationalization]. A szolgáltatásorientált web server folyamat figyeli a kéréseket, amely majd jussanak el a többi háttérfolyamatot, amely a tényleges, az eredmények R-kód. Ezeket a folyamatokat ebben a konfigurációban egy tárolóba van csomagolva egyetlen csomópontján fut. Ezen kívül olyan fejlesztési-tesztelési környezet szolgáltatás használatával kapcsolatos részletekért forduljon a Microsoft helyi képviselőjéhez.

## <a name="performance-considerations"></a>A teljesítménnyel kapcsolatos megfontolások

Machine learning számítási feladatok általában nagy számítási igényű egyaránt betanításakor, és amikor új adatok pontozása. Javasolt próbálja nem magonként egynél több pontozási folyamat futtatásához. Machine Learning-kiszolgáló lehetővé teszi az egyes tárolókban futó R-folyamatok számának meghatározásához. Az alapértelmezett érték öt folyamatokat. Egy viszonylag egyszerű, például a változók kis számú, vagy egy kis döntési fa egy lineáris regressziós modell létrehozása során növelheti a folyamatok számát. Figyelheti a CPU-terhelést, a fürtcsomópontokon meghatározni a megfelelő, a tárolók számára vonatkozó határértéket.

A GPU-kompatibilis fürt felgyorsítható a számítási feladatok és deep learning-modellek elsősorban bizonyos típusú. Nem minden munkaterhelésről kihasználhatja a GPU-k &mdash; mátrixalgebra használatát csak azok, amelyek (nagy erőforrásigényű). Például fa épülő modellekhez, beleértve a véletlenszerű erdők és a modellek, kiemelési általában származtatást nem használja ki a GPU-kkal.

Néhány adatmodell-típusokat, például a véletlenszerű erdők olyan nagymértékben párhuzamosítható a processzorok. Ezekben az esetekben gyorsabb a pontozás egyetlen kérelem több mag között osztja meg a munkaterhelést. Ezzel azonban így csökkenti a kapacitás a megadott rögzített fürtméret több pontozási kérelmek.

Általában a nyílt forráskódú R-modellek a memóriában tárolja az összes az adatokat, ezért győződjön meg arról, hogy a csomópontok rendelkeznek-e elegendő memória a folyamatok egyidejű futtatását tervezi. Machine Learning-kiszolgáló használ a modellekhez, ha a lemezen lévő adatokat tud feldolgozni tár ahelyett, hogy olvasó, mind a memóriába. Ez segít jelentősen csökkentheti a memóriára vonatkozó követelményeknek. Függetlenül attól, hogy azt használni, Machine Learning-kiszolgáló vagy a nyílt forráskódú R győződjön meg arról, hogy a pontozási folyamatokat nem memória-fogy ki a csomópontok figyelése.

## <a name="security-considerations"></a>Biztonsági szempontok

### <a name="network-encryption"></a>hálózati titkosítás

Ez a referenciaarchitektúra HTTPS engedélyezve van a fürt és a egy átmeneti tanúsítvány való kommunikációhoz [hozzunk titkosítása] [ encrypt] szolgál. Élesben helyettesítse be egy megfelelő aláíró jogosultságokat a saját tanúsítványt.

### <a name="authentication-and-authorization"></a>Hitelesítés és engedélyezés

A Machine Learning-kiszolgáló [modell Operacionalizálás] [ operationalization] pontozási kérelmek hitelesítése szükséges. Ebben a felállásban felhasználónevet és jelszót használják. A vállalati környezetben való felépítésüktől, engedélyezheti a hitelesítés [Azure Active Directory] [ AAD] vagy hozzon létre egy külön előtér [Azure API Management] [ API].

A modell Operacionalizálás tárolók a Machine Learning-kiszolgáló megfelelően működjenek telepítenie kell egy JSON webes jogkivonat (JWT) tanúsítványt. A központi telepítés egy Microsoft által biztosított tanúsítványt használ. Egy éles beállításban adja meg a saját.

Container Registry és az AKS közötti forgalom esetén érdemes lehet engedélyezni az [szerepköralapú hozzáférés-vezérlés] [ rbac] (RBAC) hozzáférés korlátozható, ha csak a szükséges.

### <a name="separate-storage"></a>Külön tárolási

Ez a referenciaarchitektúra egy egyszeri lemezképpel az alkalmazás (R) és az adatokat (a modell objektum és a pontozó szkript) bundles. Néhány esetben lehet előnyös, ha ezek egymástól. Az Azure blob- vagy elhelyezhet a modell adatait és a kód [tárolási][storage], és kérheti le az azokat tároló inicializáláskor. Ebben az esetben győződjön meg arról, hogy a tárfiók értéke csak hitelesített hozzáférést, és a HTTPS protokollt használjon.

## <a name="monitoring-and-logging-considerations"></a>Figyelés és naplózás kapcsolatos szempontok

Használja a [Kubernetes-irányítópult] [ dashboard] figyelheti az AKS-fürt általános állapotát. Tekintse meg a fürt áttekintés panel az Azure Portalon további részletekért. A [GitHub] [ github] erőforrásokat is bemutatják, hogyan viszi, megjelenik az irányítópulton az R.

Az irányítópult áttekintést nyújt egy, a fürt általános állapotát, de fontos is az egyes tárolók állapotának nyomon követését. Ehhez engedélyezze [Azure Monitor Insights] [ monitor] az Azure Portalon, vagy tekintse meg a fürt áttekintés panelen [-tárolókhoz az Azure Monitor] [ monitor-containers](az előzetes verzió).

## <a name="cost-considerations"></a>Költségekkel kapcsolatos szempontok

Machine Learning-kiszolgáló van licencelése magonként alapul, és a fürt felé, a Machine Learning-kiszolgáló száma futtatandó összes mag. Ha egy vállalati Machine Learning-kiszolgáló vagy a Microsoft SQL Server-ügyfél, a díjszabás részleteiért lépjen kapcsolatba a Microsoft helyi képviselőjéhez.

A Machine Learning-kiszolgáló nyílt forráskódú alternatív [Plumber][plumber], az R-csomag, amely a kód egy REST API-bA. Plumber teljes kevesebb, mint a Machine Learning-kiszolgáló megjelent. Például alapértelmezés szerint azt a szolgáltatásokat, adja meg a hitelesítési kérelem nem tartalmazza. Ha Plumber használ, javasoljuk, hogy engedélyezze [Azure API Management] [ API] hitelesítési részletek kezeléséhez.

Mellett licencelés, a fő szempont költsége a Kubernetes-fürt számítási erőforrásokat. A fürt elég nagynak kell lennie kezelni a várt kötet időszakokban, de ez a megközelítés hagyja erőforrások üresjárati más időpontban. Korlátozni az erőforrások üresjáratának hatását, engedélyezze a [vízszintes méretező] [ autoscaler] a fürthöz a kubectl eszközzel. Vagy használja az AKS [méretező fürt][cluster-autoscaler].

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

A referenciaimplementációt a jelen architektúra érhető el az [GitHub][github]. Egy egyszerű prediktív modellen szolgáltatás telepítése nem ismertetett lépéseket követve.

<!-- links -->
[AAD]: /azure/active-directory/fundamentals/active-directory-whatis
[API]: /azure/api-management/api-management-key-concepts
[ACR]: /azure/container-registry/container-registry-intro
[AKS]: /azure/aks/intro-kubernetes
[autoscaler]: https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/
[cluster-autoscaler]: /azure/aks/autoscaler
[monitor]: /azure/monitoring/monitoring-container-insights-overview
[dashboard]: /azure/aks/kubernetes-dashboard
[docker]: https://docs.docker.com/registry/spec/api/
[encrypt]: https://letsencrypt.org/
[gitHub]: https://github.com/Azure/RealtimeRDeployment
[K-API]: https://kubernetes.io/docs/reference/
[MMLS]: /machine-learning-server/what-is-machine-learning-server
[monitor-containers]: /azure/azure-monitor/insights/container-insights-overview
[operationalization]: /machine-learning-server/what-is-operationalization
[plumber]: https://www.rplumber.io
[RBAC]: /azure/role-based-access-control/overview
[storage]: /azure/storage/common/storage-introduction
[0]: ./_images/realtime-scoring-r.png
