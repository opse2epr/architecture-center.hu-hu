---
title: Mikroszolgáltatás-architektúrákat bemutatása
description: Mikroszolgáltatás-architektúrákat bemutatása.
author: MikeWasson
ms.date: 02/26/2019
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: microservices
ms.openlocfilehash: fbb79ea2f9a4822fb221b99ce5e94ce78067fd1f
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298421"
---
# <a name="introduction-to-microservices-architectures"></a>Mikroszolgáltatás-architektúrákat bemutatása

A mikroszolgáltatási architektúra kisebb, autonóm szolgáltatások gyűjteményéből áll. Mindegyik szolgáltatás önálló, és egyetlen üzleti képességet valósít meg. Íme néhány a mikroszolgáltatások meghatározó jellemzői közül:

- A mikroszolgáltatási architektúrában a szolgáltatások kicsik, függetlenek és lazán összekapcsoltak.
- A mikroszolgáltatások általában elég kis méretűek ahhoz, hogy fejlesztők egy kis csoportja alkalmas legyen a megírásukra és fenntartásukra.
- A szolgáltatások egymástól függetlenül is üzembe helyezhetők. Egy csapat a meglévő szolgáltatást a teljes alkalmazás újraépítése és ismételt üzembe helyezése nélkül frissítheti.
- A szolgáltatások a saját adataik vagy külső állapotuk fenntartásáért felelősek. Ez eltér a hagyományos modelltől, ahol egy külön adatréteg kezeli az adatmegőrzést.
- A szolgáltatások jól meghatározott API-k használatával kommunikálnak egymással. Az egyes szolgáltatások belső megvalósítási részletei rejtve vannak a többi szolgáltatás elől.
- Nem kell, hogy a szolgáltatások ugyanazt a technológiát, kódtárat vagy keretrendszert használják.

![Mikroszolgáltatás-alapú logikai architektúrája](../guide/architecture-styles/images/microservices-logical.svg)

<!-- markdownlint-disable MD026 -->

## <a name="why-build-microservices"></a>Miért érdemes mikroszolgáltatásokat létrehozni?

A Mikroszolgáltatások számos hasznos előnyt biztosítja:

- **Rugalmasság.** Mivel a mikroszolgáltatások egymástól függetlenül üzemelnek, a hibajavítások és a funkciók kiadása egyszerűbben felügyelhető. A szolgáltatásokat a teljes alkalmazás ismételt üzembe helyezése nélkül frissítheti, illetve visszavonhat egy frissítést, ha valami probléma merül fel. Számos hagyományos alkalmazásnál, ha egy hiba merül fel az egyik részében, az megállíthatja a teljes kiadási folyamatot, amely eredményeként az új funkciók bevezetéséhez meg kell várni a hibajavítás integrálását, tesztelését és kiadását.  

- **Kis kód, kis csapatok.** A mikroszolgáltatásoknak elég kis méretűnek kell lenniük ahhoz, hogy egyetlen termékfejlesztő-csapat képes legyen elkészíteni, tesztelni és üzembe helyezni azokat. A kis kódbázisok könnyebben átláthatóak. A nagy, monolitikus alkalmazások esetén idővel gyakran a kódfüggőségek összekuszálódnak, ezért az új funkciók hozzáadásához a kódot számos helyen módosítani kell. Mivel nem osztoznak a kódon vagy adattárakon, a mikroszolgáltatási architektúrák minimalizálják a függőségeket, így az új funkciók hozzáadása egyszerűbben elvégezhető. A kis csapatméret elősegíti a nagyobb fokú rugalmasságot. A „kétpizzás szabály” szerint egy csapatnak elég kicsinek kell lennie ahhoz, hogy két pizzával jóllakjon mindenki. Természetesen ez nem egy pontos metrika, és a csapat étvágyától is függ! A lényeg azonban az, hogy a nagy csoportok általában kevésbé hatékonyak, mivel a kommunikáció lassabb, a vezetési költségek magasabbak és a rugalmasság csökken.  

- **Technológiák vegyítése**. A csapatok kiválaszthatják a szolgáltatásukhoz legmegfelelőbb technológiát, illetve szükség szerint ezek elegyét. 

- **Rugalmasság**. Ha egy adott mikroszolgáltatás elérhetetlenné válik, az nem zavarja meg a teljes alkalmazást, amennyiben a fölérendelt mikroszolgáltatások úgy vannak megtervezve, hogy megfelelően kezeljék a hibákat (például áramköri megszakítás bevezetésével).

- **Méretezhetőség**. A mikroszolgáltatási architektúra lehetővé teszi, hogy az összes mikroszolgáltatást egymástól függetlenül méretezhesse. Így horizontálisan felskálázhatja a több erőforrást igénylő alrendszereket anélkül, hogy a teljes alkalmazást méreteznie kellene. Ha tárolókon belül helyez üzembe szolgáltatásokat, a mikroszolgáltatásokat nagyobb sűrűségben helyezheti egyetlen gazdagépre, így hatékonyabban használhatja fel az erőforrásokat.

- **Adatelkülönítés**. A sémafrissítések végrehajtása sokkal egyszerűbb, mivel csak egyetlen mikroszolgáltatást érintenek. A monolitikus alkalmazásokban a sémafrissítések nagy nehézséget jelenthetnek, mivel az alkalmazás különböző részei ugyanazokat az adatokat használhatják, kockázatossá téve ezzel a séma módosítását.

## <a name="when-should-i-build-microservices"></a>Mikor érdemes mikroszolgáltatásokat létrehozni?

Vegye figyelembe a mikroszolgáltatási architektúra:

- Kiadások gyors megjelenését igénylő, nagy méretű alkalmazások.
- Összetett alkalmazások, amelyeknek jól skálázhatónak kell lenniük.
- Részletes tartománnyal vagy több altartománnyal rendelkező alkalmazások.
- Kis fejlesztői csapatokból álló vállalatok.

## <a name="challenges"></a>Problémák

A mikroszolgáltatások előnyeit nem adják ingyen. Íme néhány a is érdemes figyelembe venni, mielőtt alkalmazná a mikroszolgáltatási architektúra kihívást jelent.

- **Összetettség**. A mikroszolgáltatás-alkalmazások több mozgó részből állnak, mint az egyenértékű monolitikus alkalmazások. Az egyes szolgáltatások egyszerűbbek, de a rendszer egésze összetettebb.

- **Fejlesztési és tesztelési**. Egy kis szolgáltatás, amely más függő szolgáltatásokra támaszkodik, mint egy írása eltérő megközelítést igényel írása egy hagyományos monolitikus vagy többrétegű alkalmazást. Meglévő eszközöket nem mindig tervezték függőségei dolgozhat. A szolgáltatás határain túlnyúló újrabontás bonyolult lehet. A szolgáltatásfüggőségek tesztelése is kihívást jelent, különösen akkor, ha az alkalmazás gyorsan fejlődik.

- **Irányítás hiánya**. A mikroszolgáltatások építésének decentralizált megközelítése rendelkezik előnyökkel, de problémákhoz is vezethet. Előfordulhat, hogy végül olyan sok különböző nyelv és keretrendszer lesz jelen, hogy nehézkessé válik az alkalmazás karbantartása. Hasznos lehet néhány projektszintű szabvány bevezetése, a csapat rugalmasságának túlzott korlátozása nélkül. Ez különösen az egész rendszert érintő funkciókra vonatkozik (például a naplózásra).

- **Hálózati torlódás és késleltetés**. A sok kis részletes szolgáltatás használata több szolgáltatások közötti kommunikációt eredményezhet. Emellett ha a szolgáltatásfüggőségek láncolata túl hosszú (A szolgáltatás meghívja a B-t, amely meghívja a C-t, stb.), a további késleltetés problémává válhat. Gondosan kell megterveznie az API-kat. Kerülje a túlságosan forgalmas API-kat, gondoljon a szerializálási formátumokra, és keressen helyet az aszinkron kommunikáció használatához.

- **Adatintegritás**. Az egyes mikroszolgáltatások a saját adataik megőrzéséért felelősek. Ennek eredményeként az adatkonzisztencia problémát jelenthet. Ahol lehetséges, támogassa a végső konzisztenciát.

- **Felügyelet**. Ahhoz, hogy sikeresen használhassa a mikroszolgáltatásokat, fejlett DevOps-kultúra szükséges. A szolgáltatások korrelált naplózása kihívást jelenthet. A naplózásnak általában több szolgáltatás-hívást kell korrelálnia egyetlen felhasználói művelethez.

- **Verziókezelés**. Egy szolgáltatás frissítéseinek tilos megszüntetnie a tőle függő szolgáltatásokat. Bármikor frissülhet akár több szolgáltatás is, ezért körültekintő tervezés nélkül problémák merülhetnek fel a visszamenőleges vagy a jövőbeli kompatibilitással kapcsolatban.

- **Készségek**. A mikroszolgáltatások több helyre elosztott rendszerek. Gondosan mérlegelje, hogy a csapat rendelkezik-e a sikerességhez szükséges szakértelemmel és tapasztalattal.

## <a name="next-steps"></a>További lépések

- A legnagyobb kihívás a mikroszolgáltatási egyik, a megfelelő szolgáltatáshatárok létrehozásához. A cikkek alapján [mikroszolgáltatások modellezés](./model/domain-analysis.md) megjelenítése a tartományvezérelt megközelítés erre a problémára.
