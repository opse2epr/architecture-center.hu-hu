---
title: "Partíció körül korlátok"
description: "Használjon particionálás kerülő adatbázis, hálózati, és számítási korlátok"
author: MikeWasson
layout: LandingPage
ms.openlocfilehash: 4371490385b24230551bf17db0075052f320b574
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="partition-around-limits"></a>Partíció körül korlátok

## <a name="use-partitioning-to-work-around-database-network-and-compute-limits"></a>Használjon particionálás kerülő adatbázis, hálózati, és számítási korlátok

A felhőben a összes services lehetőségeit növelheti a korlátokkal rendelkeznek. Az Azure szolgáltatásra vonatkozó korlátozások vannak dokumentálva [Azure-előfizetés és szolgáltatási korlátok, kvóták és megkötések][azure-limits]. Korlátok közé tartozik a magok, az adatbázis mérete, a lekérdezés átviteli sebesség és a hálózati teljesítményt száma. Ha a rendszer növekedésének megfelelően nagy, határaiba legalább egy, a működés felső korlátjának. Particionálás kerülő használja ezeket a határértékeket.

A rendszer, például a particionálásához számos módja van:

- Partícióazonosító adatbázis adatbázis mérete, a adatok i/o vagy a munkamenetek számát vonatkozó korlátozások elkerülése érdekében.

- A partíció egy várólista vagy üzenet busz létesített egyidejű kapcsolatok számát vagy a kérések száma vonatkozó korlátozások elkerülése érdekében.

- A partíció egy App Service webalkalmazás az App Service-csomagot /-példányok száma vonatkozó korlátozások elkerülése érdekében. 

Egy adatbázis lehet particionálni *vízszintesen*, *függőleges*, vagy *funkcionálisan*.

- A vízszintes particionálás, más néven a horizontális, mindegyik partíció egy részét a teljes adatkészlet adatokat tároló. A partíciók ugyanazon adatok séma megosztani. Például az ügyfelek A kezdődő&ndash;M belép egy partíciót, N&ndash;Z másik partícióra.

- A vertikális particionálás, mindegyik partíció rendelkezik az elemeknek a mezők az adattárban. Például helyezze a gyakran használt mezőket, egy partíciót, és kevesebb mint egy másik gyakran használt mezőket.

- Funkcionális particionálás, az adatok particionálása megfelelően felhasználásukról minden kötött környezet által a rendszerben. Például tárolja egy partíció és a termék leltáradatok egy másik számla adatait. A sémákban egymástól függetlenül működnek.

Részletes útmutatást, lásd: [adatparticionálás][data-partitioning-guidance].

## <a name="recommendations"></a>Javaslatok

**Az alkalmazás különböző részei partícióazonosító**. Adatbázisok egy nyilvánvaló jelölt particionálás, de is vegye figyelembe a tárolás, a gyorsítótár, a várólisták, majd példányok számítási.

**A partíciós kulcs interaktív területek elkerülése érdekében tervezése**. Ha Ön egy adatbázis partícióazonosító, de egy shard továbbra is lekérdezi a legtöbb, a kérelmeket, majd, még nem lehet megoldani a problémát. Ideális esetben terhelés lekérdezi egyenletesen elosztott összes partíciójára. Például ügyfél-azonosító szerint kivonatoló és nem első betűjének a felhasználói név, mivel bizonyos betűk további gyakran használják. Ugyanaz az elv érvényes particionálásakor az üzenet-várólista. Válassza ki, hogy az üzenetek még akkor is, terjesztési a várólisták szétosztva partíciós kulcs. További információkért lásd: [horizontális][sharding].

**Azure-előfizetés és a szolgáltatásra vonatkozó korlátozások partíció**. Az egyes összetevőkkel és szolgáltatásokkal korlátokkal rendelkeznek, de is korlátai előfizetésekhez és erőforráscsoportokhoz. Nagyon nagy alkalmazások esetén szükség lehet partíció körül ezeket a határértékeket.  

**Különböző szinteken partíció**. Fontolja meg a virtuális gép telepített adatbázis-kiszolgálót. A virtuális gép Azure Storage által támogatott virtuális merevlemez rendelkezik. A tárfiók Azure-előfizetéshez tartozik. Figyelje meg, hogy rendelkezik-e a hierarchiában lévő egyes lépések korlátok. Az adatbázis-kiszolgáló lehet egy kapcsolat szálkészletkorlátjánál. Virtuális gépek Processzor rendelkezik, és a hálózati korlátok. Tárolási IOPS-korlátok vonatkoznak rendelkezik. Az előfizetés korlátok rendelkezzen a virtuális gép magok száma. Általában célszerű a hierarchia alacsonyabb szintjén levő partíció könnyebben. Az előfizetés szintjén partíció csak nagy alkalmazások kell. 

<!-- links -->

[azure-limits]: /azure/azure-subscription-service-limits
[data-partitioning-guidance]: ../../best-practices/data-partitioning.md
[sharding]: ../../patterns/sharding.md

 