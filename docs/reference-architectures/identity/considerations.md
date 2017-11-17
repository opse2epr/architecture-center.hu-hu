---
title: "Válassza ki a helyszíni Active Directory integrálása az Azure a megoldást."
description: "Összehasonlítja a helyszíni Active Directory integrálása Azure architektúrához hivatkozik."
ms.date: 04/06/2017
ms.openlocfilehash: 413a5463d90547197c4b6834d353b4ecf61483ee
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="choose-a-solution-for-integrating-on-premises-active-directory-with-azure"></a>Válassza ki a megoldást a helyszíni Active Directory integrálása az Azure-ral

Ez a cikk összehasonlítja a helyszíni Active Directory (AD) környezetben integrálása az Azure-hálózat beállításait. Az egyes lehetőségek nyújtunk a referencia-architektúrában és a központilag telepíthető megoldás.

Számos szervezet használ [Active Directory tartományi szolgáltatások (AD DS)] [ active-directory-domain-services] hitelesítéshez társított felhasználók, számítógépek, alkalmazások vagy más erőforrásokat, amelyek szerepelnek a biztonsági azonosítók határ. Címtár- és identitáskezelési szolgáltatásainak használata általában üzemeltethető a helyszínen, de ha az alkalmazás üzemel részben helyszíni és részben az Azure-ban előfordulhat, küldő hitelesítési kérelmek az Azure-ból a helyszíni biztonsági késés. Címtár- és identitáskezelési szolgáltatásokat végrehajtása az Azure-ban csökkenti a késés.

Azure két megoldást nyújt a címtár- és identitáskezelési szolgáltatásokat végrehajtása az Azure-ban: 

* Használjon [az Azure AD] [ azure-active-directory] Active Directory-tartomány létrehozása a felhőben, és csatlakoztassa a helyszíni Active Directory-tartományhoz. [Az Azure AD Connect] [ azure-ad-connect] integrálható a helyszíni címtárakat az Azure AD.

* Az Azure-ba, a meglévő helyszíni Active Directory-infrastruktúra meghosszabbíthatja egy Azure-ban futó Active Directory tartományi szolgáltatások tartományvezérlő telepítését. Ez az architektúra napjainkban egyre általánosabbá, VPN vagy ExpressRoute kapcsolattal csatlakoztatott a helyszíni hálózat és az Azure virtuális hálózat (VNet). Ez az architektúra több változatait lehetségesek: 

    - Hozzon létre egy tartományt az Azure-ban, és csatlakoztassa azt a helyszíni Active Directory-erdőben.
    - Hozzon létre egy másik erdőben, amely megbízható-e a helyi erdőben lévő tartományokba Azure-ban.
    - Az Active Directory összevonási szolgáltatások (AD FS) központi telepítése replikálása Azure-bA. 

A következő szakaszok ismertetik részletesebben ezek közül.

## <a name="integrate-your-on-premises-domains-with-azure-ad"></a>A helyszíni tartományok integrálása az Azure ad szolgáltatással

Azure Active Directory (Azure AD) segítségével az Azure-tartomány létrehozása hozzákapcsolja azt egy a helyszíni Active Directory-tartománynak. 

Az Azure AD-címtár nincs kiterjesztése a helyszíni címtárba. Ehelyett egy másolatot, amely tartalmazza az ugyanazon objektumokat és identitások. Ezen elemek helyszíni módosításainak másolja az Azure AD, de a helyszíni tartománynév nem replikálja a Azure AD-ben végrehajtott módosításokat.

Az Azure AD egy helyszíni címtár használata nélkül is használható. Ebben az esetben az Azure AD szolgáltatásba, illetve az elsődleges adatforrás összes azonosító adatok, nem pedig egy helyi könyvtárból replikált adatokat tartalmazó.


**Előnyei**

* Nem kell fenntartani egy AD-infrastruktúrát, a felhőben. Az Azure AD teljes mértékben felügyelt és a Microsoft által fenntartott.
* Az Azure AD elérhető helyszíni identitás adatokat biztosít.
* Hitelesítési akkor fordulhat elő, így a külső alkalmazások és a felhasználók csatlakozni tudnak a helyi tartomány, az Azure-ban.

**Kihívásai**

* Identitás-szolgáltatások a felhasználók és csoportok korlátozódnak. Nincs nincs elvégezhessék a hitelesítést a szolgáltatás- és számítógépes fiókok.
* Konfigurálnia kell a kapcsolat a helyszíni tartománnyal tartani az Azure AD directory szinkronizálva. 
* Alkalmazások kell hitelesítést az Azure AD keresztül kell írni.

**[További tudnivalók...][aad]**

## <a name="ad-ds-in-azure-joined-to-an-on-premises-forest"></a>Az Azure Active Directory tartományi szolgáltatások csatlakozik egy helyszíni erdőben

Azure AD tartományi szolgáltatások (AD DS) kiszolgálók üzembe. Hozzon létre egy tartományt az Azure-ban, és csatlakoztassa azt a helyszíni Active Directory-erdőben. 

Fontolja meg ezt a beállítást, ha szeretné használni az Active Directory Tartományi szolgáltatások, amelyek jelenleg nem működik az Azure AD. 

**Előnyei**

* Rendelkezésre álló helyszíni identitás adatokat hozzáférést biztosít.
* Képes hitelesíteni a felhasználót, a szolgáltatás és a számítógép fiókok a helyszíni és az Azure-ban.
* Nem kell kezelni egy másik AD-erdőben. A tartomány az Azure-ban a helyi erdőben is tartozhatnak.
* A csoportházirend határozzák meg a helyi csoportházirend-objektumok a tartományhoz az Azure-ban is alkalmazhat.

**Kihívásai**

* Kell telepíti és kezeli a saját Active Directory tartományi szolgáltatások-kiszolgálók és a tartományt a felhőben.
* Előfordulhat, hogy egyes szinkronizálási várakozási ideje a tartománykiszolgálók a felhőben és helyszíni futtató kiszolgálók között.

**[További tudnivalók...][ad-ds]**

## <a name="ad-ds-in-azure-with-a-separate-forest"></a>Active Directory tartományi szolgáltatások az Azure-ban egy másik erdőben

Az Azure AD tartományi szolgáltatások (AD DS) kiszolgálók telepítése, de hozzon létre egy külön Active Directory [erdő] [ ad-forest-defn] , amely nem csatlakozik a helyi erdőben. Ez az erdő megbízhatónak a helyi erdőben lévő tartományokba.

Ez az architektúra a gyakori felhasználási tartalmaznak objektumok és az identitások, a felhőben tárolt, és a helyszíni egyéni tartományok telepít át a felhő biztonsági elkülönítést karbantartása.

**Előnyei**

* A helyszíni identitások megvalósítása, és külön csak Azure identitásokat.
* Nem kell replikálni a helyszíni AD-erdőben, az Azure-bA.

**Kihívásai**

* Az Azure a helyszíni identitások hitelesítéshez extra hálózati ugrásokat a helyszíni AD-kiszolgálók.
* A saját Active Directory tartományi szolgáltatások kiszolgálót telepíteni, a felhőben erdőben, és erdők közötti bizalmi kapcsolatok létesítése.

**[További tudnivalók...][ad-ds-forest]**

## <a name="extend-ad-fs-to-azure"></a>Az Azure Active Directory összevonási szolgáltatások kiterjesztése

Az Active Directory összevonási szolgáltatások (AD FS) központi telepítése replikálása Azure-ba, az Azure-beli összetevők összevont hitelesítési és engedélyezési végrehajtásához. 

Ez az architektúra a gyakori felhasználási:

* Felhasználók hitelesítéséhez és engedélyezéséhez a fiókpartner-szervezetek.
* Lehetővé teszi a felhasználók számára a szervezeti tűzfalon kívül futó webböngészőknek a hitelesítést.
* Csatlakoztatja a hitelesített külső eszközöket, például a mobileszközök engedélyezése. 

**Előnyei**

* Kihasználhatja a jogcímbarát alkalmazásokhoz.
* Lehetővé teszi a hitelesítés külső partnerekkel megbízhatónak.
* Kompatibilitási nagy számú hitelesítési protokollok megvalósítását végzi.

**Kihívásai**

* Ha már telepítette a saját Azure Active Directory tartományi szolgáltatások, az AD FS és az AD FS webalkalmazás-Proxy kiszolgálók.
* Ez az architektúra konfigurálása összetett feladat lehet.

**[További tudnivalók...][adfs]**

<!-- links -->

[aad]: ./azure-ad.md
[ad-ds]: ./adds-extend-domain.md
[ad-ds-forest]: ./adds-forest.md
[ad-forest-defn]: https://msdn.microsoft.com/library/ms676906.aspx
[adfs]: ./adfs.md

[active-directory-domain-services]: https://technet.microsoft.com/library/dd448614.aspx
[azure-active-directory]: /azure/active-directory-domain-services/active-directory-ds-overview
[azure-ad-connect]: /azure/active-directory/active-directory-aadconnect
