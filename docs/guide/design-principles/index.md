---
title: Tervezési alapelvek Azure-alkalmazásokhoz
titleSuffix: Azure Application Architecture Guide
description: Tervezési alapelvek Azure-alkalmazásokhoz.
author: MikeWasson
ms.date: 08/30/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19
ms.openlocfilehash: 8aab710ef6ffde493b80810750d2c0bc299ffaa6
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54485553"
---
# <a name="ten-design-principles-for-azure-applications"></a>Tíz tervezési alapelv Azure-alkalmazásokhoz

Ezeket a tervezési alapelveket követve skálázhatóbbá, rugalmasabbá és felügyelhetőbbé teheti alkalmazását.

**[Tervezzen az önjavítást szem előtt tartva](self-healing.md)**. Az elosztott rendszerekben időnként fellépnek hibák. Tervezheti úgy az alkalmazását, hogy kijavítsa önmagát, ha hiba történik.

**[Tervezzen mindent redundánsra](redundancy.md)**. Redundanciát építve az alkalmazásba elkerülheti a kritikus hibapontokat.

**[Minimalizálja a koordinációt](minimize-coordination.md)**. A skálázhatóság érdekében minimalizálja a koordinációt az alkalmazásszolgáltatások között.

**[Tervezzen horizontális felskálázásra](scale-out.md)**. Tervezze úgy az alkalmazását, hogy az skálázható legyen horizontálisan, igény szerint hozzá lehessen adni vagy el lehessen távolítani új példányokat.

**[Particionáljon a korlátok megkerüléséhez](partition.md)**. Használjon particionálást az adatbázis-, hálózati és számítási korlátok megkerüléséhez.

**[Tervezzen műveletekhez](design-for-operations.md)**. Tervezze úgy az alkalmazását, hogy a műveleti csapatnak kéznél legyenek a szükséges eszközök.

**[Használjon felügyelt szolgáltatásokat](managed-services.md)**. Amikor lehetséges, a szolgáltatásként nyújtott infrastruktúra (IaaS) helyett használjon szolgáltatásként nyújtott platformot (PaaS).

**[Használja a feladathoz legmegfelelőbb adattárat](use-the-best-data-store.md)**. Válassza az adataihoz és azok felhasználásához leginkább megfelelő tárolótechnológiát.

**[Tervezzen a fejlődést szem előtt tartva](design-for-evolution.md)**. Idővel minden sikeres alkalmazás változik. A fejlődést szem előtt tartó tervezés kulcsfontosságú a folyamatos innováció szempontjából.

**[Tervezzen a vállalkozás igényei szerint](build-for-business.md)**. Minden tervezési döntés legyen igazolható egy üzleti igénnyel.
