---
title: 'Az Azure virtual datacenter: A hálózati nézőpont'
description: Ismerje meg, hogyan hozhat létre az Azure-beli virtuális adatközpont
author: tracsman
manager: rossort
tags: azure-resource-manager
ms.service: virtual-network
ms.date: 09/24/2018
ms.author: jonor
ms.openlocfilehash: b10930ac6d6458872d8b626825d21bd0bed2b807
ms.sourcegitcommit: 16bc6a91b6b9565ca3bcc72d6eb27c2c4ae935e4
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/28/2018
ms.locfileid: "52550461"
---
# <a name="azure-virtual-datacenter-a-network-perspective"></a>Az Azure virtual datacenter: A hálózati nézőpont

## <a name="overview"></a>Áttekintés
A helyszíni alkalmazások az Azure-ba való migrálás, jelentős módosítások nélkül is biztosít szervezetek egy biztonságos és költséghatékony infrastruktúra előnyeit. Ezt a módszert nevezik **lift- and -shift**. Ahhoz, hogy a legtöbbet a rugalmasságot lehetséges a felhő-számítástechnika, a vállalatok számára az Azure-képességek kihasználásához architektúrák kell alakítani. A Microsoft Azure biztosít a nagy kapacitású szolgáltatások és infrastruktúra, vállalati szintű képességet és megbízhatóságot és számos választási való hibrid kapcsolathoz. 

Ügyfeleink kiválaszthatják hozzá vagy az interneten keresztül, vagy az Azure expressroute használatával privát hálózati kapcsolatot biztosít a felhőalapú szolgáltatásokhoz. A Microsoft Azure platformon, az ügyfelek zökkenőmentesen infrastruktúrán kiterjesztheti a felhőbe és többrétegű architektúrákat hozhat létre. Microsoft-partnerek fejlett funkciókat is biztosítanak azzal biztonsági szolgáltatások és virtuális berendezések az Azure-beli futtatására van optimalizálva.

Ez a cikk áttekintést nyújt minták és a terveket, amelyeket az architekturális méretezés, teljesítmény és biztonsági problémák megoldásához számos ügyfelek face során, gondolja át a felhőbe való áthelyezés masse menet. A rendszer a cégirányítási és hogyan illeszkednek a különböző szervezeti informatikai szerepköröket, a felügyelet alá áttekintését is szakasz tárgyalja. Kiemelés biztonsági követelmények és költségek optimalizálása be van kapcsolva.

## <a name="what-is-a-virtual-datacenter"></a>Mi az a virtuális adatközpont?
Felhőalapú megoldások első tervezték, hogy a nyilvános spektrum egyetlen, viszonylag elkülönített alkalmazások üzemeltetéséhez. Ez a módszer jól években dolgozott. Nyilvánvalóvá vált, majd a felhőalapú megoldások előnyeit, és több nagy méretű számítási feladatokat a felhőben is üzemeltethetők. Létfontosságú teljes életciklusa a felhőszolgáltatás-címzés biztonságra, megbízhatóságra, teljesítmény és költség aggályokat egy vagy több régióban üzemelő vált.

Az alábbi ábrán a felhőbeli üzembe helyezés biztonsági eseményáramlási kimaradást egy példát mutat be a **vörös**. A **sárga keretet** számítási feladatok hálózati virtuális berendezések optimalizálás szoba mutatja.

[![0]][0]

Virtual datacenter (VDC) született alól, méretezhető nagyvállalati számítási feladatok támogatására. A problémák vezetett be, amikor nagy léptékű alkalmazások támogatása a nyilvános felhőben is foglalkozik.

A VDC nem csupán az alkalmazás a felhőalapú számítási feladatokat. Ez a hálózati, biztonsági, felügyeleti és infrastruktúra is. Példák a DNS és a directory services. Általában biztosít egy privát kapcsolaton keresztül érik el egy helyszíni hálózat vagy datacenter. Ahogy egyre több számítási feladatok Azure-bA áthelyezni, fontos gondolja át a támogató infrastruktúra és az objektumok, amelyek az ilyen munkaterhelések vannak elhelyezve. Gondoljunk gondosan hogyan erőforrások struktúrája elkerülése érdekében több száz elterjedése **munkaterhelés szigetek** , amely külön-külön kell kezelni a független adatfolyam, modellek és megfelelőségi kihívások.

A virtuális adatközpont különálló, de a kapcsolódó entitások a gyakori támogatási funkciók, szolgáltatások és infrastruktúra gyűjteménye. Egy integrált VDC, a számítási feladatok megtekintésével méretgazdaságosság csökkentheti költségeit, és optimalizált összetevő- és a folyamat forrásadattárakból biztonságát. Egyszerűbb operations, felügyeleti és megfelelőségi ellenőrzéseket is kap.

> [!NOTE]  
> Fontos tisztában lenni azzal, hogy a VDC **nem** egy különálló Azure-termék. Különféle funkciók és képességek a pontos követelményeinek együttes használata. A VDC módja a szem előtt tartva a számítási feladatok és az Azure-használat, maximalizálhatja az erőforrások és a képességek a felhőben. A virtuális adatközpont éppen ezért a moduláris megközelítés hogyan építse fel az IT-szolgáltatások az Azure-ban, tiszteletben szervezeti szerepkörökről és feladatokról.

A VDC segítségével a vállalatok munkaterheléseket és alkalmazásokat az Azure-bA lekérése a következő forgatókönyvekhez:

-   A gazdagép több kapcsolódó számítási feladatot.
-   A helyszíni környezet a számítási feladatok migrálása az Azure-bA.
-   Számítási feladatok megvalósítása a megosztott vagy központi biztonsági és a hozzáférési követelményeket.
-   Az Azure DevOps és a központi informatikai nagyvállalatok számára megfelelően.

A magánfelhőmodell előnyeit a VDC feloldásához kulcsa egy központosított topológia, küllős topológia, az Azure-szolgáltatások kombinációját: 

- [Az Azure Virtual Network][VNet]. 
- [Hálózati biztonsági csoportok (NSG-k)][NSG].
- [Virtuális hálózatok közötti társviszony][VNetPeering]. 
- [Felhasználó által megadott útvonalak (udr-EK)][UDR].
- Azure-identitás szolgáltatásokhoz a [szerepköralapú hozzáférés-vezérlés (RBAC)][RBAC]. 
- Másik lehetőségként [Azure tűzfal][AzFW], [az Azure DNS][DNS], [Azure bejárati ajtajának] [ AFD], és [Azure virtuális WAN][vWAN].

## <a name="who-should-implement-a-virtual-datacenter"></a>Akik meg kell valósítania egy virtuális adatközpont?
Bármely Azure-ügyfél, amelyet több számítási feladatok áthelyezése az Azure-ba is kihasználhatják a közös erőforrások használatával. A méretétől függően akár egyetlen alkalmazások milyen előnyei származhatnak a minták és -összetevők a VDC összeállításához.

Ha a szervezete egy központi informatikai, hálózati, biztonsági vagy megfelelőségi csapat vagy részleg, a VDC is segít, hogy házirend pontok és a szétválasztást vámot. Emellett biztosítja az alapul szolgáló közös összetevők az egységesség lehető szabadon és a vezérlő az elvárásainak, miközben alkalmazásfejlesztő csapatok dolgoznak.

A szervezetek, keresse meg az Azure DevOps képes használni a VDC kapcsolatos fogalmakat, adja meg az Azure-erőforrások jogosult zsebe és teljes körű vezérlést, hogy a csoporton belül van. Csoportok előfizetések vagy erőforráscsoportok közös előfizetéshez tartoznak. De a hálózati és biztonsági határokat maradjon megfelelő, egy központi virtuális hálózaton, és az erőforráscsoport egy központosított szabályzatban meghatározottak szerint.

## <a name="considerations-when-you-implement-a-virtual-datacenter"></a>Egy virtuális adatközpont megvalósításának szempontok
A VDC tervezésekor több pivotal problémák vannak kell figyelembe venni:

-   Identitás- és címtárszolgáltatásokat.
-   Biztonsági infrastruktúra.
-   A felhővel való kapcsolat.
-   Kapcsolódás a felhőben.

### <a name="identity-and-directory-services"></a>Identitás- és címtárszolgáltatásokat
Identitás- és címtárszolgáltatásokat rendszer egyik fontos szempontja a minden adatközpontot, mind a helyszíni és a felhőben. Hozzáférés és -engedélyezést, a VDC szolgáltatásai minden aspektusát identitás kapcsolódik. Győződjön meg arról, hogy csak a jogosult felhasználók és a folyamatok elérése az Azure-fiók és -erőforrások, az Azure, számos különböző típusú hitelesítő adatokat használ. Ezek közé tartoznak a jelszavak az Azure-fiók, a kriptográfiai kulcsokat, a digitális aláírások és a tanúsítványok eléréséhez. 

[Az Azure multi-factor Authentication] [ MFA] van egy további Azure-szolgáltatások eléréséhez szükséges biztonsági réteget. Erős hitelesítés több egyszerű ellenőrzési lehetőség biztosít. Az ügyfelek eldönthetik tetszőlegesen választhatnak. A telefonhívás, szöveges üzenet vagy mobilalkalmazás-értesítés a lehetőségek között. 

Minden olyan nagyvállalati kell egy egyéni identitást és a hitelesítés, engedélyezés, szerepkörök és jogosultságok belül vagy között a VDC kezelését ismerteti identity management folyamat megadása. Ez a folyamat célja a biztonság és hatékonyság takarítható meg és költségeket, leállás és ismétlődő manuális feladatokat.

Vagy intézményen különböző üzletágak szolgáltatások erőforrás-igényű vegyesen lehet szükség. És az alkalmazottak gyakran eltérő szerepkörökkel, ha a különböző projektek szerepet. A VDC megfelelő együttműködését a különböző csapatok különböző, az adott szerepkör-definíciók, a rendszerekhez és a megfelelő cégirányítási fut minden egyes szükséges. A mátrix feladatkörei, hozzáférés és a jogok összetett folyamat lehet. Identitáskezelés a VDC keresztül valósul meg [Azure Active Directory (Azure AD)] [ AAD] és a szerepköralapú hozzáférés-vezérlés (RBAC).

Egy olyan megosztott információkat működő az megkeresi, kezeli, felügyeli és rendszerezi a mindennapi elemek és hálózati erőforrásokat. Ezeket az erőforrásokat a köteteket, mappák, fájlok, nyomtatók, felhasználók, csoportok, eszközök és más objektumok is tartalmazhat. A hálózaton lévő minden egyes erőforrás számít egy objektumot a directory-kiszolgáló által. Erőforrásra vonatkozó adatokat egy adott erőforrás vagy objektumhoz tartozó attribútumok gyűjteménye van tárolva.

Jelentkezzen be az Azure Active Directory (Azure AD) támaszkodik az összes Microsoft online szolgáltatás, és egyéb az identitásukat. Az Azure Active Directory egy átfogó, magas rendelkezésre állású, felhőalapú identitás- és hozzáférés-kezelő megoldás, amely ötvözi az alapvető címtárszolgáltatásokat, a fejlett identitáskezelést és az alkalmazáshozzáférés-felügyeletet. Az Azure AD integrálható a helyszíni Active Directory egyszeri bejelentkezés az összes felhőalapú és a helyileg üzemeltetett a helyszíni alkalmazások engedélyezéséhez. A helyszíni Active Directory felhasználói attribútumok automatikusan szinkronizálhatók az Azure ad-hez.

Egyetlen globális rendszergazdája nem szükséges minden engedélyeket a VDC-megvalósítás. Ehelyett minden egyes adott részleg vagy csoport, felhasználók vagy szolgáltatások a címtárszolgáltatás is rendelkezik a saját belül a VDC-erőforrások kezeléséhez szükséges engedélyeket. Engedélyek strukturálja terheléselosztást igényel. Túl sok engedély akadályozhatják teljesítmény hatékonyságát. Túl kevés vagy laza engedélyek növelje a biztonsági kockázatokat. Azure szerepköralapú hozzáférés-vezérlés (RBAC) segítségével részletes hozzáférés-vezérlést a VDC-erőforrások felajánlásával oldja meg a problémát.

#### <a name="security-infrastructure"></a>Biztonsági infrastruktúra
A biztonsági infrastruktúra forgalom elkülönítése a VDC adott virtuális hálózati szegmens és hogyan szabályozhatja bejövő és kimenő folyamatok során a VDC hivatkozik. Az Azure több-bérlős architektúra, amely megakadályozza a jogosulatlan vagy véletlen forgalom között üzemelő példányok alapján történik. Ez a virtuális hálózati elszigetelést alkalmaz, a hozzáférés-vezérlési listák (ACL), terheléselosztók, IP-szűrők és olyan forgalmi szabályzatok. Hálózati címfordítás (NAT) elkülöníti a belső hálózati forgalmat a külső forgalomtól.

Az Azure-hálót foglalja le az infrastruktúra-erőforrások bérlői számítási feladatok, és kezeli a kommunikációt egy telepítésen, és a virtuális gépek (VM). Az Azure hipervizor előírja a memóriák és a folyamatok elkülönítését virtuális gépek között, és biztonságosan a útvonalakat hálózati forgalmat a vendég operációs rendszerek bérlőihez.

#### <a name="connectivity-to-the-cloud"></a>A felhővel való kapcsolat
Egy VDC szolgáltatásajánlatokhoz ügyfelek, partnerek és a belső felhasználók külső hálózatokkal való kapcsolatot igényel. Általában nem csak az internethez, hanem a helyszíni hálózatokhoz és adatközpontok kapcsolat van szüksége.

Ügyfelek vezérlőelem, mely szolgáltatások hozzáférése és elérhető-e a nyilvános internetről használatával [Azure tűzfal] [ AzFW] , vagy a más típusú virtuális hálózati berendezések (nva-k) egyéni házirendek szerint használatával [felhasználó által megadott útvonalak][UDR], és a hálózati szűrés használatával [hálózati biztonsági csoportok][NSG]. Azt javasoljuk, hogy az összes internetkapcsolattal rendelkező erőforrásokat is biztosítható a [Azure DDoS Protection Standard][DDOS].

Vállalatok gyakran kell a VDC-implementációk a helyszíni adatközpontok és más erőforrásokhoz kapcsolódni. Így az Azure és helyszíni hálózat közötti kapcsolat egy rendkívül fontos szempont egy hatékony architektúra tervezése során. Vállalatok között a VDC-megoldások és a helyszíni csatlakozás létrehozása az Azure-ban két különböző módja van: az interneten vagy a privát közvetlen kapcsolatok átvitel során.

Egy [az Azure site-to-site VPN] [ VPN] VDC végrehajtásuk a nyilvános interneten keresztül ki a vállalat a helyszíni hálózathoz csatlakozik. A VPN-kapcsolaton keresztül a forgalom IPSec és az IKE-bújtatás használatával van titkosítva. Az Azure site-to-site-kapcsolat, rugalmas és gyorsan hozhat létre. A VPN-t nem szükséges minden olyan további beszerzésekre, minden kapcsolat csatlakozzon az interneten keresztül.

A VPN-kapcsolatok, nagy számú [Azure virtuális WAN] [ vWAN] a hálózati szolgáltatás, amely optimalizált és automatizált ág ágba irányuló kapcsolat az Azure-on keresztül. Virtuális WAN-csatlakozhat, és az Azure-ral kommunikációhoz ág eszközök konfigurálása. Ez a kapcsolat manuálisan vagy egy virtuális WAN-partneren keresztül előnyben részesített szolgáltató eszközök használatával teheti meg. Előnyben részesített szolgáltató eszközök számára, könnyen használható, kapcsolat és a konfigurációkezelés egyszerűsítését. Az Azure-WAN beépített Irányítópult segítségével időt takaríthat meg, amely azonnali hibaelhárítási elemzéseket. És az irányítópult biztosít a nagy méretű hely – hely kapcsolat megtekintéséhez egyszerűen.

[Az ExpressRoute][ExR], egy kapcsolatot az Azure-szolgáltatás lehetővé teszi, hogy a vállalat a helyszíni hálózat és a végrehajtásuk VDC közötti privát kapcsolatok. Nagyobb biztonságot, a megbízhatóság és a nagyobb sebesség, legfeljebb 10 GB/s, konzisztens késés mellett kínál. Az ExpressRoute hasznos VDC megvalósításokhoz, a vevők az ExpressRoute privát kapcsolatok társított megfelelőségi szabályok előnyeinek kihasználása. [Az ExpressRoute közvetlen] [ ExRD] , közvetlenül csatlakozik a Microsoft útválasztói 100 Gbps sebességnél nagyobb sávszélességet igénylő ügyfelek számára.

Az ExpressRoute-kapcsolatok általában telepítése magában foglalja egy ExpressRoute-szolgáltató révén. A gyorsan elkezdheti a munkát igénylő ügyfelek először használja a site-to-site VPN a VDC közötti kapcsolatot létesíteni a gyakori és a helyszíni erőforrások. Majd telepítse át az ExpressRoute-kapcsolat a fizikai összekapcsolása a szolgáltatónál befejezésekor.

#### <a name="connectivity-within-the-cloud"></a>Kapcsolat a felhőben
[Virtuális hálózatok] [ VNet] és [virtuális hálózatok közötti társviszony] [ VNetPeering] belül a VDC alapvető hálózati kapcsolódási szolgáltatások. Virtuális hálózat a VDC-erőforrások elkülönítési határ természetes garantálja. És a virtuális hálózatok közötti társviszony lehetővé teszi, hogy a másik virtuális hálózat azonos Azure-régióban vagy régiók között is között befolyásolása. Forgalomvezérlés egy virtuális hálózaton belül és virtuális hálózatok igényeinek megfelelő hozzáférés-vezérlési listák keresztül megadott biztonsági szabályok között. Információinak megtekintéséhez [hálózati biztonsági csoportok][NSG], [hálózati virtuális berendezések (nva-k)][NVA], és [egyéni Útválasztás a felhasználó által megadott útvonalak táblák][UDR].

## <a name="virtual-datacenter-overview"></a>Virtuális adatközpont áttekintése

### <a name="topology"></a>Topológia
**Küllős topológiájú** egy olyan modell, egy adott Azure-régión belül egy virtuális adatközpont kiterjesztéséhez.

[![1]][1]

A központ egy, a központi hálózathoz zóna, amely szabályozza, és megvizsgálja a bejövő és kimenő forgalom különböző zónák között: az internet, a helyszíni és a küllők. A küllős topológia ad az IT-részleg hatékony módszert egy központi helyen biztonsági házirendeknek az érvényesítését. A Virtual Network szolgáltatás hibás és a lehetséges is csökkenti.

A hub tartalmazza az általános szolgáltatás-összetevők a küllők által igénybe vett. A következő példák olyan közös központi szolgáltatások:

-   A Windows Active Directory-infrastruktúrát, harmadik felek, amelyek nem megbízható hálózatokon elérését, mielőtt a számítási feladatok a küllőkben a felhasználói hitelesítés szükséges. Ez magában foglalja a kapcsolódó Active Directory összevonási szolgáltatások (AD FS).
-   A DNS-szolgáltatás megoldásához, ha a munkaterhelés a küllők hozzáférési erőforrások a helyszíni és az interneten lévő elnevezési [Azure DNS] [ DNS] nem használja a rendszer.
-   Nyilvános kulcsokra épülő infrastruktúrával (PKI), a számítási feladatok egyszeri bejelentkezés megvalósítása.
-   Átvitelvezérlés TCP és UDP-forgalmat a küllős hálózati zónák és az internet között.
-   Átvitelvezérlés helyszíni és a küllők között.
-   Ha szükséges, egy küllő és a egy másik között folyamatvezérlés.

A VDC csökkenti a teljes költség több küllők között megosztott hub infrastruktúra használatával.

A szerepkör minden egyes küllőhöz kapcsolni, a számítási feladatok különböző típusú gazdagép is lehet. A küllők is adja meg a moduláris megközelítés megismételhető üzembe helyezések az azonos számítási feladatok. Példák a fejlesztési és tesztelési, a felhasználói tesztelés, üzem előtti gyűjteményben, és éles. A küllők is őket, és a szervezeten belül a különböző csoportok engedélyezése. Ilyen például, az Azure DevOps-csoportok. Belül egy adott küllőre egy alapvető számítási feladatok vagy a rétegek közötti forgalom ellenőrzésével összetett, többrétegű munkaterhelések üzembe helyezéséhez.

#### <a name="subscription-limits-and-multiple-hubs"></a>Előfizetés korlátai, és több hubon
Az Azure-ban minden összetevő, a bármilyen típusú helyezünk üzembe az Azure-előfizetés. Különböző Azure-előfizetésekben található Azure-összetevőket elkülönítése különböző üzletágak követelményeinek is megfelelnek. Például hozzáférési és engedélyezési szinteket beállítását.

A VDC következő egyetlen megvalósítása a küllők nagy számú is vertikálisan. De minden informatikai rendszer-platformok korlátozva van. A központi telepítés egy adott Azure-előfizetéssel, korlátozások és korlátokat rendelkező van kötve. Ilyen például, virtuális társhálózati viszonyhoz maximális számát. További információkért lásd: [Azure-előfizetés és a szolgáltatások korlátozásai, kvótái és megkötései][Limits]. Azokban az esetekben, ahol korlátok problémát is előfordulhat, hogy az architektúra is vertikálisan tovább. A modell egyetlen Központ-küllő bővíthető küllős topológia a fürtre. Egy vagy több Azure-régióban több hubon összekapcsolható a virtuális hálózatok közötti társviszony, ExpressRoute, virtuális WAN vagy helyek közötti VPN használatával.

[![2]][2]

Több bevezetésével növeli a költségeket és a felügyeleti terhek, a rendszer. Csak indokolt volna méretezhetőséget, például a rendszer korlátok, és a redundancia és a végfelhasználói teljesítmény vagy a vész-helyreállítási például regionális replikáció. Forgatókönyvek a több hubon, az összes hubs igénylő törekedni kell a ugyanazokat a működési egyszerű szolgáltatásokat kínálnak.

#### <a name="interconnection-between-spokes"></a>Összekapcsolás küllők között
Egy egyetlen küllő belül összetett, többrétegű munkaterhelések megvalósításához. Többrétegű konfigurációk minden szinthez ugyanabban a virtuális hálózatban egy alhálózat használatával is alkalmazható. A folyamatokat szűrni az NSG-k használatával.

Egy fejlesztő szeretné többrétegű számítási feladatok üzembe több virtuális hálózaton. A virtuális hálózatok közötti társviszony, küllők más küllők az azonos hub-vagy különböző hubs segítségével csatlakozhat. Egy jellemző példa rá történik a kérelmet feldolgozó kiszolgálók esetén egy küllős, vagy a virtuális hálózat. Az adatbázis különböző küllő vagy virtuális hálózaton üzembe helyezi. Ebben az esetben is könnyen összekapcsolható a küllő virtuális hálózatok közötti társviszony-, és ezáltal elkerülheti a hubon áthaladó. Annak érdekében, hogy a hub megkerülése nem megkerülése, fontos biztonsági vagy naplózási pontokat, amelyek csak az agyban előfordulhat, hogy létezik egy gondos architektúra és biztonsági felülvizsgálat kell végrehajtani.

[![3]][3]

Küllők is összekapcsolható, egy adott küllőre, amely a kiszolgálókezelési. Ezzel a módszerrel hoz létre egy kétszintű hierarchia: a küllők magasabb szint, szint: 0, a level 1, a hierarchia alacsonyabb agyként hub válnak. A központi agyhoz vagy a helyszíni hálózat és internet bizalommal a forgalom továbbítása a VDC-végrehajtás a küllők van szükség. Hub két szinttel rendelkező vezet be, hogy bonyolult útválasztással, amely eltávolítja egy egyszerű küllős kapcsolat előnyeit.

Bár az Azure lehetővé teszi, hogy a komplex topológiáit, a VDC-fogalom alapelvek egyik ismételhetőség és az egyszerűség kedvéért. Energiabefektetést igénylő felügyelet minimalizálása érdekében az egyszerű küllős terv a VDC-referenciaarchitektúra, javasoljuk, hogy.

### <a name="components"></a>Összetevők
A virtuális adatközpont megvalósítás épül fel négy alapvető összetevő típusa: **infrastruktúra**, **peremhálózatokon**, **számítási feladatok**, és  **Figyelés**.

Minden egyes összetevő típusát különböző Azure-szolgáltatások és erőforrások áll. Egy vállalati VDC végrehajtása több összetevő-típust, és több változata létezik azonos típusú összetevő-példányok épül fel. Például lehetséges, hogy számos különböző, logikailag elkülönített számítási példányok, amelyek a különböző alkalmazások. Végső soron hozhat létre a VDC használhatja ezeket a különböző összetevők típusok és példányok.

[![4]][4]

Az előző architektúrájának áttekintése a VDC-megvalósítási különböző zónákhoz tartozhatnak a Központ-küllő topológia használható különböző összetevő típusok jeleníti meg. Az infrastruktúra-összetevőket látható az architektúra különféle részein.

Egy helyszíni adatközpontban vagy a VDC végrehajtásához jó megoldás hozzáférési jogokat és jogosultságokat kell alapuló csoportot. Kezelnek pedig egyéni felhasználóknak következetesen karbantartása hozzáférési házirendek a különböző csapatokkal, és konfigurációs hibák minimalizálása érdekében. Rendelje hozzá, és távolítsa el a felhasználókat és a egy adott felhasználó jogosultságával naprakészen tartani a megfelelő csoportokban.

Minden egyes szerepkör-csoport egy egyedi előtagot kell rendelkeznie a nevüket. Ezt az előtagot egyszerűen azonosíthatja, hogy melyik csoporthoz társítva mely számítási feladatok. Például egy hitelesítési szolgáltatást üzemeltető számítási feladatok lehet nevű csoport **AuthServiceNetOps**, **AuthServiceSecOps**, **AuthServiceDevOps**, és **AuthServiceInfraOps**. Szerepkörök, vagy nem adott szolgáltatásokhoz kapcsolódó szerepkörök központosított, előfordulhat, hogy a tartalmazni **Corp**. Például **CorpNetOps**.

Számos szervezet egy változata, a következő csoportok használatával adja meg a főbb információkat a szerepkörök:

-   A központi informatikai csoport **Corp,** szabályozhatja az infrastruktúra-összetevőket tulajdonosi jogosultságokkal rendelkezik. Példák a hálózati és biztonsági. A csoport közreműködői az előfizetésen szerepét, a hub és a hálózati közreműködő jogok ellenőrzése a küllők kell érnie. Nagyméretű szervezetek gyakran osztott e felügyeleti feladatokat több csapatok között. Példák a hálózati műveletek **CorpNetOps** kizárólagos összpontosít a hálózatkezelés és a egy biztonsági műveletek a csoport **CorpSecOps** csoport felelős a tűzfal- és biztonsági szabályzatot. Ebben az esetben két különböző csoportot kell létrehozni ezeket egyéni szerepkör-hozzárendelés.
-   A fejlesztési-tesztelési csoport **AppDevOps,** felelősség alkalmazás vagy szolgáltatás számítási feladatok üzembe helyezéséhez. Ez a csoport vesz igénybe, szolgáltatott infrastruktúra telepített példányainak a virtuális gép közreműködő szerepkörének vagy egy vagy több PaaS közreműködő szerepkört. Lásd: [beépített szerepkörök az Azure-erőforrások][Roles]. A fejlesztési-tesztelési csapata is látható-e a biztonsági házirendek, az NSG-k és útválasztási házirend, udr-EK, a hub vagy az egy adott küllő belül lehet szükség. Mellett a számítási feladatokhoz közreműködői szerepkört ennek a csoportnak módosítania kell a hálózati olvasó szerepkört.
-   A művelet és a karbantartási csoport, **CorpInfraOps** vagy **AppInfraOps,** az éles számítási feladatok felügyeletének terhei rendelkezik. Ennek a csoportnak kell lennie a számítási feladatokat éles üzemű előfizetéseket az előfizetés közreműködői. Egyes szervezetek is előfordulhat, hogy kiértékelése, ha egy további eszkalációs támogatási csapat szerepkörrel rendelkező, előfizetés közreműködője az éles és a központi agyhoz előfizetés szükséges. A további csoport potenciális konfigurációs problémákat javítja az éles környezetben.

A VDC, amely csoportosítja a központi informatikai csoport, amely a hub kezelése megfelelő csoporttal rendelkezik, a munkaterhelés számára szintű létrehozott úgy van felépítve. Hub-erőforrások kezelése, mellett csak a központi informatikai csoportok külső hozzáférés és az előfizetés legfelső szintű engedélyeket szeretne szabályozza. Azonban lenne szabályozhatja munkaterhelési csoportokban, erőforrások és engedélyeit a saját virtuális hálózatához, függetlenül a a központi informatikai.

Biztonságos futtatására között különböző üzletágak több projektet a VDC-partíció. Az összes projekt különböző elkülönített környezeteket, például fejlesztési, foglalja annak és üzemi van szükség. Ezekben a környezetekben mindegyike külön Azure-előfizetések elkülönítéshez természetes.

[![5]][5]

A fentebbi ábra bemutatja azokat a szervezet projekteket, a felhasználók és csoportok és, az Azure-összetevők telepítve vannak a környezetek közötti kapcsolat.

Jellemzően IT,-környezet vagy a csomag egy rendszer, amelyben több alkalmazás üzembe helyezését és futtatását. A nagyobb cégeknek is győződjön meg arról, és tesztelheti a módosításokat és a egy éles környezetben, amely a végfelhasználók a fejlesztési környezet használatával. Szkriptjét el vannak különítve, gyakran több átmeneti környezetek között a többszakaszos üzembe helyezés (Bevezetés) engedélyezését, tesztelés és a visszaállítás Ha probléma adódik. Üzembe helyezési architektúra általában továbbra is kövesse az alapvető folyamat, amely kezdődik, fejlesztési, azonban kialakulhatnak **fejlesztési**, és éles környezetben vége **PROD**.

Az Azure DevOps fejlesztési és tesztelési, átmeneti és éles környezetek számára foglalja annak az ilyen típusú több rétegből álló környezeteket egy gyakori architektúráját áll. Szervezetek használhatnak egyetlen vagy több Azure AD-bérlőt, hozzáférés és ezekben a környezetekben jogosultságok meghatározására. Az előző ábrán látható egy esetet, ha két különböző Azure AD-bérlőt használ: egyet az Azure DevOps és foglalja annak és egy másik kizárólag az éles környezetben.

Jelen másik Azure AD-bérlők kikényszeríti a környezetek elkülönítése. Az azonos csoportján felhasználói számára, például a központi informatikai, szüksége van egy másik Azure eléréséhez másik URI használatával történő hitelesítéshez AD-bérlő módosítása a szerepkörök vagy engedélyek vagy az Azure DevOps- vagy éles környezetben a projekt. A különböző felhasználói hitelesítések eléréséhez a különböző környezetek jelenléte csökkenti a szolgáltatáskimaradásokat és egyéb az emberi hibák által okozott problémák.

#### <a name="component-type-infrastructure"></a>Összetevő típusa: infrastruktúra
Ezen összetevő típusa, ahol a legtöbb támogató infrastruktúra található. Is, a központi informatikai, biztonság, megfelelőség részlegeknek legtöbb idejüket és.

[![6]][6]

Infrastruktúra-összetevőket adjon meg egy a VDC összetevői közötti kapcsolatot biztosítja. Még a mind a küllős topológia az szerepel. A felelősség kezelésével és karbantartásával az infrastruktúra-összetevőket általában hozzá van rendelve a központi informatikai vagy biztonsági csoport.

Az informatikai infrastruktúra csapat az elsődleges feladatok egyikét, az IP-cím sémák konzisztenciájának biztosítása a vállalaton belül. A privát IP-címteret a VDC kell lennie rendelve. Nem lehet átfedésben a helyszíni hálózatokon rendelt magánhálózati IP-címek.

A helyszíni peremhálózati útválasztói vagy az Azure-környezetek NAT IP-címütközés elkerüléséhez. De komplikációk hozzáadja az infrastruktúra-összetevőket. Egyszerű a felügyeleti az egyik fő célja a VDC-a. Egy általunk javasolt megoldás használatával NAT IP-problémák kezeléséhez nem.

Infrastruktúra-összetevőket kell a következő funkciókat:

-   [**Identitás- és címtárszolgáltatásokat**][AAD]. Az Azure-ban minden erőforrástípus a hozzáférést az identitás a címtárszolgáltatások tárolt vezérli. A címtárszolgáltatás tárolja a felhasználók listában, és a hozzáférési jogosultságokat az erőforrásokhoz egy adott Azure-előfizetést. Ezek a szolgáltatások csak a felhőben is létezik. Vagy egy helyszíni Azure Active Directoryban tárolt identitással is szinkronizálható.
-   [**Virtuális hálózat**][VPN]. Virtuális hálózatok a VDC fő összetevői közül. Virtuális hálózatok használatával létrehozhat egy forgalomelkülönítési határok az Azure platformon. Virtuális hálózat egy vagy több virtuális hálózati szegmensek tevődik össze. Mindegyik rendelkezik egy adott IP hálózati előtagot, egy alhálózatot. A virtuális hálózat egy belső peremhálózati területre, ahol IaaS virtuális gépek és PaaS-szolgáltatások létesíthet a privát kommunikációs határozza meg. Egy virtuális hálózaton belüli virtuális gépek és PaaS szolgáltatások nem útján kommunikálnak közvetlenül egy másik virtuális hálózatban a virtuális gépek és PaaS-szolgáltatásokat. Ez az elkülönítés akkor fordul elő, akkor is, ha az adott ügyfél hoz létre két virtuális hálózatnak egy előfizetésen belül. Elkülönítés a kritikus fontosságú tulajdona, amely biztosítja, hogy az ügyfél virtuális gépei és kommunikációs privát virtuális hálózaton belül marad.
-   [**UDR**][UDR]. A virtuális hálózati adatforgalmat a rendszer útválasztási táblázat alapján alapértelmezés szerint. Egy felhasználó által definiált útvonal egy egyéni útválasztási táblázat, amely egy vagy több alhálózatot a hálózati rendszergazdák társítható. Ez a társítás felülírja a rendszer útválasztási táblázat viselkedését, és határozza meg a virtuális hálózaton belüli kommunikációs útvonal. Udr-EK meglétének garantálja, hogy az adott egyéni virtuális gépeket vagy a hálózati virtuális berendezések keresztül a küllő származó kimenő forgalmon továbbítására, és a terheléselosztók jelentenek a hubot és a küllők.
-   [**NSG-T**][NSG]. A hálózati biztonsági csoport biztonsági szabályokból álló listát, amely a szűrést az IP-források, IP destinations, protokollok, IP forrásportok és IP-célportok forgalom. Az NSG-t egy alhálózathoz, az Azure virtuális Gépekhez vagy mindkettőhöz társított virtuális hálózati kártya alkalmazhat. Az NSG-k nélkülözhetetlenek valósítható meg egy megfelelő adatfolyam vezérlés, a hubot és a küllők az. Az NSG-t által nyújtott biztonság szintje a függvény, mely portok megnyitása és milyen célra. További VM-szűrők például engedélyezze az IPtables állomásalapú tűzfal vagy a Windows tűzfal / célszerű telepíteni.
-   [**DNS**][DNS]. A virtuális hálózatok a VDC-erőforrások a névfeloldás a DNS-ben keresztül biztosított. Az Azure DNS szolgáltatást is nyújt [nyilvános] [ DNS] és [privát] [ PrivateDNS] névfeloldását. Privát zónák névfeloldásához egy virtuális hálózaton belül, és több virtuális hálózaton. Privát zónái, amelyek ugyanabban a régióban és régiók és -előfizetések közötti virtuális hálózatok között is. A nyilvános feloldásához az Azure DNS DNS-tartományok üzemeltetési szolgáltatást biztosít. Névfeloldás biztosít a Microsoft Azure-infrastruktúra használatával. Ha tartományait az Azure-ban üzemelteti, DNS-rekordjait a többi Azure-szolgáltatáshoz is használt hitelesítő adatokkal, API-kkal, eszközökkel és számlázási információkkal kezelheti.
-   [**Előfizetés** ] [ SubMgmt] és [ **eszközcsoport-kezelés erőforráshoz**][RGMgmt]. Előfizetés az erőforrások több csoport létrehozása az Azure-ban a természetes határok határozza meg. Egy előfizetés erőforrásait együtt vannak-e összeállítva nevű logikai tárolók **erőforráscsoportok**. Az erőforráscsoport olyan logikai csoportot a VDC-erőforrások rendszerezéséhez jelöl.
-   [**RBAC**][RBAC]. Keresztül RBAC lehetőség hozzáférjenek bizonyos Azure-erőforrásokra, valamint szervezeti szerepköröket hozzárendelni, amely lehetővé teszi, hogy a felhasználók csak bizonyos részét műveletek korlátozása. Az RBAC lehet hozzáférést biztosítani a megfelelő szerepkört rendelhet a felhasználók, csoportok és alkalmazások megfelelő hatókörébe. Szerepkör-hozzárendelés hatóköre az Azure-előfizetés, erőforráscsoport vagy egyetlen erőforrás lehet. Az RBAC lehetővé teszi, hogy az engedélyek öröklődése. Egy szülő hatókörben hozzárendelt szerepkör is hozzáférést biztosít a benne lévő gyermekei. Az RBAC feladatköröket, és csak olyan mértékű hozzáférést biztosítson a felhasználók számára, amelyek a feladataik elvégzéséhez szükségük van. RBAC például használatát, így egy alkalmazott kezelheti a virtuális gépek az előfizetéshez. Kezelheti egy másik SQL-adatbázisok ugyanazon az előfizetésen belül.
-   [**Virtuális hálózatok közötti társviszony**][VNetPeering]. A a VDC infrastruktúra létrehozásához használt alapvető funkció a virtuális hálózatok közötti társviszony. Egy olyan mechanizmus, amely összeköti a két virtuális hálózat ugyanabban a régióban, az Azure-beli adatközpont-hálózaton keresztül, vagy az Azure világszerte gerinchálózatát használva régióban.

#### <a name="component-type-perimeter-networks"></a>Összetevő típusa: peremhálózatokon
A [szegélyhálózaton] [ DMZ] összetevők és a egy DMZ hálózati hálózati kapcsolatot biztosíthat a helyszíni vagy a fizikai adatközpont-hálózatok, valamint minden olyan kapcsolat, az internet felé és felől . Emellett akkor is, ahol a hálózati és biztonsági valószínűleg részlegeknek legtöbb idejüket.

Bejövő csomagok a háttér-kiszolgálók a küllők elérése előtt kell áramlása az agyban biztonsági készülékek. Példák a tűzfal, Azonosítók és IP-CÍMEK. Mielőtt a hálózaton, a számítási feladatok az internetre irányuló csomagokat is áramlásának keresztül a biztonsági berendezésekről a peremhálózaton. Ez a folyamat célja a házirend betartatása, ellenőrzés és naplózás.

Szegélyhálózat hálózati összetevők adja meg a következő funkciókat:

-   [Virtuális hálózatok][VNet], [udr-EK][UDR], és [NSG-k][NSG]
-   [Hálózati virtuális berendezések][NVA]
-   [Az Azure Load Balancer][ALB]
-   [Az Azure Application Gateway] [ AppGW] és [webalkalmazási tűzfal (WAF)][WAF]
-   [Nyilvános IP-címek][PIP]
-   [Az Azure bejárati ajtajának][AFD]
-   [Az Azure-tűzfal][AzFW]

Általában a központi informatikai és biztonsági csapatok követelmény definíció-és a szegélyhálózat-alapú hálózatok művelet feladata.

[![7]][7]

A fenti ábrán két régebben a hozzáféréssel rendelkező érvénybe léptetése az internethez és a egy helyszíni hálózat látható. A DMZ-t és vWAN hubs rezidens mindkettő. Az agyban DMZ-t az internethez a peremhálózaton is vertikális felskálázás több farmok webalkalmazási tűzfal (WAF) vagy az Azure-tűzfal példányok használatával nagy számú üzletágak támogatásához. VWAN központban rugalmasan méretezhető ág-ág és ág – Azure kapcsolat történik egy VPN- vagy ExpressRoute-n keresztül igény szerint.

[**Virtuális hálózatok**][VNet]. A hub általában épülő üzemeltetéséhez a különböző szolgáltatások, szűrése és vizsgálja meg a bejövő és kimenő forgalmat az interneten keresztül nva-k, waf-eszközzel, több alhálózattal rendelkező virtuális hálózat és az Azure Application Gateway-példány.

[**Udr-EK**][UDR]. Az udr-EK ügyfelei tűzfalak, IDSs vagy IPSs és más virtuális berendezések üzembe. Azok az útvonalválasztást hálózati forgalom ezen biztonsági berendezésekről biztonsági határ házirendek betartását, naplózás és vizsgálat. Mind a küllős topológia az udr-EK hozhat létre. Garantálják, hogy a forgalom továbbítását keresztül az adott egyéni virtuális gépek hálózati virtuális berendezések, és a VDC által használt terheléselosztók. Győződjön meg arról, hogy a megfelelő virtuális berendezésre megtalálhatók a küllő továbbítását a virtuális gépek által generált forgalom, állítsa a egy UDR a küllős, alhálózatok. Állítsa be a belső terheléselosztó előtérbeli IP-címét a következő ugrás. A belső terheléselosztó osztja el a belső bejövő forgalmának a virtuális készülékek, illetve a load balancer háttérkészlethez.

[**Az Azure tűzfal** ] [ AzFW] egy felügyelt, felhőalapú és hálózati biztonsági szolgáltatás, amely védelmet nyújt az Azure Virtual Network-erőforrások. Teljes állapot-nyilvántartó tűzfal beépített magas rendelkezésre állás és méretezhetőség korlátlan felhőalapú szolgáltatásként is. Központilag hozhatja létre, érvényesítheti és naplózhatja az alkalmazás- és hálózatelérési szabályzatokat az előfizetésekre és a virtuális hálózatokra vonatkozóan. Azure-tűzfalon statikus nyilvános IP-címet használ a virtuális hálózati erőforrások. Lehetővé teszi a tűzfalon kívül a virtuális hálózatból érkező forgalom azonosítására. A szolgáltatás teljesen integrálva van az Azure Monitorral a naplózás és az elemzés érdekében.

[**Hálózati virtuális berendezések**][NVA]. Az agyban a peremhálózaton, az internet-hozzáféréssel rendelkező általában egy Azure tűzfal-példányt, vagy a tűzfalak vagy a webalkalmazási tűzfal (WAF) farm keresztül kezeli.

Különböző objekty lobs s Hodnotou használata általában számos webes alkalmazásokhoz. Ezek az alkalmazások általában különböző biztonsági rések és az esetleges biztonsági rések. sorból. Webalkalmazás-tűzfalak egy speciális típusú részletesebben olvashat róluk, mint az általános tűzfal webes alkalmazások, HTTP/HTTPS-elleni támadások észleléséhez használt terméket. Képest hagyomány tűzfal-technológiája, alsóbb rendelkeznek kívánt belső webkiszolgálók védelme a fenyegetésekkel szemben.

Azonban a VDC következő egyetlen megvalósítása szükség egy adott régión belül lehet üzemeltetni, mert az előfizetés követelményeinek régiók Átfedés tiltása. Ez a követelmény a VDC következő egyetlen megvalósítása érinti a regionális kimaradások lehetővé teszi.

Egy Azure tűzfal-példányt vagy NVA tűzfal farmok használja egy közös felügyeleti sík. Biztonsági szabályok munkaterheléseinek a küllő és a vezérlőket a hozzáférést a helyszíni hálózatok üzemelteti. Azure tűzfal rendelkezik beépített, méretezhetőséget, mivel NVA tűzfalak manuálisan méretezhetők egy terheléselosztó mögé. Általában egy tűzfal farm rendelkezik speciális szoftver kevésbé képest WAF. De szélesebb körű alkalmazás-hatókör szűrését, és vizsgálja meg a bejövő és kimenő forgalmat bármilyen típusú rendelkezik. Ha egy NVA-módszert használja, azok található és az Azure Marketplace-ről üzembe helyezett.

Azt javasoljuk, hogy az internetről érkező forgalomhoz Azure tűzfal-példányra, vagy az nva-k, egy készletét használja. A helyszíni forgalom egy másik használja. Biztonsági kockázat csak egy tűzfalak használatával is, nincs biztonsági határ a két fajta hálózati forgalom között lehetővé teszi. Különálló tűzfalat rétegek használatával csökkenti az ellenőrző biztonsági szabályok összetettségét, és egyértelművé teszi, hogy melyik szabály vonatkozik, hogy mely bejövő hálózati kérésekre.

Legtöbb nagyvállalat kezelheti a több tartomány. [Az Azure DNS] [ DNS] segítségével egy adott tartomány DNS-rekordjainak üzemeltetésére. Például az Azure külső load balancert, vagy a waf-eszközzel, a virtuális IP-cím (VIP) cím regisztrálni lehet a **A** rekord az Azure DNS-rekord. [**Privát DNS** ] [ PrivateDNS] is elérhetők a magánhálózati címtartománynak belüli virtuális hálózatok kezeléséhez.

[**Az Azure Load Balancer** ] [ ALB] magas rendelkezésre állású 4. rétegű, TCP vagy UDP, a szolgáltatás kínál. Terjesztheti, hogy a bejövő forgalmat egy elosztott terhelésű készlet definiált szolgáltatási példányai között. A terheléselosztó a előtér-végpontokra küldött forgalom nyilvános vagy magánhálózati IP-Címek használatához újra vagy anélkül címfordítás számára egy háttér-IP-címkészleteket. Például a hálózati virtuális berendezések vagy virtuális gépeken.

Az Azure Load Balancer megvizsgálja is, valamint a különböző kiszolgálópéldányok állapotát. Ha egy mintavételező nem válaszol, akkor a terheléselosztó nem irányít forgalmat küld a nem megfelelő állapotú példány. Az a VDC az agyban külső terheléselosztó elosztja a forgalmat az nva-k, például. A küllők a feladatokat, például egy többrétegű alkalmazás különböző virtuális gépek közötti forgalom terheléselosztási hajtja végre.

[**Az Azure bejárati ajtajának** ] [ AFD] van a Microsoft magas rendelkezésre állású és méretezhető webes gyorsítás alkalmazásplatform, globális HTTP-terheléselosztó, alkalmazásvédelem és a tartalomkézbesítési hálózat. Bejárati ajtó a peremhálózaton, a Microsoft globális hálózatának 100-nál több helyen futtatja. Bejárati ajtajának használatával hozhat létre, üzemeltetése és horizontális felskálázása a dinamikus webalkalmazások és a statikus tartalmat. Bejárati ajtajának az alkalmazását az alábbi előnyöket nyújtja: 
* Világszínvonalú végfelhasználói teljesítmény.
* Egyesített regionális vagy karbantartási automation blokk.
* BCDR automation.
* Egyesített ügyfél vagy a felhasználói adatokat. 
* Gyorsítótárazás. 
* Service insights. A platformot kínál a teljesítményt, megbízhatóságot és támogatási SLA-k, megfelelőségnek és naplózható biztonsági eljárásokat, amelyek fejlett, üzemeltetett és Azure által támogatott natív módon.

[**Az Application Gateway** ] [ AppGW] alkalmazáskézbesítési vezérlőt (ADC) biztosító dedikált virtuális berendezés van szolgáltatásként, a különböző réteg 7 terheléselosztási lehetőséget nyújt alkalmazásának. Az Application Gateway-példány processzorigényes SSL-lezárások felé történő kiszervezésével optimalizálhatják a webfarmok termelékenységét. Egyéb 7. rétegbeli útválasztási lehetőségeket, amelyek tartalmazzák a következő példákban is biztosít: 
* Bejövő forgalom Ciklikus időszeleteléses elosztását. 
* Cookie-alapú munkamenet-affinitás. 
* URL-cím-alapú útválasztás. 
* Lehetővé teszi az Egypéldányos Application Gateway mögött több webhelyet is üzemeltethet. Webalkalmazási tűzfal (WAF) Application Gateway WAF Termékváltozatában részeként is tartalmaz. Ez a Termékváltozat webalkalmazásokat a gyakori internetes biztonsági rések és az azokat kihasználó támadások ellen védelmet biztosít. Az Application Gateway konfigurálható egy internetre irányuló átjáró, egy belső átjáró vagy mindkettőt. 

[**Nyilvános IP-címek**][PIP]. Az egyes Azure-funkciókról társíthatja a Szolgáltatásvégpontok egy nyilvános IP-címre, hogy az erőforrás elérhető lesz az interneten. Ez a végpont hálózati címfordítás (NAT) használatával irányíthatja a forgalmat a belső cím és port, az Azure-beli virtuális hálózaton. Ez az elérési út a külső forgalmat a virtuális hálózatban módját. Meghatározhatja az adatforgalom átadott és hol és hogyan lehet a virtuális hálózatra lefordított nyilvános IP-címeket is beállíthatja.

[**Az Azure DDoS Protection Standard** ] [ DDOS] képest további veszélyelhárítási szolgáltatásokat nyújt a [alapszintű szolgáltatási] [ DDOS] hangolt szint kifejezetten az Azure virtuális hálózati erőforrásokat. A DDoS Protection Standard engedélyezése egyszerű, és nem kell application módosítani. Az alkalmazásvédelmi szabályzatok hangolt dedikált forgalomfigyelést és gépi tanulási algoritmus segítségével. A virtuális hálózatokon üzembe helyezett erőforrásokhoz rendelt nyilvános IP-címek szabályzatok érvényesek. Példák Azure Service Fabric, Azure Load Balancer és Azure Application Gateway-példány. Valós idejű telemetriai adatokat az Azure Monitor nézetek keresztül érhető el, a támadás során, és az előzmények. Alkalmazásréteg-védelem az Azure Application Gateway webalkalmazásokhoz használható tűzfal segítségével is hozzáadhat. Védelmet biztosítanak a nyilvános IP-címek IPv4 Azure.

#### <a name="component-type-monitoring"></a>Összetevő típusa: figyelése
Figyelési összetevők látható-e, és a más összetevők adattípusok riasztási adja meg. Csapatok mindegyikével kell férniük a figyelést az összetevők és szolgáltatások hozzáféréssel rendelkeznek. Van egy központi súgó ügyfélszolgálati vagy művelet csapat, szükség esetén hozzáférést az adatokhoz, ezek az összetevők által biztosított integrált.

Az Azure a naplózást és a szolgáltatások az Azure-ban tárolt erőforrások viselkedését követni a különböző típusú kínál. Cégirányítási és vezérlés az Azure-beli számítási feladatok alapján nem csupán a teljesítményadatok gyűjtését. naplófájl, de is specifikus eseményindító műveletek lehetővé teszi az események jelentett.

[**Az Azure Monitor**][Monitor]. Az Azure több szolgáltatást is tartalmaz, amelyek önmagukban is betöltenek adott szerepköröket vagy végrehajtanak bizonyos feladatokat a monitorozás területén. Ezek a szolgáltatások együtt egy átfogó megoldást nyújtanak az alkalmazások és az azt támogató Azure-erőforrások telemetriaadatainak gyűjtésére és elemzésére, valamint ezek alapján műveletek végrehajtására. Emellett használhatók kritikus fontosságú helyszíni erőforrások monitorozására egy hibrid monitorozási környezet biztosítása érdekében. A rendelkezésre álló eszközök és adatok megismerése az első lépés az alkalmazás teljes monitorozási stratégiájának kidolgozásában.

Naplók az Azure-ban két fő típusa van:

-   A [Azure-tevékenységnapló][ActLog]korábban néven **műveleti naplók**, az az Azure-előfizetéshez tartozó erőforrásokon végrehajtott műveletekkel betekintést nyújt. Ezek a naplók a vezérlősík események az előfizetésekre vonatkozó jelentést. Minden Azure-erőforrás naplókat eredményez.

-   [Az Azure Monitor-diagnosztikai naplók] [ DiagLog] rendszer részletes, gyakori adatokat az erőforrás a művelet az erőforrás által létrehozott naplók. Ezek a naplók a tartalom erőforrás típusa szerint változó.

[![9]][9]

A VDC fontos nyomon követheti az NSG-naplók, különösen ezeket az információkat:

-   [Eseménynaplók] [ NSGLog] információkat tartalmaz a virtuális gépeket és példányszerepköröket MAC-cím alapján milyen NSG-szabályok érvényesek.
-   [Naplók számláló] [ NSGLog] nyomon követése az egyes NSG-szabályok a forgalom engedélyezése vagy megtagadása volt futtatva futtatásainak számát.

Az összes napló tárolható a naplózási, elemzési statikus vagy biztonsági mentési célból az Azure storage-fiókot. Azure-tárfiók tárolja a naplókat, ha ügyfelek különböző típusú keretrendszerek használatával lekérni, -előkészítési, elemzése, és jelenítse meg az adatokat jelenti az állapotot és a felhőbeli erőforrások állapotát. 

A nagyobb cégeknek is kell már szerezték be olyan szabványos keretrendszer, a helyszíni rendszerek figyelése. Integrálható a felhőben üzemelő példányok által létrehozott naplók a keretrendszer kiterjesztheti azokat. Az [Azure Log Analytics] [https://docs.microsoft.com/en-us/azure/log-analytics/log-analytics-queries], szervezetek megtarthatja az összes naplózási a felhőben. A log Analytics felhőalapú szolgáltatásként van megvalósítva. Ezért kell, és gyorsan és az infrastrukturális szolgáltatásokra fordítandó minimális kiadások mellett. A log Analytics is integrálhatja a System Center-összetevőket, például a System Center Operations Manager bővítése a meglévő felügyeleti beruházások kiterjeszthetők a felhőre. 

A log Analytics szolgáltatása az Azure-ban, amely segít összegyűjtését, összekapcsolását, keressen, és operációs rendszerek, alkalmazások és infrastruktúra felhőalapú összetevők által generált napló- és teljesítményadatokat az adatokkal műveleteket végezni. Biztosít ügyfelei valós idejű az operational insights által integrált keresés és az egyéni irányítópultok segítségével elemezze a feladatait a VDC az összes rekordot.

[Az Azure Network Watcher] [ NetWatch] eszközeivel monitorozása, diagnosztizálása, és metrikákat tekinthet meg és engedélyezhető vagy tiltható le a naplókat a további erőforrások az Azure-beli virtuális hálózathoz. Sokoldalú szolgáltatása lehetővé teszi a következő funkciók és további:
-    A virtuális gép és a végpontok közötti kommunikáció monitorozása.
-    Virtuális hálózat és azok az erőforrások megtekintése.
-    Diagnosztizálja a hálózati forgalom szűrésével kapcsolatos problémákat, vagy egy virtuális gépről.
-    Hálózati a virtuális gép útválasztási problémáinak diagnosztizálása
-    Diagnosztizálja a virtuális gép kimenő kapcsolatokat.
-    Csomagok és a egy virtuális gép rögzítése.
-    Egy Azure virtuális hálózati átjáró és kapcsolatok kapcsolatos problémák diagnosztizálásához.
-    Határozza meg, internetszolgáltatók és az Azure-régiók közötti relatív késéseket.
-    Biztonsági szabályok megtekintése egy hálózati adapter.
-    Hálózati metrikák megtekintése.
-    Elemezheti a bejövő és kimenő forgalmat egy hálózati biztonsági csoportot.
-    Diagnosztikai naplók megtekintése a hálózati erőforrásokat.

A [Network Performance Monitor] [ NPM] belül az Operations Management Suite megoldás részletes hálózati információkat teljes körű biztosítja. Ezen információk közé tartozik az Azure-hálózatok és a helyszíni hálózatok egyetlen nézetben. A megoldás adott figyelők rendelkezik ExpressRoute- és nyilvános szolgáltatások.

#### <a name="component-type-workloads"></a>Összetevő típusa: számítási feladatok
Számítási feladatok összetevői a tényleges alkalmazások és szolgáltatások-ket. Ezek Ön is, ahol az alkalmazás fejlesztői részlegeknek legtöbb idejüket.

A számítási lehetőségek száma végtelen. A lehetséges munkaterhelés-típusok közül a következők:

**Belső üzleti alkalmazások** számítógépekhez készült alkalmazások létfontosságúak a vállalat a folyamatban lévő műveletet. Üzletági alkalmazások néhány közös jellemzőkkel rendelkeznek:

-   **Interaktív** természetéből. Adatok bevitele, és a visszaadott eredmények vagy jelentések.
-   **Adatvezérelt**&mdash;intenzív az adatok gyakran használják az access-adatbázisok vagy más tárolási.
-   **Integrált**&mdash;belül vagy a szervezeten kívül más rendszerekkel való integrációt kínálnak.

**Ügyfelek által használt websites (interneten vagy belső hálózatra irányuló)**. A legtöbb olyan alkalmazások, az internettel kommunikáló webhelyek. Az Azure lehetőséget biztosít az IaaS virtuális gépen, vagy a webhely futtatásához egy [Microsoft Azure App Service Web Apps funkciójával] [ WebApps] hely (PaaS). A Web Apps támogatja az integrációt a virtuális hálózatokkal, amelyek lehetővé teszik a webalkalmazások a küllőkben a VDC-a központi telepítés. Ha belső webhelyek, virtuális hálózati integráció, nem kell elérhetővé olyan internetes végpontot, az alkalmazások számára. De használhatja az internet irányítható Magáncímeket erőforrásai helyett a virtuális magánhálózaton.

**Big Data jellegű adatok és analitika**. Ha adatokat kell a vertikális felskálázás akár nagy, adatbázisok előfordulhat, hogy nem vertikális felskálázás megfelelően. Apache Hadoop technológiát kínál a rendszer a nagy számú csomópont elosztott lekérdezések párhuzamos futtatásához. Ügyfelek data számítási feladatok futtathatók IaaS virtuális gépek és PaaS, [Azure HDInsight][HDI]. HDInsight a hely-alapú virtuális hálózatban üzembe helyezést támogatják. Egy adott küllőre a VDC-fürtön is telepíthető.

**Események és üzenetküldési**. [Az Azure Event Hubs] [ EventHubs] nagy kapacitású telemetriaadat szolgáltatás, amely összegyűjti a összegyűjteni, átalakítani és több millió eseményt tárolja. Egy elosztott streamelési platform biztosít alacsony késéssel és konfigurálható idejű adatmegőrzést. Ezért nagy mennyiségű telemetria betöltését az Azure-ba, és több alkalmazás adatokat olvasni. Az Event hubs szolgáltatással egyetlen streammel támogathat valós idejű és kötegelt folyamatok.

Megvalósíthat egy nagy megbízhatóságú felhőalapú üzenetküldő szolgáltatásra alkalmazások és szolgáltatások közötti [Azure Service Bus][ServiceBus]. Kínál aszinkron közvetítőalapú üzenettovábbítás ügyfél és kiszolgáló közötti, első-first out (FIFO) üzenetkezelés, strukturált és közzétételi és feliratkozási képességek.

[![10]][10]

### <a name="multiple-vdc-implementations"></a>Több VDC-implementációk


Egyetlen VDC azonban egy adott régión belül helyezkedik el. Egy fő szolgáltatáskimaradás, amely a teljes régióra hatással lehet kitéve. Ügyfelek, amelyek magas SLA-k eléréséhez kell ugyanazon a projekten, két vagy több VDC-implementációkban helyezni különböző régiókban lévő telepítéseit a szolgáltatásaikat.

SLA vonatkozik kívül számos gyakori forgatókönyvek, ahol üzembe helyezése a VDC-megvalósításokhoz több értelme:

-   Regionális és globális jelenlétét.
-   Vészhelyreállítás.
-   Egy olyan mechanizmust átirányít az adatközpontok közötti forgalmat.

#### <a name="regional-and-global-presence"></a>Regionális és globális jelenlét
Azure-adatközpontokban világszerte számos régióban találhatók. Amikor kiválasztják a több Azure-adatközpontok, ügyfeleknek kell figyelembe venni két tényezőt összefüggő: földrajzi távokat és a késés. A legjobb felhasználói élményt kínál, a felhasználóknak meg kell kiértékelni a a VDC-megoldások és a VDC-megoldások és a végfelhasználók közötti távolság közötti földrajzi távolságtól.

A VDC-megvalósítások üzemeltető Azure-régióban kell is megfeleljen az alapján, amely a szervezet működik minden jogi joghatósága által létrehozott szabályozási követelményeknek.

#### <a name="disaster-recovery"></a>Vészhelyreállítás
A vész-helyreállítási terv megvalósítása az érintett számítási feladat típusától és a számítási feladat állapota közötti implementálható VDC szinkronizálhatók kapcsolódik. Ideális esetben a legtöbb ügyfél szeretné szinkronizálni az alkalmazásadatok közötti üzemelő példánya, amely két különböző VDC megvalósításokban egy gyors feladatátvételi olyan mechanizmus megvalósításához futtassa. A legtöbb alkalmazás késés-és nagybetűket, és ami hibákat okozhat lehetséges időkorlátja és a késleltetés az adatok szinkronizálását.

Szinkronizálás vagy a szívverés-figyelés az alkalmazások különböző VDC-implementációkban szükséges a köztük folyó kommunikációt. Különböző régiókban lévő két VDC-implementáció csatlakozhat a következő:

-   Virtuális hálózatok közötti társviszony kapcsolódhatnak hubs régiók között elosztva.
-   Az ExpressRoute privát társviszony-létesítés a VDC-hubs ugyanahhoz az ExpressRoute-kapcsolatcsoporthoz csatlakoztatott.
-   Több ExpressRoute-kapcsolatcsoporttal a vállalati gerinchálózatán keresztül kapcsolódik, és az ExpressRoute-Kapcsolatcsoportok csatlakozik a VDC-háló.
-   Site-to-site VPN-kapcsolatok a VDC hubs, az egyes Azure-régió között.

Virtuális hálózatok közötti társviszony- vagy ExpressRoute-kapcsolatok általában az előnyben részesített mechanizmus nagyobb sávszélességet és egységes késés miatt, ha a Microsoft gerinchálózatán keresztül áthaladó.

Nincs nem magic recept ellenőrzése között két vagy több VDC implementálható különböző régiókban található elosztott alkalmazáshoz. Ügyfelek hálózati minősítési tesztekkel a késés és a kapcsolatok sávszélesség kell futtatnia. Cél majd szinkron vagy aszinkron adatreplikációt-e megfelelő, és mi az optimális helyreállítási időre vonatkozó célkitűzés (RTO) is lehet a számítási feladatokhoz.

#### <a name="mechanism-to-divert-traffic-between-datacenters"></a>Az adatközpontok közötti forgalom átirányít mechanizmus
Egy hatékony módszer átirányít a forgalmat bejövő egy másikra egy adatközpontban, a a tartomány nevét (DNS) alapul. [Az Azure Traffic Manager] [ TM] egy adott VDC a végfelhasználói forgalmat a leginkább megfelelő nyilvános végponthoz való közvetlen a DNS-mechanizmus segítségével. – Mintavételek menüpontban a Traffic Manager rendszeres időközönként különböző VDC-implementációkban nyilvános végpontok a szolgáltatás állapotát ellenőrzi. Ha ezekre a végpontokra, azt automatikusan, a másodlagos VDC irányítja.

A TRAFFIC Manager együttműködik az Azure nyilvános végpontokat. Például azt szabályozhatja, vagy átirányít a forgalom Azure virtuális gépek és a megfelelő VDC webalkalmazásokat. A TRAFFIC Manager képes legyen ellenállni egy teljes Azure-régiót meghibásodása esetén is. Szabályozhatja, hogy a különböző kritériumok alapján implementálható VDC szolgáltatásvégpontokra érkező felhasználói forgalom elosztása. Például egy adott VDC, vagy válassza ki a VDC az ügyfél a legalacsonyabb hálózati késéssel rendelkező szolgáltatás hibája.

### <a name="conclusion"></a>Összegzés
A virtuális adatközpont az Adatközpont áttelepítési megközelítés a felhő által használt funkciók és képességek kombinációjából létrehozása egy méretezhető architektúra az Azure-ban. Ez maximalizálja a felhőalapú erőforrások használatával, csökkenti a költségeket, és egyszerűbbé teszi a rendszer a cégirányítási. A VDC fogalom egy közös megosztott szolgáltatások az agyi biztosító Központ-küllő topológia alapján történik. Ez lehetővé teszi, hogy egyes alkalmazások vagy munkaterhelések a küllők az. A VDC vállalati szerepkörök, ahol a különböző részlegek együttműködve struktúrája megegyezik. Példák a központi informatikai, az Azure DevOps, és a műveletek és karbantartás. Mindegyik részlege rendelkezik egy adott felsorolja azon szerepköröket és feladatokat. A VDC fogalma kielégíti a **lift- and -shift** áttelepítés. De a natív felhőalapú rendszerbe sok előnyt is biztosít.

## <a name="references"></a>Referencia
További információ a következő funkciókat a cikkben leírtak szerint:

| | | |
|-|-|-|
|Hálózati szolgáltatások|Terheléselosztás|Kapcsolatok|
|[Az Azure Virtual Network][VNet]</br>[Hálózati biztonsági csoportok][NSG]</br>[NSG-naplók][NSGLog]</br>[Felhasználó által megadott útvonalak][UDR]</br>[Hálózati virtuális berendezések][NVA]</br>[Nyilvános IP-címek][PIP]</br>[Az Azure DDoS Protection][DDOS]</br>[Az Azure-tűzfal][AzFW]</br>[Az Azure DNS][DNS]|[Az Azure bejárati ajtajának][AFD]</br>[Az Azure Load Balancer (3.)][ALB]</br>[Az Alkalmazásátjáró (7. rétegbeli)][AppGW]</br>[Webalkalmazási tűzfal][WAF]</br>[Az Azure Traffic Manager][TM]</br></br></br></br></br> |[Virtuális hálózatok közötti társviszony][VNetPeering]</br>[Virtuális magánhálózat][VPN]</br>[Virtuális WAN][vWAN]</br>[ExpressRoute][ExR]</br>[Az ExpressRoute közvetlen][ExRD]</br></br></br></br></br>
|Identitás</br>|Figyelés</br>|Ajánlott eljárások</br>|
|[Azure Active Directory][AAD]</br>[A multi-factor Authentication szolgáltatás][MFA]</br>[Szerepköralapú hozzáférés-vezérlés][RBAC]</br>[Alapértelmezett Azure AD-szerepkörök][Roles]</br></br></br> |[A Network Watcher][NetWatch]</br>[Az Azure Monitor][Monitor]</br>[Tevékenységnapló][ActLog]</br>[A figyelő a diagnosztikai naplók][DiagLog]</br>[A Microsoft Operations Management Suite][OMS]</br>[A Network Performance Monitor][NPM]|[Szegélyhálózat-alapú hálózati ajánlott eljárások][DMZ]</br>[Előfizetések kezelése][SubMgmt]</br>[Erőforrás-csoportok kezelése][RGMgmt]</br>[Azure-előfizetés korlátai][Limits] </br></br></br>|
|Más Azure-szolgáltatások|
|[Web Apps alkalmazások][WebApps]</br>[HDInsight (Hadoop)][HDI]</br>[Event Hubs][EventHubs]</br>[Szolgáltatásbusz][ServiceBus]|

## <a name="next-steps"></a>További lépések
 - Ismerkedés a [virtuális hálózatok közötti társviszony][VNetPeering], a VDC küllős formátumukban megerősítő technológia.
 - Alkalmazzon [Azure ad-ben] [ AAD] használatába [RBAC] [ RBAC] feltárása.
 - Előfizetés és a felügyeleti erőforrásmodell és a egy RBAC-modellben a struktúra, a követelményeknek és a szabályzatok a szervezet fejleszthet. A legfontosabb tevékenység tervezi. Sokkal gyakorlati tervezze meg a átszervezések, fúzió, új termék sorok és egyéb lehetőségek.

<!--Image References-->
[0]: ./images/networking-redundant-equipment.png "összetevő átfedés példái" 
[1]: ./images/networking-vdc-high-level.png "küllős VDC magas szintű példája"
[2]: ./images/networking-hub-spokes-cluster.png "fürt hubs és a küllők"
[3]: ./images/networking-spoke-to-spoke.png "Spoke-to-spoke"
[4]: ./images/networking-vdc-block-level-diagram.png "letiltása a VDC szintű ábrája"
[5]: ./images/networking-users-groups-subsciptions.png "felhasználók, csoportok, előfizetések és projektek"
[6]: ./images/networking-infrastructure-high-level.png "magas szintű infrastruktúra diagramja"
[7]: ./images/networking-highlevel-perimeter-networks.png "magas szintű infrastruktúra diagramja"
[8]: ./images/networking-vnet-peering-perimeter-neworks.png "Virtuális hálózatok közötti Társviszony és a szegélyhálózat-alapú hálózatok"
[9]: ./images/networking-high-level-diagram-monitoring.png "figyelés magas szintű diagramja"
[10]: ./images/networking-high-level-workloads.png "magas szintű diagramját számítási feladatokhoz"

<!--Link References-->
[Limits]: /azure/azure-subscription-service-limits
[Roles]: /azure/role-based-access-control/built-in-roles
[VNet]: /azure/virtual-network/virtual-networks-overview
[NSG]: /azure/virtual-network/virtual-networks-nsg
[DNS]: /azure/dns/dns-overview
[PrivateDNS]: /azure/dns/private-dns-overview
[VNetPeering]: /azure/virtual-network/virtual-network-peering-overview 
[UDR]: /azure/virtual-network/virtual-networks-udr-overview 
[RBAC]: /azure/role-based-access-control/overview
[MFA]: /azure/multi-factor-authentication/multi-factor-authentication
[AAD]: /azure/active-directory/active-directory-whatis
[VPN]: /azure/vpn-gateway/vpn-gateway-about-vpngateways 
[ExR]: /azure/expressroute/expressroute-introduction
[ExRD]: https://docs.microsoft.com/en-us/azure/expressroute/expressroute-erdirect-about
[vWAN]: /azure/virtual-wan/virtual-wan-about
[NVA]: /azure/architecture/reference-architectures/dmz/nva-ha
[AzFW]: /azure/firewall/overview
[SubMgmt]: /azure/architecture/cloud-adoption/appendix/azure-scaffold 
[RGMgmt]: /azure/azure-resource-manager/resource-group-overview
[DMZ]: /azure/best-practices-network-security
[ALB]: /azure/load-balancer/load-balancer-overview
[DDOS]: /azure/virtual-network/ddos-protection-overview
[PIP]: /azure/virtual-network/resource-groups-networking#public-ip-address
[AFD]: https://docs.microsoft.com/en-us/azure/frontdoor/front-door-overview
[AppGW]: /azure/application-gateway/application-gateway-introduction
[WAF]: /azure/application-gateway/application-gateway-web-application-firewall-overview
[Monitor]: /azure/monitoring-and-diagnostics/
[ActLog]: /azure/monitoring-and-diagnostics/monitoring-overview-activity-logs 
[DiagLog]: /azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs
[NSGLog]: /azure/virtual-network/virtual-network-nsg-manage-log
[OMS]: /azure/operations-management-suite/operations-management-suite-overview
[NPM]: /azure/log-analytics/log-analytics-network-performance-monitor
[NetWatch]: /azure/network-watcher/network-watcher-monitoring-overview
[WebApps]: /azure/app-service/
[HDI]: /azure/hdinsight/hdinsight-hadoop-introduction
[EventHubs]: /azure/event-hubs/event-hubs-what-is-event-hubs 
[ServiceBus]: /azure/service-bus-messaging/service-bus-messaging-overview
[TM]: /azure/traffic-manager/traffic-manager-overview

