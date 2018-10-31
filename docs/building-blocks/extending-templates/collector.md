---
title: Tulajdonságátalakító és -gyűjtő megvalósítása az Azure Resource Manager-sablon
description: Ismerteti, hogyan lehet egy tulajdonságátalakító és -gyűjtő megvalósítása az Azure Resource Manager-sablon
author: petertay
ms.date: 10/30/2018
ms.openlocfilehash: ad5b3a71f516ec12fee311e25c43f434f9f306ed
ms.sourcegitcommit: e9eb2b895037da0633ef3ccebdea2fcce047620f
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 10/30/2018
ms.locfileid: "50251787"
---
# <a name="implement-a-property-transformer-and-collector-in-an-azure-resource-manager-template"></a><span data-ttu-id="c4ea9-103">Tulajdonságátalakító és -gyűjtő megvalósítása az Azure Resource Manager-sablon</span><span class="sxs-lookup"><span data-stu-id="c4ea9-103">Implement a property transformer and collector in an Azure Resource Manager template</span></span>

<span data-ttu-id="c4ea9-104">A [objektum használata paraméterként egy Azure Resource Manager-sablonban][objects-as-parameters], bemutattuk, hogyan erőforrás tulajdonságértékek tárolása egy objektumot, és alkalmazza őket egy resource üzembe helyezés során.</span><span class="sxs-lookup"><span data-stu-id="c4ea9-104">In [use an object as a parameter in an Azure Resource Manager template][objects-as-parameters], you learned how to store resource property values in an object and apply them to a resource during deployment.</span></span> <span data-ttu-id="c4ea9-105">Bár ez nagyon hasznos lehet kezelni a paramétereket, akkor továbbra is szükséges, hogy az objektum tulajdonságainak az erőforrás-tulajdonságok minden alkalommal, amikor használni a sablonban.</span><span class="sxs-lookup"><span data-stu-id="c4ea9-105">While this is a very useful way to manage your parameters, it still requires you to map the object's properties to resource properties each time you use it in your template.</span></span>

<span data-ttu-id="c4ea9-106">Ez elkerülhető, hogy egy tulajdonság átalakítási és -gyűjtő sablont, amely az objektum tömb végiglépkedve és alakítja azt az erőforrás által várt JSON-sémájában valósíthat meg.</span><span class="sxs-lookup"><span data-stu-id="c4ea9-106">To work around this, you can implement a property transform and collector template that iterates your object array and transforms it into the JSON schema expected by the resource.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="c4ea9-107">Ez a megközelítés megköveteli, hogy Ön ismeri részletesen a Resource Manager-sablonok és funkciók.</span><span class="sxs-lookup"><span data-stu-id="c4ea9-107">This approach requires that you have a deep understanding of Resource Manager templates and functions.</span></span>

<span data-ttu-id="c4ea9-108">Vessünk egy pillantást is tanfolyamsorozat egy tulajdonság-gyűjtő és a egy példa, amely üzembe helyezi az átalakító egy [hálózati biztonsági csoport (NSG)][nsg].</span><span class="sxs-lookup"><span data-stu-id="c4ea9-108">Let's take a look at how we can implement a property collector and transformer with an example that deploys a [network security group (NSG)][nsg].</span></span> <span data-ttu-id="c4ea9-109">Az alábbi ábra bemutatja a sablonokat, és ezeket a sablonokat az erőforrások közötti kapcsolat:</span><span class="sxs-lookup"><span data-stu-id="c4ea9-109">The diagram below shows the relationship between our templates and our resources within those templates:</span></span>

![tulajdonság-gyűjtő és átalakító architektúra](../_images/collector-transformer.png)

<span data-ttu-id="c4ea9-111">A **hívó sablon** két forrásokat tartalmazza:</span><span class="sxs-lookup"><span data-stu-id="c4ea9-111">Our **calling template** includes two resources:</span></span>
* <span data-ttu-id="c4ea9-112">egy sablon hivatkozást, amely meghívja a **gyűjtő sablon**.</span><span class="sxs-lookup"><span data-stu-id="c4ea9-112">a template link that invokes our **collector template**.</span></span>
* <span data-ttu-id="c4ea9-113">az NSG-erőforrás üzembe helyezéséhez.</span><span class="sxs-lookup"><span data-stu-id="c4ea9-113">the NSG resource to deploy.</span></span>

<span data-ttu-id="c4ea9-114">A **gyűjtő sablon** két forrásokat tartalmazza:</span><span class="sxs-lookup"><span data-stu-id="c4ea9-114">Our **collector template** includes two resources:</span></span>
* <span data-ttu-id="c4ea9-115">egy **forráshorgony** erőforrás.</span><span class="sxs-lookup"><span data-stu-id="c4ea9-115">an **anchor** resource.</span></span>
* <span data-ttu-id="c4ea9-116">sablon hivatkozás, amely egy másolási ciklust átalakító sablonnal hív meg.</span><span class="sxs-lookup"><span data-stu-id="c4ea9-116">a template link that invokes the transform template in a copy loop.</span></span>

<span data-ttu-id="c4ea9-117">A **átalakító sablon** egyetlen erőforrást tartalmaz: egy üres sablonnal, amely átalakítja a a `source` a JSON-séma szerint az NSG-erőforrás a várt JSON-a **fő sablon**.</span><span class="sxs-lookup"><span data-stu-id="c4ea9-117">Our **transform template** includes a single resource: an empty template with a variable that transforms our `source` JSON to the JSON schema expected by our NSG resource in the **main template**.</span></span>

## <a name="parameter-object"></a><span data-ttu-id="c4ea9-118">A paraméter objektum</span><span class="sxs-lookup"><span data-stu-id="c4ea9-118">Parameter object</span></span>

<span data-ttu-id="c4ea9-119">Fogjuk használni a `securityRules` paraméter objektumot [paraméterekként objektumok][objects-as-parameters].</span><span class="sxs-lookup"><span data-stu-id="c4ea9-119">We'll be using our `securityRules` parameter object from [objects as parameters][objects-as-parameters].</span></span> <span data-ttu-id="c4ea9-120">A **átalakító sablon** átalakítja az egyes objektumok a `securityRules` az NSG-erőforrás által várt JSON-sémájában tömböt a **hívó sablon**.</span><span class="sxs-lookup"><span data-stu-id="c4ea9-120">Our **transform template** will transform each object in the `securityRules` array into the JSON schema expected by the NSG resource in our **calling template**.</span></span>

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

<span data-ttu-id="c4ea9-121">Nézzük meg a **átalakító sablon** első.</span><span class="sxs-lookup"><span data-stu-id="c4ea9-121">Let's look at our **transform template** first.</span></span>

## <a name="transform-template"></a><span data-ttu-id="c4ea9-122">Sablonok átalakítása</span><span class="sxs-lookup"><span data-stu-id="c4ea9-122">Transform template</span></span>

<span data-ttu-id="c4ea9-123">A **átalakító sablon** két az átadott paramétereket tartalmaz az **gyűjtő sablon**:</span><span class="sxs-lookup"><span data-stu-id="c4ea9-123">Our **transform template** includes two parameters that are passed from the **collector template**:</span></span> 
* <span data-ttu-id="c4ea9-124">`source` egy olyan objektum, amely fogad egy tulajdonság értékének objektum a tulajdonság tömbből.</span><span class="sxs-lookup"><span data-stu-id="c4ea9-124">`source` is an object that receives one of the property value objects from the property array.</span></span> <span data-ttu-id="c4ea9-125">Ebben a példában minden egyes objektum a `"securityRules"` tömb átadása egyenként.</span><span class="sxs-lookup"><span data-stu-id="c4ea9-125">In our example, each object from the `"securityRules"` array will be passed in one at a time.</span></span>
* <span data-ttu-id="c4ea9-126">`state` van egy tömb, amely megkapja a korábbi átalakítások összefűzött eredményeit.</span><span class="sxs-lookup"><span data-stu-id="c4ea9-126">`state` is an array that receives the concatenated results of all the previous transforms.</span></span> <span data-ttu-id="c4ea9-127">Ez az átalakított JSON gyűjteménye.</span><span class="sxs-lookup"><span data-stu-id="c4ea9-127">This is the collection of transformed JSON.</span></span>

<span data-ttu-id="c4ea9-128">A paraméterek néznek ki:</span><span class="sxs-lookup"><span data-stu-id="c4ea9-128">Our parameters look like this:</span></span>

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "source": { "type": "object" },
    "state": {
      "type": "array",
      "defaultValue": [ ]
    }
  },
```

<span data-ttu-id="c4ea9-129">A sablon is meghatároz egy nevű változó `instance`.</span><span class="sxs-lookup"><span data-stu-id="c4ea9-129">Our template also defines a variable named `instance`.</span></span> <span data-ttu-id="c4ea9-130">Hajtja végre, a tényleges átalakítás a `source` objektumot a szükséges JSON-sémájában be:</span><span class="sxs-lookup"><span data-stu-id="c4ea9-130">It performs the actual transform of our `source` object into the required JSON schema:</span></span>

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

<span data-ttu-id="c4ea9-131">Végül a `output` a sablon fűzi össze az összegyűjtött átalakítások, a `state` az aktuális átalakítás végzi paraméter a `instance` változó:</span><span class="sxs-lookup"><span data-stu-id="c4ea9-131">Finally, the `output` of our template concatenates the collected transforms of our `state` parameter with the current transform performed by our `instance` variable:</span></span>

```json
    "resources": [],
    "outputs": {
    "collection": {
      "type": "array",
      "value": "[concat(parameters('state'), variables('instance'))]"
    }
```

<span data-ttu-id="c4ea9-132">Ezután vessünk egy pillantást a **gyűjtő sablon** megtekintéséhez, hogyan továbbítja a paraméterértékeket.</span><span class="sxs-lookup"><span data-stu-id="c4ea9-132">Next, let's take a look at our **collector template** to see how it passes in our parameter values.</span></span>

## <a name="collector-template"></a><span data-ttu-id="c4ea9-133">Gyűjtő sablon</span><span class="sxs-lookup"><span data-stu-id="c4ea9-133">Collector template</span></span>

<span data-ttu-id="c4ea9-134">A **gyűjtő sablon** három paramétereket tartalmazza:</span><span class="sxs-lookup"><span data-stu-id="c4ea9-134">Our **collector template** includes three parameters:</span></span>
* <span data-ttu-id="c4ea9-135">`source` az objektum teljes paramétertömböt van.</span><span class="sxs-lookup"><span data-stu-id="c4ea9-135">`source` is our complete parameter object array.</span></span> <span data-ttu-id="c4ea9-136">A által átadott a **hívó sablon**.</span><span class="sxs-lookup"><span data-stu-id="c4ea9-136">It's passed in by the **calling template**.</span></span> <span data-ttu-id="c4ea9-137">Ez rendelkezik-e a neve megegyezik a `source` paramétert a **átalakító sablon** , de talán már észrevette, hogy egy fő különbséggel: a teljes tömb, de csak átadott tömb egy elemet a **átalakító sablon** egyszerre.</span><span class="sxs-lookup"><span data-stu-id="c4ea9-137">This has the same name as the `source` parameter in our **transform template** but there is one key difference that you may have already noticed: this is the complete array, but we only pass one element of this array to the **transform template** at a time.</span></span>
* <span data-ttu-id="c4ea9-138">`transformTemplateUri` az URI-t, a **átalakító sablon**.</span><span class="sxs-lookup"><span data-stu-id="c4ea9-138">`transformTemplateUri` is the URI of our **transform template**.</span></span> <span data-ttu-id="c4ea9-139">A sablon újrahasznosíthatóság itt paramétert azt most definiálja.</span><span class="sxs-lookup"><span data-stu-id="c4ea9-139">We're defining it as a parameter here for template reusability.</span></span>
* <span data-ttu-id="c4ea9-140">`state` adjuk át, hogy kezdetben üres tömb a **átalakító sablon**.</span><span class="sxs-lookup"><span data-stu-id="c4ea9-140">`state` is an initially empty array that we pass to our **transform template**.</span></span> <span data-ttu-id="c4ea9-141">A másolási ciklus befejezésekor az átalakított paraméter objektumok gyűjteményét tárolja.</span><span class="sxs-lookup"><span data-stu-id="c4ea9-141">It stores the collection of transformed parameter objects when the copy loop is complete.</span></span>

<span data-ttu-id="c4ea9-142">A paraméterek néznek ki:</span><span class="sxs-lookup"><span data-stu-id="c4ea9-142">Our parameters look like this:</span></span>

```json
  "parameters": {
    "source": { "type": "array" },
    "transformTemplateUri": { "type": "string" },
    "state": {
      "type": "array",
      "defaultValue": [ ]
    }
``` 

<span data-ttu-id="c4ea9-143">Ezt követően adja meg azt az nevű változó `count`.</span><span class="sxs-lookup"><span data-stu-id="c4ea9-143">Next, we define a variable named `count`.</span></span> <span data-ttu-id="c4ea9-144">A érték hossza pedig a `source` objektum Pole parametru:</span><span class="sxs-lookup"><span data-stu-id="c4ea9-144">Its value is the length of the `source` parameter object array:</span></span>

```json
  "variables": {
    "count": "[length(parameters('source'))]"
  },
```

<span data-ttu-id="c4ea9-145">Mivel előfordulhat, hogy azt gyanítja, használja azt az ismétlések számát a másolási ciklust.</span><span class="sxs-lookup"><span data-stu-id="c4ea9-145">As you might suspect, we use it for the number of iterations in our copy loop.</span></span>

<span data-ttu-id="c4ea9-146">Most vessünk egy pillantást az erőforrások.</span><span class="sxs-lookup"><span data-stu-id="c4ea9-146">Now let's take a look at our resources.</span></span> <span data-ttu-id="c4ea9-147">Azt adja meg a két erőforrás:</span><span class="sxs-lookup"><span data-stu-id="c4ea9-147">We define two resources:</span></span>
* <span data-ttu-id="c4ea9-148">`loop-0` a másolási ciklust nulla-alapú erőforrás-van.</span><span class="sxs-lookup"><span data-stu-id="c4ea9-148">`loop-0` is the zero-based resource for our copy loop.</span></span>
* <span data-ttu-id="c4ea9-149">`loop-` eredménye van kibővítve a `copyIndex(1)` függvény használatával létrehoz egy egyedi iteráció-alapú nevet az erőforráshoz, kezdve `1`.</span><span class="sxs-lookup"><span data-stu-id="c4ea9-149">`loop-` is concatenated with the result of the `copyIndex(1)` function to generate a unique iteration-based name for our resource, starting with `1`.</span></span>

<span data-ttu-id="c4ea9-150">Az erőforrások néznek ki:</span><span class="sxs-lookup"><span data-stu-id="c4ea9-150">Our resources look like this:</span></span>

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
          "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
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

<span data-ttu-id="c4ea9-151">Vizsgáljuk meg közelebbről a azt Ön átadása paramétereket a **átalakító sablon** a beágyazott sablonban.</span><span class="sxs-lookup"><span data-stu-id="c4ea9-151">Let's take a closer look at the parameters we're passing to our **transform template** in the nested template.</span></span> <span data-ttu-id="c4ea9-152">A korábban emlékeztetnek arra, hogy a `source` paraméter adja meg az aktuális objektum a `source` objektum Pole parametru.</span><span class="sxs-lookup"><span data-stu-id="c4ea9-152">Recall from earlier that our `source` parameter passes the current object in the `source` parameter object array.</span></span> <span data-ttu-id="c4ea9-153">A `state` paraméter hol történik a gyűjteményt, mert tart a másolási ciklust előző lépésében kimenete&mdash;figyelje meg, hogy a `reference()` függvény a `copyIndex()` függvény paraméterek nélkül való hivatkozáshoz a `name` az előző társított sablonobjektum&mdash;, és átadja a jelenlegi verzió továbbfejlesztésében.</span><span class="sxs-lookup"><span data-stu-id="c4ea9-153">The `state` parameter is where the collection happens, because it takes the output of the previous iteration of our copy loop&mdash;notice that the `reference()` function uses the `copyIndex()` function with no parameter to reference the `name` of our previous linked template object&mdash;and passes it to the current iteration.</span></span>

<span data-ttu-id="c4ea9-154">Végül a `output` a sablon adja vissza a `output` az utolsó ismétlése a **átalakító sablon**:</span><span class="sxs-lookup"><span data-stu-id="c4ea9-154">Finally, the `output` of our template returns the `output` of the last iteration of our **transform template**:</span></span>

```json
  "outputs": {
    "result": {
      "type": "array",
      "value": "[reference(concat('loop-', variables('count'))).outputs.collection.value]"
    }
  }
```
<span data-ttu-id="c4ea9-155">Visszaadandó counterintuitive tűnhet a `output` az utolsó ismétlése a **átalakító sablon** , a **hívó sablon** jelent meg tudjuk tárolta, mert a `source` a paraméter.</span><span class="sxs-lookup"><span data-stu-id="c4ea9-155">It may seem counterintuitive to return the `output` of the last iteration of our **transform template** to our **calling template** because it appeared we were storing it in our `source` parameter.</span></span> <span data-ttu-id="c4ea9-156">Ne feledje azonban, hogy-e az utolsó ismétlése a **átalakító sablon** , amely tartalmazza a teljes átalakított tulajdonság objektumtömb, és ez pontosan a kívánt való visszatéréshez.</span><span class="sxs-lookup"><span data-stu-id="c4ea9-156">However, remember that it's the last iteration of our **transform template** that holds the complete array of transformed property objects, and that's what we want to return.</span></span>

<span data-ttu-id="c4ea9-157">Végül tekintsük át, hogyan lehet meghívni a **gyűjtő sablon** származó a **hívó sablon**.</span><span class="sxs-lookup"><span data-stu-id="c4ea9-157">Finally, let's take a look at how to call the **collector template** from our **calling template**.</span></span>

## <a name="calling-template"></a><span data-ttu-id="c4ea9-158">Hívása a sablon</span><span class="sxs-lookup"><span data-stu-id="c4ea9-158">Calling template</span></span>

<span data-ttu-id="c4ea9-159">A **hívó sablon** nevű egyetlen paraméter meghatározása `networkSecurityGroupsSettings`:</span><span class="sxs-lookup"><span data-stu-id="c4ea9-159">Our **calling template** defines a single parameter named `networkSecurityGroupsSettings`:</span></span>

```json
...
"parameters": {
    "networkSecurityGroupsSettings": {
        "type": "object"
    }
```

<span data-ttu-id="c4ea9-160">Következő lépésként határozza meg a sablont a nevű változóhoz `collectorTemplateUri`:</span><span class="sxs-lookup"><span data-stu-id="c4ea9-160">Next, our template defines a single variable named `collectorTemplateUri`:</span></span>

```json
"variables": {
    "collectorTemplateUri": "[uri(deployment().properties.templateLink.uri, 'collector.template.json')]"
  }
```

<span data-ttu-id="c4ea9-161">Hiányol, mivel ez az URI-JÁNAK a **gyűjtő sablon** , amelyek a társított sablonerőforrás használják:</span><span class="sxs-lookup"><span data-stu-id="c4ea9-161">As you would expect, this is the URI for the **collector template** that will be used by our linked template resource:</span></span>

```json
{
    "apiVersion": "2015-01-01",
    "name": "collector",
    "type": "Microsoft.Resources/deployments",
    "properties": {
        "mode": "Incremental",
        "templateLink": {
            "uri": "[variables('collectorTemplateUri')]",
            "contentVersion": "1.0.0.0"
        },
        "parameters": {
            "source" : {"value": "[parameters('networkSecurityGroupsSettings').securityRules]"},
            "transformTemplateUri": { "value": "[uri(deployment().properties.templateLink.uri, 'transform.json')]"}
        }
    }
}
```

<span data-ttu-id="c4ea9-162">Adjuk át, hogy a két paramétert a **gyűjtő sablon**:</span><span class="sxs-lookup"><span data-stu-id="c4ea9-162">We pass two parameters to the **collector template**:</span></span>
* <span data-ttu-id="c4ea9-163">`source` az objektum tulajdonságtömb van.</span><span class="sxs-lookup"><span data-stu-id="c4ea9-163">`source` is our property object array.</span></span> <span data-ttu-id="c4ea9-164">Ebben a példában ez a `networkSecurityGroupsSettings` paraméter.</span><span class="sxs-lookup"><span data-stu-id="c4ea9-164">In our example, it's our `networkSecurityGroupsSettings` parameter.</span></span>
* <span data-ttu-id="c4ea9-165">`transformTemplateUri` a változó csak meghatározott az URI-ját a **gyűjtő sablon**.</span><span class="sxs-lookup"><span data-stu-id="c4ea9-165">`transformTemplateUri` is the variable we just defined with the URI of our **collector template**.</span></span>

<span data-ttu-id="c4ea9-166">Végül a `Microsoft.Network/networkSecurityGroups` erőforrás közvetlenül rendeli hozzá a `output` , a `collector` sablonerőforrás hogy társított annak `securityRules` tulajdonság:</span><span class="sxs-lookup"><span data-stu-id="c4ea9-166">Finally, our `Microsoft.Network/networkSecurityGroups` resource directly assigns the `output` of the `collector` linked template resource to its `securityRules` property:</span></span>

```json
    {
      "apiVersion": "2015-06-15",
      "type": "Microsoft.Network/networkSecurityGroups",
      "name": "networkSecurityGroup1",
      "location": "[resourceGroup().location]",
      "properties": {
        "securityRules": "[reference('collector').outputs.result.value]"
      }
    }
  ],
  "outputs": {
      "instance":{
          "type": "array",
          "value": "[reference('collector').outputs.result.value]"
      }

  }
```

## <a name="try-the-template"></a><span data-ttu-id="c4ea9-167">A sablon kipróbálása</span><span class="sxs-lookup"><span data-stu-id="c4ea9-167">Try the template</span></span>

<span data-ttu-id="c4ea9-168">Egy példa sablon érhető el az [GitHub][github].</span><span class="sxs-lookup"><span data-stu-id="c4ea9-168">An example template is available on [GitHub][github].</span></span> <span data-ttu-id="c4ea9-169">A sablon üzembe helyezéséhez, klónozza az adattárat, és futtassa a következő [Azure CLI-vel] [ cli] parancsokat:</span><span class="sxs-lookup"><span data-stu-id="c4ea9-169">To deploy the template, clone the repo and run the following [Azure CLI][cli] commands:</span></span>

```bash
git clone https://github.com/mspnp/template-examples.git
cd template-examples/example4-collector
az group create --location <location> --name <resource-group-name>
az group deployment create -g <resource-group-name> \
    --template-uri https://raw.githubusercontent.com/mspnp/template-examples/master/example4-collector/deploy.json \
    --parameters deploy.parameters.json
```

<!-- links -->
[objects-as-parameters]: ./objects-as-parameters.md
[resource-manager-linked-template]: /azure/azure-resource-manager/resource-group-linked-templates
[resource-manager-variables]: /azure/azure-resource-manager/resource-group-template-functions-deployment
[nsg]: /azure/virtual-network/virtual-networks-nsg
[cli]: /cli/azure/?view=azure-cli-latest
[github]: https://github.com/mspnp/template-examples
