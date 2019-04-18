---
title: Irányítópultok használata az Azure Databricks-metrikák megjelenítésére
description: Hogyan helyezhet üzembe egy lesz a Grafana irányítópultja, az Azure Databricksben teljesítményének figyelése
author: petertaylor9999
ms.date: 03/26/2019
ms.openlocfilehash: a84203a9188848e6363a80ac455332e8f6a73cda
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/17/2019
ms.locfileid: "59640311"
---
# <a name="use-dashboards-to-visualize-azure-databricks-metrics"></a>Irányítópultok használata az Azure Databricks-metrikák megjelenítésére

Ez a cikk bemutatja, hogyan állítható be egy lesz a Grafana irányítópultja, teljesítménnyel kapcsolatos problémák az Azure Databricks-feladatok figyelése.

[Az Azure Databricks](/azure/azure-databricks/) van egy gyors, hatékony és együttműködő [Apache Spark](https://spark.apache.org/)– alapú analitikai szolgáltatás, amely megkönnyíti az gyors fejlesztése és üzembe helyezése a big data-analitika és a mesterséges intelligencia (AI) megoldásokat. Figyelés az Azure Databricks számítási feladatokat éles környezetben működő kritikus összetevője. Az első lépéseként a metrikák begyűjtéséhez elemzéshez a munkaterületre. Az Azure-ban, a legjobb megoldás a naplóadatok kezeléséhez van [Azure Monitor](/azure/azure-monitor/). Az Azure Databricks nem támogatja natív módon az Azure monitor történő adatküldés napló, de egy [kódtára ezt a funkciót a](https://github.com/mspnp/spark-monitoring) érhető el [Github](https://github.com).

Ebben a könyvtárban lehetővé teszi, hogy az Azure Databricks szolgáltatás metrikák, valamint az Apache Spark-struktúra lekérdezés esemény metrikák streamelési naplózását. Miután sikeresen telepítette az Azure Databricks-fürt ebben a könyvtárban, telepíthet egy további [Grafana](https://granfana.com) irányítópultok részeként az éles környezetben üzembe helyezhető.

![Az irányítópult képernyőképe](./_images/dashboard-screenshot.png)

## <a name="prerequisites"></a>Előfeltételek

Klónozás a [Github-adattár](https://github.com/mspnp/spark-monitoring) és [az üzembe helyezési utasítások](./configure-cluster.md) létrehozása és konfigurálása az Azure Monitor naplózás az Azure Databricks szalagtár naplók elküldése az Azure Log Analytics-munkaterülethez.

## <a name="deploy-the-azure-log-analytics-workspace"></a>Az Azure Log Analytics-munkaterület üzembe helyezése

Az Azure Log Analytics-munkaterület üzembe helyezéséhez kövesse az alábbi lépéseket:

1. Keresse meg a `/perftools/deployment/loganalytics` könyvtár.
1. Üzembe helyezése a **logAnalyticsDeploy.json** Azure Resource Manager-sablon. Resource Manager-sablonok telepítésével kapcsolatos további információkért lásd: [erőforrások üzembe helyezése Resource Manager-sablonokkal és az Azure CLI-vel][rm-cli]. A sablon a következő paraméterekkel rendelkezik:

    * **Hely**: A régió, ahol a Log Analytics-munkaterületet és az irányítópultok van telepítve.
    * **serviceTier**: Rhe munkaterület tarifacsomagja. Lásd: [Itt] [ sku] az érvényes értékek listáját.
    * **dataRetention** (nem kötelező): Naplóadatok napok számát a rendszer megőrzi a Log Analytics-munkaterületen. Az alapértelmezett érték 30 nap. Ha a tarifacsomag `Free`, az adatok megőrzése 7 napig kell lennie.
    * **WorkspaceName** (nem kötelező): A munkaterület nevét. Ha nincs megadva, a sablon létrehoz egy nevet.

    ```bash
    az group deployment create --resource-group <resource-group-name> --template-file logAnalyticsDeploy.json --parameters location='East US' serviceTier='Standalone'
    ```

Ezzel a sablonnal hoz létre a munkaterületet, és előre definiált irányítópult által használt lekérdezések létrehoz.

## <a name="deploy-grafana-in-a-virtual-machine"></a>Egy virtuális gépet üzembe helyezni a Grafana

Grafana, egy nyílt forráskódú projektje, megjelenítése az idő sorozat metrikák tárolva az Azure Log Analytics-munkaterülethez a Grafana beépülő modul az Azure Monitor telepítheti. Grafana végrehajtja a virtuális gépen (VM), és a egy tárfiókot, virtuális hálózatot és egyéb erőforrásokat igényel. Virtuális gép üzembe helyezése a bitnami-minősítéssel rendelkező Grafana kép és a kapcsolódó erőforrásokat, kövesse az alábbi lépéseket:

1. Az Azure Marketplace-en kép feltételek elfogadásának a Grafana az Azure CLI használatával.

    ```bash
    az vm image accept-terms --publisher bitnami --offer grafana --plan default
    ```

1. Keresse meg a `/spark-monitoring/perftools/deployment/grafana` könyvtárat a GitHub-adattár helyi példányában.
1. Üzembe helyezése a **grafanaDeploy.json** Resource Manager-sablont az alábbiak szerint:

    ```bash
    export DATA_SOURCE="https://raw.githubusercontent.com/mspnp/spark-monitoring/master/perftools/deployment/grafana/AzureDataSource.sh"
    az group deployment create \
        --resource-group <resource-group-name> \
        --template-file grafanaDeploy.json \
        --parameters adminPass='<vm password>' dataSource=$DATA_SOURCE
    ```

Az üzembe helyezés befejezése után a bitnami-rendszerkép a Grafana telepítve van a virtuális gépen.

## <a name="update-the-grafana-password"></a>A Grafana jelszó frissítése

A telepítési folyamat részeként a Grafana telepítési parancsfájl kimenete egy ideiglenes jelszót a **rendszergazdai** felhasználói. Jelentkezzen be az ideiglenes jelszó van szüksége. Az ideiglenes jelszó beszerzése, kövesse az alábbi lépéseket:  

1. Jelentkezzen be az Azure Portalra.  
1. Válassza ki az erőforráscsoportot, ahol az erőforrások lettek telepítve.
1. Válassza ki a virtuális gép, ahol a Grafana telepítve van. Ha az alapértelmezett paraméter neve a központi telepítési sablont használta, a virtuális gép neve előtt a létrehozás **sparkmonitoring-vm-grafana**.
1. Az a **támogatás + hibaelhárítás** területén kattintson **rendszerindítási diagnosztika** a rendszerindítási Diagnosztika lap megnyitásához.
1. Kattintson a **soros napló** a rendszerindítási Diagnosztika lapon.
1. Keresse meg a következő karakterláncot: "Bitnami alkalmazás jelszavának beállítása".
1. A jelszót egy biztonságos helyre másolja.

Ezután módosítani szeretné a Grafana rendszergazdai jelszavát a következő lépésekkel:

1. Az Azure Portalon válassza ki a virtuális Gépet, és kattintson a **áttekintése**.
1. Másolja le a nyilvános IP-címet.
1. Nyisson meg egy webböngészőt, és keresse meg a következő URL-cím: `http://<IP address>:3000`.
1. Adja meg, a Grafana bejelentkezési képernyő, **rendszergazdai** a felhasználónév és a leírt lépések Grafana jelszavát használja.
1. Miután bejelentkezett, válassza ki a **konfigurációs** (a fogaskerék ikonra).
1. Válassza ki **kiszolgáló-rendszergazdai**.
1. Az a **felhasználók** lapon jelölje be a **rendszergazdai** bejelentkezési.
1. Frissítse a jelszavát.

## <a name="create-an-azure-monitor-data-source"></a>Az Azure Monitor adatforrás létrehozása

1. A szolgáltatásnév létrehozásához, amely lehetővé teszi, hogy a Grafana a Log Analytics-munkaterületre való hozzáférés kezelése. További információkért lásd: [egy Azure-beli szolgáltatásnév létrehozása az Azure CLI-vel](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest)

    ```bash
    az ad sp create-for-rbac --name http://<service principal name> --role "Log Analytics Reader"
    ```

1. Vegye figyelembe az appId, jelszót és bérlői Ez a parancs kimenetében értékeit:

    ```json
    {
        "appId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "displayName": "azure-cli-2019-03-27-00-33-39",
        "name": "http://<service principal name>",
        "password": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "tenant": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    }
    ```

1. Jelentkezzen be a Grafana leírt korábban. Válassza ki **konfigurációs** (a fogaskerék ikonra), majd **adatforrások**.
1. Az a **adatforrások** lapra, majd **adatforrás hozzáadása**.
1. Válassza ki **Azure Monitor** , az adatforrás típusa.
1. Az a **beállítások** területén adjon meg egy nevet az adatforrásra a **neve** szövegmezőbe.
1. Az a **Azure Monitor API-részletek** területén adja meg a következőket:

    * Előfizetés azonosítója: Az Azure-előfizetés azonosítóját.
    * Bérlő azonosítója: A korábban a bérlő Azonosítóját.
    * Ügyfél-azonosító: A korábban "appId" értéke.
    * Titkos Ügyfélkód: Korábban a "password" értékét.

1. Az a **Azure Log Analytics API-részletek** négyzet a szakaszban a **azonos adatokkal az Azure Monitor API-ként** jelölőnégyzetet.
1. Kattintson a **mentés és tesztelés**. Ha a Log Analytics adatforrás megfelelően van konfigurálva, megjelenik a sikert jelző üzenet.

## <a name="create-the-dashboard"></a>Az irányítópult létrehozása

Hozzon létre irányítópultokat a Grafana az alábbi lépéseket:

1. Keresse meg a `/perftools/dashboards/grafana` könyvtárat a GitHub-adattár helyi példányában.
1. Futtassa a következő parancsfájlt:

    ```bash
    export WORKSPACE=<your Azure Log Analytics workspace ID>
    export LOGTYPE=SparkListenerEvent_CL

    sh DashGen.sh
    ```

    A szkript a kimenetét egy olyan nevű fájl **SparkMonitoringDash.json**.

1. Térjen vissza a lesz a Grafana irányítópultja, és válassza ki **létrehozás** (a plusz ikonra).
1. Válassza ki **importálás**.
1. Kattintson a **.JSON kiterjesztésű fájlt feltölteni**.
1. Válassza ki a **SparkMonitoringDash.json** a 2. lépésben létrehozott fájlt.
1. Az a **beállítások** szakasz **ALA**, válassza ki a korábban létrehozott Azure Monitor-adatforrást.
1. Kattintson az **Importálás** gombra.

## <a name="visualizations-in-the-dashboards"></a>Az irányítópultok vizualizációi

Az Azure Log Analytics és a Grafana az irányítópultok tartalmaznak egy idősorozat-vizualizációkat. Az egyes az idősorozat-ábrázolása a metrikaadatok egy Apache Spark kapcsolatos [feladat](https://spark.apache.org/docs/latest/job-scheduling.html), a feladat, és minden egyes szakaszhoz alkotó feladatok fázisa.

A Vizualizációk a következők:

### <a name="job-latency"></a>Feldolgozás késéssel

Ezt a vizualizációt egy feladatot, amely egy durva nézet általános teljesítménye a feladat végrehajtási közel valós idejű jeleníti meg. A feladat-végrehajtási időtartama le a befejezési jeleníti meg. Vegye figyelembe, hogy a feladat indítási ideje nem ugyanaz, mint a feladat küldésének ideje. Késéssel jelenik meg. percentilisei (10 %-os, 30 %-os, 50 %-os, 90 %) a feladatok végrehajtásának indexeli Fürtazonosító és az alkalmazás azonosítója.

### <a name="stage-latency"></a>Fázis késés

A Vizualizáció látható minden egyes szakaszhoz, fürtönként, alkalmazásonként és minden egyes fázis késését. Ezt a vizualizációt egy adott szakaszban lassan futó azonosítására szolgáló hasznos.

### <a name="task-latency"></a>A feladat késés

Ezt a vizualizációt a feladat-végrehajtási késést mutat. Késéssel jelenik meg a fürt, a fázis neve és az alkalmazás a feladat a végrehajtás egy PERCENTILIS.

### <a name="sum-task-execution-per-host"></a>A feladat a végrehajtás Sum gazdagépenként

Ezt a vizualizációt gazdagépenként egy fürtön futó feladat végrehajtási késés az összegét mutatja. Feladat végrehajtási késés gazdagépenként megtekintése azonosítja a gazdagépek, amelyek sokkal magasabb általános feladat késéssel rendelkeznek, mint a többi gazdáéval. Ez azt jelenti, hogy feladatokat rendelkezik töredezetté vagy egyenetlenül terjesztve van a gazdagépek.

### <a name="task-metrics"></a>A feladat-metrikák

Ezt a vizualizációt egy adott feladat végrehajtásához végrehajtási metrikák mutat be. Ezek a metrikák mérete és a egy adatok shuffle időtartama, szerializálást és deszerializálást műveletek időtartama és egyéb tartalmazza. A metrikák teljes körű tekintse meg a Log Analytics-lekérdezés a panel. Ezt a vizualizációt hasznos annak megértéséhez, amelyek egy feladatot és azonosító erőforrás-használat az egyes műveletek alkotják a műveletek. A gráf kiugrások meg kell vizsgálni költséges műveleteket jelentik.

### <a name="cluster-throughput"></a>Fürt átviteli sebesség

Ezt a vizualizációt a fürt és az alkalmazások, amelyek a fürt és az alkalmazás végzett munka mennyisége által indexelt munkaelemek magas szintű nézetét. Ez a feladatok, a feladatok és a fürt, az alkalmazás és a egy perces egységekben. fázis kész szakaszok számát jeleníti meg. 

### <a name="streaming-throughputlatency"></a>Folyamatos átviteli teljesítmény/késés

Ezt a vizualizációt a structured streaming lekérdezéshez tartozó metrikákat kapcsolódik. A gráfok másodpercenkénti bemeneti sorok számát és a másodpercenként feldolgozott sorok számát mutatja. A streamelési metrikákat is szerepelnek az alkalmazásonként. Ezek a metrikák elküldésére, amikor a OnQueryProgress esemény jön létre a structured streaming lekérdezés feldolgozása, és a Vizualizáció jelöli streamelési várakozási ideje, hogy mennyi idő, ezredmásodpercben végrehajtásához szükséges egy lekérdezéskötegben.

### <a name="resource-consumption-per-executor"></a>Erőforrás-használatot végrehajtó

Ezután olyan Vizualizációk, az irányítópult megjelenítése az adott típusú erőforrás, és hogyan felhasznált minden fürtön végrehajtó száma. Ezek a Vizualizációk segítségével végrehajtó erőforrás-használatot a kiugró értékek azonosításához. Például ha egy adott végrehajtó munkahelyi lefoglalt torzítja van, erőforrás-használat lesz kell emelni a fürtben futó más végrehajtóval viszonyítva. Ez az erőforrás-fogyasztását-végrehajtó kiugrások azonosíthatók.

### <a name="executor-compute-time-metrics"></a>Végrehajtói számítási idő metrikák

Ezután egy olyan vizualizációkat az irányítópulton, amely végrehajtó aránya szerializálni idő megjelenítése deszerializálni időpontja, CPU-időt, és a Java virtuális gép ideje általános végrehajtó számítási időért. Ez azt mutatja be a négy metrika kijelölése látható legyen, hogy mennyi egyes feldolgozási általános végrehajtó működik közre.

### <a name="shuffle-metrics"></a>Shuffle metrikák

Az adatok között az összes végrehajtóval a strukturált streamelési lekérdezéshez tartozó mérőszámok shuffle Vizualizációk show végleges halmazát. Ezek közé tartozik a shuffle bájt olvasás, írt bájtok sejthető, sejthető memória és a lemezhasználatot lekérdezéseket, a fájlrendszer szolgál.

## <a name="next-steps"></a>További lépések

> [!div class="nextstepaction"]
> [A teljesítmény szűk hibaelhárítása](./performance-troubleshooting.md)

<!-- links -->

[rm-cli]: /azure/azure-resource-manager/resource-group-template-deploy-cli
[sku]: /azure/templates/Microsoft.OperationalInsights/2015-11-01-preview/workspaces#sku-object