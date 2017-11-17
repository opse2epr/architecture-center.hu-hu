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
# <a name="security-patterns"></a><span data-ttu-id="7ae66-106">Biztonsági minták</span><span class="sxs-lookup"><span data-stu-id="7ae66-106">Security patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="7ae66-107">Biztonsági, de a rendszer rosszindulatú vagy véletlen műveletek kívül a tervezett használat megelőzése érdekében, és a nyilvánosságra képességeinek adatvesztéssel.</span><span class="sxs-lookup"><span data-stu-id="7ae66-107">Security is the capability of a system to prevent malicious or accidental actions outside of the designed usage, and to prevent disclosure or loss of information.</span></span> <span data-ttu-id="7ae66-108">A felhőalapú alkalmazásokhoz ki vannak téve az internetes megbízható helyszíni határain kívül, gyakran nyissa meg a nyilvános, és nem megbízható felhasználók szolgálhat.</span><span class="sxs-lookup"><span data-stu-id="7ae66-108">Cloud applications are exposed on the Internet outside trusted on-premises boundaries, are often open to the public, and may serve untrusted users.</span></span> <span data-ttu-id="7ae66-109">Alkalmazások úgy kell megtervezni és rendszerbe oly módon, hogy védje azokat a rosszindulatú támadások, csak az engedéllyel rendelkező felhasználók korlátozza a hozzáférést és bizalmas adatok védelmét.</span><span class="sxs-lookup"><span data-stu-id="7ae66-109">Applications must be designed and deployed in a way that protects them from malicious attacks, restricts access to only approved users, and protects sensitive data.</span></span>

| <span data-ttu-id="7ae66-110">Minta</span><span class="sxs-lookup"><span data-stu-id="7ae66-110">Pattern</span></span> | <span data-ttu-id="7ae66-111">Összefoglalás</span><span class="sxs-lookup"><span data-stu-id="7ae66-111">Summary</span></span> |
| ------- | ------- |
| [<span data-ttu-id="7ae66-112">Összevont identitás</span><span class="sxs-lookup"><span data-stu-id="7ae66-112">Federated Identity</span></span>](../federated-identity.md) | <span data-ttu-id="7ae66-113">Egy külső identitásszolgáltatótól delegált hitelesítés.</span><span class="sxs-lookup"><span data-stu-id="7ae66-113">Delegate authentication to an external identity provider.</span></span> |
| [<span data-ttu-id="7ae66-114">Forgalomirányító</span><span class="sxs-lookup"><span data-stu-id="7ae66-114">Gatekeeper</span></span>](../gatekeeper.md) | <span data-ttu-id="7ae66-115">Alkalmazások és szolgáltatások védelmét egy közötti ügyfelek és az alkalmazás vagy szolgáltatás, ellenőrzi és sanitizes kérelmek és a közöttük kérelmek és az adatok továbbítja szolgáló dedikált gazdagép példánnyal.</span><span class="sxs-lookup"><span data-stu-id="7ae66-115">Protect applications and services by using a dedicated host instance that acts as a broker between clients and the application or service, validates and sanitizes requests, and passes requests and data between them.</span></span> |
| [<span data-ttu-id="7ae66-116">Kulcs valet</span><span class="sxs-lookup"><span data-stu-id="7ae66-116">Valet Key</span></span>](../valet-key.md) | <span data-ttu-id="7ae66-117">A token vagy a kulcs, amely egy meghatározott erőforrás vagy a szolgáltatás korlátozott közvetlen hozzáférést biztosít az ügyfelek használja.</span><span class="sxs-lookup"><span data-stu-id="7ae66-117">Use a token or key that provides clients with restricted direct access to a specific resource or service.</span></span> |