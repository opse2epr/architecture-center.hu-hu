---
title: Erőforrás feltételes üzembe helyezése az Azure Resource Manager-sablon
description: Ismerteti, hogyan lehet feltételes üzembe egy erőforrás dependending paraméter értékét az Azure Resource Manager-sablonok bővítése.
author: petertay
ms.date: 10/30/2018
ms.topic: article
ms.service: architecture-center
ms.subservice: reference-architecture
ms.openlocfilehash: f3d22c6437cdabcd93a781ecf7c99db5a570d7cf
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298881"
---
# <a name="conditionally-deploy-a-resource-in-an-azure-resource-manager-template"></a>Erőforrás feltételes üzembe helyezése az Azure Resource Manager-sablon

Van néhány olyan forgatókönyvet, amelyben kell terveznie a sablon üzembe helyezéséhez egy erőforrást, egy feltételt, például a jelen-e a paraméter értéke alapján. A sablon például üzembe helyezése egy virtuális hálózatot és tartalmazza a paraméterek használatával adja meg a más virtuális hálózatok társviszony-létesítéshez. Ha már nem a társviszony-létesítéshez paraméterértékeket, a társviszony-létesítési erőforrások üzembe helyezése erőforrás-kezelő nem szeretné.

Ehhez használja a [feltétel elem] [ azure-resource-manager-condition] teszteléséhez a paramétertömböt hossza az erőforrásban. Ha az érték nulla, lépjen vissza `false` megakadályozzák a telepítést, de az összes nullánál nagyobb értéket ad vissza `true` való központi telepítésének engedélyezése.

## <a name="example-template"></a>Példasablon

Lássunk erre egy példa sablon azt mutatja be, ezzel. A sablon által a [feltétel elem] [ azure-resource-manager-condition] a vezérlő üzembe helyezéséhez a `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` erőforrás. Ezt az erőforrást hoz létre, egy társviszony-létesítés két Azure virtuális hálózat ugyanabban a régióban között.

Vessünk egy pillantást a sablon minden szakasza.

A `parameters` elem definiálja nevű egyetlen paraméter `virtualNetworkPeerings`:

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

A `virtualNetworkPeerings` paraméter egy `array` , és rendelkezik a következő mintát követik:

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

A tulajdonságok a paraméterben adja meg a [társviszony-létesítés virtuális hálózatok kapcsolatos beállítások][vnet-peering-resource-schema]. Biztosítunk, amellyel az értékeket ezekhez a tulajdonságokhoz adja meg, hogy amikor a `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` erőforrást a `resources` szakaszban:

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

Van néhány dolog bevezetése a sablont ezen részében. Először a tényleges erőforrás üzembe-e egy beágyazott sablon típusú `Microsoft.Resources/deployments` , amely tartalmazza a saját sablont, amely Végezetül ténylegesen telepíti a `Microsoft.Network/virtualNetworks/virtualNetworkPeerings`.

A `name` a beágyazott sablont készítette egyedi az aktuális ismétlése összetűzésének a `copyIndex()` előtagot `vnp-`.

A `condition` elem azt határozza meg, hogy legyen-e az erőforrás feldolgozott mikor a `greater()` függvény `true`. Itt, ha tesztel a `virtualNetworkPeerings` paraméternek tömb `greater()` , mint nulla. Ha igen, az eredmény `true` és a `condition` teljesült. Ellenkező esetben rendelkezik `false`.

Ezután azt adja meg a `copy` ciklus. Ez egy `serial` ciklust, amely azt jelenti, hogy a hurok sorrendben történik, az egyes erőforrások csak az utolsó Várakozás az erőforrás üzembe helyezve. A `count` tulajdonság határozza meg, hogy hányszor a hurok ismétlődik. Itt, általában azt értékre kell állítania azt a hosszát a `virtualNetworkPeerings` tömböt, mert a konfigurációsparaméter-objektumokat adja meg a telepíteni kívánt erőforrást tartalmaz. Azonban ha még a nem érvényesítése sikertelen lesz, ha a tömb üres, mert a Resource Manager azt észleli, hogy tudjuk elérni kívánt tulajdonságok, amelyek nem léteznek. Hogy oldható meg, azonban. Vessünk egy pillantást a változókat, szüksége lesz:

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

A `workaround` változó tartalmazza a két tulajdonság, egy nevű `true` és a egy nevű `false`. A `true` tulajdonság kifejezés értékét adja vissza a `virtualNetworkPeerings` Pole parametru. A `false` tulajdonság egy üres objektumot, beleértve a Resource Manager vár, hogy elnevezett tulajdonságok kiértékelése &mdash; vegye figyelembe, hogy `false` van ténylegesen egy tömb, ugyanúgy, mint a `virtualNetworkPeerings` paraméter értéke, amely eleget tesz a érvényesítése.

A `peerings` változót használ a `workaround` ismét vizsgálatával, ha a változó hossza a `virtualNetworkPeerings` Pole parametru értéke nagyobb, mint nulla. Ha igen, a `string` értékelődik ki `true` és a `workaround` változó értéke a `virtualNetworkPeerings` Pole parametru. Ellenkező esetben való kiértékelése által `false` és a `workaround` változó kiértékeli a tömb első eleme az üres objektumhoz.

Most, hogy megkönnyítsük már az érvényesítési probléma, azt is egyszerűen adja meg a központi a `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` erőforrás átadása, a beágyazott sablonban a `name` és `properties` származó a `virtualNetworkPeerings` Pole parametru. Láthatja, ez az a `template` beágyazott elem a `properties` elem az erőforrás.

## <a name="try-the-template"></a>A sablon kipróbálása

Egy példa sablon érhető el az [GitHub][github]. A sablon üzembe helyezéséhez futtassa a következő [Azure CLI-vel] [ cli] parancsokat:

```bash
az group create --location <location> --name <resource-group-name>
az group deployment create -g <resource-group-name> \
    --template-uri https://raw.githubusercontent.com/mspnp/template-examples/master/example2-conditional/deploy.json
```

## <a name="next-steps"></a>További lépések

* Objektumok használata helyett skaláris értékek sablon paraméterekként. Lásd: [objektum használata paraméterként az Azure Resource Manager-sablon](./objects-as-parameters.md)

<!-- links -->
[azure-resource-manager-condition]: /azure/azure-resource-manager/resource-manager-templates-resources#condition
[azure-resource-manager-variable]: /azure/azure-resource-manager/resource-group-authoring-templates#variables
[vnet-peering-resource-schema]: /azure/templates/microsoft.network/virtualnetworks/virtualnetworkpeerings
[cli]: /cli/azure/?view=azure-cli-latest
[github]: https://github.com/mspnp/template-examples
