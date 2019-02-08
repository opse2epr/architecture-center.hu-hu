---
title: 'CAF: Erőforrás konzisztencia mintautasítások házirend'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Erőforrás konzisztencia mintautasítások házirend
author: BrianBlanchard
ms.openlocfilehash: 9217ae73b0edf5b40bedac1cdc961b7a34da87d1
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899135"
---
# <a name="resource-consistency-sample-policy-statements"></a>Erőforrás konzisztencia mintautasítások házirend

Egyes felhőszolgáltatási házirend-utasítások szolgálnak, amelyeket a kockázati értékelés során azonosított adott kockázatkezelés. Ezek az utasítások kell biztosítania a kockázatok tömör összegzését, és azokkal a tervek. Minden egyes utasítás definíciójának tartalmaznia kell adatot megtalálhatja:

- Technikai kockázat – a kockázatokat, ez a szabályzat foglalkozni fog összegzését.
- Házirendutasítás – a házirend követelményeinek törölje összefoglaló leírását.
- Kialakítási lehetőségeket – gyakorlati ajánlásokat készítünk, előírásoknak vagy egyéb útmutatást, amely az IT-csoportoknak és fejlesztők is használhatják, ha a házirend megvalósítása.

A következő házirend mintautasítások cím számos gyakori üzleti erőforrás konzisztencia kapcsolatos kockázatokat, és a példaként szolgálnak, hogy a házirend-utasítások használatával saját szervezete igények kidolgozásakor hivatkozhat. Vegye figyelembe, hogy ezek a példák nem jelentenek a proscriptive kell, és a vélhetően több szabályzat lehetőség kezelésekor bármely adott kockázattal rendelkező. Szorosan együttműködnek az üzleti és informatikai csapata hozza létre, azonosíthatja a házirend a kockázatok egyedi készletét leginkább megfelelő megoldásokra.

## <a name="tagging"></a>Címkézés

**Technikai kockázati**: Nem megfelelő metaadatok címkézés, az üzembe helyezett erőforrásokhoz kapcsolódó, IT-üzemeltetési támogatás vagy az erőforrások alapján szükséges SLA, fontos üzleti tevékenységet, vagy az üzemeltetési költségek optimalizálása nem rangsorolhatók. Ez az informatikai erőforrásokat és a megoldási lehetséges késedelmeket hibás foglalási eredményezheti.

**Házirendutasítás**: A következő házirendek hajtják végre:

- Üzembe helyezett eszközök kell megcímkézni. a következő értékeket: költségét, kritikusság, SLA-t és a környezetben.
- Irányítás azokat az eszközöket ellenőrizni kell a kapcsolódó költségek, kritikusság, SLA-t, alkalmazás és környezet címkézése. Előre meghatározott értékeket a cégirányítási csapata által kezelt összes értéket kell igazítás.

**A lehetséges tervezési lehetőségek**: Az Azure-ban [standard név-érték metaadat-címkéket](/azure/azure-resource-manager/resource-group-using-tags) a legtöbb erőforrást típusok támogatottak. [Az Azure Policy](/azure/governance/policy/overview) adott címkék kényszerítése az erőforrás-létrehozás részeként használatos.

## <a name="ungoverned-subscriptions"></a>Ungoverned előfizetések

**Technikai kockázati**: Az előfizetések és a felügyeleti csoportok tetszőleges létrehozását a felhő hagyatéki elkülönített szakasza nem vonatkozik megfelelően a cégirányítási szabályzatokat vezethet.

**Házirendutasítás**: Az új előfizetési lehetőségekről, vagy bármilyen üzleti szempontból alapvető fontosságú alkalmazások és a védett adatok felügyeleti csoportok létrehozása a felhő Cégirányítási csapat felülvizsgálat lesz szükség. Jóváhagyott változtatások integráljuk egy megfelelő tervezet-hozzárendelést.

**A lehetséges tervezési lehetőségek**: Rendszergazdai hozzáférés a szervezet zároljuk [az Azure felügyeleti csoportok](/azure/governance/management-groups/) , csak a jóváhagyott szabályozhatja az előfizetés létrehozása és hozzáférés-ellenőrzési folyamatot végző cégirányítási csapat tagjai.

## <a name="manage-updates-to-virtual-machines"></a>A virtuális gépek frissítéseinek kezelése

**Technikai kockázati**: Virtuális gépek (VM), amelyek nem naprakész a legújabb frissítésekkel és a szoftverjavítások ki téve a biztonság és teljesítmény problémákat, amelyek eredményezhetnek szolgáltatáskimaradásokat.

**Házirendutasítás**: Irányítás azokat az eszközöket ki kell kényszerítenie, hogy az automatikus frissítések engedélyezve vannak-e az összes üzembe helyezett virtuális gépeken. Szabálysértések lehet tekintse át a felügyeleti csoportokhoz, és operations házirendek megfelelően javítja. Eszközök, amelyek nem frissülnek automatikusan a tulajdonosa az IT-üzemeltetési folyamatok szerepelnie kell.

**A lehetséges tervezési lehetőségek**: Az Azure-ban üzemeltetett virtuális gépek, konzisztens update management segítségével megadhatja a [frissítéskezelési megoldás az Azure Automation](/azure/automation/automation-update-management).

## <a name="deployment-compliance"></a>Központi telepítés megfelelőségi

**Technikai kockázati**: Üzembehelyezési szkriptek és automatizálási eszközök, amelyek a nem teljes mértékben megvizsgált a felhő Cégirányítási csapat szabályzatot megsértő erőforrások üzemelő példányok eredményezhet.

**Házirendutasítás**: A következő házirendek hajtják végre:

- Központi telepítési eszközkészlet jóvá kell hagynia a felhő Cégirányítási csapatától folyamatos irányítás az üzembe helyezett eszközök.
- Üzembehelyezési szkriptek fenn kell tartani a központi tárházban elérhető-e a felhő Cégirányítási csapat rendszeres áttekintése és a naplózási.

**A lehetséges tervezési lehetőségek**: Konzisztens használata az [Azure tervezetek](/azure/governance/blueprints/) automatikus központi telepítések felügyeletéhez szükséges lehetővé teszi, hogy egységes Azure-erőforrások, amelyek igazodnak a szervezet szabályozási előírások és házirendek központi telepítései.

## <a name="monitoring"></a>Figyelés

**Technikai kockázati**: Nem megfelelően megvalósított vagy működése finomhangolt figyelési megakadályozhatja, hogy a számítási feladatok állapotbeli problémák vagy egyéb megfelelőségi állapotfelmérést felismerése.

**Házirendutasítás**: A következő házirendek hajtják végre:

- Cégirányítási eszközök ellenőrizni kell, hogy az összes eszköz kapcsolatos alapvető fontosságú alkalmazásokhoz, vagy a védett adatok szerepelnek megfogyatkoznak erőforrás és optimalizálás figyelése.
- Cégirányítási eszközök ellenőrizni kell, hogy a naplózási adatok megfelelő szintű az összes alapvető fontosságú alkalmazáshoz összegyűjtött vagy a védett adatok.

**A lehetséges tervezési lehetőségek**: [Az Azure Monitor](/azure/azure-monitor/overview) az alapértelmezett figyelési szolgáltatás az Azure platformon, és egységes figyelés használatával kényszeríthető [Azure tervezetek](/azure/governance/blueprints/) erőforrások üzembe helyezésekor.

## <a name="disaster-recovery"></a>Vészhelyreállítás

**Technikai kockázati**: Erőforrás hiba, törlés vagy sérülés járó üzleti szempontból alapvető fontosságú alkalmazások és szolgáltatások és a bizalmas adatok elvesztését eredményezheti.

**Házirendutasítás**: Az összes alapvető fontosságú alkalmazások és a védett adatok rendelkeznie kell valamilyen okból kimaradás lép vagy rendszerhibák üzletmenetre gyakorolt hatás minimalizálása érdekében végrehajtott biztonsági mentési és helyreállítási megoldások.

**A lehetséges tervezési lehetőségek**: Az [Azure Site Recovery] szolgáltatást biztosít a biztonsági mentési, helyreállítási és replikációs funkciók szánt üzleti folytonossági és vészhelyreállítási (BCDR) helyreállítási forgatókönyvek szolgáltatáskimaradás időtartama minimalizálása érdekében.

## <a name="next-steps"></a>További lépések

A cikkben említett kiindulási pontként minták segítségével házirendeket, amelyek igazodnak a felhő bevezetésének csomagok adott üzleti kockázatok auditálására fejlesztése.

A saját egyéni házirend-utasítások erőforrás konzisztencia kapcsolatos fejlesztésének megkezdéséhez töltse le a [erőforrás konzisztencia sablon](template.md).

-K gyorsabb elfogadása, ezen a területen, válassza a [gyakorlatban hasznosítható Cégirányítási utazás](../journeys/overview.md) , hogy a legtöbb szorosan igazodik a környezetében. Ezután módosítsa a kialakítás révén az adott vállalati házirendet érintő döntések.

> [!div class="nextstepaction"]
> [Gyakorlatban hasznosítható Cégirányítási Journey](../journeys/overview.md)
