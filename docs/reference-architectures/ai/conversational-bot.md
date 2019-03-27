---
title: Egy nagyvállalati szintű természetes nyelvi robotot hozhat létre
description: Hogyan hozhat létre egy nagyvállalati szintű természetes nyelvi bot (csevegőrobot) az Azure Bot Framework használatával.
author: roalexan
ms.date: 01/24/2019
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: azcat-ai
ms.openlocfilehash: 0f5de0eca6fbd35cca1a0e8443f363df09ffc6aa
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299527"
---
# <a name="enterprise-grade-conversational-bot"></a>Nagyvállalati szintű beszélgetőrobot

Ez a referenciaarchitektúra bemutatja, hogyan hozhat létre egy nagyvállalati szintű természetes nyelvi bot (csevegőrobot) használatával a [Azure Bot Framework][bot-framework]. Minden egyes bot különböző, de néhány gyakori minták, munkafolyamatok és technológiák érdemes figyelembe vennie. A robot kiszolgálására vállalati számítási feladatokat, különösen a nincsenek az alapfunkciókra túl sok kialakítási szempontokat. Ez a cikk a leginkább fontos tervezési szempontokat ismerteti, és a egy robusztus, biztonságos és aktívan tanulási bot létrehozásához szükséges eszközöket vezet be.

[![Az architektúra ábrája][0]][0]

A bevált gyakorlat segédprogram minták ebben az architektúrában használt rendszer teljes mértékben nyílt forráskódú és elérhető a [GitHub][git-repo-base]. 

## <a name="architecture"></a>Architektúra

Itt látható architektúrában a következő Azure-szolgáltatásokat használ. Saját bot előfordulhat, hogy nem használja ezen szolgáltatások mindegyikéhez, vagy további szolgáltatásokat is be.

### <a name="bot-logic-and-user-experience"></a>A robot logikai és a felhasználói élmény

- **[Bot Framework szolgáltatás] [ bot-framework-service]**  (BFS). Ez a szolgáltatás a robot csatlakozik egy kommunikációs alkalmazás, például a Cortana, a Facebook Messenger és Slack. Ez megkönnyíti a robot és a felhasználó közötti kommunikációt.
- **[Az Azure App Service][app-service]**. A robot alkalmazáslogika üzemel az Azure App Service-ben.

### <a name="bot-cognition-and-intelligence"></a>A robot észlelés és hírszerzés

- **[Language Understanding][luis]** (LUIS). Része [Azure Cognitive Services][cognitive-services], LUIS lehetővé teszi, hogy a robot természetes nyelvi felhasználói szándékok és entitások azonosítása megismeréséhez.
- **[Az Azure Search][search]**. A művelet egy felügyelt szolgáltatás, amely egy gyors kereshető dokumentum indexet.
- **[A QnA Maker][qna-maker]**. A QnA Maker egy olyan felhőalapú API-szolgáltatás, amely egy beszélgetés típusú, kérdésekből és válaszokból álló réteget hoz létre az adatok alapján. Általában azokat betölti a szolgáltatásban tárolt részben strukturált GYIK például tartalmazó. Ezzel hozzon létre egy Tudásbázis természetes nyelvű kérdések megválaszolásához.
- **[Webalkalmazás][webapp]**. Ha a robot van szüksége, meglévő service által nem biztosított AI-megoldások, megvalósítani a saját egyéni AI, és üzemeltetnie webalkalmazásként. Ez biztosítja, hogy egy webes végpontra robotja meghívásához.

### <a name="data-ingestion"></a>Adatfeldolgozás

A robot kell betöltött előkészített és nyers adatok fogja alkalmazni. Vegye figyelembe, amellyel ez a folyamat a következő beállítások bármelyikét:

- **[Az Azure Data Factory][data-factory]**. A Data Factory amellyel előkészíthető és automatizálható az adatok áthelyezését és átalakítását.
- **[A Logic Apps][logic-apps]**. A Logic Apps az alkalmazásokat, adatokat és szolgáltatásokat integráló munkafolyamatokat hoz létre kiszolgáló nélküli platform. A Logic Apps alkalmazások, beleértve az Office 365 adatösszekötők biztosít.
- **[Az Azure Functions][functions]**. Az Azure Functions segítségével egyéni kiszolgáló nélküli programkód, amely hív írhat egy [eseményindító] [ functions-triggers] &mdash; például, ha egy dokumentum hozzáadása a blob storage vagy a Cosmos DB.

### <a name="logging-and-monitoring"></a>Naplózás és figyelés

- **[Az Application Insights][app-insights]**. Az Application Insights segítségével jelentkezzen a robot alkalmazásmetrikák monitorozási, diagnosztikai és elemzési célból.
- **[Az Azure Blob Storage][blob]**. A BLOB storage nagy mennyiségű strukturálatlan adat tárolására van optimalizálva.
- **[A cosmos DB][cosmosdb]**. A cosmos DB kiválóan alkalmas a szolgáltatásban tárolt részben strukturált naplóadatokat, például a beszélgetések tárolására.
- **[Power BI][power-bi]**. A robot a figyelési irányítópult létrehozásához használja a Power bi-ban.

### <a name="security-and-governance"></a>Biztonsági és cégirányítási

- **[Az Azure Active Directory] [ aad]**  (Azure AD). Felhasználók egy identitásszolgáltató, például az Azure AD hitelesíti magát. A Bot Service a hitelesítési folyamat és a OAuth token kezelését végzi. Lásd: [hitelesítés hozzáadása az Azure Bot Service-n keresztül a robot][bot-authentication].
- **[Az Azure Key Vault][key-vault]**. Hitelesítő adatok és egyéb titkos kulcsok a Key vault Store.

### <a name="quality-assurance-and-enhancements"></a>Minőségbiztosítási és fejlesztések

- **[Az Azure DevOps][devops]**. Számos alkalmazás kezelését, beleértve a verziókövetés, létrehozása, tesztelése, üzembe helyezési és projektek nyomon követésére szolgáltatásokat nyújt.
- **[A VS Code] [ vscode]**  könnyű Kódszerkesztő, az alkalmazások fejlesztéséhez. Bármely más IDE hasonló funkciókat használhatja.

## <a name="design-considerations"></a>Kialakítási szempontok

Magas szinten a természetes nyelvi bot lehet osztani a robot funkció (a "agy") és a feldolgozóiszerepkör követelményeket (a "törzs"). Az agy a tartomány-t támogató összetevők, beleértve a robot és gépi Tanulási képességek tartalmazza. Más összetevők olyan tartományi agnosztikus és a cím nem funkcionális követelmények például a CI/CD, a minőségbiztosítás és a biztonság.

![A robot funkció logikai diagramja](./_images/conversational-bot-logical.png)

Ez az architektúra tulajdonságairól bevitele, mielőtt kezdjük az adatok az adatfolyam minden alösszetevője a kialakítás keresztül. Az adatfolyam adatok felhasználó által kezdeményezett, és a rendszer által kezdeményezett folyamatokat tartalmaz.

### <a name="user-message-flow"></a>Felhasználói üzenet folyamat

**Hitelesítés**. Hitelesítik magukat a robottal a kommunikációs csatornát által biztosított mechanizmus a felhasználóknak el. A bot framework támogatja a sok kommunikációs csatornákról, például a Cortana, a Microsoft Teams, a Facebook Messenger, a kommunikációs és Slack. A csatornák, lásd: [robotprogramok csatlakozni csatornák](/azure/bot-service/bot-service-manage-channels). A robot létrehozásakor az Azure Bot Service az [webes csevegési] [ webchat] csatorna konfigurálása automatikusan történik. Ez a csatorna lehetővé teszi a felhasználók közvetlenül a weblap a robottal lépnek kapcsolatba. Emellett kapcsolódhat a robot egyéni alkalmazás használatával az [közvetlen vonal](/azure/bot-service/bot-service-channel-connect-directline) csatorna. A felhasználó identitása segítségével adja meg a szerepköralapú hozzáférés-vezérlést, valamint feltárhatja, hogy személyre szabott tartalmat szolgálnak ki.

**Felhasználói üzenet**. A hitelesítést követően a felhasználó egy üzenetet küld a robot. A robot kiolvassa az üzenetet, és például továbbítja azt a természetes nyelvi hangfelismerési szolgáltatás [LUIS](/azure/cognitive-services/luis/). Ezzel a lépéssel lekéri a **leképezések** (Mi a felhasználó szeretne tenni) és **entitások** (mely a felhasználó van érdekelné dolgot). A robot majd összeállít egy lekérdezést, amely átadja egy szolgáltatás, amely információkat, például szolgál [Azure Search] [ search] dokumentum lekérésre [QnA Maker](https://www.qnamaker.ai/) – gyakori kérdések vagy egy egyéni ismeretek Alap. A robot ezekkel az eredményekkel használatával hozhat létre a választ. Egy adott lekérdezésre vonatkozó a legjobb eredményt adja, hogy a robot előfordulhat, hogy hívásokat több vissza oda-e távoli szolgáltatások.

**Válasz**. Ezen a ponton a robot megállapítása szerint a legjobb választ, és elküldi azokat a felhasználó. Ha a megbízhatósági pontszám a legjobban megfelelő választ az alacsony, a válasz lehet Egyértelműsítő kérdése vagy nyugtázása a robot nem sikerült megfelelően válaszolnak.

**Naplózás**. Ha egy felhasználói kérelem érkezik, vagy a válasz érkezik, összes beszélgetés művelet naplózni kell naplózási áruházban, teljesítmény-mérőszámokat és a külső szolgáltatások általános hibák együtt. Ezek a naplók lesz hasznos később problémák diagnosztizálásakor és növeli a rendszer.

**Visszajelzés**. Egy másik jó gyakorlat, hogy felhasználói visszajelzések és elégedettségi pontszámok gyűjtése. A robot végső válasz legfeljebb utáni a bot kérdezze meg a felhasználót, hogy kielégítő választ értékelése. Visszajelzés segítséget nyújtanak a hidegindítás probléma megoldására, beszédfelismerés, és folyamatosan javítja a válaszokat.

### <a name="system-data-flow"></a>Rendszer-adatfolyam

**ETL**. A robot információkat és a egy ETL-folyamat a háttérben a nyers adatokból kinyert Tudásbázis támaszkodik. Lehet, hogy ezeket az adatokat (SQL database) strukturált, félig strukturált (CRM-rendszer, gyakori kérdések) vagy strukturálatlan (Word-dokumentumok, PDF-, webalkalmazás-naplók). Egy ETL-alrendszer kinyeri az adatokat, meghatározott ütemezés szerint. A tartalom alakította át, és bővített, akkor egy köztes adattár, például a Cosmos DB vagy az Azure Blob Storage-be.

A köztes adatokat ezután dokumentum lekérése az Azure Search szolgáltatásba indexelt, betölti a QnA Maker, kérdés létrehozása, és válaszoljon pár, vagy be egyéni webalkalmazásokat a strukturálatlan szöveg feldolgozáshoz. Az adatok egy LUIS-modell, szándék és egyéb entitások kivonási betanításához is szolgál.

**Minőségbiztosítás**. A beszélgetés naplók segítségével diagnosztizálhatja és hibajavítás, nyújt betekintést a robot használatának és nyomon követheti az általános teljesítményét. A robot teljesítményének javítása érdekében AI-modellek átképezési visszajelzés adatok hasznos.

## <a name="building-a-bot"></a>A robot készítése

Akár egyetlen sornyi kódot ír, mielőtt fontos írni a egy működési specifikációt, így a fejlesztői csapat rendelkezik egy egyértelmű kiindulópont annak megértéséhez, mely a robot várhatóan tegye. A specifikációnak Tudásbázis különböző tartományokban található felhasználói bevitelek és várt bot válaszok viszonylag átfogó listáját tartalmaznia kell. Ez a dokumentum élő lesz egy hasznos információt az útmutató a fejlesztés és tesztelés a robot.

### <a name="ingest-data"></a>Adatok betöltése

Ezután azonosíthatja azokat az adatforrásokat, lehetővé teszi a felhasználók intelligensen interakcióba robot. Ahogy korábban említettük, ezek az adatforrások strukturált, félig strukturált vagy strukturálatlan adatkészletek tartalmazhatnak. Ha most ismerkedik, egy jó módszer, az adatok egyszeri másolatot készítsen a központi áruházban, például a Cosmos DB vagy az Azure Storage. Ahogy halad, létre kell hoznia egy automatizált adatfeldolgozási folyamat az adatok naprakészen tartása. A Data Factory, a Functions és a Logic Apps a lehetőségek egy automatizált adatfeldolgozási folyamat között. Attól függően, az adattárak és a sémákat ezek a módszerek kombinációját használhatja.

Használatának megkezdése, mivel célszerű manuálisan az Azure-erőforrások létrehozása az Azure portal használatával. Később a további gondolkodási üzembe kell ezeket az erőforrásokat üzembe helyezésének automatizálása.

### <a name="core-bot-logic-and-ux"></a>Fő bot logikai és a felhasználói felület

Ha már rendelkezik egy specifikációt és néhány adatot, amelyeket felhasználva megvalósíthatja a robot készítésének megkezdéséhez ideje. Koncentráljunk a robot fő logikai. Ez az a kódot, amely kezeli a beszélgetést a felhasználóval, többek között az útválasztási logikát Egyértelműsítő logikát, és a naplózást. Első lépésként a megismerése a [Bot Framework][bot-framework], többek között:

- Alapvető fogalmait, és különösen a keretrendszer használt terminológiával [beszélgetések], [Ennek a], és [tevékenységek].
- A [Bot összekötő szolgáltatás](/azure/bot-service/rest-api/bot-framework-rest-connector-quickstart), amely kezeli a hálózatkezelés, a robot és a csatornák között.
- Hogyan beszélgetés [állapot](/azure/bot-service/bot-builder-concept-state) fenntartását, a memóriában vagy még jobb például az Azure Blob Storage vagy az Azure Cosmos DB tárolja.
- [Közbenső](/azure/bot-service/bot-builder-basics#middleware), és hogyan használható a külső szolgáltatásokkal, például a Cognitive Services robotjait kapcsolni.

A gazdag [felhasználói élmény](/azure/bot-service/bot-service-design-user-experience), számos lehetőség.

- Használhat [kártyák](/azure/bot-service/bot-service-design-user-experience#cards) gombok, képek, forgódobok és menüket tartalmazza.
- A robot speech is támogatja.
- A robot beágyazása egy alkalmazás vagy webhely még akkor is, és az alkalmazásszolgáltatási képességeit használva.

Első lépésként létrehozhatja a robot online segítségével a [Azure Bot Service](/azure/bot-service/bot-service-quickstart)lehetőségre kattint, a rendelkezésre álló C# és Node.js-sablonokat. A robot bonyolultabb, mint azonban kell helyileg a robot létrehozásához, majd telepítheti a weben. Válassza ki az integrált fejlesztői Környezetig, például a Visual Studio vagy a Visual Studio Code és a egy programozási nyelvet. SDK-k az alábbi nyelveken érhetők el:

- [C#](https://github.com/microsoft/botbuilder-dotnet)
- [JavaScript](https://github.com/microsoft/botbuilder-js)
- [Java](https://github.com/microsoft/botbuilder-java) (előzetes verzió)
- [Python](https://github.com/microsoft/botbuilder-python) (előzetes verzió)

Kiindulási pontként letöltheti a forráskódot a robot, az Azure Bot Service segítségével létrehozott. Is [mintakód](https://github.com/Microsoft/BotBuilder-Samples/blob/master/README.md), az egyszerű echo robotok, amelyekbe beépül az AI-szolgáltatások egyszerűbben fejleszthetnek robotokat.

### <a name="add-smarts-to-your-bot"></a>A robot értelem hozzáadása

Egy egyszerű robot parancsok jól definiált listájával fogja tudni használni a szabályok-alapú megközelítést elemezni a felhasználói bevitel regex-n keresztül. Ennek az előnye, hogy determinisztikus és értelmezhetővé teszi. Azonban a robot meg kell ismernie a szándékok és entitások több természetes nyelvi üzenet, amikor nincsenek AI-szolgáltatások, amelyek segítségével.

- A LUIS kifejezetten felhasználói szándékok és entitások ismertetése. A megfelelő Közepesen méretű gyűjteménye betanításához [felhasználói bevitel](/azure/cognitive-services/luis/luis-concept-utterance) és a kívánt válaszokat, és adja vissza, a leképezések és a egy felhasználó adott üzenethez tartozó entitásokat.

- Az Azure Search mellett a LUIS képes együttműködni. Az összes releváns adatok Search használatával létrehozhat kereshető indexet. A robot a LUIS által kinyert entitások indexekhez kérdezi le. Az Azure Search is támogatja a [szinonimák][synonyms], amely is szélesebbre a háló megfelelő word-hozzárendelések.

- QnA Maker egy másik szolgáltatás, a választ, amely a megadott kérdések. Például – gyakori kérdések részben strukturált adatok általában van tanítva.

A robot egyéb AI-szolgáltatások segítségével tovább bővítheti a felhasználói élményt. A [Cognitive Services suite az előre elkészített AI](https://azure.microsoft.com/en-us/services/cognitive-services/?v=18.44a) (amely tartalmazza a LUIS és a QnA Maker) szolgáltatások esetében a vizuális, beszéd, nyelvi, keresési és hely szolgáltatásokat tartalmaz. Gyorsan adhat funkciókat, például nyelvi fordítást, helyesírás-ellenőrzést, hangulatelemzés, optikai Karakterfelismerés, hely figyelésével és a tartalom-jóváhagyás. Ezek a szolgáltatások is vezetékes, a felhasználó a robot további beszédnek és intelligensen közbenső modulként.

Egy másik lehetőség, integrálhatja a saját egyéni AI-szolgáltatás. Ez a módszer bonyolultabb, de teljes rugalmasságot biztosít a gépi tanulási algoritmus, a képzés és a modell szempontjából. Például sikerült megvalósítani a saját témakör modellezés, és algoritmust használja, mint például [LDA] [ lda] hasonló vagy megfelelő dokumentumok keresése. Egy jó megoldás, az egyéni AI-megoldását mint egy webszolgáltatási végpontot, és, hívja meg a végpont a core-robot logikai. A web service tárolt App Service-ben vagy a virtuális gépek fürtjeit. [Az Azure Machine Learning] [ aml] szolgáltatások és kódtárak segítségére lehetnek a számos [képzési](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/training) és [telepítése](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/deployment) modelljeit.

## <a name="quality-assurance-and-enhancement"></a>Minőségbiztosítási és a fejlesztés

**Naplózás**. Felhasználói beszélgetések jelentkezni a robot, beleértve az alapul szolgáló teljesítmény-mérőszámok és az esetleges hibákat. Ezek a naplók a hibaelhárítás során, felhasználói interakció érdekében ismertetése és javítása, a rendszer felbecsülhetetlen értékű válhatnak. Különböző adattárakban megfelelő a naplók különböző típusú lehet. Például a webes naplók esetében a Cosmos DB beszélgetés, és az Azure Storage nagy is észleltünk adattartalmakat. fontolja meg az Application Insights. Lásd: [közvetlenül az Azure Storage-írási][transcript-storage].

**Visszajelzés**. Emellett az is fontos tudni, hogyan elégedett felhasználója a robot interakciók vannak. Ha egy rekordot, a felhasználói visszajelzéseket, ezeket az adatokat segítségével összpontosítsa figyelmét a javítása néhány olyan interakció, és a jobb teljesítmény AI-modellek átképezési. A visszajelzés segítségével, LUIS, például a modellek újratanítása a rendszerben.

**Tesztelés**. Robotprogramok tesztelés magában foglalja az egységteszteket, integrációs teszteket, regressziós tesztek és funkcionális tesztek. Tesztelési, javasoljuk, hogy a külső szolgáltatások, például az Azure Search vagy a QnA Maker, valós HTTP-válaszok rögzítése, így azokat is futtathatók anélkül, hogy a külső szolgáltatások valós hálózati hívásokat egységtesztelés során.

>[!NOTE]
> A gyorsan elindíthatja a fejlesztést, a következő területeken, tekintse meg a [Botbuilder Utils JavaScript][git-repo-base]. Ebben a tárházban tartalmazza a beépített robotokat segédprogram mintakód [a Microsoft Bot Framework v4] [ bot-framework] és a Node.js futtatására. A következő csomagokat tartalmazza:
>
> - [A cosmos DB-naplózás Store][cosmosdb-logger]. Bemutatja, hogyan tárolhatja és lekérdezése a Cosmos DB-robot naplók.
> - [Application Insights naplózási Store][appinsights-logger]. Bemutatja, hogyan tárolhatja, és lekérdezi a robot megtekintése az Application Insightsban.
> - [Visszajelzések gyűjtése közbenső][feedback-util]. Minta közbenső szoftvert, amely egy robot felhasználói visszajelzés-kérelem mechanizmust biztosít.
> - [HTTP-vizsgálat Recorder][testing util]. Rekordok HTTP-forgalom külső szolgáltatásokból a robotnak való. Előre elkészített, LUIS, az Azure Search és a QnAMaker támogatással érhető el, de a bővítmények érhetők el minden olyan szolgáltatás támogatására. Ennek segítségével automatizálhatja a robot tesztelése.
>
> Ezeket a csomagokat, a segédprogram-mintakódot a megadott, és kapható a támogatási vagy a frissítések nem garantált.

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

Fokozatosan új funkciók vagy hibajavítások megjelenésekor a robot, mivel a legcélszerűbb több központi telepítési környezetekben, például átmeneti és éles környezetben használja. Üzemelő példány használatával [tárhelyek] [ slots] a [Azure DevOps] [ devops] lehetővé teszi, hogy ehhez állásidő nélkül. A legújabb frissítések átmeneti környezetben tesztelheti előtt érvényesítheti őket az éles környezetbe. Tekintetében terhelések kezeléséhez, az App Service célja, hogy a vertikális felskálázás vagy manuálisan vagy automatikusan. Mivel a robot a Microsoft globális adatközpont-infrastruktúrába, az App Service SLA biztosítja a magas rendelkezésre állás.

## <a name="security-considerations"></a>Biztonsági szempontok

Csakúgy, mint minden más alkalmazás, a robot is kialakítani, hogy a bizalmas adatok kezeléséhez. Ezért korlátozza a felhasználók jelentkezzen be és használhatják a robot. Is korlátozhatja az adatok érhetők el, a felhasználói identitás vagy szerepkör alapján. Identitás és hozzáférés-vezérlés az Azure ad-ben és a Key Vault használatával kulcsokat és titkos kódokat.

## <a name="manageability-considerations"></a>Felügyeleti szempontok

### <a name="monitoring-and-reporting"></a>Figyelés és jelentéskészítés

Miután a robot éles üzemben fut, szüksége lesz egy fejlesztési és üzemeltetési csapat, így biztosítható, hogy. Folyamatosan figyelni a rendszert, a bot működik, maximális teljesítménnyel. A naplókat küld az Application Insights vagy a Cosmos DB használatával figyelési-irányítópultot létrehozni, vagy maga az Application Insights, a Power bi-ban vagy egy egyéni webes alkalmazás irányítópult. Riasztások a fejlesztési és üzemeltetési csapat kapjon, ha kritikus fordult elő hiba vagy a teljesítmény-elfogadható küszöbérték alá esik.

### <a name="automated-resource-deployment"></a>Erőforrások automatikus üzembe helyezésének

A robot magát, az ad át a legfrissebb adatokat, és biztosítja, hogy a megfelelő működése nagyobb rendszer csak egy részét képezi. Az összes következő más Azure-erőforrások &mdash; vezénylési adatszolgáltatások, például a Data Factory, például a Cosmos DB és a tárolási szolgáltatások &mdash; kell üzembe helyezni. Az Azure Resource Manager keresztül elérhető az Azure portal, PowerShell vagy az Azure CLI keresztül konzisztens felügyeleti réteget biztosít. A sebesség és a konzisztencia érdemes automatizálni az üzembe helyezést, ezek a módszerek egyikének használatával.

### <a name="continuous-bot-deployment"></a>A robot a folyamatos üzembe helyezés

A robot logika telepítheti közvetlenül az IDE-ből vagy a parancssorból, például az Azure parancssori felület. Ahogy a robot kiforrottá, azonban célszerű a cikkben ismertetett módon kihasználhassák a CI/CD megoldással az Azure DevOps, például egy folyamatos üzembe helyezési folyamat [folyamatos üzembe helyezés](/azure/bot-service/bot-service-build-continuous-deployment). Ez jó módja a fennakadások nélkül használható, új funkciókkal és javításokkal teszteléshez közel éles környezetben a robot megkönnyítése érdekében. Emellett tanácsos több üzembe helyezési környezetekhez, általában legalább átmeneti és üzemi van. Az Azure DevOps támogatja ezt a megközelítést.

<!-- links -->

[0]: ./_images/conversational-bot.png
[aad]: /azure/active-directory/
[Tevékenységek]: /azure/bot-service/rest-api/bot-framework-rest-connector-activities
[aml]: /azure/machine-learning/service/
[app-insights]: /azure/azure-monitor/app/app-insights-overview
[app-service]: /azure/app-service/
[blob]: /azure/storage/blobs/storage-blobs-introduction
[bot-authentication]: /azure/bot-service/bot-builder-authentication
[bot-framework]: https://dev.botframework.com/
[bot-framework-service]: /azure/bot-service/bot-builder-basics
[cognitive-services]: /azure/cognitive-services/welcome
[beszélgetések]: /azure/bot-service/bot-service-design-conversation-flow
[cosmosdb]: /azure/cosmos-db/
[data-factory]: /azure/data-factory/
[data-factory-ref-arch]: ../data/enterprise-bi-adf.md
[devops]: https://azure.microsoft.com/solutions/devops/
[functions]: /azure/azure-functions/
[functions-triggers]: /azure/azure-functions/functions-triggers-bindings
[git-repo-appinsights-logger]: https://github.com/Microsoft/botbuilder-utils-js/tree/master/packages/botbuilder-transcript-app-insights
[git-repo-base]: https://github.com/Microsoft/botbuilder-utils-js
[git-repo-cosmosdb-logger]: https://github.com/Microsoft/botbuilder-utils-js/tree/master/packages/botbuilder-transcript-cosmosdb
[git-repo-feedback-util]: https://github.com/Microsoft/botbuilder-utils-js/tree/master/packages/botbuilder-feedback
[git-repo-testing-util]: https://github.com/Microsoft/botbuilder-utils-js/tree/master/packages/botbuilder-http-test-recorder
[testing-util]: https://github.com/Microsoft/botbuilder-utils-js/tree/master/packages/botbuilder-http-test-recorder
[key-vault]: /azure/key-vault/
[lda]: https://wikipedia.org/wiki/Latent_Dirichlet_allocation/
[logic-apps]: /azure/logic-apps/logic-apps-overview
[luis]: /azure/cognitive-services/luis/
[power-bi]: /power-bi/
[qna-maker]: /azure/cognitive-services/QnAMaker/
[search]: /azure/search/
[slots]: /azure/app-service/deploy-staging-slots/
[synonyms]: /azure/search/search-synonyms
[transcript-storage]: /azure/bot-service/bot-builder-howto-v4-storage
[Ennek a]: /azure/bot-service/bot-builder-basics#defining-a-turn
[vscode]: https://azure.microsoft.com/products/visual-studio-code/
[webapp]: /azure/app-service/overview
[webchat]: /azure/bot-service/bot-service-channel-connect-webchat?view=azure-bot-service-4.0/

[cosmosdb-logger]: https://github.com/Microsoft/botbuilder-utils-js/tree/master/packages/botbuilder-transcript-cosmosdb
[appinsights-logger]: https://github.com/Microsoft/botbuilder-utils-js/tree/master/packages/botbuilder-transcript-app-insights
[feedback-util]: https://github.com/Microsoft/botbuilder-utils-js/tree/master/packages/botbuilder-feedback
[testing util]: https://github.com/Microsoft/botbuilder-utils-js/tree/master/packages/botbuilder-http-test-recorder

