---
title: Tervezési alapelvek Azure-alkalmazásokhoz
titleSuffix: Azure Application Architecture Guide
description: Tervezési alapelvek Azure-alkalmazásokhoz.
author: MikeWasson
ms.date: 08/30/2018
ms.custom: seojan19
ms.openlocfilehash: 0ea6d6dd8a030591ce00a42aad5c693ea7809f6a
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/08/2019
ms.locfileid: "54110458"
---
# <a name="ten-design-principles-for-azure-applications"></a><span data-ttu-id="d3531-103">Tíz tervezési alapelv Azure-alkalmazásokhoz</span><span class="sxs-lookup"><span data-stu-id="d3531-103">Ten design principles for Azure applications</span></span>

<span data-ttu-id="d3531-104">Ezeket a tervezési alapelveket követve skálázhatóbbá, rugalmasabbá és felügyelhetőbbé teheti alkalmazását.</span><span class="sxs-lookup"><span data-stu-id="d3531-104">Follow these design principles to make your application more scalable, resilient, and manageable.</span></span>

<span data-ttu-id="d3531-105">**[Tervezzen az önjavítást szem előtt tartva](self-healing.md)**.</span><span class="sxs-lookup"><span data-stu-id="d3531-105">**[Design for self healing](self-healing.md)**.</span></span> <span data-ttu-id="d3531-106">Az elosztott rendszerekben időnként fellépnek hibák.</span><span class="sxs-lookup"><span data-stu-id="d3531-106">In a distributed system, failures happen.</span></span> <span data-ttu-id="d3531-107">Tervezheti úgy az alkalmazását, hogy kijavítsa önmagát, ha hiba történik.</span><span class="sxs-lookup"><span data-stu-id="d3531-107">Design your application to be self healing when failures occur.</span></span>

<span data-ttu-id="d3531-108">**[Tervezzen mindent redundánsra](redundancy.md)**.</span><span class="sxs-lookup"><span data-stu-id="d3531-108">**[Make all things redundant](redundancy.md)**.</span></span> <span data-ttu-id="d3531-109">Redundanciát építve az alkalmazásba elkerülheti a kritikus hibapontokat.</span><span class="sxs-lookup"><span data-stu-id="d3531-109">Build redundancy into your application, to avoid having single points of failure.</span></span>

<span data-ttu-id="d3531-110">**[Minimalizálja a koordinációt](minimize-coordination.md)**.</span><span class="sxs-lookup"><span data-stu-id="d3531-110">**[Minimize coordination](minimize-coordination.md)**.</span></span> <span data-ttu-id="d3531-111">A skálázhatóság érdekében minimalizálja a koordinációt az alkalmazásszolgáltatások között.</span><span class="sxs-lookup"><span data-stu-id="d3531-111">Minimize coordination between application services to achieve scalability.</span></span>

<span data-ttu-id="d3531-112">**[Tervezzen horizontális felskálázásra](scale-out.md)**. Tervezze úgy az alkalmazását, hogy az skálázható legyen horizontálisan, igény szerint hozzá lehessen adni vagy el lehessen távolítani új példányokat.</span><span class="sxs-lookup"><span data-stu-id="d3531-112">**[Design to scale out](scale-out.md)**. Design your application so that it can scale horizontally, adding or removing new instances as demand requires.</span></span>

<span data-ttu-id="d3531-113">**[Particionáljon a korlátok megkerüléséhez](partition.md)**.</span><span class="sxs-lookup"><span data-stu-id="d3531-113">**[Partition around limits](partition.md)**.</span></span> <span data-ttu-id="d3531-114">Használjon particionálást az adatbázis-, hálózati és számítási korlátok megkerüléséhez.</span><span class="sxs-lookup"><span data-stu-id="d3531-114">Use partitioning to work around database, network, and compute limits.</span></span>

<span data-ttu-id="d3531-115">**[Tervezzen műveletekhez](design-for-operations.md)**.</span><span class="sxs-lookup"><span data-stu-id="d3531-115">**[Design for operations](design-for-operations.md)**.</span></span> <span data-ttu-id="d3531-116">Tervezze úgy az alkalmazását, hogy a műveleti csapatnak kéznél legyenek a szükséges eszközök.</span><span class="sxs-lookup"><span data-stu-id="d3531-116">Design your application so that the operations team has the tools they need.</span></span>

<span data-ttu-id="d3531-117">**[Használjon felügyelt szolgáltatásokat](managed-services.md)**.</span><span class="sxs-lookup"><span data-stu-id="d3531-117">**[Use managed services](managed-services.md)**.</span></span> <span data-ttu-id="d3531-118">Amikor lehetséges, a szolgáltatásként nyújtott infrastruktúra (IaaS) helyett használjon szolgáltatásként nyújtott platformot (PaaS).</span><span class="sxs-lookup"><span data-stu-id="d3531-118">When possible, use platform as a service (PaaS) rather than infrastructure as a service (IaaS).</span></span>

<span data-ttu-id="d3531-119">**[Használja a feladathoz legmegfelelőbb adattárat](use-the-best-data-store.md)**.</span><span class="sxs-lookup"><span data-stu-id="d3531-119">**[Use the best data store for the job](use-the-best-data-store.md)**.</span></span> <span data-ttu-id="d3531-120">Válassza az adataihoz és azok felhasználásához leginkább megfelelő tárolótechnológiát.</span><span class="sxs-lookup"><span data-stu-id="d3531-120">Pick the storage technology that is the best fit for your data and how it will be used.</span></span>

<span data-ttu-id="d3531-121">**[Tervezzen a fejlődést szem előtt tartva](design-for-evolution.md)**.</span><span class="sxs-lookup"><span data-stu-id="d3531-121">**[Design for evolution](design-for-evolution.md)**.</span></span> <span data-ttu-id="d3531-122">Idővel minden sikeres alkalmazás változik.</span><span class="sxs-lookup"><span data-stu-id="d3531-122">All successful applications change over time.</span></span> <span data-ttu-id="d3531-123">A fejlődést szem előtt tartó tervezés kulcsfontosságú a folyamatos innováció szempontjából.</span><span class="sxs-lookup"><span data-stu-id="d3531-123">An evolutionary design is key for continuous innovation.</span></span>

<span data-ttu-id="d3531-124">**[Tervezzen a vállalkozás igényei szerint](build-for-business.md)**.</span><span class="sxs-lookup"><span data-stu-id="d3531-124">**[Build for the needs of business](build-for-business.md)**.</span></span> <span data-ttu-id="d3531-125">Minden tervezési döntés legyen igazolható egy üzleti igénnyel.</span><span class="sxs-lookup"><span data-stu-id="d3531-125">Every design decision must be justified by a business requirement.</span></span>
