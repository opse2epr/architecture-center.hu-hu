---
title: 'CAF: Mit jelent a felhő házirend felülvizsgálat?'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Mit jelent a felhő házirend áttekintése?
author: BrianBlanchard
ms.openlocfilehash: 2e50c6bac0118db1b6a223244cf8efc43a4ab438
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298537"
---
<!-- markdownlint-disable MD026 -->

# <a name="what-is-a-cloud-policy-review"></a>Mi az a felhőszabályzat-felülvizsgálat?

A **házirend tekintse át a felhő** felé az első lépés [cégirányítási lejárat](../overview.md) a felhőben. Ez a folyamat célja a korszerűsítheti a már meglévő vállalati informatikai házirendeknek. Amikor elkészült, a frissített házirendeket kockázatcsökkentés felhőalapú erőforrások azonos szintű adja meg. Ez a cikk ismerteti a felhő házirend-ellenőrzés során, és annak fontossága.

## <a name="why-perform-a-cloud-policy-review"></a>Miért érdemes felhőalapú házirend felülvizsgálat végrehajtása?

A legtöbb vállalkozás kezelése informatikai folyamatok végrehajtásának keresztül mely házirendek betartatását ciklustól. A kisvállalkozások ezek a szabályzatok is anecdotal, és lazán meghatározott folyamatok. Vállalat növekedésével a nagyobb cégeknek is be szabályzatoknak és folyamatoknak általában sokkal jobban kirajzolódik dokumentált és végrehajtott következetesen.

Vállalatok részletes vállalati informatikai házirendeknek, korábbi technikai döntések függőségek seep szabályzatok betartatását, ami kell. Például az általános vész helyreállítási folyamatok közé tartozik a szabályzatot, amely az előírásoknak, külső helyszíni tárolásra szalagos biztonsági másolatok megtekintéséhez. A belefoglalási feltételezi, hogy egy függőség egy típust (szalagos biztonsági mentések) technológia, amely már nem lehet a leginkább megfelelő megoldást.

Felhőbeli átalakítások hozzon létre egy természetes kihasználás is, gondolja át az örökölt szabályzat a döntéseket hozhat a múltbeli. Technikai funkciók és folyamatok alapértelmezés módosítása jelentősen a felhőben, mint öröklése a kockázatokat. Az előző példában használja, a szalagos biztonsági mentési szabályzat bevezetéséből származik az egypontos meghibásodás kockázatát által gondoskodik az adatok egy helyen, és az üzleti igények szerint a kockázat csökkentéséhez a kockázatú profil minimalizálása érdekében. A felhőbeli üzembe helyezés többféle módon, amely azonos kockázatcsökkentés sokkal alacsonyabb helyreállítási idő célkitűzései (RTO) való kézbesítéséhez. Például:

- Egy natív felhőalapú megoldás sikerült engedélyezni az SQL Azure adatbázis georeplikációs
- Hibrid megoldás az Azure Site Recovery használatával több adatközpontba egy IaaS számítási feladat replikálható.

Egy felhőalapú átalakítás végrehajtásakor szabályzatok gyakran meghatározzák számos, az eszközök, szolgáltatások és a felhő bevezetésének csapatok számára elérhető folyamatok. Ha ezek a házirendek örökölt technológiák alapulnak, gátolják előfordulhat, hogy a csapat erőfeszítések meghajtó módosítása. A legrosszabb esetben fontos házirendek teljesen figyelmen kívül hagyják a migrálás csapat megkerülő megoldások engedélyezése. Sem lesz egy elfogadható serkenti az eredményt.

## <a name="the-cloud-policy-review-process"></a>A felhő házirend-ellenőrzés során

Felhőalapú házirend felülvizsgálatok igazítása a meglévő informatikai szabályozással és informatikai biztonsági szabályzatok az [felhőalapú irányítási öt oktatnak](../overview.md): [Cost Management](../cost-management/overview.md), [biztonsági alapterv](../security-baseline/overview.md), [identitásra építve](../identity-baseline/overview.md), [erőforrás konzisztencia](../resource-consistency/overview.md), és [üzembehelyezésigyorsítás](../deployment-acceleration/overview.md).

A felülvizsgálati folyamat minden egyes e szabályok esetében kövesse az alábbi lépéseket:

1. Tekintse át a meglévő helyszíni az adott szabályok betartását, két lényeges adatpontok keres kapcsolódó szabályzatok: örökölt függőségeket, és az azonosított üzleti kockázatok.
2. Egyes üzleti kockázatok kiértékelése a kérdés feltevésével: "Az üzleti kockázat továbbra is létezik a felhőalapú modell?"
3. A kockázat még létezik-e, ha újból írja a házirendet dokumentálja a szükséges megoldás, nem a technikai megoldáson.
4. Tekintse át a frissített szabályzatot a felhő bevezetésének csapataival szükség kockázatait a lehetséges megoldások megismeréséhez.

## <a name="example-of-a-policy-review-for-a-legacy-policy"></a>Egy házirend tekintse át az örökölt szabályzat – példa

A folyamat példát kell megadni, hogy újra használjuk a szalagos biztonsági mentési szabályzat az előző szakaszban:

- A vállalati szabályzat előírásoknak, külső helyszíni tárolásra szalagos biztonsági mentések az összes éles rendszerek esetén. Ezt a házirendet láthatja a lényeges két adatpont ábrázol:
  - Egy szalagos biztonsági mentési megoldás az örökölt függőség
  - Az éles berendezést ugyanazon a fizikai helyen lévő biztonsági másolatok a tárhellyel kapcsolatos feltételezett üzleti kockázat.
- A kockázat továbbra is létezik? Igen. Akár a felhőben egy egyetlen létesítményén függés néhány kockázati létrehozása. Kisebb a valószínűsége annak, a kockázat, mint a helyszíni megoldást tartalmaz, amely hatással van, hogy az üzleti, de a kockázat még létezik-e.
- Módosítsa úgy a házirend. Adatközpontra kiterjedő katasztrófa esetén egy azt jelenti, hogy az éles rendszerek visszaállítását egy másik adatközpontban és a különböző földrajzi hely a szolgáltatáskimaradás elhárítása után 24 órán belül kell lenniük.
- Tekintse át a felhő bevezetésének csapatok számára. Attól függően, a megoldás megvalósítása nincsenek több azt jelenti, hogy az a erőforrás konzisztencia-szabályzat tartja.

## <a name="tools-to-help-create-modern-policies"></a>Eszközök segítségével modern házirendek létrehozása

Gyorsítsa fel a modern házirendek létrehozása érdekében mintául szolgáló házirendjei minden felhő irányítási öt állapítják érhető el. Ezek a minta-házirendek egyes kezdődik három tervezési feltételezések közül:

- **Natív felhőalapú**: A megoldás üzembe helyezéséhez felhőbeli natív, és alapértelmezés szerinti megoldás minimális konfigurációval Azure-ban található befektethetik is.
- **Vállalati**: A megoldás üzembe helyezéséhez összetett, és a egy hibrid felhőalapú üzembe helyezési modell igényel. Ez szükségessé teszi a bizonyos irányítási oktatnak összetettebb megvalósítását.
- **A tervezési alapelv (CDP) megfelelő felhő**: A megoldás üzembe helyezéséhez meg kell felelnie a CDP, cégirányítási sokkal nagyobb fokú igénylő meghatározott architektúra tengelyeket.  

Az egyes szabályok betartását egy minta szabályzatot kell létrehozni minden egyes ezeket a szinteket. Minden mintához hivatott gondolatait, és beszélgetések a vállalati környezeten belül. Vegye figyelembe, hogy ezek a minták nem tartozhat a megfelelően kialakított vállalati informatikai házirend alternatívájaként használható.
