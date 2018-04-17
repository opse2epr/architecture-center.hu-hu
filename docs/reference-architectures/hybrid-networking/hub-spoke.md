---
title: Küllős hálózati topológia implementálása az Azure-ban
description: Küllős hálózati topológia implementálása az Azure-ban.
author: telmosampaio
ms.date: 04/09/2018
pnp.series.title: Implement a hub-spoke network topology in Azure
pnp.series.prev: expressroute
ms.openlocfilehash: f04af90f328a0434d44ca7ea90309f3209a3b69d
ms.sourcegitcommit: f665226cec96ec818ca06ac6c2d83edb23c9f29c
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/16/2018
---
# <a name="implement-a-hub-spoke-network-topology-in-azure"></a><span data-ttu-id="ed440-103">Küllős hálózati topológia implementálása az Azure-ban</span><span class="sxs-lookup"><span data-stu-id="ed440-103">Implement a hub-spoke network topology in Azure</span></span>

<span data-ttu-id="ed440-104">Ez a referenciaarchitektúra bemutatja, hogyan lehetséges a küllős hálózati topológia implementálása az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="ed440-104">This reference architecture shows how to implement a hub-spoke topology in Azure.</span></span> <span data-ttu-id="ed440-105">Az *agy* egy virtuális hálózat (VNet) az Azure-ban, amely központi kapcsolódási pontként szolgál a helyszíni hálózathoz.</span><span class="sxs-lookup"><span data-stu-id="ed440-105">The *hub* is a virtual network (VNet) in Azure that acts as a central point of connectivity to your on-premises network.</span></span> <span data-ttu-id="ed440-106">A *küllők* az agyhoz kapcsolódó virtuális hálózatok, és a számítási feladatok elkülönítésére használhatók.</span><span class="sxs-lookup"><span data-stu-id="ed440-106">The *spokes* are VNets that peer with the hub, and can be used to isolate workloads.</span></span> <span data-ttu-id="ed440-107">A helyszíni adatközpont és az agy közötti forgalom egy ExpressRoute- vagy VPN-átjárókapcsolaton keresztül halad át.</span><span class="sxs-lookup"><span data-stu-id="ed440-107">Traffic flows between the on-premises datacenter and the hub through an ExpressRoute or VPN gateway connection.</span></span>  <span data-ttu-id="ed440-108">[**A megoldás üzembe helyezése.**](#deploy-the-solution)</span><span class="sxs-lookup"><span data-stu-id="ed440-108">[**Deploy this solution**](#deploy-the-solution).</span></span>

<span data-ttu-id="ed440-109">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="ed440-109">![[0]][0]</span></span>

<span data-ttu-id="ed440-110">*Töltse le az ][visio-download]architektúra [Visio-fájlját*</span><span class="sxs-lookup"><span data-stu-id="ed440-110">*Download a [Visio file][visio-download] of this architecture*</span></span>


<span data-ttu-id="ed440-111">A topológia előnyei a következők:</span><span class="sxs-lookup"><span data-stu-id="ed440-111">The benefits of this toplogy include:</span></span>

* <span data-ttu-id="ed440-112">**Költségmegtakarítás** a szolgáltatások központosításával, amelyeket több számítási feladat között, egyetlen helyen lehet elosztani, például a hálózati virtuális berendezések (NVA-k) és DNS-kiszolgálók.</span><span class="sxs-lookup"><span data-stu-id="ed440-112">**Cost savings** by centralizing services that can be shared by multiple workloads, such as network virtual appliances (NVAs) and DNS servers, in a single location.</span></span>
* <span data-ttu-id="ed440-113">**Az előfizetési korlátok leküzdése** a különböző előfizetésekből virtuális társhálózatok létesítésével a központi agyhoz.</span><span class="sxs-lookup"><span data-stu-id="ed440-113">**Overcome subscriptions limits** by peering VNets from different subscriptions to the central hub.</span></span>
* <span data-ttu-id="ed440-114">**Kockázatok elkülönítése** a központi IT (SecOps, InfraOps) és a számítási feladatok között (DevOps).</span><span class="sxs-lookup"><span data-stu-id="ed440-114">**Separation of concerns** between central IT (SecOps, InfraOps) and workloads (DevOps).</span></span>

<span data-ttu-id="ed440-115">Az architektúra gyakori használati módjai többek között a következők:</span><span class="sxs-lookup"><span data-stu-id="ed440-115">Typical uses for this architecture include:</span></span>

* <span data-ttu-id="ed440-116">A különböző környezetekben (például a fejlesztés, a tesztelés és a gyártás) üzembe helyezett számítási feladatok, amelyek megosztott szolgáltatásokat igényelnek (például DNS, IDS, NTP vagy AD DS).</span><span class="sxs-lookup"><span data-stu-id="ed440-116">Workloads deployed in different environments, such as development, testing, and production, that require shared services such as DNS, IDS, NTP, or AD DS.</span></span> <span data-ttu-id="ed440-117">A megosztott szolgáltatások az agyi virtuális hálózatban találhatóak, azonban minden környezet egy küllőn van üzembe helyezve, így megmarad az elszigetelés.</span><span class="sxs-lookup"><span data-stu-id="ed440-117">Shared services are placed in the hub VNet, while each environment is deployed to a spoke to maintain isolation.</span></span>
* <span data-ttu-id="ed440-118">Olyan számítási feladatok, amelyek nem igénylik az egymással való kapcsolatot, de igénylik a megosztott szolgáltatásokhoz való hozzáférést.</span><span class="sxs-lookup"><span data-stu-id="ed440-118">Workloads that do not require connectivity to each other, but require access to shared services.</span></span>
* <span data-ttu-id="ed440-119">Olyan vállalatok, amelyek központi felügyeletet igényelnek a biztonsági szempontokhoz, például egy tűzfal az agyban DMZ-ként, és küllőnkénti elkülönített felügyelet a számítási feladatokhoz.</span><span class="sxs-lookup"><span data-stu-id="ed440-119">Enterprises that require central control over security aspects, such as a firewall in the hub as a DMZ, and segregated management for the workloads in each spoke.</span></span>

## <a name="architecture"></a><span data-ttu-id="ed440-120">Architektúra</span><span class="sxs-lookup"><span data-stu-id="ed440-120">Architecture</span></span>

<span data-ttu-id="ed440-121">Az architektúra a következőkben leírt összetevőkből áll.</span><span class="sxs-lookup"><span data-stu-id="ed440-121">The architecture consists of the following components.</span></span>

* <span data-ttu-id="ed440-122">**Helyszíni hálózat**.</span><span class="sxs-lookup"><span data-stu-id="ed440-122">**On-premises network**.</span></span> <span data-ttu-id="ed440-123">A cégen belül futó helyi magánhálózat.</span><span class="sxs-lookup"><span data-stu-id="ed440-123">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="ed440-124">**VPN-eszköz**.</span><span class="sxs-lookup"><span data-stu-id="ed440-124">**VPN device**.</span></span> <span data-ttu-id="ed440-125">A helyszíni hálózat számára külső kapcsolatot biztosító eszköz vagy szolgáltatás.</span><span class="sxs-lookup"><span data-stu-id="ed440-125">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="ed440-126">A VPN-eszköz lehet hardvereszköz vagy valamilyen szoftvermegoldás, amilyen például a Windows Server 2012 Útválasztás és távelérés szolgáltatása (Routing and Remote Access Service, RRAS).</span><span class="sxs-lookup"><span data-stu-id="ed440-126">The VPN device may be a hardware device, or a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="ed440-127">A támogatott VPN-berendezések listáját és az egyes VPN-berendezések Azure-hoz való csatlakoztatásra történő konfigurálásával kapcsolatos információkat az [Információk a helyek közötti VPN Gateway-kapcsolatok VPN-eszközeiről][vpn-appliance] című cikkben találja.</span><span class="sxs-lookup"><span data-stu-id="ed440-127">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="ed440-128">**VPN virtuális hálózati átjáró vagy ExpressRoute-átjáró**.</span><span class="sxs-lookup"><span data-stu-id="ed440-128">**VPN virtual network gateway or ExpressRoute gateway**.</span></span> <span data-ttu-id="ed440-129">A virtuális hálózati átjáró a helyszíni hálózattal létesített kapcsolathoz használt VPN-eszközhöz vagy az ExpressRoute-kapcsolatcsoporthoz való csatlakozást teszi lehetővé a virtuális hálózat számára.</span><span class="sxs-lookup"><span data-stu-id="ed440-129">The virtual network gateway enables the VNet to connect to the VPN device, or ExpressRoute circuit, used for connectivity with your on-premises network.</span></span> <span data-ttu-id="ed440-130">További információért tekintse át a [helyszíni hálózat és a Microsoft Azure Virtual Network csatlakoztatásával][connect-to-an-Azure-vnet] foglalkozó cikket.</span><span class="sxs-lookup"><span data-stu-id="ed440-130">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

> [!NOTE]
> <span data-ttu-id="ed440-131">A referenciaarchitektúra üzembe helyezési szkriptjei egy VPN-átjárót használnak a kapcsolat létrehozásához és egy virtuális hálózatot az Azure-ban a helyszíni hálózat szimulációjára.</span><span class="sxs-lookup"><span data-stu-id="ed440-131">The deployment scripts for this reference architecture use a VPN gateway for connectivity, and a VNet in Azure to simulate your on-premises network.</span></span>

* <span data-ttu-id="ed440-132">**Agyi virtuális hálózat**.</span><span class="sxs-lookup"><span data-stu-id="ed440-132">**Hub VNet**.</span></span> <span data-ttu-id="ed440-133">Egy Azure virtuális hálózat agyként a küllős hálózati topológiában.</span><span class="sxs-lookup"><span data-stu-id="ed440-133">Azure VNet used as the hub in the hub-spoke topology.</span></span> <span data-ttu-id="ed440-134">Az agy a helyszíni hálózathoz való csatlakozási lehetőségek központi helye, és a küllős hálózati topológiákban tárolt különféle számítási feladatok által felvett szolgáltatások tárolási helye.</span><span class="sxs-lookup"><span data-stu-id="ed440-134">The hub is the central point of connectivity to your on-premises network, and a place to host services that can be consumed by the different workloads hosted in the spoke VNets.</span></span>

* <span data-ttu-id="ed440-135">**Átjáró-alhálózat**.</span><span class="sxs-lookup"><span data-stu-id="ed440-135">**Gateway subnet**.</span></span> <span data-ttu-id="ed440-136">A virtuális hálózati átjárók ugyanazon az alhálózaton találhatók.</span><span class="sxs-lookup"><span data-stu-id="ed440-136">The virtual network gateways are held in the same subnet.</span></span>

* <span data-ttu-id="ed440-137">**Küllő virtuális hálózatok**.</span><span class="sxs-lookup"><span data-stu-id="ed440-137">**Spoke VNets**.</span></span> <span data-ttu-id="ed440-138">Egy vagy több Azure virtuális hálózat, amelyek küllőként használatosak a küllős topológiában.</span><span class="sxs-lookup"><span data-stu-id="ed440-138">One or more Azure VNets that are used as spokes in the hub-spoke topology.</span></span> <span data-ttu-id="ed440-139">A küllőkkel elszigetelhetőek a számítási feladatok saját virtuális hálózataikban, így más küllőktől elkülönülten kezelhetőek.</span><span class="sxs-lookup"><span data-stu-id="ed440-139">Spokes can be used to isolate workloads in their own VNets, managed separately from other spokes.</span></span> <span data-ttu-id="ed440-140">Minden számítási feladat több szintet tartalmazhat, amelyek alhálózatait Azure-terheléselosztók kapcsolják össze.</span><span class="sxs-lookup"><span data-stu-id="ed440-140">Each workload might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="ed440-141">További információ az alkalmazás-infrastruktúrával kapcsolatban: [Windows rendszerű virtuális gépek számítási feladatainak futtatása][windows-vm-ra] és [Számítási feladatok futtatása Linux rendszerű virtuális gépeken][linux-vm-ra].</span><span class="sxs-lookup"><span data-stu-id="ed440-141">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

* <span data-ttu-id="ed440-142">**Virtuális társhálózatok létesítése**.</span><span class="sxs-lookup"><span data-stu-id="ed440-142">**VNet peering**.</span></span> <span data-ttu-id="ed440-143">Két, ugyanabban az Azure-régióban található virtuális hálózatot egy [társviszonykapcsolat][vnet-peering] használatával lehet összekapcsolni.</span><span class="sxs-lookup"><span data-stu-id="ed440-143">Two VNets in the same Azure region can be connected using a [peering connection][vnet-peering].</span></span> <span data-ttu-id="ed440-144">A társviszonykapcsolatok a virtuális hálózatok közötti nem tranzitív, alacsony késleltetésű kapcsolatok.</span><span class="sxs-lookup"><span data-stu-id="ed440-144">Peering connections are non-transitive, low latency connections between VNets.</span></span> <span data-ttu-id="ed440-145">A társviszonyba kapcsolt virtuális hálózatok az Azure gerinchálózatát használva, útválasztó igénye nélkül váltanak forgalmat egymás között.</span><span class="sxs-lookup"><span data-stu-id="ed440-145">Once peered, the VNets exchange traffic by using the Azure backbone, without the need for a router.</span></span> <span data-ttu-id="ed440-146">Egy küllős hálózati topológiában a virtuális társhálózatok létesítésével lehetséges az agyat minden egyes küllőhöz kapcsolni.</span><span class="sxs-lookup"><span data-stu-id="ed440-146">In a hub-spoke network topology, you use VNet peering to connect the hub to each spoke.</span></span>

> [!NOTE]
> <span data-ttu-id="ed440-147">Ez a cikk csak a [Resource Manager](/azure/azure-resource-manager/resource-group-overview) üzemelő példányokat mutatja be, de ugyanabban az előfizetésben klasszikus virtuális hálózatot is csatlakoztathat Resource Manager virtuális hálózathoz.</span><span class="sxs-lookup"><span data-stu-id="ed440-147">This article only covers [Resource Manager](/azure/azure-resource-manager/resource-group-overview) deployments, but you can also connect a classic VNet to a Resource Manager VNet in the same subscription.</span></span> <span data-ttu-id="ed440-148">Így a küllők tárolhatnak klasszikus üzemelő példányokat, mégis profitálhatnak az agyban megosztott szolgáltatásokból.</span><span class="sxs-lookup"><span data-stu-id="ed440-148">That way, your spokes can host classic deployments and still benefit from services shared in the hub.</span></span>

## <a name="recommendations"></a><span data-ttu-id="ed440-149">Javaslatok</span><span class="sxs-lookup"><span data-stu-id="ed440-149">Recommendations</span></span>

<span data-ttu-id="ed440-150">Az alábbi javaslatok a legtöbb forgatókönyvre vonatkoznak.</span><span class="sxs-lookup"><span data-stu-id="ed440-150">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="ed440-151">Kövesse ezeket a javaslatokat, ha nincsenek ezeket felülíró követelményei.</span><span class="sxs-lookup"><span data-stu-id="ed440-151">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="resource-groups"></a><span data-ttu-id="ed440-152">Erőforráscsoportok</span><span class="sxs-lookup"><span data-stu-id="ed440-152">Resource groups</span></span>

<span data-ttu-id="ed440-153">Az agyi és minden küllő virtuális hálózat implementálható különböző erőforráscsoportokban és eltérő előfizetésekben is, ha mind ugyanahhoz az Azure Active Directory (Azure AD) bérlőhöz tartozik, ugyanabban az Azure-régióban.</span><span class="sxs-lookup"><span data-stu-id="ed440-153">The hub VNet, and each spoke VNet, can be implemented in different resource groups, and even different subscriptions, as long as they belong to the same Azure Active Directory (Azure AD) tenant in the same Azure region.</span></span> <span data-ttu-id="ed440-154">Ez lehetővé teszi minden számítási feladat decentralizált felügyeletét, miközben a szolgáltatások megosztása változatlan marad az agyi virtuális hálózatban.</span><span class="sxs-lookup"><span data-stu-id="ed440-154">This allows for a decentralized management of each workload, while sharing services maintained in the hub VNet.</span></span>

### <a name="vnet-and-gatewaysubnet"></a><span data-ttu-id="ed440-155">Virtuális hálózat és GatewaySubnet</span><span class="sxs-lookup"><span data-stu-id="ed440-155">VNet and GatewaySubnet</span></span>

<span data-ttu-id="ed440-156">Hozzon létre egy *GatewaySubnet* nevű alhálózatot /27 címtartománnyal.</span><span class="sxs-lookup"><span data-stu-id="ed440-156">Create a subnet named *GatewaySubnet*, with an address range of /27.</span></span> <span data-ttu-id="ed440-157">Ez az alhálózat a virtuális hálózat átjárójához szükséges.</span><span class="sxs-lookup"><span data-stu-id="ed440-157">This subnet is required by the virtual network gateway.</span></span> <span data-ttu-id="ed440-158">Ha kioszt 32 címet erre az alhálózatra, azzal megelőzheti az átjáró mérethatárának elérését a jövőben.</span><span class="sxs-lookup"><span data-stu-id="ed440-158">Allocating 32 addresses to this subnet will help to prevent reaching gateway size limitations in the future.</span></span>

<span data-ttu-id="ed440-159">Az átjáró felállításáról további információkat az alábbi referenciaarchitektúrákban talál (a kapcsolattípustól függően):</span><span class="sxs-lookup"><span data-stu-id="ed440-159">For more information about setting up the gateway, see the following reference architectures, depending on your connection type:</span></span>

- <span data-ttu-id="ed440-160">[Hibrid hálózat ExpressRoute használatával][guidance-expressroute]</span><span class="sxs-lookup"><span data-stu-id="ed440-160">[Hybrid network using ExpressRoute][guidance-expressroute]</span></span>
- <span data-ttu-id="ed440-161">[Hibrid hálózat VPN-átjáró használatával][guidance-vpn]</span><span class="sxs-lookup"><span data-stu-id="ed440-161">[Hybrid network using a VPN gateway][guidance-vpn]</span></span>

<span data-ttu-id="ed440-162">A magasabb rendelkezésre álláshoz használhat ExpressRoute-ot és egy VPN-t feladatátvétel esetére.</span><span class="sxs-lookup"><span data-stu-id="ed440-162">For higher availability, you can use ExpressRoute plus a VPN for failover.</span></span> <span data-ttu-id="ed440-163">További információ: [Helyszíni hálózat csatlakoztatása az Azure-hoz VPN-feladatátvételt biztosító ExpressRoute használatával][hybrid-ha].</span><span class="sxs-lookup"><span data-stu-id="ed440-163">See [Connect an on-premises network to Azure using ExpressRoute with VPN failover][hybrid-ha].</span></span>

<span data-ttu-id="ed440-164">Küllős topológiát használhat átjáró nélkül is, ha nincs szükség a helyszíni hálózattal való kapcsolatra.</span><span class="sxs-lookup"><span data-stu-id="ed440-164">A hub-spoke topology can also be used without a gateway, if you don't need connectivity with your on-premises network.</span></span> 

### <a name="vnet-peering"></a><span data-ttu-id="ed440-165">Társviszony létesítése virtuális hálózatok között</span><span class="sxs-lookup"><span data-stu-id="ed440-165">VNet peering</span></span>

<span data-ttu-id="ed440-166">A virtuális társhálózatok létesítése nem tranzitív kapcsolat két virtuális hálózat között.</span><span class="sxs-lookup"><span data-stu-id="ed440-166">VNet peering is a non-transitive relationship between two VNets.</span></span> <span data-ttu-id="ed440-167">Ha két küllőt szeretne egymáshoz csatlakoztatni, vegye fontolóra egy külön társviszonykapcsolat hozzáadását az adott küllők között.</span><span class="sxs-lookup"><span data-stu-id="ed440-167">If you require spokes to connect to each other, consider adding a separate peering connection between those spokes.</span></span>

<span data-ttu-id="ed440-168">Ha azonban több küllőt is szeretne egymáshoz csatlakoztatni, gyorsan el fognak fogyni a rendelkezésre álló társviszonykapcsolatok a [virtuális hálózatonként létesíthető társviszonykapcsolatok számának korlátozása][vnet-peering-limit] miatt.</span><span class="sxs-lookup"><span data-stu-id="ed440-168">However, if you have several spokes that need to connect with each other, you will run out of possible peering connections very quickly due to the [limitation on number of VNets peerings per VNet][vnet-peering-limit].</span></span> <span data-ttu-id="ed440-169">Ilyenkor vegye fontolóra a felhasználó által megadott útvonalak (UDR-ek) használatát. Ezekkel kényszerítheti, hogy az egy adott küllőre irányuló forgalmat a rendszer egy NVA-ra küldje, amely útválasztóként működik az agyi virtuális hálózaton.</span><span class="sxs-lookup"><span data-stu-id="ed440-169">In this scenario, consider using user defined routes (UDRs) to force traffic destined to a spoke to be sent to an NVA acting as a router at the hub VNet.</span></span> <span data-ttu-id="ed440-170">Ez lehetővé teszi, hogy a küllők kapcsolódjanak egymáshoz.</span><span class="sxs-lookup"><span data-stu-id="ed440-170">This will allow the spokes to connect to each other.</span></span>

<span data-ttu-id="ed440-171">Ezenkívül konfigurálhatja is a küllőket, hogy használják az agyi virtuális hálózatot a távoli hálózatokkal való kommunikációra.</span><span class="sxs-lookup"><span data-stu-id="ed440-171">You can also configure spokes to use the hub VNet gateway to communicate with remote networks.</span></span> <span data-ttu-id="ed440-172">Ahhoz, hogy az átjáróforgalom működjön a küllő és az agy között, a távoli hálózatok pedig kapcsolódjanak, a következők szükségesek:</span><span class="sxs-lookup"><span data-stu-id="ed440-172">To allow gateway traffic to flow from spoke to hub, and connect to remote networks, you must:</span></span>

  - <span data-ttu-id="ed440-173">Konfigurálja a virtuális társhálózat-kapcsolatot az agyban az **átjáróforgalom engedélyezéséhez**.</span><span class="sxs-lookup"><span data-stu-id="ed440-173">Configure the VNet peering connection in the hub to **allow gateway transit**.</span></span>
  - <span data-ttu-id="ed440-174">Konfigurálja a virtuális társhálózat-kapcsolatot minden egyes küllőben a **távoli átjárók** használatához.</span><span class="sxs-lookup"><span data-stu-id="ed440-174">Configure the VNet peering connection in each spoke to **use remote gateways**.</span></span>
  - <span data-ttu-id="ed440-175">Konfiguráljon minden virtuális társhálózat-kapcsolatot a **továbbított forgalom engedélyezéséhez**.</span><span class="sxs-lookup"><span data-stu-id="ed440-175">Configure all VNet peering connections to **allow forwarded traffic**.</span></span>

## <a name="considerations"></a><span data-ttu-id="ed440-176">Megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="ed440-176">Considerations</span></span>

### <a name="spoke-connectivity"></a><span data-ttu-id="ed440-177">Küllőkapcsolat</span><span class="sxs-lookup"><span data-stu-id="ed440-177">Spoke connectivity</span></span>

<span data-ttu-id="ed440-178">Ha küllők között szeretne kapcsolatot létrehozni, fontolja meg egy NVA implementálását az agyban történő útválasztáshoz, illetve az UDR-ek használatát a küllőkben a forgalom továbbítása érdekében az agyhoz.</span><span class="sxs-lookup"><span data-stu-id="ed440-178">If you require connectivity between spokes, consider implementing an NVA for routing in the hub, and using UDRs in the spoke to forward traffic to the hub.</span></span>

<span data-ttu-id="ed440-179">![[2]][2]</span><span class="sxs-lookup"><span data-stu-id="ed440-179">![[2]][2]</span></span>

<span data-ttu-id="ed440-180">Ilyenkor konfigurálnia kell a társviszony-kapcsolatokat a **továbbított forgalom engedélyezéséhez**.</span><span class="sxs-lookup"><span data-stu-id="ed440-180">In this scenario, you must configure the peering connections to **allow forwarded traffic**.</span></span>

### <a name="overcoming-vnet-peering-limits"></a><span data-ttu-id="ed440-181">A virtuális társhálózatok létesítési korlátjának kiküszöbölése</span><span class="sxs-lookup"><span data-stu-id="ed440-181">Overcoming VNet peering limits</span></span>

<span data-ttu-id="ed440-182">Mindenképpen vegye figyelembe a [virtuális hálózatonként létesíthető társviszonyok számának korlátozását][vnet-peering-limit] az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="ed440-182">Make sure you consider the [limitation on number of VNets peerings per VNet][vnet-peering-limit] in Azure.</span></span> <span data-ttu-id="ed440-183">Ha több küllőre van szükség, mint amit a korlátozás megenged, fontolja meg egy agy-küllő-agy-küllő topológia létrehozását, ahol a küllők első szintje agyként is működik.</span><span class="sxs-lookup"><span data-stu-id="ed440-183">If you decide you need more spokes than the limit will allow, consider creating a hub-spoke-hub-spoke topology, where the first level of spokes also act as hubs.</span></span> <span data-ttu-id="ed440-184">Az alábbi diagram ezt a megközelítést mutatja be.</span><span class="sxs-lookup"><span data-stu-id="ed440-184">The following diagram shows this approach.</span></span>

<span data-ttu-id="ed440-185">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="ed440-185">![[3]][3]</span></span>

<span data-ttu-id="ed440-186">Ezenkívül vegye figyelembe, mely szolgáltatások vannak megosztva az agyban. Ezzel biztosítható, hogy az agy több küllőre is skálázzon.</span><span class="sxs-lookup"><span data-stu-id="ed440-186">Also consider what services are shared in the hub, to ensure the hub scales for a larger number of spokes.</span></span> <span data-ttu-id="ed440-187">Például ha az agy tűzfalszolgáltatásokat is nyújt, vegye figyelembe a tűzfal megoldás sávszélességi korlátozásait több küllő hozzáadása esetén.</span><span class="sxs-lookup"><span data-stu-id="ed440-187">For instance, if your hub provides firewall services, consider the bandwidth limits of your firewall solution when adding multiple spokes.</span></span> <span data-ttu-id="ed440-188">Lehet, hogy érdemes néhány megosztott szolgáltatást az agyak második szintjére mozgatni.</span><span class="sxs-lookup"><span data-stu-id="ed440-188">You might want to move some of these shared services to a second level of hubs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="ed440-189">A megoldás üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="ed440-189">Deploy the solution</span></span>

<span data-ttu-id="ed440-190">Ennek az architektúrának egy üzemelő példánya elérhető a [GitHubon][ref-arch-repo].</span><span class="sxs-lookup"><span data-stu-id="ed440-190">A deployment for this architecture is available on [GitHub][ref-arch-repo].</span></span> <span data-ttu-id="ed440-191">Használja a virtuális gépek minden egyes virtuális kapcsolatának tesztelése.</span><span class="sxs-lookup"><span data-stu-id="ed440-191">It uses VMs in each VNet to test connectivity.</span></span> <span data-ttu-id="ed440-192">Az **agyi virtuális hálózat** **megosztott szolgáltatási** alhálózatában nincsenek tárolva tényleges szolgáltatások.</span><span class="sxs-lookup"><span data-stu-id="ed440-192">There are no actual services hosted in the **shared-services** subnet in the **hub VNet**.</span></span>

<span data-ttu-id="ed440-193">A központi telepítés az előfizetésében hoz létre a következő erőforrás-csoportok:</span><span class="sxs-lookup"><span data-stu-id="ed440-193">The deployment creates the following resource groups in your subscription:</span></span>

- <span data-ttu-id="ed440-194">hub-nva-rg</span><span class="sxs-lookup"><span data-stu-id="ed440-194">hub-nva-rg</span></span>
- <span data-ttu-id="ed440-195">hub-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="ed440-195">hub-vnet-rg</span></span>
- <span data-ttu-id="ed440-196">a helyi üzemeltetésű-jb-rg</span><span class="sxs-lookup"><span data-stu-id="ed440-196">onprem-jb-rg</span></span>
- <span data-ttu-id="ed440-197">a helyi üzemeltetésű-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="ed440-197">onprem-vnet-rg</span></span>
- <span data-ttu-id="ed440-198">spoke1-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="ed440-198">spoke1-vnet-rg</span></span>
- <span data-ttu-id="ed440-199">spoke2-nyílás-rg</span><span class="sxs-lookup"><span data-stu-id="ed440-199">spoke2-vent-rg</span></span>

<span data-ttu-id="ed440-200">A sablonfájlokat paraméter tekintse meg ezeket a neveket, így módosítja őket, ha a paraméter fájlok frissítése az egyeztetéshez.</span><span class="sxs-lookup"><span data-stu-id="ed440-200">The template parameter files refer to these names, so if you change them, update the parameter files to match.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="ed440-201">Előfeltételek</span><span class="sxs-lookup"><span data-stu-id="ed440-201">Prerequisites</span></span>

1. <span data-ttu-id="ed440-202">Klónozza, ágaztassa vagy a zip-fájl letöltése a [architektúrák hivatkozhat] [ ref-arch-repo] GitHub-tárházban.</span><span class="sxs-lookup"><span data-stu-id="ed440-202">Clone, fork, or download the zip file for the [reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="ed440-203">Telepítés [Azure CLI 2.0][azure-cli-2].</span><span class="sxs-lookup"><span data-stu-id="ed440-203">Install [Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="ed440-204">Telepítse [az Azure építőelemei][azbb] npm-csomagot.</span><span class="sxs-lookup"><span data-stu-id="ed440-204">Install the [Azure building blocks][azbb] npm package.</span></span>

4. <span data-ttu-id="ed440-205">A parancssorból bash, vagy PowerShell kérdés, jelentkezzen be az Azure-fiókjával az alábbi parancs segítségével.</span><span class="sxs-lookup"><span data-stu-id="ed440-205">From a command prompt, bash prompt, or PowerShell prompt, log into your Azure account by using the command below.</span></span>

   ```bash
   az login
   ```

### <a name="deploy-the-simulated-on-premises-datacenter"></a><span data-ttu-id="ed440-206">A szimulált helyszíni adatközpont üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="ed440-206">Deploy the simulated on-premises datacenter</span></span>

<span data-ttu-id="ed440-207">A szimulált olyan helyszíni adatközpontban egy Azure virtuális hálózatot, központi telepítéséhez kövesse az alábbi lépéseket:</span><span class="sxs-lookup"><span data-stu-id="ed440-207">To deploy the simulated on-premises datacenter as an Azure VNet, follow these steps:</span></span>

1. <span data-ttu-id="ed440-208">Keresse meg a `hybrid-networking/hub-spoke` mappába a referencia architektúra tárház.</span><span class="sxs-lookup"><span data-stu-id="ed440-208">Navigate to the `hybrid-networking/hub-spoke` folder of the reference architectures repository.</span></span>

2. <span data-ttu-id="ed440-209">Nyissa meg az `onprem.json` fájlt.</span><span class="sxs-lookup"><span data-stu-id="ed440-209">Open the `onprem.json` file.</span></span> <span data-ttu-id="ed440-210">Cserélje le a értékeinek `adminUsername` és `adminPassword`.</span><span class="sxs-lookup"><span data-stu-id="ed440-210">Replace the values for `adminUsername` and `adminPassword`.</span></span>

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

3. <span data-ttu-id="ed440-211">(Választható) Linux üzembe helyezés esetén állítsa be `osType` való `Linux`.</span><span class="sxs-lookup"><span data-stu-id="ed440-211">(Optional) For a Linux deployment, set `osType` to `Linux`.</span></span>

4. <span data-ttu-id="ed440-212">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="ed440-212">Run the following command:</span></span>

    ```bash
    azbb -s <subscription_id> -g onprem-vnet-rg -l <location> -p onprem.json --deploy
    ```

5. <span data-ttu-id="ed440-213">Várjon, amíg az üzembe helyezés befejeződik.</span><span class="sxs-lookup"><span data-stu-id="ed440-213">Wait for the deployment to finish.</span></span> <span data-ttu-id="ed440-214">A központi telepítés létrehoz egy virtuális hálózathoz, a virtuális gép és a VPN-átjáró.</span><span class="sxs-lookup"><span data-stu-id="ed440-214">This deployment creates a virtual network, a virtual machine, and a VPN gateway.</span></span> <span data-ttu-id="ed440-215">A VPN-átjáró létrehozásához körülbelül 40 percet is igénybe vehet.</span><span class="sxs-lookup"><span data-stu-id="ed440-215">It can take about 40 minutes to create the VPN gateway.</span></span>

### <a name="deploy-the-hub-vnet"></a><span data-ttu-id="ed440-216">A virtuális hálózat központi telepítése</span><span class="sxs-lookup"><span data-stu-id="ed440-216">Deploy the hub VNet</span></span>

<span data-ttu-id="ed440-217">A virtuális hálózat központi telepítéséhez hajtsa végre az alábbi lépéseket.</span><span class="sxs-lookup"><span data-stu-id="ed440-217">To deploy the hub VNet, perform the following steps.</span></span>

1. <span data-ttu-id="ed440-218">Nyissa meg az `hub-vnet.json` fájlt.</span><span class="sxs-lookup"><span data-stu-id="ed440-218">Open the `hub-vnet.json` file.</span></span> <span data-ttu-id="ed440-219">Cserélje le a értékeinek `adminUsername` és `adminPassword`.</span><span class="sxs-lookup"><span data-stu-id="ed440-219">Replace the values for `adminUsername` and `adminPassword`.</span></span>

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. <span data-ttu-id="ed440-220">(Választható) Linux üzembe helyezés esetén állítsa be `osType` való `Linux`.</span><span class="sxs-lookup"><span data-stu-id="ed440-220">(Optional) For a Linux deployment, set `osType` to `Linux`.</span></span>

3. <span data-ttu-id="ed440-221">A `sharedKey`, adjon meg egy megosztott kulcsot, a VPN-kapcsolathoz.</span><span class="sxs-lookup"><span data-stu-id="ed440-221">For `sharedKey`, enter a shared key for the VPN connection.</span></span> 

    ```bash
    "sharedKey": "",
    ```

4. <span data-ttu-id="ed440-222">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="ed440-222">Run the following command:</span></span>

    ```bash
    azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet.json --deploy
    ```

5. <span data-ttu-id="ed440-223">Várjon, amíg az üzembe helyezés befejeződik.</span><span class="sxs-lookup"><span data-stu-id="ed440-223">Wait for the deployment to finish.</span></span> <span data-ttu-id="ed440-224">A központi telepítéshez létrehoz egy virtuális hálózatot, virtuális gép, VPN-átjáró és az átjáró kapcsolat.</span><span class="sxs-lookup"><span data-stu-id="ed440-224">This deployment creates a virtual network, a virtual machine, a VPN gateway, and a connection to the gateway.</span></span>  <span data-ttu-id="ed440-225">A VPN-átjáró létrehozásához körülbelül 40 percet is igénybe vehet.</span><span class="sxs-lookup"><span data-stu-id="ed440-225">It can take about 40 minutes to create the VPN gateway.</span></span>

### <a name="test-connectivity-with-the-hub"></a><span data-ttu-id="ed440-226">A hubbal kapcsolatának tesztelése</span><span class="sxs-lookup"><span data-stu-id="ed440-226">Test connectivity with the hub</span></span>

<span data-ttu-id="ed440-227">A szimulált a helyszíni környezetből szeretne az elosztóhoz VNet kapcsolat tesztelése.</span><span class="sxs-lookup"><span data-stu-id="ed440-227">Test conectivity from the simulated on-premises environment to the hub VNet.</span></span>

<span data-ttu-id="ed440-228">**Windows központi telepítése**</span><span class="sxs-lookup"><span data-stu-id="ed440-228">**Windows deployment**</span></span>

1. <span data-ttu-id="ed440-229">Az Azure portál segítségével nevű virtuális gép található `jb-vm1` a a `onprem-jb-rg` erőforráscsoportot.</span><span class="sxs-lookup"><span data-stu-id="ed440-229">Use the Azure portal to find the VM named `jb-vm1` in the `onprem-jb-rg` resource group.</span></span>

2. <span data-ttu-id="ed440-230">Kattintson a `Connect` való egy távolítsa el a virtuális asztal munkamenetet nyit meg.</span><span class="sxs-lookup"><span data-stu-id="ed440-230">Click `Connect` to open a remove desktop session to the VM.</span></span> <span data-ttu-id="ed440-231">A megadott jelszó használata a `onprem.json` paraméterfájl.</span><span class="sxs-lookup"><span data-stu-id="ed440-231">Use the password that you specified in the `onprem.json` parameter file.</span></span>

3. <span data-ttu-id="ed440-232">Nyissa meg a PowerShell-konzolban a virtuális Gépet, és használja a `Test-NetConnection` parancsmag futtatásával ellenőrizze, hogy tud-e csatlakozni a virtuális gép jumpbox VNet központban.</span><span class="sxs-lookup"><span data-stu-id="ed440-232">Open a PowerShell console in the VM, and use the `Test-NetConnection` cmdlet to verify that you can connect to the jumpbox VM in the hub VNet.</span></span>

   ```powershell
   Test-NetConnection 10.0.0.68 -CommonTCPPort RDP
   ```
<span data-ttu-id="ed440-233">A kimenet az alábbihoz hasonló kell kinéznie:</span><span class="sxs-lookup"><span data-stu-id="ed440-233">The output should look similar to the following:</span></span>

```powershell
ComputerName     : 10.0.0.68
RemoteAddress    : 10.0.0.68
RemotePort       : 3389
InterfaceAlias   : Ethernet 2
SourceAddress    : 192.168.1.000
TcpTestSucceeded : True
```

> [!NOTE]
> <span data-ttu-id="ed440-234">Alapértelmezés szerint a Windows Server virtuális gépen nem teszik lehetővé az ICMP-válaszokat az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="ed440-234">By default, Windows Server VMs do not allow ICMP responses in Azure.</span></span> <span data-ttu-id="ed440-235">Ha a használni kívánt `ping` kapcsolat tesztelése, az egyes virtuális gépek ICMP-forgalmat a Windows speciális tűzfalon engedélyezni kell.</span><span class="sxs-lookup"><span data-stu-id="ed440-235">If you want to use `ping` to test connectivity, you need to enable ICMP traffic in the Windows Advanced Firewall for each VM.</span></span>

<span data-ttu-id="ed440-236">**Linux-telepítés**</span><span class="sxs-lookup"><span data-stu-id="ed440-236">**Linux deployment**</span></span>

1. <span data-ttu-id="ed440-237">Az Azure portál segítségével nevű virtuális gép található `jb-vm1` a a `onprem-jb-rg` erőforráscsoportot.</span><span class="sxs-lookup"><span data-stu-id="ed440-237">Use the Azure portal to find the VM named `jb-vm1` in the `onprem-jb-rg` resource group.</span></span>

2. <span data-ttu-id="ed440-238">Kattintson a `Connect` , és másolja a `ssh` parancs jelenik meg a portálon.</span><span class="sxs-lookup"><span data-stu-id="ed440-238">Click `Connect` and copy the `ssh` command shown in the portal.</span></span> 

3. <span data-ttu-id="ed440-239">A Linux parancssorból futtassa `ssh` a szimulált helyszíni környezetben való kapcsolódáshoz.</span><span class="sxs-lookup"><span data-stu-id="ed440-239">From a Linux prompt, run `ssh` to connect to the simulated on-premises environment.</span></span> <span data-ttu-id="ed440-240">A megadott jelszó használata a `onprem.json` paraméterfájl.</span><span class="sxs-lookup"><span data-stu-id="ed440-240">Use the password that you specified in the `onprem.json` parameter file.</span></span>

4. <span data-ttu-id="ed440-241">Használja a `ping` tesztelése a virtuális gép jumpbox VNet központban parancsot:</span><span class="sxs-lookup"><span data-stu-id="ed440-241">Use the `ping` command to test connectivity to the jumpbox VM in the hub VNet:</span></span>

   ```bash
   ping 10.0.0.68
   ```

### <a name="deploy-the-spoke-vnets"></a><span data-ttu-id="ed440-242">A Vnetek küllős telepítése</span><span class="sxs-lookup"><span data-stu-id="ed440-242">Deploy the spoke VNets</span></span>

<span data-ttu-id="ed440-243">A Vnetek küllős telepítéséhez hajtsa végre az alábbi lépéseket.</span><span class="sxs-lookup"><span data-stu-id="ed440-243">To deploy the spoke VNets, perform the following steps.</span></span>

1. <span data-ttu-id="ed440-244">Nyissa meg az `spoke1.json` fájlt.</span><span class="sxs-lookup"><span data-stu-id="ed440-244">Open the `spoke1.json` file.</span></span> <span data-ttu-id="ed440-245">Cserélje le a értékeinek `adminUsername` és `adminPassword`.</span><span class="sxs-lookup"><span data-stu-id="ed440-245">Replace the values for `adminUsername` and `adminPassword`.</span></span>

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. <span data-ttu-id="ed440-246">(Választható) Linux üzembe helyezés esetén állítsa be `osType` való `Linux`.</span><span class="sxs-lookup"><span data-stu-id="ed440-246">(Optional) For a Linux deployment, set `osType` to `Linux`.</span></span>

3. <span data-ttu-id="ed440-247">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="ed440-247">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke1-vnet-rg -l <location> -p spoke1.json --deploy
   ```
  
4. <span data-ttu-id="ed440-248">Ismételje meg az 1-2 a `spoke2.json` fájlt.</span><span class="sxs-lookup"><span data-stu-id="ed440-248">Repeat steps 1-2 for the `spoke2.json` file.</span></span>

5. <span data-ttu-id="ed440-249">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="ed440-249">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke2-vnet-rg -l <location> -p spoke2.json --deploy
   ```

6. <span data-ttu-id="ed440-250">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="ed440-250">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet-peering.json --deploy
   ```

### <a name="test-connectivity"></a><span data-ttu-id="ed440-251">Kapcsolat tesztelése</span><span class="sxs-lookup"><span data-stu-id="ed440-251">Test connectivity</span></span>

<span data-ttu-id="ed440-252">A szimulált a helyszíni környezetből a Vnetek küllős való kapcsolat teszteléséhez.</span><span class="sxs-lookup"><span data-stu-id="ed440-252">Test conectivity from the simulated on-premises environment to the spoke VNets.</span></span>

<span data-ttu-id="ed440-253">**Windows központi telepítése**</span><span class="sxs-lookup"><span data-stu-id="ed440-253">**Windows deployment**</span></span>

1. <span data-ttu-id="ed440-254">Az Azure portál segítségével nevű virtuális gép található `jb-vm1` a a `onprem-jb-rg` erőforráscsoportot.</span><span class="sxs-lookup"><span data-stu-id="ed440-254">Use the Azure portal to find the VM named `jb-vm1` in the `onprem-jb-rg` resource group.</span></span>

2. <span data-ttu-id="ed440-255">Kattintson a `Connect` való egy távolítsa el a virtuális asztal munkamenetet nyit meg.</span><span class="sxs-lookup"><span data-stu-id="ed440-255">Click `Connect` to open a remove desktop session to the VM.</span></span> <span data-ttu-id="ed440-256">A megadott jelszó használata a `onprem.json` paraméterfájl.</span><span class="sxs-lookup"><span data-stu-id="ed440-256">Use the password that you specified in the `onprem.json` parameter file.</span></span>

3. <span data-ttu-id="ed440-257">Nyissa meg a PowerShell-konzolban a virtuális Gépet, és használja a `Test-NetConnection` parancsmag futtatásával ellenőrizze, hogy tud-e csatlakozni a virtuális gép jumpbox VNet központban.</span><span class="sxs-lookup"><span data-stu-id="ed440-257">Open a PowerShell console in the VM, and use the `Test-NetConnection` cmdlet to verify that you can connect to the jumpbox VM in the hub VNet.</span></span>

   ```powershell
   Test-NetConnection 10.1.0.68 -CommonTCPPort RDP
   Test-NetConnection 10.2.0.68 -CommonTCPPort RDP
   ```

<span data-ttu-id="ed440-258">**Linux-telepítés**</span><span class="sxs-lookup"><span data-stu-id="ed440-258">**Linux deployment**</span></span>

<span data-ttu-id="ed440-259">A szimulált a helyszíni környezetből a küllős Vnetek Linux virtuális gépek használata a kapcsolat, hajtsa végre az alábbi lépéseket:</span><span class="sxs-lookup"><span data-stu-id="ed440-259">To test conectivity from the simulated on-premises environment to the spoke VNets using Linux VMs, perform the following steps:</span></span>

1. <span data-ttu-id="ed440-260">Az Azure portál segítségével nevű virtuális gép található `jb-vm1` a a `onprem-jb-rg` erőforráscsoportot.</span><span class="sxs-lookup"><span data-stu-id="ed440-260">Use the Azure portal to find the VM named `jb-vm1` in the `onprem-jb-rg` resource group.</span></span>

2. <span data-ttu-id="ed440-261">Kattintson a `Connect` , és másolja a `ssh` parancs jelenik meg a portálon.</span><span class="sxs-lookup"><span data-stu-id="ed440-261">Click `Connect` and copy the `ssh` command shown in the portal.</span></span> 

3. <span data-ttu-id="ed440-262">A Linux parancssorból futtassa `ssh` a szimulált helyszíni környezetben való kapcsolódáshoz.</span><span class="sxs-lookup"><span data-stu-id="ed440-262">From a Linux prompt, run `ssh` to connect to the simulated on-premises environment.</span></span> <span data-ttu-id="ed440-263">A megadott jelszó használata a `onprem.json` paraméterfájl.</span><span class="sxs-lookup"><span data-stu-id="ed440-263">Use the password that you specified in the `onprem.json` parameter file.</span></span>

5. <span data-ttu-id="ed440-264">Használja a `ping` a virtuális gépek jumpbox kapcsolat tesztelése az egyes küllős parancs:</span><span class="sxs-lookup"><span data-stu-id="ed440-264">Use the `ping` command to test connectivity to the jumpbox VMs in each spoke:</span></span>

   ```bash
   ping 10.1.0.68
   ping 10.2.0.68
   ```

### <a name="add-connectivity-between-spokes"></a><span data-ttu-id="ed440-265">Kapcsolat hozzáadása küllők között</span><span class="sxs-lookup"><span data-stu-id="ed440-265">Add connectivity between spokes</span></span>

<span data-ttu-id="ed440-266">Ez a lépés nem kötelező.</span><span class="sxs-lookup"><span data-stu-id="ed440-266">This step is optional.</span></span> <span data-ttu-id="ed440-267">Küllők egymáshoz való csatlakozáshoz engedélyezni szeretné, ha egy newtwork virtuális készülék (NVA) használja a virtuális hálózat központban útválasztóként, és az kényszerítése forgalom a küllők az útválasztó egy másik küllős való kapcsolódási kísérlet során.</span><span class="sxs-lookup"><span data-stu-id="ed440-267">If you want to allow spokes to connect to each other, you must use a newtwork virtual appliance (NVA) as a router in the hub VNet, and force traffic from spokes to the router when trying to connect to another spoke.</span></span> <span data-ttu-id="ed440-268">Egy alapszintű egyetlen virtuális NVA mintaalkalmazás telepítését, felhasználó által definiált útvonalak (udr-EK) engedélyezéséhez a kettő együtt küllő Vnetek csatlakozni, hajtsa végre a következő lépéseket:</span><span class="sxs-lookup"><span data-stu-id="ed440-268">To deploy a basic sample NVA as a single VM, along with user-defined routes (UDRs) to allow the two spoke VNets to connect, perform the following steps:</span></span>

1. <span data-ttu-id="ed440-269">Nyissa meg az `hub-nva.json` fájlt.</span><span class="sxs-lookup"><span data-stu-id="ed440-269">Open the `hub-nva.json` file.</span></span> <span data-ttu-id="ed440-270">Cserélje le a értékeinek `adminUsername` és `adminPassword`.</span><span class="sxs-lookup"><span data-stu-id="ed440-270">Replace the values for `adminUsername` and `adminPassword`.</span></span>

    ```bash
    "adminUsername": "<user name>",
    "adminPassword": "<password>",
    ```

2. <span data-ttu-id="ed440-271">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="ed440-271">Run the following command:</span></span>

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
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md

[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/hub-spoke.png "Küllős topológia az Azure-ban"
[1]: ./images/hub-spoke-gateway-routing.svg "Küllős topológia az Azure-ban tranzitív útválasztással"
[2]: ./images/hub-spoke-no-gateway-routing.svg "Küllős topológia az Azure-ban tranzitív útválasztással, NVA használatával"
[3]: ./images/hub-spokehub-spoke.svg "Küllős-küllős topológia az Azure-ban"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
