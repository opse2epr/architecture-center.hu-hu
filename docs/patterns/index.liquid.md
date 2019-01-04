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
# <a name="cloud-design-patterns"></a>Tervezési minták felhőkhöz

[!INCLUDE [header](../../_includes/header.md)]

Ezek a tervezési minták hasznosak a megbízható, skálázható és biztonságos felhőbeli alkalmazások létrehozásához.

Mindegyik minta ismerteti az általa kezelt problémát, a minta alkalmazásának szempontjait és egy, a Microsoft Azure-on alapuló példát. A legtöbb minta tartalmaz kódmintákat vagy kódrészleteket is, amelyek a minta Azure-on való alkalmazását mutatják be. A legtöbb minta ugyanakkor minden elosztott rendszeren alkalmazható, függetlenül attól, hogy az az Azure-on vagy más felhőplatformon üzemel-e.

## <a name="problem-areas-in-the-cloud"></a>Problémás területek a felhőben

<!-- markdownlint-disable MD033 -->

<ul id="categories" class="panel">
{%- for category in categories %}
    <li>
    {% include 'pattern-category-card' %}
    </li>
{%- endfor %}
</ul>

<!-- markdownlint-enable MD033 -->

## <a name="catalog-of-patterns"></a>Mintakatalógus

| Mintázat | Összegzés |
|---------|---------|
|         |         |

{%- for pattern in patterns %} | [{{ pattern.title }}](./{{ pattern.file }}) | {{ pattern.description }} | {%- endfor %}
