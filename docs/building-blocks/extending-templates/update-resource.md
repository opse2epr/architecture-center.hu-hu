---
title: Frissítés Azure Resource Manager-sablonok az erőforráshoz
description: Ismerteti, hogyan lehet Azure Resource Manager sablonok egy erőforrás frissítése bővítése
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: fc2565819c66ee7695224ef5793e7276e6e552e0
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
ms.locfileid: "24538473"
---
# <a name="update-a-resource-in-an-azure-resource-manager-template"></a>Frissítés Azure Resource Manager-sablonok az erőforráshoz

Léteznek olyan forgatókönyvek, amelyben frissítenie kell egy erőforrást egy központi telepítése során. Ebben a forgatókönyvben használatnál erőforrás összes tulajdonság nem adható meg, amíg más, a függő erőforrások jönnek létre. Terheléselosztó háttérkészletének hoz létre, előfordulhat, hogy a virtuális gépek (VM) részt a háttérkészlet például frissítse a hálózati adapterek (NIC). És erőforrás-kezelő frissítése erőforrások támogatja a telepítés során, amíg meg kell kialakítása a sablont megfelelően hibák elkerülése érdekében, és győződjön meg arról, a központi telepítés kezelése frissítéseként.

Először hivatkoznia kell az erőforrás egyszer hozza létre, és majd hivatkozzon a erőforrás ugyanazzal a névvel, a frissítés később a sablonban. Azonban ha két azonos nevű sablonban, erőforrás-kezelő kivételt jelez. Ilyen hibák elkerülése érdekében adja meg a frissített erőforrása csatolt vagy egy subtemplate használatával néven második sablon a `Microsoft.Resources/deployments` erőforrástípus.

Ezután meg kell adnia a nevét, a meglévő tulajdonság módosítása vagy egy új nevet a beágyazott sablon hozzáadása tulajdonsághoz. Is meg kell adnia az eredeti tulajdonságok és az eredeti értékekre is. Ha nem adja meg az eredeti tulajdonságok és értékek, erőforrás-kezelő azt feltételezi, hogy hozzon létre egy új erőforrást szeretne, és törli az eredeti erőforrást.

## <a name="example-template"></a>Példa sablon

Azt mutatja be, hogy ez egy példa sablon vizsgáljuk meg. A sablon nevű virtuális hálózat telepíti `firstVNet` nevű több alhálózattal rendelkező `firstSubnet`. Ezután telepíti a virtuális hálózati adaptert (NIC) nevű `nic1` és társítja azt az alhálózatot. Ezt követően a központi telepítés erőforrást `updateVNet` tartalmaz egy beágyazott sablont, amely frissíti az `firstVNet` nevű második alhálózat hozzáadásával erőforrás `secondSubnet`. 

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
          "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
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

Vessen egy pillantást az erőforrás-objektum a `firstVNet` erőforrás első. Figyelje meg, hogy helyes-e beállításait a `firstVNet` beágyazott sablonban&mdash;ennek az az oka az erőforrás-kezelő nem engedélyezi a központi telepítés néven belül ugyanazt a sablont, és egy másik sablonba beágyazott sablonok lévőknek tekintendők. Az értékek respecifying által a `firstSubnet` erőforrás, hogy az erőforrás-kezelő törlésével és újratelepítésével helyett a meglévő erőforrás frissítése szólítja fel. Végül, az új beállítások a `secondSubnet` átveszik a frissítés során.

## <a name="try-the-template"></a>Próbálja meg a sablon

Ha azt szeretné, ez a sablon kísérletezhet, kövesse az alábbi lépéseket:

1.  Navigáljon az Azure portálon, válassza a  **+**  ikonra, és keresse meg a **sablon-üzembehelyezés** erőforrás írja be, és válassza ki azt.
2.  Keresse meg a **sablon-üzembehelyezés** lapon jelölje be a **létrehozása** gombra. Ezzel a gombbal megnyithatja a **egyéni telepítési** panelen.
3.  Válassza ki a **szerkesztése** ikonra.
4.  Az üres sablon törlésére.
5.  Másolja és illessze be a mintasablon a jobb oldali ablaktáblán.
6.  Válassza ki a **mentése** gombra.
7.  Vissza a **egyéni telepítési** panelen, de ezúttal nem néhány legördülő listák. Jelölje ki az előfizetését, vagy hozzon létre új vagy meglévő erőforráscsoportot használni, és jelöljön ki egy helyet. Tekintse meg a feltételeket és kikötéseket, és válassza ki a **elfogadom** gombra.
8.  Válassza ki a **beszerzési** gombra.

Miután a telepítés véget ért, nyissa meg a portálon megadott erőforráscsoport. Megjelenik a virtuális hálózati nevű `firstVNet` és a hálózati adapter nevű `nic1`. Kattintson a `firstVNet`, majd kattintson a `subnets`. Megjelenik a `firstSubnet` , amely eredetileg, és megjelenik a `secondSubnet` , amely hozzá lett adva a a `updateVNet` erőforrás. 

![Eredeti alhálózat és a frissített alhálózati](../_images/firstVNet-subnets.png)

Ezután térjen vissza az erőforráscsoportot, és kattintson a `nic1` kattintson `IP configurations`. Az a `IP configurations` szakaszban, a `subnet` értéke `firstSubnet (10.0.0.0/24)`. 

![nic1 IP-konfigurációk beállításait](../_images/nic1-ipconfigurations.png)

Az eredeti `firstVNet` ahelyett, hogy megújult ismételt létrehozása megtörtént. Ha `firstVNet` kellett újra létre lett hozva, `nic1` volna nem rendelhetők hozzá `firstVNet`.

## <a name="next-steps"></a>Következő lépések

* Ezzel a technikával meg van valósítva a [építőelemeket sablonprojekt](https://github.com/mspnp/template-building-blocks) és a [Azure hivatkozás architektúrák](/azure/architecture/reference-architectures/). Ezek segítségével hozzon létre egy saját architektúra, vagy telepítse a referencia-architektúrák egyikét használhatja.
