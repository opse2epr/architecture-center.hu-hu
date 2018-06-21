---
title: Feltételesen telepítése Azure Resource Manager-sablonok az erőforráshoz
description: Ismerteti, hogyan lehet a paraméter értékét egy erőforrás dependending feltételesen telepítése Azure Resource Manager sablonok bővítése
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: e911e7dc41b4f71ebfaf13a00f8cdbb5b4e2578b
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
ms.locfileid: "24538393"
---
# <a name="conditionally-deploy-a-resource-in-an-azure-resource-manager-template"></a><span data-ttu-id="82f5c-103">Feltételesen telepítése Azure Resource Manager-sablonok az erőforráshoz</span><span class="sxs-lookup"><span data-stu-id="82f5c-103">Conditionally deploy a resource in an Azure Resource Manager template</span></span>

<span data-ttu-id="82f5c-104">Léteznek olyan forgatókönyvek, amelyben kell terveznie a sablon telepítéséhez egy erőforrást egy paraméterérték megtalálható-e feltétel alapján.</span><span class="sxs-lookup"><span data-stu-id="82f5c-104">There are some scenarios in which you need to design your template to deploy a resource based on a condition, such as whether or not a parameter value is present.</span></span> <span data-ttu-id="82f5c-105">A sablon például telepíthet egy virtuális hálózatot és paramétereit, és adja meg azokat a társviszony-létesítéshez más virtuális hálózatokat tartalmaz.</span><span class="sxs-lookup"><span data-stu-id="82f5c-105">For example, your template may deploy a virtual network and include parameters to specify other virtual networks for peering.</span></span> <span data-ttu-id="82f5c-106">Már nincs megadva a társviszony-létesítéshez paraméterértékeket, ha nem szeretné, hogy Resource Manager telepítéséhez a társviszony-létesítési erőforrás.</span><span class="sxs-lookup"><span data-stu-id="82f5c-106">If you've not specified any parameter values for peering, you don't want Resource Manager to deploy the peering resource.</span></span>

<span data-ttu-id="82f5c-107">Ehhez használja a [ `condition` elem] [ azure-resource-manager-condition] az erőforráskészlet a paramétertömb hosszának teszteléséhez.</span><span class="sxs-lookup"><span data-stu-id="82f5c-107">To accomplish this, use the [`condition` element][azure-resource-manager-condition] in the resource to test the length of your parameter array.</span></span> <span data-ttu-id="82f5c-108">Az érték nulla, ha vissza `false` megakadályozzák a telepítést, de minden nullánál nagyobb értéket visszaadnia `true` való központi telepítésének engedélyezése.</span><span class="sxs-lookup"><span data-stu-id="82f5c-108">If the length is zero, return `false` to prevent deployment, but for all values greater than zero return `true` to allow deployment.</span></span>

## <a name="example-template"></a><span data-ttu-id="82f5c-109">Példa sablon</span><span class="sxs-lookup"><span data-stu-id="82f5c-109">Example template</span></span>

<span data-ttu-id="82f5c-110">Azt mutatja be, hogy ez egy példa sablon vizsgáljuk meg.</span><span class="sxs-lookup"><span data-stu-id="82f5c-110">Let's look at an example template that demonstrates this.</span></span> <span data-ttu-id="82f5c-111">A sablon által a [ `condition` elem] [ azure-resource-manager-condition] a vezérlő telepítése a `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` erőforrás.</span><span class="sxs-lookup"><span data-stu-id="82f5c-111">Our template uses the [`condition` element][azure-resource-manager-condition] to control deployment of the `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` resource.</span></span> <span data-ttu-id="82f5c-112">Ezt az erőforrást hoz létre a társviszony-létesítés között két Azure virtuális hálózat ugyanabban a régióban.</span><span class="sxs-lookup"><span data-stu-id="82f5c-112">This resource creates a peering between two Azure Virtual Networks in the same region.</span></span>

<span data-ttu-id="82f5c-113">Vessen egy pillantást a sablon minden szakasza.</span><span class="sxs-lookup"><span data-stu-id="82f5c-113">Let's take a look at each section of the template.</span></span>

<span data-ttu-id="82f5c-114">A `parameters` elem definiálja nevű egyetlen paraméter `virtualNetworkPeerings`:</span><span class="sxs-lookup"><span data-stu-id="82f5c-114">The `parameters` element defines a single parameter named `virtualNetworkPeerings`:</span></span> 

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "virtualNetworkPeerings": {
      "type": "array",
      "defaultValue": []
    }
  },
```
<span data-ttu-id="82f5c-115">A `virtualNetworkPeerings` paraméter egy `array` és a következő sémával rendelkezik:</span><span class="sxs-lookup"><span data-stu-id="82f5c-115">Our `virtualNetworkPeerings` parameter is an `array` and has the following schema:</span></span>

```json
"virtualNetworkPeerings": [
    {
        "remoteVirtualNetwork": {
            "name": "my-other-virtual-network"
        },
        "allowForwardedTraffic": true,
        "allowGatewayTransit": true,
        "useRemoteGateways": false
    }
]
```

<span data-ttu-id="82f5c-116">A paraméter tulajdonságait adja meg a [társviszony-létesítési virtuális hálózatok kapcsolatos beállítások][vnet-peering-resource-schema].</span><span class="sxs-lookup"><span data-stu-id="82f5c-116">The properties in our parameter specify the [settings related to peering virtual networks][vnet-peering-resource-schema].</span></span> <span data-ttu-id="82f5c-117">A következőkhöz nyújtunk a értékek ezekhez a tulajdonságokhoz amikor azt adja meg a `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` erőforrás a `resources` szakasz:</span><span class="sxs-lookup"><span data-stu-id="82f5c-117">We'll provide the values for these properties when we specify the `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` resource in the `resources` section:</span></span>

```json
"resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2017-05-10",
      "name": "[concat('vnp-', copyIndex())]",
      "condition": "[greater(length(parameters('virtualNetworkPeerings')), 0)]",
      "dependsOn": [
        "virtualNetworks"
      ],
      "copy": {
          "name": "iterator",
          "count": "[length(variables('peerings'))]",
          "mode": "serial"
      },
      "properties": {
        "mode": "Incremental",
        "template": {
          "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {
          },
          "variables": {
          },
          "resources": [
            {
              "type": "Microsoft.Network/virtualNetworks/virtualNetworkPeerings",
              "apiVersion": "2016-06-01",
              "location": "[resourceGroup().location]",
              "name": "[variables('peerings')[copyIndex()].name]",
              "properties": "[variables('peerings')[copyIndex()].properties]"
            }
          ],
          "outputs": {
          }
        }
      }
    }
]
```
<span data-ttu-id="82f5c-118">Nincsenek elolvashatja, melyek az a sablon részében.</span><span class="sxs-lookup"><span data-stu-id="82f5c-118">There are a couple of things going on in this part of our template.</span></span> <span data-ttu-id="82f5c-119">Először a rendszerbe állítandó tényleges erőforrás egy beágyazott sablon típusú `Microsoft.Resources/deployments` , amely tartalmazza a saját sablon, amely ténylegesen telepít a `Microsoft.Network/virtualNetworks/virtualNetworkPeerings`.</span><span class="sxs-lookup"><span data-stu-id="82f5c-119">First, the actual resource being deployed is an inline template of type `Microsoft.Resources/deployments` that includes its own template that actually deploys the `Microsoft.Network/virtualNetworks/virtualNetworkPeerings`.</span></span>

<span data-ttu-id="82f5c-120">A `name` a beágyazott a sablon köszönhetően egyedi aktuális ismétléseinek számától hozzáfűzésével a `copyIndex()` előtaggal `vnp-`.</span><span class="sxs-lookup"><span data-stu-id="82f5c-120">Our `name` for the inline template is made unique by concatenating the current iteration of the `copyIndex()` with the prefix `vnp-`.</span></span> 

<span data-ttu-id="82f5c-121">A `condition` elem megadhatja, hogy az erőforrás feldolgozott, ha a `greater()` függvény eredménye `true`.</span><span class="sxs-lookup"><span data-stu-id="82f5c-121">The `condition` element specifies that our resource should be processed when the `greater()` function evaluates to `true`.</span></span> <span data-ttu-id="82f5c-122">Ide, ha tesztel a `virtualNetworkPeerings` paraméter tömb `greater()` nullánál.</span><span class="sxs-lookup"><span data-stu-id="82f5c-122">Here, we're testing if the `virtualNetworkPeerings` parameter array is `greater()` than zero.</span></span> <span data-ttu-id="82f5c-123">Ha van, az eredmény `true` és a `condition` meggyőződött.</span><span class="sxs-lookup"><span data-stu-id="82f5c-123">If it is, it evaluates to `true` and the `condition` is satisfied.</span></span> <span data-ttu-id="82f5c-124">Ellenkező esetben van `false`.</span><span class="sxs-lookup"><span data-stu-id="82f5c-124">Otherwise, it's `false`.</span></span>

<span data-ttu-id="82f5c-125">Azt adja meg a következő a `copy` hurok.</span><span class="sxs-lookup"><span data-stu-id="82f5c-125">Next, we specify our `copy` loop.</span></span> <span data-ttu-id="82f5c-126">Ez egy `serial` ciklust, amely azt jelenti, hogy a hurok sorrendben történik, az egyes erőforrások csak az utolsó Várakozás erőforrás van telepítve.</span><span class="sxs-lookup"><span data-stu-id="82f5c-126">It's a `serial` loop that means the loop is done in sequence, with each resource waiting until the last resource has been deployed.</span></span> <span data-ttu-id="82f5c-127">A `count` tulajdonság határozza meg a hurok megismétli száma.</span><span class="sxs-lookup"><span data-stu-id="82f5c-127">The `count` property specifies the number of times the loop iterates.</span></span> <span data-ttu-id="82f5c-128">Itt, általában állapotúra kell állítani az eltelt a `virtualNetworkPeerings` tömb, mert azt telepíteni kívánja az erőforrást meghatározó paraméter-objektumokat.</span><span class="sxs-lookup"><span data-stu-id="82f5c-128">Here, normally we'd set it to the length of the `virtualNetworkPeerings` array because it contains the parameter objects specifying the resource we want to deploy.</span></span> <span data-ttu-id="82f5c-129">Azonban nézzük meg, ha érvényesítése sikertelen lesz, ha a tömb nem üres, mert a Resource Manager azt észleli, hogy azt próbál hozzáférni, amelyek nem léteznek tulajdonságok.</span><span class="sxs-lookup"><span data-stu-id="82f5c-129">However, if we do that, validation will fail if the array is empty because Resource Manager notices that we are attempting to access properties that do not exist.</span></span> <span data-ttu-id="82f5c-130">A Microsoft oldható meg, azonban.</span><span class="sxs-lookup"><span data-stu-id="82f5c-130">We can work around this, however.</span></span> <span data-ttu-id="82f5c-131">Vessen egy pillantást a változók kell:</span><span class="sxs-lookup"><span data-stu-id="82f5c-131">Let's take a look at the variables we'll need:</span></span>

```json
  "variables": {
    "workaround": {
       "true": "[parameters('virtualNetworkPeerings')]",
       "false": [{
           "name": "workaround",
           "properties": {}
       }]
     },
     "peerings": "[variables('workaround')[string(greater(length(parameters('virtualNetworkPeerings')), 0))]]"
  },
```

<span data-ttu-id="82f5c-132">A `workaround` változó tartalmazza a két tulajdonság, nevű `true` és nevű `false`.</span><span class="sxs-lookup"><span data-stu-id="82f5c-132">Our `workaround` variable includes two properties, one named `true` and one named `false`.</span></span> <span data-ttu-id="82f5c-133">A `true` tulajdonság értékének kiértékeli a `virtualNetworkPeerings` paramétertömb.</span><span class="sxs-lookup"><span data-stu-id="82f5c-133">The `true` property evaluates to the value of the `virtualNetworkPeerings` parameter array.</span></span> <span data-ttu-id="82f5c-134">A `false` tulajdonság kiértékeli, beleértve az erőforrás-kezelő vár, hogy elnevezett tulajdonságok üres objektumra&mdash;vegye figyelembe, hogy `false` van ténylegesen egy tömb, ahogy a `virtualNetworkPeerings` paraméter értéke, amely eleget tesz a érvényesítése.</span><span class="sxs-lookup"><span data-stu-id="82f5c-134">The `false` property evaluates to an empty object including the named properties that Resource Manager expects to see&mdash;note that `false` is actually an array, just as our `virtualNetworkPeerings` parameter is, which will satisfy validation.</span></span> 

<span data-ttu-id="82f5c-135">A `peerings` változó használja a `workaround` ismét vizsgálatával, ha a változó hossza a `virtualNetworkPeerings` paramétertömb érték nagyobb, mint nulla.</span><span class="sxs-lookup"><span data-stu-id="82f5c-135">Our `peerings` variable uses our `workaround` variable by once again testing if the length of the `virtualNetworkPeerings` parameter array is greater than zero.</span></span> <span data-ttu-id="82f5c-136">Ha igen, a `string` értékelődik ki `true` és a `workaround` a változó evalutes a `virtualNetworkPeerings` paramétertömb.</span><span class="sxs-lookup"><span data-stu-id="82f5c-136">If it is, the `string` evaluates to `true` and the `workaround` variable evalutes to the `virtualNetworkPeerings` parameter array.</span></span> <span data-ttu-id="82f5c-137">Ellenkező esetben az eredmény `false` és a `workaround` változó értéke az a tömb első eleme üres objektum.</span><span class="sxs-lookup"><span data-stu-id="82f5c-137">Otherwise, it evaluates to `false` and the `workaround` variable evaluates to our empty object in the first element of the array.</span></span>

<span data-ttu-id="82f5c-138">Most, hogy azt az érvényesítési probléma dolgozott, egyszerűen adható meg a központi telepítése a `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` erőforrás a beágyazott sablonban, hogy a `name` és `properties` a a `virtualNetworkPeerings` paramétertömb.</span><span class="sxs-lookup"><span data-stu-id="82f5c-138">Now that we've worked around the validation issue, we can simply specify the deployment of the `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` resource in the nested template, passing the `name` and `properties` from our `virtualNetworkPeerings` parameter array.</span></span> <span data-ttu-id="82f5c-139">Láthatja, ez csak a `template` beágyazva elem a `properties` elem az erőforrás.</span><span class="sxs-lookup"><span data-stu-id="82f5c-139">You can see this in the `template` element nested in the `properties` element of our resource.</span></span>

## <a name="next-steps"></a><span data-ttu-id="82f5c-140">Következő lépések</span><span class="sxs-lookup"><span data-stu-id="82f5c-140">Next steps</span></span>

* <span data-ttu-id="82f5c-141">Ezzel a technikával meg van valósítva a [építőelemeket sablonprojekt](https://github.com/mspnp/template-building-blocks) és a [Azure hivatkozás architektúrák](/azure/architecture/reference-architectures/).</span><span class="sxs-lookup"><span data-stu-id="82f5c-141">This technique is implemented in the [template building blocks project](https://github.com/mspnp/template-building-blocks) and the [Azure reference architectures](/azure/architecture/reference-architectures/).</span></span> <span data-ttu-id="82f5c-142">Ezek segítségével hozzon létre egy saját architektúra, vagy telepítse a referencia-architektúrák egyikét használhatja.</span><span class="sxs-lookup"><span data-stu-id="82f5c-142">You can use these to create your own architecture or deploy one of our reference architectures.</span></span>

<!-- links -->
[azure-resource-manager-condition]: /azure/azure-resource-manager/resource-group-authoring-templates#resources
[azure-resource-manager-variable]: /azure/azure-resource-manager/resource-group-authoring-templates#variables
[vnet-peering-resource-schema]: /azure/templates/microsoft.network/virtualnetworks/virtualnetworkpeerings