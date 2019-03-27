---
title: 'CAF: Mi az adatbesorolás?'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Mi az adatbesorolás?
author: BrianBlanchard
ms.openlocfilehash: bb40745b02e4ad6d7faa7054bf62443964f6784b
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299146"
---
<!-- markdownlint-disable MD026 -->

# <a name="what-is-data-classification"></a><span data-ttu-id="7902a-103">Mi az adatbesorolás?</span><span class="sxs-lookup"><span data-stu-id="7902a-103">What is data classification?</span></span>

<span data-ttu-id="7902a-104">Ez a bevezető cikk általános témakörről az adatok besorolását.</span><span class="sxs-lookup"><span data-stu-id="7902a-104">This is an introductory article on the general topic of Data Classification.</span></span> <span data-ttu-id="7902a-105">Az adatbesorolás valamennyi cégirányítási a nagyon gyakori kiindulópontként szolgál.</span><span class="sxs-lookup"><span data-stu-id="7902a-105">Data classification is a very common starting point for all governance.</span></span>

## <a name="business-risks-and-governance"></a><span data-ttu-id="7902a-106">Üzleti kockázatok és irányítás</span><span class="sxs-lookup"><span data-stu-id="7902a-106">Business risks and governance</span></span>

<span data-ttu-id="7902a-107">A szervezetek többségében három üzleti kockázatai cégirányítási beruházó fő oka lehet szűkíteni:</span><span class="sxs-lookup"><span data-stu-id="7902a-107">In most organizations, the primary reasons for investing in governance can be reduced to three business risks:</span></span>

* <span data-ttu-id="7902a-108">A felelősség korlátozására vonatkozó adatok társított</span><span class="sxs-lookup"><span data-stu-id="7902a-108">Liability associated with data breaches</span></span>
* <span data-ttu-id="7902a-109">Az üzleti valamilyen okból kimaradás lép a megszakítás</span><span class="sxs-lookup"><span data-stu-id="7902a-109">Interruption to the business from outages</span></span>
* <span data-ttu-id="7902a-110">Nem tervezett vagy váratlan kiadások</span><span class="sxs-lookup"><span data-stu-id="7902a-110">Unplanned or unexpected spending</span></span>

<span data-ttu-id="7902a-111">Nincsenek három üzleti kockázatok számos változata.</span><span class="sxs-lookup"><span data-stu-id="7902a-111">There are many variants of these three business risks.</span></span> <span data-ttu-id="7902a-112">Azonban ezek általában a leggyakrabban használt.</span><span class="sxs-lookup"><span data-stu-id="7902a-112">However, these tend to be the most common.</span></span>

## <a name="understand-then-mitigate"></a><span data-ttu-id="7902a-113">Megismerheti, majd csökkentése</span><span class="sxs-lookup"><span data-stu-id="7902a-113">Understand then mitigate</span></span>

<span data-ttu-id="7902a-114">Minden kockázati enyhíthető, mielőtt a kell értelmezni.</span><span class="sxs-lookup"><span data-stu-id="7902a-114">Before any risk can be mitigated, it must be understood.</span></span> <span data-ttu-id="7902a-115">Adatok illetéktelen behatolás felelősség, ismertetése az adatbesorolás kezdődik.</span><span class="sxs-lookup"><span data-stu-id="7902a-115">In the case of data breach liability, that understanding starts with data classification.</span></span> <span data-ttu-id="7902a-116">Adatbesorolás az a folyamat minden eszközhöz egy digitális hagyatéki, amely azonosítja az adott eszközhöz társított adatok típusát meta data jellemzők társítása.</span><span class="sxs-lookup"><span data-stu-id="7902a-116">Data classification is the process of associating a meta data characteristic to every asset in a digital estate, which identifies the type of data associated with that asset.</span></span>

<span data-ttu-id="7902a-117">A Microsoft azt javasolja, hogy minden olyan eszköz, amelynek a talált áttelepítés vagy a felhőben való üzembe helyezés lehetséges javaslatokba kell dokumentált jegyezze fel az adatok besorolását, üzleti kritikusság és számlázási feladata a tárfiókból.</span><span class="sxs-lookup"><span data-stu-id="7902a-117">Microsoft suggests that any asset which has been identified as a potential candidate for migration or deployment to the cloud should have documented meta data to record the data classification, business criticality, and billing responsibility.</span></span> <span data-ttu-id="7902a-118">Ezek három pont besorolás meg egy hosszú módja annak ismertetése, és kockázatot.</span><span class="sxs-lookup"><span data-stu-id="7902a-118">These three points of classification can go a long way to understanding and mitigating risks.</span></span>

## <a name="microsofts-data-classification"></a><span data-ttu-id="7902a-119">A Microsoft adatok besorolása</span><span class="sxs-lookup"><span data-stu-id="7902a-119">Microsoft's data classification</span></span>

<span data-ttu-id="7902a-120">Az alábbiakban látható a besorolások a Microsoft használja egy listája.</span><span class="sxs-lookup"><span data-stu-id="7902a-120">The following is a list of classifications Microsoft uses.</span></span> <span data-ttu-id="7902a-121">Iparági vagy meglévő biztonsági követelmények adatok besorolások szabványok már létezik a szervezeten belül.</span><span class="sxs-lookup"><span data-stu-id="7902a-121">Depending on your industry or existing security requirements, data classifications standards may already exist within your organization.</span></span> <span data-ttu-id="7902a-122">Ha nem szabványos, Üdvözöljük digitális hagyatéki és kockázati profilját jobb megértése érdekében ez a minta-minősítés használatával.</span><span class="sxs-lookup"><span data-stu-id="7902a-122">If no standard exists, we welcome you to use this sample classification, to help you better understand your digital estate and risk profile.</span></span>  

* <span data-ttu-id="7902a-123">**Nem üzleti:** A személyes életre, amely nem tartozik Microsoft adatait</span><span class="sxs-lookup"><span data-stu-id="7902a-123">**Non-Business:** Data from your personal life that does not belong to Microsoft</span></span>
* <span data-ttu-id="7902a-124">**Nyilvános:** Szabadon elérhető, nyilvános felhasználásra jóváhagyott üzleti adatok</span><span class="sxs-lookup"><span data-stu-id="7902a-124">**Public:** Business data that is freely available and approved for public consumption</span></span>
* <span data-ttu-id="7902a-125">**Általános:** Üzleti adatok, amelyek nem a nyilvános közönség számára ideális</span><span class="sxs-lookup"><span data-stu-id="7902a-125">**General:** Business data that is not meant for a public audience</span></span>
* <span data-ttu-id="7902a-126">**Bizalmas:** Üzleti adatok, amelyek ártalmas lehet a Microsoft Ha túlterhelt megosztott.</span><span class="sxs-lookup"><span data-stu-id="7902a-126">**Confidential:** Business data that could cause harm to Microsoft if over-shared</span></span>
* <span data-ttu-id="7902a-127">**Szigorúan bizalmas:** Üzleti adatok, amelyek okozhatnak károkat széles körű Microsoft túlterhelt megosztott.</span><span class="sxs-lookup"><span data-stu-id="7902a-127">**Highly Confidential:** Business data that would cause extensive harm to Microsoft if over-shared</span></span>

## <a name="tagging-data-classification-in-azure"></a><span data-ttu-id="7902a-128">Az Azure-ban az adatbesorolás címkézése</span><span class="sxs-lookup"><span data-stu-id="7902a-128">Tagging data classification in Azure</span></span>

<span data-ttu-id="7902a-129">Minden felhőszolgáltató kell biztosít a metaadatokat, bármely eszköz.</span><span class="sxs-lookup"><span data-stu-id="7902a-129">Every cloud provider should offer a mechanism for recording metadata about any asset.</span></span> <span data-ttu-id="7902a-130">Metaadatok létfontosságú a felhőben lévő eszközök kezeléséhez.</span><span class="sxs-lookup"><span data-stu-id="7902a-130">Metadata is vital to managing assets in the cloud.</span></span> <span data-ttu-id="7902a-131">Azure-ban mert az erőforráscímkék legyenek metaadat-tároló javasolt módszere.</span><span class="sxs-lookup"><span data-stu-id="7902a-131">In the case of Azure, resource tags are the suggested approach for metadata storage.</span></span> <span data-ttu-id="7902a-132">További információk az erőforráson címkézése az Azure-ban, tekintse meg a cikk [az Azure-erőforrások rendszerezése címkék használatával](/azure/azure-resource-manager/resource-group-using-tags).</span><span class="sxs-lookup"><span data-stu-id="7902a-132">For additional information on resource tagging in Azure, see the article on [Using tags to organize your Azure resources](/azure/azure-resource-manager/resource-group-using-tags).</span></span>

## <a name="next-steps"></a><span data-ttu-id="7902a-133">További lépések</span><span class="sxs-lookup"><span data-stu-id="7902a-133">Next steps</span></span>

<span data-ttu-id="7902a-134">Az adatbesorolásokat alkalmazni a gyakorlatban hasznosítható cégirányítási utak egyike során.</span><span class="sxs-lookup"><span data-stu-id="7902a-134">Apply data classifications during one of the actionable governance journeys.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="7902a-135">Kezdje egy gyakorlatban hasznosítható Cégirányítási utazás</span><span class="sxs-lookup"><span data-stu-id="7902a-135">Begin an Actionable Governance Journey</span></span>](../journeys/overview.md)
