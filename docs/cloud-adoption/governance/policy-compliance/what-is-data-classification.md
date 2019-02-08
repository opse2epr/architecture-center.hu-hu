---
title: 'CAF: Mi az adatok besorolását'
description: Mi az adatok besorolását?
author: BrianBlanchard
ms.date: 2/11/2019
ms.openlocfilehash: 07268e7242d92ac2581bf28b378a3c43d166620c
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899199"
---
<!-- markdownlint-disable MD026 -->

# <a name="what-is-data-classification"></a>Mi az adatok besorolását?

Ez a bevezető cikk általános témakörről az adatok besorolását. Az adatbesorolás valamennyi cégirányítási a nagyon gyakori kiindulópontként szolgál.

## <a name="business-risks-and-governance"></a>Üzleti kockázatok és irányítás

A szervezetek többségében három üzleti kockázatai cégirányítási beruházó fő oka lehet szűkíteni:

* A felelősség korlátozására vonatkozó adatok társított
* Az üzleti valamilyen okból kimaradás lép a megszakítás
* Nem tervezett vagy váratlan kiadások

Nincsenek három üzleti kockázatok számos változata. Azonban a tend kell a leggyakrabban használt.

## <a name="understand-then-mitigate"></a>Megismerheti, majd csökkentése

Minden kockázati enyhíthető, mielőtt a kell értelmezni. Adatok illetéktelen behatolás felelősség, ismertetése az adatbesorolás kezdődik. Adatbesorolás az a folyamat minden eszközhöz egy digitális hagyatéki, amely azonosítja az adott eszközhöz társított adatok típusát meta data jellemzők társítása.

A Microsoft azt javasolja, hogy minden olyan eszköz, amelynek a talált áttelepítés vagy a felhőben való üzembe helyezés lehetséges javaslatokba kell dokumentált jegyezze fel az adatok besorolását, üzleti kritikusság és számlázási feladata a tárfiókból. Ezek három pont besorolás meg egy hosszú módja annak ismertetése, és kockázatot.

## <a name="microsofts-data-classification"></a>A Microsoft adatok besorolása

Az alábbiakban látható a besorolások a Microsoft használja egy listája. Iparági vagy meglévő biztonsági követelmények adatok besorolások szabványok már létezik a szervezeten belül. Ha nem szabványos, Üdvözöljük digitális hagyatéki és kockázati profilját jobb megértése érdekében ez a minta-minősítés használatával.  

* **Nem üzleti:** A személyes életre, amely nem tartozik Microsoft adatait
* **Nyilvános:** Szabadon elérhető, nyilvános felhasználásra jóváhagyott üzleti adatok
* **Általános:** Üzleti adatok, amelyek nem a nyilvános közönség számára ideális
* **Bizalmas:** Üzleti adatok, amelyek ártalmas lehet a Microsoft Ha túlterhelt megosztott.
* **Szigorúan bizalmas:** Üzleti adatok, amelyek okozhatnak károkat széles körű Microsoft túlterhelt megosztott.

## <a name="tagging-data-classification-in-azure"></a>Az Azure-ban az adatbesorolás címkézése

Minden felhőszolgáltató kell biztosít a metaadatokat, bármely eszköz. Metaadatok létfontosságú a felhőben lévő eszközök kezeléséhez. Azure-ban mert az erőforráscímkék legyenek metaadat-tároló javasolt módszere. További információk az erőforráson címkézése az Azure-ban, tekintse meg a cikk [az Azure-erőforrások rendszerezése címkék használatával](/azure/azure-resource-manager/resource-group-using-tags).

## <a name="next-steps"></a>További lépések

Az adatbesorolásokat alkalmazni a gyakorlatban hasznosítható cégirányítási utak egyike során.

> [!div class="nextstepaction"]
> [A gyakorlatban hasznosítható Cégirányítási Journey megkezdése](../journeys/overview.md)
