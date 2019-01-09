---
title: Adatmegoldások védelme
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.openlocfilehash: 47e3be2afd14d980b98ac9659f7f1e5a4df3403f
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/08/2019
ms.locfileid: "54110875"
---
# <a name="securing-data-solutions"></a>Adatmegoldások védelme

Több a felhőben, így az adatok elérhető különösen akkor, ha működik a helyszíni adattárolókból; kizárólag az átállás okozhat néhány probléma, amely nagyobb kisegítő lehetőségek, hogy az adatok és új módon biztonságossá tenni.

## <a name="challenges"></a>Problémák

- A figyelési és elemzési számos naplókat tárolja a biztonsági események központosítása.
- A végrehajtási titkosítási és engedélyezési felügyeleti alkalmazásai és szolgáltatásai között.
- Annak biztosítása e összes a megoldás-összetevőket, központosított, amely identitás-felügyeleti funkcióit a helyszíni vagy a felhőben.

## <a name="data-protection"></a>Adatvédelem

Az első lépés az adatok védelmével, hogy mit szeretne védeni azonosítja. Egyértelmű, egyszerű és pontos tájékoztatást adjon a szabályokat, azonosítása, védheti meg és figyelheti a legfontosabb adategységek bárhol azok fejleszthet. Az eszközök, amelyek aránytalanul nagy hatással lehet a szervezet üzleti szempontból alapvető vagy nyereségességét a legerősebb védelem létesíteni. Ezek a nagy értékű eszközöket, vagy HVAs nevezzük. Az életciklus-kezelés és a biztonsági függőségek HVA szigorú elemzéseket végezhet, és megfelelő biztonsági vezérlők és feltételek. Ehhez hasonlóan azonosítása és besorolása az érzékeny eszközökre, és meghatározott technológiákat és folyamatokat automatikus alkalmazásához a biztonsági vezérlőket.

Miután az adatok védelme érdekében szüksége lett azonosítva, fontolja meg, hogyan fogja megvédeni az adatokat *inaktív* és az adatok *átvitel*.

- **Az inaktív adatok**: A fizikai adathordozón, hogy mágneses statikusan meglévő adatot vagy optikai lemezt, a helyszínen vagy a felhőben.
- **Az átvitt adatok**: Miközben adatátvitel összetevők, helyeket vagy programok, például több, mint a hálózati szolgáltatás között (a felhőbe helyszíni és fordítva) bus között, vagy egy bemeneti/kimeneti folyamat során.

Inaktív és átvitel közben az adatok védelmének kapcsolatos további információkért lásd: [Azure által nyújtott Adatbiztonság és titkosítás ajánlott eljárások](/azure/security/azure-security-data-encryption-best-practices).

## <a name="access-control"></a>Hozzáférés-vezérlés

A felhőbeli adatok védelmére központ az Identitáskezelés és hozzáférés-vezérlés kombinációját. A különböző és a felhőszolgáltatások, valamint a növekvő népszerűsége [hibrid felhő](../scenarios/hybrid-on-premises-and-cloud.md), nincsenek számos kulcsfontosságú eljárások esetén identitás és hozzáférés-vezérlés, kövesse:

- Az Identitáskezelés központosítása.
- Egyszeri bejelentkezés (SSO) engedélyezése.
- Jelszókezelés üzembe helyezése.
- Kényszeríteni a többtényezős hitelesítés (MFA) a felhasználók számára.
- Használata szerepköralapú hozzáférés-vezérlés (RBAC).
- Feltételes hozzáférési szabályzatok kell konfigurálni, amely javítja a felhasználói identitást a klasszikus fogalmát további tulajdonságokkal kapcsolatos felhasználói hely, eszköztípus, előírt biztonsági javítási szintje és így tovább.
- Szabályozhatja a helyek, ahol jönnek létre erőforrások resource manager használatával.
- Aktív monitorozása a gyanús tevékenységek esetén

További információkért lásd: [Azure identitáskezelési és hozzáférés-vezérlés ajánlott biztonsági eljárások](/azure/security/azure-security-identity-management-best-practices).

## <a name="auditing"></a>Naplózás

Az identitás- és hozzáférés figyelése a fent említettük, nem a szolgáltatásokat és alkalmazásokat a felhőben használt kell kell létrehozni a biztonsággal kapcsolatos eseményeket, amelyek segítségével figyelheti. Az elsődleges vonatkozó kérdést állít be ezen események figyelése annak érdekében, hogy az esetleges problémák elkerülése érdekében, vagy korábbi kiépítettektől hibaelhárítása a naplók mennyiségét kezeli. Felhőalapú alkalmazások általában a részek, amely a legtöbb készítése a naplózás és a telemetria bizonyos fokú tartalmaznak. Segítségével központi megfigyelési és elemzési kezelését és a nagy mennyiségű olyan információt, jelentéssel bírnak.

További információkért lásd: [Azure-naplózás és a naplózási](/azure/security/azure-log-audit).

## <a name="securing-data-solutions-in-azure"></a>Adatmegoldások védelme az Azure-ban

### <a name="encryption"></a>Titkosítás

**Virtuális gépek**. Használat [az Azure Disk Encryption](/azure/security/azure-security-disk-encryption) a Windows vagy Linux rendszerű virtuális gépekhez csatolt lemezek titkosítása. Ez a megoldás integrálódik [Azure Key Vault](/azure/key-vault/) ellenőrzése és a lemeztitkosítási kulcsokat és titkos kódokat.

**Azure Storage**. Használat [Azure Storage Service Encryption](/azure/storage/common/storage-service-encryption) automatikusan az Azure Storage-ban az inaktív adatok titkosításához. Titkosítási, visszafejtési és kulcskezelési teljes mértékben átlátható a felhasználók számára. Adatok is lehetővé teszi az átvitel során az ügyféloldali titkosítás az Azure Key Vault segítségével. További információkért lásd: [ügyféloldali titkosítás és a Microsoft Azure Storage for Azure Key Vault](/azure/storage/common/storage-client-side-encryption).

**Az SQL Database** és **Azure SQL Data Warehouse**. Használat [transzparens adattitkosítás](/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql) (TDE) az alkalmazások módosítása nélkül az adatbázisok, az azokhoz kapcsolódó biztonsági mentési és a tranzakciós naplófájlokat az valós idejű titkosítását és visszafejtését végrehajtásához. Az SQL Database segítségével is [Always Encrypted](/azure/sql-database/sql-database-always-encrypted-azure-key-vault) során adatátviteli ügyfél és kiszolgáló közötti, és az adatok használata közben a kiszolgálón, az inaktív bizalmas adatok védelme érdekében. Az Azure Key Vault használatával az Always Encrypted titkosítási kulcsok tárolásához.

### <a name="rights-management"></a>Rights management

[Az Azure Rights Management](/information-protection/understand-explore/what-is-azure-rms) egy felhőalapú szolgáltatás, amely biztonságos fájlok és e-mailek titkosítási, identitáskezelési és engedélyezési házirendeket használ. Több eszközön működik – például telefonokon, táblagépeken és számítógépeken. Információk védelme biztosítható a szervezeten belül, mind a szervezeten kívüli mert is védettek maradnak az adatokat, akkor is, ha elhagyják a szervezet területét.

### <a name="access-control"></a>Hozzáférés-vezérlés

Használat [szerepköralapú hozzáférés-vezérlés](/azure/active-directory/role-based-access-control-what-is) (RBAC) az Azure-erőforrásokhoz való hozzáférés korlátozása a felhasználói szerepkörök alapján. Ha a helyszíni Active Directory használ, [szinkronizálja az Azure AD-val](/azure/active-directory/active-directory-hybrid-identity-design-considerations-directory-sync-requirements) biztosít egy felhőalapú identitás-azonosítót fog kérni a helyszíni alapján.

Használat [feltételes hozzáférés az Azure Active Directory](/azure/active-directory/active-directory-conditional-access-azure-portal) az alkalmazásokhoz a környezetben a meghatározott feltételek alapján való hozzáférést a vezérlők kényszerítésére. Ha például a házirendutasítás eltarthat formájában: _Amikor alvállalkozók elérni kívánt a felhőalapú alkalmazások a nem megbízható hálózatokhoz, majd letiltja a hozzáférést_.

[Az Azure AD Privileged Identity Management](/azure/active-directory/active-directory-privileged-identity-management-configure) segítségével szabályozhatja, kezelhetők és figyelheti a felhasználók és milyen feladatokat számos vannak végrehajtása a rendszergazdai jogosultságokat. Ez egy fontos lépés a vállalatnál ki is végrehajthasson privilegizált műveleteket az Azure AD-ben korlátozza az Azure, Office 365-höz, vagy SaaS-alkalmazások, valamint figyelő tevékenységeik.

### <a name="network"></a>Network (Hálózat)

Az átvitt adatok védelme érdekében mindig használja az SSL/TLS különböző helyek közötti adatcsere során. Előfordulhat, hogy a teljes kommunikációs csatornát a helyszíni közötti azonosíthatók, és virtuális magánhálózati (VPN) felhőalapú infrastruktúra használatával kell vagy [ExpressRoute](/azure/expressroute/). További információkért lásd: [a felhő helyszíni Adatmegoldások kiterjesztése](../scenarios/hybrid-on-premises-and-cloud.md).

Használat [hálózati biztonsági csoportok](/azure/virtual-network/virtual-networks-nsg) (NSG-ket) a potenciális támadási vektorok számának csökkentése. A hálózati biztonsági csoport egy biztonsági szabályokból álló listát tartalmaz, amelyek engedélyezik vagy megtagadják a bejövő vagy kimenő hálózati forgalmat a forrás vagy a cél IP-címe, illetve portok vagy protokollok alapján.

Használat [virtuális hálózati Szolgáltatásvégpontok](/azure/virtual-network/virtual-network-service-endpoints-overview) védheti meg Azure Storage vagy az Azure SQL-erőforrásokat, így csak a virtuális hálózatból érkező forgalom is elérheti ezeket az erőforrásokat.

Az Azure Virtual Network (VNet) belüli virtuális gépek biztonságosan kommunikálhat más virtuális hálózatok használatával [virtuális hálózatok közötti társviszony](/azure/virtual-network/virtual-network-peering-overview). A társított virtuális hálózatok közti hálózati adatforgalom nem nyilvános. A virtuális hálózatok közötti forgalom a Microsoft gerinchálózatán belül marad.

További információkért lásd: [Azure hálózati biztonság](/azure/security/azure-network-security)

### <a name="monitoring"></a>Figyelés

[Az Azure Security Center](/azure/security-center/security-center-intro) automatikusan gyűjti, elemzi és integrálja az Azure-erőforrások, a hálózati és a csatlakoztatott partneri megoldások, például tűzfal-megoldások, a valós fenyegetések észlelése és csökkenti a vakriasztások naplóadatait.

[Log Analytics](/azure/log-analytics/log-analytics-overview) a naplók központosított hozzáférést biztosít, és segítségével elemezheti az adatokat, és egyéni riasztásokat is létrehozhat.

[Az Azure SQL Database Threat Detection](/azure/sql-database/sql-database-threat-detection) észleli az adatbázisokat elérni vagy kiaknázni a szokatlan és vélhetően kárt okozó kísérleteket jelző rendellenes tevékenységek. Biztonsági tisztviselők, vagy más kijelölt rendszergazdák a bekövetkezésük kaphatnak a gyanús adatbázis-tevékenységekről az azonnali értesítések. Minden értesítés biztosít a gyanús tevékenység részleteit, és hogyan további vizsgálata és a fenyegetés javasolja.
