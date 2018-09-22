---
title: Áttelepítés egy örökölt webes alkalmazás egy Azure-beli API-alapú architektúra
description: Egy forgatókönyv-alapú módszer használata az Azure API Management egy örökölt webalkalmazás korszerűsítéséhez be.
author: begim
ms.date: 09/13/2018
ms.openlocfilehash: f29674bd464e977c24e93f768a33ef36d6a2fccc
ms.sourcegitcommit: b7e521ba317f4fcd3253c80ac0c0a355eaaa56c5
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 09/21/2018
ms.locfileid: "46534564"
---
# <a name="migrating-a-legacy-web-application-to-an-api-based-architecture-on-azure"></a><span data-ttu-id="37f31-103">Áttelepítés egy örökölt webes alkalmazás egy Azure-beli API-alapú architektúra</span><span class="sxs-lookup"><span data-stu-id="37f31-103">Migrating a Legacy Web Application to an API-based Architecture on Azure</span></span>

<span data-ttu-id="37f31-104">Egy e-kereskedelmi cég, az utazási iparág modernizálhatja van a régi böngésző alapú szoftververmeket.</span><span class="sxs-lookup"><span data-stu-id="37f31-104">An e-commerce company in the travel industry is modernizing their legacy browser-based software stack.</span></span> <span data-ttu-id="37f31-105">Míg a meglévő stack csak a monolitikus, néhány [SOAP-alapú HTTP-szolgáltatások] [ soap] létezik a legutóbbi projektből.</span><span class="sxs-lookup"><span data-stu-id="37f31-105">While their existing stack is mostly monolithic, some [SOAP-based HTTP services][soap] exist from a recent project.</span></span> <span data-ttu-id="37f31-106">A jövőben további bevételi a bevételek néhány, a belső szellemi tulajdonság (IP) beépített lett fontolgatják meg.</span><span class="sxs-lookup"><span data-stu-id="37f31-106">In the future, they are considering additional revenue streams from monetizing some of the internal Intellectual Property (IP) that's been built.</span></span>

<span data-ttu-id="37f31-107">A projekt célja cím műszaki adósságot, folyamatos karbantartást javítása és a szolgáltatások gyorsabban és kevesebb regresszió kézbesítendő engedélyezése.</span><span class="sxs-lookup"><span data-stu-id="37f31-107">The goal of the project is address technical debt, improve ongoing maintenance and enable features to be delivered faster and with less regression.</span></span>  <span data-ttu-id="37f31-108">A projekt követi a lépcsőzetes megközelítés az egyes lépésekhez a párhuzamosan futó kockázat elkerülése érdekében:</span><span class="sxs-lookup"><span data-stu-id="37f31-108">The project will follow a stepped approach to avoid risk, with some steps running in parallel:</span></span>

* <span data-ttu-id="37f31-109">A fejlesztési csapat fogja korszerűsítheti az alkalmazás háttér-, amely áll, amelyek a virtuális gépeken üzemeltetett relációs adatbázisok</span><span class="sxs-lookup"><span data-stu-id="37f31-109">The development team will modernize the application back-end, which is composed of relational databases hosted on VMs</span></span>
* <span data-ttu-id="37f31-110">Saját fejlesztésű fejlesztői csapat fog kiírni, új üzleti funkció, amely új HTTP API-k keresztül lesz közzétéve.</span><span class="sxs-lookup"><span data-stu-id="37f31-110">Their in-house development team will write new business functionality, which will be exposed over new HTTP APIs.</span></span>
* <span data-ttu-id="37f31-111">Egy összevont fejlesztői csapat új böngészőalapú felhasználói Felületet, amely az Azure-felhőben üzemeltetett fog létrehozni.</span><span class="sxs-lookup"><span data-stu-id="37f31-111">A contracted development team will build a new browser-based UI, which will be hosted on the Azure Cloud.</span></span>

<span data-ttu-id="37f31-112">Az új alkalmazások funkciók szakaszban kézbesítése és fog *fokozatosan cserélje le* a meglévő ügyfél-kiszolgáló böngészőalapú felhasználói felületi funkciók (üzemeltethető a helyszínen) e-kereskedelmi cég ma használja, amelyen a.</span><span class="sxs-lookup"><span data-stu-id="37f31-112">The new applications features will be delivered in stages and will *gradually replace* the existing browser-based client-server UI functionality (hosted on-premises) that powers their e-commerce business today.</span></span>

<span data-ttu-id="37f31-113">A felügyeleti csoport nem szeretné feleslegesen korszerűsítéséhez.</span><span class="sxs-lookup"><span data-stu-id="37f31-113">The management team does not want to unnecessarily modernize.</span></span> <span data-ttu-id="37f31-114">Szeretnének is, hogy a hatókör és a költségek kézben.</span><span class="sxs-lookup"><span data-stu-id="37f31-114">They also want to maintain control of scope and costs.</span></span>  <span data-ttu-id="37f31-115">Ehhez a döntést, megőrizheti a meglévő SOAP HTTP-szolgáltatások éltek meg.</span><span class="sxs-lookup"><span data-stu-id="37f31-115">To do this, they have made the decision to preserve their existing SOAP HTTP services.</span></span> <span data-ttu-id="37f31-116">Is, amelyeket a lehető legnagyobb mértékben meglévő felhasználói felület módosításainak korlátozza.</span><span class="sxs-lookup"><span data-stu-id="37f31-116">They also intend to limit changes to the existing UI as much as possible.</span></span>

<span data-ttu-id="37f31-117">[Az Azure API Management (API-M)] [ apim] módjává projektek követelmények és korlátozások alkalmazhatja.</span><span class="sxs-lookup"><span data-stu-id="37f31-117">[Azure API Management (API-M)][apim] can be utilized to address many of the projects requirements and constraints.</span></span>

## <a name="architecture"></a><span data-ttu-id="37f31-118">Architektúra</span><span class="sxs-lookup"><span data-stu-id="37f31-118">Architecture</span></span>

![Minta forgatókönyv-architektúra][architecture-diagram]

<span data-ttu-id="37f31-120">Az új felhasználói Felületet, Platform--szolgáltatásként az Azure-ban üzemeltetett, és mindkét meglévő HTTP API-k és az új HTTP API-k függ.</span><span class="sxs-lookup"><span data-stu-id="37f31-120">The new UI will be hosted as Platform-as-a-Service on Azure, and will depend on both existing HTTP APIs and the new HTTP APIs.</span></span>  <span data-ttu-id="37f31-121">Ezekkel az API-felületek jobb teljesítményt, egyszerűbb integráció és bővíthetőség jövőbeli engedélyezése több gondosan megtervezett beállítva elküldjük Önnek.</span><span class="sxs-lookup"><span data-stu-id="37f31-121">These APIs will ship with a more carefully designed set of interfaces enabling better performance, easier integration, and future extensibility.</span></span>

### <a name="components-and-security"></a><span data-ttu-id="37f31-122">Összetevők és biztonság</span><span class="sxs-lookup"><span data-stu-id="37f31-122">Components and Security</span></span>

1. <span data-ttu-id="37f31-123">A már meglévő webalkalmazás üzemeltethető a helyszínen továbbra is a meglévő webszolgáltatások, is üzemeltethető a helyszínen közvetlenül felhasználása.</span><span class="sxs-lookup"><span data-stu-id="37f31-123">The existing web application, hosted on-premises continues to directly consume the existing web services, also hosted on-premises.</span></span>
2. <span data-ttu-id="37f31-124">A már meglévő webalkalmazás hívást a meglévő HTTP-szolgáltatások nem változnak, ezek a belső hívások a vállalati hálózaton belül.</span><span class="sxs-lookup"><span data-stu-id="37f31-124">The calls from the existing web app to the existing HTTP services are unchanged, these are internal calls within the corporate network.</span></span>
3. <span data-ttu-id="37f31-125">A bejövő hívások létrejönnek a felhőből a meglévő belső szolgáltatások:</span><span class="sxs-lookup"><span data-stu-id="37f31-125">Inbound calls are made from the cloud to the existing internal services:</span></span>
    * <span data-ttu-id="37f31-126">A biztonsági csapat a kommunikációhoz, a vállalati tűzfalon, és a meglévő helyszíni szolgáltatások az Azure API-M példány érkező adatforgalom engedélyezéséhez [biztonságos átvitel (HTTPs és SSL) használatával][apim-ssl].</span><span class="sxs-lookup"><span data-stu-id="37f31-126">The security team allow traffic from the Azure API-M instance to communicate, through the corporate firewall, with the existing on-premises services, [using secure transport (HTTPs/SSL)][apim-ssl].</span></span>
    * <span data-ttu-id="37f31-127">Az üzemeltetési csapat lehetővé teszi az API-M példányban a szolgáltatásokhoz csak a bejövő hívások.</span><span class="sxs-lookup"><span data-stu-id="37f31-127">The operations team will allow only inbound calls to the services from the API-M instance.</span></span> <span data-ttu-id="37f31-128">Ennek a követelménynek úgy [engedélyezési-lista az API-M-példány IP-címét] [ apim-whitelist-ip] a vállalati hálózat határain belül.</span><span class="sxs-lookup"><span data-stu-id="37f31-128">This requirement is met by [white-listing the IP address of the API-M instance][apim-whitelist-ip] within the corporate network perimeter.</span></span>
    * <span data-ttu-id="37f31-129">Új modul úgy van konfigurálva, folyamatba a helyszíni HTTP szolgáltatások kérelem (számára, hogy csak a kívülről származó kapcsolatok), amely keresése és érvényesítése [egy tanúsítványt, amely API-M biztosít] [ apim-mutualcert-auth].</span><span class="sxs-lookup"><span data-stu-id="37f31-129">A new module is configured into the on-premise HTTP services request pipeline (to act upon ONLY those connections originating externally) that will check for and validate [a certificate which API-M will provide][apim-mutualcert-auth].</span></span>
4. <span data-ttu-id="37f31-130">Az új API-val:</span><span class="sxs-lookup"><span data-stu-id="37f31-130">The new API:</span></span>
    * <span data-ttu-id="37f31-131">Az illesztett csak az API-M példány, az API-homlokzat biztosít, amelyek révén az új API nem érhető el közvetlenül.</span><span class="sxs-lookup"><span data-stu-id="37f31-131">Is surfaced only through the API-M instance, which will provide the API facade, the new API won't be accessed directly.</span></span>
    * <span data-ttu-id="37f31-132">Fejleszti és közzétett egy [Azure PaaS webes API-alkalmazás][azure-api-apps].</span><span class="sxs-lookup"><span data-stu-id="37f31-132">Is developed and published as an [Azure PaaS Web API App][azure-api-apps].</span></span>
    * <span data-ttu-id="37f31-133">Az engedélyezési listán (keresztül [webalkalmazás beállítások][azure-appservice-ip-restrict]) csak fogadására a [API-M VIP][apim-faq-vip].</span><span class="sxs-lookup"><span data-stu-id="37f31-133">Is white-listed (via [Web App settings][azure-appservice-ip-restrict]) to accept only the [API-M VIP][apim-faq-vip].</span></span>
    * <span data-ttu-id="37f31-134">Üzemel az Azure Web Apps a biztonságos átvitel/SSL engedélyezve van.</span><span class="sxs-lookup"><span data-stu-id="37f31-134">Is hosted in Azure Web Apps with Secure Transport/SSL turned on.</span></span>
    * <span data-ttu-id="37f31-135">Engedélyezési bekapcsolta, [az Azure App Service által biztosított] [ azure-appservice-auth] Azure Active Directory és az OAuth2 használatával.</span><span class="sxs-lookup"><span data-stu-id="37f31-135">Has Authorization turned on, [provided by the Azure App Service][azure-appservice-auth] using Azure Active Directory and OAuth2.</span></span>
5. <span data-ttu-id="37f31-136">Az új böngésző-alapú webes alkalmazás függ az Azure API-Management-példány *mindkét* a meglévő HTTP API-t és az új API-t.</span><span class="sxs-lookup"><span data-stu-id="37f31-136">The new browser-based Web Application will depend on the Azure API-Management instance for *both* the existing HTTP API and the new API.</span></span>

<span data-ttu-id="37f31-137">Az API-M-példány az örökölt HTTP-szolgáltatások leképezése egy új API-szerződés lesz konfigurálva.</span><span class="sxs-lookup"><span data-stu-id="37f31-137">The API-M instance will be configured to map the legacy HTTP services to a new API contract.</span></span> <span data-ttu-id="37f31-138">Ezzel az új webes felhasználói Felületet nem észleli a integrációt a régi services és API-k és az új API-kat használja.</span><span class="sxs-lookup"><span data-stu-id="37f31-138">By doing this, the new Web UI is unaware it's integrating with a set of legacy services/APIs and new APIs.</span></span> <span data-ttu-id="37f31-139">A jövőben a projektcsapat fokozatosan port az új API-funkciókat, és vonja ki az eredeti szolgáltatások tervbe.</span><span class="sxs-lookup"><span data-stu-id="37f31-139">In the future, the project team plans to gradually port functionality to the new APIs and retire the original services.</span></span> <span data-ttu-id="37f31-140">Ezek a változások kezelésének belül API-M konfiguráció így nem érinti az előtér felhasználói felületi és átalakítását munkahelyi elkerülésére.</span><span class="sxs-lookup"><span data-stu-id="37f31-140">These changes will be handled within API-M configuration leaving the front-end UI unaffected and avoiding redevelopment work.</span></span>

### <a name="alternatives"></a><span data-ttu-id="37f31-141">Alternatív megoldások</span><span class="sxs-lookup"><span data-stu-id="37f31-141">Alternatives</span></span>

* <span data-ttu-id="37f31-142">Ha a szervezet infrastruktúrára áthelyezése teljes egészében az Azure-ba, beleértve az örökölt alkalmazásokat futtató virtuális géphez lett tervezési majd API-M továbbra is lenne nagyszerű lehetőséget bármely címezhető HTTP-végpontot is előtérrendszer óta.</span><span class="sxs-lookup"><span data-stu-id="37f31-142">If the organization was planning to move their infrastructure entirely to Azure, including the VMs hosting the legacy applications, then API-M would still be a great option since it can facade any addressable HTTP endpoint.</span></span>
* <span data-ttu-id="37f31-143">Ha az ügyfél döntött, hogy tartani a meglévő végpontokat, és nem tehető közzé őket nyilvánosan, az API Management-példány tudta csatolni egy [Azure Virtual Network (VNET)][azure-vnet]:</span><span class="sxs-lookup"><span data-stu-id="37f31-143">If the customer had decided to keep the existing endpoints private and not expose them publicly, their API Management instance could be linked to an [Azure Virtual Network (VNET)][azure-vnet]:</span></span>
  * <span data-ttu-id="37f31-144">Az egy [Azure lift & shift forgatókönyv] [ azure-vm-lift-shift] csatolva az üzembe helyezett Azure Virtual Network, az ügyfél sikerült közvetlenül érjék el a háttér-szolgáltatás a privát IP-címeken keresztül.</span><span class="sxs-lookup"><span data-stu-id="37f31-144">In an [Azure lift & shift scenario][azure-vm-lift-shift] linked to their deployed Azure Virtual Network, the customer could directly address the back-end service through private IP addresses.</span></span>
  * <span data-ttu-id="37f31-145">A helyi a forgatókönyvben az API Management-példány vissza a belső szolgáltatás keresztül közvetlenül a Microsoftnak a tudta elérni egy [Azure VPN Gateway és a helyek közötti IPSec VPN-kapcsolat] [ azure-vpn] vagy [Express Útvonal] [ azure-er] ami egy [hibrid Azure - On-Premises forgatókönyv][azure-hybrid].</span><span class="sxs-lookup"><span data-stu-id="37f31-145">In the on premises scenario, the API Management instance could reach back to the internal service privately via an [Azure VPN Gateway & Site-to-site IPSec VPN connection][azure-vpn] or [Express Route][azure-er] making this a [hybrid Azure - On-Premises scenario][azure-hybrid].</span></span>
* <span data-ttu-id="37f31-146">Akkor lehet, hogy az API Management-példány saját belső módban az API Management-példány üzembe helyezésével.</span><span class="sxs-lookup"><span data-stu-id="37f31-146">It's possible to keep the API Management instance private by deploying the API Management instance in Internal mode.</span></span> <span data-ttu-id="37f31-147">Az üzembe helyezés majd is használható egy [Azure Application Gateway] [ azure-appgw] nyilvános hozzáférésének engedélyezése az egyes API-k, belső, mások továbbra is.</span><span class="sxs-lookup"><span data-stu-id="37f31-147">The deployment could then be used with an [Azure Application Gateway][azure-appgw] to enable public access for some APIs while others remain internal.</span></span> <span data-ttu-id="37f31-148">További információ a [API-M, csatlakozás belső virtuális hálózathoz, a módban az itt látható.][apim-vnet-internal]</span><span class="sxs-lookup"><span data-stu-id="37f31-148">For more information on [connecting API-M, in internal mode, to a VNET, see here.][apim-vnet-internal]</span></span>

> [!NOTE]
> <span data-ttu-id="37f31-149">Általános információk az API Management egy virtuális hálózathoz való csatlakozás [itt.][apim-vnet]</span><span class="sxs-lookup"><span data-stu-id="37f31-149">For general information on connecting API Management to a VNET, [see here.][apim-vnet]</span></span>

### <a name="availability--scalability"></a><span data-ttu-id="37f31-150">Rendelkezésre állás és méretezhetőség</span><span class="sxs-lookup"><span data-stu-id="37f31-150">Availability & Scalability</span></span>

* <span data-ttu-id="37f31-151">Az Azure API Management lehet [horizontálisan felskálázott] [ apim-scaleout] válassza ki a tarifacsomagot, majd egységek.</span><span class="sxs-lookup"><span data-stu-id="37f31-151">Azure API Management can be [Scaled out][apim-scaleout] by choosing a pricing tier and then adding units.</span></span>
* <span data-ttu-id="37f31-152">Skálázás is megtörténhet, [automatikusan az automatikus skálázás][apim-autoscale].</span><span class="sxs-lookup"><span data-stu-id="37f31-152">Scaling also happen [automatically with auto scaling][apim-autoscale].</span></span>
* <span data-ttu-id="37f31-153">[Üzembe helyezése több régióban] [ apim-multi-regions] lehetővé válik a feladatátvétel beállításai és végezhető, a [prémium szintű][apim-pricing].</span><span class="sxs-lookup"><span data-stu-id="37f31-153">[Deploying across multiple regions][apim-multi-regions] will enable fail over options and can be done in the [Premium tier][apim-pricing].</span></span>
* <span data-ttu-id="37f31-154">Érdemes lehet [integrálása az Azure Application Insights][azure-apim-ai], amely is Felfed, metrikák keresztül [Azure Monitor] [ azure-mon] figyeléshez .</span><span class="sxs-lookup"><span data-stu-id="37f31-154">Consider [Integrating with Azure Application Insights][azure-apim-ai], which also surfaces metrics through [Azure Monitor][azure-mon] for monitoring.</span></span>

## <a name="deployment"></a><span data-ttu-id="37f31-155">Környezet</span><span class="sxs-lookup"><span data-stu-id="37f31-155">Deployment</span></span>

<span data-ttu-id="37f31-156">Első lépésként [Azure API Management szolgáltatáspéldány létrehozása a portálon.][apim-create]</span><span class="sxs-lookup"><span data-stu-id="37f31-156">To get started, [create an Azure API Management instance in the portal.][apim-create]</span></span>

<span data-ttu-id="37f31-157">Másik lehetőségként választhat egy meglévő Azure Resource Manager [gyorsindítási sablon] [ azure-quickstart-templates-apim] , amelyek igazodnak-e az adott használati esetekhez.</span><span class="sxs-lookup"><span data-stu-id="37f31-157">Alternatively, you can choose from an existing Azure Resource Manager [quickstart template][azure-quickstart-templates-apim] that aligns to your specific use case.</span></span>

## <a name="pricing"></a><span data-ttu-id="37f31-158">Díjszabás</span><span class="sxs-lookup"><span data-stu-id="37f31-158">Pricing</span></span>

<span data-ttu-id="37f31-159">Az API Management – fejlesztői, alapszintű, standard és prémium szintű négy szinten érhető el.</span><span class="sxs-lookup"><span data-stu-id="37f31-159">API Management is offered in four tiers – developer, basic, standard, and premium.</span></span>  <span data-ttu-id="37f31-160">A különbség a szolgáltatásszintek, a részletes útmutatást talál a [díjszabási útmutatóját itt az Azure API Management.][apim-pricing]</span><span class="sxs-lookup"><span data-stu-id="37f31-160">You can find detailed guidance on the difference in these tiers at the [Azure API Management pricing guidance here.][apim-pricing]</span></span>

<span data-ttu-id="37f31-161">Az API Management egységek hozzáadásával vagy eltávolításával méretezhető.</span><span class="sxs-lookup"><span data-stu-id="37f31-161">Customers can scale API Management by adding and removing units.</span></span> <span data-ttu-id="37f31-162">Az egyes egységek kapacitása az adott díjcsomagtól függ.</span><span class="sxs-lookup"><span data-stu-id="37f31-162">Each unit has capacity that depends on its tier.</span></span>

> [!NOTE]
> <span data-ttu-id="37f31-163">A fejlesztői csomag az API Management-szolgáltatások értékelése is használható, de nem használható éles környezetben.</span><span class="sxs-lookup"><span data-stu-id="37f31-163">The Developer tier can be used for evaluation of the API Management features, but should not be used for production.</span></span>

<span data-ttu-id="37f31-164">Előre jelzett költségek megtekintése és testreszabása az üzemelő példány van szüksége, módosíthatja a méretezési egységek számát, és az App Service-példányok a [Díjkalkulátor][pricing-calculator].</span><span class="sxs-lookup"><span data-stu-id="37f31-164">To view projected costs and customize to your deployment needs, you can modify the number of scale units and App Service instances in the [Pricing Calculator][pricing-calculator].</span></span>

## <a name="related-resources"></a><span data-ttu-id="37f31-165">Kapcsolódó erőforrások</span><span class="sxs-lookup"><span data-stu-id="37f31-165">Related Resources</span></span>

<span data-ttu-id="37f31-166">Tekintse meg a széles körű Azure API Management [dokumentációja és referencia cikkeket.][apim]</span><span class="sxs-lookup"><span data-stu-id="37f31-166">Check out the extensive Azure API Management [documentation and reference articles.][apim]</span></span>

<!-- links -->
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
[azure-vm-lift-shift]: https://azure.microsoft.com/en-gb/resources/azure-virtual-datacenter-lift-and-shift-guide/
[standard-pricing-calc]: https://azure.com/e/
[premium-pricing-calc]: https://azure.com/e/
[apim-pricing]:https://azure.microsoft.com/en-gb/pricing/details/api-management/
[azure-quickstart-templates-apim]:https://azure.microsoft.com/resources/templates/?term=API+Management&pageNumber=1
[soap]:https://en.wikipedia.org/wiki/SOAP
[architecture-diagram]: ./media/apim-api-scenario/architecture-apim-api-scenario.png
[pricing-calculator]: https://azure.com/e/0e916a861fac464db61342d378cc0bd6