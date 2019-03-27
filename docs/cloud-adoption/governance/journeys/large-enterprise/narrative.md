---
title: 'CAF: Nagyvállalati cégirányítási narratíva'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Ez narratíva hoz létre egy alkalmazási helyzet nagyvállalati cégirányítási útra.
author: BrianBlanchard
ms.openlocfilehash: b6570b7518f129e5ac14f1639a4bf57e1b4ade69
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298654"
---
# <a name="large-enterprise-the-narrative-behind-the-governance-strategy"></a>Nagyvállalati: A cégirányítási stratégia mögött narratíva

A következő narratíva létrehozza a használati esetek egy [nagyvállalati cégirányítási utazás](./overview.md). A szállítás-k megvalósítása előtt fontos megérteni a feltételek és az indoklást, amely a narratíva is megjelennek. Ezután a saját szervezet utazás cégirányítási stratégia jobban igazíthatja.

## <a name="back-story"></a>Biztonsági másolatot a történetet

Ügyfelek jobb felhasználói élményt foglalja le a vállalati való interakció során. Az aktuális felhasználói élményt piaci erózió okozta, és a tábla egy vezető digitális igazgató (CDO) szolgáltatással vezetett. A CDO marketing és értékesítés, a digitális átalakítás, melyek továbbfejlesztett élmény fog dolgoznak. Ezenkívül számos üzleti egységek legutóbb felvett adatszakértőkön át a farm adatokat, és javíthatja a manuális felületéről tanulási és előrejelzési számos. Informatikai ezen erőfeszítés kiszélesítése, ahol is támogatja. Vannak azonban, hogy értéke kívül esik a szükséges szabályozási és biztonsági vezérlők "informatikai árnyék-infrastruktúra" tevékenységeket.

Az IT-szervezet is felé néző saját kihívásokat. Pénzügyi azt tervezi, a költségvetés folyamatos csökkentése a következő öt évben vezető néhány szükséges költségkeret darabok, ebben az évben indítása. Ezzel szemben a GDPR- és egyéb adatszuverenitási követelményekhez kényszerítése informatikai eszközök további országban adatok honosításához támogatásán. A meglévő adatközpontokat, kettő lejárt, a hardver-frissítések, további, az alkalmazottak és a felhasználói elégedettséget problémát okoz. Három további adatközpontok szükséges hardver frissíti a végrehajtás során az öt év terv. A pénzügyi vezetőnek Követnie kell figyelembe venni a felhő esetében ezek az adatközpontok alternatívájaként szabadítson fel a beruházási költségek a CIO van küld.

A CIO innovatív ötleteit, amely segíthet a vállalat rendelkezik, de ő és saját csapatok elleni következik be, és a költségek kézben korlátozódik. Jelenleg a CDO és az üzleti egysége vezetői egyik keresetekbe a felhőbeli migrálás beszélgetés a CIO társaktól érdeklődés jönnek létre. A három vezetői célja, hogy támogassa a felhő segítségével az üzleti célok elérése érdekében egymással, és azokat a feltárás és a felhő bevezetésének fázisai tervezési megkezdődött.

## <a name="business-characteristics"></a>Üzleti jellemzői

A vállalat rendelkezik a következő üzleti profil:

- Értékesítés és a műveletek ívelhet át több földrajzi területek több piacon globális ügyfelekkel.
- Az üzleti felvásárlásokkal nőtt, és a cél ügyfélkörét alapuló három üzleti egységek között működik. Költségvetés el egy összetett mátrix üzleti egységek és a funkciók.
- Az üzleti megtekinti a legtöbb informatikai befektetési kiürítési vagy egy erőforrás.

## <a name="current-state"></a>Aktuális állapot

Itt van az aktuális állapotát a vállalat informatikai és a felhőbeli műveletek:

- IT több mint 20 saját tulajdonú adatközpontok a világ minden pontján működik.
- Mindegyik adatközpont regionális bérelt vonali lazán kapcsolódó globális WAN létrehozása egy sornyi van csatlakoztatva.
- A megadott informatikai a felhő minden végfelhasználói e-mail áttelepítése révén az Office 365-fiókok. Az áttelepítés befejeződött hat hónappal ezelőtt. Ezt követően csak néhány IT-eszközeivel, a felhőben van telepítve.
- A CDO elsődleges fejlesztési csapata dolgozik a fejlesztési-tesztelési kapacitás a felhőbeli natív funkcióinak bemutatása.
- Egy üzleti egységet a felhőbeli big Data típusú van kísérletezgetést. A BI-csapattól található informatikai törekvés részt.
- A meglévő informatikai cégirányítási házirend szerint az, hogy az ügyfelek személyes azonosításra alkalmas adatokat (PII) és a pénzügyi adatok kell üzemeltethető közvetlenül a vállalat által birtokolt eszközöket. A szabályzat akkor tiltja a felhő bevezetésének kritikus fontosságú alkalmazások vagy a védett adatok.
- IT-beruházásokkal nagymértékben beruházna (CapEx) szabályozza. E befektetések éves bevezetését tervezzük, és gyakran tartalmazzák a karbantartást, csakúgy, mint 3-5 éves függően az Adatközpont meglévő adatfrissítési ciklusok csomagokat.
- A legtöbb befektetéseit a technológia, amelyek nem az éves terv szolgálhatók ki árnyékmásolat-IT erőfeszítéseinek. Ezeket erőfeszítések általában üzleti egységek által felügyelt, és akár az üzemeltetési költségeket az üzleti egység keresztül.

## <a name="future-state"></a>Jövőbeli állapot

A következő változások várhatók a következő néhány évben:

- A CIO annak érdekében, hogy a szabályzat a személyazonosításra alkalmas adatok és a pénzügyi adatok támogatása a jövőbeli célok korszerűsítheti létrejöttéhez vezet. Az informatikai szabályozással csoport két tagjára ebből a törekvésből interakcióban.
- A korai kísérletek az alkalmazás fejlesztői és BI siker vezető mutatók megjelenítése, ha azok minden egyes, ha kevés számítógépet érintő éles megoldásait a felhőre kiadás a következő 24 hónapban.
- A CIO és a pénzügyi igazgató rendelt olyan és az infrastruktúra alelnök költségek elemzése és a megvalósíthatósági vizsgálat létrehozása. Ha a vállalat is, és érdemes 5000 eszközök áthelyezése a felhőbe a következő 36 hónapban ezen erőfeszítés kiszélesítése határozza meg. A sikeres áttelepítéshez lehetővé tenné a két adatközpont, csökkenti a költségeket a több mint 100 USD kiküszöbölése érdekében a CIO M USD-t az öt év terv során. Ha három-négy adatközpontok hasonló eredményeket is működik, a költségvetés vissza a fekete, így az informatikai költségvetés további innovatív kezdeményezések támogatásához lesz.
    ![A helyszíni költség-és Azure-költségek bemutatására egy 100 M USD vissza a következő öt évben](../../../_images/governance/calculator-enterprise.png)
- Együtt a költségmegtakarítás érdekében a vállalat a tervek szerint a véglegesített CapEx összekapcsolásával, a üzemeltetési költségek (OpEx) változáskezelés néhány az IT-beruházásokkal belül informatikai. Ez a változás nagyobb költségvezérlés biztosít, amelyben más tervezett feladatok felgyorsítása használhatja.

## <a name="next-steps"></a>További lépések

A vállalat kidolgozott az alakzat cégirányítási megvalósítása a vállalati házirend. A vállalati szabályzat meghajtók számos, a műszaki döntéseket hozhat.

> [!div class="nextstepaction"]
> [Tekintse át a kezdeti vállalati szabályzat](./initial-corporate-policy.md)
