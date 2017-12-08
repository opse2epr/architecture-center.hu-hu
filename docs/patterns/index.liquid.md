---
title: "Tervezési minták felhőkhöz"
description: "Felhő kialakítási minta a Microsoft Azure"
keywords: Azure
ms.openlocfilehash: 264b8296a428f9c1b87314b782efcabc89cf010f
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="cloud-design-patterns"></a><span data-ttu-id="0015b-104">Tervezési minták felhőkhöz</span><span class="sxs-lookup"><span data-stu-id="0015b-104">Cloud Design Patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="0015b-105">Ezek a kialakítási minták a felhőben megbízható, méretezhető, biztonságos alkalmazások hasznosak.</span><span class="sxs-lookup"><span data-stu-id="0015b-105">These design patterns are useful for building reliable, scalable, secure applications in the cloud.</span></span>

<span data-ttu-id="0015b-106">Minden egyes minta a problémát, hogy a minta címek, a minta alkalmazásához szempontok és példa alapján a Microsoft Azure ismerteti.</span><span class="sxs-lookup"><span data-stu-id="0015b-106">Each pattern describes the problem that the pattern addresses, considerations for applying the pattern, and an example based on Microsoft Azure.</span></span> <span data-ttu-id="0015b-107">A minta a legtöbb mintakódok vagy kódtöredékek, amelyek bemutatják, hogyan valósítja meg a minta Azure tartalmazza.</span><span class="sxs-lookup"><span data-stu-id="0015b-107">Most of the patterns include code samples or snippets that show how to implement the pattern on Azure.</span></span> <span data-ttu-id="0015b-108">Azonban a minta a legtöbb kapcsolódik a összes elosztott rendszert, hogy az Azure-on vagy más felhőplatformokkal.</span><span class="sxs-lookup"><span data-stu-id="0015b-108">However, most of the patterns are relevant to any distributed system, whether hosted on Azure or on other cloud platforms.</span></span>

## <a name="problem-areas-in-the-cloud"></a><span data-ttu-id="0015b-109">A felhőben a problémás területek</span><span class="sxs-lookup"><span data-stu-id="0015b-109">Problem areas in the cloud</span></span>

<ul id="categories" class="panel">
<span data-ttu-id="0015b-110">{%-a kategóriák kategória %}</span><span class="sxs-lookup"><span data-stu-id="0015b-110">{%- for category in categories %}</span></span>
    <li>
    <span data-ttu-id="0015b-111">{% include "minta-kategória-kártya" %}</span><span class="sxs-lookup"><span data-stu-id="0015b-111">{% include 'pattern-category-card' %}</span></span>
    </li>
<span data-ttu-id="0015b-112">{a(z) %-endfor %}</span><span class="sxs-lookup"><span data-stu-id="0015b-112">{%- endfor %}</span></span>
</ul>

## <a name="catalog-of-patterns"></a><span data-ttu-id="0015b-113">Annak a katalógusban</span><span class="sxs-lookup"><span data-stu-id="0015b-113">Catalog of patterns</span></span>

| <span data-ttu-id="0015b-114">Minta</span><span class="sxs-lookup"><span data-stu-id="0015b-114">Pattern</span></span> | <span data-ttu-id="0015b-115">Összefoglalás</span><span class="sxs-lookup"><span data-stu-id="0015b-115">Summary</span></span> |
| ------- | ------- |
<span data-ttu-id="0015b-116">{a(z) %-a minta %} mintákra |} [{{pattern.title}}](./{{ pattern.file }}) |} {{pattern.description}} |} {a(z) %-endfor %}</span><span class="sxs-lookup"><span data-stu-id="0015b-116">{%- for pattern in patterns %} | [{{ pattern.title }}](./{{ pattern.file }}) | {{ pattern.description }} | {%- endfor %}</span></span>