---
title: Tervezzen a vállalkozás igényei szerint
description: Minden tervezési döntés legyen igazolható egy üzleti igénnyel
author: MikeWasson
ms.date: 08/30/2018
ms.openlocfilehash: f3086b36be0ead7466c33cd083f29f2c67bed440
ms.sourcegitcommit: ae8a1de6f4af7a89a66a8339879843d945201f85
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 08/31/2018
ms.locfileid: "43325900"
---
# <a name="build-for-the-needs-of-the-business"></a>Tervezzen a vállalkozás igényei szerint

## <a name="every-design-decision-must-be-justified-by-a-business-requirement"></a>Minden tervezési döntés legyen igazolható egy üzleti igénnyel

Ez a tervezési alapelv nyilvánvalónak tűnhet, de kulcsfontosságú szem előtt tartani a megoldások tervezése során. Több milliónyi, vagy néhány ezer felhasználóra számít? Elfogadható egy egyórás alkalmazáskimaradás? Kiszámítható számítási feladatokra vagy nagymértékben ingadozó forgalomra számít? Végső soron minden tervezési döntésnek igazolhatónak kell lennie egy üzleti igénnyel. 

## <a name="recommendations"></a>Javaslatok

**Határozza meg az üzleti célkitűzéseket**, beleértve a helyreállítási idő célkitűzését (RTO), a helyreállítási időkorlátot (RPO) és a maximális tolerálható kimaradást (MTO). Az architektúrával kapcsolatos döntéseket ezen információk alapján kell meghoznia. Ha például alacsony RTO-t szeretne elérni, érdemes lehet egy másodlagos régióba történő automatizált feladatátvételt megvalósítani. Ha azonban a megoldás magasabb RTO mellett is jól működik, akkor az ilyen fokú redundancia szükségtelen.

**Dokumentálja a szolgáltatói szerződéseket (SLA) és a szolgáltatási szint célkitűzéseit (SLO)**, beleértve a rendelkezésre állásra és a teljesítményre vonatkozó metrikákat. Felépíthet egy megoldást, amely 99,95%-os rendelkezésre állást biztosít. Elég lesz? A válasz egy üzleti döntés. 

**Modellezze az alkalmazást az üzleti tartomány köré**. Kezdje az üzleti követelmények elemzésével. Modellezze az alkalmazást ezen követelmények alapján. Fontolja meg egy tartományvezérelt tervezési (DDD) megközelítés használatát olyan [tartománymodellek][domain-model] létrehozásához, amelyek tükrözik az üzleti folyamatokat és használati eseteket. 

**Rögzítse a funkcionális és nem funkcionális követelményeket is**. A funkcionális követelmények alapján megítélheti, hogy az alkalmazás a megfelelő dolgokat teszi. A nem funkcionális követelmények alapján megállapíthatja, hogy egy alkalmazás *megfelelően* teszi-e ezeket a dolgokat. Fordítson kiemelt figyelmet a skálázhatóságra, a rendelkezésre állásra és a késére vonatkozó követelmények megértésére. Ezek a követelmények befolyásolják a tervezési döntéseket és a választott technológiákat.

**Bontsa fel a munkát számítási feladatok alapján**. Ebben a környezetben a „számítási feladat” kifejezés olyan különálló képességet vagy feladatot jelent, amely logikailag elválasztható más feladatoktól. A különböző számítási feladatok eltérő követelményekkel rendelkezhetnek a rendelkezésre állás, a skálázhatóság, az adatkonzisztencia és a vészhelyreállítás terén. 

**Tervezzen a növekedéssel**. Egy megoldás megfelelhet a jelenlegi igényeinek a felhasználók száma, a tranzakciók mennyisége, az adattárolás és más szempontok tekintetében. Egy robusztus alkalmazás azonban a növekedést is kezelni tudja az architektúra jelentős módosítása nélkül. Lásd: [Tervezzen horizontális felskálázásra](scale-out.md) és [Particionáljon a korlátok kiküszöböléséhez](partition.md). Azt is vegye figyelembe, hogy az üzleti modellje és üzleti követelményei idővel valószínűleg változni fognak. Ha egy alkalmazás szolgáltatásmodellje és adatmodelljei túl merevek, nehéz lesz új használati esetekre és forgatókönyvekre továbbfejleszteni. Lásd: [Tervezzen a fejlődést szem előtt tartva](design-for-evolution.md).

**Felügyelje a költségeket**. A hagyományos helyszíni alkalmazások esetében előre kell fizetnie a hardverekért (CAPEX). Egy felhőalkalmazás esetében a felhasznált erőforrásokért fizet. Mindenképpen legyen tisztában a felhasznált szolgáltatások díjszabásával. A teljes költség a hálózati sávszélesség felhasználását, a tárolást, az IP-címeket, a szolgáltatás használatát és egyéb tényezőket is tartalmaz. További információ: [Az Azure díjszabása][pricing]. Vegye figyelembe emellett az üzemeltetési költségeket is. A felhőben nem kell kezelnie a hardvereket és az egyéb infrastruktúrát, de továbbra is kezelnie kell az alkalmazásait, beleértve a DevOpsot, az incidenskezelést, a vészhelyreállítást stb. 

[domain-model]: https://martinfowler.com/eaaCatalog/domainModel.html
[pricing]: https://azure.microsoft.com/pricing/
