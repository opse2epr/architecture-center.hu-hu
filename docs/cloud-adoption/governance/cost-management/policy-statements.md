---
title: 'CAF: A Cost Management minta házirend-utasítások'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 1/4/2019
description: A Cost Management minta házirend-utasítások
author: BrianBlanchard
ms.openlocfilehash: 1c8b94ae5285fa26cdf9760892beaced2487af8b
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899411"
---
# <a name="cost-management-sample-policy-statements"></a>A Cost Management minta házirend-utasítások

Egyes felhőszolgáltatási házirend-utasítások szolgálnak, amelyeket a kockázati értékelés során azonosított adott kockázatkezelés. Ezek az utasítások kell biztosítania a kockázatok tömör összegzését, és azokkal a tervek. Minden egyes utasítás definíciójának tartalmaznia kell adatot megtalálhatja:

- Üzleti kockázat – a kockázatokat, ez a szabályzat foglalkozni fog összegzését.
- Házirendutasítás – a házirend követelményeinek törölje összefoglaló leírását.
- Kialakítási lehetőségeket – gyakorlati ajánlásokat készítünk, előírásoknak vagy egyéb útmutatást, amely az IT-csoportoknak és fejlesztők is használhatják, ha a házirend megvalósítása.

A következő házirend mintautasítások cím számos gyakori üzleti költségek kapcsolatos kockázatokat, és a példaként szolgálnak, hogy a tényleges házirend-utasítások saját szervezete igényei szerint címzés tervezése során hivatkozhat. Ezek a példák nem jelentenek a proscriptive kell, és potenciálisan több házirend-beállítások kezelésére vonatkozó bármely egyetlen azonosított kockázattal rendelkező. Szorosan együttműködnek az üzleti és informatikai csapata hozza létre, azonosíthatja a házirend az adott költség kapcsolatos kockázatokat leginkább megfelelő megoldásokra.  

## <a name="future-proofing"></a>Jövőbeli ellenőrző

**Üzleti kockázat**. Aktuális feltételek, amelyeknek nem garantálja a Cost Management fegyelem a cégirányítási csapattól egy befektetése jelenti. Azonban várhatóan befektetni a jövőben.

**Házirendutasítás**. Üzembe helyezhető a felhőben egy számlázási egység, alkalmazások és számítási feladatok és szabványok elnevezési megfelel az összes eszköz kell társítania. Ez a szabályzat biztosítja, hogy a jövőbeli Cost Management erőfeszítések től lép érvénybe.

**Kialakítási lehetőségeket**. A jövőjét alapítvány létrehozásáról szóló információkért lásd: a irányítás MVP az létrehozásához kapcsolódó hozzászólások a [kialakítás döntéstámogató útmutatók](../journeys/overview.md) CAF útmutatást részét.

## <a name="budget-overruns"></a>Költségvetési keretek túllépése

**Üzleti kockázat:** Önkiszolgáló üzembe helyezési hoz létre a túlköltekezés kockázata.

**A házirend-utasítás:** Minden felhő üzembe helyezése egy számlázási egység jóváhagyott költségvetés és a egy költségvetési korlátokat mechanizmust le kell foglalni.

**Kialakítási lehetőségeit:** Az Azure-ban, a költségvetés szabályozható [Azure Cost Management](/azure/cost-management/manage-budgets)

## <a name="underutilization"></a>Alulkihasználtságának

**Üzleti kockázat:** A vállalat rendelkezik előre fizetett a cloud services vagy által végrehajtott, amely egy adott mennyiség elkölthető egyéves kötelezettséget kell vállalni. Annak a kockázata, hogy az egyeztetett összeg után nem fogja használni, egy elveszett befektetési eredményez.

**A házirend-utasítás:** Egy lefoglalt felhőalapú költségvetés mindegyik számlázási egység felel meg évente, költségvetéseket, negyedévente szeretné költségvetése módosíthatja, és az idő lefoglalása véleményezésére vonatkozó havi tervezett és tényleges kiadásokat. A számlázási egység vezető mellett nagyobb, mint 20 %-os eltérések havonta tárgyalják. Követési célból, egy számlázási egység az összes erőforrás hozzárendelése.

**Kialakítási lehetőségeit:**

- Az Azure-ban, tervezett és tényleges kiadásokat keresztül kezelhetők [Azure Cost Management](/azure/cost-management/quick-acm-cost-analysis)
- Többféle módon is számlázási egység szerint erőforrások csoportosítása. Az Azure-ban egy [konzisztencia erőforrásmodell](../../decision-guides/resource-consistency/overview.md) kell kiválasztani a cégirányítási csapat együtt és alkalmazza az összes eszköz.

## <a name="overprovisioned-assets"></a>Szükségesnél több erőforrással ellátott adategységek

**Üzleti kockázat:** A hagyományos helyszíni adatközpontokhoz általános gyakorlat, hogy az eszközök üzembe további kapacitás megtervezése a növekedést a távoli jövőben. A felhő gyorsabban, mint a hagyományos berendezések méretezhető. Eszközök a felhőben is díjszabás technikai kapacitás alapján. Annak a kockázata, a régi helyszíni gyakorlat mesterségesen fúvódnia a felhő költségeit.

**A házirend-utasítás:** Bármely eszköz üzembe helyezhető a felhőben is figyelheti a kihasználtság és jelentéskészítés a valamelyik kapacitáshoz ajánlatunkban használat 50 %-át program regisztrálva kell lenniük. Bármely eszköz üzembe helyezhető a felhőben vannak csoportosítva vagy címkézett logikai módon, így cégirányítási csapat tagjai is léphet a szükségesnél több erőforrással ellátott adategységek bármely optimalizálása kapcsolatos számítási feladatok felelőse.

**Kialakítási lehetőségeit:**

- Az Azure-ban [az Azure Advisor](/azure/advisor/advisor-cost-recommendations) optimalizálási javaslatokat is biztosít.
- Többféle módon is számlázási egység szerint erőforrások csoportosítása. Az Azure-ban egy [konzisztencia erőforrásmodell](../../decision-guides/resource-consistency/overview.md) kell kiválasztani a cégirányítási csapat együtt és alkalmazza az összes eszköz.

## <a name="overoptimization"></a>Overoptimization

**Üzleti kockázat:** Költségkezelés érvényes ténylegesen hozhat létre az új kockázatokat. A költségkeret-beállítási optimalizálása a rendszer teljesítményének INVERZ. Ha csökkenti a költségeket, annak a kockázata, kiadások és gyenge felhasználói élményeket előállító overtightening.

**A házirend-utasítás:** Olyan eszköz, amely közvetlen hatással van az ügyfelek azonosítani kell csoportosítás, vagy a címkézést. Olyan eszköz, amely hatással van a felhasználói élmény optimalizálása, mielőtt a felhő Cégirányítási csapat módosítania kell az optimalizálás nem lehet kevesebb, mint 90 nap a kihasználtsági trendek alapján. A dokumentum bármely szezonális vagy eseményvezérelt adatlöketekkel számít a eszközök optimalizálása.

**Kialakítási lehetőségeit:**

- Az Azure-ban [Azure Monitor insights szolgáltatásainak](/azure/azure-monitor/insights/vminsights-performance) segíthet elemzési rendszer-felhasználás mintavételezését.
- Többféle módon is csoportosítási és címkézés erőforrások, szerepkörök alapján. Az Azure-ban, válasszon egy [konzisztencia erőforrásmodell](../../decision-guides/resource-consistency/overview.md) együtt a cégirányítási csapat és a alkalmazni az összes eszköz.

## <a name="next-steps"></a>További lépések

A cikkben említett kiindulási pontként minták segítségével házirendeket, amelyek igazodnak a felhő bevezetésének csomagok adott üzleti kockázatok auditálására fejlesztése.

Fejlesztés a saját egyéni házirend-utasítások Cost Management kapcsolatos első lépésként töltse le a [Cost Management-sablon](template.md).

-K gyorsabb elfogadása, ezen a területen, válassza a [gyakorlatban hasznosítható Cégirányítási utazás](../journeys/overview.md) , hogy a legtöbb szorosan igazodik a környezetében. Ezután módosítsa a kialakítás révén az adott vállalati házirendet érintő döntések.

> [!div class="nextstepaction"]
> [Gyakorlatban hasznosítható Cégirányítási Journey](../journeys/overview.md)
