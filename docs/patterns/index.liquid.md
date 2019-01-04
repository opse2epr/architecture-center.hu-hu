---
title: Tervezési minták felhőkhöz
titleSuffix: Azure Architecture Center
description: Tervezési minták felhőkhöz a Microsoft Azure-ban
keywords: Azure
author: dragon119
ms.date: 12/10/2018
ms.custom: seodec18
ms.openlocfilehash: 873d4cf02690a2cc3ffe4f35b044dedf70700fb5
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/04/2019
ms.locfileid: "54011023"
---
# <a name="cloud-design-patterns"></a><span data-ttu-id="33b2d-104">Tervezési minták felhőkhöz</span><span class="sxs-lookup"><span data-stu-id="33b2d-104">Cloud Design Patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="33b2d-105">Ezek a tervezési minták hasznosak a megbízható, skálázható és biztonságos felhőbeli alkalmazások létrehozásához.</span><span class="sxs-lookup"><span data-stu-id="33b2d-105">These design patterns are useful for building reliable, scalable, secure applications in the cloud.</span></span>

<span data-ttu-id="33b2d-106">Mindegyik minta ismerteti az általa kezelt problémát, a minta alkalmazásának szempontjait és egy, a Microsoft Azure-on alapuló példát.</span><span class="sxs-lookup"><span data-stu-id="33b2d-106">Each pattern describes the problem that the pattern addresses, considerations for applying the pattern, and an example based on Microsoft Azure.</span></span> <span data-ttu-id="33b2d-107">A legtöbb minta tartalmaz kódmintákat vagy kódrészleteket is, amelyek a minta Azure-on való alkalmazását mutatják be.</span><span class="sxs-lookup"><span data-stu-id="33b2d-107">Most of the patterns include code samples or snippets that show how to implement the pattern on Azure.</span></span> <span data-ttu-id="33b2d-108">A legtöbb minta ugyanakkor minden elosztott rendszeren alkalmazható, függetlenül attól, hogy az az Azure-on vagy más felhőplatformon üzemel-e.</span><span class="sxs-lookup"><span data-stu-id="33b2d-108">However, most of the patterns are relevant to any distributed system, whether hosted on Azure or on other cloud platforms.</span></span>

## <a name="problem-areas-in-the-cloud"></a><span data-ttu-id="33b2d-109">Problémás területek a felhőben</span><span class="sxs-lookup"><span data-stu-id="33b2d-109">Problem areas in the cloud</span></span>

<!-- markdownlint-disable MD033 -->

<ul id="categories" class="panel">
<span data-ttu-id="33b2d-110">{%- for category in categories %}</span><span class="sxs-lookup"><span data-stu-id="33b2d-110">{%- for category in categories %}</span></span>
    <li>
    <span data-ttu-id="33b2d-111">{% include 'pattern-category-card' %}</span><span class="sxs-lookup"><span data-stu-id="33b2d-111">{% include 'pattern-category-card' %}</span></span>
    </li>
<span data-ttu-id="33b2d-112">{%- endfor %}</span><span class="sxs-lookup"><span data-stu-id="33b2d-112">{%- endfor %}</span></span>
</ul>

<!-- markdownlint-enable MD033 -->

## <a name="catalog-of-patterns"></a><span data-ttu-id="33b2d-113">Mintakatalógus</span><span class="sxs-lookup"><span data-stu-id="33b2d-113">Catalog of patterns</span></span>

| <span data-ttu-id="33b2d-114">Mintázat</span><span class="sxs-lookup"><span data-stu-id="33b2d-114">Pattern</span></span> | <span data-ttu-id="33b2d-115">Összegzés</span><span class="sxs-lookup"><span data-stu-id="33b2d-115">Summary</span></span> |
|---------|---------|
|         |         |

<span data-ttu-id="33b2d-116">{%- for pattern in patterns %} | [{{ pattern.title }}](./{{ pattern.file }}) | {{ pattern.description }} | {%- endfor %}</span><span class="sxs-lookup"><span data-stu-id="33b2d-116">{%- for pattern in patterns %} | [{{ pattern.title }}](./{{ pattern.file }}) | {{ pattern.description }} | {%- endfor %}</span></span>
