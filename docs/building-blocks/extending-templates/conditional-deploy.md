---
title: "Feltételesen telepítése Azure Resource Manager-sablonok az erőforráshoz"
description: "Ismerteti, hogyan lehet a paraméter értékét egy erőforrás dependending feltételesen telepítése Azure Resource Manager sablonok bővítése"
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: e911e7dc41b4f71ebfaf13a00f8cdbb5b4e2578b
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="conditionally-deploy-a-resource-in-an-azure-resource-manager-template"></a>Feltételesen telepítése Azure Resource Manager-sablonok az erőforráshoz

Léteznek olyan forgatókönyvek, amelyben kell terveznie a sablon telepítéséhez egy erőforrást egy paraméterérték megtalálható-e feltétel alapján. A sablon például telepíthet egy virtuális hálózatot és paramétereit, és adja meg azokat a társviszony-létesítéshez más virtuális hálózatokat tartalmaz. Már nincs megadva a társviszony-létesítéshez paraméterértékeket, ha nem szeretné, hogy Resource Manager telepítéséhez a társviszony-létesítési erőforrás.

Ehhez használja a [ `condition` elem] [ azure-resource-manager-condition] az erőforráskészlet a paramétertömb hosszának teszteléséhez. Az érték nulla, ha vissza `false` megakadályozzák a telepítést, de minden nullánál nagyobb értéket visszaadnia `true` való központi telepítésének engedélyezése.

## <a name="example-template"></a>Példa sablon

Azt mutatja be, hogy ez egy példa sablon vizsgáljuk meg. A sablon által a [ `condition` elem] [ azure-resource-manager-condition] a vezérlő telepítése a `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` erőforrás. Ezt az erőforrást hoz létre a társviszony-létesítés között két Azure virtuális hálózat ugyanabban a régióban.

Vessen egy pillantást a sablon minden szakasza.

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
A `virtualNetworkPeerings` paraméter egy `array` és a következő sémával rendelkezik:

```json
"virtualNetworkPeerings": [
    {
        "remoteVirtualNetwork": {
            "name": "my-other-virtual-network"
        },
        "allowForwardedTraffic": true,
        "allowGatewayTransit": true,
        "useRemoteGateways": false
    }
]
```

A paraméter tulajdonságait adja meg a [társviszony-létesítési virtuális hálózatok kapcsolatos beállítások][vnet-peering-resource-schema]. A következőkhöz nyújtunk a értékek ezekhez a tulajdonságokhoz amikor azt adja meg a `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` erőforrás a `resources` szakasz:

```json
"resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2017-05-10",
      "name": "[concat('vnp-', copyIndex())]",
      "condition": "[greater(length(parameters('virtualNetworkPeerings')), 0)]",
      "dependsOn": [
        "virtualNetworks"
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
Nincsenek elolvashatja, melyek az a sablon részében. Először a rendszerbe állítandó tényleges erőforrás egy beágyazott sablon típusú `Microsoft.Resources/deployments` , amely tartalmazza a saját sablon, amely ténylegesen telepít a `Microsoft.Network/virtualNetworks/virtualNetworkPeerings`.

A `name` a beágyazott a sablon köszönhetően egyedi aktuális ismétléseinek számától hozzáfűzésével a `copyIndex()` előtaggal `vnp-`. 

A `condition` elem megadhatja, hogy az erőforrás feldolgozott, ha a `greater()` függvény eredménye `true`. Ide, ha tesztel a `virtualNetworkPeerings` paraméter tömb `greater()` nullánál. Ha van, az eredmény `true` és a `condition` meggyőződött. Ellenkező esetben van `false`.

Azt adja meg a következő a `copy` hurok. Ez egy `serial` ciklust, amely azt jelenti, hogy a hurok sorrendben történik, az egyes erőforrások csak az utolsó Várakozás erőforrás van telepítve. A `count` tulajdonság határozza meg a hurok megismétli száma. Itt, általában állapotúra kell állítani az eltelt a `virtualNetworkPeerings` tömb, mert azt telepíteni kívánja az erőforrást meghatározó paraméter-objektumokat. Azonban nézzük meg, ha érvényesítése sikertelen lesz, ha a tömb nem üres, mert a Resource Manager azt észleli, hogy azt próbál hozzáférni, amelyek nem léteznek tulajdonságok. A Microsoft oldható meg, azonban. Vessen egy pillantást a változók kell:

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

A `workaround` változó tartalmazza a két tulajdonság, nevű `true` és nevű `false`. A `true` tulajdonság értékének kiértékeli a `virtualNetworkPeerings` paramétertömb. A `false` tulajdonság kiértékeli, beleértve az erőforrás-kezelő vár, hogy elnevezett tulajdonságok üres objektumra&mdash;vegye figyelembe, hogy `false` van ténylegesen egy tömb, ahogy a `virtualNetworkPeerings` paraméter értéke, amely eleget tesz a érvényesítése. 

A `peerings` változó használja a `workaround` ismét vizsgálatával, ha a változó hossza a `virtualNetworkPeerings` paramétertömb érték nagyobb, mint nulla. Ha igen, a `string` értékelődik ki `true` és a `workaround` a változó evalutes a `virtualNetworkPeerings` paramétertömb. Ellenkező esetben az eredmény `false` és a `workaround` változó értéke az a tömb első eleme üres objektum.

Most, hogy azt az érvényesítési probléma dolgozott, egyszerűen adható meg a központi telepítése a `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` erőforrás a beágyazott sablonban, hogy a `name` és `properties` a a `virtualNetworkPeerings` paramétertömb. Láthatja, ez csak a `template` beágyazva elem a `properties` elem az erőforrás.

## <a name="next-steps"></a>Következő lépések

* Ezzel a technikával meg van valósítva a [építőelemeket sablonprojekt](https://github.com/mspnp/template-building-blocks) és a [Azure hivatkozás architektúrák](/azure/architecture/reference-architectures/). Ezek segítségével hozzon létre egy saját architektúra, vagy telepítse a referencia-architektúrák egyikét használhatja.

<!-- links -->
[azure-resource-manager-condition]: /azure/azure-resource-manager/resource-group-authoring-templates#resources
[azure-resource-manager-variable]: /azure/azure-resource-manager/resource-group-authoring-templates#variables
[vnet-peering-resource-schema]: /azure/templates/microsoft.network/virtualnetworks/virtualnetworkpeerings