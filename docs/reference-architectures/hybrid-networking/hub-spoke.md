---
title: "A hálózati hub-küllős topológia végrehajtása az Azure-ban"
description: "Hogyan egy hub-küllős hálózati topológia végrehajtásához az Azure-ban."
author: telmosampaio
ms.date: 05/05/2017
pnp.series.title: Implement a hub-spoke network topology in Azure
pnp.series.prev: expressroute
ms.openlocfilehash: e6f07a7962dd5728226b023700268340590d97a3
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="implement-a-hub-spoke-network-topology-in-azure"></a><span data-ttu-id="f278c-103">A hálózati hub-küllős topológia végrehajtása az Azure-ban</span><span class="sxs-lookup"><span data-stu-id="f278c-103">Implement a hub-spoke network topology in Azure</span></span>

<span data-ttu-id="f278c-104">A referencia-architektúrában bemutatja, hogyan egy hub-küllős topológia végrehajtásához az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="f278c-104">This reference architecture shows how to implement a hub-spoke topology in Azure.</span></span> <span data-ttu-id="f278c-105">A *hub* van egy virtuális hálózatot (VNet), amely különbséglemezként funkcionál középpontja a helyszíni hálózatot az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="f278c-105">The *hub* is a virtual network (VNet) in Azure that acts as a central point of connectivity to your on-premises network.</span></span> <span data-ttu-id="f278c-106">A *küllők* társkapcsolatot létesíteni a központ, amely segítségével a feladatok elkülönítése céljából Vnetek vannak.</span><span class="sxs-lookup"><span data-stu-id="f278c-106">The *spokes* are VNets that peer with the hub, and can be used to isolate workloads.</span></span> <span data-ttu-id="f278c-107">A forgalom a helyszíni adatközpontját és ExpressRoute- vagy VPN-átjáró kapcsolaton keresztül a központ között.</span><span class="sxs-lookup"><span data-stu-id="f278c-107">Traffic flows between the on-premises datacenter and the hub through an ExpressRoute or VPN gateway connection.</span></span>  <span data-ttu-id="f278c-108">[**Ez a megoldás üzembe helyezéséhez**](#deploy-the-solution).</span><span class="sxs-lookup"><span data-stu-id="f278c-108">[**Deploy this solution**](#deploy-the-solution).</span></span>

<span data-ttu-id="f278c-109">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="f278c-109">![[0]][0]</span></span>

<span data-ttu-id="f278c-110">*Töltse le a [Visio fájl] [ visio-download] ezen architektúra*</span><span class="sxs-lookup"><span data-stu-id="f278c-110">*Download a [Visio file][visio-download] of this architecture*</span></span>


<span data-ttu-id="f278c-111">A toplogy előnyei a következők:</span><span class="sxs-lookup"><span data-stu-id="f278c-111">The benefits of this toplogy include:</span></span>

* <span data-ttu-id="f278c-112">**Költségmegtakarításhoz** központosításával szolgáltatásokról, amelyek több munkaterhelés, például a hálózati virtuális készülékek (NVAs) és a DNS-kiszolgálók, egyetlen helyen megoszthatók.</span><span class="sxs-lookup"><span data-stu-id="f278c-112">**Cost savings** by centralizing services that can be shared by multiple workloads, such as network virtual appliances (NVAs) and DNS servers, in a single location.</span></span>
* <span data-ttu-id="f278c-113">**Előfizetések korlátok kapcsolatos** társviszony-létesítési Vnetekhez különböző előfizetésből a központi hub által.</span><span class="sxs-lookup"><span data-stu-id="f278c-113">**Overcome subscriptions limits** by peering VNets from different subscriptions to the central hub.</span></span>
* <span data-ttu-id="f278c-114">**Aggályokat elkülönítése** közötti központi informatikai (SecOps, InfraOps) és a munkaterhelések (DevOps).</span><span class="sxs-lookup"><span data-stu-id="f278c-114">**Separation of concerns** between central IT (SecOps, InfraOps) and workloads (DevOps).</span></span>

<span data-ttu-id="f278c-115">Ez az architektúra a jellemző használati többek között:</span><span class="sxs-lookup"><span data-stu-id="f278c-115">Typical uses for this architecture include:</span></span>

* <span data-ttu-id="f278c-116">Különböző környezetekben, például a fejlesztési, tesztelési és éles, üzembe helyezett munkaterhelések tekintetében, amely esetében a megosztott szolgáltatások, például a DNS, az Azonosítók, az NTP vagy az AD DS.</span><span class="sxs-lookup"><span data-stu-id="f278c-116">Workloads deployed in different environments, such as development, testing, and production, that require shared services such as DNS, IDS, NTP, or AD DS.</span></span> <span data-ttu-id="f278c-117">Megosztott szolgáltatások kerülnek a hub VNet, amíg minden környezetben telepít egy küllős elkülönítési fenntartásához.</span><span class="sxs-lookup"><span data-stu-id="f278c-117">Shared services are placed in the hub VNet, while each environment is deployed to a spoke to maintain isolation.</span></span>
* <span data-ttu-id="f278c-118">Egymással kapcsolatot nem igényelnek, de a megosztott szolgáltatások elérését igénylő munkaterhelések.</span><span class="sxs-lookup"><span data-stu-id="f278c-118">Workloads that do not require connectivity to each other, but require access to shared services.</span></span>
* <span data-ttu-id="f278c-119">A vállalatok, amelyek központi ellenőrzése alatt tartja a biztonsági szempontjait, például egy tűzfal DMZ, mint a központban igényelnek, és minden küllős lévő munkaterhelések felügyeleti elkülönített.</span><span class="sxs-lookup"><span data-stu-id="f278c-119">Enterprises that require central control over security aspects, such as a firewall in the hub as a DMZ, and segregated management for the workloads in each spoke.</span></span>

## <a name="architecture"></a><span data-ttu-id="f278c-120">Architektúra</span><span class="sxs-lookup"><span data-stu-id="f278c-120">Architecture</span></span>

<span data-ttu-id="f278c-121">Az architektúra a következő összetevőkből áll.</span><span class="sxs-lookup"><span data-stu-id="f278c-121">The architecture consists of the following components.</span></span>

* <span data-ttu-id="f278c-122">**A helyszíni hálózat**.</span><span class="sxs-lookup"><span data-stu-id="f278c-122">**On-premises network**.</span></span> <span data-ttu-id="f278c-123">Helyi magánhálózat fut egy szervezeten belül.</span><span class="sxs-lookup"><span data-stu-id="f278c-123">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="f278c-124">**VPN-eszköz**.</span><span class="sxs-lookup"><span data-stu-id="f278c-124">**VPN device**.</span></span> <span data-ttu-id="f278c-125">Egy eszköz vagy a helyszíni hálózathoz külső kapcsolatot biztosító szolgáltatás.</span><span class="sxs-lookup"><span data-stu-id="f278c-125">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="f278c-126">A VPN-eszköz lehet olyan hardvereszköz, vagy egy szoftveres megoldás, mint például az Útválasztás és távelérés szolgáltatás (RRAS) a Windows Server 2012-ben.</span><span class="sxs-lookup"><span data-stu-id="f278c-126">The VPN device may be a hardware device, or a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="f278c-127">Támogatott VPN-készülék és konfigurálásáról a kiválasztott VPN készülékek Azure történő kapcsolódáshoz listájáért lásd: [kapcsolatos VPN-eszközök a webhelyek közötti VPN átjáró kapcsolatok][vpn-appliance].</span><span class="sxs-lookup"><span data-stu-id="f278c-127">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="f278c-128">**VPN virtuális hálózati átjáró vagy ExpressRoute-átjáró**.</span><span class="sxs-lookup"><span data-stu-id="f278c-128">**VPN virtual network gateway or ExpressRoute gateway**.</span></span> <span data-ttu-id="f278c-129">A virtuális hálózati átjáró lehetővé teszi, hogy a virtuális hálózaton a VPN-eszköz vagy ExpressRoute-kapcsolatcsoportot, a helyszíni hálózati kapcsolattal használt való csatlakozáshoz.</span><span class="sxs-lookup"><span data-stu-id="f278c-129">The virtual network gateway enables the VNet to connect to the VPN device, or ExpressRoute circuit, used for connectivity with your on-premises network.</span></span> <span data-ttu-id="f278c-130">További információkért lásd: [egy a helyszíni hálózathoz csatlakozni a Microsoft Azure virtuális hálózat][connect-to-an-Azure-vnet].</span><span class="sxs-lookup"><span data-stu-id="f278c-130">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

> [!NOTE]
> <span data-ttu-id="f278c-131">Az üzembe helyezési parancsfájlok a referenciaarchitektúra használható VPN-átjáró a hálózati kapcsolatot, és egy Vnetet az Azure-ban szimulációjára a helyszíni hálózat.</span><span class="sxs-lookup"><span data-stu-id="f278c-131">The deployment scripts for this reference architecture use a VPN gateway for connectivity, and a VNet in Azure to simulate your on-premises network.</span></span>

* <span data-ttu-id="f278c-132">**Hub VNet**.</span><span class="sxs-lookup"><span data-stu-id="f278c-132">**Hub VNet**.</span></span> <span data-ttu-id="f278c-133">A központ-küllős topológia hub használt Azure virtuális hálózat.</span><span class="sxs-lookup"><span data-stu-id="f278c-133">Azure VNet used as the hub in the hub-spoke topology.</span></span> <span data-ttu-id="f278c-134">A központ középpontja a helyszíni hálózathoz csatlakoznak, és a hely, a különböző terhelésekhez a Vnetek küllős üzemeltetett által felhasználható gazdaszolgáltatásokat.</span><span class="sxs-lookup"><span data-stu-id="f278c-134">The hub is the central point of connectivity to your on-premises network, and a place to host services that can be consumed by the different workloads hosted in the spoke VNets.</span></span>

* <span data-ttu-id="f278c-135">**Átjáró alhálózati**.</span><span class="sxs-lookup"><span data-stu-id="f278c-135">**Gateway subnet**.</span></span> <span data-ttu-id="f278c-136">A virtuális hálózati átjárók ugyanazon az alhálózaton van használatban.</span><span class="sxs-lookup"><span data-stu-id="f278c-136">The virtual network gateways are held in the same subnet.</span></span>

* <span data-ttu-id="f278c-137">**A megosztott szolgáltatások alhálózati**.</span><span class="sxs-lookup"><span data-stu-id="f278c-137">**Shared services subnet**.</span></span> <span data-ttu-id="f278c-138">A virtuális hálózat összes küllők, például a DNS- vagy Active Directory tartományi szolgáltatások között megosztható gazdaszolgáltatásokat használt központban alhálózat.</span><span class="sxs-lookup"><span data-stu-id="f278c-138">A subnet in the hub VNet used to host services that can be shared among all spokes, such as DNS or AD DS.</span></span>

* <span data-ttu-id="f278c-139">**Vnetek küllő**.</span><span class="sxs-lookup"><span data-stu-id="f278c-139">**Spoke VNets**.</span></span> <span data-ttu-id="f278c-140">Egy vagy több Azure Vnetekhez, amely a hub-küllős topológia küllők használatosak.</span><span class="sxs-lookup"><span data-stu-id="f278c-140">One or more Azure VNets that are used as spokes in the hub-spoke topology.</span></span> <span data-ttu-id="f278c-141">Különítse el a saját Vnetek, függetlenül a más küllők felügyelt munkaterhelésekre küllők használható.</span><span class="sxs-lookup"><span data-stu-id="f278c-141">Spokes can be used to isolate workloads in their own VNets, managed separately from other spokes.</span></span> <span data-ttu-id="f278c-142">Egyes alkalmazások és szolgáltatások közé tartozik a többrétegű konfigurációk –, több alhálózattal Azure load Balancer terheléselosztók keresztül csatlakoznak.</span><span class="sxs-lookup"><span data-stu-id="f278c-142">Each workload might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="f278c-143">Az alkalmazás-infrastruktúrával kapcsolatos további információkért lásd: [futó Windows virtuális gép munkaterhelések] [ windows-vm-ra] és [futó Linux virtuális gép munkaterhelések] [ linux-vm-ra].</span><span class="sxs-lookup"><span data-stu-id="f278c-143">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

* <span data-ttu-id="f278c-144">**Vnetben társviszony-létesítés**.</span><span class="sxs-lookup"><span data-stu-id="f278c-144">**VNet peering**.</span></span> <span data-ttu-id="f278c-145">Két azonos Azure-régióban vnetek használatával egy [társviszony-létesítési kapcsolat][vnet-peering].</span><span class="sxs-lookup"><span data-stu-id="f278c-145">Two VNets in the same Azure region can be connected using a [peering connection][vnet-peering].</span></span> <span data-ttu-id="f278c-146">Társviszony-létesítési kapcsolatok nem tranzitív, alacsony késésű kapcsolatok Vnetek között.</span><span class="sxs-lookup"><span data-stu-id="f278c-146">Peering connections are non-transitive, low latency connections between VNets.</span></span> <span data-ttu-id="f278c-147">Ha nincsenek társviszonyban, a Vnetek exchange-forgalom Azure gerincét, nincs szükség az útválasztó használatával.</span><span class="sxs-lookup"><span data-stu-id="f278c-147">Once peered, the VNets exchange traffic by using the Azure backbone, without the need for a router.</span></span> <span data-ttu-id="f278c-148">A hálózati hub-küllős topológiában használhatja a központ kapcsolódni minden egyes küllős hálózatokból.</span><span class="sxs-lookup"><span data-stu-id="f278c-148">In a hub-spoke network topology, you use VNet peering to connect the hub to each spoke.</span></span>

> [!NOTE]
> <span data-ttu-id="f278c-149">Ez a cikk csak magában foglalja az [erőforrás-kezelő](/azure/azure-resource-manager/resource-group-overview) központi telepítések, de is csatlakozhatnak a klasszikus virtuális hálózatot egy erőforrás-kezelő virtuális hálózat ugyanabban az előfizetésben.</span><span class="sxs-lookup"><span data-stu-id="f278c-149">This article only covers [Resource Manager](/azure/azure-resource-manager/resource-group-overview) deployments, but you can also connect a classic VNet to a Resource Manager VNet in the same subscription.</span></span> <span data-ttu-id="f278c-150">Ily módon a küllők lehet üzemeltetni a klasszikus üzembe helyezés-továbbra is igénybe központban megosztott szolgáltatások.</span><span class="sxs-lookup"><span data-stu-id="f278c-150">That way, your spokes can host classic deployments and still benefit from services shared in the hub.</span></span>


## <a name="recommendations"></a><span data-ttu-id="f278c-151">Javaslatok</span><span class="sxs-lookup"><span data-stu-id="f278c-151">Recommendations</span></span>

<span data-ttu-id="f278c-152">Az alábbi javaslatok a legtöbb forgatókönyvre vonatkoznak.</span><span class="sxs-lookup"><span data-stu-id="f278c-152">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="f278c-153">Kövesse ezeket a javaslatokat, ha nincsenek ezeket felülíró követelményei.</span><span class="sxs-lookup"><span data-stu-id="f278c-153">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="resource-groups"></a><span data-ttu-id="f278c-154">Erőforráscsoportok</span><span class="sxs-lookup"><span data-stu-id="f278c-154">Resource groups</span></span>

<span data-ttu-id="f278c-155">A virtuális hálózat, küllős minden VNet megvalósítható másik erőforráscsoport-sablonok, és akár más előfizetések mindaddig, amíg az azonos Azure Active Directory (Azure AD) bérlői azonos Azure-régióban tartoznak.</span><span class="sxs-lookup"><span data-stu-id="f278c-155">The hub VNet, and each spoke VNet, can be implemented in different resource groups, and even different subscriptions, as long as they belong to the same Azure Active Directory (Azure AD) tenant in the same Azure region.</span></span> <span data-ttu-id="f278c-156">Ez lehetővé teszi az egyes munkaterhelések decentralizált felügyeleti szolgáltatások a virtuális hálózat központban kezelt megosztása során.</span><span class="sxs-lookup"><span data-stu-id="f278c-156">This allows for a decentralized management of each workload, while sharing services maintained in the hub VNet.</span></span>

### <a name="vnet-and-gatewaysubnet"></a><span data-ttu-id="f278c-157">Virtuális hálózat és a GatewaySubnet</span><span class="sxs-lookup"><span data-stu-id="f278c-157">VNet and GatewaySubnet</span></span>

<span data-ttu-id="f278c-158">Hozzon létre egy nevű alhálózatot *GatewaySubnet*, a /27 címtartományt.</span><span class="sxs-lookup"><span data-stu-id="f278c-158">Create a subnet named *GatewaySubnet*, with an address range of /27.</span></span> <span data-ttu-id="f278c-159">Ez az alhálózat által a virtuális hálózati átjáró szükséges.</span><span class="sxs-lookup"><span data-stu-id="f278c-159">This subnet is required by the virtual network gateway.</span></span> <span data-ttu-id="f278c-160">Lefoglalásával megelőzése érdekében az alhálózathoz 32 cím segítségével átjáró méretkorlátai a jövőben elérése.</span><span class="sxs-lookup"><span data-stu-id="f278c-160">Allocating 32 addresses to this subnet will help to prevent reaching gateway size limitations in the future.</span></span>

<span data-ttu-id="f278c-161">Az átjáró beállításával kapcsolatos további információkért tekintse meg a következő hivatkozás architektúrák, attól függően, hogy a kapcsolat típusa:</span><span class="sxs-lookup"><span data-stu-id="f278c-161">For more information about setting up the gateway, see the following reference architectures, depending on your connection type:</span></span>

- <span data-ttu-id="f278c-162">[Hibrid hálózati ExpressRoute segítségével][guidance-expressroute]</span><span class="sxs-lookup"><span data-stu-id="f278c-162">[Hybrid network using ExpressRoute][guidance-expressroute]</span></span>
- <span data-ttu-id="f278c-163">[Hibrid hálózathoz egy VPN-átjáró][guidance-vpn]</span><span class="sxs-lookup"><span data-stu-id="f278c-163">[Hybrid network using a VPN gateway][guidance-vpn]</span></span>

<span data-ttu-id="f278c-164">Magas rendelkezésre állás érdekében használhatja az ExpressRoute- és VPN-en a feladatátvételre.</span><span class="sxs-lookup"><span data-stu-id="f278c-164">For higher availability, you can use ExpressRoute plus a VPN for failover.</span></span> <span data-ttu-id="f278c-165">Lásd: [egy a helyszíni hálózat csatlakozik az Azure ExpressRoute segítségével a VPN-feladatátvételi][hybrid-ha].</span><span class="sxs-lookup"><span data-stu-id="f278c-165">See [Connect an on-premises network to Azure using ExpressRoute with VPN failover][hybrid-ha].</span></span>

<span data-ttu-id="f278c-166">A központ-küllős topológia is nélkül használható lesz átjárót, ha nem szükséges kapcsolat a helyszíni hálózattal.</span><span class="sxs-lookup"><span data-stu-id="f278c-166">A hub-spoke topology can also be used without a gateway, if you don't need connectivity with your on-premises network.</span></span> 

### <a name="vnet-peering"></a><span data-ttu-id="f278c-167">Virtuális hálózatok közötti társviszony</span><span class="sxs-lookup"><span data-stu-id="f278c-167">VNet peering</span></span>

<span data-ttu-id="f278c-168">VNet-társviszony létesítése – két Vnetek között nem tranzitív kapcsolat.</span><span class="sxs-lookup"><span data-stu-id="f278c-168">VNet peering is a non-transitive relationship between two VNets.</span></span> <span data-ttu-id="f278c-169">Ha csatlakozni egymáshoz küllők van szüksége, fontolja meg, ezek küllők külön társviszony-létesítési kapcsolatát.</span><span class="sxs-lookup"><span data-stu-id="f278c-169">If you require spokes to connect to each other, consider adding a separate peering connection between those spokes.</span></span>

<span data-ttu-id="f278c-170">Azonban ha több küllők egymással csatlakoztatni, futtatja kívül lehetséges társviszony-létesítési kapcsolatok nagyon gyorsan miatt a [tartozó Vnetek esetében egy virtuális hálózat számára a korlátozás][vnet-peering-limit].</span><span class="sxs-lookup"><span data-stu-id="f278c-170">However, if you have several spokes that need to connect with each other, you will run out of possible peering connections very quickly due to the [limitation on number of VNets peerings per VNet][vnet-peering-limit].</span></span> <span data-ttu-id="f278c-171">Ebben az esetben fontolja meg az felhasználó által megadott útvonalakat (udr-EK) használata egy, a központ VNet útválasztóként NVA küldendő küllős irányuló forgalom kényszerítése.</span><span class="sxs-lookup"><span data-stu-id="f278c-171">In this scenario, consider using user defined routes (UDRs) to force traffic destined to a spoke to be sent to an NVA acting as a router at the hub VNet.</span></span> <span data-ttu-id="f278c-172">Ez lehetővé teszi a küllők csatlakozni egymáshoz.</span><span class="sxs-lookup"><span data-stu-id="f278c-172">This will allow the spokes to connect to each other.</span></span>

<span data-ttu-id="f278c-173">Beállíthatja úgy is küllők kommunikálni a távoli hálózatokhoz a központ virtuális hálózatot átjáró használatára.</span><span class="sxs-lookup"><span data-stu-id="f278c-173">You can also configure spokes to use the hub VNet gateway to communicate with remote networks.</span></span> <span data-ttu-id="f278c-174">Az átjáró forgalmat engedélyezi a küllős hubhoz, és a távoli hálózatokhoz csatlakozhat, meg kell:</span><span class="sxs-lookup"><span data-stu-id="f278c-174">To allow gateway traffic to flow from spoke to hub, and connect to remote networks, you must:</span></span>

  - <span data-ttu-id="f278c-175">A Vnetben társviszony-létesítési kapcsolat konfigurálása a központban **átjáró átvitel engedélyezése**.</span><span class="sxs-lookup"><span data-stu-id="f278c-175">Configure the VNet peering connection in the hub to **allow gateway transit**.</span></span>
  - <span data-ttu-id="f278c-176">A Vnetben társviszony-létesítési kapcsolat konfigurálása az egyes küllős **távoli átjárókat használ**.</span><span class="sxs-lookup"><span data-stu-id="f278c-176">Configure the VNet peering connection in each spoke to **use remote gateways**.</span></span>
  - <span data-ttu-id="f278c-177">Minden Vnetben társviszony-létesítési létesített kapcsolatok konfigurálására **lehetővé a továbbított forgalom**.</span><span class="sxs-lookup"><span data-stu-id="f278c-177">Configure all VNet peering connections to **allow forwarded traffic**.</span></span>

## <a name="considerations"></a><span data-ttu-id="f278c-178">Megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="f278c-178">Considerations</span></span>

### <a name="spoke-connectivity"></a><span data-ttu-id="f278c-179">Kapcsolat küllő</span><span class="sxs-lookup"><span data-stu-id="f278c-179">Spoke connectivity</span></span>

<span data-ttu-id="f278c-180">Ha küllők közötti kapcsolat van szüksége, vegye fontolóra NVA központban útválasztáshoz, és a küllős udr-EK segítségével szeretne az elosztóhoz forgalom továbbítására.</span><span class="sxs-lookup"><span data-stu-id="f278c-180">If you require connectivity between spokes, consider implementing an NVA for routing in the hub, and using UDRs in the spoke to forward traffic to the hub.</span></span>

<span data-ttu-id="f278c-181">![[2]][2]</span><span class="sxs-lookup"><span data-stu-id="f278c-181">![[2]][2]</span></span>

<span data-ttu-id="f278c-182">Ebben az esetben konfigurálnia kell a társviszony-létesítési kapcsolatok **lehetővé a továbbított forgalom**.</span><span class="sxs-lookup"><span data-stu-id="f278c-182">In this scenario, you must configure the peering connections to **allow forwarded traffic**.</span></span>

### <a name="overcoming-vnet-peering-limits"></a><span data-ttu-id="f278c-183">Hibás a Vnetben társviszony-létesítési korlátok</span><span class="sxs-lookup"><span data-stu-id="f278c-183">Overcoming VNet peering limits</span></span>

<span data-ttu-id="f278c-184">Mindenképpen vegye figyelembe a [tartozó Vnetek esetében egy virtuális hálózat számára a korlátozás] [ vnet-peering-limit] az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="f278c-184">Make sure you consider the [limitation on number of VNets peerings per VNet][vnet-peering-limit] in Azure.</span></span> <span data-ttu-id="f278c-185">Ha úgy dönt, mint a korlát lehetővé teszi több küllők van szüksége, fontolja meg egy hub-küllős-hub-küllős topológia, ahol az első szintű küllők is összekötőként hubok létrehozása.</span><span class="sxs-lookup"><span data-stu-id="f278c-185">If you decide you need more spokes than the limit will allow, consider creating a hub-spoke-hub-spoke topology, where the first level of spokes also act as hubs.</span></span> <span data-ttu-id="f278c-186">Az alábbi ábrán látható, ezt a módszert használja.</span><span class="sxs-lookup"><span data-stu-id="f278c-186">The following diagram shows this approach.</span></span>

<span data-ttu-id="f278c-187">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="f278c-187">![[3]][3]</span></span>

<span data-ttu-id="f278c-188">Figyelembe venni az milyen szolgáltatásokat megosztott központban, annak érdekében, a központ méretezi küllők nagyobb számú.</span><span class="sxs-lookup"><span data-stu-id="f278c-188">Also consider what services are shared in the hub, to ensure the hub scales for a larger number of spokes.</span></span> <span data-ttu-id="f278c-189">Például ha a központ tűzfal szolgáltatásokat nyújt, fontolja meg a sávszélesség korlátja a tűzfal megoldás több küllők hozzáadásakor.</span><span class="sxs-lookup"><span data-stu-id="f278c-189">For instance, if your hub provides firewall services, consider the bandwidth limits of your firewall solution when adding multiple spokes.</span></span> <span data-ttu-id="f278c-190">Érdemes a megosztott szolgáltatások hubok a második szintű át.</span><span class="sxs-lookup"><span data-stu-id="f278c-190">You might want to move some of these shared services to a second level of hubs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="f278c-191">A megoldás üzembe helyezéséhez</span><span class="sxs-lookup"><span data-stu-id="f278c-191">Deploy the solution</span></span>

<span data-ttu-id="f278c-192">Ez az architektúra telepítésének érhető el a [GitHub][ref-arch-repo].</span><span class="sxs-lookup"><span data-stu-id="f278c-192">A deployment for this architecture is available on [GitHub][ref-arch-repo].</span></span> <span data-ttu-id="f278c-193">Kapcsolat tesztelése Ubuntu virtuális gépeket használ minden egyes virtuális.</span><span class="sxs-lookup"><span data-stu-id="f278c-193">It uses Ubuntu VMs in each VNet to test connectivity.</span></span> <span data-ttu-id="f278c-194">Nincsenek tárolva tényleges szolgáltatások a **megosztott szolgáltatások** alhálózatának a **hub VNet**.</span><span class="sxs-lookup"><span data-stu-id="f278c-194">There are no actual services hosted in the **shared-services** subnet in the **hub VNet**.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="f278c-195">Előfeltételek</span><span class="sxs-lookup"><span data-stu-id="f278c-195">Prerequisites</span></span>

<span data-ttu-id="f278c-196">Mielőtt a saját előfizetésének telepítése a referencia-architektúrában, a következő lépésekkel kell azt.</span><span class="sxs-lookup"><span data-stu-id="f278c-196">Before you can deploy the reference architecture to your own subscription, you must perform the following steps.</span></span>

1. <span data-ttu-id="f278c-197">Klónozza, ágaztassa vagy a zip-fájl letöltése a [AzureCAT referencia architektúra] [ ref-arch-repo] GitHub-tárházban.</span><span class="sxs-lookup"><span data-stu-id="f278c-197">Clone, fork, or download the zip file for the [AzureCAT reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="f278c-198">Ha szeretné használni az Azure parancssori felület, győződjön meg arról, hogy az Azure CLI 2.0 telepítve a számítógépre.</span><span class="sxs-lookup"><span data-stu-id="f278c-198">If you prefer to use the Azure CLI, make sure you have the Azure CLI 2.0 installed on your computer.</span></span> <span data-ttu-id="f278c-199">A parancssori felület telepítéséhez kövesse az utasításokat a [Azure CLI 2.0 telepítése][azure-cli-2].</span><span class="sxs-lookup"><span data-stu-id="f278c-199">To install the CLI, follow the instructions in [Install Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="f278c-200">Ha jobban szeret PowerShell segítségével, győződjön meg arról, hogy a legújabb PowerShell-modult az Azure-bA telepítve, a számítógépen.</span><span class="sxs-lookup"><span data-stu-id="f278c-200">If you prefer to use PowerShell, make sure you have the latest PowerShell module for Azure installed on you computer.</span></span> <span data-ttu-id="f278c-201">A legújabb Azure PowerShell-modul telepítéséhez kövesse a [az Azure PowerShell telepítése][azure-powershell].</span><span class="sxs-lookup"><span data-stu-id="f278c-201">To install the latest Azure PowerShell module, follow the instructions in [Install PowerShell for Azure][azure-powershell].</span></span>

4. <span data-ttu-id="f278c-202">A parancssorból bash, vagy PowerShell kérdés, jelentkezzen be az Azure-fiókjával használja az alábbi parancsok egyikét, és kövesse az utasításokat.</span><span class="sxs-lookup"><span data-stu-id="f278c-202">From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account by using one of the commands below, and follow the prompts.</span></span>

  ```bash
  az login
  ```

  ```powershell
  Login-AzureRmAccount
  ```

### <a name="deploy-the-simulated-on-premises-datacenter"></a><span data-ttu-id="f278c-203">A szimulált olyan helyszíni adatközpontban telepítése</span><span class="sxs-lookup"><span data-stu-id="f278c-203">Deploy the simulated on-premises datacenter</span></span>

<span data-ttu-id="f278c-204">A szimulált olyan helyszíni adatközpontban, egy Azure virtuális hálózatot telepítéséhez hajtsa végre az alábbi lépéseket.</span><span class="sxs-lookup"><span data-stu-id="f278c-204">To deploy the simulated on-premises datacenter as an Azure VNet, perform the following steps.</span></span>

1. <span data-ttu-id="f278c-205">Keresse meg a `hybrid-networking\hub-spoke\onprem` a fenti előfeltételek lépésben letöltött tárház mappát.</span><span class="sxs-lookup"><span data-stu-id="f278c-205">Navigate to the `hybrid-networking\hub-spoke\onprem` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="f278c-206">Nyissa meg a `onprem.vm.parameters.json` fájlt, és adjon meg egy felhasználónevet és jelszót a mezőkben szereplő idézőjeleket sor 11 és 12 közötti alább látható módon, majd mentse a fájlt.</span><span class="sxs-lookup"><span data-stu-id="f278c-206">Open the `onprem.vm.parameters.json` file and enter a username and password between the quotes in line 11 and 12, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. <span data-ttu-id="f278c-207">Futtassa a bash vagy a PowerShell-parancsot az alábbi történő telepítését a szimulált helyszíni környezetben egy Vnetet az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="f278c-207">Run the bash or PowerShell command below to deploy the simulated on-premises environment as a VNet in Azure.</span></span> <span data-ttu-id="f278c-208">Helyettesítse be az értékeket az előfizetését, erőforráscsoport-név, és az Azure-régió.</span><span class="sxs-lookup"><span data-stu-id="f278c-208">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./onprem.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-onprem-rg \
    --location westus
  ```

  ```powershell
  ./onprem.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-onprem-rg `
    -Location westus
  ```
  > [!NOTE]
  > <span data-ttu-id="f278c-209">Ha úgy dönt, hogy használjon másik erőforráscsoport-nevet (eltérő `ra-onprem-rg`), ügyeljen arra, hogy minden paraméter fájlok használja ezt a nevet, és szerkessze azokat használni a saját erőforráscsoport-név.</span><span class="sxs-lookup"><span data-stu-id="f278c-209">If you decide to use a different resource group name (other than `ra-onprem-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

4. <span data-ttu-id="f278c-210">Várjon, amíg a telepítés befejezéséhez.</span><span class="sxs-lookup"><span data-stu-id="f278c-210">Wait for the deployment to finish.</span></span> <span data-ttu-id="f278c-211">A telepítéshez létrehoz egy virtuális hálózatot, Ubuntu és a VPN-átjáró egy olyan virtuális géphez.</span><span class="sxs-lookup"><span data-stu-id="f278c-211">This deployment creates a virtual network, a virtual machine running Ubuntu, and a VPN gateway.</span></span> <span data-ttu-id="f278c-212">A VPN-átjáró létrehozása több mint 40 percet is igénybe vehet.</span><span class="sxs-lookup"><span data-stu-id="f278c-212">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="azure-hub-vnet"></a><span data-ttu-id="f278c-213">Az Azure hub virtuális hálózat</span><span class="sxs-lookup"><span data-stu-id="f278c-213">Azure hub VNet</span></span>

<span data-ttu-id="f278c-214">A virtuális hálózat központi telepítése, és a fentiekben létrehozott szimulált helyszíni VNet csatlakoznak, hajtsa végre az alábbi lépéseket.</span><span class="sxs-lookup"><span data-stu-id="f278c-214">To deploy the hub VNet, and connect to the simulated on-premises VNet created above, perform the following steps.</span></span>

1. <span data-ttu-id="f278c-215">Keresse meg a `hybrid-networking\hub-spoke\hub` a fenti előfeltételek lépésben letöltött tárház mappát.</span><span class="sxs-lookup"><span data-stu-id="f278c-215">Navigate to the `hybrid-networking\hub-spoke\hub` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="f278c-216">Nyissa meg a `hub.vm.parameters.json` fájlt, és adjon meg egy felhasználónevet és jelszót a mezőkben szereplő idézőjeleket sor 11 és 12 közötti alább látható módon, majd mentse a fájlt.</span><span class="sxs-lookup"><span data-stu-id="f278c-216">Open the `hub.vm.parameters.json` file and enter a username and password between the quotes in line 11 and 12, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. <span data-ttu-id="f278c-217">Nyissa meg a `hub.gateway.parameters.json` fájlt, és adjon meg egy megosztott kulcsot sorban 23, az idézőjelek között alább látható módon, majd mentse a fájlt.</span><span class="sxs-lookup"><span data-stu-id="f278c-217">Open the `hub.gateway.parameters.json` file and enter a shared key between the quotes in line 23, as shown below, then save the file.</span></span> <span data-ttu-id="f278c-218">Jegyezze fel ezt az értéket megtartásához később a központi telepítésben lévő használandó szüksége lesz.</span><span class="sxs-lookup"><span data-stu-id="f278c-218">Keep a note of this value, you will need to use it later in the deployment.</span></span>

  ```bash
  "sharedKey": "",
  ```

4. <span data-ttu-id="f278c-219">Futtassa a bash vagy a PowerShell-parancsot az alábbi történő telepítését a szimulált helyszíni környezetben egy Vnetet az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="f278c-219">Run the bash or PowerShell command below to deploy the simulated on-premises environment as a VNet in Azure.</span></span> <span data-ttu-id="f278c-220">Helyettesítse be az értékeket az előfizetését, erőforráscsoport-név, és az Azure-régió.</span><span class="sxs-lookup"><span data-stu-id="f278c-220">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./hub.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-hub-rg \
    --location westus
  ```

  ```powershell
  ./hub.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-hub-rg `
    -Location westus
  ```
  > [!NOTE]
  > <span data-ttu-id="f278c-221">Ha úgy dönt, hogy használjon másik erőforráscsoport-nevet (eltérő `ra-hub-rg`), ügyeljen arra, hogy minden paraméter fájlok használja ezt a nevet, és szerkessze azokat használni a saját erőforráscsoport-név.</span><span class="sxs-lookup"><span data-stu-id="f278c-221">If you decide to use a different resource group name (other than `ra-hub-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="f278c-222">Várjon, amíg a telepítés befejezéséhez.</span><span class="sxs-lookup"><span data-stu-id="f278c-222">Wait for the deployment to finish.</span></span> <span data-ttu-id="f278c-223">A telepítéshez létrehoz egy virtuális hálózatot, Ubuntu, a VPN-átjáró és a kapcsolatot az átjáró, az előző szakaszban létrehozott futtató virtuális gép.</span><span class="sxs-lookup"><span data-stu-id="f278c-223">This deployment creates a virtual network, a virtual machine running Ubuntu, a VPN gateway, and a connection to the gateway created in the previous section.</span></span> <span data-ttu-id="f278c-224">A VPN-átjáró létrehozása több mint 40 percet is igénybe vehet.</span><span class="sxs-lookup"><span data-stu-id="f278c-224">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="connection-from-on-premises-to-the-hub"></a><span data-ttu-id="f278c-225">Kapcsolat a helyszíni és a központ</span><span class="sxs-lookup"><span data-stu-id="f278c-225">Connection from on-premises to the hub</span></span>

<span data-ttu-id="f278c-226">Csatlakoztassa a szimulált helyszíni adatközpontját a hub VNet, hajtsa végre az alábbi lépéseket.</span><span class="sxs-lookup"><span data-stu-id="f278c-226">To connect from the simulated on-premises datacenter to the hub VNet, perform the following steps.</span></span>

1. <span data-ttu-id="f278c-227">Keresse meg a `hybrid-networking\hub-spoke\onprem` a fenti előfeltételek lépésben letöltött tárház mappát.</span><span class="sxs-lookup"><span data-stu-id="f278c-227">Navigate to the `hybrid-networking\hub-spoke\onprem` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="f278c-228">Nyissa meg a `onprem.connection.parameters.json` fájlt, és adjon meg egy megosztott kulcsot sorban 9, az idézőjelek között alább látható módon, majd mentse a fájlt.</span><span class="sxs-lookup"><span data-stu-id="f278c-228">Open the `onprem.connection.parameters.json` file and enter a shared key between the quotes in line 9, as shown below, then save the file.</span></span> <span data-ttu-id="f278c-229">A megosztott kulcs értéke az a korábban már telepítette a helyszíni átjáró használt azonosnak kell lennie.</span><span class="sxs-lookup"><span data-stu-id="f278c-229">This shared key value must be the same used in the on-premises gateway you deployed previously.</span></span>

  ```bash
  "sharedKey": "",
  ```

3. <span data-ttu-id="f278c-230">Futtassa a bash vagy a PowerShell-parancsot az alábbi történő telepítését a szimulált helyszíni környezetben egy Vnetet az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="f278c-230">Run the bash or PowerShell command below to deploy the simulated on-premises environment as a VNet in Azure.</span></span> <span data-ttu-id="f278c-231">Helyettesítse be az értékeket az előfizetését, erőforráscsoport-név, és az Azure-régió.</span><span class="sxs-lookup"><span data-stu-id="f278c-231">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./onprem.connection.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-onprem-rg \
    --location westus
  ```

  ```powershell
  ./onprem.connection.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-onprem-rg `
    -Location westus
  ```
  > [!NOTE]
  > <span data-ttu-id="f278c-232">Ha úgy dönt, hogy használjon másik erőforráscsoport-nevet (eltérő `ra-onprem-rg`), ügyeljen arra, hogy minden paraméter fájlok használja ezt a nevet, és szerkessze azokat használni a saját erőforráscsoport-név.</span><span class="sxs-lookup"><span data-stu-id="f278c-232">If you decide to use a different resource group name (other than `ra-onprem-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

4. <span data-ttu-id="f278c-233">Várjon, amíg a telepítés befejezéséhez.</span><span class="sxs-lookup"><span data-stu-id="f278c-233">Wait for the deployment to finish.</span></span> <span data-ttu-id="f278c-234">A központi telepítés szimulálása egy helyszíni adatközpontot, és a virtuális hálózat hub használt hálózatok közötti kapcsolatot hoz létre.</span><span class="sxs-lookup"><span data-stu-id="f278c-234">This deployment creates a connection between the VNet used to simulate an on-premises datacenter, and the hub VNet.</span></span>

### <a name="azure-spoke-vnets"></a><span data-ttu-id="f278c-235">Az Azure küllős Vnetek</span><span class="sxs-lookup"><span data-stu-id="f278c-235">Azure spoke VNets</span></span>

<span data-ttu-id="f278c-236">A Vnetek küllős telepíteni, és a fentiekben létrehozott virtuális hálózat központi csatlakozás, hajtsa végre az alábbi lépéseket.</span><span class="sxs-lookup"><span data-stu-id="f278c-236">To deploy the spoke VNets, and connect to the hub VNet created above, perform the following steps.</span></span>

1. <span data-ttu-id="f278c-237">Váltás a `hybrid-networking\hub-spoke\spokes` a fenti előfeltételek lépésben letöltött tárház mappát.</span><span class="sxs-lookup"><span data-stu-id="f278c-237">Switch to the `hybrid-networking\hub-spoke\spokes` folder for the repository you download in the pre-requisites step above.</span></span>

2. <span data-ttu-id="f278c-238">Nyissa meg a `spoke1.web.parameters.json` fájlt, és adjon meg egy felhasználónevet és jelszót a mezőkben szereplő idézőjeleket sor 53 és 54 közötti alább látható módon, majd mentse a fájlt.</span><span class="sxs-lookup"><span data-stu-id="f278c-238">Open the `spoke1.web.parameters.json` file and enter a username and password between the quotes in line 53 and 54, as shown below, then save the file.</span></span>

  ```bash
  "adminUsername": "XXX",
  "adminPassword": "YYY",
  ```

3. <span data-ttu-id="f278c-239">Ismételje meg az előző lépésben a fájl `spoke2.web.parameters.json`.</span><span class="sxs-lookup"><span data-stu-id="f278c-239">Repeat the previous step for file `spoke2.web.parameters.json`.</span></span>

4. <span data-ttu-id="f278c-240">Futtassa a bash vagy telepítése az első küllős, és szeretne az elosztóhoz csatakoztatni az alábbi PowerShell-parancsot.</span><span class="sxs-lookup"><span data-stu-id="f278c-240">Run the bash or PowerShell command below to deploy the first spoke and connect it to the hub.</span></span> <span data-ttu-id="f278c-241">Helyettesítse be az értékeket az előfizetését, erőforráscsoport-név, és az Azure-régió.</span><span class="sxs-lookup"><span data-stu-id="f278c-241">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./spoke.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke1-rg \
    --location westus \
    --spoke 1
  ```

  ```powershell
  ./spoke.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke1-rg `
    -Location westus `
    -Spoke 1
  ```
  > [!NOTE]
  > <span data-ttu-id="f278c-242">Ha úgy dönt, hogy használjon másik erőforráscsoport-nevet (eltérő `ra-spoke1-rg`), ügyeljen arra, hogy minden paraméter fájlok használja ezt a nevet, és szerkessze azokat használni a saját erőforráscsoport-név.</span><span class="sxs-lookup"><span data-stu-id="f278c-242">If you decide to use a different resource group name (other than `ra-spoke1-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="f278c-243">Várjon, amíg a telepítés befejezéséhez.</span><span class="sxs-lookup"><span data-stu-id="f278c-243">Wait for the deployment to finish.</span></span> <span data-ttu-id="f278c-244">A központi telepítés létrehoz egy virtuális hálózatot, a terheléselosztó három Ubuntu és Apache és egy virtuális hálózatot szeretne az elosztóhoz Vnetben társviszony-létesítési kapcsolat az előző szakaszban létrehozott futtató virtuális gépet.</span><span class="sxs-lookup"><span data-stu-id="f278c-244">This deployment creates a virtual network, a load balancer with three virtual machine running Ubuntu and Apache, and a VNet peering connection to the hub VNet created in the previous section.</span></span> <span data-ttu-id="f278c-245">A telepítés több mint 20 percig is eltarthat.</span><span class="sxs-lookup"><span data-stu-id="f278c-245">This deployment may take over 20 minutes.</span></span>

6. <span data-ttu-id="f278c-246">Futtassa a bash vagy telepítése az első küllős, és szeretne az elosztóhoz csatakoztatni az alábbi PowerShell-parancsot.</span><span class="sxs-lookup"><span data-stu-id="f278c-246">Run the bash or PowerShell command below to deploy the first spoke and connect it to the hub.</span></span> <span data-ttu-id="f278c-247">Helyettesítse be az értékeket az előfizetését, erőforráscsoport-név, és az Azure-régió.</span><span class="sxs-lookup"><span data-stu-id="f278c-247">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./spoke.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke2-rg \
    --location westus \
    --spoke 2
  ```

  ```powershell
  ./spoke.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke2-rg `
    -Location westus `
    -Spoke 2
  ```
  > [!NOTE]
  > <span data-ttu-id="f278c-248">Ha úgy dönt, hogy használjon másik erőforráscsoport-nevet (eltérő `ra-spoke2-rg`), ügyeljen arra, hogy minden paraméter fájlok használja ezt a nevet, és szerkessze azokat használni a saját erőforráscsoport-név.</span><span class="sxs-lookup"><span data-stu-id="f278c-248">If you decide to use a different resource group name (other than `ra-spoke2-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="f278c-249">Várjon, amíg a telepítés befejezéséhez.</span><span class="sxs-lookup"><span data-stu-id="f278c-249">Wait for the deployment to finish.</span></span> <span data-ttu-id="f278c-250">A központi telepítés létrehoz egy virtuális hálózatot, a terheléselosztó három Ubuntu és Apache és egy virtuális hálózatot szeretne az elosztóhoz Vnetben társviszony-létesítési kapcsolat az előző szakaszban létrehozott futtató virtuális gépet.</span><span class="sxs-lookup"><span data-stu-id="f278c-250">This deployment creates a virtual network, a load balancer with three virtual machine running Ubuntu and Apache, and a VNet peering connection to the hub VNet created in the previous section.</span></span> <span data-ttu-id="f278c-251">A telepítés több mint 20 percig is eltarthat.</span><span class="sxs-lookup"><span data-stu-id="f278c-251">This deployment may take over 20 minutes.</span></span>

### <a name="azure-hub-vnet-peering-to-spoke-vnets"></a><span data-ttu-id="f278c-252">Azure hub Vnetben társviszony-létesítést úgy küllős Vnetek</span><span class="sxs-lookup"><span data-stu-id="f278c-252">Azure hub VNet peering to spoke VNets</span></span>

<span data-ttu-id="f278c-253">A virtuális hálózat társviszony-létesítési kapcsolatok VNet központ telepítéséhez hajtsa végre az alábbi lépéseket.</span><span class="sxs-lookup"><span data-stu-id="f278c-253">To deploy the VNet peering connections for the hub VNet, perform the following steps.</span></span>

1. <span data-ttu-id="f278c-254">Váltás a `hybrid-networking\hub-spoke\hub` a fenti előfeltételek lépésben letöltött tárház mappát.</span><span class="sxs-lookup"><span data-stu-id="f278c-254">Switch to the `hybrid-networking\hub-spoke\hub` folder for the repository you download in the pre-requisites step above.</span></span>

2. <span data-ttu-id="f278c-255">Futtassa a bash vagy a PowerShell-parancsot a társviszony-létesítési kapcsolat telepítéséhez az első küllős.</span><span class="sxs-lookup"><span data-stu-id="f278c-255">Run the bash or PowerShell command below to deploy the peering connection to the first spoke.</span></span> <span data-ttu-id="f278c-256">Helyettesítse be az értékeket az előfizetését, erőforráscsoport-név, és az Azure-régió.</span><span class="sxs-lookup"><span data-stu-id="f278c-256">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./hub.peering.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-hub-rg \
    --location westus \
    --spoke 1
  ```

  ```powershell
  ./hub.peering.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-hub-rg `
    -Location westus `
    -Spoke 1
  ```

2. <span data-ttu-id="f278c-257">Futtassa a bash vagy a PowerShell-parancsot a második küllős központi telepítése a társviszony-létesítési kapcsolat.</span><span class="sxs-lookup"><span data-stu-id="f278c-257">Run the bash or PowerShell command below to deploy the peering connection to the second spoke.</span></span> <span data-ttu-id="f278c-258">Helyettesítse be az értékeket az előfizetését, erőforráscsoport-név, és az Azure-régió.</span><span class="sxs-lookup"><span data-stu-id="f278c-258">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./hub.peering.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-hub-rg \
    --location westus \
    --spoke 2
  ```

  ```powershell
  ./hub.peering.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-hub-rg `
    -Location westus `
    -Spoke 2
  ```

### <a name="test-connectivity"></a><span data-ttu-id="f278c-259">Kapcsolat tesztelése</span><span class="sxs-lookup"><span data-stu-id="f278c-259">Test connectivity</span></span>

<span data-ttu-id="f278c-260">Győződjön meg arról, hogy a hub-küllős topológia egy helyszíni adatközpontbeli dolgozott, kövesse az alábbi lépéseket.</span><span class="sxs-lookup"><span data-stu-id="f278c-260">To verify that the hub-spoke topology connected to an on-premises datacenter deployment worked, follow these steps.</span></span>

1. <span data-ttu-id="f278c-261">Azure-portálról [] [portal] csatlakozás az előfizetéshez, és keresse meg a `ra-onprem-vm1` a virtuális gép a `ra-onprem-rg` erőforráscsoportot.</span><span class="sxs-lookup"><span data-stu-id="f278c-261">From the [Azure portal][portal], connect to your subscription, and navigate to the `ra-onprem-vm1` virtual machine in the `ra-onprem-rg` resource group.</span></span>

2. <span data-ttu-id="f278c-262">Az a `Overview` panelen, vegye figyelembe a `Public IP address` a virtuális gép számára.</span><span class="sxs-lookup"><span data-stu-id="f278c-262">In the `Overview` blade, note the `Public IP address` for the VM.</span></span>

3. <span data-ttu-id="f278c-263">Csatlakozhat egy SSH-ügyfél az IP-cím, a felhasználónév és a telepítés során megadott jelszóval fent leírt eseteket.</span><span class="sxs-lookup"><span data-stu-id="f278c-263">Use an SSH client to connect to the IP address you noted above using the user name and password you specified during deployment.</span></span>

4. <span data-ttu-id="f278c-264">A virtuális Gépen a parancssorból csatlakoznia, az az alábbi parancs futtatásával ellenőrizze a Spoke1 VNet és a helyszíni virtuális hálózat közötti kapcsolatot.</span><span class="sxs-lookup"><span data-stu-id="f278c-264">From the command prompt on the VM you connected to, run the command below to test connectivity from the on-premises VNet to the Spoke1 VNet.</span></span>

  ```bash
  ping 10.1.1.37
  ```

### <a name="add-connectivity-between-spokes"></a><span data-ttu-id="f278c-265">Adja hozzá a küllők közötti kapcsolat</span><span class="sxs-lookup"><span data-stu-id="f278c-265">Add connectivity between spokes</span></span>

<span data-ttu-id="f278c-266">Ha szeretné engedélyezni a küllők csatlakozni egymáshoz, udr-EK minden küllős, amely a VNet központban átjáró más küllők irányuló forgalom továbbítására kell telepítenie.</span><span class="sxs-lookup"><span data-stu-id="f278c-266">If you want to allow spokes to connect to each other, you must deploy UDRs to each spoke that forward traffic destined to other spokes to the gateway in the hub VNet.</span></span> <span data-ttu-id="f278c-267">Hajtsa végre az alábbi lépések végrehajtásával ellenőrizze, hogy jelenleg nem képes egy küllős a másikra, majd a udr-EK központi telepítése és tesztelése újra.</span><span class="sxs-lookup"><span data-stu-id="f278c-267">Perform the following steps to verify that currently you are not able to connect from a spoke to another, then deploy the UDRs and test connectivity again.</span></span>

1. <span data-ttu-id="f278c-268">Ha nem kapcsolódik a virtuális gép jumpbox már ismételje meg a fenti 1 – 4.</span><span class="sxs-lookup"><span data-stu-id="f278c-268">Repeat steps 1 to 4 above, if you are not connected to the jumpbox VM any longer.</span></span>

2. <span data-ttu-id="f278c-269">Csatlakozhat a webkiszolgálók küllős 1 egyikére.</span><span class="sxs-lookup"><span data-stu-id="f278c-269">Connect to one of the web servers in spoke 1.</span></span>

  ```bash
  ssh 10.1.1.37
  ```

3. <span data-ttu-id="f278c-270">Küllős 1 és 2 küllős közötti kapcsolat tesztelése.</span><span class="sxs-lookup"><span data-stu-id="f278c-270">Test the connectivity between spoke 1 and spoke 2.</span></span> <span data-ttu-id="f278c-271">Akkor kell sikertelen.</span><span class="sxs-lookup"><span data-stu-id="f278c-271">It should fail.</span></span>

  ```bash
  ping 10.1.2.37
  ```

4. <span data-ttu-id="f278c-272">Lépjen vissza a számítógép parancssort.</span><span class="sxs-lookup"><span data-stu-id="f278c-272">Switch back to your computer's command prompt.</span></span>

5. <span data-ttu-id="f278c-273">Váltás a `hybrid-networking\hub-spoke\spokes` a fenti előfeltételek lépésben letöltött tárház mappát.</span><span class="sxs-lookup"><span data-stu-id="f278c-273">Switch to the `hybrid-networking\hub-spoke\spokes` folder for the repository you downloaded in the pre-requisites step above.</span></span>

6. <span data-ttu-id="f278c-274">Futtassa a bash vagy a PowerShell-parancsot az alábbi egy UDR telepítéséhez az első küllős.</span><span class="sxs-lookup"><span data-stu-id="f278c-274">Run the bash or PowerShell command below to deploy an UDR to the first spoke.</span></span> <span data-ttu-id="f278c-275">Helyettesítse be az értékeket az előfizetését, erőforráscsoport-név, és az Azure-régió.</span><span class="sxs-lookup"><span data-stu-id="f278c-275">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./spoke.udr.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke1-rg \
    --location westus \
    --spoke 1
  ```

  ```powershell
  ./spoke.udr.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke1-rg `
    -Location westus `
    -Spoke 1
  ```

7. <span data-ttu-id="f278c-276">Futtassa a bash vagy a PowerShell-parancsot az alábbi központi telepítése egy UDR a második küllős.</span><span class="sxs-lookup"><span data-stu-id="f278c-276">Run the bash or PowerShell command below to deploy an UDR to the second spoke.</span></span> <span data-ttu-id="f278c-277">Helyettesítse be az értékeket az előfizetését, erőforráscsoport-név, és az Azure-régió.</span><span class="sxs-lookup"><span data-stu-id="f278c-277">Substitute the values with your subscription, resource group name, and Azure region.</span></span>

  ```bash
  sh ./spoke.udr.deploy.sh --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
    --resourcegroup ra-spoke2-rg \
    --location westus \
    --spoke 2
  ```

  ```powershell
  ./spoke.udr.deploy.ps1 -Subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx `
    -ResourceGroup ra-spoke2-rg `
    -Location westus `
    -Spoke 2
  ```

8. <span data-ttu-id="f278c-278">Váltás a ssh terminál.</span><span class="sxs-lookup"><span data-stu-id="f278c-278">Switch back to the ssh terminal.</span></span>

9. <span data-ttu-id="f278c-279">Küllős 1 és 2 küllős közötti kapcsolat tesztelése.</span><span class="sxs-lookup"><span data-stu-id="f278c-279">Test the connectivity between spoke 1 and spoke 2.</span></span> <span data-ttu-id="f278c-280">Sikeres legyen.</span><span class="sxs-lookup"><span data-stu-id="f278c-280">It should succeed.</span></span>

  ```bash
  ping 10.1.2.37
  ```

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azure-powershell]: /powershell/azure/install-azure-ps?view=azuresmps-3.7.0
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
[0]: ./images/hub-spoke.png "Az Azure-ban hub-küllős topológia"
[1]: ./images/hub-spoke-gateway-routing.svg "Tranzitív útválasztási az Azure-ban hub-küllős topológia"
[2]: ./images/hub-spoke-no-gateway-routing.svg "Tranzitív útválasztási NVA használata az Azure-ban hub-küllős topológia"
[3]: ./images/hub-spokehub-spoke.svg "Az Azure-ban hub-küllős-hub-küllős topológia"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
