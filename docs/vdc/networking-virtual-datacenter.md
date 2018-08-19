---
title: Az Azure Virtual Datacenter – A hálózati nézőpont
description: Ismerje meg, hogyan hozhat létre az Azure-beli virtuális adatközpont
author: tracsman
manager: rossort
tags: azure-resource-manager
ms.service: virtual-network
ms.date: 04/3/2018
ms.author: jonor
ms.openlocfilehash: 8df02bbb6d926f0ab5402fab4eb2b25fdc2057cd
ms.sourcegitcommit: 8ec48a0e2c080c9e2e0abbfdbc463622b28de2f2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 08/18/2018
ms.locfileid: "40211478"
---
# <a name="azure-virtual-datacenter-a-network-perspective"></a>Az Azure Virtual Datacenter: A hálózati nézőpont

## <a name="overview"></a>Áttekintés
A helyszíni alkalmazások az Azure-ba való migrálás, (egy, a "lift and shift" néven ismert megközelítés), jelentős módosítások nélkül is értékelemeket biztosít a szervezetek a biztonságos és költséghatékony infrastruktúra. Azonban ahhoz, hogy a legtöbbet a rugalmasságot lehetséges a felhő-számítástechnika, vállalatok kell alakítani az architektúrák Azure-képességek kihasználásához. A Microsoft Azure biztosít, rendkívül nagy kapacitású szolgáltatások és infrastruktúra, vállalati szintű képességet és megbízhatóságát és számos választási való hibrid kapcsolathoz. Ügyfeleink kiválaszthatják hozzá vagy az interneten keresztül, vagy az Azure expressroute használatával privát hálózati kapcsolatot biztosít a felhőalapú szolgáltatásokhoz. A Microsoft Azure platform lehetővé teszi, hogy az ügyfelek zökkenőmentesen infrastruktúrán kiterjesztheti a felhőbe, és több rétegből álló architektúrákat hozhat létre. Ezenkívül a Microsoft partnerei fejlett funkciókat biztosítanak azzal biztonsági szolgáltatások és virtuális berendezések az Azure-beli futtatására van optimalizálva.

Ez a cikk áttekintést nyújt minták és a terveket, amelyeket az architekturális méretezés, teljesítmény és biztonsági problémák megoldásához használható számos ügyfelek face fájlszerkesztés járatára masse áthelyezése a felhőbe. Bemutatja, hogyan illeszkednek a különböző szervezeti informatikai szerepköröket, a felügyelet alá áttekintése és a rendszer a cégirányítási is tárgyalt, kiemeléssel biztonsági követelmények és költségek optimalizálása.

## <a name="what-is-a-virtual-data-center"></a>Mi az a virtuális adatközpont?
A korai napban felhőmegoldások tervezték, hogy a gazdagép egyetlen, viszonylag elkülönített, alkalmazások, a nyilvános spektrum. Ez a módszer jól években dolgozott. Azonban, a felhő nyújtotta előnyök megoldások nyilvánvalóvá vált, és több nagy méretű számítási feladatokat is címzés, a biztonság, megbízhatóság, teljesítmény, a felhőben üzemeltetett és a egy központi telepítések vonatkozik költség, vagy további régiókban vált létfontosságú teljes életciklusa a felhőszolgáltatás.

Az alábbi ábrán a felhőbeli üzembe helyezés néhány példa a biztonsági rések (piros box) és optimalizálás Hálózati virtuális berendezések szoba számítási feladatokhoz (sárga keretet) mutatja be.

[![0]][0]

A virtuális központ (VDC) született, a méretezés a vállalati számítási feladatokat és a problémák vezetett be, amikor nagy léptékű alkalmazások támogatása a nyilvános felhőben kezelésére szükség szükségességét.

A VDC nem csak az alkalmazás számítási feladatokat a felhőben, de is a hálózati, biztonsági, felügyeleti és infrastruktúra (például a DNS és a Directory Services). Általában is biztosít egy privát kapcsolaton keresztül érik el egy helyszíni hálózat vagy data center. Ahogy egyre több számítási feladatok Azure-bA áthelyezni, fontos gondolja át a támogató infrastruktúra és az objektumok, amelyek az ilyen munkaterhelések vannak elhelyezve. Szem előtt tartva gondosan erőforrások struktúrája hogyan elkerüléséhez elterjedését, több száz "munkaterhelés-szigetek", amely független az adatfolyam, modellek és megfelelőségi kihívások külön-külön kell kezelni.

Egy virtuális adatközpont lényegében egy különálló, de a kapcsolódó entitások a gyakori támogatási funkciók, szolgáltatások és infrastruktúra gyűjteménye. A számítási feladatok megtekintésével, egy integrált VDC, alacsonyabb költségek miatt méretgazdaságosság, optimalizált biztonság összetevő- és a folyamat forrásadattárakból, könnyebben operations, felügyeleti és megfelelőségi ellenőrzésekhez vegye figyelembe.

> [!NOTE]
> Fontos tisztában lenni azzal, hogy a VDC **nem** egy különálló Azure-termék, de a különböző funkciók és képességek a pontos követelményeinek kombinációja. vDC módja a szem előtt tartva a számítási feladatok és az Azure-használat, maximalizálhatja az erőforrások és a képességek a felhőben. A virtuális tartományvezérlő éppen ezért a moduláris megközelítés informatikai szolgáltatásokat az Azure-beli szervezeti szerepkörök és felelősségek tiszteletben készítéséhez.

A VDC segítségével a vállalatok munkaterheléseket és alkalmazásokat az Azure-bA lekérése a következő forgatókönyvekhez:

-   Több kapcsolódó számítási üzemeltetése
-   Az Azure-bA a helyszíni környezet migrálása számítási feladatokhoz
-   Megosztott vagy központi biztonsági és a hozzáférési követelmények megvalósítása a számítási feladatok
-   A DevOps és a központi informatikai nagyvállalatok számára megfelelő keverése

A kulcs a VDC, előnyeit feloldásához egy központi topológia (küllős topológia) az Azure-szolgáltatások kombinációját: [Azure virtuális hálózat][VNet], [NSG-k] [ NSG], [Virtuális hálózatok közötti társviszony-létesítés][VNetPeering], [felhasználó által megadott útvonalak (UDR)][UDR], és az Azure-identitás [szerep alapja Hozzáférés-vezérlés (RBAC)][RBAC].

## <a name="who-needs-a-virtual-data-center"></a>Egy virtuális adatközpont kell?
Számítási feladatok Azure-bA néhány nagyobb bármely Azure-ügyfél, amelyeket át kell szem előtt tartva közös erőforrások használatával is kihasználhatják. Mértékétől függően akár egyetlen alkalmazások milyen előnyei származhatnak a minták és a egy VDC összeállításához összetevőket.

Ha a szervezet rendelkezik egy központi informatikai, hálózati, biztonsági és/vagy megfelelőségi csapat vagy részleg, a VDC segíthet kényszeríteni házirend pontok, a vámot, elkülönítése és az alapul szolgáló közös összetevők egységességének biztosítása miközben a alkalmazásfejlesztő csapat akár szabadon és vezérlőt ugyanúgy, mint a követelményeinek megfelelő.

Szervezeteknek, amelyek a fejlesztési és üzemeltetési szeretne kipróbálhatják az Azure-erőforrások jogosult zsebe biztosítanak, és hozzáférhetnek a csoport (vagy az előfizetést, vagy az erőforrást egy közös előfizetésben), de a hálózaton belül teljes körű vezérlést VDC fogalmakat és biztonsági határokat tartózkodási megfelelő, egy agyi virtuális hálózat és az erőforráscsoport egy központosított szabályzatban meghatározottak szerint.

## <a name="considerations-on-implementing-a-virtual-data-center"></a>Egy virtuális adatközpont végrehajtási szempontok
A VDC tervezésekor több pivotal problémák vannak kell figyelembe venni:

-   Identitás- és címtárszolgáltatásokat
-   Biztonsági infrastruktúra
-   A felhővel való kapcsolat
-   Kapcsolat a felhőben

### <a name="identity-and-directory-service"></a>Identitás- és címtárszolgáltatás
Identitás- és címtárszolgáltatásokat kulcsfontosságú az összes adat erőforrásokból, mind a helyszíni és a felhőben. Hozzáférés és -engedélyezést, a VDC szolgáltatásai minden aspektusát identitás kapcsolódik. Annak biztosítására, hogy csak a jogosult felhasználók és a folyamatok hozzá az Azure-fiók és -erőforrások, Azure, számos különböző típusú hitelesítő adatokat használ. Ezek közé tartoznak a jelszavak (az Azure-fiók eléréséhez), a kriptográfiai kulcsokat, a digitális aláírások és a tanúsítványok. [*Az Azure multi-factor Authentication* (MFA)] [ MFA] van egy további Azure-szolgáltatások eléréséhez szükséges biztonsági réteget. Az Azure MFA több egyszerű ellenőrzési lehetőség erős hitelesítést tesz lehetővé – telefonhívás, SMS vagy mobilalkalmazásbeli értesítés –, és lehetővé teszi ügyfeleink számára válassza a tetszőlegesen választhatnak.

Minden olyan nagyvállalati kell egy leíró egyedi azonosítók kezelését, a hitelesítés, engedélyezés, szerepkörök és jogosultságok belül vagy között a VDC identity management folyamat megadása. Ez a folyamat célja a biztonság és hatékonyság közben csökkentésével költségeket, leállás és ismétlődő manuális feladatokat kell lennie.

Vállalati, illetve céget szükség, hogy a szolgáltatások különböző sor-az-üzleti (LOB) erőforrás-igényű vegyesen, és alkalmazottak gyakran különböző szerepkörök esetén, a különböző projektekkel rendelkezik. A VDC megfelelő együttműködését a különböző csapatok különböző, az adott szerepkör-definíciók, a rendszerekhez és a megfelelő cégirányítási fut minden egyes szükséges. A mátrix feladatkörei, hozzáférés és a jogok rendkívül összetettek lehetnek. Identitáskezelés a VDC keresztül valósul meg [ *Azure Active Directory* (AAD)] [ AAD] és a szerepköralapú hozzáférés-vezérlés (RBAC).

Címtárszolgáltatás a megosztott adatok keresése, kezelése, felügyelete és mindennapi elemek és hálózati erőforrások rendszerezése infrastruktúráját. Ezeket az erőforrásokat a köteteket, mappák, fájlok, nyomtatók, felhasználók, csoportok, eszközök és más objektumok is tartalmazhat. A hálózaton lévő minden egyes erőforrás számít egy objektumot a directory-kiszolgáló által. Erőforrásra vonatkozó adatokat egy adott erőforrás vagy objektumhoz tartozó attribútumok gyűjteménye van tárolva.

Az összes Microsoft online szolgáltatás az Azure Active Directory (AAD) használt a bejelentkezéshez, és egyéb az identitásukat. Az Azure Active Directory egy átfogó, magas rendelkezésre állású, felhőalapú identitás- és hozzáférés-kezelő megoldás, amely ötvözi az alapvető címtárszolgáltatásokat, a fejlett identitáskezelést és az alkalmazáshozzáférés-felügyeletet. AAD integrálhatók a helyszíni Active Directory egyszeri bejelentkezés az összes felhőalapú és a helyileg üzemeltetett (helyszíni) engedélyezése az alkalmazások. A helyszíni Active Directory felhasználói attribútumok automatikusan szinkronizálhatók az aad-be.

Egyetlen globális rendszergazdája nem szükséges a VDC összes engedélyeket. Minden egyes adott részleg (vagy a felhasználók és a címtárszolgáltatás szolgáltatáscsoport) inkább rendelkezhet az a saját belül a VDC-erőforrások kezeléséhez szükséges engedélyeket. Engedélyek strukturálja terheléselosztást igényel. Túl sok engedély akadályozhatják a teljesítmény hatékonyságát, és túl kevés vagy laza engedélyek növelhető a biztonsági kockázatokat. Az Azure szerepköralapú hozzáférés-vezérlés (RBAC) részletes hozzáférés-vezérlést a VDC-erőforrások felajánlásával oldja meg a problémát, segítséget nyújt.

#### <a name="security-infrastructure"></a>Biztonsági infrastruktúra
Biztonsági infrastruktúra, a VDC kontextusában főként kapcsolódik a VDC adott virtuális hálózati szegmens a forgalom elkülönítése, és hogyan szabályozható a bejövő és kimenő folyamatok során a VDC. Az Azure több-bérlős architektúra, amely megakadályozza a jogosulatlan vagy véletlen forgalom közötti alapul, használja a virtuális hálózat (VNet) elkülönítés, hozzáférés-vezérlési listák (ACL), terheléselosztók és IP-szűrőket, valamint olyan forgalmi szabályzatok. Hálózati címfordítás (NAT) elkülöníti a belső hálózati forgalmat a külső forgalomtól.

Az Azure-hálót foglalja le az infrastruktúra-erőforrások bérlői számítási feladatok, és kezeli a kommunikációt egy telepítésen, és a virtuális gépek (VM). Az Azure hipervizor előírja a memóriák és a folyamatok elkülönítését virtuális gépek között, és biztonságosan a útvonalakat hálózati forgalmat a vendég operációs rendszerek bérlőihez.

#### <a name="connectivity-to-the-cloud"></a>A felhővel való kapcsolat
A VDC kapcsolódnia kell az ügyfelek, partnerek és/vagy a belső felhasználók szolgáltatásajánlatokhoz külső hálózatokat. Ez általában azt jelenti, hogy a kapcsolat nem csak az internethez, hanem a helyszíni hálózatok és adatközpontok.

Ügyfelek hozhat létre biztonsági házirendeket, melyekkel mi, és hogyan adott VDC üzemeltetett szolgáltatások-e, vagy az internetről használata a hálózati virtuális berendezések (Szűrés és a forgalom ellenőrzési), és egyéni útválasztási házirendek és a hálózati szűrés) Felhasználó által meghatározott útválasztást és a hálózati biztonsági csoportok).

Vállalatok gyakran kell Ügyfélszempontokat a helyszíni adatközpontok és más erőforrásokhoz kapcsolódni. Az Azure és helyszíni hálózat közötti kapcsolat ezért rendkívül fontos szempont egy hatékony architektúra tervezése során. Vállalatok egy szoftverhálózat közötti VDC és a helyszíni kapcsolat létrehozása az Azure-ban két különböző módja van: az interneten keresztül, illetve közvetlen privát kapcsolatok átvitel során.

Egy [ **az Azure Site-to-Site VPN** ] [ VPN] az összekapcsolási szolgáltatása a helyszíni hálózat között az interneten keresztül, és a VDC biztonságos keresztül létrehozott titkosítva kapcsolatok (IPsec/IKE-alagutak). Az Azure Site-to-Site kapcsolat, rugalmas, gyors létrehozásához, és nincs szükség semmilyen további beszerzésekre, minden kapcsolat csatlakozzon az interneten keresztül.

[**Az ExpressRoute** ] [ ExR] egy kapcsolatot az Azure-szolgáltatás, amely a VDC és a helyszíni hálózat között magánkapcsolatok létrehozását teszi lehetővé. Az ExpressRoute-kapcsolatok nem a nyilvános interneten keresztül, és nagyobb biztonságot, megbízhatóságot és egységes késést és nagyobb sebesség (legfeljebb 10 GB/s). Az ExpressRoute nagyon hasznos az Ügyfélszempontokat, mint a megfelelőségi szabályok magánkapcsolatok társított előnyeit magonkénti ExpressRoute.

Az ExpressRoute-kapcsolatok telepítése magában foglalja a révén egy ExpressRoute-szolgáltató. A gyorsan elkezdheti a munkát igénylő ügyfelek azt közös először használja a Site-to-Site VPN a VDC közötti kapcsolatot létesíteni a helyszíni erőforrások és majd telepítse át az ExpressRoute-kapcsolat.

#### <a name="connectivity-within-the-cloud"></a>*Kapcsolat a felhőben*
[Virtuális hálózatok] [ VNet] és [virtuális hálózatok közötti társviszony-létesítés] [ VNetPeering] belül a VDC alapvető hálózati kapcsolódási szolgáltatások. Virtuális hálózat garantálja a VDC-erőforrások elkülönítési határ természetes, és a virtuális hálózatok közötti társviszony lehetővé teszi, hogy a másik virtuális hálózat azonos Azure-régióban vagy régiók között is között befolyásolása. Forgalomvezérlés egy virtuális hálózaton belül és virtuális hálózatok között meg kell egyeznie a biztonsági szabályok megadva a hozzáférés-vezérlési listák ([hálózati biztonsági csoport][NSG]), [hálózati virtuális berendezések ] [ NVA], és egyéni útválasztási táblázatokat ([UDR][UDR]).

## <a name="virtual-data-center-overview"></a>Virtuális adatközpont – áttekintés

### <a name="topology"></a>Topológia
_Küllős topológia_ egy olyan modell, egy adott Azure-régión belül egy virtuális adatközpont bővítése.

[![1]][1]

A hub a központi zóna, amely szabályozza, és megvizsgálja a bejövő és/vagy kimenő forgalom különböző zónák között: az Internet, a helyszíni és a küllők. A küllős topológia a hatékony módszert egy központi helyen, a biztonsági szabályzatok kényszerítése a lehetséges helytelen konfiguráció és a kitettség csökkentése mellett az informatikai részleg biztosítja.

A hub tartalmazza az általános szolgáltatás-összetevők a küllők által igénybe vett. Az alábbiakban néhány tipikus példák közös központi szolgáltatások:

-   A Windows Active Directory-infrastruktúrát (a kapcsolódó AD FS szolgáltatással) való elérése nem megbízható hálózatokhoz, mielőtt hozzáférhetne a számítási feladatok a küllőkben a harmadik felek felhasználói hitelesítés szükséges
-   A DNS-szolgáltatás elnevezési a küllők hozzáférési erőforrások a helyszíni és az interneten a számítási feladat megoldásához
-   A PKI-infrastruktúra, a számítási feladatok egyszeri bejelentkezés megvalósítása
-   Átvitelvezérlés (TCP/UDP), a küllő és az Internet között
-   A küllő és a helyszíni közötti adatfolyam vezérlés
-   Ha szükséges, egy küllő és a egy másik között folyamatvezérlés

A VDC csökkenti a teljes költség több küllők között megosztott hub infrastruktúra használatával.

A szerepkör minden egyes küllőhöz kapcsolni, a számítási feladatok különböző típusú gazdagép is lehet. A küllők is megadhat a moduláris megközelítés megismételhető üzembe helyezések (például fejlesztési és tesztelési, a felhasználói tesztelés az üzem előtti és éles környezetek) az azonos számítási feladatok. A küllők őket, és lehetővé teszi a különböző csoportokba a szervezetben (például a DevOps-csoportok) használatával is lehet. Belül egy adott küllőre egy alapvető számítási feladatok vagy a rétegek közötti forgalom ellenőrzésével összetett, többrétegű munkaterhelések üzembe helyezéséhez.

#### <a name="subscription-limits-and-multiple-hubs"></a>Előfizetés korlátai, és több hubon
Az Azure-ban minden összetevő, a bármilyen típusú helyezünk üzembe egy Azure-előfizetésben. Az elkülönítés az Azure-előfizetések az Azure összetevőit objekty lobs s Hodnotou különböző, például a hozzáférési és engedélyezési szinteket beállítása követelményeinek is megfelelnek.

Egyetlen VDC is vertikálisan nagy számú küllők, bár minden informatikai rendszer-platformok korlátozva van. A központi telepítés kötve van egy adott Azure-előfizetéssel, korlátozások és korlátokat rendelkező (például tekintse meg a maximális számú virtuális hálózatok közötti társviszony - [Azure-előfizetés és a szolgáltatások korlátozásai, kvótái és megkötései] [ Limits] részletekért). Azokban az esetekben, ahol a korlátok lehet probléma, az architektúra méretezhetők akár tovább csökkenthetők a modell egyetlen Központ-küllő küllős topológia a fürtre kiterjedő. Egy vagy több Azure-régióban több hubon összekapcsolható, virtuális hálózatok közötti társviszony-létesítés, az expressroute-on vagy a helyek közötti VPN használatával.

[![2]][2]

Több bevezetésével növeli a költségeket és a felügyeleti terhek, a rendszer, és csak akkor igazolhatónak méretezhetőség (példák: system-korlátok, és a redundancia) és a regionális replikáció (példák: végfelhasználói teljesítmény vagy katasztrófa utáni helyreállítás). Forgatókönyvek a több hubon, az összes hubs igénylő törekedni kell a ugyanazokat a működési egyszerű szolgáltatásokat kínálnak.

#### <a name="interconnection-between-spokes"></a>Összekapcsolás küllők között
Egy egyetlen küllő belül összetett több rétegek számítási feladatok végrehajtására. Többrétegű konfigurációk használatával (egy minden szinthez) alhálózatok ugyanabban a vnetben, és NSG-k használata a flow szűrés valósítható meg.

Másrészről olyan érdemes egy többrétegű számítási feladatok üzembe több virtuális. Virtuális hálózatok közötti társviszony használja, küllők csatlakozhatnak az ugyanazon hub-vagy különböző hubs más küllők. Egy jellemző példa rá a helyzet ahol alkalmazás feldolgozó kiszolgálók vannak egy küllős (VNet), miközben az adatbázis egy másik küllő (VNet) van üzembe helyezve. Ebben az esetben is könnyen összekapcsolható a küllő virtuális hálózatok közötti társviszony-, és ezáltal elkerülheti a hubon áthaladó. Annak érdekében, hogy a hub megkerülése nem megkerülése, fontos biztonsági vagy naplózási pontokat, amelyeket lehet, hogy csak a hub szerepel egy gondos architektúra és biztonsági felülvizsgálat kell végrehajtani.

[![3]][3]

Küllők is összekapcsolható, egy adott küllőre, amely a kiszolgálókezelési. Ezzel a módszerrel hoz létre egy kétszintű hierarchia: a küllőkben a magasabb szinten (0. szint) a hierarchia alacsonyabb agyként (1. szint) a hub válnak. A küllők a VDC kell továbbítani a forgalmat a helyszíni hálózat és internet vagy bizalommal a központi agyhoz. Hub két szinttel rendelkező vezet be, hogy bonyolult útválasztással, amely eltávolítja egy egyszerű küllős kapcsolat előnyeit.

Bár az Azure lehetővé teszi, hogy a komplex topológiáit, a VDC-fogalom alapelvek egyik ismételhetőség és az egyszerűség kedvéért. Energiabefektetést igénylő felügyelet minimalizálása érdekében a küllős egyszerű kialakítás ajánlott a VDC-referenciaarchitektúra.

### <a name="components"></a>Összetevők
Egy virtuális adatközpont épül fel négy alapvető összetevő típusa: **infrastruktúra**, **peremhálózatokon**, **számítási feladatok**, és **figyelés**.

Minden egyes összetevő típusát különböző Azure-szolgáltatások és erőforrások áll. A VDC-példányok többféle összetevőket, és több változata létezik azonos típusú összetevő épül fel. Előfordulhat például, különböző alkalmazások képviselő számos különböző, logikailag elkülönített számítási példányokat. Végső soron hozhat létre a VDC használhatja ezeket a különböző összetevők típusok és példányok.

[![4]][4]

Az előző architektúrájának áttekintése a VDC-a Központ-küllő topológia különböző zónák használható különböző összetevő típusok jeleníti meg. Az infrastruktúra-összetevőket látható az architektúra különféle részein.

Bevált gyakorlat szerint (egy a helyszíni tartományvezérlő vagy a VDC) hozzáférési jogokat és jogosultságokat kell lennie, Csoportalapú. Csoportok kezelése, egyéni felhasználók helyett segít a hozzáférési szabályzatok következetesen fenntartása a különböző csapatok és segédeszközök konfigurációs hibák minimalizálása. Hozzárendelése és eltávolítása a felhasználók és a megfelelő csoportok segítségével egy adott felhasználó a jogosultságok rendszeres frissítése.

Minden egyes szerepkör-csoport a nevüket, így könnyen azonosíthatja, hogy melyik csoporthoz társítva mely számítási feladatok egy egyedi előtagot kell rendelkeznie. Például egy hitelesítési szolgáltatást üzemeltető számítási feladatok lehet nevű csoport *AuthServiceNetOps, AuthServiceSecOps, AuthServiceDevOps és AuthServiceInfraOps.* Hasonlóképpen a szerepkör vagy szerepkörök nem egy adott szolgáltatáshoz kapcsolódó központi, a "Corp", az tartalmazni sikerült *CorpNetOps* például.

Számos szervezet egy változata, a következő csoportok használatával adja meg a főbb információkat a szerepkörök:

-   A *központi informatikai csoport (vállalat)* rendelkezik tulajdonosi jogosultságokkal a vezérlőelemet (például a hálózati és biztonsági) infrastruktúra-összetevőket, és ezért kell az előfizetés közreműködői szerepköre (és felügyelheti a hub) és a hálózati közreműködő jogok a küllők. Nagyméretű szervezet gyakran felosztani ezeket a felügyeleti feladatköröket például; több csapatok között a hálózati műveletek (CorpNetOps) csoport (kizárólagos összpontosít a hálózatkezelés) és a egy biztonsági műveletek (CorpSecOps) csoport (tűzfal- és biztonsági házirend felelős). Ebben az esetben két különböző csoportot kell létrehozni ezeket egyéni szerepkör-hozzárendelés.
-   A *fejlesztési & tesztcsoportot (AppDevOps)* felelősség (alkalmazások vagy szolgáltatások) számítási feladatok üzembe helyezéséhez. Ez a csoport vesz igénybe, a szolgáltatott infrastruktúra telepített példányainak, illetve egy virtuális gép közreműködő szerepkör vagy a további PaaS-közreműködői szerepkör (lásd: [Azure szerepköralapú hozzáférés-vezérlés beépített szerepkörei][Roles]). A fejlesztési és tesztelési csapat igény szerint szükség lehet a biztonsági házirendek (NSG-k) és a hubhoz vagy egy adott küllő útválasztási házirendek (UDR) láthatók. Ezért mellett közreműködői számítási feladatokhoz a szerepköröket, ez a csoport módosítania kell a hálózati olvasó szerepkört.
-   A *működéséért és karbantartásáért csoport (CorpInfraOps vagy AppInfraOps)* feladata a termelési számítási feladatok kezelésére. Ennek a csoportnak kell lennie a számítási feladatokat éles üzemű előfizetéseket az előfizetés közreműködői. Egyes szervezetek is előfordulhat, hogy kiértékelése, ha szükségük van egy további eszkalációs támogatási csapat szerepkörrel rendelkező előfizetés közreműködője éles környezetben, és a központi agyhoz az előfizetésben, annak érdekében, hogy javítsa ki a potenciális konfigurációs problémákat az éles környezetben környezet.

A VDC van felépítve, hogy a hub kezelése a központi informatikai csoportok számára létrehozott csoportok megfelelő csoportok számítási feladatok szintjén. Hub-erőforrások kezelése mellett a központi informatikai csoportok lenne külső hozzáférés és a legfelső szintű engedélyeket az előfizetésben. Munkaterhelés-csoport azonban erőforrásokat és a virtuális hálózat, egymástól függetlenül a központi informatikai engedélyek lenne.

A VDC biztonságos futtatására több projektet különböző sor-az-üzleti (LOB) keresztül kell particionálni kell. Minden projekthez szükséges különböző izolált környezetben (fejlesztői, foglalja annak, éles környezetben). Ezekben a környezetekben mindegyike külön Azure-előfizetések elkülönítéshez természetes.

[![5]][5]

A fentebbi ábra bemutatja azokat a szervezet projekteket, felhasználók, csoportok és, az Azure-összetevők telepítve vannak a környezetek közötti kapcsolat.

Általában az informatikai környezetben (vagy szint) egy rendszer, amelyben több alkalmazást üzembe és végrehajtva. A nagyobb vállalatok használata a fejlesztési környezet (amelyben módosításait eredetileg és tesztelt) és a egy éles környezetben (mi végfelhasználók használja). Szkriptjét el vannak különítve, gyakran több átmeneti környezetek között a többszakaszos üzembe helyezés (Bevezetés) engedélyezését, tesztelés és a visszaállítás problémák esetén. Üzembe helyezési architektúra jelentős eltérések lehetnek, de általában fejlesztési (fejlesztői), és az éles (ÉLES) egészen az alapvető folyamat továbbra is követi.

Az ilyen típusú több rétegből álló környezeteket egy gyakori architektúráját DevOps (fejlesztés és tesztelés) foglalja annak (átmeneti) és éles környezetek áll. Szervezetek használhatnak egyetlen vagy több Azure AD-bérlőt, hozzáférés és ezekben a környezetekben jogosultságok meghatározására. Az előző ábrán látható egy esetet, ha két különböző Azure AD-bérlőt használ: egyet a fejlesztési és üzemeltetési és foglalja annak és a egy másik kizárólag az éles környezetben.

Jelen másik Azure AD-bérlők kikényszeríti a környezetek elkülönítése. Az ugyanazon felhasználói csoporthoz (például egy központi informatikai) kell elvégezniük egy másik URI-t egy másik AD-bérlőhöz való hozzáféréshez a szerepkörök vagy engedélyek a fejlesztési és üzemeltetési vagy az éles környezetek a projekt módosítása. A különböző környezetek eléréséhez különböző felhasználói hitelesítés jelenlétét csökkenti a szolgáltatáskimaradásokat és egyéb az emberi hibák által okozott problémákat.

#### <a name="component-type-infrastructure"></a>Összetevő típusa: infrastruktúra
Ezen összetevő típusa, ahol a legtöbb támogató infrastruktúra található. Is, a központi informatikai, biztonság, megfelelőség részlegeknek legtöbb idejüket, illetve.

[![6]][6]

Infrastruktúra-összetevőket adjon meg egy olyan VDC összetevői közötti kapcsolatot biztosítja, és mind a küllős topológia az szerepelnek. A felelősség kezelésével és karbantartásával az infrastruktúra-összetevőket általában hozzá van rendelve a központi informatikai és/vagy a biztonsági csapatnak.

Az informatikai infrastruktúra csapat az elsődleges feladatok egyikét, az IP-cím sémák konzisztenciájának biztosítása a vállalaton belül. A magánhálózati IP-címtartomány a VDC kell lennie rendelve, és nincs átfedésben a helyszíni hálózatokon rendelt magánhálózati IP-címek.

Bár a helyszíni peremhálózati útválasztói vagy az Azure-környezetek NAT IP-címütközés elkerülheti, komplikációk hozzáadja az infrastruktúra-összetevőket. Egyszerű kezelés a az egyik fő célja a VDC, így használ a NAT IP-problémák kezeléséhez nem javasolt megoldás.

Infrastruktúra-összetevőket a következő funkciókat tartalmazza:

-   [**Identitás- és címtárszolgáltatásokat**][AAD]. Az Azure-ban minden erőforrástípus a hozzáférést az identitás a címtárszolgáltatások tárolt vezérli. A címtárszolgáltatás nem csak a felhasználók listáját, de az erőforrásokhoz való hozzáférési jogosultságokat is tárol egy adott Azure-előfizetést. Ezek a szolgáltatások is létezik, csak felhőalapú, vagy is szinkronizálható a helyszíni Active Directoryban tárolt identitással.
-   [**Virtuális hálózat**][VPN]. Virtuális hálózatok egyik fő összetevője a VDC, és lehetővé teszi a forgalomelkülönítési határok létrehozása az Azure platformon. Virtuális hálózat egy vagy több virtuális hálózati szegmenseket, amelyek mindegyike egy adott IP-hálózati előtagot (alhálózat) áll. A virtuális hálózat egy belső peremhálózati területre, ahol IaaS virtuális gépek és PaaS-szolgáltatások létesíthet a privát kommunikációs határozza meg. Virtuális gépek (és a PaaS-szolgáltatások) egy virtuális hálózat közvetlenül a virtuális gépeket nem lehet kommunikálni (és PaaS-szolgáltatások) egy másik virtuális hálózatban, akkor is, ha mindkét virtuális hálózat hozzák létre az adott ügyfél azonos előfizetéshez tartozó. Elkülönítés kritikus tulajdonság, amely biztosítja az ügyfél virtuális gépei, kommunikációs privát virtuális hálózaton belül marad.
-   [**UDR**][UDR]. A virtuális hálózati adatforgalmat a rendszer útválasztási táblázat alapján alapértelmezés szerint. Egy felhasználó definiálása út felülírja a rendszer útválasztási táblázat viselkedését, és a egy virtuális hálózaton belüli kommunikációs útvonal definiálhat egy vagy több alhálózatot hálózati rendszergazdák egyéni útvonaltábla társítható. Udr-EK jelenléte garantálja, hogy a kimenő forgalom származó adott egyéni virtuális gépeket és/vagy a hálózati virtuális berendezések és a jelen, a hubot és a küllők terheléselosztók küllő átmenő.
-   [**NSG-T**][NSG]. A hálózati biztonsági csoport az szűrést IP-források, IP-cél, protokollok, IP-Forrásportok és cél IP-portok az adatforgalom-kiszolgálóként működő biztonsági szabályok listája. Az NSG-t egy alhálózathoz, az Azure virtuális Gépekhez vagy mindkettőhöz társított virtuális hálózati kártya alkalmazhatók. Az NSG-k nélkülözhetetlenek valósítható meg egy megfelelő adatfolyam vezérlés, a hubot és a küllők az. Az NSG-t által nyújtott biztonság szintje egy függvényt, mely portok megnyitása, és milyen célra. További VM-enkénti szűrők például engedélyezze az IPtables állomásalapú tűzfal vagy a Windows tűzfal kell telepíteniük.
-   [**DNS**][DNS]. A névfeloldás a virtuális hálózatok a VDC-erőforrásainak DNS-en keresztül van megadva. Az Azure DNS szolgáltatást is nyújt [nyilvános][DNS] és [privát] [ PrivateDNS] névfeloldását. Privát zónák névfeloldásához egy virtuális hálózaton belül, és több virtuális hálózaton. Privát zónák csak span ugyanabban a régióban, hanem a régiók és -előfizetések közötti virtuális hálózatok között is. A nyilvános feloldásához az Azure DNS biztosít egy üzemeltetési szolgáltatás DNS-tartományok biztosítani a névfeloldást a Microsoft Azure infrastruktúráját használja. Ha tartományait az Azure-ban üzemelteti, DNS-rekordjait a többi Azure-szolgáltatáshoz is használt hitelesítő adatokkal, API-kkal, eszközökkel és számlázási információkkal kezelheti.
-   [**Előfizetés** ] [ SubMgmt] és [ **erőforráscsoport felügyeleti**][RGMgmt]. Előfizetés az erőforrások több csoport létrehozása az Azure-ban a természetes határok határozza meg. Egy előfizetés erőforrásait együtt vannak összeszerelt nevű erőforráscsoportok logikai tárolók. Az erőforráscsoport olyan logikai csoportot a VDC-erőforrások rendszerezéséhez jelöl.
-   [**RBAC**][RBAC]. RBAC, keresztül térkép szervezeti szerepkörhöz adott Azure-erőforrások, így korlátozhatja a felhasználók csak bizonyos részét műveletek hozzáférési jogokkal együtt. Az RBAC lehet hozzáférést biztosítani a megfelelő szerepkört rendelhet a felhasználók, csoportok és alkalmazások megfelelő hatókörébe. Szerepkör-hozzárendelés hatóköre az Azure-előfizetés, erőforráscsoport vagy egyetlen erőforrás lehet. Az RBAC lehetővé teszi, hogy az engedélyek öröklődése. Egy szülő hatókörben hozzárendelt szerepkör is hozzáférést biztosít az ebben lévő gyermekei. Az RBAC használatával, feladatköröket, és csak olyan mértékű hozzáférést biztosítson a felhasználók számára, amelyek a feladataik elvégzéséhez szükségük van. Például, hogy a virtuális gépek található egy előfizetésben, kezelése, miközben egy másik SQL-adatbázisok kezelheti ugyanazon az előfizetésen belül egy alkalmazott RBAC használatát.
-   [**Virtuális hálózatok közötti társviszony-létesítés**][VNetPeering]. Az alapvető funkció a VDC infrastruktúra létrehozásához használt virtuális hálózatok közötti Társviszony olyan mechanizmus, amely összeköti a két virtuális hálózatok (Vnetek) ugyanabban a régióban az Azure adatközpont-hálózat, vagy az Azure világszerte gerinchálózatát használva régiók között elosztva.

#### <a name="component-type-perimeter-networks"></a>Összetevő típusa: Peremhálózatokon
[Szegélyhálózat] [ DMZ] összetevőket (más néven DMZ-hálózatokban) lehetővé teszi, hogy adja meg a hálózati kapcsolat a helyszíni vagy a fizikai adatközpont-hálózatot, valamint minden olyan kapcsolat, az Internet felé és felől. Emellett akkor is, ahol a hálózati és biztonsági valószínűleg részlegeknek legtöbb idejüket.

Bejövő csomagok kell továbbítva, például az Azonosítók és IP-CÍMEK, a tűzfal az agyban biztonsági készülékek a háttér-kiszolgálók a küllők elérése előtt. A számítási feladatok az internetre irányuló csomagokat is áramlásának keresztül a biztonsági berendezéseket a szegélyhálózaton a házirendek betartatását, vizsgálati és naplózási célokra, mielőtt a hálózaton.

Szegélyhálózat hálózati összetevők adja meg a következő funkciókat:

-   [Virtuális hálózatok][VNet], [UDR][UDR], [NSG-t][NSG]
-   [Hálózati virtuális készüléken][NVA]
-   [Load Balancer][ALB]
-   [Az Application Gateway][AppGW] / [WAF][WAF]
-   [Nyilvános IP-címek][PIP]

Általában a központi informatikai és biztonsági csapatok követelmény definíció- és a szegélyhálózat-alapú hálózatok műveletek felelősséget.

[![7]][7]

A fenti ábrán annak érdekében, két régebben a hozzáférést az internet és a egy helyszíni hálózat egyaránt megtalálhatók a hubhoz a jeleníti meg. Egyetlen hubon, az internethez a peremhálózaton is vertikális felskálázás objekty lobs s Hodnotou, több farmok webalkalmazás-tűzfalak (alsóbb) és/vagy a tűzfalak használatával nagy számú támogatásához.

[**Virtuális hálózatok** ] [ VNet] a hub általában épül, a más típusú szűrési és ellenőrzést végző bejövő és kimenő forgalmat az interneten keresztül az nva-k, az alsóbb és az Azure-szolgáltatások üzemeltetésére több alhálózattal rendelkező virtuális hálózat Application Gateway átjárókkal.

[**UDR** ] [ UDR] UDR használatával ügyfelek telepíthetnek tűzfalak, IDS/IPS, és más virtuális készülékeket és a hálózati forgalom irányítása révén e biztonsági készülékek biztonsági határ házirend betartatása naplózás és vizsgálat. Mind a küllős topológia az garantálja, hogy az adott egyéni virtuális gépeket, a hálózati virtuális berendezések és a terheléselosztók a VDC által használt forgalom továbbítását az udr-EK hozható létre. Garantáljuk, hogy a megfelelő virtuális berendezésre megtalálhatók a küllő átvitel közben a virtuális gépek által generált forgalom, egy udr-t kell a belső terheléselosztó előtérbeli IP-címét állítsa a következő ugrás a küllő alhálózata állítani. A belső terheléselosztó osztja el a belső bejövő forgalmának a virtuális készülékek (load balancer háttérkészlethez).

[![8]][8]

[**Hálózati virtuális berendezések** ] [ NVA] a hub a peremhálózaton, az internet-hozzáféréssel rendelkező normál esetben kezelhető a farm tűzfalak és/vagy webalkalmazás-tűzfalak (alsóbb).

Különböző objekty lobs s Hodnotou leggyakrabban használt számos webes alkalmazásokhoz, és ezek az alkalmazások általában a különböző biztonsági rések és az esetleges biztonsági rések. sorból. Webes alkalmazások tűzfalai termék részletesebben olvashat róluk, mint az általános tűzfal webalkalmazások (HTTP/HTTPS) elleni támadások észleléséhez használt különleges típusú. Képest hagyomány tűzfal-technológiája, alsóbb rendelkeznek kívánt belső webkiszolgálók védelme a fenyegetésekkel szemben.

Egy tűzfal farm csoport dolgozik a biztonsági szabályok a küllők lévő üzemeltetett a munkaterhelések védelme érdekében ugyanazt az általános felügyeleti csoportban területtel tűzfalak, hozzáférés-vezérlése a helyszíni hálózatokkal. Egy tűzfal farm rendelkezik kisebb specializált képest egy webalkalmazási Tűzfallal rendelkező szoftvert, de szűrését, és vizsgálja meg a bejövő és kimenő forgalmat bármilyen típusú széles körű alkalmazása hatóköre. Tűzfal-farmok általában hálózati virtuális berendezések (nva-k), amelyek elérhetők az Azure Marketplace-en keresztül megvalósított az Azure-ban.

Javasoljuk, hogy használjon egy Nva-készletet az internetről érkező forgalomhoz, és a egy másik származó forgalmat a helyszíni. Biztonsági kockázatot jelent, csak egyetlen nva-k használatával is, mivel nincs biztonsági határ a két fajta hálózati forgalom között biztosít. Különálló nva-k használata csökkenti az ellenőrző biztonsági szabályok összetettségét, és egyértelművé teszi, hogy melyik szabály vonatkozik, hogy mely bejövő hálózati kérésekre.

Legtöbb nagyvállalat kezelheti a több tartomány. Az Azure DNS segítségével az egyes tartományokhoz tartozó DNS-rekordjainak üzemeltetésére. Például az Azure külső load balancer (vagy az alsóbb) Virtual IP Address (VIP) regisztrálni lehet az A rekord az Azure DNS-rekord.

[**Az Azure Load Balancer** ] [ ALB] az Azure load balancer kínál a magas rendelkezésre állású 4. réteg (TCP, UDP) szolgáltatás, amely juttathatja el a bejövő forgalom meghatározott egy elosztott terhelésű készlet szolgáltatási példányai között. A terheléselosztó előtérbeli végpontokból (nyilvános IP-végpontok vagy magánhálózati IP-végpontok) küldött forgalmat újra vagy anélkül címfordítás számára egy háttér-IP-címkészlet (példák folyamatban; Hálózati virtuális berendezések vagy virtuális gépeken).

Az Azure Load Balancer is megvizsgálja, valamint a különböző kiszolgálópéldányok állapotát, és ha egy mintavételező nem válaszol a terheléselosztó nem irányít forgalmat küld a nem megfelelő állapotú példány. Az a VDC van egy külső terheléselosztó jelenlétét az agyban (például elosztás a forgalmat az nva-k), és a küllők (például egy többrétegű alkalmazás különböző virtuális gépek közötti forgalom terheléselosztási feladatok végrehajtásához).

[**Az Application Gateway** ] [ AppGW] Microsoft Azure Application Gateway egy alkalmazáskézbesítési vezérlőt (ADC) biztosító dedikált virtuális berendezés-szolgáltatás, 7 terheléselosztási réteg különböző ajánlat lehetőséget nyújt alkalmazásának. Ez lehetővé teszi, hogy optimalizálhatják a webfarmok termelékenységét a processzorigényes SSL-lezárások application Gateway felé történő kiszervezésével. Ezen túlmenően egyéb 7. rétegbeli útválasztási lehetőségeket is kínál, beleértve a bejövő forgalom ciklikus időszeleteléses elosztását, a cookie-alapú munkamenet-affinitást, az URL-alapú útválasztást, valamint egyetlen Application Gateway mögött több webhelyet is üzemeltethet. Az Application Gateway WAF termékváltozata tartalmaz egy webalkalmazási tűzfalat is, Ez a Termékváltozat webalkalmazásokat a gyakori internetes biztonsági rések és az azokat kihasználó támadások ellen védelmet biztosít. Az Application Gateway szolgáltatást internetes átjáróként, csak belső használatú átjáróként vagy a kettő kombinációjaként lehet konfigurálni. 

[**Nyilvános IP-címek** ] [ PIP] egyes Azure-szolgáltatások engedélyezése, hogy társítsa a nyilvános IP-címre, amely lehetővé teszi, hogy az erőforrást az internetről elérhető végpontok. Ez a végpont hálózati címfordítás (NAT) használatával irányíthatja a forgalmat a belső cím és port, az Azure-beli virtuális hálózaton. Ez az elérési út a külső forgalmat a virtuális hálózatban módját. A nyilvános IP-címek beállítható úgy, hogy mely forgalom átadott és hol és hogyan lehet a virtuális hálózaton keresztül lefordított meghatározása.

#### <a name="component-type-monitoring"></a>Összetevő típusa: figyelése
Figyelési összetevők látható-e, és a más összetevők adattípusok riasztási adja meg. Csapatok mindegyikével kell férniük a figyelést az összetevők és szolgáltatások hozzáféréssel rendelkeznek. Vannak egy központi súgó segélyszolgálathoz vagy a műveleti csoportok, hozzáférés az adatokhoz, ezek az összetevők által biztosított integrált kell.

Naplózás és nyomon követéséhez a viselkedését az Azure-szolgáltatások figyelésének különböző típusú Azure-ajánlatok üzemeltetett erőforrásokhoz. Cégirányítási és vezérlés az Azure-beli számítási feladatok nem alapján nem csupán a teljesítményadatok gyűjtését. napló, de is lehetővé teszi bizonyos jelentett események alapján, műveleteket aktiválhat.

[**Az Azure Monitor** ] [ Monitor] – Azure több szolgáltatást tartalmaz, amelyek külön-külön végrehajtani egy adott szerepkör vagy a feladatot a figyelés területen. Ezek a szolgáltatások együtt egy átfogó megoldást nyújtanak az alkalmazások és az azt támogató Azure-erőforrások telemetriaadatainak gyűjtésére és elemzésére, valamint ezek alapján műveletek végrehajtására. Emellett használhatók kritikus fontosságú helyszíni erőforrások monitorozására egy hibrid monitorozási környezet biztosítása érdekében. A rendelkezésre álló eszközök és adatok megismerése az első lépés az alkalmazás teljes monitorozási stratégiájának kidolgozásában.

Naplók az Azure-ban két fő típusa van:

-   [**A Tevékenységnaplók** ] [ ActLog] (mint "Műveleti napló" is nevezik) az Azure-előfizetéshez tartozó erőforrásokon végrehajtott műveletekkel kapcsolatos információk találhatók meg. Ezek a naplók a vezérlősík események az előfizetésekre vonatkozó jelentést. Minden Azure-erőforrás naplókat eredményez.

-   [**Az Azure diagnosztikai naplók** ] [ DiagLog] az erőforrás által létrehozott naplók, a műveletet az erőforrás vonatkozó részletes, gyakori adatokkal szolgálnak. Ezek a naplók a tartalom erőforrás típusa szerint változó.

[![9]][9]

Az egy VDC nagyon fontos az NSG-k naplók, különösen ezeket az információkat nyomon követéséhez:

-   [**Eseménynaplók**][NSGLog]: arról nyújt tájékoztatást, virtuális gépeket és példányszerepköröket MAC-cím alapján milyen NSG-szabályok érvényesek.
-   [**Naplók számláló**][NSGLog]: nyomon követi az egyes NSG-szabályt a rendszer a forgalom engedélyezése vagy megtagadása hajtott végre futtatásainak számát.

Az összes napló naplózási, elemzési statikus vagy biztonsági mentési célból tárolhatók az Azure Storage-fiókokról. A naplók találhatók, az Azure storage-fiókot, ha ügyfelek különböző típusú keretrendszerek használatával lekérni, -előkészítési, elemzése, és jelenítse meg az adatokat jelenti az állapotot és a felhőbeli erőforrások állapotát.

A nagyobb vállalatok olyan szabványos keretrendszer, a helyszíni rendszerek figyelése már vásárolt, és kiterjesztheti a felhőben üzemelő példányok által létrehozott naplók integrálása a keretrendszer. Olyan szervezeteknek, amelyek az összes naplózási tartani a felhőben, [a Log Analytics] [... / log-analytics/log-analytics-overview .md] remek választás lehet. A Log Analytics felhőalapú szolgáltatásként van megvalósítva, mivel rendelkezhet, és gyorsan az infrastrukturális szolgáltatásokra fordítandó minimális kiadások mellett a. A log Analytics is integrálhatók a System Center-összetevőket, például a System Center Operations Manager bővítése a meglévő felügyeleti beruházások kiterjeszthetők a felhőre.

A log Analytics szolgáltatása az Azure-ban, amely segít összegyűjtését, összekapcsolását, keressen, és operációs rendszerek, alkalmazások és infrastruktúra felhőalapú összetevők által generált napló- és teljesítményadatokat az adatokkal műveleteket végezni. Ügyfelek elemezze a feladatait egy VDC az összes rekordot integrált keresést és egyéni irányítópultok segítségével valós idejű az operational insights biztosít.

A [Network Performance monitort (NPM)] [ NPM] megoldás OMS belül is biztosít részletes hálózati információkat-végpontok, beleértve az Azure-hálózatok és a helyszíni hálózatok egyetlen nézetben. Az ExpressRoute- és nyilvános szolgáltatások adott figyelőt.

#### <a name="component-type-workloads"></a>Összetevő típusa: számítási feladatok
Számítási feladatok összetevői a tényleges alkalmazások és szolgáltatások-ket. Emellett akkor is, ahol az alkalmazás fejlesztői részlegeknek legtöbb idejüket.

A számítási lehetőségek száma végtelen szempontból alapvető fontosságú rendszereivel. A lehetséges munkaterhelés-típusok közül a következők:

**Belső ÜZLETÁGI alkalmazások**. Üzleti alkalmazások olyan nagyvállalati a folyamatban lévő műveletet, kritikus fontosságú számítógépes alkalmazásokat. LOB-alkalmazások néhány közös jellemzőkkel rendelkeznek:

-   **Interaktív**. ÜZLETÁGI alkalmazások interaktívak jellege: adatok, és a visszaadott eredmény/jelentések.
-   **Adatvezérelt**. LOB-alkalmazások olyan nagy számításigényű a gyakori hozzáférést az adatbázisok vagy más tárolási adatok.
-   **Integrált**. LOB-alkalmazások ajánlat integráció más rendszerekkel belül vagy a szervezeten kívülre.

**Ügyfelek által használt webhelyek (interneten vagy belső kapcsolódó)**. A legtöbb alkalmazás, amely az internettel olyan webhelyek. Az Azure lehetőséget nyújt a webhelyek IaaS virtuális gépen vagy a futtatásához egy [Azure Web Apps] [ WebApps] hely (PaaS). Az Azure Web Apps támogatja a virtuális hálózatok, amelyek lehetővé teszik az üzembe helyezés a Web Apps a küllőkben a VDC-a-integrációt. Belső irányuló webhelyeket, a VNET-integráció, ha nem kell az alkalmazások internetes végpont teszi közzé, de is inkább az erőforrások nem internetes irányítható magáncímeket a magánhálózati vnet-n keresztül.

**Big Data típusú adatok/elemzések** vertikális felskálázás akár egy igen nagy mennyiségű adat van szüksége, amikor adatbázisok előfordulhat, hogy nem vertikális felskálázás megfelelően. Hadoop technológiát kínál a rendszer a nagy számú csomópont elosztott lekérdezések párhuzamos futtatásához. Az ügyfél rendelkezik lehetőség data számítási feladatok futtatása az IaaS virtuális gépek vagy PaaS ([HDInsight][HDI]). HDInsight helyalapú vnetbe üzembe helyezést támogatják, is telepíthető egy adott küllőre a VDC-fürtön.

**Események és üzenetküldési**. [Az Azure Event Hubs] [ EventHubs] egy rendkívül nagy kapacitású, telemetriai feldolgozó szolgáltatás, amely összegyűjti a összegyűjteni, átalakítani és több millió eseményt tárolja. Elosztott streamelési platform biztosít alacsony késéssel és konfigurálható idejű adatmegőrzést biztosít, lehetővé téve nagy mennyiségű telemetria betöltését az Azure-ba, és több alkalmazás adatokat olvasni. Az Event hubs szolgáltatással egyetlen streammel támogathat valós idejű és kötegelt folyamatok.

A magas megbízhatóságú felhőalapú üzenetküldő szolgáltatásra alkalmazások és szolgáltatások közötti megvalósítható [Azure Service Bus] [ ServiceBus] , hogy aszinkron ajánlatok által felügyelt üzenetküldési ügyfél és kiszolgáló közötti a a strukturált első-first out (FIFO) üzenetkezelés, közzététel és feliratkozás lehetőségét.

[![10]][10]

### <a name="multiple-vdc"></a>Több VDC
Ez a cikk az eddigi egy egyetlen VDC, az alapvető összetevők és a egy rugalmas VDC közreműködik architektúra leíró rendelkezik összpontosít. Azure-funkciók például az Azure load balancer, nva-k, a rendelkezésre állási csoportok, méretezési csoportokat, illetve más mechanizmusok járulnak hozzá a rendszer, amely a termelési szolgáltatásba szilárd SLA-szintek létrehozását teszik lehetővé.

Egyetlen VDC azonban egy adott régión belül üzemel, és jelentős kimaradás, amelyek a teljes régióra hatással lehetnek kitéve. Ügyfelek, amelyek magas SLA-k elérése érdekében a szolgáltatás üzemelő példányainak helyezni különböző régiókban lévő két (vagy több) Ügyfélszempontokat ugyanazon a projekten keresztül kell.

SLA vonatkozik kívül számos gyakori forgatókönyvek, ahol üzembe helyezése több Ügyfélszempontokat logikus:

-   Regionális és globális jelenlét
-   Vészhelyreállítás
-   A tartományvezérlő közötti forgalom átirányít mechanizmus

#### <a name="regionalglobal-presence"></a>Regionális és globális jelenlét
Azure-adatközpontokban világszerte számos régióban találhatók. Több Azure-beli adatközpontok kiválasztásakor ügyfeleknek kell figyelembe venni két tényezőt összefüggő: földrajzi távokat és a késés. Az ügyfeleknek kell kiértékelni a az Ügyfélszempontokat, és a legjobb felhasználói élményt nyújtani a VDC és a végfelhasználók közötti távolság közötti földrajzi távolságtól.

Az Azure-régió, ahol Ügyfélszempontokat üzemeltetett is meg kell felelniük a szabályozási követelmények alapján, amely a szervezet működik minden jogi joghatósága által létrehozott.

#### <a name="disaster-recovery"></a>Vészhelyreállítás
A vész-helyreállítási terv megvalósítása erősen kapcsolódik, az érintett számítási feladat típusától és a számítási feladatok állapota, között különböző Ügyfélszempontokat szinkronizálhatók. Ideális esetben a legtöbb ügyfél szeretné szinkronizálni az alkalmazás adatokat két különböző Ügyfélszempontokat egy gyors feladatátvételi olyan mechanizmus megvalósításához az üzemelő példányok között. A legtöbb alkalmazás késés-és nagybetűket, és ami hibákat okozhat lehetséges időkorlátja és a késleltetés az adatok szinkronizálását.

Szinkronizálás vagy a szívverés-figyelés az alkalmazások a különböző Ügyfélszempontokat szükséges a köztük folyó kommunikációt. Különböző régiókban lévő két Ügyfélszempontokat keresztül csatlakozhat:

-   – Virtuális hálózatok közötti társviszony-létesítés virtuális hálózatok közötti Társviszony kapcsolódhatnak hubs régióban
-   Az ExpressRoute privát társviszony-létesítés a VDC-hubs ugyanahhoz az ExpressRoute-kapcsolatcsoporthoz csatlakoztatott
-   több ExpressRoute-kapcsolatcsoporttal a vállalati gerinchálózatán keresztül kapcsolódik, és a VDC-háló csatlakozik az ExpressRoute-Kapcsolatcsoportok
-   Site-to-Site VPN-kapcsolatok a VDC hubs, az egyes Azure-régió között

Virtuális hálózatok közötti Társviszony- vagy ExpressRoute-kapcsolatok általában az előnyben részesített mechanizmus nagyobb sávszélességet és egységes késés miatt, ha a Microsoft gerinchálózatán keresztül áthaladó.

Nincs nem magic recept különböző régiókban található két (vagy több) különböző Ügyfélszempontokat között elosztott alkalmazás ellenőrzése. Ügyfelek hálózati minősítési tesztekkel a késés és a kapcsolatok és a cél sávszélesség szinkron vagy aszinkron adatreplikációt-e megfelelő, és mi az optimális helyreállítási időre vonatkozó célkitűzés (RTO) lehet a számítási feladatok esetében érdemes futtatnia.

#### <a name="mechanism-to-divert-traffic-between-dc"></a>A tartományvezérlő közötti forgalom átirányít mechanizmus
Egy hatékony módszer átirányít a forgalmat bejövő a egy tartományvezérlő, DNS alapul. [Az Azure Traffic Manager] [ TM] a végfelhasználói forgalmat a leginkább megfelelő nyilvános végponthoz való közvetlen egy adott VDC a a tartománynévrendszer (DNS) mechanizmust használ. Keresztül mintavételek menüpontban a Traffic Manager rendszeresen ellenőrzi a különböző Ügyfélszempontokat nyilvános végpontok a szolgáltatás állapotát, és ezekre a végpontokra meghiúsulásának azt továbbítja automatikusan a másodlagos VDC.

A TRAFFIC Manager az Azure nyilvános végpontok működik, és használható, például vezérlő/átirányít az Azure virtuális gépek és a megfelelő VDC a Web Apps-forgalom. A TRAFFIC Manager képes legyen ellenállni egy teljes Azure-régiót meghibásodása esetén is, és szabályozhatja a különböző Ügyfélszempontokat (például hiba egy adott VDC, vagy a VDC válassza a szolgáltatás több feltétel alapján szolgáltatásvégpontokra érkező felhasználói forgalom elosztása a legalacsonyabb hálózati késéssel rendelkező ügyfél).

### <a name="conclusion"></a>Összegzés
A virtuális adatközpont egy data center migrálása a felhőbe, hozzon létre egy méretezhető architektúra az Azure-ban, amely maximalizálja a felhőalapú erőforrások használatával, csökkentve a költségeket, és egyszerűsítse a rendszer cégirányítási funkciók és képességek kombinációját használó megközelítést. A VDC fogalma a Központ-küllő topológia közös megosztott szolgáltatások az agyi, és lehetővé teszi az adott alkalmazások és számítási feladatok a küllők az alapján történik. A VDC vállalati szerepkörök, ahol különböző részlegek (központi informatikai, DevOps, üzemeltetése és karbantartása) együttműködnek egymással, a szerepkörök és feladatkörök egy adott lista egyes struktúrája megegyezik. A VDC-megfelel a "Lift and Shift" áttelepítés, de a natív felhőalapú rendszerbe sok előnyt is biztosít.

## <a name="references"></a>Referencia
Ez a dokumentum a következő funkciókat is tárgyalja. További hivatkozásokra kattintva.

| | | |
|-|-|-|
|Hálózati szolgáltatások|Terheléselosztás|Kapcsolatok|
|[Azure virtuális hálózatok][VNet]</br>[Hálózati biztonsági csoportok][NSG]</br>[NSG-naplók][NSGLog]</br>[Felhasználó által megadott útvonal][UDR]</br>[Hálózati virtuális berendezések][NVA]</br>[Nyilvános IP-címek][PIP]</br>[DNS]|[Az Azure Load Balancer (3.) ][ALB]</br>[Az Alkalmazásátjáró (7. rétegbeli) ][AppGW]</br>[Webalkalmazási tűzfal][WAF]</br>[Az Azure Traffic Manager][TM] |[Virtuális hálózatok közötti Társviszony][VNetPeering]</br>[Virtuális magánhálózat][VPN]</br>[ExpressRoute][ExR]
|Identitás</br>|Figyelés</br>|Ajánlott eljárások</br>|
|[Azure Active Directory][AAD]</br>[A multi-factor Authentication szolgáltatás][MFA]</br>[Szerepkör alap hozzáférés-vezérlés][RBAC]</br>[Alapértelmezett AAD-szerepkörök][Roles] |[Az Azure Monitor][Monitor]</br>[Tevékenységnaplók][ActLog]</br>[Diagnosztikai naplók][DiagLog]</br>[A Microsoft Operations Management Suite][OMS]</br>[A Network Performance Monitor][NPM]|[Szegélyhálózat-alapú hálózatok, ajánlott eljárások][DMZ]</br>[Előfizetések kezelése][SubMgmt]</br>[Erőforrás-csoportok kezelése][RGMgmt]</br>[Azure-előfizetés korlátai][Limits] |
|Más Azure-szolgáltatások|
|[Az Azure Web Apps alkalmazások][WebApps]</br>[Hdinsight (Hadoop) ][HDI]</br>[Event Hubs][EventHubs]</br>[Szolgáltatásbusz][ServiceBus]|



## <a name="next-steps"></a>További lépések
 - Ismerkedés a [virtuális hálózatok közötti társviszony-létesítés][VNetPeering], a megerősítő technológiát a VDC küllős tervek
 - Alkalmazzon [AAD] [ AAD] használatába [RBAC] [ RBAC] feltárása
 - Előfizetés és az erőforráscsoport felügyeleti modell fejlesztése és RBAC modell szerkezetét, követelmények, teljesítéséhez és a szervezet szabályzatainak. A legfontosabb tevékenység tervezi. Sokkal gyakorlati tervezze meg a átszervezések, fúzió, új termékcsaládok, stb.

<!--Image References-->
[0]: ./images/networking-redundant-equipment.png "összetevő átfedés példái" 
[1]: ./images/networking-vdc-high-level.png "küllős VDC magas szintű példája"
[2]: ./images/networking-hub-spokes-cluster.png "fürt hubs és a küllők"
[3]: ./images/networking-spoke-to-spoke.png "Spoke-to-spoke"
[4]: ./images/networking-vdc-block-level-diagram.png "letiltása a VDC szintű ábrája"
[5]: ./images/networking-users-groups-subsciptions.png "felhasználók, csoportok, előfizetések és projektek"
[6]: ./images/networking-infrastructure-high-level.png "magas szintű infrastruktúra diagramja"
[7]: ./images/networking-highlevel-perimeter-networks.png "magas szintű infrastruktúra diagramja"
[8]: ./images/networking-vnet-peering-perimeter-neworks.png "virtuális hálózatok közötti Társviszony és a szegélyhálózat-alapú hálózatok"
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
[NVA]: /azure/architecture/reference-architectures/dmz/nva-ha
[SubMgmt]: /azure/azure-resource-manager/resource-manager-subscription-governance 
[RGMgmt]: /azure/azure-resource-manager/resource-group-overview
[DMZ]: /azure/best-practices-network-security
[ALB]: /azure/load-balancer/load-balancer-overview
[PIP]: /azure/virtual-network/resource-groups-networking#public-ip-address
[AppGW]: /azure/application-gateway/application-gateway-introduction
[WAF]: /azure/application-gateway/application-gateway-web-application-firewall-overview
[Monitor]: /azure/monitoring-and-diagnostics/
[ActLog]: /azure/monitoring-and-diagnostics/monitoring-overview-activity-logs 
[DiagLog]: /azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs
[NSGLog]: /azure/virtual-network/virtual-network-nsg-manage-log
[OMS]: /azure/operations-management-suite/operations-management-suite-overview
[NPM]: /azure/log-analytics/log-analytics-network-performance-monitor
[WebApps]: /azure/app-service/
[HDI]: /azure/hdinsight/hdinsight-hadoop-introduction
[EventHubs]: /azure/event-hubs/event-hubs-what-is-event-hubs 
[ServiceBus]: /azure/service-bus-messaging/service-bus-messaging-overview
[TM]: /azure/traffic-manager/traffic-manager-overview
