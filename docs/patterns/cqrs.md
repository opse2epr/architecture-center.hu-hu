---
title: CQRS
description: Különböző felületek használatával elkülönítheti az adatolvasó műveleteket az adatfrissítő műveletektől.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- data-management
- design-implementation
- performance-scalability
ms.openlocfilehash: de9530f7dd55c0ce5460cd3b58ab9f216c9b5c8c
ms.sourcegitcommit: fb22348f917a76e30a6c090fcd4a18decba0b398
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/16/2018
ms.locfileid: "53450870"
---
# <a name="command-and-query-responsibility-segregation-cqrs-pattern"></a><span data-ttu-id="42ef3-104">Command and Query Responsibility Segregation (CQRS) minta</span><span class="sxs-lookup"><span data-stu-id="42ef3-104">Command and Query Responsibility Segregation (CQRS) pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="42ef3-105">Különböző felületek használatával elkülönítheti az adatolvasó műveleteket az adatfrissítő műveletektől.</span><span class="sxs-lookup"><span data-stu-id="42ef3-105">Segregate operations that read data from operations that update data by using separate interfaces.</span></span> <span data-ttu-id="42ef3-106">Ezzel maximalizálhatja a teljesítményt, a méretezhetőséget és a biztonságot.</span><span class="sxs-lookup"><span data-stu-id="42ef3-106">This can maximize performance, scalability, and security.</span></span> <span data-ttu-id="42ef3-107">Nagyfokú rugalmassága révén támogatja a rendszer fejlődését, és megakadályozza, hogy a frissítési parancsok a tartományi szint egyesítési ütközéseket okozzanak.</span><span class="sxs-lookup"><span data-stu-id="42ef3-107">Supports the evolution of the system over time through higher flexibility, and prevents update commands from causing merge conflicts at the domain level.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="42ef3-108">Kontextus és probléma</span><span class="sxs-lookup"><span data-stu-id="42ef3-108">Context and problem</span></span>

<span data-ttu-id="42ef3-109">A hagyományos adatkezelő rendszerek esetében mind a parancsokat (az adatok frissítéseit), mind a lekérdezéseket (az adatokra irányuló kéréseket) ugyanazon az egyetlen adattárban található entitáskészleten hajtja végre a rendszer.</span><span class="sxs-lookup"><span data-stu-id="42ef3-109">In traditional data management systems, both commands (updates to the data) and queries (requests for data) are executed against the same set of entities in a single data repository.</span></span> <span data-ttu-id="42ef3-110">Ezek az entitások lehetnek egy az SQL Serverhez hasonló relációs adatbázisban található egy vagy több tábla soraiból álló halmazok.</span><span class="sxs-lookup"><span data-stu-id="42ef3-110">These entities can be a subset of the rows in one or more tables in a relational database such as SQL Server.</span></span>

<span data-ttu-id="42ef3-111">Ezekben a rendszerekben általában minden létrehozási, olvasási, frissítési és törlési (CRUD) művelet az entitás ugyanazon formáján mennek végbe.</span><span class="sxs-lookup"><span data-stu-id="42ef3-111">Typically in these systems, all create, read, update, and delete (CRUD) operations are applied to the same representation of the entity.</span></span> <span data-ttu-id="42ef3-112">Például az adat-hozzáférési réteg (DAL) beolvas egy ügyfelet képviselő adatátviteli objektumot (DTO-t) az adattárból, és az megjelenik a képernyőn.</span><span class="sxs-lookup"><span data-stu-id="42ef3-112">For example, a data transfer object (DTO) representing a customer is retrieved from the data store by the data access layer (DAL) and displayed on the screen.</span></span> <span data-ttu-id="42ef3-113">A felhasználó frissíti a DTO egyes mezőit (például adatkötéssel), majd a DAL ismételten menti DTO-t az adattárban.</span><span class="sxs-lookup"><span data-stu-id="42ef3-113">A user updates some fields of the DTO (perhaps through data binding) and the DTO is then saved back in the data store by the DAL.</span></span> <span data-ttu-id="42ef3-114">A rendszer az olvasási és írási műveletetekhez is ugyanazt a DTO-t használja.</span><span class="sxs-lookup"><span data-stu-id="42ef3-114">The same DTO is used for both the read and write operations.</span></span> <span data-ttu-id="42ef3-115">Az ábrán egy hagyományos CRUD-architektúra látható.</span><span class="sxs-lookup"><span data-stu-id="42ef3-115">The figure illustrates a traditional CRUD architecture.</span></span>

![Hagyományos CRUD-architektúra](./_images/command-and-query-responsibility-segregation-cqrs-tradition-crud.png)

<span data-ttu-id="42ef3-117">A hagyományos CRUD-kialakítások akkor működnek jól, ha csak korlátozott üzleti logikát alkalmaz az adatműveletekre.</span><span class="sxs-lookup"><span data-stu-id="42ef3-117">Traditional CRUD designs work well when only limited business logic is applied to the data operations.</span></span> <span data-ttu-id="42ef3-118">A fejlesztői eszközök által biztosított felépítő mechanizmus nagyon gyorsan képes az adat-hozzáférési kód létrehozására, amely aztán szükség szerint testreszabható.</span><span class="sxs-lookup"><span data-stu-id="42ef3-118">Scaffold mechanisms provided by development tools can create data access code very quickly, which can then be customized as required.</span></span>

<span data-ttu-id="42ef3-119">A hagyományos CRUD megközelítésnek azonban néhány hátránya is van:</span><span class="sxs-lookup"><span data-stu-id="42ef3-119">However, the traditional CRUD approach has some disadvantages:</span></span>

- <span data-ttu-id="42ef3-120">Ez gyakran azt jelenti, hogy az adatok írási és olvasási ábrázolása nem egyezik, például az egyik több oszloppal vagy tulajdonsággal rendelkezik, amelyeket akkor is megfelelően kell frissíteni, ha nem szükségesek egy adott művelethez.</span><span class="sxs-lookup"><span data-stu-id="42ef3-120">It often means that there's a mismatch between the read and write representations of the data, such as additional columns or properties that must be updated correctly even though they aren't required as part of an operation.</span></span>

- <span data-ttu-id="42ef3-121">Fennáll az adatok közti verseny kialakulásának veszélye, ha a rekordok egy olyan együttműködési tartomány adattárában vannak zárolva, amelyben ugyanazon az adatkészleten párhuzamosan több aktor is végez műveleteket.</span><span class="sxs-lookup"><span data-stu-id="42ef3-121">It risks data contention when records are locked in the data store in a collaborative domain, where multiple actors operate in parallel on the same set of data.</span></span> <span data-ttu-id="42ef3-122">Illetve ha optimista zárolás használatakor az egyidejű frissítések ütköznek egymással.</span><span class="sxs-lookup"><span data-stu-id="42ef3-122">Or update conflicts caused by concurrent updates when optimistic locking is used.</span></span> <span data-ttu-id="42ef3-123">Az említett kockázatok esélye a rendszer összetettségének és teljesítményének növekedésével egyre nagyobb lesz.</span><span class="sxs-lookup"><span data-stu-id="42ef3-123">These risks increase as the complexity and throughput of the system grows.</span></span> <span data-ttu-id="42ef3-124">Emellett a hagyományos megközelítés negatív hatással lehet a teljesítményre az adattárra és az adatelérési rétegre eső terhelés, valamint a lekérdezések az adatok beolvasásához szükséges bonyolultsága miatt.</span><span class="sxs-lookup"><span data-stu-id="42ef3-124">In addition, the traditional approach can have a negative effect on performance due to load on the data store and data access layer, and the complexity of queries required to retrieve information.</span></span>

- <span data-ttu-id="42ef3-125">Ez bonyolultabbá teheti a biztonság és az engedélyek kezelését, mivel minden entitáson végrehajthatók olvasási és írási műveletek is, ezáltal az adatok nem megfelelő kontextusban is megjelenhetnek.</span><span class="sxs-lookup"><span data-stu-id="42ef3-125">It can make managing security and permissions more complex because each entity is subject to both read and write operations, which might expose data in the wrong context.</span></span>

## <a name="solution"></a><span data-ttu-id="42ef3-126">Megoldás</span><span class="sxs-lookup"><span data-stu-id="42ef3-126">Solution</span></span>

<span data-ttu-id="42ef3-127">A Command and Query Responsibility Segregation (CQRS) olyan minta, amely külön felületek használatával elválasztja az adatolvasási műveleteket (lekérdezéseket) az adatfrissítési műveletektől (parancsoktól).</span><span class="sxs-lookup"><span data-stu-id="42ef3-127">Command and Query Responsibility Segregation (CQRS) is a pattern that segregates the operations that read data (queries) from the operations that update data (commands) by using separate interfaces.</span></span> <span data-ttu-id="42ef3-128">Ez azt jelenti, hogy a lekérdezésekhez és a frissítésekhez nem ugyanazt az adatmodellt használja.</span><span class="sxs-lookup"><span data-stu-id="42ef3-128">This means that the data models used for querying and updates are different.</span></span> <span data-ttu-id="42ef3-129">A modellek az ábrán látható módon elkülöníthetők, bár ez nem alapvető követelmény.</span><span class="sxs-lookup"><span data-stu-id="42ef3-129">The models can then be isolated, as shown in the following figure, although that's not an absolute requirement.</span></span>

![Alapszintű CQRS-architektúra](./_images/command-and-query-responsibility-segregation-cqrs-basic.png)

<span data-ttu-id="42ef3-131">A CRUD-alapú rendszerekben használt egyetlen adatmodellhez képest a CQRS-alapú rendszerekben a külön lekérdezési és frissítési modellek használata egyszerűbbé teszi a tervezést és a megvalósítást.</span><span class="sxs-lookup"><span data-stu-id="42ef3-131">Compared to the single data model used in CRUD-based systems, the use of separate query and update models for the data in CQRS-based systems simplifies design and implementation.</span></span> <span data-ttu-id="42ef3-132">Azonban ennek egyik hátránya, hogy a CRUD-kialakításoktól eltérően a CQRS-kód nem hozható létre automatikusan felépítő mechanizmusok használatával.</span><span class="sxs-lookup"><span data-stu-id="42ef3-132">However, one disadvantage is that unlike CRUD designs, CQRS code can't automatically be generated using scaffold mechanisms.</span></span>

<span data-ttu-id="42ef3-133">Az olvasási adatok lekérdezési modellje és az írási adatok frissítési modellje ugyanahhoz a fizikai tárolóhoz fér hozzá, például az SQL-nézetek vagy menet közben létrehozott leképezések használatával.</span><span class="sxs-lookup"><span data-stu-id="42ef3-133">The query model for reading data and the update model for writing data can access the same physical store, perhaps by using SQL views or by generating projections on the fly.</span></span> <span data-ttu-id="42ef3-134">Gyakori eljárás azonban az adatok külön fizikai tárolóban való elkülönítése a teljesítmény, a méretezhetőség és a biztonság maximalizálása érdekében, ahogy azt a következő ábra is mutatja.</span><span class="sxs-lookup"><span data-stu-id="42ef3-134">However, it's common to separate the data into different physical stores to maximize performance, scalability, and security, as shown in the next figure.</span></span>

![CQRS-architektúra külön olvasási és írási tárolókkal](./_images/command-and-query-responsibility-segregation-cqrs-separate-stores.png)

<span data-ttu-id="42ef3-136">Az olvasási tároló lehet az írási tároló csak olvasható replikája, de az olvasási és írási tárolók lehetnek teljesen eltérő szerkezetűek is.</span><span class="sxs-lookup"><span data-stu-id="42ef3-136">The read store can be a read-only replica of the write store, or the read and write stores can have a different structure altogether.</span></span> <span data-ttu-id="42ef3-137">Az olvasási tároló több, csak olvasható replikájának használatával jelentősen növelhető a lekérdezések teljesítménye és az alkalmazás felhasználói felületének válaszideje, különösen az elosztott forgatókönyvekben, amelyekben a csak olvasható replikák az alkalmazáspéldányokhoz közel találhatók.</span><span class="sxs-lookup"><span data-stu-id="42ef3-137">Using multiple read-only replicas of the read store can greatly increase query performance and application UI responsiveness, especially in distributed scenarios where read-only replicas are located close to the application instances.</span></span> <span data-ttu-id="42ef3-138">Egyes adatbázis-rendszerek (SQL Server) további funkciókat kínálnak, például a rendelkezésre állás maximalizálását szolgáló feladatátvételi replikákat.</span><span class="sxs-lookup"><span data-stu-id="42ef3-138">Some database systems (SQL Server) provide additional features such as failover replicas to maximize availability.</span></span>

<span data-ttu-id="42ef3-139">Az olvasási és írási tárolók különválasztása lehetővé teszi a terhelésnek megfelelő méretezésüket is.</span><span class="sxs-lookup"><span data-stu-id="42ef3-139">Separation of the read and write stores also allows each to be scaled appropriately to match the load.</span></span> <span data-ttu-id="42ef3-140">Az olvasási tárolók például jellemzően jóval nagyobb terhelésnek vannak kitéve, mint az írási tárolók.</span><span class="sxs-lookup"><span data-stu-id="42ef3-140">For example, read stores typically encounter a much higher load than write stores.</span></span>

<span data-ttu-id="42ef3-141">Ha a lekérdezési/olvasási modell denormalizált adatokat tartalmaz (lásd: [Materialized View minta](materialized-view.md)), akkor a teljesítmény az alkalmazásban szereplő egyes nézetek adatainak beolvasásakor vagy a rendszerbeli adatok lekérdezésekor maximalizálható.</span><span class="sxs-lookup"><span data-stu-id="42ef3-141">When the query/read model contains denormalized data (see [Materialized View pattern](materialized-view.md)), performance is maximized when reading data for each of the views in an application or when querying the data in the system.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="42ef3-142">Problémák és megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="42ef3-142">Issues and considerations</span></span>

<span data-ttu-id="42ef3-143">A minta megvalósítása során az alábbi pontokat vegye figyelembe:</span><span class="sxs-lookup"><span data-stu-id="42ef3-143">Consider the following points when deciding how to implement this pattern:</span></span>

- <span data-ttu-id="42ef3-144">Az adattárak az olvasási és írási műveletek számára külön fizikai tárolókban való elosztásával növelhető a rendszer teljesítménye és biztonsága, azonban ez a rugalmasság és a végleges konzisztencia szempontjából bonyolíthatja a helyzetet.</span><span class="sxs-lookup"><span data-stu-id="42ef3-144">Dividing the data store into separate physical stores for read and write operations can increase the performance and security of a system, but it can add complexity in terms of resiliency and eventual consistency.</span></span> <span data-ttu-id="42ef3-145">Az olvasási modell tárolóját frissíteni kell ahhoz, hogy megjelenjenek benne az írási modellen végzett módosítások, így nehéz lehet megállapítani, ha egy felhasználó elavult olvasási adatokon alapuló kérelmet bocsát ki, ami azt jelenti, hogy a művelet nem végezhető el.</span><span class="sxs-lookup"><span data-stu-id="42ef3-145">The read model store must be updated to reflect changes to the write model store, and it can be difficult to detect when a user has issued a request based on stale read data, which means that the operation can't be completed.</span></span>

    > <span data-ttu-id="42ef3-146">A végleges konzisztencia leírása az [Adatkonzisztencia – Ismertető](https://msdn.microsoft.com/library/dn589800.aspx) című cikkben található.</span><span class="sxs-lookup"><span data-stu-id="42ef3-146">For a description of eventual consistency see the [Data Consistency Primer](https://msdn.microsoft.com/library/dn589800.aspx).</span></span>

- <span data-ttu-id="42ef3-147">A CQRS-t érdemes lehet a rendszernek csak az olyan korlátozott részein használni, ahol a leginkább hasznos.</span><span class="sxs-lookup"><span data-stu-id="42ef3-147">Consider applying CQRS to limited sections of your system where it will be most valuable.</span></span>

- <span data-ttu-id="42ef3-148">A végleges konzisztencia kialakításának általános megközelítése az Event Sourcing és a CQRS együttes használata. Ily módon az írási modell egy hozzáfűzéssel bővíthető eseménystream, amelyet a parancsok végrehajtása vezérel.</span><span class="sxs-lookup"><span data-stu-id="42ef3-148">A typical approach to deploying eventual consistency is to use event sourcing in conjunction with CQRS so that the write model is an append-only stream of events driven by execution of commands.</span></span> <span data-ttu-id="42ef3-149">Ezek az események az olvasási modellként működő materializált nézetek frissítéséhez használatosak.</span><span class="sxs-lookup"><span data-stu-id="42ef3-149">These events are used to update materialized views that act as the read model.</span></span> <span data-ttu-id="42ef3-150">Bővebb információ: [Event Sourcing és CQRS](/azure/architecture/patterns/cqrs#event-sourcing-and-cqrs).</span><span class="sxs-lookup"><span data-stu-id="42ef3-150">For more information see [Event Sourcing and CQRS](/azure/architecture/patterns/cqrs#event-sourcing-and-cqrs).</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="42ef3-151">Mikor érdemes ezt a mintát használni?</span><span class="sxs-lookup"><span data-stu-id="42ef3-151">When to use this pattern</span></span>

<span data-ttu-id="42ef3-152">Az alábbi helyzetekben alkalmazza ezt a mintát:</span><span class="sxs-lookup"><span data-stu-id="42ef3-152">Use this pattern in the following situations:</span></span>

- <span data-ttu-id="42ef3-153">Együttműködési tartományok, ahol ugyanazokon az adatokon egyszerre több műveletet is végrehajt a rendszer.</span><span class="sxs-lookup"><span data-stu-id="42ef3-153">Collaborative domains where multiple operations are performed in parallel on the same data.</span></span> <span data-ttu-id="42ef3-154">A CQRS révén megfelelő részletességgel adhatók meg a parancsok a tartományi szint egyesítési ütközéseinek minimalizálása érdekében (bármely fellépő ütközés egyesíthető a parancs által), még akkor is, ha látszólag ugyanazon adattípus frissítéséről van szó.</span><span class="sxs-lookup"><span data-stu-id="42ef3-154">CQRS allows you to define commands with enough granularity to minimize merge conflicts at the domain level (any conflicts that do arise can be merged by the command), even when updating what appears to be the same type of data.</span></span>

- <span data-ttu-id="42ef3-155">Feladatalapú felhasználói felületek, ahol a felhasználók lépésenként haladnak végig egy összetett folyamaton, vagy összetett tartományi modellekkel.</span><span class="sxs-lookup"><span data-stu-id="42ef3-155">Task-based user interfaces where users are guided through a complex process as a series of steps or with complex domain models.</span></span> <span data-ttu-id="42ef3-156">Ezenkívül olyan csapatok számára is hasznos, akik már ismerik a tartományvezérelt tervezési (DDD) technikákat.</span><span class="sxs-lookup"><span data-stu-id="42ef3-156">Also, useful for teams already familiar with domain-driven design (DDD) techniques.</span></span> <span data-ttu-id="42ef3-157">Az írási modell egy üzleti logikát, bemenet-ellenőrzést és üzleti ellenőrzést tartalmazó teljes parancsfeldolgozási veremmel rendelkezik annak biztosítása érdekében, hogy mindig minden konzisztens legyen az írási modell minden egyes összesítésével (a társított objektumok minden egyes fürtjét az adatok változásának egységeként kezeli).</span><span class="sxs-lookup"><span data-stu-id="42ef3-157">The write model has a full command-processing stack with business logic, input validation, and business validation to ensure that everything is always consistent for each of the aggregates (each cluster of associated objects treated as a unit for data changes) in the write model.</span></span> <span data-ttu-id="42ef3-158">Az olvasási modell nem tartalmaz üzleti logikát és érvényesítési vermet, csupán egy DTO-t ad vissza a nézetmodellben való használatra.</span><span class="sxs-lookup"><span data-stu-id="42ef3-158">The read model has no business logic or validation stack and just returns a DTO for use in a view model.</span></span> <span data-ttu-id="42ef3-159">Az olvasási modell véglegesen konzisztens az írási modellel.</span><span class="sxs-lookup"><span data-stu-id="42ef3-159">The read model is eventually consistent with the write model.</span></span>

- <span data-ttu-id="42ef3-160">Olyan forgatókönyvek, amelyekben az adatolvasási teljesítményt az adatírási teljesítménytől külön kell finomhangolni, különösen, ha az olvasási/írási arány nagyon magas, illetve ha horizontális skálázásra van szükség.</span><span class="sxs-lookup"><span data-stu-id="42ef3-160">Scenarios where performance of data reads must be fine tuned separately from performance of data writes, especially when the read/write ratio is very high, and when horizontal scaling is required.</span></span> <span data-ttu-id="42ef3-161">Az olvasási műveletek száma például számos rendszerben sokszorta több az írási műveletek számánál.</span><span class="sxs-lookup"><span data-stu-id="42ef3-161">For example, in many systems the number of read operations is many times greater than the number of write operations.</span></span> <span data-ttu-id="42ef3-162">Ezen igény kiszolgálása érdekében fontolja meg az olvasási modell kiterjesztését, az írási modellt viszont egyetlen, vagy mindössze néhány példányon futtassa.</span><span class="sxs-lookup"><span data-stu-id="42ef3-162">To accommodate this, consider scaling out the read model, but running the write model on only one or a few instances.</span></span> <span data-ttu-id="42ef3-163">Alacsony számú írási modell-példány használatával csökkenthető az egyesítési ütközések előfordulása is.</span><span class="sxs-lookup"><span data-stu-id="42ef3-163">A small number of write model instances also helps to minimize the occurrence of merge conflicts.</span></span>

- <span data-ttu-id="42ef3-164">Olyan forgatókönyvek, amelyekben egy fejlesztői csapat az írási modell részét képező összetett tartományi modellre összpontosíthat, egy másik csapat pedig az olvasási modellre és a felhasználói felületekre.</span><span class="sxs-lookup"><span data-stu-id="42ef3-164">Scenarios where one team of developers can focus on the complex domain model that is part of the write model, and another team can focus on the read model and the user interfaces.</span></span>

- <span data-ttu-id="42ef3-165">Olyan forgatókönyvek, amelyekben a rendszer az idő előrehaladtával várhatóan fejlődni fog, így a modell több verzióját is tartalmazhatja, illetve ahol az üzleti szabályok rendszeresen változnak.</span><span class="sxs-lookup"><span data-stu-id="42ef3-165">Scenarios where the system is expected to evolve over time and might contain multiple versions of the model, or where business rules change regularly.</span></span>

- <span data-ttu-id="42ef3-166">Integráció más rendszerekkel, különösen az Event Sourcinggal kombinálva, amely esetben egy alrendszer átmeneti meghibásodása nem lehet hatással a többi alrendszer rendelkezésre állására.</span><span class="sxs-lookup"><span data-stu-id="42ef3-166">Integration with other systems, especially in combination with event sourcing, where the temporal failure of one subsystem shouldn't affect the availability of the others.</span></span>

<span data-ttu-id="42ef3-167">A minta használata a következő esetekben nem ajánlott:</span><span class="sxs-lookup"><span data-stu-id="42ef3-167">This pattern isn't recommended in the following situations:</span></span>

- <span data-ttu-id="42ef3-168">Ahol a tartomány vagy az üzleti szabályok egyszerűek.</span><span class="sxs-lookup"><span data-stu-id="42ef3-168">Where the domain or the business rules are simple.</span></span>

- <span data-ttu-id="42ef3-169">Ahol egy egyszerű CRUD stílusú felhasználói felület és az ahhoz kapcsolódó adat-hozzáférési műveletek is elegendőek.</span><span class="sxs-lookup"><span data-stu-id="42ef3-169">Where a simple CRUD-style user interface and the related data access operations are sufficient.</span></span>

- <span data-ttu-id="42ef3-170">A teljes rendszeren történő megvalósítás esetén.</span><span class="sxs-lookup"><span data-stu-id="42ef3-170">For implementation across the whole system.</span></span> <span data-ttu-id="42ef3-171">Egy általános adatkezelési forgatókönyvnek vannak olyan adott összetevői, amelyek esetében a CQRS hasznos lehet, viszont számottevő és szükségtelen összetettséget jelenthet, ha nincs rá igazán szükség.</span><span class="sxs-lookup"><span data-stu-id="42ef3-171">There are specific components of an overall data management scenario where CQRS can be useful, but it can add considerable and unnecessary complexity when it isn't required.</span></span>

## <a name="event-sourcing-and-cqrs"></a><span data-ttu-id="42ef3-172">Az Event Sourcing és a CQRS</span><span class="sxs-lookup"><span data-stu-id="42ef3-172">Event Sourcing and CQRS</span></span>

<span data-ttu-id="42ef3-173">A CQRS mintát gyakran használják az Event Sourcing mintával együtt.</span><span class="sxs-lookup"><span data-stu-id="42ef3-173">The CQRS pattern is often used along with the Event Sourcing pattern.</span></span> <span data-ttu-id="42ef3-174">A CQRS-alapú rendszerek külön olvasási és írási adatmodelleket használnak, amelyek mindegyike a kapcsolódó feladatokhoz van igazítva, és gyakran fizikailag elkülönített tárolóban találhatók.</span><span class="sxs-lookup"><span data-stu-id="42ef3-174">CQRS-based systems use separate read and write data models, each tailored to relevant tasks and often located in physically separate stores.</span></span> <span data-ttu-id="42ef3-175">Az [Event Sourcing](event-sourcing.md) minta használata esetén az események tárolója és az információk hivatalos forrása az írási modell.</span><span class="sxs-lookup"><span data-stu-id="42ef3-175">When used with the [Event Sourcing](event-sourcing.md) pattern, the store of events is the write model, and is the official source of information.</span></span> <span data-ttu-id="42ef3-176">A CQRS-alapú rendszerek olvasási modellje az adatok materializált nézetit biztosítja, általában nagy mértékben denormalizált nézetekként.</span><span class="sxs-lookup"><span data-stu-id="42ef3-176">The read model of a CQRS-based system provides materialized views of the data, typically as highly denormalized views.</span></span> <span data-ttu-id="42ef3-177">Ezek a nézetek az alkalmazás felhasználói felületeihez és megjelenítési követelményeihez vannak igazítva, ezzel pedig maximalizálható a megjelenítési és lekérdezési teljesítmény.</span><span class="sxs-lookup"><span data-stu-id="42ef3-177">These views are tailored to the interfaces and display requirements of the application, which helps to maximize both display and query performance.</span></span>

<span data-ttu-id="42ef3-178">Ha egy adott időpont aktuális adatai helyett az eseménystreamet használja írási tárolóként, azzal elkerülhetők az egyetlen összesítésen felmerülő frissítési ütközések, valamint maximalizálható a teljesítmény és a méretezhetőség.</span><span class="sxs-lookup"><span data-stu-id="42ef3-178">Using the stream of events as the write store, rather than the actual data at a point in time, avoids update conflicts on a single aggregate and maximizes performance and scalability.</span></span> <span data-ttu-id="42ef3-179">Az események az olvasási tároló feltöltéséhez használt adatok materializált nézetének aszinkron létrehozásához használhatók.</span><span class="sxs-lookup"><span data-stu-id="42ef3-179">The events can be used to asynchronously generate materialized views of the data that are used to populate the read store.</span></span>

<span data-ttu-id="42ef3-180">Mivel az eseménytároló az információk hivatalos forrása, a materializált nézetek törölhetők, és a rendszer fejlődésekor az összes múltbeli esemény visszajátszható a jelenlegi állapot új ábrázolásának létrehozása érdekében, illetve ha az olvasási modell módosítására van szükség.</span><span class="sxs-lookup"><span data-stu-id="42ef3-180">Because the event store is the official source of information, it is possible to delete the materialized views and replay all past events to create a new representation of the current state when the system evolves, or when the read model must change.</span></span> <span data-ttu-id="42ef3-181">A materializált nézetek lényegében az adatok tartós, csak olvasható gyorsítótáraként működnek.</span><span class="sxs-lookup"><span data-stu-id="42ef3-181">The materialized views are in effect a durable read-only cache of the data.</span></span>

<span data-ttu-id="42ef3-182">A CQRS és az Event Sourcing minta együttes használatakor vegye figyelembe az alábbiakat:</span><span class="sxs-lookup"><span data-stu-id="42ef3-182">When using CQRS combined with the Event Sourcing pattern, consider the following:</span></span>

- <span data-ttu-id="42ef3-183">Mint minden olyan rendszernél, ahol az írási és az olvasási adattárak elkülönülnek, az ezen mintán alapuló rendszerek esetében csak a végleges konzisztencia valósul meg.</span><span class="sxs-lookup"><span data-stu-id="42ef3-183">As with any system where the write and read stores are separate, systems based on this pattern are only eventually consistent.</span></span> <span data-ttu-id="42ef3-184">Az esemény létrehozása és az adattár frissítése között bizonyos fokú késésre lehet számítani.</span><span class="sxs-lookup"><span data-stu-id="42ef3-184">There will be some delay between the event being generated and the data store being updated.</span></span>

- <span data-ttu-id="42ef3-185">A minta összetetté teszi a rendszert, mivel kód létrehozására van szükség az események indításához és kezeléséhez, valamint a lekérdezésekhez vagy az olvasási modellhez szükséges megfelelő nézetek vagy objektumok összeállításához vagy frissítéséhez.</span><span class="sxs-lookup"><span data-stu-id="42ef3-185">The pattern adds complexity because code must be created to initiate and handle events, and assemble or update the appropriate views or objects required by queries or a read model.</span></span> <span data-ttu-id="42ef3-186">A CQRS minta az Event Sourcing mintával való együttes használatából eredő összetettség megnehezítheti a sikeres megvalósítást, és a rendszerek tervezése szempontjából is eltérő megközelítést igényel.</span><span class="sxs-lookup"><span data-stu-id="42ef3-186">The complexity of the CQRS pattern when used with the Event Sourcing pattern can make a successful implementation more difficult, and requires a different approach to designing systems.</span></span> <span data-ttu-id="42ef3-187">Az Event Sourcing azonban megkönnyítheti a tartomány modellezését, illetve a nézetek újraépítését vagy új nézetek létrehozását, mivel az adatok módosításainak célját megőrzi.</span><span class="sxs-lookup"><span data-stu-id="42ef3-187">However, event sourcing can make it easier to model the domain, and makes it easier to rebuild views or create new ones because the intent of the changes in the data is preserved.</span></span>

- <span data-ttu-id="42ef3-188">A materializált nézetek létrehozásához jelentős feldolgozási időre és erőforrás-használatra van szükség, ha azokat az adott entitások vagy entitásgyűjtemények eseményeinek visszajátszásával és kezelésével az olvasási modellben vagy az adatok leképezéseiben használja.</span><span class="sxs-lookup"><span data-stu-id="42ef3-188">Generating materialized views for use in the read model or projections of the data by replaying and handling the events for specific entities or collections of entities can require significant processing time and resource usage.</span></span> <span data-ttu-id="42ef3-189">Ez különösen akkor igaz, ha hosszú időszakok értékeinek összesítésére vagy elemzésére van szükség, mivel előfordulhat, hogy minden kapcsolódó eseményt meg kell vizsgálni.</span><span class="sxs-lookup"><span data-stu-id="42ef3-189">This is especially true if it requires summation or analysis of values over long periods, because all the associated events might need to be examined.</span></span> <span data-ttu-id="42ef3-190">Ezt megoldhatja az adatok üzemezett időközönként elkészített pillanatképeinek használatával, például egy adott művelet elvégzéseinek összesített számával vagy egy entitás aktuális állapotával.</span><span class="sxs-lookup"><span data-stu-id="42ef3-190">Resolve this by implementing snapshots of the data at scheduled intervals, such as a total count of the number of a specific action that have occurred, or the current state of an entity.</span></span>

## <a name="example"></a><span data-ttu-id="42ef3-191">Példa</span><span class="sxs-lookup"><span data-stu-id="42ef3-191">Example</span></span>

<span data-ttu-id="42ef3-192">Az alábbi kód részleteket mutat be egy CQRS-megvalósítási példából, amely külön definíciót használ az olvasási és írási modellekhez.</span><span class="sxs-lookup"><span data-stu-id="42ef3-192">The following code shows some extracts from an example of a CQRS implementation that uses different definitions for the read and the write models.</span></span> <span data-ttu-id="42ef3-193">A modellfelületek nem határozzák meg az alapul szolgáló adattárak jellemzőit, továbbá a felületek elkülönülése miatt a fejlődésük és finomhangolásuk is függetlenül valósulhat meg.</span><span class="sxs-lookup"><span data-stu-id="42ef3-193">The model interfaces don't dictate any features of the underlying data stores, and they can evolve and be fine-tuned independently because these interfaces are separated.</span></span>

<span data-ttu-id="42ef3-194">A következő kód bemutatja az olvasási modell definícióját.</span><span class="sxs-lookup"><span data-stu-id="42ef3-194">The following code shows the read model definition.</span></span>

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

<span data-ttu-id="42ef3-195">A rendszer lehetővé teszi a felhasználóknak számára a termékek értékelését.</span><span class="sxs-lookup"><span data-stu-id="42ef3-195">The system allows users to rate products.</span></span> <span data-ttu-id="42ef3-196">Az alkalmazás kódja ezt az alábbi kódban szereplő `RateProduct` parancs használatával éri el.</span><span class="sxs-lookup"><span data-stu-id="42ef3-196">The application code does this using the `RateProduct` command shown in the following code.</span></span>

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

<span data-ttu-id="42ef3-197">A rendszer a `ProductsCommandHandler` osztályt használja az alkalmazás által küldött parancsok kezelésére.</span><span class="sxs-lookup"><span data-stu-id="42ef3-197">The system uses the `ProductsCommandHandler` class to handle commands sent by the application.</span></span> <span data-ttu-id="42ef3-198">Az ügyfelek általában egy üzenetkezelő rendszeren, például egy üzenetsoron keresztül küldenek a parancsokat a tartománynak.</span><span class="sxs-lookup"><span data-stu-id="42ef3-198">Clients typically send commands to the domain through a messaging system such as a queue.</span></span> <span data-ttu-id="42ef3-199">A parancskezelő fogadja ezeket a parancsokat, és meghívja a tartományi felület metódusait.</span><span class="sxs-lookup"><span data-stu-id="42ef3-199">The command handler accepts these commands and invokes methods of the domain interface.</span></span> <span data-ttu-id="42ef3-200">Az egyes parancsok részletességének célja a kérések ütközési esélyeinek csökkentése.</span><span class="sxs-lookup"><span data-stu-id="42ef3-200">The granularity of each command is designed to reduce the chance of conflicting requests.</span></span> <span data-ttu-id="42ef3-201">A következő kód a `ProductsCommandHandler` osztály vázlatát mutatja be.</span><span class="sxs-lookup"><span data-stu-id="42ef3-201">The following code shows an outline of the `ProductsCommandHandler` class.</span></span>

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

<span data-ttu-id="42ef3-202">A következő kód az írási modell `IProductsDomain` felületét mutatja be.</span><span class="sxs-lookup"><span data-stu-id="42ef3-202">The following code shows the `IProductsDomain` interface from the write model.</span></span>

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

<span data-ttu-id="42ef3-203">Azt is figyelje meg, hogy az `IProductsDomain` felület olyan metódusokat tartalmaz, amelyek jelentéssel bírnak a tartományban.</span><span class="sxs-lookup"><span data-stu-id="42ef3-203">Also notice how the `IProductsDomain` interface contains methods that have a meaning in the domain.</span></span> <span data-ttu-id="42ef3-204">Egy CRUD-környezetben ezek a metódusok a legtöbb esetben általános névvel lennénk ellátva (például `Save` vagy `Update`), és az egyetlen argumentumuk egy DTO lenne.</span><span class="sxs-lookup"><span data-stu-id="42ef3-204">Typically, in a CRUD environment these methods would have generic names such as `Save` or `Update`, and have a DTO as the only argument.</span></span> <span data-ttu-id="42ef3-205">A CQRS-megközelítés kialakítható úgy, hogy megfeleljen a vállalat üzleti és leltárkezelő rendszerei igényeinek.</span><span class="sxs-lookup"><span data-stu-id="42ef3-205">The CQRS approach can be designed to meet the needs of this organization's business and inventory management systems.</span></span>

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="42ef3-206">Kapcsolódó minták és útmutatók</span><span class="sxs-lookup"><span data-stu-id="42ef3-206">Related patterns and guidance</span></span>

<span data-ttu-id="42ef3-207">Az alábbi minták és útmutatások hasznosak lehetnek a minta használatakor:</span><span class="sxs-lookup"><span data-stu-id="42ef3-207">The following patterns and guidance are useful when implementing this pattern:</span></span>

- <span data-ttu-id="42ef3-208">A CQRS és más architektúrastílusok összehasonlításáért lásd: [Architektúrastílusok](/azure/architecture/guide/architecture-styles/) és [CQRS architektúrastílus](/azure/architecture/guide/architecture-styles/cqrs).</span><span class="sxs-lookup"><span data-stu-id="42ef3-208">For a comparison of CQRS with other architectural styles, see [Architecture styles](/azure/architecture/guide/architecture-styles/) and [CQRS architecture style](/azure/architecture/guide/architecture-styles/cqrs).</span></span>

- <span data-ttu-id="42ef3-209">[Adatkonzisztencia – Ismertető](https://msdn.microsoft.com/library/dn589800.aspx).</span><span class="sxs-lookup"><span data-stu-id="42ef3-209">[Data Consistency Primer](https://msdn.microsoft.com/library/dn589800.aspx).</span></span> <span data-ttu-id="42ef3-210">Részletezi az olvasási és írási adattárak végleges konzisztenciája által okozott általánosan felmerülő problémákat a CQRS minta használatakor, illetve azt is, hogy ezek a problémák hogyan oldhatók meg.</span><span class="sxs-lookup"><span data-stu-id="42ef3-210">Explains the issues that are typically encountered due to eventual consistency between the read and write data stores when using the CQRS pattern, and how these issues can be resolved.</span></span>

- <span data-ttu-id="42ef3-211">[Adatparticionálási útmutató](https://msdn.microsoft.com/library/dn589795.aspx).</span><span class="sxs-lookup"><span data-stu-id="42ef3-211">[Data Partitioning Guidance](https://msdn.microsoft.com/library/dn589795.aspx).</span></span> <span data-ttu-id="42ef3-212">Ismerteti, hogyan oszthatók fel a CQRS mintában használt olvasási és írási adattárak olyan partíciókra, amelyek a méretezhetőség, a versenyhelyzetek kialakulásának csökkentése és a teljesítmény optimalizálása érdekében külön kezelhetők és érhetők el.</span><span class="sxs-lookup"><span data-stu-id="42ef3-212">Describes how the read and write data stores used in the CQRS pattern can be divided into partitions that can be managed and accessed separately to improve scalability, reduce contention, and optimize performance.</span></span>

- <span data-ttu-id="42ef3-213">[Event Sourcing minta](event-sourcing.md).</span><span class="sxs-lookup"><span data-stu-id="42ef3-213">[Event Sourcing Pattern](event-sourcing.md).</span></span> <span data-ttu-id="42ef3-214">Részletesebben ismerteti, hogyan használható az Event Sourcing a CQRS mintával a feladatok egyszerűsítéséhez az összetett tartományokban a teljesítmény, a méretezhetőség és a válaszkészség növelése mellett.</span><span class="sxs-lookup"><span data-stu-id="42ef3-214">Describes in more detail how Event Sourcing can be used with the CQRS pattern to simplify tasks in complex domains while improving performance, scalability, and responsiveness.</span></span> <span data-ttu-id="42ef3-215">Emellett azt is bemutatja, hogyan biztosítható a tranzakciós adatok konzisztenciája a teljes körű naplók és előzmények fenntartása mellett, ami lehetővé teszi a kompenzáló műveletek végrehajtását.</span><span class="sxs-lookup"><span data-stu-id="42ef3-215">As well as how to provide consistency for transactional data while maintaining full audit trails and history that can enable compensating actions.</span></span>

- <span data-ttu-id="42ef3-216">[A Materialized View minta](materialized-view.md).</span><span class="sxs-lookup"><span data-stu-id="42ef3-216">[Materialized View Pattern](materialized-view.md).</span></span> <span data-ttu-id="42ef3-217">A CQRS megvalósítás olvasási modellje tartalmazhatja az írási modell adatainak materializált nézeteit, illetve a materializált nézetek létrehozására is használható.</span><span class="sxs-lookup"><span data-stu-id="42ef3-217">The read model of a CQRS implementation can contain materialized views of the write model data, or the read model can be used to generate materialized views.</span></span>

- <span data-ttu-id="42ef3-218">Minták és gyakorlatok útmutatója – [A CQRS felfedezése](https://aka.ms/cqrs).</span><span class="sxs-lookup"><span data-stu-id="42ef3-218">The patterns & practices guide [CQRS Journey](https://aka.ms/cqrs).</span></span> <span data-ttu-id="42ef3-219">Különösen [bemutatása a parancs Query Responsibility Segregation minta](https://msdn.microsoft.com/library/jj591573.aspx) bemutatja a mintát mikor hasznos lehet, és [Epilógus: Learned tanítás](https://msdn.microsoft.com/library/jj591568.aspx) segít megérteni, hogy ez a minta használata során problémák.</span><span class="sxs-lookup"><span data-stu-id="42ef3-219">In particular, [Introducing the Command Query Responsibility Segregation Pattern](https://msdn.microsoft.com/library/jj591573.aspx) explores the pattern and when it's useful, and [Epilogue: Lessons Learned](https://msdn.microsoft.com/library/jj591568.aspx) helps you understand some of the issues that come up when using this pattern.</span></span>

- <span data-ttu-id="42ef3-220">A [Martin Fowler – CQRS](https://martinfowler.com/bliki/CQRS.html) című bejegyzés ismerteti a minta alapvető működését, továbbá egyéb hasznos források hivatkozásait is tartalmazza.</span><span class="sxs-lookup"><span data-stu-id="42ef3-220">The post [CQRS by Martin Fowler](https://martinfowler.com/bliki/CQRS.html), which explains the basics of the pattern and links to other useful resources.</span></span>
