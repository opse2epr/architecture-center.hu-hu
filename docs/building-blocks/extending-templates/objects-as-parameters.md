---
title: Az objektum használata Azure Resource Manager-sablon egyik paraméterének
description: Ismerteti, hogyan lehet Azure Resource Manager sablonok használata az objektumok paraméterekként bővítése
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: 76f8b9d459f4ab3147b52762b7c26552ec92c7a3
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/06/2018
---
# <a name="use-an-object-as-a-parameter-in-an-azure-resource-manager-template"></a><span data-ttu-id="05c9e-103">Az objektum használata Azure Resource Manager-sablon egyik paraméterének</span><span class="sxs-lookup"><span data-stu-id="05c9e-103">Use an object as a parameter in an Azure Resource Manager template</span></span>

<span data-ttu-id="05c9e-104">Ha Ön [Azure Resource Manager-sablonok készítésének][azure-resource-manager-create-template], lehetősége van adja meg az erőforrás-tulajdonság értékeinek közvetlenül a sablonban vagy határozzon meg egy paramétert, majd adjon meg értékeket a telepítés során.</span><span class="sxs-lookup"><span data-stu-id="05c9e-104">When you [author Azure Resource Manager templates][azure-resource-manager-create-template], you can either specify resource property values directly in the template or define a parameter and provide values during deployment.</span></span> <span data-ttu-id="05c9e-105">Beleegyezik abba, hogy minden egyes tulajdonság értéke kisebb környezetekben a paraméter használható, de egy legfeljebb 255 paraméterek üzemelő példányonként.</span><span class="sxs-lookup"><span data-stu-id="05c9e-105">It's fine to use a parameter for each property value for small deployments, but there is a limit of 255 parameters per deployment.</span></span> <span data-ttu-id="05c9e-106">Miután nagyobb és összetettebb központi paraméterek kívül is futtathatja.</span><span class="sxs-lookup"><span data-stu-id="05c9e-106">Once you get to larger and more complex deployments you may run out of parameters.</span></span>

<span data-ttu-id="05c9e-107">Egy probléma megoldásához módja egy objektum használata helyett értéket paraméterként.</span><span class="sxs-lookup"><span data-stu-id="05c9e-107">One way to solve this problem is to use an object as a parameter instead of a value.</span></span> <span data-ttu-id="05c9e-108">Ehhez a paraméter a sablonban megadott, és adjon meg egyetlen érték helyett egy JSON-objektum üzembe helyezése során.</span><span class="sxs-lookup"><span data-stu-id="05c9e-108">To do this, define the parameter in your template and specify a JSON object instead of a single value during deployment.</span></span> <span data-ttu-id="05c9e-109">A paraméter használatával altulajdonságok, majd hivatkozzon a [ `parameter()` függvény] [ azure-resource-manager-functions] és pont operátort a sablonban.</span><span class="sxs-lookup"><span data-stu-id="05c9e-109">Then, reference the subproperties of the parameter using the [`parameter()` function][azure-resource-manager-functions] and dot operator in your template.</span></span>

<span data-ttu-id="05c9e-110">Vessen egy pillantást egy példa egy virtuális hálózati erőforráshoz.</span><span class="sxs-lookup"><span data-stu-id="05c9e-110">Let's take a look at an example that deploys a virtual network resource.</span></span> <span data-ttu-id="05c9e-111">Először adjuk meg a `VNetSettings` paraméter a sablon és a `type` való `object`:</span><span class="sxs-lookup"><span data-stu-id="05c9e-111">First, let's specify a `VNetSettings` parameter in our template and set the `type` to `object`:</span></span>

```json
...
"parameters": {
    "VNetSettings":{"type":"object"}
},
```
<span data-ttu-id="05c9e-112">A következő most adja meg az értékeket a `VNetSettings` objektum:</span><span class="sxs-lookup"><span data-stu-id="05c9e-112">Next, let's provide values for the `VNetSettings` object:</span></span>

> [!NOTE]
> <span data-ttu-id="05c9e-113">Paraméter értékének megadására, deploment közben, lásd: a **paraméterek** szakasza [megérteni a felépítését és Azure Resource Manager-sablonok szintaxisát][azure-resource-manager-authoring-templates].</span><span class="sxs-lookup"><span data-stu-id="05c9e-113">To learn how to provide parameter values during deploment, see the **parameters** section of [understand the structure and syntax of Azure Resource Manager templates][azure-resource-manager-authoring-templates].</span></span> 

```json
"parameters":{
    "VNetSettings":{
        "value":{
            "name":"VNet1",
            "addressPrefixes": [
                {
                    "name": "firstPrefix",
                    "addressPrefix": "10.0.0.0/22"
                }
            ],
            "subnets":[
                {
                    "name": "firstSubnet",
                    "addressPrefix": "10.0.0.0/24"
                },
                {
                    "name":"secondSubnet",
                    "addressPrefix":"10.0.1.0/24"
                }
            ]
        }
    }
}
```

<span data-ttu-id="05c9e-114">Ahogy látja, az egyetlen ténylegesen paraméterrel három altulajdonságok: `name`, `addressPrefixes`, és `subnets`.</span><span class="sxs-lookup"><span data-stu-id="05c9e-114">As you can see, our single parameter actually specifies three subproperties: `name`, `addressPrefixes`, and `subnets`.</span></span> <span data-ttu-id="05c9e-115">Ezek altulajdonságok mindegyikének vagy értékre, vagy más altulajdonságok határozza meg.</span><span class="sxs-lookup"><span data-stu-id="05c9e-115">Each of these subproperties either specifies a value or other subproperties.</span></span> <span data-ttu-id="05c9e-116">Az eredménye, hogy az egyetlen paramétert határoz meg a virtuális hálózati telepítéséhez szükséges összes értéket.</span><span class="sxs-lookup"><span data-stu-id="05c9e-116">The result is that our single parameter specifies all the values necessary to deploy our virtual network.</span></span>

<span data-ttu-id="05c9e-117">Most tegyük rendelkezik egy pillantást a sablont, tekintse meg a többi bemutatja a `VNetSettings` objektummal:</span><span class="sxs-lookup"><span data-stu-id="05c9e-117">Now let's have a look at the rest of our template to see how the `VNetSettings` object is used:</span></span>

```json
...
"resources": [
    {
        "apiVersion": "2015-06-15",
        "type": "Microsoft.Network/virtualNetworks",
        "name": "[parameters('VNetSettings').name]",
        "location":"[resourceGroup().location]",
        "properties": {
          "addressSpace":{
              "addressPrefixes": [
                "[parameters('VNetSettings').addressPrefixes[0].addressPrefix]"
              ]
          },
          "subnets":[
              {
                  "name":"[parameters('VNetSettings').subnets[0].name]",
                  "properties": {
                      "addressPrefix": "[parameters('VNetSettings').subnets[0].addressPrefix]"
                  }
              },
              {
                  "name":"[parameters('VNetSettings').subnets[1].name]",
                  "properties": {
                      "addressPrefix": "[parameters('VNetSettings').subnets[1].addressPrefix]"
                  }
              }
          ]
        }
    }
  ]
```
<span data-ttu-id="05c9e-118">Értékeit a `VNetSettings` objektum által a virtuális hálózati erőforrás használata szükséges tulajdonságot is vonatkozik a `parameters()` mindkét függvény a `[]` tömb indexelő és a pont operátort.</span><span class="sxs-lookup"><span data-stu-id="05c9e-118">The values of our `VNetSettings` object are applied to the properties required by our virtual network resource using the `parameters()` function with both the `[]` array indexer and the dot operator.</span></span> <span data-ttu-id="05c9e-119">Ezt a módszert akkor működik, ha szeretné az értékeket a paraméter objektum statikusan alkalmazása az erőforrás.</span><span class="sxs-lookup"><span data-stu-id="05c9e-119">This approach works if you just want to statically apply the values of the parameter object to the resource.</span></span> <span data-ttu-id="05c9e-120">Azonban ha a telepítés során a tulajdonságértékek tömb dinamikusan hozzárendelni kívánt használhatja egy [másolási ciklust][azure-resource-manager-create-multiple-instances].</span><span class="sxs-lookup"><span data-stu-id="05c9e-120">However, if you want to dynamically assign an array of property values during deployment you can use a [copy loop][azure-resource-manager-create-multiple-instances].</span></span> <span data-ttu-id="05c9e-121">A másolási ciklust használatához értékeket ad meg egy JSON-tömb erőforrás tulajdonság, és a másolási ciklust dinamikusan vonatkozik az értékeket az erőforrás-tulajdonságokat.</span><span class="sxs-lookup"><span data-stu-id="05c9e-121">To use a copy loop, you provide a JSON array of resource property values and the copy loop dynamically applies the values to the resource's properties.</span></span> 

<span data-ttu-id="05c9e-122">Vegye figyelembe a dinamikus módszer használatakor az egyik megoldandó probléma van.</span><span class="sxs-lookup"><span data-stu-id="05c9e-122">There is one issue to be aware of if you use the dynamic approach.</span></span> <span data-ttu-id="05c9e-123">Bemutatják a problémát, vessen egy pillantást a tulajdonságértékek tipikus tömb.</span><span class="sxs-lookup"><span data-stu-id="05c9e-123">To demonstrate the issue, let's take a look at a typical array of property values.</span></span> <span data-ttu-id="05c9e-124">Ebben a példában a tulajdonságok értékeit tárolják a változóban.</span><span class="sxs-lookup"><span data-stu-id="05c9e-124">In this example the values for our properties are stored in a variable.</span></span> <span data-ttu-id="05c9e-125">Két tudunk értesítést tömbállandó Itt&mdash;olyan gyermektartománya `firstProperty` és nevű `secondProperty`.</span><span class="sxs-lookup"><span data-stu-id="05c9e-125">Notice we have two arrays here&mdash;one named `firstProperty` and one named `secondProperty`.</span></span> 

```json
"variables": {
    "firstProperty": [
        {
            "name": "A",
            "type": "typeA"
        },
        {
            "name": "B",
            "type": "typeB"
        },
        {
            "name": "C",
            "type": "typeC"
        }
    ],
    "secondProperty": [
        "one","two", "three"
    ]
}
```

<span data-ttu-id="05c9e-126">Most vessen egy pillantást a módját a tulajdonságai, a változóban a másolási ciklust használatával érhetők el azt.</span><span class="sxs-lookup"><span data-stu-id="05c9e-126">Now let's take a look at the way we access the properties in the variable using a copy loop.</span></span>

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    ...
    "copy": {
        "name": "copyLoop1",
        "count": "[length(variables('firstProperty'))]"
    },
    ...
    "properties": {
        "name": { "value": "[variables('firstProperty')[copyIndex()].name]" },
        "type": { "value": "[variables('firstProperty')[copyIndex()].type]" },
        "number": { "value": "[variables('secondProperty')[copyIndex()]]" }
    }
}
```

<span data-ttu-id="05c9e-127">A `copyIndex()` függvény az aktuális a ciklus ismétléseinek számától másolása, és használjuk, amely index mindegyik előálló egyidejűleg.</span><span class="sxs-lookup"><span data-stu-id="05c9e-127">The `copyIndex()` function returns the current iteration of the copy loop, and we use that as an index into each of the two arrays simultaneously.</span></span>

<span data-ttu-id="05c9e-128">Ez a két tömb hosszának esetén remekül működik.</span><span class="sxs-lookup"><span data-stu-id="05c9e-128">This works fine when the two arrays are the same length.</span></span> <span data-ttu-id="05c9e-129">A probléma merül fel, ha hiba végrehajtott, és a két tömbnek különböző hosszúságú&mdash;ebben az esetben a sablon üzembe helyezése során érvényesítése sikertelen lesz.</span><span class="sxs-lookup"><span data-stu-id="05c9e-129">The issue arises if you've made a mistake and the two arrays are different lengths&mdash;in this case your template will fail validation during deployment.</span></span> <span data-ttu-id="05c9e-130">Egyetlen objektum összes tulajdonságának belefoglalja probléma elkerülheti, mert sokkal egyszerűbb, ha az érték nem hiányzik.</span><span class="sxs-lookup"><span data-stu-id="05c9e-130">You can avoid this issue by including all your properties in a single object, because it is much easier to see when a value is missing.</span></span> <span data-ttu-id="05c9e-131">Például Ismerkedjen meg a paraméter egy másik objektum mely minden eleme a `propertyObject` unióját a `firstProperty` és `secondProperty` a korábbi tömbök.</span><span class="sxs-lookup"><span data-stu-id="05c9e-131">For example, let's take a look another parameter object in which each element of the `propertyObject` array is the union of the `firstProperty` and `secondProperty` arrays from earlier.</span></span>

```json
"variables": {
    "propertyObject": [
        {
            "name": "A",
            "type": "typeA",
            "number": "one"
        },
        {
            "name": "B",
            "type": "typeB",
            "number": "two"
        },
        {
            "name": "C",
            "type": "typeC"
        }
    ]
}
```

<span data-ttu-id="05c9e-132">Figyelje meg a harmadik a tömb elemei?</span><span class="sxs-lookup"><span data-stu-id="05c9e-132">Notice the third element in the array?</span></span> <span data-ttu-id="05c9e-133">Hiányzik a `number` tulajdonságot, de a sokkal egyszerűbb, figyelje meg, hogy már nem fogadta, ha éppen szerzőként paraméterértékek ily módon.</span><span class="sxs-lookup"><span data-stu-id="05c9e-133">It's missing the `number` property, but it's much easier to notice that you've missed it when you're authoring the parameter values this way.</span></span>

## <a name="using-a-property-object-in-a-copy-loop"></a><span data-ttu-id="05c9e-134">A másolási ciklust tulajdonság objektum használatával</span><span class="sxs-lookup"><span data-stu-id="05c9e-134">Using a property object in a copy loop</span></span>

<span data-ttu-id="05c9e-135">Ez a megközelítés válik még több akkor hasznos, ha a [soros másolási ciklust] kombinálva [azure-erőforrás-kezelő-létrehozása – több], különösen a gyermek erőforrásokat üzembe helyezi.</span><span class="sxs-lookup"><span data-stu-id="05c9e-135">This approach becomes even more useful when combined with the [serial copy loop][azure-resource-manager-create-multiple], particularly for deploying child resources.</span></span> 

<span data-ttu-id="05c9e-136">Ennek vizsgáljuk meg a sablont, amely telepíti egy [hálózati biztonsági csoport (NSG)] [ nsg] két kapcsolatbiztonsági szabályait.</span><span class="sxs-lookup"><span data-stu-id="05c9e-136">To demonstrate this, let's look at a template that deploys a [network security group (NSG)][nsg] with two security rules.</span></span> 

<span data-ttu-id="05c9e-137">Vessen egy pillantást a paramétereket.</span><span class="sxs-lookup"><span data-stu-id="05c9e-137">First, let's take a look at our parameters.</span></span> <span data-ttu-id="05c9e-138">Ha úgy tekintünk, a sablon megtanulhatja, hogy azt meghatározta nevű egy paraméter `networkSecurityGroupsSettings` , amely tartalmazza egy tömb nevű `securityRules`.</span><span class="sxs-lookup"><span data-stu-id="05c9e-138">When we look at our template we'll see that we've defined one parameter named `networkSecurityGroupsSettings` that includes an array named `securityRules`.</span></span> <span data-ttu-id="05c9e-139">A tömb két JSON olyan objektumokat tartalmaz, adja meg a biztonsági szabály beállításait egy számát.</span><span class="sxs-lookup"><span data-stu-id="05c9e-139">This array contains two JSON objects that specify a number of settings for a security rule.</span></span>

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters":{ 
      "networkSecurityGroupsSettings": {
      "value": {
          "securityRules": [
            {
              "name": "RDPAllow",
              "description": "allow RDP connections",
              "direction": "Inbound",
              "priority": 100,
              "sourceAddressPrefix": "*",
              "destinationAddressPrefix": "10.0.0.0/24",
              "sourcePortRange": "*",
              "destinationPortRange": "3389",
              "access": "Allow",
              "protocol": "Tcp"
            },
            {
              "name": "HTTPAllow",
              "description": "allow HTTP connections",
              "direction": "Inbound",
              "priority": 200,
              "sourceAddressPrefix": "*",
              "destinationAddressPrefix": "10.0.1.0/24",
              "sourcePortRange": "*",
              "destinationPortRange": "80",
              "access": "Allow",
              "protocol": "Tcp"
            }
          ]
        }
      }
    }
  }
```

<span data-ttu-id="05c9e-140">Most vessen egy pillantást a sablont.</span><span class="sxs-lookup"><span data-stu-id="05c9e-140">Now let's take a look at our template.</span></span> <span data-ttu-id="05c9e-141">Az első nevű erőforrás `NSG1` az NSG-t telepíti.</span><span class="sxs-lookup"><span data-stu-id="05c9e-141">Our first resource named `NSG1` deploys the NSG.</span></span> <span data-ttu-id="05c9e-142">A második nevű erőforrás `loop-0` két funkciókat hajtja végre: első, azt `dependsOn` az NSG-t, a telepítés nem indul el, amíg `NSG1` befejeződött, és ez az első a ciklus ismétléseinek számától szekvenciális.</span><span class="sxs-lookup"><span data-stu-id="05c9e-142">Our second resource named `loop-0` performs two functions: first, it `dependsOn` the NSG so its deployment doesn't begin until `NSG1` is completed, and it is the first iteration of the sequential loop.</span></span> <span data-ttu-id="05c9e-143">A harmadik erőforrás egy beágyazott sablont, amely a biztonsági szabályok objektum az az utolsó példában látható módon a paraméterértékeket telepít.</span><span class="sxs-lookup"><span data-stu-id="05c9e-143">Our third resource is a nested template that deploys our security rules using an object for its parameter values as in the last example.</span></span>

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
      "networkSecurityGroupsSettings": {"type":"object"}
  },
  "variables": {},
  "resources": [
    {
      "apiVersion": "2015-06-15",
      "type": "Microsoft.Network/networkSecurityGroups",
      "name": "NSG1",
      "location":"[resourceGroup().location]",
      "properties": {
          "securityRules":[]
      }
    },
    {
        "apiVersion": "2015-01-01",
        "type": "Microsoft.Resources/deployments",
        "name": "loop-0",
        "dependsOn": [
            "NSG1"
        ],
        "properties": {
            "mode":"Incremental",
            "parameters":{},
            "template": {
                "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
                "contentVersion": "1.0.0.0",
                "parameters": {},
                "variables": {},
                "resources": [],
                "outputs": {}
            }
        }       
    },
    {
        "apiVersion": "2015-01-01",
        "type": "Microsoft.Resources/deployments",
        "name": "[concat('loop-', copyIndex(1))]",
        "dependsOn": [
          "[concat('loop-', copyIndex())]"
        ],
        "copy": {
          "name": "iterator",
          "count": "[length(parameters('networkSecurityGroupsSettings').securityRules)]"
        },
        "properties": {
          "mode": "Incremental",
          "template": {
            "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
            "contentVersion": "1.0.0.0",
           "parameters": {},
            "variables": {},
            "resources": [
                {
                    "name": "[concat('NSG1/' , parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].name)]",
                    "type": "Microsoft.Network/networkSecurityGroups/securityRules",
                    "apiVersion": "2016-09-01",
                    "location":"[resourceGroup().location]",
                    "properties":{
                        "description": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].description]",
                        "priority":"[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].priority]",
                        "protocol":"[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].protocol]",
                        "sourcePortRange": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].sourcePortRange]",
                        "destinationPortRange": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].destinationPortRange]",
                        "sourceAddressPrefix": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].sourceAddressPrefix]",
                        "destinationAddressPrefix": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].destinationAddressPrefix]",
                        "access":"[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].access]",
                        "direction":"[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].direction]"
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

<span data-ttu-id="05c9e-144">Megtudhatja, hogy hogyan adtuk meg, hogy a tulajdonság értékek részletes bemutatása a `securityRules` gyermek-erőforrás.</span><span class="sxs-lookup"><span data-stu-id="05c9e-144">Let's take a closer look at how we specify our property values in the `securityRules` child resource.</span></span> <span data-ttu-id="05c9e-145">A tulajdonságokat hivatkozott használatával a `parameter()` függvény, majd azt a pontot operátor használata hivatkozik a `securityRules` tömb indexelik ismétléseinek aktuális értéke.</span><span class="sxs-lookup"><span data-stu-id="05c9e-145">All of our properties are referenced using the `parameter()` function, and then we use the dot operator to reference our `securityRules` array, indexed by the current value of the iteration.</span></span> <span data-ttu-id="05c9e-146">Végezetül használatával egy másik pont operátor hivatkoznak az objektum neve.</span><span class="sxs-lookup"><span data-stu-id="05c9e-146">Finally, we use another dot operator to reference the name of the object.</span></span> 

## <a name="try-the-template"></a><span data-ttu-id="05c9e-147">Próbálja meg a sablon</span><span class="sxs-lookup"><span data-stu-id="05c9e-147">Try the template</span></span>

<span data-ttu-id="05c9e-148">Ha azt szeretné, ez a sablon kísérletezhet, kövesse az alábbi lépéseket:</span><span class="sxs-lookup"><span data-stu-id="05c9e-148">If you would like to experiment with this template, follow these steps:</span></span> 

1.  <span data-ttu-id="05c9e-149">Navigáljon az Azure portálon, válassza a **+** ikonra, és keresse meg a **sablon-üzembehelyezés** erőforrás írja be, és válassza ki azt.</span><span class="sxs-lookup"><span data-stu-id="05c9e-149">Go to the Azure portal, select the **+** icon, and search for the **template deployment** resource type, and select it.</span></span>
2.  <span data-ttu-id="05c9e-150">Keresse meg a **sablon-üzembehelyezés** lapon jelölje be a **létrehozása** gombra.</span><span class="sxs-lookup"><span data-stu-id="05c9e-150">Navigate to the **template deployment** page, select the **create** button.</span></span> <span data-ttu-id="05c9e-151">Ezzel a gombbal megnyithatja a **egyéni telepítési** panelen.</span><span class="sxs-lookup"><span data-stu-id="05c9e-151">This button opens the **custom deployment** blade.</span></span>
3.  <span data-ttu-id="05c9e-152">Válassza ki a **sablon szerkesztése** gombra.</span><span class="sxs-lookup"><span data-stu-id="05c9e-152">Select the **edit template** button.</span></span>
4.  <span data-ttu-id="05c9e-153">Az üres sablon törlésére.</span><span class="sxs-lookup"><span data-stu-id="05c9e-153">Delete the empty template.</span></span> 
5.  <span data-ttu-id="05c9e-154">Másolja és illessze be a mintasablon a jobb oldali ablaktáblán.</span><span class="sxs-lookup"><span data-stu-id="05c9e-154">Copy and paste the sample template into the right pane.</span></span>
6.  <span data-ttu-id="05c9e-155">Válassza ki a **mentése** gombra.</span><span class="sxs-lookup"><span data-stu-id="05c9e-155">Select the **save** button.</span></span>
7.  <span data-ttu-id="05c9e-156">Amikor a rendszer visszairányítja a **egyéni telepítési** ablaktáblán válassza előbb a **paraméterek szerkesztése** gombra.</span><span class="sxs-lookup"><span data-stu-id="05c9e-156">When you are returned to the **custom deployment** pane, select the **edit parameters** button.</span></span>
8.  <span data-ttu-id="05c9e-157">Az a **paraméterek szerkesztése** panelen, törölje a meglévő sablon.</span><span class="sxs-lookup"><span data-stu-id="05c9e-157">On the **edit parameters** blade, delete the existing template.</span></span>
9.  <span data-ttu-id="05c9e-158">Másolja és illessze be a fenti mintasablon paraméter.</span><span class="sxs-lookup"><span data-stu-id="05c9e-158">Copy and paste the sample parameter template from above.</span></span>
10. <span data-ttu-id="05c9e-159">Válassza ki a **mentése** gomb, ami visszaadja, hogy a **egyéni telepítési** panelen.</span><span class="sxs-lookup"><span data-stu-id="05c9e-159">Select the **save** button, which returns you to the **custom deployment** blade.</span></span>
11. <span data-ttu-id="05c9e-160">Az a **egyéni telepítési** panelen jelölje ki az előfizetését, vagy létrehozhat új vagy meglévő erőforráscsoportot használni, és jelöljön ki egy helyet.</span><span class="sxs-lookup"><span data-stu-id="05c9e-160">On the **custom deployment** blade, select your subscription, either create new or use existing resource group, and select a location.</span></span> <span data-ttu-id="05c9e-161">Tekintse át a használati feltételeket, és válassza ki a **elfogadom** jelölőnégyzetet.</span><span class="sxs-lookup"><span data-stu-id="05c9e-161">Review the terms and conditions, and select the **I agree** checkbox.</span></span>
12. <span data-ttu-id="05c9e-162">Válassza ki a **beszerzési** gombra.</span><span class="sxs-lookup"><span data-stu-id="05c9e-162">Select the **purchase** button.</span></span>

## <a name="next-steps"></a><span data-ttu-id="05c9e-163">További lépések</span><span class="sxs-lookup"><span data-stu-id="05c9e-163">Next steps</span></span>

* <span data-ttu-id="05c9e-164">Ezek a technológiák megvalósítása után bővítheti a [tulajdonság objektum átalakító és adatgyűjtő](./collector.md).</span><span class="sxs-lookup"><span data-stu-id="05c9e-164">You can expand upon these techniques to implement a [property object transformer and collector](./collector.md).</span></span> <span data-ttu-id="05c9e-165">Az átalakító és adatgyűjtő technikák általános és a sablonok alapján lehet társítani.</span><span class="sxs-lookup"><span data-stu-id="05c9e-165">The transformer and collector techniques are more general and can be linked from your templates.</span></span>
* <span data-ttu-id="05c9e-166">Ezzel a módszerrel is implementálva van a [építőelemeket sablonprojekt](https://github.com/mspnp/template-building-blocks) és a [Azure hivatkozás architektúrák](/azure/architecture/reference-architectures/).</span><span class="sxs-lookup"><span data-stu-id="05c9e-166">This technique is also implemented in the [template building blocks project](https://github.com/mspnp/template-building-blocks) and the [Azure reference architectures](/azure/architecture/reference-architectures/).</span></span> <span data-ttu-id="05c9e-167">A sablonok milyen azt már megvalósított ezzel a módszerrel tekintheti meg.</span><span class="sxs-lookup"><span data-stu-id="05c9e-167">You can review our templates to see how we've implemented this technique.</span></span>

<!-- links -->
[azure-resource-manager-authoring-templates]: /azure/azure-resource-manager/resource-group-authoring-templates
[azure-resource-manager-create-template]: /azure/azure-resource-manager/resource-manager-create-first-template
[azure-resource-manager-create-multiple-instances]: /azure/azure-resource-manager/resource-group-create-multiple
[azure-resource-manager-functions]: /azure/azure-resource-manager/resource-group-template-functions-resource
[nsg]: /azure/virtual-network/virtual-networks-nsg