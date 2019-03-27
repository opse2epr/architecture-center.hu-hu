---
title: Eseményvezérelt architektúra
titleSuffix: Azure Application Architecture Guide
description: Ismerteti a előnyeit, kihívásait és ajánlott eljárások az eseményvezérelt és IoT-architektúráival az Azure-ban.
author: MikeWasson
ms.date: 08/30/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19
ms.openlocfilehash: b83a919e4ccd41d20b508b10604365a0b4ca90af
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299196"
---
# <a name="event-driven-architecture-style"></a>Eseményvezérelt architektúra

Az eseményvezérelt architektúra az eseménystreamet létrehozó **eseménykészítőkből** és az eseményeket figyelő **eseményfeldolgozókból** áll.

![Az eseményvezérelt architektúra stílusának ábrája](./images/event-driven.svg)

Az események kézbesítése közel valós időben történik, így a feldolgozók azonnal reagálhatnak rájuk, amint bekövetkeznek. A készítők el vannak választva a feldolgozóktól – a készítő nem tudja, hogy mely feldolgozók figyelik az eseményeket. A feldolgozók szintén el vannak választva egymástól, és minden feldolgozó minden eseményt lát. Ez eltér a [versengő felhasználók][competing-consumers] mintától, amelyben a feldolgozók egy üzenetsorból kérik le az üzeneteket, és az üzenetek feldolgozására csak egyszer kerül sor (amennyiben nem történik hiba). Bizonyos rendszerekben, amilyen például az IoT, az események betöltése nagyon nagy mennyiségben történik.

Az eseményvezérelt architektúra közzétevői/előfizetői vagy eseménystreamelési modellt használhat.

- **Közzétevői/előfizetői**: Az üzenetkezelési infrastruktúra nyomon követi az előfizetéseket. Esemény közzétételekor az eseményt minden előfizetőnek elküldi. Az esemény fogadását követően az esemény nem játszható vissza, és az új előfizetők nem látják az eseményt.

- **Eseménystreamelés**: Eseményeket egy naplóba írja a program. A rendszer szigorú sorrendben tárolja az eseményeket (egy partíción belül), és meg is őrzi őket. Az ügyfelek nem fizetnek elő a streamre, hanem a stream bármelyik részét beolvashatják. Az ügyfél felelőssége, hogy továbbhaladjon a streamen belül. Ez azt jelenti, hogy az ügyfél bármikor csatlakozhat, és visszajátszhatja az eseményeket.

A feldolgozói oldalon több gyakori változat áll rendelkezésre:

- **Egyszerű eseményfeldolgozás**. Az esemény azonnali műveletet aktivál a feldolgozónál. Az Azure Functions és egy Service Bus-trigger használatával például kiválthatja egy függvény végrehajtását, amikor közzétesznek egy üzenetet egy Service Bus-témakörben.

- **Összetett eseményfeldolgozás**. A feldolgozó az eseményadatokban található mintákat keresve dolgoz fel egy eseménysorozatot egy olyan technológiával, mint például az Azure Stream Analytics vagy az Apache Storm. Összesíthet például beágyazott eszközből származó adatokat egy adott időtartamra vonatkozóan, és értesítést hozhat létre, ha a mozgó átlag átlép egy bizonyos küszöbértéket.

- **Eseménystream-feldolgozás**. Egy adatstreamelési platformot (például az Azure IoT Hubot vagy az Apache Kafkát) folyamatként használva olvashatja be az eseményeket, és továbbíthatja azokat a streamfeldolgozóknak. A streamfeldolgozók feldolgozzák vagy átalakítják a streamet. Több streamfeldolgozó is használható az alkalmazás különböző alrendszereihez. Ez a megközelítés jó megoldás lehet IoT számítási feladatokhoz.

Az események forrása lehet a rendszeren kívüli is, ilyenek például egy IoT-megoldás fizikai eszközei. Ebben az esetben a rendszernek képesnek kell lennie az adatoknak olyan mennyiségben és átviteli sebességen történő beolvasására, amelyet az adatforrás megkövetel.

A fenti logikai ábrán az egyes feldolgozótípusok külön dobozként jelennek meg. A gyakorlatban sokszor előfordul, hogy egy feldolgozó több példánya fut is annak érdekében, nehogy a feldolgozó hibaérzékeny ponttá váljon a rendszerben. Több példányra a nagy mennyiségű vagy gyakori események kezeléséhez is szükség lehet. Emellett egy adott feldolgozó több szálon is feldolgozhat eseményeket. Ez kihívásokat támaszthat, ha az eseményeket sorrendben kell feldolgozni, vagy „pontosan egyszeri” szemantikára van szükség. Lásd: [A koordináció minimalizálása][minimize-coordination].

## <a name="when-to-use-this-architecture"></a>Mikor érdemes ezt az architektúrát használni?

- Több alrendszernek ugyanazokat az eseményeket kell feldolgoznia.
- Valós idejű feldolgozásra van szükség, minimális időbeli késés mellett.
- Összetett eseményfeldolgozásra van szükség, például mintaegyeztetésre vagy időtartamokra vonatkozó összesítésre.
- Nagy mennyiségű vagy gyakoriságú adatról van szó, például IoT esetén.

## <a name="benefits"></a>Előnyök

- A létrehozók és a feldolgozók el vannak választva egymástól.
- Nincs szükség pontok közötti integrációra. Az új feldolgozók egyszerűen adhatók hozzá a rendszerhez.
- A feldolgozók az érkezésükkor azonnal reagálhatnak az eseményekre.
- Hatékonyan méretezhető és elosztható.
- Az alrendszerek az eseménystream önálló nézetével rendelkeznek.

## <a name="challenges"></a>Problémák

- Garantált kézbesítés. Egyes rendszerekben, különösen IoT-forgatókönyvek esetén, alapvető fontosságú az események kézbesítésének garantálása.
- Események feldolgozása sorrendben vagy pontosan egyszer. Általában minden feldolgozótípus több példányban fut a rugalmasság és a méretezhetőség érdekében. Ez kihívást támaszthat, ha az eseményeket sorrendben kell feldolgozni (egy feldolgozótípuson belül), illetve ha a feldolgozási logika nem idempotens.

 <!-- links -->

[competing-consumers]: ../../patterns/competing-consumers.md
[minimize-coordination]: ../design-principles/minimize-coordination.md
