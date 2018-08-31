---
title: 'Enterprise Cloud Adoption: Cégirányítási tervezése az Azure-ban több csapat'
description: Útmutató a több csapat, több számítási feladatot és több környezetet az Azure cégirányítási vezérlők konfigurálása
author: petertaylor9999
ms.openlocfilehash: 2ad9fac6604d2766fed1df828f63e65c8a570888
ms.sourcegitcommit: ae8a1de6f4af7a89a66a8339879843d945201f85
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 08/31/2018
ms.locfileid: "43326691"
---
# <a name="enterprise-cloud-adoption-governance-design-for-multiple-teams"></a>Enterprise Cloud Adoption: Cégirányítási tervezés több fejlesztőcsapatoknak

Ez az útmutató célja, amely megkönnyíti a folyamatot az Azure-ban a több csapat, több számítási feladatot, és több környezetet erőforrás cégirányítási modell létrehozása.  Azt fogjuk elméleti cégirányítási követelményeket tekintse meg, majd nyissa meg több szolgálnak példákkal, amelyek megfelelnek-e ezeknek a követelményeknek.

> [!NOTE]
> A részletes leírásáért lásd **környezetek** tekintse meg a 

Követelmények:
* A vállalati átmenet új felhőalapú szerepkörökkel és a felhasználók felelőssége terveket, és ezért van szükség az Identitáskezelés az Azure-ban különböző erőforrás-hozzáférési igényekkel rendelkező több csapatok számára. A következő felhasználók identitását tárolásához az identitáskezelési rendszerekkel van szükség:
  1. A szervezet tulajdonjogának felelős személy **előfizetések**.
  2. Az egyes felelős a szervezetben a **megosztott infrastruktúra-erőforrások** segítségével a helyszíni hálózat csatlakoztatása az Azure-beli virtuális hálózathoz. 
  3. Két egyéni felhasználók számára a szervezetben kezeléséért egy **munkaterhelés**. 
* Több **környezetek**. Környezet az erőforrások, például a virtual machines, a virtuális hálózat és a hálózati forgalom-útválasztási szolgáltatások logikai csoportosítása. Ezeket a csoportokat az erőforrások hasonló felügyeleti és biztonsági követelményekkel rendelkeznek, és általában egy adott célhoz, például a tesztelési vagy éles környezetben használják. Ebben a példában a követelmény a három környezetekhez:
  1. A **megosztott infrastruktúra** , amely tartalmazza az erőforrások más környezetekben a számítási feladatok által közösen. Például egy virtuális hálózatot az átjáró-alhálózatot, amely kapcsolatot biztosít a helyszínen.
  2. A **éles környezetben** a legszigorúbb biztonsági házirendeknek. Előfordulhat, hogy belső vagy külső internetkapcsolattal rendelkező számítási feladatok közé tartozik.
  3. A **fejlesztési környezet** proof-of-concept és tesztelési munkához. Ebben a környezetben rendelkezik biztonsági, megfelelőségi és költség házirendek, amelyek különböznek az éles környezetben.
* A **engedélyek modelljét legalacsonyabb jogosultsági** amelyben a felhasználók nem engedélyek alapértelmezés szerint rendelkezik. A modell támogatnia kell a következőket:
  * Egy megbízható egyfelhasználós engedéllyel az erőforrás-hozzáférési engedélyek hozzárendelése az előfizetések szintjén.
  * Minden egyes számítási feladatok felelőse alapértelmezés szerint erőforrásokhoz való hozzáférés megtagadva. Erőforrás hozzáférési jogosultságokat explicit módon a egyetlen megbízható felhasználó az előfizetések szintjén.
  * Felügyeleti hozzáférés a megosztott infrastruktúra-erőforrások, a megosztott infrastruktúra tulajdonosa korlátozódik.  
  * Minden számítási feladat a számítási feladatok tulajdonosának korlátozott felügyeleti hozzáférés.
  * A vállalat nem szeretné kell kezelnie a szerepkörök egymástól függetlenül a három környezeteket, ezért a használatát igényli [beépített RBAC-szerepkör] [ rbac-built-in-roles] csak. Ha a vállalat egyéni RBAC-szerepkörök, egy folyamat szükséges egyéni szerepkörök szinkronizálása a három környezetek között. 
* Költségek nyomon követése számítási feladatok tulajdonosának neve, a környezet vagy mindkettő. 

## <a name="identity-management"></a>Identitáskezelés

Azt is tervezhet a cégirányítási modell az Identitáskezelés, mielőtt fontos tudni, hogy a szerepkör magában foglalja a négy fő területet:

* Felügyelet: a folyamatok és eszközök számára létrehozása, szerkesztése és törlése a felhasználói identitás.
* Hitelesítés: felhasználó hitelesített hitelesítő adatokat, például egy felhasználónév és jelszó ellenőrzése
* Hitelesítés: erőforrások meghatározása egy hitelesített felhasználó számára engedélyezett a hozzáférés, illetve milyen műveleteket rendelkeznek engedéllyel az.
* Naplózás: naplók és egyéb biztonsági problémák felderítéséhez információkat rendszeres időközönként áttekintésével kapcsolatos felhasználói identitás. Ide tartoznak a gyanús használati mintákat, rendszeres időközönként a pontos, vannak-e a felhasználói engedélyek áttekintése és egyéb funkciók áttekintése.

Az Azure-identitás megbízható csak egy szolgáltatás, és ez az Azure Active Directory (Azure AD). Azt az Azure AD-felhasználók hozzáadása és használja a fent felsorolt összes lesz. Azonban mielőtt áttekintjük, hogyan konfiguráljuk az Azure ad-ben, fontos tudni, hogy a kiemelt jogosultságú fiókok ezekhez a szolgáltatásokhoz való hozzáférés kezelésére használhatók.

Ha a szervezet regisztrált az Azure-fiókkal, legalább egy Azure **fióktulajdonos** hozzá volt rendelve. Emellett az Azure AD **bérlői** lett létrehozva, kivéve, ha egy meglévő bérlőt már hozzá lett rendelve a szervezet más Microsoft-szolgáltatások, például az Office 365 használatát. A **globális rendszergazdai** teljes körű engedélyekkel az Azure AD bérlő lett társítva létrehozásakor. 

Az Azure-fiók tulajdonosa és az Azure AD globális rendszergazda a felhasználói identitásokat egy rendkívül biztonságos identitás rendszer, amely a Microsoft által felügyelt vannak tárolva. Az Azure-fiók tulajdonosa létrehozása, frissítése és törlése előfizetések jogosult. Az Azure AD globális rendszergazda az Azure ad-ben számos művelet végrehajtására jogosult, de ez a kialakítási útmutató a létrehozását és törlését a felhasználói identitás az fogunk összpontosítani. 

> [!NOTE]
> A szervezet előfordulhat, hogy már rendelkezik meglévő Azure AD-bérlővel, ha egy meglévő Office 365 vagy az Intune-licencet a fiókjához társított.

Az Azure-fiók tulajdonosa van engedélye létrehozása, frissítése és törlése előfizetések: 

![Azure-fiókot az Azure Account Manager és az Azure AD globális rendszergazda](../_images/governance-3-0.png)
*1. ábra. Egy Azure-fiók Account Manager, és az Azure AD globális rendszergazda.*

Az Azure AD **globális rendszergazdai** jogosult arra, hogy felhasználói fiókokat hozhat létre:  

![Azure-fiókot az Azure Account Manager és az Azure AD globális rendszergazda](../_images/governance-3-0a.png)
*2. ábra. Az Azure AD globális rendszergazda a bérlő hoz létre a szükséges felhasználói fiókokat.*

Az első két fiókok **App1 számítási feladatok felelőse** és **App2 számítási feladatok felelőse** van minden olyan személy, a szervezet a számítási feladatok kezeléséért társított. A **hálózati műveletek** fiókot a megosztott infrastruktúra-erőforrások felelős személy tulajdonában van. Végül a **előfizetés tulajdonosa** fiókhoz társítva az előfizetések tulajdonjogának felelős személy.

## <a name="resource-access-permissions-model-of-least-privilege"></a>Hozzáférési engedélyek erőforrásmodell legalacsonyabb jogosultsági

Most, hogy az identitás felügyeleti rendszer- és felhasználói fiókok létrehozása után dönthet arról, hogy hogyan tudjuk alkalmazni szerepköralapú hozzáférés-vezérlés (RBAC) szerepkört minden egyes fiókhoz a legalacsonyabb jogosultsági engedélyek modellje van.

Egy másik követelmény figyelmezteti a minden számítási feladathoz társított erőforrások legyen különítve egymástól, úgy, hogy nem egy számítási feladatok felelőse bármely más számítási feladatok nem saját felügyeleti hozzáféréssel rendelkezik. Emellett van ez a modell használatával csak megvalósításához szükséges [Azure RBAC beépített szerepkörei][rbac-built-in-roles].

Minden egyes RBAC szerepkör alkalmazzák az Azure-ban három hatókörök valamelyikére: **előfizetés**, **erőforráscsoport**, majd egy egyéni **erőforrás**. Szerepkörök, az alacsonyabb hatókörök öröklődnek. Például, ha egy felhasználó hozzá van rendelve a [beépített tulajdonosi szerepkör] [ rbac-built-in-owner] az előfizetés szintjén, hogy a szerepkör is hozzá van rendelve a felhasználó az erőforráscsoportot és az egyes erőforrások szintjén, bírálni.

Ezért legkisebb jogosultság hozzáférési döntse el, a műveleteket kell, hogy egy modell létrehozásához egy bizonyos típusú felhasználó számára engedélyezett a minden egyes ezeken a hatókörökön igénybe vehet. Ha például a követelmény, csak a munkaterhelésnek, és nincs más kapcsolódó erőforrásokhoz való hozzáférés kezelésére jogosult számítási feladat tulajdonosa. Ha hozzárendelése volt a [beépített tulajdonosi szerepkör] [ rbac-built-in-owner] az előfizetések szintjén minden egyes számítási feladatok felelőse lenne hozzáférésük felügyeleti minden számítási feladatokhoz.

Vessünk egy pillantást a két példa engedély modell egy kicsit jobban megérteni a fogalom. Az első példában a modell csak a szolgáltatás rendszergazdája számára erőforráscsoportok létrehozása megbízik. A második példában a modell hozzárendeli a beépített tulajdonosi szerepkör minden egyes számítási feladatok felelőse az előfizetések szintjén. 

Mindkét példa a hozzárendelt előfizetés szolgáltatás-rendszergazda van a [beépített tulajdonosi szerepkör] [ rbac-built-in-owner] az előfizetések szintjén. Emlékeztetnek arra, hogy a [beépített tulajdonosi szerepkör] [ rbac-built-in-owner] erőforrásokhoz való hozzáférés kezelésével, beleértve az összes engedélyt.
![előfizetés tulajdonosi szerepkörrel rendelkező szolgáltatás-rendszergazda](../_images/governance-2-1.png)
*3. ábra. Egy előfizetés szolgáltatás-rendszergazda szerepkörrel a beépített tulajdonosa.* 

1. Az első példában van **A számítási feladatok felelőse** nincs engedélyekkel az előfizetések szintjén - azok nem jogosult erőforrás access management alapértelmezés szerint. Ez a felhasználó szeretné a saját számítási erőforrások telepítéséhez és kezeléséhez. Kapcsolatba kell lépnie a **szolgáltatás-rendszergazda** kérelem létrehozásához az erőforráscsoport.
![erőforrás-csoportba tartozó számítási feladatok tulajdonosának kérelmek létrehozása](../_images/governance-2-2.png)  

2. A **szolgáltatás-rendszergazda** áttekinti a kérelmet, és létrehoz **erőforrás-csoportba tartozó**. Ezen a ponton **A számítási feladatok felelőse** még nem rendelkezik engedéllyel semmilyen teendője nincs.
![szolgáltatás-rendszergazdai erőforrás-csoportba tartozó hoz létre.](../_images/governance-2-3.png)

3. A **szolgáltatás-rendszergazda** hozzáadja **A számítási feladatok felelőse** való **erőforrás-csoportba tartozó** és hozzárendeli a [közreműködő beépített szerepkört](/azure/role-based-access-control/built-in-roles#contributor). A közreműködő szerepkört az összes engedélyt megadja a **erőforrás-csoportba tartozó** kivételével kezelése engedély.
![szolgáltatás-rendszergazda ad hozzá a számítási feladatok felelőse egy erőforráscsoport, egy](../_images/governance-2-4.png)

4. Tegyük fel, amelyek **A számítási feladatok felelőse** megtekintéséhez a monitorozási adatok a számítási feladatok kapacitástervezése részeként Processzor és a hálózati forgalmat a csapattagok két követelmény rendelkezik. Mivel **A számítási feladatok felelőse** van a közreműködői szerepkörrel, nincs engedélye a felhasználót **egy erőforráscsoport**. Ezek a kérést, el kell küldenie a **szolgáltatás-rendszergazda**.
![számítási feladatok tulajdonos kérések munkaterhelési közreműködők erőforráscsoport lehet hozzáadni.](../_images/governance-2-5.png)

5. A **szolgáltatás-rendszergazda** felülvizsgálja a kérést, és hozzáadja a két **munkaterhelés közreműködői** felhasználók **erőforrás-csoportba tartozó**. A két felhasználó egyike sem szükséges kezelheti az erőforrásokat, így kap engedélyt a [beépített olvasó szerepkör](/azure/role-based-access-control/built-in-roles#contributor). 
![szolgáltatás-rendszergazdai erőforrás-csoportba tartozó ad hozzá munkaterhelés közreműködők](../_images/governance-2-6.png)

6. Ezután **számítási feladatok felelőse B** is szükséges a munkaterhelésnek erőforrásokat tartalmazó erőforráscsoport. A **A számítási feladatok felelőse**, **számítási feladatok felelőse B** kezdetben nem rendelkezik engedéllyel semmilyen művelet végrehajtására az előfizetések szintjén, így azok kérelmet kell küldenie a **szolgáltatás-rendszergazda**. 
![számítási feladatok felelőse B kérelmek B erőforráscsoport létrehozása](../_images/governance-2-7.png)

7. A **szolgáltatás-rendszergazda** felülvizsgálja a kérést, és létrehoz **erőforrás – B csoport**. ![Szolgáltatás-rendszergazda létrehoz erőforrás – B csoport](../_images/governance-2-8.png)

8. A **szolgáltatás-rendszergazda** majd hozzáadja **számítási feladatok felelőse B** való **erőforrás – B csoport** és hozzárendeli a [közreműködő beépített szerepkört](https://docs.microsoft.com/en-us/azure/role-based-access-control/built-in-roles#contributor). 
![Szolgáltatás-rendszergazda számítási feladatok tulajdonosának B hozzá erőforrás – B csoport](../_images/governance-2-9.png)

Ezen a ponton a számítási feladatok tulajdonosai mindegyike van különítve a saját erőforráscsoport. Bármely más erőforráscsoport-erőforrásokhoz való hozzáférés kezelése egyetlen sem a számítási feladatok tulajdonosai vagy a csoport tagjai rendelkezik. 

![Az erőforráscsoportok, A és B előfizetést](../_images/governance-2-10.png)
*4. ábra. A két számítási feladatok saját erőforráscsoport elkülönített előfizetés.*

Ebben a modellben a minimális jogosultság modell – az minden felhasználóhoz hozzá van rendelve a megfelelő jogosultságot a helyes erőforrás-kezelés hatókörben.

Ugyanakkor vegye figyelembe, hogy ebben a példában minden egyes feladathoz hajtotta végre a **szolgáltatás-rendszergazda**. Bár ez egy egyszerű példa, és előfordulhat, hogy nem jelennek meg, mert csak két számítási feladatok tulajdonosai volt probléma, könnyebbé vált az imagine a problémákat, amelyek egy nagy szervezet eredményezne típusú. Ha például a **szolgáltatás-rendszergazda** betartásával késleltetésekhez kérések a szűk keresztmetszetté válhat.  

Vessünk egy pillantást a második példa, amely csökkenti a által végzett tevékenységek számát a **szolgáltatás-rendszergazda**. 

1. Ebben a modellben **A számítási feladatok felelőse** hozzá van rendelve a [beépített tulajdonosi szerepkör] [ rbac-built-in-owner] az előfizetések szintjén, lehetővé téve a saját erőforráscsoport létrehozásához:  **erőforrás-csoportba tartozó**. ![Szolgáltatás-rendszergazda előfizetést hozzáadja A számítási feladatok felelőse](../_images/governance-2-11.png)

2. Amikor **erőforrás-csoportba tartozó** jön létre, **A számítási feladatok felelőse** alapértelmezés szerint kerül, és örökli a [beépített tulajdonosa] [ rbac-built-in-owner] a szerepkör a egy előfizetésre.
![Számítási feladatok felelőse egy erőforrás-csoportba tartozó hoz létre.](../_images/governance-2-12.png)

3. A [beépített tulajdonosi szerepkör] [ rbac-built-in-owner] biztosít **A számítási feladatok felelőse** az erőforráscsoport való hozzáférés kezelése engedély. **A számítási feladatok felelőse** hozzáadja két **munkaterhelés közreműködők** és hozzárendeli a [beépített olvasó szerepkör] [ rbac-built-in-owner] hozzájuk. 
![A számítási feladatok felelőse hozzáadja a számítási feladatok közreműködők](../_images/governance-2-13.png)

4. **Szolgáltatás-rendszergazda** már **számítási feladatok felelőse B** a beépített tulajdonosi szerepkörrel rendelkező az előfizetéshez. 
![Szolgáltatás-rendszergazda hozzáadja a számítási feladatok tulajdonosai B előfizetéshez](../_images/governance-2-14.png)

5. **Számítási feladatok felelőse B** hoz létre **erőforrás – B csoport** , és alapértelmezés szerint meg van adva. Ismét **számítási feladatok felelőse B** a beépített tulajdonosi szerepkör örökli az előfizetési hatókört.
![Számítási feladatok tulajdonosának B hoz létre erőforrás – B csoport](../_images/governance-2-15.png)

Vegye figyelembe, hogy ebben a modellben a **szolgáltatás-rendszergazda** hajtotta végre, mint az az első példában a delegálás felügyeleti hozzáférés miatt minden egyes számítási feladatok tulajdonosai kevesebb műveletet.

![Az erőforráscsoportok, A és B előfizetést](../_images/governance-2-16.png)
*5. ábra. A szolgáltatás-rendszergazdák és a két számítási feladatok tulajdonosai egy előfizetésben, az összes hozzárendelt a beépített tulajdonosi szerepkör.*

Azonban mivel mindkettő **A számítási feladatok felelőse** és **számítási feladatok felelőse B** a beépített tulajdonosi szerepkörhöz vannak hozzárendelve az előfizetések szintjén, minden egyes örökölt vannak a beépített tulajdonosi szerepkör a többi összes erőforrás a csoport. Ez azt jelenti, hogy nem csak rendelkezik egy másik erőforrásokhoz való teljes hozzáférés, is képesek egymás erőforráscsoportok felügyeleti hozzáférés delegálásához. Például **számítási feladatok felelőse B** rendelkezik semmilyen más felhasználó való hozzáadására vonatkozó jogok **erőforrás-csoportba tartozó** és minden olyan szerepkört rendelhet hozzájuk, beleértve a beépített tulajdonosi szerepkör.

Ha összehasonlítva minden példánál a követelményeknek, láthatjuk, hogy mindkét példa támogatja-e a megbízható egyetlen felhasználó erőforrás hozzáférési jogosultságokat biztosíthat a két számítási feladatok tulajdonosai számára engedélyezett előfizetési hatókörben. Nincs erőforrás-felügyelet elérésének alapértelmezés szerint egyes a két számítási feladatok tulajdonosai, és ha szükséges a **szolgáltatás-rendszergazda** explicit módon engedélyek hozzárendelése őket. Csak az első példában azonban a követelmény, hogy minden számítási feladathoz társított erőforrások elkülönülnek egymástól, hogy nincs számítási feladatok felelőse minden egyéb munkaterhelés-erőforrásokhoz való hozzáférés nem támogatja.

## <a name="resource-management-model"></a>Erőforrás-kezelő modellje

Most, hogy alakítottuk ki a legalacsonyabb jogosultsági engedélyek modell, továbbvezet vessen egy pillantást az alábbi cégirányítási modellek gyakorlati alkalmazásmódjai. A követelmények, hogy támogatnia kell a következő három környezetekben a visszahívás:
1. **Megosztott infrastruktúra:** minden számítási feladatok által megosztott erőforrásokat egy vagy több csoport. Ezek olyan erőforrások, például hálózati átjárók, tűzfalak és biztonsági szolgáltatásokat.  
2. **Fejlesztés:** több csoportot az erőforrások több, nem éles számítási feladatok kész. Ezek az erőforrások proof-of-concept, tesztelése és egyéb fejlesztői tevékenységek szolgálnak. Előfordulhat, hogy ezeket az erőforrásokat egy több Könnyített cégirányítási modell megnövekedett fejlesztőknek nyújtott rugalmasság engedélyezéséhez.
3. **Éles:** erőforrások több éles számítási feladatok több csoportjait. Ezeket az erőforrásokat a nyilvános és internetkapcsolattal rendelkező alkalmazás-összetevők tárolására szolgálnak. Ezeket az erőforrásokat általában a legfontosabb cégirányítási és biztonsági modellek az erőforrások, az alkalmazás kódjának és az adatok védelme a jogosulatlan hozzáférés rendelkezik.

Minden egyes ezekben a környezetekben, nyomon követheti a költségadatok követelmény rendelkezünk **számítási feladatok felelőse**, **környezet**, vagy mindkettőt. Azt jelenti, szeretnénk tudni, hogy a folyamatban lévő költségét a **megosztott infrastruktúra**, egyaránt a magánszemélyek felmerülő költségeket az **fejlesztési** és **éles** környezetekben, és végül pedig a teljes költségének **fejlesztési** és **éles**. 

Már megtanulta, hogy két szinten erőforrások hatóköre: **előfizetés** és **erőforráscsoport**. Az első döntési ezért környezetekben a rendezése **előfizetés**. Csak két lehetőség közül választhat: egyetlen előfizetéssel, vagy több előfizetés is. 

Mielőtt megnézzük a példák az egyes modellek, tekintse át a felügyeleti struktúra az előfizetések az Azure-ban. 

A követelmények, hogy van egy egyéni előfizetések felelős a szervezetben, és ez a felhasználó tulajdonában lévő a visszahívás a **előfizetés tulajdonosa** fiók az Azure AD-bérlőben. Azonban ennek a fióknak nincs engedélye hozhatnak létre előfizetéseket. Csak a **Azure-fiók tulajdonosa** van engedélye ehhez: ![](../_images/governance-3-0b.png) 
 *6. ábra. Az Azure-fióktulajdonos egy előfizetést hoz létre.*

Az előfizetés létrehozása után a **Azure-fiók tulajdonosa** adhat hozzá a **előfizetés tulajdonosa** az előfizetéshez, a fiók a **tulajdonosa** szerepkör:

![](../_images/governance-3-0c.png)
*7. ábra. Az Azure-fiók tulajdonosa hozzáadja a **előfizetés tulajdonosa** felhasználói fiók az előfizetéshez a **tulajdonosa** szerepkör.*

A **előfizetés tulajdonosa** most már létrehozhat **erőforráscsoportok** és delegálása erőforráshozzáférés-kezelés.

Első vegyünk egy példát erőforrás felügyeleti modell használatával egyetlen előfizetéssel. Az első döntés, hogyan lehet a három környezetekbe erőforráscsoportok igazítása. Két lehetőség van:
1. Minden környezet egyetlen igazítása Az összes megosztott infrastruktúra-erőforrások telepítése egyetlen **megosztott infrastruktúra** erőforráscsoportot. Egy telepített fejlesztési feladatokhoz kapcsolódó összes erőforrás **fejlesztési** erőforráscsoportot. Az éles számítási feladatokhoz kapcsolódó összes erőforrás a rendszer üzembe helyezi egy **éles** erőforráscsoportot a **éles** környezetben. 
2. Hozzon létre külön erőforrás-csoportok az egyes munkaterhelésekhez tartozó elnevezési és címkék használatával való megfelelés érdekében az erőforráscsoportok három környezeteket.  

Az első lehetőség kiértékelésével lássunk is hozzá. Fogjuk használni az engedélyezési modellt, amely már volt szó az előző szakaszban az erőforráscsoportok hoz létre, és hozzáadja a felhasználókat számukra a beépített vagy egy előfizetés szolgáltatás-rendszergazda **közreműködői** vagy **olvasó** szerepkör. 

1. Az első erőforráscsoport üzembe helyezett jelöli a **megosztott infrastruktúra** környezetben. A **előfizetés tulajdonosa** hoz létre egy erőforráscsoportot a megosztott infrastruktúra-erőforrások nevű **netops megosztott rg**. 
![](../_images/governance-3-0d.png)
2. A **előfizetés tulajdonosa** hozzáadja a **hálózati műveletek felhasználó** az erőforráscsoportot és engedményeseire fiókot a **közreműködői** szerepkör. 
![](../_images/governance-3-0e.png)
3. A **hálózati műveletek felhasználó** létrehoz egy [VPN-átjáró](/azure/vpn-gateway/vpn-gateway-about-vpngateways) és konfigurálja, hogy a helyszíni VPN-berendezéshez való csatlakozáshoz. A **hálózati műveletek** felhasználói is vonatkozik egy pár [címkék](/azure/azure-resource-manager/resource-group-using-tags) az egyes erőforrások: *környezet: megosztott* és *managedBy:netOps* . Ha a **előfizetés szolgáltatás-rendszergazda** exportálások költség jelentés, a költségek lesz igazítva, az egyes ezekkel a címkékkel. Ez lehetővé teszi a **előfizetés szolgáltatás-rendszergazda** forgáspont költségek használatával a *környezet* címke és a *managedBy* címke. Figyelje meg a **erőforráskorlátok** Ez a számláló az ábra jobb felső sarkában található. Minden egyes Azure-előfizetés [szolgáltatási korlátozásaival](/azure/azure-subscription-service-limits), és ezek a korlátok hatásának megértéséhez szívesen válaszolunk minden egyes előfizetés esetén a virtuális hálózatok korlátozását. Egy alapértelmezett korlátja előfizetésenként 50 virtuális hálózatot, és az első virtuális hálózat üzembe helyezése után jelenleg 49 érhető el.
![](../_images/governance-3-1.png)
4. Két további erőforráscsoport van telepítve, az első nevű *éles-rg*. Ez az erőforráscsoport igazodik a **éles** környezetben. A második nevű *dev-rg* és igazodik a **fejlesztési** környezetben. Éles számítási feladathoz tartozó összes erőforrást a rendszer telepíti a **éles** környezet és a fejlesztési feladatokhoz kapcsolódó összes erőforrás telepítve vannak a **fejlesztési** környezet . Ebben a példában helyezünk csak üzembe két számítási feladatokat az egyes ezen két környezet, hogy minden olyan Azure-előfizetési szolgáltatási korlátok nem észlel. Fontos azonban, vegye figyelembe, hogy minden erőforráscsoport rendelkezik-e a 800 erőforrásokat az adott erőforráscsoport esetében a korlát. Ezért ha azt megőrizni az egyes erőforráscsoportokhoz számítási feladatok hozzáadása, lehetséges, hogy elérhető-e ezt a korlátot. 
![](../_images/governance-3-2.png)
5. Az első **számítási feladatok felelőse** kérelmet küld a **előfizetés szolgáltatás-rendszergazda** , és minden meg van adva a **fejlesztési** és **éles** környezeti erőforrás csoportok a **közreműködői** szerepkör. Amint már tudja, a **közreműködői** szerepkör lehetővé teszi, hogy a felhasználó minden olyan szerepkört rendel egy másik felhasználó eltérő művelet végrehajtásához. Az első **számítási feladatok felelőse** most már létrehozhat a munkaterhelésnek társított erőforrások.
![](../_images/governance-3-3.png)
6. Az első **számítási feladatok felelőse** hoz létre a két erőforrás-csoportok az egyes virtuális gépek két virtuális hálózat. Az első **számítási feladatok felelőse** vonatkozik a *környezet* és *managedBy* címkéket, hogy az összes erőforrást. Vegye figyelembe, hogy az Azure-szolgáltatás korlát számláló mostantól: 47, a fennmaradó virtuális hálózatok.
![](../_images/governance-3-4.png)
7. A virtuális hálózatok mindegyike nem rendelkezik kapcsolattal a helyszíni azok létrehozásakor. Az ilyen típusú architektúra minden virtuális hálózatnak kell létesíthető a *központ – vnet* a a **megosztott infrastruktúra** környezetben. Virtuális hálózati társviszony két külön virtuális hálózatok közötti kapcsolatot hoz létre, és lehetővé teszi, hogy a hálózati forgalom áramlását közöttük. Vegye figyelembe, hogy virtuális hálózatok közötti társviszony létesítése nem tranzitív természetüknél fogva. A társviszony-létesítés meg kell adni a csatlakoztatott két virtuális hálózat minden egyes, és ha csak a virtuális hálózatok egyik Megadja, hogy a társviszonyt a kapcsolat nem fejeződött be. Mutatja be, ennek következménye az első **számítási feladatok felelőse** megadja a közötti társviszony **éles virtuális hálózatok közötti** és **központ – vnet**. Az első társviszony jön létre, de nincs forgalmat, mert a kiegészítő társviszonyt **központ – vnet** való **éles virtuális hálózatok közötti** még nem lett megadva. Az első **számítási feladatok felelőse** névjegyeket a **hálózati műveletek** felhasználó és a kérelmek ezen kiegészítő társviszony-létesítési kapcsolat.
![](../_images/governance-3-5.png)
8. A **hálózati műveletek** felhasználó ellenőrzi a kérés, hagyja, majd adja meg a társviszony beállításai a **központ – vnet**. A társviszony-létesítési kapcsolat most már befejeződött, és a hálózati forgalmat a két virtuális hálózat között.
![](../_images/governance-3-6.png)
9. Ezután egy második **számítási feladatok felelőse** kérelmet küld a **előfizetés szolgáltatás-rendszergazda** és adnak hozzá a meglévő **éles** és **fejlesztés**  környezeti erőforrás csoportok a **közreműködői** szerepkör. A második **számítási feladatok felelőse** ugyanazokkal az engedélyekkel rendelkezik az összes erőforrás az első **számítási feladatok felelőse** az egyes erőforráscsoportokban. 
![](../_images/governance-3-7.png)
10. A második **számítási feladatok felelőse** alhálózatot hoz létre a a **éles virtuális hálózatok közötti** virtuális hálózatot, majd hozzáadja a két virtuális gépet. A második **számítási feladatok felelőse** vonatkozik a *környezet* és *managedBy* címkéket az egyes erőforrásokhoz.
![](../_images/governance-3-8.png) 

Ez a példa felügyeleti erőforrásmodell lehetővé teszi számunkra, hogy a három kötelező környezetekben-erőforrások kezeléséhez. A megosztott infrastruktúra-erőforrások védettek, mert csak egyetlen felhasználó engedélyekkel ezen erőforrások eléréséhez az előfizetésben. A számítási feladatok tulajdonosai mindegyike használhatja a megosztás infrastruktúra-erőforrások a tényleges megosztott erőforrásokat magukat az engedélyek nélkül. Azonban ez a felügyeleti modell sikertelen lesz a számítási feladatok elkülönítése – minden, a két követelmény **számítási feladatok** érhessék el az erőforrásokat, a másik munkaterhelés. 

Van egy másik fontos szempont az ebben a modellben, előfordulhat, hogy azonnal nem válik egyértelművé. A példában a rendszer **app1 számítási feladatok felelőse** , amely a társviszony-kapcsolatot a kért a **központ – vnet** a helyszíni kapcsolat biztosítására. A **hálózati műveletek** felhasználói kérelmet az adott számítási feladatok üzembe helyezett erőforrások alapján értékeli ki. Ha a **előfizetés tulajdonosa** hozzáadott **app2 számítási feladatok felelőse** együtt a **közreműködői** szerepkör, hogy a felhasználó kellett a felügyeleti hozzáférési jogosultsága ahhoz, hogy az összes erőforrás a  **termékkatalógus-rg** erőforráscsoportot. 

![](../_images/governance-3-10.png)

Ez azt jelenti, hogy **app2 számítási feladatok felelőse** saját alhálózaton lévő virtuális gépek üzembe helyezéséhez engedély volna a **éles virtuális hálózatok közötti** virtuális hálózatot. Alapértelmezés szerint azon virtuális gépek most már elérhetők a helyszíni hálózathoz. A **hálózati műveletek** felhasználó még nem ismeri az ezeken a gépeken, és nem hagyta jóvá a csatlakozásukat a helyszínen.

Ezután nézzük meg a különböző környezetekben és számítási feladatok több erőforrások csoportjainak egyetlen előfizetéssel. Vegye figyelembe, hogy az előző példában szereplő, az erőforrásokat az egyes környezetekhez is könnyen azonosítható, mert ugyanazt az erőforráscsoportot. Most, hogy a csoporton már nincs szükségünk kell egy erőforrás csoport elnevezési konvenciót funkció támaszkodnak. 

1. A **megosztott infrastruktúra** változatlan marad, így erőforrások továbbra is fennáll egy külön erőforráscsoportot ebben a modellben. Minden számítási feladathoz szükséges két erőforráscsoport - mindegyikéhez a **fejlesztési** és **éles** környezetekben. Az első számítási feladathoz a **előfizetés tulajdonosa** két erőforráscsoport hoz létre. Az első nevű **app1-éles-rg** és a másodikat nevű **app1-dev-rg**. Ahogy arra már korábban, ezt az elnevezési konvenciót azonosítja az erőforrásokat, mint az első számítási feladatok, a társítás **app1**, és a **fejlesztési** vagy **prod** környezetben. Ismét a *előfizetés* hozzáad a tulajdonosának **app1 számítási feladatok felelőse** az erőforráscsoportot, a **közreműködői** szerepkör.
![](../_images/governance-3-12.png)
2. Az első példához hasonló **app1 számítási feladatok felelőse** helyez üzembe egy nevű virtuális hálózatot **app1-éles-vnet** , a **éles** környezet és a egy másik nevesített **app1-dev-vnet** , a **fejlesztési** környezetben. Ismét **app1 számítási feladatok felelőse** kérelmet küld a **hálózati műveletek** felhasználói társviszony-létesítési kapcsolat létrehozásához. Vegye figyelembe, hogy **app1 számítási feladatok felelőse** ad hozzá ugyanazt a címkét, az első példa, valamint a korlát számláló lett csökken az előfizetés fennmaradó 47 virtuális hálózatokhoz.
![](../_images/governance-3-13.png)
3. A **előfizetés tulajdonosa** ekkor létrehozza a két erőforráscsoport **app2 számítási feladatok felelőse**. Az azonos konvenciókat megegyezik a következő **app1 számítási feladatok felelőse**, az erőforráscsoportok nevesített **app2-éles-rg** és **app2-dev-rg**. A **előfizetés tulajdonosa** hozzáadja **app2 számítási feladatok felelőse** egyes tartalmazó erőforráscsoportokat a **közreműködői** szerepkör.
![](../_images/governance-3-14.png)
4. *Számítási feladatok felelőse App2* virtuális hálózatok és virtuális gépek üzembe helyezi az erőforráscsoportok ugyanazokat az elnevezési konvenciókat. Címkék hozzáadása és a korlát számláló lett megemlíteni, 45, a fennmaradó virtuális hálózatokhoz a *előfizetés*.
![](../_images/governance-3-15.png)
5. *App2 számítási feladatok felelőse* kérelmet küld a *hálózati műveletek* felhasználói között is létesíthet a *app2-éles-vnet* az a *központ – vnet*. A *hálózati műveletek* felhasználó hoz létre a társviszony-létesítési kapcsolat.
![](../_images/governance-3-16.png)

Az eredményül kapott felügyeleti modell számos fontos különbség az első példában hasonlít:
* A két számítási mindegyike el különítve, számítási feladatok és a környezetben.
* Ez a modell, mint az első példa modell két további virtuális hálózat szükséges. Bár ez nem fontos különbség a két számítási feladatokkal, a számítási feladatok ehhez a modellhez száma elméleti korlátja: 24. 
* Erőforrások már nem az egyes környezetekhez egyetlen erőforráscsoportban vannak csoportosítva. Erőforrások csoportosítása az egyes környezetekben használt elnevezési konvencióinak ismeretét igényli. 
* A virtuális Társhálózat-kapcsolatokat mindegyike volt és által hagyja jóvá a *hálózati műveletek* felhasználói.

Most lássuk a resource management-modellben több előfizetést. Ebben a modellben, azt fogja igazítása külön előfizetésre három környezeteket: egy **megosztott szolgáltatások** -előfizetéssel, **éles** előfizetését, és végül egy **fejlesztési** előfizetés. Ez a modell szempontjai használó egyetlen előfizetéssel, hogy rendelkezünk, annak eldöntése, hogyan igazítása a számítási feladatok erőforráscsoportok modell hasonlóak. Korábban már kiderült, hogy az egyes munkaterhelésekhez tartozó erőforráscsoport létrehozása eleget tesz a számítási feladatok elkülönítése követelmény, így az ebben a példában a modell en mutatjuk.

1. Ebben a modellben háromféle *előfizetések*: *megosztott infrastruktúra*, *éles*, és *fejlesztési*. Az alábbi három előfizetés szükséges egy *előfizetés tulajdonosa*, és az egyszerű példában mindhárom fogjuk használni ugyanazzal a fiókkal. A *megosztott infrastruktúra* erőforrásokat a fenti az első két példa hasonlóképpen felügyelt, és az első számítási feladat társítva van a *app1-rg* a a *éles* környezet és az azonos nevű erőforrás csoportosíthatja a *fejlesztési* környezetben. A *app1 számítási feladatok felelőse* minden az erőforráscsoportot a *közreműködői* szerepkör. 
![](../_images/governance-3-17.png)
2. Csakúgy, mint a korábbi példákban *app1 számítási feladatok felelőse* létrehozza az erőforrásokat, és a társviszony-kapcsolatot igényel a *megosztott infrastruktúra* virtuális hálózatot. *Az App1 számítási feladatok felelőse* csak hozzáadja a *managedBy* címkét, mert már nem szükséges a *környezet* címke. Azt jelenti, erőforrások vannak az egyes környezetek csoportba kerülnek ugyanazon *előfizetés* és a *környezet* címke már redundáns. A korlát számláló értéke megemlíteni fennmaradó 49 virtuális hálózatokhoz.
![](../_images/governance-3-18.png)
3. Végül a *előfizetés tulajdonosa* megismétli a folyamatot a második terheléshez, az erőforráscsoportok hozzáadásával a *app2 számítási feladatok felelőse* az a * közreműködői szerepkört. Minden környezet előfizetések a korlát számláló megemlíteni fennmaradó 48 virtuális hálózatokhoz. 

Ez a felügyeleti modell rendelkezik a fenti második példa előnyeit. Azonban a legfontosabb különbség, hogy korlátok kisebbek a probléma oka, hogy az férgek több mint két *előfizetések*. A hátránya, hogy a címkék által nyomon követett költségadatok összesíteni kell között mindhárom *előfizetések*. 

Ezért prioritását a követelményektől függően ezen két példa erőforrás felügyeleti modellek bármelyikét kiválaszthatja. Ha várhatóan, hogy a szervezet nem fogják tudni elérni a szolgáltatásra vonatkozó korlátozások szolgáltatást egy előfizetéshez, a több erőforráscsoportokkal használhatja egyetlen előfizetéssel. Ezzel szemben, ha a szervezet helyesen számos számítási feladatokat, az egyes környezetekhez több előfizetést jobb megoldás lehet.

## <a name="implementing-the-resource-management-model"></a>A resource management-modell megvalósítása

Hogy megismerkedett az Azure-erőforrásokhoz való hozzáférést szabályozó több különböző modell. Most végigvezetjük a mindegyike egy előfizetéssel rendelkező felügyeleti erőforrásmodell megvalósításához szükséges lépéseket a **megosztott infrastruktúra**, **éles**, és **fejlesztési** tervezési útmutató a környezeteket. Beállítjuk, hogy az egyik **előfizetés tulajdonosa** három környezetekhez. Minden számítási feladat fog különíthető el a egy **erőforráscsoport** együtt egy **számítási feladatok felelőse** szolgáltatással együtt a **közreműködői** szerepkör.

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

## <a name="next-steps"></a>További lépések
> [!div class="nextstepaction"]
> [Hogyan helyezhető üzembe egy alapvető infrastruktúrával](../infrastructure/basic-workload.md)

<!-- links -->
[understand-resource-access-in-azure]: /azure/role-based-access-control/rbac-and-directory-admin-roles

<!-- links -->

[rbac-built-in-owner]: /azure/role-based-access-control/built-in-roles#owner
[rbac-built-in-roles]: /azure/role-based-access-control/built-in-roles
