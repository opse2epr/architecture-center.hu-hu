---
title: Ajánlott eljárások Azure-bA nagyvállalatok számára
description: A vállalatok számára is annak biztosítására, biztonságos és kezelhető környezetet használó scaffold ismerteti.
author: rdendtler
ms.author: rodend
ms.date: 9/22/2018
ms.openlocfilehash: 66af73f5bfc7f7145c20446af05f33a9d69e6c28
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/04/2019
ms.locfileid: "54011735"
---
# <a name="azure-enterprise-scaffold-prescriptive-subscription-governance"></a>Az Azure enterprise scaffold: Előíró előfizetés-irányítás

Vállalatok egyre inkább vannak bevezetése a nyilvános felhő rugalmasságát és rugalmasságot biztosít. A felhő erősségeit bevételi lehetőségeket, és a vállalati erőforrás-használat optimalizálása azok használatára. Microsoft Azure lehetőséget kínál számos szolgáltatásokat és képességeket, hogy a vállalatok számára állítsa össze a például a számítási feladatok és alkalmazások széles választékának cím építőelemeket.

A Microsoft Azure használata mellett döntenek csak az első lépés a felhő előnyeinek eléréséhez. A második lépés van megértése, hogyan a vállalat hatékonyan használható az Azure és azonosíthatja a referenciakonfiguráció-képességeket, amelyek hasonló kérdések cím teljesülniük kell:

* "Vagyok fontos szempont az adatok elkülönítése; Hogyan tudok biztosíthatja, hogy saját adatok és rendszerek megfelelnek-e a szabályozási követelményeknek?"
* "Hogyan tudom mi minden erőforrás támogatja az így I számlája, és pontosan vissza számláját?"
* "Szeretnék győződjön meg arról, hogy minden Microsoft üzembe helyezése, vagy hajtsa végre a nyilvános felhőben elindítja a biztonsági így először Hogyan tudok segíti, hogy megkönnyítsék?"

A potenciális a nincs guard rails-üres előfizetés tűnhet. Az üres helyet a áthelyezése az Azure-ba is akadályozhatják.

Ez a cikk kiindulási pontot kell irányítási és kiegyensúlyozása, agilitást és műszaki szakembereknek szánt biztosít. Ez bemutatja a, az enterprise scaffold, amely végigvezeti a szervezetek megvalósítása és kezelése az Azure-környezetek biztonságos módon. A keretrendszer leghatékonyabb vezérlők fejlesztéséhez nyújt.

## <a name="need-for-governance"></a>Cégirányítási szükség

Az Azure-ba történő áthelyezésekor kell venni a cégirányítási elején annak biztosítása érdekében a vállalaton belül a felhő sikeres használatát, a témakörben. Sajnos az idő és a egy átfogó cégirányítási rendszer létrehozásának bürokráciával azt jelenti, hogy egyes üzleti csoportok lépjen közvetlenül a szolgáltatónak vállalati informatikai közreműködése nélkül. Ez a megközelítés a vállalati nyitva hagyhatja megtámadására, ha az erőforrások megfelelően nem felügyelt. A nyilvános felhőben – rugalmasságot, a rugalmasságot és fogyasztásalapú díjszabás – mutatókat fontos üzleti csoportokat szeretne gyorsan teljesítheti az ügyfelek (belső és külső) igényeinek megfelelően. De vállalati adatok és rendszerek hatékonyan védelmének biztosításához szükséges.

Épület létrehozásakor szerkezetkialakító segítségével hozzon létre egy struktúra alapját. A scaffold végigvezeti az általános elvet követik, és állandó rendszerek csatlakoztatnia kell a forráshorgony pontokat biztosít. Egy enterprise scaffold ugyanez: rugalmas vezérlők és az Azure-képességek a környezetben, és a központi jellegűek struktúrát biztosító a nyilvános felhő alapú szolgáltatások. A kapcsolat építői biztosít (informatikai és üzleti csoportok) létrehozásához és csatlakoztatásához az új szolgáltatások gyors kézbesítés szem alapját.

A scaffold tudjuk a különböző fürtméretekkel járó az ügyfelek számos marketingmódszerek kigyűjtötte tanácsok alapul. Ezek az ügyfelek köre a kisebb szervezetek natív felhőalapú megoldások fejlesztésével és a felhőben is nagyobb a multinacionális cégeknek, és független szoftverszállítók számítási feladatokhoz való-megoldások fejlesztése. Az enterprise scaffold "célirányosan" rugalmas hagyományos informatikai számítási feladatok és a rugalmas számítási feladatok; támogatásához például a szoftver--szolgáltatásként (SaaS) alkalmazásokat létrehozó fejlesztők az Azure platform képességei alapján.

Az enterprise scaffold minden új előfizetés Azure-ban alapjául szolgál. Ez lehetővé teszi, hogy a rendszergazdáival együttműködve biztosítják a számítási feladatok megfelelnek a minimális cégirányítási követelmények a szervezet meggátolja, hogy üzleti csoportokat vagy a fejlesztők gyorsan felel meg a saját céljainak nélkül. Az a tapasztalat, hogy ez jelentősen lerövidíti ahelyett, hogy akadályozza a nyilvános felhőben növekedési.

> [!NOTE]
> A Microsoft közzétette az előzetes verzió egy új funkció nevű [Azure tervezetek](/azure/governance/blueprints/overview) , amely lehetővé teszi, hogy a csomag, kezelése és üzembe helyezése a gyakori rendszerképek, sablonok, szabályzatok és parancsfájlok között, előfizetések és a felügyeleti csoportokat. Ez a funkció akkor a híd mintamodell a scaffold célját és helyezi üzembe a modellt, a szervezet között.
>
Az alábbi képen a scaffold összetevői láthatók. A foundation egy alapos terv, a kezelési hierarchia és az előfizetések támaszkodik. A területei legyenek elérhetők a Resource Manager-házirendek és erős elnevezési szabványait áll. A scaffold a többi Azure-szolgáltatások és szolgáltatások engedélyezése és a egy biztonságos és kezelhető környezetet hoz vannak maghoz.

![Enterprise scaffold](./_images/scaffoldv2.png)

## <a name="define-your-hierarchy"></a>Adja meg a hierarchiában

A scaffold alapját a hierarchia és az Azure nagyvállalati beléptetés, keresztül előfizetések és erőforráscsoportok közötti kapcsolat. A nagyvállalati beléptetés határozza meg, a alakzat és a egy szerződéses szempontból vállalaton belüli Azure-szolgáltatások használatát. A nagyvállalati szerződés keretében további feloszthatja a részlegek, fiókok, a környezetét, és végül az előfizetések és az erőforrás csoportok, a szervezet szerkezetével.

![hierarchia](./_images/agreement.png)

Azure-előfizetéssel az alapszintű egység, hol található összes erőforrást. Azure-magok, virtuális hálózatok és egyéb erőforrások száma például számos korlátait is meghatározza. Az Azure Resource Groups segítségével tovább finomíthatja az előfizetésen és erőforrások több természetes csoportosítása engedélyezése.

Minden vállalat különböző, és a fenti ábrán a hierarchia lehetővé teszi, hogy jelentős rugalmasságot nyújt, az Azure hogyan vannak rendszerezve a vállalaton belül. Modellezés, a hierarchiában, hogy a számlázási, erőforrás-kezelés a vállalat igényeinek megfelelően, és erőforrás-hozzáférés az első &mdash; és legfontosabb &mdash; döntési választja ki a nyilvános felhőben indításakor.

### <a name="departments-and-accounts"></a>Szervezeti egységek és fiókok

Az Azure-regisztrációk három gyakori minta a következők:

* A **működési** minta

    ![funkcionális](./_images/functional.png)
* A **üzleti egység** minta

    ![vállalata számára](./_images/business.png)
* A **földrajzi** minta

    ![földrajzi](./_images/geographic.png)

Bár ezek a minták mindegyikén az helyén a **üzleti egység** minta egyre kik a szervezet modellező rugalmasságot biztosít a költség, modell, valamint a vezérlő span tükröző. A Microsoft Core mérnöki és a műveleti csapat által létrehozott egy adatforrásbeli, a **üzleti egység** mintát, amely nagyon hatékony modellezve **Szövetségi**, **állapot**, és  **Helyi**. (További információkért lásd: [előfizetésekhez és erőforráscsoportokhoz a vállalaton belüli rendszerezése](https://azure.microsoft.com/blog/organizing-subscriptions-and-resource-groups-within-the-enterprise/).)

### <a name="management-groups"></a>Felügyeleti csoportok

A Microsoft nemrég kiadott egy új lehetőség a modellezés, a hierarchiában: [Az Azure felügyeleti csoportok](/azure/azure-resource-manager/management-groups-overview). Felügyeleti csoportok jóval rugalmasabb, mint a szervezeti egységek és a fiókok és ágyazhatók be legfeljebb hat szintje. Felügyeleti csoportok lehetővé teszik, hogy ne a számlázási hierarchiában, kizárólag az erőforrások hatékony kezelését a hierarchia létrehozása. Felügyeleti csoportok is tükrözi a számlázási hierarchia, és gyakran vállalatok kezdje ezzel a módszerrel. Azonban a felügyeleti csoportok a power akkor, ha azokat használja a szervezet modell ahol kapcsolódó előfizetések &mdash; függetlenül, ahol a számlázási hierarchia &mdash; vannak csoportosítva, és közös szerepkörrel kell, valamint szabályzatok és kezdeményezések. Néhány példa:

* **Éles környezetben, illetve nem éles**. Egyes vállalatok hozzon létre felügyeleti csoportot, és azok éles környezetben, és nem éles üzemű előfizetéseket. Felügyeleti csoportok lehetővé teszik, hogy ezeket az ügyfeleket több könnyedén kezelheti a szerepköröket és szabályzatok, például: nem éles előfizetés előfordulhat, hogy lehetővé teszi a fejlesztők "közreműködői" hozzáféréssel, de az éles környezetben csak az "olvasó" hozzáféréssel rendelkeznek.
* **Belső szolgáltatások és a külső szolgáltatások**. Sokkal éles/nem éles üzemi, például a vállalatok gyakran rendelkeznek eltérő követelmények vonatkoznak, a házirendek és a szerepkörök szolgáltatás belső és külső (az ügyfelek által használt) szolgáltatási.

Úgy Gondoltuk, felügyeleti csoportok jól, kezdeményezések mellett az Azure Policy az Azure hatékony cégirányítási gerincét.

### <a name="subscriptions"></a>Előfizetések

Amikor eldönti, a szervezeti fiókok (vagy egy felügyeleti csoportok), elsősorban helyzet hogyan, az Azure-környezet a szervezet megfelelő elosztására használ. Előfizetések, azonban akkor is, ahol a tényleges munka történik, és itt a döntések befolyásolja a biztonsággal, méretezhetőséggel és a számlázás.  Számos szervezetben az útmutatókat, tekintse meg a következő minták:

* **Alkalmazás/szolgáltatás**: Előfizetések képviseli egy alkalmazás vagy szolgáltatás (alkalmazások portfólióját)
* **Életciklus**: Előfizetések képviseli egy olyan szolgáltatás, például éles vagy fejlesztői életciklusát.
* **Részleg**: Előfizetések képviselik a szervezetben lévő részlegek számára.

Az első két mintákat a leggyakrabban használt, és mindkettő erősen ajánlott. Az életciklus módszer a legtöbb szervezet számára megfelelő. Ebben az esetben az általános ajánlás, hogy két alapszintű előfizetés. "Éles" és "Nem éles üzemi", majd az erőforráscsoportokat a kezdetét vette az a környezetben további.

### <a name="resource-groups"></a>Erőforráscsoportok

Az Azure Resource Manager lehetővé teszi, hogy a felügyeleti, számlázási vagy természeti affinitásra jelentéssel bíró csoportokba erőforrásokat helyezze. Erőforráscsoportok olyan tárolók, az erőforrások, amelyek egy közös életciklusának vagy megosztani egy attribútum, például a "minden SQL-kiszolgáló" vagy "Egy alkalmazás".

Erőforráscsoportok nem ágyazható be, és az erőforrások csak egy erőforráscsoporthoz is tartozhatnak. Minden erőforrás egy erőforráscsoportba tartozó műveleteket hajthat végre bizonyos műveleteket. Például egy erőforráscsoport törlése eltávolítja az összes erőforrást az erőforráscsoporton belül. Az előfizetések, például vannak közös minták erőforráscsoportok létrehozásakor és változnak "Hagyományos informatikai" számítási "Agilis informatikai" számítási feladatokhoz:

* "A hagyományos informatikai" számítási feladatok leggyakrabban szerint vannak csoportosítva elemek belül az azonos életciklus, például egy alkalmazást. Alkalmazás által a csoportosítás lehetővé teszi egyéni alkalmazás felügyeletét.
* "Agilis informatikai" számítási feladatok általában a külső ügyfelek által használt felhőalkalmazások összpontosíthat. Az erőforráscsoportok gyakran tükrözik a rétegek (például a webes szint, App-réteget) központi telepítés és felügyelet.

> [!NOTE]
> A számítási feladatok ismertetése segíti erőforrás-csoport stratégia. Ezeknek a mintáknak is vegyes és egyezést. Például egy megosztott szolgáltatások erőforráscsoport "Agile" erőforráscsoportok, ugyanabban az előfizetésben.

## <a name="naming-standards"></a>Elnevezési szabályai

Az első oszlop a scaffold, egy egységes elnevezési szabványnak. Jól megtervezett elnevezési szabványait lehetővé teszi a portálon, a számlán, és parancsfájlok belüli erőforrások azonosítására. Valószínűleg már rendelkezik a helyszíni infrastruktúrára vonatkozó meglévő elnevezési szabványait. Azure ad környezetében, amikor az Azure-erőforrások, ki kell elnevezési szabványokat.

> [!TIP]
> Az elnevezési konvenciók:
> * Tekintse át, és ahol lehetséges elfogadja a [Patterns and practices nevű útmutató](https://docs.microsoft.com/en-us/azure/architecture/best-practices/naming-conventions). Ez az útmutató segítségével könnyebben meghatározhatja az egy jelentéssel bíró elnevezési szabványnak, és a széles körű példákat talál.
> * Resource Manager-házirendek használatával segít, hogy elnevezési szabályai
>
>Ne feledje, hogy nehéz később módosítható a neve, most tehát néhány percig fog mentse, problémajegyek később.

Ezek az erőforrások több gyakran használt és keresni az alábbiakra koncentráljon az elnevezési szabványait.  Minden erőforráscsoport például érdemes követnie az egyértelműség érdekében erős szabvány.

### <a name="resource-tags"></a>Erőforráscímkék

Erőforráscímkék szorosan igazítva elnevezési szabványoknak. Erőforrásokat ad az előfizetések, ahogy azt a logikailag kategorizálhatja a számlázás, felügyeleti és üzemeltetési célból egyre fontosabbá válik. További információkért lásd: [címkék használata az Azure-erőforrás rendszerezéséhez](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-using-tags).

> [!IMPORTANT]
> Címkék személyes információkat is tartalmazhat, és a GDPR-szabályzat alá tartozhatnak. Gondosan tervezze meg a címkék kezelése. GDPR kapcsolatos általános információkat keres, ha további tájékoztatást az általános adatvédelmi rendelet a [Szolgáltatásmegbízhatósági portálon](https://servicetrust.microsoft.com/ViewPage/GDPRGetStarted).

A címkék a felügyeleti és számlázási túl sok szempontból használnak. Ezeket gyakran használják az automation részeként (lásd a későbbi szakasz ismerteti). Ez ütközéseket okozhat, ha nem veszi figyelembe meghozni. Az ajánlott eljárás, hogy határozza meg az összes gyakori címkéket (például ApplicationOwner, CostCenter) vállalati szintű, és egységesen alkalmazhatja őket az automation-erőforrások üzembe helyezése.

## <a name="azure-policy-and-initiatives"></a>Az Azure Policy és kezdeményezések

A második pillar, a scaffold keretein belül a [Azure Policy és kezdeményezések](/azure/azure-policy/azure-policy-introduction) kockázat kezelése (a hatások) szabályok tartat be az erőforrásokat és szolgáltatásokat az előfizetésekben keresztül. Az Azure kezdeményezések házirendeket, amelyek célja, hogy egyetlen cél elérése gyűjteményei. Az Azure policy és kezdeményezések majd hozzárendelt erőforrás hatókört az egyes házirendek kényszerítését a kezdéshez.

Azure házirend- és kezdeményezések még hatékonyabbak lehetnek, a felügyeleti csoportok a korábban említett együtt használva. Felügyeleti csoportok engedélyezése a hozzárendelését egy kezdeményezést vagy előfizetések teljes készletéhez.

### <a name="common-uses-of-resource-manager-policies"></a>Gyakori használati Resource Manager-házirendek

Az Azure-házirendek és kezdeményezések az Azure-eszközkészlet egy hatékony eszköz. Szabályzatok lehetővé teszik a cégeket azzal, hogy szabályozza a "Hagyományos informatikai" számítási feladatokhoz, amelyek lehetővé teszik a stabilitását, miközben is lehetővé teszi a munkaterhelések "Agile"; szükséges LOB-alkalmazások például anélkül, hogy a vállalatok számára, további kockázati megnyitását vevő alkalmazások fejlesztéséhez használható. A leggyakrabban használt minták házirendek láthatjuk a következők:

* **GEO-megfelelőségi/adatszuverenitás**. Az Azure-régiók – folyamatosan bővülő listájának rendelkezik világszerte. Vállalatok gyakran kell biztosítja, hogy egy adott hatókörben található erőforrások szabályozási követelmények teljesítésére egy földrajzi régióban maradjanak.
* **Kerülje az adatokhoz hozzáférést biztosító kiszolgálók nyilvánosan**. Az Azure policy is tiltják a telepítésben az egyes erőforrásokhoz. A leggyakrabban használt, hogy hozzon létre egy házirendet, hogy megtagadja a nyilvános IP-cím egy adott hatókörön belül nem szándékolt módon kiszolgáló kapcsolódik az internethez elkerülve létrehozását.
* **Cost Management és a metaadatok**. Erőforráscímkék erőforrásra és erőforráscsoportra, például a CostCenter, tulajdonosa, és egyéb fontos számlázási adatok hozzáadása gyakran használják. Ezekkel a címkékkel hasznosak a pontos számlázáshoz és az erőforrások kezelését. Házirendeket alkalmazhatnak az alkalmazás összes telepített erőforrás, így könnyebben kezelheti az erőforrások címkék.

### <a name="common-uses-of-initiatives"></a>Gyakori használati irányelveinek

A kezdeményezések bevezetése megadott vállalatok logikai házirendek csoportosíthatja, és a teljes nyomon követése. További kezdeményezések támogatja a vállalat "Agilis" és a "hagyományos" számítási feladat igényeinek kielégítésére. Úgy találtuk, hogy kezdeményezések igazán használja, de gyakran látható:

* **Engedélyezze a monitorozást az Azure Security Centerben**. Ez az az Azure Policy és a egy kiváló példa milyen kezdeményezés van egy alapértelmezett-kezdeményezéshez. Lehetővé teszi a szabályzatok, amelyek azonosítják a nem titkosított SQL-adatbázisok, virtuális gépek biztonsági réseinek és a gyakori biztonsági kapcsolatos igényeinek.
* **Szabályozási adott kezdeményezés**. Vállalatok gyakran jogszabályi követelmény (mint amilyen a HIPAA) közös házirendek csoportba, hogy a vezérlőelemek és a megfelelőség érdekében, hogy ezek a vezérlőelemek hatékonyan nyomon.
* **Erőforrástípusok & termékváltozatok**. Létrehozása, amely korlátozza a típusú erőforrásokat, valamint a termékváltozatok, amely telepíthető telepíthető kezdeményezések segítségével szabályozhatja a költségeket, és győződjön meg arról, a szervezet csak a csapat készségeitől és eljárások támogatásához rendelkezik erőforrások üzembe helyezése.

> [!TIP]
> Azt javasoljuk, hogy mindig kezdeményezési definíciókat használja használjon a szabályzatdefiníciók helyett. Hatókör, például az előfizetés vagy a felügyeleti csoportban,-kezdeményezéshez hozzárendelését követően is egyszerűen hozzáadhat egy másik szabályzat a kezdeményezés bármely hozzárendelések módosítása nélkül. Ez lehetővé teszi az ismertetése, mi van alkalmazva, és nyomon követi a megfelelőségi sokkal egyszerűbb.

### <a name="policy-and-initiative-assignments"></a>Házirend és a kezdeményezés-hozzárendelések

Szabályzatok és a logikai kezdeményezések alapján csoportosítani a létrehozása után hozzá kell rendelnie a szabályzatot hatókör, hogy a felügyeleti csoport, egy előfizetést vagy akár egy erőforráscsoportot. Hozzárendelések lehetővé teszik egy alárendelt hatókört is kizárását a szabályzat-hozzárendelés. Például megtagadja az egy előfizetésen belül nyilvános IP-címek létrehozása, ha hozzárendelés segítségével létrehozhat egy erőforráscsoportot, a védett DMZ csatlakozik egy kivételt.

Több szabályzat példák azt mutatják be, hogyan házirend és a kezdeményezések alkalmazható Azure-ban a különböző erőforrások található [GitHub](https://github.com/Azure/azure-policy) tárház.

## <a name="identity-and-access-management"></a>Identitás- és hozzáférés-kezelés

Első és legfontosabb, kérdések egyike, kérje meg, saját magának a nyilvános felhőben kezdve Ha "kell hozzáféréssel rendelkező erőforrást?" és a "hogyan szabályozhatja a hozzáférést?" Így vagy letiltva a hozzáférés az Azure Portalra, és erőforrások a portálon való hozzáférés szabályozása alapvető fontosságú a hosszú távú sikeres és a biztonsági eszközök a felhőben.

A a feladatnak az erőforrásokhoz való hozzáférés biztonságossá először konfigurálása az identitásszolgáltató és majd a szerepkörök és hozzáférés konfigurálásához. Az Azure Active Directory (Azure AD), a helyszíni Active Directoryhoz csatlakoztatott az alapja az Azure-identitás. Hogy van-e említett az Azure AD *nem* Active Directory és a hozzá tartozó fontos tudni, mi az Azure AD-bérlő van, és hogyan kapcsolódik az Azure-regisztrációjában.  Tekintse át az elérhető [információk](../getting-started/azure-resource-access.md) próbál a jeggyel alapos rálátással az Azure AD és az AD. Csatlakozás és az Active Directory, az Azure AD szinkronizálása, telepítse és konfigurálja a [AD Connect eszköz](/azure/active-directory/connect/active-directory-aadconnect) helyszíni.

![Arch.png](./_images/arch.png)

Azure jelent, ha egy előfizetés hozzáférés-vezérlést voltak: Rendszergazdai vagy Társadminisztrátori. A portálon erőforrások eléréséhez a klasszikus modellt hallgatólagos az egy előfizetéshez való hozzáférést. A részletesebb vezérlés hiánya való megfelelő hozzáférés-vezérlést biztosítanak egy adott Azure-regisztráció előfizetések elterjedése vezetett. Az előfizetések elterjedése már nincs rá szükség. A szerepköralapú hozzáférés-vezérlés (RBAC) hozzárendelhet felhasználókat adja meg a gyakori hozzáférést, például a "tulajdonos", "közreműködő" vagy "olvasó", vagy saját szerepköröket is létrehozhat standard szerepkörök

Szerepköralapú hozzáférés megvalósításához, a következő erősen ajánlott:

* Szabályozza a rendszergazdai vagy Társadminisztrátori az előfizetés, a következő szerepkörök rendelkeznek széles körű engedélyekkel. Csak ki kell adja hozzá az előfizetés tulajdonosa társadminisztrátorként, ha szükségük van a felügyelt Azure klasszikus üzemi modellben.

* Felügyeleti csoportok segítségével [szerepkörök](/azure/azure-resource-manager/management-groups-overview#management-group-access) több előfizetésre kiterjedő és azokat az előfizetés szintjén kezelésének terhe csökkentése érdekében.
* Az Azure-felhasználók felvétele csoportba (például az alkalmazás X tulajdonosai) az Active Directoryban. A szinkronizált csoport használatával adja meg a csoport tagjai az erőforráscsoport, amely tartalmazza az alkalmazás kezelése megfelelő jogosultságokkal.
* Hajtsa végre az alapelvnek megadását, a **legalacsonyabb jogosultsági** a várt munkához szükséges.

> [!IMPORTANT]
>Fontolja meg [Azure AD Privileged Identity Management](/azure/active-directory/privileged-identity-management/pim-configure), az Azure [multi-factor Authentication](/azure/active-directory/authentication/howto-mfa-getstarted) és [feltételes hozzáférési](/azure/active-directory/active-directory-conditional-access-azure-portal) lehetőségeket a nagyobb biztonságot és az Azure-előfizetések közötti felügyeleti műveleteket hajthat végre további látható-e. Ezek a képességek további védelme és kezelése az identity származnak (attól függően, a szolgáltatás) érvényes Azure AD Premium-licenccel. Az Azure AD PIM lehetővé teszi, hogy a "Just-in-Time" rendszergazdai hozzáférést a jóváhagyási munkafolyamat, valamint a rendszergazdai aktiválás és a tevékenységek teljes ellenőrzését. Az Azure MFA egy másik fontos képesség, és lehetővé teszi a kétlépéses ellenőrzést, az Azure Portalra való bejelentkezéshez. Feltételes hozzáférés-vezérlés kombinálva hatékonyan tud kezelni a biztonsági sérülés kockázatát.

Tervezése és előkészítése az identitás és hozzáférés-vezérlés és az Azure-Identitáskezelés ajánlott eljárás a következő ([hivatkozás](/azure/security/azure-security-identity-management-best-practices)) a legjobb kockázati kockázatcsökkentési stratégia lehet alkalmazni, és a kötelező kell tekinteni egyik minden a központi telepítés.

## <a name="security"></a>Biztonság

Biztonsági aggályokat volt a legnagyobb blockers hagyományosan a felhőre való egyikét. Informatikai kockázatkezelők és biztonsági osztályok kell győződjön meg arról, hogy az Azure-erőforrások védettek, és alapértelmezés szerint biztonságos. Az Azure számos olyan képességet, amelyek kihasználhatják a erőforrások védelmét, valamint észlelheti és elkerülése érdekében fenyegetések elleni ezeket az erőforrásokat biztosít.

### <a name="azure-security-center"></a>Azure Security Center

A [az Azure Security Center](/azure/security-center/security-center-intro) erőforrások biztonsági állapotát egyesített áttekintést nyújt a komplex veszélyforrások elleni védelem mellett a környezetben. Az Azure Security Center nyílt platformon, amely lehetővé teszi, hogy a szoftver, amely rendkívüli létrehozása Microsoft-partnerek és a képességek javításához. Az alapkonfiguráció képességeit az Azure Security Center (ingyenes csomag) biztosít az értékelés és javaslatok előmozdító benyomásokkal meg biztonsági állapotát. A fizetős szintek például igény szerinti rendszergazdai hozzáférés és az adaptív alkalmazásvezérlők (Engedélyezés) további és értékes képességek engedélyezéséhez.

> [!TIP]
>Az Azure security center folyamatosan fokozott rendkívül hatékonyan hozhatók létre, és új funkciókat tartalmazó kihasználhatja a fenyegetések és a vállalati védelme. Erősen ajánlott mindig az ASC engedélyezéséhez.

### <a name="azure-resource-locks"></a>Az Azure erőforrás-zárolások

Mivel a szervezet alapvető szolgáltatásához ad hozzá az előfizetések, üzleti fenntartásához egyre fontosabbá válik. A megszakadása, ami gyakran látható egy típus parancsfájlok és eszközök működik a Azure-előfizetés véletlenül erőforrások törlése nem kívánt következményeit. [Erőforrás-zárolások](/azure/azure-resource-manager/resource-group-lock-resources) korlátozhatja a műveletek az értékes erőforrásokat, módosítsa vagy törölje őket egy jelentős hatással lehet. Zárolások lépnek egy előfizetés, erőforráscsoport vagy akár az egyes erőforrások. A gyakori használati eset, hogy alapvető erőforrások, például a virtuális hálózatok, az átjárók, a hálózati biztonsági csoportok és a kulcs tárfiókok zárolások vonatkoznak.

### <a name="secure-devops-toolkit"></a>Biztonságos fejlesztési és üzemeltetési eszközkészlet

A "biztonságos fejlesztési és üzemeltetési Kit for Azure" (AzSK) gyűjteménye, parancsfájlok, eszközök, bővítmények, automatizálását, stb. eredetileg a Microsoft által létrehozott saját IT-csoport és a nyílt forráskód a Githubon keresztül kiadott ([hivatkozás](https://github.com/azsk/DevOpsKit-docs)). AzSK a teljes körű Azure-előfizetés és az erőforrások biztonságának caters széles körű automatizálással és zökkenőmentes biztonsági integrálása a natív fejlesztési ops munkafolyamatok útmutatás nyújtása a biztonságos fejlesztési ops az alábbi 6 fókuszterületek elvégzéséhez csapatok igényeinek megfelelően:

* Az előfizetés biztonságossá tétele
* Biztonságos fejlesztési engedélyezése
* CI/CD biztonsági integrálása
* Folyamatos garancia
* Riasztások és -Monitorozás
* Felhőbeli kockázati Cégirányítási

![Az Azure és-üzemeltetői eszközkészlet](_images/Secure_DevOps_Kit_Azure.png)

A AzSK széles választékának eszközök, a szkriptek és a egy teljes körű Azure cégirányítási terv fontos részét képező információt pedig ez beépítése az scaffold kulcsfontosságúak a szervezetek kockázati kezelés céljaihoz támogatása

### <a name="azure-update-management"></a>Az Azure az Update Management

A kulcsfontosságú feladatokat, megteheti a környezet biztonságban egyik, győződjön meg arról, hogy a kiszolgálók telepítve van a legújabb frissítéseket. Bár vannak ehhez számos eszközt, az Azure biztosít a [Azure Update Management](/azure/automation/automation-update-management) megoldás a azonosítása és a kritikus fontosságú operációsrendszer-javítások bevezetését.  Lehetővé teszi, az Azure Automation használata (amely tárgyalja a [automatizálása](#automate) szakaszban az útmutató későbbi részében.

## <a name="monitor-and-alerts"></a>A figyelő és riasztás

Gyűjtése, és telemetriát biztosító üzemel, a tevékenységek elemzése, teljesítmény-mérőszámok, egészségügyi és az összes az Azure-előfizetést használ a szolgáltatások rendelkezésre állása fontos proaktív módon kezelheti az alkalmazások és infrastruktúra és a egy alapvető kell minden Azure-előfizetés. Minden Azure-szolgáltatás tevékenységeket tartalmazó naplók, metrikák és diagnosztikai naplók formájában telemetriai adatokat küldenek.

* **A Tevékenységnaplók** az előfizetésekben erőforrásokon végrehajtott összes műveletet ismerteti
* **Metrikák** numerikus adatok egy erőforrást, a teljesítmény és a egy erőforrás állapotát leíró a kibocsátott
* **Diagnosztikai naplók** projektsablon egy Azure-szolgáltatás által kibocsátott, és adja meg a műveletet, hogy a szolgáltatás gazdag, gyakori adatait.

Ezek az információk tekinthetők meg és több szinten megtudjuk, és folyamatosan fejlesztjük. Az Azure biztosít **megosztott**, **core** és **részletes** figyelési képességek az Azure-erőforrások az alábbi ábrán a szolgáltatások keretében.
![Figyelés](./_images/monitoring.png)

### <a name="shared-capabilities"></a>Közös képességek

* **Riasztások**: Begyűjtheti minden napló, esemény- és metrikaadatokat Azure-erőforrásoktól érkező, de nem képes értesítést kapni a kritikus fontosságú feltételek és az azoknak, ezeket az adatokat csak akkor hasznos a nagy horderejű céllal és a törvényszéki. Azure-riasztások proaktívan kaphat értesítést feltételeket határoz meg az alkalmazások és infrastruktúra között. Riasztási szabályok naplók, az események és mérőszámok, amely beállítja a címzettek értesítése Műveletcsoportok használatával hoz létre. Műveletcsoportok is lehetővé teszi, hogy külső műveleteket, mint például a webhookok használata az Azure Automation-runbookok és az Azure Functions futtatásához szervizelési automatizálása.

* **Az irányítópultok**: Az irányítópultok lehetővé teszi összesített figyelési nézetei és kombinálhatók az adatok minden erőforrásban és előfizetésnél, hogy a telemetria Azure-erőforrások egy vállalati szintű betekintést. Létrehozása és konfigurálása saját nézeteket, és megosztja őket másokkal. Létrehozhat például, egy irányítópult csempéi különböző információkat biztosít, többek között az Azure SQL DB, a PostgreSQL-hez készült Azure DB és a MySQL-hez készült Azure DB, az összes Azure database szolgáltatásban az adatbázisok álló.

* **Metrikaböngésző**: Mérőszám játszik numerikus értékek jön létre az Azure-erőforrások (pl. % Processzor, lemez I/O, a művelet és az erőforrások teljesítményét betekintést biztosító. A Metrikaböngésző meghatározása és a metrikák, amelyben érdekli a Log Analytics összesítő és elemzési küldhet.

### <a name="core-monitoring"></a>Alapvető monitorozás

* **Az Azure Monitor**: Az Azure Monitor az alapvető platformszolgáltatás, amely egyetlen adatforrás az Azure-erőforrások figyeléséhez. A Azure Monitor az Azure Portal felületének kínál egy központi jump pont ki a figyelési funkciók többek között az átfogó figyelési képességek az Application Insights – Naplóelemzés, figyelés hálózati, felügyeleti megoldások Azure-ban, és Szolgáltatástérképek. Az Azure monitorban jelenítheti meg, lekérdezés, az útvonal, archiválására és metrikákat és naplókat a teljes felhőalapú hagyatéki különböző Azure-erőforrások érkező cselekedhet. A portálon kívül kérheti le adatokat a Monitor PowerShell-parancsmagok, Cross Platform CLI vagy az Azure Monitor REST API-kon keresztül.

* **Az Azure Advisor**: Az Azure Advisor folyamatosan figyeli a telemetriát az előfizetések és a környezetek között, valamint ajánlásokkal pénzt takaríthat meg, és jobb teljesítmény, biztonság és az erőforrások rendelkezésre állása az Azure-erőforrások optimalizálása érdekében ajánlott eljárásait, amely az alkalmazások alkotják.

* **Állapotfigyelő szolgáltatás**: Az Azure Service Health az Azure-szolgáltatásokkal, amely befolyásolhatja az alkalmazások, valamint a segítséget nyújt az ütemezett karbantartási időszak tervezése azonosítja a problémákat.

* **Tevékenységnapló**: A tevékenységnapló előfizetés erőforrásainak összes műveletet ismerteti. Biztosít auditnaplót meghatározni a "mi", "ki" és "when" bármely létesítése, frissítése és törlési művelet erőforrásokon. Tevékenységnapló eseményeit tárolja a platform és 90 napig lekérdezhetők. Betöltheti az Tevékenységnaplók Log analyticsbe hosszabb megőrzési időszakok beállításának és mélyebb lekérdezését és elemzését az között több erőforrást.

### <a name="deep-application-monitoring"></a>Alkalmazások részletes monitorozása

* **Az Application Insights**: Az Application Insights lehetővé teszi, hogy az alkalmazás adott telemetriai adatok gyűjtésére, és figyelje a teljesítményét, rendelkezésre állás és a felhőben vagy a helyszíni alkalmazások használatát. Az alkalmazás a támogatott SDK-kal több nyelvet, például a .NET, JavaScript, JAVA, Node.js, a Ruby és Python alakíthatja ki. Application Insights-események vannak betöltődnek az ugyanazon a Log Analytics adattár, amely támogatja az infrastruktúra és a biztonsági figyelés korrelációját engedélyezi és események összevonása egy részletes lekérdezési nyelvet révén.

### <a name="deep-infrastructure-monitoring"></a>Infrastruktúra részletes monitorozása

* **Log Analytics**: A log Analytics központi szerepet tölt az Azure monitoring telemetriai és egyéb adatokat gyűjt különböző forrásokból, és a egy lekérdezési nyelvet és elemzési motor, amely betekintést nyerhet az alkalmazások és erőforrások működését, megadásával. Is dolgozhat közvetlenül a Log Analytics-adatok rendkívül nagy teljesítményt nyújtva naplókereséseken és nézeteken keresztül, vagy használhatja az egyéb Azure-szolgáltatások, amelyek tárolják az adataikat a Log Analyticsben, például az Application Insights vagy az Azure Security Center elemzési eszközökkel hajthat végre.

* **A hálózatfigyelés**: Az Azure hálózati figyelési szolgáltatásai lehetővé teszik betekintést nyerhet a hálózati adatforgalmat, a teljesítmény, a biztonság, a kapcsolat és a szűk keresztmetszeteket. Egy jól megtervezett hálózati kialakítások tartalmaznia kell az Azure hálózati szolgáltatások, például a Network Watcher és ExpressRoute-figyelő konfigurálása.

* **Felügyeleti megoldások**: Felügyeleti megoldások, amelyeket a csomagolt logika, a insights és a egy alkalmazás vagy szolgáltatás az előre definiált Log Analytics-lekérdezéseket. A Log Analytics támaszkodnak az tárolni és elemezni az eseményadatokat alapjaként. Minta felügyeleti megoldások közé tartozik, tárolók és az Azure SQL Database-elemzések figyelése.

* **A Service Map**: A Service Map az infrastruktúra-összetevőket, a folyamatok és a köztük fennálló függőségek grafikus betekintést biztosít a más számítógépeken és a külső folyamatok. Integrálható események, teljesítményadatok és felügyeleti megoldásokat a Log Analyticsben.

> [!TIP]
> Mielőtt létrehozná a riasztásokat egyenként, létrehozása és kezelése, amelyek segítségével különböző Azure Alerts Műveletcsoportok megosztott készletét. Ez lehetővé teszi a címzettlisták, az értesítés kézbesítéséhez használt módszerek (e-mailben, SMS telefonszámok) és a külső műveleteket webhookok életciklusának központilag karbantartása (az Azure Automation-runbookok, az Azure Functions és a Logic Apps, ITSM).

## <a name="cost-management"></a>Költségkezelés

A kapcsoló, amely a helyszínen a felhőből a nyilvános felhőbe való áthelyezésekor fognak adódni fontosabb változását foglalja össze egyik, tőkeráfordítási (hardvervásárlással) származó működési kiadásokat (kellene fizetnie szolgáltatás használata során azt). Ez a kapcsoló az állomások CAPEX költségekké alakítását is elérhetővé teszi a több gondosan felügyelnie a költségeket. A felhő előnye, hogy, alapvetően és pozitív hatással lehet a költségét, csupán kikapcsolt (vagy átméretezési) bekapcsolja azt, nincs szükség esetén használja. Szándékosan a költségkezeléshez a felhőben, ajánlott gyakorlat és a egy érett ügyfelek által naponta.

A Microsoft biztosít egy több eszközt, hogy tud-e a vizualizációra, nyomon követheti, és kezelheti a költségeket. Is biztosítunk teljes körű API-k és testreszabását cost management integrálható a saját eszközök és az irányítópultok engedélyezéséhez. Ezek az eszközök lazán vannak csoportosítva: Az Azure Portal-funkciók és a külső funkciók

### <a name="azure-portal-capabilities"></a>Az Azure Portal képességei

Ezek olyan eszközöket, a költség, valamint a műveletek lehetővé teszi az azonnali információval szolgálnak

* **Előfizetés Átköltöztetéssel**: A portálon található a [Azure Cost Analysis](/azure/cost-management/overview) nézetet biztosít egy pillantást a költségeket, és a napi információkat felhőköltéseiket erőforrás vagy erőforráscsoport.
* **Az Azure Cost Management**: A termék megvásárlása esetén a Microsoft Cloudyn eredményét, és lehetővé teszi, hogy a kezelése és elemzése az Azure más nyilvános felhőszolgáltatók kapcsolatos költségek, valamint költségek. Mindkettő ingyenes és fizetős szintek, a képességek nagyszerű rengeteg ott vannak látható módon a [áttekintése](/azure/cost-management/overview).
* **Az Azure költségvetése és Műveletcsoportok** , hogy milyen somethings költségeket, és valami kapcsolatos, amíg a közelmúltban lett több manuális gyakorlat. Azure költségvetése és annak API-k bevezetése, most már lehetősége műveletek létrehozása (ahogy az [ez](https://channel9.msdn.com/Shows/Azure-Friday/Managing-costs-with-the-Azure-Budgets-API-and-Action-Groups) példa) költségek találata egy küszöbértéket. Ha 100 %-a költségvetés vagy [egy másik példa] elér például állítja le a "test" erőforráscsoport.
* **Az Azure Advisor** valamit a költségeket, hogy csak a felét élesben, ezért a másik fele van, hogy mi a teendő velük a. [Az Azure Advisor](/azure/advisor/advisor-overview) pénzt takaríthat meg, a megbízhatóság javításához vagy még biztonságosabbá elvégzendő műveleteket a javaslatokat nyújt.

### <a name="external-cost-management-tools"></a>Külső költségkezelési eszközökkel

* **A Power bi Azure Consumption Insights**. Biztosan hozhatja létre saját vizualizációkat, a szervezet számára? Ha igen, majd az Azure Consumption Insights tartalomcsomag a Power BI a kívánt eszközben dolgozhat. A tartalomcsomag és a Power bi hozhat létre egyéni vizualizációkat, amelyek a szervezet használja, részletesebb elemzéseket végezhet a költségek, és adja hozzá a további Adatbővítés egyéb adatforrásait.

* **Használatalapú API**. A [fogyasztás API-k](/rest/api/consumption/) programozás alapú hozzáférést biztosítanak a költség- és használati adatokat költségvetéseket, a fenntartott példányok és a piactér-díjak információk mellett. Ezen API-k csak a vállalati Belépéseket és a egy Web Direct-előfizetéssel elérhető azonban ezek lehetővé teszik, a költségadatok integrálása a saját eszközök és az adattárházakhoz. Látható, az Azure CLI használatával is elérheti ezeket az API-k [Itt](/cli/azure/consumption?view=azure-cli-latest).

Amikor az ügyfelek, akik használta a felhőbeli hosszú ideig, és azok használata "érett" között áttekintjük, láthatjuk, erősen ajánlott eljárásokat számos

* **Aktívan figyeljük költségek**. Érett az Azure-felhasználók folyamatosan azok a szervezetek, költségek figyelése, és szükség esetén tegye. Egyes szervezetek még dedikált elemzéseket végezhet és használati módosítási javaslatokat tehet, és ezek a személyek több mint fizet magukat egy fel nem használt HDInsight-fürtöt, amely a futási idejének hónapok megtalálják az első alkalommal.
* **A fenntartott példányok használatához**. A felhőbeli költségek kezelése egy másik fő bérlőben, hogy az ideális eszközt használva az adott feladathoz. Ha egy 24 x 7-es kell maradnia IaaS virtuális Gépen, majd a fenntartott példány használatával menti, jelentős költséget takaríthat meg. Az egyensúlyt a virtuális gépek leállítását automatizálása és a fenntartott példányok használatával között találja a felhasználói élményt és -elemzés vesz igénybe.
* **Automation hatékony használata**: Számos számítási feladatok nem kell futnia minden nap. Akár kikapcsolásával egy virtuális gép naponta 4 órás időszakra takaríthat meg, 15 %-a költségek. Automation gyorsan fizet a saját maga.
* **Az erőforráscímkék használata láthatósági**: Ebben a dokumentumban említett máshol, az erőforráscímkék használata teszi lehetővé a költségek jobb elemzés céljából.

A Cost management egy, core, a hatékony és hatékony az nyilvános felhő futtató szakterületi. Olyan vállalatok, amelyek sikeres érhet el fogja tudni a költségek csökkentését és a tényleges igény szerint figyelésekor overbuying való, és igény szerint bízva származnak.

## <a name="automate"></a>Automatizálás

Számos képességeket, amelyek különbözteti meg a felhőszolgáltatóknak, amelyek segítségével a szervezetek a lejárat egyik automation, amely azokat a beépített szintjét.  Automation never-ending folyamat során a rendszer és a szervezet a felhőbe helyezi át, bármilyen szükséges erőforrással és idővel épületben támogatásán terület.  Automation az ügyfélellenőrzésnek a szervizelése problémák többek között az erőforrások (ahol kiadásában közvetlenül, egy másik scaffold alapfogalom, sablonok és DevOps) egységes bevezetést számos célra szolgál.  Automation a "kötőszövet" az Azure scaffold, és hivatkozásokat tartalmaz minden terület együtt.

Számos módon, építse ki ezt a funkciót, az első féltől származó eszközökkel, például az Azure Automation, EventGrid, valamint a harmadik féltől származó jelentős mennyiségű-AzureCLI-eszközök például a Terraform, a Jenkins, a Chef és Puppet (hogy néhányat említsünk) biztosított eszközökkel. Alapvető a műveleti csapat képes elvégezni, automatizálhatja a következők: Azure Automation, Event Grid és az Azure Cloud Shell:

* **Az Azure Automation**: Egy olyan felhő alapú funkció, amely lehetővé teszi, hogy a szerző Runbookok (a PowerShell vagy Python), és lehetővé teszi, hogy a folyamatok automatizálása, konfigurálhat erőforrásokat és -javítások alkalmazása a még van.  [Az Azure Automation](/azure/automation/automation-intro) számos képletkategória közötti platform képességei, amely az üzemelő példány szerves azonban túl széles körű, de itt mélyebben van.
* **Event Grid**: Ez [szolgáltatás](/azure/event-grid) egy teljes körűen felügyelt esemény útválasztási rendszer nézzük meg eseményekre reagáló belül az Azure-környezetben. Az Automation az érett felhőbeli szervezetek kötőszövet is, például Event Grid a helyes automatizálási kötőszövet. Event Grid használatával létrehozhat egy egyszerű, kiszolgáló nélküli, művelet egy e-mail küldése, amikor egy új erőforrás jön létre, és ennek az erőforrásnak jelentkezzen be egy adatbázist. Hogy ugyanazon Event Grid értesítése, ha törölnek egy erőforrást, és az elem eltávolítása az adatbázisból.
* **Az Azure Cloud Shell**: van egy interaktív, böngészőalapú [rendszerhéj](/azure/cloud-shell/overview) az Azure-erőforrások kezeléséhez. Teljes környezet vagy a PowerShell vagy a Bash, amely biztosítja, hogy biztosan konzisztens környezetet használhatnak, amelyről a parancsfájlok futtatásához szükséges (és karban az Ön számára) el. Az Azure Cloud Shell - már telepítve van – további kulcs eszközök hozzáférést biztosít a automatizálható a környezet többek között [Azure CLI-vel](/cli/azure/get-started-with-azure-cli?view=azure-cli-latest), [Terraform](/azure/virtual-machines/linux/terraform-install-configure) és egyre nagyobb számban elérhető további [eszközök ](https://azure.microsoft.com/updates/cloud-shell-new-cli-tools-and-font-size-selection/) tárolók, adatbázisok (sqlcmd) és egyéb kezeléséhez.

Automation nappali tagozatos feladat, és gyorsan válik az egyik legfontosabb működési feladat a felhő csapaton belül. Szervezetek számára, amelyek a megközelítést az "első automatizálása" az Azure-nagyobb sikeresek legyenek:

* Költségek kezelése: aktívan lehetőségek keresést és az automation újra az erőforrásokat, a méretezési csoport – felfelé és lefelé méretezés, és kapcsolja ki a nem használt erőforrások létrehozásához.
* Működési rugalmasság: automation (valamint a sablonok és DevOps) használatával, amely növeli a rendelkezésre állási, növeli a biztonságot, és lehetővé teszi, hogy a csapata üzleti problémák megoldására összpontosíthat ismételhetőség szintű kapjanak.

## <a name="templates-and-devops"></a>Sablonok és a fejlesztés és üzemeltetés

Kiemelt az automatizálás szakaszban a cél szervezetként kell verziókövetési sablonok és parancsfájlok segítségével-erőforrások kiépítése és a környezetek interaktív konfigurációs minimalizálása érdekében. Ez a megközelítés az "infrastruktúra mint kód" együtt egy megvalósítása olyan fegyelmezett fejlesztési és üzemeltetési folyamatot a folyamatos üzembe helyezéshez is biztosítani a konzisztenciát és a környezetek eltéréseket csökkentheti. Szinte minden Azure-erőforrás keresztül telepíthető [Azure Resource Manager (ARM) JSON-sablonok](/azure/azure-resource-manager/resource-group-template-deploy) PowerShell vagy az Azure platform CLI és eszközök, például a származó (amely első osztályú támogatást Hashicorp Terraform együtt és az Azure Cloud Shell integráltan).

Például a cikk [ehhez](https://blogs.msdn.microsoft.com/mvpawardprogram/2018/05/01/azure-resource-manager/) adjon meg egy kiváló vitafórum kapcsolatos ajánlott eljárásokat és ARM-sablonok alkalmazása egy fejlesztési és üzemeltetési megközelítés tapasztalatokat a [Azure DevOps](/azure/devops/user-guide/?view=vsts) lánc eszközzel. Az idő és munka fejlesztéséhez egy adott, a szervezet követelményeinek sablonok álló alapkészlet igénybe, és a folyamatos teljesítés fejleszthet folyamatok DevOps-eszköz láncok (az Azure DevOps, a Jenkins, bambusz, Teamcity, Sétatér), kifejezetten rendszergazdák számára az éles üzemben futó és QA környezetekben. Az egy nagy könyvtár [Azure gyors üzembe helyezési sablonokat](https://github.com/Azure/azure-quickstart-templates) a Githubon, hogy a kiindulási pontként használható sablonokat, és gyorsan hozhat létre felhőalapú kézbesítési folyamatok Azure DevOps.

Ajánlott eljárásként éles előfizetések vagy erőforráscsoportok a cél kell kell használó RBAC security alapértelmezés szerint és a folyamatos készregyártás automatizált folyamatok alapján az egyszerű szolgáltatások összes erőforrás kiépítéséhez használatával interaktív felhasználó letiltása és minden alkalmazás kódját. Nem rendszergazdai vagy a fejlesztői érintse interaktív módon az erőforrások konfigurálása az Azure Portalon. Ezt a szintű DevOps összehangolt vesz igénybe, és az Azure scaffold funkcióira használja, és, hogy az megfeleljen a szervezetek számára, hogy növelje a méretezési csoport egységes és biztonságosabb környezetet biztosít.

> [!TIP]
> Tervezése és fejlesztése összetett ARM-sablonok, [kapcsolódó sablonok](/azure/azure-resource-manager/resource-group-linked-templates) rendszerezése és újrabontás bonyolult erőforrás kapcsolatok monolitikus JSON-fájlokból. Ez lehetővé teszi, hogy külön-külön kezelheti az erőforrásokat, és könnyebben olvasható, testable és újrafelhasználható, győződjön meg a sablonokat.

Azure a nagy kapacitású felhőszolgáltatók és a helyszíni kiszolgálók rengeteg a szervezet áthelyezése a felhőbe, mert ugyanezek a fogalmak, amely a felhőbeli szolgáltatók és az SaaS-alkalmazások használatával használata nyújt a szervezet tudunk reagálni az üzleti igényeire sokkal hatékonyabb módja.

## <a name="core-network"></a>A központi hálózat

Az Azure scaffold referenciamodellje végső összetevője mag, hogyan a szervezet fér hozzá, az Azure biztonságos módon. Erőforrásokhoz való hozzáférés lehet (a vállalati hálózaton belül) belső vagy külső (az interneten) keresztül. A felhasználók a szervezet véletlenül a nem megfelelő helyszíni helyezni erőforrásokat, és potenciálisan rosszindulatú hozzáférés meg is gyerekjáték. Csakúgy, mint a helyszíni eszközök, vállalatok hozzá kell adnia a megfelelő szabályozásokkal győződjön meg arról, hogy az Azure-felhasználók a helyes döntések meghozatalában. Az előfizetés-irányítás hogy ezzel a alapvető erőforrások egyszerű hozzáférés vezérlése érdekében. Az alapvető erőforrások állnak:

* **Virtuális hálózatok** alhálózatok tároló objektumok. Bár ez nem feltétlenül szükséges, azt gyakran használják alkalmazások belső vállalati erőforrásokhoz való csatlakozáskor.
* **Felhasználó által megadott útvonalak** lehetővé teszik az útvonaltábla küldésével forgalom hálózati virtuális készüléken keresztül vagy a virtuális társhálózatokban lévő távoli átjárót egy alhálózaton belül módosíthatja.
* **Virtuális hálózatok közötti Társviszony** lehetővé teszi, hogy zökkenőmentesen csatlakoztathatja a két vagy több Azure virtuális hálózat létrehozása összetettebb hub & küllő kialakításokat, vagy megosztott szolgáltatások hálózatok.
* **Szolgáltatásvégpontokat**. Múltbeli időpont PaaS-szolgáltatások különböző módszerekkel történő biztonságos hozzáférés a ezeket az erőforrásokat a virtuális hálózatokról hivatkozni. A Szolgáltatásvégpontok csak az engedélyezett PaaS szolgáltatásokhoz való biztonságos hozzáférési kapcsolódó végpontok, általános biztonságának növelése.
* **Biztonsági csoportok** meghatározó szabályokat adja meg és- tárolókról az Azure-erőforrások bejövő és kimenő adatforgalom engedélyezéséhez vagy letiltásához lehetővé teszi számos képletkategória vannak. [Biztonsági csoportok](/azure/virtual-network/security-overview) áll a biztonsági szabályok, amelyek a kiegészíthető **Szolgáltatáscímkék** (amelyek meghatározzák például azurekeyvault értékre van, az Sql és egyéb gyakori Azure-szolgáltatások), és **alkalmazáscsoportok** (amelyek meghatározzák, és az alkalmazás struktúra, például WebServers, AppServers és például)

> [!TIP]
> Használjon szolgáltatáscímkéket és alkalmazásbiztonsági csoportokból a hálózati biztonsági csoportokban nem csupán áttekinthetősége a szabályok – amely elengedhetetlen ismertetése hatás -, hanem csökkenti a fölösleges terhelése nagyobb alhálózat hatékony mikroszegmentációt engedélyezéséhez és Növelje a rugalmasságot.

### <a name="virtual-data-center"></a>Virtuális adatközpont

Az Azure biztosít mind a belső funkciókat biztosítanak, és a külső a funkciók a kiterjedt partneri hálózat, amelyek lehetővé teszik, hogy rendelkezik egy érvényes biztonsági forgalmazóval. Ami még fontosabb, a Microsoft biztosít, ajánlott eljárások és útmutató formájában a [Azure virtuális adatközpont](/azure/architecture/vdc/networking-virtual-datacenter). Helyezi át az egyetlen számítási feladat több számítási feladatok, amelyek hibrid képességeit kihasználva, mivel a VDC útmutatást biztosít Önnek "recept" növekszik, ahogy a számítási feladatokat az Azure-ban növelheti a rugalmas, hálózat engedélyezéséhez.  

## <a name="next-steps"></a>További lépések

Cégirányítási elengedhetetlen a sikeres Azure. Ez a cikk egy enterprise scaffold technikai végrehajtásának célozza, de csak éri el a szélesebb körű folyamat és az összetevők közötti kapcsolatok. A házirend irányítási folyamatok felülről lefelé, és határozza meg az üzleti szeretné elérni. Természetesen magában foglalja a cégirányítási modell létrehozása az Azure-ban képviselői az informatikai, de ami még fontosabb Cégvezetők csoport és a biztonság és a kockázatkezelés erős ábrázolás kell rendelkeznie. A végén az enterprise scaffold van veszélyeztetettségének üzleti megkönnyítése érdekében a szervezet céljaira és célok

Most, hogy az előfizetés-irányítás megismerkedett, ideje, hogy ezek a javaslatok a gyakorlatban. Lásd: [példák az Azure előfizetés-irányítás végrehajtási](azure-scaffold-examples.md).
