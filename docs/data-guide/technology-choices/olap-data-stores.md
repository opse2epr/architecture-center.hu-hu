---
title: Egy OLAP-adattároló kiválasztása
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: f3041b95696c9408a2c9ab747fe1ec3041db0743
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/31/2018
---
# <a name="choosing-an-olap-data-store-in-azure"></a>Egy OLAP-adattároló kiválasztása az Azure-ban

Online analitikus feldolgozási (OLAP) egy olyan technológia, nagy üzleti adatbázisok rendezi, amely támogatja az összetett elemzést. Ez a témakör összehasonlítja az OLAP-megoldások Azure-ban.

> [!NOTE]
> Egy OLAP-adattároló használatával kapcsolatos további információkért lásd: [Online analitikus feldolgozási](../scenarios/online-analytical-processing.md).

## <a name="what-are-your-options-when-choosing-an-olap-data-store"></a>Az OLAP-adatok kiválasztásakor Mik a beállítások tárolásához?

Az Azure az OLAP-core követelményeknek megfelelő összes a következő adatokat tárolja:

- [SQL Server oszloptárindexekkel](/sql/relational-databases/indexes/get-started-with-columnstore-for-real-time-operational-analytics)
- [Az Azure Analysis Services](/azure/analysis-services/analysis-services-overview)
- [SQL Server Analysis Services (SSAS)](/sql/analysis-services/analysis-services)

SQL Server Analysis Services (SSAS) OLAP és üzleti intelligenciát biztosító alkalmazások adatok adatbányászati funkciókat kínál. SSAS telepítése a helyi kiszolgálón, vagy az Azure virtuális gépen futtatni. Az Azure Analysis Services egy teljes körűen felügyelt szolgáltatás, amely az SSAS jelentős funkciót biztosít. Az Azure Analysis Services támogatja a csatlakozást [a különféle adatforrások](/azure/analysis-services/analysis-services-datasource) a felhőben és a szervezet helyszíni.

Fürtözött Oszlopcentrikus indexek az SQL Server 2014 és újabb verziók esetén az Azure SQL Database, valamint és OLAP munkaterhelések alkalmasak. Azonban kezdve az SQL Server 2016 (beleértve az Azure SQL Database), kihasználhatja a hibrid tranzakciós/analytics segítségével frissíthető fürtözetlen oszlopcentrikus indexek (HTAP) feldolgozása. HTAP lehetővé teszi az OLTP és az azonos platformon, amely tárolja az adatok többszörös lemásolását, így nem kell, és szükségtelenné teszi a különböző OLTP és az OLAP-rendszerek feldolgozása OLAP. További információkért lásd: [valós idejű működési elemzési Ismerkedés az Oszlopcentrikus](/sql/relational-databases/indexes/get-started-with-columnstore-for-real-time-operational-analytics).

## <a name="key-selection-criteria"></a>Kulcs kiválasztási feltételek

A választási lehetőségek szűkítéséhez indítása ezen kérdések megválaszolásával:

- A saját kiszolgálóit kezelése helyett egy felügyelt szolgáltatás van szüksége?

- Azure Active Directory (Azure AD) segítségével biztonságos hitelesítés szükséges?

- Szeretné valós idejű elemzések elvégzéséhez? Ha igen, szűkítenie a lehetőségeit a valós idejű elemzési támogató. 

    *Valós idejű elemzési* ebben a környezetben egyedüli adatforrásként, például egy vállalati erőforrás-tervező (ERP) alkalmazást, az operatív és az elemzés munkaterhelés futtatandó vonatkozik. Ha integrálja a különböző forrásokból származó adatok, vagy szélsőséges analytics teljesítmény szükséges előre összesített adatokat, például a kockák használatával kell, külön adatraktárban még szüksége lehet.

- Meg szeretné használni, előre összesített adatokat, például arra, hogy szemantikai modellek, amelyek analytics több üzleti felhasználó rövid? Ha igen, válassza ki, amely támogatja a többdimenziós kockák vagy táblázatos szemantikai modellek. 

    Aggregátumok biztosító segítséget következetesen kiszámításához az adatok összesítések. Előre összesített adatokat is biztosít a nagy teljesítményt program több oszlopot a különböző sorok meghatározásakor. Lehet, hogy előre összesített többdimenziós kockák vagy táblázatos szemantikai modellek az adatokat.

- Az OLTP adattároló túl, több forrásból származó adatok integrálni kell? Ha igen, vegye figyelembe a beállításokat, amelyek könnyen integrálható a több adatforrást.

## <a name="capability-matrix"></a>Képesség mátrix

A következő táblázat összefoglalja a főbb változásai képességeit.

### <a name="general-capabilities"></a>Általános képességek

| | Azure Analysis Services | SQL Server Analysis Services | SQL Server oszloptárindexekkel | Az Azure SQL adatbázis oszloptárindexekkel |
| --- | --- | --- | --- | --- |
| Van a felügyelt | Igen | Nem | Nem | Igen |
| Többdimenziós kockák támogatja | Nem | Igen | Nem | Nem |
| A táblázatos szemantikai modellekhez támogatja | Igen | Igen | Nem | Nem |
| Könnyen integrálható a több adatforráshoz | Igen | Igen | No <sup>1</sup> | No <sup>1</sup> |
| Valós idejű elemzési támogatja | Nem | Nem | Igen | Igen |
| Adatok másolása adatbázisból folyamat igényel | Igen | Igen | Nem | Nem |
| Azure AD-integráció | Igen | Nem | Nem <sup>2</sup> | Igen |

[1] bár az SQL Server és az Azure SQL Database lekérdezésének, és integrálhatja a több külső adatforrás nem használható, továbbra is hozhat létre egy folyamatot, amely elvégzi ezt a segítségével [SSIS](/sql/integration-services/sql-server-integration-services) vagy [Azure Data Factory](/azure/data-factory/). Egy Azure virtuális Gépen található SQL Server rendelkezik-e további beállításokat, például a csatolt kiszolgálók és [PolyBase](/sql/relational-databases/polybase/polybase-guide). További információkért lásd: [csővezeték-vezénylési folyamatábrán és adatmozgás](../technology-choices/pipeline-orchestration-data-movement.md).

Egy Azure virtuális gépen futó SQL Server [2] csatlakozás nem támogatott az Azure AD-fiókkal. A tartomány Active Directory-fiókot használni.

### <a name="scalability-capabilities"></a>Méretezhetőség képességek

| | Azure Analysis Services | SQL Server Analysis Services | SQL Server oszloptárindexekkel | Az Azure SQL adatbázis oszloptárindexekkel |
| --- | --- | --- | --- | --- |
| A magas rendelkezésre állás érdekében redundáns területi kiszolgálók  | Igen | Nem | Igen | Igen |
| Támogatja a lekérdezés kibővítési  | Igen | Nem | Igen | Nem |
| Dinamikus méretezhetőség (felskálázott)  | Igen | Nem | Igen | Nem |

