---
title: Tervezzen műveletekhez
titleSuffix: Azure Application Architecture Guide
description: Alkalmazások tervezése, így az üzemeltetési csapat rendelkezésére álljanak a szükséges eszközök.
author: MikeWasson
ms.date: 08/30/2018
ms.custom: seojan19
ms.openlocfilehash: c14f3d14330633e7a8c88eedd38f1844291195b1
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/08/2019
ms.locfileid: "54110407"
---
# <a name="design-for-operations"></a>Tervezzen műveletekhez

## <a name="design-an-application-so-that-the-operations-team-has-the-tools-they-need"></a>Tervezze úgy az alkalmazásait, hogy az üzemeltetési csapat rendelkezésére álljanak a szükséges eszközök

A felhő drámai mértékben megváltoztatta az üzemeltetési csapat szerepét. Többé már nem felelősek a hardverek és az alkalmazásokat futtató infrastruktúra felügyeletéért.  Ezzel együtt az üzemeltetés továbbra is kritikus részét képezi egy sikeres felhőalkalmazás futtatásának. Az üzemeltetési csapat fontos feladatai közé az alábbiak tartoznak:

- Környezet
- Figyelés
- Eszkalálás
- Incidensmegoldás
- Biztonsági naplózás

A hatékony naplózás és nyomkövetés különösen fontos a felhőalkalmazásokban. Az üzemeltetési csapat tervezésbe és előkészítésbe történő bevonásával gondoskodhat róla, hogy az alkalmazás biztosítsa számukra a sikerhez szükséges adatokat és elemzések alapjául szolgáló betekintéseket.  <!-- to do: Link to DevOps checklist -->

## <a name="recommendations"></a>Javaslatok

**Tervezzen mindent megfigyelhetőnek**. Miután az üzembe helyezésére sor került és a megoldás fut, a naplók és nyomkövetések biztosítják az elsődleges betekintést a rendszerbe. A *nyomkövetés* egy útvonalat rögzít a rendszeren keresztül, és képes kimutatni a szűk keresztmetszeteket, teljesítményproblémákat, illetve az esetleges hibák előfordulásának a helyét. A *naplózás* adott eseményeket rögzít, például az alkalmazások állapotváltozásait, hibáit és kivételeit. A naplózást éles környezetben végezze, különben pont akkor nem fognak rendelkezésére állni a kellő információk, amikor a leginkább szüksége lenne rájuk.

**Tegye lehetővé a monitorozást**. A monitorozás betekintést enged abba, hogy mennyire jól (vagy gyengén) teljesít egy alkalmazás a rendelkezésre állás, teljesítmény és rendszerállapot terén. A monitorozás például kiderítheti, hogy megfelel-e a szolgáltató szerződés előírásainak. A monitorozás a rendszer normál működését figyeli meg. Olyannyira valós idejűnek kell lennie, amennyire csak lehetséges, hogy az üzemeltetési csapat gyorsan reagálhasson a problémákra. A monitorozás ideális esetben elháríthatja a problémákat, mielőtt azok valamilyen kritikus hibát eredményeznének. További információ: [Monitorozás és diagnosztika][monitoring].

**Tegye lehetővé a kiváltó okok elemzését**. A kiváltó okok elemzése a hibák alapvető okának a megkeresésére szolgáló folyamat. A hiba bekövetkezése után kerül rá sor.

**Használjon elosztott nyomkövetést**. Használjon olyan felhőszintű elosztott nyomkövetési rendszert, amelyet egyidejűség és aszinkronitás jellemez. A nyomkövetéseknek magukban kell foglalniuk egy, a szolgáltatáshatárokon átnyúló korrelációs azonosítót. Egyetlen művelet több alkalmazásszolgáltatás meghívásával is járhat. Ha egy művelet sikertelen, a korrelációs azonosító segítségével határozható meg a hiba oka.

**Szabványosítsa a naplókat és mérőszámokat**. Az üzemeltetési csapatnak a megoldás különféle szolgáltatásaiból származó naplókat kell összesítenie. Ha minden szolgáltatás saját naplózási formátumot használ, nagyon nehézzé vagy egyenesen lehetetlenné válik a hasznos információk kinyerése. Határozzon meg egy általános sémát, amely olyan mezőket tartalmaz, mint például a korrelációs azonosító, az eseménynév, a küldő IP-címe és így tovább. Az egyes szolgáltatások az alapvető sémán alapuló egyéni sémákat hozhatnak létre, amelyek további mezőket tartalmaznak.

**Automatizálja a felügyeleti feladatokat**, beleértve a kiépítést, az üzembe helyezést és a monitorozást. Az automatizálás megismételhetővé teszi a feladatokat, amelyek nem lesznek annyira kitéve az emberi mulasztásoknak.

**A konfigurációt kezelje kódként**. A konfigurációs fájlokat vezesse fel egy verziókövetési rendszerbe, így nyomon követheti és verzióval láthatja el a módosításokat, és szükség esetén visszaállítást is végezhet.

<!-- links -->

[monitoring]: ../../best-practices/monitoring.md
