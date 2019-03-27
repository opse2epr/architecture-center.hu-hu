---
title: 'CAF: Nagyvállalati – többfelhős fejlődést szem előtt tartva'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Nagyvállalati – többfelhős fejlődést szem előtt tartva
author: BrianBlanchard
ms.openlocfilehash: c72040912a99ad232e367ae750e9bb2caa77cbf2
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299378"
---
# <a name="large-enterprise-multi-cloud-evolution"></a>Nagyvállalati: Többfelhős fejlődés

## <a name="evolution-of-the-narrative"></a>A narratíva fejlődésére

Microsoft tisztában van vele, hogy ügyfeleink speciális célokra vannak bevezetése több felhő. Tartson a folyamatban szereplő fiktív cég ez sem kivétel. Az Azure bevezetési folyamata párhuzamosan az üzleti siker a megszerzése egy kis, de egymást kiegészítő üzleti vezetett. Hogy az üzleti fut a különböző felhőszolgáltatók saját informatikai műveleteket.

Ez a cikk bemutatja, hogyan dolgot módosítani, amikor integrálják az új szervezeti. A narratíva alkalmazásában feltételezzük, a cég egyes a leírt az ügyfélfolyamat cégirányítási fejlesztések befejeződött.

### <a name="evolution-of-the-current-state"></a>Az aktuális állapot a fejlődést szem előtt tartva

Ez narratíva korábbi szakaszában a vállalat kezdte költségirányítás megvalósítása, és a költségek figyelése, felhők költségkeret-beállítási bekerül a vállalat normál működési kiadásait.

Azóta, néhány dolog változott, amely hatással lesz a cégirányítási:

- Identitás az Active Directory helyszíni példányát határozza meg. Hibrid identitás – Azure Active Directory-replikáció Eszközértesítések segítik elő.
- IT-üzemeltetők és a Felhőbeli műveletek nagymértékben kezeli az Azure Monitor és a kapcsolódó automatizálását.
- Vész-helyreállítási / üzletmenet-folytonossági Azure Vault példányok vezérli.
- Az Azure Security Center segítségével figyelheti a támadások és biztonsági problémákat.
- Az Azure Security Center és az Azure Monitor is használja a felhőbe irányítása figyeléséhez.
- Azure tervezetek, az Azure Policy és a felügyeleti csoportok segítségével automatizálhatja a megfelelőségi szabályzat.

### <a name="evolution-of-the-future-state"></a>A jövőbeli állapot a fejlődést szem előtt tartva

A cél, hogy a beszerzési vállalati integrálhatja meglévő operations, amikor csak lehetséges.

## <a name="evolution-of-tangible-risks"></a>A képzés résztvevői hasznos képességekkel kockázatok a fejlődést szem előtt tartva

**Üzleti beszerzési költsége**: Az új üzleti felvásárlás nyár kerül piacra, körülbelül öt év nyereséges kell. A lassú megtérülési, mert a tábla szeretne beszerzési költségek, a lehető legnagyobb mértékben szabályozhatja. Annak a kockázata költségvezérlés és technikai integráció egymással ütköző.

Az üzleti kockázat néhány műszaki kockázatok bővíthet.

- Van esély az előállító további beszerzési költségek felhőbe való migrálást.
- Az új környezet, mivel nem megfelelően szabályozott vagy szabálymegsértéseknek az eredményül kapott kockázatát is van.

## <a name="evolution-of-the-policy-statements"></a>A házirend-utasítások fejlődésére

Az alábbi eltérésekkel házirend segítségével csökkentheti a kockázatokat és útmutató új végrehajtása.

1. Minden objektum egy másodlagos felhőbeli meglévő üzemeltetési felügyeleti és monitorozási eszközökkel, biztonsági keresztül kell figyelni.
2. Az összes szervezeti egységek integrálni kell a meglévő identitásszolgáltató.
3. Az elsődleges identitásszolgáltató kell szabályozza a másodlagos felhőbeli eszközökhöz való hitelesítéshez.

## <a name="evolution-of-the-best-practices"></a>Ajánlott eljárást a fejlődést szem előtt tartva

A cikk ezen szakasza a cégirányítási MVP megtervezése és az Azure új szabályzatokat és az Azure Cost Management implementációja fejlődik. Két kialakítási módosítások együtt, a vállalati szabályzat új utasításokat fog teljesítéséhez.

1. Csatlakozás a hálózatok - hajtja végre a hálózatkezelés és informatikai biztonsági, cégirányítási által támogatott
    1. A sor mpls virtuális Magánhálózat/bérelt szolgáltatói kapcsolat hozzáadása az új felhőre hálózatok integrálni. Útválasztási táblázatok és a tűzfal-konfigurációk hozzáadása szabályozza hozzáférés és a forgalom a környezetek között.
2. Identitás-szolgáltatóktól egyesíthetők. A számítási feladatokat a másodlagos felhőben üzemeltetett, attól függően számos van az identity provider összevonása lehetőségek közül. Az alábbiakban néhány példa:
    1. Alkalmazásokhoz, melyek az OAuth 2 hitelesítéséhez az Active Directory, a másodlagos felhőbeli felhasználók sikerült egyszerűen replikálható a meglévő Azure AD-bérlővel.
    2. A rendkívüli a két helyszíni identitás-szolgáltatóktól közötti összevonási lehetővé tenné az Azure-bA replikálni az új Active Directory-tartományok felhasználóit.
3. Eszközök hozzáadása az Azure Site Recovery szolgáltatásba
    1. Az Azure Site Recovery eszközként hibrid/több-cloud Az elejétől lett létrehozva.
    2. Lehet, hogy a virtuális gépek másodlagos felhőbeli tudja védeni a helyszíni eszközök védelmére szolgáló ugyanazokat az Azure Site Recovery a folyamatokat.
4. Eszközök hozzáadása az Azure Cost Management
    1. Az Azure Cost Management egy többfelhős eszközként az elejétől lett létrehozva.
    2. Lehet, hogy egyes felhőszolgáltatók esetében az Azure Cost Management szolgáltatással kompatibilis virtuális gépek másodlagos felhőbeli. További költségekkel terhelhetik.
5. Eszközök hozzáadása az Azure monitornak
    1. Az Azure Monitor az elejétől a hibrid felhő eszközként lett létrehozva.
    2. Lehet, hogy a virtuális gépek másodlagos felhőbeli kompatibilis az Azure Monitor-ügynökök, lehetővé téve számukra az Azure Monitor szerepeltetni a műveletek figyelése.
6. Cégirányítási kényszerítési eszközök
    1. Cégirányítási kényszerítési felhőspecifikus.
    2. A cégirányítási utazás a vállalati házirend nem állnak. A megvalósítás változó lehet cloud cloud, míg a házirend-utasítások a másodlagos szolgáltató is alkalmazható.

A többszörös felhőre való áttérés növekedésével a fenti tervezési fejlődést szem előtt tartva továbbra is részletes.

## <a name="next-steps"></a>További lépések

Számos nagy méretű vállalatok számára, az a felhő Cégirányítási öt Dsciplines való bevezetésének blockers lehet. A következő cikk rendelkezik néhány további ötlete, így cégirányítási egy csapat sport segítik a hosszú távú sikert a felhőben.

> [!div class="nextstepaction"]
> [Cégirányítási rétege](./multiple-layers-of-governance.md)
