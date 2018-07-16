---
title: SAP fejlesztési/tesztelési feladatokhoz
description: Fejlesztési-tesztelési környezet SAP-forgatókönyv
author: AndrewDibbins
ms.date: 7/11/18
ms.openlocfilehash: 675a5cb4b1ee4001ca50d24c145ce1a177f90da4
ms.sourcegitcommit: 71cbef121c40ef36e2d6e3a088cb85c4260599b9
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 07/14/2018
ms.locfileid: "39061124"
---
# <a name="sap-for-devtest-workloads"></a><span data-ttu-id="9dd41-103">SAP fejlesztési/tesztelési feladatokhoz</span><span class="sxs-lookup"><span data-stu-id="9dd41-103">SAP for dev/test workloads</span></span>

<span data-ttu-id="9dd41-104">Ebben a példában egy fejlesztési-tesztelési megvalósítását SAP NetWeaver futtatása Windows vagy Linux-alapú környezetben az Azure-ban útmutatást nyújt.</span><span class="sxs-lookup"><span data-stu-id="9dd41-104">This example provides guidance for how to run a dev/test implementation of SAP NetWeaver in a Windows or Linux environment on Azure.</span></span> <span data-ttu-id="9dd41-105">A használt adatbázisa AnyDB, az SAP kifejezés bármely támogatott adatbázis-kezelő (amely nem az SAP HANA).</span><span class="sxs-lookup"><span data-stu-id="9dd41-105">The database used is AnyDB, the SAP term for any supported DBMS (that isn't SAP HANA).</span></span> <span data-ttu-id="9dd41-106">Ez az architektúra nem éles környezetekhez lett kialakítva, mert csak egyetlen virtuális géphez (VM) üzemel, és annak méretét is módosítható a szervezet igényeinek megfelelően.</span><span class="sxs-lookup"><span data-stu-id="9dd41-106">Because this architecture is designed for non-production environments, it's deployed with just a single virtual machine (VM) and it's size can be changed to accommodate your organization's needs.</span></span>

<span data-ttu-id="9dd41-107">Éles környezetben használja esetben tekintse át az SAP referenciaarchitektúrák alatt érhető el:</span><span class="sxs-lookup"><span data-stu-id="9dd41-107">For production use cases review the SAP reference architectures available below:</span></span>

* <span data-ttu-id="9dd41-108">[SAP netweaver a AnyDB][sap-netweaver]</span><span class="sxs-lookup"><span data-stu-id="9dd41-108">[SAP netweaver for AnyDB][sap-netweaver]</span></span>
* <span data-ttu-id="9dd41-109">[SAP S/4hana-t][sap-hana]</span><span class="sxs-lookup"><span data-stu-id="9dd41-109">[SAP S/4Hana][sap-hana]</span></span>
* <span data-ttu-id="9dd41-110">[Nagyméretű Azure-példányokon futó SAP][sap-large]</span><span class="sxs-lookup"><span data-stu-id="9dd41-110">[SAP on Azure large instances][sap-large]</span></span>

## <a name="related-use-cases"></a><span data-ttu-id="9dd41-111">Kapcsolódó alkalmazási helyzetek</span><span class="sxs-lookup"><span data-stu-id="9dd41-111">Related use cases</span></span>

<span data-ttu-id="9dd41-112">Ebben a forgatókönyvben a következő használati esetek, vegye figyelembe:</span><span class="sxs-lookup"><span data-stu-id="9dd41-112">Consider this scenario for the following use cases:</span></span>

* <span data-ttu-id="9dd41-113">A nem kritikus SAP nem termelési számítási feladatokhoz (védőfal, fejlesztési, tesztelési, minőségbiztosítási)</span><span class="sxs-lookup"><span data-stu-id="9dd41-113">Non-critical SAP non-productive workloads (sandbox, development, test, quality assurance)</span></span>
* <span data-ttu-id="9dd41-114">A nem kritikus fontosságú SAP business one számítási feladatok</span><span class="sxs-lookup"><span data-stu-id="9dd41-114">Non-critical SAP business one workloads</span></span>

## <a name="architecture"></a><span data-ttu-id="9dd41-115">Architektúra</span><span class="sxs-lookup"><span data-stu-id="9dd41-115">Architecture</span></span>

![Ábra](media/sap-2tier/SAP-Infra-2Tier_finalversion.png)

<span data-ttu-id="9dd41-117">Ebben a forgatókönyvben egy SAP-rendszer önálló adatbázis és a SAP alkalmazás kiszolgáló egyetlen virtuális gépen, az adatfolyam-gyűjteményre, a forgatókönyv segítségével a következőképpen mutatja be:</span><span class="sxs-lookup"><span data-stu-id="9dd41-117">This scenario covers the provision of a single SAP system database and SAP application Server on a single virtual machine, the data flows through the scenario as follows:</span></span>

1. <span data-ttu-id="9dd41-118">-Ügyfelek a bemutatási szint használatához az SAP grafikus felhasználói felülettel, vagy más felhasználói felületek (az Internet Explorer, az Excel vagy más webes alkalmazás) a helyszíni eléréséhez az Azure-alapú SAP-rendszerhez.</span><span class="sxs-lookup"><span data-stu-id="9dd41-118">Customers from the Presentation Tier use their SAP gui, or other user interfaces (Internet Explorer, Excel or other web application) on premise to access the Azure based SAP system.</span></span>
2. <span data-ttu-id="9dd41-119">Kapcsolat a létrehozott Express Route használatával biztosítja.</span><span class="sxs-lookup"><span data-stu-id="9dd41-119">Connectivity is provided through the use of the established Express Route.</span></span> <span data-ttu-id="9dd41-120">Az Expressroute az Azure-ban, az Expressroute-átjárója megszakad.</span><span class="sxs-lookup"><span data-stu-id="9dd41-120">The Express Route is terminated in Azure at the Express Route Gateway.</span></span> <span data-ttu-id="9dd41-121">Hálózati forgalmi útvonalak az Express Route-átjárón keresztül az átjáró-alhálózat és az átjáró-alhálózat az alkalmazás szinten küllő alhálózathoz (lásd a [küllős] [ hub-spoke] minta) és a egy hálózati biztonsági keresztül Átjáró, a SAP alkalmazás virtuális géphez.</span><span class="sxs-lookup"><span data-stu-id="9dd41-121">Network traffic routes through the Express Route gateway to the Gateway Subnet and from the gateway subnet to the Application Tier Spoke subnet (see the [hub-spoke][hub-spoke] pattern) and via a Network Security Gateway to the SAP application virtual machine.</span></span>
3. <span data-ttu-id="9dd41-122">Az identitás felügyeleti kiszolgálók hitelesítési szolgáltatásokat nyújtanak.</span><span class="sxs-lookup"><span data-stu-id="9dd41-122">The identity management servers provide authentication services.</span></span>
4. <span data-ttu-id="9dd41-123">A Jump-Box helyi felügyeleti képességeket biztosít.</span><span class="sxs-lookup"><span data-stu-id="9dd41-123">The Jump Box provides local management capabilities.</span></span>

### <a name="components"></a><span data-ttu-id="9dd41-124">Összetevők</span><span class="sxs-lookup"><span data-stu-id="9dd41-124">Components</span></span>

* <span data-ttu-id="9dd41-125">[Erőforráscsoportok](/azure/azure-resource-manager/resource-group-overview#resource-groups) Azure-erőforrások logikai tárolója.</span><span class="sxs-lookup"><span data-stu-id="9dd41-125">[Resource Groups](/azure/azure-resource-manager/resource-group-overview#resource-groups) is a logical container for Azure resources.</span></span>
* <span data-ttu-id="9dd41-126">[Virtuális hálózatok](/azure/virtual-network/virtual-networks-overview) az alapja az Azure-on belüli hálózati kommunikáció</span><span class="sxs-lookup"><span data-stu-id="9dd41-126">[Virtual Networks](/azure/virtual-network/virtual-networks-overview) is the basis of network communications within Azure</span></span>
* <span data-ttu-id="9dd41-127">[Virtuális gép](/azure/virtual-machines/windows/overview) Azure Virtual Machines biztosít az igény szerinti, nagy léptékben méretezhető, biztonságos és virtualizált infrastruktúra Windows vagy Linux-kiszolgáló használatával</span><span class="sxs-lookup"><span data-stu-id="9dd41-127">[Virtual Machine](/azure/virtual-machines/windows/overview) Azure Virtual Machines provides on-demand, high-scale, secure, virtualized infrastructure using Windows or Linux Server</span></span>
* <span data-ttu-id="9dd41-128">[Express Route](/azure/expressroute/expressroute-introduction) lehetővé teszi, kiterjesztheti helyszíni hálózatait a Microsoft cloud, amelyet egy kapcsolatszolgáltató biztosít egy privát kapcsolaton keresztül.</span><span class="sxs-lookup"><span data-stu-id="9dd41-128">[Express Route](/azure/expressroute/expressroute-introduction) lets you extend your on-premises networks into the Microsoft cloud over a private connection facilitated by a connectivity provider.</span></span>
* <span data-ttu-id="9dd41-129">[Hálózati biztonsági csoport](/azure/virtual-network/security-overview) lehetővé teszi a hálózati virtuális hálózatban lévő erőforrásokra irányuló forgalmat.</span><span class="sxs-lookup"><span data-stu-id="9dd41-129">[Network Security Group](/azure/virtual-network/security-overview) lets you limit network traffic to resources in a virtual network.</span></span> <span data-ttu-id="9dd41-130">A hálózati biztonsági csoport egy biztonsági szabályokból álló listát tartalmaz, amelyek engedélyezik vagy megtagadják a bejövő vagy kimenő hálózati forgalmat a forrás vagy a cél IP-címe, illetve portok vagy protokollok alapján.</span><span class="sxs-lookup"><span data-stu-id="9dd41-130">A network security group contains a list of security rules that allow or deny inbound or outbound network traffic based on source or destination IP address, port, and protocol.</span></span> 

## <a name="considerations"></a><span data-ttu-id="9dd41-131">Megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="9dd41-131">Considerations</span></span>

### <a name="availability"></a><span data-ttu-id="9dd41-132">Rendelkezésre állás</span><span class="sxs-lookup"><span data-stu-id="9dd41-132">Availability</span></span>

 <span data-ttu-id="9dd41-133">A Microsoft kínál a szolgáltatásiszint-szerződés (SLA) egyetlen Virtuálisgép-példányokhoz.</span><span class="sxs-lookup"><span data-stu-id="9dd41-133">Microsoft offers a service level agreement (SLA) for single VM instances.</span></span> <span data-ttu-id="9dd41-134">További információ a Microsoft Azure szolgáltatásiszint-szerződés virtuális gépek [SLA-t a virtuális gépek](https://azure.microsoft.com/support/legal/sla/virtual-machines)</span><span class="sxs-lookup"><span data-stu-id="9dd41-134">For more information on Microsoft Azure Service Level Agreement for Virtual Machines [SLA For Virtual Machines](https://azure.microsoft.com/support/legal/sla/virtual-machines)</span></span>

### <a name="scalability"></a><span data-ttu-id="9dd41-135">Méretezhetőség</span><span class="sxs-lookup"><span data-stu-id="9dd41-135">Scalability</span></span>

<span data-ttu-id="9dd41-136">Általános méretezhető megoldások, tekintse át a [méretezési ellenőrzőlista] [ scalability] a az Azure Architecture Centert.</span><span class="sxs-lookup"><span data-stu-id="9dd41-136">For general guidance on designing scalable solutions, see the [scalability checklist][scalability] in the Azure Architecture Center.</span></span>

### <a name="security"></a><span data-ttu-id="9dd41-137">Biztonság</span><span class="sxs-lookup"><span data-stu-id="9dd41-137">Security</span></span>

<span data-ttu-id="9dd41-138">Általános megoldások biztonságos, tekintse át a [Azure Security dokumentációja][security].</span><span class="sxs-lookup"><span data-stu-id="9dd41-138">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="9dd41-139">Rugalmasság</span><span class="sxs-lookup"><span data-stu-id="9dd41-139">Resiliency</span></span>

<span data-ttu-id="9dd41-140">Rugalmas megoldások tervezésével kapcsolatos általános útmutatásért lásd: [rugalmas alkalmazások tervezése az Azure][resiliency].</span><span class="sxs-lookup"><span data-stu-id="9dd41-140">For general guidance on designing resilient solutions, see [Designing resilient applications for Azure][resiliency].</span></span>

## <a name="pricing"></a><span data-ttu-id="9dd41-141">Díjszabás</span><span class="sxs-lookup"><span data-stu-id="9dd41-141">Pricing</span></span>

<span data-ttu-id="9dd41-142">Ismerje meg a forgatókönyv futtatásával járó költségeket, a szolgáltatások mindegyike a költségkalkulátor az előre konfigurált.</span><span class="sxs-lookup"><span data-stu-id="9dd41-142">Explore the cost of running this scenario, all of the services are pre-configured in the cost calculator.</span></span>  <span data-ttu-id="9dd41-143">Tekintse meg, hogyan díjszabását szeretné módosítani az adott használatra esetben módosítsa a megfelelő változókat egyezik a várt forgalomhoz.</span><span class="sxs-lookup"><span data-stu-id="9dd41-143">To see how the pricing would change for your particular use case change the appropriate variables to match your expected traffic.</span></span>

<span data-ttu-id="9dd41-144">Adtunk meg beolvasni a várt forgalom mennyisége alapján négy példa költség profilok:</span><span class="sxs-lookup"><span data-stu-id="9dd41-144">We have provided four sample cost profiles based on amount of traffic you expect to get:</span></span>

|<span data-ttu-id="9dd41-145">Méret</span><span class="sxs-lookup"><span data-stu-id="9dd41-145">Size</span></span>|<span data-ttu-id="9dd41-146">A SAP</span><span class="sxs-lookup"><span data-stu-id="9dd41-146">SAPs</span></span>|<span data-ttu-id="9dd41-147">Virtuális gép típusa</span><span class="sxs-lookup"><span data-stu-id="9dd41-147">VM Type</span></span>|<span data-ttu-id="9dd41-148">Storage</span><span class="sxs-lookup"><span data-stu-id="9dd41-148">Storage</span></span>|<span data-ttu-id="9dd41-149">Azure díjkalkulátor</span><span class="sxs-lookup"><span data-stu-id="9dd41-149">Azure Pricing Calculator</span></span>|
|----|----|-------|-------|---------------|
|<span data-ttu-id="9dd41-150">Kicsi</span><span class="sxs-lookup"><span data-stu-id="9dd41-150">Small</span></span>|<span data-ttu-id="9dd41-151">8000</span><span class="sxs-lookup"><span data-stu-id="9dd41-151">8000</span></span>|<span data-ttu-id="9dd41-152">D8s_v3</span><span class="sxs-lookup"><span data-stu-id="9dd41-152">D8s_v3</span></span>|<span data-ttu-id="9dd41-153">2xP20, 1xP10</span><span class="sxs-lookup"><span data-stu-id="9dd41-153">2xP20, 1xP10</span></span>|[<span data-ttu-id="9dd41-154">Kis</span><span class="sxs-lookup"><span data-stu-id="9dd41-154">Small</span></span>](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1)|
|<span data-ttu-id="9dd41-155">Közepes</span><span class="sxs-lookup"><span data-stu-id="9dd41-155">Medium</span></span>|<span data-ttu-id="9dd41-156">16000</span><span class="sxs-lookup"><span data-stu-id="9dd41-156">16000</span></span>|<span data-ttu-id="9dd41-157">D16s_v3</span><span class="sxs-lookup"><span data-stu-id="9dd41-157">D16s_v3</span></span>|<span data-ttu-id="9dd41-158">3xP20, 1xP10</span><span class="sxs-lookup"><span data-stu-id="9dd41-158">3xP20, 1xP10</span></span>|[<span data-ttu-id="9dd41-159">Közepes</span><span class="sxs-lookup"><span data-stu-id="9dd41-159">Medium</span></span>](https://azure.com/e/465bd07047d148baab032b2f461550cd)|
<span data-ttu-id="9dd41-160">Nagy</span><span class="sxs-lookup"><span data-stu-id="9dd41-160">Large</span></span>|<span data-ttu-id="9dd41-161">32000</span><span class="sxs-lookup"><span data-stu-id="9dd41-161">32000</span></span>|<span data-ttu-id="9dd41-162">E32s_v3</span><span class="sxs-lookup"><span data-stu-id="9dd41-162">E32s_v3</span></span>|<span data-ttu-id="9dd41-163">3xP20, 1xP10</span><span class="sxs-lookup"><span data-stu-id="9dd41-163">3xP20, 1xP10</span></span>|[<span data-ttu-id="9dd41-164">Nagy méretű</span><span class="sxs-lookup"><span data-stu-id="9dd41-164">Large</span></span>](https://azure.com/e/ada2e849d68b41c3839cc976000c6931)|
<span data-ttu-id="9dd41-165">Extra nagy</span><span class="sxs-lookup"><span data-stu-id="9dd41-165">Extra Large</span></span>|<span data-ttu-id="9dd41-166">64000</span><span class="sxs-lookup"><span data-stu-id="9dd41-166">64000</span></span>|<span data-ttu-id="9dd41-167">M64s</span><span class="sxs-lookup"><span data-stu-id="9dd41-167">M64s</span></span>|<span data-ttu-id="9dd41-168">4xP20, 1xP10</span><span class="sxs-lookup"><span data-stu-id="9dd41-168">4xP20, 1xP10</span></span>|[<span data-ttu-id="9dd41-169">Extra nagy</span><span class="sxs-lookup"><span data-stu-id="9dd41-169">Extra Large</span></span>](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef)|

<span data-ttu-id="9dd41-170">Megjegyzés: egy útmutató, díjszabás, és csak azt jelzi, hogy a virtuális gépek és tárolási költségeit (kivéve, hálózati, biztonsági másolatot készíteni tárolás és adatkezelés bejövő/kimenő forgalom költségeit).</span><span class="sxs-lookup"><span data-stu-id="9dd41-170">Note: pricing is a guide and only indicates the VMs and storage costs (excludes, networking, backup storage and data ingress/egress charges).</span></span>

* <span data-ttu-id="9dd41-171">[Kis](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1): egy kis rendszer áll VM-típus D8s_v3 8 x vCPUs, 32 GB RAM-MAL és 200 GB ideiglenes tárhely, emellett két 512 GB és a egy 128 GB-os prémium szintű tárolólemezeket.</span><span class="sxs-lookup"><span data-stu-id="9dd41-171">[Small](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1): A small system consists of VM type D8s_v3 with 8x vCPUs, 32GB RAM and 200GB temp storage, additionally two 512GB and one 128GB premium storage disks.</span></span>
* <span data-ttu-id="9dd41-172">[Közepes](https://azure.com/e/465bd07047d148baab032b2f461550cd): egy közepes méretű rendszer áll VM-típus D16s_v3 16 x vCPUs, 64 GB RAM-MAL és 400 GB ideiglenes tárhely, emellett három 512 GB és a egy 128 GB-os prémium szintű tárolólemezeket.</span><span class="sxs-lookup"><span data-stu-id="9dd41-172">[Medium](https://azure.com/e/465bd07047d148baab032b2f461550cd): A medium system consists of VM type D16s_v3 with 16x vCPUs, 64GB RAM and 400GB temp storage, additionally three 512GB and one 128GB premium storage disks.</span></span>
* <span data-ttu-id="9dd41-173">[Nagy](https://azure.com/e/ada2e849d68b41c3839cc976000c6931): egy nagy méretű rendszer áll VM-típus E32s_v3 32 x vCPUs, 256 GB RAM-MAL és 512 GB ideiglenes tárhely, emellett három 512 GB és a egy 128 GB-os prémium szintű tárolólemezeket.</span><span class="sxs-lookup"><span data-stu-id="9dd41-173">[Large](https://azure.com/e/ada2e849d68b41c3839cc976000c6931): A large system consists of VM type E32s_v3 with 32x vCPUs, 256GB RAM and 512GB temp storage, additionally three 512GB and one 128GB premium storage disks.</span></span>
* <span data-ttu-id="9dd41-174">[Extra nagy](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef): egy extra nagy méretű rendszer áll típusú, a virtuális gép M64s 64 x vCPUs, 1024 GB RAM-MAL és 2000 GB ideiglenes tárhely, továbbá négy 512 GB és a egy 128 GB-os prémium szintű tárolólemezeket.</span><span class="sxs-lookup"><span data-stu-id="9dd41-174">[Extra Large](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef): An extra large system consists of a VM type M64s with 64x vCPUs, 1024GB RAM and 2000GB temp storage, additionally four 512GB and one 128GB premium storage disks.</span></span>

## <a name="deployment"></a><span data-ttu-id="9dd41-175">Környezet</span><span class="sxs-lookup"><span data-stu-id="9dd41-175">Deployment</span></span>

<span data-ttu-id="9dd41-176">A fenti forgatókönyv hasonlít az alapul szolgáló infrastruktúra üzembe helyezéséhez használja az üzembe helyezés gomb</span><span class="sxs-lookup"><span data-stu-id="9dd41-176">To deploy the underlying infrastructure similar to the scenario above, please use the deploy button</span></span>

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fsap-2tier%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

<span data-ttu-id="9dd41-177">\* SAP nem lesz telepítve, végrehajthatja az után az infrastruktúra beépített manuálisan kell.</span><span class="sxs-lookup"><span data-stu-id="9dd41-177">\* SAP will not be installed, you'll need to do this after the infrastructure is built manually.</span></span>

<!-- links -->
[reference architecture]:  /azure/architecture/reference-architectures/sap
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[sap-netweaver]: /azure/architecture/reference-architectures/sap/sap-netweaver
[sap-hana]: /azure/architecture/reference-architectures/sap/sap-s4hana
[sap-large]: /azure/architecture/reference-architectures/sap/hana-large-instances
[hub-spoke]: /azure/architecture/reference-architectures/hybrid-networking/hub-spoke