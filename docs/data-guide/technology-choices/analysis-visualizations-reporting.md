---
title: Egy data analytics kiválasztása és jelentéskészítési technológia
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.openlocfilehash: a5e793c9caf50daca4ef7e40c49e54f25f04e856
ms.sourcegitcommit: e7e0e0282fa93f0063da3b57128ade395a9c1ef9
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/05/2018
ms.locfileid: "52902416"
---
# <a name="choosing-a-data-analytics-technology-in-azure"></a>Egy data analytics technológia kiválasztása az Azure-ban

A legtöbb big data-megoldás célja az, hogy elemzéssel és jelentéskészítéssel betekintést nyújtson az adatokba. Ez magában foglalhatja előre konfigurált jelentéseket és vizualizációkat, vagy adatok interaktív áttekintését. 

## <a name="what-are-your-options-when-choosing-a-data-analytics-technology"></a>Mik azok a beállítások data analytics technológia kiválasztásakor?

Az elemzés, a Vizualizációk és a jelentéskészítés az Azure-ban, igényeitől függően több lehetőség áll rendelkezésre:

- [Power BI](/power-bi/)
- [Jupyter notebook](https://jupyter.readthedocs.io/en/latest/index.html)
- [Zeppelin notebookok](https://zeppelin.apache.org/)
- [A Microsoft Azure-jegyzetfüzetek](https://notebooks.azure.com/)

### <a name="power-bi"></a>Power BI

[Power bi-ban](/power-bi/) üzleti elemzési eszközök együttese. Száz adatforráshoz csatlakozhat, és ad hoc elemzést is használható. Lásd: [ebben a listában](/power-bi/desktop-data-sources) jelenleg elérhető adatforrások. Használat [Power BI Embedded](https://azure.microsoft.com/services/power-bi-embedded/) Power BI integrálható a saját alkalmazásokon belül anélkül, hogy bármilyen további licencelést.

Szervezetek használhatják a Power BI jelentések létrehozására, és közzéteheti őket a szervezet számára. Bárki létrehozhat személyre szabott irányítópultok, a cégirányítási és [beépített biztonsági](/power-bi/service-admin-power-bi-security). A Power bi-ban használt [Azure Active Directory](/azure/active-directory/) (Azure AD), jelentkezzen be a Power BI szolgáltatásban és a Power BI bejelentkezési hitelesítő adatok, amikor egy felhasználó megpróbál hozzáférni a hitelesítést megkövetelő használ felhasználók hitelesítésére.

### <a name="jupyter-notebooks"></a>Jupyter-notebookok 

[A Jupyter Notebooks](https://jupyter.readthedocs.io/en/latest/index.html) adja meg, amely lehetővé teszi az adatszakértők, hozzon létre egy böngészőalapú rendszerhéj *notebook* fájlokat, Python, Scala és R kód és a markdown szöveget tartalmaznak, így működhet megosztása hatékony módszert és dokumentálása kód és a egy dokumentum eredményez.

HDInsight-fürtök, például a Spark vagy a Hadoop, a legtöbb fajtáinak származnak [előre konfigurált Jupyter notebookok a](/azure/hdinsight/spark/apache-spark-jupyter-notebook-kernels) adatok használatához és feldolgozási feladatok elküldése. HDInsight-fürtöt használ típusától függően egy vagy több kernelekkel értelmezésében és a kódja fut. biztosítjuk. Például a HDInsight Spark-fürtök közül választhat a Spark-motort használó Python- vagy a Scala kód végrehajtása a Spark-kapcsolódó kernelt biztosítanak.

Jupyter-notebookok elemzése, jelenítenek meg, és a egy BI/reporting eszközzel, mint a Power BI speciális vizualizációkészítő előtt az adatok feldolgozása egy kiváló környezetet biztosítanak.

### <a name="zeppelin-notebooks"></a>Zeppelin notebookok

[Zeppelin notebookok](https://zeppelin.apache.org/) egy másik lehetőség egy böngészőalapú rendszerhéj, Jupyter funkciói hasonlóak a rendszer. Egyes HDInsight-fürtök térjen [Zeppelin-jegyzetfüzetek az előre konfigurált](/azure/hdinsight/spark/apache-spark-zeppelin-notebook). Azonban ha használ egy [HDInsight interaktív lekérdezés](/azure/hdinsight/interactive-query/apache-interactive-query-get-started) (Hive LLAP)-fürt [Zeppelin](/azure/hdinsight/hdinsight-connect-hive-zeppelin) jelenleg csak Ön által választott jegyzetfüzet, interaktív Hive-lekérdezések futtatásához használhatja. Ezenkívül ha használ egy [tartományhoz csatlakoztatott HDInsight-fürt](/azure/hdinsight/domain-joined/apache-domain-joined-introduction), Zeppelin-jegyzetfüzetek esetén kizárólag, amely lehetővé teszi, hogy különböző felhasználói bejelentkezések notebookok és az alapul szolgáló Hive-táblákban való hozzáférés szabályozásához rendeljen.

### <a name="microsoft-azure-notebooks"></a>A Microsoft Azure-jegyzetfüzetek

[Az Azure notebookok](https://notebooks.azure.com/) Jupyter Notebooks-alapú egy online szolgáltatás, amely lehetővé teszi az adatszakértők, létrehozása, futtatása és megosztása a felhő alapú tárak Jupyter-Notebookjait. Azure notebookok végrehajtási környezetet biztosít a Python 2, a Python 3, F#, és az R, és számos diagramkészítési kódtárakat biztosít az adatai, például a ggplot, matplotlib, bokeh és seaborn megjelenítéséhez.

Azure notebookok Jupyter notebookok egy HDInsight-fürtön futó, amely csatlakozik a fürt alapértelmezett tárfiókot, eltérően nem biztosít adatokat. Meg kell [adatok betöltése](https://notebooks.azure.com/Microsoft/libraries/samples/html/Getting%20to%20your%20Data%20in%20Azure%20Notebooks.ipynb) többféle módon, az ilyen online forrásból származó adatokat tölti le, az Azure BLOB vagy Table Storage használata, az SQL-adatbázishoz szeretne csatlakozni vagy a Másolás varázslóval az adatok betöltése az Azure Data Factoryhoz.

Fő előnyök:

* Az ingyenes szolgáltatás&mdash;nem Azure-előfizetés szükséges.
* Nem kell telepítenie a Jupyter és a támogató R vagy Python disztribúciók helyileg&mdash;használja egy böngészőben.
* A saját online tárak, és azok bármely eszközről elérhetők.
* Ossza meg a notebookok közreműködőkkel együtt.

Szempontok:

* Ön nem fér hozzá a notebookok offline állapotban lesz.
* Az ingyenes notebook szolgáltatás korlátozott feldolgozási képességek nem lehet elég nagy vagy összetett modelleket taníthat be.

## <a name="key-selection-criteria"></a>Fontosabb kritériumok

Így szűkítheti, első lépésként a kérdések megválaszolása:

- Szüksége számos adatforráshoz csatlakozhat egy központi helyen, a tartomány teljes adatokat számára létrehozandó jelentésekhez biztosít? Ha igen, válasszon olyan beállítás, amely lehetővé teszi, hogy az adatforrások 100s csatlakozni.

- Biztosan dinamikus vizualizációkat ágyazhat be egy külső webhely vagy alkalmazás? Ha igen, válasszon egy beállítást, amely beágyazási képességeket biztosít.

- Tervezési a Vizualizációk és jelentések során szeretné offline állapotban van? Ha igen, válassza ki egy beállítást, az offline képességeiről.

- Nagy feldolgozási teljesítményt nagy vagy összetett AI-modellek betanításához, vagy használni a rendkívül nagy méretű adatkészleteket kell? Ha igen, válasszon olyan beállítás, amely a big data-fürtökhöz csatlakozhat.

## <a name="capability-matrix"></a>Képességmátrix

A következő táblázat összefoglalja a fő különbségeket, a képességek. 

### <a name="general-capabilities"></a>Általános képességek

| | Power BI | Jupyter-notebookok | Zeppelin notebookok | A Microsoft Azure-jegyzetfüzetek |
| --- | --- | --- | --- | --- |
| A big data-fürt létrehozása speciális feldolgozó kapcsolódni | Igen | Igen | Igen | Nem |
| Felügyelt szolgáltatás | Igen | Igen <sup>1</sup> | Igen <sup>1</sup> | Igen |
| Az adatforrások 100s csatlakozni | Igen | Nem | Nem | Nem |
| Offline képességek | Igen <sup>2</sup> | Nem | Nem | Nem |
| Képességek beágyazása | Igen | Nem | Nem | Nem |
| Adatok automatikus frissítése | Igen | Nem | Nem | Nem |
| Számos nyílt forráskódú csomag való hozzáférés | Nem | Igen <sup>3</sup> | Igen <sup>3</sup> | Igen <sup>4</sup> |
| Adatok átalakítása és tisztítását beállításai | [A Power Query](https://powerbi.microsoft.com/blog/getting-started-with-power-query-part-i/), R | 40 nyelvek, többek között a Python, R, Julia és Scala | mint 20 interprety, beleértve a Python, JDBC és R | Python, F#, R |
| Díjszabás | Az ingyenes Power BI Desktop (szerzői műveletek), lásd: [díjszabás](https://powerbi.microsoft.com/pricing/) beállítások üzemeltetéséhez | Ingyenes | Ingyenes | Ingyenes |
| Többfelhasználós együttműködés | [Igen](/power-bi/service-how-to-collaborate-distribute-dashboards-reports) | Igen (megosztása vagy például a többfelhasználós kiszolgálóval [JupyterHub](https://github.com/jupyterhub/jupyterhub)) | Igen | Igen (a megosztás) |

[1] Ha egy felügyelt HDInsight-fürt része.

[2] a Power BI Desktop használatával.

[2] kereshet a [Maven tárházból](https://search.maven.org/) a Közösség által biztosított csomagok.

[3] Python-csomagokat a pip vagy conda használatával telepíthető. R-csomagok CRAN vagy a GitHub is telepíthető. A csomagok F# keresztül nuget.org használatával telepítheti a [Paket Függőségkezelő](https://fsprojects.github.io/Paket/).

