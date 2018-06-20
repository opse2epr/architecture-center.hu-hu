---
title: 'Azure bevezetése: köztes'
description: A közbenső szintű Tudásbázis, hogy a vállalati szüksége van Azure elfogadása
author: petertay
ms.openlocfilehash: 227d9558647ed8076b2832d95e192f2f0c43b9db
ms.sourcegitcommit: 26b04f138a860979aea5d253ba7fecffc654841e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 06/19/2018
ms.locfileid: "36206361"
---
# <a name="azure-cloud-adoption-guide-intermediate-overview"></a>Azure-felhőbe bevezetési útmutató: Köztes áttekintése

A legalapvetőbb elfogadása fázisban mutatta az Azure-erőforrás irányítás főbb fogalmait és kifejezéseit. A legalapvetőbb szakasz úgy lett kialakítva, az első lépések az Azure elfogadása út, és központi telepítése egy kis csoportot egy egyszerű munkaterhelés keresztül telefonon meg. A valóságban legtöbb nagy szervezet már rendelkezik számos olyan csoportok, amelyek egyszerre sok különböző terhelésekhez dolgoznak. Teheti meg, mert egy egyszerű irányítás modell nincs elegendő több szervezeti komplex és fejlesztési forgatókönyvek kezeléséhez.

A fókusz a köztes szakasz Azure elfogadásának több csapat dolgozik a több új Azure fejlesztési munkaterhelés a cégirányítási modellt tervez.  

A felhasználók, akik az útmutató ezen szakasza a következő személyeknek a szervezeten belül:
- *Pénzügyi:* Azure, a házirendek és eljárások erőforrás fogyasztás költségeit, beleértve a számlázási és jóváírási felelős pénzügyi kötelezettségvállalást tulajdonosa.
- *Központi informatikai:* felelős a szervezet felhőbeli erőforrásokat, beleértve az erőforrás-kezelés és a hozzáférést, valamint a munkaterhelés állapotának és figyelésére.
- *A megosztott infrastruktúra tulajdonos*: technikai szerepkörök feladata a hálózati kapcsolatot a felhőbe helyszíni.
- *Biztonsági műveletek*: felelős végrehajtási biztonsági házirend ki kell terjeszteni a helyszíni biztonsági határ Azure tartalmazza. Előfordulhat, hogy is saját biztonsági infrastruktúra az Azure-ban titkos kulcsok tárolásához.
- *Számítási feladatok felelőse:* felelős a munkaterhelés közzététele az Azure-bA. Attól függően, hogy a szervezet a fejlesztési csapat felépítését ez lehet a fejlesztési segítséget, a program vezető vagy érdeklődési mérnöki build. A közzétételi folyamat részeként a központi telepítés a Azure-erőforrások is tartalmazhat.
  - *Munkaterhelés közreműködői:* hozzájárul a munkaterhelés Azure közzétételi felelős. Olvasási hozzáférés Azure-erőforrások lehet szükség a teljesítmény figyelése, vagy hangolása. Nem szükséges engedéllyel létrehozására, frissítésére, és törli az erőforrást.

## <a name="section-1-azure-concepts-for-multiple-workloads-and-multiple-teams"></a>1. szakasz: Azure több munkaterhelés és több csapat fogalmak

A legalapvetőbb elfogadása fázisban, megtudta, néhány alapvető tudnivalók az Azure belső funkciói és, valamint hogyan erőforrások létrehozása, olvasása, frissítése és törlése. Identitására vonatkozó is megtanulta, és adott Azure csak megbízik az Azure Active Directory (AD) hitelesítéséhez és engedélyezéséhez a felhasználók, akik hozzá az erőforrásokhoz.

Is használatba az Azure irányítás eszközök kezeléséhez a szervezet Azure-erőforrások konfigurálása megismerését. A legalapvetőbb szakaszában azt venni, hogyan egy csoportot egy egyszerű alkalmazások és szolgáltatások telepítéséhez szükséges erőforrásokhoz való hozzáférésre. A valóságban a szervezet lesz áll, több csapat dolgozik a több munkaterhelés egyidejűleg. 

Mielőtt megkezdjük, vessen egy pillantást milyen kifejezés **munkaterhelés** valójában jelenti. Egy kifejezés, amely általában nincs tisztában funkciók, például egy alkalmazás vagy szolgáltatás egy tetszőleges egysége meghatározásához. Azt gondolja át a munkaterhelések szempontjából a kiszolgáló, valamint a szükséges egyéb szolgáltatások, például egy adatbázis üzembe helyezett kódot az összetevők. Ez egy helyi alkalmazás vagy szolgáltatás hasznos definícióját, de a felhőben igazolnia kell a rajta bontsa ki. 

A felhőben a munkaterhelés nem csak magában foglalja az összetevőket, de is tartalmazza, valamint a felhőben található erőforrásokat. Magában foglalja az felhőalapú erőforrásokat a definíciójának részeként miatt a fogalom, mint a **infrastruktúra, kód**. A "hogyan Azure működik a" explainer megismert az Azure-erőforrások az orchestrator szolgáltatás telepítése. Az orchestrator szolgáltatás elérhetővé teszi ezt a funkciót egy webes API-t, és a webes API hívása több eszközökkel, például a Powershell, az Azure parancssori felület (CLI) és az Azure-portálon keresztül. Ez azt jelenti, hogy az erőforrások adható meg az alkalmazással társított kódot az összetevők mellett tárolhatja géppel olvasható fájlban.

Ez lehetővé teszi, hogy a kód összetevők és a szükséges felhőben lévő erőforrások és a további lehetővé teszi, hogy a munkaterhelések meghatározása, hogy **elkülönítése** a munkaterhelések. Munkaterhelések elkülöníthető a erőforrások vannak rendezve, hálózati topológia, vagy egyéb attribútumai. Munkaterhelések elkülönítése célja egy adott erőforráshoz a munkaterhelési csoporthoz hozzárendelni, a csapat egymástól függetlenül kezelheti ezeket az erőforrásokat minden szempontját. Ez lehetővé teszi több a csapatok megoszthatnak erőforrás szolgáltatások az Azure-ban közben megakadályozza a véletlen törlés vagy egymás erőforrások módosítása.

Elkülönítés is lehetővé teszi, hogy egy másik fogalom, mint a [DevOps](https://azure.microsoft.com/solutions/devops/). DevOps magában foglalja a szoftver fejlesztési eljárásokat, amelyek magukban foglalják a szoftverfejlesztői, és a fenti IT-üzemeltetők, de használni a lehető legnagyobb mértékben hozzáadja. DevOps alapelvei egyik folyamatos integrációt és folyamatos kézbesítési (CI/CD) néven ismert. Folyamatos integrációt minden alkalommal, amikor egy fejlesztő véglegesíti kódváltoztatást futtatott automatizált build folyamatokat foglalja magában, és folyamatos kézbesítési hivatkozik az, hogy ez a kód különféle automatikus folyamatot **környezetek** , mint egy **fejlesztőkörnyezet** tesztelési vagy egy **éles környezetben** végső központi telepítéshez.

## <a name="section-2-governance-design-for-multiple-teams-and-multiple-workloads"></a>2. szakasz: Irányítás kialakítása több csapat és több munkaterhelés

Az a [eligazodást szakasz](/azure/architecture/cloud-adoption-guide/adoption-intro/overview) az Azure-felhőbe bevezetési útmutató bevezetett felhő irányítás, amelynek. Megtudta, hogyan tervezhető olyan egyszerű irányítás modell dolgozik a egyetlen munkaterhelési csoportot. 

A köztes fázisban a [irányítás tervezési útmutató](governance-design-guide.md) több csapat, több munkaterhelés, és több környezetek hozzáadása a legalapvetőbb fogalmak kibővíti. Miután, de a dokumentum a példákból alkalmazhat a tervezési alapelvek tervezése és megvalósítása a szervezet goverance modell.

## <a name="section-3-implementing-a-resource-management-model"></a>3. szakasz: A felügyeleti modell megvalósítása

A szervezete cloud irányítás modell Azure resource access felügyeleti eszközök, a munkatársak és a hozzáférés-kezelési meghatározta szabályok között metszetének jelöli. Az goverance tervezési útmutatójának megismerte az Azure-erőforrások elérésére vonatkozó több különböző modell. Most végigvezetjük a felügyeleti modell egy előfizetéssel az egyes végrehajtásához szükséges lépéseket a **megosztott infrastruktúra**, **éles**, és **fejlesztési** a tervezési útmutatóból környezetekben. Igazolnia kell egy **előfizetés tulajdonosa** minden három környezethez. Egyes munkaterhelések fog lehet különíteni az egy **erőforráscsoport** rendelkező egy **számítási feladatok felelőse** hozzá, amelyeknél a **közreműködő** szerepkör.

> [!NOTE]
> Olvasási [az az Azure-erőforrások hozzáférésének megismerése] [ understand-resource-access-in-azure] tudhat meg többet az Azure-fiókok és -előfizetések közötti kapcsolat. 

Kövesse az alábbi lépéseket:

1. Hozzon létre egy [Azure-fiók](/azure/active-directory/sign-up-organization) Ha a szervezet nem rendelkezik. A személy, aki az Azure-fiókot regisztrál az Azure-fiók rendszergazdája lesz, és a szervezet vezetőségének munkáját segítik ki kell választania egy adott számára, hogy ezt a szerepkört. Ez a személy felelős:
    * Előfizetések létrehozása és
    * létrehozása és felügyelete [Azure Active Directory (AD)](/azure/active-directory/active-directory-whatis) bérlői, felhasználói identitást előfizetésekben tárolja.    
2. A szervezet egyik vezetőségi tagja úgy dönt, hogy mely személyek felelősek.
    * Felhasználói azonosító; kezelése egy [Azure AD-bérlő](/azure/active-directory/develop/active-directory-howto-tenant) alapértelmezés szerint jön létre, ha a szervezet Azure-fiók létrehozása, és a fiókadminisztrátor meg van adva a [Azure AD globális rendszergazda](/azure/active-directory/active-directory-assign-admin-roles-azure-portal#details-about-the-global-administrator-role) alapértelmezés szerint. A szervezet választhat egy másik felhasználó által a felhasználói identitás kezelése [az Azure AD globális rendszergazdai szerepkör hozzárendelése, hogy a felhasználó](/azure/active-directory/active-directory-users-assign-role-azure-portal). 
    * Előfizetések, ami azt jelenti, hogy ezek a felhasználók:
        * erőforrás-használat, hogy az előfizetéshez kapcsolódó költségeket kezelése
        * bevezetéséhez és karbantartásához legkevesebb engedélyt modell az erőforrások eléréséhez, és
        * nyomon követjük, szolgáltatásra vonatkozó korlátozások.
    * Megosztott infrastruktúra-szolgáltatásokat (Ha a szervezet úgy dönt, hogy ez a modell), ami azt jelenti, a felhasználónak kell megőrizni:
        * a helyszíni Azure hálózati kapcsolatot, és 
        * a hálózati kapcsolat az Azure virtuális hálózati társviszony-létesítés keresztül tulajdonjogát.
    * Munkaterhelések tulajdonosai. 
3. Az Azure AD globális rendszergazda [hoz létre az új felhasználói fiókok](/azure/active-directory/add-users-azure-active-directory) esetében:
    * a személy, aki a **előfizetés tulajdonosa** minden egyes környezet előfizetéshez. Vegye figyelembe, hogy ez csak akkor, ha az előfizetés **szolgáltatás-rendszergazda** való előfizetés/környezetben erőforrás-hozzáférés kezeléséhez nem lehet szolgáltatásait.
    * a személy, aki a **hálózati műveletek felhasználói**, és
    * a személyek **munkaterhelés tulajdonos(ok)**.
4. Az Azure-fiók rendszergazda hoz létre a következő három előfizetések használata a [Azure-fiók portálon](https://account.azure.com):
    * az előfizetés a **megosztott infrastruktúra** környezetben
    * az előfizetés a **éles** környezetben, és 
    * az előfizetés a **fejlesztési** környezetben. 
5. Az Azure-fiók rendszergazdai [az előfizetés szolgáltatás tulajdonosa hozzá minden egyes előfizetés](/azure/billing/billing-add-change-azure-subscription-administrator#add-an-rbac-owner-admin-for-a-subscription-in-azure-portal).
6. Hozzon létre a jóváhagyási folyamatot **munkaterhelések tulajdonosai** lekérni az erőforrás-csoportok létrehozását. A jóváhagyási folyamatot valósítható számos szempont, például keresztül e-mailek, vagy beállíthatja, hogy egy folyamat felügyeleti eszköz használatával, mint a [Sharepoint munkafolyamatok](https://support.office.com/article/introduction-to-sharepoint-workflow-07982276-54e8-4e17-8699-5056eff4d9e3). A jóváhagyási folyamatot is kövesse az alábbi lépéseket:  
    * A **számítási feladatok felelőse** egyaránt szükség Azure-erőforrások az anyagjegyzék előkészíti a **fejlesztési** környezetben **éles** környezet, illetve mindkettőt, és elküldi a úgy, hogy a **előfizetés tulajdonosa**.
    * A **előfizetés tulajdonosa** ellenőrzi, hogy az anyagjegyzék, és érvényesíti a kért erőforrások, és győződjön meg arról, hogy a kért erőforrások a tervezett használhatók az - például ellenőrzését, amely a kért [ Virtuálisgép-méretek](/azure/virtual-machines/windows/sizes) helyes-e.
    * A kérelem nem engedélyezett, ha a **számítási feladatok felelőse** értesítést kap. Ha a kérelem jóváhagyták, a **előfizetés tulajdonosa** [a kért erőforrás csoportot hoz létre](/azure/azure-resource-manager/resource-group-portal#manage-resource-groups) a szervezet a következő [elnevezési konvenciói](/azure/architecture/best-practices/naming-conventions), [ hozzáadja a **számítási feladatok felelőse** ](/azure/role-based-access-control/role-assignments-portal#add-access) rendelkező a [ **közreműködő** szerepkör](/azure/role-based-access-control/built-in-roles#contributor) és értesítést küld a **számítási feladatok felelőse** az erőforráscsoport lett létrehozva.
7. A munkaterhelések tulajdonosai kérjen egy virtuális hálózati kapcsolat a megosztott infrastruktúra tulajdonos társviszony jóváhagyási folyamatot létrehozni. Csakúgy, mint az előző lépésben a jóváhagyási folyamat implementálhatók e-mail vagy egy folyamat felügyeleti eszköz használatával.

Most, hogy a cégirányítási modell bevezetése a megosztott infrastruktúra-szolgáltatásokat is telepíthet.

## <a name="section-4-deploy-shared-infrastructure-services"></a>4. szakasz: megosztott infrastruktúra-szolgáltatások telepítése

Több [hibrid hálózati referencia architektúra](/azure/architecture/reference-architectures/hybrid-networking/) , hogy a szervezet csatlakozhatnak a helyszíni hálózat az Azure-bA. A referencia architektúra mindegyikének magában foglalja a központi telepítések, szükség van egy előfizetési azonosító. A telepítés során adja meg az előfizetéshez tartozó előfizetés-azonosító a **infrastructre megosztott** környezetben. Is szüksége lesz a sablon fájlok szerkesztésével adja meg az erőforráscsoportot, amely kezeli a **hálózati műveletek** felhasználói, vagy, akkor is az alapértelmezett erőforrás-csoportok használata a központi telepítésben lévő, és adja hozzá a **hálózati műveletek**  rendelkező felhasználó az **közreműködő** szerepkör hozzájuk.

<!-- links -->
[understand-resource-access-in-azure]: /azure/role-based-access-control/rbac-and-directory-admin-roles