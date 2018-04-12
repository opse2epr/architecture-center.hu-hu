---
title: Online tranzakciófeldolgozás (OLTP)
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 07e7f680c8ee5e8589ff7cd2236ff95f6ee84f4c
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/31/2018
---
# <a name="online-transaction-processing-oltp"></a>Online tranzakciófeldolgozás (OLTP)

A felügyeleti [tranzakciós adatok](../concepts/transactional-data.md) használ a számítógéprendszerek lehet hivatkozni, Online tranzakciófeldolgozási (OLTP). Az OLTP rendszerek regisztrálhatna üzleti fordulnak elő a szervezet napi működését, és támogatja az adatokba, így következtetéseket lekérdezését.

![OLTP az Azure-ban](./images/oltp-data-pipeline.png)

## <a name="when-to-use-this-solution"></a>Ez a megoldás használatával

Ha hatékony feldolgozni, és üzleti tranzakciók tárolja, és azonnal elérhetővé tétele az ügyfélalkalmazások konzisztens módon van szüksége, válassza a OLTP. Használja az ebbe az architektúrába, ha bármilyen anyagi feldolgozási késedelem hatással lenne egy negatív az üzleti mindennapi műveleteinek.

OLTP rendszerek hatékonyan feldolgozni, és a tranzakciók, valamint a lekérdezés tranzakciós adatok tárolására készültek. A cél hatékonyan feldolgozási és tárolása az egyes tranzakciók által az OLTP rendszerek részben úgy érhető el, adatok normalizálási &mdash; Ez azt jelenti, hogy ossza az adatok kisebb csoportjai, amelyek kevesebb redundáns. Támogatja a hatékonyságát, mivel lehetővé teszi, hogy az OLTP rendszerek nagyszámú tranzakciók egymástól függetlenül feldolgozni, és elkerülhető a felesleges feldolgozási szükséges redundáns adatok mellett adatok integritásának fenntartása.

## <a name="challenges"></a>Problémák
Végrehajtási és az OLTP rendszerek használata néhány kihívást hozhat létre:

- Az OLTP rendszerek megfelelőek nem mindig összesítések kezelése keresztül nagy mennyiségű adat, bár kivételek, például egy tervezett jól az SQL Server-alapú megoldás. Az adatok, támaszkodó összesített számítások az egyes tranzakciók több mint több millió, alapján Analytics nagyon erőforrás-igényes az OLTP rendszerek. Lassú hajtható végre, és lehet el egy slow-down következtében letiltásával más tranzakciók az adatbázisban.
- Ha elemzés végrehajtása, és az adatok magas normalizált jelentéskészítés, a lekérdezések általában összetett, mert irányuló legtöbb lekérdezésnek kell deszerializálni optimalizálására illesztések használatával az adatok. Emellett az OLTP rendszerek adatbázis-objektumok elnevezési szabályai általában tömör és állapotára. A nagyobb normalizálási alapján tömör elnevezési konvenciók kialakulhat nehezebben OLTP rendszerek üzleti felhasználók DBA vagy az adatok a fejlesztők az nélkül lekérdezéséhez.
- Lassú a lekérdezési teljesítményt, attól függően, hogy a tárolt tranzakciók száma tranzakciók előzményeinek határozatlan ideig tárolja, és túl sok adat tárolása táblában vezethet. A gyakori megoldás, hogy az OLTP rendszerben (például az aktuális pénzügyi év) megfelelő ablakban karbantartása és egyéb rendszerekre, például egy adatpiacot korábbi adatok továbbítása vagy [adatraktár](../technology-choices/data-warehouses.md).

## <a name="oltp-in-azure"></a>OLTP az Azure-ban

Alkalmazások, mint az üzemeltetett webhelyek [App Service Web Apps](/azure/app-service/app-service-web-overview), az App Service-ben futó REST API-k vagy hordozható vagy asztali alkalmazások az OLTP rendszerek általában a REST API-t a közvetítő keresztül kommunikálnak.

A gyakorlatban a munkaterhelések többségéhez nincsenek tisztán OLTP. Nincs általában egy [analitikai összetevő](../scenarios/online-analytical-processing.md) is. Emellett nincs valós idejű jelentéskészítést, például a jelentések futtatása ellen a műveleti rendszerfiókok iránti kereslet. Ez más néven a HTAP (hibrid tranzakciós és Analytical Processing). További információkért lásd: [Online Analytical Processing (OLAP) adattárolókhoz](../technology-choices/olap-data-stores.md).

## <a name="technology-choices"></a>Technológiai lehetőségek

Adattárolás:

- [Azure SQL Database](/azure/sql-database/)
- [SQL Server egy Azure virtuális gépen](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-server-iaas-overview?toc=%2Fazure%2Fvirtual-machines%2Fwindows%2Ftoc.json)
- [Azure Database for MySQL](/azure/mysql/)
- [Azure Database for PostgreSQL](/azure/postgresql/)

További információkért lásd: [egy OLTP adattároló kiválasztása](../technology-choices/oltp-data-stores.md)

Adatforrások:

- [Az App service](/azure/app-service/)
- [Mobile Apps](/azure/app-service-mobile/)

