---
title: 'CAF: Nagyvállalati – identitásra építve fejlődést szem előtt tartva'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Nagyvállalati – identitásra építve fejlődést szem előtt tartva
author: BrianBlanchard
ms.openlocfilehash: 7eff9d8e8fd0fd40afa1d9815941fcce8d447579
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298637"
---
# <a name="large-enterprise-identity-baseline-evolution"></a>Nagyvállalati: Az identitáskezelési alapkonfiguráció fejlődése

Ez a cikk a narratíva haladásával a cégirányítási MVP identitásra építve vezérlőelemek hozzáadásával.

## <a name="evolution-of-the-narrative"></a>A narratíva fejlődésére

A két adatközpont felhőalapú migrálásának üzleti indoklásának hagyta jóvá a pénzügyi vezetőnek Követnie. A technikai megvalósíthatósági vizsgálat során több akadályát felderített:

- Védett adatok és alapvető fontosságú alkalmazásokat jelentenek a 25 %-a számítási feladatok a két adatközpontjában. Egyik sem lehet, amíg a jelenlegi cégirányítási házirendek lefektetése személyazonosításra alkalmas adatok nem szükséges, és a kritikus fontosságú alkalmazások rendelkeznek lett modernizálta.
- az eszközök ezen adatközpontokban 7 % nem kompatibilisek felhő. Az Adatközpont szerződés megszűnése előtt egy másodlagos adatközpontba kerül.
- 15 %, az eszközök az Adatközpont (750 virtuális gépek) rendelkezik egy függőség az örökölt vagy külső multi-factor Authentication hitelesítést.
- A VPN-kapcsolatot, amely összekapcsolja a meglévő adatközpontok és az Azure nem nyújt elegendő adatátviteli sebességet vagy késés kivonása az adatközponton belül a két éves ütemterv eszközök áttelepíteni.

Az első két akadályát párhuzamosan problémák elhárításáról vannak folyamatban. Ez a cikk a harmadik és a negyedik akadályát megoldásának foglalkozik.

### <a name="evolution-of-the-cloud-governance-team"></a>A felhő Cégirányítási csapat a fejlődést szem előtt tartva

A felhő Cégirányítási csapata folyamatosan bővül. Miatt a további támogatási identitás felügyeletével kapcsolatban, az identitásra építve csapattól egy rendszergazda már részt vesz egy heti értekezlet tartani a meglévő csoporttagok változásokat figyelembe.

### <a name="evolution-of-the-current-state"></a>Az aktuális állapot a fejlődést szem előtt tartva

Az informatikai csapat jóváhagyás előrelépés két adatközpont kivonja a CIO és a pénzügyi igazgató az Igazgatókra csomagokkal rendelkezik. Azonban informatikai van, az érintett 750 (15 %) az eszközök ezen adatközpontokban kell valahol a felhőben nem helyezhetők.

### <a name="evolution-of-the-future-state"></a>A jövőbeli állapot a fejlődést szem előtt tartva

Az új jövőbeli állapot csomagok, a 750 virtuális gépek migrálása az örökölt hitelesítési követelmények robusztusabb identitásra építve megoldásra van szüksége. Az eszközök más adatközpontokban hasonló százalékos túl az alábbi két adatközpont mutatták érinteni várható.
A jövőbeli állapot mostantól is csak egy kapcsolatot a felhőszolgáltatók és a vállalati mpls virtuális Magánhálózat/bérelt-vonal megoldás.

A jelenlegi és jövőbeli állapot módosítások elérhetővé, amely az új házirend-utasítások lesz szükség az új kockázatokat.

## <a name="evolution-of-tangible-risks"></a>A képzés résztvevői hasznos képességekkel kockázatok a fejlődést szem előtt tartva

**Áttelepítés során üzleti megszakítás**. Migrálás a felhőbe szabályozott, korlátozott idejű kockázata, hogy kezelhető hoz létre. Egy másik részére, amelyben a világ mozgó elévülési hardver sokkal nagyobb eséllyel. A kockázatcsökkentési stratégia az üzleti tevékenységek megszakítások elkerülése érdekében van szükség.

**Meglévő identitás függőségek**. Függőségek a meglévő hitelesítési és identitáskezelési szolgáltatásokat késleltetés, esetleg egyes számítási feladatok a felhőbe való migrálásának megakadályozása. Vissza a két adatközpont időben hiba millió dollárt az Adatközpont lease díjat számítunk fel.

Az üzleti kockázat néhány műszaki kockázatok bővíthet:

- Az örökölt hitelesítés nem feltétlenül érhető el a felhőben, korlátozza az egyes alkalmazások üzembe helyezését.
- Az aktuális megoldáshoz a harmadik felek többtényezős hitelesítés nem feltétlenül érhető el a felhőben, korlátozza az egyes alkalmazások üzembe helyezését.
- Retooling áthelyezése vagy sikerült hozhat létre leállások, illetve adja hozzá a költségek.
- Sebességének és stabilitásának a VPN-áttelepítési előfordulhat, hogy akadályozza.
- A felhőbe érkező forgalmat a globális hálózati egyéb részeinek biztonsági problémákat okozhat.

## <a name="evolution-of-the-policy-statements"></a>A házirend-utasítások fejlődésére

Az alábbi eltérésekkel házirend segítségével csökkentheti a kockázatokat és útmutató új végrehajtása.

1. A kiválasztott felhőszolgáltató csak olyan hitelesítése örökölt módszerek segítségével módszert.
2. A kiválasztott felhőszolgáltató kell kínálnak az aktuális megoldáshoz a külső MFA-hitelesítéshez.
3. Nagy sebességű privát kapcsolatot kell létrehozni a felhőszolgáltatók és a felhőszolgáltató csatlakozik az adatközpontok globális hálózatának köszönhetően a vállalat telekommunikációs szolgáltató között.
4. Megfelelő biztonsági követelmények létrehozása történik, amíg nincs bejövő nyilvános forgalom a felhőben üzemeltetett vállalati eszközök férhetnek hozzá. Minden port blokkolva vannak, a globális WAN-on kívül bármilyen forrásból.

## <a name="evolution-of-the-best-practices"></a>Ajánlott eljárást a fejlődést szem előtt tartva

A cégirányítási MVP tervezési haladásával adja hozzá az Azure új szabályzatokat és az Active Directory megvalósítását a virtuális gépen. Együttesen két kialakítási módosítások teljesítéséhez a vállalati szabályzat új utasításokat.

Az alábbiakban az új ajánlott eljárásokat:

1. Szegélyhálózat (DMZ) tervezet: Engedélyezi a kommunikációt a következő megoldás és a helyszíni Active Directory-kiszolgálók a DMZ-t a helyszíni oldalán kell konfigurálni. Ez az ajánlott eljárás a DMZ-t az Active Directory Domain Services engedélyezése hálózati határok között van szükség.
2. Az Azure Resource Manager-sablonok:
    1. Adjon meg egy NSG letiltja a külső és belső forgalom engedélyezési listáján.
    1. Két AD egy elosztott terhelésű pár terhelés, arany lemezképen alapuló virtuális gépek telepítése. Az első rendszerindításkor a rendszerképet futtatja a PowerShell-szkriptet a tartományhoz, és regisztrálnia kell a tartományi szolgáltatások. További információkért lásd: [kiterjesztheti az Active Directory tartományi szolgáltatások (AD DS) az Azure-bA](../../../../reference-architectures/identity/adds-extend-domain.md).
3. Az Azure Policy: Az NSG vonatkozik a összes erőforrást.
4. Az Azure tervezet
    1. Hozzon létre egy nevű tervezet `active-directory-virtual-machines`.
    1. Adja hozzá minden egyes AD sablonok és házirendek a tervezet.
    1. A tervrajz közzététele a megfelelő felügyeleti csoporthoz.
    1. A tervezet bármely előfizetés örökölt vagy külső MFA hitelesítés megkövetelése a alkalmazni.
    1. Az Azure-ban futó AD-példánya most már használható kiterjesztése, a helyszíni AD-megoldás, lehetővé téve, hogy integrálható a meglévő MFA eszközt, és adja meg a jogcímalapú hitelesítés, mind a meglévő Active Directory-funkciók révén.

## <a name="conclusion"></a>Összegzés

Ezek a változások ad hozzá a cégirányítási MVP segít csökkenteni a kockázatok ebben a cikkben számos lehetővé teszi az egyes cloud bevezetési csapat gyorsan a roadblock ugorja át.

## <a name="next-steps"></a>További lépések

Felhőre való áttérés haladásával és további üzleti értéket, a kockázatokat és a felhő cégirányítási is meg kell fejlődik. Az alábbiakban néhány olyan fejlesztések, amely akkor fordulhat elő. Tartson a folyamatban lévő fiktív vállalat a következő eseményindító a védett adatok felvétele a cloud bevezetési tervet. Ezt a módosítást a további biztonsági ellenőrzéseket lesz szükség.

> [!div class="nextstepaction"]
> [Biztonsági alapkonfiguráció fejlődést szem előtt tartva](./security-baseline-evolution.md)
