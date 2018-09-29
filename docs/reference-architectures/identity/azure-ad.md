---
title: Helyszíni AD-tartományok integrálása az Azure Active Directoryval
description: Biztonságos hibrid hálózati architektúra implementálása az Azure Active Directory használatával.
author: telmosampaio
pnp.series.title: Identity management
ms.date: 11/28/2016
pnp.series.next: adds-extend-domain
pnp.series.prev: ./index
cardTitle: Integrate on-premises AD with Azure AD
ms.openlocfilehash: b5cd4d353c6d6b149c9c1b5547f8da8c25ad5a85
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 09/28/2018
ms.locfileid: "47429230"
---
# <a name="integrate-on-premises-active-directory-domains-with-azure-active-directory"></a>Helyszíni Active Directory-tartományok integrálása az Azure Active Directoryval

Az Azure Active Directory (Azure AD) egy felhőalapú több-bérlős címtár- és identitásszolgáltatás. A referenciaarchitektúra a helyszíni Active Directory-tartományoknak a felhőalapú identitáshitelesítés biztosítása érdekében az Azure AD-vel való integrálására vonatkozó ajánlott eljárásokat jeleníti meg. [**A megoldás üzembe helyezése**.](#deploy-the-solution)

[![0]][0] 

*Töltse le az architektúra [Visio-fájlját][visio-download].*

> [!NOTE]
> Az egyszerűség kedvéért a diagram csak az Azure AD közvetlen kapcsolatait mutatja, valamint a hitelesítés és az identitás-összevonás részeként esetleg felmerülő, nem protokollhoz kötődő forgalmat. Például egy webalkalmazás átirányíthatja a webböngészőt a kérés hitelesítésére az Azure AD-n keresztül. A hitelesítés megtörténtét követően a kérés visszaadható a webalkalmazásnak a megfelelő identitásadatokkal.
> 

A referenciaarchitektúra tipikus alkalmazásai lehetnek:

* Webalkalmazások üzembe helyezése az Azure-ban, amelyek hozzáférést biztosítanak a vállalat távoli felhasználói számára.
* Önkiszolgáló képességek implementálása a végfelhasználók számára, például a jelszavak visszaállítására és a csoportfelügyelet delegálására. Vegye figyelembe, hogy ehhez az Azure AD Premium Edition használata szükséges.
* Olyan architektúrák, amelyekben a helyszíni hálózat és az alkalmazás Azure virtuális hálózata nem VPN-alagúton vagy ExpressRoute-kapcsolatcsoportoton keresztül kapcsolódik.

> [!NOTE]
> Az Azure AD hitelesítheti a felhasználókat és alkalmazásokat, amelyek egy szervezet címtárában található identitását. Egyes alkalmazások és szolgáltatások, például az SQL Server esetében szükség lehet a számítógép hitelesítésére is, ebben az esetben ez a megoldás nem megfelelő.
> 

További szempontok: [Megoldás választása a helyszíni Active Directory Azure-ral való integrálásához][considerations]. 

## <a name="architecture"></a>Architektúra

Az architektúra a következő összetevőket tartalmazza.

* **Azure AD-bérlő**. A vállalat által létrehozott [Azure AD][azure-active-directory]-példány. Ez a helyszíni Active Directoryból másolt objektumok tárolásával címtárszolgáltatásként szolgál a felhőalapú alkalmazásokhoz, valamint identitásszolgáltatásokat biztosít.
* **Alhálózat a webes rétegben**. Ez az alhálózat webalkalmazásokat futtató virtuális gépeket tartalmaz. Az Azure AD szolgálhat identitásközvetítőként ehhez az alkalmazáshoz.
* **Helyszíni AD DS-kiszolgáló**. Egy helyszíni címtár- és identitásszolgáltatás. Az AD FS-címtár szinkronizálható az Azure AD-vel, így az képessé tehető a helyszíni felhasználók hitelesítésére.
* **Azure AD Connect szinkronizálási kiszolgáló**. Az [Azure AD Connect][azure-ad-connect] szinkronizálási szolgáltatást futtató helyszíni számítógép. Ez a szolgáltatás szinkronizálja az Active Directoryban tárolt adatokat az Azure AD-vel. Például ha a helyszíni környezetben kioszt vagy megszüntet csoportokat vagy felhasználókat, ezeket a változásokat az Azure AD-be is propagálja a rendszer. 
  
  > [!NOTE]
  > Biztonsági okokból az Azure AD a felhasználói jelszavakat kivonatként tárolja. Ha a felhasználó jelszavát vissza kell állítani, ezt a helyszíni környezetben kell elvégezni, majd az új kivonatot le kell küldeni az Azure AD-be. Az Azure AD Premium Edition kiadásainak szolgáltatásaival automatizálható ez a feladat, így a felhasználók maguk állíthatják vissza a jelszavaikat.
  > 

* **Virtuális gépek az N szintű alkalmazáshoz**. Ez az üzemelő példány tartalmazza egy N szintű alkalmazás infrastruktúráját. Ezekkel az erőforrásokkal kapcsolatban további információkért lásd: [Virtuális gépek futtatása N szintű architektúrához][implementing-a-multi-tier-architecture-on-Azure].

## <a name="recommendations"></a>Javaslatok

Az alábbi javaslatok a legtöbb forgatókönyvre vonatkoznak. Kövesse ezeket a javaslatokat, ha nincsenek ezeket felülíró követelményei. 

### <a name="azure-ad-connect-sync-service"></a>Azure AD Connect szinkronizálási szolgáltatás

Az Azure AD Connect szinkronizálási szolgáltatás biztosítja, hogy a felhőben tárolt identitásadatok összhangban legyenek a helyszínen tároltakkal. A szolgáltatás az Azure AD Connect szoftver használatával telepíthető. 

Az Azure AD Connect szinkronizálási szolgáltatás implementálása előtt mérje fel a vállalat szinkronizálási követelményeit. Például hogy mit kell szinkronizálni, melyik tartományokból és milyen gyakran. További információkért lásd: [A címtár-szinkronizálási követelmények felmérése][aad-sync-requirements].

Az Azure AD Connect szinkronizálási szolgáltatást futtathatja egy virtuális gépen vagy egy helyszínen üzemeltetett számítógépen is. Az Active Directory-címtárban lévő információk változékonyságától függően az Azure AD Connect szinkronizálási szolgáltatás terhelése az Azure AD első szinkronizálását követően valószínűleg nem lesz túl magas. A szolgáltatás egy virtuális gépen futtatva szükség esetén könnyebben skálázható. A virtuális gép tevékenységének monitorozásával (lásd a Monitorozási szempontok című szakaszt) állapíthatja meg, hogy van-e szükség skálázásra.

Ha több helyszíni tartománnyal rendelkezik egy erdőben, javasoljuk, hogy a teljes erdőre vonatkozó adatokat egyetlen Azure AD-bérlőn tárolja és szinkronizálja. Szűrje a több tartományban is előforduló identitásokra vonatkozó információkat, hogy elkerülje a duplikációt, és minden egyes identitás csak egyszer jelenjen meg az Azure AD-ben. A duplikáció inkonzisztenciához vezethet az adatok szinkronizálásakor. További információkért lásd alább a Topológia című szakaszt. 

Szűréssel biztosítsa, hogy csak a szükséges adatok legyenek az Azure AD-ben tárolva. Lehetséges például, hogy a vállalat az inaktív fiókok adatait nem szeretné az Azure AD-ben tárolni. A szűrés történhet csoport, tartomány, szervezeti egység (OU) vagy attribútum alapján. A szűrők kombinálásával összetettebb szabályok is létrehozhatók. Például szinkronizálhatja egy adott tartománynak csak azokat az adatait, amelyek egy adott értékkel rendelkeznek egy kijelölt attribútumban. Részletes információkért lásd: [Az Azure AD Connect szinkronizálása: a szűrés konfigurálása][aad-filtering].

Az AD Connect szinkronizálási szolgáltatás magas rendelkezésre állásának implementálásához futtasson egy másodlagos átmeneti kiszolgálót. További információért lásd a Topológiai javaslatok című szakaszt.

### <a name="security-recommendations"></a>Biztonsági javaslatok

**Felhasználói jelszókezelés** Az Azure AD Premium Edition kiadásai támogatják a jelszóvisszaírást, így a helyi felhasználók önkiszolgálóan állíthatják vissza jelszavaikat az Azure Portalon. Ezt a szolgáltatást csak a vállalat jelszóbiztonsági szabályzatának áttekintése után engedélyezze. Például korlátozhatja, hogy mely felhasználók módosíthassák a jelszavukat, és testre szabhatja a jelszókezelési folyamatot. További információkért lásd: [A jelszókezelés testreszabása a vállalat igényeinek megfelelően][aad-password-management]. 

**A kívülről elérhető helyszíni alkalmazások védelme.** Az Azure AD-alkalmazásproxy segítségével szabályozott hozzáférést biztosíthat a külső felhasználók számára a helyszíni webalkalmazásokhoz. Az alkalmazást csak azok a felhasználók használhatják, akik érvényes hitelesítő adatokkal rendelkeznek az Azure-címtárban. További információkért lásd a következő cikket: [Alkalmazásproxy engedélyezése az Azure Portalon][aad-application-proxy].

**Az Azure AD aktív monitorozása a gyanús tevékenységek jeleinek észlelésére.**    Vegye fontolóra az Azure AD Premium P2 kiadás használatát, amely tartalmazza az Azure AD Identity Protection szolgáltatást is. Az Identity Protection adaptív gépi tanulási algoritmusok és heurisztika segítségével észleli a rendellenességeket és a kockázati eseményeket, amelyek azt jelezhetik, hogy valamely identitás biztonsága sérült. Például képes észlelni a potenciálisan szokatlan tevékenységeket, például a szabálytalan bejelentkezési tevékenységeket, az ismeretlen forrásból vagy gyanús tevékenységeket mutató IP-címekről származó bejelentkezéseket, vagy az esetleg fertőzött eszközökről való bejelentkezéseket. Az adatok alapján az Identity Protection jelentéseket és riasztásokat hoz létre, amelyek segítségével megvizsgálhatja a kockázati eseményeket, és megteheti a szükséges intézkedéseket. További információkért lásd: [Azure Active Directory Identity Protection][aad-identity-protection].
  
Az Azure Portalon az Azure AD jelentéskészítési szolgáltatásával monitorozhatja a rendszerben előforduló, a biztonsággal kapcsolatos tevékenységeket. A jelentések használatával kapcsolatos további információkért lásd az [Azure Active Directory jelentéskészítési útmutatóját][aad-reporting-guide].

### <a name="topology-recommendations"></a>Topológiai javaslatok

Az Azure AD Connect megfelelő konfigurálásával valósítsa meg a vállalat követelményeihez leginkább illő topológiát. Az Azure AD Connect által támogatott topológiák közé a következők tartoznak:

* **Egyetlen erdő, egyetlen Azure AD-címtár**. Ebben a topológiában az Azure AD Connect az objektumokat és az identitásadatokat egyetlen helyszíni erdő egy vagy több tartományából egyetlen Azure AD-bérlőre szinkronizálja. Ez az Azure AD Connect gyorstelepítés során megvalósított alapértelmezett topológia.
  
  > [!NOTE]
  > Ne használjon több Azure AD Connect szinkronizálási kiszolgálót az egyazon helyszíni erdőben lévő különböző tartományok egyazon Azure AD-bérlőre való csatlakoztatására, hacsak nem egy átmeneti módban futó kiszolgálót futtat (lásd alább).
  > 
  > 

* **Több erdő, egyetlen Azure AD-címtár**. Ebben a topológiában az Azure AD Connect az objektumokat és az identitásadatokat több erdőből egyetlen Azure AD-bérlőre szinkronizálja. Akkor alkalmazza ezt a topológiát, ha a vállalat több helyszíni erdővel rendelkezik. Az identitásadatok konszolidálhatóak, így mindegyik egyedi felhasználói egyszer csak jelenik meg az Azure AD-címtárban, még akkor is, ha ugyanaz a felhasználó több erdőben is szerepel. Mindegyik erdő ugyanazt az Azure AD Connect szinkronizálási kiszolgálót használja. Az Azure AD Connect szinkronizálási kiszolgálónak egyik tartományhoz sem kell tartoznia, de mindegyik erdőből elérhetőnek kell lennie.
  
  > [!NOTE]
  > Ebben a topológiában ne használjon külön Azure AD Connect szinkronizálási kiszolgálókat az egyes helyszíni erdők egy adott Azure AD-bérlőre való csatlakoztatásához. Ez az identitásadatok duplikálását eredményezheti az Azure AD-ben, ha a felhasználók több erdőben is jelen vannak.
  > 
  > 

* **Több erdő, külön topológiák**. Ez a topológia több külön erdőből egyesít identitásadatokat egyetlen Azure AD-bérlőre az egyes erdőket külön entitásokként kezelve. Ez a topológia akkor hasznos, ha több vállalat erdőit egyesíti, és mindegyik felhasználó identitásadatai csak egy erdőben találhatóak meg.
  
  > [!NOTE]
  > Ha az egyes erdők globális címlistái (GAL) szinkronizálva vannak, az egyes erdőkben lévő felhasználók egy másik erdőben kapcsolatként jelen lehetnek. Ez akkor fordulhat elő, ha a vállalat a Forefront Identity Manager 2010 vagy a Microsoft Identity Manager 2016 használatával megvalósította a GALSyncet. Ebben a forgatókönyvben, megadhatja, hogy a felhasználók a *Mail* attribútumukkal legyenek azonosítva. Az identitásokat az *ObjectSID* és az *msExchMasterAccountSID* attribútumok használatával is egyeztetheti. Ez akkor hasznos, ha egy vagy több, letiltott fiókokat tartalmazó erőforráserdővel rendelkezik.
  > 
  > 

* **Átmeneti kiszolgáló**. Ebben a konfigurációban egy második Azure AD Connect szinkronizálási kiszolgáló példányt futtat az elsővel párhuzamosan. Ez a struktúra az alábbiakhoz hasonló forgatókönyveket tesz lehetővé:
  
  * Magas rendelkezésre állás.
  * Az Azure AD Connect szinkronizálási kiszolgáló új konfigurációjának tesztelése és üzembe helyezése.
  * Új kiszolgáló bevezetése és a régi konfiguráció leszerelése. 
    
    Ezekben a forgatókönyvekben a második példány *átmeneti módban* fut. A kiszolgáló rögzíti az importált objektumokat és a szinkronizálási adatokat az adatbázisában, de nem továbbítja azokat az Azure AD szolgáltatásba. Ha letiltja az átmeneti módot, a kiszolgáló elkezdi az adatokat az Azure AD-be írni, valamint visszaírni a jelszavakat a helyszíni címtárakba (ahol alkalmazható). További információkért lásd: [Az Azure AD Connect szinkronizálása: üzemeltetési feladatok és szempontok][aad-connect-sync-operational-tasks].

* **Több Azure AD-címtár**. Egy vállalathoz egyetlen Azure AD-címtárat javasolt létrehozni, de előfordulhatnak olyan esetek, amikor az adatokat külön Azure AD-címtárakba kell particionálni. Ebben az esetben a szinkronizálási és jelszó-visszaírási problémák elkerülése érdekében gondoskodjon róla, hogy a helyi erdő minden objektuma csak egyetlen Azure AD-címtárban jelenjen meg. A forgatókönyv implementálásához konfiguráljon külön Azure AD Connect szinkronizálási kiszolgálót minden egyes Azure AD-címtárhoz, és szűrés segítségével biztosítsa, hogy az egyes Azure AD Connect szinkronizálási kiszolgálók egymást kölcsönösen kizáró objektumhalmazokon működjenek. 

További információk ezekről a topológiákról: [Azure AD Connect-topológiák][aad-topologies].

### <a name="user-authentication"></a>Felhasználóhitelesítés

Alapértelmezés szerint az Azure AD Connect szinkronizálási kiszolgáló a Jelszókivonat-szinkronizálás a helyszíni tartomány és az Azure AD között állítja be, és az Azure AD szolgáltatás feltételezi, hogy felhasználók a hitelesítéshez ugyanazt a jelszót, hogy a helyszíni használnak. A legtöbb vállalat számára ez megfelelő, de érdemes figyelembe venni a vállalat meglévő szabályzatait és infrastruktúráját. Példa:

* A szervezete biztonsági házirendjével megtilthatja a jelszókivonatok felhőbe. Ebben az esetben fontolja meg a szervezet [átmenő hitelesítés](/azure/active-directory/connect/active-directory-aadconnect-pass-through-authentication).
* Esetleg az lehet az elvárás, hogy a felhasználók a felhőerőforrásokat a vállalati hálózaton lévő, tartományra csatlakozott gépekről zökkenőmentes egyszeri bejelentkezéssel (SSO) érhessék el.
* A vállalatban esetleg már be vannak vezetve az Active Directory összevonási szolgáltatások (AD FS) vagy valamely más, harmadik féltől származó összevonási szolgáltató. Az Azure AD megfelelő konfigurálásával ez az infrastruktúra beállítható a felhőben tárolt jelszóadatok helyett a hitelesítés és az egyszeri bejelentkezés használatára.

További információkért lásd: [Az Azure AD Connect felhasználói bejelentkezési beállításai][aad-user-sign-in].

### <a name="azure-ad-application-proxy"></a>Azure AD-alkalmazásproxy 

Az Azure AD használatával biztosíthatja a helyszíni alkalmazások elérését.

A helyszíni webalkalmazásokat megnyithatja az Azure AD-alkalmazásproxy összetevő által kezelt alkalmazásproxy-összekötők használatával. Az alkalmazásproxy-összekötő kimenő hálózati kapcsolatot nyit az Azure AD-alkalmazásproxyhoz, és a távoli felhasználók kéréseit a rendszer az Azure AD-ből ezen a kapcsolaton keresztül irányítja vissza a webalkalmazásokba. Így nem szükséges helyszíni bemenő portokat nyitni a helyszíni tűzfalon, és csökken a vállalat támadási felülete.

További információkért lásd: [Alkalmazások közzététele az Azure AD-alkalmazásproxyval][aad-application-proxy].

### <a name="object-synchronization"></a>Objektumszinkronizáció 

Az Azure AD Connect alapértelmezett konfigurációja a következő cikkben meghatározott szabályok alapján szinkronizálja a helyi Active Directory-címtár objektumait: [Az Azure AD Connect szinkronizálása: az alapértelmezett konfiguráció ismertetése][ aad-connect-sync-default-rules]. A szabályoknak megfelelő objektumok szinkronizálva lesznek, a többi objektum figyelmen kívül lesz hagyva. Néhány példa szabály:

* A felhasználói objektumoknak rendelkezniük kell egy egyedi *sourceAnchor* attribútummal, és az *accountEnabled* attribútumnak ki kell lennie töltve.
* A felhasználói objektumoknak rendelkezniük kell egy *sAMAccountName* attribútummal, és nem kezdődhetnek az *Azure AD_* vagy *MSOL_* szövegrésszel.

Az Azure AD Connect több szabályt is alkalmaz a felhasználói, kapcsolattartói, csoport-, ForeignSecurityPrincipal és számítógép-objektumokra. Szükség esetén az Azure AD Connecttel telepített szinkronizálási szabályszerkesztővel módosíthatja az alapértelmezett szabálykészletet. További információért lásd: [Az Azure AD Connect szinkronizálása: az alapértelmezett konfiguráció ismertetése][aad-connect-sync-default-rules].

Saját szűrők megadásával korlátozhatja a tartomány vagy szervezeti egység által szinkronizálandó objektumok mennyiségét. Másik megoldásként alkalmazhat összetettebb egyéni szűrőket is, amilyenek például a következő szakaszban vannak leírva: [Az Azure AD Connect szinkronizálása: a szűrés konfigurálása][aad-filtering].

### <a name="monitoring"></a>Figyelés 

Az állapotmonitorozást a következő, helyszínen telepített ügynökök végzik:

* Az Azure AD Connect telepít egy ügynököt, amely a szinkronizálási műveletekre vonatkozó adatokat rögzíti. Az Azure Portalon az Azure AD Connect Health panelen monitorozhatja az állapotát és a teljesítményét. További információkért lásd: [Az Azure AD Connect Health szinkronizálási szolgáltatás használata][aad-health].
* Az AD DS-tartományok és -címtárak Azure-ból való monitorozásához telepítse az Azure AD Connect Health for AD DS-ügynököt egy, a helyszíni tartományban lévő gépen. Az Azure Portal Azure Active Directory Connect Health panele használatával monitorozható az állapot. További információkért lásd: [Az Azure AD Connect Health használata az AD DS szolgáltatással][aad-health-adds] 
* Telepítse az Azure AD Connect Health for AD FS-ügynököt a helyszínen futó szolgáltatások állapotának monitorozására, és az Azure Portal Azure Active Directory Connect Health panelén monitorozhatja az AD FS-t. További információkért lásd: [Az Azure AD Connect Health használata az AD FS szolgáltatással][aad-health-adfs]

Az AD Connect Health-ügynökök telepítésével és a vonatkozó követelményekkel kapcsolatos további információkért lásd: [Az Azure AD Connect Health-ügynök telepítése][aad-agent-installation].

## <a name="scalability-considerations"></a>Méretezési szempontok

Az Azure AD szolgáltatás támogatja a replikaalapú skálázást, egy az írási műveleteket kezelő elsődleges replikával és több csak olvasható másodlagos replikával. Az Azure AD transzparens módon átirányítja a másodlagos replikákra irányuló írási kísérleteket az elsődleges replikára, majd biztosítja a végleges konzisztenciát. Az elsődleges replikán végrehajtott valamennyi módosítás propagálva lesz a másodlagos replikákra. Ez az architektúra jól skálázható, mivel az Azure AD-re irányuló műveletek nagy része olvasási, nem pedig írási művelet. További információkért lásd: [Azure AD: a georedundáns, magas rendelkezésre állású, elosztott felhőcímtárunk technikai részletei][aad-scalability].

Az Azure AD Connect szinkronizálási kiszolgáló esetében határozza meg, hogy várhatóan hány objektumot fog szinkronizálni a helyi címtárból. Ha az objektumok száma kevesebb mint 100 000, használhatja az Azure AD Connecttel biztosított alapértelmezett SQL Server Express LocalDB szoftvert. Nagyobb mennyiségű objektum esetén telepítse az SQL-kiszolgáló éles verzióját, és végezze el az Azure AD Connect egyéni telepítését, amely során megadja, hogy annak az SQL Server egy meglévő példányát kell használnia.

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

Az Azure AD szolgáltatás földrajzilag elosztott, és több, a világ különböző pontjain lévő adatközpontban fut automatikus feladatátvétellel. Ha valamelyik adatközpont elérhetetlenné válik, az Azure AD biztosítja, hogy a címtáradatok a példányok eléréséhez legalább két, regionálisan elosztott adatközpontban rendelkezésre álljanak.

> [!NOTE]
> Az Azure AD alapszintű és prémium szolgáltatások szolgáltatói szerződése (SLA) legalább 99,9%-os rendelkezésre állást garantál. Az Azure AD ingyenes szintjéhez nem tartozik SLA. További információt az [Azure Active Directory szolgáltatói szerződést][sla-aad] ismertető szakaszban talál.
> 
> 

Fontolja meg az Azure AD Connect szinkronizálási kiszolgáló egy második példányának átmeneti módú kiépítését a rendelkezésre állás növelése érdekében, amint azt a Topológiai javaslatok című szakasz ismerteti. 

Ha nem az Azure AD Connecthez tartozó SQL Server Express LocalDB-példányt használja, vegye fontolóra az SQL-fürtözés alkalmazását a magas rendelkezésre állás eléréséhez. Az Azure AD Connect a tükrözést, az Always Ont és a hasonló megoldásokat nem támogatja.

Az Azure AD Connect szinkronizálási kiszolgáló magas rendelkezésre állásának biztosításával, valamint a meghibásodások utáni helyreállítással kapcsolatos további szempontokat lásd: [Azure AD Connect-szinkronizálás: üzemeltetési feladatok és szempontok – vészhelyreállítás][aad-sync-disaster-recovery].

## <a name="manageability-considerations"></a>Felügyeleti szempontok

Az Azure AD kezelése kapcsán két szempontot kell figyelembe venni:

* Az Azure AD felügyelete a felhőben.
* Az Azure AD Connect szinkronizálási kiszolgálók karbantartása.

Az Azure AD a következő lehetőségeket kínálja a tartományok és a címtárak kezeléséhez a felhőben: 

* **Azure Active Directory PowerShell-modul**. Akkor használja ezt a [modult][aad-powershell], ha általános Azure AD felügyeleti feladatokat kell szkriptelnie, például felhasználó- és tartománykezelést, valamint az egyszeri bejelentkezés konfigurálását.
* **Az Azure Portal Azure AD-felügyelet panele**. Ez a panel a címtár interaktív felügyeleti nézetét mutatja, és lehetővé teszi az Azure AD legtöbb funkciójának vezérlését és konfigurálását. 

Az Azure AD Connect a következő eszközöket telepíti a helyszíni gépekről induló Azure AD Connect szinkronizálási szolgáltatások karbantartásához:
  
* **Microsoft Azure Active Directory Connect-konzol**. Ezzel az eszközzel módosíthatja az Azure AD Sync-kiszolgáló konfigurációját, testre szabhatja a szinkronizálás menetét, engedélyezheti vagy letilthatja az átmeneti módot, és átállíthatja a felhasználói bejelentkezési módot. Engedélyezheti, hogy az Active Directory FS-bejelentkezés a helyszíni infrastruktúra használatával történjen.
* **Szinkronizálási szolgáltatáskezelő**. Az eszköz *Műveletek* lapján felügyelheti a szinkronizálás folyamatát, és észlelheti, ha a folyamat bármely része meghiúsult. Az eszköz használatával manuálisan indíthatja a szinkronizálást. Az *Összekötők* lapon felügyelheti a tartományok összekötőit, amelyekhez a szinkronizáló vezérlő csatlakozik.
* **Szinkronizálási szabályszerkesztő**. Az eszköz segítségével testre szabhatja az objektumok átalakításának módját azok másolása során a helyszíni címtár és az Azure AD között. Az eszköz segítségével további attribútumokat és objektumokat adhat meg a szinkronizáláshoz, majd szűrők alkalmazásával határozhatja meg, hogy mely objektumok legyenek vagy ne legyenek szinkronizálva. További információkért lásd [Az Azure AD Connect szinkronizálása: az alapértelmezett konfiguráció ismertetése][aad-connect-sync-default-rules] című dokumentum Szinkronizálási szabályszerkesztő című szakaszát.

Az Azure AD Connect kezelésével kapcsolatos további információkért és tippekért lásd: [Az Azure AD Connect szinkronizálása: ajánlott eljárások az alapértelmezett konfiguráció módosításához][aad-sync-best-practices].

## <a name="security-considerations"></a>Biztonsági szempontok

Feltételes hozzáférés-vezérlés használatával utasítsa el a váratlan forrásokból származó hitelesítési kéréseket:

- Aktiválja az [Azure Multi-Factor Authentication (MFA)][azure-multifactor-authentication] többtényezős hitelesítést, ha egy felhasználó egy nem megbízható helyről, például az interneten keresztül próbál meg kapcsolódni a megbízható hálózat helyett.

- A felhasználó eszközének platformtípusa (iOS, Android, Windows Mobile, Windows) alapján határozza meg az alkalmazásokra és szolgáltatásokra vonatkozó hozzáférési szabályzatot.

- Rögzítse a felhasználói eszközök engedélyezett/letiltott állapotát, és építse be ezeket az információkat is a hozzáférési szabályzat ellenőrzéseibe. Például ha a felhasználó telefonja elveszett vagy ellopták, azt letiltottként kell rögzíteni, hogy ne lehessen a hozzáféréshez felhasználni.

- Szabályozza a felhasználók erőforrásokhoz való hozzáférését csoporttagság alapján. [Az Azure AD dinamikus tagsági szabályainak][aad-dynamic-membership-rules] segítségével egyszerűsítheti a csoportok adminisztrációját. Ennek rövid áttekintéséért lásd: [Bevezetés a dinamikus csoporttagságba][aad-dynamic-memberships].

- Az Azure AD Identity Protection feltételes hozzáférési kockázati szabályzatainak használatával gondoskodjon megfelelő védelemről a szokatlan bejelentkezési tevékenységek és egyéb események figyelésével.

További információkért lásd: [Azure Active Directory feltételes hozzáférés][aad-conditional-access].

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

Az ezeknek a javaslatoknak és szempontoknak a figyelembe vételével megvalósított referenciaarchitektúra egy üzemelő példánya elérhető a GitHubon. Ez a referenciaarchitektúra üzembe helyez egy szimulált helyszíni hálózatot az Azure-ban, használhatja a teszteléshez és kísérletezéshez. A referenciaarchitektúra Windows vagy Linux rendszerű virtuális gépeken helyezhető üzembe az alábbi utasításokat követve: 

1. Kattintson az alábbi gombra:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Freference-architectures%2Fmaster%2Fidentity%2Fazure-ad%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. Ha a hivatkozás megnyílt az Azure Portalon, meg kell adnia néhány beállítás értékét: 
   * Az **Erőforráscsoport** neve már meg van adva a paraméterfájlban, ezért válassza az **Új létrehozása** lehetőséget és a szövegmezőbe írja az `ra-aad-onpremise-rg` karakterláncot.
   * Válassza ki a régiót a **Hely** legördülő listából.
   * Ne szerkessze a **Sablon gyökér szintű URI-je** vagy a **Paraméter gyökér szintű URI-je** szövegmezőt.
   * Az **Operációs rendszer típusa** legördülő listából válassza a **Windows** vagy a **Linux** lehetőséget.
   * Tekintse át a használati feltételeket, majd kattintson az **Elfogadom a fenti feltételeket** lehetőségre.
   * Kattintson a **Vásárlás** gombra.
3. Várjon, amíg az üzembe helyezés befejeződik.
4. A paraméterfájlokban előre megadott rendszergazdai felhasználónevek és jelszavak szerepelnek. Erősen ajánlott ezeket az összes virtuális gép esetében azonnal lecserélni. Kattintson az Azure Portalon az egyes virtuális gépekre, majd a **Támogatás + hibaelhárítás** panel **Jelszó alaphelyzetbe állítása** elemére. A **Mód** legördülő listában válassza a **Jelszó alaphelyzetbe állítása** lehetőséget, majd adjon meg új értéket a **Felhasználónév** és a **Jelszó** mezőben. Az új felhasználó nevének és jelszavának megőrzéséhez kattintson a **Frissítés** gombra.

<!-- links -->

[implementing-a-multi-tier-architecture-on-Azure]: ../virtual-machines-windows/n-tier.md

[aad-agent-installation]: /azure/active-directory/active-directory-aadconnect-health-agent-install
[aad-application-proxy]: /azure/active-directory/active-directory-application-proxy-enable
[aad-conditional-access]: /azure/active-directory//active-directory-conditional-access
[aad-connect-sync-default-rules]: /azure/active-directory/hybrid/concept-azure-ad-connect-sync-default-configuration
[aad-connect-sync-operational-tasks]: /azure/active-directory/hybrid/how-to-connect-sync-operations
[aad-dynamic-memberships]: https://youtu.be/Tdiz2JqCl9Q
[aad-dynamic-membership-rules]: /azure/active-directory/active-directory-accessmanagement-groups-with-advanced-rules
[aad-editions]: /azure/active-directory/active-directory-editions
[aad-filtering]: /azure/active-directory/hybrid/how-to-connect-sync-configure-filtering
[aad-health]: /azure/active-directory/active-directory-aadconnect-health-sync
[aad-health-adds]: /azure/active-directory/active-directory-aadconnect-health-adds
[aad-health-adfs]: /azure/active-directory/active-directory-aadconnect-health-adfs
[aad-identity-protection]: /azure/active-directory/active-directory-identityprotection
[aad-password-management]: /azure/active-directory/active-directory-passwords-customize
[aad-powershell]: https://msdn.microsoft.com/library/azure/mt757189.aspx
[aad-reporting-guide]: /azure/active-directory/active-directory-reporting-guide
[aad-scalability]: https://blogs.technet.microsoft.com/enterprisemobility/2014/09/02/azure-ad-under-the-hood-of-our-geo-redundant-highly-available-distributed-cloud-directory/
[aad-sync-best-practices]: /azure/active-directory/hybrid/how-to-connect-sync-best-practices-changing-default-configuration
[aad-sync-disaster-recovery]: /azure/active-directory/hybrid/how-to-connect-sync-operations#disaster-recovery
[aad-sync-requirements]: /azure/active-directory/active-directory-hybrid-identity-design-considerations-directory-sync-requirements
[aad-topologies]: /azure/active-directory/hybrid/plan-connect-topologies
[aad-user-sign-in]: /azure/active-directory/hybrid/plan-connect-user-signin
[azure-active-directory]: /azure/active-directory-domain-services/active-directory-ds-overview
[azure-ad-connect]: /azure/active-directory/hybrid/whatis-hybrid-identity
[azure-multifactor-authentication]: /azure/multi-factor-authentication/multi-factor-authentication
[considerations]: ./considerations.md
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[sla-aad]: https://azure.microsoft.com/support/legal/sla/active-directory/v1_0/
[visio-download]: https://archcenter.blob.core.windows.net/cdn/identity-architectures.vsdx


[0]: ./images/azure-ad.png "Felhőbeli identitásarchitektúra az Azure Active Directory használatával"
