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
# <a name="security-patterns"></a>Biztonsági minták

[!INCLUDE [header](../../_includes/header.md)]

A biztonságosság egy rendszer azon képessége, hogy meg tudja-e akadályozni a tervezett felhasználáson kívüli, rosszindulatú vagy véletlen műveleteket, és meg tudja-e előzni az információk nyilvánosságra kerülését vagy elvesztését. A felhőalapú alkalmazások elérhetők az interneten a megbízható helyszíni területek határain kívül is, gyakran nyilvánosak, és nem megbízható felhasználókat is kiszolgálhatnak. Az alkalmazásokat úgy kell megtervezni és üzembe helyezni, hogy védve legyenek a rosszindulatú támadások ellen, csak a jogosult felhasználók férhessenek hozzájuk, és védjék a bizalmas adatokat.

|                    Pattern                     |                                                                                                         Összefoglalás                                                                                                         |
|------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [Federated Identity](../federated-identity.md) |                                                                                A hitelesítést delegálhatja egy külső identitásszolgáltatónak.                                                                                |
|         [Gatekeeper](../gatekeeper.md)         | Védheti az alkalmazásokat és szolgáltatásokat egy dedikált üzemeltető példány segítségével, amely közvetítőként szolgál az ügyfelek és az alkalmazás vagy szolgáltatás között, érvényesíti és vírusmentesíti a kéréseket, valamint közvetíti a kéréseket és az adatokat közöttük. |
|          [Valet Key](../valet-key.md)          |                                                        Jogkivonatot vagy kulcsot használhat, amely korlátozott közvetlen hozzáférést biztosít az ügyfelek számára egy adott erőforráshoz vagy szolgáltatáshoz.                                                        |
