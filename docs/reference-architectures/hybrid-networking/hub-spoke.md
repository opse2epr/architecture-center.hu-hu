---
title: "Küllős hálózati topológia implementálása az Azure-ban"
description: "Küllős hálózati topológia implementálása az Azure-ban."
author: telmosampaio
ms.date: 02/23/2018
pnp.series.title: Implement a hub-spoke network topology in Azure
pnp.series.prev: expressroute
ms.openlocfilehash: 1a2855f0d4a903fc4d7a022aef20ea73fe763e2c
ms.sourcegitcommit: 2123c25b1a0b5501ff1887f98030787191cf6994
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/08/2018
---
# <a name="implement-a-hub-spoke-network-topology-in-azure"></a><span data-ttu-id="4a471-103">Küllős hálózati topológia implementálása az Azure-ban</span><span class="sxs-lookup"><span data-stu-id="4a471-103">Implement a hub-spoke network topology in Azure</span></span>

<span data-ttu-id="4a471-104">Ez a referenciaarchitektúra bemutatja, hogyan lehetséges a küllős hálózati topológia implementálása az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="4a471-104">This reference architecture shows how to implement a hub-spoke topology in Azure.</span></span> <span data-ttu-id="4a471-105">Az *agy* egy virtuális hálózat (VNet) az Azure-ban, amely központi kapcsolódási pontként szolgál a helyszíni hálózathoz.</span><span class="sxs-lookup"><span data-stu-id="4a471-105">The *hub* is a virtual network (VNet) in Azure that acts as a central point of connectivity to your on-premises network.</span></span> <span data-ttu-id="4a471-106">A *küllők* az agyhoz kapcsolódó virtuális hálózatok, és a számítási feladatok elkülönítésére használhatók.</span><span class="sxs-lookup"><span data-stu-id="4a471-106">The *spokes* are VNets that peer with the hub, and can be used to isolate workloads.</span></span> <span data-ttu-id="4a471-107">A helyszíni adatközpont és az agy közötti forgalom egy ExpressRoute- vagy VPN-átjárókapcsolaton keresztül halad át.</span><span class="sxs-lookup"><span data-stu-id="4a471-107">Traffic flows between the on-premises datacenter and the hub through an ExpressRoute or VPN gateway connection.</span></span>  <span data-ttu-id="4a471-108">[**A megoldás üzembe helyezése.**](#deploy-the-solution)</span><span class="sxs-lookup"><span data-stu-id="4a471-108">[**Deploy this solution**](#deploy-the-solution).</span></span>

<span data-ttu-id="4a471-109">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="4a471-109">![[0]][0]</span></span>

<span data-ttu-id="4a471-110">*Töltse le az ][visio-download]architektúra [Visio-fájlját*</span><span class="sxs-lookup"><span data-stu-id="4a471-110">*Download a [Visio file][visio-download] of this architecture*</span></span>


<span data-ttu-id="4a471-111">A topológia előnyei a következők:</span><span class="sxs-lookup"><span data-stu-id="4a471-111">The benefits of this toplogy include:</span></span>

* <span data-ttu-id="4a471-112">**Költségmegtakarítás** a szolgáltatások központosításával, amelyeket több számítási feladat között, egyetlen helyen lehet elosztani, például a hálózati virtuális berendezések (NVA-k) és DNS-kiszolgálók.</span><span class="sxs-lookup"><span data-stu-id="4a471-112">**Cost savings** by centralizing services that can be shared by multiple workloads, such as network virtual appliances (NVAs) and DNS servers, in a single location.</span></span>
* <span data-ttu-id="4a471-113">**Az előfizetési korlátok leküzdése** a különböző előfizetésekből virtuális társhálózatok létesítésével a központi agyhoz.</span><span class="sxs-lookup"><span data-stu-id="4a471-113">**Overcome subscriptions limits** by peering VNets from different subscriptions to the central hub.</span></span>
* <span data-ttu-id="4a471-114">**Kockázatok elkülönítése** a központi IT (SecOps, InfraOps) és a számítási feladatok között (DevOps).</span><span class="sxs-lookup"><span data-stu-id="4a471-114">**Separation of concerns** between central IT (SecOps, InfraOps) and workloads (DevOps).</span></span>

<span data-ttu-id="4a471-115">Az architektúra gyakori használati módjai többek között a következők:</span><span class="sxs-lookup"><span data-stu-id="4a471-115">Typical uses for this architecture include:</span></span>

* <span data-ttu-id="4a471-116">A különböző környezetekben (például a fejlesztés, a tesztelés és a gyártás) üzembe helyezett számítási feladatok, amelyek megosztott szolgáltatásokat igényelnek (például DNS, IDS, NTP vagy AD DS).</span><span class="sxs-lookup"><span data-stu-id="4a471-116">Workloads deployed in different environments, such as development, testing, and production, that require shared services such as DNS, IDS, NTP, or AD DS.</span></span> <span data-ttu-id="4a471-117">A megosztott szolgáltatások az agyi virtuális hálózatban találhatóak, azonban minden környezet egy küllőn van üzembe helyezve, így megmarad az elszigetelés.</span><span class="sxs-lookup"><span data-stu-id="4a471-117">Shared services are placed in the hub VNet, while each environment is deployed to a spoke to maintain isolation.</span></span>
* <span data-ttu-id="4a471-118">Olyan számítási feladatok, amelyek nem igénylik az egymással való kapcsolatot, de igénylik a megosztott szolgáltatásokhoz való hozzáférést.</span><span class="sxs-lookup"><span data-stu-id="4a471-118">Workloads that do not require connectivity to each other, but require access to shared services.</span></span>
* <span data-ttu-id="4a471-119">Olyan vállalatok, amelyek központi felügyeletet igényelnek a biztonsági szempontokhoz, például egy tűzfal az agyban DMZ-ként, és küllőnkénti elkülönített felügyelet a számítási feladatokhoz.</span><span class="sxs-lookup"><span data-stu-id="4a471-119">Enterprises that require central control over security aspects, such as a firewall in the hub as a DMZ, and segregated management for the workloads in each spoke.</span></span>

## <a name="architecture"></a><span data-ttu-id="4a471-120">Architektúra</span><span class="sxs-lookup"><span data-stu-id="4a471-120">Architecture</span></span>

<span data-ttu-id="4a471-121">Az architektúra a következőkben leírt összetevőkből áll.</span><span class="sxs-lookup"><span data-stu-id="4a471-121">The architecture consists of the following components.</span></span>

* <span data-ttu-id="4a471-122">**Helyszíni hálózat**.</span><span class="sxs-lookup"><span data-stu-id="4a471-122">**On-premises network**.</span></span> <span data-ttu-id="4a471-123">A cégen belül futó helyi magánhálózat.</span><span class="sxs-lookup"><span data-stu-id="4a471-123">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="4a471-124">**VPN-eszköz**.</span><span class="sxs-lookup"><span data-stu-id="4a471-124">**VPN device**.</span></span> <span data-ttu-id="4a471-125">A helyszíni hálózat számára külső kapcsolatot biztosító eszköz vagy szolgáltatás.</span><span class="sxs-lookup"><span data-stu-id="4a471-125">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="4a471-126">A VPN-eszköz lehet hardvereszköz vagy valamilyen szoftvermegoldás, amilyen például a Windows Server 2012 Útválasztás és távelérés szolgáltatása (Routing and Remote Access Service, RRAS).</span><span class="sxs-lookup"><span data-stu-id="4a471-126">The VPN device may be a hardware device, or a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="4a471-127">A támogatott VPN-berendezések listáját és az egyes VPN-berendezések Azure-hoz való csatlakoztatásra történő konfigurálásával kapcsolatos információkat az [Információk a helyek közötti VPN Gateway-kapcsolatok VPN-eszközeiről][vpn-appliance] című cikkben találja.</span><span class="sxs-lookup"><span data-stu-id="4a471-127">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="4a471-128">**VPN virtuális hálózati átjáró vagy ExpressRoute-átjáró**.</span><span class="sxs-lookup"><span data-stu-id="4a471-128">**VPN virtual network gateway or ExpressRoute gateway**.</span></span> <span data-ttu-id="4a471-129">A virtuális hálózati átjáró a helyszíni hálózattal létesített kapcsolathoz használt VPN-eszközhöz vagy az ExpressRoute-kapcsolatcsoporthoz való csatlakozást teszi lehetővé a virtuális hálózat számára.</span><span class="sxs-lookup"><span data-stu-id="4a471-129">The virtual network gateway enables the VNet to connect to the VPN device, or ExpressRoute circuit, used for connectivity with your on-premises network.</span></span> <span data-ttu-id="4a471-130">További információért tekintse át a [helyszíni hálózat és a Microsoft Azure Virtual Network csatlakoztatásával][connect-to-an-Azure-vnet] foglalkozó cikket.</span><span class="sxs-lookup"><span data-stu-id="4a471-130">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

> [!NOTE]
> <span data-ttu-id="4a471-131">A referenciaarchitektúra üzembe helyezési szkriptjei egy VPN-átjárót használnak a kapcsolat létrehozásához és egy virtuális hálózatot az Azure-ban a helyszíni hálózat szimulációjára.</span><span class="sxs-lookup"><span data-stu-id="4a471-131">The deployment scripts for this reference architecture use a VPN gateway for connectivity, and a VNet in Azure to simulate your on-premises network.</span></span>

* <span data-ttu-id="4a471-132">**Agyi virtuális hálózat**.</span><span class="sxs-lookup"><span data-stu-id="4a471-132">**Hub VNet**.</span></span> <span data-ttu-id="4a471-133">Egy Azure virtuális hálózat agyként a küllős hálózati topológiában.</span><span class="sxs-lookup"><span data-stu-id="4a471-133">Azure VNet used as the hub in the hub-spoke topology.</span></span> <span data-ttu-id="4a471-134">Az agy a helyszíni hálózathoz való csatlakozási lehetőségek központi helye, és a küllős hálózati topológiákban tárolt különféle számítási feladatok által felvett szolgáltatások tárolási helye.</span><span class="sxs-lookup"><span data-stu-id="4a471-134">The hub is the central point of connectivity to your on-premises network, and a place to host services that can be consumed by the different workloads hosted in the spoke VNets.</span></span>

* <span data-ttu-id="4a471-135">**Átjáró-alhálózat**.</span><span class="sxs-lookup"><span data-stu-id="4a471-135">**Gateway subnet**.</span></span> <span data-ttu-id="4a471-136">A virtuális hálózati átjárók ugyanazon az alhálózaton találhatók.</span><span class="sxs-lookup"><span data-stu-id="4a471-136">The virtual network gateways are held in the same subnet.</span></span>

* <span data-ttu-id="4a471-137">**Küllő virtuális hálózatok**.</span><span class="sxs-lookup"><span data-stu-id="4a471-137">**Spoke VNets**.</span></span> <span data-ttu-id="4a471-138">Egy vagy több Azure virtuális hálózat, amelyek küllőként használatosak a küllős topológiában.</span><span class="sxs-lookup"><span data-stu-id="4a471-138">One or more Azure VNets that are used as spokes in the hub-spoke topology.</span></span> <span data-ttu-id="4a471-139">A küllőkkel elszigetelhetőek a számítási feladatok saját virtuális hálózataikban, így más küllőktől elkülönülten kezelhetőek.</span><span class="sxs-lookup"><span data-stu-id="4a471-139">Spokes can be used to isolate workloads in their own VNets, managed separately from other spokes.</span></span> <span data-ttu-id="4a471-140">Minden számítási feladat több szintet tartalmazhat, amelyek alhálózatait Azure-terheléselosztók kapcsolják össze.</span><span class="sxs-lookup"><span data-stu-id="4a471-140">Each workload might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="4a471-141">További információ az alkalmazás-infrastruktúrával kapcsolatban: [Windows rendszerű virtuális gépek számítási feladatainak futtatása][windows-vm-ra] és [Számítási feladatok futtatása Linux rendszerű virtuális gépeken][linux-vm-ra].</span><span class="sxs-lookup"><span data-stu-id="4a471-141">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

* <span data-ttu-id="4a471-142">**Virtuális társhálózatok létesítése**.</span><span class="sxs-lookup"><span data-stu-id="4a471-142">**VNet peering**.</span></span> <span data-ttu-id="4a471-143">Két, ugyanabban az Azure-régióban található virtuális hálózatot egy [társviszonykapcsolat][vnet-peering] használatával lehet összekapcsolni.</span><span class="sxs-lookup"><span data-stu-id="4a471-143">Two VNets in the same Azure region can be connected using a [peering connection][vnet-peering].</span></span> <span data-ttu-id="4a471-144">A társviszonykapcsolatok a virtuális hálózatok közötti nem tranzitív, alacsony késleltetésű kapcsolatok.</span><span class="sxs-lookup"><span data-stu-id="4a471-144">Peering connections are non-transitive, low latency connections between VNets.</span></span> <span data-ttu-id="4a471-145">A társviszonyba kapcsolt virtuális hálózatok az Azure gerinchálózatát használva, útválasztó igénye nélkül váltanak forgalmat egymás között.</span><span class="sxs-lookup"><span data-stu-id="4a471-145">Once peered, the VNets exchange traffic by using the Azure backbone, without the need for a router.</span></span> <span data-ttu-id="4a471-146">Egy küllős hálózati topológiában a virtuális társhálózatok létesítésével lehetséges az agyat minden egyes küllőhöz kapcsolni.</span><span class="sxs-lookup"><span data-stu-id="4a471-146">In a hub-spoke network topology, you use VNet peering to connect the hub to each spoke.</span></span>

> [!NOTE]
> <span data-ttu-id="4a471-147">Ez a cikk csak a [Resource Manager](/azure/azure-resource-manager/resource-group-overview) üzemelő példányokat mutatja be, de ugyanabban az előfizetésben klasszikus virtuális hálózatot is csatlakoztathat Resource Manager virtuális hálózathoz.</span><span class="sxs-lookup"><span data-stu-id="4a471-147">This article only covers [Resource Manager](/azure/azure-resource-manager/resource-group-overview) deployments, but you can also connect a classic VNet to a Resource Manager VNet in the same subscription.</span></span> <span data-ttu-id="4a471-148">Így a küllők tárolhatnak klasszikus üzemelő példányokat, mégis profitálhatnak az agyban megosztott szolgáltatásokból.</span><span class="sxs-lookup"><span data-stu-id="4a471-148">That way, your spokes can host classic deployments and still benefit from services shared in the hub.</span></span>

## <a name="recommendations"></a><span data-ttu-id="4a471-149">Javaslatok</span><span class="sxs-lookup"><span data-stu-id="4a471-149">Recommendations</span></span>

<span data-ttu-id="4a471-150">Az alábbi javaslatok a legtöbb forgatókönyvre vonatkoznak.</span><span class="sxs-lookup"><span data-stu-id="4a471-150">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="4a471-151">Kövesse ezeket a javaslatokat, ha nincsenek ezeket felülíró követelményei.</span><span class="sxs-lookup"><span data-stu-id="4a471-151">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="resource-groups"></a><span data-ttu-id="4a471-152">Erőforráscsoportok</span><span class="sxs-lookup"><span data-stu-id="4a471-152">Resource groups</span></span>

<span data-ttu-id="4a471-153">Az agyi és minden küllő virtuális hálózat implementálható különböző erőforráscsoportokban és eltérő előfizetésekben is, ha mind ugyanahhoz az Azure Active Directory (Azure AD) bérlőhöz tartozik, ugyanabban az Azure-régióban.</span><span class="sxs-lookup"><span data-stu-id="4a471-153">The hub VNet, and each spoke VNet, can be implemented in different resource groups, and even different subscriptions, as long as they belong to the same Azure Active Directory (Azure AD) tenant in the same Azure region.</span></span> <span data-ttu-id="4a471-154">Ez lehetővé teszi minden számítási feladat decentralizált felügyeletét, miközben a szolgáltatások megosztása változatlan marad az agyi virtuális hálózatban.</span><span class="sxs-lookup"><span data-stu-id="4a471-154">This allows for a decentralized management of each workload, while sharing services maintained in the hub VNet.</span></span>

### <a name="vnet-and-gatewaysubnet"></a><span data-ttu-id="4a471-155">Virtuális hálózat és GatewaySubnet</span><span class="sxs-lookup"><span data-stu-id="4a471-155">VNet and GatewaySubnet</span></span>

<span data-ttu-id="4a471-156">Hozzon létre egy *GatewaySubnet* nevű alhálózatot /27 címtartománnyal.</span><span class="sxs-lookup"><span data-stu-id="4a471-156">Create a subnet named *GatewaySubnet*, with an address range of /27.</span></span> <span data-ttu-id="4a471-157">Ez az alhálózat a virtuális hálózat átjárójához szükséges.</span><span class="sxs-lookup"><span data-stu-id="4a471-157">This subnet is required by the virtual network gateway.</span></span> <span data-ttu-id="4a471-158">Ha kioszt 32 címet erre az alhálózatra, azzal megelőzheti az átjáró mérethatárának elérését a jövőben.</span><span class="sxs-lookup"><span data-stu-id="4a471-158">Allocating 32 addresses to this subnet will help to prevent reaching gateway size limitations in the future.</span></span>

<span data-ttu-id="4a471-159">Az átjáró felállításáról további információkat az alábbi referenciaarchitektúrákban talál (a kapcsolattípustól függően):</span><span class="sxs-lookup"><span data-stu-id="4a471-159">For more information about setting up the gateway, see the following reference architectures, depending on your connection type:</span></span>

- <span data-ttu-id="4a471-160">[Hibrid hálózat ExpressRoute használatával][guidance-expressroute]</span><span class="sxs-lookup"><span data-stu-id="4a471-160">[Hybrid network using ExpressRoute][guidance-expressroute]</span></span>
- <span data-ttu-id="4a471-161">[Hibrid hálózat VPN-átjáró használatával][guidance-vpn]</span><span class="sxs-lookup"><span data-stu-id="4a471-161">[Hybrid network using a VPN gateway][guidance-vpn]</span></span>

<span data-ttu-id="4a471-162">A magasabb rendelkezésre álláshoz használhat ExpressRoute-ot és egy VPN-t feladatátvétel esetére.</span><span class="sxs-lookup"><span data-stu-id="4a471-162">For higher availability, you can use ExpressRoute plus a VPN for failover.</span></span> <span data-ttu-id="4a471-163">További információ: [Helyszíni hálózat csatlakoztatása az Azure-hoz VPN-feladatátvételt biztosító ExpressRoute használatával][hybrid-ha].</span><span class="sxs-lookup"><span data-stu-id="4a471-163">See [Connect an on-premises network to Azure using ExpressRoute with VPN failover][hybrid-ha].</span></span>

<span data-ttu-id="4a471-164">Küllős topológiát használhat átjáró nélkül is, ha nincs szükség a helyszíni hálózattal való kapcsolatra.</span><span class="sxs-lookup"><span data-stu-id="4a471-164">A hub-spoke topology can also be used without a gateway, if you don't need connectivity with your on-premises network.</span></span> 

### <a name="vnet-peering"></a><span data-ttu-id="4a471-165">Társviszony létesítése virtuális hálózatok között</span><span class="sxs-lookup"><span data-stu-id="4a471-165">VNet peering</span></span>

<span data-ttu-id="4a471-166">A virtuális társhálózatok létesítése nem tranzitív kapcsolat két virtuális hálózat között.</span><span class="sxs-lookup"><span data-stu-id="4a471-166">VNet peering is a non-transitive relationship between two VNets.</span></span> <span data-ttu-id="4a471-167">Ha két küllőt szeretne egymáshoz csatlakoztatni, vegye fontolóra egy külön társviszonykapcsolat hozzáadását az adott küllők között.</span><span class="sxs-lookup"><span data-stu-id="4a471-167">If you require spokes to connect to each other, consider adding a separate peering connection between those spokes.</span></span>

<span data-ttu-id="4a471-168">Ha azonban több küllőt is szeretne egymáshoz csatlakoztatni, gyorsan el fognak fogyni a rendelkezésre álló társviszonykapcsolatok a [virtuális hálózatonként létesíthető társviszonykapcsolatok számának korlátozása][vnet-peering-limit] miatt.</span><span class="sxs-lookup"><span data-stu-id="4a471-168">However, if you have several spokes that need to connect with each other, you will run out of possible peering connections very quickly due to the [limitation on number of VNets peerings per VNet][vnet-peering-limit].</span></span> <span data-ttu-id="4a471-169">Ilyenkor vegye fontolóra a felhasználó által megadott útvonalak (UDR-ek) használatát. Ezekkel kényszerítheti, hogy az egy adott küllőre irányuló forgalmat a rendszer egy NVA-ra küldje, amely útválasztóként működik az agyi virtuális hálózaton.</span><span class="sxs-lookup"><span data-stu-id="4a471-169">In this scenario, consider using user defined routes (UDRs) to force traffic destined to a spoke to be sent to an NVA acting as a router at the hub VNet.</span></span> <span data-ttu-id="4a471-170">Ez lehetővé teszi, hogy a küllők kapcsolódjanak egymáshoz.</span><span class="sxs-lookup"><span data-stu-id="4a471-170">This will allow the spokes to connect to each other.</span></span>

<span data-ttu-id="4a471-171">Ezenkívül konfigurálhatja is a küllőket, hogy használják az agyi virtuális hálózatot a távoli hálózatokkal való kommunikációra.</span><span class="sxs-lookup"><span data-stu-id="4a471-171">You can also configure spokes to use the hub VNet gateway to communicate with remote networks.</span></span> <span data-ttu-id="4a471-172">Ahhoz, hogy az átjáróforgalom működjön a küllő és az agy között, a távoli hálózatok pedig kapcsolódjanak, a következők szükségesek:</span><span class="sxs-lookup"><span data-stu-id="4a471-172">To allow gateway traffic to flow from spoke to hub, and connect to remote networks, you must:</span></span>

  - <span data-ttu-id="4a471-173">Konfigurálja a virtuális társhálózat-kapcsolatot az agyban az **átjáróforgalom engedélyezéséhez**.</span><span class="sxs-lookup"><span data-stu-id="4a471-173">Configure the VNet peering connection in the hub to **allow gateway transit**.</span></span>
  - <span data-ttu-id="4a471-174">Konfigurálja a virtuális társhálózat-kapcsolatot minden egyes küllőben a **távoli átjárók** használatához.</span><span class="sxs-lookup"><span data-stu-id="4a471-174">Configure the VNet peering connection in each spoke to **use remote gateways**.</span></span>
  - <span data-ttu-id="4a471-175">Konfiguráljon minden virtuális társhálózat-kapcsolatot a **továbbított forgalom engedélyezéséhez**.</span><span class="sxs-lookup"><span data-stu-id="4a471-175">Configure all VNet peering connections to **allow forwarded traffic**.</span></span>

## <a name="considerations"></a><span data-ttu-id="4a471-176">Megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="4a471-176">Considerations</span></span>

### <a name="spoke-connectivity"></a><span data-ttu-id="4a471-177">Küllőkapcsolat</span><span class="sxs-lookup"><span data-stu-id="4a471-177">Spoke connectivity</span></span>

<span data-ttu-id="4a471-178">Ha küllők között szeretne kapcsolatot létrehozni, fontolja meg egy NVA implementálását az agyban történő útválasztáshoz, illetve az UDR-ek használatát a küllőkben a forgalom továbbítása érdekében az agyhoz.</span><span class="sxs-lookup"><span data-stu-id="4a471-178">If you require connectivity between spokes, consider implementing an NVA for routing in the hub, and using UDRs in the spoke to forward traffic to the hub.</span></span>

<span data-ttu-id="4a471-179">![[2]][2]</span><span class="sxs-lookup"><span data-stu-id="4a471-179">![[2]][2]</span></span>

<span data-ttu-id="4a471-180">Ilyenkor konfigurálnia kell a társviszony-kapcsolatokat a **továbbított forgalom engedélyezéséhez**.</span><span class="sxs-lookup"><span data-stu-id="4a471-180">In this scenario, you must configure the peering connections to **allow forwarded traffic**.</span></span>

### <a name="overcoming-vnet-peering-limits"></a><span data-ttu-id="4a471-181">A virtuális társhálózatok létesítési korlátjának kiküszöbölése</span><span class="sxs-lookup"><span data-stu-id="4a471-181">Overcoming VNet peering limits</span></span>

<span data-ttu-id="4a471-182">Mindenképpen vegye figyelembe a [virtuális hálózatonként létesíthető társviszonyok számának korlátozását][vnet-peering-limit] az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="4a471-182">Make sure you consider the [limitation on number of VNets peerings per VNet][vnet-peering-limit] in Azure.</span></span> <span data-ttu-id="4a471-183">Ha több küllőre van szükség, mint amit a korlátozás megenged, fontolja meg egy agy-küllő-agy-küllő topológia létrehozását, ahol a küllők első szintje agyként is működik.</span><span class="sxs-lookup"><span data-stu-id="4a471-183">If you decide you need more spokes than the limit will allow, consider creating a hub-spoke-hub-spoke topology, where the first level of spokes also act as hubs.</span></span> <span data-ttu-id="4a471-184">Az alábbi diagram ezt a megközelítést mutatja be.</span><span class="sxs-lookup"><span data-stu-id="4a471-184">The following diagram shows this approach.</span></span>

<span data-ttu-id="4a471-185">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="4a471-185">![[3]][3]</span></span>

<span data-ttu-id="4a471-186">Ezenkívül vegye figyelembe, mely szolgáltatások vannak megosztva az agyban. Ezzel biztosítható, hogy az agy több küllőre is skálázzon.</span><span class="sxs-lookup"><span data-stu-id="4a471-186">Also consider what services are shared in the hub, to ensure the hub scales for a larger number of spokes.</span></span> <span data-ttu-id="4a471-187">Például ha az agy tűzfalszolgáltatásokat is nyújt, vegye figyelembe a tűzfal megoldás sávszélességi korlátozásait több küllő hozzáadása esetén.</span><span class="sxs-lookup"><span data-stu-id="4a471-187">For instance, if your hub provides firewall services, consider the bandwidth limits of your firewall solution when adding multiple spokes.</span></span> <span data-ttu-id="4a471-188">Lehet, hogy érdemes néhány megosztott szolgáltatást az agyak második szintjére mozgatni.</span><span class="sxs-lookup"><span data-stu-id="4a471-188">You might want to move some of these shared services to a second level of hubs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="4a471-189">A megoldás üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="4a471-189">Deploy the solution</span></span>

<span data-ttu-id="4a471-190">Ennek az architektúrának egy üzemelő példánya elérhető a [GitHubon][ref-arch-repo].</span><span class="sxs-lookup"><span data-stu-id="4a471-190">A deployment for this architecture is available on [GitHub][ref-arch-repo].</span></span> <span data-ttu-id="4a471-191">Ez minden virtuális hálózatban Ubuntu virtuális gépeket használ a kapcsolat tesztelésére.</span><span class="sxs-lookup"><span data-stu-id="4a471-191">It uses Ubuntu VMs in each VNet to test connectivity.</span></span> <span data-ttu-id="4a471-192">Az **agyi virtuális hálózat** **megosztott szolgáltatási** alhálózatában nincsenek tárolva tényleges szolgáltatások.</span><span class="sxs-lookup"><span data-stu-id="4a471-192">There are no actual services hosted in the **shared-services** subnet in the **hub VNet**.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="4a471-193">Előfeltételek</span><span class="sxs-lookup"><span data-stu-id="4a471-193">Prerequisites</span></span>

<span data-ttu-id="4a471-194">Mielőtt üzembe helyezhetné saját előfizetésében a referenciaarchitektúrát, az alábbi lépéseket kell elvégeznie.</span><span class="sxs-lookup"><span data-stu-id="4a471-194">Before you can deploy the reference architecture to your own subscription, you must perform the following steps.</span></span>

1. <span data-ttu-id="4a471-195">Klónozza, ágaztassa el vagy töltse le az [AzureCAT referenciaarchitektúrák][ref-arch-repo] GitHub-adattár zip-fájlját.</span><span class="sxs-lookup"><span data-stu-id="4a471-195">Clone, fork, or download the zip file for the [AzureCAT reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="4a471-196">Győződjön meg arról, hogy az Azure CLI 2.0 telepítve van a számítógépén.</span><span class="sxs-lookup"><span data-stu-id="4a471-196">Make sure you have the Azure CLI 2.0 installed on your computer.</span></span> <span data-ttu-id="4a471-197">Információk a CLI telepítésével kapcsolatban: [Az Azure CLI 2.0 telepítése][azure-cli-2].</span><span class="sxs-lookup"><span data-stu-id="4a471-197">For CLI installation instructions, see [Install Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="4a471-198">Telepítse a [Azure buulding blokkok] [ azbb] npm csomag.</span><span class="sxs-lookup"><span data-stu-id="4a471-198">Install the [Azure buulding blocks][azbb] npm package.</span></span>

4. <span data-ttu-id="4a471-199">Egy parancs parancssori futtatásával, vagy PowerShell kérdés, jelentkezzen be az Azure-fiókjával az alábbi parancs segítségével bash, és kövesse az utasításokat.</span><span class="sxs-lookup"><span data-stu-id="4a471-199">From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account by using the command below, and follow the prompts.</span></span>

  ```bash
  az login
  ```

### <a name="deploy-the-simulated-on-premises-datacenter-using-azbb"></a><span data-ttu-id="4a471-200">A szimulált olyan helyszíni adatközpontban azbb használatával telepítése</span><span class="sxs-lookup"><span data-stu-id="4a471-200">Deploy the simulated on-premises datacenter using azbb</span></span>

<span data-ttu-id="4a471-201">A szimulált olyan helyszíni adatközpontban egy Azure virtuális hálózatot, központi telepítéséhez kövesse az alábbi lépéseket:</span><span class="sxs-lookup"><span data-stu-id="4a471-201">To deploy the simulated on-premises datacenter as an Azure VNet, follow these steps:</span></span>

1. <span data-ttu-id="4a471-202">Navigáljon az előfeltételekre vonatkozó előző lépésekben letöltött adattár `hybrid-networking\hub-spoke\` mappájához.</span><span class="sxs-lookup"><span data-stu-id="4a471-202">Navigate to the `hybrid-networking\hub-spoke\` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="4a471-203">Nyissa meg a `onprem.json` fájlt, és adjon meg egy felhasználónevet és jelszót között a mezőkben szereplő idézőjeleket sor 36 37, alább látható módon, majd mentse a fájlt.</span><span class="sxs-lookup"><span data-stu-id="4a471-203">Open the `onprem.json` file and enter a username and password between the quotes in line 36 and 37, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. <span data-ttu-id="4a471-204">A sor 38, a `osType`, típus `Windows` vagy `Linux` Windows Server 2016 Datacenter, vagy Ubuntu 16.04 a jumpbox az operációs rendszer telepítéséhez.</span><span class="sxs-lookup"><span data-stu-id="4a471-204">On line 38, for `osType`, type `Windows` or `Linux` to install either Windows Server 2016 Datacenter, or Ubuntu 16.04 as the operating system for the jumpbox.</span></span>

4. <span data-ttu-id="4a471-205">Futtatás `azbb` a szimulált helyi üzemeltetésű környezet telepítése a lent látható módon.</span><span class="sxs-lookup"><span data-stu-id="4a471-205">Run `azbb` to deploy the simulated onprem environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g onprem-vnet-rg - l <location> -p onoprem.json --deploy
  ```
  > [!NOTE]
  > <span data-ttu-id="4a471-206">Ha úgy dönt, más erőforráscsoport-nevet használ (az `onprem-vnet-rg` névtől eltérőt), győződjön meg róla, hogy minden, ezt a nevet viselő paraméterfájlt megkeres és szerkeszti őket, hogy a saját erőforráscsoport-nevét használják.</span><span class="sxs-lookup"><span data-stu-id="4a471-206">If you decide to use a different resource group name (other than `onprem-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="4a471-207">Várjon, amíg az üzembe helyezés befejeződik.</span><span class="sxs-lookup"><span data-stu-id="4a471-207">Wait for the deployment to finish.</span></span> <span data-ttu-id="4a471-208">A központi telepítés létrehoz egy virtuális hálózathoz, a virtuális gép és a VPN-átjáró.</span><span class="sxs-lookup"><span data-stu-id="4a471-208">This deployment creates a virtual network, a virtual machine, and a VPN gateway.</span></span> <span data-ttu-id="4a471-209">Egy VPN-átjáró létrehozása 40 percnél is tovább tarthat.</span><span class="sxs-lookup"><span data-stu-id="4a471-209">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="azure-hub-vnet"></a><span data-ttu-id="4a471-210">Azure agyi virtuális hálózat</span><span class="sxs-lookup"><span data-stu-id="4a471-210">Azure hub VNet</span></span>

<span data-ttu-id="4a471-211">Az agyi virtuális hálózat üzembe helyezéséhez és a korábban létrehozott, szimulált helyszíni virtuális hálózathoz való kapcsolódáshoz kövesse az alábbi lépéseket.</span><span class="sxs-lookup"><span data-stu-id="4a471-211">To deploy the hub VNet, and connect to the simulated on-premises VNet created above, perform the following steps.</span></span>

1. <span data-ttu-id="4a471-212">Nyissa meg a `hub-vnet.json` fájlt, és adjon meg egy felhasználónevet és jelszót a mezőkben szereplő idézőjeleket sor 39 és 40 közötti alább látható módon.</span><span class="sxs-lookup"><span data-stu-id="4a471-212">Open the `hub-vnet.json` file and enter a username and password between the quotes in line 39 and 40, as shown below.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

2. <span data-ttu-id="4a471-213">A sor a 41-es, `osType`, típus `Windows` vagy `Linux` Windows Server 2016 Datacenter, vagy Ubuntu 16.04 a jumpbox az operációs rendszer telepítéséhez.</span><span class="sxs-lookup"><span data-stu-id="4a471-213">On line 41, for `osType`, type `Windows` or `Linux` to install either Windows Server 2016 Datacenter, or Ubuntu 16.04 as the operating system for the jumpbox.</span></span>

3. <span data-ttu-id="4a471-214">Adjon meg egy megosztott kulcsot sorban 72, az idézőjelek között alább látható módon, majd mentse a fájlt.</span><span class="sxs-lookup"><span data-stu-id="4a471-214">Enter a shared key between the quotes in line 72, as shown below, then save the file.</span></span>

  ```bash
  "sharedKey": "",
  ```

4. <span data-ttu-id="4a471-215">Futtatás `azbb` a szimulált helyi üzemeltetésű környezet telepítése a lent látható módon.</span><span class="sxs-lookup"><span data-stu-id="4a471-215">Run `azbb` to deploy the simulated onprem environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g hub-vnet-rg - l <location> -p hub-vnet.json --deploy
  ```
  > [!NOTE]
  > <span data-ttu-id="4a471-216">Ha úgy dönt, más erőforráscsoport-nevet használ (az `hub-vnet-rg` névtől eltérőt), győződjön meg róla, hogy minden, ezt a nevet viselő paraméterfájlt megkeres és szerkeszti őket, hogy a saját erőforráscsoport-nevét használják.</span><span class="sxs-lookup"><span data-stu-id="4a471-216">If you decide to use a different resource group name (other than `hub-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="4a471-217">Várjon, amíg az üzembe helyezés befejeződik.</span><span class="sxs-lookup"><span data-stu-id="4a471-217">Wait for the deployment to finish.</span></span> <span data-ttu-id="4a471-218">A központi telepítéshez létrehoz egy virtuális hálózatot, virtuális gép, VPN-átjáró és az átjáró, az előző szakaszban létrehozott kapcsolat.</span><span class="sxs-lookup"><span data-stu-id="4a471-218">This deployment creates a virtual network, a virtual machine, a VPN gateway, and a connection to the gateway created in the previous section.</span></span> <span data-ttu-id="4a471-219">Egy VPN-átjáró létrehozása 40 percnél is tovább tarthat.</span><span class="sxs-lookup"><span data-stu-id="4a471-219">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="optional-test-connectivity-from-onprem-to-hub"></a><span data-ttu-id="4a471-220">(Választható) A helyi üzemeltetésű hubhoz kapcsolat tesztelése</span><span class="sxs-lookup"><span data-stu-id="4a471-220">(Optional) Test connectivity from onprem to hub</span></span>

<span data-ttu-id="4a471-221">A szimulált a helyszíni környezetből szeretne az elosztóhoz használata a Windows virtuális gépek virtuális hálózat kapcsolat, hajtsa végre az alábbi lépéseket.</span><span class="sxs-lookup"><span data-stu-id="4a471-221">To test conectivity from the simulated on-premises environment to the hub VNet using Windows VMs, perform the following steps.</span></span>

1. <span data-ttu-id="4a471-222">Azure-portálról, navigáljon a `onprem-jb-rg` erőforráscsoportot, majd kattintson a a `jb-vm1` virtuálisgép-erőforrást.</span><span class="sxs-lookup"><span data-stu-id="4a471-222">From the Azure portal, navigate to the `onprem-jb-rg` resource group, then click on the `jb-vm1` virtual machine resource.</span></span>

2.  <span data-ttu-id="4a471-223">A virtuális gép panelről a portál felső bal oldali sarokban, kattintson a `Connect`, és kövesse az utasításokat a távoli asztal segítségével csatlakoztassa a virtuális Gépet.</span><span class="sxs-lookup"><span data-stu-id="4a471-223">On the top left hand corner of your VM blade in the portal, click `Connect`, and follow the prompts to use remote desktop to connect to the VM.</span></span> <span data-ttu-id="4a471-224">Győződjön meg arról, hogy a megadott felhasználónévnek és jelszónak meg 36 és 37 a sorokban használandó a `onprem.json` fájlt.</span><span class="sxs-lookup"><span data-stu-id="4a471-224">Make sure to use the username and password you specified in lines 36 and 37 in the `onprem.json` file.</span></span>

3. <span data-ttu-id="4a471-225">Nyissa meg a PowerShell-konzolban a virtuális Gépet, és használja a `Test-NetConnection` parancsmag segítségével győződjön meg arról, hogy képes csatlakozni a központ jumpbox VM alább látható módon.</span><span class="sxs-lookup"><span data-stu-id="4a471-225">Open a PowerShell console in the VM, and use the `Test-NetConnection` cmdlet to verify that you can connect to the hub jumpbox VM as shown below.</span></span>

  ```powershell
  Test-NetConnection 10.0.0.68 -CommonTCPPort RDP
  ```
  > [!NOTE]
  > <span data-ttu-id="4a471-226">Alapértelmezés szerint a Windows Server virtuális gépen nem teszik lehetővé az ICMP-válaszokat az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="4a471-226">By default, Windows Server VMs do not allow ICMP responses in Azure.</span></span> <span data-ttu-id="4a471-227">Ha a használni kívánt `ping` kapcsolat tesztelése, az egyes virtuális gépek ICMP-forgalmat a Windows speciális tűzfalon engedélyezni kell.</span><span class="sxs-lookup"><span data-stu-id="4a471-227">If you want to use `ping` to test connectivity, you need to enable ICMP traffic in the Windows Advanced Firewall for each VM.</span></span>

<span data-ttu-id="4a471-228">A szimulált a helyszíni környezetből szeretne az elosztóhoz használata a Linux virtuális gépek virtuális hálózat kapcsolat, hajtsa végre az alábbi lépéseket:</span><span class="sxs-lookup"><span data-stu-id="4a471-228">To test conectivity from the simulated on-premises environment to the hub VNet using Linux VMs, perform the following steps:</span></span>

1. <span data-ttu-id="4a471-229">Azure-portálról, navigáljon a `onprem-jb-rg` erőforráscsoportot, majd kattintson a a `jb-vm1` virtuálisgép-erőforrást.</span><span class="sxs-lookup"><span data-stu-id="4a471-229">From the Azure portal, navigate to the `onprem-jb-rg` resource group, then click on the `jb-vm1` virtual machine resource.</span></span>

2. <span data-ttu-id="4a471-230">A virtuális gép panelről a portál felső bal oldali sarokban, kattintson a `Connect`, majd másolja a `ssh` jelenik meg a portál a parancsot.</span><span class="sxs-lookup"><span data-stu-id="4a471-230">On the top left hand corner of your VM blade in the portal, click `Connect`, and then copy the `ssh` command shown on the portal.</span></span> 

3. <span data-ttu-id="4a471-231">A Linux parancssorból futtassa `ssh` való kapcsolódáshoz a helyszíni szimulált környezetben jumpbox witht a fenti 2. lépésben másolt információt alább látható módon.</span><span class="sxs-lookup"><span data-stu-id="4a471-231">From a Linux prompt, run `ssh` to connect to the simulated on-premises environment jumpbox witht the information you copied in step 2 above, as shown below.</span></span>

  ```bash
  ssh <your_user>@<public_ip_address>
  ```

4. <span data-ttu-id="4a471-232">37 sort a megadott jelszó használata a `onprem.json` fájl a virtuális Géphez való csatlakozáshoz.</span><span class="sxs-lookup"><span data-stu-id="4a471-232">Use the password you specified in line 37 in the `onprem.json` file to the connect to the VM.</span></span>

5. <span data-ttu-id="4a471-233">Használja a `ping` parancs hub jumpbox a kapcsolat tesztelése a lent látható módon.</span><span class="sxs-lookup"><span data-stu-id="4a471-233">Use the `ping` command to test connectivity to the hub jumpbox, as shown below.</span></span>

  ```bash
  ping 10.0.0.68
  ```

### <a name="azure-spoke-vnets"></a><span data-ttu-id="4a471-234">Azure küllő virtuális hálózatok</span><span class="sxs-lookup"><span data-stu-id="4a471-234">Azure spoke VNets</span></span>

<span data-ttu-id="4a471-235">A Vnetek küllős telepítéséhez hajtsa végre az alábbi lépéseket.</span><span class="sxs-lookup"><span data-stu-id="4a471-235">To deploy the spoke VNets, perform the following steps.</span></span>

1. <span data-ttu-id="4a471-236">Nyissa meg a `spoke1.json` fájlt, és adjon meg egy felhasználónevet és jelszót között a mezőkben szereplő idézőjeleket sorok 47 és 48, alább látható módon, majd mentse a fájlt.</span><span class="sxs-lookup"><span data-stu-id="4a471-236">Open the `spoke1.json` file and enter a username and password between the quotes in lines 47 and 48, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

2. <span data-ttu-id="4a471-237">A sor 49, a `osType`, típus `Windows` vagy `Linux` Windows Server 2016 Datacenter, vagy Ubuntu 16.04 a jumpbox az operációs rendszer telepítéséhez.</span><span class="sxs-lookup"><span data-stu-id="4a471-237">On line 49, for `osType`, type `Windows` or `Linux` to install either Windows Server 2016 Datacenter, or Ubuntu 16.04 as the operating system for the jumpbox.</span></span>

3. <span data-ttu-id="4a471-238">Futtatás `azbb` központi telepítése az első küllős VNet környezet alább látható módon.</span><span class="sxs-lookup"><span data-stu-id="4a471-238">Run `azbb` to deploy the first spoke VNet environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g spoke1-vnet-rg - l <location> -p spoke1.json --deploy
  ```
  
  > [!NOTE]
  > <span data-ttu-id="4a471-239">Ha úgy dönt, más erőforráscsoport-nevet használ (az `spoke1-vnet-rg` névtől eltérőt), győződjön meg róla, hogy minden, ezt a nevet viselő paraméterfájlt megkeres és szerkeszti őket, hogy a saját erőforráscsoport-nevét használják.</span><span class="sxs-lookup"><span data-stu-id="4a471-239">If you decide to use a different resource group name (other than `spoke1-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

3. <span data-ttu-id="4a471-240">Ismételje meg az 1-fájl `spoke2.json`.</span><span class="sxs-lookup"><span data-stu-id="4a471-240">Repeat step 1 above for file `spoke2.json`.</span></span>

4. <span data-ttu-id="4a471-241">Futtatás `azbb` központi telepítése a második küllős VNet környezet alább látható módon.</span><span class="sxs-lookup"><span data-stu-id="4a471-241">Run `azbb` to deploy the second spoke VNet environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g spoke2-vnet-rg - l <location> -p spoke2.json --deploy
  ```
  > [!NOTE]
  > <span data-ttu-id="4a471-242">Ha úgy dönt, más erőforráscsoport-nevet használ (az `spoke2-vnet-rg` névtől eltérőt), győződjön meg róla, hogy minden, ezt a nevet viselő paraméterfájlt megkeres és szerkeszti őket, hogy a saját erőforráscsoport-nevét használják.</span><span class="sxs-lookup"><span data-stu-id="4a471-242">If you decide to use a different resource group name (other than `spoke2-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

### <a name="azure-hub-vnet-peering-to-spoke-vnets"></a><span data-ttu-id="4a471-243">Azure agyi virtuális társhálózatok létesítése küllő virtuális hálózatokhoz</span><span class="sxs-lookup"><span data-stu-id="4a471-243">Azure hub VNet peering to spoke VNets</span></span>

<span data-ttu-id="4a471-244">A központ virtuális hálózat számára a Vnetek küllős társviszony-létesítési kapcsolat létrehozásához hajtsa végre az alábbi lépéseket.</span><span class="sxs-lookup"><span data-stu-id="4a471-244">To create a peering connection from the hub VNet to the spoke VNets, perform the following steps.</span></span>

1. <span data-ttu-id="4a471-245">Nyissa meg a `hub-vnet-peering.json` fájlt, és ellenőrizze, hogy az egyes a virtuális hálózati társviszony sor 29 kezdve az erőforráscsoport nevét, és a virtuális hálózat neve helyesen-e.</span><span class="sxs-lookup"><span data-stu-id="4a471-245">Open the `hub-vnet-peering.json` file and verify that the resource group name, and virtual network name for each of the virtual network peerings starting in line 29 are correct.</span></span>

2. <span data-ttu-id="4a471-246">Futtatás `azbb` központi telepítése az első küllős VNet környezet alább látható módon.</span><span class="sxs-lookup"><span data-stu-id="4a471-246">Run `azbb` to deploy the first spoke VNet environment as shown below.</span></span>

  ```bash
  azbb -s <subscription_id> -g hub-vnet-rg - l <location> -p hub-vnet-peering.json --deploy
  ```

  > [!NOTE]
  > <span data-ttu-id="4a471-247">Ha úgy dönt, más erőforráscsoport-nevet használ (az `hub-vnet-rg` névtől eltérőt), győződjön meg róla, hogy minden, ezt a nevet viselő paraméterfájlt megkeres és szerkeszti őket, hogy a saját erőforráscsoport-nevét használják.</span><span class="sxs-lookup"><span data-stu-id="4a471-247">If you decide to use a different resource group name (other than `hub-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

### <a name="test-connectivity"></a><span data-ttu-id="4a471-248">Kapcsolat tesztelése</span><span class="sxs-lookup"><span data-stu-id="4a471-248">Test connectivity</span></span>

<span data-ttu-id="4a471-249">A szimulált a helyszíni környezetből a küllős Vnetek Windows virtuális gépek használata a kapcsolat teszteléséhez a következő lépésekkel.</span><span class="sxs-lookup"><span data-stu-id="4a471-249">To test conectivity from the simulated on-premises environment to the spoke VNets using Windows VMs, perform the following steps.</span></span>

1. <span data-ttu-id="4a471-250">Azure-portálról, navigáljon a `onprem-jb-rg` erőforráscsoportot, majd kattintson a a `jb-vm1` virtuálisgép-erőforrást.</span><span class="sxs-lookup"><span data-stu-id="4a471-250">From the Azure portal, navigate to the `onprem-jb-rg` resource group, then click on the `jb-vm1` virtual machine resource.</span></span>

2.  <span data-ttu-id="4a471-251">A virtuális gép panelről a portál felső bal oldali sarokban, kattintson a `Connect`, és kövesse az utasításokat a távoli asztal segítségével csatlakoztassa a virtuális Gépet.</span><span class="sxs-lookup"><span data-stu-id="4a471-251">On the top left hand corner of your VM blade in the portal, click `Connect`, and follow the prompts to use remote desktop to connect to the VM.</span></span> <span data-ttu-id="4a471-252">Győződjön meg arról, hogy a megadott felhasználónévnek és jelszónak meg 36 és 37 a sorokban használandó a `onprem.json` fájlt.</span><span class="sxs-lookup"><span data-stu-id="4a471-252">Make sure to use the username and password you specified in lines 36 and 37 in the `onprem.json` file.</span></span>

3. <span data-ttu-id="4a471-253">Nyissa meg a PowerShell-konzolban a virtuális Gépet, és használja a `Test-NetConnection` parancsmag segítségével győződjön meg arról, hogy képes csatlakozni a központ jumpbox VM alább látható módon.</span><span class="sxs-lookup"><span data-stu-id="4a471-253">Open a PowerShell console in the VM, and use the `Test-NetConnection` cmdlet to verify that you can connect to the hub jumpbox VM as shown below.</span></span>

  ```powershell
  Test-NetConnection 10.1.0.68 -CommonTCPPort RDP
  Test-NetConnection 10.2.0.68 -CommonTCPPort RDP
  ```

<span data-ttu-id="4a471-254">A szimulált a helyszíni környezetből a küllős Vnetek Linux virtuális gépek használata a kapcsolat, hajtsa végre az alábbi lépéseket:</span><span class="sxs-lookup"><span data-stu-id="4a471-254">To test conectivity from the simulated on-premises environment to the spoke VNets using Linux VMs, perform the following steps:</span></span>

1. <span data-ttu-id="4a471-255">Azure-portálról, navigáljon a `onprem-jb-rg` erőforráscsoportot, majd kattintson a a `jb-vm1` virtuálisgép-erőforrást.</span><span class="sxs-lookup"><span data-stu-id="4a471-255">From the Azure portal, navigate to the `onprem-jb-rg` resource group, then click on the `jb-vm1` virtual machine resource.</span></span>

2. <span data-ttu-id="4a471-256">A virtuális gép panelről a portál felső bal oldali sarokban, kattintson a `Connect`, majd másolja a `ssh` jelenik meg a portál a parancsot.</span><span class="sxs-lookup"><span data-stu-id="4a471-256">On the top left hand corner of your VM blade in the portal, click `Connect`, and then copy the `ssh` command shown on the portal.</span></span> 

3. <span data-ttu-id="4a471-257">A Linux parancssorból futtassa `ssh` való kapcsolódáshoz a helyszíni szimulált környezetben jumpbox witht a fenti 2. lépésben másolt információt alább látható módon.</span><span class="sxs-lookup"><span data-stu-id="4a471-257">From a Linux prompt, run `ssh` to connect to the simulated on-premises environment jumpbox witht the information you copied in step 2 above, as shown below.</span></span>

  ```bash
  ssh <your_user>@<public_ip_address>
  ```

5. <span data-ttu-id="4a471-258">37 sort a megadott jelszó használata a `onprem.json` fájl a virtuális Géphez való csatlakozáshoz.</span><span class="sxs-lookup"><span data-stu-id="4a471-258">Use the password you specified in line 37 in the `onprem.json` file to the connect to the VM.</span></span>

6. <span data-ttu-id="4a471-259">Használja a `ping` parancsot a virtuális gépek jumpbox kapcsolat teszteléséhez minden küllős, az alább látható módon.</span><span class="sxs-lookup"><span data-stu-id="4a471-259">Use the `ping` command to test connectivity to the jumpbox VMs in each spoke, as shown below.</span></span>

  ```bash
  ping 10.1.0.68
  ping 10.2.0.68
  ```

### <a name="add-connectivity-between-spokes"></a><span data-ttu-id="4a471-260">Kapcsolat hozzáadása küllők között</span><span class="sxs-lookup"><span data-stu-id="4a471-260">Add connectivity between spokes</span></span>

<span data-ttu-id="4a471-261">Ha szeretné engedélyezni a küllők csatlakozni egymáshoz, szeretné egy newtwork virtuális készülék (NVA) használja a központ virtuális hálózatot útválasztóként, és annak kikényszerítéséhez forgalom a küllők az útválasztóhoz egy másik küllős való kapcsolódási kísérlet során.</span><span class="sxs-lookup"><span data-stu-id="4a471-261">If you want to allow spokes to connect to each other, you need to use a newtwork virtual appliance (NVA) as a router in the hub virtual netowrk, and force traffic from spokes to the router when trying to connect to another spoke.</span></span> <span data-ttu-id="4a471-262">Egy alapszintű mintaalkalmazás NVA telepítését egy virtuális és a szükséges uder által megadott útvonalak engedélyezi a két küllős Vnetek csatlakozni, hajtsa végre a következő lépéseket:</span><span class="sxs-lookup"><span data-stu-id="4a471-262">To deploy a basic sample NVA as a single VM, and the necessary uder defined routes to allow the two spoke VNets to connect, perform the following steps:</span></span>

1. <span data-ttu-id="4a471-263">Nyissa meg a `hub-nva.json` fájlt, és adjon meg egy felhasználónevet és jelszót az 14., 13 sorok idézőjelek között alább látható módon, majd mentse a fájlt.</span><span class="sxs-lookup"><span data-stu-id="4a471-263">Open the `hub-nva.json` file and enter a username and password between the quotes in lines 13 and 14, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```
2. <span data-ttu-id="4a471-264">Futtatás `azbb` központi telepítése az NVA virtuális gép és a felhasználó által megadott útvonalak.</span><span class="sxs-lookup"><span data-stu-id="4a471-264">Run `azbb` to deploy the NVA VM and user defined routes.</span></span>

  ```bash
  azbb -s <subscription_id> -g hub-nva-rg - l <location> -p hub-nva.json --deploy
  ```
  > [!NOTE]
  > <span data-ttu-id="4a471-265">Ha úgy dönt, más erőforráscsoport-nevet használ (az `hub-nva-rg` névtől eltérőt), győződjön meg róla, hogy minden, ezt a nevet viselő paraméterfájlt megkeres és szerkeszti őket, hogy a saját erőforráscsoport-nevét használják.</span><span class="sxs-lookup"><span data-stu-id="4a471-265">If you decide to use a different resource group name (other than `hub-nva-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

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
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md

[visio-download]: https://archcenter.azureedge.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/hub-spoke.png "Küllős topológia az Azure-ban"
[1]: ./images/hub-spoke-gateway-routing.svg "Küllős topológia az Azure-ban tranzitív útválasztással"
[2]: ./images/hub-spoke-no-gateway-routing.svg "Küllős topológia az Azure-ban tranzitív útválasztással, NVA használatával"
[3]: ./images/hub-spokehub-spoke.svg "Küllős-küllős topológia az Azure-ban"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
