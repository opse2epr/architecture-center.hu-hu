---
title: Összevont identitás
description: Egy külső identitásszolgáltatótól delegált hitelesítés.
keywords: Kialakítási mintája
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- security
ms.openlocfilehash: a1edbdd080309383201d33e73602e2f18928c080
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="federated-identity-pattern"></a>Összevont identitás-minta

[!INCLUDE [header](../_includes/header.md)]

Egy külső identitásszolgáltatótól delegált hitelesítés. Ez fejlesztési egyszerűsítése, minimalizálása érdekében a felhasználókezelés kapcsolatos követelmények és az alkalmazás a felhasználói élmény javításához.

## <a name="context-and-problem"></a>A környezetben, és probléma

Felhasználók általában kell az üzleti kapcsolattal rendelkeznek több alkalmazáshoz megadott és a különböző szervezetek üzemelteti. Ezek a felhasználók előfordulhat, hogy kell minden egyes megadott (és más) hitelesítő adatokat használja. Ez a következőket teheti:

- **Egy különálló felhasználói felület miatt**. Felhasználók gyakran elfelejti bejelentkezési hitelesítő adataival, ha sok különböző néhányat a meglévők közül.

- **Biztonsági rések elérhetővé**. Amikor egy felhasználó elhagyja a vállalatot a fiók azonnal kell megszüntetett. Akkor is könnyen eltéveszthetők ezt a nagy méretű szervezeteknek.

- **Felhasználókezelés nehezíti**. Rendszergazdák kell a felhasználói hitelesítő adatok kezelése és például a jelszó-emlékeztető kezeléséről további feladatok elvégzéséhez.

Felhasználók általában inkább ugyanazokat a hitelesítő adatokat használja ezeket az alkalmazásokat.

## <a name="solution"></a>Megoldás

Olyan hitelesítési mechanizmus, amellyel összevont identitás megvalósításához. Külön felhasználói hitelesítést az alkalmazáskódban, és delegálja a hitelesítés a megbízható identitásszolgáltató. Ez leegyszerűsítheti a fejlesztési és engedélyezése a felhasználók hitelesítése az identitás-szolgáltatóktól (IdP) szélesebb körének elvégezhető ugyanakkor minimalizálja az adminisztratív terhek. Lehetővé teszi egyértelműen absztrakcióhoz engedélyezési hitelesítést.

A megbízható Identitásszolgáltatók közé tartozik a vállalati könyvtárak, a helyszíni összevonási szolgáltatások, más biztonsági jogkivonat (STS) által nyújtott szolgáltatások üzleti partnerek vagy a közösségi Identitásszolgáltatók, amelyek felhasználókat hitelesítheti rendelkező, például egy Microsoft Google, Yahoo!-s vagy Facebook-fiókkal.

Ábra bemutatja az összevont identitási mintát, amikor egy ügyfél-alkalmazás kell elérni egy szolgáltatást, amelyhez hitelesítés szükséges. Az IdP, hogy az STS szolgáltatással együttműködik a hitelesítés történik. Az IdP, amelyek információval szolgálnak a hitelesített felhasználó biztonsági jogkivonatokat bocsát ki. Néven jogcímek, az információ magában foglalja a felhasználó identitását, és más információkat, például a szerepköri tagság és részletesebb hozzáférési jogosultsággal is tartalmazhatja.

![Összevont hitelesítés áttekintése](./_images/federated-identity-overview.png)


Ez a modell jogcímalapú hozzáférés-vezérlés gyakran nevezik. Alkalmazások és szolgáltatások szolgáltatásait és funkcióit a token lévő jogcímek alapján történő hozzáférés engedélyezésére. A szolgáltatás, amelyhez hitelesítés szükséges a kiállító terjesztési hely megbízhatónak kell lennie. Az ügyfélalkalmazás kapcsolatba lép a kiállító terjesztési hely, amely végrehajtja a hitelesítést. Ha a hitelesítés sikeres, a kiállító terjesztési hely a jogcímeket, amely azonosítja a felhasználót, hogy az STS (vegye figyelembe, hogy a kiállító terjesztési hely és STS lehet ugyanazt a szolgáltatást) tartalmazó jogkivonatot ad vissza. Az STS átalakíthatja és kiegészítheti a jogcímeket a jogkivonatban, előre meghatározott szabályok alapján, az ügyfélnek való visszaküldés előtt. Az ügyfélalkalmazás majd továbbíthatja a jogkivonat a szolgáltatás, magát.

> A megbízhatósági lánc további STSs lehet. Például a forgatókönyvben a későbbiekben olvashat, egy a helyszíni STS bizalmi kapcsolatok egy másik STS fér hozzá a felhasználó hitelesítésére identitásszolgáltató felelős. A megoldás közös vállalati környezetben, ahol van egy helyszíni STS és könyvtár.

Összevont hitelesítés identitások megbízó különböző tartományok között kiadására szabványalapú megoldást nyújt, és egyszeri bejelentkezést támogatja. Azt egyre gyakrabban, alkalmazások, különösen felhőben üzemeltetett alkalmazások, az összes típusa, mert identitás-szolgáltatóktól közvetlen hálózaton keresztül nélkül egyszeri bejelentkezést támogatja. A felhasználó hitelesítő adatainak megadását minden alkalmazás nem rendelkezik. Ez növeli a biztonságot, mivel megakadályozza, hogy a különböző alkalmazások eléréséhez szükséges hitelesítő adatok létrehozása, és a felhasználó hitelesítő adatait az eredeti identitásszolgáltató kivételével az összes is elrejti. Alkalmazások információk csak a hitelesített identitást lévő a jogkivonat.

Összevont identitás is rendelkezik a legnagyobb előnye, hogy az identitás- és hitelesítő adatok kezelése az identity provider felelősségét-e. Az alkalmazás vagy szolgáltatás nem szükséges felügyeleti funkciókat-azonosítót fog kérni. Emellett a vállalati környezetben, a vállalati címtárban nem kell tudni a felhasználó ha megbízhatónak fogja tekinteni az identitásszolgáltató. Ezzel eltávolítja az összes adminisztratív terhek a címtáron belül a felhasználói identitás kezelése.

## <a name="issues-and-considerations"></a>Problémákat és szempontok

Alkalmazások, amelyek megvalósítják az összevont hitelesítés tervezésekor vegye figyelembe a következőket:

- A hitelesítés történhet a hibaérzékeny pontok kialakulását. Ha több adatközpontot az alkalmazást telepít központilag, fontolja meg, az alkalmazás megbízhatósági és rendelkezésre állás fenntartása azonos adatközpontokban történő telepítésének az identity management mechanizmus.

- Hitelesítési eszközök a szerepkör jogcím szerepel a hitelesítési jogkivonat-alapú hozzáférés-vezérlés konfigurálása lehetővé teszik. Ezt gyakran nevezik szerepköralapú hozzáférés-vezérlést (RBAC), és engedélyezheti, hogy a szolgáltatások és az erőforrásokhoz való hozzáférés ellenőrzése részletesebb felügyeletét.

- Ellentétben a vállalati címtárban szolgáltatáson alapuló Jogcímalapú hitelesítés használata a közösségi identitás-szolgáltatóktól általában egy e-mail címet, és lehet, hogy a nem hitelesített felhasználó adatainak nem megadása. Néhány közösségi Identitásszolgáltatók, például egy Microsoft-fiókot, adja meg a csak egy egyedi azonosítót. Az alkalmazás általában kell néhány regisztrált felhasználók adatainak megőrzése, és ezeket az adatokat az azonosító szerepel a jogkivonatában lévő jogcímeket egyező lesz. Általában ez segítségével történik regisztrációs a felhasználó általi első elérésének a alkalmazást, és információt van majd be a nézetmodellbe, a jogkivonat további jogcímekként minden hitelesítés után.

- Ha nincs konfigurálva az STS egynél több identitásszolgáltató, mely identitásszolgáltató, a felhasználó a hitelesítéshez a rendszer átirányítja azt kell észleli. A folyamat elnevezése a hitelesítőtartomány feltárását. Lehet, hogy az STS ehhez automatikusan egy e-mail cím vagy a felhasználó neve, amely a felhasználót, olyan altartomány, az alkalmazást, amely a felhasználó fér hozzá, a felhasználó IP-hatókört, vagy tárolja a felhasználó böngészőben a cookie tartalma alapján. Például, ha a felhasználó által megadott e-mail cím a Microsoft a tartományban, például a user@live.com, az STS átirányítja a felhasználót a Microsoft-fiókja bejelentkezési oldalára. A későbbi látogatások alkalmával az STS egy cookie-t használhatja annak jelzésére, hogy volt-e az utolsó bejelentkezés Microsoft-fiókkal. Automatikus észlelés nem határozható meg a hitelesítőtartomány, ha az STS megjeleníti a hitelesítőtartomány feltárási lapjának, amely tartalmazza a megbízható identitás-szolgáltatóktól, és a felhasználó használni kívánja azt kell választania.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes használni ezt a mintát

Ez a kialakítás például akkor hasznos, ha forgatókönyvek:

- **Egyszeri bejelentkezés a vállalati**. Ebben a forgatókönyvben kell hitelesíteni az alkalmazottakat, anélkül, hogy azokat bejelentkezni, minden alkalommal, amikor egy alkalmazás, amikor meglátogatják a vállalati biztonsági határán kívüli a felhőben üzemeltetett vállalati alkalmazások esetében. A felhasználói élmény ugyanaz, mint a helyszíni alkalmazások használata, ha azok még hitelesített a vállalati hálózathoz történő bejelentkezéskor, és ezt követően hozzáféréssel rendelkezzenek az összes releváns alkalmazásokhoz anélkül, hogy jelentkezzen be újra.

- **Összevont identitás több partnerekkel**. Ebben a forgatókönyvben kell hitelesíti az alkalmazottak és az üzleti partnerek, akik nem rendelkeznek fiókkal a vállalati címtárban. Ez a közös az olyan vállalatok alkalmazások, alkalmazások, amely harmadik féltől származó szolgáltatással integrálható, valamint arról, hogy a vállalatok különböző informatikai rendszerek ahol kell egyesíteni vagy megosztott erőforrások.

- **Összevont identitás SaaS-alkalmazásokhoz az**. Ebben a forgatókönyvben független szoftvergyártók kiszolgálása egy használatra kész több bérlők vagy ügyfelek számára. Mindegyik bérlő megfelelő identitásszolgáltató használatával hitelesít. Például üzleti felhasználók fogja használni a vállalati hitelesítő adatok, amíg a fogyasztók és az ügyfelek a bérlő a közösségi identitás hitelesítő adatait fogja használni.

Ez a minta nem lehet a következő esetekben lehet hasznos:

- Az alkalmazás minden felhasználó hitelesítése egy identitásszolgáltatótól, és nincs szükség semmilyen más identitásszolgáltató hitelesítést. Ez az üzleti alkalmazásokat, amelyek használják a vállalati címtárban (elérhető az alkalmazásban) a hitelesítéshez, egy VPN-kapcsolattal, vagy (a felhőben üzemeltetett forgatókönyvek) a helyszíni címtár közötti virtuális hálózati kapcsolaton keresztül a tipikus és a az alkalmazás.

- Az alkalmazás eredetileg egy másik hitelesítési módszert, lehet, hogy használatával egyéni felhasználói az áruházakkal, beépített, vagy nem rendelkezik a jogcímalapú technológiák által használt egyeztetés szabványok kezelésére képes. Jogcímalapú hitelesítés és hozzáférés-vezérlés alkalmazásával meglévő alkalmazásokba összetett, és valószínűleg nem költséghatékony lehet.

## <a name="example"></a>Példa

A szervezetek egy több-bérlős szoftver a Microsoft Azure-ban szolgáltatott szoftverként (SaaS) alkalmazás üzemelteti. Az alkalmazás tartalmazza a webhelyet, ahol a bérlők a saját felhasználók kezelésére használható. Az alkalmazás lehetővé teszi, hogy a bérlők számára a webhely elérését egy összevont identitás, amely szerint az Active Directory összevonási szolgáltatások (ADFS) jön létre, amikor a felhasználók hitelesítése azzal az adott szervezete saját Active Directory használatával.

![Hogyan egy nagyvállalati előfizetőn felhasználók az alkalmazás elérése](./_images/federated-identity-multitenat.png)


Az ábrán látható, hogyan bérlők hitelesítik magukat a saját identitásszolgáltató (1. lépés), ebben az esetben az AD FS. A bérlő a sikeres hitelesítés után az AD FS jogkivonatot ad ki. Az ügyfél böngészője a SaaS-alkalmazás összevonás-szolgáltatóként, amely megbízik a biztonsági jogkivonat, amely érvényes a Szolgáltatottszoftver-összevonás-szolgáltatóként (2. lépés) eléréséhez a bérlő ADFS által kiállított jogkivonatokat a token továbbítja. Ha szükséges, a Szolgáltatottszoftver-összevonási szolgáltató végez átalakítás a jogkivonatában lévő jogcímeket a jogcímeket, hogy az alkalmazás felismeri (3. lépés) az ügyfélböngészőnek az új jogkivonat visszatérése előtt. Az alkalmazás a Szolgáltatottszoftver-összevonási szolgáltató által kiállított jogkivonatokat megbízik, és használja a rendszer a jogcímeket a token (4. lépés)-engedélyezési szabályok vonatkoznak.

Bérlők számára nem szükséges az alkalmazás eléréséhez hitelesítő adatok megjegyzése, és a vállalat a bérlői rendszergazda konfigurálhat a saját ADFS az alkalmazáshoz hozzáférő felhasználók listáját.

## <a name="related-guidance"></a>Kapcsolódó útmutató

- [A Microsoft Azure Active Directoryban](https://azure.microsoft.com/services/active-directory/)
- [Active Directory Domain Services](https://msdn.microsoft.com/library/bb897402.aspx)
- [Az Active Directory összevonási szolgáltatások](https://msdn.microsoft.com/library/bb897402.aspx)
- [Több-bérlős alkalmazások identitáskezelése a Microsoft Azure-ban](https://azure.microsoft.com/documentation/articles/guidance-multitenant-identity/)
- [Több-bérlős alkalmazásokhoz az Azure-ban](https://azure.microsoft.com/documentation/articles/dotnet-develop-multitenant-applications/)
