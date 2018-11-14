---
title: Az SAP-feladatokat az Azure-ban fejlesztési és tesztelési környezetek
description: Fejlesztési és tesztelési környezetet hozhat létre SAP számítási feladatokhoz.
author: AndrewDibbins
ms.date: 7/11/18
ms.openlocfilehash: 1cfd15b0287a1979ae5ad60e41a0b1627a2e115c
ms.sourcegitcommit: 0a31fad9b68d54e2858314ca5fe6cba6c6b95ae4
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/13/2018
ms.locfileid: "51610804"
---
# <a name="devtest-environments-for-sap-workloads-on-azure"></a><span data-ttu-id="1383c-103">Az SAP-feladatokat az Azure-ban fejlesztési és tesztelési környezetek</span><span class="sxs-lookup"><span data-stu-id="1383c-103">Dev/test environments for SAP workloads on Azure</span></span>

<span data-ttu-id="1383c-104">Ez a példa bemutatja, hogyan lehet létrehozni egy fejlesztési-tesztelési környezetet az SAP NetWeaver az olyan Windows vagy Linux-környezet az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="1383c-104">This example shows how to establish a dev/test environment for SAP NetWeaver in a Windows or Linux environment on Azure.</span></span> <span data-ttu-id="1383c-105">A használt adatbázisa AnyDB, az SAP kifejezés bármely támogatott adatbázis-kezelő (amely nem az SAP HANA).</span><span class="sxs-lookup"><span data-stu-id="1383c-105">The database used is AnyDB, the SAP term for any supported DBMS (that isn't SAP HANA).</span></span> <span data-ttu-id="1383c-106">Ez az architektúra nem éles környezetekhez lett kialakítva, mert csak egyetlen virtuális géphez (VM) üzemel, és annak méretét is módosítható a szervezet igényeinek megfelelően.</span><span class="sxs-lookup"><span data-stu-id="1383c-106">Because this architecture is designed for non-production environments, it's deployed with just a single virtual machine (VM) and it's size can be changed to accommodate your organization's needs.</span></span>

<span data-ttu-id="1383c-107">Éles környezetben használja esetben tekintse át az SAP referenciaarchitektúrák alatt érhető el:</span><span class="sxs-lookup"><span data-stu-id="1383c-107">For production use cases review the SAP reference architectures available below:</span></span>

* <span data-ttu-id="1383c-108">[SAP NetWeaver a AnyDB][sap-netweaver]</span><span class="sxs-lookup"><span data-stu-id="1383c-108">[SAP NetWeaver for AnyDB][sap-netweaver]</span></span>
* <span data-ttu-id="1383c-109">[SAP S/4HANA-T][sap-hana]</span><span class="sxs-lookup"><span data-stu-id="1383c-109">[SAP S/4HANA][sap-hana]</span></span>
* <span data-ttu-id="1383c-110">[Nagyméretű Azure-példányokon futó SAP][sap-large]</span><span class="sxs-lookup"><span data-stu-id="1383c-110">[SAP on Azure large instances][sap-large]</span></span>

## <a name="relevant-use-cases"></a><span data-ttu-id="1383c-111">Alkalmazási helyzetek</span><span class="sxs-lookup"><span data-stu-id="1383c-111">Relevant use cases</span></span>

<span data-ttu-id="1383c-112">Egyéb alkalmazási helyzetek a következők:</span><span class="sxs-lookup"><span data-stu-id="1383c-112">Other relevant use cases include:</span></span>

* <span data-ttu-id="1383c-113">A nem kritikus SAP nem termelési számítási feladatokhoz (védőfal, fejlesztési, tesztelési, minőségbiztosítási)</span><span class="sxs-lookup"><span data-stu-id="1383c-113">Non-critical SAP non-productive workloads (sandbox, development, test, quality assurance)</span></span>
* <span data-ttu-id="1383c-114">A nem kritikus fontosságú SAP számítási feladata</span><span class="sxs-lookup"><span data-stu-id="1383c-114">Non-critical SAP business workloads</span></span>

## <a name="architecture"></a><span data-ttu-id="1383c-115">Architektúra</span><span class="sxs-lookup"><span data-stu-id="1383c-115">Architecture</span></span>

![Az SAP-feladatokat a fejlesztési és tesztelési környezetek architektúra diagramja](media/architecture-sap-dev-test.png)

<span data-ttu-id="1383c-117">Ez a forgatókönyv azt mutatja be, az SAP-rendszer önálló adatbázis és SAP-alkalmazáskiszolgáló az egyetlen virtuális gép üzembe helyezése.</span><span class="sxs-lookup"><span data-stu-id="1383c-117">This scenario demonstrates provisioning a single SAP system database and SAP application server on a single virtual machine.</span></span> <span data-ttu-id="1383c-118">A áramlanak keresztül az adatok a forgatókönyv a következő:</span><span class="sxs-lookup"><span data-stu-id="1383c-118">The data flows through the scenario as follows:</span></span>

1. <span data-ttu-id="1383c-119">Az SAP felhasználói felület vagy más ügyféleszközök (az Excel, egy webes böngésző vagy egyéb webalkalmazás) az Azure-alapú SAP-rendszer elérésére használják.</span><span class="sxs-lookup"><span data-stu-id="1383c-119">Customers use the SAP user interface or other client tools (Excel, a web browser, or other web application) to access the Azure-based SAP system.</span></span>
2. <span data-ttu-id="1383c-120">Kapcsolat egy meglévő ExpressRoute használatával biztosítja.</span><span class="sxs-lookup"><span data-stu-id="1383c-120">Connectivity is provided through the use of an established ExpressRoute.</span></span> <span data-ttu-id="1383c-121">Az ExpressRoute-kapcsolat leállítása az Azure-ban, az ExpressRoute-átjárót.</span><span class="sxs-lookup"><span data-stu-id="1383c-121">The ExpressRoute connection is terminated in Azure at the ExpressRoute gateway.</span></span> <span data-ttu-id="1383c-122">A hálózati forgalom az ExpressRoute-átjárót az átjáró-alhálózathoz, valamint az átjáró-alhálózat az alkalmazásrétegek küllő alhálózathoz útvonalakat (lásd a [küllős] [ hub-spoke] minta) és a egy hálózati biztonsági keresztül Átjáró, a SAP alkalmazás virtuális géphez.</span><span class="sxs-lookup"><span data-stu-id="1383c-122">Network traffic routes through the ExpressRoute gateway to the gateway subnet, and from the gateway subnet to the application-tier spoke subnet (see the [hub-spoke][hub-spoke] pattern) and via a Network Security Gateway to the SAP application virtual machine.</span></span>
3. <span data-ttu-id="1383c-123">Az identitás felügyeleti kiszolgálók hitelesítési szolgáltatásokat nyújtanak.</span><span class="sxs-lookup"><span data-stu-id="1383c-123">The identity management servers provide authentication services.</span></span>
4. <span data-ttu-id="1383c-124">A jump boxon helyi felügyeleti képességeket biztosít.</span><span class="sxs-lookup"><span data-stu-id="1383c-124">The jump box provides local management capabilities.</span></span>

### <a name="components"></a><span data-ttu-id="1383c-125">Összetevők</span><span class="sxs-lookup"><span data-stu-id="1383c-125">Components</span></span>

* <span data-ttu-id="1383c-126">[Virtuális hálózatok](/azure/virtual-network/virtual-networks-overview) Azure-on belüli hálózati kommunikáció alapját képezi.</span><span class="sxs-lookup"><span data-stu-id="1383c-126">[Virtual Networks](/azure/virtual-network/virtual-networks-overview) are the basis of network communication within Azure.</span></span>
* <span data-ttu-id="1383c-127">[Virtuális gép](/azure/virtual-machines/windows/overview) Azure virtuális gépek igény szerinti, nagy léptékben méretezhető, biztonságos és virtualizált infrastruktúra Windows vagy Linux-kiszolgáló használatával biztosít.</span><span class="sxs-lookup"><span data-stu-id="1383c-127">[Virtual Machine](/azure/virtual-machines/windows/overview) Azure Virtual Machines provides on-demand, high-scale, secure, virtualized infrastructure using Windows or Linux Server.</span></span>
* <span data-ttu-id="1383c-128">[Az ExpressRoute](/azure/expressroute/expressroute-introduction) lehetővé teszi, kiterjesztheti helyszíni hálózatait a Microsoft cloud, amelyet egy kapcsolatszolgáltató biztosít egy privát kapcsolaton keresztül.</span><span class="sxs-lookup"><span data-stu-id="1383c-128">[ExpressRoute](/azure/expressroute/expressroute-introduction) lets you extend your on-premises networks into the Microsoft cloud over a private connection facilitated by a connectivity provider.</span></span>
* <span data-ttu-id="1383c-129">[Hálózati biztonsági csoport](/azure/virtual-network/security-overview) lehetővé teszi a hálózati virtuális hálózatban lévő erőforrásokra irányuló forgalmat.</span><span class="sxs-lookup"><span data-stu-id="1383c-129">[Network Security Group](/azure/virtual-network/security-overview) lets you limit network traffic to resources in a virtual network.</span></span> <span data-ttu-id="1383c-130">A hálózati biztonsági csoport egy biztonsági szabályokból álló listát tartalmaz, amelyek engedélyezik vagy megtagadják a bejövő vagy kimenő hálózati forgalmat a forrás vagy a cél IP-címe, illetve portok vagy protokollok alapján.</span><span class="sxs-lookup"><span data-stu-id="1383c-130">A network security group contains a list of security rules that allow or deny inbound or outbound network traffic based on source or destination IP address, port, and protocol.</span></span> 
* <span data-ttu-id="1383c-131">[Erőforráscsoportok](/azure/azure-resource-manager/resource-group-overview#resource-groups) logikai tárolóként szolgálnak az Azure-erőforrásokhoz.</span><span class="sxs-lookup"><span data-stu-id="1383c-131">[Resource Groups](/azure/azure-resource-manager/resource-group-overview#resource-groups) act as logical containers for Azure resources.</span></span>

## <a name="considerations"></a><span data-ttu-id="1383c-132">Megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="1383c-132">Considerations</span></span>

### <a name="availability"></a><span data-ttu-id="1383c-133">Rendelkezésre állás</span><span class="sxs-lookup"><span data-stu-id="1383c-133">Availability</span></span>

 <span data-ttu-id="1383c-134">A Microsoft kínál a szolgáltatásiszint-szerződés (SLA) egyetlen Virtuálisgép-példányokhoz.</span><span class="sxs-lookup"><span data-stu-id="1383c-134">Microsoft offers a service level agreement (SLA) for single VM instances.</span></span> <span data-ttu-id="1383c-135">További információ a Microsoft Azure szolgáltatásiszint-szerződés virtuális gépek [SLA-t a virtuális gépek](https://azure.microsoft.com/support/legal/sla/virtual-machines)</span><span class="sxs-lookup"><span data-stu-id="1383c-135">For more information on Microsoft Azure Service Level Agreement for Virtual Machines [SLA For Virtual Machines](https://azure.microsoft.com/support/legal/sla/virtual-machines)</span></span>

### <a name="scalability"></a><span data-ttu-id="1383c-136">Méretezhetőség</span><span class="sxs-lookup"><span data-stu-id="1383c-136">Scalability</span></span>

<span data-ttu-id="1383c-137">Általános méretezhető megoldások, tekintse át a [méretezési ellenőrzőlista] [ scalability] a az Azure Architecture Centert.</span><span class="sxs-lookup"><span data-stu-id="1383c-137">For general guidance on designing scalable solutions, see the [scalability checklist][scalability] in the Azure Architecture Center.</span></span>

### <a name="security"></a><span data-ttu-id="1383c-138">Biztonság</span><span class="sxs-lookup"><span data-stu-id="1383c-138">Security</span></span>

<span data-ttu-id="1383c-139">Általános megoldások biztonságos, tekintse át a [Azure Security dokumentációja][security].</span><span class="sxs-lookup"><span data-stu-id="1383c-139">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="1383c-140">Rugalmasság</span><span class="sxs-lookup"><span data-stu-id="1383c-140">Resiliency</span></span>

<span data-ttu-id="1383c-141">Rugalmas megoldások tervezésével kapcsolatos általános útmutatásért lásd: [rugalmas alkalmazások tervezése az Azure][resiliency].</span><span class="sxs-lookup"><span data-stu-id="1383c-141">For general guidance on designing resilient solutions, see [Designing resilient applications for Azure][resiliency].</span></span>

## <a name="pricing"></a><span data-ttu-id="1383c-142">Díjszabás</span><span class="sxs-lookup"><span data-stu-id="1383c-142">Pricing</span></span>

<span data-ttu-id="1383c-143">Ebben a forgatókönyvben futtatásával járó költségeket felfedezheti érdekében a szolgáltatások összes előre konfigurált, a következő költséget számító példa.</span><span class="sxs-lookup"><span data-stu-id="1383c-143">To help you explore the cost of running this scenario, all of the services are pre-configured in the cost calculator examples below.</span></span> <span data-ttu-id="1383c-144">Tekintse meg, hogyan díjszabását szeretné módosítani az adott használati esetekhez, módosítsa a megfelelő változókat egyezik a várt forgalomhoz.</span><span class="sxs-lookup"><span data-stu-id="1383c-144">To see how the pricing would change for your particular use case, change the appropriate variables to match your expected traffic.</span></span>

<span data-ttu-id="1383c-145">Adtunk meg négy példa költség profilok kaphat forgalom mennyisége alapján:</span><span class="sxs-lookup"><span data-stu-id="1383c-145">We have provided four sample cost profiles based on amount of traffic you expect to receive:</span></span>

|<span data-ttu-id="1383c-146">Méret</span><span class="sxs-lookup"><span data-stu-id="1383c-146">Size</span></span>|<span data-ttu-id="1383c-147">A SAP</span><span class="sxs-lookup"><span data-stu-id="1383c-147">SAPs</span></span>|<span data-ttu-id="1383c-148">Virtuális gép típusa</span><span class="sxs-lookup"><span data-stu-id="1383c-148">VM Type</span></span>|<span data-ttu-id="1383c-149">Storage</span><span class="sxs-lookup"><span data-stu-id="1383c-149">Storage</span></span>|<span data-ttu-id="1383c-150">Azure díjkalkulátor</span><span class="sxs-lookup"><span data-stu-id="1383c-150">Azure Pricing Calculator</span></span>|
|----|----|-------|-------|---------------|
|<span data-ttu-id="1383c-151">Kicsi</span><span class="sxs-lookup"><span data-stu-id="1383c-151">Small</span></span>|<span data-ttu-id="1383c-152">8000</span><span class="sxs-lookup"><span data-stu-id="1383c-152">8000</span></span>|<span data-ttu-id="1383c-153">D8s_v3</span><span class="sxs-lookup"><span data-stu-id="1383c-153">D8s_v3</span></span>|<span data-ttu-id="1383c-154">2xP20, 1xP10</span><span class="sxs-lookup"><span data-stu-id="1383c-154">2xP20, 1xP10</span></span>|[<span data-ttu-id="1383c-155">Kis</span><span class="sxs-lookup"><span data-stu-id="1383c-155">Small</span></span>](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1)|
|<span data-ttu-id="1383c-156">Közepes</span><span class="sxs-lookup"><span data-stu-id="1383c-156">Medium</span></span>|<span data-ttu-id="1383c-157">16000</span><span class="sxs-lookup"><span data-stu-id="1383c-157">16000</span></span>|<span data-ttu-id="1383c-158">D16s_v3</span><span class="sxs-lookup"><span data-stu-id="1383c-158">D16s_v3</span></span>|<span data-ttu-id="1383c-159">3xP20, 1xP10</span><span class="sxs-lookup"><span data-stu-id="1383c-159">3xP20, 1xP10</span></span>|[<span data-ttu-id="1383c-160">Közepes</span><span class="sxs-lookup"><span data-stu-id="1383c-160">Medium</span></span>](https://azure.com/e/465bd07047d148baab032b2f461550cd)|
<span data-ttu-id="1383c-161">Nagy</span><span class="sxs-lookup"><span data-stu-id="1383c-161">Large</span></span>|<span data-ttu-id="1383c-162">32000</span><span class="sxs-lookup"><span data-stu-id="1383c-162">32000</span></span>|<span data-ttu-id="1383c-163">E32s_v3</span><span class="sxs-lookup"><span data-stu-id="1383c-163">E32s_v3</span></span>|<span data-ttu-id="1383c-164">3xP20, 1xP10</span><span class="sxs-lookup"><span data-stu-id="1383c-164">3xP20, 1xP10</span></span>|[<span data-ttu-id="1383c-165">Nagy méretű</span><span class="sxs-lookup"><span data-stu-id="1383c-165">Large</span></span>](https://azure.com/e/ada2e849d68b41c3839cc976000c6931)|
<span data-ttu-id="1383c-166">Extra nagy</span><span class="sxs-lookup"><span data-stu-id="1383c-166">Extra Large</span></span>|<span data-ttu-id="1383c-167">64000</span><span class="sxs-lookup"><span data-stu-id="1383c-167">64000</span></span>|<span data-ttu-id="1383c-168">M64s</span><span class="sxs-lookup"><span data-stu-id="1383c-168">M64s</span></span>|<span data-ttu-id="1383c-169">4xP20, 1xP10</span><span class="sxs-lookup"><span data-stu-id="1383c-169">4xP20, 1xP10</span></span>|[<span data-ttu-id="1383c-170">Extra nagy</span><span class="sxs-lookup"><span data-stu-id="1383c-170">Extra Large</span></span>](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef)|

> [!NOTE]
> <span data-ttu-id="1383c-171">A díjszabás is egy útmutatót, amely csak azt jelzi, hogy a virtuális gépek és tárolási költségeit.</span><span class="sxs-lookup"><span data-stu-id="1383c-171">This pricing is a guide that only indicates the VMs and storage costs.</span></span> <span data-ttu-id="1383c-172">Nem tartalmazza a hálózati, biztonsági mentési tár, és a bejövő/kimenő adatforgalom díját.</span><span class="sxs-lookup"><span data-stu-id="1383c-172">It excludes networking, backup storage, and data ingress/egress charges.</span></span>

* <span data-ttu-id="1383c-173">[Kis](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1): egy kis rendszer áll VM-típus D8s_v3 8 x vCPUs, 32 GB RAM-MAL és 200 GB ideiglenes tárhely, emellett két 512 GB és a egy 128 GB-os prémium szintű tárolólemezeket.</span><span class="sxs-lookup"><span data-stu-id="1383c-173">[Small](https://azure.com/e/9d26b9612da9466bb7a800eab56e71d1): A small system consists of VM type D8s_v3 with 8x vCPUs, 32 GB RAM and 200 GB temp storage, additionally two 512 GB and one 128 GB premium storage disks.</span></span>
* <span data-ttu-id="1383c-174">[Közepes](https://azure.com/e/465bd07047d148baab032b2f461550cd): egy közepes méretű rendszer áll VM-típus D16s_v3 16 x vCPUs, 64 GB RAM-MAL és 400 GB ideiglenes tárhely, emellett három 512 GB és a egy 128 GB-os prémium szintű tárolólemezeket.</span><span class="sxs-lookup"><span data-stu-id="1383c-174">[Medium](https://azure.com/e/465bd07047d148baab032b2f461550cd): A medium system consists of VM type D16s_v3 with 16x vCPUs, 64 GB RAM and 400 GB temp storage, additionally three 512 GB and one 128 GB premium storage disks.</span></span>
* <span data-ttu-id="1383c-175">[Nagy](https://azure.com/e/ada2e849d68b41c3839cc976000c6931): egy nagy méretű rendszer áll VM-típus E32s_v3 32 x vCPUs, 256 GB RAM-MAL és 512 GB ideiglenes tárhely, emellett három 512 GB és a egy 128 GB-os prémium szintű tárolólemezeket.</span><span class="sxs-lookup"><span data-stu-id="1383c-175">[Large](https://azure.com/e/ada2e849d68b41c3839cc976000c6931): A large system consists of VM type E32s_v3 with 32x vCPUs, 256 GB RAM and 512 GB temp storage, additionally three 512GB and one 128GB premium storage disks.</span></span>
* <span data-ttu-id="1383c-176">[Extra nagy](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef): egy extra nagy méretű rendszer áll típusú, a virtuális gép M64s 64 x vCPUs, 1024 GB RAM-MAL és 2000 GB ideiglenes tárhely, továbbá négy 512 GB és a egy 128 GB-os prémium szintű tárolólemezeket.</span><span class="sxs-lookup"><span data-stu-id="1383c-176">[Extra Large](https://azure.com/e/975fb58a965c4fbbb54c5c9179c61cef): An extra large system consists of a VM type M64s with 64x vCPUs, 1024 GB RAM and 2000 GB temp storage, additionally four 512 GB and one 128 GB premium storage disks.</span></span>

## <a name="deployment"></a><span data-ttu-id="1383c-177">Környezet</span><span class="sxs-lookup"><span data-stu-id="1383c-177">Deployment</span></span>

<span data-ttu-id="1383c-178">Ebben a forgatókönyvben az alapul szolgáló infrastruktúra telepítéséhez kattintson ide.</span><span class="sxs-lookup"><span data-stu-id="1383c-178">Click here to deploy the underlying infrastructure for this scenario.</span></span>

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fsap-2tier%2Fazuredeploy.json" target="_blank">
    <img src="https://azuredeploy.net/deploybutton.png"/>
</a>

> [!NOTE]
> <span data-ttu-id="1383c-179">SAP és Oracle nincs telepítve a központi telepítés során.</span><span class="sxs-lookup"><span data-stu-id="1383c-179">SAP and Oracle are not installed during this deployment.</span></span> <span data-ttu-id="1383c-180">Ezek az összetevők telepítését külön-külön kell.</span><span class="sxs-lookup"><span data-stu-id="1383c-180">You will need to deploy these components separately.</span></span>

<!-- links -->
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[sap-netweaver]: /azure/architecture/reference-architectures/sap/sap-netweaver
[sap-hana]: /azure/architecture/reference-architectures/sap/sap-s4hana
[sap-large]: /azure/architecture/reference-architectures/sap/hana-large-instances
[hub-spoke]: /azure/architecture/reference-architectures/hybrid-networking/hub-spoke
