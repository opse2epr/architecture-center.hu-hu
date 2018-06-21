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
# <a name="cloud-design-patterns"></a>Tervezési minták felhőkhöz

[!INCLUDE [header](../../_includes/header.md)]

Ezek a tervezési minták hasznosak a megbízható, skálázható és biztonságos felhőbeli alkalmazások létrehozásához.

Mindegyik minta ismerteti az általa kezelt problémát, a minta alkalmazásának szempontjait és egy, a Microsoft Azure-on alapuló példát. A legtöbb minta tartalmaz kódmintákat vagy kódrészleteket is, amelyek a minta Azure-on való alkalmazását mutatják be. A legtöbb minta ugyanakkor minden elosztott rendszeren alkalmazható, függetlenül attól, hogy az az Azure-on vagy más felhőplatformon üzemel-e.

## <a name="problem-areas-in-the-cloud"></a>Problémás területek a felhőben

<ul id="categories" class="panel">
{%- for category in categories %}
    <li>
    {% include 'pattern-category-card' %}
    </li>
{%- endfor %}
</ul>

## <a name="catalog-of-patterns"></a>Mintakatalógus

| Mintázat | Összegzés |
|---------|---------|
|         |         |

{%- for pattern in patterns %} | [{{ pattern.title }}](./{{ pattern.file }}) | {{ pattern.description }} | {%- endfor %}
