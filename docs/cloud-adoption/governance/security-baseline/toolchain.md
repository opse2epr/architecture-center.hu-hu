---
title: 'CAF: Biztonsági alapkonfiguráció eszközök az Azure-ban'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Magyarázat az eszközök, amely megkönnyíti a továbbfejlesztett biztonsági alapterv, az Azure-ban
author: BrianBlanchard
ms.openlocfilehash: b316626c8ad717514f7f592abefa0f33a92afdca
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899183"
---
# <a name="security-baseline-tools-in-azure"></a>Biztonsági alapkonfiguráció eszközök az Azure-ban

[Alapvető biztonsági](overview.md) egyike a [öt oktatnak, felhőalapú Cégirányítási](../governance-disciplines.md). Ez fegyelem összpontosít módját a házirendekben, amelyek védelmét a hálózati eszközök és a legtöbb fontosabb helyezkednek el egy Felhőszolgáltató megoldás az adatok. Belül a felhő irányítási öt állapítják alapvető biztonsági tartalmazza a digitális hagyatéki és az adatok besorolását. Dokumentáció a kockázatokat, üzleti tűréshatár és kockázatcsökkentési stratégia az adatok, eszközök és hálózati biztonsági társított is tartalmaz. Technikai szempontból, ide is bevonása döntésekhez kapcsolatos [titkosítási](../../decision-guides/encryption/overview.md), [hálózati követelmények](../../decision-guides/software-defined-network/overview.md), [hibrid identitáskezelési stratégiák](../../decision-guides/identity/overview.md), és eszközök a [automatizálásához végrehajtás](../../decision-guides/policy-enforcement/overview.md) között a biztonsági szabályzatok [erőforráscsoportok](../../decision-guides/resource-consistency/overview.md).

Az Azure-eszközök, amelyek segíthetnek a házirendek és a folyamatok, amelyek támogatják az alapvető biztonsági részletes listáját a következő:

|                                                            | [Az Azure portal](https://azure.microsoft.com/features/azure-portal/) / [Resource Manager](/azure/azure-resource-manager/resource-group-overview)  | [Azure Key Vault](/azure/key-vault)  | [Azure AD](/azure/active-directory/fundamentals/active-directory-whatis) | [Azure Policy](/azure/governance/policy/overview) | [Azure Security Center](/azure/security-center/security-center-intro) | [Azure Monitor](/azure/azure-monitor/overview) |
|------------------------------------------------------------|---------------------------------|-----------------|----------|--------------|-----------------------|---------------|
| Hozzáférés-vezérlés alkalmazását az erőforrásokhoz és erőforrás-létrehozás   | Igen                             | Nem              | Igen      | Nem           | Nem                    | Nem            |
| Biztonságos virtuális hálózatok                                    | Igen                             | Nem              | Nem       | Igen          | Nem                    | Nem            |
| Virtuális meghajtók titkosítása                                     | Nem                              | Igen             | Nem       | Nem           | Nem                    | Nem            |
| PaaS-tárolási és adatbázis titkosítása                         | Nem                              | Igen             | Nem       | Nem           | Nem                    | Nem            |
| Hibrid identitás-szolgáltatások kezelése                            | Nem                              | Nem              | Igen      | Nem           | Nem                    | Nem            |
| Erőforrás engedélyezett típusokkal korlátozása                         | Nem                              | Nem              | Nem       | Igen          | Nem                    | Nem            |
| A földrajzi régióhoz kötött korlátozások érvényesítése                          | Nem                              | Nem              | Nem       | Igen          | Nem                    | Nem            |
| Hálózatok és az erőforrások biztonsági állapotának figyelése          | Nem                              | Nem              | Nem       | Nem           | Igen                   | Igen           |
| Rosszindulatú tevékenység észlelése                                  | Nem                              | Nem              | Nem       | Nem           | Igen                   | Igen           |
| Előrelátó módon a biztonsági rések észlelése                        | Nem                              | Nem              | Nem       | Nem           | Igen                   | Nem            |
| Biztonsági mentés és vészhelyreállítás konfigurálása                     | Igen                             | Nem              | Nem       | Nem           | Nem                    | Nem            |

Az Azure biztonsági eszközök és szolgáltatások teljes listáját lásd: [biztonsági szolgáltatások és az Azure-on elérhető](/azure/security/azure-security-services-technologies).

Emellett akkor is nagyon gyakori az ügyfelek számára használja a harmadik felektől származó eszközök alapvető biztonsági tevékenységeket elősegítő. További információkért tekintse meg a cikket [biztonsági megoldások integrálása az Azure Security Center](/azure/security-center/security-center-partner-integration).

Biztonsági eszközök, valamint a [Microsoft Trust Center](https://www.microsoft.com/trustcenter/guidance/risk-assessment) széles körű útmutatást, a jelentések és a kapcsolódó dokumentáció, amelyek segítségével a migrálási tervezés folyamatát részeként kockázatbecslés tartalmazza.
