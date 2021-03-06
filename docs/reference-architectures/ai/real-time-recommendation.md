---
title: Valós idejű ajánlás API létrehozása az Azure-ban
description: A gépi tanulási automatizálhatja az Azure Databricks és az Azure adatelemzési virtuális gépek (DSVM) segítségével az Azure-ban a modell betanításához javaslatokat.
author: njray
ms.date: 12/12/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: azcat-ai
ms.openlocfilehash: c7e7423da11667c90d53247c2c5303a8fbd1a76a
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/17/2019
ms.locfileid: "59640158"
---
# <a name="build-a-real-time-recommendation-api-on-azure"></a>Valós idejű ajánlás API létrehozása az Azure-ban

Ez a referenciaarchitektúra bemutatja egy Azure Databricks használatával javaslat modell betanítását, és helyezze üzembe, API-k az Azure Cosmos DB, az Azure Machine Learning és az Azure Kubernetes Service (AKS) használatával. Ez az architektúra is általánosítható legtöbb javaslat motor helyzetekre, termékek, filmek és hírek is.

Az architektúra egy referenciaimplementációt érhető el az [GitHub][als-example].

![A gépi tanulási modell betanításához filmjánlásokat architektúrája](./_images/recommenders-architecture.png)

**A forgatókönyv**: Egy media szervezet célja az filmet vagy videó javaslatokat biztosít a felhasználóknak. Személyre szabott javaslatok biztosításával a szervezet megfelel a több üzleti céljait, beleértve a megnövekedett átkattintásos díjszabás, a webhelyen, és nagyobb felhasználói elégedettséget megnövekedett engagement.

Ez a referenciaarchitektúra és üzembe helyezése egy valós idejű ajánló szolgáltatás API, amely képes biztosítani az adott felhasználó számára a legfontosabb 10 movie javaslatok van.

Ez az ajánlás modellhez az adatok az adatfolyam a következőképpen történik:

1. Nyomon követheti a felhasználói viselkedés. Háttérszolgáltatás naplózásra, például amikor egy felhasználó egy filmet értékeli vagy egy termék vagy news cikk.

2. Az adatok betöltéséhez az Azure Databricksbe az elérhető [adatforrás][data-source].

3. Az adatok előkészítése, és azt fel képzési és egy tesztelési a modell betanításához. ([Ez az útmutató] [ guide] vágását meghatározó adatok lehetőségeket ismerteti.)

4. Alkalmas a [Spark együttműködési szűrést] [ als] modell az adatokhoz.

5. Értékelje ki a minősítés és rangsorolása mérőszámok segítségével a modell minőségét. ([Ez az útmutató] [ eval-guide] részletesen a ajánló kiértékelheti a metrikákat.)

6. A felső 10 javaslatok felhasználónként és áruházbeli precompute gyorsítótárként Azure Cosmos DB-ben.

7. API-szolgáltatás üzembe az aks-tárolóba, és üzembe helyezése az API-t az Azure Machine Learning API-k használatával.

8. Ha a háttérszolgáltatáshoz kérelem olvas be egy felhasználó, hívja meg a javaslatok az aks-ben a felső 10 javaslatokat kaphat, és megjeleníti őket a felhasználó üzemeltetett API.

## <a name="architecture"></a>Architektúra

Ez az architektúra a következő összetevőkből áll:

[Az Azure Databricks][databricks]. Databricks egy fejlesztési környezetet a bemeneti adatok előkészítése, és a Spark-fürtön ajánló modell betanításához. Az Azure Databricks egy interaktív munkaterület futtassa, és közösen dolgozhatnak a notebookok bármilyen adatfeldolgozás és a gépi tanulási feladatok is biztosít.

[Az Azure Kubernetes Service] [ aks] (AKS). Az AKS üzembe helyezése és a egy gépi tanulási modell szolgáltatás API-t a Kubernetes-fürt üzembe helyezése szolgál. Az AKS üzemelteti a tárolóalapú modell méretezhetőséget, amely megfelel az átviteli sebességet megkövetelő, identitás- és hozzáférés-kezelés és naplózás és szolgáltatásállapot-figyelést.

[Az Azure Cosmos DB][cosmosdb]. A cosmos DB egy globálisan elosztott adatbázis-szolgáltatás tárolja a felső 10 ajánlott filmek minden felhasználóhoz. Azure Cosmos DB a ebben a forgatókönyvben is használható, mert biztosít alacsony késéssel (10 ezredmásodperc 99. percentilis), olvassa el a legfontosabb ajánlott elemeket az adott felhasználó számára.

[Az Azure Machine Learning szolgáltatás][mls]. Ez a szolgáltatás segítségével nyomon követheti és kezelheti a machine learning-modellek, és majd becsomagolhatja és ezek a modellek üzembe helyezése méretezhető AKS-környezetben.

[A Microsoft Recommenders][github]. A nyílt forráskódú tárház segédprogram-kódot tartalmaz, és különböző minták segítik a felhasználók létrehozásához, kiértékelése, modellezést és a egy ajánló rendszer használatának megkezdéséhez.

## <a name="performance-considerations"></a>A teljesítménnyel kapcsolatos megfontolások

Teljesítményének valós idejű javaslatokat számára elsődleges jelentőségű azért ajánlások általában esik a kritikus útvonalat a kérelem egy felhasználó által a webhelyen.

AKS és Azure Cosmos DB kombinációja lehetővé teszi, hogy ez az architektúra a javaslatokat is ad egy közepes méretű számítási feladatok minimális fenntartási az jó kiindulási pont biztosítása. Ez az architektúra egy terhelési teszt 200 egyidejű felhasználóval, ajánlásokkal medián késése, 60 MS-os, és 180 kérelmek másodpercenkénti adatátviteli kapacitással hajt végre. A terhelési teszt volt futtatva, szemben az alapértelmezett telepítési konfigurációt (egy 3 12 vcpu-k x D3 v2 AKS-fürtöt, 42 GB memória, és 11 000 [kérelemegység (RU) másodpercenként] [ ru] kiépítése az Azure Cosmos DB).

![Teljesítmény-grafikon](./_images/recommenders-performance.png)

![Átviteli sebesség-grafikon](./_images/recommenders-throughput.png)

Az Azure Cosmos DB a kulcsrakész globális disztribúciót és az összes adatbázis követelményeknek, az alkalmazás hasznosságát használata ajánlott. A kis mértékben [gyorsabb késés][latency], érdemes lehet [Azure Redis Cache] [ redis] helyett az Azure Cosmos DB keresések kiszolgálása érdekében. Redis Cache javíthatja a teljesítményt a háttér-tárolja az adatok magas használó rendszerek.

## <a name="scalability-considerations"></a>Méretezési szempontok

Ha nem tervezi a Spark használata, vagy ha kisebb számítási feladatokhoz, ahol nem kell terjesztési, érdemes [adatelemző virtuális gép] [ dsvm] (DSVM) az Azure Databricks helyett. DSVM használata egy Azure virtuális gépen, a deep learning-keretrendszerek és gép tanulási és adatelemző eszközei. Mint az Azure Databricks, egy adatelemző virtuális GÉPET hoz létre minden olyan modellt is kell üzembe helyezte azt az aks-en keresztül az Azure Machine Learning szolgáltatás.

Során képzés, nagyobb rögzített méretű Spark-fürt kiépítése az Azure Databricks, vagy konfigurálja [az automatikus skálázás][autoscaling]. Ha az automatikus skálázás engedélyezve van, Databricks figyeli a fürt terhelésének és felskálázással, és ahonnan szükség esetén. Kiépítés vagy horizontális felskálázás nagyobb fürt rendelkezik egy nagy mennyiségű adat méretét, és ha szeretné az adat-előkészítési vagy feladatok modellezési szükséges idő csökkentése érdekében.

Az AKS-fürtöt, a teljesítmény, átviteli sebesség igényeinek megfelelően méretezhető. Ügyeljen arra, hogy a vertikális felskálázás száma [podok] [ scale] használja ki teljesen a fürtöt, és skálázhatja a [csomópontok] [ nodes] az igény, a fürt a szolgáltatás. A teljesítmény-és átviteli sebességet megkövetelő a ajánló szolgáltatás fürt méretezése további információkért lásd: [méretezése az Azure Container Service-fürtök][blog].

Azure Cosmos DB teljesítményének kezelésére, megbecsülheti a szükséges másodpercenként olvasott számát, és kiépítése az száma [kérelemegység / másodperc] [ ru] (teljesítmény) szükséges. Ajánlott eljárások használata [particionálási és horizontális skálázást][partition-data].

## <a name="cost-considerations"></a>Költségekkel kapcsolatos szempontok

Költség ebben a forgatókönyvben a fő illesztőprogramok a következők:

- Az Azure Databricks-fürt szükséges képzés méretének.
- Az AKS-fürt méretét a teljesítményre vonatkozó követelmények teljesítéséhez szükséges.
- Az Azure Cosmos DB Kérelemegységek kiépítette a hálózatiteljesítmény-igények kielégítéséhez.

Az Azure Databricks költségek kezelése az átképezési ritkábban, és kikapcsolja a Spark-fürtöt, amikor nincs használatban. Az AKS és Azure Cosmos DB költségek kötődnek, az átviteli sebesség és a hely által igényelt teljesítmény, és a webhely forgalmának mennyiségétől függően felfelé és lefelé skálázhatja.

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

Ez az architektúra üzembe helyezéséhez kövesse a **Azure Databricks** utasításait a [telepítő dokumentum][setup]. Röviden utasításokat kell tennie:

1. Hozzon létre egy [Azure Databricks-munkaterület][workspace].

1. Hozzon létre egy új fürtöt az Azure Databricksben a következő beállításokkal:

    - Fürt üzemmód: Standard
    - Databricks futtatókörnyezet-verziója: 4.3-as (tartalmazza az Apache Spark 2.3.1, Scala 2.11-et)
    - Python-verzió: 3
    - Illesztőprogram típusa: Standard\_DS3\_v2
    - Feldolgozó típusa: Standard szintű\_DS3\_v2 (minimális és maximális száma szükség szerint)
    - Automatikus lezárás: (a megadása kötelező)
    - Spark-Config: (a megadása kötelező)
    - Környezeti változók: (kötelezőként)

1. Hozzon létre egy személyes hozzáférési tokent belül a [Azure Databricks-munkaterület][workspace]. Tekintse meg az Azure Databricks hitelesítési [dokumentáció] [ adbauthentication] részleteiről.

1. Klónozás a [Microsoft Recommenders] [ github] környezetbe adattár, ahol futtathat parancsfájlokat (pl. a helyi számítógép).

1. Kövesse a **rövid telepítési** telepítési utasításokat a [a megfelelő kódtárak telepítése] [ setup] az Azure databricks szolgáltatásban.

1. Kövesse a **rövid telepítési** telepítési utasításokat a [előkészítése az Azure Databricks az operacionalizáláshoz][setupo16n].

1. Importálás a [ALS Movie Operacionalizálás notebook] [ als-example] egyszerűen a munkaterületre. Miután bejelentkezett az Azure Databricks-munkaterület, tegye a következőket:

    a. Kattintson a **kezdőlap** a munkaterület bal oldalán.

    b. Kattintson a jobb gombbal az üres helyet a kezdőkönyvtárban. Válassza ki **importálás**.

    c. Válassza ki **URL-cím**, és illessze be a következőt a szövegmezőhöz: `https://github.com/Microsoft/Recommenders/blob/master/notebooks/05_operationalize/als_movie_o16n.ipynb`

    d. Kattintson az **Importálás** gombra.

1. Nyissa meg a notebook az Azure databricksben, és csatolja a konfigurált fürtöt.

1. A jegyzetfüzet futtatásához használja a javaslatok API létrehozásához szükséges Azure-erőforrások létrehozása, amely az adott felhasználó számára a top-10 movie javaslatot is tartalmaz.

## <a name="related-architectures"></a>Kapcsolódó referenciaarchitektúrák

Is készítettük el egy referencia-architektúra, amely a Spark és az Azure Databricks végrehajtásához ütemezett [kötegelt pontozási folyamatokat][batch-scoring]. A referenciaarchitektúra egy új javaslatok rendszeresen létrehozásához ajánlott módszer megismeréséhez tekintse meg.

<!-- links -->
[aci]: /azure/container-instances/container-instances-overview
[aad]: /azure/active-directory-b2c/active-directory-b2c-overview
[adbauthentication]: https://docs.azuredatabricks.net/api/latest/authentication.html#generate-a-token
[aks]: /azure/aks/intro-kubernetes
[als]: https://spark.apache.org/docs/latest/ml-collaborative-filtering.html
[als-example]: https://github.com/Microsoft/Recommenders/blob/master/notebooks/05_operationalize/als_movie_o16n.ipynb
[autoscaling]: https://docs.azuredatabricks.net/user-guide/clusters/sizing.html
[autoscale]: https://docs.azuredatabricks.net/user-guide/clusters/sizing.html#autoscaling
[availability]: /azure/architecture/checklist/availability
[batch-scoring]: /azure/architecture/reference-architectures/ai/batch-scoring-databricks
[blob]: /azure/storage/blobs/storage-blobs-introduction
[blog]: https://blogs.technet.microsoft.com/machinelearning/2018/03/20/scaling-azure-container-service-cluster/
[clusters]: https://docs.azuredatabricks.net/user-guide/clusters/configure.html
[cosmosdb]: /azure/cosmos-db/introduction
[data-source]: https://docs.azuredatabricks.net/spark/latest/data-sources/index.html
[databricks]: /azure/azure-databricks/what-is-azure-databricks
[dsvm]: /azure/machine-learning/data-science-virtual-machine/overview
[dsvm-ubuntu]: /azure/machine-learning/data-science-virtual-machine/dsvm-ubuntu-intro
[eval-guide]: https://github.com/Microsoft/Recommenders/blob/master/notebooks/03_evaluate/evaluation.ipynb
[free]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
[github]: https://github.com/Microsoft/Recommenders
[guide]: https://github.com/Microsoft/Recommenders/blob/master/notebooks/01_prepare_data/data_split.ipynb
[latency]: https://github.com/jessebenson/azure-performance
[mls]: /azure/machine-learning/service/
[n-tier]: /azure/architecture/reference-architectures/n-tier/n-tier-cassandra
[ndcg]: https://en.wikipedia.org/wiki/Discounted_cumulative_gain
[nodes]: /azure/aks/scale-cluster
[notebook]: https://github.com/Microsoft/Recommenders/notebooks/00_quick_start/als_pyspark_movielens.ipynb
[partition-data]: /azure/cosmos-db/partition-data
[redis]: /azure/redis-cache/cache-overview
[regions]: https://azure.microsoft.com/global-infrastructure/services/?products=virtual-machines&regions=all
[resiliency]: /azure/architecture/resiliency/
[ru]: /azure/cosmos-db/request-units
[sec-docs]: /azure/security/
[setup]: https://github.com/Microsoft/Recommenders/blob/master/SETUP.md#repository-installation
[setupo16n]: https://github.com/Microsoft/Recommenders/blob/master/SETUP.md#prepare-azure-databricks-for-operationalization
[scale]: /azure/aks/tutorial-kubernetes-scale
[sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_8/
[vm-size]: /azure/virtual-machines/virtual-machines-linux-change-vm-size
[workspace]: https://docs.azuredatabricks.net/getting-started/index.html
