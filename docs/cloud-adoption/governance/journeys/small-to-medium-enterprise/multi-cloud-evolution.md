---
title: 'CAF: Kis és közepes méretű vállalat – többfelhős fejlődést szem előtt tartva'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Kis és közepes méretű vállalat – többfelhős fejlődést szem előtt tartva
author: BrianBlanchard
ms.openlocfilehash: 8bfe56f999ddef9d954fad9a157277301d81a98e
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298853"
---
# <a name="small-to-medium-enterprise-multi-cloud-evolution"></a>Small-to-medium enterprise: Többfelhős fejlődés

Ez a cikk a narratíva haladásával vezérlőelemeket több felhő bevezetésére vonatkozóan.

## <a name="evolution-of-the-narrative"></a>A narratíva fejlődésére

Microsoft tisztában van vele, hogy ügyfeleink speciális célokra vannak bevezetése több felhő. A képzeletbeli vevő tartson a folyamatban lévő ez sem kivétel. Az Azure bevezetési folyamata párhuzamosan az üzleti siker a megszerzése egy kis, de egymást kiegészítő üzleti vezetett. Hogy az üzleti fut a különböző felhőszolgáltatók saját informatikai műveleteket.

Ez a cikk bemutatja, hogyan dolgot módosítani, amikor integrálják az új szervezeti. A narratíva alkalmazásában feltételezzük, a cég egyes a leírt az ügyfélfolyamat cégirányítási fejlesztések befejeződött.

### <a name="evolution-of-the-current-state"></a>Az aktuális állapot a fejlődést szem előtt tartva

Ez narratíva korábbi szakaszában a vállalat kezdte aktívan leküldése az éles üzemi alkalmazások pedig a felhőbe, CI/CD-folyamatok keretében.

Azóta, néhány dolog változott, amely hatással lesz a cégirányítási:

- Identitás az Active Directory helyszíni példányát határozza meg. Hibrid identitás – Azure Active Directory-replikáció Eszközértesítések segítik elő.
- IT-üzemeltetők és a Felhőbeli műveletek nagymértékben kezeli az Azure Monitor és a kapcsolódó automatizálását.
- Vész-helyreállítási / üzletmenet-folytonossági Azure Vault példányok vezérli.
- Az Azure Security Center segítségével figyelheti a támadások és biztonsági problémákat.
- Az Azure Security Center és az Azure Monitor is használja a felhőbe irányítása figyeléséhez.
- Azure tervek, az Azure Policy és az Azure felügyeleti csoportok segítségével automatizálhatja a megfelelőségi szabályzat.

### <a name="evolution-of-the-future-state"></a>A jövőbeli állapot a fejlődést szem előtt tartva

A cél, hogy a beszerzési vállalati integrálhatja meglévő operations, amikor csak lehetséges.

## <a name="evolution-of-tangible-risks"></a>A képzés résztvevői hasznos képességekkel kockázatok a fejlődést szem előtt tartva

**Üzleti beszerzési költsége**: Az új üzleti felvásárlás nyár kerül piacra, körülbelül öt év nyereséges kell. A lassú megtérülési, mert a tábla szeretne beszerzési költségek, a lehető legnagyobb mértékben szabályozhatja. Annak a kockázata költségvezérlés és technikai integráció egymással ütköző.

Az üzleti kockázat néhány műszaki kockázatok bővíthet:

- Migrálás a felhőbe eredménykészlethez további beszerzési költségek
- Az új környezet előfordulhat, hogy nem lehet megfelelően szabályozott szabálymegsértéseknek járhat.

## <a name="evolution-of-the-policy-statements"></a>A házirend-utasítások fejlődésére

Az alábbi eltérésekkel házirend segítségével csökkentheti a kockázatokat és útmutató új végrehajtása.

1. Minden objektum egy másodlagos felhőben kell figyelni meglévő üzemeltetési felügyeleti és monitorozási eszközökkel, biztonsági keresztül
2. Az összes szervezeti egységben kell integrálható a meglévő identitásszolgáltató
3. Az elsődleges identitásszolgáltató kell szabályozzák a másodlagos felhőbeli eszközökhöz való hitelesítés

## <a name="evolution-of-the-best-practices"></a>Ajánlott eljárást a fejlődést szem előtt tartva

A cikk ezen szakasza a cégirányítási MVP megtervezése és az Azure új szabályzatokat és az Azure Cost Management implementációja fejlődik. Két kialakítási módosítások együtt, a vállalati szabályzat új utasításokat fog teljesítéséhez.

1. Csatlakozás a hálózatokat. Ez a lépés hajtja végre a hálózati és IT-biztonsági csoportokat, és a felhő Cégirányítási csapat által támogatott. Az mpls virtuális Magánhálózat/bérelt-vonal szolgáltatói kapcsolat hozzáadása az új felhőre hálózatok integrálni. Útválasztási táblázatok és a tűzfal-konfigurációk hozzáadása szabályozza hozzáférés és a forgalom a környezetek között.
2. Identitás-szolgáltatóktól egyesíthetők. A számítási feladatokat a másodlagos felhőben üzemeltetett, attól függően számos van az identity provider összevonása lehetőségek közül. Az alábbiakban néhány példa:
    1. Alkalmazásokhoz, melyek az OAuth 2 hitelesítéséhez az Active Directoryból a másodlagos felhőbeli felhasználók egyszerűen replikálható a meglévő Azure AD-bérlővel. Ez biztosítja, hogy minden felhasználó hitelesíthető a bérlőben.
    2. Összevonási, vagyis lehetővé teszi a szervezeti egységek áramlását az Active Directory a helyszínen, majd be az Azure AD-példányt.
3. Eszközök hozzáadása az Azure Site Recovery szolgáltatásba.
    1. Az Azure Site Recovery úgy lett kialakítva, hibrid vagy több-cloud eszköz az elejétől.
    2. Lehet, hogy a virtuális gépek másodlagos felhőbeli tudja védeni a helyszíni eszközök védelmére szolgáló ugyanazokat az Azure Site Recovery a folyamatokat.
4. Eszközök hozzáadása az Azure Cost Management
    1. Az Azure Cost Management egy többfelhős eszközként az elejétől úgy lett kialakítva.
    2. Lehet, hogy egyes felhőszolgáltatók esetében az Azure Cost Management szolgáltatással kompatibilis virtuális gépek másodlagos felhőbeli. További költségekkel terhelhetik.
5. Eszközök hozzáadása az Azure Monitor.
    1. Az Azure Monitor úgy lett kialakítva, a tervezéstől a hibrid felhő eszközként.
    2. Lehet, hogy a virtuális gépek másodlagos felhőbeli kompatibilis az Azure Monitor-ügynökök, lehetővé téve számukra az Azure Monitor szerepeltetni a műveletek figyelése.
6. Cégirányítási kényszerítési eszközök:
    1. Cégirányítási kényszerítési felhőspecifikus.
    2. A cégirányítási utazás a vállalati házirend nem felhőspecifikus állnak. A megvalósítás változó lehet cloud cloud, míg a szabályzatok alkalmazhatók a másodlagos szolgáltató.

A többszörös felhőre való áttérés növekedésével a fenti tervezési fejlődést szem előtt tartva továbbra is részletes.

## <a name="conclusion"></a>Összegzés

A cikksorozat vázolt cégirányítási ajánlott eljárásokat, kitalált vállalat tapasztalatainak igazítva rendszergazdánál veszi kezdetét. Kis elindításával, de a megfelelő épül a vállalat sikerült gyorsan és cégirányítási a megfelelő időben a megfelelő mennyiségű még továbbra is érvényesek. Az MVP önmagában nem védte az ügyfél. Ehelyett a kockázatok, és adja hozzá a védelmi foundation létrehozza azt. Itt cégirányítási rétegeit szabályzati képzés résztvevői hasznos képességekkel kockázatok csökkentése érdekében. Az itt bemutatott pontos út nem a 100 %-os bármely olvasó a folyamattal rendelkező igazítása Ehelyett szolgál mintaként a növekményes cégirányítási. Az olvasó javasolt elképzelt az ajánlott eljárások a saját egyedi korlátozások és irányítási követelmények vonatkoznak.