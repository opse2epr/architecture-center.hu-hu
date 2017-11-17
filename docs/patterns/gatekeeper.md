---
title: "Forgalomirányító"
description: "Alkalmazások és szolgáltatások védelmét egy közötti ügyfelek és az alkalmazás vagy szolgáltatás, ellenőrzi és sanitizes kérelmek és a közöttük kérelmek és az adatok továbbítja szolgáló dedikált gazdagép példánnyal."
keywords: "Kialakítási mintája"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories: security
ms.openlocfilehash: 39f8548bbccb5e19d433f65b2e7e09147d676996
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="gatekeeper-pattern"></a>Forgalomirányító minta

[!INCLUDE [header](../_includes/header.md)]

Alkalmazások és szolgáltatások védelmét egy közötti ügyfelek és az alkalmazás vagy szolgáltatás, ellenőrzi és sanitizes kérelmek és a közöttük kérelmek és az adatok továbbítja szolgáló dedikált gazdagép példánnyal. Ez egy további biztonsági réteget nyújt, és a rendszer támadási felületét.

## <a name="context-and-problem"></a>A környezetben, és probléma

Alkalmazások fogadása és feldolgozása kérelmek által elérhetővé tenni azok funkcióiról az ügyfelek számára. A felhőben üzemeltetett esetben alkalmazások teszi közzé a végpontok ügyfelek kapcsolódni, és általában tartalmaznak az ügyfelektől érkező kérelmek kezeléséhez a kódot. Ez a kód hajtja végre a hitelesítési és érvényesítési, néhány vagy az összes kérelem feldolgozását, és valószínűleg hozzáférések tárolási és egyéb szolgáltatások, az ügyfél nevében.

Ha egy rosszindulatú felhasználó képes okoz a rendszerben, valamint az alkalmazás eléréséhez az üzemeltetési környezetbe, a biztonsági mechanizmust például a hitelesítő adatokat használ, és tárolási kulcsokat, és a szolgáltatások és fér hozzá, adatok ki vannak téve. Ennek eredményeképpen a rosszindulatú felhasználók is hozzáférhetnek segédülésen bizalmas adatokat és más szolgáltatások védelmét.

## <a name="solution"></a>Megoldás

Ügyfelek hozzáférjenek a bizalmas adatokat és a szolgáltatások kockázatának minimalizálása érdekében használata leválasztja az állomások vagy a kódot, amely feldolgozza a kérelmeket, és a tárolási fér hozzá a nyilvános végpontok feladatait. Egy homlokzati vagy egy dedikált feladatot, amely kommunikál az ügyfelek és a kérelem majd átadja használatával érhető el ez&mdash;akár leválasztott felületen keresztül&mdash;az állomások vagy a feladatokat, amelyek fogja kezelni a kérést. Az ábra magas szintű áttekintést nyújt az ebben a mintában.

![Ez a kialakítás áttekintése](./_images/gatekeeper-diagram.png)


Forgalomirányító minta segítségével egyszerűen védelme a tárolási, vagy lehet használható legyen egy átfogóbb homlokzati védelméhez összes az alkalmazás funkcióit. A fontos tényezők, melyeknek a következők:

- **A vezérelt érvényesítése.** A forgalomirányító ellenőrzi az összes kérelmet, és azokat, amelyek nem felelnek meg az érvényesítési követelményének elutasítja.
- **Korlátozott kockázat és kapta.** A forgalomirányító nem fér hozzá a hitelesítő adatokat vagy a tároló és a szolgáltatások eléréséhez a megbízható gazdagép által használt kulcsok. Ha a forgalomirányító biztonsága sérül, a támadó nem eléréséhez a hitelesítő adatokat vagy kulcsok.
- **Megfelelő biztonsági.** A forgalomirányító módban fut. a korlátozott jogosultságú, amíg a többi alkalmazáshoz szükséges a tároló elérése és szolgáltatások teljes megbízhatósági módban fut. Ha a forgalomirányító sérült, nem közvetlenül fér hozzá az alkalmazás szolgáltatások vagy adat.

Ezt a mintát úgy viselkedik, mint a szokásos hálózati topográfia tűzfal. Lehetővé teszi a forgalomirányítóhoz vizsgálja meg a kéréseket, és a döntést arról, hogy a kérelem továbbítása a megbízható gazdagép (más néven a keymaster), amely elvégzi a szükséges feladatok-e. Ehhez a döntéshez általában a forgalomirányító érvényesítéséhez és a kérés tartalma sanitize előtt a megbízható gazdagép be van szükség.

## <a name="issues-and-considerations"></a>Problémákat és szempontok

Ebben a mintában megvalósításához meghatározásakor, vegye figyelembe a következő szempontokat:

- Győződjön meg arról, hogy a megbízható gazdagépek a forgalomirányító továbbítja kérelmek csak belső vagy védett végpontot, és csak a forgalomirányító csatlakozni. A megbízható gazdagépek nem teszi közzé, bármely külső végpontok száma vagy felületek.
- A forgalomirányító korlátozott jogosultságú módban kell futnia. Általában ez azt jelenti, futtatja a forgalomirányító és a megbízható gazdagép üzemeltetett szolgáltatások külön vagy virtuális gépek.
- A gatekeeper ne hajtsa végre a bármely alkalmazás vagy szolgáltatás kapcsolódó feldolgozás, vagy adatokhoz hozzáférni. A függvény csak sanitize kéréseket, és ellenőrizheti. Módosítania kell a megbízható gazdagépek kérelmek további ellenőrzés végrehajtása, de az alapvető érvényesítési a forgalomirányító kell elvégezni.
- Használjon biztonságos kommunikációs csatornát (HTTPS, SSL vagy TLS) között a forgalomirányító és a megbízható gazdagépek vagy feladatok Ha ez lehetséges. Azonban az egyes üzemeltetési környezetekben nem támogatják az HTTPS belső végpontokon.
- A további réteget ad hozzá az alkalmazás a forgalomirányító mintát végrehajtásához valószínű, hogy néhány hatással teljesítmény miatt a további feldolgozás és a hálózati kommunikációra van szükség.
- A forgalomirányító példány lehet a hibaérzékeny pontok kialakulását. Hiba a hatás minimalizálása érdekében fontolja meg a további példányok telepítése és az automatikus skálázás mechanizmus segítségével kapacitás ahhoz rendelkezésre állásának biztosításához.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes használni ezt a mintát

Ebben a mintában a következőkhöz hasznos:

- Alkalmazások, amelyek kezelik a bizalmas adatokat, hogy szolgáltatások kell nagyfokú védelmét a rosszindulatú támadások ellen, vagy kritikus fontosságú műveleteket, amelyek nem szakítható tenni.
- Az elosztott alkalmazásnál, ahol szükséges kérelem ellenőrzése a fő feladatok külön-külön végezze el, vagy központosíthatja az ellenőrzés karbantartási és a felügyelet egyszerűbbé tétele érdekében.

## <a name="example"></a>Példa

A felhőben üzemeltetett forgatókönyvek ebben a mintában a forgalomirányító szerepkör vagy a megbízható szerepköröket és szolgáltatásokat az alkalmazásban a virtuális gép leválasztásával valósítható meg. Ezt úgy teheti meg egy belső végpont, várólista vagy tárolási egy köztes kommunikációs mechanizmus. Az ábra azt mutatja be, belső végpont használatával.

![Példa a minta Felhőszolgáltatások webes és feldolgozói szerepkörök használatával](./_images/gatekeeper-endpoint.png)


## <a name="related-patterns"></a>Kapcsolódó minták

A [Valet kulcs mintát](valet-key.md) lehet is megfelelő Forgalomirányító minta végrehajtása során. Ha kommunikál a forgalomirányító és a megbízható szerepkörök közötti tanácsos a megfelelő biztonság érdekében a kulcsok vagy korlátozni az erőforrások eléréséhez szükséges engedélyekkel jogkivonatok használatával. A token vagy a kulcs, amely egy meghatározott erőforrás vagy a szolgáltatás korlátozott közvetlen hozzáférést biztosít az ügyfelek használatát ismerteti.
