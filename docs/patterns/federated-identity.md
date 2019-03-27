---
title: Összevont identitás mintája
titleSuffix: Cloud Design Patterns
description: A hitelesítést delegálhatja egy külső identitásszolgáltatónak.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: f4e8d2dff1c653cf2b8ef0109f1cc3027e5428b9
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298585"
---
# <a name="federated-identity-pattern"></a>Összevont identitás mintája

[!INCLUDE [header](../_includes/header.md)]

A hitelesítést delegálhatja egy külső identitásszolgáltatónak. Ezzel leegyszerűsítheti a fejlesztést, minimalizálhatja a szükséges felhasználói adminisztrációt és javíthatja az alkalmazás felhasználói élményét.

## <a name="context-and-problem"></a>Kontextus és probléma

A felhasználóknak általában több, különböző üzleti partner által biztosított és üzemeltetett alkalmazást kell használniuk. Előfordulhat, hogy ezeknek a felhasználóknak mindegyikhez adott (eltérő) hitelesítő adatokat kell használniuk. Mindez:

- **Nem egységes felhasználói élményt okozhat**. A felhasználók gyakran elfelejtik a bejelentkezési hitelesítő adataikat, ha sok különböző adatot kell kezelniük.

- **Biztonsági réseket idézhet elő**. Ha egy felhasználó kilép a cégtől, a fiókját azonnal meg kell szüntetni. Nagy cégeknél erről könnyű megfeledkezni.

- **Bonyolítja a felhasználókezelést**. A rendszergazdáknak kell kezelniük az összes felhasználó hitelesítő adatait, és pluszfeladatokat (például jelszó-emlékeztetők küldését) kell elvégezniük.

A felhasználók általában jobban szeretnék ugyanazokat a hitelesítő adatokat használni az összes alkalmazáshoz.

## <a name="solution"></a>Megoldás

Használjon összevont identitás használatára képes hitelesítési mechanizmust. Különítse el a felhasználói hitelesítést az alkalmazáskódtól, és delegálja a hitelesítést egy megbízható identitásszolgáltatóra. Ezzel leegyszerűsítheti a fejlesztést, és lehetővé teheti, hogy a felhasználók szélesebb palettáról válasszanak identitásszolgáltatót a hitelesítéshez, és közben a lehető legkisebbre csökkentsék az adminisztratív terhelést. Azt is lehetővé teszi, hogy egyértelműen elválassza a hitelesítést az engedélyezéstől.

A megbízható identitásszolgáltatók között vannak céges címtárak, helyszíni összevonási szolgáltatások, külső üzleti partnerek által biztosított biztonsági jegykiadó szolgáltatások (STS), vagy éppen közösségi identitásszolgáltatók, amelyek képesek kezelni például a Microsoft-, Google-, Yahoo!- vagy Facebook-fiókkal rendelkező felhasználók hitelesítését.

Az ábra az összevont identitás mintáját mutatja be, amikor egy ügyfélnek épp egy hitelesítést igénylő szolgáltatáshoz kell hozzáférnie. A hitelesítést egy STS-sel együttműködő identitásszolgáltató hajtja végre. Az identitásszolgáltató a hitelesített felhasználóról információt szolgáltató biztonsági jogkivonatokat ad ki. Ez a jogcímnek is nevezett információ tartalmazza a felhasználó identitását, és tartalmazhat más adatokat is, például a szerepkörtagságot és részletesebb hozzáférési jogosultságokat.

![Az összevont hitelesítés áttekintése](./_images/federated-identity-overview.png)

Ezt a modellt gyakran nevezik jogcímalapú hozzáférés-vezérlésnek. Az alkalmazások és a szolgáltatások a jogkivonatban található jogcímek alapján engednek hozzáférést a szolgáltatásokhoz és funkciókhoz. A hitelesítést igénylő szolgáltatásnak meg kell bíznia az identitásszolgáltatóban. Az ügyfélalkalmazás kapcsolatba lép a hitelesítést elvégző identitásszolgáltatóval. Ha a hitelesítés sikeres, az identitásszolgáltató a felhasználót azonosító jogcímeket tartalmazó jogkivonatot ad vissza az STS-nek (tartsa szem előtt, hogy az identitásszolgáltató és az STS lehet ugyanaz a szolgáltatás is). Az STS az ügyfélnek való visszaadás előtt előre meghatározott szabályok alapján átalakíthatja és kiegészítheti a jogcímeket. Az ügyfélalkalmazás ezután továbbíthatja ezt a jogkivonatot a szolgáltatásnak az identitása bizonyítékaként.

> A megbízhatósági láncban lehetnek további STS-ek. A később ismertetett forgatókönyvben például egy helyszíni STS megbízik egy másik, a felhasználó hitelesítése céljából egy identitásszolgáltatóhoz való hozzáférésért felelős STS-ben. Ez a megközelítés gyakori nagyvállalati forgatókönyvekben, ahol van helyszíni STS és címtár is.

Az összevont hitelesítés szabványalapú megoldást nyújt a különböző tartományokban található identitásokba fektetett bizalom problémájára, és képes az egyszeri bejelentkezés támogatására. Minden típusú alkalmazásnál egyre elterjedtebb, különösen a felhőalapú alkalmazásoknál, mert anélkül támogatja az egyszeri bejelentkezést, hogy közvetlen hálózati kapcsolatot igényelne az identitásszolgáltatókhoz. A felhasználónak nem kell minden alkalmazáshoz megadnia a hitelesítő adatait. Ez növeli a biztonságot, mivel nem kell létrehozni a sok különböző alkalmazáshoz való hozzáféréshez szükséges hitelesítő adatokat, és az eredeti identitásszolgáltatón kívül minden alkalmazás elől elrejthetők a felhasználó hitelesítő adatai. Az alkalmazások csak a jogkivonatban található hitelesített identitásadatokat látják.

Az összevont identitásnak az is nagy előnye, hogy az identitás és a hitelesítő adatok kezelése az identitásszolgáltató felelőssége. Az alkalmazásnak vagy szolgáltatásnak nem kell identitáskezelési funkciókat biztosítania. Ezenfelül céges forgatókönyvekben a céges címtárnak nem kell tudnia a felhasználóról, ha megbízik az identitásszolgáltatóban. Ezzel megszűnik a felhasználók identitásának címtáron belül való kezelésének adminisztratív terhe.

## <a name="issues-and-considerations"></a>Problémák és megfontolandó szempontok

Összevont hitelesítést megvalósító alkalmazások tervezésekor vegye figyelembe az alábbiakat:

- A hitelesítés kritikus hibapont lehet a rendszeren belül. Ha több adatközpontba telepíti az alkalmazását, érdemes lehet az identitáskezelő mechanizmust is telepíteni ugyanazokba az adatközpontokba, hogy megmaradjon az alkalmazás megbízhatósága és rendelkezésre állása.

- A hitelesítési eszközök segítségével konfigurálható a hitelesítési jogkivonatban található szerepkörjogcímeken alapuló hozzáférés-vezérlés. Ezt gyakran nevezik szerepköralapú hozzáférés-vezérlésnek (RBAC), és részletesebb szintű hozzáférés-vezérlést tesz lehetővé a szolgáltatások és erőforrások vonatkozásában.

- A céges címtáraktól eltérően a közösségi identitásszolgáltatókat használó jogcímalapú hitelesítés az e-mail-címen és esetleg a néven kívül általában nem ad meg más adatot a hitelesített felhasználóról. Néhány közösségi identitásszolgáltató – például egy Microsoft-fiók – csak egy egyedi azonosítót biztosít. Az alkalmazásnak általában kezelnie kell néhány adatot a regisztrált felhasználókról, és képesnek kell lennie megfeleltetni ezt az információt a jogkivonat jogcímeiben tárolt azonosítóval. Ezt általában a regisztráció során szerzi be, amikor a felhasználó először használja az alkalmazást, majd az adatok további jogcímekként kerülnek be a jogkivonatba az egyes hitelesítések után.

- Ha egynél több identitásszolgáltató van konfigurálva az STS-hez, annak észlelnie kell, hogy melyik identitásszolgáltatóhoz kell átirányítania az adott felhasználót hitelesítésre. Ennek a folyamatnak a kezdőtartomány felderítése a neve. Lehetséges, hogy az STS erre képes automatikusan a felhasználó által megadott e-mail-cím vagy felhasználónév, a felhasználó által használni kívánt alkalmazás altartománya, a felhasználó IP-címtartománya vagy egy, a felhasználó böngészőjében tárolt cookie tartalma alapján. Ha például a felhasználó egy Microsoft-tartományba eső e-mail-címet adott meg (pl.: user@live.com), az STS a felhasználót a Microsoft-fiók bejelentkezési oldalára irányítja át. A későbbi látogatások alkalmával az STS egy cookie segítségével jelezheti, hogy az utolsó bejelentkezés egy Microsoft-fiók segítségével történt. Ha az automatikus észlelés nem tudja meghatározni a kezdőtartományt, az STS megjelenít egy kezdőtartomány-felderítő oldalt, amely kilistázza a megbízható identitásszolgáltatókat, és a felhasználónak kell kiválasztania, melyiket szeretné igénybe venni.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes ezt a mintát használni?

Ez a minta például az alábbi forgatókönyvek esetében lehet hasznos:

- **Egyszeri bejelentkezés a vállalatban**. Ebben a forgatókönyvben az alkalmazottakat hitelesíteni kell a felhőben üzemeltetett, vállalati biztonsági határon kívül eső céges alkalmazásokhoz, de nem kell minden alkalommal bejelentkezniük, amikor használnak egy alkalmazást. A felhasználói élmény ugyanaz, mint a helyszíni alkalmazások használatakor, azaz a hitelesítés egy vállalati hálózatra való bejelentkezéskor történik, és onnantól bejelentkezés nélkül használható az összes releváns alkalmazás.

- **Összevont identitás több partnerrel**. Ebben a forgatókönyvben a céges alkalmazottakat és a céges címtárban fiókkal nem rendelkező üzleti partnereket is hitelesítenie kell. Ez a gyakori a cégek közötti alkalmazások és a külső szolgáltatást integráló alkalmazások esetében, valamint olyan helyzetben, amely során különböző informatikai rendszert alkalmazó vállalatok egyesülnek vagy megosztják az erőforrásaikat.

- **Összevont identitás SaaS-alkalmazásokban**. Ebben a forgatókönyvben független szoftvergyártók biztosítanak használatra kész szolgáltatást több ügyfélnek vagy bérlőnek. Mindegyik bérlő egy megfelelő identitásszolgáltató használatával végzi el a hitelesítést. Az üzleti felhasználók például a céges hitelesítő adatokat fogják használni, a bérlő felhasználói és ügyfelei pedig a közösségi identitásuk hitelesítő adatait.

Nem érdemes ezt a mintát használni a következő helyzetekben:

- Az alkalmazás összes felhasználója hitelesíthető egy identitásszolgáltatóval, és nincs szükség más identitásszolgáltatóval való hitelesítésre. Ez olyan üzleti alkalmazásoknál gyakori, amelyek az alkalmazásból elérhető céges címtárat használnak hitelesítéshez VPN- vagy (felhőben futtatott forgatókönyvek esetében) a helyszíni címtár és az alkalmazás közötti kapcsolatot biztosító virtuális hálózati kapcsolaton keresztül.

- Az alkalmazás eredetileg egy másik hitelesítési mechanizmussal készült, esetleg egyéni felhasználói tárolókkal, vagy nem képes kezelni a jogcímalapú technológiák által használt egyeztetési szabványokat. A jogcímalapú hitelesítés és hozzáférés-vezérlés meglévő alkalmazásokra történő alkalmazása összetett feladat, és valószínűleg nem túl költséghatékony.

## <a name="example"></a>Példa

Egy cég több-bérlős szolgáltatott szoftvert (SaaS) üzemeltet alkalmazásként a Microsoft Azure-ban. Az alkalmazás tartalmaz egy webhelyet, amelyet a bérlők használhatnak az alkalmazás a saját felhasználóik számára történő kezeléséhez. Az alkalmazás lehetővé teszi a bérlők számára a webhely elérését egy Active Directory összevonási szolgáltatások (ADFS) által létrehozott összevont identitás használatával, ha a felhasználót a cég saját Active Directoryja hitelesítette.

![Az alkalmazás elérési módja a felhasználók számára nagyvállalati előfizető esetében](./_images/federated-identity-multitenat.png)

Az ábrán látható, hogyan történik meg a bérlők hitelesítése a saját identitásszolgáltatójukkal (1. lépés), ebben az esetben az ADFS-sel. Egy bérlő sikeres hitelesítése után az ADFS kiad egy jogkivonatot. Az ügyfél böngészője továbbítja ezt a jogkivonatot a SaaS-alkalmazás összevonási szolgáltatójának – amely megbízik a bérlő ADFS-e által kiállított jogkivonatokban –, hogy egy, a SaaS összevonási szolgáltatójához érvényes jogkivonatot kapjon vissza (2. lépés). Ha szükséges, a SaaS összevonási szolgáltató a jogkivonatban található jogcímeket az új jogkivonat ügyfélböngészőnek való visszaadása előtt olyan jogcímekké alakítja, amelyeket az alkalmazás felismer (3. lépés). Az alkalmazás megbízik az SaaS összevonási szolgáltató által kiállított jogkivonatokban, és a jogkivonat jogcímeivel alkalmazza az engedélyezési szabályokat (4. lépés).

A bérlőknek nem kell emlékezniük a különböző hitelesítő adataikra az alkalmazás eléréséhez, és a bérlő vállalata konfigurálhatja azon felhasználók listáját a saját ADFS-ben, akik hozzáférhetnek az alkalmazáshoz.

## <a name="related-guidance"></a>Kapcsolódó útmutatók

- [Microsoft Azure Active Directory](https://azure.microsoft.com/services/active-directory/)
- [Active Directory Domain Services](https://msdn.microsoft.com/library/bb897402.aspx)
- [Active Directory összevonási szolgáltatások](https://msdn.microsoft.com/library/bb897402.aspx)
- [Több-bérlős alkalmazások identitáskezelése a Microsoft Azure-ban](/azure/architecture/multitenant-identity)
- [Több-bérlős alkalmazások az Azure-ban](/azure/dotnet-develop-multitenant-applications)
