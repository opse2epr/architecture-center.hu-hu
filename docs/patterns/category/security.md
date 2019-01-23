---
title: Biztonsági minták
titleSuffix: Cloud Design Patterns
description: A biztonságosság egy rendszer azon képessége, hogy meg tudja-e akadályozni a tervezett felhasználáson kívüli, rosszindulatú vagy véletlen műveleteket, és meg tudja-e előzni az információk nyilvánosságra kerülését vagy elvesztését. A felhőalapú alkalmazások elérhetők az interneten a megbízható helyszíni területek határain kívül is, gyakran nyilvánosak, és nem megbízható felhasználókat is kiszolgálhatnak. Az alkalmazásokat úgy kell megtervezni és üzembe helyezni, hogy védve legyenek a rosszindulatú támadások ellen, csak a jogosult felhasználók férhessenek hozzájuk, és védjék a bizalmas adatokat.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: c18255dccdbd8659705884a4141f5ecd099d9ece
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54482085"
---
# <a name="security-patterns"></a><span data-ttu-id="03c6a-106">Biztonsági minták</span><span class="sxs-lookup"><span data-stu-id="03c6a-106">Security patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="03c6a-107">A biztonságosság egy rendszer azon képessége, hogy meg tudja-e akadályozni a tervezett felhasználáson kívüli, rosszindulatú vagy véletlen műveleteket, és meg tudja-e előzni az információk nyilvánosságra kerülését vagy elvesztését.</span><span class="sxs-lookup"><span data-stu-id="03c6a-107">Security is the capability of a system to prevent malicious or accidental actions outside of the designed usage, and to prevent disclosure or loss of information.</span></span> <span data-ttu-id="03c6a-108">A felhőalapú alkalmazások elérhetők az interneten a megbízható helyszíni területek határain kívül is, gyakran nyilvánosak, és nem megbízható felhasználókat is kiszolgálhatnak.</span><span class="sxs-lookup"><span data-stu-id="03c6a-108">Cloud applications are exposed on the Internet outside trusted on-premises boundaries, are often open to the public, and may serve untrusted users.</span></span> <span data-ttu-id="03c6a-109">Az alkalmazásokat úgy kell megtervezni és üzembe helyezni, hogy védve legyenek a rosszindulatú támadások ellen, csak a jogosult felhasználók férhessenek hozzájuk, és védjék a bizalmas adatokat.</span><span class="sxs-lookup"><span data-stu-id="03c6a-109">Applications must be designed and deployed in a way that protects them from malicious attacks, restricts access to only approved users, and protects sensitive data.</span></span>

|                    <span data-ttu-id="03c6a-110">Pattern</span><span class="sxs-lookup"><span data-stu-id="03c6a-110">Pattern</span></span>                     |                                                                                                         <span data-ttu-id="03c6a-111">Összefoglalás</span><span class="sxs-lookup"><span data-stu-id="03c6a-111">Summary</span></span>                                                                                                         |
|------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [<span data-ttu-id="03c6a-112">Federated Identity</span><span class="sxs-lookup"><span data-stu-id="03c6a-112">Federated Identity</span></span>](../federated-identity.md) |                                                                                <span data-ttu-id="03c6a-113">A hitelesítést delegálhatja egy külső identitásszolgáltatónak.</span><span class="sxs-lookup"><span data-stu-id="03c6a-113">Delegate authentication to an external identity provider.</span></span>                                                                                |
|         [<span data-ttu-id="03c6a-114">Gatekeeper</span><span class="sxs-lookup"><span data-stu-id="03c6a-114">Gatekeeper</span></span>](../gatekeeper.md)         | <span data-ttu-id="03c6a-115">Védheti az alkalmazásokat és szolgáltatásokat egy dedikált üzemeltető példány segítségével, amely közvetítőként szolgál az ügyfelek és az alkalmazás vagy szolgáltatás között, érvényesíti és vírusmentesíti a kéréseket, valamint közvetíti a kéréseket és az adatokat közöttük.</span><span class="sxs-lookup"><span data-stu-id="03c6a-115">Protect applications and services by using a dedicated host instance that acts as a broker between clients and the application or service, validates and sanitizes requests, and passes requests and data between them.</span></span> |
|          [<span data-ttu-id="03c6a-116">Valet Key</span><span class="sxs-lookup"><span data-stu-id="03c6a-116">Valet Key</span></span>](../valet-key.md)          |                                                        <span data-ttu-id="03c6a-117">Jogkivonatot vagy kulcsot használhat, amely korlátozott közvetlen hozzáférést biztosít az ügyfelek számára egy adott erőforráshoz vagy szolgáltatáshoz.</span><span class="sxs-lookup"><span data-stu-id="03c6a-117">Use a token or key that provides clients with restricted direct access to a specific resource or service.</span></span>                                                        |
