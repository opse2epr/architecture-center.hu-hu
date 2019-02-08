---
title: 'CAF: Architektúrával kapcsolatos döntés útmutatók'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: További információ a felhő bevezetésének keretrendszer architektúrával kapcsolatos döntés útmutatók.
author: rotycenh
ms.openlocfilehash: b43047b5fc5b636bc84b9f28ec24846730e63672
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899362"
---
# <a name="architectural-decision-guides"></a><span data-ttu-id="bd0f7-103">Architektúrával kapcsolatos döntés útmutatók</span><span class="sxs-lookup"><span data-stu-id="bd0f7-103">Architectural decision guides</span></span>

<span data-ttu-id="bd0f7-104">Az architektúrával kapcsolatos döntés útmutatók a felhő bevezetésének keretrendszer írja le, minták és a modellek, amelyek segítenek a felhőalapú cégirányítási tervezési útmutatást létrehozásakor.</span><span class="sxs-lookup"><span data-stu-id="bd0f7-104">The architectural decision guides in the Cloud Adoption Framework describe patterns and models that help when creating cloud governance design guidance.</span></span> <span data-ttu-id="bd0f7-105">Minden döntés útmutató egy alapvető infrastruktúra összetevője magánfelhők koncentrál, és megjeleníti a lehetséges minták vagy modellekkel támogatják az adott felhő központi telepítési forgatókönyv.</span><span class="sxs-lookup"><span data-stu-id="bd0f7-105">Each decision guide focuses on one core infrastructure component of cloud deployments and lists potential patterns or models intended to support specific cloud deployment scenarios.</span></span>

<span data-ttu-id="bd0f7-106">Amikor hozzákezd a szervezete felhőben cégirányítási létesíteni, gyakorlatban hasznosítható cégirányítási Journey egy alapkonfiguráció ütemterv adja meg.</span><span class="sxs-lookup"><span data-stu-id="bd0f7-106">When you begin to establish cloud governance for your organization,  actionable governance journeys provide a baseline roadmap.</span></span> <span data-ttu-id="bd0f7-107">Ezek az utak legyen azonban feltételezéseket követelményeknek és a prioritások, amelyek nem tükrözik azokat a szervezet.</span><span class="sxs-lookup"><span data-stu-id="bd0f7-107">However, these journeys make assumptions about requirements and priorities that may not reflect those of your organization.</span></span>
<span data-ttu-id="bd0f7-108">Döntés az útmutatók a minta cégirányítási Journey kiegészítik azáltal, hogy helyettesítő mintákat, és modellek, amelyek segítenek a példa-tervezési segédlet a saját igényeinek megfelelően az architekturális tervezés kiválasztott lehetőségeket igazítása.</span><span class="sxs-lookup"><span data-stu-id="bd0f7-108">These decision guides supplement the sample governance journeys by providing alternative patterns and models that help you align the architectural design choices made in the example design guidance with your own requirements.</span></span>

## <a name="design-guidance-categories"></a><span data-ttu-id="bd0f7-109">Tervezési útmutatást kategóriák</span><span class="sxs-lookup"><span data-stu-id="bd0f7-109">Design guidance categories</span></span>

<span data-ttu-id="bd0f7-110">A következő kategóriák jelöli, hogy a felhő központi telepítések egy alapvető technológia.</span><span class="sxs-lookup"><span data-stu-id="bd0f7-110">Each of the following categories represents a foundational technology of all cloud deployments.</span></span> <span data-ttu-id="bd0f7-111">Az egyes választott a példa tervezési Journey, amely nem felel meg a követelményeknek vizsgálja meg a beállításokat alább vonatkozó részében egy minta vagy a modell jobban az igényeinek megfelelő kiválasztása.</span><span class="sxs-lookup"><span data-stu-id="bd0f7-111">For each choice in the example design journeys that doesn’t match your requirements, examine the options in the relevant section below to choose a pattern or model better suited to your needs.</span></span>

<span data-ttu-id="bd0f7-112">[Előfizetések](./subscriptions/overview.md): Tervezze meg a felhőbeli üzemelő példány előfizetés tervezési és a fiók tulajdonjogát, a számlázásra és a felügyeleti funkciókat a szervezet megfelelő struktúra.</span><span class="sxs-lookup"><span data-stu-id="bd0f7-112">[Subscriptions](./subscriptions/overview.md): Plan your cloud deployment's subscription design and account structure to match your organization's ownership, billing, and management capabilities.</span></span>

<span data-ttu-id="bd0f7-113">[identitás](./identity/overview.md): Felhőalapú identitás-szolgáltatások integrálható a meglévő identitás erőforrások hozzáférés-vezérlés az informatikai környezetben támogatásához.</span><span class="sxs-lookup"><span data-stu-id="bd0f7-113">[Identity](./identity/overview.md): Integrate cloud-based identity services with your existing identity resources to support access control within your IT environment.</span></span>

<span data-ttu-id="bd0f7-114">[A házirend betartatása](./policy-enforcement/overview.md): Határozzon meg, és az erőforrások és a felhőben üzembe helyezett számítási feladatok a szervezeti házirend szabályok érvényesítése.</span><span class="sxs-lookup"><span data-stu-id="bd0f7-114">[Policy Enforcement](./policy-enforcement/overview.md): Define and enforce organizational policy rules for resources and workloads that you deploy to the cloud.</span></span>

<span data-ttu-id="bd0f7-115">[Erőforrás konzisztencia](./resource-consistency/overview.md): Győződjön meg arról, hogy üzembe helyezés és a szervezet a felhőalapú erőforrások igazítása kényszerítése erőforrás felügyeleti és a házirend követelményeinek.</span><span class="sxs-lookup"><span data-stu-id="bd0f7-115">[Resource Consistency](./resource-consistency/overview.md): Ensure that deployment and organization of your cloud-based resources align to enforce resource management and policy requirements.</span></span>

<span data-ttu-id="bd0f7-116">[Erőforrás-címkézési](./resource-tagging/overview.md): A számlázási modellekkel, felhő számlázási megközelítések, felügyeleti, hozzáférés-vezérlés és működési hatékonyságát a felhőalapú erőforrások rendezéséhez.</span><span class="sxs-lookup"><span data-stu-id="bd0f7-116">[Resource Tagging](./resource-tagging/overview.md): Organize your cloud-based resources to support billing models, cloud accounting approaches, management, access control, and operational efficiency.</span></span> <span data-ttu-id="bd0f7-117">Erőforrás-címkézési szükséges egy egységes és jól szervezett elnevezési és metaadatok sémát.</span><span class="sxs-lookup"><span data-stu-id="bd0f7-117">Resource tagging requires a consistent and well-organized naming and metadata scheme.</span></span>

<span data-ttu-id="bd0f7-118">[Szoftveralapú hálózatok](./software-defined-network/overview.md): Gyors üzembe helyezési és módosítása a virtualizált hálózati funkciói segítségével a felhőbe biztonságos számítási feladatok üzembe helyezéséhez.</span><span class="sxs-lookup"><span data-stu-id="bd0f7-118">[Software Defined Networks](./software-defined-network/overview.md): Deploy secure workloads to the cloud using rapid deployment and modification of virtualized networking capabilities.</span></span> <span data-ttu-id="bd0f7-119">Szoftveralapú hálózat (SDNs) támogatja a rugalmas munkafolyamatok, erőforrások elkülönítése, és felhőalapú rendszerek integrálását az meglévő IT-infrastruktúrája.</span><span class="sxs-lookup"><span data-stu-id="bd0f7-119">Software-defined networks (SDNs) can support agile workflows, isolate resources, and integrate cloud-based systems with your existing IT infrastructure.</span></span>

<span data-ttu-id="bd0f7-120">[Titkosítási](./encryption/overview.md): A titkosítással, fontos szempont a biztonsági belül üzembe helyezése a felhőben tárolt bizalmas adatok védelme.</span><span class="sxs-lookup"><span data-stu-id="bd0f7-120">[Encryption](./encryption/overview.md): Secure your sensitive data using encryption, an important aspect of security within a cloud deployment.</span></span>

<span data-ttu-id="bd0f7-121">[Naplók és a jelentéskészítés](./log-and-report/overview.md): Felhőalapú erőforrások által létrehozott figyelő naplóadatokat.</span><span class="sxs-lookup"><span data-stu-id="bd0f7-121">[Logs and Reporting](./log-and-report/overview.md): Monitor log data generated by cloud-based resources.</span></span> <span data-ttu-id="bd0f7-122">Adatok elemzése betekintést állapotát az operatív, a karbantartással és a számítási feladatok házirend kényszerítési állapotát.</span><span class="sxs-lookup"><span data-stu-id="bd0f7-122">Analyzing data provides health-related insights into the operations, maintenance, and policy enforcement status of workloads.</span></span>

## <a name="next-steps"></a><span data-ttu-id="bd0f7-123">További lépések</span><span class="sxs-lookup"><span data-stu-id="bd0f7-123">Next steps</span></span>

<span data-ttu-id="bd0f7-124">Ismerje meg, hogyan előfizetések és fiókok erre a célra a felhőbeli üzembe helyezés alap.</span><span class="sxs-lookup"><span data-stu-id="bd0f7-124">Learn how subscriptions and accounts serve as the base of a cloud deployment.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="bd0f7-125">Előfizetések kialakítása</span><span class="sxs-lookup"><span data-stu-id="bd0f7-125">Subscriptions design</span></span>](subscriptions/overview.md)
