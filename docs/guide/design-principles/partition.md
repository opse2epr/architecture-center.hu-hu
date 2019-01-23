---
title: Particionáljon a korlátok kiküszöböléséhez
titleSuffix: Azure Application Architecture Guide
description: Használjon particionálást az adatbázis-, hálózati és számítási korlátok megkerüléséhez.
author: MikeWasson
ms.date: 08/30/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19
ms.openlocfilehash: 76590d2a0c16df9d599e7d4a856a84b5e3bdcec8
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54486012"
---
# <a name="partition-around-limits"></a>Particionáljon a korlátok kiküszöböléséhez

## <a name="use-partitioning-to-work-around-database-network-and-compute-limits"></a>Particionálás használata az adatbázis-, hálózati és számítási korlátok megkerüléséhez

A felhőben minden szolgáltatás vertikális felskálázhatósága korlátozott. Az Azure-szolgáltatások korlátait [Az Azure-előfizetések és -szolgáltatások korlátozásai, kvótái és megkötései][azure-limits] című cikk ismerteti. A korlátok a magok számára, az adatbázisméretre, a lekérdezések átviteli sebességére és a hálózati átviteli sebességre vonatkoznak. Ha egy rendszer elér egy bizonyos méretet, beleütközhet egy vagy több ilyen korlátba. A korlátok megkerüléséhez particionálást használhat.

A rendszer számos módon particionálható, például:

- Adatbázis particionálása az adatbázis méretére, az adatokhoz kapcsolódó I/O-műveletekre vagy az egyszerre futó munkamenetek számára vonatkozó korlátok elkerüléséhez.

- Üzenetsor vagy üzenetbusz particionálása a kérések vagy az egyidejű kapcsolatok számára vonatkozó korlátok elkerüléséhez.

- App Service-webalkalmazás particionálása az App Service-csomagonkénti példányok számára vonatkozó korlátok elkerüléséhez.

Egy adatbázis particionálható *horizontálisan*, *vertikálisan* vagy *funkcionálisan*.

- Horizontális particionálás (más néven horizontális skálázás) esetén mindegyik partíció a teljes adathalmaz egy részhalmazának adatait tartalmazza. A partíciók adatsémája megegyezik. Például az A&ndash;M betűvel kezdődő ügyfelek egy partícióra kerülnek, az N&ndash;Z közöttiek pedig egy másikra.

- Vertikális particionáláskor mindegyik partíció az adattárban található elemekre vonatkozó mezőket egy részhalmazát tartalmazza. Az egyik partícióba például a gyakran használt mezőket helyezheti, egy másikba pedig a ritkábban használtakat.

- Funkcionális particionáláskor az adatok particionálása az alapján történik, hogy a rendszer egyes kapcsolódó kontextusai hogyan használják őket. Az egyik partícióban tárolhatja például a számlázási adatokat, egy másikban pedig a termékleltárral kapcsolatos adatokat. A sémák egymástól függetlenek.

Részletesebb útmutatásért tekintse meg az [adatparticionálással][data-partitioning-guidance] foglalkozó témakört.

## <a name="recommendations"></a>Javaslatok

**Particionálja az alkalmazás különböző részeit**. Az adatbázisok kézenfekvő jelöltek a particionálásra, érdemes lehet azonban megfontolni a tárterület, a gyorsítótár, az üzenetsorok és a számítási példányok particionálását is.

**Tervezze meg a partíciókulcsot a túlzott terhelés elkerülése érdekében**. Ha egy adatbázis particionálása után továbbra is egyetlen partíció kapja a kérések többségét, akkor még nem oldotta meg a problémát. Ideális esetben a terhelés egyenletesen oszlik el a partíciók között. A kivonatolást például ügyfélazonosító szerint végezze, ne pedig az ügyfél nevének első betűje alapján, mivel bizonyos betűk gyakoribbak. Ugyanaz az elv érvényes az üzenetsorok particionálására is. Olyan partíciókulcsot válasszon, amely egyenletesen osztja el az üzeneteket az üzenetsorok között. További információt a [horizontális skálázást][sharding] ismertető témakörben tekinthet meg.

**Az Azure-előfizetések és a -szolgáltatások korlátaihoz igazodva végezze a particionálást**. Az egyes összetevők és szolgáltatások mellett korlátozások vonatkoznak az előfizetésekre és erőforráscsoportokra is. A nagyon nagy alkalmazások esetén előfordulhat, hogy ezek korlátai szerint kell végeznie a particionálást.

**Különböző szinteken végezze a particionálást**. Tegyük fel, hogy egy virtuális gépen helyez üzembe egy adatbázis-kiszolgálót. A virtuális gép egy virtuális merevlemezzel rendelkezik, amelyet az Azure Storage támogat. A tárfiók egy Azure-előfizetéshez tartozik. Vegye figyelembe, hogy a hierarchia mindegyik lépésére korlátok vonatkoznak. Az adatbázis-kiszolgálóra a kapcsolatkészletre vonatkozó korlát lehet érvényes. A virtuális gépre processzor- és hálózati korlátok vonatkoznak. A tárolóra IOPS-korlátok érvényesek. Az előfizetésben korlátozott a virtuális gép magjainak a száma. Általában egyszerűbb a hierarchia alacsonyabb szintjén particionálni. Az előfizetés szintjén végzett particionálás csak nagy alkalmazások esetén szükséges.

<!-- links -->

[azure-limits]: /azure/azure-subscription-service-limits
[data-partitioning-guidance]: ../../best-practices/data-partitioning.md
[sharding]: ../../patterns/sharding.md
