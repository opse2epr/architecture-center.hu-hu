---
title: Leépítő minta
titleSuffix: Cloud Design Patterns
description: Növekményesen migrálhat egy korábbi rendszert oly módon, hogy egyes funkciódarabokat fokozatosan új alkalmazásokra és szolgáltatásokra cserél.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: f139d368c98256c0190753930983a47df81a5134
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54480558"
---
# <a name="strangler-pattern"></a><span data-ttu-id="00a9e-104">Leépítő minta</span><span class="sxs-lookup"><span data-stu-id="00a9e-104">Strangler pattern</span></span>

<span data-ttu-id="00a9e-105">Növekményesen migrálhat egy korábbi rendszert oly módon, hogy egyes funkciódarabokat fokozatosan új alkalmazásokra és szolgáltatásokra cserél.</span><span class="sxs-lookup"><span data-stu-id="00a9e-105">Incrementally migrate a legacy system by gradually replacing specific pieces of functionality with new applications and services.</span></span> <span data-ttu-id="00a9e-106">A korábbi rendszer funkcióinak lecserélésével idővel az új rendszer felváltja a régi rendszer összes funkcióját, leépítve a régi rendszert, és lehetővé téve annak leszerelését.</span><span class="sxs-lookup"><span data-stu-id="00a9e-106">As features from the legacy system are replaced, the new system eventually replaces all of the old system's features, strangling the old system and allowing you to decommission it.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="00a9e-107">Kontextus és probléma</span><span class="sxs-lookup"><span data-stu-id="00a9e-107">Context and problem</span></span>

<span data-ttu-id="00a9e-108">A rendszerek öregedésével az azok alapjául szolgáló fejlesztési eszközök, üzemeltetési technológia és a rendszerarchitektúrák is jelentősen elavulttá válhatnak.</span><span class="sxs-lookup"><span data-stu-id="00a9e-108">As systems age, the development tools, hosting technology, and even system architectures they were built on can become increasingly obsolete.</span></span> <span data-ttu-id="00a9e-109">Új szolgáltatások és funkciók hozzáadásával ezen alkalmazások összetettsége jelentősen nőhet, így a karbantartásuk, illetve a további funkciók hozzáadása nehezebb lesz.</span><span class="sxs-lookup"><span data-stu-id="00a9e-109">As new features and functionality are added, the complexity of these applications can increase dramatically, making them harder to maintain or add new features to.</span></span>

<span data-ttu-id="00a9e-110">Egy összetett rendszer teljes cseréje hatalmas feladat lehet.</span><span class="sxs-lookup"><span data-stu-id="00a9e-110">Completely replacing a complex system can be a huge undertaking.</span></span> <span data-ttu-id="00a9e-111">Gyakran fokozatosan kell áttérni az új rendszerre a régi rendszer megtartása mellett a még nem migrált funkciók kezelése céljából.</span><span class="sxs-lookup"><span data-stu-id="00a9e-111">Often, you will need a gradual migration to a new system, while keeping the old system to handle features that haven't been migrated yet.</span></span> <span data-ttu-id="00a9e-112">Egy alkalmazás két különálló verziójának futtatásakor azonban az ügyfeleknek tudniuk kell, hol találhatóak az egyes funkciók.</span><span class="sxs-lookup"><span data-stu-id="00a9e-112">However, running two separate versions of an application means that clients have to know where particular features are located.</span></span> <span data-ttu-id="00a9e-113">Funkciók vagy szolgáltatások migrálásakor az ügyfeleket minden alkalommal frissíteni kell, hogy az új helyre mutassanak.</span><span class="sxs-lookup"><span data-stu-id="00a9e-113">Every time a feature or service is migrated, clients need to be updated to point to the new location.</span></span>

## <a name="solution"></a><span data-ttu-id="00a9e-114">Megoldás</span><span class="sxs-lookup"><span data-stu-id="00a9e-114">Solution</span></span>

<span data-ttu-id="00a9e-115">Fokozatosan cserélje le az egyes funkciókat új alkalmazásokra és szolgáltatásokra.</span><span class="sxs-lookup"><span data-stu-id="00a9e-115">Incrementally replace specific pieces of functionality with new applications and services.</span></span> <span data-ttu-id="00a9e-116">Egy előtérrendszer létrehozásával elfoghatja a korábbi háttérrendszernek címzett kérelmeket.</span><span class="sxs-lookup"><span data-stu-id="00a9e-116">Create a façade that intercepts requests going to the backend legacy system.</span></span> <span data-ttu-id="00a9e-117">Az előtérrendszer ezeket a kérelmeket a korábbi alkalmazáshoz vagy az új szolgáltatásokhoz irányítja.</span><span class="sxs-lookup"><span data-stu-id="00a9e-117">The façade routes these requests either to the legacy application or the new services.</span></span> <span data-ttu-id="00a9e-118">A meglévő szolgáltatások fokozatosan migrálhatóak az új rendszerre, és a fogyasztók továbbra is ugyanazt a felületet használhatják anélkül, hogy bármit érzékelnének a migrálásból.</span><span class="sxs-lookup"><span data-stu-id="00a9e-118">Existing features can be migrated to the new system gradually, and consumers can continue using the same interface, unaware that any migration has taken place.</span></span>

![A Leépítő minta ábrája](./_images/strangler.png)

<span data-ttu-id="00a9e-120">Ez a minta segít minimálisra csökkenteni a migrálással járó kockázatokat, és időben jobban elosztja a fejlesztési folyamatot.</span><span class="sxs-lookup"><span data-stu-id="00a9e-120">This pattern helps to minimize risk from the migration, and spread the development effort over time.</span></span> <span data-ttu-id="00a9e-121">Miközben az előtérrendszer biztonságosan a megfelelő alkalmazáshoz irányítja a felhasználókat, tetszés szerint ütemezve adhat funkciókat az új rendszerhez a korábbi alkalmazás működésének fenntartása mellett.</span><span class="sxs-lookup"><span data-stu-id="00a9e-121">With the façade safely routing users to the correct application, you can add functionality to the new system at whatever pace you like, while ensuring the legacy application continues to function.</span></span> <span data-ttu-id="00a9e-122">A funkciók új rendszerre való migrálása idővel „leépíti” a régi rendszer funkcionalitását, és nem lesz rá többet szükség.</span><span class="sxs-lookup"><span data-stu-id="00a9e-122">Over time, as features are migrated to the new system, the legacy system is eventually "strangled" and is no longer necessary.</span></span> <span data-ttu-id="00a9e-123">A folyamat befejeződése után a régi rendszer biztonságosan megszüntethető.</span><span class="sxs-lookup"><span data-stu-id="00a9e-123">Once this process is complete, the legacy system can safely be retired.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="00a9e-124">Problémák és megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="00a9e-124">Issues and considerations</span></span>

- <span data-ttu-id="00a9e-125">Fontolja meg, hogyan kívánja kezelni azokat a szolgáltatásokat és adattárakat, amelyeket esetlegesen az új és a régi rendszer is használhat.</span><span class="sxs-lookup"><span data-stu-id="00a9e-125">Consider how to handle services and data stores that are potentially used by both new and legacy systems.</span></span> <span data-ttu-id="00a9e-126">Biztosítsa, hogy a két rendszer egyidejűleg hozzáférhessen ezekhez az erőforrásokhoz.</span><span class="sxs-lookup"><span data-stu-id="00a9e-126">Make sure both can access these resources side-by-side.</span></span>
- <span data-ttu-id="00a9e-127">Úgy alakítsa ki az új alkalmazások és szolgáltatások szerkezetét, hogy könnyen elfoghatóak és lecserélhetőek legyenek a jövőbeli leépítési migrálások során.</span><span class="sxs-lookup"><span data-stu-id="00a9e-127">Structure new applications and services in a way that they can easily be intercepted and replaced in future strangler migrations.</span></span>
- <span data-ttu-id="00a9e-128">A migrálás befejeződése után a leépítési előtérrendszer megszüntethető, vagy régebbi ügyfelek adapterévé alakítható.</span><span class="sxs-lookup"><span data-stu-id="00a9e-128">At some point, when the migration is complete, the strangler façade will either go away or evolve into an adaptor for legacy clients.</span></span>
- <span data-ttu-id="00a9e-129">Gondoskodjon róla, hogy az előtérrendszer lépést tartson a migrálással.</span><span class="sxs-lookup"><span data-stu-id="00a9e-129">Make sure the façade keeps up with the migration.</span></span>
- <span data-ttu-id="00a9e-130">Ügyeljen rá, hogy az előtérrendszer ne váljon kritikus meghibásodási ponttá, és ne képezhessen teljesítménybeli szűk keresztmetszetet.</span><span class="sxs-lookup"><span data-stu-id="00a9e-130">Make sure the façade doesn't become a single point of failure or a performance bottleneck.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="00a9e-131">Mikor érdemes ezt a mintát használni?</span><span class="sxs-lookup"><span data-stu-id="00a9e-131">When to use this pattern</span></span>

<span data-ttu-id="00a9e-132">Ezt a mintát háttéralkalmazások új architektúrákra való fokozatos migrálásához használhatja.</span><span class="sxs-lookup"><span data-stu-id="00a9e-132">Use this pattern when gradually migrating a back-end application to a new architecture.</span></span>

<span data-ttu-id="00a9e-133">Nem érdemes ezt a mintát használni, ha:</span><span class="sxs-lookup"><span data-stu-id="00a9e-133">This pattern may not be suitable:</span></span>

- <span data-ttu-id="00a9e-134">A háttérrendszernek küldött kérelmek nem foghatók el.</span><span class="sxs-lookup"><span data-stu-id="00a9e-134">When requests to the back-end system cannot be intercepted.</span></span>
- <span data-ttu-id="00a9e-135">Kisebb rendszerek esetében, ahol a teljes körű lecserélés összetettsége alacsony.</span><span class="sxs-lookup"><span data-stu-id="00a9e-135">For smaller systems where the complexity of wholesale replacement is low.</span></span>

## <a name="related-guidance"></a><span data-ttu-id="00a9e-136">Kapcsolódó útmutatók</span><span class="sxs-lookup"><span data-stu-id="00a9e-136">Related guidance</span></span>

- <span data-ttu-id="00a9e-137">A Martin Fowler blogbejegyzés [StranglerApplication](https://www.martinfowler.com/bliki/StranglerApplication.html)</span><span class="sxs-lookup"><span data-stu-id="00a9e-137">Martin Fowler's blog post on [StranglerApplication](https://www.martinfowler.com/bliki/StranglerApplication.html)</span></span>
