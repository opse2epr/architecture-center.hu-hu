---
title: Webalkalmazás-várólista-munkavégző architektúra stílus
description: Előnyeit, kihívást és ajánlott eljárások a webalkalmazás-várólista-munkavégző architektúrák ismerteti az Azure-on
author: MikeWasson
ms.openlocfilehash: 545472e71ffcd43717ad24af0dc9218a221ca910
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
ms.locfileid: "24539801"
---
# <a name="web-queue-worker-architecture-style"></a>Webalkalmazás-várólista-munkavégző architektúra stílus

Ebbe az architektúrába alapvető összetevői vannak egy **előtér-webkiszolgáló** ügyfélkérelmek, szolgál, és egy **munkavégző** , erőforrás-igényes feladatok, hosszan futó munkafolyamatok vagy kötegelt feladatot hajt végre.  A dolgozó keresztül kommunikál a előtér egy **üzenet-várólista**.  

![](./images/web-queue-worker-logical.svg)

Más összetevők, amelyek gyakran vannak építve az architektúra a következők:

- Egy vagy több adatbázist. 
- A gyorsítótár az adatbázisból tárolásához a gyors olvasása.
- Statikus tartalom szolgáltatásához CDN
- Távoli szolgáltatások, például az e-mailben vagy SMS-szolgáltatás. Gyakran ezek által biztosított harmadik felek számára.
- Identitásszolgáltató-hitelesítéshez.

A webes és feldolgozói olyan állapot nélküli is. A munkamenet-állapot egy elosztott gyorsítótár is tárolható. Hosszan futó munka a dolgozó aszinkron módon végezhető el. A munkavégző üzenetek várakozási által kiváltott, vagy kötegelt feldolgozásra ütemezhető. A dolgozó választható összetevőként. Ha hosszú ideig futó művelet, a munkavégző elhagyható.  

Az előtérbeli webes API-k állhat. Ügyféloldali a webes API képes használni egy egyoldalas alkalmazás, amely lehetővé teszi az AJAX-hívások, illetve egy webalkalmazást.

## <a name="when-to-use-this-architecture"></a>Mikor érdemes használni, ez az architektúra

A webalkalmazás-várólista-munkavégző architektúra általában felügyelt számítási szolgáltatás, vagy az Azure App Service szolgáltatásban, vagy az Azure Cloud Services segítségével van megvalósítva. 

Vegye figyelembe a architektúra stílus:

- Viszonylag egyszerű tartománnyal rendelkező alkalmazások.
- Az alkalmazásoknak néhány hosszan futó munkafolyamatok vagy kötegelt műveleteket.
- Ha használni kívánt felügyelt szolgáltatásokat, nem pedig infrastruktúra (IaaS) szolgáltatás.

## <a name="benefits"></a>Előnyök

- Viszonylag egyszerű architektúra, amely könnyen értelmezhető.
- Könnyű telepítése és kezelése.
- Egyértelmű elkülönítése vonatkozik.
- Az előtér a rendszer leválasztja a dolgozó aszinkron üzenetküldéshez használ.
- Az előtér- és a feldolgozói is méretezhető egymástól függetlenül.

## <a name="challenges"></a>Kihívásai

- Gondos tervezés nélkül az előtér- és a feldolgozói válhat nagy, egységes összetevők, amelyek a nehezen karbantartása és frissítése.
- Nem lehet rejtett függőségei, ha az előtér- és munkavégző adatok sémák vagy kódmodulok megosztása. 

## <a name="best-practices"></a>Ajánlott eljárások

- Az ügyfél egy tetszetős API tenni. Lásd: [API ajánlott tervezési eljárásokat][api-design].
- Automatikus skálázás módosítások a terhelés kezelésére. Lásd: [automatikus skálázás gyakorlati tanácsok][autoscaling].
- Gyorsítótár félig statikus adatok. Lásd: [gyakorlati tanácsok gyorsítótárazás][caching].
- A CDN és a gazdagép statikus tartalmat használja. Lásd: [CDN gyakorlati tanácsok][cdn].
- Adott esetben polyglot adatmegőrzési használja. Lásd: [használjon a legjobb adatok tárolásához a feladat][polyglot].
- Partíció adatot, ezzel növelve a méretezhetőséget, versengés csökkentése és a teljesítmény optimalizálása. Lásd: [gyakorlati tanácsok particionálás adatok][data-partition].


## <a name="web-queue-worker-on-azure-app-service"></a>Az Azure App Service Web-várólista-Worker

Ez a szakasz ismerteti a ajánlott webes-várólista-munkavégző architektúra, amely használja az Azure App Service. 

![](./images/web-queue-worker-physical.png)

Az előtér egy Azure App Service webalkalmazásba, lett megvalósítva, és a dolgozó, webjobs-feladat lett megvalósítva. A web app és a webjobs-feladat az App Service-csomag, amely a Virtuálisgép-példányok társított. 

Az üzenet-várólista vagy az Azure Service Bus, vagy az Azure Storage várólisták használható. (Az ábrán látható az Azure Storage üzenetsorába.)

Azure Redis Cache munkamenet-állapot és az egyéb kis késésű hozzáférést igénylő adatokat tárolja.

Az Azure CDN használatával például képek, CSS vagy HTML statikus tartalom gyorsítótárazása.

A tároláshoz válassza ki a tárolási technológiákat, az alkalmazás igényeinek leginkább illő. Használhat több tárolási technológiákat (polyglot adatmegőrzési). Az ábrán látható, az Azure SQL Database és Azure Cosmos DB ezt az elképzelést mutatja be.  

További részletekért lásd: [App Service web alkalmazás referencia-architektúrában][scalable-web-app].

### <a name="additional-considerations"></a>Néhány fontos megjegyzés

- Nem minden tranzakcióban halad át a várólista és a feldolgozói tárhelyre. Az előtér-webkiszolgáló közvetlenül egyszerű olvasási/írási műveleteket hajthat végre. Erőforrás-igényes feladatok vagy hosszan futó munkafolyamatok munkavállalók készültek. Bizonyos esetekben előfordulhat, nem kell egy munkavégző egyáltalán.

- A beépített automatikus skálázási funkciót, az App Service segítségével terjessze ki a Virtuálisgép-példányok száma. Ha az alkalmazás terhelésének előre jelezhető mintákat követi, használja az ütemezés alapján automatikus skálázási. Ha a terhelést előre nem látható, a metrikák-alapú automatikus skálázás szabályok használata.      

- Érdemes a webalkalmazást és a webjobs-feladat üzembe külön App Service-csomagokról. Ily módon, ezeket külön Virtuálisgép-példányok üzemelnek, és egymástól függetlenül is méretezhető. 

- Használjon külön App Service-csomagokról éles és tesztelését. Ellenkező esetben termelésre használatakor a csomagot, akkor a tesztet futtatja a virtuális gépek éles.

- Üzembe helyezési segítségével központi telepítések felügyeletéhez szükséges. Ez lehetővé teszi egy átmeneti helyet frissített verzióját telepítse, majd az új verzióra keresztül felcserélése. Ezenkívül lehetővé teszi vissza az előző verzió a felcserélése, ha a frissítés probléma merült fel.

<!-- links -->

[api-design]: ../../best-practices/api-design.md
[autoscaling]: ../../best-practices/auto-scaling.md
[caching]: ../../best-practices/caching.md
[cdn]: ../../best-practices/cdn.md
[data-partition]: ../../best-practices/data-partitioning.md
[polyglot]: ../design-principles/use-the-best-data-store.md
[scalable-web-app]: ../../reference-architectures/app-service-web-app/scalable-web-app.md