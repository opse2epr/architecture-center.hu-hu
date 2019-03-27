---
title: Az Azure alapszintű vállalati integráció
titleSuffix: Azure Reference Architectures
description: Javasolt architektúra megvalósítása az Azure Logic Apps és az Azure API Management egyszerű vállalati integráció mintát.
services: logic-apps
author: mattfarm
ms.reviewer: jonfan, estfan, LADocs
ms.topic: reference-architecture
ms.date: 12/03/2018
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: integration-services
ms.openlocfilehash: 76e422ead7e53c582a9d64ab1da643c3990749d6
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298713"
---
# <a name="basic-enterprise-integration-on-azure"></a>Alapszintű vállalati integráció az Azure-ban

Ez a referenciaarchitektúra használ [Azure integrációs szolgáltatások] [ integration-services] vállalati háttérrendszerre hívásainak vezénylésére. A háttérrendszerekhez szoftver feltétlenül tartalmazzák a szoftverszolgáltatások (SaaS) rendszerek, az Azure-szolgáltatásokkal, mint, és a vállalat meglévő webszolgáltatások.

Az Azure integrációs szolgáltatások gyűjteménye, alkalmazások és adatok integrálása-szolgáltatások. Ez az architektúra két ezeket a szolgáltatásokat használja: [A Logic Apps] [ logic-apps] előkészíthető a munkafolyamatokat, és [az API Management] [ apim] katalógusok az API-k létrehozásához. Ez az architektúra olyan alapszintű integrációs forgatókönyvek, ahol a sématelepítési munkafolyamatot a Háttérszolgáltatásokhoz szinkron hívások elegendő. A kifinomultabb architektúra használatával [várólisták és események](./queues-events.md) Ez a alapvető architektúra épül.

![Architekturális diagramja – egyszerű vállalati integráció](./_images/simple-enterprise-integration.png)

## <a name="architecture"></a>Architektúra

Az architektúra a következő összetevőkből áll:

- **A háttérrendszerek**. A diagram jobb oldalán látható a különböző háttérrendszerekhez, hogy a vállalat helyezte üzembe, vagy támaszkodik. Ezek lehetnek olyan SaaS-rendszerekhez, más Azure-szolgáltatásokhoz, vagy a webszolgáltatásokat REST vagy a SOAP végpontokat tesznek közzé.

- **Az Azure Logic Apps**. [A Logic Apps] [ logic-apps] az alkalmazásokat, adatokat és szolgáltatásokat integráló vállalati munkafolyamatokat hoz létre kiszolgáló nélküli platform. Ebben az architektúrában a logic Apps alkalmazások a HTTP-kérések által aktivált. Összetettebb vezénylési munkafolyamatokat is beágyazhatja. Használja a Logic Apps [összekötők] [ logic-apps-connectors] való integrációhoz gyakran használt szolgáltatásokat. A Logic Apps-összekötők több száz kínál, és létrehozhat egyéni összekötőket.

- **Az Azure API Management**. [Az API Management] [ apim] közzétételéhez katalógusok HTTP API-k, előléptetni újbóli és a egy felügyelt szolgáltatás. Az API Management kapcsolódó két összetevőből áll:

  - **API-átjáró**. Az API-átjáró fogadja el a HTTP-hívások, és továbbítja a háttérben.

  - **Fejlesztői portál**. Az Azure API Management minden példányának hozzáférést biztosít egy [fejlesztői portál][apim-dev-portal]. Ezen a portálon hozzáférést biztosít a fejlesztők dokumentáció és Kódminták az API-k meghívására szolgáló. API-k a fejlesztői portálon is tesztelheti.

  Ebben az architektúrában összetett API-k által épített [importálása logic apps] [ apim-logic-app] API-k formájában. A meglévő webszolgáltatásokat is importálhat [OpenAPI importálása] [ apim-openapi] (Swagger) specifikációk vagy [SOAP API importálása] [ apim-soap] WSDL-fájlból előírásoknak.

  Az API-átjáró segítségével különítse el a háttér-előtér-ügyfélről. Például azt is újraírási URL-címek, vagy átalakíthatja kérelmek, mielőtt elérnék a háttérrendszer. Például a hitelesítést, az eltérő eredetű erőforrások megosztása (CORS) támogatásával és a válaszok gyorsítótárazását számos általános megfontolások is kezeli.

- **Azure DNS**. [Az Azure DNS] [ dns] DNS-tartományok egy üzemeltetési szolgáltatás. Az Azure DNS névfeloldás biztosít a Microsoft Azure-infrastruktúra használatával. A tartományok Azure-ban üzemelteti, azonos hitelesítő adatokkal, API-kkal, eszközökkel és számlázási használata más Azure-szolgáltatások DNS-rekordok is kezelheti. Egyéni tartománynév, amilyen a contoso.com használatához hozzon létre az egyéni tartománynév leképezése az IP-cím DNS-rekordjait. További információkért lásd: [egyéni tartománynév beállítása az API Management][apim-domain].

- **Azure Active Directory (Azure AD)**. Használat [Azure ad-ben] [ aad] hívja az API-átjáró azon ügyfelek hitelesítéséhez. Azure ad-ben az OpenID Connect (OIDC) protokoll használatát támogatja. Az ügyfelek hozzáférési jogkivonat beszerzése az Azure ad-ből, és API-átjáró [érvényesíti a jogkivonatot] [ apim-jwt] a kérelem engedélyezéséhez. A Standard vagy prémium szintű API Management használata esetén az Azure AD is biztosíthatja a fejlesztői portálra való hozzáférést.

## <a name="recommendations"></a>Javaslatok

Az adott követelményei eltérhetnek az itt látható az általános architektúrát. A jelen szakaszban leírt javaslatokat tekintse kiindulópontnak.

### <a name="api-management"></a>API Management

Használja az API Management alapszintű, Standard vagy prémium szintű csomag esetében. Ezek a rétegek egy éles szolgáltatásiszint-szerződés (SLA) kínál, és támogatja a horizontális felskálázás az Azure-régión belül. Az API Management átviteli kapacitás mérése *egységek*. Mindegyik tarifacsomag maximális kibővített rendelkezik. A prémium szint is támogatja a horizontális felskálázás több Azure-régiók között. Válassza ki a réteg alapján a szolgáltatás beállítása és az átviteli sebességgel. További információkért lásd: [az API Management díjszabása] [ apim-pricing] és [az Azure API Management-példány kapacitása][apim-capacity].

Az Azure API Management minden példányához tartozik egy alapértelmezett tartománynevet, amely altartománya, `azure-api.net` & használva, például `contoso.azure-api.net`. Célszerű beállítani egy [egyéni tartomány] [ apim-domain] a szervezet számára.

### <a name="logic-apps"></a>Logic Apps

A Logic Apps a legalkalmasabb forgatókönyvek, amelyek nem igényelnek közel valós idejű választ, mint például az aszinkron vagy félig hosszú ideig futó API-hívások. Ha kis késleltetésre szükség, például az a hívás, amely letiltja a felhasználói felületet, használja egy másik technológia. Például használja az Azure Functions vagy a webes API-k üzembe helyezését az Azure App Service. Az API Management segítségével előtér-az API-t az API-fogyasztókat.

### <a name="region"></a>Régió

Hálózati késés minimalizálása érdekében helyezze el az API Management és a Logic Apps ugyanabban a régióban. Általánosságban elmondható válassza ki a régiót, amelyben a felhasználókhoz legközelebb eső (vagy a háttérszolgáltatások legközelebbi).

Az erőforráscsoport szintén van régiója. Ebben a régióban megadja a központi telepítési metaadatok tárolására és hol hajtsa végre a központi telepítési sablont. Üzembe helyezés során javítható a rendelkezésre állás, helyezze az erőforráscsoport és erőforrások ugyanabban a régióban.

## <a name="scalability-considerations"></a>Méretezési szempontok

Adja hozzá az API Management a méretezhetőség növeléséhez [gyorsítótárazási házirendek] [ apim-caching] ahol lehetséges. Gyorsítótárazás is segít csökkenteni a háttér-szolgáltatások.

A nagyobb kapacitást kínálnak, horizontális felskálázása az Azure API Management alapszintű, Standard és prémium szinten egy Azure-régióban. A szolgáltatás használata elemezhető a a **metrikák** menüben válassza a **kapacitás metrika** lehetőséget, és majd vertikális felskálázás vagy leskálázás megfelelő módon. A frissítés vagy a méretezési csoport folyamat is igénybe vehet a 15-45 percre a alkalmazni.

Az API Management-szolgáltatás méretezése javaslatok:

- Vegye figyelembe a forgalmi mintázatait, amikor a méretezést. Több ideiglenes forgalmat az ügyfelek nagyobb kapacitásra van szüksége.

- Összhangban kapacitásértéket, amely nagyobb, mint 66 % vertikális kell utalhat.

- Összhangban kapacitásértéket, amely a 20 %-os utalhat vertikális leskálázás lehetőséget.

- Mielőtt engedélyezné a terhelés, éles környezetben, mindig terhelési tesztek és egy tipikus terhelés mellett az API Management szolgáltatás.

A prémium szintű méretezhető API Management-példány több Azure-régiók között. Ez lehetővé teszi az API Management jogosult magasabb SLA, és lehetővé teszi több régióban a felhasználók közel szolgáltatások kiépítése.

A Logic Apps, kiszolgáló nélküli modell azt jelenti, hogy a rendszergazdák nem kell a szolgáltatás méretezhetősége tervezése. A szolgáltatás automatikusan méretezi magát, hogy megfeleljenek az igényeknek.

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

Tekintse át az SLA-t az egyes szolgáltatásokhoz:

- [Az API Management szolgáltatásiszint-szerződés][apim-sla]
- [A Logic Apps szolgáltatásiszint-szerződés][logic-apps-sla]

Az API Management a prémium szint két vagy több régióban üzembe helyezheti, ha, magasabb SLA jogosult. Lásd: [az API Management díjszabása][apim-pricing].

### <a name="backups"></a>Biztonsági másolatok

Rendszeresen [biztonsági mentése] [ apim-backup] az API Management-konfigurációt. A biztonsági másolat Store egy helyet, vagy az Azure-régióban, amely eltér a régió, ahol a szolgáltatás üzembe helyezése. Alapján a [RTO][rto], vészhelyreállítási stratégia kiválasztása:

- A vész-helyreállítási esemény egy új API Management-példány üzembe helyezése, az új példányt a biztonsági másolat visszaállítása és szolgáltatással átirányíthassa a DNS-rekordokat.

- Az API Management szolgáltatás passzív példánya ne egy másik Azure-régióban. Rendszeresen állítsa vissza a biztonsági mentések példányhoz, az aktív szolgáltatás szinkronban tartja. Egy vész-helyreállítási esemény során a szolgáltatás helyreállításához meg kell csak szolgáltatással átirányíthassa a DNS-rekordokat. Ez a megközelítés további költséget áll, mert után kell fizetni, a passzív példányra, de csökkenti a helyreállítási ideje.

A logic apps esetében javasoljuk, hogy a konfiguráció a kódot, megközelítést, biztonsági mentése és visszaállítása. Mivel a logic apps kiszolgáló nélküli, gyorsan létrehozhatja őket az Azure Resource Manager-sablonok. Mentés a sablonok verziókövetési rendszerben, a sablonok integrálása a folyamatos integráció/folyamatos készregyártás (CI/CD) folyamatot. A vész-helyreállítási esemény helyezheti üzembe a sablont egy új régióban.

Egy másik régióba telepít egy logikai alkalmazást, frissítse a konfigurációt, az API Management szolgáltatásban. Az API-k frissítheti **háttérrendszer** tulajdonság egy egyszerű PowerShell-parancsprogram használatával.

## <a name="manageability-considerations"></a>Felügyeleti szempontok

Hozzon létre külön erőforráscsoportok éles környezetben, fejlesztési, és tesztelési környezetek. Külön erőforráscsoportok megkönnyítik a központi telepítések felügyeletéhez szükséges, a tesztkörnyezetek törlését és a hozzáférési jogosultságok hozzárendelése.

Amikor erőforrásokat rendel erőforráscsoportok, vegye figyelembe a következőket:

- **Életciklus**. Általánosságban elmondható helyezze ugyanabba az erőforráscsoportba az azonos életciklussal rendelkező erőforrások.

- **Hozzáférés**. A csoportokban található erőforrások hozzáférési házirendeket alkalmazza, használhatja [szerepköralapú hozzáférés-vezérlés] [ rbac] (RBAC).

- **Számlázási**. Megtekintheti az erőforráscsoport költségeinek összesítése.

- **Az API Management tarifacsomag**. A fejlesztői csomag használata a fejlesztési-tesztelési környezetet. A költségek csökkentésére üzem előtti tesztelés során, egy replikát, az éles környezet üzembe helyezése, a tesztek futtatásához, és majd állítsa le.

### <a name="deployment"></a>Környezet

Használat [Azure Resource Manager-sablonok] [ arm] az Azure-erőforrások üzembe helyezéséhez. Sablonok megkönnyítik a PowerShell vagy az Azure CLI használatával üzembe helyezések automatizálását.

Helyezze el az API Management és minden egyes logikai alkalmazás a saját külön Resource Manager-sablonok. Külön sablonok használatával tárolhatja verziókövetési rendszerekben az erőforrásokat. Telepítheti a sablonok együtt vagy külön-külön a CI/CD-folyamat részeként.

### <a name="versions"></a>Verziók

Minden alkalommal, amikor a logikai alkalmazás konfigurációjának módosítása, vagy a frissítést egy Resource Manager-sablon használatával helyezhet üzembe az Azure másolatot készít azokról azt a verziót, és tartja a futtatási előzmények rendelkező összes verzió. Ezen verziói segítségével nyomon követheti a korábbi módosítások vagy aktuális konfigurációját a logikai alkalmazás egy verziójának támogatása. Ha például állíthat vissza egy logikai alkalmazást egy korábbi verzióját.

Az API Management két különálló, de egymást kiegészítő versioning fogalmak támogatja:

- *Verziók* engedélyezi, választhat egy API-verzió alapján saját igényeinek megfelelő, például, v1, v2, beta vagy éles üzemi API-fogyasztókat.

- *Változatok* API-rendszergazdák nem kompatibilitástörő változások API-ban, és telepítheti majd a változásokat az API-fogyasztókat tájékoztatni a módosításokat a módosítási napló engedélyezése.

Győződjön meg arról, egy változat fejlesztői környezetben, és ezt a módosítást, más környezetekben üzembe helyezése a Resource Manager-sablonok használatával. További információkért lásd: [az API több verziójának közzététele][apim-versions]

Változatok használatával az API tesztelése az aktuális és a felhasználók módosítása előtt. Terheléses tesztelés vagy integrációs tesztelés azonban ez a módszer nem ajánlott. Használjon inkább külön vizsgálat vagy üzem előtti környezetben.

## <a name="diagnostics-and-monitoring"></a>Diagnosztika és figyelés

Használat [Azure Monitor] [ monitor] az API Management és a Logic Apps a műveletek figyelése. Az Azure Monitor nyújt információkat a metrikák minden szolgáltatáshoz beállítva az alapján, és alapértelmezés szerint engedélyezve van. További információkért lásd:

- [A közzétett API-k monitorozása][apim-monitor]
- [Állapot figyelése, állítsa be a diagnosztikai naplózás, és kapcsolja be a riasztásokat az Azure Logic Apps][logic-apps-monitor]

Minden szolgáltatás van még ezeket a beállításokat:

- Mélyebb elemzésre és dashboarding küldése a Logic Apps naplók [Azure Log Analytics][logic-apps-log-analytics].

- Fejlesztési és üzemeltetési figyelés az Azure Application Insights beállítása az API Management.

- Az API Management támogatja a [Power BI megoldássablon az egyéni API-analytics][apim-pbi]. Ez a megoldássablon használhatja a saját elemzési megoldás létrehozásához. Az üzleti felhasználók a Power BI jelentések elérhetővé teszi.

## <a name="security-considerations"></a>Biztonsági szempontok

Bár ez a lista nem teljes mértékben ismerteti az összes ajánlott biztonsági eljárások, az alábbiakban néhány biztonsági szempontot, amely kifejezetten ebben az architektúrában a alkalmazni:

- Az Azure API Management szolgáltatást egy rögzített nyilvános IP-címmel rendelkezik. Korlátozza a hozzáférést az API Management csak az IP-címét, a Logic Apps-végpontok hívása. További információkért lásd: [bejövő IP-címek korlátozása][logic-apps-restrict-ip].

- Ahhoz, hogy a felhasználók rendelkeznek a megfelelő hozzáférési szintekkel, a szerepköralapú hozzáférés-vezérlés (RBAC) használja.

- Biztonságos nyilvános, az API Management API-végpontokat, OAuth vagy OpenID Connect használatával. A biztonságos nyilvános API-végpontokat, egy identitásszolgáltató konfigurálása, és adjon hozzá egy JSON webes jogkivonat (JWT) érvényesség-ellenőrzési szabályzat. További információkért lásd: [OAuth 2.0 segítségével az Azure Active Directory és az API Management API-k védelme][apim-oauth].

- Csatlakoztassa a háttérszolgáltatások az API Management kölcsönös tanúsítványok használatával.

- Az API Management API-k a HTTPS kényszerítése.

### <a name="storing-secrets"></a>Titkos kódok tárolása

Soha ne tároljon jelszavakat, hozzáférési kulcsokat és kapcsolati sztringekat a forrásvezérlőben. Ha ezekre az értékekre szükség, biztonságos, majd központilag telepítenie ezeket az értékeket a megfelelő módszerek.

Ha egy logikai alkalmazást, amely nem hozható létre összekötő belül bizalmas értéket igényel, tárolja ezeket az értékeket az Azure Key Vaultban, és hivatkozni tudjon rájuk a Resource Manager-sablonból. Használja a központi telepítési sablon paramétereit és paraméterfájlt az egyes környezetekhez. További információkért lásd: [biztonságos paraméterek és a egy munkafolyamaton belül bemenetek][logic-apps-secure].

Az API Management objektumokban, úgynevezett használatával kezeli a titkos kulcsok *értékek nevű* vagy *tulajdonságok*. Ezek az objektumok tárolja az értékeket, amelyeket az API Management házirendek keresztül érheti el biztonságosan. További információkért lásd: [nevű értékek használata az Azure API Management házirendek][apim-properties].

## <a name="cost-considerations"></a>Költségekkel kapcsolatos szempontok

Ha még futnak az összes API Management-példány díja. Ha vertikálisan rendelkezik, és nincs szükség ilyen szintű teljesítmény az idő, manuálisan vertikális leskálázás, vagy konfigurálja [az automatikus skálázás][apim-autoscale].

Használja a Logic Apps egy [kiszolgáló nélküli](/azure/logic-apps/logic-apps-serverless-overview) modell. A számlázás a műveleti és összekötőhöz végrehajtási alapján lesz kiszámítva. További információkért lásd: [Logic Apps díjszabási](https://azure.microsoft.com/pricing/details/logic-apps/). Jelenleg nincs a Logic Apps szint szempontjai.

## <a name="next-steps"></a>További lépések

A nagyobb megbízhatóság és skálázhatóság használatával üzenetsorokat és eseményeket a háttérrendszer rendszerek szétválaszthatók. Ez a minta-sorozat következő referencia-architektúra látható: [Vállalati integráció, az üzenetsorok és események](./queues-events.md).

<!-- links -->

[aad]: /azure/active-directory
[apim]: /azure/api-management
[apim-autoscale]: /azure/api-management/api-management-howto-autoscale
[apim-backup]: /azure/api-management/api-management-howto-disaster-recovery-backup-restore
[apim-caching]: /azure/api-management/api-management-howto-cache
[apim-capacity]: /azure/api-management/api-management-capacity
[apim-dev-portal]: /azure/api-management/api-management-key-concepts#a-namedeveloper-portal-a-developer-portal
[apim-domain]: /azure/api-management/configure-custom-domain
[apim-jwt]: /azure/api-management/policies/authorize-request-based-on-jwt-claims
[apim-logic-app]: /azure/api-management/import-logic-app-as-api
[apim-monitor]: /azure/api-management/api-management-howto-use-azure-monitor
[apim-oauth]: /azure/api-management/api-management-howto-protect-backend-with-aad
[apim-openapi]: /azure/api-management/import-api-from-oas
[apim-pbi]: https://aka.ms/apimpbi
[apim-pricing]: https://azure.microsoft.com/pricing/details/api-management/
[apim-properties]: /azure/api-management/api-management-howto-properties
[apim-sla]: https://azure.microsoft.com/support/legal/sla/api-management/
[apim-soap]: /azure/api-management/import-soap-api
[apim-versions]: /azure/api-management/api-management-get-started-publish-versions
[arm]: /azure/azure-resource-manager/resource-group-authoring-templates
[dns]: /azure/dns/
[integration-services]: https://azure.microsoft.com/product-categories/integration/
[logic-apps]: /azure/logic-apps/logic-apps-overview
[logic-apps-connectors]: /azure/connectors/apis-list
[logic-apps-log-analytics]: /azure/logic-apps/logic-apps-monitor-your-logic-apps-oms
[logic-apps-monitor]: /azure/logic-apps/logic-apps-monitor-your-logic-apps
[logic-apps-restrict-ip]: /azure/logic-apps/logic-apps-securing-a-logic-app#restrict-incoming-ip-addresses
[logic-apps-secure]: /azure/logic-apps/logic-apps-securing-a-logic-app#secure-parameters-and-inputs-within-a-workflow
[logic-apps-sla]: https://azure.microsoft.com/support/legal/sla/logic-apps
[monitor]: /azure/azure-monitor/overview
[rbac]: /azure/role-based-access-control/overview
[rto]: ../../resiliency/index.md#rto-and-rpo