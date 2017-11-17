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
# <a name="cloud-design-patterns"></a>Tervezési minták felhőkhöz

[!INCLUDE [header](../../_includes/header.md)]

Ezek a kialakítási minták a felhőben megbízható, méretezhető, biztonságos alkalmazások hasznosak.

Minden egyes minta a problémát, hogy a minta címek, a minta alkalmazásához szempontok és példa alapján a Microsoft Azure ismerteti. A minta a legtöbb mintakódok vagy kódtöredékek, amelyek bemutatják, hogyan valósítja meg a minta Azure tartalmazza. Azonban a minta a legtöbb kapcsolódik a összes elosztott rendszert, hogy az Azure-on vagy más felhőplatformokkal.

## <a name="problem-areas-in-the-cloud"></a>A felhőben a problémás területek

<ul id="categories" class="panel">
{%-a kategóriák kategória %}
    <li>
    {% include "minta-kategória-kártya" %}
    </li>
{a(z) %-endfor %}
</ul>

## <a name="catalog-of-patterns"></a>Annak a katalógusban

| Minta | Összefoglalás |
| ------- | ------- |
{a(z) %-a minta %} mintákra |} [{{pattern.title}}](./{{ pattern.file }}) |} {{pattern.description}} |} {a(z) %-endfor %}