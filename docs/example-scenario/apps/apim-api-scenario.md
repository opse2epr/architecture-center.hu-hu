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
# <a name="migrating-a-legacy-web-application-to-an-api-based-architecture-on-azure"></a>Áttelepítés egy örökölt webes alkalmazás egy Azure-beli API-alapú architektúra

Egy e-kereskedelmi cég, az utazási iparág modernizálhatja van a régi böngésző alapú szoftververmeket. Míg a meglévő stack csak a monolitikus, néhány [SOAP-alapú HTTP-szolgáltatások] [ soap] létezik a legutóbbi projektből. A jövőben további bevételi a bevételek néhány, a belső szellemi tulajdonság (IP) beépített lett fontolgatják meg.

A projekt célja cím műszaki adósságot, folyamatos karbantartást javítása és a szolgáltatások gyorsabban és kevesebb regresszió kézbesítendő engedélyezése.  A projekt követi a lépcsőzetes megközelítés az egyes lépésekhez a párhuzamosan futó kockázat elkerülése érdekében:

* A fejlesztési csapat fogja korszerűsítheti az alkalmazás háttér-, amely áll, amelyek a virtuális gépeken üzemeltetett relációs adatbázisok
* Saját fejlesztésű fejlesztői csapat fog kiírni, új üzleti funkció, amely új HTTP API-k keresztül lesz közzétéve.
* Egy összevont fejlesztői csapat új böngészőalapú felhasználói Felületet, amely az Azure-felhőben üzemeltetett fog létrehozni.

Az új alkalmazások funkciók szakaszban kézbesítése és fog *fokozatosan cserélje le* a meglévő ügyfél-kiszolgáló böngészőalapú felhasználói felületi funkciók (üzemeltethető a helyszínen) e-kereskedelmi cég ma használja, amelyen a.

A felügyeleti csoport nem szeretné feleslegesen korszerűsítéséhez. Szeretnének is, hogy a hatókör és a költségek kézben.  Ehhez a döntést, megőrizheti a meglévő SOAP HTTP-szolgáltatások éltek meg. Is, amelyeket a lehető legnagyobb mértékben meglévő felhasználói felület módosításainak korlátozza.

[Az Azure API Management (API-M)] [ apim] módjává projektek követelmények és korlátozások alkalmazhatja.

## <a name="architecture"></a>Architektúra

![Minta forgatókönyv-architektúra][architecture-diagram]

Az új felhasználói Felületet, Platform--szolgáltatásként az Azure-ban üzemeltetett, és mindkét meglévő HTTP API-k és az új HTTP API-k függ.  Ezekkel az API-felületek jobb teljesítményt, egyszerűbb integráció és bővíthetőség jövőbeli engedélyezése több gondosan megtervezett beállítva elküldjük Önnek.

### <a name="components-and-security"></a>Összetevők és biztonság

1. A már meglévő webalkalmazás üzemeltethető a helyszínen továbbra is a meglévő webszolgáltatások, is üzemeltethető a helyszínen közvetlenül felhasználása.
2. A már meglévő webalkalmazás hívást a meglévő HTTP-szolgáltatások nem változnak, ezek a belső hívások a vállalati hálózaton belül.
3. A bejövő hívások létrejönnek a felhőből a meglévő belső szolgáltatások:
    * A biztonsági csapat a kommunikációhoz, a vállalati tűzfalon, és a meglévő helyszíni szolgáltatások az Azure API-M példány érkező adatforgalom engedélyezéséhez [biztonságos átvitel (HTTPs és SSL) használatával][apim-ssl].
    * Az üzemeltetési csapat lehetővé teszi az API-M példányban a szolgáltatásokhoz csak a bejövő hívások. Ennek a követelménynek úgy [engedélyezési-lista az API-M-példány IP-címét] [ apim-whitelist-ip] a vállalati hálózat határain belül.
    * Új modul úgy van konfigurálva, folyamatba a helyszíni HTTP szolgáltatások kérelem (számára, hogy csak a kívülről származó kapcsolatok), amely keresése és érvényesítése [egy tanúsítványt, amely API-M biztosít] [ apim-mutualcert-auth].
4. Az új API-val:
    * Az illesztett csak az API-M példány, az API-homlokzat biztosít, amelyek révén az új API nem érhető el közvetlenül.
    * Fejleszti és közzétett egy [Azure PaaS webes API-alkalmazás][azure-api-apps].
    * Az engedélyezési listán (keresztül [webalkalmazás beállítások][azure-appservice-ip-restrict]) csak fogadására a [API-M VIP][apim-faq-vip].
    * Üzemel az Azure Web Apps a biztonságos átvitel/SSL engedélyezve van.
    * Engedélyezési bekapcsolta, [az Azure App Service által biztosított] [ azure-appservice-auth] Azure Active Directory és az OAuth2 használatával.
5. Az új böngésző-alapú webes alkalmazás függ az Azure API-Management-példány *mindkét* a meglévő HTTP API-t és az új API-t.

Az API-M-példány az örökölt HTTP-szolgáltatások leképezése egy új API-szerződés lesz konfigurálva. Ezzel az új webes felhasználói Felületet nem észleli a integrációt a régi services és API-k és az új API-kat használja. A jövőben a projektcsapat fokozatosan port az új API-funkciókat, és vonja ki az eredeti szolgáltatások tervbe. Ezek a változások kezelésének belül API-M konfiguráció így nem érinti az előtér felhasználói felületi és átalakítását munkahelyi elkerülésére.

### <a name="alternatives"></a>Alternatív megoldások

* Ha a szervezet infrastruktúrára áthelyezése teljes egészében az Azure-ba, beleértve az örökölt alkalmazásokat futtató virtuális géphez lett tervezési majd API-M továbbra is lenne nagyszerű lehetőséget bármely címezhető HTTP-végpontot is előtérrendszer óta.
* Ha az ügyfél döntött, hogy tartani a meglévő végpontokat, és nem tehető közzé őket nyilvánosan, az API Management-példány tudta csatolni egy [Azure Virtual Network (VNET)][azure-vnet]:
  * Az egy [Azure lift & shift forgatókönyv] [ azure-vm-lift-shift] csatolva az üzembe helyezett Azure Virtual Network, az ügyfél sikerült közvetlenül érjék el a háttér-szolgáltatás a privát IP-címeken keresztül.
  * A helyi a forgatókönyvben az API Management-példány vissza a belső szolgáltatás keresztül közvetlenül a Microsoftnak a tudta elérni egy [Azure VPN Gateway és a helyek közötti IPSec VPN-kapcsolat] [ azure-vpn] vagy [Express Útvonal] [ azure-er] ami egy [hibrid Azure - On-Premises forgatókönyv][azure-hybrid].
* Akkor lehet, hogy az API Management-példány saját belső módban az API Management-példány üzembe helyezésével. Az üzembe helyezés majd is használható egy [Azure Application Gateway] [ azure-appgw] nyilvános hozzáférésének engedélyezése az egyes API-k, belső, mások továbbra is. További információ a [API-M, csatlakozás belső virtuális hálózathoz, a módban az itt látható.][apim-vnet-internal]

> [!NOTE]
> Általános információk az API Management egy virtuális hálózathoz való csatlakozás [itt.][apim-vnet]

### <a name="availability--scalability"></a>Rendelkezésre állás és méretezhetőség

* Az Azure API Management lehet [horizontálisan felskálázott] [ apim-scaleout] válassza ki a tarifacsomagot, majd egységek.
* Skálázás is megtörténhet, [automatikusan az automatikus skálázás][apim-autoscale].
* [Üzembe helyezése több régióban] [ apim-multi-regions] lehetővé válik a feladatátvétel beállításai és végezhető, a [prémium szintű][apim-pricing].
* Érdemes lehet [integrálása az Azure Application Insights][azure-apim-ai], amely is Felfed, metrikák keresztül [Azure Monitor] [ azure-mon] figyeléshez .

## <a name="deployment"></a>Környezet

Első lépésként [Azure API Management szolgáltatáspéldány létrehozása a portálon.][apim-create]

Másik lehetőségként választhat egy meglévő Azure Resource Manager [gyorsindítási sablon] [ azure-quickstart-templates-apim] , amelyek igazodnak-e az adott használati esetekhez.

## <a name="pricing"></a>Díjszabás

Az API Management – fejlesztői, alapszintű, standard és prémium szintű négy szinten érhető el.  A különbség a szolgáltatásszintek, a részletes útmutatást talál a [díjszabási útmutatóját itt az Azure API Management.][apim-pricing]

Az API Management egységek hozzáadásával vagy eltávolításával méretezhető. Az egyes egységek kapacitása az adott díjcsomagtól függ.

> [!NOTE]
> A fejlesztői csomag az API Management-szolgáltatások értékelése is használható, de nem használható éles környezetben.

Előre jelzett költségek megtekintése és testreszabása az üzemelő példány van szüksége, módosíthatja a méretezési egységek számát, és az App Service-példányok a [Díjkalkulátor][pricing-calculator].

## <a name="related-resources"></a>Kapcsolódó erőforrások

Tekintse meg a széles körű Azure API Management [dokumentációja és referencia cikkeket.][apim]

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