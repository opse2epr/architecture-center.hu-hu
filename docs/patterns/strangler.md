---
title: Strangler minta
description: Növekményesen áttelepítése egy korábbi rendszer fokozatosan cseréje nehezen funkció az új alkalmazásokkal és szolgáltatásokkal.
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: d03e8a1ef9077b6e00ea5a17423bf7e09b68111a
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
ms.locfileid: "24540985"
---
# <a name="strangler-pattern"></a><span data-ttu-id="cbbb3-103">Strangler minta</span><span class="sxs-lookup"><span data-stu-id="cbbb3-103">Strangler pattern</span></span>

<span data-ttu-id="cbbb3-104">Növekményesen áttelepítése egy korábbi rendszer fokozatosan cseréje nehezen funkció az új alkalmazásokkal és szolgáltatásokkal.</span><span class="sxs-lookup"><span data-stu-id="cbbb3-104">Incrementally migrate a legacy system by gradually replacing specific pieces of functionality with new applications and services.</span></span> <span data-ttu-id="cbbb3-105">Lecseréli a korábbi rendszer szolgáltatásait, mert az új rendszer végül lecseréli a régi rendszerre a Funkciók, a régi rendszerre strangling, és lehetővé téve az leszerelése.</span><span class="sxs-lookup"><span data-stu-id="cbbb3-105">As features from the legacy system are replaced, the new system eventually replaces all of the old system's features, strangling the old system and allowing you to decommission it.</span></span> 

## <a name="context-and-problem"></a><span data-ttu-id="cbbb3-106">A környezetben, és probléma</span><span class="sxs-lookup"><span data-stu-id="cbbb3-106">Context and problem</span></span>

<span data-ttu-id="cbbb3-107">Mivel volt épülő rendszerek élettartamát a fejlesztői eszközök, üzemeltetési technológia, és még akkor is, függvényében egyre válhat elavult.</span><span class="sxs-lookup"><span data-stu-id="cbbb3-107">As systems age, the development tools, hosting technology, and even system architectures they were built on can become increasingly obsolete.</span></span> <span data-ttu-id="cbbb3-108">Új szolgáltatások és funkciók hozzáadása, ezek az alkalmazások összetettsége jelentősen növekedhet, így nehezebb karbantartása, vagy adja hozzá az új szolgáltatások.</span><span class="sxs-lookup"><span data-stu-id="cbbb3-108">As new features and functionality are added, the complexity of these applications can increase dramatically, making them harder to maintain or add new features to.</span></span>

<span data-ttu-id="cbbb3-109">Egy összetett rendszer teljes csere hatalmas vállalkozás is lehet.</span><span class="sxs-lookup"><span data-stu-id="cbbb3-109">Completely replacing a complex system can be a huge undertaking.</span></span> <span data-ttu-id="cbbb3-110">Gyakran szüksége lesz egy fokozatos áttelepítés új rendszerre, miközben megőrzi a régi rendszerre funkciókat, amelyek még nem lettek áttelepítve kezelésére.</span><span class="sxs-lookup"><span data-stu-id="cbbb3-110">Often, you will need a gradual migration to a new system, while keeping the old system to handle features that haven't been migrated yet.</span></span> <span data-ttu-id="cbbb3-111">Azonban az alkalmazás két külön verzióit futtató azt jelenti, hogy az ügyfelek tudni, hogy hol találhatók az adott szolgáltatások.</span><span class="sxs-lookup"><span data-stu-id="cbbb3-111">However, running two separate versions of an application means that clients have to know where particular features are located.</span></span> <span data-ttu-id="cbbb3-112">Minden alkalommal, amikor egy szolgáltatás vagy szolgáltatás áttelepítése, ügyfelek frissítenie kell a helyre mutasson.</span><span class="sxs-lookup"><span data-stu-id="cbbb3-112">Every time a feature or service is migrated, clients need to be updated to point to the new location.</span></span>

## <a name="solution"></a><span data-ttu-id="cbbb3-113">Megoldás</span><span class="sxs-lookup"><span data-stu-id="cbbb3-113">Solution</span></span>

<span data-ttu-id="cbbb3-114">Növekményesen cserélje funkció nehezen új alkalmazásokhoz és szolgáltatásokhoz.</span><span class="sxs-lookup"><span data-stu-id="cbbb3-114">Incrementally replace specific pieces of functionality with new applications and services.</span></span> <span data-ttu-id="cbbb3-115">Hozzon létre egy homlokzati, amely elfogja a beérkező kéréseket a háttérrendszer örökölt címen.</span><span class="sxs-lookup"><span data-stu-id="cbbb3-115">Create a façade that intercepts requests going to the backend legacy system.</span></span> <span data-ttu-id="cbbb3-116">A homlokzati ezeket a kérelmeket, vagy az örökölt alkalmazást vagy az új szolgáltatások irányítja.</span><span class="sxs-lookup"><span data-stu-id="cbbb3-116">The façade routes these requests either to the legacy application or the new services.</span></span> <span data-ttu-id="cbbb3-117">Meglévő szolgáltatások telepíthetők át az új rendszerre fokozatosan, és a felhasználók továbbra is használhatják az ugyanazon a felületen érzékelte, hogy az áttelepítési megtörtént.</span><span class="sxs-lookup"><span data-stu-id="cbbb3-117">Existing features can be migrated to the new system gradually, and consumers can continue using the same interface, unaware that any migration has taken place.</span></span>

![](./_images/strangler.png)  

<span data-ttu-id="cbbb3-118">Ez a minta segítségével az áttelepítésből minimálisra, és fejlesztési részéről az erőfeszítés eloszlatva idő.</span><span class="sxs-lookup"><span data-stu-id="cbbb3-118">This pattern helps to minimize risk from the migration, and spread the development effort over time.</span></span> <span data-ttu-id="cbbb3-119">Biztonságosan útválasztási a felhasználók a megfelelő alkalmazás homlokzati hozzáadhat funkció az új rendszerre bármilyen ütemben van lehetősége, miközben gondoskodik az örökölt alkalmazás továbbra is működik.</span><span class="sxs-lookup"><span data-stu-id="cbbb3-119">With the façade safely routing users to the correct application, you can add functionality to the new system at whatever pace you like, while ensuring the legacy application continues to function.</span></span> <span data-ttu-id="cbbb3-120">Idővel szolgáltatások áttelepítése az új rendszerre, a korábbi rendszer van végül "strangled" és már nem szükséges.</span><span class="sxs-lookup"><span data-stu-id="cbbb3-120">Over time, as features are migrated to the new system, the legacy system is eventually "strangled" and is no longer necessary.</span></span> <span data-ttu-id="cbbb3-121">Ez a folyamat befejeződése után a korábbi rendszer biztonságosan lehet visszavonni.</span><span class="sxs-lookup"><span data-stu-id="cbbb3-121">Once this process is complete, the legacy system can safely be retired.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="cbbb3-122">Problémákat és szempontok</span><span class="sxs-lookup"><span data-stu-id="cbbb3-122">Issues and considerations</span></span>

- <span data-ttu-id="cbbb3-123">Fontolja meg az új és a korábbi rendszerek által esetleg használt szolgáltatásaikat és adataikat tárolók kezelése.</span><span class="sxs-lookup"><span data-stu-id="cbbb3-123">Consider how to handle services and data stores that are potentially used by both new and legacy systems.</span></span> <span data-ttu-id="cbbb3-124">Győződjön meg arról, hogy mindkét hozzáférhet az ezen erőforrások-mellé.</span><span class="sxs-lookup"><span data-stu-id="cbbb3-124">Make sure both can access these resources side-by-side.</span></span>
- <span data-ttu-id="cbbb3-125">Struktúra új alkalmazások és szolgáltatások oly módon, hogy azok könnyen megszerezhetik, és lecseréli a jövőben strangler áttelepítéseket.</span><span class="sxs-lookup"><span data-stu-id="cbbb3-125">Structure new applications and services in a way that they can easily be intercepted and replaced in future strangler migrations.</span></span>
- <span data-ttu-id="cbbb3-126">Bármikor az áttelepítés akkor fejeződött be, amikor a strangler homlokzati fog eltűnik vagy be egy adapter esetében a hagyományos fejleszteni.</span><span class="sxs-lookup"><span data-stu-id="cbbb3-126">At some point, when the migration is complete, the strangler façade will either go away or evolve into an adaptor for legacy clients.</span></span>
- <span data-ttu-id="cbbb3-127">Ellenőrizze, hogy a homlokzati összhangban van-e az áttelepítés.</span><span class="sxs-lookup"><span data-stu-id="cbbb3-127">Make sure the façade keeps up with the migration.</span></span>
- <span data-ttu-id="cbbb3-128">Ellenőrizze, hogy a homlokzati egyetlen pont, hibát vagy a teljesítménybeli szűk keresztmetszetek nem válik.</span><span class="sxs-lookup"><span data-stu-id="cbbb3-128">Make sure the façade doesn't become a single point of failure or a performance bottleneck.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="cbbb3-129">Mikor érdemes használni ezt a mintát</span><span class="sxs-lookup"><span data-stu-id="cbbb3-129">When to use this pattern</span></span>

<span data-ttu-id="cbbb3-130">Ezt a mintát használja, egy háttér-alkalmazás egy új architektúra fokozatosan áttelepítésekor.</span><span class="sxs-lookup"><span data-stu-id="cbbb3-130">Use this pattern when gradually migrating a back-end application to a new architecture.</span></span>

<span data-ttu-id="cbbb3-131">Ez a minta nem alkalmasak lehetnek:</span><span class="sxs-lookup"><span data-stu-id="cbbb3-131">This pattern may not be suitable:</span></span>

- <span data-ttu-id="cbbb3-132">Ha a kérelmeket a háttér-rendszer elfogta nem.</span><span class="sxs-lookup"><span data-stu-id="cbbb3-132">When requests to the back-end system cannot be intercepted.</span></span>
- <span data-ttu-id="cbbb3-133">Azon kisebb rendszerekhez nagybani helyettesítő összetettsége alacsony.</span><span class="sxs-lookup"><span data-stu-id="cbbb3-133">For smaller systems where the complexity of wholesale replacement is low.</span></span>

## <a name="related-guidance"></a><span data-ttu-id="cbbb3-134">Kapcsolódó útmutató</span><span class="sxs-lookup"><span data-stu-id="cbbb3-134">Related guidance</span></span>

- [<span data-ttu-id="cbbb3-135">Víruskereső sérülés réteg minta</span><span class="sxs-lookup"><span data-stu-id="cbbb3-135">Anti-Corruption Layer pattern</span></span>](./anti-corruption-layer.md)
- [<span data-ttu-id="cbbb3-136">Átjáró útválasztás minta</span><span class="sxs-lookup"><span data-stu-id="cbbb3-136">Gateway Routing pattern</span></span>](./gateway-routing.md)


 

