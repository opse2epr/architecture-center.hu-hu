---
title: Implementálhat DMZ-t az Azure és az Internet között
titleSuffix: Azure Reference Architectures
description: Interneteléréssel rendelkező, biztonságos hibrid hálózati architektúra megvalósítása az Azure-ban.
author: telmosampaio
ms.date: 10/22/2018
ms.custom: seodec18
pnp.series.title: Network DMZ
pnp.series.next: nva-ha
pnp.series.prev: secure-vnet-hybrid
cardTitle: DMZ between Azure and the Internet
ms.openlocfilehash: ec87cf9aa69bbfea9e40f740fe27e3183bc45fc7
ms.sourcegitcommit: 88a68c7e9b6b772172b7faa4b9fd9c061a9f7e9d
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/08/2018
ms.locfileid: "53119982"
---
# <a name="implement-a-dmz-between-azure-and-the-internet"></a><span data-ttu-id="9c0d2-103">Implementálhat DMZ-t az Azure és az Internet között</span><span class="sxs-lookup"><span data-stu-id="9c0d2-103">Implement a DMZ between Azure and the Internet</span></span>

<span data-ttu-id="9c0d2-104">Ez a referenciaarchitektúra egy biztonságos hibrid hálózatot mutat be, amely kiterjeszti a helyszíni hálózatot az Azure-ba, és internetes forgalmat is fogad.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-104">This reference architecture shows a secure hybrid network that extends an on-premises network to Azure and also accepts Internet traffic.</span></span> <span data-ttu-id="9c0d2-105">[**A megoldás üzembe helyezése.**](#deploy-the-solution)</span><span class="sxs-lookup"><span data-stu-id="9c0d2-105">[**Deploy this solution**](#deploy-the-solution).</span></span>

![Biztonságos hibrid hálózati architektúra](./images/dmz-public.png)

<span data-ttu-id="9c0d2-107">*Töltse le az architektúra [Visio-fájlját][visio-download].*</span><span class="sxs-lookup"><span data-stu-id="9c0d2-107">*Download a [Visio file][visio-download] of this architecture.*</span></span>

<span data-ttu-id="9c0d2-108">Ez a referenciaarchitektúra a [DMZ az Azure és a helyszíni adatközpont közötti implementálásával kapcsolatos][implementing-a-secure-hybrid-network-architecture] cikkben ismertetett architektúrát bővíti ki.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-108">This reference architecture extends the architecture described in [Implementing a DMZ between Azure and your on-premises datacenter][implementing-a-secure-hybrid-network-architecture].</span></span> <span data-ttu-id="9c0d2-109">Hozzáadja a nyilvános DMZ-t, amely kezeli az internetes forgalmat a helyszíni hálózat érkező forgalmat kezelő privát DMZ mellett.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-109">It adds a public DMZ that handles Internet traffic, in addition to the private DMZ that handles traffic from the on-premises network.</span></span>

<span data-ttu-id="9c0d2-110">Az architektúra gyakori használati módjai többek között a következők:</span><span class="sxs-lookup"><span data-stu-id="9c0d2-110">Typical uses for this architecture include:</span></span>

- <span data-ttu-id="9c0d2-111">Hibrid alkalmazások, amelyekben a számítási feladatok részben a helyszínen, részben pedig az Azure-ban futnak.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-111">Hybrid applications where workloads run partly on-premises and partly in Azure.</span></span>
- <span data-ttu-id="9c0d2-112">Azure-infrastruktúra, amely átirányítja a helyszínről és az internetről bejövő forgalmat.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-112">Azure infrastructure that routes incoming traffic from on-premises and the Internet.</span></span>

## <a name="architecture"></a><span data-ttu-id="9c0d2-113">Architektúra</span><span class="sxs-lookup"><span data-stu-id="9c0d2-113">Architecture</span></span>

<span data-ttu-id="9c0d2-114">Az architektúra a következőkben leírt összetevőkből áll.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-114">The architecture consists of the following components.</span></span>

- <span data-ttu-id="9c0d2-115">**Nyilvános IP-cím (PIP)**.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-115">**Public IP address (PIP)**.</span></span> <span data-ttu-id="9c0d2-116">A nyilvános végpont IP-címe.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-116">The IP address of the public endpoint.</span></span> <span data-ttu-id="9c0d2-117">Az internethez csatlakozó külső felhasználók ezen a címen keresztül férhetnek hozzá a rendszerhez.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-117">External users connected to the Internet can access the system through this address.</span></span>
- <span data-ttu-id="9c0d2-118">**Hálózati virtuális berendezés (NVA)**.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-118">**Network virtual appliance (NVA)**.</span></span> <span data-ttu-id="9c0d2-119">Ez az architektúra tartalmaz egy különálló NVA-készletet az internetről érkező forgalom számára.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-119">This architecture includes a separate pool of NVAs for traffic originating on the Internet.</span></span>
- <span data-ttu-id="9c0d2-120">**Azure Load Balancer**.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-120">**Azure load balancer**.</span></span> <span data-ttu-id="9c0d2-121">Az internetről érkező összes kérés a terheléselosztón halad keresztül, amely elosztja azokat a nyilvános DMZ-n található NVA-k között.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-121">All incoming requests from the Internet pass through the load balancer and are distributed to the NVAs in the public DMZ.</span></span>
- <span data-ttu-id="9c0d2-122">**Nyilvános DMZ bejövő alhálózata**.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-122">**Public DMZ inbound subnet**.</span></span> <span data-ttu-id="9c0d2-123">Ez az alhálózat fogadja az Azure Load Balancer kéréseit.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-123">This subnet accepts requests from the Azure load balancer.</span></span> <span data-ttu-id="9c0d2-124">Ezután átadja a bejövő kéréseket a nyilvános DMZ egyik NVA-jának.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-124">Incoming requests are passed to one of the NVAs in the public DMZ.</span></span>
- <span data-ttu-id="9c0d2-125">**Nyilvános DMZ kimenő alhálózata**.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-125">**Public DMZ outbound subnet**.</span></span> <span data-ttu-id="9c0d2-126">Az NVA által jóváhagyott kérések ezen az alhálózaton haladnak át a webes réteg belső terheléselosztója felé.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-126">Requests that are approved by the NVA pass through this subnet to the internal load balancer for the web tier.</span></span>

## <a name="recommendations"></a><span data-ttu-id="9c0d2-127">Javaslatok</span><span class="sxs-lookup"><span data-stu-id="9c0d2-127">Recommendations</span></span>

<span data-ttu-id="9c0d2-128">Az alábbi javaslatok a legtöbb forgatókönyvre vonatkoznak.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-128">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="9c0d2-129">Kövesse ezeket a javaslatokat, ha nincsenek ezeket felülíró követelményei.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-129">Follow these recommendations unless you have a specific requirement that overrides them.</span></span>

### <a name="nva-recommendations"></a><span data-ttu-id="9c0d2-130">Hálózati virtuális készülékre vonatkozó javaslatok</span><span class="sxs-lookup"><span data-stu-id="9c0d2-130">NVA recommendations</span></span>

<span data-ttu-id="9c0d2-131">Használjon egy NVA-készletet az internetről érkező forgalomhoz, és egy másikat a helyszínről érkezőhöz.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-131">Use one set of NVAs for traffic originating on the Internet, and another for traffic originating on-premises.</span></span> <span data-ttu-id="9c0d2-132">Ha egyetlen NVA-készlet használ mindkettőhöz, az biztonsági kockázatot jelent, mivel nincs biztonsági határ a két fajta hálózati forgalom között.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-132">Using only one set of NVAs for both is a security risk, because it provides no security perimeter between the two sets of network traffic.</span></span> <span data-ttu-id="9c0d2-133">A különálló NVA-k használata csökkenti az ellenőrző biztonsági szabályok összetettségét, és egyértelművé teszi, hogy melyik szabály vonatkozik az egyes bejövő hálózati kérésekre.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-133">Using separate NVAs reduces the complexity of checking security rules, and makes it clear which rules correspond to each incoming network request.</span></span> <span data-ttu-id="9c0d2-134">Az egyik NVA-készlet csak az internetes forgalomra, a másik pedig csak a helyszínről érkező forgalomra vonatkozó szabályokat implementál.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-134">One set of NVAs implements rules for Internet traffic only, while another set of NVAs implement rules for on-premises traffic only.</span></span>

<span data-ttu-id="9c0d2-135">Alkalmazzon egy 7. rétegbeli NVA-t, amely megszakítja az alkalmazások kapcsolatát az NVA szintjén, és fenntartja a kompatibilitást a háttérszintekkel.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-135">Include a layer-7 NVA to terminate application connections at the NVA level and maintain compatibility with the backend tiers.</span></span> <span data-ttu-id="9c0d2-136">Ez garantálja a szimmetrikus kapcsolatokat, ahol a háttérbeli rétegekből érkező válaszforgalom az NVA-n halad keresztül.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-136">This guarantees symmetric connectivity where response traffic from the backend tiers returns through the NVA.</span></span>

### <a name="public-load-balancer-recommendations"></a><span data-ttu-id="9c0d2-137">Nyilvános terheléselosztóra vonatkozó javaslatok</span><span class="sxs-lookup"><span data-stu-id="9c0d2-137">Public load balancer recommendations</span></span>

<span data-ttu-id="9c0d2-138">A skálázhatóság és rendelkezésre állás érdekében a nyilvános DMZ NVA-it egy [rendelkezésre állási csoportban][availability-set] helyezze üzembe, és egy [internetkapcsolattal rendelkező terheléselosztó][load-balancer] használatával ossza el az internetes kéréseket a rendelkezésre állási csoport NVA-i között.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-138">For scalability and availability, deploy the public DMZ NVAs in an [availability set][availability-set] and use an [Internet facing load balancer][load-balancer] to distribute Internet requests across the NVAs in the availability set.</span></span>

<span data-ttu-id="9c0d2-139">Konfigurálja úgy a terheléselosztót, hogy az csak az internetes forgalomhoz szükséges portokból fogadjon el kéréseket.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-139">Configure the load balancer to accept requests only on the ports necessary for Internet traffic.</span></span> <span data-ttu-id="9c0d2-140">Korlátozza például a 80-as portra a beérkező HTTP-, valamint a 443-as portra a beérkező HTTPS-kéréseket.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-140">For example, restrict inbound HTTP requests to port 80 and inbound HTTPS requests to port 443.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="9c0d2-141">Méretezési szempontok</span><span class="sxs-lookup"><span data-stu-id="9c0d2-141">Scalability considerations</span></span>

<span data-ttu-id="9c0d2-142">Javasoljuk, hogy helyezzen el egy terheléselosztót a nyilvános DMZ előtt a kezdetekkor abban az esetben is, ha az architektúrában kezdetben egyetlen NVA szükséges a nyilvános DMZ-ben.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-142">Even if your architecture initially requires a single NVA in the public DMZ, we recommend putting a load balancer in front of the public DMZ from the beginning.</span></span> <span data-ttu-id="9c0d2-143">Ez megkönnyíti a több NVA-ra való skálázást a jövőben, ha szükséges.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-143">That will make it easier to scale to multiple NVAs in the future, if needed.</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="9c0d2-144">Rendelkezésre állási szempontok</span><span class="sxs-lookup"><span data-stu-id="9c0d2-144">Availability considerations</span></span>

<span data-ttu-id="9c0d2-145">Az internetkapcsolattal rendelkező terheléselosztó megköveteli, hogy a nyilvános DMZ bejövő alhálózatán található minden NVA implementáljon egy [állapotmintát][lb-probe].</span><span class="sxs-lookup"><span data-stu-id="9c0d2-145">The Internet facing load balancer requires each NVA in the public DMZ inbound subnet to implement a [health probe][lb-probe].</span></span> <span data-ttu-id="9c0d2-146">A terheléselosztó nem elérhetőnek tekinti azokat az állapotmintákat, amelyek nem válaszolnak ezen a végponton, és a kéréseket a rendelkezésre állási csoport más NVA-i számára továbbítja.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-146">A health probe that fails to respond on this endpoint is considered to be unavailable, and the load balancer will direct requests to other NVAs in the same availability set.</span></span> <span data-ttu-id="9c0d2-147">Vegye figyelembe, hogy ha egyik NVA sem válaszol, az alkalmazás összeomlik, ezért fontos a monitorozás konfigurálása, amely értesíti a fejlesztési és üzemeltetési csapat tagjait, ha a kifogástalan NVA-k száma egy meghatározott küszöbérték alá esik.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-147">Note that if all NVAs fail to respond, your application will fail, so it's important to have monitoring configured to alert DevOps when the number of healthy NVA instances falls below a defined threshold.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="9c0d2-148">Felügyeleti szempontok</span><span class="sxs-lookup"><span data-stu-id="9c0d2-148">Manageability considerations</span></span>

<span data-ttu-id="9c0d2-149">Monitorozása és felügyelete az nva-k a nyilvános DMZ-ben a a felügyeleti alhálózaton található jumpbox kell elvégeznie.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-149">All monitoring and management for the NVAs in the public DMZ should be performed by the jumpbox in the management subnet.</span></span> <span data-ttu-id="9c0d2-150">A [DMZ az Azure és a helyszíni adatközpont közötti implementálásával kapcsolatos][implementing-a-secure-hybrid-network-architecture] cikkben ismertetett módon definiáljon egyetlen hálózati útvonalat a helyszíni hálózattól az átjárón keresztül a jumpboxig a hozzáférés korlátozása érdekében.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-150">As discussed in [Implementing a DMZ between Azure and your on-premises datacenter][implementing-a-secure-hybrid-network-architecture], define a single network route from the on-premises network through the gateway to the jumpbox, in order to restrict access.</span></span>

<span data-ttu-id="9c0d2-151">Ha a helyszíni hálózat és az Azure közötti átjárókapcsolat nem érhető el, a jumpboxot elérheti egy nyilvános IP-cím üzembe helyezésével és a jumpboxhoz való hozzáadásával, majd az internetről való bejelentkezéssel.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-151">If gateway connectivity from your on-premises network to Azure is down, you can still reach the jumpbox by deploying a public IP address, adding it to the jumpbox, and logging in from the Internet.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="9c0d2-152">Biztonsági szempontok</span><span class="sxs-lookup"><span data-stu-id="9c0d2-152">Security considerations</span></span>

<span data-ttu-id="9c0d2-153">Ez a referenciaarchitektúra többszintű biztonságot valósít meg:</span><span class="sxs-lookup"><span data-stu-id="9c0d2-153">This reference architecture implements multiple levels of security:</span></span>

- <span data-ttu-id="9c0d2-154">Az internetkapcsolattal rendelkező terheléselosztó átirányítja a kéréseket a nyilvános DMZ bejövő alhálózatán található NVA-khoz, kizárólag az alkalmazás számára szükséges portokon.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-154">The Internet facing load balancer directs requests to the NVAs in the inbound public DMZ subnet, and only on the ports necessary for the application.</span></span>
- <span data-ttu-id="9c0d2-155">A nyilvános DMZ bejövő és kimenő alhálózatára vonatkozó NSG-szabályok megakadályozzák az NVA-k feltörését az NSG-szabályokon kívül eső kérések blokkolásával.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-155">The NSG rules for the inbound and outbound public DMZ subnets prevent the NVAs from being compromised, by blocking requests that fall outside of the NSG rules.</span></span>
- <span data-ttu-id="9c0d2-156">Az NVA-k NAT-útválasztási konfigurációja átirányítja a 80-as és 443-as portokon bejövő kéréseket a webes rétegben lévő terheléselosztóhoz, de figyelmen hagyja az összes többi porton érkező kéréseket.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-156">The NAT routing configuration for the NVAs directs incoming requests on port 80 and port 443 to the web tier load balancer, but ignores requests on all other ports.</span></span>

<span data-ttu-id="9c0d2-157">Érdemes az összes porton érkező bejövő kérésekről naplót készíteni.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-157">You should log all incoming requests on all ports.</span></span> <span data-ttu-id="9c0d2-158">Rendszeresen ellenőrizze a naplókat, és fordítson különös figyelmet a várt paramétereken kívül eső kérésekre, mert ezek behatolási kísérleteket jelezhetnek.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-158">Regularly audit the logs, paying attention to requests that fall outside of expected parameters, as these may indicate intrusion attempts.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="9c0d2-159">A megoldás üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="9c0d2-159">Deploy the solution</span></span>

<span data-ttu-id="9c0d2-160">Az ezeknek a javaslatoknak a figyelembe vételével megvalósított referenciaarchitektúra egy üzemelő példánya elérhető a [GitHubon][github-folder].</span><span class="sxs-lookup"><span data-stu-id="9c0d2-160">A deployment for a reference architecture that implements these recommendations is available on [GitHub][github-folder].</span></span>

### <a name="prerequisites"></a><span data-ttu-id="9c0d2-161">Előfeltételek</span><span class="sxs-lookup"><span data-stu-id="9c0d2-161">Prerequisites</span></span>

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-resources"></a><span data-ttu-id="9c0d2-162">Erőforrások üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="9c0d2-162">Deploy resources</span></span>

1. <span data-ttu-id="9c0d2-163">Keresse meg a `/dmz/secure-vnet-dmz` referencia architektúrák GitHub-adattár mappát.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-163">Navigate to the `/dmz/secure-vnet-dmz` folder of the reference architectures GitHub repository.</span></span>

2. <span data-ttu-id="9c0d2-164">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="9c0d2-164">Run the following command:</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <region> -p onprem.json --deploy
    ```

3. <span data-ttu-id="9c0d2-165">Futtassa az alábbi parancsot:</span><span class="sxs-lookup"><span data-stu-id="9c0d2-165">Run the following command:</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <region> -p secure-vnet-hybrid.json --deploy
    ```

### <a name="connect-the-on-premises-and-azure-gateways"></a><span data-ttu-id="9c0d2-166">A helyszíni és az Azure-átjárók csatlakoztatása</span><span class="sxs-lookup"><span data-stu-id="9c0d2-166">Connect the on-premises and Azure gateways</span></span>

<span data-ttu-id="9c0d2-167">Ebben a lépésben két helyi hálózati átjáró csatlakozik.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-167">In this step, you will connect the two local network gateways.</span></span>

1. <span data-ttu-id="9c0d2-168">Az Azure Portalon keresse meg a létrehozott erőforráscsoportot.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-168">In the Azure Portal, navigate to the resource group that you created.</span></span>

2. <span data-ttu-id="9c0d2-169">Keresse meg az erőforrást nevű `ra-vpn-vgw-pip` , és másolja az IP-cím látható a **áttekintése** panelen.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-169">Find the resource named `ra-vpn-vgw-pip` and copy the IP address shown in the **Overview** blade.</span></span>

3. <span data-ttu-id="9c0d2-170">Keresse meg az erőforrást nevű `onprem-vpn-lgw`.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-170">Find the resource named `onprem-vpn-lgw`.</span></span>

4. <span data-ttu-id="9c0d2-171">Kattintson a **konfigurációs** panelen.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-171">Click the **Configuration** blade.</span></span> <span data-ttu-id="9c0d2-172">A **IP-cím**, illessze be a 2. lépésben felírt IP-címet.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-172">Under **IP address**, paste in the IP address from step 2.</span></span>

    ![Az IP-cím mezőt képernyőképe](./images/local-net-gw.png)

5. <span data-ttu-id="9c0d2-174">Kattintson a **mentése** és várjon, amíg a művelet befejeződik.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-174">Click **Save** and wait for the operation to complete.</span></span> <span data-ttu-id="9c0d2-175">Körülbelül 5 percet is igénybe vehet.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-175">It can take about 5 minutes.</span></span>

6. <span data-ttu-id="9c0d2-176">Keresse meg az erőforrást nevű `onprem-vpn-gateway1-pip`.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-176">Find the resource named `onprem-vpn-gateway1-pip`.</span></span> <span data-ttu-id="9c0d2-177">Másolja ki az IP-cím látható a **áttekintése** panelen.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-177">Copy the IP address shown in the **Overview** blade.</span></span>

7. <span data-ttu-id="9c0d2-178">Keresse meg az erőforrást nevű `ra-vpn-lgw`.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-178">Find the resource named `ra-vpn-lgw`.</span></span>

8. <span data-ttu-id="9c0d2-179">Kattintson a **konfigurációs** panelen.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-179">Click the **Configuration** blade.</span></span> <span data-ttu-id="9c0d2-180">A **IP-cím**, illessze be a 6. lépésben felírt IP-címet.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-180">Under **IP address**, paste in the IP address from step 6.</span></span>

9. <span data-ttu-id="9c0d2-181">Kattintson a **mentése** és várjon, amíg a művelet befejeződik.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-181">Click **Save** and wait for the operation to complete.</span></span>

10. <span data-ttu-id="9c0d2-182">A kapcsolat ellenőrzéséhez nyissa meg a **kapcsolatok** minden átjáró paneljét.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-182">To verify the connection, go to the **Connections** blade for each gateway.</span></span> <span data-ttu-id="9c0d2-183">A állapotúnak kell lennie **csatlakoztatva**.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-183">The status should be **Connected**.</span></span>

### <a name="verify-that-network-traffic-reaches-the-web-tier"></a><span data-ttu-id="9c0d2-184">Győződjön meg arról, hogy a hálózati forgalom eléri a webes szint</span><span class="sxs-lookup"><span data-stu-id="9c0d2-184">Verify that network traffic reaches the web tier</span></span>

1. <span data-ttu-id="9c0d2-185">Az Azure Portalon keresse meg a létrehozott erőforráscsoportot.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-185">In the Azure Portal, navigate to the resource group that you created.</span></span>

2. <span data-ttu-id="9c0d2-186">Keresse meg az erőforrást nevű `pub-dmz-lb`, amely a terheléselosztó a nyilvános DMZ előtt.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-186">Find the resource named `pub-dmz-lb`, which is the load balancer in front of the public DMZ.</span></span>

3. <span data-ttu-id="9c0d2-187">Másolja a nyilvános IP-addess származó a **áttekintése** panelen, és nyissa meg ezt a címet egy webböngészőben.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-187">Copy the public IP addess from the **Overview** blade and open this address in a web browser.</span></span> <span data-ttu-id="9c0d2-188">Megtekintheti az Apache2 kiszolgáló alapértelmezett kezdőlapját.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-188">You should see the default Apache2 server home page.</span></span>

4. <span data-ttu-id="9c0d2-189">Keresse meg az erőforrást nevű `int-dmz-lb`, azaz előtti terheléselosztó tartománynévcímkéje a privát DMZ-t.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-189">Find the resource named `int-dmz-lb`, which is the load balancer in front of the private DMZ.</span></span> <span data-ttu-id="9c0d2-190">Másolja a magánhálózati IP-címet a **áttekintése** panelen.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-190">Copy the private IP address from the **Overview** blade.</span></span>

5. <span data-ttu-id="9c0d2-191">Keresse meg a virtuális gép nevű `jb-vm1`.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-191">Find the VM named `jb-vm1`.</span></span> <span data-ttu-id="9c0d2-192">Kattintson a **Connect** és a távoli asztal használata a virtuális Géphez való csatlakozáshoz.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-192">Click **Connect** and use Remote Desktop to connect to the VM.</span></span> <span data-ttu-id="9c0d2-193">A felhasználónév és jelszó a onprem.json fájlban vannak megadva.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-193">The user name and password are specified in the onprem.json file.</span></span>

6. <span data-ttu-id="9c0d2-194">A távoli asztali munkamenetet a nyisson meg egy webböngészőt, és keresse meg az IP-címet a 4. lépéssel.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-194">From the Remote Desktop Session, open a web browser and navigate to the IP address from step 4.</span></span> <span data-ttu-id="9c0d2-195">Megtekintheti az Apache2 kiszolgáló alapértelmezett kezdőlapját.</span><span class="sxs-lookup"><span data-stu-id="9c0d2-195">You should see the default Apache2 server home page.</span></span>

[availability-set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/dmz/secure-vnet-dmz

[implementing-a-secure-hybrid-network-architecture]: ./secure-vnet-hybrid.md
[iptables]: https://help.ubuntu.com/community/IptablesHowTo
[lb-probe]: /azure/load-balancer/load-balancer-custom-probe-overview
[load-balancer]: /azure/load-balancer/load-balancer-Internet-overview
[network-security-group]: /azure/virtual-network/virtual-networks-nsg

[visio-download]: https://archcenter.blob.core.windows.net/cdn/dmz-reference-architectures.vsdx
