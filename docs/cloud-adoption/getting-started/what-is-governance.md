---
title: 'CAF: Mi az a felhőerőforrás-szabályozás?'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.date: 02/11/2019
description: MAGYARÁZAT felhőalapú erőforrás-szabályozása az Azure-ban
author: petertaylor9999
ms.openlocfilehash: 0989a5aad8a6290cce07fd71771c690bbd615e0d
ms.sourcegitcommit: 0a8a60d782facc294f7f78ec0e9033e3ee16bf4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/08/2019
ms.locfileid: "59068854"
---
<!-- markdownlint-disable MD026 -->

# <a name="what-is-cloud-resource-governance"></a>Mi az a felhőerőforrás-szabályozás?

A [Azure működése?](what-is-azure.md), hogy megtanulta, hogy az Azure a kiszolgálókon és hálózati hardvereivel futtató virtualizált hardver- és a felhasználók nevében. Az Azure lehetővé teszi, hogy a munkahelyi alkalmazások fejlesztését és IT-részlegek számára egyszerűvé létrehozása, olvasása, frissítése és törlése az erőforrásokat, igény szerint, agilisek lehetnek.

Azonban amíg erőforrásokhoz való korlátlan hozzáférés is, hogy a fejlesztők nagyon hatékony, is vezethet, váratlan költségek. Ha például egy fejlesztői csapat előfordulhat, hogy jóvá kell hagyni tesztelési erőforrások üzembe helyezésének, de felejtse el törölni őket, ha a tesztelés. Ezek az erőforrások továbbra is lépheti túl a költségeket, annak ellenére, hogy azok nem lesznek jóváhagyott vagy szükséges.

A megoldás az erőforrás-hozzáférés szabályozása. **Cégirányítási** kezelését, megfigyelését és az Azure-erőforrások megfelelnek a szervezet követelményeinek használatának naplózása folyamatban van.

<!-- markdownlint-disable MD034 -->

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE2ii94]

<!-- markdownlint-enable MD034 -->

Ezek a követelmények az egyes cégeknek, egyedi, így a cégirányítási időkorlátokat nem hasznos. Ehelyett szolgáltatás minden szervezet számára, azok az Azure két elsődleges cégirányítási eszközökkel cégirányítási modell tervezéséhez: **szerepköralapú hozzáférés-vezérlés (RBAC)** és **erőforrás-szabályzat**.

Az RBAC szerepkörök határozza meg, és a szerepkörök határozzák meg az egyes adott szerepkörhöz rendelt felhasználók képességeit. Például a **tulajdonosa** szerepkör lehetővé teszi, hogy az összes képességek (létrehozása, olvasása, frissítése és törlése) egy erőforráshoz, amíg a **olvasó** szerepkör lehetővé teszi, hogy a csak olvasási képességet. Szerepkörök egy széles körét, amely számos erőforrástípusok vonatkozik, vagy vonatkozik néhány keskeny hatóköre lehet definiálni.

Erőforrás-házirendek erőforrás-létrehozás vonatkozó szabályok meghatározásához. Például egy erőforrás-szabályzat korlátozhatja az egy adott előre jóváhagyott méretű virtuális gép Termékváltozata. Egy másik erőforrás-szabályzat egy hozzárendelt költséghely a címke alkalmazása sikerült kényszerítése, az erőforrás létrehozásához a kérelem elküldésekor.

Ezek az eszközök konfigurálásakor fontos irányítási és szervezeti rugalmasságot. A szigorúbb cégirányítási házirendjében a kevésbé rugalmas a fejlesztők és informatikai dolgozók lesz. A cégirányítási korlátozó házirend további manuális lépések, például a fejlesztő töltsön ki egy űrlapot vagy e-mail üzenetet küldhet a cégirányítási csapat tagjai manuálisan hozzon létre egy erőforrást igénylő lehet szükség. A cégirányítási csapata kapacitása véges, és előfordulhat, hogy megfelelően vannak megtervezve, eredményez a fejlesztői csapatok unproductively Várakozás keletkezhetnek költségek, törlés előtt létrehozott és a felesleges erőforrások az erőforrásaikat.

## <a name="next-steps"></a>További lépések

Most, hogy megismerkedett a felhőbeli erőforrás-szabályozás fogalmát, tudjon meg többet az erőforrásokhoz való hozzáférés kezelésének módját az Azure-ban.

> [!div class="nextstepaction"]
> [További tudnivalók az Azure-beli erőforrás-hozzáférés](azure-resource-access.md)
