---
title: 'Az Azure virtual datacenter: A hálózati nézőpont'
description: Ismerje meg, hogyan hozhat létre az Azure-beli virtuális adatközpont
author: tracsman
manager: rossort
tags: azure-resource-manager
ms.service: virtual-network
ms.date: 11/28/2018
ms.author: jonor
ms.openlocfilehash: 1d8a9e860ab1a66104dc4133eb5f22ffb4706b84
ms.sourcegitcommit: 5a3fa0bf35376bbe4a6dd668f2d7d44f9cf9c806
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/14/2018
ms.locfileid: "53411684"
---
# <a name="azure-virtual-datacenter-a-network-perspective"></a>Az Azure virtual datacenter: A hálózati nézőpont

## <a name="overview"></a>Áttekintés

Migrálás a helyszíni alkalmazások Azure-bA értékelemeket biztosít a szervezetek a biztonságos és költséghatékony infrastruktúra, akkor is, ha az alkalmazások minimális változtatása mellett települnek át. Azonban ahhoz, hogy a legtöbbet a rugalmasságot lehetséges a felhő-számítástechnika, vállalatok kell fejlesztheti tovább az Azure-képességek kihasználásához architektúrák. 

A Microsoft Azure biztosít, rendkívül nagy kapacitású szolgáltatásokat és az infrastruktúrát, vállalati szintű képességet és megbízhatósággal. Ezek a szolgáltatások és infrastruktúra kínálnak számos választási lehetőségek a hibrid kapcsolat, így az ügyfelek kiválaszthatják a nyilvános interneten keresztül, vagy egy privát Azure ExpressRoute-kapcsolaton keresztül érheti el őket. Microsoft-partnerek fejlett funkciókat is biztosítanak azzal biztonsági szolgáltatások és virtuális berendezések az Azure-beli futtatására van optimalizálva.

Ügyfeleink kiválaszthatják hozzá vagy az interneten keresztül, vagy az Azure expressroute használatával privát hálózati kapcsolatot biztosít a felhőalapú szolgáltatásokhoz. A Microsoft Azure platformon, az ügyfelek zökkenőmentesen infrastruktúrán kiterjesztheti a felhőbe és többrétegű architektúrákat hozhat létre. Microsoft-partnerek fejlett funkciókat is biztosítanak azzal biztonsági szolgáltatások és virtuális berendezések az Azure-beli futtatására van optimalizálva.

## <a name="what-is-the-virtual-datacenter"></a>Mi az a Virtual datacenter?

A kezdetekor a felhőben lett lényegében egy platform nyilvános üzemeltetéséhez. A vállalatok kezdődátuma tisztában vannak a felhővel, és már belső üzleti alkalmazásait a felhőbe áthelyezni. Ilyen típusú alkalmazásokat nincsenek a fokozott biztonság, megbízhatóság, teljesítmény és költség szempontok szükséges további rugalmasságának biztosítása érdekében a módon cloud services kézbesítése. Ez a módszer új infrastruktúrára burkolatúnak és hálózati szolgáltatások kialakítva, hogy ez rugalmasságot biztosít, hanem a új szolgáltatások a méretezés, a vészhelyreállítás és egyéb szempontok.

## <a name="what-is-a-virtual-datacenter"></a>Mi az a virtuális adatközpont?
Felhőalapú megoldások első tervezték, hogy a nyilvános spektrum egyetlen, viszonylag elkülönített alkalmazások üzemeltetéséhez. Ez a módszer jól években dolgozott. Nyilvánvalóvá vált, majd a felhőalapú megoldások előnyeit, és több nagy méretű számítási feladatokat a felhőben is üzemeltethetők. Létfontosságú teljes életciklusa a felhőszolgáltatás-címzés biztonságra, megbízhatóságra, teljesítmény és költség aggályokat egy vagy több régióban üzemelő vált.

Az alábbi ábrán a felhőbeli üzembe helyezés biztonsági eseményáramlási kimaradást egy példát mutat be a **vörös**. A **sárga keretet** számítási feladatok hálózati virtuális berendezések optimalizálás szoba mutatja.

[![0]][0]

A virtuális adatközpontok (VDC) méretezési lehetőségek érhetők el támogatják a vállalati számítási feladatokat, míg a vezetett be, amikor nagy léptékű alkalmazások támogatása a nyilvános felhőben problémák kezelésére szükség terheléselosztási szükségességét született fogalma.

A VDC-implementáció nem képvisel csak az alkalmazás számítási feladatokat a felhőben, de is a hálózati, biztonsági, felügyeleti és infrastruktúra (például a DNS és a Directory Services). Több és több egy vállalati számítási feladatokat áthelyezni az Azure-ba, fontos, gondolja át a támogató infrastruktúra és az objektumok kerülnek, ezeket a feladatokat. Szem előtt tartva gondosan erőforrások struktúrája hogyan elkerüléséhez elterjedését, több száz "munkaterhelés-szigetek", amely független az adatfolyam, modellek és megfelelőségi kihívások külön-külön kell kezelni.

Javaslatok és a egy közös támogató funkciók, szolgáltatások és infrastruktúra külön, de a kapcsolódó entitások gyűjteményét megvalósításához ajánlott eljárások a VDC fogalma. A számítási feladatokat a VDC identitásalapú megtekintésével alacsonyabb költségek miatt méretgazdaságosság, optimalizált biztonság összetevő- és a folyamat forrásadattárakból, könnyebben operations, felügyeleti és megfelelőségi ellenőrzésekhez vegye figyelembe.

> [!NOTE]
> Fontos tisztában lenni azzal, hogy a VDC **nem** egy különálló Azure-termék, de a különböző funkciók és képességek a pontos követelményeinek kombinációja. A VDC módja a szem előtt tartva a számítási feladatok és az Azure-használat, maximalizálhatja az erőforrások és a képességek a felhőben. Egy moduláris megközelítése az informatikai szolgáltatásokat az Azure-ban a vállalati szervezeti szerepkörökről és feladatokról felhőplatformon kialakítását.

A VDC-megvalósítás segítségével a vállalatok munkaterheléseket és alkalmazásokat az Azure-bA lekérése a következő esetekben:

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

A kulcs a VDC előnyeit feloldásához egy központosított küllős hálózati topológiák az Azure-szolgáltatások és funkciók:

* [Az Azure Virtual Network][VNet],
* [Hálózati biztonsági csoportok][NSG],
* [Virtuális hálózatok közötti társviszony][VNetPeering],
* [Felhasználó által megadott útvonalakat (UDR)][UDR], és
* Az Azure-identitás [szerepköralapú hozzáférés-vezérlés (RBAC)] [ RBAC] és opcionálisan [Azure tűzfal][AzFW], [AzureDNS-ben] [ DNS], [Azure bejárati ajtajának][AFD], és [Azure virtuális WAN][vWAN].

## <a name="who-should-implement-a-virtual-datacenter"></a>Akik meg kell valósítania egy Virtual datacenter?

Bármely Azure-ügyfél, amely úgy döntött, a felhő bevezetésére is kihasználhatják az erőforráscsoport közös használatra konfigurálása az összes alkalmazás hatékonyságát. A mértékétől függően akár egyetlen alkalmazások milyen előnyei származhatnak a minták és -összetevők a VDC-megvalósítás összeállításához.

Ha a szervezet rendelkezik egy központi informatikai, hálózati, biztonsági és/vagy megfelelőségi csapat vagy részleg, a VDC végrehajtási segítségével kényszerítése házirend pontok, a vámot, elkülönítése és az alapul szolgáló közös összetevők egységességének biztosítása a alkalmazásfejlesztő csapatok dolgoznak, miközben sokkal szabadsága és a vezérlő, mert a követelményeinek megfelelő.

A szervezetek, fejlesztési és üzemeltetési szeretne is igénybe vehetik a jogosult zsebe Azure-erőforrások és a csoport (vagy az előfizetést, vagy az erőforrást egy közös előfizetésben), de a hálózaton belül teljes körű vezérlést hozzáférhetnek a VDC fogalmakat és biztonsági határokat tartózkodási megfelelő, egy agyi virtuális hálózat és az erőforráscsoport egy központosított szabályzatban meghatározottak szerint.

## <a name="considerations-for-implementing-a-virtual-datacenter"></a>Egy virtuális adatközpont megvalósítása során megfontolandó szempontok

A VDC-megvalósítás megtervezésekor van több pivotal tényezőt érdemes figyelembe venni:

### <a name="identity-and-directory-services"></a>Identitás- és címtárszolgáltatásokat
Identitás- és címtárszolgáltatásokat rendszer egyik fontos szempontja a minden adatközpontot, mind a helyszíni és a felhőben. Hozzáférés és -engedélyezést, a VDC szolgáltatásai minden aspektusát identitás kapcsolódik. Győződjön meg arról, hogy csak a jogosult felhasználók és a folyamatok elérése az Azure-fiók és -erőforrások, az Azure, számos különböző típusú hitelesítő adatokat használ. Ezek közé tartoznak a jelszavak az Azure-fiók, a kriptográfiai kulcsokat, a digitális aláírások és a tanúsítványok eléréséhez. 

### <a name="identity-and-directory-service"></a>Identitás- és címtárszolgáltatás

Identitás- és címtárszolgáltatásokat rendszer egyik fontos szempontja a minden adatközpontot, mind a helyszíni és a felhőben. Hozzáférés és -engedélyezést, a VDC-megvalósítás szolgáltatásai minden aspektusát identitás kapcsolódik. Annak biztosítására, hogy csak a jogosult felhasználók és a folyamatok hozzá az Azure-fiók és -erőforrások, Azure, számos különböző típusú hitelesítő adatokat használ. Ezek közé tartoznak a jelszavak (az Azure-fiók eléréséhez), a kriptográfiai kulcsokat, a digitális aláírások és a tanúsítványok. [Az Azure multi-factor Authentication (MFA)] [ MFA] van egy további Azure-szolgáltatások eléréséhez szükséges biztonsági réteget. Az Azure MFA több egyszerű ellenőrzési lehetőség erős hitelesítést tesz lehetővé – telefonhívás, SMS vagy mobilalkalmazásbeli értesítés –, és lehetővé teszi ügyfeleink számára válassza a tetszőlegesen választhatnak.

Minden olyan nagyvállalati kell egy leíró egyedi azonosítók kezelését, a hitelesítés, engedélyezés, szerepkörök és jogosultságok belül vagy között VDC végrehajtásuk identity management folyamat megadása. Ez a folyamat célja a biztonság és hatékonyság közben csökkentésével költségeket, leállás és ismétlődő manuális feladatokat kell lennie.

Vállalati, illetve céget szükség, hogy a szolgáltatások különböző sor-az-üzleti (LOB) erőforrás-igényű vegyesen, és alkalmazottak gyakran különböző szerepkörök esetén, a különböző projektekkel rendelkezik. A VDC megfelelő együttműködését a különböző csapatok különböző, az adott szerepkör-definíciók, a rendszerekhez és a megfelelő cégirányítási fut minden egyes szükséges. A mátrix feladatkörei, hozzáférés és a jogok összetett folyamat lehet. Identitáskezelés a VDC keresztül valósul meg [ *Azure Active Directory* (Azure AD)] [ AAD] és a szerepköralapú hozzáférés-vezérlés (RBAC).

Egy olyan megosztott információkat működő az megkeresi, kezeli, felügyeli és rendszerezi a mindennapi elemek és hálózati erőforrásokat. Ezeket az erőforrásokat a köteteket, mappák, fájlok, nyomtatók, felhasználók, csoportok, eszközök és más objektumok is tartalmazhat. A hálózaton lévő minden egyes erőforrás számít egy objektumot a directory-kiszolgáló által. Erőforrásra vonatkozó adatokat egy adott erőforrás vagy objektumhoz tartozó attribútumok gyűjteménye van tárolva.

Jelentkezzen be az Azure Active Directory (Azure AD) támaszkodik az összes Microsoft online szolgáltatás, és egyéb az identitásukat. Az Azure Active Directory egy átfogó, magas rendelkezésre állású, felhőalapú identitás- és hozzáférés-kezelő megoldás, amely ötvözi az alapvető címtárszolgáltatásokat, a fejlett identitáskezelést és az alkalmazáshozzáférés-felügyeletet. Az Azure AD integrálható a helyszíni Active Directory egyszeri bejelentkezés az összes felhőalapú és a helyileg üzemeltetett a helyszíni alkalmazások engedélyezéséhez. A helyszíni Active Directory felhasználói attribútumok automatikusan szinkronizálhatók az Azure ad-hez.

Egyetlen globális rendszergazdája nem szükséges minden engedélyeket a VDC-megvalósítás. Ehelyett minden adott osztály, csoport, felhasználók vagy szolgáltatások a címtárszolgáltatás is rendelkezik a saját erőforrásaikat a VDC-megvalósítás belül kezeléséhez szükséges engedélyeket. Engedélyek strukturálja terheléselosztást igényel. Túl sok engedély akadályozhatják a teljesítmény hatékonyságát, és túl kevés vagy laza engedélyek növelhető a biztonsági kockázatokat. Azure szerepköralapú hozzáférés-vezérlés (RBAC) részletes hozzáférés-vezérlést az erőforrásokra a VDC-megvalósítás felajánlásával oldja meg a problémát, segítséget nyújt.

#### <a name="security-infrastructure"></a>Biztonsági infrastruktúra

Biztonsági infrastruktúra hivatkozik a VDC-megvalósítás adott virtuális hálózati szegmens a forgalom elkülönítése. Ez az infrastruktúra adja meg, hogyan bejövő és kimenő végezhető el a VDC-megvalósítás. Az Azure-alapú egy több-bérlős architektúra, amely megakadályozza a jogosulatlan vagy véletlen forgalom között központi telepítések segítségével a virtuális hálózat (VNet) elkülönítés, hozzáférés-vezérlési listák (ACL), a terheléselosztók, IP-szűrők és olyan forgalmi szabályzatok betöltése. Hálózati címfordítás (NAT) elkülöníti a belső hálózati forgalmat a külső forgalomtól.

Az Azure-hálót foglalja le az infrastruktúra-erőforrások bérlői számítási feladatok, és kezeli a kommunikációt egy telepítésen, és a virtuális gépek (VM). Az Azure hipervizor előírja a memóriák és a folyamatok elkülönítését virtuális gépek között, és biztonságosan a útvonalakat hálózati forgalmat a vendég operációs rendszerek bérlőihez.

#### <a name="connectivity-to-the-cloud"></a>A felhővel való kapcsolat

A VDC-megvalósítás kapcsolódnia kell az ügyfelek, partnerek és/vagy a belső felhasználók szolgáltatásajánlatokhoz külső hálózatokhoz. Kapcsolat igényeket hivatkozik, nem csak az internethez, hanem a helyszíni hálózatokhoz és az adatközpontokban.

Ügyfelek vezérlőelem, mely szolgáltatások hozzáférése és elérhető-e a nyilvános internetről használatával [Azure tűzfal] [ AzFW] , vagy a más típusú virtuális hálózati berendezések (nva-k) egyéni házirendek szerint használatával [felhasználó által megadott útvonalak][UDR], és a hálózati szűrés használatával [hálózati biztonsági csoportok][NSG]. Azt javasoljuk, hogy az összes internetkapcsolattal rendelkező erőforrásokat is biztosítható a [Azure DDoS Protection Standard][DDOS].

Vállalatok VDC végrehajtásuk a helyszíni adatközpontok és más erőforrásokhoz kapcsolódni szükségessé. Az Azure és helyszíni hálózat közötti kapcsolat egy rendkívül fontos szempont, egy hatékony architektúra tervezése során. A vállalatok a két különböző módon hozhat létre az összekapcsolás rendelkeznek: az interneten keresztül, illetve közvetlen privát kapcsolatok átvitel során.

Egy [ **az Azure Site-to-Site VPN** ] [ VPN] az összekapcsolási szolgáltatása a helyszíni hálózatok és a egy VDC végrehajtása, biztonságos keresztül létrehozott között az interneten keresztül titkosított kapcsolat (IPsec/IKE-alagutak). Az Azure Site-to-Site kapcsolat, rugalmas, gyors létrehozásához, és nincs szükség semmilyen további beszerzésekre, minden kapcsolat csatlakozzon az interneten keresztül.

A VPN-kapcsolatok, nagy számú [ **Azure virtuális WAN** ] [ vWAN] egy hálózati szolgáltatás, amely optimalizált, és automatizált ág ágba irányuló kapcsolat Azure-t. A Virtual WAN segítségével ágeszközöket csatlakoztathat és konfigurálhat az Azure-ral való kommunikációra. Csatlakozás és a konfigurálás elvégezhető manuálisan vagy egy virtuális WAN-partneren keresztül előnyben részesített szolgáltató eszközök használatával. Előnyben részesített szolgáltató eszközök használata lehetővé teszi a könnyű használat, kapcsolat és a konfigurációkezelés egyszerűsítése. Az Azure WAN beépített irányítópultjával azonnali hibaelhárítási segítséget kaphat, amelynek köszönhetően időt takaríthat meg, valamint könnyen áttekintheti a nagy kiterjedésű helyek közötti kapcsolatokat.

[**Az ExpressRoute** ] [ ExR] egy kapcsolatot az Azure szolgáltatás, amely lehetővé teszi a privát kapcsolatok a VDC-megvalósítás és bármely helyszíni hálózat között. Az ExpressRoute-kapcsolatok nem a nyilvános interneten keresztül, és nagyobb biztonságot, megbízhatóságot és egységes késést és nagyobb sebesség (legfeljebb 10 GB/s). Az ExpressRoute hasznos a VDC-implementációkban, ExpressRoute privát kapcsolatok társított megfelelőségi szabályok előnyeit magonkénti. Az ExpressRoute közvetlen,][ExRD] kapcsolódhat közvetlenül a Microsoft útválasztói 100 GB/s sávszélesség nagyobb igényekkel rendelkező ügyfél számára.

Az ExpressRoute-kapcsolatok általában telepítése magában foglalja egy ExpressRoute-szolgáltató révén. A gyorsan elkezdheti a munkát igénylő ügyfelek szokás kezdetben a VDC-megvalósítás és a helyszíni erőforrások közötti kapcsolatot, majd telepítse át az ExpressRoute-kapcsolat a helyek közötti VPN használatával során a fizikai, az összekapcsolás a szolgáltató számára befejeződött.

#### <a name="connectivity-within-the-cloud"></a>*Kapcsolat a felhőben*

[Virtuális hálózatok] [ VNet] és [virtuális hálózatok közötti társviszony-létesítés] [ VNetPeering] belül a VDC-implementáció alapvető hálózati kapcsolat szolgáltatások. Virtuális hálózat garantálja a VDC-erőforrások elkülönítési határ természetes, és a virtuális hálózatok közötti társviszony lehetővé teszi, hogy a másik virtuális hálózat azonos Azure-régióban vagy régiók között is között befolyásolása. Forgalomvezérlés egy virtuális hálózaton belül és virtuális hálózatok között meg kell egyeznie a biztonsági szabályok megadva a hozzáférés-vezérlési listák ([hálózati biztonsági csoport][NSG]), [hálózati virtuális berendezések ] [ NVA], és egyéni útválasztási táblázatokat ([UDR][UDR]).

## <a name="virtual-datacenter-overview"></a>Virtuális adatközpont áttekintése

### <a name="topology"></a>Topológia

_Küllős topológiájú_ egy olyan modell, a hálózati topológia virtuális adatközpont megvalósításának tervezéséhez. 

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

Az Azure-ban minden összetevő, a bármilyen típusú helyezünk üzembe egy Azure-előfizetésben. Az elkülönítés az Azure-előfizetések az Azure összetevőit objekty lobs s Hodnotou különböző, például a hozzáférési és engedélyezési szinteket beállítása követelményeinek is megfelelnek.

A VDC következő egyetlen megvalósítása is vertikálisan küllők, nagy számú ugyan csakúgy, mint minden informatikai rendszer platformok korlátozva van. A központi telepítés kötve van egy adott Azure-előfizetéssel, korlátozások és korlátokat rendelkező (például tekintse meg a maximális számú virtuális hálózatok közötti társviszony - [Azure-előfizetés és a szolgáltatások korlátozásai, kvótái és megkötései] [ Limits] részletekért). Azokban az esetekben, ahol a korlátok lehet probléma, az architektúra méretezhetők akár tovább csökkenthetők a modell egyetlen Központ-küllő küllős topológia a fürtre kiterjedő. Egy vagy több Azure-régióban több hubon összekapcsolható, virtuális hálózatok közötti társviszony-létesítés, az ExpressRoute, virtuális WAN vagy helyek közötti VPN használatával.

[![2]][2]

Több bevezetésével növeli a költségeket és a felügyeleti terhek, a rendszer. Csak indokolt volna méretezhetőséget, például a rendszer korlátok, és a redundancia és a végfelhasználói teljesítmény vagy a vész-helyreállítási például regionális replikáció. Forgatókönyvek a több hubon, az összes hubs igénylő törekedni kell a ugyanazokat a működési egyszerű szolgáltatásokat kínálnak.

#### <a name="interconnection-between-spokes"></a>Összekapcsolás küllők között

Egy egyetlen küllő belül összetett több rétegek számítási feladatok végrehajtására. Többrétegű konfigurációk használatával (egy minden szinthez) alhálózatok ugyanabban a vnetben, és NSG-k használata a flow szűrés valósítható meg.

Egy fejlesztő szeretné többrétegű számítási feladatok üzembe több virtuális hálózaton. A virtuális hálózatok közötti társviszony, küllők más küllők az azonos hub-vagy különböző hubs segítségével csatlakozhat. Egy jellemző példa rá történik a kérelmet feldolgozó kiszolgálók esetén egy küllős, vagy a virtuális hálózat. Az adatbázis különböző küllő vagy virtuális hálózaton üzembe helyezi. Ebben az esetben is könnyen összekapcsolható a küllő virtuális hálózatok közötti társviszony-, és ezáltal elkerülheti a hubon áthaladó. Annak érdekében, hogy a hub megkerülése nem megkerülése, fontos biztonsági vagy naplózási pontokat, amelyek csak az agyban előfordulhat, hogy létezik egy gondos architektúra és biztonsági felülvizsgálat kell végrehajtani.

[![3]][3]

Küllők is összekapcsolható, egy adott küllőre, amely a kiszolgálókezelési. Ezzel a módszerrel hoz létre egy kétszintű hierarchia: a küllőkben a magasabb szinten (0. szint) a hierarchia alacsonyabb agyként (1. szint) a hub válnak. A küllők a VDC-megvalósítási továbbítani a forgalmat a központi agyhoz úgy, hogy a forgalmat a cél a helyszíni hálózat vagy a nyilvános interneten átvitel is szükségesek. Hub két szinttel rendelkező vezet be, hogy bonyolult útválasztással, amely eltávolítja egy egyszerű küllős kapcsolat előnyeit.

Bár az Azure lehetővé teszi, hogy a komplex topológiáit, a VDC-fogalom alapelvek egyik ismételhetőség és az egyszerűség kedvéért. Energiabefektetést igénylő felügyelet minimalizálása érdekében az egyszerű küllős terv a VDC-referenciaarchitektúra, javasoljuk, hogy.

### <a name="components"></a>Összetevők

A virtuális adatközpont épül fel négy alapvető összetevő típusa: **Infrastruktúra**, **peremhálózatokon**, **számítási feladatok**, és **figyelési**.

Minden egyes összetevő típusát különböző Azure-szolgáltatások és erőforrások áll. A VDC-megvalósítás többféle összetevőket, és több változata létezik azonos típusú összetevő-példányok épül fel. Előfordulhat például, különböző alkalmazások képviselő számos különböző, logikailag elkülönített számítási példányokat. Végső soron hozhat létre a VDC használhatja ezeket a különböző összetevők típusok és példányok.

[![4]][4]

Az előző fogalmi architektúrájának áttekintése a VDC-különböző zónákhoz tartozhatnak a Központ-küllő topológia használható különböző összetevő típusok jeleníti meg. Az infrastruktúra-összetevőket látható az architektúra különféle részein.

Jó gyakorlat szerint általában hozzáférési jogokat és jogosultságokat kell Csoportalapú. Egyéni felhasználók helyett a csoportok kezelésével foglalkoznak megkönnyíti a hozzáférési házirendek azáltal, hogy a különböző csapatokkal felügyelet egységes módon.  és konfigurációs hibák minimalizálása segédeszközök. Hozzárendelése és eltávolítása a felhasználók és a megfelelő csoportok segítségével egy adott felhasználó a jogosultságok rendszeres frissítése.

Minden egyes szerepkör-csoport egy egyedi előtagot kell rendelkeznie a nevüket. Ezt az előtagot egyszerűen azonosíthatja, hogy melyik csoporthoz társítva mely számítási feladatok. Például egy hitelesítési szolgáltatást üzemeltető számítási feladatok lehet nevű csoport **AuthServiceNetOps**, **AuthServiceSecOps**, **AuthServiceDevOps**, és **AuthServiceInfraOps**. Szerepkörök, vagy nem adott szolgáltatásokhoz kapcsolódó szerepkörök központosított, előfordulhat, hogy a tartalmazni **Corp**. Például **CorpNetOps**.

Számos szervezet egy változata, a következő csoportok használatával adja meg a főbb információkat a szerepkörök:

-   A központi informatikai csoport **Corp,** szabályozhatja az infrastruktúra-összetevőket tulajdonosi jogosultságokkal rendelkezik. Példák a hálózati és biztonsági. A csoport közreműködői az előfizetésen szerepét, a hub és a hálózati közreműködő jogok ellenőrzése a küllők kell érnie. Nagyméretű szervezetek gyakran osztott e felügyeleti feladatokat több csapatok között. Példák a hálózati műveletek **CorpNetOps** kizárólagos összpontosít a hálózatkezelés és a egy biztonsági műveletek a csoport **CorpSecOps** csoport felelős a tűzfal- és biztonsági szabályzatot. Ebben az esetben két különböző csoportot kell létrehozni ezeket egyéni szerepkör-hozzárendelés.
-   A fejlesztési-tesztelési csoport **AppDevOps,** felelősség alkalmazás vagy szolgáltatás számítási feladatok üzembe helyezéséhez. Ez a csoport vesz igénybe, szolgáltatott infrastruktúra telepített példányainak a virtuális gép közreműködő szerepkörének vagy egy vagy több PaaS közreműködő szerepkört. Lásd: [beépített szerepkörök az Azure-erőforrások][Roles]. A fejlesztési-tesztelési csapata is látható-e a biztonsági házirendek, az NSG-k és útválasztási házirend, udr-EK, a hub vagy az egy adott küllő belül lehet szükség. Mellett a számítási feladatokhoz közreműködői szerepkört ennek a csoportnak módosítania kell a hálózati olvasó szerepkört.
-   A művelet és a karbantartási csoport, **CorpInfraOps** vagy **AppInfraOps,** az éles számítási feladatok felügyeletének terhei rendelkezik. Ennek a csoportnak kell lennie a számítási feladatokat éles üzemű előfizetéseket az előfizetés közreműködői. Egyes szervezetek is előfordulhat, hogy kiértékelése, ha egy további eszkalációs támogatási csapat szerepkörrel rendelkező, előfizetés közreműködője az éles és a központi agyhoz előfizetés szükséges. A további csoport potenciális konfigurációs problémákat javítja az éles környezetben.

A VDC célja, így a központi informatikai csoportok kezelése a hub létrehozott csoportok megfelelő csoportok számítási feladatok szintjén. Csak a hub-erőforrások kezelése, mellett a központi informatikai csoport az külső hozzáférés és a legfelső szintű engedélyeket az előfizetésben. Munkaterhelés-csoport is tudnak erőforrások és a virtuális hálózat, egymástól függetlenül a központi informatikai engedélyeket.

A VDC particionálása biztonságosan üzemeltetésére több projektet különböző sorok üzleti (LOB) között. Minden projekthez szükséges különböző izolált környezetben (fejlesztői, foglalja annak, éles környezetben). Ezekben a környezetekben mindegyike külön Azure-előfizetések elkülönítéshez természetes.

[![5]][5]

A fentebbi ábra bemutatja azokat a szervezet projekteket, a felhasználók és csoportok és, az Azure-összetevők telepítve vannak a környezetek közötti kapcsolat.

Általában az informatikai környezetben (vagy szint) egy rendszer, amelyben több alkalmazást üzembe és végrehajtva. A nagyobb vállalatok használata a fejlesztési környezet (amelyben módosításait eredetileg és tesztelt) és a egy éles környezetben (mi végfelhasználók használja). Szkriptjét el vannak különítve, gyakran több átmeneti környezetek között a többszakaszos üzembe helyezés (Bevezetés) engedélyezését, tesztelés és a visszaállítás problémák felmerülése esetén szükséges elhárítására. Üzembe helyezési architektúra jelentős eltérések lehetnek, de általában fejlesztési (fejlesztői), és az éles (ÉLES) egészen az alapvető folyamat továbbra is követi.

Az Azure DevOps fejlesztési és tesztelési, átmeneti és éles környezetek számára foglalja annak az ilyen típusú több rétegből álló környezeteket egy gyakori architektúráját áll. Szervezetek használhatnak egyetlen vagy több Azure AD-bérlőt, hozzáférés és ezekben a környezetekben jogosultságok meghatározására. Az előző ábrán látható egy esetet, ha két különböző Azure AD-bérlőt használ: egyet az Azure DevOps és foglalja annak és egy másik kizárólag az éles környezetben.

Jelen másik Azure AD-bérlők kikényszeríti a környezetek elkülönítése. Az azonos csoportján felhasználói számára, például a központi informatikai, szüksége van egy másik Azure eléréséhez másik URI használatával történő hitelesítéshez AD-bérlő módosítása a szerepkörök vagy engedélyek vagy az Azure DevOps- vagy éles környezetben a projekt. A különböző felhasználói hitelesítések eléréséhez a különböző környezetek jelenléte csökkenti a szolgáltatáskimaradásokat és egyéb az emberi hibák által okozott problémák.

#### <a name="component-type-infrastructure"></a>Összetevő típusa: Infrastruktúra

Ezen összetevő típusa, ahol a legtöbb támogató infrastruktúra található. Is, a központi informatikai, biztonság, megfelelőség részlegeknek legtöbb idejüket, illetve.

[![6]][6]

Infrastruktúra-összetevőket egy összekapcsolása a VDC-megvalósítás különböző összetevőire vonatkozó meg, és mind a küllős topológia az szerepelnek. A felelősség kezelésével és karbantartásával az infrastruktúra-összetevőket általában hozzá van rendelve a központi informatikai és/vagy a biztonsági csapatnak.

Az informatikai infrastruktúra csapat az elsődleges feladatok egyikét, az IP-cím sémák konzisztenciájának biztosítása a vállalaton belül. A VDC-megvalósítás rendelt magánhálózati IP-címterület konzisztensnek kell lennie, és nincsenek átfedésben a magánhálózati IP-címek hozzárendelve a helyszíni hálózatokon.

Bár a helyszíni peremhálózati útválasztói vagy az Azure-környezetek NAT IP-címütközés elkerülheti, komplikációk hozzáadja az infrastruktúra-összetevőket. Egyszerű a felügyeleti az egyik fő célja a VDC a így használ a NAT IP-problémák kezeléséhez nem javasolt megoldás.

Infrastruktúra-összetevőket kell a következő funkciókat:

-   [**Identitás- és címtárszolgáltatásokat**][AAD]. Az Azure-ban minden erőforrástípus a hozzáférést az identitás a címtárszolgáltatások tárolt vezérli. A címtárszolgáltatás nem csak a felhasználók listáját, de az erőforrásokhoz való hozzáférési jogosultságokat is tárol egy adott Azure-előfizetést. Ezek a szolgáltatások is létezik, csak felhőalapú, vagy is szinkronizálható a helyszíni Active Directoryban tárolt identitással.
-   [**Virtuális hálózat**][VPN]. Virtuális hálózatok egyik fő összetevője a VDC, és lehetővé teszi a forgalomelkülönítési határok létrehozása az Azure platformon. Virtuális hálózat egy vagy több virtuális hálózati szegmenseket, amelyek mindegyike egy adott IP-hálózati előtagot (alhálózat) áll. A virtuális hálózat egy belső peremhálózati területre, ahol IaaS virtuális gépek és PaaS-szolgáltatások létesíthet a privát kommunikációs határozza meg. Virtuális gépek (és a PaaS-szolgáltatások) egy virtuális hálózat közvetlenül a virtuális gépeket nem lehet kommunikálni (és PaaS-szolgáltatások) egy másik virtuális hálózatban, akkor is, ha mindkét virtuális hálózat hozzák létre az adott ügyfél azonos előfizetéshez tartozó. Elkülönítés kritikus tulajdonság, amely biztosítja az ügyfél virtuális gépei, kommunikációs privát virtuális hálózaton belül marad.
-   [**UDR**][UDR]. A virtuális hálózati adatforgalmat a rendszer útválasztási táblázat alapján alapértelmezés szerint. Egy felhasználó által definiált útvonal a hálózati rendszergazdák egyéni útvonaltábla társítható felülírja a rendszer útválasztási táblázat viselkedését, és a egy virtuális hálózaton belüli kommunikációs útvonal definiálhat egy vagy több alhálózatot. Udr-EK jelenléte garantálja, hogy a kimenő forgalom származó adott egyéni virtuális gépeket és/vagy a hálózati virtuális berendezések és a jelen, a hubot és a küllők terheléselosztók küllő átmenő.
-   [**NSG-T**][NSG]. A hálózati biztonsági csoport az szűrést IP-források, IP-cél, protokollok, IP-Forrásportok és cél IP-portok az adatforgalom-kiszolgálóként működő biztonsági szabályok listája. Az NSG-t egy alhálózathoz, az Azure virtuális Gépekhez vagy mindkettőhöz társított virtuális hálózati kártya alkalmazhatók. Az NSG-k nélkülözhetetlenek valósítható meg egy megfelelő adatfolyam vezérlés, a hubot és a küllők az. Az NSG-t által nyújtott biztonság szintje egy függvényt, mely portok megnyitása, és milyen célra. További VM-enkénti szűrők például engedélyezze az IPtables állomásalapú tűzfal vagy a Windows tűzfal kell telepíteniük.
-   [**DNS**][DNS]. A névfeloldás az erőforrások a virtuális hálózatok a VDC-végrehajtás DNS-en keresztül van megadva. Az Azure DNS szolgáltatást is nyújt [nyilvános] [ DNS] és [privát] [ PrivateDNS] névfeloldását. Privát zónák névfeloldásához egy virtuális hálózaton belül, és több virtuális hálózaton. Privát zónák csak span ugyanabban a régióban, hanem a régiók és -előfizetések közötti virtuális hálózatok között is. A nyilvános feloldásához az Azure DNS biztosít egy üzemeltetési szolgáltatás DNS-tartományok biztosítani a névfeloldást a Microsoft Azure infrastruktúráját használja. Ha tartományait az Azure-ban üzemelteti, DNS-rekordjait a többi Azure-szolgáltatáshoz is használt hitelesítő adatokkal, API-kkal, eszközökkel és számlázási információkkal kezelheti.
-   [**Előfizetés** ] [ SubMgmt] és [ **erőforráscsoport felügyeleti**][RGMgmt]. Előfizetés az erőforrások több csoport létrehozása az Azure-ban a természetes határok határozza meg. Egy előfizetés erőforrásait együtt vannak összeszerelt nevű erőforráscsoportok logikai tárolók. Az erőforráscsoport olyan logikai csoportot a VDC-végrehajtás az erőforrások rendszerezéséhez jelöl.
-   [**RBAC**][RBAC]. RBAC, keresztül térkép szervezeti szerepkörhöz adott Azure-erőforrások, így korlátozhatja a felhasználók csak bizonyos részét műveletek hozzáférési jogokkal együtt. Az RBAC lehet hozzáférést biztosítani a megfelelő szerepkört rendelhet a felhasználók, csoportok és alkalmazások megfelelő hatókörébe. Szerepkör-hozzárendelés hatóköre az Azure-előfizetés, erőforráscsoport vagy egyetlen erőforrás lehet. Az RBAC lehetővé teszi, hogy az engedélyek öröklődése. Egy szülő hatókörben hozzárendelt szerepkör is hozzáférést biztosít az ebben lévő gyermekei. Az RBAC használatával, feladatköröket, és csak olyan mértékű hozzáférést biztosítson a felhasználók számára, amelyek a feladataik elvégzéséhez szükségük van. Például, hogy a virtuális gépek található egy előfizetésben, kezelése, miközben egy másik SQL-adatbázisok kezelheti ugyanazon az előfizetésen belül egy alkalmazott RBAC használatát.
-   [**Virtuális hálózatok közötti társviszony-létesítés**][VNetPeering]. A a VDC infrastruktúra létrehozásához használt alapvető funkció a virtuális hálózatok közötti Társviszony olyan mechanizmus, amely összeköti a két virtuális hálózatok (Vnetek) ugyanabban a régióban az Azure-adatközpont-hálózat, vagy az Azure világszerte gerinchálózatát használva régióban.

#### <a name="component-type-perimeter-networks"></a>Összetevő típusa: Szegélyhálózat

[Szegélyhálózat] [ DMZ] összetevők engedélyezése a helyszíni vagy a fizikai adatközpont-hálózatok, valamint minden olyan kapcsolat, az Internet felé és felől közötti hálózati kapcsolat. Emellett akkor is, ahol a hálózati és biztonsági valószínűleg részlegeknek legtöbb idejüket.

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

A fenti ábrán annak érdekében, két régebben a hozzáférést az internet és a egy helyszíni hálózat egyaránt megtalálhatók a DMZ-t és vWAN hubs a jeleníti meg. Az agyban DMZ-t a peremhálózat és a internet is vertikális felskálázás objekty lobs s Hodnotou, több farmok webalkalmazás-tűzfalak (alsóbb) és/vagy az Azure-tűzfalak használatával nagy számú támogatásához. VWAN központban ág és ág a kapcsolatot az Azure nagy mértékben skálázható ág történik VPN vagy ExpressRoute-n keresztül igény szerint.

[**Virtuális hálózatok**][VNet]. A hub általában épülő üzemeltetéséhez a különböző szolgáltatások, szűrése és vizsgálja meg a bejövő és kimenő forgalmat az interneten keresztül nva-k, waf-eszközzel, több alhálózattal rendelkező virtuális hálózat és az Azure Application Gateway-példány.

[**UDR** ] [ UDR] Udr használatával ügyfelek telepíthetnek tűzfalak, IDS/IPS, és más virtuális készülékeket és a hálózati forgalom irányítása révén e biztonsági készülékek biztonsági határ házirend betartatása naplózás és vizsgálat. Udr-EK hozhat létre mind a küllős topológia az garantálja, hogy az adott egyéni virtuális gépeket, hálózati virtuális berendezések keresztül forgalom továbbítására, és a VDC-implementáció által használt terheléselosztók. Garantáljuk, hogy a megfelelő virtuális berendezésre megtalálhatók a küllő átvitel közben a virtuális gépek által generált forgalom, egy udr-t kell a belső terheléselosztó előtérbeli IP-címét állítsa a következő ugrás a küllő alhálózata állítani. A belső terheléselosztó osztja el a belső bejövő forgalmának a virtuális készülékek (load balancer háttérkészlethez).

[**Az Azure tűzfal** ] [ AzFW] egy felügyelt, felhőalapú és hálózati biztonsági szolgáltatás, amely védelmet nyújt az Azure Virtual Network-erőforrások. Teljes állapot-nyilvántartó tűzfal beépített magas rendelkezésre állás és méretezhetőség korlátlan felhőalapú szolgáltatásként is. Központilag hozhatja létre, érvényesítheti és naplózhatja az alkalmazás- és hálózatelérési szabályzatokat az előfizetésekre és a virtuális hálózatokra vonatkozóan. Azure-tűzfalon statikus nyilvános IP-címet használ a virtuális hálózati erőforrások. Lehetővé teszi a tűzfalon kívül a virtuális hálózatból érkező forgalom azonosítására. A szolgáltatás teljesen integrálva van az Azure Monitorral a naplózás és az elemzés érdekében.

[**Hálózati virtuális berendezések**][NVA]. Az agyban a peremhálózaton, az internet-hozzáféréssel rendelkező általában egy Azure tűzfal-példányt, vagy a tűzfalak vagy a webalkalmazási tűzfal (WAF) farm keresztül kezeli.

Különböző objekty lobs s Hodnotou használata általában számos webes alkalmazásokhoz. Ezek az alkalmazások általában különböző biztonsági rések és az esetleges biztonsági rések. sorból. Webalkalmazás-tűzfalak egy speciális típusú részletesebben olvashat róluk, mint az általános tűzfal webes alkalmazások, HTTP/HTTPS-elleni támadások észleléséhez használt terméket. Képest hagyomány tűzfal-technológiája, alsóbb rendelkeznek kívánt belső webkiszolgálók védelme a fenyegetésekkel szemben.

Az Azure-tűzfal vagy NVA tűzfalak egyaránt használja egy közös felügyeleti sík, a küllők lévő üzemeltetett a munkaterhelések védelme érdekében a biztonsági szabályok vannak beállítva, és a hozzáférés-vezérlése a helyszíni hálózatok. Az Azure-tűzfal rendelkezik beépített, méretezhetőséget, mivel NVA tűzfalak manuálisan méretezhetők egy terheléselosztó mögé. Általában egy tűzfal farm rendelkezik kisebb specializált képest egy webalkalmazási Tűzfallal rendelkező szoftvert, de szélesebb körű alkalmazás hatóköre szűrését, és vizsgálja meg a bejövő és kimenő forgalmat bármilyen típusú. Ha egy NVA-módszert használja, azok található és az Azure marketplace-ről üzembe helyezett.

Használjon egy Azure-tűzfalak (vagy az nva-k) a forgalom az interneten, és egy másikat a helyszíni forgalom származik. Biztonsági kockázat, csak egy tűzfalak használatával is, nincs biztonsági határ a két fajta hálózati forgalom között lehetővé teszi. Különálló tűzfalat rétegek használatával csökkenti az ellenőrző biztonsági szabályok összetettségét, és egyértelművé teszi, hogy melyik szabály vonatkozik, hogy mely bejövő hálózati kérésekre.

Azt javasoljuk, hogy az internetről érkező forgalomhoz Azure tűzfal-példányra, vagy az nva-k, egy készletét használja. A helyszíni forgalom egy másik használja. Biztonsági kockázat csak egy tűzfalak használatával is, nincs biztonsági határ a két fajta hálózati forgalom között lehetővé teszi. Különálló tűzfalat rétegek használatával csökkenti az ellenőrző biztonsági szabályok összetettségét, és egyértelművé teszi, hogy melyik szabály vonatkozik, hogy mely bejövő hálózati kérésekre.

[**Az Azure Load Balancer** ] [ ALB] kínál a magas rendelkezésre állású 4. réteg (TCP, UDP) szolgáltatás, amely juttathatja el a bejövő forgalom meghatározott egy elosztott terhelésű készlet szolgáltatási példányai között. A terheléselosztó előtérbeli végpontokból (nyilvános IP-végpontok vagy magánhálózati IP-végpontok) küldött forgalmat újra vagy anélkül címfordítás számára egy háttér-IP-címkészlet (példák folyamatban; Hálózati virtuális berendezések vagy virtuális gépeken).

Az Azure Load Balancer is megvizsgálja, valamint a különböző kiszolgálópéldányok állapotát, és ha egy mintavételező nem válaszol a terheléselosztó nem irányít forgalmat küld a nem megfelelő állapotú példány. Az a VDC a küllős topológia a külső terheléselosztó van telepítve. A hubon a load balancer segítségével hatékonyan juthatnak el a forgalmat a küllők az szolgáltatásaihoz, és a küllők a load Balancer terheléselosztók segítségével felügyelhető az alkalmazásforgalomba.

[**Az Azure bejárati ajtajának** ] [ AFD] (AFD), a Microsoft magas rendelkezésre állású és méretezhető webes gyorsítás alkalmazásplatform, globális HTTP-terheléselosztó, alkalmazásvédelem és a Content Delivery Network. 100-nál több helyen, a Microsoft Edge globális hálózatában fut, a AFD segítségével hozhat létre, működtethessék és horizontális felskálázása dinamikus webes alkalmazások és a statikus tartalom is. AFD világszínvonalú végfelhasználói teljesítmény, egységes területi/blokk karbantartási automation, BCDR automation, egységes ügyfelek és a felhasználók információt, gyorsítótár és service insights segítségével biztosít. A platformot kínál a teljesítmény, a megbízhatóság és a támogatási SLA-k, megfelelőségnek és a fejlett, amellyel a naplózható biztonsági eljárások üzemeltetett, és az Azure által natívan támogatja.

[**Az Application Gateway** ] [ AppGW] Microsoft Azure Application Gateway egy alkalmazáskézbesítési vezérlőt (ADC) biztosító dedikált virtuális berendezés-szolgáltatás, 7 terheléselosztási réteg különböző ajánlat lehetőséget nyújt alkalmazásának. Ez lehetővé teszi, hogy optimalizálhatják a webfarmok termelékenységét a processzorigényes SSL-lezárások application Gateway felé történő kiszervezésével. Ezen túlmenően egyéb 7. rétegbeli útválasztási lehetőségeket is kínál, beleértve a bejövő forgalom ciklikus időszeleteléses elosztását, a cookie-alapú munkamenet-affinitást, az URL-alapú útválasztást, valamint egyetlen Application Gateway mögött több webhelyet is üzemeltethet. Az Application Gateway WAF termékváltozata tartalmaz egy webalkalmazási tűzfalat is, Ez a Termékváltozat webalkalmazásokat a gyakori internetes biztonsági rések és az azokat kihasználó támadások ellen védelmet biztosít. Az Application Gateway szolgáltatást internetes átjáróként, csak belső használatú átjáróként vagy a kettő kombinációjaként lehet konfigurálni. 

[**Az Application Gateway** ] [ AppGW] alkalmazáskézbesítési vezérlőt (ADC) biztosító dedikált virtuális berendezés van szolgáltatásként, a különböző réteg 7 terheléselosztási lehetőséget nyújt alkalmazásának. Az Application Gateway-példány processzorigényes SSL-lezárások felé történő kiszervezésével optimalizálhatják a webfarmok termelékenységét. Egyéb 7. rétegbeli útválasztási lehetőségeket, amelyek tartalmazzák a következő példákban is biztosít: 
* Bejövő forgalom Ciklikus időszeleteléses elosztását. 
* Cookie-alapú munkamenet-affinitás. 
* URL-cím-alapú útválasztás. 
* Lehetővé teszi az Egypéldányos Application Gateway mögött több webhelyet is üzemeltethet. Webalkalmazási tűzfal (WAF) Application Gateway WAF Termékváltozatában részeként is tartalmaz. Ez a Termékváltozat webalkalmazásokat a gyakori internetes biztonsági rések és az azokat kihasználó támadások ellen védelmet biztosít. Az Application Gateway konfigurálható egy internetre irányuló átjáró, egy belső átjáró vagy mindkettőt. 

[**Nyilvános IP-címek**][PIP]. Az egyes Azure-funkciókról társíthatja a Szolgáltatásvégpontok egy nyilvános IP-címre, hogy az erőforrás elérhető lesz az interneten. Ez a végpont hálózati címfordítás (NAT) használatával irányíthatja a forgalmat a belső cím és port, az Azure-beli virtuális hálózaton. Ez az elérési út a külső forgalmat a virtuális hálózatban módját. Meghatározhatja az adatforgalom átadott és hol és hogyan lehet a virtuális hálózatra lefordított nyilvános IP-címeket is beállíthatja.

[**Az Azure DDoS Protection Standard** ] [ DDOS] képest további veszélyelhárítási szolgáltatásokat nyújt a [alapszintű szolgáltatási] [ DDOS] hangolt szint kifejezetten az Azure virtuális hálózati erőforrásokat. A DDoS Protection Standard engedélyezése egyszerű, és nem kell application módosítani. Az alkalmazásvédelmi szabályzatok hangolt dedikált forgalomfigyelést és gépi tanulási algoritmus segítségével. A virtuális hálózatokon üzembe helyezett erőforrásokhoz rendelt nyilvános IP-címek szabályzatok érvényesek. Példák Azure Service Fabric, Azure Load Balancer és Azure Application Gateway-példány. Valós idejű telemetriai adatokat az Azure Monitor nézetek keresztül érhető el, a támadás során, és az előzmények. Alkalmazásréteg-védelem az Azure Application Gateway webalkalmazásokhoz használható tűzfal segítségével is hozzáadhat. Védelmet biztosítanak a nyilvános IP-címek IPv4 Azure.

#### <a name="component-type-monitoring"></a>Összetevő típusa: Figyelés

Figyelési összetevők látható-e, és a más összetevők adattípusok riasztási adja meg. Csapatok mindegyikével kell férniük a figyelést az összetevők és szolgáltatások hozzáféréssel rendelkeznek. Vannak egy központi súgó segélyszolgálathoz vagy a műveleti csoportok, az adatokhoz, ezek az összetevők által biztosított integrált hozzáférést igényelnek.

Az Azure a naplózást és a szolgáltatások az Azure-ban tárolt erőforrások viselkedését követni a különböző típusú kínál. Cégirányítási és vezérlés az Azure-beli számítási feladatok alapján nem csupán a teljesítményadatok gyűjtését. naplófájl, de is specifikus eseményindító műveletek lehetővé teszi az események jelentett.

[**Az Azure Monitor**][Monitor]. Az Azure több szolgáltatást is tartalmaz, amelyek önmagukban is betöltenek adott szerepköröket vagy végrehajtanak bizonyos feladatokat a monitorozás területén. Ezek a szolgáltatások együtt egy átfogó megoldást nyújtanak az alkalmazások és az azt támogató Azure-erőforrások telemetriaadatainak gyűjtésére és elemzésére, valamint ezek alapján műveletek végrehajtására. Emellett használhatók kritikus fontosságú helyszíni erőforrások monitorozására egy hibrid monitorozási környezet biztosítása érdekében. A rendelkezésre álló eszközök és adatok megismerése az első lépés az alkalmazás teljes monitorozási stratégiájának kidolgozásában.

Naplók az Azure-ban két fő típusa van:

-   A [Azure-tevékenységnapló][ActLog]korábban néven **műveleti naplók**, az az Azure-előfizetéshez tartozó erőforrásokon végrehajtott műveletekkel betekintést nyújt. Ezek a naplók a vezérlősík események az előfizetésekre vonatkozó jelentést. Minden Azure-erőforrás naplókat eredményez.

-   [Az Azure Monitor-diagnosztikai naplók] [ DiagLog] rendszer részletes, gyakori adatokat az erőforrás a művelet az erőforrás által létrehozott naplók. Ezek a naplók a tartalom erőforrás típusa szerint változó.

[![9]][9]

Fontos, hogy az NSG-k naplók, különösen ezeket az információkat nyomon:

-   [Eseménynaplók] [ NSGLog] információkat tartalmaz a virtuális gépeket és példányszerepköröket MAC-cím alapján milyen NSG-szabályok érvényesek.
-   [Naplók számláló] [ NSGLog] nyomon követése az egyes NSG-szabályok a forgalom engedélyezése vagy megtagadása volt futtatva futtatásainak számát.

Az összes napló tárolható a naplózási, elemzési statikus vagy biztonsági mentési célból az Azure storage-fiókot. Azure-tárfiók tárolja a naplókat, ha ügyfelek különböző típusú keretrendszerek használatával lekérni, -előkészítési, elemzése, és jelenítse meg az adatokat jelenti az állapotot és a felhőbeli erőforrások állapotát. 

A nagyobb cégeknek is kell már szerezték be olyan szabványos keretrendszer, a helyszíni rendszerek figyelése. Integrálható a felhőben üzemelő példányok által létrehozott naplók a keretrendszer kiterjesztheti azokat. Az [Azure Log Analytics] [https://docs.microsoft.com/en-us/azure/log-analytics/log-analytics-queries], szervezetek megtarthatja az összes naplózási a felhőben. A log Analytics felhőalapú szolgáltatásként van megvalósítva. Ezért kell, és gyorsan és az infrastrukturális szolgáltatásokra fordítandó minimális kiadások mellett. A log Analytics is integrálhatja a System Center-összetevőket, például a System Center Operations Manager bővítése a meglévő felügyeleti beruházások kiterjeszthetők a felhőre. 

A log Analytics szolgáltatása az Azure-ban, amely segít összegyűjtését, összekapcsolását, keressen, és operációs rendszerek, alkalmazások és infrastruktúra felhőalapú összetevők által generált napló- és teljesítményadatokat az adatokkal műveleteket végezni. Ügyfelek biztosít a rekordok elemezze a feladatait a VDC-példányában integrált keresést és egyéni irányítópultok segítségével valós idejű az operational insights.

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

#### <a name="component-type-workloads"></a>Összetevő típusa: Számítási feladatok

Számítási feladatok összetevői a tényleges alkalmazások és szolgáltatások-ket. Emellett akkor is, ahol az alkalmazás fejlesztői részlegeknek legtöbb idejüket.

A számítási lehetőségek száma végtelen. A lehetséges munkaterhelés-típusok közül a következők:

**Belső ÜZLETÁGI alkalmazások**: Üzleti alkalmazások olyan nagyvállalati a folyamatban lévő műveletet, kritikus fontosságú számítógépes alkalmazásokat. LOB-alkalmazások néhány közös jellemzőkkel rendelkeznek:

-   **Interaktív** természetéből. Adatok bevitele, és a visszaadott eredmények vagy jelentések.
-   **Adatvezérelt**&mdash;intenzív az adatok gyakran használják az access-adatbázisok vagy más tárolási.
-   **Integrált**&mdash;belül vagy a szervezeten kívül más rendszerekkel való integrációt kínálnak.

**Ügyfelek által használt webhelyek (interneten vagy belső kapcsolódó)**: A legtöbb alkalmazás, amely az internettel olyan webhelyek. Az Azure lehetőséget nyújt a webhelyek IaaS virtuális gépen vagy a futtatásához egy [Azure Web Apps] [ WebApps] hely (PaaS). Az Azure Web Apps-integráció, amelyek lehetővé teszik a központi telepítés, a Web Apps, a küllős hálózati zónában lévő virtuális hálózatok támogatja. Internetkapcsolattal rendelkező belső webhelyek nem szükséges nyilvános internetes végpont teszi közzé, mert az erőforrásokat a magánhálózati vnet nem internetes irányítható Magáncímeket keresztül érhető el.

**Big Data típusú adatok/elemzések**: Ha adatokat kell a vertikális felskálázás akár nagy, adatbázisok előfordulhat, hogy nem vertikális felskálázás megfelelően. Hadoop technológiát kínál a rendszer a nagy számú csomópont elosztott lekérdezések párhuzamos futtatásához. Az ügyfél rendelkezik lehetőség data számítási feladatok futtatása az IaaS virtuális gépek vagy PaaS ([HDInsight][HDI]). HDInsight helyalapú vnetbe üzembe helyezést támogatják, is telepíthető egy adott küllőre a VDC-fürtön.

**Események és üzenetküldési**: [Az Azure Event Hubs] [ EventHubs] egy rendkívül nagy kapacitású, telemetriai feldolgozó szolgáltatás, amely összegyűjti a összegyűjteni, átalakítani és több millió eseményt tárolja. Elosztott streamelési platform biztosít alacsony késéssel és konfigurálható idejű adatmegőrzést biztosít, lehetővé téve nagy mennyiségű telemetria betöltését az Azure-ba, és több alkalmazás adatokat olvasni. Az Event hubs szolgáltatással egyetlen streammel támogathat valós idejű és kötegelt folyamatok.

Megvalósíthat egy nagy megbízhatóságú felhőalapú üzenetküldő szolgáltatásra alkalmazások és szolgáltatások közötti [Azure Service Bus][ServiceBus]. Kínál aszinkron közvetítőalapú üzenettovábbítás ügyfél és kiszolgáló közötti, első-first out (FIFO) üzenetkezelés, strukturált és közzétételi és feliratkozási képességek.

[![10]][10]

### <a name="making-the-vdc-highly-available-multiple-vdcs"></a>A VDC magas rendelkezésre állásúvá: több Ügyfélszempontokat

Eddig ez a cikk rendelkezik kialakításával foglalkozik, egy egyetlen VDC, mely az alapvető összetevők és architektúra, amely hozzájárul a rugalmasság. Azure-funkciók például az Azure load balancer, nva-k, a rendelkezésre állási csoportok, méretezési csoportokat, illetve más mechanizmusok, amelyekkel szilárd SLA-szintek beépítése az éles-szolgáltatások rendszer hozzájárul.

Azonban egy egyetlen VDC implementálása jellemzően egy adott régión belül, mert lehet bármely főbb szolgáltatáskimaradás, amely a teljes régióra hatással van kitéve. Ügyfeleink számára, amelyeknek nagy SLA-k a szolgáltatás üzemelő példányainak helyezni különböző régiókban lévő két (vagy több) VDC megvalósításokban ugyanazon a projekten keresztül kell védeni.

SLA vonatkozik kívül számos gyakori forgatókönyvek, ahol üzembe helyezése a VDC-megvalósításokhoz több értelme:

-   Regionális és globális jelenlétét.
-   Vészhelyreállítás.
-   Egy olyan mechanizmust átirányít az adatközpontok közötti forgalmat.

#### <a name="regionalglobal-presence"></a>Regionális és globális jelenlét

Azure-adatközpontokban világszerte számos régióban találhatók. Több Azure-adatközpontok kiválasztásakor ügyfeleknek kell figyelembe venni két tényezőt összefüggő: földrajzi távokat és a késés. A legjobb felhasználói élményt nyújtani, értékelje ki a minden VDC megvalósítása, valamint az egyes VDC végrehajtása és a végfelhasználók közötti távolságot közötti földrajzi távolságtól.

A régió, mely VDC megvalósításokhoz üzemeltetett meg kell felelniük az alapján, amely a szervezet működik minden jogi joghatósága által létrehozott szabályozási követelményeknek.

#### <a name="disaster-recovery"></a>Vészhelyreállítás

A vész-helyreállítási terv kialakítása a számítási feladatok és az állapota, között implementálható VDC munkaterhelések szinkronizálhatók típusú függ. Ideális esetben a legtöbb ügyfél szerint egy gyors feladatátvételi mechanizmust, és előfordulhat, hogy alkalmazás adatok szinkronizálását több VDC-megvalósítások üzemelő példányok között. Azonban vész-helyreállítási tervek tervezésekor fontos vegye figyelembe, hogy a legtöbb alkalmazás érzékenyek a késést, amely az adatszinkronizálás okozhatja.

Szinkronizálás és a szívverés-figyelés az alkalmazások különböző VDC-megvalósításokban ez szükséges a hálózaton keresztül kommunikálnak. Különböző régiókban lévő két VDC-implementáció keresztül csatlakozhat:

-   – Virtuális hálózatok közötti társviszony-létesítés virtuális hálózatok közötti Társviszony kapcsolódhatnak hubs régióban
-   Az ExpressRoute privát társviszony-létesítés ugyanahhoz az ExpressRoute-kapcsolatcsoporthoz csatlakoztatott minden VDC megvalósítása a hubs
-   több ExpressRoute-kapcsolatcsoporttal a vállalati gerinchálózatán keresztül kapcsolódik, és a több VDC megvalósításokban csatlakozik az ExpressRoute-Kapcsolatcsoportok
-   Site-to-Site VPN-kapcsolatok között a VDC-megvalósítások az egyes Azure-régiókban hub időzónája

Virtuális hálózatok közötti Társviszony- vagy ExpressRoute-kapcsolatok általában a hálózati kapcsolat miatt a nagyobb sávszélességet és egységes késés szintek előnyben részesített típusát, amikor a Microsoft gerinchálózatán keresztül áthaladó.

Azt javasoljuk, hogy ellenőrizze a késés és sávszélesség az ezeket a kapcsolatokat, és eldönteni, hogy szinkron vagy aszinkron adatreplikációt megfelelő ügyfelek hálózati minősítési vizsgálat futtatása eredményei alapján. Fontos továbbá rálátással ezeket az eredményeket az optimális helyreállítási időre vonatkozó célkitűzés (RTO) fényében.

#### <a name="disaster-recovery-diverting-traffic-from-one-region-to-another"></a>Vész-helyreállítási: egy régióból a másikba forgalom átirányítása

[Az Azure Traffic Manager] [ TM] rendszeres időközönként a nyilvános végpontokat különböző VDC-implementációkban szolgáltatás állapotát ellenőrzi és, ha ezekre a végpontokra meghibásodik, azt továbbítja az automatikusan a másodlagos VDC a tartománynév használatával Tartománynévrendszer (DNS). 

Mert DNS használ, a Traffic Manager van, csak az Azure nyilvános végpontokra való használatra.  A szolgáltatás általában segítségével szabályozhatja, vagy átirányít a forgalom Azure virtuális gépek és a Web Apps-példány a VDC-megvalósítás kifogástalan állapotú. A TRAFFIC Manager képes legyen ellenállni egy teljes Azure-régiót meghibásodása esetén is, és szabályozhatja a különböző kritériumok alapján különböző Ügyfélszempontokat szolgáltatásvégpontokra érkező felhasználói forgalom elosztása. Ha például egy adott VDC megvalósítása, illetve a VDC megvalósítása a legalacsonyabb hálózati késéssel rendelkező szolgáltatásának sikertelen.

### <a name="summary"></a>Összegzés

A virtuális adatközpont adatközpont áttelepítési létrehozása egy méretezhető architektúra az Azure-ban, amely maximalizálja a felhőalapú erőforrások használatával, csökkenti a költségeket, és egyszerűbbé teszi a rendszer cégirányítási megközelítés. A VDC alapul egy küllős hálózati topológiában, közös megosztott szolgáltatások az agyi, és lehetővé teszi az adott alkalmazások és számítási feladatok a küllők az. A VDC is megfelel a struktúra vállalati szerepkörök, ahol különböző részlegek központi informatikai, DevOps, üzemeltetése és karbantartása, például az összes együttműködve adott szerepkörökhöz végrehajtása közben. A VDC-megfelel a "Lift and Shift" áttelepítés, de a natív felhőalapú rendszerbe sok előnyt is biztosít.

## <a name="references"></a>Referencia

Ez a dokumentum a következő funkciókat is tárgyalja. A hivatkozásokat követve tudhat meg többet.

| | | |
|-|-|-|
|Hálózati szolgáltatások|Terheléselosztás|Kapcsolatok|
|[Azure virtuális hálózatok][VNet]</br>[Hálózati biztonsági csoportok][NSG]</br>[NSG-naplók][NSGLog]</br>[Felhasználó által megadott útvonal][UDR]</br>[Hálózati virtuális berendezések][NVA]</br>[Nyilvános IP-címek][PIP]</br>[Az Azure DDOS][DDOS]</br>[Az Azure-tűzfal][AzFW]</br>[Az Azure DNS][DNS]|[Az Azure bejárati ajtajának][AFD]</br>[Az Azure Load Balancer (3.) ][ALB]</br>[Az Alkalmazásátjáró (7. rétegbeli) ][AppGW]</br>[Webalkalmazási tűzfal][WAF]</br>[Az Azure Traffic Manager][TM]</br></br></br></br></br> |[Virtuális hálózatok közötti Társviszony][VNetPeering]</br>[Virtuális magánhálózat][VPN]</br>[Virtuális WAN][vWAN]</br>[ExpressRoute][ExR]</br>[Az ExpressRoute közvetlen][ExRD]</br></br></br></br></br>
|Identitás</br>|Figyelés</br>|Ajánlott eljárások</br>|
|[Azure Active Directory][AAD]</br>[A multi-factor Authentication szolgáltatás][MFA]</br>[Szerepkör alap hozzáférés-vezérlés][RBAC]</br>[Alapértelmezett Azure AD-szerepkörök][Roles]</br></br></br> |[A Network Watcher][NetWatch]</br>[Az Azure Monitor][Monitor]</br>[Tevékenységnaplók][ActLog]</br>[Diagnosztikai naplók][DiagLog]</br>[A Microsoft Operations Management Suite][OMS]</br>[A Network Performance Monitor][NPM]|[Szegélyhálózat-alapú hálózatok, ajánlott eljárások][DMZ]</br>[Előfizetések kezelése][SubMgmt]</br>[Erőforrás-csoportok kezelése][RGMgmt]</br>[Azure-előfizetés korlátai][Limits] </br></br></br>|
|Más Azure-szolgáltatások|
|[Az Azure Web Apps alkalmazások][WebApps]</br>[Hdinsight (Hadoop) ][HDI]</br>[Event Hubs][EventHubs]</br>[Szolgáltatásbusz][ServiceBus]|

## <a name="next-steps"></a>További lépések

 - Ismerkedés a [virtuális hálózatok közötti társviszony-létesítés][VNetPeering], a megerősítő technológiát a VDC küllős tervek
 - Alkalmazzon [Azure ad-ben] [ AAD] használatába [RBAC] [ RBAC] feltárása
 - Fejlesztés az előfizetésben és erőforráscsoportban felügyeleti modell és a struktúra, a követelményeknek és a szabályzatok a szervezet RBAC-modellben. A legfontosabb tevékenység tervezi. Gyakorlati, sokkal átszervezések, fúzió, új termékcsaládok, stb. m tervezése <!--Image References-->
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