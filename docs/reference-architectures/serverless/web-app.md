---
title: Kiszolgáló nélküli webalkalmazás
description: Látható, a kiszolgáló nélküli webalkalmazás és webes API referencia-architektúra
author: MikeWasson
ms.date: 10/16/2018
ms.openlocfilehash: c2b46a60a57381ac3fd3f77cffe53b2dab2dacd6
ms.sourcegitcommit: 113a7248b9793c670b0f2d4278d30ad8616abe6c
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 10/16/2018
ms.locfileid: "49349955"
---
# <a name="serverless-web-application"></a>Kiszolgáló nélküli webalkalmazás 

Ez a referenciaarchitektúra bemutatja a kiszolgáló nélküli webalkalmazás. Az alkalmazást a statikus tartalmat szolgáltat az Azure Blob Storage-ból, és a egy API-t az Azure Functions használatával valósítja meg. Az API beolvassa az adatokat a Cosmos DB és az eredményeket adja vissza a webalkalmazáshoz. Az architektúra egy referenciaimplementációt érhető el az [GitHub][github].

![](./_images/serverless-web-app.png)
 
A kiszolgáló nélküli kifejezés két különálló, de kapcsolódó jelentéssel rendelkezik:

- **Háttérmodul-szolgáltatás** (BaaS). Háttérbeli felhőszolgáltatásokat, például adatbázisok és a storage, az API-k, amelyek lehetővé teszik ügyfélalkalmazások számára ezek a szolgáltatások való közvetlen csatlakozáshoz adja meg. 
- **A Functions szolgáltatás** (FaaS). Ebben a modellben egy "függvény" olyan kódot, amely a felhőben üzemel, és fut egy teljesen kivonatolja a kódot futtató kiszolgálók üzemeltetési környezet. 

Mindkét definíciók rendelkezik közös a ötlete, amellyel a fejlesztők és a fejlesztési és üzemeltetési munkatársai nem szükséges központi telepítése, konfigurálása és -kiszolgálók kezelése. Ez a referenciaarchitektúra Azure Functions használatával FaaS összpontosít, bár a webes tartalmat szolgáltató az Azure Blob Storage-ból példaként szolgál a háttérkomponens-szolgáltatás. A FaaS néhány fontos jellemzői a következők:

1. A számítási erőforrások a platform által igényelt dinamikusan kiosztott.
1. Díjszabás fogyasztásalapú: csak a kód végrehajtását használt számítási erőforrások díjkötelesek.
1. A számítási erőforrások igény szerint méretezhető alapján a forgalom, anélkül, hogy a fejlesztő bármely konfigurációs tennie kellene.

Ha külső eseményindító történik, például egy HTTP-kérelem vagy egy üzenetsorba érkező üzeneteket a függvények végrehajtása történik. Ez lehetővé teszi egy [eseményvezérelt architektúra stílusának] [ event-driven] természetes, kiszolgáló nélküli architektúrák. Munka az architektúra összetevői közötti koordinációhoz, üzenetközvetítők vagy pub/sub minták használata javasolt. Az Azure-ban üzenetkezelési technológiák közötti választással kapcsolatos útmutatásért lásd: [kézbesíti az üzeneteket az Azure-szolgáltatások közötti választás][azure-messaging].

## <a name="architecture"></a>Architektúra
Az architektúra a következőkben leírt összetevőkből áll.

**A BLOB Storage-**. Statikus tartalmak, például HTML, CSS és JavaScript-fájlok, az Azure Blob Storage tárolása és kiszolgálása ügyfeleknek használatával [statikus webhelyüzemeltetésre][static-hosting]. Az összes dinamikus interakció a háttérrendszeri API-k hívása JavaScript-kód történik. Nincs kiszolgálóoldali kód megjeleníteni a weblapot. Index dokumentumok és egyéni 404-es hibalapok statikus webhelyüzemeltetésre támogatja.

> [!NOTE]
> Statikus webhely üzemeltetése jelenleg [előzetes][static-hosting-preview].

**CDN**. Használat [Azure Content Delivery Network] [ cdn] (CDN) a gyorsítótár a kisebb késés és a gyorsabb tartalomkézbesítés a tartalom, valamint biztosító HTTPS-végpontokat.

**Function Apps**. [Az Azure Functions] [ functions] kiszolgáló nélküli számítási megoldás. Az eseményvezérelt modellt használ, ahol a kódrészleteket (a "function") egy eseményindító hív. Ebben az architektúrában a függvény akkor aktiválódnak, ha egy ügyfél HTTP-kérést küld. A kérelem API-átjáró, az alábbiakban mindig továbbít.

**Az API Management**. [Az API Management] [ apim] helyezkedik el a HTTP-függvényt egy API-átjárót nyújt. Az API Management segítségével közzétehet és kezelhet az ügyfélalkalmazások által használt API-k. Egy átjáró használatával segít leválasztja az előtér-alkalmazást a háttérrendszeri API-kon keresztül. Például az API Management újraírási URL-címek, kérelmek átalakítása, mielőtt elérnék a háttér, kérelem vagy válasz fejlécek beállítása és így tovább.

Az API Management is használható például általános megfontolások megvalósításához:

- Korlátozza a használati kvótákat és a sebesség kényszerítése
- Az OAuth-jogkivonatok hitelesítés ellenőrzése
- Eltérő eredetű kérelmek (CORS) engedélyezése
- Válaszok gyorsítótárazásának
- Monitorozási és naplózási kéréseket  

Ha már nincs szüksége az összes az API Management által biztosított funkciókat, egy másik lehetőség, hogy használja [Functions-proxyk][functions-proxy]. Ez a funkció az Azure Functions lehetővé teszi, hogy egyetlen API-felületet több függvényalkalmazásra háttér-funkciók útvonalak létrehozásával meghatározhatja. Függvényproxykat korlátozott átalakításokat is elvégezheti a HTTP-kérés és válasz. Azonban nem biztosítanak azonos gazdag csoportházirend-alapú képességeit az API Management.

**A cosmos DB**. [A cosmos DB] [ cosmosdb] többmodelles adatbázis-szolgáltatás. A jelen esetben a függvény alkalmazás olvas be dokumentumok Cosmos DB az ügyfél HTTP GET kérések válaszul.

**Azure Active Directory** (Azure AD). Felhasználók jelentkezzen be a webalkalmazás az Azure AD hitelesítő adatait. Az Azure AD hozzáférési jogkivonatot az API-hoz, a webes alkalmazás használó API-kérések hitelesítéséhez adja vissza. (lásd: [hitelesítési](#authentication)).

**Az Azure Monitor**. [A figyelő] [ monitor] teljesítmény-mérőszámokat a megoldásban üzembe helyezett Azure-szolgáltatásokkal gyűjti. Ezek az irányítópult vizualizációjával képet is kaphat a megoldás állapotát. Azt is gyűjti az alkalmazásnaplókat.

**Az Azure folyamatok**. [A folyamatok] [ pipelines] van egy folyamatos integrációs (CI) és a folyamatos továbbítás (CD) a buildek esetében teszteket, szolgáltatást, és központilag telepíti az alkalmazást.

## <a name="recommendations"></a>Javaslatok

### <a name="function-app-plans"></a>Függvényalkalmazás-csomagok

Az Azure Functions támogatja a két üzemeltetési modell. Az a **használatalapú csomag**, számítási teljesítmény lefoglalása automatikusan történik meg, amikor a kódja fut.  Az a **App Service-ben** csomag, a virtuális gépek lefoglalásának a kódot. Az App Service-csomag határozza meg, hogy a virtuális gépek számát és a Virtuálisgép-méretet. 

Vegye figyelembe, hogy az App Service-csomag nem feltétlenül *kiszolgáló nélküli*, a fent megadott definíció szerint. A programozási modell megegyezik, azonban &mdash; függvény ugyanazt a kódot futtathatja a használatalapú és App Service-csomag.

Íme néhány tényező, hogy milyen típusú a csomag használatára kiválasztásakor vegye figyelembe:

- **Hidegindítási**. A használatalapú csomagok esetében a függvény, amely még nem lett meghívva nemrég számítunk fel némi késést a következő futásakor. Erre a további késésre felosztására, és a futásidejű környezet előkészítése okozza. A szolgáltatás általában sorrendjéről másodperc, de számos tényezőtől függ, többek között a tartalomfüggőségeket, ezt be kell. További információkért lásd: [ismertetése kiszolgáló nélküli hideg Start][functions-cold-start]. Hidegindítási oka az, általában több, interaktív számítási feladatai (HTTP-eseményindítók) szempont aszinkron üzenet-alapú számítási feladatokhoz (üzenetsor vagy az event hubs eseményindítók), mint a további késleltetés a felhasználók közvetlenül figyelhető meg.
- **Időkorlát**.  A használatalapú csomagban után időtúllépés egy függvény végrehajtását egy [konfigurálható] [ functions-timeout] , (és a egy legfeljebb 10 perces) időszak
- **Virtuális hálózat elkülönítési**. App Service-csomag használatával lehetővé teszi, hogy a függvények futását található egy [App Service Environment-környezet][ase], azaz egy dedikált és elkülönített üzemeltetési környezet.
- **Díjszabási modell**. A használatalapú csomag a végrehajtások száma és erőforrás-használat alapján történik (memória &times; végrehajtási idő). Az App Service-csomag Virtuálisgép-példány Termékváltozat óraszám alapján számítjuk fel. Gyakran előfordul a használatalapú csomag lehet olcsóbb, mint az App Service-csomag, mert csak kell fizetnie a számítási erőforrásokat, amelyekkel. Ez különösen igaz, ha a forgalom csúcsok és csatornák. Azonban ha egy alkalmazás állandó nagy átviteli sebességet, az App Service-csomag Előfordulhat, hogy kevesebb költséggel jár, mint a használatalapú csomag.
- **Skálázás**. Egy a használatalapú modell nagy előnye az, hogy méretezés dinamikusan igény alapján a bejövő forgalmat. Bár a skálázás gyorsan történik, akkor van még mindig készüljön fel. Bizonyos számítási feladatokhoz érdemes szándékosan overprovision a virtuális gépeket, hogy a szolgáltatás a forgalom a nulla készüljön fel időt kezelheti. Ebben az esetben fontolja meg az App Service-csomag.

### <a name="function-app-boundaries"></a>Függvény alkalmazás határok

A *függvényalkalmazás* végrehajtásához egy vagy több szükséges gazdaszolgáltatást *funkciók*. Függvényalkalmazás segítségével számos funkciót csoportosíthat egy logikai egységként. A functions belül egy függvényalkalmazást, ossza meg az ugyanazon alkalmazás beállításait, futtatási csomagot, és a központi telepítés életciklusa. Minden függvény alkalmazás rendelkezik a saját állomásnevet.  

Függvényalkalmazások csoport funkciók, amelyek életciklusa és beállításokat használja. Másik függvényalkalmazás a függvények, amelyek nem azonos életciklussal üzemeltetve lesz. 

Fontolja meg a mikroszolgáltatási megközelítést, ahol a minden függvényalkalmazáshoz egy mikroszolgáltatásban valószínűleg álló számos kapcsolódó funkciókat jelenti. A mikroszolgáltatási architektúrában a szolgáltatások laza összekapcsolással és magas működési kohézióval kell rendelkeznie. *Lazán* összefüggő azt jelenti, hogy egy szolgáltatás anélkül, hogy más szolgáltatások egyszerre frissítendő módosíthatja. *Javul* jelenti, hogy egy szolgáltatás egyetlen, jól definiált céllal rendelkezik. Ezek ötleteket további ismertetéséhez lásd: [mikroszolgáltatások tervezése: tartományelemzés][microservices-domain-analysis].

### <a name="function-bindings"></a>Függvény-kötések

Függvények [kötések] [ functions-bindings] amikor csak lehetséges. Kötések kódját csatlakozhat az adatokhoz és más Azure-szolgáltatásokkal integrálható deklaratív módszert biztosítanak. Bemeneti kötés tölti fel a bemeneti paraméter egy külső adatforrásból. Kimeneti kötés elküldi a függvény visszaadott értékének fogadására, például egy adatbázis vagy üzenetsor.

Ha például a `GetStatus` függvény a referenciaimplementációt a használja a Cosmos DB [bemeneti kötést][cosmosdb-input-binding]. Ez a kötés kereshet meg egy dokumentumot a Cosmos DB lekérdezési paraméterek, amelyeket a rendszer a HTTP-kérelem lekérdezési karakterlánc használatával van konfigurálva. Ha a dokumentum megtalálható, átadva a függvénynek paraméterként.

```csharp
[FunctionName("GetStatusFunction")]
public static Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Function, "get", Route = null)] HttpRequest req, 
    [CosmosDB(
        databaseName: "%COSMOSDB_DATABASE_NAME%",
        collectionName: "%COSMOSDB_DATABASE_COL%",
        ConnectionStringSetting = "COSMOSDB_CONNECTION_STRING",
        Id = "{Query.deviceId}",
        PartitionKey = "{Query.deviceId}")] dynamic deviceStatus, 
    ILogger log)
{
    ...
}
```

Kötések használata esetén nem kell közvetlenül a szolgáltatást, amely egyszerűbbé teszi a függvénykódot és az az adatforrás vagy a fogadó adatait is kivonatolja műsorgazdája kód írására. Bizonyos esetekben azonban szükség lehet a kötés biztosít összetettségű logikát. Ebben az esetben használja közvetlenül az Azure-ügyfél SDK-k.

## <a name="scalability-considerations"></a>Méretezési szempontok

**Függvények**. A használatalapú csomag esetében a HTTP-eseményindítóval skálázható alapján, a forgalmat. Az egyidejű függvény példányok száma korlátozva van, de minden példány egyszerre egynél több kérést is feldolgozni. Az App Service-csomag a HTTP-eseményindítóval méretezi a Virtuálisgép-példányok, rögzített érték vagy teljesítményigényektől függ az automatikus skálázási szabályok száma szerint. További információ: [Azure Functions méretezése és üzemeltetése][functions-scale]. 

**A cosmos DB**. A Cosmos DB átviteli kapacitás mérése [kérelemegység] [ ru] (RU). Egy 1 – RU átviteli sebesség egy 1 KB méretű dokumentum beolvasásához szükséges átviteli felel meg. Annak érdekében, hogy az elmúlt 10 000-et egy Cosmos DB-tárolók skálázása RU, meg kell adnia egy [partíciókulcs] [ partition-key] amikor hozza létre a tárolót, és a partíciókulcs tartalmazza minden egyes dokumentum létrehozott. Partíciókulcsok kapcsolatos további információkért lásd: [particionálási és horizontális az Azure Cosmos DB][cosmosdb-scale].

**Az API Management**. Az API Management horizontálisan, és támogatja a szabályalapú automatikus skálázást. Vegye figyelembe, hogy a méretezési tart legalább 20 perc. Ha a forgalom beül, a várt maximális burst forgalom kell kiépítenie. Az automatikus skálázás azonban hasznos óránkénti vagy napi változata, a forgalom kezelésére. További információkért lásd: [automatikus skálázása az Azure API Management-példány][apim-scale].

## <a name="disaster-recovery-considerations"></a>Vészhelyreállítási szempontok

Az üzembe helyezés itt látható egy adott Azure-régióban található. A vész-helyreállítási rugalmasabb módszer, a földrajzi elosztás funkcióinak kihasználása a különböző szolgáltatások:

- Az API Management támogatja a több régióból álló üzemelő, amely tetszőleges számú Azure-régiók között oszthatja el a egyetlen API Management-példány is használható. További információkért lásd: [egy Azure API Management-szolgáltatáspéldány üzembe helyezése több Azure-régióban][api-geo].

- Használat [Traffic Manager] [ tm] HTTP kérelmeket az elsődleges régióba. Ha az adott régióban futó a Függvényalkalmazás nem érhető el, a Traffic Manager feladatátvételt végezhet egy másodlagos régióba.

- A cosmos DB támogatja a [több fő régióban][cosmosdb-geo], amely lehetővé teszi, hogy minden olyan régióban, adja hozzá a Cosmos DB-fiókot az írási műveletek. Ha nem engedélyezi több főkiszolgálós, továbbra is adhatók át az elsődleges írási régió. A Cosmos DB ügyféloldali SDK-kkal és az Azure-függvény kötések automatikusan kezeli a feladatátvételt, így nem kell minden olyan alkalmazás konfigurációs beállításainak frissítése.

## <a name="security-considerations"></a>Biztonsági szempontok

### <a name="authentication"></a>Hitelesítés

A `GetStatus` API-referencia megvalósítása az Azure AD-kérések hitelesítéséhez. Az Azure AD támogatja az Open ID Connect protokollal, ami egy olyan hitelesítési protokoll az OAuth 2 protokoll épülnek.

Ebben az architektúrában az ügyfélalkalmazás egy egyoldalas alkalmazás (SPA), amely futtatja a böngészőben. Így az implicit engedélyezési folyamat megfelelő ügyfélalkalmazás az ilyen típusú ügyfélkódot vagy az engedélyezési kódot, rejtett, nem vezetnek. (Lásd: [melyik OAuth 2.0-s folyamat használjam?] [oauth-flow]). Itt látható a teljes folyamat:

1. A felhasználó a "Bejelentkezés" hivatkozásra a webalkalmazás kattint.
1. A böngészőben a rendszer átirányítja az Azure AD bejelentkezési oldal. 
1. A felhasználó bejelentkezik.
1. Az Azure AD vissza az ügyfélalkalmazás, beleértve a hozzáférési jogkivonatot az URL-cím töredék irányítja át.
1. A webalkalmazás az API-t hív meg, ha a hozzáférési jogkivonat szerepel a hitelesítési fejlécet. Az Alkalmazásazonosítót, a hozzáférési jogkivonatot a jogcím a célközönség (aud) zajlik. 
1. A háttérrendszeri API érvényesíti a hozzáférési jogkivonatot.

Hitelesítés konfigurálása:

- Alkalmazás regisztrálása az Azure AD-bérlőben. Ez létrehoz egy Alkalmazásazonosítót, amely az ügyfél tartalmazza a bejelentkezési URL-címmel.

- A függvény alkalmazás Azure AD-hitelesítés engedélyezéséhez. További információkért lásd: [hitelesítése és engedélyezése Azure App Service-ben][app-service-auth].

- Szabályzat hozzáadása és az API Management az előzetes engedélyezéshez érvényesítésével azonosítsa a hozzáférési jogkivonat a kérelem:

    ```xml
    <validate-jwt header-name="Authorization" failed-validation-httpcode="401" failed-validation-error-message="Unauthorized. Access token is missing or invalid.">
        <openid-config url="https://login.microsoftonline.com/[Azure AD tenant ID]/.well-known/openid-configuration" />
        <required-claims>
            <claim name="aud">
                <value>[Application ID]</value>
            </claim>
        </required-claims>
    </validate-jwt>
    ```

További részletekért tekintse meg a [GitHub információs][readme].

### <a name="authorization"></a>Engedélyezés

Számos alkalmazásban a háttérrendszeri API ellenőriznie kell, hogy egy felhasználó jogosult-e egy adott művelet végrehajtásához szükséges engedéllyel. Azt javasoljuk, hogy használjon [jogcímalapú engedélyezési][claims], amelyben a felhasználó adatai van az identitásszolgáltató (esetünkben az Azure AD) által közölt és használt engedélyezési döntésekhez. 

Egyes jogcímek biztosított belül az azonosító jogkivonat, amely az Azure AD és az ügyfél ad vissza. Ezeket a jogcímeket a belül a függvényalkalmazás az X-MS-CLIENT-egyszerű a kérelemben szereplő tartományfejléc megvizsgálásával kérheti le. Egyéb jogcímek használata [Microsoft Graph] [ graph] lekérdezése az Azure ad-ben (szükséges felhasználói beleegyezés bejelentkezés során). 

Például ha egy alkalmazás az Azure ad-ben regisztrálja, meghatározhatja egy alkalmazás-szerepkörök készletét regisztrációs alkalmazásjegyzékben. Amikor egy felhasználó bejelentkezik az alkalmazásba, az Azure AD tartalmaz egy "szerepkör" jogcím az egyes szerepkörökhöz, amelyek a felhasználó megkapta-e (beleértve a csoporttagság örökölt szerepkörök). 

A referenciaimplementációt, a függvény ellenőrzi, hogy a hitelesített felhasználó tagja a `GetStatus` alkalmazás-szerepkörökhöz. Ha nem, a függvény egy HTTP jogosulatlan (401-es) választ ad vissza. 

```csharp
[FunctionName("GetStatusFunction")]
public static Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)] HttpRequest req, 
    [CosmosDB(
        databaseName: "%COSMOSDB_DATABASE_NAME%",
        collectionName: "%COSMOSDB_DATABASE_COL%",
        ConnectionStringSetting = "COSMOSDB_CONNECTION_STRING",
        Id = "{Query.deviceId}",
        PartitionKey = "{Query.deviceId}")] dynamic deviceStatus, 
    ILogger log)
{
    log.LogInformation("Processing GetStatus request.");

    return req.HandleIfAuthorizedForRoles(new[] { GetDeviceStatusRoleName },
        async () =>
        {
            string deviceId = req.Query["deviceId"];
            if (deviceId == null)
            {
                return new BadRequestObjectResult("Missing DeviceId");
            }

            return await Task.FromResult<IActionResult>(deviceStatus != null
                    ? (ActionResult)new OkObjectResult(deviceStatus)
                    : new NotFoundResult());
        },
        log);
}
```

Ez a Kódpélda az `HandleIfAuthorizedForRoles` bővítmény módszerrel, amely ellenőrzi, hogy a szerepkör jogcím és a HTTP 401-es adja vissza, ha a jogcím nem található. A forráskód annak [Itt][HttpRequestAuthorizationExtensions]. Figyelje meg, hogy `HandleIfAuthorizedForRoles` vesz igénybe egy `ILogger` paraméter. Jogosulatlan kérelmek naplózni kell, hogy egy naplóban, és segítségével diagnosztizálhatja a problémákat, ha szükséges. Egy időben kerülje a kiszivárgását a HTTP 401-es válasz lévő bármely részletes információkat.

### <a name="cors"></a>CORS

Ez a referenciaarchitektúra a a webalkalmazás és az API nem osztoznak az azonos forrásból. Ez azt jelenti, amikor az alkalmazás meghívja az API-t, hogy az eltérő eredetű kérelem. A böngésző biztonsági beállításai megakadályozzák, hogy egy weblap AJAX-kérelmeket küldjön egy másik tartományba. Ez a korlátozás nevezzük a *azonoseredet-* és megakadályozza, hogy egy rosszindulatú webhely érzékeny adatok olvasása a másik helyről. Eltérő eredetű kérelem engedélyezéséhez adjon hozzá egy eltérő eredetű erőforrások megosztása (CORS) [házirend] [ cors-policy] , az API Management-átjáró:

```xml
<cors allow-credentials="true">
    <allowed-origins>
        <origin>[Website URL]</origin>
    </allowed-origins>
    <allowed-methods>
        <method>GET</method>
    </allowed-methods>
    <allowed-headers>
        <header>*</header>
    </allowed-headers>
</cors>
```

Ebben a példában a **engedélyezése – hitelesítő adatok** attribútum **igaz**. Ez engedélyezi a böngésző (beleértve a cookie-k) hitelesítő adatok küldése a kérelemmel. Ellenkező esetben alapértelmezés szerint a böngésző nem küld egy eltérő eredetű kérelem hitelesítő adatokat.

> [!NOTE] 
> Legyen nagyon körültekintő beállítással kapcsolatos **engedélyezése – hitelesítő adatok** való **igaz**, mert azt jelenti, hogy egy webhely is küldhetnek a felhasználó hitelesítő adatait az API-t, a felhasználó nevében, nem a felhasználó nem is tud. Meg kell bíznia az engedélyezett forrása.

### <a name="enforce-https"></a>HTTPS kényszerítése

A maximális biztonság érdekében van szükség a kérelem folyamata során HTTPS:

- **CDN**. Az Azure CDN a HTTPS támogatja a `*.azureedge.net` altartomány alapértelmezés szerint. A CDN-t az egyéni tartománynevek a HTTPS engedélyezéséhez tekintse [oktatóanyag: HTTPS konfigurálása Azure CDN egyéni tartományon][cdn-https]. 

- **Statikus webhely üzemeltetése**. Engedélyezze a "[biztonságos átvitelre van szükség][storage-https]" lehetőséget a tárfiók. Ha ez a beállítás engedélyezve van, a storage-fiókot csak lehetővé teszi a biztonságos HTTPS-kapcsolatok érkező kérelmeket. 

- **Az API Management**. Az API-k csak a HTTPS protokoll használatára konfigurálja. Az Azure Portalon, vagy egy Resource Manager-sablon segítségével konfigurálható:

    ```json
    {
        "apiVersion": "2018-01-01",
        "type": "apis",
        "name": "dronedeliveryapi",
        "dependsOn": [
            "[concat('Microsoft.ApiManagement/service/', variables('apiManagementServiceName'))]"
        ],
        "properties": {
            "displayName": "Drone Delivery API",
            "description": "Drone Delivery API",
            "path": "api",
            "protocols": [ "HTTPS" ]
        },
        ...
    }
    ```

- **Az Azure Functions**. Engedélyezze a "[csak HTTPS][functions-https]" beállítást. 

### <a name="lock-down-the-function-app"></a>A függvényalkalmazás zárolását

A függvény összes hívást végre kell hajtania az API-átjáró. Akkor érhető el, a következőképpen:

- Egy függvény kulcs megkövetelése a függvényalkalmazás konfigurálása. Az API Management-átjáró a függvény kulcsot fogja tartalmazni, amikor meghívja a függvényalkalmazás. Ez megakadályozza, hogy az ügyfelek a függvény közvetlenül, az átjáró kihagyásával. 

- Az API Management gateway rendelkezik egy [statikus IP-cím][apim-ip]. Az Azure-függvényt, hogy csak a statikus IP-címről indított hívások korlátozása. További információkért lásd: [Azure App Service statikus IP-korlátozások][app-service-ip-restrictions]. (Ez a funkció csak a Standard szintű szolgáltatások esetében érhető el.) 

### <a name="protect-application-secrets"></a>Titkos alkalmazáskulcsok védelme

Ne tároljon az alkalmazások titkos adatait, például az adatbázis hitelesítő adatait, a kód vagy a konfigurációs fájlokban. Ehelyett használja az alkalmazásbeállításokat, amelyeket a rendszer titkosítja az Azure-ban. További információkért lásd: [biztonság az Azure App Service és az Azure Functions][app-service-security].

Azt is megteheti a titkos alkalmazáskulcsok tárolhatja a Key Vaultban. Ez lehetővé teszi, hogy a titkos kulcsok tárolására központosítása, a terjesztési, és hogyan és mikor rálátást titkos kulcsok figyelése. További információkért lásd: [konfigurálása az Azure-webalkalmazások titkos Key vault olvasásához][key-vault-web-app]. Vegye azonban figyelembe, hogy Functions eseményindítók és kötések betölteni a konfigurációs beállítások az alkalmazás beállításait. Nincs beépített mód konfigurálása az eseményindítók és kötések használata a Key Vault titkos kulcsok.

## <a name="devops-considerations"></a>Fejlesztési és üzemeltetési szempontok

### <a name="api-versioning"></a>API-verziószámozás

Egy API-ját egy szolgáltatás és -ügyfelek vagy ügyfél között. Verziókezelés támogatja az API-szerződés. Ha egy API-t használhatatlanná tévő változást, vezessen be egy új API-verzió. Helyezze üzembe az új verzió-párhuzamos egy külön függvényalkalmazásban az eredeti verzióval. Ez lehetővé teszi a meglévő ügyfelek áttelepítését az új API ügyfélalkalmazások megszakítása nélkül. Végül, a korábbi verziót is kivezetjük. API-k verziókezelése kapcsolatos további információkért lásd: [Versioning webes RESTful API][api-versioning].

API-módosítás ne sérüljenek frissítések központi telepítése az új verzió ugyanaz a Függvényalkalmazás az előkészítési pont. Ellenőrizze a telepítés sikeres volt, és majd felcserélheti a manuálisan előkészített verzió éles verziójával.

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

Ez a referenciaarchitektúra üzembe helyezéséhez keresse meg a [GitHub információs][readme]. 

<!-- links -->

[api-versioning]: ../../best-practices/api-design.md#versioning-a-restful-web-api
[apim]: /azure/api-management/api-management-key-concepts
[apim-ip]: /azure/api-management/api-management-faq#is-the-api-management-gateway-ip-address-constant-can-i-use-it-in-firewall-rules
[api-geo]: /azure/api-management/api-management-howto-deploy-multi-region
[apim-scale]: /azure/api-management/api-management-howto-autoscale
[app-service-auth]: /azure/app-service/app-service-authentication-overview
[app-service-ip-restrictions]: /azure/app-service/app-service-ip-restrictions
[app-service-security]: /azure/app-service/app-service-security
[ase]: /azure/app-service/environment/intro
[azure-messaging]: /azure/event-grid/compare-messaging-services
[claims]: https://en.wikipedia.org/wiki/Claims-based_identity
[cdn]: https://azure.microsoft.com/services/cdn/
[cdn-https]: /azure/cdn/cdn-custom-ssl
[cors-policy]: /azure/api-management/api-management-cross-domain-policies
[cosmosdb]: /azure/cosmos-db/introduction
[cosmosdb-geo]: /azure/cosmos-db/distribute-data-globally
[cosmosdb-input-binding]: /azure/azure-functions/functions-bindings-cosmosdb-v2#input
[cosmosdb-scale]: /azure/cosmos-db/partition-data
[event-driven]: ../../guide/architecture-styles/event-driven.md
[functions]: /azure/azure-functions/functions-overview
[functions-bindings]: /azure/azure-functions/functions-triggers-bindings
[functions-cold-start]: https://blogs.msdn.microsoft.com/appserviceteam/2018/02/07/understanding-serverless-cold-start/
[functions-https]: /azure/app-service/app-service-web-tutorial-custom-ssl#enforce-https
[functions-proxy]: /azure-functions/functions-proxies
[functions-scale]: /azure/azure-functions/functions-scale
[functions-timeout]: /azure/azure-functions/functions-scale#consumption-plan
[graph]: https://developer.microsoft.com/graph/docs/concepts/overview
[key-vault-web-app]: /azure/key-vault/tutorial-web-application-keyvault
[microservices-domain-analysis]: ../../microservices/domain-analysis.md
[monitor]: /azure/azure-monitor/overview
[oauth-flow]: https://auth0.com/docs/api-auth/which-oauth-flow-to-use
[partition-key]: /azure/cosmos-db/partition-data
[pipelines]: /azure/devops/pipelines/index
[ru]: /azure/cosmos-db/request-units
[static-hosting]: /azure/storage/blobs/storage-blob-static-website
[static-hosting-preview]: https://azure.microsoft.com/blog/azure-storage-static-web-hosting-public-preview/
[storage-https]: /azure/storage/common/storage-require-secure-transfer
[tm]: /azure/traffic-manager/traffic-manager-overview

[github]: https://github.com/mspnp/serverless-reference-implementation
[HttpRequestAuthorizationExtensions]: https://github.com/mspnp/serverless-reference-implementation/blob/master/src/DroneStatus/dotnet/DroneStatusFunctionApp/HttpRequestAuthorizationExtensions.cs
[readme]: https://github.com/mspnp/serverless-reference-implementation/blob/master/README.md