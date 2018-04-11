---
title: Tervezzen mindent redundánsra
description: Kerülje a hibaérzékeny pontokat redundancia épület az alkalmazásba.
author: MikeWasson
layout: LandingPage
ms.openlocfilehash: 05ccf78c2cfbcd4e2d26200e70463d388d54f671
ms.sourcegitcommit: a7aae13569e165d4e768ce0aaaac154ba612934f
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/30/2018
---
# <a name="make-all-things-redundant"></a>Tervezzen mindent redundánsra

## <a name="build-redundancy-into-your-application-to-avoid-having-single-points-of-failure"></a>Az alkalmazásba, hogy kerülje az egyetlen ponton felmerülő hibákat redundancia összeállítása

Egy rugalmas alkalmazás útvonalak hibát körül. A kritikus elérési utak azonosítását az alkalmazásban. Nincs redundancia az elérési út minden ponton? Ha egy alrendszer nem sikerül, az alkalmazás meghiúsul keresztül valamilyen más?

## <a name="recommendations"></a>Javaslatok 

**Fontolja meg az üzleti követelmények**. Egy az rendszerbe épített redundanciájának hatással lehet, költségeket és bonyodalmakat is. Az architektúra tájékoztatni kell által az üzleti követelményeinek, például a helyreállítási idő célkitűzése (RTO). Például egy több területi drágább, mint egyetlen területi központi telepítések, és bonyolultabb kezeléséhez. Szüksége lesz a feladatátvételi és a feladat-visszavétel üzemeltetési eljárásokat. A további költségek és az egyes üzleti forgatókönyvek és mások számára nem indokolt lehet.

**Helyi virtuális gép egy terheléselosztó mögött**. Egyetlen virtuális gép ne használjon a kritikus fontosságú munkaterhelésekhez. Ehelyett a több virtuális gép egy terheléselosztó mögött elhelyezni. Ha a virtuális gép nem érhető el, a terheléselosztó osztja el a forgalmat a fennmaradó kifogástalan állapotú virtuális gépek. Ez a konfiguráció központi telepítéséről további tudnivalókért lásd: [több virtuális géphez, amely méretezhetőséget és a rendelkezésre állási][multi-vm-blueprint].

![](./images/load-balancing.svg)

**Adatbázisok replikálása**. Az Azure SQL Database és Cosmos DB automatikusan replikálja az adatokat a régión belül, és engedélyezheti a georeplikáció régiók között. Ha egy infrastruktúra-szolgáltatási adatbázis megoldást használ, válasszon egyet, amely támogatja a replikációt és feladatátvételt, például a [SQL Server Always On rendelkezésre állási csoportok][sql-always-on]. 

**A georeplikáció engedélyezéséhez**. A georeplikáció [Azure SQL Database] [ sql-geo-replication] és [Cosmos DB] [ cosmosdb-geo-replication] hozza létre, egy vagy több másodlagos olvasható replika adatait másodlagos régióban. Nem tervezett kimaradás esetén az adatbázis átveheti a másodlagos régióba az írásokhoz.

**A rendelkezésre állás érdekében partíció**. Adatbázis particionálási gyakran használják a méretezhetőség javítása érdekében, de azt is tovább fejlesztheti rendelkezésre állását. Ha egy shard leáll, a más szilánkok továbbra is elérhetők. Egy shard hibája miatt csak megszakítja majd a teljes tranzakció egy részét. 

**Egynél több régióban telepítendő**. A legnagyobb rendelkezésre állást, az csak egy régió az alkalmazást telepíti. Ily módon a ritka eset, amikor probléma hatással van egy teljes terület, az alkalmazás átveheti egy másik régióban. Az alábbi ábrán egy több területi alkalmazás Azure Traffic Manager feladatátvételi kezelésére.

![](images/failover.svg)

**Előtér / háttér feladatátvételi szinkronizálása**. Azure Traffic Manager használatával az előtér feladatátvételt. Ha egyetlen előtér nem érhető el, a Traffic Manager átirányítja az új kérelmek a másodlagos régióba. Az adatbázis-megoldásától függően szükség lehet koordinálja az adatbázis feladatátvétele. 

**Használja az Automatikus feladatátvétel, de manuális feladat-visszavétel**. Automatikus feladatátvétel, de nem automatikus feladat-visszavétel a Traffic Manager használata. Automatikus feladat-visszavétel hordoz magában, ha fennáll a kockázata, akkor előfordulhat, hogy váltson át az elsődleges régióban, mielőtt a régióban teljesen kifogástalan. Ehelyett ellenőrizze, hogy minden alkalmazás alrendszer kifogástalan előtt manuálisan visszavétele. Is attól függően, hogy az adatbázis szükség lehet Adatkonzisztencia ellenőrzése előtt vissza sikertelenek lesznek.

**A Traffic Manager közé tartozik a redundancia**. A TRAFFIC Manager ponttá lehetséges hiba. Tekintse át a Traffic Manager SLA-t, és határozza meg, hogy kizárólag a Traffic Manager segítségével megfelel-e az üzleti követelményeinek, a magas rendelkezésre állás érdekében. Ha nem, akkor vegyen fel egy másik forgalom felügyeleti megoldás, a feladat-visszavételre. Ha az Azure Traffic Manager szolgáltatás nem sikerül, módosítsa a CNAME rekordot a DNS-ben, hogy a többi felügyeleti szolgálat mutasson.



<!-- links -->

[multi-vm-blueprint]: ../../reference-architectures/virtual-machines-windows/multi-vm.md

[cassandra]: http://cassandra.apache.org/
[cosmosdb-geo-replication]: /azure/cosmos-db/distribute-data-globally
[sql-always-on]: https://msdn.microsoft.com/library/hh510230.aspx
[sql-geo-replication]: /azure/sql-database/sql-database-geo-replication-overview
