---
title: 'CAF: Házirend-kényszerítési döntési útmutató'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Ismerje meg a házirend-kényszerítési előfizetések Azure áttelepítések az alapvető tervezési prioritásként.
author: rotycenh
ms.openlocfilehash: 8e1b447c36051da14231c4f93da463e6f7d96e76
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298645"
---
# <a name="policy-enforcement-decision-guide"></a>Házirend-kényszerítési döntési útmutató

Szervezeti házirend meghatározása nem hatékony, kivéve, ha egy lehet kényszeríteni a szervezetben. Bármilyen típusú felhőbeli migrálás tervezésének kulcsfontosságú annak meghatározása, hogyan lehet a legjobban úgy, hogy a meglévő informatikai folyamatokat, amelyek a teljes felhőalapú hagyatéki szabályzatoknak való megfelelés maximalizálása a cloud platform által biztosított eszközökkel.

![Küldik az ábrázolást legalább a házirend-kényszerítési beállításait a legösszetettebb, az alábbi hivatkozások a jump igazítva](../../_images/discovery-guides/discovery-guide-policy-enforcement.png)

Ugrás ide: [Ajánlott eljárások alapterv](#baseline-recommended-practices) | [szabályzatot a megfelelőség ellenőrzése](#policy-compliance-monitoring) | [házirend betartatása](#policy-enforcement)  |  [ Szervezetszintű házirend](#cross-organization-policy) | [automatikus végrehajtás](#automated-enforcement)

A felhő hagyatéki növekedésével megmérkőzött a megfelelő kell karbantartani, és érvényesíteni a házirendet egy nagyobb tömb erőforrásokat, és az előfizetések között. A hagyatéki nagyobb lekérdezi, és növelje a szervezet a házirend követelményeinek, a házirend-kényszerítési folyamatok hatókörének kell bontsa ki a konzisztens házirend megfelelést, valamint gyors megsértésének észlelése.

Az erőforrás vagy az előfizetés szintjén a platform által biztosított házirend kényszerítő mechanizmust általában elegendőek a kisebb felhőalapú ingatlanadót. Nagyobb telepítések adja meg egy nagyobb kényszerítési hatókör, és előfordulhat, hogy kell kihasználásához kifinomultabb kényszerítési mechanizmus használata esetén a központi telepítési szabványok, az erőforrás-csoportosítást és a szervezet és a naplózási házirend betartatása integrálása és jelentési rendszerrel.

A házirend-kényszerítési folyamatok hatókörének meghatározásához legfontosabb tényezők a szervezet [cégirányítási követelmények a felhő](/azure/architecture/cloud-adoption/governance/overview), méretét és a felhő hagyatéki, és hogyan is megjelenik a szervezet a jellege[előfizetések kialakítása](../subscriptions/overview.md). A hagyatéki vagy nagyobb kell központilag kezelheti a házirend betartatása méretének növekedése is eloszthatók kényszerítési hatókör növekedését.

## <a name="baseline-recommended-practices"></a>Ajánlott eljárások alapterv

Egyetlen előfizetés és a magánfelhők egyszerű sok vállalati házirendeket, amelyek natív, erőforrások és-előfizetések az Azure platform szolgáltatásainak használata kényszeríthető. A konzisztens használata során a CAF tárgyalt mintáit [útmutatók döntési](../overview.md) segíthetnek megállapítani egy adott házirend betartatása nélkül megfelelőségi alapkonfiguráció szintjét.

Példa:

- [A központi telepítési sablonok](../resource-consistency/overview.md) kioszthatja a szabványos szerkezetének és konfigurációjának erőforrásait.
- [Címkézési és szabványok elnevezési](../resource-tagging/overview.md) műveletek rendszerezése és a számlázási és üzleti igényeinek támogatásához.
- Forgalom felügyeleti és a hálózati korlátozások megvalósítható [szoftveralapú hálózatkezelés](../software-defined-network/overview.md).
- [Szerepköralapú hozzáférés-vezérlés](../identity/overview.md) biztonságossá teheti, illetve a felhőbeli erőforrások elkülönítése.

Indítsa el a felhő a házirend betartatása tervezési megvizsgálásával hogyan segíthet a tárgyalt teljes ezek az útmutatók a szabványos minták alkalmazásának a szervezeti követelményeknek.

## <a name="policy-compliance-monitoring"></a>A házirend-megfelelőség ellenőrzése

Első lépés a túli egyszerűen az Azure platform által biztosított házirend kényszerítési mechanizmusokra hagyatkoznia lehetővé teszi a felhőalapú alkalmazások ellenőrzése annak ellenőrzése, és a szolgáltatások megfeleljenek a szervezeti házirend. Ide tartoznak a végrehajtási értesítési funkciókat felelős felek riasztás, ha egy erőforrás nem megfelelővé válik.  Hatékony [naplózás és jelentéskészítés](../log-and-report/overview.md) felhőbeli számítási feladatok megfelelőségi állapota egy vállalati szabályzat végrehajtási stratégia kulcsfontosságú része.

A felhők hagyatéki nő, a további eszközök például [az Azure Security Center](/azure/security-center/) is adja meg a beépített biztonsági és a fenyegetésészlelés, és alkalmazza a központosított szabályzatkezelés és riasztási az a helyszíni és felhőbeli számítógépadatok.

## <a name="policy-enforcement"></a>Szabályzatbetartatás

Az Azure-ban alkalmazhatja a konfigurációs beállítások és az erőforrás létrehozási szabályok a felügyeleti csoport, az előfizetés vagy az erőforrás csoport szintjén házirend igazításának biztosítása érdekében.

[Az Azure Policy](/azure/governance/policy/overview) létrehozásához, hozzárendeléséhez és házirendek kezelése az Azure-szolgáltatások. A szabályzatok különböző szabályokat és hatásokat kényszerítenek ki az erőforrásokon, hogy azok megfeleljenek a vállalati szabványoknak és szolgáltatói szerződéseknek. Az Azure Policy kiértékeli az erőforrások nem felel meg a hozzárendelt szabályzatok. Például érdemes korlátozni az SKU-méret a virtuális gépek a környezetben. Implementálva van a megfelelő házirendet, miután új és meglévő erőforrások megfelelőségének becsülhető meg. A megfelelő házirend, a meglévő erőforrások megfelelőségének tehető meg.

## <a name="cross-organization-policy"></a>Szervezetszintű házirend

A felhő hagyatéki során sok előfizetést igénylő kényszerítési növekedésével kell házirend konzisztencia biztosításához hagyatéki kiterjedő érvényesítési felhőstratégia összpontosíthat.

A [előfizetések kialakítása](../subscriptions/overview.md) a szervezeti felépítés vonatkozik szabályzat fiókra kell. Támogatja az összetett szervezeten belül az előfizetések kialakítása mellett [az Azure felügyeleti csoportok](../subscriptions/overview.md#management-groups) hozzárendelése az Azure Policy szabályok több előfizetéshez is használható.

## <a name="automated-enforcement"></a>Automatikus végrehajtás

Szabványos központi telepítési sablonok hatékony kisebb méretű, míg [Azure tervezetek](/azure/governance/blueprints/overview) lehetővé teszi a nagy méretű szabványos kiépítési és telepítési vezénylését az Azure-megoldások. Számítási feladatok több előfizetéshez is üzembe helyezhetők a létrehozott bármely erőforrás konzisztens házirend-beállításokkal.

A felhőalapú és helyszíni erőforrások integrálása informatikai környezetek esetében szükség lehet, használja a naplózás és jelentéskészítés a rendszerek, hibrid monitorozási képességeket biztosít. A külső vagy egyéni működési monitorozási rendszerek előfordulhat, hogy olyan további házirend-kényszerítési funkciókat biztosítanak. A nagyobb méretű vagy több érett felhőbeli ingatlanadót, fontolja meg ezek a rendszerek integrálását az felhőbeli eszköz a legjobb módja.

## <a name="next-steps"></a>További lépések

Ismerje meg, rendszerezése és a magánfelhők támogatásához előfizetés tervezési és cégirányítási célok szabványosíthatja erőforrás konzisztencia használatáról.

> [!div class="nextstepaction"]
> [Erőforrás-konzisztencia](../resource-consistency/overview.md)
