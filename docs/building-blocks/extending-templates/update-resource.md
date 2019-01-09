---
title: Az Azure Resource Manager-sablon egy erőforrás frissítése
description: Ismerteti, hogyan lehet egy erőforrás frissítése az Azure Resource Manager-sablonok bővítése.
author: petertay
ms.date: 10/31/2018
ms.openlocfilehash: 927826283163b2ae45575035168d6238de98dc00
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/08/2019
ms.locfileid: "54113416"
---
# <a name="update-a-resource-in-an-azure-resource-manager-template"></a><span data-ttu-id="fbc3e-103">Az Azure Resource Manager-sablon egy erőforrás frissítése</span><span class="sxs-lookup"><span data-stu-id="fbc3e-103">Update a resource in an Azure Resource Manager template</span></span>

<span data-ttu-id="fbc3e-104">Léteznek olyan forgatókönyvek, ahol frissítenie kell egy erőforrás egy üzembe helyezés során.</span><span class="sxs-lookup"><span data-stu-id="fbc3e-104">There are some scenarios in which you need to update a resource during a deployment.</span></span> <span data-ttu-id="fbc3e-105">Ebben a forgatókönyvben egy erőforráshoz tartozó összes tulajdonság nem adható meg, amíg nem más, a függő erőforrások jönnek létre előforduló.</span><span class="sxs-lookup"><span data-stu-id="fbc3e-105">You might encounter this scenario when you cannot specify all the properties for a resource until other, dependent resources are created.</span></span> <span data-ttu-id="fbc3e-106">A terheléselosztó háttérbeli készletet hoz létre, előfordulhat, hogy a virtuális gépeken (VM), hogy felveszi a háttérkészletben például frissítse a hálózati adapterek (NIC).</span><span class="sxs-lookup"><span data-stu-id="fbc3e-106">For example, if you create a backend pool for a load balancer, you might update the network interfaces (NICs) on your virtual machines (VMs) to include them in the backend pool.</span></span> <span data-ttu-id="fbc3e-107">És a Resource Manager támogatja a frissítési erőforrások üzembe helyezése során, míg kell tervezzen a sablont megfelelően hibák elkerülése érdekében, és győződjön meg arról, a központi telepítés kezelése frissítésként.</span><span class="sxs-lookup"><span data-stu-id="fbc3e-107">And while Resource Manager supports updating resources during deployment, you must design your template correctly to avoid errors and to ensure the deployment is handled as an update.</span></span>

<span data-ttu-id="fbc3e-108">Először is hivatkoznia kell az erőforrás egyszer hozza létre, és ezután hivatkozhat ugyanazzal a névvel, azt később frissíteni az erőforrás a sablonban.</span><span class="sxs-lookup"><span data-stu-id="fbc3e-108">First, you must reference the resource once in the template to create it and then reference the resource by the same name to update it later.</span></span> <span data-ttu-id="fbc3e-109">Azonban ha a két erőforrás rendelkezik egy sablon ugyanazzal a névvel, erőforrás-kezelő kivételt jelez.</span><span class="sxs-lookup"><span data-stu-id="fbc3e-109">However, if two resources have the same name in a template, Resource Manager throws an exception.</span></span> <span data-ttu-id="fbc3e-110">Ez a hiba elkerülése érdekében adja meg a frissített erőforrás van csatolva, vagy néven egy subtemplate a második sablon a `Microsoft.Resources/deployments` erőforrástípus.</span><span class="sxs-lookup"><span data-stu-id="fbc3e-110">To avoid this error, specify the updated resource in a second template that's either linked or included as a subtemplate using the `Microsoft.Resources/deployments` resource type.</span></span>

<span data-ttu-id="fbc3e-111">A második vagy adjon meg módosítani a meglévő tulajdonság neve vagy egy tulajdonság hozzáadása a beágyazott sablon új nevét.</span><span class="sxs-lookup"><span data-stu-id="fbc3e-111">Second, you must either specify the name of the existing property to change or a new name for a property to add in the nested template.</span></span> <span data-ttu-id="fbc3e-112">Is meg kell adnia az eredeti tulajdonságok és az eredeti értékekre is.</span><span class="sxs-lookup"><span data-stu-id="fbc3e-112">You must also specify the original properties and their original values.</span></span> <span data-ttu-id="fbc3e-113">Ha nem adja meg az eredeti tulajdonságait és értékeit, a Resource Manager feltételezi, hogy hozzon létre egy új erőforrást szeretne, és törli az eredeti erőforrás.</span><span class="sxs-lookup"><span data-stu-id="fbc3e-113">If you fail to provide the original properties and values, Resource Manager assumes you want to create a new resource and deletes the original resource.</span></span>

## <a name="example-template"></a><span data-ttu-id="fbc3e-114">Példasablon</span><span class="sxs-lookup"><span data-stu-id="fbc3e-114">Example template</span></span>

<span data-ttu-id="fbc3e-115">Lássunk erre egy példa sablon azt mutatja be, ezzel.</span><span class="sxs-lookup"><span data-stu-id="fbc3e-115">Let's look at an example template that demonstrates this.</span></span> <span data-ttu-id="fbc3e-116">A sablon üzembe helyez egy nevű virtuális hálózatot `firstVNet` nevű alhálózattal rendelkező `firstSubnet`.</span><span class="sxs-lookup"><span data-stu-id="fbc3e-116">Our template deploys a virtual network named `firstVNet` that has one subnet named `firstSubnet`.</span></span> <span data-ttu-id="fbc3e-117">Ezután üzembe helyez egy virtuális hálózati adapter (NIC) nevű `nic1` , és társítja azt az alhálózatot.</span><span class="sxs-lookup"><span data-stu-id="fbc3e-117">It then deploys a virtual network interface (NIC) named `nic1` and associates it with our subnet.</span></span> <span data-ttu-id="fbc3e-118">Ezt követően egy központi telepítési erőforrás nevű `updateVNet` tartalmaz egy beágyazott sablont, amely frissíti a `firstVNet` adja hozzá egy második alhálózatot nevű erőforrás `secondSubnet`.</span><span class="sxs-lookup"><span data-stu-id="fbc3e-118">Then, a deployment resource named `updateVNet` includes a nested template that updates our `firstVNet` resource by adding a second subnet named `secondSubnet`.</span></span>

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {},
  "resources": [
      {
      "apiVersion": "2016-03-30",
      "name": "firstVNet",
      "location":"[resourceGroup().location]",
      "type": "Microsoft.Network/virtualNetworks",
      "properties": {
          "addressSpace":{"addressPrefixes": [
              "10.0.0.0/22"
          ]},
          "subnets":[
              {
                  "name":"firstSubnet",
                  "properties":{
                    "addressPrefix":"10.0.0.0/24"
                  }
              }
            ]
      }
    },
    {
        "apiVersion": "2015-06-15",
        "type":"Microsoft.Network/networkInterfaces",
        "name":"nic1",
        "location":"[resourceGroup().location]",
        "dependsOn": [
            "firstVNet"
        ],
        "properties": {
            "ipConfigurations":[
                {
                    "name":"ipconfig1",
                    "properties": {
                        "privateIPAllocationMethod":"Dynamic",
                        "subnet": {
                            "id": "[concat(resourceId('Microsoft.Network/virtualNetworks','firstVNet'),'/subnets/firstSubnet')]"
                        }
                    }
                }
            ]
        }
    },
    {
      "apiVersion": "2015-01-01",
      "type": "Microsoft.Resources/deployments",
      "name": "updateVNet",
      "dependsOn": [
          "nic1"
      ],
      "properties": {
        "mode": "Incremental",
        "parameters": {},
        "template": {
          "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {},
          "variables": {},
          "resources": [
              {
                  "apiVersion": "2016-03-30",
                  "name": "firstVNet",
                  "location":"[resourceGroup().location]",
                  "type": "Microsoft.Network/virtualNetworks",
                  "properties": {
                      "addressSpace": "[reference('firstVNet').addressSpace]",
                      "subnets":[
                          {
                              "name":"[reference('firstVNet').subnets[0].name]",
                              "properties":{
                                  "addressPrefix":"[reference('firstVNet').subnets[0].properties.addressPrefix]"
                                  }
                          },
                          {
                              "name":"secondSubnet",
                              "properties":{
                                  "addressPrefix":"10.0.1.0/24"
                                  }
                          }
                     ]
                  }
              }
          ],
          "outputs": {}
          }
        }
    }
  ],
  "outputs": {}
}
```

<span data-ttu-id="fbc3e-119">Vessünk egy pillantást az erőforrás-objektumokat a `firstVNet` erőforrás első.</span><span class="sxs-lookup"><span data-stu-id="fbc3e-119">Let's take a look at the resource object for our `firstVNet` resource first.</span></span> <span data-ttu-id="fbc3e-120">Figyelje meg, hogy helyes-e a beállításait az `firstVNet` beágyazott sablonban&mdash;ennek az az oka az erőforrás-kezelő nem engedélyezi a központi telepítési névvel belül ugyanazt a sablont, és beágyazott sablonok másik sablont kell tekinteni.</span><span class="sxs-lookup"><span data-stu-id="fbc3e-120">Notice that we respecify the settings for our `firstVNet` in a nested template&mdash;this is because Resource Manager doesn't allow the same deployment name within the same template and nested templates are considered to be a different template.</span></span> <span data-ttu-id="fbc3e-121">Az értékek respecifying által a `firstSubnet` erőforrás, azt mondjuk neki, erőforrás-kezelő törlését és újbóli üzembe helyezés helyett a meglévő erőforrás frissítése.</span><span class="sxs-lookup"><span data-stu-id="fbc3e-121">By respecifying our values for our `firstSubnet` resource, we are telling Resource Manager to update the existing resource instead of deleting it and redeploying it.</span></span> <span data-ttu-id="fbc3e-122">Végül az új beállítások a `secondSubnet` mértékének a frissítés közben.</span><span class="sxs-lookup"><span data-stu-id="fbc3e-122">Finally, our new settings for `secondSubnet` are picked up during this update.</span></span>

## <a name="try-the-template"></a><span data-ttu-id="fbc3e-123">A sablon kipróbálása</span><span class="sxs-lookup"><span data-stu-id="fbc3e-123">Try the template</span></span>

<span data-ttu-id="fbc3e-124">Egy példa sablon érhető el az [GitHub][github].</span><span class="sxs-lookup"><span data-stu-id="fbc3e-124">An example template is available on [GitHub][github].</span></span> <span data-ttu-id="fbc3e-125">A sablon üzembe helyezéséhez futtassa a következő [Azure CLI-vel] [ cli] parancsokat:</span><span class="sxs-lookup"><span data-stu-id="fbc3e-125">To deploy the template, run the following [Azure CLI][cli] commands:</span></span>

```bash
az group create --location <location> --name <resource-group-name>
az group deployment create -g <resource-group-name> \
    --template-uri https://raw.githubusercontent.com/mspnp/template-examples/master/example1-update/deploy.json
```

<span data-ttu-id="fbc3e-126">Miután a telepítés véget ért, nyissa meg az erőforráscsoportot a portálon megadott.</span><span class="sxs-lookup"><span data-stu-id="fbc3e-126">Once deployment has finished, open the resource group you specified in the portal.</span></span> <span data-ttu-id="fbc3e-127">Megjelenik egy nevű virtuális hálózatot `firstVNet` és a egy hálózati Adaptert `nic1`.</span><span class="sxs-lookup"><span data-stu-id="fbc3e-127">You see a virtual network named `firstVNet` and a NIC named `nic1`.</span></span> <span data-ttu-id="fbc3e-128">Kattintson a `firstVNet`, majd kattintson a `subnets`.</span><span class="sxs-lookup"><span data-stu-id="fbc3e-128">Click `firstVNet`, then click `subnets`.</span></span> <span data-ttu-id="fbc3e-129">Megjelenik a `firstSubnet` , amely eredetileg létrehozták, és megjelenik a `secondSubnet` hozzáadott a `updateVNet` erőforrás.</span><span class="sxs-lookup"><span data-stu-id="fbc3e-129">You see the `firstSubnet` that was originally created, and you see the `secondSubnet` that was added in the `updateVNet` resource.</span></span>

![Eredeti alhálózat és a frissített alhálózat](../_images/firstVNet-subnets.png)

<span data-ttu-id="fbc3e-131">Ezután lépjen vissza az erőforráscsoportot, és kattintson a `nic1` kattintson `IP configurations`.</span><span class="sxs-lookup"><span data-stu-id="fbc3e-131">Then, go back to the resource group and click `nic1` then click `IP configurations`.</span></span> <span data-ttu-id="fbc3e-132">Az a `IP configurations` szakaszban a `subnet` értékre van állítva `firstSubnet (10.0.0.0/24)`.</span><span class="sxs-lookup"><span data-stu-id="fbc3e-132">In the `IP configurations` section, the `subnet` is set to `firstSubnet (10.0.0.0/24)`.</span></span>

![nic1 IP-konfiguráció beállításai](../_images/nic1-ipconfigurations.png)

<span data-ttu-id="fbc3e-134">Az eredeti `firstVNet` helyett a rendszer frissítette újra létre kell hozni.</span><span class="sxs-lookup"><span data-stu-id="fbc3e-134">The original `firstVNet` has been updated instead of recreated.</span></span> <span data-ttu-id="fbc3e-135">Ha `firstVNet` kellett újból létrejött, `nic1` akkor nem kell társítani `firstVNet`.</span><span class="sxs-lookup"><span data-stu-id="fbc3e-135">If `firstVNet` had been recreated, `nic1` would not be associated with `firstVNet`.</span></span>

## <a name="next-steps"></a><span data-ttu-id="fbc3e-136">További lépések</span><span class="sxs-lookup"><span data-stu-id="fbc3e-136">Next steps</span></span>

* <span data-ttu-id="fbc3e-137">Ismerje meg, hogyan telepítsék központilag az erőforrás, például e jelen-e a paraméter értékét egy feltétel alapján.</span><span class="sxs-lookup"><span data-stu-id="fbc3e-137">Learn how deploy a resource based on a condition, such as whether a parameter value is present.</span></span> <span data-ttu-id="fbc3e-138">Lásd: [erőforrás feltételes üzembe helyezése az Azure Resource Manager-sablon](./conditional-deploy.md).</span><span class="sxs-lookup"><span data-stu-id="fbc3e-138">See [Conditionally deploy a resource in an Azure Resource Manager template](./conditional-deploy.md).</span></span>

[cli]: /cli/azure/?view=azure-cli-latest
[github]: https://github.com/mspnp/template-examples
