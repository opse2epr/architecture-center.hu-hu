---
title: "Útmutató: Azure AD bérlő kialakítása"
description: "Azure-bérlőhöz tervezési útmutató a legalapvetőbb felhő bevezetési stratégia részeként"
author: telmosampaio
ms.openlocfilehash: 5bf710f74bde04e041f2896b4a638c3513e4aa44
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/09/2018
---
# <a name="guidance-azure-ad-tenant-design"></a>Útmutató: Azure AD bérlő kialakítása

Az Azure AD-bérlő biztosít a digitális identitásokat szolgáltatások és egy vagy több használt névterek [Azure-előfizetések](subscription-explainer.md). Ha a legalapvetőbb elfogadása vázlat, már megtanulta, [az Azure AD-bérlő beszerzése][how-to-get-aad-tenant]. 

## <a name="design-considerations"></a>Kialakítási szempontok

- A legalapvetőbb elfogadása szakaszban el lehet kezdeni az egyetlen Azure AD-bérlő. A szervezet meglévő Office 365-előfizetéssel, vagy az Azure-előfizetéssel rendelkezik, ha már rendelkezik az Azure AD-bérlő használhat. Ha nem rendelkezik vagy vagy ezek részletesebb információk [az Azure AD-bérlő beszerzése][how-to-get-aad-tenant]. 
- A köztes és speciális elfogadása fázisból áll akkor megtudhatja, hogyan szinkronizálásához, vagy összevonni a helyszíni címtárakat az Azure ad-val. Ez lehetővé teszi a helyszíni digitális identitás használatára az Azure ad-ben. Azonban a legalapvetőbb szakaszban fel szeretné venni az új felhasználók, amelyek csak az egyetlen az identitással rendelkezik az Azure AD-bérlő. Ezeket az identitásokat kezeléséért is lehet. Például, hogy az új Azure-bA a helyi Active Directory-felhasználók, az Azure AD az board-felhasználók, amelyek már nem szeretne van Azure-erőforrások eléréséhez, és egyéb felhasználói engedélyek vált.

## <a name="next-steps"></a>További lépések

* Most, hogy az Azure AD-bérlő, további [a felhasználó hozzáadása][azure-ad-add-user]. Miután egy vagy több új felhasználók felvétele az Azure AD-bérlő, a következő lépés megtanulni [Azure-előfizetések](subscription-explainer.md).

<!-- Links -->

[azure-ad-add-user]: /azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-manage-azure-ad]: /azure/active-directory/active-directory-administer?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-tenant]: /azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json
[docs-associate-subscription]: /azure/active-directory/active-directory-how-subscriptions-associated-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[how-to-get-aad-tenant]: /azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json