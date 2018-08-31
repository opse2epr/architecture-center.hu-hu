---
title: 'Enterprise Cloud Adoption: Mi a felhőbeli erőforrás-szabályozás?'
description: Az erőforrás-hozzáférés szabályozása az Azure-ban fogalmát ismertetése
author: petertaylor9999
ms.openlocfilehash: 17b95e56c033f6085285ef1b5cf69895ad3dcb51
ms.sourcegitcommit: ae8a1de6f4af7a89a66a8339879843d945201f85
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 08/31/2018
ms.locfileid: "43326531"
---
# <a name="enterprise-cloud-adoption-what-is-cloud-resource-governance"></a>Enterprise Cloud Adoption: Mi a felhőbeli erőforrás-szabályozás?

A [Azure működése?](what-is-azure.md), hogy megtanulta, hogy az Azure a kiszolgálókon és hálózati hardvereivel futtató virtualizált hardver- és a felhasználók nevében. Az Azure lehetővé teszi, hogy a szervezet fejlődés és IT-részlegek számára egyszerűvé létrehozása, olvasása, frissítése és törlése az erőforrásokat, igény szerint legyünk.

Azonban amíg így unrestricted erőforrás-hozzáférés a fejlesztők számára is, így nagyon hatékony, a költségek nem kívánt következményekkel is vezethet. Ha például egy fejlesztői csapat előfordulhat, hogy jóvá kell hagyni tesztelési erőforrások üzembe helyezésének, de felejtse el törölni őket, ha a tesztelés. Ezek az erőforrások továbbra is beleszámítanak a költségek, annak ellenére, hogy azok használata már nem jóváhagyott vagy szükséges. 

A megoldás a probléma az erőforrás-hozzáférés **cégirányítási**. Cégirányítási kezelését, megfigyelését és naplózás használata Azure-erőforrások megfelelnek a kitűzött célokat, és a szervezet követelményeinek folyamatban hivatkozik. 

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE2ii94] 

Következő célokat és követelményeket egyediek-e az egyes cégeknek, ezért nem lehetséges, hogy a cégirányítási időkorlátokat. Inkább az Azure csomagszűrő két elsődleges cégirányítási eszközök, **erőforráshoz szerepköralapú hozzáférés-vezérlés (RBAC)**, és **erőforrás-szabályzat**, és a szolgáltatás minden szervezet számára a használatuk cégirányítási modell tervezéséhez.

Az RBAC szerepkörök határozza meg, és a szerepkörök határozzák meg a képességeket a szerepkörhöz rendelt felhasználók. Például a **tulajdonosa** szerepkör lehetővé teszi, hogy az összes képességek (létrehozása, olvasása, frissítése és törlése) egy erőforráshoz, amíg a **olvasó** szerepkörök lehetővé teszi, hogy a csak olvasási képességet. Szerepkörök egy széles körét, amely különféle erőforrásokhoz vonatkozik, vagy vonatkozik néhány keskeny hatóköre lehet definiálni. 

Erőforrás-házirendek erőforrás-létrehozás vonatkozó szabályok meghatározásához. Például egy erőforrás-szabályzat korlátozhatja az egy adott előtti-appproved méretű virtuális gép Termékváltozata. Másik lehetőségként egy erőforrás-szabályzat kényszerítheti a költséghely a címke hozzáadása az erőforrás létrehozásához a kérelem elküldésekor. 

Ezek az eszközök konfigurálása, ha fontos szempont szervezeti rugalmasság és irányítás osztja el. Ez azt jelenti, hogy a szigorúbb cégirányítási házirendjében a kevésbé rugalmas válnak a fejlesztők és informatikai dolgozók. Ennek az oka egy korlátozó goverance házirendet szükség lehet további manuális lépések, például az eszközjelszó töltsön ki egy űrlapot, vagy manuálisan hozzon létre egy erőforrást a cégirányítási csapat egy személy e-mail küldése a fejlesztő. A goverance csapata véges képességekkel rendelkezik, és előfordulhat, hogy legyen várakozó fel, Várakozás a létrehozott és a felesleges erőforrások keletkezhetnek költségek addig azokat törölni kell az erőforrások rendszer fejlesztőcsapatok eredményez.

## <a name="next-steps"></a>További lépések

Most, hogy megismerkedett a felhőbeli erőforrások goverance fogalmát, helyezze át további részleteket ismerhet meg [erőforrásokhoz való hozzáférés kezelésének módját](azure-resource-access.md) előkészítésekor megtudhatja, hogyan tervezhet egy cégirányítási modellt használ az Azure-ban egy [egyetlen adaptercsoportba](../governance/governance-single-team.md) vagy [több csapat](../governance/governance-multiple-teams.md).

> [!div class="nextstepaction"]
> [További tudnivalók az Azure-beli erőforrás-hozzáférés](azure-resource-access.md)