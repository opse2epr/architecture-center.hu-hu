---
title: 'CAF: Gyűjtse össze a digitális hagyatéki Hardverleltár-adatait'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
description: Hogyan lehet egy digitális hagyatéki készletének gyűjtéséhez.
author: BrianBlanchard
ms.date: 12/10/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.openlocfilehash: 0c68ff1e5ff51395698d37fb9b59c7a76c9479b7
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55897252"
---
# <a name="gather-inventory-data-for-a-digital-estate"></a><span data-ttu-id="121e7-103">Gyűjtse össze a digitális hagyatéki Hardverleltár-adatait</span><span class="sxs-lookup"><span data-stu-id="121e7-103">Gather inventory data for a digital estate</span></span>

<span data-ttu-id="121e7-104">Az első lépés egy készlet fejlesztésének [digitális hagyatéki tervezési](overview.md).</span><span class="sxs-lookup"><span data-stu-id="121e7-104">Developing an inventory is the first step in [Digital Estate Planning](overview.md).</span></span> <span data-ttu-id="121e7-105">Ezt a folyamatot, az IT-eszközeivel, konkrét üzleti funkciókat támogató listáját későbbi elemzés céljából, valamint ésszerűsítés lenne gyűjthetők.</span><span class="sxs-lookup"><span data-stu-id="121e7-105">In this process, a list of IT assets that support specific business functions would be collected for later analysis and rationalization.</span></span> <span data-ttu-id="121e7-106">Ez a cikk feltételezi, hogy egy elemzési alulról felfelé megközelítést tervezési igényeinek leginkább megfelelő.</span><span class="sxs-lookup"><span data-stu-id="121e7-106">This article assumes that a bottom-up approach to analysis is most appropriate for planning needs.</span></span> <span data-ttu-id="121e7-107">További információkért lásd: [megközelítések a digitális hagyatéki tervezési](./approach.md).</span><span class="sxs-lookup"><span data-stu-id="121e7-107">For more information, see [Approaches to digital estate planning](./approach.md).</span></span>

## <a name="take-inventory-of-a-digital-estate"></a><span data-ttu-id="121e7-108">Egy digitális hagyatéki felmérése</span><span class="sxs-lookup"><span data-stu-id="121e7-108">Take inventory of a digital estate</span></span>

<span data-ttu-id="121e7-109">A készlet egy digitális hagyatéki támogatása a digitális átalakulás kívánt és a megfelelő átalakulásunkhoz függően változik.</span><span class="sxs-lookup"><span data-stu-id="121e7-109">The inventory supporting a digital estate changes depending on the desired digital transformation and corresponding transformation journey.</span></span>

- <span data-ttu-id="121e7-110">**Működési átalakítási**.</span><span class="sxs-lookup"><span data-stu-id="121e7-110">**Operational transformation**.</span></span> <span data-ttu-id="121e7-111">Egy operatív átalakítás során gyakran javasolt, hogy a készlet gyűjt be ellenőrzési eszközök, amelyek hozhat létre minden olyan virtuális gépek és kiszolgálók központi listáját.</span><span class="sxs-lookup"><span data-stu-id="121e7-111">During an operational transformation, it is often advised that the inventory be collected from scanning tools which can create a centralized list of all VMs and servers.</span></span> <span data-ttu-id="121e7-112">Egyes eszközök hálózatleképezések is létrehozhat, és függőségeket, amelyek segítségével határozza meg a számítási feladatok igazítása.</span><span class="sxs-lookup"><span data-stu-id="121e7-112">Some tools can also create network mappings and dependencies, which will help define workload alignment.</span></span>

- <span data-ttu-id="121e7-113">**Növekményes átalakításában.**</span><span class="sxs-lookup"><span data-stu-id="121e7-113">**Incremental transformation.**</span></span> <span data-ttu-id="121e7-114">Növekményes átalakulása a szoftverleltár az ügyfél kezdődik.</span><span class="sxs-lookup"><span data-stu-id="121e7-114">Inventory for an incremental transformation begins with the customer.</span></span> <span data-ttu-id="121e7-115">A vásárlói élményt, elejétől a végéig leképezési remek megkezdéséhez.</span><span class="sxs-lookup"><span data-stu-id="121e7-115">Mapping the customer experience from start to finish is a good place to begin.</span></span> <span data-ttu-id="121e7-116">Igazítás, alkalmazások, API-k, adatokat és egyéb eszközök leképező hoz létre egy részletes leltározást elemzés céljából.</span><span class="sxs-lookup"><span data-stu-id="121e7-116">Aligning that map to applications, APIs, data, and other assets will create a detailed inventory for analysis.</span></span>

- <span data-ttu-id="121e7-117">**Azokat a káros átalakításában.**</span><span class="sxs-lookup"><span data-stu-id="121e7-117">**Disruptive transformation.**</span></span> <span data-ttu-id="121e7-118">Azokat a káros átalakítása a termék vagy szolgáltatás összpontosít.</span><span class="sxs-lookup"><span data-stu-id="121e7-118">Disruptive transformation focuses on the product or service.</span></span> <span data-ttu-id="121e7-119">Itt egy készlet akadályozza a piacon elérhető, és a szükséges képességek lehetőségek leképezés tartalmazhat.</span><span class="sxs-lookup"><span data-stu-id="121e7-119">From there an inventory would include a mapping of the opportunities to disrupt the market and the capabilities needed.</span></span>

## <a name="accuracy-and-completeness-of-an-inventory"></a><span data-ttu-id="121e7-120">Pontosság és a egy készlet teljességéről</span><span class="sxs-lookup"><span data-stu-id="121e7-120">Accuracy and completeness of an inventory</span></span>

<span data-ttu-id="121e7-121">Egy készlet ritkán teljes végzett az első példányát.</span><span class="sxs-lookup"><span data-stu-id="121e7-121">An inventory is seldom fully complete in its first iteration.</span></span> <span data-ttu-id="121e7-122">Erősen javasolt, hogy a Felhőstratégia csapatának tagjai különböző igazítása az érdekelt felek és a kiemelt felhasználók érvényesíteni a készlet.</span><span class="sxs-lookup"><span data-stu-id="121e7-122">It is highly advised that various members of the Cloud Strategy team align stakeholders and power users to validate the inventory.</span></span> <span data-ttu-id="121e7-123">Ha lehetséges, további eszközök, például a hálózati és függőségi elemzést küldött forgalmat eszközök azonosításához használható, de nem szerepelnek a leltározás.</span><span class="sxs-lookup"><span data-stu-id="121e7-123">When possible, additional tools like network and dependency analysis can be used to identify assets that are being sent traffic, but are not in the inventory.</span></span>

## <a name="next-steps"></a><span data-ttu-id="121e7-124">További lépések</span><span class="sxs-lookup"><span data-stu-id="121e7-124">Next steps</span></span>

<span data-ttu-id="121e7-125">Egy készlet összeállítani, és a érvényesítve, akkor is rendszerezhetők.</span><span class="sxs-lookup"><span data-stu-id="121e7-125">Once an inventory is compiled and validated, it can rationalized.</span></span> <span data-ttu-id="121e7-126">Készlet ésszerűsítés a következő lépés a digitális hagyatéki tervezési.</span><span class="sxs-lookup"><span data-stu-id="121e7-126">Inventory Rationalization is the next step to digital estate planning.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="121e7-127">A digitális hagyatéki ésszerűsítése</span><span class="sxs-lookup"><span data-stu-id="121e7-127">Rationalize the digital estate</span></span>](rationalize.md)