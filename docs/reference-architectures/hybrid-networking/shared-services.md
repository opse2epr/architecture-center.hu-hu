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
ms.openlocfilehash: dd7632c3a84f6a0cee5d8b35e6a943ab8c52caf8
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54488307"
---
# <a name="implement-a-hub-spoke-network-topology-with-shared-services-in-azure"></a><span data-ttu-id="59126-103">Közös szolgáltatásokkal küllős hálózati topológia implementálása az Azure-ban</span><span class="sxs-lookup"><span data-stu-id="59126-103">Implement a hub-spoke network topology with shared services in Azure</span></span>

<span data-ttu-id="59126-104">Ez a referenciaarchitektúra épül, amely a [küllős] [ guidance-hub-spoke] referenciaarchitektúra, amely minden küllő tudják használni az agyban megosztott szolgáltatások tartalmazza.</span><span class="sxs-lookup"><span data-stu-id="59126-104">This reference architecture builds on the [hub-spoke][guidance-hub-spoke] reference architecture to include shared services in the hub that can be consumed by all spokes.</span></span> <span data-ttu-id="59126-105">A felhőbe való migrálás egy adatközpontban, és létrehozását az első lépés egy [virtuális adatközpont], meg kell osztania az első szolgáltatások az identitások és a biztonság.</span><span class="sxs-lookup"><span data-stu-id="59126-105">As a first step toward migrating a datacenter to the cloud, and building a [virtual datacenter], the first services you need to share are identity and security.</span></span> <span data-ttu-id="59126-106">Ez a referenciaarchitektúra bemutatja, hogyan terjeszthető ki az Active Directory-szolgáltatások a helyi adatközpontból az Azure, és hogyan adhat hozzá egy hálózati virtuális készüléket (NVA) működő, a tűzfal egy küllős topológiában.</span><span class="sxs-lookup"><span data-stu-id="59126-106">This reference architecture shows you how to extend your Active Directory services from your on-premises datacenter to Azure, and how to add a network virtual appliance (NVA) that can act as a firewall, in a hub-spoke topology.</span></span>  <span data-ttu-id="59126-107">[**A megoldás üzembe helyezése.**](#deploy-the-solution)</span><span class="sxs-lookup"><span data-stu-id="59126-107">[**Deploy this solution**](#deploy-the-solution).</span></span>

> [!NOTE]
> <span data-ttu-id="59126-108">Ebben a forgatókönyvben használatával is elvégezhető [Azure tűzfal](/azure/firewall/), egy felhőalapú hálózati biztonsági szolgáltatás.</span><span class="sxs-lookup"><span data-stu-id="59126-108">This scenario can also be accomplished using [Azure Firewall](/azure/firewall/), a cloud-based network security service.</span></span>

![Megosztott szolgáltatások topológia az Azure-ban](./images/shared-services.png)

<span data-ttu-id="59126-110">*Töltse le az architektúra [Visio-fájlját][visio-download]*</span><span class="sxs-lookup"><span data-stu-id="59126-110">*Download a [Visio file][visio-download] of this architecture*</span></span>

<span data-ttu-id="59126-111">Ez a topológia előnyei a következők:</span><span class="sxs-lookup"><span data-stu-id="59126-111">The benefits of this topology include:</span></span>

- <span data-ttu-id="59126-112">**Költségmegtakarítás** a szolgáltatások központosításával, amelyeket több számítási feladat között, egyetlen helyen lehet elosztani, például a hálózati virtuális berendezések (NVA-k) és DNS-kiszolgálók.</span><span class="sxs-lookup"><span data-stu-id="59126-112">**Cost savings** by centralizing services that can be shared by multiple workloads, such as network virtual appliances (NVAs) and DNS servers, in a single location.</span></span>
- <span data-ttu-id="59126-113">**Az előfizetési korlátok leküzdése** a különböző előfizetésekből virtuális társhálózatok létesítésével a központi agyhoz.</span><span class="sxs-lookup"><span data-stu-id="59126-113">**Overcome subscriptions limits** by peering VNets from different subscriptions to the central hub.</span></span>
- <span data-ttu-id="59126-114">**Kockázatok elkülönítése** a központi IT (SecOps, InfraOps) és a számítási feladatok között (DevOps).</span><span class="sxs-lookup"><span data-stu-id="59126-114">**Separation of concerns** between central IT (SecOps, InfraOps) and workloads (DevOps).</span></span>

<span data-ttu-id="59126-115">Az architektúra gyakori használati módjai többek között a következők:</span><span class="sxs-lookup"><span data-stu-id="59126-115">Typical uses for this architecture include:</span></span>

- <span data-ttu-id="59126-116">A különböző környezetekben (például a fejlesztés, a tesztelés és a gyártás) üzembe helyezett számítási feladatok, amelyek megosztott szolgáltatásokat igényelnek (például DNS, IDS, NTP vagy AD DS).</span><span class="sxs-lookup"><span data-stu-id="59126-116">Workloads deployed in different environments, such as development, testing, and production, that require shared services such as DNS, IDS, NTP, or AD DS.</span></span> <span data-ttu-id="59126-117">A megosztott szolgáltatások az agyi virtuális hálózatban találhatóak, azonban minden környezet egy küllőn van üzembe helyezve, így megmarad az elszigetelés.</span><span class="sxs-lookup"><span data-stu-id="59126-117">Shared services are placed in the hub VNet, while each environment is deployed to a spoke to maintain isolation.</span></span>
- <span data-ttu-id="59126-118">Olyan számítási feladatok, amelyek nem igénylik az egymással való kapcsolatot, de igénylik a megosztott szolgáltatásokhoz való hozzáférést.</span><span class="sxs-lookup"><span data-stu-id="59126-118">Workloads that do not require connectivity to each other, but require access to shared services.</span></span>
- <span data-ttu-id="59126-119">Olyan vállalatok, amelyek központi felügyeletet igényelnek a biztonsági szempontokhoz, például egy tűzfal az agyban DMZ-ként, és küllőnkénti elkülönített felügyelet a számítási feladatokhoz.</span><span class="sxs-lookup"><span data-stu-id="59126-119">Enterprises that require central control over security aspects, such as a firewall in the hub as a DMZ, and segregated management for the workloads in each spoke.</span></span>

## <a name="architecture"></a><span data-ttu-id="59126-120">Architektúra</span><span class="sxs-lookup"><span data-stu-id="59126-120">Architecture</span></span>

<span data-ttu-id="59126-121">Az architektúra a következőkben leírt összetevőkből áll.</span><span class="sxs-lookup"><span data-stu-id="59126-121">The architecture consists of the following components.</span></span>

- <span data-ttu-id="59126-122">**Helyszíni hálózat**.</span><span class="sxs-lookup"><span data-stu-id="59126-122">**On-premises network**.</span></span> <span data-ttu-id="59126-123">A cégen belül futó helyi magánhálózat.</span><span class="sxs-lookup"><span data-stu-id="59126-123">A private local-area network running within an organization.</span></span>

- <span data-ttu-id="59126-124">**VPN-eszköz**.</span><span class="sxs-lookup"><span data-stu-id="59126-124">**VPN device**.</span></span> <span data-ttu-id="59126-125">A helyszíni hálózat számára külső kapcsolatot biztosító eszköz vagy szolgáltatás.</span><span class="sxs-lookup"><span data-stu-id="59126-125">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="59126-126">A VPN-eszköz lehet hardvereszköz vagy valamilyen szoftvermegoldás, amilyen például a Windows Server 2012 Útválasztás és távelérés szolgáltatása (Routing and Remote Access Service, RRAS).</span><span class="sxs-lookup"><span data-stu-id="59126-126">The VPN device may be a hardware device, or a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="59126-127">A támogatott VPN-berendezések listáját és az egyes VPN-berendezések Azure-hoz való csatlakoztatásra történő konfigurálásával kapcsolatos információkat az [Információk a helyek közötti VPN Gateway-kapcsolatok VPN-eszközeiről][vpn-appliance] című cikkben találja.</span><span class="sxs-lookup"><span data-stu-id="59126-127">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

- <span data-ttu-id="59126-128">**VPN virtuális hálózati átjáró vagy ExpressRoute-átjáró**.</span><span class="sxs-lookup"><span data-stu-id="59126-128">**VPN virtual network gateway or ExpressRoute gateway**.</span></span> <span data-ttu-id="59126-129">A virtuális hálózati átjáró a helyszíni hálózattal létesített kapcsolathoz használt VPN-eszközhöz vagy az ExpressRoute-kapcsolatcsoporthoz való csatlakozást teszi lehetővé a virtuális hálózat számára.</span><span class="sxs-lookup"><span data-stu-id="59126-129">The virtual network gateway enables the VNet to connect to the VPN device, or ExpressRoute circuit, used for connectivity with your on-premises network.</span></span> <span data-ttu-id="59126-130">További információért tekintse át a [helyszíni hálózat és a Microsoft Azure Virtual Network csatlakoztatásával][connect-to-an-Azure-vnet] foglalkozó cikket.</span><span class="sxs-lookup"><span data-stu-id="59126-130">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

> [!NOTE]
> <span data-ttu-id="59126-131">A referenciaarchitektúra üzembe helyezési szkriptjei egy VPN-átjárót használnak a kapcsolat létrehozásához és egy virtuális hálózatot az Azure-ban a helyszíni hálózat szimulációjára.</span><span class="sxs-lookup"><span data-stu-id="59126-131">The deployment scripts for this reference architecture use a VPN gateway for connectivity, and a VNet in Azure to simulate your on-premises network.</span></span>

- <span data-ttu-id="59126-132">**Agyi virtuális hálózat**.</span><span class="sxs-lookup"><span data-stu-id="59126-132">**Hub VNet**.</span></span> <span data-ttu-id="59126-133">Egy Azure virtuális hálózat agyként a küllős hálózati topológiában.</span><span class="sxs-lookup"><span data-stu-id="59126-133">Azure VNet used as the hub in the hub-spoke topology.</span></span> <span data-ttu-id="59126-134">Az agy a helyszíni hálózathoz való csatlakozási lehetőségek központi helye, és a küllős hálózati topológiákban tárolt különféle számítási feladatok által felvett szolgáltatások tárolási helye.</span><span class="sxs-lookup"><span data-stu-id="59126-134">The hub is the central point of connectivity to your on-premises network, and a place to host services that can be consumed by the different workloads hosted in the spoke VNets.</span></span>

- <span data-ttu-id="59126-135">**Átjáró-alhálózat**.</span><span class="sxs-lookup"><span data-stu-id="59126-135">**Gateway subnet**.</span></span> <span data-ttu-id="59126-136">A virtuális hálózati átjárók ugyanazon az alhálózaton találhatók.</span><span class="sxs-lookup"><span data-stu-id="59126-136">The virtual network gateways are held in the same subnet.</span></span>

- <span data-ttu-id="59126-137">**Megosztott szolgáltatások alhálózata**.</span><span class="sxs-lookup"><span data-stu-id="59126-137">**Shared services subnet**.</span></span> <span data-ttu-id="59126-138">Az agyi virtuális hálózat egyik alhálózata, amely olyan szolgáltatások tárolására szolgál, amelyek megoszthatóak minden küllő között, mint például a DNS vagy az AD DS.</span><span class="sxs-lookup"><span data-stu-id="59126-138">A subnet in the hub VNet used to host services that can be shared among all spokes, such as DNS or AD DS.</span></span>

- <span data-ttu-id="59126-139">**DMZ-alhálózat**.</span><span class="sxs-lookup"><span data-stu-id="59126-139">**DMZ subnet**.</span></span> <span data-ttu-id="59126-140">Az agyi virtuális hálózat a gazdagép nva-kkal működő, biztonsági eszközökkel, például a tűzfalak használt alhálózat.</span><span class="sxs-lookup"><span data-stu-id="59126-140">A subnet in the hub VNet used to host NVAs that can act as security appliances, such as firewalls.</span></span>

- <span data-ttu-id="59126-141">**Küllő virtuális hálózatok**.</span><span class="sxs-lookup"><span data-stu-id="59126-141">**Spoke VNets**.</span></span> <span data-ttu-id="59126-142">Egy vagy több Azure virtuális hálózat, amelyek küllőként használatosak a küllős topológiában.</span><span class="sxs-lookup"><span data-stu-id="59126-142">One or more Azure VNets that are used as spokes in the hub-spoke topology.</span></span> <span data-ttu-id="59126-143">A küllőkkel elszigetelhetőek a számítási feladatok saját virtuális hálózataikban, így más küllőktől elkülönülten kezelhetőek.</span><span class="sxs-lookup"><span data-stu-id="59126-143">Spokes can be used to isolate workloads in their own VNets, managed separately from other spokes.</span></span> <span data-ttu-id="59126-144">Minden számítási feladat több szintet tartalmazhat, amelyek alhálózatait Azure-terheléselosztók kapcsolják össze.</span><span class="sxs-lookup"><span data-stu-id="59126-144">Each workload might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="59126-145">További információ az alkalmazás-infrastruktúrával kapcsolatban: [Windows rendszerű virtuális gépek számítási feladatainak futtatása][windows-vm-ra] és [Számítási feladatok futtatása Linux rendszerű virtuális gépeken][linux-vm-ra].</span><span class="sxs-lookup"><span data-stu-id="59126-145">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

- <span data-ttu-id="59126-146">**Virtuális társhálózatok létesítése**.</span><span class="sxs-lookup"><span data-stu-id="59126-146">**VNet peering**.</span></span> <span data-ttu-id="59126-147">Két, ugyanabban az Azure-régióban található virtuális hálózatot egy [társviszonykapcsolat][vnet-peering] használatával lehet összekapcsolni.</span><span class="sxs-lookup"><span data-stu-id="59126-147">Two VNets in the same Azure region can be connected using a [peering connection][vnet-peering].</span></span> <span data-ttu-id="59126-148">A társviszonykapcsolatok a virtuális hálózatok közötti nem tranzitív, alacsony késleltetésű kapcsolatok.</span><span class="sxs-lookup"><span data-stu-id="59126-148">Peering connections are non-transitive, low latency connections between VNets.</span></span> <span data-ttu-id="59126-149">A társviszonyba kapcsolt virtuális hálózatok az Azure gerinchálózatát használva, útválasztó igénye nélkül váltanak forgalmat egymás között.</span><span class="sxs-lookup"><span data-stu-id="59126-149">Once peered, the VNets exchange traffic by using the Azure backbone, without the need for a router.</span></span> <span data-ttu-id="59126-150">Egy küllős hálózati topológiában a virtuális társhálózatok létesítésével lehetséges az agyat minden egyes küllőhöz kapcsolni.</span><span class="sxs-lookup"><span data-stu-id="59126-150">In a hub-spoke network topology, you use VNet peering to connect the hub to each spoke.</span></span>

> [!NOTE]
> <span data-ttu-id="59126-151">Ez a cikk csak a [Resource Manager](/azure/azure-resource-manager/resource-group-overview) üzemelő példányokat mutatja be, de ugyanabban az előfizetésben klasszikus virtuális hálózatot is csatlakoztathat Resource Manager virtuális hálózathoz.</span><span class="sxs-lookup"><span data-stu-id="59126-151">This article only covers [Resource Manager](/azure/azure-resource-manager/resource-group-overview) deployments, but you can also connect a classic VNet to a Resource Manager VNet in the same subscription.</span></span> <span data-ttu-id="59126-152">Így a küllők tárolhatnak klasszikus üzemelő példányokat, mégis profitálhatnak az agyban megosztott szolgáltatásokból.</span><span class="sxs-lookup"><span data-stu-id="59126-152">That way, your spokes can host classic deployments and still benefit from services shared in the hub.</span></span>

## <a name="recommendations"></a><span data-ttu-id="59126-153">Javaslatok</span><span class="sxs-lookup"><span data-stu-id="59126-153">Recommendations</span></span>

<span data-ttu-id="59126-154">Az összes javaslatot a [küllős] [ guidance-hub-spoke] referenciaarchitektúra is alkalmazni kell a megosztott szolgáltatások referenciaarchitektúra.</span><span class="sxs-lookup"><span data-stu-id="59126-154">All the recommendations for the [hub-spoke][guidance-hub-spoke] reference architecture also apply to the shared services reference architecture.</span></span>

<span data-ttu-id="59126-155">Emellett az alábbi javaslatok a a megosztott szolgáltatások legtöbb forgatókönyvre vonatkoznak.</span><span class="sxs-lookup"><span data-stu-id="59126-155">Also, the following recommendations apply for most scenarios under shared services.</span></span> <span data-ttu-id="59126-156">Kövesse ezeket a javaslatokat, ha nincsenek ezeket felülíró követelményei.</span><span class="sxs-lookup"><span data-stu-id="59126-156">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="identity"></a><span data-ttu-id="59126-157">Identitás</span><span class="sxs-lookup"><span data-stu-id="59126-157">Identity</span></span>

<span data-ttu-id="59126-158">A legtöbb vállalati szervezetek Active Directory Directory Services (ADDS) környezet rendelkezik a helyszíni adatközpontját.</span><span class="sxs-lookup"><span data-stu-id="59126-158">Most enterprise organizations have an Active Directory Directory Services (ADDS) environment in their on-premises datacenter.</span></span> <span data-ttu-id="59126-159">Felügyeleti eszközök a helyszíni hálózatból az Azure-bA áthelyezni, ettől az ADDS elősegítése érdekében ajánlott gazdagép ADDS-tartományvezérlőkhöz az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="59126-159">To facilitate management of assets moved to Azure from your on-premises network that depend on ADDS, it is recommended to host ADDS domain controllers in Azure.</span></span>

<span data-ttu-id="59126-160">Ha használja a csoportházirend-objektumok, külön-külön szabályozhatja az Azure és a helyszíni környezetben, a kívánt használja egy másik Active Directory-hely az egyes Azure-régióban.</span><span class="sxs-lookup"><span data-stu-id="59126-160">If you make use of Group Policy Objects, that you want to control separately for Azure and your on-premises environment, use a different AD site for each Azure region.</span></span> <span data-ttu-id="59126-161">Helyezze a tartományvezérlők egy központi virtuális hálózatban (hub) függő számítási feladatokhoz férhet hozzá.</span><span class="sxs-lookup"><span data-stu-id="59126-161">Place your domain controllers in a central VNet (hub) that dependent workloads can access.</span></span>

### <a name="security"></a><span data-ttu-id="59126-162">Biztonság</span><span class="sxs-lookup"><span data-stu-id="59126-162">Security</span></span>

<span data-ttu-id="59126-163">Helyváltoztatáskor számítási feladatokat a helyszíni környezetből az Azure-ba, néhány ilyen számítási feladat kell virtuális gépeken üzemeltetni.</span><span class="sxs-lookup"><span data-stu-id="59126-163">As you move workloads from your on-premises environment to Azure, some of these workloads will require to be hosted in VMs.</span></span> <span data-ttu-id="59126-164">Megfelelőségi okokból szükség lehet kikényszerítheti forgalomról azokat a munkaterheléseket.</span><span class="sxs-lookup"><span data-stu-id="59126-164">For compliance reasons, you may need to enforce restrictions on traffic traversing those workloads.</span></span>

<span data-ttu-id="59126-165">Az Azure biztonságát és teljesítményét szolgáltatások különböző típusú gazdagép-hálózati virtuális berendezések (nva-k) is használhatja.</span><span class="sxs-lookup"><span data-stu-id="59126-165">You can use network virtual appliances (NVAs) in Azure to host different types of security and performance services.</span></span> <span data-ttu-id="59126-166">Ha ismeri a helyszíni berendezések adott halmazát még ma, ajánlott az azonos virtuális berendezések használata az Azure-ban, ha vannak ilyenek.</span><span class="sxs-lookup"><span data-stu-id="59126-166">If you are familiar with a given set of appliances on-premises today, it is recommended to use the same virtualized appliances in Azure, where applicable.</span></span>

> [!NOTE]
> <span data-ttu-id="59126-167">Ez a referenciaarchitektúra üzembe helyezési szkriptjei egy Ubuntu virtuális gép IP-továbbítás engedélyezve van egy hálózati virtuális berendezésen utánzására használja.</span><span class="sxs-lookup"><span data-stu-id="59126-167">The deployment scripts for this reference architecture use an Ubuntu VM with IP forwarding enabled to mimic a network virtual appliance.</span></span>

## <a name="considerations"></a><span data-ttu-id="59126-168">Megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="59126-168">Considerations</span></span>

### <a name="overcoming-vnet-peering-limits"></a><span data-ttu-id="59126-169">A virtuális társhálózatok létesítési korlátjának kiküszöbölése</span><span class="sxs-lookup"><span data-stu-id="59126-169">Overcoming VNet peering limits</span></span>

<span data-ttu-id="59126-170">Mindenképpen vegye figyelembe a [virtuális hálózatonként létesíthető társviszonyok számának korlátozását][vnet-peering-limit] az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="59126-170">Make sure you consider the [limitation on number of VNets peerings per VNet][vnet-peering-limit] in Azure.</span></span> <span data-ttu-id="59126-171">Ha több küllőre van szükség, mint amit a korlátozás megenged, fontolja meg egy agy-küllő-agy-küllő topológia létrehozását, ahol a küllők első szintje agyként is működik.</span><span class="sxs-lookup"><span data-stu-id="59126-171">If you decide you need more spokes than the limit will allow, consider creating a hub-spoke-hub-spoke topology, where the first level of spokes also act as hubs.</span></span> <span data-ttu-id="59126-172">Az alábbi diagram ezt a megközelítést mutatja be.</span><span class="sxs-lookup"><span data-stu-id="59126-172">The following diagram shows this approach.</span></span>

![Eseményközpont-küllős-küllős topológia az Azure-ban](./images/hub-spokehub-spoke.svg)

<span data-ttu-id="59126-174">Ezenkívül vegye figyelembe, mely szolgáltatások vannak megosztva az agyban. Ezzel biztosítható, hogy az agy több küllőre is skálázzon.</span><span class="sxs-lookup"><span data-stu-id="59126-174">Also consider what services are shared in the hub, to ensure the hub scales for a larger number of spokes.</span></span> <span data-ttu-id="59126-175">Például ha az agy tűzfalszolgáltatásokat is nyújt, vegye figyelembe a tűzfal megoldás sávszélességi korlátozásait több küllő hozzáadása esetén.</span><span class="sxs-lookup"><span data-stu-id="59126-175">For instance, if your hub provides firewall services, consider the bandwidth limits of your firewall solution when adding multiple spokes.</span></span> <span data-ttu-id="59126-176">Lehet, hogy érdemes néhány megosztott szolgáltatást az agyak második szintjére mozgatni.</span><span class="sxs-lookup"><span data-stu-id="59126-176">You might want to move some of these shared services to a second level of hubs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="59126-177">A megoldás üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="59126-177">Deploy the solution</span></span>

<span data-ttu-id="59126-178">Ennek az architektúrának egy üzemelő példánya elérhető a [GitHubon][ref-arch-repo].</span><span class="sxs-lookup"><span data-stu-id="59126-178">A deployment for this architecture is available on [GitHub][ref-arch-repo].</span></span> <span data-ttu-id="59126-179">Az üzembe helyezés a következő erőforráscsoportokat hozza létre az előfizetésben:</span><span class="sxs-lookup"><span data-stu-id="59126-179">The deployment creates the following resource groups in your subscription:</span></span>

- <span data-ttu-id="59126-180">eseményközpont-ad-rg</span><span class="sxs-lookup"><span data-stu-id="59126-180">hub-adds-rg</span></span>
- <span data-ttu-id="59126-181">eseményközpont-nva-rg</span><span class="sxs-lookup"><span data-stu-id="59126-181">hub-nva-rg</span></span>
- <span data-ttu-id="59126-182">eseményközpont-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="59126-182">hub-vnet-rg</span></span>
- <span data-ttu-id="59126-183">rendszert-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="59126-183">onprem-vnet-rg</span></span>
- <span data-ttu-id="59126-184">spoke1-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="59126-184">spoke1-vnet-rg</span></span>
- <span data-ttu-id="59126-185">spoke2-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="59126-185">spoke2-vnet-rg</span></span>

<span data-ttu-id="59126-186">A sablon paraméterfájljai ezekre a nevekre hivatkoznak, ezért ha módosítja őket, a paraméterfájlokat is frissítse.</span><span class="sxs-lookup"><span data-stu-id="59126-186">The template parameter files refer to these names, so if you change them, update the parameter files to match.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="59126-187">Előfeltételek</span><span class="sxs-lookup"><span data-stu-id="59126-187">Prerequisites</span></span>

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-the-simulated-on-premises-datacenter-using-azbb"></a><span data-ttu-id="59126-188">Üzembe helyezése az azbb használatával szimulált helyszíni adatközpont</span><span class="sxs-lookup"><span data-stu-id="59126-188">Deploy the simulated on-premises datacenter using azbb</span></span>

<span data-ttu-id="59126-189">Ez a lépés telepíti a szimulált helyszíni adatközpont Azure virtuális hálózatként.</span><span class="sxs-lookup"><span data-stu-id="59126-189">This step deploys the simulated on-premises datacenter as an Azure VNet.</span></span>

1. <span data-ttu-id="59126-190">Keresse meg a `hybrid-networking\shared-services-stack\` mappájában, a GitHub-adattárban.</span><span class="sxs-lookup"><span data-stu-id="59126-190">Navigate to the `hybrid-networking\shared-services-stack\` folder of the GitHub repository.</span></span>

2. <span data-ttu-id="59126-191">Nyissa meg az `onprem.json` fájlt.</span><span class="sxs-lookup"><span data-stu-id="59126-191">Open the `onprem.json` file.</span></span>

3. <span data-ttu-id="59126-192">Keresse meg az összes példányát `UserName`, `adminUserName`,`Password`, és `adminPassword`.</span><span class="sxs-lookup"><span data-stu-id="59126-192">Search for all instances of `UserName`, `adminUserName`,`Password`, and `adminPassword`.</span></span> <span data-ttu-id="59126-193">Adja meg az értékeket a felhasználónevet és jelszót a paraméterek, és mentse a fájlt.</span><span class="sxs-lookup"><span data-stu-id="59126-193">Enter values for the user name and password in the parameters and save the file.</span></span>

4. <span data-ttu-id="59126-194">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="59126-194">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g onprem-vnet-rg -l <location> -p onprem.json --deploy
   ```

5. <span data-ttu-id="59126-195">Várjon, amíg az üzembe helyezés befejeződik.</span><span class="sxs-lookup"><span data-stu-id="59126-195">Wait for the deployment to finish.</span></span> <span data-ttu-id="59126-196">A központi telepítéshez létrehoz egy virtuális hálózatot, a Windows és a egy VPN-átjárót futtató virtuális gépet.</span><span class="sxs-lookup"><span data-stu-id="59126-196">This deployment creates a virtual network, a virtual machine running Windows, and a VPN gateway.</span></span> <span data-ttu-id="59126-197">Egy VPN-átjáró létrehozása 40 percnél is tovább tarthat.</span><span class="sxs-lookup"><span data-stu-id="59126-197">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="deploy-the-hub-vnet"></a><span data-ttu-id="59126-198">Az agyi virtuális hálózat üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="59126-198">Deploy the hub VNet</span></span>

<span data-ttu-id="59126-199">Ebben a lépésben helyez üzembe az agyi virtuális hálózat, és csatlakoztatja a szimulált helyszíni virtuális hálózathoz.</span><span class="sxs-lookup"><span data-stu-id="59126-199">This step deploys the hub VNet and connects it to the simulated on-premises VNet.</span></span>

1. <span data-ttu-id="59126-200">Nyissa meg az `hub-vnet.json` fájlt.</span><span class="sxs-lookup"><span data-stu-id="59126-200">Open the `hub-vnet.json` file.</span></span>

2. <span data-ttu-id="59126-201">Keresse meg `adminPassword` , és adja meg egy felhasználónevet és jelszót a paraméterek.</span><span class="sxs-lookup"><span data-stu-id="59126-201">Search for `adminPassword` and enter a user name and password in the parameters.</span></span>

3. <span data-ttu-id="59126-202">Keresse meg az összes példányát `sharedKey` és a egy megosztott kulcsot adjon meg egy értéket.</span><span class="sxs-lookup"><span data-stu-id="59126-202">Search for all instances of `sharedKey` and enter a value for a shared key.</span></span> <span data-ttu-id="59126-203">Mentse a fájlt.</span><span class="sxs-lookup"><span data-stu-id="59126-203">Save the file.</span></span>

   ```json
   "sharedKey": "abc123",
   ```

4. <span data-ttu-id="59126-204">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="59126-204">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet.json --deploy
   ```

5. <span data-ttu-id="59126-205">Várjon, amíg az üzembe helyezés befejeződik.</span><span class="sxs-lookup"><span data-stu-id="59126-205">Wait for the deployment to finish.</span></span> <span data-ttu-id="59126-206">A központi telepítés egy virtuális hálózatot, egy virtuális gépet, a VPN-átjáró és az előző szakaszban létrehozott átjáróhoz való kapcsolatot hoz létre.</span><span class="sxs-lookup"><span data-stu-id="59126-206">This deployment creates a virtual network, a virtual machine, a VPN gateway, and a connection to the gateway created in the previous section.</span></span> <span data-ttu-id="59126-207">A VPN-átjáró több mint 40 percet is igénybe vehet.</span><span class="sxs-lookup"><span data-stu-id="59126-207">The VPN gateway can take more than 40 minutes to complete.</span></span>

### <a name="deploy-ad-ds-in-azure"></a><span data-ttu-id="59126-208">Az Azure AD DS üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="59126-208">Deploy AD DS in Azure</span></span>

<span data-ttu-id="59126-209">Ebben a lépésben üzembe helyezi az AD DS-tartományvezérlők az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="59126-209">This step deploys AD DS domain controllers in Azure.</span></span>

1. <span data-ttu-id="59126-210">Nyissa meg az `hub-adds.json` fájlt.</span><span class="sxs-lookup"><span data-stu-id="59126-210">Open the `hub-adds.json` file.</span></span>

2. <span data-ttu-id="59126-211">Keresse meg az összes példányát `Password` és `adminPassword`.</span><span class="sxs-lookup"><span data-stu-id="59126-211">Search for all instances of `Password` and `adminPassword`.</span></span> <span data-ttu-id="59126-212">Adja meg az értékeket a felhasználónevet és jelszót a paraméterek, és mentse a fájlt.</span><span class="sxs-lookup"><span data-stu-id="59126-212">Enter values for the user name and password in the parameters and save the file.</span></span>

3. <span data-ttu-id="59126-213">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="59126-213">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-adds-rg -l <location> -p hub-adds.json --deploy
   ```

<span data-ttu-id="59126-214">Ebben a lépésben üzembe helyezés eltarthat néhány percig, mert a két virtuális gép csatlakoztatja a szimulált helyszíni adatközpont tartományhoz, és telepíti őket az AD DS.</span><span class="sxs-lookup"><span data-stu-id="59126-214">This deployment step may take several minutes, because it joins the two VMs to the domain hosted in the simulated on-premises datacenter, and installs AD DS on them.</span></span>

### <a name="deploy-the-spoke-vnets"></a><span data-ttu-id="59126-215">A küllő virtuális hálózatok üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="59126-215">Deploy the spoke VNets</span></span>

<span data-ttu-id="59126-216">Ebben a lépésben üzembe helyezi a küllő virtuális hálózatokhoz.</span><span class="sxs-lookup"><span data-stu-id="59126-216">This step deploys the spoke VNets.</span></span>

1. <span data-ttu-id="59126-217">Nyissa meg az `spoke1.json` fájlt.</span><span class="sxs-lookup"><span data-stu-id="59126-217">Open the `spoke1.json` file.</span></span>

2. <span data-ttu-id="59126-218">Keresse meg `adminPassword` , és adja meg egy felhasználónevet és jelszót a paraméterek.</span><span class="sxs-lookup"><span data-stu-id="59126-218">Search for `adminPassword` and enter a user name and password in the parameters.</span></span>

3. <span data-ttu-id="59126-219">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="59126-219">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke1-vnet-rg -l <location> -p spoke1.json --deploy
   ```

4. <span data-ttu-id="59126-220">Ismételje meg a 1. és 2 fájl `spoke2.json`.</span><span class="sxs-lookup"><span data-stu-id="59126-220">Repeat steps 1 and 2 for the file `spoke2.json`.</span></span>

5. <span data-ttu-id="59126-221">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="59126-221">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke2-vnet-rg -l <location> -p spoke2.json --deploy
   ```

### <a name="peer-the-hub-vnet-to-the-spoke-vnets"></a><span data-ttu-id="59126-222">A küllő virtuális hálózatokhoz az agyi virtuális hálózat társviszonyba állítása</span><span class="sxs-lookup"><span data-stu-id="59126-222">Peer the hub VNet to the spoke VNets</span></span>

<span data-ttu-id="59126-223">Az agyi virtuális hálózat közötti társviszony-létesítési kapcsolat létrehozásához a küllő virtuális hálózatokhoz, futtassa a következő parancsot:</span><span class="sxs-lookup"><span data-stu-id="59126-223">To create a peering connection from the hub VNet to the spoke VNets, run the following command:</span></span>

```bash
azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet-peering.json --deploy
```

### <a name="deploy-the-nva"></a><span data-ttu-id="59126-224">Az nva-t üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="59126-224">Deploy the NVA</span></span>

<span data-ttu-id="59126-225">Ebben a lépésben üzembe helyezi az NVA a `dmz` alhálózat.</span><span class="sxs-lookup"><span data-stu-id="59126-225">This step deploys an NVA in the `dmz` subnet.</span></span>

1. <span data-ttu-id="59126-226">Nyissa meg az `hub-nva.json` fájlt.</span><span class="sxs-lookup"><span data-stu-id="59126-226">Open the `hub-nva.json` file.</span></span>

2. <span data-ttu-id="59126-227">Keresse meg `adminPassword` , és adja meg egy felhasználónevet és jelszót a paraméterek.</span><span class="sxs-lookup"><span data-stu-id="59126-227">Search for `adminPassword` and enter a user name and password in the parameters.</span></span>

3. <span data-ttu-id="59126-228">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="59126-228">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-nva-rg -l <location> -p hub-nva.json --deploy
   ```

### <a name="test-connectivity"></a><span data-ttu-id="59126-229">Kapcsolat tesztelése</span><span class="sxs-lookup"><span data-stu-id="59126-229">Test connectivity</span></span>

<span data-ttu-id="59126-230">Az agyi virtuális hálózat, a szimulált helyszíni környezetből conectivity teszteléséhez.</span><span class="sxs-lookup"><span data-stu-id="59126-230">Test conectivity from the simulated on-premises environment to the hub VNet.</span></span>

1. <span data-ttu-id="59126-231">Az Azure Portal használatával keresse meg a `jb-vm1` nevű virtuális gépet az `onprem-jb-rg` erőforráscsoportban.</span><span class="sxs-lookup"><span data-stu-id="59126-231">Use the Azure portal to find the VM named `jb-vm1` in the `onprem-jb-rg` resource group.</span></span>

2. <span data-ttu-id="59126-232">Kattintson a `Connect` parancsra egy, a virtuális gépre irányuló távoli asztali munkamenet megnyitásához.</span><span class="sxs-lookup"><span data-stu-id="59126-232">Click `Connect` to open a remote desktop session to the VM.</span></span> <span data-ttu-id="59126-233">Használja az `onprem.json` paraméterfájlban megadott jelszót.</span><span class="sxs-lookup"><span data-stu-id="59126-233">Use the password that you specified in the `onprem.json` parameter file.</span></span>

3. <span data-ttu-id="59126-234">Nyisson meg egy PowerShell-konzolt a virtuális gépen, és a `Test-NetConnection` parancsmaggal győződjön meg arról, hogy képes-e csatlakozni a jumpbox virtuális Géphez az agyi virtuális hálózat.</span><span class="sxs-lookup"><span data-stu-id="59126-234">Open a PowerShell console in the VM, and use the `Test-NetConnection` cmdlet to verify that you can connect to the jumpbox VM in the hub VNet.</span></span>

   ```powershell
   Test-NetConnection 10.0.0.68 -CommonTCPPort RDP
   ```

<span data-ttu-id="59126-235">A kimenetnek a következőképpen kell kinéznie:</span><span class="sxs-lookup"><span data-stu-id="59126-235">The output should look similar to the following:</span></span>

```powershell
ComputerName     : 10.0.0.68
RemoteAddress    : 10.0.0.68
RemotePort       : 3389
InterfaceAlias   : Ethernet 2
SourceAddress    : 192.168.1.000
TcpTestSucceeded : True
```

> [!NOTE]
> <span data-ttu-id="59126-236">Alapértelmezés szerint Windows Serveres virtuális gépek nem teszik lehetővé az ICMP-válaszok az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="59126-236">By default, Windows Server VMs do not allow ICMP responses in Azure.</span></span> <span data-ttu-id="59126-237">Ha a használni kívánt `ping` kapcsolat teszteléséhez, engedélyeznie kell a fokozott biztonságú Windows tűzfal az ICMP-forgalmat az egyes virtuális Gépekhez.</span><span class="sxs-lookup"><span data-stu-id="59126-237">If you want to use `ping` to test connectivity, you need to enable ICMP traffic in the Windows Advanced Firewall for each VM.</span></span>

<span data-ttu-id="59126-238">Ismételje meg a sames a küllő virtuális hálózatokhoz csatlakozni:</span><span class="sxs-lookup"><span data-stu-id="59126-238">Repeat the sames steps to test connectivity to the spoke VNets:</span></span>

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
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md

[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
