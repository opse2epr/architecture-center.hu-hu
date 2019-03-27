---
title: 'CAF: A Cost Management eszközök az Azure-ban'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: A Cost Management eszközök az Azure-ban
author: BrianBlanchard
ms.openlocfilehash: 58dfa604863f704fd9b9fbb8d0693447cecdaf84
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299292"
---
# <a name="cost-management-tools-in-azure"></a>A Cost Management eszközök az Azure-ban

[Cost Management](overview.md) egyike a [öt oktatnak, felhőalapú Cégirányítási](../governance-disciplines.md). Ez fegyelem módját a felhő költségeit, tervek, felhőalapú költségvetéseket, figyelés lefoglalása összpontosít, és felhőalapú költségvetése kényszerítését, költséges rendellenességek észlelése és a felhő cégirányítási módosításával terv esetén a tényleges kiadásokat helytelen igazítású.

Az Azure natív eszközök, amelyek segíthetnek a házirendek és a folyamatok, amelyek támogatják a cégirányítási fegyelem részletes listáját a következő:

|  | [Azure Portal](https://azure.microsoft.com/features/azure-portal/)  | [Azure Cost Management](/azure/cost-management/overview-cost-mgt)  | [Azure EA Content Pack](/power-bi/service-connect-to-azure-enterprise)  | [Azure Policy](/azure/governance/policy/overview) |
|---------|---------|---------|---------|---------|
|Nagyvállalati szerződés szükséges?     | Nem         | Igen (nem kötelező a [Cloudyn](/azure/cost-management/overview))         | Igen         | Nem         |
|Költségvetés-ellenőrzés     | Nem         | Igen         | Nem         | Igen         |
|A figyelő a költségkeret-beállítási egyetlen erőforrás    | Igen         | Igen         | Igen         | Nem         |
|Kiadások között több erőforrást figyelő    | Nem         | Igen        | Igen         | Nem         |
|Ellenőrzés egyetlen erőforrás kiadások     | Igen – manuális méretezése         | Igen         | Nem         | Igen         |
|Kiadások között több erőforrást kényszerítése    | Nem         | Igen         | Nem         | Igen         |
|Figyelheti, és megfigyelheti a szabályszerűségeket     | Igen – korlátozott         | Igen        | Igen         | Nem         |
|Költségkeret rendellenességek észlelése     | Nem         | Igen        | Igen         | Nem        |
|Eltérések üzletkötés     | Nem        | Igen        | Igen        | Nem        |
