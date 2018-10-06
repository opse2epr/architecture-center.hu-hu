---
title: Valós idejű csalásészlelés az Azure-ban
description: Rosszindulatú tevékenység valós időben észlelheti az Azure Event Hubs és a Stream Analytics használatával.
author: alexbuckgit
ms.date: 07/05/2018
ms.openlocfilehash: 4de988731aa1c5b0e4c0ba06fa5aed59e2bb7d81
ms.sourcegitcommit: b2a4eb132857afa70201e28d662f18458865a48e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 10/05/2018
ms.locfileid: "48818666"
---
# <a name="real-time-fraud-detection-on-azure"></a>Valós idejű csalásészlelés az Azure-ban

Ebben a példaforgatókönyvben a csalárd tranzakciókat vagy más rendellenes tevékenység észleli a valós idejű adatok elemzése kívánó szervezetek számára fontos.

Lehetséges-alkalmazások tartalmaznak rosszindulatú hitelkártya tevékenység vagy a mobiltelefon hívások azonosítása. Online analitikus rendszerek hagyományos átalakíthatja és elemezheti az adatokat a rendellenes tevékenységek azonosítása órát is igénybe vehet.

Például az Event Hubs és a Stream Analytics teljes körűen felügyelt Azure-szolgáltatások használatával a vállalatok szükségtelenné teszi a az egyes kiszolgálók felügyelete során költségeit, és a Microsoft szakértői az adatok gyűjtése a felhőben és a valós idejű elemzések felhasználásával. Ebben a forgatókönyvben kifejezetten címeket a rosszindulatú tevékenységek felismerése. Ha más igények data Analytics, a rendelkezésre álló lista tekintse át [Azure elemzési szolgáltatások][product-category].

Ez a minta egy része, egy szélesebb körű adatok feldolgozási architektúra és stratégia jelöli. A cikk későbbi részében egy általános architektúrát ezt egyéb lehetőségeket ismertetése.

## <a name="relevant-use-cases"></a>Alkalmazási helyzetek

Ebben a forgatókönyvben a következő használati esetek, vegye figyelembe:

* Észleli a csaló mobiltelefon meghívja a távközlési forgatókönyvekben.
* Rosszindulatú hitelkártya-tranzakciók a banki intézmények azonosítása.
* Kiskereskedelem és e-kereskedelmi forgatókönyvekben csaló vásárlások azonosítása.

## <a name="architecture"></a>Architektúra

![Az Azure-összetevőket a csalások valós idejű észlelése forgatókönyv architektúrájának áttekintése][architecture]

Ebben a forgatókönyvben egy valós idejű elemzési folyamatok háttér-összetevőinek ismerteti. Áramlanak keresztül az adatok a forgatókönyv a következő:

1. Mobiltelefon hívása metaadatainak egy Azure Event Hubs-példány is küld a forrásrendszerben. 
2. Stream Analytics-feladat elindult, amelyeket a hub-eseményforrás keresztül adatokat fogad.
3. A Stream Analytics-feladat egy előre definiált lekérdezést a bemeneti stream át és elemez, akkor egy rosszindulatú tranzakció algoritmus alapján futtatja. Ez a lekérdezés a stream szegmentálásához különböző historikus egységekbe átfedésmentes ablak használ.
4. A Stream Analytics-feladat a naplófájlba írja az átalakított stream jelölő észlelt egy kimeneti csaló hívásokat fogadó Azure Blob storage-ban.

### <a name="components"></a>Összetevők

* [Az Azure Event Hubs] [ docs-event-hubs] egy valós idejű streamelési platform és Eseményfeldolgozási szolgáltatás, amely fogadására és feldolgozására másodpercenként több millió van. Az Event Hubs képes az elosztott szoftverek és eszközök által generált események, adatok vagy telemetria feldolgozására és tárolására. Ebben a forgatókönyvben az Event Hubs kap a rosszindulatú tevékenység elemezni az összes telefonhívás metaadatait.
* [Az Azure Stream Analytics] [ docs-stream-analytics] olyan eseményfeldolgozó motor, amely a nagy mennyiségű adat a eszközök és más adatforrásokhoz is elemezhet. Támogatja a kinyerésekor információk adatfolyamokból, minták és kapcsolatok is. Ezek a minták más alárendelt műveletek is indíthat. Ebben a forgatókönyvben a Stream Analytics azonosíthatja a csaló hívások Event hubs szolgáltatástól érkező bemeneti stream alakítja át.
* [A BLOB storage-](/azure/storage/blobs/storage-blobs-introduction) ebben a forgatókönyvben a Stream Analytics-feladat eredményeinek tárolására szolgál.

## <a name="considerations"></a>Megfontolandó szempontok

### <a name="alternatives"></a>Alternatív megoldások

A valós idejű üzenetbetöltés, az adattárolás, az adatfolyam-feldolgozás, storage, az analitikus adatok és elemzési és jelentéskészítési számos technológiai lehetőségek érhetők el. Ezek a beállítások, képességek és fontosabb kritériumok áttekintését lásd: [Big data-architektúrák: valós idejű feldolgozás](/azure/architecture/data-guide/technology-choices/real-time-ingestion) az Azure-Adatarchitektúrához.

Ezenkívül az adathamisítások felderítése összetett algoritmusokat állíthat elő különféle machine learning-szolgáltatások az Azure-ban. Ezek a beállítások áttekintéséhez lásd: [technológiai lehetőségek a machine Learning](/azure/architecture/data-guide/technology-choices/data-science-and-machine-learning) a a [Azure-Adatarchitektúrához](../../data-guide/index.md).

### <a name="availability"></a>Rendelkezésre állás

Az Azure Monitor egységes felhasználói felületet biztosít a különböző Azure-szolgáltatások figyelésére. További információkért lásd: [a Microsoft Azure figyelés](/azure/monitoring-and-diagnostics/monitoring-overview). Az Event Hubs és Stream Analytics is integrálva van az Azure Monitor szolgáltatással. 

Más rendelkezésre állási lehetőségekért lásd: a [rendelkezésre állási ellenőrzőlista] [ availability] a az Azure Architecture Centert.

### <a name="scalability"></a>Méretezhetőség

Ez a forgatókönyv összetevőinek kapacitású adatfeldolgozást és nagy mértékben párhuzamosított valós idejű elemzési lettek kialakítva. Azure Event hubs szolgáltatás rugalmasan méretezhető, fogadására és feldolgozására másodpercenként több millió alacsony késleltetéssel képes. Az Event Hubs képes [automatikus vertikális felskálázás](/azure/event-hubs/event-hubs-auto-inflate) használati igényeknek átviteli egységek számát. Az Azure Stream Analytics képes a streamelési adatok több forrásból származó nagy mennyiségű elemzéséhez. Szerint növelje meg a Stream Analytics vertikálisan felskálázhatja [folyamatos átviteli egységek](/azure/stream-analytics/stream-analytics-streaming-unit-consumption) lefoglalva a folyamatos átviteli feladatok végrehajtásához.

Általános tervezési méretezhető forgatókönyvet, tekintse át a [méretezési ellenőrzőlista] [ scalability] az Azure Architecture Centert a.

### <a name="security"></a>Biztonság

Az Azure Event Hubs védi az adatokat egy [hitelesítési és biztonsági modell] [ docs-event-hubs-security-model] közös hozzáférésű Jogosultságkód (SAS) jogkivonatokat és az esemény-közzétevők kombinációja alapján. Egy esemény-közzétevő egy eseményközpontba egy virtuális végpontot határozza meg. A kiadó csak használható üzeneteket küldeni egy eseményközpontba. Nem alkalmas közzétevőtől származó üzenetek fogadásához.

Általános megoldások biztonságos, tekintse át a [Azure Security dokumentációja][security].

### <a name="resiliency"></a>Rugalmasság

Rugalmas megoldások tervezésével kapcsolatos általános útmutatásért lásd: [rugalmas alkalmazások tervezése az Azure][resiliency].

## <a name="deploy-the-scenario"></a>A forgatókönyv megvalósításához

Ez a forgatókönyv üzembe helyezéséhez kövesse ezt [részletes oktatóanyag] [ tutorial] manuális központi telepítése az egyes összetevők forgatókönyv bemutatására. Ebben az oktatóanyagban egy .NET-ügyfélalkalmazás készítése a minta telefonhívás metaadatokat, és adatokat küld az event hubs-példánnyal is biztosít.

## <a name="pricing"></a>Díjszabás

Ebben a forgatókönyvben költségének megismeréséhez, a szolgáltatások mindegyike a költségkalkulátor az előre konfigurált. Tekintse meg, hogyan díjszabását szeretné módosítani az adott használati esetekhez, módosítsa a megfelelő változók várható adatmennyiség megfelelően.

Adtunk meg beolvasni a várt forgalom mennyisége alapján három példa költség profilt:

* [Kis][small-pricing]: keresztül egy standard folyamatos átviteli egység havi 1 millió esemény feldolgozására.
* [Közepes][medium-pricing]: öt standard folyamatos átviteli egység / hó – 100 millió esemény feldolgozására.
* [Nagy][large-pricing]: 20 standard szintű streamelési egységgel havonta keresztül 999 millió esemény feldolgozására.

## <a name="related-resources"></a>Kapcsolódó források (lehet, hogy a cikkek angol nyelvűek)

Csalások észlelése az összetettebb esetekhez egy gépi tanulási modellt is kihasználhatják. Machine Learning-kiszolgáló használatával létrehozott forgatókönyvek, lásd: [Machine Learning-kiszolgáló használatával csalásészlelés][r-server-fraud-detection]. Más megoldássablonokkal, Machine Learning-kiszolgáló használatával, lásd: [Data science forgatókönyvek és megoldássablonok][docs-r-server-sample-solutions]. Az Azure Data Lake Analytics használatával például megoldást talál [Using Azure Data Lake- és R csalások felderítéséhez][technet-fraud-detection].

<!-- links -->
[product-category]: https://azure.microsoft.com/product-categories/analytics/
[tutorial]: /azure/stream-analytics/stream-analytics-real-time-fraud-detection
[small-pricing]: https://azure.com/e/74149ec312c049ccba79bfb3cfa67606
[medium-pricing]: https://azure.com/e/4fc94f7376de484d8ae67a6958cae60a
[large-pricing]: https://azure.com/e/7da8804396f9428a984578700003ba42
[architecture]: ./media/architecture-fraud-detection.png
[docs-event-hubs]: /azure/event-hubs/event-hubs-what-is-event-hubs
[docs-event-hubs-security-model]: /azure/event-hubs/event-hubs-authentication-and-security-model-overview
[docs-stream-analytics]: /azure/stream-analytics/stream-analytics-introduction
[docs-r-server-sample-solutions]: /machine-learning-server/r/sample-solutions
[r-server-fraud-detection]: https://microsoft.github.io/r-server-fraud-detection/
[technet-fraud-detection]: https://blogs.technet.microsoft.com/machinelearning/2017/06/28/using-azure-data-lake-and-r-for-fraud-detection/
[availability]: /azure/architecture/checklist/availability
[scalability]: /azure/architecture/checklist/scalability
[resiliency]: ../../resiliency/index.md
[security]: /azure/security/

