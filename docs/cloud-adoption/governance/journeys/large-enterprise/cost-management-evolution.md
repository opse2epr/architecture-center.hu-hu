---
title: 'CAF: Nagyvállalati – Cost Management fejlődést szem előtt tartva'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Nagyvállalati – Cost Management fejlődést szem előtt tartva
author: BrianBlanchard
ms.openlocfilehash: 6bf63e36f6fb19dd0e5f9a16a7f66eb6e0ed54ff
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899226"
---
# <a name="large-enterprise-cost-management-evolution"></a>Nagyvállalati: A Cost Management fejlődést szem előtt tartva

Ez a cikk a narratíva haladásával a termék minimális működőképes (MVP) cégirányítási költségirányítás hozzáadásával.

## <a name="evolution-of-the-narrative"></a>A narratíva fejlődésére

Bevezetési nőtt. a tolerancia kijelző a cégirányítási MVP meghatározott túl. Kiadások növekedése most indokolja, hogy időt, befektetés a felhő Cégirányítási csapat monitorozását és ellenőrzését minták költségeit.

Az innováció, világos illesztőprogramként informatikai legfőképp pedig a költséghely a továbbiakban nem jelenik. Az IT-szervezet kínál a nagyobb értéket, ahogy az informatikai és a pénzügyi igazgató egyetértenek abban, hogy az idő jobbra szerkezetűek az informatikai szerepkör játszik a vállalat. Egyéb változások többek között a pénzügyi vezetőnek Követnie szeretné a közvetlen használatalapú megközelítés az üzleti egységek egyik kanadai ága elszámolása felhőbe. A két kivont adatközpont egyik volt kizárólag üzemeltetett eszközök, üzleti egység kanadai műveletekhez. Ebben a modellben az az üzleti egység kanadai leányvállalata közvetlenül a működési költségeket, az üzemeltetett eszközökhöz kapcsolódó díjat kell fizetni. Ez a modell lehetővé teszi, hogy informatikai kisebb valaki más kezeléséről a költségkeret-beállítási fókusz és további információk az érték létrehozása. Azonban az átállás megkezdése előtt Cost Management azokat az eszközöket kell teljesülniük.

### <a name="evolution-of-current-state"></a>A jelenlegi állapotában a fejlődést szem előtt tartva

Ez narratíva korábbi szakaszában az informatikai csapat volt aktív áthelyezése éles számítási feladatok, védett adatokkal az Azure-bA.

Azóta, néhány dolog változott, amely hatással lesz a cégirányítási:

- 5000 eszközök kivonása kockázatosként megjelölt felhasználókról két adatközpontokból el lettek távolítva. Beszerzés és informatikai biztonsági most megszüntetés a fennmaradó fizikai eszközök.
- Az alkalmazás fejlesztői csapatok CI/CD-folyamatok üzembe helyezéséhez a natív felhőalkalmazások, lényegesen befolyásolnánk az ügyfelek számos van megvalósítva.
- A BI-csapattól összesítését, válogatott, insight és a vezetési kézzelfogható előnyök, az üzleti tevékenységekhez előrejelzési folyamatok hozott létre. Ezeket az előrejelzéseket immár minden szereplője creative új termékeivel és szolgáltatásaival.

### <a name="evolution-of-future-state"></a>Jövőbeli állapot a fejlődést szem előtt tartva

- A Cost figyelési és jelentéskészítési az vehető fel a felhőalapú megoldás. Jelentéskészítés kell elősegítsék a funkciók a felhőköltségek által felhasznált közvetlen működési kiadásait. További jelentési lehetővé teszi, hogy a költségkeret-beállítási figyelése és műszaki útmutatást nyújtanak a cost management. A kanadai ágnak a részleg közvetlenül számlázzuk.

## <a name="evolution-of-tangible-risks"></a>A képzés résztvevői hasznos képességekkel kockázatok a fejlődést szem előtt tartva

**Költségvetés-ellenőrzés**: Nincs egy rejlő kockázat, amely önkiszolgáló képességek az új platformra túlzott mértékű és váratlan költségek eredményez. A költségek figyelése és folyamatos költségeket kockázatot cégirányítási folyamatok annak érdekében, hogy a tervezett költségvetés folyamatos ciklustól kell lennie.

Az üzleti kockázat néhány műszaki kockázatok bővíthet:

- Annak a kockázata, a tényleges költségek meghaladja a csomagban.
- Üzleti feltételek módosítása. Ha igen, lesz esetek, amikor egy üzleti függvényt kell használnia, mint a várt, a kiadások rendellenességeiről vezető további felhőszolgáltatások. Annak a kockázata, hogy ezek a további költségek lenne ellentétben a terv a szükséges korrekciós kerettúllépésként látható. Ha ez sikeres, a kanadai kísérlet segíthet a kockázat csökkentése érdekében.
- Annak a kockázata, a rendszer éppen szükségesnél több erőforrással ellátott, a felesleges költségeket eredményez.

## <a name="evolution-of-the-policy-statements"></a>A házirend-utasítások fejlődésére

Az alábbi eltérésekkel házirend segítségével csökkentheti a kockázatokat és útmutató új végrehajtása.

1. A felhő Cégirányítási csapat hetente terv szemben az összes felhőbeli költségeket kell figyeli. Jelentéskészítés a felhő és a terv közötti eltéréseket, hogy informatikai vezetői megosztva, és pénzügyi havonta. Az összes felhőbeli költségeket és a terv frissítések meg kell vizsgálni az informatikai vezetői és pénzügyi havonta.
2. Minden költséget le kell foglalni egy üzleti függvény accountability célokra
3. Felhőalapú eszközök folyamatosan figyelni kell az optimalizálási lehetőségek
4. Felhőalapú Cégirányítási azokat az eszközöket kell korlátoznia a méretezési beállítások egy jóváhagyott listához konfigurációk eszköz. Az eszközök kell győződjön meg arról, hogy az összes eszköz felderíthető és a költségek figyelési megoldás által nyomon követett.
5. Üzembe helyezés tervezése, során bármilyen társított a termelési számítási feladatokhoz üzemeltetéséhez szükséges felhőerőforrások dokumentálni kell. Ez a dokumentáció segít finomíthatja a költségvetés és készítse elő a drágább beállítások használatának elkerülése érdekében további automatizálását. A folyamat során figyelmet kell fordítani a felhőbeli szolgáltató, például a fenntartott példányok által kínált különböző engedmény eszközök vagy a költségek csökkentésének licenc.
6. Tulajdonosok szükséges vegyen részt az összes alkalmazás jobban szabályozhatja a felhőköltségek tanított eljárások a számítási feladatok optimalizálása.

## <a name="evolution-of-the-best-practices"></a>Ajánlott eljárást a fejlődést szem előtt tartva

A cikk ezen szakasza a cégirányítási MVP megtervezése és az Azure új szabályzatokat és az Azure Cost Management implementációja fejlődik. Két kialakítási módosítások együtt, a vállalati szabályzat új utasításokat fog teljesítéséhez.

1. Az Azure Enterprise portálon a részleg rendszergazda kanadai központi telepítésére vonatkozó számlázási változásai.
2. Az Azure Cost Management megvalósításához.
    1. Létrehozza a megfelelő szintű hozzáférési hatókörrel, hogy összhangba kerüljenek az előfizetés és erőforrás-csoportosítás mintához. Feltéve, hogy a cégirányítási MVP ciklustól meghatározott előzetes cikkek ehhez arra lenne szükség **regisztrációs fiók hatókörében** hozzáférését a felhőalapú Cégirányítási csapat végrehajtása a magas szintű reporting. Kell irányítási, például a kanadai beszerzési csapat kívül további csapatainak **erőforrás csoporthatókör** hozzáférést.
    2. Költségvetési létesíteni az Azure Cost Managementben.
    3. Tekintse át, és reagálhat rájuk kezdeti ajánlások. Azt javasoljuk, hogy rendelkezik egy ismétlődő folyamat a jelentéskészítési folyamat támogatásához.
    4. Konfigurálja, és hajtsa végre az Azure Cost felügyelet – jelent, kezdeti és a ismétlődő.
3. Az Azure Policy Update.
    1. Naplózási címkézés, a felügyeleti csoport, az előfizetés és az erőforráscsoport értékeit bármilyen eltérés azonosításához.
    2. Termékváltozat méretbeállítások korlátozni a központi telepítés tervezési dokumentációban szereplő SKU-k létrehozásához.

## <a name="conclusion"></a>Összegzés

Cégirányítási hozzáadása a fenti folyamatok és az MVP segít mérsékelni az számos olyan kockázatok cégirányítási módosításai társított költségek. Ezek készítsen a láthatóságot, elszámoltathatóság és a költségek csökkentését kell optimalizálni.

## <a name="next-steps"></a>További lépések

Felhőre való áttérés folyamatosan fejlődnek, és további üzleti értéket, kockázatokat és a felhő cégirányítási kell is fejlődik. Tartson a folyamatban lévő fiktív vállalat a következő lépésben használja a cégirányítási befektetés több felhő kezelése.

> [!div class="nextstepaction"]
> [Több felhő fejlődést szem előtt tartva](./multi-cloud-evolution.md)