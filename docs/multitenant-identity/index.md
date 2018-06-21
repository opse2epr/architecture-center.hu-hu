---
title: Identitáskezelés a több-bérlős alkalmazásokban
description: Ajánlott eljárások hitelesítéshez, engedélyezéshez, és identitáskezeléshez több-bérlős alkalmazások esetén.
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.next: tailspin
ms.openlocfilehash: c363ac01e798b522fa95f39586e28fe3af5fae4a
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
ms.locfileid: "24541673"
---
# <a name="manage-identity-in-multitenant-applications"></a>Identitáskezelés több-bérlős alkalmazásokban

A cikksorozat ajánlott eljárásokat ismertet a több-bérlős módhoz az Azure AD hitelesítésre és identitáskezelésre való használata esetén.

[![GitHub](../_images/github.png) Mintakód][sample application]

Több-bérlős alkalmazás létrehozásakor az elsőként jelentkező kihívások egyike a felhasználói identitások kezelése, mivel mostantól minden felhasználó egy bérlőhöz tartozik. Példa:

* A felhasználók bejelentkeznek a szervezeti hitelesítő adataikkal.
* A felhasználóknak hozzá kell férniük a szervezet adataihoz, de más bérlők adataihoz nem.
* A szervezet regisztrálhat az alkalmazásra, majd alkalmazás-szerepköröket rendelhet az egyes tagokhoz.

Az Azure Active Directory (Azure AD) remek funkciókkal rendelkezik, amelyek az összes itt felsorolt forgatókönyvet támogatják.

A cikksorozat kiegészítéseként elkészítettük egy több-bérlős alkalmazás [teljes körű megvalósítását][sample application]. A cikkek az alkalmazás létrehozása során szerzett tapasztalatainkat dolgozzák fel. Az alkalmazás létrehozásának első lépéseként tekintse meg [A Surveys alkalmazás futtatása][running-the-app] című cikket.

## <a name="introduction"></a>Bevezetés

Tegyük fel, hogy egy felhőalapú vállalati SaaS-alkalmazást szeretne elkészíteni. Az alkalmazásnak természetesen lesznek felhasználói:

![Felhasználók](./images/users.png)

Ezek a felhasználók viszont különböző szervezetekhez tartoznak:

![Szervezeti felhasználók](./images/org-users.png)

Példa: A Tailspin előfizetéseket értékesít SaaS-alkalmazásához. A Contoso és a Fabrikam előfizetnek erre az alkalmazásra. Amikor Ágnes (`alice@contoso`) bejelentkezik, az alkalmazásnak fel kell ismernie, hogy Ágnes a Contoso vállalathoz tartozik.

* Ágnesnek *hozzáférésre van szüksége* a Contoso adataihoz.
* Ágnes *nem férhet hozzá* a Fabrikam adataihoz.

Ez az útmutató bemutatja, hogyan kezelheti a felhasználói identitásokat egy több-bérlős alkalmazásban, ha az [Azure Active Directoryt][AzureAD] (AzureAD) használja a bejelentkezés és hitelesítés kezeléséhez.

## <a name="what-is-multitenancy"></a>Mi az a több-bérlős mód?
A *bérlőt* felhasználók egy csoportja alkotja. Egy SaaS-alkalmazásban a bérlő az alkalmazás egyik előfizetőjét vagy ügyfelét jelenti. A *több-bérlős* egy olyan architektúra, ahol egy alkalmazás egy fizikai példányán több bérlő osztozik. Bár a bérlők osztoznak a fizikai erőforrásokon (például a virtuális gépeken vagy a tárterületen), minden bérlő saját logikai példánnyal rendelkezik az alkalmazásból.

Az alkalmazásadatokat a felhasználók egy bérlőn belül általában megosztják egymással, más bérlők felhasználóival azonban nem.

![Több-bérlős architektúra](./images/multitenant.png)

Hasonlítsa össze ezt az architektúrát az egybérlős architektúrával, ahol minden egyes bérlő saját fizikai példánnyal rendelkezik. Egybérlős architektúra esetén bérlők hozzáadásakor új példányokat kell indítani az alkalmazásból.

![Egybérlős alkalmazás](./images/single-tenant.png)

### <a name="multitenancy-and-horizontal-scaling"></a>Több-bérlős architektúra és vízszintes méretezés
Ha méretezésre van szükség a felhőben, a leggyakoribb megoldás további fizikai példányok hozzáadása. Ez más néven a *vízszintes méretezés* vagy *horizontális felskálázás*. Vegyünk példaként egy webalkalmazást. Nagyobb forgalom kiszolgálásához hozzáadhat több virtuális gépet, és elhelyezheti azokat egy terheléselosztó mögé. Minden virtuális gép a webalkalmazás egy külön fizikai példányát futtatja.

![Terheléselosztás egy webhely esetében](./images/load-balancing.png)

Bármelyik kérés átirányítható bármelyik példányra. A rendszer minden része egyetlen logikai példányként működik. Leállíthat vagy el is indíthat egy virtuális gépeket anélkül, hogy ezzel hatással lenne a felhasználókra. Ebben az architektúrában minden fizikai példány több-bérlős, és a méretezés további példányok hozzáadásával történik. Ha az egyik példány leáll, az nincs hatással egyik bérlőre sem.

## <a name="identity-in-a-multitenant-app"></a>Identitás több-bérlős alkalmazásokban
A több-bérlős alkalmazások esetében a felhasználókat a bérlők kontextusában kell figyelembe venni.

**Hitelesítés**

* A felhasználók a szervezeti hitelesítő adatokkal jelentkeznek be az alkalmazásba. Nem kell saját felhasználói fiókot létrehozniuk az alkalmazáshoz.
* Az egyazon szervezeten belüli felhasználók ugyanahhoz a bérlőhöz tartoznak.
* Ha egy felhasználó bejelentkezik, az alkalmazás tudja, hogy a felhasználó melyik bérlőhöz tartozik.

**Engedélyezés**

* Egy felhasználó műveleteinek engedélyezésekor (például egy erőforrás megtekintésekor) az alkalmazásnak a felhasználó bérlőjét is figyelembe kell vennie.
* A felhasználók különböző szerepkörökkel rendelkezhetnek az alkalmazásban, mint például a „Rendszergazda” vagy az „Általános jogú felhasználó”. A szerepkör-hozzárendeléseket az ügyfélnek kell kezelnie, nem az SaaS-szolgáltatónak.

**Példa:** Ágnes, a Contoso alkalmazottja megnyitja az alkalmazást a böngészőből, és rákattint a „Bejelentkezés” gombra. A rendszer átirányítja őt egy bejelentkezési képernyőre, ahol meg kell adnia a vállalati bejelentkezési adatokat (felhasználónevet és jelszót). Ezen a ponton `alice@contoso.com` néven van bejelentkezve az alkalmazásba. Az alkalmazás azt is felismeri, hogy Ágnes az alkalmazás rendszergazdai szintű felhasználója. Rendszergazdaként láthatja a Contoso vállalathoz tartozó erőforrások teljes listáját. A Fabrikam vállalat erőforrásait azonban nem látja, mivel csak a saját bérlőjében rendelkezik rendszergazdai jogosultságokkal.

Ebben az útmutatóban kifejezetten az Azure AD-vel való identitáskezeléssel foglalkozunk.

* Feltételezzük, hogy az ügyfél az Azure AD-ben tárolja felhasználói profiljait (az Office 365- és a Dynamics CRM-bérlőket is beleértve)
* A helyszíni Active Directoryval (AD) rendelkező ügyfelek az [Azure AD Connect][ADConnect] segítségével szinkronizálhatják helyszíni AD-jüket az Azure AD-vel.

Ha egy helyszíni AD-vel rendelkező ügyfél (a vállalati informatikai házirend miatt vagy egyéb okból kifolyólag) nem tudja használni az Azure AD Connect szolgáltatást, az SaaS-szolgáltató az Active Directory összevonási szolgáltatások (AD FS) használatával is összevonható az ügyfél AD-jével. Ezt a lehetőséget az [Összevonás az ügyfél AD FS szolgáltatásával] című rész ismerteti.

Ez az útmutató más szempontból (pl. adatparticionálás, bérlőnkénti konfiguráció stb.) nem tárgyalja a több-bérlős architektúrákat.

[**Tovább**][tailpin]



<!-- Links -->
[ADConnect]: /azure/active-directory/active-directory-aadconnect
[AzureAD]: /azure/active-directory

[Összevonás az ügyfél AD FS szolgáltatásával]: adfs.md
[tailpin]: tailspin.md

[running-the-app]: ./run-the-app.md
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
