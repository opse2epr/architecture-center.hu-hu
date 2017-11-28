---
title: "Az Azure Resource Manager-sablon funkciójának bővítése"
description: "Az Azure Resource Manager-sablon funkciójának bővítésére vonatkozó tippeket és trükköket ismertet"
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: d9cad6cfbfe8d42bcb21ef9e77504b80aef2d694
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="extend-azure-resource-manager-template-functionality"></a>Az Azure Resource Manager-sablon funkciójának bővítése

2016-ban az AzureCAT minták és gyakorlatok csapata létrehozott egy, az Azure Resource Manager-[sablonhoz való építőelemekből álló készletet](https://github.com/mspnp/template-building-blocks/wiki). A cél az erőforrások üzembe helyezésének egyszerűsítése volt. Mindegyik építőelem tartalmaz egy különböző paraméterfájlok által megadott erőforráskészleteket telepítő, előre létrehozott sablonokból álló készletet.

Az építőelem-sablonok kombinálhatók egymással a nagyobb és összetettebb üzembe helyezések létrehozásához. Egy virtuális gép Azure-ban való létrehozásához szükség van például egy virtuális hálózatra, tárfiókokra és más erőforrásokra. A [virtuális hálózat építőelem-sablon](https://github.com/mspnp/template-building-blocks/wiki/VNet-(v1)) üzembe helyezi a virtuális hálózatot és az alhálózatokat. A [virtuális gép építőelem-sablon](https://github.com/mspnp/template-building-blocks/wiki/Windows-and-Linux-VMs-(v1)) tárfiókokat, hálózati adaptereket és magukat a virtuális gépeket helyezi üzembe. Ezután létrehozhat egy szkriptet vagy sablont, amellyel meghívja mindkét építőelem-sablont a megfelelő paraméterfájlokkal együtt, hogy egy művelettel telepítsen egy teljes architektúrát.

Az építőelem-sablonok fejlesztése során a minták és gyakorlatok csapat számos koncepciót megalkotott az Azure Resource Manager-sablon funkciójának bővítésére. Ebben a sorozatban bemutatunk néhányat ezen koncepciók közül, hogy a saját sablonjaiban használhassa őket.

> [!NOTE]
> A cikkekben feltételezzük, hogy jól ismeri az Azure Resource Manager-sablonokat.