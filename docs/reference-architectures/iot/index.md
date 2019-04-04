---
title: Azure IoT-referenciaarchitektúra
description: Javasolt architektúra az Azure-on futó IoT-alkalmazásokhoz PaaS- (szolgáltatásként nyújtott platform) összetevőkkel
titleSuffix: Azure Reference Architectures
author: MikeWasson
ms.date: 01/09/2019
ms.openlocfilehash: 5a4b104044f3e64ffdce98e3952201d397d41f33
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58344580"
---
# <a name="azure-iot-reference-architecture"></a>Azure IoT-referenciaarchitektúra

Ez a referenciaarchitektúra egy javasolt architektúrát mutat be az Azure-on futó IoT-alkalmazásokhoz PaaS- (szolgáltatásként nyújtott platform) összetevőkkel.

![Az architektúra ábrája](./_images/iot.png)

Az IoT-alkalmazásokban lényegében az **eszközök** adatokat küldenek, amelyekből **megállapítások** jönnek létre. Ezek a megállapítások aztán **műveleteket** indítanak az üzletmenet vagy valamely folyamat javítására. Vegyünk például egy motort (az eszköz), amely hőmérsékletadatokat küld. Az adatok alapján a rendszer kiértékeli, hogy a motor a várakozásnak megfelelően működik-e (a megállapítás). A megállapítás alapján proaktívan rangsorolható a motor karbantartásának ütemezése (a művelet).

Ez a referenciaarchitektúra Azure PaaS- (szolgáltatásként nyújtott platform) összetevőket használ. Egyéb lehetőségek az IoT-megoldások létrehozására az Azure-on:

- [Azure IoT Central](/azure/iot-central/). Az IoT Central egy teljeskörűen felügyelt SaaS- (szolgáltatásként nyújtott szoftver) megoldás. Absztrahálja a műszaki választásokat, így Önnek kizárólag a megoldásra összpontosíthat. Az egyszerűségért cserébe kevésbé testreszabható, mint a PaaS-alapú megoldások.
- Azure-beli virtuális gépeken üzembe helyezett OSS-összetevők, például a SMACK-verem (Spark, Mesos, Akka, Cassandra, Kafka) használatával. Ez a módszer nagy fokú irányítást biztosít, de összetettebb.

Magas szinten a telemetriaadatok feldolgozásának két módja van, a gyakori és a ritka elérésű útvonalak. A különbség a késésre és az adatelérésre vonatkozó követelményekben rejlik.

- A **gyakori elérésű útvonal** közel valós időben elemzi a beérkező adatokat. A gyakori elérésű útvonalon a telemetriát rendkívül kis késéssel kell feldolgozni. A gyakori elérésű útvonalakat általában egy streamfeldolgozó motorral valósítják meg. A kimenet aktiválhat egy riasztást, vagy kiírható egy strukturált formátumba, amely aztán elemzőeszközökkel lekérdezhető.
- A **ritka elérésű útvonal** kötegelt feldolgozást végez hosszabb időközönként (óránként vagy naponta). A ritka elérésű útvonal általában nagy mennyiségű adatot kezel, de az eredményeknek nem kell olyan időszerűnek lenniük, mint a gyakori elérési útvonalnál. A ritka elérésű útvonalon rögzített nyers telemetriaadatokat egy kötegelt folyamatba táplálja be a rendszer.

## <a name="architecture"></a>Architektúra

Az architektúra az alábbi összetevőkből áll. Nem minden alkalmazáshoz szükséges az itt felsorolt összes összetevő.

**IoT-eszközök**. Az eszközök biztonságosan regisztrálhatnak a felhőben, majd a felhőre csatlakozva küldhetnek és fogadhatnak adatokat. Egyes eszközök lehetnek **peremhálózati eszközök**, amelyek az adatfeldolgozás egy részét magán az eszközön vagy egy helyszíni átjárón végzik. A peremhálózati feldolgozáshoz az [Azure IoT Edge](/azure/iot-edge/) használatát javasoljuk.

**Felhőátjáró**. A felhőátjáró egy felhőalapú központot biztosít az eszközök számára, amelyen keresztül biztonságosan csatlakozhatnak a felhőhöz és küldhetnek adatokat. Emellett eszközkezelési képességeket is kínál, többek között az eszközök irányítását és vezérlését. A felhőátjárókhoz az [IoT Hub](/azure/iot-hub/) használatát javasoljuk. Az IoT Hub egy üzemeltetett felhőszolgáltatás, amely üzenetközvetítőként működik az eszközök és a háttérszolgáltatások között. Az IoT Hub biztonságos kapcsolódási, eseménybetöltési, kétirányú kommunikációs és eszközkezelési képességeket biztosít.

**Eszközkiépítés**. Eszközök mennyiségi regisztrálása és csatlakoztatása esetén az [IoT Hub Device Provisioning Service](/azure/iot-dps/) (DPS) használatát javasoljuk. A DPS segítségével nagy mennyiségben oszthat ki és regisztrálhat eszközöket adott Azure IoT Hub-végpontokra.

**Streamfeldolgozás**. A streamfeldolgozás nagy méretű adatrekordstreameket dolgoz fel, és szabályokat értékel ki ezekre a streamekre. Streamfeldolgozáshoz az [Azure Stream Analytics](/azure/stream-analytics/) használatát javasoljuk. A Stream Analytics használatával összetett elemzéseket hajthatók végre nagy adatmennyiségeken időablak-készítési függvényekkel, streamaggregációkkal, valamint külső adatforrások összekapcsolásával. Egy másik lehetőség az Apache Spark és az [Azure Databricks](/azure/azure-databricks/) használata.

A gépi tanulás révén prediktív algoritmusok hajthatók végre telemetria-előzményadatok alapján, így például prediktív karbantartási és hasonló forgatókönyvek alakíthatók ki. A gépi tanuláshoz az [Azure Machine Learning szolgáltatás](/azure/machine-learning/service/) használatát javasoljuk.

A **közepes elérésű tárolás** olyan adatokat tárol, amelyeknek azonnal elérhetőnek kell lenniük az eszközön jelentéskészítési és vizualizációs célokra. Közepes elérésű tároláshoz a [Cosmos DB](/azure/cosmos-db/introduction) használatát javasoljuk. A Cosmos DB egy globálisan elosztott, többmodelles adatbázis.

A **ritka elérésű tárolás** hosszabb távra megőrzött és kötegelten feldolgozott adatokat tárol. Ritka elérésű tároláshoz az [Azure Blob Storage](/azure/storage/blobs/storage-blobs-introduction) használatát javasoljuk. Az adatok a Blob Storage-tárolókban határozatlan időre alacsony áron archiválhatók, és könnyen elérhetők kötegelt feldolgozásra.

Az **adatátalakítás** a telemetriastreamet módosítja és összesíti. Ilyen módosítások a protokollátalakítások is, például a bináris adatok JSON-formátumba való alakítása vagy az adatpontok kombinálása. Ha az adatokat át kell alakítani az IoT Hub elérése előtt, javasoljuk egy [protokoll-átjáró](/azure/iot-hub/iot-hub-protocol-gateway) használatát (az ábrán nem szerepel). Ellenkező esetben az adatok az IoT Hub elérése után alakíthatók át. Ebben az esetben az [Azure Functions](/azure/azure-functions/) használatát javasoljuk. A Functions beépített IoT Hub-, Cosmos DB- és Blob Storage-integrációval rendelkezik.

Az **üzletifolyamat-integráció** műveleteket hajt végre az eszközadatokból táplálkozó megállapítások alapján. Ilyen művelet lehet a tájékoztató üzenetek tárolása, a riasztások létrehozása, az e-mail- vagy SMS-üzenetek küldése vagy a CRM-integráció. Az üzletifolyamat-integrációhoz az [Azure Logic Apps](/azure/logic-apps/logic-apps-overview) használatát javasoljuk.

A **felhasználókezeléssel** szabályozható, hogy mely felhasználók vagy csoportok hajthatnak végre műveleteket az eszközökön, például frissíthetik a belső vezérlőprogramot. Ez emellett a felhasználók alkalmazásokon belüli képességeit is szabályozza. A felhasználók hitelesítéséhez és engedélyezéséhez az [Azure Active Directory](/azure/active-directory/) használatát javasoljuk.

## <a name="scalability-considerations"></a>Méretezési szempontok

Az IoT-alkalmazásokat egymástól függetlenül méretezhető, különálló szolgáltatásokként érdemes megvalósítani. A méretezés során a következő szempontokat érdemes figyelembe vennie:

**IoT Hub**. Az IoT Hub esetében a következő méretezési tényezők lépnek fel:

- Az IoT Hubba [naponta maximálisan küldhető üzenetek kvótája](/azure/iot-hub/iot-hub-devguide-quotas-throttling).
- Az egyes IoT Hub-példányokon csatlakoztatott eszközök kvótája.
- Betöltési sebesség &mdash; milyen gyorsan képes az IoT Hub betölteni az üzeneteket.
- Feldolgozási sebesség &mdash; mennyi időt vesz igénybe a bejövő üzenetek feldolgozása.

Üzembe helyezéskor minden IoT hub bizonyos számú egységet tartalmaz egy adott szinten. A szint és az egységek száma határozza meg az eszközök által naponta a hubra maximálisan küldhető üzenetek kvótáját. További információkért lásd: Az IoT Hub kvótái és szabályozása. A hubok a folyamatban lévő műveletek megszakítása nélkül skálázhatók fel vertikálisan.

**Stream Analytics**. A Stream Analytics-feladatok legjobban akkor méretezhetők, ha a Stream Analytics-folyamat minden pontján párhuzamosak, a bemenettől a lekérdezésen keresztül a kimenetig. A teljes mértékben párhuzamos feladatokban a Stream Analytics több számítási csomópont között tudja szétosztani a munkát. Máskülönben a Stream Analyticsnek egy helyen kell kombinálnia a streamadatokat. További információért lásd [az Azure Stream Analytics-lekérdezések párhozamosításának előnyeit ismertető](/azure/stream-analytics/stream-analytics-parallelization) cikket.

Az IoT Hub automatikusan particionálja az eszközüzeneteket az eszközazonosító alapján. Egy adott eszközről az összes üzenet mindig ugyanarra a partícióra érkezik, egy partícióra azonban több eszközről is érkeznek üzenetek. A párhuzamosításhoz használt egység tehát a partícióazonosító.

**Functions**. Az Event Hubs-végpont olvasásakor meg van határozva a függvénypéldányok maximális száma eseményközpont-partíciónként. A maximális feldolgozási sebességét az határozza meg, hogy egy függvénypéldány milyen gyorsan képes feldolgozni az eseményeket egyetlen partícióról. A függvénynek kötegekben érdemes feldolgoznia az üzeneteket.

**Cosmos DB**. Egy Cosmos DB-gyűjtemény horizontális felskálázásához egy partíciókulccsal hozza létre a gyűjteményt, és a partíciókulcsot minden Ön által írt dokumentumban szerepeltesse. További információért lásd [a partíciókulcsokkal kapcsolatos ajánlott eljárásokat ismertető szakaszt](/azure/cosmos-db/partition-data#best-practices-when-choosing-a-partition-key).

- Ha eszközönként egyetlen dokumentumot tárol, és azt frissíti, az eszköz azonosítója megfelelő partíciókulcs. Az írási műveletek egyenlően oszlanak el a kulcsok között. Az egyes partíciók mérete szigorúan korlátozva van, mivel mindegyik kulcsértékhez csak egyetlen dokumentum tartozik.
- Ha eszközüzenetenként külön dokumentumot tárol, hamar túllépné a partíciónkénti 10 GB-os korlátot, ha az eszközazonosítót használná partíciókulcsként. Az üzenetazonosító ebben az esetben megfelelőbb partíciókulcs. Az eszközazonosítót ilyen esetben is érdemes belefoglalnia a dokumentumba indexelési és lekérdezési célokkal.

## <a name="security-considerations"></a>Biztonsági szempontok

### <a name="trustworthy-and-secure-communication"></a>Megbízható és biztonságos kommunikáció

Az eszközökről érkező és a nekik küldött összes üzenetnek megbízhatónak kell lennie. Ha egy eszköz nem támogatja az alábbi titkosítási funkciókat, a helyi hálózatokra kell korlátoznia, és a hálózatok közötti összes kommunikációnak helyszíni átjárókon keresztül kell zajlania:

- Adattitkosítás egy bizonyíthatóan biztonságos, nyilvánosan elemzett és széles körben bevezetett, szimmetrikus kulcsú titkosítási algoritmussal.
- Digitális aláírás egy bizonyíthatóan biztonságos, nyilvánosan elemzett és széles körben bevezetett, szimmetrikus kulcsú aláírási algoritmussal.
- A TLS 1.2 támogatása a TCP- vagy egyéb streamalapú kommunikációs útvonalakon, vagy a DTLS 1.2 támogatása a datagramalapú kommunikációs útvonalakon. Az X.509 tanúsítvány kezelésének támogatása nem kötelező, és kiváltható a TLS számítási és átviteli szempontból hatékonyabb előre megosztott kulcsú módjával, amely az AES és az SHA-2 algoritmusok támogatásával is megvalósítható.
- Frissíthető kulcstároló és eszközönkénti kulcsok. Minden eszköznek egyedi kulcsadatokkal vagy jogkivonatokkal kell rendelkeznie, amelyek azonosítják a rendszer felé. Az eszközök kulcsait biztonságosan, magukon az eszközökön kell tárolni (például egy biztonságos kulcstárolóval). Az eszközöknek képesnek kell lenniük a kulcsok vagy jogkivonatok rendszeres időközönként, illetve vészhelyzet (pl. a rendszer feltörésekor) esetén történő reaktív frissítésére.
- Az eszközök belső vezérlőprogramjának és a rajtuk lévő alkalmazásszoftvereknek engedélyezniük kell a frissítéseket a felfedezett biztonsági rések javítása érdekében.

Számos eszköz azonban túlzottan korlátozott, és nem tudja támogatni ezeket a követelményeket. Ilyen esetben helyszíni átjárót érdemes használnia. Az eszközök biztonságosan kapcsolódnak a helyszíni átjáróhoz egy helyi hálózaton keresztül, és az átjáró teszi lehetővé a biztonságos kommunikációt a felhő felé.

### <a name="physical-tamper-proofing"></a>Illetéktelen hozzáférés elleni fizikai védelem

Erősen ajánlott az eszközt úgy kialakítani, hogy védelmet nyújtson az illetéktelen hozzáférésre irányuló fizikai kísérletek ellen, és így segítsen biztosítani a teljes rendszer biztonsági integritását és megbízhatóságát.

Például:

- Válasszon olyan mikrovezérlőket/mikroprocesszorokat vagy kiegészítő hardvereket, amelyek biztosítják a biztonságos tárolást és a titkosítási kulcsok használatát, például a platformmegbízhatósági modul (TPM) integrációját.
- Biztonságos rendszertöltő és biztonságos szoftverbetöltés, amelyeket a TPM biztosít.
- Használjon érzékelőket a behatolási kísérletek és az eszköz környezetének illetéktelen módosítására irányuló kísérletek felderítésére, riasztásokkal és az eszköz lehetséges „digitális önmegsemmisítésével”.

A további biztonsági szempontokért lásd [az eszközök internetes hálózatának (IoT) biztonsági architektúráját ismertető szakaszt](/azure/iot-fundamentals/iot-security-architecture).

### <a name="monitoring-and-logging"></a>Monitorozás és naplózás

A naplózó és monitorozó rendszerekkel megállapítható, hogy a megoldás jól működik-e, emellett használhatók a hibaelhárítás támogatására is. A monitorozó és naplózó rendszerekkel megválaszolhatók az alábbi üzemeltetési kérdések:

- Hibásak az eszközök vagy rendszerek?
- Megfelelően vannak konfigurálva az eszközök vagy rendszerek?
- Pontos adatokat állítanak elő az eszközök vagy rendszerek?
- Megfelelnek a rendszerek a vállalkozás és a végfelhasználók elvárásainak?

A naplózó és monitorozó eszközök általában az alábbi négy összetevőt tartalmazzák:

- Rendszerteljesítmény- és idővonal-vizualizációs eszközök a rendszer monitorozására és alapszintű hibaelhárításra.
- Pufferelt adatbetöltés a naplóadatok pufferelésére.
- Hosszú távú tároló a naplóadatok megőrzésére.
- Keresési és lekérdezési képességek a naplóadatok megtekintésére és felhasználására a részletes hibaelhárításhoz.

A monitorozó rendszerekkel megállapításokat kaphat az IoT-megoldások állapotáról, biztonságáról, stabilitásáról és teljesítményéről illetően. Ezek a rendszerek részletesebb betekintést is lehetővé tesznek, rögzítik az összetevők konfigurációváltozásait, és kivonatolt naplóadatokat biztosítanak, amelyek feltárhatnak potenciális biztonsági réseket, javíthatják az incidenskezelési folyamatokat, és segíthetik a rendszer tulajdonosát a problémák elhárításában. Az átfogó monitorozási megoldásokban az adatokat le lehet kérdezni adott alrendszerenként, vagy több alrendszer összesítéseként.

A monitorozási rendszerek fejlesztésének első lépéseként meg kell határozni a hibátlan működést, a szabályozási megfelelőséget és a naplózási követelményeket. A gyűjtött metrikák a következők lehetnek:

- A fizikai és peremhálózati eszközök, valamint infrastruktúra-összetevők által jelentett konfigurációváltozások.
- Az alkalmazások által jelentett konfigurációváltozások, biztonsági naplók, kérésmennyiségek, válaszidők, hibaarányok és szemétgyűjtési statisztikák a felügyelt nyelveken.
- Az adatbázisok, adatmegőrzési tárolók és gyorsítótárak által jelentett lekérdezési és írási teljesítmények, sémamódosítások, biztonsági naplók, zárolások és holtpontok, indexelési teljesítmény, processzor-, memória- és lemezhasználat.
- A felügyelt szolgáltatások (IaaS, PaaS, SaaS és FaaS) által jelentett állapotmetrikák és konfigurációváltozások, amelyek befolyásolják a függő rendszerek állapotát és teljesítményét.

A monitorozási metrikák vizualizációi felhívják az üzemeltetők figyelmét a rendszer instabilitásaira, és elősegítik az incidensek megoldását.

### <a name="tracing-telemetry"></a>Nyomkövetési telemetria

A nyomkövetési telemetriával az üzemeltető nyomon követheti egy adott telemetriaadat útját a létrejöttétől kezdve a teljes rendszerben. Ez a nyomkövetés a hibák keresésében és elhárításában kap fontos szerepet. Az Azure IoT Hubot és az [IoT Hub eszközoldali SDK-it](/azure/iot-hub/iot-hub-devguide-sdks) használó IoT-megoldások esetében a követési datagramok a felhőből az eszközre küldött üzenetként keletkezhetnek, amelyek aztán beépülnek a telemetriastreambe.

### <a name="logging"></a>Naplózás

A naplózórendszerek rendkívül fontosak annak megértéséhez, hogy az adott megoldások milyen műveleteket vagy tevékenységeket hajtottak végre, hogy milyen hibák léptek fel, és a hibák javításában is segítséget nyújthatnak. A naplók elemzésével megérthetők és orvosolhatók a hibaállapotok, javíthatók a teljesítményjellemzők, és biztosítható a vonatkozó szabályoknak és szabályozásoknak való megfelelőség.

Bár az egyszerű szöveges naplózás bevezetési költségei alacsonyabbak, a gépek ezt sokkal nehezebben tudják elemezni/olvasni. Javasoljuk a strukturált naplózás használatát, mivel az így gyűjtött adatok a gépek által is elemezhetők, és emberi olvasásra is alkalmasak. A strukturált naplózás környezeti adatokkal és metaadatokkal egészíti ki a naplóadatokat. A strukturált naplózásban a tulajdonságok kulcs/érték párokként vagy – a jobb kereshetőség és lekérdezhetőség érdekében – egy rögzített séma szerint formázott elsődleges elemek.

## <a name="next-steps"></a>További lépések

- A javasolt architektúrát és a megvalósítási lehetőségeket a [Microsoft Azure IoT referenciaarchitektúráját](http://aka.ms/iotrefarchitecture) tartalmazó dokumentum (PDF) ismerteti részletesebben.

- A különféle Azure IoT-szolgáltatások részletes dokumentációjáért lásd [az Azure IoT alapjait ismertető cikket](/azure/iot-fundamentals/).

- A [GitHubon](https://github.com/mspnp/iot-guidance) talál egy példát egy IoT-implementációra.
