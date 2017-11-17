---
title: "Egy tulajdonság átalakító és a gyűjtő valósítania az Azure Resource Manager-sablonok"
description: "Ismerteti, hogyan lehet egy tulajdonság átalakító és a gyűjtő valósítania az Azure Resource Manager-sablonok"
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: 893779e652b845b3d936d11936dc767ef632fa43
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="implement-a-property-transformer-and-collector-in-an-azure-resource-manager-template"></a>Egy tulajdonság átalakító és a gyűjtő valósítania az Azure Resource Manager-sablonok

A [objektum használata Azure Resource Manager-sablon egyik paraméterének][objects-as-parameters], megtudta, hogyan erőforrás tulajdonságértékek tárolása egy objektumot, és alkalmazni azokat az erőforráshoz való központi telepítése során. Ez nagyon hasznos módja a paraméterek kezeléséhez, miközben továbbra is igényel az objektum tulajdonságainak hozzárendelését erőforrás-tulajdonságok minden alkalommal, amikor a sablon használatát.

Ez elkerülhető, hogy a tulajdonság-átalakítási és -gyűjtő sablont, amely megismétli a object tömbben, és az erőforrás által várt JSON-séma alakítja valósíthat meg.

> [!IMPORTANT]
> Ez a megközelítés megköveteli, hogy Ön ismeri részletesen a Resource Manager sablonok és a funkciók.

Vessen egy pillantást hogyan ket lehet megvalósítani egy tulajdonság adatgyűjtő és fogyasztótípusokkal egy példa egy [hálózati biztonsági csoport (NSG)][nsg]. Az alábbi ábrán a sablonokat, és ezeket a sablonokat az erőforrások közötti kapcsolatot ábrázolja:

![tulajdonság-gyűjtő és az átalakító architektúrája](../_images/collector-transformer.png)

A **hívó sablon** két forrásokat tartalmazza:
* a sablon hivatkozást, amellyel elindítja a **adatgyűjtő sablon**.
* a telepítendő NSG-erőforrás.

A **adatgyűjtő sablon** két forrásokat tartalmazza:
* egy **horgonyzási** erőforrás.
* egy sablon hivatkozást, amely hívja meg a másolási ciklust átalakító-sablon.

A **átalakító sablon** tartalmaz, amely egyetlen erőforrásra: egy üres sablont egy változóhoz, amely átalakítja a `source` , amelyet az NSG-erőforrás a várt JSON-séma a JSON a **fő sablon**.

## <a name="parameter-object"></a>A paraméter objektum

Fogjuk használni a `securityRules` paraméter objektum [paraméterekként objektumok][objects-as-parameters]. A **átalakító sablon** átalakítja az egyes objektumok a `securityRules` tömb be, amelyet az NSG-erőforrás a várt JSON-séma a **hívó sablon**.

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

Vizsgáljuk meg a **átalakító sablon** első.

## <a name="transform-template"></a>Átalakítás sablon

A **átalakító sablon** az átadott két paramétereket tartalmaz a **adatgyűjtő sablon**: 
* `source`van olyan objektum, amely fogad egy tulajdonság értékének objektum a tulajdonság-tárolótömb. A fenti példában minden egyes objektum a `"securityRules"` tömb átadása egyenként.
* `state`van olyan tömb, amely megkapja a korábbi átalakítások összefűzött eredményeit. Ez az átalakítani kívánt JSON gyűjteménye.

A paraméterek néznek ki:

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

A sablon is meghatároz egy nevű változó `instance`. A tényleges tranform segítségével végzi a `source` objektumot a szükséges JSON-séma be:

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

Végezetül a `output` a sablon az összegyűjtött átalakítások összefűzi a `state` végzi el az aktuális átalakítás paraméterrel a `instance` változó:

```json
  "outputs": {
    "collection": {
      "type": "array",
      "value": "[concat(parameters('state'), variables('instance'))]"
    }
```

A következő vessen egy pillantást a **adatgyűjtő sablon** hogyan továbbítja a paraméterértékeket a megtekintéséhez.

## <a name="collector-template"></a>Adatgyűjtő sablon

A **adatgyűjtő sablon** három paramétereket tartalmaz:
* `source`a teljes objektum paramétertömb van. A által átadott a **hívó sablon**. A neve megegyezik annak a `source` paramétere a **átalakító sablon** , de egy fő különbség, hogy talán már észrevette: Ez a teljes tömb, de azt csak átadni a tömb egy elemet a **átalakító sablon** egyszerre.
* `transformTemplateUri`az URI-azonosítója a **átalakító sablon**. A sablon újrahasznosításának itt paramétert azt még meghatározása.
* `state`kezdetben üres tömb, hogy a **átalakító sablon**. A másolási ciklust befejezésekor átalakított paraméter objektumok gyűjteménye tárolja.

A paraméterek néznek ki:

```json
  "parameters": {
    "source": { "type": "array" },
    "transformTemplateUri": { "type": "string" },
    "state": {
      "type": "array",
      "defaultValue": [ ]
    }
``` 

A következő nevű változó meghatároztuk `count`. Az érték hossza a `source` paraméter object tömbben:

```json
  "variables": {
    "count": "[length(parameters('source'))]"
  },
```

Mivel előfordulhat, hogy azt gyanítja, azt használatát ismétlések számát a másolási ciklust.

Most vessen egy pillantást az erőforrásokat. Azt adja meg a két erőforrások:
* `loop-0`a másolási ciklust a nulla alapú erőforrás van.
* `loop-`van kibővítve eredményét a `copyIndex(1)` működnek, mint az erőforrás, kezdve egyedi iterációs alapú nevet létrehozni `1`.

Az erőforrások néznek ki:

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

Ismerkedjen meg a paramétereket, azt hogy átadni részletes bemutatása a **átalakító sablon** a beágyazott sablonban. A visszahívás korábbi, amely a `source` paraméter adja meg az aktuális objektum a `source` paraméter object tömbben. A `state` paraméter hol történik a gyűjteményhez, mert az előző munkamenetben, a másolási ciklust kimenete tart&mdash;figyelje meg, hogy a `reference()` függvény a `copyIndex()` függvény a nem a hivatkozniparaméter`name` az előző csatolt sablon objektum&mdash;, és átadja az aktuális iteráció.

Végezetül a `output` a sablon adja vissza a `output` , az utolsó lépésében a **átalakító sablon**:

```json
  "outputs": {
    "result": {
      "type": "array",
      "value": "[reference(concat('loop-', variables('count'))).outputs.collection.value]"
    }
  }
```
A megszokás vissza a `output` az utolsó lépésében a a **átalakító sablon** való a **hívó sablon** mert jelent meg azt is tárolja a `source` a paraméter. Ne feledje azonban, hogy-e az utolsó lépésében a **átalakító sablon** , amely tartalmazza a teljes átalakított tulajdonság objektumokból álló tömb, és pontosan a kívánt való visszatéréshez.

Végezetül vessen egy pillantást hogyan hívhatja meg a **adatgyűjtő sablon** a a **hívó sablon**.

## <a name="calling-template"></a>Hívása sablon

A **hívó sablon** nevű egyetlen paraméter meghatározza `networkSecurityGroupsSettings`:

```json
...
"parameters": {
    "networkSecurityGroupsSettings": {
        "type": "object"
    }
```

Ezt követően a sablon nevű egyetlen változót definiál `collectorTemplateUri`:

```json
"variables": {
    "collectorTemplateUri": "[uri(deployment().properties.templateLink.uri, 'collector.template.json')]"
  }
```

Teheti meg, mert ez az URI-JÁNAK a **adatgyűjtő sablon** , amely a csatolt sablonerőforrás használják:

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

Azt adja át a két paraméter a **adatgyűjtő sablon**:
* `source`az objektum tulajdonságtömb van. A példánkban ez a `networkSecurityGroupsSettings` paraméter.
* `transformTemplateUri`az URI-azonosítója csak meghatározott, a változó a **adatgyűjtő sablon**.

Végezetül a `Microsoft.Network/networkSecurityGroups` erőforrás közvetlenül rendeli hozzá a `output` , a `collector` kapcsolódó sablon erőforrást a `securityRules` tulajdonság:

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

## <a name="next-steps"></a>Következő lépések

* Ezzel a technikával meg van valósítva a [építőelemeket sablonprojekt](https://github.com/mspnp/template-building-blocks) és a [Azure hivatkozás architektúrák](/azure/architecture/reference-architectures/). Ezek segítségével hozzon létre egy saját architektúra, vagy telepítse a referencia-architektúrák egyikét használhatja.

<!-- links -->
[objects-as-parameters]: ./objects-as-parameters.md
[resource-manager-linked-template]: /azure/azure-resource-manager/resource-group-linked-templates
[resource-manager-variables]: /azure/azure-resource-manager/resource-group-template-functions-deployment
[nsg]: /azure/virtual-network/virtual-networks-nsg
