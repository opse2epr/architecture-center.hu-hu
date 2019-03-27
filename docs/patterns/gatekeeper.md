---
title: Forgalomirányító minta
titleSuffix: Cloud Design Patterns
description: Védheti az alkalmazásokat és szolgáltatásokat egy dedikált üzemeltető példány segítségével, amely közvetítőként szolgál az ügyfelek és az alkalmazás vagy szolgáltatás között, érvényesíti és vírusmentesíti a kéréseket, valamint közvetíti a kéréseket és az adatokat közöttük.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 99d3f1f3e4ea1f189fbdfc4cb9d9b3d18e8551a2
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298578"
---
# <a name="gatekeeper-pattern"></a>Forgalomirányító minta

[!INCLUDE [header](../_includes/header.md)]

Védheti az alkalmazásokat és szolgáltatásokat egy dedikált üzemeltető példány segítségével, amely közvetítőként szolgál az ügyfelek és az alkalmazás vagy szolgáltatás között, érvényesíti és vírusmentesíti a kéréseket, valamint közvetíti a kéréseket és az adatokat közöttük. Ez egy további biztonsági réteget biztosíthat, és csökkenti a számítógép támadható felületét.

## <a name="context-and-problem"></a>Kontextus és probléma

Az alkalmazások kérések fogadásával és feldolgozásával teszik elérhetővé a funkcióikat az ügyfelek számára. Felhőben futtatott forgatókönyvek esetén az alkalmazások végpontokat tesznek elérhetővé, amelyekhez az ügyfelek csatlakoznak, és általában tartalmazzák az ügyfelektől érkező kérelmek kezeléséhez szükséges kódot. Ez a kód elvégzi a hitelesítést és az ellenőrzést, a kérelmek feldolgozását vagy annak egy részét, és általában hozzáfér tárolókhoz és egyéb szolgáltatásokhoz az ügyfél nevében.

Ha egy rosszindulatú felhasználó feltöri a rendszert, és hozzáférést szerez az alkalmazás üzemeltetési környezetéhez, elérhetővé válnak számára az alkalmazás által használt biztonsági mechanizmusok, például a hitelesítő adatok és a tárkulcsok, valamint az alkalmazás által elért szolgáltatások és adatok. Ezáltal a rosszindulatú felhasználó korlátlan hozzáférést szerezhet a bizalmas adatokhoz és más szolgáltatásokhoz.

## <a name="solution"></a>Megoldás

Ha minimálisra szeretné csökkenteni annak az esélyét, hogy ügyfelek hozzáférjenek bizalmas adatokhoz és szolgáltatásokhoz, válassza le a nyilvános végpontokat elérhetővé tévő gazdagépeket vagy feladatokat arról a kódról, amely a kérelmek feldolgozását végzi, illetve hozzáfér a tárolókhoz. Ezt egy előtérrendszer vagy dedikált feladat alkalmazásával teheti meg, amely közvetlenül érintkezik az ügyfelekkel, és &mdash;például egy leválasztott felületen keresztül&mdash; továbbítja a kéréseket az azokat kezelő gazdagépek vagy feladatok számára. Az ábra általános áttekintést nyújt erről a mintáról.

![Általános áttekintés a mintáról](./_images/gatekeeper-diagram.png)

A Forgalomirányító minta használható egyszerűen a tároló védelmére, vagy átfogó előtérrendszerként az alkalmazás összes funkciójának védelmére. Néhány fontosabb tényező:

- **Szabályozott ellenőrzés**. A forgalomirányító ellenőrzi az összes kérést, és elutasítja azokat, amelyek nem felelnek meg az ellenőrzési követelményeknek.
- **Korlátozott kockázat és kitettség**. A forgalomirányítónak nincs hozzáférése a megbízható gazdagép által használt hitelesítő adatokhoz vagy kulcsokhoz a tároló és a szolgáltatások eléréséhez. Ha feltörik a forgalomirányítót, a támadó nem fér hozzá ezekhez a hitelesítő adatokhoz vagy kulcsokhoz.
- **A megfelelő biztonsági**. A forgalomirányító korlátozott jogosultságú módban fut, az alkalmazás többi része viszont teljesen megbízható módban fut, hogy hozzáférhessen a tárolóhoz és a szolgáltatásokhoz. Ha a forgalomirányító biztonsága sérül, nem tud közvetlenül hozzáférni az alkalmazás szolgáltatásaihoz vagy adataihoz.

Egy tipikus felépítésű hálózatban ez a minta gyakorlatilag tűzfalként funkcionál. Lehetővé teszi a forgalomirányító számára, hogy a kérések megvizsgálása után eldöntse, továbbítható-e a kérés a megbízható gazdagépnek (más néven kulcskezelő főkiszolgálónak), amely aztán végrehajtja a szükséges feladatot. Ehhez a forgalomirányító először ellenőrzi és vírusmentesíti a kéréseket, majd továbbítja őket a megbízható gazdagépnek.

## <a name="issues-and-considerations"></a>Problémák és megfontolandó szempontok

A minta megvalósítása során az alábbi pontokat vegye figyelembe:

- Ügyeljen rá, hogy a megbízható gazdagépek, amelyek számára a forgalomirányító kéréseket továbbít, kizárólag belső és védett végpontokat tesznek elérhetővé, és kizárólag a forgalomirányítóhoz kapcsolódnak. Fontos, hogy a megbízható gazdagépek ne tegyenek elérhetővé külső végpontokat vagy csatolókat.
- A forgalomirányítónak korlátozott jogosultságú módban kell futnia. Ez általában úgy oldható meg, hogy a forgalomirányító és a megbízható gazdagép külön-külön üzemeltetett szolgáltatásban vagy virtuális gépen fut.
- A forgalomirányító ne végezzen az alkalmazással vagy a szolgáltatásokkal kapcsolatos feldolgozási feladatot, illetve ne rendelkezzen hozzáféréssel semmilyen adathoz. A forgalomirányító feladata kizárólag a kérések ellenőrzése és vírusmentesítése. A megbízható gazdagépeknek néha további ellenőrzéseket kell végeznie a kéréseken, de az alapvető ellenőrzési feladatokat a forgalomirányító látja el.
- Amikor csak lehetséges, használjon biztonságos kommunikációs csatornát (HTTPS, SSL vagy TLS) a forgalomirányító és a megbízható gazdagép, illetve a feladatok között. Egyes üzemeltetési környezetek nem támogatják a HTTPS használatát a belső végpontokon.
- A Forgalomirányító minta implementálásához szükséges extra réteg az alkalmazáshoz való hozzáadása valószínűleg hatással van a teljesítményre a szükséges további feldolgozás és hálózati kommunikáció miatt.
- A forgalomirányító-példány kritikus meghibásodási pont lehet a rendszeren belül. Az esetleges hibák hatásának minimálisra csökkentése érdekében fontolja meg további példányok telepítését és egy automatikus skálázási rendszer használatát a megfelelő kapacitás és a folyamatos rendelkezésre állás biztosítása érdekében.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes ezt a mintát használni?

Ez a minta az alábbi esetekben hasznos:

- Olyan alkalmazásokhoz, amelyek bizalmas információkat kezelnek, és szolgáltatásaik jelentős védelmet igényelnek a rosszindulatú támadások ellen, illetve az üzletmenet szempontjából kritikus fontosságú műveleteket hajtanak végre, amelyek nem szakadhatnak meg.
- Olyan elosztott alkalmazásokhoz, ahol a fő feladatoktól elkülönítve kell ellenőrizni a kéréseket, illetve az ellenőrzés központi elvégzéséhez a karbantartás és az adminisztráció leegyszerűsítése érdekében.

## <a name="example"></a>Példa

Felhőben futtatott forgatókönyvek esetén ez a minta úgy implementálható, hogy az alkalmazás megbízható szerepköreiről és szolgáltatásairól leválasztják a forgalomirányító szerepkört vagy virtuális gépet. Ehhez köztes kommunikációs mechanizmusként egy belső végpontot, üzenetsort vagy tárolót kell használni. Az ábrán egy belső végpont használata látható.

![Példa arra, amikor a minta a Cloud Services webes és feldolgozói szerepköreit használja](./_images/gatekeeper-endpoint.png)

## <a name="related-patterns"></a>Kapcsolódó minták

A [Pótkulcs minta](./valet-key.md) is releváns lehet a forgalomirányító minta implementálásakor. A forgalomirányító és a megbízható szerepkörök közötti kommunikáció esetén érdemes olyan kulcsok vagy tokenek használatával növelni a biztonságot, amelyek korlátozzák az erőforrásokhoz való hozzáférésre vonatkozó engedélyeket. Leírja, hogyan használhatóak a tokenek vagy kulcsok, amelyek korlátozott közvetlen hozzáférést biztosítanak az ügyfelek számára egy adott erőforráshoz vagy szolgáltatáshoz.
