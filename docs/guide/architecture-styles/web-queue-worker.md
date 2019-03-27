---
title: Webüzenetsor-feldolgozó architektúrastílus
titleSuffix: Azure Application Architecture Guide
description: Előnyeit, kihívásait és ajánlott eljárások a Webüzenetsor-feldolgozó architektúráinak ismerteti az Azure-ban.
author: MikeWasson
ms.date: 08/30/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19
ms.openlocfilehash: b471d270af09df7ffd58dfdd49e7d03d05bfe582
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299038"
---
# <a name="web-queue-worker-architecture-style"></a>Webüzenetsor-feldolgozó architektúrastílus

Az ilyen architektúrák alapelemei: egy **webes előtér**, amely kiszolgálja az ügyfélkérelmeket, és egy **feldolgozó**, amely erőforrás-igényes munkákat, hosszan futó munkafolyamatokat vagy kötegelt feladatokat hajt végre.  A webes előtér egy **üzenetsor** segítségével kommunikál a feldolgozóval.

![Webüzenetsor-feldolgozó architektúrastílus logikai diagramja](./images/web-queue-worker-logical.svg)

Az ebbe az architektúrába gyakran beillesztett más összetevők:

- Egy vagy több adatbázis.
- Egy gyorsítótár az adatbázis adatainak tárolására a gyors olvasás érdekében.
- CDN a statikus tartalom kiszolgálásához
- Távoli szolgáltatások, például e-mail- vagy SMS-szolgáltatás. Ezeket gyakran egy külső fél biztosítja.
- Identitásszolgáltató a hitelesítéshez.

A web és a feldolgozó is állapot nélküli. A munkamenet-állapot tárolható egy megosztott gyorsítótárban. A hosszan futó munkákat a feldolgozó aszinkron módon végzi el. A feldolgozó aktiválható üzenetekkel az üzenetsoron, vagy futhat egy ütemezésben a kötegelt feldolgozáshoz. A feldolgozó egy választható összetevő. Ha nincs hosszan futó művelet, a feldolgozó elhagyható.

Az előtér állhat egy webes API-ból. Ügyféloldalon a webes API-t használhatja egy AJAX-hívásokat indító egyoldalas alkalmazás vagy egy natív ügyfélalkalmazás.

## <a name="when-to-use-this-architecture"></a>Mikor érdemes ezt az architektúrát használni?

A webüzenetsor-feldolgozó architektúra implementálása általában felügyelt számítási szolgáltatások használatával történik, vagy az Azure App Service vagy az Azure Cloud Services segítségével.

Fontolja meg ennek az architektúrastílusnak a használatát a következőkhöz:

- Alkalmazások viszonylag egyszerű tartománnyal.
- Alkalmazások néhány hosszan futó munkafolyamattal vagy kötegelt műveletekkel.
- Ha felügyelt szolgáltatásokat szeretne használni szolgáltatott infrastruktúra (IaaS) helyett.

## <a name="benefits"></a>Előnyök

- Viszonylag egyszerű és könnyen érthető architektúra.
- Könnyen telepíthető és felügyelhető.
- A kockázatok egyértelmű elkülönítése.
- Az előteret a rendszer aszinkron üzenetküldés segítségével választja le a feldolgozóról.
- Az előtér és a feldolgozó egymástól függetlenül skálázható.

## <a name="challenges"></a>Problémák

- Gondos tervezés nélkül az előtér és a feldolgozó hatalmas, monolitikus, végül nehezen karbantartható és frissíthető összetevővé válhat.
- Lehetnek rejtett függőségek, ha az előtér és a feldolgozó közös sémákat vagy kódmodulokat használ.

## <a name="best-practices"></a>Ajánlott eljárások

- Jól megtervezett API közzététele az ügyfelek számára. Tekintse meg az [API-tervezés – Ajánlott eljárások][api-design] szakaszt.
- Automatikus skálázás a terhelés változásának kezelésére. Lásd az [automatikus skálázással kapcsolatos ajánlott eljárásokat][autoscaling].
- Gyorsítótárazza a félig statikus adatokat. Lásd a [gyorsítótárazás ajánlott eljárásait][caching].
- CDN használata a statikus tartalom üzemeltetéséhez. Tekintse meg az [Ajánlott tanácsok az Azure CDN-nel kapcsolatban][cdn] szakaszt.
- Többnyelvű adatmegőrzés használata, ha szükséges. Tekintse meg a [Használja a feladathoz legmegfelelőbb adattárat][polyglot] szakaszt.
- Adatok particionálása a méretezhetőség növeléséhez, a versengés kiküszöböléséhez és a teljesítmény optimalizálásához. Tekintse meg az [Ajánlott adatparticionálási eljárások][data-partition] szakaszt.

## <a name="web-queue-worker-on-azure-app-service"></a>A webüzenetsor-feldolgozó az Azure App Service-en

Ez a szakasz ismertet egy ajánlott, az Azure App Service-t használó webüzenetsor-feldolgozót.

![Webüzenetsor-feldolgozó architektúrastílus fizikai diagramja](./images/web-queue-worker-physical.png)

Az előtér egy ismert Azure App Service webalkalmazásként, a feldolgozó pedig WebJob-feladatként van implementálva. A webalkalmazás és a WebJob is társítva van egy App Service-csomaghoz, amely biztosítja a virtuálisgép-példányokat.

Az üzenetsorhoz használhat Azure Service Bus- vagy Azure Storage-üzenetsort is. (Az ábrán egy Azure Storage-üzenetsor látható.)

Az Azure Redis Cache tárolja a munkamenet állapotát, és más, alacsony késésű elérést igénylő adatokat.

Az Azure CDN használatával gyorsítótárazható statikus tartalom, például képek, CSS vagy HTML.

A tároláshoz válassza ki az alkalmazás igényeinek legmegfelelőbb tárolási technológiákat. Használhat több tárolási technológiát (többnyelvű adatmegőrzés). Az elképzelés ábrázolására az ábra bemutatja az Azure SQL Database-t és az Azure Cosmos DB-t.

További részletek: [App Service-webalkalmazás referenciaarchitektúrája][scalable-web-app].

### <a name="additional-considerations"></a>Néhány fontos megjegyzés

- Nem minden tranzakciónak kell áthaladnia az üzenetsoron és a feldolgozón a tároló felé. A webes előtér közvetlenül hajthat végre egyszerű olvasási/írási műveleteket. A feldolgozók erőforrás-igényes feladatokhoz vagy hosszan futó munkafolyamatokhoz lettek kialakítva. Bizonyos esetekben előfordulhat, hogy nincs is szükség feldolgozóra.

- Az App Service beépített automatikus skálázási szolgáltatásával horizontálisan felskálázhatja a virtuálisgép-példányok számát. Ha az alkalmazás terhelése előre megjósolható mintákat követ, használjon ütemezésalapú automatikus skálázást. Ha a terhelés nem jósolható meg előre, használjon mérőszám-alapú automatikus méretezési szabályokat.

- Érdemes lehet a webalkalmazást és a WebJob-feladatot külön App Service-csomagba helyezni. Ily módon üzemeltethetők külön virtuálisgép-példányokon, és skálázhatók egymástól függetlenül.

- Az éles üzemhez és a teszteléshez használjon külön App Service-csomagot. Ha ugyanis ugyanazt a csomagot használja az éles üzemhez és a teszteléshez, akkor a tesztek élesben működő virtuális gépeken futnak.

- Az üzembe helyezések kezeléséhez használjon üzembehelyezési pontokat. Ez lehetővé teszi, hogy telepítsen egy frissített verziót egy átmeneti helyen, majd átváltson az új verzióra. Így arra is lehetősége van, hogy visszaváltson a korábbi verzióra, ha probléma adódna a frissítéssel.

<!-- links -->

[api-design]: ../../best-practices/api-design.md
[autoscaling]: ../../best-practices/auto-scaling.md
[caching]: ../../best-practices/caching.md
[cdn]: ../../best-practices/cdn.md
[data-partition]: ../../best-practices/data-partitioning.md
[polyglot]: ../design-principles/use-the-best-data-store.md
[scalable-web-app]: ../../reference-architectures/app-service-web-app/scalable-web-app.md