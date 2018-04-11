---
title: 'Azure bevezetése: eligazodást'
description: Az alapkonfiguráció szintű bevezessék az Azure számára szükséges vállalati Tudásbázis
author: petertay
ms.openlocfilehash: e9421b610e4eb07a3ed37bca56e513b0689484ef
ms.sourcegitcommit: 9ba82cf84cee06ccba398ec04c51dab0e1ca8974
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/13/2018
---
# <a name="adopting-azure-foundational"></a>Azure bevezetése: eligazodást

Az első szakasza a vállalati szervezeti lejárat Azure bevezetése. Az ebben a szakaszban végén a szervezet dolgozói telepíthetik egyszerű munkaterhelések az Azure-bA.

Az alábbi lista tartalmazza a feladatok befejezése a legalapvetőbb Bevezetési fázis. A lista létrehozási fokozatos így elvégzendő feladatokhoz sorrendben. Ha korábban a feladat befejeződött, helyezze át a következő tevékenység a listában. 

1. Az Azure belső megismerése:
    - **Explainer:** [hogyan Azure működik?](azure-explainer.md)
2. Vállalati digitális identitásokat az Azure-ban megismerése:
    - **Explainer:** [Mi az az Azure Active Directory-bérlő?](tenant-explainer.md)
    - **Hogyan:** [Azure Active Directory-bérlő beszerzése](/azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json)
    - **Útmutató:** [az Azure AD bérlő tervezési útmutató](tenant.md)
    - **Hogyan:** [új felhasználók hozzáadása az Azure Active Directoryban](/azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json)    
3. Azure-előfizetések megismerése:
    - **Explainer:** [Mi az Azure-előfizetés?](subscription-explainer.md)
    - **Útmutató:** [Azure-előfizetés kialakítása](subscription.md)
4. Erőforrás-kezelés az Azure-ban megismerése: 
    - **Explainer:** [Mi az Azure Resource Manager?](resource-manager-explainer.md)
    - **Explainer:** [Mi az az Azure-erőforráscsoport?](resource-group-explainer.md)
    - **Explainer:** [az az Azure-erőforrások hozzáférésének megismerése](/azure/active-directory/active-directory-understanding-resource-access?toc=/azure/architecture/cloud-adoption-guide/toc.json)
    - **Hogyan:** [az Azure portál használata az Azure erőforrás-csoport létrehozása](/azure/azure-resource-manager/resource-group-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)
    - **Útmutató:** [Azure-erőforrás csoport tervezési útmutató](resource-group.md)
    - **Útmutató:** [elnevezési konvenciói Azure-erőforrások](/azure/architecture/best-practices/naming-conventions?toc=/azure/architecture/cloud-adoption-guide/toc.json)
5. Telepítsen egy alapszintű Azure-architektúra:
    - További tudnivalók az Azure compute beállítások, például infrastruktúra,--szolgáltatás (IaaS) és a Platform,--szolgáltatás (PaaS) a különböző típusú a a [áttekintése Azure számítási lehetőségek](/azure/architecture/guide/technology-choices/compute-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json).
    - Most, hogy megismerte a különböző típusú Azure számítási lehetőségek, válasszon egy PaaS webalkalmazás vagy az infrastruktúra-szolgáltatási virtuális gép az Azure-ban az első erőforrásként:
    - PaaS: Platformok bemutatása:
        - **Hogyan:** [központi telepítése egy alapszintű webalkalmazást az Azure-bA](/azure/app-service/app-service-web-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **Útmutató:** központi telepítésének eljárásai bizonyítása egy [alapvető webalkalmazás](/azure/architecture/reference-architectures/app-service-web-app/basic-web-app?toc=/azure/architecture/cloud-adoption-guide/toc.json) az Azure-bA
    - Infrastruktúra-szolgáltatási: Virtuális hálózati bemutatása:
        - **Explainer:** [Azure-beli virtuális hálózat](/azure/virtual-network/virtual-networks-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **Hogyan:** [telepíthet virtuális hálózatot az Azure portál használatával](/azure/virtual-network/virtual-networks-create-vnet-arm-pportal?toc=/azure/architecture/cloud-adoption-guide/toc.json)
    - Számvitel: Telepítsen egy egyetlen virtuális machine(VM) munkaterhelés (a Windows és Linux):
        - **Hogyan:** [telepítse a Windows virtuális gépek az Azure portálon](/azure/virtual-machines/windows/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **Útmutató:** [bizonyítása eljárások az Azure-on futó Windows virtuális gép](/azure/architecture/reference-architectures/virtual-machines-windows/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **Hogyan:** [Linux virtuális gép telepítse az Azure portálon](/azure/virtual-machines/linux/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json)
        - **Útmutató:** [bizonyítása eljárások az Azure-on futó Linux virtuális gép](/azure/architecture/reference-architectures/virtual-machines-linux/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json)
