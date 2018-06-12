---
title: 'Explainer: Mi az a felhő irányítás?'
description: Az erőforrás-irányítás fogalmát ismerteti az Azure és a felhő
author: petertay
ms.openlocfilehash: 63b04089aad5fc736641f8aaa6ff5247ea8ba13e
ms.sourcegitcommit: b3d74d8a89b2224fc796ce0e89cea447af43a0d4
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 06/11/2018
ms.locfileid: "35290697"
---
# <a name="explainer-what-is-cloud-resource-governance"></a>Explainer: Mi az a felhő erőforrás irányítás?

Az a [hogyan Azure működik](azure-explainer.md) explainer, megtudta, hogy Azure gyűjteményei, kiszolgálók és hálózati eszközt futtató virtualizált hardver- és a felhasználók nevében. Azure lehetővé teszi, hogy a szervezet fejlesztői és az informatikai részlegeknek képesnek kell lennie a gyors egyszerűvé létrehozása, olvasása, frissítése és törli az erőforrást, igény szerint.

Azonban amíg korlátozás jogosultságot ad erőforrások elérése a fejlesztők számára biztosíthat nagyon nagy, nem kívánt költség következményekre is vezethet. Például a fejlesztői csapat előfordulhat, hogy jóvá kell hagyni tesztelési erőforráscsoport telepíteni, de elfelejti törli őket, ha a tesztelés. Ezek az erőforrások továbbra is keletkeznek költségek, annak ellenére, hogy azok használata már nem jóváhagyott vagy szükséges. 

Ez a probléma megoldása az erőforrás-hozzáférés **cégirányítási**. Cégirányítási kezelését, megfigyelését és naplózás céljai az Azure-erőforrások használatát és a szervezet folyamatban hivatkozik. 

Következő célokat és követelményeket egyediek minden szervezet számára, hogy nem létezik univerzális megközelítése irányítás szerepelhet. Ehelyett Azure megvalósítja a két elsődleges irányítás eszközök **erőforrás-alapú hozzáférés-vezérlést (RBAC)**, és **erőforrás-házirend**, és a használatuk irányítás modell tervezéséhez minden szervezet számára.

Szerepalapú határozza meg a szerepköröket, és szerepkörök definiálása a képességeket a szerepkörhöz hozzárendelt felhasználó számára. Például a **tulajdonos** szerepkör lehetővé teszi, hogy az összes képességei (létrehozása, olvasása, frissítése és törlése)-erőforrásokhoz, amíg a **olvasó** szerepkörök lehetővé teszi, hogy a csak olvasási képességet. Szerepkörök egy széles körét, amely különféle erőforrásokhoz vonatkozik, vagy néhány alkalmazó keskeny hatókör is definiálhatók. 

Erőforrás-házirendek erőforrás létrehozása vonatkozó szabályok meghatározásához. Például egy erőforrás-házirend korlátozhatja a virtuális gépek egy adott előtti-appproved értékre a Termékváltozat. Vagy egy erőforrás-házirend alkalmazását kényszeríthetik költséghely rendelkező címke hozzáadása a kérelem az erőforrás létrehozásához. 

Ezek az eszközök konfigurálásakor fontos szempont szervezeti agilitást és irányítási van terheléselosztás. Ez azt jelenti, hogy a szigorúbb a cégirányítási házirend, a kisebb gyors a fejlesztők és informatikai dolgozók válik. Ennek az az oka egy korlátozó goverance házirend szükség lehet további manuális lépések, például az eszközjelszó űrlap kitöltése vagy e-mailt küldeni egy személy a cégirányítási csapat manuálisan az erőforrás létrehozása a fejlesztő. A goverance team véges képességekkel rendelkezik, és előfordulhat, hogy válnak a várakozó, Várakozás a információforrásait, hogy létezik és nem szükséges erőforrások keletkezhetnek költségek, amíg azok várakozási idő után lehet törölni, a rendszer fejlesztési csapat eredményez.

## <a name="next-steps"></a>További lépések

A következő lépés az Azure elfogadásakor [megismerése az Azure-ban digitális identitásokat](tenant-explainer.md) és [az első felhasználó létrehozása az Azure AD][docs-add-users-to-aad].

<!-- Links -->

[docs-add-users-to-aad]: /azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json