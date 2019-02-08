---
title: 'CAF: Natív alapvető biztonsági házirend'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Natív alapvető biztonsági házirend
author: BrianBlanchard
ms.openlocfilehash: fc161009f6ce7aa2b775fe6b3b53ff1e1d62a848
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899037"
---
# <a name="cloud-native-security-baseline-policy"></a>Natív alapvető biztonsági házirend

[Alapvető biztonsági](overview.md) egyike a [felhőalapú irányítási öt oktatnak](../governance-disciplines.md). Ez fegyelem általános biztonsági témaköreit, beleértve az védelmét a hálózati, digitális eszközeit, adatokat és egyéb összpontosít. Foglaltak szerint a [házirend felülvizsgálati útmutató](../policy-compliance/what-is-a-cloud-policy-review.md), a CAF tartalmaz három szinten képes **házirend minta**: Natív, Enterprise és a felhő tervezési alapelv megfelelő minden öt szakterületen. Ez a cikk ismerteti a natív Felhőalkalmazások minta szabályzatot a biztonsági alapkonfiguráció tantárgy.

> [!NOTE]
> A Microsoft vállalati Diktálás vagy informatikai szabályzatnak nem helyzetben van. Ebből a cikkből egy belső házirend felülvizsgálatra előkészítéséhez. Feltételezzük, hogy ez a minta házirend kiterjesztett, érvényesítve, és mielőtt megpróbálja használni, a vállalati házirend tesztelve lett. Ez a minta-házirend használata, nem ajánlott.

## <a name="policy-alignment"></a>A házirend-igazítás

Ez a minta házirend szintetizál egy Felhőbeli natív forgatókönyv, ami azt jelenti, hogy az eszközök és az Azure által biztosított platform elegendőek a központi telepítés kapcsolatos üzleti kockázatok csökkentése. Ebben a forgatókönyvben azt feltételezzük, hogy egy egyszerű konfigurálás az alapértelmezett Azure-szolgáltatások védelmet biztosít a megfelelő eszközt.

## <a name="cloud-security-and-compliance"></a>Felhőbiztonság és -megfelelőség

Biztonsági integrálva van az Azure-ajánlat egyedi biztonsági előnyöket származó globális biztonsági intelligenciából, kifinomult a ügyfél felé irányuló vezérlők és a egy biztonságos, megerősített infrastruktúra minden szempontját. Ez a hatékony kombináció védelmet biztosít az alkalmazások és az adatok számára, támogatja a megfelelőség felé tett lépéseket, és költséghatékony biztonsági megoldást biztosít a különböző méretű vállalatok számára. Ezzel a módszerrel hoz létre egy erős kezdő pozíció egy-egy biztonsági házirend, de nem negálandó egyaránt erős ajánlott biztonsági eljárások a biztonsági szolgáltatások használt kapcsolódó van szükség.

### <a name="built-in-security-controls"></a>Beépített biztonsági vezérlőkkel

Meglehetősen nehéz egy erős biztonsági infrastruktúra fenntartásához, amikor a biztonsági vezérlők nem intuitív, és külön-külön kell konfigurálni. Az Azure tartalmaz beépített biztonsági vezérlők között különböző szolgáltatások, amelyek segítségével adatokat és a számítási feladatok gyors védelem, és kezelni a kockázatokat hibrid környezetekben. Integrált partneri megoldások segítségével is könnyen átmenet meglévő védelmet a felhőbe.

### <a name="cloud-native-identity-policies"></a>Felhőbeli natív identitás házirendek

Identitás neve lesz az új határ vezérlősík a biztonság érdekében átvenni az adott szerepkör hagyományos hálózati-központú szempontjából. Régebben a hálózati egyre elválasztó váltak, és a külső védelem nem lehet hatásos előtti alakulását a saját eszközök használata (BYOD), és a felhőbeli alkalmazásokhoz. Azure-Identitáskezelés és hozzáférés-vezérlés engedélyezése az alkalmazások a zökkenőmentes és biztonságos hozzáférést.

Egy mintául szolgáló felhőbeli natív felhőalapú és helyszíni címtárak közötti identitás szabályzatának lehetnek például a következő követelményeknek:

* A szerepköralapú hozzáférés-vezérlés (RBAC), a multi-factor authentication (MFA) és egyszeri bejelentkezés (SSO) erőforrásokhoz való hitelesített hozzáférés
* Gyors csökkenti a biztonsági sérülés gyanús felhasználói identitások
* – Igény szerinti (JIT), csak elegendő a fölösleges emelt szintű rendszergazdai hitelesítő adatok megjelenítésének korlátozása a feladat-feladat alapon hozzáférést
* Kiterjesztett felhasználói identitás és hozzáférés a több környezetben kezeli az Azure Active Directory házirendek

Fontos tudni, bár [identitásra építve](../identity-baseline/overview.md) alapvető biztonsági környezetében a [öt oktatnak, felhőalapú Cégirányítási](../overview.md) ki [identitásra építve](../identity-baseline/overview.md) , a saját szabályok betartását, külön biztonsági alaptervből.

### <a name="network-access-policies"></a>Hálózati hozzáférési szabályzatok

Hálózati vezérlő magában foglalja a konfigurálási, kezelési és biztonságossá tétele hálózati elemek, például a virtuális hálózat, terheléselosztás, a DNS és átjárók. A vezérlők, amelyikkel a szolgáltatások – kommunikáció és együttműködését. Az Azure tartalmaz egy hatékony és biztonságos hálózati infrastruktúra támogatja az alkalmazás és szolgáltatás hálózati kapcsolati követelményeinek. Hálózati kapcsolat a helyszíni között az Azure-ban található erőforrások között lehetőség, és az Azure-ban üzemeltetett erőforrásokhoz, és hogy, és az internetről és az Azure.

Egy felhőbeli natív házirend hálózati vezérlők, például az alábbi követelmények a következők lehetnek:

* Hibrid kapcsolatok a helyszíni erőforrásokhoz (bár technikailag lehetséges az Azure-ban), előfordulhat, hogy nem engedélyezi a Felhőbeli natív házirendben. Hibrid kapcsolat bizonyul szükséges, robusztus nagyvállalati biztonsági házirend minta lenne több megfelelő hivatkozást.
* Felhasználók és az Azure-beli virtuális hálózatok és a hálózati biztonsági csoportok segítségével biztonságos kapcsolatot is létrehozhat.
* Natív Windows Azure tűzfal védi a gazdagépek a korlátozott port hozzáférés a rosszindulatú hálózati forgalmat. Egy jó példa a szabályzat letiltása (vagy engedélyezze) adatforgalmát közvetlenül egy virtuális Gépet a követelmény lehet RDP - 3389-es TCP/UDP-porton keresztül.
* Szolgáltatások, például a webalkalmazási tűzfal és az Azure DDoS Protection védelme érdekében az alkalmazások és az Azure-ban futó virtuális gépek rendelkezésre állásának biztosításához. Ezek a funkciók nem szabad le van tiltva vagy helytelen.

### <a name="data-protection"></a>Adatvédelem

Az adatvédelem a felhőben, a kulcsok egyikét a számlázást a lehetséges állapotok, amelyben az adatok akkor fordulhat elő, és milyen vezérlők érhetők el az egyes állapotokhoz. Céljából az Azure data biztonság és titkosítás ajánlott eljárások javaslatokat a következő adatok állapotok összpontosít:

* Szolgáltatások a virtuális gépek tárolási és SQL Database beépített titkosítási vezérlést.
* Adatok helyeződnek át felhők és ügyfelei között, mivel azt védhetők iparági szabványnak megfelelő titkosítási protokollok használatával.
* Az Azure Key Vault lehetővé teszi, hogy a felhasználók biztonságosan tárolhatják és titkosítási kulcsok és egyéb titkok használják a felhőalapú alkalmazások és szolgáltatások számára.
* Az Azure Information Protection besorolása, címkézése és védelme a bizalmas adatok segítségével.

Ezeket a funkciókat az Azure-bA épített, amíg a fenti mindegyike konfigurációt igényel, és megnövelheti a költségek. Az egyes natív Felhőbeli szolgáltatások igazítás egy [adatok besorolási stratégiája](../policy-compliance/what-is-data-classification.md) erősen ajánlott.

### <a name="security-monitoring"></a>A biztonság monitorozása

Proaktív stratégiát, amely naplózza az erőforrásokat az azonosítani azokat a nem felel meg a vállalati szabványoknak vagy ajánlott eljárások a biztonságfigyelés legyen. Az Azure Security Center egységes biztonsági alapterv és fejlett fenyegetésvédelmet biztosít hibrid felhőalapú számítási feladatokhoz. A Security Centerrel biztonsági szabályzatokat alkalmazhat a számítási feladatok, korlátozhatja a fenyegetéseknek való kitettséget, és észlelheti és elháríthatja a támadásokat, beleértve a:

* Egyesített képet összes helyszíni és felhőbeli számítási feladatok az Azure Security Centerrel
* Folyamatos megfigyelés és a biztonság állapotát, a megfelelőség biztosítása és a biztonsági rések javítása
* Interaktív eszközökkel és fenyegetésekkel kapcsolatos környezetalapú tudásbázissal az egyszerűsített vizsgálatok
* Széles körű naplózási és integrációs szolgáltatásaival meglévő biztonsági információk

### <a name="extending-cloud-native-policies"></a>Kiterjeszti a Felhőbeli natív házirendek

A felhő használatával csökkentheti a biztonsági terheket. A Microsoft Azure-adatközpontok fizikai biztonságot biztosít, és a cloud platform infrastruktúra fenyegetések, például DDoS-támadások ellen védelmet nyújt. Tekintve, hogy a Microsoft kiberbiztonsági szakemberek dolgozik a biztonsági naponta több ezer rendelkezik, az erőforrásokat érő mérséklésére jelentős. Valójában a szervezetek számára lett egyszer aggódik a biztonságos volt-e a felhőbe, miközben a legtöbb most már tisztában vele, hogy adja meg, hogy a szállítók, például a Microsoft a személyek és a speciális infrastruktúra győződjön meg arról, befektetés mértékét, a felhő ténylegesen biztonságosabb, mint a legtöbb a helyszíni adatközpontjaik között.

A Felhőbeli natív biztonsági alapterv befektetés, mellett is, ajánlott, hogy alapvető biztonsági házirendek kibővíteni az alapértelmezett Felhőbeli natív szabályzatok. A következő példák kiterjesztett szabályzatokat kell figyelembe venni, akár natív felhőalapú környezetben:

* **Virtuális gépek védelmét**. Biztonsági minden szervezet prioritást kell lennie, és ennek során hatékonyan igényel számos dolog. Kell a biztonsági állapot értékelése, biztonsági fenyegetésekkel, és ezután észlelése és gyorsan reagáljanak az előforduló fenyegetéseket.
* **Virtuális gép tartalmának védelme**. Rendszeres automatikus biztonsági másolatok beállításához elengedhetetlen felhasználói hibák ellen. Ez a beállítás nem elegendő az, ha; is gondoskodnia kell arról, hogy a biztonsági mentések ellenállóbbak a kibertámadásokkal szemben, biztonságos, és rendelkezésére áll, amikor szüksége van rájuk.
* **Virtuális gépek és alkalmazások figyelését**. Ez a minta magában foglalja a több feladat, többek között a virtuális gépek ismertetése, közötti párbeszéd jelenti az állapotát betekintést és a létrehozó módon is figyelhetik az alkalmazásokat, ezek a virtuális gépek futtatásához. Ezek a feladatok összes alapvető fontosságú az alkalmazások folyamatos futtatásával éjjel-nappal.

## <a name="next-steps"></a>További lépések

Most, hogy áttekintette a natív felhőalapú megoldások a minta alapvető biztonsági házirend, térjen vissza a [házirend felülvizsgálati útmutató](../policy-compliance/what-is-a-cloud-policy-review.md) erre a példára, hozza létre saját szabályzatokat, a felhőre való áttérés kiépítésének megkezdésére.

> [!div class="nextstepaction"]
> [A saját a szabályzat felülvizsgálati útmutató alapján szabályzatokat hozhat létre](../policy-compliance/what-is-a-cloud-policy-review.md)
