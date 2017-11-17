---
title: "Egy a helyszíni hálózathoz való kapcsolódáshoz Azure megoldás kiválasztása"
description: "Hasonlítja össze egy a helyszíni hálózathoz való kapcsolódáshoz Azure architektúrák hivatkozik."
author: telmosampaio
ms.date: 04/06/2017
ms.openlocfilehash: 274b9df1817632a7f3eaafa8bf02e965fdc3feea
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="choose-a-solution-for-connecting-an-on-premises-network-to-azure"></a><span data-ttu-id="7f6f5-103">Egy a helyszíni hálózathoz való kapcsolódáshoz Azure megoldás kiválasztása</span><span class="sxs-lookup"><span data-stu-id="7f6f5-103">Choose a solution for connecting an on-premises network to Azure</span></span>

<span data-ttu-id="7f6f5-104">Ez a cikk egy a helyszíni hálózathoz való kapcsolódáshoz az Azure Virtual Network (VNet) lehetőségeket hasonlítja össze.</span><span class="sxs-lookup"><span data-stu-id="7f6f5-104">This article compares options for connecting an on-premises network to an Azure Virtual Network (VNet).</span></span> <span data-ttu-id="7f6f5-105">Az egyes lehetőségek nyújtunk a referencia-architektúrában és a központilag telepíthető megoldás.</span><span class="sxs-lookup"><span data-stu-id="7f6f5-105">We provide a reference architecture and a deployable solution for each option.</span></span>

## <a name="vpn-connection"></a><span data-ttu-id="7f6f5-106">VPN-kapcsolat</span><span class="sxs-lookup"><span data-stu-id="7f6f5-106">VPN connection</span></span>

<span data-ttu-id="7f6f5-107">A virtuális magánhálózat (VPN) segítségével csatlakozzon a helyi hálózaton egy Azure virtuális hálózatot egy IPSec VPN-alagúton keresztül.</span><span class="sxs-lookup"><span data-stu-id="7f6f5-107">Use a virtual private network (VPN) to connect your on-premises network with an Azure VNet through an IPSec VPN tunnel.</span></span>

<span data-ttu-id="7f6f5-108">Ez az architektúra hibrid alkalmazásoknál, ahol a forgalom között helyszíni hardver és a felhő várhatóan világos, vagy a rendszer a rugalmasság és a feldolgozási kapacitása révén a felhő némileg kiterjesztett késése kereskedelmi.</span><span class="sxs-lookup"><span data-stu-id="7f6f5-108">This architecture is suitable for hybrid applications where the traffic between on-premises hardware and the cloud is likely to be light, or you are willing to trade slightly extended latency for the flexibility and processing power of the cloud.</span></span>

<span data-ttu-id="7f6f5-109">**Előnyei**</span><span class="sxs-lookup"><span data-stu-id="7f6f5-109">**Benefits**</span></span>

- <span data-ttu-id="7f6f5-110">Konfigurálása egyszerű.</span><span class="sxs-lookup"><span data-stu-id="7f6f5-110">Simple to configure.</span></span>

<span data-ttu-id="7f6f5-111">**Kihívásai**</span><span class="sxs-lookup"><span data-stu-id="7f6f5-111">**Challenges**</span></span>

- <span data-ttu-id="7f6f5-112">Szükséges egy helyszíni VPN-eszközön.</span><span class="sxs-lookup"><span data-stu-id="7f6f5-112">Requires an on-premises VPN device.</span></span>
- <span data-ttu-id="7f6f5-113">Bár a Microsoft biztosítja, hogy a 99,9 %-os rendelkezésre állás minden VPN-átjáró, a szolgáltatásiszint-szerződés vonatkozik a VPN-átjáró, és nem a hálózati kapcsolat az átjáróhoz.</span><span class="sxs-lookup"><span data-stu-id="7f6f5-113">Although Microsoft guarantees 99.9% availability for each VPN Gateway, this SLA only covers the VPN gateway, and not your network connection to the gateway.</span></span>
- <span data-ttu-id="7f6f5-114">VPN-kapcsolaton keresztül Azure VPN Gateway jelenleg legfeljebb 200 MB/s sávszélesség.</span><span class="sxs-lookup"><span data-stu-id="7f6f5-114">A VPN connection over Azure VPN Gateway currently supports a maximum of 200 Mbps bandwidth.</span></span> <span data-ttu-id="7f6f5-115">Szükség lehet az Azure virtual Networkhöz particionálásához több VPN-kapcsolatokon keresztül, ha várhatóan hosszabb legyen, mint a teljesítményt.</span><span class="sxs-lookup"><span data-stu-id="7f6f5-115">You may need to partition your Azure virtual network across multiple VPN connections if you expect to exceed this throughput.</span></span>

<span data-ttu-id="7f6f5-116">**[További tudnivalók...][vpn]**</span><span class="sxs-lookup"><span data-stu-id="7f6f5-116">**[Read more...][vpn]**</span></span>

## <a name="azure-expressroute-connection"></a><span data-ttu-id="7f6f5-117">Az Azure ExpressRoute-kapcsolat</span><span class="sxs-lookup"><span data-stu-id="7f6f5-117">Azure ExpressRoute connection</span></span>

<span data-ttu-id="7f6f5-118">ExpressRoute-kapcsolatot egy külső kapcsolatot közvetítésével saját, dedikált kapcsolatot használja.</span><span class="sxs-lookup"><span data-stu-id="7f6f5-118">ExpressRoute connections use a private, dedicated connection through a third-party connectivity provider.</span></span> <span data-ttu-id="7f6f5-119">A magánhálózati kapcsolat az Azure a helyszíni hálózat kiterjesztése.</span><span class="sxs-lookup"><span data-stu-id="7f6f5-119">The private connection extends your on-premises network into Azure.</span></span> 

<span data-ttu-id="7f6f5-120">Ez az architektúra futó nagyméretű, kritikus, méretezhetőség magas fokú igénylő munkaterhelések hibrid alkalmazások alkalmas.</span><span class="sxs-lookup"><span data-stu-id="7f6f5-120">This architecture is suitable for hybrid applications running large-scale, mission-critical workloads that require a high degree of scalability.</span></span> 

<span data-ttu-id="7f6f5-121">**Előnyei**</span><span class="sxs-lookup"><span data-stu-id="7f6f5-121">**Benefits**</span></span>

- <span data-ttu-id="7f6f5-122">Sokkal nagyobb sávszélesség érhető el; legfeljebb 10 GB/s attól függően, hogy a kapcsolat szolgáltatóját.</span><span class="sxs-lookup"><span data-stu-id="7f6f5-122">Much higher bandwidth available; up to 10 Gbps depending on the connectivity provider.</span></span>
- <span data-ttu-id="7f6f5-123">Támogatja a dinamikus méretezés sávszélesség során alacsonyabb igényű időszakainak költségek csökkentése érdekében.</span><span class="sxs-lookup"><span data-stu-id="7f6f5-123">Supports dynamic scaling of bandwidth to help reduce costs during periods of lower demand.</span></span> <span data-ttu-id="7f6f5-124">Azonban nem minden kapcsolat szolgáltatók kell ezt a beállítást.</span><span class="sxs-lookup"><span data-stu-id="7f6f5-124">However, not all connectivity providers have this option.</span></span>
- <span data-ttu-id="7f6f5-125">Engedélyezheti a szervezet nemzeti felhők, attól függően, hogy a kapcsolat szolgáltatójánál közvetlen hozzáférést.</span><span class="sxs-lookup"><span data-stu-id="7f6f5-125">May allow your organization direct access to national clouds, depending on the connectivity provider.</span></span>
- <span data-ttu-id="7f6f5-126">99,9 %-os SLA-elérhetőséget az egész kapcsolaton keresztül.</span><span class="sxs-lookup"><span data-stu-id="7f6f5-126">99.9% availability SLA across the entire connection.</span></span>

<span data-ttu-id="7f6f5-127">**Kihívásai**</span><span class="sxs-lookup"><span data-stu-id="7f6f5-127">**Challenges**</span></span>

- <span data-ttu-id="7f6f5-128">Összetett feladat lehet beállítása.</span><span class="sxs-lookup"><span data-stu-id="7f6f5-128">Can be complex to set up.</span></span> <span data-ttu-id="7f6f5-129">Egy ExpressRoute-kapcsolat létrehozásához szükséges külső kapcsolatot szolgáltatóval működő.</span><span class="sxs-lookup"><span data-stu-id="7f6f5-129">Creating an ExpressRoute connection requires working with a third-party connectivity provider.</span></span> <span data-ttu-id="7f6f5-130">A szolgáltató feladata a hálózati kapcsolat történő üzembe helyezéséhez.</span><span class="sxs-lookup"><span data-stu-id="7f6f5-130">The provider is responsible for provisioning the network connection.</span></span>
- <span data-ttu-id="7f6f5-131">Helyszíni nagy sávszélességű útválasztók igényel.</span><span class="sxs-lookup"><span data-stu-id="7f6f5-131">Requires high-bandwidth routers on-premises.</span></span>

<span data-ttu-id="7f6f5-132">**[További tudnivalók...][expressroute]**</span><span class="sxs-lookup"><span data-stu-id="7f6f5-132">**[Read more...][expressroute]**</span></span>

## <a name="expressroute-with-vpn-failover"></a><span data-ttu-id="7f6f5-133">A VPN-feladatátvételi ExpressRoute</span><span class="sxs-lookup"><span data-stu-id="7f6f5-133">ExpressRoute with VPN failover</span></span>

<span data-ttu-id="7f6f5-134">A beállítások az előző két, ExpressRoute segítségével normál körülmények között, de feladatátadás a VPN-kapcsolatot, ha a kapcsolat az ExpressRoute-kapcsolatcsoport a veszteség egyesíti.</span><span class="sxs-lookup"><span data-stu-id="7f6f5-134">This options combines the previous two, using ExpressRoute in normal conditions, but failing over to a VPN connection if there is a loss of connectivity in the ExpressRoute circuit.</span></span>

<span data-ttu-id="7f6f5-135">Ebbe az architektúrába, ami a hibrid alkalmazásokat, amelyek az ExpressRoute nagyobb sávszélességet kell, és azt is igénylik, magas rendelkezésre állású hálózati kapcsolat megfelelő.</span><span class="sxs-lookup"><span data-stu-id="7f6f5-135">This architecture is suitable for hybrid applications that need the higher bandwidth of ExpressRoute, and also require highly available network connectivity.</span></span> 

<span data-ttu-id="7f6f5-136">**Előnyei**</span><span class="sxs-lookup"><span data-stu-id="7f6f5-136">**Benefits**</span></span>

- <span data-ttu-id="7f6f5-137">Magas rendelkezésre állású Ha ExpressRoute-kapcsolatcsoport nem sikerül, a tartalék kapcsolatot azonban alacsonyabb sávszélességű hálózaton.</span><span class="sxs-lookup"><span data-stu-id="7f6f5-137">High availability if the ExpressRoute circuit fails, although the fallback connection is on a lower bandwidth network.</span></span>

<span data-ttu-id="7f6f5-138">**Kihívásai**</span><span class="sxs-lookup"><span data-stu-id="7f6f5-138">**Challenges**</span></span>

- <span data-ttu-id="7f6f5-139">Összetett konfigurálásához.</span><span class="sxs-lookup"><span data-stu-id="7f6f5-139">Complex to configure.</span></span> <span data-ttu-id="7f6f5-140">Akkor be kell állítania egy VPN-kapcsolat és a ExpressRoute-kapcsolatcsoportot.</span><span class="sxs-lookup"><span data-stu-id="7f6f5-140">You need to set up both a VPN connection and an ExpressRoute circuit.</span></span>
- <span data-ttu-id="7f6f5-141">Redundáns hardveres (VPN-készülék), valamint a redundáns Azure VPN Gateway kapcsolat, amelynek díjakat kell fizetnie kell.</span><span class="sxs-lookup"><span data-stu-id="7f6f5-141">Requires redundant hardware (VPN appliances), and a redundant Azure VPN Gateway connection for which you pay charges.</span></span>

<span data-ttu-id="7f6f5-142">**[További tudnivalók...][expressroute-vpn-failover]**</span><span class="sxs-lookup"><span data-stu-id="7f6f5-142">**[Read more...][expressroute-vpn-failover]**</span></span>

<!-- links -->
[expressroute]: ./expressroute.md
[expressroute-vpn-failover]: ./expressroute-vpn-failover.md
[vpn]: ./vpn.md