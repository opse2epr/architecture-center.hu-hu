---
title: Nagykövet minta
titleSuffix: Cloud Design Patterns
description: Olyan segítő szolgáltatásokat hozhat létre, amelyek egy otthoni használatra szánt szolgáltatás vagy alkalmazás nevében küldenek hálózati kéréseket.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: bbf83cdd4bc850c641a0559942f71013e9ce9553
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54486288"
---
# <a name="ambassador-pattern"></a>Nagykövet minta

Olyan segítő szolgáltatásokat hozhat létre, amelyek egy otthoni használatra szánt szolgáltatás vagy alkalmazás nevében küldenek hálózati kéréseket. A nagykövet szolgáltatás elképzelhető egy folyamaton kívüli proxyként, amely az ügyféllel közös elhelyezésű.

Ez a minta hasznos lehet a gyakori ügyfélkapcsolati feladatok, például a monitorozás, naplózás, útválasztás, biztonság (pl. TLS), valamint [rugalmassági minták][resiliency-patterns] kiszervezésére, nyelvtől független módon. Gyakran használják a hálózati képességek bővítésére örökölt alkalmazások vagy más, nehezen módosítható alkalmazások esetében. Továbbá lehetővé teszi, hogy a funkciókat egy specializált csapat implementálhassa.

## <a name="context-and-problem"></a>Kontextus és probléma

Rugalmas felhőalapú alkalmazások olyan funkciókat igényelnek, mint például [áramkör-megszakítás](./circuit-breaker.md), útválasztás, mérés és figyelési és a képesség a hálózati konfigurációs frissítések. Előfordulhat, hogy az örökölt alkalmazásokat vagy a meglévő kódtárakat nehéz vagy lehetetlen frissíteni ezen funkciók hozzáadásával, mert a kód már nem felügyelt, vagy a fejlesztőcsapat nem tudja egyszerűen módosítani.

Hálózati hívások esetén is jelentős konfigurációra lehet szükség a kapcsolat, hitelesítés és engedélyezés terén. Ha ezeket a hívásokat több, sokféle nyelvből és keretrendszerből álló alkalmazásban használjuk, a hívásokat minden példányra vonatkozóan külön kell konfigurálni. Ezenfelül lehetséges, hogy a hálózati és biztonsági funkciókat egy vállalaton belüli központi csapatnak kell majd felügyelnie. Nagy méretű kódbázis esetén kockázatos lehet a csapat számára egy olyan alkalmazáskódot frissíteni, amelyet nem ismer.

## <a name="solution"></a>Megoldás

Helyezze az ügyfél-keretrendszereket és -könyvtárakat egy külső folyamatba, amely proxyként funkcionál az alkalmazás és a külső szolgáltatások között. Telepítse a proxyt ugyanabban a gazdakörnyezetben, mint az alkalmazást, hogy így lehetővé tegye az útválasztás, a rugalmasság és a biztonsági funkciók irányítását, és elkerülje a kiszolgálóval kapcsolatos hozzáférési korlátozásokat. A nagykövet minta segítségével továbbá szabványosíthatja és kiterjesztheti a kialakítást. A proxy figyelni tudja a teljesítmény-mérőszámokat, például a várakozási időt vagy az erőforrás-használatot, a monitorozás pedig ugyanabban a gazdakörnyezetben történik, ahol az alkalmazás is van.

![A Nagykövet minta ábrája](./_images/ambassador.png)

A nagykövetre kiszervezett funkciók az alkalmazástól függetlenül is felügyelhetők. Frissítheti és módosíthatja a nagykövetet anélkül, hogy megzavarná az alkalmazás örökölt funkcióit. Azt is lehetővé teszi, hogy külön, specializált csapatok implementálhassák és tarthassák karban a nagykövetre ruházott biztonsági, hálózati és hitelesítési funkciókat.

Nagykövet-szolgáltatások telepíthető egy [oldalkocsi](./sidecar.md) , egy felhasználó alkalmazás vagy szolgáltatás életciklusának végigkísérésére. Vagy akár telepítheti a nagykövetet démonként vagy Windows szolgáltatásként, ha a nagyköveten több különálló, azonos gazdagépen futó folyamat osztozik. Ha a felhasználó szolgáltatás tárolóba van helyezve, a nagykövetet külön tárolóként kell létrehozni ugyanazon a gazdagépen, és konfigurálni kell a megfelelő hivatkozásokat a kommunikációhoz.

## <a name="issues-and-considerations"></a>Problémák és megfontolandó szempontok

- A proxy némi késéses többletterhelést okoz. Mérje fel, hogy esetleg jobb megközelítés-e egy ügyféloldali kódtár, amelyet közvetlenül az alkalmazás hív meg.
- Fontolja meg, milyen hatásai lehetnek, ha a proxy általánosított funkciókat is tartalmaz. Például a nagykövet kezelheti az újrapróbálkozásokat, de ez kockázatos lehet, hacsak nem idempotens mindegyik művelet.
- Gondolja át, hogy milyen mechanizmussal teheti lehetővé az ügyfél számára, hogy kontextust továbbítson a proxy felé úgy, hogy ez visszafelé is működjön. Például kapcsolja ki a HTTP-kérelmek fejlécei esetében az újrapróbálkozást, vagy adja meg az újrapróbálkozások maximális számát.
- Gondolja át, hogyan fogja csomagolni és üzembe helyezni a proxyt.
- Fontolja meg, hogy egyetlen megosztott példányt szeretne-e használni minden ügyfél számára, vagy inkább ügyfelenként külön-külön példányt.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes ezt a mintát használni?

Használja ezt a mintát, ha:

- Egy közös ügyfélkapcsolati funkciókészletet kell létrehoznia különböző nyelvekhez és keretrendszerekhez.
- Ki kell szerveznie az egyidejű ügyfélkapcsolati problémákat infrastruktúra-fejlesztőkre vagy más specializált csapatokra.
- Támogatnia kell a felhő- vagy a fürtkapcsolati követelményeket egy örökölt vagy nehezen módosítható alkalmazásban.

Nem érdemes ezt a mintát használni, ha:

- A hálózati kérések késései kritikus következményekkel járnak. A proxy némi többletterhelést okoz (habár minimális mértékben), és ez esetenként befolyásolhatja az alkalmazás működését.
- Ha az ügyfélkapcsolati funkciókat egyetlen nyelv használja. Ebben az esetben jobb megoldás lehet egy ügyféloldali kódtár, amelyet csomag formájában oszthat szét a fejlesztői csapatok között.
- Ha a kapcsolati funkciók nem általánosíthatók, és mélyebb integrációjuk szükséges az ügyfélalkalmazással.

## <a name="example"></a>Példa

Az alábbi ábrán látható, ahogyan egy alkalmazás nagykövetproxyn keresztül küld egy kérést egy távoli szolgáltatásnak. A nagykövet biztosítja az útválasztást, az áramköri megszakítást és a naplózást. Meghívja a távoli szolgáltatást, majd visszaadja a választ az ügyfélalkalmazásnak:

![A Nagykövet minta – példa](./_images/ambassador-example.png)

## <a name="related-guidance"></a>Kapcsolódó útmutatók

- [Oldalkocsi minta](./sidecar.md)

<!-- links -->

[resiliency-patterns]: ./category/resiliency.md
