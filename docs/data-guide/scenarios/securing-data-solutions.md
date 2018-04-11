---
title: Data-megoldások biztonságossá tétele
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 57897c31a8abdcd801874bf92d60360f7a80d1fa
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/14/2018
---
# <a name="securing-data-solutions"></a>Data-megoldások biztonságossá tétele

Számos a felhőben, így az adatok érhető el különösen akkor, ha működik a helyszíni adattárolókhoz, kizárólag az átállás okozhat egyes probléma, amely szerint ehhez az adatokat, majd a biztonság használandó új módszereket megnövekedett kisegítő.

## <a name="challenges"></a>Kihívásai

* A figyelés és a biztonsági események számos naplók tárolt elemzése központosítása.
* Végrehajtási titkosítási és engedélyezési felügyeleti az alkalmazások és szolgáltatások között.
* Győződjön meg arról, hogy a központi identity management works összes a megoldás-összetevő e helyszíni vagy a felhőben.

## <a name="data-protection"></a>Adatvédelem

Adatok védelmével kapcsolatos első lépése a védendő elemeket azonosítja. Törölje a jelet, egyszerű és pontos tájékoztatást adjon irányelveket azonosítása, védheti meg és figyelheti a legfontosabb adategységeket bárhol azok fejlesztéséhez. Eszközök, amelyek a szervezet kritikus vagy a nyereségességre nézve le aránytalanul nagy hatással lehet a legerősebb védelem létrehozásához. Ezek a nagy értékű eszközökhöz vagy HVAs nevezzük. Szigorú elemzést HVA életciklusának és biztonsági függőségi, és megfelelő biztonsági vezérlők és a feltételek létrehozásához. Ehhez hasonlóan azonosítása bizalmas eszközök és technológiák és biztonsági vezérlők alkalmazhatja folyamatok meghatározása.

Után meg kell védenie adatok azonosítása, fontolja meg, hogyan kíván védelemmel ellátni a adatok *aktívan* és az adatok *átvitel*.

* **Inaktív adat**: fizikai adathordozóra, hogy mágneses statikusan meglévő adatot vagy optikai lemezt, a helyszínen vagy a felhőben.
* **Adatokat átvitel közben**: miközben adatátvitel összetevők, a helyek vagy a programok, például több, mint a hálózati szolgáltatás között (a felhőbe helyszíni és fordítva) buszhoz között, vagy egy bemeneti/kimeneti folyamat során.

Inaktív vagy az átvitel során az adatok védelmének kapcsolatos további információkért lásd: [Azure Adatbiztonság és -titkosítás gyakorlati tanácsok](/azure/security/azure-security-data-encryption-best-practices).

## <a name="access-control"></a>Access Control

Központi helyet foglalnak el a felhőben tárolt adatainak védelme az Identitáskezelés és a hozzáférés-vezérlést. A különböző és a felhőszolgáltatások típusát, valamint a növekvő időben népszerűvé vált a [hibrid felhő](../scenarios/hybrid-on-premises-and-cloud.md), vannak-érdemes követnie, amikor a identitás és hozzáférés-vezérlés több kulcs eljárásokat:

* Az Identitáskezelés központosítása.
* Egyszeri bejelentkezés (SSO) engedélyezése.
* Jelszó management központi telepítését.
* Kényszeríteni a többtényezős hitelesítés (MFA) a felhasználók számára.
* Használjon szerepköralapú hozzáférés-vezérlést (RBAC).
* Feltételes hozzáférési házirendek úgy kell beállítani, amely a felhasználói identitás klasszikus fogalmát javítja a felhasználó helye, eszköztípus, javítási szintje és így tovább kapcsolatos további tulajdonságokkal.
* Vezérlő helye, ahol erőforrások jönnek létre resource manager használatával.
* Aktívan figyeli, hogy gyanús tevékenységek

További információkért lásd: [Azure Identitáskezelés és a hozzáférés szabályozásához ajánlott biztonsági eljárások](/azure/security/azure-security-identity-management-best-practices).

## <a name="auditing"></a>Naplózás

Az identitást és a figyelés azt már korábban említettük, a szolgáltatásokat és alkalmazásokat, amelyek használatával a felhő kell kell biztonsági eseményeket létrehozni, amely a figyelheti. Ezek az események figyelésével elsődleges feladat kezeli a naplókat, ahhoz, hogy a problémák elkerülése érdekében, és a múltbeli kiépítettektől eltérő hibakeresést mennyiségét. Az egyes felhőalapú alkalmazások, amelyek a legtöbb készítése a naplózási és telemetriai bizonyos fokú sok mozgó elemeket tartalmazhat. A központi és -elemző kezelése és a nagy mennyiségű információ értelmezhető használata.

További információkért lásd: [Azure-naplózás és a naplózási](/azure/security/azure-log-audit).



## <a name="securing-data-solutions-in-azure"></a>Az Azure data-megoldások biztonságossá tétele

### <a name="encryption"></a>Titkosítás

**Virtuális gépek**. Használjon [Azure Disk Encryption](/azure/security/azure-security-disk-encryption) a csatlakoztatott lemezek a Windows vagy Linux rendszerű virtuális gépek titkosításához. Ez a megoldás integrálódik az [Azure Key Vault](/azure/key-vault/) a lemez-titkosítási kulcsok és titkos kódokat, és szabályozhatja. 

**Azure Storage**. Használjon [Azure Storage szolgáltatás titkosítási](/azure/storage/common/storage-service-encryption) titkosítani az adatokat az Azure tárolás közben. Titkosítási, visszafejtési és kulcskezelés rendszer teljesen átlátható a felhasználók számára. Adatok is védve legyenek az átvitel során az Azure Key Vault ügyféloldali titkosítás használatával. További információkért lásd: [ügyféloldali titkosítás és a Microsoft Azure tárolás az Azure Key Vault](/azure/storage/common/storage-client-side-encryption).

**SQL-adatbázis** és **Azure SQL Data Warehouse**. Használjon [átlátható adattitkosítási](/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql) (TDE) valós idejű titkosítási és visszafejtési az adatbázist, a társított biztonsági másolatok és a tranzakciós naplófájlok végrehajtásához az alkalmazások módosítása nélkül. SQL-adatbázis is használható [mindig titkosítja](/azure/sql-database/sql-database-always-encrypted-azure-key-vault) ügyfél és kiszolgáló között, és miközben használatban van az adatok mozgása során a kiszolgálón, inaktív bizalmas adatok védelme érdekében. Az Azure Key Vault segítségével a mindig titkosított titkosítási kulcsok tárolásához. 

### <a name="rights-management"></a>Rights management

[Az Azure Rights Management](/information-protection/understand-explore/what-is-azure-rms) egy felhőalapú szolgáltatás, amely biztonságos fájlok és e-mailek titkosítási, identitáskezelési és engedélyezési házirendeket használ. Több eszköz is használható – telefonokon, táblagépeken és számítógépeken. Információk védhetők a szervezeten belül, mind a szervezeten kívüli mert is védettek maradnak az adatokat, akkor is, ha elhagyják a szervezet területét.

### <a name="access-control"></a>Hozzáférés-vezérlés

Használjon [szerepköralapú hozzáférés-vezérlés](/azure/active-directory/role-based-access-control-what-is) (RBAC) Azure-erőforrások való hozzáférés korlátozása a felhasználói szerepkörök alapján. Ha a helyszíni Active Directory használ, akkor [szinkronizálása az Azure ad-val](/azure/active-directory/active-directory-hybrid-identity-design-considerations-directory-sync-requirements) biztosít a felhasználók a helyszíni identitás alapján felhő megadásával.

Használjon [feltételes hozzáférés az Azure Active Directoryban](/azure/active-directory/active-directory-conditional-access-azure-portal) szabályozza a hozzáférést a megadott feltételek alapján a környezetben futó alkalmazások érvényesítését. Például a házirend-utasítás sikerült formájában: _alvállalkozói megpróbált hozzáférni a felhőalapú alkalmazások, amelyek nem megbízható hálózatokon, ha majd blokkolja a hozzáférést_. 

[Az Azure AD Privileged Identity Management](/azure/active-directory/active-directory-privileged-identity-management-configure) kezelése, szabályozása és figyelése, és milyen típusú feladatok vannak végrehajtásához a rendszergazda jogosultságokkal a felhasználók segítséget. Ez egy fontos lépés korlátozza, aki a szervezet végezhet el a jogosultságokhoz kötött műveletek az Azure ad-ben, az Azure, az Office 365 vagy az SaaS-alkalmazások esetén figyelőjével továbbá a hozzájuk tartozó tevékenység.

### <a name="network"></a>Network (Hálózat)

Az adatok védelmére átvitel, mindig használjon SSL/TLS amikor adatcsere különböző helyek között. Előfordulhat, hogy a teljes kommunikációs csatornát a helyszíni közötti elkülönítése, és a felhőalapú infrastruktúra használatával vagy a virtuális magánhálózat (VPN) kell vagy [ExpressRoute](/azure/expressroute/). További információkért lásd: [kiterjesztése a felhőbe a helyszíni megoldás](../scenarios/hybrid-on-premises-and-cloud.md).

Használjon [hálózati biztonsági csoportok](/azure/virtual-network/virtual-networks-nsg) (NSG-k) a potenciális támadási számának csökkentése érdekében. A hálózati biztonsági csoport egy biztonsági szabályokból álló listát tartalmaz, amelyek engedélyezik vagy megtagadják a bejövő vagy kimenő hálózati forgalmat a forrás vagy a cél IP-címe, illetve portok vagy protokollok alapján. 

Használjon [virtuális hálózati Szolgáltatásvégpontok](/azure/virtual-network/virtual-network-service-endpoints-overview) az Azure SQL- vagy Azure tárolási erőforrások biztonságba helyezése, hogy csak a virtuális hálózati forgalmát férhetnek hozzá ezekhez az erőforrásokhoz.

Virtuális gépek az Azure Virtual Network (VNet) belül biztonságosan kommunikálhat más Vnetekről használatával [virtuális hálózati társviszony-létesítés](/azure/virtual-network/virtual-network-peering-overview). A társított virtuális hálózatok közti hálózati adatforgalom nem nyilvános. A virtuális hálózatok közötti forgalom a Microsoft gerinchálózatán belül marad.

További információkért lásd: [Azure hálózati biztonság](/azure/security/azure-network-security)

### <a name="monitoring"></a>Figyelés

[Az Azure Security Center](/azure/security-center/security-center-intro) automatikusan gyűjti, elemzi és integrálja az Azure-erőforrások, a hálózati és az összekapcsolt partneri megoldások, például a tűzfal megoldások valós fenyegetések észlelése és a vakriasztások számának csökkentése érdekében naplóadatait. 

[Naplófájl Analytics](/azure/log-analytics/log-analytics-overview) központosított hozzáférést biztosít a a naplók és segítségével elemezheti az adatokat, és hozzon létre egyéni riasztások.

[Az Azure SQL adatbázis Fenyegetésészlelés](/azure/sql-database/sql-database-threat-detection) észleli a rendellenes tevékenységek jelző szokatlan és potenciálisan káros kísérletek eléréséhez, vagy adatbázisok kihasználására. Biztonsági tisztviselő vagy más kijelölt rendszergazdák adatbázis gyanús tevékenységekről az azonnali értesítések fogadására azok bekövetkezésekor. Minden értesítés részletesen bemutatja a gyanús tevékenység, illetve további vizsgálata, és a fenyegetések mérséklésére javasolja.


