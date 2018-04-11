---
title: Oldalkocsi minta
description: Egy alkalmazás elkülönítési és beágyazás egy külön folyamatban vagy a tároló összetevőinek telepítéséhez.
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: ec168009aa99f412c3f1222a1c404ea4ea5cb669
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="sidecar-pattern"></a>Oldalkocsi minta

Egy alkalmazás elkülönítési és beágyazás egy külön folyamatban vagy a tároló összetevőinek telepítéséhez. Ebben a mintában a heterogén összetevők és technológiák állhatnak alkalmazások is engedélyezheti.

Ebben a mintában nevű *oldalkocsi* , mert azt egy motorkerékpárja csatolva oldalkocsi hasonlít. A mintában a oldalkocsi csatolva van egy szülő-alkalmazást, és a támogató szolgáltatásokat biztosítja az alkalmazás. A oldalkocsi is megosztja a szülő alkalmazás létrehozása és a velük a szülő már nincs azonos életciklussal. A oldalkocsi mintát van más néven a sidekick mintát, és a felbontás ellen minta.

## <a name="context-and-problem"></a>A környezetben, és probléma

Alkalmazások és szolgáltatások gyakran megkövetelik a kapcsolódó funkciókat, például figyelés, a naplózás, konfigurációs vagy hálózati szolgáltatások. A perifériák feladatok külön összetevők vagy a szolgáltatások valósítható meg. 

Ha ezek szorosan integrált az alkalmazásba, futtathatók legyenek meg az alkalmazás, így a megosztott erőforrások hatékony használatát. Azonban ez azt is jelenti nem jól elkülönített kerül, és ezek az összetevők egyikében kimaradás hatással lehet a többi összetevő vagy a teljes alkalmazás. Is akkor általában kell használnia az alkalmazás ugyanezen a nyelven használatával. Ennek eredményeképpen az összetevő és az alkalmazás rendelkezik Bezárás egyes egymással.

Ha az alkalmazás az szolgáltatásaiba van kiválasztott, majd minden egyes szolgáltatás építhetők több különböző nyelvet és technológiákat. Amíg ez a nagyobb rugalmasságot biztosít, az azt jelenti, hogy minden összetevő saját függőségei vannak, és nyelvspecifikus szalagtárak az alapul szolgáló platform és a szülőalkalmazás megosztott erőforrások eléréséhez szükséges. Ezek a szolgáltatások emellett telepítése, a szolgáltatások külön is késés hozzáadni az alkalmazáshoz. A kód és nyelvspecifikus konfigurációkezelővel függőségeinek kezelése jelentős összetettségét, különösen olyan üzemeltetési, telepítése és kezelése is hozzáadhat.

## <a name="solution"></a>Megoldás

Közös elhelyezése a feladatokat az elsődleges alkalmazással javul készlete, de a saját folyamat vagy a tárolót, és egy egységes felületet biztosít a platformszolgáltatások több nyelv belülre. 

![](./_images/sidecar.png)

Egy oldalkocsi szolgáltatás része nem feltétlenül az alkalmazás, de van csatlakoztatva. Bárhol az alkalmazás állapotba kerül. Az elsődleges alkalmazás telepített szolgáltatások vagy a folyamatok hányadossal támogatják. Egy motorkerékpárja a oldalkocsi csatolva van egy motorkerékpárja, és minden motorkerékpárja rendelkezhet saját oldalkocsi. Ugyanúgy egy oldalkocsi szolgáltatás megoszt a szülőalkalmazás sorsáról. Az alkalmazás egyes példányainak a oldalkocsi példányának telepítése és mellette megtalálható. 

Oldalkocsi minta használatával a következő előnyöket nyújtja:

- Oldalkocsi nem függ össze az elsődleges alkalmazás futtatókörnyezetben és programozási nyelv, így nem kell egy oldalkocsi nyelvenként fejlesztéséhez. 

- A oldalkocsi férhetnek hozzá az erőforrásokhoz, az elsődleges alkalmazás. Például oldalkocsi figyelheti a oldalkocsi, mind az elsődleges alkalmazás által használt rendszer-erőforrások. 

- Az elsődleges alkalmazás a közelében, mert nincs jelentős késleltetés közötti kommunikáció során.

- Még a alkalmazásokat, amelyek nem egy kiegészítő mechanizmust használhatja oldalkocsi bővíthetők ugyanabban a gazdagép vagy az elsődleges alkalmazás altárolója saját folyamatként csatolásával.

Oldalkocsi mintát gyakran használt tárolók és a oldalkocsi tároló vagy sidekick tároló néven. 

## <a name="issues-and-considerations"></a>Problémákat és szempontok

- Vegye figyelembe a központi telepítés és a szolgáltatások, folyamatok és tárolók központi telepítéséhez használandó csomagolás formátumban. Tárolók különösen alkalmasak a oldalkocsi mintát.
- Egy oldalkocsi szolgáltatás tervezésekor gondosan adja meg a folyamatok közti kommunikációs mechanizmus. Próbálja nyelvet vagy keretrendszert független technológiák használatára, kivéve a teljesítményre vonatkozó követelmények ügyeljen, hogy praktikus.
- Előtt funkció üzembe oldalkocsi, vegye figyelembe, hogy azt kellene működne egy önálló szolgáltatásként vagy a hagyományosabb démon.
- Vegye figyelembe, hogy a funkció tárként találhatja vagy egy hagyományos bővítmény mechanizmussal is. Előfordulhat, hogy nyelvspecifikus szalagtárak mélyebb szintű integrációt és kisebb hálózati terhelés.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes használni ezt a mintát

Ez mintát, mikor használja:

- Az elsődleges alkalmazás nyelv és keretrendszer heterogén készletét használja. Egy összetevő egy oldalkocsi szolgáltatásban található különböző keretrendszerek használatával különböző nyelveken írt alkalmazások által igénybe vehető.
- Egy összetevő egy távoli munkacsoport vagy egy másik szervezet tulajdonában van.
- Egy összetevő vagy szolgáltatás közös elhelyezése szükséges az alkalmazás ugyanazon a gazdagépen
- Egy szolgáltatás, amely a teljes életciklusát a fő alkalmazás közösen használja, de egymástól függetlenül frissíthetők van szüksége.
- Erőforrás-korlátozások részletes szabályozhatják egy adott erőforráshoz vagy az összetevő szükség van. Például előfordulhat, hogy korlátozni szeretné a megadott összetevő által használt memória mennyiségét. A oldalkocsi összetevőhöz telepítheti, és független az alkalmazás memória-felhasználás kezelésére.

Ez a minta nem alkalmasak lehetnek:

- Ha a folyamatok közti kommunikációs kell lehet optimalizálni. A szülő alkalmazás- és oldalkocsi közötti kommunikáció némi többletterhelést okoz, különösen a hívások a késés tartalmazza. Ez nem lehet egy elfogadható kompromisszum chatty kapcsolatok.
- Kis alkalmazások ahol költségeket, az erőforrás üzembe egy oldalkocsi szolgáltatás az összes olyan példány nincs érdemes elkülönítési előnyeit.
- Ha a szolgáltatás kell másképp méretezhető, mint vagy egymástól függetlenül a fő alkalmazásokból. Ha igen, jobb megoldás lehet különálló szolgáltatásként a szolgáltatás telepítéséhez.

## <a name="example"></a>Példa

A oldalkocsi minta nem számos forgatókönyv alkalmazandó. Gyakori példák:

- Infrastruktúra API. Az infrastruktúra fejlesztői csapat olyan szolgáltatás, amely mellett minden egyes alkalmazás helyett a nyelvspecifikus ügyféloldali kódtára a infrastruktúra eléréséhez a rendszer létrehozza. A szolgáltatás oldalkocsi betöltve és egy közös réteget biztosít az infrastruktúra-szolgáltatásokat, beleértve a naplózási, a környezeti adatok, a konfigurációs adattároló, a felderítés, ellenőrzi, és figyelő szolgáltatások. A oldalkocsi is figyeli a szülő alkalmazás a gazdagép-környezetben és a folyamat (vagy tároló), és naplózza az adatokat egy központosított szolgáltatáshoz.
- Kezelheti a NGINX/haproxy esetén. Központi telepítés NGINX környezet állapotát figyeli, majd frissíti a NGINX konfigurációs fájlt, és újraindul a folyamat, amikor szükség van az állapot oldalkocsi szolgáltatás.
- Diplomata oldalkocsi. Központi telepítése egy [diplomata] [ ambassador] oldalkocsi mint szolgáltatás. Az alkalmazás meghívja a nagykövetet, amely leírók megtörje logging, útválasztási, áramkör kérelmezéséhez keresztül, és más csatlakozási kapcsolódó szolgáltatások.
- Proxy-kiszervezés. Egy NGINX proxy kezelésére szolgáló statikus fájl tartalma a szolgáltatás egy node.js szolgáltatáspéldány elé helyezze el.


## <a name="related-guidance"></a>Kapcsolódó útmutató

- [Diplomata minta][ambassador]


[ambassador]: ./ambassador.md

