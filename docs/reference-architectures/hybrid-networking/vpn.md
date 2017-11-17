---
title: "Egy helyszíni hálózat csatlakozik az Azure VPN-nel"
description: "Hogyan egy biztonságos pont-pont hálózati architektúra, amely egy Azure virtuális hálózatra és egy a helyszíni hálózathoz egy VPN-nel csatlakoztatott kiterjedő végrehajtásához."
author: RohitSharma-pnp
ms.date: 11/28/2016
pnp.series.title: Connect an on-premises network to Azure
pnp.series.next: expressroute
pnp.series.prev: ./index
cardTitle: VPN
ms.openlocfilehash: 66b2605c551148fadcdee6808c4e85940089f1e5
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="connect-an-on-premises-network-to-azure-using-a-vpn-gateway"></a><span data-ttu-id="16d51-103">Egy helyszíni hálózat csatlakozik az Azure VPN-átjáró használatával</span><span class="sxs-lookup"><span data-stu-id="16d51-103">Connect an on-premises network to Azure using a VPN gateway</span></span>

<span data-ttu-id="16d51-104">A referencia-architektúrában bemutatja, hogyan egy a helyszíni hálózat kiterjesztése az Azure-ba, pont-pont virtuális magánhálózat (VPN) segítségével.</span><span class="sxs-lookup"><span data-stu-id="16d51-104">This reference architecture shows how to extend an on-premises network to Azure, using a site-to-site virtual private network (VPN).</span></span> <span data-ttu-id="16d51-105">A forgalom között a helyszíni hálózat és az Azure Virtual Network (VNet), az IPSec VPN-alagúton keresztül.</span><span class="sxs-lookup"><span data-stu-id="16d51-105">Traffic flows between the on-premises network and an Azure Virtual Network (VNet) through an IPSec VPN tunnel.</span></span> [<span data-ttu-id="16d51-106">**Ez a megoldás üzembe helyezéséhez**.</span><span class="sxs-lookup"><span data-stu-id="16d51-106">**Deploy this solution**.</span></span>](#deploy-the-solution)

<span data-ttu-id="16d51-107">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="16d51-107">![[0]][0]</span></span>

<span data-ttu-id="16d51-108">*Töltse le a [Visio fájl] [ visio-download] ezen architektúra.*</span><span class="sxs-lookup"><span data-stu-id="16d51-108">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="16d51-109">Architektúra</span><span class="sxs-lookup"><span data-stu-id="16d51-109">Architecture</span></span> 

<span data-ttu-id="16d51-110">Az architektúra a következő összetevőkből áll.</span><span class="sxs-lookup"><span data-stu-id="16d51-110">The architecture consists of the following components.</span></span>

* <span data-ttu-id="16d51-111">**A helyszíni hálózat**.</span><span class="sxs-lookup"><span data-stu-id="16d51-111">**On-premises network**.</span></span> <span data-ttu-id="16d51-112">Helyi magánhálózat fut egy szervezeten belül.</span><span class="sxs-lookup"><span data-stu-id="16d51-112">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="16d51-113">**VPN-készülék**.</span><span class="sxs-lookup"><span data-stu-id="16d51-113">**VPN appliance**.</span></span> <span data-ttu-id="16d51-114">Egy eszköz vagy a helyszíni hálózathoz külső kapcsolatot biztosító szolgáltatás.</span><span class="sxs-lookup"><span data-stu-id="16d51-114">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="16d51-115">Lehet, hogy a VPN-készülék olyan hardvereszköz, vagy egy szoftveres megoldás, mint például az Útválasztás és távelérés szolgáltatás (RRAS) a Windows Server 2012-ben.</span><span class="sxs-lookup"><span data-stu-id="16d51-115">The VPN appliance may be a hardware device, or it can be a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="16d51-116">Támogatott VPN-készülék és azok tudjanak csatlakozni az Azure VPN gatewayhez konfigurálásáról listájáért lásd: a kijelölt eszköz-cikkben található utasításokat [kapcsolatos VPN-eszközök a webhelyek közötti VPN átjáró kapcsolatok] [ vpn-appliance].</span><span class="sxs-lookup"><span data-stu-id="16d51-116">For a list of supported VPN appliances and information on configuring them to connect to an Azure VPN gateway, see the instructions for the selected device in the article [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="16d51-117">**Virtuális hálózathoz (VNet)**.</span><span class="sxs-lookup"><span data-stu-id="16d51-117">**Virtual network (VNet)**.</span></span> <span data-ttu-id="16d51-118">A felhőalapú alkalmazások és az Azure VPN gateway a összetevőket található azonos [VNet][azure-virtual-network].</span><span class="sxs-lookup"><span data-stu-id="16d51-118">The cloud application and the components for the Azure VPN gateway reside in the same [VNet][azure-virtual-network].</span></span>

* <span data-ttu-id="16d51-119">**Az Azure VPN gateway**.</span><span class="sxs-lookup"><span data-stu-id="16d51-119">**Azure VPN gateway**.</span></span> <span data-ttu-id="16d51-120">A [VPN-átjáró] [ azure-vpn-gateway] szolgáltatás lehetővé teszi, hogy a virtuális hálózat csatlakozni a helyszíni hálózathoz egy VPN-készülék keresztül.</span><span class="sxs-lookup"><span data-stu-id="16d51-120">The [VPN gateway][azure-vpn-gateway] service enables you to connect the VNet to the on-premises network through a VPN appliance.</span></span> <span data-ttu-id="16d51-121">További információkért lásd: [egy a helyszíni hálózathoz csatlakozni a Microsoft Azure virtuális hálózat][connect-to-an-Azure-vnet].</span><span class="sxs-lookup"><span data-stu-id="16d51-121">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span> <span data-ttu-id="16d51-122">A VPN-átjáró a következő elemeket tartalmazza:</span><span class="sxs-lookup"><span data-stu-id="16d51-122">The VPN gateway includes the following elements:</span></span>
  
  * <span data-ttu-id="16d51-123">**Virtuális hálózati átjáró**.</span><span class="sxs-lookup"><span data-stu-id="16d51-123">**Virtual network gateway**.</span></span> <span data-ttu-id="16d51-124">Egy virtuális VPN-készülék biztosít a virtuális hálózat erőforrás.</span><span class="sxs-lookup"><span data-stu-id="16d51-124">A resource that provides a virtual VPN appliance for the VNet.</span></span> <span data-ttu-id="16d51-125">Ez felelős adatforgalom a helyi hálózatról a virtuális hálózatba.</span><span class="sxs-lookup"><span data-stu-id="16d51-125">It is responsible for routing traffic from the on-premises network to the VNet.</span></span>
  * <span data-ttu-id="16d51-126">**Helyi hálózati átjáró**.</span><span class="sxs-lookup"><span data-stu-id="16d51-126">**Local network gateway**.</span></span> <span data-ttu-id="16d51-127">A helyszíni VPN-készülék absztrakciós.</span><span class="sxs-lookup"><span data-stu-id="16d51-127">An abstraction of the on-premises VPN appliance.</span></span> <span data-ttu-id="16d51-128">A felhő alkalmazásról, a helyszíni hálózat a hálózati forgalom az átjáró keresztül történik.</span><span class="sxs-lookup"><span data-stu-id="16d51-128">Network traffic from the cloud application to the on-premises network is routed through this gateway.</span></span>
  * <span data-ttu-id="16d51-129">**Kapcsolat**.</span><span class="sxs-lookup"><span data-stu-id="16d51-129">**Connection**.</span></span> <span data-ttu-id="16d51-130">A kapcsolat tulajdonságai adja meg a kapcsolat típusát (IPSec) és a kulcsot, a helyszíni VPN-készülék megosztott forgalom titkosításához.</span><span class="sxs-lookup"><span data-stu-id="16d51-130">The connection has properties that specify the connection type (IPSec) and the key shared with the on-premises VPN appliance to encrypt traffic.</span></span>
  * <span data-ttu-id="16d51-131">**Átjáró alhálózati**.</span><span class="sxs-lookup"><span data-stu-id="16d51-131">**Gateway subnet**.</span></span> <span data-ttu-id="16d51-132">A virtuális hálózati átjáró saját alhálózatba, amely különböző vonatkoznak, az ajánlások szakaszban leírt használatban van.</span><span class="sxs-lookup"><span data-stu-id="16d51-132">The virtual network gateway is held in its own subnet, which is subject to various requirements, described in the Recommendations section below.</span></span>

* <span data-ttu-id="16d51-133">**A felhő alkalmazás**.</span><span class="sxs-lookup"><span data-stu-id="16d51-133">**Cloud application**.</span></span> <span data-ttu-id="16d51-134">Az alkalmazást az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="16d51-134">The application hosted in Azure.</span></span> <span data-ttu-id="16d51-135">A többrétegű konfigurációk – Azure load Balancer terheléselosztók keresztül kapcsolódik, több alhálózattal rendelkező tartalmazhat.</span><span class="sxs-lookup"><span data-stu-id="16d51-135">It might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="16d51-136">Az alkalmazás-infrastruktúrával kapcsolatos további információkért lásd: [futó Windows virtuális gép munkaterhelések] [ windows-vm-ra] és [futó Linux virtuális gép munkaterhelések] [ linux-vm-ra].</span><span class="sxs-lookup"><span data-stu-id="16d51-136">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

* <span data-ttu-id="16d51-137">**Belső terheléselosztó**.</span><span class="sxs-lookup"><span data-stu-id="16d51-137">**Internal load balancer**.</span></span> <span data-ttu-id="16d51-138">Hálózati forgalmat a VPN-átjárót a felhőalapú alkalmazásokhoz, a belső terheléselosztók keresztül van átirányítva.</span><span class="sxs-lookup"><span data-stu-id="16d51-138">Network traffic from the VPN gateway is routed to the cloud application through an internal load balancer.</span></span> <span data-ttu-id="16d51-139">A terheléselosztó előtér-alkalmazás az alhálózaton található.</span><span class="sxs-lookup"><span data-stu-id="16d51-139">The load balancer is located in the front-end subnet of the application.</span></span>

## <a name="recommendations"></a><span data-ttu-id="16d51-140">Javaslatok</span><span class="sxs-lookup"><span data-stu-id="16d51-140">Recommendations</span></span>

<span data-ttu-id="16d51-141">Az alábbi javaslatok a legtöbb forgatókönyvre vonatkoznak.</span><span class="sxs-lookup"><span data-stu-id="16d51-141">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="16d51-142">Kövesse ezeket a javaslatokat, ha nincsenek ezeket felülíró követelményei.</span><span class="sxs-lookup"><span data-stu-id="16d51-142">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="vnet-and-gateway-subnet"></a><span data-ttu-id="16d51-143">Virtuális hálózat és az átjáró-alhálózatot</span><span class="sxs-lookup"><span data-stu-id="16d51-143">VNet and gateway subnet</span></span>

<span data-ttu-id="16d51-144">Hozzon létre egy Azure virtuális hálózatot egy cím elegendő hely számára összes szükséges erőforrást.</span><span class="sxs-lookup"><span data-stu-id="16d51-144">Create an Azure VNet with an address space large enough for all of your required resources.</span></span> <span data-ttu-id="16d51-145">Győződjön meg arról, hogy a virtuális hálózat címtér rendelkezik-e elegendő hely marad, ha további virtuális gépek valószínűleg a jövőben van szükség.</span><span class="sxs-lookup"><span data-stu-id="16d51-145">Ensure that the VNet address space has sufficient room for growth if additional VMs are likely to be needed in the future.</span></span> <span data-ttu-id="16d51-146">A virtuális hálózat címtartománya nem fedhetik a helyszíni hálózattal.</span><span class="sxs-lookup"><span data-stu-id="16d51-146">The address space of the VNet must not overlap with the on-premises network.</span></span> <span data-ttu-id="16d51-147">Például a fenti ábra a cím terület 10.20.0.0/16 használ a virtuális hálózat.</span><span class="sxs-lookup"><span data-stu-id="16d51-147">For example, the diagram above uses the address space 10.20.0.0/16 for the VNet.</span></span>

<span data-ttu-id="16d51-148">Hozzon létre egy nevű alhálózatot *GatewaySubnet*, a /27 címtartományt.</span><span class="sxs-lookup"><span data-stu-id="16d51-148">Create a subnet named *GatewaySubnet*, with an address range of /27.</span></span> <span data-ttu-id="16d51-149">Ez az alhálózat által a virtuális hálózati átjáró szükséges.</span><span class="sxs-lookup"><span data-stu-id="16d51-149">This subnet is required by the virtual network gateway.</span></span> <span data-ttu-id="16d51-150">Lefoglalásával megelőzése érdekében az alhálózathoz 32 cím segítségével átjáró méretkorlátai a jövőben elérése.</span><span class="sxs-lookup"><span data-stu-id="16d51-150">Allocating 32 addresses to this subnet will help to prevent reaching gateway size limitations in the future.</span></span> <span data-ttu-id="16d51-151">Emellett helyez az alhálózat a címterület közepén.</span><span class="sxs-lookup"><span data-stu-id="16d51-151">Also, avoid placing this subnet in the middle of the address space.</span></span> <span data-ttu-id="16d51-152">Bevált gyakorlat is, hogy az átjáró alhálózatának a címtartomány a virtuális hálózat címterület felső végén.</span><span class="sxs-lookup"><span data-stu-id="16d51-152">A good practice is to set the address space for the gateway subnet at the upper end of the VNet address space.</span></span> <span data-ttu-id="16d51-153">Az ábrán is látható példa 10.20.255.224/27.</span><span class="sxs-lookup"><span data-stu-id="16d51-153">The example shown in the diagram uses 10.20.255.224/27.</span></span>  <span data-ttu-id="16d51-154">Íme egy gyors eljárás kiszámításához a [CIDR]:</span><span class="sxs-lookup"><span data-stu-id="16d51-154">Here is a quick procedure to calculate the [CIDR]:</span></span>

1. <span data-ttu-id="16d51-155">Állítsa be a változó bitek a címterületen belülre a vnet, akár a bits alatt az átjáró-alhálózat használja, akkor a többi bit értéke 0, 1.</span><span class="sxs-lookup"><span data-stu-id="16d51-155">Set the variable bits in the address space of the VNet to 1, up to the bits being used by the gateway subnet, then set the remaining bits to 0.</span></span>
2. <span data-ttu-id="16d51-156">Az eredményül kapott bits átalakítása decimális, és azt, az átjáró alhálózatának méretét a előtag hossza értéke egy címtartománnyal express.</span><span class="sxs-lookup"><span data-stu-id="16d51-156">Convert the resulting bits to decimal and express it as an address space with the prefix length set to the size of the gateway subnet.</span></span>

<span data-ttu-id="16d51-157">Például egy IP-címtartománnyal 10.20.0.0/16 a vnet, alkalmazza a fenti #1. lépés lesz 10.20.0b11111111.0b11100000.</span><span class="sxs-lookup"><span data-stu-id="16d51-157">For example, for a VNet with an IP address range of 10.20.0.0/16, applying step #1 above becomes 10.20.0b11111111.0b11100000.</span></span>  <span data-ttu-id="16d51-158">Konvertálása, decimális és kifejező azt, egy 10.20.255.224/27 adja eredményül.</span><span class="sxs-lookup"><span data-stu-id="16d51-158">Converting that to decimal and expressing it as an address space yields 10.20.255.224/27.</span></span> 

> [!WARNING]
> <span data-ttu-id="16d51-159">Nem kell telepítenie a virtuális gépek az átjáró alhálózatához.</span><span class="sxs-lookup"><span data-stu-id="16d51-159">Do not deploy any VMs to the gateway subnet.</span></span> <span data-ttu-id="16d51-160">Is nem rendel egy NSG-t az alhálózaton, mint okozhat az átjáró működését.</span><span class="sxs-lookup"><span data-stu-id="16d51-160">Also, do not assign an NSG to this subnet, as it will cause the gateway to stop functioning.</span></span>
> 
> 

### <a name="virtual-network-gateway"></a><span data-ttu-id="16d51-161">Virtuális hálózati átjáró</span><span class="sxs-lookup"><span data-stu-id="16d51-161">Virtual network gateway</span></span>

<span data-ttu-id="16d51-162">A virtuális hálózati átjáró nyilvános IP-címnek lefoglalni.</span><span class="sxs-lookup"><span data-stu-id="16d51-162">Allocate a public IP address for the virtual network gateway.</span></span>

<span data-ttu-id="16d51-163">Hozza létre a virtuális hálózati átjárót, az átjáró-alhálózatot, és rendelje hozzá az újonnan lefoglalt nyilvános IP-cím.</span><span class="sxs-lookup"><span data-stu-id="16d51-163">Create the virtual network gateway in the gateway subnet and assign it the newly allocated public IP address.</span></span> <span data-ttu-id="16d51-164">Nyomja meg az átjáró típusa, amely a leginkább megfelel a követelményeknek, és a VPN-készülék szerint engedélyezve van, amely:</span><span class="sxs-lookup"><span data-stu-id="16d51-164">Use the gateway type that most closely matches your requirements and that is enabled by your VPN appliance:</span></span>

- <span data-ttu-id="16d51-165">Hozzon létre egy [átjáró házirendalapú] [ policy-based-routing] Ha kell szigorúan szabályozzák a rendszer kérést átirányítja milyen címelőtagokat házirend feltételek alapján.</span><span class="sxs-lookup"><span data-stu-id="16d51-165">Create a [policy-based gateway][policy-based-routing] if you need to closely control how requests are routed based on policy criteria such as address prefixes.</span></span> <span data-ttu-id="16d51-166">Csoportházirend-alapú átjárók használható statikus útválasztás, és csak a pont-pont kapcsolatok működnek.</span><span class="sxs-lookup"><span data-stu-id="16d51-166">Policy-based gateways use static routing, and only work with site-to-site connections.</span></span>

- <span data-ttu-id="16d51-167">Hozzon létre egy [útválasztó-alapú átjáró] [ route-based-routing] csatlakozhat a helyszíni hálózat, az RRAS használata, ha támogatja a többhelyes vagy kereszt-terület kapcsolatokat, vagy valósítja meg a VNet – VNet kapcsolatokhoz (beleértve a továbbítja, amely haladnak át több Vnetekről).</span><span class="sxs-lookup"><span data-stu-id="16d51-167">Create a [route-based gateway][route-based-routing] if you connect to the on-premises network using RRAS, support multi-site or cross-region connections, or implement VNet-to-VNet connections (including routes that traverse multiple VNets).</span></span> <span data-ttu-id="16d51-168">Útválasztó-alapú átjárók használja, a hálózatok közötti közvetlen forgalom dinamikus útválasztást.</span><span class="sxs-lookup"><span data-stu-id="16d51-168">Route-based gateways use dynamic routing to direct traffic between networks.</span></span> <span data-ttu-id="16d51-169">Azok tűri hibák jobb, mint a statikus útvonal, hálózati elérési útról, mert azok próbálkozhat alternatív útvonalak.</span><span class="sxs-lookup"><span data-stu-id="16d51-169">They can tolerate failures in the network path better than static routes because they can try alternative routes.</span></span> <span data-ttu-id="16d51-170">Útválasztó-alapú átjárók is csökkentheti a felügyeleti mert útvonalak előfordulhat, hogy nem kell manuálisan frissül, amikor megváltozik hálózati címe.</span><span class="sxs-lookup"><span data-stu-id="16d51-170">Route-based gateways can also reduce the management overhead because routes might not need to be updated manually when network addresses change.</span></span>

<span data-ttu-id="16d51-171">Támogatott VPN-készülékek listájáért lásd: [kapcsolatos VPN-eszközök a webhelyek közötti VPN átjáró kapcsolatok][vpn-appliances].</span><span class="sxs-lookup"><span data-stu-id="16d51-171">For a list of supported VPN appliances, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliances].</span></span>

> [!NOTE]
> <span data-ttu-id="16d51-172">Az átjáró létrehozása után nem módosítható nélkül törli, majd újra létrehozza az átjáró átjáró típusok között.</span><span class="sxs-lookup"><span data-stu-id="16d51-172">After the gateway has been created, you cannot change between gateway types without deleting and re-creating the gateway.</span></span>
> 
> 

<span data-ttu-id="16d51-173">Válassza ki az Azure VPN gateway SKU, amely a leginkább megfelel a átviteli követelményeknek.</span><span class="sxs-lookup"><span data-stu-id="16d51-173">Select the Azure VPN gateway SKU that most closely matches your throughput requirements.</span></span> <span data-ttu-id="16d51-174">Az Azure VPN-átjáró a következő táblázatban látható három termékváltozatok érhető el.</span><span class="sxs-lookup"><span data-stu-id="16d51-174">Azure VPN gateway is available in three SKUs shown in the following table.</span></span> 

| <span data-ttu-id="16d51-175">SKU</span><span class="sxs-lookup"><span data-stu-id="16d51-175">SKU</span></span> | <span data-ttu-id="16d51-176">VPN átviteli sebesség</span><span class="sxs-lookup"><span data-stu-id="16d51-176">VPN Throughput</span></span> | <span data-ttu-id="16d51-177">Maximális IPSec-alagutak</span><span class="sxs-lookup"><span data-stu-id="16d51-177">Max IPSec Tunnels</span></span> |
| --- | --- | --- |
| <span data-ttu-id="16d51-178">Basic</span><span class="sxs-lookup"><span data-stu-id="16d51-178">Basic</span></span> |<span data-ttu-id="16d51-179">100 Mbps</span><span class="sxs-lookup"><span data-stu-id="16d51-179">100 Mbps</span></span> |<span data-ttu-id="16d51-180">10</span><span class="sxs-lookup"><span data-stu-id="16d51-180">10</span></span> |
| <span data-ttu-id="16d51-181">Standard</span><span class="sxs-lookup"><span data-stu-id="16d51-181">Standard</span></span> |<span data-ttu-id="16d51-182">100 Mbps</span><span class="sxs-lookup"><span data-stu-id="16d51-182">100 Mbps</span></span> |<span data-ttu-id="16d51-183">10</span><span class="sxs-lookup"><span data-stu-id="16d51-183">10</span></span> |
| <span data-ttu-id="16d51-184">Kiemelkedő teljesítmény</span><span class="sxs-lookup"><span data-stu-id="16d51-184">High Performance</span></span> |<span data-ttu-id="16d51-185">200 Mbps</span><span class="sxs-lookup"><span data-stu-id="16d51-185">200 Mbps</span></span> |<span data-ttu-id="16d51-186">30</span><span class="sxs-lookup"><span data-stu-id="16d51-186">30</span></span> |

> [!NOTE]
> <span data-ttu-id="16d51-187">Az alapvető termékváltozata nem kompatibilis a Azure ExpressRoute.</span><span class="sxs-lookup"><span data-stu-id="16d51-187">The Basic SKU is not compatible with Azure ExpressRoute.</span></span> <span data-ttu-id="16d51-188">Is [módosítása a Termékváltozat] [ changing-SKUs] az átjáró létrehozása után.</span><span class="sxs-lookup"><span data-stu-id="16d51-188">You can [change the SKU][changing-SKUs] after the gateway has been created.</span></span>
> 
> 

<span data-ttu-id="16d51-189">Az, hogy mennyi ideig az átjáró kiépítését alapján, és elérhető van szó.</span><span class="sxs-lookup"><span data-stu-id="16d51-189">You are charged based on the amount of time that the gateway is provisioned and available.</span></span> <span data-ttu-id="16d51-190">Lásd: [VPN-átjáró árképzési][azure-gateway-charges].</span><span class="sxs-lookup"><span data-stu-id="16d51-190">See [VPN Gateway Pricing][azure-gateway-charges].</span></span>

<span data-ttu-id="16d51-191">Hozzon létre az átjáró alhálózatának vonatkozó útválasztási szabályokat, hogy közvetlen bejövő alkalmazás forgalom a belső terheléselosztó ahelyett, hogy közvetlenül az alkalmazás virtuális gépek átadása kérésének engedélyezése az átjárót.</span><span class="sxs-lookup"><span data-stu-id="16d51-191">Create routing rules for the gateway subnet that direct incoming application traffic from the gateway to the internal load balancer, rather than allowing requests to pass directly to the application VMs.</span></span>

### <a name="on-premises-network-connection"></a><span data-ttu-id="16d51-192">A helyszíni hálózati kapcsolat</span><span class="sxs-lookup"><span data-stu-id="16d51-192">On-premises network connection</span></span>

<span data-ttu-id="16d51-193">A helyi hálózati átjáró létrehozása.</span><span class="sxs-lookup"><span data-stu-id="16d51-193">Create a local network gateway.</span></span> <span data-ttu-id="16d51-194">Adja meg a helyszíni VPN-készülék, és a helyszíni hálózat a címtér nyilvános IP-címét.</span><span class="sxs-lookup"><span data-stu-id="16d51-194">Specify the public IP address of the on-premises VPN appliance, and the address space of the on-premises network.</span></span> <span data-ttu-id="16d51-195">Vegye figyelembe, hogy a helyszíni VPN-készülék rendelkeznie kell egy nyilvános IP-címet, amely hozzáfér a helyi hálózati átjáró az Azure VPN Gateway.</span><span class="sxs-lookup"><span data-stu-id="16d51-195">Note that the on-premises VPN appliance must have a public IP address that can be accessed by the local network gateway in Azure VPN Gateway.</span></span> <span data-ttu-id="16d51-196">A VPN-eszköz mögött egy hálózati címfordítási (NAT) eszköz nem található.</span><span class="sxs-lookup"><span data-stu-id="16d51-196">The VPN device cannot be located behind a network address translation (NAT) device.</span></span>

<span data-ttu-id="16d51-197">A virtuális hálózati átjáró és a helyi hálózati átjáró webhelyek kapcsolatot létrehozni.</span><span class="sxs-lookup"><span data-stu-id="16d51-197">Create a site-to-site connection for the virtual network gateway and the local network gateway.</span></span> <span data-ttu-id="16d51-198">Válassza ki a pont-pont (IPSec) kapcsolattípus, és adja meg a megosztott kulcsot.</span><span class="sxs-lookup"><span data-stu-id="16d51-198">Select the site-to-site (IPSec) connection type, and specify the shared key.</span></span> <span data-ttu-id="16d51-199">Az Azure VPN gateway-webhelyek titkosítás az IPSec protokoll használatával az előmegosztott kulcsos hitelesítés alapul.</span><span class="sxs-lookup"><span data-stu-id="16d51-199">Site-to-site encryption with the Azure VPN gateway is based on the IPSec protocol, using preshared keys for authentication.</span></span> <span data-ttu-id="16d51-200">Adja meg a kulcsot az Azure VPN gateway létrehozásakor.</span><span class="sxs-lookup"><span data-stu-id="16d51-200">You specify the key when you create the Azure VPN gateway.</span></span> <span data-ttu-id="16d51-201">A VPN-készülék helyileg futó ugyanazzal a kulccsal kell konfigurálnia.</span><span class="sxs-lookup"><span data-stu-id="16d51-201">You must configure the VPN appliance running on-premises with the same key.</span></span> <span data-ttu-id="16d51-202">Más hitelesítési mechanizmusok jelenleg nem támogatottak.</span><span class="sxs-lookup"><span data-stu-id="16d51-202">Other authentication mechanisms are not currently supported.</span></span>

<span data-ttu-id="16d51-203">Győződjön meg arról, hogy a helyi útválasztási infrastruktúra úgy van beállítva, a VPN-eszköz az Azure VNet címek szánt kérelmek továbbítása.</span><span class="sxs-lookup"><span data-stu-id="16d51-203">Ensure that the on-premises routing infrastructure is configured to forward requests intended for addresses in the Azure VNet to the VPN device.</span></span>

<span data-ttu-id="16d51-204">Nyissa meg a felhő alkalmazáshoz, a helyszíni hálózat szükséges portja.</span><span class="sxs-lookup"><span data-stu-id="16d51-204">Open any ports required by the cloud application in the on-premises network.</span></span>

<span data-ttu-id="16d51-205">Ellenőrizze, hogy a kapcsolat ellenőrzéséhez:</span><span class="sxs-lookup"><span data-stu-id="16d51-205">Test the connection to verify that:</span></span>

* <span data-ttu-id="16d51-206">A helyszíni VPN-készülék megfelelően irányítja a forgalmat a felhő-alkalmazás az Azure VPN-átjárón keresztül.</span><span class="sxs-lookup"><span data-stu-id="16d51-206">The on-premises VPN appliance correctly routes traffic to the cloud application through the Azure VPN gateway.</span></span>
* <span data-ttu-id="16d51-207">A virtuális hálózat megfelelően irányítja a forgalmat vissza a helyszíni hálózat számára.</span><span class="sxs-lookup"><span data-stu-id="16d51-207">The VNet correctly routes traffic back to the on-premises network.</span></span>
* <span data-ttu-id="16d51-208">Mindkét irányban tiltott forgalmat megfelelően le van tiltva.</span><span class="sxs-lookup"><span data-stu-id="16d51-208">Prohibited traffic in both directions is blocked correctly.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="16d51-209">Méretezési szempontok</span><span class="sxs-lookup"><span data-stu-id="16d51-209">Scalability considerations</span></span>

<span data-ttu-id="16d51-210">Korlátozott függőleges méretezhetőség érhet el, a Basic vagy Standard VPN Gateway SKU-n áthelyezését a magas teljesítmény VPN Termékváltozat.</span><span class="sxs-lookup"><span data-stu-id="16d51-210">You can achieve limited vertical scalability by moving from the Basic or Standard VPN Gateway SKUs to the High Performance VPN SKU.</span></span>

<span data-ttu-id="16d51-211">VPN-forgalmát nagy mennyiségű teheti Vnetek fontolja meg a különböző terhelésekhez külön kisebb Vnetek történő terjesztése, és azok a VPN-átjáró konfigurálása.</span><span class="sxs-lookup"><span data-stu-id="16d51-211">For VNets that expect a large volume of VPN traffic, consider distributing the different workloads into separate smaller VNets and configuring a VPN gateway for each of them.</span></span>

<span data-ttu-id="16d51-212">A VNet is particionálása, vízszintes vagy függőleges irányú.</span><span class="sxs-lookup"><span data-stu-id="16d51-212">You can partition the VNet either horizontally or vertically.</span></span> <span data-ttu-id="16d51-213">Vízszintes particionálása, áthelyezni egyes Virtuálisgép-példányok egyes rétegek az új VNet alhálózatainak.</span><span class="sxs-lookup"><span data-stu-id="16d51-213">To partition horizontally, move some VM instances from each tier into subnets of the new VNet.</span></span> <span data-ttu-id="16d51-214">Az eredménye, hogy minden virtuális hálózat rendelkezik-e ugyanazokat a struktúra és funkciókat.</span><span class="sxs-lookup"><span data-stu-id="16d51-214">The result is that each VNet has the same structure and functionality.</span></span> <span data-ttu-id="16d51-215">Függőleges particionálásához, Újratervezés az egyes rétegek segítségével kiszámolja a funkciók különböző logikai területekre (például rendelések kezelése, a számlázással, felhasználói fiókok kezelése és így tovább).</span><span class="sxs-lookup"><span data-stu-id="16d51-215">To partition vertically, redesign each tier to divide the functionality into different logical areas (such as handling orders, invoicing, customer account management, and so on).</span></span> <span data-ttu-id="16d51-216">Minden egyes területhez majd a saját virtuális kell elhelyezni.</span><span class="sxs-lookup"><span data-stu-id="16d51-216">Each functional area can then be placed in its own VNet.</span></span>

<span data-ttu-id="16d51-217">Egy a helyszíni Active Directory-tartományvezérlőt a VNet replikálásának és valósít meg a virtuális hálózat DNS segíthet a helyszíni áramlanak a felhőbe biztonsági és felügyeleti forgalom bizonyos részének csökkentése érdekében.</span><span class="sxs-lookup"><span data-stu-id="16d51-217">Replicating an on-premises Active Directory domain controller in the VNet, and implementing DNS in the VNet, can help to reduce some of the security-related and administrative traffic flowing from on-premises to the cloud.</span></span> <span data-ttu-id="16d51-218">További információkért lásd: [kiterjesztése az Active Directory tartományi szolgáltatások (AD DS) az Azure-bA][adds-extend-domain].</span><span class="sxs-lookup"><span data-stu-id="16d51-218">For more information, see [Extending Active Directory Domain Services (AD DS) to Azure][adds-extend-domain].</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="16d51-219">Rendelkezésre állási szempontok</span><span class="sxs-lookup"><span data-stu-id="16d51-219">Availability considerations</span></span>

<span data-ttu-id="16d51-220">Győződjön meg arról, hogy a helyszíni hálózat továbbra is elérhető az Azure VPN gatewayhez van szüksége, ha valósítja meg a helyszíni VPN-átjáró a feladatátvevő fürtökben.</span><span class="sxs-lookup"><span data-stu-id="16d51-220">If you need to ensure that the on-premises network remains available to the Azure VPN gateway, implement a failover cluster for the on-premises VPN gateway.</span></span>

<span data-ttu-id="16d51-221">Ha a szervezete több helyszíni helyeken, hozzon létre [többhelyes kapcsolatok] [ vpn-gateway-multi-site] egy vagy több Azure Vnet számára.</span><span class="sxs-lookup"><span data-stu-id="16d51-221">If your organization has multiple on-premises sites, create [multi-site connections][vpn-gateway-multi-site] to one or more Azure VNets.</span></span> <span data-ttu-id="16d51-222">Ez a megközelítés megadását követeli meg dinamikus (útválasztó-alapú), ezért győződjön meg arról, hogy a helyszíni VPN-átjáró támogatja-e ezt a szolgáltatást.</span><span class="sxs-lookup"><span data-stu-id="16d51-222">This approach requires dynamic (route-based) routing, so make sure that the on-premises VPN gateway supports this feature.</span></span>

<span data-ttu-id="16d51-223">További információk a szolgáltatásszint-szerződések: [SLA-t, a VPN-átjáró][sla-for-vpn-gateway].</span><span class="sxs-lookup"><span data-stu-id="16d51-223">For details about service level agreements, see [SLA for VPN Gateway][sla-for-vpn-gateway].</span></span> 

## <a name="manageability-considerations"></a><span data-ttu-id="16d51-224">Felügyeleti szempontok</span><span class="sxs-lookup"><span data-stu-id="16d51-224">Manageability considerations</span></span>

<span data-ttu-id="16d51-225">A helyszíni VPN készülékek diagnosztikai információk figyelését.</span><span class="sxs-lookup"><span data-stu-id="16d51-225">Monitor diagnostic information from on-premises VPN appliances.</span></span> <span data-ttu-id="16d51-226">Ez az eljárás attól függ, hogy a VPN-készülék által nyújtott szolgáltatásokat.</span><span class="sxs-lookup"><span data-stu-id="16d51-226">This process depends on the features provided by the VPN appliance.</span></span> <span data-ttu-id="16d51-227">Ha használja az Útválasztás és távelérés szolgáltatás Windows Server 2012-ben például [RRAS naplózási][rras-logging].</span><span class="sxs-lookup"><span data-stu-id="16d51-227">For example, if you are using the Routing and Remote Access Service on Windows Server 2012, [RRAS logging][rras-logging].</span></span>

<span data-ttu-id="16d51-228">Használjon [Azure VPN gateway diagnosztika] [ gateway-diagnostic-logs] kapcsolódási problémák vonatkozó információkat.</span><span class="sxs-lookup"><span data-stu-id="16d51-228">Use [Azure VPN gateway diagnostics][gateway-diagnostic-logs] to capture information about connectivity issues.</span></span> <span data-ttu-id="16d51-229">Ezek a naplók információkat, például a forrás és a célhelyek csatlakozási kérelmek, használt protokollt, és hogyan jött létre a kapcsolat (vagy miért nem sikerült) nyomon követésére használható.</span><span class="sxs-lookup"><span data-stu-id="16d51-229">These logs can be used to track information such as the source and destinations of connection requests, which protocol was used, and how the connection was established (or why the attempt failed).</span></span>

<span data-ttu-id="16d51-230">Figyeléséhez használja a felügyeleti naplók az Azure portálon elérhető Azure VPN-átjáró a műveleti naplókat.</span><span class="sxs-lookup"><span data-stu-id="16d51-230">Monitor the operational logs of the Azure VPN gateway using the audit logs available in the Azure portal.</span></span> <span data-ttu-id="16d51-231">Külön elérhetők naplók a helyi hálózati átjáró, az Azure-hálózatot átjáró és a kapcsolat.</span><span class="sxs-lookup"><span data-stu-id="16d51-231">Separate logs are available for the local network gateway, the Azure network gateway, and the connection.</span></span> <span data-ttu-id="16d51-232">Ezt az információt az átjáró végrehajtott módosítások követésére használható, és akkor lehet hasznos, ha korábban már működő átjáró valamilyen okból kifolyólag nem működik.</span><span class="sxs-lookup"><span data-stu-id="16d51-232">This information can be used to track any changes made to the gateway, and can be useful if a previously functioning gateway stops working for some reason.</span></span>

<span data-ttu-id="16d51-233">![[2]][2]</span><span class="sxs-lookup"><span data-stu-id="16d51-233">![[2]][2]</span></span>

<span data-ttu-id="16d51-234">Figyelje a kapcsolatot, és kövesse a kapcsolatra vonatkozó hibaesemények.</span><span class="sxs-lookup"><span data-stu-id="16d51-234">Monitor connectivity, and track connectivity failure events.</span></span> <span data-ttu-id="16d51-235">Használhatja például a felügyeleti csomag [Nagios] [ nagios] rögzítéséhez, és megjeleníti ezt.</span><span class="sxs-lookup"><span data-stu-id="16d51-235">You can use a monitoring package such as [Nagios][nagios] to capture and report this information.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="16d51-236">Biztonsági szempontok</span><span class="sxs-lookup"><span data-stu-id="16d51-236">Security considerations</span></span>

<span data-ttu-id="16d51-237">Minden egyes VPN-átjáró különböző megosztott kulcs létrehozása.</span><span class="sxs-lookup"><span data-stu-id="16d51-237">Generate a different shared key for each VPN gateway.</span></span> <span data-ttu-id="16d51-238">Megosztott kulcs erős a segítségével ellenáll a találgatásos támadások során.</span><span class="sxs-lookup"><span data-stu-id="16d51-238">Use a strong shared key to help resist brute-force attacks.</span></span>

> [!NOTE]
> <span data-ttu-id="16d51-239">Az Azure Key Vault jelenleg nem használható az Azure VPN gateway kulcsokat preshare.</span><span class="sxs-lookup"><span data-stu-id="16d51-239">Currently, you cannot use Azure Key Vault to preshare keys for the Azure VPN gateway.</span></span>
> 
> 

<span data-ttu-id="16d51-240">Győződjön meg arról, hogy a helyszíni VPN-készülék használ, amely titkosítási metódus [kompatibilis az Azure VPN gateway][vpn-appliance-ipsec].</span><span class="sxs-lookup"><span data-stu-id="16d51-240">Ensure that the on-premises VPN appliance uses an encryption method that is [compatible with the Azure VPN gateway][vpn-appliance-ipsec].</span></span> <span data-ttu-id="16d51-241">A házirendalapú útválasztást, az Azure VPN gateway AES256 AES128 és a 3DES titkosítási algoritmus támogatja.</span><span class="sxs-lookup"><span data-stu-id="16d51-241">For policy-based routing, the Azure VPN gateway supports the AES256, AES128, and 3DES encryption algorithms.</span></span> <span data-ttu-id="16d51-242">Útválasztó-alapú átjárók AES256 és 3DES támogatja.</span><span class="sxs-lookup"><span data-stu-id="16d51-242">Route-based gateways support AES256 and 3DES.</span></span>

<span data-ttu-id="16d51-243">Ha a helyszíni VPN-készülék szegélyhálózaton (DMZ) van-e a peremhálózat és az Internet között tűzfal található, lehetséges, hogy konfigurálása [további tűzfalszabályok] [ additional-firewall-rules] engedélyezi a pont-pont VPN-kapcsolatot.</span><span class="sxs-lookup"><span data-stu-id="16d51-243">If your on-premises VPN appliance is on a perimeter network (DMZ) that has a firewall between the perimeter network and the Internet, you might have to configure [additional firewall rules][additional-firewall-rules] to allow the site-to-site VPN connection.</span></span>

<span data-ttu-id="16d51-244">Ha az alkalmazás virtuális adatokat küld az interneten, fontolja meg a [végrehajtási kényszerített bújtatás] [ forced-tunneling] irányíthatja az összes internetes-forgalmat a helyi hálózaton keresztül.</span><span class="sxs-lookup"><span data-stu-id="16d51-244">If the application in the VNet sends data to the Internet, consider [implementing forced tunneling][forced-tunneling] to route all Internet-bound traffic through the on-premises network.</span></span> <span data-ttu-id="16d51-245">Ez a megközelítés lehetővé teszi naplózása által az alkalmazás a helyszíni infrastruktúra kimenő kérelmek.</span><span class="sxs-lookup"><span data-stu-id="16d51-245">This approach enables you to audit outgoing requests made by the application from the on-premises infrastructure.</span></span>

> [!NOTE]
> <span data-ttu-id="16d51-246">A kényszerített bújtatás hatással lehet az Azure-szolgáltatások (a tároló szolgáltatás, például) és a Windows Licenckezelő való kapcsolódást.</span><span class="sxs-lookup"><span data-stu-id="16d51-246">Forced tunneling can impact connectivity to Azure services (the Storage Service, for example) and the Windows license manager.</span></span>
> 
> 


## <a name="troubleshooting"></a><span data-ttu-id="16d51-247">Hibaelhárítás</span><span class="sxs-lookup"><span data-stu-id="16d51-247">Troubleshooting</span></span> 

<span data-ttu-id="16d51-248">VPN-kapcsolatos gyakori hibák elhárításával kapcsolatos általános információkért lásd: [közös VPN hibaelhárítási kapcsolódó hibák][troubleshooting-vpn-errors].</span><span class="sxs-lookup"><span data-stu-id="16d51-248">For general information on troubleshooting common VPN-related errors, see [Troubleshooting common VPN related errors][troubleshooting-vpn-errors].</span></span>

<span data-ttu-id="16d51-249">A következő javaslatok még meghatározásához hasznos, ha a helyszíni VPN-készülék megfelelően működik-e.</span><span class="sxs-lookup"><span data-stu-id="16d51-249">The following recommendations are useful for determining if your on-premises VPN appliance is functioning correctly.</span></span>

- <span data-ttu-id="16d51-250">**A VPN-készülék a hibákat, a hibák által létrehozott naplófájlok ellenőrzése.**</span><span class="sxs-lookup"><span data-stu-id="16d51-250">**Check any log files generated by the VPN appliance for errors or failures.**</span></span>

    <span data-ttu-id="16d51-251">Ez segít meghatározni, ha a VPN-készülék megfelelően működik-e.</span><span class="sxs-lookup"><span data-stu-id="16d51-251">This will help you determine if the VPN appliance is functioning correctly.</span></span> <span data-ttu-id="16d51-252">Ezek az információk helyét a készülék függően változnak.</span><span class="sxs-lookup"><span data-stu-id="16d51-252">The location of this information will vary according to your appliance.</span></span> <span data-ttu-id="16d51-253">Például RRAS használ a Windows Server 2012, a következő PowerShell-parancsot a hiba esemény megjelenítheti az RRAS szolgáltatás is használhatja:</span><span class="sxs-lookup"><span data-stu-id="16d51-253">For example, if you are using RRAS on Windows Server 2012, you can use the following PowerShell command to display error event information for the RRAS service:</span></span>

    ```PowerShell
    Get-EventLog -LogName System -EntryType Error -Source RemoteAccess | Format-List -Property *
    ```

    <span data-ttu-id="16d51-254">A *üzenet* az egyes bejegyzések tulajdonság tartalmazza a hiba leírása.</span><span class="sxs-lookup"><span data-stu-id="16d51-254">The *Message* property of each entry provides a description of the error.</span></span> <span data-ttu-id="16d51-255">Néhány gyakori példák:</span><span class="sxs-lookup"><span data-stu-id="16d51-255">Some common examples are:</span></span>

        - Inability to connect, possibly due to an incorrect IP address specified for the Azure VPN gateway in the RRAS VPN network interface configuration.

        ```
        EventID            : 20111
        MachineName        : on-prem-vm
        Data               : {41, 3, 0, 0}
        Index              : 14231
        Category           : (0)
        CategoryNumber     : 0
        EntryType          : Error
        Message            : RoutingDomainID- {00000000-0000-0000-0000-000000000000}: A demand dial connection to the remote
                             interface AzureGateway on port VPN2-4 was successfully initiated but failed to complete
                             successfully because of the  following error: The network connection between your computer and
                             the VPN server could not be established because the remote server is not responding. This could
                             be because one of the network devices (for example, firewalls, NAT, routers, and so on) between your computer
                             and the remote server is not configured to allow VPN connections. Please contact your
                             Administrator or your service provider to determine which device may be causing the problem.
        Source             : RemoteAccess
        ReplacementStrings : {{00000000-0000-0000-0000-000000000000}, AzureGateway, VPN2-4, The network connection between
                             your computer and the VPN server could not be established because the remote server is not
                             responding. This could be because one of the network devices (for example, firewalls, NAT, routers, and so on)
                             between your computer and the remote server is not configured to allow VPN connections. Please
                             contact your Administrator or your service provider to determine which device may be causing the
                             problem.}
        InstanceId         : 20111
        TimeGenerated      : 3/18/2016 1:26:02 PM
        TimeWritten        : 3/18/2016 1:26:02 PM
        UserName           :
        Site               :
        Container          :
        ```

        - The wrong shared key being specified in the RRAS VPN network interface configuration.

        ```
        EventID            : 20111
        MachineName        : on-prem-vm
        Data               : {233, 53, 0, 0}
        Index              : 14245
        Category           : (0)
        CategoryNumber     : 0
        EntryType          : Error
        Message            : RoutingDomainID- {00000000-0000-0000-0000-000000000000}: A demand dial connection to the remote
                             interface AzureGateway on port VPN2-4 was successfully initiated but failed to complete
                             successfully because of the  following error: Internet key exchange (IKE) authentication credentials are unacceptable.

        Source             : RemoteAccess
        ReplacementStrings : {{00000000-0000-0000-0000-000000000000}, AzureGateway, VPN2-4, IKE authentication credentials are
                             unacceptable.
                             }
        InstanceId         : 20111
        TimeGenerated      : 3/18/2016 1:34:22 PM
        TimeWritten        : 3/18/2016 1:34:22 PM
        UserName           :
        Site               :
        Container          :
        ```

    <span data-ttu-id="16d51-256">Eseménynapló információ kísérel meg csatlakozni az RRAS szolgáltatás, a következő PowerShell-paranccsal is szerezheti be:</span><span class="sxs-lookup"><span data-stu-id="16d51-256">You can also obtain event log information about attempts to connect through the RRAS service using the following PowerShell command:</span></span> 

    ```
    Get-EventLog -LogName Application -Source RasClient | Format-List -Property *
    ```

    <span data-ttu-id="16d51-257">Csatlakozás meghibásodása ezt a naplót az alábbihoz hasonló hibák fogja tartalmazni:</span><span class="sxs-lookup"><span data-stu-id="16d51-257">In the event of a failure to connect, this log will contain errors that look similar to the following:</span></span>

    ```
    EventID            : 20227
    MachineName        : on-prem-vm
    Data               : {}
    Index              : 4203
    Category           : (0)
    CategoryNumber     : 0
    EntryType          : Error
    Message            : CoId={B4000371-A67F-452F-AA4C-3125AA9CFC78}: The user SYSTEM dialed a connection named
                         AzureGateway that has failed. The error code returned on failure is 809.
    Source             : RasClient
    ReplacementStrings : {{B4000371-A67F-452F-AA4C-3125AA9CFC78}, SYSTEM, AzureGateway, 809}
    InstanceId         : 20227
    TimeGenerated      : 3/18/2016 1:29:21 PM
    TimeWritten        : 3/18/2016 1:29:21 PM
    UserName           :
    Site               :
    Container          :
    ```

- <span data-ttu-id="16d51-258">**Ellenőrizze a kapcsolatot és továbbítását a VPN-átjáró között.**</span><span class="sxs-lookup"><span data-stu-id="16d51-258">**Verify connectivity and routing across the VPN gateway.**</span></span>

    <span data-ttu-id="16d51-259">A VPN-készülék előfordulhat, hogy nem lehet megfelelően útválasztási az Azure VPN Gateway forgalmát.</span><span class="sxs-lookup"><span data-stu-id="16d51-259">The VPN appliance may not be correctly routing traffic through the Azure VPN Gateway.</span></span> <span data-ttu-id="16d51-260">Használjon például egy eszköz [PsPing] [ psping] ellenőrizni a kapcsolatot és továbbítását a VPN-átjáró között.</span><span class="sxs-lookup"><span data-stu-id="16d51-260">Use a tool such as [PsPing][psping] to verify connectivity and routing across the VPN gateway.</span></span> <span data-ttu-id="16d51-261">Például egy olyan kiszolgálóra, a virtuális hálózaton található a helyi gépről kapcsolat tesztelése, a következő parancsot (cseréje `<<web-server-address>>` a webalkalmazás-kiszolgáló címe):</span><span class="sxs-lookup"><span data-stu-id="16d51-261">For example, to test connectivity from an on-premises machine to a web server located on the VNet, run the following command (replacing `<<web-server-address>>` with the address of the web server):</span></span>

    ```
    PsPing -t <<web-server-address>>:80
    ```

    <span data-ttu-id="16d51-262">Ha a helyszíni gépre irányíthatja a forgalmat a webkiszolgálónak, a következőhöz hasonló kimenetnek kell megjelennie:</span><span class="sxs-lookup"><span data-stu-id="16d51-262">If the on-premises machine can route traffic to the web server, you should see output similar to the following:</span></span>

    ```
    D:\PSTools>psping -t 10.20.0.5:80

    PsPing v2.01 - PsPing - ping, latency, bandwidth measurement utility
    Copyright (C) 2012-2014 Mark Russinovich
    Sysinternals - www.sysinternals.com

    TCP connect to 10.20.0.5:80:
    Infinite iterations (warmup 1) connecting test:
    Connecting to 10.20.0.5:80 (warmup): 6.21ms
    Connecting to 10.20.0.5:80: 3.79ms
    Connecting to 10.20.0.5:80: 3.44ms
    Connecting to 10.20.0.5:80: 4.81ms

      Sent = 3, Received = 3, Lost = 0 (0% loss),
      Minimum = 3.44ms, Maximum = 4.81ms, Average = 4.01ms
    ```

    <span data-ttu-id="16d51-263">A helyi számítógép nem tud kommunikálni a megadott helyre, ha ehhez hasonló üzenetet fog látni:</span><span class="sxs-lookup"><span data-stu-id="16d51-263">If the on-premises machine cannot communicate with the specified destination, you will see messages like this:</span></span>

    ```
    D:\PSTools>psping -t 10.20.1.6:80

    PsPing v2.01 - PsPing - ping, latency, bandwidth measurement utility
    Copyright (C) 2012-2014 Mark Russinovich
    Sysinternals - www.sysinternals.com

    TCP connect to 10.20.1.6:80:
    Infinite iterations (warmup 1) connecting test:
    Connecting to 10.20.1.6:80 (warmup): This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80: This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80: This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80: This operation returned because the timeout period expired.
    Connecting to 10.20.1.6:80:
      Sent = 3, Received = 0, Lost = 3 (100% loss),
      Minimum = 0.00ms, Maximum = 0.00ms, Average = 0.00ms
    ```

- <span data-ttu-id="16d51-264">**Győződjön meg arról, hogy a helyi tűzfal engedélyezi-e a VPN-adatforgalmat, és a megfelelő portokat.**</span><span class="sxs-lookup"><span data-stu-id="16d51-264">**Verify that the on-premises firewall allows VPN traffic to pass and that the correct ports are opened.**</span></span>

- <span data-ttu-id="16d51-265">**Győződjön meg arról, hogy a helyszíni VPN-készülék használ, amely titkosítási metódus [kompatibilis az Azure VPN gateway][vpn-appliance].**</span><span class="sxs-lookup"><span data-stu-id="16d51-265">**Verify that the on-premises VPN appliance uses an encryption method that is [compatible with the Azure VPN gateway][vpn-appliance].**</span></span> <span data-ttu-id="16d51-266">A házirendalapú útválasztást, az Azure VPN gateway AES256 AES128 és a 3DES titkosítási algoritmus támogatja.</span><span class="sxs-lookup"><span data-stu-id="16d51-266">For policy-based routing, the Azure VPN gateway supports the AES256, AES128, and 3DES encryption algorithms.</span></span> <span data-ttu-id="16d51-267">Útválasztó-alapú átjárók AES256 és 3DES támogatja.</span><span class="sxs-lookup"><span data-stu-id="16d51-267">Route-based gateways support AES256 and 3DES.</span></span>

<span data-ttu-id="16d51-268">Az alábbi javaslatokat hasznosak meghatározhatja, hogy az Azure VPN gateway probléma:</span><span class="sxs-lookup"><span data-stu-id="16d51-268">The following recommendations are useful for determining if there is a problem with the Azure VPN gateway:</span></span>

- <span data-ttu-id="16d51-269">**Vizsgálja meg [Azure VPN-átjáró diagnosztikai naplók] [ gateway-diagnostic-logs] az esetleges problémákat.**</span><span class="sxs-lookup"><span data-stu-id="16d51-269">**Examine [Azure VPN gateway diagnostic logs][gateway-diagnostic-logs] for potential issues.**</span></span>

- <span data-ttu-id="16d51-270">**Győződjön meg arról, hogy ugyanazt a megosztott hitelesítési kulcsot az Azure VPN gateway és a helyszíni VPN-készülék vannak konfigurálva.**</span><span class="sxs-lookup"><span data-stu-id="16d51-270">**Verify that the Azure VPN gateway and on-premises VPN appliance are configured with the same shared authentication key.**</span></span>

    <span data-ttu-id="16d51-271">A megosztott kulcs tárolja az Azure VPN gateway a következő Azure CLI-parancs használatával tekintheti meg:</span><span class="sxs-lookup"><span data-stu-id="16d51-271">You can view the shared key stored by the Azure VPN gateway using the following Azure CLI command:</span></span>

    ```
    azure network vpn-connection shared-key show <<resource-group>> <<vpn-connection-name>>
    ```

    <span data-ttu-id="16d51-272">A paranccsal a helyszíni VPN-készülék megfelelő megjelenítése a megosztott kulcs van konfigurálva, hogy alapplatformjaként.</span><span class="sxs-lookup"><span data-stu-id="16d51-272">Use the command appropriate for your on-premises VPN appliance to show the shared key configured for that appliance.</span></span>

    <span data-ttu-id="16d51-273">Ellenőrizze, hogy a *GatewaySubnet* alhálózat az Azure VPN gateway okozó nincs társítva egy NSG.</span><span class="sxs-lookup"><span data-stu-id="16d51-273">Verify that the *GatewaySubnet* subnet holding the Azure VPN gateway is not associated with an NSG.</span></span>

    <span data-ttu-id="16d51-274">Az alhálózati adatokat, a következő Azure CLI-parancs használatával tekintheti meg:</span><span class="sxs-lookup"><span data-stu-id="16d51-274">You can view the subnet details using the following Azure CLI command:</span></span>

    ```
    azure network vnet subnet show -g <<resource-group>> -e <<vnet-name>> -n GatewaySubnet
    ```

    <span data-ttu-id="16d51-275">Gondoskodjon arról, hogy nincs nevű adatmező *hálózati biztonsági csoport azonosítója*. A következő példa egy példányát az eredményeit jeleníti meg a *GatewaySubnet* , amely rendelkezik hozzárendelt NSG-t (*VPN-átjáró-csoport*).</span><span class="sxs-lookup"><span data-stu-id="16d51-275">Ensure there is no data field named *Network Security Group id*. The following example shows the results for an instance of the *GatewaySubnet* that has an assigned NSG (*VPN-Gateway-Group*).</span></span> <span data-ttu-id="16d51-276">Emiatt előfordulhat, hogy megfelelően működik, ha bármely szabály lett meghatározva az NSG az átjárót.</span><span class="sxs-lookup"><span data-stu-id="16d51-276">This can prevent the gateway from working correctly if there are any rules defined for this NSG.</span></span>

    ```
    C:\>azure network vnet subnet show -g profx-prod-rg -e profx-vnet -n GatewaySubnet
        info:    Executing command network vnet subnet show
        + Looking up virtual network "profx-vnet"
        + Looking up the subnet "GatewaySubnet"
        data:    Id                              : /subscriptions/########-####-####-####-############/resourceGroups/profx-prod-rg/providers/Microsoft.Network/virtualNetworks/profx-vnet/subnets/GatewaySubnet
        data:    Name                            : GatewaySubnet
        data:    Provisioning state              : Succeeded
        data:    Address prefix                  : 10.20.3.0/27
        data:    Network Security Group id       : /subscriptions/########-####-####-####-############/resourceGroups/profx-prod-rg/providers/Microsoft.Network/networkSecurityGroups/VPN-Gateway-Group
        info:    network vnet subnet show command OK
    ```

- <span data-ttu-id="16d51-277">**Győződjön meg arról, hogy a virtuális gépeket az Azure virtuális hálózatot a forgalmat a érkező a virtuális hálózaton kívül van konfigurálva.**</span><span class="sxs-lookup"><span data-stu-id="16d51-277">**Verify that the virtual machines in the Azure VNet are configured to permit traffic coming in from outside the VNet.**</span></span>

    <span data-ttu-id="16d51-278">Bármely, a virtuális gépeket tartalmazó alhálózattal társított NSG-szabályok keresése</span><span class="sxs-lookup"><span data-stu-id="16d51-278">Check any NSG rules associated with subnets containing these virtual machines.</span></span> <span data-ttu-id="16d51-279">Minden NSG-szabályok a következő Azure CLI-parancs használatával tekintheti meg:</span><span class="sxs-lookup"><span data-stu-id="16d51-279">You can view all NSG rules using the following Azure CLI command:</span></span>

    ```
    azure network nsg show -g <<resource-group>> -n <<nsg-name>>
    ```

- <span data-ttu-id="16d51-280">**Győződjön meg arról, hogy csatlakozik-e az Azure VPN gateway.**</span><span class="sxs-lookup"><span data-stu-id="16d51-280">**Verify that the Azure VPN gateway is connected.**</span></span>

    <span data-ttu-id="16d51-281">A következő Azure PowerShell-parancs segítségével az Azure VPN-kapcsolat jelenlegi állapotának ellenőrzése.</span><span class="sxs-lookup"><span data-stu-id="16d51-281">You can use the following Azure PowerShell command to check the current status of the Azure VPN connection.</span></span> <span data-ttu-id="16d51-282">A `<<connection-name>>` paramétere az Azure VPN-kapcsolat, amely a virtuális hálózati átjáró és a helyi átjáró nevét.</span><span class="sxs-lookup"><span data-stu-id="16d51-282">The `<<connection-name>>` parameter is the name of the Azure VPN connection that links the virtual network gateway and the local gateway.</span></span>

    ```
    Get-AzureRmVirtualNetworkGatewayConnection -Name <<connection-name>> - ResourceGroupName <<resource-group>>
    ```

    <span data-ttu-id="16d51-283">Az alábbi kódtöredékek jelölje ki a kimenetet állít elő, ha az átjáró csatlakoztatva (az első példában), és leválasztott (a második példában):</span><span class="sxs-lookup"><span data-stu-id="16d51-283">The following snippets highlight the output generated if the gateway is connected (the first example), and disconnected (the second example):</span></span>

    ```
    PS C:\> Get-AzureRmVirtualNetworkGatewayConnection -Name profx-gateway-connection -ResourceGroupName profx-prod-rg

    AuthorizationKey           :
    VirtualNetworkGateway1     : Microsoft.Azure.Commands.Network.Models.PSVirtualNetworkGateway
    VirtualNetworkGateway2     :
    LocalNetworkGateway2       : Microsoft.Azure.Commands.Network.Models.PSLocalNetworkGateway
    Peer                       :
    ConnectionType             : IPsec
    RoutingWeight              : 0
    SharedKey                  : ####################################
    ConnectionStatus           : Connected
    EgressBytesTransferred     : 55254803
    IngressBytesTransferred    : 32227221
    ProvisioningState          : Succeeded
    ...
    ```

    ```
    PS C:\> Get-AzureRmVirtualNetworkGatewayConnection -Name profx-gateway-connection2 -ResourceGroupName profx-prod-rg

    AuthorizationKey           :
    VirtualNetworkGateway1     : Microsoft.Azure.Commands.Network.Models.PSVirtualNetworkGateway
    VirtualNetworkGateway2     :
    LocalNetworkGateway2       : Microsoft.Azure.Commands.Network.Models.PSLocalNetworkGateway
    Peer                       :
    ConnectionType             : IPsec
    RoutingWeight              : 0
    SharedKey                  : ####################################
    ConnectionStatus           : NotConnected
    EgressBytesTransferred     : 0
    IngressBytesTransferred    : 0
    ProvisioningState          : Succeeded
    ...
    ```

<span data-ttu-id="16d51-284">Az alábbi javaslatokat hasznosak meghatározhatja, hogy a gazdagép VM konfigurációs, a hálózati sávszélesség-használatot vagy az alkalmazások teljesítményének problémát:</span><span class="sxs-lookup"><span data-stu-id="16d51-284">The following recommendations are useful for determining if there is an issue with Host VM configuration, network bandwidth utilization, or application performance:</span></span>

- <span data-ttu-id="16d51-285">**Győződjön meg arról, hogy az alhálózat az Azure virtuális gépeken futó vendég operációs rendszerben a tűzfal megfelelően van konfigurálva a helyi IP-címtartományok engedélyezett forgalmat engedélyezi.**</span><span class="sxs-lookup"><span data-stu-id="16d51-285">**Verify that the firewall in the guest operating system running on the Azure VMs in the subnet is configured correctly to allow permitted traffic from the on-premises IP ranges.**</span></span>

- <span data-ttu-id="16d51-286">**Győződjön meg arról, hogy a forgalom mennyiségének nincs közel az Azure VPN gateway számára elérhető sávszélesség korlátja.**</span><span class="sxs-lookup"><span data-stu-id="16d51-286">**Verify that the volume of traffic is not close to the limit of the bandwidth available to the Azure VPN gateway.**</span></span>

    <span data-ttu-id="16d51-287">Hogyan ellenőrizheti, ennek a VPN-készülék helyileg futó függ.</span><span class="sxs-lookup"><span data-stu-id="16d51-287">How to verify this depends on the VPN appliance running on-premises.</span></span> <span data-ttu-id="16d51-288">Például RRAS használatakor a Windows Server 2012 segítségével Teljesítményfigyelő nyomon követheti a fogadott és a VPN-kapcsolaton keresztül továbbított adatok mennyiségét.</span><span class="sxs-lookup"><span data-stu-id="16d51-288">For example, if you are using RRAS on Windows Server 2012, you can use Performance Monitor to track the volume of data being received and transmitted over the VPN connection.</span></span> <span data-ttu-id="16d51-289">Használja a *RAS teljes* objektum, jelölje be a *bájt/s* és *továbbítása-bájtok/s* számlálókat:</span><span class="sxs-lookup"><span data-stu-id="16d51-289">Using the *RAS Total* object, select the *Bytes Received/Sec* and *Bytes Transmitted/Sec* counters:</span></span>

    <span data-ttu-id="16d51-290">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="16d51-290">![[3]][3]</span></span>

    <span data-ttu-id="16d51-291">Az eredmények (a Basic és Standard termékváltozat 100 MB/s és a magas teljesítmény termékváltozat 200 MB/s) a VPN-átjáró számára elérhető sávszélesség kell összehasonlítani:</span><span class="sxs-lookup"><span data-stu-id="16d51-291">You should compare the results with the bandwidth available to the VPN gateway (100 Mbps for the Basic and Standard SKUs, and 200 Mbps for the High Performance SKU):</span></span>

    <span data-ttu-id="16d51-292">![[4]][4]</span><span class="sxs-lookup"><span data-stu-id="16d51-292">![[4]][4]</span></span>

- <span data-ttu-id="16d51-293">**Győződjön meg arról, hogy telepítette a megfelelő számát és a virtuális gépek méretét az alkalmazás terhelést.**</span><span class="sxs-lookup"><span data-stu-id="16d51-293">**Verify that you have deployed the right number and size of VMs for your application load.**</span></span>

    <span data-ttu-id="16d51-294">Határozza meg, ha a virtuális gépeket az Azure VNet bármelyikét lassan működik.</span><span class="sxs-lookup"><span data-stu-id="16d51-294">Determine if any of the virtual machines in the Azure VNet are running slowly.</span></span> <span data-ttu-id="16d51-295">Ha igen, akkor lehetséges, hogy túlterheltté, lehetséges, hogy túl kevés a terhelés kezelésére, vagy előfordulhat, hogy a terheléselosztókhoz nincs megfelelően konfigurálva.</span><span class="sxs-lookup"><span data-stu-id="16d51-295">If so, they may be overloaded, there may be too few to handle the load, or the load-balancers may not be configured correctly.</span></span> <span data-ttu-id="16d51-296">Annak meghatározására, [rögzíteni és elemezni a diagnosztikai adatok][azure-vm-diagnostics].</span><span class="sxs-lookup"><span data-stu-id="16d51-296">To determine this, [capture and analyze diagnostic information][azure-vm-diagnostics].</span></span> <span data-ttu-id="16d51-297">Az eredményeket az Azure portál használatával ellenőrizheti, de sok külső eszközök is elérhető, amely is részletes információkat adnak a teljesítményadatokat.</span><span class="sxs-lookup"><span data-stu-id="16d51-297">You can examine the results using the Azure portal, but many third-party tools are also available that can provide detailed insights into the performance data.</span></span>

- <span data-ttu-id="16d51-298">**Győződjön meg arról, hogy az alkalmazás lehetővé teszi a felhőalapú erőforrások hatékony használatát.**</span><span class="sxs-lookup"><span data-stu-id="16d51-298">**Verify that the application is making efficient use of cloud resources.**</span></span>

    <span data-ttu-id="16d51-299">Eszköz alkalmazáskód annak meghatározásához, hogy alkalmazásokat a lehető legjobb felhasználását, az erőforrások létesített minden egyes virtuális gépen.</span><span class="sxs-lookup"><span data-stu-id="16d51-299">Instrument application code running on each VM to determine whether applications are making the best use of resources.</span></span> <span data-ttu-id="16d51-300">Használhatja például a [Application Insights][application-insights].</span><span class="sxs-lookup"><span data-stu-id="16d51-300">You can use tools such as [Application Insights][application-insights].</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="16d51-301">A megoldás üzembe helyezéséhez</span><span class="sxs-lookup"><span data-stu-id="16d51-301">Deploy the solution</span></span>


<span data-ttu-id="16d51-302">**Prequisites.**</span><span class="sxs-lookup"><span data-stu-id="16d51-302">**Prequisites.**</span></span> <span data-ttu-id="16d51-303">Rendelkeznie kell egy meglévő helyi infrastruktúra már be van állítva a megfelelő hálózati berendezések.</span><span class="sxs-lookup"><span data-stu-id="16d51-303">You must have an existing on-premises infrastructure already configured with a suitable network appliance.</span></span>

<span data-ttu-id="16d51-304">A megoldás üzembe helyezéséhez, a következő lépésekkel.</span><span class="sxs-lookup"><span data-stu-id="16d51-304">To deploy the solution, perform the following steps.</span></span>

1. <span data-ttu-id="16d51-305">Kattintson a lenti gombra:</span><span class="sxs-lookup"><span data-stu-id="16d51-305">Click the button below:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fvpn%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. <span data-ttu-id="16d51-306">Várjon, amíg a hivatkozásra kattintva nyissa meg az Azure portálon, majd kövesse az alábbi lépéseket:</span><span class="sxs-lookup"><span data-stu-id="16d51-306">Wait for the link to open in the Azure portal, then follow these steps:</span></span> 
   * <span data-ttu-id="16d51-307">Az **Erőforráscsoport** neve már meg van adva a paraméterfájlban, ezért válassza az **Új létrehozása** lehetőséget és a szövegmezőbe írja az `ra-hybrid-vpn-rg` karakterláncot.</span><span class="sxs-lookup"><span data-stu-id="16d51-307">The **Resource group** name is already defined in the parameter file, so select **Create New** and enter `ra-hybrid-vpn-rg` in the text box.</span></span>
   * <span data-ttu-id="16d51-308">Válassza ki a régiót a **Hely** legördülő listából.</span><span class="sxs-lookup"><span data-stu-id="16d51-308">Select the region from the **Location** drop down box.</span></span>
   * <span data-ttu-id="16d51-309">Ne szerkessze a **Sablon gyökér szintű URI-je** vagy a **Paraméter gyökér szintű URI-je** szövegmezőt.</span><span class="sxs-lookup"><span data-stu-id="16d51-309">Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.</span></span>
   * <span data-ttu-id="16d51-310">Tekintse át a használati feltételeket, majd kattintson az **Elfogadom a fenti feltételeket** lehetőségre.</span><span class="sxs-lookup"><span data-stu-id="16d51-310">Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.</span></span>
   * <span data-ttu-id="16d51-311">Kattintson a **beszerzési** gombra.</span><span class="sxs-lookup"><span data-stu-id="16d51-311">Click the **Purchase** button.</span></span>
3. <span data-ttu-id="16d51-312">Várjon, amíg az üzembe helyezés befejeződik.</span><span class="sxs-lookup"><span data-stu-id="16d51-312">Wait for the deployment to complete.</span></span>



<!-- links -->

[adds-extend-domain]: ../identity/adds-extend-domain.md
[expressroute]: ../hybrid-networking/expressroute.md
[windows-vm-ra]: ../virtual-machines-windows/index.md
[linux-vm-ra]: ../virtual-machines-linux/index.md

[naming conventions]: /azure/guidance/guidance-naming-conventions

[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[arm-templates]: /azure/resource-group-authoring-templates
[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-portal]: /azure/azure-portal/resource-group-portal
[azure-powershell]: /azure/powershell-azure-resource-manager
[azure-virtual-network]: /azure/virtual-network/virtual-networks-overview
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[azure-vpn-gateway]: https://azure.microsoft.com/services/vpn-gateway/
[azure-gateway-charges]: https://azure.microsoft.com/pricing/details/vpn-gateway/
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[vpn-gateway-multi-site]: /azure/vpn-gateway/vpn-gateway-multi-site
[policy-based-routing]: https://en.wikipedia.org/wiki/Policy-based_routing
[route-based-routing]: https://en.wikipedia.org/wiki/Static_routing
[network-security-group]: /azure/virtual-network/virtual-networks-nsg
[sla-for-vpn-gateway]: https://azure.microsoft.com/support/legal/sla/vpn-gateway/v1_2/
[additional-firewall-rules]: https://technet.microsoft.com/library/dn786406.aspx#firewall
[nagios]: https://www.nagios.org/
[azure-vpn-gateway-diagnostics]: http://blogs.technet.com/b/keithmayer/archive/2014/12/18/diagnose-azure-virtual-network-vpn-connectivity-issues-with-powershell.aspx
[ping]: https://technet.microsoft.com/library/ff961503.aspx
[tracert]: https://technet.microsoft.com/library/ff961507.aspx
[psping]: http://technet.microsoft.com/sysinternals/jj729731.aspx
[nmap]: http://nmap.org
[changing-SKUs]: https://azure.microsoft.com/blog/azure-virtual-network-gateway-improvements/
[gateway-diagnostic-logs]: http://blogs.technet.com/b/keithmayer/archive/2015/12/07/step-by-step-capturing-azure-resource-manager-arm-vnet-gateway-diagnostic-logs.aspx
[troubleshooting-vpn-errors]: https://blogs.technet.microsoft.com/rrasblog/2009/08/12/troubleshooting-common-vpn-related-errors/
[rras-logging]: https://www.petri.com/enable-diagnostic-logging-in-windows-server-2012-r2-routing-and-remote-access
[create-on-prem-network]: https://technet.microsoft.com/library/dn786406.aspx#routing
[azure-vm-diagnostics]: https://azure.microsoft.com/blog/windows-azure-virtual-machine-monitoring-with-wad-extension/
[application-insights]: /azure/application-insights/app-insights-overview-usage
[forced-tunneling]: https://azure.microsoft.com/documentation/articles/vpn-gateway-about-forced-tunneling/
[vpn-appliances]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[visio-download]: https://archcenter.azureedge.net/cdn/hybrid-network-architectures.vsdx
[vpn-appliance-ipsec]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices#ipsec-parameters
<!--[solution-script]: https://github.com/mspnp/reference-architectures/tree/master/guidance-hybrid-network-vpn/Deploy-ReferenceArchitecture.ps1-->
<!--[solution-script-bash]: https://github.com/mspnp/reference-architectures/tree/master/guidance-hybrid-network-vpn/deploy-reference-architecture.sh-->
<!--[virtualNetworkGateway-parameters]: https://github.com/mspnp/reference-architectures/tree/master/guidance-hybrid-network-vpn/parameters/virtualNetworkGateway.parameters.json-->
[azure-cli]: https://azure.microsoft.com/documentation/articles/xplat-cli-install/
[CIDR]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[0]: ./images/vpn.png "A helyszíni és az Azure infrastruktúra-áthidalás hibrid hálózati"
[2]: ../_images/guidance-hybrid-network-vpn/audit-logs.png "A naplók az Azure-portálon"
[3]: ../_images/guidance-hybrid-network-vpn/RRAS-perf-counters.png "Teljesítményszámlálók a VPN-hálózati forgalom figyelése"
[4]: ../_images/guidance-hybrid-network-vpn/RRAS-perf-graph.png "Példa VPN hálózat teljesítménye ábra"