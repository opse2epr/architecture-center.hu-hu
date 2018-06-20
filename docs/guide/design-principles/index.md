---
title: Tervezési alapelvek Azure-alkalmazásokhoz
description: Tervezési alapelvek Azure-alkalmazásokhoz
author: MikeWasson
ms.openlocfilehash: 462896098c668c0775464ca498925266cd73c6e1
ms.sourcegitcommit: 26b04f138a860979aea5d253ba7fecffc654841e
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 06/19/2018
ms.locfileid: "36206804"
---
# <a name="design-principles-for-azure-applications"></a><span data-ttu-id="fb92a-103">Tervezési alapelvek Azure-alkalmazásokhoz</span><span class="sxs-lookup"><span data-stu-id="fb92a-103">Design principles for Azure applications</span></span>

<span data-ttu-id="fb92a-104">Ezeket a tervezési alapelveket követve skálázhatóbbá, rugalmasabbá és felügyelhetőbbé teheti alkalmazását.</span><span class="sxs-lookup"><span data-stu-id="fb92a-104">Follow these design principles to make your application more scalable, resilient, and manageable.</span></span> 

<span data-ttu-id="fb92a-105">**[Tervezzen az önjavítást szem előtt tartva](self-healing.md)**.</span><span class="sxs-lookup"><span data-stu-id="fb92a-105">**[Design for self healing](self-healing.md)**.</span></span> <span data-ttu-id="fb92a-106">Az elosztott rendszerekben időnként fellépnek hibák.</span><span class="sxs-lookup"><span data-stu-id="fb92a-106">In a distributed system, failures happen.</span></span> <span data-ttu-id="fb92a-107">Tervezheti úgy az alkalmazását, hogy kijavítsa önmagát, ha hiba történik.</span><span class="sxs-lookup"><span data-stu-id="fb92a-107">Design your application to be self healing when failures occur.</span></span>

<span data-ttu-id="fb92a-108">**[Tervezzen mindent redundánsra](redundancy.md)**.</span><span class="sxs-lookup"><span data-stu-id="fb92a-108">**[Make all things redundant](redundancy.md)**.</span></span> <span data-ttu-id="fb92a-109">Redundanciát építve az alkalmazásba elkerülheti a kritikus hibapontokat.</span><span class="sxs-lookup"><span data-stu-id="fb92a-109">Build redundancy into your application, to avoid having single points of failure.</span></span>
 
<span data-ttu-id="fb92a-110">**[Minimalizálja a koordinációt](minimize-coordination.md)**.</span><span class="sxs-lookup"><span data-stu-id="fb92a-110">**[Minimize coordination](minimize-coordination.md)**.</span></span> <span data-ttu-id="fb92a-111">A skálázhatóság érdekében minimalizálja a koordinációt az alkalmazásszolgáltatások között.</span><span class="sxs-lookup"><span data-stu-id="fb92a-111">Minimize coordination between application services to achieve scalability.</span></span>
 
<span data-ttu-id="fb92a-112">**[Tervezzen horizontális felskálázásra](scale-out.md)**. Tervezze úgy az alkalmazását, hogy az skálázható legyen horizontálisan, igény szerint hozzá lehessen adni vagy el lehessen távolítani új példányokat.</span><span class="sxs-lookup"><span data-stu-id="fb92a-112">**[Design to scale out](scale-out.md)**. Design your application so that it can scale horizontally, adding or removing new instances as demand requires.</span></span>

<span data-ttu-id="fb92a-113">**[Particionáljon a korlátok megkerüléséhez](partition.md)**.</span><span class="sxs-lookup"><span data-stu-id="fb92a-113">**[Partition around limits](partition.md)**.</span></span> <span data-ttu-id="fb92a-114">Használjon particionálást az adatbázis-, hálózati és számítási korlátok megkerüléséhez.</span><span class="sxs-lookup"><span data-stu-id="fb92a-114">Use partitioning to work around database, network, and compute limits.</span></span>

<span data-ttu-id="fb92a-115">**[Tervezzen műveletekhez](design-for-operations.md)**.</span><span class="sxs-lookup"><span data-stu-id="fb92a-115">**[Design for operations](design-for-operations.md)**.</span></span> <span data-ttu-id="fb92a-116">Tervezze úgy az alkalmazását, hogy a műveleti csapatnak kéznél legyenek a szükséges eszközök.</span><span class="sxs-lookup"><span data-stu-id="fb92a-116">Design your application so that the operations team has the tools they need.</span></span>

<span data-ttu-id="fb92a-117">**[Használjon felügyelt szolgáltatásokat](managed-services.md)**.</span><span class="sxs-lookup"><span data-stu-id="fb92a-117">**[Use managed services](managed-services.md)**.</span></span> <span data-ttu-id="fb92a-118">Amikor lehetséges, a szolgáltatásként nyújtott infrastruktúra (IaaS) helyett használjon szolgáltatásként nyújtott platformot (PaaS).</span><span class="sxs-lookup"><span data-stu-id="fb92a-118">When possible, use platform as a service (PaaS) rather than infrastructure as a service (IaaS).</span></span>

<span data-ttu-id="fb92a-119">**[Használja a feladathoz legmegfelelőbb adattárat](use-the-best-data-store.md)**.</span><span class="sxs-lookup"><span data-stu-id="fb92a-119">**[Use the best data store for the job](use-the-best-data-store.md)**.</span></span> <span data-ttu-id="fb92a-120">Válassza az adataihoz és azok felhasználásához leginkább megfelelő tárolótechnológiát.</span><span class="sxs-lookup"><span data-stu-id="fb92a-120">Pick the storage technology that is the best fit for your data and how it will be used.</span></span> 
 
<span data-ttu-id="fb92a-121">**[Tervezzen a fejlődést szem előtt tartva](design-for-evolution.md)**.</span><span class="sxs-lookup"><span data-stu-id="fb92a-121">**[Design for evolution](design-for-evolution.md)**.</span></span> <span data-ttu-id="fb92a-122">Idővel minden sikeres alkalmazás változik.</span><span class="sxs-lookup"><span data-stu-id="fb92a-122">All successful applications change over time.</span></span> <span data-ttu-id="fb92a-123">A fejlődést szem előtt tartó tervezés kulcsfontosságú a folyamatos innováció szempontjából.</span><span class="sxs-lookup"><span data-stu-id="fb92a-123">An evolutionary design is key for continuous innovation.</span></span>

<span data-ttu-id="fb92a-124">**[Tervezzen a vállalkozás igényei szerint](build-for-business.md)**.</span><span class="sxs-lookup"><span data-stu-id="fb92a-124">**[Build for the needs of business](build-for-business.md)**.</span></span> <span data-ttu-id="fb92a-125">Minden tervezési döntés legyen igazolható egy üzleti igénnyel.</span><span class="sxs-lookup"><span data-stu-id="fb92a-125">Every design decision must be justified by a business requirement.</span></span>

