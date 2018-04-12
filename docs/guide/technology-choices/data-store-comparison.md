---
title: Az adattár kiválasztásának kritériumai
description: Az Azure számítási lehetőségeinek áttekintése
author: MikeWasson
ms.openlocfilehash: 9cb2f77b854a38450490bc96bf0b6a2998ceb1c7
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/06/2018
---
# <a name="criteria-for-choosing-a-data-store"></a><span data-ttu-id="35eaa-103">Az adattár kiválasztásának kritériumai</span><span class="sxs-lookup"><span data-stu-id="35eaa-103">Criteria for choosing a data store</span></span>

<span data-ttu-id="35eaa-104">Az Azure számos adattárolási megoldástípust támogat, amelyek mindegyike más funkciókat és képességeket biztosít.</span><span class="sxs-lookup"><span data-stu-id="35eaa-104">Azure supports many types of data storage solutions, each providing different features and capabilities.</span></span> <span data-ttu-id="35eaa-105">Ez a cikk az adattárak értékelésekor szükséges összehasonlítási kritériumokat ismerteti.</span><span class="sxs-lookup"><span data-stu-id="35eaa-105">This article describes the comparison criteria you should use when evaluating a data store.</span></span> <span data-ttu-id="35eaa-106">A cél az, hogy segítséget nyújtson az aktuális megoldás igényeinek megfelelő adattárolási típus meghatározásában.</span><span class="sxs-lookup"><span data-stu-id="35eaa-106">The goal is to help you determine which data storage types can meet your solution's requirements.</span></span>

## <a name="general-considerations"></a><span data-ttu-id="35eaa-107">Általános szempontok</span><span class="sxs-lookup"><span data-stu-id="35eaa-107">General Considerations</span></span>

<span data-ttu-id="35eaa-108">Az összehasonlítás megkezdéséhez gyűjtsön annyit az alábbi, az adatigényeire vonatkozó információkból, amennyit csak tud.</span><span class="sxs-lookup"><span data-stu-id="35eaa-108">To start your comparison, gather as much of the following information as you can about your data needs.</span></span> <span data-ttu-id="35eaa-109">Ez az információ segít annak meghatározásában, hogy melyik adattárolási típus felel meg az igényeinek.</span><span class="sxs-lookup"><span data-stu-id="35eaa-109">This information will help you to determine which data storage types will meet your needs.</span></span>

### <a name="functional-requirements"></a><span data-ttu-id="35eaa-110">Funkcionális követelmények</span><span class="sxs-lookup"><span data-stu-id="35eaa-110">Functional requirements</span></span>

- <span data-ttu-id="35eaa-111">**Adatformátum**.</span><span class="sxs-lookup"><span data-stu-id="35eaa-111">**Data format**.</span></span> <span data-ttu-id="35eaa-112">Milyen típusú adatokat szándékozik tárolni?</span><span class="sxs-lookup"><span data-stu-id="35eaa-112">What type of data are you intending to store?</span></span> <span data-ttu-id="35eaa-113">A gyakori típusok közé tartoznak a tranzakciós adatok, a JSON-objektumok, a telemetriaadatok, a keresési indexek és az egybesimított fájlok.</span><span class="sxs-lookup"><span data-stu-id="35eaa-113">Common types include transactional data, JSON objects, telemetry, search indexes, or flat files.</span></span>
- <span data-ttu-id="35eaa-114">**Az adatok mérete**.</span><span class="sxs-lookup"><span data-stu-id="35eaa-114">**Data size**.</span></span> <span data-ttu-id="35eaa-115">Mekkora a tárolni kívánt entitások mérete?</span><span class="sxs-lookup"><span data-stu-id="35eaa-115">How large are the entities you need to store?</span></span> <span data-ttu-id="35eaa-116">Ezeket az entitásokat egyetlen dokumentumként kell fenntartani, vagy azok feloszthatók több dokumentumra, táblára, gyűjteményre stb.?</span><span class="sxs-lookup"><span data-stu-id="35eaa-116">Will these entities need to be maintained as a single document, or can they be split across multiple documents, tables, collections, and so forth?</span></span>
- <span data-ttu-id="35eaa-117">**Skálázás és struktúra**.</span><span class="sxs-lookup"><span data-stu-id="35eaa-117">**Scale and structure**.</span></span> <span data-ttu-id="35eaa-118">Milyen általános tárolókapacitásra van szüksége?</span><span class="sxs-lookup"><span data-stu-id="35eaa-118">What is the overall amount of storage capacity you need?</span></span> <span data-ttu-id="35eaa-119">Tervezi az adatok particionálását?</span><span class="sxs-lookup"><span data-stu-id="35eaa-119">Do you anticipate partitioning your data?</span></span> 
- <span data-ttu-id="35eaa-120">**Adatkapcsolatok**.</span><span class="sxs-lookup"><span data-stu-id="35eaa-120">**Data relationships**.</span></span> <span data-ttu-id="35eaa-121">Az adatoknak támogatniuk kell-e az egy-a-többhöz vagy a több-a-többhöz kapcsolatokat?</span><span class="sxs-lookup"><span data-stu-id="35eaa-121">Will your data need to support one-to-many or many-to-many relationships?</span></span> <span data-ttu-id="35eaa-122">Maguk a kapcsolatok fontos részét képezik az adatoknak?</span><span class="sxs-lookup"><span data-stu-id="35eaa-122">Are relationships themselves an important part of the data?</span></span> <span data-ttu-id="35eaa-123">Szükség lesz-e az adatok egyesítésére vagy egyéb típusú kombinálására ugyanazon vagy más külső adatkészletekből származó adatokkal?</span><span class="sxs-lookup"><span data-stu-id="35eaa-123">Will you need to join or otherwise combine data from within the same dataset, or from external datasets?</span></span> 
- <span data-ttu-id="35eaa-124">**Konzisztenciamodell**.</span><span class="sxs-lookup"><span data-stu-id="35eaa-124">**Consistency model**.</span></span> <span data-ttu-id="35eaa-125">Mennyire fontos, hogy az egy csomóponton végzett frissítések más csomópontokon is megjelenjenek, mielőtt további módosításokat lehetne eszközölni?</span><span class="sxs-lookup"><span data-stu-id="35eaa-125">How important is it for updates made in one node to appear in other nodes, before further changes can be made?</span></span> <span data-ttu-id="35eaa-126">Elfogadható-e a végleges konzisztencia?</span><span class="sxs-lookup"><span data-stu-id="35eaa-126">Can you accept eventual consistency?</span></span> <span data-ttu-id="35eaa-127">Szüksége van-e ACID-garanciákra az egyes tranzakciókhoz?</span><span class="sxs-lookup"><span data-stu-id="35eaa-127">Do you need ACID guarantees for transactions?</span></span>
- <span data-ttu-id="35eaa-128">**A séma rugalmassága**.</span><span class="sxs-lookup"><span data-stu-id="35eaa-128">**Schema flexibility**.</span></span> <span data-ttu-id="35eaa-129">Milyen sémát fog alkalmazni az adatokon?</span><span class="sxs-lookup"><span data-stu-id="35eaa-129">What kind of schemas will you apply to your data?</span></span> <span data-ttu-id="35eaa-130">Rögzített sémát, esetleg írási séma vagy olvasási séma megközelítést fog használni?</span><span class="sxs-lookup"><span data-stu-id="35eaa-130">Will you use a fixed schema, a schema-on-write approach, or a schema-on-read approach?</span></span>
- <span data-ttu-id="35eaa-131">**Egyidejűség**.</span><span class="sxs-lookup"><span data-stu-id="35eaa-131">**Concurrency**.</span></span> <span data-ttu-id="35eaa-132">Milyen egyidejűségi mechanizmust kíván használni az adatok frissítésekor és szinkronizálásakor?</span><span class="sxs-lookup"><span data-stu-id="35eaa-132">What kind of concurrency mechanism do you want to use when updating and synchronizing data?</span></span> <span data-ttu-id="35eaa-133">Az alkalmazás sok olyan frissítést fog végezni, amelyek ütközhetnek?</span><span class="sxs-lookup"><span data-stu-id="35eaa-133">Will the application perform many updates that could potentially conflict.</span></span> <span data-ttu-id="35eaa-134">Ha igen, lehetséges, hogy rekordzárolási és pesszimista egyidejűségi vezérlőkre lesz szüksége.</span><span class="sxs-lookup"><span data-stu-id="35eaa-134">If so, you may requiring record locking and pessimistic concurrency control.</span></span> <span data-ttu-id="35eaa-135">Vagy támogathatja-e az optimista egyidejűségi vezérlőket?</span><span class="sxs-lookup"><span data-stu-id="35eaa-135">Alternatively, can you support optimistic concurrency controls?</span></span> <span data-ttu-id="35eaa-136">Ha igen, elegendő-e az egyszerű időbélyegző-alapú egyidejűségi vezérlés, vagy szüksége van-e többverziós egyidejűségi vezérlésre?</span><span class="sxs-lookup"><span data-stu-id="35eaa-136">If so, is simple timestamp-based concurrency control enough, or do you need the added functionality of multi-version concurrency control?</span></span>
- <span data-ttu-id="35eaa-137">**Adatáthelyezés**.</span><span class="sxs-lookup"><span data-stu-id="35eaa-137">**Data movement**.</span></span> <span data-ttu-id="35eaa-138">A megoldásnak végre kell-e hajtania ETL-feladatokat az adatok másik tárolóba vagy adattárházba történő áthelyezéséhez?</span><span class="sxs-lookup"><span data-stu-id="35eaa-138">Will your solution need to perform ETL tasks to move data to other stores or data warehouses?</span></span>
- <span data-ttu-id="35eaa-139">**Az adatok életciklusa**.</span><span class="sxs-lookup"><span data-stu-id="35eaa-139">**Data lifecycle**.</span></span> <span data-ttu-id="35eaa-140">Az adatok egyszer írhatók és többször olvashatók?</span><span class="sxs-lookup"><span data-stu-id="35eaa-140">Is the data write-once, read-many?</span></span> <span data-ttu-id="35eaa-141">Áthelyezhetők-e ritkán használt vagy offline tárhelyre?</span><span class="sxs-lookup"><span data-stu-id="35eaa-141">Can it be moved into cool or cold storage?</span></span>
- <span data-ttu-id="35eaa-142">**Egyéb támogatott funkciók**.</span><span class="sxs-lookup"><span data-stu-id="35eaa-142">**Other supported features**.</span></span> <span data-ttu-id="35eaa-143">Szüksége van-e bármilyen más konkrét funkcióra, például sémaérvényesítésre, összesítésre, indexelésre, teljes szöveges keresésre, esetleg MapReduce-feladatra vagy egyéb lekérdezési képességekre?</span><span class="sxs-lookup"><span data-stu-id="35eaa-143">Do you need any other specific features, such as schema validation, aggregation, indexing, full-text search, MapReduce, or other query capabilities?</span></span>

### <a name="non-functional-requirements"></a><span data-ttu-id="35eaa-144">Nem funkcionális követelmények</span><span class="sxs-lookup"><span data-stu-id="35eaa-144">Non-functional requirements</span></span>

- <span data-ttu-id="35eaa-145">**Teljesítmény és skálázhatóság**.</span><span class="sxs-lookup"><span data-stu-id="35eaa-145">**Performance and scalability**.</span></span> <span data-ttu-id="35eaa-146">Milyen igényei vannak az adatteljesítményre vonatkozóan?</span><span class="sxs-lookup"><span data-stu-id="35eaa-146">What are your data performance requirements?</span></span> <span data-ttu-id="35eaa-147">Vannak-e egyedi igényei az adatok betöltése és feldolgozása szempontjából?</span><span class="sxs-lookup"><span data-stu-id="35eaa-147">Do you have specific requirements for data ingestion rates and data processing rates?</span></span> <span data-ttu-id="35eaa-148">Milyen elfogadható válaszidők vonatkoznak az adatok lekérdezésére és összesítésére a beolvasás után?</span><span class="sxs-lookup"><span data-stu-id="35eaa-148">What are the acceptable response times for querying and aggregation of data once ingested?</span></span> <span data-ttu-id="35eaa-149">Milyen mértékű vertikális felskálázásra lesz szüksége az adattárhoz?</span><span class="sxs-lookup"><span data-stu-id="35eaa-149">How large will you need the data store to scale up?</span></span> <span data-ttu-id="35eaa-150">A munkaterhelések nagyrészt olvasási vagy nagyrészt írási műveltekből állnak?</span><span class="sxs-lookup"><span data-stu-id="35eaa-150">Is your workload more read-heavy or write-heavy?</span></span>
- <span data-ttu-id="35eaa-151">**Megbízhatóság**.</span><span class="sxs-lookup"><span data-stu-id="35eaa-151">**Reliability**.</span></span> <span data-ttu-id="35eaa-152">Milyen általános SLA-támogatásra van szüksége?</span><span class="sxs-lookup"><span data-stu-id="35eaa-152">What overall SLA do you need to support?</span></span> <span data-ttu-id="35eaa-153">Milyen szintű hibatűrést kíván nyújtani az adatfelhasználók számára?</span><span class="sxs-lookup"><span data-stu-id="35eaa-153">What level of fault-tolerance do you need to provide for data consumers?</span></span> <span data-ttu-id="35eaa-154">Milyen típusú biztonsági mentési és visszaállítási képességekre van szüksége?</span><span class="sxs-lookup"><span data-stu-id="35eaa-154">What kind of backup and restore capabilities do you need?</span></span> 
- <span data-ttu-id="35eaa-155">**Replikáció**.</span><span class="sxs-lookup"><span data-stu-id="35eaa-155">**Replication**.</span></span> <span data-ttu-id="35eaa-156">Szükség lesz-e az adatok több replika vagy régió közötti szétosztására?</span><span class="sxs-lookup"><span data-stu-id="35eaa-156">Will your data need to be distributed among multiple replicas or regions?</span></span> <span data-ttu-id="35eaa-157">Milyen típusú adatreplikációs képességekre van szüksége?</span><span class="sxs-lookup"><span data-stu-id="35eaa-157">What kind of data replication capabilities do you require?</span></span> 
- <span data-ttu-id="35eaa-158">**Korlátok**.</span><span class="sxs-lookup"><span data-stu-id="35eaa-158">**Limits**.</span></span> <span data-ttu-id="35eaa-159">Beleférnek-e egy adott adattár korlátaiba a skálázási, a kapcsolatok számára vonatkozó, illetve az átviteli követelményei?</span><span class="sxs-lookup"><span data-stu-id="35eaa-159">Will the limits of a particular data store support your requirements for scale, number of connections, and throughput?</span></span> 

### <a name="management-and-cost"></a><span data-ttu-id="35eaa-160">Felügyelet és költségek</span><span class="sxs-lookup"><span data-stu-id="35eaa-160">Management and cost</span></span>

- <span data-ttu-id="35eaa-161">**Felügyelt szolgáltatás**.</span><span class="sxs-lookup"><span data-stu-id="35eaa-161">**Managed service**.</span></span> <span data-ttu-id="35eaa-162">Ha lehetséges, használjon felügyelt adatszolgáltatást, hacsak nem kifejezetten olyan képességekre van szüksége, amelyek csak egy IaaS által üzemeltetett adattárban találhatók meg.</span><span class="sxs-lookup"><span data-stu-id="35eaa-162">When possible, use a managed data service, unless you require specific capabilities that can only be found in an IaaS-hosted data store.</span></span>
- <span data-ttu-id="35eaa-163">**Régiónkénti rendelkezésre állás**.</span><span class="sxs-lookup"><span data-stu-id="35eaa-163">**Region availability**.</span></span> <span data-ttu-id="35eaa-164">Felügyelt szolgáltatások esetén a szolgáltatás elérhető-e az összes Azure-régióban?</span><span class="sxs-lookup"><span data-stu-id="35eaa-164">For managed services, is the service available in all Azure regions?</span></span> <span data-ttu-id="35eaa-165">A megoldást bizonyos Azure-régiókban kell-e üzemeltetni?</span><span class="sxs-lookup"><span data-stu-id="35eaa-165">Does your solution need to be hosted in certain Azure regions?</span></span>
- <span data-ttu-id="35eaa-166">**Hordozhatóság**.</span><span class="sxs-lookup"><span data-stu-id="35eaa-166">**Portability**.</span></span> <span data-ttu-id="35eaa-167">Szükség lesz-e az adatok helyszíni vagy külső adatközpontokba, esetleg egyéb felhőalapú üzemeltetési környezetekbe történő migrálására?</span><span class="sxs-lookup"><span data-stu-id="35eaa-167">Will your data need to migrated to on-premises, external datacenters, or other cloud hosting environments?</span></span>
- <span data-ttu-id="35eaa-168">**Licencelés**.</span><span class="sxs-lookup"><span data-stu-id="35eaa-168">**Licensing**.</span></span> <span data-ttu-id="35eaa-169">Előnyben részesíti-e a jogvédett licencet az OSS licenctípussal szemben?</span><span class="sxs-lookup"><span data-stu-id="35eaa-169">Do you have a preference of a proprietary versus OSS license type?</span></span> <span data-ttu-id="35eaa-170">Vonatkozik-e egyéb külső korlátozás a használt licenc típusára vonatkozóan?</span><span class="sxs-lookup"><span data-stu-id="35eaa-170">Are there any other external restrictions on what type of license you can use?</span></span>
- <span data-ttu-id="35eaa-171">**A teljes költség**.</span><span class="sxs-lookup"><span data-stu-id="35eaa-171">**Overall cost**.</span></span> <span data-ttu-id="35eaa-172">Milyen mértékű a megoldásban használt szolgáltatás teljes költsége?</span><span class="sxs-lookup"><span data-stu-id="35eaa-172">What is the overall cost of using the service within your solution?</span></span> <span data-ttu-id="35eaa-173">Hány példánynak kell futnia az üzemidő- és a teljesítménybeli követelmények kiszolgálásához?</span><span class="sxs-lookup"><span data-stu-id="35eaa-173">How many instances will need to run, to support your uptime and throughput requirements?</span></span> <span data-ttu-id="35eaa-174">Ennél a számításnál az üzemeltetés költségeit is vegye figyelembe.</span><span class="sxs-lookup"><span data-stu-id="35eaa-174">Consider operations costs in this calculation.</span></span> <span data-ttu-id="35eaa-175">A felügyelt szolgáltatások használatának egyik előnye az alacsonyabb üzemeltetési költség.</span><span class="sxs-lookup"><span data-stu-id="35eaa-175">One reason to prefer managed services is the reduced operational cost.</span></span>
- <span data-ttu-id="35eaa-176">**Költséghatékonyság**.</span><span class="sxs-lookup"><span data-stu-id="35eaa-176">**Cost effectiveness**.</span></span> <span data-ttu-id="35eaa-177">Lehetséges-e az adatok particionálása a költséghatékonyabb tárolás érdekében?</span><span class="sxs-lookup"><span data-stu-id="35eaa-177">Can you partition your data, to store it more cost effectively?</span></span> <span data-ttu-id="35eaa-178">Például áthelyezhetők-e a nagy méretű objektumok egy költséges relációs adatbázisból egy objektumtárolóba?</span><span class="sxs-lookup"><span data-stu-id="35eaa-178">For example, can you move large objects out of an expensive relational database into an object store?</span></span>

### <a name="security"></a><span data-ttu-id="35eaa-179">Biztonság</span><span class="sxs-lookup"><span data-stu-id="35eaa-179">Security</span></span>

- <span data-ttu-id="35eaa-180">**Biztonság**.</span><span class="sxs-lookup"><span data-stu-id="35eaa-180">**Security**.</span></span> <span data-ttu-id="35eaa-181">Milyen típusú titkosításra van szüksége?</span><span class="sxs-lookup"><span data-stu-id="35eaa-181">What type of encryption do you require?</span></span> <span data-ttu-id="35eaa-182">Szüksége van titkosításra inaktív állapotban?</span><span class="sxs-lookup"><span data-stu-id="35eaa-182">Do you need encryption at rest?</span></span> <span data-ttu-id="35eaa-183">Milyen hitelesítési mechanizmust szeretne használni az adatokhoz való kapcsolódáshoz?</span><span class="sxs-lookup"><span data-stu-id="35eaa-183">What authentication mechanism do you want to use to connect to your data?</span></span>
- <span data-ttu-id="35eaa-184">**Naplózás**.</span><span class="sxs-lookup"><span data-stu-id="35eaa-184">**Auditing**.</span></span> <span data-ttu-id="35eaa-185">Milyen típusú naplót szeretne létrehozni?</span><span class="sxs-lookup"><span data-stu-id="35eaa-185">What kind of audit log do you need to generate?</span></span>
- <span data-ttu-id="35eaa-186">**Hálózati követelmények**.</span><span class="sxs-lookup"><span data-stu-id="35eaa-186">**Networking requirements**.</span></span> <span data-ttu-id="35eaa-187">Szeretné-e korlátozni vagy egyéb módon kezelni a más hálózati forrásokból származó adatokat?</span><span class="sxs-lookup"><span data-stu-id="35eaa-187">Do you need to restrict or otherwise manage access to your data from other network resources?</span></span> <span data-ttu-id="35eaa-188">Az adatok elérésére csak az Azure-környezeten belülről van szükség?</span><span class="sxs-lookup"><span data-stu-id="35eaa-188">Does data need to be accessible only from inside the Azure environment?</span></span> <span data-ttu-id="35eaa-189">Az adatoknak adott IP-címekről vagy alhálózatokról is elérhetőnek kell lenniük?</span><span class="sxs-lookup"><span data-stu-id="35eaa-189">Does the data need to be accessible from specific IP addresses or subnets?</span></span> <span data-ttu-id="35eaa-190">Elérhetőnek kell-e lenniük a helyszínen üzemeltetett alkalmazások vagy szolgáltatások, esetleg egyéb külső adatközpontok számára?</span><span class="sxs-lookup"><span data-stu-id="35eaa-190">Does it need to be accessible from applications or services hosted on-premises or in other external datacenters?</span></span>

### <a name="devops"></a><span data-ttu-id="35eaa-191">DevOps</span><span class="sxs-lookup"><span data-stu-id="35eaa-191">DevOps</span></span>

- <span data-ttu-id="35eaa-192">**Képzettség**.</span><span class="sxs-lookup"><span data-stu-id="35eaa-192">**Skill set**.</span></span> <span data-ttu-id="35eaa-193">Vannak-e olyan programozási nyelvek, operációs rendszerek és egyéb technológiák, amelyek használatában a csapata különösen képzett?</span><span class="sxs-lookup"><span data-stu-id="35eaa-193">Are there particular programming languages, operating systems, or other technology that your team is particularly adept at using?</span></span> <span data-ttu-id="35eaa-194">Vannak-e olyanok, amelyekkel a csapata nehezen boldogulna?</span><span class="sxs-lookup"><span data-stu-id="35eaa-194">Are there others that would be difficult for your team to work with?</span></span>
- <span data-ttu-id="35eaa-195">**Ügyfelek**. Elérhető-e megfelelő szintű ügyféltámogatás az Ön által használt fejlesztési nyelvekhez?</span><span class="sxs-lookup"><span data-stu-id="35eaa-195">**Clients** Is there good client support for your development languages?</span></span>

<span data-ttu-id="35eaa-196">A következő szakaszokban különböző adattármodelleket hasonlítunk össze a számításifeladat-profil, az adattípusok és az alkalmazási helyzetekre vonatkozó példák alapján.</span><span class="sxs-lookup"><span data-stu-id="35eaa-196">The following sections compare various data store models in terms of workload profile, data types, and example use cases.</span></span>

## <a name="relational-database-management-systems-rdbms"></a><span data-ttu-id="35eaa-197">Relációsadatbázis-kezelő rendszerek (RDBMS)</span><span class="sxs-lookup"><span data-stu-id="35eaa-197">Relational database management systems (RDBMS)</span></span>

<table>
<tr><td><span data-ttu-id="35eaa-198"><strong>Számítási feladat</strong></span><span class="sxs-lookup"><span data-stu-id="35eaa-198"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="35eaa-199">Mind az új rekordok létrehozása, mind a meglévő adatok frissítése rendszeresen megtörténik.</span><span class="sxs-lookup"><span data-stu-id="35eaa-199">Both the creation of new records and updates to existing data happen regularly.</span></span></li>
            <li><span data-ttu-id="35eaa-200">Több műveletet kell egyetlen tranzakción belül végrehajtani.</span><span class="sxs-lookup"><span data-stu-id="35eaa-200">Multiple operations have to be completed in a single transaction.</span></span></li>
            <li><span data-ttu-id="35eaa-201">Kereszttáblák létrehozásához összesítő függvényekre van szükség.</span><span class="sxs-lookup"><span data-stu-id="35eaa-201">Requires aggregation functions to perform cross-tabulation.</span></span></li>
            <li><span data-ttu-id="35eaa-202">Jól integrálhatónak kell lennie a jelentéskészítő eszközökkel.</span><span class="sxs-lookup"><span data-stu-id="35eaa-202">Strong integration with reporting tools is required.</span></span></li>
            <li><span data-ttu-id="35eaa-203">A kapcsolatok kényszerítése az adatbázis korlátozásai révén valósul meg.</span><span class="sxs-lookup"><span data-stu-id="35eaa-203">Relationships are enforced using database constraints.</span></span></li>
            <li><span data-ttu-id="35eaa-204">A lekérdezési teljesítmény optimalizálása indexekkel történik.</span><span class="sxs-lookup"><span data-stu-id="35eaa-204">Indexes are used to optimize query performance.</span></span></li>
            <li><span data-ttu-id="35eaa-205">Lehetővé teszi az adatok bizonyos részhalmazainak elérését.</span><span class="sxs-lookup"><span data-stu-id="35eaa-205">Allows access to specific subsets of data.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="35eaa-206"><strong>Adattípus</strong></span><span class="sxs-lookup"><span data-stu-id="35eaa-206"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="35eaa-207">Az adatok nagymértékben normalizáltak.</span><span class="sxs-lookup"><span data-stu-id="35eaa-207">Data is highly normalized.</span></span></li>
            <li><span data-ttu-id="35eaa-208">Az adatbázissémák szükségesek és kényszerítettek.</span><span class="sxs-lookup"><span data-stu-id="35eaa-208">Database schemas are required and enforced.</span></span></li>
            <li><span data-ttu-id="35eaa-209">Több-a-többhöz kapcsolatok az adatbázisban szereplő adatentitások között.</span><span class="sxs-lookup"><span data-stu-id="35eaa-209">Many-to-many relationships between data entities in the database.</span></span></li>
            <li><span data-ttu-id="35eaa-210">A korlátozásokat a séma határozza meg, és alkalmazza az adatbázisban szereplő minden adatra.</span><span class="sxs-lookup"><span data-stu-id="35eaa-210">Constraints are defined in the schema and imposed on any data in the database.</span></span></li>
            <li><span data-ttu-id="35eaa-211">Szükséges az adatok nagy fokú integritása.</span><span class="sxs-lookup"><span data-stu-id="35eaa-211">Data requires high integrity.</span></span> <span data-ttu-id="35eaa-212">Az indexeket és kapcsolatokat pontosan karban kell tartani.</span><span class="sxs-lookup"><span data-stu-id="35eaa-212">Indexes and relationships need to be maintained accurately.</span></span></li>
            <li><span data-ttu-id="35eaa-213">Szükséges az adatok erős konzisztenciája.</span><span class="sxs-lookup"><span data-stu-id="35eaa-213">Data requires strong consistency.</span></span> <span data-ttu-id="35eaa-214">A tranzakciók működésének módja biztosítja, hogy minden adat minden felhasználó és folyamat számára teljes mértékben megegyezzen.</span><span class="sxs-lookup"><span data-stu-id="35eaa-214">Transactions operate in a way that ensures all data are 100% consistent for all users and processes.</span></span></li>
            <li><span data-ttu-id="35eaa-215">Az egyes adatbeviteleknek kis vagy közepes méretűnek kell lenniük.</span><span class="sxs-lookup"><span data-stu-id="35eaa-215">Size of individual data entries is intended to be small to medium-sized.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="35eaa-216"><strong>Példák</strong></span><span class="sxs-lookup"><span data-stu-id="35eaa-216"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="35eaa-217">Üzletág (emberierőforrás-kezelés, ügyfélkapcsolat-kezelés, vállalati erőforrás-tervezés)</span><span class="sxs-lookup"><span data-stu-id="35eaa-217">Line of business  (human capital management, customer relationship management, enterprise resource planning)</span></span></li>
            <li><span data-ttu-id="35eaa-218">Leltárkezelés</span><span class="sxs-lookup"><span data-stu-id="35eaa-218">Inventory management</span></span></li>
            <li><span data-ttu-id="35eaa-219">Jelentéskészítési adatbázis</span><span class="sxs-lookup"><span data-stu-id="35eaa-219">Reporting database</span></span></li>
            <li><span data-ttu-id="35eaa-220">Könyvelés</span><span class="sxs-lookup"><span data-stu-id="35eaa-220">Accounting</span></span></li>
            <li><span data-ttu-id="35eaa-221">Eszközkezelés</span><span class="sxs-lookup"><span data-stu-id="35eaa-221">Asset management</span></span></li>
            <li><span data-ttu-id="35eaa-222">Alaptőke-kezelés</span><span class="sxs-lookup"><span data-stu-id="35eaa-222">Fund management</span></span></li>
            <li><span data-ttu-id="35eaa-223">Rendeléskezelés</span><span class="sxs-lookup"><span data-stu-id="35eaa-223">Order management</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="document-databases"></a><span data-ttu-id="35eaa-224">Dokumentum-adatbázisok</span><span class="sxs-lookup"><span data-stu-id="35eaa-224">Document databases</span></span>

<table>
<tr><td><span data-ttu-id="35eaa-225"><strong>Számítási feladat</strong></span><span class="sxs-lookup"><span data-stu-id="35eaa-225"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="35eaa-226">Általános célú.</span><span class="sxs-lookup"><span data-stu-id="35eaa-226">General purpose.</span></span></li>
            <li><span data-ttu-id="35eaa-227">Gyakoriak a beszúrási és frissítési műveletek.</span><span class="sxs-lookup"><span data-stu-id="35eaa-227">Insert and update operations are common.</span></span> <span data-ttu-id="35eaa-228">Mind az új rekordok létrehozása, mind a meglévő adatok frissítése rendszeresen megtörténik.</span><span class="sxs-lookup"><span data-stu-id="35eaa-228">Both the creation of new records and updates to existing data happen regularly.</span></span></li>
            <li><span data-ttu-id="35eaa-229">Nincs objektumrelációs impedanica-eltérés.</span><span class="sxs-lookup"><span data-stu-id="35eaa-229">No object-relational impedance mismatch.</span></span> <span data-ttu-id="35eaa-230">A dokumentumok jobban illeszkednek az alkalmazáskódban használt objektumstruktúrákhoz.</span><span class="sxs-lookup"><span data-stu-id="35eaa-230">Documents can better match the object structures used in application code.</span></span></li>
            <li><span data-ttu-id="35eaa-231">Gyakrabban használnak optimista konkurenciát.</span><span class="sxs-lookup"><span data-stu-id="35eaa-231">Optimistic concurrency is more commonly used.</span></span></li>
            <li><span data-ttu-id="35eaa-232">Az adatokat a felhasználó alkalmazásnak kell módosítania és feldolgoznia.</span><span class="sxs-lookup"><span data-stu-id="35eaa-232">Data must be modified and processed by consuming application.</span></span></li>
            <li><span data-ttu-id="35eaa-233">Az adatokhoz többmezős indexelésre van szükség.</span><span class="sxs-lookup"><span data-stu-id="35eaa-233">Data requires index on multiple fields.</span></span></li>
            <li><span data-ttu-id="35eaa-234">Egyes dokumentumokat egyetlen blokként olvas be és ír a rendszer.</span><span class="sxs-lookup"><span data-stu-id="35eaa-234">Individual documents are retrieved and written as a single block.</span></span></li>
    </td>
</tr>
<tr><td><span data-ttu-id="35eaa-235"><strong>Adattípus</strong></span><span class="sxs-lookup"><span data-stu-id="35eaa-235"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="35eaa-236">Az adatok denormalizált módon is kezelhetők.</span><span class="sxs-lookup"><span data-stu-id="35eaa-236">Data can be managed in de-normalized way.</span></span></li>
            <li><span data-ttu-id="35eaa-237">Az egyes dokumentumadatok mérete viszonylag kicsi.</span><span class="sxs-lookup"><span data-stu-id="35eaa-237">Size of individual document data is relatively small.</span></span></li>
            <li><span data-ttu-id="35eaa-238">Minden egyes dokumentumtípus használhatja a saját sémáját.</span><span class="sxs-lookup"><span data-stu-id="35eaa-238">Each document type can use its own schema.</span></span></li>
            <li><span data-ttu-id="35eaa-239">A dokumentumok nem kötelező mezőket is tartalmazhatnak.</span><span class="sxs-lookup"><span data-stu-id="35eaa-239">Documents can include optional fields.</span></span></li>
            <li><span data-ttu-id="35eaa-240">A dokumentumadatok részben strukturáltak, ami azt jelenti, hogy az egyes mezők adattípusai nincsenek szigorúan meghatározva.</span><span class="sxs-lookup"><span data-stu-id="35eaa-240">Document data is semi-structured, meaning that data types of each field are not strictly defined.</span></span></li>
            <li><span data-ttu-id="35eaa-241">Az adatösszesítés támogatott.</span><span class="sxs-lookup"><span data-stu-id="35eaa-241">Data aggregation is supported.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="35eaa-242"><strong>Példák</strong></span><span class="sxs-lookup"><span data-stu-id="35eaa-242"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="35eaa-243">Termékkatalógus</span><span class="sxs-lookup"><span data-stu-id="35eaa-243">Product catalog</span></span></li>
            <li><span data-ttu-id="35eaa-244">Felhasználói fiókok</span><span class="sxs-lookup"><span data-stu-id="35eaa-244">User accounts</span></span></li>
            <li><span data-ttu-id="35eaa-245">Anyagjegyzék</span><span class="sxs-lookup"><span data-stu-id="35eaa-245">Bill of materials</span></span></li>
            <li><span data-ttu-id="35eaa-246">Személyre szabás</span><span class="sxs-lookup"><span data-stu-id="35eaa-246">Personalization</span></span></li>
            <li><span data-ttu-id="35eaa-247">Tartalomkezelés</span><span class="sxs-lookup"><span data-stu-id="35eaa-247">Content management</span></span></li>
            <li><span data-ttu-id="35eaa-248">Műveleti adatok</span><span class="sxs-lookup"><span data-stu-id="35eaa-248">Operations data</span></span></li>
            <li><span data-ttu-id="35eaa-249">Leltárkezelés</span><span class="sxs-lookup"><span data-stu-id="35eaa-249">Inventory management</span></span></li>
            <li><span data-ttu-id="35eaa-250">Tranzakció-előzményadatok</span><span class="sxs-lookup"><span data-stu-id="35eaa-250">Transaction history data</span></span></li>
            <li><span data-ttu-id="35eaa-251">Más NoSQL-tárolók materializált nézete.</span><span class="sxs-lookup"><span data-stu-id="35eaa-251">Materialized view of other NoSQL stores.</span></span> <span data-ttu-id="35eaa-252">Fájl cseréje/BLOB-indexelés.</span><span class="sxs-lookup"><span data-stu-id="35eaa-252">Replaces file/BLOB indexing.</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="keyvalue-stores"></a><span data-ttu-id="35eaa-253">Kulcs/érték tárolók</span><span class="sxs-lookup"><span data-stu-id="35eaa-253">Key/value stores</span></span>

<table>
<tr><td><span data-ttu-id="35eaa-254"><strong>Számítási feladat</strong></span><span class="sxs-lookup"><span data-stu-id="35eaa-254"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="35eaa-255">Az adatokat egy szótárhoz hasonlóan azonosítja és éri el.</span><span class="sxs-lookup"><span data-stu-id="35eaa-255">Data is identified and accessed using a single ID key, like a dictionary.</span></span></li>
            <li><span data-ttu-id="35eaa-256">Nagy mértékben skálázható.</span><span class="sxs-lookup"><span data-stu-id="35eaa-256">Massively scalable.</span></span></li>
            <li><span data-ttu-id="35eaa-257">Nincs szükség illesztésre, zárolásra vagy egyesítésre.</span><span class="sxs-lookup"><span data-stu-id="35eaa-257">No joins, lock, or unions are required.</span></span></li>
            <li><span data-ttu-id="35eaa-258">Nem használ összesítési mechanizmusokat.</span><span class="sxs-lookup"><span data-stu-id="35eaa-258">No aggregation mechanisms are used.</span></span></li>
            <li><span data-ttu-id="35eaa-259">A másodlagos indexeket általában nem használja.</span><span class="sxs-lookup"><span data-stu-id="35eaa-259">Secondary indexes are generally not used.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="35eaa-260"><strong>Adattípus</strong></span><span class="sxs-lookup"><span data-stu-id="35eaa-260"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="35eaa-261">Az adatok mérete általában nagy.</span><span class="sxs-lookup"><span data-stu-id="35eaa-261">Data size tends to be large.</span></span></li>
            <li><span data-ttu-id="35eaa-262">Minden kulcs egyetlen értékhez van társítva, amely egy nem felügyelt adatblob.</span><span class="sxs-lookup"><span data-stu-id="35eaa-262">Each key is associated with a single value, which is an unmanaged data BLOB.</span></span></li>
            <li><span data-ttu-id="35eaa-263">Nincs sémakényszerítés.</span><span class="sxs-lookup"><span data-stu-id="35eaa-263">There is no schema enforcement.</span></span></li>
            <li><span data-ttu-id="35eaa-264">Nincsenek entitások közötti kapcsolatok.</span><span class="sxs-lookup"><span data-stu-id="35eaa-264">No relationships between entities.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="35eaa-265"><strong>Példák</strong></span><span class="sxs-lookup"><span data-stu-id="35eaa-265"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="35eaa-266">Adatok gyorsítótárazása</span><span class="sxs-lookup"><span data-stu-id="35eaa-266">Data caching</span></span></li>
            <li><span data-ttu-id="35eaa-267">Munkamenet-kezelés</span><span class="sxs-lookup"><span data-stu-id="35eaa-267">Session management</span></span></li>
            <li><span data-ttu-id="35eaa-268">Felhasználói beállítások és profilkezelés</span><span class="sxs-lookup"><span data-stu-id="35eaa-268">User preference and profile management</span></span></li>
            <li><span data-ttu-id="35eaa-269">Termékjavaslat és reklámszolgáltatás</span><span class="sxs-lookup"><span data-stu-id="35eaa-269">Product recommendation and ad serving</span></span></li>
            <li><span data-ttu-id="35eaa-270">Szótárak</span><span class="sxs-lookup"><span data-stu-id="35eaa-270">Dictionaries</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="graph-databases"></a><span data-ttu-id="35eaa-271">Gráfadatbázisok</span><span class="sxs-lookup"><span data-stu-id="35eaa-271">Graph databases</span></span>

<table>
<tr><td><span data-ttu-id="35eaa-272"><strong>Számítási feladat</strong></span><span class="sxs-lookup"><span data-stu-id="35eaa-272"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="35eaa-273">Az adatelemek közötti kapcsolatok nagyon összetettek, számos ugrással a kapcsolódó adatelemek között.</span><span class="sxs-lookup"><span data-stu-id="35eaa-273">The relationships between data items are very complex, involving many hops between related data items.</span></span></li>
            <li><span data-ttu-id="35eaa-274">Az adatelemek közötti kapcsolat dinamikus, és az idő előrehaladtával változik.</span><span class="sxs-lookup"><span data-stu-id="35eaa-274">The relationship between data items are dynamic and change over time.</span></span></li>
            <li><span data-ttu-id="35eaa-275">Az objektumok közötti kapcsolatok kiváló hozzáférést biztosítanak, a bejáráshoz nincs szükségük külső kulcsokra vagy illesztésekre.</span><span class="sxs-lookup"><span data-stu-id="35eaa-275">Relationships between objects are first-class citizens, without requiring foreign-keys and joins to traverse.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="35eaa-276"><strong>Adattípus</strong></span><span class="sxs-lookup"><span data-stu-id="35eaa-276"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="35eaa-277">Az adatok csomópontokból és kapcsolatokból állnak.</span><span class="sxs-lookup"><span data-stu-id="35eaa-277">Data is comprised of nodes and relationships.</span></span></li>
            <li><span data-ttu-id="35eaa-278">A csomópontok a táblázatok soraihoz vagy a JSON-dokumentumokhoz hasonlítanak.</span><span class="sxs-lookup"><span data-stu-id="35eaa-278">Nodes are similar to table rows or JSON documents.</span></span></li>
            <li><span data-ttu-id="35eaa-279">A kapcsolatok épp olyan fontosak, mint a csomópontok, és a lekérdezési nyelvben közvetlenül megjelennek.</span><span class="sxs-lookup"><span data-stu-id="35eaa-279">Relationships are just as important as nodes, and are exposed directly in the query language.</span></span></li>
            <li><span data-ttu-id="35eaa-280">Az összetett objektumokat (például egy több telefonszámmal rendelkező személy) a rendszer általában több kisebb csomópontra osztja szét, amelyeket bejárható kapcsolatok kötnek össze.</span><span class="sxs-lookup"><span data-stu-id="35eaa-280">Composite objects, such as a person with multiple phone numbers, tend to be broken into separate, smaller nodes, combined with traversable relationships</span></span> </li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="35eaa-281"><strong>Példák</strong></span><span class="sxs-lookup"><span data-stu-id="35eaa-281"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="35eaa-282">Szervezeti diagramok</span><span class="sxs-lookup"><span data-stu-id="35eaa-282">Organization charts</span></span></li>
            <li><span data-ttu-id="35eaa-283">Közösségi diagramok</span><span class="sxs-lookup"><span data-stu-id="35eaa-283">Social graphs</span></span></li>
            <li><span data-ttu-id="35eaa-284">Csalások észlelése</span><span class="sxs-lookup"><span data-stu-id="35eaa-284">Fraud detection</span></span></li>
            <li><span data-ttu-id="35eaa-285">Elemzés</span><span class="sxs-lookup"><span data-stu-id="35eaa-285">Analytics</span></span></li>
            <li><span data-ttu-id="35eaa-286">Javaslati motorok</span><span class="sxs-lookup"><span data-stu-id="35eaa-286">Recommendation engines</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="column-family-databases"></a><span data-ttu-id="35eaa-287">Oszlopcsalád-adatbázisok</span><span class="sxs-lookup"><span data-stu-id="35eaa-287">Column-family databases</span></span>

<table>
<tr><td><span data-ttu-id="35eaa-288"><strong>Számítási feladat</strong></span><span class="sxs-lookup"><span data-stu-id="35eaa-288"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="35eaa-289">A legtöbb oszlopcsalád-adatbázis rendkívül gyorsan hajtja végre az írási műveleteket.</span><span class="sxs-lookup"><span data-stu-id="35eaa-289">Most column-family databases perform write operations extremely quickly.</span></span></li>
            <li><span data-ttu-id="35eaa-290">A frissítési és törlési műveletek ritkák.</span><span class="sxs-lookup"><span data-stu-id="35eaa-290">Update and delete operations are rare.</span></span></li>
            <li><span data-ttu-id="35eaa-291">Nagy teljesítmény és kis késleltetésű hozzáférés biztosítására tervezték.</span><span class="sxs-lookup"><span data-stu-id="35eaa-291">Designed to provide high throughput and low-latency access.</span></span></li>
            <li><span data-ttu-id="35eaa-292">Támogatja egy jóval nagyobb méretű rekordon belüli adott mezőkészletre irányuló egyszerű lekérdezési hozzáféréseket.</span><span class="sxs-lookup"><span data-stu-id="35eaa-292">Supports easy query access to a particular set of fields within a much larger record.</span></span></li>
            <li><span data-ttu-id="35eaa-293">Nagy mértékben skálázható.</span><span class="sxs-lookup"><span data-stu-id="35eaa-293">Massively scalable.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="35eaa-294"><strong>Adattípus</strong></span><span class="sxs-lookup"><span data-stu-id="35eaa-294"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="35eaa-295">Az adatok egy kulcsoszlopból és egy vagy több oszlopcsaládból álló táblákban tárolódnak.</span><span class="sxs-lookup"><span data-stu-id="35eaa-295">Data is stored in tables consisting of a key column and one or more column families.</span></span></li>
            <li><span data-ttu-id="35eaa-296">Az adott oszlopok az egyes sorokban eltérőek lehetnek.</span><span class="sxs-lookup"><span data-stu-id="35eaa-296">Specific columns can vary by individual rows.</span></span></li>
            <li><span data-ttu-id="35eaa-297">Az egyes cellák „get” és „put” parancsokkal érhetők el.</span><span class="sxs-lookup"><span data-stu-id="35eaa-297">Individual cells are accessed via get and put commands</span></span></li>
            <li><span data-ttu-id="35eaa-298">Egy vizsgálat paranccsal több sort ad vissza.</span><span class="sxs-lookup"><span data-stu-id="35eaa-298">Multiple rows are returned using a scan command.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="35eaa-299"><strong>Példák</strong></span><span class="sxs-lookup"><span data-stu-id="35eaa-299"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="35eaa-300">Javaslatok</span><span class="sxs-lookup"><span data-stu-id="35eaa-300">Recommendations</span></span></li>
            <li><span data-ttu-id="35eaa-301">Személyre szabás</span><span class="sxs-lookup"><span data-stu-id="35eaa-301">Personalization</span></span></li>
            <li><span data-ttu-id="35eaa-302">Érzékelői adatok</span><span class="sxs-lookup"><span data-stu-id="35eaa-302">Sensor data</span></span></li>
            <li><span data-ttu-id="35eaa-303">Telemetria</span><span class="sxs-lookup"><span data-stu-id="35eaa-303">Telemetry</span></span></li>
            <li><span data-ttu-id="35eaa-304">Üzenetkezelés</span><span class="sxs-lookup"><span data-stu-id="35eaa-304">Messaging</span></span></li>
            <li><span data-ttu-id="35eaa-305">Közösségi médiaelemzés</span><span class="sxs-lookup"><span data-stu-id="35eaa-305">Social media analytics</span></span></li>
            <li><span data-ttu-id="35eaa-306">Webes elemzés</span><span class="sxs-lookup"><span data-stu-id="35eaa-306">Web analytics</span></span></li>
            <li><span data-ttu-id="35eaa-307">Tevékenység figyelése</span><span class="sxs-lookup"><span data-stu-id="35eaa-307">Activity monitoring</span></span></li>
            <li><span data-ttu-id="35eaa-308">Időjárási és egyéb idősorozat-adatok</span><span class="sxs-lookup"><span data-stu-id="35eaa-308">Weather and other time-series data</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="search-engine-databases"></a><span data-ttu-id="35eaa-309">Keresőmotor-adatbázisok</span><span class="sxs-lookup"><span data-stu-id="35eaa-309">Search engine databases</span></span>

<table>
<tr><td><span data-ttu-id="35eaa-310"><strong>Számítási feladat</strong></span><span class="sxs-lookup"><span data-stu-id="35eaa-310"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="35eaa-311">Több forrásból és szolgáltatástól származó adatok indexelése.</span><span class="sxs-lookup"><span data-stu-id="35eaa-311">Indexing data from multiple sources and services.</span></span></li>
            <li><span data-ttu-id="35eaa-312">A lekérdezések ad-hoc jellegűek, és összetettek lehetnek.</span><span class="sxs-lookup"><span data-stu-id="35eaa-312">Queries are ad-hoc and can be complex.</span></span></li>
            <li><span data-ttu-id="35eaa-313">Összesítést igényel.</span><span class="sxs-lookup"><span data-stu-id="35eaa-313">Requires aggregation.</span></span></li>
            <li><span data-ttu-id="35eaa-314">Teljes szöveges keresést igényel.</span><span class="sxs-lookup"><span data-stu-id="35eaa-314">Full text search is required.</span></span></li>
            <li><span data-ttu-id="35eaa-315">Ad-hoc jellegű önkiszolgáló lekérdezéseket igényel.</span><span class="sxs-lookup"><span data-stu-id="35eaa-315">Ad hoc self-service query is required.</span></span></li>
            <li><span data-ttu-id="35eaa-316">Minden mező indexelésével végzett adatelemzésre van szükség.</span><span class="sxs-lookup"><span data-stu-id="35eaa-316">Data analysis with index on all fields is required.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="35eaa-317"><strong>Adattípus</strong></span><span class="sxs-lookup"><span data-stu-id="35eaa-317"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="35eaa-318">Részben strukturált, vagy strukturálatlan</span><span class="sxs-lookup"><span data-stu-id="35eaa-318">Semi-structured or unstructured</span></span></li>
            <li><span data-ttu-id="35eaa-319">Szöveg</span><span class="sxs-lookup"><span data-stu-id="35eaa-319">Text</span></span></li>
            <li><span data-ttu-id="35eaa-320">Strukturált adatokra hivatkozó szöveg</span><span class="sxs-lookup"><span data-stu-id="35eaa-320">Text with reference to structured data</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="35eaa-321"><strong>Példák</strong></span><span class="sxs-lookup"><span data-stu-id="35eaa-321"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="35eaa-322">Termékkatalógusok</span><span class="sxs-lookup"><span data-stu-id="35eaa-322">Product catalogs</span></span></li>
            <li><span data-ttu-id="35eaa-323">Keresés webhelyen</span><span class="sxs-lookup"><span data-stu-id="35eaa-323">Site search</span></span></li>
            <li><span data-ttu-id="35eaa-324">Naplózás</span><span class="sxs-lookup"><span data-stu-id="35eaa-324">Logging</span></span></li>
            <li><span data-ttu-id="35eaa-325">Elemzés</span><span class="sxs-lookup"><span data-stu-id="35eaa-325">Analytics</span></span></li>
            <li><span data-ttu-id="35eaa-326">Vásárlási webhelyek</span><span class="sxs-lookup"><span data-stu-id="35eaa-326">Shopping sites</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="data-warehouse"></a><span data-ttu-id="35eaa-327">Adattárház</span><span class="sxs-lookup"><span data-stu-id="35eaa-327">Data warehouse</span></span>

<table>
<tr><td><span data-ttu-id="35eaa-328"><strong>Számítási feladat</strong></span><span class="sxs-lookup"><span data-stu-id="35eaa-328"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="35eaa-329">Adatelemzés</span><span class="sxs-lookup"><span data-stu-id="35eaa-329">Data analytics</span></span></li>
            <li><span data-ttu-id="35eaa-330">Enterprise BI</span><span class="sxs-lookup"><span data-stu-id="35eaa-330">Enterprise BI</span></span>   </li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="35eaa-331"><strong>Adattípus</strong></span><span class="sxs-lookup"><span data-stu-id="35eaa-331"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="35eaa-332">Több forrásból származó előzményadatok.</span><span class="sxs-lookup"><span data-stu-id="35eaa-332">Historical data from multiple sources.</span></span></li>
            <li><span data-ttu-id="35eaa-333">Általában a denormalizált egy &quot;csillag&quot; vagy &quot;snowflake&quot; séma-, tény-és dimenziótáblák álló.</span><span class="sxs-lookup"><span data-stu-id="35eaa-333">Usually denormalized in a &quot;star&quot; or &quot;snowflake&quot; schema, consisting of fact and dimension tables.</span></span></li>
            <li><span data-ttu-id="35eaa-334">Általában ütemezés szerint töltődik fel új adatokkal.</span><span class="sxs-lookup"><span data-stu-id="35eaa-334">Usually loaded with new data on a scheduled basis.</span></span></li>
            <li><span data-ttu-id="35eaa-335">A dimenziótáblák gyakran egy entitás több korábbi verzióját is tartalmazzák, amit <em>lassan változó dimenziónak</em> nevezünk.</span><span class="sxs-lookup"><span data-stu-id="35eaa-335">Dimension tables often include multiple historic versions of an entity, referred to as a <em>slowly changing dimension</em>.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="35eaa-336"><strong>Példák</strong></span><span class="sxs-lookup"><span data-stu-id="35eaa-336"><strong>Examples</strong></span></span></td>
    <td><span data-ttu-id="35eaa-337">Egy vállalati adattárház, amely elemzési modellek, jelentések és irányítópultok számára biztosít adatokat.</span><span class="sxs-lookup"><span data-stu-id="35eaa-337">An enterprise data warehouse that provides data for analytical models, reports, and dashboards.</span></span>
    </td>
</tr>
</table>


## <a name="time-series-databases"></a><span data-ttu-id="35eaa-338">Idősorozat-adatbázisok</span><span class="sxs-lookup"><span data-stu-id="35eaa-338">Time series databases</span></span>

<table>
<tr><td><span data-ttu-id="35eaa-339"><strong>Számítási feladat</strong></span><span class="sxs-lookup"><span data-stu-id="35eaa-339"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="35eaa-340">A műveletek túlnyomó részét (95–99%) írási műveletek adják.</span><span class="sxs-lookup"><span data-stu-id="35eaa-340">An overwhelmingly proportion of operations (95-99%) are writes.</span></span></li>
            <li><span data-ttu-id="35eaa-341">A rekordok hozzáfűzése általában időrend szerint történik.</span><span class="sxs-lookup"><span data-stu-id="35eaa-341">Records are generally appended sequentially in time order.</span></span></li>
            <li><span data-ttu-id="35eaa-342">A frissítések ritkák.</span><span class="sxs-lookup"><span data-stu-id="35eaa-342">Updates are rare.</span></span></li>
            <li><span data-ttu-id="35eaa-343">A törlések tömegesen fordulnak elő, céljaik általában egybefüggő blokkok vagy rekordok.</span><span class="sxs-lookup"><span data-stu-id="35eaa-343">Deletes occur in bulk, and are made to contiguous blocks or records.</span></span></li>
            <li><span data-ttu-id="35eaa-344">Az olvasási kérések lehetnek az elérhető memóriánál nagyobb méretűek.</span><span class="sxs-lookup"><span data-stu-id="35eaa-344">Read requests can be larger than available memory.</span></span></li>
            <li><span data-ttu-id="35eaa-345">Az&#39;s gyakori, hogy mi történjen, párhuzamosan több olvasása.</span><span class="sxs-lookup"><span data-stu-id="35eaa-345">It&#39;s common for multiple reads to occur simultaneously.</span></span></li>
            <li><span data-ttu-id="35eaa-346">A rendszer az adatokat szakaszosan olvassa be időrend szerint növekvő vagy csökkenő sorrendben.</span><span class="sxs-lookup"><span data-stu-id="35eaa-346">Data is read sequentially in either ascending or descending time order.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="35eaa-347"><strong>Adattípus</strong></span><span class="sxs-lookup"><span data-stu-id="35eaa-347"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="35eaa-348">Az elsődleges kulcsként és rendezési mechanizmusként használt időbélyegző.</span><span class="sxs-lookup"><span data-stu-id="35eaa-348">A time stamp that is used as the primary key and sorting mechanism.</span></span></li>
            <li><span data-ttu-id="35eaa-349">A bejegyzésből származó mérések vagy a bejegyzés által képviselt elemek leírása.</span><span class="sxs-lookup"><span data-stu-id="35eaa-349">Measurements from the entry or descriptions of what the entry represents.</span></span></li>
            <li><span data-ttu-id="35eaa-350">A bejegyzés típusára, eredetére és egyéb információira vonatkozó további adatokat meghatározó címkék.</span><span class="sxs-lookup"><span data-stu-id="35eaa-350">Tags that define additional information about the type, origin, and other information about the entry.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="35eaa-351"><strong>Példák</strong></span><span class="sxs-lookup"><span data-stu-id="35eaa-351"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="35eaa-352">Monitorozás és eseménytelemetria.</span><span class="sxs-lookup"><span data-stu-id="35eaa-352">Monitoring and event telemetry.</span></span></li>
            <li><span data-ttu-id="35eaa-353">Érzékelő- és egyéb IoT-adatok.</span><span class="sxs-lookup"><span data-stu-id="35eaa-353">Sensor or other IoT data.</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="object-storage"></a><span data-ttu-id="35eaa-354">Objektumtár</span><span class="sxs-lookup"><span data-stu-id="35eaa-354">Object storage</span></span>

<table>
<tr><td><span data-ttu-id="35eaa-355"><strong>Számítási feladat</strong></span><span class="sxs-lookup"><span data-stu-id="35eaa-355"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="35eaa-356">Kulcs azonosítja.</span><span class="sxs-lookup"><span data-stu-id="35eaa-356">Identified by key.</span></span></li>
            <li><span data-ttu-id="35eaa-357">Az objektumok nyilvánosan vagy privát módon is hozzáférhetők.</span><span class="sxs-lookup"><span data-stu-id="35eaa-357">Objects may be publicly or privately accessible.</span></span></li>
            <li><span data-ttu-id="35eaa-358">A tartalom általában egy táblázat, kép- vagy videofájl vagy hasonló adategység.</span><span class="sxs-lookup"><span data-stu-id="35eaa-358">Content is typically an asset such as a spreadsheet, image, or video file.</span></span></li>
            <li><span data-ttu-id="35eaa-359">A tartalomnak tartósnak (állandónak) kell lennie, és bármely alkalmazásszinten vagy virtuális gépen külső tartalomnak kell lennie.</span><span class="sxs-lookup"><span data-stu-id="35eaa-359">Content must be durable (persistent), and external to any application tier or virtual machine.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="35eaa-360"><strong>Adattípus</strong></span><span class="sxs-lookup"><span data-stu-id="35eaa-360"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="35eaa-361">Az adatok mérete nagy.</span><span class="sxs-lookup"><span data-stu-id="35eaa-361">Data size is large.</span></span></li>
            <li><span data-ttu-id="35eaa-362">Blobadatok.</span><span class="sxs-lookup"><span data-stu-id="35eaa-362">Blob data.</span></span></li>
            <li><span data-ttu-id="35eaa-363">Az érték nem átlátszó.</span><span class="sxs-lookup"><span data-stu-id="35eaa-363">Value is opaque.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="35eaa-364"><strong>Példák</strong></span><span class="sxs-lookup"><span data-stu-id="35eaa-364"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="35eaa-365">Képek, videók, Office-dokumentumok, PDF-fájlok</span><span class="sxs-lookup"><span data-stu-id="35eaa-365">Images, videos, office documents, PDFs</span></span></li>
            <li><span data-ttu-id="35eaa-366">CSS, szkriptek, CSV</span><span class="sxs-lookup"><span data-stu-id="35eaa-366">CSS, Scripts, CSV</span></span></li>
            <li><span data-ttu-id="35eaa-367">Statikus HTML, JSON</span><span class="sxs-lookup"><span data-stu-id="35eaa-367">Static HTML, JSON</span></span></li>
            <li><span data-ttu-id="35eaa-368">Napló- és vizsgálati fájlok</span><span class="sxs-lookup"><span data-stu-id="35eaa-368">Log and audit files</span></span></li>
            <li><span data-ttu-id="35eaa-369">Adatbázisok biztonsági mentése</span><span class="sxs-lookup"><span data-stu-id="35eaa-369">Database backups</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="shared-files"></a><span data-ttu-id="35eaa-370">Megosztott fájlok</span><span class="sxs-lookup"><span data-stu-id="35eaa-370">Shared files</span></span>

<table>
<tr><td><span data-ttu-id="35eaa-371"><strong>Számítási feladat</strong></span><span class="sxs-lookup"><span data-stu-id="35eaa-371"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="35eaa-372">Áttelepítés meglévő alkalmazásokból, amelyek kommunikálnak a fájlrendszerrel.</span><span class="sxs-lookup"><span data-stu-id="35eaa-372">Migration from existing apps that interact with the file system.</span></span></li>
            <li><span data-ttu-id="35eaa-373">SMB-kapcsolatot igényel.</span><span class="sxs-lookup"><span data-stu-id="35eaa-373">Requires SMB interface.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="35eaa-374"><strong>Adattípus</strong></span><span class="sxs-lookup"><span data-stu-id="35eaa-374"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="35eaa-375">Egy hierarchikus mappakészlet fájljai.</span><span class="sxs-lookup"><span data-stu-id="35eaa-375">Files in a hierarchical set of folders.</span></span></li>
            <li><span data-ttu-id="35eaa-376">Szabványos I/O-kódtárakkal elérhető.</span><span class="sxs-lookup"><span data-stu-id="35eaa-376">Accessible with standard I/O libraries.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="35eaa-377"><strong>Példák</strong></span><span class="sxs-lookup"><span data-stu-id="35eaa-377"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="35eaa-378">Örökölt fájlok</span><span class="sxs-lookup"><span data-stu-id="35eaa-378">Legacy files</span></span></li>
            <li><span data-ttu-id="35eaa-379">A megosztott tartalom bizonyos számú virtuális gép vagy alkalmazáspéldány számára elérhető</span><span class="sxs-lookup"><span data-stu-id="35eaa-379">Shared content accessible among a number of VMs or app instances</span></span></li>
        </ul>
    </td>
</tr>
</table>
