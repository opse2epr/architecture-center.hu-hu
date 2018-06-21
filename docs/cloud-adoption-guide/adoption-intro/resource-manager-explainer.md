---
title: 'Explainer: Mi az Azure Resource Manager?'
description: A belső működik az Azure Resource Manager ismerteti
author: petertay
ms.openlocfilehash: 60f09901bdc4b292abd73335b78c7d56a76f27a6
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/09/2018
ms.locfileid: "29060792"
---
# <a name="explainer-what-is-azure-resource-manager"></a>Explainer: Mi az Azure Resource Manager?

Az a [hogyan Azure működik?](azure-explainer.md) explainer, megismerte az Azure belső felépítésének. Ez az architektúra az elosztott alkalmazások által kezelt belső Azure-szolgáltatásokat futtató előtér tartalmazza.

Az Azure előtér tartalmaz egy Azure Resource Manager nevű szolgáltatás. Az Azure Resource Manager felelős az életciklus-erőforrások Azure-ban üzemeltetett való létrehozása a törlés. Számos módon az Azure Resource Manager interaktív &mdash; Powershell, az Azure parancssori felület, az SDK-k használatával &mdash; egyes eszközök, de csak egy burkoló Azure Resource Manager által üzemeltetett RESTful API keresztül.

Az Azure Resource Manager által biztosított RESTful API egységes illesztőfelületet olyan keresztül **erőforrás-szolgáltató**. Erőforrás-szolgáltatók egyszerűen csak Azure-szolgáltatásokat, amelyek létrehozása, olvasása, frissítése és törlése az Azure-erőforrások. Valójában a RESTful API ezek a módszerek magában foglalja. 

A RESTful API szükséges egy jogkivonatot a felhasználóhoz, egy **előfizetés-azonosító**, valami új - és egy **erőforráscsoport azonosítója**. Oktatóanyag a következőket ismerteti az erőforráscsoportokat a [erőforrás csoport explainer](resource-group-explainer.md) cikk. Az Azure Resource Manager is szükséges a **bérlői azonosító**, amely a hozzáférési jogkivonat részeként van kódolva. 

Amikor egy erőforrás létrehozásához érvényes API-hívás érkezik, az Azure Resource Manager kapacitás megkeresi a megadott régión belül, és szükséges fájlok egy átmeneti helyre másolja. A háló tartományvezérlőre, az állvány elküldi a kérelmet, és a háló vezérlő az erőforrásokat foglal le. A háló tartományvezérlő válaszol a kérésre sikerességét vagy sikertelenségét értesítéssel, valamint egy **erőforrás-azonosító** az újonnan létrehozott erőforrás. Ezek négy azonosítók belső az Azure-ban tároljuk, és együttesen üzembe helyezett erőforrás egyedi azonosítóként szolgál.

## <a name="next-steps"></a>További lépések

* Most, hogy a belső működésének Azure Resource Manager megértéséhez további [tudnivalók az erőforráscsoportokról](resource-group-explainer.md) segítséget az első erőforráscsoport létrehozása.
