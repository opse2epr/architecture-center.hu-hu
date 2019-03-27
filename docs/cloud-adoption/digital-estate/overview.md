---
title: 'CAF: Mi az a digitális tulajdon?'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
description: Mi az a digitális tulajdon?
author: BrianBlanchard
ms.date: 12/10/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.openlocfilehash: d23bdbdd98226e8b9cdb9bbb5fefa5a918a498d8
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298450"
---
<!-- markdownlint-disable MD026 -->

# <a name="caf-what-is-a-digital-estate"></a><span data-ttu-id="3237c-103">CAF: Mi az a digitális tulajdon?</span><span class="sxs-lookup"><span data-stu-id="3237c-103">CAF: What is a digital estate?</span></span>

<span data-ttu-id="3237c-104">Minden modern vállalati digitális hagyatéki valamilyen rendelkezik.</span><span class="sxs-lookup"><span data-stu-id="3237c-104">Every modern company has some form of digital estate.</span></span> <span data-ttu-id="3237c-105">Sokkal hasonlóan egy fizikai hagyatéki digitális szűrőként is egy absztrakt referencia képzés résztvevői hasznos képességekkel, számos tulajdonában lévő eszközök.</span><span class="sxs-lookup"><span data-stu-id="3237c-105">Much like a physical estate, a digital estate is an abstract reference to a number of tangible, owned assets.</span></span> <span data-ttu-id="3237c-106">A digitális hagyatéki azok az eszközök állnak virtuális gépek (VM), kiszolgálók, alkalmazások, adatok és így tovább.</span><span class="sxs-lookup"><span data-stu-id="3237c-106">In a digital estate, those assets are comprised of virtual machines (VMs), servers, applications, data, and so on.</span></span> <span data-ttu-id="3237c-107">Alapvetően egy digitális hagyatéki gyűjteménye a kiemelt üzleti folyamatokat és támogató operatív IT-eszközeivel.</span><span class="sxs-lookup"><span data-stu-id="3237c-107">Essentially, a digital estate is the collection of IT assets that power business processes and supporting operations.</span></span>

<span data-ttu-id="3237c-108">A digitális hagyatéki fontosságát legnyilvánvalóbb tervezési és a digitális átalakítás folyamatában végrehajtása során.</span><span class="sxs-lookup"><span data-stu-id="3237c-108">The importance of a digital estate is most obvious during planning and execution of Digital Transformation efforts.</span></span> <span data-ttu-id="3237c-109">Átalakítás Journey, során a digitális hagyatéki hogyan Felhőstratégia csapatok képezze le az üzleti eredmények terveket és erőfeszítések műszaki felszabadítása érdekében.</span><span class="sxs-lookup"><span data-stu-id="3237c-109">During transformation journeys, the digital estate is how Cloud Strategy teams map the business outcomes to release plans and technical efforts.</span></span> <span data-ttu-id="3237c-110">Most, hogy az összes kezdődik-készlet és a digitális eszközeit mérése a szervezet tulajdonosa.</span><span class="sxs-lookup"><span data-stu-id="3237c-110">That all starts with an inventory and measurement of the digital assets the organization owns today.</span></span>

## <a name="how-can-a-digital-estate-be-measured"></a><span data-ttu-id="3237c-111">Hogyan lehet mérni a digitális hagyatéki?</span><span class="sxs-lookup"><span data-stu-id="3237c-111">How can a digital estate be measured?</span></span>

<span data-ttu-id="3237c-112">A digitális hagyatéki mérése a kívánt üzleti eredmények függően változik.</span><span class="sxs-lookup"><span data-stu-id="3237c-112">The measurement of a digital estate changes depending on the desired business outcomes.</span></span>

- <span data-ttu-id="3237c-113">**Infrastruktúra-áttelepítések**.</span><span class="sxs-lookup"><span data-stu-id="3237c-113">**Infrastructure migrations**.</span></span> <span data-ttu-id="3237c-114">Amikor egy szervezet aktív elérésű, és arra törekszik, hogy optimalizálja a költség, a üzemeltetési folyamatokat, a rugalmasságot vagy a műveletek optimalizálása más aspektusait, a digitális hagyatéki elsősorban virtuális gépek, kiszolgálók és a kapcsolódó számítási feladatok.</span><span class="sxs-lookup"><span data-stu-id="3237c-114">When an organization is inward facing and seeks to optimize cost, operational processes, agility, or other aspects of optimizing operations, the digital estate focuses on VMs, servers, and supporting workloads.</span></span>

- <span data-ttu-id="3237c-115">**Alkalmazásinnovációt**.</span><span class="sxs-lookup"><span data-stu-id="3237c-115">**Application innovation**.</span></span> <span data-ttu-id="3237c-116">Ügyfél-obsessed átalakítások a fókuszt eltér egy kicsit.</span><span class="sxs-lookup"><span data-stu-id="3237c-116">For customer-obsessed transformations, the lens is a bit different.</span></span> <span data-ttu-id="3237c-117">A fókuszt az alkalmazások, az API-k és a tranzakciós adatok, amelyek támogatják az ügyfelek kell helyezni.</span><span class="sxs-lookup"><span data-stu-id="3237c-117">The focus should be placed in the applications, APIs, and transactional data that support the customers.</span></span> <span data-ttu-id="3237c-118">Virtuális gépek és a hálózati berendezések gyakran kisebb fókusz történik.</span><span class="sxs-lookup"><span data-stu-id="3237c-118">VMs and network appliances are often of less focus.</span></span>

- <span data-ttu-id="3237c-119">**Az adatvezérelt innovációt**.</span><span class="sxs-lookup"><span data-stu-id="3237c-119">**Data-driven innovation**.</span></span> <span data-ttu-id="3237c-120">A mai digitális modellvezérelt piacon meglehetősen nehéz indítsa el az új termék vagy szolgáltatás az adatok erős alaprendszert nélkül.</span><span class="sxs-lookup"><span data-stu-id="3237c-120">In today's digitally driven market, it's difficult to launch a new product or service without a strong foundation in data.</span></span> <span data-ttu-id="3237c-121">Azokat a káros átalakítás során a célja az adatok a silók a további a szervezeten belül.</span><span class="sxs-lookup"><span data-stu-id="3237c-121">During disruptive transformation, the focus is more on the silos of data across the organization.</span></span>

<span data-ttu-id="3237c-122">Egy szervezet tisztában van azzal a legfontosabb képernyő átalakítás, miután sokkal könnyebben kezelhető digitális hagyatéki tervezési válik.</span><span class="sxs-lookup"><span data-stu-id="3237c-122">Once an organization understands the most important form of transformation, digital estate planning becomes much easier to manage.</span></span>

> [!TIP]
> <span data-ttu-id="3237c-123">Minden átalakítási típusú azokkal a három nézet mérhető.</span><span class="sxs-lookup"><span data-stu-id="3237c-123">Each type of transformation could be measured with any of the three views.</span></span> <span data-ttu-id="3237c-124">További szokás a vállalatok számára, hogy az összes három átalakítások végre párhuzamosan.</span><span class="sxs-lookup"><span data-stu-id="3237c-124">Further, it is common for companies to execute all three transformations in parallel.</span></span> <span data-ttu-id="3237c-125">Erősen ajánlott, amely a vezetői és szilárdan igazítani kapcsolatos legfontosabb üzleti siker érdekében az átalakítás Felhőstratégia csapat.</span><span class="sxs-lookup"><span data-stu-id="3237c-125">It is highly suggested that the leadership and Cloud Strategy team be firmly aligned regarding the transformation that is most important for business success.</span></span> <span data-ttu-id="3237c-126">A közös nyelvi és metrikák alapját a ismertetése erre a célra több kezdeményezések között.</span><span class="sxs-lookup"><span data-stu-id="3237c-126">That understanding will serve as the basis for common language and metrics across multiple initiatives.</span></span>

## <a name="how-can-a-financial-model-be-updated-to-reflect-the-digital-estate"></a><span data-ttu-id="3237c-127">Hogyan lehet egy pénzügyi modellt frissíteni fogja a digitális hagyatéki?</span><span class="sxs-lookup"><span data-stu-id="3237c-127">How can a financial model be updated to reflect the digital estate?</span></span>

<span data-ttu-id="3237c-128">A digitális hagyatéki elemzésének vezérlik a felhő bevezetésének tevékenységek.</span><span class="sxs-lookup"><span data-stu-id="3237c-128">An analysis of the digital estate will drive cloud adoption activities.</span></span> <span data-ttu-id="3237c-129">Pénzügyi modelleket azáltal, hogy felhőalapú költségszámítási modelljeit, a visszatérési viszont vonják ráta (ROI), emellett tájékoztatja.</span><span class="sxs-lookup"><span data-stu-id="3237c-129">It will also inform financial models by providing cloud costing models, which will in turn drive the return on investment (ROI).</span></span>

<span data-ttu-id="3237c-130">A digitális hagyatéki elemzés végrehajtásához hajtsa végre az alábbi lépéseket:</span><span class="sxs-lookup"><span data-stu-id="3237c-130">To complete the digital estate analysis, perform the following steps:</span></span>

1. [<span data-ttu-id="3237c-131">Elemzési módszer meghatározása</span><span class="sxs-lookup"><span data-stu-id="3237c-131">Determine analysis approach</span></span>](approach.md)
1. [<span data-ttu-id="3237c-132">Aktuális állapot leltáradatokat tudjanak gyűjteni</span><span class="sxs-lookup"><span data-stu-id="3237c-132">Collect current state inventory</span></span>](inventory.md)
1. [<span data-ttu-id="3237c-133">Az eszközök a digitális hagyatéki ésszerűsítése</span><span class="sxs-lookup"><span data-stu-id="3237c-133">Rationalize the assets in the digital estate</span></span>](rationalize.md)
1. [<span data-ttu-id="3237c-134">Eszközök ajánlatok díjszabása kiszámításához felhőbe igazítása</span><span class="sxs-lookup"><span data-stu-id="3237c-134">Align assets to cloud offerings to calculate pricing</span></span>](calculate.md)

<span data-ttu-id="3237c-135">Pénzügyi modelleket és áttelepítési sorait a rendszerezhetők és árú hagyatéki megfelelően módosítható.</span><span class="sxs-lookup"><span data-stu-id="3237c-135">Financial models and migration backlogs can be modified to reflect the rationalized and priced estate.</span></span>

## <a name="next-steps"></a><span data-ttu-id="3237c-136">További lépések</span><span class="sxs-lookup"><span data-stu-id="3237c-136">Next steps</span></span>

<span data-ttu-id="3237c-137">Digitális hagyatéki tervezési elkezdése előtt meg kell határoznia, milyen megközelítést használni.</span><span class="sxs-lookup"><span data-stu-id="3237c-137">Before digital estate planning can begin, you must determine what approach to use.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="3237c-138">A digitális hagyatéki tervezési megközelítés</span><span class="sxs-lookup"><span data-stu-id="3237c-138">Approaches to digital estate planning</span></span>](approach.md)