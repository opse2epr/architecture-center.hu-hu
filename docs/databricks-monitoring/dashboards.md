---
title: Az irányítópultok használatával jelenítheti meg az Azure Databricks-metrikák
description: Hogyan helyezhet üzembe egy lesz a Grafana irányítópultja, az Azure Databricksben teljesítményének figyelése
author: petertaylor9999
ms.date: 03/26/2019
ms.openlocfilehash: dbc04b00a781dd20c3224b5a031a8d98ddadce94
ms.sourcegitcommit: 9854bd27fb5cf92041bbfb743d43045cd3552a69
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/27/2019
ms.locfileid: "58503312"
---
# <a name="use-dashboards-to-visualize-azure-databricks-metrics"></a><span data-ttu-id="787f1-103">Az irányítópultok használatával jelenítheti meg az Azure Databricks-metrikák</span><span class="sxs-lookup"><span data-stu-id="787f1-103">Use dashboards to visualize Azure Databricks metrics</span></span>

<span data-ttu-id="787f1-104">[Az Azure Databricks](/azure/azure-databricks/) van egy gyors, hatékony és együttműködő [Apache Spark](https://spark.apache.org/)– alapú analitikai szolgáltatás, amely megkönnyíti az gyors fejlesztése és üzembe helyezése a big data-analitika és a mesterséges intelligencia (AI) megoldásokat.</span><span class="sxs-lookup"><span data-stu-id="787f1-104">[Azure Databricks](/azure/azure-databricks/) is a fast, powerful, and collaborative [Apache Spark](https://spark.apache.org/)–based analytics service that makes it easy to rapidly develop and deploy big data analytics and artificial intelligence (AI) solutions.</span></span> <span data-ttu-id="787f1-105">Figyelés az Azure Databricks számítási feladatokat éles környezetben működő kritikus összetevője.</span><span class="sxs-lookup"><span data-stu-id="787f1-105">Monitoring is a critical component of operating Azure Databricks workloads in production.</span></span> <span data-ttu-id="787f1-106">Az első lépéseként a metrikák begyűjtéséhez elemzéshez a munkaterületre.</span><span class="sxs-lookup"><span data-stu-id="787f1-106">The first step is to gather metrics into a workspace for analysis.</span></span> <span data-ttu-id="787f1-107">Az Azure-ban, a legjobb megoldás a naplóadatok kezeléséhez van [Azure Monitor](/azure/azure-monitor/).</span><span class="sxs-lookup"><span data-stu-id="787f1-107">In Azure, the best solution for managing log data is [Azure Monitor](/azure/azure-monitor/).</span></span> <span data-ttu-id="787f1-108">Az Azure Databricks nem támogatja natív módon az Azure monitor történő adatküldés napló, de egy [kódtára ezt a funkciót a](https://github.com/mspnp/spark-monitoring) érhető el [Github](https://github.com).</span><span class="sxs-lookup"><span data-stu-id="787f1-108">Azure Databricks does not natively support sending log data to Azure monitor, but a [library for this functionality](https://github.com/mspnp/spark-monitoring) is available in [Github](https://github.com).</span></span>

<span data-ttu-id="787f1-109">Ebben a könyvtárban lehetővé teszi, hogy az Azure Databricks szolgáltatás metrikák, valamint az Apache Spark-struktúra lekérdezés esemény metrikák streamelési naplózását.</span><span class="sxs-lookup"><span data-stu-id="787f1-109">This library enables logging of Azure Databricks service metrics as well as Apache Spark structure streaming query event metrics.</span></span> <span data-ttu-id="787f1-110">Miután sikeresen telepítette az Azure Databricks-fürt ebben a könyvtárban, telepíthet egy további [Azure Monitor](/azure/azure-monitor/) vagy [Grafana](https://granfana.com) irányítópultokat az éles üzemben futó részeként üzembe helyezhető környezet.</span><span class="sxs-lookup"><span data-stu-id="787f1-110">Once you've successfully deployed this library to an Azure Databricks cluster, you can further deploy a set of [Azure Monitor](/azure/azure-monitor/) or [Grafana](https://granfana.com) dashboards that you can deploy as part of your production environment.</span></span> <span data-ttu-id="787f1-111">Ez a dokumentum ismerteti azokat a teljesítménnyel kapcsolatos hibákat, és hogyan azonosíthatja azokat az ilyen irányítópultok közös típusú tartalmaz.</span><span class="sxs-lookup"><span data-stu-id="787f1-111">This document includes a discussion of the common types of performance issues and how to identify them using these dashboards.</span></span>

![Az irányítópult képernyőképe](./_images/dashboard-screenshot.png)

## <a name="prequisites"></a><span data-ttu-id="787f1-113">Byok</span><span class="sxs-lookup"><span data-stu-id="787f1-113">Prequisites</span></span>

<span data-ttu-id="787f1-114">Klónozás a [Github-adattár](https://github.com/mspnp/spark-monitoring) és [az üzembe helyezési utasítások](./configure-cluster.md) létrehozása és konfigurálása az Azure Monitor naplózás az Azure Databricks szalagtár naplók elküldése az Azure Log Analytics-munkaterülethez.</span><span class="sxs-lookup"><span data-stu-id="787f1-114">Clone the [Github repository](https://github.com/mspnp/spark-monitoring) and [follow the deployment instructions](./configure-cluster.md) to build and configure the Azure Monitor logging for Azure Databricks library to send logs to your Azure Log Analytics workspace.</span></span>

## <a name="deploy-the-azure-log-analytics-workspace"></a><span data-ttu-id="787f1-115">Az Azure Log Analytics-munkaterület üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="787f1-115">Deploy the Azure Log Analytics workspace</span></span>

<span data-ttu-id="787f1-116">Az Azure Log Analytics-munkaterület üzembe helyezéséhez kövesse az alábbi lépéseket:</span><span class="sxs-lookup"><span data-stu-id="787f1-116">To deploy the Azure Log Analytics workspace, follow these steps:</span></span>

1. <span data-ttu-id="787f1-117">Keresse meg a `/perftools/deployment/loganalytics` könyvtár.</span><span class="sxs-lookup"><span data-stu-id="787f1-117">Navigate to the `/perftools/deployment/loganalytics` directory.</span></span>
1. <span data-ttu-id="787f1-118">Üzembe helyezése a **logAnalyticsDeploy.json** Azure Resource Manager-sablon.</span><span class="sxs-lookup"><span data-stu-id="787f1-118">Deploy the **logAnalyticsDeploy.json** Azure Resource Manager template.</span></span> <span data-ttu-id="787f1-119">Resource Manager-sablonok telepítésével kapcsolatos további információkért lásd: [erőforrások üzembe helyezése Resource Manager-sablonokkal és az Azure CLI-vel][rm-cli].</span><span class="sxs-lookup"><span data-stu-id="787f1-119">For more information about deploying Resource Manager templates, see [Deploy resources with Resource Manager templates and Azure CLI][rm-cli].</span></span> <span data-ttu-id="787f1-120">A sablon a következő paraméterekkel rendelkezik:</span><span class="sxs-lookup"><span data-stu-id="787f1-120">The template has the following parameters:</span></span>

    * <span data-ttu-id="787f1-121">**Hely**: A régió, ahol a Log Analytics-munkaterületet és az irányítópultok van telepítve.</span><span class="sxs-lookup"><span data-stu-id="787f1-121">**location**: The region where the Log Analytics workspace and dashboards are deployed.</span></span>
    * <span data-ttu-id="787f1-122">**serviceTier**: Rhe munkaterület tarifacsomagja.</span><span class="sxs-lookup"><span data-stu-id="787f1-122">**serviceTier**: Rhe workspace pricing tier.</span></span> <span data-ttu-id="787f1-123">Lásd: [Itt] [ sku] az érvényes értékek listáját.</span><span class="sxs-lookup"><span data-stu-id="787f1-123">See [here][sku] for a list of valid values.</span></span>
    * <span data-ttu-id="787f1-124">**dataRetention** (nem kötelező): Naplóadatok napok számát a rendszer megőrzi a Log Analytics-munkaterületen.</span><span class="sxs-lookup"><span data-stu-id="787f1-124">**dataRetention** (optional): The number of days log data is retained in the Log Analytics workspace.</span></span> <span data-ttu-id="787f1-125">Az alapértelmezett érték 30 nap.</span><span class="sxs-lookup"><span data-stu-id="787f1-125">The default value is 30 days.</span></span> <span data-ttu-id="787f1-126">Ha a tarifacsomag `Free`, az adatok megőrzése 7 napig kell lennie.</span><span class="sxs-lookup"><span data-stu-id="787f1-126">If the pricing tier is `Free`, the data retention must be 7 days.</span></span>
    * <span data-ttu-id="787f1-127">**WorkspaceName** (nem kötelező): A munkaterület nevét.</span><span class="sxs-lookup"><span data-stu-id="787f1-127">**workspaceName** (optional): A name for the workspace.</span></span> <span data-ttu-id="787f1-128">Ha nincs megadva, a sablon létrehoz egy nevet.</span><span class="sxs-lookup"><span data-stu-id="787f1-128">If not specified, the template generates a name.</span></span>

    ```bash
    az group deployment create --resource-group <resource-group-name> --template-file logAnalyticsDeploy.json --parameters location='East US' serviceTier='Standalone'
    ```

<span data-ttu-id="787f1-129">Ezzel a sablonnal hoz létre a munkaterületet, és előre meghatározott lekérdezések által irányítópult által használt létrehoz.</span><span class="sxs-lookup"><span data-stu-id="787f1-129">This template creates the workspace and also creates a set of predefined queries that are used by by dashboard.</span></span>

## <a name="deploy-grafana-in-a-virtual-machine"></a><span data-ttu-id="787f1-130">Egy virtuális gépet üzembe helyezni a Grafana</span><span class="sxs-lookup"><span data-stu-id="787f1-130">Deploy Grafana in a virtual machine</span></span>

<span data-ttu-id="787f1-131">Grafana, egy nyílt forráskódú projektje, megjelenítése az idő sorozat metrikák tárolva az Azure Log Analytics-munkaterülethez a Grafana beépülő modul az Azure Monitor telepítheti.</span><span class="sxs-lookup"><span data-stu-id="787f1-131">Grafana is an open source project you can deploy to visualize the time series metrics stored in your Azure Log Analytics workspace using the Grafana plugin for Azure Monitor.</span></span> <span data-ttu-id="787f1-132">Grafana végrehajtja a virtuális gépen (VM), és a egy tárfiókot, virtuális hálózatot és egyéb erőforrásokat igényel.</span><span class="sxs-lookup"><span data-stu-id="787f1-132">Grafana executes on a virtual machine (VM) and requires a storage account, virtual network, and other resources.</span></span> <span data-ttu-id="787f1-133">Virtuális gép üzembe helyezése a bitnami-minősítéssel rendelkező Grafana kép és a kapcsolódó erőforrásokat, kövesse az alábbi lépéseket:</span><span class="sxs-lookup"><span data-stu-id="787f1-133">To deploy a virtual machine with the bitnami certified Grafana image and associated resources, follow these steps:</span></span>

1. <span data-ttu-id="787f1-134">Az Azure Marketplace-en kép feltételek elfogadásának a Grafana az Azure CLI használatával.</span><span class="sxs-lookup"><span data-stu-id="787f1-134">Use the Azure CLI to accept the Azure Marketplace image terms for Grafana.</span></span>

    ```bash
    az vm image accept-terms --publisher bitnami --offer grafana --plan default
    ```

1. <span data-ttu-id="787f1-135">Keresse meg a `/spark-monitoring/perftools/deployment/grafana` könyvtárat a GitHub-adattár helyi példányában.</span><span class="sxs-lookup"><span data-stu-id="787f1-135">Navigate to the `/spark-monitoring/perftools/deployment/grafana` directory in your local copy of the GitHub repo.</span></span>
1. <span data-ttu-id="787f1-136">Üzembe helyezése a **grafanaDeploy.json** Resource Manager-sablont az alábbiak szerint:</span><span class="sxs-lookup"><span data-stu-id="787f1-136">Deploy the **grafanaDeploy.json** Resource Manager template as follows:</span></span>

    ```bash
    export DATA_SOURCE="https://raw.githubusercontent.com/mspnp/spark-monitoring/master/perftools/deployment/grafana/AzureDataSource.sh"
    az group deployment create \
        --resource-group <resource-group-name> \
        --template-file grafanaDeploy.json \
        --parameters adminPass='<vm password>' dataSource=$DATA_SOURCE
    ```

<span data-ttu-id="787f1-137">Az üzembe helyezés befejezése után a bitnami-rendszerkép a Grafana telepítve van a virtuális gépen.</span><span class="sxs-lookup"><span data-stu-id="787f1-137">Once the deployment is complete, the bitnami image of Grafana is installed on the virtual machine.</span></span>

## <a name="update-the-grafana-password"></a><span data-ttu-id="787f1-138">A Grafana jelszó frissítése</span><span class="sxs-lookup"><span data-stu-id="787f1-138">Update the Grafana password</span></span>

<span data-ttu-id="787f1-139">A telepítési folyamat részeként a Grafana telepítési parancsfájl kimenete egy ideiglenes jelszót a **rendszergazdai** felhasználói.</span><span class="sxs-lookup"><span data-stu-id="787f1-139">As part of the setup process, the Grafana installation script outputs a temporary password for the **admin** user.</span></span> <span data-ttu-id="787f1-140">Jelentkezzen be az ideiglenes jelszó van szüksége.</span><span class="sxs-lookup"><span data-stu-id="787f1-140">You need this temporary password to sign in.</span></span> <span data-ttu-id="787f1-141">Az ideiglenes jelszó beszerzése, kövesse az alábbi lépéseket:</span><span class="sxs-lookup"><span data-stu-id="787f1-141">To obtain the temporary password, follow these steps:</span></span>  

1. <span data-ttu-id="787f1-142">Jelentkezzen be az Azure Portalra.</span><span class="sxs-lookup"><span data-stu-id="787f1-142">Log in to the Azure portal.</span></span>  
1. <span data-ttu-id="787f1-143">Válassza ki az erőforráscsoportot, ahol az erőforrások lettek telepítve.</span><span class="sxs-lookup"><span data-stu-id="787f1-143">Select the resource group where the resources were deployed.</span></span>
1. <span data-ttu-id="787f1-144">Válassza ki a virtuális gép, ahol a Grafana telepítve van.</span><span class="sxs-lookup"><span data-stu-id="787f1-144">Select the VM where Grafana was installed.</span></span> <span data-ttu-id="787f1-145">Ha az alapértelmezett paraméter neve a központi telepítési sablont használta, a virtuális gép neve előtt a létrehozás **sparkmonitoring-vm-grafana**.</span><span class="sxs-lookup"><span data-stu-id="787f1-145">If you used the default parameter name in the deployment template, the VM name is prefaced with **sparkmonitoring-vm-grafana**.</span></span>
1. <span data-ttu-id="787f1-146">Az a **támogatás + hibaelhárítás** területén kattintson **rendszerindítási diagnosztika** a rendszerindítási Diagnosztika lap megnyitásához.</span><span class="sxs-lookup"><span data-stu-id="787f1-146">In the **Support + troubleshooting** section, click **Boot diagnostics** to open the boot diagnostics page.</span></span>
1. <span data-ttu-id="787f1-147">Kattintson a **soros napló** a rendszerindítási Diagnosztika lapon.</span><span class="sxs-lookup"><span data-stu-id="787f1-147">Click **Serial log** on the boot diagnostics page.</span></span>
1. <span data-ttu-id="787f1-148">Keresse meg a következő karakterláncot: "Bitnami alkalmazás jelszavának beállítása".</span><span class="sxs-lookup"><span data-stu-id="787f1-148">Search for the following string: "Setting Bitnami application password to".</span></span>
1. <span data-ttu-id="787f1-149">A jelszót egy biztonságos helyre másolja.</span><span class="sxs-lookup"><span data-stu-id="787f1-149">Copy the password to a safe location.</span></span>

<span data-ttu-id="787f1-150">Ezután módosítani szeretné a Grafana rendszergazdai jelszavát a következő lépésekkel:</span><span class="sxs-lookup"><span data-stu-id="787f1-150">Next, change the Grafana administrator password by following these steps:</span></span>

1. <span data-ttu-id="787f1-151">Az Azure Portalon válassza ki a virtuális Gépet, és kattintson a **áttekintése**.</span><span class="sxs-lookup"><span data-stu-id="787f1-151">In the Azure portal, select the VM and click **Overview**.</span></span>
1. <span data-ttu-id="787f1-152">Másolja le a nyilvános IP-címet.</span><span class="sxs-lookup"><span data-stu-id="787f1-152">Copy the public IP address.</span></span>
1. <span data-ttu-id="787f1-153">Nyisson meg egy webböngészőt, és keresse meg a következő URL-cím: `http://<IP addresss>:3000`.</span><span class="sxs-lookup"><span data-stu-id="787f1-153">Open a web browser and navigate to the following URL: `http://<IP addresss>:3000`.</span></span>
1. <span data-ttu-id="787f1-154">Adja meg, a Grafana bejelentkezési képernyő, **rendszergazdai** a felhasználónév és a leírt lépések Grafana jelszavát használja.</span><span class="sxs-lookup"><span data-stu-id="787f1-154">At the Grafana log in screen, enter **admin** for the user name, and use the Grafana password from the previous steps.</span></span>
1. <span data-ttu-id="787f1-155">Miután bejelentkezett, válassza ki a **konfigurációs** (a fogaskerék ikonra).</span><span class="sxs-lookup"><span data-stu-id="787f1-155">Once logged in, select **Configuration** (the gear icon).</span></span>
1. <span data-ttu-id="787f1-156">Válassza ki **kiszolgáló-rendszergazdai**.</span><span class="sxs-lookup"><span data-stu-id="787f1-156">Select **Server Admin**.</span></span>
1. <span data-ttu-id="787f1-157">Az a **felhasználók** lapon jelölje be a **rendszergazdai** bejelentkezési.</span><span class="sxs-lookup"><span data-stu-id="787f1-157">On the **Users** tab, select the **admin** login.</span></span>
1. <span data-ttu-id="787f1-158">Frissítse a jelszavát.</span><span class="sxs-lookup"><span data-stu-id="787f1-158">Update the password.</span></span>

## <a name="create-an-azure-monitor-data-source"></a><span data-ttu-id="787f1-159">Az Azure Monitor adatforrás létrehozása</span><span class="sxs-lookup"><span data-stu-id="787f1-159">Create an Azure Monitor data source</span></span>

1. <span data-ttu-id="787f1-160">A szolgáltatásnév létrehozásához, amely lehetővé teszi, hogy a Grafana a Log Analytics-munkaterületre való hozzáférés kezelése.</span><span class="sxs-lookup"><span data-stu-id="787f1-160">Create a service principal that allows Grafana to manage access to your Log Analytics workspace.</span></span> <span data-ttu-id="787f1-161">További információkért lásd: [egy Azure-beli szolgáltatásnév létrehozása az Azure CLI-vel](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest)</span><span class="sxs-lookup"><span data-stu-id="787f1-161">For more information, see [Create an Azure service principal with Azure CLI](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest)</span></span>

    ```bash
    az ad sp create-for-rbac --name http://<service principal name> --role "Log Analytics Reader"
    ```

1. <span data-ttu-id="787f1-162">Vegye figyelembe az appId, jelszót és bérlői Ez a parancs kimenetében értékeit:</span><span class="sxs-lookup"><span data-stu-id="787f1-162">Note the values for appId, password, and tenant in the output from this command:</span></span>

    ```json
    {
        "appId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "displayName": "azure-cli-2019-03-27-00-33-39",
        "name": "http://<service principal name>",
        "password": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "tenant": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    }
    ```

1. <span data-ttu-id="787f1-163">Jelentkezzen be a Grafana leírt korábban.</span><span class="sxs-lookup"><span data-stu-id="787f1-163">Log into Grafana as described earlier.</span></span> <span data-ttu-id="787f1-164">Válassza ki **konfigurációs** (a fogaskerék ikonra), majd **adatforrások**.</span><span class="sxs-lookup"><span data-stu-id="787f1-164">Select **Configuration** (the gear icon) and then **Data Sources**.</span></span>
1. <span data-ttu-id="787f1-165">Az a **adatforrások** lapra, majd **adatforrás hozzáadása**.</span><span class="sxs-lookup"><span data-stu-id="787f1-165">In the **Data Sources** tab, click **Add data source**.</span></span>
1. <span data-ttu-id="787f1-166">Válassza ki **Azure Monitor** , az adatforrás típusa.</span><span class="sxs-lookup"><span data-stu-id="787f1-166">Select **Azure Monitor** as the data source type.</span></span>
1. <span data-ttu-id="787f1-167">Az a **beállítások** területén adjon meg egy nevet az adatforrásra a **neve** szövegmezőbe.</span><span class="sxs-lookup"><span data-stu-id="787f1-167">In the **Settings** section, enter a name for the data source in the **Name** textbox.</span></span>
1. <span data-ttu-id="787f1-168">Az a **Azure Monitor API-részletek** területén adja meg a következőket:</span><span class="sxs-lookup"><span data-stu-id="787f1-168">In the **Azure Monitor API Details** section, enter the following information:</span></span>

    * <span data-ttu-id="787f1-169">Előfizetés azonosítója: Az Azure-előfizetés azonosítóját.</span><span class="sxs-lookup"><span data-stu-id="787f1-169">Subscription Id: Your Azure subscription ID.</span></span>
    * <span data-ttu-id="787f1-170">Bérlő azonosítója: A korábban a bérlő Azonosítóját.</span><span class="sxs-lookup"><span data-stu-id="787f1-170">Tenant Id: The tenant ID from earlier.</span></span>
    * <span data-ttu-id="787f1-171">Ügyfél-azonosító: A korábban "appId" értéke.</span><span class="sxs-lookup"><span data-stu-id="787f1-171">Client Id: The value of "appId" from earlier.</span></span>
    * <span data-ttu-id="787f1-172">Titkos Ügyfélkód: Korábban a "password" értékét.</span><span class="sxs-lookup"><span data-stu-id="787f1-172">Client Secret: The value of "password" from earlier.</span></span>

1. <span data-ttu-id="787f1-173">Az a **Azure Log Analytics API-részletek** négyzet a szakaszban a **azonos adatokkal az Azure Monitor API-ként** jelölőnégyzetet.</span><span class="sxs-lookup"><span data-stu-id="787f1-173">In the **Azure Log Analytics API Details** section, check the **Same Details as Azure Monitor API** checkbox.</span></span>
1. <span data-ttu-id="787f1-174">Kattintson a **mentés és tesztelés**.</span><span class="sxs-lookup"><span data-stu-id="787f1-174">Click **Save & Test**.</span></span> <span data-ttu-id="787f1-175">Ha a Log Analytics adatforrás megfelelően van konfigurálva, megjelenik a sikert jelző üzenet.</span><span class="sxs-lookup"><span data-stu-id="787f1-175">If the Log Analytics data source is correctly configured, a success message is displayed.</span></span>

## <a name="create-the-dashboard"></a><span data-ttu-id="787f1-176">Az irányítópult létrehozása</span><span class="sxs-lookup"><span data-stu-id="787f1-176">Create the dashboard</span></span>

<span data-ttu-id="787f1-177">Hozzon létre irányítópultokat a Grafana az alábbi lépéseket:</span><span class="sxs-lookup"><span data-stu-id="787f1-177">Create the dashboards in Grafana by following these steps:</span></span>

1. <span data-ttu-id="787f1-178">Keresse meg a `/perftools/dashboards/grafana` könyvtárat a GitHub-adattár helyi példányában.</span><span class="sxs-lookup"><span data-stu-id="787f1-178">Navigate to the `/perftools/dashboards/grafana` directory in your local copy of the GitHub repo.</span></span>
1. <span data-ttu-id="787f1-179">Futtassa a következő parancsfájlt:</span><span class="sxs-lookup"><span data-stu-id="787f1-179">Run the following script:</span></span>

    ```bash
    export WORKSPACE=<your Azure Log Analytics workspace ID>
    export LOGTYPE=SparkListenerEvent_CL

    sh DashGen.sh
    ```

    <span data-ttu-id="787f1-180">A szkript a kimenetét egy olyan nevű fájl **SparkMonitoringDash.json**.</span><span class="sxs-lookup"><span data-stu-id="787f1-180">The output from the script is a file named **SparkMonitoringDash.json**.</span></span>

1. <span data-ttu-id="787f1-181">Térjen vissza a lesz a Grafana irányítópultja, és válassza ki **létrehozás** (a plusz ikonra).</span><span class="sxs-lookup"><span data-stu-id="787f1-181">Return to the Grafana dashboard and select **Create** (the plus icon).</span></span>
1. <span data-ttu-id="787f1-182">Válassza ki **importálás**.</span><span class="sxs-lookup"><span data-stu-id="787f1-182">Select **Import**.</span></span>
1. <span data-ttu-id="787f1-183">Kattintson a **.JSON kiterjesztésű fájlt feltölteni**.</span><span class="sxs-lookup"><span data-stu-id="787f1-183">Click **Upload .json File**.</span></span>
1. <span data-ttu-id="787f1-184">Válassza ki a **SparkMonitoringDash.json** a 2. lépésben létrehozott fájlt.</span><span class="sxs-lookup"><span data-stu-id="787f1-184">Select the **SparkMonitoringDash.json** file created in step 2.</span></span>
1. <span data-ttu-id="787f1-185">Az a **beállítások** szakasz **ALA**, válassza ki a korábban létrehozott Azure Monitor-adatforrást.</span><span class="sxs-lookup"><span data-stu-id="787f1-185">In the **Options** section, under **ALA**, select the Azure Monitor data source created earlier.</span></span>
1. <span data-ttu-id="787f1-186">Kattintson az **Importálás** gombra.</span><span class="sxs-lookup"><span data-stu-id="787f1-186">Click **Import**.</span></span>

## <a name="visualizations-in-the-dashboards"></a><span data-ttu-id="787f1-187">Az irányítópultok vizualizációi</span><span class="sxs-lookup"><span data-stu-id="787f1-187">Visualizations in the dashboards</span></span>

<span data-ttu-id="787f1-188">Az Azure Log Analytics és a Grafana az irányítópultok tartalmaznak egy idősorozat-vizualizációkat.</span><span class="sxs-lookup"><span data-stu-id="787f1-188">Both the Azure Log Analytics and Grafana dashboards include a set of time-series visualizations.</span></span> <span data-ttu-id="787f1-189">Az egyes az idősorozat-ábrázolása a metrikaadatok egy Apache Spark kapcsolatos [feladat](https://spark.apache.org/docs/latest/job-scheduling.html), a feladat, és minden egyes szakaszhoz alkotó feladatok fázisa.</span><span class="sxs-lookup"><span data-stu-id="787f1-189">Each graph is time-series plot of metric data related to an Apache Spark [job](https://spark.apache.org/docs/latest/job-scheduling.html), stages of the job, and tasks that make up each stage.</span></span>

<span data-ttu-id="787f1-190">A Vizualizációk a következők:</span><span class="sxs-lookup"><span data-stu-id="787f1-190">The visualizations are as follows:</span></span>

### <a name="job-latency"></a><span data-ttu-id="787f1-191">Feldolgozás késéssel</span><span class="sxs-lookup"><span data-stu-id="787f1-191">Job latency</span></span>

<span data-ttu-id="787f1-192">Ezt a vizualizációt egy feladatot, amely egy durva nézet az általános hátráltató feladat végrehajtási közel valós idejű jeleníti meg.</span><span class="sxs-lookup"><span data-stu-id="787f1-192">This visualization shows execution latency for a job, which is a coarse view on the overall peformance of a job.</span></span> <span data-ttu-id="787f1-193">A feladat-végrehajtási időtartama le a befejezési jeleníti meg.</span><span class="sxs-lookup"><span data-stu-id="787f1-193">Displays the job execution duration from start to completion.</span></span> <span data-ttu-id="787f1-194">Vegye figyelembe, hogy a feladat indítási ideje nem ugyanaz, mint a feladat küldésének ideje.</span><span class="sxs-lookup"><span data-stu-id="787f1-194">Note that the job start time is not the same as the job submission time.</span></span> <span data-ttu-id="787f1-195">Késéssel jelenik meg. percentilisei (10 %-os, 30 %-os, 50 %-os, 90 %) a feladatok végrehajtásának indexeli Fürtazonosító és az alkalmazás azonosítója.</span><span class="sxs-lookup"><span data-stu-id="787f1-195">Latency is represented as percentiles (10%, 30%, 50%, 90%) of job execution indexed by cluster ID and application ID.</span></span>

### <a name="stage-latency"></a><span data-ttu-id="787f1-196">Fázis késés</span><span class="sxs-lookup"><span data-stu-id="787f1-196">Stage latency</span></span>

<span data-ttu-id="787f1-197">A Vizualizáció látható minden egyes szakaszhoz, fürtönként, alkalmazásonként és minden egyes fázis késését.</span><span class="sxs-lookup"><span data-stu-id="787f1-197">The visualization shows the latency of each stage per cluster, per application, and per individual stage.</span></span> <span data-ttu-id="787f1-198">Ezt a vizualizációt egy adott szakaszban lassan futó azonosítására szolgáló hasznos.</span><span class="sxs-lookup"><span data-stu-id="787f1-198">This visualization is useful for identifying a particular stage that is running slowly.</span></span>

### <a name="task-latency"></a><span data-ttu-id="787f1-199">A feladat késés</span><span class="sxs-lookup"><span data-stu-id="787f1-199">Task latency</span></span>

<span data-ttu-id="787f1-200">Ezt a vizualizációt a feladat-végrehajtási késést mutat.</span><span class="sxs-lookup"><span data-stu-id="787f1-200">This visualization shows task execution latency.</span></span> <span data-ttu-id="787f1-201">Késéssel jelenik meg a fürt, a fázis neve és az alkalmazás a feladat a végrehajtás egy PERCENTILIS.</span><span class="sxs-lookup"><span data-stu-id="787f1-201">Latency is represented as a percentile of task execution per cluster, stage name, and application.</span></span>

### <a name="sum-task-execution-per-host"></a><span data-ttu-id="787f1-202">A feladat a végrehajtás Sum gazdagépenként</span><span class="sxs-lookup"><span data-stu-id="787f1-202">Sum Task Execution per host</span></span>

<span data-ttu-id="787f1-203">Ezt a vizualizációt gazdagépenként egy fürtön futó feladat végrehajtási késés az összegét mutatja.</span><span class="sxs-lookup"><span data-stu-id="787f1-203">This visualization shows the sum of task execution latency per host running on a cluster.</span></span> <span data-ttu-id="787f1-204">Feladat végrehajtási késés gazdagépenként megtekintése azonosítja a gazdagépek, amelyek sokkal magasabb általános feladat késéssel rendelkeznek, mint a többi gazdáéval.</span><span class="sxs-lookup"><span data-stu-id="787f1-204">Viewing task execution latency per host identifies hosts that have much higher overall task latency than other hosts.</span></span> <span data-ttu-id="787f1-205">Ez azt jelenti, hogy feladatokat rendelkezik töredezetté vagy egyenetlenül terjesztve van a gazdagépek.</span><span class="sxs-lookup"><span data-stu-id="787f1-205">This may mean that tasks have been inefficiently or unevenly distributed to hosts.</span></span>

### <a name="task-metrics"></a><span data-ttu-id="787f1-206">A feladat-metrikák</span><span class="sxs-lookup"><span data-stu-id="787f1-206">Task metrics</span></span>

<span data-ttu-id="787f1-207">Ezt a vizualizációt egy adott feladat végrehajtásához végrehajtási metrikák mutat be.</span><span class="sxs-lookup"><span data-stu-id="787f1-207">This visualization shows a set of the execution metrics for a given task's execution.</span></span> <span data-ttu-id="787f1-208">Ezek a metrikák mérete és a egy adatok shuffle időtartama, szerializálást és deszerializálást műveletek időtartama és egyéb tartalmazza.</span><span class="sxs-lookup"><span data-stu-id="787f1-208">These metrics include the size and duration of a data shuffle, duration of serialization and deserialization operations, and others.</span></span> <span data-ttu-id="787f1-209">A metrikák teljes körű tekintse meg a Log Analytics-lekérdezés a panel.</span><span class="sxs-lookup"><span data-stu-id="787f1-209">For the full set of metrics, view the Log Analytics query for the panel.</span></span> <span data-ttu-id="787f1-210">Ezt a vizualizációt hasznos annak megértéséhez, amelyek egy feladatot és azonosító erőforrás-használat az egyes műveletek alkotják a műveletek.</span><span class="sxs-lookup"><span data-stu-id="787f1-210">This visualization is useful for understanding the operations that make up a task and identifying resource consumption of each operation.</span></span> <span data-ttu-id="787f1-211">A gráf kiugrások meg kell vizsgálni költséges műveleteket jelentik.</span><span class="sxs-lookup"><span data-stu-id="787f1-211">Spikes in the graph represent costly operations that should be investigated.</span></span>

### <a name="cluster-throughput"></a><span data-ttu-id="787f1-212">Fürt átviteli sebesség</span><span class="sxs-lookup"><span data-stu-id="787f1-212">Cluster throughput</span></span>

<span data-ttu-id="787f1-213">Ezt a vizualizációt a fürt és az alkalmazások, amelyek a fürt és az alkalmazás végzett munka mennyisége által indexelt munkaelemek magas szintű nézetét.</span><span class="sxs-lookup"><span data-stu-id="787f1-213">This visualization is a high level view of work items indexed by cluster and application to represent the amount of work done per cluster and application.</span></span> <span data-ttu-id="787f1-214">Ez a feladatok, a feladatok és a fürt, az alkalmazás és a egy perces egységekben. fázis kész szakaszok számát jeleníti meg.</span><span class="sxs-lookup"><span data-stu-id="787f1-214">It shows the number of jobs, tasks, and stages completed per cluster, application, and stage in one minute increments.</span></span> 

### <a name="streaming-throughputlatency"></a><span data-ttu-id="787f1-215">Folyamatos átviteli teljesítmény/késés</span><span class="sxs-lookup"><span data-stu-id="787f1-215">Streaming Throughput/Latency</span></span>

<span data-ttu-id="787f1-216">Ez visualzation kapcsolódik a structured streaming lekérdezéshez tartozó metrikákat.</span><span class="sxs-lookup"><span data-stu-id="787f1-216">This visualzation is related to the metrics associated with a structured streaming query.</span></span> <span data-ttu-id="787f1-217">A gráfok másodpercenkénti bemeneti sorok számát és a másodpercenként feldolgozott sorok számát mutatja.</span><span class="sxs-lookup"><span data-stu-id="787f1-217">The graphs shows the number of input rows per second and the number of rows processed per second.</span></span> <span data-ttu-id="787f1-218">A streamelési metrikákat is szerepelnek az alkalmazásonként.</span><span class="sxs-lookup"><span data-stu-id="787f1-218">The streaming metrics are also represented per application.</span></span> <span data-ttu-id="787f1-219">Ezek a metrikák elküldésére, amikor a OnQueryProgress esemény jön létre a structured streaming lekérdezés feldolgozása, és a Vizualizáció jelöli streamelési várakozási ideje, hogy mennyi idő, ezredmásodpercben végrehajtásához szükséges egy lekérdezéskötegben.</span><span class="sxs-lookup"><span data-stu-id="787f1-219">These metrics are sent when the OnQueryProgress event is generated as the structured streaming query is processed and the visualization represents streaming latency as the amount of time, in milliseconds, taken to execute a query batch.</span></span>

### <a name="resource-consumption-per-executor"></a><span data-ttu-id="787f1-220">Erőforrás-használatot végrehajtó</span><span class="sxs-lookup"><span data-stu-id="787f1-220">Resource consumption per executor</span></span>

<span data-ttu-id="787f1-221">Ezután olyan Vizualizációk, az irányítópult megjelenítése az adott típusú erőforrás, és hogyan felhasznált minden fürtön végrehajtó száma.</span><span class="sxs-lookup"><span data-stu-id="787f1-221">Next is a set of visualizations for the dashboard show the particular type of resource and how it is consumed per executor on each cluster.</span></span> <span data-ttu-id="787f1-222">Ezek a Vizualizációk segítségével végrehajtó erőforrás-használatot a kiugró értékek azonosításához.</span><span class="sxs-lookup"><span data-stu-id="787f1-222">These visualizations help identify outliers in resource consumption per executor.</span></span> <span data-ttu-id="787f1-223">Például ha egy adott végrehajtó munkahelyi lefoglalt torzítja van, erőforrás-használat lesz kell emelni a fürtben futó más végrehajtóval viszonyítva.</span><span class="sxs-lookup"><span data-stu-id="787f1-223">For example, if the work allocation for a particular executor is skewed, resource consumption will be elevated in relation to other executors running on the cluster.</span></span> <span data-ttu-id="787f1-224">Ez az erőforrás-fogyasztását-végrehajtó kiugrások azonosíthatók.</span><span class="sxs-lookup"><span data-stu-id="787f1-224">This can be identified by spikes in the resource consumption for an executor.</span></span>

### <a name="executor-compute-time-metrics"></a><span data-ttu-id="787f1-225">Végrehajtói számítási idő metrikák</span><span class="sxs-lookup"><span data-stu-id="787f1-225">Executor compute time metrics</span></span>

<span data-ttu-id="787f1-226">Ezután egy olyan vizualizációkat az irányítópulton, amely végrehajtó aránya szerializálni idő megjelenítése deszerializálni időpontja, CPU-időt, és a Java virtuális gép ideje általános végrehajtó számítási időért.</span><span class="sxs-lookup"><span data-stu-id="787f1-226">Next is a set of visualizations for the dashboard that show the ratio of executor serialize time, deserialize time, CPU time, and Java virtual machine time to overall executor compute time.</span></span> <span data-ttu-id="787f1-227">Ez azt mutatja be a négy metrika kijelölése látható legyen, hogy mennyi egyes feldolgozási általános végrehajtó működik közre.</span><span class="sxs-lookup"><span data-stu-id="787f1-227">This demonstrates visually how much each of these four metrics are contributing to overall executor processing.</span></span>

### <a name="shuffle-metrics"></a><span data-ttu-id="787f1-228">Shuffle metrikák</span><span class="sxs-lookup"><span data-stu-id="787f1-228">Shuffle metrics</span></span>

<span data-ttu-id="787f1-229">Az adatok között az összes végrehajtóval a strukturált streamelési lekérdezéshez tartozó mérőszámok shuffle Vizualizációk show végleges halmazát.</span><span class="sxs-lookup"><span data-stu-id="787f1-229">The final set of visualizations show the data shuffle metrics associated with a structured streaming query across all executors.</span></span> <span data-ttu-id="787f1-230">Ezek közé tartozik a shuffle bájt olvasás, írt bájtok sejthető, sejthető memória és a lemezhasználatot lekérdezéseket, a fájlrendszer szolgál.</span><span class="sxs-lookup"><span data-stu-id="787f1-230">These include shuffle bytes read, shuffle bytes written, shuffle memory, and disk usage in queries where the file system is used.</span></span>

<!-- links -->

[rm-cli]: /azure/azure-resource-manager/resource-group-template-deploy-cli
[sku]: /azure/templates/Microsoft.OperationalInsights/2015-11-01-preview/workspaces#sku-object