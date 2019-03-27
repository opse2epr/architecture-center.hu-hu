---
title: 'CAF: Kis és közepes méretű vállalat – alapvető biztonsági fejlődést szem előtt tartva'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: MAGYARÁZAT kis és közepes méretű vállalati – alapvető biztonsági fejlődést szem előtt tartva
author: BrianBlanchard
ms.openlocfilehash: 05bb2f7023c999cdf7154b8189a104ba06950720
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299374"
---
# <a name="caf-small-to-medium-enterprise-security-baseline-evolution"></a>CAF: Small-to-medium enterprise: A biztonsági alapkonfiguráció fejlődése

Ez a cikk a narratíva haladásával biztonsági ellenőrzéseket, amely támogatja a védett adatokról a felhőre való hozzáadásával.

## <a name="evolution-of-the-narrative"></a>A narratíva fejlődésére

Informatikai és üzleti vezető volt elégedett az informatikai, alkalmazásfejlesztés, a korai fázisban Kísérletezési eredményeit, és BI csapat. Vegye figyelembe ezeket a kísérleteket képzés résztvevői hasznos képességekkel üzleti értékeket, a ezeket csapatok engedélyezni kell a védett adatok integrálása megoldások. Ez elindítja a vállalati szabályzat módosításai, de is van szükség a felhőbeli cégirányítási implementációval továbbfejlesztett változata, mielőtt a védett adatok a felhőben is megnyílik.

### <a name="evolution-of-the-cloud-governance-team"></a>A felhő Cégirányítási csapat a fejlődést szem előtt tartva

Adja meg a változó narratíva, és az eddig biztosítunk támogatást, a felhő Cégirányítási csapata most már megtekintett eltérően. A két rendszergazdák, akik a csapat lépések most már tapasztalt felhőben dolgozó tervezők, tekinthetők meg. Ez narratíva fejleszt, mint a érzete árkonstrukcióra legyenek a felhő jegyektől több egy felhőalapú gazdagépőr szolgáltatás szerepkört.

Bár a különbség a változás is, ez fontos különbség informatikai kulturális környezet létrehozásához egy cégirányítási – fókuszban lévő. A felhő Gondviselője megtisztítja a kantinjaik innovatív felhőben dolgozó tervezők által végzett. A két lehetséges szerepkör a természetes fennakadások nélkül használható, és célok ellentétes rendelkezik. Másrészről egy Felhőbeli Gazdagépőr segít biztonságban a felhőben, így más felhőben dolgozó tervezők gyorsabban, kisebb kantinjaik a helyezheti át. Ezenkívül egy felhőalapú Gazdagépőr van készítéshez kapcsolódó sablonok, amelyek gyorsabb üzembe helyezési és bevezetési, így az innováció gyorsító, valamint a felhő irányítási öt oktatnak a defender.

### <a name="evolution-of-the-current-state"></a>Az aktuális állapot a fejlődést szem előtt tartva

Ez narratíva elején az alkalmazás fejlesztői csapatok továbbra is dolgozott a fejlesztési-tesztelési minőségben, és továbbra is a kísérleti fázisban volt a BI-csapattól. Üzemeltetett IT két üzemeltetett infrastruktúra környezet, éles és a DR nevű.

Azóta, néhány dolog változott, amely hatással lesz a cégirányítási:

- Az alkalmazásfejlesztési csoportnak van megvalósítva egy CI/CD-folyamat számára üzembe helyezése felhőbeli natív jobb felhasználói élményt. Amelyet az alkalmazás még nem dolgozhat védett adatok, így nem éles üzemre.
- Az üzleti intelligencia csapat belül informatikai aktívan a logisztika, a készlet és a harmadik féltől származó felhőbeli programnyelveket. Ezeket az adatokat használja fel új előrejelzéseket, üzleti folyamatok sikerült formázása. Azonban ezen előrejelzéseket és insights azokat nem döntéstámogató ügyfél és a pénzügyi adatok integrálhatók a data platform.
- Az informatikai csapat a DR-adatközponthoz kivonása az a CIO és a pénzügyi igazgató az Igazgatókra folyik. A DR adatközpontban 2000 eszközök több mint 1000 elavult, vagy át.
- A személyes adatok és a pénzügyi adatok lazán meghatározott irányelvek rendelkezik lett modernizálta. Azonban az új vállalati szabályzatok olyan függő kapcsolódó biztonsági és cégirányítási szabályzatok megvalósítását. Csoportok továbbra is leáll.

### <a name="evolution-of-the-future-state"></a>A jövőbeli állapot a fejlődést szem előtt tartva

Az alkalmazás fejlesztői és BI csapat által korai kísérletek adatsoron lehetséges az ügyfeleknek környezeteket és az adatvezérelt döntések. A következő 18 hónapban a felhő bevezetésének bontsa ezen megoldások éles környezetben való üzembe helyezésével mindkét csapat szeretné.

A fennmaradó hat hónapban a felhő Cégirányítási csapat fogja végrehajtani, hogy a felhő bevezetésének csapatok ezeket az adatközpontokban a védett adatok áttelepítése a biztonsági és cégirányítási követelmények.

A jelenlegi és jövőbeli állapot módosításai az új házirend-utasítások igénylő új kockázatokat teszik elérhetővé.

## <a name="evolution-of-tangible-risks"></a>A képzés résztvevői hasznos képességekkel kockázatok a fejlődést szem előtt tartva

**Adatok illetéktelen behatolás**: Minden olyan új adatplatformra elfogadásakor nincs lehetséges adatok megsértésével kapcsolatos források természetéből növekedését. A felhőalapú technológiák bevezetése szerelők nőtt feladatkörök csökkentheti a kockázat megoldások megvalósítását. Annak érdekében, hogy ezeket a technikusok feladatok teljesítése egy hatékony biztonsági és szabályozási stratégiát kell végrehajtani.

Az üzleti kockázat néhány műszaki kockázatok bővíthet:

- Alapvető fontosságú applicationss vagy a védett adatok véletlenül is telepíthetők.
- Védett adatok tárolása miatt gyenge titkosítás döntések során előfordulhat, hogy ki lehetnek téve.
- Jogosulatlan felhasználók is hozzáférhetnek a védett adatok.
- Külső behatolás védett adatokhoz való hozzáférést eredményezhet.
- Külső behatolás vagy szolgáltatásmegtagadási támadások ellen, egy üzleti megszakítás előfordulhat.
- Védett adatokhoz való jogosulatlan hozzáféréssel lehetővé teheti a szervezet vagy a munkaviszony módosításokat.
- Új biztonsági rések létrehozhat új behatolás vagy a hozzáférés lehetőségeket.
- Inkonzisztens központi telepítési folyamat, amely adatszivárgás vagy -megszakítások kezelése vezethet biztonsági rések eredményezhet.
- Konfigurációs csúszásokat vagy kihagyott javítások nem kívánt biztonsági rések, amely adatszivárgás vagy -megszakítások kezelése vezethet eredményezhet.

## <a name="evolution-of-the-policy-statements"></a>A házirend-utasítások fejlődésére

Az alábbi eltérésekkel házirend segítségével csökkentheti a kockázatokat és útmutató új végrehajtása. A lista hosszú úgy tűnik, de ezek a házirendek bevezetésének lehet könnyebb, mint a megjelenik.

1. Az összes telepített erőforrás kell lennie az alapján kategorizáljuk, kritikusság és az adatok besorolás. Besorolások vannak, át kell néznie a felhő Cégirányítási csapatot és az alkalmazás tulajdonosa, a felhőben való üzembe helyezés előtt.
2. Az alkalmazásokat, amelyek tárolni, vagy hozzá a védett adatokhoz is eltérően, amelyek nem kezelhető. Minimális, érdemes lehet szegmentált a védett adatok jogosulatlan hozzáférés elkerülése érdekében.
3. Védett adatok inaktív állapotban titkosítani kell.
4. Emelt szintű engedélyekkel a védett adatokat tartalmazó minden szegmensben kivételt kell lennie. Az ilyen kivételek lesz rögzítve a Cloud Cégirányítási csapatával, és rendszeresen ellenőrzi.
5. Alhálózatok, amely tartalmazza a védett adatok más alhálózatokkal elkülönítve kell lennie. Védett adatok az alhálózatok közötti hálózati adatforgalom rendszeresen naplózva lesz.
6. Nincs alhálózat, amely tartalmazza a védett adatok közvetlenül elérhetők a nyilvános interneten keresztül vagy az adatközpontok között. Köztes alhálózatokon keresztül hozzáférést ezekhez az alhálózatokhoz kell átirányítani. Ezekhez az alhálózatokhoz való hozzáférés teljes csomag vizsgálatát, és blokkolja a függvények által elvégezhető tűzfal megoldás segítségével kell származnia.
7. Irányítás azokat az eszközöket kell naplózása és érvényesíti a hálózati konfigurációs követelményei határozzák meg a biztonsági csapat.
8. Cégirányítási eszközök által jóváhagyott lemezképeket csak a virtuális gép üzembe helyezése kell korlátozni.
9. Amikor csak lehetséges, csomópont-konfiguráció kezelése bármely vendég operációs rendszer konfigurációja kell alkalmazni házirend követelményeinek.
10. Irányítás azokat az eszközöket ki kell kényszerítenie, hogy az automatikus frissítések engedélyezve vannak-e az összes telepített erőforrás. Szabálysértések lehet tekintse át a felügyeleti csoportokhoz, és operations házirendek megfelelően javítja. Eszközök, amelyek nem frissülnek automatikusan a tulajdonosa az IT-üzemeltetési folyamatok szerepelnie kell.
11. Az új előfizetési lehetőségekről, vagy bármilyen üzleti szempontból alapvető fontosságú alkalmazások és a védett adatok felügyeleti csoportok létrehozása a felhő Cégirányítási csapattól, győződjön meg arról, hogy van-e hozzárendelve a megfelelő tervezet felülvizsgálat lesz szükség.
12. A minimális jogosultságon alapuló hozzáférés-modell bármely felügyeleti csoport vagy az előfizetést, amely üzleti szempontból kritikus alkalmazásokat tartalmaz, vagy a védett adatok lépnek érvénybe.
13. Trendek és az azokat kihasználó támadások ellen, amelyek hatással lehetnek a magánfelhők számára meg kell vizsgálni rendszeresen a biztonsági csapat számára a felhőben használt biztonsági eszközök által.
14. Központi telepítési eszközkészlet jóvá kell hagynia a felhő Cégirányítási csapatától folyamatos irányítás az üzembe helyezett eszközök.
15. Üzembehelyezési szkriptek kell fenntartani egy központi tárházban elérhető-e a felhő Cégirányítási csapat rendszeres áttekintése és a naplózási.
16. Cégirányítási folyamatok naplózások üzembehelyezési ponton kell megadni, és az összes eszköz közötti összhangot rendszeres ciklusokat, tartalmaznia kell.
17. Minden olyan ügyfél-hitelesítést igénylő alkalmazások üzembe helyezését, amely kompatibilis a belső felhasználók számára az elsődleges identitásszolgáltató jóváhagyott identitásszolgáltatót kell használnia.
18. Felhőalapú cégirányítási folyamatok tartalmaznia kell a negyedéves felülvizsgálatok identity management csapatok számára. Az ilyen segítségével azonosíthatja a rosszindulatú vagy a használati mintákat, meg kell akadályozni a felhő-eszköz konfigurációja.

## <a name="evolution-of-the-best-practices"></a>Ajánlott eljárást a fejlődést szem előtt tartva

A cégirányítási MVP kialakítás tartalmazza az új Azure szabályzatokat és az Azure Cost Management megvalósítását fejlődik. Két kialakítási módosítások együtt, a vállalati szabályzat új utasításokat fog teljesítéséhez.

1. A hálózati és informatikai biztonsági csoportokkal fogja meghatározni a hálózati követelmények. A felhő Cégirányítási csapat támogatni fogja a beszélgetést.
2. Az identitás- és IT-biztonsági csoportokat fogja identitással kapcsolatos követelmények meghatározása, és végezze el a szükséges módosításokat a helyi Active Directory-megvalósítás. A felhő Cégirányítási csapat áttekinti a módosításokat.
3. Hozzon létre egy tárház Azure DevOps tároló és a verziónak megfelelő Azure Resource Manager-sablonok és a parancsfájlalapú konfigurációk.
4. Az Azure Security Center végrehajtása:
    1. Az Azure Security Center konfigurálása, amely tartalmazza a védett adatok besorolások felügyeleti csoporthoz.
    2. Állítsa be az automatikus kiépítést a javítási a megfelelőség biztosítása alapértelmezés szerint.
    3. Operációs rendszer biztonsági konfigurációival létesíteni. A IT-biztonsági csoportja meghatározza a konfiguráció.
    4. Az informatikai biztonsági csapat a Security Center kezdeti használatát támogatja. Áttérés a Security Center használatát az IT-biztonsági csapatának, de a hozzáférés céljából megbízhatóan fejlődő cégirányítási karbantartása.
    5. Hozzon létre egy Resource Manager-sablon, amely tükrözi a változtatásokat egy előfizetésen belül a Security Center-konfigurációja szükséges.
5. Frissítse az összes előfizetéshez tartozó Azure szabályzatokat:
    1. Naplózási és a kritikusság és az adatok besorolás kényszerítése a minden felügyeleti csoportok és az előfizetések, azonosíthatja a védett adatok besorolásokkal rendelkező előfizetése.
    2. Naplózás, és csak a jóváhagyott lemezképeket használatának kényszerítése.
6. Frissítse a védett adatok besorolások tartalmazó összes előfizetéshez tartozó Azure szabályzatokat:
    1. Naplózási és a standard szintű Azure RBAC-szerepkörök csak használatának kényszerítése.
    2. Naplózási és megkövetelik a titkosítást az összes storage-fiókok és a tárolt fájlok az egyes csomópontokon.
    3. Naplózási és az alkalmazás egy NSG-t az összes hálózati adapterekhez és alhálózatokhoz kényszerítése. A hálózati és informatikai biztonsági csoportokkal fogja meghatározni az NSG-t.
    4. Naplózási és engedélyezett hálózati alhálózat és virtuális hálózatok száma hálózati adapterenként használatának kényszerítése.
    5. Naplózási és a felhasználó által megadott útvonaltáblák korlátozás kényszerítése.
    6. A beépített szabályzatok alkalmazása Vendég konfiguráció a következőképpen:
        1. Annak naplózása, hogy a windowsos webkiszolgálók biztonságos kommunikációs protokollokat használnak
        2. Naplózási, hogy a jelszó biztonsági beállításai helyesen vannak-e beállítva a Linux és Windows-gépeken belül
7. Tűzfal-konfiguráció:
    1. Azonosítsa, amely megfelel a szükséges biztonsági követelményeket Azure tűzfal-konfiguráció. Lehetőségként azonosítson egy kompatibilis külső berendezés, amely az Azure-ral kompatibilis.
    2. Hozzon létre egy Resource Manager-sablon üzembe helyezéséhez szükséges konfigurációk a tűzfal.
8. Az Azure tervezet:
    1. Hozzon létre egy új tervezet nevű `protected-data`.
    2. A tűzfal és az Azure Security Center-sablonok hozzáadása a tervezethez.
    3. Adja hozzá az új szabályzatok a védett adatok előfizetésekhez.
    4. A tervrajz közzététele a védett adatok melyik aktuális csomagokat, a üzemeltetési felügyeleti csoporthoz.
    5. Az új tervezet alkalmazása az összes érintett előfizetés, meglévő tervezetek mellett.

## <a name="conclusion"></a>Összegzés

Biztonsági cégirányítási társított hozzáadása a fenti folyamatok és a cégirányítási MVP módosításai segítségével számos olyan kockázatok csökkentése érdekében. Együttesen adnak hozzá a hálózati, identitáskezelési és biztonsági adatok védelméhez szükséges eszközöket.

## <a name="next-steps"></a>További lépések

Felhőre való áttérés folyamatosan fejlődnek, és további üzleti értéket, kockázatokat és a felhő cégirányítási igényeknek is. Tartson a folyamatban szereplő fiktív cég a következő lépés, hogy üzleti szempontból alapvető fontosságú számítási feladatait támogatja. Ez az a pont, amikor szükség van az erőforrás konzisztencia-ellenőrzések.

> [!div class="nextstepaction"]
> [Erőforrás konzisztencia fejlődést szem előtt tartva](./resource-consistency-evolution.md)
