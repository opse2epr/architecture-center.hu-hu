---
title: 'CAF: Identitás alapmértékekkel, a mutatók és a szervezet kockázattűrési határát'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Identitás alapmértékekkel, a mutatók és a szervezet kockázattűrési határát
author: BrianBlanchard
ms.openlocfilehash: 4722de66308f3d18885ca930925e68e0e756ec03
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298538"
---
# <a name="identity-baseline-metrics-indicators-and-risk-tolerance"></a>Identitás alapmértékekkel, a mutatók és a szervezet kockázattűrési határát

Ebből a cikkből segítséget nyújt az üzleti szervezet kockázattűrési határát összeszámolása, az identitásra építve összefügg. Metrikák és a mutatók segítségével hozhat létre egy üzleti esetet, befektetés végzett az identitásra építve fegyelem stabilizálódó.

## <a name="metrics"></a>Mérőszámok

Identitásra építve az azonosító, hitelesítési és engedélyezési személyek, csoportok felhasználók, vagy automatizált folyamatokkal és a felhőalapú környezetekben lévő erőforrások megfelelő hozzáférést biztosít számukra összpontosít. A kockázati részeként információkat gyűjthet, érdemes analysis kapcsolatos a meghatározásához, hogy mekkora kockázatot identitás-szolgáltatások, arcfelismerés, és mennyire fontos befektetési az identitásra építve az irányítási, hogy a tervezett felhőalapú környezetekben.

A következő példák hasznos metrikák mérik a szervezet kockázattűrési határát, az identitásra építve területen belül összegyűjtendő:

- **Identitáskezelési rendszerek mérete**. Felhasználók, csoportok vagy egyéb az identitáskezelési rendszerek kezelhető objektumok teljes száma.
- **A teljes méretét a címtárszolgáltatások infrastruktúrájának**. Directory-erdők, tartományok és a szervezet által használt bérlők száma.
- **Az örökölt vagy a helyszíni hitelesítési mechanizmusok függőségi**. Függ az örökölt hitelesítési mechanizmusok vagy harmadik féltől származó multi-factor authentication (MFA) szolgáltatások számítási feladatok száma.
- **Felhőbeli üzembe helyezett címtárszolgáltatások mértékben**. Directory-erdők, tartományok és a felhőben üzembe helyezte a bérlők száma.
- **Felhőbeli üzembe helyezett aktív könyvtárak**. Üzembe helyezhető a felhőben Active Directory-kiszolgálók száma.
- **Felhőbeli üzembe helyezett szervezeti egységek**. Active Directory szervezeti egységek számát a felhőbe telepített.
- **Összevonási mértékét**. Az identitásra építve rendszerek száma saját intézménye rendszerei összevonva.  
- **Emelt szintű felhasználók**. Erőforrások vagy a felügyeleti eszközök emelt szintű hozzáféréssel rendelkező felhasználói fiókok száma.
- **Az RBAC használatát**. Az előfizetések, erőforráscsoportok vagy egyes erőforrásokat nem kezelhető a szerepköralapú hozzáférés-vezérlés (RBAC) száma.
- **Hitelesítési jogcímek**. Sikeres és sikertelen felhasználói hitelesítési próbálkozások számát.
- **Engedélyezési jogcímek**. Próbálkozások száma a sikeres és sikertelen az által a felhasználók hozzáférhessenek az erőforrásokhoz.
- **Feltört fiókok**. A sérült egyes felhasználói fiókok száma.

## <a name="risk-tolerance-indicators"></a>Kockázati tolerancia mutatók

A szervezet identitás-infrastruktúra összetettségét nagymértékben kapcsolódó identitásra építve kapcsolatos kockázatokat. Ha a felhasználók és csoportok összes felügyelt egyetlen címtárban vagy felhőbeli natív identitásszolgáltató használatával minimális integráció más szolgáltatásokkal, a kockázati szint valószínűleg is kis. Azonban az üzleti igények növekedésének, az identitásra építve rendszerek előfordulhat, hogy támogatnia kell bonyolultabb esetek, például a cége belső vagy külső identitásszolgáltatókkal összevonási támogatásához több címtár. Ezek a rendszerek egyre összetettebb, kockázati növekszik.

A felhő bevezetésének korai szakaszában, az informatikai biztonsági csoport és az üzleti résztvevőkkel azonosításához együttműködve [üzleti kockázatai](business-risks.md) kapcsolatos identitás, majd vizsgálja meg, az elfogadható referenciakonfigurációjához tartozó identitás szervezet kockázattűrési határát. Ez a szakasz CAF útmutatás példákat talál, de a részletes kockázatokat és a vállalat vagy a központi telepítések alapterveit eltérő lehet.

Ha már van egy alapkonfiguráció elfogadhatatlan mértékű növekedést az azonosított kockázatok a minimális referenciaalapokhoz képest történő létrehozásához. Ezek referenciaalapokhoz képest történő eseményindítók a szerepét, amikor szüksége van arra kockázatok csökkentése érdekében. A következő néhány példa a hogyan kapcsolódnak az identitás a mérőszámokat, például a fent említett, az identitásra építve fegyelem megnövekedett befektetést igazolja.

- **Felhasználói fiók száma eseményindító**. A vállalati felhasználók, csoportok vagy egyéb az identitáskezelési rendszerek a felügyelt objektumok száma több, mint X, befektetés az identitásra építve fegyelem annak biztosítása érdekében a hatékony cégirányítási fiókok nagyszámú keresztül előnyökkel jár.
- **A helyszíni identitás függőségi eseményindító**. A vállalati számítási feladatok migrálása a felhőbe az örökölt hitelesítési képességek vagy harmadik féltől származó MFA igénylő tervezése kell fektethet be az identitásra építve fegyelem újrabontási vagy további felhőalapú infrastruktúra telepítése kockázatok csökkentése érdekében.
- **Directory szolgáltatások összetettséget eseményindító**. Egy vállalati fenntartása, amely több, mint X száma az egyes erdők, tartományok vagy címtárbérlők kell beruházni az identitásra építve fegyelem fiókok kezelése a kockázatok csökkentése érdekében, és a hatékonyság problémák kapcsolódó elosztva a több felhasználói hitelesítő adatok többféle rendszer.
- **A felhőben directory services eseményindító**. A vállalati üzemeltetési X Active Directory server virtuális gépek (VM) a felhőben üzemel, illetve az X számot szervezeti egységek számát a felhő alapú kiszolgálókra felügyelt kihasználhatják a befektetés az identitásra építve fegyelem optimalizálása bármilyen helyszíni vagy más külső identitás szolgáltatások integrációja.
- **Összevonási eseményindító**. A végrehajtási identitás-összevonást az identitásra építve külső rendszerek száma X vállalat fektet be az identitásra építve fegyelem annak biztosítása érdekében a konzisztens a szervezeti házirend összevonási tagok között a is kihasználhatják.
- **Emelt szintű hozzáférés eseményindító**. A vállalat több mint %-a felügyeleti eszközöket és erőforrásokat az emelt szintű engedélyekkel rendelkező felhasználók X figyelembe kell venni, portfóliónk fejlesztésén az identitásra építve fegyelem való nem szándékos túlzott kiosztása a felhasználók számára a hozzáférés kockázatát.
- **Az RBAC-eseményindító**. Egy vállalat alatt X %-a-erőforrások szerepköralapú hozzáférés-vezérlési módszereivel figyelembe kell venni, hardverberuházás az identitásra építve fegyelem optimalizált módjai a felhasználói hozzáférések hozzárendelése az erőforrások azonosítására.
- **Hitelesítési hiba eseményindító**. Ha hitelesítési hibák több, mint X % kísérletet, győződjön meg arról, hogy hitelesítési módszerek nem külső támadás alatt áll, és, hogy a felhasználók meg tudják helyesen hitelesítési módokat kell használnia az identitásra építve fegyelem kell beruházni képviselő cég.
- **Engedélyezési hiba eseményindító**. Próbálja meg egy vállalati, ahol hozzáférési kísérleteket a rendszer elutasítja több mint %-OS X kell beruházni az identitásra építve fegyelem javítása az alkalmazás- és hozzáférés-vezérlés frissítését, és azonosíthatja a potenciálisan rosszindulatú hozzáférésektől.
- **Sérült biztonságú fiók eseményindító**. Sérült biztonságú fiókok száma több, mint X vállalat kell fektethet be az identitásra építve fegyelem erősségét és biztonságának hitelesítési mechanizmusok, és javíthatja a mechanizmusok feltört fiókoktól kockázatok csökkentése érdekében.

A pontos mérőszámokat és eseményindítók összevetheti a szervezet kockázattűrési határát, és a szintet, befektetés az identitásra építve fegyelem használja a szervezet specifikus, de hasznos bázi. a felhő Cégirányítási csapaton belül leírásáért lásd a fenti példák szolgáljon.

## <a name="next-steps"></a>További lépések

Használatával a [Cloud Management-sablon](./template.md), dokumentum-mérőszámok és hibatűrő mutatók, amelyek a jelenlegi cloud bevezetési tervet.

Épület kockázatokat és a hibatűrést, az identitásra építve szabályzat betartása vonatkozó és egy folyamatot hoz létre.

> [!div class="nextstepaction"]
> [Szabályzat betartása folyamatok létrehozása](compliance-processes.md)
