---
title: Content Delivery Network útmutató
description: Útmutatás a Content Delivery Network (CDN) Azure-ban üzemeltetett nagy sávszélesség-tartalom.
author: dragon119
ms.date: 02/02/2018
pnp.series.title: Best Practices
ms.openlocfilehash: 42b73db08ecef858f5279ea292cf8c0df77b847c
ms.sourcegitcommit: 29fbcb1eec44802d2c01b6d3bcf7d7bd0bae65fc
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/27/2018
---
# <a name="best-practices-for-using-content-delivery-networks-cdns"></a>Gyakorlati tanácsok a tartalomkézbesítési hálózat (tartalomtovábbító) használatát

Egy tartalomkézbesítési hálózat (CDN) egy olyan elosztott hálózati kiszolgálók hatékonyan által biztosított webes tartalom a felhasználók számára. Tartalomtovábbító gyorsítótárazott tartalom peremhálózati kiszolgálóinak hamarosan végfelhasználók számára, a késés csökkentése érdekében érdemes tárolni. 

Statikus tartalom, például a lemezképek, a stíluslapok, a dokumentumok, a ügyféloldali parancsprogramok és a HTML-lapok tartalomtovábbító általában használják. A CDN használatával főbb előnyei a következők: kisebb késést biztosít és a gyorsabb tartalom felhasználók földrajzi helytől függetlenül viszonyítva az adatközpontban Ha az alkalmazás üzemel. Tartalomtovábbító is segíthet a webes alkalmazás terhelésének csökkentése érdekében, mert az alkalmazás nem rendelkezik a tartalom a CDN futó érkező kérelmeket.
 
![CDN-diagramja](./images/cdn/CDN.png)

Az Azure a [Azure tartalom Delivery Network](/azure/cdn/cdn-overview) globális CDN megoldást van, az Azure vagy bármely más helyen található, amelyek tartalmak nagy sávszélességű kézbesítéséhez. Azure CDN használatával, az Azure blob Storage tárolóban, webalkalmazás, virtuális gép, bármely nyilvánosan elérhető webkiszolgáló betöltött nyilvánosan elérhető objektumok gyorsítótárazásához. 

Ez a témakör néhány általános, ajánlott eljárások és szempontokat a CDN használata esetén. Azure CDN használatával kapcsolatos további tudnivalókért lásd: [CDN dokumentáció](/azure/cdn/).

## <a name="how-and-why-a-cdn-is-used"></a>Miért és hogyan lehet a CDN használata

A CDN a jellemző használati többek között:  

* Az ügyfélalkalmazások, gyakran egy webhelyről statikus erőforrások kézbesítéséhez. Ezeket az erőforrásokat lehet képek, stíluslapok, dokumentumok, fájlok, ügyféloldali parancsfájlokat, HTML-oldalaihoz, HTML-töredék vagy bármely más, a kiszolgáló nem kell módosítani az egyes kérelmek tartalmat. Az alkalmazás létrehozása az elemek futási időben, és elérhetővé teszi azokat a CDN (például a jelenlegi híreket egy Alkalmazáslista létrehozásával), de azt nem az egyes kérelmek.
* Az eszközök, például mobiltelefon és táblagép nyilvános statikus és megosztott tartalom továbbítása. Magának az alkalmazásnak egy webszolgáltatás, amelyet az API-k kínál az ügyfelek számára a különböző eszközökön futó. A CDN-t is biztosíthat statikus adatkészletek (webszolgáltatás) keresztül az ügyfelek számára, lehet, hogy az ügyfél felhasználói felületének létrehozásához. Például a CDN használatával terjesztheti a JSON- vagy XML-dokumentumot.
* A számítási erőforrásokat szolgál teljes webhelyeket, amelyeket csak nyilvános statikus tartalom az ügyfelek számára, anélkül, hogy a dedikált állnak.
* Adatfolyam-továbbítási videofájlok az ügyfélnek az igény szerinti. Videó számos előnyt biztosít az alacsony késéssel és elérhető az globálisan található adatközpontokban, amelyek CDN kapcsolatot létesíteni a megbízható kapcsolatot. A Microsoft Azure Media Services (AMS) továbbítanak tartalmat a CDN későbbi terjesztés közvetlenül az Azure CDN integrálható. További információkért lásd: [Streaming végpontok áttekintése](/azure/media-services/media-services-streaming-endpoints-overview).
* A felhasználói élmény a felhasználók számára, különösen az adatközpontban, az alkalmazást futtató távol található általában javítása. Ezek a felhasználók más módon csökkenhet, nagyobb késleltetéssel járhat. Egy webes alkalmazás a tartalom teljes mérete nagy részét gyakran statikus, és a CDN használata segíthet fenntartásához, teljesítmény és általános felhasználói élmény, így kiküszöböli a követelmény több adatközpontokban az alkalmazás központi telepítése során. Azure CDN-csomóponti helyek listáját lásd: [Azure CDN POP-helyek](/azure/cdn/cdn-pop-locations/).
* (Az eszközök internetes hálózatát) az IoT-megoldások támogatása. Eszközök és az IoT-megoldás részt készülékek hatalmas mennyiségű sikerült könnyen ne terhelje tovább az alkalmazás Ha kellett továbbíthatja belső vezérlőprogram közvetlenül az egyes eszközök.
* Másolás csúcsait és emelkedéseit anélkül, hogy a skálázandó alkalmazás igény, elkerülve a csatlakozása következtében nőtt fenntartási költségeihez. Például amikor az operációs rendszer frissítése a hardveres eszköz, például egy adott modell útválasztó, vagy a felhasználói eszköz, például egy intelligens TV megjelenik, lesz hatalmas csúcs igény szerint a felhasználók és eszközök millióira által rövid idő alatt letöltés.

## <a name="challenges"></a>Kihívásai

Figyelembe kell venni a CDN használata tervezése során több akadályok merülnek fel.  

* **Központi telepítési**. Döntse el, a forrás, ahol a CDN lekéri a tartalmat, és hogy mikor szükséges a tartalom telepítése több tároló rendszerben. Vegye figyelembe a statikus tartalom és erőforrások telepítésének folyamata. Szükség lehet például egy külön megoldás a tartalom betöltése az Azure blob Storage tárolóban.
* **Verziókövetés és a cache-control**. Vegye figyelembe, hogyan fogja frissíteni a statikus tartalom, és új verziók helyezhetők üzembe. Megérteni, hogyan a CDN hajt végre, gyorsítótárazás és az idő-Élettartam (TTL). További Azure CDN: [gyorsítótárazás működése](/azure/cdn/cdn-how-caching-works).
* **Tesztelési**. A CDN-beállításokat, ha a fejlesztés és tesztelés az alkalmazás helyileg vagy tesztelési környezetben helyi teszteléséhez nehézkes lehet.
* **Keresés motor optimalizálási (keresőmotor-Optimalizáláshoz)**. Tartalom, például a képeket és a dokumentumok szolgáltatott más tartományokból a CDN használatakor. Ez lehet hatással a keresőmotor-Optimalizáláshoz ehhez a tartalomhoz.
* **Tartalom biztonsági**. Nem minden tartalomtovábbító ajánlatot bármely formájára vonatkozó hozzáférés-vezérlés a tartalomhoz. Egyes CDN szolgáltatások, Azure CDN, beleértve a jogkivonat-alapú hitelesítés CDN-tartalom védelméhez támogatja. További információkért lásd: [védelmét biztosító Azure Content Delivery Network eszközök token hitelesítéssel](/azure/cdn/cdn-token-auth).
* **Ügyfél biztonsági**. Az ügyfelek kapcsolódását az olyan környezetben, amely nem teszi lehetővé az erőforrásokhoz való hozzáférést a CDN a. Ennek oka lehet egy korlátozott biztonsági környezetben, amely korlátozza a hozzáférést csak ismert források, vagy egy, az erőforrások lapon eredeti csakis a betöltését. A tartalék megvalósításához szükség van kezelik az ilyen eseteket.
* **Rugalmasság**. A CDN egy potenciális hibaérzékeny pontot az alkalmazáshoz. 

Olyan esetekben, ahol CDN lehet, hogy kevesebb hasznos belefoglalása:  

* Ha a tartalom egy alacsony találati aránya, az elérhető lehet csak néhány alkalommal állapotában érvényes (határozza meg a live idő beállítása). 
* Ha az adatok személyes, többek között a nagyobb vállalatok vagy ellátási lánc ökoszisztéma.

## <a name="general-guidelines-and-good-practices"></a>Általános irányelvek és bevált gyakorlatok

A CDN használata minimálisra csökkentése az alkalmazás terhelését, és rendelkezésre állás és teljesítmény maximalizálása érdekében egy jó módszer. Vegye figyelembe, hogy ezt a stratégiát, az összes a megfelelő tartalom és az alkalmazás által használt erőforrások bevezetése. Vegye figyelembe az alábbi szakaszok a pontok, a CDN használata a stratégia tervezésekor.

### <a name="deployment"></a>Környezet
Statikus tartalom esetleg kiosztása és egymástól függetlenül az alkalmazás telepítve, ha az alkalmazás központi telepítési csomagot vagy a folyamat nem tartalmaznak. Vegye figyelembe, hogy ez milyen hatással a versioning mind az alkalmazás-összetevők és a statikus tartalom kezeléséhez használt módszert.

Vegye figyelembe, hogy kötegelése és minification technikák segítségével az ügyfelek a betöltési idők csökkentése érdekében. Több fájl kötegelése egyesít egyetlen fájlt. Minification eltávolítja felesleges karaktereket parancsfájlok és CSS fájlok funkció módosítása nélkül.

Ha a tartalom telepítése további helyre van szüksége, ez lesz a telepítési folyamat egy további lépést. Ha az alkalmazás számára a CDN-t frissíti a tartalmat, lehet, hogy rendszeres időközönként, vagy az adott esemény bekövetkezésekor kell tárolnia a frissített tartalom semmilyen további helyek, valamint a végpont a CDN a.

Vegye figyelembe, hogyan fogja kezelni a helyi fejlesztési és tesztelési, amikor néhány statikus tartalom várhatóan CDN helyről. A build script részeként például a CDN a tartalmat előre telepíthet. Használhat fordítási direktíváit vagy jelzők szabályozásához, hogy az alkalmazás betölti az erőforrások. Például hibakeresési módban, az alkalmazás tölthető statikus erőforrások egy helyi mappába. Az alkalmazás kiadási módban a CDN-t használja.

Vegye figyelembe a fájltömörítés, például a gzip (GNU zip) beállításait. Tömörítés hajthatja végre a forrás kiszolgálón a webes alkalmazás-gazdaszolgáltatás vagy közvetlenül a peremhálózati kiszolgálóinak a CDN-t. További információkért lásd: [javítása a fájlok tömörítése a Azure CDN](/azure/cdn/cdn-improve-performance).


### <a name="routing-and-versioning"></a>Az Útválasztás és versioning
CDN-példány használható a különböző időpontokban szeretne. Például az alkalmazás új verziójának telepítésekor érdemes lehet egy új CDN-t használja, és tartsa meg a régi CDN (tartalom okozó régebbi formátumú) korábbi verzióihoz. Ha a tartalom eredete használja az Azure blob Storage tárolóban, hozzon létre egy külön tárfiókot vagy egy külön tárolóba, és a CDN-végpont mutasson. 

Ne használja a lekérdezési karakterlánc jelölésére a CDN megtalálható erőforrásokhoz való hivatkozások az alkalmazás különböző verzióit, mert, amikor a tartalom lekérése az Azure blob storage, a lekérdezési karakterlánc része az erőforrás neve (a blob neve). Ezt a módszert használja szintén befolyásolhatják, hogy az ügyfél milyen módon gyorsítótárazza a erőforrások.

Statikus tartalom új verzióinak telepítését, ha egy alkalmazás módosítását kihívást lehet, ha az előző erőforrások gyorsítótárazza a CDN-t. További információk: gyorsítótár vezérlőn szakasz alatt.

Fontolja meg a CDN ország tartalom hozzáférés korlátozása. Az Azure CDN lehetővé teszi szűrheti a kéréseket a származási alapján, és korlátozza a tartalom letöltéséhez. További információkért lásd: [korlátozza a hozzáférést a tartalmat az ország](/azure/cdn/cdn-restrict-access-by-country/).

### <a name="cache-control"></a>Gyorsítótár-vezérlő
Vegye figyelembe a rendszerben gyorsítótárazás kezelése. Például az Azure CDN csak is globális gyorsítótárazási szabályokat állíthat be, és majd egyéni adott forrás végpontok gyorsítótárazását. Azt is meghatározhatja, hogyan gyorsítótárazását végzi el a CDN a gyorsítótár-irányelv fejlécek küldését a forráson. 

További információkért lásd: [gyorsítótárazás működése](/azure/cdn/cdn-how-caching-works).

Objektumok elérhetőnek kell lennie a CDN elkerülése érdekében törölje ezeket a forrásból, eltávolíthatja vagy törölni a CDN-végpontot, vagy a blob storage esetén ellenőrizze a tároló vagy blob személyes. Azonban elemek nem lesznek eltávolítva az az idő TTL eléréséig. A CDN-végpontok manuálisan is törölheti.

### <a name="security"></a>Biztonság

A CDN által biztosított tartalmat, keresztül HTTPS (SSL), a CDN által biztosított tanúsítvány használatával, valamint szabványos HTTP-kapcsolaton keresztül. Annak érdekében, ne böngésző vegyes tartalomról, szükség lehet a HTTPS használatával betöltött lapok statikus tartalmat igénylő a HTTPS protokoll használatával.

Ha az eszközök statikus betűtípus fájlok például a CDN használatával, problémák léphetnek fel azonos eredetű házirend használatakor egy *XMLHttpRequest* hívás ezekkel az erőforrásokkal kérhet egy másik tartományban. Sok webböngészővel eltérő eredetű erőforrások megosztása (CORS), kivéve, ha a webkiszolgáló úgy van beállítva, a megfelelő válaszfejlécek beállítása megakadályozza. Beállíthatja, hogy a CDN és a CORS támogatja az alábbi módszerek egyikének használatával:

* A CORS fejlécek hozzáadása a válaszokat a CDN konfigurálásához. További információkért lásd: [az Azure CDN szolgáltatás használata a CORS](/azure/cdn/cdn-cors). 
* Ha a forrás az Azure blob Storage tárolóban, vegye fel a CORS-szabályokat a storage-végponthoz. További információkért lásd: [Cross-Origin Resource Sharing (CORS) támogatása az Azure Storage szolgáltatásainak](http://msdn.microsoft.com/library/azure/dn535601.aspx).
* Konfigurálja a CORS fejlécek beállítása az alkalmazást. Lásd például: [engedélyezi eltérő eredetű kérések (CORS)](/aspnet/core/security/cors) az ASP.NET Core dokumentációjában.

### <a name="cdn-fallback"></a>CDN tartalék
Vegye figyelembe, hogyan az alkalmazás fogja a folyamatosan növekvő adatmennyiségnek hiba vagy ideiglenes hiányában a CDN-t. Előfordulhat, hogy ügyfélalkalmazások másolja a helyileg (az ügyfél) a gyorsítótárazott erőforrások használhatják az előző kérelem során, vagy megadhat kódot, amely hibát észlel, és helyette a forrásból (az alkalmazás mappájában, vagy az Azure blob-erőforrások kérelmek az tároló, amely az erőforrások) Ha a CDN nem érhető el.
