---
title: "Content Delivery Network útmutató"
description: "Útmutatás a Content Delivery Network (CDN) Azure-ban üzemeltetett nagy sávszélesség-tartalom."
author: dragon119
ms.date: 09/30/2016
pnp.series.title: Best Practices
ms.openlocfilehash: 94036c803552d5e7061f99e6dd0ca9e563a32690
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="content-delivery-network"></a>Content Delivery Network
[!INCLUDE [header](../_includes/header.md)]

A Microsoft Azure tartalom Delivery Network (CDN) az Azure vagy bármely más helyen tárolt tartalmak nagy sávszélességű kézbesítéséhez globális megoldást kínál a fejlesztők. CDN használatával gyorsítótárazhatja a nyilvánosan elérhető objektumok betöltődnek az Azure blob Storage tárolóban, webalkalmazás, virtuális gép, alkalmazásmappa vagy más HTTP/HTTPS-hely. A CDN-gyorsítótár stratégiai helyét adja meg a maximális sávszélesség kézbesíteni a tartalmakat a felhasználók is kell tartani. A CDN tipikus felhasználási területe a statikus tartalmak, képek, stíluslapok, dokumentumok, fájlok, az ügyféloldali parancsprogramok és HTML-lapok például kézbesítéséhez.

Akkor is használható a CDN a gyorsítótár a dinamikus tartalom, például a PDF-jelentés vagy a diagramon a megadott bemeneti adatok; alapján Ha a különböző felhasználók által biztosított az ugyanazon bemeneti értékeket az eredmény a azonosnak kell lennie.

A CDN használatával főbb előnyei a következők: kisebb késést biztosít, és a felhasználók számára, függetlenül a földrajzi hely viszonyítva az adatközpontban Ha az alkalmazás üzemel tartalom gyorsabb kézbesítését.  

![CDN-diagramja](./images/cdn/CDN.png)

A CDN-t használó alkalmazás terhelésének csökkentésére, mert szükséges eléréséhez, és a tartalom feldolgozása mentesített is segítségül szolgálhat. A terhelés csökkenése segítségével növelése teljesítményét és méretezhetőségét, az alkalmazás, valamint csökkenti a teljesítményt és rendelkezésre állást valamilyen konkrét szintje eléréséhez szükséges egyéb feldolgozási erőforrás üzemeltetési költségek minimalizálása érdekében.

## <a name="how-and-why-a-cdn-is-used"></a>Miért és hogyan lehet a CDN használata
A CDN a jellemző használati többek között:  

* Az ügyfélalkalmazások, gyakran egy webhelyről statikus erőforrások kézbesítéséhez. Ezeket az erőforrásokat lehet képek, stíluslapok, dokumentumok, fájlok, ügyféloldali parancsfájlokat, HTML-oldalaihoz, HTML-töredék vagy bármely más, a kiszolgáló nem kell módosítani az egyes kérelmek tartalmat. Az alkalmazás létrehozása az elemek futási időben, és elérhetővé teszi azokat a CDN (például a jelenlegi híreket egy Alkalmazáslista létrehozásával), de azt nem az egyes kérelmek.
* Az eszközök, például mobiltelefon és táblagép nyilvános statikus és megosztott tartalom továbbítása. Magának az alkalmazásnak egy webszolgáltatás, amelyet az API-k kínál az ügyfelek számára a különböző eszközökön futó. A CDN-t is biztosíthat statikus adatkészletek (webszolgáltatás) keresztül az ügyfelek számára, lehet, hogy az ügyfél felhasználói felületének létrehozásához. Például a CDN használatával terjesztheti a JSON- vagy XML-dokumentumot.
* A számítási erőforrásokat szolgál teljes webhelyeket, amelyeket csak nyilvános statikus tartalom az ügyfelek számára, anélkül, hogy a dedikált állnak.
* Adatfolyam-továbbítási videofájlok az ügyfélnek az igény szerinti. Videó számos előnyt biztosít az alacsony késéssel és elérhető az globálisan található adatközpontokban, amelyek CDN kapcsolatot létesíteni a megbízható kapcsolatot. A Microsoft Azure Media Services (AMS) továbbítanak tartalmat a CDN későbbi terjesztés közvetlenül az Azure CDN integrálható. További információkért lásd: [Streaming végpontok áttekintése](/azure/media-services/media-services-streaming-endpoints-overview).
* A felhasználói élmény a felhasználók számára, különösen az adatközpontban, az alkalmazást futtató távol található általában javítása. Ezek a felhasználók más módon csökkenhet, nagyobb késleltetéssel járhat. Egy webes alkalmazás a tartalom teljes mérete nagy részét gyakran statikus, és a CDN használata segíthet fenntartásához, teljesítmény és általános felhasználói élmény, így kiküszöböli a követelmény több adatközpontokban az alkalmazás központi telepítése során.
* (Az eszközök internetes hálózatát) az IoT-megoldások támogató alkalmazások a növekvő terhelés kezelésére. Az ilyen eszközök és az érintett készülékek hatalmas mennyiségű sikerült könnyen ne terhelje tovább az alkalmazás Ha kellett szórási üzenetek feldolgozásához, és a belső vezérlőprogram frissítési terjesztési közvetlenül az egyes eszközök kezeléséhez.
* Másolás csúcsait és emelkedéseit anélkül, hogy a skálázandó alkalmazás igény, elkerülve a csatlakozása következtében nőtt fenntartási költségeihez. Például amikor az operációs rendszer frissítése a hardveres eszköz, például egy adott modell útválasztó, vagy a felhasználói eszköz, például egy intelligens TV megjelenik, lesz hatalmas csúcs igény szerint a felhasználók és eszközök millióira által rövid idő alatt letöltés.

Az alábbi listában láthatók példák a Közepes idő az első bájtig eltelt különböző földrajzi helyekről. A cél webes szerepkör a rendszer Azure nyugati Velünk. Egy erős korrelációs nagyobb program miatt a CDN és a CDN csomóponthoz közelségi kapcsolat között van. Azure CDN-csomóponti helyek teljes listája megtalálható [Azure Content Delivery Network (CDN) csomóponti helyek](/azure/cdn/cdn-pop-locations/).

|  | (Forrás) első bájtig eltelt ideje (ms) | Idő (ms), első (CDN) | % CDN idő fokozása |
| --- | --- | --- | --- |
| \*San Jose, CA |47.5 |46.5 |2% |
| \*\*Dulles, VA |109 |40.5 |169% |
| Buenos Aires, AR |210 |151 |39% |
| \*Londoni, UK |195 |44 |343% |
| Shanghai, CN |242 |206 |17% |
| \*Szingapúr |214 |74 |189% |
| \*Tokió, JP |163 |48 |204% |
| Szöul, KR |190 |190 |0% |

\*Az Azure CDN csomóponttal rendelkező ugyanabban a városban.  
\*\*Az Azure CDN csomóponttal rendelkező szomszédos városban.  

## <a name="challenges"></a>Kihívásai
Figyelembe kell venni a CDN használata tervezése során több akadályok merülnek fel:  

* **Központi telepítési**. Döntse el, a forrás, ahol a CDN lekéri a tartalmat, és hogy mikor szükséges központi telepítése a tartalom több tároló rendszerrel (például a CDN és a másik helyet).

  Az alkalmazástelepítési mechanizmussal statikus tartalomra és az erőforrások telepítése, valamint a központi telepítése az alkalmazásfájlok, például a ASPX lapok folyamat során figyelembe kell venni. Szükség lehet például egy külön megoldás a tartalom betöltése az Azure blob Storage tárolóban.
* **Verziókövetés és a cache-control**. Vegye figyelembe, hogyan fogja frissíteni a statikus tartalom, és új verziók helyezhetők üzembe. Lehet, hogy a CDN-tartalom [kiürítve](/azure/cdn/cdn-purge-endpoint/) az Azure portál használatával, ha az eszközök új verziói érhetők el. Ez hasonló bizonyulhat kezelésének ügyféloldali gyorsítótárazás, például, amely egy webböngészőben következik be.
* **Tesztelési**. A CDN-beállításokat, ha a fejlesztés és tesztelés az alkalmazás helyileg vagy tesztelési környezetben helyi teszteléséhez nehézkes lehet.
* **Keresés motor optimalizálási (keresőmotor-Optimalizáláshoz)**. Tartalom, például a képeket és a dokumentumok szolgáltatott más tartományokból a CDN használatakor. Ez lehet hatással a keresőmotor-Optimalizáláshoz ehhez a tartalomhoz.
* **Tartalom biztonsági**. Például az Azure CDN számos CDN szolgáltatás jelenleg nem képes bármilyen típusú hozzáférés-vezérlés a tartalomhoz.
* **Ügyfél biztonsági**. Az ügyfelek kapcsolódását az olyan környezetben, amely nem teszi lehetővé az erőforrásokhoz való hozzáférést a CDN a. Ennek oka lehet egy korlátozott biztonsági környezetben, amely korlátozza a hozzáférést csak ismert források, vagy egy, az erőforrások lapon eredeti csakis a betöltését. A tartalék megvalósításához szükség van kezelik az ilyen eseteket.
* **Rugalmasság**. A CDN egy potenciális hibaérzékeny pontot az alkalmazáshoz. Egy alacsonyabb rendelkezésre állási SLA-t, mint a blob-tároló (amely használható a tartalmat közvetlenül továbbít) úgy van szükség lehet egy tartalék mechanizmus kritikus tartalom vegye fontolóra.

  Figyelheti a CDN-tartalom hozzáférhetőségét, a sávszélesség, az átvitt adatok, a találatok, a gyorsítótár találati arány és gyorsítótár metrikák az Azure-portálról [valós idejű](/azure/cdn/cdn-real-time-stats/) és [összesítő jelentések](/azure/cdn/cdn-analyze-usage-patterns/).

Olyan esetekben, ahol CDN lehet, hogy kevesebb hasznos belefoglalása:  

* Ha a tartalom egy alacsony találati aránya, az elérhető lehet csak néhány alkalommal állapotában érvényes (határozza meg a live idő beállítása). Egy elem letöltődik először az Ön tudomásával két tranzakciós költségek és a CDN a forrásból, majd a CDN és az ügyfél.
* Ha az adatok személyes, többek között a nagyobb vállalatok vagy ellátási lánc ökoszisztéma.

## <a name="general-guidelines-and-good-practices"></a>Általános irányelvek és bevált gyakorlatok
A CDN használata minimálisra csökkentése az alkalmazás terhelését, és rendelkezésre állás és teljesítmény maximalizálása érdekében egy jó módszer. Vegye figyelembe, hogy ezt a stratégiát, az összes a megfelelő tartalom és az alkalmazás által használt erőforrások bevezetése. A CDN használata a stratégia tervezésekor, vegye figyelembe a következő szakaszokban pontok:  

### <a name="origin"></a>Forrás
A CDN a tartalom központi telepítése egyszerűen csak adja meg, hogy a CDN szolgáltatás használni fog eléréséhez, és a tartalom gyorsítótárazása HTTP vagy HTTPS végpont.

A végpont adhatja meg az Azure blob storage tárolót, amely tárolja a CDN keresztül szállítani szeretné a statikus tartalmat. A tároló nyilvánosként kell megjelölni. Nyilvános tárolókban lévő csak a nyilvános olvasási hozzáféréssel rendelkező blobot a CDN keresztül érhetők el.

A végpont adhat meg egy nevű mappát **cdn** gyökérkönyvtárában található egyik alkalmazás számítási rétegek (például egy webes szerepkör vagy egy virtuális gép). Az eredmények a kérelmek az erőforrások dinamikus erőforrások, például a ASPX lapok, beleértve a gyorsítótárban tartja a CDN-t. A minimális gyorsítótár időszak: 300 másodperc. Bármely rövidebb időszak megakadályozza, hogy a tartalom a CDN telepített (további információkért tekintse meg a következő fejléc *vezérlő gyorsítótár* alatt).

Ha Azure Web Apps használ, a végpont értéke a gyökérmappában található azon a helyen a hely kiválasztásával a CDN példányának létrehozásakor. A hely tartalmát mindegyikét a CDN érhető el.

A legtöbb esetben az egyik számítási az alkalmazás egy mappában a CDN-végpont mutat kínál nagyobb rugalmasságot és vezérlő. Például egyszerűbb jelenlegi és jövőbeli útválasztási követelmények kezeléséhez, és dinamikusan generálni a statikus tartalom, például a lemezkép miniatűrökön.

Használhat [lekérdezési karakterláncok](/azure/cdn/cdn-query-string/) megkülönböztetni azokat a gyorsítótárban objektumok, ha a tartalom kézbesítésével dinamikus forrásokból, például a ASPX lapok. Azonban ez a viselkedés letiltható beállítása az Azure portálon, ha megadja a CDN-végpontot. Tartalomtovábbítás blobtárolóból, amikor lekérdezési karakterláncok tekintendők karakterlánc-literálnak, a két elemet, azonos nevű, de eltérő lekérdezési karakterláncokkal rendelkezik, külön elemeket a CDN tárolódnak.

URL-címet írja át a erőforrások, például parancsfájlokat és más tartalmak, a fájlok áthelyezésével a CDN forrásmappát elkerülése érdekében úgy használhatja.

Az Azure storage blobs használata a CDN a tartalom tárolásához, a blobok erőforrások URL-címe esetén a tároló és a blob neve a kis-és nagybetűket.

Egyéni források vagy Azure Web Apps használatakor forrásokra mutató hivatkozások adja meg a CDN-példány elérési útja. Például a következő megadja a képfájl a **képek** mappát a hely, amely a CDN kézbesíti:

```XML
<img src="http://[your-cdn-endpoint].azureedge.net/Images/image.jpg" />
```

### <a name="deployment"></a>Környezet
Statikus tartalom esetleg kiosztása és egymástól függetlenül az alkalmazás telepítve, ha az alkalmazás központi telepítési csomagot vagy a folyamat nem tartalmaznak. Vegye figyelembe, hogy ez milyen hatással a versioning mind az alkalmazás-összetevők és a statikus tartalom kezeléséhez használt módszert.

Vegye figyelembe, hogy kötegelése (több fájlok egyesítése egy fájlba) és a parancsfájl és a CSS-fájlok (eltávolítja a szükségtelen például szóköz, új sor karaktereket, megjegyzések és egyéb karakterek) minification kezelésének módját. Ezek a gyakran használt módszerek, amely a betöltési idők csökkentheti az ügyfelek számára, és kompatibilisek-e a CDN a tartalom továbbítása. További információkért lásd: [Bundling és Minification](http://www.asp.net/mvc/tutorials/mvc-4/bundling-and-minification).

Ha a tartalom telepítése további helyre van szüksége, ez lesz a telepítési folyamat egy további lépést. Ha az alkalmazás számára a CDN-t frissíti a tartalmat, lehet, hogy rendszeres időközönként, vagy az adott esemény bekövetkezésekor kell tárolnia a frissített tartalom semmilyen további helyek, valamint a végpont a CDN a.

A CDN-végpont a helyi Azure emulátorban, a Visual Studio alkalmazás nem tudja beállítani. Ez a korlátozás egységek tesztelése, működési tesztelése és végső központi telepítés előtti tesztelés hatással lesz. Engedélyeznie kell a implementálásával alternatív módszert. Például akkor képes előtelepítése a tartalom és a CDN egy egyéni alkalmazást vagy segédprogram segítségével, és hajtsa végre a tesztelés során az időszakot, amelyben vannak gyorsítótárazva. Másik megoldásként használhatja a fordítási direktíváit vagy globális állandók szabályozhatja a, ahol az alkalmazás betölti az erőforrásokat. Például hibakeresési módban történő futtatásakor az sikerült betölteni az erőforrások, például ügyféloldali – csomagok és más tartalmak egy helyi mappából, és a CDN használata a kiadási módban történő futtatásakor.

Vegye figyelembe, hogy a CDN támogatásához kívánt tömörítési módszer kiválasztásához:

* Is [tömörítésének engedélyezéséhez](/azure/cdn/cdn-improve-performance/) a forrás kiszolgálón, ahol eset a CDN tömörítési alapértelmezés szerint támogatják, és tömörített tartalmat továbbít egy formátumban, például zip- vagy gzip ügyfelek a. A CDN-végpont alkalmazás mappát használ, amikor a kiszolgáló előfordulhat, hogy tömörítéséhez néhány automatikusan azonos módon kézbesíti az közvetlenül egy webes böngésző vagy egyéb típusú ügyfél. A formátum a értékétől függ a **elfogadás kódolás** az ügyfél által küldött kérelem fejléce. Azure az alapértelmezett mechanizmus, hogy automatikusan tömöríti a tartalmat, ha a CPU-felhasználás 50 % alatt van. Ha az alkalmazás futtatásához egy felhőalapú szolgáltatást használ, a szolgáltatás beállításainak módosítása lehet szükség egy indítási feladat segítségével kapcsolja be a dinamikus kimenet az IIS tömörítést. Lásd: [Microsoft Azure CDN keresztül a webes szerepkör gzip tömörítés engedélyezése](http://blogs.msdn.com/b/avkashchauhan/archive/2012/03/05/enableing-gzip-compression-with-windows-azure-cdn-through-web-role.aspx) további információt.
* Engedélyezheti a tömörítést közvetlenül a CDN peremhálózati kiszolgálóinak, amelyben eset a CDN-t a fájlok tömöríti és szolgálják a végfelhasználók számára. További információkért lásd: [Azure CDN tömörítési](/azure/cdn/cdn-improve-performance/).

### <a name="routing-and-versioning"></a>Az Útválasztás és versioning
CDN-példány használható a különböző időpontokban szeretne. Például az alkalmazás új verziójának telepítésekor érdemes lehet egy új CDN-t használja, és tartsa meg a régi CDN (tartalom okozó régebbi formátumú) korábbi verzióihoz. Ha a tartalom eredete használja az Azure blob Storage tárolóban, hozzon létre egy külön tárfiókot vagy egy külön tárolóba, és a CDN-végpont mutasson. A CDN-végpont a cdn gyökérmappájába, az alkalmazáson belül használja, ha URL-cím átírása technikák segítségével kéréseiket egy másik mappába.

Ne használja a lekérdezési karakterlánc jelölésére a CDN megtalálható erőforrásokhoz való hivatkozások az alkalmazás különböző verzióit, mert, amikor a tartalom lekérése az Azure blob storage, a lekérdezési karakterlánc része az erőforrás neve (a blob neve). Ezt a módszert használja szintén befolyásolhatják, hogy az ügyfél milyen módon gyorsítótárazza a erőforrások.

Statikus tartalom új verzióinak telepítését, ha egy alkalmazás módosítását kihívást lehet, ha az előző erőforrások gyorsítótárazza a CDN-t. További információkért tekintse meg a szakasz *vezérlő gyorsítótár*).

Fontolja meg a CDN ország tartalom hozzáférés korlátozása. Az Azure CDN lehetővé teszi szűrheti a kéréseket a származási alapján, és korlátozza a tartalom letöltéséhez. További információkért lásd: [korlátozza a hozzáférést a tartalmat az ország](/azure/cdn/cdn-restrict-access-by-country/).

### <a name="cache-control"></a>Gyorsítótár-vezérlő
Vegye figyelembe a rendszerben gyorsítótárazás kezelése. Például egy mappa használatakor, mivel a CDN-forrása megadhatja a gyorsítótárazhatóságának lapok, amelyek létrehozzák a tartalmat, és a tartalom lejárati idejének összes erőforrást egy adott mappában. Gyorsítótár tulajdonságai a CDN, valamint a szabványos HTTP-fejlécek ügyfélre is megadható. Ön már meg kell kezelése, a kiszolgáló és az ügyfél gyorsítótára, bár a CDN használata segít, hogy Ön több tudomást hogyan a tartalom gyorsítótárazva van, és ha.

Objektumok, hogy nem érhető el a CDN a törölheti azokat a forrásból (blob-tároló vagy alkalmazás *cdn* gyökérmappájába), távolítsa el vagy törölni a CDN-végpontot, vagy esetén blob-tároló, ellenőrizze a tároló vagy titkos blob . Azonban elemek törlődni fog a CDN csak akkor, ha azok idő TTL lejárata. Ha nincs gyorsítótár lejárati időszak (például a tartalom Ha be van töltve, a blob-tároló) van megadva, gyorsítótárazza a CDN a hét napig.  Manuálisan is [a CDN-végpont törlése](/azure/cdn/cdn-purge-endpoint/).

A webalkalmazás, beállíthatja a gyorsítótárazás és az összes tartalom lejárati használatával a *clientCache* eleme a *system.webServer/staticContent* szakasz a web.config fájl. Ne feledje, hogy ha a web.config fájlt helyez egy mappát az hatással van a fájlokat a mappában és annak összes almappája.

Ha hoz létre a tartalmat a CDN a dinamikusan (például az alkalmazás kódjában), győződjön meg arról, hogy megadja a *Cache.SetExpires* minden oldalon tulajdonság. A CDN nem gyorsítótárazzák a kimenete a felügyelt oldalak felhívásainak alapértelmezett gyorsítótárazhatóságának beállítása *nyilvános*.  Győződjön meg arról, hogy a tartalom nem elvetett és nagyon rövid időközönként az alkalmazásból a módosítás megfelelő értéket a gyorsítótár lejárati időszak beállítása  

### <a name="security"></a>Biztonság
A CDN által biztosított tartalmat, keresztül HTTPS (SSL), a CDN által biztosított tanúsítvány használatával, valamint szabványos HTTP-kapcsolaton keresztül. Annak érdekében, ne böngésző vegyes tartalomról, szükség lehet a HTTPS használatával betöltött lapok statikus tartalmat igénylő a HTTPS protokoll használatával.

Ha az eszközök statikus betűtípus fájlok például a CDN használatával, problémák léphetnek fel azonos eredetű házirend használatakor egy *XMLHttpRequest* hívás ezekkel az erőforrásokkal kérhet egy másik tartományban. Sok webböngészővel eltérő eredetű erőforrások megosztása (CORS), kivéve, ha a webkiszolgáló úgy van beállítva, a megfelelő válaszfejlécek beállítása megakadályozza. Beállíthatja, hogy a CDN és a CORS támogatja az alábbi módszerek egyikének használatával:

* A CDN szabálymotor segítségével CORS fejlécek hozzá a válaszokat. Ez a módszer az általában a legjobb melyiket használja, mert helyettesítő és több specifikus engedélyezett eredetet egyaránt támogatottak. További információkért lásd: [az Azure CDN szolgáltatás használata a CORS](https://docs.microsoft.com/en-us/azure/cdn/cdn-cors). 
* Adja hozzá a *CorsRule* szolgáltatás tulajdonságait. Ez a módszer is használhatja, ha a forrás, amelyből továbbítja a tartalmat az Azure blob Storage tárolóban. A szabályban megadhatja az engedélyezett eredetet a CORS kérelmek, az engedélyezett módszerek, például a GET és a maximális élettartamát másodpercben. a szabály (az az időszak, amelyen belül az ügyfél kell igényelnie a kapcsolt erőforrásokban az eredeti tartalom betöltése után). Ha be kell állítani a CORS Storage használata a CDN, csak a "*" helyettesítő karakter támogatott engedélyezett eredetet listája. További információkért lásd: [Cross-Origin Resource Sharing (CORS) támogatása az Azure Storage szolgáltatásainak](http://msdn.microsoft.com/library/azure/dn535601.aspx).
* Kimenő szabályok konfigurálása az alkalmazás konfigurációs fájljának beállítani egy *hozzáférés-vezérlési-engedélyezése-forrás* minden válasz fejléce. Ez a módszer is használhatja, ha a forrás kiszolgálón fut az IIS. Ha ezt a módszert használja a CDN, csak a "*" helyettesítő karakter támogatott engedélyezett eredetet listája. Átdolgozás szabályok használatával kapcsolatos további információkért lásd: [URL-újraíró modult](http://www.iis.net/learn/extensions/url-rewrite-module).

### <a name="custom-domains"></a>Egyéni tartományok
Az Azure CDN lehetővé teszi egy [egyéni tartománynév](/azure/cdn/cdn-map-content-to-custom-domain/) és a CDN keresztül elérésére használt. Is beállíthat egy egyéni altartomány nevét használja a *CNAME* rekord a DNS-ben. Ezzel a megközelítéssel absztrakciós és vezérlés további réteget biztosít.

Ha egy *CNAME*, SSL nem használható, mert a CDN a saját egyetlen SSL-tanúsítványt használ, és ez a tanúsítvány nem fog egyezni az egyéni tartomány/altartománynevek.

### <a name="cdn-fallback"></a>CDN tartalék
Vegye figyelembe, hogyan az alkalmazás fogja a folyamatosan növekvő adatmennyiségnek hiba vagy ideiglenes hiányában a CDN-t. Előfordulhat, hogy ügyfélalkalmazások másolja a helyileg (az ügyfél) a gyorsítótárazott erőforrások használhatják az előző kérelem során, vagy megadhat kódot, amely hibát észlel, és helyette a forrásból (az alkalmazás mappájában, vagy az Azure blob-erőforrások kérelmek az tároló, amely az erőforrások) Ha a CDN nem érhető el.

Az alábbi példában látható, a tartalék mechanizmusok használatával [címke segítő](https://docs.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/intro) Razor nézetben.

```HTML
...
<link rel="stylesheet" href="https://[your-cdn-endpoint].azureedge.net/lib/bootstrap/dist/css/bootstrap.min.css"
      asp-fallback-href="~/lib/bootstrap/dist/css/bootstrap.min.css"
      asp-fallback-test-class="sr-only" asp-fallback-test-property="position" asp-fallback-test-value="absolute"/>
<link rel="stylesheet" href="~/css/site.min.css" asp-append-version="true"/>
...
<script src="https://[your-cdn-endpoint].azureedge.net/lib/jquery/dist/jquery-2.2.0.min.js"
        asp-fallback-src="~/lib/jquery/dist/jquery.min.js"
        asp-fallback-test="window.jQuery">
</script>
<script src="https://[your-cdn-endpoint].azureedge.net/lib/bootstrap/dist/js/bootstrap.min.js"
        asp-fallback-src="~/lib/bootstrap/dist/js/bootstrap.min.js"
        asp-fallback-test="window.jQuery && window.jQuery.fn && window.jQuery.fn.modal">
</script>
...
```

### <a name="search-engine-optimization"></a>Optimalizálás érdekében
Ha a keresőmotor-Optimalizáláshoz fontos szempont az alkalmazás, a következő feladatokat:

* Tartalmaznak egy *Rel* kanonikus fejléc a következő minden lap vagy az erőforrás.
* Használja a *CNAME* altartomány rögzítése és elérhessék az erőforrásokat, ezzel a névvel.
* Vegye figyelembe a tényt, hogy a CDN IP-címe lehet olyan országban vagy régióban, amely eltér az alkalmazás maga a hatását.
* Ha a Azure blob storage-t a forrás, karbantartása a azonos a CDN, mint az alkalmazás mappájához erőforrásainak fájlstruktúra.

### <a name="monitoring-and-logging"></a>Figyelés és naplózás
A CDN a alkalmazásfigyelési stratégia észlelésére, és mérheti hibák részeként vagy a kiterjesztett várakozási ideje eseményeket.  Figyelés a CDN profil managerből az Azure portál webhelyen érhető el.

A CDN naplózásának engedélyezéséről, és figyelje ezt a naplót a napi szintű működése részeként.

Vegye figyelembe a használati minták CDN forgalom elemzése. Az Azure-portálon biztosít, amelyek lehetővé teszik a figyelheti eszközök:

* Sávszélesség,
* Az adatok átvitele,
* Találatok (állapotkódok)
* Gyorsítótár állapota
* Gyorsítótár TALÁLATI arány és
* IPV4-/ IPV6 kérelmek aránya.

További információkért lásd: [elemzése CDN használati minták](/azure/cdn/cdn-analyze-usage-patterns/).

### <a name="cost-implications"></a>Költség gyakorolt hatása
A CDN kimenő adatátviteli számolnak.  Emellett a blob storage használatakor az eszközök tárolásához van szó, a storage-tranzakció betöltésekor a CDN adat az alkalmazásról. A tartalom frissesség biztosításához, de nem olyan rövid, az alkalmazás tartalmának ismételt újból betölteni vagy blob-tároló és a CDN reális gyorsítótár lejárati időszak beállítása

Ritkán letöltött elemek merülnek fel, amelyekre a két tranzakciós költségek nélkül bármely jelentős mértékben csökken a kiszolgáló terhelését.

### <a name="bundling-and-minification"></a>Kötegelése és minification
Kötegelése és minification használatával az erőforrások, például JavaScript-kód és tárolja a CDN HTML-lapok méretének csökkentésére. Ezt a stratégiát segítségével ezeket az elemeket az ügyfél letöltési szükséges idő csökkentése érdekében.

ASP.NET bundling és minification is kezeli. MVC projekt adhat meg a csomagok a *BundleConfig.cs*. Egy hivatkozást a minified parancsfájl csomagot hozta létre hívja a *Script.Render* módszer általában a kódban a nézet osztály. Ezt a hivatkozást tartalmaz egy lekérdezési karakterlánc, amely tartalmaz egy kivonatot, amely a csomag tartalmának alapul. Ha módosítja a csomag tartalmát, a létrehozott kivonat is módosul.  

Alapértelmezés szerint Azure CDN célpéldánynál a *lekérdezési karakterlánc állapot* beállításával le vannak tiltva. Ahhoz, hogy a CDN megfelelően kezeli a frissített parancsfájl kötegek, engedélyeznie kell a *lekérdezési karakterlánc állapot* a CDN-példány beállítása. Vegye figyelembe, hogy eltarthat egy óráig vagy tovább előtt a beállítás akkor lép érvénybe.

### <a name="features"></a>Szolgáltatások

Azure több CDN termék rendelkezik. CDN kiválasztásakor vegye figyelembe az egyes termékek támogató szolgáltatások. Lásd: [Azure CDN szolgáltatásai] [ cdn-features] részleteiről. Kezdésként érdemes prémium szolgáltatások a következők:

- **[Szabálymotor](/azure/cdn/cdn-rules-engine)**. A szabályok motor lehetővé teszi, hogy segítségével testre szabhatja a HTTP-kérések kezelésének módja, például bizonyos típusú tartalom kézbesítésével blokkolja, a gyorsítótárazási házirend meghatározása és a HTTP-fejlécek módosítása. 

- **[Valós idejű statisztikák](/azure/cdn/cdn-real-time-stats)**. Valós idejű adatok, például a sávszélesség, a gyorsítótár állapotok és a CDN-profilt, létesített egyidejű kapcsolatok figyelésére és fogadására [valós idejű riasztások](/azure/cdn/cdn-real-time-alerts). 


## <a name="rules-engine-url-rewriting-example"></a>Szabályok motor írja át például URL-címe

A következő ábra bemutatja, hogyan hajthat végre [URL-cím újraírását](https://technet.microsoft.com/library/ee215194.aspx) a CDN használata esetén. A CDN a gyorsítótárazott tartalom kérelmet átirányítja az alkalmazás gyökérkönyvtárában típusa az erőforrás (például parancsfájlok vagy képeket) belül meghatározott mappákról.  

![Szabályok motor diagramja](./images/cdn/rules-engine.png)

Ezek a szabályok újraírási hajtsa végre a következő átirányítások:

* Az első szabály lehetővé teszi egy verzió beágyazása a fájlnév egy erőforrást, amely figyelmen kívül hagyja. Például *Filename_v123.jpg* rendszer újraírja *Filename.jpg*.
* A következő négy szabályok bemutatják, hogyan szeretné irányítani a kérelmeket, ha nem szeretné, hogy az erőforrások tárolásához mappában *cdn** a webes szerepkör gyökérkönyvtárában. A szabályai leképezik a *cdn/képek*, *cdn-tartalom*, *cdn/parancsfájlok*, és *cdn/kötegek* a megfelelő legfelső szintű mappáihoz a webhely URL-címek szerepkör.

Vegye figyelembe, hogy írja át a URL-cím segítségével meg kell néhány módosítást erőforrások kötegelése.     

## <a name="more-information"></a>További információ
* [Azure CDN](https://azure.microsoft.com/services/cdn/)
* [Az Azure Content Delivery Network (CDN) dokumentációja](https://azure.microsoft.com/documentation/services/cdn/)
* [Az Azure CDN szolgáltatás használata](/azure/cdn/cdn-create-new-endpoint/)
* [Egy felhőalapú szolgáltatás integrálása az Azure CDN](/azure/cdn/cdn-cloud-service-with-cdn/)(https://azure.microsoft.com/blog/2011/03/18/best-practices-for-the-windows-azure-content-delivery-network/)


<!-- links -->

[cdn-features]: /azure/cdn/cdn-overview#azure-cdn-features
