---
title: Útmutató a Content Delivery Networkhöz
titleSuffix: Best practices for cloud applications
description: Útmutató a Content Delivery Networköt (CDN) használata az Azure-ban üzemeltetett nagy sávszélességű tartalmak továbbítására.
author: dragon119
ms.date: 02/02/2018
ms.topic: best-practice
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: f4ffe9c5cdd7a53ab8359ef303076c5e53e9e45c
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299570"
---
# <a name="content-delivery-networks-cdns"></a>Tartalomkézbesítési hálózatok (CDN)

Egy tartalomkézbesítési hálózat (CDN) kiszolgálók olyan elosztott hálózata, amely hatékonyan kézbesíti a webes tartalmakat a felhasználóknak. A CDN-ek a késés minimalizálása érdekében gyorsítótárazott tartalmat tárolnak végfelhasználók közelében lévő peremhálózati kiszolgálókon.

A CDN-ek segítségével általában statikus tartalmak, például képek, stíluslapok, dokumentumok, ügyféloldali szkriptek vagy HTML-oldalak továbbíthatók. A CDN használatának főbb előnyei a következők: kisebb késés, gyorsabb tartalomkézbesítés a felhasználóknak függetlenül attól, hogy a földrajzi helyük mennyire van közel az alkalmazást üzemeltető adatközponthoz. A CDN segítségével a webalkalmazás terhelése is csökkenthető, mert az alkalmazásnak nem kell a CDN-en üzemeltetett tartalmakra vonatkozó kéréseket kiszolgálnia.

![CDN diagram](./images/cdn/CDN.png)

Az Azure-ban az [Azure Content Delivery Network](/azure/cdn/cdn-overview) egy globális CDN-megoldás az Azure-ban vagy más helyeken üzemeltetett nagy sávszélességű tartalmak továbbítására. Az Azure CDN használatával lehetőség van a nyilvánosan elérhető, az Azure Blob-tárolóból, webalkalmazásokból, virtuális gépekről vagy bármilyen nyilvánosan elérhető webkiszolgálóról betöltött objektumok gyorsítótárazására.

Ez a témakör néhány, a CDN használata esetén általánosan ajánlott eljárást és szempontot ismertet. További információkért lásd: [Azure CDN](/azure/cdn/).

## <a name="how-and-why-a-cdn-is-used"></a>Hogyan és miért érdemes használni a CDN-t

A CDN tipikus alkalmazásai:  

- Statisztikai erőforrások biztosítása ügyfélalkalmazásokhoz, gyakran egy webhelyről. Ezek az erőforrások lehetnek képek, stíluslapok, dokumentumok, fájlok, ügyféloldali szkriptek, HTML-lapok, HTML-töredékek vagy bármilyen tartalom, amelyet a kiszolgálónak nem kell módosítania az egyes kéréseknél. Az alkalmazás a futásidőben létrehozhat elemeket, és elérhetővé teheti azokat a CDN számára (például létrehoz egy listát az aktuális hírekről), de azt nem teszi meg minden kérelemnél.

- Nyilvános statikus és megosztott tartalom biztosítása például mobiltelefonok, táblagépek és hasonló eszközök számára. Maga az alkalmazás egy szolgáltatás, amely API-t biztosít különböző eszközökön futó ügyfeleknek. A CDN a webes szolgáltatáson keresztül statikus adatkészleteket is biztosít az ügyfeleknek, például az ügyféloldali felhasználói felület létrehozásához. A CDN használható például JSON- vagy XML-dokumentumok terjesztésére.

- Egész, csak nyilvános statikus tartalomból álló webhelyek kiszolgálása ügyfeleknek dedikált számítási erőforrások igénye nélkül.

- Videofájlok streamelése az ügyfélnek igény szerint. A videók szempontjából előnyös a CDN-kapcsolatot biztosító, globálisan elhelyezett adatközpont által biztosított alacsony késés és megbízható kapcsolat. A Microsoft Azure Media Services (AMS) az Azure CDN-t integrálva szolgáltat tartalmat közvetlenül a CDN-nek további terjesztés céljából. További információk: [Streamvégpontok áttekintése](/azure/media-services/media-services-streaming-endpoints-overview).

- Általában jobb felhasználói élmény, különösen az alkalmazást üzemeltető adatközponttól távol található felhasználók esetében. Ezek a felhasználók más esetben esetleg nagyobb késést tapasztalhatnának. A webalkalmazásokban gyakran a tartalom teljes méretének nagy része statikus, és a CDN használata segíthet fenntartani a teljesítményt és az általános felhasználói élményt úgy is, hogy nem kell az alkalmazást több adatközpontban üzembe helyezni. Az Azure CDN-csomópontok helyeinek listája: [Azure CDN POP-helyek](/azure/cdn/cdn-pop-locations/).

- IoT- (Eszközök internetes hálózata) megoldások támogatása. Az IoT-megoldásokba bevont nagy számú eszköz és berendezés könnyen túlterhelhetne egy alkalmazást, ha a firmware-frissítéseket közvetlenül kéne terjesztenie minden eszközre.

- A csúcspontok és hirtelen kiugrások kezelése igény szerint anélkül, hogy skálázni kellene az alkalmazást, így elkerülhetők a skálázás miatti megnövekedett futtatási költségek. Amikor például megjelenik egy operációsrendszer-frissítés egy hardvereszközhöz, például egy adott routermodellhez, vagy egy olyan fogyasztói eszközhöz, mint egy okostelevízió, nagy igénycsúcs várható, hiszen felhasználók és eszközök milliói töltik azt le rövid idő alatt.

## <a name="challenges"></a>Problémák

A CDN használatának tervezésekor több esetleges problémát is figyelembe kell venni.

- **Üzembe helyezés**. Határozza meg a forrást, ahonnan a CDN lekéri a tartalmat, és azt, hogy telepítenie kell-e a tartalmat egynél több tárolórendszerben. Vegye figyelembe a statikus tartalom és erőforrások üzembe helyezésének folyamatát. Előfordulhat például, hogy implementálnia kell egy külön lépést a tartalom Azure Blob Storage-ba való betöltéséhez.

- **Verziókezelés és gyorsítótár-vezérlés**. Gondolja át, hogyan fogja frissíteni a statikus tartalmat, és hogyan fog új verziókat telepíteni. Ismerje meg a CDN gyorsítótárazásának működését és az élettartam (TTL) kezelését. Az Azure CDN esetében tekintse meg [A gyorsítótárazás működése](/azure/cdn/cdn-how-caching-works) című szakaszt.

- **Tesztelés**. A CDN-beállítások helyi tesztelése nehézkes lehet, ha helyben vagy átmeneti környezetben zajlik az alkalmazás fejlesztése és tesztelése.

- **Keresőmotor-optimalizálás (SEO)**. CDN használata esetében a képek, dokumentumok és a hasonló tartalmak kiszolgálása különböző tartományból történik. Ez hatással lehet a tartalomhoz tartozó keresőmotor-optimalizálásra.

- **Tartalombiztonság**. Nem minden CDN biztosít hozzáférés-vezérlést a tartalomhoz. Bizonyos CDN-szolgáltatások (például az Azure CDN) támogatják a jogkivonat-alapú hitelesítést a CDN-tartalom védelme érdekében. További információk: [Az Azure Content Delivery Network objektumok biztosítása jogkivonat-alapú hitelesítéssel](/azure/cdn/cdn-token-auth).

- **Ügyfélbiztonság**. Az ügyfelek kapcsolódhatnak olyan környezetből, amely nem teszi lehetővé a CDN-en lévő erőforrásokhoz való hozzáférést. Ez lehet egy biztonsági okból korlátozott környezet, amely csupán néhány ismert forráshoz enged hozzáférést, vagy egy olyan, amely a lap forrását kivéve mindenhonnan megakadályozza az erőforrások betöltését. Az ilyen esetek kezeléséhez szükséges egy tartalék megvalósítás.

- **Rugalmasság**. A CDN egy potenciálisan hibaérzékeny pontja az alkalmazásoknak.

Forgatókönyvek, ahol a CDN nem túl hasznos megoldás:  

- Ha a tartalom találati aránya alacsony, előfordulhat, hogy az érvényessége során (ezt az alapértelmezett élettartamra vonatkozó beállítás határozza meg) csupán párszor használhatják.

- Ha az adat bizalmas, például nagyvállalatoknál vagy ellátási lánccal kapcsolatos ökoszisztémáknál.

## <a name="general-guidelines-and-good-practices"></a>Általános irányelvek és bevált gyakorlatok

A CDN használata kiváló módszer az alkalmazások terhelésének minimalizálására, valamint a lehető legmagasabb szintű rendelkezésre állás és teljesítmény elérésére. Érdemes lehet ezt a stratégiát alkalmazni az alkalmazás által használt összes megfelelő tartalomhoz és erőforráshoz. Amikor megtervezi a CDN-használati stratégiáját, vegye figyelembe az alábbi szempontokat.

### <a name="deployment"></a>Környezet

Előfordulhat, hogy a statikus tartalmat az alkalmazástól függetlenül kell létrehozni és telepíteni, ha nem vonja be az alkalmazástelepítési csomagba vagy folyamatba. Gondolja végig, milyen hatással lesz ez az alkalmazás összetevői és a statikus erőforrás-tartalom kezeléséhez használt verziókezelési stratégiára.

Érdemes lehet kötegelési és kicsinyítési technikákat alkalmazni az ügyfeleknél tapasztalható betöltési idő csökkentése érdekében. A kötegelés több fájlt egyesít egy fájlba. A kicsinyítés eltávolítja a szükségtelen karaktereket a szkriptekből és a CSS-fájlokból, de a működőképességet ez nem befolyásolja.

Ha a tartalmat egy további helyen is üzembe kell helyeznie, ez egy extra lépés lesz az üzembe helyezési folyamatban. Ha az alkalmazás frissíti a CDN tartalmát, például rendszeres időközönként vagy egy eseményre válaszként, a frissített tartalmat minden további helyen és a CDN végpontján is tárolnia kell.

Gondolja át, hogyan fogja kezelni a helyi fejlesztést és tesztelést, ha valamely statikus tartalom kiszolgálása egy CDN-ről várható. Megteheti például, hogy előre telepíti a tartalmat a CDN-re a buildszkript részeként. Másik megoldásként használhat fordítási irányelveket vagy jelzőket annak szabályozására, hogy hogyan töltse be az alkalmazás az erőforrásokat. Hibakeresési módban például az alkalmazás be tudná tölteni a statikus erőforrásokat egy helyi mappából. Kiadási módban az alkalmazás a CDN-t használná.

Gondolja végig a fájltömörítési lehetőségeket, például a gzip (GNU zip) formátumot. A tömörítés végrehajtható a forráskiszolgálón a webalkalmazás üzemeltetésével vagy közvetlenül a peremhálózati kiszolgálókon a CDN segítségével. További információk: [A teljesítmény javítása a fájlok tömörítésével az Azure CDN-ben](/azure/cdn/cdn-improve-performance).

### <a name="routing-and-versioning"></a>Útválasztás és verziókezelés

Lehetséges, hogy különböző időpontokban különböző CDN-példányokat kell használnia. Előfordulhat például, hogy az alkalmazás új verziójának telepítésekor egy új CDN-t szeretne használni, a korábbi verziókhoz viszont meg szeretné tartani a régi CDN-t (amely régebbi formátumban tárolja a tartalmat). Ha a tartalom forrásának az Azure Blob Storage-t használja, létrehozhat egy külön tárfiókot vagy egy külön tárolót, és a CDN-végpontot irányíthatja arra.

A CDN-en lévő erőforrásokra mutató hivatkozásokban ne használja a lekérdezési sztringet az alkalmazás különböző verzióinak jelölésére, mert amikor az Azure Blob Storage-ból kér le tartalmat, a lekérdezési sztring része az erőforrás nevének (a blobnévnek). Ez a megközelítés arra is hatással lehet, hogy hogyan gyorsítótárazza az ügyfél az erőforrásokat.

A statikus tartalom új verzióinak telepítése egy alkalmazás frissítésekor kihívást jelenthet, ha a korábbi erőforrások a CDN-en lettek gyorsítótárazva. További információkért tekintse meg alább a gyorsítótár-vezérlésre vonatkozó szakaszt.

Érdemes lehet országonként korlátozni a CDN tartalmának elérését. Az Azure CDN lehetővé teszi a kérések szűrését a forrás országa alapján, és a kézbesített tartalom korlátozását. További információkat [a tartalom elérésének országonként történő korlátozásának](/azure/cdn/cdn-restrict-access-by-country/) ismertetésében talál.

### <a name="cache-control"></a>Gyorsítótár-vezérlés

Gondolja át, hogyan működjön a gyorsítótárazás a rendszerében. Az Azure CDN-ben például beállíthat globális gyorsítótárazási szabályokat, majd megadhat egyéni gyorsítótárazási beállításokat adott forrásvégpontokhoz. A forrásnál lévő gyorsítótárirányelv-fejlécek küldésével azt is szabályozhatja, hogy hogyan menjen végbe a gyorsítótárazás egy CDN-ben.

További információkért lásd: [A gyorsítótárazás működése](/azure/cdn/cdn-how-caching-works).

Ha azt szeretné, hogy bizonyos objektumok ne legyenek elérhetők a CDN-en, törölheti őket a forrásról, eltávolíthatja vagy törölheti a CDN-végpontot, illetve Blob Storage esetén priváttá teheti a tárolót vagy blobot. Az elemek azonban az alapértelmezett élettartam lejártáig nem törlődnek. A CDN-végpontok manuálisan is véglegesen törölhetők.

### <a name="security"></a>Biztonság

A CDN a CDN által kiállított tanúsítvány használatával képes HTTPS (SSL) protokollon keresztül is kézbesíteni tartalmat, és standard HTTP-n keresztül is. A böngészőben megjelenő figyelmeztetések és a vegyes tartalom elkerülése érdekében érdemes HTTPS protokollon keresztül elérnie a HTTPS-en keresztül betöltött oldalakon megjelenő statikus tartalmat.

Ha statikus objektumokat (pl. betűkészletfájlokat) kézbesít a CDN használatával, előfordulhat, hogy az azonos eredethez kapcsolódó szabályzatproblémák merülnek fel, ha *XMLHttpRequest* hívást használ ezen erőforrások különböző tartományból való lekérésére. Sok webböngésző nem engedélyezi az eltérő eredetű erőforrások megosztását (CORS), ha a webkiszolgáló nincs a megfelelő válaszfejlécek beállítására konfigurálva. A CDN-t beállíthatja a CORS támogatására az alábbi módszerek egyikével:

- A CDN-t konfigurálhatja arra, hogy CORS fejléceket adjon a válaszokhoz. További információk: [Az Azure CDN használata a CORS-szal](/azure/cdn/cdn-cors).

- Ha a forrás az Azure Blob Storage-ban van, hozzáadhat CORS-szabályokat a tároló végpontjához. További információk: [Az eltérő eredetű erőforrás-megosztás (CORS) támogatása az Azure Storage szolgáltatásokhoz](/rest/api/storageservices/Cross-Origin-Resource-Sharing--CORS--Support-for-the-Azure-Storage-Services).

- Konfigurálja az alkalmazást a CORS fejlécek beállítására. Lásd a [engedélyezi eltérő eredetű kérelmek (CORS)](/aspnet/core/security/cors) az ASP.NET Core dokumentációjában.

### <a name="cdn-fallback"></a>CDN-tartalék

Gondolja át, hogyan fog kezelni az alkalmazása egy hibát vagy a CDN átmeneti elérhetetlenségét. Előfordulhat, hogy ügyfélalkalmazások képesek a korábbi kérések során helyben (az ügyfélen) gyorsítótárazott erőforrások másolatainak használatára, vagy szerepeltethet olyan kódot, amely észleli a meghibásodást, és ha a CDN nem érhető el, inkább a forrásról kér erőforrásokat (az erőforrásokat tároló alkalmazásmappából vagy Azure Blob-tárolóról).
