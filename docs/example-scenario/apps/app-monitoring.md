---
title: Webalkalmazás-monitorozás az Azure-ban
description: Monitorozhatja az Azure App Service-szel üzemeltetett webalkalmazásokat.
author: adamboeglin
ms.date: 09/12/2018
ms.openlocfilehash: ea57ba50f4e9390d5527587752c3bebad01b6139
ms.sourcegitcommit: 42797fffb82bbbf86f6deb1da52c61d456be631e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 10/13/2018
ms.locfileid: "49313216"
---
# <a name="web-application-monitoring-on-azure"></a>Webalkalmazás-monitorozás az Azure-ban

Platform a platformszolgáltatás (PaaS) ajánlatok az Azure-ban kezelheti a számítási erőforrásokat, az Ön számára, és bizonyos értelemben módosítása hogyan, központi telepítésének figyelése. Az Azure több figyelési szolgáltatásokat, amelyek mindegyike hajt végre egy adott szerepkör tartalmaz. Ezek a szolgáltatások együtt, a gyűjtéséhez, elemzéséhez és az alkalmazások és az Azure-erőforrások használata származó telemetriai adatok alapján átfogó megoldást nyújthat.

Ebben a forgatókönyvben oldja meg a figyelési szolgáltatásokat használhatja, és a egy adatfolyam modell több adatforráshoz való használatra ismerteti. Esetén, a figyelés, a számos eszközöket és szolgáltatásokat az Azure-környezetek működik. Ebben a forgatókönyvben lehetőséget választjuk azonnal elérhető szolgáltatások pontosan, mert azok könnyen feldolgozható. A cikk későbbi részében más figyelési lehetőségek ismertetése.

## <a name="relevant-use-cases"></a>Alkalmazási helyzetek

Ebben a forgatókönyvben a következő használati esetek, vegye figyelembe:

- A telemetria figyelése webalkalmazás beállítása futási.
- Az Azure-ban üzembe helyezett alkalmazás előtér- és telemetria gyűjtése.
- Figyelési metrikák és az Azure-szolgáltatásokkal kapcsolatos kvóták.

## <a name="architecture"></a>Architektúra

![Alkalmazás figyelése architektúra ábrája][architecture]

Ebben a forgatókönyvben egy alkalmazás és az adatszint egy felügyelt Azure-környezet használja. A áramlanak keresztül az adatok a forgatókönyv a következő:

1. A felhasználó kommunikál az alkalmazást.
2. A böngésző és az app service telemetriát küldik.
3. Az Application Insights összegyűjti és elemzi az alkalmazás állapotával, teljesítményével és használati adatokat.
4. A fejlesztők és rendszergazdák állapotával, teljesítményével és használati adatokat tekintheti át.
5. Az Azure SQL Database telemetriai adatokat küldenek.
6. Az Azure Monitor gyűjti és elemzi az infrastruktúra-mérőszámok és kvóták.
7. A log Analytics gyűjti és elemzi a naplókat és mérőszámokat.
8. A fejlesztők és rendszergazdák állapotával, teljesítményével és használati adatokat tekintheti át.

### <a name="components"></a>Összetevők

- [Az Azure App Service](/azure/app-service/) egy PaaS szolgáltatás, fejlesztésére és üzemeltetésére alkalmazásokat a felügyelt virtuális gépeken. A mögöttes számítási infrastruktúra, amelyen az alkalmazások futtatása van kezelve. Az App Service erőforrás-használati kvóták és, alkalmazásmetrikák figyelését teszi lehetővé az naplózása diagnosztikai adatokat, és a riasztások a metrikák alapján. Még jobb létrehozásához használhatja az Application Insights [rendelkezésre állási tesztek] [ availability-tests] a különféle régiókból származó alkalmazás teszteléséhez.
- [Az Application Insights] [ application-insights] egy bővíthető alkalmazásteljesítmény-felügyeleti (APM) szolgáltatás a fejlesztők számára, és támogatja a több platformra. Az alkalmazás figyeli, észleli a teljesítményproblémákat és hibákat például alkalmazás rendellenességeket, és telemetriai adatokat küld az Azure Portalon. Az Application Insights használható naplózási, elosztott nyomkövetést és egyéni metrikákat is.
- [Az Azure Monitor] [ azure-monitor] alapszinten infrastruktúrát biztosít a [metrikák és naplók] [ metrics] a legtöbb szolgáltatás az Azure-ban. Többféle módon, beleértve a diagramkészítési őket az Azure Portalon, a hozzájuk férni a REST API-n keresztül vagy a lekérdezési őket a metrikák használhatja a PowerShell vagy parancssori felület használatával. Az Azure Monitor is kínál az adatok közvetlenül [A log Analytics és az egyéb szolgáltatások], ahol lekérdezheti és más helyszíni vagy felhőbeli forrásokból származó adatokat kombinálni.
- [Log Analytics] [ log-analytics] segítségével összehasonlíthatja a használati és teljesítményadatokat Application Insights által gyűjtött és az adatok között az Azure-erőforrások, amelyek támogatják az alkalmazást. Ebben a példában a [Azure Log Analytics-ügynököket] [ Azure Log Analytics agent] paranccsal küldje le az SQL Server-naplók a Log analyticsbe. Lekérdezések és adatok megtekintése az Azure portal Log Analytics panel írhat.

## <a name="considerations"></a>Megfontolandó szempontok

Ajánlott eljárás, hogy az Application Insights hozzáadása a kódhoz: fejlesztés a [Application Insights SDK-k][Application Insights SDKs], és testre szabhatja alkalmazásonként. A legtöbb alkalmazás-keretrendszerek ezek nyílt forráskódú SDK-k érhetők el. Bővítését és összegyűjtötte az adatokat, az SDK-Ink választékából mind a tesztelési és éles üzemelő példányok használatát beépítheti a fejlesztési folyamatot. A fő követelmény, az alkalmazás egy közvetlen vagy közvetett üzemel, a Applications Insights betöltési végpontjához az internetre irányuló címmel rendelkező üzemeltetett rendelkezik. Ezután telemetriát adhat hozzá, vagy egy meglévő eszköztelemetria-gyűjtést bővítését.

A futásidejű ellenőrzés egy másik egyszerű megoldást a kezdéshez. A gyűjtött telemetria konfigurációs fájlok keresztül kell szabályozni. Például hozzáadhatja a futtatókörnyezet módszereket, amelyek lehetővé teszik például eszközök [Application Insights Állapotfigyelőt] [ Application Insights Status Monitor] az SDK-k üzembe helyezését a helyes mappát, és adja hozzá a megfelelő konfigurációkat, a kezdéshez figyelés.

Például az Application Insightsba, a Log Analytics eszközt biztosít [adatelemzés egyszerre][analyzing data across sources], összetett lekérdezések létrehozásáról és [proaktív értesítéseket küldő] [ sending proactive alerts] által megadott feltételek alapján. A telemetriát is megtekintheti [az Azure Portalon][the Azure portal]. A log Analytics értékét hozzáadja a meglévő figyelési szolgáltatásokat, például az [Azure Monitor] [Azure Monitor], és a helyszíni környezetekben is figyelheti.

Az Application Insights és a Log Analytics használat [Azure Log Analytics lekérdezési nyelv][Azure Log Analytics Query Language]. Is [erőforrások közötti lekérdezések](https://azure.microsoft.com/blog/query-across-resources) egyetlen lekérdezést az Application Insights és a Log Analytics által összegyűjtött telemetriai adatok elemzéséhez.

Az Azure Monitor, az Application Insights és összes Küldés a Log Analytics [riasztások](/azure/monitoring-and-diagnostics/monitoring-overview-alerts). Például az Azure Monitor riasztások a CPU-kihasználtság, miközben az Application Insights-riasztások alkalmazásszintű mérőszámokat, például a kiszolgáló válaszideje a platform-szintű metrikákat. Az Azure Monitor riasztások az új események az Azure-tevékenységnapló, míg a Log Analytics kiadhatnak konfigurálta a használatára a szolgáltatások mérőszámok vagy esemény adatokkal kapcsolatos riasztások. [Az Azure monitorban riasztásokat egyesített](/azure/monitoring-and-diagnostics/monitoring-overview-unified-alerts) van egy új, egyesített riasztási tapasztalattal az Azure-ban, amely egy másik besorolást.

### <a name="alternatives"></a>Alternatív megoldások

Ez a cikk bemutatja a népszerű szolgáltatások kényelmesen elérhető figyelési lehetőségek, de számos választási lehetőség, többek között a saját naplózási mechanizmus létrehozásának lehetősége van. Ajánlott eljárás, hogy ki rétegek a megoldás elkészítése során adja hozzá a figyelési szolgáltatásokat. Az alábbiakban néhány lehetséges bővítmények és alternatívák:

- A Grafana használatával az Azure Monitor és az Application Insights metrikák összesítése a [Azure Monitor adatok forrása a Grafana][Azure Monitor Data Source For Grafana].
- [Adatok kutya] [ data-dog] funkcióit az Azure Monitor-összekötő
- Figyelési funkciók használatával automatizálhatja [Azure Automation][Azure Automation].
- Adja hozzá a kommunikáció [ITSM-megoldásba][ITSM solutions].
- Bővíteni a Log Analytics egy [felügyeleti megoldás][management solution].

### <a name="scalability-and-availability"></a>Méretezhetőség és rendelkezésre állás

Ebben a forgatókönyvben a PaaS-megoldások figyeli a nagy része, mert kényelmesen kezeléséhez rendelkezésre állását és méretezhetőségét, és élvezik szolgáltatásiszint-szerződések (SLA) összpontosít. Például App Services tartalmaz egy garantált [SLA] [ SLA] a rendelkezésre állás érdekében.

Az Application Insights rendelkezik [korlátok] [ app-insights-limits] a hány kérelmek másodpercenkénti feldolgozási. Ha a kérés korlátot túllépi, üzenet szabályozás tapasztalhat. Ennek megelőzése érdekében megvalósítása [szűrés] [ message-filtering] vagy [mintavételi] [ message-sampling] adatátviteli sebesség csökkentése érdekében

Magas rendelkezésre állási szempontok futtatja, az alkalmazás azonban olyan a fejlesztő feladata. Skála kapcsolatos információkért lásd a [méretezési szempontok](#scalability-considerations) az alapszintű webalkalmazás referenciaarchitektúrája szakaszát. Alkalmazás üzembe helyezését követően beállíthatja-tesztek [a rendelkezésre állás monitorozása] [ monitor its availability] Application Insights használatával.

### <a name="security"></a>Biztonság

Bizalmas információkat és megfelelőségi követelmények hatással az adatgyűjtés, megőrzés és -tárolás. Tudjon meg többet [Application Insights] [ application-insights] és [Log Analytics] [ log-analytics] telemetriai adatok kezeléséhez.

A következő biztonsági szempontokat is lehet alkalmazni:

- Személyes információk kezelésére, ha a fejlesztők a saját adataik gyűjtése, vagy meglévő telemetriai adatok kiegészítése tervének kialakításához.
- Fontolja meg az adatok megőrzésével. Például az Application Insights telemetriai adatait 90 napig megőrzi. Archivált adatok hosszabb ideig a Microsoft Power bi-ban, a folyamatos exportálás vagy a REST API segítségével szeretne hozzáférni. A Storage-díjak érvényesek.
- Azure-erőforrások adatokat, és kik tekinthetik meg egy adott alkalmazásból származó telemetriai adatok való hozzáférés szabályozásához való hozzáférés korlátozásához. A telemetria figyelése hozzáférés zárolása érdekében olvassa el [erőforrások, szerepkörök és hozzáférés-vezérlés az Application Insights][Resources, roles, and access control in Application Insights].
- Érdemes lehet az alkalmazás kódja meg, hogy a felhasználók hozzáadása, amely korlátozza az alkalmazás adatbetöltést verzió vagy címke jelölők olvasási/írási hozzáférést. Az Application Insights nincs nem szabályozza az egyes adatelemek után a rendszer elküldte őket az erőforrás, így ha egy felhasználó hozzáfér minden adat, az összes adat hozzáféréssel rendelkeznek az egyedi erőforrásokat.
- Adjon hozzá [cégirányítási](/azure/security/governance-in-azure) mechanizmusok szabályzat kényszerítése, vagy az Azure-erőforrások vezérlők költség, szükség esetén. Például, például a házirendek és a szerepköralapú hozzáférés-vezérlés biztonsági figyelés a Log Analytics használatához, vagy használjon [Azure Policy](/azure/azure-policy/azure-policy-introduction) hozhat létre, rendelhet hozzá, és kezelhet szabályzatdefiníciókat.
- Figyelheti a potenciális biztonsági problémákat, és a egy helyen jeleníti meg az Azure-erőforrások biztonsági állapotát, érdemes lehet [az Azure Security Center](/azure/security-center/security-center-intro).

## <a name="pricing"></a>Díjszabás

Költségek figyelése is gyorsan összeadódhatnak, ezért érdemes meghozni díjszabás, megismerheti, mit figyel, és ellenőrizze az egyes szolgáltatásokhoz kapcsolódó díjak. Az Azure Monitor biztosítja [alapmetrikák] [ basic metrics] költségek figyelése során ingyenesen [Application Insights] [ application-insights-pricing] és [ Log Analytics] [ log-analytics] betöltött adatok mennyiségét és a tesztek futtatása száma alapján.

Segít megismerkedni, használja a [díjkalkulátor] [ pricing] alapján. Ha szeretné látni, hogyan díjszabását szeretné módosítani az adott használati esetekhez, egyezik a várt üzembe helyezési a különböző beállításainak módosítására.

Hibakeresés közben és után az alkalmazás közzététele az Azure portal Application insights telemetria érkezik. Tesztelési célokra és a díjak elkerülése érdekében a korlátozott mennyiségű telemetriai van kialakítva. További mutatók hozzáadásához a telemetriai adatok korlátot is növelheti. Tekintse meg a részletesebb vezérléshez [Application Insights-mintavétel][Sampling in Application Insights].

Üzembe helyezés után megtekintheti a [élő metrikák Stream] [ Live Metrics Stream] teljesítménymutatók. Ezeket az adatokat nem tárolja---valós idejű metrikák---jelenik meg, de a telemetriát is gyűjtés és elemzés később. Nem jár az élő Stream adatokat.

A Log Analyticset a szolgáltatásba betöltött GB-onként számlázzuk. Az adatok az Azure Log Analytics szolgáltatásba betöltött minden hónap első 5 GB ingyenes érhető el, és a díjmentes első 31 nap Log Analytics-munkaterület az adatok megőrződnek. 

## <a name="next-steps"></a>További lépések

Tekintse meg ezeket az erőforrásokat, amelyek segítségével a saját figyelési megoldás használatának első lépései:

[Alapszintű webalkalmazás referenciaarchitektúrája][Basic web application reference architecture]

[Az ASP.NET-webalkalmazás monitorozásának indítása][Start monitoring your ASP.NET Web Application]

[Azure virtuális gépekről történő adatgyűjtést][Collect data about Azure Virtual Machines]

## <a name="related-resources"></a>Kapcsolódó források (lehet, hogy a cikkek angol nyelvűek)

[Az Azure-alkalmazások és erőforrások figyelése][Monitoring Azure applications and resources]

[Az Azure Application Insights futásidejű kivételek észlelése és diagnosztizálása][Find and diagnose run-time exceptions with Azure Application Insights]

<!-- links -->
[architecture]: ./media/architecture-app-monitoring.png
[availability-tests]: /azure/application-insights/app-insights-monitor-web-app-availability
[application-insights]: /azure/application-insights/app-insights-overview
[azure-monitor]: /azure/monitoring-and-diagnostics/monitoring-overview-azure-monitor
[metrics]: /azure/monitoring-and-diagnostics/monitoring-supported-metrics
[A log Analytics és az egyéb szolgáltatások]: /azure/log-analytics/log-analytics-azure-storage
[log-analytics]: /azure/log-analytics/log-analytics-overview
[Azure Log Analytics agent]: https://blogs.msdn.microsoft.com/sqlsecurity/2017/12/28/azure-log-analytics-oms-agent-now-collects-sql-server-audit-logs/
[application-insights-pricing]: https://azure.microsoft.com/pricing/details/application-insights/
[Application Insights SDKs]: /azure/application-insights/app-insights-asp-net
[Application Insights Status Monitor]: https://azure.microsoft.com/updates/application-insights-status-monitor-and-sdk-updated/
[analyzing data across sources]: /azure/log-analytics/log-analytics-dashboards
[sending proactive alerts]: /azure/log-analytics/log-analytics-alerts
[the Azure portal]: /azure/log-analytics/log-analytics-tutorial-dashboards
[Azure Log Analytics Query Language]: https://docs.loganalytics.io/docs/Learn
[cross-resource queries]: https://azure.microsoft.com/blog/query-across-resources/
[alerts]: /azure/monitoring-and-diagnostics/monitoring-overview-alerts
[Alerts (Preview)]: /azure/monitoring-and-diagnostics/monitoring-overview-unified-alerts
[Azure Monitor Data Source For Grafana]: https://grafana.com/plugins/grafana-azure-monitor-datasource
[Azure Automation]: /azure/automation/automation-intro
[ITSM solutions]: https://azure.microsoft.com/blog/itsm-connector-for-azure-is-now-generally-available/
[management solution]: /azure/monitoring/monitoring-solutions
[SLA]: https://azure.microsoft.com/support/legal/sla/app-service/v1_4/
[monitor its availability]: /azure/application-insights/app-insights-monitor-web-app-availability
[Resources, roles, and access control in Application Insights]: /azure/application-insights/app-insights-resources-roles-access-control
[basic metrics]: /azure/monitoring-and-diagnostics/monitoring-supported-metrics
[pricing]: https://azure.microsoft.com/pricing/calculator/#log-analyticsc126d8c1-ec9c-4e5b-9b51-4db95d06a9b1
[Sampling in Application Insights]: /azure/application-insights/app-insights-sampling
[Live Metrics Stream]: /azure/application-insights/app-insights-live-stream
[Basic web application reference architecture]: /azure/architecture/reference-architectures/app-service-web-app/basic-web-app#scalability-considerations
[Start monitoring your ASP.NET Web Application]: /azure/application-insights/quick-monitor-portal
[Collect data about Azure Virtual Machines]: /azure/log-analytics/log-analytics-quick-collect-azurevm
[Monitoring Azure applications and resources]: /azure/monitoring-and-diagnostics/monitoring-overview
[Find and diagnose run-time exceptions with Azure Application Insights]: /azure/application-insights/app-insights-tutorial-runtime-exceptions
[data-dog]: https://www.datadoghq.com/blog/azure-monitoring-enhancements/
[app-insights-limits]: /azure/azure-subscription-service-limits#application-insights-limits
[message-filtering]: /azure/application-insights/app-insights-api-filtering-sampling
[message-sampling]: /azure/application-insights/app-insights-sampling
