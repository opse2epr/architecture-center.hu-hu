---
title: Forgatókönyvek és példák az előfizetés-irányítás
description: Megvalósítása az Azure-előfizetés cégirányítási gyakori szituációhoz kínál példákat.
author: rdendtler
ms.date: 01/03/2017
ms.author: rodend
ms.topic: guide
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.openlocfilehash: cbf3aae20639d26a73aac07e1b66374af09fbb38
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54486202"
---
# <a name="examples-of-implementing-azure-enterprise-scaffold"></a>Példák az Azure enterprise scaffold megvalósítása
Ez a cikk példákat tartalmaz a vállalat hogyan valósíthat meg a javaslatok egy [Azure enterprise scaffold](azure-scaffold.md). Ajánlott eljárások a gyakori forgatókönyvek bemutatják egy kitalált, Contoso nevű vállalat használ.

## <a name="background"></a>Háttér
Contoso egy globális vállalat, amely az ellátási lánc megoldásokat kínál azon ügyfeleinek. Biztosítanak minden szoftverfrissítési csomagolt modellre szolgáltatásmodellben telepített helyszíni.  Szoftverek fejlesztésére, szerte a világon, az jelentős fejlesztési központok az Indiában, az Egyesült Államok és Kanada.

A független Szoftverszállító része a vállalat több független részlegek kezelheti a termékeket egy jelentős üzleti oszlik. Mindegyik üzleti egység rendelkezik a saját fejlesztők, a termék kezelők és a tervezők munkáját.

A vállalati technológiai szolgáltatások (ETS) üzleti egység központi IT-képességeket biztosít, és amelyeknél az üzleti egységek állomás alkalmazásaikat több adatközpontban kezeli. Az adatközpontok kezelése, valamint a ETS szervezet biztosít, és kezeli (például az e-mailek és a websites) központi együttműködés és a hálózati és telefonos szolgáltatások. Ezek is kisebb üzleti egység, akik nem rendelkeznek a kezelőszemélyzet ügyfelek által használt számítási feladatok kezelése.

Ez a cikk alábbi használt:

* Dave a ETS Azure rendszergazda.
* Ágnes a Contoso fejlesztési az ellátási lánc részleg.

Contoso cégnek szüksége van egy sor üzleti alkalmazás és a egy ügyfelek által használt alkalmazást hozhat létre. Döntött, hogy az alkalmazások futtatásához az Azure-ban. Dave beolvassa a [előíró előfizetés-irányítás](azure-scaffold.md) című cikket, és már készen áll az ajánlások megvalósítása.

## <a name="scenario-1-line-of-business-application"></a>1. forgatókönyv: üzleti alkalmazás
Contoso fejleszt egy forrás kódot (BitBucket) világszerte használják a fejlesztők a rendszer.  Az alkalmazás infrastruktúrát használja a szolgáltatás (IaaS) üzemeltetésére, és a webkiszolgálók és a egy adatbázis-kiszolgáló áll. A fejlesztők a fejlesztői környezetükben lévő kiszolgálók eléréséhez, de nincs szükségük a kiszolgálók az Azure-ban való hozzáférést. Contoso ETS szeretné engedélyezni az alkalmazás tulajdonosa és csapat az alkalmazás kezelése céljából. Az alkalmazás csak akkor használható, miközben a Contoso vállalati hálózaton. Dave konfigurálnia kell az előfizetés ehhez az alkalmazáshoz. A jövőben az előfizetés is fogja futtatni más fejlesztői kapcsolódó szoftvereket.  

### <a name="naming-standards--resource-groups"></a>Elnevezési szabályai és erőforrás-csoportok
Dave támogatja a fejlesztői eszközök által az üzleti egységek közösen használt előfizetést hoz létre. Dave létre kell hoznia az előfizetést és erőforráscsoportot csoportok (a az alkalmazás és a hálózatok) adjon kifejező nevet. A következő előfizetésben és erőforráscsoportban csoportokat hoz létre:

| Elem | Név | Leírás |
| --- | --- | --- |
| Előfizetés |Contoso ETS DeveloperTools éles környezetben |Támogatja az általános fejlesztői eszközökhöz |
| Erőforráscsoport |bitbucket-prod-rg |Tartalmazza az alkalmazások webalkalmazás-kiszolgáló és adatbázis-kiszolgáló |
| Erőforráscsoport |corenetworks-prod-rg |Tartalmazza a virtuális hálózatok és helyek közötti átjárókapcsolat |

### <a name="role-based-access-control"></a>Szerepköralapú hozzáférés-vezérlés
Miután létrehozta a saját előfizetés, Dave szeretne győződjön meg arról, hogy a megfelelő csapatok és az alkalmazástulajdonosok hozzáférhet erőforrásaikat. Dave felismeri, hogy egyes csapatok különböző követelményekkel rendelkezik. Ő már használja a csoportokat, amelyek az Azure Active Directoryhoz a Contoso a helyszíni Active Directory (AD) szinkronizált, és a megfelelő szintű csapatok számára hozzáférést biztosít.

Dave rendeli hozzá az előfizetés a következő szerepkörök:

| Szerepkör | Hozzárendelve | Leírás |
| --- | --- | --- |
| [Tulajdonos](/azure/role-based-access-control/built-in-roles#owner) |A felügyelt azonosítója a Contoso AD |Ez az azonosító csak a Contoso-Identity Management eszközzel idő szerinti (JIT) hozzáférési vezérli, és biztosítja, hogy teljes mértékben naplózza az előfizetés-tulajdonosi hozzáféréssel |
| [Biztonsági olvasó](/azure/role-based-access-control/built-in-roles#security-reader) |Adatvédelmi és kockázatkezelési felügyeleti részlege |Ez a szerepkör lehetővé teszi a felhasználóknak tekintse meg az Azure Security Center és az erőforrások állapota |
| [Hálózati közreműködő](/azure/role-based-access-control/built-in-roles#network-contributor) |Hálózati |Ez a szerepkör lehetővé teszi, hogy a Contoso-csoport a helyek közötti VPN- és a virtuális hálózatok kezelése |
| *Egyéni szerepkör* |Alkalmazás tulajdonosa |Dave hoz létre egy szerepkört, amely engedélyezi a csoporton belüli erőforrások módosítását. További információkért lásd: [egyéni szerepkörök az Azure RBAC-ben](/azure/role-based-access-control/custom-roles) |

### <a name="policies"></a>Szabályzatok
Dave az előfizetés-erőforrások kezeléséhez a következő követelményekkel rendelkezik:

* Mivel a fejlesztői eszközöket támogatja a fejlesztők világszerte, akkor nem szeretné a felhasználók megakadályozása az erőforrások létrehozásának bármelyik régióban. Azonban hogy tudnia kell, ahol erőforrások jönnek létre.
* Érintett költségek, ő. Ezért azt szeretné, hogy megakadályozza, hogy az alkalmazástulajdonosok szükségtelenül drága virtuális gépekre.  
* Mivel ez az alkalmazás a fejlesztők számos üzleti egység szolgál ki, szeretné az egyes erőforrások címkékkel az üzleti egység és az alkalmazás tulajdonosa. Ezek a címkék használatával ETS is ki a megfelelő csoportokkal.

A következő hoz létre [Azure házirendek](/azure/azure-policy/azure-policy-introduction):

| Mező | Hatás | Leírás |
| --- | --- | --- |
| hely |Naplózási |Az adott régióban az erőforrások létrehozását naplózása |
| típus |elutasítás |A G-sorozatú virtuális gépek létrehozásának megtagadása |
| címkék |elutasítás |Alkalmazás tulajdonosa címke megkövetelése |
| címkék |elutasítás |Költség center címke megkövetelése |
| címkék |Hozzáfűzése |Fűzze hozzá a címke neve **részleghez** , értéke pedig **ETS** az összes erőforráshoz |

### <a name="resource-tags"></a>Erőforráscímkék
Dave tisztában van azzal, hogy ő kell rendelkeznie a számlán, hogy azonosítsa a költséghely a bitbucket-alapú megvalósítás információkat. Ezenkívül Dave szeretné tudni, hogy ETS tulajdonában lévő összes erőforrást.

Adja a következő [címkék](/azure/azure-resource-manager/resource-group-using-tags) erőforráscsoportok és erőforrásokhoz.

| Címke neve | Címke értéke |
| --- | --- |
| ApplicationOwner |Ezt az alkalmazást kezelő személy neve |
| CostCenter |A költségközpont által kell fizetnie, az Azure-használat |
| Részleghez |**ETS** (az üzleti egység az előfizetéshez tartozó) |

### <a name="core-network"></a>A központi hálózat
A Contoso ETS információt adatvédelmi és kockázatkezelési-kezelési csapatunk áttekinti a Dave a javasolt csomag az alkalmazás az Azure-bA áthelyezni. Győződjön meg arról, hogy az alkalmazás internettel érintkező nem szeretnének.  Dave is tartalmaz, amely a későbbiekben az Azure-ba kerül fejlesztői alkalmazások. Ezek az alkalmazások a nyilvános felületek van szükség.  Mindezen követelmények teljesítése érdekében Phil nyújt külső és belső virtuális hálózatok és a hozzáférés korlátozása a hálózati biztonsági csoport.

A következő erőforrások hoz létre:

| Erőforrás típusa | Név | Leírás |
| --- | --- | --- |
| Virtual Network |internal-vnet |A BitBucket-alkalmazással használja, és a Contoso vállalati hálózat expressroute-on keresztül kapcsolódik.  Egy alhálózat (`bitbucket`) biztosít az alkalmazás egy adott IP-címtér |
| Virtual Network |external-vnet |Nyilvános végpontok jövőbeli alkalmazásokhoz érhető el |
| Hálózati biztonsági csoport |bitbucket-nsg |Biztosítja, hogy ilyen számítási feladatok támadási felületének minimálisra csökken azáltal, hogy csak a 443-as porton az alhálózat ahol az alkalmazás él kapcsolatok (`bitbucket`) |

### <a name="resource-locks"></a>Erőforrás-zárolások
Dave felismeri, hogy a kapcsolat a Contoso vállalati hálózat és a belső virtuális hálózathoz kell védeni a wayward parancsfájlok vagy véletlen törlés.

A következő hoz létre [erőforrászárat](/azure/azure-resource-manager/resource-group-lock-resources):

| Zárolás típusa | Erőforrás | Leírás |
| --- | --- | --- |
| **CanNotDelete** |internal-vnet |Megakadályozza, hogy a felhasználók a virtuális hálózat vagy alhálózat törlése, de nem akadályozza meg az új alhálózat hozzáadása |

### <a name="azure-automation"></a>Azure Automation
Dave nem az alkalmazás automatizálásához. Létrehozott egy Azure Automation-fiókot, bár ő nem kezdetben használhatja.

### <a name="azure-security-center"></a>Azure Security Center
Contoso IT service management gyorsan azonosíthatja és fenyegetések kezelni kell. Emellett szeretné megérteni, milyen problémák is előfordulhatnak.  

Ezek a követelmények teljesítéséhez, Dave lehetővé teszi a [az Azure Security Center](/azure/security-center/security-center-intro), és a biztonsági olvasó szerepkör hozzáférést biztosít.

## <a name="scenario-2-customer-facing-app"></a>2. forgatókönyv: ügyfelek által használt alkalmazás
Az üzleti vezető szerepet tölt be az ellátási lánc üzleti egységet azonosította a különböző lehetőségeket a Contoso-ügyfelek motiváltságot növelik a hűségprogramok használatán keresztül kártya használatával. Alice csapata létre kell hoznia az alkalmazást, és úgy dönt, hogy az Azure növeli az üzleti igények kielégítése érdekében képesek. Alice a ETS konfigurálása két előfizetéssel és az alkalmazás működési Dave együttműködik.

### <a name="azure-subscriptions"></a>Azure-előfizetések
Dave jelentkezik be az Azure Enterprise portálon, és láthatja, hogy az ellátási lánc osztály már létezik.  Azonban mivel ez a projekt az első alkalmazásfejlesztési projekt az ellátási lánc csapat az Azure-ban, Dave Alice fejlesztői csapat az új fiók szükség felismeri.  Ő a csapata számára az "R & D" fiókot hoz létre, és hozzáférést rendel a Alice. Alice jelentkezik be az Azure Portalon, és két előfizetéssel hoz létre: egyet a fejlesztési kiszolgálók, valamint ahhoz, hogy az üzemi kiszolgálók tárolásához.  Marcela követi a korábban létrehozott elnevezési szabványait létrehozásakor a következő előfizetéseket:

| Előfizetés használata | Név |
| --- | --- |
| Fejlesztés |Contoso SupplyChain ResearchDevelopment LoyaltyCard fejlesztés |
| Üzemi |Contoso SupplyChain Operations LoyaltyCard éles környezetben |

### <a name="policies"></a>Szabályzatok
Dave és Alice ismertetjük az alkalmazást, és azonosítani, hogy az alkalmazás csak szolgál ügyfeleink Észak-amerikai régióban.  Alice és csapata tervezze meg az alkalmazás létrehozása az Azure Application Service-környezet és az Azure SQL használatával. Előfordulhat, hogy létre kell hozniuk a virtuális gépek a fejlesztés során.  Alice, győződjön meg arról, hogy saját fejlesztők rendelkezik-e a vizsgálata, és vizsgálja meg a problémákat anélkül ETS behúzza szükséges szeretné.

Az a **fejlesztési előfizetés**, hoznak létre a következő szabályzatot:

| Mező | Hatás | Leírás |
| --- | --- | --- |
| hely |Naplózási |Az adott régióban az erőforrások létrehozását naplózása |

Ezek nem korlátozza a termékváltozat fejlesztési hozhat létre a felhasználó típusát, és nem kívánnak címkék erőforrások vagy erőforráscsoportok.

A a **éles előfizetés**, akkor hozza létre az alábbi szabályzatokat:

| Mező | Hatás | Leírás |
| --- | --- | --- |
| hely |elutasítás |Az Egyesült Államokbeli adatközpontok kívüli erőforrások létrehozásának megtagadása |
| címkék |elutasítás |Alkalmazás tulajdonosa címke megkövetelése |
| címkék |elutasítás |Szükséges a department címke |
| címkék |Hozzáfűzése |Tag hozzáfűzése minden egyes erőforrás-csoportba, amely azt jelzi, hogy éles környezetben |

Ezek nem korlátozza a felhasználó létrehozhat éles környezetben termékváltozat típusát.

### <a name="resource-tags"></a>Erőforráscímkék
Dave tisztában van azzal, hogy ő kell rendelkeznie kell azonosítani a megfelelő üzleti csoportokat számlázási és tulajdonjogát. Az erőforráscsoport és erőforrások erőforráscímkék kiadásként definiálja.

| Címke neve | Címke értéke |
| --- | --- |
| ApplicationOwner |Ezt az alkalmazást kezelő személy neve |
| Részleg |A költségközpont által kell fizetnie, az Azure-használat |
| EnvironmentType |**Éles** (annak ellenére, hogy az előfizetése tartalmazza-e **éles** a nevében, ennek a címkének lehetővé teszi, hogy könnyen azonosító erőforrásokat a portálon vagy a számlán megtekintve) |

### <a name="core-networks"></a>Alapvető hálózat
A Contoso ETS információt adatvédelmi és kockázatkezelési-kezelési csapatunk áttekinti a Dave a javasolt csomag az alkalmazás az Azure-bA áthelyezni. Szeretné, hogy a hűségprogramok használatán keresztül kártya alkalmazás megfelelően elkülönített és védett hálózati DMZ-t.  Ez a követelmény teljesítéséhez, Dave és Alice hozzon létre egy külső virtuális hálózatot és a egy hálózati biztonsági csoportot a hűségprogramok használatán keresztül Card alkalmazást –, a Contoso vállalati hálózat elkülönítésére.  

Az a **fejlesztési előfizetés**, hozhatnak létre:

| Erőforrás típusa | Név | Leírás |
| --- | --- | --- |
| Virtual Network |internal-vnet |Szolgálja ki a Contoso hűségprogramok használatán keresztül kártya fejlesztési környezet és az expressroute-on keresztül kapcsolódik a Contoso vállalati hálózat |

Az a **éles előfizetés**, hozhatnak létre:

| Erőforrás típusa | Név | Leírás |
| --- | --- | --- |
| Virtual Network |external-vnet |Hűségprogramok használatán keresztül kártya-alkalmazást, és közvetlenül a Contoso ExpressRoute nincs csatlakoztatva. Kód a rendszer továbbítja a forráskód rendszeren keresztül közvetlenül a PaaS-szolgáltatások |
| Hálózati biztonsági csoport |loyaltycard-nsg |Biztosítja, hogy az ilyen számítási feladatok támadási felületét azáltal, hogy csak a korlátozott kommunikáció a TCP 443-as másodpercekre csökken.  Contoso is vizsgálja a további védelem webalkalmazási tűzfal használata |

### <a name="resource-locks"></a>Erőforrás-zárolások
Dave és Alice ruháznak, és úgy dönt, hogy az erőforrás-zárolások hozzáadása az egyes során egy errant kód leküldéses véletlen törlés megelőzése érdekében a környezetben a legfontosabb erőforrásokat.

Akkor hozzon létre a következő zárolást:

| Zárolás típusa | Erőforrás | Leírás |
| --- | --- | --- |
| **CanNotDelete** |external-vnet |Megakadályozza, hogy a felhasználók a virtuális hálózat vagy alhálózat törlése. A zárolás nem akadályozza meg az új alhálózat hozzáadása |

### <a name="azure-automation"></a>Azure Automation
Alice és fejlesztési csapata rendelkeznek széles körű runbookok ehhez az alkalmazáshoz a környezet kezeléséhez. A runbookok lehetővé teszi az alkalmazás és más fejlesztési és üzemeltetési feladatokat csomópontok hozzáadása és törléséről.

Ezek a runbookok használatához engedélyezze [Automation](/azure/automation/automation-intro).

### <a name="azure-security-center"></a>Azure Security Center
Contoso IT service management gyorsan azonosíthatja és fenyegetések kezelni kell. Emellett szeretné megérteni, milyen problémák is előfordulhatnak.  

Ezek a követelmények teljesítéséhez, Dave lehetővé teszi, hogy az Azure Security Center. Phil biztosítja, hogy az Azure Security Center által figyelt az erőforrásokat, és a DevOps és biztonsági csapatok hozzáférést biztosít.

## <a name="next-steps"></a>További lépések
* Resource Manager-sablonok létrehozásával kapcsolatos további információkért lásd: [ajánlott eljárások az Azure Resource Manager-sablonok létrehozására](/azure/azure-resource-manager/resource-manager-template-best-practices).
