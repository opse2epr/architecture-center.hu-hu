---
title: Online analitikus feldolgozás (OLAP)
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.openlocfilehash: beed0d642e85096efc0b6fe492181b8dcd771d2d
ms.sourcegitcommit: e7e0e0282fa93f0063da3b57128ade395a9c1ef9
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/05/2018
ms.locfileid: "52902598"
---
# <a name="online-analytical-processing-olap"></a>Online analitikus feldolgozás (OLAP)

Online analitikus feldolgozási (OLAP) olyan technológia, amely rendszerezi a nagyméretű adatbázisok, és támogatja az összetett elemzésére. Anélkül, hogy negatívan befolyásolná a tranzakciós rendszerek összetett elemzési lekérdezések végrehajtásához használható.

Az adatbázisok, amely egy üzleti összes tranzakció tárolására használja, és a rekordok az úgynevezett [online tranzakciófeldolgozás (OLTP)](online-transaction-processing.md) adatbázisok. Ezek az adatbázisok általában a rekordokat, amelyek a megadott egyszerre csak egy rendelkeznek. Gyakran nagy mennyiségű, amely a szervezet értékes információkat tartalmaznak. Az adatbázisok által használt OLTP, azonban nem tervezték elemzés céljából. Válaszok adatbázisaihoz való beolvasásakor ezért túl sok időt venne igénybe. OLAP-rendszerek arra tervezték, hogy az üzleti intelligenciával kapcsolatos információk kinyerése egy rendkívül nagy teljesítményt nyújtva az adatok segítségével módszert. Ennek az oka az OLAP-adatbázisok (nagy erőforrásigényű) olvasása, alacsony írási számítási feladatok vannak optimalizálva.

![Az Azure-ban OLAP](../images/olap-data-pipeline.png) 

## <a name="semantic-modeling"></a>Szemantikai modellezés

A szemantikai adatmodell a fogalmi modell, amely tartalmaz adatelemet jelentését ismerteti. Szervezetek gyakran rendelkeznek saját feltételei biztosítják a dolog, néha a szinonimák, vagy akár különböző jelentéssel azonos kifejezést. Például egy készlet adatbázis előfordulhat, hogy nyomon követheti a berendezés, eszköz-Azonosítóval és a egy sorszám, de utalhat a sorozatszám, eszköz-azonosító néven egy értékesítési adatbázis Nincs egyszerű mód arra, hogy ezeket az értékeket egy olyan modell, kapcsolat nélkül. 

Szemantikai modellezés biztosít absztrakciós az adatbázissémát át, hogy a felhasználóknak nem kell tudni, hogy az alapul szolgáló adatok struktúrák. Ez megkönnyíti a végfelhasználók számára használatával adatokat lekérdezni keresztül az alapul szolgáló séma aggregátumokat és illesztések végrehajtása nélkül. Ezenkívül általában oszlopok váltják fel nagyobb mértékben felhasználóbarát nevek, hogy a környezet és az adatok jelentését kézenfekvő.

Szemantikai modellezés döntő többsége olvasási forgatókönyvek, például az elemzési és az üzleti intelligencia (OLAP), a használt további írási műveltekből tranzakciós adatok feldolgozási (OLTP) helyett. Ez az, főleg egy tipikus szemantikai réteg jellege miatt:

- Összesítés viselkedések vannak beállítva, hogy a jelentéskészítő eszközökkel megfelelően megjeleníteni azokat.
- Üzleti logika és számítások határozza meg.
- Idő-orientált számításokat is.
- Adatok gyakran integrált, több forrásból. 

Hagyományosan a szemantikai réteg felett van elhelyezve egy adattárházat ebből kifolyólag.

![A szemantikai réteg között egy adattárházat, és jelentéskészítési eszköz, például ábrája](../images/semantic-modeling.png)

Szemantikai modellek elsődleges két típusa van:

* **Táblázatos**. Relációs modellezési szerkezeteket (modell, táblák és oszlopok) használja. Belsőleg metaadatok (kockákat, dimenziókat, mértékeket) szerkezeteket modellezési OLAP öröklődik. Kód és a parancsfájl használata az OLAP-metaadatok.
* **Többdimenziós**. Hagyományos OLAP modellezési (kockákat, dimenziókat, mértékeket) szerkezeteket használ.

Kapcsolódó Azure-szolgáltatás:
- [Az Azure Analysis Services](https://azure.microsoft.com/services/analysis-services/)

## <a name="example-use-case"></a>Példa használati esetekhez

Egy szervezet nagy-adatbázisban tárolt adatokat tartalmaz. Az adatok elérhetővé tétele az üzleti felhasználók és az ügyfelek a saját jelentéseket hozhatnak létre, vagy elemzést végezni szeretné. Egy lehetőség, hogy csak az adatbázis ezen felhasználók közvetlen hozzáférést biztosít. Vannak azonban több hátrányai ezzel, beleértve a biztonság kezelése és hozzáférés szabályozása. Az adatbázis, beleértve a táblák és oszlopok nevei a kialakítás is, egy felhasználó megértése nehéz lehet. Felhasználók kell tudni, hogy mely táblázatokat kívánja lekérdezni, hogyan kell csatlakoztatni a táblázatok és más üzleti logika, amely a alkalmazni kell a megfelelő eredményt. Felhasználók is egy lekérdezési nyelvet, például az SQL, még akkor is, a kezdéshez ismernie kell. Általában ez vezet, metrikák jelentéskészítés több felhasználó számára, de különböző eredményekkel.

Egy másik lehetőség, hogy magába foglalja az összes olyan információ, amelyet a felhasználóknak kell Szemantikus modellbe. A szemantikai modell egyszerűbben lekérdezhetők, a felhasználók egy jelentéskészítő eszközzel választott eszközükön. A szemantikai modell által megadott adatok biztosítása, hogy az összes felhasználó számára látható egy egységes valós elemmé egyetlen verziója, a data warehouse-ból kéri le. A szemantikai modell emellett egyszerű táblaneveket és oszlopneveket, táblák, leírások, számítások és sorszintű biztonság közötti kapcsolatokat.

## <a name="typical-traits-of-semantic-modeling"></a>Tipikus képességekre vonatkozó szemantikai modellezés

Szemantikai modellezés és analitikus feldolgozás általában a következő jellemzőkkel rendelkezik:

| Követelmény | Leírás |
| --- | --- |
| Séma | Séma, erősen kényszerítése|
| Használja a tranzakciók | Nem |
| Zárolási stratégia | None |
| Frissíthető | Nem (legtöbbször adatkocka újraszámítása) |
| Appendable | Nem (legtöbbször adatkocka újraszámítása) |
| Számítási feladat | Nehéz olvasási, csak olvasható |
| Indexelés | Többdimenziós indexelése |
| Datum mérete | Kis és közepes méretű |
| Modell | Többdimenziós |
| Adatalakzat:| Adatkocka vagy a csillag/snowflake-séma |
| Rugalmas lekérdezés | Rendkívül rugalmas |
| Méretezési csoport: | Nagy méretű (10 egység-100s GB) |

## <a name="when-to-use-this-solution"></a>Ez a megoldás használata

Vegye figyelembe a OLAP a következő esetekben:

- Anélkül, hogy negatívan befolyásolná az OLTP rendszerek gyors, hajtsa végre a komplex analitikai és ad hoc lekérdezéseket kell. 
- Lehetővé szeretné tenni az üzleti felhasználók együtt egy egyszerű módja az adatokból jelentések készítése
- Adjon meg egy számot, amely lehetővé teszi a felhasználók számára a gyors és következetes eredményeinek beolvasása összesítések szeretné. 

Az OLAP különösen hasznos alkalmazásának összesített számításokat nagy mennyiségű adat keresztül. Az OLAP-rendszerek olvasási forgatókönyvek, például az elemzési és az üzleti intelligencia vannak optimalizálva. Az OLAP lehetővé teszi a felhasználóknak szegmens többdimenziós adatok (például egy kimutatás) kétféleképpen lehet megtekinteni vagy az adatok Szűrés adott értékekre szeletekre. Ez a folyamat néha "Tovább szeletelve és darabolva" az adatokat, és megteheti függetlenül attól, hogy az adatok particionálása több adatforrás között. Ez segít a felhasználóknak keresse meg a mintákat, trendeket, és Fedezze fel az adatokat a részletek a hagyományos analitikai ismerete nélkül.

Szemantikai modellek segítségével az üzleti felhasználók absztrakt kapcsolat hagyhatják, és megkönnyíti az adatok gyors elemzését.

## <a name="challenges"></a>Problémák

Minden OLAP-rendszerek előnyöket biztosítanak azok előállításához néhány kihívásokat:

- OLTP-rendszerekben adatokat tranzakciók a különböző forrásokból származó áramlanak keresztül folyamatosan frissül, mivel az OLAP-adattárak általában frissülnek sokkal lassabbá időközönként, attól függően, üzleti igényeknek megfelelően. Ez azt jelenti, hogy az OLAP-rendszerek jobban megfelel módosítások azonnali válaszokat helyett stratégiai üzleti döntéseket hozhasson. Bizonyos szintű Adattisztítás és vezénylési is, és naprakész állapotban tarthatja az OLAP-adattárakban tervezni kell.
- Ellentétben a hagyományos, normalizált, relációs táblák OLTP-rendszerekben található OLAP-adatmodellek általában többdimenziós. Így nehéz vagy lehetetlen közvetlen leképezése az entitás-kapcsolatot vagy objektumorientált modellek, ahol minden attribútum egy oszlop van leképezve. Az OLAP-rendszerek általában inkább egy csillag vagy snowflake sémát hagyományos normalizálási helyett.

## <a name="olap-in-azure"></a>Az Azure-ban OLAP

Az Azure-ban tárolt adatok OLTP-rendszerekben, mint például az Azure SQL Database átmásolja az az OLAP-rendszer, például [Azure Analysis Services](/azure/analysis-services/analysis-services-overview). Adatáttekintési és vizualizációs eszközök, például [Power BI](https://powerbi.microsoft.com), az Excel és a külső beállítások Analysis Services-kiszolgálóhoz csatlakozzon, és biztosíthatja a felhasználók számára a modellezett adatok interaktív és vizuálisan gazdag betekintést. A folyamat az adatok OLTP adatokból OLAP általában vezényelt használatával hajthatók végre az SQL Server Integration Services használatával [Azure Data Factory](/azure/data-factory/concepts-integration-runtime).

Az Azure-ban a következő adattárakat összes OLAP alapvető követelményei felel meg:

- [Az Oszlopcentrikus indexek az SQL Server](/sql/relational-databases/indexes/get-started-with-columnstore-for-real-time-operational-analytics)
- [Az Azure Analysis Services](/azure/analysis-services/analysis-services-overview)
- [SQL Server Analysis Services (SSAS)](/sql/analysis-services/analysis-services)

Az SQL Server Analysis Services (SSAS) OLAP és adatok adatbányászati funkció az üzleti intelligenciára épülő alkalmazások érhetők el. Az SSAS helyi kiszolgálón telepíti, vagy az Azure-beli virtuális gépen futtatni. Az Azure Analysis Services egy teljes körűen felügyelt szolgáltatás, amely az SSAS azonos főbb szolgáltatásokat. Az Azure Analysis Services támogatja a csatlakozást [különböző adatforrásokból](/azure/analysis-services/analysis-services-datasource) a felhőben és a szervezet helyszíni.

Fürtözött Oszlopcentrikus indexek érhetők el az SQL Server 2014 és újabb, valamint az Azure SQL Database- és OLAP számítási feladatokhoz ideális. Azonban az SQL Server 2016 (beleértve az Azure SQL Database) kezdve kihasználhatja a hibrid tranzakciós és analitikai (HTAP) feldolgozása révén frissíthető fürtözetlen oszlopcentrikus index esetében. HTAP lehetővé teszi, hogy OLTP és OLAP típusú feldolgozási az azonos platformon, amely szükségtelenné teszi az adatok több példányát tárolja, és szükségtelenné teszi a különböző OLTP és OLAP típusú rendszerek. További információkért lásd: [Oszlopcentrikus Ismerkedés a valós idejű operatív elemzést](/sql/relational-databases/indexes/get-started-with-columnstore-for-real-time-operational-analytics).

## <a name="key-selection-criteria"></a>Fontosabb kritériumok

Így szűkítheti, első lépésként a kérdések megválaszolása:

- Szeretne egy felügyelt szolgáltatás, hanem a saját kiszolgálók kezelése?

- Szükség van az Azure Active Directory (Azure AD) biztonságos hitelesítési?

- Biztosan végezhető valós idejű elemzés? Ha igen, szűkíthetők, amelyek támogatják a valós idejű elemzési lehetőségeit. 

    *Valós idejű elemzési* ebben a környezetben egy adatforráshoz, például a vállalati erőforrástervező (ERP) alkalmazással, az operatív és a egy elemzési számítási feladatok által futtatott vonatkozik. Ha több forrásból származó adatokat integrálhat, vagy szélsőséges analitikai teljesítménynövekedést előre összesített adatok, például kockákat használatával van szüksége, továbbra is szükség lehet egy külön data warehouse-bA.

- Szükséges előre összesített adatokat, a szemantikai modelleket, amelyek analytics további üzleti felhasználó rövid biztosítanak például használni? Ha igen, válasszon egy beállítást, amely támogatja a többdimenziós kockák vagy táblázatos szemantikai modellek. 

    Összesítések nyújtó segítségével a felhasználók konzisztens módon kiszámítása az adatok összesítések. Előre összesített adatokat is lehetővé teszi nagy teljesítménynek esetén több oszlopot közötti sorok számát. Adatok előzetes összesítésére, többdimenziós kockák vagy táblázatos szemantikai modellek is.

- Szükség van integrálhatók több forrásból is, meghaladja az OLTP-adattárhoz adatok? Ha igen, fontolja meg a beállításokat, amelyek könnyen integrálhatók több adatforrást.

## <a name="capability-matrix"></a>Képességmátrix

A következő táblázat összefoglalja a fő különbségeket, a képességek.

### <a name="general-capabilities"></a>Általános képességek

| | Azure Analysis Services | SQL Server Analysis Services | Az Oszlopcentrikus indexek az SQL Server | Az Oszlopcentrikus indexek az Azure SQL Database |
| --- | --- | --- | --- | --- |
| A felügyelt szolgáltatás | Igen | Nem | Nem | Igen |
| Többdimenziós kockák támogatja | Nem | Igen | Nem | Nem |
| Támogatja a táblázatos szemantikai modelleket | Igen | Igen | Nem | Nem |
| Egyszerű integrálás több adatforráshoz | Igen | Igen | No <sup>1</sup> | No <sup>1</sup> |
| Támogatja a valós idejű elemzés | Nem | Nem | Igen | Igen |
| Folyamat adatokat másol (ok) hoz van szükség | Igen | Igen | Nem | Nem |
| Az Azure AD-integráció | Igen | Nem | Nem <sup>2</sup> | Igen |

[1] bár az SQL Server és az Azure SQL Database lekérdezésének, és integrálhatja a több külső adatforrás nem használható, továbbra is hozhat létre egy folyamatot, amely azért teszi ezt használatával [SSIS](/sql/integration-services/sql-server-integration-services) vagy [Azure Data Factory](/azure/data-factory/). Egy Azure-beli virtuális gépen futó SQL Server rendszer további beállítások, például a csatolt kiszolgálók és [PolyBase](/sql/relational-databases/polybase/polybase-guide). További információkért lásd: [folyamat vezénylési, átvitelvezérlés és adatáthelyezés](../technology-choices/pipeline-orchestration-data-movement.md).

A(z) [2] csatlakozás egy Azure virtuális gépen futó SQL Server egy Azure AD-fiók használata nem támogatott. Használja helyette a tartomány Active Directory-fiókot.

### <a name="scalability-capabilities"></a>Skálázhatósági képességeket.

|                                                  | Azure Analysis Services | SQL Server Analysis Services | Az Oszlopcentrikus indexek az SQL Server | Az Oszlopcentrikus indexek az Azure SQL Database |
|--------------------------------------------------|-------------------------|------------------------------|-------------------------------------|---------------------------------------------|
| Redundáns regionális kiszolgálók magas rendelkezésre állás érdekében |           Igen           |              Nem              |                 Igen                 |                     Igen                     |
|             Támogatja a lekérdezés horizontális felskálázás             |           Igen           |              Nem              |                 Igen                 |                     Nem                      |
|          A dinamikus méretezhetőség (vertikális felskálázási)          |           Igen           |              Nem              |                 Igen                 |                     Nem                      |

