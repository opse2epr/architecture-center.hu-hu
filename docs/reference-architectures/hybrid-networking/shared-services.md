---
title: A megosztott szolgáltatások hub-küllős hálózati topológia végrehajtása az Azure-ban
description: Hogyan megvalósításához a hub-küllős hálózati topológia megosztott szolgáltatással az Azure-ban.
author: telmosampaio
ms.date: 02/25/2018
pnp.series.title: Implement a hub-spoke network topology with shared services in Azure
pnp.series.prev: hub-spoke
ms.openlocfilehash: 83367a3be2f7a1e33c2ef7018d42f70aae99104d
ms.sourcegitcommit: f665226cec96ec818ca06ac6c2d83edb23c9f29c
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/16/2018
---
# <a name="implement-a-hub-spoke-network-topology-with-shared-services-in-azure"></a><span data-ttu-id="59037-103">A megosztott szolgáltatások hub-küllős hálózati topológia végrehajtása az Azure-ban</span><span class="sxs-lookup"><span data-stu-id="59037-103">Implement a hub-spoke network topology with shared services in Azure</span></span>

<span data-ttu-id="59037-104">A referencia-architektúrában épít, a [hub-küllős] [ guidance-hub-spoke] architektúrával, amelyek közé tartozik a megosztott szolgáltatások összes küllők által felhasználható központban hivatkozik.</span><span class="sxs-lookup"><span data-stu-id="59037-104">This reference architecture builds on the [hub-spoke][guidance-hub-spoke] reference architecture to include shared services in the hub that can be consumed by all spokes.</span></span> <span data-ttu-id="59037-105">Első lépésként a datacenter áttelepítése a felhőbe, és felépítése felé egy [virtuális datacenter], az első szolgáltatásokat kell megosztania identitás- és biztonsági.</span><span class="sxs-lookup"><span data-stu-id="59037-105">As a first step toward migrating a datacenter to the cloud, and building a [virtual datacenter], the first services you need to share are identity and security.</span></span> <span data-ttu-id="59037-106">A referencia-architektúrában bemutatja, hogyan terjeszthető ki az Active Directory-szolgáltatások a helyszíni adatközpontját a Azure-ba, és hozzáadása a hálózati virtuális készülék (NVA), amely működhet, és a tűzfalon, hub-küllős topológiában.</span><span class="sxs-lookup"><span data-stu-id="59037-106">This reference architecture shows you how to extend your Active Directory services from your on-premises datacenter to Azure, and how to add a network virtual appliance (NVA) that can act as a firewall, in a hub-spoke topology.</span></span>  <span data-ttu-id="59037-107">[**A megoldás üzembe helyezése.**](#deploy-the-solution)</span><span class="sxs-lookup"><span data-stu-id="59037-107">[**Deploy this solution**](#deploy-the-solution).</span></span>

<span data-ttu-id="59037-108">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="59037-108">![[0]][0]</span></span>

<span data-ttu-id="59037-109">*Töltse le az ][visio-download]architektúra [Visio-fájlját*</span><span class="sxs-lookup"><span data-stu-id="59037-109">*Download a [Visio file][visio-download] of this architecture*</span></span>

<span data-ttu-id="59037-110">Ez a topológia a következő előnyöket nyújtja:</span><span class="sxs-lookup"><span data-stu-id="59037-110">The benefits of this topology include:</span></span>

* <span data-ttu-id="59037-111">**Költségmegtakarítás** a szolgáltatások központosításával, amelyeket több számítási feladat között, egyetlen helyen lehet elosztani, például a hálózati virtuális berendezések (NVA-k) és DNS-kiszolgálók.</span><span class="sxs-lookup"><span data-stu-id="59037-111">**Cost savings** by centralizing services that can be shared by multiple workloads, such as network virtual appliances (NVAs) and DNS servers, in a single location.</span></span>
* <span data-ttu-id="59037-112">**Az előfizetési korlátok leküzdése** a különböző előfizetésekből virtuális társhálózatok létesítésével a központi agyhoz.</span><span class="sxs-lookup"><span data-stu-id="59037-112">**Overcome subscriptions limits** by peering VNets from different subscriptions to the central hub.</span></span>
* <span data-ttu-id="59037-113">**Kockázatok elkülönítése** a központi IT (SecOps, InfraOps) és a számítási feladatok között (DevOps).</span><span class="sxs-lookup"><span data-stu-id="59037-113">**Separation of concerns** between central IT (SecOps, InfraOps) and workloads (DevOps).</span></span>

<span data-ttu-id="59037-114">Az architektúra gyakori használati módjai többek között a következők:</span><span class="sxs-lookup"><span data-stu-id="59037-114">Typical uses for this architecture include:</span></span>

* <span data-ttu-id="59037-115">A különböző környezetekben (például a fejlesztés, a tesztelés és a gyártás) üzembe helyezett számítási feladatok, amelyek megosztott szolgáltatásokat igényelnek (például DNS, IDS, NTP vagy AD DS).</span><span class="sxs-lookup"><span data-stu-id="59037-115">Workloads deployed in different environments, such as development, testing, and production, that require shared services such as DNS, IDS, NTP, or AD DS.</span></span> <span data-ttu-id="59037-116">A megosztott szolgáltatások az agyi virtuális hálózatban találhatóak, azonban minden környezet egy küllőn van üzembe helyezve, így megmarad az elszigetelés.</span><span class="sxs-lookup"><span data-stu-id="59037-116">Shared services are placed in the hub VNet, while each environment is deployed to a spoke to maintain isolation.</span></span>
* <span data-ttu-id="59037-117">Olyan számítási feladatok, amelyek nem igénylik az egymással való kapcsolatot, de igénylik a megosztott szolgáltatásokhoz való hozzáférést.</span><span class="sxs-lookup"><span data-stu-id="59037-117">Workloads that do not require connectivity to each other, but require access to shared services.</span></span>
* <span data-ttu-id="59037-118">Olyan vállalatok, amelyek központi felügyeletet igényelnek a biztonsági szempontokhoz, például egy tűzfal az agyban DMZ-ként, és küllőnkénti elkülönített felügyelet a számítási feladatokhoz.</span><span class="sxs-lookup"><span data-stu-id="59037-118">Enterprises that require central control over security aspects, such as a firewall in the hub as a DMZ, and segregated management for the workloads in each spoke.</span></span>

## <a name="architecture"></a><span data-ttu-id="59037-119">Architektúra</span><span class="sxs-lookup"><span data-stu-id="59037-119">Architecture</span></span>

<span data-ttu-id="59037-120">Az architektúra a következőkben leírt összetevőkből áll.</span><span class="sxs-lookup"><span data-stu-id="59037-120">The architecture consists of the following components.</span></span>

* <span data-ttu-id="59037-121">**Helyszíni hálózat**.</span><span class="sxs-lookup"><span data-stu-id="59037-121">**On-premises network**.</span></span> <span data-ttu-id="59037-122">A cégen belül futó helyi magánhálózat.</span><span class="sxs-lookup"><span data-stu-id="59037-122">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="59037-123">**VPN-eszköz**.</span><span class="sxs-lookup"><span data-stu-id="59037-123">**VPN device**.</span></span> <span data-ttu-id="59037-124">A helyszíni hálózat számára külső kapcsolatot biztosító eszköz vagy szolgáltatás.</span><span class="sxs-lookup"><span data-stu-id="59037-124">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="59037-125">A VPN-eszköz lehet hardvereszköz vagy valamilyen szoftvermegoldás, amilyen például a Windows Server 2012 Útválasztás és távelérés szolgáltatása (Routing and Remote Access Service, RRAS).</span><span class="sxs-lookup"><span data-stu-id="59037-125">The VPN device may be a hardware device, or a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="59037-126">A támogatott VPN-berendezések listáját és az egyes VPN-berendezések Azure-hoz való csatlakoztatásra történő konfigurálásával kapcsolatos információkat az [Információk a helyek közötti VPN Gateway-kapcsolatok VPN-eszközeiről][vpn-appliance] című cikkben találja.</span><span class="sxs-lookup"><span data-stu-id="59037-126">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="59037-127">**VPN virtuális hálózati átjáró vagy ExpressRoute-átjáró**.</span><span class="sxs-lookup"><span data-stu-id="59037-127">**VPN virtual network gateway or ExpressRoute gateway**.</span></span> <span data-ttu-id="59037-128">A virtuális hálózati átjáró a helyszíni hálózattal létesített kapcsolathoz használt VPN-eszközhöz vagy az ExpressRoute-kapcsolatcsoporthoz való csatlakozást teszi lehetővé a virtuális hálózat számára.</span><span class="sxs-lookup"><span data-stu-id="59037-128">The virtual network gateway enables the VNet to connect to the VPN device, or ExpressRoute circuit, used for connectivity with your on-premises network.</span></span> <span data-ttu-id="59037-129">További információért tekintse át a [helyszíni hálózat és a Microsoft Azure Virtual Network csatlakoztatásával][connect-to-an-Azure-vnet] foglalkozó cikket.</span><span class="sxs-lookup"><span data-stu-id="59037-129">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

> [!NOTE]
> <span data-ttu-id="59037-130">A referenciaarchitektúra üzembe helyezési szkriptjei egy VPN-átjárót használnak a kapcsolat létrehozásához és egy virtuális hálózatot az Azure-ban a helyszíni hálózat szimulációjára.</span><span class="sxs-lookup"><span data-stu-id="59037-130">The deployment scripts for this reference architecture use a VPN gateway for connectivity, and a VNet in Azure to simulate your on-premises network.</span></span>

* <span data-ttu-id="59037-131">**Agyi virtuális hálózat**.</span><span class="sxs-lookup"><span data-stu-id="59037-131">**Hub VNet**.</span></span> <span data-ttu-id="59037-132">Egy Azure virtuális hálózat agyként a küllős hálózati topológiában.</span><span class="sxs-lookup"><span data-stu-id="59037-132">Azure VNet used as the hub in the hub-spoke topology.</span></span> <span data-ttu-id="59037-133">Az agy a helyszíni hálózathoz való csatlakozási lehetőségek központi helye, és a küllős hálózati topológiákban tárolt különféle számítási feladatok által felvett szolgáltatások tárolási helye.</span><span class="sxs-lookup"><span data-stu-id="59037-133">The hub is the central point of connectivity to your on-premises network, and a place to host services that can be consumed by the different workloads hosted in the spoke VNets.</span></span>

* <span data-ttu-id="59037-134">**Átjáró-alhálózat**.</span><span class="sxs-lookup"><span data-stu-id="59037-134">**Gateway subnet**.</span></span> <span data-ttu-id="59037-135">A virtuális hálózati átjárók ugyanazon az alhálózaton találhatók.</span><span class="sxs-lookup"><span data-stu-id="59037-135">The virtual network gateways are held in the same subnet.</span></span>

* <span data-ttu-id="59037-136">**Megosztott szolgáltatások alhálózata**.</span><span class="sxs-lookup"><span data-stu-id="59037-136">**Shared services subnet**.</span></span> <span data-ttu-id="59037-137">Az agyi virtuális hálózat egyik alhálózata, amely olyan szolgáltatások tárolására szolgál, amelyek megoszthatóak minden küllő között, mint például a DNS vagy az AD DS.</span><span class="sxs-lookup"><span data-stu-id="59037-137">A subnet in the hub VNet used to host services that can be shared among all spokes, such as DNS or AD DS.</span></span>

* <span data-ttu-id="59037-138">**DMZ alhálózati**.</span><span class="sxs-lookup"><span data-stu-id="59037-138">**DMZ subnet**.</span></span> <span data-ttu-id="59037-139">A központban alhálózat virtuális hálózat, amely működhet, és biztonsági készülékek, például a tűzfalaknak NVAs tárolására szolgálnak.</span><span class="sxs-lookup"><span data-stu-id="59037-139">A subnet in the hub VNet used to host NVAs that can act as security appliances, such as firewalls.</span></span>

* <span data-ttu-id="59037-140">**Küllő virtuális hálózatok**.</span><span class="sxs-lookup"><span data-stu-id="59037-140">**Spoke VNets**.</span></span> <span data-ttu-id="59037-141">Egy vagy több Azure virtuális hálózat, amelyek küllőként használatosak a küllős topológiában.</span><span class="sxs-lookup"><span data-stu-id="59037-141">One or more Azure VNets that are used as spokes in the hub-spoke topology.</span></span> <span data-ttu-id="59037-142">A küllőkkel elszigetelhetőek a számítási feladatok saját virtuális hálózataikban, így más küllőktől elkülönülten kezelhetőek.</span><span class="sxs-lookup"><span data-stu-id="59037-142">Spokes can be used to isolate workloads in their own VNets, managed separately from other spokes.</span></span> <span data-ttu-id="59037-143">Minden számítási feladat több szintet tartalmazhat, amelyek alhálózatait Azure-terheléselosztók kapcsolják össze.</span><span class="sxs-lookup"><span data-stu-id="59037-143">Each workload might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="59037-144">További információ az alkalmazás-infrastruktúrával kapcsolatban: [Windows rendszerű virtuális gépek számítási feladatainak futtatása][windows-vm-ra] és [Számítási feladatok futtatása Linux rendszerű virtuális gépeken][linux-vm-ra].</span><span class="sxs-lookup"><span data-stu-id="59037-144">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

* <span data-ttu-id="59037-145">**Virtuális társhálózatok létesítése**.</span><span class="sxs-lookup"><span data-stu-id="59037-145">**VNet peering**.</span></span> <span data-ttu-id="59037-146">Két, ugyanabban az Azure-régióban található virtuális hálózatot egy [társviszonykapcsolat][vnet-peering] használatával lehet összekapcsolni.</span><span class="sxs-lookup"><span data-stu-id="59037-146">Two VNets in the same Azure region can be connected using a [peering connection][vnet-peering].</span></span> <span data-ttu-id="59037-147">A társviszonykapcsolatok a virtuális hálózatok közötti nem tranzitív, alacsony késleltetésű kapcsolatok.</span><span class="sxs-lookup"><span data-stu-id="59037-147">Peering connections are non-transitive, low latency connections between VNets.</span></span> <span data-ttu-id="59037-148">A társviszonyba kapcsolt virtuális hálózatok az Azure gerinchálózatát használva, útválasztó igénye nélkül váltanak forgalmat egymás között.</span><span class="sxs-lookup"><span data-stu-id="59037-148">Once peered, the VNets exchange traffic by using the Azure backbone, without the need for a router.</span></span> <span data-ttu-id="59037-149">Egy küllős hálózati topológiában a virtuális társhálózatok létesítésével lehetséges az agyat minden egyes küllőhöz kapcsolni.</span><span class="sxs-lookup"><span data-stu-id="59037-149">In a hub-spoke network topology, you use VNet peering to connect the hub to each spoke.</span></span>

> [!NOTE]
> <span data-ttu-id="59037-150">Ez a cikk csak a [Resource Manager](/azure/azure-resource-manager/resource-group-overview) üzemelő példányokat mutatja be, de ugyanabban az előfizetésben klasszikus virtuális hálózatot is csatlakoztathat Resource Manager virtuális hálózathoz.</span><span class="sxs-lookup"><span data-stu-id="59037-150">This article only covers [Resource Manager](/azure/azure-resource-manager/resource-group-overview) deployments, but you can also connect a classic VNet to a Resource Manager VNet in the same subscription.</span></span> <span data-ttu-id="59037-151">Így a küllők tárolhatnak klasszikus üzemelő példányokat, mégis profitálhatnak az agyban megosztott szolgáltatásokból.</span><span class="sxs-lookup"><span data-stu-id="59037-151">That way, your spokes can host classic deployments and still benefit from services shared in the hub.</span></span>

## <a name="recommendations"></a><span data-ttu-id="59037-152">Javaslatok</span><span class="sxs-lookup"><span data-stu-id="59037-152">Recommendations</span></span>

<span data-ttu-id="59037-153">Kapcsolatos ajánlások a [hub-küllős] [ guidance-hub-spoke] referencia-architektúrában is alkalmazhat a megosztott szolgáltatások referencia-architektúrában.</span><span class="sxs-lookup"><span data-stu-id="59037-153">All the recommendations for the [hub-spoke][guidance-hub-spoke] reference architecture also apply to the shared services reference architecture.</span></span> 

<span data-ttu-id="59037-154">Az alábbi javaslatokat is, a legtöbb esetben a megosztott szolgáltatások alkalmazni.</span><span class="sxs-lookup"><span data-stu-id="59037-154">ALso, the following recommendations apply for most scenarios under shared services.</span></span> <span data-ttu-id="59037-155">Kövesse ezeket a javaslatokat, ha nincsenek ezeket felülíró követelményei.</span><span class="sxs-lookup"><span data-stu-id="59037-155">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="identity"></a><span data-ttu-id="59037-156">Identitás</span><span class="sxs-lookup"><span data-stu-id="59037-156">Identity</span></span>

<span data-ttu-id="59037-157">A legtöbb vállalati szervezet egy Active Directory Directory Services (ADDS) környezettel rendelkezik, azok a helyszíni adatközpontban.</span><span class="sxs-lookup"><span data-stu-id="59037-157">Most enterprise organizations have an Active Directory Directory Services (ADDS) environment in their on-premises datacenter.</span></span> <span data-ttu-id="59037-158">Lehetővé teszi a felügyeleti eszközök áthelyezése az Azure a helyszíni hálózatból ADDS függő, javasoljuk, hogy állomás ADDS tartományvezérlők az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="59037-158">To facilitate management of assets moved to Azure from your on-premises network that depend on ADDS, it is recommended to host ADDS domain controllers in Azure.</span></span>

<span data-ttu-id="59037-159">Ha a csoportházirend-objektumokat, külön-külön szabályozhatja az Azure és a helyszíni környezetben szeretné használni a különböző AD helyet használja minden Azure-régiót.</span><span class="sxs-lookup"><span data-stu-id="59037-159">If you make use of Group Policy Objects, that you want to control separately for Azure and your on-premises environment, use a different AD site for each Azure region.</span></span> <span data-ttu-id="59037-160">A tartományvezérlők tegyen függő munkaterhelések hozzáférő központi VNet (hub).</span><span class="sxs-lookup"><span data-stu-id="59037-160">Place your domain controllers in a central VNet (hub) that dependent workloads can access.</span></span>

### <a name="security"></a><span data-ttu-id="59037-161">Biztonság</span><span class="sxs-lookup"><span data-stu-id="59037-161">Security</span></span>

<span data-ttu-id="59037-162">Amikor munkaterhelések a helyszíni környezetből Azure-ba, az ilyen terhelések némelyike el kell futhat virtuális gépeken.</span><span class="sxs-lookup"><span data-stu-id="59037-162">As you move workloads from your on-premises environment to Azure, some of these workloads will require to be hosted in VMs.</span></span> <span data-ttu-id="59037-163">Megfelelőségi okokból szükség lehet kényszeríteni azokat a munkaterheléseket áthaladó forgalom korlátozásait.</span><span class="sxs-lookup"><span data-stu-id="59037-163">For compliance reasons, you may need to enforce restrictions on traffic traversing those workloads.</span></span> 

<span data-ttu-id="59037-164">Az Azure szolgáltatásban az állomás különböző biztonsági és teljesítménynövelő szolgáltatások hálózati virtuális készülékek (NVAs) is használhatja.</span><span class="sxs-lookup"><span data-stu-id="59037-164">You can use network virtual appliances (NVAs) in Azure to host different types of security and performance services.</span></span> <span data-ttu-id="59037-165">Ha ismeri a helyszíni készülékek adott halmazát ma, javasoljuk, hogy az azonos virtuális készülékek használata Azure adott esetben.</span><span class="sxs-lookup"><span data-stu-id="59037-165">If you are familiar with a given set of appliances on-premises today, it is recommended to use the same virtualized appliances in Azure, where applicable.</span></span>

> [!NOTE]
> <span data-ttu-id="59037-166">Az üzembe helyezési parancsfájlok a referenciaarchitektúra Ubuntu virtuális gép IP-továbbítás engedélyezve van, hogy a hálózati virtuális készülék utánozzák használja.</span><span class="sxs-lookup"><span data-stu-id="59037-166">The deployment scripts for this reference architecture use an Ubuntu VM with IP forwarding enabled to mimic a network virtual appliance.</span></span>

## <a name="considerations"></a><span data-ttu-id="59037-167">Megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="59037-167">Considerations</span></span>

### <a name="overcoming-vnet-peering-limits"></a><span data-ttu-id="59037-168">A virtuális társhálózatok létesítési korlátjának kiküszöbölése</span><span class="sxs-lookup"><span data-stu-id="59037-168">Overcoming VNet peering limits</span></span>

<span data-ttu-id="59037-169">Mindenképpen vegye figyelembe a [virtuális hálózatonként létesíthető társviszonyok számának korlátozását][vnet-peering-limit] az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="59037-169">Make sure you consider the [limitation on number of VNets peerings per VNet][vnet-peering-limit] in Azure.</span></span> <span data-ttu-id="59037-170">Ha több küllőre van szükség, mint amit a korlátozás megenged, fontolja meg egy agy-küllő-agy-küllő topológia létrehozását, ahol a küllők első szintje agyként is működik.</span><span class="sxs-lookup"><span data-stu-id="59037-170">If you decide you need more spokes than the limit will allow, consider creating a hub-spoke-hub-spoke topology, where the first level of spokes also act as hubs.</span></span> <span data-ttu-id="59037-171">Az alábbi diagram ezt a megközelítést mutatja be.</span><span class="sxs-lookup"><span data-stu-id="59037-171">The following diagram shows this approach.</span></span>

<span data-ttu-id="59037-172">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="59037-172">![[3]][3]</span></span>

<span data-ttu-id="59037-173">Ezenkívül vegye figyelembe, mely szolgáltatások vannak megosztva az agyban. Ezzel biztosítható, hogy az agy több küllőre is skálázzon.</span><span class="sxs-lookup"><span data-stu-id="59037-173">Also consider what services are shared in the hub, to ensure the hub scales for a larger number of spokes.</span></span> <span data-ttu-id="59037-174">Például ha az agy tűzfalszolgáltatásokat is nyújt, vegye figyelembe a tűzfal megoldás sávszélességi korlátozásait több küllő hozzáadása esetén.</span><span class="sxs-lookup"><span data-stu-id="59037-174">For instance, if your hub provides firewall services, consider the bandwidth limits of your firewall solution when adding multiple spokes.</span></span> <span data-ttu-id="59037-175">Lehet, hogy érdemes néhány megosztott szolgáltatást az agyak második szintjére mozgatni.</span><span class="sxs-lookup"><span data-stu-id="59037-175">You might want to move some of these shared services to a second level of hubs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="59037-176">A megoldás üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="59037-176">Deploy the solution</span></span>

<span data-ttu-id="59037-177">Ennek az architektúrának egy üzemelő példánya elérhető a [GitHubon][ref-arch-repo].</span><span class="sxs-lookup"><span data-stu-id="59037-177">A deployment for this architecture is available on [GitHub][ref-arch-repo].</span></span> <span data-ttu-id="59037-178">Ez minden virtuális hálózatban Ubuntu virtuális gépeket használ a kapcsolat tesztelésére.</span><span class="sxs-lookup"><span data-stu-id="59037-178">It uses Ubuntu VMs in each VNet to test connectivity.</span></span> <span data-ttu-id="59037-179">Az **agyi virtuális hálózat** **megosztott szolgáltatási** alhálózatában nincsenek tárolva tényleges szolgáltatások.</span><span class="sxs-lookup"><span data-stu-id="59037-179">There are no actual services hosted in the **shared-services** subnet in the **hub VNet**.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="59037-180">Előfeltételek</span><span class="sxs-lookup"><span data-stu-id="59037-180">Prerequisites</span></span>

<span data-ttu-id="59037-181">Mielőtt üzembe helyezhetné saját előfizetésében a referenciaarchitektúrát, az alábbi lépéseket kell elvégeznie.</span><span class="sxs-lookup"><span data-stu-id="59037-181">Before you can deploy the reference architecture to your own subscription, you must perform the following steps.</span></span>

1. <span data-ttu-id="59037-182">Klónozza, ágaztassa vagy a zip-fájl letöltése a [architektúrák hivatkozhat] [ ref-arch-repo] GitHub-tárházban.</span><span class="sxs-lookup"><span data-stu-id="59037-182">Clone, fork, or download the zip file for the [reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="59037-183">Győződjön meg arról, hogy az Azure CLI 2.0 telepítve van a számítógépén.</span><span class="sxs-lookup"><span data-stu-id="59037-183">Make sure you have the Azure CLI 2.0 installed on your computer.</span></span> <span data-ttu-id="59037-184">Információk a CLI telepítésével kapcsolatban: [Az Azure CLI 2.0 telepítése][azure-cli-2].</span><span class="sxs-lookup"><span data-stu-id="59037-184">For CLI installation instructions, see [Install Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="59037-185">Telepítse [az Azure építőelemei][azbb] npm-csomagot.</span><span class="sxs-lookup"><span data-stu-id="59037-185">Install the [Azure building blocks][azbb] npm package.</span></span>

4. <span data-ttu-id="59037-186">Egy parancs parancssori futtatásával, vagy PowerShell kérdés, jelentkezzen be az Azure-fiókjával az alábbi parancs segítségével bash, és kövesse az utasításokat.</span><span class="sxs-lookup"><span data-stu-id="59037-186">From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account by using the command below, and follow the prompts.</span></span>

   ```bash
   az login
   ```

### <a name="deploy-the-simulated-on-premises-datacenter-using-azbb"></a><span data-ttu-id="59037-187">A szimulált olyan helyszíni adatközpontban azbb használatával telepítése</span><span class="sxs-lookup"><span data-stu-id="59037-187">Deploy the simulated on-premises datacenter using azbb</span></span>

<span data-ttu-id="59037-188">A szimulált olyan helyszíni adatközpontban egy Azure virtuális hálózatot, központi telepítéséhez kövesse az alábbi lépéseket:</span><span class="sxs-lookup"><span data-stu-id="59037-188">To deploy the simulated on-premises datacenter as an Azure VNet, follow these steps:</span></span>

1. <span data-ttu-id="59037-189">Navigáljon az előfeltételekre vonatkozó előző lépésekben letöltött adattár `hybrid-networking\shared-services-stack\` mappájához.</span><span class="sxs-lookup"><span data-stu-id="59037-189">Navigate to the `hybrid-networking\shared-services-stack\` folder for the repository you downloaded in the pre-requisites step above.</span></span>

2. <span data-ttu-id="59037-190">Nyissa meg a `onprem.json` fájlt, és adjon meg egy felhasználónevet és jelszót között a mezőkben szereplő idézőjeleket sor 45 és 46, alább látható módon, majd mentse a fájlt.</span><span class="sxs-lookup"><span data-stu-id="59037-190">Open the `onprem.json` file and enter a username and password between the quotes in line 45 and 46, as shown below, then save the file.</span></span>

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```

3. <span data-ttu-id="59037-191">Futtatás `azbb` a szimulált helyi üzemeltetésű környezet telepítése a lent látható módon.</span><span class="sxs-lookup"><span data-stu-id="59037-191">Run `azbb` to deploy the simulated onprem environment as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g onprem-vnet-rg - l <location> -p onoprem.json --deploy
   ```
   > [!NOTE]
   > <span data-ttu-id="59037-192">Ha úgy dönt, más erőforráscsoport-nevet használ (az `onprem-vnet-rg` névtől eltérőt), győződjön meg róla, hogy minden, ezt a nevet viselő paraméterfájlt megkeres és szerkeszti őket, hogy a saját erőforráscsoport-nevét használják.</span><span class="sxs-lookup"><span data-stu-id="59037-192">If you decide to use a different resource group name (other than `onprem-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

4. <span data-ttu-id="59037-193">Várjon, amíg az üzembe helyezés befejeződik.</span><span class="sxs-lookup"><span data-stu-id="59037-193">Wait for the deployment to finish.</span></span> <span data-ttu-id="59037-194">A központi telepítés létrehoz egy virtuális hálózatot, Windows és a VPN-átjáró egy olyan virtuális géphez.</span><span class="sxs-lookup"><span data-stu-id="59037-194">This deployment creates a virtual network, a virtual machine running Windows, and a VPN gateway.</span></span> <span data-ttu-id="59037-195">Egy VPN-átjáró létrehozása 40 percnél is tovább tarthat.</span><span class="sxs-lookup"><span data-stu-id="59037-195">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="azure-hub-vnet"></a><span data-ttu-id="59037-196">Azure agyi virtuális hálózat</span><span class="sxs-lookup"><span data-stu-id="59037-196">Azure hub VNet</span></span>

<span data-ttu-id="59037-197">Az agyi virtuális hálózat üzembe helyezéséhez és a korábban létrehozott, szimulált helyszíni virtuális hálózathoz való kapcsolódáshoz kövesse az alábbi lépéseket.</span><span class="sxs-lookup"><span data-stu-id="59037-197">To deploy the hub VNet, and connect to the simulated on-premises VNet created above, perform the following steps.</span></span>

1. <span data-ttu-id="59037-198">Nyissa meg a `hub-vnet.json` fájlt, és adjon meg egy felhasználónevet és jelszót között a mezőkben szereplő idézőjeleket sor 50 és 51, alább látható módon.</span><span class="sxs-lookup"><span data-stu-id="59037-198">Open the `hub-vnet.json` file and enter a username and password between the quotes in line 50 and 51, as shown below.</span></span>

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```

2. <span data-ttu-id="59037-199">A sor 52, a `osType`, típus `Windows` vagy `Linux` Windows Server 2016 Datacenter, vagy Ubuntu 16.04 a jumpbox az operációs rendszer telepítéséhez.</span><span class="sxs-lookup"><span data-stu-id="59037-199">On line 52, for `osType`, type `Windows` or `Linux` to install either Windows Server 2016 Datacenter, or Ubuntu 16.04 as the operating system for the jumpbox.</span></span>

3. <span data-ttu-id="59037-200">Adjon meg egy megosztott kulcsot sorban 83, az idézőjelek között alább látható módon, majd mentse a fájlt.</span><span class="sxs-lookup"><span data-stu-id="59037-200">Enter a shared key between the quotes in line 83, as shown below, then save the file.</span></span>

   ```bash
   "sharedKey": "",
   ```

4. <span data-ttu-id="59037-201">Futtatás `azbb` a szimulált helyi üzemeltetésű környezet telepítése a lent látható módon.</span><span class="sxs-lookup"><span data-stu-id="59037-201">Run `azbb` to deploy the simulated onprem environment as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg - l <location> -p hub-vnet.json --deploy
   ```
   > [!NOTE]
   > <span data-ttu-id="59037-202">Ha úgy dönt, más erőforráscsoport-nevet használ (az `hub-vnet-rg` névtől eltérőt), győződjön meg róla, hogy minden, ezt a nevet viselő paraméterfájlt megkeres és szerkeszti őket, hogy a saját erőforráscsoport-nevét használják.</span><span class="sxs-lookup"><span data-stu-id="59037-202">If you decide to use a different resource group name (other than `hub-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

5. <span data-ttu-id="59037-203">Várjon, amíg az üzembe helyezés befejeződik.</span><span class="sxs-lookup"><span data-stu-id="59037-203">Wait for the deployment to finish.</span></span> <span data-ttu-id="59037-204">A központi telepítéshez létrehoz egy virtuális hálózatot, virtuális gép, VPN-átjáró és az átjáró, az előző szakaszban létrehozott kapcsolat.</span><span class="sxs-lookup"><span data-stu-id="59037-204">This deployment creates a virtual network, a virtual machine, a VPN gateway, and a connection to the gateway created in the previous section.</span></span> <span data-ttu-id="59037-205">Egy VPN-átjáró létrehozása 40 percnél is tovább tarthat.</span><span class="sxs-lookup"><span data-stu-id="59037-205">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="adds-in-azure"></a><span data-ttu-id="59037-206">Az Azure ad hozzá</span><span class="sxs-lookup"><span data-stu-id="59037-206">ADDS in Azure</span></span>

<span data-ttu-id="59037-207">A tartományvezérlőket az ADDS az Azure-ban, hajtsa végre az alábbi lépéseket.</span><span class="sxs-lookup"><span data-stu-id="59037-207">To deploy the ADDS domain controllers in Azure, perform the following steps.</span></span>

1. <span data-ttu-id="59037-208">Nyissa meg a `hub-adds.json` fájlt, és adjon meg egy felhasználónevet és jelszót a mezőkben szereplő idézőjeleket sorok 14 és 15 közé alább látható módon, majd mentse a fájlt.</span><span class="sxs-lookup"><span data-stu-id="59037-208">Open the `hub-adds.json` file and enter a username and password between the quotes in lines 14 and 15, as shown below, then save the file.</span></span>

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```

2. <span data-ttu-id="59037-209">Futtatás `azbb` a ADDS tartományvezérlők telepítése a lent látható módon.</span><span class="sxs-lookup"><span data-stu-id="59037-209">Run `azbb` to deploy the ADDS domain controllers as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-adds-rg - l <location> -p hub-adds.json --deploy
   ```
  
   > [!NOTE]
   > <span data-ttu-id="59037-210">Ha úgy dönt, más erőforráscsoport-nevet használ (az `hub-adds-rg` névtől eltérőt), győződjön meg róla, hogy minden, ezt a nevet viselő paraméterfájlt megkeres és szerkeszti őket, hogy a saját erőforráscsoport-nevét használják.</span><span class="sxs-lookup"><span data-stu-id="59037-210">If you decide to use a different resource group name (other than `hub-adds-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

   > [!NOTE]
   > <span data-ttu-id="59037-211">Ez a központi telepítés részét eltarthat néhány percig, mivel a két virtuális gép csatlakoztatása a tartományhoz, a szimulált helyszíni adatközpontban, majd az AD DS telepítése rajtuk futó igényel.</span><span class="sxs-lookup"><span data-stu-id="59037-211">This part of the deployment may take several minutes, since it requires joining the two VMs to the domain hosted int he simulated on-premises datacenter, then installing AD DS on them.</span></span>

### <a name="nva"></a><span data-ttu-id="59037-212">NVA</span><span class="sxs-lookup"><span data-stu-id="59037-212">NVA</span></span>

<span data-ttu-id="59037-213">Az NVA telepítéséhez a `dmz` alhálózati, hajtsa végre a következő lépéseket:</span><span class="sxs-lookup"><span data-stu-id="59037-213">To deploy an NVA in the `dmz` subnet, perform the following steps:</span></span>

1. <span data-ttu-id="59037-214">Nyissa meg a `hub-nva.json` fájlt, és adjon meg egy felhasználónevet és jelszót az 14., 13 sorok idézőjelek között alább látható módon, majd mentse a fájlt.</span><span class="sxs-lookup"><span data-stu-id="59037-214">Open the `hub-nva.json` file and enter a username and password between the quotes in lines 13 and 14, as shown below, then save the file.</span></span>

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```
2. <span data-ttu-id="59037-215">Futtatás `azbb` központi telepítése az NVA virtuális gép és a felhasználó által megadott útvonalak.</span><span class="sxs-lookup"><span data-stu-id="59037-215">Run `azbb` to deploy the NVA VM and user defined routes.</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-nva-rg - l <location> -p hub-nva.json --deploy
   ```
   > [!NOTE]
   > <span data-ttu-id="59037-216">Ha úgy dönt, más erőforráscsoport-nevet használ (az `hub-nva-rg` névtől eltérőt), győződjön meg róla, hogy minden, ezt a nevet viselő paraméterfájlt megkeres és szerkeszti őket, hogy a saját erőforráscsoport-nevét használják.</span><span class="sxs-lookup"><span data-stu-id="59037-216">If you decide to use a different resource group name (other than `hub-nva-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

### <a name="azure-spoke-vnets"></a><span data-ttu-id="59037-217">Azure küllő virtuális hálózatok</span><span class="sxs-lookup"><span data-stu-id="59037-217">Azure spoke VNets</span></span>

<span data-ttu-id="59037-218">A Vnetek küllős telepítéséhez hajtsa végre az alábbi lépéseket.</span><span class="sxs-lookup"><span data-stu-id="59037-218">To deploy the spoke VNets, perform the following steps.</span></span>

1. <span data-ttu-id="59037-219">Nyissa meg a `spoke1.json` fájlt, és adjon meg egy felhasználónevet és jelszót között a mezőkben szereplő idézőjeleket sorok 52 és 53, alább látható módon, majd mentse a fájlt.</span><span class="sxs-lookup"><span data-stu-id="59037-219">Open the `spoke1.json` file and enter a username and password between the quotes in lines 52 and 53, as shown below, then save the file.</span></span>

   ```bash
   "adminUsername": "XXX",
   "adminPassword": "YYY",
   ```

2. <span data-ttu-id="59037-220">A sor 54, a `osType`, típus `Windows` vagy `Linux` Windows Server 2016 Datacenter, vagy Ubuntu 16.04 a jumpbox az operációs rendszer telepítéséhez.</span><span class="sxs-lookup"><span data-stu-id="59037-220">On line 54, for `osType`, type `Windows` or `Linux` to install either Windows Server 2016 Datacenter, or Ubuntu 16.04 as the operating system for the jumpbox.</span></span>

3. <span data-ttu-id="59037-221">Futtatás `azbb` központi telepítése az első küllős VNet környezet alább látható módon.</span><span class="sxs-lookup"><span data-stu-id="59037-221">Run `azbb` to deploy the first spoke VNet environment as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke1-vnet-rg - l <location> -p spoke1.json --deploy
   ```
  
   > [!NOTE]
   > <span data-ttu-id="59037-222">Ha úgy dönt, más erőforráscsoport-nevet használ (az `spoke1-vnet-rg` névtől eltérőt), győződjön meg róla, hogy minden, ezt a nevet viselő paraméterfájlt megkeres és szerkeszti őket, hogy a saját erőforráscsoport-nevét használják.</span><span class="sxs-lookup"><span data-stu-id="59037-222">If you decide to use a different resource group name (other than `spoke1-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

4. <span data-ttu-id="59037-223">Ismételje meg az 1-fájl `spoke2.json`.</span><span class="sxs-lookup"><span data-stu-id="59037-223">Repeat step 1 above for file `spoke2.json`.</span></span>

5. <span data-ttu-id="59037-224">Futtatás `azbb` központi telepítése a második küllős VNet környezet alább látható módon.</span><span class="sxs-lookup"><span data-stu-id="59037-224">Run `azbb` to deploy the second spoke VNet environment as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke2-vnet-rg - l <location> -p spoke2.json --deploy
   ```
   > [!NOTE]
   > <span data-ttu-id="59037-225">Ha úgy dönt, más erőforráscsoport-nevet használ (az `spoke2-vnet-rg` névtől eltérőt), győződjön meg róla, hogy minden, ezt a nevet viselő paraméterfájlt megkeres és szerkeszti őket, hogy a saját erőforráscsoport-nevét használják.</span><span class="sxs-lookup"><span data-stu-id="59037-225">If you decide to use a different resource group name (other than `spoke2-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

### <a name="azure-hub-vnet-peering-to-spoke-vnets"></a><span data-ttu-id="59037-226">Azure agyi virtuális társhálózatok létesítése küllő virtuális hálózatokhoz</span><span class="sxs-lookup"><span data-stu-id="59037-226">Azure hub VNet peering to spoke VNets</span></span>

<span data-ttu-id="59037-227">A központ virtuális hálózat számára a Vnetek küllős társviszony-létesítési kapcsolat létrehozásához hajtsa végre az alábbi lépéseket.</span><span class="sxs-lookup"><span data-stu-id="59037-227">To create a peering connection from the hub VNet to the spoke VNets, perform the following steps.</span></span>

1. <span data-ttu-id="59037-228">Nyissa meg a `hub-vnet-peering.json` fájlt, és ellenőrizze, hogy az egyes a virtuális hálózati társviszony sor 29 kezdve az erőforráscsoport nevét, és a virtuális hálózat neve helyesen-e.</span><span class="sxs-lookup"><span data-stu-id="59037-228">Open the `hub-vnet-peering.json` file and verify that the resource group name, and virtual network name for each of the virtual network peerings starting in line 29 are correct.</span></span>

2. <span data-ttu-id="59037-229">Futtatás `azbb` központi telepítése az első küllős VNet környezet alább látható módon.</span><span class="sxs-lookup"><span data-stu-id="59037-229">Run `azbb` to deploy the first spoke VNet environment as shown below.</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg - l <location> -p hub-vnet-peering.json --deploy
   ```

   > [!NOTE]
   > <span data-ttu-id="59037-230">Ha úgy dönt, más erőforráscsoport-nevet használ (az `hub-vnet-rg` névtől eltérőt), győződjön meg róla, hogy minden, ezt a nevet viselő paraméterfájlt megkeres és szerkeszti őket, hogy a saját erőforráscsoport-nevét használják.</span><span class="sxs-lookup"><span data-stu-id="59037-230">If you decide to use a different resource group name (other than `hub-vnet-rg`), make sure to search for all parameter files that use that name and edit them to use your own resource group name.</span></span>

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
[virtuális datacenter]: https://aka.ms/vdc
[virtual datacenter]: https://aka.ms/vdc
[vnet-peering]: /azure/virtual-network/virtual-network-peering-overview
[vnet-peering-limit]: /azure/azure-subscription-service-limits#networking-limits
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[windows-vm-ra]: ../virtual-machines-windows/index.md

[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-hub-spoke.vsdx
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[0]: ./images/shared-services.png "A megosztott szolgáltatások topológia az Azure-ban"
[3]: ./images/hub-spokehub-spoke.svg "Küllős-küllős topológia az Azure-ban"
[ARM-Templates]: https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/
