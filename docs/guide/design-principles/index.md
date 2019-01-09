---
title: Tervezési alapelvek Azure-alkalmazásokhoz
titleSuffix: Azure Application Architecture Guide
description: Tervezési alapelvek Azure-alkalmazásokhoz.
author: MikeWasson
ms.date: 08/30/2018
ms.custom: seojan19
ms.openlocfilehash: 0ea6d6dd8a030591ce00a42aad5c693ea7809f6a
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/08/2019
ms.locfileid: "54110458"
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
