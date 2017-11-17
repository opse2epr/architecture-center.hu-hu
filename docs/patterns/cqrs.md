---
title: CQRS
description: "Elkülönítse műveletekkel kapcsolatos adatokat olvasni az operatív adatokat frissítő külön-felületek használatával."
keywords: "Kialakítási mintája"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- data-management
- design-implementation
- performance-scalability
ms.openlocfilehash: f36f759b16566a6c46bf78b8c8b8df4fcd2c493d
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="command-and-query-responsibility-segregation-cqrs-pattern"></a><span data-ttu-id="cc721-104">Szabály parancs és a lekérdezés felelősségi elkülönítése (CQRS)</span><span class="sxs-lookup"><span data-stu-id="cc721-104">Command and Query Responsibility Segregation (CQRS) pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="cc721-105">Elkülönítse műveletekkel kapcsolatos adatokat olvasni az operatív adatokat frissítő külön-felületek használatával.</span><span class="sxs-lookup"><span data-stu-id="cc721-105">Segregate operations that read data from operations that update data by using separate interfaces.</span></span> <span data-ttu-id="cc721-106">Maximalizálhatja a teljesítményt, a méretezhetőség és a biztonsági.</span><span class="sxs-lookup"><span data-stu-id="cc721-106">This can maximize performance, scalability, and security.</span></span> <span data-ttu-id="cc721-107">Támogatja a rendszer fejlődéséhez magasabb rugalmassága időbeli, előfordulhat, hogy a frissítési parancsok, amely a tartományi szinten egyesítési ütközések.</span><span class="sxs-lookup"><span data-stu-id="cc721-107">Supports the evolution of the system over time through higher flexibility, and prevent update commands from causing merge conflicts at the domain level.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="cc721-108">A környezetben, és probléma</span><span class="sxs-lookup"><span data-stu-id="cc721-108">Context and problem</span></span>

<span data-ttu-id="cc721-109">Hagyományos felügyeleti rendszerekkel parancsokat (az adatok frissítések) és a lekérdezések (a kérelmek adatok) ugyanazokat a egyetlen tárházban entitások elleni végre.</span><span class="sxs-lookup"><span data-stu-id="cc721-109">In traditional data management systems, both commands (updates to the data) and queries (requests for data) are executed against the same set of entities in a single data repository.</span></span> <span data-ttu-id="cc721-110">Ezeket az entitásokat is lehet a sorok egy relációs adatbázisban, például az SQL Server egy vagy több táblájában.</span><span class="sxs-lookup"><span data-stu-id="cc721-110">These entities can be a subset of the rows in one or more tables in a relational database such as SQL Server.</span></span>

<span data-ttu-id="cc721-111">Általában az ezekben a rendszerekben összes létrehozása, Olvasás, frissítés és Törlés (CRUD) operations entitás azonos jellemzőkkel is vonatkozik.</span><span class="sxs-lookup"><span data-stu-id="cc721-111">Typically in these systems, all create, read, update, and delete (CRUD) operations are applied to the same representation of the entity.</span></span> <span data-ttu-id="cc721-112">Például egy átviteli-objektumot (DTO) jelölő, az ügyfél az adatelérési réteg (DAL) által az adatok tárolójából beolvasni, és a képernyőn látható.</span><span class="sxs-lookup"><span data-stu-id="cc721-112">For example, a data transfer object (DTO) representing a customer is retrieved from the data store by the data access layer (DAL) and displayed on the screen.</span></span> <span data-ttu-id="cc721-113">A felhasználó frissíti a DTO egyes mezőit (lehet, hogy a adatkötés), és a DTO majd kerül vissza az adattár a DAL által.</span><span class="sxs-lookup"><span data-stu-id="cc721-113">A user updates some fields of the DTO (perhaps through data binding) and the DTO is then saved back in the data store by the DAL.</span></span> <span data-ttu-id="cc721-114">Az azonos DTO szolgál az olvasási és írási műveletet.</span><span class="sxs-lookup"><span data-stu-id="cc721-114">The same DTO is used for both the read and write operations.</span></span> <span data-ttu-id="cc721-115">Az ábra azt mutatja be, egy hagyományos CRUD-architektúrát.</span><span class="sxs-lookup"><span data-stu-id="cc721-115">The figure illustrates a traditional CRUD architecture.</span></span>

![A hagyományos CRUD-architektúra](./_images/command-and-query-responsibility-segregation-cqrs-tradition-crud.png)

<span data-ttu-id="cc721-117">Hagyományos CRUD munkahelyi tervez, megfelelő, ha csak korlátozott üzleti logika vonatkozik, az adatok műveletek.</span><span class="sxs-lookup"><span data-stu-id="cc721-117">Traditional CRUD designs work well when only limited business logic is applied to the data operations.</span></span> <span data-ttu-id="cc721-118">Scaffold mechanizmusok fejlesztői eszközök hozhatnak létre a adatok hozzáférési kód nagyon gyorsan, amely majd testreszabható, amennyiben szükséges.</span><span class="sxs-lookup"><span data-stu-id="cc721-118">Scaffold mechanisms provided by development tools can create data access code very quickly, which can then be customized as required.</span></span>

<span data-ttu-id="cc721-119">A hagyományos CRUD megközelítés azonban néhány hátránya rendelkezik:</span><span class="sxs-lookup"><span data-stu-id="cc721-119">However, the traditional CRUD approach has some disadvantages:</span></span>

- <span data-ttu-id="cc721-120">Ez gyakran azt jelenti, hogy nem egyeznek az adatokhoz, például további oszlopok vagy frissítenie kell a megfelelő annak ellenére, hogy ezek nem egy művelet kötelező tulajdonságok olvasási és írási ábrázolásai között.</span><span class="sxs-lookup"><span data-stu-id="cc721-120">It often means that there's a mismatch between the read and write representations of the data, such as additional columns or properties that must be updated correctly even though they aren't required as part of an operation.</span></span>

- <span data-ttu-id="cc721-121">Az adatok versengés kockázatok, rekordok az együttműködést tartományban, ahol több gyakrabban fog működni, ugyanazokat az adatokat, párhuzamos adattárban zárolásakor.</span><span class="sxs-lookup"><span data-stu-id="cc721-121">It risks data contention when records are locked in the data store in a collaborative domain, where multiple actors operate in parallel on the same set of data.</span></span> <span data-ttu-id="cc721-122">Vagy frissítés, ha optimista zárolás egyidejű frissítések okozza.</span><span class="sxs-lookup"><span data-stu-id="cc721-122">Or update conflicts caused by concurrent updates when optimistic locking is used.</span></span> <span data-ttu-id="cc721-123">A kockázatok növelje a összetettségét és -növelés sebességét, a rendszer.</span><span class="sxs-lookup"><span data-stu-id="cc721-123">These risks increase as the complexity and throughput of the system grows.</span></span> <span data-ttu-id="cc721-124">Emellett a hagyományos megközelítés teljesítményére negatív hatással lehet az adattárban és az adatelérési réteg betöltése, és a lekérdezések összetettsége adatbeolvasás szükséges.</span><span class="sxs-lookup"><span data-stu-id="cc721-124">In addition, the traditional approach can have a negative effect on performance due to load on the data store and data access layer, and the complexity of queries required to retrieve information.</span></span>

- <span data-ttu-id="cc721-125">Segítségükkel kezelése biztonsági és engedélyek összetettebb mert minden egyes egység közölt olvasási és írási műveletek végrehajtását, amelyek esetleg felfedi a hibás környezetből adatokat.</span><span class="sxs-lookup"><span data-stu-id="cc721-125">It can make managing security and permissions more complex because each entity is subject to both read and write operations, which might expose data in the wrong context.</span></span>

> <span data-ttu-id="cc721-126">Bemutatják a CRUD megközelítés határain, lásd: [CRUD, csak ha akkor is biztosít az](https://blogs.msdn.microsoft.com/maarten_mullender/2004/07/23/crud-only-when-you-can-afford-it-revisited/).</span><span class="sxs-lookup"><span data-stu-id="cc721-126">For a deeper understanding of the limits of the CRUD approach see [CRUD, Only When You Can Afford It](https://blogs.msdn.microsoft.com/maarten_mullender/2004/07/23/crud-only-when-you-can-afford-it-revisited/).</span></span>

## <a name="solution"></a><span data-ttu-id="cc721-127">Megoldás</span><span class="sxs-lookup"><span data-stu-id="cc721-127">Solution</span></span>

<span data-ttu-id="cc721-128">Parancs és a lekérdezés felelősségi elkülönítése (CQRS) származik, a minta-adatok (lekérdezések) olvasása műveletek a rétegmodellről, amely a műveleteket (parancsok) adatokat frissítő külön-felületek használatával.</span><span class="sxs-lookup"><span data-stu-id="cc721-128">Command and Query Responsibility Segregation (CQRS) is a pattern that segregates the operations that read data (queries) from the operations that update data (commands) by using separate interfaces.</span></span> <span data-ttu-id="cc721-129">Ez azt jelenti, hogy az adatok modellek lekérdezésekor használja, és frissíti eltérőek.</span><span class="sxs-lookup"><span data-stu-id="cc721-129">This means that the data models used for querying and updates are different.</span></span> <span data-ttu-id="cc721-130">A modellek majd lehet különíteni, bár ez nem egy abszolút követelmény az alábbi ábrán látható módon.</span><span class="sxs-lookup"><span data-stu-id="cc721-130">The models can then be isolated, as shown in the following figure, although that's not an absolute requirement.</span></span>

![Egy alapszintű CQRS architektúrája](./_images/command-and-query-responsibility-segregation-cqrs-basic.png)

<span data-ttu-id="cc721-132">Az egyetlen adatmodell CRUD-alapú rendszerekben használt képest, CQRS-alapú rendszerek adatainak külön lekérdezési és frissítési modellek használatának egyszerűbbé teszi a tervezési és megvalósítási.</span><span class="sxs-lookup"><span data-stu-id="cc721-132">Compared to the single data model used in CRUD-based systems, the use of separate query and update models for the data in CQRS-based systems simplifies design and implementation.</span></span> <span data-ttu-id="cc721-133">Azonban egyik hátránya, hogy eltérően CRUD tervek CQRS kód nem automatikusan jön létre scaffold mechanizmusok használatával.</span><span class="sxs-lookup"><span data-stu-id="cc721-133">However, one disadvantage is that unlike CRUD designs, CQRS code can't automatically be generated using scaffold mechanisms.</span></span>

<span data-ttu-id="cc721-134">A lekérdezési modelljét olvasási adatok és a frissítési modell az adatok írása ugyanarra a fizikai tároló, elérheti, lehet, hogy az SQL-nézetek használatával vagy parancsprogramok leképezések létrehozásakor.</span><span class="sxs-lookup"><span data-stu-id="cc721-134">The query model for reading data and the update model for writing data can access the same physical store, perhaps by using SQL views or by generating projections on the fly.</span></span> <span data-ttu-id="cc721-135">Azonban általában külön az adatok különböző fizikai tárolók maximalizálása a teljesítmény, méretezhetőség és biztonság, a következő ábrán látható módon.</span><span class="sxs-lookup"><span data-stu-id="cc721-135">However, it's common to separate the data into different physical stores to maximize performance, scalability, and security, as shown in the next figure.</span></span>

![Egy, külön olvasási és írási tárolók CQRS architektúrája](./_images/command-and-query-responsibility-segregation-cqrs-separate-stores.png)

<span data-ttu-id="cc721-137">Az olvasási tároló lehet egy csak olvasható replika az írási tároló, vagy olvasási és írási tárolókat, hogy egy másik struktúra regisztrálását.</span><span class="sxs-lookup"><span data-stu-id="cc721-137">The read store can be a read-only replica of the write store, or the read and write stores can have a different structure altogether.</span></span> <span data-ttu-id="cc721-138">Az olvasási tároló több csak olvasható replika használatával jelentősen növelheti lekérdezés teljesítmény- és alkalmazás felhasználói felületén reakcióidőt, különösen az elosztott forgatókönyvek, ahol csak olvasható replika találhatók megközelíti a alkalmazáspéldányok.</span><span class="sxs-lookup"><span data-stu-id="cc721-138">Using multiple read-only replicas of the read store can greatly increase query performance and application UI responsiveness, especially in distributed scenarios where read-only replicas are located close to the application instances.</span></span> <span data-ttu-id="cc721-139">Egyes adatbázis-rendszerek (SQL Server) adja meg a további funkciók, például a feladatátvételi replikák rendelkezésre állásának maximalizálását.</span><span class="sxs-lookup"><span data-stu-id="cc721-139">Some database systems (SQL Server) provide additional features such as failover replicas to maximize availability.</span></span>

<span data-ttu-id="cc721-140">Olvasási és írási tárolókat elválasztását is lehetővé teszi, hogy mindegyik felel meg a betöltési megfelelően méretezhető.</span><span class="sxs-lookup"><span data-stu-id="cc721-140">Separation of the read and write stores also allows each to be scaled appropriately to match the load.</span></span> <span data-ttu-id="cc721-141">Olvasási tárolók például jellemzően előforduló sokkal nagyobb terhelést, mint a tárolók írása.</span><span class="sxs-lookup"><span data-stu-id="cc721-141">For example, read stores typically encounter a much higher load than write stores.</span></span>

<span data-ttu-id="cc721-142">Ha a lekérdezés/olvasás modell tartalmaz a nem normalizált adatok (lásd: [materializált nézet mintát](materialized-view.md)), teljesítmény teljes méretű adatok az egyes nézetek egy alkalmazás olvasásakor, vagy a rendszer az adatok lekérdezésekor.</span><span class="sxs-lookup"><span data-stu-id="cc721-142">When the query/read model contains denormalized data (see [Materialized View pattern](materialized-view.md)), performance is maximized when reading data for each of the views in an application or when querying the data in the system.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="cc721-143">Problémákat és szempontok</span><span class="sxs-lookup"><span data-stu-id="cc721-143">Issues and considerations</span></span>

<span data-ttu-id="cc721-144">Ebben a mintában megvalósításához meghatározásakor, vegye figyelembe a következő szempontokat:</span><span class="sxs-lookup"><span data-stu-id="cc721-144">Consider the following points when deciding how to implement this pattern:</span></span>

- <span data-ttu-id="cc721-145">Az adatok felosztásával tárolás külön fizikai tárolók a Read és írási műveletek növelheti a teljesítményt és a rendszer biztonsági, de azt is hozzáadhat a rugalmasság és a végleges konzisztencia tekintetében összetettségi.</span><span class="sxs-lookup"><span data-stu-id="cc721-145">Dividing the data store into separate physical stores for read and write operations can increase the performance and security of a system, but it can add complexity in terms of resiliency and eventual consistency.</span></span> <span data-ttu-id="cc721-146">Az olvasási modell tároló frissíteni kell, hogy az írási modell tároló módosításait, és nehézkes észlelés, ha egy felhasználó egy kérelem alapján elavult adatolvasási, ami azt jelenti, hogy a művelet nem fejezhető ki lehet.</span><span class="sxs-lookup"><span data-stu-id="cc721-146">The read model store must be updated to reflect changes to the write model store, and it can be difficult to detect when a user has issued a request based on stale read data, which means that the operation can't be completed.</span></span>

    > <span data-ttu-id="cc721-147">A végleges konzisztencia leírását lásd a [adatok konzisztencia ismertetése](https://msdn.microsoft.com/library/dn589800.aspx).</span><span class="sxs-lookup"><span data-stu-id="cc721-147">For a description of eventual consistency see the [Data Consistency Primer](https://msdn.microsoft.com/library/dn589800.aspx).</span></span>

- <span data-ttu-id="cc721-148">Érdemes lehet a rendszer korlátozott szakaszok a legyenek számára a legértékesebb CQRS alkalmazni.</span><span class="sxs-lookup"><span data-stu-id="cc721-148">Consider applying CQRS to limited sections of your system where it will be most valuable.</span></span>

- <span data-ttu-id="cc721-149">Egy tipikus megoldás telepítésével a végleges konzisztencia, esemény, hogy az írási modell egy hozzáfűző csak az események streamjét a parancsok végrehajtása által vezérelt forrás CQRS együtt használandó.</span><span class="sxs-lookup"><span data-stu-id="cc721-149">A typical approach to deploying eventual consistency is to use event sourcing in conjunction with CQRS so that the write model is an append-only stream of events driven by execution of commands.</span></span> <span data-ttu-id="cc721-150">Ezek az események materializált nézet olvasási modellként viselkedő szolgálnak.</span><span class="sxs-lookup"><span data-stu-id="cc721-150">These events are used to update materialized views that act as the read model.</span></span> <span data-ttu-id="cc721-151">További információ: [esemény forrás- és CQRS](https://msdn.microsoft.com/library/dn568103.aspx#EventSourcingandCQRS).</span><span class="sxs-lookup"><span data-stu-id="cc721-151">For more information see [Event Sourcing and CQRS](https://msdn.microsoft.com/library/dn568103.aspx#EventSourcingandCQRS).</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="cc721-152">Mikor érdemes használni ezt a mintát</span><span class="sxs-lookup"><span data-stu-id="cc721-152">When to use this pattern</span></span>

<span data-ttu-id="cc721-153">Használja ezt a mintát a következő esetekben:</span><span class="sxs-lookup"><span data-stu-id="cc721-153">Use this pattern in the following situations:</span></span>

- <span data-ttu-id="cc721-154">Együttműködési tartományok, ahol több műveleteket, párhuzamos ugyanazokat az adatokat.</span><span class="sxs-lookup"><span data-stu-id="cc721-154">Collaborative domains where multiple operations are performed in parallel on the same data.</span></span> <span data-ttu-id="cc721-155">CQRS adhatók meg minimálisra csökkentése érdekében a tartományi szinten (felmerülő ütközés egyesíthető a parancs), egyesítési ütközések elég lépésköz parancsok is megtalálható az azonos típusú adatok frissítésekor.</span><span class="sxs-lookup"><span data-stu-id="cc721-155">CQRS allows you to define commands with enough granularity to minimize merge conflicts at the domain level (any conflicts that do arise can be merged by the command), even when updating what appears to be the same type of data.</span></span>

- <span data-ttu-id="cc721-156">Feladat-alapú felhasználói felületek, ahol felhasználók végigvezeti egy összetett folyamat lépésből áll, vagy összetett tartomány modellekkel való sorozataként.</span><span class="sxs-lookup"><span data-stu-id="cc721-156">Task-based user interfaces where users are guided through a complex process as a series of steps or with complex domain models.</span></span> <span data-ttu-id="cc721-157">Ezenkívül hasznos csapatának már ismeri a tartomány-központú kialakítás (nnn) technikák.</span><span class="sxs-lookup"><span data-stu-id="cc721-157">Also, useful for teams already familiar with domain-driven design (DDD) techniques.</span></span> <span data-ttu-id="cc721-158">Az írási modellnek van egy parancs feldolgozása verem üzleti logikát, bemenet-ellenőrzéshez és üzleti-ellenőrzéssel győződjön meg arról, hogy minden mindig konzisztens minden, a aggregátumok (minden az adatok változásait egy egységként kezelt kapcsolódó objektumok fürt) a az írási modell.</span><span class="sxs-lookup"><span data-stu-id="cc721-158">The write model has a full command-processing stack with business logic, input validation, and business validation to ensure that everything is always consistent for each of the aggregates (each cluster of associated objects treated as a unit for data changes) in the write model.</span></span> <span data-ttu-id="cc721-159">Az olvasási modell üzleti logika és érvényesítési verem rendelkezik, és csak egy nézet modell adja vissza egy DTO való használatra.</span><span class="sxs-lookup"><span data-stu-id="cc721-159">The read model has no business logic or validation stack and just returns a DTO for use in a view model.</span></span> <span data-ttu-id="cc721-160">Az olvasási modell idővel konzisztenssé váljanak a írási modell.</span><span class="sxs-lookup"><span data-stu-id="cc721-160">The read model is eventually consistent with the write model.</span></span>

- <span data-ttu-id="cc721-161">Forgatókönyvek, ahol adatok olvasási műveletek teljesítményének finom kell beállított külön teljesítményének adatok írási műveletekről, különösen akkor, ha az olvasási/írási arányt nagyon magas, és ha horizontális skálázás szükség.</span><span class="sxs-lookup"><span data-stu-id="cc721-161">Scenarios where performance of data reads must be fine tuned separately from performance of data writes, especially when the read/write ratio is very high, and when horizontal scaling is required.</span></span> <span data-ttu-id="cc721-162">Például számos rendszer beolvasott száma műveletek van nagyobb sokszor, amely írási műveletek száma.</span><span class="sxs-lookup"><span data-stu-id="cc721-162">For example, in many systems the number of read operations is many times greater that the number of write operations.</span></span> <span data-ttu-id="cc721-163">Ez teljesítése érdekében fontolja meg az olvasási modell kiterjesztése, de az írási modell futó csak egy vagy néhány példányát.</span><span class="sxs-lookup"><span data-stu-id="cc721-163">To accommodate this, consider scaling out the read model, but running the write model on only one or a few instances.</span></span> <span data-ttu-id="cc721-164">A kis mennyiségű írási modell példánnyal is segíti a egyesítési ütközések előfordulása minimalizálása érdekében.</span><span class="sxs-lookup"><span data-stu-id="cc721-164">A small number of write model instances also helps to minimize the occurrence of merge conflicts.</span></span>

- <span data-ttu-id="cc721-165">Azok az esetek, amikor a fejlesztők egy csoportot, amely része a írási modell összetett tartománymodell összpontosíthat, így egy másik csapatban az olvasási és a felhasználói felületek összpontosíthat.</span><span class="sxs-lookup"><span data-stu-id="cc721-165">Scenarios where one team of developers can focus on the complex domain model that is part of the write model, and another team can focus on the read model and the user interfaces.</span></span>

- <span data-ttu-id="cc721-166">Esetek, amikor a rendszer várhatóan verzióinformációk, így a modell több verziót tartalmaz, vagy ha az üzleti szabályok rendszeresen módosítsa.</span><span class="sxs-lookup"><span data-stu-id="cc721-166">Scenarios where the system is expected to evolve over time and might contain multiple versions of the model, or where business rules change regularly.</span></span>

- <span data-ttu-id="cc721-167">Integráció más rendszerekkel, különösen, valamint esemény forrás, amennyiben a egy alrendszer historikus hibája nem befolyásolja a többi rendelkezésre állását.</span><span class="sxs-lookup"><span data-stu-id="cc721-167">Integration with other systems, especially in combination with event sourcing, where the temporal failure of one subsystem shouldn't affect the availability of the others.</span></span>

<span data-ttu-id="cc721-168">Ez a minta a következő esetekben nem ajánlott:</span><span class="sxs-lookup"><span data-stu-id="cc721-168">This pattern isn't recommended in the following situations:</span></span>

- <span data-ttu-id="cc721-169">A tartomány vagy az üzleti szabályok esetén egyszerű.</span><span class="sxs-lookup"><span data-stu-id="cc721-169">Where the domain or the business rules are simple.</span></span>

- <span data-ttu-id="cc721-170">Ha egy egyszerű CRUD-működő felhasználói felület és a kapcsolódó adatelérési műveletek elegendőek.</span><span class="sxs-lookup"><span data-stu-id="cc721-170">Where a simple CRUD-style user interface and the related data access operations are sufficient.</span></span>

- <span data-ttu-id="cc721-171">A teljes rendszer végrehajtásához.</span><span class="sxs-lookup"><span data-stu-id="cc721-171">For implementation across the whole system.</span></span> <span data-ttu-id="cc721-172">Vannak olyan általános felügyeleti forgatókönyvet ahol CQRS hasznosak lehetnek, de jelentős és szükségtelen összetettségét, adhat hozzá, amikor nem szükséges az egyes összetevőinek.</span><span class="sxs-lookup"><span data-stu-id="cc721-172">There are specific components of an overall data management scenario where CQRS can be useful, but it can add considerable and unnecessary complexity when it isn't required.</span></span>

## <a name="event-sourcing-and-cqrs"></a><span data-ttu-id="cc721-173">Esemény forrás- és CQRS</span><span class="sxs-lookup"><span data-stu-id="cc721-173">Event Sourcing and CQRS</span></span>

<span data-ttu-id="cc721-174">CQRS mintát gyakran használják az esemény forrás mintát együtt.</span><span class="sxs-lookup"><span data-stu-id="cc721-174">The CQRS pattern is often used along with the Event Sourcing pattern.</span></span> <span data-ttu-id="cc721-175">CQRS-alapú rendszerek külön olvasási használja, és írja adatmodellekben, minden egyes igazított kapcsolódó feladatok, és gyakran fizikailag elkülönülő tárolóban található.</span><span class="sxs-lookup"><span data-stu-id="cc721-175">CQRS-based systems use separate read and write data models, each tailored to relevant tasks and often located in physically separate stores.</span></span> <span data-ttu-id="cc721-176">Használata esetén a [esemény forrás](event-sourcing.md) mintát, a tároló események írási modellje, és a hivatalos információforrás.</span><span class="sxs-lookup"><span data-stu-id="cc721-176">When used with the [Event Sourcing](event-sourcing.md) pattern, the store of events is the write model, and is the official source of information.</span></span> <span data-ttu-id="cc721-177">A CQRS rendszerbe olvasási modelljét biztosít az adatok, materializált nézetek általában magas denormalizált nézetek.</span><span class="sxs-lookup"><span data-stu-id="cc721-177">The read model of a CQRS-based system provides materialized views of the data, typically as highly denormalized views.</span></span> <span data-ttu-id="cc721-178">Ezek a nézetek a felületek vannak szabva, és megjeleníti az alkalmazás, amely segít a maximalizálása érdekében is megjelennek, illetve lekérdezési teljesítmény követelményeinek.</span><span class="sxs-lookup"><span data-stu-id="cc721-178">These views are tailored to the interfaces and display requirements of the application, which helps to maximize both display and query performance.</span></span>

<span data-ttu-id="cc721-179">Az események streamjét használata az írási tároló ahelyett, hogy a tényleges adatok egy időben, ezzel elkerülheti a egyetlen összesítő frissítés ütközéseket, és a lehető legnagyobbra teljesítményét és méretezhetőségét.</span><span class="sxs-lookup"><span data-stu-id="cc721-179">Using the stream of events as the write store, rather than the actual data at a point in time, avoids update conflicts on a single aggregate and maximizes performance and scalability.</span></span> <span data-ttu-id="cc721-180">Az események aszinkron módon az olvasási tároló feltöltésére használatos olyan adatok materializált nézetek létrehozása céljából használható.</span><span class="sxs-lookup"><span data-stu-id="cc721-180">The events can be used to asynchronously generate materialized views of the data that are used to populate the read store.</span></span>

<span data-ttu-id="cc721-181">Mivel az esemény tároló információk hivatalos forrását, úgy is módosíthat a materializált nézeteket, és a korábbi eseményeket, hozzon létre egy új ábrázolása aktuális állapotát, ha a rendszer követi, vagy ha módosítania kell az olvasási modell visszajátszásos.</span><span class="sxs-lookup"><span data-stu-id="cc721-181">Because the event store is the official source of information, it is possible to delete the materialized views and replay all past events to create a new representation of the current state when the system evolves, or when the read model must change.</span></span> <span data-ttu-id="cc721-182">A materializált nézetekhez érvényben az adatok tartósságát, csak olvasható gyorsítótárának.</span><span class="sxs-lookup"><span data-stu-id="cc721-182">The materialized views are in effect a durable read-only cache of the data.</span></span>

<span data-ttu-id="cc721-183">Az esemény forrás mintát együtt CQRS használ, vegye figyelembe a következőket:</span><span class="sxs-lookup"><span data-stu-id="cc721-183">When using CQRS combined with the Event Sourcing pattern, consider the following:</span></span>

- <span data-ttu-id="cc721-184">Csakúgy, mint bármely rendszer, ha külön-e írási és olvasási tárolókat, ez a minta alapján rendszereket csak idővel konzisztenssé.</span><span class="sxs-lookup"><span data-stu-id="cc721-184">As with any system where the write and read stores are separate, systems based on this pattern are only eventually consistent.</span></span> <span data-ttu-id="cc721-185">Néhány generálása az esemény és az adattár frissítendő késleltetés lesz.</span><span class="sxs-lookup"><span data-stu-id="cc721-185">There will be some delay between the event being generated and the data store being updated.</span></span>

- <span data-ttu-id="cc721-186">A minta összetettsége ad hozzá, mert kód kezdeményezése és kezelhetnek eseményeket, és állítsa össze vagy a megfelelő nézet, és a lekérdezések, illetve egy olvasási modell által igényelt objektumokat frissíteni kell létrehozni.</span><span class="sxs-lookup"><span data-stu-id="cc721-186">The pattern adds complexity because code must be created to initiate and handle events, and assemble or update the appropriate views or objects required by queries or a read model.</span></span> <span data-ttu-id="cc721-187">Az esemény forrás mintázat használata esetén a CQRS minta összetettsége is megnehezítik sikeres végrehajtása, és a rendszer tervezése eltérő megközelítést igényel.</span><span class="sxs-lookup"><span data-stu-id="cc721-187">The complexity of the CQRS pattern when used with the Event Sourcing pattern can make a successful implementation more difficult, and requires a different approach to designing systems.</span></span> <span data-ttu-id="cc721-188">Azonban esemény forrás megkönnyítheti a modell a tartományhoz, és megkönnyíti a nézetek építse újra, vagy újakat létrehozni, mert az adatok változásait szándékával megőrződik.</span><span class="sxs-lookup"><span data-stu-id="cc721-188">However, event sourcing can make it easier to model the domain, and makes it easier to rebuild views or create new ones because the intent of the changes in the data is preserved.</span></span>

- <span data-ttu-id="cc721-189">Generálása materializált nézetek az olvasási modell használatra, vagy az adatok visszajátszását, és konkrét személyek vagy entitások gyűjteményét események kezelésére leképezések megkövetelheti jelentős feldolgozási idő és erőforrás-használat.</span><span class="sxs-lookup"><span data-stu-id="cc721-189">Generating materialized views for use in the read model or projections of the data by replaying and handling the events for specific entities or collections of entities can require significant processing time and resource usage.</span></span> <span data-ttu-id="cc721-190">Ez azért különösen igaz, ha hosszú távú igényli összegzési vagy értékek elemzése, mert előfordulhat, hogy a kapcsolódó események kell vizsgálni.</span><span class="sxs-lookup"><span data-stu-id="cc721-190">This is especially true if it requires summation or analysis of values over long periods, because all the associated events might need to be examined.</span></span> <span data-ttu-id="cc721-191">A megoldást pillanatképeket készíteni a végrehajtási rendszeres időközönként, például egy adott művelet történt teljes száma, vagy egy entitás jelenlegi állapotával.</span><span class="sxs-lookup"><span data-stu-id="cc721-191">Resolve this by implementing snapshots of the data at scheduled intervals, such as a total count of the number of a specific action that have occurred, or the current state of an entity.</span></span>

## <a name="example"></a><span data-ttu-id="cc721-192">Példa</span><span class="sxs-lookup"><span data-stu-id="cc721-192">Example</span></span>

<span data-ttu-id="cc721-193">A következő kód bemutatja, néhány példa a CQRS megvalósítása, amely más definíciók használ az olvasási és írási modellek kivonata.</span><span class="sxs-lookup"><span data-stu-id="cc721-193">The following code shows some extracts from an example of a CQRS implementation that uses different definitions for the read and the write models.</span></span> <span data-ttu-id="cc721-194">A modell felületek nem előírják a mögöttes adatokat tároló jellemzője, és fejlődnek, és lehet, hogy finomhangolható egymástól függetlenül konfigurációkezelővel elkülönül egymástól.</span><span class="sxs-lookup"><span data-stu-id="cc721-194">The model interfaces don't dictate any features of the underlying data stores, and they can evolve and be fine-tuned independently because these interfaces are separated.</span></span>

<span data-ttu-id="cc721-195">A következő kód bemutatja az olvasási modell-definícióban.</span><span class="sxs-lookup"><span data-stu-id="cc721-195">The following code shows the read model definition.</span></span>

```csharp
// Query interface
namespace ReadModel
{
  public interface ProductsDao
  {
    ProductDisplay FindById(int productId);
    ICollection<ProductDisplay> FindByName(string name);
    ICollection<ProductInventory> FindOutOfStockProducts();
    ICollection<ProductDisplay> FindRelatedProducts(int productId);
  }

  public class ProductDisplay
  {
    public int Id { get; set; }
    public string Name { get; set; }
    public string Description { get; set; }
    public decimal UnitPrice { get; set; }
    public bool IsOutOfStock { get; set; }
    public double UserRating { get; set; }
  }

  public class ProductInventory
  {
    public int Id { get; set; }
    public string Name { get; set; }
    public int CurrentStock { get; set; }
  }
}
```

<span data-ttu-id="cc721-196">A rendszer lehetővé teszi a felhasználóknak arány termékek.</span><span class="sxs-lookup"><span data-stu-id="cc721-196">The system allows users to rate products.</span></span> <span data-ttu-id="cc721-197">Az alkalmazás kódjának elvégzi ezt használja a `RateProduct` parancs az alábbi kódban látható.</span><span class="sxs-lookup"><span data-stu-id="cc721-197">The application code does this using the `RateProduct` command shown in the following code.</span></span>

```csharp
public interface ICommand
{
  Guid Id { get; }
}

public class RateProduct : ICommand
{
  public RateProduct()
  {
    this.Id = Guid.NewGuid();
  }
  public Guid Id { get; set; }
  public int ProductId { get; set; }
  public int Rating { get; set; }
  public int UserId {get; set; }
}
```

<span data-ttu-id="cc721-198">A rendszer használja a `ProductsCommandHandler` osztály kezelni az alkalmazás által küldött parancsokat.</span><span class="sxs-lookup"><span data-stu-id="cc721-198">The system uses the `ProductsCommandHandler` class to handle commands sent by the application.</span></span> <span data-ttu-id="cc721-199">Ügyfelek általában parancsainak elküldését a tartomány egy üzenetkezelési rendszerén, például egy üzenetsort keresztül.</span><span class="sxs-lookup"><span data-stu-id="cc721-199">Clients typically send commands to the domain through a messaging system such as a queue.</span></span> <span data-ttu-id="cc721-200">A parancskezelő elfogadja ezeket a parancsokat, és a tartományi kapcsolat módszereket hív.</span><span class="sxs-lookup"><span data-stu-id="cc721-200">The command handler accepts these commands and invokes methods of the domain interface.</span></span> <span data-ttu-id="cc721-201">Az egyes parancsok granularitási célja, hogy csökkenti annak esélyét ütköző kérelmek.</span><span class="sxs-lookup"><span data-stu-id="cc721-201">The granularity of each command is designed to reduce the chance of conflicting requests.</span></span> <span data-ttu-id="cc721-202">A következő kód bemutatja áttekintése, amelyek a `ProductsCommandHandler` osztály.</span><span class="sxs-lookup"><span data-stu-id="cc721-202">The following code shows an outline of the `ProductsCommandHandler` class.</span></span>

```csharp
public class ProductsCommandHandler :
    ICommandHandler<AddNewProduct>,
    ICommandHandler<RateProduct>,
    ICommandHandler<AddToInventory>,
    ICommandHandler<ConfirmItemShipped>,
    ICommandHandler<UpdateStockFromInventoryRecount>
{
  private readonly IRepository<Product> repository;

  public ProductsCommandHandler (IRepository<Product> repository)
  {
    this.repository = repository;
  }

  void Handle (AddNewProduct command)
  {
    ...
  }

  void Handle (RateProduct command)
  {
    var product = repository.Find(command.ProductId);
    if (product != null)
    {
      product.RateProduct(command.UserId, command.Rating);
      repository.Save(product);
    }
  }

  void Handle (AddToInventory command)
  {
    ...
  }

  void Handle (ConfirmItemsShipped command)
  {
    ...
  }

  void Handle (UpdateStockFromInventoryRecount command)
  {
    ...
  }
}
```

<span data-ttu-id="cc721-203">A következő kódot tartalmazza a `IProductsDomain` írási modellből felületet.</span><span class="sxs-lookup"><span data-stu-id="cc721-203">The following code shows the `IProductsDomain` interface from the write model.</span></span>

```csharp
public interface IProductsDomain
{
  void AddNewProduct(int id, string name, string description, decimal price);
  void RateProduct(int userId, int rating);
  void AddToInventory(int productId, int quantity);
  void ConfirmItemsShipped(int productId, int quantity);
  void UpdateStockFromInventoryRecount(int productId, int updatedQuantity);
}
```

<span data-ttu-id="cc721-204">Is figyelje meg, hogyan a `IProductsDomain` felület, amely megegyezik a tartomány módszerek tartalmazza.</span><span class="sxs-lookup"><span data-stu-id="cc721-204">Also notice how the `IProductsDomain` interface contains methods that have a meaning in the domain.</span></span> <span data-ttu-id="cc721-205">Általában CRUD környezetben ezek a módszerek kellene általános nevek például `Save` vagy `Update`, és egy DTO egyetlen argumentumaként.</span><span class="sxs-lookup"><span data-stu-id="cc721-205">Typically, in a CRUD environment these methods would have generic names such as `Save` or `Update`, and have a DTO as the only argument.</span></span> <span data-ttu-id="cc721-206">A CQRS megközelítés is kell megtervezni, hogy a szervezet üzleti és a szoftverleltár-kezelő rendszerek igényeinek.</span><span class="sxs-lookup"><span data-stu-id="cc721-206">The CQRS approach can be designed to meet the needs of this organization's business and inventory management systems.</span></span>

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="cc721-207">Útmutató és a kapcsolódó minták</span><span class="sxs-lookup"><span data-stu-id="cc721-207">Related patterns and guidance</span></span>

<span data-ttu-id="cc721-208">A következő mintákat és útmutatókat hasznosak, ha ezt a mintát megvalósító:</span><span class="sxs-lookup"><span data-stu-id="cc721-208">The following patterns and guidance are useful when implementing this pattern:</span></span>

- <span data-ttu-id="cc721-209">Egy CQRS más architekturális stílusú, lásd: [architektúra stílusok](/azure/architecture/guide/architecture-styles/) és [CQRS architektúra stílus](/azure/architecture/guide/architecture-styles/cqrs).</span><span class="sxs-lookup"><span data-stu-id="cc721-209">For a comparison of CQRS with other architectural styles, see [Architecture styles](/azure/architecture/guide/architecture-styles/) and [CQRS architecture style](/azure/architecture/guide/architecture-styles/cqrs).</span></span>

- <span data-ttu-id="cc721-210">[Adatok konzisztencia ismertetése](https://msdn.microsoft.com/library/dn589800.aspx).</span><span class="sxs-lookup"><span data-stu-id="cc721-210">[Data Consistency Primer](https://msdn.microsoft.com/library/dn589800.aspx).</span></span> <span data-ttu-id="cc721-211">A problémákat, amelyek között az olvasás a végleges konzisztencia miatt általában hibát, és adattároló írni a CQRS mintát, és hogyan oldhatók fel ezeket a problémákat ismerteti.</span><span class="sxs-lookup"><span data-stu-id="cc721-211">Explains the issues that are typically encountered due to eventual consistency between the read and write data stores when using the CQRS pattern, and how these issues can be resolved.</span></span>

- <span data-ttu-id="cc721-212">[Útmutatás particionálás adatok](https://msdn.microsoft.com/library/dn589795.aspx).</span><span class="sxs-lookup"><span data-stu-id="cc721-212">[Data Partitioning Guidance](https://msdn.microsoft.com/library/dn589795.aspx).</span></span> <span data-ttu-id="cc721-213">Ismerteti, hogyan CQRS mintájában használt olvasási és írási adattárolókhoz lehet osztani kívánt kezelhetők, és külön-külön érhető el, méretezhetőség javítása érdekében csökkentheti a versengés, és a teljesítmény optimalizálása.</span><span class="sxs-lookup"><span data-stu-id="cc721-213">Describes how the read and write data stores used in the CQRS pattern can be divided into partitions that can be managed and accessed separately to improve scalability, reduce contention, and optimize performance.</span></span>

- <span data-ttu-id="cc721-214">[Minta forrás esemény](event-sourcing.md).</span><span class="sxs-lookup"><span data-stu-id="cc721-214">[Event Sourcing Pattern](event-sourcing.md).</span></span> <span data-ttu-id="cc721-215">Részletesebben ismerteti, hogyan esemény forrás használható CQRS mintájú összetett tartományok feladatok egyszerűbbé teheti a teljesítmény, méretezhetőség és reakcióidőt fokozása mellett.</span><span class="sxs-lookup"><span data-stu-id="cc721-215">Describes in more detail how Event Sourcing can be used with the CQRS pattern to simplify tasks in complex domains while improving performance, scalability, and responsiveness.</span></span> <span data-ttu-id="cc721-216">Valamint hogy hogyan konzisztencia tranzakciós adatok teljes naplózási előzmények kezelése és engedélyezheti a kompenzációs műveletek előzményeinek megőrzése.</span><span class="sxs-lookup"><span data-stu-id="cc721-216">As well as how to provide consistency for transactional data while maintaining full audit trails and history that can enable compensating actions.</span></span>

- <span data-ttu-id="cc721-217">[A materializált nézet mintát](materialized-view.md).</span><span class="sxs-lookup"><span data-stu-id="cc721-217">[Materialized View Pattern](materialized-view.md).</span></span> <span data-ttu-id="cc721-218">A CQRS megvalósításának olvasási modelljét tartalmazhat írási modell adatok materializált nézeteket, vagy az olvasási modell használható materializált nézetek létrehozása céljából.</span><span class="sxs-lookup"><span data-stu-id="cc721-218">The read model of a CQRS implementation can contain materialized views of the write model data, or the read model can be used to generate materialized views.</span></span>

- <span data-ttu-id="cc721-219">A minták és gyakorlatok útmutató [CQRS út](http://aka.ms/cqrs).</span><span class="sxs-lookup"><span data-stu-id="cc721-219">The patterns & practices guide [CQRS Journey](http://aka.ms/cqrs).</span></span> <span data-ttu-id="cc721-220">Különösen [vezet be, a parancs lekérdezés felelősségi elkülönítése mintát](https://msdn.microsoft.com/library/jj591573.aspx) felderíti a mintázat és mikor célszerű, és [zárszó: megszerzett tapasztalatok](https://msdn.microsoft.com/library/jj591568.aspx) segít megérteni a problémák némelyikéről amely kapja meg, ha ezt a mintát használja.</span><span class="sxs-lookup"><span data-stu-id="cc721-220">In particular, [Introducing the Command Query Responsibility Segregation Pattern](https://msdn.microsoft.com/library/jj591573.aspx) explores the pattern and when it's useful, and [Epilogue: Lessons Learned](https://msdn.microsoft.com/library/jj591568.aspx) helps you understand some of the issues that come up when using this pattern.</span></span>

- <span data-ttu-id="cc721-221">A post [által Anna Fowler CQRS](http://martinfowler.com/bliki/CQRS.html), amely mintát és hivatkozások használatának alapjait ismerteti más hasznos erőforrásokhoz.</span><span class="sxs-lookup"><span data-stu-id="cc721-221">The post [CQRS by Martin Fowler](http://martinfowler.com/bliki/CQRS.html), which explains the basics of the pattern and links to other useful resources.</span></span>

- <span data-ttu-id="cc721-222">[Greg Young bejegyzéseket](http://codebetter.com/gregyoung/), amely megismerkedhet a CQRS mintát sok aspektusait.</span><span class="sxs-lookup"><span data-stu-id="cc721-222">[Greg Young’s posts](http://codebetter.com/gregyoung/), which explore many aspects of the CQRS pattern.</span></span>
