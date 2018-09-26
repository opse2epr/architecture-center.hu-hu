---
title: Helyszíni hálózat csatlakoztatása az Azure-hoz VPN használatával
description: A jelen cikk azt ismerteti, hogyan lehet implementálni egy helyek közötti biztonságos hálózati architektúrát, amely a VPN használatával összekapcsolt Azure-beli virtuális hálózatból és helyszíni hálózatból áll.
author: RohitSharma-pnp
ms.date: 11/28/2016
pnp.series.title: Connect an on-premises network to Azure
pnp.series.next: expressroute
pnp.series.prev: ./index
cardTitle: VPN
ms.openlocfilehash: d0966791642ee174d020c960c9fc6e72d8768539
ms.sourcegitcommit: b38ba378c9d6110da2dfd50b4233fadd94604bb0
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 09/25/2018
ms.locfileid: "47167438"
---
# <a name="connect-an-on-premises-network-to-azure-using-a-vpn-gateway"></a><span data-ttu-id="723ac-103">Helyszíni hálózat csatlakoztatása az Azure-hoz VPN-átjáró használatával</span><span class="sxs-lookup"><span data-stu-id="723ac-103">Connect an on-premises network to Azure using a VPN gateway</span></span>

<span data-ttu-id="723ac-104">Ez a referenciaarchitektúra bemutatja, hogyan lehet kibővíteni a helyszíni hálózatot az Azure-ra helyek közötti virtuális magánhálózat (VPN) használatával.</span><span class="sxs-lookup"><span data-stu-id="723ac-104">This reference architecture shows how to extend an on-premises network to Azure, using a site-to-site virtual private network (VPN).</span></span> <span data-ttu-id="723ac-105">A forgalom a helyszíni hálózat és egy Azure Virtual Network (VNet) között egy IPsec VPN-alagúton halad át.</span><span class="sxs-lookup"><span data-stu-id="723ac-105">Traffic flows between the on-premises network and an Azure Virtual Network (VNet) through an IPSec VPN tunnel.</span></span> [<span data-ttu-id="723ac-106">**A megoldás üzembe helyezése**.</span><span class="sxs-lookup"><span data-stu-id="723ac-106">**Deploy this solution**.</span></span>](#deploy-the-solution)

<span data-ttu-id="723ac-107">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="723ac-107">![[0]][0]</span></span>

<span data-ttu-id="723ac-108">*Töltse le az architektúra [Visio-fájlját][visio-download].*</span><span class="sxs-lookup"><span data-stu-id="723ac-108">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="723ac-109">Architektúra</span><span class="sxs-lookup"><span data-stu-id="723ac-109">Architecture</span></span> 

<span data-ttu-id="723ac-110">Az architektúra a következőkben leírt összetevőkből áll.</span><span class="sxs-lookup"><span data-stu-id="723ac-110">The architecture consists of the following components.</span></span>

* <span data-ttu-id="723ac-111">**Helyszíni hálózat**.</span><span class="sxs-lookup"><span data-stu-id="723ac-111">**On-premises network**.</span></span> <span data-ttu-id="723ac-112">A cégen belül futó helyi magánhálózat.</span><span class="sxs-lookup"><span data-stu-id="723ac-112">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="723ac-113">**VPN-berendezés**.</span><span class="sxs-lookup"><span data-stu-id="723ac-113">**VPN appliance**.</span></span> <span data-ttu-id="723ac-114">A helyszíni hálózat számára külső kapcsolatot biztosító eszköz vagy szolgáltatás.</span><span class="sxs-lookup"><span data-stu-id="723ac-114">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="723ac-115">A VPN-berendezés lehet hardvereszköz vagy valamilyen szoftvermegoldás, amilyen például a Windows Server 2012 Útválasztás és távelérés szolgáltatása (Routing and Remote Access Service, RRAS).</span><span class="sxs-lookup"><span data-stu-id="723ac-115">The VPN appliance may be a hardware device, or it can be a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="723ac-116">A támogatott VPN-berendezések listájáért és azok Azure-hoz való csatlakoztatásra történő konfigurálásával kapcsolatos információkért tekintse meg a kiválasztott eszközre vonatkozó utasításokat az [Információk a helyek közötti VPN Gateway-kapcsolatok VPN-eszközeiről][vpn-appliance] című cikkben.</span><span class="sxs-lookup"><span data-stu-id="723ac-116">For a list of supported VPN appliances and information on configuring them to connect to an Azure VPN gateway, see the instructions for the selected device in the article [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="723ac-117">**Virtuális hálózat (VNet)**.</span><span class="sxs-lookup"><span data-stu-id="723ac-117">**Virtual network (VNet)**.</span></span> <span data-ttu-id="723ac-118">A felhőalapú alkalmazás és az Azure VPN-átjáró összetevői ugyanazon a [virtuális hálózaton][azure-virtual-network] találhatók.</span><span class="sxs-lookup"><span data-stu-id="723ac-118">The cloud application and the components for the Azure VPN gateway reside in the same [VNet][azure-virtual-network].</span></span>

* <span data-ttu-id="723ac-119">**Azure VPN Gateway**.</span><span class="sxs-lookup"><span data-stu-id="723ac-119">**Azure VPN gateway**.</span></span> <span data-ttu-id="723ac-120">A [VPN Gateway][azure-vpn-gateway] szolgáltatás lehetővé teszi, hogy VPN-berendezésen keresztül csatlakoztassa a virtuális hálózatot a helyszíni hálózathoz.</span><span class="sxs-lookup"><span data-stu-id="723ac-120">The [VPN gateway][azure-vpn-gateway] service enables you to connect the VNet to the on-premises network through a VPN appliance.</span></span> <span data-ttu-id="723ac-121">További információért tekintse át a [helyszíni hálózat és a Microsoft Azure Virtual Network csatlakoztatásával][connect-to-an-Azure-vnet] foglalkozó cikket.</span><span class="sxs-lookup"><span data-stu-id="723ac-121">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span> <span data-ttu-id="723ac-122">A VPN-átjáró az alábbi elemeket tartalmazza:</span><span class="sxs-lookup"><span data-stu-id="723ac-122">The VPN gateway includes the following elements:</span></span>
  
  * <span data-ttu-id="723ac-123">**Virtuális hálózati átjáró**.</span><span class="sxs-lookup"><span data-stu-id="723ac-123">**Virtual network gateway**.</span></span> <span data-ttu-id="723ac-124">Egy, a virtuális hálózathoz virtuális VPN-berendezést biztosító erőforrás.</span><span class="sxs-lookup"><span data-stu-id="723ac-124">A resource that provides a virtual VPN appliance for the VNet.</span></span> <span data-ttu-id="723ac-125">Ez felelős az adatforgalom helyszíni hálózatról virtuális hálózatra való irányításáért.</span><span class="sxs-lookup"><span data-stu-id="723ac-125">It is responsible for routing traffic from the on-premises network to the VNet.</span></span>
  * <span data-ttu-id="723ac-126">**Helyi hálózati átjáró**.</span><span class="sxs-lookup"><span data-stu-id="723ac-126">**Local network gateway**.</span></span> <span data-ttu-id="723ac-127">A helyszíni VPN-készülék absztrakciója.</span><span class="sxs-lookup"><span data-stu-id="723ac-127">An abstraction of the on-premises VPN appliance.</span></span> <span data-ttu-id="723ac-128">A felhőalkalmazásról a helyszíni hálózatra irányuló hálózati forgalom ezen az átjárón halad át.</span><span class="sxs-lookup"><span data-stu-id="723ac-128">Network traffic from the cloud application to the on-premises network is routed through this gateway.</span></span>
  * <span data-ttu-id="723ac-129">**Kapcsolat**.</span><span class="sxs-lookup"><span data-stu-id="723ac-129">**Connection**.</span></span> <span data-ttu-id="723ac-130">A kapcsolat olyan tulajdonságokkal rendelkezik, amelyek megadják a kapcsolat típusát (IPSec) és a helyszíni VPN-berendezéssel megosztott kulcsot a forgalom titkosításához.</span><span class="sxs-lookup"><span data-stu-id="723ac-130">The connection has properties that specify the connection type (IPSec) and the key shared with the on-premises VPN appliance to encrypt traffic.</span></span>
  * <span data-ttu-id="723ac-131">**Átjáró-alhálózat**.</span><span class="sxs-lookup"><span data-stu-id="723ac-131">**Gateway subnet**.</span></span> <span data-ttu-id="723ac-132">A virtuális hálózati átjáró a saját alhálózatán található, amelynek számos, az alább található Javaslatok szakaszban leírt követelménynek meg kell felelnie.</span><span class="sxs-lookup"><span data-stu-id="723ac-132">The virtual network gateway is held in its own subnet, which is subject to various requirements, described in the Recommendations section below.</span></span>

* <span data-ttu-id="723ac-133">**Felhőalkalmazás**.</span><span class="sxs-lookup"><span data-stu-id="723ac-133">**Cloud application**.</span></span> <span data-ttu-id="723ac-134">Az Azure-ban üzemeltetett alkalmazás.</span><span class="sxs-lookup"><span data-stu-id="723ac-134">The application hosted in Azure.</span></span> <span data-ttu-id="723ac-135">Több réteget is foglalhat magában, amelyek alhálózatait Azure-terheléselosztók kapcsolják össze.</span><span class="sxs-lookup"><span data-stu-id="723ac-135">It might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="723ac-136">További információ az alkalmazás-infrastruktúrával kapcsolatban: [Windows rendszerű virtuális gépek számítási feladatainak futtatása][windows-vm-ra] és [Számítási feladatok futtatása Linux rendszerű virtuális gépeken][linux-vm-ra].</span><span class="sxs-lookup"><span data-stu-id="723ac-136">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

* <span data-ttu-id="723ac-137">**Belső terheléselosztó**.</span><span class="sxs-lookup"><span data-stu-id="723ac-137">**Internal load balancer**.</span></span> <span data-ttu-id="723ac-138">A rendszer a VPN-átjáróról érkező hálózati forgalmat egy belső terheléselosztón keresztül irányítja a felhőalkalmazásba.</span><span class="sxs-lookup"><span data-stu-id="723ac-138">Network traffic from the VPN gateway is routed to the cloud application through an internal load balancer.</span></span> <span data-ttu-id="723ac-139">A terheléselosztó az alkalmazás előtérbeli alhálózatán található.</span><span class="sxs-lookup"><span data-stu-id="723ac-139">The load balancer is located in the front-end subnet of the application.</span></span>

## <a name="recommendations"></a><span data-ttu-id="723ac-140">Javaslatok</span><span class="sxs-lookup"><span data-stu-id="723ac-140">Recommendations</span></span>

<span data-ttu-id="723ac-141">Az alábbi javaslatok a legtöbb forgatókönyvre vonatkoznak.</span><span class="sxs-lookup"><span data-stu-id="723ac-141">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="723ac-142">Kövesse ezeket a javaslatokat, ha nincsenek ezeket felülíró követelményei.</span><span class="sxs-lookup"><span data-stu-id="723ac-142">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="vnet-and-gateway-subnet"></a><span data-ttu-id="723ac-143">Virtuális hálózat és átjáró-alhálózat</span><span class="sxs-lookup"><span data-stu-id="723ac-143">VNet and gateway subnet</span></span>

<span data-ttu-id="723ac-144">Hozzon létre egy Azure virtuális hálózatot egy, az összes szükséges erőforrás számára elegendő méretű címtérrel.</span><span class="sxs-lookup"><span data-stu-id="723ac-144">Create an Azure VNet with an address space large enough for all of your required resources.</span></span> <span data-ttu-id="723ac-145">Ügyeljen arra, hogy a virtuális hálózat címtere a növekedésnek is biztosítson helyet, ha van rá esély, hogy a jövőben további virtuális gépekre is szükség lesz.</span><span class="sxs-lookup"><span data-stu-id="723ac-145">Ensure that the VNet address space has sufficient room for growth if additional VMs are likely to be needed in the future.</span></span> <span data-ttu-id="723ac-146">A virtuális hálózat címtere nem lehet átfedésben a helyszíni hálózattal.</span><span class="sxs-lookup"><span data-stu-id="723ac-146">The address space of the VNet must not overlap with the on-premises network.</span></span> <span data-ttu-id="723ac-147">A fenti ábra például a 10.20.0.0/16 címteret használja a virtuális hálózathoz.</span><span class="sxs-lookup"><span data-stu-id="723ac-147">For example, the diagram above uses the address space 10.20.0.0/16 for the VNet.</span></span>

<span data-ttu-id="723ac-148">Hozzon létre egy *GatewaySubnet* nevű alhálózatot /27 címtartománnyal.</span><span class="sxs-lookup"><span data-stu-id="723ac-148">Create a subnet named *GatewaySubnet*, with an address range of /27.</span></span> <span data-ttu-id="723ac-149">Ez az alhálózat a virtuális hálózat átjárójához szükséges.</span><span class="sxs-lookup"><span data-stu-id="723ac-149">This subnet is required by the virtual network gateway.</span></span> <span data-ttu-id="723ac-150">Ha kioszt 32 címet erre az alhálózatra, azzal megelőzheti az átjáró mérethatárának elérését a jövőben.</span><span class="sxs-lookup"><span data-stu-id="723ac-150">Allocating 32 addresses to this subnet will help to prevent reaching gateway size limitations in the future.</span></span> <span data-ttu-id="723ac-151">Lehetőleg a címtér közepére se helyezze ezt az alhálózatot.</span><span class="sxs-lookup"><span data-stu-id="723ac-151">Also, avoid placing this subnet in the middle of the address space.</span></span> <span data-ttu-id="723ac-152">Bevált gyakorlat az átjáró-alhálózat címterét a virtuális hálózat címterének felső végén beállítani.</span><span class="sxs-lookup"><span data-stu-id="723ac-152">A good practice is to set the address space for the gateway subnet at the upper end of the VNet address space.</span></span> <span data-ttu-id="723ac-153">Az ábrán is látható példa a 10.20.255.224/27 címteret használja.</span><span class="sxs-lookup"><span data-stu-id="723ac-153">The example shown in the diagram uses 10.20.255.224/27.</span></span>  <span data-ttu-id="723ac-154">Íme egy gyors eljárás a [CIDR] kiszámításához:</span><span class="sxs-lookup"><span data-stu-id="723ac-154">Here is a quick procedure to calculate the [CIDR]:</span></span>

1. <span data-ttu-id="723ac-155">A virtuális hálózat címterében lévő változó biteket állítsa 1-re, egészen az átjáró-alhálózat által használt bitekig, majd a többit állítsa 0-ra.</span><span class="sxs-lookup"><span data-stu-id="723ac-155">Set the variable bits in the address space of the VNet to 1, up to the bits being used by the gateway subnet, then set the remaining bits to 0.</span></span>
2. <span data-ttu-id="723ac-156">Alakítsa át az eredményül kapott biteket decimálisra, és fejezze ki címtérként, az előtag hosszúságát pedig állítsa az átjáró-alhálózat hosszúságára.</span><span class="sxs-lookup"><span data-stu-id="723ac-156">Convert the resulting bits to decimal and express it as an address space with the prefix length set to the size of the gateway subnet.</span></span>

<span data-ttu-id="723ac-157">Egy 10.20.0.0/16 IP-címtartományú virtuális hálózat esetében például az 1. lépés alkalmazása a következő eredményt adja: 10.20.0b11111111.0b11100000.</span><span class="sxs-lookup"><span data-stu-id="723ac-157">For example, for a VNet with an IP address range of 10.20.0.0/16, applying step #1 above becomes 10.20.0b11111111.0b11100000.</span></span>  <span data-ttu-id="723ac-158">Ha ezt átváltja decimálisra, és címtérként fejezi ki, a következőt kapja: 10.20.255.224/27.</span><span class="sxs-lookup"><span data-stu-id="723ac-158">Converting that to decimal and expressing it as an address space yields 10.20.255.224/27.</span></span> 

> [!WARNING]
> <span data-ttu-id="723ac-159">Ne telepítsen virtuális gépet az átjáró-alhálózatba.</span><span class="sxs-lookup"><span data-stu-id="723ac-159">Do not deploy any VMs to the gateway subnet.</span></span> <span data-ttu-id="723ac-160">Arra is ügyeljen, hogy ne rendeljen NSG-t ehhez az alhálózathoz, különben az átjáró nem fog működni.</span><span class="sxs-lookup"><span data-stu-id="723ac-160">Also, do not assign an NSG to this subnet, as it will cause the gateway to stop functioning.</span></span>
> 
> 

### <a name="virtual-network-gateway"></a><span data-ttu-id="723ac-161">Virtuális hálózati átjáró</span><span class="sxs-lookup"><span data-stu-id="723ac-161">Virtual network gateway</span></span>

<span data-ttu-id="723ac-162">Foglaljon le egy nyilvános IP-címet a virtuális hálózati átjáróhoz.</span><span class="sxs-lookup"><span data-stu-id="723ac-162">Allocate a public IP address for the virtual network gateway.</span></span>

<span data-ttu-id="723ac-163">Hozza létre a virtuális hálózati átjárót az átjáró-alhálózatban, és rendelje hozzá az újonnan lefoglalt nyilvános IP-címet.</span><span class="sxs-lookup"><span data-stu-id="723ac-163">Create the virtual network gateway in the gateway subnet and assign it the newly allocated public IP address.</span></span> <span data-ttu-id="723ac-164">Használja az igényeinek leginkább megfelelő, és a VPN-berendezés által engedélyezett átjárótípust:</span><span class="sxs-lookup"><span data-stu-id="723ac-164">Use the gateway type that most closely matches your requirements and that is enabled by your VPN appliance:</span></span>

- <span data-ttu-id="723ac-165">[Házirendalapú átjárót][policy-based-routing] hozzon létre, ha házirendfeltételek, például címelőtagok alapján szoros felügyeletet szeretne gyakorolni a kérelmek útválasztása fölött.</span><span class="sxs-lookup"><span data-stu-id="723ac-165">Create a [policy-based gateway][policy-based-routing] if you need to closely control how requests are routed based on policy criteria such as address prefixes.</span></span> <span data-ttu-id="723ac-166">A házirendalapú átjárók statikus útválasztást használnak, és csak a helyek közötti kapcsolatoknál használhatók.</span><span class="sxs-lookup"><span data-stu-id="723ac-166">Policy-based gateways use static routing, and only work with site-to-site connections.</span></span>

- <span data-ttu-id="723ac-167">[Útvonalalapú átjárót][route-based-routing] hozzon létre, ha RRAS használatával csatlakozik a helyszíni hálózathoz, támogatja a többhelyes vagy régiók közötti kapcsolatokat, vagy virtuális hálózatok közötti kapcsolatokat implementál (beleértve a több virtuális hálózaton áthaladó útvonalakat).</span><span class="sxs-lookup"><span data-stu-id="723ac-167">Create a [route-based gateway][route-based-routing] if you connect to the on-premises network using RRAS, support multi-site or cross-region connections, or implement VNet-to-VNet connections (including routes that traverse multiple VNets).</span></span> <span data-ttu-id="723ac-168">Az útválasztó-alapú átjárók dinamikus útválasztást használnak a hálózatok között.</span><span class="sxs-lookup"><span data-stu-id="723ac-168">Route-based gateways use dynamic routing to direct traffic between networks.</span></span> <span data-ttu-id="723ac-169">Ezek jobban tűrik a hálózati útvonal hibáit, mint a statikus útvonalak, mert próbálkozhatnak más útvonalakkal.</span><span class="sxs-lookup"><span data-stu-id="723ac-169">They can tolerate failures in the network path better than static routes because they can try alternative routes.</span></span> <span data-ttu-id="723ac-170">Az útválasztó-alapú átjárók is csökkenthetik a felügyeleti terheket, mert az útvonalakat nem feltétlenül kell manuálisan frissíteni, ha változik a hálózati cím.</span><span class="sxs-lookup"><span data-stu-id="723ac-170">Route-based gateways can also reduce the management overhead because routes might not need to be updated manually when network addresses change.</span></span>

<span data-ttu-id="723ac-171">A támogatott VPN-berendezések listáját az [Információk a helyek közötti VPN Gateway-kapcsolatok VPN-eszközeiről][vpn-appliances] cikkben találja meg.</span><span class="sxs-lookup"><span data-stu-id="723ac-171">For a list of supported VPN appliances, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliances].</span></span>

> [!NOTE]
> <span data-ttu-id="723ac-172">Az átjáró létrehozása után az átjárótípusok között csak az átjáró törlésével és újbóli létrehozásával válthat.</span><span class="sxs-lookup"><span data-stu-id="723ac-172">After the gateway has been created, you cannot change between gateway types without deleting and re-creating the gateway.</span></span>
> 
> 

<span data-ttu-id="723ac-173">Válassza ki az Azure VPN-átjáró azon termékváltozatát, amely leginkább megfelel a teljesítménybeli követelményeknek.</span><span class="sxs-lookup"><span data-stu-id="723ac-173">Select the Azure VPN gateway SKU that most closely matches your throughput requirements.</span></span> <span data-ttu-id="723ac-174">További informayion, lásd: [átjáró-termékváltozatok][azure-gateway-skus]</span><span class="sxs-lookup"><span data-stu-id="723ac-174">For more informayion, see [Gateway SKUs][azure-gateway-skus]</span></span>

> [!NOTE]
> <span data-ttu-id="723ac-175">Az alapszintű SKU nem kompatibilis az Azure ExpressRoute-tal.</span><span class="sxs-lookup"><span data-stu-id="723ac-175">The Basic SKU is not compatible with Azure ExpressRoute.</span></span> <span data-ttu-id="723ac-176">A [termékváltozat][changing-SKUs] az átjáró létrehozása után is módosítható.</span><span class="sxs-lookup"><span data-stu-id="723ac-176">You can [change the SKU][changing-SKUs] after the gateway has been created.</span></span>
> 
> 

<span data-ttu-id="723ac-177">A díjat az átjáró üzemképes állapotának és rendelkezésre állásának időtartama számoljuk fel.</span><span class="sxs-lookup"><span data-stu-id="723ac-177">You are charged based on the amount of time that the gateway is provisioned and available.</span></span> <span data-ttu-id="723ac-178">Információk: [A VPN Gateway díjszabása][azure-gateway-charges].</span><span class="sxs-lookup"><span data-stu-id="723ac-178">See [VPN Gateway Pricing][azure-gateway-charges].</span></span>

<span data-ttu-id="723ac-179">Ahelyett, hogy közvetlenül engedné áthaladni a kérelmeket az alkalmazás virtuális gépeire, hozzon létre útválasztási szabályokat a bejövő alkalmazásforgalmat az átjáróról a belső terheléselosztóra irányító átjáró-alhálózathoz.</span><span class="sxs-lookup"><span data-stu-id="723ac-179">Create routing rules for the gateway subnet that direct incoming application traffic from the gateway to the internal load balancer, rather than allowing requests to pass directly to the application VMs.</span></span>

### <a name="on-premises-network-connection"></a><span data-ttu-id="723ac-180">Helyszíni hálózati kapcsolat</span><span class="sxs-lookup"><span data-stu-id="723ac-180">On-premises network connection</span></span>

<span data-ttu-id="723ac-181">Hozzon létre egy helyi hálózati átjárót.</span><span class="sxs-lookup"><span data-stu-id="723ac-181">Create a local network gateway.</span></span> <span data-ttu-id="723ac-182">Adja meg a helyszíni VPN-berendezés nyilvános IP-címét és a helyszíni hálózat címterét.</span><span class="sxs-lookup"><span data-stu-id="723ac-182">Specify the public IP address of the on-premises VPN appliance, and the address space of the on-premises network.</span></span> <span data-ttu-id="723ac-183">Vegye figyelembe, hogy a helyszíni VPN-berendezésnek nyilvános IP-címmel kell rendelkeznie ahhoz, hogy az Azure VPN Gateway helyi hálózati átjárója hozzáférhessen.</span><span class="sxs-lookup"><span data-stu-id="723ac-183">Note that the on-premises VPN appliance must have a public IP address that can be accessed by the local network gateway in Azure VPN Gateway.</span></span> <span data-ttu-id="723ac-184">A VPN-eszköz nem helyezhető egy hálózati címfordítás (NAT) mögé.</span><span class="sxs-lookup"><span data-stu-id="723ac-184">The VPN device cannot be located behind a network address translation (NAT) device.</span></span>

<span data-ttu-id="723ac-185">Hozzon létre egy helyek közötti kapcsolatot a virtuális hálózati átjáróhoz és a helyi hálózati átjáróhoz.</span><span class="sxs-lookup"><span data-stu-id="723ac-185">Create a site-to-site connection for the virtual network gateway and the local network gateway.</span></span> <span data-ttu-id="723ac-186">Válassza ki a helyek közötti (IPSec) kapcsolattípust, és adja meg a megosztott kulcsot.</span><span class="sxs-lookup"><span data-stu-id="723ac-186">Select the site-to-site (IPSec) connection type, and specify the shared key.</span></span> <span data-ttu-id="723ac-187">Az Azure VPN-átjáróval végzett helyek közötti titkosítás az IPSec protokollon alapul, és előre megosztott kulcsokat használ a hitelesítéshez.</span><span class="sxs-lookup"><span data-stu-id="723ac-187">Site-to-site encryption with the Azure VPN gateway is based on the IPSec protocol, using preshared keys for authentication.</span></span> <span data-ttu-id="723ac-188">A kulcsot az Azure VPN-átjáró létrehozásakor adja meg.</span><span class="sxs-lookup"><span data-stu-id="723ac-188">You specify the key when you create the Azure VPN gateway.</span></span> <span data-ttu-id="723ac-189">A helyszínen futó VPN-berendezést ugyanazzal a kulccsal kell konfigurálni.</span><span class="sxs-lookup"><span data-stu-id="723ac-189">You must configure the VPN appliance running on-premises with the same key.</span></span> <span data-ttu-id="723ac-190">Más hitelesítési mechanizmusok jelenleg nem támogatottak.</span><span class="sxs-lookup"><span data-stu-id="723ac-190">Other authentication mechanisms are not currently supported.</span></span>

<span data-ttu-id="723ac-191">Győződjön meg arról, hogy a helyszíni útválasztási infrastruktúra úgy van beállítva, hogy az Azure virtuális hálózatban lévő címekre szánt kérelmeket továbbítsa a VPN-eszközre.</span><span class="sxs-lookup"><span data-stu-id="723ac-191">Ensure that the on-premises routing infrastructure is configured to forward requests intended for addresses in the Azure VNet to the VPN device.</span></span>

<span data-ttu-id="723ac-192">Nyissa meg a felhőalkalmazás által igényelt portokat a helyszíni hálózatban.</span><span class="sxs-lookup"><span data-stu-id="723ac-192">Open any ports required by the cloud application in the on-premises network.</span></span>

<span data-ttu-id="723ac-193">A kapcsolatot tesztelve ellenőrizze a következőket:</span><span class="sxs-lookup"><span data-stu-id="723ac-193">Test the connection to verify that:</span></span>

* <span data-ttu-id="723ac-194">A helyszíni VPN-berendezés megfelelően irányítja-e a forgalmat a felhőalkalmazásba az Azure VPN-átjárón keresztül.</span><span class="sxs-lookup"><span data-stu-id="723ac-194">The on-premises VPN appliance correctly routes traffic to the cloud application through the Azure VPN gateway.</span></span>
* <span data-ttu-id="723ac-195">A virtuális hálózat megfelelően visszairányítja-e a forgalmat a helyszíni hálózatba.</span><span class="sxs-lookup"><span data-stu-id="723ac-195">The VNet correctly routes traffic back to the on-premises network.</span></span>
* <span data-ttu-id="723ac-196">A tiltott forgalom mindkét irányba megfelelően blokkolva van-e.</span><span class="sxs-lookup"><span data-stu-id="723ac-196">Prohibited traffic in both directions is blocked correctly.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="723ac-197">Méretezési szempontok</span><span class="sxs-lookup"><span data-stu-id="723ac-197">Scalability considerations</span></span>

<span data-ttu-id="723ac-198">Az alapszintű vagy standard VPN Gateway termékváltozatoktól a nagy teljesítményű VPN termékváltozatra áttérve korlátozott függőleges méretezhetőséget érhet el.</span><span class="sxs-lookup"><span data-stu-id="723ac-198">You can achieve limited vertical scalability by moving from the Basic or Standard VPN Gateway SKUs to the High Performance VPN SKU.</span></span>

<span data-ttu-id="723ac-199">A várhatóan nagy mennyiségű VPN-forgalmat bonyolító virtuális hálózatok esetében érdemes lehet elosztani az eltérő számítási feladatokat külön kisebb virtuális hálózatokra, és mindegyikhez konfigurálni egy VPN-átjárót.</span><span class="sxs-lookup"><span data-stu-id="723ac-199">For VNets that expect a large volume of VPN traffic, consider distributing the different workloads into separate smaller VNets and configuring a VPN gateway for each of them.</span></span>

<span data-ttu-id="723ac-200">A virtuális hálózatot particionálhatja vízszintesen vagy függőlegesen is.</span><span class="sxs-lookup"><span data-stu-id="723ac-200">You can partition the VNet either horizontally or vertically.</span></span> <span data-ttu-id="723ac-201">A vízszintes particionáláshoz helyezzen át virtuálisgép-példányokat az egyes szintekről az új virtuális hálózat alhálózataiba.</span><span class="sxs-lookup"><span data-stu-id="723ac-201">To partition horizontally, move some VM instances from each tier into subnets of the new VNet.</span></span> <span data-ttu-id="723ac-202">Ennek eredményképp minden virtuális hálózat struktúrája és működése azonos lesz.</span><span class="sxs-lookup"><span data-stu-id="723ac-202">The result is that each VNet has the same structure and functionality.</span></span> <span data-ttu-id="723ac-203">A függőleges particionáláshoz tervezze át az egyes szinteket úgy, hogy a funkcionalitás különböző logikai területekre oszoljon (például megrendelések kezelése, számlázás, ügyfélfiókok kezelése stb.).</span><span class="sxs-lookup"><span data-stu-id="723ac-203">To partition vertically, redesign each tier to divide the functionality into different logical areas (such as handling orders, invoicing, customer account management, and so on).</span></span> <span data-ttu-id="723ac-204">Ezután minden funkcionális terület elhelyezhető a saját virtuális hálózatában.</span><span class="sxs-lookup"><span data-stu-id="723ac-204">Each functional area can then be placed in its own VNet.</span></span>

<span data-ttu-id="723ac-205">Ha replikál egy helyszíni Active Directory-tartományvezérlőt a virtuális hálózatban, és implementál egy DNS-t a virtuális hálózatban, azzal csökkentheti a helyszín és a felhő közötti, biztonsággal kapcsolatos és adminisztratív forgalmat.</span><span class="sxs-lookup"><span data-stu-id="723ac-205">Replicating an on-premises Active Directory domain controller in the VNet, and implementing DNS in the VNet, can help to reduce some of the security-related and administrative traffic flowing from on-premises to the cloud.</span></span> <span data-ttu-id="723ac-206">További információk: [Az Azure Active Directory Domain Services (AD DS) kiterjesztése az Azure-ra][adds-extend-domain].</span><span class="sxs-lookup"><span data-stu-id="723ac-206">For more information, see [Extending Active Directory Domain Services (AD DS) to Azure][adds-extend-domain].</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="723ac-207">Rendelkezésre állási szempontok</span><span class="sxs-lookup"><span data-stu-id="723ac-207">Availability considerations</span></span>

<span data-ttu-id="723ac-208">Ha biztosítania kell, hogy az Azure VPN-átjáró folyamatosan el tudja érni a helyszíni hálózatot, implementáljon egy feladatátvételi fürtöt a helyszíni VPN-átjáróhoz.</span><span class="sxs-lookup"><span data-stu-id="723ac-208">If you need to ensure that the on-premises network remains available to the Azure VPN gateway, implement a failover cluster for the on-premises VPN gateway.</span></span>

<span data-ttu-id="723ac-209">Ha a vállalat több helyszíni hellyel rendelkezik, hozzon létre [többhelyes kapcsolatokat][vpn-gateway-multi-site] egy vagy több Azure virtuális hálózathoz.</span><span class="sxs-lookup"><span data-stu-id="723ac-209">If your organization has multiple on-premises sites, create [multi-site connections][vpn-gateway-multi-site] to one or more Azure VNets.</span></span> <span data-ttu-id="723ac-210">Ehhez a megközelítéshez dinamikus (útvonalalapú) útválasztásra van szükség, ezért ügyeljen arra, hogy a helyszíni VPN-átjáró támogassa ezt a szolgáltatást.</span><span class="sxs-lookup"><span data-stu-id="723ac-210">This approach requires dynamic (route-based) routing, so make sure that the on-premises VPN gateway supports this feature.</span></span>

<span data-ttu-id="723ac-211">További információk a szolgáltatói szerződésekről: [A VPN Gateway szolgáltatói szerződése][sla-for-vpn-gateway].</span><span class="sxs-lookup"><span data-stu-id="723ac-211">For details about service level agreements, see [SLA for VPN Gateway][sla-for-vpn-gateway].</span></span> 

## <a name="manageability-considerations"></a><span data-ttu-id="723ac-212">Felügyeleti szempontok</span><span class="sxs-lookup"><span data-stu-id="723ac-212">Manageability considerations</span></span>

<span data-ttu-id="723ac-213">Monitorozza a helyszíni VPN-berendezésekről származó diagnosztikai információkat.</span><span class="sxs-lookup"><span data-stu-id="723ac-213">Monitor diagnostic information from on-premises VPN appliances.</span></span> <span data-ttu-id="723ac-214">Ez a folyamat a VPN-berendezés által megadott szolgáltatásoktól függ.</span><span class="sxs-lookup"><span data-stu-id="723ac-214">This process depends on the features provided by the VPN appliance.</span></span> <span data-ttu-id="723ac-215">Ha például a Windows Server 2012 útválasztási és távelérési szolgáltatását használja, akkor az [RRAS naplózástól][rras-logging].</span><span class="sxs-lookup"><span data-stu-id="723ac-215">For example, if you are using the Routing and Remote Access Service on Windows Server 2012, [RRAS logging][rras-logging].</span></span>

<span data-ttu-id="723ac-216">A kapcsolati problémákra vonatkozó információk rögzítéséhez használja az [Azure VPN-átjáró-diagnosztikát][gateway-diagnostic-logs].</span><span class="sxs-lookup"><span data-stu-id="723ac-216">Use [Azure VPN gateway diagnostics][gateway-diagnostic-logs] to capture information about connectivity issues.</span></span> <span data-ttu-id="723ac-217">Ezen naplók segítségével olyan információk követhetők nyomon például, mint a kapcsolati kérelmek forrása és célja, a használt protokoll típusa és a kapcsolat létrehozásának módja (vagy a meghiúsulás oka).</span><span class="sxs-lookup"><span data-stu-id="723ac-217">These logs can be used to track information such as the source and destinations of connection requests, which protocol was used, and how the connection was established (or why the attempt failed).</span></span>

<span data-ttu-id="723ac-218">Monitorozza az Azure VPN-átjáró működési naplóit az Azure Portalon elérhető auditnaplók segítségével.</span><span class="sxs-lookup"><span data-stu-id="723ac-218">Monitor the operational logs of the Azure VPN gateway using the audit logs available in the Azure portal.</span></span> <span data-ttu-id="723ac-219">A helyi hálózati átjáróhoz, az Azure hálózati átjáróhoz és a kapcsolathoz külön naplók érhetők el.</span><span class="sxs-lookup"><span data-stu-id="723ac-219">Separate logs are available for the local network gateway, the Azure network gateway, and the connection.</span></span> <span data-ttu-id="723ac-220">Ezzel az információval nyomon követhetők az átjáró módosításai, és hasznosak lehetnek, ha egy korábban funkcionáló átjáró működése valamiért leáll.</span><span class="sxs-lookup"><span data-stu-id="723ac-220">This information can be used to track any changes made to the gateway, and can be useful if a previously functioning gateway stops working for some reason.</span></span>

<span data-ttu-id="723ac-221">![[2]][2]</span><span class="sxs-lookup"><span data-stu-id="723ac-221">![[2]][2]</span></span>

<span data-ttu-id="723ac-222">Monitorozza a kapcsolatokat, és kövesse nyomon a hibaeseményeket.</span><span class="sxs-lookup"><span data-stu-id="723ac-222">Monitor connectivity, and track connectivity failure events.</span></span> <span data-ttu-id="723ac-223">Az információ rögzítéséhez és jelentéséhez használhat olyan monitorozási csomagokat, mint amilyen például a [Nagios][nagios].</span><span class="sxs-lookup"><span data-stu-id="723ac-223">You can use a monitoring package such as [Nagios][nagios] to capture and report this information.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="723ac-224">Biztonsági szempontok</span><span class="sxs-lookup"><span data-stu-id="723ac-224">Security considerations</span></span>

<span data-ttu-id="723ac-225">Minden VPN-átjáróhoz hozzon létre különböző megosztott kulcsot.</span><span class="sxs-lookup"><span data-stu-id="723ac-225">Generate a different shared key for each VPN gateway.</span></span> <span data-ttu-id="723ac-226">Egy erős megosztott kulccsal könnyebb ellenállni a találgatásos támadásoknak.</span><span class="sxs-lookup"><span data-stu-id="723ac-226">Use a strong shared key to help resist brute-force attacks.</span></span>

> [!NOTE]
> <span data-ttu-id="723ac-227">Az Azure Key Vault jelenleg nem használható az Azure VPN-átjáró kulcsainak előzetes megosztására.</span><span class="sxs-lookup"><span data-stu-id="723ac-227">Currently, you cannot use Azure Key Vault to preshare keys for the Azure VPN gateway.</span></span>
> 
> 

<span data-ttu-id="723ac-228">Győződjön meg arról, hogy a helyszíni VPN-készülék [az Azure VPN-átjáróval kompatibilis][vpn-appliance-ipsec] titkosítási módszert alkalmaz.</span><span class="sxs-lookup"><span data-stu-id="723ac-228">Ensure that the on-premises VPN appliance uses an encryption method that is [compatible with the Azure VPN gateway][vpn-appliance-ipsec].</span></span> <span data-ttu-id="723ac-229">A házirendalapú útválasztás esetében az Azure VPN-átjáró az AES256, az AES128 és a 3DES titkosítási algoritmust támogatja.</span><span class="sxs-lookup"><span data-stu-id="723ac-229">For policy-based routing, the Azure VPN gateway supports the AES256, AES128, and 3DES encryption algorithms.</span></span> <span data-ttu-id="723ac-230">Az útvonalalapú átjárók az AES256 és 3DES titkosítási algoritmust támogatják.</span><span class="sxs-lookup"><span data-stu-id="723ac-230">Route-based gateways support AES256 and 3DES.</span></span>

<span data-ttu-id="723ac-231">Ha a helyszíni VPN-berendezés olyan szegélyhálózaton (DMZ) található, amely tűzfalat tart fenn a szegélyhálózat és az internet között, akkor előfordulhat, hogy [további tűzfalszabályokat][additional-firewall-rules] kell konfigurálnia a helyek közötti VPN-kapcsolat engedélyezéséhez.</span><span class="sxs-lookup"><span data-stu-id="723ac-231">If your on-premises VPN appliance is on a perimeter network (DMZ) that has a firewall between the perimeter network and the Internet, you might have to configure [additional firewall rules][additional-firewall-rules] to allow the site-to-site VPN connection.</span></span>

<span data-ttu-id="723ac-232">Ha a virtuális hálózaton lévő alkalmazás adatokat küld az internetre, érdemes lehet [kényszerített bújtatást implementálni][forced-tunneling] és az internetre irányuló összes forgalmat a helyszíni hálózatra irányítani.</span><span class="sxs-lookup"><span data-stu-id="723ac-232">If the application in the VNet sends data to the Internet, consider [implementing forced tunneling][forced-tunneling] to route all Internet-bound traffic through the on-premises network.</span></span> <span data-ttu-id="723ac-233">Ez a megközelítés lehetővé teszi az alkalmazás által a helyszíni infrastruktúráról kimenő kérelmek naplózását.</span><span class="sxs-lookup"><span data-stu-id="723ac-233">This approach enables you to audit outgoing requests made by the application from the on-premises infrastructure.</span></span>

> [!NOTE]
> <span data-ttu-id="723ac-234">A kényszerített bújtatás hatással lehet az Azure-szolgáltatásokhoz (például a Storage Service-hez) és a Windows licenckezelőhöz való kapcsolódásra.</span><span class="sxs-lookup"><span data-stu-id="723ac-234">Forced tunneling can impact connectivity to Azure services (the Storage Service, for example) and the Windows license manager.</span></span>
> 
> 


## <a name="troubleshooting"></a><span data-ttu-id="723ac-235">Hibaelhárítás</span><span class="sxs-lookup"><span data-stu-id="723ac-235">Troubleshooting</span></span> 

<span data-ttu-id="723ac-236">Általános információk a VPN-nel kapcsolatos gyakori hibák elhárításáról: [VPN-nel kapcsolatos gyakori hibák elhárítása][troubleshooting-vpn-errors].</span><span class="sxs-lookup"><span data-stu-id="723ac-236">For general information on troubleshooting common VPN-related errors, see [Troubleshooting common VPN related errors][troubleshooting-vpn-errors].</span></span>

<span data-ttu-id="723ac-237">Az alábbi javaslatok segítenek meghatározni, hogy a helyszíni VPN-berendezés megfelelően működik-e.</span><span class="sxs-lookup"><span data-stu-id="723ac-237">The following recommendations are useful for determining if your on-premises VPN appliance is functioning correctly.</span></span>

- <span data-ttu-id="723ac-238">**Ellenőrizze a VPN-berendezés által létrehozott naplófájlokat hibák és meghibásodások vonatkozásában.**</span><span class="sxs-lookup"><span data-stu-id="723ac-238">**Check any log files generated by the VPN appliance for errors or failures.**</span></span>

    <span data-ttu-id="723ac-239">Ez segít meghatározni, hogy a VPN-berendezés helyesen működik-e.</span><span class="sxs-lookup"><span data-stu-id="723ac-239">This will help you determine if the VPN appliance is functioning correctly.</span></span> <span data-ttu-id="723ac-240">Ennek az információnak a helye a berendezéstől függően változik.</span><span class="sxs-lookup"><span data-stu-id="723ac-240">The location of this information will vary according to your appliance.</span></span> <span data-ttu-id="723ac-241">Ha például a Windows Server 2012 RRAS szolgáltatását használja, az alábbi PowerShell-paranccsal jelenítheti meg az RRAS szolgáltatással kapcsolatos hibaeseményekre vonatkozó információkat:</span><span class="sxs-lookup"><span data-stu-id="723ac-241">For example, if you are using RRAS on Windows Server 2012, you can use the following PowerShell command to display error event information for the RRAS service:</span></span>

    ```PowerShell
    Get-EventLog -LogName System -EntryType Error -Source RemoteAccess | Format-List -Property *
    ```

    <span data-ttu-id="723ac-242">Az egyes bejegyzések *Üzenet* tulajdonsága tartalmaz hibaleírást.</span><span class="sxs-lookup"><span data-stu-id="723ac-242">The *Message* property of each entry provides a description of the error.</span></span> <span data-ttu-id="723ac-243">Néhány gyakori példa:</span><span class="sxs-lookup"><span data-stu-id="723ac-243">Some common examples are:</span></span>

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

    <span data-ttu-id="723ac-244">Az alábbi PowerShell-paranccsal lekérheti a RRAS szolgáltatáson keresztül megkísérelt csatlakozásra vonatkozó eseménynapló-adatokat is:</span><span class="sxs-lookup"><span data-stu-id="723ac-244">You can also obtain event log information about attempts to connect through the RRAS service using the following PowerShell command:</span></span> 

    ```
    Get-EventLog -LogName Application -Source RasClient | Format-List -Property *
    ```

    <span data-ttu-id="723ac-245">Ha nem sikerül a csatlakozás, ez a napló az alábbihoz hasonló hibákat fog tartalmazni:</span><span class="sxs-lookup"><span data-stu-id="723ac-245">In the event of a failure to connect, this log will contain errors that look similar to the following:</span></span>

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

- <span data-ttu-id="723ac-246">**Ellenőrizze a kapcsolatot és az útválasztást a VPN-átjárón.**</span><span class="sxs-lookup"><span data-stu-id="723ac-246">**Verify connectivity and routing across the VPN gateway.**</span></span>

    <span data-ttu-id="723ac-247">Előfordulhat, hogy a VPN-berendezés nem megfelelően irányítja a forgalmat az Azure VPN Gatewayen keresztül.</span><span class="sxs-lookup"><span data-stu-id="723ac-247">The VPN appliance may not be correctly routing traffic through the Azure VPN Gateway.</span></span> <span data-ttu-id="723ac-248">Egy [PsPing][psping] eszközhöz hasonló eszközzel ellenőrizheti a kapcsolatot és az útválasztást a VPN-átjárón.</span><span class="sxs-lookup"><span data-stu-id="723ac-248">Use a tool such as [PsPing][psping] to verify connectivity and routing across the VPN gateway.</span></span> <span data-ttu-id="723ac-249">Ha például tesztelni szeretné a csatlakozást egy helyszíni gép és egy, a virtuális hálózaton található webkiszolgáló között, futtassa az alábbi parancsot (a `<<web-server-address>>` helyére írja a webkiszolgáló címét):</span><span class="sxs-lookup"><span data-stu-id="723ac-249">For example, to test connectivity from an on-premises machine to a web server located on the VNet, run the following command (replacing `<<web-server-address>>` with the address of the web server):</span></span>

    ```
    PsPing -t <<web-server-address>>:80
    ```

    <span data-ttu-id="723ac-250">Ha a helyszíni gép át tudja irányítani a forgalmat a webkiszolgálóra, az alábbihoz hasonló kimenetnek kell megjelennie:</span><span class="sxs-lookup"><span data-stu-id="723ac-250">If the on-premises machine can route traffic to the web server, you should see output similar to the following:</span></span>

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

    <span data-ttu-id="723ac-251">Ha a helyi számítógép nem tud kommunikálni a megadott céllal, az alábbihoz hasonló üzenetek jelennek meg:</span><span class="sxs-lookup"><span data-stu-id="723ac-251">If the on-premises machine cannot communicate with the specified destination, you will see messages like this:</span></span>

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

- <span data-ttu-id="723ac-252">**Ellenőrizze, hogy a helyszíni tűzfal engedélyezi-e a VPN-forgalom áthaladását, és hogy a megfelelő portok vannak-e megnyitva.**</span><span class="sxs-lookup"><span data-stu-id="723ac-252">**Verify that the on-premises firewall allows VPN traffic to pass and that the correct ports are opened.**</span></span>

- <span data-ttu-id="723ac-253">**Ellenőrizze, hogy a helyszíni VPN-berendezés az [Azure VPN-átjáróval][vpn-appliance] kompatibilis titkosítási módszert használja-e.**</span><span class="sxs-lookup"><span data-stu-id="723ac-253">**Verify that the on-premises VPN appliance uses an encryption method that is [compatible with the Azure VPN gateway][vpn-appliance].**</span></span> <span data-ttu-id="723ac-254">A házirendalapú útválasztás esetében az Azure VPN-átjáró az AES256, az AES128 és a 3DES titkosítási algoritmust támogatja.</span><span class="sxs-lookup"><span data-stu-id="723ac-254">For policy-based routing, the Azure VPN gateway supports the AES256, AES128, and 3DES encryption algorithms.</span></span> <span data-ttu-id="723ac-255">Az útvonalalapú átjárók az AES256 és 3DES titkosítási algoritmust támogatják.</span><span class="sxs-lookup"><span data-stu-id="723ac-255">Route-based gateways support AES256 and 3DES.</span></span>

<span data-ttu-id="723ac-256">Az alábbi javaslatok segítenek megállapítani, van-e probléma az Azure VPN-átjáróval:</span><span class="sxs-lookup"><span data-stu-id="723ac-256">The following recommendations are useful for determining if there is a problem with the Azure VPN gateway:</span></span>

- <span data-ttu-id="723ac-257">**Nézze át az [Azure VPN-átjáró diagnosztikai naplóit][gateway-diagnostic-logs], hogy nincs-e nyoma problémának.**</span><span class="sxs-lookup"><span data-stu-id="723ac-257">**Examine [Azure VPN gateway diagnostic logs][gateway-diagnostic-logs] for potential issues.**</span></span>

- <span data-ttu-id="723ac-258">**Ellenőrizze, hogy az Azure VPN-átjáró és a helyszíni VPN-berendezés ugyanazt a megosztott hitelesítési kulcsot használja-e.**</span><span class="sxs-lookup"><span data-stu-id="723ac-258">**Verify that the Azure VPN gateway and on-premises VPN appliance are configured with the same shared authentication key.**</span></span>

    <span data-ttu-id="723ac-259">Az Azure VPN-átjáró által tárolt megosztott kulcsot az alábbi Azure CLI-paranccsal tekintheti meg:</span><span class="sxs-lookup"><span data-stu-id="723ac-259">You can view the shared key stored by the Azure VPN gateway using the following Azure CLI command:</span></span>

    ```
    azure network vpn-connection shared-key show <<resource-group>> <<vpn-connection-name>>
    ```

    <span data-ttu-id="723ac-260">Használja a helyszíni VPN-berendezéshez megfelelő parancsot a berendezéshez konfigurált megosztott kulcs megjelenítéséhez.</span><span class="sxs-lookup"><span data-stu-id="723ac-260">Use the command appropriate for your on-premises VPN appliance to show the shared key configured for that appliance.</span></span>

    <span data-ttu-id="723ac-261">Győződjön meg arról, hogy az Azure VPN-átjárónak helyet adó *GatewaySubnet* alhálózat nincs NSG-hez társítva.</span><span class="sxs-lookup"><span data-stu-id="723ac-261">Verify that the *GatewaySubnet* subnet holding the Azure VPN gateway is not associated with an NSG.</span></span>

    <span data-ttu-id="723ac-262">Az alhálózati adatokat a következő Azure CLI-paranccsal tekintheti meg:</span><span class="sxs-lookup"><span data-stu-id="723ac-262">You can view the subnet details using the following Azure CLI command:</span></span>

    ```
    azure network vnet subnet show -g <<resource-group>> -e <<vnet-name>> -n GatewaySubnet
    ```

    <span data-ttu-id="723ac-263">Ügyeljen arra, hogy ne legyen *Hálózati biztonsági csoport azonosítója* nevű adatmező. Az alábbi példa bemutatja egy olyan *GatewaySubnet*-példány eredményeit, amelyhez hozzá van rendelve egy NSG (*VPN-Gateway-Group*).</span><span class="sxs-lookup"><span data-stu-id="723ac-263">Ensure there is no data field named *Network Security Group id*. The following example shows the results for an instance of the *GatewaySubnet* that has an assigned NSG (*VPN-Gateway-Group*).</span></span> <span data-ttu-id="723ac-264">Emiatt előfordulhat, hogy az átjáró hibásan működik, ha vannak szabályok meghatározva az NSG-hez.</span><span class="sxs-lookup"><span data-stu-id="723ac-264">This can prevent the gateway from working correctly if there are any rules defined for this NSG.</span></span>

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

- <span data-ttu-id="723ac-265">**Ellenőrizze, hogy az Azure virtuális hálózat virtuális gépeinek konfigurációja engedélyezi-e a virtuális hálózaton kívülről érkező forgalmat.**</span><span class="sxs-lookup"><span data-stu-id="723ac-265">**Verify that the virtual machines in the Azure VNet are configured to permit traffic coming in from outside the VNet.**</span></span>

    <span data-ttu-id="723ac-266">Ellenőrizze az összes, a virtuális gépeket tartalmazó alhálózatokhoz társított NSG-szabályt.</span><span class="sxs-lookup"><span data-stu-id="723ac-266">Check any NSG rules associated with subnets containing these virtual machines.</span></span> <span data-ttu-id="723ac-267">Az alábbi Azure CLI-paranccsal megtekintheti az összes NSG-szabályt:</span><span class="sxs-lookup"><span data-stu-id="723ac-267">You can view all NSG rules using the following Azure CLI command:</span></span>

    ```
    azure network nsg show -g <<resource-group>> -n <<nsg-name>>
    ```

- <span data-ttu-id="723ac-268">**Ellenőrizze, hogy csatlakoztatva van-e az Azure VPN-átjáró.**</span><span class="sxs-lookup"><span data-stu-id="723ac-268">**Verify that the Azure VPN gateway is connected.**</span></span>

    <span data-ttu-id="723ac-269">A következő Azure PowerShell-paranccsal ellenőrizheti az Azure VPN-kapcsolat aktuális állapotát.</span><span class="sxs-lookup"><span data-stu-id="723ac-269">You can use the following Azure PowerShell command to check the current status of the Azure VPN connection.</span></span> <span data-ttu-id="723ac-270">A `<<connection-name>>` paraméter a virtuális hálózati átjárót és a helyi átjárót társító Azure VPN-kapcsolat neve.</span><span class="sxs-lookup"><span data-stu-id="723ac-270">The `<<connection-name>>` parameter is the name of the Azure VPN connection that links the virtual network gateway and the local gateway.</span></span>

    ```
    Get-AzureRmVirtualNetworkGatewayConnection -Name <<connection-name>> - ResourceGroupName <<resource-group>>
    ```

    <span data-ttu-id="723ac-271">Az alábbi kódrészletek kiemelik azt a kimenetet, amely az átjáró csatlakoztatásakor (első példa) és leválasztásakor (második példa) jön létre:</span><span class="sxs-lookup"><span data-stu-id="723ac-271">The following snippets highlight the output generated if the gateway is connected (the first example), and disconnected (the second example):</span></span>

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

<span data-ttu-id="723ac-272">Az alábbi javaslatok segítenek megállapítani, hogy van-e probléma a virtuális gazdagép konfigurációjával, a hálózati sávszélesség felhasználásával vagy az alkalmazás teljesítményével:</span><span class="sxs-lookup"><span data-stu-id="723ac-272">The following recommendations are useful for determining if there is an issue with Host VM configuration, network bandwidth utilization, or application performance:</span></span>

- <span data-ttu-id="723ac-273">**Ellenőrizze, hogy az Azure-beli virtuális gépeken futó vendég operációs rendszer tűzfala helyesen van-e konfigurálva, tehát engedi-e a helyszíni IP-tartományokról kiinduló forgalmat.**</span><span class="sxs-lookup"><span data-stu-id="723ac-273">**Verify that the firewall in the guest operating system running on the Azure VMs in the subnet is configured correctly to allow permitted traffic from the on-premises IP ranges.**</span></span>

- <span data-ttu-id="723ac-274">**Ellenőrizze, hogy a forgalom mennyisége nem közelít-e az Azure VPN-átjárón rendelkezésre álló sávszélesség korlátjához.**</span><span class="sxs-lookup"><span data-stu-id="723ac-274">**Verify that the volume of traffic is not close to the limit of the bandwidth available to the Azure VPN gateway.**</span></span>

    <span data-ttu-id="723ac-275">Ennek az ellenőrzési módja a helyszínen futó VPN-berendezéstől függ.</span><span class="sxs-lookup"><span data-stu-id="723ac-275">How to verify this depends on the VPN appliance running on-premises.</span></span> <span data-ttu-id="723ac-276">Ha például a Windows Server 2012 RRAS szolgáltatását használja, a teljesítményfigyelő segítségével nyomon követheti a VPN-kapcsolaton keresztül fogadott és küldött adatok mennyiségét.</span><span class="sxs-lookup"><span data-stu-id="723ac-276">For example, if you are using RRAS on Windows Server 2012, you can use Performance Monitor to track the volume of data being received and transmitted over the VPN connection.</span></span> <span data-ttu-id="723ac-277">A *Távelérés (RAS) áttekintése* objektum segítségével válassza ki a *Fogadott bájtok/mp* és a *Küldési sebesség (bájt/s)* számlálókat:</span><span class="sxs-lookup"><span data-stu-id="723ac-277">Using the *RAS Total* object, select the *Bytes Received/Sec* and *Bytes Transmitted/Sec* counters:</span></span>

    <span data-ttu-id="723ac-278">![[3]][3]</span><span class="sxs-lookup"><span data-stu-id="723ac-278">![[3]][3]</span></span>

    <span data-ttu-id="723ac-279">Az eredményeket hasonlítsa össze a VPN-átjáró számára elérhető sávszélességgel (az alapszintű és a standard termékváltozat esetében 100 Mb/s, illetve 200 Mb/s nagy teljesítményű termékváltozat esetében):</span><span class="sxs-lookup"><span data-stu-id="723ac-279">You should compare the results with the bandwidth available to the VPN gateway (100 Mbps for the Basic and Standard SKUs, and 200 Mbps for the High Performance SKU):</span></span>

    <span data-ttu-id="723ac-280">![[4]][4]</span><span class="sxs-lookup"><span data-stu-id="723ac-280">![[4]][4]</span></span>

- <span data-ttu-id="723ac-281">**Győződjön meg arról, hogy az alkalmazásterheléshez megfelelő számban és méretben telepített virtuális gépeket.**</span><span class="sxs-lookup"><span data-stu-id="723ac-281">**Verify that you have deployed the right number and size of VMs for your application load.**</span></span>

    <span data-ttu-id="723ac-282">Állapítsa meg, hogy nem lassúak-e az Azure virtuális hálózat virtuális gépei.</span><span class="sxs-lookup"><span data-stu-id="723ac-282">Determine if any of the virtual machines in the Azure VNet are running slowly.</span></span> <span data-ttu-id="723ac-283">Ha igen, akkor lehet, hogy túl vannak terhelve. Elképzelhető, hogy túl kevés gép van a terhelés kezeléséhez, vagy nincsenek megfelelően konfigurálva a terheléselosztók.</span><span class="sxs-lookup"><span data-stu-id="723ac-283">If so, they may be overloaded, there may be too few to handle the load, or the load-balancers may not be configured correctly.</span></span> <span data-ttu-id="723ac-284">Ennek meghatározásához, [rögzítse és elemezze a diagnosztikai adatokat][azure-vm-diagnostics].</span><span class="sxs-lookup"><span data-stu-id="723ac-284">To determine this, [capture and analyze diagnostic information][azure-vm-diagnostics].</span></span> <span data-ttu-id="723ac-285">Az eredményeket megvizsgálhatja az Azure Portal segítségével, de számos külső féltől származó, a teljesítményadatok részleteibe bepillantást engedő eszköz is a rendelkezésére áll.</span><span class="sxs-lookup"><span data-stu-id="723ac-285">You can examine the results using the Azure portal, but many third-party tools are also available that can provide detailed insights into the performance data.</span></span>

- <span data-ttu-id="723ac-286">**Ellenőrizze, hogy az alkalmazás hatékonyan használja-e a felhőerőforrásokat.**</span><span class="sxs-lookup"><span data-stu-id="723ac-286">**Verify that the application is making efficient use of cloud resources.**</span></span>

    <span data-ttu-id="723ac-287">Az egyes virtuális gépeken futó kialakítási alkalmazáskód, amelynek feladata meghatározni, hogy az alkalmazások a lehető legjobban kihasználják-e az erőforrásokat.</span><span class="sxs-lookup"><span data-stu-id="723ac-287">Instrument application code running on each VM to determine whether applications are making the best use of resources.</span></span> <span data-ttu-id="723ac-288">Használhatja például az [Application Insights][application-insights] eszközt.</span><span class="sxs-lookup"><span data-stu-id="723ac-288">You can use tools such as [Application Insights][application-insights].</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="723ac-289">A megoldás üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="723ac-289">Deploy the solution</span></span>


<span data-ttu-id="723ac-290">**Előfeltételek.**</span><span class="sxs-lookup"><span data-stu-id="723ac-290">**Prequisites.**</span></span> <span data-ttu-id="723ac-291">Rendelkeznie kell egy megfelelő hálózati berendezéssel konfigurált meglévő helyszíni infrastruktúrával.</span><span class="sxs-lookup"><span data-stu-id="723ac-291">You must have an existing on-premises infrastructure already configured with a suitable network appliance.</span></span>

<span data-ttu-id="723ac-292">A megoldás üzembe helyezéséhez hajtsa végre az alábbi lépéseket.</span><span class="sxs-lookup"><span data-stu-id="723ac-292">To deploy the solution, perform the following steps.</span></span>

1. <span data-ttu-id="723ac-293">Kattintson az alábbi gombra:</span><span class="sxs-lookup"><span data-stu-id="723ac-293">Click the button below:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fvpn%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. <span data-ttu-id="723ac-294">Várja meg, amíg a hivatkozás megnyílik az Azure Portalon, majd kövesse az alábbi lépéseket:</span><span class="sxs-lookup"><span data-stu-id="723ac-294">Wait for the link to open in the Azure portal, then follow these steps:</span></span> 
   * <span data-ttu-id="723ac-295">Az **Erőforráscsoport** neve már meg van adva a paraméterfájlban, ezért válassza az **Új létrehozása** lehetőséget és a szövegmezőbe írja az `ra-hybrid-vpn-rg` karakterláncot.</span><span class="sxs-lookup"><span data-stu-id="723ac-295">The **Resource group** name is already defined in the parameter file, so select **Create New** and enter `ra-hybrid-vpn-rg` in the text box.</span></span>
   * <span data-ttu-id="723ac-296">Válassza ki a régiót a **Hely** legördülő listából.</span><span class="sxs-lookup"><span data-stu-id="723ac-296">Select the region from the **Location** drop down box.</span></span>
   * <span data-ttu-id="723ac-297">Ne szerkessze a **Sablon gyökér szintű URI-je** vagy a **Paraméter gyökér szintű URI-je** szövegmezőt.</span><span class="sxs-lookup"><span data-stu-id="723ac-297">Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.</span></span>
   * <span data-ttu-id="723ac-298">Tekintse át a használati feltételeket, majd kattintson az **Elfogadom a fenti feltételeket** lehetőségre.</span><span class="sxs-lookup"><span data-stu-id="723ac-298">Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.</span></span>
   * <span data-ttu-id="723ac-299">Kattintson a **Vásárlás** gombra.</span><span class="sxs-lookup"><span data-stu-id="723ac-299">Click the **Purchase** button.</span></span>
3. <span data-ttu-id="723ac-300">Várjon, amíg az üzembe helyezés befejeződik.</span><span class="sxs-lookup"><span data-stu-id="723ac-300">Wait for the deployment to complete.</span></span>



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
[azure-gateway-skus]: /azure/vpn-gateway/vpn-gateway-about-vpngateways#gwsku
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
[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-architectures.vsdx
[vpn-appliance-ipsec]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices#ipsec-parameters
[virtualNetworkGateway-parameters]: https://github.com/mspnp/hybrid-networking/vpn/parameters/virtualNetworkGateway.parameters.json
[azure-cli]: https://azure.microsoft.com/documentation/articles/xplat-cli-install/
[CIDR]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
[0]: ./images/vpn.png "Helyszíni és Azure infrastruktúrákat áthidaló hibrid hálózat"
[2]: ../_images/guidance-hybrid-network-vpn/audit-logs.png "Auditnaplók az Azure Portalon"
[3]: ../_images/guidance-hybrid-network-vpn/RRAS-perf-counters.png "Teljesítményszámlálók a VPN-hálózati forgalom monitorozásához"
[4]: ../_images/guidance-hybrid-network-vpn/RRAS-perf-graph.png "Példa VPN-hálózat teljesítménye ábra"
