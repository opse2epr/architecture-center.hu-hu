---
title: Diplomata minta
description: Segítő szolgáltatások által küldött kérelmek nevében fogyasztói szolgáltatás vagy alkalmazás létrehozására.
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: 6c545619aab6a5817e55854350e3769834df27cd
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
ms.locfileid: "24540793"
---
# <a name="ambassador-pattern"></a>Diplomata minta

Segítő szolgáltatások által küldött kérelmek nevében fogyasztói szolgáltatás vagy alkalmazás létrehozására. Diplomata szolgáltatásnak közösen van elhelyezve az ügyfél-folyamaton kívüli proxyként tekinthetők.

Ebben a mintában hasznosak lehetnek a gyakori ügyfél kapcsolat feladatait, például a figyelés, naplózási, útválasztást, biztonságot (például a TLS), és [rugalmassági minták] [ resiliency-patterns] nyelvű független módon. Gyakran használják az örökölt alkalmazások vagy más alkalmazásokat, amelyek a nehezen módosítása, hogy a hálózati lehetőségek bővítése. Azt is engedélyezheti a speciális csoport valósít meg ezeket a funkciókat.

## <a name="context-and-problem"></a>A környezetben, és probléma

Rugalmas felhőalapú alkalmazások szükséges szolgáltatások, mint [áramkör legfrissebb][circuit-breaker], útválasztási, mérési és figyelésére és a hálózati konfigurációs módosításokat lehessen. Nehéz vagy lehetetlen frissíteni örökölt alkalmazások vagy a meglévő kódot szalagtárak ezeket a szolgáltatásokat hozzáadni, mert a kód már nem kezelt, vagy egyszerűen nem lehet módosítani a fejlesztő csapat által.

Hálózati hívások is kapcsolat, hitelesítési és engedélyezési jelentős konfigurálásra lehet szükség. Ha az adott hívások is használ, több alkalmazás, segítségével több nyelv és keretrendszer, a hívások kell konfigurálni minden esetben. Ezenkívül hálózati és biztonsági funkcióinak esetleg a szervezeten belüli központi csoport felügyeli. A nagyméretű kódbázis lehet, hogy a csoport frissítése az alkalmazás kódja nem ismeri a kockázatos.

## <a name="solution"></a>Megoldás

Ügyfél keretrendszerek és könyvtárak üzembe egy külső folyamat, amely az alkalmazások és szolgáltatások külső közötti proxyként funkcionál. A-proxyt üzembe helyezni az ugyanabban a gazdagép-környezetben az alkalmazás útválasztási, a rugalmasság, a biztonsági funkciók lehetővé, és minden állomás kapcsolatos hozzáférési korlátozások elkerülése érdekében. A diplomata minta segítségével szabványosítására és instrumentation kiterjesztése. A proxy figyelheti például várakozási ideje vagy erőforrás-használati metrikák, és a figyelés történik, az alkalmazás ugyanazon gazdagép-környezetben.

![](./_images/ambassador.png)

Funkciókat, amelyek a diplomata-t kiszervezzen az alkalmazás függetlenül is felügyelhetők. Frissítheti és a diplomata módosíthatja az alkalmazás régebbi működését megzavarása nélkül. Azt is lehetővé teszi, hogy külön, speciális csoportjai bevezetéséhez és karbantartásához biztonsági, hálózati vagy hitelesítési szolgáltatások, amelyek a diplomata helyezte.

Diplomata szolgáltatások telepíthető központilag a [oldalkocsi] [ sidecar] kísérő egy fogyasztó alkalmazás vagy szolgáltatás teljes életciklusát. Azt is megteheti Ha egy diplomata megosztott egy közös gazdagép több különálló folyamat, központilag telepíthető egy démon vagy a Windows-szolgáltatás. Ha a fogyasztó szolgáltatás van indexelése, a diplomata kell létrehozni külön tárolóként ugyanazon a gazdagépen, a megfelelő hivatkozások kommunikációra konfigurálva.

## <a name="issues-and-considerations"></a>Problémákat és szempontok

- A proxy ad némi késés többletterhelést okoz. Vegye figyelembe, hogy egy ügyféloldali kódtár, közvetlenül az alkalmazás által elindított jobb megközelítés-e.
- Vegye figyelembe a lehetséges hatása általánosított szolgáltatásainak a proxy. Például a diplomata újrapróbálkozások tudta kezelni, de, amely nem feltétlenül biztonságos kivéve, ha minden műveletnél az idempotent.
- Fontolja meg az eljárást, amely engedélyezi az ügyfél környezete felelt meg a proxy felé, valamint az ügyfélnek. Például a HTTP-kérelmek fejléceinek tilthatják le az újra gombra, vagy adja meg a próbálkozások maximális számát.
- Vegye figyelembe, hogyan fogja csomagba, és a-proxyt üzembe helyezni.
- Vegye figyelembe, hogy egyetlen megosztott példány minden ügyfél vagy egy példányát minden ügyfél számára.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes használni ezt a mintát

Használja ezt a mintát mikor meg:

- Több nyelvet vagy keretrendszert az ügyfél-kapcsolati funkciók közös beállításkészletre készítéséhez szükséges.
- Több területet ügyfél kapcsolat kapcsolatos kérdéseket infrastruktúra fejlesztők vagy más konkrétabb csoportokkal kiszervezése kell.
- Támogatja a felhő vagy a fürt hálózati kapcsolati követelményeinek egy örökölt alkalmazás vagy egy alkalmazásban, bonyolult módosítani kell.

Ez a minta nem alkalmasak lehetnek:

- Hálózati kérés várakozási ideje esetén fontos. A proxy bevezet némi többletterhelést okoz, noha a minimális, és egyes esetekben ez hatással lehet az alkalmazás.
- Ha ügyfél-kapcsolati funkciók egyetlen nyelvet használnak fel. Ebben az esetben a jobb megoldás lehet egy ügyféloldali kódtára a fejlesztési csapat terjesztett csomag.
- Amikor kapcsolati funkciók nem általánosítva, és az ügyfél alkalmazással szorosabb integrációt igényel.

## <a name="example"></a>Példa

Az alábbi ábrán látható, a kérést továbbítja az egy diplomata proxyn keresztül egy távoli szolgáltatási kérelmet. A diplomata biztosít útválasztási megtörje áramkör és naplózás. Meghívja a távoli szolgáltatást, és majd visszaadja a választ az ügyfélalkalmazás:

![](./_images/ambassador-example.png) 

## <a name="related-guidance"></a>Kapcsolódó útmutató

- [Oldalkocsi minta](./sidecar.md)

<!-- links -->

[circuit-breaker]: ./circuit-breaker.md
[resiliency-patterns]: ./category/resiliency.md
[sidecar]: ./sidecar.md
