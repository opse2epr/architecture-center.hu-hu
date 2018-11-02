---
title: Egy küllős hálózati topológia közös szolgáltatásokkal megvalósítása az Azure-ban
description: Hogyan lehet közös szolgáltatásokkal küllős hálózati topológia implementálása az Azure-ban.
author: telmosampaio
ms.date: 10/09/2018
pnp.series.title: Implement a hub-spoke network topology with shared services in Azure
pnp.series.prev: hub-spoke
ms.openlocfilehash: 209791950c79760ea8aaafc77ff779d6207410ce
ms.sourcegitcommit: dbbf914757b03cdee7a274204f9579fa63d7eed2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/02/2018
ms.locfileid: "50916278"
---
# <a name="implement-a-hub-spoke-network-topology-with-shared-services-in-azure"></a><span data-ttu-id="1e43f-103">Közös szolgáltatásokkal küllős hálózati topológia implementálása az Azure-ban</span><span class="sxs-lookup"><span data-stu-id="1e43f-103">Implement a hub-spoke network topology with shared services in Azure</span></span>

<span data-ttu-id="1e43f-104">Ez a referenciaarchitektúra épül, amely a [küllős] [ guidance-hub-spoke] referenciaarchitektúra, amely minden küllő tudják használni az agyban megosztott szolgáltatások tartalmazza.</span><span class="sxs-lookup"><span data-stu-id="1e43f-104">This reference architecture builds on the [hub-spoke][guidance-hub-spoke] reference architecture to include shared services in the hub that can be consumed by all spokes.</span></span> <span data-ttu-id="1e43f-105">A felhőbe való migrálás egy adatközpontban, és létrehozását az első lépés egy [virtuális adatközpont], meg kell osztania az első szolgáltatások az identitások és a biztonság.</span><span class="sxs-lookup"><span data-stu-id="1e43f-105">As a first step toward migrating a datacenter to the cloud, and building a [virtual datacenter], the first services you need to share are identity and security.</span></span> <span data-ttu-id="1e43f-106">Ez a referenciaarchitektúra bemutatja, hogyan terjeszthető ki az Active Directory-szolgáltatások a helyi adatközpontból az Azure, és hogyan adhat hozzá egy hálózati virtuális készüléket (NVA) működő, a tűzfal egy küllős topológiában.</span><span class="sxs-lookup"><span data-stu-id="1e43f-106">This reference architecture shows you how to extend your Active Directory services from your on-premises datacenter to Azure, and how to add a network virtual appliance (NVA) that can act as a firewall, in a hub-spoke topology.</span></span>  <span data-ttu-id="1e43f-107">[**A megoldás üzembe helyezése.**](#deploy-the-solution)</span><span class="sxs-lookup"><span data-stu-id="1e43f-107">[**Deploy this solution**](#deploy-the-solution).</span></span>

<span data-ttu-id="1e43f-108">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="1e43f-108">![[0]][0]</span></span>

<span data-ttu-id="1e43f-109">*Töltse le az architektúra [Visio-fájlját][visio-download]*</span><span class="sxs-lookup"><span data-stu-id="1e43f-109">*Download a [Visio file][visio-download] of this architecture*</span></span>

<span data-ttu-id="1e43f-110">Ez a topológia előnyei a következők:</span><span class="sxs-lookup"><span data-stu-id="1e43f-110">The benefits of this topology include:</span></span>

* <span data-ttu-id="1e43f-111">**Költségmegtakarítás** a szolgáltatások központosításával, amelyeket több számítási feladat között, egyetlen helyen lehet elosztani, például a hálózati virtuális berendezések (NVA-k) és DNS-kiszolgálók.</span><span class="sxs-lookup"><span data-stu-id="1e43f-111">**Cost savings** by centralizing services that can be shared by multiple workloads, such as network virtual appliances (NVAs) and DNS servers, in a single location.</span></span>
* <span data-ttu-id="1e43f-112">**Az előfizetési korlátok leküzdése** a különböző előfizetésekből virtuális társhálózatok létesítésével a központi agyhoz.</span><span class="sxs-lookup"><span data-stu-id="1e43f-112">**Overcome subscriptions limits** by peering VNets from different subscriptions to the central hub.</span></span>
* <span data-ttu-id="1e43f-113">**Kockázatok elkülönítése** a központi IT (SecOps, InfraOps) és a számítási feladatok között (DevOps).</span><span class="sxs-lookup"><span data-stu-id="1e43f-113">**Separation of concerns** between central IT (SecOps, InfraOps) and workloads (DevOps).</span></span>

<span data-ttu-id="1e43f-114">Az architektúra gyakori használati módjai többek között a következők:</span><span class="sxs-lookup"><span data-stu-id="1e43f-114">Typical uses for this architecture include:</span></span>

* <span data-ttu-id="1e43f-115">A különböző környezetekben (például a fejlesztés, a tesztelés és a gyártás) üzembe helyezett számítási feladatok, amelyek megosztott szolgáltatásokat igényelnek (például DNS, IDS, NTP vagy AD DS).</span><span class="sxs-lookup"><span data-stu-id="1e43f-115">Workloads deployed in different environments, such as development, testing, and production, that require shared services such as DNS, IDS, NTP, or AD DS.</span></span> <span data-ttu-id="1e43f-116">A megosztott szolgáltatások az agyi virtuális hálózatban találhatóak, azonban minden környezet egy küllőn van üzembe helyezve, így megmarad az elszigetelés.</span><span class="sxs-lookup"><span data-stu-id="1e43f-116">Shared services are placed in the hub VNet, while each environment is deployed to a spoke to maintain isolation.</span></span>
* <span data-ttu-id="1e43f-117">Olyan számítási feladatok, amelyek nem igénylik az egymással való kapcsolatot, de igénylik a megosztott szolgáltatásokhoz való hozzáférést.</span><span class="sxs-lookup"><span data-stu-id="1e43f-117">Workloads that do not require connectivity to each other, but require access to shared services.</span></span>
* <span data-ttu-id="1e43f-118">Olyan vállalatok, amelyek központi felügyeletet igényelnek a biztonsági szempontokhoz, például egy tűzfal az agyban DMZ-ként, és küllőnkénti elkülönített felügyelet a számítási feladatokhoz.</span><span class="sxs-lookup"><span data-stu-id="1e43f-118">Enterprises that require central control over security aspects, such as a firewall in the hub as a DMZ, and segregated management for the workloads in each spoke.</span></span>

## <a name="architecture"></a><span data-ttu-id="1e43f-119">Architektúra</span><span class="sxs-lookup"><span data-stu-id="1e43f-119">Architecture</span></span>

<span data-ttu-id="1e43f-120">Az architektúra a következőkben leírt összetevőkből áll.</span><span class="sxs-lookup"><span data-stu-id="1e43f-120">The architecture consists of the following components.</span></span>

* <span data-ttu-id="1e43f-121">**Helyszíni hálózat**.</span><span class="sxs-lookup"><span data-stu-id="1e43f-121">**On-premises network**.</span></span> <span data-ttu-id="1e43f-122">A cégen belül futó helyi magánhálózat.</span><span class="sxs-lookup"><span data-stu-id="1e43f-122">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="1e43f-123">**VPN-eszköz**.</span><span class="sxs-lookup"><span data-stu-id="1e43f-123">**VPN device**.</span></span> <span data-ttu-id="1e43f-124">A helyszíni hálózat számára külső kapcsolatot biztosító eszköz vagy szolgáltatás.</span><span class="sxs-lookup"><span data-stu-id="1e43f-124">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="1e43f-125">A VPN-eszköz lehet hardvereszköz vagy valamilyen szoftvermegoldás, amilyen például a Windows Server 2012 Útválasztás és távelérés szolgáltatása (Routing and Remote Access Service, RRAS).</span><span class="sxs-lookup"><span data-stu-id="1e43f-125">The VPN device may be a hardware device, or a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="1e43f-126">A támogatott VPN-berendezések listáját és az egyes VPN-berendezések Azure-hoz való csatlakoztatásra történő konfigurálásával kapcsolatos információkat az [Információk a helyek közötti VPN Gateway-kapcsolatok VPN-eszközeiről][vpn-appliance] című cikkben találja.</span><span class="sxs-lookup"><span data-stu-id="1e43f-126">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="1e43f-127">**VPN virtuális hálózati átjáró vagy ExpressRoute-átjáró**.</span><span class="sxs-lookup"><span data-stu-id="1e43f-127">**VPN virtual network gateway or ExpressRoute gateway**.</span></span> <span data-ttu-id="1e43f-128">A virtuális hálózati átjáró a helyszíni hálózattal létesített kapcsolathoz használt VPN-eszközhöz vagy az ExpressRoute-kapcsolatcsoporthoz való csatlakozást teszi lehetővé a virtuális hálózat számára.</span><span class="sxs-lookup"><span data-stu-id="1e43f-128">The virtual network gateway enables the VNet to connect to the VPN device, or ExpressRoute circuit, used for connectivity with your on-premises network.</span></span> <span data-ttu-id="1e43f-129">További információért tekintse át a [helyszíni hálózat és a Microsoft Azure Virtual Network csatlakoztatásával][connect-to-an-Azure-vnet] foglalkozó cikket.</span><span class="sxs-lookup"><span data-stu-id="1e43f-129">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

> [!NOTE]
> <span data-ttu-id="1e43f-130">A referenciaarchitektúra üzembe helyezési szkriptjei egy VPN-átjárót használnak a kapcsolat létrehozásához és egy virtuális hálózatot az Azure-ban a helyszíni hálózat szimulációjára.</span><span class="sxs-lookup"><span data-stu-id="1e43f-130">The deployment scripts for this reference architecture use a VPN gateway for connectivity, and a VNet in Azure to simulate your on-premises network.</span></span>

* <span data-ttu-id="1e43f-131">**Agyi virtuális hálózat**.</span><span class="sxs-lookup"><span data-stu-id="1e43f-131">**Hub VNet**.</span></span> <span data-ttu-id="1e43f-132">Egy Azure virtuális hálózat agyként a küllős hálózati topológiában.</span><span class="sxs-lookup"><span data-stu-id="1e43f-132">Azure VNet used as the hub in the hub-spoke topology.</span></span> <span data-ttu-id="1e43f-133">Az agy a helyszíni hálózathoz való csatlakozási lehetőségek központi helye, és a küllős hálózati topológiákban tárolt különféle számítási feladatok által felvett szolgáltatások tárolási helye.</span><span class="sxs-lookup"><span data-stu-id="1e43f-133">The hub is the central point of connectivity to your on-premises network, and a place to host services that can be consumed by the different workloads hosted in the spoke VNets.</span></span>

* <span data-ttu-id="1e43f-134">**Átjáró-alhálózat**.</span><span class="sxs-lookup"><span data-stu-id="1e43f-134">**Gateway subnet**.</span></span> <span data-ttu-id="1e43f-135">A virtuális hálózati átjárók ugyanazon az alhálózaton találhatók.</span><span class="sxs-lookup"><span data-stu-id="1e43f-135">The virtual network gateways are held in the same subnet.</span></span>

* <span data-ttu-id="1e43f-136">**Megosztott szolgáltatások alhálózata**.</span><span class="sxs-lookup"><span data-stu-id="1e43f-136">**Shared services subnet**.</span></span> <span data-ttu-id="1e43f-137">Az agyi virtuális hálózat egyik alhálózata, amely olyan szolgáltatások tárolására szolgál, amelyek megoszthatóak minden küllő között, mint például a DNS vagy az AD DS.</span><span class="sxs-lookup"><span data-stu-id="1e43f-137">A subnet in the hub VNet used to host services that can be shared among all spokes, such as DNS or AD DS.</span></span>

* <span data-ttu-id="1e43f-138">**DMZ-alhálózat**.</span><span class="sxs-lookup"><span data-stu-id="1e43f-138">**DMZ subnet**.</span></span> <span data-ttu-id="1e43f-139">Az agyi virtuális hálózat a gazdagép nva-kkal működő, biztonsági eszközökkel, például a tűzfalak használt alhálózat.</span><span class="sxs-lookup"><span data-stu-id="1e43f-139">A subnet in the hub VNet used to host NVAs that can act as security appliances, such as firewalls.</span></span>

* <span data-ttu-id="1e43f-140">**Küllő virtuális hálózatok**.</span><span class="sxs-lookup"><span data-stu-id="1e43f-140">**Spoke VNets**.</span></span> <span data-ttu-id="1e43f-141">Egy vagy több Azure virtuális hálózat, amelyek küllőként használatosak a küllős topológiában.</span><span class="sxs-lookup"><span data-stu-id="1e43f-141">One or more Azure VNets that are used as spokes in the hub-spoke topology.</span></span> <span data-ttu-id="1e43f-142">A küllőkkel elszigetelhetőek a számítási feladatok saját virtuális hálózataikban, így más küllőktől elkülönülten kezelhetőek.</span><span class="sxs-lookup"><span data-stu-id="1e43f-142">Spokes can be used to isolate workloads in their own VNets, managed separately from other spokes.</span></span> <span data-ttu-id="1e43f-143">Minden számítási feladat több szintet tartalmazhat, amelyek alhálózatait Azure-terheléselosztók kapcsolják össze.</span><span class="sxs-lookup"><span data-stu-id="1e43f-143">Each workload might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="1e43f-144">További információ az alkalmazás-infrastruktúrával kapcsolatban: [Windows rendszerű virtuális gépek számítási feladatainak futtatása][windows-vm-ra] és [Számítási feladatok futtatása Linux rendszerű virtuális gépeken][linux-vm-ra].</span><span class="sxs-lookup"><span data-stu-id="1e43f-144">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

* <span data-ttu-id="1e43f-145">**Virtuális társhálózatok létesítése**.</span><span class="sxs-lookup"><span data-stu-id="1e43f-145">**VNet peering**.</span></span> <span data-ttu-id="1e43f-146">Két, ugyanabban az Azure-régióban található virtuális hálózatot egy [társviszonykapcsolat][vnet-peering] használatával lehet összekapcsolni.</span><span class="sxs-lookup"><span data-stu-id="1e43f-146">Two VNets in the same Azure region can be connected using a [peering connection][vnet-peering].</span></span> <span data-ttu-id="1e43f-147">A társviszonykapcsolatok a virtuális hálózatok közötti nem tranzitív, alacsony késleltetésű kapcsolatok.</span><span class="sxs-lookup"><span data-stu-id="1e43f-147">Peering connections are non-transitive, low latency connections between VNets.</span></span> <span data-ttu-id="1e43f-148">A társviszonyba kapcsolt virtuális hálózatok az Azure gerinchálózatát használva, útválasztó igénye nélkül váltanak forgalmat egymás között.</span><span class="sxs-lookup"><span data-stu-id="1e43f-148">Once peered, the VNets exchange traffic by using the Azure backbone, without the need for a router.</span></span> <span data-ttu-id="1e43f-149">Egy küllős hálózati topológiában a virtuális társhálózatok létesítésével lehetséges az agyat minden egyes küllőhöz kapcsolni.</span><span class="sxs-lookup"><span data-stu-id="1e43f-149">In a hub-spoke network topology, you use VNet peering to connect the hub to each spoke.</span></span>

> [!NOTE]
> <span data-ttu-id="1e43f-150">Ez a cikk csak a [Resource Manager](/azure/azure-resource-manager/resource-group-overview) üzemelő példányokat mutatja be, de ugyanabban az előfizetésben klasszikus virtuális hálózatot is csatlakoztathat Resource Manager virtuális hálózathoz.</span><span class="sxs-lookup"><span data-stu-id="1e43f-150">This article only covers [Resource Manager](/azure/azure-resource-manager/resource-group-overview) deployments, but you can also connect a classic VNet to a Resource Manager VNet in the same subscription.</span></span> <span data-ttu-id="1e43f-151">Így a küllők tárolhatnak klasszikus üzemelő példányokat, mégis profitálhatnak az agyban megosztott szolgáltatásokból.</span><span class="sxs-lookup"><span data-stu-id="1e43f-151">That way, your spokes can host classic deployments and still benefit from services shared in the hub.</span></span>

## <a name="recommendations"></a><span data-ttu-id="1e43f-152">Javaslatok</span><span class="sxs-lookup"><span data-stu-id="1e43f-152">Recommendations</span></span>

<span data-ttu-id="1e43f-153">Az összes javaslatot a [küllős] [ guidance-hub-spoke] referenciaarchitektúra is alkalmazni kell a megosztott szolgáltatások referenciaarchitektúra.</span><span class="sxs-lookup"><span data-stu-id="1e43f-153">All the recommendations for the [hub-spoke][guidance-hub-spoke] reference architecture also apply to the shared services reference architecture.</span></span> 

<span data-ttu-id="1e43f-154">Emellett az alábbi javaslatok a a megosztott szolgáltatások legtöbb forgatókönyvre vonatkoznak.</span><span class="sxs-lookup"><span data-stu-id="1e43f-154">Also, the following recommendations apply for most scenarios under shared services.</span></span> <span data-ttu-id="1e43f-155">Kövesse ezeket a javaslatokat, ha nincsenek ezeket felülíró követelményei.</span><span class="sxs-lookup"><span data-stu-id="1e43f-155">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="identity"></a><span data-ttu-id="1e43f-156">Identitás</span><span class="sxs-lookup"><span data-stu-id="1e43f-156">Identity</span></span>

<span data-ttu-id="1e43f-157">A legtöbb vállalati szervezetek Active Directory Directory Services (ADDS) környezet rendelkezik a helyszíni adatközpontját.</span><span class="sxs-lookup"><span data-stu-id="1e43f-157">Most enterprise organizations have an Active Directory Directory Services (ADDS) environment in their on-premises datacenter.</span></span> <span data-ttu-id="1e43f-158">Felügyeleti eszközök a helyszíni hálózatból az Azure-bA áthelyezni, ettől az ADDS elősegítése érdekében ajánlott gazdagép ADDS-tartományvezérlőkhöz az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="1e43f-158">To facilitate management of assets moved to Azure from your on-premises network that depend on ADDS, it is recommended to host ADDS domain controllers in Azure.</span></span>

<span data-ttu-id="1e43f-159">Ha használja a csoportházirend-objektumok, külön-külön szabályozhatja az Azure és a helyszíni környezetben, a kívánt használja egy másik Active Directory-hely az egyes Azure-régióban.</span><span class="sxs-lookup"><span data-stu-id="1e43f-159">If you make use of Group Policy Objects, that you want to control separately for Azure and your on-premises environment, use a different AD site for each Azure region.</span></span> <span data-ttu-id="1e43f-160">Helyezze a tartományvezérlők egy központi virtuális hálózatban (hub) függő számítási feladatokhoz férhet hozzá.</span><span class="sxs-lookup"><span data-stu-id="1e43f-160">Place your domain controllers in a central VNet (hub) that dependent workloads can access.</span></span>

### <a name="security"></a><span data-ttu-id="1e43f-161">Biztonság</span><span class="sxs-lookup"><span data-stu-id="1e43f-161">Security</span></span>

<span data-ttu-id="1e43f-162">Helyváltoztatáskor számítási feladatokat a helyszíni környezetből az Azure-ba, néhány ilyen számítási feladat kell virtuális gépeken üzemeltetni.</span><span class="sxs-lookup"><span data-stu-id="1e43f-162">As you move workloads from your on-premises environment to Azure, some of these workloads will require to be hosted in VMs.</span></span> <span data-ttu-id="1e43f-163">Megfelelőségi okokból szükség lehet kikényszerítheti forgalomról azokat a munkaterheléseket.</span><span class="sxs-lookup"><span data-stu-id="1e43f-163">For compliance reasons, you may need to enforce restrictions on traffic traversing those workloads.</span></span> 

<span data-ttu-id="1e43f-164">Az Azure biztonságát és teljesítményét szolgáltatások különböző típusú gazdagép-hálózati virtuális berendezések (nva-k) is használhatja.</span><span class="sxs-lookup"><span data-stu-id="1e43f-164">You can use network virtual appliances (NVAs) in Azure to host different types of security and performance services.</span></span> <span data-ttu-id="1e43f-165">Ha ismeri a helyszíni berendezések adott halmazát még ma, ajánlott az azonos virtuális berendezések használata az Azure-ban, ha vannak ilyenek.</span><span class="sxs-lookup"><span data-stu-id="1e43f-165">If you are familiar with a given set of appliances on-premises today, it is recommended to use the same virtualized appliances in Azure, where applicable.</span></span>

> [!NOTE]
> <span data-ttu-id="1e43f-166">Ez a referenciaarchitektúra üzembe helyezési szkriptjei egy Ubuntu virtuális gép IP-továbbítás engedélyezve van egy hálózati virtuális berendezésen utánzására használja.</span><span class="sxs-lookup"><span data-stu-id="1e43f-166">The deployment scripts for this reference architecture use an Ubuntu VM with IP forwarding enabled to mimic a network virtual appliance.</span></span>

## <a name="considerations"></a><span data-ttu-id="1e43f-167">Megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="1e43f-167">Considerations</span></span>

### <a name="overcoming-vnet-peering-limits"></a><span data-ttu-id="1e43f-168">A virtuális társhálózatok létesítési korlátjának kiküszöbölése</span><span class="sxs-lookup"><span data-stu-id="1e43f-168">Overcoming VNet peering limits</span></span>

<span data-ttu-id="1e43f-169">Mindenképpen vegye figyelembe a [virtuális hálózatonként létesíthető társviszonyok számának korlátozását][vnet-peering-limit] az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="1e43f-169">Make sure you consider the [limitation on number of VNets peerings per VNet][vnet-peering-limit] in Azure.</span></span> <span data-ttu-id="1e43f-170">Ha több küllőre van szükség, mint amit a korlátozás megenged, fontolja meg egy agy-küllő-agy-küllő topológia létrehozását, ahol a küllők első szintje agyként is működik.</span><span class="sxs-lookup"><span data-stu-id="1e43f-170">If you decide you need more spokes than the limit will allow, consider creating a hub-spoke-hub-spoke topology, where the first level of spokes also act as hubs.</span></span> <span data-ttu-id="1e43f-171">Az alábbi diagram ezt a megközelítést mutatja be.</span><span class="sxs-lookup"><span data-stu-id="1e43f-171">The following diagram shows this approach.</span></span>

<span data-ttu-id="1e43f-172">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="1e43f-172">![[3]][3]</span></span>

<span data-ttu-id="1e43f-173">Ezenkívül vegye figyelembe, mely szolgáltatások vannak megosztva az agyban. Ezzel biztosítható, hogy az agy több küllőre is skálázzon.</span><span class="sxs-lookup"><span data-stu-id="1e43f-173">Also consider what services are shared in the hub, to ensure the hub scales for a larger number of spokes.</span></span> <span data-ttu-id="1e43f-174">Például ha az agy tűzfalszolgáltatásokat is nyújt, vegye figyelembe a tűzfal megoldás sávszélességi korlátozásait több küllő hozzáadása esetén.</span><span class="sxs-lookup"><span data-stu-id="1e43f-174">For instance, if your hub provides firewall services, consider the bandwidth limits of your firewall solution when adding multiple spokes.</span></span> <span data-ttu-id="1e43f-175">Lehet, hogy érdemes néhány megosztott szolgáltatást az agyak második szintjére mozgatni.</span><span class="sxs-lookup"><span data-stu-id="1e43f-175">You might want to move some of these shared services to a second level of hubs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="1e43f-176">A megoldás üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="1e43f-176">Deploy the solution</span></span>

<span data-ttu-id="1e43f-177">Ennek az architektúrának egy üzemelő példánya elérhető a [GitHubon][ref-arch-repo].</span><span class="sxs-lookup"><span data-stu-id="1e43f-177">A deployment for this architecture is available on [GitHub][ref-arch-repo].</span></span> <span data-ttu-id="1e43f-178">Az üzembe helyezés a következő erőforráscsoportokat hozza létre az előfizetésben:</span><span class="sxs-lookup"><span data-stu-id="1e43f-178">The deployment creates the following resource groups in your subscription:</span></span>

- <span data-ttu-id="1e43f-179">eseményközpont-ad-rg</span><span class="sxs-lookup"><span data-stu-id="1e43f-179">hub-adds-rg</span></span>
- <span data-ttu-id="1e43f-180">eseményközpont-nva-rg</span><span class="sxs-lookup"><span data-stu-id="1e43f-180">hub-nva-rg</span></span>
- <span data-ttu-id="1e43f-181">eseményközpont-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="1e43f-181">hub-vnet-rg</span></span>
- <span data-ttu-id="1e43f-182">rendszert-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="1e43f-182">onprem-vnet-rg</span></span>
- <span data-ttu-id="1e43f-183">spoke1-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="1e43f-183">spoke1-vnet-rg</span></span>
- <span data-ttu-id="1e43f-184">spoke2-művele-rg</span><span class="sxs-lookup"><span data-stu-id="1e43f-184">spoke2-vent-rg</span></span>

<span data-ttu-id="1e43f-185">A sablon paraméterfájljai ezekre a nevekre hivatkoznak, ezért ha módosítja őket, a paraméterfájlokat is frissítse.</span><span class="sxs-lookup"><span data-stu-id="1e43f-185">The template parameter files refer to these names, so if you change them, update the parameter files to match.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="1e43f-186">Előfeltételek</span><span class="sxs-lookup"><span data-stu-id="1e43f-186">Prerequisites</span></span>

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-the-simulated-on-premises-datacenter-using-azbb"></a><span data-ttu-id="1e43f-187">Üzembe helyezése az azbb használatával szimulált helyszíni adatközpont</span><span class="sxs-lookup"><span data-stu-id="1e43f-187">Deploy the simulated on-premises datacenter using azbb</span></span>

<span data-ttu-id="1e43f-188">Ez a lépés telepíti a szimulált helyszíni adatközpont Azure virtuális hálózatként.</span><span class="sxs-lookup"><span data-stu-id="1e43f-188">This step deploys the simulated on-premises datacenter as an Azure VNet.</span></span>

1. <span data-ttu-id="1e43f-189">Keresse meg a `hybrid-networking\shared-services-stack\` mappájában, a GitHub-adattárban.</span><span class="sxs-lookup"><span data-stu-id="1e43f-189">Navigate to the `hybrid-networking\shared-services-stack\` folder of the GitHub repository.</span></span>

2. <span data-ttu-id="1e43f-190">Nyissa meg az `onprem.json` fájlt.</span><span class="sxs-lookup"><span data-stu-id="1e43f-190">Open the `onprem.json` file.</span></span> 

3. <span data-ttu-id="1e43f-191">Keresse meg az összes példányát `UserName`, `adminUserName`,`Password`, és `adminPassword`.</span><span class="sxs-lookup"><span data-stu-id="1e43f-191">Search for all instances of `UserName`, `adminUserName`,`Password`, and `adminPassword`.</span></span> <span data-ttu-id="1e43f-192">Adja meg az értékeket a felhasználónevet és jelszót a paraméterek, és mentse a fájlt.</span><span class="sxs-lookup"><span data-stu-id="1e43f-192">Enter values for the user name and password in the parameters and save the file.</span></span> 

4. <span data-ttu-id="1e43f-193">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="1e43f-193">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g onprem-vnet-rg -l <location> -p onprem.json --deploy
   ```
5. <span data-ttu-id="1e43f-194">Várjon, amíg az üzembe helyezés befejeződik.</span><span class="sxs-lookup"><span data-stu-id="1e43f-194">Wait for the deployment to finish.</span></span> <span data-ttu-id="1e43f-195">A központi telepítéshez létrehoz egy virtuális hálózatot, a Windows és a egy VPN-átjárót futtató virtuális gépet.</span><span class="sxs-lookup"><span data-stu-id="1e43f-195">This deployment creates a virtual network, a virtual machine running Windows, and a VPN gateway.</span></span> <span data-ttu-id="1e43f-196">Egy VPN-átjáró létrehozása 40 percnél is tovább tarthat.</span><span class="sxs-lookup"><span data-stu-id="1e43f-196">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="deploy-the-hub-vnet"></a><span data-ttu-id="1e43f-197">Az agyi virtuális hálózat üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="1e43f-197">Deploy the hub VNet</span></span>

<span data-ttu-id="1e43f-198">Ebben a lépésben helyez üzembe az agyi virtuális hálózat, és csatlakoztatja a szimulált helyszíni virtuális hálózathoz.</span><span class="sxs-lookup"><span data-stu-id="1e43f-198">This step deploys the hub VNet and connects it to the simulated on-premises VNet.</span></span>

1. <span data-ttu-id="1e43f-199">Nyissa meg az `hub-vnet.json` fájlt.</span><span class="sxs-lookup"><span data-stu-id="1e43f-199">Open the `hub-vnet.json` file.</span></span> 

2. <span data-ttu-id="1e43f-200">Keresse meg `adminPassword` , és adja meg egy felhasználónevet és jelszót a paraméterek.</span><span class="sxs-lookup"><span data-stu-id="1e43f-200">Search for `adminPassword` and enter a user name and password in the parameters.</span></span> 

3. <span data-ttu-id="1e43f-201">Keresse meg az összes példányát `sharedKey` és a egy megosztott kulcsot adjon meg egy értéket.</span><span class="sxs-lookup"><span data-stu-id="1e43f-201">Search for all instances of `sharedKey` and enter a value for a shared key.</span></span> <span data-ttu-id="1e43f-202">Mentse a fájlt.</span><span class="sxs-lookup"><span data-stu-id="1e43f-202">Save the file.</span></span>

   ```bash
   "sharedKey": "abc123",
   ```

4. <span data-ttu-id="1e43f-203">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="1e43f-203">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet.json --deploy
   ```

5. <span data-ttu-id="1e43f-204">Várjon, amíg az üzembe helyezés befejeződik.</span><span class="sxs-lookup"><span data-stu-id="1e43f-204">Wait for the deployment to finish.</span></span> <span data-ttu-id="1e43f-205">A központi telepítés egy virtuális hálózatot, egy virtuális gépet, a VPN-átjáró és az előző szakaszban létrehozott átjáróhoz való kapcsolatot hoz létre.</span><span class="sxs-lookup"><span data-stu-id="1e43f-205">This deployment creates a virtual network, a virtual machine, a VPN gateway, and a connection to the gateway created in the previous section.</span></span> <span data-ttu-id="1e43f-206">A VPN-átjáró több mint 40 percet is igénybe vehet.</span><span class="sxs-lookup"><span data-stu-id="1e43f-206">The VPN gateway can take more than 40 minutes to complete.</span></span>

### <a name="deploy-ad-ds-in-azure"></a><span data-ttu-id="1e43f-207">Az Azure AD DS üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="1e43f-207">Deploy AD DS in Azure</span></span>

<span data-ttu-id="1e43f-208">Ebben a lépésben üzembe helyezi az AD DS-tartományvezérlők az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="1e43f-208">This step deploys AD DS domain controllers in Azure.</span></span>

1. <span data-ttu-id="1e43f-209">Nyissa meg az `hub-adds.json` fájlt.</span><span class="sxs-lookup"><span data-stu-id="1e43f-209">Open the `hub-adds.json` file.</span></span>

2. <span data-ttu-id="1e43f-210">Keresse meg az összes példányát `Password` és `adminPassword`.</span><span class="sxs-lookup"><span data-stu-id="1e43f-210">Search for all instances of `Password` and `adminPassword`.</span></span> <span data-ttu-id="1e43f-211">Adja meg az értékeket a felhasználónevet és jelszót a paraméterek, és mentse a fájlt.</span><span class="sxs-lookup"><span data-stu-id="1e43f-211">Enter values for the user name and password in the parameters and save the file.</span></span> 

3. <span data-ttu-id="1e43f-212">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="1e43f-212">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-adds-rg -l <location> -p hub-adds.json --deploy
   ```
  
<span data-ttu-id="1e43f-213">Ebben a lépésben üzembe helyezés eltarthat néhány percig, mert a két virtuális gép csatlakoztatja a szimulált helyszíni adatközpont tartományhoz, és telepíti őket az AD DS.</span><span class="sxs-lookup"><span data-stu-id="1e43f-213">This deployment step may take several minutes, because it joins the two VMs to the domain hosted in the simulated on-premises datacenter, and installs AD DS on them.</span></span>

### <a name="deploy-the-spoke-vnets"></a><span data-ttu-id="1e43f-214">A küllő virtuális hálózatok üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="1e43f-214">Deploy the spoke VNets</span></span>

<span data-ttu-id="1e43f-215">Ebben a lépésben üzembe helyezi a küllő virtuális hálózatokhoz.</span><span class="sxs-lookup"><span data-stu-id="1e43f-215">This step deploys the spoke VNets.</span></span>

1. <span data-ttu-id="1e43f-216">Nyissa meg az `spoke1.json` fájlt.</span><span class="sxs-lookup"><span data-stu-id="1e43f-216">Open the `spoke1.json` file.</span></span>

2. <span data-ttu-id="1e43f-217">Keresse meg `adminPassword` , és adja meg egy felhasználónevet és jelszót a paraméterek.</span><span class="sxs-lookup"><span data-stu-id="1e43f-217">Search for `adminPassword` and enter a user name and password in the parameters.</span></span> 

3. <span data-ttu-id="1e43f-218">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="1e43f-218">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke1-vnet-rg -l <location> -p spoke1.json --deploy
   ```
  
4. <span data-ttu-id="1e43f-219">Ismételje meg a 1. és 2 fájl `spoke2.json`.</span><span class="sxs-lookup"><span data-stu-id="1e43f-219">Repeat steps 1 and 2 for the file `spoke2.json`.</span></span>

5. <span data-ttu-id="1e43f-220">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="1e43f-220">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke2-vnet-rg -l <location> -p spoke2.json --deploy
   ```

### <a name="peer-the-hub-vnet-to-the-spoke-vnets"></a><span data-ttu-id="1e43f-221">A küllő virtuális hálózatokhoz az agyi virtuális hálózat társviszonyba állítása</span><span class="sxs-lookup"><span data-stu-id="1e43f-221">Peer the hub VNet to the spoke VNets</span></span>

<span data-ttu-id="1e43f-222">Az agyi virtuális hálózat közötti társviszony-létesítési kapcsolat létrehozásához a küllő virtuális hálózatokhoz, futtassa a következő parancsot:</span><span class="sxs-lookup"><span data-stu-id="1e43f-222">To create a peering connection from the hub VNet to the spoke VNets, run the following command:</span></span>

```bash
azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet-peering.json --deploy
```

### <a name="deploy-the-nva"></a><span data-ttu-id="1e43f-223">Az nva-t üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="1e43f-223">Deploy the NVA</span></span>

<span data-ttu-id="1e43f-224">Ebben a lépésben üzembe helyezi az NVA a `dmz` alhálózat.</span><span class="sxs-lookup"><span data-stu-id="1e43f-224">This step deploys an NVA in the `dmz` subnet.</span></span>

1. <span data-ttu-id="1e43f-225">Nyissa meg az `hub-nva.json` fájlt.</span><span class="sxs-lookup"><span data-stu-id="1e43f-225">Open the `hub-nva.json` file.</span></span>

2. <span data-ttu-id="1e43f-226">Keresse meg `adminPassword` , és adja meg egy felhasználónevet és jelszót a paraméterek.</span><span class="sxs-lookup"><span data-stu-id="1e43f-226">Search for `adminPassword` and enter a user name and password in the parameters.</span></span> 

3. <span data-ttu-id="1e43f-227">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="1e43f-227">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-nva-rg -l <location> -p hub-nva.json --deploy
   ```

### <a name="test-connectivity"></a><span data-ttu-id="1e43f-228">Kapcsolat tesztelése</span><span class="sxs-lookup"><span data-stu-id="1e43f-228">Test connectivity</span></span> 

<span data-ttu-id="1e43f-229">Az agyi virtuális hálózat, a szimulált helyszíni környezetből conectivity teszteléséhez.</span><span class="sxs-lookup"><span data-stu-id="1e43f-229">Test conectivity from the simulated on-premises environment to the hub VNet.</span></span>

1. <span data-ttu-id="1e43f-230">Az Azure Portal használatával keresse meg a `jb-vm1` nevű virtuális gépet az `onprem-jb-rg` erőforráscsoportban.</span><span class="sxs-lookup"><span data-stu-id="1e43f-230">Use the Azure portal to find the VM named `jb-vm1` in the `onprem-jb-rg` resource group.</span></span>

2. <span data-ttu-id="1e43f-231">Kattintson a `Connect` parancsra egy, a virtuális gépre irányuló távoli asztali munkamenet megnyitásához.</span><span class="sxs-lookup"><span data-stu-id="1e43f-231">Click `Connect` to open a remote desktop session to the VM.</span></span> <span data-ttu-id="1e43f-232">Használja az `onprem.json` paraméterfájlban megadott jelszót.</span><span class="sxs-lookup"><span data-stu-id="1e43f-232">Use the password that you specified in the `onprem.json` parameter file.</span></span>

3. <span data-ttu-id="1e43f-233">Nyisson meg egy PowerShell-konzolt a virtuális gépen, és a `Test-NetConnection` parancsmaggal győződjön meg arról, hogy képes-e csatlakozni a jumpbox virtuális Géphez az agyi virtuális hálózat.</span><span class="sxs-lookup"><span data-stu-id="1e43f-233">Open a PowerShell console in the VM, and use the `Test-NetConnection` cmdlet to verify that you can connect to the jumpbox VM in the hub VNet.</span></span>

   ```powershell
   Test-NetConnection 10.0.0.68 -CommonTCPPort RDP
   ```
<span data-ttu-id="1e43f-234">A kimenetnek a következőképpen kell kinéznie:</span><span class="sxs-lookup"><span data-stu-id="1e43f-234">The output should look similar to the following:</span></span>

```powershell
ComputerName     : 10.0.0.68
RemoteAddress    : 10.0.0.68
RemotePort       : 3389
InterfaceAlias   : Ethernet 2
SourceAddress    : 192.168.1.000
TcpTestSucceeded : True
```

> [!NOTE]
> <span data-ttu-id="1e43f-235">Alapértelmezés szerint Windows Serveres virtuális gépek nem teszik lehetővé az ICMP-válaszok az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="1e43f-235">By default, Windows Server VMs do not allow ICMP responses in Azure.</span></span> <span data-ttu-id="1e43f-236">Ha a használni kívánt `ping` kapcsolat teszteléséhez, engedélyeznie kell a fokozott biztonságú Windows tűzfal az ICMP-forgalmat az egyes virtuális Gépekhez.</span><span class="sxs-lookup"><span data-stu-id="1e43f-236">If you want to use `ping` to test connectivity, you need to enable ICMP traffic in the Windows Advanced Firewall for each VM.</span></span>

<span data-ttu-id="1e43f-237">Ismételje meg a sames a küllő virtuális hálózatokhoz csatlakozni:</span><span class="sxs-lookup"><span data-stu-id="1e43f-237">Repeat the sames steps to test connectivity to the spoke VNets:</span></span>

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
[0]: ./images/shared-services.png "Megosztott szolgáltatások topológia az Azure-ban"
[3]: ./images/hub-spokehub-spoke.svg "Küllős-küllős topológia az Azure-ban"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
