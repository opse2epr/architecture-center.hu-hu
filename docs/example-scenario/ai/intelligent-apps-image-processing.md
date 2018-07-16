---
title: Képek besorolása a biztosítási kárigényekben, az Azure-ban
description: Bevált forgatókönyv képfeldolgozás, az Azure-alkalmazások készítéséhez.
author: david-stanford
ms.date: 07/05/2018
ms.openlocfilehash: 361a88234fd9ed918ab7664893f86666b4328b8c
ms.sourcegitcommit: 71cbef121c40ef36e2d6e3a088cb85c4260599b9
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 07/14/2018
ms.locfileid: "39060829"
---
# <a name="image-classification-for-insurance-claims-on-azure"></a>Képek besorolása a biztosítási kárigényekben, az Azure-ban

Ebben a példaforgatókönyvben akkor számára ideális, dolgozhassa fel.

Lehetséges alkalmazások közé tartozik a divat webhelyekhez képek besorolása, a szöveget és képeket a biztosítási követeléseket elemzése vagy a játék képernyőképek küldött telemetriai adatok megismerése. Hagyományosan vállalatok lenne kell fejleszthet szaktudását a gépi tanulási modelleket, a modelleket taníthat be, és végül futtassa a rendszerképek saját egyéni folyamaton kívül a képek az adatok beolvasásához.

Azure-szolgáltatások például a Computer Vision API és az Azure Functions használatával a vállalatok szükségtelenné teszi a az egyes kiszolgálók felügyelete során költségeit, és azzal a szakértelemmel, amelyeket a Microsoft már körül feldolgozási rendszerképek kihasználva A cognitive services. Ebben a forgatókönyvben kifejezetten címek egy kép feldolgozási forgatókönyvet. Ha eltérő AI igényeit, fontolja meg a teljes körű [Cognitive Services][cognitive-docs].

## <a name="related-use-cases"></a>Kapcsolódó alkalmazási helyzetek

Ebben a forgatókönyvben a következő használati esetek, vegye figyelembe:

* Divat webhely-rendszerképek besorolása.
* Játékok pillanatképeiért küldött telemetriai adatok besorolását.

## <a name="architecture"></a>Architektúra

![Intelligens alkalmazások architektúra - számítógépes látástechnológia][architecture-computer-vision]

Ebben a forgatókönyvben egy webes vagy mobilalkalmazásaiba háttér-összetevőinek ismerteti. Áramlanak keresztül az adatok a forgatókönyv a következő:

1. Az Azure Functions az API-réteget funkcionál. Ezen API-k engedélyezése az alkalmazás-rendszerképek feltöltése és a Cosmos DB-adatokat lekérni.

2. Képek feltöltése mikor keresztül egy API-hívás, Blob storage-ban van tárolva.

3. Új fájlok hozzáadásával a Blob storage elindít egy EventGrid értesítést kell küldeni egy Azure-függvényt.

4. Az Azure Functions egy hivatkozást küld az újonnan feltöltött fájlt a Computer Vision API elemzéséhez.

5. A Computer Vision API az adatok adott vissza, ha az Azure Functions feljegyzi megőrizni a képek metaadataira mellett az elemzés eredményeit a Cosmos DB-ben.

### <a name="components"></a>Összetevők

* [Computer Vision API] [ computer-vision-docs] a Cognitive Services-csomag része, és minden egyes képe kapcsolatos információk olvashatók be.

* [Az Azure Functions] [ functions-docs] a háttérrendszeri API kínál a webes alkalmazás, valamint a feltöltött képek eseményfeldolgozás.

* [Event Grid] [ eventgrid-docs] elindít egy eseményt, amikor a blob storage-bA feltöltött új lemezképet. A lemezkép ezután feldolgozása az Azure functions használatával.

* [A BLOB Storage-] [ storage-docs] tárolja az összes rendszerkép fájlt, amely a webes alkalmazás is bármely statikus fájlként a webes alkalmazás használ fel lesz töltve.

* [A cosmos DB] [ cosmos-docs] minden egyes feltöltött, beleértve a Computer Vision API, a feldolgozás eredményét lemezkép metaadatait tárolja.

## <a name="alternatives"></a>Alternatív megoldások

* [Custom Vision Service][custom-vision-docs]. A Computer Vision API-készletet ad vissza, [besorolás-alapú kategóriák][cv-categories]. Ha kell, amely nem a Computer Vision API által visszaadott adatok feldolgozásához, fontolja meg a Custom Vision Service, amely hozhat létre olyan egyéni rendszerkép osztályozó eszközökkel.

* [Az Azure Search][azure-search-docs]. Ha a használati eset szerint az adott feltételnek-rendszerképek keresése a metaadatok lekérdezése, fontolja meg az Azure Search használatával. Jelenleg előzetes verzióban elérhető [Cognitive search] [ cognitive-search] zökkenőmentesen integrálható az ebben a munkafolyamatban.

## <a name="considerations"></a>Megfontolandó szempontok

### <a name="scalability"></a>Méretezhetőség

Az esetek többségében ez a forgatókönyv összetevőinek összes felügyelt szolgáltatások, amelyek automatikusan skálázzák. Néhány fontosabb kivételek: az Azure Functions egy legfeljebb 200 példányra határral rendelkezik. Ha ezen felüli van szüksége, fontolja meg több régióban vagy alkalmazás tervek.

A cosmos DB nem automatikus skálázás kiosztott kérelemegységek (RU) tekintetében.  A követelmények becsléséhez tekintse át [kérelemegységek] [ request-units] dokumentációban. A teljes mértékben kihasználhatja a Cosmos DB méretezése meg kell is vessen egy pillantást [kulcsok particionálása][partition-key].

NoSQL-adatbázisok gyakran kereskedelmi rendelkezésre állását, méretezhetőségét és a partíció (abban az értelemben, a CAP-tétel) konzisztencia.  Azonban esetén kulcs-érték típusú adatmodelleket ebben a forgatókönyvben használt, tranzakció-konzisztencia ritkán szükség van, a legtöbb műveletekre atomi definíció szerint. További útmutatást [a megfelelő adattároló kiválasztása](../../guide/technology-choices/data-store-overview.md) az architektúra-központ érhető el.

Általános méretezhető megoldások, tekintse át a [méretezési ellenőrzőlista] [ scalability] a az Azure Architecture Centert.

### <a name="security"></a>Biztonság

[Felügyelt szolgáltatásidentitások] [ msi] (MSI) belső fiókjába más erőforrásokhoz való hozzáférést biztosítanak, és az Azure Functions majd hozzárendelve. Csak a szükséges erőforrásokhoz való hozzáférés engedélyezése ezen identitások győződjön meg arról, hogy semmi sem extra ki van téve a függvények (és esetleg az ügyfelek számára).  

Általános megoldások biztonságos, tekintse át a [Azure Security dokumentációja][security].

### <a name="resiliency"></a>Rugalmasság

Ebben a forgatókönyvben a összetevőket felügyeli, így egy regionális szinten az összes rugalmas automatikusan.

Rugalmas megoldások tervezésével kapcsolatos általános útmutatásért lásd: [rugalmas alkalmazások tervezése az Azure][resiliency].

## <a name="pricing"></a>Díjszabás

Ebben a forgatókönyvben költségének megismeréséhez, a szolgáltatások mindegyike a költségkalkulátor az előre konfigurált. Tekintse meg, hogyan díjszabását szeretné módosítani az adott használati esetekhez, módosítsa a megfelelő változókat egyezik a várt forgalomhoz.

Három példa költség-profilok forgalom mennyisége alapján biztosítunk (feltételezzük összes rendszerképekkel 100kb méretű):

* [Kis][pricing]: Ez ad eredményül, amelyek feldolgozása &lt; 5000 képek egy hónapban.
* [Közepes][medium-pricing]: Ez havi 500 000 lemezképek feldolgozására utal.
* [Nagy][large-pricing]: Ez havi 50 milliót lemezképek feldolgozására utal.

## <a name="related-resources"></a>Kapcsolódó erőforrások

A képzési ebben a forgatókönyvben, lásd: [egy kiszolgáló nélküli webalkalmazás létrehozása az Azure-ban][serverless].  

Mielőtt ez éles környezetben, tekintse át az Azure Functions [ajánlott eljárások][functions-best-practices].

<!-- links -->
[pricing]: https://azure.com/e/f9b59d238b43423683db73f4a31dc380
[medium-pricing]: https://azure.com/e/7c7fc474db344b87aae93bc29ae27108
[large-pricing]: https://azure.com/e/cbadbca30f8640d6a061f8457a74ba7d
[functions-docs]: /azure/azure-functions/
[computer-vision-docs]: /azure/cognitive-services/computer-vision/home
[storage-docs]: /azure/storage/
[azure-search-docs]: /azure/search/
[cognitive-search]: /azure/search/cognitive-search-concept-intro
[architecture-computer-vision]: ./media/architecture-computer-vision.png
[serverless]: /azure/functions/tutorial-static-website-serverless-api-with-database
[cosmos-docs]: /azure/cosmos-db/
[eventgrid-docs]: /azure/event-grid/
[cognitive-docs]: /azure/#pivot=products&panel=ai
[custom-vision-docs]: /azure/cognitive-services/Custom-Vision-Service/home
[cv-categories]: /azure/cognitive-services/computer-vision/home#the-86-category-concept
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[functions-best-practices]: /azure/azure-functions/functions-best-practices
[msi]: /azure/app-service/app-service-managed-service-identity
[request-units]: /azure/cosmos-db/request-units
[partition-key]: /azure/cosmos-db/partition-data