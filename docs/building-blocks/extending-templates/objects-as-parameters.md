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
ms.locfileid: "30847175"
---
# <a name="use-an-object-as-a-parameter-in-an-azure-resource-manager-template"></a>Az objektum használata Azure Resource Manager-sablon egyik paraméterének

Ha Ön [Azure Resource Manager-sablonok készítésének][azure-resource-manager-create-template], lehetősége van adja meg az erőforrás-tulajdonság értékeinek közvetlenül a sablonban vagy határozzon meg egy paramétert, majd adjon meg értékeket a telepítés során. Beleegyezik abba, hogy minden egyes tulajdonság értéke kisebb környezetekben a paraméter használható, de egy legfeljebb 255 paraméterek üzemelő példányonként. Miután nagyobb és összetettebb központi paraméterek kívül is futtathatja.

Egy probléma megoldásához módja egy objektum használata helyett értéket paraméterként. Ehhez a paraméter a sablonban megadott, és adjon meg egyetlen érték helyett egy JSON-objektum üzembe helyezése során. A paraméter használatával altulajdonságok, majd hivatkozzon a [ `parameter()` függvény] [ azure-resource-manager-functions] és pont operátort a sablonban.

Vessen egy pillantást egy példa egy virtuális hálózati erőforráshoz. Először adjuk meg a `VNetSettings` paraméter a sablon és a `type` való `object`:

```json
...
"parameters": {
    "VNetSettings":{"type":"object"}
},
```
A következő most adja meg az értékeket a `VNetSettings` objektum:

> [!NOTE]
> Paraméter értékének megadására, deploment közben, lásd: a **paraméterek** szakasza [megérteni a felépítését és Azure Resource Manager-sablonok szintaxisát][azure-resource-manager-authoring-templates]. 

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

Ahogy látja, az egyetlen ténylegesen paraméterrel három altulajdonságok: `name`, `addressPrefixes`, és `subnets`. Ezek altulajdonságok mindegyikének vagy értékre, vagy más altulajdonságok határozza meg. Az eredménye, hogy az egyetlen paramétert határoz meg a virtuális hálózati telepítéséhez szükséges összes értéket.

Most tegyük rendelkezik egy pillantást a sablont, tekintse meg a többi bemutatja a `VNetSettings` objektummal:

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
Értékeit a `VNetSettings` objektum által a virtuális hálózati erőforrás használata szükséges tulajdonságot is vonatkozik a `parameters()` mindkét függvény a `[]` tömb indexelő és a pont operátort. Ezt a módszert akkor működik, ha szeretné az értékeket a paraméter objektum statikusan alkalmazása az erőforrás. Azonban ha a telepítés során a tulajdonságértékek tömb dinamikusan hozzárendelni kívánt használhatja egy [másolási ciklust][azure-resource-manager-create-multiple-instances]. A másolási ciklust használatához értékeket ad meg egy JSON-tömb erőforrás tulajdonság, és a másolási ciklust dinamikusan vonatkozik az értékeket az erőforrás-tulajdonságokat. 

Vegye figyelembe a dinamikus módszer használatakor az egyik megoldandó probléma van. Bemutatják a problémát, vessen egy pillantást a tulajdonságértékek tipikus tömb. Ebben a példában a tulajdonságok értékeit tárolják a változóban. Két tudunk értesítést tömbállandó Itt&mdash;olyan gyermektartománya `firstProperty` és nevű `secondProperty`. 

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

Most vessen egy pillantást a módját a tulajdonságai, a változóban a másolási ciklust használatával érhetők el azt.

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

A `copyIndex()` függvény az aktuális a ciklus ismétléseinek számától másolása, és használjuk, amely index mindegyik előálló egyidejűleg.

Ez a két tömb hosszának esetén remekül működik. A probléma merül fel, ha hiba végrehajtott, és a két tömbnek különböző hosszúságú&mdash;ebben az esetben a sablon üzembe helyezése során érvényesítése sikertelen lesz. Egyetlen objektum összes tulajdonságának belefoglalja probléma elkerülheti, mert sokkal egyszerűbb, ha az érték nem hiányzik. Például Ismerkedjen meg a paraméter egy másik objektum mely minden eleme a `propertyObject` unióját a `firstProperty` és `secondProperty` a korábbi tömbök.

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

Figyelje meg a harmadik a tömb elemei? Hiányzik a `number` tulajdonságot, de a sokkal egyszerűbb, figyelje meg, hogy már nem fogadta, ha éppen szerzőként paraméterértékek ily módon.

## <a name="using-a-property-object-in-a-copy-loop"></a>A másolási ciklust tulajdonság objektum használatával

Ez a megközelítés válik még több akkor hasznos, ha a [soros másolási ciklust] kombinálva [azure-erőforrás-kezelő-létrehozása – több], különösen a gyermek erőforrásokat üzembe helyezi. 

Ennek vizsgáljuk meg a sablont, amely telepíti egy [hálózati biztonsági csoport (NSG)] [ nsg] két kapcsolatbiztonsági szabályait. 

Vessen egy pillantást a paramétereket. Ha úgy tekintünk, a sablon megtanulhatja, hogy azt meghatározta nevű egy paraméter `networkSecurityGroupsSettings` , amely tartalmazza egy tömb nevű `securityRules`. A tömb két JSON olyan objektumokat tartalmaz, adja meg a biztonsági szabály beállításait egy számát.

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

Most vessen egy pillantást a sablont. Az első nevű erőforrás `NSG1` az NSG-t telepíti. A második nevű erőforrás `loop-0` két funkciókat hajtja végre: első, azt `dependsOn` az NSG-t, a telepítés nem indul el, amíg `NSG1` befejeződött, és ez az első a ciklus ismétléseinek számától szekvenciális. A harmadik erőforrás egy beágyazott sablont, amely a biztonsági szabályok objektum az az utolsó példában látható módon a paraméterértékeket telepít.

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

Megtudhatja, hogy hogyan adtuk meg, hogy a tulajdonság értékek részletes bemutatása a `securityRules` gyermek-erőforrás. A tulajdonságokat hivatkozott használatával a `parameter()` függvény, majd azt a pontot operátor használata hivatkozik a `securityRules` tömb indexelik ismétléseinek aktuális értéke. Végezetül használatával egy másik pont operátor hivatkoznak az objektum neve. 

## <a name="try-the-template"></a>Próbálja meg a sablon

Ha azt szeretné, ez a sablon kísérletezhet, kövesse az alábbi lépéseket: 

1.  Navigáljon az Azure portálon, válassza a **+** ikonra, és keresse meg a **sablon-üzembehelyezés** erőforrás írja be, és válassza ki azt.
2.  Keresse meg a **sablon-üzembehelyezés** lapon jelölje be a **létrehozása** gombra. Ezzel a gombbal megnyithatja a **egyéni telepítési** panelen.
3.  Válassza ki a **sablon szerkesztése** gombra.
4.  Az üres sablon törlésére. 
5.  Másolja és illessze be a mintasablon a jobb oldali ablaktáblán.
6.  Válassza ki a **mentése** gombra.
7.  Amikor a rendszer visszairányítja a **egyéni telepítési** ablaktáblán válassza előbb a **paraméterek szerkesztése** gombra.
8.  Az a **paraméterek szerkesztése** panelen, törölje a meglévő sablon.
9.  Másolja és illessze be a fenti mintasablon paraméter.
10. Válassza ki a **mentése** gomb, ami visszaadja, hogy a **egyéni telepítési** panelen.
11. Az a **egyéni telepítési** panelen jelölje ki az előfizetését, vagy létrehozhat új vagy meglévő erőforráscsoportot használni, és jelöljön ki egy helyet. Tekintse át a használati feltételeket, és válassza ki a **elfogadom** jelölőnégyzetet.
12. Válassza ki a **beszerzési** gombra.

## <a name="next-steps"></a>További lépések

* Ezek a technológiák megvalósítása után bővítheti a [tulajdonság objektum átalakító és adatgyűjtő](./collector.md). Az átalakító és adatgyűjtő technikák általános és a sablonok alapján lehet társítani.
* Ezzel a módszerrel is implementálva van a [építőelemeket sablonprojekt](https://github.com/mspnp/template-building-blocks) és a [Azure hivatkozás architektúrák](/azure/architecture/reference-architectures/). A sablonok milyen azt már megvalósított ezzel a módszerrel tekintheti meg.

<!-- links -->
[azure-resource-manager-authoring-templates]: /azure/azure-resource-manager/resource-group-authoring-templates
[azure-resource-manager-create-template]: /azure/azure-resource-manager/resource-manager-create-first-template
[azure-resource-manager-create-multiple-instances]: /azure/azure-resource-manager/resource-group-create-multiple
[azure-resource-manager-functions]: /azure/azure-resource-manager/resource-group-template-functions-resource
[nsg]: /azure/virtual-network/virtual-networks-nsg