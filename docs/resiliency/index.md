---
title: Rugalmas alkalmazások tervezése az Azure-hoz
description: Rugalmas alkalmazások felépítése az Azure-ban magas rendelkezésre álláshoz és vészhelyreállításhoz.
author: MikeWasson
ms.date: 12/18/2018
ms.custom: resiliency
ms.openlocfilehash: ef8fd64756c483528aa83048e23f6387dedb74d6
ms.sourcegitcommit: 7d9efe716e8c9e99f3fafa9d0213d48c23d9713d
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/09/2019
ms.locfileid: "54160859"
---
# <a name="designing-resilient-applications-for-azure"></a>Rugalmas alkalmazások tervezése az Azure-hoz

Az elosztott rendszerekben időnként fellépnek hibák. Meghibásodhatnak a hardverek. Átmeneti hibák lehetnek a hálózaton. Ritkán a teljes szolgáltatásban vagy régióban üzemszünet állhat elő, de ezekre is fel kell készülni.

A megbízható alkalmazások felhőben való felépítése különbözik a vállalati környezetben való felépítésüktől. Míg régebben megoldást jelenthetett, ha jobb minőségű hardvert vett a vertikális felskálázás érdekében, a felhőalapú környezetekben ehelyett horizontálisan kell felskáláznia. A felhőalapú környezetek költségei a közös hardverek használata révén alacsonyan tartható. Az összes hiba megakadályozása helyett a cél a hibák hatásának minimálisra csökkentése a rendszerben.

Ez a cikk a rugalmas alkalmazások a Microsoft Azure-ban történő felépítését ismerteti. A cikk a *rugalmasság* és a kapcsolódó fogalmak ismertetésével kezdődik. Ezután ismerteti a rugalmasság elérésének egy, a tervezéstől és implementálástól a központi telepítésig és üzemeltetésig terjedő folyamatát, amely az alkalmazások élettartama során strukturált megközelítést használ.

<!-- markdownlint-disable MD026 -->

## <a name="what-is-resiliency"></a>Mi a rugalmasság?

<!-- markdownlint-enable MD026 -->

A **rugalmasság** a rendszer azon képessége, hogy helyreálljon a hibák után, és folytassa a működést. Nem a hibák *elkerüléséről* szól, hanem a hibákra adott olyan *válaszokról*, amelyek kiküszöbölik az állásidőt és az adatveszteséget. A rugalmasság célja, hogy az alkalmazás egy hibát követően teljesen működőképes állapotba térjen vissza.

A rugalmasság két fontos szempontja a magas rendelkezésre állás és a vészhelyreállítás.

- A **magas rendelkezésre állás** az alkalmazás azon képessége, amellyel kifogástalan állapotban tudja folytatni a működést, jelentős állásidő nélkül. A „kifogástalan állapot” alatt azt értjük, hogy az alkalmazás válaszol, és a felhasználók csatlakozni tudnak az alkalmazáshoz, valamint használni tudják azt.  
- A **vészhelyreállítás** a ritka, de nagyobb incidensekből való helyreállítás képessége: ezek nem átmeneti, széleskörű hibák, például a szolgáltatás megszakadása, ami hatással van egy teljes régióra. A vészhelyreállításba tartozik az adatok biztonsági mentése és archiválása, és manuális beavatkozást foglalhat magába, például az adatbázis biztonsági mentésből való helyreállítását.

A magas rendelkezésre állás és a vészhelyreállítás úgy képzelhető el a legegyszerűbben, hogy a vészhelyreállítás akkor kezdődik, amikor a hiba hatása nagyobb annál, mint amit a magas rendelkezésre állás kezelni képes.  

Amikor rugalmasságra törekszik a tervezéskor, ismernie kell a rendelkezésre állási követelményeket. Mennyi állásidő fogadható el? Ez részben a költség függvénye. Mennyibe kerül a lehetséges állásidő az üzletének? Mennyit kell befektetnie az alkalmazás magas rendelkezésre állásúvá tételéhez? Azt is meg kell határoznia, hogy mit jelent az alkalmazás rendelkezésre állása. Az alkalmazás „üzemen kívül” van-e, ha egy ügyfél el tud küldeni egy rendelést, de a rendszer nem tudja feldolgozni azt a normál időkereten belül? Egy bizonyos típusú üzemszünet előfordulásának esélyét is vegye figyelembe, valamint hogy a kockázatcsökkentési stratégia költséghatékony-e.

Egy másik gyakori kifejezés az **üzletmenet folytonossága**, amely kedvezőtlen feltételek, például természeti katasztrófa vagy egy szolgáltatás kiesése alatt és után az alapvető üzleti funkciók elvégzésének képességét jelenti. Az üzletmenet folytonosságába beletartozik az üzlet teljes működése, beleértve a fizikai létesítményeket, a személyeket, a kommunikációt, a szállítást és az informatikát. Ez a cikk a felhőalapú alkalmazásokra összpontosít, de a rugalmasság tervezését az általános üzletmenet-folytonossági követelményeket figyelembe véve kell elvégezni.

Az **adatok biztonsági mentése** a DR kritikus része. Ha egy alkalmazás állapot nélküli összetevői meghibásodnak, mindig újból üzembe helyezheti őket. Adatvesztés esetén azonban a rendszer nem tud visszatérni stabil állapotba. Az adatokról biztonsági mentést kell készíteni, ideális esetben egy másik régióban, egy egész régióra kiterjedő katasztrófa esetére.

A biztonsági mentés nem azonos az *adatreplikációval*. Adatreplikáció esetén az adatok másolása szinte valós időben történik, hogy a rendszer gyorsan átadhassa a műveleteket egy replikának. Számos adatbázisrendszer támogatja a replikálást, például az SQL Server támogatja az SQL Server Always On rendelkezésre állási csoportokat. Az adatreplikáció csökkentheti a kimaradás utáni helyreálláshoz szükséges időt, mivel biztosítja, hogy az adatok replikája mindig készenlétben legyen. Az adatreplikáció azonban nem véd az emberi hibák ellen. Ha az adatok emberi hiba miatt sérülnek, ezek a sérült adatok egyszerűen átmásolódnak a replikákba. Ezért a DR-stratégiájának tartalmaznia kell a hosszú távú biztonsági mentést.

## <a name="process-to-achieve-resiliency"></a>A rugalmasság elérésének folyamata

A rugalmasság nem kezelhető bővítményként. A rendszer integrált részeként kell megtervezni, és a gyakorlat részévé kell tenni. Itt láthat egy általános modellt, amelyet követhet:

1. Az üzleti igények alapján **határozza meg** a rendelkezésreállási követelményeit.
2. **Tervezze meg** az alkalmazást a rugalmassághoz. Olyan architektúrával kezdjen, amely bevált gyakorlatokat követ, majd azonosítsa az architektúrában a lehetséges hibapontokat.
3. **Implementáljon** stratégiákat a hibák észleléséhez és az azokból való helyreálláshoz.
4. **Tesztelje** az implementálást a hibák szimulálásával és kényszerített feladatátvételek aktiválásával.
5. **Helyezze üzembe** az alkalmazást egy megbízható, megismételhető folyamattal.
6. **Monitorozza** az alkalmazást a hibák észlelése érdekében. A rendszer monitorozásával felmérheti az alkalmazás állapotát, és szükség esetén válaszolhat az incidensekre.
7. **Tegyen intézkedéseket**, ha manuális beavatkozást igénylő hibák fordulnak elő.

A cikk további részében részletesebben tárgyaljuk az egyes lépéseket.

## <a name="define-your-availability-requirements"></a>A rendelkezésreállási követelmények meghatározása

A rugalmasság tervezése az üzleti követelményekkel kezdődik. Az alábbiakban néhány olyan módszert találhat, amelyek bemutatják, miként veheti figyelembe a rugalmasságot ezeket a szempontokat szem előtt tartva.

### <a name="decompose-by-workload"></a>Felbontás számítási feladat alapján

Sok felhőalapú megoldás több számítási feladatból áll. Ebben a környezetben a „számítási feladat” kifejezés olyan különálló képességet vagy számítási feladatot jelent, amely logikusan elválasztható más feladatoktól az üzleti logika és az adattárolási követelmények szerint. Egy e-kereskedelmi alkalmazás például a következő számítási feladatokat tartalmazhatja:

- Termékkatalógus tallózása és keresés.
- Rendelések létrehozása és nyomon követése.
- Javaslatok megtekintése.

E számítási feladatok különböző követelményekkel rendelkezhetnek a rendelkezésre állás, a skálázhatóság, az adatkonzisztencia és a vészhelyreállítás alapján. Olyan üzleti döntéseket szükséges meghozni, amelyek a költségek és a kockázatok kiegyenlítésére vonatkoznak.

A használati mintákat is érdemes figyelembe venni. Vannak bizonyos kritikus időszakok, amikor a rendszernek elérhetőnek kell lennie? Egy adóbevallást készítő szolgáltatás például nem állhat le közvetlenül a bevallás elküldésének határideje előtt, egy videóstreamelő szolgáltatásnak pedig működnie kell egy nagy sportesemény alatt. A kritikus időszakok alatt létesíthet redundáns üzemelő példányokat több régióban, így az alkalmazás feladatátvétele akkor is biztosított, ha az egyik régió meghibásodik. Ám mivel a több régióból álló üzemelő példányok fenntartása költségesebb lehet, ezért a kevésbé kritikus időkben egyetlen régióban futtathatja az alkalmazást. Bizonyos esetekben a további költségek csökkenthetők olyan modern, kiszolgáló nélküli technológiákkal, amelyek használatalapú számlázást alkalmaznak, így a kihasználatlan számítási erőforrásokért nem kell fizetnie.

### <a name="rto-and-rpo"></a>RTO és RPO

Két fontos mérőszámot érdemes figyelembe vennie: a helyreállítási időre vonatkozó célkitűzést és a helyreállítási időkorlátot, mivel ezek a vészhelyreállításhoz tartoznak.

- A **helyreállítási időre vonatkozó célkitűzés** (RTO) az a maximális elfogadható idő, ameddig egy alkalmazás elérhetetlen maradhat egy incidens után. Ha az RTO 90 perc, akkor a katasztrófa kezdetétől számított 90 percen belül helyre kell tudnia állítani az alkalmazás futó állapotát. Ha nagyon alacsony RTO-val kell számolnia, készenléti módban tarthat egy aktív/passzív konfigurációt futtató második regionális üzemelő példányt, amely védelmet nyújt a regionális leállásokkal szemben. Bizonyos esetekben előfordulhat, hogy üzembe kell helyeznie egy aktív/passzív konfigurációt a még alacsonyabb RTO eléréséhez.

- A **helyreállítási időkorlát** (RPO) a katasztrófa során elfogadható adatveszteség maximális ideje. Ha például egyetlen adatbázisban tárol adatokat, és nem készít replikát egy másik adatbázisba, és óránként végez biztonsági mentéseket, akkor legfeljebb egy órányi adatot veszíthet.

Az RTO és az RPO egy rendszer nem funkcionális követelményei, amelyeket az üzleti követelményeknek kell meghatározniuk. Az értékek kiszámításához érdemes lehet kockázatfelmérést végezni, és átfogóan megismerni az állásidő és az adatvesztés járulékos költségeit.

### <a name="mttr-and-mtbf"></a>MTTR és MTBF

A rendelkezésre állás másik két gyakori mérőszáma a helyreállítás átlagos ideje (MTTR) és a hibák közötti átlagos idő (MTBF). Ezeket a mérőszámokat általában a szolgáltatók használják annak megállapítására, hogy hol szükséges redundanciát hozzáadni a felhőszolgáltatásokhoz és hogy melyik SLA-t biztosítsák az ügyfelek számára.

A **helyreállítás átlagos ideje** (MTTR) az az átlagos idő, amely az összetevő egy hiba utáni helyreállításához szükséges. Az MTTR az összetevő empirikus jellemzője. Az egyes összetevők MTTR-mérőszáma alapján megbecsülheti egy teljes alkalmazás MTTR-mérőszámát. Az alkalmazás alacsony MTTR-értékű összetevőkből történő létrehozása azt eredményezi, hogy az alkalmazás alacsony átlagos MTTR-értékkel rendelkezik – így gyorsan helyreállítható a hibák után.

A **hibák közötti átlagos idő** (MTBF) az a futásidő, amelyre az összetevő számíthat a szolgáltatáskimaradások között. Ez a mérőszám segíthet kiszámítani, milyen gyakran válik elérhetetlenné egy szolgáltatás. Egy nem megbízható összetevő MTBF-értéke alacsony, ezért az összetevő SLA-száma is alacsony lesz. Ugyanakkor az alacsony MTBF-érték korrigálható az összetevő több példányának üzembe helyezésével és a példányok közötti feladatátvétel beállításával.

> [!NOTE]
> Ha az összetevők BÁRMELY MTTR-értéke egy magas rendelkezésreállási környezetben meghaladja a rendszer RTO-értékét, a rendszer hibája az üzletmenet elfogadhatatlan mértékű szüneteltetését okozza. A rendszert nem lehet majd visszaállítani a megadott RTO-n belül.

### <a name="slas"></a>SLA-k

Az Azure-ban a [szolgáltatási szerződés][sla] (SLA) ismerteti a Microsoft az üzemidővel és hálózati elérhetőséggel kapcsolatos vállalásait. Ha egy adott szolgáltatás SLA-ja 99,9%-os, az azt jelenti, hogy a szolgáltatás várhatóan az idő 99,9%-ában elérhető.

> [!NOTE]
> Az Azure SLA jóváírást is biztosít, ha az SLA nem valósul meg, valamint az egyes szolgáltatások „rendelkezésre állásának” adott definícióit is megadja. Az SLA ezen eleme kényszerítési házirendként működik.

Saját cél SLA-kat kell meghatároznia a megoldás összes számítási feladatához. Az SLA lehetővé teszi annak kiértékelését, hogy az architektúra megfelel-e az üzleti követelményeknek. Ha például egy számítási feladat 99,99%-os üzemidőt igényel, de egy 99,9%-os SLA-val rendelkező szolgáltatásra támaszkodik, akkor a szolgáltatás nem lehet rendszerkritikus meghibásodási pont. Az egyik megoldás az, ha a szolgáltatás meghibásodása esetére rendelkezik egy tartalék elérési úttal, vagy ha más intézkedéseket tesz a szolgáltatás hibájából való helyreállításhoz.

Az alábbi táblázat a különböző SLA-szintek lehetséges halmozott állásidejét mutatja be.

| SLA | Állásidő hetente | Állásidő havonta | Állásidő évente |
| --- | --- | --- | --- |
| 99% |1,68 óra |7,2 óra |3,65 nap |
| 99.9% |10,1 perc |43,2 perc |8,76 óra |
| 99.95% |5 perc |21,6 perc |4,38 óra |
| 99.99% |1,01 perc |4,32 perc |52,56 perc |
| 99.999% |6 másodperc |25,9 másodperc |5,26 perc |

Természetesen a nagyobb rendelkezésre állás a jobb, ha minden más tényezőt figyelmen kívül hagyhatunk. De minél több „9-est” szeretne elérni, annál jobban nő a rendelkezésreállási szint elérésének költsége és összetettsége. A 99,99%-os üzemidő havonta összesen körülbelül 5 perc állásidőt jelent. Megéri a további összetettséget és költségeket az öt „9-es” elérése? A válasz az üzleti követelményektől függ.

Az alábbiakban néhány további szempontot találhat az SLA meghatározásához:

- Négy „9-es” (99,99%) eléréséhez valószínűleg nem támaszkodhat manuális beavatkozásra a hibákból való helyreállításhoz. Az alkalmazásnak képesnek kell lennie az öndiagnózisra és az önjavításra.
- A négy „9-esen” túl már kihívást jelent az állásidő elég gyors észlelése az SLA feltételeinek kielégítéséhez.
- Gondoljon az SLA-ban megszabott, a mérés alapját képező időtartományra. Minél kisebb a tartomány, annál szigorúbbak a feltételek. Valószínűleg nem éri meg óránkénti vagy napi üzemidőben meghatározni az SLA-t.
- Fontolja meg az MTBF- és az MTTR-mérőszám használatát. Minél alacsonyabb az SLA, annál ritkábban állhat le a szolgáltatás, és annál gyorsabban is kell helyreállnia.

### <a name="composite-slas"></a>Összetett SLA-k

Vegyünk egy olyan App Service-webalkalmazást, amely az Azure SQL Database adatbázisába ír. Az oktatóanyag összeállításakor ezek az Azure-szolgáltatások a következő SLA-kkal rendelkeztek:

- App Service Web Apps = 99,95%
- SQL Database = 99,99%

![Összetett SLA](./images/sla1.png)

Mi az alkalmazás esetében elvárt maximális állásidő? Ha valamelyik szolgáltatás meghibásodik, akkor a teljes alkalmazás meghibásodik. Általánosságban az egyes szolgáltatások meghibásodásának valószínűsége független egymástól, ezért a jelen alkalmazás összetett SLA-ja 99,95% &times; 99,99% = 99,94%. Ez alacsonyabb az egyes SLA-knál, ami nem meglepő, mert a több szolgáltatásra támaszkodó alkalmazások több lehetséges hibaponttal rendelkeznek.

Az összetett SLA viszont javítható független tartalék útvonalak létrehozásával. Ha például az SQL Database nem érhető el, állítsa üzenetsorba a tranzakciókat későbbi feldolgozás céljából.

![Összetett SLA](./images/sla2.png)

E kialakítás révén az alkalmazás akkor is elérhető, ha épp nem tud csatlakozni az adatbázishoz. Az alkalmazás azonban leáll, ha az adatbázis és az üzenetsor egyidejűleg hibásodik meg. Az egyidejű hiba várt időhányada 0,0001 &times; 0,001, ezért az ehhez a kombinált útvonalhoz tartozó összetett SLA:  

- Adatbázis VAGY üzenetsor = 1,0 &minus; (0,0001 &times; 0,001) = 99,99999%

A teljes összetett SLA:

- Webalkalmazás ÉS (adatbázis VAGY üzenetsor) = 99,95% &times; 99,99999% = ~99,95%

De ennek a módszernek hátrányai is vannak. Az alkalmazáslogika összetettebb, az üzenetsorért fizetni kell, és adatkonzisztenciával kapcsolatos problémák is felmerülhetnek.

**SLA többrégiós üzemelő példányokhoz**. Egy másik magas rendelkezésre állású megoldás az alkalmazás több régióban történő üzembe helyezése: ha az alkalmazás meghibásodik egy régióban, az Azure Traffic Manager révén feladatátvétel hajtható végre. Többrégiós üzemelő példány esetében az összetett SLA képlete a következő.

Legyen *N* az egy régióban üzembe helyezett alkalmazás összetett SLA-ja, *R* pedig azon régiók száma, amelyekben az alkalmazás üzembe van helyezve. Annak a várt esélye, hogy az alkalmazás egyidejűleg hibásodik meg minden régióban, ((1 &minus; N) ^ R).

Ha például egyetlen régió SLA-ja 99,95%-os, akkor

- Két régió összetett SLA-ja = (1 &minus; (0,9995 ^ 2)) = 99,999975%
- Négy régió összetett SLA-ja = (1 &minus; (0,9995 ^ 4)) = 99,999999%

Figyelembe kell vennie a [Traffic Manager SLA-ját][tm-sla] is. Az oktatóanyag összeállításakor a Traffic Manager SLA SLA-ja 99,99%.

A feladatátvétel továbbá nem azonnali lefutású az aktív-passzív konfigurációkban, így a feladatátvétel során szolgáltatáskiesés fordulhat elő. Lásd: [Traffic Manager-végpont monitorozása és feladatátvétele][tm-failover].

A kiszámított SLA-szám hasznos kiindulási alap, de nem mutat teljes képet a rendelkezésre állásról. Gyakran előfordul, hogy egy alkalmazás teljesítménye leállás nélkül csökken, ha nem kritikus útvonal hibásodik meg. Vegyünk például egy könyvkatalógust megjelenítő alkalmazást. Ha az alkalmazás nem tudja lekérni a borítók miniatűrjét, helyőrző képet jeleníthet meg helyette. Ebben az esetben a kép lekérésének sikertelensége nem csökkenti az alkalmazás üzemidejét, de hatással van a felhasználói élményre.  

## <a name="design-for-resiliency"></a>Rugalmasságot szem előtt tartó tervezés

A tervezési fázis során javasolt hibaállapot-elemzést (FMA) végezni. Az FMA célja a lehetséges meghibásodási pontok megtalálása, és annak definiálása, hogyan reagáljon az alkalmazás ezekre a hibákra.

- Hogyan észleli az alkalmazás az adott hibatípust?
- Hogyan reagál az alkalmazás az adott hibatípusra?
- Hogyan naplózza és monitorozza az adott hibatípust?

Az FMA eljárásra vonatkozó további információkért – a kifejezetten az Azure-ra vonatkozó ajánlásokkal együtt – lásd [az Azure rugalmasságáról szóló műszaki útmutató hibaállapot-elemzéssel][fma] foglalkozó részét.

### <a name="example-of-identifying-failure-modes-and-detection-strategy"></a>Példa a hibaállapotok azonosítására és az észlelési stratégiára

**Meghibásodási pont:** Külső webszolgáltatás/API meghívása.

| Hibaállapot | Észlelési stratégia |
| --- | --- |
| A szolgáltatás nem érhető el |HTTP 5xx |
| Throttling |HTTP 429 (Túl sok kérelem) |
| Hitelesítés |HTTP 401 (Jogosulatlan) |
| Lassú válasz |A kérés túllépi az időkorlátot |

### <a name="redundancy-and-designing-for-failure"></a>Redundancia és meghibásodásra felkészült tervezés

A hibák a hatásukat illetően eltérőek lehetnek. Bizonyos hardveres hibák – például egy meghibásodott lemez – csak egyetlen gazdagépet érintenek. Egy meghibásodott hálózati kapcsoló egy teljes kiszolgálószekrényt érinthet. Kevésbé gyakori meghibásodások teljes adatközpontok működését zavarhatják meg, például egy áramkimaradás esetén az adatközpontban. Ritkán egy teljes régió válhat elérhetetlenné.

Az alkalmazások rugalmasságának egyik legfőbb eleme a redundancia. Ezt a redundanciát azonban meg kell terveznie az alkalmazás fejlesztésekor. A szükséges redundancia szintje az üzleti követelményeitől függ. Nem szükséges minden alkalmazáshoz régiók közötti redundancia, hogy védelmet nyújtson a regionális kimaradással szemben. Általában a nagyobb redundancia és megbízhatóság magasabb költséget és összetettséget von magával.  

Az Azure számos funkciót kínál, amellyel az alkalmazások számára biztosítja a redundanciát a meghibásodások minden szintjén, egyetlen virtuális géptől kezdve egészen a teljes régiókig.

![Az Azure rugalmassági funkciói](./images/redundancy.svg)

**Egyetlen virtuális gép**. Az Azure [üzemidőre vonatkozó SLA-t](https://azure.microsoft.com/support/legal/sla/virtual-machines) biztosít a virtuális gépekhez. (A virtuális gépnek prémium szintű tárolást kell használnia minden operációsrendszer-lemez és adatlemez esetében.) Bár magasabb SLA-t kaphat két vagy több virtuális gép futtatása esetén, egyetlen virtuális gép is elég megbízható lehet bizonyos számítási feladatokhoz. Az éles számítási feladatokhoz azonban két vagy több virtuális gép használatát javasoljuk a redundancia biztosítása érdekében.

**Rendelkezésre állási csoportok**. A helyi hardverhibák, például lemezek vagy hálózati kapcsolók meghibásodása elleni védelem érdekében helyezzen üzembe két vagy több virtuális gépet egy rendelkezésre állási csoportban. Egy rendelkezésre állási csoport két vagy több *tartalék tartományból* áll, amelyek közös áramforrással és hálózati kapcsolóval rendelkeznek. A rendelkezésre állási csoportban található virtuális gépek el vannak osztva a tartalék tartományok között, ezért ha hardverhiba merül fel az egyik tartalék tartományban, a hálózati forgalom átirányítható a többi tartalék tartományban található virtuális gépekre. További információt a rendelkezésre állási csoportokról [a Windows rendszerű virtuális gépek rendelkezésre állásának az Azure-ban történő kezelését](/azure/virtual-machines/windows/manage-availability) ismertető cikkben talál.

**Rendelkezésre állási zónák**.  A rendelkezésre állási zónák egy Azure-régió fizikailag elkülönített zónáit jelentik. Mindegyik rendelkezésre állási zóna különálló áramforrással, hálózattal és hűtéssel rendelkezik. Ha a virtuális gépeket több rendelkezésre állási zónában helyezi üzembe, azzal segít megvédeni az alkalmazásokat a teljes adatközpontra kiterjedő meghibásodásokkal szemben. Nem mindegyik régió támogatja a rendelkezésreállási zónákat. A támogatott régiók és szolgáltatások listáját lásd [az Azure rendelkezésreállási zónáit](/azure/availability-zones/az-overview) ismertető részben.

Ha rendelkezésreállási zónák használatát tervezi az üzemelő példányban, először ellenőrizze, hogy az alkalmazás architektúrája és kódbázisa támogatja-e ezt a konfigurációt. Ha kereskedelmi forgalomban kapható, használatra kész szoftvert helyez üzembe, forduljon a szoftver gyártójához, és tesztelje alaposan a szoftvert az éles üzembe helyezés előtt. Egy alkalmazásnak fenn kell tartania az állapotot, és meg kell akadályoznia az adatvesztést a konfigurált zónában tapasztalható szolgáltatáskimaradás során. Az alkalmazásnak alkalmasnak kell lennie egy rugalmas és elosztott infrastruktúrában való futtatásra anélkül, hogy a kódbázisban nem módosítható infrastruktúra-összetevők lennének megadva.

**Azure Site Recovery**.  Replikálhatja az Azure-beli virtuális gépeket egy másik Azure-régióba az üzletmenet-folytonosság és a vészhelyreállítás érdekében. Időszakos vészhelyreállítási tesztekkel biztosíthatja a követelményeknek való megfelelést. A rendszer a megadott beállításokkal replikálja a virtuális gépet a kiválasztott régióba, így Ön visszaállíthatja alkalmazásait, ha a forrásrégióban leállás történne. További információt az [Azure-beli virtuális gépek ASR használatával történő replikálását ismertető részben][site-recovery] talál. Vegye figyelembe a szolgáltatás itt található RTO- és RPO-számát, és a teszteléskor győződjön meg arról, hogy a helyreállítási idő és a helyreállítási pont megfelel az igényeinek.

**Párosított régiók**. Annak érdekében, hogy védelmet nyújtson egy alkalmazás számára regionális kimaradás esetén, helyezze üzembe az alkalmazást egyszerre több régióban, és az Azure Traffic Manager használatával ossza el az internetes forgalmat a különböző régiók között. Minden egyes Azure-régió párban áll egy másikkal. Ezek együtt egy [régiópárt](/azure/best-practices-availability-paired-regions) alkotnak. Dél-Brazília kivételével a régiópárok azonos földrajzi helyen találhatók, hogy adóügyi és törvényi szempontból megfeleljenek az adatok tárolási helyére vonatkozó előírásoknak.

Amikor többrégiós alkalmazásokat tervez, vegye figyelembe, hogy a régiók közötti hálózati késés magasabb, mint egy régión belül. Például, ha egy adatbázist replikál a feladatátvétel támogatása érdekében, használjon szinkron adatreplikációt a régión belül, míg aszinkron adatreplikációt a régiók között.

| &nbsp; | Rendelkezésre állási csoport | Rendelkezésre állási zóna | Azure Site Recovery/párosított régió |
|--------|------------------|-------------------|---------------|
| Hiba hatóköre | Kiszolgálószekrény | Adatközpont | Régió |
| Kérések útválasztása | Load Balancer | Zónák közötti terheléselosztó | Traffic Manager |
| Hálózati késleltetés | Nagyon alacsony | Alacsony | Közepes vagy magas |
| Virtuális hálózat  | VNet | VNet | Régiók közötti virtuális hálózatok közötti társviszony |

## <a name="implement-resiliency-strategies"></a>Rugalmassági stratégiák megvalósítása

Ebben a szakaszban áttekintünk néhány gyakori rugalmassági stratégiát. Ezek legtöbbje az alkalmazott technológiától függetlenül működik. Az ebben a szakaszban található leírások az egyes technikák általános működési elvét foglalják össze, további információkra mutató hivatkozásokkal.

**Újrapróbálkozás átmeneti meghibásodások esetén**. Az átmeneti meghibásodások oka lehet a hálózati kapcsolat pillanatnyi megszakadása, az adatbázis-kapcsolat megszakadása, vagy foglalt szolgáltatás miatti időtúllépés. Az átmeneti meghibásodások gyakran megoldódnak a kérelem újbóli elküldésével. Sok Azure-szolgáltatás esetében az ügyfél SDK automatikus újrapróbálkozásokat valósít meg, a hívó számára átlátható módon; lásd [Szolgáltatásspecifikus újrapróbálkozásokkal kapcsolatos útmutatás][retry-service-specific guidance].

Mindegyik újrapróbálkozás növeli a teljes késést. Túl sok sikertelen kérelem továbbá szűk keresztmetszetet eredményezhet, mivel a függőben lévő kérelmek halmozódnak az üzenetsorban. Ezek a blokkolt kérelmek olyan kritikus rendszererőforrásokat akadályozhatnak, mint a memória, szálak, adatbázis-kapcsolatok stb., ezáltal pedig kaszkádolt meghibásodásokhoz vezethetnek. Ennek elkerülése végett növelni kell az egyes újrapróbálkozások közötti időt, és korlátozni a sikertelen kérelmek lehetséges számát.

![Újrapróbálkozási kísérletek ábrája](./images/retry.png)

**Terheléselosztás példányok között**. A skálázhatóság érdekében a felhőalkalmazásoknak képesnek kell lenniük a további példányok hozzáadása révén történő horizontális felskálázásra. Ez a módszer is növeli a rugalmasságot, mert a nem megfelelő állapotú példányokat ki lehet iktatni a rotációból. Például:

- Helyezzen két vagy több virtuális gépet terheléselosztó mögé. A terheléselosztó az összes virtuális gép között osztja el a forgalmat. Lásd az [elosztott terhelésű virtuális gépek üzemeltetése a skálázhatóság és a rendelkezésre állás érdekében][ra-multi-vm] témakörrel foglalkozó részt.
- Skálázzon fel horizontálisan egy Azure App Service alkalmazást több példányra. Az App Service automatikusan elosztja a terhelést a példányok között. Lásd: [Alapszintű webalkalmazás][ra-basic-web].
- Az [Azure Traffic Managerrel][tm] ossza el a forgalmat végpontok között.

**Adatok replikálása**. Az adatok replikálása egy általános stratégia, amellyel az adattárak nem átmeneti meghibásodásai kezelhetők. Számos tárolótechnológia – mint például az Azure Storage, az Azure SQL Database, a Cosmos DB és az Apache Cassandra – beépített replikálással rendelkezik. Az olvasási és írási útvonalakra egyaránt gondolni kell. A tárolótechnológiától függően rendelkezhet több írható replikával, vagy egy írható és több írásvédett replikával.

A rendelkezésre állás maximalizálása érdekében a replikákat több régióban is el lehet helyezni. Ez azonban növeli az adatok replikálásával járó késést. A régiók közötti replikálás általában aszinkron módon történik, ami végleges konzisztenciájú modellt jelent, valamint azt, hogy egy replika meghibásodása adatveszteséggel járhat.

Az [Azure Site Recoveryvel][site-recovery] Azure-beli virtuális gépeket replikálhat egy régióból egy másikba. A Site Recovery folyamatosan végzi az adatok replikálását a célrégióba. Ha az elsődleges helyen kimaradás történik, a rendszer átadja a feladatokat a másodlagos helynek

**Leállás nélküli teljesítménycsökkenés**. Ha egy szolgáltatás meghibásodik, és nincs feladatátvételi útvonal, az alkalmazás képes lehet leállás nélküli teljesítménycsökkenéssel tovább futni, továbbra is elfogadható felhasználói élményt nyújtva. Például:

- Helyezzen egy munkaelemet üzenetsorba későbbi kezelés céljából.
- Adjon vissza becsült értéket.
- Használjon helyi gyorsítótárban tárolt adatokat.
- Jelenítsen meg hibaüzenetet a felhasználó számára. (Ez jobb lehetőség, mint ha az alkalmazás nem válaszol a további kérelmekre.)

**Nagy forgalmú felhasználók szabályozása**. Előfordulhat, hogy néhány felhasználó túlzott terhelést okoz. Ez más felhasználókra is hatással lehet, csökkentve az alkalmazás teljes rendelkezésre állását.

Ha egyetlen ügyfél túlzott mennyiségű kérelmet küld, az alkalmazás bizonyos ideig szabályozhatja a felhasználó forgalmát. A szabályozási időszak alatt az alkalmazás visszautasítja az adott ügyféltől érkező néhány vagy összes kérelmet (a konkrét szabályozási stratégiától függően). A szabályozási küszöbérték függhet az ügyfél szolgáltatásszintjétől.

A szabályozás nem feltétlenül jelent kártételi szándékot az ügyfél részéről – csupán azt jelenti, hogy túllépte a szolgáltatási kvótáját. Egyes esetekben a fogyasztók rendszeresen túlléphetik a kvótájukat, vagy egyéb helytelen magatartást tanúsíthatnak. Ilyen esetben további lépésként lehetséges a felhasználó blokkolása. Ez általában API-kulcs vagy IP-címtartomány blokkolása révén történik. További információkért lásd: [Szabályozási minta][throttling-pattern].

**Áramköri megszakító használata**. Az [áramköri megszakítási][circuit-breaker-pattern] minta meggátolhatja, hogy egy alkalmazás olyan műveletet próbáljon többször végrehajtani, amely nagy eséllyel lesz sikertelen. Az áramköri megszakító egy szolgáltatás meghívásait burkolja, és nyomon követi a nemrégiben történt hibákat. Ha a hibaszám túllépi a küszöbértéket, az áramköri megszakító egy hibakódot ad vissza a szolgáltatás hívása nélkül. Ez időt biztosít a szolgáltatásnak, hogy helyreálljon.

**Adatforgalmi kiugrások csökkentése terheléskiegyenlítéssel**.
Az alkalmazások adatforgalmának esetleges hirtelen kiugrásai túlterhelhetik a háttérkiszolgálón futó szolgáltatásokat. Ha egy háttérszolgáltatás nem képes elég gyorsan válaszolni a kérelmekre, az sorban állásra kényszerítheti a kérelmeket, vagy az alkalmazás szolgáltatás általi szabályozását válthatja ki. Ennek elkerülése végett az üzenetsor pufferként használható. Új munkaelem megjelenésekor a háttérszolgáltatás azonnali meghívása helyett az alkalmazás aszinkron futáshoz állít sorba egy munkaelemet. Az üzenetsor pufferként működik, amely csökkenti a terhelési kiugrásokat. További információkért lásd az [üzenetsor-alapú terheléskiegyenlítési mintával][load-leveling-pattern] foglalkozó részt.

**Kritikus erőforrások elkülönítése**. Egy alrendszerben bekövetkező meghibásodás időnként kaszkádolás révén elterjedhet, az alkalmazás más részein is meghibásodásokat okozva. Ilyen akkor történhet, ha egy hiba miatt bizonyos erőforrások, mint például szálak vagy szoftvercsatornák, nem szabadulnak fel időben, ami erőforrásfogyáshoz vezet.

Ennek elkerülése végett a rendszer elkülönített csoportokra particionálható, aminek eredményeképpen az egy partícióban bekövetkező hiba nem vezet a teljes rendszer leállásához. Ezt a módszert Válaszfal mintának is szokás nevezni.

Példák:

- Particionáljon egy adatbázist (például bérlőnként), és mindegyik partícióhoz rendelje webkiszolgáló-példányok külön készletét.  
- Használjon külön szálkészleteket a különböző szolgáltatások meghívásainak elkülönítésére. Ez segít meggátolni a kaszkádolás révén terjedő hibákat, ha az egyik szolgáltatás meghibásodik. Példaként tekintse meg a Netflix [Hystrix könyvtárát][hystrix].
- [Tárolók][containers] felhasználásával korlátozza az egyes alrendszerek által elérhető erőforrások mennyiségét.

![A Válaszfal minta ábrája](./images/bulkhead.png)

**Kompenzáló tranzakciók alkalmazása**. A [kompenzáló tranzakciók][compensating-transaction-pattern] más, befejezett tranzakciók hatásait szüntetik meg. Az elosztott rendszerekben rendkívül nehéz lehet az erős tranzakció-konzisztencia megvalósítása. A kompenzáló tranzakciók révén megvalósítható a konzisztencia azáltal, hogy kisebb, különálló tranzakciókat használunk, amelyek mindegyike visszavonható.

Például utazásfoglalás esetében az ügyfél foglalhat autót, hotelszobát és repülőjegyet. Ha e lépések bármelyike meghiúsul, az egész művelet meghiúsul. Az egész művelet egyetlen elosztott tranzakcióval történő megvalósítása helyett definiálható kompenzáló tranzakció mindegyik lépéshez. Az autófoglalás visszavonásához például törölheti a foglalást. A teljes művelet befejezése érdekében koordinátor hajtja végre az összes lépést. Ha valamelyik lépés meghiúsul, a koordinátor kompenzáló tranzakciókkal vonja vissza a már befejezett lépéseket.

## <a name="test-for-resiliency"></a>Rugalmasság tesztelése

A rugalmasságot általában véve nem lehet ugyanúgy tesztelni, mint az alkalmazások működőképességét (egységtesztek futtatásával stb). Ehelyett azt kell megvizsgálni, hogyan teljesít a végpontok közötti munkaterhelés csak időnként jelentkező hibafeltételek mellett.

A tesztelés iteratív folyamat. Tesztelje az alkalmazást, mérje meg az eredményt, elemezze és kezelje az esetleges kapott hibákat, majd ismételje meg a folyamatot.

**Tesztelés hibabeszúrással**. Tesztelje a rendszer meghibásodásokkal szembeni rugalmasságát valódi hibák kiváltása vagy azok szimulálása révén. Alább található néhány gyakori meghibásodási helyzet:

- Állítson le virtuálisgép-példányokat.
- Váltsa ki egyes folyamatok összeomlását.
- Alkalmazzon lejárt tanúsítványokat.
- Módosítson hozzáférési kulcsokat.
- Állítsa le a DNS-szolgáltatást a tartományvezérlőkön.
- Korlátozza az elérhető rendszererőforrásokat, mint például a RAM-ot vagy a szálak számát.
- Válasszon le lemezeket.
- Helyezzen ismételten üzembe egy virtuális gépet.

Mérje meg a helyreállítási időtartamokat, és győződjön meg róla, hogy azok megfelelnek az üzleti követelményeinek. Több hibaállapot kombinációját is tesztelje. Ügyeljen a hibák elkülönített kezelésére, és a kaszkádolásos terjedésük kiküszöbölésére.

Ez is egy érv a lehetséges hibapontok tervezési fázis során történő elemzése mellett. Az elemzés eredményei szolgálhatnak a tesztelési terve bemeneteiként.

**Terheléses tesztelés**. A terheléses tesztelés kulcsfontosságú a csak terhelés alatt bekövetkező hibák, mint például a háttéradatbázis túlterhelése vagy a szolgáltatásszabályozás azonosítása szempontjából. A csúcsterhelést tesztelje, termelési adatok vagy a termelési adatokhoz lehető legközelebb álló szintetikus adatok használatával. A cél annak megállapítása, hogyan viselkedik az alkalmazás valós körülmények között.

**Vészhelyreállítási próbák**. A jó vészhelyreállítás terv önmagában nem elegendő. Rendszeres tesztelés is szükséges, hogy a helyreállítási terv biztosan jól működjön, amikor szükség van rá. Az [Azure Site Recoveryvel][site-recovery] replikálhatja az Azure-beli virtuális gépeket, és [vészhelyreállítási próbákat][site-recovery-test-failover] hajthat végre az üzemi alkalmazások vagy a folyamatban lévő replikáció zavarása nélkül.

## <a name="deploy-using-reliable-processes"></a>Üzembe helyezés megbízható folyamatokkal

Miután egy alkalmazás éles telepítése megtörtént, a frissítések hibaforrások lehetnek. A legrosszabb esetben egy rossz frissítés szolgáltatáskiesést okozhat. Ennek elkerülése végett az üzembehelyezési folyamatnak kiszámíthatónak és ismételhetőnek kell lennie. A telepítés magában foglalja az Azure-erőforrások kiépítését, az alkalmazáskód telepítését és a konfigurációs beállítások alkalmazását. A frissítések ezek némelyikére vagy mindegyikére terjedhetnek ki.

A lényeg, hogy a manuális telepítés során gyakran fordul elő hiba – emiatt ajánlott automatizált, idempotens folyamat alkalmazása, amely igény szerint futtatható, meghibásodás esetén pedig újból futtatható.

- Automatizálja az Azure-erőforrások kiépítését Azure Resource Manager-sablonokkal.
- Használja az [Azure Automation célállapot-konfigurációt][dsc] (DSC) a virtuális gépek konfigurálásához.
- Használjon automatizált üzembehelyezési folyamatot az alkalmazáskódhoz.

Az *infrastruktúra mint kód* és a *megváltoztathatatlan infrastruktúra* két, rugalmas üzembe helyezéshez kapcsolódó fogalom.

- Az **infrastruktúra mint kód** az infrastruktúra kód használatával történő kiépítését és konfigurálását jelenti. Az infrastruktúra mint kód megvalósítható deklaratív vagy imperatív módszerrel (vagy a kettő kombinációjával). A Resource Manager-sablonok használata a deklaratív módszer egy példája. A PowerShell-parancsprogramok használata az imperatív módszer egy példája.
- A **megváltoztathatatlan infrastruktúra** elve szerint az éles üzembe helyezés után nem szabad módosítani az infrastruktúrát. Ellenkező esetben előállhat olyan helyzet, amelyben alkalmi változtatások alkalmazása megnehezíti annak megállapítását, hogy pontosan mi változott, valamint a rendszerre vonatkozó döntések meghozatalát.

Kérdéses lehet még az alkalmazásfrissítések bevezetésének mikéntje. Ajánlott a kék-zöld üzembe helyezés vagy a tesztcsoportos kiadások használata; mindkét módszer a frissítések szigorúan ellenőrzött bevezetését alkalmazza az esetleges rossz üzembe helyezés hatásainak minimalizálása céljából.

- [A kék-zöld üzembe helyezés][blue-green] módszere esetében a rendszer az élő alkalmazástól elkülönített éles környezetben helyezi üzembe a frissítést. Az üzemelő példány ellenőrzése után irányítsa át a forgalmat a frissített verzióhoz. Ez például az Azure App Service Web Appsban előkészítési pontok használatával lehetséges.
- [A tesztcsoportos kiadások][canary-release] a kék-zöld üzembe helyezéshez hasonlóan működnek. A teljes forgalom frissített verzióhoz való átirányítása helyett csak a felhasználók egy kis csoportja számára kell bevezetni a frissítést, a forgalom egy részét irányítva át az új üzemelő példányhoz. Ha probléma merül fel, váltson vissza a régi üzemelő példányra. Ellenkező esetben fokozatosan irányítsa át a forgalom többi részét az új verzióhoz, amíg az adatforgalom 100%-a át nem kerül.

Bármelyik módszert is választja, biztosítsa a legutóbbi biztosan jól működő üzemelő példányhoz való visszatérés lehetőségét arra az esetre, ha az új verzió nem működne. Továbbá ha hiba lép fel, az alkalmazásnaplóknak jelezniük kell, melyik verzió okozta a hibát.

## <a name="monitor-to-detect-failures"></a>Monitorozás a hibák észlelése érdekében

A megfigyelés és a diagnosztika létfontosságú a rugalmasság szempontjából. Ha hiba lép fel, tudnia kell róla, hogy hiba történt, és a hiba kiváltó okairól szóló információkra is szüksége van.

A kiterjedt, elosztott rendszerek megfigyelése meglehetősen nehéz. Vegyünk például egy néhány tucat virtuális gépen futó alkalmazást – nem célszerű minden egyes virtuális gépre bejelentkezni, és egyenként végignézni a naplófájlokat a hiba elhárítása céljából. Valószínű, hogy a virtuális gépek példányszáma sem statikus. A virtuális gépek száma nőhet vagy csökkenhet az alkalmazás skálázása során, időnként pedig meghibásodhat egy-egy példány, és az újbóli kiépítésére lehet szükség. Ezen felül a felhőalkalmazások általában több adattárral (Azure Storage, SQL Database, Cosmos DB, Redis Cache) dolgoznak, és egyetlen felhasználói művelet több alrendszerre is kiterjedhet.

A megfigyelés és diagnosztika folyamata több különálló szakaszból áll:

![Összetett SLA](./images/monitoring.png)

- **Rendszerállapot**. A megfigyeléshez és diagnosztikához szükséges nyers adatok különböző forrásokból származhatnak, beleértve az alkalmazásnaplókat, webkiszolgáló-naplókat, az operációs rendszer teljesítményszámlálóit, adatbázisnaplókat és az Azure-platformba beépített diagnosztikákat. A legtöbb Azure-szolgáltatás rendelkezik diagnosztikai funkcióval, amely a problémák okainak feltárására használható.
- **Gyűjtés és tárolás**. A nyers rendszerállapot-megfigyelési adatokat különböző helyeken és formátumokban tárolhatja a rendszer (pl.: alkalmazáskövetési naplók, IIS-naplók, teljesítményszámlálók). E különálló forrásokat össze kell gyűjteni, összesíteni kell, és megbízható tárolóban kell elhelyezni.
- **Elemzés és diagnosztika**. Az adatok összesítése után, az elemzésük révén végezhető el a hibaelhárítás és tekinthető át az alkalmazás állapota.
- **Megjelenítés és riasztások**. Ebben a szakaszban a telemetriaadatok olyan formában jelennek meg, amelyben a kezelő gyorsan észre tudja venni a problémákat vagy tendenciákat. Ilyenek például az irányítópultok vagy az email-riasztások.  

A megfigyelés nem ugyanaz, mint a hibaészlelés. Egy alkalmazás például észlelhet átmeneti hibát, és újból próbálkozhat, szolgáltatáskiesés nélkül. Viszont naplóznia is kell az újrapróbálkozás műveletét, ami által megfigyelhető a hiba gyakorisága, és általános képet kaphat az alkalmazás állapotáról.

Az alkalmazásnaplók a diagnosztikai adatok fontos forrását képezik. Ajánlott eljárások az alkalmazásnaplózáshoz:

- Naplózzon éles környezetben. E nélkül a leginkább szükséges információkról maradhat le.
- Naplózza a szolgáltatáshatárokon történő eseményeket. Használjon korrelációs azonosítót, amely átnyúlik a szolgáltatáshatárokon. Ha egy tranzakció több szolgáltatáson nyúlik keresztül, és az egyikük meghibásodik, a korrelációs azonosító segít megtalálni a tranzakció meghiúsulásának okát.
- Használjon szemantikus naplózást, más néven strukturált naplózást. A strukturálatlan naplók megnehezítik a naplóadatok használatának és elemzésének automatizálását, amelyre felhőszinten szükség van.
- Használjon aszinkron naplózást. Egyéb esetben maga a naplózórendszer is okozhatja az alkalmazás meghibásodását azáltal, hogy a kérelmek felgyülemlését okozza, mivel a naplózási esemény írására való várakozás blokkolja a többi kérelmet.
- Az alkalmazásnaplózás nem ugyanaz, mint a naplózás (auditing). A naplózás megfelelőségi vagy jogszabályi megfeleléssel kapcsolatos célból történik. Emiatt a naplózási rekordoknak teljesnek kell lenniük, és nem fogadható el az elvetésük a tranzakciók feldolgozása során. Ha egy alkalmazás működését naplózni kell, a naplózást el kell választani a diagnosztikai naplózástól.

A megfigyeléssel és diagnosztikával kapcsolatos további információkért lásd a [megfigyelési és diagnosztikai útmutatót][monitoring-guidance] tartalmazó szakaszt.

## <a name="respond-to-failures"></a>Válasz a hibákra

A korábbi szakaszok az automatizált helyreállítási stratégiákról szóltak, amelyek létfontosságúak a magas rendelkezésre állás szempontjából. Időnként azonban kézi beavatkozás szükséges.

- **Riasztások**. Kísérje figyelemmel az alkalmazása olyan figyelmeztető jeleit, amelyek proaktív beavatkozást igényelhetnek. Ha például az SQL Database vagy a Cosmos DB rendszeresen szabályozza az alkalmazását, lehetséges, hogy növelnie kell az adatbázis-kapacitását, vagy optimalizálnia kell a lekérdezéseit. Ebben a példában, bár az alkalmazás képes átláthatóan kezelni a szabályozási hibákat, a telemetria riasztást küld, hogy nyomon követhesse az eseményeket.  
- **Manuális feladatátvétel**. Egyes rendszerek nem tudják automatikusan átadni a feladataikat. Az ilyen esetekben manuális feladatátvételre van szükség. Az [Azure Site Recoveryvel][site-recovery] konfigurált Azure-beli virtuális gépeken [feladatátvételt hajthat végre][site-recovery-failover], és percek alatt helyreállíthatja a virtuális gépeit egy másik régióban.
- **Működési készenlét tesztelése**. Ha az alkalmazás egy másodlagos régióba adja át a feladatait, akkor tesztelje a működési készenlétet, mielőtt visszaadja a feladatokat az elsődleges régióba. A teszt ellenőrzi, hogy az elsődleges régió állapota megfelelő-e, és készen áll-e a forgalom újbóli fogadására.
- **Adatkonzisztencia-ellenőrzés**. Ha egy adattár meghibásodik, adatinkonzisztencia jelentkezhet, amikor a tároló ismét elérhetővé válik. Ez főleg akkor jellemző, ha a rendszer replikálta az adatokat.
- **Visszaállítás biztonsági másolatból**. Ha például az SQL Database-ben regionális leállás történik, a georedundáns visszaállítással helyre tudja állítani az adatbázist a legutóbbi biztonsági másolatból.

Dokumentálja és tesztelje a vészhelyreállítási tervet. Értékelje az alkalmazáshibák üzletmenetre gyakorolt hatását. A lehető legnagyobb mértékben automatizálja a folyamatot, és dokumentálja az összes manuális lépést, például a manuális feladatátvételt vagy az adatok biztonsági másolatból történő visszaállítását. A terv ellenőrzéséhez és továbbfejlesztéséhez rendszeresen tesztelje a vészhelyreállítási folyamatot.

## <a name="summary"></a>Összegzés

A cikk holisztikus szempontból tárgyalta a rugalmasságot, és kiemelte a felhőalapú környezet néhány egyedi kihívását. Ezek közé tartozik a felhőalapú számítógép-használat elosztott jellege, a hagyományos hardverek használata és az átmeneti hálózati hibák jelenléte.

A cikk lényeges pontjai a következők:

- A rugalmasságnak köszönhetően magasabb lesz a rendelkezésre állás, és átlagosan kevesebb idő szükséges a hibák utáni helyreálláshoz.
- A rugalmasság elérése a felhőben más technikákat igényel, mint a hagyományos helyszíni megoldások esetében.
- A rugalmasság nem jön létre magától. Meg kell tervezni, és már a kezdetektől be kell építeni.
- A rugalmasság az alkalmazás életciklusának minden részére kihat, a tervezéstől a kódoláson át egészen a műveletekig.
- Teszteljen és monitorozzon!

<!-- links -->

[blue-green]: https://martinfowler.com/bliki/BlueGreenDeployment.html
[canary-release]: https://martinfowler.com/bliki/CanaryRelease.html
[circuit-breaker-pattern]: https://msdn.microsoft.com/library/dn589784.aspx
[compensating-transaction-pattern]: https://msdn.microsoft.com/library/dn589804.aspx
[containers]: https://en.wikipedia.org/wiki/Operating-system-level_virtualization
[dsc]: /azure/automation/automation-dsc-overview
[contingency-planning-guide]: https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-34r1.pdf
[fma]: failure-mode-analysis.md
[hystrix]: https://medium.com/netflix-techblog/introducing-hystrix-for-resilience-engineering-13531c1ab362
[jmeter]: https://jmeter.apache.org/
[load-leveling-pattern]: ../patterns/queue-based-load-leveling.md
[monitoring-guidance]: ../best-practices/monitoring.md
[ra-basic-web]: ../reference-architectures/app-service-web-app/basic-web-app.md
[ra-multi-vm]: ../reference-architectures/virtual-machines-windows/multi-vm.md
[checklist]: ../checklist/resiliency.md
[retry-pattern]: ../patterns/retry.md
[retry-service-specific guidance]: ../best-practices/retry-service-specific.md
[sla]: https://azure.microsoft.com/support/legal/sla/
[throttling-pattern]: ../patterns/throttling.md
[tm]: https://azure.microsoft.com/services/traffic-manager/
[tm-failover]: /azure/traffic-manager/traffic-manager-monitoring
[tm-sla]: https://azure.microsoft.com/support/legal/sla/traffic-manager
[site-recovery]:/azure/site-recovery/azure-to-azure-quickstart/
[site-recovery-test-failover]:/azure/site-recovery/azure-to-azure-tutorial-dr-drill/
[site-recovery-failover]:/azure/site-recovery/azure-to-azure-tutorial-failover-failback/
