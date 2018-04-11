---
title: Frissítés Azure Resource Manager-sablonok az erőforráshoz
description: Ismerteti, hogyan lehet Azure Resource Manager sablonok egy erőforrás frissítése bővítése
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: fc2565819c66ee7695224ef5793e7276e6e552e0
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="update-a-resource-in-an-azure-resource-manager-template"></a><span data-ttu-id="67d0f-103">Frissítés Azure Resource Manager-sablonok az erőforráshoz</span><span class="sxs-lookup"><span data-stu-id="67d0f-103">Update a resource in an Azure Resource Manager template</span></span>

<span data-ttu-id="67d0f-104">Léteznek olyan forgatókönyvek, amelyben frissítenie kell egy erőforrást egy központi telepítése során.</span><span class="sxs-lookup"><span data-stu-id="67d0f-104">There are some scenarios in which you need to update a resource during a deployment.</span></span> <span data-ttu-id="67d0f-105">Ebben a forgatókönyvben használatnál erőforrás összes tulajdonság nem adható meg, amíg más, a függő erőforrások jönnek létre.</span><span class="sxs-lookup"><span data-stu-id="67d0f-105">You might encounter this scenario when you cannot specify all the properties for a resource until other, dependent resources are created.</span></span> <span data-ttu-id="67d0f-106">Terheléselosztó háttérkészletének hoz létre, előfordulhat, hogy a virtuális gépek (VM) részt a háttérkészlet például frissítse a hálózati adapterek (NIC).</span><span class="sxs-lookup"><span data-stu-id="67d0f-106">For example, if you create a backend pool for a load balancer, you might update the network interfaces (NICs) on your virtual machines (VMs) to include them in the backend pool.</span></span> <span data-ttu-id="67d0f-107">És erőforrás-kezelő frissítése erőforrások támogatja a telepítés során, amíg meg kell kialakítása a sablont megfelelően hibák elkerülése érdekében, és győződjön meg arról, a központi telepítés kezelése frissítéseként.</span><span class="sxs-lookup"><span data-stu-id="67d0f-107">And while Resource Manager supports updating resources during deployment, you must design your template correctly to avoid errors and to ensure the deployment is handled as an update.</span></span>

<span data-ttu-id="67d0f-108">Először hivatkoznia kell az erőforrás egyszer hozza létre, és majd hivatkozzon a erőforrás ugyanazzal a névvel, a frissítés később a sablonban.</span><span class="sxs-lookup"><span data-stu-id="67d0f-108">First, you must reference the resource once in the template to create it and then reference the resource by the same name to update it later.</span></span> <span data-ttu-id="67d0f-109">Azonban ha két azonos nevű sablonban, erőforrás-kezelő kivételt jelez.</span><span class="sxs-lookup"><span data-stu-id="67d0f-109">However, if two resources have the same name in a template, Resource Manager throws an exception.</span></span> <span data-ttu-id="67d0f-110">Ilyen hibák elkerülése érdekében adja meg a frissített erőforrása csatolt vagy egy subtemplate használatával néven második sablon a `Microsoft.Resources/deployments` erőforrástípus.</span><span class="sxs-lookup"><span data-stu-id="67d0f-110">To avoid this error, specify the updated resource in a second template that's either linked or included as a subtemplate using the `Microsoft.Resources/deployments` resource type.</span></span>

<span data-ttu-id="67d0f-111">Ezután meg kell adnia a nevét, a meglévő tulajdonság módosítása vagy egy új nevet a beágyazott sablon hozzáadása tulajdonsághoz.</span><span class="sxs-lookup"><span data-stu-id="67d0f-111">Second, you must either specify the name of the existing property to change or a new name for a property to add in the nested template.</span></span> <span data-ttu-id="67d0f-112">Is meg kell adnia az eredeti tulajdonságok és az eredeti értékekre is.</span><span class="sxs-lookup"><span data-stu-id="67d0f-112">You must also specify the original properties and their original values.</span></span> <span data-ttu-id="67d0f-113">Ha nem adja meg az eredeti tulajdonságok és értékek, erőforrás-kezelő azt feltételezi, hogy hozzon létre egy új erőforrást szeretne, és törli az eredeti erőforrást.</span><span class="sxs-lookup"><span data-stu-id="67d0f-113">If you fail to provide the original properties and values, Resource Manager assumes you want to create a new resource and deletes the original resource.</span></span>

## <a name="example-template"></a><span data-ttu-id="67d0f-114">Példa sablon</span><span class="sxs-lookup"><span data-stu-id="67d0f-114">Example template</span></span>

<span data-ttu-id="67d0f-115">Azt mutatja be, hogy ez egy példa sablon vizsgáljuk meg.</span><span class="sxs-lookup"><span data-stu-id="67d0f-115">Let's look at an example template that demonstrates this.</span></span> <span data-ttu-id="67d0f-116">A sablon nevű virtuális hálózat telepíti `firstVNet` nevű több alhálózattal rendelkező `firstSubnet`.</span><span class="sxs-lookup"><span data-stu-id="67d0f-116">Our template deploys a virtual network  named `firstVNet` that has one subnet named `firstSubnet`.</span></span> <span data-ttu-id="67d0f-117">Ezután telepíti a virtuális hálózati adaptert (NIC) nevű `nic1` és társítja azt az alhálózatot.</span><span class="sxs-lookup"><span data-stu-id="67d0f-117">It then deploys a virtual network interface (NIC) named `nic1` and associates it with our subnet.</span></span> <span data-ttu-id="67d0f-118">Ezt követően a központi telepítés erőforrást `updateVNet` tartalmaz egy beágyazott sablont, amely frissíti az `firstVNet` nevű második alhálózat hozzáadásával erőforrás `secondSubnet`.</span><span class="sxs-lookup"><span data-stu-id="67d0f-118">Then, a deployment resource named `updateVNet` includes a nested template that updates our `firstVNet` resource by adding a second subnet named `secondSubnet`.</span></span> 

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
          "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
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

<span data-ttu-id="67d0f-119">Vessen egy pillantást az erőforrás-objektum a `firstVNet` erőforrás első.</span><span class="sxs-lookup"><span data-stu-id="67d0f-119">Let's take a look at the resource object for our `firstVNet` resource first.</span></span> <span data-ttu-id="67d0f-120">Figyelje meg, hogy helyes-e beállításait a `firstVNet` beágyazott sablonban&mdash;ennek az az oka az erőforrás-kezelő nem engedélyezi a központi telepítés néven belül ugyanazt a sablont, és egy másik sablonba beágyazott sablonok lévőknek tekintendők.</span><span class="sxs-lookup"><span data-stu-id="67d0f-120">Notice that we respecify the settings for our `firstVNet` in a nested template&mdash;this is because Resource Manager doesn't allow the same deployment name within the same template and nested templates are considered to be a different template.</span></span> <span data-ttu-id="67d0f-121">Az értékek respecifying által a `firstSubnet` erőforrás, hogy az erőforrás-kezelő törlésével és újratelepítésével helyett a meglévő erőforrás frissítése szólítja fel.</span><span class="sxs-lookup"><span data-stu-id="67d0f-121">By respecifying our values for our `firstSubnet` resource, we are telling Resource Manager to update the existing resource instead of deleting it and redeploying it.</span></span> <span data-ttu-id="67d0f-122">Végül, az új beállítások a `secondSubnet` átveszik a frissítés során.</span><span class="sxs-lookup"><span data-stu-id="67d0f-122">Finally, our new settings for `secondSubnet` are picked up during this update.</span></span>

## <a name="try-the-template"></a><span data-ttu-id="67d0f-123">Próbálja meg a sablon</span><span class="sxs-lookup"><span data-stu-id="67d0f-123">Try the template</span></span>

<span data-ttu-id="67d0f-124">Ha azt szeretné, ez a sablon kísérletezhet, kövesse az alábbi lépéseket:</span><span class="sxs-lookup"><span data-stu-id="67d0f-124">If you would like to experiment with this template, follow these steps:</span></span>

1.  <span data-ttu-id="67d0f-125">Navigáljon az Azure portálon, válassza a  **+**  ikonra, és keresse meg a **sablon-üzembehelyezés** erőforrás írja be, és válassza ki azt.</span><span class="sxs-lookup"><span data-stu-id="67d0f-125">Go to the Azure portal, select the **+** icon, and search for the **template deployment** resource type, and select it.</span></span>
2.  <span data-ttu-id="67d0f-126">Keresse meg a **sablon-üzembehelyezés** lapon jelölje be a **létrehozása** gombra.</span><span class="sxs-lookup"><span data-stu-id="67d0f-126">Navigate to the **template deployment** page, select the **create** button.</span></span> <span data-ttu-id="67d0f-127">Ezzel a gombbal megnyithatja a **egyéni telepítési** panelen.</span><span class="sxs-lookup"><span data-stu-id="67d0f-127">This button opens the **custom deployment** blade.</span></span>
3.  <span data-ttu-id="67d0f-128">Válassza ki a **szerkesztése** ikonra.</span><span class="sxs-lookup"><span data-stu-id="67d0f-128">Select the **edit** icon.</span></span>
4.  <span data-ttu-id="67d0f-129">Az üres sablon törlésére.</span><span class="sxs-lookup"><span data-stu-id="67d0f-129">Delete the empty template.</span></span>
5.  <span data-ttu-id="67d0f-130">Másolja és illessze be a mintasablon a jobb oldali ablaktáblán.</span><span class="sxs-lookup"><span data-stu-id="67d0f-130">Copy and paste the sample template into the right pane.</span></span>
6.  <span data-ttu-id="67d0f-131">Válassza ki a **mentése** gombra.</span><span class="sxs-lookup"><span data-stu-id="67d0f-131">Select the **save** button.</span></span>
7.  <span data-ttu-id="67d0f-132">Vissza a **egyéni telepítési** panelen, de ezúttal nem néhány legördülő listák.</span><span class="sxs-lookup"><span data-stu-id="67d0f-132">You return to the **custom deployment** pane, but this time there are some drop-down list boxes.</span></span> <span data-ttu-id="67d0f-133">Jelölje ki az előfizetését, vagy hozzon létre új vagy meglévő erőforráscsoportot használni, és jelöljön ki egy helyet.</span><span class="sxs-lookup"><span data-stu-id="67d0f-133">Select your subscription, either create new or use existing resource group, and select a location.</span></span> <span data-ttu-id="67d0f-134">Tekintse meg a feltételeket és kikötéseket, és válassza ki a **elfogadom** gombra.</span><span class="sxs-lookup"><span data-stu-id="67d0f-134">Review the terms and conditions, then select the **I agree** button.</span></span>
8.  <span data-ttu-id="67d0f-135">Válassza ki a **beszerzési** gombra.</span><span class="sxs-lookup"><span data-stu-id="67d0f-135">Select the **purchase** button.</span></span>

<span data-ttu-id="67d0f-136">Miután a telepítés véget ért, nyissa meg a portálon megadott erőforráscsoport.</span><span class="sxs-lookup"><span data-stu-id="67d0f-136">Once deployment has finished, open the resource group you specified in the portal.</span></span> <span data-ttu-id="67d0f-137">Megjelenik a virtuális hálózati nevű `firstVNet` és a hálózati adapter nevű `nic1`.</span><span class="sxs-lookup"><span data-stu-id="67d0f-137">You see a virtual network named `firstVNet` and a NIC named `nic1`.</span></span> <span data-ttu-id="67d0f-138">Kattintson a `firstVNet`, majd kattintson a `subnets`.</span><span class="sxs-lookup"><span data-stu-id="67d0f-138">Click `firstVNet`, then click `subnets`.</span></span> <span data-ttu-id="67d0f-139">Megjelenik a `firstSubnet` , amely eredetileg, és megjelenik a `secondSubnet` , amely hozzá lett adva a a `updateVNet` erőforrás.</span><span class="sxs-lookup"><span data-stu-id="67d0f-139">You see the `firstSubnet` that was originally created, and you see the `secondSubnet` that was added in the `updateVNet` resource.</span></span> 

![Eredeti alhálózat és a frissített alhálózati](../_images/firstVNet-subnets.png)

<span data-ttu-id="67d0f-141">Ezután térjen vissza az erőforráscsoportot, és kattintson a `nic1` kattintson `IP configurations`.</span><span class="sxs-lookup"><span data-stu-id="67d0f-141">Then, go back to the resource group and click `nic1` then click `IP configurations`.</span></span> <span data-ttu-id="67d0f-142">Az a `IP configurations` szakaszban, a `subnet` értéke `firstSubnet (10.0.0.0/24)`.</span><span class="sxs-lookup"><span data-stu-id="67d0f-142">In the `IP configurations` section, the `subnet` is set to `firstSubnet (10.0.0.0/24)`.</span></span> 

![nic1 IP-konfigurációk beállításait](../_images/nic1-ipconfigurations.png)

<span data-ttu-id="67d0f-144">Az eredeti `firstVNet` ahelyett, hogy megújult ismételt létrehozása megtörtént.</span><span class="sxs-lookup"><span data-stu-id="67d0f-144">The original `firstVNet` has been updated instead of recreated.</span></span> <span data-ttu-id="67d0f-145">Ha `firstVNet` kellett újra létre lett hozva, `nic1` volna nem rendelhetők hozzá `firstVNet`.</span><span class="sxs-lookup"><span data-stu-id="67d0f-145">If `firstVNet` had been recreated, `nic1` would not be associated with `firstVNet`.</span></span>

## <a name="next-steps"></a><span data-ttu-id="67d0f-146">Következő lépések</span><span class="sxs-lookup"><span data-stu-id="67d0f-146">Next steps</span></span>

* <span data-ttu-id="67d0f-147">Ezzel a technikával meg van valósítva a [építőelemeket sablonprojekt](https://github.com/mspnp/template-building-blocks) és a [Azure hivatkozás architektúrák](/azure/architecture/reference-architectures/).</span><span class="sxs-lookup"><span data-stu-id="67d0f-147">This technique is implemented in the [template building blocks project](https://github.com/mspnp/template-building-blocks) and the [Azure reference architectures](/azure/architecture/reference-architectures/).</span></span> <span data-ttu-id="67d0f-148">Ezek segítségével hozzon létre egy saját architektúra, vagy telepítse a referencia-architektúrák egyikét használhatja.</span><span class="sxs-lookup"><span data-stu-id="67d0f-148">You can use these to create your own architecture or deploy one of our reference architectures.</span></span>
