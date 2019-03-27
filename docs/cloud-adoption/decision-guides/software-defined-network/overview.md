---
title: 'CAF: Szoftveralapú hálózatok'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Core szolgáltatásként az Azure áttelepítések a szoftver meghatározott hálózatok vitafórum
author: rotycenh
ms.openlocfilehash: d164ba488552715dc97719329ae9de3fcf5d83ed
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298789"
---
# <a name="caf-software-defined-network-decision-guide"></a>CAF: Szoftver szoftveralapú hálózati döntési útmutató

Szoftveralapú hálózatkezelés (SDN) egy hálózati architektúrát, célja, hogy a virtualizált hálózati funkcióit központilag felügyelt, konfigurálhatók, és szoftver módosított. SDN-alapú absztrakciós réteget biztosít a fizikai hálózati infrastruktúrán, és lehetővé teszi, hogy a virtualizált fizikai egyenértékű útválasztók, tűzfalak és más hálózatkezelési hardverek lenne egy a helyszíni hálózathoz is megtalálhatók.

SDN-alapú lehetővé teszi, hogy az informatikai munkatársak konfigurálhatja és telepítheti a hálózati struktúrák és képességeket, amelyek támogatják a virtualizált erőforrásokat használó számítási feladat igényeinek megfelelően. Szoftveralapú KözpontiTelepítés-felügyelet a rugalmas lehetővé teszi, hogy a hálózati erőforrások gyors módosítást, és lehetővé teszi mindkét Agilis és a hagyományos üzembe helyezési modellt támogatják. Az SDN-alapú technológiával létrehozott virtualizált hálózatok olyan kritikus fontosságú, biztonságos hálózatok létrehozása a nyilvános felhőalapú platformon.

## <a name="networking-decision-guide"></a>Hálózati döntési útmutató

![Hálózati beállítások és a legkevésbé küldik az ábrázolást a legösszetettebb, az alábbi hivatkozások a jump igazítva](../../_images/discovery-guides/discovery-guide-sdn.png)

Ugrás ide: [Csak a PaaS](paas-only.md) | [Felhőbeli natív](cloud-native.md) || [Felhőalapú DMZ](cloud-dmz.md) [hibrid](hybrid.md) | [Hub/küllő modell](hub-spoke.md) | [további](#learn-more)

SDN-alapú díjszabás és a feladatok összetettsége különböző mértékben számos lehetőséget kínál. A fenti felderítési útmutató gyors személyre ezek a beállítások a legjobban megfelelnek az adott üzleti és technológiai stratégiák hivatkozást biztosít.

A jelen útmutatóban kihasználás, amely előtt a hálózati architektúra kapcsolatos döntéseket Felhőstratégia csapata készített számos fontos döntések függ. Legfontosabb a tartalmak között vannak érintő döntések a [digitális hagyatéki definíció](../../digital-estate/overview.md) és [előfizetések kialakítása](../subscriptions/overview.md) (amelynek is szükség lehet a felhő számlázási kapcsolatos döntések a bemenetek és globális piacot stratégiák).

Kisebb, egyetlen régióban üzemelő példányok legfeljebb 1000 virtuális gépek kevésbé valószínű, hogy ez kihasználás is jelentősen befolyásolja. Ezzel szemben nagy bevezetési erőfeszítések több mint 1000 virtuális gépek, több üzleti egységek vagy több földrajzi politic piacok meghódításáról, jelentősen befolyásolhatja, az SDN-alapú döntést és a kulcs kihasználás is.

## <a name="choosing-the-right-virtual-networking-architectures"></a>A megfelelő virtuális hálózati architektúrák kiválasztása

Ez a szakasz a döntést az útmutató segítségével válassza ki a megfelelő virtuális hálózati architektúrák tartalmazó gyűjteménnyel bővítjük.

Számos módon végrehajtja a szoftveralapú Hálózatkezelési technológiákat a felhő alapú virtuális hálózatot hozhat létre. Hogyan használja az áttelepítéshez a virtuális hálózatok felépítésének, és hogyan kommunikáljanak az ezekhez a hálózatokhoz meglévő IT-infrastruktúrája a terhelési követelményeknek és a cégirányítási követelmények függ.

Melyik virtuális hálózati architektúrát vagy architektúrák tervezésekor a felhőbe való migrálást kombinációját megtervezésekor vegye figyelembe a következő kérdéseket, annak meghatározásához, hogy a szervezete szükségleteinek:

| Kérdés | Csak PaaS esetén | Natív felhőalapú | Felhőalapú DMZ | Hibrid | Küllős topológiájú |
|-----|-----|-----|-----|-----|-----|
| Lesz a számítási feladat csak a PaaS-szolgáltatások használata és hálózati képességek a szolgáltatások által biztosított ismertetettek igényel? | Igen | Nem | Nem | Nem | Nem |
| Nem a számítási feladatok helyszíni alkalmazásokkal való integrációra van szükség? | Nem | Nem | Igen | Igen | Igen |
| Ön létrehozott fejlett biztonsági szabályzatok és a helyszíni és a felhő közötti biztonságos kapcsolatok hálózatok? | Nem | Nem | Nem | Igen | Igen |
| Igényel a számítási feladatok felhőbeli identitás szolgáltatásokon keresztül nem támogatott hitelesítési szolgáltatásokat, vagy tegye meg kell közvetlen hozzáférés a helyszíni tartományvezérlőket? | Nem | Nem | Nem | Igen | Igen |
| Szükség lesz üzembe helyezheti és kezelheti a virtuális gépek és a számítási feladatok nagy számú? | Nem | Nem | Nem | Nem | Igen |
| Meg kell adnia a központosított kezelés és a helyszíni kapcsolat közben delegálása egyes számítási feladatok Teams erőforrásai feletti felügyeletet? | Nem | Nem | Nem | Nem | Igen |

## <a name="virtual-networking-architectures"></a>Virtuális hálózati architektúrák

Ismerje meg, további információért a elsődleges definiált hálózati architektúrák:

- [**Csak a PaaS**](paas-only.md): Platformszolgáltatás (PaaS) termékek támogató platform korlátozott beépített hálózatkezelési szolgáltatások készlete, és nem igényel egy meghatározott szoftveralapú hálózati, számítási követelmények támogatására.
- [**Natív felhőalapú**](cloud-native.md): Egy felhőbeli natív virtuális hálózaton, a alapértelmezett szoftveresen definiált hálózati architektúrát, amikor üzembe erőforrásokat az olyan felhőalapú platform.
- [**Szegélyhálózat (DMZ) a felhő**](cloud-dmz.md): A helyszíni és a felhő között korlátozott kapcsolatot biztosít a hálózaton keresztül egy demilitarizált zóna, a felhőalapú környezetben a végrehajtása védett.
- [**Hibrid**](hybrid.md): A hibrid hálózati architektúra lehetővé teszi, hogy a virtuális hálózatot a helyszíni erőforrások eléréséhez, és ez fordítva is igaz.
- [**Küllős topológiájú**](hub-spoke.md): A küllős topológiájú architektúránk segítségével központilag külső kapcsolatok, valamint a megosztott szolgáltatások kezelésére, az egyes számítási feladatok elkülönítésére és potenciális előfizetési korlátok leküzdése.

## <a name="learn-more"></a>Részletek

A szoftveralapú hálózatkezelés az Azure platform a további információt a következő témakörben találhat.

- [Az Azure Virtual Network](/azure/virtual-network/virtual-networks-overview). Az Azure-ban az alapvető SDN-funkció az Azure virtuális hálózat, amely közvetítőként szolgál egy felhőalapú analóg fizikai, helyszíni hálózatok által biztosított. Virtuális hálózatok szerepét egy alapértelmezett elkülönítési határt, a platform-erőforrások közötti is.
- [Azure-beli hálózati biztonsági eljárások](/azure/security/azure-security-network-security-best-practices). Javaslatok az Azure biztonsági csapat, a virtuális hálózatok konfigurálása a biztonsági rések minimalizálása érdekében.

## <a name="next-steps"></a>További lépések

Ismerje meg, hogyan naplók, figyelési és jelentési vannak felhőbeli számítási feladatok állapotát és a szabályzat megfelelőségének kezelése az üzemeltetési csapatok által használt.

> [!div class="nextstepaction"]
> [Naplók és jelentéskészítés](../log-and-report/overview.md)
