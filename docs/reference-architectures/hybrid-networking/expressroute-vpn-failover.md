---
title: Magas rendelkezésre állású hibrid hálózati architektúra megvalósítása
description: A jelen cikk azt ismerteti, hogyan lehet kiépíteni egy helyek közötti biztonságos hálózati architektúrát, amely a VPN-átjáróval feladatátvételt biztosító ExpressRoute használatával összekapcsolt Azure-beli virtuális hálózatból és helyszíni hálózatból áll.
author: telmosampaio
ms.date: 10/22/2017
pnp.series.title: Connect an on-premises network to Azure
pnp.series.prev: expressroute
cardTitle: Improving availability
ms.openlocfilehash: 31bf471dbff3661face7d94fbec0973d81541ec7
ms.sourcegitcommit: dbbf914757b03cdee7a274204f9579fa63d7eed2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/02/2018
ms.locfileid: "50916413"
---
# <a name="connect-an-on-premises-network-to-azure-using-expressroute-with-vpn-failover"></a><span data-ttu-id="ce22e-103">Helyszíni hálózat csatlakoztatása az Azure-hoz VPN-feladatátvételt biztosító ExpressRoute használatával</span><span class="sxs-lookup"><span data-stu-id="ce22e-103">Connect an on-premises network to Azure using ExpressRoute with VPN failover</span></span>

<span data-ttu-id="ce22e-104">Ez a referenciaarchitektúra azt mutatja be, hogy hogyan kapcsolható össze egy helyszíni hálózat egy Azure-beli virtuális hálózattal az ExpressRoute segítségével, amely helyek közötti virtuális magánhálózatot (VPN) biztosít feladatátvételi kapcsolatként.</span><span class="sxs-lookup"><span data-stu-id="ce22e-104">This reference architecture shows how to connect an on-premises network to an Azure virtual network (VNet) using ExpressRoute, with a site-to-site virtual private network (VPN) as a failover connection.</span></span> <span data-ttu-id="ce22e-105">A helyszíni hálózat és az Azure-beli virtuális hálózat közötti forgalom egy ExpressRoute-kapcsolaton keresztül halad át.</span><span class="sxs-lookup"><span data-stu-id="ce22e-105">Traffic flows between the on-premises network and the Azure VNet through an ExpressRoute connection.</span></span> <span data-ttu-id="ce22e-106">Ha az ExpressRoute-kapcsolatcsoportban megszakad a kapcsolat, a forgalmat egy IPSec VPN-alagúton keresztül továbbítja a rendszer.</span><span class="sxs-lookup"><span data-stu-id="ce22e-106">If there is a loss of connectivity in the ExpressRoute circuit, traffic is routed through an IPSec VPN tunnel.</span></span> [<span data-ttu-id="ce22e-107">**A megoldás üzembe helyezése**.</span><span class="sxs-lookup"><span data-stu-id="ce22e-107">**Deploy this solution**.</span></span>](#deploy-the-solution)

<span data-ttu-id="ce22e-108">Vegye figyelembe, hogy ha az ExpressRoute-kapcsolatcsoport nem áll rendelkezésre, a VPN-útválasztás csak a magánhálózati társviszony-létesítésen alapuló kapcsolatokat fogja kezelni.</span><span class="sxs-lookup"><span data-stu-id="ce22e-108">Note that if the ExpressRoute circuit is unavailable, the VPN route will only handle private peering connections.</span></span> <span data-ttu-id="ce22e-109">A nyilvános és a Microsoft társviszony-létesítésen alapuló kapcsolatok az interneten fognak továbbítódni.</span><span class="sxs-lookup"><span data-stu-id="ce22e-109">Public peering and Microsoft peering connections will pass over the Internet.</span></span> 

<span data-ttu-id="ce22e-110">![[0]][0]</span><span class="sxs-lookup"><span data-stu-id="ce22e-110">![[0]][0]</span></span>

<span data-ttu-id="ce22e-111">*Töltse le az architektúra [Visio-fájlját][visio-download].*</span><span class="sxs-lookup"><span data-stu-id="ce22e-111">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="ce22e-112">Architektúra</span><span class="sxs-lookup"><span data-stu-id="ce22e-112">Architecture</span></span> 

<span data-ttu-id="ce22e-113">Az architektúra a következőkben leírt összetevőkből áll.</span><span class="sxs-lookup"><span data-stu-id="ce22e-113">The architecture consists of the following components.</span></span>

* <span data-ttu-id="ce22e-114">**Helyszíni hálózat**.</span><span class="sxs-lookup"><span data-stu-id="ce22e-114">**On-premises network**.</span></span> <span data-ttu-id="ce22e-115">A cégen belül futó helyi magánhálózat.</span><span class="sxs-lookup"><span data-stu-id="ce22e-115">A private local-area network running within an organization.</span></span>

* <span data-ttu-id="ce22e-116">**VPN-berendezés**.</span><span class="sxs-lookup"><span data-stu-id="ce22e-116">**VPN appliance**.</span></span> <span data-ttu-id="ce22e-117">A helyszíni hálózat számára külső kapcsolatot biztosító eszköz vagy szolgáltatás.</span><span class="sxs-lookup"><span data-stu-id="ce22e-117">A device or service that provides external connectivity to the on-premises network.</span></span> <span data-ttu-id="ce22e-118">A VPN-berendezés lehet hardvereszköz vagy valamilyen szoftvermegoldás, amilyen például a Windows Server 2012 Útválasztás és távelérés szolgáltatása (Routing and Remote Access Service, RRAS).</span><span class="sxs-lookup"><span data-stu-id="ce22e-118">The VPN appliance may be a hardware device, or it can be a software solution such as the Routing and Remote Access Service (RRAS) in Windows Server 2012.</span></span> <span data-ttu-id="ce22e-119">A támogatott VPN-berendezések listáját és az egyes VPN-berendezések Azure-hoz való csatlakoztatásra történő konfigurálásával kapcsolatos információkat az [Információk a helyek közötti VPN Gateway-kapcsolatok VPN-eszközeiről][vpn-appliance] című cikkben találja.</span><span class="sxs-lookup"><span data-stu-id="ce22e-119">For a list of supported VPN appliances and information on configuring selected VPN appliances for connecting to Azure, see [About VPN devices for Site-to-Site VPN Gateway connections][vpn-appliance].</span></span>

* <span data-ttu-id="ce22e-120">**ExpressRoute-kapcsolatcsoport**.</span><span class="sxs-lookup"><span data-stu-id="ce22e-120">**ExpressRoute circuit**.</span></span> <span data-ttu-id="ce22e-121">A kapcsolatszolgáltató által biztosított 2. vagy 3. rétegbeli kapcsolatcsoport, amely a peremhálózati útválasztókon keresztül kapcsolja össze a helyszíni hálózatot az Azure-ral.</span><span class="sxs-lookup"><span data-stu-id="ce22e-121">A layer 2 or layer 3 circuit supplied by the connectivity provider that joins the on-premises network with Azure through the edge routers.</span></span> <span data-ttu-id="ce22e-122">A kapcsolatcsoport a kapcsolatszolgáltató által felügyelt hardveres infrastruktúrát használja.</span><span class="sxs-lookup"><span data-stu-id="ce22e-122">The circuit uses the hardware infrastructure managed by the connectivity provider.</span></span>

* <span data-ttu-id="ce22e-123">**ExpressRoute virtuális hálózati átjáró**.</span><span class="sxs-lookup"><span data-stu-id="ce22e-123">**ExpressRoute virtual network gateway**.</span></span> <span data-ttu-id="ce22e-124">Az ExpressRoute virtuális hálózati átjáró lehetővé teszi, hogy a virtuális hálózat csatlakozni tudjon a helyszíni hálózattal létesített kapcsolathoz használt ExpressRoute-kapcsolatcsoporthoz.</span><span class="sxs-lookup"><span data-stu-id="ce22e-124">The ExpressRoute virtual network gateway enables the VNet to connect to the ExpressRoute circuit used for connectivity with your on-premises network.</span></span>

* <span data-ttu-id="ce22e-125">**VPN virtuális hálózati átjáró**.</span><span class="sxs-lookup"><span data-stu-id="ce22e-125">**VPN virtual network gateway**.</span></span> <span data-ttu-id="ce22e-126">A VPN virtuális hálózati átjáró a helyszíni hálózaton található VPN-berendezéshez való csatlakozást teszi lehetővé a virtuális hálózat számára.</span><span class="sxs-lookup"><span data-stu-id="ce22e-126">The VPN virtual network gateway enables the VNet to connect to the VPN appliance in the on-premises network.</span></span> <span data-ttu-id="ce22e-127">A VPN virtuális hálózati átjáró úgy van konfigurálva, hogy kizárólag a VPN-berendezésen keresztül fogadjon el kéréseket a helyszíni hálózattól.</span><span class="sxs-lookup"><span data-stu-id="ce22e-127">The VPN virtual network gateway is configured to accept requests from the on-premises network only through the VPN appliance.</span></span> <span data-ttu-id="ce22e-128">További információért tekintse át a [helyszíni hálózat és a Microsoft Azure Virtual Network csatlakoztatásával][connect-to-an-Azure-vnet] foglalkozó cikket.</span><span class="sxs-lookup"><span data-stu-id="ce22e-128">For more information, see [Connect an on-premises network to a Microsoft Azure virtual network][connect-to-an-Azure-vnet].</span></span>

* <span data-ttu-id="ce22e-129">**VPN-kapcsolat**.</span><span class="sxs-lookup"><span data-stu-id="ce22e-129">**VPN connection**.</span></span> <span data-ttu-id="ce22e-130">A kapcsolat olyan tulajdonságokkal rendelkezik, amelyek megadják a kapcsolat típusát (IPSec) és a helyszíni VPN-berendezéssel megosztott kulcsot a forgalom titkosításához.</span><span class="sxs-lookup"><span data-stu-id="ce22e-130">The connection has properties that specify the connection type (IPSec) and the key shared with the on-premises VPN appliance to encrypt traffic.</span></span>

* <span data-ttu-id="ce22e-131">**Azure Virtual Network (VNet)**.</span><span class="sxs-lookup"><span data-stu-id="ce22e-131">**Azure Virtual Network (VNet)**.</span></span> <span data-ttu-id="ce22e-132">Mindegyik virtuális hálózat egy adott Azure-régióban található, és több alkalmazásréteget is üzemeltethet.</span><span class="sxs-lookup"><span data-stu-id="ce22e-132">Each VNet resides in a single Azure region, and can host multiple application tiers.</span></span> <span data-ttu-id="ce22e-133">Az alkalmazásrétegek alhálózatokkal szegmentálhatók az egyes virtuális hálózatokban.</span><span class="sxs-lookup"><span data-stu-id="ce22e-133">Application tiers can be segmented using subnets in each VNet.</span></span>

* <span data-ttu-id="ce22e-134">**Átjáró-alhálózat**.</span><span class="sxs-lookup"><span data-stu-id="ce22e-134">**Gateway subnet**.</span></span> <span data-ttu-id="ce22e-135">A virtuális hálózati átjárók ugyanazon az alhálózaton találhatók.</span><span class="sxs-lookup"><span data-stu-id="ce22e-135">The virtual network gateways are held in the same subnet.</span></span>

* <span data-ttu-id="ce22e-136">**Felhőalkalmazás**.</span><span class="sxs-lookup"><span data-stu-id="ce22e-136">**Cloud application**.</span></span> <span data-ttu-id="ce22e-137">Az Azure-ban üzemeltetett alkalmazás.</span><span class="sxs-lookup"><span data-stu-id="ce22e-137">The application hosted in Azure.</span></span> <span data-ttu-id="ce22e-138">Több réteget is foglalhat magában, amelyek alhálózatait Azure-terheléselosztók kapcsolják össze.</span><span class="sxs-lookup"><span data-stu-id="ce22e-138">It might include multiple tiers, with multiple subnets connected through Azure load balancers.</span></span> <span data-ttu-id="ce22e-139">További információ az alkalmazás-infrastruktúrával kapcsolatban: [Windows rendszerű virtuális gépek számítási feladatainak futtatása][windows-vm-ra] és [Számítási feladatok futtatása Linux rendszerű virtuális gépeken][linux-vm-ra].</span><span class="sxs-lookup"><span data-stu-id="ce22e-139">For more information about the application infrastructure, see [Running Windows VM workloads][windows-vm-ra] and [Running Linux VM workloads][linux-vm-ra].</span></span>

## <a name="recommendations"></a><span data-ttu-id="ce22e-140">Javaslatok</span><span class="sxs-lookup"><span data-stu-id="ce22e-140">Recommendations</span></span>

<span data-ttu-id="ce22e-141">Az alábbi javaslatok a legtöbb forgatókönyvre vonatkoznak.</span><span class="sxs-lookup"><span data-stu-id="ce22e-141">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="ce22e-142">Kövesse ezeket a javaslatokat, ha nincsenek ezeket felülíró követelményei.</span><span class="sxs-lookup"><span data-stu-id="ce22e-142">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="vnet-and-gatewaysubnet"></a><span data-ttu-id="ce22e-143">Virtuális hálózat és GatewaySubnet</span><span class="sxs-lookup"><span data-stu-id="ce22e-143">VNet and GatewaySubnet</span></span>

<span data-ttu-id="ce22e-144">Az ExpressRoute és a VPN virtuális hálózati átjárót ugyanazon a virtuális hálózaton hozza létre.</span><span class="sxs-lookup"><span data-stu-id="ce22e-144">Create the ExpressRoute virtual network gateway and the VPN virtual network gateway in the same VNet.</span></span> <span data-ttu-id="ce22e-145">Ez azt jelenti, hogy ugyanazon a *GatewaySubnet* nevű alhálózaton kell osztozniuk.</span><span class="sxs-lookup"><span data-stu-id="ce22e-145">This means that they should share the same subnet named *GatewaySubnet*.</span></span>

<span data-ttu-id="ce22e-146">Ha a virtuális hálózat már tartalmaz egy *GatewaySubnet* nevű alhálózatot, gondoskodjon róla, hogy az /27 vagy nagyobb címtérrel rendelkezzen.</span><span class="sxs-lookup"><span data-stu-id="ce22e-146">If the VNet already includes a subnet named *GatewaySubnet*, ensure that it has a /27 or larger address space.</span></span> <span data-ttu-id="ce22e-147">Ha a meglévő alhálózat túl kicsi, távolítsa el az alábbi PowerShell-paranccsal:</span><span class="sxs-lookup"><span data-stu-id="ce22e-147">If the existing subnet is too small, use the following PowerShell command to remove the subnet:</span></span> 

```powershell
$vnet = Get-AzureRmVirtualNetworkGateway -Name <yourvnetname> -ResourceGroupName <yourresourcegroup>
Remove-AzureRmVirtualNetworkSubnetConfig -Name GatewaySubnet -VirtualNetwork $vnet
```

<span data-ttu-id="ce22e-148">Ha a virtuális hálózat nem tartalmaz **GatewaySubnet** nevű alhálózatot, hozzon létre egy újat az alábbi PowerShell-paranccsal:</span><span class="sxs-lookup"><span data-stu-id="ce22e-148">If the VNet does not contain a subnet named **GatewaySubnet**, create a new one using the following Powershell command:</span></span>

```powershell
$vnet = Get-AzureRmVirtualNetworkGateway -Name <yourvnetname> -ResourceGroupName <yourresourcegroup>
Add-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet -AddressPrefix "10.200.255.224/27"
$vnet = Set-AzureRmVirtualNetwork -VirtualNetwork $vnet
```

### <a name="vpn-and-expressroute-gateways"></a><span data-ttu-id="ce22e-149">VPN- és ExpressRoute-átjárók</span><span class="sxs-lookup"><span data-stu-id="ce22e-149">VPN and ExpressRoute gateways</span></span>

<span data-ttu-id="ce22e-150">Gondoskodjon róla, hogy a cég megfeleljen az [ExpressRoute előfeltételeinek][expressroute-prereq], mert ezek hiányában nem tud az Azure-hoz csatlakozni.</span><span class="sxs-lookup"><span data-stu-id="ce22e-150">Verify that your organization meets the [ExpressRoute prerequisite requirements][expressroute-prereq] for connecting to Azure.</span></span>

<span data-ttu-id="ce22e-151">Ha már rendelkezik VPN virtuális hálózati átjáróval az Azure-beli virtuális hálózaton, távolítsa el az alábbi PowerShell-paranccsal:</span><span class="sxs-lookup"><span data-stu-id="ce22e-151">If you already have a VPN virtual network gateway in your Azure VNet, use the following  Powershell command to remove it:</span></span>

```powershell
Remove-AzureRmVirtualNetworkGateway -Name <yourgatewayname> -ResourceGroupName <yourresourcegroup>
```

<span data-ttu-id="ce22e-152">Az ExpressRoute-kapcsolat létrehozásához kövesse a [hibrid hálózati architektúra Azure ExpressRoute használatával történő megvalósításáról][implementing-expressroute] szóló cikk utasításait.</span><span class="sxs-lookup"><span data-stu-id="ce22e-152">Follow the instructions in [Implementing a hybrid network architecture with Azure ExpressRoute][implementing-expressroute] to establish your ExpressRoute connection.</span></span>

<span data-ttu-id="ce22e-153">A VPN virtuális hálózati átjárókapcsolat létrehozásához kövesse a [hibrid hálózati architektúra Azure és helyszíni VPN használatával történő megvalósításáról][implementing-vpn] szóló cikk utasításait.</span><span class="sxs-lookup"><span data-stu-id="ce22e-153">Follow the instructions in [Implementing a hybrid network architecture with Azure and On-premises VPN][implementing-vpn] to establish your VPN virtual network gateway connection.</span></span>

<span data-ttu-id="ce22e-154">A virtuális hálózati átjárókapcsolatok létrehozását követően tesztelje a környezetet az alábbi lépéseket követve:</span><span class="sxs-lookup"><span data-stu-id="ce22e-154">After you have established the virtual network gateway connections, test the environment as follows:</span></span>

1. <span data-ttu-id="ce22e-155">Ellenőrizze, hogy tud-e kapcsolódni a helyszíni hálózatból az Azure-beli virtuális hálózathoz.</span><span class="sxs-lookup"><span data-stu-id="ce22e-155">Make sure you can connect from your on-premises network to your Azure VNet.</span></span>
2. <span data-ttu-id="ce22e-156">Kérje meg a szolgáltatót az ExpressRoute-kapcsolat leállítására tesztelés céljából.</span><span class="sxs-lookup"><span data-stu-id="ce22e-156">Contact your provider to stop ExpressRoute connectivity for testing.</span></span>
3. <span data-ttu-id="ce22e-157">Ellenőrizze, hogy továbbra is tud-e kapcsolódni a helyszíni hálózatból az Azure-beli virtuális hálózathoz a VPN virtuális hálózati átjárókapcsolat használatával.</span><span class="sxs-lookup"><span data-stu-id="ce22e-157">Verify that you can still connect from your on-premises network to your Azure VNet using the VPN virtual network gateway connection.</span></span>
4. <span data-ttu-id="ce22e-158">Kérje meg a szolgáltatót az ExpressRoute-kapcsolat újbóli létrehozására.</span><span class="sxs-lookup"><span data-stu-id="ce22e-158">Contact your provider to reestablish ExpressRoute connectivity.</span></span>

## <a name="considerations"></a><span data-ttu-id="ce22e-159">Megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="ce22e-159">Considerations</span></span>

<span data-ttu-id="ce22e-160">Az ExpressRoute-tal kapcsolatos szempontokat a [hibrid hálózati architektúra Azure ExpressRoute használatával történő megvalósításáról][guidance-expressroute] szóló cikkben tekintheti át.</span><span class="sxs-lookup"><span data-stu-id="ce22e-160">For ExpressRoute considerations, see the [Implementing a Hybrid Network Architecture with Azure ExpressRoute][guidance-expressroute] guidance.</span></span>

<span data-ttu-id="ce22e-161">A helyek közötti VPN-nel kapcsolatos szempontokat a [hibrid hálózati architektúra Azure és helyszíni VPN használatával történő megvalósításáról][guidance-vpn] szóló cikkben tekintheti át.</span><span class="sxs-lookup"><span data-stu-id="ce22e-161">For site-to-site VPN considerations, see the [Implementing a Hybrid Network Architecture with Azure and On-premises VPN][guidance-vpn] guidance.</span></span>

<span data-ttu-id="ce22e-162">Az Azure-biztonsággal kapcsolatos általános szempontokat [a Microsoft Cloud Services szolgáltatásokkal és a hálózati biztonsággal][best-practices-security] foglalkozó cikkben találja.</span><span class="sxs-lookup"><span data-stu-id="ce22e-162">For general Azure security considerations, see [Microsoft cloud services and network security][best-practices-security].</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="ce22e-163">A megoldás üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="ce22e-163">Deploy the solution</span></span>

<span data-ttu-id="ce22e-164">**Előfeltételek.**</span><span class="sxs-lookup"><span data-stu-id="ce22e-164">**Prequisites.**</span></span> <span data-ttu-id="ce22e-165">Rendelkeznie kell egy megfelelő hálózati berendezéssel konfigurált meglévő helyszíni infrastruktúrával.</span><span class="sxs-lookup"><span data-stu-id="ce22e-165">You must have an existing on-premises infrastructure already configured with a suitable network appliance.</span></span>

<span data-ttu-id="ce22e-166">A megoldás üzembe helyezéséhez hajtsa végre az alábbi lépéseket.</span><span class="sxs-lookup"><span data-stu-id="ce22e-166">To deploy the solution, perform the following steps.</span></span>

1. <span data-ttu-id="ce22e-167">Kattintson az alábbi gombra:</span><span class="sxs-lookup"><span data-stu-id="ce22e-167">Click the button below:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute-vpn-failover%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. <span data-ttu-id="ce22e-168">Várja meg, amíg a hivatkozás megnyílik az Azure Portalon, majd kövesse az alábbi lépéseket:</span><span class="sxs-lookup"><span data-stu-id="ce22e-168">Wait for the link to open in the Azure portal, then follow these steps:</span></span>   
   * <span data-ttu-id="ce22e-169">Az **Erőforráscsoport** neve már meg van adva a paraméterfájlban, ezért válassza az **Új létrehozása** lehetőséget és a szövegmezőbe írja az `ra-hybrid-vpn-er-rg` karakterláncot.</span><span class="sxs-lookup"><span data-stu-id="ce22e-169">The **Resource group** name is already defined in the parameter file, so select **Create New** and enter `ra-hybrid-vpn-er-rg` in the text box.</span></span>
   * <span data-ttu-id="ce22e-170">Válassza ki a régiót a **Hely** legördülő listából.</span><span class="sxs-lookup"><span data-stu-id="ce22e-170">Select the region from the **Location** drop down box.</span></span>
   * <span data-ttu-id="ce22e-171">Ne szerkessze a **Sablon gyökér szintű URI-je** vagy a **Paraméter gyökér szintű URI-je** szövegmezőt.</span><span class="sxs-lookup"><span data-stu-id="ce22e-171">Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.</span></span>
   * <span data-ttu-id="ce22e-172">Tekintse át a használati feltételeket, majd kattintson az **Elfogadom a fenti feltételeket** lehetőségre.</span><span class="sxs-lookup"><span data-stu-id="ce22e-172">Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.</span></span>
   * <span data-ttu-id="ce22e-173">Kattintson a **Vásárlás** gombra.</span><span class="sxs-lookup"><span data-stu-id="ce22e-173">Click the **Purchase** button.</span></span>
3. <span data-ttu-id="ce22e-174">Várjon, amíg az üzembe helyezés befejeződik.</span><span class="sxs-lookup"><span data-stu-id="ce22e-174">Wait for the deployment to complete.</span></span>
4. <span data-ttu-id="ce22e-175">Kattintson az alábbi gombra:</span><span class="sxs-lookup"><span data-stu-id="ce22e-175">Click the button below:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fhybrid-networking%2Fexpressroute-vpn-failover%2Fazuredeploy-expressRouteCircuit.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
5. <span data-ttu-id="ce22e-176">Várja meg, amíg a hivatkozás megnyílik az Azure Portalon, majd kövesse az alábbi lépéseket:</span><span class="sxs-lookup"><span data-stu-id="ce22e-176">Wait for the link to open in the Azure portal, then enter then follow these steps:</span></span>
   * <span data-ttu-id="ce22e-177">Válassza az **Erőforráscsoport** szakasz **Meglévő használata** elemét, és írja be az `ra-hybrid-vpn-er-rg` karakterláncot a szövegmezőbe.</span><span class="sxs-lookup"><span data-stu-id="ce22e-177">Select **Use existing** in the **Resource group** section and enter `ra-hybrid-vpn-er-rg` in the text box.</span></span>
   * <span data-ttu-id="ce22e-178">Válassza ki a régiót a **Hely** legördülő listából.</span><span class="sxs-lookup"><span data-stu-id="ce22e-178">Select the region from the **Location** drop down box.</span></span>
   * <span data-ttu-id="ce22e-179">Ne szerkessze a **Sablon gyökér szintű URI-je** vagy a **Paraméter gyökér szintű URI-je** szövegmezőt.</span><span class="sxs-lookup"><span data-stu-id="ce22e-179">Do not edit the **Template Root Uri** or the **Parameter Root Uri** text boxes.</span></span>
   * <span data-ttu-id="ce22e-180">Tekintse át a használati feltételeket, majd kattintson az **Elfogadom a fenti feltételeket** lehetőségre.</span><span class="sxs-lookup"><span data-stu-id="ce22e-180">Review the terms and conditions, then click the **I agree to the terms and conditions stated above** checkbox.</span></span>
   * <span data-ttu-id="ce22e-181">Kattintson a **Vásárlás** gombra.</span><span class="sxs-lookup"><span data-stu-id="ce22e-181">Click the **Purchase** button.</span></span>

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
[visio-download]: https://archcenter.blob.core.windows.net/cdn/hybrid-network-architectures.vsdx
[0]: ./images/expressroute-vpn-failover.png "ExpressRoute és VPN-átjáró használatával megvalósított magas rendelkezésre állású hibrid hálózat architektúrája"
