---
title: Tervezési minták felhőkhöz
description: Tervezési minták felhőkhöz a Microsoft Azure-ban
keywords: Azure
ms.openlocfilehash: 4747c896fc6fc5866be782d76c5290d6b49ad451
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/06/2018
ms.locfileid: "30848257"
---
# <a name="cloud-design-patterns"></a><span data-ttu-id="bbd43-104">Tervezési minták felhőkhöz</span><span class="sxs-lookup"><span data-stu-id="bbd43-104">Cloud Design Patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="bbd43-105">Ezek a tervezési minták hasznosak a megbízható, skálázható és biztonságos felhőbeli alkalmazások létrehozásához.</span><span class="sxs-lookup"><span data-stu-id="bbd43-105">These design patterns are useful for building reliable, scalable, secure applications in the cloud.</span></span>

<span data-ttu-id="bbd43-106">Mindegyik minta ismerteti az általa kezelt problémát, a minta alkalmazásának szempontjait és egy, a Microsoft Azure-on alapuló példát.</span><span class="sxs-lookup"><span data-stu-id="bbd43-106">Each pattern describes the problem that the pattern addresses, considerations for applying the pattern, and an example based on Microsoft Azure.</span></span> <span data-ttu-id="bbd43-107">A legtöbb minta tartalmaz kódmintákat vagy kódrészleteket is, amelyek a minta Azure-on való alkalmazását mutatják be.</span><span class="sxs-lookup"><span data-stu-id="bbd43-107">Most of the patterns include code samples or snippets that show how to implement the pattern on Azure.</span></span> <span data-ttu-id="bbd43-108">A legtöbb minta ugyanakkor minden elosztott rendszeren alkalmazható, függetlenül attól, hogy az az Azure-on vagy más felhőplatformon üzemel-e.</span><span class="sxs-lookup"><span data-stu-id="bbd43-108">However, most of the patterns are relevant to any distributed system, whether hosted on Azure or on other cloud platforms.</span></span>

## <a name="problem-areas-in-the-cloud"></a><span data-ttu-id="bbd43-109">Problémás területek a felhőben</span><span class="sxs-lookup"><span data-stu-id="bbd43-109">Problem areas in the cloud</span></span>

<ul id="categories" class="panel">
<span data-ttu-id="bbd43-110">{%- for category in categories %}</span><span class="sxs-lookup"><span data-stu-id="bbd43-110">{%- for category in categories %}</span></span>
    <li>
    <span data-ttu-id="bbd43-111">{% include 'pattern-category-card' %}</span><span class="sxs-lookup"><span data-stu-id="bbd43-111">{% include 'pattern-category-card' %}</span></span>
    </li>
<span data-ttu-id="bbd43-112">{%- endfor %}</span><span class="sxs-lookup"><span data-stu-id="bbd43-112">{%- endfor %}</span></span>
</ul>

## <a name="catalog-of-patterns"></a><span data-ttu-id="bbd43-113">Mintakatalógus</span><span class="sxs-lookup"><span data-stu-id="bbd43-113">Catalog of patterns</span></span>

| <span data-ttu-id="bbd43-114">Mintázat</span><span class="sxs-lookup"><span data-stu-id="bbd43-114">Pattern</span></span> | <span data-ttu-id="bbd43-115">Összegzés</span><span class="sxs-lookup"><span data-stu-id="bbd43-115">Summary</span></span> |
|---------|---------|
|         |         |

<span data-ttu-id="bbd43-116">{%- for pattern in patterns %} | [{{ pattern.title }}](./{{ pattern.file }}) | {{ pattern.description }} | {%- endfor %}</span><span class="sxs-lookup"><span data-stu-id="bbd43-116">{%- for pattern in patterns %} | [{{ pattern.title }}](./{{ pattern.file }}) | {{ pattern.description }} | {%- endfor %}</span></span>
