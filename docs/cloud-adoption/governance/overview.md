---
title: 'CAF: Irányítás az Azure-hoz a Microsoft CAF'
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Irányítás az Azure-hoz a Microsoft CAF
author: BrianBlanchard
ms.openlocfilehash: b7a0739a42a27def34955577acffc7ab7292ac50
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299370"
---
# <a name="governance-in-the-microsoft-caf-for-azure"></a>Irányítás az Azure-hoz a Microsoft CAF

A felhő üzleti támogató technológiákat kapcsolatos új paradigmákat hoz létre. Ezek új paradigmákat műszakok is okozhatja a hogyan olyan technológiák is elfogadott, felügyelt és szabályozott. Akár egész adatközpontok megsemmisül, és újból végre egy felügyelet nélküli folyamat egy kódsorral is, ha rendelkezünk újragondolja a hagyományos megközelítés. Ez a egyaránt igaz, amikor a cégirányítási.

A szervezet számára a meglévő szabályzatok szabályozzák a helyszíni informatikai környezetekben, felhőalapú cégirányítási ki kell egészítenie a ezek szabályzatok. Azonban a vállalati házirend-integráció a helyszíni és a felhő között a felhőbeli cégirányítási lejárati ideje és a felhőben digitális hagyatéki függően változhat. A felhő hagyatéki haladásával idővel, így lesz a felhő cégirányítási folyamatok és a szabályzatok.

Az útmutató ezen szakasza a CAF két célra szolgál:

* Adja meg, amelyek közös megtapasztalhatják, hogy az ügyfelek milyen gyakran lép fel döntéstámogató ügyfél Journey. Minden egyes magába foglalja az üzleti kockázatok, kockázatcsökkentés és technikai megoldások megvalósításának tervezési útmutatóval kapcsolatos vállalati szabályzatok. Szerint szükségességét a tervezési útmutatást az adott Azure-bA. Ezek az utak egyéb útmutatást sikerült alkalmazni a felhő-agnosztikus vagy többfelhős megközelítés részeként.
* Súgó az olvasók, amely megfelel a üzleti igények, többek között a cégirányítási több nyilvános felhők – részletes útmutató a vállalati házirendek, eljárások, fejlesztését, és az eszközök különféle cégirányítási személyre szabott megoldásokat hozhat létre.

Ez a tartalom a felhőben Cégirányítási csapat számára készült. Emellett akkor is a felhőbeli tervezők, amely a felhőalapú cégirányítási erős alaprendszert fejlesztéshez szükséges.

## <a name="audience"></a>Célközönség

A tartalom a CAF hatással van, az üzleti, a technológia és a vállalatok kulturális környezetét. Ez a szakasza a CAF erősen interakcióba számítástechnikai biztonsági, az informatikai szabályozással, pénzügyi, üzleti vezetők, a hálózat, identitás, a, és a bevezetési csapat a felhő. Nincsenek különböző közös függőségek ezen személyek, ezért ez az útmutató segítségével, a felhőben dolgozó tervezők által a facilitative megközelítést. Ezeket csapatok számára veremigazítás lehet, hogy egy egyszeri erőfeszítés, de bizonyos esetekben ezek más személyeknek ismétlődő interakció okoz.

A Felhőtervező szolgál gondolkodási vezető egyeztető ezek célközönséggel összegyűjthetők. Ebben a gyűjteményben az útmutatók a tartalom célja a Felhőtervező megfelelő beszélgetés, a megfelelő célközönségnek, a meghajtó szükséges döntések elősegítése érdekében. Üzleti átalakítást, amely jogosult a felhő által függ a Felhőtervező szerepkör segítségével az üzleti döntések útmutató és informatikai.

**Ez a szakasz mérnök szakirány felhőalapú:** Minden egyes szakasza a CAF különböző specializáció vagy a Felhőtervező szerepkör variant jelöli. Ez a szakasza a CAF egy szenvedélyét csökkentése vagy technikai kockázatok ürítése a felhőben dolgozó tervezők lett tervezve. Ugyanezek a szakemberek felhőalapú jegyektől, tekintse meg a felhőszolgáltatók számos, hogy inkább felhőalapú őrei, vagy együttesen a felhő Cégirányítási csapat. Az egyes gyakorlatban hasznosítható vásárló által bejárt a cikkek bemutatják, hogyan az összeállítás és a szerepkör a Felhőbeli Cégirányítási csapat idővel változhat.

## <a name="using-this-guide"></a>Ez az útmutató használata

Az olvasók, akik ezt az útmutatót az elejétől a végéig ezt a tartalmat fog támogatási párhuzamosan, egy stabil felhőalapú adatirányítási stratégia kidolgozásában felhőalapú megvalósítás. Az útmutató végigvezeti az olvasót az elmélet, és ez a stratégia megvalósítását.

A tanfolyamot elmélet és gyors elérése az Azure végrehajtása, a használatának első lépései a [gyakorlatban hasznosítható cégirányítási Journey áttekintése](./journeys/overview.md). Ez az útmutató segítségével az olvasó kezdhetik, és fejlesztheti tovább a felhő bevezetését erőfeszítések párhuzamos szabályozási igényeiknek.

## <a name="next-steps"></a>További lépések

Tekintse át a gyakorlatban hasznosítható cégirányítási Journey.

> [!div class="nextstepaction"]
> [Gyakorlatban hasznosítható Cégirányítási Journey](./journeys/overview.md)
