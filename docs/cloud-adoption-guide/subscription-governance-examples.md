---
title: Forgatókönyvek és példák előfizetés irányításhoz |} Microsoft Docs
description: Azure-előfizetés-szabályozás megvalósításához a gyakori forgatókönyvek megvalósításához példákat.
services: azure-resource-manager
documentationcenter: na
author: rdendtler
manager: timlt
editor: tysonn
ms.assetid: e8fbeeb8-d7a1-48af-804f-6fe1a6024bcb
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 01/03/2017
ms.author: rodend;karlku;tomfitz
ms.openlocfilehash: 4a4b25e6a6e54fd18c8ae21a88cab39143c3bf48
ms.sourcegitcommit: 4ec010846b9b5545c843a32e08293f906e512302
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 05/17/2018
ms.locfileid: "34299972"
---
# <a name="examples-of-implementing-azure-enterprise-scaffold"></a>Az Azure enterprise scaffold implementációi
Ez a cikk alapján vállalati hogyan valósíthatja meg a javaslatok példákat tartalmaz egy [Azure enterprise scaffold](subscription-governance.md). Gyakori helyzetek ajánlott eljárásai bemutatják egy kitalált, Contoso nevű vállalat használ.

## <a name="background"></a>Háttér
Contoso vállalat világszerte ellátási lánc megoldást nyújt az ügyfelek. Ezek biztosítanak mindent egy szoftverfrissítési a modell csomagolt modellre telepített helyszíni.  Ők fejlesztik ki szoftver jelentős fejlesztési adatközpontok India, az Amerikai Egyesült Államok és Kanada között a világ körüli.

A vállalat ISV része több független részlegek termékek kezelése jelentős üzleti van osztva. Minden részleg rendelkezik, saját fejlesztők számára, a termék kezelők és a fejlesztők.

A vállalati informatikai szolgáltatások (ETS) részleg központi informatikai képességet biztosít, és ahol üzleti egységek tárolására az alkalmazások több adatközpontok kezeli. Az adatközpontok kezelése, valamint a ETS szervezet biztosít, és kezeli (például az e-mailek és webhelyek) központi együttműködés és a hálózati vagy a telefonos szolgáltatások. Akkor is kisebb részlegek műveleti személyzet nem rendelkező ügyfélkapcsolati munkaterhelések kezelése.

Ez a cikk a következő személyeknek használt:

* Dave a ETS Azure rendszergazda.
* Alice Contoso igazgató a fejlesztés ellátási lánc részleg.

Contoso kell hozhat létre egy üzleti alkalmazást és egy ügyfélkapcsolati alkalmazást. Az alkalmazások futtatásához Azure határozott. Dave beolvassa a [előíró előfizetés irányítás](subscription-governance.md) a következő cikket, és készen áll a a ajánlások megvalósításával.

## <a name="scenario-1-line-of-business-application"></a>1. forgatókönyv: üzleti alkalmazás
A Contoso egy forrás kód kezelő rendszer (Bitbucketből) keresztül történik a fejlesztők által használandó fejleszt.  Az alkalmazás infrastruktúra használ szolgáltatásként (IaaS) futtató, és a webkiszolgálók és adatbázis-kiszolgáló áll. Fejlesztők fejlesztői környezetükben lévő kiszolgálók eléréséhez, de nem a kiszolgálók az Azure-ban való hozzáférésre van szükségük. Contoso ETS szeretné az alkalmazás tulajdonos és a csoport, az alkalmazás kezeléséhez. Az alkalmazás csak akkor közben a Contoso vállalati hálózaton. Dave kell kötnie az előfizetés ehhez az alkalmazáshoz. Az előfizetés is tárolni fogja más fejlesztéssel kapcsolatos szoftver a jövőben.  

### <a name="naming-standards--resource-groups"></a>Elnevezési szabályai & erőforráscsoportok
Dave támogatja a fejlesztők által az üzleti egységek közösen használt eszközök egy előfizetést hoz létre. Dave kell kifejező nevet (a az alkalmazás és a hálózatok) előfizetésbe és erőforráscsoportba csoportjai számára. Létrehoz a következő előfizetés és az erőforrás-csoportok:

| Elem | Name (Név) | Leírás |
| --- | --- | --- |
| Előfizetés |Contoso ETS DeveloperTools éles |Támogatja a közös fejlesztői eszközök |
| Erőforráscsoport |bitbucket-termék-rg |Tartalmazza az alkalmazások webalkalmazás-kiszolgáló és adatbázis-kiszolgáló |
| Erőforráscsoport |corenetworks-termék-rg |A virtuális hálózatok és a webhelyek közötti átjáró kapcsolat tartalmazza |

### <a name="role-based-access-control"></a>Szerepköralapú hozzáférés-vezérlés
Miután létrehozta az előfizetést, Dave biztosítani szeretné, hogy a megfelelő csapat és alkalmazástulajdonosok érhetik el az erőforrásaikat. Dave felismeri, hogy egyes csapatok különböző követelményekkel rendelkezik. Ezután a csoportokat, amelyek a Contoso a helyszíni Active Directory (AD) az Azure Active Directoryval szinkronizált használja, és biztosítja a hozzáférést a csoportok a megfelelő szintű.

Dave rendeli hozzá az előfizetés a következő szerepkörök:

| Szerepkör | Hozzárendelve a következőhöz: | Leírás |
| --- | --- | --- |
| [Tulajdonos](/azure/role-based-access-control/built-in-roles#owner) |Contoso-azonosító a felügyelt AD |Ez az azonosító csak a Contoso-Identity Management eszközzel idő szerinti (JIT) hozzáférési vezérli, és biztosítja, hogy teljesen naplózza az előfizetés tulajdonosa hozzáférés |
| [Biztonsági olvasó](/azure/role-based-access-control/built-in-roles#security-reader) |Adatvédelmi és kockázatkezelési felügyeleti részleg |Ez a szerepkör lehetővé teszi a felhasználók az Azure Security Center és az erőforrások állapotának megtekintése |
| [Hálózati közreműködő](/azure/role-based-access-control/built-in-roles#network-contributor) |Hálózati adapterek |Ez a szerepkör lehetővé teszi, hogy a Contoso-hálózati adapterek a webhelyek közötti VPN és a virtuális hálózatok kezeléséhez |
| *Egyéni szerepkör* |Alkalmazás tulajdonosa |Dave szerepet hoz létre, amely lehetővé teszi az erőforráscsoporton belül erőforrásainak módosítása. További információkért lásd: [egyéni szerepkörök az Azure RBAC](/azure/role-based-access-control/custom-roles) |

### <a name="policies"></a>Házirendek
Dave az előfizetésben szereplő erőforrások kezeléséhez a következő követelményekkel rendelkezik:

* A fejlesztői eszközök támogatják a fejlesztők keresztül történik, mert azt nem szeretné, hogy erőforrásokat hozzon létre bármely régióban felhasználó számára. Azonban hogy tudnia kell, ahol erőforrások jönnek létre.
* Érintett költséggel. Ezért azt szeretné, hogy ezáltal megakadályozhatja, hogy a alkalmazástulajdonosok feleslegesen költséges virtuális gépek létrehozását.  
* Ez az alkalmazás számos üzleti egységek fejlesztők is kiszolgál, mert szeretné az egyes erőforrások az üzleti egység és a kérelem tulajdonossal rendelkező címke. Ezek a címkék használatával ETS a megfelelő csoportokkal is kiszámlázni.

A következő hoz [Azure házirendek](/azure/azure-policy/azure-policy-introduction):

| Mező | Következmény | Leírás |
| --- | --- | --- |
| location |Naplózási |Naplózási bármely régióban erőforrás létrehozása |
| type |elutasítás |G sorozatú virtuális gépek létrehozását megtagadása |
| tags |elutasítás |Igényelnek az alkalmazás tulajdonosa címke |
| tags |elutasítás |Költség center címke megkövetelése |
| tags |hozzáfűzése |A címke neve hozzáfűzése **részleghez** értéke pedig **ETS** az összes erőforráshoz |

### <a name="resource-tags"></a>Az erőforráscímkék
Dave tisztában van azzal, hogy ő rendelkeznie kell a számlázási azonosíthatja a költségközpont BitBucket végrehajtásához meg. Emellett Dave szeretné tudni, hogy az erőforrások ETS birtokló.

Adja a következő [címkék](/azure/azure-resource-manager/resource-group-using-tags) erőforráscsoportok és erőforrásokhoz.

| A címke neve | Címke |
| --- | --- |
| ApplicationOwner |Ezt az alkalmazást kezelő személy nevét |
| Költséghely |A csoport, amely az Azure felhasználásra fizeti költséghely |
| Részleghez |**ETS** (az üzleti egység az előfizetéshez tartozó) |

### <a name="core-network"></a>Központi hálózat
A Contoso ETS információk adatvédelmi és kockázatkezelési vezetőség ellenőrzi, hogy Dave által javasolt tervezi helyezze át az alkalmazást az Azure-bA. Győződjön meg arról, hogy az alkalmazás nem kommunikál az internettel szeretnék.  Dave is rendelkezik, amely a későbbiekben csak az Azure-ba kerül fejlesztői alkalmazások. Ezekhez az alkalmazásokhoz igényelnek közös felületek.  Ezek a követelmények teljesítéséhez szolgáltat belső és külső virtuális hálózatokat egyaránt, és korlátozhatja a hozzáférést hálózati biztonsági csoport.

Létrehoz a következőket:

| Erőforrás típusa | Name (Név) | Leírás |
| --- | --- | --- |
| Virtual Network |belső virtuális hálózat |A Bitbucketből alkalmazással használja, és ExpressRoute keresztül csatlakozik, a Contoso vállalati hálózathoz.  Egy alhálózat (`bitbucket`) biztosít az alkalmazást egy adott IP-címtér |
| Virtual Network |külső-hálózatok |A jövőbeli nyilvánosan elérhető végpontok igénylő alkalmazások érhető el |
| Hálózati biztonsági csoport |bitbucket-nsg |Biztosítja, hogy a támadási felületet, a munkaterhelés minimalizálható a kapcsolatok csak a 443-as porton az alhálózat ahol él, az alkalmazás engedélyezése (`bitbucket`) |

### <a name="resource-locks"></a>Erőforrás-zárolások
Dave felismeri, hogy a Contoso vállalati hálózati kapcsolat a belső virtuális hálózathoz védeni kell a wayward parancsfájlok vagy véletlen törlés.

A következő hoz [erőforrás zárolási](/azure/azure-resource-manager/resource-group-lock-resources):

| Zárolás típusa | Erőforrás | Leírás |
| --- | --- | --- |
| **CanNotDelete** |belső virtuális hálózat |Megakadályozza, hogy a felhasználók törlése a virtuális hálózat vagy az alhálózatot, de nem akadályozza meg az új alhálózat hozzáadása |

### <a name="azure-automation"></a>Azure Automation
Dave automatizálható az alkalmazás nem rendelkezik. Bár a létrehozott egy Azure Automation-fiók, akkor nem kezdetben használja ezt a.

### <a name="azure-security-center"></a>Azure Security Center
Contoso informatikai szolgáltatások kezelésében gyorsan azonosíthatja és fenyegetések kezelni kell. Akkor is szeretnék megismerni, hogy milyen problémákat előfordulhat, hogy létezik.  

Ezek a követelmények teljesítéséhez, Dave lehetővé teszi, hogy a [az Azure Security Center](/azure/security-center/security-center-intro), és a biztonsági olvasó szerepkört hozzáférést biztosít.

## <a name="scenario-2-customer-facing-app"></a>2. forgatókönyv: ügyfélkapcsolati alkalmazás
Az üzleti vezetőségének munkáját segítik a ellátási lánc részleg azonosította a Contoso-ügyfelek növeléséhez hűség kártyával különböző lehetőségek. Ágnes team létre kell hoznia ezt az alkalmazást, és úgy dönt, hogy Azure növeli az üzleti igények kielégítéséhez képesek. Alice a fejlesztés, és az alkalmazás működtetéséhez két előfizetések konfigurálása ETS Dave működik.

### <a name="azure-subscriptions"></a>Azure-előfizetések
Dave jelentkezik be az Azure-vállalati portálra, és láthatja, hogy az ellátási lánc osztály már létezik.  Azonban mivel ebben a projektben az első alkalmazásfejlesztési projekt az ellátási lánc csoport az Azure-ban, a Dave felismeri Ágnes fejlesztői csapat az új fiók szükséges.  Ezután a csapata számára "R & D" fiókot hoz létre, és hozzáférést rendel Alice. Alice jelentkezik be az Azure-portálon, és létrehoz két előfizetések: egyet a fejlesztési kiszolgáló és egy ahhoz, hogy az üzemi kiszolgálók tárolásához.  A korábban létrehozott elnevezési szabályai ő követ, amikor a következő előfizetések létrehozása:

| Előfizetés használata | Name (Név) |
| --- | --- |
| Fejlesztés |Contoso SupplyChain ResearchDevelopment LoyaltyCard fejlesztési |
| Production |Contoso SupplyChain műveletek LoyaltyCard éles |

### <a name="policies"></a>Házirendek
Dave és Alice ismertetik az alkalmazást, és azonosíthatja, hogy az alkalmazás csak szolgál az Észak-amerikai régió ügyfelek.  Alice és csapata tervez használni az Azure alkalmazás Service-környezet és az Azure SQL az alkalmazás létrehozására. Előfordulhat, hogy a fejlesztés során a virtuális gépek létrehozásához szükséges.  Alice szeretne arról, hogy a fejlesztők a vizsgálatát, és vizsgálja meg a problémákat anélkül, hogy a ETS húzza szükséges erőforrásokat.

Az a **fejlesztési előfizetés**, hoznak létre a következő házirendet:

| Mező | Következmény | Leírás |
| --- | --- | --- |
| location |Naplózási |Naplózási bármely régióban erőforrás létrehozása |

Nem korlátozzák a felhasználó hozhat létre a fejlesztési sku típusú, és címkék az erőforráscsoport-sablonok és erőforrások nem igénylik.

Az a **éles előfizetés**, akkor hozza létre a következő házirendeket:

| Mező | Következmény | Leírás |
| --- | --- | --- |
| location |elutasítás |Az Amerikai Egyesült Államok adatközpontok kívül semmilyen erőforráshoz létrehozását megtagadása |
| tags |elutasítás |Igényelnek az alkalmazás tulajdonosa címke |
| tags |elutasítás |Részleg címke megkövetelése |
| tags |hozzáfűzése |Címke hozzáfűzése minden erőforráscsoportban, amely jelzi, éles környezetben |

Azok nem korlátozza a termékváltozat éles hozhat létre a felhasználó milyen típusú.

### <a name="resource-tags"></a>Az erőforráscímkék
Dave tisztában van azzal, hogy ő rendelkeznie kell a megfelelő üzleti azonosítására a számlázási és a tulajdonosi információk. Az erőforráscsoport-sablonok és erőforrások erőforráscímkék kiadásként definiálja.

| A címke neve | Címke |
| --- | --- |
| ApplicationOwner |Ezt az alkalmazást kezelő személy nevét |
| Részleg |A csoport, amely az Azure felhasználásra fizeti költséghely |
| EnvironmentType |**Éles** (annak ellenére, hogy az előfizetése **éles** nevét, ennek a címkének lehetővé teszi a könnyebbé teszi a beazonosítást kezelőkonzoljából nézve a portál vagy a számlázási erőforrások) |

### <a name="core-networks"></a>Alapvető hálózat
A Contoso ETS információk adatvédelmi és kockázatkezelési vezetőség ellenőrzi, hogy Dave által javasolt tervezi helyezze át az alkalmazást az Azure-bA. Győződjön meg arról, hogy a hűség Card alkalmazást megfelelően elkülönített és védett, egy Szegélyhálózaton hálózati szeretnék.  Ez a követelmény teljesítéséhez Dave és Alice hozzon létre egy külső virtuális hálózatot és a hálózati biztonsági csoport különítheti el a hűség Card alkalmazást, a Contoso vállalati hálózatról.  

Az a **fejlesztési előfizetés**, hogy létre:

| Erőforrás típusa | Name (Név) | Leírás |
| --- | --- | --- |
| Virtual Network |belső virtuális hálózat |A Contoso hűség kártya fejlesztőkörnyezet szolgál, és ExpressRoute keresztül csatlakozik, a Contoso vállalati hálózat |

Az a **éles előfizetés**, hoznak létre:

| Erőforrás típusa | Name (Név) | Leírás |
| --- | --- | --- |
| Virtual Network |külső-hálózatok |A hűség Card alkalmazást használja, és nem közvetlenül kapcsolódó Contoso ExpressRoute. Kód kerül a forráskód rendszeren keresztül közvetlenül a PaaS szolgáltatások |
| Hálózati biztonsági csoport |loyaltycard-nsg |Biztosítja, hogy a támadási felületet, a munkaterhelés minimalizálható a csak a TCP 443-as porton az adathoz kötött kommunikáció engedélyezése.  Contoso is vizsgálja a további védelem webalkalmazási tűzfal használata |

### <a name="resource-locks"></a>Erőforrás-zárolások
Dave és Alice révén, és úgy dönt, hogy az erőforrás zárolások hozzáadása a a kulcsfontosságú erőforrások során egy errant kód leküldéses véletlen törlésének megakadályozása a környezetben.

Akkor hozzon létre a következő zárolási:

| Zárolás típusa | Erőforrás | Leírás |
| --- | --- | --- |
| **CanNotDelete** |külső-hálózatok |Megakadályozhatja, hogy a felhasználók a törlése a virtuális hálózat vagy az alhálózatot. A zárolás nem akadályozza meg az új alhálózat hozzáadása |

### <a name="azure-automation"></a>Azure Automation
Alice és a fejlesztői csapat rendelkezik az ehhez az alkalmazáshoz a környezete kezeléséhez alapos runbookok. A runbookok lehetővé teszi az alkalmazás és más DevOps feladatok csomópontok hozzáadása vagy törlése.

Ezek a runbookok használatához engedélyezze [Automation](/azure/automation/automation-intro).

### <a name="azure-security-center"></a>Azure Security Center
Contoso informatikai szolgáltatások kezelésében gyorsan azonosíthatja és fenyegetések kezelni kell. Akkor is szeretnék megismerni, hogy milyen problémákat előfordulhat, hogy létezik.  

Ezek a követelmények teljesítéséhez, Dave lehetővé teszi, hogy az Azure Security Center. Ezután biztosítja, hogy az Azure Security Center által figyelt az erőforrásokat, és a DevOps és biztonsági csoportok hozzáférést biztosít.

## <a name="next-steps"></a>További lépések
* Resource Manager-sablonok létrehozásával kapcsolatos további tudnivalókért lásd: [ajánlott eljárások Azure Resource Manager-sablonok létrehozásához](/azure/azure-resource-manager/resource-manager-template-best-practices).
