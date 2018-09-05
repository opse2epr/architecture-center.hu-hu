---
title: Ajánlott eljárások Azure-bA nagyvállalatok számára
description: A vállalatok számára is annak biztosítására, biztonságos és kezelhető környezetet használó scaffold ismerteti.
author: rdendtler
ms.date: 03/31/2017
ms.author: rodend;karlku;tomfitz
ms.openlocfilehash: 91adce796ae7785d3831e9628fce0193076eec9b
ms.sourcegitcommit: f4069cf68456b5c74acb1b890dc4e45e11f12b59
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 09/04/2018
ms.locfileid: "43675783"
---
# <a name="azure-enterprise-scaffold---prescriptive-subscription-governance"></a>Az Azure enterprise scaffold - előíró előfizetés-irányítás
Vállalatok egyre inkább vannak bevezetése a nyilvános felhő rugalmasságát és rugalmasságot biztosít. Azok az igénybe vett bevételi lehetőségeket, vagy a vállalati erőforrások optimalizálása a felhőalapú erősségeit. Microsoft Azure lehetőséget kínál számos, hogy a vállalatok számára például a számítási feladatok és alkalmazások széles választékának cím építőelemeket állíthatnak össze. 

Azonban, hogy hol kezdjenek gyakran nehéz. Miután eldöntötte használható az Azure, néhány kérdést gyakran felmerülő:

* "Hogyan megfelelnek az egyes országokban az adatok elkülönítése a jogi követelmények?"
* "Hogyan tegye biztosítható, hogy valaki nem véletlenül módosítja olyan kritikus rendszererőforrásokat?"
* "Hogyan tudom mi minden erőforrás támogatja az így I számlája, és pontosan vissza számláját?"

A potenciális a nincs guard rails-üres előfizetés tűnhet. Az üres helyet a áthelyezése az Azure-ba is akadályozhatják.

Ez a cikk címe kell irányítási és elosztása, rugalmasságot és műszaki szakembereknek szánt kiindulási pontot biztosít. Ez bemutatja a, az enterprise scaffold, amely a megvalósítása és kezelése az Azure-előfizetései szervezetek. 

## <a name="need-for-governance"></a>Cégirányítási szükség
Az Azure-ba történő áthelyezésekor kell venni a cégirányítási elején annak biztosítása érdekében a vállalaton belül a felhő sikeres használatát, a témakörben. Sajnos az idő és a egy átfogó cégirányítási rendszer létrehozásának bürokráciával azt jelenti, hogy egyes üzleti csoportok közvetlenül léphet gyártók vállalati informatikai közreműködése nélkül. Ez a megközelítés hagyhatja, a vállalati nyissa meg a biztonsági réseinek Ha az erőforrások megfelelően nem kezelt. A nyilvános felhőben – rugalmasságot, a rugalmasságot és fogyasztásalapú díjszabás – mutatókat fontos üzleti csoportokat szeretne gyorsan teljesítheti az ügyfelek (belső és külső) igényeinek megfelelően. De vállalati adatok és rendszerek hatékonyan védelmének biztosításához szükséges.

A valós életben szerkezetkialakító segítségével hozza létre a struktúra. A scaffold végigvezeti az általános elvet követik, és állandó rendszerek csatlakoztatnia kell a forráshorgony pontokat biztosít. Egy enterprise scaffold ugyanez: rugalmas vezérlők és az Azure-képességek a környezetben, és a központi jellegűek struktúrát biztosító a nyilvános felhő alapú szolgáltatások. A kapcsolat építői biztosít (informatikai és üzleti csoportok) létrehozásához és csatlakoztatásához az új szolgáltatások alapját.

A scaffold tudjuk a különböző fürtméretekkel járó az ügyfelek számos marketingmódszerek kigyűjtötte tanácsok alapul. Ezen ügyfelek köre a Fortune 500-as vállalatok és a független szoftvergyártók a felhőbeli megoldások fejlesztésével és migrálás vannak a felhőben történő megoldásfejlesztéshez kis szervezetek. Az enterprise scaffold "célirányosan" rugalmas hagyományos informatikai számítási feladatok és a rugalmas számítási feladatok; támogatásához szoftver--szolgáltatásként (SaaS) alkalmazásokat létrehozó fejlesztők, mint például az Azure képességeken alapulnak.

Az enterprise scaffold minden új előfizetés Azure-ban alapjául szolgál. Ez lehetővé teszi, hogy a rendszergazdáival együttműködve biztosítják a számítási feladatok megfelelnek a minimális cégirányítási követelmények a szervezet meggátolja, hogy üzleti csoportokat vagy a fejlesztők gyorsan felel meg a saját céljainak nélkül.

> [!IMPORTANT]
> Cégirányítási elengedhetetlen a sikeres Azure. Ez a cikk egy enterprise scaffold technikai végrehajtásának célozza, de csak éri el a szélesebb körű folyamat és az összetevők közötti kapcsolatok. A házirend irányítási folyamatok felülről lefelé, és határozza meg az üzleti szeretné elérni. Természetesen magában foglalja a cégirányítási modell létrehozása az Azure-ban képviselői az informatikai, de ami még fontosabb Cégvezetők csoport és a biztonság és a kockázatkezelés erős ábrázolás kell rendelkeznie. A végén az enterprise scaffold veszélyeztetettségének üzleti megkönnyítése érdekében a szervezet céljaira és célok van.
> 
> 

Az alábbi képen a scaffold összetevőit ismerteti. A foundation egy alapos terv, szervezeti egységek, a fiókok és előfizetések támaszkodik. A területei legyenek elérhetők a Resource Manager-házirendek és erős elnevezési szabványait áll. A scaffold a többi Azure-képességek core származik, és funkcióit, hogy engedélyezze a biztonságos és kezelhető környezet.

![scaffold összetevők](./_images/components.png)

> [!NOTE]
> Azure gyors növekedésnek 2008 bevezetése óta. Ennek a növekedésnek a Microsoft mérnöki csapataival, kezeléséhez és a szolgáltatások üzembe helyezése a megközelítés újragondolja szükséges. Az Azure Resource Manager-modell 2014-ben jelent meg, és lecseréli azokat a klasszikus üzemi modellben. Resource Manager lehetővé teszi a szervezetek számára, hogy könnyebben üzembe helyezése, rendszerezheti és felügyelete az Azure-erőforrások. Erőforrás-kezelő ezerszer tartalmaz, az összetett, saját megoldások gyorsabb központi telepítés létrehozásakor. Részletes hozzáférés-vezérlés, valamint az erőforrások megcímkézését metaadatokkal is tartalmaz. A Microsoft azt javasolja, hogy létrehozott összes erőforrást a Resource Manager modellel. Az enterprise scaffold explicit módon a Resource Manager-modell lett tervezve.
> 
> 

## <a name="define-your-hierarchy"></a>Adja meg a hierarchiában
A scaffold alapjaira az Azure nagyvállalati beléptetés (és a vállalati portál). A nagyvállalati beléptetés meghatározza az alakzatot, és használja az Azure-szolgáltatások a vállalaton belül, és cégirányítási alapvető struktúráját. A nagyvállalati szerződés keretében belüli ügyfelek képesek vertikálisan tovább particionálhatja a részlegek, fiókok, és végül előfizetések környezetét. Azure-előfizetéssel az alapszintű egység, hol található összes erőforrást. Azure-magok, erőforrások és egyéb száma például számos korlátait is meghatározza.

![hierarchia](./_images/agreement.png)

Minden vállalat különböző, és az előző ábrán a hierarchia lehetővé teszi, hogy jelentős rugalmasságot nyújt, az Azure hogyan vannak rendszerezve a vállalaton belül. Ez a dokumentum hasonlíthatók-k megvalósítása előtt kell modellezheti a hierarchiában, és a számlázás, az erőforrás-hozzáférés és az összetettség gyakorolt hatás megértéséhez.

Az Azure-regisztrációk három gyakori minta a következők:

* A **működési** minta
  
    ![funkcionális](./_images/functional.png)
* A **üzleti egység** minta 
  
    ![vállalata számára](./_images/business.png)
* A **földrajzi** minta
  
    ![földrajzi](./_images/geographic.png)

Alkalmazza a scaffold kiterjeszteni a cégirányítási követelményeket a vállalat az előfizetés az előfizetési szinten.

## <a name="naming-standards"></a>Elnevezési szabályai
Az első oszlop a scaffold, az elnevezési szabványoknak. Jól megtervezett elnevezési szabványait lehetővé teszi a portálon, a számlán, és parancsfájlok belüli erőforrások azonosítására. Valószínűleg már rendelkezik a helyszíni infrastruktúrára vonatkozó szabványok elnevezési. Azure ad környezetében, amikor az Azure-erőforrások, ki kell elnevezési szabványokat. A környezet minden szinten hatékonyabb felügyelet elnevezési szabványt megkönnyítése.

> [!TIP]
> Az elnevezési konvenciók:
> * Tekintse át, és ahol lehetséges elfogadja a [Patterns and practices nevű útmutató](../../best-practices/naming-conventions.md). Ez az útmutató segítségével könnyebben meghatározhatja az olyan jelentéssel bíró elnevezési konvenciót.
> * Használjon camelCasing nevét erőforrásokhoz (például: myResourceGroup és vnetNetworkName). Megjegyzés: Nincsenek bizonyos erőforrások, például a storage-fiókok, ahol az egyetlen lehetősége: kisbetű (és egyetlen más speciális karakter).
> * Érdemes lehet elnevezési érvényesíthet (a következő szakaszban leírt) Azure Resource Manager-házirendek használatával.
> 
> Az előző tippek egy egységes kulcselnevezési konvenció megvalósításához nyújtanak segítséget.

## <a name="policies-and-auditing"></a>Szabályzatok és naplózás
A második pillar, a scaffold magában foglalja a létrehozás [Azure házirendek](/azure/azure-policy/azure-policy-introduction) és [naplózás a tevékenységnapló](/azure/azure-resource-manager/resource-group-audit). Resource Manager-házirendek biztosít az Azure-ban kockázat kezelését. Megadhatja, hogy az adatok elkülönítése korlátozása, kényszerítése és bizonyos műveletek naplózási házirendeket. 

* A házirend az alapértelmezett **engedélyezése** rendszer. Műveletek definiálása és erőforrásokat, amelyek megtagadása vagy az erőforrásokon végzett műveletek naplózásához a szabályzatok hozzárendelését, szabályozhatja.
* Szabályzatok a szabályzatdefiníció a szabályzat adatdefiníciós nyelv (if-majd feltételek) ismerteti.
* JSON (Javascript Object Notation) formátumban fájlokkal kell létrehoznia szabályzatokat. Házirend meghatározása után, rendelje hozzá egy adott hatókörhöz: előfizetés, erőforráscsoport vagy erőforrás.

Szabályzatokkal rendelkezik, amelyek lehetővé teszik a minden részletre kiterjedő megközelítés a forgatókönyvek a több művelet. A műveletek a következők:

* **Megtagadási**: letiltja az erőforrás-kérelem
* **Naplózási**: lehetővé teszi, hogy a kérelmet, de egy sort ad hozzá a tevékenységnapló (amely a riasztásokat, illetve runbookok való használható)
* **Hozzáfűzés**: hozzáadja a megadott adatokat az erőforráshoz. Például ha ott nem egy "CostCenter" címkét egy erőforráson, adja hozzá az adott címkével, alapértelmezett értékkel.

### <a name="common-uses-of-resource-manager-policies"></a>Gyakori használati Resource Manager-házirendek
Az Azure Resource Manager-házirendek az Azure-eszközkészlet egy hatékony eszköz. Lehetővé teszik, hogy az erőforrások címkézése keresztül költséghely azonosításához, és győződjön meg arról, hogy a megfelelőség érdekében teljesülnek-e a váratlan költségek elkerülése érdekében. Szabályzatok a beépített naplózási szolgáltatások vannak kombinálva, akkor is fashion összetett és rugalmas megoldások. Szabályzatok lehetővé teszik a cégeket azzal, hogy "A hagyományos informatikai" és "Agile" számítási feladatok; vezérlők például az ügyfél alkalmazások fejlesztésével. A leggyakrabban használt minták házirendek láthatjuk a következők:

* **GEO-megfelelőségi/adatszuverenitás** – az Azure biztosít régiókban világszerte. Vállalatok milyen gyakran szeretne szabályozhatja, ahol erőforrás jön létre (hogy az adatok elkülönítése, vagy csak az erőforrások teljes felhasználói számára az erőforrások közelében való létrehozása érdekében).
* **Költségkezelés** – Azure-előfizetés számos típusú és méretezési erőforrások is tartalmazhat. Vállalatok milyen gyakran szeretne győződjön meg arról, hogy standard szintű kerülje a szükségtelenül nagy erőforrásokat, amelyek dollárban egy hónapban, vagy több száz költséggel jár.
* **Szükséges címke keresztül cégirányítási alapértelmezett** -címkék igénylő az egyik leggyakoribb és magas kívánt funkcióinak előnyeit. Az Azure Resource Manager-házirendek használatával vállalatok tudnak győződjön meg arról, hogy egy erőforrás megfelelően van-e megjelölve. A leggyakoribb címke található: részleg, az erőforrás tulajdonosa és a környezet típusát (például: éles, teszt, fejlesztői)

**Példák**

"A hagyományos informatikai" előfizetés-üzleti alkalmazások

* Szervezeti és tulajdonosi címkék az összes erőforrás kényszerítése
* Az Észak-amerikai régió erőforrás létrehozásának korlátozása
* A G sorozatú virtuális gépek és a HDInsight-fürtök létrehozása korlátozása

"Agilis" környezetet egy üzleti egységet, a felhőalapú alkalmazások létrehozása

* Megfelel az adatszuverenitási követelményekhez, engedélyezze az erőforrások létrehozását csak egy adott régióban.
* Az összes erőforrás környezetcímke kényszerítése. Ha egy erőforrás létrehozása nélkül a címke fűzze hozzá a **környezet: ismeretlen** címkét az erőforráshoz.
* Naplózási erőforrások jönnek létre Észak-Amerikán kívüli, de nem akadályozzák meg.
* Naplózási nagy költségű erőforrások létrehozásakor.

> [!TIP]
> A leggyakoribb szervezetenként Resource Manager-házirendek használata a vezérlőbe *ahol* erőforrásokat lehet létrehozni és *mi* típusú erőforrások hozhatók létre. A vezérlők biztosítása mellett *ahol* és *mi*, sok vállalat rendelkezzenek erőforrások a megfelelő metaadatok vissza használatalapú számlázási szabályzatok használatával. Javasoljuk, hogy eszközszinten alkalmazni házirendeket az előfizetés szintjén:
> 
> * GEO-megfelelőségi/adatok elkülönítése
> * Költségkezelés
> * Szükséges címke (azzal a üzleti igények, például a BillTo, az alkalmazás tulajdonosa)
> 
> Hatókör alacsonyabb szinten további házirendeket is alkalmazhat.
> 
> 

### <a name="audit---what-happened"></a>Naplózási – Mi történt?
Megtekintheti, hogyan működik a környezetben, felhasználói tevékenységet naplózni kell. Azure-ban a legtöbb erőforrástípusok diagnosztikai naplók, amelyet elemezhet egy napló eszköz vagy az Azure Log Analyticsben hozzon létre. Tevékenységnaplók több előfizetések biztosít egy részleg vagy vállalati nézet gyűjthet. A naplórekordok olyan fontos diagnosztikai eszköz és a egy kulcsfontosságú mechanizmus, amellyel Azure-beli kiváltó események.

Resource Manager üzembe helyezések műveletnaplóinak alapján megállapíthatja, hogy engedélyezze a **műveletek** , helye és ki végzett vett igénybe. Tevékenységnaplók gyűjthetők össze, és összesítve, eszközök, mint például a Log Analytics használatával.


## <a name="resource-group"></a>Erőforráscsoport
Resource Manager lehetővé teszi, hogy a felügyeleti, számlázási vagy természeti affinitásra jelentéssel bíró csoportokba erőforrásokat helyezze. Ahogy korábban említettük, az Azure két üzembe helyezési modellel rendelkezik. A korábbi Klasszikus modell esetében alapvető egysége volt az előfizetés. Előfizetés, amely nagy mennyiségű előfizetések létrejöttét vezetett belüli erőforrások felosztania nehéz volt. A Resource Manager-modell látott erőforráscsoportok bevezetését. Erőforráscsoportok olyan tárolók, az erőforrások, amelyek egy közös életciklussal vagy megosztani egy attribútum, például a "minden SQL-kiszolgáló" vagy "Egy alkalmazás".

Erőforrás-csoportok nem lehetnek benne egymást, és az erőforrások csak egy erőforráscsoporthoz is tartozhatnak. Bizonyos műveleteket alkalmazhat az összes erőforrás egy erőforráscsoportban. Például egy erőforráscsoport törlése eltávolítja az összes erőforrást az erőforráscsoporton belül. Általában helyez egy teljes alkalmazás vagy a kapcsolódó rendszer ugyanabban az erőforráscsoportban. Például egy háromszintű alkalmazás webalkalmazás Contoso nevű tartalmazná a webkiszolgáló, kiszolgáló és az SQL server ugyanabban az erőforráscsoportban.

> [!TIP]
> Miként célszerű megszerveznie az erőforráscsoportok eltérhet "Hagyományos informatikai" számítási "Agilis informatikai" számítási feladatokhoz:
> 
> * "A hagyományos informatikai" számítási feladatok leggyakrabban csoportosítva elemei ugyanaz az életciklusuk, ilyen lehet például egy alkalmazás. Alkalmazás által a csoportosítás lehetővé teszi egyéni alkalmazás felügyeletét.
> * "Agilis informatikai" számítási feladatok általában a külső ügyfelek által használt felhőalkalmazások összpontosíthat. Az erőforráscsoportok tükrözik a rétegek (például a webes szint, App-réteget) központi telepítés és felügyelet.
> 
> A számítási feladatok ismertetése segíti erőforrás-csoport stratégia.


## <a name="resource-tags"></a>Erőforráscímkék
A szervezet felhasználói erőforrások hozzáadása az előfizetéshez, ahogy azt rendelni az erőforrások a megfelelő részleg, a vevő és a környezet egyre fontosabbá válik. Metaadatok csatlakoztathat erőforrásai a [címkék](/azure/azure-resource-manager/resource-group-using-tags). Címkék használatával az erőforrás vagy a tulajdonos kapcsolatos adatok megadása. A címkék lehetővé teszi csak nem aggregált és csoportosíthatók az erőforrások számos lehetőséget kínál, de az adatokat a jóváírási célokra. Megjelölheti 15 kulcs: érték párok az erőforrásokat. 

Erőforráscímkék rugalmasak, és a legtöbb erőforrást kell csatlakoztatni. Gyakori erőforrás-címkék példák:

* BillTo
* Szervezeti egység (vagy üzleti egység)
* Környezet (éles környezetben, a fázis, fejlesztés)
* Szint (webes réteg, alkalmazásrétegek)
* Alkalmazás tulajdonosa
* Projektnév

![tags](./_images/resource-group-tagging.png)

A címkék további példákért lásd [ajánlott az Azure-erőforrások elnevezési konvenciói](../../best-practices/naming-conventions.md).

> [!TIP]
> Vegye figyelembe, hogy egy szabályzatot, amely az előírásoknak, a címkézés teszi:
> 
> * Erőforráscsoportok
> * Storage
> * Virtuális gépek
> * Alkalmazás-Service-környezetek/webes kiszolgálók
> 
> A címkézési stratégia azonosítja az előfizetések között milyen metaadatok az üzleti, pénzügyi, biztonsági, kockázatkezelési és a környezet általános felügyeletéhez szükséges. 



## <a name="role-based-access-control"></a>Szerepköralapú hozzáférés-vezérlés
Valószínűleg arra utasítja a saját maga "ki kell erőforrásokhoz való hozzáférés?" és a "hogyan szabályozhatja a hozzáférést?" Így vagy letiltva a hozzáférés az Azure Portalra, és erőforrások a portálon való hozzáférés szabályozása alapvető fontosságú. 

Azure jelent, ha egy előfizetés hozzáférés-vezérlést voltak: rendszergazdaként vagy társ-rendszergazdaként. A portálon erőforrások eléréséhez a klasszikus modellt hallgatólagos az egy előfizetéshez való hozzáférést. A részletesebb vezérlés hiánya való megfelelő hozzáférés-vezérlést biztosítanak egy adott Azure-regisztráció előfizetések elterjedése vezetett.

Az előfizetések elterjedése már nincs rá szükség. A [szerepköralapú hozzáférés-vezérlés](/azure/role-based-access-control/overview), felhasználók hozzárendelése standard szerepkörök (például a szerepkörök közös "olvasó" és "író" típusú). Egyéni szerepköröket is meghatározhat.

> [!TIP]
> Szerepköralapú hozzáférés-vezérlés megvalósításához:
> * Csatlakozzon a vállalati identitás tárolására (leggyakrabban az Active Directory) az Azure Active Directory az AD Connect eszköz használatával.
> * A rendszergazda/Társadminisztrátor egy felügyelt identitással előfizetés szabályozza. **Nem** rendszergazda/társadminisztrátor hozzárendelése egy új előfizetés tulajdonosa. Ehelyett használjon biztosításához RBAC-szerepkörök **tulajdonosa** személy vagy csoport jogait.
> * Az Azure-felhasználók felvétele csoportba (például az alkalmazás X tulajdonosai) az Active Directoryban. A szinkronizált csoport használatával adja meg a csoport tagjai az erőforráscsoport, amely tartalmazza az alkalmazás kezelése megfelelő jogosultságokkal.
> * Hajtsa végre az alapelvnek megadását, a **legalacsonyabb jogosultsági** a várt munkához szükséges. Példa:
>   * Üzembe helyezési csoport: Egy csoportot, amely csak helyezhet üzembe erőforrásokat.
>   * Virtuálisgép-kezelő: Egy csoportot, amely képes virtuális gépek újraindítása (Ha a műveletek)
> 
> Ezek a tippek segítséget nyújt a felhasználói hozzáférés felügyelete az előfizetésében.

## <a name="azure-resource-locks"></a>Az Azure erőforrás-zárolások
A szervezet alapvető szolgáltatásához ad hozzá az előfizetés, mert azt annak érdekében, hogy ezek a szolgáltatások rendelkezésre álló üzleti fenntartásához egyre fontosabbá válik. [Erőforrás-zárolások](/azure/azure-resource-manager/resource-group-lock-resources) korlátozhatja a műveletek az értékes erőforrásokat, módosítsa vagy törölje őket jelentős hatást gyakorol az alkalmazások és a felhő-infrastruktúra rendelkezik. Egy előfizetés, erőforráscsoport vagy erőforrás zárolása alkalmazhat. Zárolások általában alapvető erőforrások, például a virtuális hálózatok, az átjárók és a storage-fiókok a alkalmazni. 

Erőforrás-zárolások jelenleg támogatja a két érték: védve, és csak olvasható. Védve azt jelenti, hogy a felhasználók (a szükséges engedélyekkel) is olvasni vagy módosítani az erőforrást, de nem lehet törölni. Csak olvasható azt jelenti, hogy a jogosult felhasználók nem lehet törölni vagy módosítani az erőforrást.

Hozzon létre vagy felügyeleti zárolások törlése, hozzáféréssel kell rendelkeznie `Microsoft.Authorization/*` vagy `Microsoft.Authorization/locks/*` műveleteket.
A beépített szerepkörök csak a tulajdonos és a felhasználói hozzáférés rendszergazdája megkapják ezeket a műveleteket.

> [!TIP]
> Alapvető hálózati beállítások zárolásokat kell védeni. Az átjáró véletlen törlését, site-to-site VPN lenne katasztrofális Azure-előfizetéssel. Az Azure nem engedélyezi, hogy a használatban lévő virtuális hálózat törlése, de további korlátozásokat alkalmaz egy hasznos eszközeikről. 
> 
> * Virtuális hálózat: védve
> * Hálózati biztonsági csoport: védve
> * Házirendek: védve
> 
> Szabályzatok is olyan kulcsfontosságúak a megfelelő ellenőrzéseket fenntartását. Azt javasoljuk, hogy a alkalmazni egy **védve** zárolást házirendeket használt.

## <a name="core-networking-resources"></a>Alapvető hálózati erőforrásai
Erőforrásokhoz való hozzáférés lehet (a vállalati hálózaton belül) belső vagy külső (az interneten) keresztül. A felhasználók a szervezet véletlenül a nem megfelelő helyszíni helyezni erőforrásokat, és potenciálisan rosszindulatú hozzáférés meg is gyerekjáték. Csakúgy, mint a helyszíni eszközök, vállalatok hozzá kell adnia a megfelelő szabályozásokkal győződjön meg arról, hogy az Azure-felhasználók a helyes döntések meghozatalában. Az előfizetés-irányítás hogy ezzel a alapvető erőforrások egyszerű hozzáférés vezérlése érdekében. Az alapvető erőforrások állnak:

* **Virtuális hálózatok** alhálózatok tároló objektumok. Bár ez nem feltétlenül szükséges, azt gyakran használják alkalmazások belső vállalati erőforrásokhoz való csatlakozáskor.
* **Hálózati biztonsági csoportok** hasonlóak a tűzfalat, és hogyan erőforrás "beszélhetünk" a hálózaton keresztül szabályokat adja meg. Hogyan szabályozható nyújtanak / Ha egy alhálózat (vagy a virtuális gép) kapcsolódhatnak az interneten vagy más alhálózatok ugyanabban a virtuális hálózatban.

![hálózati szolgáltatásmag](./_images/core-network.png)

> [!TIP]
> A hálózatkezelés:
> * Kívülre irányuló számítási feladatok és belső számítási feladatok számára dedikált virtuális hálózatok létrehozásához. Ez a megközelítés csökkenti az esélyét, hogy véletlenül a belső, egy külső irányuló tárhelyen lévő munkaterhelések szánt virtuális gépek elhelyezése.
> * Konfigurálja a hálózati biztonsági csoportok a hozzáférés korlátozásához. Legalább a belső virtuális hálózatokról az interneten letiltására, és megakadályozza a hozzáférést a vállalati hálózathoz külső virtuális hálózatokat.
> 
> Ezek a tippek biztonságos hálózati erőforrások megvalósításához nyújtanak segítséget.

## <a name="automation"></a>Automation
Az erőforrások felügyelete a külön-külön is időigényes és potenciálisan hibalehetőségeket rejt magában az egyes műveletek. Az Azure a többek között az Azure Automation, a Logic Apps és az Azure Functions különböző automatizálási képességeket nyújt. [Az Azure Automation](/azure/automation/automation-intro) lehetővé teszi a rendszergazdáknak létrehozni és meghatározni a runbookok az erőforrások felügyelete a gyakori feladatok kezeléséhez. Runbookok vagy egy PowerShell Kódszerkesztő, vagy egy, a grafikus szerkesztő használatával hoz létre. Összetett többlépcsős munkafolyamatokat hozhat létre. Az Azure Automation gyakori feladatokat, mint a nem használt erőforrásokat leállítása, vagy -erőforrások létrehozását egy adott eseményindító válaszul emberi beavatkozás nélkül kezeléséhez gyakran használják.

> [!TIP]
> Az automation:
> * Azure Automation-fiók létrehozása, és tekintse át a rendelkezésre álló runbookok (grafikus és a parancsot. sor) érhető el a [forgatókönyv-katalógusában](/azure/automation/automation-runbook-gallery).
> * Importálhatja, és testre szabhatja a saját használatra főbb forgatókönyvek.
> 
> Egy általános forgatókönyv elindítása/leállítása virtuális gépek ütemezés szerint lehetősége. Nincsenek példa runbookok érhetők el a katalógus, amely ebben a forgatókönyvben kezelni és arról szól, hogyan annak kibontásához.
> 
> 

## <a name="azure-security-center"></a>Azure Security Center
Például a felhőre a legnagyobb blockers egyik lett fontos szempont biztonsági keresztül. Győződjön meg arról, hogy az Azure-erőforrások biztonságos kell informatikai kockázatkezelők és biztonsági részlegek számára. 

A [az Azure Security Center](/azure/security-center/security-center-intro) az előfizetésekben lévő erőforrások biztonsági állapotának egy központi áttekintést nyújt a, és ajánlásokkal segíti a feltört erőforrásokat a megelőzése érdekében. Engedélyezheti, hogy a részletesebb szabályzatok (például az adott erőforrás-csoportok lehetővé teszik a vállalat a megfelelő, azok állapotát a kockázatnak, érintik alkalmazása szabályzatokat). Végül az Azure Security Center nyílt platformon, amely lehetővé teszi a Microsoft-partnerek és független szoftverszállítók szoftver, amely az Azure Security Center a képességek javításához rendkívüli létrehozásához. 

> [!TIP]
> Az Azure Security Center az egyes előfizetésekben alapértelmezés szerint engedélyezve van. Adatgyűjtés engedélyezése az Azure Security Center használatával telepítse az ügynököt, és megkezdi az adatgyűjtést a virtuális gépekről, engedélyeznie kell.
> 
> ![adatgyűjtés](./_images/data-collection.png)
> 
> 

## <a name="next-steps"></a>További lépések
* Most, hogy az előfizetés-irányítás megismerkedett, ideje, hogy ezek a javaslatok a gyakorlatban. Lásd: [példák az Azure előfizetés-irányítás végrehajtási](azure-scaffold-examples.md).

> [!div class="nextstepaction"]
> [Példa megvalósítása](azure-scaffold-examples.md)
