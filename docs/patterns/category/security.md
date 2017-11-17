---
title: "Biztonsági minták"
description: "Biztonsági, de a rendszer rosszindulatú vagy véletlen műveletek kívül a tervezett használat megelőzése érdekében, és a nyilvánosságra képességeinek adatvesztéssel. A felhőalapú alkalmazásokhoz ki vannak téve az internetes megbízható helyszíni határain kívül, gyakran nyissa meg a nyilvános, és nem megbízható felhasználók szolgálhat. Alkalmazások úgy kell megtervezni és rendszerbe oly módon, hogy védje azokat a rosszindulatú támadások, csak az engedéllyel rendelkező felhasználók korlátozza a hozzáférést és bizalmas adatok védelmét."
keywords: "Kialakítási mintája"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: 266b5c4283d82a107783fc7a746f065be9027b51
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="security-patterns"></a>Biztonsági minták

[!INCLUDE [header](../../_includes/header.md)]

Biztonsági, de a rendszer rosszindulatú vagy véletlen műveletek kívül a tervezett használat megelőzése érdekében, és a nyilvánosságra képességeinek adatvesztéssel. A felhőalapú alkalmazásokhoz ki vannak téve az internetes megbízható helyszíni határain kívül, gyakran nyissa meg a nyilvános, és nem megbízható felhasználók szolgálhat. Alkalmazások úgy kell megtervezni és rendszerbe oly módon, hogy védje azokat a rosszindulatú támadások, csak az engedéllyel rendelkező felhasználók korlátozza a hozzáférést és bizalmas adatok védelmét.

| Minta | Összefoglalás |
| ------- | ------- |
| [Összevont identitás](../federated-identity.md) | Egy külső identitásszolgáltatótól delegált hitelesítés. |
| [Forgalomirányító](../gatekeeper.md) | Alkalmazások és szolgáltatások védelmét egy közötti ügyfelek és az alkalmazás vagy szolgáltatás, ellenőrzi és sanitizes kérelmek és a közöttük kérelmek és az adatok továbbítja szolgáló dedikált gazdagép példánnyal. |
| [Kulcs valet](../valet-key.md) | A token vagy a kulcs, amely egy meghatározott erőforrás vagy a szolgáltatás korlátozott közvetlen hozzáférést biztosít az ügyfelek használja. |