---
title: 'CAF: Műveletek áttekintése'
description: A Microsoft Cloud bevezetési keretrendszert az Azure operations tartalmának áttekintése
author: petertaylor9999
ms.date: 09/20/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.openlocfilehash: 2667eb54dcea7621f32f6f328c1817be55119a1f
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55897133"
---
# <a name="caf-operations-overview"></a><span data-ttu-id="b3c2e-103">CAF: Műveletek áttekintése</span><span class="sxs-lookup"><span data-stu-id="b3c2e-103">CAF: Operations overview</span></span>

<span data-ttu-id="b3c2e-104">Ez a szakasza a CAF a témakör a **műveletek**.</span><span class="sxs-lookup"><span data-stu-id="b3c2e-104">This section of the CAF the topic of **operations**.</span></span>

<span data-ttu-id="b3c2e-105">Után a vállalat folytat egy **digitális átalakulás**, a tervezési és megvalósítási csapatok végzett munka többsége a meglévő számítási feladatok migrálása a helyszínről az Azure-ba, a fejlesztés és tesztelés új fog szerveződik felhőbeli natív alkalmazásokat az Azure és az új innovatív Azure-szolgáltatások beépítése meglévő helyszíni számítási feladatokat.</span><span class="sxs-lookup"><span data-stu-id="b3c2e-105">Once your enterprise is engaged in a **digital transformation**, a majority of the work done on the design and implementation teams will revolve around migrating existing workloads from on-premises to Azure, developing and testing new cloud-native applications in Azure, and incorporating new innovative Azure services into existing on-premises workloads.</span></span> <span data-ttu-id="b3c2e-106">Azonban ezeket is csak az első lépés a digitális átalakulást.</span><span class="sxs-lookup"><span data-stu-id="b3c2e-106">However, these are just the first step in the digital transformation.</span></span> <span data-ttu-id="b3c2e-107">Miután ezek a számítási feladatok és az Azure-ban, a következő lépés az, hogy **működéséhez** azokat a felhőben.</span><span class="sxs-lookup"><span data-stu-id="b3c2e-107">Once these workloads are up and running in Azure, the next step is to **operate** them in the cloud.</span></span>

<span data-ttu-id="b3c2e-108">Az IT-folyamatok és nem funkcionális követelmények szükségesek ahhoz, hogy az Azure-ban, egy üzleti számítási feladatok futtatása a felhőben működő hivatkozik.</span><span class="sxs-lookup"><span data-stu-id="b3c2e-108">Operating in the cloud refers to the IT processes and non-functional requirements necessary to run workloads in Azure as a business.</span></span> <span data-ttu-id="b3c2e-109">Ez magában foglalja a számítási feladatok elemzése a függőségek szűk, vészhelyreállítás és egyéb stratégiák fejlesztése figyelését.</span><span class="sxs-lookup"><span data-stu-id="b3c2e-109">This includes monitoring of workloads, analyzing dependencies for bottlenecks, developing strategies for disaster recovery, and more.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="b3c2e-110">Működési mentességre felülvizsgálat létrehozása</span><span class="sxs-lookup"><span data-stu-id="b3c2e-110">Establish an operational fitness review</span></span>](operational-fitness-review.md)
