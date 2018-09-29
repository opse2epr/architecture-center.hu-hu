---
title: Objektum használata paraméterként az Azure Resource Manager-sablon
description: Ismerteti, hogyan lehet Azure Resource Manager-sablonok objektum használata paraméterként lehetőségeinek bővítése
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: 27bc4be02f202ae5d6a3c28553a8c8afe435f743
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 09/28/2018
ms.locfileid: "47429315"
---
# <a name="use-an-object-as-a-parameter-in-an-azure-resource-manager-template"></a><span data-ttu-id="81e06-103">Objektum használata paraméterként az Azure Resource Manager-sablon</span><span class="sxs-lookup"><span data-stu-id="81e06-103">Use an object as a parameter in an Azure Resource Manager template</span></span>

<span data-ttu-id="81e06-104">Ha Ön [Azure Resource Manager-sablonok készítése][azure-resource-manager-create-template], lehetősége van adja meg az erőforrás-tulajdonság értékeinek közvetlenül a sablonban vagy adjon meg paramétert, majd adja meg az értékeket üzembe helyezés során.</span><span class="sxs-lookup"><span data-stu-id="81e06-104">When you [author Azure Resource Manager templates][azure-resource-manager-create-template], you can either specify resource property values directly in the template or define a parameter and provide values during deployment.</span></span> <span data-ttu-id="81e06-105">Nyugodtan mindegyik tulajdonság értékét a kisméretű környezetek egy paraméter használható, de egy legfeljebb 255 paraméterek száma üzemelő példányonként.</span><span class="sxs-lookup"><span data-stu-id="81e06-105">It's fine to use a parameter for each property value for small deployments, but there is a limit of 255 parameters per deployment.</span></span> <span data-ttu-id="81e06-106">Miután a nagyobb és összetettebb üzembe helyezések paraméterei tartományon kívül is futtathat.</span><span class="sxs-lookup"><span data-stu-id="81e06-106">Once you get to larger and more complex deployments you may run out of parameters.</span></span>

<span data-ttu-id="81e06-107">A probléma megoldásához egyik módja az objektum használata paraméterként egy érték helyett.</span><span class="sxs-lookup"><span data-stu-id="81e06-107">One way to solve this problem is to use an object as a parameter instead of a value.</span></span> <span data-ttu-id="81e06-108">Ehhez adja meg a paramétert a sablont, és adja meg a JSON-objektum egyetlen érték helyett üzembe helyezés során.</span><span class="sxs-lookup"><span data-stu-id="81e06-108">To do this, define the parameter in your template and specify a JSON object instead of a single value during deployment.</span></span> <span data-ttu-id="81e06-109">Ezután hivatkozhat a altulajdonságokat a paraméter használatával a [ `parameter()` függvény] [ azure-resource-manager-functions] és a sablon pont operátort.</span><span class="sxs-lookup"><span data-stu-id="81e06-109">Then, reference the subproperties of the parameter using the [`parameter()` function][azure-resource-manager-functions] and dot operator in your template.</span></span>

<span data-ttu-id="81e06-110">Vessünk egy pillantást egy példa, amely üzembe helyez egy virtuális hálózati erőforrás.</span><span class="sxs-lookup"><span data-stu-id="81e06-110">Let's take a look at an example that deploys a virtual network resource.</span></span> <span data-ttu-id="81e06-111">Először tekintsük meg a `VNetSettings` paramétert a sablon és a `type` való `object`:</span><span class="sxs-lookup"><span data-stu-id="81e06-111">First, let's specify a `VNetSettings` parameter in our template and set the `type` to `object`:</span></span>

```json
...
"parameters": {
    "VNetSettings":{"type":"object"}
},
```
<span data-ttu-id="81e06-112">Következő lépésként hozzunk adja meg az értékeket a `VNetSettings` objektum:</span><span class="sxs-lookup"><span data-stu-id="81e06-112">Next, let's provide values for the `VNetSettings` object:</span></span>

> [!NOTE]
> <span data-ttu-id="81e06-113">Ismerje meg, hogyan deploment során adja meg a paraméterértékeket, tekintse meg a **paraméterek** szakaszában [struktúra és az Azure Resource Manager-sablonok szintaxisát][azure-resource-manager-authoring-templates].</span><span class="sxs-lookup"><span data-stu-id="81e06-113">To learn how to provide parameter values during deploment, see the **parameters** section of [understand the structure and syntax of Azure Resource Manager templates][azure-resource-manager-authoring-templates].</span></span> 

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

<span data-ttu-id="81e06-114">Amint látható, az egyetlen ténylegesen paraméterrel három altulajdonságok: `name`, `addressPrefixes`, és `subnets`.</span><span class="sxs-lookup"><span data-stu-id="81e06-114">As you can see, our single parameter actually specifies three subproperties: `name`, `addressPrefixes`, and `subnets`.</span></span> <span data-ttu-id="81e06-115">Ezek altulajdonságok mindegyike vagy értéket vagy más altulajdonságok adja meg.</span><span class="sxs-lookup"><span data-stu-id="81e06-115">Each of these subproperties either specifies a value or other subproperties.</span></span> <span data-ttu-id="81e06-116">Az eredménye, hogy az egyetlen paramétert adja meg a virtuális hálózat üzembe helyezéséhez szükséges összes értékét.</span><span class="sxs-lookup"><span data-stu-id="81e06-116">The result is that our single parameter specifies all the values necessary to deploy our virtual network.</span></span>

<span data-ttu-id="81e06-117">Most nézzük meg, hogy a sablon a `VNetSettings` objektumot használja:</span><span class="sxs-lookup"><span data-stu-id="81e06-117">Now let's have a look at the rest of our template to see how the `VNetSettings` object is used:</span></span>

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
<span data-ttu-id="81e06-118">Értékét a `VNetSettings` objektum érvénybe lépnek a tulajdonságokat, a virtuális hálózati erőforrás használata szükséges a `parameters()` mindkét függvény a `[]` indexer, és a pont operátor a tömb.</span><span class="sxs-lookup"><span data-stu-id="81e06-118">The values of our `VNetSettings` object are applied to the properties required by our virtual network resource using the `parameters()` function with both the `[]` array indexer and the dot operator.</span></span> <span data-ttu-id="81e06-119">Ez a megközelítés akkor működik, ha csak át szeretné statikusan Objekt paraméter értékét az erőforrásra alkalmazni kívánt.</span><span class="sxs-lookup"><span data-stu-id="81e06-119">This approach works if you just want to statically apply the values of the parameter object to the resource.</span></span> <span data-ttu-id="81e06-120">Azonban ha dinamikusan üzembe helyezés során a tulajdonság egy olyan értéktömböt hozzárendelni kívánt használhatja egy [másolási ciklust][azure-resource-manager-create-multiple-instances].</span><span class="sxs-lookup"><span data-stu-id="81e06-120">However, if you want to dynamically assign an array of property values during deployment you can use a [copy loop][azure-resource-manager-create-multiple-instances].</span></span> <span data-ttu-id="81e06-121">A másolási ciklust használandó értékeket ad meg egy JSON-tömböt erőforrás tulajdonság és a másolási ciklust dinamikusan vonatkozik az értékeket az erőforrás-tulajdonságok.</span><span class="sxs-lookup"><span data-stu-id="81e06-121">To use a copy loop, you provide a JSON array of resource property values and the copy loop dynamically applies the values to the resource's properties.</span></span> 

<span data-ttu-id="81e06-122">Érdemes figyelembe vennie a dinamikus megközelítés használatakor egy probléma van.</span><span class="sxs-lookup"><span data-stu-id="81e06-122">There is one issue to be aware of if you use the dynamic approach.</span></span> <span data-ttu-id="81e06-123">A probléma bemutatása érdekében vessünk egy pillantást egy tipikus tulajdonság értékek tömbje.</span><span class="sxs-lookup"><span data-stu-id="81e06-123">To demonstrate the issue, let's take a look at a typical array of property values.</span></span> <span data-ttu-id="81e06-124">Ebben a példában a tulajdonságok értékeit egy változó vannak tárolva.</span><span class="sxs-lookup"><span data-stu-id="81e06-124">In this example the values for our properties are stored in a variable.</span></span> <span data-ttu-id="81e06-125">Figyelje meg, hogy van két Tárolótömböket Itt&mdash;gyermektartománya `firstProperty` és a egy nevű `secondProperty`.</span><span class="sxs-lookup"><span data-stu-id="81e06-125">Notice we have two arrays here&mdash;one named `firstProperty` and one named `secondProperty`.</span></span> 

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

<span data-ttu-id="81e06-126">Most vessünk egy pillantást a változó segítségével egy másolási ciklust tulajdonság el módja.</span><span class="sxs-lookup"><span data-stu-id="81e06-126">Now let's take a look at the way we access the properties in the variable using a copy loop.</span></span>

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

<span data-ttu-id="81e06-127">A `copyIndex()` függvény az aktuális a ciklus ismétléseinek számától példányt adja vissza, és használjuk, amely index két tömb minden egyes egyszerre.</span><span class="sxs-lookup"><span data-stu-id="81e06-127">The `copyIndex()` function returns the current iteration of the copy loop, and we use that as an index into each of the two arrays simultaneously.</span></span>

<span data-ttu-id="81e06-128">Ez jól működik az, ha a két tömbök ugyanolyan hosszúságú.</span><span class="sxs-lookup"><span data-stu-id="81e06-128">This works fine when the two arrays are the same length.</span></span> <span data-ttu-id="81e06-129">A probléma merül fel, ha már állított be, és a két tömbök különböző hosszúságú&mdash;ebben az esetben a sablon üzembe helyezése során érvényesítése sikertelen lesz.</span><span class="sxs-lookup"><span data-stu-id="81e06-129">The issue arises if you've made a mistake and the two arrays are different lengths&mdash;in this case your template will fail validation during deployment.</span></span> <span data-ttu-id="81e06-130">Mivel sokkal egyszerűbb, ha egy érték nem hiányzik, akkor a probléma elkerülése érdekében egyetlen objektumot, beleértve az összes tulajdonság által.</span><span class="sxs-lookup"><span data-stu-id="81e06-130">You can avoid this issue by including all your properties in a single object, because it is much easier to see when a value is missing.</span></span> <span data-ttu-id="81e06-131">Például, tekintsük át egy másik paraméter objektum mely minden eleme a `propertyObject` Pole je Uniója a `firstProperty` és `secondProperty` korábban a tömbök.</span><span class="sxs-lookup"><span data-stu-id="81e06-131">For example, let's take a look another parameter object in which each element of the `propertyObject` array is the union of the `firstProperty` and `secondProperty` arrays from earlier.</span></span>

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

<span data-ttu-id="81e06-132">Figyelje meg, hogy a harmadik elem a tömbben található?</span><span class="sxs-lookup"><span data-stu-id="81e06-132">Notice the third element in the array?</span></span> <span data-ttu-id="81e06-133">Hiányzik a `number` tulajdonság, de vegye figyelembe, hogy korábban kihagyta, ha Ön szerzőként paraméter értékét így sokkal könnyebb a.</span><span class="sxs-lookup"><span data-stu-id="81e06-133">It's missing the `number` property, but it's much easier to notice that you've missed it when you're authoring the parameter values this way.</span></span>

## <a name="using-a-property-object-in-a-copy-loop"></a><span data-ttu-id="81e06-134">A másolási ciklust tulajdonság objektum használatával</span><span class="sxs-lookup"><span data-stu-id="81e06-134">Using a property object in a copy loop</span></span>

<span data-ttu-id="81e06-135">Ez a megközelítés akkor lesz még több hasznos, kombinálva a [soros másolási ciklust] [azure-resource-manager-létrehozása-több], különösen a gyermek-erőforrások üzembe helyezése.</span><span class="sxs-lookup"><span data-stu-id="81e06-135">This approach becomes even more useful when combined with the [serial copy loop][azure-resource-manager-create-multiple], particularly for deploying child resources.</span></span> 

<span data-ttu-id="81e06-136">Ennek nézzük meg a sablont, amely üzembe helyez egy [hálózati biztonsági csoport (NSG)] [ nsg] két kapcsolatbiztonsági szabályait.</span><span class="sxs-lookup"><span data-stu-id="81e06-136">To demonstrate this, let's look at a template that deploys a [network security group (NSG)][nsg] with two security rules.</span></span> 

<span data-ttu-id="81e06-137">Először vessünk egy pillantást a paramétereket.</span><span class="sxs-lookup"><span data-stu-id="81e06-137">First, let's take a look at our parameters.</span></span> <span data-ttu-id="81e06-138">Ha megnézzük a sablont, láthatjuk, hogy meghatároztuk nevű egy paraméter `networkSecurityGroupsSettings` , amely tartalmaz egy tömb nevű `securityRules`.</span><span class="sxs-lookup"><span data-stu-id="81e06-138">When we look at our template we'll see that we've defined one parameter named `networkSecurityGroupsSettings` that includes an array named `securityRules`.</span></span> <span data-ttu-id="81e06-139">A tömb két JSON-objektumok beállításainak biztonsági szabályok számát tartalmazza.</span><span class="sxs-lookup"><span data-stu-id="81e06-139">This array contains two JSON objects that specify a number of settings for a security rule.</span></span>

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

<span data-ttu-id="81e06-140">Most vessünk egy pillantást sablont.</span><span class="sxs-lookup"><span data-stu-id="81e06-140">Now let's take a look at our template.</span></span> <span data-ttu-id="81e06-141">Az első resource nevű `NSG1` az NSG-t telepíti.</span><span class="sxs-lookup"><span data-stu-id="81e06-141">Our first resource named `NSG1` deploys the NSG.</span></span> <span data-ttu-id="81e06-142">A második erőforrás nevű `loop-0` két funkciókat hajtja végre: először azt `dependsOn` az NSG-t, a telepítés nem indul el, amíg `NSG1` befejeződött, és az első példányát, a szekvenciális hurok.</span><span class="sxs-lookup"><span data-stu-id="81e06-142">Our second resource named `loop-0` performs two functions: first, it `dependsOn` the NSG so its deployment doesn't begin until `NSG1` is completed, and it is the first iteration of the sequential loop.</span></span> <span data-ttu-id="81e06-143">A harmadik erőforrás egy beágyazott sablont, amely üzembe helyezi a biztonsági szabályok használatával egy objektumot a paraméterértékeket, az utolsó példában látható módon.</span><span class="sxs-lookup"><span data-stu-id="81e06-143">Our third resource is a nested template that deploys our security rules using an object for its parameter values as in the last example.</span></span>

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
                "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
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
            "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
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

<span data-ttu-id="81e06-144">Vessünk hogyan adjuk meg a tulajdonság a közelebbről a `securityRules` gyermek-erőforrás.</span><span class="sxs-lookup"><span data-stu-id="81e06-144">Let's take a closer look at how we specify our property values in the `securityRules` child resource.</span></span> <span data-ttu-id="81e06-145">A tulajdonságokat hivatkozott használatával a `parameter()` függvény, utána pedig a pont operátor használata hivatkozhat a `securityRules` tömb, a jelenlegi érték meghaladja az iteráció által indexelt.</span><span class="sxs-lookup"><span data-stu-id="81e06-145">All of our properties are referenced using the `parameter()` function, and then we use the dot operator to reference our `securityRules` array, indexed by the current value of the iteration.</span></span> <span data-ttu-id="81e06-146">Egy másik pont operátor, az objektum nevére hivatkozhatnak használjuk.</span><span class="sxs-lookup"><span data-stu-id="81e06-146">Finally, we use another dot operator to reference the name of the object.</span></span> 

## <a name="try-the-template"></a><span data-ttu-id="81e06-147">A sablon kipróbálása</span><span class="sxs-lookup"><span data-stu-id="81e06-147">Try the template</span></span>

<span data-ttu-id="81e06-148">Ha szeretné, ez a sablon kísérletezhet, kövesse az alábbi lépéseket:</span><span class="sxs-lookup"><span data-stu-id="81e06-148">If you would like to experiment with this template, follow these steps:</span></span> 

1.  <span data-ttu-id="81e06-149">Nyissa meg az Azure Portalon, válassza a **+** ikonra, majd keresse meg a **sablonalapú telepítés** erőforrás írja be, és kattintson rá.</span><span class="sxs-lookup"><span data-stu-id="81e06-149">Go to the Azure portal, select the **+** icon, and search for the **template deployment** resource type, and select it.</span></span>
2.  <span data-ttu-id="81e06-150">Keresse meg a **sablonalapú telepítés** lapon válassza ki a **létrehozása** gombra.</span><span class="sxs-lookup"><span data-stu-id="81e06-150">Navigate to the **template deployment** page, select the **create** button.</span></span> <span data-ttu-id="81e06-151">Ezzel a gombbal megnyithatja a **egyéni üzembehelyezési** panelen.</span><span class="sxs-lookup"><span data-stu-id="81e06-151">This button opens the **custom deployment** blade.</span></span>
3.  <span data-ttu-id="81e06-152">Válassza ki a **sablon szerkesztése** gombra.</span><span class="sxs-lookup"><span data-stu-id="81e06-152">Select the **edit template** button.</span></span>
4.  <span data-ttu-id="81e06-153">Az üres sablon törlése.</span><span class="sxs-lookup"><span data-stu-id="81e06-153">Delete the empty template.</span></span> 
5.  <span data-ttu-id="81e06-154">Másolja és illessze be a mintasablon a jobb oldali ablaktáblán.</span><span class="sxs-lookup"><span data-stu-id="81e06-154">Copy and paste the sample template into the right pane.</span></span>
6.  <span data-ttu-id="81e06-155">Válassza ki a **mentése** gombra.</span><span class="sxs-lookup"><span data-stu-id="81e06-155">Select the **save** button.</span></span>
7.  <span data-ttu-id="81e06-156">Amikor a rendszer visszairányítja az **egyéni üzembehelyezési** panelen válassza a **paraméterek szerkesztése** gombra.</span><span class="sxs-lookup"><span data-stu-id="81e06-156">When you are returned to the **custom deployment** pane, select the **edit parameters** button.</span></span>
8.  <span data-ttu-id="81e06-157">Az a **paraméterek szerkesztése** panelen, a meglévő sablon törlése.</span><span class="sxs-lookup"><span data-stu-id="81e06-157">On the **edit parameters** blade, delete the existing template.</span></span>
9.  <span data-ttu-id="81e06-158">Másolja és illessze be a paraméter mintasablon a fent.</span><span class="sxs-lookup"><span data-stu-id="81e06-158">Copy and paste the sample parameter template from above.</span></span>
10. <span data-ttu-id="81e06-159">Válassza ki a **mentése** gombra, amely visszaadja, hogy a **egyéni üzembehelyezési** panelen.</span><span class="sxs-lookup"><span data-stu-id="81e06-159">Select the **save** button, which returns you to the **custom deployment** blade.</span></span>
11. <span data-ttu-id="81e06-160">Az a **egyéni üzembehelyezési** panelen válassza ki az előfizetését, vagy új létrehozása vagy meglévő erőforráscsoport használata, és válasszon ki egy helyet.</span><span class="sxs-lookup"><span data-stu-id="81e06-160">On the **custom deployment** blade, select your subscription, either create new or use existing resource group, and select a location.</span></span> <span data-ttu-id="81e06-161">Tekintse át a feltételeket és kikötéseket, és válassza ki a **elfogadom** jelölőnégyzetet.</span><span class="sxs-lookup"><span data-stu-id="81e06-161">Review the terms and conditions, and select the **I agree** checkbox.</span></span>
12. <span data-ttu-id="81e06-162">Válassza ki a **beszerzési** gombra.</span><span class="sxs-lookup"><span data-stu-id="81e06-162">Select the **purchase** button.</span></span>

## <a name="next-steps"></a><span data-ttu-id="81e06-163">További lépések</span><span class="sxs-lookup"><span data-stu-id="81e06-163">Next steps</span></span>

* <span data-ttu-id="81e06-164">Bővítheti, ezek a technológiák megvalósítása után egy [objektum tulajdonságátalakító és -gyűjtő](./collector.md).</span><span class="sxs-lookup"><span data-stu-id="81e06-164">You can expand upon these techniques to implement a [property object transformer and collector](./collector.md).</span></span> <span data-ttu-id="81e06-165">Az átalakító és adatgyűjtő technikák általános és a sablonok alapján lehet társítani.</span><span class="sxs-lookup"><span data-stu-id="81e06-165">The transformer and collector techniques are more general and can be linked from your templates.</span></span>
* <span data-ttu-id="81e06-166">Ez a módszer is implementálva van a [építőelemeket sablonprojekt](https://github.com/mspnp/template-building-blocks) és a [Azure-referenciaarchitektúrák](/azure/architecture/reference-architectures/).</span><span class="sxs-lookup"><span data-stu-id="81e06-166">This technique is also implemented in the [template building blocks project](https://github.com/mspnp/template-building-blocks) and the [Azure reference architectures](/azure/architecture/reference-architectures/).</span></span> <span data-ttu-id="81e06-167">A sablonok megtekintéséhez, hogy ezt a technikát általunk megvalósított tekintheti meg.</span><span class="sxs-lookup"><span data-stu-id="81e06-167">You can review our templates to see how we've implemented this technique.</span></span>

<!-- links -->
[azure-resource-manager-authoring-templates]: /azure/azure-resource-manager/resource-group-authoring-templates
[azure-resource-manager-create-template]: /azure/azure-resource-manager/resource-manager-create-first-template
[azure-resource-manager-create-multiple-instances]: /azure/azure-resource-manager/resource-group-create-multiple
[azure-resource-manager-functions]: /azure/azure-resource-manager/resource-group-template-functions-resource
[nsg]: /azure/virtual-network/virtual-networks-nsg