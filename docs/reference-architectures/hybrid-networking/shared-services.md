---
title: Küllős hálózati topológia implementálása
titleSuffix: Azure Reference Architectures
description: Közös szolgáltatásokkal küllős hálózati topológia implementálása az Azure-ban.
author: telmosampaio
ms.date: 10/09/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seodec18, networking
ms.openlocfilehash: 982505b4661ec907a86d59f567f2dc09e916b9a7
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298573"
---
# <a name="implement-a-hub-spoke-network-topology-with-shared-services-in-azure"></a><span data-ttu-id="3c68f-103">Közös szolgáltatásokkal küllős hálózati topológia implementálása az Azure-ban</span><span class="sxs-lookup"><span data-stu-id="3c68f-103">Implement a hub-spoke network topology with shared services in Azure</span></span>

<span data-ttu-id="3c68f-104">Ez a referenciaarchitektúra épül, amely a [küllős] [ guidance-hub-spoke] referenciaarchitektúra, amely minden küllő tudják használni az agyban megosztott szolgáltatások tartalmazza.</span><span class="sxs-lookup"><span data-stu-id="3c68f-104">This reference architecture builds on the [hub-spoke][guidance-hub-spoke] reference architecture to include shared services in the hub that can be consumed by all spokes.</span></span> <span data-ttu-id="3c68f-105">A felhőbe való migrálás egy adatközpontban, és létrehozását az első lépés egy [virtuális adatközpont], meg kell osztania az első szolgáltatások az identitások és a biztonság.</span><span class="sxs-lookup"><span data-stu-id="3c68f-105">As a first step toward migrating a datacenter to the cloud, and building a [virtual datacenter], the first services you need to share are identity and security.</span></span> <span data-ttu-id="3c68f-106">Ez a referenciaarchitektúra bemutatja, hogyan terjeszthető ki az Active Directory-szolgáltatások a helyi adatközpontból az Azure, és hogyan adhat hozzá egy hálózati virtuális készüléket (NVA) működő, a tűzfal egy küllős topológiában.</span><span class="sxs-lookup"><span data-stu-id="3c68f-106">This reference architecture shows you how to extend your Active Directory services from your on-premises datacenter to Azure, and how to add a network virtual appliance (NVA) that can act as a firewall, in a hub-spoke topology.</span></span>  <span data-ttu-id="3c68f-107">[**A megoldás üzembe helyezése.**](#deploy-the-solution)</span><span class="sxs-lookup"><span data-stu-id="3c68f-107">[**Deploy this solution**](#deploy-the-solution).</span></span>

> [!NOTE]
> <span data-ttu-id="3c68f-108">Ebben a forgatókönyvben használatával is elvégezhető [Azure tűzfal](/azure/firewall/), egy felhőalapú hálózati biztonsági szolgáltatás.</span><span class="sxs-lookup"><span data-stu-id="3c68f-108">This scenario can also be accomplished using [Azure Firewall](/azure/firewall/), a cloud-based network security service.</span></span>

![Megosztott szolgáltatások topológia az Azure-ban](./images/shared-services.png)

<span data-ttu-id="3c68f-110">*Töltse le az architektúra [Visio-fájlját][visio-download]*</span><span class="sxs-lookup"><span data-stu-id="3c68f-110">*Download a [Visio file][visio-download] of this architecture*</span></span>

<span data-ttu-id="3c68f-111">Ez a topológia előnyei a következők:</span><span class="sxs-lookup"><span data-stu-id="3c68f-111">The benefits of this topology include:</span></span>

- <span data-ttu-id="3c68f-112">**Költségmegtakarítás** a szolgáltatások központosításával, amelyeket több számítási feladat között, egyetlen helyen lehet elosztani, például a hálózati virtuális berendezések (NVA-k) és DNS-kiszolgálók.</span><span class="sxs-lookup"><span data-stu-id="3c68f-112">**Cost savings** by centralizing services that can be shared by multiple workloads, such as network virtual appliances (NVAs) and DNS servers, in a single location.</span></span>
- <span data-ttu-id="3c68f-113">**Az előfizetési korlátok leküzdése** a különböző előfizetésekből virtuális társhálózatok létesítésével a központi agyhoz.</span><span class="sxs-lookup"><span data-stu-id="3c68f-113">**Overcome subscriptions limits** by peering VNets from different subscriptions to the central hub.</span></span>
- <span data-ttu-id="3c68f-114">**Kockázatok elkülönítése** a központi IT (SecOps, InfraOps) és a számítási feladatok között (DevOps).</span><span class="sxs-lookup"><span data-stu-id="3c68f-114">**Separation of concerns** between central IT (SecOps, InfraOps) and workloads (DevOps).</span></span>

<span data-ttu-id="3c68f-115">Az architektúra gyakori használati módjai többek között a következők:</span><span class="sxs-lookup"><span data-stu-id="3c68f-115">Typical uses for this architecture include:</span></span>

- <span data-ttu-id="3c68f-116">A különböző környezetekben (például a fejlesztés, a tesztelés és a gyártás) üzembe helyezett számítási feladatok, amelyek megosztott szolgáltatásokat igényelnek (például DNS, IDS, NTP vagy AD DS).</span><span class="sxs-lookup"><span data-stu-id="3c68f-116">Workloads deployed in different environments, such as development, testing, and production, that require shared services such as DNS, IDS, NTP, or AD DS.</span></span> <span data-ttu-id="3c68f-117">A megosztott szolgáltatások az agyi virtuális hálózatban találhatóak, azonban minden környezet egy küllőn van üzembe helyezve, így megmarad az elszigetelés.</span><span class="sxs-lookup"><span data-stu-id="3c68f-117">Shared services are placed in the hub VNet, while each environment is deployed to a spoke to maintain isolation.</span></span>
- <span data-ttu-id="3c68f-118">Olyan számítási feladatok, amelyek nem igénylik az egymással való kapcsolatot, de igénylik a megosztott szolgáltatásokhoz való hozzáférést.</span><span class="sxs-lookup"><span data-stu-id="3c68f-118">Workloads that do not require connectivity to each other, but require access to shared services.</span></span>
- <span data-ttu-id="3c68f-119">Olyan vállalatok, amelyek központi felügyeletet igényelnek a biztonsági szempontokhoz, például egy tűzfal az agyban DMZ-ként, és küllőnkénti elkülönített felügyelet a számítási feladatokhoz.</span><span class="sxs-lookup"><span data-stu-id="3c68f-119">Enterprises that require central control over security aspects, such as a firewall in the hub as a DMZ, and segregated management for the workloads in each spoke.</span></span>

## <a name="architecture"></a><span data-ttu-id="3c68f-120">Architektúra</span><span class="sxs-lookup"><span data-stu-id="3c68f-120">Architecture</span></span>

<span data-ttu-id="3c68f-121">Az architektúra a következőkben leírt összetevőkből áll.</span><span class="sxs-lookup"><span data-stu-id="3c68f-121">The architecture consists of the following components.</span></span>

- <span data-ttu-id="3c68f-122">**Helyszíni hálózat**.</span><span class="sxs-lookup"><span data-stu-id="3c68f-122">**On-premises network**.</span></span> <span data-ttu-id="3c68f-123">A cégen belül futó helyi magánhálózat.</span><span class="sxs-lookup"><span data-stu-id="3c68f-123">A private local-area network running within an organization.</span></span>

- <span data-ttu-id="3c68f-124">**VPN-eszköz**.</span><span class="sxs-lookup"><span data-stu-id="3c68f-124">**VPN device**.</span></span> <span data-ttu-id="3c68f-125">A helyszíni hálózat számára külső kapcsolatot biztosító eszköz vagy szolgáltatás.</span><span class="sxs-lookup"><span data-stu-id="3c68f-125">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="3c68f-126">A VPN-eszköz lehet hardvereszköz vagy valamilyen szoftvermegoldás, amilyen például a Windows Server 2012 Útválasztás és távelérés szolgáltatása (Routing and Remote Access Service, RRAS).</span><span class="sxs-lookup"><span data-stu-id="3c68f-126">The VPN device may be a hardware device, or a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="3c68f-127">A támogatott VPN-berendezések listáját és az egyes VPN-berendezések Azure-hoz való csatlakoztatásra történő konfigurálásával kapcsolatos információkat az [Információk a helyek közötti VPN Gateway-kapcsolatok VPN-eszközeiről][vpn-appliance] című cikkben találja.</span><span class="sxs-lookup"><span data-stu-id="3c68f-127">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

- <span data-ttu-id="3c68f-128">**VPN virtuális hálózati átjáró vagy ExpressRoute-átjáró**.</span><span class="sxs-lookup"><span data-stu-id="3c68f-128">**VPN virtual network gateway or ExpressRoute gateway**.</span></span> <span data-ttu-id="3c68f-129">A virtuális hálózati átjáró a helyszíni hálózattal létesített kapcsolathoz használt VPN-eszközhöz vagy az ExpressRoute-kapcsolatcsoporthoz való csatlakozást teszi lehetővé a virtuális hálózat számára.</span><span class="sxs-lookup"><span data-stu-id="3c68f-129">The virtual network gateway enables the VNet to connect to the VPN device, or ExpressRoute circuit, used for connectivity with your on-premises network.</span></span> <span data-ttu-id="3c68f-130">További információért tekintse át a [helyszíni hálózat és a Microsoft Azure Virtual Network csatlakoztatásával][connect-to-an-Azure-vnet] foglalkozó cikket.</span><span class="sxs-lookup"><span data-stu-id="3c68f-130">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

> [!NOTE]
> <span data-ttu-id="3c68f-131">A referenciaarchitektúra üzembe helyezési szkriptjei egy VPN-átjárót használnak a kapcsolat létrehozásához és egy virtuális hálózatot az Azure-ban a helyszíni hálózat szimulációjára.</span><span class="sxs-lookup"><span data-stu-id="3c68f-131">The deployment scripts for this reference architecture use a VPN gateway for connectivity, and a VNet in Azure to simulate your on-premises network.</span></span>

- <span data-ttu-id="3c68f-132">**Agyi virtuális hálózat**.</span><span class="sxs-lookup"><span data-stu-id="3c68f-132">**Hub VNet**.</span></span> <span data-ttu-id="3c68f-133">Egy Azure virtuális hálózat agyként a küllős hálózati topológiában.</span><span class="sxs-lookup"><span data-stu-id="3c68f-133">Azure VNet used as the hub in the hub-spoke topology.</span></span> <span data-ttu-id="3c68f-134">Az agy a helyszíni hálózathoz való csatlakozási lehetőségek központi helye, és a küllős hálózati topológiákban tárolt különféle számítási feladatok által felvett szolgáltatások tárolási helye.</span><span class="sxs-lookup"><span data-stu-id="3c68f-134">The hub is the central point of connectivity to your on-premises network, and a place to host services that can be consumed by the different workloads hosted in the spoke VNets.</span></span>

- <span data-ttu-id="3c68f-135">**Átjáró-alhálózat**.</span><span class="sxs-lookup"><span data-stu-id="3c68f-135">**Gateway subnet**.</span></span> <span data-ttu-id="3c68f-136">A virtuális hálózati átjárók ugyanazon az alhálózaton találhatók.</span><span class="sxs-lookup"><span data-stu-id="3c68f-136">The virtual network gateways are held in the same subnet.</span></span>

- <span data-ttu-id="3c68f-137">**Megosztott szolgáltatások alhálózata**.</span><span class="sxs-lookup"><span data-stu-id="3c68f-137">**Shared services subnet**.</span></span> <span data-ttu-id="3c68f-138">Az agyi virtuális hálózat egyik alhálózata, amely olyan szolgáltatások tárolására szolgál, amelyek megoszthatóak minden küllő között, mint például a DNS vagy az AD DS.</span><span class="sxs-lookup"><span data-stu-id="3c68f-138">A subnet in the hub VNet used to host services that can be shared among all spokes, such as DNS or AD DS.</span></span>

- <span data-ttu-id="3c68f-139">**DMZ-alhálózat**.</span><span class="sxs-lookup"><span data-stu-id="3c68f-139">**DMZ subnet**.</span></span> <span data-ttu-id="3c68f-140">Az agyi virtuális hálózat a gazdagép nva-kkal működő, biztonsági eszközökkel, például a tűzfalak használt alhálózat.</span><span class="sxs-lookup"><span data-stu-id="3c68f-140">A subnet in the hub VNet used to host NVAs that can act as security appliances, such as firewalls.</span></span>

- <span data-ttu-id="3c68f-141">**Küllő virtuális hálózatok**.</span><span class="sxs-lookup"><span data-stu-id="3c68f-141">**Spoke VNets**.</span></span> <span data-ttu-id="3c68f-142">Egy vagy több Azure virtuális hálózat, amelyek küllőként használatosak a küllős topológiában.</span><span class="sxs-lookup"><span data-stu-id="3c68f-142">One or more Azure VNets that are used as spokes in the hub-spoke topology.</span></span> <span data-ttu-id="3c68f-143">A küllőkkel elszigetelhetőek a számítási feladatok saját virtuális hálózataikban, így más küllőktől elkülönülten kezelhetőek.</span><span class="sxs-lookup"><span data-stu-id="3c68f-143">Spokes can be used to isolate workloads in their own VNets, managed separately from other spokes.</span></span> <span data-ttu-id="3c68f-144">Minden számítási feladat több szintet tartalmazhat, amelyek alhálózatait Azure-terheléselosztók kapcsolják össze.</span><span class="sxs-lookup"><span data-stu-id="3c68f-144">Each workload might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="3c68f-145">További információ az alkalmazás-infrastruktúrával kapcsolatban: [Windows rendszerű virtuális gépek számítási feladatainak futtatása][windows-vm-ra] és [Számítási feladatok futtatása Linux rendszerű virtuális gépeken][linux-vm-ra].</span><span class="sxs-lookup"><span data-stu-id="3c68f-145">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

- <span data-ttu-id="3c68f-146">**Virtuális társhálózatok létesítése**.</span><span class="sxs-lookup"><span data-stu-id="3c68f-146">**VNet peering**.</span></span> <span data-ttu-id="3c68f-147">Két virtuális hálózat hitelesítéssel lehet csatlakozni egy [társviszonykapcsolat][vnet-peering].</span><span class="sxs-lookup"><span data-stu-id="3c68f-147">Two VNets can be connected using a [peering connection][vnet-peering].</span></span> <span data-ttu-id="3c68f-148">A társviszonykapcsolatok a virtuális hálózatok közötti nem tranzitív, alacsony késleltetésű kapcsolatok.</span><span class="sxs-lookup"><span data-stu-id="3c68f-148">Peering connections are non-transitive, low latency connections between VNets.</span></span> <span data-ttu-id="3c68f-149">A társviszonyba kapcsolt virtuális hálózatok az Azure gerinchálózatát használva, útválasztó igénye nélkül váltanak forgalmat egymás között.</span><span class="sxs-lookup"><span data-stu-id="3c68f-149">Once peered, the VNets exchange traffic by using the Azure backbone, without the need for a router.</span></span> <span data-ttu-id="3c68f-150">Egy küllős hálózati topológiában a virtuális társhálózatok létesítésével lehetséges az agyat minden egyes küllőhöz kapcsolni.</span><span class="sxs-lookup"><span data-stu-id="3c68f-150">In a hub-spoke network topology, you use VNet peering to connect the hub to each spoke.</span></span> <span data-ttu-id="3c68f-151">Az ugyanabban a régióban, vagy (globális virtuális hálózatok közötti Társviszony) különböző régiókban található virtuális hálózatok társviszonyt.</span><span class="sxs-lookup"><span data-stu-id="3c68f-151">You can peer virtual networks in the same region, or different regions (Global VNet Peering).</span></span> <span data-ttu-id="3c68f-152">További információkért lásd: [-követelmények és korlátozások][vnet-peering-requirements].</span><span class="sxs-lookup"><span data-stu-id="3c68f-152">For more information, see [Requirements and constraints][vnet-peering-requirements].</span></span>

> [!NOTE]
> <span data-ttu-id="3c68f-153">Ez a cikk csak a [Resource Manager](/azure/azure-resource-manager/resource-group-overview) üzemelő példányokat mutatja be, de ugyanabban az előfizetésben klasszikus virtuális hálózatot is csatlakoztathat Resource Manager virtuális hálózathoz.</span><span class="sxs-lookup"><span data-stu-id="3c68f-153">This article only covers [Resource Manager](/azure/azure-resource-manager/resource-group-overview) deployments, but you can also connect a classic VNet to a Resource Manager VNet in the same subscription.</span></span> <span data-ttu-id="3c68f-154">Így a küllők tárolhatnak klasszikus üzemelő példányokat, mégis profitálhatnak az agyban megosztott szolgáltatásokból.</span><span class="sxs-lookup"><span data-stu-id="3c68f-154">That way, your spokes can host classic deployments and still benefit from services shared in the hub.</span></span>

## <a name="recommendations"></a><span data-ttu-id="3c68f-155">Javaslatok</span><span class="sxs-lookup"><span data-stu-id="3c68f-155">Recommendations</span></span>

<span data-ttu-id="3c68f-156">Az összes javaslatot a [küllős] [ guidance-hub-spoke] referenciaarchitektúra is alkalmazni kell a megosztott szolgáltatások referenciaarchitektúra.</span><span class="sxs-lookup"><span data-stu-id="3c68f-156">All the recommendations for the [hub-spoke][guidance-hub-spoke] reference architecture also apply to the shared services reference architecture.</span></span>

<span data-ttu-id="3c68f-157">Emellett az alábbi javaslatok a a megosztott szolgáltatások legtöbb forgatókönyvre vonatkoznak.</span><span class="sxs-lookup"><span data-stu-id="3c68f-157">Also, the following recommendations apply for most scenarios under shared services.</span></span> <span data-ttu-id="3c68f-158">Kövesse ezeket a javaslatokat, ha nincsenek ezeket felülíró követelményei.</span><span class="sxs-lookup"><span data-stu-id="3c68f-158">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="identity"></a><span data-ttu-id="3c68f-159">Identitás</span><span class="sxs-lookup"><span data-stu-id="3c68f-159">Identity</span></span>

<span data-ttu-id="3c68f-160">A legtöbb vállalati szervezetek Active Directory Directory Services (ADDS) környezet rendelkezik a helyszíni adatközpontját.</span><span class="sxs-lookup"><span data-stu-id="3c68f-160">Most enterprise organizations have an Active Directory Directory Services (ADDS) environment in their on-premises datacenter.</span></span> <span data-ttu-id="3c68f-161">Felügyeleti eszközök a helyszíni hálózatból az Azure-bA áthelyezni, ettől az ADDS elősegítése érdekében ajánlott gazdagép ADDS-tartományvezérlőkhöz az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="3c68f-161">To facilitate management of assets moved to Azure from your on-premises network that depend on ADDS, it is recommended to host ADDS domain controllers in Azure.</span></span>

<span data-ttu-id="3c68f-162">Ha csoportházirend-objektumok, külön-külön szabályozhatja az Azure és a helyszíni környezetben, a kívánt használhatja egy másik Active Directory-hely az egyes Azure-régióban.</span><span class="sxs-lookup"><span data-stu-id="3c68f-162">If you use Group Policy Objects, that you want to control separately for Azure and your on-premises environment, use a different AD site for each Azure region.</span></span> <span data-ttu-id="3c68f-163">Helyezze a tartományvezérlők egy központi virtuális hálózatban (hub) függő számítási feladatokhoz férhet hozzá.</span><span class="sxs-lookup"><span data-stu-id="3c68f-163">Place your domain controllers in a central VNet (hub) that dependent workloads can access.</span></span>

### <a name="security"></a><span data-ttu-id="3c68f-164">Biztonság</span><span class="sxs-lookup"><span data-stu-id="3c68f-164">Security</span></span>

<span data-ttu-id="3c68f-165">Helyváltoztatáskor számítási feladatokat a helyszíni környezetből az Azure-ba, néhány ilyen számítási feladat kell virtuális gépeken üzemeltetni.</span><span class="sxs-lookup"><span data-stu-id="3c68f-165">As you move workloads from your on-premises environment to Azure, some of these workloads will require to be hosted in VMs.</span></span> <span data-ttu-id="3c68f-166">Megfelelőségi okokból szükség lehet kikényszerítheti forgalomról azokat a munkaterheléseket.</span><span class="sxs-lookup"><span data-stu-id="3c68f-166">For compliance reasons, you may need to enforce restrictions on traffic traversing those workloads.</span></span>

<span data-ttu-id="3c68f-167">Az Azure biztonságát és teljesítményét szolgáltatások különböző típusú gazdagép-hálózati virtuális berendezések (nva-k) is használhatja.</span><span class="sxs-lookup"><span data-stu-id="3c68f-167">You can use network virtual appliances (NVAs) in Azure to host different types of security and performance services.</span></span> <span data-ttu-id="3c68f-168">Ha ismeri a helyszíni berendezések adott halmazát még ma, ajánlott az azonos virtuális berendezések használata az Azure-ban, ha vannak ilyenek.</span><span class="sxs-lookup"><span data-stu-id="3c68f-168">If you are familiar with a given set of appliances on-premises today, it is recommended to use the same virtualized appliances in Azure, where applicable.</span></span>

> [!NOTE]
> <span data-ttu-id="3c68f-169">Ez a referenciaarchitektúra üzembe helyezési szkriptjei egy Ubuntu virtuális gép IP-továbbítás engedélyezve van egy hálózati virtuális berendezésen utánzására használja.</span><span class="sxs-lookup"><span data-stu-id="3c68f-169">The deployment scripts for this reference architecture use an Ubuntu VM with IP forwarding enabled to mimic a network virtual appliance.</span></span>

## <a name="considerations"></a><span data-ttu-id="3c68f-170">Megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="3c68f-170">Considerations</span></span>

### <a name="overcoming-vnet-peering-limits"></a><span data-ttu-id="3c68f-171">A virtuális társhálózatok létesítési korlátjának kiküszöbölése</span><span class="sxs-lookup"><span data-stu-id="3c68f-171">Overcoming VNet peering limits</span></span>

<span data-ttu-id="3c68f-172">Mindenképpen vegye figyelembe a [virtuális hálózatonként létesíthető társviszonyok számának korlátozását][vnet-peering-limit] az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="3c68f-172">Make sure you consider the [limitation on number of VNets peerings per VNet][vnet-peering-limit] in Azure.</span></span> <span data-ttu-id="3c68f-173">Ha több küllőre van szükség, mint amit a korlátozás megenged, fontolja meg egy agy-küllő-agy-küllő topológia létrehozását, ahol a küllők első szintje agyként is működik.</span><span class="sxs-lookup"><span data-stu-id="3c68f-173">If you decide you need more spokes than the limit will allow, consider creating a hub-spoke-hub-spoke topology, where the first level of spokes also act as hubs.</span></span> <span data-ttu-id="3c68f-174">Az alábbi diagram ezt a megközelítést mutatja be.</span><span class="sxs-lookup"><span data-stu-id="3c68f-174">The following diagram shows this approach.</span></span>

![Eseményközpont-küllős-küllős topológia az Azure-ban](./images/hub-spokehub-spoke.svg)

<span data-ttu-id="3c68f-176">Ezenkívül vegye figyelembe, mely szolgáltatások vannak megosztva az agyban. Ezzel biztosítható, hogy az agy több küllőre is skálázzon.</span><span class="sxs-lookup"><span data-stu-id="3c68f-176">Also consider what services are shared in the hub, to ensure the hub scales for a larger number of spokes.</span></span> <span data-ttu-id="3c68f-177">Például ha az agy tűzfalszolgáltatásokat is nyújt, vegye figyelembe a tűzfal megoldás sávszélességi korlátozásait több küllő hozzáadása esetén.</span><span class="sxs-lookup"><span data-stu-id="3c68f-177">For instance, if your hub provides firewall services, consider the bandwidth limits of your firewall solution when adding multiple spokes.</span></span> <span data-ttu-id="3c68f-178">Lehet, hogy érdemes néhány megosztott szolgáltatást az agyak második szintjére mozgatni.</span><span class="sxs-lookup"><span data-stu-id="3c68f-178">You might want to move some of these shared services to a second level of hubs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="3c68f-179">A megoldás üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="3c68f-179">Deploy the solution</span></span>

<span data-ttu-id="3c68f-180">Ennek az architektúrának egy üzemelő példánya elérhető a [GitHubon][ref-arch-repo].</span><span class="sxs-lookup"><span data-stu-id="3c68f-180">A deployment for this architecture is available on [GitHub][ref-arch-repo].</span></span> <span data-ttu-id="3c68f-181">Az üzembe helyezés a következő erőforráscsoportokat hozza létre az előfizetésben:</span><span class="sxs-lookup"><span data-stu-id="3c68f-181">The deployment creates the following resource groups in your subscription:</span></span>

- <span data-ttu-id="3c68f-182">eseményközpont-ad-rg</span><span class="sxs-lookup"><span data-stu-id="3c68f-182">hub-adds-rg</span></span>
- <span data-ttu-id="3c68f-183">eseményközpont-nva-rg</span><span class="sxs-lookup"><span data-stu-id="3c68f-183">hub-nva-rg</span></span>
- <span data-ttu-id="3c68f-184">eseményközpont-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="3c68f-184">hub-vnet-rg</span></span>
- <span data-ttu-id="3c68f-185">rendszert-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="3c68f-185">onprem-vnet-rg</span></span>
- <span data-ttu-id="3c68f-186">spoke1-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="3c68f-186">spoke1-vnet-rg</span></span>
- <span data-ttu-id="3c68f-187">spoke2-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="3c68f-187">spoke2-vnet-rg</span></span>

<span data-ttu-id="3c68f-188">A sablon paraméterfájljai ezekre a nevekre hivatkoznak, ezért ha módosítja őket, a paraméterfájlokat is frissítse.</span><span class="sxs-lookup"><span data-stu-id="3c68f-188">The template parameter files refer to these names, so if you change them, update the parameter files to match.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="3c68f-189">Előfeltételek</span><span class="sxs-lookup"><span data-stu-id="3c68f-189">Prerequisites</span></span>

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-the-simulated-on-premises-datacenter-using-azbb"></a><span data-ttu-id="3c68f-190">Üzembe helyezése az azbb használatával szimulált helyszíni adatközpont</span><span class="sxs-lookup"><span data-stu-id="3c68f-190">Deploy the simulated on-premises datacenter using azbb</span></span>

<span data-ttu-id="3c68f-191">Ez a lépés telepíti a szimulált helyszíni adatközpont Azure virtuális hálózatként.</span><span class="sxs-lookup"><span data-stu-id="3c68f-191">This step deploys the simulated on-premises datacenter as an Azure VNet.</span></span>

1. <span data-ttu-id="3c68f-192">Keresse meg a `hybrid-networking\shared-services-stack\` mappájában, a GitHub-adattárban.</span><span class="sxs-lookup"><span data-stu-id="3c68f-192">Navigate to the `hybrid-networking\shared-services-stack\` folder of the GitHub repository.</span></span>

2. <span data-ttu-id="3c68f-193">Nyissa meg az `onprem.json` fájlt.</span><span class="sxs-lookup"><span data-stu-id="3c68f-193">Open the `onprem.json` file.</span></span>

3. <span data-ttu-id="3c68f-194">Keresse meg az összes példányát `UserName`, `adminUserName`,`Password`, és `adminPassword`.</span><span class="sxs-lookup"><span data-stu-id="3c68f-194">Search for all instances of `UserName`, `adminUserName`,`Password`, and `adminPassword`.</span></span> <span data-ttu-id="3c68f-195">Adja meg az értékeket a felhasználónevet és jelszót a paraméterek, és mentse a fájlt.</span><span class="sxs-lookup"><span data-stu-id="3c68f-195">Enter values for the user name and password in the parameters and save the file.</span></span>

4. <span data-ttu-id="3c68f-196">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="3c68f-196">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g onprem-vnet-rg -l <location> -p onprem.json --deploy
   ```

5. <span data-ttu-id="3c68f-197">Várjon, amíg az üzembe helyezés befejeződik.</span><span class="sxs-lookup"><span data-stu-id="3c68f-197">Wait for the deployment to finish.</span></span> <span data-ttu-id="3c68f-198">A központi telepítéshez létrehoz egy virtuális hálózatot, a Windows és a egy VPN-átjárót futtató virtuális gépet.</span><span class="sxs-lookup"><span data-stu-id="3c68f-198">This deployment creates a virtual network, a virtual machine running Windows, and a VPN gateway.</span></span> <span data-ttu-id="3c68f-199">Egy VPN-átjáró létrehozása 40 percnél is tovább tarthat.</span><span class="sxs-lookup"><span data-stu-id="3c68f-199">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="deploy-the-hub-vnet"></a><span data-ttu-id="3c68f-200">Az agyi virtuális hálózat üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="3c68f-200">Deploy the hub VNet</span></span>

<span data-ttu-id="3c68f-201">Ebben a lépésben helyez üzembe az agyi virtuális hálózat, és csatlakoztatja a szimulált helyszíni virtuális hálózathoz.</span><span class="sxs-lookup"><span data-stu-id="3c68f-201">This step deploys the hub VNet and connects it to the simulated on-premises VNet.</span></span>

1. <span data-ttu-id="3c68f-202">Nyissa meg az `hub-vnet.json` fájlt.</span><span class="sxs-lookup"><span data-stu-id="3c68f-202">Open the `hub-vnet.json` file.</span></span>

2. <span data-ttu-id="3c68f-203">Keresse meg `adminPassword` , és adja meg egy felhasználónevet és jelszót a paraméterek.</span><span class="sxs-lookup"><span data-stu-id="3c68f-203">Search for `adminPassword` and enter a user name and password in the parameters.</span></span>

3. <span data-ttu-id="3c68f-204">Keresse meg az összes példányát `sharedKey` és a egy megosztott kulcsot adjon meg egy értéket.</span><span class="sxs-lookup"><span data-stu-id="3c68f-204">Search for all instances of `sharedKey` and enter a value for a shared key.</span></span> <span data-ttu-id="3c68f-205">Mentse a fájlt.</span><span class="sxs-lookup"><span data-stu-id="3c68f-205">Save the file.</span></span>

   ```json
   "sharedKey": "abc123",
   ```

4. <span data-ttu-id="3c68f-206">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="3c68f-206">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet.json --deploy
   ```

5. <span data-ttu-id="3c68f-207">Várjon, amíg az üzembe helyezés befejeződik.</span><span class="sxs-lookup"><span data-stu-id="3c68f-207">Wait for the deployment to finish.</span></span> <span data-ttu-id="3c68f-208">A központi telepítés egy virtuális hálózatot, egy virtuális gépet, a VPN-átjáró és az előző szakaszban létrehozott átjáróhoz való kapcsolatot hoz létre.</span><span class="sxs-lookup"><span data-stu-id="3c68f-208">This deployment creates a virtual network, a virtual machine, a VPN gateway, and a connection to the gateway created in the previous section.</span></span> <span data-ttu-id="3c68f-209">A VPN-átjáró több mint 40 percet is igénybe vehet.</span><span class="sxs-lookup"><span data-stu-id="3c68f-209">The VPN gateway can take more than 40 minutes to complete.</span></span>

### <a name="deploy-ad-ds-in-azure"></a><span data-ttu-id="3c68f-210">Az Azure AD DS üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="3c68f-210">Deploy AD DS in Azure</span></span>

<span data-ttu-id="3c68f-211">Ebben a lépésben üzembe helyezi az AD DS-tartományvezérlők az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="3c68f-211">This step deploys AD DS domain controllers in Azure.</span></span>

1. <span data-ttu-id="3c68f-212">Nyissa meg az `hub-adds.json` fájlt.</span><span class="sxs-lookup"><span data-stu-id="3c68f-212">Open the `hub-adds.json` file.</span></span>

2. <span data-ttu-id="3c68f-213">Keresse meg az összes példányát `Password` és `adminPassword`.</span><span class="sxs-lookup"><span data-stu-id="3c68f-213">Search for all instances of `Password` and `adminPassword`.</span></span> <span data-ttu-id="3c68f-214">Adja meg az értékeket a felhasználónevet és jelszót a paraméterek, és mentse a fájlt.</span><span class="sxs-lookup"><span data-stu-id="3c68f-214">Enter values for the user name and password in the parameters and save the file.</span></span>

3. <span data-ttu-id="3c68f-215">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="3c68f-215">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-adds-rg -l <location> -p hub-adds.json --deploy
   ```

<span data-ttu-id="3c68f-216">Ebben a lépésben üzembe helyezés eltarthat néhány percig, mert a két virtuális gép csatlakoztatja a szimulált helyszíni adatközpont tartományhoz, és telepíti őket az AD DS.</span><span class="sxs-lookup"><span data-stu-id="3c68f-216">This deployment step may take several minutes, because it joins the two VMs to the domain hosted in the simulated on-premises datacenter, and installs AD DS on them.</span></span>

### <a name="deploy-the-spoke-vnets"></a><span data-ttu-id="3c68f-217">A küllő virtuális hálózatok üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="3c68f-217">Deploy the spoke VNets</span></span>

<span data-ttu-id="3c68f-218">Ebben a lépésben üzembe helyezi a küllő virtuális hálózatokhoz.</span><span class="sxs-lookup"><span data-stu-id="3c68f-218">This step deploys the spoke VNets.</span></span>

1. <span data-ttu-id="3c68f-219">Nyissa meg az `spoke1.json` fájlt.</span><span class="sxs-lookup"><span data-stu-id="3c68f-219">Open the `spoke1.json` file.</span></span>

2. <span data-ttu-id="3c68f-220">Keresse meg `adminPassword` , és adja meg egy felhasználónevet és jelszót a paraméterek.</span><span class="sxs-lookup"><span data-stu-id="3c68f-220">Search for `adminPassword` and enter a user name and password in the parameters.</span></span>

3. <span data-ttu-id="3c68f-221">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="3c68f-221">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke1-vnet-rg -l <location> -p spoke1.json --deploy
   ```

4. <span data-ttu-id="3c68f-222">Ismételje meg a 1. és 2 fájl `spoke2.json`.</span><span class="sxs-lookup"><span data-stu-id="3c68f-222">Repeat steps 1 and 2 for the file `spoke2.json`.</span></span>

5. <span data-ttu-id="3c68f-223">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="3c68f-223">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke2-vnet-rg -l <location> -p spoke2.json --deploy
   ```

### <a name="peer-the-hub-vnet-to-the-spoke-vnets"></a><span data-ttu-id="3c68f-224">A küllő virtuális hálózatokhoz az agyi virtuális hálózat társviszonyba állítása</span><span class="sxs-lookup"><span data-stu-id="3c68f-224">Peer the hub VNet to the spoke VNets</span></span>

<span data-ttu-id="3c68f-225">Az agyi virtuális hálózat közötti társviszony-létesítési kapcsolat létrehozásához a küllő virtuális hálózatokhoz, futtassa a következő parancsot:</span><span class="sxs-lookup"><span data-stu-id="3c68f-225">To create a peering connection from the hub VNet to the spoke VNets, run the following command:</span></span>

```bash
azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet-peering.json --deploy
```

### <a name="deploy-the-nva"></a><span data-ttu-id="3c68f-226">Az nva-t üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="3c68f-226">Deploy the NVA</span></span>

<span data-ttu-id="3c68f-227">Ebben a lépésben üzembe helyezi az NVA a `dmz` alhálózat.</span><span class="sxs-lookup"><span data-stu-id="3c68f-227">This step deploys an NVA in the `dmz` subnet.</span></span>

1. <span data-ttu-id="3c68f-228">Nyissa meg az `hub-nva.json` fájlt.</span><span class="sxs-lookup"><span data-stu-id="3c68f-228">Open the `hub-nva.json` file.</span></span>

2. <span data-ttu-id="3c68f-229">Keresse meg `adminPassword` , és adja meg egy felhasználónevet és jelszót a paraméterek.</span><span class="sxs-lookup"><span data-stu-id="3c68f-229">Search for `adminPassword` and enter a user name and password in the parameters.</span></span>

3. <span data-ttu-id="3c68f-230">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="3c68f-230">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-nva-rg -l <location> -p hub-nva.json --deploy
   ```

### <a name="test-connectivity"></a><span data-ttu-id="3c68f-231">Kapcsolat tesztelése</span><span class="sxs-lookup"><span data-stu-id="3c68f-231">Test connectivity</span></span>

<span data-ttu-id="3c68f-232">Az agyi virtuális hálózat, a szimulált helyszíni környezetből conectivity teszteléséhez.</span><span class="sxs-lookup"><span data-stu-id="3c68f-232">Test conectivity from the simulated on-premises environment to the hub VNet.</span></span>

1. <span data-ttu-id="3c68f-233">Az Azure Portal használatával keresse meg a `jb-vm1` nevű virtuális gépet az `onprem-jb-rg` erőforráscsoportban.</span><span class="sxs-lookup"><span data-stu-id="3c68f-233">Use the Azure portal to find the VM named `jb-vm1` in the `onprem-jb-rg` resource group.</span></span>

2. <span data-ttu-id="3c68f-234">Kattintson a `Connect` parancsra egy, a virtuális gépre irányuló távoli asztali munkamenet megnyitásához.</span><span class="sxs-lookup"><span data-stu-id="3c68f-234">Click `Connect` to open a remote desktop session to the VM.</span></span> <span data-ttu-id="3c68f-235">Használja az `onprem.json` paraméterfájlban megadott jelszót.</span><span class="sxs-lookup"><span data-stu-id="3c68f-235">Use the password that you specified in the `onprem.json` parameter file.</span></span>

3. <span data-ttu-id="3c68f-236">Nyisson meg egy PowerShell-konzolt a virtuális gépen, és a `Test-NetConnection` parancsmaggal győződjön meg arról, hogy képes-e csatlakozni a jumpbox virtuális Géphez az agyi virtuális hálózat.</span><span class="sxs-lookup"><span data-stu-id="3c68f-236">Open a PowerShell console in the VM, and use the `Test-NetConnection` cmdlet to verify that you can connect to the jumpbox VM in the hub VNet.</span></span>

   ```powershell
   Test-NetConnection 10.0.0.68 -CommonTCPPort RDP
   ```

<span data-ttu-id="3c68f-237">A kimenetnek a következőképpen kell kinéznie:</span><span class="sxs-lookup"><span data-stu-id="3c68f-237">The output should look similar to the following:</span></span>

```powershell
ComputerName     : 10.0.0.68
RemoteAddress    : 10.0.0.68
RemotePort       : 3389
InterfaceAlias   : Ethernet 2
SourceAddress    : 192.168.1.000
TcpTestSucceeded : True
```

> [!NOTE]
> <span data-ttu-id="3c68f-238">Alapértelmezés szerint Windows Serveres virtuális gépek nem teszik lehetővé az ICMP-válaszok az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="3c68f-238">By default, Windows Server VMs do not allow ICMP responses in Azure.</span></span> <span data-ttu-id="3c68f-239">Ha a használni kívánt `ping` kapcsolat teszteléséhez, engedélyeznie kell a fokozott biztonságú Windows tűzfal az ICMP-forgalmat az egyes virtuális Gépekhez.</span><span class="sxs-lookup"><span data-stu-id="3c68f-239">If you want to use `ping` to test connectivity, you need to enable ICMP traffic in the Windows Advanced Firewall for each VM.</span></span>

<span data-ttu-id="3c68f-240">Ismételje meg a sames a küllő virtuális hálózatokhoz csatlakozni:</span><span class="sxs-lookup"><span data-stu-id="3c68f-240">Repeat the sames steps to test connectivity to the spoke VNets:</span></span>

```powershell
Test-NetConnection 10.1.0.68 -CommonTCPPort RDP
Test-NetConnection 10.2.0.68 -CommonTCPPort RDP
```

<!-- links -->

[azure-cli-2]: /azure/install-azure-cli
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[guidance-hub-spoke]: ./hub-spoke.md
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[best-practices-security]: /azure/best-practices-network-securit
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[guidance-expressroute]: ./expressroute.md
[guidance-vpn]: ./vpn.md
[linux-vm-ra]: ../virtual-machines-linux/index.md
[hybrid-ha]: ./expressroute-vpn-failover.md
[naming conventions]: /azure/guidance/guidance-naming-conventions
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[virtuális adatközpont]: https://aka.ms/vdc
[virtual datacenter]: https://aka.ms/vdc
[vnet-peering]: /azure/virtual-network/virtual-network-peering-overview
[vnet-peering-limit]: /azure/azure-subscription-service-limits#networking-limits
[vnet-peering-requirements]: /azure/virtual-network/virtual-network-peering-overview#requirements-and-constraints
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md

[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
