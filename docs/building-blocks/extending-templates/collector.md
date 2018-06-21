---
title: Egy tulajdonság átalakító és a gyűjtő valósítania az Azure Resource Manager-sablonok
description: Ismerteti, hogyan lehet egy tulajdonság átalakító és a gyűjtő valósítania az Azure Resource Manager-sablonok
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: 893779e652b845b3d936d11936dc767ef632fa43
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
ms.locfileid: "24538665"
---
# <a name="implement-a-property-transformer-and-collector-in-an-azure-resource-manager-template"></a><span data-ttu-id="6d379-103">Egy tulajdonság átalakító és a gyűjtő valósítania az Azure Resource Manager-sablonok</span><span class="sxs-lookup"><span data-stu-id="6d379-103">Implement a property transformer and collector in an Azure Resource Manager template</span></span>

<span data-ttu-id="6d379-104">A [objektum használata Azure Resource Manager-sablon egyik paraméterének][objects-as-parameters], megtudta, hogyan erőforrás tulajdonságértékek tárolása egy objektumot, és alkalmazni azokat az erőforráshoz való központi telepítése során.</span><span class="sxs-lookup"><span data-stu-id="6d379-104">In [use an object as a parameter in an Azure Resource Manager template][objects-as-parameters], you learned how to store resource property values in an object and apply them to a resource during deployment.</span></span> <span data-ttu-id="6d379-105">Ez nagyon hasznos módja a paraméterek kezeléséhez, miközben továbbra is igényel az objektum tulajdonságainak hozzárendelését erőforrás-tulajdonságok minden alkalommal, amikor a sablon használatát.</span><span class="sxs-lookup"><span data-stu-id="6d379-105">While this is a very useful way to manage your parameters, it still requires you to map the object's properties to resource properties each time you use it in your template.</span></span>

<span data-ttu-id="6d379-106">Ez elkerülhető, hogy a tulajdonság-átalakítási és -gyűjtő sablont, amely megismétli a object tömbben, és az erőforrás által várt JSON-séma alakítja valósíthat meg.</span><span class="sxs-lookup"><span data-stu-id="6d379-106">To work around this, you can implement a property transform and collector template that iterates your object array and transforms it into the JSON schema expected by the resource.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="6d379-107">Ez a megközelítés megköveteli, hogy Ön ismeri részletesen a Resource Manager sablonok és a funkciók.</span><span class="sxs-lookup"><span data-stu-id="6d379-107">This approach requires that you have a deep understanding of Resource Manager templates and functions.</span></span>

<span data-ttu-id="6d379-108">Vessen egy pillantást hogyan ket lehet megvalósítani egy tulajdonság adatgyűjtő és fogyasztótípusokkal egy példa egy [hálózati biztonsági csoport (NSG)][nsg].</span><span class="sxs-lookup"><span data-stu-id="6d379-108">Let's take a look at how we can implement a property collector and transformer with an example that deploys a [network security group (NSG)][nsg].</span></span> <span data-ttu-id="6d379-109">Az alábbi ábrán a sablonokat, és ezeket a sablonokat az erőforrások közötti kapcsolatot ábrázolja:</span><span class="sxs-lookup"><span data-stu-id="6d379-109">The diagram below shows the relationship between our templates and our resources within those templates:</span></span>

![tulajdonság-gyűjtő és az átalakító architektúrája](../_images/collector-transformer.png)

<span data-ttu-id="6d379-111">A **hívó sablon** két forrásokat tartalmazza:</span><span class="sxs-lookup"><span data-stu-id="6d379-111">Our **calling template** includes two resources:</span></span>
* <span data-ttu-id="6d379-112">a sablon hivatkozást, amellyel elindítja a **adatgyűjtő sablon**.</span><span class="sxs-lookup"><span data-stu-id="6d379-112">a template link that invokes our **collector template**.</span></span>
* <span data-ttu-id="6d379-113">a telepítendő NSG-erőforrás.</span><span class="sxs-lookup"><span data-stu-id="6d379-113">the NSG resource to deploy.</span></span>

<span data-ttu-id="6d379-114">A **adatgyűjtő sablon** két forrásokat tartalmazza:</span><span class="sxs-lookup"><span data-stu-id="6d379-114">Our **collector template** includes two resources:</span></span>
* <span data-ttu-id="6d379-115">egy **horgonyzási** erőforrás.</span><span class="sxs-lookup"><span data-stu-id="6d379-115">an **anchor** resource.</span></span>
* <span data-ttu-id="6d379-116">egy sablon hivatkozást, amely hívja meg a másolási ciklust átalakító-sablon.</span><span class="sxs-lookup"><span data-stu-id="6d379-116">a template link that invokes the transform template in a copy loop.</span></span>

<span data-ttu-id="6d379-117">A **átalakító sablon** tartalmaz, amely egyetlen erőforrásra: egy üres sablont egy változóhoz, amely átalakítja a `source` , amelyet az NSG-erőforrás a várt JSON-séma a JSON a **fő sablon**.</span><span class="sxs-lookup"><span data-stu-id="6d379-117">Our **transform template** includes a single resource: an empty template with a variable that transforms our `source` JSON to the JSON schema expected by our NSG resource in the **main template**.</span></span>

## <a name="parameter-object"></a><span data-ttu-id="6d379-118">A paraméter objektum</span><span class="sxs-lookup"><span data-stu-id="6d379-118">Parameter object</span></span>

<span data-ttu-id="6d379-119">Fogjuk használni a `securityRules` paraméter objektum [paraméterekként objektumok][objects-as-parameters].</span><span class="sxs-lookup"><span data-stu-id="6d379-119">We'll be using our `securityRules` parameter object from [objects as parameters][objects-as-parameters].</span></span> <span data-ttu-id="6d379-120">A **átalakító sablon** átalakítja az egyes objektumok a `securityRules` tömb be, amelyet az NSG-erőforrás a várt JSON-séma a **hívó sablon**.</span><span class="sxs-lookup"><span data-stu-id="6d379-120">Our **transform template** will transform each object in the `securityRules` array into the JSON schema expected by the NSG resource in our **calling template**.</span></span>

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

<span data-ttu-id="6d379-121">Vizsgáljuk meg a **átalakító sablon** első.</span><span class="sxs-lookup"><span data-stu-id="6d379-121">Let's look at our **transform template** first.</span></span>

## <a name="transform-template"></a><span data-ttu-id="6d379-122">Átalakítás sablon</span><span class="sxs-lookup"><span data-stu-id="6d379-122">Transform template</span></span>

<span data-ttu-id="6d379-123">A **átalakító sablon** az átadott két paramétereket tartalmaz a **adatgyűjtő sablon**:</span><span class="sxs-lookup"><span data-stu-id="6d379-123">Our **transform template** includes two parameters that are passed from the **collector template**:</span></span> 
* <span data-ttu-id="6d379-124">`source`van olyan objektum, amely fogad egy tulajdonság értékének objektum a tulajdonság-tárolótömb.</span><span class="sxs-lookup"><span data-stu-id="6d379-124">`source` is an object that receives one of the property value objects from the property array.</span></span> <span data-ttu-id="6d379-125">A fenti példában minden egyes objektum a `"securityRules"` tömb átadása egyenként.</span><span class="sxs-lookup"><span data-stu-id="6d379-125">In our example, each object from the `"securityRules"` array will be passed in one at a time.</span></span>
* <span data-ttu-id="6d379-126">`state`van olyan tömb, amely megkapja a korábbi átalakítások összefűzött eredményeit.</span><span class="sxs-lookup"><span data-stu-id="6d379-126">`state` is an array that receives the concatenated results of all the previous transforms.</span></span> <span data-ttu-id="6d379-127">Ez az átalakítani kívánt JSON gyűjteménye.</span><span class="sxs-lookup"><span data-stu-id="6d379-127">This is the collection of transformed JSON.</span></span>

<span data-ttu-id="6d379-128">A paraméterek néznek ki:</span><span class="sxs-lookup"><span data-stu-id="6d379-128">Our parameters look like this:</span></span>

```json
{
  "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "source": { "type": "object" },
    "state": {
      "type": "array",
      "defaultValue": [ ]
    }
  },
```

<span data-ttu-id="6d379-129">A sablon is meghatároz egy nevű változó `instance`.</span><span class="sxs-lookup"><span data-stu-id="6d379-129">Our template also defines a variable named `instance`.</span></span> <span data-ttu-id="6d379-130">A tényleges tranform segítségével végzi a `source` objektumot a szükséges JSON-séma be:</span><span class="sxs-lookup"><span data-stu-id="6d379-130">It performs the actual tranform of our `source` object into the required JSON schema:</span></span>

```json
  "variables": {
    "instance": [
      {
        "name": "[parameters('source').name]",
        "properties":{
            "description": "[parameters('source').description]",
            "protocol": "[parameters('source').protocol]",
            "sourcePortRange": "[parameters('source').sourcePortRange]",
            "destinationPortRange": "[parameters('source').destinationPortRange]",
            "sourceAddressPrefix": "[parameters('source').sourceAddressPrefix]",
            "destinationAddressPrefix": "[parameters('source').destinationAddressPrefix]",
            "access": "[parameters('source').access]",
            "priority": "[parameters('source').priority]",
            "direction": "[parameters('source').direction]"            
        }
      }
    ]

  },
```

<span data-ttu-id="6d379-131">Végezetül a `output` a sablon az összegyűjtött átalakítások összefűzi a `state` végzi el az aktuális átalakítás paraméterrel a `instance` változó:</span><span class="sxs-lookup"><span data-stu-id="6d379-131">Finally, the `output` of our template concatenates the collected transforms of our `state` parameter with the current transform performed by our `instance` variable:</span></span>

```json
  "outputs": {
    "collection": {
      "type": "array",
      "value": "[concat(parameters('state'), variables('instance'))]"
    }
```

<span data-ttu-id="6d379-132">A következő vessen egy pillantást a **adatgyűjtő sablon** hogyan továbbítja a paraméterértékeket a megtekintéséhez.</span><span class="sxs-lookup"><span data-stu-id="6d379-132">Next, let's take a look at our **collector template** to see how it passes in our parameter values.</span></span>

## <a name="collector-template"></a><span data-ttu-id="6d379-133">Adatgyűjtő sablon</span><span class="sxs-lookup"><span data-stu-id="6d379-133">Collector template</span></span>

<span data-ttu-id="6d379-134">A **adatgyűjtő sablon** három paramétereket tartalmaz:</span><span class="sxs-lookup"><span data-stu-id="6d379-134">Our **collector template** includes three parameters:</span></span>
* <span data-ttu-id="6d379-135">`source`a teljes objektum paramétertömb van.</span><span class="sxs-lookup"><span data-stu-id="6d379-135">`source` is our complete parameter object array.</span></span> <span data-ttu-id="6d379-136">A által átadott a **hívó sablon**.</span><span class="sxs-lookup"><span data-stu-id="6d379-136">It's passed in by the **calling template**.</span></span> <span data-ttu-id="6d379-137">A neve megegyezik annak a `source` paramétere a **átalakító sablon** , de egy fő különbség, hogy talán már észrevette: Ez a teljes tömb, de azt csak átadni a tömb egy elemet a **átalakító sablon** egyszerre.</span><span class="sxs-lookup"><span data-stu-id="6d379-137">This has the same name as the `source` parameter in our **transform template** but there is one key difference that you may have already noticed: this is the complete array, but we only pass one element of this array to the **transform template** at a time.</span></span>
* <span data-ttu-id="6d379-138">`transformTemplateUri`az URI-azonosítója a **átalakító sablon**.</span><span class="sxs-lookup"><span data-stu-id="6d379-138">`transformTemplateUri` is the URI of our **transform template**.</span></span> <span data-ttu-id="6d379-139">A sablon újrahasznosításának itt paramétert azt még meghatározása.</span><span class="sxs-lookup"><span data-stu-id="6d379-139">We're defining it as a parameter here for template reusability.</span></span>
* <span data-ttu-id="6d379-140">`state`kezdetben üres tömb, hogy a **átalakító sablon**.</span><span class="sxs-lookup"><span data-stu-id="6d379-140">`state` is an initially empty array that we pass to our **transform template**.</span></span> <span data-ttu-id="6d379-141">A másolási ciklust befejezésekor átalakított paraméter objektumok gyűjteménye tárolja.</span><span class="sxs-lookup"><span data-stu-id="6d379-141">It stores the collection of transformed parameter objects when the copy loop is complete.</span></span>

<span data-ttu-id="6d379-142">A paraméterek néznek ki:</span><span class="sxs-lookup"><span data-stu-id="6d379-142">Our parameters look like this:</span></span>

```json
  "parameters": {
    "source": { "type": "array" },
    "transformTemplateUri": { "type": "string" },
    "state": {
      "type": "array",
      "defaultValue": [ ]
    }
``` 

<span data-ttu-id="6d379-143">A következő nevű változó meghatároztuk `count`.</span><span class="sxs-lookup"><span data-stu-id="6d379-143">Next, we define a variable named `count`.</span></span> <span data-ttu-id="6d379-144">Az érték hossza a `source` paraméter object tömbben:</span><span class="sxs-lookup"><span data-stu-id="6d379-144">Its value is the length of the `source` parameter object array:</span></span>

```json
  "variables": {
    "count": "[length(parameters('source'))]"
  },
```

<span data-ttu-id="6d379-145">Mivel előfordulhat, hogy azt gyanítja, azt használatát ismétlések számát a másolási ciklust.</span><span class="sxs-lookup"><span data-stu-id="6d379-145">As you might suspect, we use it for the number of iterations in our copy loop.</span></span>

<span data-ttu-id="6d379-146">Most vessen egy pillantást az erőforrásokat.</span><span class="sxs-lookup"><span data-stu-id="6d379-146">Now let's take a look at our resources.</span></span> <span data-ttu-id="6d379-147">Azt adja meg a két erőforrások:</span><span class="sxs-lookup"><span data-stu-id="6d379-147">We define two resources:</span></span>
* <span data-ttu-id="6d379-148">`loop-0`a másolási ciklust a nulla alapú erőforrás van.</span><span class="sxs-lookup"><span data-stu-id="6d379-148">`loop-0` is the zero-based resource for our copy loop.</span></span>
* <span data-ttu-id="6d379-149">`loop-`van kibővítve eredményét a `copyIndex(1)` működnek, mint az erőforrás, kezdve egyedi iterációs alapú nevet létrehozni `1`.</span><span class="sxs-lookup"><span data-stu-id="6d379-149">`loop-` is concatenated with the result of the `copyIndex(1)` function to generate a unique iteration-based name for our resource, starting with `1`.</span></span>

<span data-ttu-id="6d379-150">Az erőforrások néznek ki:</span><span class="sxs-lookup"><span data-stu-id="6d379-150">Our resources look like this:</span></span>

```json
  "resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2015-01-01",
      "name": "loop-0",
      "properties": {
        "mode": "Incremental",
        "parameters": { },
        "template": {
          "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": { },
          "variables": { },
          "resources": [ ],
          "outputs": {
            "collection": {
              "type": "array",
              "value": "[parameters('state')]"
            }
          }
        }
      }
    },
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2015-01-01",
      "name": "[concat('loop-', copyindex(1))]",
      "copy": {
        "name": "iterator",
        "count": "[variables('count')]",
        "mode": "serial"
      },
      "dependsOn": [
        "loop-0"
      ],
      "properties": {
        "mode": "Incremental",
        "templateLink": { "uri": "[parameters('transformTemplateUri')]" },
        "parameters": {
          "source": { "value": "[parameters('source')[copyindex()]]" },
          "state": { "value": "[reference(concat('loop-', copyindex())).outputs.collection.value]" }
        }
      }
    }
  ],
```

<span data-ttu-id="6d379-151">Ismerkedjen meg a paramétereket, azt hogy átadni részletes bemutatása a **átalakító sablon** a beágyazott sablonban.</span><span class="sxs-lookup"><span data-stu-id="6d379-151">Let's take a closer look at the parameters we're passing to our **transform template** in the nested template.</span></span> <span data-ttu-id="6d379-152">A visszahívás korábbi, amely a `source` paraméter adja meg az aktuális objektum a `source` paraméter object tömbben.</span><span class="sxs-lookup"><span data-stu-id="6d379-152">Recall from earlier that our `source` parameter passes the current object in the `source` parameter object array.</span></span> <span data-ttu-id="6d379-153">A `state` paraméter hol történik a gyűjteményhez, mert az előző munkamenetben, a másolási ciklust kimenete tart&mdash;figyelje meg, hogy a `reference()` függvény a `copyIndex()` függvény a nem a hivatkozniparaméter`name` az előző csatolt sablon objektum&mdash;, és átadja az aktuális iteráció.</span><span class="sxs-lookup"><span data-stu-id="6d379-153">The `state` parameter is where the collection happens, because it takes the output of the previous iteration of our copy loop&mdash;notice that the `reference()` function uses the `copyIndex()` function with no parameter to reference the `name` of our previous linked template object&mdash;and passes it to the current iteration.</span></span>

<span data-ttu-id="6d379-154">Végezetül a `output` a sablon adja vissza a `output` , az utolsó lépésében a **átalakító sablon**:</span><span class="sxs-lookup"><span data-stu-id="6d379-154">Finally, the `output` of our template returns the `output` of the last iteration of our **transform template**:</span></span>

```json
  "outputs": {
    "result": {
      "type": "array",
      "value": "[reference(concat('loop-', variables('count'))).outputs.collection.value]"
    }
  }
```
<span data-ttu-id="6d379-155">A megszokás vissza a `output` az utolsó lépésében a a **átalakító sablon** való a **hívó sablon** mert jelent meg azt is tárolja a `source` a paraméter.</span><span class="sxs-lookup"><span data-stu-id="6d379-155">It may seem counterintuitive to return the `output` of the last iteration of our **transform template** to our **calling template** because it appeared we were storing it in our `source` parameter.</span></span> <span data-ttu-id="6d379-156">Ne feledje azonban, hogy-e az utolsó lépésében a **átalakító sablon** , amely tartalmazza a teljes átalakított tulajdonság objektumokból álló tömb, és pontosan a kívánt való visszatéréshez.</span><span class="sxs-lookup"><span data-stu-id="6d379-156">However, remember that it's the last iteration of our **transform template** that holds the complete array of transformed property objects, and that's what we want to return.</span></span>

<span data-ttu-id="6d379-157">Végezetül vessen egy pillantást hogyan hívhatja meg a **adatgyűjtő sablon** a a **hívó sablon**.</span><span class="sxs-lookup"><span data-stu-id="6d379-157">Finally, let's take a look at how to call the **collector template** from our **calling template**.</span></span>

## <a name="calling-template"></a><span data-ttu-id="6d379-158">Hívása sablon</span><span class="sxs-lookup"><span data-stu-id="6d379-158">Calling template</span></span>

<span data-ttu-id="6d379-159">A **hívó sablon** nevű egyetlen paraméter meghatározza `networkSecurityGroupsSettings`:</span><span class="sxs-lookup"><span data-stu-id="6d379-159">Our **calling template** defines a single parameter named `networkSecurityGroupsSettings`:</span></span>

```json
...
"parameters": {
    "networkSecurityGroupsSettings": {
        "type": "object"
    }
```

<span data-ttu-id="6d379-160">Ezt követően a sablon nevű egyetlen változót definiál `collectorTemplateUri`:</span><span class="sxs-lookup"><span data-stu-id="6d379-160">Next, our template defines a single variable named `collectorTemplateUri`:</span></span>

```json
"variables": {
    "collectorTemplateUri": "[uri(deployment().properties.templateLink.uri, 'collector.template.json')]"
  }
```

<span data-ttu-id="6d379-161">Teheti meg, mert ez az URI-JÁNAK a **adatgyűjtő sablon** , amely a csatolt sablonerőforrás használják:</span><span class="sxs-lookup"><span data-stu-id="6d379-161">As you would expect, this is the URI for the **collector template** that will be used by our linked template resource:</span></span>

```json
{
    "apiVersion": "2015-01-01",
    "name": "collector",
    "type": "Microsoft.Resources/deployments",
    "properties": {
        "mode": "Incremental",
        "templateLink": {
            "uri": "[variables('linkedTemplateUri')]",
            "contentVersion": "1.0.0.0"
        },
        "parameters": {
            "source" : {"value": "[parameters('networkSecurityGroupsSettings').securityRules]"},
            "transformTemplateUri": { "value": "[uri(deployment().properties.templateLink.uri, 'transform.json')]"}
        }
    }
}
```

<span data-ttu-id="6d379-162">Azt adja át a két paraméter a **adatgyűjtő sablon**:</span><span class="sxs-lookup"><span data-stu-id="6d379-162">We pass two parameters to the **collector template**:</span></span>
* <span data-ttu-id="6d379-163">`source`az objektum tulajdonságtömb van.</span><span class="sxs-lookup"><span data-stu-id="6d379-163">`source` is our property object array.</span></span> <span data-ttu-id="6d379-164">A példánkban ez a `networkSecurityGroupsSettings` paraméter.</span><span class="sxs-lookup"><span data-stu-id="6d379-164">In our example, it's our `networkSecurityGroupsSettings` parameter.</span></span>
* <span data-ttu-id="6d379-165">`transformTemplateUri`az URI-azonosítója csak meghatározott, a változó a **adatgyűjtő sablon**.</span><span class="sxs-lookup"><span data-stu-id="6d379-165">`transformTemplateUri` is the variable we just defined with the URI of our **collector template**.</span></span>

<span data-ttu-id="6d379-166">Végezetül a `Microsoft.Network/networkSecurityGroups` erőforrás közvetlenül rendeli hozzá a `output` , a `collector` kapcsolódó sablon erőforrást a `securityRules` tulajdonság:</span><span class="sxs-lookup"><span data-stu-id="6d379-166">Finally, our `Microsoft.Network/networkSecurityGroups` resource directly assigns the `output` of the `collector` linked template resource to its `securityRules` property:</span></span>

```json
    {
      "apiVersion": "2015-06-15",
      "type": "Microsoft.Network/networkSecurityGroups",
      "name": "networkSecurityGroup1",
      "location": "[resourceGroup().location]",
      "properties": {
        "securityRules": "[reference('firstResource').outputs.result.value]"
      }
    }
  ],
  "outputs": {
      "instance":{
          "type": "array",
          "value": "[reference('firstResource').outputs.result.value]"
      }

  }
```

## <a name="next-steps"></a><span data-ttu-id="6d379-167">Következő lépések</span><span class="sxs-lookup"><span data-stu-id="6d379-167">Next steps</span></span>

* <span data-ttu-id="6d379-168">Ezzel a technikával meg van valósítva a [építőelemeket sablonprojekt](https://github.com/mspnp/template-building-blocks) és a [Azure hivatkozás architektúrák](/azure/architecture/reference-architectures/).</span><span class="sxs-lookup"><span data-stu-id="6d379-168">This technique is implemented in the [template building blocks project](https://github.com/mspnp/template-building-blocks) and the [Azure reference architectures](/azure/architecture/reference-architectures/).</span></span> <span data-ttu-id="6d379-169">Ezek segítségével hozzon létre egy saját architektúra, vagy telepítse a referencia-architektúrák egyikét használhatja.</span><span class="sxs-lookup"><span data-stu-id="6d379-169">You can use these to create your own architecture or deploy one of our reference architectures.</span></span>

<!-- links -->
[objects-as-parameters]: ./objects-as-parameters.md
[resource-manager-linked-template]: /azure/azure-resource-manager/resource-group-linked-templates
[resource-manager-variables]: /azure/azure-resource-manager/resource-group-template-functions-deployment
[nsg]: /azure/virtual-network/virtual-networks-nsg
