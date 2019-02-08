---
title: 'CAF: Nagyvállalati cégirányítási utazás'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Nagyvállalati cégirányítási utazás
author: BrianBlanchard
ms.openlocfilehash: 689b600524c3be6c505ade8c5aaa447d772c6e35
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899061"
---
# <a name="caf-large-enterprise-governance-journey"></a>CAF: Nagyvállalati cégirányítási utazás

## <a name="best-practice-overview"></a>Ajánlott eljárások áttekintése

Cégirányítási folyamaton keresztül a cégirányítási lejárat különböző szakaszaiban kitalált vállalat tapasztalatainak követi. Valós felhasználói utak alapján. Az ajánlott gyakorlati tanácsok a korlátozásokat, és kitalált vállalat alapulnak.

Kiindulópontként gyors Ez az Áttekintés egy minimális működőképes termékek (MVP) esetében az ajánlott eljárások alapján cégirányítási határozza meg. Egyes cégirányítási fejlesztések, további ajánlott eljárások hozzáadása új üzleti hivatkozásokat is tartalmaz, vagy műszaki kockázatok bontakozik ki.

> [!WARNING]
> Az MVP egy alapkonfiguráció kiindulási pontjaként feltételezések alapján. Még a minimálisan szükséges ajánlott eljárásokat egyedi üzleti kockázatot és kockázati tolerancia által üzemeltetett vállalati szabályzatok alapján. Ha szeretné látni, ha ilyen Előfeltevések vonatkozik Önre, olvassa el a [hosszabb narratíva](./narrative.md) Ez a cikk következő.

### <a name="governance-best-practice"></a>Cégirányítási ajánlott eljárás

Ez az ajánlott eljárás, amely egy szervezet segítségével gyorsan és következetesen több Azure-előfizetések hozzáadása cégirányítási guardrails alaprendszert funkcionál.

### <a name="resource-organization"></a>Erőforrás-szervezet

Az alábbi ábrán látható a cégirányítási MVP-hierarchia a erőforrások rendszerezéséhez.

![Erőforrás-szervezet diagramja](../../../_images/governance/resource-organization.png)

Minden alkalmazást kell telepíteni a felügyeleti csoport, az előfizetés és az erőforrás csoporthierarchia megfelelő területén. Tervezési központi telepítése során a felhő Cégirányítási csapat hoz létre a szükséges csomópontok biztosítson hatékony eszközöket a felhő bevezetésének csapatok a hierarchiában.

1. Felügyeleti csoport részletes hierarchia, amely tükrözi a geography környezet (éles környezet, nem éles üzemi) írja be mindegyik üzleti egység számára.
2. Egy előfizetés az egyes egyedi kombinációja üzleti egység, a földrajzi hely, a környezet és a "Kategorizálási Application."
3. Minden alkalmazáshoz külön erőforrást csoport.
4. Ez a csoportosítás a hierarchia minden szintjén egységes elnevezési rendszeréhez kell alkalmazni.

![Nagyvállalati erőforrás szervezeti diagram](../../../_images/governance/large-enterprise-resource-organization.png)

Ezek a minták a növekedésre feleslegesen megfelel a hierarchia nélkül adja meg.

[!INCLUDE [governance-of-resources](../../../../../includes/cloud-adoption/governance/governance-of-resources.md)]

## <a name="governance-evolutions"></a>Cégirányítási fejlesztések

Az MVP üzembe helyezését követően további rétegeit cégirányítási gyorsan beépíthető a környezetben. Íme néhány módszer az MVP speciális üzleti igényeinek változását követve:

- [A védett adatok alapvető biztonsági](./security-baseline-evolution.md)
- [Az üzletmenet szempontjából kritikus alkalmazások erőforrás-konfigurációk](./resource-consistency-evolution.md)
- [A Cost Management vezérlők](./cost-management-evolution.md)
- [Vezérlők többfelhős fejlődést szem előtt tartva](./multi-cloud-evolution.md)

<!-- markdownlint-disable MD026 -->

## <a name="what-does-this-best-practice-do"></a>Mit jelent ez az ajánlott eljárás?

Az MVP a módszereket, és az eszközök a [üzembe helyezési gyorsítás](../../deployment-acceleration/overview.md) fegyelem számára a vállalati szabályzat alkalmazását létrehozása történik. Különösen az MVP Azure tervezetek, Azure Policy és használ az Azure felügyeleti csoportok néhány alapvető vállalati házirendek alkalmazása a fiktív cég a narratíva meghatározottak szerint. Ezek a vállalati házirendek érvénybe lépnek az Azure Resource Manager-sablonokkal és az Azure házirendek használatával az identitások és a biztonság nagyon kis méretű alap.

![Egy növekményes cégirányítási MVP – példa](../../../_images/governance/governance-mvp.png)

## <a name="evolving-the-best-practice"></a>Az ajánlott eljárás szerint növekvő

Az idő múlásával a cégirányítási MVP való összpontosításnak köszönhetően a felügyeleti eljárásaink használható. Bevezetési aknázhatja nyújtotta fejlett lehetőségeket, mint az üzleti kockázat nő. Ezek a kockázatok csökkentése érdekében a CAF cégirányítási modellen belül különböző oktatnak fejlődik. Az oktatóanyag-sorozatban a további cikkek a vállalati házirend, kitalált vállalat érintő alakulása tárgyalják. Ezek a fejlesztések között három oktatnak fordulhat elő:

- Identitásra építve, áttelepítési függőségek, a a narratíva változása
- A Cost Management bevezetési skálázását követve rugalmasan méretezhető.
- Biztonsági alapterv, mint a védett adatok van üzembe helyezve.
- Erőforrás konzisztencia, mint az IT-üzemeltetési kezdődik, üzleti szempontból alapvető fontosságú számítási feladatok támogatása.

![Egy növekményes cégirányítási MVP – példa](../../../_images/governance/governance-evolution-large.png)

## <a name="next-steps"></a>További lépések

Most, hogy ismeri a cégirányítási MVP, és kövesse a cégirányítási fejlesztések, ötlete van, olvassa el a támogató narratíva további környezet.

> [!div class="nextstepaction"]
> [Olvassa el a támogató narratíva](./narrative.md)
