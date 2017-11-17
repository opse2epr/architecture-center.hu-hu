---
title: "Adattároló kiválasztásának szempontjai"
description: "Az Azure compute beállítások áttekintése"
author: MikeWasson
ms.openlocfilehash: 7fb75cd334438c5b985fa04ad8afe3236f2391f8
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="criteria-for-choosing-a-data-store"></a><span data-ttu-id="53764-103">Adattároló kiválasztásának szempontjai</span><span class="sxs-lookup"><span data-stu-id="53764-103">Criteria for choosing a data store</span></span>

<span data-ttu-id="53764-104">Azure támogatja számos különböző típusú adatok tárolási megoldások, mindegyik különböző szolgáltatásokat és képességeket biztosít.</span><span class="sxs-lookup"><span data-stu-id="53764-104">Azure supports many types of data storage solutions, each providing different features and capabilities.</span></span> <span data-ttu-id="53764-105">Ez a cikk ismerteti a összehasonlítási feltétel kiértékelésekor adattárat kell használnia.</span><span class="sxs-lookup"><span data-stu-id="53764-105">This article describes the comparison criteria you should use when evaluating a data store.</span></span> <span data-ttu-id="53764-106">A cél, hogy a segítségével meghatározhatja, hogy mely adattípusok tároló megfelel a megoldás követelményei.</span><span class="sxs-lookup"><span data-stu-id="53764-106">The goal is to help you determine which data storage types can meet your solution's requirements.</span></span>

## <a name="general-considerations"></a><span data-ttu-id="53764-107">Általános megfontolások</span><span class="sxs-lookup"><span data-stu-id="53764-107">General Considerations</span></span>

<span data-ttu-id="53764-108">Az összehasonlítás elindításához gyűjtse össze a következő információkat, mint az adatokkal kapcsolatos igények megfelelően.</span><span class="sxs-lookup"><span data-stu-id="53764-108">To start your comparison, gather as much of the following information as you can about your data needs.</span></span> <span data-ttu-id="53764-109">Ez az információ segít meghatározni, mely adattípusok tárolási igényeinek.</span><span class="sxs-lookup"><span data-stu-id="53764-109">This information will help you to determine which data storage types will meet your needs.</span></span>

### <a name="functional-requirements"></a><span data-ttu-id="53764-110">Működési követelmények</span><span class="sxs-lookup"><span data-stu-id="53764-110">Functional requirements</span></span>

- <span data-ttu-id="53764-111">**Az adatformátum**.</span><span class="sxs-lookup"><span data-stu-id="53764-111">**Data format**.</span></span> <span data-ttu-id="53764-112">Milyen típusú adatok vannak, tárolni szándékozik?</span><span class="sxs-lookup"><span data-stu-id="53764-112">What type of data are you intending to store?</span></span> <span data-ttu-id="53764-113">Használt típusok a következők: tranzakciós adatok, JSON objektumok, telemetriai, keresési indexek vagy egybesimított fájlokba.</span><span class="sxs-lookup"><span data-stu-id="53764-113">Common types include transactional data, JSON objects, telemetry, search indexes, or flat files.</span></span>
- <span data-ttu-id="53764-114">**Adatok mérete**.</span><span class="sxs-lookup"><span data-stu-id="53764-114">**Data size**.</span></span> <span data-ttu-id="53764-115">Hogyan nagy kell tárolni entitások vannak?</span><span class="sxs-lookup"><span data-stu-id="53764-115">How large are the entities you need to store?</span></span> <span data-ttu-id="53764-116">Ezeket az entitásokat kell egyetlen dokumentum fenn kell tartani, vagy azok oszthatók több dokumentumok, táblák, gyűjtemények, és így tovább?</span><span class="sxs-lookup"><span data-stu-id="53764-116">Will these entities need to be maintained as a single document, or can they be split across multiple documents, tables, collections, and so forth?</span></span>
- <span data-ttu-id="53764-117">**Méretezés és struktúra**.</span><span class="sxs-lookup"><span data-stu-id="53764-117">**Scale and structure**.</span></span> <span data-ttu-id="53764-118">Mi az az általános tárolókapacitáson van szüksége?</span><span class="sxs-lookup"><span data-stu-id="53764-118">What is the overall amount of storage capacity you need?</span></span> <span data-ttu-id="53764-119">Hajtsa végre, amelyek várhatóan az adatok particionálása?</span><span class="sxs-lookup"><span data-stu-id="53764-119">Do you anticipate partitioning your data?</span></span> 
- <span data-ttu-id="53764-120">**Adatok kapcsolatok**.</span><span class="sxs-lookup"><span data-stu-id="53764-120">**Data relationships**.</span></span> <span data-ttu-id="53764-121">Az adatok kell egy-a-többhöz vagy a több-több kapcsolat támogatásához?</span><span class="sxs-lookup"><span data-stu-id="53764-121">Will your data need to support one-to-many or many-to-many relationships?</span></span> <span data-ttu-id="53764-122">A kapcsolatok magukat az adatokat egyik fontos része?</span><span class="sxs-lookup"><span data-stu-id="53764-122">Are relationships themselves an important part of the data?</span></span> <span data-ttu-id="53764-123">Szükség lesz való csatlakozásra, és ugyanahhoz az adatkészlethez belül, vagy a külső adatkészletek adatait egyébként felhasználó?</span><span class="sxs-lookup"><span data-stu-id="53764-123">Will you need to join or otherwise combine data from within the same dataset, or from external datasets?</span></span> 
- <span data-ttu-id="53764-124">**Konzisztencia-modell**.</span><span class="sxs-lookup"><span data-stu-id="53764-124">**Consistency model**.</span></span> <span data-ttu-id="53764-125">Mennyire fontos a azt egy csomópontban jelennek meg más csomópontokat, mielőtt további módosításokat hajthat végrehajtott frissítések?</span><span class="sxs-lookup"><span data-stu-id="53764-125">How important is it for updates made in one node to appear in other nodes, before further changes can be made?</span></span> <span data-ttu-id="53764-126">Fogadhat végleges konzisztencia?</span><span class="sxs-lookup"><span data-stu-id="53764-126">Can you accept eventual consistency?</span></span> <span data-ttu-id="53764-127">Szüksége van az egyes tranzakciókra vonatkozóan ACID garanciák?</span><span class="sxs-lookup"><span data-stu-id="53764-127">Do you need ACID guarantees for transactions?</span></span>
- <span data-ttu-id="53764-128">**Séma rugalmasságát**.</span><span class="sxs-lookup"><span data-stu-id="53764-128">**Schema flexibility**.</span></span> <span data-ttu-id="53764-129">Milyen sémák rendszer alkalmaz az adatok?</span><span class="sxs-lookup"><span data-stu-id="53764-129">What kind of schemas will you apply to your data?</span></span> <span data-ttu-id="53764-130">Egy rögzített sémájába, a séma módosításkor megközelítés vagy a séma-a-olvasási megközelítést fog használni?</span><span class="sxs-lookup"><span data-stu-id="53764-130">Will you use a fixed schema, a schema-on-write approach, or a schema-on-read approach?</span></span>
- <span data-ttu-id="53764-131">**Párhuzamossági**.</span><span class="sxs-lookup"><span data-stu-id="53764-131">**Concurrency**.</span></span> <span data-ttu-id="53764-132">Milyen párhuzamossági mechanizmus kívánja használja, ha a frissítés és az adatok szinkronizálását?</span><span class="sxs-lookup"><span data-stu-id="53764-132">What kind of concurrency mechanism do you want to use when updating and synchronizing data?</span></span> <span data-ttu-id="53764-133">Az alkalmazás akkor fogja elvégezni, amely potenciálisan ütközhetnek számos frissítést.</span><span class="sxs-lookup"><span data-stu-id="53764-133">Will the application perform many updates that could potentially conflict.</span></span> <span data-ttu-id="53764-134">Ha igen, előfordulhat, hogy a rekord zárolás és pesszimista feldolgozási vezérlő igénylő.</span><span class="sxs-lookup"><span data-stu-id="53764-134">If so, you may requiring record locking and pessimistic concurrency control.</span></span> <span data-ttu-id="53764-135">Másik lehetőségként támogathat egyidejű hozzáférések optimista vezérlők?</span><span class="sxs-lookup"><span data-stu-id="53764-135">Alternatively, can you support optimistic concurrency controls?</span></span> <span data-ttu-id="53764-136">Ha igen, egyszerű időbélyeg-alapú feldolgozási vezérlő elég, vagy van szüksége több verzió párhuzamossági vezérlő hozzáadott új funkció?</span><span class="sxs-lookup"><span data-stu-id="53764-136">If so, is simple timestamp-based concurrency control enough, or do you need the added functionality of multi-version concurrency control?</span></span>
- <span data-ttu-id="53764-137">**Adatátvitel**.</span><span class="sxs-lookup"><span data-stu-id="53764-137">**Data movement**.</span></span> <span data-ttu-id="53764-138">A megoldás kell áthelyezni az adatokat más tárolók vagy adatraktárak ETL feladatok elvégzéséhez?</span><span class="sxs-lookup"><span data-stu-id="53764-138">Will your solution need to perform ETL tasks to move data to other stores or data warehouses?</span></span>
- <span data-ttu-id="53764-139">**Adatok életciklus**.</span><span class="sxs-lookup"><span data-stu-id="53764-139">**Data lifecycle**.</span></span> <span data-ttu-id="53764-140">Az adatok írási van – egyszer, olvassa el a többhöz?</span><span class="sxs-lookup"><span data-stu-id="53764-140">Is the data write-once, read-many?</span></span> <span data-ttu-id="53764-141">Az mozgathatók ritkán vagy a cold storage?</span><span class="sxs-lookup"><span data-stu-id="53764-141">Can it be moved into cool or cold storage?</span></span>
- <span data-ttu-id="53764-142">**Egyéb támogatott szolgáltatás**.</span><span class="sxs-lookup"><span data-stu-id="53764-142">**Other supported features**.</span></span> <span data-ttu-id="53764-143">Nem kell semmilyen más szolgáltatást, például a sémaérvényesítés, összesítési, indexelő, teljes szöveges keresés, MapReduce, vagy más lekérdezési képességeket?</span><span class="sxs-lookup"><span data-stu-id="53764-143">Do you need any other specific features, such as schema validation, aggregation, indexing, full-text search, MapReduce, or other query capabilities?</span></span>

### <a name="non-functional-requirements"></a><span data-ttu-id="53764-144">A nem működési követelmények</span><span class="sxs-lookup"><span data-stu-id="53764-144">Non-functional requirements</span></span>

- <span data-ttu-id="53764-145">**Teljesítmény és méretezhetőség**.</span><span class="sxs-lookup"><span data-stu-id="53764-145">**Performance and scalability**.</span></span> <span data-ttu-id="53764-146">Mik a adatok teljesítményre vonatkozó követelmények?</span><span class="sxs-lookup"><span data-stu-id="53764-146">What are your data performance requirements?</span></span> <span data-ttu-id="53764-147">Konkrét követelmények adatforgalmi adatfeldolgozást díjakat és az adatok feldolgozási sebességet van?</span><span class="sxs-lookup"><span data-stu-id="53764-147">Do you have specific requirements for data ingestion rates and data processing rates?</span></span> <span data-ttu-id="53764-148">Mik a elfogadható válaszidejét kérdez le, és az adatok egyszer okozhatnak összesítése?</span><span class="sxs-lookup"><span data-stu-id="53764-148">What are the acceptable response times for querying and aggregation of data once ingested?</span></span> <span data-ttu-id="53764-149">Hogyan nagy szükség van-e az adatokat tároló méretezést kívánó?</span><span class="sxs-lookup"><span data-stu-id="53764-149">How large will you need the data store to scale up?</span></span> <span data-ttu-id="53764-150">A számítási feladatok van, további műveleteket olvasási vagy írási műveleteket?</span><span class="sxs-lookup"><span data-stu-id="53764-150">Is your workload more read-heavy or write-heavy?</span></span>
- <span data-ttu-id="53764-151">**Megbízhatóság**.</span><span class="sxs-lookup"><span data-stu-id="53764-151">**Reliability**.</span></span> <span data-ttu-id="53764-152">Milyen általános SLA szüksége támogatásához?</span><span class="sxs-lookup"><span data-stu-id="53764-152">What overall SLA do you need to support?</span></span> <span data-ttu-id="53764-153">Milyen szintű hibatűrést van szüksége az adatfelhasználók számára?</span><span class="sxs-lookup"><span data-stu-id="53764-153">What level of fault-tolerance do you need to provide for data consumers?</span></span> <span data-ttu-id="53764-154">Milyen típusú biztonsági mentési és visszaállítási képességeket van szüksége?</span><span class="sxs-lookup"><span data-stu-id="53764-154">What kind of backup and restore capabilities do you need?</span></span> 
- <span data-ttu-id="53764-155">**Replikációs**.</span><span class="sxs-lookup"><span data-stu-id="53764-155">**Replication**.</span></span> <span data-ttu-id="53764-156">Több replikák és régiók közötti elosztását kell az adatokat?</span><span class="sxs-lookup"><span data-stu-id="53764-156">Will your data need to be distributed among multiple replicas or regions?</span></span> <span data-ttu-id="53764-157">Milyen típusú adatok replikációs képességek szükség van-e?</span><span class="sxs-lookup"><span data-stu-id="53764-157">What kind of data replication capabilities do you require?</span></span> 
- <span data-ttu-id="53764-158">**Korlátok**.</span><span class="sxs-lookup"><span data-stu-id="53764-158">**Limits**.</span></span> <span data-ttu-id="53764-159">Egy adott adattár határain támogatja a követelmények a méretezés, a kapcsolatokat, és az átvitel száma?</span><span class="sxs-lookup"><span data-stu-id="53764-159">Will the limits of a particular data store support your requirements for scale, number of connections, and throughput?</span></span> 

### <a name="management-and-cost"></a><span data-ttu-id="53764-160">Felügyeleti és a költséghatékonyság</span><span class="sxs-lookup"><span data-stu-id="53764-160">Management and cost</span></span>

- <span data-ttu-id="53764-161">**A felügyelt**.</span><span class="sxs-lookup"><span data-stu-id="53764-161">**Managed service**.</span></span> <span data-ttu-id="53764-162">Ha lehetséges, egy felügyelt adatok szolgáltatását használja, ha szüksége van, amely csak egy üzemeltetett IaaS-tárolóban található meghatározott jogosultságokat.</span><span class="sxs-lookup"><span data-stu-id="53764-162">When possible, use a managed data service, unless you require specific capabilities that can only be found in an IaaS-hosted data store.</span></span>
- <span data-ttu-id="53764-163">**Régiónkénti elérhetőség**.</span><span class="sxs-lookup"><span data-stu-id="53764-163">**Region availability**.</span></span> <span data-ttu-id="53764-164">Felügyelt szolgáltatások esetén érhető el a szolgáltatás az összes Azure-régiók?</span><span class="sxs-lookup"><span data-stu-id="53764-164">For managed services, is the service available in all Azure regions?</span></span> <span data-ttu-id="53764-165">Nem a megoldás kell bizonyos Azure-régiók-ban üzemeltetett?</span><span class="sxs-lookup"><span data-stu-id="53764-165">Does your solution need to be hosted in certain Azure regions?</span></span>
- <span data-ttu-id="53764-166">**Hordozhatósága**.</span><span class="sxs-lookup"><span data-stu-id="53764-166">**Portability**.</span></span> <span data-ttu-id="53764-167">Az adatok kell áttelepíteni a helyszíni, külső adatközpontok vagy egyéb felhőalapú üzemeltetési környezeteket?</span><span class="sxs-lookup"><span data-stu-id="53764-167">Will your data need to migrated to on-premises, external datacenters, or other cloud hosting environments?</span></span>
- <span data-ttu-id="53764-168">**Licencelési**.</span><span class="sxs-lookup"><span data-stu-id="53764-168">**Licensing**.</span></span> <span data-ttu-id="53764-169">Rendelkezik egy védett OSS licenc típusa és előnyben?</span><span class="sxs-lookup"><span data-stu-id="53764-169">Do you have a preference of a proprietary versus OSS license type?</span></span> <span data-ttu-id="53764-170">Vannak-e egyéb külső korlátozások licenc típusa használhatja?</span><span class="sxs-lookup"><span data-stu-id="53764-170">Are there any other external restrictions on what type of license you can use?</span></span>
- <span data-ttu-id="53764-171">**A teljes költség**.</span><span class="sxs-lookup"><span data-stu-id="53764-171">**Overall cost**.</span></span> <span data-ttu-id="53764-172">Mi az a teljes költség szempontjából, a szolgáltatás belül a megoldás használatával?</span><span class="sxs-lookup"><span data-stu-id="53764-172">What is the overall cost of using the service within your solution?</span></span> <span data-ttu-id="53764-173">Hány példányban kell futtatni, saját hasznos üzemidő és átviteli igényeinek támogatására?</span><span class="sxs-lookup"><span data-stu-id="53764-173">How many instances will need to run, to support your uptime and throughput requirements?</span></span> <span data-ttu-id="53764-174">Vegye figyelembe a számítási műveletek költségét.</span><span class="sxs-lookup"><span data-stu-id="53764-174">Consider operations costs in this calculation.</span></span> <span data-ttu-id="53764-175">Felügyelt szolgáltatásokat inkább akkor működési költséghatékony.</span><span class="sxs-lookup"><span data-stu-id="53764-175">One reason to prefer managed services is the reduced operational cost.</span></span>
- <span data-ttu-id="53764-176">**Költség hatékonyságának**.</span><span class="sxs-lookup"><span data-stu-id="53764-176">**Cost effectiveness**.</span></span> <span data-ttu-id="53764-177">Képes az adatok tárolására, hatékonyan további költség partícióazonosító?</span><span class="sxs-lookup"><span data-stu-id="53764-177">Can you partition your data, to store it more cost effectively?</span></span> <span data-ttu-id="53764-178">Például lehet egy drága relációs adatbázisból nagy objektumok helyez át egy objektum áruházban?</span><span class="sxs-lookup"><span data-stu-id="53764-178">For example, can you move large objects out of an expensive relational database into an object store?</span></span>

### <a name="security"></a><span data-ttu-id="53764-179">Biztonság</span><span class="sxs-lookup"><span data-stu-id="53764-179">Security</span></span>

- <span data-ttu-id="53764-180">**Biztonság**.</span><span class="sxs-lookup"><span data-stu-id="53764-180">**Security**.</span></span> <span data-ttu-id="53764-181">Milyen típusú titkosítást igényelnek?</span><span class="sxs-lookup"><span data-stu-id="53764-181">What type of encryption do you require?</span></span> <span data-ttu-id="53764-182">Kell-e titkosítását?</span><span class="sxs-lookup"><span data-stu-id="53764-182">Do you need encryption at rest?</span></span> <span data-ttu-id="53764-183">Milyen hitelesítési módszert szeretné használni az adatokhoz történő kapcsolódáshoz?</span><span class="sxs-lookup"><span data-stu-id="53764-183">What authentication mechanism do you want to use to connect to your data?</span></span>
- <span data-ttu-id="53764-184">**Naplózás**.</span><span class="sxs-lookup"><span data-stu-id="53764-184">**Auditing**.</span></span> <span data-ttu-id="53764-185">Milyen típusú naplózást szeretne létrehozni?</span><span class="sxs-lookup"><span data-stu-id="53764-185">What kind of audit log do you need to generate?</span></span>
- <span data-ttu-id="53764-186">**Hálózati követelmények**.</span><span class="sxs-lookup"><span data-stu-id="53764-186">**Networking requirements**.</span></span> <span data-ttu-id="53764-187">Szüksége korlátozhatja, vagy ellenkező esetben az adatok hozzáférésének más hálózati erőforrások kezelése?</span><span class="sxs-lookup"><span data-stu-id="53764-187">Do you need to restrict or otherwise manage access to your data from other network resources?</span></span> <span data-ttu-id="53764-188">Nem szükséges adatokat kell csak érhetők el az Azure-környezeten belül?</span><span class="sxs-lookup"><span data-stu-id="53764-188">Does data need to be accessible only from inside the Azure environment?</span></span> <span data-ttu-id="53764-189">Nem a szükséges adatokat az adott IP-címek és alhálózatok érhető el?</span><span class="sxs-lookup"><span data-stu-id="53764-189">Does the data need to be accessible from specific IP addresses or subnets?</span></span> <span data-ttu-id="53764-190">Nem azt kell ki alkalmazásaiban vagy az üzemeltetett szolgáltatások a helyszíni vagy más külső adatközpontokban elérhető?</span><span class="sxs-lookup"><span data-stu-id="53764-190">Does it need to be accessible from applications or services hosted on-premises or in other external datacenters?</span></span>

### <a name="devops"></a><span data-ttu-id="53764-191">DevOps</span><span class="sxs-lookup"><span data-stu-id="53764-191">DevOps</span></span>

- <span data-ttu-id="53764-192">**Szakértelem set**.</span><span class="sxs-lookup"><span data-stu-id="53764-192">**Skill set**.</span></span> <span data-ttu-id="53764-193">Vannak-e bizonyos programozási nyelvek, operációs rendszerek és egyéb technológia, hogy a csoport az különösen adept?</span><span class="sxs-lookup"><span data-stu-id="53764-193">Are there particular programming languages, operating systems, or other technology that your team is particularly adept at using?</span></span> <span data-ttu-id="53764-194">Vannak-e más, a csapat dolgozhat nehéz lenne?</span><span class="sxs-lookup"><span data-stu-id="53764-194">Are there others that would be difficult for your team to work with?</span></span>
- <span data-ttu-id="53764-195">**Ügyfelek** helyes ügyfél támogatása a fejlesztési nyelveket van?</span><span class="sxs-lookup"><span data-stu-id="53764-195">**Clients** Is there good client support for your development languages?</span></span>

<span data-ttu-id="53764-196">A következő szakaszok különböző adatokat tároló modellek munkaterhelés-profilt, az adattípusokat és példák az alkalmazási helyzetekre összehasonlítása.</span><span class="sxs-lookup"><span data-stu-id="53764-196">The following sections compare various data store models in terms of workload profile, data types, and example use cases.</span></span>

## <a name="relational-database-management-systems-rdbms"></a><span data-ttu-id="53764-197">Relációs adatbázis-kezelő rendszerek (RDBMS)</span><span class="sxs-lookup"><span data-stu-id="53764-197">Relational database management systems (RDBMS)</span></span>

<table>
<tr><td><span data-ttu-id="53764-198">**Számítási feladat**</span><span class="sxs-lookup"><span data-stu-id="53764-198">**Workload**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="53764-199">Mindkét létrehozása új rekordok és adatok frissítése rendszeresen fordulhat elő.</span><span class="sxs-lookup"><span data-stu-id="53764-199">Both the creation of new records and updates to existing data happen regularly.</span></span></li>
            <li><span data-ttu-id="53764-200">Több műveletet kell egy tranzakción belül kell végrehajtani.</span><span class="sxs-lookup"><span data-stu-id="53764-200">Multiple operations have to be completed in a single transaction.</span></span></li>
            <li><span data-ttu-id="53764-201">Az aggregátumfüggvényeket típusát végrehajtásához szükséges.</span><span class="sxs-lookup"><span data-stu-id="53764-201">Requires aggregation functions to perform cross-tabulation.</span></span></li>
            <li><span data-ttu-id="53764-202">Jelentéskészítő eszközök erős integrálva szükség.</span><span class="sxs-lookup"><span data-stu-id="53764-202">Strong integration with reporting tools is required.</span></span></li>
            <li><span data-ttu-id="53764-203">Kapcsolatok használatával adatbázis korlátozások érvényesek lesznek.</span><span class="sxs-lookup"><span data-stu-id="53764-203">Relationships are enforced using database constraints.</span></span></li>
            <li><span data-ttu-id="53764-204">Indexek használt lekérdezési teljesítmény optimalizálása érdekében.</span><span class="sxs-lookup"><span data-stu-id="53764-204">Indexes are used to optimize query performance.</span></span></li>
            <li><span data-ttu-id="53764-205">Lehetővé teszi, hogy az adatok meghatározott fájlcsoportokat a hozzáférést.</span><span class="sxs-lookup"><span data-stu-id="53764-205">Allows access to specific subsets of data.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="53764-206">**Adattípus**</span><span class="sxs-lookup"><span data-stu-id="53764-206">**Data type**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="53764-207">Adatok magas normalizált.</span><span class="sxs-lookup"><span data-stu-id="53764-207">Data is highly normalized.</span></span></li>
            <li><span data-ttu-id="53764-208">Adatbázis-sémák szükségesek, de lépnek érvénybe.</span><span class="sxs-lookup"><span data-stu-id="53764-208">Database schemas are required and enforced.</span></span></li>
            <li><span data-ttu-id="53764-209">Az adatbázis entitások közötti kapcsolatok több-a-többhöz.</span><span class="sxs-lookup"><span data-stu-id="53764-209">Many-to-many relationships between data entities in the database.</span></span></li>
            <li><span data-ttu-id="53764-210">Korlátozások a sémában definiált és partíciószeletre vonatkozó adatokat az adatbázisban.</span><span class="sxs-lookup"><span data-stu-id="53764-210">Constraints are defined in the schema and imposed on any data in the database.</span></span></li>
            <li><span data-ttu-id="53764-211">Adatok integritása magas szintű igényel.</span><span class="sxs-lookup"><span data-stu-id="53764-211">Data requires high integrity.</span></span> <span data-ttu-id="53764-212">Indexek és kapcsolatok kell pontosan kell tartani.</span><span class="sxs-lookup"><span data-stu-id="53764-212">Indexes and relationships need to be maintained accurately.</span></span></li>
            <li><span data-ttu-id="53764-213">Adatok az erős konzisztencia igényel.</span><span class="sxs-lookup"><span data-stu-id="53764-213">Data requires strong consistency.</span></span> <span data-ttu-id="53764-214">Tranzakciók, amely biztosítja az összes adat 100 %-os összes felhasználójához és folyamatok konzisztens módon fog működni.</span><span class="sxs-lookup"><span data-stu-id="53764-214">Transactions operate in a way that ensures all data are 100% consistent for all users and processes.</span></span></li>
            <li><span data-ttu-id="53764-215">Egyes bejegyzések mérete olyan kis és közepes méretű lehet.</span><span class="sxs-lookup"><span data-stu-id="53764-215">Size of individual data entries is intended to be small to medium-sized.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="53764-216">**Példák**</span><span class="sxs-lookup"><span data-stu-id="53764-216">**Examples**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="53764-217">Üzletági (emberi beruházási felügyeletet, az Ügyfélkapcsolat-kezelés, vállalati erőforrás tervezése)</span><span class="sxs-lookup"><span data-stu-id="53764-217">Line of business  (human capital management, customer relationship management, enterprise resource planning)</span></span></li>
            <li><span data-ttu-id="53764-218">Készlet kezelése</span><span class="sxs-lookup"><span data-stu-id="53764-218">Inventory management</span></span></li>
            <li><span data-ttu-id="53764-219">Jelentéskészítő adatbázis</span><span class="sxs-lookup"><span data-stu-id="53764-219">Reporting database</span></span></li>
            <li><span data-ttu-id="53764-220">Könyvelés</span><span class="sxs-lookup"><span data-stu-id="53764-220">Accounting</span></span></li>
            <li><span data-ttu-id="53764-221">Eszközök kezelése</span><span class="sxs-lookup"><span data-stu-id="53764-221">Asset management</span></span></li>
            <li><span data-ttu-id="53764-222">Alap kezelése</span><span class="sxs-lookup"><span data-stu-id="53764-222">Fund management</span></span></li>
            <li><span data-ttu-id="53764-223">Kezelési</span><span class="sxs-lookup"><span data-stu-id="53764-223">Order management</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="document-databases"></a><span data-ttu-id="53764-224">Dokumentum-adatbázisokat</span><span class="sxs-lookup"><span data-stu-id="53764-224">Document databases</span></span>

<table>
<tr><td><span data-ttu-id="53764-225">**Számítási feladat**</span><span class="sxs-lookup"><span data-stu-id="53764-225">**Workload**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="53764-226">Általános célú.</span><span class="sxs-lookup"><span data-stu-id="53764-226">General purpose.</span></span></li>
            <li><span data-ttu-id="53764-227">Beszúrási és frissítési műveletek megegyeznek.</span><span class="sxs-lookup"><span data-stu-id="53764-227">Insert and update operations are common.</span></span> <span data-ttu-id="53764-228">Mindkét létrehozása új rekordok és adatok frissítése rendszeresen fordulhat elő.</span><span class="sxs-lookup"><span data-stu-id="53764-228">Both the creation of new records and updates to existing data happen regularly.</span></span></li>
            <li><span data-ttu-id="53764-229">Nincs objektum relációs impedancia nem megfelelő.</span><span class="sxs-lookup"><span data-stu-id="53764-229">No object-relational impedance mismatch.</span></span> <span data-ttu-id="53764-230">Dokumentumok jobban is megegyezhet alkalmazáskód használt objektum struktúrákat.</span><span class="sxs-lookup"><span data-stu-id="53764-230">Documents can better match the object structures used in application code.</span></span></li>
            <li><span data-ttu-id="53764-231">Egyidejű hozzáférések optimista több gyakran használják.</span><span class="sxs-lookup"><span data-stu-id="53764-231">Optimistic concurrency is more commonly used.</span></span></li>
            <li><span data-ttu-id="53764-232">Adatok kell módosíthatók és dolgozza fel az alkalmazás fel.</span><span class="sxs-lookup"><span data-stu-id="53764-232">Data must be modified and processed by consuming application.</span></span></li>
            <li><span data-ttu-id="53764-233">Adatok index több mező szükséges.</span><span class="sxs-lookup"><span data-stu-id="53764-233">Data requires index on multiple fields.</span></span></li>
            <li><span data-ttu-id="53764-234">Egyes dokumentumok beolvasni, és egyetlen blokkot megírva.</span><span class="sxs-lookup"><span data-stu-id="53764-234">Individual documents are retrieved and written as a single block.</span></span></li>
    </td>
</tr>
<tr><td><span data-ttu-id="53764-235">**Adattípus**</span><span class="sxs-lookup"><span data-stu-id="53764-235">**Data type**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="53764-236">Adatok deszerializálni normalizált módon kezelhetők.</span><span class="sxs-lookup"><span data-stu-id="53764-236">Data can be managed in de-normalized way.</span></span></li>
            <li><span data-ttu-id="53764-237">A dokumentum egyes adatok mérete viszonylag kicsi.</span><span class="sxs-lookup"><span data-stu-id="53764-237">Size of individual document data is relatively small.</span></span></li>
            <li><span data-ttu-id="53764-238">Minden egyes dokumentumtípus használhatja a saját séma.</span><span class="sxs-lookup"><span data-stu-id="53764-238">Each document type can use its own schema.</span></span></li>
            <li><span data-ttu-id="53764-239">Dokumentumok választható mezők szerepelhetnek.</span><span class="sxs-lookup"><span data-stu-id="53764-239">Documents can include optional fields.</span></span></li>
            <li><span data-ttu-id="53764-240">A dokumentum adata félig strukturált, ami azt jelenti, hogy adatokat minden mező nem feltétlenül típusait.</span><span class="sxs-lookup"><span data-stu-id="53764-240">Document data is semi-structured, meaning that data types of each field are not strictly defined.</span></span></li>
            <li><span data-ttu-id="53764-241">Adatösszesítés esetén támogatott.</span><span class="sxs-lookup"><span data-stu-id="53764-241">Data aggregation is supported.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="53764-242">**Példák**</span><span class="sxs-lookup"><span data-stu-id="53764-242">**Examples**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="53764-243">Termékkatalógus</span><span class="sxs-lookup"><span data-stu-id="53764-243">Product catalog</span></span></li>
            <li><span data-ttu-id="53764-244">Felhasználói fiókok</span><span class="sxs-lookup"><span data-stu-id="53764-244">User accounts</span></span></li>
            <li><span data-ttu-id="53764-245">Anyagjegyzék</span><span class="sxs-lookup"><span data-stu-id="53764-245">Bill of materials</span></span></li>
            <li><span data-ttu-id="53764-246">Személyre szabás</span><span class="sxs-lookup"><span data-stu-id="53764-246">Personalization</span></span></li>
            <li><span data-ttu-id="53764-247">Tartalomkezelés</span><span class="sxs-lookup"><span data-stu-id="53764-247">Content management</span></span></li>
            <li><span data-ttu-id="53764-248">Operatív adatok</span><span class="sxs-lookup"><span data-stu-id="53764-248">Operations data</span></span></li>
            <li><span data-ttu-id="53764-249">Készlet kezelése</span><span class="sxs-lookup"><span data-stu-id="53764-249">Inventory management</span></span></li>
            <li><span data-ttu-id="53764-250">Tranzakció előzményadatok</span><span class="sxs-lookup"><span data-stu-id="53764-250">Transaction history data</span></span></li>
            <li><span data-ttu-id="53764-251">A más NoSQL-tárolókon materializált nézet.</span><span class="sxs-lookup"><span data-stu-id="53764-251">Materialized view of other NoSQL stores.</span></span> <span data-ttu-id="53764-252">Cserél fájl/BLOB indexelő.</span><span class="sxs-lookup"><span data-stu-id="53764-252">Replaces file/BLOB indexing.</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="keyvalue-stores"></a><span data-ttu-id="53764-253">Kulcs-érték tárolók</span><span class="sxs-lookup"><span data-stu-id="53764-253">Key/value stores</span></span>

<table>
<tr><td><span data-ttu-id="53764-254">**Számítási feladat**</span><span class="sxs-lookup"><span data-stu-id="53764-254">**Workload**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="53764-255">Adatok azonosítja, és egy azonosító kulcs, például a dictionary használatával érhető el.</span><span class="sxs-lookup"><span data-stu-id="53764-255">Data is identified and accessed using a single ID key, like a dictionary.</span></span></li>
            <li><span data-ttu-id="53764-256">Nagymértékben méretezhető.</span><span class="sxs-lookup"><span data-stu-id="53764-256">Massively scalable.</span></span></li>
            <li><span data-ttu-id="53764-257">Nem csatlakozik, lock vagy egyesítésekhez szükség.</span><span class="sxs-lookup"><span data-stu-id="53764-257">No joins, lock, or unions are required.</span></span></li>
            <li><span data-ttu-id="53764-258">Nincs összesítési mechanizmusokat használják.</span><span class="sxs-lookup"><span data-stu-id="53764-258">No aggregation mechanisms are used.</span></span></li>
            <li><span data-ttu-id="53764-259">Másodlagos indexek általában nem használják.</span><span class="sxs-lookup"><span data-stu-id="53764-259">Secondary indexes are generally not used.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="53764-260">**Adattípus**</span><span class="sxs-lookup"><span data-stu-id="53764-260">**Data type**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="53764-261">Adatok mérete általában nagy.</span><span class="sxs-lookup"><span data-stu-id="53764-261">Data size tends to be large.</span></span></li>
            <li><span data-ttu-id="53764-262">Minden kulcs társítva, amely egy nem felügyelt adatok BLOB egyetlen értéket.</span><span class="sxs-lookup"><span data-stu-id="53764-262">Each key is associated with a single value, which is an unmanaged data BLOB.</span></span></li>
            <li><span data-ttu-id="53764-263">Nincs séma kényszerítési van.</span><span class="sxs-lookup"><span data-stu-id="53764-263">There is no schema enforcement.</span></span></li>
            <li><span data-ttu-id="53764-264">Nincs entitások közötti kapcsolatok.</span><span class="sxs-lookup"><span data-stu-id="53764-264">No relationships between entities.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="53764-265">**Példák**</span><span class="sxs-lookup"><span data-stu-id="53764-265">**Examples**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="53764-266">Adatok gyorsítótárazása</span><span class="sxs-lookup"><span data-stu-id="53764-266">Data caching</span></span></li>
            <li><span data-ttu-id="53764-267">Munkamenet-kezelés</span><span class="sxs-lookup"><span data-stu-id="53764-267">Session management</span></span></li>
            <li><span data-ttu-id="53764-268">Felhasználói beállítások és profilok felügyeletének</span><span class="sxs-lookup"><span data-stu-id="53764-268">User preference and profile management</span></span></li>
            <li><span data-ttu-id="53764-269">A termék javaslat és ad szolgál</span><span class="sxs-lookup"><span data-stu-id="53764-269">Product recommendation and ad serving</span></span></li>
            <li><span data-ttu-id="53764-270">szótárak</span><span class="sxs-lookup"><span data-stu-id="53764-270">Dictionaries</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="graph-databases"></a><span data-ttu-id="53764-271">Graph-adatbázisok</span><span class="sxs-lookup"><span data-stu-id="53764-271">Graph databases</span></span>

<table>
<tr><td><span data-ttu-id="53764-272">**Számítási feladat**</span><span class="sxs-lookup"><span data-stu-id="53764-272">**Workload**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="53764-273">Adatok elemek között kapcsolatok nagyon bonyolultak, kapcsolódó adatelemek között hány ugrást használata esetén.</span><span class="sxs-lookup"><span data-stu-id="53764-273">The relationships between data items are very complex, involving many hops between related data items.</span></span></li>
            <li><span data-ttu-id="53764-274">Az adatok elemek közötti kapcsolat dinamikusak, és változnak az idők.</span><span class="sxs-lookup"><span data-stu-id="53764-274">The relationship between data items are dynamic and change over time.</span></span></li>
            <li><span data-ttu-id="53764-275">Objektumok közötti kapcsolatok első osztályú építését, anélkül, hogy a külső kulcsokat és átjárása illesztések.</span><span class="sxs-lookup"><span data-stu-id="53764-275">Relationships between objects are first-class citizens, without requiring foreign-keys and joins to traverse.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="53764-276">**Adattípus**</span><span class="sxs-lookup"><span data-stu-id="53764-276">**Data type**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="53764-277">Adatok csomópontok és a kapcsolatok magában foglalja.</span><span class="sxs-lookup"><span data-stu-id="53764-277">Data is comprised of nodes and relationships.</span></span></li>
            <li><span data-ttu-id="53764-278">Csomópontok a táblázat sorainak vagy JSON-dokumentumok hasonlóak.</span><span class="sxs-lookup"><span data-stu-id="53764-278">Nodes are similar to table rows or JSON documents.</span></span></li>
            <li><span data-ttu-id="53764-279">Kapcsolatok éppen olyan fontos, mint a csomópontok, és közvetlenül a lekérdezés nyelven érhetők el.</span><span class="sxs-lookup"><span data-stu-id="53764-279">Relationships are just as important as nodes, and are exposed directly in the query language.</span></span></li>
            <li><span data-ttu-id="53764-280">Összetett objektumok, például egy személy, a telefonszámokat, az egyes különálló, kisebb csomópontok lehet osztani együtt traversable kapcsolatok</span><span class="sxs-lookup"><span data-stu-id="53764-280">Composite objects, such as a person with multiple phone numbers, tend to be broken into separate, smaller nodes, combined with traversable relationships</span></span> </li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="53764-281">**Példák**</span><span class="sxs-lookup"><span data-stu-id="53764-281">**Examples**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="53764-282">Szervezeti diagramok</span><span class="sxs-lookup"><span data-stu-id="53764-282">Organization charts</span></span></li>
            <li><span data-ttu-id="53764-283">Közösségi diagramjait</span><span class="sxs-lookup"><span data-stu-id="53764-283">Social graphs</span></span></li>
            <li><span data-ttu-id="53764-284">Csalások észlelése</span><span class="sxs-lookup"><span data-stu-id="53764-284">Fraud detection</span></span></li>
            <li><span data-ttu-id="53764-285">Elemzés</span><span class="sxs-lookup"><span data-stu-id="53764-285">Analytics</span></span></li>
            <li><span data-ttu-id="53764-286">A javaslat motorok</span><span class="sxs-lookup"><span data-stu-id="53764-286">Recommendation engines</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="column-family-databases"></a><span data-ttu-id="53764-287">Oszlop-család adatbázisok</span><span class="sxs-lookup"><span data-stu-id="53764-287">Column-family databases</span></span>

<table>
<tr><td><span data-ttu-id="53764-288">**Számítási feladat**</span><span class="sxs-lookup"><span data-stu-id="53764-288">**Workload**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="53764-289">A legtöbb oszlop-család adatbázisok rendkívül gyorsan írási műveletek végrehajtására.</span><span class="sxs-lookup"><span data-stu-id="53764-289">Most column-family databases perform write operations extremely quickly.</span></span></li>
            <li><span data-ttu-id="53764-290">Frissítse és törlési műveletek ritkák.</span><span class="sxs-lookup"><span data-stu-id="53764-290">Update and delete operations are rare.</span></span></li>
            <li><span data-ttu-id="53764-291">Célja, hogy nagyobb teljesítményt és alacsony késésű hozzáférést biztosítanak.</span><span class="sxs-lookup"><span data-stu-id="53764-291">Designed to provide high throughput and low-latency access.</span></span></li>
            <li><span data-ttu-id="53764-292">Támogatja az egyszerű lekérdezés hozzáférésének egy meghatározott sokkal nagyobb rekord mezőit.</span><span class="sxs-lookup"><span data-stu-id="53764-292">Supports easy query access to a particular set of fields within a much larger record.</span></span></li>
            <li><span data-ttu-id="53764-293">Nagymértékben méretezhető.</span><span class="sxs-lookup"><span data-stu-id="53764-293">Massively scalable.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="53764-294">**Adattípus**</span><span class="sxs-lookup"><span data-stu-id="53764-294">**Data type**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="53764-295">Egy kulcsoszlop és egy vagy több oszlopcsaláddal táblák adatokat tárolja.</span><span class="sxs-lookup"><span data-stu-id="53764-295">Data is stored in tables consisting of a key column and one or more column families.</span></span></li>
            <li><span data-ttu-id="53764-296">Egyes oszlopok az egyes sorok eltérőek lehetnek.</span><span class="sxs-lookup"><span data-stu-id="53764-296">Specific columns can vary by individual rows.</span></span></li>
            <li><span data-ttu-id="53764-297">Az egyes cellák get keresztül elért, és helyezze parancsok</span><span class="sxs-lookup"><span data-stu-id="53764-297">Individual cells are accessed via get and put commands</span></span></li>
            <li><span data-ttu-id="53764-298">Több sort visszaadja a vizsgálat paranccsal.</span><span class="sxs-lookup"><span data-stu-id="53764-298">Multiple rows are returned using a scan command.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="53764-299">**Példák**</span><span class="sxs-lookup"><span data-stu-id="53764-299">**Examples**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="53764-300">Javaslatok</span><span class="sxs-lookup"><span data-stu-id="53764-300">Recommendations</span></span></li>
            <li><span data-ttu-id="53764-301">Személyre szabás</span><span class="sxs-lookup"><span data-stu-id="53764-301">Personalization</span></span></li>
            <li><span data-ttu-id="53764-302">Érzékelői adatok</span><span class="sxs-lookup"><span data-stu-id="53764-302">Sensor data</span></span></li>
            <li><span data-ttu-id="53764-303">Telemetria</span><span class="sxs-lookup"><span data-stu-id="53764-303">Telemetry</span></span></li>
            <li><span data-ttu-id="53764-304">Üzenetküldés</span><span class="sxs-lookup"><span data-stu-id="53764-304">Messaging</span></span></li>
            <li><span data-ttu-id="53764-305">Közösségi médiaelemzés használatával</span><span class="sxs-lookup"><span data-stu-id="53764-305">Social media analytics</span></span></li>
            <li><span data-ttu-id="53764-306">Webes elemzőeszközt</span><span class="sxs-lookup"><span data-stu-id="53764-306">Web analytics</span></span></li>
            <li><span data-ttu-id="53764-307">Figyelése</span><span class="sxs-lookup"><span data-stu-id="53764-307">Activity monitoring</span></span></li>
            <li><span data-ttu-id="53764-308">Időjárás és egyéb idősorozat adatok</span><span class="sxs-lookup"><span data-stu-id="53764-308">Weather and other time-series data</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="search-engine-databases"></a><span data-ttu-id="53764-309">Keresési adatbázisok</span><span class="sxs-lookup"><span data-stu-id="53764-309">Search engine databases</span></span>

<table>
<tr><td><span data-ttu-id="53764-310">**Számítási feladat**</span><span class="sxs-lookup"><span data-stu-id="53764-310">**Workload**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="53764-311">Az indexelési adatok több forrásokból és szolgáltatásokat.</span><span class="sxs-lookup"><span data-stu-id="53764-311">Indexing data from multiple sources and services.</span></span></li>
            <li><span data-ttu-id="53764-312">Lekérdezések alkalmi és összetett feladat lehet.</span><span class="sxs-lookup"><span data-stu-id="53764-312">Queries are ad-hoc and can be complex.</span></span></li>
            <li><span data-ttu-id="53764-313">Megköveteli összesítési.</span><span class="sxs-lookup"><span data-stu-id="53764-313">Requires aggregation.</span></span></li>
            <li><span data-ttu-id="53764-314">Teljes szöveges keresés szükség.</span><span class="sxs-lookup"><span data-stu-id="53764-314">Full text search is required.</span></span></li>
            <li><span data-ttu-id="53764-315">Ad hoc önkiszolgáló lekérdezés megadása kötelező.</span><span class="sxs-lookup"><span data-stu-id="53764-315">Ad hoc self-service query is required.</span></span></li>
            <li><span data-ttu-id="53764-316">Minden mező indexével Data elemzésre szükség.</span><span class="sxs-lookup"><span data-stu-id="53764-316">Data analysis with index on all fields is required.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="53764-317">**Adattípus**</span><span class="sxs-lookup"><span data-stu-id="53764-317">**Data type**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="53764-318">Félig strukturált vagy strukturálatlan</span><span class="sxs-lookup"><span data-stu-id="53764-318">Semi-structured or unstructured</span></span></li>
            <li><span data-ttu-id="53764-319">Szöveg</span><span class="sxs-lookup"><span data-stu-id="53764-319">Text</span></span></li>
            <li><span data-ttu-id="53764-320">Szöveg strukturált adatok alapján</span><span class="sxs-lookup"><span data-stu-id="53764-320">Text with reference to structured data</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="53764-321">**Példák**</span><span class="sxs-lookup"><span data-stu-id="53764-321">**Examples**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="53764-322">A termék katalógusok</span><span class="sxs-lookup"><span data-stu-id="53764-322">Product catalogs</span></span></li>
            <li><span data-ttu-id="53764-323">Hely keresése</span><span class="sxs-lookup"><span data-stu-id="53764-323">Site search</span></span></li>
            <li><span data-ttu-id="53764-324">Naplózás</span><span class="sxs-lookup"><span data-stu-id="53764-324">Logging</span></span></li>
            <li><span data-ttu-id="53764-325">Elemzés</span><span class="sxs-lookup"><span data-stu-id="53764-325">Analytics</span></span></li>
            <li><span data-ttu-id="53764-326">Helyek vásárlás</span><span class="sxs-lookup"><span data-stu-id="53764-326">Shopping sites</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="data-warehouse"></a><span data-ttu-id="53764-327">Adattárház</span><span class="sxs-lookup"><span data-stu-id="53764-327">Data warehouse</span></span>

<table>
<tr><td><span data-ttu-id="53764-328">**Számítási feladat**</span><span class="sxs-lookup"><span data-stu-id="53764-328">**Workload**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="53764-329">Adatelemzés</span><span class="sxs-lookup"><span data-stu-id="53764-329">Data analytics</span></span></li>
            <li><span data-ttu-id="53764-330">Vállalati BI</span><span class="sxs-lookup"><span data-stu-id="53764-330">Enterprise BI</span></span>   </li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="53764-331">**Adattípus**</span><span class="sxs-lookup"><span data-stu-id="53764-331">**Data type**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="53764-332">A különböző forrásokból származó előzményadatok.</span><span class="sxs-lookup"><span data-stu-id="53764-332">Historical data from multiple sources.</span></span></li>
            <li><span data-ttu-id="53764-333">"Csillag" vagy "snowflake" séma, amely tény-és dimenziótáblák általában denormalizált.</span><span class="sxs-lookup"><span data-stu-id="53764-333">Usually denormalized in a "star" or "snowflake" schema, consisting of fact and dimension tables.</span></span></li>
            <li><span data-ttu-id="53764-334">Általában be az új adatokkal ütemezés szerint.</span><span class="sxs-lookup"><span data-stu-id="53764-334">Usually loaded with new data on a scheduled basis.</span></span></li>
            <li><span data-ttu-id="53764-335">Dimenziótáblák többek között általában egy entitás néven több történelmi verziója egy *lassan a dimenzió módosításával*.</span><span class="sxs-lookup"><span data-stu-id="53764-335">Dimension tables often include multiple historic versions of an entity, referred to as a *slowly changing dimension*.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="53764-336">**Példák**</span><span class="sxs-lookup"><span data-stu-id="53764-336">**Examples**</span></span></td>
    <td><span data-ttu-id="53764-337">A vállalati adatokat biztosít a elemzési modellek, jelentések és irányítópultok az adatraktár.</span><span class="sxs-lookup"><span data-stu-id="53764-337">An enterprise data warehouse that provides data for analytical models, reports, and dashboards.</span></span>
    </td>
</tr>
</table>


## <a name="time-series-databases"></a><span data-ttu-id="53764-338">Idő adatsorozat adatbázisok</span><span class="sxs-lookup"><span data-stu-id="53764-338">Time series databases</span></span>

<table>
<tr><td><span data-ttu-id="53764-339">**Számítási feladat**</span><span class="sxs-lookup"><span data-stu-id="53764-339">**Workload**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="53764-340">Egy túlnyomórészt műveletek (95-99 %) hányadát-e írási műveleteket.</span><span class="sxs-lookup"><span data-stu-id="53764-340">An overwhelmingly proportion of operations (95-99%) are writes.</span></span></li>
            <li><span data-ttu-id="53764-341">Rekord idő sorrendben általában hozzáfűződik egymás után.</span><span class="sxs-lookup"><span data-stu-id="53764-341">Records are generally appended sequentially in time order.</span></span></li>
            <li><span data-ttu-id="53764-342">Frissítések ritkák.</span><span class="sxs-lookup"><span data-stu-id="53764-342">Updates are rare.</span></span></li>
            <li><span data-ttu-id="53764-343">Törlések tömeges fordul elő, és egymást követő blokkot vagy rekordok történik.</span><span class="sxs-lookup"><span data-stu-id="53764-343">Deletes occur in bulk, and are made to contiguous blocks or records.</span></span></li>
            <li><span data-ttu-id="53764-344">Olvasási kérések lehet nagyobb, mint a rendelkezésre álló memória.</span><span class="sxs-lookup"><span data-stu-id="53764-344">Read requests can be larger than available memory.</span></span></li>
            <li><span data-ttu-id="53764-345">Esetében gyakori, hogy mi történjen, párhuzamosan több olvasása.</span><span class="sxs-lookup"><span data-stu-id="53764-345">It's common for multiple reads to occur simultaneously.</span></span></li>
            <li><span data-ttu-id="53764-346">Adatolvasás egymás után növekvő vagy csökkenő sorrendben idő.</span><span class="sxs-lookup"><span data-stu-id="53764-346">Data is read sequentially in either ascending or descending time order.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="53764-347">**Adattípus**</span><span class="sxs-lookup"><span data-stu-id="53764-347">**Data type**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="53764-348">A primary key és rendezési mechanizmust, amely időbélyegző.</span><span class="sxs-lookup"><span data-stu-id="53764-348">A time stamp that is used as the primary key and sorting mechanism.</span></span></li>
            <li><span data-ttu-id="53764-349">A belépési vagy a bejegyzést jelöli leírását mérték.</span><span class="sxs-lookup"><span data-stu-id="53764-349">Measurements from the entry or descriptions of what the entry represents.</span></span></li>
            <li><span data-ttu-id="53764-350">További információt a típusát, a forrás és az egyéb információkat a bejegyzés megadása címkéket.</span><span class="sxs-lookup"><span data-stu-id="53764-350">Tags that define additional information about the type, origin, and other information about the entry.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="53764-351">**Példák**</span><span class="sxs-lookup"><span data-stu-id="53764-351">**Examples**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="53764-352">Figyelés és az esemény telemetriai adatokat.</span><span class="sxs-lookup"><span data-stu-id="53764-352">Monitoring and event telemetry.</span></span></li>
            <li><span data-ttu-id="53764-353">Érzékelő vagy egyéb IoT-adatokat.</span><span class="sxs-lookup"><span data-stu-id="53764-353">Sensor or other IoT data.</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="object-storage"></a><span data-ttu-id="53764-354">Objektumtár</span><span class="sxs-lookup"><span data-stu-id="53764-354">Object storage</span></span>

<table>
<tr><td><span data-ttu-id="53764-355">**Számítási feladat**</span><span class="sxs-lookup"><span data-stu-id="53764-355">**Workload**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="53764-356">Kulcs azonosítja.</span><span class="sxs-lookup"><span data-stu-id="53764-356">Identified by key.</span></span></li>
            <li><span data-ttu-id="53764-357">Lehet, hogy objektumok nyilvánosan vagy egyénileg érhető el.</span><span class="sxs-lookup"><span data-stu-id="53764-357">Objects may be publicly or privately accessible.</span></span></li>
            <li><span data-ttu-id="53764-358">Tartalom van általában egy eszköz, például egy táblázat, kép vagy videofájl.</span><span class="sxs-lookup"><span data-stu-id="53764-358">Content is typically an asset such as a spreadsheet, image, or video file.</span></span></li>
            <li><span data-ttu-id="53764-359">Lehet, hogy a tartalom tartós (állandó), és a külső alkalmazás réteg vagy virtuális gép.</span><span class="sxs-lookup"><span data-stu-id="53764-359">Content must be durable (persistent), and external to any application tier or virtual machine.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="53764-360">**Adattípus**</span><span class="sxs-lookup"><span data-stu-id="53764-360">**Data type**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="53764-361">Adatok mérete nagy.</span><span class="sxs-lookup"><span data-stu-id="53764-361">Data size is large.</span></span></li>
            <li><span data-ttu-id="53764-362">Blobadatok.</span><span class="sxs-lookup"><span data-stu-id="53764-362">Blob data.</span></span></li>
            <li><span data-ttu-id="53764-363">Értéke nem átlátszó.</span><span class="sxs-lookup"><span data-stu-id="53764-363">Value is opaque.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="53764-364">**Példák**</span><span class="sxs-lookup"><span data-stu-id="53764-364">**Examples**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="53764-365">Képek, videók, office-dokumentumok, PDF-fájlok</span><span class="sxs-lookup"><span data-stu-id="53764-365">Images, videos, office documents, PDFs</span></span></li>
            <li><span data-ttu-id="53764-366">CSS, parancsfájlok, CSV</span><span class="sxs-lookup"><span data-stu-id="53764-366">CSS, Scripts, CSV</span></span></li>
            <li><span data-ttu-id="53764-367">Statikus HTML, JSON</span><span class="sxs-lookup"><span data-stu-id="53764-367">Static HTML, JSON</span></span></li>
            <li><span data-ttu-id="53764-368">Napló-és naplózás</span><span class="sxs-lookup"><span data-stu-id="53764-368">Log and audit files</span></span></li>
            <li><span data-ttu-id="53764-369">Adatbázisok biztonsági mentése</span><span class="sxs-lookup"><span data-stu-id="53764-369">Database backups</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="shared-files"></a><span data-ttu-id="53764-370">Megosztott fájlok</span><span class="sxs-lookup"><span data-stu-id="53764-370">Shared files</span></span>

<table>
<tr><td><span data-ttu-id="53764-371">**Számítási feladat**</span><span class="sxs-lookup"><span data-stu-id="53764-371">**Workload**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="53764-372">Áttelepítés a meglévő alkalmazás, amely interakciót folytatni a fájlrendszerben.</span><span class="sxs-lookup"><span data-stu-id="53764-372">Migration from existing apps that interact with the file system.</span></span></li>
            <li><span data-ttu-id="53764-373">SMB-kapcsolat szükséges.</span><span class="sxs-lookup"><span data-stu-id="53764-373">Requires SMB interface.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="53764-374">**Adattípus**</span><span class="sxs-lookup"><span data-stu-id="53764-374">**Data type**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="53764-375">Fájlok, mappák hierarchikus meg.</span><span class="sxs-lookup"><span data-stu-id="53764-375">Files in a hierarchical set of folders.</span></span></li>
            <li><span data-ttu-id="53764-376">Elérhető a szabványos i/o-könyvtárak.</span><span class="sxs-lookup"><span data-stu-id="53764-376">Accessible with standard I/O libraries.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="53764-377">**Példák**</span><span class="sxs-lookup"><span data-stu-id="53764-377">**Examples**</span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="53764-378">A hagyományos fájlok</span><span class="sxs-lookup"><span data-stu-id="53764-378">Legacy files</span></span></li>
            <li><span data-ttu-id="53764-379">Megosztott tartalom érhető el a virtuális gépek vagy az app-példányok száma között</span><span class="sxs-lookup"><span data-stu-id="53764-379">Shared content accessible among a number of VMs or app instances</span></span></li>
        </ul>
    </td>
</tr>
</table>
