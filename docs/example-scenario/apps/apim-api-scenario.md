---
title: Régi webalkalmazás migrálása egy API-alapú architektúrába az Azure-ban
description: Modernizálhatja régi webalkalmazásait az Azure API Management használatával.
author: begim
ms.date: 09/13/2018
ms.openlocfilehash: 1aa7ea6dc895146e13677dd9867fb2530f0a8f04
ms.sourcegitcommit: 62945777e519d650159f0f963a2489b6bb6ce094
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 10/09/2018
ms.locfileid: "48876789"
---
# <a name="migrating-a-legacy-web-application-to-an-api-based-architecture-on-azure"></a>Régi webalkalmazás migrálása egy API-alapú architektúrába az Azure-ban

Egy e-kereskedelmi cég, az utazási iparág modernizálhatja van a régi böngésző alapú szoftververmeket. Míg a meglévő stack csak a monolitikus, néhány [SOAP-alapú HTTP-szolgáltatások] [ soap] létezik a legutóbbi projektből. Néhány belső szellemi tulajdon, amely már fejlesztik a bevételi lehetőségekről további bevételi létrehozásának fontolgatják.

A projekt célok közé tartozik a címzési műszaki adósságot, folyamatos karbantartást javítása és felgyorsítja a kevesebb regresszió hibák funkcióinak fejlesztését. A projekt elkerülése érdekében a kockázat, az egyes lépésekhez párhuzamosan iteratív folyamat használja:

* A fejlesztői csapat fogja korszerűsítheti a alkalmazás háttérben álló virtuális gépeken üzemeltetett relációs adatbázis.
* A belső fejlesztésű fejlesztőcsapat fog kiírni, új üzleti funkció, amely új HTTP API-k keresztül lesz közzétéve.
* A szerződés fejlesztői csapat új böngészőalapú felhasználói Felületet, amely az Azure-ban üzemeltetett fog létrehozni.

Új alkalmazás szolgáltatásai fázisában kézbesítése történik. Csatlakoznak az *fokozatosan cserélje le* a meglévő ügyfél-kiszolgáló böngészőalapú felhasználói felületi funkciók (üzemeltethető a helyszínen) e-kereskedelmi cég ma használja, amelyen a.

A felügyeleti csoport nem szeretné feleslegesen korszerűsítéséhez. Szeretnének is, hogy a hatókör és a költségek kézben. Ehhez akkor döntött, megőrizheti a meglévő SOAP HTTP-szolgáltatások. Ezek kíván minimalizálása érdekében a módosításokat a meglévő felhasználói felületre. [Az Azure API Management (APIM)] [ apim] alkalmazhatja a projekt követelményeket és korlátokat számos megoldása érdekében.

## <a name="architecture"></a>Architektúra

![Architektúradiagram][architecture]

Az új felhasználói felület platform (PaaS) alkalmazás az Azure-ban üzemeltetett, és a meglévő és az új HTTP API-k függ. Ezen API-k elküldjük Önnek jobban kialakított kapcsolatok engedélyezése a jobb teljesítmény érdekében, egyszerűbb integráció és bővíthetőség jövőbeli vannak beállítva.

### <a name="components-and-security"></a>Összetevők és biztonság

1. A meglévő helyszíni webes alkalmazás továbbra is közvetlenül a a meglévő helyszíni webszolgáltatások felhasználása.
2. A meglévő HTTP-szolgáltatások és a meglévő webes alkalmazás közötti hívások változatlan marad. Hívásokat a rendszer belső, a vállalati hálózathoz.
3. A bejövő hívások a meglévő belső szolgáltatások létrejönnek az Azure-ból:
    * A biztonsági csapat, lehetővé teszi az APIM-példányra átadni a vállalati tűzfalon keresztül a meglévő helyszíni szolgáltatásokhoz érkező forgalom [biztonságos átvitel (HTTPS és SSL) használatával][apim-ssl].
    * Az üzemeltetési csapat lehetővé teszi a szolgáltatások csak az APIM-példányra a bejövő hívások. Ennek a követelménynek úgy [engedélyezési-lista az IP-címet az APIM-példány] [ apim-whitelist-ip] a vállalati hálózat határain belül.
    * Új modul a helyszíni HTTP szolgáltatások kérelem folyamatba van konfigurálva (számára, hogy **csak** e kívülről származó kapcsolatok), amely ellenőrzi [egy tanúsítványt, amely APIM biztosít] [apim-mutualcert-auth].
1. Az új API-val:
    * Van végzetesnek csak az APIM-példányra, amely az API-homlokzat biztosít. Az új API-t közvetlenül nem érhetők el.
    * Fejleszti és közzétett egy [Azure PaaS webes API-alkalmazás][azure-api-apps].
    * Az engedélyezési listán (keresztül [webalkalmazás beállítások][azure-appservice-ip-restrict]) csak fogadására a [APIM VIP][apim-faq-vip].
    * Üzemel az Azure Web Apps a biztonságos átvitel/SSL engedélyezve van.
    * Van engedélyezve, engedélyezési [az Azure App Service által biztosított] [ azure-appservice-auth] Azure Active Directory és az OAuth2 használatával.
2. Az új böngésző-alapú webes alkalmazás függ az Azure API Management-példány **mindkét** a meglévő HTTP API-t és az új API-t.

Az örökölt HTTP-szolgáltatások leképezése egy új API-szerződés az APIM-példányra lesz konfigurálva. Ezzel az új webes felhasználói Felületet nem észleli, az integráció az örökölt services és API-k és az új API-k készlete. A jövőben a projektcsapat fog fokozatosan port az új API-funkciókat és kivonása az eredeti szolgáltatásokat. Ezek a változások kezelésének belül APIM-konfiguráció, így nem érinti az előtér felhasználói felületi és átalakítását munkahelyi elkerülésére.

### <a name="alternatives"></a>Alternatív megoldások

* Ha a szervezet infrastruktúrára áthelyezése teljes egészében az Azure-ba, beleértve az örökölt alkalmazásokat futtató virtuális géphez lett tervezési majd APIM továbbra is lenne nagyszerű lehetőséget óta bármely címezhető HTTP-végpont az előtérrendszer működhet.
* Ha az ügyfél döntött, hogy tartani a meglévő végpontokat, és nem tehető közzé őket nyilvánosan, az API Management-példány tudta csatolni egy [Azure Virtual Network (VNet)][azure-vnet]:
  * Az egy [Azure átemeléses forgatókönyv] [ azure-vm-lift-shift] csatolva az üzembe helyezett Azure Virtual Network, az ügyfél sikerült közvetlenül érjék el a háttér-szolgáltatás a privát IP-címeken keresztül.
  * Helyszíni esetben az API Management-példány tudta elérni a belső szolgáltatás közvetlenül a Microsoftnak keresztül vissza a egy [Azure VPN gateway és a helyek közötti IPSec VPN-kapcsolat] [ azure-vpn] vagy [ Az ExpressRoute] [ azure-er] ami egy [hibrid Azure és helyszíni megoldásként][azure-hybrid].
* Az API Management-példány képes tartani privát belső módban az API Management-példány üzembe helyezésével. Az üzembe helyezés majd is használható egy [Azure Application Gateway] [ azure-appgw] nyilvános hozzáférésének engedélyezése az egyes API-k, belső, mások továbbra is. További információkért lásd: [APIM csatlakoztatása virtuális hálózathoz belső módban][apim-vnet-internal].

> [!NOTE]
> Általános információk az API Management egy virtuális hálózathoz való csatlakozás [Itt][apim-vnet].

### <a name="availability-and-scalability"></a>Rendelkezésre állás és méretezhetőség

* Az Azure API Management lehet [horizontálisan felskálázott] [ apim-scaleout] válassza ki a tarifacsomagot, majd egységek.
* Skálázás is megtörténhet, [automatikusan az automatikus skálázás][apim-autoscale].
* [Üzembe helyezése több régióban] [ apim-multi-regions] lehetővé válik a feladatátvétel beállításai és végezhető, a [prémium szintű][apim-pricing].
* Érdemes lehet [integrálása az Azure Application Insights][azure-apim-ai], amely is Felfed, metrikák keresztül [Azure Monitor] [ azure-mon] figyeléshez .

## <a name="deployment"></a>Környezet

Első lépésként [Azure API Management szolgáltatáspéldány létrehozása a portálon.][apim-create]

Másik lehetőségként választhat egy meglévő Azure Resource Manager [gyorsindítási sablon] [ azure-quickstart-templates-apim] , amelyek igazodnak-e az adott használati esetekhez.

## <a name="pricing"></a>Díjszabás

Az API Management négy szinten érhető el: fejlesztői, alapszintű, standard és prémium. A különbség a szolgáltatásszintek, a részletes útmutatást talál a [díjszabási útmutatóját itt az Azure API Management.][apim-pricing]

Az API Management egységek hozzáadásával vagy eltávolításával méretezhető. Az egyes egységek kapacitása az adott díjcsomagtól függ.

> [!NOTE]
> A fejlesztői csomag az API Management-szolgáltatások értékelése is használható. A fejlesztői csomag nem használható éles üzemi környezetek részei.

Előre jelzett költségek megtekintése és testreszabása az üzemelő példány van szüksége, módosíthatja a méretezési egységek számát, és az App Service-példányok a [Azure Díjkalkulátor][pricing-calculator].

## <a name="related-resources"></a>Kapcsolódó források (lehet, hogy a cikkek angol nyelvűek)

Tekintse meg a széles körű Azure API Management [dokumentációja és referencia cikkeket.][apim]

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
