---
title: "Integrálható a helyszíni AD-tartományok az Azure Active Directoryval"
description: "Hogyan lehet egy Azure Active Directory használatával biztonságos hibrid hálózati architektúra végrehajtásához."
author: telmosampaio
pnp.series.title: Identity management
ms.date: 11/28/2016
pnp.series.next: adds-extend-domain
pnp.series.prev: ./index
cardTitle: Integrate on-premises AD with Azure AD
ms.openlocfilehash: dd4cf0369974ea68d240ed294b1c50972d361d74
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="integrate-on-premises-active-directory-domains-with-azure-active-directory"></a>A helyszíni Active Directory-tartományok integrálása az Azure Active Directoryval

Azure Active Directory (Azure AD) a felhő alapú több-bérlős címtár- és identitáskezelési szolgáltatást. A referencia-architektúrában jeleníti meg az ajánlott eljárások a helyszíni Active Directory-tartományok integrálása az Azure AD identity felhőalapú hitelesítés biztosítására. [**Ez a megoldás üzembe helyezéséhez**.](#deploy-the-solution)

[![0]][0] 

*Töltse le a [Visio fájl] [ visio-download] ezen architektúra.*

> [!NOTE]
> Az egyszerűség a diagram csak azt mutatja be, a kapcsolatok közvetlenül kapcsolódik az Azure AD, és a protokoll kapcsolatos nem fordulhat elő, hitelesítés és identitás-összevonási részeként. Például egy webes alkalmazás is átirányítási a webböngésző hitelesíteni a kérelmet az Azure AD. Ha hitelesítése megtörtént, a kérelem átadhatók vissza a webalkalmazás, a megfelelő azonosító adatok.
> 

A referenciaarchitektúra tipikus használati többek között:

* A webalkalmazások Azure szolgáltatásba telepített, amely hozzáférést biztosít a szervezet tartozik távoli felhasználók számára.
* Végrehajtási a végfelhasználók számára, például a jelszavak alaphelyzetbe állítását és a csoport kezelésének delegálása önkiszolgálói képességeit. Vegye figyelembe, hogy ehhez a Azure AD prémium kiadás.
* Architektúrák, amelyben a helyszíni hálózat és az alkalmazás Azure virtuális hálózat nem kapcsolódnak VPN-alagúton vagy ExpressRoute-kapcsolatcsoportot.

> [!NOTE]
> Az Azure AD jelenleg csak a felhasználói hitelesítést is támogatja. Egyes alkalmazások és szolgáltatások, például az SQL Server, szükség lehet a számítógépek hitelesítését, ebben az esetben ez a megoldás nem megfelelő.
> 

További szempontokat lásd: [megoldás választása a integrálása a helyszíni Active Directoryról szinkronizálva az Azure][considerations]. 

## <a name="architecture"></a>Architektúra

Az architektúra a következő részből áll.

* **Az Azure AD-bérlő**. Példányának [az Azure AD] [ azure-active-directory] a szervezet által létrehozott. Objektumok másolja a helyszíni Active Directoryból való elhelyezésével úgy működik, mint egy címtárszolgáltatás felhőalapú alkalmazásokhoz, és identitás-szolgáltatásokat biztosít.
* **Webes réteg alhálózati**. Ez az alhálózat egy webalkalmazást futtató virtuális gépeken tartalmazza. Az Azure AD egy identitás broker ehhez az alkalmazáshoz működhet.
* **A helyszíni AD DS kiszolgálói**. A helyszíni címtár- és identitáskezelési szolgáltatásnak. Az Active Directory tartományi szolgáltatások directory lehet szinkronizálni az Azure AD hitelesíti a helyi felhasználókat, hogy engedélyezve legyen.
* **Az Azure AD Connect szinkronizálási kiszolgálót**. Egy helyszíni számítógép, amelyen a [az Azure AD Connect] [ azure-ad-connect] a szinkronizálási szolgáltatás. Ez a szolgáltatás az Azure ad Szolgáltatásba a helyszíni Active Directoryban tárolt adatokat szinkronizálja. Például kiépítése, vagy a felhasználók és csoportok a helyszíni kiosztásának megszüntetése, ha ezek a változások propagálása az Azure AD. 
  
  > [!NOTE]
  > Biztonsági okokból az Azure AD felhasználói jelszavak tárolja egy kivonatot. Ha a felhasználó a jelszó alaphelyzetbe állítását igényli, ennek kell lennie a helyszínen történik, és az új kivonatoló kell küldeni az Azure AD. Az Azure AD prémium kiadás tartalmazza a automatizálhat Ez a feladat engedélyezése a felhasználóknak új jelszót készíthessenek maguknak szolgáltatásokat.
  > 

* **Virtuális gépek N szintű alkalmazáshoz**. A központi telepítés infrastruktúra N szintű alkalmazásokra vonatkozóan tartalmazza. Ezekkel az erőforrásokkal kapcsolatos további információkért lásd: [N szintű architektúrát a virtuális gépek futtatásához][implementing-a-multi-tier-architecture-on-Azure].

## <a name="recommendations"></a>Javaslatok

Az alábbi javaslatok a legtöbb forgatókönyvre vonatkoznak. Kövesse ezeket a javaslatokat, ha nincsenek ezeket felülíró követelményei. 

### <a name="azure-ad-connect-sync-service"></a>Az Azure AD Connect szinkronizálási szolgáltatás

Az Azure AD Connect szinkronizálási szolgáltatás biztosítja, hogy a felhőben tárolt azonosító adatok, amelyek a helyszínen tárolt összhangban. Az Azure AD Connect szoftver a szolgáltatást telepíti. 

Az Azure AD Connect szinkronizálása előtt, a szervezet szinkronizálási követelmények meghatározása. Például, mi a szinkronizálás, mely tartományok, és hogy milyen gyakran. További információkért lásd: [határozza meg a címtár-szinkronizálás követelményeinek][aad-sync-requirements].

Az Azure AD Connect szinkronizálási szolgáltatást futtathatja a virtuális gép vagy egy számítógépen helyben tárolt. Attól függően, hogy az információ az Active Directory címtárban volatilitásától az Azure AD Connect szinkronizálási szolgáltatás terhelése valószínű, hogy az Azure AD az első szinkronizálást követően magas lehet. A szolgáltatás fut a virtuális gép megkönnyíti a kiszolgáló méretezési, ha szükséges. A figyelő a virtuális Gépen a figyelés szempontjai című részben leírtak szerint annak megállapításához, hogy a skálázás tevékenység szükség.

Ha egy erdő több helyszíni tartományban van, érdemes tárolni, és a teljes erdőben egyetlen az adatok szinkronizálása az Azure AD bérlői. Egynél több tartományban előforduló identitások információi szűréséhez, így a minden egyes identitás csak egyszer jelenik meg az Azure ad-ben, ahelyett, hogy éppen ismétlődik. Ismétlődést inkonzisztenciát vezethet, ha adatok szinkronizálása. További információkért lásd: az alábbi topológia szakaszt. 

Használja a szűréshez, hogy csak a szükséges adatokat az Azure AD-ban tárolja. Például a szervezet nem szeretné inaktív fiókok adatait tárolja az Azure ad-ben. Szűrés lehet a csoport-, a tartomány, szervezeti egység (OU)-alapú vagy Attribútumalapú. Az összetettebb szabályokat létrehozni szűrők kombinálhatja. Például tudott szinkronizálni, amelyek egy adott értéket a kijelölt attribútum egy tartományban lévő objektumok számára. Részletes információkért lásd: [az Azure AD Connect szinkronizálása: a szűrés konfigurálása][aad-filtering].

Az AD Connect szinkronizálási szolgáltatás magas rendelkezésre állásának megvalósításához, egy másodlagos átmeneti kiszolgálón fut. További információ az című topológia javaslatokat.

### <a name="security-recommendations"></a>Biztonsági javaslatok

**Felhasználói jelszavak kezelését.** A prémium szintű Azure AD-kiadásokat támogatja a jelszóvisszaírás engedélyezése az Azure portálon az önkiszolgáló jelszó-átállításra végrehajtásához a helyi felhasználók. Ez a funkció csak engedélyezni kell a szervezet biztonsági jelszóházirend áttekintése után. Például korlátozhatja a felhasználók módosíthatják a jelszavukat, és átalakítható a jelszó felügyeletét. További információkért lásd: [jelszókezelés testreszabása a szervezet igényeinek megfelelően][aad-password-management]. 

**A helyszíni alkalmazások kívülről elérhető védelmét.** Az Azure AD alkalmazásproxy segítségével keresztül az Azure AD külső felhasználók számára a helyszíni webalkalmazások szabályozott hozzáférést biztosítanak. Csak a felhasználók az Azure-címtár érvényes hitelesítő adatokkal rendelkező engedélye az alkalmazás használatára. További információkért lásd: a cikk [alkalmazásproxy engedélyezése az Azure-portálon][aad-application-proxy].

**Azure ad-val gyanús tevékenység jeleit aktív figyelését.**    Azure AD Premium P2 kiadása, amely tartalmazza az Azure AD Identity Protection érdemes lehet. Azonosító adatok védelmét adaptív gépi tanulási algoritmusok és heurisztikus használja a rendellenességek észlelését, és eseményeket, amelyek azt jelzik, hogy identitás feltörték kockázatát. Például azonosítására képes például szabálytalan bejelentkezési tevékenységek, bejelentkezések ismeretlen forrásból származó vagy IP-címekről a gyanús tevékenységeket, vagy előfordulhat, hogy fertőzött eszközökről bejelentkezések potenciálisan szokatlan tevékenységet. Ezeket az adatokat használva Identity Protection állít elő, jelentéseket és riasztásokat, amely lehetővé teszi, hogy a kockázati eseményekről vizsgálata, és hajtsa végre a megfelelő műveletet. További információkért lásd: [Azure Active Directory Identity Protection][aad-identity-protection].
  
Az Azure-portálon az Azure AD jelentéskészítési szolgáltatás segítségével biztonsági tevékenységek a rendszer figyelni. Ezek a jelentések használatával kapcsolatos további információkért lásd: [Azure Active Directory-jelentéskészítés – útmutató][aad-reporting-guide].

### <a name="topology-recommendations"></a>Topológia javaslatok

Konfigurálja az Azure AD Connect a topológia, amely a legjobban illik a szervezet követelményeinek megvalósításához. Az Azure AD Connect támogató topológiák közé tartoznak a következők:

* **Egyetlen erdő, egyetlen Azure AD-címtár**. Ebben a topológiában az Azure AD Connect szinkronizálása objektumok és az azonosító adatokat egyetlen egy vagy több tartományt a helyi erdő egyetlen Azure AD-bérlő. Ez az az alapértelmezett topológia valósítják meg az Azure AD Connect a gyorstelepítés.
  
  > [!NOTE]
  > Nem több Azure AD Connect szinkronizálási kiszolgálót való csatlakozáskor használandó eltérő tartományokban helyszíni ugyanabban az erdőben az azonos Azure AD-bérlő, kivéve, ha futtatja a kiszolgáló átmeneti módban, az alábbiakban.
  > 
  > 

* **Több erdő, egyetlen Azure AD-címtár**. Ebben a topológiában az Azure AD Connect szinkronizálása az objektumok és az azonosító adatok több erdőből egyetlen Azure AD-bérlő. Ez a topológia használja, ha a szervezete helyszíni egynél több erdő. A konszolidáció azonosító adatokat, hogy minden egyedi felhasználói egyszer jelennek meg az Azure AD-címtárban, akkor is, ha ugyanaz a felhasználó egynél több erdő. Összes erdőben ugyanazon az Azure AD Connect szinkronizálási kiszolgáló használatára. Az Azure AD Connect szinkronizálási kiszolgálót nem kell minden tartománynak lehet része, de ennek elérhetőnek kell lennie az összes erdőben.
  
  > [!NOTE]
  > Ebben a topológiában ne használjon külön Azure AD Connect szinkronizálási kiszolgálók minden olyan erdőben, a helyszíni csatlakozni egy egyetlen Azure AD bérlői. Ez eredményezhet duplikált azonosító adatokat az Azure AD-Ha a felhasználók több erdőben találhatók.
  > 
  > 

* **Több erdő, külön topológiák**. Ez a topológia azonosító adatokat a különálló erdők egyesít egyetlen Azure AD-bérlő minden erdők külön entitásokként kezelésére. Ez a topológia akkor hasznos, ha meg vannak kombinálja a különböző szervezetek erdők és a azonosító adatok minden felhasználónak csak egy erdő használatban van.
  
  > [!NOTE]
  > Szinkronizálva a globális listák (GAL) minden olyan erdőben, ha egy felhasználó egy erdő lehet egy másik ügyfélként. Ez akkor fordulhat elő, ha a szervezet a Forefront Identity manager 2010 vagy a Microsoft Identity Manager 2016 javítást alkalmazott GALSync. Ebben a forgatókönyvben, megadhatja, hogy a felhasználók által azonosítható a *Mail* attribútum. Identitások használatával is egyben a *ObjectSID* és *msExchMasterAccountSID* attribútumok. Ez akkor hasznos, ha egy vagy több erőforráserdővel letiltott fiókok.
  > 
  > 

* **Átmeneti server**. Ebben a konfigurációban a második példányát az Azure AD Connect szinkronizálási kiszolgálót az első párhuzamosan futtatja. Ez a struktúra, mint a forgatókönyveket teszi lehetővé:
  
  * Magas rendelkezésre állású.
  * Tesztelése és üzembe helyezése az Azure AD Connect szinkronizálási kiszolgáló új konfigurációt.
  * Új kiszolgáló bevezetésével, és egy régi konfigurációs leszerelése. 
    
    Ezekben az esetekben a második példány fut *átmeneti módban*. A kiszolgáló importált objektumokra és a szinkronizálási adatokat rögzíti az adatbázisában, de nem felel meg az adatokat az Azure ad Szolgáltatásba. Ha letiltja az átmeneti környezetű üzemmód, a kiszolgáló adatok írása az Azure AD, pedig is kezdődik jelszó írási-biztonsági készít a helyszíni címtárak történő végrehajtása megfelelő. További információkért lásd: [az Azure AD Connect szinkronizálása: működtetési feladatok és szempontok][aad-connect-sync-operational-tasks].

* **Több Azure AD-címtártól**. Javasoljuk, hogy hozzon létre egy egyetlen Azure AD könyvtár, egy olyan szervezet, de előfordulhat, hogy ha szükség a partíciónak az adatait keresztben külön az Azure AD-címtártól. Ebben az esetben szinkronizálás és a jelszó késleltetve visszaírt problémák elkerülése biztosításával, hogy a helyi erdő minden objektum csak egy Azure AD-címtár megjelenik. Ez a forgatókönyv végrehajtásához konfigurálhatja külön az Azure AD Connect szinkronizálása minden Azure AD-címtár kiszolgálókat, és úgy, hogy minden Azure AD Connect szinkronizálási kiszolgáló működik, egymást kölcsönösen kizáró csoportján objektumok szűrése. 

Ezek a topológiák kapcsolatos további információkért lásd: [az Azure AD Connect topológiák][aad-topologies].

### <a name="user-authentication"></a>Felhasználóhitelesítés

Alapértelmezés szerint az Azure AD Connect szinkronizálási kiszolgáló konfigurálása a jelszó-szinkronizálás a helyszíni tartományi és az Azure AD között, és az Azure AD szolgáltatás azt feltételezi, hogy a felhasználók hitelesítéséhez azáltal, hogy ugyanazt a jelszót, hogy a helyszíni használnak. A legtöbb szervezet számára ez megfelelő, de vegye figyelembe a szervezet meglévő házirendek és infrastruktúrát. Példa:

* A szervezete biztonsági házirendjével, amelyek esetleg szinkronizálása a felhőbe azok kivonatai.
* Szüksége lehet, hogy a felhasználók észlelnek zökkenőmentes egyszeri bejelentkezés (SSO) felhő elérésekor erőforrások a tartományhoz csatlakoztatott számítógépek a vállalati hálózaton.
* A szervezet előfordulhat, hogy már rendelkezik, az Active Directory összevonási szolgáltatások (AD FS) vagy egy harmadik féltől származó összevonási szolgáltató telepítve. Az Azure AD az infrastruktúra megvalósítása hitelesítés és egyszeri Bejelentkezéssel, nem pedig a felhőben tárolt jelszó-információkat használatával konfigurálhatja.

További információkért lásd: [az Azure AD Connect felhasználói bejelentkezési beállítások][aad-user-sign-in].

### <a name="azure-ad-application-proxy"></a>Az Azure AD-alkalmazásproxy 

Az Azure AD használatával a helyszíni alkalmazások eléréséhez.

A helyszíni webalkalmazások az Azure AD application proxy összetevő által kezelt alkalmazás proxy összekötők használatával teszi közzé. Az alkalmazásproxy-összekötő megnyílik egy kimenő internetkapcsolattal az Azure AD-alkalmazásproxy, és a távoli felhasználók rendszer kérést átirányítja vissza az Azure AD ezen a kapcsolaton keresztül a webalkalmazások. Ez a helyszíni bejövő portok megnyitása nem kell, és csökkenti a támadási felületet, a szervezet által elérhetővé tett.

További információkért lásd: [alkalmazások közzététele az Azure AD-alkalmazásproxy használatával][aad-application-proxy].

### <a name="object-synchronization"></a>Szinkronizációs 

Az Azure AD Connect alapértelmezett konfiguráció szinkronizálja a helyi Active Directory címtárban, a cikkben meghatározott szabályok alapján lévő objektumok [az Azure AD Connect szinkronizálása: az alapértelmezett konfiguráció ismertetése] [ aad-connect-sync-default-rules]. Objektumok, amelyek megfelelnek a szabályok szinkronizálva van, amíg az összes többi objektum figyelmen kívül lesznek hagyva. Néhány példa szabály:

* Felhasználói objektumok rendelkeznie kell egy egyedi *sourceAnchor* attribútum és a *accountEnabled* attribútum ki kell tölteni.
* Felhasználói objektumok rendelkeznie kell egy *sAMAccountName* attribútumot, és a szöveg nem kezdődhet *Azure AD_* vagy *MSOL_*.

Az Azure AD Connect számos szabályt felhasználói, partner, csoport, ForeignSecurityPrincipal, és objektumokra vonatkozik. A szerkesztővel szinkronizálási szabályok az Azure AD Connect telepítése, ha módosítania kell az alapértelmezett szabályok készletét. További információkért lásd: [az Azure AD Connect szinkronizálása: az alapértelmezett konfiguráció ismertetése][aad-connect-sync-default-rules]).

Megadhatja a saját tartomány vagy szervezeti egység szinkronizálandó objektumok szűrőket. Alternatív megoldásként Megvalósíthat összetettebb egyéni szűrés például, amelyek a [az Azure AD Connect szinkronizálása: a szűrés konfigurálása][aad-filtering].

### <a name="monitoring"></a>Figyelés 

Állapotfigyelés a következő ügynököt telepíteni a helyszíni végzi el:

* Az Azure AD Connect telepítése olyan ügynök, amely a szinkronizálási műveleteire vonatkozó adatokat rögzíti. Az Azure-portálon az Azure AD Connect Health panel segítségével a állapotának és teljesítményének figyelése. További információkért lásd: [használata Azure AD Connect Health szinkronizálási szolgáltatás][aad-health].
* Az Active Directory-tartományok és az Azure-ból könyvtárak állapotának figyelésére, telepítse az Azure AD Connect Health ügynök Active Directory tartományi szolgáltatások a tartományba a helyszíni gépen. Állapotfigyelés használni az Azure Active Directory Connect Health panel az Azure portálon. További információkért lásd: [használata Azure AD Connect Health használata az Active Directory tartományi szolgáltatások][aad-health-adds] 
* Az Azure AD Connect Health AD FS-ügynököt a helyszíni szolgáltatás állapotának figyelésére, és az Azure-portálon az Azure Active Directory Connect Health panel használatával figyelheti az AD FS telepítése. További információkért lásd: [használata az Azure AD Connect Health AD FS-sel][aad-health-adfs]

Az AD Connect Health-ügynökök és a rájuk vonatkozó követelményeket telepítésével kapcsolatos további információkért lásd: [az Azure AD Connect Health-ügynök telepítése][aad-agent-installation].

## <a name="scalability-considerations"></a>Méretezési szempontok

Az Azure AD szolgáltatás támogatja a méretezhetőséget, a replikákat, amelyek egyetlen elsődleges replika, amely kezeli az írási műveletek és több csak olvasható másodlagos replika alapján. Az Azure AD transzparens módon átirányítja a felhasználókat megkísérelt írások felé irányuló másodlagos replika az elsődleges másodpéldányhoz, és végleges konzisztencia biztosítja. Az elsődleges másodpéldányhoz végrehajtott valamennyi módosítást továbbítja a másodlagos replikákon. Ez az architektúra arányosan is mivel a legtöbb műveleteket az Azure AD írások helyett olvasási műveletek. További információkért lásd: [az Azure AD: a technikai részletek a georedundáns, magas rendelkezésre állású, elosztott felhő könyvtár alatt][aad-scalability].

Az Azure AD Connect szinkronizálási kiszolgáló határozza meg a hány objektumok valószínűleg a helyi címtárban lévő szinkronizálása. Ha kevesebb mint 100 000 objektummal rendelkezik, az alapértelmezett SQL Server Express LocalDB szoftverre az Azure AD Connect is használhatja. Ha nagy számú objektumok, telepítse az SQL-kiszolgáló éles verzióját, és adja meg, hogy az SQL Server meglévő példányát kell használnia, az Azure AD Connect egyéni telepítése.

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

Az Azure AD szolgáltatás földrajzi eloszlása és futtatása több adatokban adatközpontok automatikus feladatátvételi világszerte terjedésének. Ha egy adott adatközpont elérhetetlenné válik, az Azure AD biztosítja, hogy a címtár adataihoz elérhető példány hozzáférési legalább két a több – elosztott adatközpontokban.

> [!NOTE]
> A szolgáltatásiszint-szerződés (SLA) Azure AD alapszintű és a prémium szolgáltatások garanciák legalább 99,9 % rendelkezésre állását. Az Azure AD ingyenes szint nincs SLA van. További információkért lásd: [SLA-t az Azure Active Directory][sla-aad].
> 
> 

Fontolja meg a kiépítés második példányát az Azure AD Connect szinkronizálási kiszolgáló átmeneti módban magasabb rendelkezésre állását, a topológia javaslatok szakaszban leírtaknak megfelelően. 

Ha az SQL Server Express LocalDB példányát az Azure AD Connect mellékelt nem használja, érdemes lehet magas rendelkezésre állás fürtszolgáltatás SQL. Az Azure AD Connect nem támogatottak a megoldások, például a tükrözött és a mindig bekapcsolva.

Az Azure AD Connect szinkronizálási kiszolgálót, és egy meghibásodás után helyreállítása magas rendelkezésre állás elérése érdekében kapcsolatos további szempontokat lásd: [az Azure AD Connect szinkronizálása: működtetési feladatok és szempontok - vész-helyreállítási] [ aad-sync-disaster-recovery].

## <a name="manageability-considerations"></a>Felügyeleti szempontok

Két szempontot kell az Azure AD kezelése:

* Azure AD szolgáltatás felügyeletéről a felhőben.
* Az Azure AD Connect szinkronizálási kiszolgálók karbantartása.

Az Azure AD-tartományok és a felhőben könyvtárak kezeléséhez az alábbi lehetőségeket kínálja: 

* **Az Azure Active Directory PowerShell-modul**. Ezzel [modul] [ aad-powershell] gyakori az Azure AD felügyeleti feladatokat, mint felhasználókezelés, tartományok és az egyszeri bejelentkezés konfigurálása parancsfájllal történő szüksége.
* **Az Azure AD-felügyelet panel az Azure portálon**. Ezen a panelen annak a könyvtárnak interaktív felügyeleti nézete, és lehetővé teszi a konfigurálhatja az Azure AD a legtöbb funkcióit, és szabályozhatja. 

Az Azure AD Connect telepíti a következő eszközöket a helyszíni gépeket az Azure AD Connect szinkronizálási szolgáltatások karbantartása:
  
* **A Microsoft Azure Active Directory Connect konzol**. Ez az eszköz lehetővé teszi az Azure AD Sync-kiszolgáló konfigurációjának módosítása, testre szabhatja a szinkronizálási módját, engedélyezése vagy az átmeneti környezetű üzemmód letiltása és a felhasználói bejelentkezési mód váltani. Vegye figyelembe, hogy az Active Directory FS bejelentkezés a helyszíni infrastruktúra használatával engedélyezheti.
* **Synchronization Service Managert**. Használja a *műveletek* lapján ezt az eszközt a szinkronizálási folyamat kezelése és észleli, hogy a folyamat bármely részét nem sikerült. Ez az eszköz segítségével manuálisan szinkronizálások indíthat el. A *összekötők* lap lehetővé teszi a tartományok, a Szinkronizáló vezérlő csatolt kapcsolatainak szabályozására.
* **Szinkronizálási szabályok szerkesztő**. Az eszköz segítségével testre szabhatja objektumok átalakításából származnak, ha a helyszíni címtár és az Azure AD közötti másolja őket. Ez az eszköz adhatók meg további attribútumokat és -szinkronizálási objektumok, majd végrehajtja a úgy, hogy mely objektumok kell, vagy nem fognak szinkronizálódni határozza meg. További információkért lásd: a szinkronizálási szabály szerkesztő szakaszt a következő dokumentumban [az Azure AD Connect szinkronizálása: az alapértelmezett konfiguráció ismertetése][aad-connect-sync-default-rules].

További információk és tippek az Azure AD Connect kezelése [az Azure AD Connect szinkronizálása: gyakorlati tanácsok az alapértelmezett konfiguráció módosításának][aad-sync-best-practices].

## <a name="security-considerations"></a>Biztonsági szempontok

Feltételes hozzáférés-vezérlés használatával hitelesítési kérelmek elutasítása váratlan forrásokból:

- Eseményindító [Azure multi-factor Authentication (MFA)] [ azure-multifactor-authentication] Ha egy felhasználó megpróbál kapcsolódni például tartalmainak helyről megbízható hálózat helyett az interneten keresztül.

- Az eszköztípus platform (iOS, Android, Windows Mobile, Windows) felhasználó segítségével meghatározhatja az alkalmazások és szolgáltatások házirend.

- Jegyezze fel a felhasználói eszközök a engedélyezett vagy letiltott állapotban, és ezek az információk beépítse a hozzáférési házirend ellenőrzése. Például ha a felhasználó telefonja elveszett vagy ellopták, feljegyzik eléréséhez használt megakadályozása érdekében le van tiltva.

- Felhasználói hozzáférést az erőforrásokhoz csoporttagság alapján. Használjon [az Azure AD dinamikus tagsági szabályok] [ aad-dynamic-membership-rules] egyszerűbbé teheti a felügyeleti csoport. Egy rövid megtudhatja, hogyan működik, [csoportok dinamikus csoporttagságok bemutatása][aad-dynamic-memberships].

- Házirendekkel a feltételes hozzáférés kockázatot jelent az Azure AD Identity Protection szokatlan bejelentkezési tevékenység vagy egyéb esemény alapján speciális védelem biztosításához.

További információkért lásd: [Azure Active Directory feltételes hozzáférés][aad-conditional-access].

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezéséhez

A környezet egy referencia-architektúrában, amely megvalósítja a javaslatok és szempontokat a Githubon érhető el. A referencia-architektúrában telepíti egy szimulált helyi hálózati teszteléséhez és kísérletezhet használható az Azure-ban. A referencia-architektúrában üzembe helyezhetők Windows vagy Linux rendszerű virtuális gépeken az alábbi utasításokat követve: 

1. Kattintson a lenti gombra:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fidentity%2Fazure-ad%2Fazuredeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>
2. Ha a hivatkozás megnyílt az Azure Portalon, meg kell adnia néhány beállítás értékét: 
   * Az **Erőforráscsoport** neve már meg van adva a paraméterfájlban, ezért válassza az **Új létrehozása** lehetőséget és a szövegmezőbe írja az `ra-aad-onpremise-rg` karakterláncot.
   * Válassza ki a régiót a **Hely** legördülő listából.
   * Ne szerkessze a **Sablon gyökér szintű URI-je** vagy a **Paraméter gyökér szintű URI-je** szövegmezőt.
   * Válassza ki **windows** vagy **linux** a a **operációsrendszer-típus** a legördülő listából.
   * Tekintse át a használati feltételeket, majd kattintson az **Elfogadom a fenti feltételeket** lehetőségre.
   * Kattintson a **beszerzési** gombra.
3. Várjon, amíg az üzembe helyezés befejeződik.
4. A paraméter-fájlok közé tartoznak a kódolt rendszergazda felhasználói neveket és jelszavakat, és erősen ajánlott, hogy azonnal módosítsa a virtuális gépekre is. Kattintson az egyes virtuális gépek az Azure portálon, majd kattintson a **jelszó-átállítási** a a **támogatási + hibaelhárítási** panelen. Válassza ki **jelszó-átállítási** a a **mód** legördülő mezőben, majd válasszon egy új **felhasználónév** és **jelszó**. Az új felhasználó nevének és jelszavának megőrzéséhez kattintson a **Frissítés** gombra.

<!-- links -->

[implementing-a-multi-tier-architecture-on-Azure]: ../virtual-machines-windows/n-tier.md

[aad-agent-installation]: /azure/active-directory/active-directory-aadconnect-health-agent-install
[aad-application-proxy]: /azure/active-directory/active-directory-application-proxy-enable
[aad-conditional-access]: /azure/active-directory//active-directory-conditional-access
[aad-connect-sync-default-rules]: /azure/active-directory/active-directory-aadconnectsync-understanding-default-configuration
[aad-connect-sync-operational-tasks]: /azure/active-directory/active-directory-aadconnectsync-operations#staging-mode
[aad-dynamic-memberships]: https://youtu.be/Tdiz2JqCl9Q
[aad-dynamic-membership-rules]: /azure/active-directory/active-directory-accessmanagement-groups-with-advanced-rules
[aad-editions]: /azure/active-directory/active-directory-editions
[aad-filtering]: /azure/active-directory/active-directory-aadconnectsync-configure-filtering
[aad-health]: /azure/active-directory/active-directory-aadconnect-health-sync
[aad-health-adds]: /azure/active-directory/active-directory-aadconnect-health-adds
[aad-health-adfs]: /azure/active-directory/active-directory-aadconnect-health-adfs
[aad-identity-protection]: /azure/active-directory/active-directory-identityprotection
[aad-password-management]: /azure/active-directory/active-directory-passwords-customize
[aad-powershell]: https://msdn.microsoft.com/library/azure/mt757189.aspx
[aad-reporting-guide]: /azure/active-directory/active-directory-reporting-guide
[aad-scalability]: https://blogs.technet.microsoft.com/enterprisemobility/2014/09/02/azure-ad-under-the-hood-of-our-geo-redundant-highly-available-distributed-cloud-directory/
[aad-sync-best-practices]: /azure/active-directory/active-directory-aadconnectsync-best-practices-changing-default-configuration
[aad-sync-disaster-recovery]: /azure/active-directory/active-directory-aadconnectsync-operations#disaster-recovery
[aad-sync-requirements]: /azure/active-directory/active-directory-hybrid-identity-design-considerations-directory-sync-requirements
[aad-topologies]: /azure/active-directory/active-directory-aadconnect-topologies
[aad-user-sign-in]: /azure/active-directory/active-directory-aadconnect-user-signin
[azure-active-directory]: /azure/active-directory-domain-services/active-directory-ds-overview
[azure-ad-connect]: /azure/active-directory/active-directory-aadconnect
[azure-multifactor-authentication]: /azure/multi-factor-authentication/multi-factor-authentication
[considerations]: ./considerations.md
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[sla-aad]: https://azure.microsoft.com/support/legal/sla/active-directory/v1_0/
[visio-download]: https://archcenter.azureedge.net/cdn/identity-architectures.vsdx


[0]: ./images/azure-ad.png "az Azure Active Directoryval identitás architektúra"