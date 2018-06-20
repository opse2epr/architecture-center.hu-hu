---
title: A megosztott szolgáltatások hub-küllős hálózati topológia végrehajtása az Azure-ban
description: Hogyan megvalósításához a hub-küllős hálózati topológia megosztott szolgáltatással az Azure-ban.
author: telmosampaio
ms.date: 06/19/2018
pnp.series.title: Implement a hub-spoke network topology with shared services in Azure
pnp.series.prev: hub-spoke
ms.openlocfilehash: 5e5029dd7de78c6953229364f9e8ae2789c2b348
ms.sourcegitcommit: f7418f8bdabc8f5ec33ae3551e3fbb466782caa5
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 06/19/2018
ms.locfileid: "36209559"
---
# <a name="implement-a-hub-spoke-network-topology-with-shared-services-in-azure"></a><span data-ttu-id="8f057-103">A megosztott szolgáltatások hub-küllős hálózati topológia végrehajtása az Azure-ban</span><span class="sxs-lookup"><span data-stu-id="8f057-103">Implement a hub-spoke network topology with shared services in Azure</span></span>

<span data-ttu-id="8f057-104">A referencia-architektúrában épít, a [hub-küllős] [ guidance-hub-spoke] architektúrával, amelyek közé tartozik a megosztott szolgáltatások összes küllők által felhasználható központban hivatkozik.</span><span class="sxs-lookup"><span data-stu-id="8f057-104">This reference architecture builds on the [hub-spoke][guidance-hub-spoke] reference architecture to include shared services in the hub that can be consumed by all spokes.</span></span> <span data-ttu-id="8f057-105">Első lépésként a datacenter áttelepítése a felhőbe, és felépítése felé egy [virtuális datacenter], az első szolgáltatásokat kell megosztania identitás- és biztonsági.</span><span class="sxs-lookup"><span data-stu-id="8f057-105">As a first step toward migrating a datacenter to the cloud, and building a [virtual datacenter], the first services you need to share are identity and security.</span></span> <span data-ttu-id="8f057-106">A referencia-architektúrában bemutatja, hogyan terjeszthető ki az Active Directory-szolgáltatások a helyszíni adatközpontját a Azure-ba, és hozzáadása a hálózati virtuális készülék (NVA), amely működhet, és a tűzfalon, hub-küllős topológiában.</span><span class="sxs-lookup"><span data-stu-id="8f057-106">This reference architecture shows you how to extend your Active Directory services from your on-premises datacenter to Azure, and how to add a network virtual appliance (NVA) that can act as a firewall, in a hub-spoke topology.</span></span>  <span data-ttu-id="8f057-107">[**A megoldás üzembe helyezése.**](#deploy-the-solution)</span><span class="sxs-lookup"><span data-stu-id="8f057-107">[**Deploy this solution**](#deploy-the-solution).</span></span>

<span data-ttu-id="8f057-108">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="8f057-108">![[0]][0]</span></span>

<span data-ttu-id="8f057-109">*Töltse le az ][visio-download]architektúra [Visio-fájlját*</span><span class="sxs-lookup"><span data-stu-id="8f057-109">*Download a [Visio file][visio-download] of this architecture*</span></span>

<span data-ttu-id="8f057-110">Ez a topológia a következő előnyöket nyújtja:</span><span class="sxs-lookup"><span data-stu-id="8f057-110">The benefits of this topology include:</span></span>

* <span data-ttu-id="8f057-111">**Költségmegtakarítás** a szolgáltatások központosításával, amelyeket több számítási feladat között, egyetlen helyen lehet elosztani, például a hálózati virtuális berendezések (NVA-k) és DNS-kiszolgálók.</span><span class="sxs-lookup"><span data-stu-id="8f057-111">**Cost savings** by centralizing services that can be shared by multiple workloads, such as network virtual appliances (NVAs) and DNS servers, in a single location.</span></span>
* <span data-ttu-id="8f057-112">**Az előfizetési korlátok leküzdése** a különböző előfizetésekből virtuális társhálózatok létesítésével a központi agyhoz.</span><span class="sxs-lookup"><span data-stu-id="8f057-112">**Overcome subscriptions limits** by peering VNets from different subscriptions to the central hub.</span></span>
* <span data-ttu-id="8f057-113">**Kockázatok elkülönítése** a központi IT (SecOps, InfraOps) és a számítási feladatok között (DevOps).</span><span class="sxs-lookup"><span data-stu-id="8f057-113">**Separation of concerns** between central IT (SecOps, InfraOps) and workloads (DevOps).</span></span>

<span data-ttu-id="8f057-114">Az architektúra gyakori használati módjai többek között a következők:</span><span class="sxs-lookup"><span data-stu-id="8f057-114">Typical uses for this architecture include:</span></span>

* <span data-ttu-id="8f057-115">A különböző környezetekben (például a fejlesztés, a tesztelés és a gyártás) üzembe helyezett számítási feladatok, amelyek megosztott szolgáltatásokat igényelnek (például DNS, IDS, NTP vagy AD DS).</span><span class="sxs-lookup"><span data-stu-id="8f057-115">Workloads deployed in different environments, such as development, testing, and production, that require shared services such as DNS, IDS, NTP, or AD DS.</span></span> <span data-ttu-id="8f057-116">A megosztott szolgáltatások az agyi virtuális hálózatban találhatóak, azonban minden környezet egy küllőn van üzembe helyezve, így megmarad az elszigetelés.</span><span class="sxs-lookup"><span data-stu-id="8f057-116">Shared services are placed in the hub VNet, while each environment is deployed to a spoke to maintain isolation.</span></span>
* <span data-ttu-id="8f057-117">Olyan számítási feladatok, amelyek nem igénylik az egymással való kapcsolatot, de igénylik a megosztott szolgáltatásokhoz való hozzáférést.</span><span class="sxs-lookup"><span data-stu-id="8f057-117">Workloads that do not require connectivity to each other, but require access to shared services.</span></span>
* <span data-ttu-id="8f057-118">Olyan vállalatok, amelyek központi felügyeletet igényelnek a biztonsági szempontokhoz, például egy tűzfal az agyban DMZ-ként, és küllőnkénti elkülönített felügyelet a számítási feladatokhoz.</span><span class="sxs-lookup"><span data-stu-id="8f057-118">Enterprises that require central control over security aspects, such as a firewall in the hub as a DMZ, and segregated management for the workloads in each spoke.</span></span>

## <a name="architecture"></a><span data-ttu-id="8f057-119">Architektúra</span><span class="sxs-lookup"><span data-stu-id="8f057-119">Architecture</span></span>

<span data-ttu-id="8f057-120">Az architektúra a következőkben leírt összetevőkből áll.</span><span class="sxs-lookup"><span data-stu-id="8f057-120">The architecture consists of the following components.</span></span>

* <span data-ttu-id="8f057-121">**Helyszíni hálózat**.</span><span class="sxs-lookup"><span data-stu-id="8f057-121">**On-premises network**.</span></span> <span data-ttu-id="8f057-122">A cégen belül futó helyi magánhálózat.</span><span class="sxs-lookup"><span data-stu-id="8f057-122">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="8f057-123">**VPN-eszköz**.</span><span class="sxs-lookup"><span data-stu-id="8f057-123">**VPN device**.</span></span> <span data-ttu-id="8f057-124">A helyszíni hálózat számára külső kapcsolatot biztosító eszköz vagy szolgáltatás.</span><span class="sxs-lookup"><span data-stu-id="8f057-124">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="8f057-125">A VPN-eszköz lehet hardvereszköz vagy valamilyen szoftvermegoldás, amilyen például a Windows Server 2012 Útválasztás és távelérés szolgáltatása (Routing and Remote Access Service, RRAS).</span><span class="sxs-lookup"><span data-stu-id="8f057-125">The VPN device may be a hardware device, or a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="8f057-126">A támogatott VPN-berendezések listáját és az egyes VPN-berendezések Azure-hoz való csatlakoztatásra történő konfigurálásával kapcsolatos információkat az [Információk a helyek közötti VPN Gateway-kapcsolatok VPN-eszközeiről][vpn-appliance] című cikkben találja.</span><span class="sxs-lookup"><span data-stu-id="8f057-126">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="8f057-127">**VPN virtuális hálózati átjáró vagy ExpressRoute-átjáró**.</span><span class="sxs-lookup"><span data-stu-id="8f057-127">**VPN virtual network gateway or ExpressRoute gateway**.</span></span> <span data-ttu-id="8f057-128">A virtuális hálózati átjáró a helyszíni hálózattal létesített kapcsolathoz használt VPN-eszközhöz vagy az ExpressRoute-kapcsolatcsoporthoz való csatlakozást teszi lehetővé a virtuális hálózat számára.</span><span class="sxs-lookup"><span data-stu-id="8f057-128">The virtual network gateway enables the VNet to connect to the VPN device, or ExpressRoute circuit, used for connectivity with your on-premises network.</span></span> <span data-ttu-id="8f057-129">További információért tekintse át a [helyszíni hálózat és a Microsoft Azure Virtual Network csatlakoztatásával][connect-to-an-Azure-vnet] foglalkozó cikket.</span><span class="sxs-lookup"><span data-stu-id="8f057-129">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

> [!NOTE]
> <span data-ttu-id="8f057-130">A referenciaarchitektúra üzembe helyezési szkriptjei egy VPN-átjárót használnak a kapcsolat létrehozásához és egy virtuális hálózatot az Azure-ban a helyszíni hálózat szimulációjára.</span><span class="sxs-lookup"><span data-stu-id="8f057-130">The deployment scripts for this reference architecture use a VPN gateway for connectivity, and a VNet in Azure to simulate your on-premises network.</span></span>

* <span data-ttu-id="8f057-131">**Agyi virtuális hálózat**.</span><span class="sxs-lookup"><span data-stu-id="8f057-131">**Hub VNet**.</span></span> <span data-ttu-id="8f057-132">Egy Azure virtuális hálózat agyként a küllős hálózati topológiában.</span><span class="sxs-lookup"><span data-stu-id="8f057-132">Azure VNet used as the hub in the hub-spoke topology.</span></span> <span data-ttu-id="8f057-133">Az agy a helyszíni hálózathoz való csatlakozási lehetőségek központi helye, és a küllős hálózati topológiákban tárolt különféle számítási feladatok által felvett szolgáltatások tárolási helye.</span><span class="sxs-lookup"><span data-stu-id="8f057-133">The hub is the central point of connectivity to your on-premises network, and a place to host services that can be consumed by the different workloads hosted in the spoke VNets.</span></span>

* <span data-ttu-id="8f057-134">**Átjáró-alhálózat**.</span><span class="sxs-lookup"><span data-stu-id="8f057-134">**Gateway subnet**.</span></span> <span data-ttu-id="8f057-135">A virtuális hálózati átjárók ugyanazon az alhálózaton találhatók.</span><span class="sxs-lookup"><span data-stu-id="8f057-135">The virtual network gateways are held in the same subnet.</span></span>

* <span data-ttu-id="8f057-136">**Megosztott szolgáltatások alhálózata**.</span><span class="sxs-lookup"><span data-stu-id="8f057-136">**Shared services subnet**.</span></span> <span data-ttu-id="8f057-137">Az agyi virtuális hálózat egyik alhálózata, amely olyan szolgáltatások tárolására szolgál, amelyek megoszthatóak minden küllő között, mint például a DNS vagy az AD DS.</span><span class="sxs-lookup"><span data-stu-id="8f057-137">A subnet in the hub VNet used to host services that can be shared among all spokes, such as DNS or AD DS.</span></span>

* <span data-ttu-id="8f057-138">**DMZ alhálózati**.</span><span class="sxs-lookup"><span data-stu-id="8f057-138">**DMZ subnet**.</span></span> <span data-ttu-id="8f057-139">A központban alhálózat virtuális hálózat, amely működhet, és biztonsági készülékek, például a tűzfalaknak NVAs tárolására szolgálnak.</span><span class="sxs-lookup"><span data-stu-id="8f057-139">A subnet in the hub VNet used to host NVAs that can act as security appliances, such as firewalls.</span></span>

* <span data-ttu-id="8f057-140">**Küllő virtuális hálózatok**.</span><span class="sxs-lookup"><span data-stu-id="8f057-140">**Spoke VNets**.</span></span> <span data-ttu-id="8f057-141">Egy vagy több Azure virtuális hálózat, amelyek küllőként használatosak a küllős topológiában.</span><span class="sxs-lookup"><span data-stu-id="8f057-141">One or more Azure VNets that are used as spokes in the hub-spoke topology.</span></span> <span data-ttu-id="8f057-142">A küllőkkel elszigetelhetőek a számítási feladatok saját virtuális hálózataikban, így más küllőktől elkülönülten kezelhetőek.</span><span class="sxs-lookup"><span data-stu-id="8f057-142">Spokes can be used to isolate workloads in their own VNets, managed separately from other spokes.</span></span> <span data-ttu-id="8f057-143">Minden számítási feladat több szintet tartalmazhat, amelyek alhálózatait Azure-terheléselosztók kapcsolják össze.</span><span class="sxs-lookup"><span data-stu-id="8f057-143">Each workload might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="8f057-144">További információ az alkalmazás-infrastruktúrával kapcsolatban: [Windows rendszerű virtuális gépek számítási feladatainak futtatása][windows-vm-ra] és [Számítási feladatok futtatása Linux rendszerű virtuális gépeken][linux-vm-ra].</span><span class="sxs-lookup"><span data-stu-id="8f057-144">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

* <span data-ttu-id="8f057-145">**Virtuális társhálózatok létesítése**.</span><span class="sxs-lookup"><span data-stu-id="8f057-145">**VNet peering**.</span></span> <span data-ttu-id="8f057-146">Két, ugyanabban az Azure-régióban található virtuális hálózatot egy [társviszonykapcsolat][vnet-peering] használatával lehet összekapcsolni.</span><span class="sxs-lookup"><span data-stu-id="8f057-146">Two VNets in the same Azure region can be connected using a [peering connection][vnet-peering].</span></span> <span data-ttu-id="8f057-147">A társviszonykapcsolatok a virtuális hálózatok közötti nem tranzitív, alacsony késleltetésű kapcsolatok.</span><span class="sxs-lookup"><span data-stu-id="8f057-147">Peering connections are non-transitive, low latency connections between VNets.</span></span> <span data-ttu-id="8f057-148">A társviszonyba kapcsolt virtuális hálózatok az Azure gerinchálózatát használva, útválasztó igénye nélkül váltanak forgalmat egymás között.</span><span class="sxs-lookup"><span data-stu-id="8f057-148">Once peered, the VNets exchange traffic by using the Azure backbone, without the need for a router.</span></span> <span data-ttu-id="8f057-149">Egy küllős hálózati topológiában a virtuális társhálózatok létesítésével lehetséges az agyat minden egyes küllőhöz kapcsolni.</span><span class="sxs-lookup"><span data-stu-id="8f057-149">In a hub-spoke network topology, you use VNet peering to connect the hub to each spoke.</span></span>

> [!NOTE]
> <span data-ttu-id="8f057-150">Ez a cikk csak a [Resource Manager](/azure/azure-resource-manager/resource-group-overview) üzemelő példányokat mutatja be, de ugyanabban az előfizetésben klasszikus virtuális hálózatot is csatlakoztathat Resource Manager virtuális hálózathoz.</span><span class="sxs-lookup"><span data-stu-id="8f057-150">This article only covers [Resource Manager](/azure/azure-resource-manager/resource-group-overview) deployments, but you can also connect a classic VNet to a Resource Manager VNet in the same subscription.</span></span> <span data-ttu-id="8f057-151">Így a küllők tárolhatnak klasszikus üzemelő példányokat, mégis profitálhatnak az agyban megosztott szolgáltatásokból.</span><span class="sxs-lookup"><span data-stu-id="8f057-151">That way, your spokes can host classic deployments and still benefit from services shared in the hub.</span></span>

## <a name="recommendations"></a><span data-ttu-id="8f057-152">Ajánlatok</span><span class="sxs-lookup"><span data-stu-id="8f057-152">Recommendations</span></span>

<span data-ttu-id="8f057-153">Kapcsolatos ajánlások a [hub-küllős] [ guidance-hub-spoke] referencia-architektúrában is alkalmazhat a megosztott szolgáltatások referencia-architektúrában.</span><span class="sxs-lookup"><span data-stu-id="8f057-153">All the recommendations for the [hub-spoke][guidance-hub-spoke] reference architecture also apply to the shared services reference architecture.</span></span> 

<span data-ttu-id="8f057-154">Az alábbi javaslatokat is, a legtöbb esetben a megosztott szolgáltatások alkalmazni.</span><span class="sxs-lookup"><span data-stu-id="8f057-154">ALso, the following recommendations apply for most scenarios under shared services.</span></span> <span data-ttu-id="8f057-155">Kövesse ezeket a javaslatokat, ha nincsenek ezeket felülíró követelményei.</span><span class="sxs-lookup"><span data-stu-id="8f057-155">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="identity"></a><span data-ttu-id="8f057-156">Identitáskezelés</span><span class="sxs-lookup"><span data-stu-id="8f057-156">Identity</span></span>

<span data-ttu-id="8f057-157">A legtöbb vállalati szervezet egy Active Directory Directory Services (ADDS) környezettel rendelkezik, azok a helyszíni adatközpontban.</span><span class="sxs-lookup"><span data-stu-id="8f057-157">Most enterprise organizations have an Active Directory Directory Services (ADDS) environment in their on-premises datacenter.</span></span> <span data-ttu-id="8f057-158">Lehetővé teszi a felügyeleti eszközök áthelyezése az Azure a helyszíni hálózatból ADDS függő, javasoljuk, hogy állomás ADDS tartományvezérlők az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="8f057-158">To facilitate management of assets moved to Azure from your on-premises network that depend on ADDS, it is recommended to host ADDS domain controllers in Azure.</span></span>

<span data-ttu-id="8f057-159">Ha a csoportházirend-objektumokat, külön-külön szabályozhatja az Azure és a helyszíni környezetben szeretné használni a különböző AD helyet használja minden Azure-régiót.</span><span class="sxs-lookup"><span data-stu-id="8f057-159">If you make use of Group Policy Objects, that you want to control separately for Azure and your on-premises environment, use a different AD site for each Azure region.</span></span> <span data-ttu-id="8f057-160">A tartományvezérlők tegyen függő munkaterhelések hozzáférő központi VNet (hub).</span><span class="sxs-lookup"><span data-stu-id="8f057-160">Place your domain controllers in a central VNet (hub) that dependent workloads can access.</span></span>

### <a name="security"></a><span data-ttu-id="8f057-161">Biztonság</span><span class="sxs-lookup"><span data-stu-id="8f057-161">Security</span></span>

<span data-ttu-id="8f057-162">Amikor munkaterhelések a helyszíni környezetből Azure-ba, az ilyen terhelések némelyike el kell futhat virtuális gépeken.</span><span class="sxs-lookup"><span data-stu-id="8f057-162">As you move workloads from your on-premises environment to Azure, some of these workloads will require to be hosted in VMs.</span></span> <span data-ttu-id="8f057-163">Megfelelőségi okokból szükség lehet kényszeríteni azokat a munkaterheléseket áthaladó forgalom korlátozásait.</span><span class="sxs-lookup"><span data-stu-id="8f057-163">For compliance reasons, you may need to enforce restrictions on traffic traversing those workloads.</span></span> 

<span data-ttu-id="8f057-164">Az Azure szolgáltatásban az állomás különböző biztonsági és teljesítménynövelő szolgáltatások hálózati virtuális készülékek (NVAs) is használhatja.</span><span class="sxs-lookup"><span data-stu-id="8f057-164">You can use network virtual appliances (NVAs) in Azure to host different types of security and performance services.</span></span> <span data-ttu-id="8f057-165">Ha ismeri a helyszíni készülékek adott halmazát ma, javasoljuk, hogy az azonos virtuális készülékek használata Azure adott esetben.</span><span class="sxs-lookup"><span data-stu-id="8f057-165">If you are familiar with a given set of appliances on-premises today, it is recommended to use the same virtualized appliances in Azure, where applicable.</span></span>

> [!NOTE]
> <span data-ttu-id="8f057-166">Az üzembe helyezési parancsfájlok a referenciaarchitektúra Ubuntu virtuális gép IP-továbbítás engedélyezve van, hogy a hálózati virtuális készülék utánozzák használja.</span><span class="sxs-lookup"><span data-stu-id="8f057-166">The deployment scripts for this reference architecture use an Ubuntu VM with IP forwarding enabled to mimic a network virtual appliance.</span></span>

## <a name="considerations"></a><span data-ttu-id="8f057-167">Megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="8f057-167">Considerations</span></span>

### <a name="overcoming-vnet-peering-limits"></a><span data-ttu-id="8f057-168">A virtuális társhálózatok létesítési korlátjának kiküszöbölése</span><span class="sxs-lookup"><span data-stu-id="8f057-168">Overcoming VNet peering limits</span></span>

<span data-ttu-id="8f057-169">Mindenképpen vegye figyelembe a [virtuális hálózatonként létesíthető társviszonyok számának korlátozását][vnet-peering-limit] az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="8f057-169">Make sure you consider the [limitation on number of VNets peerings per VNet][vnet-peering-limit] in Azure.</span></span> <span data-ttu-id="8f057-170">Ha több küllőre van szükség, mint amit a korlátozás megenged, fontolja meg egy agy-küllő-agy-küllő topológia létrehozását, ahol a küllők első szintje agyként is működik.</span><span class="sxs-lookup"><span data-stu-id="8f057-170">If you decide you need more spokes than the limit will allow, consider creating a hub-spoke-hub-spoke topology, where the first level of spokes also act as hubs.</span></span> <span data-ttu-id="8f057-171">Az alábbi diagram ezt a megközelítést mutatja be.</span><span class="sxs-lookup"><span data-stu-id="8f057-171">The following diagram shows this approach.</span></span>

<span data-ttu-id="8f057-172">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="8f057-172">![[3]][3]</span></span>

<span data-ttu-id="8f057-173">Ezenkívül vegye figyelembe, mely szolgáltatások vannak megosztva az agyban. Ezzel biztosítható, hogy az agy több küllőre is skálázzon.</span><span class="sxs-lookup"><span data-stu-id="8f057-173">Also consider what services are shared in the hub, to ensure the hub scales for a larger number of spokes.</span></span> <span data-ttu-id="8f057-174">Például ha az agy tűzfalszolgáltatásokat is nyújt, vegye figyelembe a tűzfal megoldás sávszélességi korlátozásait több küllő hozzáadása esetén.</span><span class="sxs-lookup"><span data-stu-id="8f057-174">For instance, if your hub provides firewall services, consider the bandwidth limits of your firewall solution when adding multiple spokes.</span></span> <span data-ttu-id="8f057-175">Lehet, hogy érdemes néhány megosztott szolgáltatást az agyak második szintjére mozgatni.</span><span class="sxs-lookup"><span data-stu-id="8f057-175">You might want to move some of these shared services to a second level of hubs.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="8f057-176">A megoldás üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="8f057-176">Deploy the solution</span></span>

<span data-ttu-id="8f057-177">Ennek az architektúrának egy üzemelő példánya elérhető a [GitHubon][ref-arch-repo].</span><span class="sxs-lookup"><span data-stu-id="8f057-177">A deployment for this architecture is available on [GitHub][ref-arch-repo].</span></span> <span data-ttu-id="8f057-178">A központi telepítés az előfizetésében hoz létre a következő erőforrás-csoportok:</span><span class="sxs-lookup"><span data-stu-id="8f057-178">The deployment creates the following resource groups in your subscription:</span></span>

- <span data-ttu-id="8f057-179">hub hozzáadása rg</span><span class="sxs-lookup"><span data-stu-id="8f057-179">hub-adds-rg</span></span>
- <span data-ttu-id="8f057-180">hub-nva-rg</span><span class="sxs-lookup"><span data-stu-id="8f057-180">hub-nva-rg</span></span>
- <span data-ttu-id="8f057-181">hub-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="8f057-181">hub-vnet-rg</span></span>
- <span data-ttu-id="8f057-182">a helyi üzemeltetésű-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="8f057-182">onprem-vnet-rg</span></span>
- <span data-ttu-id="8f057-183">spoke1-vnet-rg</span><span class="sxs-lookup"><span data-stu-id="8f057-183">spoke1-vnet-rg</span></span>
- <span data-ttu-id="8f057-184">spoke2-nyílás-rg</span><span class="sxs-lookup"><span data-stu-id="8f057-184">spoke2-vent-rg</span></span>

<span data-ttu-id="8f057-185">A sablonfájlokat paraméter tekintse meg ezeket a neveket, így módosítja őket, ha a paraméter fájlok frissítése az egyeztetéshez.</span><span class="sxs-lookup"><span data-stu-id="8f057-185">The template parameter files refer to these names, so if you change them, update the parameter files to match.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="8f057-186">Előfeltételek</span><span class="sxs-lookup"><span data-stu-id="8f057-186">Prerequisites</span></span>

1. <span data-ttu-id="8f057-187">Klónozza, ágaztassa vagy a zip-fájl letöltése a [architektúrák hivatkozhat] [ ref-arch-repo] GitHub-tárházban.</span><span class="sxs-lookup"><span data-stu-id="8f057-187">Clone, fork, or download the zip file for the [reference architectures][ref-arch-repo] GitHub repository.</span></span>

2. <span data-ttu-id="8f057-188">Telepítés [Azure CLI 2.0][azure-cli-2].</span><span class="sxs-lookup"><span data-stu-id="8f057-188">Install [Azure CLI 2.0][azure-cli-2].</span></span>

3. <span data-ttu-id="8f057-189">Telepítse [az Azure építőelemei][azbb] npm-csomagot.</span><span class="sxs-lookup"><span data-stu-id="8f057-189">Install the [Azure building blocks][azbb] npm package.</span></span>

4. <span data-ttu-id="8f057-190">A parancssorból bash, vagy PowerShell kérdés, jelentkezzen be az Azure-fiókjával az alábbi parancs segítségével.</span><span class="sxs-lookup"><span data-stu-id="8f057-190">From a command prompt, bash prompt, or PowerShell prompt, login to your Azure account by using the command below.</span></span>

   ```bash
   az login
   ```

### <a name="deploy-the-simulated-on-premises-datacenter-using-azbb"></a><span data-ttu-id="8f057-191">A szimulált olyan helyszíni adatközpontban azbb használatával telepítése</span><span class="sxs-lookup"><span data-stu-id="8f057-191">Deploy the simulated on-premises datacenter using azbb</span></span>

<span data-ttu-id="8f057-192">Ez a lépés a szimulált helyszíni adatközpontját telepíti, egy Azure virtuális hálózatot.</span><span class="sxs-lookup"><span data-stu-id="8f057-192">This step deploys the simulated on-premises datacenter as an Azure VNet.</span></span>

1. <span data-ttu-id="8f057-193">Keresse meg a `hybrid-networking\shared-services-stack\` mappában található a GitHub-tárházban.</span><span class="sxs-lookup"><span data-stu-id="8f057-193">Navigate to the `hybrid-networking\shared-services-stack\` folder of the GitHub repository.</span></span>

2. <span data-ttu-id="8f057-194">Nyissa meg az `onprem.json` fájlt.</span><span class="sxs-lookup"><span data-stu-id="8f057-194">Open the `onprem.json` file.</span></span> 

3. <span data-ttu-id="8f057-195">Keresse meg az összes példányát `Password` és `adminPassword`.</span><span class="sxs-lookup"><span data-stu-id="8f057-195">Search for all instances of `Password` and `adminPassword`.</span></span> <span data-ttu-id="8f057-196">Adja meg a felhasználónevet és jelszót a paraméterek értékeit, és mentse a fájlt.</span><span class="sxs-lookup"><span data-stu-id="8f057-196">Enter values for the user name and password in the parameters and save the file.</span></span> 

4. <span data-ttu-id="8f057-197">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="8f057-197">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g onprem-vnet-rg -l <location> -p onprem.json --deploy
   ```
5. <span data-ttu-id="8f057-198">Várjon, amíg az üzembe helyezés befejeződik.</span><span class="sxs-lookup"><span data-stu-id="8f057-198">Wait for the deployment to finish.</span></span> <span data-ttu-id="8f057-199">A központi telepítés létrehoz egy virtuális hálózatot, Windows és a VPN-átjáró egy olyan virtuális géphez.</span><span class="sxs-lookup"><span data-stu-id="8f057-199">This deployment creates a virtual network, a virtual machine running Windows, and a VPN gateway.</span></span> <span data-ttu-id="8f057-200">Egy VPN-átjáró létrehozása 40 percnél is tovább tarthat.</span><span class="sxs-lookup"><span data-stu-id="8f057-200">The VPN gateway creation can take more than 40 minutes to complete.</span></span>

### <a name="deploy-the-hub-vnet"></a><span data-ttu-id="8f057-201">A virtuális hálózat központi telepítése</span><span class="sxs-lookup"><span data-stu-id="8f057-201">Deploy the hub VNet</span></span>

<span data-ttu-id="8f057-202">Ez a lépés a hub VNet telepíti, és csatlakozik a a szimulált helyszíni virtuális hálózat.</span><span class="sxs-lookup"><span data-stu-id="8f057-202">This step deploys the hub VNet and connects it to the simulated on-premises VNet.</span></span>

1. <span data-ttu-id="8f057-203">Nyissa meg az `hub-vnet.json` fájlt.</span><span class="sxs-lookup"><span data-stu-id="8f057-203">Open the `hub-vnet.json` file.</span></span> 

2. <span data-ttu-id="8f057-204">Keresse meg `adminPassword` , és írja be egy felhasználónevet és jelszót a paraméterek.</span><span class="sxs-lookup"><span data-stu-id="8f057-204">Search for `adminPassword` and enter a user name and password in the parameters.</span></span> 

3. <span data-ttu-id="8f057-205">Keresse meg az összes példányát `sharedKey` és egy megosztott kulcsot adjon meg egy értéket.</span><span class="sxs-lookup"><span data-stu-id="8f057-205">Search for all instances of `sharedKey` and enter a value for a shared key.</span></span> <span data-ttu-id="8f057-206">Mentse a fájlt.</span><span class="sxs-lookup"><span data-stu-id="8f057-206">Save the file.</span></span>

   ```bash
   "sharedKey": "abc123",
   ```

4. <span data-ttu-id="8f057-207">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="8f057-207">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet.json --deploy
   ```

5. <span data-ttu-id="8f057-208">Várjon, amíg az üzembe helyezés befejeződik.</span><span class="sxs-lookup"><span data-stu-id="8f057-208">Wait for the deployment to finish.</span></span> <span data-ttu-id="8f057-209">A központi telepítéshez létrehoz egy virtuális hálózatot, virtuális gép, VPN-átjáró és az átjáró, az előző szakaszban létrehozott kapcsolat.</span><span class="sxs-lookup"><span data-stu-id="8f057-209">This deployment creates a virtual network, a virtual machine, a VPN gateway, and a connection to the gateway created in the previous section.</span></span> <span data-ttu-id="8f057-210">A VPN-átjáró legfeljebb 40 percet is igénybe vehet.</span><span class="sxs-lookup"><span data-stu-id="8f057-210">The VPN gateway can take more than 40 minutes to complete.</span></span>

### <a name="deploy-ad-ds-in-azure"></a><span data-ttu-id="8f057-211">Az Azure Active Directory tartományi szolgáltatások telepítése</span><span class="sxs-lookup"><span data-stu-id="8f057-211">Deploy AD DS in Azure</span></span>

<span data-ttu-id="8f057-212">Ez a lépés telepíti az Azure Active Directory tartományi szolgáltatások tartományvezérlő.</span><span class="sxs-lookup"><span data-stu-id="8f057-212">This step deploys AD DS domain controllers in Azure.</span></span>

1. <span data-ttu-id="8f057-213">Nyissa meg az `hub-adds.json` fájlt.</span><span class="sxs-lookup"><span data-stu-id="8f057-213">Open the `hub-adds.json` file.</span></span>

2. <span data-ttu-id="8f057-214">Keresse meg az összes példányát `Password` és `adminPassword`.</span><span class="sxs-lookup"><span data-stu-id="8f057-214">Search for all instances of `Password` and `adminPassword`.</span></span> <span data-ttu-id="8f057-215">Adja meg a felhasználónevet és jelszót a paraméterek értékeit, és mentse a fájlt.</span><span class="sxs-lookup"><span data-stu-id="8f057-215">Enter values for the user name and password in the parameters and save the file.</span></span> 

3. <span data-ttu-id="8f057-216">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="8f057-216">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-adds-rg -l <location> -p hub-adds.json --deploy
   ```
  
<span data-ttu-id="8f057-217">A központi telepítési lépés eltarthat néhány percig, mert ez a két virtuális gép csatlakozik a tartományhoz, a szimulált helyszíni adatközpontban futtatott, és telepíti az Active Directory tartományi szolgáltatások rajtuk.</span><span class="sxs-lookup"><span data-stu-id="8f057-217">This deployment step may take several minutes, because it joins the two VMs to the domain hosted in the simulated on-premises datacenter, and installs AD DS on them.</span></span>

### <a name="deploy-the-spoke-vnets"></a><span data-ttu-id="8f057-218">A Vnetek küllős telepítése</span><span class="sxs-lookup"><span data-stu-id="8f057-218">Deploy the spoke VNets</span></span>

<span data-ttu-id="8f057-219">Ez a lépés telepíti a Vnetek küllős.</span><span class="sxs-lookup"><span data-stu-id="8f057-219">This step deploys the spoke VNets.</span></span>

1. <span data-ttu-id="8f057-220">Nyissa meg az `spoke1.json` fájlt.</span><span class="sxs-lookup"><span data-stu-id="8f057-220">Open the `spoke1.json` file.</span></span>

2. <span data-ttu-id="8f057-221">Keresse meg `adminPassword` , és írja be egy felhasználónevet és jelszót a paraméterek.</span><span class="sxs-lookup"><span data-stu-id="8f057-221">Search for `adminPassword` and enter a user name and password in the parameters.</span></span> 

3. <span data-ttu-id="8f057-222">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="8f057-222">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke1-vnet-rg -l <location> -p spoke1.json --deploy
   ```
  
4. <span data-ttu-id="8f057-223">Ismételje meg a 1. és 2 fájl `spoke2.json`.</span><span class="sxs-lookup"><span data-stu-id="8f057-223">Repeat steps 1 and 2 for the file `spoke2.json`.</span></span>

5. <span data-ttu-id="8f057-224">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="8f057-224">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g spoke2-vnet-rg -l <location> -p spoke2.json --deploy
   ```

### <a name="peer-the-hub-vnet-to-the-spoke-vnets"></a><span data-ttu-id="8f057-225">A virtuális hálózat hubot a Vnetek küllős partnert</span><span class="sxs-lookup"><span data-stu-id="8f057-225">Peer the hub VNet to the spoke VNets</span></span>

<span data-ttu-id="8f057-226">Társviszony-létesítési kapcsolatot létesíthet a virtuális hálózat központi és a Vnetek küllős, futtassa a következő parancsot:</span><span class="sxs-lookup"><span data-stu-id="8f057-226">To create a peering connection from the hub VNet to the spoke VNets, run the following command:</span></span>

```bash
azbb -s <subscription_id> -g hub-vnet-rg -l <location> -p hub-vnet-peering.json --deploy
```

### <a name="deploy-the-nva"></a><span data-ttu-id="8f057-227">Az NVA telepítése</span><span class="sxs-lookup"><span data-stu-id="8f057-227">Deploy the NVA</span></span>

<span data-ttu-id="8f057-228">Ez a lépés telepíti az NVA a `dmz` alhálózati.</span><span class="sxs-lookup"><span data-stu-id="8f057-228">This step deploys an NVA in the `dmz` subnet.</span></span>

1. <span data-ttu-id="8f057-229">Nyissa meg az `hub-nva.json` fájlt.</span><span class="sxs-lookup"><span data-stu-id="8f057-229">Open the `hub-nva.json` file.</span></span>

2. <span data-ttu-id="8f057-230">Keresse meg `adminPassword` , és írja be egy felhasználónevet és jelszót a paraméterek.</span><span class="sxs-lookup"><span data-stu-id="8f057-230">Search for `adminPassword` and enter a user name and password in the parameters.</span></span> 

3. <span data-ttu-id="8f057-231">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="8f057-231">Run the following command:</span></span>

   ```bash
   azbb -s <subscription_id> -g hub-nva-rg -l <location> -p hub-nva.json --deploy
   ```

### <a name="test-connectivity"></a><span data-ttu-id="8f057-232">Kapcsolat tesztelése</span><span class="sxs-lookup"><span data-stu-id="8f057-232">Test connectivity</span></span> 

<span data-ttu-id="8f057-233">A szimulált a helyszíni környezetből szeretne az elosztóhoz VNet kapcsolat tesztelése.</span><span class="sxs-lookup"><span data-stu-id="8f057-233">Test conectivity from the simulated on-premises environment to the hub VNet.</span></span>

1. <span data-ttu-id="8f057-234">Az Azure portál segítségével nevű virtuális gép található `jb-vm1` a a `onprem-jb-rg` erőforráscsoportot.</span><span class="sxs-lookup"><span data-stu-id="8f057-234">Use the Azure portal to find the VM named `jb-vm1` in the `onprem-jb-rg` resource group.</span></span>

2. <span data-ttu-id="8f057-235">Kattintson a `Connect` számára a virtuális gép távoli asztali munkamenetet nyit meg.</span><span class="sxs-lookup"><span data-stu-id="8f057-235">Click `Connect` to open a remote desktop session to the VM.</span></span> <span data-ttu-id="8f057-236">A megadott jelszó használata a `onprem.json` paraméterfájl.</span><span class="sxs-lookup"><span data-stu-id="8f057-236">Use the password that you specified in the `onprem.json` parameter file.</span></span>

3. <span data-ttu-id="8f057-237">Nyissa meg a PowerShell-konzolban a virtuális Gépet, és használja a `Test-NetConnection` parancsmag futtatásával ellenőrizze, hogy tud-e csatlakozni a virtuális gép jumpbox VNet központban.</span><span class="sxs-lookup"><span data-stu-id="8f057-237">Open a PowerShell console in the VM, and use the `Test-NetConnection` cmdlet to verify that you can connect to the jumpbox VM in the hub VNet.</span></span>

   ```powershell
   Test-NetConnection 10.0.0.68 -CommonTCPPort RDP
   ```
<span data-ttu-id="8f057-238">A kimenet az alábbihoz hasonló kell kinéznie:</span><span class="sxs-lookup"><span data-stu-id="8f057-238">The output should look similar to the following:</span></span>

```powershell
ComputerName     : 10.0.0.68
RemoteAddress    : 10.0.0.68
RemotePort       : 3389
InterfaceAlias   : Ethernet 2
SourceAddress    : 192.168.1.000
TcpTestSucceeded : True
```

> [!NOTE]
> <span data-ttu-id="8f057-239">Alapértelmezés szerint a Windows Server virtuális gépen nem teszik lehetővé az ICMP-válaszokat az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="8f057-239">By default, Windows Server VMs do not allow ICMP responses in Azure.</span></span> <span data-ttu-id="8f057-240">Ha a használni kívánt `ping` kapcsolat tesztelése, az egyes virtuális gépek ICMP-forgalmat a Windows speciális tűzfalon engedélyezni kell.</span><span class="sxs-lookup"><span data-stu-id="8f057-240">If you want to use `ping` to test connectivity, you need to enable ICMP traffic in the Windows Advanced Firewall for each VM.</span></span>

<span data-ttu-id="8f057-241">Ismételje meg a sames a Vnetek küllős kapcsolat teszteléséhez:</span><span class="sxs-lookup"><span data-stu-id="8f057-241">Repeat the sames steps to test connectivity to the spoke VNets:</span></span>

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
