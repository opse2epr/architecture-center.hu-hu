---
title: Használjon felügyelt szolgáltatásokat
titleSuffix: Azure Application Architecture Guide
description: Ha lehetséges, használjon platformot (PaaS) szolgáltatás infrastruktúrán szolgáltatás (IaaS).
author: MikeWasson
ms.date: 08/30/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19
ms.openlocfilehash: f08914e1eacc4f02fdb16093fe590f46249e3da3
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54485808"
---
# <a name="use-managed-services"></a><span data-ttu-id="96c0c-103">Használjon felügyelt szolgáltatásokat</span><span class="sxs-lookup"><span data-stu-id="96c0c-103">Use managed services</span></span>

## <a name="when-possible-use-platform-as-a-service-paas-rather-than-infrastructure-as-a-service-iaas"></a><span data-ttu-id="96c0c-104">Amikor lehetséges, a szolgáltatásként nyújtott infrastruktúra (IaaS) helyett használjon szolgáltatásként nyújtott platformot (PaaS)</span><span class="sxs-lookup"><span data-stu-id="96c0c-104">When possible, use platform as a service (PaaS) rather than infrastructure as a service (IaaS)</span></span>

<span data-ttu-id="96c0c-105">Az IaaS olyan, mintha egy doboznyi alkatrész lenne.</span><span class="sxs-lookup"><span data-stu-id="96c0c-105">IaaS is like having a box of parts.</span></span> <span data-ttu-id="96c0c-106">Bármit létrehozhat, de saját magának kell összeállítania.</span><span class="sxs-lookup"><span data-stu-id="96c0c-106">You can build anything, but you have to assemble it yourself.</span></span> <span data-ttu-id="96c0c-107">A felügyelt szolgáltatások konfigurálása és felügyelete egyszerűbb.</span><span class="sxs-lookup"><span data-stu-id="96c0c-107">Managed services are easier to configure and administer.</span></span> <span data-ttu-id="96c0c-108">Nem kell virtuális gépeket üzembe helyeznie, virtuális hálózatokat beállítania, javításokat és frissítéseket kezelnie, illetve egyéb olyan feladatokat elvégeznie, amelyek a szoftverek virtuális gépen való futtatásához kapcsolódnak.</span><span class="sxs-lookup"><span data-stu-id="96c0c-108">You don't need to provision VMs, set up VNets, manage patches and updates, and all of the other overhead associated with running software on a VM.</span></span>

<span data-ttu-id="96c0c-109">Tegyük fel, hogy az alkalmazásnak üzenetsorra van szüksége.</span><span class="sxs-lookup"><span data-stu-id="96c0c-109">For example, suppose your application needs a message queue.</span></span> <span data-ttu-id="96c0c-110">Beállíthat saját üzenetküldési szolgáltatást egy virtuális gépen a RabbitMQ vagy hasonló szolgáltatás segítségével.</span><span class="sxs-lookup"><span data-stu-id="96c0c-110">You could set up your own messaging service on a VM, using something like RabbitMQ.</span></span> <span data-ttu-id="96c0c-111">Az Azure Service Bus azonban már biztosít egy megbízható üzenetküldési szolgáltatást, és a beállítása is egyszerűbb.</span><span class="sxs-lookup"><span data-stu-id="96c0c-111">But Azure Service Bus already provides reliable messaging as service, and it's simpler to set up.</span></span> <span data-ttu-id="96c0c-112">Egyszerűen hozzon létre egy Service Bus-névteret (ez az üzembehelyezési szkripttel is elvégezhető), majd hívja meg a Service Bust az ügyfél SDK használatával.</span><span class="sxs-lookup"><span data-stu-id="96c0c-112">Just create a Service Bus namespace (which can be done as part of a deployment script) and then call Service Bus using the client SDK.</span></span>

<span data-ttu-id="96c0c-113">Természetesen az alkalmazásnak lehetnek konkrét követelményei, amelyek miatt az IaaS módszer megfelelőbb lehet.</span><span class="sxs-lookup"><span data-stu-id="96c0c-113">Of course, your application may have specific requirements that make an IaaS approach more suitable.</span></span> <span data-ttu-id="96c0c-114">Azonban még az IaaS-alapú alkalmazások esetén is érdemes olyan helyeket keresnie, ahol természetes lehet a felügyelt szolgáltatások használata.</span><span class="sxs-lookup"><span data-stu-id="96c0c-114">However, even if your application is based on IaaS, look for places where it may be natural to incorporate managed services.</span></span> <span data-ttu-id="96c0c-115">Ezek közé tartoznak az üzenetsorok, a gyorsítótár és az adattárolás.</span><span class="sxs-lookup"><span data-stu-id="96c0c-115">These include cache, queues, and data storage.</span></span>

| <span data-ttu-id="96c0c-116">Az alábbiak futtatása helyett...</span><span class="sxs-lookup"><span data-stu-id="96c0c-116">Instead of running...</span></span> | <span data-ttu-id="96c0c-117">Fontolja meg a következőket...</span><span class="sxs-lookup"><span data-stu-id="96c0c-117">Consider using...</span></span> |
|-----------------------|-------------|
| <span data-ttu-id="96c0c-118">Active Directory</span><span class="sxs-lookup"><span data-stu-id="96c0c-118">Active Directory</span></span> | <span data-ttu-id="96c0c-119">Azure Active Directory tartományi szolgáltatások</span><span class="sxs-lookup"><span data-stu-id="96c0c-119">Azure Active Directory Domain Services</span></span> |
| <span data-ttu-id="96c0c-120">Elasticsearch</span><span class="sxs-lookup"><span data-stu-id="96c0c-120">Elasticsearch</span></span> | <span data-ttu-id="96c0c-121">Azure Search</span><span class="sxs-lookup"><span data-stu-id="96c0c-121">Azure Search</span></span> |
| <span data-ttu-id="96c0c-122">Hadoop</span><span class="sxs-lookup"><span data-stu-id="96c0c-122">Hadoop</span></span> | <span data-ttu-id="96c0c-123">HDInsight</span><span class="sxs-lookup"><span data-stu-id="96c0c-123">HDInsight</span></span> |
| <span data-ttu-id="96c0c-124">IIS</span><span class="sxs-lookup"><span data-stu-id="96c0c-124">IIS</span></span> | <span data-ttu-id="96c0c-125">App Service</span><span class="sxs-lookup"><span data-stu-id="96c0c-125">App Service</span></span> |
| <span data-ttu-id="96c0c-126">MongoDB</span><span class="sxs-lookup"><span data-stu-id="96c0c-126">MongoDB</span></span> | <span data-ttu-id="96c0c-127">Cosmos DB</span><span class="sxs-lookup"><span data-stu-id="96c0c-127">Cosmos DB</span></span> |
| <span data-ttu-id="96c0c-128">Redis</span><span class="sxs-lookup"><span data-stu-id="96c0c-128">Redis</span></span> | <span data-ttu-id="96c0c-129">Azure Redis Cache</span><span class="sxs-lookup"><span data-stu-id="96c0c-129">Azure Redis Cache</span></span> |
| <span data-ttu-id="96c0c-130">SQL Server</span><span class="sxs-lookup"><span data-stu-id="96c0c-130">SQL Server</span></span> | <span data-ttu-id="96c0c-131">Azure SQL Database</span><span class="sxs-lookup"><span data-stu-id="96c0c-131">Azure SQL Database</span></span> |
