---
title: Képbesorolás biztosítási követelésekhez az Azure-ban
description: Képfeldolgozást építhet be Azure-alkalmazásaiba.
author: david-stanford
ms.date: 07/05/2018
ms.openlocfilehash: 9640f8b5454891ed00f669bada9f7c9c69b89734
ms.sourcegitcommit: 0a31fad9b68d54e2858314ca5fe6cba6c6b95ae4
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/13/2018
ms.locfileid: "51610532"
---
# <a name="image-classification-for-insurance-claims-on-azure"></a>Képbesorolás biztosítási követelésekhez az Azure-ban

Ebben a forgatókönyvben fontos ideális, dolgozhassa fel.

Lehetséges alkalmazások közé tartozik a divat webhelyekhez képek besorolása, a szöveget és képeket a biztosítási követeléseket elemzése vagy a játék képernyőképek küldött telemetriai adatok megismerése. Hagyományosan vállalatok lenne kell fejleszthet szaktudását a gépi tanulási modelleket, a modelleket taníthat be, és végül futtassa a rendszerképek saját egyéni folyamaton kívül a képek az adatok beolvasásához.

Azure-szolgáltatások például a Computer Vision API és az Azure Functions használatával a vállalatok szükségtelenné teszi a az egyes kiszolgálók felügyelete során költségeit, és azzal a szakértelemmel, amelyeket a Microsoft már körül feldolgozási rendszerképek kihasználva A cognitive Services. Ebben a példaforgatókönyvben kifejezetten egy képfeldolgozási használatieset-címek. Ha eltérő AI igényeit, fontolja meg a teljes körű [Cognitive Services](/azure/#pivot=products&panel=ai).

## <a name="relevant-use-cases"></a>Alkalmazási helyzetek

Egyéb alkalmazási helyzetek a következők:

* Divat webhely-rendszerképek besorolása.
* Játékok pillanatképeiért küldött telemetriai adatok besorolása.

## <a name="architecture"></a>Architektúra

![Képek besorolása architektúrája][architecture]

Ebben a forgatókönyvben egy webes vagy mobilalkalmazásaiba háttér-összetevőinek ismerteti. Áramlanak keresztül az adatok a forgatókönyv a következő:

1. Az API-réteget az Azure Functions használatával lett összeállítva. Ezen API-k engedélyezése az alkalmazás-rendszerképek feltöltése és a Cosmos DB-adatokat lekérni.
2. Képek feltöltése mikor keresztül egy API-hívás, Blob storage-ban van tárolva.
3. Új fájlok hozzáadásával a Blob storage elindít egy Event Grid értesítést kell küldeni egy Azure-függvényt.
4. Az Azure Functions egy hivatkozást küld az újonnan feltöltött fájlt a Computer Vision API elemzéséhez.
5. A Computer Vision API az adatok adott vissza, ha az Azure Functions feljegyzi megőrizni a képek metaadatai és az elemzés eredményeit a Cosmos DB-ben.

### <a name="components"></a>Összetevők

* [Computer Vision API](/azure/cognitive-services/computer-vision/home) a Cognitive Services-csomag része, és minden egyes képe kapcsolatos információk olvashatók be.
* [Az Azure Functions](/azure/azure-functions/functions-overview) a háttérrendszeri API-t biztosít a webes alkalmazás, valamint a feltöltött képek eseményfeldolgozás.
* [Event Grid](/azure/event-grid/overview) elindít egy eseményt, amikor a blob storage-bA feltöltött új lemezképet. A lemezkép ezután feldolgozása az Azure functions használatával.
* [A BLOB storage-](/azure/storage/blobs/storage-blobs-introduction) tárolja az összes rendszerkép fájlt, amely a webes alkalmazás is bármely statikus fájlként a webes alkalmazás használ fel lesz töltve.
* [A cosmos DB](/azure/cosmos-db/introduction) minden egyes feltöltött, beleértve a Computer Vision API, a feldolgozás eredményét lemezkép metaadatait tárolja.

## <a name="alternatives"></a>Alternatív megoldások

* [Custom Vision Service](/azure/cognitive-services/custom-vision-service/home). A Computer Vision API-készletet ad vissza, [besorolás-alapú kategóriák][cv-categories]. Ha kell, amely nem a Computer Vision API által visszaadott adatok feldolgozásához, fontolja meg a Custom Vision Service, amely hozhat létre olyan egyéni rendszerkép osztályozó eszközökkel.
* [Azure Search](/azure/search/search-what-is-azure-search). Ha a használati eset szerint az adott feltételnek-rendszerképek keresése a metaadatok lekérdezése, fontolja meg az Azure Search használatával. Jelenleg előzetes verzióban elérhető [Cognitive search](/azure/search/cognitive-search-concept-intro) zökkenőmentesen integrálható az ebben a munkafolyamatban.

## <a name="considerations"></a>Megfontolandó szempontok

### <a name="scalability"></a>Méretezhetőség

Ebben a példában a forgatókönyvben használt összetevőket a legtöbb felügyelt szolgáltatások, amelyek automatikusan skálázzák. Néhány fontosabb kivételek: az Azure Functions egy legfeljebb 200 példányra határral rendelkezik. Ha ezt a korlátot mellett van szüksége, fontolja meg több régióban vagy alkalmazás tervek.

A cosmos DB nem automatikus skálázási kiosztott kérelemegységek (RU) tekintetében. A követelmények becsléséhez tekintse át [kérelemegységek](/azure/cosmos-db/request-units) dokumentációban. Megismerheti a teljes mértékben kihasználhatja a Cosmos DB méretezése, hogyan [kulcsok particionálása](/azure/cosmos-db/partition-data) munkahelyi a cosmos DB.

NoSQL-adatbázisok gyakran kereskedelmi konzisztencia (abban az értelemben, a CAP-tétel), a rendelkezésre állás, a méretezhetőség és a particionálást. Ebben a példában a forgatókönyvben egy kulcs-érték típusú adatok modellt használja, és a tranzakció-konzisztencia ritkán van szükség, mivel a legtöbb műveletek atomi definíció szerint. További útmutatást [a megfelelő adattároló kiválasztása](../../guide/technology-choices/data-store-overview.md) érhető el az Azure Architecture Centert. Ha a magas konzisztencia igényel, akkor [a konzisztenciaszint kiválasztása](/azure/cosmos-db/consistency-levels) a cosmos DB.

Általános méretezhető megoldások, tekintse át a [méretezési ellenőrzőlista] [ scalability] a az Azure Architecture Centert.

### <a name="security"></a>Biztonság

[Felügyelt identitások az Azure-erőforrások] [ msi] más fiókját a belső erőforrásokhoz való hozzáférést biztosítanak, és az Azure Functions majd hozzárendelve. Csak a szükséges erőforrásokhoz való hozzáférés engedélyezése ezen identitások győződjön meg arról, hogy semmi sem extra ki van téve a függvények (és esetleg az ügyfelek számára).

Általános megoldások biztonságos, tekintse át a [Azure Security dokumentációja][security].

### <a name="resiliency"></a>Rugalmasság

Ebben a forgatókönyvben a összetevőket felügyeli, így egy regionális szinten az összes rugalmas automatikusan.

Rugalmas megoldások tervezésével kapcsolatos általános útmutatásért lásd: [rugalmas alkalmazások tervezése az Azure][resiliency].

## <a name="pricing"></a>Díjszabás

Ebben a forgatókönyvben költségének megismeréséhez, a szolgáltatások mindegyike a költségkalkulátor az előre konfigurált. Tekintse meg, hogyan díjszabását szeretné módosítani az adott használati esetekhez, módosítsa a megfelelő változókat egyezik a várt forgalomhoz.

Három példa költség-profilok forgalom mennyisége alapján biztosítunk (feltételezzük összes rendszerképekkel 100 kb méretű):

* [Kis][small-pricing]: utal. a díjszabási példa feldolgozási &lt; 5000 képek egy hónapban.
* [Közepes][medium-pricing]: a díjszabási Példa havi 500 000 képek feldolgozása utal.
* [Nagy][large-pricing]: a díjszabási Példa havi 50 milliót képek feldolgozása utal.

## <a name="related-resources"></a>Kapcsolódó források (lehet, hogy a cikkek angol nyelvűek)

A képzési, lásd: [egy kiszolgáló nélküli webalkalmazás létrehozása az Azure-ban][serverless].

Ebben a példaforgatókönyvben az éles környezetben üzembe helyezése előtt tekintse át az ajánlott eljárásokat, [optimalizálás, teljesítményének és megbízhatóságának az Azure Functions][functions-best-practices].

<!-- links -->
[architecture]: ./media/architecture-intelligent-apps-image-processing.png
[small-pricing]: https://azure.com/e/f9b59d238b43423683db73f4a31dc380
[medium-pricing]: https://azure.com/e/7c7fc474db344b87aae93bc29ae27108
[large-pricing]: https://azure.com/e/cbadbca30f8640d6a061f8457a74ba7d
[cognitive-search]: /azure/search/cognitive-search-concept-intro
[serverless]: /azure/functions/tutorial-static-website-serverless-api-with-database
[cv-categories]: /azure/cognitive-services/computer-vision/home#the-86-category-concept
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[functions-best-practices]: /azure/azure-functions/functions-best-practices
[msi]: /azure/app-service/app-service-managed-service-identity
