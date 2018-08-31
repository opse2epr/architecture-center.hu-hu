---
title: Tervezési alapelvek Azure-alkalmazásokhoz
description: Tervezési alapelvek Azure-alkalmazásokhoz
author: MikeWasson
ms.date: 08/30/2018
ms.openlocfilehash: 5dd5d02019723ce57ba377d99b3965d0d7ed4079
ms.sourcegitcommit: ae8a1de6f4af7a89a66a8339879843d945201f85
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 08/31/2018
ms.locfileid: "43326074"
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

