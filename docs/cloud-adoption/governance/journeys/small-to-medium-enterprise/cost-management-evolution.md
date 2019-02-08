---
title: 'CAF: Kis és közepes méretű vállalat – Cost Management fejlődést szem előtt tartva  '
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 2/11/2019
description: MAGYARÁZAT kis és közepes méretű vállalati – Cost Management fejlődést szem előtt tartva
author: BrianBlanchard
ms.openlocfilehash: ca070bfca3efbf9548faa8cf28a1adffefd4a33a
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55898758"
---
# <a name="small-to-medium-enterprise-cost-management-evolution"></a>Small-to-medium enterprise: A Cost Management fejlődést szem előtt tartva

Ez a cikk a narratíva haladásával a cégirányítási MVP költségirányítás hozzáadásával.

## <a name="evolution-of-the-narrative"></a>A narratíva fejlődésére

Bevezetési túl a költségek tolerancia kijelző a cégirányítási MVP meghatározott nőtt. Ez a lehetőség egy jó dolog, megfelel-e a "DR" adatközpontból áttelepítéseket. Idő, befektetés indokolja, hogy a kiadások növekedése most már a felhő Cégirányítási csapattól.

### <a name="evolution-of-the-current-state"></a>Az aktuális állapot a fejlődést szem előtt tartva

Ez narratíva előző fázisában informatikai korábban kivont 100 %-a DR-adatközponthoz. Az alkalmazások fejlesztését és BI-csoportok lettek készen áll az éles forgalmat.

Azóta, néhány dolog változott, amely hatással lesz a cégirányítási:

- Az áttelepítés csapata még kezdődött meg az éles adatközponton kívüli át virtuális gépek.
- Az alkalmazás fejlesztői csapatok aktívan van leküldése az éles üzemi alkalmazások pedig a felhőbe, CI/CD-folyamatok keretében. Ezeket az alkalmazásokat reaktív méretezheti a felhasználói igényeknek.
- Az üzleti intelligencia csapatot belül informatikai rendelkezik kézbesíti a felhőbeli prediktív elemzési eszközök száma. a felhőben összesített adatokat mennyiségű fejlesztéseiről.
- Ennek a növekedésnek mindegyikét támogatja a véglegesített üzleti eredményeket. Azonban költségek gomba megkezdődött. Előre jelzett költségvetése vártnál gyorsabban növekszik. A pénzügyi vezetőnek Követnie kell a költségkezeléshez jobb módszerek.

### <a name="evolution-of-the-future-state"></a>A jövőbeli állapot a fejlődést szem előtt tartva

A Cost figyelési és jelentéskészítési az vehető fel a felhőalapú megoldás. Informatikai továbbra is szolgálja ki, a költség elszámolási ház. Ez azt jelenti, hogy a cloud services fizetési továbbra is IT-beszerzési származnak. Azonban reporting kell elősegítsék a funkciók a felhőköltségek által felhasznált közvetlen működési kiadásait. Ez a modell "Vissza megjelenítése" a felhőbe a számlázási modell nevezzük.

A jelenlegi és jövőbeli állapot módosítások elérhetővé, amely az új házirend-utasítások lesz szükség az új kockázatokat.

## <a name="evolution-of-tangible-risks"></a>A képzés résztvevői hasznos képességekkel kockázatok a fejlődést szem előtt tartva

**Költségvetés-ellenőrzés**: Nincs egy rejlő kockázat, amely önkiszolgáló képességek az új platformra túlzott mértékű és váratlan költségek eredményez. A költségek figyelése és folyamatos költségeket kockázatot cégirányítási folyamatok annak érdekében, hogy a tervezett költségvetés folyamatos ciklustól kell lennie.

Az üzleti kockázat néhány műszaki kockázatok bővíthet:

- Tényleges költségek meghaladhatja a tervet.
- Üzleti feltételek módosítása. Ha igen, lesz esetek, amikor egy üzleti függvényt kell használnia, mint a várt, a kiadások rendellenességeiről vezető további felhőszolgáltatások. Annak a kockázata, hogy a felesleges költségeket minősülnek Kerettúllépések, a terv a szükséges korrekciós figyelésekor.
- Rendszerek sikerült kell overprovisioned, a felesleges költségeket eredményez.

## <a name="evolution-of-the-policy-statements"></a>A házirend-utasítások fejlődésére

Az alábbi eltérésekkel házirend segítségével csökkentheti a kockázatokat és útmutató új végrehajtása.

1. A cégirányítási csapat hetente terv szemben az összes felhőbeli költségeket kell figyeli. Jelentéskészítés a felhő és a terv közötti eltéréseket, hogy informatikai vezetői megosztva, és pénzügyi havonta. Az összes felhőbeli költségeket és a terv frissítések meg kell vizsgálni az informatikai vezetői és pénzügyi havonta.
2. Minden költséget accountability célokra üzleti függvény le kell foglalni.
3. Felhőalapú eszközök az optimalizálási lehetőségek folyamatosan figyelni kell.
4. Felhőalapú Cégirányítási azokat az eszközöket kell korlátoznia a méretezési beállítások egy jóváhagyott listához konfigurációk eszköz. Az eszközök kell győződjön meg arról, hogy az összes eszköz felderíthető és a költségek figyelési megoldás által nyomon követett.
5. Üzembe helyezés tervezése, során bármilyen társított a termelési számítási feladatokhoz üzemeltetéséhez szükséges felhőerőforrások dokumentálni kell. Ez a dokumentáció segít finomíthatja a költségvetés és készítse elő a további automation drágább beállítások használatának elkerülése érdekében. A folyamat során figyelmet kell fordítani a felhőbeli szolgáltató, például a fenntartott példányok vagy licencköltségeit kedvezményeket által kínált különböző engedmény eszközök.
6. Tulajdonosok szükséges vegyen részt az összes alkalmazás jobban szabályozhatja a felhőköltségek tanított eljárások a számítási feladatok optimalizálása.

## <a name="evolution-of-the-best-practices"></a>Ajánlott eljárást a fejlődést szem előtt tartva

A cikk ezen szakasza a cégirányítási MVP megtervezése és az Azure új szabályzatokat és az Azure Cost Management implementációja fejlődik. Két kialakítási módosítások együtt, a vállalati szabályzat új utasításokat fog teljesítéséhez.

1. Az Azure Cost Management megvalósítása
    1. Létrehozza a hozzáférést az előfizetés minta és az erőforrás konzisztencia fegyelem megfelelő hatókörét. Feltéve, hogy a cégirányítási MVP ciklustól meghatározott előzetes cikkeket, ehhez **regisztrációs fiók hatókörében** a felhő Cégirányítási csapat végrehajtása a jelentéskészítő magas szintű hozzáférést. További csapatok cégirányítási kívül szükség lehet **erőforrás csoporthatókör** hozzáférést.
    2. Költségvetési létesíteni az Azure Cost Managementben.
    3. Tekintse át, és reagálhat rájuk kezdeti ajánlások. Rendelkezik egy ismétlődő folyamat támogatásához a jelentéskészítés.
    4. Konfigurálja, és hajtsa végre az Azure Cost felügyelet – jelent, kezdeti és a ismétlődő.
2. Az Azure Policy Update
    1. Naplózás, a címkézés, a felügyeleti csoport, előfizetés és erőforrás-csoport értékeinek bármilyen eltérés azonosításához.
    2. Termékváltozat méretbeállítások korlátozni a központi telepítés tervezési dokumentációban szereplő SKU-k létrehozásához.

## <a name="conclusion"></a>Összegzés

Cégirányítási hozzáadása ezeket a folyamatokat és az MVP segít mérsékelni az számos olyan kockázatok cégirányítási módosításai társított költségek. Ezek készítsen a láthatóságot, elszámoltathatóság és a költségek csökkentését kell optimalizálni.

## <a name="next-steps"></a>További lépések

Felhőre való áttérés folyamatosan fejlődnek, és további üzleti értéket, kockázatokat és a felhő cégirányítási kell is fejlődik. Tartson a folyamatban lévő fiktív vállalat a következő lépésben használja a cégirányítási befektetés több felhő kezelése.

> [!div class="nextstepaction"]
> [Több felhő fejlődést szem előtt tartva](./multi-cloud-evolution.md)
