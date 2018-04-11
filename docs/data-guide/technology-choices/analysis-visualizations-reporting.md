---
title: Egy adatelemzés kiválasztása és jelentéskészítési technológia
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 830c61bba64a6971c815330887e5cdcc4f2b5f56
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-a-data-analytics-technology-in-azure"></a>Olyan adatok analytics technológia kiválasztása az Azure-ban

A legtöbb big data-megoldások célja az adatok elemzési és jelentéskészítési betekintést. Ez magában foglalhatja előre konfigurált jelentéseket és a képi megjelenítések, vagy az adatok interaktív áttekintését. 

## <a name="what-are-your-options-when-choosing-a-data-analytics-technology"></a>Mik azok a beállítások adatok analytics technológia kiválasztásakor?

Elemzés, képi megjelenítéseket, és jelentéskészítési Azure igényeitől függően több lehetőség áll rendelkezésre:

- [Power BI](/power-bi/)
- [Jupyter notebook](https://jupyter.readthedocs.io/en/latest/index.html)
- [Zeppelin notebookok](https://zeppelin.apache.org/)
- [A Microsoft Azure-Jegyzetfüzeteihez](https://notebooks.azure.com/)

### <a name="power-bi"></a>Power BI

[A Power BI](/power-bi/) üzleti elemzőeszközök együttese. Adatforrások több száz csatlakozhat, és alkalmi elemzéshez használható. Lásd: [ebben a listában](/power-bi/desktop-data-sources) jelenleg elérhető adatforrást. Használjon [Power BI Embedded](https://azure.microsoft.com/services/power-bi-embedded/) integrálható a Power BI saját alkalmazásokon belül anélkül, hogy további licencelést.

A szervezetek a Power BI használatával jelentéseket készíthet, és közzéteheti a szervezet számára. Mindenki személyre szabott irányítópultokat hozhat létre az irányítás és [a beépített biztonsági](/power-bi/service-admin-power-bi-security). A Power BI által használt [Azure Active Directory](/azure/active-directory/) (az Azure AD) a Power BI-ban, és a Power BI bejelentkezési hitelesítő adatait, amikor egy felhasználó próbál hitelesítést igénylő erőforrásokhoz férnek hozzá a bejelentkező felhasználók hitelesítéséhez.

### <a name="jupyter-notebooks"></a>Jupyter-notebookok 

[Jupyter notebookok](https://jupyter.readthedocs.io/en/latest/index.html) adjon meg egy böngészőalapú felület, amely lehetővé teszi, hogy hozzon létre adatszakértőkön *notebook* Python, Scala vagy R kód és a markdown szöveg, így megosztása együttműködni hatékony módot tartalmazó fájlokat és dokumentálása kódot és egy dokumentumot eredményez.

A HDInsight-fürtök, Spark vagy Hadoop, például a legtöbb fajtáinak származnak [előre konfigurált Jupyter notebookok rendelkező](/azure/hdinsight/spark/apache-spark-jupyter-notebook-kernels) adatok való interakció, és feldolgozás feladatok elküldésekor. HDInsight-fürtöt használ típusától függően egy vagy több kernelek értelmezése és a kódja fognak adni. A HDInsight Spark-fürtjei például kiválaszthatja a Python vagy Scala kódot a Spark használata Spark kapcsolatos kernelt adja meg.

Jupyter notebookok elemzése, megjelenítésére, de az adatok, például a Power BI BI/reporting eszközzel speciális képi megjelenítés létrehozása előtt feldolgozása egy nagyszerű környezetet biztosítson.

### <a name="zeppelin-notebooks"></a>Zeppelin notebookok

[Zeppelin notebookok](https://zeppelin.apache.org/) van lehetősége, hogy egy böngészőalapú felület, funkcióinak Jupyter hasonló. Néhány HDInsight-fürtök származnak [Zeppelin notebookok az előre konfigurált](/azure/hdinsight/spark/apache-spark-zeppelin-notebook). Azonban ha használ egy [HDInsight interaktív lekérdezés](/azure/hdinsight/interactive-query/apache-interactive-query-get-started) (Hive LLAP) fürt [Zeppelin](/azure/hdinsight/hdinsight-connect-hive-zeppelin) jelenleg a notebook használó interaktív Hive-lekérdezések futtatása csak választott. Is használata egy [HDInsight-fürt tartományhoz](/azure/hdinsight/domain-joined/apache-domain-joined-introduction), Zeppelin notebookok esetén kizárólag, amely lehetővé teszi a különböző felhasználói bejelentkezések történő hozzáférés szabályozása érdekében jegyzetfüzetek és a mögöttes Hive táblák hozzárendelni.

### <a name="microsoft-azure-notebooks"></a>A Microsoft Azure-Jegyzetfüzeteihez

[Az Azure notebookok](https://notebooks.azure.com/) Jupyter notebookok-alapú egy olyan online szolgáltatás, amely lehetővé teszi az adatszakértőkön át a létrehozása, futtatása és a felhő alapú tárak Jupyter notebookok megosztásához. Azure notebookok Python 2, Python 3, F # és R végrehajtási környezetet biztosít, és megjeleníteni az adatokat, például ggplot matplotlib, bokeh vagy seaborn több diagramkészítési szalagtárak biztosít.

Ellentétben a Jupyter notebookok fut a HDInsight-fürtöt, amely kapcsolódik a fürt alapértelmezett tárfiókot, Azure notebookok nem biztosít adatokat. Meg kell [adatok betöltése](https://notebooks.azure.com/Microsoft/libraries/samples/html/Getting%20to%20your%20Data%20in%20Azure%20Notebooks.ipynb) különféle módokon, ilyen online forrásból származó adatok letöltése, Azure-BLOB és Table Storage való interakció, SQL-adatbázishoz szeretne csatlakozni, vagy másolása varázsló az adatok betöltése az Azure Data Factory.

Főbb előnyei:

* Ingyenes service&mdash;nincs Azure-előfizetés szükséges.
* Nem szükséges telepíteni a Jupyter és a támogató R vagy Python felosztás helyileg&mdash;használja a böngészőben.
* A saját online tárak, és azokat bármilyen eszközön keresztüli elérését.
* A notebookok megosztása közreműködők.

Szempontok:

* Nem érhető el kapcsolat nélküli módban jegyzetfüzetek fogjuk.
* A szabad notebook szolgáltatás korlátozott képességek nem lehet elég nagy vagy összetett modell betanításához.

## <a name="key-selection-criteria"></a>Kulcs kiválasztási feltételek

A választási lehetőségek szűkítéséhez indítása ezen kérdések megválaszolásával:

- Szüksége nagyszámú adatforrásokhoz való kapcsolódáshoz biztosít egy központi helyen, a tartomány terjed adatok jelentéseket készíteni? Ha igen, válassza ki, amely lehetővé teszi a 100-as egység adatforrások való kapcsolódáshoz.

- Szeretné a dinamikus képi megjelenítések beágyazása a külső webhely vagy alkalmazás? Ha igen, válassza ki, amely beágyazási képességeket biztosít.

- A képi megjelenítések és jelentések közben szeretné offline? Ha igen, válassza ki a kapcsolat nélküli képességekkel.

- Nagy vagy összetett AI modell betanításához vagy nagyon nagy adatkészletekkel dolgozni túl nagy feldolgozási teljesítmény szükséges? Ha igen, válassza ki, amely csatlakozni tudna a big Data típusú adatok fürt.

## <a name="capability-matrix"></a>Képesség mátrix

A következő táblázat összefoglalja a főbb változásai képességeit. 

### <a name="general-capabilities"></a>Általános képességek

| | Power BI | Jupyter-notebookok | Zeppelin notebookok | A Microsoft Azure-Jegyzetfüzeteihez |
| --- | --- | --- | --- | --- |
| Csatlakozzon a speciális feldolgozás big Data típusú adatok fürthöz | Igen | Igen | Igen | Nem |
| Felügyelt szolgáltatás | Igen | Igen <sup>1</sup> | Igen <sup>1</sup> | Igen |
| 100-as egység adatforrások kapcsolódni | Igen | Nem | Nem | Nem |
| Kapcsolat nélküli képességek | Igen <sup>2</sup> | Nem | Nem | Nem |
| Képességek beágyazás | Igen | Nem | Nem | Nem |
| Automatikus frissítés | Igen | Nem | Nem | Nem |
| Számos nyílt forráskódú csomagok elérésére | Nem | Igen <sup>3</sup> | Igen <sup>3</sup> | Igen <sup>4</sup> |
| Adatok átalakítása/tisztításokat beállításai | [Power Query](https://powerbi.microsoft.com/blog/getting-started-with-power-query-part-i/), R | 40 nyelven, beleértve a Python, R, Ágnes és Scala | 20 + értelmezők, beleértve a Python, JDBC és R | Python, F # R |
| Díjszabás | Ingyenes Power BI Desktop (szerzői) című [árképzési](https://powerbi.microsoft.com/pricing/) beállítások tárolásához | Ingyenes | Ingyenes | Ingyenes |
| Többfelhasználós együttműködés | [Igen](/power-bi/service-how-to-collaborate-distribute-dashboards-reports) | Igen (megosztása vagy hasonló többfelhasználós kiszolgálóval [JupyterHub](https://github.com/jupyterhub/jupyterhub)) | Igen | Igen (a megosztás) |

[1] Ha egy felügyelt HDInsight-fürt része.

[2] Power BI Desktop használatával.

[2] kereshet a [Maven-tárház](http://search.maven.org/) csomagok közösségi része volt.

[3] Python-csomagokat a pip vagy conda használatával is telepíthető. R csomagok CRAN vagy GitHub telepíthető. Az F # csomagok keresztül nuget.org használatával is telepíthető a [Paket függőségi manager](https://fsprojects.github.io/Paket/).

