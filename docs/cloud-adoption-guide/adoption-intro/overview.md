---
title: 'Az Azure bevezetésének: alapvető'
description: Ismerteti az alapszintű, hogy a vállalati Azure elfogadására igényel Tudásbázis
author: petertay
ms.openlocfilehash: b5a0a4a2c4ed1d06c97774b0eca643a89a5a2110
ms.sourcegitcommit: c704d5d51c8f9bbab26465941ddcf267040a8459
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 07/24/2018
ms.locfileid: "39229167"
---
# <a name="adopting-microsoft-azure-foundational"></a>A Microsoft Azure bevezetésével: alapvető

Egy szervezet, amely új a felhőalapú technológiák terén nehézkes lehet dönthet arról, hogy a megfelelő helyen a bevezetési folyamata megkezdéséhez. Az alapszintű bevezetési szakasz célja a kiindulási pont biztosítása. Miután a szervezeten belül másokhoz is dolgozott már keresztül ebben a szakaszban rendelkeznek minden a tudását és szakismereteit az Azure-bA egy egyszerű számítási feladat a számítási erőforrások telepítéséhez szükséges. 

> [!NOTE]
> Ez az útmutató nem tárgyalja alkalmazások fejlesztését. Az Azure-alkalmazások fejlesztésével kapcsolatos további információkért lásd: a [útmutató az Azure Alkalmazásarchitektúrájához](/azure/architecture/guide/).

Az útmutató ezen szakasza a célközönségét a következő személyeknek a szervezeten belül:

- *Pénzügyi:* a pénzügyi kötelezettségvállalás nélkül, az Azure-ba, szabályzatokat és eljárásokat erőforrás használat költségeit, beleértve a számlázás és költséghelyi elszámolás kialakításáért felelős tulajdonosa.
- *Központi informatikai:* felelős a szervezet felhőbeli erőforrásokhoz, beleértve az erőforrás-kezelés és a hozzáférés és a számítási feladatok állapota és figyelése.
- *Számítási feladatok:* minden fejlesztési szerepkör, amely a számítási feladatok üzembe helyezése az Azure-ba, beleértve, fejlesztők és tesztelők, részt, és állítsa össze a mérnökök.

## <a name="section-1-azure-basics"></a>1. szakasz: Azure-alapokkal

Ez a bevezető szakasz célja a *pénzügyi* és *központi informatikai* személyeknek. A lépéseknek az ismertetése, ez a szakasz megszereznek alapvető ismeretekkel [Azure működéséről](azure-explainer.md) megismerését előkészítésekor a [felhőalapú cégirányítási fogalma](governance-explainer.md). Is lehet hasznos, ha a *számítási feladatok* tekintse át ezt a tartalmat, amelyekkel megismerheti, hogyan történik az erőforrás-hozzáférés kezelése a szervezetben.

## <a name="section-2-governance-design-guide"></a>2. szakasz: Cégirányítási kialakítási útmutató

Most, hogy megismerkedett az Azure működéséről és bevezetése az Azure az első lépés a felhő goverance alapjait, van ismerkedem [erőforráshozzáférés-kezelés](azure-resource-access.md) az Azure-ban. Ez a cikk ismerteti az Azure-szolgáltatások erőforrás-hozzáférési kérelmek és a vezérlőket, azt ellenőrzi ezeket a kérelmeket.

A következő lépés a tanulási útmutató [cégirányítási mintájának](governance-how-to.md) egyetlen csapatok számára. Ez a cikk bemutatja, hogyan konfigurálhatja az erőforrás-hozzáférési felügyeleti szolgáltatások és a vezérlők megismerkedett a korábban.

## <a name="section-3-implementing-a-basic-resource-access-management-model"></a>3. szakasz: Egy alapvető erőforrás-hozzáférési felügyeleti modell megvalósítása

Az utolsó lépés a bevezetési folyamata az, hogy a korábban készült cégirányítási modell megvalósítása. 

Első lépésként a munkahely megköveteli az Azure-fiók. Ha a szervezet rendelkezik egy meglévő [Microsoft nagyvállalati szerződés](https://www.microsoft.com/licensing/licensing-programs/enterprise.aspx) , amely nem tartalmazza az Azure, Azure azáltal, hogy a vonatkozó előzetes pénzügyi kötelezettségvállalást is hozzáadhatók. Lásd: [nagyvállalati licencek használata az Azure](https://azure.microsoft.com/pricing/enterprise-agreement/) további információt. 

Az Azure-fiók létrehozásakor a szervezete az Azure adjon meg egy személy **fióktulajdonos**. Az Azure Active Directory (Azure AD) bérlő majd jön létre alapértelmezés szerint. Az Azure **fióktulajdonos** kell [a felhasználói fiók létrehozása](/azure/active-directory/add-users-azure-active-directory) számára a cégnél a személy, aki a **számítási feladatok felelőse**. 

Mellett, az Azure **fióktulajdonos** kell [hozzon létre egy előfizetést](https://docs.microsoft.com/partner-center/create-a-new-subscription) és [az Azure AD-bérlő társítása](/azure/active-directory/fundamentals/active-directory-how-subscriptions-associated-directory) vele.

Végezetül most, hogy az előfizetés jön létre, és az Azure AD-bérlővel társítva hozzá, akkor is [adja hozzá a **számítási feladatok felelőse** az előfizetéshez, a beépített **tulajdonosa** szerepkör](/azure/billing/billing-add-change-azure-subscription-administrator#add-an-rbac-owner-for-a-subscription-in-azure-portal).

## <a name="section-4-deploy-a-basic-workload-architecture-to-azure"></a>4. szakasz: Helyezze üzembe az Azure egy alapvető számítási feladatok architektúra

Az ebben a szakaszban célközönségét a *számítási feladatok felelőse* személy. *Számítási feladatok* határozza meg a számítási és hálózatkezelési azok számítási feladatokhoz, válassza ki a megfelelő erőforrásokat erőforrásigények kielégítéséhez, és telepítheti őket. 

Az alapszintű bevezetési szakasz a számítási feladatok felelőse kiválaszthatják alapszintű webalkalmazásokat, vagy egy virtuális hálózat (VNet) és a virtuális gép (VM). Az Azure számítási lehetőségeinek különböző típusaival kapcsolatos további információkért tekintse át a [áttekintése az Azure számítási lehetőségek](/azure/architecture/guide/technology-choices/compute-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json).

Függetlenül attól, hogy mely számítási lehetőséget választja, egyes történő központi telepítését igényli a **erőforráscsoport**. A **számítási feladatok felelőse** kell [hozza létre az erőforráscsoportot](/azure/azure-resource-manager/vs-azure-tools-resource-groups-deployment-projects-create-deploy). Az üzembe helyezés részeként a **számítási feladatok felelőse** az erőforráscsoport nevét adja meg. Ez a név a következő szakaszban használható.

### <a name="basic-web-application-paas"></a>Alapszintű webalkalmazás (PaaS)

Alapszintű webalkalmazás, válassza az 5 perces gyors útmutatók az egyik a [web apps-dokumentáció](/azure/app-service?toc=/azure/architecture/cloud-adoption-guide/toc.json) , és kövesse a lépéseket. 

> [!NOTE]
> Alapértelmezés szerint a rövid útmutató egyes fogja telepíteni egy erőforráscsoportot. Ebben az esetben a **számítási feladatok felelőse** nem kell explicit módon hozzon létre egy erőforráscsoportot. Ellenkező esetben üzembe helyezése a webalkalmazás a fent létrehozott erőforráscsoportot.

Az egyszerű alkalmazások és szolgáltatások üzembe helyezését követően többet is megtudhat telepítéséhez a bevált gyakorlatokat kapcsolatos egy [alapszintű webalkalmazás](/azure/architecture/reference-architectures/app-service-web-app/basic-web-app?toc=/azure/architecture/cloud-adoption-guide/toc.json) az Azure-bA.

### <a name="single-windows-or-linux-vm-iaas"></a>Egyetlen Windows vagy Linux rendszerű virtuális gépek (IaaS)

Egy egyszerű számítási feladat, amely a virtuális gépen fut az első lépés, hogy a virtuális hálózat üzembe helyezése. Például a virtual machines, a terheléselosztók és átjárók minden IaaS-erőforrásokat az Azure-beli virtuális hálózat szükséges. Ismerje meg [Azure virtuális hálózatok](/azure/virtual-network/virtual-networks-overview?toc=/azure/architecture/cloud-adoption-guide/toc.json), majd kövesse a lépéseket [üzembe helyezése egy virtuális hálózatot az Azure-ban a portál](/azure/virtual-network/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json). Az Azure Portalon a virtuális hálózatra vonatkozó beállításainak megadásakor adja meg a fent létrehozott erőforráscsoport nevét.

A következő lépés, hogy egyetlen Windows vagy Linux rendszerű virtuális gép üzembe kell-e. Windows virtuális gép esetén kövesse a lépéseket a [Windows virtuális gép üzembe helyezése az Azure Portal](/azure/virtual-machines/windows/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json). Újra amikor az Azure Portalon adja meg a virtuális gép beállításait, adja meg a fent létrehozott erőforráscsoport nevét.

Miután követte a lépéseket, és a virtuális Gépet üzembe helyezett, megismerkedhet a [bevált eljárások az Azure-on futó Windows virtuális gép](/azure/architecture/reference-architectures/virtual-machines-windows/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json). Linux rendszerű virtuális gép esetén kövesse a lépéseket a [Linux virtuális gép üzembe helyezése az Azure-bA a portállal](/azure/virtual-machines/linux/quick-create-portal?toc=/azure/architecture/cloud-adoption-guide/toc.json). Hogy még többet is megtudhat [bevált eljárások az Azure-on futó Linux rendszerű virtuális gép](/azure/architecture/reference-architectures/virtual-machines-linux/single-vm?toc=/azure/architecture/cloud-adoption-guide/toc.json).

## <a name="next-steps"></a>További lépések

A következő feladat a felhőre való előkészületként a [ **fázis köztes**](../intermediate-stage/overview.md). A köztes fázisban ismertetik a fut több számítási feladatok és cégirányítási modellek több csapatok számára a helyszíni hálózat kiterjesztése.