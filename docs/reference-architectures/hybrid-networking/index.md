---
title: Helyszíni hálózat csatlakoztatása az Azure-hoz
titleSuffix: Azure Reference Architectures
description: Referenciaarchitektúrák összehasonlítása egy helyszíni hálózat az Azure-hoz való csatlakoztatásához.
author: telmosampaio
ms.date: 07/02/2018
ms.openlocfilehash: de509b6d95805f4fc871f6dbd76a87d2c0bec6f1
ms.sourcegitcommit: 88a68c7e9b6b772172b7faa4b9fd9c061a9f7e9d
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/08/2018
ms.locfileid: "53119914"
---
# <a name="choose-a-solution-for-connecting-an-on-premises-network-to-azure"></a><span data-ttu-id="93429-103">Válasszon egy megoldást a helyszíni hálózat Azure-hoz való csatlakoztatásához</span><span class="sxs-lookup"><span data-stu-id="93429-103">Choose a solution for connecting an on-premises network to Azure</span></span>

<span data-ttu-id="93429-104">Ez a cikk a helyszíni hálózat egy Azure-beli virtuális hálózathoz (VNethez) történő csatlakoztatásának lehetőségeit hasonlítja össze.</span><span class="sxs-lookup"><span data-stu-id="93429-104">This article compares options for connecting an on-premises network to an Azure Virtual Network (VNet).</span></span> <span data-ttu-id="93429-105">Mindegyik lehetőséghez elérhető egy referenciaarchitektúra.</span><span class="sxs-lookup"><span data-stu-id="93429-105">For each option, a more detailed reference architecture is available.</span></span>

## <a name="vpn-connection"></a><span data-ttu-id="93429-106">VPN-kapcsolat</span><span class="sxs-lookup"><span data-stu-id="93429-106">VPN connection</span></span>

<span data-ttu-id="93429-107">A [VPN-átjáró](/azure/vpn-gateway/vpn-gateway-about-vpngateways) a virtuális hálózati átjárók egyik típusa, amely titkosított adatforgalmat továbbít egy Azure-beli virtuális hálózat és egy helyszíni hely között.</span><span class="sxs-lookup"><span data-stu-id="93429-107">A [VPN gateway](/azure/vpn-gateway/vpn-gateway-about-vpngateways) is a type of virtual network gateway that sends encrypted traffic between an Azure virtual network and an on-premises location.</span></span> <span data-ttu-id="93429-108">A titkosított adatforgalom a nyilvános interneten halad át.</span><span class="sxs-lookup"><span data-stu-id="93429-108">The encrypted traffic goes over the public Internet.</span></span>

<span data-ttu-id="93429-109">Ez az architektúra olyan hibrid alkalmazások esetében megfelelő, ahol a helyszíni hardver és a felhő közötti forgalom várhatóan alacsony, illetve ha a felhő nyújtotta rugalmasságért és feldolgozási kapacitásért cserébe elfogadhatónak tart némileg nagyobb késést.</span><span class="sxs-lookup"><span data-stu-id="93429-109">This architecture is suitable for hybrid applications where the traffic between on-premises hardware and the cloud is likely to be light, or you are willing to trade slightly extended latency for the flexibility and processing power of the cloud.</span></span>

<span data-ttu-id="93429-110">**Előnyök**</span><span class="sxs-lookup"><span data-stu-id="93429-110">**Benefits**</span></span>

- <span data-ttu-id="93429-111">Egyszerűen konfigurálható.</span><span class="sxs-lookup"><span data-stu-id="93429-111">Simple to configure.</span></span>

<span data-ttu-id="93429-112">**Problémák**</span><span class="sxs-lookup"><span data-stu-id="93429-112">**Challenges**</span></span>

- <span data-ttu-id="93429-113">Helyszíni VPN-eszközt igényel.</span><span class="sxs-lookup"><span data-stu-id="93429-113">Requires an on-premises VPN device.</span></span>
- <span data-ttu-id="93429-114">Bár a Microsoft 99,9%-os rendelkezésre állást biztosít minden VPN-átjáróhoz, az [SLA](https://azure.microsoft.com/support/legal/sla/vpn-gateway/) csak a VPN-átjáróra vonatkozik, az átjáróval létesített hálózati kapcsolatra nem.</span><span class="sxs-lookup"><span data-stu-id="93429-114">Although Microsoft guarantees 99.9% availability for each VPN Gateway, this [SLA](https://azure.microsoft.com/support/legal/sla/vpn-gateway/) only covers the VPN gateway, and not your network connection to the gateway.</span></span>
- <span data-ttu-id="93429-115">Az Azure VPN Gatewayen keresztül létesített VPN-kapcsolat jelenleg legfeljebb 1,25 Gbps sávszélességet támogat.</span><span class="sxs-lookup"><span data-stu-id="93429-115">A VPN connection over Azure VPN Gateway currently supports a maximum of 1.25 Gbps bandwidth.</span></span> <span data-ttu-id="93429-116">Előfordulhat, hogy az Azure virtuális hálózatot particionálnia kell több VPN-kapcsolat között, ha várhatóan át fogja lépni ezt az átviteli sebességet.</span><span class="sxs-lookup"><span data-stu-id="93429-116">You may need to partition your Azure virtual network across multiple VPN connections if you expect to exceed this throughput.</span></span>

<span data-ttu-id="93429-117">**Referenciaarchitektúra**</span><span class="sxs-lookup"><span data-stu-id="93429-117">**Reference architecture**</span></span>

- [<span data-ttu-id="93429-118">Hibrid hálózat VPN-átjáróval</span><span class="sxs-lookup"><span data-stu-id="93429-118">Hybrid network with VPN gateway</span></span>](./vpn.md)

## <a name="azure-expressroute-connection"></a><span data-ttu-id="93429-119">Azure ExpressRoute-kapcsolat</span><span class="sxs-lookup"><span data-stu-id="93429-119">Azure ExpressRoute connection</span></span>

<span data-ttu-id="93429-120">Az [ExpressRoute](/azure/expressroute/)-kapcsolatok dedikált, privát kapcsolatot használnak egy külső kapcsolatszolgáltatón keresztül.</span><span class="sxs-lookup"><span data-stu-id="93429-120">[ExpressRoute](/azure/expressroute/) connections use a private, dedicated connection through a third-party connectivity provider.</span></span> <span data-ttu-id="93429-121">A privát kapcsolat kiterjeszti a helyszíni hálózatot az Azure-ba.</span><span class="sxs-lookup"><span data-stu-id="93429-121">The private connection extends your on-premises network into Azure.</span></span> 

<span data-ttu-id="93429-122">Ez az architektúra megfelelő az olyan nagy méretű, kritikus fontosságú számítási feladatokat futtató hibrid alkalmazásokhoz, amelyek nagyfokú méretezhetőséget igényelnek.</span><span class="sxs-lookup"><span data-stu-id="93429-122">This architecture is suitable for hybrid applications running large-scale, mission-critical workloads that require a high degree of scalability.</span></span> 

<span data-ttu-id="93429-123">**Előnyök**</span><span class="sxs-lookup"><span data-stu-id="93429-123">**Benefits**</span></span>

- <span data-ttu-id="93429-124">Sokkal nagyobb sávszélesség áll rendelkezésre; a kapcsolatszolgáltatótól függően legfeljebb 10 Gbps.</span><span class="sxs-lookup"><span data-stu-id="93429-124">Much higher bandwidth available; up to 10 Gbps depending on the connectivity provider.</span></span>
- <span data-ttu-id="93429-125">Támogatja a dinamikus sávszélesség-méretezést, amellyel alacsony igénybevétel esetén csökkenthetők a költségek.</span><span class="sxs-lookup"><span data-stu-id="93429-125">Supports dynamic scaling of bandwidth to help reduce costs during periods of lower demand.</span></span> <span data-ttu-id="93429-126">Ezt a lehetőséget azonban nem minden kapcsolatszolgáltató támogatja.</span><span class="sxs-lookup"><span data-stu-id="93429-126">However, not all connectivity providers have this option.</span></span>
- <span data-ttu-id="93429-127">A kapcsolat szolgáltatójától függően lehetővé teheti a vállalat számára az országos felhőkhöz való hozzáférést.</span><span class="sxs-lookup"><span data-stu-id="93429-127">May allow your organization direct access to national clouds, depending on the connectivity provider.</span></span>
- <span data-ttu-id="93429-128">99,9%-os rendelkezésre állási SLA a teljes kapcsolatra vonatkozóan.</span><span class="sxs-lookup"><span data-stu-id="93429-128">99.9% availability SLA across the entire connection.</span></span>

<span data-ttu-id="93429-129">**Problémák**</span><span class="sxs-lookup"><span data-stu-id="93429-129">**Challenges**</span></span>

- <span data-ttu-id="93429-130">Beállítása bonyolult lehet.</span><span class="sxs-lookup"><span data-stu-id="93429-130">Can be complex to set up.</span></span> <span data-ttu-id="93429-131">ExpressRoute-kapcsolat létrehozásához külső kapcsolatszolgáltatóra van szükség.</span><span class="sxs-lookup"><span data-stu-id="93429-131">Creating an ExpressRoute connection requires working with a third-party connectivity provider.</span></span> <span data-ttu-id="93429-132">A hálózati kapcsolat kiépítése a szolgáltató feladata.</span><span class="sxs-lookup"><span data-stu-id="93429-132">The provider is responsible for provisioning the network connection.</span></span>
- <span data-ttu-id="93429-133">Nagy sebességű helyszíni útválasztókat igényel.</span><span class="sxs-lookup"><span data-stu-id="93429-133">Requires high-bandwidth routers on-premises.</span></span>

<span data-ttu-id="93429-134">**Referenciaarchitektúra**</span><span class="sxs-lookup"><span data-stu-id="93429-134">**Reference architecture**</span></span>

- [<span data-ttu-id="93429-135">Hibrid hálózat ExpressRoute-tal</span><span class="sxs-lookup"><span data-stu-id="93429-135">Hybrid network with ExpressRoute</span></span>](./expressroute.md)

## <a name="expressroute-with-vpn-failover"></a><span data-ttu-id="93429-136">ExpressRoute VPN-alapú feladatátvétellel</span><span class="sxs-lookup"><span data-stu-id="93429-136">ExpressRoute with VPN failover</span></span>

<span data-ttu-id="93429-137">Ez a lehetőség lényegében az előző kettő kombinációja. Normál körülmények között ExpressRoute-ot használ, ha azonban az ExpressRoute-kapcsolatcsoportban megszakad a kapcsolat, akkor egy VPN-kapcsolat veszi át a feladatokat.</span><span class="sxs-lookup"><span data-stu-id="93429-137">This options combines the previous two, using ExpressRoute in normal conditions, but failing over to a VPN connection if there is a loss of connectivity in the ExpressRoute circuit.</span></span>

<span data-ttu-id="93429-138">Ez az architektúra olyan hibrid alkalmazásokhoz megfelelő, amelyek az ExpressRoute által nyújtott nagyobb sávszélességet és a magas rendelkezésre állású hálózati kapcsolatot is igénylik.</span><span class="sxs-lookup"><span data-stu-id="93429-138">This architecture is suitable for hybrid applications that need the higher bandwidth of ExpressRoute, and also require highly available network connectivity.</span></span> 

<span data-ttu-id="93429-139">**Előnyök**</span><span class="sxs-lookup"><span data-stu-id="93429-139">**Benefits**</span></span>

- <span data-ttu-id="93429-140">Magas rendelkezésre állás az ExpressRoute-kapcsolatcsoport meghibásodása esetén, azonban a tartalék kapcsolat egy alacsonyabb sávszélességű hálózaton fut.</span><span class="sxs-lookup"><span data-stu-id="93429-140">High availability if the ExpressRoute circuit fails, although the fallback connection is on a lower bandwidth network.</span></span>

<span data-ttu-id="93429-141">**Problémák**</span><span class="sxs-lookup"><span data-stu-id="93429-141">**Challenges**</span></span>

- <span data-ttu-id="93429-142">Konfigurálása bonyolult.</span><span class="sxs-lookup"><span data-stu-id="93429-142">Complex to configure.</span></span> <span data-ttu-id="93429-143">Egy VPN-kapcsolatot és egy ExpressRoute-kapcsolatcsoportot is be kell állítania.</span><span class="sxs-lookup"><span data-stu-id="93429-143">You need to set up both a VPN connection and an ExpressRoute circuit.</span></span>
- <span data-ttu-id="93429-144">Redundáns hardvert (VPN-berendezéseket) igényel, valamint redundáns Azure VPN Gateway-kapcsolatot is, amelynek külön díja van.</span><span class="sxs-lookup"><span data-stu-id="93429-144">Requires redundant hardware (VPN appliances), and a redundant Azure VPN Gateway connection for which you pay charges.</span></span>

<span data-ttu-id="93429-145">**Referenciaarchitektúra**</span><span class="sxs-lookup"><span data-stu-id="93429-145">**Reference architecture**</span></span>

- [<span data-ttu-id="93429-146">Hibrid hálózat ExpressRoute-tal és VPN-feladatátvétellel</span><span class="sxs-lookup"><span data-stu-id="93429-146">Hybrid network with ExpressRoute and VPN failover</span></span>](./expressroute-vpn-failover.md)

## <a name="hub-spoke-network-topology"></a><span data-ttu-id="93429-147">Küllős hálózati topológia</span><span class="sxs-lookup"><span data-stu-id="93429-147">Hub-spoke network topology</span></span>

<span data-ttu-id="93429-148">A küllős hálózati topológiával elkülöníthetők a számítási feladatok, ugyanakkor megoszthatók az olyan szolgáltatások, mint az identitáskezelés és a biztonság.</span><span class="sxs-lookup"><span data-stu-id="93429-148">A hub-spoke network topology is a way to isolate workloads while sharing services such as identity and security.</span></span> <span data-ttu-id="93429-149">Az agy egy virtuális hálózat (VNet) az Azure-ban, amely központi kapcsolódási pontként szolgál a helyszíni hálózathoz.</span><span class="sxs-lookup"><span data-stu-id="93429-149">The hub is a virtual network (VNet) in Azure that acts as a central point of connectivity to your on-premises network.</span></span> <span data-ttu-id="93429-150">A küllők az agyhoz kapcsolódó virtuális hálózatok.</span><span class="sxs-lookup"><span data-stu-id="93429-150">The spokes are VNets that peer with the hub.</span></span> <span data-ttu-id="93429-151">A közös szolgáltatások üzembe helyezése az agyon történik, az egyes számítási feladatokat a küllőkön vannak.</span><span class="sxs-lookup"><span data-stu-id="93429-151">Shared services are deployed in the hub, while individual workloads are deployed as spokes.</span></span>

<span data-ttu-id="93429-152">**Referenciaarchitektúrák**</span><span class="sxs-lookup"><span data-stu-id="93429-152">**Reference architectures**</span></span>

- [<span data-ttu-id="93429-153">Küllős topológia</span><span class="sxs-lookup"><span data-stu-id="93429-153">Hub-spoke topology</span></span>](./hub-spoke.md)
- [<span data-ttu-id="93429-154">Küllős topológia közös szolgáltatásokkal</span><span class="sxs-lookup"><span data-stu-id="93429-154">Hub-spoke with shared services</span></span>](./shared-services.md)
