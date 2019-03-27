---
title: Filmajánló az Azure-on
description: Gépi tanulás használata film- és termékajánlók, illetve egyéb javaslatok automatizálására Azure-beli modell betanítására szolgáló gépi tanulási megoldások és Azure Data Science Virtual Machine (DSVM) használatával.
author: njray
ms.date: 01/09/2019
ms.custom: azcat-ai, AI
social_image_url: /azure/architecture/example-scenario/ai/media/architecture-movie-recommender.png
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.openlocfilehash: be7959c1201e3a2aaf92c192d73a0ca47a990934
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299678"
---
# <a name="movie-recommendations-on-azure"></a>Filmajánló az Azure-on

A példa forgatókönyv bemutatja, hogyan egy üzleti használhatja a machine learning-ügyfelek számára, akik a termékekre vonatkozó javaslatok automatizálásához. Az Azure adatelemzési virtuális gép (DSVM) betanítja a modellt az Azure-on, filmek javasolja a felhasználók számára, amely filmek kapott minősítések alapján történik.

Javaslatok a kiskereskedelmi media hírek a különböző iparágakban hasznos lehet. A lehetséges alkalmazások termékekre vonatkozó javaslatok virtuális tárolja, híreket biztosít, vagy ossza meg javaslatokat nyújtó, vagy zene javaslatokat nyújtó tartalmazza. Hagyományosan vállalkozások felvétel kész, és az ügyfelek számára személyre szabott ajánlásokat asszisztensek kellett. Még ma felügyelniük Azure-ügyfél beállítások megértéséhez modelleket taníthat be a nagy mennyiségű személyre szabott ajánlásokat is biztosítunk.

## <a name="relevant-use-cases"></a>Alkalmazási helyzetek

Ebben a forgatókönyvben a következő használati esetek, vegye figyelembe:

* Film javaslatok-webhelyen.
* Fogyasztói termékekre vonatkozó javaslatok a mobilalkalmazásban.
* Hírek javaslattal médiafolyamokhoz.

## <a name="architecture"></a>Architektúra

![A gépi tanulási modell betanításához filmjánlásokat architektúrája][architecture]

Ez a forgatókönyv ismerteti, a képzés és a Spark használatával gépi tanulási modell kiértékelése [legkisebb négyzetek váltakozó] [ als] (ALS) algoritmus egy adatkészleten, film minősítése. Ez a forgatókönyv lépései a következők következő:

1. Az előtér-webhely vagy az app service felhasználói-movie kölcsönhatások, felhasználó, az elem és a numerikus minősítés rekordokat tartalmazó tábláját szereplő korábbi adatokat gyűjti össze.

2. A begyűjtött korábbi adatok egy blob storage-ban tárolt.

3. A DSVM kísérletezhet velük, a Spark ALS ajánló modell productize gyakran használják. A ALS modell tanítása az betanítási adatkészletet, amely a megfelelő adatok felosztása stratégia alkalmazásával a teljes adatkészlet előállítása. Például az adatkészlet felosztása csoportok véletlenszerűen, időrendi sorrendben, is, vagy műanyaggal rétegezett, attól függően, az üzleti követelményt. Más gépi tanulási feladatok hasonlóan egy ajánló érvényesítési értékelési mérőszámok használatával (például a pontosság\@*k*, visszaírási\@*k*, [térkép] [ map], [nDCG\@k][ndcg]).

4. Az Azure Machine Learning szolgáltatás a koordinációs a kísérleti, például a hiperparaméter kezdik és a modellkezelési szolgál.

5. Betanított modell megőrződik az Azure Cosmos DB, amely ezután alkalmazható felső megközelítéssel *k* filmek az adott felhasználó számára.

6. A modell majd Azure Container Instances szolgáltatásban vagy az Azure Kubernetes Service használatával telepít egy webes vagy az app service-be.

Részletes útmutató és ajánló-szolgáltatás méretezése, lásd: [valós idejű ajánlás API létrehozása az Azure-ban][ref-arch].

### <a name="components"></a>Összetevők

* [Adatelemző virtuális gép] [ dsvm] (DSVM) egy Azure virtuális gépen, a deep learning-keretrendszerek és eszközök a machine learning és az adattudomány. A DSVM rendelkezik egy önálló a Spark környezet ALS futtatásához használható.

* [Az Azure Blob storage] [ blob] filmjánlásokat adatkészletének tárolja.

* [Az Azure Machine Learning szolgáltatás] [ mls] gyorsabb létrehozását, kezelését és a machine learning-modellek üzembe helyezéséhez használt.

* [Az Azure Cosmos DB] [ cosmosdb] lehetővé teszi, hogy a globálisan elosztott és többmodelles adatbázis-tárolás.

* [Az Azure Container Instances] [ aci] szolgál a betanított modellek üzembe helyezése web- vagy app serviceshez, igény szerint segítségével [Azure Kubernetes Service][aks].

### <a name="alternatives"></a>Alternatív megoldások

[Az Azure Databricks] [ databricks] egy felügyelt Spark fürt, ahol a modell betanítása és kiértékelése történik. Beállíthat egy felügyelt Spark-környezetben, percek és [automatikus skálázási] [ autoscale] felfelé és lefelé, hogy az erőforrások és a fürtök manuális skálázás társított költségek csökkentése érdekében. Egy másik erőforrás-megtakarítási lehetőség, inaktív konfigurálása [fürtök] [ clusters] automatikusan leáll.

## <a name="considerations"></a>Megfontolandó szempontok

### <a name="availability"></a>Rendelkezésre állás

Machine learning-beépített alkalmazások osztották két erőforrás-összetevőt: képzési erőforrásokat és a kiszolgáló forrásokat. Képzés általában szükséges erőforrásokat éles kérések közvetlenül éri el ezeket az erőforrásokat, nem kell magas rendelkezésre állás érdekében. Kiszolgáló számára szükséges erőforrások magas rendelkezésre állású ügyfél kérelmek kiszolgálására van szükség.

A képzés, a DSVM érhető el a [több régióban] [ regions] szerte a világon, és megfelel-e a [szolgáltatói szerződést] [ sla] (SLA) számára virtuális gépek. A kiszolgáló, az Azure Kubernetes Service biztosít egy [magas rendelkezésre állású] [ ha] infrastruktúra. Ügynökcsomópontok is hajtsa végre a [SLA] [ sla-aks] virtuális gépek számára.

### <a name="scalability"></a>Méretezhetőség

Ha nagy mennyiségű adat méret, méretezhető képzési idő lerövidítheti a dsvm-hez. Méretezheti a virtuális gép felfelé vagy lefelé módosításával a [Virtuálisgép-méret][vm-size]. Válassza ki, a memória méretét elég nagy az adatkészlet a memóriában, és a egy nagyobb vCPU-számot idő csökkentése érdekében, hogy képzési vesz igénybe.

### <a name="security"></a>Biztonság

Ebben a forgatókönyvben az Azure Active Directory használatával hitelesítheti a felhasználókat a [a dsvm-hez való hozzáférés][dsvm-id], amely tartalmazza a kódot, modelleket és (memóriabeli) adatait. Adatok az Azure Storage tárolja a dsvm-hez, a betöltése előtt, ha automatikusan titkosítva használatával [a Storage Service Encryption][storage-security]. Engedélyek az Azure Active Directory-hitelesítés vagy a szerepköralapú hozzáférés-vezérlés használatával kezelhetők.

## <a name="deploy-this-scenario"></a>Ez a forgatókönyv megvalósítható

**Előfeltételek**: Meglévő Azure-fiókkal kell rendelkeznie. Ha nem rendelkezik Azure-előfizetéssel, első lépésként létrehozhat egy [ingyenes][free] fiókot.

Ebben a forgatókönyvben a kód érhető el a [Microsoft Recommenders tárház][github].

Futtassa a következő lépésekkel a [ALS rövid notebook][notebook]:

1. [Hozzon létre egy DSVM] [ dsvm-ubuntu] az Azure Portalról.

2. Klónozza az adattárat a notebookok mappában:

    ```shell
    cd notebooks
    git clone https://github.com/Microsoft/Recommenders
    ```

3. Telepítse a következő témakörben ismertetett conda-függőségeket a [SETUP.md] [ setup] fájlt.

4. Egy böngészőben nyissa meg a virtuális gép jupyterlab, és navigáljon a `notebooks/00_quick_start/als_pyspark_movielens.ipynb`.

5. Hajtsa végre a jegyzetfüzetet.

## <a name="related-resources"></a>Kapcsolódó források (lehet, hogy a cikkek angol nyelvűek)

Részletes útmutató és ajánló-szolgáltatás méretezése, lásd: [valós idejű ajánlás API létrehozása az Azure-ban][ref-arch]. Oktatóanyagok és példák a javaslattételi rendszerek: [Microsoft Recommenders tárház][github].

[architecture]: ./media/architecture-movie-recommender.png
[aci]: /azure/container-instances/container-instances-overview
[aad]: /azure/active-directory-b2c/active-directory-b2c-overview
[aks]: /azure/aks/intro-kubernetes
[als]: https://spark.apache.org/docs/latest/ml-collaborative-filtering.html
[autoscale]: https://docs.azuredatabricks.net/user-guide/clusters/sizing.html#autoscaling
[blob]: /azure/storage/blobs/storage-blobs-introduction
[clusters]: https://docs.azuredatabricks.net/user-guide/clusters/configure.html
[cosmosdb]: /azure/cosmos-db/introduction
[databricks]: /azure/azure-databricks/what-is-azure-databricks
[dsvm]: /azure/machine-learning/data-science-virtual-machine/overview
[dsvm-id]: /azure/machine-learning/data-science-virtual-machine/dsvm-common-identity
[dsvm-ubuntu]: /azure/machine-learning/data-science-virtual-machine/dsvm-ubuntu-intro
[free]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
[github]: https://github.com/Microsoft/Recommenders
[ha]: /azure/aks/container-service-quotas
[map]: https://en.wikipedia.org/wiki/Evaluation_measures_(information_retrieval)
[mls]: /azure/machine-learning/service/
[n-tier]: /azure/architecture/reference-architectures/n-tier/n-tier-cassandra
[ndcg]: https://en.wikipedia.org/wiki/Discounted_cumulative_gain
[notebook]: https://github.com/Microsoft/Recommenders/notebooks/00_quick_start/als_pyspark_movielens.ipynb
[ref-arch]: /azure/architecture/reference-architectures/ai/real-time-recommendation
[regions]: https://azure.microsoft.com/en-us/global-infrastructure/services/?products=virtual-machines&regions=all
[resiliency]: /azure/architecture/resiliency/
[sec-docs]: /azure/security/
[setup]: https://github.com/Microsoft/Recommenders/blob/master/SETUP.md%60
[sla]: https://azure.microsoft.com/en-us/support/legal/sla/virtual-machines/v1_8/
[sla-aks]: https://azure.microsoft.com/en-us/support/legal/sla/kubernetes-service/v1_0/
[storage-security]: /azure/storage/common/storage-service-encryption
[vm-size]: /azure/virtual-machines/virtual-machines-linux-change-vm-size
