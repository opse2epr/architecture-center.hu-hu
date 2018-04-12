---
title: Online analitikus feldolgozás (OLAP)
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 92b71934f2081e95c3c9b0d4dc9edeb3885b12e8
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/06/2018
---
# <a name="online-analytical-processing-olap"></a>Online analitikus feldolgozás (OLAP)

Online analitikus feldolgozási (OLAP) egy olyan technológia, nagy üzleti adatbázisok rendezi, amely támogatja az összetett elemzést. Anélkül, hogy negatívan befolyásolná a tranzakciós rendszerek bonyolult elemzési lekérdezések végrehajtásához használható.

Az adatbázisok, amely egy üzleti használ a tranzakciók és nevezzük [online tranzakció-feldolgozási (OLTP)](online-transaction-processing.md) adatbázisok. Ezeket az adatbázisokat általában azt jelzi, hogy a rendszer a megadott egyszerre csak egy rendelkeznek. Gyakran tartalmaznak, amely a szervezet számára a legértékesebb sok. Az OLTP, a használt adatbázisok azonban nem képesek elemzés céljából. Ezért válaszok lekérése ezeket az adatbázisokat, idő és erőfeszítés tekintetében költségesnek számít kinyerni. OLAP-rendszerek arra tervezték, hogy az adatok magas performant az üzleti intelligenciával kapcsolatos információk kinyerése érdekében módon. Ennek oka az, olvasási, írási alacsony munkaterhelések gyakori vannak optimalizálva, OLAP-adatbázisokat.

![Az Azure-ban OLAP](../images/olap-data-pipeline.png) 

## <a name="semantic-modeling"></a>Szemantikai modellezés

A szemantikai adatmodell a fogalmi modell, amely leírja a benne található adatelemek szerinti. Szervezetek sokszor a saját feltételei dolgot, egyes esetekben a szinonimák, vagy akár különböző jelentésekkel azonos kifejezés. Például egy készlet adatbázis előfordulhat, hogy nyomon berendezés, eszköz-Azonosítóval és a sorszámot, de az értékesítési adatbázis utalhat a sorozatszám eszköz azonosítóként Nincs egyszerű mód más ezeket az értékeket egy modell, amely leírja a kapcsolat nélkül. 

Szemantikai modellezési, hogy a felhasználók nem szükséges tudni, hogy az alapul szolgáló adatstruktúrák biztosítanak az adatbázisséma egy Elvonási szint. Így könnyebben a végfelhasználók számára az adatok lekérdezése nélkül aggregátumok és illesztések az alapul szolgáló séma keresztül hajtja végre. Emellett általában oszlopok váltják fel felhasználóbarát nevek, és az adatok jelentését viszonylag kézenfekvő legyenek.

Szemantikai modellezési olvasási-nehéz helyzetekben, például elemzés és üzleti intelligencia (OLAP) használt döntő többsége figyelésekor további írási műveleteket tranzakciós adatok feldolgozási (OLTP). Egy tipikus szemantikai réteg jellege miatt legtöbbször nem:

- Összesítési viselkedéshez, hogy a jelentéskészítési eszközök jelennek meg azok megfelelően vannak beállítva.
- Üzleti logika és számítások határozza meg.
- Idő célú számítások tartoznak.
- Adatok gyakran integrálva van a különböző forrásokból származó. 

Hagyományosan a szemantikai réteg felett van elhelyezve egy adatraktár ezen okok miatt.

![A szemantikai réteg között adatraktárt és jelentéskészítési eszköz, például ábrája](../images/semantic-modeling.png)

A szemantikai modellek két elsődleges típusa van:

* **A táblázatos**. Relációs modellezési szerkezetek (modell, táblázatok, oszlopok) használja. Belsőleg metaadatok (kockák, dimenziókat, mértékeket) szerkezetek modellezési OLAP öröklődik. Kód és parancsfájl használata az OLAP-metaadatok.
* **Többdimenziós**. Hagyományos OLAP modellezési szerkezetek (kockák, dimenziókat, mértékeket) használja.

Kapcsolódó Azure-szolgáltatás:
- [Az Azure Analysis Services](https://azure.microsoft.com/services/analysis-services/)

## <a name="example-use-case"></a>Példa használati eset

Egy szervezet nagy adatbázisban tárolt adatok rendelkezik. Azt szeretné elérhetővé tenni az adatok üzleti felhasználók és a felhasználók a saját jelentések létrehozásához, és néhány elemzést. Egy elem csak a felhasználók közvetlen hozzáférésének az adatbázisban. Van azonban több hátrányai ezzel, beleértve a biztonság kezelése és hozzáférés szabályozása. Az adatbázis, a táblák, illetve az oszlopok, beleértve a Tervező is, egy felhasználó megértése nehéz lehet. Felhasználók kellene, hogy mely táblák lekérdezéséhez, hogyan kell csatlakoztatni azokat a táblákat és más üzleti logika, amely a megfelelő eredményt kell alkalmazni. Felhasználók például még a kezdéshez SQL lekérdezésnyelvet tudnia kell. Általában ez vezet, jelentéskészítő elérhető metrikák több felhasználó, de eltérő eredményt.

Egy másik lehetőség egy foglalják magukban a felhasználók a szemantikai modell szükséges információk. A szemantikai modell könnyebben lehet lekérdezni a felhasználók az általuk választott jelentéskészítési eszközzel. A szemantikai modell által szolgáltatott adatokat a rendszer egy adatraktárból, győződjön meg arról, hogy minden felhasználó tekintse meg a valóságnak egyetlen verziója hívja elő. A szemantikai modell emellett rövid tábla, az oszlopnevek, táblák, a leírásokat, a számítások és a sorszintű biztonság közötti kapcsolatokat.

## <a name="typical-traits-of-semantic-modeling"></a>A szemantikai modellezési tipikus jellemzők

A szemantikai modellezési és elemzésfeldolgozási általában a következő jellemzőkkel rendelkezik:

| Követelmény | Leírás |
| --- | --- |
| Séma | Írás, erősen kényszerített séma|
| Használja a tranzakciók | Nem |
| Zárolási stratégia | None |
| Updateable | Nem (általában szükséges adatkocka recomputing) |
| Appendable | Nem (általában szükséges adatkocka recomputing) |
| Számítási feladat | Nagy mennyiségű olvasási, csak olvasható |
| Indexelés | Többdimenziós indexelő |
| Datum mérete | Kis, közepes méretű |
| Modell | Többdimenziós |
| Adatok alakzat:| Adatkocka vagy csillag/snowflake séma |
| Lekérdezés rugalmasságot | Rugalmas |
| Skála: | Nagy (10 egység-100-as egység GB) |

## <a name="when-to-use-this-solution"></a>Ez a megoldás használatával

Vegye figyelembe a OLAP a következő esetekben:

- Összetett analitikai és ad hoc lekérdezéseket gyorsan, anélkül, hogy negatívan befolyásolná az OLTP rendszerek hajt végre kell. 
- Kívánt biztosít üzleti felhasználók egyszerűen az adatok alapján jelentések készítése
- Adjon meg egy számot az összesítések, amelyek lehetővé teszik a felhasználók gyors, konzisztens eredmények el szeretné. 

OLAP különösen fontos összesített számítások alkalmazásának keresztül nagy mennyiségű adat. OLAP-rendszerek olvasás-nehéz helyzetekben, például elemzés és a business intelligence vannak optimalizálva. OLAP lehetővé teszi a felhasználóknak szegmens többdimenziós adatok időszeletekre tekintheti meg (például egy olyan kimutatás) két dimenzió vagy adott értékek szerint szűrje az adatokat. Ezt a folyamatot nevezik néha "szeletelhető és feldarabolható" az adatokat, függetlenül attól, hogy az adatok particionálása több adatforrások keresztül hajtható végre. Ez segít a felhasználók kereshetnek trendek, helyszíni minták, és áttekintheti az adatokat a hagyományos adatelemzés részleteit ismerete nélkül.

Szemantikai modellek segítségével az üzleti felhasználók absztrakt kapcsolat összetett szolgáltatásokkal, és könnyebben adatok gyors elemzését.

## <a name="challenges"></a>Problémák

Az összes, az OLAP-rendszerek előnyöket biztosítanak azok néhány kihívást eredményez:

- Az OLTP rendszerek adatok folyamatosan frissül a különböző forrásokból folyik a tranzakciók, mivel OLAP-adattároló általában sokkal lassabb időközönként, attól függően, hogy az üzleti igények frissítése. Ez azt jelenti, hogy az OLAP-rendszerek alkalmasabbak módosítások azonnali válaszokat, hanem stratégiai üzleti döntéseket hozhat. Bizonyos fokú adatok tisztításokat és az orchestration kell az OLAP-adattároló naprakész állapotban tartása érdekében meg kell tervezni.
- Hagyományos, normalizált, relációs táblákat a OLTP rendszerek, eltérően OLAP adatmodellekben általában többdimenziós. Így nehéz vagy lehetetlen közvetlenül hozzárendelése egyedkapcsolat vagy objektumorientált modellek, ahol minden attribútum egy oszlopra van leképezve. OLAP-rendszerek általában inkább egy csillag vagy snowflake séma hagyományos normalizálási helyett.

## <a name="olap-in-azure"></a>Az Azure-ban OLAP

Az Azure data tárolt OLTP rendszerek például az Azure SQL Database másolódik az OLAP-rendszert, például a [Azure Analysis Services](/azure/analysis-services/analysis-services-overview). Adatok feltárása és a képi megjelenítés eszközök, például [Power BI](https://powerbi.microsoft.com), az Excel és a külső beállítások Analysis Services-kiszolgálókkal, és a modellezett adatok magas interaktív és vizuálisan részletes betekintést biztosít a felhasználók. Az adatáramlás OLTP adatokból való OLAP általában vezénylését használatával hajtható végre SQL Server Integration Services használatával [Azure Data Factory](/azure/data-factory/concepts-integration-runtime).

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

|                                                  | Azure Analysis Services | SQL Server Analysis Services | SQL Server oszloptárindexekkel | Az Azure SQL adatbázis oszloptárindexekkel |
|--------------------------------------------------|-------------------------|------------------------------|-------------------------------------|---------------------------------------------|
| A magas rendelkezésre állás érdekében redundáns területi kiszolgálók |           Igen           |              Nem              |                 Igen                 |                     Igen                     |
|             Támogatja a lekérdezés kibővítési             |           Igen           |              Nem              |                 Igen                 |                     Nem                      |
|          Dinamikus méretezhetőség (felskálázott)          |           Igen           |              Nem              |                 Igen                 |                     Nem                      |

