---
title: Ajánlott eljárások a vállalatok áthelyezése az Azure-bA |} Microsoft Docs
description: Egy, a vállalatok használatával biztosíthatja a biztonságos és kezelhető környezet scaffold ismerteti.
services: azure-resource-manager
documentationcenter: na
author: rdendtler
manager: timlt
editor: tysonn
ms.assetid: 8692f37e-4d33-4100-b472-a8da37ce628f
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/31/2017
ms.author: rodend;karlku;tomfitz
ms.openlocfilehash: 8bc2ed253b951ea04eb25b76f67dcc2092dc94ad
ms.sourcegitcommit: 4ec010846b9b5545c843a32e08293f906e512302
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 05/17/2018
ms.locfileid: "34299976"
---
# <a name="azure-enterprise-scaffold---prescriptive-subscription-governance"></a>Az Azure enterprise scaffold - részletes utasításokkal megadott előfizetés-irányítás
A vállalatok egyre vannak bevezetése a nyilvános felhőben az agilitást és a rugalmasságot. A felhő szintjeiről bevételt, vagy a vállalati erőforrások optimalizálására azok vannak használatával. A Microsoft Azure kínál számos, hogy a vállalatok például építőelemeket munkaterheléseket és alkalmazásokat az széles köréről megoldására összeállíthat. 

De, hogy tudnák, hogyan kezdjek hozzá gyakran nehéz. Miután Azure használja, néhány kérdést gyakran merülhetnek fel:

* "Hogyan megfelelnek a törvény írja elő az adatok közös joghatóság alá, az egyes országokban?"
* "Hogyan do I győződjön meg arról, hogy valaki nem módosíthatja a kritikus rendszer?"
* "Honnan tudhatom, hogy mi minden erőforrás támogat, I is fiókot használja, és pontosan vissza számlázási?"

Az üres előfizetés nincs védőkorlátokat a potenciális ijesztőnek. Az üres helyet is akadályozhatják a áthelyezése az Azure-bA.

Ez a cikk műszaki szakemberek számára a tárhelyfelhasználás irányításhoz, és kiegyenlítheti a agilitást a kell kiindulási pontot biztosít. Akkor jelenik meg az olyan vállalati scaffold, amely végigvezeti a szervezetek megvalósítása és kezelése az Azure-előfizetések. 

## <a name="need-for-governance"></a>Az irányítás szükséges
Amikor helyezi át az Azure-ba, meg kell oldania a témakör az irányítás korai sikeres használatát a vállalaton belül a felhő biztosításához. Sajnos az idő és egy átfogó irányítás rendszer létrehozásának bürokrácia azt jelenti, hogy egyes üzleti csoportok lépjen közvetlenül szállítók vállalati informatikai bevonása nélkül. Ez a megközelítés hagyhatja, a vállalati nyissa meg a biztonsági réseinek Ha az erőforrások nem megfelelően kezeli. A nyilvánosfelhő - agilitást, a rugalmasságot és fogyasztás alapján árképzési - jellemzői számára fontos üzleti csoportok, amelyeket gyorsan igényeinek megfelelően (belső és külső). De a nagyvállalati informatikai adatok és rendszerek hatékonyan védelmének biztosításához szükséges.

A valós életben állványok segítségével hozza létre a struktúra. A scaffold részletesen az általános elvet követik, és csatlakoztatni kell állandó rendszerek horgonyzási pontokat biztosít. Egy vállalati scaffold megegyezik: rugalmas vezérlők és az Azure-képességek, amelyek a környezetet, és a központi jellegűek alapot biztosítanak a nyilvános felhő épülő szolgáltatások készlete. A szerkesztők biztosít (informatikai és üzleti csoportok) létrehozása és csatolása a új szolgáltatások alaprendszert.

A scaffold eljárások azt összegyűjtötte a különböző méretű ügyfelek sok bevonására alapul. Ezen megoldások Fortune 500 vállalatok és a független szoftvergyártók áttelepítése és a felhőalapú megoldások felhőben kisebb szervezetek ügyfelek közé. A vállalati scaffold "erre a célra kialakított" rugalmas hagyományos informatikai munkaterhelések és a gyors munkaterhelések; támogatásához például a fejlesztők szoftver,--szolgáltatás (SaaS) alkalmazások létrehozása Azure-képességek alapján.

A vállalati scaffold szándék szerint az Azure-ban minden egyes új előfizetés alapját. Ez lehetővé teszi a rendszergazdáknak munkaterhelések megfelelnek-e a minimális cégirányítási követelmények a szervezet anélkül, hogy megakadályozza az üzleti csoportok és a fejlesztők gyorsan teljesíti a saját kitűzött célokat.

> [!IMPORTANT]
> Cégirányítási elengedhetetlen a sikeres Azure. Ez a cikk egy vállalati scaffold műszaki végrehajtásának célozza, de csak koppint szélesebb körű folyamatáról és az összetevők közötti kapcsolat. Házirend irányítási felülről lefelé zajló kommunikációról, és határozza meg az üzleti szeretne elérése. Természetesen a cégirányítási-modell Azure foglal magában képviselőinek informatikai, de ennél is fontosabb rendelkeznie kell az üzleti csoport vezetők és adatvédelmi és kockázatkezelési management erős ábrázolását. A végén egy vállalati scaffold célja veszélyeztetettségének üzleti egy szervezet céljaira és célok megkönnyítése érdekében.
> 
> 

Az alábbi képen a scaffold összetevőit mutatja be. A foundation egy teli tervet a részlegek, fiókok és -előfizetések támaszkodik. Az oszlopok erőforrás-kezelő házirendek és az erős elnevezési szabályai állnak. A scaffold a többi Azure-képességek core származik, és szolgáltatásokat, hogy engedélyezze a biztonságos és kezelhető környezet.

![scaffold összetevők](./_images/components.png)

> [!NOTE]
> Azure gyors nőtt 2008 bevezetése óta. A növekedési mérnöki munkacsoportok Microsoft rethink azok megközelítés kezelése és szolgáltatások telepítése szükséges. Az Azure Resource Manager modellt 2014-ben került bevezetésre, és a felváltja a klasszikus üzembe helyezési modellben. Erőforrás-kezelő lehetővé teszi a szervezetek könnyebben telepítheti, rendezése és Azure-erőforrások szabályozását. Erőforrás-kezelő tartalmazza a párhuzamos folyamatkezelést biztosítja, összetett, egymástól kölcsönösen függnek megoldások gyorsabb telepítése erőforrások létrehozásakor. Több szintű hozzáférés-vezérlés és a metaadatok címke erőforrások lehetőséget is tartalmaz. A Microsoft azt javasolja, hogy minden erőforrás az erőforrás-kezelő használatával hozzon létre. A vállalati scaffold explicit módon végzi a Resource Manager modellt.
> 
> 

## <a name="define-your-hierarchy"></a>Adja meg a hierarchiában
A scaffold építkezve az Azure a nagyvállalati beléptetés (és a vállalati portál). A nagyvállalati beléptetés határozza meg az alakzat és használó Azure-szolgáltatásokat a vállalaton belül, irányítási core szerkezete. A nagyvállalati szerződés belül az ügyfelek képesek további megkönnyíti a részlegek, fiókok, és végül előfizetések környezet. Az Azure-előfizetésre a alapvető egysége, ahol minden erőforrás található. Egyúttal meghatározza, milyen több határokon belül Azure, például a magok, erőforrások stb.

![hierarchia](./_images/agreement.png)

Minden vállalati eltér, és az előző ábrán a hierarchia lehetővé teszi, hogy hogyan vannak rendszerezve Azure a vállalaton belül jelentős rugalmasságot. Végrehajtási az ebben a dokumentumban szereplő útmutatást, mielőtt a hierarchiában a modell és megismerheti a hatását a számlázás, erőforrás-hozzáférés és összetettségét.

Az Azure-regisztrációkat a három közös minták a következők:

* A **működési** minta
  
    ![funkcionális](./_images/functional.png)
* A **részleg** minta 
  
    ![vállalata számára](./_images/business.png)
* A **földrajzi** minta
  
    ![földrajzi](./_images/geographic.png)

Az előfizetés kiterjesztése a cégirányítási követelményeket a vállalat az előfizetés szintjén scaffold telepítését.

## <a name="naming-standards"></a>Elnevezési szabályai
Az első oszlop, a scaffold van elnevezési szabványoknak. Tetszetős elnevezési szabályai lehetővé teszik a portálon, a számlázási és parancsfájlban erőforrások azonosítása. Nagy valószínűséggel már szabványok elnevezési a helyszíni infrastruktúrát. Amikor Azure ad hozzá a környezetben, akkor ki kell terjeszteni elnevezési szabványokat a az Azure-erőforrások. Szabványos elnevezési segítik a környezet minden szinten hatékonyabb felügyeletet.

> [!TIP]
> Az elnevezési konvenciókat:
> * Tekintse át, és lehetőség szerint elfogadja a [minták és gyakorlatok útmutatást](../best-practices/naming-conventions.md). Ez az útmutató segítséget nyújt a adja meg egy beszédes elnevezési szabványnak.
> * Használjon camelCasing erőforrások (például a contoso.com és vnetNetworkName) nevét. Megjegyzés: Nincsenek bizonyos erőforrások, például a storage-fiókok, ahol a egyetlen lehetősége kisbetű (és más különleges karaktereket).
> * Fontolja meg a házirendekkel Azure Resource Manager (a következő szakaszban leírt) elnevezési szabályai érvényesítését.
> 
> Az előző tippek segítségével megvalósíthatja az egységes elnevezési szabályokat alkalmaz.

## <a name="policies-and-auditing"></a>Házirendek és a naplózás
A scaffold a második pillar magában foglalja [Azure házirendek](/azure/azure-policy/azure-policy-introduction) és [a műveletnapló naplózás](/azure/azure-resource-manager/resource-group-audit). Erőforrás-kezelő házirendek biztosítják azokat az Azure-ban kockázat kezelése. Megadhatja, amelyek biztosítják az adatok közös joghatóság alá korlátozása, kényszerítése, vagy bizonyos műveletek naplózási házirendeket. 

* A házirend az alapértelmezett **engedélyezése** rendszer. Műveletek és szabályzatok hozzárendelését meg vagy az erőforrás műveletek naplózhatók erőforrásokhoz szabályozhatja.
* Egy házirend adatdefiníciós nyelv (if-majd feltételek) a házirend-definíciók alapján házirendek ismerteti.
* Létrehozhat olyan formázott fájlok JSON (Javascript Object Notation) a házirendeket. Házirend meghatározása után az egy adott hatókörhöz hozzárendelt: előfizetés, az erőforráscsoportot, vagy az erőforrás.

Házirendek, amelyek lehetővé teszik a minden részletre kiterjedő módjáról az esetek több műveletet kell végrehajtani. A műveletek a következők:

* **Megtagadási**: blokkolja az erőforrás-kérelem
* **Naplózási**: lehetővé teszi, hogy a kérelmet, de egy sort ad hozzá a tevékenységnapló (amely riasztások vagy runbookok indításához használható)
* **Hozzáfűzendő**: hozzáadja az erőforrás megadott adatokat. Például ha nem a "CostCenter" címke erőforráson, vegye fel a címkét az alapértelmezett értéket.

### <a name="common-uses-of-resource-manager-policies"></a>Gyakori használati módjai erőforrás-kezelő házirendek
Az Azure Resource Manager házirendek az Azure eszközkészlet egy hatékony eszköz. Lehetővé teszik váratlan költségek, a címkézés erőforrásoknak költséghely azonosításához, és győződjön meg arról, hogy a előírásainak követelmények teljesülnek-e elkerülése érdekében. Házirendek együtt a beépített naplózási szolgáltatásokat, ha akkor is fashion összetett és rugalmas megoldásokat. Szabályzatok lehetővé teszik a vállalatokkal, hogy a vezérlők biztosít a munkaterhelések "Hagyományos informatikai" és "Agile" munkaterhelések; például az ügyfél-alkalmazások fejlesztésével. A leggyakrabban használt minták házirendek látható a következők:

* **Földrajzi-megfelelőségi/adatok közös joghatóság alá** -Azure biztosít régiók keresztül történik. A vállalatok gyakran szeretne szabályozhatja, ahol erőforrások jönnek létre (hogy adatok közös joghatóság alá, vagy csak biztosításához erőforrások az erőforrások end fogyasztóinak közelében való létrehozása).
* **Költségkezelésére** -egy Azure-előfizetés számos típusok és a skála erőforrásai szerepelhetnek. Vállalatok gyakran szeretne győződjön meg arról, hogy standard előfizetések kerülje a feleslegesen erőforrásokat, amelyek is költségeket, de ténylegesen el egy hónapban, vagy több száz.
* **Alapértelmezett irányítás szükséges címkéket keresztül** -címkék használata a leggyakoribb és magas kívánt funkciók valamelyike. Azure Resource Manager házirendek segítségével a vállalatok egyre jobban tudja győződjön meg arról, hogy egy erőforrás címkéznie. A leggyakoribb címkék: részleg, az erőforrás tulajdonosa és a környezet típusát (például - éles, teszt, fejlesztési)

**Példák**

"Hagyományos informatikai" előfizetés-üzletági alkalmazások

* Az összes erőforrás részleg és a tulajdonos címkék kényszerítése
* Észak-amerikai régió erőforrás létrehozása korlátozása
* Korlátozása a abban, hogy G sorozatú virtuális gépek és a HDInsight-fürtök létrehozása

A felhőalapú alkalmazások létrehozása részleg "Gyors" környezet

* Adatok közös joghatóság alá követelményeinek, az erőforrások létrehozásának engedélyezése a csak egy adott régióban.
* Az összes erőforrás környezetcímke kényszerítése. Ha egy erőforrást egy címke nélkül jön létre, a **környezet: ismeretlen** címkén belül, hogy az erőforrás.
* Naplózási erőforrások Észak-Amerikában kívül jönnek létre, de nem akadályozzák meg.
* Naplózási nagy költségű erőforrások létrehozásakor.

> [!TIP]
> A leggyakrabban használt erőforrás-kezelő házirendek szervezet használata vezérlőhöz *ahol* erőforrásokat lehet létrehozni és *mi* típusú erőforrás hozhatók létre. A vezérlők biztosít mellett *ahol* és *mi*, számos vállalat házirendek használatával biztosíthatja a erőforrások tartoznak a megfelelő metaadatok vissza fogyasztás számlázási. Azt javasoljuk, hogy az előfizetés szintjén, a házirendek:
> 
> * Földrajzi-megfelelőségi/adatok közös joghatóság alá
> * Költségkezelés
> * Szükséges címkéket (üzleti igény, például a BillTo, az alkalmazás tulajdonosa azzal)
> 
> Hatókör alacsonyabb szinteken további házirendeket is alkalmazhat.
> 
> 

### <a name="audit---what-happened"></a>Naplózási – Mi történt?
Megtekintheti, hogyan működik-e a környezetében, felhasználói tevékenységet naplózni kell. A legtöbb erőforrástípusok esetében az Azure diagnosztikai naplók, amelyek a napló eszköz vagy az Azure Naplóelemzés létrehozása. Több előfizetések biztosításához egy részleg vagy vállalati nézet gyűjthet tevékenységi naplóit. Naplózási bejegyzések fontos diagnosztikai eszköz és a kritikus fontosságú eljárást, amely eseményindító eseményeket az Azure környezetben.

A Resource Manager üzembe helyezések tevékenységi naplóit lehetővé teszik a határozza meg a **műveletek** , amely tartott helye és végző felhasználók listáját. Tevékenységi naplóit is, majd összesíti a Naplóelemzési hasonló eszközökkel.

## <a name="resource-tags"></a>Az erőforráscímkék
Mivel a szervezet felhasználói erőforrások hozzáadása az előfizetéshez, lesz egyre fontosabb rendelni az erőforrások a megfelelő részleg, az ügyfél és a környezet. Metaadatok csatolhat erőforrásoknak [címkék](/azure/azure-resource-manager/resource-group-using-tags). Címkék használata az erőforrás vagy a tulajdonos ismertetik. Címkék lehetővé teszik a nem csak összesítése és csoportokat alkotnak az erőforrásokból különböző módokon, de az adatokat a jóváírási céljából. Legfeljebb 15 kulcs: érték párok rendelkező erőforrások jelölheti meg. 

Erőforrás-címkék használata rugalmas, és a legtöbb erőforrást kell csatlakoztatni. Közös erőforráscímkék többek között:

* BillTo
* Szervezeti egység (vagy üzleti egység)
* Környezet (éles ponton fejlesztés)
* Réteg (webes réteg, alkalmazás szint)
* Alkalmazás tulajdonosa
* Projektnév

![tags](./_images/resource-group-tagging.png)

Címkék további példákért lásd [Azure-erőforrások elnevezési szabályai ajánlott](../best-practices/naming-conventions.md).

> [!TIP]
> Vegye figyelembe, hogy így egy egykiszolgálós használatát írja elő, a címkézés házirendet:
> 
> * Erőforráscsoportok
> * Storage
> * Virtuális gépek
> * Alkalmazáskiszolgálók környezetek vagy webes szolgáltatás
> 
> A címkézési stratégia azonosítja az előfizetések között, milyen metaadat szükséges az üzleti, pénzügyi, biztonsági, kockázatkezelés és a környezet általános kezelése. 

## <a name="resource-group"></a>Erőforráscsoport
Erőforrás-kezelő lehetővé teszi, amelyre az erőforrások kezelése, számlázási vagy természetes affinitás jelentéssel bíró csoportokba. Amint azt korábban említettük, az Azure két üzembe helyezési modellel rendelkezik. A felügyeleti alapvető egysége a korábbi Klasszikus modell, az előfizetés volt. Előfizetés, amely nagy mennyiségű előfizetések létrehozásához vezetett erőforrások lebontva bonyolult volt. A Resource Manager modellt látott erőforráscsoportok bevezetése. Erőforráscsoportok olyan tárolók, az erőforrások, amelyek egy közös életciklussal vagy megosztani egy attribútum, például az "összes SQL-kiszolgáló" vagy "Alkalmazás A".

Erőforrás-csoportok nem lehetnek benne egymáshoz, és erőforrások csak egy erőforrás-csoporthoz is tartozhatnak. Bizonyos műveleteket egy erőforráscsoportban található összes erőforrást is alkalmazhat. Például egy erőforráscsoport törlése eltávolítja az erőforráscsoporton belül az összes erőforrást. Általában helyez egy teljes alkalmazás vagy a kapcsolódó rendszer ugyanabban az erőforráscsoportban. Például egy Contoso webalkalmazás nevű háromrétegű alkalmazás tartalmazná a webkiszolgálón, alkalmazás vagy az SQL server ugyanabban az erőforráscsoportban.

> [!TIP]
> Az erőforráscsoportok elrendezése változhat "Hagyományos informatikai" munkaterhelések "Gyors informatikai" munkaterhelések:
> 
> * "Hagyományos informatikai" munkaterhelések elemek belül az azonos életciklussal, például egy alkalmazás által leggyakrabban vannak csoportosítva. Csoportosítás alkalmazás lehetővé teszi, hogy az egyes Alkalmazáskezelés.
> * "Gyors informatikai" munkaterhelések az egyes külső ügyfélkapcsolati felhőalkalmazásokhoz összpontosítanak. Az erőforráscsoportok egyeznie kell a központi telepítést végez (például a webes réteg, App réteget) rétegei és kezelése.
> 
> A számítási feladatok ismertetése segít erőforrás csoport stratégia kidolgozása.

## <a name="role-based-access-control"></a>Szerepköralapú hozzáférés-vezérlés
Valószínűleg arra utasítja a saját kezűleg "akik hozzáférhetnek a erőforrások?" és "hogyan kezelheti a hozzáférést?" Engedélyezése vagy letiltása az Azure-portálon való hozzáférés és a portálon erőforrásokhoz való hozzáférés szabályozása elengedhetetlen. 

Ha Azure jelent, hozzáférés-vezérlést a előfizetés voltak: rendszergazdának vagy társadminisztrátornak. A portál az erőforrások eléréséhez a Klasszikus modell hallgatólagos előfizetésre való hozzáférést. Ez részletesebb vezérlést nyújtanak a megfelelő hozzáférés-vezérlés az Azure-tagság a kereslet az olyan előfizetések elterjedése hiánya.

Ez a előfizetések elterjedése már nem szükséges. Szerepköralapú hozzáférés-vezérlést felhasználókat rendelhet szabványos szerepkörök (például a szerepkörök közös "olvasó" és "író" típusok). Egyéni szerepkörök is meghatározhat.

> [!TIP]
> Szerepköralapú hozzáférés-vezérlés megvalósításához:
> * A vállalati identitás tárolására (leggyakrabban az Active Directory) Azure Active Directoryhoz csatlakozni az AD Connect eszköz használatával.
> * A rendszergazdai vagy Társadminisztrátori felügyelt identitásával előfizetés szabályozzák. **Nem** rendszergazdai vagy társadminisztrátori hozzárendelése egy új előfizetés tulajdonosa. Ehelyett az RBAC-szerepkörök használatával biztosíthat **tulajdonos** jogosultsága ahhoz, hogy egy csoport vagy egy személy.
> * Azure-felhasználók felvétele csoportba (például X Alkalmazástulajdonosok) az Active Directoryban. A szinkronizált csoport segítségével adja meg a csoport tagjai az alkalmazást tartalmazó erőforráscsoportot kezeléséhez szükséges engedélyekkel.
> * Biztosítása elve kövesse a **legalacsonyabb jogosultsági szint** a várt munkához szükséges. Példa:
>   * Üzembe helyezési csoport: Egy csoport, amely csak akkor tudja telepíteni az erőforrásokat.
>   * Virtuálisgép-kezelő: Egy csoportot, amely képes indítsa újra a virtuális gépek (műveletek)
> 
> Ezek a tippek segítségével felügyelheti a felhasználók hozzáférését az előfizetésből.

## <a name="azure-resource-locks"></a>Azure-erőforrás zárolások feloldása
A szervezet alapvető szolgáltatások ad hozzá az előfizetés, mert lesz egyre fontosabb annak érdekében, hogy ezek a szolgáltatások elérhetők-e üzleti megszűnésének megelőzése érdekében. [Erőforrás zárolások](/azure/azure-resource-manager/resource-group-lock-resources) engedélyezi, hogy hol módosítsa vagy törölje azokat az alkalmazások és a felhőalapú infrastruktúra jelentős hatással van a jó minőségű erőforrásokon végrehajtott műveletek korlátozása. Zárolások alkalmazhat egy előfizetés, az erőforráscsoportot, vagy az erőforrás. Általában zárolások alkalmazása eligazodást erőforrások, például a virtuális hálózatok, az átjárók és a storage-fiókok. 

Erőforrás zárolások jelenleg két értéket támogatja: CanNotDelete és csak olvasható. CanNotDelete azt jelenti, hogy a felhasználói (a megfelelő jogosultságokkal) is még olvasni vagy módosítani az erőforrást, de nem lehet törölni. Csak olvasható azt jelenti, hogy a jogosult felhasználók nem lehet törölni vagy módosítani az erőforrást.

Hozzon létre, vagy törölje a felügyeleti zárolás, hozzáféréssel kell rendelkeznie `Microsoft.Authorization/*` vagy `Microsoft.Authorization/locks/*` műveletek.
A beépített szerepkörök csak a tulajdonos és a felhasználói hozzáférés adminisztrátora kapnak ezeket a műveleteket.

> [!TIP]
> Alapvető hálózati beállítások zárral kell védeni. Átjáró véletlen törlés telephelyek közötti VPN lenne katasztrofális Azure-előfizetéshez. Azure nem engedélyezi, hogy törli a virtuális hálózatot, amely használatban van, de további korlátozásokat alkalmaz hasznos elővigyázatosságból. 
> 
> * Virtuális hálózat: CanNotDelete
> * Hálózati biztonsági csoport: CanNotDelete
> * Házirendek: CanNotDelete
> 
> Házirendek karbantartására megfelelő vezérlők is nélkülözhetetlenek. Azt javasoljuk, hogy alkalmazza a **CanNotDelete** zárolást szerezni a szabályzatok, amely használatban van.

## <a name="core-networking-resources"></a>Alapvető hálózati erőforrások
Erőforrásokhoz való hozzáférést lehet (a vállalati hálózatból) belső vagy külső (internetes) keresztül. Is egyszerűen a szervezet felhasználói erőforrások véletlenül be a megfelelő hellyel, és potenciálisan nyissa meg ezeket a rosszindulatú hozzáférésektől. Csakúgy, mint a helyszíni eszközök, a vállalatok megfelelő ellenőrzéssel győződjön meg arról, hogy Azure-felhasználók a megfelelő döntéseket kell hozzáadnia. A előfizetés irányítás azt határozza meg, amely biztosítja a alapvető hozzáférés-vezérlést alapvető erőforrásai. Az alapvető erőforrások foglalják magukban:

* **Virtuális hálózatok** alhálózatok tároló objektumok. Bár nem feltétlenül szükséges, gyakran használják, ha van alkalmazások belső vállalati erőforrásokhoz való csatlakozás.
* **Hálózati biztonsági csoportok** tűzfal hasonló, és adjon meg szabályokat, a hogyan erőforrás "működik" a hálózaton keresztül. Hogyan részletes szabályozását biztosítják / Ha egy alhálózat (vagy a virtuális gép) csatlakozni tud az internethez vagy más alhálózatok ugyanabban a virtuális hálózatban.

![hálózati szolgáltatásmag](./_images/core-network.png)

> [!TIP]
> Hálózati:
> * Külső hálózat irányába néző munkaterhelések és a belső hálózati munkaterhelések virtuális hálózatok létrehozása. Ennek a megközelítésnek az esélyét, hogy véletlenül helyezi el a virtuális gépek esetén, amelyeket külső irányuló térben belső munkaterhelések csökkenti.
> * Konfigurálja a hálózati biztonsági csoportokkal való hozzáférés korlátozására. Legalább az interneten történő hozzáférésének blokkolását, a belső virtuális hálózatok, és a vállalati hálózathoz történő hozzáférésének blokkolását, a külső virtuális hálózatok.
> 
> Ezek a tippek segítségével megvalósíthatja a biztonságos hálózati erőforrásokhoz.

### <a name="automation"></a>Automation
Erőforrások kezelése külön-külön csak időigényes és potenciálisan nagyon eséllyel fordulnak elő bizonyos műveletek hiba. Azure Azure Automation, a Logic Apps és az Azure Functions különböző automatizálási képességeket nyújt. [Azure Automation szolgáltatásbeli](/azure/automation/automation-intro) lehetővé teszi a rendszergazdáknak létrehozni és meghatározni a runbookok számára az erőforrások kezelése a gyakori feladatokat. A runbookok vagy egy PowerShell-kód szerkesztővel, vagy egy grafikus szerkesztő segítségével hoz létre. Összetett többlépcsős munkafolyamatokat hozhat létre. Azure Automation szolgáltatásbeli gyakran gyakori feladatokat, mint a nem használt erőforrások leállítása vagy az erőforrások létrehozása egy adott eseményindító válaszul emberi beavatkozás anélkül kezelésére használt.

> [!TIP]
> Az automatizáláshoz:
> * Egy Azure Automation-fiók létrehozásához, és tekintse át a rendelkezésre álló runbookok (grafikus és a parancs sor) érhető el a [forgatókönyvek](/azure/automation/automation-runbook-gallery).
> * Importálja és testre szabhatja a legfontosabb forgatókönyvek saját használatra.
> 
> Egy általános forgatókönyv azt a képességet indítási/leállítási virtuális gépek ütemezés szerint. Nincsenek példa runbookok érhetők el a gyűjteményben, amely ebben a forgatókönyvben kezelni és szól, hogyan bontsa ki.
> 
> 

## <a name="azure-security-center"></a>Azure Security Center
Lehet, hogy a legnagyobb blockers elfogadása felhőbe egyik volt fontos szempont biztonsági. Informatikai kockázat vezetők és biztonsági osztályok kell, és győződjön meg arról, hogy az Azure-erőforrások biztonságos. 

A [az Azure Security Center](/azure/security-center/security-center-intro) központi az előfizetések erőforrások biztonsági állapotát jeleníti meg, és ajánlásokat, amelyekkel megakadályozható a fertőzött erőforrásokat. Ez részletesebb házirendek (például fog alkalmazni házirendeket, amelyek lehetővé teszik a vállalat személyessé tétele érdekében azok állapotát a kockázat, azok címzési adott erőforráscsoportban) engedélyezéséhez. Végül az Azure Security Center, amely lehetővé teszi a Microsoft-partnerek és a független szoftvergyártók létrehozása az Azure Security Center javítása érdekében a képességei keresztül csatlakozó szoftver nyílt platformon. 

> [!TIP]
> Az Azure Security Center az egyes előfizetések alapértelmezés szerint engedélyezve van. Azonban engedélyeznie kell a virtuális gépek számára engedélyezi a az ügynök telepítése, és megkezdi az adatgyűjtést az Azure Security Center adatok gyűjtése.
> 
> ![Adatok gyűjtése](./_images/data-collection.png)
> 
> 

## <a name="next-steps"></a>További lépések
* Most, hogy az előfizetés irányítás megismerte felderítheti, hogy ezek az ajánlások a gyakorlatban is. Lásd: [Azure-előfizetés irányítás implementációi](subscription-governance-examples.md).
