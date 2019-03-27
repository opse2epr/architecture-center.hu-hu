---
title: 'CAF: Hogy működik az Azure?'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
description: Az Azure belső működésére ismertetése
author: petertaylor9999
ms.date: 02/11/2019
ms.openlocfilehash: 724d16a810865dd947a7ade34766818c8ea525a1
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299159"
---
<!-- markdownlint-disable MD026 -->

# <a name="how-does-azure-work"></a><span data-ttu-id="7bb02-103">Hogy működik az Azure?</span><span class="sxs-lookup"><span data-stu-id="7bb02-103">How does Azure work?</span></span>

<span data-ttu-id="7bb02-104">Az Azure a Microsoft nyilvános felhő platformja.</span><span class="sxs-lookup"><span data-stu-id="7bb02-104">Azure is Microsoft's public cloud platform.</span></span> <span data-ttu-id="7bb02-105">Az Azure gyűjteménye, többek között platform (PaaS), szolgáltatott infrastruktúra (IaaS), adatbázis-szolgáltatás (DBaaS), és sok más szolgáltatás szolgáltatásokat kínál.</span><span class="sxs-lookup"><span data-stu-id="7bb02-105">Azure offers a large collection of services including platform as a service (PaaS), infrastructure as a service (IaaS), database as a service (DBaaS), and many others.</span></span> <span data-ttu-id="7bb02-106">De mi pontosan az Azure, és hogyan működik?</span><span class="sxs-lookup"><span data-stu-id="7bb02-106">But what exactly is Azure, and how does it work?</span></span>

<!-- markdownlint-disable MD034 -->

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE2ixGo]

<!-- markdownlint-enable MD034 -->

<span data-ttu-id="7bb02-107">Azure-ban, mint más felhőplatformokon támaszkodik egy technológia, más néven **virtualizálási**.</span><span class="sxs-lookup"><span data-stu-id="7bb02-107">Azure, like other cloud platforms, relies on a technology known as **virtualization**.</span></span> <span data-ttu-id="7bb02-108">A legtöbb számítógép hardvere emulálni a szoftver, mivel a legtöbb számítógép hardvere egyszerűen véglegesen vagy véglegesen pontosvesszővel titkosításúnak szilícium utasításokat egy készletét.</span><span class="sxs-lookup"><span data-stu-id="7bb02-108">Most computer hardware can be emulated in software, because most computer hardware is simply a set of instructions permanently or semi-permanently encoded in silicon.</span></span> <span data-ttu-id="7bb02-109">Az emuláció réteg, amely leképezi a szoftver utasításokat utasításait a hardver használatával, virtualizált hardverrel is hajtsa végre a szoftvereket, mintha magára a hardverkövetelmények.</span><span class="sxs-lookup"><span data-stu-id="7bb02-109">Using an emulation layer that maps software instructions to hardware instructions, virtualized hardware can execute in software as if it were the actual hardware itself.</span></span>

<span data-ttu-id="7bb02-110">Alapvetően a felhőben, hajtsa végre a virtualizált hardverrel ügyfeleik nevében egy vagy több adatközpontokban fizikai kiszolgálók egy csoportja.</span><span class="sxs-lookup"><span data-stu-id="7bb02-110">Essentially, the cloud is a set of physical servers in one or more datacenters that execute virtualized hardware on behalf of customers.</span></span> <span data-ttu-id="7bb02-111">Igen, hogyan nem a felhő létrehozása, indítása, leállítása és virtualizált hardverrel példánya több millió egyszerre törlése ügyfelek milliói számára?</span><span class="sxs-lookup"><span data-stu-id="7bb02-111">So how does the cloud create, start, stop, and delete millions of instances of virtualized hardware for millions of customers simultaneously?</span></span>

<span data-ttu-id="7bb02-112">Ennek megértéséhez Tekintsünk meg a hardver, az Adatközpont architektúráját.</span><span class="sxs-lookup"><span data-stu-id="7bb02-112">To understand this, let's look at the architecture of the hardware in the datacenter.</span></span>  <span data-ttu-id="7bb02-113">Minden adatközpontban olyan visszamegyek a kiszolgáló állványt kiszolgálók gyűjteménye.</span><span class="sxs-lookup"><span data-stu-id="7bb02-113">Within each datacenter is a collection of servers sitting in server racks.</span></span> <span data-ttu-id="7bb02-114">Minden kiszolgáló állványban található számos kiszolgáló **panelek** és a egy hálózati kapcsoló hálózati kapcsolatot és a egy power terjesztési egységek (PDU) biztosít a power.</span><span class="sxs-lookup"><span data-stu-id="7bb02-114">Each server rack contains many server **blades** as well as a network switch providing network connectivity and a power distribution unit (PDU) providing power.</span></span> <span data-ttu-id="7bb02-115">Nagyobb egységeket, más néven a állványt néha együtt vannak csoportosítva **fürtök**.</span><span class="sxs-lookup"><span data-stu-id="7bb02-115">Racks are sometimes grouped together in larger units known as **clusters**.</span></span>

<span data-ttu-id="7bb02-116">Minden állványt vagy fürt a kiszolgálók többsége vannak kijelölve, a felhasználó nevében a virtualizált hardverrel példányok futtatásához.</span><span class="sxs-lookup"><span data-stu-id="7bb02-116">Within each rack or cluster, most of the servers are designated to run these virtualized hardware instances on behalf of the user.</span></span> <span data-ttu-id="7bb02-117">Azonban a kiszolgálók számos felhőjét felügyeleti szoftverek a hálóvezérlő néven.</span><span class="sxs-lookup"><span data-stu-id="7bb02-117">However, a number of the servers run cloud management software known as a fabric controller.</span></span> <span data-ttu-id="7bb02-118">A **hálóvezérlő** egy elosztott alkalmazás számos feladatokkal.</span><span class="sxs-lookup"><span data-stu-id="7bb02-118">The **fabric controller** is a distributed application with many responsibilities.</span></span> <span data-ttu-id="7bb02-119">Foglalja le a szolgáltatások, a kiszolgáló és a rajta futó szolgáltatások állapotát figyeli, és kijavítja a kiszolgálók, amikor meghiúsulnak.</span><span class="sxs-lookup"><span data-stu-id="7bb02-119">It allocates services, monitors the health of the server and the services running on it, and heals servers when they fail.</span></span>

<span data-ttu-id="7bb02-120">Minden példánya a hálóvezérlő csatlakoztatva van egy másik hárompéldányos készletet felhőalapú vezénylési szoftvert, általában más néven futtató kiszolgálók egy **előtér**.</span><span class="sxs-lookup"><span data-stu-id="7bb02-120">Each instance of the fabric controller is connected to another set of servers running cloud orchestration software, typically known as a **front end**.</span></span> <span data-ttu-id="7bb02-121">Az előtér üzemelteti a webes szolgáltatásokat, RESTful API-k és a felhő hajtja végre a függvények használt belső Azure-adatbázisok.</span><span class="sxs-lookup"><span data-stu-id="7bb02-121">The front end hosts the web services, RESTful APIs, and internal Azure databases used for all functions the cloud performs.</span></span>

<span data-ttu-id="7bb02-122">Így például az előtér tárolja-e a szolgáltatásokat, amelyek az Azure-erőforrásokat lefoglalni a felhasználói kérések kezelésére [virtuális hálózatok][vnet], [virtuális gépek] [ vms], és a szolgáltatásokat, mint például [Cosmos DB][cosmosdb].</span><span class="sxs-lookup"><span data-stu-id="7bb02-122">For example, the front end hosts the services that handle customer requests to allocate Azure resources such as [virtual networks][vnet], [virtual machines][vms], and services like [Cosmos DB][cosmosdb].</span></span> <span data-ttu-id="7bb02-123">Az előtér először ellenőrzi a felhasználó, és ellenőrzi a felhasználó jogosult-e a kért erőforrások lefoglalása.</span><span class="sxs-lookup"><span data-stu-id="7bb02-123">First, the front end validates the user and verifies the user is authorized to allocate the requested resources.</span></span> <span data-ttu-id="7bb02-124">Ha igen, az előtér olvas, keresse meg a kiszolgáló állvány elegendő kapacitással az adatbázis, és majd arra utasítja a hálóvezérlő a az állványra szerelt, az erőforrás lefoglalásához.</span><span class="sxs-lookup"><span data-stu-id="7bb02-124">If so, the front end consults a database to locate a server rack with sufficient capacity, and then instructs the fabric controller on the rack to allocate the resource.</span></span>

<span data-ttu-id="7bb02-125">Tehát nagyon egyszerűen Azure gyűjteménye hatalmas, kiszolgálók és a hálózati eszközt, valamint összetett elosztott alkalmazások, amelyekkel a konfiguráció és a művelet a virtualizált hardver- és az ezeken a kiszolgálókon.</span><span class="sxs-lookup"><span data-stu-id="7bb02-125">So, very simply, Azure is a huge collection of servers and networking hardware, along with a complex set of distributed applications that orchestrate the configuration and operation of the virtualized hardware and software on those servers.</span></span> <span data-ttu-id="7bb02-126">És ennek vezénylését, amely az Azure ezért sokoldalúak - felhasználók nem lesznek karbantartásáért felelős és frissítése hardver, az Azure összes ezt a háttérben hajtja végre.</span><span class="sxs-lookup"><span data-stu-id="7bb02-126">And it is this orchestration that makes Azure so powerful - users are no longer responsible for maintaining and upgrading hardware, Azure does all this behind the scenes.</span></span>

## <a name="next-steps"></a><span data-ttu-id="7bb02-127">További lépések</span><span class="sxs-lookup"><span data-stu-id="7bb02-127">Next steps</span></span>

<span data-ttu-id="7bb02-128">Most, hogy megismerkedett az Azure belső működésére, felhőalapú erőforrás-szabályozása ismerje.</span><span class="sxs-lookup"><span data-stu-id="7bb02-128">Now that you understand the internal functioning of Azure, learn about cloud resource governance.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="7bb02-129">További tudnivalók az erőforrás-szabályozása</span><span class="sxs-lookup"><span data-stu-id="7bb02-129">Learn about resource governance</span></span>](what-is-governance.md)

<!-- Links -->

[cosmosdb]: /azure/cosmos-db/introduction
[docs-add-users-to-aad]: /azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json
[vms]: /azure/virtual-machines/
[vnet]: /azure/virtual-network/virtual-networks-overview
