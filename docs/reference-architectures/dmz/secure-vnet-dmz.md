---
title: Implementálhat DMZ-t az Azure és az Internet között
titleSuffix: Azure Reference Architectures
description: Interneteléréssel rendelkező, biztonságos hibrid hálózati architektúra megvalósítása az Azure-ban.
author: telmosampaio
ms.date: 10/22/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seodec18, networking
ms.openlocfilehash: 45ae8de1138b738fdfb42bdf57402711e1be6ebb
ms.sourcegitcommit: 14226018a058e199523106199be9c07f6a3f8592
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/31/2019
ms.locfileid: "55482909"
---
# <a name="implement-a-dmz-between-azure-and-the-internet"></a><span data-ttu-id="d8aa0-103">Implementálhat DMZ-t az Azure és az Internet között</span><span class="sxs-lookup"><span data-stu-id="d8aa0-103">Implement a DMZ between Azure and the Internet</span></span>

<span data-ttu-id="d8aa0-104">Ez a referenciaarchitektúra egy biztonságos hibrid hálózatot mutat be, amely kiterjeszti a helyszíni hálózatot az Azure-ba, és internetes forgalmat is fogad.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-104">This reference architecture shows a secure hybrid network that extends an on-premises network to Azure and also accepts Internet traffic.</span></span> <span data-ttu-id="d8aa0-105">[**A megoldás üzembe helyezése.**](#deploy-the-solution)</span><span class="sxs-lookup"><span data-stu-id="d8aa0-105">[**Deploy this solution**](#deploy-the-solution).</span></span>

> [!NOTE]
> <span data-ttu-id="d8aa0-106">Ebben a forgatókönyvben használatával is elvégezhető [Azure tűzfal](/azure/firewall/), egy felhőalapú hálózati biztonsági szolgáltatás.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-106">This scenario can also be accomplished using [Azure Firewall](/azure/firewall/), a cloud-based network security service.</span></span>

![Biztonságos hibrid hálózati architektúra](./images/dmz-public.png)

<span data-ttu-id="d8aa0-108">*Töltse le az architektúra [Visio-fájlját][visio-download].*</span><span class="sxs-lookup"><span data-stu-id="d8aa0-108">*Download a [Visio file][visio-download] of this architecture.*</span></span>

<span data-ttu-id="d8aa0-109">Ez a referenciaarchitektúra a [DMZ az Azure és a helyszíni adatközpont közötti implementálásával kapcsolatos][implementing-a-secure-hybrid-network-architecture] cikkben ismertetett architektúrát bővíti ki.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-109">This reference architecture extends the architecture described in [Implementing a DMZ between Azure and your on-premises datacenter][implementing-a-secure-hybrid-network-architecture].</span></span> <span data-ttu-id="d8aa0-110">Hozzáadja a nyilvános DMZ-t, amely kezeli az internetes forgalmat a helyszíni hálózat érkező forgalmat kezelő privát DMZ mellett.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-110">It adds a public DMZ that handles Internet traffic, in addition to the private DMZ that handles traffic from the on-premises network.</span></span>

<span data-ttu-id="d8aa0-111">Az architektúra gyakori használati módjai többek között a következők:</span><span class="sxs-lookup"><span data-stu-id="d8aa0-111">Typical uses for this architecture include:</span></span>

- <span data-ttu-id="d8aa0-112">Hibrid alkalmazások, amelyekben a számítási feladatok részben a helyszínen, részben pedig az Azure-ban futnak.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-112">Hybrid applications where workloads run partly on-premises and partly in Azure.</span></span>
- <span data-ttu-id="d8aa0-113">Azure-infrastruktúra, amely átirányítja a helyszínről és az internetről bejövő forgalmat.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-113">Azure infrastructure that routes incoming traffic from on-premises and the Internet.</span></span>

## <a name="architecture"></a><span data-ttu-id="d8aa0-114">Architektúra</span><span class="sxs-lookup"><span data-stu-id="d8aa0-114">Architecture</span></span>

<span data-ttu-id="d8aa0-115">Az architektúra a következőkben leírt összetevőkből áll.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-115">The architecture consists of the following components.</span></span>

- <span data-ttu-id="d8aa0-116">**Nyilvános IP-cím (PIP)**.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-116">**Public IP address (PIP)**.</span></span> <span data-ttu-id="d8aa0-117">A nyilvános végpont IP-címe.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-117">The IP address of the public endpoint.</span></span> <span data-ttu-id="d8aa0-118">Az internethez csatlakozó külső felhasználók ezen a címen keresztül férhetnek hozzá a rendszerhez.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-118">External users connected to the Internet can access the system through this address.</span></span>
- <span data-ttu-id="d8aa0-119">**Hálózati virtuális berendezés (NVA)**.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-119">**Network virtual appliance (NVA)**.</span></span> <span data-ttu-id="d8aa0-120">Ez az architektúra tartalmaz egy különálló NVA-készletet az internetről érkező forgalom számára.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-120">This architecture includes a separate pool of NVAs for traffic originating on the Internet.</span></span>
- <span data-ttu-id="d8aa0-121">**Azure Load Balancer**.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-121">**Azure load balancer**.</span></span> <span data-ttu-id="d8aa0-122">Az internetről érkező összes kérés a terheléselosztón halad keresztül, amely elosztja azokat a nyilvános DMZ-n található NVA-k között.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-122">All incoming requests from the Internet pass through the load balancer and are distributed to the NVAs in the public DMZ.</span></span>
- <span data-ttu-id="d8aa0-123">**Nyilvános DMZ bejövő alhálózata**.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-123">**Public DMZ inbound subnet**.</span></span> <span data-ttu-id="d8aa0-124">Ez az alhálózat fogadja az Azure Load Balancer kéréseit.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-124">This subnet accepts requests from the Azure load balancer.</span></span> <span data-ttu-id="d8aa0-125">Ezután átadja a bejövő kéréseket a nyilvános DMZ egyik NVA-jának.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-125">Incoming requests are passed to one of the NVAs in the public DMZ.</span></span>
- <span data-ttu-id="d8aa0-126">**Nyilvános DMZ kimenő alhálózata**.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-126">**Public DMZ outbound subnet**.</span></span> <span data-ttu-id="d8aa0-127">Az NVA által jóváhagyott kérések ezen az alhálózaton haladnak át a webes réteg belső terheléselosztója felé.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-127">Requests that are approved by the NVA pass through this subnet to the internal load balancer for the web tier.</span></span>

## <a name="recommendations"></a><span data-ttu-id="d8aa0-128">Javaslatok</span><span class="sxs-lookup"><span data-stu-id="d8aa0-128">Recommendations</span></span>

<span data-ttu-id="d8aa0-129">Az alábbi javaslatok a legtöbb forgatókönyvre vonatkoznak.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-129">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="d8aa0-130">Kövesse ezeket a javaslatokat, ha nincsenek ezeket felülíró követelményei.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-130">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="nva-recommendations"></a><span data-ttu-id="d8aa0-131">Hálózati virtuális készülékre vonatkozó javaslatok</span><span class="sxs-lookup"><span data-stu-id="d8aa0-131">NVA recommendations</span></span>

<span data-ttu-id="d8aa0-132">Használjon egy NVA-készletet az internetről érkező forgalomhoz, és egy másikat a helyszínről érkezőhöz.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-132">Use one set of NVAs for traffic originating on the Internet, and another for traffic originating on-premises.</span></span> <span data-ttu-id="d8aa0-133">Ha egyetlen NVA-készlet használ mindkettőhöz, az biztonsági kockázatot jelent, mivel nincs biztonsági határ a két fajta hálózati forgalom között.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-133">Using only one set of NVAs for both is a security risk, because it provides no security perimeter between the two sets of network traffic.</span></span> <span data-ttu-id="d8aa0-134">A különálló NVA-k használata csökkenti az ellenőrző biztonsági szabályok összetettségét, és egyértelművé teszi, hogy melyik szabály vonatkozik az egyes bejövő hálózati kérésekre.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-134">Using separate NVAs reduces the complexity of checking security rules, and makes it clear which rules correspond to each incoming network request.</span></span> <span data-ttu-id="d8aa0-135">Az egyik NVA-készlet csak az internetes forgalomra, a másik pedig csak a helyszínről érkező forgalomra vonatkozó szabályokat implementál.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-135">One set of NVAs implements rules for Internet traffic only, while another set of NVAs implement rules for on-premises traffic only.</span></span>

<span data-ttu-id="d8aa0-136">Alkalmazzon egy 7. rétegbeli NVA-t, amely megszakítja az alkalmazások kapcsolatát az NVA szintjén, és fenntartja a kompatibilitást a háttérszintekkel.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-136">Include a layer-7 NVA to terminate application connections at the NVA level and maintain compatibility with the backend tiers.</span></span> <span data-ttu-id="d8aa0-137">Ez garantálja a szimmetrikus kapcsolatokat, ahol a háttérbeli rétegekből érkező válaszforgalom az NVA-n halad keresztül.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-137">This guarantees symmetric connectivity where response traffic from the backend tiers returns through the NVA.</span></span>

### <a name="public-load-balancer-recommendations"></a><span data-ttu-id="d8aa0-138">Nyilvános terheléselosztóra vonatkozó javaslatok</span><span class="sxs-lookup"><span data-stu-id="d8aa0-138">Public load balancer recommendations</span></span>

<span data-ttu-id="d8aa0-139">A skálázhatóság és rendelkezésre állás érdekében a nyilvános DMZ NVA-it egy [rendelkezésre állási csoportban][availability-set] helyezze üzembe, és egy [internetkapcsolattal rendelkező terheléselosztó][load-balancer] használatával ossza el az internetes kéréseket a rendelkezésre állási csoport NVA-i között.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-139">For scalability and availability, deploy the public DMZ NVAs in an [availability set][availability-set] and use an [Internet facing load balancer][load-balancer] to distribute Internet requests across the NVAs in the availability set.</span></span>

<span data-ttu-id="d8aa0-140">Konfigurálja úgy a terheléselosztót, hogy az csak az internetes forgalomhoz szükséges portokból fogadjon el kéréseket.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-140">Configure the load balancer to accept requests only on the ports necessary for Internet traffic.</span></span> <span data-ttu-id="d8aa0-141">Korlátozza például a 80-as portra a beérkező HTTP-, valamint a 443-as portra a beérkező HTTPS-kéréseket.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-141">For example, restrict inbound HTTP requests to port 80 and inbound HTTPS requests to port 443.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="d8aa0-142">Méretezési szempontok</span><span class="sxs-lookup"><span data-stu-id="d8aa0-142">Scalability considerations</span></span>

<span data-ttu-id="d8aa0-143">Javasoljuk, hogy helyezzen el egy terheléselosztót a nyilvános DMZ előtt a kezdetekkor abban az esetben is, ha az architektúrában kezdetben egyetlen NVA szükséges a nyilvános DMZ-ben.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-143">Even if your architecture initially requires a single NVA in the public DMZ, we recommend putting a load balancer in front of the public DMZ from the beginning.</span></span> <span data-ttu-id="d8aa0-144">Ez megkönnyíti a több NVA-ra való skálázást a jövőben, ha szükséges.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-144">That will make it easier to scale to multiple NVAs in the future, if needed.</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="d8aa0-145">Rendelkezésre állási szempontok</span><span class="sxs-lookup"><span data-stu-id="d8aa0-145">Availability considerations</span></span>

<span data-ttu-id="d8aa0-146">Az internetkapcsolattal rendelkező terheléselosztó megköveteli, hogy a nyilvános DMZ bejövő alhálózatán található minden NVA implementáljon egy [állapotmintát][lb-probe].</span><span class="sxs-lookup"><span data-stu-id="d8aa0-146">The Internet facing load balancer requires each NVA in the public DMZ inbound subnet to implement a [health probe][lb-probe].</span></span> <span data-ttu-id="d8aa0-147">A terheléselosztó nem elérhetőnek tekinti azokat az állapotmintákat, amelyek nem válaszolnak ezen a végponton, és a kéréseket a rendelkezésre állási csoport más NVA-i számára továbbítja.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-147">A health probe that fails to respond on this endpoint is considered to be unavailable, and the load balancer will direct requests to other NVAs in the same availability set.</span></span> <span data-ttu-id="d8aa0-148">Vegye figyelembe, hogy ha egyik NVA sem válaszol, az alkalmazás összeomlik, ezért fontos a monitorozás konfigurálása, amely értesíti a fejlesztési és üzemeltetési csapat tagjait, ha a kifogástalan NVA-k száma egy meghatározott küszöbérték alá esik.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-148">Note that if all NVAs fail to respond, your application will fail, so it's important to have monitoring configured to alert DevOps when the number of healthy NVA instances falls below a defined threshold.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="d8aa0-149">Felügyeleti szempontok</span><span class="sxs-lookup"><span data-stu-id="d8aa0-149">Manageability considerations</span></span>

<span data-ttu-id="d8aa0-150">Monitorozása és felügyelete az nva-k a nyilvános DMZ-ben a a felügyeleti alhálózaton található jumpbox kell elvégeznie.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-150">All monitoring and management for the NVAs in the public DMZ should be performed by the jumpbox in the management subnet.</span></span> <span data-ttu-id="d8aa0-151">A [DMZ az Azure és a helyszíni adatközpont közötti implementálásával kapcsolatos][implementing-a-secure-hybrid-network-architecture] cikkben ismertetett módon definiáljon egyetlen hálózati útvonalat a helyszíni hálózattól az átjárón keresztül a jumpboxig a hozzáférés korlátozása érdekében.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-151">As discussed in [Implementing a DMZ between Azure and your on-premises datacenter][implementing-a-secure-hybrid-network-architecture], define a single network route from the on-premises network through the gateway to the jumpbox, in order to restrict access.</span></span>

<span data-ttu-id="d8aa0-152">Ha a helyszíni hálózat és az Azure közötti átjárókapcsolat nem érhető el, a jumpboxot elérheti egy nyilvános IP-cím üzembe helyezésével és a jumpboxhoz való hozzáadásával, majd az internetről való bejelentkezéssel.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-152">If gateway connectivity from your on-premises network to Azure is down, you can still reach the jumpbox by deploying a public IP address, adding it to the jumpbox, and logging in from the Internet.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="d8aa0-153">Biztonsági szempontok</span><span class="sxs-lookup"><span data-stu-id="d8aa0-153">Security considerations</span></span>

<span data-ttu-id="d8aa0-154">Ez a referenciaarchitektúra többszintű biztonságot valósít meg:</span><span class="sxs-lookup"><span data-stu-id="d8aa0-154">This reference architecture implements multiple levels of security:</span></span>

- <span data-ttu-id="d8aa0-155">Az internetkapcsolattal rendelkező terheléselosztó átirányítja a kéréseket a nyilvános DMZ bejövő alhálózatán található NVA-khoz, kizárólag az alkalmazás számára szükséges portokon.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-155">The Internet facing load balancer directs requests to the NVAs in the inbound public DMZ subnet, and only on the ports necessary for the application.</span></span>
- <span data-ttu-id="d8aa0-156">A nyilvános DMZ bejövő és kimenő alhálózatára vonatkozó NSG-szabályok megakadályozzák az NVA-k feltörését az NSG-szabályokon kívül eső kérések blokkolásával.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-156">The NSG rules for the inbound and outbound public DMZ subnets prevent the NVAs from being compromised, by blocking requests that fall outside of the NSG rules.</span></span>
- <span data-ttu-id="d8aa0-157">Az NVA-k NAT-útválasztási konfigurációja átirányítja a 80-as és 443-as portokon bejövő kéréseket a webes rétegben lévő terheléselosztóhoz, de figyelmen hagyja az összes többi porton érkező kéréseket.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-157">The NAT routing configuration for the NVAs directs incoming requests on port 80 and port 443 to the web tier load balancer, but ignores requests on all other ports.</span></span>

<span data-ttu-id="d8aa0-158">Érdemes az összes porton érkező bejövő kérésekről naplót készíteni.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-158">You should log all incoming requests on all ports.</span></span> <span data-ttu-id="d8aa0-159">Rendszeresen ellenőrizze a naplókat, és fordítson különös figyelmet a várt paramétereken kívül eső kérésekre, mert ezek behatolási kísérleteket jelezhetnek.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-159">Regularly audit the logs, paying attention to requests that fall outside of expected parameters, as these may indicate intrusion attempts.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="d8aa0-160">A megoldás üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="d8aa0-160">Deploy the solution</span></span>

<span data-ttu-id="d8aa0-161">Az ezeknek a javaslatoknak a figyelembe vételével megvalósított referenciaarchitektúra egy üzemelő példánya elérhető a [GitHubon][github-folder].</span><span class="sxs-lookup"><span data-stu-id="d8aa0-161">A deployment for a reference architecture that implements these recommendations is available on [GitHub][github-folder].</span></span>

### <a name="prerequisites"></a><span data-ttu-id="d8aa0-162">Előfeltételek</span><span class="sxs-lookup"><span data-stu-id="d8aa0-162">Prerequisites</span></span>

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-resources"></a><span data-ttu-id="d8aa0-163">Erőforrások üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="d8aa0-163">Deploy resources</span></span>

1. <span data-ttu-id="d8aa0-164">Keresse meg a `/dmz/secure-vnet-dmz` referencia architektúrák GitHub-adattár mappát.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-164">Navigate to the `/dmz/secure-vnet-dmz` folder of the reference architectures GitHub repository.</span></span>

2. <span data-ttu-id="d8aa0-165">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="d8aa0-165">Run the following command:</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <region> -p onprem.json --deploy
    ```

3. <span data-ttu-id="d8aa0-166">Keresse meg a `/dmz/ssecure-vnet-hybrid` referencia architektúrák GitHub-adattár mappát.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-166">Navigate to the `/dmz/ssecure-vnet-hybrid` folder of the reference architectures GitHub repository.</span></span>

4. <span data-ttu-id="d8aa0-167">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="d8aa0-167">Run the following command:</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <region> -p secure-vnet-hybrid.json --deploy
    ```

### <a name="connect-the-on-premises-and-azure-gateways"></a><span data-ttu-id="d8aa0-168">A helyszíni és az Azure-átjárók csatlakoztatása</span><span class="sxs-lookup"><span data-stu-id="d8aa0-168">Connect the on-premises and Azure gateways</span></span>

<span data-ttu-id="d8aa0-169">Ebben a lépésben két helyi hálózati átjáró csatlakozik.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-169">In this step, you will connect the two local network gateways.</span></span>

1. <span data-ttu-id="d8aa0-170">Az Azure Portalon keresse meg a létrehozott erőforráscsoportot.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-170">In the Azure Portal, navigate to the resource group that you created.</span></span>

2. <span data-ttu-id="d8aa0-171">Keresse meg az erőforrást nevű `ra-vpn-vgw-pip` , és másolja az IP-cím látható a **áttekintése** panelen.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-171">Find the resource named `ra-vpn-vgw-pip` and copy the IP address shown in the **Overview** blade.</span></span>

3. <span data-ttu-id="d8aa0-172">Keresse meg az erőforrást nevű `onprem-vpn-lgw`.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-172">Find the resource named `onprem-vpn-lgw`.</span></span>

4. <span data-ttu-id="d8aa0-173">Kattintson a **konfigurációs** panelen.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-173">Click the **Configuration** blade.</span></span> <span data-ttu-id="d8aa0-174">A **IP-cím**, illessze be a 2. lépésben felírt IP-címet.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-174">Under **IP address**, paste in the IP address from step 2.</span></span>

    ![Az IP-cím mezőt képernyőképe](./images/local-net-gw.png)

5. <span data-ttu-id="d8aa0-176">Kattintson a **mentése** és várjon, amíg a művelet befejeződik.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-176">Click **Save** and wait for the operation to complete.</span></span> <span data-ttu-id="d8aa0-177">Körülbelül 5 percet is igénybe vehet.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-177">It can take about 5 minutes.</span></span>

6. <span data-ttu-id="d8aa0-178">Keresse meg az erőforrást nevű `onprem-vpn-gateway1-pip`.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-178">Find the resource named `onprem-vpn-gateway1-pip`.</span></span> <span data-ttu-id="d8aa0-179">Másolja ki az IP-cím látható a **áttekintése** panelen.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-179">Copy the IP address shown in the **Overview** blade.</span></span>

7. <span data-ttu-id="d8aa0-180">Keresse meg az erőforrást nevű `ra-vpn-lgw`.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-180">Find the resource named `ra-vpn-lgw`.</span></span>

8. <span data-ttu-id="d8aa0-181">Kattintson a **konfigurációs** panelen.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-181">Click the **Configuration** blade.</span></span> <span data-ttu-id="d8aa0-182">A **IP-cím**, illessze be a 6. lépésben felírt IP-címet.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-182">Under **IP address**, paste in the IP address from step 6.</span></span>

9. <span data-ttu-id="d8aa0-183">Kattintson a **mentése** és várjon, amíg a művelet befejeződik.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-183">Click **Save** and wait for the operation to complete.</span></span>

10. <span data-ttu-id="d8aa0-184">A kapcsolat ellenőrzéséhez nyissa meg a **kapcsolatok** minden átjáró paneljét.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-184">To verify the connection, go to the **Connections** blade for each gateway.</span></span> <span data-ttu-id="d8aa0-185">A állapotúnak kell lennie **csatlakoztatva**.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-185">The status should be **Connected**.</span></span>

### <a name="verify-that-network-traffic-reaches-the-web-tier"></a><span data-ttu-id="d8aa0-186">Győződjön meg arról, hogy a hálózati forgalom eléri a webes szint</span><span class="sxs-lookup"><span data-stu-id="d8aa0-186">Verify that network traffic reaches the web tier</span></span>

1. <span data-ttu-id="d8aa0-187">Az Azure Portalon keresse meg a létrehozott erőforráscsoportot.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-187">In the Azure Portal, navigate to the resource group that you created.</span></span>

2. <span data-ttu-id="d8aa0-188">Keresse meg az erőforrást nevű `pub-dmz-lb`, amely a terheléselosztó a nyilvános DMZ előtt.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-188">Find the resource named `pub-dmz-lb`, which is the load balancer in front of the public DMZ.</span></span>

3. <span data-ttu-id="d8aa0-189">Másolja a nyilvános IP-addess származó a **áttekintése** panelen, és nyissa meg ezt a címet egy webböngészőben.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-189">Copy the public IP addess from the **Overview** blade and open this address in a web browser.</span></span> <span data-ttu-id="d8aa0-190">Megtekintheti az Apache2 kiszolgáló alapértelmezett kezdőlapját.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-190">You should see the default Apache2 server home page.</span></span>

4. <span data-ttu-id="d8aa0-191">Keresse meg az erőforrást nevű `int-dmz-lb`, azaz előtti terheléselosztó tartománynévcímkéje a privát DMZ-t.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-191">Find the resource named `int-dmz-lb`, which is the load balancer in front of the private DMZ.</span></span> <span data-ttu-id="d8aa0-192">Másolja a magánhálózati IP-címet a **áttekintése** panelen.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-192">Copy the private IP address from the **Overview** blade.</span></span>

5. <span data-ttu-id="d8aa0-193">Keresse meg a virtuális gép nevű `jb-vm1`.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-193">Find the VM named `jb-vm1`.</span></span> <span data-ttu-id="d8aa0-194">Kattintson a **Connect** és a távoli asztal használata a virtuális Géphez való csatlakozáshoz.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-194">Click **Connect** and use Remote Desktop to connect to the VM.</span></span> <span data-ttu-id="d8aa0-195">A felhasználónév és jelszó a onprem.json fájlban vannak megadva.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-195">The user name and password are specified in the onprem.json file.</span></span>

6. <span data-ttu-id="d8aa0-196">A távoli asztali munkamenetet a nyisson meg egy webböngészőt, és keresse meg az IP-címet a 4. lépéssel.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-196">From the Remote Desktop Session, open a web browser and navigate to the IP address from step 4.</span></span> <span data-ttu-id="d8aa0-197">Megtekintheti az Apache2 kiszolgáló alapértelmezett kezdőlapját.</span><span class="sxs-lookup"><span data-stu-id="d8aa0-197">You should see the default Apache2 server home page.</span></span>

[availability-set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/dmz/secure-vnet-dmz

[implementing-a-secure-hybrid-network-architecture]: ./secure-vnet-hybrid.md
[iptables]: https://help.ubuntu.com/community/IptablesHowTo
[lb-probe]: /azure/load-balancer/load-balancer-custom-probe-overview
[load-balancer]: /azure/load-balancer/load-balancer-Internet-overview
[network-security-group]: /azure/virtual-network/virtual-networks-nsg

[visio-download]: https://archcenter.blob.core.windows.net/cdn/dmz-reference-architectures.vsdx
