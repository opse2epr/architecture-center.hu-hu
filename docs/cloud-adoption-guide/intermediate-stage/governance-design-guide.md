---
title: 'Cégirányítási kialakítási Útmutató: az Azure-ban több csapat az új fejlesztési'
description: Útmutató a Azure irányítás vezérlők több csapat, több munkaterhelés és több környezet konfigurálása
author: petertay
ms.openlocfilehash: 05f1f9bb24af4f4da55b15c1aca2c71bc0b65411
ms.sourcegitcommit: b3d74d8a89b2224fc796ce0e89cea447af43a0d4
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 06/11/2018
ms.locfileid: "35290722"
---
# <a name="governance-design-guide-new-development-in-azure-for-multiple-teams"></a>Cégirányítási kialakítási Útmutató: az Azure-ban több csapat az új fejlesztési

Ez az útmutató célja, ismerje meg, a folyamat az Azure-erőforrás irányítás modell tervezéséhez nyújt segítséget.  Azt fogja nézze meg adott elméleti cégirányítási követelményeket, majd a halad át néhány példa megvalósításokhoz, amelyek ezen igények kielégítéséhez. 

A követelmények a következők:
* Az Identitáskezelés több csoportokat, az Azure-ban a különböző erőforrás-hozzáférési követelmények. Az identity management rendszer tárolja a következő felhasználók identitásának:
  1. A szervezet tulajdonjogát felelős az egyes **előfizetések**.
  2. Az egyes felelős a szervezet a **megosztott infrastruktúra erőforrások** a helyszíni hálózat egy Azure virtuális hálózathoz való kapcsolódáshoz használt. 
  3. Két egyéni felhasználók számára a szervezetében kezeléséért egy **munkaterhelés**. 
* Több **környezetek**. Korábban megtanulta, mivel egy olyan környezetben a hasonló felügyeleti és biztonsági követelmények erőforrások logikai csoportosítása. Három környezetekben szükséges:
  1. A **megosztott hálózatiinfrastruktúra-környezeten** , amely tartalmazza az erőforrások más környezetekben munkaterhelés osztozik. Például egy virtuális hálózati átjáró alhálózattal, amely kapcsolatot biztosít a helyszínen.
  2. A **éles környezetben** a leginkább korlátozó biztonsági házirendek. Belső vagy külső irányuló munkaterhelések is tartalmazhat.
  3. A **fejlesztőkörnyezet** a koncepció igazolása és tesztelési munka. Ebben a környezetben van biztonsági, megfelelőségi és költség házirendek, amelyek eltérnek a termelési környezetben.
* A **legalacsonyabb jogosultsági szint engedélymodelljét** mely felhasználók engedélye nincs alapértelmezés szerint. A modell a következőket kell támogatnia:
  * Erőforrás-hozzáférési engedélyek hozzárendelése engedéllyel rendelkező előfizetés hatókörben egy megbízható felhasználó.
  * Minden számítási feladatok felelőse alapértelmezés szerint erőforrásokhoz való hozzáférés megtagadva. Erőforrás hozzáférési jogok explicit módon a egyetlen megbízható felhasználó azon az előfizetési hatókört.
  * A kezelési hozzáférés a megosztott infrastruktúra erőforrásokhoz, a megosztott infrastruktúra tulajdonosa korlátozódik.  
  * A kezelési hozzáférés korlátozva van a számítási feladatok felelőse minden munkaterhelés számára.
  * A használatára [beépített RBAC-szerepkörök] [ rbac-built-in-roles] csak. Nem egyedi az RBAC-szerepkörök.
* Költségek nyomon követése munkaterhelés tulajdonos neve vagy környezetben. 

## <a name="identity-management"></a>Identitáskezelés

Identitáskezelés irányítás tekinthetők azt kialakítani, mielőtt fontos tudni, hogy a négy főbb területet, a szerepkör magában:

* Felügyelete: a folyamatokat és eszközöket a létrehozása, szerkesztése és törlése a felhasználó identitása.
* Hitelesítési: felhasználó hitelesítő adatait, például egy felhasználónév és jelszó érvényesítésével ellenőrzése
* Engedélyezési: erőforrások meghatározása hitelesített felhasználók számára engedélyezett hozzáférési és/vagy milyen műveletek elvégzésére jogosultságot.
* Naplózás: rendszeres időközönként a naplók és a biztonsági problémák felderítéséhez adatok megtekintésével kapcsolatos felhasználói azonosító. Ez magában foglalja, gyanús használati szokásokról, rendszeresen tekintse át a felhasználói engedélyek ellenőrzéséhez, hogy pontosak, és egyéb funkcióinak áttekintése.

Az Azure-identitás megbízható csak egy szolgáltatás, és amely Azure Active Directory (AD). A Microsoft felhasználók hozzáadása az Azure AD és használja azt az összes fent felsorolt funkciók lesz. Fontos azonban, mielőtt úgy tekintünk, hogyan konfigurálását végezzük el az Azure AD, fontos tudni, hogy a kiemelt jogosultságú fiókok, amely segítségével a szolgáltatásokhoz való hozzáférés kezelése.

Ha a szervezet regisztrált az Azure-fiók esetén legalább egy Azure **fiók tulajdonosának** hozzá lett rendelve, és az Azure AD **bérlői** hozták létre, ha nincs már társított Azure AD-bérlő a egyéb Microsoft-szolgáltatásokat, például az Office 365 szervezet használatát. A **globális rendszergazda** teljes körű engedélyekkel a az Azure AD bérlő van társítva a létrehozásakor. 

Mind az Azure-fiók tulajdonosának és az Azure AD globális rendszergazda felhasználói identitások a Microsoft által felügyelt rendkívül biztonságos identitásrendszere tárolja. Az Azure-fiók tulajdonosának jogosult létrehozása, frissítése és előfizetések törlése. Az Azure AD globális rendszergazda Azure AD-ben sok műveletek végrehajtására jogosult, de a jelen tervezési útmutató azt kell összpontosítania létrehozását és törlését a felhasználói identitás. 

> [!NOTE]
> A szervezet már meglévő Azure AD-bérlő előfordulhat, hogy rendelkezik, ha egy meglévő Office 365 vagy a fiókjához társított Intune-licenc.

Az Azure-fiók tulajdonosának jogosult létrehozása, frissítése és előfizetések törlése: 

![Az Azure-fiókot kezelő és az Azure AD globális rendszergazda Azure-fiók](../_images/governance-3-0.png)
*1. ábra. Azure-fiókot Ügyfélfelelőshöz és az Azure AD globális rendszergazda.*

Az Azure AD **globális rendszergazda** jogosult felhasználói fiókokat hozhat létre:  

![Az Azure-fiókot kezelő és az Azure AD globális rendszergazda Azure-fiók](../_images/governance-3-0a.png)
*2. ábra. Az Azure AD globális rendszergazda a szükséges felhasználói fiókokat a bérlő hoz létre.*

Az első két fiókok **App1 számítási feladatok felelőse** és **App2 számítási feladatok felelőse** azok minden a szervezetében, a munkaterhelés kezeléséért egy adott társított. A **hálózati műveletek** az egyes, a megosztott infrastruktúra erőforrásainak felelős a fiók tulajdonosa. Végezetül a **előfizetés tulajdonosa** fiókkal társítva az előfizetések tulajdonjogát felelős személy.

## <a name="resource-access-permissions-model-of-least-privilege"></a>Erőforrás hozzáférés engedélymodelljét legalacsonyabb jogosultsági szint

Most, hogy az identitás felügyeleti rendszer- és felhasználói fiókok létrehozása, döntse el, hogyan fog érvénybe lépni szerepköralapú hozzáférés-vezérlést (RBAC) szerepkörök a legalacsonyabb jogosultsági szint engedélymodelljét támogatásához minden felhasználói fiókhoz kell.

Az egyes munkaterhelésekhez tartozó erőforrások el legyen különítve egymástól úgy, hogy nem egy számítási feladatok felelőse bármely más munkaterhelés nem saját felügyeleti hozzáfér a többi követelmény van. Ez kizárólag a modell megvalósításához követelmény tudunk [beépített szerepkörök az Azure RBAC][rbac-built-in-roles].

Egy Azure-ban három hatókörök RBAC szerepkörönként vonatkozik: **előfizetés**, **erőforráscsoport**, majd egy adott **erőforrás**. Szerepkörök, alacsonyabb hatókörök örökölt. Például, ha a felhasználó hozzá van rendelve a [beépített tulajdonosi szerepkör] [ rbac-built-in-owner] az előfizetési szinten, hogy a szerepkör is hozzá van rendelve az erőforráscsoportot és az egyes erőforrások szintjén felhasználó kivéve, ha azt felülbírálja.

Ezért legalább jogosultság hozzáférési kell döntse el, a műveleteket csak egy modell létrehozásához egy bizonyos típusú felhasználó engedélyezett mindegyik ezen három hatókörök érvénybe. Például a követelmény csak a munkaterhelés, és nincs más társított erőforrásokhoz való hozzáférés kezelése engedéllyel kell rendelkeznie a munkaterhelés tulajdonosa van. Ha rendelhető hozzá a rendszer a [beépített tulajdonosi szerepkör] [ rbac-built-in-owner] előfizetés hatókörből minden számítási feladatok felelőse felügyeleti hozzáférés az összes munkaterheléseknek kellene lennie.

Vessen egy pillantást a fogalom megértéséhez még jobban két példa engedély modellek. Az első példában tekinthetők megbízhatónak tekinti csak a szolgáltatás-rendszergazda erőforráscsoportokat létrehozni. A második példában az tekinthetők a beépített tulajdonosi szerepkört rendel minden számítási feladatok felelőse előfizetés hatókörben. 

Mindkét példákban hozzárendelt előfizetés szolgáltatás-rendszergazda van a [beépített tulajdonosi szerepkör] [ rbac-built-in-owner] előfizetés hatókörben. Visszahívása, amely a [beépített tulajdonosi szerepkör] [ rbac-built-in-owner] erőforrásokhoz való hozzáférés felügyeletét beleértve minden engedélyt megad a.
![tulajdonosi szerepkör rendelkező előfizetés szolgáltatás-rendszergazda](../_images/governance-2-1.png)
*3. ábra. Egy előfizetés szolgáltatás-rendszergazda a beépített tulajdonosi szerepét.* 

1. Az első példában tudunk **A számítási feladatok felelőse** rendelkező nincs engedélye az előfizetési hatókört - alapértelmezés szerint rendelkeznek nincs erőforrás hozzáférési jogosultságait. Ez a felhasználó azt szeretné, telepítheti és kezelheti az erőforrásokat a terhelés. Kapcsolatba kell lépnie a **szolgáltatás-rendszergazda** kérelem létrejön egy erőforráscsoportot.
![erőforrás-csoport A munkaterhelés tulajdonos kérelmek létrehozása](../_images/governance-2-2.png)  

2. A **szolgáltatás-rendszergazda** ellenőrzi, hogy a kérelmet, és létrehoz **erőforrás csoport A**. Ezen a ponton **A számítási feladatok felelőse** még nem rendelkezik engedélyekkel ahhoz, hogy semmit.
![szolgáltatás-rendszergazda létrehoz erőforrás csoport A](../_images/governance-2-3.png)

3. A **szolgáltatás-rendszergazda** hozzáadja **A számítási feladatok felelőse** való **erőforrás csoport A** és hozzárendeli a [beépített közreműködői szerepkör](/azure/role-based-access-control/built-in-roles#contributor). A közreműködői szerepkör minden engedélyt megad a **erőforrás csoport A** kivéve kezelése engedéllyel.
![szolgáltatás-rendszergazda számítási feladatok felelőse ad hozzá egy erőforráscsoporthoz egy](../_images/governance-2-4.png)

4. Tegyük fel, amelyek **A számítási feladatok felelőse** van szükség a csoport tagjai két megtekintése a figyelési adatok az alkalmazások és szolgáltatások kapacitástervezését részeként Processzor- és hálózati forgalmat. Mivel **A számítási feladatok felelőse** van hozzárendelve a közreműködő szerepkört, nem rendelkeznek a felhasználó-hozzáadási jogosultsággal **erőforráscsoport A**. Azok a kérést kell küldenie a **szolgáltatás-rendszergazda**.
![munkaterhelés tulajdonos kérések munkaterhelési közreműködők csoportjába vehető fel erőforrás](../_images/governance-2-5.png)

5. A **szolgáltatás-rendszergazda** ellenőrzi, hogy a kérelmet, és hozzáadja a két **munkaterhelés közreműködői** felhasználók **erőforrás csoport A**. A két felhasználó egyike sem kezelheti az erőforrásokat, így hozzárendeli az engedély szükséges a [beépített olvasó szerepkört](/azure/role-based-access-control/built-in-roles#contributor). 
![szolgáltatás-rendszergazda erőforrás csoport A munkaterhelés közreműködők hozzáadása](../_images/governance-2-6.png)

6. Ezt követően **számítási feladatok felelőse B** feltételt is igényli egy erőforráscsoportot a munkaterhelés számára az erőforrások tárolásához. A **A számítási feladatok felelőse**, **számítási feladatok felelőse B** eredetileg nem rendelkezik engedéllyel semmilyen művelet végrehajtására az előfizetés hatókörben, így azok a kérést kell küldenie a **szolgáltatás-rendszergazda**. 
![számítási feladatok felelőse B kérelmek B erőforráscsoport létrehozása](../_images/governance-2-7.png)

7. A **szolgáltatás-rendszergazda** ellenőrzi, hogy a kérelmet, és létrehoz **erőforráscsoport B**. ![Szolgáltatás-rendszergazda B erőforráscsoport létrehozása](../_images/governance-2-8.png)

8. A **szolgáltatás-rendszergazda** hozzáadja **számítási feladatok felelőse B** való **erőforráscsoport B** és hozzárendeli a [beépített közreműködői szerepkör](https://docs.microsoft.com/en-us/azure/role-based-access-control/built-in-roles#contributor). 
![Szolgáltatás-rendszergazda erőforráscsoport B munkaterhelés tulajdonos B hozzáadása](../_images/governance-2-9.png)

Ezen a ponton a munkaterhelések tulajdonosai mindegyikének elkülönül a saját erőforráscsoportban. Egy másik erőforráscsoportban az erőforrásokhoz való hozzáférés a munkaterhelések tulajdonosai vagy a csoport tagjai közül egyikhez sem tartozik. 

![az A és B erőforráscsoportokhoz előfizetés](../_images/governance-2-10.png)
*4. ábra. Két, saját erőforrás csoporttal elkülönített munkaterhelések tulajdonosai rendelkező előfizetés.*

Ebben a modellben viszont egy legalacsonyabb jogosultsági modell - minden felhasználó a megfelelő engedéllyel a helyes erőforrás felügyeleti hatókörben van hozzárendelve.

Azonban vegye figyelembe, hogy minden feladat ebben a példában végzett el a **szolgáltatás-rendszergazda**. Ez egy egyszerű példa, és előfordulhat, hogy tűnik, hogy egy probléma miatt csak két munkaterhelések tulajdonosai, de napjainkban könnyen képzelhető el a problémákat, amelyek egy nagy szervezet eredményezne típusú. Például a **szolgáltatás-rendszergazda** szűk keresztmetszetet jelenthet a kérések késleltetése eredményező nagy hátralék.  

Vessen egy pillantást csökkenthető a feladatok által végrehajtott második példáját a **szolgáltatás-rendszergazda**. 

1. Ebben a modellben **A számítási feladatok felelőse** hozzá van rendelve a [beépített tulajdonosi szerepkör] [ rbac-built-in-owner] az előfizetési hatókört, lehetővé téve a saját erőforráscsoport létrehozásához:  **erőforrás-csoport A**. ![Szolgáltatás-rendszergazda hozzáadja A számítási feladatok felelőse előfizetés](../_images/governance-2-11.png)

2. Ha **erőforrás csoport A** jön létre, **A számítási feladatok felelőse** alapértelmezés szerint hozzáadva, és örököl a [beépített tulajdonos] [ rbac-built-in-owner] szerepkörnek a előfizetési hatókört.
![A számítási feladatok felelőse erőforrás csoportba tartozó hoz létre.](../_images/governance-2-12.png)

3. A [beépített tulajdonosi szerepkör] [ rbac-built-in-owner] biztosít **A számítási feladatok felelőse** az erőforráscsoport való hozzáférés kezelésére jogosult. **A számítási feladatok felelőse** hozzáadja két **munkaterhelés közreműködők** és hozzárendeli a [beépített olvasó szerepkört] [ rbac-built-in-owner] hozzájuk. 
![A számítási feladatok felelőse munkaterhelés közreműködők hozzáadása](../_images/governance-2-13.png)

4. **Szolgáltatás-rendszergazda** most hozzáadja **számítási feladatok felelőse B** a beépített tulajdonosi szerepkört az előfizetés számára. 
![Szolgáltatás-rendszergazda hozzáadja a munkaterhelések tulajdonosai B előfizetéshez](../_images/governance-2-14.png)

5. **Számítási feladatok felelőse B** hoz létre **erőforráscsoport B** és alapértelmezés szerint hozzáadva. Ebben az esetben **számítási feladatok felelőse B** beépített tulajdonosi szerepkör örökli az előfizetési hatókört.
![Munkaterhelés tulajdonos B erőforráscsoport B hoz létre.](../_images/governance-2-15.png)

Vegye figyelembe, hogy ebben a modellben a **szolgáltatás-rendszergazda** végrehajtott műveletek kevesebb, mint az első delegálását a kezelési hozzáférés miatt az egyes az egyes munkaterhelések tulajdonosai.

![az A és B erőforráscsoportokhoz előfizetés](../_images/governance-2-16.png)
*5. ábra. Egy előfizetés szolgáltatás-rendszergazda és a két munkaterhelések tulajdonosai, az összes hozzárendelt a beépített tulajdonosi szerepkört.*

Azonban mivel mindkét **A számítási feladatok felelőse** és **számítási feladatok felelőse B** a beépített tulajdonos szerepkör az előfizetési hatókört, akkor minden egyes örökölt vannak egymás erőforrás beépített tulajdonosi szerepkör a csoport. Ez azt jelenti, hogy nem csak rendelkeznek teljes hozzáféréssel egy másik erőforrásokat is képesek egymás erőforráscsoportok felügyeleti hozzáférés delegálásához. Például **számítási feladatok felelőse B** rendelkezik más felhasználók hozzáadására vonatkozó jogok **erőforrás csoport A** és rendelhet szerepköröket azokat, beleértve a beépített tulajdonosi szerepkört.

Ha azt hasonlítsa össze az egyes például, hogy a követelmények, látható, hogy mindkét példák támogatja-e az előfizetési hatókört, amely jogosult erőforrás hozzáférési jogot a két munkaterhelések tulajdonosai egyetlen megbízható felhasználója. A két munkaterhelések tulajdonosai mindegyikének erőforrás-felügyelet elérésének nem rendelkezett alapértelmezés szerint, és szükséges az **szolgáltatás-rendszergazda** explicit módon engedélyek hozzárendelése őket. Azonban csak az első példában támogatja a követelmény, hogy az egyes munkaterhelésekhez tartozó erőforrások el különítve egymástól úgy, hogy nincs számítási feladatok felelőse hozzáfér minden egyéb munkaterhelés erőforrásaihoz.

## <a name="resource-management-model"></a>Felügyeleti modell

Most, hogy azt már úgy tervezték, a legalacsonyabb jogosultsági szint engedélymodelljét, most lépjen tovább vessen egy pillantást az irányítás modellek gyakorlati alkalmazásai. A követelmények, hogy a következő három környezetekben támogatott kell a visszahívás:
1. **A megosztott infrastruktúra:** egy különálló csoportot az összes munkaterhelés osztozik rajtuk erőforrások. Ezek azok az erőforrások, például a hálózati átjárók, tűzfalak és biztonsági szolgáltatásokat.  
2. **Fejlesztési:** több, nem éles képviselő erőforrások több csoport készen áll a munkaterhelések. Ezeket az erőforrásokat a koncepció igazolása, tesztelését és egyéb fejlesztői tevékenységek érvényesek. Ezeket az erőforrásokat is lehetnek több laza irányítás modell, mert ahhoz, hogy áttelepíthesse nőtt fejlesztői agilitást.
3. **Éles:** több termelési számítási feladatokhoz képviselő erőforrások több csoporthoz. A privát és nyilvános irányuló kérelem összetevők üzemeltetéséhez használt ezeket az erőforrásokat. Ezeket az erőforrásokat a legfontosabb irányítás és biztonsági modellek számára az erőforrások, az alkalmazás kódja és az adatok védelme a jogosulatlan hozzáférés általában rendelkeznek.

Az egyes ezekhez a környezetekhez, nyomon követheti költség által követelmény tudunk **számítási feladatok felelőse**, **környezet**, vagy mindkettőt. Ez azt jelenti, hogy szeretnénk tudni, hogy a folyamatban lévő költségét az **megosztott infrastruktúra**, az egyéni felhasználók számára is költségeit a **fejlesztési** és **éles** környezetekben, és végül a teljes költségének **fejlesztési** és **éles**. 

Már megtanulta, hogy két szinttel elhelyezkedő erőforrások hatóköre: **előfizetés** és **erőforráscsoport**. Az első döntési ezért rendezése a környezetek által **előfizetés**. Csak két lehetőség közül választhat: egy-egy előfizetéshez, vagy több előfizetés. 

Mielőtt úgy tekintünk, ezek a modellek egyes példák, tekintsük át, az Azure-előfizetések struktúra. 

A követelmények, hogy a szervezet, amely felelős az előfizetések egy adott van, és a felhasználó tulajdonában előhívja a **előfizetés tulajdonosa** -fiókba az Azure AD-bérlő. Azonban ennek a fióknak nincs engedélye előfizetést létrehozni. Csak a **Azure fiók tulajdonosának** jogosult-e ehhez: ![](../_images/governance-3-0b.png) 
 *6. ábra. Egy Azure-fiók tulajdonosának egy előfizetést hoz létre.*

Miután létrehozta az előfizetést, a **Azure fiók tulajdonosának** adhat hozzá a **előfizetés tulajdonosa** fiókot az előfizetés a **tulajdonos** szerepkör:

![](../_images/governance-3-0c.png)
*7. ábra. Az Azure-fiók tulajdonosának hozzáadja a **előfizetés tulajdonosa** felhasználói fiókot az előfizetés a **tulajdonos** szerepkör.*

A **előfizetés tulajdonosa** hozhatók létre **erőforráscsoportok** és delegálása erőforrás hozzáférés-kezelés.

Első egy példa felügyeleti modell segítségével egyetlen előfizetéssel vizsgáljuk meg. Az első döntési, hogyan erőforráscsoportok három környezetekhez igazítása. Jelenleg két lehetőség közül választhat:
1. Igazítás minden környezet egyetlen erőforráscsoportként működnek. Az összes megosztott infrastruktúra-erőforrások telepítése egyetlen **megosztott infrastruktúra** erőforráscsoportot. Fejlesztési munkaterhelések társított összes erőforrás egyetlen telepített **fejlesztési** erőforráscsoportot. Termelési számítási feladatokhoz tartozó összes erőforrások telepítése egyetlen **éles** tartozó erőforráscsoport a **éles** környezetben. 
2. Egy külön erőforráscsoporttal, elnevezési és címkék használatával való megfelelés érdekében a három környezetek egyes erőforráscsoportok munkaterhelések igazítása.  

Hozzuk, az első lehetőség kiértékelése. Fogjuk használni az engedélyek modelljét, amiről beszéltünk az előző szakaszban az erőforráscsoportok létrehozó, és hozzáadja a felhasználókat számukra a beépített vagy egyetlen előfizetés szolgáltatás-rendszergazda **közreműködő** vagy **olvasó** szerepkör. 

1. Az első erőforráscsoport telepített jelöli a **megosztott infrastruktúra** környezetben. A **előfizetés tulajdonosa** hoz létre egy erőforráscsoportot a megosztott infrastruktúra erőforrásainak nevű **netops megosztott rg**. 
![](../_images/governance-3-0d.png)
2. A **előfizetés tulajdonosa** hozzáadja a **hálózati műveletek felhasználói** fiókot az erőforráscsoportot és rendel a **közreműködő** szerepkör. 
![](../_images/governance-3-0e.png)
3. A **hálózati műveletek felhasználói** létrehoz egy [VPN-átjáró](/azure/vpn-gateway/vpn-gateway-about-vpngateways) és konfigurálja a helyszíni VPN-készülék való kapcsolódáshoz. A **hálózati műveletek** felhasználói is vonatkozik két [címkék](/azure/azure-resource-manager/resource-group-using-tags) az erőforrások mindegyikének: *környezet: megosztott* és *managedBy:netOps* . Ha a **előfizetés szolgáltatás-rendszergazda** export költségekkel jelentéssel költségek fog egyeztetni az összes ezekkel a címkékkel. Ez lehetővé teszi a **előfizetés szolgáltatás-rendszergazda** forgáspont költségek a használatával a *környezet* címke és a *felügyeli* címke. Figyelje meg a **erőforrás korlátok** számláló, a felső sarkában található ábra. Minden Azure-előfizetéssel rendelkezik [szolgáltatási korlátait](/azure/azure-subscription-service-limits), és megismerheti a hatását, hogy ezek a korlátozások a virtuális hálózatok korlátozását, az egyes előfizetésekhez fogjuk követni. Alapértelmezett maximális előfizetésenként 50 virtuális hálózatok, és az első virtuális hálózat telepítését követően érhetők el most 49.
![](../_images/governance-3-1.png)
4. Két további erőforráscsoportok vannak telepítve, az első nevű *termék-rg*. Ez az erőforráscsoport igazodik a **éles** környezetben. A második nevű *fejlesztői-rg* és igazodik a **fejlesztési** környezetben. Termelési számítási feladatokhoz tartozó összes erőforrást a rendszer telepíti a **éles** környezet és a fejlesztői munkaterhelések társított összes erőforrás telepít a **fejlesztési** környezet . Ebben a példában azt fogja csak két munkaterhelések telepítése minden egyes két környezetben, azt nem előforduló bármely Azure-előfizetés szolgáltatásra vonatkozó korlátozások. Azonban fontos figyelembe venni, hogy rendelkezik-e az adott erőforráscsoport 800 erőforrások legfeljebb minden erőforráscsoportban. Ezért azt tartsa csoportba való felvételekor munkaterhelések minden erőforrás esetén lehetséges, hogy elérhető-e ezt a határt. 
![](../_images/governance-3-2.png)
5. Az első **számítási feladatok felelőse** egy kérést küld a **előfizetés szolgáltatás-rendszergazda** , és az egyes ad hozzá a **fejlesztési** és **éles** környezet erőforráscsoportokat rendelkező a **közreműködő** szerepkör. Mivel korábban megjegyzett a **közreműködő** szerepkör lehetővé teszi a felhasználói szerepkört rendel egy másik felhasználó eltérő bármilyen művelet végrehajtására. Az első **számítási feladatok felelőse** hozhatók létre az alkalmazások és szolgáltatások társított erőforrásokat.
![](../_images/governance-3-3.png)
6. Az első **számítási feladatok felelőse** egy virtuális hálózatot hoz létre minden egyes virtuális gépek két két erőforráscsoport. Az első **számítási feladatok felelőse** alkalmazza a *környezet* és *felügyeli* címkék minden erőforráshoz. Vegye figyelembe, hogy az Azure-szolgáltatások korlát számláló most a hátralévő 47 virtuális hálózatok.
![](../_images/governance-3-4.png)
7. Minden virtuális hálózaton nincs kapcsolat a helyszíni, amikor létrehozzák őket. Az ilyen típusú architektúra, minden egyes virtuális hálózati kell társítottak, hogy a *hub-vnet* a a **megosztott infrastruktúra** környezetben. Virtuális hálózati társviszony-létesítés kapcsolatot hoz létre két külön virtuális hálózatok között, és lehetővé teszi, hogy a hálózati forgalom áramlását közöttük. Vegye figyelembe, hogy virtuális hálózati társviszony-létesítés nem eredendően tranzitív. Társviszony-létesítés kell megadni minden csatlakoztatott két virtuális hálózaton, és ha csak egy virtuális hálózathoz társviszony-létesítés határozza meg a kapcsolat nem fejeződött be. Mutatja be, ennek hatását az első **számítási feladatok felelőse** megadja a társviszony-létesítés közötti **termék-vnet** és **hub-vnet**. Az első társviszony-létesítés létrejött, de nincs forgalmat, mert a kiegészítő társviszony **hub-vnet** való **termék-vnet** még nincs megadva. Az első **számítási feladatok felelőse** névjegyeket a **hálózati műveletek** felhasználói és a kiegészítő kapcsolat társviszony kérelmek.
![](../_images/governance-3-5.png)
8. A **hálózati műveletek** felhasználói ellenőrzi, hogy a kérelem, hagyja, majd adja meg a társviszony-létesítés beállításai a **hub-vnet**. A társviszony-létesítési kapcsolat befejeződött, és hálózati forgalmat a két virtuális hálózatok között.
![](../_images/governance-3-6.png)
9. Most, egy második **számítási feladatok felelőse** egy kérést küld a **előfizetés szolgáltatás-rendszergazda** , és a meglévő **éles** és **fejlesztési**  környezet erőforráscsoportokat rendelkező a **közreműködő** szerepkör. A második **számítási feladatok felelőse** ugyanazokkal az engedélyekkel rendelkezik az összes erőforrás az első **számítási feladatok felelőse** minden erőforráscsoportban. 
![](../_images/governance-3-7.png)
10. A második **számítási feladatok felelőse** lévő alhálózatot hoz létre a **termék-vnet** virtuális hálózatot, majd hozzáadja a két virtuális gép. A második **számítási feladatok felelőse** alkalmazza a *környezet* és *felügyeli* címkék az egyes erőforrásokhoz.
![](../_images/governance-3-8.png) 

A példa felügyeleti modell lehetővé teszi, hogy a három kötelező környezetekben az erőforrások kezelése. A megosztott infrastruktúra erőforrásainak védettek, mivel ezen erőforrások hozzáférése az előfizetés csak egy-egy felhasználóhoz. A munkaterhelések tulajdonosai egy anélkül, hogy azokat az engedélyeket a tényleges megosztott erőforrások magukat az erőforrások megosztása infrastruktúra használatára képesek. Ez a felügyeleti modell nem sikerül a követelmény az alkalmazások és szolgáltatások elkülönítés – minden, a két **munkaterhelések tulajdonosai** férhetnek hozzá az erőforrásokat, a másik fél munkaterhelés. 

Nincs egy másik fontos szempont, előfordulhat, hogy azonnal nem válik egyértelművé ebben a modellben. A jelen példában volt **app1 számítási feladatok felelőse** a hálózati társviszony-létesítési kapcsolatot kért, amely a **hub-vnet** helyszíni kapcsolatot biztosít. A **hálózati műveletek** felhasználói értékeli ki, hogy a munkaterhelés telepített erőforrások alapuló kérelem. Ha a **előfizetés tulajdonosa** hozzáadott **app2 számítási feladatok felelőse** rendelkező a **közreműködő** szerepkör, a felhasználó rendelkezik-e felügyeleti hozzáférési jogosultsága ahhoz, hogy az összes erőforrást a  **termék-rg** erőforráscsoportot. 

![](../_images/governance-3-10.png)

Ez azt jelenti, hogy **app2 számítási feladatok felelőse** kellett engedéllyel való üzembe helyezéshez saját alhálózat virtuális gépek a **termék-vnet** virtuális hálózat. Alapértelmezés szerint azon virtuális gépek most már rendelkezik a helyszíni hálózat elérését. A **hálózati műveletek** felhasználó nem ismeri a ezeket a gépeket, és nem találta megfelelőnek a csatlakozásukat a helyszínen.

Ezt követően a különböző környezetek és a munkaterhelések több erőforrások csoportokkal egyetlen előfizetéssel vizsgáljuk meg. Vegye figyelembe, hogy az előző példában szereplő, az egyes környezet erőforrásait könnyen azonosítható volt, mert ugyanazt az erőforráscsoportot. Most, hogy már nem adott csoportosítás, igazolnia kell erőforrás csoport elnevezési funkció támaszkodnak. 

1. A **megosztott infrastruktúra** változatlan marad, így erőforrások továbbra is fennáll külön erőforráscsoport ebben a modellben. Minden számítási feladat által igényelt két erőforráscsoport - beli a **fejlesztési** és **éles** környezetekben. Az első munkaterhelés a **előfizetés tulajdonosa** hoz létre két erőforráscsoport. Az első nevű **app1-termék-rg** és a második érték nevű **app1-fejlesztői-rg**. A fentiekben taglaltak, ezt az elnevezési konvenciót azonosítja az erőforrások, mint az első munkaterhelés társított **app1**, és vagy a **fejlesztői** vagy **termék** környezetben. Ebben az esetben a *előfizetés* tulajdonos hozzáadja **app1 számítási feladatok felelőse** az erőforrás-csoport számára a **közreműködői** szerepkör.
![](../_images/governance-3-12.png)
2. Az első példához hasonló **app1 számítási feladatok felelőse** nevű virtuális hálózat telepíti **app1-termék-vnet** számára a **éles** környezetben, és egy másik nevesített **app1-fejlesztői-vnet** számára a **fejlesztési** környezetben. Ebben az esetben **app1 számítási feladatok felelőse** egy kérést küld a **hálózati műveletek** felhasználói társviszony-létesítési VPN-kapcsolat létrehozásához. Vegye figyelembe, hogy **app1 számítási feladatok felelőse** adja ugyanazt a címkét az első példában, és a kapacitás a számláló az előfizetés fennmaradó 47 virtuális hálózatok száma csökken új.
![](../_images/governance-3-13.png)
3. A **előfizetés tulajdonosa** most létrehozza a két erőforráscsoport **app2 számítási feladatok felelőse**. Az azonos egyezmények megegyezik a következő **app1 számítási feladatok felelőse**, az erőforráscsoportok megnevezett **app2-termék-rg** és **app2-fejlesztői-rg**. A **előfizetés tulajdonosa** hozzáadja **app2 számítási feladatok felelőse** az egyes tartalmazó erőforráscsoportokat a **közreműködő** szerepkör.
![](../_images/governance-3-14.png)
4. *App2 számítási feladatok felelőse* telepíti a virtuális hálózatok és virtuális gépek az erőforráscsoportok az ugyanazokat az elnevezési konvenciókat. Címkék ad hozzá, és a korlát számláló száma csökken, a fennmaradó 45 virtuális hálózatokhoz a *előfizetés*.
![](../_images/governance-3-15.png)
5. *App2 számítási feladatok felelőse* egy kérést küld a *hálózati műveletek* egyenrangú felhasználói a *app2-termék-vnet* rendelkező a *hub-vnet*. A *hálózati műveletek* felhasználó a társviszony-létesítési kapcsolatot hoz létre.
![](../_images/governance-3-16.png)

Az eredményül kapott felügyeleti modell hasonlít az első példában több kulcs eltérésekkel:
* A két munkaterhelések egy elkülönített munkaterhelés és a környezetben.
* Ez a modell két további virtuális hálózatok, mint az első példában modell szükséges. Ez nem fontos különbség a két munkaterhelések, a munkaterhelések ehhez a modellhez elméleti korlátot: 24. 
* Erőforrások többé nem egyetlen erőforráscsoportként működnek környezetben vannak csoportosítva. Erőforrások csoportosítása a környezetben használt elnevezési konvenciókat ismeretét igényli. 
* A virtuális hálózati társviszonyban kapcsolatok mindegyikének lett vizsgálni és jóvá a *hálózati műveletek* felhasználó.

Most már a felügyeleti modell használatával több előfizetéssel vizsgáljuk meg. Ebben a modellben, azt fogja igazítása a három különálló előfizetés környezetek mindegyikének: egy **megosztott szolgáltatások** előfizetés, **éles** előfizetés, és végül a **fejlesztési** előfizetés. Ez a modell szempontjai hasonlóak a modell használatával egy-egy előfizetéshez abban, hogy döntse el, hogy a munkaterhelések erőforráscsoportra igazítása. A Microsoft már meghatározta, hogy az egyes munkaterhelésekhez tartozó erőforráscsoport létrehozása megfelel a munkaterhelés elkülönítési követelmény, azt fogja odatapadjon ebben a példában a modellt.

1. Ebben a modellben van három *előfizetések*: *megosztott infrastruktúra*, *éles*, és *fejlesztési*. Az alábbi három előfizetések igényel a *előfizetés tulajdonosa*, és az egyszerű példában ugyanazt a felhasználói fiókot mind a három fogjuk használni. A *megosztott infrastruktúra* erőforrások az első két fenti példák a hasonló módon kezelik, és az első munkaterhelés társítva van a *app1-rg* a a *éles* bejövő csoportosítás környezet és az azonos nevű erőforrás a *fejlesztési* környezetben. A *app1 számítási feladatok felelőse* minden az erőforráscsoport a *közreműködő* szerepkör. 
![](../_images/governance-3-17.png)
2. Csakúgy, mint a korábbi példában, *app1 számítási feladatok felelőse* hoz létre az erőforrásokat, és a társviszony-létesítési kapcsolatot igényel a *megosztott infrastruktúra* virtuális hálózat. *Az App1 számítási feladatok felelőse* csak hozzáadja a *felügyeli* címkét, mert már nincs szükség a *környezet* címke. Ez azt jelenti, hogy erőforrások vannak az egyes környezetekben csoportba kerülnek, ugyanúgy *előfizetés* és a *környezet* címke már redundáns. A korlát a számláló akkor száma csökken, a fennmaradó 49 virtuális hálózatokhoz.
![](../_images/governance-3-18.png)
3. Végezetül a *előfizetés tulajdonosa* a folyamat ismétlődik a második terheléshez, tartalmazó erőforráscsoportokat hozzáadása a *app2 számítási feladatok felelőse* a a * közreműködői szerepkört. A korlát a számláló az egyes a környezet előfizetések akkor száma csökken, a fennmaradó 48 virtuális hálózatokhoz. 

A felügyeleti modellnek a második példában előnyeit. Azonban a legfontosabb különbség az, hogy korlátok kisebbek, a probléma oka az oka, hogy azok van elosztva, több mint két *előfizetések*. A hátránya, hogy a címkék követik költség adatokat kell gyűjtődnek mind a hármat *előfizetések*. 

Ezért kiválaszthatja, bármely ezek két példa erőforrás modell attól függően, hogy a követelmények prioritását. Ha várhatóan, hogy a szervezet nem érte el a szolgáltatásra vonatkozó korlátozások egy-egy előfizetéshez tartozó, egy-egy előfizetéshez több erőforráscsoportok használható. Ezzel szemben ha a szervezet azt feltételezi, hogy számos különféle munkaterheléshez, környezetben több előfizetéssel jobb megoldás lehet.

<!-- ## Summary



## Next steps -->


<!-- links -->

[rbac-built-in-owner]: /azure/role-based-access-control/built-in-roles#owner
[rbac-built-in-roles]: /azure/role-based-access-control/built-in-roles
