---
title: Helyszíni Active Directory integrálása az Azure-ban
titleSuffix: Azure Reference Architectures
description: Referenciaarchitektúrák összehasonlítása a helyszíni Active Directory Azure-ban való integrálásához.
ms.date: 07/02/2018
ms.custom: seodec18
ms.openlocfilehash: 905dedda6de1a107f55b2f7651441780a685aea7
ms.sourcegitcommit: 88a68c7e9b6b772172b7faa4b9fd9c061a9f7e9d
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/08/2018
ms.locfileid: "53119863"
---
# <a name="choose-a-solution-for-integrating-on-premises-active-directory-with-azure"></a>Válasszon egy megoldást a helyszíni Active Directory Azure-ban való integráláshoz.

Ez a cikk a helyszíni Active Directory (AD) környezet Azure-hálózattal való integrálásának lehetőségeit hasonlítja össze. Mindegyik lehetőséghez elérhető egy referenciaarchitektúra.

Számos vállalat használja az Active Directory Domain Services (AD DS) szolgáltatásokat a felhasználókhoz, számítógépekhez, alkalmazásokhoz vagy egyéb, biztonsági határral védett erőforrásokhoz társított identitások hitelesítésére. A címtár- és identitásszolgáltatások általában helyszíni üzemeltetésűek, ám ha az alkalmazás üzemeltetése részben a helyszínen, részben pedig az Azure-ban történik, akkor késés léphet fel a hitelesítési kérések az Azure-ból a helyszínre történő küldése során. A késés csökkenthető a címtár- és identitásszolgáltatások Azure-beli implementálásával.

Az Azure két megoldást kínál a címtár- és identitásszolgáltatások Azure-beli implementációjához:

- Az [Azure AD] [ azure-active-directory] használatával létrehozhat egy Active Directory-tartományt a felhőben, és csatlakoztathatja azt a helyszíni Active Directory-tartományhoz. [Az Azure AD Connect] [ azure-ad-connect] integrálja a helyszíni címtárakat az Azure AD-vel.

- Kiterjesztheti meglévő helyszíni Active Directory-infrastruktúráját az Azure-ba, ha egy olyan virtuális gépet helyez üzembe az Azure-ban, amely tartományvezérlőként futtatja az AD DS-t. Ez az architektúra általában akkor használatos, amikor a helyszíni hálózatot és az Azure-beli virtuális hálózatot (VNet) VPN- vagy ExpressRoute-kapcsolat köti össze. Ennek az architektúrának több változata is lehetséges:

  - Létrehozhat egy tartományt az Azure-ban, és csatlakoztathatja azt a helyszíni AD-erdőhöz.
  - Létrehozhat egy külön erdőt az Azure-ban, amelyet a helyszíni erdő tartományai megbízhatónak tekintenek.
  - Replikálhatja az Active Directory összevonási szolgáltatások (AD FS) egy üzemelő példányát az Azure-ba.

A következő szakaszokban a felsorolt lehetőségek részletes leírása szerepel.

## <a name="integrate-your-on-premises-domains-with-azure-ad"></a>Helyszíni tartományok integrálása az Azure AD-be

Az Azure Active Directory (Azure AD) használatával hozzon létre egy tartományt az Azure-ban, és csatlakoztassa azt egy helyszíni Active Directory-tartományhoz.

Az Azure AD-címtár nem a helyszíni címtár kiterjesztése,  hanem egy ugyanazon objektumokat és identitásokat tartalmazó másolat. Az elemeken a helyszínen elvégzett módosításokat a rendszer átmásolja az Azure AD-be, az Azure AD-ben végrehajtott módosításokat azonban nem replikálja a helyszíni tartományra.

Az Azure AD-t egy helyszíni címtár használata nélkül is használhatja. Ebben az esetben az Azure AD az azonosító adatok elsődleges forrásaként szolgál, és nem tartalmaz a helyszíni címtárból replikált adatokat.

**Előnyök**

- Nem kell fenntartania AD-infrastruktúrát a felhőben. Az Azure AD fenntartását és felügyeletét teljes mértékben a Microsoft végzi.
- Az Azure AD ugyanazokat az azonosító adatokat biztosítja, amelyek a helyszínen is elérhetők.
- A hitelesítés az Azure-ban is megtörténhet, ezáltal a külső alkalmazásoknak és a felhasználóknak ritkábban kell kapcsolatba lépniük a helyszíni tartománnyal.

**Problémák**

- Az identitásszolgáltatások a felhasználókra és a csoportokra korlátozódnak. Nincs lehetőség a szolgáltatás- és számítógépes fiókok hitelesítésére.
- Az Azure AD-címtár folyamatos szinkronizálásához konfigurálnia kell a kapcsolatot a helyszíni tartománnyal. 
- Előfordulhat, hogy az alkalmazásokat újra kell írni az Azure AD-n keresztüli hitelesítés engedélyezéséhez.

**Referenciaarchitektúra**

- [Helyszíni Active Directory-tartományok integrálása az Azure Active Directoryval][aad]

## <a name="ad-ds-in-azure-joined-to-an-on-premises-forest"></a>Helyszíni erdőhöz csatlakozó AD DS az Azure-ban

Azure AD Domain Services- (AD DS) kiszolgálókat helyezhet üzembe az Azure-ban. Létrehozhat egy tartományt az Azure-ban, és csatlakoztathatja azt a helyszíni AD-erdőhöz. 

Fontolja meg ezt a lehetőséget, ha az AD DS azon funkcióit szeretné használni, amelyek jelenleg nincsenek implementálva az Azure AD-ben. 

**Előnyök**

- Hozzáférést biztosít a helyszínen is elérhető azonosító adatokhoz.
- Felhasználói, szolgáltatás- és számítógépes fiókok hitelesítését a helyszíni környezetben és az Azure-ban is elvégezheti.
- Nincs szükség külön AD-erdő kezelésére. Az Azure-beli tartomány a helyi erdőhöz is tartozhat.
- A helyszíni csoportszabályzat-objektumok által meghatározott csoportszabályzatot az Azure-beli tartományra is alkalmazhatja.

**Problémák**

- Saját magának kell elvégeznie az AD DS-kiszolgálók és a tartomány üzembe helyezését és kezelését a felhőben.
- A felhőbeli tartománykiszolgálók és helyszínen futó kiszolgálók között előfordulhat némi szinkronizálási késés.

**Referenciaarchitektúra**

- [Az Azure Active Directory Domain Services (AD DS) kiterjesztése az Azure-ra][ad-ds]

## <a name="ad-ds-in-azure-with-a-separate-forest"></a>Külön erdővel rendelkező AD DS az Azure-ban

Helyezzen üzembe AD Domain Services- (AD DS) kiszolgálókat az Azure-ban, de hozzon létre egy külön Active Directory-[erdőt][ad-forest-defn], amely elkülönül a helyszíni erdőtől. Ezt az erdőt a helyszíni erdő tartományai megbízhatónak tekintik.

Az architektúra gyakori használati módjai például a felhőben tárolt objektumok és identitások biztonsági elkülönítésének fenntartása, illetve az egyes tartományok migrálása a helyszínről a felhőbe.

**Előnyök**

- Helyszíni és külön, csak az Azure-hoz tartozó identitásokat is megvalósíthat.
- Nem kell replikációt végezni a helyszíni AD-erdőből az Azure-ba.

**Problémák**

- A helyszíni identitások Azure-beli hitelesítéshez több, a helyszíni AD-kiszolgálókhoz való hálózati ugrásra van szükség.
- Saját AD DS-kiszolgálókat és erdőt kell üzembe helyeznie a felhőben, és az erdők között megfelelő megbízhatósági kapcsolatokat kell létesítenie.

**Referenciaarchitektúra**

- [Active Directory Domain Services- (AD DS-) erőforráserdő létrehozása az Azure-ban][ad-ds-forest]

## <a name="extend-ad-fs-to-azure"></a>Az AD FS kiterjesztése az Azure-ra

Replikáljon az Active Directory összevonási szolgáltatások (AD FS) egy üzemelő példányát az Azure-ba az Azure-beli összetevők összevont hitelesítéséhez és engedélyezéséhez. 

Az architektúra gyakori használati módjai a következők:

- Partnerszervezetek felhasználóinak hitelesítése és engedélyezése.
- A felhasználói hitelesítés engedélyezése a vállalati tűzfalon kívül futó webböngészőkből.
- A felhasználók hitelesített külső eszközökről, például mobileszközökről történő csatlakozásának engedélyezése. 

**Előnyök**

- Kihasználhatja a jogcímbarát alkalmazások előnyeit.
- Lehetővé teszi, hogy megbízzon a külső partnerekben a hitelesítés tekintetében.
- Nagy hitelesítésiprotokoll-készletekkel is kompatibilis.

**Problémák**

- Saját kezűleg kell üzembe helyeznie AD DS, AD FS és AD FS webalkalmazás-proxy kiszolgálóit az Azure-ban.
- Az architektúra konfigurálása bonyolult lehet.

**Referenciaarchitektúra**

- [Az Active Directory összevonási szolgáltatások (AD FS) kiterjesztése az Azure-ra][adfs]

<!-- links -->

[aad]: ./azure-ad.md
[ad-ds]: ./adds-extend-domain.md
[ad-ds-forest]: ./adds-forest.md
[ad-forest-defn]: /windows/desktop/AD/forests
[adfs]: ./adfs.md

[azure-active-directory]: /azure/active-directory-domain-services/active-directory-ds-overview
[azure-ad-connect]: /azure/active-directory/hybrid/whatis-hybrid-identity
