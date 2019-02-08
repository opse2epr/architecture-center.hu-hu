---
title: 'CAF: Házirend-kényszerítési döntési útmutató'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Ismerje meg a házirend-kényszerítési előfizetések Azure áttelepítések az alapvető tervezési prioritásként.
author: rotycenh
ms.openlocfilehash: 372926453ee4ae0502250e9b69fe8a0ea94f0ffe
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899043"
---
# <a name="policy-enforcement-decision-guide"></a>Házirend-kényszerítési döntési útmutató

Szervezeti házirend meghatározása nem hatékony, kivéve, ha egy lehet kényszeríteni a szervezetben. Bármilyen típusú felhőbeli migrálás tervezésének kulcsfontosságú annak meghatározása, hogyan lehet a legjobban úgy, hogy a meglévő informatikai folyamatokat, amelyek a teljes felhőalapú hagyatéki szabályzatoknak való megfelelés maximalizálása a cloud platform által biztosított eszközökkel.

![Küldik az ábrázolást legalább a házirend-kényszerítési beállításait a legösszetettebb, az alábbi hivatkozások a jump igazítva](../../_images/discovery-guides/discovery-guide-policy-enforcement.png)

Ugrás ide: [Ajánlott eljárások alapterv](#baseline-recommended-practices) | [szabályzatot a megfelelőség ellenőrzése](#policy-compliance-monitoring) | [házirend betartatása](#policy-enforcement)  |  [ Szervezetszintű házirend](#cross-organization-policy) | [automatikus végrehajtás](#automated-enforcement)

A felhő hagyatéki növekedésével megmérkőzött a megfelelő kell karbantartani, és érvényesíteni a házirendet egy nagyobb tömbje erőforrásokat, előfizetések és bérlők között. A nagyobb a hagyatéki, annál összetettebb a kényszerítő mechanizmust kell annak biztosítása érdekében a konzisztens megfelelést és gyors megsértésének észlelése. Az erőforrás vagy az előfizetés szintjén a platform által biztosított házirend kényszerítő mechanizmust elegendőek általában a kisebb magánfelhők számára, míg a nagyobb telepítések kihasználásához kifinomultabb mechanizmusok használata esetén a központi telepítési szabványok, előfordulhat, hogy kell erőforrás-csoportosítást és a szervezeten belül, és a naplózási házirend betartatása integrálása és jelentési rendszerrel.

A kulcs kihasználás kiválasztásakor a szabályzat végrehajtási stratégia összetettsége elsősorban összpontosít előfizetések vagy a bérlők által igényelt számát a [előfizetések kialakítása](../subscriptions/overview.md). Vezérlő belül a felhő hagyatéki különféle felhasználói szerepkörök számára biztosított mennyisége befolyásolhatja, valamint ezek a döntések.

## <a name="baseline-recommended-practices"></a>Ajánlott eljárások alapterv

Egyetlen előfizetés és a magánfelhők egyszerű sok vállalati házirendeket, amelyek a legtöbb felhőalapú platformon natív szolgáltatásaival kényszeríthető. Még ezen a szinten viszonylag kevés a központi telepítés bonyolultságát, a minták konzisztens használata során a CAF tárgyalt [útmutatók döntési](../overview.md) segíthet a szabályzatoknak való megfelelés alapkonfiguráció szintű létesíteni.

Példa:

- [A központi telepítési sablonok](../resource-consistency/overview.md) kioszthatja a szabványos szerkezetének és konfigurációjának erőforrásait.
- [Címkézési és szabványok elnevezési](../resource-tagging/overview.md) műveletek rendszerezése és a számlázási és üzleti igényeinek támogatásához.
- Forgalom felügyeleti és a hálózati korlátozások megvalósítható [szoftveralapú hálózatkezelés](../software-defined-network/overview.md).
- [Szerepköralapú hozzáférés-vezérlés](../identity/overview.md) biztonságossá teheti, illetve a felhőbeli erőforrások elkülönítése.

Indítsa el a felhő a házirend betartatása tervezési megvizsgálásával hogyan segíthet a tárgyalt teljes ezek az útmutatók a szabványos minták alkalmazásának a szervezeti követelményeknek.

## <a name="policy-compliance-monitoring"></a>A házirend-megfelelőség ellenőrzése

Egy másik kulcsfontosságú tényező, még viszonylag kis magánfelhők számára, a rendszer azon képessége, győződjön meg arról, hogy a felhőalapú alkalmazások és szolgáltatások megfelelnek a szervezeti házirend, azonnal a felelős felek értesítése, ha egy erőforrás nem megfelelővé válik. Hatékony [naplózás és jelentéskészítés](../log-and-report/overview.md) felhőbeli számítási feladatok megfelelőségi állapota egy vállalati szabályzat végrehajtási stratégia kulcsfontosságú része.

A felhők hagyatéki nő, a további eszközök például [az Azure Security Center](/azure/security-center/) is adja meg a beépített biztonsági és a fenyegetésészlelés, és alkalmazza a központosított szabályzatkezelés és riasztási az a helyszíni és felhőbeli számítógépadatok.

## <a name="policy-enforcement"></a>Szabályzat érvénybe léptetése

Konfigurációs beállítások és házirend igazításának biztosítása érdekében erőforrás létrehozási szabályok az előfizetés szintjén is alkalmazhat.

[Az Azure Policy](/azure/governance/policy/overview) létrehozásához, hozzárendeléséhez és házirendek kezelése az Azure-szolgáltatások. A szabályzatok különböző szabályokat és hatásokat kényszerítenek ki az erőforrásokon, hogy azok megfeleljenek a vállalati szabványoknak és szolgáltatói szerződéseknek. Az Azure Policy kiértékeli az erőforrások nem felel meg a hozzárendelt szabályzatok. Például érdemes korlátozni az SKU-méret a virtuális gépek a környezetben. Implementálva van a megfelelő házirendet, miután új és meglévő erőforrások megfelelőségének becsülhető meg. A megfelelő házirend, a meglévő erőforrások megfelelőségének tehető meg.

## <a name="cross-organization-policy"></a>Szervezetszintű házirend

A felhő hagyatéki során sok előfizetést igénylő kényszerítési növekedésével kell egy bérlői szintű végrehajtási stratégia házirend konzisztencia biztosításához összpontosíthat.

A [előfizetések kialakítása](../subscriptions/overview.md) a szervezeti felépítés vonatkozik szabályzat fiókra kell. Támogatja az összetett szervezeten belül az előfizetések kialakítása mellett [az Azure felügyeleti csoportok](../subscriptions/overview.md#management-groups) hozzárendelése az Azure Policy szabályok több előfizetéshez is használható.

## <a name="automated-enforcement"></a>Automatikus végrehajtás

Szabványos központi telepítési sablonok hatékony kisebb méretű, míg [Azure tervezetek](/azure/governance/blueprints/overview) lehetővé teszi a nagy méretű szabványos kiépítési és telepítési vezénylését az Azure-megoldások. Számítási feladatok több előfizetéshez is üzembe helyezhetők a létrehozott bármely erőforrás konzisztens házirend-beállításokkal.

A felhőalapú és helyszíni erőforrások integrálása informatikai környezetek esetében szükség lehet, használja a naplózás és jelentéskészítés a rendszerek, hibrid monitorozási képességeket biztosít. A külső vagy egyéni működési monitorozási rendszerek előfordulhat, hogy olyan további házirend-kényszerítési funkciókat biztosítanak. Összetett felhőalapú ingatlanadót, fontolja meg a legjobb módja, ezek a rendszerek integrálása a felhőbeli eszközöket.

## <a name="next-steps"></a>További lépések

Ismerje meg, rendszerezése és a magánfelhők támogatásához előfizetés tervezési és cégirányítási célok szabványosíthatja erőforrás konzisztencia használatáról.

> [!div class="nextstepaction"]
> [Erőforrás-konzisztencia](../resource-consistency/overview.md)
