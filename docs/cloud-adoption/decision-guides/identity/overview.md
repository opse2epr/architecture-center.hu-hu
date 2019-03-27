---
title: 'CAF: Identitáskezelés döntési – útmutató'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: További információ a identitás Azure áttelepítések core szolgáltatás.
author: rotycenh
ms.openlocfilehash: 511e27c7d63968417a7eb47764d2e7768496d34f
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298674"
---
# <a name="identity-decision-guide"></a>Identitáskezelés döntési – útmutató

Bármilyen környezet akár helyi, hibrid, vagy csak felhőalapú, IT-igényeket is szabályozhatja, mely a rendszergazdák, a felhasználók és csoportok hozzáféréssel rendelkezik az erőforrásokhoz. Identitás és hozzáférés-kezelés (IAM) szolgáltatás lehetővé teszi a felhőbeli hozzáférés-vezérlés kezelése.

![Küldik az ábrázolást legalább az identitásbeállításokat a legösszetettebb, az alábbi hivatkozások a jump igazítva](../../_images/discovery-guides/discovery-guide-identity.png)

Ugrás ide: [Identity Integration követelmények meghatározása](#determine-identity-integration-requirements) | [Felhőbeli natív](#cloud-baseline) | [címtár-szinkronizálás](#directory-synchronization) | [a felhőben tartományi szolgáltatások](#cloud-hosted-domain-services) | [Active Directory összevonási szolgáltatások](#active-directory-federation-services) | [identitás integrációs](#evolving-identity-integration)  |  [További](#learn-more)

Többféleképpen is Identitáskezelés a felhőalapú környezetekben, amelyek eltérőek lehetnek a költségek és munkaráfordítás alól. A felhőalapú identitás-szolgáltatások strukturálja kulcsfontosságú tényező az integráció a meglévő helyszíni identitás-infrastruktúrát a szükséges.

Az Azure-ban az Azure Active Directory (Azure AD) felhőalapú erőforrások hozzáférés-vezérléshez és identitáskezeléshez kezelés alapszinten biztosít. Azonban ha a szervezet Active Directory (AD) infrastruktúra rendelkezik olyan összetett erdő struktúrájába vagy testre szabott szervezeti egységhez (OU), a felhőalapú számítási feladatokhoz szükség lehet az Azure AD-címtár-szinkronizálás identitások konzisztens halmazát csoportok és szerepkörök között a helyszíni és felhőalapú környezetekben. Emellett függ az örökölt hitelesítési mechanizmusok alkalmazások támogatása az Active Directory Domain Services (AD DS) a felhőben telepítését lehet szükség.

## <a name="determine-identity-integration-requirements"></a>Identity integration követelmények meghatározása

| Kérdés | Felhő-alapkonfiguráció | A címtár-szinkronizálás | Cloud-hosted Domain Services | AD összevonási szolgáltatások |
|------|------|------|------|------|
| Jelenleg nincs egy helyi címtárszolgáltatás? | Igen | Nem | Nem | Nem |
| A számítási feladatokhoz szükséges használata a felhasználók és csoportok a felhőben és helyszíni környezetben között? | Nem | Igen | Nem | Nem |
| Hajtsa végre a számítási feladatok függ az örökölt hitelesítési mechanizmusok, például a Kerberos vagy NTLM? | Nem | Nem | Igen | Igen |
| Szükség van az egyszeri bejelentkezés több identitásszolgáltató között? | Nem | Nem | Nem | Igen |

Az Azure-bA a migrálás tervezése részeként kell határozza meg, hogyan lehet a legjobban integrálható a meglévő identity management és a felhőalapú identitás-szolgáltatások. Az alábbiakban gyakori integrációs forgatókönyvek.

### <a name="cloud-baseline"></a>Felhő-alapkonfiguráció

Azure ad-ben, a natív identitás- és hozzáférés-kezelés (IAM) rendszer felhasználók és csoportok el az felügyeleti funkciókat az Azure platformon. Ha a szervezet nem rendelkezik egy jelentős helyszíni identitáskezelési megoldás, és azt tervezi, hogy felhőalapú hitelesítési mechanizmusok való kompatibilitás számítási feladatok migrálása, gondoskodjon az Ön identitáskezelési infrastruktúráját az Azure AD alapként fejlesztéséhez.

**A felhő alapvető feltételezések**. Egy teljesen natív felhőalapú identitás-infrastruktúra használata feltételezi, hogy a következő:

- A felhőalapú erőforrások nem lesznek a helyszíni címtárszolgáltatások és az Active Directory-kiszolgálók függőségekkel rendelkeznek, vagy távolítsa el ezeket a függőségeket a számítási feladatok módosíthatók.
- Az alkalmazás vagy szolgáltatás-munkaterhelések migrált kompatibilis az Azure AD-hitelesítési mechanizmusok támogatásához, vagy támogatja azokat könnyen módosítható. Az Azure AD csak internetes használatra kész hitelesítési mechanizmusokkal, mint például a SAML, OAuth és OpenID Connect támaszkodik. Meglévő számítási feladatok ettől az örökölt hitelesítési protokollok, mint például a Kerberos vagy NTLM használatával kell újratervezhetők kell a felhő alapvető minta használatával a felhőbe való migrálás előtt.

> [!TIP]
> Teljes egészében az Azure AD-ba való migrálás az identitás-szolgáltatások szükségtelenné teszi a saját identitás-infrastruktúrát, jelentősen leegyszerűsíti az informatikai felügyelet karbantartása.
>
> Az Azure AD viszont nem egy hagyományos helyszíni Active Directory-infrastruktúra teljes értékű alternatívájaként. Címtárszolgáltatások, például az örökölt hitelesítési módszereket, a számítógép-kezelés vagy a csoportházirend nem lehet elérhető a felhőbeli üzembe helyezés további eszközöket vagy szolgáltatások nélkül.
>
> Forgatókönyvek, ahol meg kell integrálhatja a helyszíni identitások vagy a domain services és a magánfelhők számára, lásd: a címtár-szinkronizálás és a felhőben üzemeltetett domain services-minták az alábbiak ismertetik.

### <a name="directory-synchronization"></a>A címtár-szinkronizálás

A szervezet számára a meglévő helyszíni Active Directory infrastruktúrával a címtár-szinkronizálás legtöbbször a legjobb megoldást a meglévő felhasználó- és hozzáférés-kezelés megőrzése szükséges IAM képességet biztosít a felhőbeli erőforrások kezelése során. Ez a folyamat folyamatosan az Azure AD között replikálja a címtáradatok és a helyszíni címtárszolgáltatások, lehetővé teszi a közös hitelesítő adatokat a felhasználók és a egy egységes identitások, a szerepkör és a jogosultsági rendszert a teljes szervezet számára.

Megjegyzés: Office 365 elfogadó szervezetek már megvalósított [címtár-szinkronizálás](/office365/enterprise/set-up-directory-synchronization) azok a helyszíni Active Directory-infrastruktúrát és az Azure Active Directory között.

**Címtár-szinkronizálás feltételezések**. Szinkronizált identitáskezelési megoldások használata feltételezi, hogy a következő:

- Kell egy közös felhasználói fiókok és csoportok a felhőben és helyszíni informatikai infrastruktúra karbantartása.
- A helyszíni identitás-szolgáltatások támogatják az Azure AD-replikációt.

> [!TIP]
> Felhőalapú számítási feladatokat, amelyek a helyszíni Active Directory-kiszolgálók által biztosított örökölt hitelesítési mechanizmusok függenek, és nem támogatottak az Azure AD a helyszíni tartományi szolgáltatásokhoz való kapcsolódás, vagy a virtuális kiszolgálók továbbra is szükséges a Ezek a szolgáltatások nyújtásához a felhőalapú környezetben. A helyszíni identitás-szolgáltatások használatával is a felhőbeli és helyszíni hálózat közötti kapcsolat függőségekkel mutatja be.

### <a name="cloud-hosted-domain-services"></a>Felhőben futó tartományi szolgáltatások

Ha munkaterheléseknek, amelyek a jogcímalapú hitelesítés örökölt protokollok, mint például a Kerberos vagy NTLM használatával, és azokat a munkaterheléseket nem refactored, modern hitelesítési protokollok, mint például a SAML vagy OAuth és OpenID Connect fogadására, szükség lehet áttelepítése néhány, a tartományi szolgáltatások a felhőbe, a felhőbeli üzemelő példány részeként.

Ez a minta magában foglalja a felhő alapú virtuális hálózataihoz adja meg az Active Directory Domain Services (AD DS) a felhőben lévő erőforrások Active Directory futó virtuális gépek üzembe helyezéséről. Meglévő alkalmazások és szolgáltatások áttelepítése a felhőbeli hálózati kell tudni használni ezeket a felhőben üzemeltetett címtárkiszolgálók kisebb módosításokkal.

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
