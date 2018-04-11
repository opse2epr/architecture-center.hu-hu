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
ms.openlocfilehash: ce8d20ae82ae7d5ba00b4bc264a5c4d90fc383bd
ms.sourcegitcommit: ea7108f71dab09175ff69322874d1bcba800a37a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/17/2018
---
# <a name="command-and-query-responsibility-segregation-cqrs-pattern"></a><span data-ttu-id="6d515-104">Command and Query Responsibility Segregation (CQRS) minta</span><span class="sxs-lookup"><span data-stu-id="6d515-104">Command and Query Responsibility Segregation (CQRS) pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="6d515-105">Különböző felületek használatával elkülönítheti az adatolvasó műveleteket az adatfrissítő műveletektől.</span><span class="sxs-lookup"><span data-stu-id="6d515-105">Segregate operations that read data from operations that update data by using separate interfaces.</span></span> <span data-ttu-id="6d515-106">Ezzel maximalizálhatja a teljesítményt, a méretezhetőséget és a biztonságot.</span><span class="sxs-lookup"><span data-stu-id="6d515-106">This can maximize performance, scalability, and security.</span></span> <span data-ttu-id="6d515-107">A nagyfokú rugalmassága révén támogatja a rendszer fejlődését, és megakadályozza, hogy a frissítési parancsok egyesítési ütközéseket okozzanak a tartományi szinten.</span><span class="sxs-lookup"><span data-stu-id="6d515-107">Supports the evolution of the system over time through higher flexibility, and prevent update commands from causing merge conflicts at the domain level.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="6d515-108">Kontextus és probléma</span><span class="sxs-lookup"><span data-stu-id="6d515-108">Context and problem</span></span>

<span data-ttu-id="6d515-109">A hagyományos adatkezelő rendszerek esetében mind a parancsokat (az adatok frissítéseit), mind a lekérdezéseket (az adatokra irányuló kéréseket) ugyanazon az egyetlen adattárban található entitáskészleten hajtja végre a rendszer.</span><span class="sxs-lookup"><span data-stu-id="6d515-109">In traditional data management systems, both commands (updates to the data) and queries (requests for data) are executed against the same set of entities in a single data repository.</span></span> <span data-ttu-id="6d515-110">Ezek az entitások lehetnek egy az SQL Serverhez hasonló relációs adatbázisban található egy vagy több tábla soraiból álló halmazok.</span><span class="sxs-lookup"><span data-stu-id="6d515-110">These entities can be a subset of the rows in one or more tables in a relational database such as SQL Server.</span></span>

<span data-ttu-id="6d515-111">Ezekben a rendszerekben általában minden létrehozási, olvasási, frissítési és törlési (CRUD) művelet az entitás ugyanazon formáján mennek végbe.</span><span class="sxs-lookup"><span data-stu-id="6d515-111">Typically in these systems, all create, read, update, and delete (CRUD) operations are applied to the same representation of the entity.</span></span> <span data-ttu-id="6d515-112">Például az adat-hozzáférési réteg (DAL) beolvas egy ügyfelet képviselő adatátviteli objektumot (DTO-t) az adattárból, és az megjelenik a képernyőn.</span><span class="sxs-lookup"><span data-stu-id="6d515-112">For example, a data transfer object (DTO) representing a customer is retrieved from the data store by the data access layer (DAL) and displayed on the screen.</span></span> <span data-ttu-id="6d515-113">A felhasználó frissíti a DTO egyes mezőit (például adatkötéssel), majd a DAL ismételten menti DTO-t az adattárban.</span><span class="sxs-lookup"><span data-stu-id="6d515-113">A user updates some fields of the DTO (perhaps through data binding) and the DTO is then saved back in the data store by the DAL.</span></span> <span data-ttu-id="6d515-114">A rendszer az olvasási és írási műveletetekhez is ugyanazt a DTO-t használja.</span><span class="sxs-lookup"><span data-stu-id="6d515-114">The same DTO is used for both the read and write operations.</span></span> <span data-ttu-id="6d515-115">Az ábrán egy hagyományos CRUD-architektúra látható.</span><span class="sxs-lookup"><span data-stu-id="6d515-115">The figure illustrates a traditional CRUD architecture.</span></span>

![Hagyományos CRUD-architektúra](./_images/command-and-query-responsibility-segregation-cqrs-tradition-crud.png)

<span data-ttu-id="6d515-117">A hagyományos CRUD-kialakítások akkor működnek jól, ha csak korlátozott üzleti logikát alkalmaz az adatműveletekre.</span><span class="sxs-lookup"><span data-stu-id="6d515-117">Traditional CRUD designs work well when only limited business logic is applied to the data operations.</span></span> <span data-ttu-id="6d515-118">A fejlesztői eszközök által biztosított felépítő mechanizmus nagyon gyorsan képes az adat-hozzáférési kód létrehozására, amely aztán szükség szerint testreszabható.</span><span class="sxs-lookup"><span data-stu-id="6d515-118">Scaffold mechanisms provided by development tools can create data access code very quickly, which can then be customized as required.</span></span>

<span data-ttu-id="6d515-119">A hagyományos CRUD megközelítésnek azonban néhány hátránya is van:</span><span class="sxs-lookup"><span data-stu-id="6d515-119">However, the traditional CRUD approach has some disadvantages:</span></span>

- <span data-ttu-id="6d515-120">Ez gyakran azt jelenti, hogy az adatok írási és olvasási ábrázolása nem egyezik, például az egyik több oszloppal vagy tulajdonsággal rendelkezik, amelyeket akkor is megfelelően kell frissíteni, ha nem szükségesek egy adott művelethez.</span><span class="sxs-lookup"><span data-stu-id="6d515-120">It often means that there's a mismatch between the read and write representations of the data, such as additional columns or properties that must be updated correctly even though they aren't required as part of an operation.</span></span>

- <span data-ttu-id="6d515-121">Fennáll az adatok közti verseny kialakulásának veszélye, ha a rekordok egy olyan együttműködési tartomány adattárában vannak zárolva, amelyben ugyanazon az adatkészleten párhuzamosan több aktor is végez műveleteket.</span><span class="sxs-lookup"><span data-stu-id="6d515-121">It risks data contention when records are locked in the data store in a collaborative domain, where multiple actors operate in parallel on the same set of data.</span></span> <span data-ttu-id="6d515-122">Illetve ha optimista zárolás használatakor az egyidejű frissítések ütköznek egymással.</span><span class="sxs-lookup"><span data-stu-id="6d515-122">Or update conflicts caused by concurrent updates when optimistic locking is used.</span></span> <span data-ttu-id="6d515-123">Az említett kockázatok esélye a rendszer összetettségének és teljesítményének növekedésével egyre nagyobb lesz.</span><span class="sxs-lookup"><span data-stu-id="6d515-123">These risks increase as the complexity and throughput of the system grows.</span></span> <span data-ttu-id="6d515-124">Emellett a hagyományos megközelítés negatív hatással lehet a teljesítményre az adattárra és az adatelérési rétegre eső terhelés, valamint a lekérdezések az adatok beolvasásához szükséges bonyolultsága miatt.</span><span class="sxs-lookup"><span data-stu-id="6d515-124">In addition, the traditional approach can have a negative effect on performance due to load on the data store and data access layer, and the complexity of queries required to retrieve information.</span></span>

- <span data-ttu-id="6d515-125">Ez bonyolultabbá teheti a biztonság és az engedélyek kezelését, mivel minden entitáson végrehajthatók olvasási és írási műveletek is, ezáltal az adatok nem megfelelő kontextusban is megjelenhetnek.</span><span class="sxs-lookup"><span data-stu-id="6d515-125">It can make managing security and permissions more complex because each entity is subject to both read and write operations, which might expose data in the wrong context.</span></span>

> <span data-ttu-id="6d515-126">A CRUD megközelítés korlátairól részletesebben a [CRUD – csak akkor, ha megengedheti](https://blogs.msdn.microsoft.com/maarten_mullender/2004/07/23/crud-only-when-you-can-afford-it-revisited/) című cikkben olvashat.</span><span class="sxs-lookup"><span data-stu-id="6d515-126">For a deeper understanding of the limits of the CRUD approach see [CRUD, Only When You Can Afford It](https://blogs.msdn.microsoft.com/maarten_mullender/2004/07/23/crud-only-when-you-can-afford-it-revisited/).</span></span>

## <a name="solution"></a><span data-ttu-id="6d515-127">Megoldás</span><span class="sxs-lookup"><span data-stu-id="6d515-127">Solution</span></span>

<span data-ttu-id="6d515-128">A Command and Query Responsibility Segregation (CQRS) olyan minta, amely külön felületek használatával elválasztja az adatolvasási műveleteket (lekérdezéseket) az adatfrissítési műveletektől (parancsoktól).</span><span class="sxs-lookup"><span data-stu-id="6d515-128">Command and Query Responsibility Segregation (CQRS) is a pattern that segregates the operations that read data (queries) from the operations that update data (commands) by using separate interfaces.</span></span> <span data-ttu-id="6d515-129">Ez azt jelenti, hogy a lekérdezésekhez és a frissítésekhez nem ugyanazt az adatmodellt használja.</span><span class="sxs-lookup"><span data-stu-id="6d515-129">This means that the data models used for querying and updates are different.</span></span> <span data-ttu-id="6d515-130">A modellek az ábrán látható módon elkülöníthetők, bár ez nem alapvető követelmény.</span><span class="sxs-lookup"><span data-stu-id="6d515-130">The models can then be isolated, as shown in the following figure, although that's not an absolute requirement.</span></span>

![Alapszintű CQRS-architektúra](./_images/command-and-query-responsibility-segregation-cqrs-basic.png)

<span data-ttu-id="6d515-132">A CRUD-alapú rendszerekben használt egyetlen adatmodellhez képest a CQRS-alapú rendszerekben a külön lekérdezési és frissítési modellek használata egyszerűbbé teszi a tervezést és a megvalósítást.</span><span class="sxs-lookup"><span data-stu-id="6d515-132">Compared to the single data model used in CRUD-based systems, the use of separate query and update models for the data in CQRS-based systems simplifies design and implementation.</span></span> <span data-ttu-id="6d515-133">Azonban ennek egyik hátránya, hogy a CRUD-kialakításoktól eltérően a CQRS-kód nem hozható létre automatikusan felépítő mechanizmusok használatával.</span><span class="sxs-lookup"><span data-stu-id="6d515-133">However, one disadvantage is that unlike CRUD designs, CQRS code can't automatically be generated using scaffold mechanisms.</span></span>

<span data-ttu-id="6d515-134">Az olvasási adatok lekérdezési modellje és az írási adatok frissítési modellje ugyanahhoz a fizikai tárolóhoz fér hozzá, például az SQL-nézetek vagy menet közben létrehozott leképezések használatával.</span><span class="sxs-lookup"><span data-stu-id="6d515-134">The query model for reading data and the update model for writing data can access the same physical store, perhaps by using SQL views or by generating projections on the fly.</span></span> <span data-ttu-id="6d515-135">Gyakori eljárás azonban az adatok külön fizikai tárolóban való elkülönítése a teljesítmény, a méretezhetőség és a biztonság maximalizálása érdekében, ahogy azt a következő ábra is mutatja.</span><span class="sxs-lookup"><span data-stu-id="6d515-135">However, it's common to separate the data into different physical stores to maximize performance, scalability, and security, as shown in the next figure.</span></span>

![CQRS-architektúra külön olvasási és írási tárolókkal](./_images/command-and-query-responsibility-segregation-cqrs-separate-stores.png)

<span data-ttu-id="6d515-137">Az olvasási tároló lehet az írási tároló csak olvasható replikája, de az olvasási és írási tárolók lehetnek teljesen eltérő szerkezetűek is.</span><span class="sxs-lookup"><span data-stu-id="6d515-137">The read store can be a read-only replica of the write store, or the read and write stores can have a different structure altogether.</span></span> <span data-ttu-id="6d515-138">Az olvasási tároló több, csak olvasható replikájának használatával jelentősen növelhető a lekérdezések teljesítménye és az alkalmazás felhasználói felületének válaszideje, különösen az elosztott forgatókönyvekben, amelyekben a csak olvasható replikák az alkalmazáspéldányokhoz közel találhatók.</span><span class="sxs-lookup"><span data-stu-id="6d515-138">Using multiple read-only replicas of the read store can greatly increase query performance and application UI responsiveness, especially in distributed scenarios where read-only replicas are located close to the application instances.</span></span> <span data-ttu-id="6d515-139">Egyes adatbázis-rendszerek (SQL Server) további funkciókat kínálnak, például a rendelkezésre állás maximalizálását szolgáló feladatátvételi replikákat.</span><span class="sxs-lookup"><span data-stu-id="6d515-139">Some database systems (SQL Server) provide additional features such as failover replicas to maximize availability.</span></span>

<span data-ttu-id="6d515-140">Az olvasási és írási tárolók különválasztása lehetővé teszi a terhelésnek megfelelő méretezésüket is.</span><span class="sxs-lookup"><span data-stu-id="6d515-140">Separation of the read and write stores also allows each to be scaled appropriately to match the load.</span></span> <span data-ttu-id="6d515-141">Az olvasási tárolók például jellemzően jóval nagyobb terhelésnek vannak kitéve, mint az írási tárolók.</span><span class="sxs-lookup"><span data-stu-id="6d515-141">For example, read stores typically encounter a much higher load than write stores.</span></span>

<span data-ttu-id="6d515-142">Ha a lekérdezési/olvasási modell denormalizált adatokat tartalmaz (lásd: [Materialized View minta](materialized-view.md)), akkor a teljesítmény az alkalmazásban szereplő egyes nézetek adatainak beolvasásakor vagy a rendszerbeli adatok lekérdezésekor maximalizálható.</span><span class="sxs-lookup"><span data-stu-id="6d515-142">When the query/read model contains denormalized data (see [Materialized View pattern](materialized-view.md)), performance is maximized when reading data for each of the views in an application or when querying the data in the system.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="6d515-143">Problémák és megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="6d515-143">Issues and considerations</span></span>

<span data-ttu-id="6d515-144">A minta megvalósítása során az alábbi pontokat vegye figyelembe:</span><span class="sxs-lookup"><span data-stu-id="6d515-144">Consider the following points when deciding how to implement this pattern:</span></span>

- <span data-ttu-id="6d515-145">Az adattárak az olvasási és írási műveletek számára külön fizikai tárolókban való elosztásával növelhető a rendszer teljesítménye és biztonsága, azonban ez a rugalmasság és a végleges konzisztencia szempontjából bonyolíthatja a helyzetet.</span><span class="sxs-lookup"><span data-stu-id="6d515-145">Dividing the data store into separate physical stores for read and write operations can increase the performance and security of a system, but it can add complexity in terms of resiliency and eventual consistency.</span></span> <span data-ttu-id="6d515-146">Az olvasási modell tárolóját frissíteni kell ahhoz, hogy megjelenjenek benne az írási modellen végzett módosítások, így nehéz lehet megállapítani, ha egy felhasználó elavult olvasási adatokon alapuló kérelmet bocsát ki, ami azt jelenti, hogy a művelet nem végezhető el.</span><span class="sxs-lookup"><span data-stu-id="6d515-146">The read model store must be updated to reflect changes to the write model store, and it can be difficult to detect when a user has issued a request based on stale read data, which means that the operation can't be completed.</span></span>

    > <span data-ttu-id="6d515-147">A végleges konzisztencia leírása az [Adatkonzisztencia – Ismertető](https://msdn.microsoft.com/library/dn589800.aspx) című cikkben található.</span><span class="sxs-lookup"><span data-stu-id="6d515-147">For a description of eventual consistency see the [Data Consistency Primer](https://msdn.microsoft.com/library/dn589800.aspx).</span></span>

- <span data-ttu-id="6d515-148">A CQRS-t érdemes lehet a rendszernek csak az olyan korlátozott részein használni, ahol a leginkább hasznos.</span><span class="sxs-lookup"><span data-stu-id="6d515-148">Consider applying CQRS to limited sections of your system where it will be most valuable.</span></span>

- <span data-ttu-id="6d515-149">A végleges konzisztencia kialakításának általános megközelítése az Event Sourcing és a CQRS együttes használata. Ily módon az írási modell egy hozzáfűzéssel bővíthető eseménystream, amelyet a parancsok végrehajtása vezérel.</span><span class="sxs-lookup"><span data-stu-id="6d515-149">A typical approach to deploying eventual consistency is to use event sourcing in conjunction with CQRS so that the write model is an append-only stream of events driven by execution of commands.</span></span> <span data-ttu-id="6d515-150">Ezek az események az olvasási modellként működő materializált nézetek frissítéséhez használatosak.</span><span class="sxs-lookup"><span data-stu-id="6d515-150">These events are used to update materialized views that act as the read model.</span></span> <span data-ttu-id="6d515-151">Bővebb információ: [Event Sourcing és CQRS](/azure/architecture/patterns/cqrs#event-sourcing-and-cqrs).</span><span class="sxs-lookup"><span data-stu-id="6d515-151">For more information see [Event Sourcing and CQRS](/azure/architecture/patterns/cqrs#event-sourcing-and-cqrs).</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="6d515-152">Mikor érdemes ezt a mintát használni?</span><span class="sxs-lookup"><span data-stu-id="6d515-152">When to use this pattern</span></span>

<span data-ttu-id="6d515-153">Az alábbi helyzetekben alkalmazza ezt a mintát:</span><span class="sxs-lookup"><span data-stu-id="6d515-153">Use this pattern in the following situations:</span></span>

- <span data-ttu-id="6d515-154">Együttműködési tartományok, ahol ugyanazokon az adatokon egyszerre több műveletet is végrehajt a rendszer.</span><span class="sxs-lookup"><span data-stu-id="6d515-154">Collaborative domains where multiple operations are performed in parallel on the same data.</span></span> <span data-ttu-id="6d515-155">A CQRS révén megfelelő részletességgel adhatók meg a parancsok a tartományi szint egyesítési ütközéseinek minimalizálása érdekében (bármely fellépő ütközés egyesíthető a parancs által), még akkor is, ha látszólag ugyanazon adattípus frissítéséről van szó.</span><span class="sxs-lookup"><span data-stu-id="6d515-155">CQRS allows you to define commands with enough granularity to minimize merge conflicts at the domain level (any conflicts that do arise can be merged by the command), even when updating what appears to be the same type of data.</span></span>

- <span data-ttu-id="6d515-156">Feladatalapú felhasználói felületek, ahol a felhasználók lépésenként haladnak végig egy összetett folyamaton, vagy összetett tartományi modellekkel.</span><span class="sxs-lookup"><span data-stu-id="6d515-156">Task-based user interfaces where users are guided through a complex process as a series of steps or with complex domain models.</span></span> <span data-ttu-id="6d515-157">Ezenkívül olyan csapatok számára is hasznos, akik már ismerik a tartományvezérelt tervezési (DDD) technikákat.</span><span class="sxs-lookup"><span data-stu-id="6d515-157">Also, useful for teams already familiar with domain-driven design (DDD) techniques.</span></span> <span data-ttu-id="6d515-158">Az írási modell egy üzleti logikát, bemenet-ellenőrzést és üzleti ellenőrzést tartalmazó teljes parancsfeldolgozási veremmel rendelkezik annak biztosítása érdekében, hogy mindig minden konzisztens legyen az írási modell minden egyes összesítésével (a társított objektumok minden egyes fürtjét az adatok változásának egységeként kezeli).</span><span class="sxs-lookup"><span data-stu-id="6d515-158">The write model has a full command-processing stack with business logic, input validation, and business validation to ensure that everything is always consistent for each of the aggregates (each cluster of associated objects treated as a unit for data changes) in the write model.</span></span> <span data-ttu-id="6d515-159">Az olvasási modell nem tartalmaz üzleti logikát és érvényesítési vermet, csupán egy DTO-t ad vissza a nézetmodellben való használatra.</span><span class="sxs-lookup"><span data-stu-id="6d515-159">The read model has no business logic or validation stack and just returns a DTO for use in a view model.</span></span> <span data-ttu-id="6d515-160">Az olvasási modell véglegesen konzisztens az írási modellel.</span><span class="sxs-lookup"><span data-stu-id="6d515-160">The read model is eventually consistent with the write model.</span></span>

- <span data-ttu-id="6d515-161">Olyan forgatókönyvek, amelyekben az adatolvasási teljesítményt az adatírási teljesítménytől külön kell finomhangolni, különösen, ha az olvasási/írási arány nagyon magas, illetve ha horizontális skálázásra van szükség.</span><span class="sxs-lookup"><span data-stu-id="6d515-161">Scenarios where performance of data reads must be fine tuned separately from performance of data writes, especially when the read/write ratio is very high, and when horizontal scaling is required.</span></span> <span data-ttu-id="6d515-162">Az olvasási műveletek száma például számos rendszerben sokszorta több az írási műveletek számánál.</span><span class="sxs-lookup"><span data-stu-id="6d515-162">For example, in many systems the number of read operations is many times greater than the number of write operations.</span></span> <span data-ttu-id="6d515-163">Ezen igény kiszolgálása érdekében fontolja meg az olvasási modell kiterjesztését, az írási modellt viszont egyetlen, vagy mindössze néhány példányon futtassa.</span><span class="sxs-lookup"><span data-stu-id="6d515-163">To accommodate this, consider scaling out the read model, but running the write model on only one or a few instances.</span></span> <span data-ttu-id="6d515-164">Alacsony számú írási modell-példány használatával csökkenthető az egyesítési ütközések előfordulása is.</span><span class="sxs-lookup"><span data-stu-id="6d515-164">A small number of write model instances also helps to minimize the occurrence of merge conflicts.</span></span>

- <span data-ttu-id="6d515-165">Olyan forgatókönyvek, amelyekben egy fejlesztői csapat az írási modell részét képező összetett tartományi modellre összpontosíthat, egy másik csapat pedig az olvasási modellre és a felhasználói felületekre.</span><span class="sxs-lookup"><span data-stu-id="6d515-165">Scenarios where one team of developers can focus on the complex domain model that is part of the write model, and another team can focus on the read model and the user interfaces.</span></span>

- <span data-ttu-id="6d515-166">Olyan forgatókönyvek, amelyekben a rendszer az idő előrehaladtával várhatóan fejlődni fog, így a modell több verzióját is tartalmazhatja, illetve ahol az üzleti szabályok rendszeresen változnak.</span><span class="sxs-lookup"><span data-stu-id="6d515-166">Scenarios where the system is expected to evolve over time and might contain multiple versions of the model, or where business rules change regularly.</span></span>

- <span data-ttu-id="6d515-167">Integráció más rendszerekkel, különösen az Event Sourcinggal kombinálva, amely esetben egy alrendszer átmeneti meghibásodása nem lehet hatással a többi alrendszer rendelkezésre állására.</span><span class="sxs-lookup"><span data-stu-id="6d515-167">Integration with other systems, especially in combination with event sourcing, where the temporal failure of one subsystem shouldn't affect the availability of the others.</span></span>

<span data-ttu-id="6d515-168">A minta használata a következő esetekben nem ajánlott:</span><span class="sxs-lookup"><span data-stu-id="6d515-168">This pattern isn't recommended in the following situations:</span></span>

- <span data-ttu-id="6d515-169">Ahol a tartomány vagy az üzleti szabályok egyszerűek.</span><span class="sxs-lookup"><span data-stu-id="6d515-169">Where the domain or the business rules are simple.</span></span>

- <span data-ttu-id="6d515-170">Ahol egy egyszerű CRUD stílusú felhasználói felület és az ahhoz kapcsolódó adat-hozzáférési műveletek is elegendőek.</span><span class="sxs-lookup"><span data-stu-id="6d515-170">Where a simple CRUD-style user interface and the related data access operations are sufficient.</span></span>

- <span data-ttu-id="6d515-171">A teljes rendszeren történő megvalósítás esetén.</span><span class="sxs-lookup"><span data-stu-id="6d515-171">For implementation across the whole system.</span></span> <span data-ttu-id="6d515-172">Egy általános adatkezelési forgatókönyvnek vannak olyan adott összetevői, amelyek esetében a CQRS hasznos lehet, viszont számottevő és szükségtelen összetettséget jelenthet, ha nincs rá igazán szükség.</span><span class="sxs-lookup"><span data-stu-id="6d515-172">There are specific components of an overall data management scenario where CQRS can be useful, but it can add considerable and unnecessary complexity when it isn't required.</span></span>

## <a name="event-sourcing-and-cqrs"></a><span data-ttu-id="6d515-173">Az Event Sourcing és a CQRS</span><span class="sxs-lookup"><span data-stu-id="6d515-173">Event Sourcing and CQRS</span></span>

<span data-ttu-id="6d515-174">A CQRS mintát gyakran használják az Event Sourcing mintával együtt.</span><span class="sxs-lookup"><span data-stu-id="6d515-174">The CQRS pattern is often used along with the Event Sourcing pattern.</span></span> <span data-ttu-id="6d515-175">A CQRS-alapú rendszerek külön olvasási és írási adatmodelleket használnak, amelyek mindegyike a kapcsolódó feladatokhoz van igazítva, és gyakran fizikailag elkülönített tárolóban találhatók.</span><span class="sxs-lookup"><span data-stu-id="6d515-175">CQRS-based systems use separate read and write data models, each tailored to relevant tasks and often located in physically separate stores.</span></span> <span data-ttu-id="6d515-176">Az [Event Sourcing](event-sourcing.md) minta használata esetén az események tárolója és az információk hivatalos forrása az írási modell.</span><span class="sxs-lookup"><span data-stu-id="6d515-176">When used with the [Event Sourcing](event-sourcing.md) pattern, the store of events is the write model, and is the official source of information.</span></span> <span data-ttu-id="6d515-177">A CQRS-alapú rendszerek olvasási modellje az adatok materializált nézetit biztosítja, általában nagy mértékben denormalizált nézetekként.</span><span class="sxs-lookup"><span data-stu-id="6d515-177">The read model of a CQRS-based system provides materialized views of the data, typically as highly denormalized views.</span></span> <span data-ttu-id="6d515-178">Ezek a nézetek az alkalmazás felhasználói felületeihez és megjelenítési követelményeihez vannak igazítva, ezzel pedig maximalizálható a megjelenítési és lekérdezési teljesítmény.</span><span class="sxs-lookup"><span data-stu-id="6d515-178">These views are tailored to the interfaces and display requirements of the application, which helps to maximize both display and query performance.</span></span>

<span data-ttu-id="6d515-179">Ha egy adott időpont aktuális adatai helyett az eseménystreamet használja írási tárolóként, azzal elkerülhetők az egyetlen összesítésen felmerülő frissítési ütközések, valamint maximalizálható a teljesítmény és a méretezhetőség.</span><span class="sxs-lookup"><span data-stu-id="6d515-179">Using the stream of events as the write store, rather than the actual data at a point in time, avoids update conflicts on a single aggregate and maximizes performance and scalability.</span></span> <span data-ttu-id="6d515-180">Az események az olvasási tároló feltöltéséhez használt adatok materializált nézetének aszinkron létrehozásához használhatók.</span><span class="sxs-lookup"><span data-stu-id="6d515-180">The events can be used to asynchronously generate materialized views of the data that are used to populate the read store.</span></span>

<span data-ttu-id="6d515-181">Mivel az eseménytároló az információk hivatalos forrása, a materializált nézetek törölhetők, és a rendszer fejlődésekor az összes múltbeli esemény visszajátszható a jelenlegi állapot új ábrázolásának létrehozása érdekében, illetve ha az olvasási modell módosítására van szükség.</span><span class="sxs-lookup"><span data-stu-id="6d515-181">Because the event store is the official source of information, it is possible to delete the materialized views and replay all past events to create a new representation of the current state when the system evolves, or when the read model must change.</span></span> <span data-ttu-id="6d515-182">A materializált nézetek lényegében az adatok tartós, csak olvasható gyorsítótáraként működnek.</span><span class="sxs-lookup"><span data-stu-id="6d515-182">The materialized views are in effect a durable read-only cache of the data.</span></span>

<span data-ttu-id="6d515-183">A CQRS és az Event Sourcing minta együttes használatakor vegye figyelembe az alábbiakat:</span><span class="sxs-lookup"><span data-stu-id="6d515-183">When using CQRS combined with the Event Sourcing pattern, consider the following:</span></span>

- <span data-ttu-id="6d515-184">Mint minden olyan rendszernél, ahol az írási és az olvasási adattárak elkülönülnek, az ezen mintán alapuló rendszerek esetében csak a végleges konzisztencia valósul meg.</span><span class="sxs-lookup"><span data-stu-id="6d515-184">As with any system where the write and read stores are separate, systems based on this pattern are only eventually consistent.</span></span> <span data-ttu-id="6d515-185">Az esemény létrehozása és az adattár frissítése között bizonyos fokú késésre lehet számítani.</span><span class="sxs-lookup"><span data-stu-id="6d515-185">There will be some delay between the event being generated and the data store being updated.</span></span>

- <span data-ttu-id="6d515-186">A minta összetetté teszi a rendszert, mivel kód létrehozására van szükség az események indításához és kezeléséhez, valamint a lekérdezésekhez vagy az olvasási modellhez szükséges megfelelő nézetek vagy objektumok összeállításához vagy frissítéséhez.</span><span class="sxs-lookup"><span data-stu-id="6d515-186">The pattern adds complexity because code must be created to initiate and handle events, and assemble or update the appropriate views or objects required by queries or a read model.</span></span> <span data-ttu-id="6d515-187">A CQRS minta az Event Sourcing mintával való együttes használatából eredő összetettség megnehezítheti a sikeres megvalósítást, és a rendszerek tervezése szempontjából is eltérő megközelítést igényel.</span><span class="sxs-lookup"><span data-stu-id="6d515-187">The complexity of the CQRS pattern when used with the Event Sourcing pattern can make a successful implementation more difficult, and requires a different approach to designing systems.</span></span> <span data-ttu-id="6d515-188">Az Event Sourcing azonban megkönnyítheti a tartomány modellezését, illetve a nézetek újraépítését vagy új nézetek létrehozását, mivel az adatok módosításainak célját megőrzi.</span><span class="sxs-lookup"><span data-stu-id="6d515-188">However, event sourcing can make it easier to model the domain, and makes it easier to rebuild views or create new ones because the intent of the changes in the data is preserved.</span></span>

- <span data-ttu-id="6d515-189">A materializált nézetek létrehozásához jelentős feldolgozási időre és erőforrás-használatra van szükség, ha azokat az adott entitások vagy entitásgyűjtemények eseményeinek visszajátszásával és kezelésével az olvasási modellben vagy az adatok leképezéseiben használja.</span><span class="sxs-lookup"><span data-stu-id="6d515-189">Generating materialized views for use in the read model or projections of the data by replaying and handling the events for specific entities or collections of entities can require significant processing time and resource usage.</span></span> <span data-ttu-id="6d515-190">Ez különösen akkor igaz, ha hosszú időszakok értékeinek összesítésére vagy elemzésére van szükség, mivel előfordulhat, hogy minden kapcsolódó eseményt meg kell vizsgálni.</span><span class="sxs-lookup"><span data-stu-id="6d515-190">This is especially true if it requires summation or analysis of values over long periods, because all the associated events might need to be examined.</span></span> <span data-ttu-id="6d515-191">Ezt megoldhatja az adatok üzemezett időközönként elkészített pillanatképeinek használatával, például egy adott művelet elvégzéseinek összesített számával vagy egy entitás aktuális állapotával.</span><span class="sxs-lookup"><span data-stu-id="6d515-191">Resolve this by implementing snapshots of the data at scheduled intervals, such as a total count of the number of a specific action that have occurred, or the current state of an entity.</span></span>

## <a name="example"></a><span data-ttu-id="6d515-192">Példa</span><span class="sxs-lookup"><span data-stu-id="6d515-192">Example</span></span>

<span data-ttu-id="6d515-193">Az alábbi kód részleteket mutat be egy CQRS-megvalósítási példából, amely külön definíciót használ az olvasási és írási modellekhez.</span><span class="sxs-lookup"><span data-stu-id="6d515-193">The following code shows some extracts from an example of a CQRS implementation that uses different definitions for the read and the write models.</span></span> <span data-ttu-id="6d515-194">A modellfelületek nem határozzák meg az alapul szolgáló adattárak jellemzőit, továbbá a felületek elkülönülése miatt a fejlődésük és finomhangolásuk is függetlenül valósulhat meg.</span><span class="sxs-lookup"><span data-stu-id="6d515-194">The model interfaces don't dictate any features of the underlying data stores, and they can evolve and be fine-tuned independently because these interfaces are separated.</span></span>

<span data-ttu-id="6d515-195">A következő kód bemutatja az olvasási modell definícióját.</span><span class="sxs-lookup"><span data-stu-id="6d515-195">The following code shows the read model definition.</span></span>

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

<span data-ttu-id="6d515-196">A rendszer lehetővé teszi a felhasználóknak számára a termékek értékelését.</span><span class="sxs-lookup"><span data-stu-id="6d515-196">The system allows users to rate products.</span></span> <span data-ttu-id="6d515-197">Az alkalmazás kódja ezt az alábbi kódban szereplő `RateProduct` parancs használatával éri el.</span><span class="sxs-lookup"><span data-stu-id="6d515-197">The application code does this using the `RateProduct` command shown in the following code.</span></span>

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

<span data-ttu-id="6d515-198">A rendszer a `ProductsCommandHandler` osztályt használja az alkalmazás által küldött parancsok kezelésére.</span><span class="sxs-lookup"><span data-stu-id="6d515-198">The system uses the `ProductsCommandHandler` class to handle commands sent by the application.</span></span> <span data-ttu-id="6d515-199">Az ügyfelek általában egy üzenetkezelő rendszeren, például egy üzenetsoron keresztül küldenek a parancsokat a tartománynak.</span><span class="sxs-lookup"><span data-stu-id="6d515-199">Clients typically send commands to the domain through a messaging system such as a queue.</span></span> <span data-ttu-id="6d515-200">A parancskezelő fogadja ezeket a parancsokat, és meghívja a tartományi felület metódusait.</span><span class="sxs-lookup"><span data-stu-id="6d515-200">The command handler accepts these commands and invokes methods of the domain interface.</span></span> <span data-ttu-id="6d515-201">Az egyes parancsok részletességének célja a kérések ütközési esélyeinek csökkentése.</span><span class="sxs-lookup"><span data-stu-id="6d515-201">The granularity of each command is designed to reduce the chance of conflicting requests.</span></span> <span data-ttu-id="6d515-202">A következő kód a `ProductsCommandHandler` osztály vázlatát mutatja be.</span><span class="sxs-lookup"><span data-stu-id="6d515-202">The following code shows an outline of the `ProductsCommandHandler` class.</span></span>

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

<span data-ttu-id="6d515-203">A következő kód az írási modell `IProductsDomain` felületét mutatja be.</span><span class="sxs-lookup"><span data-stu-id="6d515-203">The following code shows the `IProductsDomain` interface from the write model.</span></span>

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

<span data-ttu-id="6d515-204">Azt is figyelje meg, hogy az `IProductsDomain` felület olyan metódusokat tartalmaz, amelyek jelentéssel bírnak a tartományban.</span><span class="sxs-lookup"><span data-stu-id="6d515-204">Also notice how the `IProductsDomain` interface contains methods that have a meaning in the domain.</span></span> <span data-ttu-id="6d515-205">Egy CRUD-környezetben ezek a metódusok a legtöbb esetben általános névvel lennénk ellátva (például `Save` vagy `Update`), és az egyetlen argumentumuk egy DTO lenne.</span><span class="sxs-lookup"><span data-stu-id="6d515-205">Typically, in a CRUD environment these methods would have generic names such as `Save` or `Update`, and have a DTO as the only argument.</span></span> <span data-ttu-id="6d515-206">A CQRS-megközelítés kialakítható úgy, hogy megfeleljen a vállalat üzleti és leltárkezelő rendszerei igényeinek.</span><span class="sxs-lookup"><span data-stu-id="6d515-206">The CQRS approach can be designed to meet the needs of this organization's business and inventory management systems.</span></span>

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="6d515-207">Kapcsolódó minták és útmutatók</span><span class="sxs-lookup"><span data-stu-id="6d515-207">Related patterns and guidance</span></span>

<span data-ttu-id="6d515-208">Az alábbi minták és útmutatások hasznosak lehetnek a minta használatakor:</span><span class="sxs-lookup"><span data-stu-id="6d515-208">The following patterns and guidance are useful when implementing this pattern:</span></span>

- <span data-ttu-id="6d515-209">A CQRS és más architektúrastílusok összehasonlításáért lásd: [Architektúrastílusok](/azure/architecture/guide/architecture-styles/) és [CQRS architektúrastílus](/azure/architecture/guide/architecture-styles/cqrs).</span><span class="sxs-lookup"><span data-stu-id="6d515-209">For a comparison of CQRS with other architectural styles, see [Architecture styles](/azure/architecture/guide/architecture-styles/) and [CQRS architecture style](/azure/architecture/guide/architecture-styles/cqrs).</span></span>

- <span data-ttu-id="6d515-210">[Adatkonzisztencia – Ismertető](https://msdn.microsoft.com/library/dn589800.aspx).</span><span class="sxs-lookup"><span data-stu-id="6d515-210">[Data Consistency Primer](https://msdn.microsoft.com/library/dn589800.aspx).</span></span> <span data-ttu-id="6d515-211">Részletezi az olvasási és írási adattárak végleges konzisztenciája által okozott általánosan felmerülő problémákat a CQRS minta használatakor, illetve azt is, hogy ezek a problémák hogyan oldhatók meg.</span><span class="sxs-lookup"><span data-stu-id="6d515-211">Explains the issues that are typically encountered due to eventual consistency between the read and write data stores when using the CQRS pattern, and how these issues can be resolved.</span></span>

- <span data-ttu-id="6d515-212">[Adatparticionálási útmutató](https://msdn.microsoft.com/library/dn589795.aspx).</span><span class="sxs-lookup"><span data-stu-id="6d515-212">[Data Partitioning Guidance](https://msdn.microsoft.com/library/dn589795.aspx).</span></span> <span data-ttu-id="6d515-213">Ismerteti, hogyan oszthatók fel a CQRS mintában használt olvasási és írási adattárak olyan partíciókra, amelyek a méretezhetőség, a versenyhelyzetek kialakulásának csökkentése és a teljesítmény optimalizálása érdekében külön kezelhetők és érhetők el.</span><span class="sxs-lookup"><span data-stu-id="6d515-213">Describes how the read and write data stores used in the CQRS pattern can be divided into partitions that can be managed and accessed separately to improve scalability, reduce contention, and optimize performance.</span></span>

- <span data-ttu-id="6d515-214">[Event Sourcing minta](event-sourcing.md).</span><span class="sxs-lookup"><span data-stu-id="6d515-214">[Event Sourcing Pattern](event-sourcing.md).</span></span> <span data-ttu-id="6d515-215">Részletesebben ismerteti, hogyan használható az Event Sourcing a CQRS mintával a feladatok egyszerűsítéséhez az összetett tartományokban a teljesítmény, a méretezhetőség és a válaszkészség növelése mellett.</span><span class="sxs-lookup"><span data-stu-id="6d515-215">Describes in more detail how Event Sourcing can be used with the CQRS pattern to simplify tasks in complex domains while improving performance, scalability, and responsiveness.</span></span> <span data-ttu-id="6d515-216">Emellett azt is bemutatja, hogyan biztosítható a tranzakciós adatok konzisztenciája a teljes körű naplók és előzmények fenntartása mellett, ami lehetővé teszi a kompenzáló műveletek végrehajtását.</span><span class="sxs-lookup"><span data-stu-id="6d515-216">As well as how to provide consistency for transactional data while maintaining full audit trails and history that can enable compensating actions.</span></span>

- <span data-ttu-id="6d515-217">[A Materialized View minta](materialized-view.md).</span><span class="sxs-lookup"><span data-stu-id="6d515-217">[Materialized View Pattern](materialized-view.md).</span></span> <span data-ttu-id="6d515-218">A CQRS megvalósítás olvasási modellje tartalmazhatja az írási modell adatainak materializált nézeteit, illetve a materializált nézetek létrehozására is használható.</span><span class="sxs-lookup"><span data-stu-id="6d515-218">The read model of a CQRS implementation can contain materialized views of the write model data, or the read model can be used to generate materialized views.</span></span>

- <span data-ttu-id="6d515-219">Minták és gyakorlatok útmutatója – [A CQRS felfedezése](http://aka.ms/cqrs).</span><span class="sxs-lookup"><span data-stu-id="6d515-219">The patterns & practices guide [CQRS Journey](http://aka.ms/cqrs).</span></span> <span data-ttu-id="6d515-220">[A Command Query Responsibility Segregation minta](https://msdn.microsoft.com/library/jj591573.aspx) című cikk bemutatja a mintát, illetve részletesen taglalja, hogy mikor érdemes azt használni, az [Epilógus: tapasztalatok](https://msdn.microsoft.com/library/jj591568.aspx) című cikk pedig segít a minta használata során felmerülő problémák egy részének megértésében.</span><span class="sxs-lookup"><span data-stu-id="6d515-220">In particular, [Introducing the Command Query Responsibility Segregation Pattern](https://msdn.microsoft.com/library/jj591573.aspx) explores the pattern and when it's useful, and [Epilogue: Lessons Learned](https://msdn.microsoft.com/library/jj591568.aspx) helps you understand some of the issues that come up when using this pattern.</span></span>

- <span data-ttu-id="6d515-221">A [Martin Fowler – CQRS](http://martinfowler.com/bliki/CQRS.html) című bejegyzés ismerteti a minta alapvető működését, továbbá egyéb hasznos források hivatkozásait is tartalmazza.</span><span class="sxs-lookup"><span data-stu-id="6d515-221">The post [CQRS by Martin Fowler](http://martinfowler.com/bliki/CQRS.html), which explains the basics of the pattern and links to other useful resources.</span></span>

- <span data-ttu-id="6d515-222">Greg Young [bejegyzéseiben](http://codebetter.com/gregyoung/) számos szempontból járja körül a CQRS mintát.</span><span class="sxs-lookup"><span data-stu-id="6d515-222">[Greg Young’s posts](http://codebetter.com/gregyoung/), which explore many aspects of the CQRS pattern.</span></span>
