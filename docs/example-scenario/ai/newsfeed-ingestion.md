---
title: Az Azure-ban hírcsatornák átirányítandó és elemzési hírei
description: Létrehoz egy folyamatot feldolgozására, és a szöveg, képek, hangulatát és más RSS-hírcsatornák csak Azure-szolgáltatások, többek között az Azure Cosmos DB és az Azure Cognitive Services használata az adatok elemzéséhez.
author: njray
ms.date: 2/1/2019
ms.custom: azcat-ai, AI
social_image_url: /azure/architecture/example-scenario/ai/media/mass-ingestion-newsfeeds-architecture.png
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.openlocfilehash: e1bc636103753d474b545a6e3e9118b2ef302b34
ms.sourcegitcommit: ea97ac004c38c6b456794c1a8eef29f8d2b77d50
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/26/2019
ms.locfileid: "58489236"
---
# <a name="mass-ingestion-and-analysis-of-news-feeds-on-azure"></a>Az Azure-ban hírcsatornák átirányítandó és elemzési hírei

Ebben a példaforgatókönyvben átirányítandó és közel valós idejű elemzése a nyilvános RSS-hírcsatornák használata dokumentumok folyamatot ismerteti.  Az Azure Cognitive Services használatával például szövegfordítás, arcfelismerés és hangulatfelismerés hasznos információkat.

Ez a forgatókönyv példáit tartalmazza [angol][english], [orosz][russian], és [német] [ german] hírek hírcsatornák, de könnyen kiterjesztheti ezt más RSS-hírcsatornák. Az üzembe helyezés megkönnyítéséhez az adatok gyűjtése, feldolgozása és elemzése alapján teljes egészében az Azure-szolgáltatásokat.

## <a name="relevant-use-cases"></a>Alkalmazási helyzetek

Bár ebben a forgatókönyvben az RSS-hírcsatornák feldolgozási alapul, fontos bármely dokumentum, a webhely vagy a cikk, ahol kell:

* A választott nyelven szöveg lefordítása.
* Keresse meg a kulcskifejezéseket, az entitások és a felhasználói vélemények a digitális tartalom.
* Képek digitális cikk társított objektumokat, szöveg és tereptárgyak felismerése képes észlelni.
* Képes észlelni a nemek és a korszűrő személyek digitális tartalmak társított képet.

## <a name="architecture"></a>Architektúra

![][architecture]

Az adatok a következőképpen folyamatok a megoldáson keresztül:

1. Egy RSS-hírcsatorna a generátort, amely lekéri az adatokat egy dokumentum vagy cikk funkcionál. Például egy cikken, az adatok általában tartalmaz egy címet, a hír eredeti törzse összegzését, és időnként lemezképeket.

2. A generátor vagy a betöltési folyamat szúr be egy Azure Cosmos DB a cikket, és minden hozzá kapcsolódó képek [gyűjtemény][collection].

3. Értesítés aktiválása egy betöltés függvényt az Azure Functions, amely a cikk szövegébe tárolja a cosmos DB és a cikk képek (ha vannak) az Azure Blob Storage.  A cikk majd átadja a sorban következő üzenetsornak.

4. Fordítás függvény a várólista esemény váltja ki. Használja a [fordítása Text API] [ translate-text] Azure Cognitive Services, észlelje a nyelvet, fordítás, ha szükséges, valamint a róluk szóló véleményeket, kulcskifejezéseket és entitások gyűjteni a szervezet és a cím. Ezután átadja a cikk a sorban következő üzenetsornak.

5. A hibakeresés függvény meghívása a sorban álló cikkben. Használja a [Computer Vision] [ vision] szolgáltatás észleli a objektumok arcrész és írt szavak társított lemezképet, majd átadja a cikk a sorban következő üzenetsornak.

6. A face függvény meghívása a várólistán lévő cikk indító. Használja a [Azure Face API] [ face] szolgáltatás észleli a nemek és a korszűrő társított lemezképet az arcokat, majd továbbítja a cikk a sorban következő üzenetsornak.

7. Amikor a függvények teljes, a értesítendő függvény aktiválódik. Ez betölti a cikk a feldolgozott rekordok, és megkeresi azokat a kívánt eredményt. Ha található, a tartalom meg van jelölve, és a egy értesítést küld a rendszer a kiválasztott.

Minden egyes feldolgozási lépést, a függvény az Azure Cosmos DB kiírja az eredményeket. Végső soron az adatok segítségével igény szerint. Például használhatja, javíthatja az üzleti folyamatok, keresse meg az új ügyfeleket vagy ügyfél-elégedettségi problémák azonosításához.

### <a name="components"></a>Összetevők

Az alábbi listában szereplő Azure-összetevőket ebben a példában szolgál.

* [Az Azure Storage] [ storage] nyers kép és a egy cikk társított videofájlok tárolására szolgál. Másodlagos tárfiók létrehozása az Azure App Service és az Azure-függvény kódjának és a naplók tárolására használt.

* [Az Azure Cosmos DB] [ cosmos-db] visszatartással article szöveg, kép- és nyomkövetési adatokat. A Cognitive Services lépések eredményeit a rendszer szintén itt tárolja.

* [Az Azure Functions] [ functions] végrehajtja a függvénykódot az üzenetek üzenetsorba való válaszol, és a bejövő tartalom átalakítása. [Az Azure App Service] [ aas] üzemelteti a függvénykódot és tárolókonfigurációhoz dolgozza fel a rekordokat. Ebben a forgatókönyvben öt függvényt tartalmazza: Betöltési, átalakítási, objektum, arc, észlelése, és értesíti.

* [Az Azure Service Bus] [ service-bus] üzemelteti az Azure Service Bus-üzenetsorok, a functions által használt.

* [Az Azure Cognitive Services] [ acs] biztosít a mesterséges Intelligencia megvalósítása alapján a folyamat a [Computer Vision] [ vision] szolgáltatás [arc API][face], és [szöveg lefordítása] [ translate-text] gépi fordítási szolgáltatás.

* [Az Azure Application Insights] [ aai] analytics segítségével diagnosztizálhatja a problémákat és az alkalmazás működésének megértése biztosít.

### <a name="alternatives"></a>Alternatív megoldások

* Ez az adatfolyam egy másik minta használata helyett egy minta alapján az értesítési várólista és az Azure Functions használatával. Ha például [Azure Service Bus-témakörök] [ topics] segítségével párhuzamosan, a cikk különböző részeinek kommunikálniuk kész ebben a példában a soros feldolgozási folyamatokat. További információkért olvassa el a [üzenetsorok és témakörök][queues-topics].

* Használat [Azure Logic Apps] [ logic-app] megvalósítása a függvénykódot és a végrehajtásához a rekord szintű zárolás például [Redlock] [ redlock] (párhuzamos szükséges amíg az Azure Cosmos DB támogatja a [részleges dokumentum frissítések][partial]). További információ [funkciók és a Logic Apps összehasonlítása][compare].

* Ez az architektúra a meglévő Azure-szolgáltatások helyett egyéni AI-összetevők használatával hajtja végre. Például az a folyamat használatával egy testre szabott modellt, amely észleli az adott személyek figyelésekor az általános személyek száma, a Nemet, a kép kiterjesztése, és ebben a példában az életkor adatokat gyűjteni. Testre szabott gépi tanulási és AI-modellek használata ebben az architektúrában, hozhat létre a modellek RESTful végpontokon, így azok nem hívható meg az Azure Functions.

* Egy másik bemeneti mechanizmus használata az RSS-hírcsatornák helyett. Több generátorok vagy betöltési folyamatok segítségével Azure Cosmos DB és az Azure Storage-hírcsatorna.

## <a name="considerations"></a>Megfontolandó szempontok

Ebben a példaforgatókönyvben az egyszerűség kedvéért használja a mindössze néhány elérhető API-k és az Azure Cognitive Services szolgáltatások. Például képek képes elemezni a [Text Analytics API][text-analytics]. Ebben a forgatókönyvben a Célnyelv adatforrásmérete angol, de módosíthatja a bemenetet bármely [támogatott nyelven] [ language] tetszőleges.

### <a name="scalability"></a>Méretezhetőség

Az Azure Functions méretezése függ a [szolgáltatási csomag] [ plan] használja. Ez a megoldás feltételezi, hogy egy [Használatalapú csomag][plan-c], mely számítási power lefoglalása automatikusan történik az funkciók, szükség esetén. A fizetés csak amikor a függvények futnak. Egy másik lehetőség egy [Azure App Service] [ plan-aas] csomagra, amely lehetővé teszi a különböző mennyiségű erőforrás lefoglalásához a rétegek közötti skálázását.

Az Azure Cosmos DB, a fontos, hogy a munkaterhelés nagyjából egyenletes elosztása bizonyos megfelelően nagy számú [kulcsok particionálása][keys]. Egy tárolóban tárolt adatok teljes mennyisége vagy a teljes mennyisége nem korlátozott [átviteli] [ throughput] egy tároló által támogatott.

### <a name="management-and-logging"></a>Kezelés és naplózás

Ez a megoldás használ [Application Insights] [ aai] teljesítmény és a naplózási információk gyűjtéséhez. Az Application Insights egy példányát a központi telepítést, mint az üzembe helyezéshez szükséges egyéb szolgáltatásokat ugyanabban az erőforráscsoportban jön létre.

A megoldás által létrehozott naplók megtekintése:

1. Lépjen a [az Azure portal] [ portal] , és keresse meg az üzembe helyezéshez létrehozott erőforráscsoportot.

2. Kattintson a **Application Insights** példány.

3. Az a **Application Insights** területén lépjen **vizsgálat\\keresési** , és keresse meg az adatokat.

### <a name="security"></a>Biztonság

Az Azure Cosmos DB használ a biztonságos kapcsolódás és a közös hozzáférésű jogosultságkód – a C\# SDK-t a Microsoft által biztosított. Nincsenek nincs más kifelé irányuló surface területeket. További tudnivalók a biztonságról [ajánlott eljárások] [ db-practices] az Azure Cosmos DB.

## <a name="pricing"></a>Díjszabás

A becsült napi költség, hogy a központi telepítés rendszer körülbelül \$20 áthelyezése a rendszeren keresztül adatot nem tartalmazó régiója.

Az Azure Cosmos DB hatékony, de a lehető legnagyobb tekintetében [költség] [ db-cost] ebben a telepítésben. Egy másik tárolási megoldás az Azure Functions kódjaiba, a megadott újrabontás használhatja.

Díjszabás attól függően változik, az Azure Functions a [terv] [ function-plan] fusson.

## <a name="deploy-the-scenario"></a>A forgatókönyv megvalósításához

> [!NOTE]
> Meglévő Azure-fiókkal kell rendelkeznie. Ha nem rendelkezik Azure-előfizetéssel, első lépésként létrehozhat egy [ingyenes][free] fiókot.

Ebben a forgatókönyvben a kód érhető el a [GitHub] [ github] tárház. Ez a tárház tartalmazza az eseménylétrehozó alkalmazást, amely a folyamat ebben a bemutatóban hírcsatornák összeállításához forráskódját.

[architecture]: ./media/mass-ingestion-newsfeeds-architecture.png
[aai]: /azure/azure-monitor/app/app-insights-overview
[aas]: https://azure.microsoft.com/try/app-service/
[acs]: https://azure.microsoft.com/services/cognitive-services/directory/
[collection]: /rest/api/cosmos-db/collections
[compare]: /azure/azure-functions/functions-compare-logic-apps-ms-flow-webjobs#compare-azure-functions-and-azure-logic-apps
[cosmos-db]: /azure/cosmos-db/introduction
[db-cost]: https://azure.microsoft.com/pricing/details/cosmos-db/
[db-practices]: /azure/cosmos-db/database-security
[db-collection]: /azure/cosmos-db/databases-containers-items
[english]: https://www.nasa.gov/rss/dyn/breaking_news.rss
[face]: /azure/cognitive-services/face/overview
[free]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
[functions]: /azure/azure-functions/functions-overview
[function-plan]: /azure/azure-functions/functions-scale
[german]: http://www.bamf.de/SiteGlobals/Functions/RSS/DE/Feed/RSSNewsfeed_Meldungen
[github]: https://github.com/Azure/cognitive-services
[keys]: /azure/cosmos-db/partition-data
[language]: /azure/cognitive-services/translator/reference/v3-0-languages
[logic-app]: /azure/logic-apps/logic-apps-overview
[queues-topics]: /azure/service-bus-messaging/service-bus-queues-topics-subscriptions
[partial]: https://feedback.azure.com/forums/263030-azure-cosmos-db/suggestions/6693091-be-able-to-do-partial-updates-on-document
[plan]: /azure/azure-functions/functions-scale
[plan-aas]: /azure/azure-functions/functions-scale#app-service-plan
[plan-c]: /azure/azure-functions/functions-scale#consumption-plan
[portal]: http://portal.azure.com
[redlock]: https://redis.io/topics/distlock
[russian]: http://government.ru/all/rss/
[service-bus]: /azure/service-bus-messaging/
[storage]: /azure/storage/common/storage-account-overview 
[throughput]: /azure/cosmos-db/scaling-throughput
[topics]: /azure/service-bus-messaging/service-bus-dotnet-how-to-use-topics-subscriptions
[text-analytics]: /azure/cognitive-services/text-analytics/
[translate-text]: /azure/cognitive-services/translator/translator-info-overview
[vision]: /azure/cognitive-services/computer-vision/home
