---
title: 'CAF: Kis és közepes méretű enterprise - mögött a adatirányítási stratégia a narratíva'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Ez narratíva hoz létre egy alkalmazási helyzet a kis és közepes méretű vállalat cégirányítási útra.
author: BrianBlanchard
ms.openlocfilehash: 563e909ffc2a39b0b52bd26d3a88715fa969a9c2
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299375"
---
# <a name="small-to-medium-enterprise-the-narrative-behind-the-governance-strategy"></a>Small-to-medium enterprise: A cégirányítási stratégia mögött narratíva

A következő narratíva ismerteti a használati eset az [kis és közepes méretű vállalat cégirányítási utazás](./overview.md). A szállítás-k megvalósítása előtt fontos megérteni a feltételek és az indoklást, amely a narratíva is megjelennek. Ezután a saját szervezet utazás cégirányítási stratégia jobban igazíthatja.

## <a name="back-story"></a>Biztonsági másolatot a történetet

A igazgatótanácsi az év lépései dobja fel többféle módon az üzleti terveket. Azok az ügyfelek elégedettségét szerezni a piaci részesedés növelése vezetői küld. Akkor is küld új termékek és szolgáltatások, amelyek a vállalat fog elhelyezése gondolkodási az iparág vezető szolgáltatóként. Is indítani egy párhuzamos annak érdekében, hogy csökkentheti a veszteséget és csökkentheti a szükségtelen költségeket. Megfélemlítsék, bár az a tábla és a vezetői műveletek megjelenítése, hogy ez a tevékenységi kifejezetten processzormagonkénti nagy lehető a jövőbeli növekedést is.

Korábban ezek stratégiai beszélgetések a vállalat informatikai rendelkezik kizárt. Azonban, mert a jövőbeni stratégiai belsőleg kapcsolódó technikai fejlődés informatikai helyet foglal a táblázat segít a big Data jellegű csomagok, rendelkezik. Informatikai most, hogy új módokon várt. A csapat valóban nem előkészített módosítások, és valószínűleg elsajátítható a kihívást jelent.

## <a name="business-characteristics"></a>Üzleti jellemzői

A vállalat rendelkezik a következő üzleti profil:

- Értékesítés és a műveletek találhatók egyetlen országban, és a globális ügyfelek alacsony százalék.
- Költségvetés funkciókat, beleértve a értékesítési, marketinges, műveletek, igazítva, egyetlen üzleti egységet, működik az üzleti és informatikai.
- Az üzleti megtekinti a legtöbb informatikai befektetési kiürítési vagy egy erőforrás.

## <a name="current-state"></a>Aktuális állapot

Itt van az aktuális állapotát a vállalat informatikai és a felhőbeli műveletek:

- Informatikai két üzemeltetett infrastruktúra-környezetben működik. Egy adott környezetben éles eszközöket tartalmaz. A második vész-helyreállítási és tartalmaz néhány fejlesztési-tesztelési eszközök. Két másik szolgáltató által üzemeltetett ezekben a környezetekben. Informatikai hivatkozik ezen két adatközpont éles és a DR jelölik.
- A megadott informatikai a felhő minden végfelhasználói e-mail áttelepítése révén az Office 365-fiókok. Az áttelepítés hat hónapja fejeződött be. Néhány más IT-eszközeivel, a felhőben van telepítve.
- Az alkalmazás fejlesztői csapatok dolgoznak fejlesztési-tesztelési kapacitás a felhőbeli natív funkcióinak bemutatása.
- Az üzleti intelligenciára épülő (BI) csapata van kísérleteztek az összeválogatott adatokat az új platformon és felhőben big Data típusú adatok.
- A vállalat rendelkezik egy lazán meghatározott házirend, amely megállapítja, hogy felhasználói személyes azonosításra alkalmas adatokat (PII), és pénzügyi adatokat nem lehet üzemeltetni a felhőben, amely korlátozza a jelenlegi üzemelő alapvető fontosságú alkalmazásokhoz.
- IT-beruházásokkal nagymértékben beruházna (CapEx) szabályozza. E befektetések éves bevezetését tervezzük. Az elmúlt néhány évben a befektetések vette kis több, mint az alapszintű karbantartási követelményeket.

## <a name="future-state"></a>Jövőbeli állapot

A következő változások várhatók a következő néhány évben:

- Az informatikai rendszer áttekintése a szabályzat a személyazonosításra alkalmas adatok és a pénzügyi adatokat, hogy a jövőbeli állapot célok.
- Az alkalmazások fejlesztését és BI Csapatok szeretne a felhőalapú megoldások az éles környezetben a következő hónapban 24 alapuló a vizuális customer engagement és az új termékek szabadítani.
- Ebben az évben az informatikai csapat fejeződik be a vész helyreállítási feladatok futtatásához a DR-adatközponthoz kivonása áttelepítése 2000 virtuális gépek által a felhőben. Ez várhatóan előállított egy becsült 25 dolláros M USD költséget takaríthat meg a következő öt évben.
    ![A helyszíni költség-és Azure-költségek bemutatására egy 25 dolláros M USD vissza a következő öt évben](../../../_images/governance/calculator-small-to-medium-enterprise.png)
- Belül módosításához, hogyan teszi az IT-beruházásokkal a áthelyezése a véglegesített CapEx-üzemeltetési költségek (OpEx), a csomagok a vállalat informatikai. Ez a változás biztosít nagyobb költségvezérlés és engedélyezése, hogy gyorsítsa fel más tervezett erőfeszítéseket.

## <a name="next-steps"></a>További lépések

A vállalat kidolgozott az alakzat cégirányítási megvalósítása a vállalati házirend. A vállalati szabályzat meghajtók számos, a műszaki döntéseket hozhat.

> [!div class="nextstepaction"]
> [Tekintse át a kezdeti vállalati szabályzat](./initial-corporate-policy.md)
