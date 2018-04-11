---
title: Explainer - Mi az az Azure-erőforráscsoport?
description: Egy erőforráscsoportot az Azure belső funkciói
author: petertay
ms.openlocfilehash: e7c7334bd88c28f57498486bd2bed3c349565222
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/09/2018
---
# <a name="what-is-an-azure-resource-group"></a>Mi az az Azure-erőforráscsoport?

Az a [Mi az Azure Resource Manager?](resource-manager-explainer.md) explainer a cikkben megtanulta, hogy az Azure Resource Manager megköveteli a **erőforráscsoport azonosítója** létrehozásához a rendszer telefonhívást indít, amikor olvasása, frissítése vagy erőforrás törlése. Az erőforráscsoport azonosítója hivatkozik egy **erőforráscsoport**. Erőforráscsoport egyszerűen Azure Resource Manager erőforrások érvényes azonosító, amely egy csoportba. Az erőforráscsoport azonosítója lehetővé teszi, hogy az Azure Resource Manager ez. rendelkező erőforrások csoportja a műveletek végrehajtásához

Például végezheti a felhasználó egy **törlése** hívása az Azure Resource Manager RESTful API-k az erőforráscsoport azonosítója megadása nélkül, beleértve a megadott erőforrás-azonosító. Az Azure Resource Manager le az összes erőforrás a megadott erőforráscsoport azonosítója egy belső Azure adatbázist, és a RESTful API-t mindegyik erőforrás törlése.

Erőforráscsoport nem tartalmazhat különböző előfizetéshez erőforrásokat. Ennek az az oka közötti Bérlőazonosító és előfizetés-azonosító egy-a-többhöz kapcsolat áll fenn &mdash; több előfizetés is megbízhat ugyanannak a bérlőnek a hitelesítést és engedélyezést biztosító, de minden előfizetés is megbízhat csak egy bérlő. Is egy-a-többhöz kapcsolat van előfizetés-azonosító és az erőforráscsoport azonosítója közötti &mdash; több erőforráscsoportok tartozhat ugyanahhoz az előfizetéshez, de minden erőforráscsoport csak egyetlen előfizetéssel is tartozik. Végül egy-a-többhöz kapcsolat van erőforráscsoport azonosítója és erőforrás-azonosító közötti &mdash; egyetlen erőforráscsoportként működnek rendelkezhet több erőforrást, de az egyes erőforrások csak egy egységes erőforrás-csoporthoz is tartozhatnak.

## <a name="next-steps"></a>További lépések

* Most, hogy az Azure erőforráscsoport-sablonok megismerte megpróbálhatnak eligazodást [erőforrásokhoz való hozzáférés korlátozása](/azure/active-directory/active-directory-understanding-resource-access?toc=/azure/architecture/cloud-adoption-guide/toc.json). Bár a legalapvetőbb Bevezetési fázis egy része nincs, fontos a köztes Bevezetési fázis során. Akkor is [az első erőforráscsoport létrehozása](/azure/azure-resource-manager/resource-group-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json) és tekintse át a [kialakítási útmutató a Azure erőforráscsoport-sablonok](resource-group.md).
