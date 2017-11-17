---
title: "Egy magas rendelkezésre állású hibrid hálózati architektúra megvalósítása"
description: "Hogyan egy biztonságos pont-pont hálózati architektúra, amely egy Azure virtuális hálózatra és egy a helyszíni hálózathoz csatlakoztatott ExpressRoute a VPN-átjáró feladatátvételi kiterjedő végrehajtásához."
author: telmosampaio
ms.date: 11/28/2016
pnp.series.title: Connect an on-premises network to Azure
pnp.series.prev: expressroute
cardTitle: Improving availability
ms.openlocfilehash: 4c101f17e5e91085b61178f9efb2bc5acb61189c
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="connect-an-on-premises-network-to-azure-using-expressroute-with-vpn-failover"></a><span data-ttu-id="025f5-103">Egy helyszíni hálózat csatlakozik az Azure ExpressRoute segítségével a VPN-feladatátvételi</span><span class="sxs-lookup"><span data-stu-id="025f5-103">Connect an on-premises network to Azure using ExpressRoute with VPN failover</span></span>

<span data-ttu-id="025f5-104">A referencia-architektúrában bemutatja, hogyan csatlakozzon egy helyi hálózati egy Azure virtuális hálózatot (VNet) ExpressRoute, pont-pont virtuális magánhálózat (VPN), a feladatátvételi kapcsolat használatával.</span><span class="sxs-lookup"><span data-stu-id="025f5-104">This reference architecture shows how to connect an on-premises network to an Azure virtual network (VNet) using ExpressRoute, with a site-to-site virtual private network (VPN) as a failover connection.</span></span> <span data-ttu-id="025f5-105">A helyszíni hálózat és az ExpressRoute-kapcsolaton keresztül az Azure VNet közötti forgalmat.</span><span class="sxs-lookup"><span data-stu-id="025f5-105">Traffic flows between the on-premises network and the Azure VNet through an ExpressRoute connection.</span></span> <span data-ttu-id="025f5-106">Ha a kapcsolat az ExpressRoute-kapcsolatcsoport leállását, továbbítódik az IPSec VPN-alagúton keresztül.</span><span class="sxs-lookup"><span data-stu-id="025f5-106">If there is a loss of connectivity in the ExpressRoute circuit, traffic is routed through an IPSec VPN tunnel.</span></span> [<span data-ttu-id="025f5-107">**Ez a megoldás üzembe helyezéséhez**.</span><span class="sxs-lookup"><span data-stu-id="025f5-107">**Deploy this solution**.</span></span>](#deploy-the-solution)

<span data-ttu-id="025f5-108">Vegye figyelembe, hogy az ExpressRoute-kapcsolatcsoport nem érhető el, ha a VPN-útvonal csak kezelnek magánhálózati társviszony-létesítési kapcsolatok.</span><span class="sxs-lookup"><span data-stu-id="025f5-108">Note that if the ExpressRoute circuit is unavailable, the VPN route will only handle private peering connections.</span></span> <span data-ttu-id="025f5-109">Nyilvános társviszony-létesítést és a Microsoft társviszony-létesítés kapcsolatok az interneten keresztül továbbítani fogja.</span><span class="sxs-lookup"><span data-stu-id="025f5-109">Public peering and Microsoft peering connections will pass over the Internet.</span></span> 

<span data-ttu-id="025f5-110">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="025f5-110">![[0]][0]</span></span>

<span data-ttu-id="025f5-111">*Töltse le a [Visio fájl] [ visio-download] ezen architektúra.*</span><span class="sxs-lookup"><span data-stu-id="025f5-111">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="025f5-112">Architektúra</span><span class="sxs-lookup"><span data-stu-id="025f5-112">Architecture</span></span> 

<span data-ttu-id="025f5-113">Az architektúra a következő összetevőkből áll.</span><span class="sxs-lookup"><span data-stu-id="025f5-113">The architecture consists of the following components.</span></span>

* <span data-ttu-id="025f5-114">**A helyszíni hálózat**.</span><span class="sxs-lookup"><span data-stu-id="025f5-114">**On-premises network**.</span></span> <span data-ttu-id="025f5-115">Helyi magánhálózat fut egy szervezeten belül.</span><span class="sxs-lookup"><span data-stu-id="025f5-115">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="025f5-116">**VPN-készülék**.</span><span class="sxs-lookup"><span data-stu-id="025f5-116">**VPN appliance**.</span></span> <span data-ttu-id="025f5-117">Egy eszköz vagy a helyszíni hálózathoz külső kapcsolatot biztosító szolgáltatás.</span><span class="sxs-lookup"><span data-stu-id="025f5-117">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="025f5-118">Lehet, hogy a VPN-készülék olyan hardvereszköz, vagy egy szoftveres megoldás, mint például az Útválasztás és távelérés szolgáltatás (RRAS) a Windows Server 2012-ben.</span><span class="sxs-lookup"><span data-stu-id="025f5-118">The VPN appliance may be a hardware device, or it can be a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="025f5-119">Támogatott VPN-készülék és konfigurálásáról a kiválasztott VPN készülékek Azure történő kapcsolódáshoz listájáért lásd: [kapcsolatos VPN-eszközök a webhelyek közötti VPN átjáró kapcsolatok][vpn-appliance].</span><span class="sxs-lookup"><span data-stu-id="025f5-119">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="025f5-120">**ExpressRoute-kapcsolatcsoportot**.</span><span class="sxs-lookup"><span data-stu-id="025f5-120">**ExpressRoute circuit**.</span></span> <span data-ttu-id="025f5-121">A 2. vagy 3 rétegbeli áramkör által biztosított a kapcsolat szolgáltatójánál, hogy a peremhálózati útválasztók az Azure-ral csatlakozik a helyi hálózaton.</span><span class="sxs-lookup"><span data-stu-id="025f5-121">A layer 2 or layer 3 circuit supplied by the connectivity provider that joins the on-premises network with Azure through the edge routers.</span></span> <span data-ttu-id="025f5-122">A kapcsolatcsoport a hardver-infrastruktúrát kezeli a kapcsolat szolgáltatóját használja.</span><span class="sxs-lookup"><span data-stu-id="025f5-122">The circuit uses the hardware infrastructure managed by the connectivity provider.</span></span>

* <span data-ttu-id="025f5-123">**ExpressRoute virtuális hálózati átjáró**.</span><span class="sxs-lookup"><span data-stu-id="025f5-123">**ExpressRoute virtual network gateway**.</span></span> <span data-ttu-id="025f5-124">Az ExpressRoute virtuális hálózati átjáró lehetővé teszi, hogy a kapcsolat a helyszíni hálózaton használt ExpressRoute-kapcsolatcsoportot csatlakozni a virtuális hálózat.</span><span class="sxs-lookup"><span data-stu-id="025f5-124">The ExpressRoute virtual network gateway enables the VNet to connect to the ExpressRoute circuit used for connectivity with your on-premises network.</span></span>

* <span data-ttu-id="025f5-125">**VPN virtuális hálózati átjáró**.</span><span class="sxs-lookup"><span data-stu-id="025f5-125">**VPN virtual network gateway**.</span></span> <span data-ttu-id="025f5-126">A VPN-virtuális hálózati átjáró lehetővé teszi, hogy a helyi hálózaton a VPN-készülék csatlakozni a virtuális hálózat.</span><span class="sxs-lookup"><span data-stu-id="025f5-126">The VPN virtual network gateway enables the VNet to connect to the VPN appliance in the on-premises network.</span></span> <span data-ttu-id="025f5-127">A VPN-virtuális hálózati átjárót a helyszíni hálózat kéréseket fogad csak a VPN-készülék keresztül van konfigurálva.</span><span class="sxs-lookup"><span data-stu-id="025f5-127">The VPN virtual network gateway is configured to accept requests from the on-premises network only through the VPN appliance.</span></span> <span data-ttu-id="025f5-128">További információkért lásd: [egy a helyszíni hálózathoz csatlakozni a Microsoft Azure virtuális hálózat][connect-to-an-Azure-vnet].</span><span class="sxs-lookup"><span data-stu-id="025f5-128">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

* <span data-ttu-id="025f5-129">**VPN-kapcsolat**.</span><span class="sxs-lookup"><span data-stu-id="025f5-129">**VPN connection**.</span></span> <span data-ttu-id="025f5-130">A kapcsolat tulajdonságai adja meg a kapcsolat típusát (IPSec) és a kulcsot, a helyszíni VPN-készülék megosztott forgalom titkosításához.</span><span class="sxs-lookup"><span data-stu-id="025f5-130">The connection has properties that specify the connection type (IPSec) and the key shared with the on-premises VPN appliance to encrypt traffic.</span></span>

* <span data-ttu-id="025f5-131">**Az Azure Virtual Network (VNet)**.</span><span class="sxs-lookup"><span data-stu-id="025f5-131">**Azure Virtual Network (VNet)**.</span></span> <span data-ttu-id="025f5-132">Minden virtuális hálózat egyetlen Azure régióban található, és több alkalmazásrétegek üzemeltethet.</span><span class="sxs-lookup"><span data-stu-id="025f5-132">Each VNet resides in a single Azure region, and can host multiple application tiers.</span></span> <span data-ttu-id="025f5-133">Alkalmazás rétegeket is szegmentált, minden egyes virtuális alhálózatok használatával.</span><span class="sxs-lookup"><span data-stu-id="025f5-133">Application tiers can be segmented using subnets in each VNet.</span></span>

* <span data-ttu-id="025f5-134">**Átjáró alhálózati**.</span><span class="sxs-lookup"><span data-stu-id="025f5-134">**Gateway subnet**.</span></span> <span data-ttu-id="025f5-135">A virtuális hálózati átjárók ugyanazon az alhálózaton van használatban.</span><span class="sxs-lookup"><span data-stu-id="025f5-135">The virtual network gateways are held in the same subnet.</span></span>

* <span data-ttu-id="025f5-136">**A felhő alkalmazás**.</span><span class="sxs-lookup"><span data-stu-id="025f5-136">**Cloud application**.</span></span> <span data-ttu-id="025f5-137">Az alkalmazást az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="025f5-137">The application hosted in Azure.</span></span> <span data-ttu-id="025f5-138">A többrétegű konfigurációk – Azure load Balancer terheléselosztók keresztül kapcsolódik, több alhálózattal rendelkező tartalmazhat.</span><span class="sxs-lookup"><span data-stu-id="025f5-138">It might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="025f5-139">Az alkalmazás-infrastruktúrával kapcsolatos további információkért lásd: [futó Windows virtuális gép munkaterhelések] [ windows-vm-ra] és [futó Linux virtuális gép munkaterhelések] [ linux-vm-ra].</span><span class="sxs-lookup"><span data-stu-id="025f5-139">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

## <a name="recommendations"></a><span data-ttu-id="025f5-140">Javaslatok</span><span class="sxs-lookup"><span data-stu-id="025f5-140">Recommendations</span></span>

<span data-ttu-id="025f5-141">Az alábbi javaslatok a legtöbb forgatókönyvre vonatkoznak.</span><span class="sxs-lookup"><span data-stu-id="025f5-141">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="025f5-142">Kövesse ezeket a javaslatokat, ha nincsenek ezeket felülíró követelményei.</span><span class="sxs-lookup"><span data-stu-id="025f5-142">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="vnet-and-gatewaysubnet"></a><span data-ttu-id="025f5-143">Virtuális hálózat és a GatewaySubnet</span><span class="sxs-lookup"><span data-stu-id="025f5-143">VNet and GatewaySubnet</span></span>

<span data-ttu-id="025f5-144">Az ExpressRoute virtuális hálózati átjáró és a VPN-virtuális hálózati átjáró létrehozása ugyanazt a virtuális hálózatot.</span><span class="sxs-lookup"><span data-stu-id="025f5-144">Create the ExpressRoute virtual network gateway and the VPN virtual network gateway in the same VNet.</span></span> <span data-ttu-id="025f5-145">Ez azt jelenti, hogy ugyanazon az alhálózaton nevű kell közös *GatewaySubnet*.</span><span class="sxs-lookup"><span data-stu-id="025f5-145">This means that they should share the same subnet named *GatewaySubnet*.</span></span>

<span data-ttu-id="025f5-146">Ha a virtuális hálózat már tartalmaz egy nevű alhálózat *GatewaySubnet*, győződjön meg arról, hogy egy /27 vagy nagyobb címtartományt.</span><span class="sxs-lookup"><span data-stu-id="025f5-146">If the VNet already includes a subnet named *GatewaySubnet*, ensure that it has a /27 or larger address space.</span></span> <span data-ttu-id="025f5-147">Ha a meglévő alhálózati túl kicsi, a következő PowerShell-parancs segítségével távolítsa el az alhálózatot:</span><span class="sxs-lookup"><span data-stu-id="025f5-147">If the existing subnet is too small, use the following PowerShell command to remove the subnet:</span></span> 

```powershell
$vnet = Get-AzureRmVirtualNetworkGateway -Name <yourvnetname> -ResourceGroupName <yourresourcegroup>
Remove-AzureRmVirtualNetworkSubnetConfig -Name GatewaySubnet -VirtualNetwork $vnet
```

<span data-ttu-id="025f5-148">Ha a virtuális hálózat nem tartalmaz nevű alhálózat **GatewaySubnet**, hozzon létre egy újat a következő Powershell-parancsot:</span><span class="sxs-lookup"><span data-stu-id="025f5-148">If the VNet does not contain a subnet named **GatewaySubnet**, create a new one using the following Powershell command:</span></span>

```powershell
$vnet = Get-AzureRmVirtualNetworkGateway -Name <yourvnetname> -ResourceGroupName <yourresourcegroup>
Add-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet -AddressPrefix "10.200.255.224/27"
$vnet = Set-AzureRmVirtualNetwork -VirtualNetwork $vnet
```

### <a name="vpn-and-expressroute-gateways"></a><span data-ttu-id="025f5-149">VPN- és az ExpressRoute-átjáró</span><span class="sxs-lookup"><span data-stu-id="025f5-149">VPN and ExpressRoute gateways</span></span>

<span data-ttu-id="025f5-150">Győződjön meg arról, hogy megfelel-e a szervezet a [ExpressRoute előfeltételként szükséges követelmények] [ expressroute-prereq] Azure való kapcsolódáshoz.</span><span class="sxs-lookup"><span data-stu-id="025f5-150">Verify that your organization meets the [ExpressRoute prerequisite requirements][expressroute-prereq] for connecting to Azure.</span></span>

<span data-ttu-id="025f5-151">Ha az Azure virtuális hálózat már rendelkezik egy VPN-virtuális hálózati átjáró, a következő Powershell-parancs segítségével távolítsa el:</span><span class="sxs-lookup"><span data-stu-id="025f5-151">If you already have a VPN virtual network gateway in your Azure VNet, use the following  Powershell command to remove it:</span></span>

```powershell
Remove-AzureRmVirtualNetworkGateway -Name <yourgatewayname> -ResourceGroupName <yourresourcegroup>
```

<span data-ttu-id="025f5-152">Kövesse az utasításokat a [egy hibrid hálózati architektúra az Azure ExpressRoute végrehajtási] [ implementing-expressroute] az ExpressRoute-kapcsolatot létesíteni.</span><span class="sxs-lookup"><span data-stu-id="025f5-152">Follow the instructions in [Implementing a hybrid network architecture with Azure ExpressRoute][implementing-expressroute] to establish your ExpressRoute connection.</span></span>

<span data-ttu-id="025f5-153">Kövesse az utasításokat a [egy hibrid hálózati architektúra az Azure és a helyszíni VPN-végrehajtási] [ implementing-vpn] a VPN-virtuális hálózati átjáró kapcsolat létrehozásához.</span><span class="sxs-lookup"><span data-stu-id="025f5-153">Follow the instructions in [Implementing a hybrid network architecture with Azure and On-premises VPN][implementing-vpn] to establish your VPN virtual network gateway connection.</span></span>

<span data-ttu-id="025f5-154">Miután a virtuális hálózati átjáró-kapcsolatok létesítése, tesztkörnyezetben a következőképpen:</span><span class="sxs-lookup"><span data-stu-id="025f5-154">After you have established the virtual network gateway connections, test the environment as follows:</span></span>

1. <span data-ttu-id="025f5-155">Győződjön meg arról, hogy a helyi hálózaton is elérheti az Azure virtuális hálózat.</span><span class="sxs-lookup"><span data-stu-id="025f5-155">Make sure you can connect from your on-premises network to your Azure VNet.</span></span>
2. <span data-ttu-id="025f5-156">Lépjen kapcsolatba a szolgáltatót, hogy állítsa le az ExpressRoute-kapcsolatot a teszteléshez.</span><span class="sxs-lookup"><span data-stu-id="025f5-156">Contact your provider to stop ExpressRoute connectivity for testing.</span></span>
3. <span data-ttu-id="025f5-157">Győződjön meg arról, hogy továbbra is csatlakozhat a helyszíni hálózatból az Azure virtuális hálózat a virtuális hálózati átjáró VPN-kapcsolat használatával.</span><span class="sxs-lookup"><span data-stu-id="025f5-157">Verify that you can still connect from your on-premises network to your Azure VNet using the VPN virtual network gateway connection.</span></span>
4. <span data-ttu-id="025f5-158">Kérje a szolgáltató újbóli ExpressRoute-kapcsolatot.</span><span class="sxs-lookup"><span data-stu-id="025f5-158">Contact your provider to reestablish ExpressRoute connectivity.</span></span>

## <a name="considerations"></a><span data-ttu-id="025f5-159">Megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="025f5-159">Considerations</span></span>

<span data-ttu-id="025f5-160">ExpressRoute szempontok, tekintse meg a [egy hibrid hálózati architektúra az Azure ExpressRoute végrehajtási] [ guidance-expressroute] útmutatást.</span><span class="sxs-lookup"><span data-stu-id="025f5-160">For ExpressRoute considerations, see the [Implementing a Hybrid Network Architecture with Azure ExpressRoute][guidance-expressroute] guidance.</span></span>

<span data-ttu-id="025f5-161">Pont-pont VPN szempontok, tekintse meg a [egy hibrid hálózati architektúra az Azure és a helyszíni VPN-végrehajtási] [ guidance-vpn] útmutatást.</span><span class="sxs-lookup"><span data-stu-id="025f5-161">For site-to-site VPN considerations, see the [Implementing a Hybrid Network Architecture with Azure and On-premises VPN][guidance-vpn] guidance.</span></span>

<span data-ttu-id="025f5-162">Általános Azure biztonsági szempontokat lásd: [Microsoft cloud services és a hálózati biztonság][best-practices-security].</span><span class="sxs-lookup"><span data-stu-id="025f5-162">For general Azure security considerations, see [Microsoft cloud services and network security][best-practices-security].</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="025f5-163">A megoldás üzembe helyezéséhez</span><span class="sxs-lookup"><span data-stu-id="025f5-163">Deploy the solution</span></span>

<span data-ttu-id="025f5-164">**Prequisites.**</span><span class="sxs-lookup"><span data-stu-id="025f5-164">**Prequisites.**</span></span> <span data-ttu-id="025f5-165">Rendelkeznie kell egy meglévő helyi infrastruktúra már be van állítva a megfelelő hálózati berendezések.</span><span class="sxs-lookup"><span data-stu-id="025f5-165">You must have an existing on-premises infrastructure already configured with a suitable network appliance.</span></span>

<span data-ttu-id="025f5-166">A megoldás üzembe helyezéséhez, a következő lépésekkel.</span><span class="sxs-lookup"><span data-stu-id="025f5-166">To deploy the solution, perform the following steps.</span></span>

1. <span data-ttu-id="025f5-167">Kattintson a lenti gombra:</span><span class="sxs-lookup"><span data-stu-id="025f5-167">Click the button below:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute-vpn-failover%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. <span data-ttu-id="025f5-168">Várjon, amíg a hivatkozásra kattintva nyissa meg az Azure portálon, majd kövesse az alábbi lépéseket:</span><span class="sxs-lookup"><span data-stu-id="025f5-168">Wait for the link to open in the Azure portal, then follow these steps:</span></span>   
   * <span data-ttu-id="025f5-169">Az **Erőforráscsoport** neve már meg van adva a paraméterfájlban, ezért válassza az **Új létrehozása** lehetőséget és a szövegmezőbe írja az `ra-hybrid-vpn-er-rg` karakterláncot.</span><span class="sxs-lookup"><span data-stu-id="025f5-169">The **Resource group** name is already defined in the parameter file, so select **Create New** and enter `ra-hybrid-vpn-er-rg` in the text box.</span></span>
   * <span data-ttu-id="025f5-170">Válassza ki a régiót a **Hely** legördülő listából.</span><span class="sxs-lookup"><span data-stu-id="025f5-170">Select the region from the **Location** drop down box.</span></span>
   * <span data-ttu-id="025f5-171">Ne szerkessze a **Sablon gyökér szintű URI-je** vagy a **Paraméter gyökér szintű URI-je** szövegmezőt.</span><span class="sxs-lookup"><span data-stu-id="025f5-171">Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.</span></span>
   * <span data-ttu-id="025f5-172">Tekintse át a használati feltételeket, majd kattintson az **Elfogadom a fenti feltételeket** lehetőségre.</span><span class="sxs-lookup"><span data-stu-id="025f5-172">Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.</span></span>
   * <span data-ttu-id="025f5-173">Kattintson a **beszerzési** gombra.</span><span class="sxs-lookup"><span data-stu-id="025f5-173">Click the **Purchase** button.</span></span>
3. <span data-ttu-id="025f5-174">Várjon, amíg az üzembe helyezés befejeződik.</span><span class="sxs-lookup"><span data-stu-id="025f5-174">Wait for the deployment to complete.</span></span>
4. <span data-ttu-id="025f5-175">Kattintson a lenti gombra:</span><span class="sxs-lookup"><span data-stu-id="025f5-175">Click the button below:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute-vpn-failover%2Fazuredeploy-expressRouteCircuit.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
5. <span data-ttu-id="025f5-176">Várjon, amíg a hivatkozásra kattintva nyissa meg az Azure portálon, majd adja meg, majd kövesse az alábbi lépéseket:</span><span class="sxs-lookup"><span data-stu-id="025f5-176">Wait for the link to open in the Azure portal, then enter then follow these steps:</span></span>
   * <span data-ttu-id="025f5-177">Válassza ki **használata meglévő** a a **erőforráscsoport** szakaszt, és adja meg `ra-hybrid-vpn-er-rg` a szövegmezőben.</span><span class="sxs-lookup"><span data-stu-id="025f5-177">Select **Use existing** in the **Resource group** section and enter `ra-hybrid-vpn-er-rg` in the text box.</span></span>
   * <span data-ttu-id="025f5-178">Válassza ki a régiót a **Hely** legördülő listából.</span><span class="sxs-lookup"><span data-stu-id="025f5-178">Select the region from the **Location** drop down box.</span></span>
   * <span data-ttu-id="025f5-179">Ne szerkessze a **Sablon gyökér szintű URI-je** vagy a **Paraméter gyökér szintű URI-je** szövegmezőt.</span><span class="sxs-lookup"><span data-stu-id="025f5-179">Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.</span></span>
   * <span data-ttu-id="025f5-180">Tekintse át a használati feltételeket, majd kattintson az **Elfogadom a fenti feltételeket** lehetőségre.</span><span class="sxs-lookup"><span data-stu-id="025f5-180">Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.</span></span>
   * <span data-ttu-id="025f5-181">Kattintson a **beszerzési** gombra.</span><span class="sxs-lookup"><span data-stu-id="025f5-181">Click the **Purchase** button.</span></span>

<!-- links -->

[windows-vm-ra]: ../virtual-machines-windows/index.md
[linux-vm-ra]: ../virtual-machines-linux/index.md


[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[vpn-appliance]: /azure/vpn-gateway/vpn-gateway-about-vpn-devices
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[connect-to-an-Azure-vnet]: https://technet.microsoft.com/library/dn786406.aspx
[expressroute-prereq]: /azure/expressroute/expressroute-prerequisites
[implementing-expressroute]: ./expressroute.md
[implementing-vpn]: ./vpn.md
[guidance-expressroute]: ./expressroute.md
[guidance-vpn]: ./vpn.md
[best-practices-security]: /azure/best-practices-network-security
[visio-download]: https://archcenter.azureedge.net/cdn/hybrid-network-architectures.vsdx
[0]: ./images/expressroute-vpn-failover.png "Egy magas rendelkezésre állású hibrid hálózati architektúra ExpressRoute- és VPN-átjáró architektúrája"
