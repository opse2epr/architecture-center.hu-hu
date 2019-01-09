---
title: A Python-modellek valós idejű pontozása
titleSuffix: Azure Reference Architectures
description: Ez a referenciaarchitektúra bemutatja, hogyan helyezhet üzembe Python-modelleket az Azure-ra előrejelzéseket valós idejű webszolgáltatásként.
author: msalvaris
ms.date: 11/09/2018
ms.custom: azcat-ai
ms.openlocfilehash: b40072b43630adf13e8ead0b6aa6bec59ca1bfac
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/08/2019
ms.locfileid: "54110892"
---
# <a name="real-time-scoring-of-python-scikit-learn-and-deep-learning-models-on-azure"></a>Valós idejű pontozási Python Scikit-ismerje meg, és a deep learning-modellek az Azure-ban

Ez a referenciaarchitektúra bemutatja, hogyan helyezhet üzembe Python modelleket webszolgáltatásként, hogy a valós idejű előrejelzéseket végezzen. Két esetben terjed ki: reguláris Python-modellek és a konkrét követelmények, deep learning-modellek üzembe helyezésének telepítése. Mindkét forgatókönyvet használja az architektúra látható.

Ez az architektúra két hivatkozás megvalósításait érhetők el a Githubon, egyet-egyet [rendszeres Python modellek] [ github-python] és a egy [deep learning-modellek] [ github-dl].

![Az Azure-ban a Python-modellek a valós idejű pontozás architektúra diagramja](./_images/python-model-architecture.png)

## <a name="scenarios"></a>Forgatókönyvek

A referencia-megvalósítások Ez az architektúra használatával két forgatókönyvet mutatják be.

**1. forgatókönyv: Gyakori kérdések a megfelelő**. Ebből a forgatókönyvből megtudhatja, hogyan helyezhet üzembe egy gyakori kérdések (GYIK) megfelelő modellt webszolgáltatásként, amely felhasználói kérdések adatokat biztosít. Ebben a forgatókönyvben az architektúra diagramját "bemeneti adatok" felhasználói kérdések felel meg a gyakori kérdések listáját tartalmazó szöveges karakterlánc hivatkozik. Ebben a forgatókönyvben készült a [scikit-további] [ scikit] machine learning-kódtár Pythonhoz készült, de bármilyen forgatókönyvhöz, amely a Python-modell segítségével valós idejű előrejelzéseket általánosítva is.

Ebben a forgatókönyvben az adatok egy részét Stack Overflow kérdés, amely tartalmazza az eredeti kérdések, JavaScript, az ismétlődő kérdések és a válaszok címkével használ. Ez betanítja a scikit-ismerje meg a folyamat használatával előrejelzi az egyes az eredeti kérdések a ismétlődő kérdés egyezés valószínűségét. Ezek az előrejelzések egy REST API-végpont használatával valós időben történik.

Az architektúra az alkalmazás folyamata a következőképpen történik:

1. Az ügyfél a kódolt kérdés adatokkal HTTP POST-kérelmet küld.

2. A Flask-alkalmazás kibontja a kérdést a kérelemből.

3. A kérdés küld a scikit-featurization és pontozás adatfolyamat-modell további.

4. A megfelelő – gyakori kérdések a kérdéseket a pontszámok irányíthatja át, JSON-objektum és az ügyfél számára.

Íme egy Képernyőkép a PéldaAlkalmazás, amely az eredményeket fel:

![A PéldaAlkalmazás képernyőképe](./_images/python-faq-matches.png)

**2. forgatókönyv: Kép besorolási**. Ez a forgatókönyv bemutatja, hogyan modell üzembe helyezése Konvolúciós Neurális hálózat (CNN) webszolgáltatásként, amely előrejelzéseket meg képeket. Ebben a forgatókönyvben az architektúra diagramját "bemeneti adatok" képfájlok hivatkozik. Cnn-EK nagyon hatékonyak a számítógépes látástechnológiai olyan feladatokhoz, mint a képek besorolása és objektumfelismeréshez. Ebben a forgatókönyvben a keretrendszereket, TensorFlow, a Keras (háttérrendszerrel a tensorflow-hoz) és a PyTorch lett tervezve. Azonban azt is általánosítva bármilyen forgatókönyvhöz, amely valós idejű előrejelzéseket deep learning-modellek segítségével.

Ez a forgatókönyv ResNet-152 előre betanított modell előre jelezni az eseménykategóriát épít – 1 K (1000 osztályok) adatkészlet tanított használ (lásd az alábbi ábrát) lemezkép tartozik. Ezek az előrejelzések egy REST API-végpont használatával valós időben történik.

![Előrejelzés – példa](./_images/python-example-predictions.png)

Deep learning-modellhez az alkalmazás folyamata a következőképpen történik:

1. Az ügyfél a kódolt kép adatokkal HTTP POST-kérelmet küld.

2. A Flask-alkalmazás kibontja a lemezképet a kérelemből.

3. A rendszerkép üzenetfájlrekordok, és a modell pontozása küldött.

4. A pontértéket irányíthatja át, JSON-objektum, és az ügyfél számára.

## <a name="architecture"></a>Architektúra

Ez az architektúra a következő összetevőkből áll.

**[Virtuális gép] [ vm]**  (VM). A virtuális gép jelenik meg, mint például az eszköz &mdash; helyi vagy felhőbeli &mdash; , amely a HTTP-kérést küldhet.

**[Az Azure Kubernetes Service] [ aks]**  (AKS) segítségével helyezze üzembe az alkalmazást a Kubernetes-fürtön. Az AKS egyszerűsíti az üzembe helyezés és a kubernetes az operations. A fürt a Python-modell rendszeres vagy GPU-kompatibilis virtuális gépek deep learning-modellek csak CPU virtuális gépek használatával konfigurálható.

**[Terheléselosztó][lb]**. A szolgáltatás kívülről elérhetővé egy terheléselosztót, az AKS által kiosztott szolgál. A háttér-podok átirányítja a forgalmat a terheléselosztótól.

**[A docker Hub] [ docker]**  tárolja a Docker-rendszerkép üzembe helyezett Kubernetes-fürtön. A docker Hub az architektúra lett választva, mert egyszerűen használható, és az alapértelmezett lemezkép adattár a Docker-felhasználók. [Az Azure Container Registry] [ acr] az architektúra is használható.

## <a name="performance-considerations"></a>A teljesítménnyel kapcsolatos megfontolások

Valós idejű pontozási architektúrák az átviteli teljesítmény válik domináns veszi figyelembe. Python-modellek rendszeres általánosan elfogadott, hogy a processzorok elegendőek a számítási feladatok kezelésére.

Azonban deep learning számítási feladatokhoz, amikor sebessége szűk keresztmetszetet gpu-k általánosan biztosítja jobban [teljesítmény] [ gpus-vs-cpus] processzorok képest. Megfelelő processzorokat használata GPU-teljesítményét, nagy mennyiségű processzor-fürt általában van szükség.

Mindkét forgatókönyvre érvényes ez az architektúra a processzorok is használhat, de deep learning-modellek, gpu-kat nyújt a jelentősen nagyobb átviteli sebesség értékek hasonló díjmentes CPU fürt képest. Az AKS támogatja a GPU-k, ez az egyik előnye az AKS használatával Ez az architektúra. Ezenkívül deep learning-telepítések általában modellek használata nagy számú paraméterrel. Gpu-k használata megakadályozza, hogy a versengés az erőforrásokat a modell és a webes szolgáltatást, amely a hiba csak CPU-telepítések között.

## <a name="scalability-considerations"></a>Méretezési szempontok

Gondoskodunk a rendszeres Python modellek, ahol ki van építve az AKS-fürt csak CPU-alapú virtuális gépekhez, amikor [horizontális felskálázás a podok számát][manually-scale-pods]. A cél, hogy használja ki teljesen a fürtöt. Skálázás processzorigényét és a podok a meghatározott függ. Kubernetes is támogatja a [az automatikus skálázás] [ autoscale-pods] a processzorhasználat podok száma vagy egyéb, módosíthatja a podok kiválaszthatja a metrikákat. A [méretező fürt] [ autoscaler] (az előzetes verzió) skálázhatja ügynökcsomópontok függőben van a podok alapján.

Deep learning-forgatókönyvek, GPU-kompatibilis virtuális gépek használatával erőforráskorlátok a podok is úgy, hogy egy GPU hozzá van rendelve egy pod. Virtuális gép használt típusától függően kell [a fürt csomópontjainak méretezése] [ scale-cluster] a szolgáltatás iránti igény kielégítése érdekében. Ezt az Azure CLI-vel és a kubectl használatával könnyedén megteheti.

## <a name="monitoring-and-logging-considerations"></a>Figyelés és naplózás kapcsolatos szempontok

### <a name="aks-monitoring"></a>AKS-figyelés

Betekintést nyerhet az AKS-teljesítmény, használja a [-tárolókhoz az Azure Monitor] [ monitor-containers] funkció. Tartományvezérlők, a csomópontok és a Kubernetes, a metrikák API-n keresztül a rendelkezésre álló tárolók memória- és metrikákat gyűjt.

Az alkalmazás üzembe helyezése közben figyelheti az AKS-fürtöt, győződjön meg arról, hogy a várt módon működik, a csomópontokon működnek, és minden podok is futnak. Bár használhatja a [kubectl] [ kubectl] is beolvasni a pod állapotát, a Kubernetes parancssori eszköz tartalmaz egy webes irányítópultot, a fürt állapota és a felügyeleti alapszintű figyeléséhez.

![A Kubernetes-irányítópult képernyőképe](./_images/python-kubernetes-dashboard.png)

A fürt és a csomópontok általános állapotának megtekintéséhez nyissa meg a **csomópontok** a Kubernetes-irányítópult szakaszában. Ha egy csomópont inaktív vagy nem sikerült, megjelenítheti a hibanaplókat, onnan hivatkozó oldalakon találhat. Hasonlóképpen, nyissa meg a **Podok** és **központi telepítések** szakaszokban talál információt a podok számát és a központi telepítés állapotát.

### <a name="aks-logs"></a>AKS-naplók

Az AKS automatikusan naplóz minden stdout/stderr a podok a fürt naplóit. Tekintse át ezeket a kubectl használatával és a is csomópont-szintű eseményeit, és a naplókat. További információkért lásd: a telepítés lépéseit.

Használat [-tárolókhoz az Azure Monitor] [ monitor-containers] metrikák és a linuxhoz, ami a Log Analytics-munkaterületet a Log Analytics-ügynököket tárolóalapú verziója keresztül naplóinak összegyűjtésére.

## <a name="security-considerations"></a>Biztonsági szempontok

Használja az [Azure Security Centert][security-center] az Azure-erőforrások biztonsági állapotának egy központi felületen való nyomon követéséhez. Security Center monitorozza a potenciális biztonsági problémákat, és az üzembe helyezés, a biztonsági állapotának átfogó képet nyújt, bár az ügynök AKS-csomópontok nem figyel. A Security Center Azure-előfizetésenként külön konfigurálandó. Biztonsági adatok gyűjtésének engedélyezése a leírtak szerint [az Azure-előfizetés a Security Center Standard verziójába történő felvételét][get-started]. Az adatgyűjtés engedélyezése esetén a Security Center automatikusan figyeli az előfizetés alatt létrehozott összes virtuális gépet.

**Műveletek**. Jelentkezzen be egy AKS-fürtöt az Azure Active Directory (Azure AD) hitelesítési jogkivonat használatával, konfigurálja az Azure AD használata az AKS [felhasználói hitelesítés][aad-auth]. Fürt rendszergazdák is konfigurálhatják Kubernetes szerepköralapú hozzáférés-vezérlés (RBAC) a felhasználó identitását, vagy a könyvtár a csoport tagsága alapján.

Használat [RBAC] [ rbac] üzembe helyezett Azure-erőforrásokhoz való hozzáférés szabályozása. Az RBAC lehetővé teszi, hogy engedélyezési szerepköröket rendeljen a fejlesztő és üzemeltető csapata tagjaihoz. Egy felhasználó több szerepkört kell hozzárendelni, és létrehozhat egyéni szerepköröket a még részletesebb [engedélyek].

**HTTPS**. Biztonsági szempontból ajánlott az alkalmazás kell HTTPS kényszerítése és HTTP-kérést. Használja az [bejövőforgalom-vezérlőjéhez] [ ingress-controller] üzembe helyezéséhez a fordított proxy, amely lezárja az SSL, és átirányítja a HTTP-kérelmekre. További információkért lásd: [hozzon létre egy HTTPS bejövőforgalom-vezérlőt az Azure Kubernetes Service (AKS)][https-ingress].

**Hitelesítés**. Ez a megoldás nem korlátozza a végpontok való hozzáférést. A vállalati környezetben való felépítésüktől az architektúra üzembe helyezéséhez a végpontok védelme keresztül API-kulcsokat, és valamilyen felhasználói hitelesítés hozzáadása az ügyfélalkalmazásnak.

**Tárolóregisztrációs adatbázis**. Ez a megoldás nyilvános beállításjegyzék tárolja a Docker-rendszerképet használ. A kódot, amely az alkalmazás függ, és a modell tartalmazza a rendszerképben. Vállalati alkalmazások és kell használnia, privát regisztrációs adatbázis hardvermeghibásodásokkal szemben, rosszindulatú kódot futtató annak érdekében, hogy a tárolóban az adatokat illetéktelen kezekbe kerüljenek.

**A DDoS protection**. Érdemes lehet engedélyezni az [DDoS Protection Standard][ddos]. Alapszintű DDoS protection az Azure platform részeként engedélyezve van, noha a DDoS Protection Standard kockázatcsökkentési képességeket biztosít, amelyek kifejezetten az Azure virtuális hálózati erőforrások, amelyek ideálisak.

**Naplózás**. Ajánlott eljárások használata naplóadatokat, például a felhasználói jelszavakat és egyéb információkat, amelyek segítségével biztonsági csalások tisztítási tárolása előtt.

## <a name="deployment"></a>Környezet

Ez a referenciaarchitektúra üzembe helyezéséhez kövesse a GitHub-adattárak leírt lépéseket követve:

- [Rendszeres Python-modell][github-python]
- [Deep learning-modellek][github-dl]

<!-- links -->

[aad-auth]: /azure/aks/aad-integration
[acr]: /azure/container-registry/
[something]: https://kubernetes.io/docs/reference/access-authn-authz/authentication/
[aks]: /azure/aks/intro-kubernetes
[autoscaler]: /azure/aks/autoscaler
[autoscale-pods]: /azure/aks/tutorial-kubernetes-scale#autoscale-pods
[azcopy]: /azure/storage/common/storage-use-azcopy-linux
[ddos]: /azure/virtual-network/ddos-protection-overview
[docker]: https://hub.docker.com/
[get-started]: /azure/security-center/security-center-get-started
[github-python]: https://github.com/Azure/MLAKSDeployment
[github-dl]: https://github.com/Microsoft/AKSDeploymentTutorial
[gpus-vs-cpus]: https://azure.microsoft.com/en-us/blog/gpus-vs-cpus-for-deployment-of-deep-learning-models/
[https-ingress]: /azure/aks/ingress-tls
[ingress-controller]: https://kubernetes.io/docs/concepts/services-networking/ingress/
[kubectl]: https://kubernetes.io/docs/tasks/tools/install-kubectl/
[lb]: /azure/load-balancer/load-balancer-overview
[manually-scale-pods]: /azure/aks/tutorial-kubernetes-scale#manually-scale-pods
[monitor-containers]: /azure/monitoring/monitoring-container-insights-overview
[engedélyek]: /azure/aks/concepts-identity
[rbac]: /azure/active-directory/role-based-access-control-what-is
[scale-cluster]: /azure/aks/scale-cluster
[scikit]: https://pypi.org/project/scikit-learn/
[security-center]: /azure/security-center/security-center-intro
[vm]: /azure/virtual-machines/
