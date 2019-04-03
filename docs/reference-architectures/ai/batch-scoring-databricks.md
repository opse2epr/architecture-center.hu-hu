---
title: Spark-modellek kötegelt kiértékelése az Azure Databricksben
description: Hozzon létre egy méretezhető megoldás, a kötegelt pontozási egy Apache Spark osztályozási modell egy ütemezés szerint az Azure Databricks használatával.
author: njray
ms.date: 02/07/2019
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: azcat-ai
ms.openlocfilehash: cba8d272ddbdbf2c2da94f68b288e9fb79be7de2
ms.sourcegitcommit: 1a3cc91530d56731029ea091db1f15d41ac056af
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/03/2019
ms.locfileid: "58887811"
---
# <a name="batch-scoring-of-spark-machine-learning-models-on-azure-databricks"></a>Batch-pontozás a Spark machine learning-modellek az Azure Databricksben

Ez a referenciaarchitektúra bemutatja, hogyan hozhat létre egy méretezhető megoldás, a kötegelt pontozási egy Apache Spark osztályozási modell egy ütemezés szerint az Azure Databricks, egy Azure-ra optimalizált Apache Spark-alapú elemzési platform használatával. A megoldás, amely más forgatókönyvek is általánosítva sablonként is használható.

Az architektúra egy referenciaimplementációt érhető el az [GitHub][github].

![Spark-modellek kötegelt kiértékelése az Azure Databricksben](./_images/batch-scoring-spark.png)

**A forgatókönyv**: Az eszközintelligencia-(nagy erőforrásigényű) iparági szeretne a költségek és a társított vehesse a műszaki hibáknak váratlan állásidő minimalizálása érdekében. IoT-adatok használatával gyűjtött gépeik, létrehozhat egy prediktív karbantartási modellt. Ez a modell lehetővé teszi, hogy az üzleti összetevők karbantartása proaktív módon, és javítsa ki őket, mielőtt meghiúsulnak. A lehető legnagyobb vehesse a műszaki összetevő használja, akkor költségek csökkentését és csökkenthető a leállások.

Egy prediktív karbantartási modell gyűjti az adatokat a gépeket, és megőrzi a korábbi példa az összetevők hibáinak. A modell majd az összetevők aktuális állapotának figyelésére, és előre jelezni, ha egy adott összetevő a közeljövőben nem használható. Gyakori használati esetek és modellezés megközelítéseket, lásd: [Azure AI-útmutató a prediktív karbantartási megoldások][ai-guide].

Ez a referenciaarchitektúra lett tervezve, amelyek új adatokat az összetevő gépekről jelenléte esetén. Feldolgozás az alábbi lépésekből áll:

1. Betöltheti az adatokat, egy Azure Databricks-adattár az alakzatot a külső adatokat az adattárból.

2. Egy gépi tanulási modell tanítása az adatok átalakítása a tanítási adathalmazt be, akkor jön létre a Spark MLlib modell. MLlib áll a leggyakrabban használt gépi tanulási algoritmusok és segédprogramok optimalizáltuk, hogy kihasználhassák a Spark adatok skálázhatósági képességeket.

3. A betanított modell előre jelezni a alkalmazni (besorolása) összetevő hibák által az adatok átalakítása egy pontozó adatok csoportba. Az adatok Spark MLLib modell pontozása.

4. Eredmények Store a Databricks adatok áruházban utófeldolgozása használatalapú.

A notebookok biztosított [GitHub] [ github] egyes feladatok végrehajtásához.

## <a name="architecture"></a>Architektúra

Az architektúra határozza meg, amely teljes egészében tartalmazza az adatfolyam [Azure Databricks] [ databricks] egy készlete alapján egymás után végrehajtott [notebookok] [ notebooks]. Az alábbi összetevőkből áll:

**[Adatfájlok][github]**. A referenciaimplementációt használja egy szimulált adatok statikus adatok öt fájl található.

**[Adatbetöltési][notebooks]**. Az adatok betöltési notebook letölti a bemeneti adatok fájlokat Databricks adatkészletek egy gyűjteménybe. IoT-eszközökről származó adatok a való életből vett helyzet adatfolyam lenne-Databricks elérhető storage – például az Azure SQL-kiszolgáló vagy az Azure Blob storage-be. Databricks támogatja több [adatforrások][data-sources].

**Betanítási folyamat**. Ez a jegyzetfüzet hajtja végre a szolgáltatás mérnöki notebook-analysis adatkészlet létrehozása a feldolgozott adatokból. Ezután végrehajtja a modell létrehozása jegyzetfüzet, amely a machine learning-modell használatával betanítja a [Apache Spark MLlib] [ mllib] skálázható gépi tanulási kódtár.

**Pontozó folyamat**. Ez a jegyzetfüzet hajt végre a szolgáltatás mérnöki notebook pontozó-adatkészlet létrehozása a feldolgozott adatokból, és végrehajtja a pontozási notebookot. A pontozó notebookot használja a betanított [Spark MLlib] [ mllib-spark] modellt létrehozni az adatokat a megfigyelések a pontozási adatkészletében. Az előrejelzés tárolja az eredményeket tároló, a Databricks-tárolót az új adatkészletet.

**A Scheduler**. A Databricks ütemezett [feladat] [ job] leírók kötegelt pontozási a Spark-modell. A feladat végrehajtása a pontozási folyamat jegyzetfüzet, a változó argumentumoknak keresztül notebook paramétereket adja meg a hozhat létre, a pontozó adatkészlet és az eredménykészletet adatok tárolására.

A forgatókönyv egy folyamat folyamatként jön létre. Minden egyes notebook arra optimalizáltuk, hogy hajtsa végre az egyes műveletek egy batch-beállítás: adatfeldolgozást, a funkciófejlesztési, a modell létrehozásának és a modell scorings. Ehhez a szolgáltatás mérnöki notebook célja bármely, a képzés, a hitelesítési, tesztelési vagy pontozási tevékenységek általános adatkészlet létrehozásához. Ebben a forgatókönyvben használjuk egy historikus split stratégia ezeket a műveleteket, így a notebook paraméterek használhatók dátumtartomány szűrés beállítása.

A forgatókönyv egy batch-folyamatot hoz létre, mert egy nem kötelező vizsgálat notebookok böngészhet a folyamat jegyzetfüzet kimenetét készletét biztosítunk. Ezek a GitHub-adattárban található:

- `1a_raw-data_exploring`
- `2a_feature_exploration`
- `2b_model_testing`
- `3b_model_scoring_evaluation`

## <a name="recommendations"></a>Javaslatok

Databricks be van állítva, így betölteni és adatelemzésre új adatokkal betanított modellek üzembe helyezése. Is használt Databricks ebben a forgatókönyvben, mert a következő további előnyökkel:

- Egyszeri bejelentkezésének támogatása az Azure Active Directory hitelesítő adataival.
- Feladatütemező feladatai gyártási folyamatok végrehajtásához.
- Teljes mértékben interaktív notebook együttműködéssel, irányítópultok, a REST API-k.
- Korlátlan számú fürtök, amelyek bármilyen méretű méretezhető.
- Speciális biztonsági, a szerepköralapú hozzáférés-vezérlés és a naplók.

Együttműködik az Azure Databricks szolgáltatást, használja a Databricks [munkaterület] [ workspace] felületet a böngészőben vagy a [parancssori felület] [ cli] (CLI). A Databricks parancssori felület bármilyen platformon, amely támogatja a Python 2.7.9 3.6 érhető el.

A referenciaimplementáció használ [notebookok] [ notebooks] sorrendben feladatok végrehajtásához. Minden egyes jegyzetfüzet, a bemeneti adatok ugyanabban az adattárban való köztes adatösszetevők (képzés, tesztelési, pontozási vagy eredmények adatkészletek) tárolja. A célja, hogy könnyen használható, az adott használati esetekhez igény szerint. A gyakorlatban a jegyzetfüzetek olvasása és írása vissza közvetlenül a storage-bA az Azure Databricks példány az adatforrás kíván csatlakozni.

Figyelheti a feladat végrehajtása a Databricks felhasználói felület, az adattár vagy a Databricks [CLI] [ cli] szükség szerint. A fürt használatával figyelheti a [Eseménynapló] [ log] és egyéb [metrikák] [ metrics] Databricks biztosító.

## <a name="performance-considerations"></a>A teljesítménnyel kapcsolatos megfontolások

Egy Azure Databricks-fürt automatikus skálázás alapértelmezés szerint lehetővé teszi, hogy futásidőben, Databricks dinamikusan reallocates feldolgozók, hogy figyelembe vegye a feladat jellemzői. Előfordulhat, hogy a folyamat egyes részeit több nagy számítási igényű, mint mások. Databricks hozzáadja a további feldolgozó során ezeket a fázisokat a feladat (és eltávolítja őket, amikor szükség van rájuk már nem). Az automatikus skálázás megkönnyíti a nagy eléréséhez [kihasználtság fürt][cluster], mert nem kell egy munkaterheléshez a fürt üzembe helyezése.

Ezenkívül az összetettebb ütemezett folyamatok használatával fejleszthetők [Azure Data Factory] [ adf] az Azure Databricks.

## <a name="storage-considerations"></a>A tárterülettel kapcsolatos szempontok

A referenciaimplementációt, az adatok vannak tárolva közvetlenül az egyszerűség kedvéért egy Databricks-tároló. Éles környezetben azonban az adatok tárolhatók a felhőalapú adattárolás például [Azure Blob Storage][blob]. [Databricks] [ databricks-connect] is támogatja az Azure Data Lake Store, az Azure SQL Data Warehouse, Azure Cosmos DB, az Apache Kafka és Hadoop.

## <a name="cost-considerations"></a>Költségekkel kapcsolatos szempontok

Az Azure Databricks Spark prémium egy kapcsolódó költségek ajánlattal. Emellett nincsenek standard és prémium szintű Databricks [tarifacsomagok][pricing].

A jelen esetben a standard díjcsomag is használhatók. Azonban ha az adott alkalmazás automatikus skálázás fürtökhöz nagyobb méretű számítási feladatokat vagy interaktív Databricks irányítópultok kezelésére, a prémium szintű megnövelheti további költségeket.

A megoldás notebookok bármely Spark-alapú platform, távolítsa el a Databricks-specifikus csomagok minimális módosítások is futtathatja. Tekintse meg a következő hasonló megoldásokat az Azure különböző platformokhoz:

- [Az Azure-ban a Python Machine Learning Studióba][python-aml]
- [Az SQL Server R services][sql-r]
- [PySpark-az Azure Data Science virtuális gépen][py-dvsm]

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

Ez a referenciaarchitektúra üzembe helyezéséhez kövesse az ismertetett lépéseket a [GitHub] [ github] tárházat hozhat létre egy méretezhető megoldás, a batch az Azure Databricks Spark modellek pontozási.

## <a name="related-architectures"></a>Kapcsolódó referenciaarchitektúrák

Emellett készítettük el egy referencia-architektúra, amely létrehozásához használja a Spark [valós idejű javaslattételi rendszerek] [ recommendation] az offline, előre kiszámított pontszámokat. A javaslattételi rendszerek olyan gyakori forgatókönyvek, ahol pontszámok kötegelt feldolgozásra.

[adf]: https://azure.microsoft.com/blog/operationalize-azure-databricks-notebooks-using-data-factory/
[ai-guide]: /azure/machine-learning/team-data-science-process/cortana-analytics-playbook-predictive-maintenance
[blob]: https://docs.databricks.com/spark/latest/data-sources/azure/azure-storage.html
[cli]: https://docs.databricks.com/user-guide/dev-tools/databricks-cli.html
[cluster]: https://docs.azuredatabricks.net/user-guide/clusters/sizing.html
[databricks]: /azure/azure-databricks/
[databricks-connect]: /azure/azure-databricks/databricks-connect-to-data-sources
[data-sources]: https://docs.databricks.com/spark/latest/data-sources/index.html
[github]: https://github.com/Azure/BatchSparkScoringPredictiveMaintenance
[job]: https://docs.databricks.com/user-guide/jobs.html
[log]: https://docs.databricks.com/user-guide/clusters/event-log.html
[metrics]: https://docs.databricks.com/user-guide/clusters/metrics.html
[mllib]: https://docs.databricks.com/spark/latest/mllib/index.html
[mllib-spark]: https://docs.databricks.com/spark/latest/mllib/index.html#apache-spark-mllib
[notebooks]: https://docs.databricks.com/user-guide/notebooks/index.html
[pricing]: https://azure.microsoft.com/en-us/pricing/details/databricks/
[python-aml]: https://gallery.azure.ai/Notebook/Predictive-Maintenance-Modelling-Guide-Python-Notebook-1
[py-dvsm]: https://gallery.azure.ai/Tutorial/Predictive-Maintenance-using-PySpark
[recommendation]: /azure/architecture/reference-architectures/ai/real-time-recommendation
[sql-r]: https://gallery.azure.ai/Tutorial/Predictive-Maintenance-Modeling-Guide-using-SQL-R-Services-1
[workspace]: https://docs.databricks.com/user-guide/workspace.html
