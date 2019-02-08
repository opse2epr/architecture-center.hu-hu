---
title: 'CAF: Szoftveralapú hálózatok – PaaS csak'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: A PaaS-felhő csak modell veszik górcső alá alapú hálózatkezelési funkció
author: rotycenh
ms.openlocfilehash: 2f3f82d781ddb6544721e82e7b7d795222a2f8ff
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899246"
---
# <a name="software-defined-networks-paas-only"></a><span data-ttu-id="75727-103">A szoftveralapú hálózatok: Csak PaaS</span><span class="sxs-lookup"><span data-stu-id="75727-103">Software Defined Networks: PaaS-only</span></span>

<span data-ttu-id="75727-104">Platformszolgáltatás (PaaS) erőforrás platformként megvalósításának, a telepítési folyamat automatikusan hálózatot hoz létre egy feltételezett alapul szolgáló vezérlők a hálózaton, többek között a terheléselosztást, blokkolja a port és egyéb PaaS kapcsolatokat csak korlátozott számú szolgáltatások.</span><span class="sxs-lookup"><span data-stu-id="75727-104">When you implement a platform as a service (PaaS) resource, the deployment process automatically creates an assumed underlying network with a limited number of controls over that network, including load balancing, port blocking, and connections to other PaaS services.</span></span>

<span data-ttu-id="75727-105">Az Azure-ban, a PaaS-erőforrás számos különböző lehet [üzembe helyezett](/azure/virtual-network/virtual-network-for-azure-services) vagy [csatlakozik](/azure/virtual-network/virtual-network-service-endpoints-overview) egy virtuális hálózatot, lehetővé téve az ezekhez az erőforrásokhoz való integrációhoz a meglévő virtuális hálózati infrastruktúra.</span><span class="sxs-lookup"><span data-stu-id="75727-105">In Azure, several PaaS resource types can be [deployed into](/azure/virtual-network/virtual-network-for-azure-services) or [connected to](/azure/virtual-network/virtual-network-service-endpoints-overview) a virtual network, allowing these resources to integrate with your existing virtual networking infrastructure.</span></span> <span data-ttu-id="75727-106">Azonban a sok esetben ezek hálózati natív módon a PaaS-erőforrásokhoz, által biztosított lehetőségeket alapértelmezés támaszkodva csak hálózati architektúrát, PaaS, elegendő számítási feladat igényeinek.</span><span class="sxs-lookup"><span data-stu-id="75727-106">However, in many cases a PaaS only networking architecture, relying only on these default networking capabilities natively provided by PaaS resources, is sufficient to meet workload requirements.</span></span>

<span data-ttu-id="75727-107">Ha egy PaaS, csak a hálózati architektúra használatát fontolgatja, mindenképpen ellenőrizze, hogy a szükséges előfeltételek összhangba kerüljenek a követelmények.</span><span class="sxs-lookup"><span data-stu-id="75727-107">If you are considering a PaaS only networking architecture, be sure you validate that the required assumptions align with your requirements.</span></span>

## <a name="paas-only-assumptions"></a><span data-ttu-id="75727-108">Csak PaaS feltételezések</span><span class="sxs-lookup"><span data-stu-id="75727-108">PaaS-only assumptions</span></span>

<span data-ttu-id="75727-109">A PaaS-csak hálózati architektúra üzembe helyezéséhez feltételezi, hogy a következő:</span><span class="sxs-lookup"><span data-stu-id="75727-109">Deploying a PaaS-only networking architecture assumes the following:</span></span>

- <span data-ttu-id="75727-110">Az alkalmazás üzembe helyezéséhez egy különálló alkalmazás, vagy egyéb PaaS-erőforrásokhoz függ.</span><span class="sxs-lookup"><span data-stu-id="75727-110">The application being deployed is a standalone application OR is dependent on only other PaaS resources.</span></span>
- <span data-ttu-id="75727-111">Az IT-üzemeltetési csapatokra frissítheti az eszközök, tanfolyamok és folyamatokat a felügyeleti, konfigurációs és központi telepítését a különálló PaaS-alkalmazások.</span><span class="sxs-lookup"><span data-stu-id="75727-111">Your IT operations teams can update their tools, training, and processes to support management, configuration, and deployment of standalone PaaS applications.</span></span>
- <span data-ttu-id="75727-112">A PaaS-alkalmazás nem szerepel egy szélesebb körű felhőalapú áttelepítési erőfeszítés, amely IaaS-erőforrásokat tartalmazza.</span><span class="sxs-lookup"><span data-stu-id="75727-112">The PaaS application is not part of a broader cloud migration effort that will include IaaS resources.</span></span>

<span data-ttu-id="75727-113">Ilyen Előfeltevések igazítva, és üzembe helyezését egy PaaS-csak hálózati minimális minősítők.</span><span class="sxs-lookup"><span data-stu-id="75727-113">These assumptions are minimum qualifiers aligned to deploying a PaaS-only network.</span></span> <span data-ttu-id="75727-114">Amíg ez a megközelítés lehetséges, hogy összhangba kerüljenek egy egyetlen alkalmazás telepítési követelményeinek, a felhő bevezetésének csapat vizsgálja meg ezeket a hosszú távú kérdéseket:</span><span class="sxs-lookup"><span data-stu-id="75727-114">While this approach may align with the requirements of a single application deployment, your Cloud Adoption Team should examine these long-term questions:</span></span>

- <span data-ttu-id="75727-115">A központi telepítés bővített a hatókörben vagy egyéb nem PaaS-erőforrásokhoz való hozzáférést igényelnek a méretezési csoport?</span><span class="sxs-lookup"><span data-stu-id="75727-115">Will this deployment expand in scope or scale to require access to other non-PaaS resources?</span></span>
- <span data-ttu-id="75727-116">Egyéb PaaS üzemelő példányok tervezett túl az aktuális megoldáshoz?</span><span class="sxs-lookup"><span data-stu-id="75727-116">Are other PaaS deployments planned beyond the current solution?</span></span>
- <span data-ttu-id="75727-117">Rendelkezik a szervezet más a migrálás a felhőbe jövőbeli terveket?</span><span class="sxs-lookup"><span data-stu-id="75727-117">Does the organization have plans for other future cloud migrations?</span></span>

<span data-ttu-id="75727-118">Az ezekre a kérdésekre adott válaszokat lenne nem zárja ki, hogy egy csapat egy PaaS egyetlen művelettel, de értendők, mielőtt az utolsó.</span><span class="sxs-lookup"><span data-stu-id="75727-118">The answers to these questions would not preclude a team from choosing a PaaS only option but should be considered before making a final decision.</span></span>
