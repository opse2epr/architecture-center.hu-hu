---
title: Oldalkocsi minta
titleSuffix: Cloud Design Patterns
description: Egy alkalmazás összetevőit külön folyamatban vagy tárolóban helyezheti üzembe, így elkülönítést és beágyazást biztosíthat.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 2b0e46a06f7fe47f281f726f73128db1d7dd1067
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298833"
---
# <a name="sidecar-pattern"></a>Oldalkocsi minta

Egy alkalmazás összetevőit külön folyamatban vagy tárolóban helyezheti üzembe, így elkülönítést és beágyazást biztosíthat. Ez a minta azt is lehetővé teheti, hogy az alkalmazások heterogén összetevőkből és technológiákból álljanak.

Ezt a mintát *Oldalkocsi* mintának nevezik, mert egy motorhoz csatolt oldalkocsihoz hasonlít. A mintában az oldalkocsi egy szülőalkalmazáshoz van csatolva, és támogató szolgáltatásokat biztosít az alkalmazásnak. Az oldalkocsi ugyanakkora élettartammal rendelkezik, mint a szülőalkalmazás, együtt jönnek létre, és együtt vonhatók ki. Az oldalkocsi minta egy felbontási minta, amelyet segítőtárs mintának is neveznek.

## <a name="context-and-problem"></a>Kontextus és probléma

Az alkalmazások és szolgáltatások gyakran megkövetelnek kapcsolódó funkciókat, például monitorozást, naplózást, konfigurálást és hálózati szolgáltatásokat. Ezek a perifériafeladatok különálló összetevőkként vagy szolgáltatásokként is megvalósíthatók.

Ha szorosan integráltak az alkalmazásba, futtathatók ugyanabban a folyamatban, mint az alkalmazás, így biztosítható a megosztott erőforrások hatékony használata. Ez azonban azt is jelenti, hogy nem jól elkülönítettek, ezért egy kimaradás az összetevők egyikében hatással lehet a többi összetevőre vagy az egész alkalmazásra is. Továbbá ugyanannak a nyelvnek a használatával kell megvalósítani őket, mint amelyet a szülőalkalmazás használ. Ennek eredményeképpen az összetevő és az alkalmazás erősen függ egymástól.

Ha az alkalmazás szolgáltatásokra van bontva, akkor minden egyes szolgáltatás felépíthető több különböző nyelv és technológia használatával. Míg ez nagyobb rugalmasságot biztosít, azt is jelenti, hogy minden összetevőnek vannak saját függőségei, és nyelvspecifikus kódtárakra van szükségük az alapul szolgáló platform és a szülőalkalmazással megosztott erőforrások eléréséhez. Továbbá ezeknek a tulajdonságoknak a különálló szolgáltatásként való telepítése késleltetést okozhat az alkalmazásban. A nyelvspecifikus felületek kódjainak és függőségeinek a kezelése nagyobb összetettséget eredményezhet, különösen az üzemeltetésnél, az üzembe helyezésnél és a kezelésnél.

## <a name="solution"></a>Megoldás

Helyezzen egy kohézióval jellemezhető feladatcsoportot arra a helyre, ahol a főalkalmazás is található, de a feladatcsoportot külön folyamatba vagy tárolóba helyezze, hogy minden nyelven egységes felületet hozzon létre a platformszolgáltatásoknak.

![Az oldalkocsi minta ábrája](./_images/sidecar.png)

Egy oldalkocsi szolgáltatás nem feltétlenül az alkalmazás része, de kapcsolódik hozzá. Mindenhová követi a szülőalkalmazást. Az oldalkocsik olyan folyamatokat és szolgáltatásokat támogatnak, amelyek az elsődleges alkalmazással lettek üzembe helyezve. A motorkerékpárok esetében az oldalkocsi egy motorkerékpárhoz csatlakozik, és minden motorkerékpárnak lehet saját oldalkocsija. Ehhez hasonlóan egy oldalkocsi szolgáltatás is követi a szülőalkalmazását. Az alkalmazás minden egyes példányához kapcsolódik egy telepített oldalkocsi példány, amely vele együtt fut.

Az oldalkocsi minta használatának előnyei:

- Az oldalkocsi független az elsődleges alkalmazásától, ami a futtatókörnyezetet és a programozási nyelvet illeti, így nem kell nyelvenként külön oldalkocsit fejleszteni.

- Az oldalkocsi hozzáférhet ugyanazokhoz az erőforrásokhoz, amelyekhez az elsődleges alkalmazás is hozzáfér. Az oldalkocsi például monitorozhatja mind az oldalkocsi, mind az elsődleges alkalmazás által használt rendszer-erőforrásokat.

- Mivel közel helyezkedik el az elsődleges alkalmazáshoz, nincs jelentős késleltetés a közöttük zajló kommunikáció során.

- Még a bővítési mechanizmust nem biztosító alkalmazásoknál is használhatja az oldalkocsit a funkciók kibővítéséhez úgy, hogy saját folyamatként csatolja ugyanazon a gazdagépen vagy altárolóban, ahol az elsődleges alkalmazás is található.

Az oldalkocsi mintát gyakran használják tárolókhoz is, amely esetben oldalkocsi tároló vagy segítőtárs tároló jön létre.

## <a name="issues-and-considerations"></a>Problémák és megfontolandó szempontok

- Gondolja át, milyen üzembe helyezési és csomagolási formátumot használ a szolgáltatások, folyamatok és tárolók telepítéséhez. A tárolók kiválóan alkalmasak az oldalkocsi minta használatához.
- Egy oldalkocsi szolgáltatás tervezésekor alaposan gondolja át, milyen folyamatok közötti kommunikációs mechanizmus mellett dönt. Próbáljon nyelvtől vagy keretrendszertől független technológiákat használni, kivéve, ha a teljesítményre vonatkozó követelmények ezt problémássá teszik.
- Mielőtt funkciókat rendel az oldalkocsihoz, gondolja át, hogy önálló szolgáltatásként vagy démonként nem működne-e jobban.
- Vegye figyelembe azt is, hogy a funkció kódtárként vagy egy hagyományos bővítménymechanizmussal is megvalósítható. Előfordulhat, hogy a nyelvspecifikus kódtárak mélyebb szintű integrációval és kisebb hálózati terheléssel rendelkeznek.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes ezt a mintát használni?

Használja ezt a mintát, ha:

- Az elsődleges alkalmazása egy heterogén nyelv és keretrendszer használ. Egy oldalkocsi szolgáltatásban található összetevőt különféle nyelveken írott és különféle keretrendszereket használó alkalmazások is felhasználhatnak.
- Egy összetevő egy távoli csapat vagy egy másik vállalat tulajdonában van.
- Egy összetevőt vagy szolgáltatást ugyanazon a gazdagépen kell elhelyezni, mint az alkalmazást.
- Egy olyan szolgáltatásra van szüksége, amely ugyanakkora életciklussal rendelkezik, mint a főalkalmazása, de egymástól függetlenül frissíthetők.
- Egy erőforrás vagy összetevő erőforrás-korlátozásait részletesen kívánja szabályozni. Előfordulhat például, hogy korlátozni szeretné a megadott összetevő által használt memória mennyiségét. Az összetevőt telepítheti oldalkocsiként, és a főalkalmazástól függetlenül kezelheti a memóriahasználatot.

Nem érdemes ezt a mintát használni, ha:

- Ha optimalizálni kívánja a folyamatok közötti kommunikációt. A szülőalkalmazás és az oldalkocsi közötti kommunikáció némi többletterhelést okoz, amely leginkább a hívásokat érintő késésekben nyilvánul meg. Ez nem minden esetben fogadható el kompromisszumként a forgalmas felületek esetében.
- Kisebb alkalmazásoknál, ahol annak az erőforrásköltsége, hogy minden példányhoz egy oldalkocsi szolgáltatás legyen üzembe helyezve, túlságosan magas az elkülönítés előnyeihez viszonyítva.
- Ha a szolgáltatást a főalkalmazásoktól eltérően, vagy azoktól függetlenül kell skálázni. Ebben az esetben jobb megoldás lehet különálló szolgáltatásként üzembe helyezni.

## <a name="example"></a>Példa

Az oldalkocsi minta számos forgatókönyv esetén alkalmazható. Gyakori példák:

- Infrastruktúra-API. Az infrastruktúra-fejlesztő csapat egy olyan szolgáltatást hoz létre, amelyet a rendszer minden alkalmazás mellé telepít, nem pedig egy nyelvspecifikus ügyfélkódtárat, amellyel elérhető az infrastruktúra. A szolgáltatás egy oldalkocsiként van betöltve, és egy közös réteget biztosít az infrastruktúra-szolgáltatásoknak, beleértve a naplózást, a környezeti adatokat, a konfigurációs adattárakat, a felderítést, az állapot-ellenőrzéseket és a figyelő szolgáltatásokat. Az oldalkocsi a szülőalkalmazás gazdagép-környezetét és folyamatát (vagy tárolóját) is monitorozza, és naplózza az adatokat egy központosított szolgáltatásba.
- NGINX/HAproxy felügyelete. Telepítse az NGINX-et egy olyan oldalkocsi szolgáltatással, amely a környezet állapotát monitorozza, majd frissíti az NGINX konfigurációs fájlját és újraindítja a folyamatot, ha állapotmódosításra van szükség.
- Nagykövet oldalkocsi. Üzembe helyezése egy [Nagykövet](./ambassador.md) szolgáltatást oldalkocsiként. Az alkalmazás meghívja a nagykövetet, amely a bejelentkezéseket, az útválasztást, az áramköri megszakítást és az egyéb, csatlakozáshoz kapcsolódó szolgáltatásokat kezeli.
- Proxykiszervezés. Helyezzen egy NGINX-proxyt egy node.js szolgáltatáspéldány elé a szolgáltatás statikusfájl-tartalmai kiszolgálásának kezeléséhez.

## <a name="related-guidance"></a>Kapcsolódó útmutatók

- [Nagykövet minta](./ambassador.md)
