---
title: "Útmutató az Azure-adatarchitektúrához"
description: 
author: zoinerTejada
ms:date: 02/12/2018
layout: LandingPage
ms.openlocfilehash: 848601f27faf56ea069852d8983e4d10fbad9d77
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/14/2018
---
# <a name="azure-data-architecture-guide"></a>Útmutató az Azure-adatarchitektúrához

Ez az útmutató az adatközpontú megoldások a Microsoft Azure-ban történő kialakításához szolgál egy strukturált megközelítéssel. Az útmutató az ügyfélesetekből származó tapasztalatok alapján készült.

## <a name="introduction"></a>Bevezetés

A felhő megváltoztatja az alkalmazások tervezésének módját, beleértve az adatok feldolgozását és tárolását is. A _többnyelvű perzisztencián_ alapuló megoldások több specializált, adott funkciók nyújtására optimalizált adattárat használnak egyetlen általános célú, egy megoldás összes adatát kezelő adatbázis helyett. Ennek eredményeképp a megoldás adatai más perspektívát öltenek. Nincsenek többé többszörös üzletilogika-rétegek, amelyek egyetlen adatrétegen végeznek írást és olvasást. Ehelyett a megoldások egy *adatfolyamaton* alapulnak, amely bemutatja, hogyan áramlanak keresztül az adatok a megoldáson, hol történik a feldolgozásuk, hol tárolódnak, és hogyan használja fel őket a folyamat következő összetevője. 

## <a name="how-this-guide-is-structured"></a>Az útmutató felépítése

Ez az útmutató egy alapvető különbözőségre, a *relációs* adatok és *nem relációs* adatok megkülönböztetésére épít. 

![](./images/guide-steps.svg)

A relációs adatok általában egy hagyományos RDBMS-ben vagy adattárházban tárolódnak. Előre definiált sémával („íráskor meghatározott séma”) rendelkeznek, amelyre megkötések vonatkoznak a hivatkozási integritás megőrzése érdekében. A legtöbb relációs adatbázis Structured Query Language-t (SQL) használ a lekérdezéshez. A relációs adatbázisokat használó megoldások közé tartozik az online tranzakciófeldolgozás (OLTP) és az online analitikus feldolgozás (OLAP).

Nem relációs adat bármely olyan adat, amely nem használja a hagyományos RDBMS-rendszerekben található [relációs modellt](https://en.wikipedia.org/wiki/Relational_model). Ide tartozhatnak kulcsértékadatok, JSON-adatok, grafikonadatok, idősorozat-adatok és más adattípusok. A *NoSQL* kifejezés olyan adatbázisokat jelöl, amelyek különféle típusú nem relációs adatok tárolására szolgálnak. A kifejezés azonban nem teljesen pontos, mivel számos nem relációs adattár támogat SQL-kompatibilis lekérdezéseket. A nem relációs adatok és a NoSQL-adatbázisok gyakran előtérbe kerülnek a *big data*-megoldásokkal kapcsolatban. A big data típusú architektúrák kialakításuknak köszönhetően képesek kezelni az olyan adatok bevitelét, feldolgozását és elemzését, amelyek túl nagyok vagy túl összetettek a hagyományos adatbázisrendszerek számára. 

E két fő kategórián belül az adatarchitektúra útmutatója az alábbi szakaszokat tartalmazza:

- **Alapelvek.** Áttekintő cikkek, amelyek bemutatják az ilyen típusú adatokkal folytatott munkához szükséges főbb fogalmakat.
- **Forgatókönyvek.** Adatforgatókönyvek reprezentatív gyűjteménye, amely a kapcsolódó Azure-szolgáltatásokat és az adott forgatókönyvnek megfelelő architektúrát is tárgyalja.
- **Technológiai lehetőségek.** Az Azure-on elérhető különféle adattechnológiák részletes összehasonlítása, ideértve a nyílt forráskódú lehetőségeket is. Az egyes kategóriákon belül egy képességmátrix, valamint a választáshoz szükséges fontosabb kritériumok találhatók, amelyek segítségével kiválaszthatja az adott forgatókönyvnek megfelelő technológiát.

Ennek az útmutatónak nem célja az adatelemzés vagy az adatbázis-elmélet oktatása – e témákban számos tankönyv érhető el. A célja ehelyett, hogy segítse Önt az adott forgatókönyvnek megfelelő adatarchitektúra vagy adatfolyamat kiválasztásában, valamint ezután az igényeinek leginkább megfelelő Azure-szolgáltatások és -technológiák kiválasztásában. Ha már eldöntötte, milyen architektúrát szeretne használni, továbbléphet a technológiai lehetőségekhez.

## <a name="traditional-rdbms"></a>Hagyományos RDBMS

### <a name="concepts"></a>Alapelvek

- [Relációs adatok](./concepts/relational-data.md) 
- [Tranzakciós adatok](./concepts/transactional-data.md) 
- [Szemantikai modellezés](./concepts/semantic-modeling.md) 

### <a name="scenarios"></a>Forgatókönyvek

- [Online analitikus feldolgozás (OLAP)](./scenarios/online-analytical-processing.md)
- [Online tranzakciófeldolgozás (OLTP)](./scenarios/online-transaction-processing.md) 
- [Adattárházak és adatpiacok](./scenarios/data-warehousing.md)
- [ETL](./scenarios/etl.md) 

## <a name="big-data-and-nosql"></a>Big Data és NoSQL

### <a name="concepts"></a>Alapelvek

- [Nem relációs adattárak](./concepts/non-relational-data.md)
- [CSV- és JSON-fájlok használata](./concepts/csv-and-json.md)
- [Big data-architektúrák](./concepts/big-data.md)
- [Bővített analitika](./concepts/advanced-analytics.md) 
- [Gépi tanulás nagy léptékben](./concepts/machine-learning-at-scale.md)

### <a name="scenarios"></a>Forgatókönyvek

- [Kötegelt feldolgozás](./scenarios/batch-processing.md)
- [Valós idejű feldolgozás](./scenarios/real-time-processing.md)
- [Szabad formátumú szöveges keresés](./scenarios/search.md)
- [Interaktív adatfeltárás](./scenarios/interactive-data-exploration.md)
- [Természetes nyelvek feldolgozása](./scenarios/natural-language-processing.md)
- [Idősorozattal kapcsolatos megoldások](./scenarios/time-series.md)

## <a name="cross-cutting-concerns"></a>Általános megfontolások

- [Adatátvitel](./scenarios/data-transfer.md) 
- [Helyszíni adatmegoldások kiterjesztése a felhőre](./scenarios/hybrid-on-premises-and-cloud.md) 
- [Adatmegoldások védelme](./scenarios/securing-data-solutions.md) 
