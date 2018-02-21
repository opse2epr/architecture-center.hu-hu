---
title: "Online tranzakció-feldolgozási (OLTP)"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 07e7f680c8ee5e8589ff7cd2236ff95f6ee84f4c
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/14/2018
---
# <a name="online-transaction-processing-oltp"></a><span data-ttu-id="646a1-102">Online tranzakció-feldolgozási (OLTP)</span><span class="sxs-lookup"><span data-stu-id="646a1-102">Online transaction processing (OLTP)</span></span>

<span data-ttu-id="646a1-103">A felügyeleti [tranzakciós adatok](../concepts/transactional-data.md) használ a számítógéprendszerek lehet hivatkozni, Online tranzakciófeldolgozási (OLTP).</span><span class="sxs-lookup"><span data-stu-id="646a1-103">The management of [transactional data](../concepts/transactional-data.md) using computer systems is referred to as Online Transaction Processing (OLTP).</span></span> <span data-ttu-id="646a1-104">Az OLTP rendszerek regisztrálhatna üzleti fordulnak elő a szervezet napi működését, és támogatja az adatokba, így következtetéseket lekérdezését.</span><span class="sxs-lookup"><span data-stu-id="646a1-104">OLTP systems record business interactions as they occur in the day-to-day operation of the organization, and support querying of this data to make inferences.</span></span>

![OLTP az Azure-ban](./images/oltp-data-pipeline.png)

## <a name="when-to-use-this-solution"></a><span data-ttu-id="646a1-106">Ez a megoldás használatával</span><span class="sxs-lookup"><span data-stu-id="646a1-106">When to use this solution</span></span>

<span data-ttu-id="646a1-107">Ha hatékony feldolgozni, és üzleti tranzakciók tárolja, és azonnal elérhetővé tétele az ügyfélalkalmazások konzisztens módon van szüksége, válassza a OLTP.</span><span class="sxs-lookup"><span data-stu-id="646a1-107">Choose OLTP when you need to efficiently process and store business transactions and immediately make them available to client applications in a consistent way.</span></span> <span data-ttu-id="646a1-108">Használja az ebbe az architektúrába, ha bármilyen anyagi feldolgozási késedelem hatással lenne egy negatív az üzleti mindennapi műveleteinek.</span><span class="sxs-lookup"><span data-stu-id="646a1-108">Use this architecture when any tangible delay in processing would have a negative impact on the day-to-day operations of the business.</span></span>

<span data-ttu-id="646a1-109">OLTP rendszerek hatékonyan feldolgozni, és a tranzakciók, valamint a lekérdezés tranzakciós adatok tárolására készültek.</span><span class="sxs-lookup"><span data-stu-id="646a1-109">OLTP systems are designed to efficiently process and store transactions, as well as query transactional data.</span></span> <span data-ttu-id="646a1-110">A cél hatékonyan feldolgozási és tárolása az egyes tranzakciók által az OLTP rendszerek részben úgy érhető el, adatok normalizálási &mdash; Ez azt jelenti, hogy ossza az adatok kisebb csoportjai, amelyek kevesebb redundáns.</span><span class="sxs-lookup"><span data-stu-id="646a1-110">The goal of efficiently processing and storing individual transactions by an OLTP system is partly accomplished by data normalization &mdash; that is, breaking the data up into smaller chunks that are less redundant.</span></span> <span data-ttu-id="646a1-111">Támogatja a hatékonyságát, mivel lehetővé teszi, hogy az OLTP rendszerek nagyszámú tranzakciók egymástól függetlenül feldolgozni, és elkerülhető a felesleges feldolgozási szükséges redundáns adatok mellett adatok integritásának fenntartása.</span><span class="sxs-lookup"><span data-stu-id="646a1-111">This supports efficiency because it enables the OLTP system to process large numbers of transactions independently, and avoids extra processing needed to maintain data integrity in the presence of redundant data.</span></span>

## <a name="challenges"></a><span data-ttu-id="646a1-112">Kihívásai</span><span class="sxs-lookup"><span data-stu-id="646a1-112">Challenges</span></span>
<span data-ttu-id="646a1-113">Végrehajtási és az OLTP rendszerek használata néhány kihívást hozhat létre:</span><span class="sxs-lookup"><span data-stu-id="646a1-113">Implementing and using an OLTP system can create a few challenges:</span></span>

- <span data-ttu-id="646a1-114">Az OLTP rendszerek megfelelőek nem mindig összesítések kezelése keresztül nagy mennyiségű adat, bár kivételek, például egy tervezett jól az SQL Server-alapú megoldás.</span><span class="sxs-lookup"><span data-stu-id="646a1-114">OLTP systems are not always good for handling aggregates over large amounts of data, although there are exceptions, such as a well-planned SQL Server-based solution.</span></span> <span data-ttu-id="646a1-115">Az adatok, támaszkodó összesített számítások az egyes tranzakciók több mint több millió, alapján Analytics nagyon erőforrás-igényes az OLTP rendszerek.</span><span class="sxs-lookup"><span data-stu-id="646a1-115">Analytics against the data, that rely on aggregate calculations over millions of individual transactions, are very resource intensive for an OLTP system.</span></span> <span data-ttu-id="646a1-116">Lassú hajtható végre, és lehet el egy slow-down következtében letiltásával más tranzakciók az adatbázisban.</span><span class="sxs-lookup"><span data-stu-id="646a1-116">They can be slow to execute and can cause a slow-down by blocking other transactions in the database.</span></span>
- <span data-ttu-id="646a1-117">Ha elemzés végrehajtása, és az adatok magas normalizált jelentéskészítés, a lekérdezések általában összetett, mert irányuló legtöbb lekérdezésnek kell deszerializálni optimalizálására illesztések használatával az adatok.</span><span class="sxs-lookup"><span data-stu-id="646a1-117">When conducting analytics and reporting on data that is highly normalized, the queries tend to be complex, because most queries need to de-normalize the data by using joins.</span></span> <span data-ttu-id="646a1-118">Emellett az OLTP rendszerek adatbázis-objektumok elnevezési szabályai általában tömör és állapotára.</span><span class="sxs-lookup"><span data-stu-id="646a1-118">Also, naming conventions for database objects in OLTP systems tend to be terse and succinct.</span></span> <span data-ttu-id="646a1-119">A nagyobb normalizálási alapján tömör elnevezési konvenciók kialakulhat nehezebben OLTP rendszerek üzleti felhasználók DBA vagy az adatok a fejlesztők az nélkül lekérdezéséhez.</span><span class="sxs-lookup"><span data-stu-id="646a1-119">The increased normalization coupled with terse naming conventions makes OLTP systems difficult for business users to query, without the help of a DBA or data developer.</span></span>
- <span data-ttu-id="646a1-120">Lassú a lekérdezési teljesítményt, attól függően, hogy a tárolt tranzakciók száma tranzakciók előzményeinek határozatlan ideig tárolja, és túl sok adat tárolása táblában vezethet.</span><span class="sxs-lookup"><span data-stu-id="646a1-120">Storing the history of transactions indefinitely and storing too much data in any one table can lead to slow query performance, depending on the number of transactions stored.</span></span> <span data-ttu-id="646a1-121">A gyakori megoldás, hogy az OLTP rendszerben (például az aktuális pénzügyi év) megfelelő ablakban karbantartása és egyéb rendszerekre, például egy adatpiacot korábbi adatok továbbítása vagy [adatraktár](../technology-choices/data-warehouses.md).</span><span class="sxs-lookup"><span data-stu-id="646a1-121">The common solution is to maintain a relevant window of time (such as the current fiscal year) in the OLTP system and offload historical data to other systems, such as a data mart or [data warehouse](../technology-choices/data-warehouses.md).</span></span>

## <a name="oltp-in-azure"></a><span data-ttu-id="646a1-122">OLTP az Azure-ban</span><span class="sxs-lookup"><span data-stu-id="646a1-122">OLTP in Azure</span></span>

<span data-ttu-id="646a1-123">Alkalmazások, mint az üzemeltetett webhelyek [App Service Web Apps](/azure/app-service/app-service-web-overview), az App Service-ben futó REST API-k vagy hordozható vagy asztali alkalmazások az OLTP rendszerek általában a REST API-t a közvetítő keresztül kommunikálnak.</span><span class="sxs-lookup"><span data-stu-id="646a1-123">Applications such as websites hosted in [App Service Web Apps](/azure/app-service/app-service-web-overview), REST APIs running in App Service, or mobile or desktop applications communicate with the OLTP system, typically via a REST API intermediary.</span></span>

<span data-ttu-id="646a1-124">A gyakorlatban a munkaterhelések többségéhez nincsenek tisztán OLTP.</span><span class="sxs-lookup"><span data-stu-id="646a1-124">In practice, most workloads are not purely OLTP.</span></span> <span data-ttu-id="646a1-125">Nincs általában egy [analitikai összetevő](../scenarios/online-analytical-processing.md) is.</span><span class="sxs-lookup"><span data-stu-id="646a1-125">There tends to be an [analytical component](../scenarios/online-analytical-processing.md) as well.</span></span> <span data-ttu-id="646a1-126">Emellett nincs valós idejű jelentéskészítést, például a jelentések futtatása ellen a műveleti rendszerfiókok iránti kereslet.</span><span class="sxs-lookup"><span data-stu-id="646a1-126">In addition, there is an increasing demand for real-time reporting, such as running reports against the operational system.</span></span> <span data-ttu-id="646a1-127">Ez más néven a HTAP (hibrid tranzakciós és Analytical Processing).</span><span class="sxs-lookup"><span data-stu-id="646a1-127">This is also referred to as HTAP (Hybrid Transactional and Analytical Processing).</span></span> <span data-ttu-id="646a1-128">További információkért lásd: [Online Analytical Processing (OLAP) adattárolókhoz](../technology-choices/olap-data-stores.md).</span><span class="sxs-lookup"><span data-stu-id="646a1-128">For more information, see [Online Analytical Processing (OLAP) data stores](../technology-choices/olap-data-stores.md).</span></span>

## <a name="technology-choices"></a><span data-ttu-id="646a1-129">Technológiai lehetőségek</span><span class="sxs-lookup"><span data-stu-id="646a1-129">Technology choices</span></span>

<span data-ttu-id="646a1-130">Adattárolás:</span><span class="sxs-lookup"><span data-stu-id="646a1-130">Data storage:</span></span>

- [<span data-ttu-id="646a1-131">Azure SQL Database</span><span class="sxs-lookup"><span data-stu-id="646a1-131">Azure SQL Database</span></span>](/azure/sql-database/)
- [<span data-ttu-id="646a1-132">SQL Server egy Azure virtuális gépen</span><span class="sxs-lookup"><span data-stu-id="646a1-132">SQL Server in an Azure VM</span></span>](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-server-iaas-overview?toc=%2Fazure%2Fvirtual-machines%2Fwindows%2Ftoc.json)
- [<span data-ttu-id="646a1-133">Azure Database for MySQL</span><span class="sxs-lookup"><span data-stu-id="646a1-133">Azure Database for MySQL</span></span>](/azure/mysql/)
- [<span data-ttu-id="646a1-134">Azure Database for PostgreSQL</span><span class="sxs-lookup"><span data-stu-id="646a1-134">Azure Database for PostgreSQL</span></span>](/azure/postgresql/)

<span data-ttu-id="646a1-135">További információkért lásd: [egy OLTP adattároló kiválasztása](../technology-choices/oltp-data-stores.md)</span><span class="sxs-lookup"><span data-stu-id="646a1-135">For more information, see [Choosing an OLTP data store](../technology-choices/oltp-data-stores.md)</span></span>

<span data-ttu-id="646a1-136">Adatforrások:</span><span class="sxs-lookup"><span data-stu-id="646a1-136">Data sources:</span></span>

- [<span data-ttu-id="646a1-137">Az App service</span><span class="sxs-lookup"><span data-stu-id="646a1-137">App service</span></span>](/azure/app-service/)
- [<span data-ttu-id="646a1-138">Mobile Apps</span><span class="sxs-lookup"><span data-stu-id="646a1-138">Mobile Apps</span></span>](/azure/app-service-mobile/)
