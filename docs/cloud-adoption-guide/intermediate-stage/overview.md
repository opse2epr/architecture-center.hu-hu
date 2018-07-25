---
title: 'Az Azure bevezetésének: köztes'
description: Ismerteti, hogy a vállalati Azure elfogadására szüksége van az ismeretek köztes szintre állítása
author: petertay
ms.openlocfilehash: a1f93616f5f1ecf4f395ce39bbb037ef6ab5991b
ms.sourcegitcommit: c704d5d51c8f9bbab26465941ddcf267040a8459
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 07/24/2018
ms.locfileid: "39229235"
---
# <a name="azure-cloud-adoption-guide-intermediate-overview"></a>Az Azure Cloud bevezetési útmutató: Köztes áttekintése

Az alapszintű bevezetési szakasz megismerhette, az Azure erőforrás-szabályozása alapvető fogalmait. Az alapvető. fázis úgy lett kialakítva, az első lépésekhez az Azure bevezetését, és azt megismerhette, hogyan küldhet egy kisebb csoportot egy egyszerű számítási feladatok üzembe helyezése. A valóságban Ez leginkább nagy szervezetek is használnak, amely számos különböző számítási feladatok dolgozunk egy időben számos csapat rendelkezik. Hiányol, mert egy egyszerű cégirányítási modell nem több szervezeti összetett és fejlesztési forgatókönyvek kezeléséhez elegendő.

A lépéseknek az ismertetése, a Azure bevezetését köztes szakaszában a cégirányítási modellje több csapat dolgozik a több új Azure-fejlesztési számítási feladatot tervez.  

Az útmutató ezen szakasza a célközönségét a következő személyeknek a szervezeten belül:
- *Pénzügyi:* a pénzügyi kötelezettségvállalás nélkül, az Azure-ba, szabályzatokat és eljárásokat erőforrás használat költségeit, beleértve a számlázás és költséghelyi elszámolás kialakításáért felelős tulajdonosa.
- *Központi informatikai:* felelős erőforrás-kezelést és a hozzáférés, beleértve a szervezet felhőbeli erőforrások, valamint a számítási feladatok állapotának és figyeléséhez.
- *Megosztott infrastruktúra tulajdonosa*: felelős műszaki szerepkörök hálózati kapcsolat a helyszínről a felhőbe.
- *Biztonsági műveletek*: vállal felelősséget az ehhez a biztonsági szabályzat kiterjesztése a megvalósítása a helyszíni biztonsági határ Azure-ra. Előfordulhat, hogy is az Azure-ban a saját biztonsági infrastruktúra titkos kulcsok tárolásához.
- *Számítási feladatok felelőse:* felelős közzététele a számítási feladatok Azure-bA. Attól függően, a szervezet fejlesztői csapatok felépítését ez lehet egy fejlesztési vezető, program vezető vagy műszaki vezető build. A közzétételi folyamat része lehet olyan az üzembe helyezés az Azure-erőforrások.
  - *Számítási feladatok közreműködői:* hozzájárul a közzététel az Azure-bA a számítási feladat felelős. Megkövetelheti a figyelési és hangolási teljesítmény olvasási hozzáféréssel az Azure-erőforrásokhoz. Nem szükséges engedélyek létrehozása, frissítése, vagy törölje az erőforrást.

## <a name="section-1-azure-concepts-for-multiple-workloads-and-multiple-teams"></a>1. szakasz: Azure fogalmak több és több csapat

Az alapszintű bevezetési szakasz, néhány alapvető tudnivalók az Azure belső elemeinek megtanult és erőforrások jönnek létre, olvassa el, hogyan frissíthetők és törölhetők. Azt is megtanulta kapcsolatos identitás, és, hogy az Azure csak megbízik az Azure Active Directory (AD), akik hozzáférhetnek ezeket az erőforrásokat a felhasználók hitelesítéséhez és engedélyezéséhez.

Szintén elindul megismerkedett a konfigurálása az Azure cégirányítási eszközök használatát a szervezet Azure-erőforrások kezeléséhez. Megvizsgáltunk, hogyan szabályozzák a hozzáférést egy csoportot egy egyszerű számítási feladat üzembe helyezéséhez szükséges erőforrásokat. A valóságban a végzett a szervezet fog dolgozik több számítási feladat egyszerre több csapat áll. 

Mielőtt vessünk egy pillantást milyen kifejezés **munkaterhelés** ténylegesen jelenti. Egy kifejezés, amely általában egy tetszőleges egysége funkciók, például egy alkalmazás vagy szolgáltatás meghatározásához érthető legyen. Azt gondolja végig egy számítási feladat tekintetében a kód összetevők telepített egy kiszolgáló, valamint más szolgáltatásokat, például egy adatbázis szükséges. Ez hasznos definícióját egy helyszíni alkalmazást vagy szolgáltatást, de a felhőben, bontsa ki kell. 

A felhőben a munkaterhelés nem csak magában foglalja a minden összetevő, de is tartalmaz a felhőbeli erőforrásokat is. Azt a felhőbeli erőforrások részét képező a definíció egy fogalom, más néven miatt **infrastruktúra mint kód**. Az "Azure működése" ismertető szokásaival az Azure-erőforrások az orchestrator szolgáltatás által telepített. Az orchestrator szolgáltatás elérhetővé teszi ezt a funkciót, a webes API-k és a webes API nem hívható meg több eszközök, például az Azure portal, Powershell és az Azure parancssori felület (CLI) használatával. Ez azt jelenti, hogy adható meg az erőforrásokat a kód összetevőket az alkalmazáshoz társított együtt tárolt gépi olvasásra alkalmas fájlban.

Ez lehetővé teszi számunkra meghatározásához egy számítási feladat tekintetében kód összetevők és a szükséges felhőbeli erőforrásokat, és a további lehetővé teszi számunkra, hogy **elkülönítése** a számítási feladatokat. Számítási feladatok lehet elkülönített egyébként erőforrások vannak rendszerezve, hálózati topológia, vagy egyéb attribútumokkal. A számítási feladatok elkülönítése célja, hogy egy csapat a számítási feladat erőforrásoknál társítani, a csapat egymástól függetlenül kezelheti ezeket az erőforrásokat minden aspektusát. Ez lehetővé teszi több a csapatok megoszthatnak erőforrás-kezelési szolgáltatások az Azure-ban megakadályozza, hogy az a véletlen törlés vagy módosítására a többi összes erőforrást.

Ez az elkülönítés is lehetővé teszi egy másik fogalom, más néven [fejlesztési és üzemeltetési](https://azure.microsoft.com/solutions/devops/). Fejlesztési és üzemeltetési a szoftverfejlesztési gyakorlatban, amely tartalmazza a szoftverek fejlesztési és operatív informatikai is tartalmaz, de hozzáadja a lehető legnagyobb mértékben automatizálást használni. A fejlesztési és üzemeltetési alapelveit egyik folyamatos integrációt és folyamatos készregyártás (CI/CD) néven ismert. Folyamatos integráció az automatizált összeállítási folyamatairól minden alkalommal, amikor egy fejlesztő véglegesítések kódváltoztatást futtatott hivatkozik, és a folyamatos teljesítés folyamatokra vonatkozik, az automatikus, hogy ez a kód üzembe helyezni különböző **környezetek** például egy **fejlesztési környezet** tesztelési vagy egy **éles környezetben** végső üzembe helyezéshez.

## <a name="section-2-governance-design-for-multiple-teams-and-multiple-workloads"></a>2. szakasz: Több csapat és a több számítási feladat Cégirányítási megtervezése

Az a [eligazodást fázis](/azure/architecture/cloud-adoption-guide/adoption-intro/overview) , az Azure cloud bevezetési útmutató mutatta be a felhő cégirányítási fogalmát. Megtudhatta, hogyan tervezhet egy egyetlen csapat dolgozik egy egyetlen számítási feladat egy egyszerű cégirányítási modellt. 

A köztes fázisban a [cégirányítási kialakítási útmutató](governance-design-guide.md) több csapat, több számítási feladatot és több környezethez hozzáadandó alaprendszeri alapfogalmait tartalmazó gyűjteménnyel bővítjük. Miután a dokumentumban szereplő példák forrásanyagot a tervezési alapelvek alkalmazhat, amelyek segítenek a szervezet goverance modell.

## <a name="section-3-implementing-a-resource-management-model"></a>3. szakasz: Erőforrás-felügyeleti modellt megvalósítása

A szervezet felhőalapú cégirányítási modell az Azure resource kezelési eszközök, az emberek és meghatározta access management szabályok között metszetében jelöli. Az goverance tervezési útmutatóban megismerhette számos különböző modellek az Azure-erőforrásokhoz való hozzáférést szabályozzák. Most végigvezetjük a mindegyike egy előfizetéssel rendelkező felügyeleti erőforrásmodell megvalósításához szükséges lépéseket a **megosztott infrastruktúra**, **éles**, és **fejlesztési** tervezési útmutató a környezeteket. Beállítjuk, hogy az egyik **előfizetés tulajdonosa** három környezetekhez. Minden számítási feladat fog különíthető el a egy **erőforráscsoport** együtt egy **számítási feladatok felelőse** szolgáltatással együtt a **közreműködői** szerepkör.

> [!NOTE]
> Olvasási [az Azure-beli erőforrás-hozzáférésének megismerése] [ understand-resource-access-in-azure] tudhat meg többet az Azure-fiókok és előfizetések közötti kapcsolat. 

Kövesse az alábbi lépéseket:

1. Hozzon létre egy [Azure-fiók](/azure/active-directory/sign-up-organization) , ha a szervezete még nincs ilyen. A regisztráló felhasználót állítja be az Azure-fiók az Azure-fiók rendszergazdája lesz, és a szervezet vezetői jelöljön ki egy személy feltételezzük ezzel a szerepkörrel. Ez a személy lesz felelős:
    * Előfizetés létrehozása és
    * Létrehozása és felügyelete [Azure Active Directory (AD)](/azure/active-directory/active-directory-whatis) bérlők számára, amelyek tárolják a felhasználó-identitást ezen előfizetések.    
2. A szervezet vezetőségi úgy dönt, hogy mely személyek felelősek.
    * Felhasználói identitás; kezelése egy [Azure AD-bérlő](/azure/active-directory/develop/active-directory-howto-tenant) alapértelmezés szerint jön létre, ha a szervezet Azure-fiók létrejön, és a fiók rendszergazdája kerülnek be a [Azure AD globális rendszergazda](/azure/active-directory/active-directory-assign-admin-roles-azure-portal#details-about-the-global-administrator-role) alapértelmezés szerint. A szervezet választhat egy másik felhasználó által a felhasználói identitások kezelésére [, hogy a felhasználó az Azure AD globális rendszergazdai szerepkör hozzárendelése](/azure/active-directory/active-directory-users-assign-role-azure-portal). 
    * Olyan előfizetéseket, ami azt jelenti, hogy ezek a felhasználók:
        * Erőforrás-használat az adott előfizetéshez társított költségek kezelése
        * Megvalósításához és fenntartásához legalább engedélyezési modellekkel az erőforrások eléréséhez, és
        * Nyomon követheti, szolgáltatási korlátai.
    * Megosztott infrastruktúra-szolgáltatások (Ha a szervezet úgy dönt, hogy ez a modell), ami azt jelenti, ez a felhasználó felelős:
        * Az Azure hálózati kapcsolatot, a helyszíni és a 
        * Hálózati kapcsolat Azure-beli virtuális hálózati társviszony-létesítésen keresztül tulajdonjogát.
    * Számítási feladatok tulajdonosai. 
3. Az Azure AD globális rendszergazda [hoz létre az új felhasználói fiókok](/azure/active-directory/add-users-azure-active-directory) számára:
    * A személy, aki a **előfizetés tulajdonosa** minden környezethez társított minden egyes előfizetés esetén. Vegye figyelembe, hogy ez a szükséges csak akkor, ha az előfizetés **szolgáltatás-rendszergazda** nem lesz az összes előfizetés környezet erőforrás-hozzáférés kezelése biztosítja.
    * A személy, aki a **hálózati műveletek felhasználó**, és
    * A személyek, akik **munkaterhelés tulajdonost**.
4. Az Azure-fiók rendszergazdai hoz létre a következő három előfizetés használatával a [Azure fiókportálon](https://account.azure.com):
    * Előfizetés a **megosztott infrastruktúra** környezetben,
    * Előfizetés a **éles** környezetben, és 
    * Előfizetés a **fejlesztési** környezetben. 
5. Az Azure-fiók rendszergazdai [az előfizetés szolgáltatás tulajdonosa ad hozzá minden egyes előfizetés](/azure/billing/billing-add-change-azure-subscription-administrator#add-an-rbac-owner-admin-for-a-subscription-in-azure-portal).
6. A jóváhagyási folyamat létrehozása **számítási feladatok** erőforráscsoportok létrehozása kéréséhez. A jóváhagyási folyamat implementálható számos módon, például e-mailben, vagy beállíthatja a folyamat felügyeleti eszköz használatával, mint például a [Sharepoint munkafolyamatok](https://support.office.com/article/introduction-to-sharepoint-workflow-07982276-54e8-4e17-8699-5056eff4d9e3). A jóváhagyási folyamat kövesse az alábbi lépéseket:  
    * A **számítási feladatok felelőse** szükséges Azure-erőforrásokat akár anyagjegyzék előkészíti a **fejlesztési** környezet, **éles** környezetet, vagy mindkettőt, és elküldi azt a **előfizetés tulajdonosa**.
    * A **előfizetés tulajdonosa** áttekinti az anyagjegyzék, és érvényesíti a kért erőforrások annak érdekében, hogy a kért erőforrások a tervezett használható – például ellenőrzését, amely a kért [ Virtuálisgép-méretek](/azure/virtual-machines/windows/sizes) helyes-e.
    * Ha a kérés nem engedélyezett, a **számítási feladatok felelőse** értesítést kap. Ha a kérelmet jóváhagyták, a **előfizetés tulajdonosa** [a kért erőforráscsoportot hoz létre](/azure/azure-resource-manager/resource-group-portal#manage-resource-groups) betartják a munkahelyen [elnevezési konvenciók](/azure/architecture/best-practices/naming-conventions), [ hozzáadja a **számítási feladatok felelőse** ](/azure/role-based-access-control/role-assignments-portal#add-access) együtt a [ **közreműködői** szerepkör](/azure/role-based-access-control/built-in-roles#contributor) , és értesítést küld a **számításifeladatokfelelőse** , amely az erőforráscsoport létrehozása.
7. Hozzon létre olyan jóváhagyási folyamatot, a virtuális hálózati társviszonyt a kapcsolat a megosztott infrastruktúra tulajdonosától kérhet a munkaterhelések tulajdonosai számára. Csakúgy, mint az előző lépésben, a jóváhagyási folyamat segítségével valósítható meg e-mailt vagy egy folyamat felügyeleti eszközt.

Most, hogy a cégirányítási modell megvalósítása a megosztott infrastruktúra-szolgáltatást is üzembe helyezhet.

## <a name="section-4-deploy-shared-infrastructure-services"></a>4. szakasz: megosztott infrastruktúra-szolgáltatások üzembe helyezése

Több [hibrid hálózat referenciaarchitektúrák](/azure/architecture/reference-architectures/hybrid-networking/) , hogy a szervezet használatával a helyszíni hálózat csatlakoztatása az Azure-bA. Ezek a referenciaarchitektúrák mindegyike tartalmaz egy központi telepítés egy előfizetés-azonosító szükséges. A telepítés során adja meg az előfizetés-azonosító az előfizetéshez társított a **infrastructre megosztott** környezetben. Adja meg az erőforráscsoportot, amely felügyeli a sablonfájlokat szerkesztése is kell a **hálózati műveletek** felhasználói, vagy, akkor is az alapértelmezett erőforráscsoportok használata az üzemelő példányban, és adja hozzá a **hálózati műveletek**  rendelkező felhasználó a **közreműködői** szerepkör hozzájuk.

<!-- links -->
[understand-resource-access-in-azure]: /azure/role-based-access-control/rbac-and-directory-admin-roles