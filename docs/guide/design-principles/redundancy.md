---
title: Tervezzen mindent redundánsra
titleSuffix: Azure Application Architecture Guide
description: Redundanciát építve az alkalmazásba elkerülheti a kritikus meghibásodási pontokat.
author: MikeWasson
ms.date: 08/30/2018
ms.custom: seojan19
ms.openlocfilehash: 784413f637e4f857a2ba3775f3d4de66c91fbd95
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/08/2019
ms.locfileid: "54111845"
---
# <a name="make-all-things-redundant"></a>Tervezzen mindent redundánsra

## <a name="build-redundancy-into-your-application-to-avoid-having-single-points-of-failure"></a>Redundanciát építve az alkalmazásba elkerülheti a kritikus meghibásodási pontokat

A rugalmas alkalmazások átirányítással megkerülik a hibákat. Azonosítsa a kritikus útvonalakat az alkalmazásban. Van redundancia az útvonal minden pontján? Ha egy alrendszer meghiúsul, akkor az alkalmazás feladatainak átvétele megoldott?

## <a name="recommendations"></a>Javaslatok

**Vegye figyelembe az üzleti követelményeket**. A rendszerbe épített redundancia mennyisége befolyásolhatja a költségeket és az összetettséget. Az architektúrának az üzleti követelményeken kell alapulnia, például a helyreállítási időre vonatkozó célkitűzésen (RTO). Egy több régióban történő üzembe helyezés például drágább, mint az egyrégiós üzembe helyezések, és bonyolultabb a kezelése. Műveleti eljárásokra lesz szükség a feladatátvétel és a feladat-visszavétel kezeléséhez. A nagyobb költségek és összetettség egyes üzleti forgatókönyvek esetében indokolt, mások esetében azonban nem.

**Helyezzen virtuális gépeket egy terheléselosztó mögé**. Ne használjon önálló virtuális gépeket a kritikus fontosságú számítási feladatokhoz. Ehelyett helyezzen több virtuális gépet egy terheléselosztó mögé. Ha valamelyik virtuális gép elérhetetlenné válik, a terheléselosztó a többi megfelelő állapotú virtuális gépre osztja el a forgalmat. A konfiguráció üzembe helyezésével kapcsolatos tudnivalókért tekintse meg [a skálázhatóságot és a rendelkezésre állást több virtuális géppel megvalósító rendszer ismertetését][multi-vm-blueprint].

![Elosztott terhelésű virtuális gépek ábrája](./images/load-balancing.svg)

**Replikáljon adatbázisokat**. Az Azure SQL Database és a Cosmos DB automatikusan replikálja az adatokat a régión belül, és engedélyezheti a georeplikációt a régiók között. Ha IaaS adatbázis-megoldást használ, válasszon olyat, amely támogatja a replikációt és a feladatátvételt, például az [SQL Server Always On rendelkezésre állási csoportokat][sql-always-on].

**Engedélyezze a georeplikációt**. Az [Azure SQL Database][sql-geo-replication] és a [Cosmos DB][cosmosdb-geo-replication] georeplikációja az adatok másodlagos olvasható replikáját hozza létre egy vagy több másodlagos régióban. Leállás esetén az adatbázis feladatátvételt hajt végre a másodlagos régióba írásra.

**Partíciók rendelkezésre álláshoz**. Gyakran használnak adatbázis-particionálást a jobb skálázhatóság érdekében, de ez a módszer a rendelkezésre állást is javíthatja. Ha egy szegmens leáll, a többi szegmens továbbra is elérhető. Az egyik szegmens hibája csak a tranzakciók egy részét zavarja meg.

**Végezzen üzembe helyezést több régióban**. A legnagyobb rendelkezésre állás érdekében egy vagy több régióban helyezze üzembe az alkalmazást. Így azon ritka esetekben, amikor egy probléma egy teljes régióra hatással van, az alkalmazás feladatátvételt végezhet egy másik régióra. Az alábbi ábrán egy több régiót használó alkalmazás látható, amely az Azure Traffic Managerrel kezeli a feladatátvételt.

![Az Azure Traffic Managerrel kezeli a feladatátvételt ábrája](./images/failover.svg)

**Szinkronizálja az előtérbeli és a háttérbeli feladatátvételt**. Az Azure Traffic Managerrel végezhet feladatátvételt az előtérben. Ha az előtérrendszer az egyik régióban elérhetetlenné válik, a Traffic Manager a másodlagos régióba irányítja át az új kéréseket. Az adatbázis-megoldástól függően előfordulhat, hogy koordinálnia kell az adatbázis feladatátvételét.

**Használjon automatikus feladatátvételt, de manuális feladat-visszavételt**. A Traffic Managert használja az automatikus feladatátvételhez, az automatikus feladat-visszavételhez azonban ne. Az automatikus feladat-visszavétellel kockáztatja, hogy az elsődleges régióra vált még az előtt, hogy a régió megfelelő állapotú lenne. Ehelyett a manuális feladat-visszavétel előtt ellenőrizze, hogy az alkalmazás minden alrendszere megfelelő állapotú-e. Ezenkívül az adatbázistól függően lehet, hogy ellenőriznie kell az adatkonzisztenciát a feladat-visszavétel előtt.

**Legyen a rendszerben redundancia a Traffic Managerhez**. A Traffic Manager egy lehetséges meghibásodási pont. Tekintse át a Traffic Manager szolgáltatói szerződését, és döntse el, hogy a Traffic Manager egyedüli használata elegendő-e vállalata magas rendelkezésre állásra vonatkozó követelményeihez. Ha nem, akkor a biztonság érdekében érdemes lehet hozzáadni egy másik forgalomkezelési szolgáltatást a feladat-visszavételhez. Ha az Azure Traffic Manager szolgáltatás meghibásodik, módosítsa a CNAME rekordját a DNS-ben úgy, hogy a többi forgalomkezelő szolgáltatásra mutasson.

<!-- links -->

[multi-vm-blueprint]: ../../reference-architectures/virtual-machines-windows/multi-vm.md

[cassandra]: https://cassandra.apache.org/
[cosmosdb-geo-replication]: /azure/cosmos-db/distribute-data-globally
[sql-always-on]: https://msdn.microsoft.com/library/hh510230.aspx
[sql-geo-replication]: /azure/sql-database/sql-database-geo-replication-overview
