---
title: Objektum használata paraméterként az Azure Resource Manager-sablon
description: Ismerteti, hogyan lehet az Azure Resource Manager-sablonok objektum használata paraméterként funkcióinak bővítése érdekében.
author: petertay
ms.date: 10/30/2018
ms.openlocfilehash: f0826d8ed1ce446d295ebdacc66d8b0bef0b0dec
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/08/2019
ms.locfileid: "54111206"
---
# <a name="use-an-object-as-a-parameter-in-an-azure-resource-manager-template"></a>Objektum használata paraméterként az Azure Resource Manager-sablon

Ha Ön [Azure Resource Manager-sablonok készítése][azure-resource-manager-create-template], lehetősége van adja meg az erőforrás-tulajdonság értékeinek közvetlenül a sablonban vagy adjon meg paramétert, majd adja meg az értékeket üzembe helyezés során. Nyugodtan mindegyik tulajdonság értékét a kisméretű környezetek egy paraméter használható, de egy legfeljebb 255 paraméterek száma üzemelő példányonként. Miután a nagyobb és összetettebb üzembe helyezések paraméterei tartományon kívül is futtathat.

A probléma megoldásához egyik módja az objektum használata paraméterként egy érték helyett. Ehhez adja meg a paramétert a sablont, és adja meg a JSON-objektum egyetlen érték helyett üzembe helyezés során. Ezután hivatkozhat a altulajdonságokat a paraméter használatával a [ `parameter()` függvény] [ azure-resource-manager-functions] és a sablon pont operátort.

Vessünk egy pillantást egy példa, amely üzembe helyez egy virtuális hálózati erőforrás. Először tekintsük meg a `VNetSettings` paramétert a sablon és a `type` való `object`:

```json
...
"parameters": {
    "VNetSettings":{"type":"object"}
},
```

Következő lépésként hozzunk adja meg az értékeket a `VNetSettings` objektum:

> [!NOTE]
> Ismerje meg, hogyan adja meg a paraméterértékeket üzembe helyezés során, tekintse meg a **paraméterek** szakaszában [struktúra és az Azure Resource Manager-sablonok szintaxisát] [ azure-resource-manager-authoring-templates] .

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

Amint látható, az egyetlen ténylegesen paraméterrel három altulajdonságok: `name`, `addressPrefixes`, és `subnets`. Ezek altulajdonságok mindegyike vagy értéket vagy más altulajdonságok adja meg. Az eredménye, hogy az egyetlen paramétert adja meg a virtuális hálózat üzembe helyezéséhez szükséges összes értékét.

Most nézzük meg, hogy a sablon a `VNetSettings` objektumot használja:

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

Értékét a `VNetSettings` objektum érvénybe lépnek a tulajdonságokat, a virtuális hálózati erőforrás használata szükséges a `parameters()` mindkét függvény a `[]` indexer, és a pont operátor a tömb. Ez a megközelítés akkor működik, ha csak át szeretné statikusan Objekt paraméter értékét az erőforrásra alkalmazni kívánt. Azonban ha dinamikusan üzembe helyezés során a tulajdonság egy olyan értéktömböt hozzárendelni kívánt használhatja egy [másolási ciklust][azure-resource-manager-create-multiple-instances]. A másolási ciklust használandó értékeket ad meg egy JSON-tömböt erőforrás tulajdonság és a másolási ciklust dinamikusan vonatkozik az értékeket az erőforrás-tulajdonságok.

Érdemes figyelembe vennie a dinamikus megközelítés használatakor egy probléma van. A probléma bemutatása érdekében vessünk egy pillantást egy tipikus tulajdonság értékek tömbje. Ebben a példában a tulajdonságok értékeit egy változó vannak tárolva. Figyelje meg, hogy van két Tárolótömböket Itt&mdash;gyermektartománya `firstProperty` és a egy nevű `secondProperty`.

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

Most vessünk egy pillantást a változó segítségével egy másolási ciklust tulajdonság el módja.

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

A `copyIndex()` függvény az aktuális a ciklus ismétléseinek számától példányt adja vissza, és használjuk, amely index két tömb minden egyes egyszerre.

Ez jól működik az, ha a két tömbök ugyanolyan hosszúságú. A probléma merül fel, ha már állított be, és a két tömbök különböző hosszúságú&mdash;ebben az esetben a sablon üzembe helyezése során érvényesítése sikertelen lesz. Mivel sokkal egyszerűbb, ha egy érték nem hiányzik, akkor a probléma elkerülése érdekében egyetlen objektumot, beleértve az összes tulajdonság által. Például, tekintsük át egy másik paraméter objektum mely minden eleme a `propertyObject` Pole je Uniója a `firstProperty` és `secondProperty` korábban a tömbök.

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

Figyelje meg, hogy a harmadik elem a tömbben található? Hiányzik a `number` tulajdonság, de vegye figyelembe, hogy korábban kihagyta, ha Ön szerzőként paraméter értékét így sokkal könnyebb a.

## <a name="using-a-property-object-in-a-copy-loop"></a>A másolási ciklust tulajdonság objektum használatával

Ez a megközelítés akkor lesz még több hasznos, kombinálva a [soros másolási ciklust] [azure-resource-manager-létrehozása-több], különösen a gyermek-erőforrások üzembe helyezése.

Ennek nézzük meg a sablont, amely üzembe helyez egy [hálózati biztonsági csoport (NSG)] [ nsg] két kapcsolatbiztonsági szabályait.

Először vessünk egy pillantást a paramétereket. Ha megnézzük a sablont, láthatjuk, hogy meghatároztuk nevű egy paraméter `networkSecurityGroupsSettings` , amely tartalmaz egy tömb nevű `securityRules`. A tömb két JSON-objektumok beállításainak biztonsági szabályok számát tartalmazza.

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

Most vessünk egy pillantást sablont. Az első resource nevű `NSG1` az NSG-t telepíti. A második erőforrás nevű `loop-0` két funkciókat hajtja végre: először azt `dependsOn` az NSG-t, a telepítés nem indul el, amíg `NSG1` befejeződött, és az első példányát, a szekvenciális hurok. A harmadik erőforrás egy beágyazott sablont, amely üzembe helyezi a biztonsági szabályok használatával egy objektumot a paraméterértékeket, az utolsó példában látható módon.

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

Vessünk hogyan adjuk meg a tulajdonság a közelebbről a `securityRules` gyermek-erőforrás. A tulajdonságokat hivatkozott használatával a `parameter()` függvény, utána pedig a pont operátor használata hivatkozhat a `securityRules` tömb, a jelenlegi érték meghaladja az iteráció által indexelt. Egy másik pont operátor, az objektum nevére hivatkozhatnak használjuk.

## <a name="try-the-template"></a>A sablon kipróbálása

Egy példa sablon érhető el az [GitHub][github]. A sablon üzembe helyezéséhez, klónozza az adattárat, és futtassa a következő [Azure CLI-vel] [ cli] parancsokat:

```bash
git clone https://github.com/mspnp/template-examples.git
cd template-examples/example3-object-param
az group create --location <location> --name <resource-group-name>
az group deployment create -g <resource-group-name> \
    --template-uri https://raw.githubusercontent.com/mspnp/template-examples/master/example3-object-param/deploy.json \
    --parameters deploy.parameters.json
```

## <a name="next-steps"></a>További lépések

- Ismerje meg, hogyan hozhat létre egy sablont, amely végighalad a objektum tömböt és egy JSON-sémájában alakítja azt. Lásd: [egy tulajdonságátalakító és -gyűjtő megvalósítása az Azure Resource Manager-sablon](./collector.md)

<!-- links -->

[azure-resource-manager-authoring-templates]: /azure/azure-resource-manager/resource-group-authoring-templates
[azure-resource-manager-create-template]: /azure/azure-resource-manager/resource-manager-create-first-template
[azure-resource-manager-create-multiple-instances]: /azure/azure-resource-manager/resource-group-create-multiple
[azure-resource-manager-functions]: /azure/azure-resource-manager/resource-group-template-functions-resource
[nsg]: /azure/virtual-network/virtual-networks-nsg
[cli]: /cli/azure/?view=azure-cli-latest
[github]: https://github.com/mspnp/template-examples
