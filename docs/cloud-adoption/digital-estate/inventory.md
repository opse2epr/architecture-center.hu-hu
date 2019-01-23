---
title: Gyűjtse össze a digitális hagyatéki Hardverleltár-adatait
titleSuffix: Enterprise Cloud Adoption
description: Digitális hagyatéki-készlet létrehozása
author: BrianBlanchard
ms.date: 12/10/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.openlocfilehash: 5c2f270cf8de81c8a94d1f924f51611e657ed0ed
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54481796"
---
# <a name="enterprise-cloud-adoption-gather-inventory-data-for-a-digital-estate"></a><span data-ttu-id="cf075-103">Enterprise Cloud Adoption: Gyűjtse össze a digitális hagyatéki Hardverleltár-adatait</span><span class="sxs-lookup"><span data-stu-id="cf075-103">Enterprise Cloud Adoption: Gather inventory data for a digital estate</span></span>

<span data-ttu-id="cf075-104">Az első lépés egy készlet fejlesztésének [digitális hagyatéki tervezési](overview.md).</span><span class="sxs-lookup"><span data-stu-id="cf075-104">Developing an inventory is the first step in [Digital Estate Planning](overview.md).</span></span> <span data-ttu-id="cf075-105">Ezt a folyamatot, az IT-eszközeivel, konkrét üzleti funkciókat támogató listáját későbbi elemzés céljából, valamint ésszerűsítés lenne gyűjthetők.</span><span class="sxs-lookup"><span data-stu-id="cf075-105">In this process, a list of IT assets that support specific business functions would be collected for later analysis and rationalization.</span></span> <span data-ttu-id="cf075-106">Ez a cikk feltételezi, hogy egy elemzési alulról felfelé megközelítést tervezési igényeinek leginkább megfelelő.</span><span class="sxs-lookup"><span data-stu-id="cf075-106">This article assumes that a bottom-up approach to analysis is most appropriate for planning needs.</span></span> <span data-ttu-id="cf075-107">További információkért lásd: [megközelítések a digitális hagyatéki tervezési](./approach.md).</span><span class="sxs-lookup"><span data-stu-id="cf075-107">For more information, see [Approaches to digital estate planning](./approach.md).</span></span>

## <a name="how-can-a-digital-estate-be-inventoried"></a><span data-ttu-id="cf075-108">Hogyan lehet egy digitális hagyatéki leltározott?</span><span class="sxs-lookup"><span data-stu-id="cf075-108">How can a digital estate be inventoried?</span></span>

<span data-ttu-id="cf075-109">A készlet támogatása a digitális hagyatéki függően a kívánt módosítja, a digitális átalakulás és a megfelelő Átalakulásunkhoz.</span><span class="sxs-lookup"><span data-stu-id="cf075-109">The inventory supporting a digital estate changes depending on the desired Digital Transformation and corresponding Transformation Journey.</span></span>

- <span data-ttu-id="cf075-110">Működési átalakítás: Egy operatív átalakítás során gyakran javasolt, hogy a készlet gyűjt be ellenőrzési eszközök, amelyek hozhat létre minden olyan virtuális gépek és kiszolgálók központi listáját.</span><span class="sxs-lookup"><span data-stu-id="cf075-110">Operational Transformation: During an operational transformation, it is often advised that the inventory be collected from scanning tools which can create a centralized list of all VMs and servers.</span></span> <span data-ttu-id="cf075-111">Egyes eszközök hálózatleképezések is létrehozhat, és függőségeket, amelyek segítségével határozza meg a számítási feladatok igazítása.</span><span class="sxs-lookup"><span data-stu-id="cf075-111">Some tools can also create network mappings and dependencies, which will help define workload alignment.</span></span>

- <span data-ttu-id="cf075-112">Növekményes átalakítás: Növekményes átalakulása a szoftverleltár az ügyfél kezdődik.</span><span class="sxs-lookup"><span data-stu-id="cf075-112">Incremental Transformation: Inventory for an incremental transformation begins with the customer.</span></span> <span data-ttu-id="cf075-113">A vásárlói élményt, elejétől a végéig leképezési remek megkezdéséhez.</span><span class="sxs-lookup"><span data-stu-id="cf075-113">Mapping the customer experience from start to finish is a good place to begin.</span></span> <span data-ttu-id="cf075-114">Igazítás, alkalmazások, API-k, adatokat és egyéb eszközök leképező hoz létre egy részletes leltározást elemzés céljából.</span><span class="sxs-lookup"><span data-stu-id="cf075-114">Aligning that map to applications, APIs, data, and other assets will create a detailed inventory for analysis.</span></span>

- <span data-ttu-id="cf075-115">Azokat a káros átalakítás: Azokat a káros átalakítása a termék vagy szolgáltatás összpontosít.</span><span class="sxs-lookup"><span data-stu-id="cf075-115">Disruptive Transformation: Disruptive transformation focuses on the product or service.</span></span> <span data-ttu-id="cf075-116">Itt egy készlet akadályozza a piacon elérhető, és a szükséges képességek lehetőségek leképezés tartalmazhat.</span><span class="sxs-lookup"><span data-stu-id="cf075-116">From there an inventory would include a mapping of the opportunities to disrupt the market and the capabilities needed.</span></span>

## <a name="accuracy-and-completeness-of-an-inventory"></a><span data-ttu-id="cf075-117">Pontosság és a egy készlet teljességéről</span><span class="sxs-lookup"><span data-stu-id="cf075-117">Accuracy and completeness of an inventory</span></span>

<span data-ttu-id="cf075-118">Egy készlet ritkán teljes végzett az első példányát.</span><span class="sxs-lookup"><span data-stu-id="cf075-118">An inventory is seldom fully complete in its first iteration.</span></span> <span data-ttu-id="cf075-119">Erősen javasolt, hogy a Felhőstratégia csapatának tagjai különböző igazítása az érdekelt felek és a kiemelt felhasználók érvényesíteni a készlet.</span><span class="sxs-lookup"><span data-stu-id="cf075-119">It is highly advised that various members of the Cloud Strategy team align stakeholders and power users to validate the inventory.</span></span> <span data-ttu-id="cf075-120">Ha lehetséges, további eszközök, például a hálózati és függőségi elemzést küldött forgalmat eszközök azonosításához használható, de nem szerepelnek a leltározás.</span><span class="sxs-lookup"><span data-stu-id="cf075-120">When possible, additional tools like network and dependency analysis can be used to identify assets that are being sent traffic, but are not in the inventory.</span></span>

## <a name="next-steps"></a><span data-ttu-id="cf075-121">További lépések</span><span class="sxs-lookup"><span data-stu-id="cf075-121">Next steps</span></span>

<span data-ttu-id="cf075-122">Egy készlet összeállítani, és a érvényesítve, akkor is rendszerezhetők.</span><span class="sxs-lookup"><span data-stu-id="cf075-122">Once an inventory is compiled and validated, it can rationalized.</span></span> <span data-ttu-id="cf075-123">Készlet ésszerűsítés a következő lépés a digitális hagyatéki tervezési.</span><span class="sxs-lookup"><span data-stu-id="cf075-123">Inventory Rationalization is the next step to digital estate planning.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="cf075-124">A digitális hagyatéki ésszerűsítése</span><span class="sxs-lookup"><span data-stu-id="cf075-124">Rationalize the digital estate</span></span>](rationalize.md)