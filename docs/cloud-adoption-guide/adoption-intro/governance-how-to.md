---
title: Az Azure cégirányítási kialakítási útmutató
description: Útmutató ahhoz, hogy egy felhasználó egy egyszerű számítási feladat üzembe helyezéséhez Azure cégirányítási vezérlők konfigurálása
author: petertay
ms.openlocfilehash: 78545400fc0b09262dce3d0e577442443468b2ed
ms.sourcegitcommit: c704d5d51c8f9bbab26465941ddcf267040a8459
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 07/24/2018
ms.locfileid: "39229695"
---
# <a name="azure-governance-design-guide"></a>Az Azure cégirányítási kialakítási útmutató

A jelen tervezési útmutató célközönségét a *központi informatikai* személy a szervezetben. *Központi informatikai* , amelyek segítenek a szervezet cégirányítási architektúra felelős. Amint már tudja, az a [Mi a felhőbeli erőforrás-szabályozás?](governance-explainer.md) ismertető, cégirányítási hivatkozik folyamatban kezelését, megfigyelését és naplózás használata Azure-erőforrások megfelelnek a kitűzött célokat, és a szervezet követelményeinek.

Ez az útmutató célja, amely megkönnyíti a szervezet cégirányítási architektúra tervezése naplófájlbejegyzéseket átnézve elméleti cégirányítási céljainak és követelményeinek. Ezt követően szó lesz konfigurálása az Azure cégirányítási eszközök számára. 

Az alapszintű bevezetési szakasz a célunk, üzembe helyezéséhez egy egyszerű számítási feladat Azure-bA. Az eredmény a következő követelményeknek:
* Az Identitáskezelés egyetlen **számítási feladatok felelőse** üzembe helyezése és karbantartása a egyszerű számítási feladat felelős. A számítási feladatok felelőse létrehozása, olvasása, frissítése és törlése, erőforrás, valamint a identity management rendszer ezeket a jogokat más felhasználóknak meghatalmazásához engedélyekkel engedélyezésére van szükség.
* Az egyszerű számítási feladat összes erőforrásainak kezelése egyetlen felügyeleti egységet.

## <a name="licensing-azure"></a>Licencek használata az Azure

Mielőtt megkezdjük a cégirányítási modell tervezéséhez, fontos tudni, hogyan licencelve van-e az Azure. Ennek oka az, a rendszergazdai fiókok az Azure-licenchez tartozó rendelkezik a legmagasabb szintű összes az Azure-erőforrásokhoz való hozzáférés. Ezek a rendszergazdai fiókok vevőrendelési a cégirányítási modell.  

> [!NOTE]
> Ha a szervezet rendelkezik egy meglévő [Microsoft nagyvállalati szerződés](https://www.microsoft.com/licensing/licensing-programs/enterprise.aspx) , amely nem tartalmazza az Azure, Azure azáltal, hogy a vonatkozó előzetes pénzügyi kötelezettségvállalást is hozzáadhatók. Lásd: [nagyvállalati licencek használata az Azure](https://azure.microsoft.com/pricing/enterprise-agreement/) további információt. 

Az Azure ad hozzá a szervezet nagyvállalati szerződéssel, a szervezet lett arra kéri, hogy hozzon létre egy **Azure-fiók**. A fiók létrehozása során egy **Azure-fiók tulajdonosa** lett létrehozva, valamint egy Azure Active Directory (Azure AD) bérlő az olyan **globális rendszergazdai** fiók. Az Azure AD-bérlő egy logikai szerkezet, amely egy Azure AD biztonságos, dedikált példányát jelöli.

![Azure-fiókot az Azure Account Manager és az Azure AD globális rendszergazda](../_images/governance-3-0.png)
*1. ábra. Egy Azure-fiók Account Manager, és az Azure AD globális rendszergazda.*

## <a name="identity-management"></a>Identitáskezelés

Az Azure csak megbízik [Azure ad-ben](/azure/active-directory) felhasználók hitelesítéséhez és engedélyezéséhez a felhasználói hozzáférést az erőforrásokhoz, így az Azure AD identity management rendszerben. Az Azure AD globális rendszergazda a legmagasabb szintű jogosultságokkal rendelkezik, és identitás, beleértve a felhasználók létrehozása és az engedélyek hozzárendelése a kapcsolódó összes művelet elvégezhető. 

A követelmény, az Identitáskezelés egyetlen **számítási feladatok felelőse** üzembe helyezése és karbantartása a egyszerű számítási feladat felelős. A számítási feladatok felelőse létrehozása, olvasása, frissítése és törlése, erőforrás, valamint a identity management rendszer ezeket a jogokat más felhasználóknak meghatalmazásához engedélyekkel engedélyezésére van szükség.

Az Azure AD globális rendszergazda létrehozza a **számítási feladatok felelőse** fiók számára a **számítási feladatok felelőse**:

![Az Azure AD globális rendszergazda létrehozza a számítási feladatok tulajdonosának fiókot](../_images/governance-1-2.png)
*2. ábra. Az Azure AD globális rendszergazda a számítási feladatok tulajdonosának felhasználói fiókot hoz létre.*

A Microsoft nem tudják erőforrás-hozzáférési engedélyek hozzárendelése, amíg a felhasználó nincs hozzáadva egy **előfizetés**, így azt kell végeznie, amelyek a következő két szakasz. 

## <a name="resource-management-scope"></a>Erőforrás-kezelési hatókör

A szervezet által üzembe helyezett erőforrások számának növekedésével ezeket az erőforrásokat szabályozó összetettségét is növekszik. Az erőforrások a csoportok különböző szintjein részletességgel, más néven kezeléséhez a szervezet logikai tárolóhierarchiában az Azure csomagszűrő **hatókör**. 

A legfelső szintű erőforrás felügyeleti hatóköre van az **előfizetés** szintjét. Az Azure által létrehozott előfizetés **fióktulajdonos**, akik megállapítja, hogy a pénzügyi kötelezettségvállalás nélkül, és az összes Azure-erőforrások lemondott az előfizetéshez tartozó:

![Az Azure-fiók tulajdonosa egy előfizetést hoz létre](../_images/governance-1-3.png)
*3. ábra. Az Azure-fiók tulajdonosa egy előfizetést hoz létre.*

Az előfizetés létrehozásakor, az Azure **fióktulajdonos** az előfizetés hozzárendeli egy Azure AD-bérlőben, és az Azure AD-bérlő hitelesítési és engedélyezési szolgál:

![Az Azure-fiók tulajdonosa az előfizetés társítja, az Azure AD-bérlő](../_images/governance-1-4.png)
*4. ábra. Az Azure-fiók tulajdonosa az előfizetés az Azure AD-bérlő társítja.*

Észrevehető, hogy jelenleg nem, ami azt jelenti, hogy senki nem jogosult erőforrások kezelése az előfizetéshez tartozó felhasználó nem. A valóságban a végzett a **fióktulajdonos** az előfizetés tulajdonosa, amely jogosult arra, hogy bármilyen műveletet végrehajthat egy erőforrást az előfizetésben. Azonban a gyakorlatilag a **fióktulajdonos** , valószínűleg több, mint a szervezet pénzügyi személy, és nem vállal a létrehozása, olvasása, frissítése és törlése erőforrások – ezeket a feladatokat kell elvégeznie a **számítási feladatok felelőse**. Ezért hozzá kell adnunk a a **számítási feladatok felelőse** az előfizetés és rendelhet hozzá engedélyeket.

Mivel a **fióktulajdonos** jelenleg az egyetlen felhasználó-hozzáadási jogosultsággal rendelkező a **számítási feladatok felelőse** adnak hozzá az előfizetéséhez, a **számítási feladatok felelőse** az előfizetéshez:

![Az Azure-fiók tulajdonosa hozzáadja a ** számítási feladatok tulajdonosának ** előfizetéshez](../_images/governance-1-5.png)
*5. ábra. Az Azure-fiók tulajdonosa az előfizetés ad hozzá a számítási feladatok felelőse.*

Az Azure **fióktulajdonos** engedélyt ad a **számítási feladatok felelőse** hozzárendelésével egy [szerepköralapú hozzáférés-vezérlés (RBAC)](/azure/role-based-access-control/) szerepkör. Az RBAC szerepkör engedélyek egy halmazát határozza meg, amely a **számítási feladatok felelőse** rendelkezik egy egyéni erőforrás típusa vagy egy erőforrás-típus.

Figyelje meg, hogy ebben a példában a **fióktulajdonos** hozzá van rendelve a [beépített **tulajdonosa** szerepkör](/azure/role-based-access-control/built-in-roles#owner): 

![A ** számítási feladatok tulajdonosának ** hozzá volt rendelve a beépített tulajdonosi szerepkör](../_images/governance-1-6.png)
*6. ábra. A számítási feladatok felelőse a beépített tulajdonosi szerepkör van hozzárendelve.*

A beépített **tulajdonosa** szerepkör az összes engedélyt megadja a **számítási feladatok felelőse** az előfizetések szintjén. 

> [!IMPORTANT]
> Az Azure **fiók tulajdonosa** felelős a pénzügyi kötelezettségvállalás, az előfizetéshez tartozó, de a **számítási feladatok felelőse** ugyanazokkal az engedélyekkel rendelkezik. A **fióktulajdonos** meg kell bíznia az **számítási feladatok felelőse** erőforrást, amely az előfizetés Költségvetésen belül üzembe helyezéséhez.

A következő szint a felügyeleti hatókör a **erőforráscsoport** szintjét. Egy erőforráscsoportot az erőforrások logikai tárolója. Műveletek a alkalmazni az erőforráscsoport szintjén alkalmazni a csoportban található összes erőforrást. Azt is vegye figyelembe, hogy minden egyes felhasználó engedélyeinek örökli a következő szintre be, ha explicit módon módosultak a hatókörben. 

Ennek szemléltetésére nézzük, mi történik, ha a **számítási feladatok felelőse** létrehoz egy erőforráscsoportot:

![A ** számítási feladatok tulajdonosának ** létrehoz egy erőforráscsoportot](../_images/governance-1-7.png)
*7. ábra. A számítási feladatok felelőse létrehoz egy erőforráscsoportot, és a beépített tulajdonosi szerepkör erőforrás-csoportot a csoporthatókörben, örökli.*

Ismét a beépített **tulajdonosa** szerepkör az összes engedélyt megadja a **számítási feladatok felelőse** az erőforrás-csoport hatókörben. Ahogy már volt szó korábbi, ezt a szerepkört az előfizetés szintjén öröklődik. Ha egy másik szerepkör van rendelve a felhasználóhoz a hatókörben, csak ez a hatókör vonatkozik.

A legalsó szinten lévő felügyeleti hatókör jelenleg a **erőforrás** szintjét. Az erőforráscsoport szintjén alkalmazzák-műveletek csak erőforrásra vonatkoznak. És ismét az erőforráscsoport szintjén engedélyek erőforrás csoporthatókör öröklődnek. Például nézzük, mi történik, ha a **számítási feladatok felelőse** helyez üzembe egy [virtuális hálózat](/azure/virtual-network/virtual-networks-overview) az erőforrás-csoportba:

![A ** számítási feladatok tulajdonosának ** létrehoz egy erőforrást](../_images/governance-1-8.png)
*8. ábra. A számítási feladatok felelőse erőforrást hoz létre, és örökli a beépített tulajdonosi szerepkör a erőforrás hatókörben.*

A **számítási feladatok felelőse** örökli a tulajdonosi szerepkör a erőforrás hatókörben, ami azt jelenti, hogy a számítási feladatok felelőse a virtuális hálózat minden olyan engedéllyel rendelkezik. 

## <a name="summary"></a>Összegzés

Ebben a cikkben megtanulta:

* Az Azure csak az Azure AD identity Management megbízik.
* Előfizetés erőforrás-kezelés legnagyobb hatókörét, és minden egyes előfizetés Azure AD-bérlő társítva. Csak a felhasználók a társított Azure ad-bérlő az előfizetésben található erőforrások eléréséhez.
* Erőforrás-kezelési hatókör három szinten: előfizetés, erőforráscsoport és erőforrások. Engedélyek az RBAC-szerepkörök használatával minden egyes hatókörben vannak hozzárendelve. RBAC-szerepkörök öröklődnek nagyobb hatókörrel hatókör csökkentéséhez.

## <a name="next-steps"></a>További lépések

Lépjen vissza a [alapszintű bevezetési szakasz áttekintése](overview.md) megtudhatja, hogyan e goverance modell megvalósításához. Ezután válasszon ki egy számítási feladat, és megtudhatja, hogyan helyezze üzembe.