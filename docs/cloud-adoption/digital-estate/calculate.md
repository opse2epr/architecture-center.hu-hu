---
title: 'CAF: A digitális hagyatéki a költségmodellek igazítása'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
description: Igazítás a digitális hagyatéki felhőköltségek az előre jelzett költség modelleket.
author: BrianBlanchard
ms.date: 12/10/2018
ms.topic: guide
ms.openlocfilehash: b37d833106ad487faadab7c4b7ae397fa237d3d3
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55897789"
---
# <a name="align-cost-models-with-the-digital-estate-to-forecast-cloud-costs"></a>A digitális hagyatéki felhőköltségek az előre jelzett költség modelleket igazítása

Miután egy digitális hagyatéki rendelkezik lett rendszerezhetők, azt is illeszkedjenek a egyenértékű költségszámítási modellek a választott felhőalapú szolgáltatón. Megvizsgálja a költségmodellek is nehézkes anélkül, hogy egy adott felhőszolgáltató összpontosítani. Ahhoz, hogy a cikkben szereplő példákat képzés résztvevői hasznos képességekkel, az Azure a feltételezett felhőszolgáltató is.

Az Azure eszközök, amelyekkel kezelheti a felhő költségeinek átlátható és pontosságának, hogy a legtöbbet az Azure és más felhők díjszabása. Az eszközök figyeléséhez, lefoglalni és optimalizálhatja a felhőbeli költségeket, így lehetővé teszi az ügyfelek számára, hogy gyorsítsa fel a jövőbeli befektetések magabiztosan.

- [Az Azure Migrate](/azure/migrate/migrate-overview). Az Azure migrate van talán a leginkább költséghatékony módja költség modell igazítását. Ez az eszköz lehetővé teszi, hogy digitális hagyatéki [készlet](inventory.md), [ésszerűsítés korlátozott](rationalize.md), és a számítások egy eszközt a költségek.

- [Teljes bekerülési költséget számító költsége](https://azure.com/tco). Csökkentheti a teljes tulajdonosi költség, a helyszíni infrastruktúra és az Azure-felhő platform. Az Azure költségkalkulátor segítségével megbecsülheti a költségmegtakarítást is kiderülhet által az alkalmazás számítási feladatok migrálása az Azure-bA. Egyszerűen adjon meg egy rövid leírást a helyszíni környezet egy azonnali jelentés.

- [Díjkalkulátor](https://azure.microsoft.com/en-in/pricing/). Becsülje meg a várható havi számla díjkalkulátorunk használatával. Nyomon követheti tényleges fiókhasználatát és számláját a számlázási portálon bármikor. Állítsa be elszámolási értesítések automatikus e-mail értesítést kapjon, ha költségei túllépik a összeget.

- [Az Azure Cost Management](https://azure.microsoft.com/services/cost-management/). A Microsoft kisegítő a Cloudyn által licencelt Azure Cost Management egy többfelhős költségkezelő felügyeleti megoldás, amely segít a legjobb és a felügyelhető az Azure és egyéb felhőerőforrások. Felhőalapú használati és számlázási adatok alkalmazás program felületek (API) keresztül gyűjt az Azure, Amazon Web Services és a Google Cloud Platform. Az így nyert adatok alapján egy egységes nézetben juthat részletes információhoz az összes platform erőforrás-felhasználásáról és költségeiről. Folyamatosan figyelheti a felhőhasználatot és a költségek időbeli alakulását. A túlköltekezés elkerülése érdekében összehasonlíthatja tényleges felhőbeli költéseit az előirányzott költségvetéssel. Feltárhatja a szokásostól eltérő kiadásokat és a nem hatékony felhasználási módokat is. Előzményadatok használatával a felhőhasználatot és az előrejelzési pontosság növeléséhez.
