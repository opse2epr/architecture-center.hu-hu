---
title: Webalkalmazás migrálása API-alapú architektúrába
titleSuffix: Azure Example Scenarios
description: Modernizálhatja régi webalkalmazásait az Azure API Management használatával.
author: begim
ms.date: 09/13/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.custom: fasttrack
social_image_url: /azure/architecture/example-scenario/apps/media/architecture-apim-api-scenario.png
ms.openlocfilehash: cf3d4b7ed7fce04e6688f68e382caeec78abd100
ms.sourcegitcommit: 3b15d65e7c35a19506e562c444343f8467b6a073
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/25/2019
ms.locfileid: "54907962"
---
# <a name="migrating-a-legacy-web-application-to-an-api-based-architecture-on-azure"></a><span data-ttu-id="d30a2-103">Régi webalkalmazás migrálása egy API-alapú architektúrába az Azure-ban</span><span class="sxs-lookup"><span data-stu-id="d30a2-103">Migrating a legacy web application to an API-based architecture on Azure</span></span>

<span data-ttu-id="d30a2-104">Egy e-kereskedelmi cég, az utazási iparág modernizálhatja van a régi böngésző alapú szoftververmeket.</span><span class="sxs-lookup"><span data-stu-id="d30a2-104">An e-commerce company in the travel industry is modernizing their legacy browser-based software stack.</span></span> <span data-ttu-id="d30a2-105">Míg a meglévő stack csak a monolitikus, néhány [SOAP-alapú HTTP-szolgáltatások] [ soap] létezik a legutóbbi projektből.</span><span class="sxs-lookup"><span data-stu-id="d30a2-105">While their existing stack is mostly monolithic, some [SOAP-based HTTP services][soap] exist from a recent project.</span></span> <span data-ttu-id="d30a2-106">Néhány belső szellemi tulajdon, amely már fejlesztik a bevételi lehetőségekről további bevételi létrehozásának fontolgatják.</span><span class="sxs-lookup"><span data-stu-id="d30a2-106">They are considering the creation of additional revenue streams to monetize some of the internal intellectual property that's been developed.</span></span>

<span data-ttu-id="d30a2-107">A projekt célok közé tartozik a címzési műszaki adósságot, folyamatos karbantartást javítása és felgyorsítja a kevesebb regresszió hibák funkcióinak fejlesztését.</span><span class="sxs-lookup"><span data-stu-id="d30a2-107">Goals for the project include addressing technical debt, improving ongoing maintenance, and accelerating feature development with fewer regression bugs.</span></span> <span data-ttu-id="d30a2-108">A projekt elkerülése érdekében a kockázat, az egyes lépésekhez párhuzamosan iteratív folyamat használja:</span><span class="sxs-lookup"><span data-stu-id="d30a2-108">The project will use an iterative process to avoid risk, with some steps performed in parallel:</span></span>

- <span data-ttu-id="d30a2-109">A fejlesztői csapat fogja korszerűsítheti a alkalmazás háttérben álló virtuális gépeken üzemeltetett relációs adatbázis.</span><span class="sxs-lookup"><span data-stu-id="d30a2-109">The development team will modernize the application back end, which is composed of relational databases hosted on VMs.</span></span>
- <span data-ttu-id="d30a2-110">A belső fejlesztésű fejlesztőcsapat ír üzleti funkciókat új HTTP API-k keresztül lesz közzétéve.</span><span class="sxs-lookup"><span data-stu-id="d30a2-110">The in-house development team will write new business functionality that will be exposed over new HTTP APIs.</span></span>
- <span data-ttu-id="d30a2-111">A szerződés fejlesztői csapat új böngészőalapú felhasználói Felületet, amely az Azure-ban üzemeltetett fog létrehozni.</span><span class="sxs-lookup"><span data-stu-id="d30a2-111">A contract development team will build a new browser-based UI, which will be hosted in Azure.</span></span>

<span data-ttu-id="d30a2-112">Új alkalmazás szolgáltatásai fázisában kézbesítése történik.</span><span class="sxs-lookup"><span data-stu-id="d30a2-112">New application features will be delivered in stages.</span></span> <span data-ttu-id="d30a2-113">Ezek a szolgáltatások fokozatosan lecseréli a meglévő ügyfél-kiszolgáló böngészőalapú felhasználói felületi funkciók (üzemeltethető a helyszínen) e-kereskedelmi cég ma használja, amelyen a.</span><span class="sxs-lookup"><span data-stu-id="d30a2-113">These features will gradually replace the existing browser-based client-server UI functionality (hosted on-premises) that powers their e-commerce business today.</span></span>

<span data-ttu-id="d30a2-114">A felügyeleti csoport nem szeretné feleslegesen korszerűsítéséhez.</span><span class="sxs-lookup"><span data-stu-id="d30a2-114">The management team does not want to modernize unnecessarily.</span></span> <span data-ttu-id="d30a2-115">Szeretnének is, hogy a hatókör és a költségek kézben.</span><span class="sxs-lookup"><span data-stu-id="d30a2-115">They also want to maintain control of scope and costs.</span></span> <span data-ttu-id="d30a2-116">Ehhez akkor döntött, megőrizheti a meglévő SOAP HTTP-szolgáltatások.</span><span class="sxs-lookup"><span data-stu-id="d30a2-116">To do this, they have decided to preserve their existing SOAP HTTP services.</span></span> <span data-ttu-id="d30a2-117">Ezek kíván minimalizálása érdekében a módosításokat a meglévő felhasználói felületre.</span><span class="sxs-lookup"><span data-stu-id="d30a2-117">They also intend to minimize changes to the existing UI.</span></span> <span data-ttu-id="d30a2-118">[Az Azure API Management (APIM)] [ apim] alkalmazhatja a projekt követelményeket és korlátokat számos megoldása érdekében.</span><span class="sxs-lookup"><span data-stu-id="d30a2-118">[Azure API Management (APIM)][apim] can be utilized to address many of the project's requirements and constraints.</span></span>

## <a name="architecture"></a><span data-ttu-id="d30a2-119">Architektúra</span><span class="sxs-lookup"><span data-stu-id="d30a2-119">Architecture</span></span>

![Architektúradiagram][architecture]

<span data-ttu-id="d30a2-121">Az új felhasználói felület platform (PaaS) alkalmazás az Azure-ban üzemeltetett, és a meglévő és az új HTTP API-k függ.</span><span class="sxs-lookup"><span data-stu-id="d30a2-121">The new UI will be hosted as a platform as a service (PaaS) application on Azure, and will depend on both existing and new HTTP APIs.</span></span> <span data-ttu-id="d30a2-122">Ezen API-k elküldjük Önnek jobban kialakított kapcsolatok engedélyezése a jobb teljesítmény érdekében, egyszerűbb integráció és bővíthetőség jövőbeli vannak beállítva.</span><span class="sxs-lookup"><span data-stu-id="d30a2-122">These APIs will ship with a better-designed set of interfaces enabling better performance, easier integration, and future extensibility.</span></span>

### <a name="components-and-security"></a><span data-ttu-id="d30a2-123">Összetevők és biztonság</span><span class="sxs-lookup"><span data-stu-id="d30a2-123">Components and Security</span></span>

1. <span data-ttu-id="d30a2-124">A meglévő helyszíni webes alkalmazás továbbra is közvetlenül a a meglévő helyszíni webszolgáltatások felhasználása.</span><span class="sxs-lookup"><span data-stu-id="d30a2-124">The existing on-premises web application will continue to directly consume the existing on-premises web services.</span></span>
2. <span data-ttu-id="d30a2-125">A meglévő HTTP-szolgáltatások és a meglévő webes alkalmazás közötti hívások változatlan marad.</span><span class="sxs-lookup"><span data-stu-id="d30a2-125">Calls from the existing web app to the existing HTTP services will remain unchanged.</span></span> <span data-ttu-id="d30a2-126">Hívásokat a rendszer belső, a vállalati hálózathoz.</span><span class="sxs-lookup"><span data-stu-id="d30a2-126">These calls are internal to the corporate network.</span></span>
3. <span data-ttu-id="d30a2-127">A bejövő hívások a meglévő belső szolgáltatások létrejönnek az Azure-ból:</span><span class="sxs-lookup"><span data-stu-id="d30a2-127">Inbound calls are made from Azure to the existing internal services:</span></span>
    - <span data-ttu-id="d30a2-128">A biztonsági csapat, lehetővé teszi az APIM-példányra átadni a vállalati tűzfalon keresztül a meglévő helyszíni szolgáltatásokhoz érkező forgalom [biztonságos átvitel (HTTPS és SSL) használatával][apim-ssl].</span><span class="sxs-lookup"><span data-stu-id="d30a2-128">The security team allows traffic from the APIM instance to pass through the corporate firewall to the existing on-premises services [using secure transport (HTTPS/SSL)][apim-ssl].</span></span>
    - <span data-ttu-id="d30a2-129">Az üzemeltetési csapat lehetővé teszi a szolgáltatások csak az APIM-példányra a bejövő hívások.</span><span class="sxs-lookup"><span data-stu-id="d30a2-129">The operations team will allow inbound calls to the services only from the APIM instance.</span></span> <span data-ttu-id="d30a2-130">Ennek a követelménynek úgy [engedélyezési-lista az IP-címet az APIM-példány] [ apim-whitelist-ip] a vállalati hálózat határain belül.</span><span class="sxs-lookup"><span data-stu-id="d30a2-130">This requirement is met by [white-listing the IP address of the APIM instance][apim-whitelist-ip] within the corporate network perimeter.</span></span>
    - <span data-ttu-id="d30a2-131">Új modul a helyszíni HTTP szolgáltatások kérelem folyamatba van konfigurálva (számára, hogy **csak** e kívülről származó kapcsolatok), amely ellenőrzi [egy tanúsítványt, amely APIM biztosít] [apim-mutualcert-auth].</span><span class="sxs-lookup"><span data-stu-id="d30a2-131">A new module is configured into the on-premises HTTP services request pipeline (to act upon **only** those connections originating externally), which will validate [a certificate which APIM will provide][apim-mutualcert-auth].</span></span>
4. <span data-ttu-id="d30a2-132">Az új API-val:</span><span class="sxs-lookup"><span data-stu-id="d30a2-132">The new API:</span></span>
    - <span data-ttu-id="d30a2-133">Van végzetesnek csak az APIM-példányra, amely az API-homlokzat biztosít.</span><span class="sxs-lookup"><span data-stu-id="d30a2-133">Is surfaced only through the APIM instance, which will provide the API facade.</span></span> <span data-ttu-id="d30a2-134">Az új API-t közvetlenül nem érhetők el.</span><span class="sxs-lookup"><span data-stu-id="d30a2-134">The new API won't be accessed directly.</span></span>
    - <span data-ttu-id="d30a2-135">Fejleszti és közzétett egy [Azure PaaS webes API-alkalmazás][azure-api-apps].</span><span class="sxs-lookup"><span data-stu-id="d30a2-135">Is developed and published as an [Azure PaaS Web API App][azure-api-apps].</span></span>
    - <span data-ttu-id="d30a2-136">Az engedélyezési listán (keresztül [webalkalmazás beállítások][azure-appservice-ip-restrict]) csak fogadására a [APIM VIP][apim-faq-vip].</span><span class="sxs-lookup"><span data-stu-id="d30a2-136">Is white-listed (via [Web App settings][azure-appservice-ip-restrict]) to accept only the [APIM VIP][apim-faq-vip].</span></span>
    - <span data-ttu-id="d30a2-137">Üzemel az Azure Web Apps a biztonságos átvitel/SSL engedélyezve van.</span><span class="sxs-lookup"><span data-stu-id="d30a2-137">Is hosted in Azure Web Apps with Secure Transport/SSL turned on.</span></span>
    - <span data-ttu-id="d30a2-138">Van engedélyezve, engedélyezési [az Azure App Service által biztosított] [ azure-appservice-auth] Azure Active Directory és az OAuth2 használatával.</span><span class="sxs-lookup"><span data-stu-id="d30a2-138">Has authorization enabled, [provided by the Azure App Service][azure-appservice-auth] using Azure Active Directory and OAuth2.</span></span>
5. <span data-ttu-id="d30a2-139">Az új böngésző-alapú webes alkalmazás függ az Azure API Management-példány **mindkét** a meglévő HTTP API-t és az új API-t.</span><span class="sxs-lookup"><span data-stu-id="d30a2-139">The new browser-based web application will depend on the Azure API Management instance for **both** the existing HTTP API and the new API.</span></span>

<span data-ttu-id="d30a2-140">Az örökölt HTTP-szolgáltatások leképezése egy új API-szerződés az APIM-példányra lesz konfigurálva.</span><span class="sxs-lookup"><span data-stu-id="d30a2-140">The APIM instance will be configured to map the legacy HTTP services to a new API contract.</span></span> <span data-ttu-id="d30a2-141">Ezzel az új webes felhasználói Felületet nem észleli, az integráció az örökölt services és API-k és az új API-k készlete.</span><span class="sxs-lookup"><span data-stu-id="d30a2-141">By doing this, the new Web UI is unaware of the integration with a set of legacy services/APIs and new APIs.</span></span> <span data-ttu-id="d30a2-142">A jövőben a projektcsapat fog fokozatosan port az új API-funkciókat és kivonása az eredeti szolgáltatásokat.</span><span class="sxs-lookup"><span data-stu-id="d30a2-142">In the future, the project team will gradually port functionality to the new APIs and retire the original services.</span></span> <span data-ttu-id="d30a2-143">Ezek a változások kezelésének belül APIM-konfiguráció, így nem érinti az előtér felhasználói felületi és átalakítását munkahelyi elkerülésére.</span><span class="sxs-lookup"><span data-stu-id="d30a2-143">These changes will be handled within APIM configuration, leaving the front-end UI unaffected and avoiding redevelopment work.</span></span>

### <a name="alternatives"></a><span data-ttu-id="d30a2-144">Alternatív megoldások</span><span class="sxs-lookup"><span data-stu-id="d30a2-144">Alternatives</span></span>

- <span data-ttu-id="d30a2-145">Ha a szervezet infrastruktúrára áthelyezése teljes egészében az Azure-ba, beleértve az örökölt alkalmazásokat futtató virtuális géphez lett tervezési majd APIM továbbra is lenne nagyszerű lehetőséget óta bármely címezhető HTTP-végpont az előtérrendszer működhet.</span><span class="sxs-lookup"><span data-stu-id="d30a2-145">If the organization was planning to move their infrastructure entirely to Azure, including the VMs hosting the legacy applications, then APIM would still be a great option since it can act as a facade for any addressable HTTP endpoint.</span></span>
- <span data-ttu-id="d30a2-146">Ha az ügyfél döntött, hogy tartani a meglévő végpontokat, és nem tehető közzé őket nyilvánosan, az API Management-példány tudta csatolni egy [Azure Virtual Network (VNet)][azure-vnet]:</span><span class="sxs-lookup"><span data-stu-id="d30a2-146">If the customer had decided to keep the existing endpoints private and not expose them publicly, their API Management instance could be linked to an [Azure Virtual Network (VNet)][azure-vnet]:</span></span>
  - <span data-ttu-id="d30a2-147">Az egy [Azure átemeléses forgatókönyv] [ azure-vm-lift-shift] csatolva az üzembe helyezett Azure Virtual Network, az ügyfél sikerült közvetlenül érjék el a háttér-szolgáltatás a privát IP-címeken keresztül.</span><span class="sxs-lookup"><span data-stu-id="d30a2-147">In an [Azure lift and shift scenario][azure-vm-lift-shift] linked to their deployed Azure Virtual Network, the customer could directly address the back-end service through private IP addresses.</span></span>
  - <span data-ttu-id="d30a2-148">Helyszíni esetben az API Management-példány tudta elérni a belső szolgáltatás közvetlenül a Microsoftnak keresztül vissza a egy [Azure VPN gateway és a helyek közötti IPSec VPN-kapcsolat] [ azure-vpn] vagy [ Az ExpressRoute] [ azure-er] ami egy [hibrid Azure és helyszíni megoldásként][azure-hybrid].</span><span class="sxs-lookup"><span data-stu-id="d30a2-148">In the on-premises scenario, the API Management instance could reach back to the internal service privately via an [Azure VPN gateway and site-to-site IPSec VPN connection][azure-vpn] or [ExpressRoute][azure-er] making this a [hybrid Azure and on-premises scenario][azure-hybrid].</span></span>
- <span data-ttu-id="d30a2-149">Az API Management-példány képes tartani privát belső módban az API Management-példány üzembe helyezésével.</span><span class="sxs-lookup"><span data-stu-id="d30a2-149">The API Management instance can be kept private by deploying the API Management instance in Internal mode.</span></span> <span data-ttu-id="d30a2-150">Az üzembe helyezés majd is használható egy [Azure Application Gateway] [ azure-appgw] nyilvános hozzáférésének engedélyezése az egyes API-k, belső, mások továbbra is.</span><span class="sxs-lookup"><span data-stu-id="d30a2-150">The deployment could then be used with an [Azure Application Gateway][azure-appgw] to enable public access for some APIs while others remain internal.</span></span> <span data-ttu-id="d30a2-151">További információkért lásd: [APIM csatlakoztatása virtuális hálózathoz belső módban][apim-vnet-internal].</span><span class="sxs-lookup"><span data-stu-id="d30a2-151">For more information, see [Connecting APIM in internal mode to a VNET][apim-vnet-internal].</span></span>

> [!NOTE]
> <span data-ttu-id="d30a2-152">Általános információk az API Management egy virtuális hálózathoz való csatlakozás [Itt][apim-vnet].</span><span class="sxs-lookup"><span data-stu-id="d30a2-152">For general information on connecting API Management to a VNET, [see here][apim-vnet].</span></span>

### <a name="availability-and-scalability"></a><span data-ttu-id="d30a2-153">Rendelkezésre állás és méretezhetőség</span><span class="sxs-lookup"><span data-stu-id="d30a2-153">Availability and scalability</span></span>

- <span data-ttu-id="d30a2-154">Az Azure API Management lehet [horizontálisan felskálázott] [ apim-scaleout] válassza ki a tarifacsomagot, majd egységek.</span><span class="sxs-lookup"><span data-stu-id="d30a2-154">Azure API Management can be [scaled out][apim-scaleout] by choosing a pricing tier and then adding units.</span></span>
- <span data-ttu-id="d30a2-155">Skálázás is megtörténhet, [automatikusan az automatikus skálázás][apim-autoscale].</span><span class="sxs-lookup"><span data-stu-id="d30a2-155">Scaling also happen [automatically with auto scaling][apim-autoscale].</span></span>
- <span data-ttu-id="d30a2-156">[Üzembe helyezése több régióban] [ apim-multi-regions] lehetővé válik a feladatátvétel beállításai és végezhető, a [prémium szintű][apim-pricing].</span><span class="sxs-lookup"><span data-stu-id="d30a2-156">[Deploying across multiple regions][apim-multi-regions] will enable fail over options and can be done in the [Premium tier][apim-pricing].</span></span>
- <span data-ttu-id="d30a2-157">Érdemes lehet [integrálása az Azure Application Insights][azure-apim-ai], amely is Felfed, metrikák keresztül [Azure Monitor] [ azure-mon] figyeléshez .</span><span class="sxs-lookup"><span data-stu-id="d30a2-157">Consider [Integrating with Azure Application Insights][azure-apim-ai], which also surfaces metrics through [Azure Monitor][azure-mon] for monitoring.</span></span>

## <a name="deploy-the-scenario"></a><span data-ttu-id="d30a2-158">A forgatókönyv megvalósításához</span><span class="sxs-lookup"><span data-stu-id="d30a2-158">Deploy the scenario</span></span>

<span data-ttu-id="d30a2-159">Első lépésként [Azure API Management szolgáltatáspéldány létrehozása a portálon.][apim-create]</span><span class="sxs-lookup"><span data-stu-id="d30a2-159">To get started, [create an Azure API Management instance in the portal.][apim-create]</span></span>

<span data-ttu-id="d30a2-160">Másik lehetőségként választhat egy meglévő Azure Resource Manager [gyorsindítási sablon] [ azure-quickstart-templates-apim] , amelyek igazodnak-e az adott használati esetekhez.</span><span class="sxs-lookup"><span data-stu-id="d30a2-160">Alternatively, you can choose from an existing Azure Resource Manager [quickstart template][azure-quickstart-templates-apim] that aligns to your specific use case.</span></span>

## <a name="pricing"></a><span data-ttu-id="d30a2-161">Díjszabás</span><span class="sxs-lookup"><span data-stu-id="d30a2-161">Pricing</span></span>

<span data-ttu-id="d30a2-162">Az API Management négy szinten érhető el: fejlesztői, alapszintű, standard és prémium.</span><span class="sxs-lookup"><span data-stu-id="d30a2-162">API Management is offered in four tiers: developer, basic, standard, and premium.</span></span> <span data-ttu-id="d30a2-163">A különbség a szolgáltatásszintek, a részletes útmutatást talál a [díjszabási útmutatóját itt az Azure API Management.][apim-pricing]</span><span class="sxs-lookup"><span data-stu-id="d30a2-163">You can find detailed guidance on the difference in these tiers at the [Azure API Management pricing guidance here.][apim-pricing]</span></span>

<span data-ttu-id="d30a2-164">Az API Management egységek hozzáadásával vagy eltávolításával méretezhető.</span><span class="sxs-lookup"><span data-stu-id="d30a2-164">Customers can scale API Management by adding and removing units.</span></span> <span data-ttu-id="d30a2-165">Az egyes egységek kapacitása az adott díjcsomagtól függ.</span><span class="sxs-lookup"><span data-stu-id="d30a2-165">Each unit has capacity that depends on its tier.</span></span>

> [!NOTE]
> <span data-ttu-id="d30a2-166">A fejlesztői csomag az API Management-szolgáltatások értékelése is használható.</span><span class="sxs-lookup"><span data-stu-id="d30a2-166">The Developer tier can be used for evaluation of the API Management features.</span></span> <span data-ttu-id="d30a2-167">A fejlesztői csomag nem használható éles üzemi környezetek részei.</span><span class="sxs-lookup"><span data-stu-id="d30a2-167">The Developer tier should not be used for production.</span></span>

<span data-ttu-id="d30a2-168">Előre jelzett költségek megtekintése és testreszabása az üzemelő példány van szüksége, módosíthatja a méretezési egységek számát, és az App Service-példányok a [Azure Díjkalkulátor][pricing-calculator].</span><span class="sxs-lookup"><span data-stu-id="d30a2-168">To view projected costs and customize to your deployment needs, you can modify the number of scale units and App Service instances in the [Azure Pricing Calculator][pricing-calculator].</span></span>

## <a name="related-resources"></a><span data-ttu-id="d30a2-169">Kapcsolódó források (lehet, hogy a cikkek angol nyelvűek)</span><span class="sxs-lookup"><span data-stu-id="d30a2-169">Related resources</span></span>

<span data-ttu-id="d30a2-170">Tekintse át a széles körű Azure API Management [dokumentációja és referencia cikkek][apim].</span><span class="sxs-lookup"><span data-stu-id="d30a2-170">Review the extensive Azure API Management [documentation and reference articles][apim].</span></span>

<!-- links -->

[architecture]: ./media/architecture-apim-api-scenario.png
[apim-create]: /azure/api-management/get-started-create-service-instance
[apim-git]: /azure/api-management/api-management-configuration-repository-git
[apim-multi-regions]: /azure/api-management/api-management-howto-deploy-multi-region
[apim-autoscale]: /azure/api-management/api-management-howto-autoscale
[apim-scaleout]: /azure/api-management/upgrade-and-scale
[azure-apim-ai]: /azure/api-management/api-management-howto-app-insights
[azure-ai]: /azure/application-insights/
[azure-mon]: /azure/monitoring-and-diagnostics/monitoring-overview
[azure-appgw]: /azure/application-gateway/application-gateway-introduction
[apim-vnet-internal]: /azure/api-management/api-management-howto-integrate-internal-vnet-appgateway
[apim-vnet]: /azure/api-management/api-management-using-with-vnet
[azure-hybrid]: /azure/architecture/reference-architectures/hybrid-networking/
[azure-er]: /azure/expressroute/expressroute-introduction
[azure-vpn]: /azure/vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal
[azure-vnet]: /azure/virtual-network/virtual-networks-overview
[azure-appservice-auth]: /azure/app-service/app-service-authentication-overview#identity-providers
[apim-faq-vip]: /azure/api-management/api-management-faq#is-the-api-management-gateway-ip-address-constant-can-i-use-it-in-firewall-rules
[azure-appservice-ip-restrict]: /azure/app-service/app-service-ip-restrictions
[azure-api-apps]: /azure/app-service/
[apim-ssl]: /azure/api-management/api-management-howto-manage-protocols-ciphers
[apim-mutualcert-auth]: /azure/api-management/api-management-howto-mutual-certificates
[apim-whitelist-ip]: /azure/api-management/api-management-faq#is-the-api-management-gateway-ip-address-constant-can-i-use-it-in-firewall-rules
[anti-corruption-layer-pattern]: /azure/architecture/patterns/anti-corruption-layer
[apim]: /azure/api-management/api-management-key-concepts
[apim-api-design-guidance]: /azure/architecture/best-practices/api-design
[visualstudio-youtube-solid-design]: https://youtu.be/agkWYPUcLpg
[azure-vm-lift-shift]: https://azure.microsoft.com/resources/azure-virtual-datacenter-lift-and-shift-guide/
[standard-pricing-calc]: https://azure.com/e/
[premium-pricing-calc]: https://azure.com/e/
[apim-pricing]: https://azure.microsoft.com/pricing/details/api-management/
[azure-quickstart-templates-apim]: https://azure.microsoft.com/resources/templates/?term=API+Management&pageNumber=1
[soap]: https://en.wikipedia.org/wiki/SOAP
[pricing-calculator]: https://azure.com/e/0e916a861fac464db61342d378cc0bd6
