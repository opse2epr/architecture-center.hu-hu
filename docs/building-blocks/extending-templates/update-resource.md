---
title: Az Azure Resource Manager-sablon egy erőforrás frissítése
description: Ismerteti, hogyan lehet egy erőforrás frissítése az Azure Resource Manager-sablonok bővítése
author: petertay
ms.date: 10/31/2018
ms.openlocfilehash: dc97534e658c9728ac617b4e52031e2553600458
ms.sourcegitcommit: e9eb2b895037da0633ef3ccebdea2fcce047620f
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 10/30/2018
ms.locfileid: "50251821"
---
# <a name="update-a-resource-in-an-azure-resource-manager-template"></a>Az Azure Resource Manager-sablon egy erőforrás frissítése

Léteznek olyan forgatókönyvek, ahol frissítenie kell egy erőforrás egy üzembe helyezés során. Ebben a forgatókönyvben egy erőforráshoz tartozó összes tulajdonság nem adható meg, amíg nem más, a függő erőforrások jönnek létre előforduló. A terheléselosztó háttérbeli készletet hoz létre, előfordulhat, hogy a virtuális gépeken (VM), hogy felveszi a háttérkészletben például frissítse a hálózati adapterek (NIC). És a Resource Manager támogatja a frissítési erőforrások üzembe helyezése során, míg kell tervezzen a sablont megfelelően hibák elkerülése érdekében, és győződjön meg arról, a központi telepítés kezelése frissítésként.

Először is hivatkoznia kell az erőforrás egyszer hozza létre, és ezután hivatkozhat ugyanazzal a névvel, azt később frissíteni az erőforrás a sablonban. Azonban ha a két erőforrás rendelkezik egy sablon ugyanazzal a névvel, erőforrás-kezelő kivételt jelez. Ez a hiba elkerülése érdekében adja meg a frissített erőforrás van csatolva, vagy néven egy subtemplate a második sablon a `Microsoft.Resources/deployments` erőforrástípus.

A második vagy adjon meg módosítani a meglévő tulajdonság neve vagy egy tulajdonság hozzáadása a beágyazott sablon új nevét. Is meg kell adnia az eredeti tulajdonságok és az eredeti értékekre is. Ha nem adja meg az eredeti tulajdonságait és értékeit, a Resource Manager feltételezi, hogy hozzon létre egy új erőforrást szeretne, és törli az eredeti erőforrás.

## <a name="example-template"></a>Példasablon

Lássunk erre egy példa sablon azt mutatja be, ezzel. A sablon üzembe helyez egy nevű virtuális hálózatot `firstVNet` nevű alhálózattal rendelkező `firstSubnet`. Ezután üzembe helyez egy virtuális hálózati adapter (NIC) nevű `nic1` , és társítja azt az alhálózatot. Ezt követően egy központi telepítési erőforrás nevű `updateVNet` tartalmaz egy beágyazott sablont, amely frissíti a `firstVNet` adja hozzá egy második alhálózatot nevű erőforrás `secondSubnet`. 

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {},
  "resources": [
      {
      "apiVersion": "2016-03-30",
      "name": "firstVNet",
      "location":"[resourceGroup().location]",
      "type": "Microsoft.Network/virtualNetworks",
      "properties": {
          "addressSpace":{"addressPrefixes": [
              "10.0.0.0/22"
          ]},
          "subnets":[              
              {
                  "name":"firstSubnet",
                  "properties":{
                    "addressPrefix":"10.0.0.0/24"
                  }
              }
            ]
      }
    },
    {
        "apiVersion": "2015-06-15",
        "type":"Microsoft.Network/networkInterfaces",
        "name":"nic1",
        "location":"[resourceGroup().location]",
        "dependsOn": [
            "firstVNet"
        ],
        "properties": {
            "ipConfigurations":[
                {
                    "name":"ipconfig1",
                    "properties": {
                        "privateIPAllocationMethod":"Dynamic",
                        "subnet": {
                            "id": "[concat(resourceId('Microsoft.Network/virtualNetworks','firstVNet'),'/subnets/firstSubnet')]"
                        }
                    }
                }
            ]
        }
    },
    {
      "apiVersion": "2015-01-01",
      "type": "Microsoft.Resources/deployments",
      "name": "updateVNet",
      "dependsOn": [
          "nic1"
      ],
      "properties": {
        "mode": "Incremental",
        "parameters": {},
        "template": {
          "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {},
          "variables": {},
          "resources": [
              {
                  "apiVersion": "2016-03-30",
                  "name": "firstVNet",
                  "location":"[resourceGroup().location]",
                  "type": "Microsoft.Network/virtualNetworks",
                  "properties": {
                      "addressSpace": "[reference('firstVNet').addressSpace]",
                      "subnets":[
                          {
                              "name":"[reference('firstVNet').subnets[0].name]",
                              "properties":{
                                  "addressPrefix":"[reference('firstVNet').subnets[0].properties.addressPrefix]"
                                  }
                          },
                          {
                              "name":"secondSubnet",
                              "properties":{
                                  "addressPrefix":"10.0.1.0/24"
                                  }
                          }
                     ]
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

Vessünk egy pillantást az erőforrás-objektumokat a `firstVNet` erőforrás első. Figyelje meg, hogy helyes-e a beállításait az `firstVNet` beágyazott sablonban&mdash;ennek az az oka az erőforrás-kezelő nem engedélyezi a központi telepítési névvel belül ugyanazt a sablont, és beágyazott sablonok másik sablont kell tekinteni. Az értékek respecifying által a `firstSubnet` erőforrás, azt mondjuk neki, erőforrás-kezelő törlését és újbóli üzembe helyezés helyett a meglévő erőforrás frissítése. Végül az új beállítások a `secondSubnet` mértékének a frissítés közben.

## <a name="try-the-template"></a>A sablon kipróbálása

Egy példa sablon érhető el az [GitHub][github]. A sablon üzembe helyezéséhez futtassa a következő [Azure CLI-vel] [ cli] parancsokat:

```bash
az group create --location <location> --name <resource-group-name>
az group deployment create -g <resource-group-name> \
    --template-uri https://raw.githubusercontent.com/mspnp/template-examples/master/example1-update/deploy.json
```

Miután a telepítés véget ért, nyissa meg az erőforráscsoportot a portálon megadott. Megjelenik egy nevű virtuális hálózatot `firstVNet` és a egy hálózati Adaptert `nic1`. Kattintson a `firstVNet`, majd kattintson a `subnets`. Megjelenik a `firstSubnet` , amely eredetileg létrehozták, és megjelenik a `secondSubnet` hozzáadott a `updateVNet` erőforrás. 

![Eredeti alhálózat és a frissített alhálózat](../_images/firstVNet-subnets.png)

Ezután lépjen vissza az erőforráscsoportot, és kattintson a `nic1` kattintson `IP configurations`. Az a `IP configurations` szakaszban a `subnet` értékre van állítva `firstSubnet (10.0.0.0/24)`. 

![nic1 IP-konfiguráció beállításai](../_images/nic1-ipconfigurations.png)

Az eredeti `firstVNet` helyett a rendszer frissítette újra létre kell hozni. Ha `firstVNet` kellett újból létrejött, `nic1` akkor nem kell társítani `firstVNet`.

## <a name="next-steps"></a>További lépések

* Ismerje meg, hogyan telepítsék központilag az erőforrás, például e jelen-e a paraméter értékét egy feltétel alapján. Lásd: [erőforrás feltételes üzembe helyezése az Azure Resource Manager-sablon](./conditional-deploy.md).

[cli]: /cli/azure/?view=azure-cli-latest
[github]: https://github.com/mspnp/template-examples