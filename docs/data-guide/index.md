---
title: Útmutató az Azure-adatarchitektúrához
description: ''
author: zoinerTejada
ms.date: 02/12/2018
layout: LandingPage
ms.topic: landing-page
ms.openlocfilehash: 9c36762ae44e450fd2f827f59eceba2ce729e251
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/08/2019
ms.locfileid: "54112906"
---
# <a name="azure-data-architecture-guide"></a>Útmutató az Azure-adatarchitektúrához

Ez az útmutató az adatközpontú megoldások a Microsoft Azure-ban történő kialakításához szolgál egy strukturált megközelítéssel. Az útmutató az ügyfélesetekből származó tapasztalatok alapján készült.

## <a name="introduction"></a>Bevezetés

A felhő megváltoztatja az alkalmazások tervezésének módját, beleértve az adatok feldolgozását és tárolását is. A _többnyelvű perzisztencián_ alapuló megoldások több specializált, adott funkciók nyújtására optimalizált adattárat használnak egyetlen általános célú, egy megoldás összes adatát kezelő adatbázis helyett. Ennek eredményeképp a megoldás adatai más perspektívát öltenek. Nincsenek többé többszörös üzletilogika-rétegek, amelyek egyetlen adatrétegen végeznek írást és olvasást. Ehelyett a megoldások egy *adatfolyamaton* alapulnak, amely bemutatja, hogyan áramlanak keresztül az adatok a megoldáson, hol történik a feldolgozásuk, hol tárolódnak, és hogyan használja fel őket a folyamat következő összetevője.

## <a name="how-this-guide-is-structured"></a>Az útmutató felépítése

Ez az útmutató az adatkezelő megoldások két általános kategóriája, a *hagyományos RDBMS számítási feladatok* és a *big data-megoldások* köré épül fel.

**[Hagyományos RDBMS számítási feladatok](./relational-data/index.md)**. Ilyen számítási feladat például az online tranzakciófeldolgozás (OLTP) és az online analitikus feldolgozás (OLAP). Az OLTP-rendszerekben általában relációs adatokkal lehet találkozni, amelyek előre definiált sémával és a hivatkozási integritás megőrzését szolgáló megkötésekkel rendelkeznek. Gyakran a cégen belüli különböző forrásokból egy adattárházba gyűjtik az adatokat, és egy ETL-folyamattal helyezik át és konvertálják a forrásadatokat.

![Hagyományos RDBMS számítási feladatok](./images/guide-rdbms.svg)

**[Big data-megoldások](./big-data/index.md)**. A big data típusú architektúrát olyan adatok betöltésére, feldolgozására és elemzésére tervezték, amelyek túl nagyok vagy összetettek lennének a hagyományos adatbázisrendszerek számára. Az adatok feldolgozása történhet kötegelve vagy valós időben. A big data-megoldások általában nagy mennyiségű nem relációs adat feldolgozását végzik (például kulcs-érték típusú adatok, JSON-dokumentumok vagy idősorozat-adatok). A hagyományos RDBMS-rendszerek általában nem alkalmasak ilyen típusú adatok tárolására. A *NoSQL* kifejezés olyan adatbázis-típusokat jelöl, amelyek nem relációs adatok tárolására szolgálnak. (A kifejezés nem teljesen pontos, mivel számos nem relációs adattár támogat SQL-kompatibilis lekérdezéseket.)

![Big data-megoldások](./images/guide-big-data.svg)

A két kategória nem zárja ki egymást kölcsönösen, és van közöttük némi átfedés, mégis úgy gondoljuk, hogy alkalmas keretet biztosítanak a téma bemutatásához. Az útmutató bemutatja az egyes kategóriák **gyakori forgatókönyveit**, valamint a kapcsolódó Azure-szolgáltatásokat és az adott forgatókönyvnek megfelelő architektúrát. Az útmutató ezenkívül összehasonlítja az Azure-beli adatkezelő megoldásokhoz rendelkezésre álló **technológiai lehetőségeket**, beleértve a nyílt forráskódú lehetőségeket is. Az egyes kategóriákon belül egy képességmátrix, valamint a választáshoz szükséges fontosabb kritériumok találhatók, amelyek segítségével kiválaszthatja az adott forgatókönyvnek megfelelő technológiát.

Ennek az útmutatónak nem célja az adatelemzés vagy az adatbázis-elmélet oktatása – e témákban számos tankönyv érhető el. A célja ehelyett, hogy segítse Önt az adott forgatókönyvnek megfelelő adatarchitektúra vagy adatfolyamat kiválasztásában, valamint ezután az igényeinek leginkább megfelelő Azure-szolgáltatások és -technológiák kiválasztásában. Ha már eldöntötte, milyen architektúrát szeretne használni, továbbléphet a technológiai lehetőségekhez.
