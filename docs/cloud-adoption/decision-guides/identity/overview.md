---
title: 'CAF: Identitáskezelés döntési – útmutató'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: További információ a identitás Azure áttelepítések core szolgáltatás.
author: rotycenh
ms.openlocfilehash: addc11928a0bc72917a28b468a04880720a1fdf4
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899203"
---
# <a name="identity-decision-guide"></a>Identitáskezelés döntési – útmutató

Bármilyen környezet akár helyi, hibrid, vagy csak felhőalapú, IT-igényeket is szabályozhatja, mely a rendszergazdák, a felhasználók és csoportok hozzáféréssel rendelkezik az erőforrásokhoz. Identitás és hozzáférés-kezelés (IAM) szolgáltatás lehetővé teszi a felhőbeli hozzáférés-vezérlés kezelése.

![Küldik az ábrázolást legalább az identitásbeállításokat a legösszetettebb, az alábbi hivatkozások a jump igazítva](../../_images/discovery-guides/discovery-guide-identity.png)

Ugrás ide: [Identity Integration követelmények meghatározása](#determine-identity-integration-requirements) | [Felhőbeli natív](#cloud-baseline) | [címtár-szinkronizálás](#directory-synchronization) | [a felhőben tartományi szolgáltatások](#cloud-hosted-domain-services) | [Active Directory összevonási szolgáltatások](#active-directory-federation-services) | [identitás integrációs](#evolving-identity-integration)  |  [További](#learn-more)

Többféleképpen is Identitáskezelés a felhőalapú környezetekben, amelyek eltérőek lehetnek a költségek és munkaráfordítás alól. A felhőalapú identitás-szolgáltatások strukturálja kulcsfontosságú tényező az integráció a meglévő helyszíni identitás-infrastruktúrát a szükséges.

A szoftverszolgáltatások (SaaS) identitáskezelési megoldások szoftver felhőalapú adja meg a felhőbeli erőforrások hozzáférés-vezérléshez és identitáskezeléshez kezelés alapszinten. Azonban ha a szervezet Active Directory (AD) infrastruktúra rendelkezik olyan összetett erdő struktúrájába vagy testre szabott szervezeti egységhez (OU), a felhőalapú számítási feladatokhoz szükség lehet címtárreplikáció a felhőbe az identitások, csoportok, a konzisztens és a szerepkörök között a helyszíni és felhőalapú környezetekben. Kötelező globális megoldás directory replikáció esetén bonyolultságát jelentősen növelheti. Emellett függ az örökölt hitelesítési mechanizmusok alkalmazások támogatása a tartományi szolgáltatások a felhőben üzembe helyezési lehet szükség.

## <a name="determine-identity-integration-requirements"></a>Identity integration követelmények meghatározása

| Kérdés | Felhő-alapkonfiguráció | A címtár-szinkronizálás | Cloud-hosted Domain Services | AD összevonási szolgáltatások |
|------|------|------|------|------|
| Jelenleg nincs egy helyi címtárszolgáltatás? | Igen | Nem | Nem | Nem |
| Szükség van a számítási feladatokat a helyszíni identitás-szolgáltatások hitelesítése? | Nem | Igen | Nem | Nem |
| Hajtsa végre a számítási feladatok függ az örökölt hitelesítési mechanizmusok, például a Kerberos vagy NTLM? | Nem | Nem | Igen | Nem |
| Integráció az között felhőalapú és helyszíni identitás-szolgáltatások lehetetlen? | Nem | Nem | Igen | Nem |
| Szükség van az egyszeri bejelentkezés több identitásszolgáltató között? | Nem | Nem | Nem | Igen |

Az Azure-bA a migrálás tervezése részeként kell határozza meg, hogyan lehet a legjobban integrálható a meglévő identity management és a felhőalapú identitás-szolgáltatások. Az alábbiakban gyakori integrációs forgatókönyvek.

### <a name="cloud-baseline"></a>Felhő-alapkonfiguráció

Nyilvános felhő platformok natív IAM rendszert biztosítanak a felhasználók és csoportok el felügyeleti funkciókat az. Ha a szervezet nem rendelkezik egy jelentős helyszíni identitáskezelési megoldás, és azt tervezi, hogy felhőalapú hitelesítési mechanizmusok való kompatibilitás számítási feladatok migrálása, kialakítson a személyazonosság-infrastruktúra használatával olyan natív felhőalapú identitásszolgáltatás.

**A felhő alapvető feltételezések**. Egy teljesen natív felhőalapú identitás-infrastruktúra használata feltételezi, hogy a következő:

- A felhőalapú erőforrások nem lesznek a helyszíni címtárszolgáltatások és az Active Directory-kiszolgálók függőségekkel rendelkeznek, vagy a számítási feladatok módosítható, hogy távolítsa el ezeket a függőségeket a.
- Az alkalmazás vagy szolgáltatás-munkaterhelések migrált felhőalapú identitás-szolgáltatóktól kompatibilis hitelesítési mechanizmusok támogatásához, vagy támogatja azokat könnyen módosítható. Felhőbeli natív Identitásszolgáltatók internetes használatra kész hitelesítési mechanizmusokkal, mint például a SAML, OAuth és OpenID Connect támaszkodnak. Meglévő számítási feladatok ettől az örökölt hitelesítési protokollok, mint például a Kerberos vagy NTLM használatával kell újratervezhetők lehet a felhőbe való migrálás előtt.

> [!TIP]
> A legtöbb natív felhőalapú identitás-szolgáltatások nem teljes helyettesítik a hagyományos helyszíni címtárak el. Címtárszolgáltatások, például a számítógép-felügyelet vagy a csoport házirend nem lehet elérhető a további eszközök vagy a szolgáltatások használata nélkül.

Teljes áttelepítés az identitás-szolgáltatások felhőalapú szolgáltatóra szükségtelenné teszi a saját identitás-infrastruktúrát, jelentősen leegyszerűsíti az informatikai felügyelet karbantartása.

### <a name="directory-synchronization"></a>A címtár-szinkronizálás

Egy meglévő identitás-infrastruktúrát használó szervezetek számára a címtár-szinkronizálás legtöbbször a legjobb megoldást a meglévő felhasználó- és hozzáférés-kezelés megőrzése szükséges IAM képességet biztosít a felhőbeli erőforrások kezelése során. Ez a folyamat folyamatosan a címtáradatok a felhő között replikálja, és a helyszíni környezetben, egyszeri bejelentkezés (SSO) engedélyezése a felhasználók és a egy egységes identitások, a szerepkör és a jogosultsági rendszert a teljes szervezet számára.

Megjegyzés: Office 365 elfogadó szervezetek már megvalósított [címtár-szinkronizálás](/office365/enterprise/set-up-directory-synchronization) azok a helyszíni Active Directory-infrastruktúrát és az Azure Active Directory között.

**Címtár-szinkronizálás feltételezések**. Szinkronizált identitáskezelési megoldások használata feltételezi, hogy a következő:

- Kell egy közös felhasználói fiókok és csoportok a felhőben és helyszíni informatikai infrastruktúra karbantartása.
- A helyszíni identitás-szolgáltatások támogatja a replikációt a felhőalapú identitás-szolgáltatónál.
- Egyszeri bejelentkezés mechanizmusok felhőbeli és helyszíni identitás-szolgáltatóktól hozzáférő felhasználók van szükség.

> [!TIP]
> Felhőalapú számítási feladatokat, amelyek az örökölt hitelesítési mechanizmusok, például az Azure ad-ben továbbra is szükséges a helyszíni tartományi szolgáltatásokhoz való kapcsolódás, vagy virtuális kiszolgálók a felhőalapú környezetben nem felhőalapú identitás-szolgáltatások által támogatott függenek. Ezek a szolgáltatások biztosítása. A helyszíni identitás-szolgáltatások használatával is a felhőbeli és helyszíni hálózat közötti kapcsolat függőségekkel mutatja be.

### <a name="cloud-hosted-domain-services"></a>Felhőben futó tartományi szolgáltatások

Ha munkaterheléseknek, amelyek a jogcímalapú hitelesítés örökölt protokollok, mint például a Kerberos vagy NTLM használatával, és azokat a munkaterheléseket nem refactored, modern hitelesítési protokollok, mint például a SAML vagy OAuth és OpenID Connect fogadására, szükség lehet áttelepítése néhány, a tartományi szolgáltatások a felhőbe, a felhőbeli üzemelő példány részeként.

A központi telepítési típus magában foglalja a felhő alapú virtuális hálózatok adja meg a tartományi szolgáltatások a felhőben lévő erőforrásokhoz az Active Directory futó virtuális gépek üzembe helyezéséről. Meglévő alkalmazások és szolgáltatások áttelepítése a felhőbeli hálózati használatát a felhőben üzemeltetett címtárkiszolgálók kisebb módosításokkal képesnek kell lennie.

Valószínű, hogy a meglévő címtárakban és a tartományi szolgáltatások továbbra is a helyszíni környezetben használható. Ebben a forgatókönyvben javasoljuk, hogy is használhatja a címtár-szinkronizálás közös biztosítja a felhasználók és szerepkörök a felhőben és a helyszíni környezetben.

**A felhőben tartományszolgáltatási feltételezések**. A következő könyvtár áttelepítés feltételezi:

- A számítási feladatok jogcímalapú hitelesítési protokollok, mint például a Kerberos vagy NTLM használatával függenek.
- A munkaterhelési virtuális gépek kell lennie a tartományhoz csatlakoztatott felügyeleti vagy alkalmazás számára az Active Directory csoport házirend céljából.

> [!TIP]
> Amíg a címtár áttelepítés ügyfélparancsfájl felhőben üzemeltetett a tartományi szolgáltatások biztosítják a nagyfokú rugalmasságot, amikor a számítási feladatokat migráló, ezeket a szolgáltatásokat biztosít a felhőbeli virtuális hálózaton belüli virtuális gépek futtatásához az informatikai RÉSZLEG összetettsége növelése felügyeleti feladatokat. A felhők kiforrottá migrálási folyamatba, vizsgálja meg ezeket a kiszolgálókat üzemeltető hosszú távú karbantartási követelményeit. Vegye figyelembe, hogy újrabontás meglévő számítási feladatok, például az Azure Active Directory felhőalapú identitás-szolgáltatóktól való kompatibilitás érdekében csökkentheti a felhőben üzemeltetett kiszolgálókon kell.

### <a name="active-directory-federation-services"></a>Active Directory összevonási szolgáltatások

Identitás-összevonási megbízhatósági kapcsolatok lehetővé teszik a gyakori hitelesítési és engedélyezési képességek többféle identity management rendszer hoz létre. Egyszeri bejelentkezési képességek majd több a szervezet vagy az ügyfelek vagy üzleti partnerek által kezelt identitáskezelési rendszerek több tartományban is támogatottak.

Az Azure AD támogatja a helyszíni Active Directory-tartományok használata az összevonási [Active Directory összevonási szolgáltatások](/azure/active-directory/hybrid/how-to-connect-fed-whatis) (AD FS). Tekintse meg a referencia-architektúra [az AD FS kiterjesztése az Azure-bA](../../../reference-architectures/identity/adfs.md) megtekintéséhez, hogy ez kialakítható az Azure-ban.

## <a name="evolving-identity-integration"></a>Identitás integrációs

Identitásintegráció iteratív folyamat. Előfordulhat, hogy szeretné elindítani a felhasználók és a egy kezdeti üzembe helyezéshez megfelelő szerepkörök kis számú natív felhőalapú megoldással. A migrálás kiforrottá, fontolja meg egy összevont modell bevezetése vagy a helyszíni identitás-szolgáltatások a felhőben, teljes könyvtárnévnek áttelepítés végrehajtásához. Nyissa meg újra az áttelepítési folyamat minden egyes ismétléskor identitás stratégiáját.

## <a name="learn-more"></a>Részletek

Lásd a következő identitás-szolgáltatások további információ az Azure platformon.

- [Az Azure AD](https://azure.microsoft.com/services/active-directory). Az Azure AD felhőalapú identitásszolgáltatásokat biztosít. Ez lehetővé teszi, hogy az Azure-erőforrások és a vezérlő Identitáskezelés, az eszköz regisztrálása, felhasználókiépítés, alkalmazás hozzáférés-vezérlés és az adatvédelem való hozzáférés kezelése.
- [Az Azure AD Connect](/azure/active-directory/hybrid/whatis-hybrid-identity). Az Azure AD Connect eszköz az Azure AD-példányban együttműködnek a meglévő identitáskezelési megoldások, lehetővé téve a felhőben a meglévő directory szinkronizálását teszi lehetővé.
- [Szerepköralapú hozzáférés-vezérlés](/azure/role-based-access-control/overview) (RBAC). Az Azure AD RBAC hatékony és biztonságos kezeléséhez a felügyeleti sík az erőforrásokhoz való hozzáférést biztosít. Feladatok és a felelősségi szerepkörök vannak szervezve, és ezek a szerepkörök felhasználók vannak hozzárendelve. Az RBAC lehetővé teszi ki és milyen műveleteket hajthat végre a felhasználó az erőforráson erőforrásokhoz való hozzáférést.
- [Azure AD Privileged Identity Management](/azure/active-directory/privileged-identity-management/pim-configure) (PIM). A PIM csökkenti az erőforrás-hozzáférési jogosultságok expozíciós idejének, és növeli a jelentéseket és riasztásokat keresztül használatuk átláthatóvá. Korlátozza a felhasználók jogosultságai a "csak az időben" véve (JIT), vagy a jogosultságok hozzárendelése a rövidebb időtartamot, utána jogosultságokkal automatikusan visszavonódnak.
- [A helyszíni Active Directory-tartományok integrálása az Azure Active Directory](../../../reference-architectures/identity/azure-ad.md). Ez a referenciaarchitektúra azt szemlélteti, címtár-szinkronizálás a helyszíni Active Directory-tartományok és az Azure AD között.
- [Az Active Directory Domain Services (AD DS) kiterjesztése az Azure-bA.](../../../reference-architectures/identity/adds-extend-domain.md) Ez a referenciaarchitektúra azt szemlélteti, AD DS-kiszolgálók bővítése domain servicesben a felhőalapú erőforrások üzembe helyezése.
- [Az Active Directory összevonási szolgáltatások (AD FS) kiterjesztése az Azure-bA](../../../reference-architectures/identity/adfs.md). Ez a referenciaarchitektúra konfigurálja az Active Directory összevonási szolgáltatások (AD FS) összevont hitelesítés és engedélyezés az Azure AD-címtárhoz való végrehajtásához.

## <a name="next-steps"></a>További lépések

Ismerje meg, hogyan valósíthat meg a házirend betartatása a felhőben.

> [!div class="nextstepaction"]
> [A házirend betartatása](../policy-enforcement/overview.md)
