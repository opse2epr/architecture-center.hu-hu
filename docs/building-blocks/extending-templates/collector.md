---
title: Tulajdonságátalakító és -gyűjtő megvalósítása az Azure Resource Manager-sablon
description: Ismerteti, hogyan lehet egy tulajdonságátalakító és -gyűjtő megvalósítása az Azure Resource Manager-sablon.
author: petertay
ms.date: 10/30/2018
ms.topic: article
ms.service: architecture-center
ms.subservice: reference-architecture
ms.openlocfilehash: 5dc910b8577e27059761db417e6350cd38cdf2b8
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54484720"
---
# <a name="implement-a-property-transformer-and-collector-in-an-azure-resource-manager-template"></a>Tulajdonságátalakító és -gyűjtő megvalósítása az Azure Resource Manager-sablon

A [objektum használata paraméterként egy Azure Resource Manager-sablonban][objects-as-parameters], bemutattuk, hogyan erőforrás tulajdonságértékek tárolása egy objektumot, és alkalmazza őket egy resource üzembe helyezés során. Bár ez nagyon hasznos lehet kezelni a paramétereket, akkor továbbra is szükséges, hogy az objektum tulajdonságainak az erőforrás-tulajdonságok minden alkalommal, amikor használni a sablonban.

Ez elkerülhető, hogy egy tulajdonság átalakítási és -gyűjtő sablont, amely az objektum tömb végiglépkedve és alakítja azt az erőforrás által várt JSON-sémájában valósíthat meg.

> [!IMPORTANT]
> Ez a megközelítés megköveteli, hogy Ön ismeri részletesen a Resource Manager-sablonok és funkciók.

Vessünk egy pillantást is tanfolyamsorozat egy tulajdonság-gyűjtő és a egy példa, amely üzembe helyezi az átalakító egy [hálózati biztonsági csoport (NSG)][nsg]. Az alábbi ábra bemutatja a sablonokat, és ezeket a sablonokat az erőforrások közötti kapcsolat:

![tulajdonság-gyűjtő és átalakító architektúra](../_images/collector-transformer.png)

A **hívó sablon** két forrásokat tartalmazza:

- egy sablon hivatkozást, amely meghívja a **gyűjtő sablon**.
- az NSG-erőforrás üzembe helyezéséhez.

A **gyűjtő sablon** két forrásokat tartalmazza:

- egy **forráshorgony** erőforrás.
- sablon hivatkozás, amely egy másolási ciklust átalakító sablonnal hív meg.

A **átalakító sablon** egyetlen erőforrást tartalmaz: egy üres sablonnal, amely átalakítja a a `source` a JSON-séma szerint az NSG-erőforrás a várt JSON-a **fő sablon**.

## <a name="parameter-object"></a>A paraméter objektum

Fogjuk használni a `securityRules` paraméter objektumot [paraméterekként objektumok][objects-as-parameters]. A **átalakító sablon** átalakítja az egyes objektumok a `securityRules` az NSG-erőforrás által várt JSON-sémájában tömböt a **hívó sablon**.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
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

Nézzük meg a **átalakító sablon** első.

## <a name="transform-template"></a>Sablonok átalakítása

A **átalakító sablon** két az átadott paramétereket tartalmaz az **gyűjtő sablon**:

- `source` egy olyan objektum, amely fogad egy tulajdonság értékének objektum a tulajdonság tömbből. Ebben a példában minden egyes objektum a `"securityRules"` tömb átadása egyenként.
- `state` van egy tömb, amely megkapja a korábbi átalakítások összefűzött eredményeit. Ez az átalakított JSON gyűjteménye.

A paraméterek néznek ki:

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

A sablon is meghatároz egy nevű változó `instance`. Hajtja végre, a tényleges átalakítás a `source` objektumot a szükséges JSON-sémájában be:

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

Végül a `output` a sablon fűzi össze az összegyűjtött átalakítások, a `state` az aktuális átalakítás végzi paraméter a `instance` változó:

```json
    "resources": [],
    "outputs": {
    "collection": {
      "type": "array",
      "value": "[concat(parameters('state'), variables('instance'))]"
    }
```

Ezután vessünk egy pillantást a **gyűjtő sablon** megtekintéséhez, hogyan továbbítja a paraméterértékeket.

## <a name="collector-template"></a>Gyűjtő sablon

A **gyűjtő sablon** három paramétereket tartalmazza:

- `source` az objektum teljes paramétertömböt van. A által átadott a **hívó sablon**. Ez rendelkezik-e a neve megegyezik a `source` paramétert a **átalakító sablon** , de talán már észrevette, hogy egy fő különbséggel: a teljes tömb, de csak átadott tömb egy elemet a **átalakító sablon** egyszerre.
- `transformTemplateUri` az URI-t, a **átalakító sablon**. A sablon újrahasznosíthatóság itt paramétert azt most definiálja.
- `state` adjuk át, hogy kezdetben üres tömb a **átalakító sablon**. A másolási ciklus befejezésekor az átalakított paraméter objektumok gyűjteményét tárolja.

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

Ezt követően adja meg azt az nevű változó `count`. A érték hossza pedig a `source` objektum Pole parametru:

```json
  "variables": {
    "count": "[length(parameters('source'))]"
  },
```

Mivel előfordulhat, hogy azt gyanítja, használja azt az ismétlések számát a másolási ciklust.

Most vessünk egy pillantást az erőforrások. Azt adja meg a két erőforrás:

- `loop-0` a másolási ciklust nulla-alapú erőforrás-van.
- `loop-` eredménye van kibővítve a `copyIndex(1)` függvény használatával létrehoz egy egyedi iteráció-alapú nevet az erőforráshoz, kezdve `1`.

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

Vizsgáljuk meg közelebbről a azt Ön átadása paramétereket a **átalakító sablon** a beágyazott sablonban. A korábban emlékeztetnek arra, hogy a `source` paraméter adja meg az aktuális objektum a `source` objektum Pole parametru. A `state` paraméter hol történik a gyűjteményt, mert tart a másolási ciklust előző lépésében kimenete&mdash;figyelje meg, hogy a `reference()` függvény a `copyIndex()` függvény paraméterek nélkül való hivatkozáshoz a `name` az előző társított sablonobjektum&mdash;, és átadja a jelenlegi verzió továbbfejlesztésében.

Végül a `output` a sablon adja vissza a `output` az utolsó ismétlése a **átalakító sablon**:

```json
  "outputs": {
    "result": {
      "type": "array",
      "value": "[reference(concat('loop-', variables('count'))).outputs.collection.value]"
    }
  }
```

Visszaadandó counterintuitive tűnhet a `output` az utolsó ismétlése a **átalakító sablon** , a **hívó sablon** jelent meg tudjuk tárolta, mert a `source` a paraméter. Ne feledje azonban, hogy-e az utolsó ismétlése a **átalakító sablon** , amely tartalmazza a teljes átalakított tulajdonság objektumtömb, és ez pontosan a kívánt való visszatéréshez.

Végül tekintsük át, hogyan lehet meghívni a **gyűjtő sablon** származó a **hívó sablon**.

## <a name="calling-template"></a>Hívása a sablon

A **hívó sablon** nevű egyetlen paraméter meghatározása `networkSecurityGroupsSettings`:

```json
...
"parameters": {
    "networkSecurityGroupsSettings": {
        "type": "object"
    }
```

Következő lépésként határozza meg a sablont a nevű változóhoz `collectorTemplateUri`:

```json
"variables": {
    "collectorTemplateUri": "[uri(deployment().properties.templateLink.uri, 'collector.template.json')]"
  }
```

Hiányol, mivel ez az URI-JÁNAK a **gyűjtő sablon** , amelyek a társított sablonerőforrás használják:

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

Adjuk át, hogy a két paramétert a **gyűjtő sablon**:

- `source` az objektum tulajdonságtömb van. Ebben a példában ez a `networkSecurityGroupsSettings` paraméter.
- `transformTemplateUri` a változó csak meghatározott az URI-ját a **gyűjtő sablon**.

Végül a `Microsoft.Network/networkSecurityGroups` erőforrás közvetlenül rendeli hozzá a `output` , a `collector` sablonerőforrás hogy társított annak `securityRules` tulajdonság:

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

## <a name="try-the-template"></a>A sablon kipróbálása

Egy példa sablon érhető el az [GitHub][github]. A sablon üzembe helyezéséhez, klónozza az adattárat, és futtassa a következő [Azure CLI-vel] [ cli] parancsokat:

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
