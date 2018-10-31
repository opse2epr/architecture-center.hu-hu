---
title: Erőforrás feltételes üzembe helyezése az Azure Resource Manager-sablon
description: Ismerteti, hogyan lehet feltételes üzembe egy erőforrás dependending paraméter értékét az Azure Resource Manager-sablonok bővítése
author: petertay
ms.date: 10/30/2018
ms.openlocfilehash: 2c74e17a5f38f9225b696640a23b55b1285276bb
ms.sourcegitcommit: e9eb2b895037da0633ef3ccebdea2fcce047620f
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 10/30/2018
ms.locfileid: "50251838"
---
# <a name="conditionally-deploy-a-resource-in-an-azure-resource-manager-template"></a><span data-ttu-id="85fcd-103">Erőforrás feltételes üzembe helyezése az Azure Resource Manager-sablon</span><span class="sxs-lookup"><span data-stu-id="85fcd-103">Conditionally deploy a resource in an Azure Resource Manager template</span></span>

<span data-ttu-id="85fcd-104">Van néhány olyan forgatókönyvet, amelyben kell terveznie a sablon üzembe helyezéséhez egy erőforrást, egy feltételt, például a jelen-e a paraméter értéke alapján.</span><span class="sxs-lookup"><span data-stu-id="85fcd-104">There are some scenarios in which you need to design your template to deploy a resource based on a condition, such as whether or not a parameter value is present.</span></span> <span data-ttu-id="85fcd-105">A sablon például üzembe helyezése egy virtuális hálózatot és tartalmazza a paraméterek használatával adja meg a más virtuális hálózatok társviszony-létesítéshez.</span><span class="sxs-lookup"><span data-stu-id="85fcd-105">For example, your template may deploy a virtual network and include parameters to specify other virtual networks for peering.</span></span> <span data-ttu-id="85fcd-106">Ha már nem a társviszony-létesítéshez paraméterértékeket, a társviszony-létesítési erőforrások üzembe helyezése erőforrás-kezelő nem szeretné.</span><span class="sxs-lookup"><span data-stu-id="85fcd-106">If you've not specified any parameter values for peering, you don't want Resource Manager to deploy the peering resource.</span></span>

<span data-ttu-id="85fcd-107">Ehhez használja a [feltétel elem] [ azure-resource-manager-condition] teszteléséhez a paramétertömböt hossza az erőforrásban.</span><span class="sxs-lookup"><span data-stu-id="85fcd-107">To accomplish this, use the [condition element][azure-resource-manager-condition] in the resource to test the length of your parameter array.</span></span> <span data-ttu-id="85fcd-108">Ha az érték nulla, lépjen vissza `false` megakadályozzák a telepítést, de az összes nullánál nagyobb értéket ad vissza `true` való központi telepítésének engedélyezése.</span><span class="sxs-lookup"><span data-stu-id="85fcd-108">If the length is zero, return `false` to prevent deployment, but for all values greater than zero return `true` to allow deployment.</span></span>

## <a name="example-template"></a><span data-ttu-id="85fcd-109">Példasablon</span><span class="sxs-lookup"><span data-stu-id="85fcd-109">Example template</span></span>

<span data-ttu-id="85fcd-110">Lássunk erre egy példa sablon azt mutatja be, ezzel.</span><span class="sxs-lookup"><span data-stu-id="85fcd-110">Let's look at an example template that demonstrates this.</span></span> <span data-ttu-id="85fcd-111">A sablon által a [feltétel elem] [ azure-resource-manager-condition] a vezérlő üzembe helyezéséhez a `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` erőforrás.</span><span class="sxs-lookup"><span data-stu-id="85fcd-111">Our template uses the [condition element][azure-resource-manager-condition] to control deployment of the `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` resource.</span></span> <span data-ttu-id="85fcd-112">Ezt az erőforrást hoz létre, egy társviszony-létesítés két Azure virtuális hálózat ugyanabban a régióban között.</span><span class="sxs-lookup"><span data-stu-id="85fcd-112">This resource creates a peering between two Azure Virtual Networks in the same region.</span></span>

<span data-ttu-id="85fcd-113">Vessünk egy pillantást a sablon minden szakasza.</span><span class="sxs-lookup"><span data-stu-id="85fcd-113">Let's take a look at each section of the template.</span></span>

<span data-ttu-id="85fcd-114">A `parameters` elem definiálja nevű egyetlen paraméter `virtualNetworkPeerings`:</span><span class="sxs-lookup"><span data-stu-id="85fcd-114">The `parameters` element defines a single parameter named `virtualNetworkPeerings`:</span></span> 

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
<span data-ttu-id="85fcd-115">A `virtualNetworkPeerings` paraméter egy `array` , és rendelkezik a következő mintát követik:</span><span class="sxs-lookup"><span data-stu-id="85fcd-115">Our `virtualNetworkPeerings` parameter is an `array` and has the following schema:</span></span>

```json
"virtualNetworkPeerings": [
    {
      "name": "firstVNet/peering1",
      "properties": {
          "remoteVirtualNetwork": {
              "id": "[resourceId('Microsoft.Network/virtualNetworks','secondVNet')]"
          },
          "allowForwardedTraffic": true,
          "allowGatewayTransit": true,
          "useRemoteGateways": false
      }
    }
]
```

<span data-ttu-id="85fcd-116">A tulajdonságok a paraméterben adja meg a [társviszony-létesítés virtuális hálózatok kapcsolatos beállítások][vnet-peering-resource-schema].</span><span class="sxs-lookup"><span data-stu-id="85fcd-116">The properties in our parameter specify the [settings related to peering virtual networks][vnet-peering-resource-schema].</span></span> <span data-ttu-id="85fcd-117">Biztosítunk, amellyel az értékeket ezekhez a tulajdonságokhoz adja meg, hogy amikor a `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` erőforrást a `resources` szakaszban:</span><span class="sxs-lookup"><span data-stu-id="85fcd-117">We'll provide the values for these properties when we specify the `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` resource in the `resources` section:</span></span>

```json
"resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2017-05-10",
      "name": "[concat('vnp-', copyIndex())]",
      "condition": "[greater(length(parameters('virtualNetworkPeerings')), 0)]",
      "dependsOn": [
        "firstVNet", "secondVNet"
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
<span data-ttu-id="85fcd-118">Van néhány dolog bevezetése a sablont ezen részében.</span><span class="sxs-lookup"><span data-stu-id="85fcd-118">There are a couple of things going on in this part of our template.</span></span> <span data-ttu-id="85fcd-119">Először a tényleges erőforrás üzembe-e egy beágyazott sablon típusú `Microsoft.Resources/deployments` , amely tartalmazza a saját sablont, amely Végezetül ténylegesen telepíti a `Microsoft.Network/virtualNetworks/virtualNetworkPeerings`.</span><span class="sxs-lookup"><span data-stu-id="85fcd-119">First, the actual resource being deployed is an inline template of type `Microsoft.Resources/deployments` that includes its own template that actually deploys the `Microsoft.Network/virtualNetworks/virtualNetworkPeerings`.</span></span>

<span data-ttu-id="85fcd-120">A `name` a beágyazott sablont készítette egyedi az aktuális ismétlése összetűzésének a `copyIndex()` előtagot `vnp-`.</span><span class="sxs-lookup"><span data-stu-id="85fcd-120">Our `name` for the inline template is made unique by concatenating the current iteration of the `copyIndex()` with the prefix `vnp-`.</span></span> 

<span data-ttu-id="85fcd-121">A `condition` elem azt határozza meg, hogy legyen-e az erőforrás feldolgozott mikor a `greater()` függvény `true`.</span><span class="sxs-lookup"><span data-stu-id="85fcd-121">The `condition` element specifies that our resource should be processed when the `greater()` function evaluates to `true`.</span></span> <span data-ttu-id="85fcd-122">Itt, ha tesztel a `virtualNetworkPeerings` paraméternek tömb `greater()` , mint nulla.</span><span class="sxs-lookup"><span data-stu-id="85fcd-122">Here, we're testing if the `virtualNetworkPeerings` parameter array is `greater()` than zero.</span></span> <span data-ttu-id="85fcd-123">Ha igen, az eredmény `true` és a `condition` teljesült.</span><span class="sxs-lookup"><span data-stu-id="85fcd-123">If it is, it evaluates to `true` and the `condition` is satisfied.</span></span> <span data-ttu-id="85fcd-124">Ellenkező esetben rendelkezik `false`.</span><span class="sxs-lookup"><span data-stu-id="85fcd-124">Otherwise, it's `false`.</span></span>

<span data-ttu-id="85fcd-125">Ezután azt adja meg a `copy` ciklus.</span><span class="sxs-lookup"><span data-stu-id="85fcd-125">Next, we specify our `copy` loop.</span></span> <span data-ttu-id="85fcd-126">Ez egy `serial` ciklust, amely azt jelenti, hogy a hurok sorrendben történik, az egyes erőforrások csak az utolsó Várakozás az erőforrás üzembe helyezve.</span><span class="sxs-lookup"><span data-stu-id="85fcd-126">It's a `serial` loop that means the loop is done in sequence, with each resource waiting until the last resource has been deployed.</span></span> <span data-ttu-id="85fcd-127">A `count` tulajdonság határozza meg, hogy hányszor a hurok ismétlődik.</span><span class="sxs-lookup"><span data-stu-id="85fcd-127">The `count` property specifies the number of times the loop iterates.</span></span> <span data-ttu-id="85fcd-128">Itt, általában azt értékre kell állítania azt a hosszát a `virtualNetworkPeerings` tömböt, mert a konfigurációsparaméter-objektumokat adja meg a telepíteni kívánt erőforrást tartalmaz.</span><span class="sxs-lookup"><span data-stu-id="85fcd-128">Here, normally we'd set it to the length of the `virtualNetworkPeerings` array because it contains the parameter objects specifying the resource we want to deploy.</span></span> <span data-ttu-id="85fcd-129">Azonban ha még a nem érvényesítése sikertelen lesz, ha a tömb üres, mert a Resource Manager azt észleli, hogy tudjuk elérni kívánt tulajdonságok, amelyek nem léteznek.</span><span class="sxs-lookup"><span data-stu-id="85fcd-129">However, if we do that, validation will fail if the array is empty because Resource Manager notices that we are attempting to access properties that do not exist.</span></span> <span data-ttu-id="85fcd-130">Hogy oldható meg, azonban.</span><span class="sxs-lookup"><span data-stu-id="85fcd-130">We can work around this, however.</span></span> <span data-ttu-id="85fcd-131">Vessünk egy pillantást a változókat, szüksége lesz:</span><span class="sxs-lookup"><span data-stu-id="85fcd-131">Let's take a look at the variables we'll need:</span></span>

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

<span data-ttu-id="85fcd-132">A `workaround` változó tartalmazza a két tulajdonság, egy nevű `true` és a egy nevű `false`.</span><span class="sxs-lookup"><span data-stu-id="85fcd-132">Our `workaround` variable includes two properties, one named `true` and one named `false`.</span></span> <span data-ttu-id="85fcd-133">A `true` tulajdonság kifejezés értékét adja vissza a `virtualNetworkPeerings` Pole parametru.</span><span class="sxs-lookup"><span data-stu-id="85fcd-133">The `true` property evaluates to the value of the `virtualNetworkPeerings` parameter array.</span></span> <span data-ttu-id="85fcd-134">A `false` tulajdonság egy üres objektumot, beleértve a Resource Manager vár, hogy elnevezett tulajdonságok kiértékelése &mdash; vegye figyelembe, hogy `false` van ténylegesen egy tömb, ugyanúgy, mint a `virtualNetworkPeerings` paraméter értéke, amely eleget tesz a érvényesítése.</span><span class="sxs-lookup"><span data-stu-id="85fcd-134">The `false` property evaluates to an empty object including the named properties that Resource Manager expects to see &mdash; note that `false` is actually an array, just as our `virtualNetworkPeerings` parameter is, which will satisfy validation.</span></span> 

<span data-ttu-id="85fcd-135">A `peerings` változót használ a `workaround` ismét vizsgálatával, ha a változó hossza a `virtualNetworkPeerings` Pole parametru értéke nagyobb, mint nulla.</span><span class="sxs-lookup"><span data-stu-id="85fcd-135">Our `peerings` variable uses our `workaround` variable by once again testing if the length of the `virtualNetworkPeerings` parameter array is greater than zero.</span></span> <span data-ttu-id="85fcd-136">Ha igen, a `string` értékelődik ki `true` és a `workaround` változó értéke a `virtualNetworkPeerings` Pole parametru.</span><span class="sxs-lookup"><span data-stu-id="85fcd-136">If it is, the `string` evaluates to `true` and the `workaround` variable evaluates to the `virtualNetworkPeerings` parameter array.</span></span> <span data-ttu-id="85fcd-137">Ellenkező esetben való kiértékelése által `false` és a `workaround` változó kiértékeli a tömb első eleme az üres objektumhoz.</span><span class="sxs-lookup"><span data-stu-id="85fcd-137">Otherwise, it evaluates to `false` and the `workaround` variable evaluates to our empty object in the first element of the array.</span></span>

<span data-ttu-id="85fcd-138">Most, hogy megkönnyítsük már az érvényesítési probléma, azt is egyszerűen adja meg a központi a `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` erőforrás átadása, a beágyazott sablonban a `name` és `properties` származó a `virtualNetworkPeerings` Pole parametru.</span><span class="sxs-lookup"><span data-stu-id="85fcd-138">Now that we've worked around the validation issue, we can simply specify the deployment of the `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` resource in the nested template, passing the `name` and `properties` from our `virtualNetworkPeerings` parameter array.</span></span> <span data-ttu-id="85fcd-139">Láthatja, ez az a `template` beágyazott elem a `properties` elem az erőforrás.</span><span class="sxs-lookup"><span data-stu-id="85fcd-139">You can see this in the `template` element nested in the `properties` element of our resource.</span></span>

## <a name="try-the-template"></a><span data-ttu-id="85fcd-140">A sablon kipróbálása</span><span class="sxs-lookup"><span data-stu-id="85fcd-140">Try the template</span></span>

<span data-ttu-id="85fcd-141">Egy példa sablon érhető el az [GitHub][github].</span><span class="sxs-lookup"><span data-stu-id="85fcd-141">An example template is available on [GitHub][github].</span></span> <span data-ttu-id="85fcd-142">A sablon üzembe helyezéséhez futtassa a következő [Azure CLI-vel] [ cli] parancsokat:</span><span class="sxs-lookup"><span data-stu-id="85fcd-142">To deploy the template, run the following [Azure CLI][cli] commands:</span></span>

```bash
az group create --location <location> --name <resource-group-name>
az group deployment create -g <resource-group-name> \
    --template-uri https://raw.githubusercontent.com/mspnp/template-examples/master/example2-conditional/deploy.json
```

## <a name="next-steps"></a><span data-ttu-id="85fcd-143">További lépések</span><span class="sxs-lookup"><span data-stu-id="85fcd-143">Next steps</span></span>

* <span data-ttu-id="85fcd-144">Objektumok használata helyett skaláris értékek sablon paraméterekként.</span><span class="sxs-lookup"><span data-stu-id="85fcd-144">Use objects instead of scalar values as template parameters.</span></span> <span data-ttu-id="85fcd-145">Lásd: [objektum használata paraméterként az Azure Resource Manager-sablon](./objects-as-parameters.md)</span><span class="sxs-lookup"><span data-stu-id="85fcd-145">See [Use an object as a parameter in an Azure Resource Manager template](./objects-as-parameters.md)</span></span>

<!-- links -->
[azure-resource-manager-condition]: /azure/azure-resource-manager/resource-group-authoring-templates#resources
[azure-resource-manager-variable]: /azure/azure-resource-manager/resource-group-authoring-templates#variables
[vnet-peering-resource-schema]: /azure/templates/microsoft.network/virtualnetworks/virtualnetworkpeerings
[cli]: /cli/azure/?view=azure-cli-latest
[github]: https://github.com/mspnp/template-examples