---
title: 'CAF: Végrehajtható szabályozási folyamatok'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Végrehajtható szabályozási folyamatok
author: BrianBlanchard
layout: LandingPage
ms.topic: landing-page
ms.openlocfilehash: 0e3b1b6211ebb4faaa1aac3c13887a4a35ed3c63
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298622"
---
# <a name="actionable-governance-journeys"></a>Végrehajtható szabályozási folyamatok

A cégirányítási Journey ebben a szakaszban a CAF cégirányítási modell a növekményes megközelítést mutatnak be. Létrehozhat egy Agilis cégirányítási platform, amely a várható fejlődési útját bármely felhőbeli cégirányítási forgatókönyv igényeinek.

## <a name="review-and-adopt-cloud-governance-best-practices"></a>Tekintse át, és elfogadja a felhő cégirányítási ajánlott eljárások

Indítása le egy bevezetési elérési útját, válassza ki a következő utak egyike. Minden egyes út egy sorozat, ajánlott eljárások alapján egy képzeletbeli felhasználói élményt ismerteti. A növekményes megközelítés a CAF cégirányítási modell új olvasók javasolt, hogy tekintse át a magas szintű cégirányítási elmélet bemutatása, az alábbi vagy ajánlott elfogadása előtt.

<!-- markdownlint-disable MD033 -->

<ul class="panelContent cardsZ">
<li style="display: flex; flex-direction: column;">
    <a href="./small-to-medium-enterprise/overview.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Small-to-Medium Enterprise</h3>
                        <p>Cégirányítási az út az olyan vállalatok, amelyek kevesebb mint öt adatközpontok tulajdonosa, és kezelheti a költségeket a egy központi informatikai vagy költséghelyi visszacsatolási modell.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./large-enterprise/overview.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Nagyvállalati</h3>
                        <p>Cégirányítási út vállalatok számára saját ötnél több adatközpontban és a költségek kezelése több üzleti egység vonatkozásában.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

<!-- markdownlint-enable MD033 -->

## <a name="an-incremental-approach-to-cloud-governance"></a>A cégirányítási felhőalapú növekményes megközelítés

A felhő bevezetésének egy út nem egy cél. A vizualizáción nincsenek egyértelmű mérföldkő és üzleti kézzelfogható előnyök. Azonban végső felhőre való áttérés állapota ismeretlen általában egy vállalat megkezdésekor az utazás. Felhőalapú cégirányítási hoz létre, hogy a vállalat az egész az út egy biztonságos elérési úton guardrails.

Ezek az irányítási utak fiktív cég, a valódi ügyfelek Journey alapján tapasztalatok ismertetik. Minden egyes út követi az ügyfél a felhőre való áttérés a cégirányítási aspektusait.

### <a name="establishing-an-end-state"></a>Egy befejezési állapotot létrehozásáról

A célhely nélkül az út csak wandering van. Fontos a nyers látás, a befejezési állapotának megállapítása az első lépés előtt. A következő szemléltető ábra biztosít egy hivatkozási befejezési állapotát. Nem a kiindulási pont, de azt mutatja, hogy a potenciális célhelyéhez.

![A CAF cégirányítási modell szemléltető ábra](../../_images/operational-transformation-govern-highres.png)

A CAF cégirányítási modell egyik fontos az utazás közben azonosítja. Minden terület kockázatok további felhőszolgáltatások fogad el kell inspiráló a vállalat különböző típusú vonatkozik. Ennek keretében a cégirányítási utazás azonosítja a szükséges műveleteket a felhő Cégirányítási csapat számára. Úton elhelyezkedő minden egyes alapelv CAF cégirányítási modell leírása további. Széles körben ezek a következők:

**Vállalati szabályzatok**. Felhőalapú vállalati szabályzatok meghatározhatják. A cégirányítási utazás összpontosít a vállalati házirend különféle területeivel kapcsolatban:

- Üzleti kockázatai: Azonosító és a vállalati kockázatok ismertetése.
- Házirend és megfelelőség: Átalakítás kockázatokat, házirend-utasítások, amelyek támogatják a megfelelőségi követelményeket.
- Folyamatok: Annak biztosítása, hogy a megadott szabályzatok betartását.

**Felhőalapú irányítási öt oktatnak**. Ezek a szabályok támogatja a vállalati szabályzatoknak. Minden egyes fegyelem a vállalat megvédi az esetleges buktatókat:

- Cost Management
- Biztonsági alapkonfiguráció
- Erőforrás-konzisztencia
- Identitáskezelési alapkonfiguráció
- Üzembe helyezés gyorsítása

Vállalati szabályzatok alapvetően a észleli a potenciális problémák korai figyelmeztető rendszert szolgál. Minderről a vállalat kockázatok csökkentése, és hozzon létre guardrails segítségével.

### <a name="grow-to-the-end-state"></a>Hogy a befejezési állapota

Mivel a cégirányítási követelmények a cloud bevezetési folyamata során is fejlődik, cégirányítási megközelítés szükség. Vállalatok már nem várja meg, egy kis csapata guardrails és tervekkel létrehozásához, minden highway *első lépése előtt*. Üzleti eredmények várhatóan több gyorsan és problémamentesen. Az informatikai szabályozással kell is gyorsan és tartja a ritmust üzleti követelményeknek a felhőre való áttérés során legyen naprakész, és elkerülheti az "informatikai RÉSZLEG tudta."

Egy **növekményes cégirányítási** megközelítés lehetővé teszi e tulajdonságok. Növekményes cégirányítási vállalati házirendek, folyamatok és eszközök elfogadása és cégirányítási alapjait létesíteni egy kis készletét támaszkodik. Hogy alapítvány neve egy **termék minimális működőképes (MVP)**. Az MVP lehetővé teszi a cégirányítási csapat gyorsan cégirányítási beépítse megvalósításokhoz bevezetési életciklusa során. Az MVP a cloud bevezetési folyamat során bármikor hozható létre. Azonban célszerű a lehető leghamarabb elfogadják az MVP.

Gyorsan reagáljanak a kockázatok változó teszi lehetővé teszi, hogy új módokon léphet kapcsolatba a felhő Cégirányítási csapat. A felhő Cégirányítási csapat a Felhőstratégia csapata is csatlakoztatni, mert a scouts, áthelyezése előtt a felhő bevezetésének csapatok ábrázolási útvonalakat, és gyorsan létrehozó guardrails a bevezetési terveket társított kockázatok csökkentése érdekében. Ezek just-in-time cégirányítási rétegek nevezzük **cégirányítási fejlesztések**. Ezzel a módszerrel adatirányítási stratégia haladásával egy lépést előre a felhő bevezetésének csapatok.

Az alábbi ábrán látható egy egyszerű cégirányítási MVP és három cégirányítási fejlesztések. A fejlesztések során további vállalati házirendeket új kockázatok csökkentése érdekében. A központi telepítési gyorsítás fegyelem majd alkalmazza ezeket a módosításokat minden telepítés között.

![Növekményes Cégirányítási fejlesztések – példa](../../_images/governance/incremental-governance-example.png)

> [!NOTE]
> Cégirányítási nem helyettesíti a főbb funkciók elvégzésére – például a biztonság, hálózatkezelés, identitás, pénzügyi, fejlesztési és üzemeltetési vagy operations. A vizualizáción lesz folytatott interakciót és a függőségek minden függvény tagok. Ezekhez a tagokhoz fel kell venni a felhő Cégirányítási csapat a gyorsabb döntéseket hozhat, és műveleteket.

## <a name="choosing-a-governance-journey"></a>Irányítás az út kiválasztása

Az utak bemutatják, hogyan lehet egy cégirányítási MVP megvalósításához. Itt minden egyes út bemutatja, hogyan a felhő Cégirányítási csapata együttműködhet a felhő bevezetésének csapatok előre bevezetési erőfeszítések felgyorsítása partnerként. A CAF cégirányítási modell irányítási az alkalmazás végigvezeti a későbbi fejlesztések révén foundation rendszerből.

Cégirányítási áttérés első lépésként válassza ki az alábbi két lehetőség közül. A beállítások szintetizált ügyfélélményt alapulnak. A címek navigáció megkönnyítése érdekében a vállalat méretétől alapulnak. Az olvasó döntési azonban összetettebb művelet lehet. Az alábbi táblázatok a két lehetőség különbségeit tagolni.

> [!NOTE]
> Nem valószínű, hogy mindkét utazás teljesen az adott helyzethez igazítja. Válassza ki, amelyik utazás legközelebbi, és kiindulási pontként használhatja. A folyamatok teljes további információ segítségével testre szabhatja a döntéseket hozhat, az adott feltételnek.

### <a name="business-characteristics"></a>Üzleti jellemzői

|                                            | Small-to-medium enterprise                                                                              | Nagyvállalati                                                                                               |
|--------------------------------------------|---------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------|
| Földrajzi hely (ország vagy az egyes geopolitikai régiókban) | Az ügyfelek vagy a személyzet található nagymértékben egy földrajzi hely                                                      | Az ügyfelek vagy a személyzet található különböző földrajzi régiók                                                              |
| Érintett részlegeket                    | Egyetlen üzleti egység                                                                                    | Több üzleti egység                                                                                        |
| Informatikai költségvetést                                  | Egyetlen informatikai költségvetést                                                                                        | A költségkeret különböző üzleti egységek                                                                         |
| IT-beruházásokkal                             | Beruházna (CapEx) – adatvezérelt beruházásokkal éves bevezetését tervezzük, és csak alapszintű karbantartási általában foglalkozik. | CapEx adatvezérelt beruházásokkal éves bevezetését tervezzük, és többek között általában a karbantartás és a egy frissítési ciklus 3-5 évig. |

### <a name="current-state-before-adopting-cloud-governance"></a>Felhőalapú cégirányítási bevezetése előtti jelenlegi állapota

|                                             | Small-to-medium enterprise                                                                               | Nagyvállalati                                                                                                          |
|---------------------------------------------|----------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------|
| Adatközpont- vagy harmadikfél-alapú szolgáltatók | Kevesebb mint öt adatközpontok                                                                                  | Több mint öt adatközpontok                                                                                                   |
| Hálózat                                  | Nincs WAN vagy 1 &ndash; 2 WAN-szolgáltatók                                                                             | Összetett hálózati vagy globális WAN                                                                                             |
| Identitás                                    | Egyetlen erdő, egyetlen tartományhoz. Nem követelmény, jogcímalapú hitelesítésre vagy külső MFA eszközök. | Komplex, több erdő, több tartomány. Az alkalmazásoknak jogcímalapú hitelesítésre vagy külső MFA eszközök. |

### <a name="desired-future-state-after-evolving-cloud-governance"></a>Felhőalapú cégirányítási fejlődő után kívánt jövőbeli állapot

|                                              | Small-to-medium enterprise                                                                        | Nagyvállalati                                                                                        |
|----------------------------------------------|---------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------|
| Cost Management – a felhő számlázási           | Költséghelyi visszacsatolási modell. A számlázás keresztül van központi informatikai.                                                | A jóváírási modell. A számlázás sikerült terjeszthetők IT-beszerzési keresztül.                                  |
| Biztonsági alapkonfiguráció – a védett adatok           | Vállalati pénzügyi adatokat és IP. Korlátozott a vásárlói adatokat. Nincsenek külső megfelelőségi követelmények.     | Több gyűjtemény ügyfelei, pénzügyi és PII adatokat. Fontolja meg a külső megfelelőségi szükségessé. |
| Erőforrás konzisztencia – alapvető fontosságú alkalmazásokhoz | Kimaradások nehéznek, de nem felelősséggel káros. Meglévő IT-üzemeltetők olyan viszonylag éretlen. | Kimaradások definiált és figyelemmel kísérni a pénzügyi hatással van. IT-üzemeltetési hozták létre, és részletes.         |

E két Journey ügyfelek esetében, akik felhőalapú cégirányítási beruházni élmény két szélsőséges képviseli. A legtöbb vállalat jelenik meg a fenti két esetben kombinációját. Miután megtekintette a utazás leírásában, a CAF cégirányítási modellt használja a cégirányítási beszélgetés indítása, és módosítsa a referenciakonfiguráció Journey jobban az igényeinek.

## <a name="next-steps"></a>További lépések

Ezek az utak közül választhat:

> [!div class="nextstepaction"]
> [Kis és közepes méretű vállalat cégirányítási utazás](./small-to-medium-enterprise/overview.md)
>
> [Nagyvállalati cégirányítási utazás](./large-enterprise/overview.md)
