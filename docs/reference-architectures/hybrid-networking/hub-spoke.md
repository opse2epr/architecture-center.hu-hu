---
title: Küllős hálózati topológia implementálása az Azure-ban
titleSuffix: Azure Reference Architectures
description: Küllős hálózati topológia implementálása az Azure-ban.
author: telmosampaio
ms.date: 10/08/2018
ms.custom: seodec18
ms.openlocfilehash: c7cf2923856b3c659876afcc89bb312e492c6409
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/08/2019
ms.locfileid: "54112447"
---
# <a name="implement-a-hub-spoke-network-topology-in-azure"></a><span data-ttu-id="3a008-103">Küllős hálózati topológia implementálása az Azure-ban</span><span class="sxs-lookup"><span data-stu-id="3a008-103">Implement a hub-spoke network topology in Azure</span></span>

<span data-ttu-id="3a008-104">Ez a referenciaarchitektúra bemutatja, hogyan lehetséges a küllős hálózati topológia implementálása az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="3a008-104">This reference architecture shows how to implement a hub-spoke topology in Azure.</span></span> <span data-ttu-id="3a008-105">Az *agy* egy virtuális hálózat (VNet) az Azure-ban, amely központi kapcsolódási pontként szolgál a helyszíni hálózathoz.</span><span class="sxs-lookup"><span data-stu-id="3a008-105">The *hub* is a virtual network (VNet) in Azure that acts as a central point of connectivity to your on-premises network.</span></span> <span data-ttu-id="3a008-106">A *küllők* az agyhoz kapcsolódó virtuális hálózatok, és a számítási feladatok elkülönítésére használhatók.</span><span class="sxs-lookup"><span data-stu-id="3a008-106">The *spokes* are VNets that peer with the hub, and can be used to isolate workloads.</span></span> <span data-ttu-id="3a008-107">A helyszíni adatközpont és az agy közötti forgalom egy ExpressRoute- vagy VPN-átjárókapcsolaton keresztül halad át.</span><span class="sxs-lookup"><span data-stu-id="3a008-107">Traffic flows between the on-premises datacenter and the hub through an ExpressRoute or VPN gateway connection.</span></span> <span data-ttu-id="3a008-108">[**A megoldás üzembe helyezése.**](#deploy-the-solution)</span><span class="sxs-lookup"><span data-stu-id="3a008-108">[**Deploy this solution**](#deploy-the-solution).</span></span>

<span data-ttu-id="3a008-109">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="3a008-109">![[0]][0]</span></span>

<span data-ttu-id="3a008-110">*Töltse le az architektúra [Visio-fájlját][visio-download]*</span><span class="sxs-lookup"><span data-stu-id="3a008-110">*Download a [Visio file][visio-download] of this architecture*</span></span>

<span data-ttu-id="3a008-111">A topológia előnyei a következők:</span><span class="sxs-lookup"><span data-stu-id="3a008-111">The benefits of this toplogy include:</span></span>

- <span data-ttu-id="3a008-112">**Költségmegtakarítás** a szolgáltatások központosításával, amelyeket több számítási feladat között, egyetlen helyen lehet elosztani, például a hálózati virtuális berendezések (NVA-k) és DNS-kiszolgálók.</span><span class="sxs-lookup"><span data-stu-id="3a008-112">**Cost savings** by centralizing services that can be shared by multiple workloads, such as network virtual appliances (NVAs) and DNS servers, in a single location.</span></span>
- <span data-ttu-id="3a008-113">**Az előfizetési korlátok leküzdése** a különböző előfizetésekből virtuális társhálózatok létesítésével a központi agyhoz.</span><span class="sxs-lookup"><span data-stu-id="3a008-113">**Overcome subscriptions limits** by peering VNets from different subscriptions to the central hub.</span></span>
- <span data-ttu-id="3a008-114">**Kockázatok elkülönítése** a központi IT (SecOps, InfraOps) és a számítási feladatok között (DevOps).</span><span class="sxs-lookup"><span data-stu-id="3a008-114">**Separation of concerns** between central IT (SecOps, InfraOps) and workloads (DevOps).</span></span>

<span data-ttu-id="3a008-115">Az architektúra gyakori használati módjai többek között a következők:</span><span class="sxs-lookup"><span data-stu-id="3a008-115">Typical uses for this architecture include:</span></span>

- <span data-ttu-id="3a008-116">A különböző környezetekben (például a fejlesztés, a tesztelés és a gyártás) üzembe helyezett számítási feladatok, amelyek megosztott szolgáltatásokat igényelnek (például DNS, IDS, NTP vagy AD DS).</span><span class="sxs-lookup"><span data-stu-id="3a008-116">Workloads deployed in different environments, such as development, testing, and production, that require shared services such as DNS, IDS, NTP, or AD DS.</span></span> <span data-ttu-id="3a008-117">A megosztott szolgáltatások az agyi virtuális hálózatban találhatóak, azonban minden környezet egy küllőn van üzembe helyezve, így megmarad az elszigetelés.</span><span class="sxs-lookup"><span data-stu-id="3a008-117">Shared services are placed in the hub VNet, while each environment is deployed to a spoke to maintain isolation.</span></span>
- <span data-ttu-id="3a008-118">Olyan számítási feladatok, amelyek nem igénylik az egymással való kapcsolatot, de igénylik a megosztott szolgáltatásokhoz való hozzáférést.</span><span class="sxs-lookup"><span data-stu-id="3a008-118">Workloads that do not require connectivity to each other, but require access to shared services.</span></span>
- <span data-ttu-id="3a008-119">Olyan vállalatok, amelyek központi felügyeletet igényelnek a biztonsági szempontokhoz, például egy tűzfal az agyban DMZ-ként, és küllőnkénti elkülönített felügyelet a számítási feladatokhoz.</span><span class="sxs-lookup"><span data-stu-id="3a008-119">Enterprises that require central control over security aspects, such as a firewall in the hub as a DMZ, and segregated management for the workloads in each spoke.</span></span>

## <a name="architecture"></a><span data-ttu-id="3a008-120">Architektúra</span><span class="sxs-lookup"><span data-stu-id="3a008-120">Architecture</span></span>

<span data-ttu-id="3a008-121">Az architektúra a következőkben leírt összetevőkből áll.</span><span class="sxs-lookup"><span data-stu-id="3a008-121">The architecture consists of the following components.</span></span>

- <span data-ttu-id="3a008-122">**Helyszíni hálózat**.</span><span class="sxs-lookup"><span data-stu-id="3a008-122">**On-premises network**.</span></span> <span data-ttu-id="3a008-123">A cégen belül futó helyi magánhálózat.</span><span class="sxs-lookup"><span data-stu-id="3a008-123">A private local-area network running within an organization.</span></span>

- <span data-ttu-id="3a008-124">**VPN-eszköz**.</span><span class="sxs-lookup"><span data-stu-id="3a008-124">**VPN device**.</span></span> <span data-ttu-id="3a008-125">A helyszíni hálózat számára külső kapcsolatot biztosító eszköz vagy szolgáltatás.</span><span class="sxs-lookup"><span data-stu-id="3a008-125">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="3a008-126">A VPN-eszköz lehet hardvereszköz vagy valamilyen szoftvermegoldás, amilyen például a Windows Server 2012 Útválasztás és távelérés szolgáltatása (Routing and Remote Access Service, RRAS).</span><span class="sxs-lookup"><span data-stu-id="3a008-126">The VPN device may be a hardware device, or a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="3a008-127">A támogatott VPN-berendezések listáját és az egyes VPN-berendezések Azure-hoz való csatlakoztatásra történő konfigurálásával kapcsolatos információkat az [Információk a helyek közötti VPN Gateway-kapcsolatok VPN-eszközeiről][vpn-appliance] című cikkben találja.</span><span class="sxs-lookup"><span data-stu-id="3a008-127">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

- <span data-ttu-id="3a008-128">**VPN virtuális hálózati átjáró vagy ExpressRoute-átjáró**.</span><span class="sxs-lookup"><span data-stu-id="3a008-128">**VPN virtual network gateway or ExpressRoute gateway**.</span></span> <span data-ttu-id="3a008-129">A virtuális hálózati átjáró a helyszíni hálózattal létesített kapcsolathoz használt VPN-eszközhöz vagy az ExpressRoute-kapcsolatcsoporthoz való csatlakozást teszi lehetővé a virtuális hálózat számára.</span><span class="sxs-lookup"><span data-stu-id="3a008-129">The virtual network gateway enables the VNet to connect to the VPN device, or ExpressRoute circuit, used for connectivity with your on-premises network.</span></span> <span data-ttu-id="3a008-130">További információért tekintse át a [helyszíni hálózat és a Microsoft Azure Virtual Network csatlakoztatásával][connect-to-an-Azure-vnet] foglalkozó cikket.</span><span class="sxs-lookup"><span data-stu-id="3a008-130">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

> [!NOTE]
> <span data-ttu-id="3a008-131">A referenciaarchitektúra üzembe helyezési szkriptjei egy VPN-átjárót használnak a kapcsolat létrehozásához és egy virtuális hálózatot az Azure-ban a helyszíni hálózat szimulációjára.</span><span class="sxs-lookup"><span data-stu-id="3a008-131">The deployment scripts for this reference architecture use a VPN gateway for connectivity, and a VNet in Azure to simulate your on-premises network.</span></span>

- <span data-ttu-id="3a008-132">**Agyi virtuális hálózat**.</span><span class="sxs-lookup"><span data-stu-id="3a008-132">**Hub VNet**.</span></span> <span data-ttu-id="3a008-133">Egy Azure virtuális hálózat agyként a küllős hálózati topológiában.</span><span class="sxs-lookup"><span data-stu-id="3a008-133">Azure VNet used as the hub in the hub-spoke topology.</span></span> <span data-ttu-id="3a008-134">Az agy a helyszíni hálózathoz való csatlakozási lehetőségek központi helye, és a küllős hálózati topológiákban tárolt különféle számítási feladatok által felvett szolgáltatások tárolási helye.</span><span class="sxs-lookup"><span data-stu-id="3a008-134">The hub is the central point of connectivity to your on-premises network, and a place to host services that can be consumed by the different workloads hosted in the spoke VNets.</span></span>

- <span data-ttu-id="3a008-135">**Átjáró-alhálózat**.</span><span class="sxs-lookup"><span data-stu-id="3a008-135">**Gateway subnet**.</span></span> <span data-ttu-id="3a008-136">A virtuális hálózati átjárók ugyanazon az alhálózaton találhatók.</span><span class="sxs-lookup"><span data-stu-id="3a008-136">The virtual network gateways are held in the same subnet.</span></span>

- <span data-ttu-id="3a008-137">**Küllő virtuális hálózatok**.</span><span class="sxs-lookup"><span data-stu-id="3a008-137">**Spoke VNets**.</span></span> <span data-ttu-id="3a008-138">Egy vagy több Azure virtuális hálózat, amelyek küllőként használatosak a küllős topológiában.</span><span class="sxs-lookup"><span data-stu-id="3a008-138">One or more Azure VNets that are used as spokes in the hub-spoke topology.</span></span> <span data-ttu-id="3a008-139">A küllőkkel elszigetelhetőek a számítási feladatok saját virtuális hálózataikban, így más küllőktől elkülönülten kezelhetőek.</span><span class="sxs-lookup"><span data-stu-id="3a008-139">Spokes can be used to isolate workloads in their own VNets, managed separately from other spokes.</span></span> <span data-ttu-id="3a008-140">Minden számítási feladat több szintet tartalmazhat, amelyek alhálózatait Azure-terheléselosztók kapcsolják össze.</span><span class="sxs-lookup"><span data-stu-id="3a008-140">Each workload might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="3a008-141">További információ az alkalmazás-infrastruktúrával kapcsolatban: [Windows rendszerű virtuális gépek számítási feladatainak futtatása][windows-vm-ra] és [Számítási feladatok futtatása Linux rendszerű virtuális gépeken][linux-vm-ra].</span><span class="sxs-lookup"><span data-stu-id="3a008-141">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

- <span data-ttu-id="3a008-142">**Virtuális társhálózatok létesítése**.</span><span class="sxs-lookup"><span data-stu-id="3a008-142">**VNet peering**.</span></span> <span data-ttu-id="3a008-143">Két virtuális hálózat hitelesítéssel lehet csatlakozni egy [társviszonykapcsolat][vnet-peering].</span><span class="sxs-lookup"><span data-stu-id="3a008-143">Two VNets can be connected using a [peering connection][vnet-peering].</span></span> <span data-ttu-id="3a008-144">A társviszonykapcsolatok a virtuális hálózatok közötti nem tranzitív, alacsony késleltetésű kapcsolatok.</span><span class="sxs-lookup"><span data-stu-id="3a008-144">Peering connections are non-transitive, low latency connections between VNets.</span></span> <span data-ttu-id="3a008-145">A társviszonyba kapcsolt virtuális hálózatok az Azure gerinchálózatát használva, útválasztó igénye nélkül váltanak forgalmat egymás között.</span><span class="sxs-lookup"><span data-stu-id="3a008-145">Once peered, the VNets exchange traffic by using the Azure backbone, without the need for a router.</span></span> <span data-ttu-id="3a008-146">Egy küllős hálózati topológiában a virtuális társhálózatok létesítésével lehetséges az agyat minden egyes küllőhöz kapcsolni.</span><span class="sxs-lookup"><span data-stu-id="3a008-146">In a hub-spoke network topology, you use VNet peering to connect the hub to each spoke.</span></span> <span data-ttu-id="3a008-147">Az ugyanabban a régióban, vagy eltérő régiókban lévő virtuális hálózatok társviszonyt.</span><span class="sxs-lookup"><span data-stu-id="3a008-147">You can peer virtual networks in the same region, or different regions.</span></span> <span data-ttu-id="3a008-148">További információkért lásd: [-követelmények és korlátozások][vnet-peering-requirements].</span><span class="sxs-lookup"><span data-stu-id="3a008-148">For more information, see [Requirements and constraints][vnet-peering-requirements].</span></span>

> [!NOTE]
> <span data-ttu-id="3a008-149">Ez a cikk csak a [Resource Manager](/azure/azure-resource-manager/resource-group-overview) üzemelő példányokat mutatja be, de ugyanabban az előfizetésben klasszikus virtuális hálózatot is csatlakoztathat Resource Manager virtuális hálózathoz.</span><span class="sxs-lookup"><span data-stu-id="3a008-149">This article only covers [Resource Manager](/azure/azure-resource-manager/resource-group-overview) deployments, but you can also connect a classic VNet to a Resource Manager VNet in the same subscription.</span></span> <span data-ttu-id="3a008-150">Így a küllők tárolhatnak klasszikus üzemelő példányokat, mégis profitálhatnak az agyban megosztott szolgáltatásokból.</span><span class="sxs-lookup"><span data-stu-id="3a008-150">That way, your spokes can host classic deployments and still benefit from services shared in the hub.</span></span>

## <a name="recommendations"></a><span data-ttu-id="3a008-151">Javaslatok</span><span class="sxs-lookup"><span data-stu-id="3a008-151">Recommendations</span></span>

<span data-ttu-id="3a008-152">Az alábbi javaslatok a legtöbb forgatókönyvre vonatkoznak.</span><span class="sxs-lookup"><span data-stu-id="3a008-152">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="3a008-153">Kövesse ezeket a javaslatokat, ha nincsenek ezeket felülíró követelményei.</span><span class="sxs-lookup"><span data-stu-id="3a008-153">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="resource-groups"></a><span data-ttu-id="3a008-154">Erőforráscsoportok</span><span class="sxs-lookup"><span data-stu-id="3a008-154">Resource groups</span></span>

<span data-ttu-id="3a008-155">Az agyi virtuális hálózat, és minden küllő virtuális hálózat implementálható különböző erőforráscsoportokban és eltérő előfizetésekben is.</span><span class="sxs-lookup"><span data-stu-id="3a008-155">The hub VNet, and each spoke VNet, can be implemented in different resource groups, and even different subscriptions.</span></span> <span data-ttu-id="3a008-156">Különböző előfizetésekben található virtuális hálózatok társviszonyba állítása akkor, ha mindkét előfizetés társíthatók az azonos vagy eltérő Azure Active Directory-bérlő.</span><span class="sxs-lookup"><span data-stu-id="3a008-156">When you peer virtual networks in different subscriptions, both subscriptions can be associated to the same or different Azure Active Directory tenant.</span></span> <span data-ttu-id="3a008-157">Ez lehetővé teszi minden számítási feladat decentralizált felügyeletét, miközben a szolgáltatások megosztása változatlan marad az agyi virtuális hálózatban.</span><span class="sxs-lookup"><span data-stu-id="3a008-157">This allows for a decentralized management of each workload, while sharing services maintained in the hub VNet.</span></span>

### <a name="vnet-and-gatewaysubnet"></a><span data-ttu-id="3a008-158">Virtuális hálózat és GatewaySubnet</span><span class="sxs-lookup"><span data-stu-id="3a008-158">VNet and GatewaySubnet</span></span>

<span data-ttu-id="3a008-159">Hozzon létre egy *GatewaySubnet* nevű alhálózatot /27 címtartománnyal.</span><span class="sxs-lookup"><span data-stu-id="3a008-159">Create a subnet named *GatewaySubnet*, with an address range of /27.</span></span> <span data-ttu-id="3a008-160">Ez az alhálózat a virtuális hálózat átjárójához szükséges.</span><span class="sxs-lookup"><span data-stu-id="3a008-160">This subnet is required by the virtual network gateway.</span></span> <span data-ttu-id="3a008-161">Ha kioszt 32 címet erre az alhálózatra, azzal megelőzheti az átjáró mérethatárának elérését a jövőben.</span><span class="sxs-lookup"><span data-stu-id="3a008-161">Allocating 32 addresses to this subnet will help to prevent reaching gateway size limitations in the future.</span></span>

<span data-ttu-id="3a008-162">Az átjáró felállításáról további információkat az alábbi referenciaarchitektúrákban talál (a kapcsolattípustól függően):</span><span class="sxs-lookup"><span data-stu-id="3a008-162">For more information about setting up the gateway, see the following reference architectures, depending on your connection type:</span></span>

- <span data-ttu-id="3a008-163">[Hibrid hálózat ExpressRoute használatával][guidance-expressroute]</span><span class="sxs-lookup"><span data-stu-id="3a008-163">[Hybrid network using ExpressRoute][guidance-expressroute]</span></span>
- <span data-ttu-id="3a008-164">[Hibrid hálózat VPN-átjáró használatával][guidance-vpn]</span><span class="sxs-lookup"><span data-stu-id="3a008-164">[Hybrid network using a VPN gateway][guidance-vpn]</span></span>

<span data-ttu-id="3a008-165">A magasabb rendelkezésre álláshoz használhat ExpressRoute-ot és egy VPN-t feladatátvétel esetére.</span><span class="sxs-lookup"><span data-stu-id="3a008-165">For higher availability, you can use ExpressRoute plus a VPN for failover.</span></span> <span data-ttu-id="3a008-166">További információ: [Helyszíni hálózat csatlakoztatása az Azure-hoz VPN-feladatátvételt biztosító ExpressRoute használatával][hybrid-ha].</span><span class="sxs-lookup"><span data-stu-id="3a008-166">See [Connect an on-premises network to Azure using ExpressRoute with VPN failover][hybrid-ha].</span></span>

<span data-ttu-id="3a008-167">Küllős topológiát használhat átjáró nélkül is, ha nincs szükség a helyszíni hálózattal való kapcsolatra.</span><span class="sxs-lookup"><span data-stu-id="3a008-167">A hub-spoke topology can also be used without a gateway, if you don't need connectivity with your on-premises network.</span></span>

### <a name="vnet-peering"></a><span data-ttu-id="3a008-168">Társviszony létesítése virtuális hálózatok között</span><span class="sxs-lookup"><span data-stu-id="3a008-168">VNet peering</span></span>

<span data-ttu-id="3a008-169">A virtuális társhálózatok létesítése nem tranzitív kapcsolat két virtuális hálózat között.</span><span class="sxs-lookup"><span data-stu-id="3a008-169">VNet peering is a non-transitive relationship between two VNets.</span></span> <span data-ttu-id="3a008-170">Ha két küllőt szeretne egymáshoz csatlakoztatni, vegye fontolóra egy külön társviszonykapcsolat hozzáadását az adott küllők között.</span><span class="sxs-lookup"><span data-stu-id="3a008-170">If you require spokes to connect to each other, consider adding a separate peering connection between those spokes.</span></span>

<span data-ttu-id="3a008-171">Ha azonban több küllőt is szeretne egymáshoz csatlakoztatni, gyorsan el fognak fogyni a rendelkezésre álló társviszonykapcsolatok a [virtuális hálózatonként létesíthető társviszonykapcsolatok számának korlátozása][vnet-peering-limit] miatt.</span><span class="sxs-lookup"><span data-stu-id="3a008-171">However, if you have several spokes that need to connect with each other, you will run out of possible peering connections very quickly due to the [limitation on number of VNets peerings per VNet][vnet-peering-limit].</span></span> <span data-ttu-id="3a008-172">Ilyenkor vegye fontolóra a felhasználó által megadott útvonalak (UDR-ek) használatát. Ezekkel kényszerítheti, hogy az egy adott küllőre irányuló forgalmat a rendszer egy NVA-ra küldje, amely útválasztóként működik az agyi virtuális hálózaton.</span><span class="sxs-lookup"><span data-stu-id="3a008-172">In this scenario, consider using user defined routes (UDRs) to force traffic destined to a spoke to be sent to an NVA acting as a router at the hub VNet.</span></span> <span data-ttu-id="3a008-173">Ez lehetővé teszi, hogy a küllők kapcsolódjanak egymáshoz.</span><span class="sxs-lookup"><span data-stu-id="3a008-173">This will allow the spokes to connect to each other.</span></span>

<span data-ttu-id="3a008-174">Ezenkívül konfigurálhatja is a küllőket, hogy használják az agyi virtuális hálózatot a távoli hálózatokkal való kommunikációra.</span><span class="sxs-lookup"><span data-stu-id="3a008-174">You can also configure spokes to use the hub VNet gateway to communicate with remote networks.</span></span> <span data-ttu-id="3a008-175">Ahhoz, hogy az átjáróforgalom működjön a küllő és az agy között, a távoli hálózatok pedig kapcsolódjanak, a következők szükségesek:</span><span class="sxs-lookup"><span data-stu-id="3a008-175">To allow gateway traffic to flow from spoke to hub, and connect to remote networks, you must:</span></span>

- <span data-ttu-id="3a008-176">Konfigurálja a virtuális társhálózat-kapcsolatot az agyban az **átjáróforgalom engedélyezéséhez**.</span><span class="sxs-lookup"><span data-stu-id="3a008-176">Configure the VNet peering connection in the hub to **allow gateway transit**.</span></span>
- <span data-ttu-id="3a008-177">Konfigurálja a virtuális társhálózat-kapcsolatot minden egyes küllőben a **távoli átjárók** használatához.</span><span class="sxs-lookup"><span data-stu-id="3a008-177">Configure the VNet peering connection in each spoke to **use remote gateways**.</span></span>
- <span data-ttu-id="3a008-178">Konfiguráljon minden virtuális társhálózat-kapcsolatot a **továbbított forgalom engedélyezéséhez**.</span><span class="sxs-lookup"><span data-stu-id="3a008-178">Configure all VNet peering connections to **allow forwarded traffic**.</span></span>

## <a name="considerations"></a><span data-ttu-id="3a008-179">Megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="3a008-179">Considerations</span></span>

### <a name="spoke-connectivity"></a><span data-ttu-id="3a008-180">Küllőkapcsolat</span><span class="sxs-lookup"><span data-stu-id="3a008-180">Spoke connectivity</span></span>

<span data-ttu-id="3a008-181">Ha küllők között szeretne kapcsolatot létrehozni, fontolja meg egy NVA implementálását az agyban történő útválasztáshoz, illetve az UDR-ek használatát a küllőkben a forgalom továbbítása érdekében az agyhoz.</span><span class="sxs-lookup"><span data-stu-id="3a008-181">If you require connectivity between spokes, consider implementing an NVA for routing in the hub, and using UDRs in the spoke to forward traffic to the hub.</span></span>

<span data-ttu-id="3a008-182">![[2]][2]</span><span class="sxs-lookup"><span data-stu-id="3a008-182">![[2]][2]</span></span>

<span data-ttu-id="3a008-183">Ilyenkor konfigurálnia kell a társviszony-kapcsolatokat a **továbbított forgalom engedélyezéséhez**.</span><span class="sxs-lookup"><span data-stu-id="3a008-183">In this scenario, you must configure the peering connections to **allow forwarded traffic**.</span></span>

### <a name="overcoming-vnet-peering-limits"></a><span data-ttu-id="3a008-184">A virtuális társhálózatok létesítési korlátjának kiküszöbölése</span><span class="sxs-lookup"><span data-stu-id="3a008-184">Overcoming VNet peering limits</span></span>

<span data-ttu-id="3a008-185">Mindenképpen vegye figyelembe a [virtuális hálózatonként létesíthető társviszonyok számának korlátozását][vnet-peering-limit] az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="3a008-185">Make sure you consider the [limitation on number of VNets peerings per VNet][vnet-peering-limit] in Azure.</span></span> <span data-ttu-id="3a008-186">Ha több küllőre van szükség, mint amit a korlátozás megenged, fontolja meg egy agy-küllő-agy-küllő topológia létrehozását, ahol a küllők első szintje agyként is működik.</span><span class="sxs-lookup"><span data-stu-id="3a008-186">If you decide you need more spokes than the limit will allow, consider creating a hub-spoke-hub-spoke topology, where the first level of spokes also act as hubs.</span></span> <span data-ttu-id="3a008-187">Az alábbi diagram ezt a megközelítést mutatja be.</span><span class="sxs-lookup"><span data-stu-id="3a008-187">The following diagram shows this approach.</span></span>

<span data-ttu-id="3a008-188">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="3a008-188">![[3]][3]</span></span>

<span data-ttu-id="3a008-189">Ezenkívül vegye figyelembe, mely szolgáltatások vannak megosztva az agyban. Ezzel biztosítható, hogy az agy több küllőre is skálázzon.</span><span class="sxs-lookup"><span data-stu-id="3a008-189">Also consider what services are shared in the hub, to ensure the hub scales for a larger number of spokes.</span></span> <span data-ttu-id="3a008-190">Például ha az agy tűzfalszolgáltatásokat is nyújt, vegye figyelembe a tűzfal megoldás sávszélességi korlátozásait több küllő hozzáadása esetén.</span><span class="sxs-lookup"><span data-stu-id="3a008-190">For instance, if your hub provides firewall services, consider the bandwidth limits of your firewall solution when adding multiple spokes.</span></span> <span data-ttu-id="3a008-191">Lehet, hogy érdemes néhány megosztott szolgáltatást az agyak második szintjére mozgatni.</span><span class="sxs-lookup"><span data-stu-id="3a008-191">You might want to move some of these shared services to a second level of hubs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="3a008-192">A megoldás üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="3a008-192">Deploy the solution</span></span>

<span data-ttu-id="3a008-193">Ennek az architektúrának egy üzemelő példánya elérhető a [GitHubon][ref-arch-repo].</span><span class="sxs-lookup"><span data-stu-id="3a008-193">A deployment for this architecture is available on [GitHub][ref-arch-repo].</span></span> <span data-ttu-id="3a008-194">Használ a virtuális gépeket az egyes virtuális hálózatok kapcsolatának tesztelése.</span><span class="sxs-lookup"><span data-stu-id="3a008-194">It uses VMs in each VNet to test connectivity.</span></span> <span data-ttu-id="3a008-195">Az **agyi virtuális hálózat** **megosztott szolgáltatási** alhálózatában nincsenek tárolva tényleges szolgáltatások.</span><span class="sxs-lookup"><span data-stu-id="3a008-195">There are no actual services hosted in the **shared-services** subnet in the **hub VNet**.</span></span>

<span data-ttu-id="3a008-196">Az üzembe helyezés a következő erőforráscsoportokat hozza létre az előfizetésben:</span><span class="sxs-lookup"><span data-stu-id="3a008-196">The deployment creates the following resource groups in your subscription:</span></span>

- <span data-ttu-id="3a008-197">eseményközpont-nva-rg</span><span class="sxs-lookup"><span data-stu-id="3a008-197">hub-nva-rg</span></span>
- <span data-ttu-id="3a008-198">eseményközpont-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="3a008-198">hub-vnet-rg</span></span>
- <span data-ttu-id="3a008-199">rendszert-jb-rg</span><span class="sxs-lookup"><span data-stu-id="3a008-199">onprem-jb-rg</span></span>
- <span data-ttu-id="3a008-200">rendszert-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="3a008-200">onprem-vnet-rg</span></span>
- <span data-ttu-id="3a008-201">spoke1-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="3a008-201">spoke1-vnet-rg</span></span>
- <span data-ttu-id="3a008-202">spoke2-művele-rg</span><span class="sxs-lookup"><span data-stu-id="3a008-202">spoke2-vent-rg</span></span>

<span data-ttu-id="3a008-203">A sablon paraméterfájljai ezekre a nevekre hivatkoznak, ezért ha módosítja őket, a paraméterfájlokat is frissítse.</span><span class="sxs-lookup"><span data-stu-id="3a008-203">The template parameter files refer to these names, so if you change them, update the parameter files to match.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="3a008-204">Előfeltételek</span><span class="sxs-lookup"><span data-stu-id="3a008-204">Prerequisites</span></span>

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-the-simulated-on-premises-datacenter"></a><span data-ttu-id="3a008-205">A szimulált helyszíni adatközpont üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="3a008-205">Deploy the simulated on-premises datacenter</span></span>

<span data-ttu-id="3a008-206">A szimulált helyszíni adatközpont Azure virtuális hálózatként üzembe helyezéséhez kövesse az alábbi lépéseket:</span><span class="sxs-lookup"><span data-stu-id="3a008-206">To deploy the simulated on-premises datacenter as an Azure VNet, follow these steps:</span></span>

1. <span data-ttu-id="3a008-207">Keresse meg a `hybrid-networking/hub-spoke` mappába a referencia-architektúrák tárház.</span><span class="sxs-lookup"><span data-stu-id="3a008-207">Navigate to the `hybrid-networking/hub-spoke` folder of the reference architectures repository.</span></span>

2. <span data-ttu-id="3a008-208">Nyissa meg az `onprem.json` fájlt.</span><span class="sxs-lookup"><span data-stu-id="3a008-208">Open the `onprem.json` file.</span></span> <span data-ttu-id="3a008-209">Cserélje le a tartozó értékeket `adminUsername` és `adminPassword`.</span><span class="sxs-lookup"><span data-stu-id="3a008-209">Replace the values for `adminUsername` and `adminPassword`.</span></span>

    ```json
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

3. <span data-ttu-id="3a008-210">(Nem kötelező) Állítsa be a Linux-telepítéshez `osType` való `Linux`.</span><span class="sxs-lookup"><span data-stu-id="3a008-210">(Optional) For a Linux deployment, set `osType` to `Linux`.</span></span>

4. <span data-ttu-id="3a008-211">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="3a008-211">Run the following command:</span></span>

    ```bash
    azbb -s <subscription_id> -g onprem-vnet-rg -l <location> -p onprem.json --deploy
    ```

5. <span data-ttu-id="3a008-212">Várjon, amíg az üzembe helyezés befejeződik.</span><span class="sxs-lookup"><span data-stu-id="3a008-212">Wait for the deployment to finish.</span></span> <span data-ttu-id="3a008-213">Ehhez a központi telepítéshez létrehoz egy virtuális hálózatot, virtuális gép és egy VPN-átjárót.</span><span class="sxs-lookup"><span data-stu-id="3a008-213">This deployment creates a virtual network, a virtual machine, and a VPN gateway.</span></span> <span data-ttu-id="3a008-214">A VPN-átjáró létrehozása körülbelül 40 percet is igénybe vehet.</span><span class="sxs-lookup"><span data-stu-id="3a008-214">It can take about 40 minutes to create the VPN gateway.</span></span>

### <a name="deploy-the-hub-vnet"></a><span data-ttu-id="3a008-215">Az agyi virtuális hálózat üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="3a008-215">Deploy the hub VNet</span></span>

<span data-ttu-id="3a008-216">Az agyi virtuális hálózat üzembe helyezéséhez hajtsa végre az alábbi lépéseket.</span><span class="sxs-lookup"><span data-stu-id="3a008-216">To deploy the hub VNet, perform the following steps.</span></span>

1. <span data-ttu-id="3a008-217">Nyissa meg az `hub-vnet.json` fájlt.</span><span class="sxs-lookup"><span data-stu-id="3a008-217">Open the `hub-vnet.json` file.</span></span> <span data-ttu-id="3a008-218">Cserélje le a tartozó értékeket `adminUsername` és `adminPassword`.</span><span class="sxs-lookup"><span data-stu-id="3a008-218">Replace the values for `adminUsername` and `adminPassword`.</span></span>

    ```json
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. <span data-ttu-id="3a008-219">(Nem kötelező) Állítsa be a Linux-telepítéshez `osType` való `Linux`.</span><span class="sxs-lookup"><span data-stu-id="3a008-219">(Optional) For a Linux deployment, set `osType` to `Linux`.</span></span>

3. <span data-ttu-id="3a008-220">Keresse meg a mindkét példányát `sharedKey` , és adjon meg egy megosztott kulcsot a VPN-kapcsolat.</span><span class="sxs-lookup"><span data-stu-id="3a008-220">Find both instances of `sharedKey` and enter a shared key for the VPN connection.</span></span> <span data-ttu-id="3a008-221">Az értékeknek meg kell egyezniük.</span><span class="sxs-lookup"><span data-stu-id="3a008-221">The values must match.</span></span>

    ```json
    "sharedKey": "",
    ```

4. <span data-ttu-id="3a008-222">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="3a008-222">Run the following command:</span></span>

    ```bash
    azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet.json --deploy
    ```

5. <span data-ttu-id="3a008-223">Várjon, amíg az üzembe helyezés befejeződik.</span><span class="sxs-lookup"><span data-stu-id="3a008-223">Wait for the deployment to finish.</span></span> <span data-ttu-id="3a008-224">A központi telepítés egy virtuális hálózatot, egy virtuális gépet, a VPN-átjáró és az átjáró kapcsolatot hoz létre.</span><span class="sxs-lookup"><span data-stu-id="3a008-224">This deployment creates a virtual network, a virtual machine, a VPN gateway, and a connection to the gateway.</span></span>  <span data-ttu-id="3a008-225">A VPN-átjáró létrehozása körülbelül 40 percet is igénybe vehet.</span><span class="sxs-lookup"><span data-stu-id="3a008-225">It can take about 40 minutes to create the VPN gateway.</span></span>

### <a name="test-connectivity-to-the-hub-vnet-mdash-windows-deployment"></a><span data-ttu-id="3a008-226">Az agyi virtuális hálózat csatlakozni &mdash; Windows deployment</span><span class="sxs-lookup"><span data-stu-id="3a008-226">Test connectivity to the hub VNet &mdash; Windows deployment</span></span>

<span data-ttu-id="3a008-227">Az agyi virtuális hálózat Windows virtuális gépeket használ, a szimulált helyszíni környezetből conectivity teszteléséhez, kövesse az alábbi lépéseket:</span><span class="sxs-lookup"><span data-stu-id="3a008-227">To test conectivity from the simulated on-premises environment to the hub VNet using Windows VMs, follow these steps:</span></span>

1. <span data-ttu-id="3a008-228">Az Azure Portal használatával keresse meg a `jb-vm1` nevű virtuális gépet az `onprem-jb-rg` erőforráscsoportban.</span><span class="sxs-lookup"><span data-stu-id="3a008-228">Use the Azure portal to find the VM named `jb-vm1` in the `onprem-jb-rg` resource group.</span></span>

2. <span data-ttu-id="3a008-229">Kattintson a `Connect` parancsra egy, a virtuális gépre irányuló távoli asztali munkamenet megnyitásához.</span><span class="sxs-lookup"><span data-stu-id="3a008-229">Click `Connect` to open a remote desktop session to the VM.</span></span> <span data-ttu-id="3a008-230">Használja az `onprem.json` paraméterfájlban megadott jelszót.</span><span class="sxs-lookup"><span data-stu-id="3a008-230">Use the password that you specified in the `onprem.json` parameter file.</span></span>

3. <span data-ttu-id="3a008-231">Nyisson meg egy PowerShell-konzolt a virtuális gépen, és a `Test-NetConnection` parancsmaggal győződjön meg arról, hogy képes-e csatlakozni a jumpbox virtuális Géphez az agyi virtuális hálózat.</span><span class="sxs-lookup"><span data-stu-id="3a008-231">Open a PowerShell console in the VM, and use the `Test-NetConnection` cmdlet to verify that you can connect to the jumpbox VM in the hub VNet.</span></span>

   ```powershell
   Test-NetConnection 10.0.0.68 -CommonTCPPort RDP
   ```

<span data-ttu-id="3a008-232">A kimenetnek a következőképpen kell kinéznie:</span><span class="sxs-lookup"><span data-stu-id="3a008-232">The output should look similar to the following:</span></span>

```powershell
ComputerName     : 10.0.0.68
RemoteAddress    : 10.0.0.68
RemotePort       : 3389
InterfaceAlias   : Ethernet 2
SourceAddress    : 192.168.1.000
TcpTestSucceeded : True
```

> [!NOTE]
> <span data-ttu-id="3a008-233">Alapértelmezés szerint Windows Serveres virtuális gépek nem teszik lehetővé az ICMP-válaszok az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="3a008-233">By default, Windows Server VMs do not allow ICMP responses in Azure.</span></span> <span data-ttu-id="3a008-234">Ha a használni kívánt `ping` kapcsolat teszteléséhez, engedélyeznie kell a fokozott biztonságú Windows tűzfal az ICMP-forgalmat az egyes virtuális Gépekhez.</span><span class="sxs-lookup"><span data-stu-id="3a008-234">If you want to use `ping` to test connectivity, you need to enable ICMP traffic in the Windows Advanced Firewall for each VM.</span></span>

### <a name="test-connectivity-to-the-hub-vnet-mdash-linux-deployment"></a><span data-ttu-id="3a008-235">Az agyi virtuális hálózat csatlakozni &mdash; Linux üzembe helyezés</span><span class="sxs-lookup"><span data-stu-id="3a008-235">Test connectivity to the hub VNet &mdash; Linux deployment</span></span>

<span data-ttu-id="3a008-236">Az agyi virtuális hálózat használata Linux rendszerű virtuális gépek, a szimulált helyszíni környezetből conectivity teszteléséhez, kövesse az alábbi lépéseket:</span><span class="sxs-lookup"><span data-stu-id="3a008-236">To test conectivity from the simulated on-premises environment to the hub VNet using Linux VMs, follow these steps:</span></span>

1. <span data-ttu-id="3a008-237">Az Azure Portal használatával keresse meg a `jb-vm1` nevű virtuális gépet az `onprem-jb-rg` erőforráscsoportban.</span><span class="sxs-lookup"><span data-stu-id="3a008-237">Use the Azure portal to find the VM named `jb-vm1` in the `onprem-jb-rg` resource group.</span></span>

2. <span data-ttu-id="3a008-238">Kattintson a `Connect` , és másolja a `ssh` parancs jelenik meg a portálon.</span><span class="sxs-lookup"><span data-stu-id="3a008-238">Click `Connect` and copy the `ssh` command shown in the portal.</span></span>

3. <span data-ttu-id="3a008-239">A Linux. Ehhez futtassa `ssh` csatlakozhat a szimulált helyszíni környezetet.</span><span class="sxs-lookup"><span data-stu-id="3a008-239">From a Linux prompt, run `ssh` to connect to the simulated on-premises environment.</span></span> <span data-ttu-id="3a008-240">Használja az `onprem.json` paraméterfájlban megadott jelszót.</span><span class="sxs-lookup"><span data-stu-id="3a008-240">Use the password that you specified in the `onprem.json` parameter file.</span></span>

4. <span data-ttu-id="3a008-241">Használja a `ping` parancsot az agyi virtuális hálózat a jumpbox virtuális Géphez való kapcsolódás tesztelése:</span><span class="sxs-lookup"><span data-stu-id="3a008-241">Use the `ping` command to test connectivity to the jumpbox VM in the hub VNet:</span></span>

   ```shell
   ping 10.0.0.68
   ```

### <a name="deploy-the-spoke-vnets"></a><span data-ttu-id="3a008-242">A küllő virtuális hálózatok üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="3a008-242">Deploy the spoke VNets</span></span>

<span data-ttu-id="3a008-243">A küllő virtuális hálózatok üzembe helyezéséhez hajtsa végre az alábbi lépéseket.</span><span class="sxs-lookup"><span data-stu-id="3a008-243">To deploy the spoke VNets, perform the following steps.</span></span>

1. <span data-ttu-id="3a008-244">Nyissa meg az `spoke1.json` fájlt.</span><span class="sxs-lookup"><span data-stu-id="3a008-244">Open the `spoke1.json` file.</span></span> <span data-ttu-id="3a008-245">Cserélje le a tartozó értékeket `adminUsername` és `adminPassword`.</span><span class="sxs-lookup"><span data-stu-id="3a008-245">Replace the values for `adminUsername` and `adminPassword`.</span></span>

    ```json
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. <span data-ttu-id="3a008-246">(Nem kötelező) Állítsa be a Linux-telepítéshez `osType` való `Linux`.</span><span class="sxs-lookup"><span data-stu-id="3a008-246">(Optional) For a Linux deployment, set `osType` to `Linux`.</span></span>

3. <span data-ttu-id="3a008-247">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="3a008-247">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke1-vnet-rg -l <location> -p spoke1.json --deploy
   ```

4. <span data-ttu-id="3a008-248">Ismételje meg az 1 – 2. lépéseket a `spoke2.json` fájlt.</span><span class="sxs-lookup"><span data-stu-id="3a008-248">Repeat steps 1-2 for the `spoke2.json` file.</span></span>

5. <span data-ttu-id="3a008-249">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="3a008-249">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke2-vnet-rg -l <location> -p spoke2.json --deploy
   ```

6. <span data-ttu-id="3a008-250">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="3a008-250">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet-peering.json --deploy
   ```

### <a name="test-connectivity-to-the-spoke-vnets-mdash-windows-deployment"></a><span data-ttu-id="3a008-251">A küllő virtuális hálózatokhoz csatlakozni &mdash; Windows deployment</span><span class="sxs-lookup"><span data-stu-id="3a008-251">Test connectivity to the spoke VNets &mdash; Windows deployment</span></span>

<span data-ttu-id="3a008-252">A küllő virtuális hálózatokhoz, Windows virtuális gépek használata a szimulált helyszíni környezetből conectivity teszteléséhez hajtsa végre az alábbi lépéseket:</span><span class="sxs-lookup"><span data-stu-id="3a008-252">To test conectivity from the simulated on-premises environment to the spoke VNets using Windows VMs, perform the following steps:</span></span>

1. <span data-ttu-id="3a008-253">Az Azure Portal használatával keresse meg a `jb-vm1` nevű virtuális gépet az `onprem-jb-rg` erőforráscsoportban.</span><span class="sxs-lookup"><span data-stu-id="3a008-253">Use the Azure portal to find the VM named `jb-vm1` in the `onprem-jb-rg` resource group.</span></span>

2. <span data-ttu-id="3a008-254">Kattintson a `Connect` parancsra egy, a virtuális gépre irányuló távoli asztali munkamenet megnyitásához.</span><span class="sxs-lookup"><span data-stu-id="3a008-254">Click `Connect` to open a remote desktop session to the VM.</span></span> <span data-ttu-id="3a008-255">Használja az `onprem.json` paraméterfájlban megadott jelszót.</span><span class="sxs-lookup"><span data-stu-id="3a008-255">Use the password that you specified in the `onprem.json` parameter file.</span></span>

3. <span data-ttu-id="3a008-256">Nyisson meg egy PowerShell-konzolt a virtuális gépen, és a `Test-NetConnection` parancsmaggal győződjön meg arról, hogy képes-e csatlakozni a jumpbox virtuális gépeket a küllő virtuális hálózatok a.</span><span class="sxs-lookup"><span data-stu-id="3a008-256">Open a PowerShell console in the VM, and use the `Test-NetConnection` cmdlet to verify that you can connect to the jumpbox VMs in the spoke VNets.</span></span>

   ```powershell
   Test-NetConnection 10.1.0.68 -CommonTCPPort RDP
   Test-NetConnection 10.2.0.68 -CommonTCPPort RDP
   ```

### <a name="test-connectivity-to-the-spoke-vnets-mdash-linux-deployment"></a><span data-ttu-id="3a008-257">A küllő virtuális hálózatokhoz csatlakozni &mdash; Linux üzembe helyezés</span><span class="sxs-lookup"><span data-stu-id="3a008-257">Test connectivity to the spoke VNets &mdash; Linux deployment</span></span>

<span data-ttu-id="3a008-258">A küllő virtuális hálózatokhoz, Linux rendszerű virtuális gépek használata a szimulált helyszíni környezetből conectivity teszteléséhez hajtsa végre az alábbi lépéseket:</span><span class="sxs-lookup"><span data-stu-id="3a008-258">To test conectivity from the simulated on-premises environment to the spoke VNets using Linux VMs, perform the following steps:</span></span>

1. <span data-ttu-id="3a008-259">Az Azure Portal használatával keresse meg a `jb-vm1` nevű virtuális gépet az `onprem-jb-rg` erőforráscsoportban.</span><span class="sxs-lookup"><span data-stu-id="3a008-259">Use the Azure portal to find the VM named `jb-vm1` in the `onprem-jb-rg` resource group.</span></span>

2. <span data-ttu-id="3a008-260">Kattintson a `Connect` , és másolja a `ssh` parancs jelenik meg a portálon.</span><span class="sxs-lookup"><span data-stu-id="3a008-260">Click `Connect` and copy the `ssh` command shown in the portal.</span></span>

3. <span data-ttu-id="3a008-261">A Linux. Ehhez futtassa `ssh` csatlakozhat a szimulált helyszíni környezetet.</span><span class="sxs-lookup"><span data-stu-id="3a008-261">From a Linux prompt, run `ssh` to connect to the simulated on-premises environment.</span></span> <span data-ttu-id="3a008-262">Használja az `onprem.json` paraméterfájlban megadott jelszót.</span><span class="sxs-lookup"><span data-stu-id="3a008-262">Use the password that you specified in the `onprem.json` parameter file.</span></span>

4. <span data-ttu-id="3a008-263">Használja a `ping` parancsot minden egyes küllőben a jumpbox virtuális gépek kapcsolatának teszteléséhez:</span><span class="sxs-lookup"><span data-stu-id="3a008-263">Use the `ping` command to test connectivity to the jumpbox VMs in each spoke:</span></span>

   ```bash
   ping 10.1.0.68
   ping 10.2.0.68
   ```

### <a name="add-connectivity-between-spokes"></a><span data-ttu-id="3a008-264">Kapcsolat hozzáadása küllők között</span><span class="sxs-lookup"><span data-stu-id="3a008-264">Add connectivity between spokes</span></span>

<span data-ttu-id="3a008-265">Ez a lépés nem kötelező.</span><span class="sxs-lookup"><span data-stu-id="3a008-265">This step is optional.</span></span> <span data-ttu-id="3a008-266">Ha azt szeretné, hogy a küllők kapcsolódjanak egymáshoz, használja az agyi virtuális hálózat útválasztó egy hálózati virtuális készüléket (NVA), és az kényszerít ki forgalmat a küllők az útválasztó egy másik küllőhöz kapcsolódási kísérlet során.</span><span class="sxs-lookup"><span data-stu-id="3a008-266">If you want to allow spokes to connect to each other, you must use a network virtual appliance (NVA) as a router in the hub VNet, and force traffic from spokes to the router when trying to connect to another spoke.</span></span> <span data-ttu-id="3a008-267">Egy alapszintű példa nva-t, mint egyetlen virtuális gép üzembe helyezéséhez a felhasználó által megadott útvonalak (udr-EK), hogy a kettő együtt küllő a virtuális hálózatokhoz való csatlakozás, hajtsa végre az alábbi lépéseket:</span><span class="sxs-lookup"><span data-stu-id="3a008-267">To deploy a basic sample NVA as a single VM, along with user-defined routes (UDRs) to allow the two spoke VNets to connect, perform the following steps:</span></span>

1. <span data-ttu-id="3a008-268">Nyissa meg az `hub-nva.json` fájlt.</span><span class="sxs-lookup"><span data-stu-id="3a008-268">Open the `hub-nva.json` file.</span></span> <span data-ttu-id="3a008-269">Cserélje le a tartozó értékeket `adminUsername` és `adminPassword`.</span><span class="sxs-lookup"><span data-stu-id="3a008-269">Replace the values for `adminUsername` and `adminPassword`.</span></span>

    ```json
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. <span data-ttu-id="3a008-270">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="3a008-270">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-nva-rg -l <location> -p hub-nva.json --deploy
   ```

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[best-practices-security]: /azure/best-practices-network-securit
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[guidance-expressroute]: ./expressroute.md
[guidance-vpn]: ./vpn.md
[linux-vm-ra]: ../virtual-machines-linux/index.md
[hybrid-ha]: ./expressroute-vpn-failover.md
[naming conventions]: /azure/guidance/guidance-naming-conventions
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[vnet-peering]: /azure/virtual-network/virtual-network-peering-overview
[vnet-peering-limit]: /azure/azure-subscription-service-limits#networking-limits
[vnet-peering-requirements]: /azure/virtual-network/virtual-network-manage-peering#requirements-and-constraints
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md
[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures

[0]: ./images/hub-spoke.png "Küllős topológia az Azure-ban"
[1]: ./images/hub-spoke-gateway-routing.svg "Küllős topológia az Azure-ban tranzitív útválasztással"
[2]: ./images/hub-spoke-no-gateway-routing.svg "Küllős topológia az Azure-ban tranzitív útválasztással, NVA használatával"
[3]: ./images/hub-spokehub-spoke.svg "Küllős-küllős topológia az Azure-ban"
