---
title: 'CAF: Identitás alapkonfiguráció mintautasítások házirend'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Identitás alapkonfiguráció mintautasítások házirend
author: BrianBlanchard
ms.openlocfilehash: 5fad9265b9c048ee502c7e084ddd03faa0ad3e23
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899399"
---
# <a name="identity-baseline-sample-policy-statements"></a>Identitás alapkonfiguráció mintautasítások házirend

Egyes felhőszolgáltatási házirend-utasítások szolgálnak, amelyeket a kockázati értékelés során azonosított adott kockázatkezelés. Ezek az utasítások kell biztosítania a kockázatok tömör összegzését, és azokkal a tervek. Minden egyes utasítás definíciójának tartalmaznia kell adatot megtalálhatja:

- Technikai kockázat – a kockázatokat, ez a szabályzat foglalkozni fog összegzését.
- Házirendutasítás – a házirend követelményeinek törölje összefoglaló leírását.
- Kialakítási lehetőségeket – gyakorlati ajánlásokat készítünk, előírásoknak vagy egyéb útmutatást, amely az IT-csoportoknak és fejlesztők is használhatják, ha a házirend megvalósítása.

A következő minta házirend utasításokat egy közös identitással kapcsolatos üzleti kockázatai számú, és a példaként szolgálnak, hogy a házirend-utasítások használatával saját szervezete igények kidolgozásakor hivatkozhat. Ezek a példák nem jelentenek a proscriptive kell, és a vélhetően több házirend-beállítások kezelésére vonatkozó bármely adott kockázattal rendelkező. Szorosan együttműködnek az üzleti és informatikai csapata hozza létre, azonosíthatja a házirend a kockázatok egyedi készletét leginkább megfelelő megoldásokra.

## <a name="lack-of-access-controls"></a>Hozzáférés-vezérlés hiánya

**Technikai kockázati**: Elégtelen vagy ad hoc hozzáférés-vezérlési beállításokkal bevezethet illetéktelen hozzáférés veszélyét a bizalmas vagy kritikus fontosságú erőforrásokhoz.

**Házirendutasítás**: Az összes eszköz üzembe helyezhető a felhőben az identitások és a jelenlegi cégirányítási szabályzatok által jóváhagyott szerepkörök használatával lehet irányítani.

**A lehetséges tervezési lehetőségek**: [Az Azure Active Directory feltételes hozzáférés](/azure/active-directory/conditional-access/overview) van az alapértelmezett hozzáférés-vezérlési mechanizmus az Azure-ban.

## <a name="overprovisioned-access"></a>Szükségesnél több erőforrással ellátott hozzáférés

**Technikai kockázati**: Felhasználóknak és csoportoknak az a terület erőforrások kézben felelős vezető kimaradások vagy a biztonsági rések jogosulatlan módosítások eredményezhet.

**Házirendutasítás**: A következő házirendek hajtják végre:

- Legkisebb jogosultság hozzáférés modell alapvető fontosságú alkalmazásokat bármely erőforrás vonatkozni fog, vagy a védett adatok.
- Emelt szintű engedélyek kivételt kell lennie, és az ilyen kivételek rögzíteni kell a felhő Cégirányítási csapatával. Kivételek rendszeresen naplózva lesz.

**A lehetséges tervezési lehetőségek**: Tekintse át a [ajánlott eljárások Azure-Identitáskezelés](/azure/security/azure-security-identity-management-best-practices) , amely korlátozza a hozzáférés alapján szerepköralapú hozzáférés-vezérlés (RBAC) stratégia megvalósításához a [ismernie kell](https://wikipedia.org/wiki/Need_to_know) és [minimális jogosultság biztonsági](https://wikipedia.org/wiki/Principle_of_least_privilege) alapelveket.

## <a name="lack-of-shared-management-accounts-between-on-premises-and-the-cloud"></a>A megosztott fiókok a helyszíni és a felhő között hiánya

**Technikai kockázati**: Informatikai felügyelet vagy a fiókok a helyszíni Active Directoryban a felügyeleti munkatársak előfordulhat, hogy nem rendelkezik megfelelő szintű hozzáféréssel felhőalapú erőforrások nem lehet hatékonyan resolve működési vagy biztonsági problémákat.

**Házirendutasítás**: A helyszíni Active Directory-infrastruktúra minden olyan csoport, amely az emelt szintű jogosultságokkal hozzá kell rendelni egy jóváhagyott RBAC-szerepkört.

**A lehetséges tervezési lehetőségek**: A felhőalapú Azure Active Directory és a helyszíni Active Directory közötti hibrid identitáskezelési megoldás megvalósításához, és vegye fel a szükséges helyszíni csoportokat a munkájához szükséges RBAC-szerepkörökhöz.

## <a name="weak-authentication-mechanisms"></a>Gyenge hitelesítési mechanizmusok

**Technikai kockázati**: Sérült biztonságú vagy vélhetően feltört jelszavak, a jogosulatlan hozzáférés biztonságossá tétele a felhőalapú rendszerek főbb kockázata, így identitáskezelő-rendszer kellően biztonságos felhasználói hitelesítési módszerrel, például alapszintű felhasználó/jelszó kombináció, vezethet.

**Házirendutasítás**: Minden fiók a védett erőforrásokhoz egy multi-factor authentication (MFA) módszer használatával való bejelentkezés szükségesek.

**A lehetséges tervezési lehetőségek**: Az Azure Active Directory megvalósítása [Azure multi-factor Authentication](/azure/active-directory/authentication/concept-mfa-howitworks) a felhasználó hitelesítési folyamat részeként.

## <a name="isolated-identity-providers"></a>Elkülönített Identitásszolgáltatók

**Technikai kockázati**: Az Identitásszolgáltatók nem kompatibilis erőforrások vagy szolgáltatások megosztja az ügyfelek vagy egyéb üzleti partnerek foglalásiegység eredményezhet.

**Házirendutasítás**: Minden olyan ügyfél-hitelesítést igénylő alkalmazások üzembe helyezését, amely kompatibilis a belső felhasználók számára az elsődleges identitásszolgáltató jóváhagyott identitásszolgáltatót kell használnia.

**A lehetséges tervezési lehetőségek**: Alkalmazzon [az Azure Active Directory összevonási](/azure/active-directory/hybrid/whatis-fed) között a belső és a felhasználói identitás-szolgáltatóktól.

## <a name="identity-reviews"></a>Identitás-értékelések

**Technikai kockázati**: Idő üzleti változás új magánfelhők számára, vagy más biztonsági szempontok is növelheti az erőforrások védelme a jogosulatlan hozzáférés kockázatát.

**Házirendutasítás**: Felhőalapú Cégirányítási folyamatok tartalmaznia kell negyedéves felülvizsgálati identity management csapataival rosszindulatú, meg kell akadályozni a felhő-eszköz konfigurációja használati minták azonosítása céljából.

**A lehetséges tervezési lehetőségek**: Negyedéves biztonsági felülvizsgálat értekezlet, amely tartalmazza a cégirányítási csapat tagjai és az informatikai munkatársak kezeléséért identitás-szolgáltatások létrehozásához. Tekintse át a meglévő biztonsági adatok és metrikák létrehozására, hézagok az aktuális identitás-kezelési házirend, és azokat az eszközöket, és a frissítési szabályzat minden olyan új kockázatok csökkentése érdekében.

## <a name="next-steps"></a>További lépések

A cikkben említett kiindulási pontként minták segítségével házirendeket, amelyek igazodnak a felhő bevezetésének csomagok adott üzleti kockázatok auditálására fejlesztése.

A saját egyéni házirend-utasítások identitásra építve kapcsolatos fejlesztésének megkezdéséhez töltse le a [identitásra építve sablon](template.md).

-K gyorsabb elfogadása, ezen a területen, válassza a [gyakorlatban hasznosítható Cégirányítási utazás](../journeys/overview.md) , hogy a legtöbb szorosan igazodik a környezetében. Ezután módosítsa a kialakítás révén az adott vállalati házirendet érintő döntések.

> [!div class="nextstepaction"]
> [Gyakorlatban hasznosítható Cégirányítási Journey](../journeys/overview.md)