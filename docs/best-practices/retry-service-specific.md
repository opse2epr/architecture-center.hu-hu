---
title: Szolgáltatásspecifikus útmutató az újrapróbálkozási mechanizmushoz
description: Szolgáltatásspecifikus útmutató az újrapróbálkozási mechanizmus beállításához.
author: dragon119
ms.date: 07/13/2016
pnp.series.title: Best Practices
ms.openlocfilehash: 72dfb59c3357c5f14806a33ef5f6cdd3e7937915
ms.sourcegitcommit: 8b5fc0d0d735793b87677610b747f54301dcb014
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 07/29/2018
ms.locfileid: "39334164"
---
# <a name="retry-guidance-for-specific-services"></a><span data-ttu-id="4af3c-103">Újrapróbálkozási útmutatás adott szolgáltatásoknál</span><span class="sxs-lookup"><span data-stu-id="4af3c-103">Retry guidance for specific services</span></span>

<span data-ttu-id="4af3c-104">A legtöbb Azure-szolgáltatás és ügyfél-SDK tartalmaz egy újrapróbálkozási mechanizmust.</span><span class="sxs-lookup"><span data-stu-id="4af3c-104">Most Azure services and client SDKs include a retry mechanism.</span></span> <span data-ttu-id="4af3c-105">Ezek azonban eltérőek lehetnek, mert minden szolgáltatásnak eltérő jellemzői és követelményei vannak, így mindegyik újrapróbálkozási mechanizmus az adott szolgáltatásra van szabva.</span><span class="sxs-lookup"><span data-stu-id="4af3c-105">However, these differ because each service has different characteristics and requirements, and so each retry mechanism is tuned to a specific service.</span></span> <span data-ttu-id="4af3c-106">Ez az útmutató összefoglalást ad az Azure-szolgáltatások többségére jellemző újrapróbálkozási funkciókról, továbbá segítséget nyújt a szolgáltatás újrapróbálkozási mechanizmusának használatához, módosításához vagy kibővítéséhez.</span><span class="sxs-lookup"><span data-stu-id="4af3c-106">This guide summarizes the retry mechanism features for the majority of Azure services, and includes information to help you use, adapt, or extend the retry mechanism for that service.</span></span>

<span data-ttu-id="4af3c-107">Általános útmutatást az átmeneti hibák kezeléséhez, valamint a szolgáltatásokat és erőforrásokat célzó kapcsolatok és műveletek újrapróbálásához az [újrapróbálkozási útmutatásban](./transient-faults.md) talál.</span><span class="sxs-lookup"><span data-stu-id="4af3c-107">For general guidance on handling transient faults, and retrying connections and operations against services and resources, see [Retry guidance](./transient-faults.md).</span></span>

<span data-ttu-id="4af3c-108">A következő táblázat az útmutatóban érintett Azure-szolgáltatások újrapróbálkozási funkcióiról nyújt áttekintést.</span><span class="sxs-lookup"><span data-stu-id="4af3c-108">The following table summarizes the retry features for the Azure services described in this guidance.</span></span>

| <span data-ttu-id="4af3c-109">**Szolgáltatás**</span><span class="sxs-lookup"><span data-stu-id="4af3c-109">**Service**</span></span> | <span data-ttu-id="4af3c-110">**Újrapróbálkozási képességek**</span><span class="sxs-lookup"><span data-stu-id="4af3c-110">**Retry capabilities**</span></span> | <span data-ttu-id="4af3c-111">**Szabályzatkonfiguráció**</span><span class="sxs-lookup"><span data-stu-id="4af3c-111">**Policy configuration**</span></span> | <span data-ttu-id="4af3c-112">**Hatókör**</span><span class="sxs-lookup"><span data-stu-id="4af3c-112">**Scope**</span></span> | <span data-ttu-id="4af3c-113">**Telemetriafunkciók**</span><span class="sxs-lookup"><span data-stu-id="4af3c-113">**Telemetry features**</span></span> |
| --- | --- | --- | --- | --- |
| <span data-ttu-id="4af3c-114">**[Azure Active Directory](#azure-active-directory)**</span><span class="sxs-lookup"><span data-stu-id="4af3c-114">**[Azure Active Directory](#azure-active-directory)**</span></span> |<span data-ttu-id="4af3c-115">Natív, az ADAL-kódtár része</span><span class="sxs-lookup"><span data-stu-id="4af3c-115">Native in ADAL library</span></span> |<span data-ttu-id="4af3c-116">Beágyazva az ADAL-kódtárba</span><span class="sxs-lookup"><span data-stu-id="4af3c-116">Embeded into ADAL library</span></span> |<span data-ttu-id="4af3c-117">Belső</span><span class="sxs-lookup"><span data-stu-id="4af3c-117">Internal</span></span> |<span data-ttu-id="4af3c-118">None</span><span class="sxs-lookup"><span data-stu-id="4af3c-118">None</span></span> |
| <span data-ttu-id="4af3c-119">**[Cosmos DB](#cosmos-db)**</span><span class="sxs-lookup"><span data-stu-id="4af3c-119">**[Cosmos DB](#cosmos-db)**</span></span> |<span data-ttu-id="4af3c-120">Natív, a szolgáltatás része</span><span class="sxs-lookup"><span data-stu-id="4af3c-120">Native in service</span></span> |<span data-ttu-id="4af3c-121">Nem konfigurálható</span><span class="sxs-lookup"><span data-stu-id="4af3c-121">Non-configurable</span></span> |<span data-ttu-id="4af3c-122">Globális</span><span class="sxs-lookup"><span data-stu-id="4af3c-122">Global</span></span> |<span data-ttu-id="4af3c-123">TraceSource</span><span class="sxs-lookup"><span data-stu-id="4af3c-123">TraceSource</span></span> |
| <span data-ttu-id="4af3c-124">**[Az Event Hubs](#event-hubs)**</span><span class="sxs-lookup"><span data-stu-id="4af3c-124">**[Event Hubs](#event-hubs)**</span></span> |<span data-ttu-id="4af3c-125">Natív, az ügyfél része</span><span class="sxs-lookup"><span data-stu-id="4af3c-125">Native in client</span></span> |<span data-ttu-id="4af3c-126">Szoftveres</span><span class="sxs-lookup"><span data-stu-id="4af3c-126">Programmatic</span></span> |<span data-ttu-id="4af3c-127">Ügyfél</span><span class="sxs-lookup"><span data-stu-id="4af3c-127">Client</span></span> |<span data-ttu-id="4af3c-128">None</span><span class="sxs-lookup"><span data-stu-id="4af3c-128">None</span></span> |
| <span data-ttu-id="4af3c-129">**[A redis Cache](#azure-redis-cache)**</span><span class="sxs-lookup"><span data-stu-id="4af3c-129">**[Redis Cache](#azure-redis-cache)**</span></span> |<span data-ttu-id="4af3c-130">Natív, az ügyfél része</span><span class="sxs-lookup"><span data-stu-id="4af3c-130">Native in client</span></span> |<span data-ttu-id="4af3c-131">Szoftveres</span><span class="sxs-lookup"><span data-stu-id="4af3c-131">Programmatic</span></span> |<span data-ttu-id="4af3c-132">Ügyfél</span><span class="sxs-lookup"><span data-stu-id="4af3c-132">Client</span></span> |<span data-ttu-id="4af3c-133">TextWriter</span><span class="sxs-lookup"><span data-stu-id="4af3c-133">TextWriter</span></span> |
| <span data-ttu-id="4af3c-134">**[Keresés](#azure-search)**</span><span class="sxs-lookup"><span data-stu-id="4af3c-134">**[Search](#azure-search)**</span></span> |<span data-ttu-id="4af3c-135">Natív, az ügyfél része</span><span class="sxs-lookup"><span data-stu-id="4af3c-135">Native in client</span></span> |<span data-ttu-id="4af3c-136">Szoftveres</span><span class="sxs-lookup"><span data-stu-id="4af3c-136">Programmatic</span></span> |<span data-ttu-id="4af3c-137">Ügyfél</span><span class="sxs-lookup"><span data-stu-id="4af3c-137">Client</span></span> |<span data-ttu-id="4af3c-138">ETW vagy egyéni</span><span class="sxs-lookup"><span data-stu-id="4af3c-138">ETW or Custom</span></span> |
| <span data-ttu-id="4af3c-139">**[Service Bus](#service-bus)**</span><span class="sxs-lookup"><span data-stu-id="4af3c-139">**[Service Bus](#service-bus)**</span></span> |<span data-ttu-id="4af3c-140">Natív, az ügyfél része</span><span class="sxs-lookup"><span data-stu-id="4af3c-140">Native in client</span></span> |<span data-ttu-id="4af3c-141">Szoftveres</span><span class="sxs-lookup"><span data-stu-id="4af3c-141">Programmatic</span></span> |<span data-ttu-id="4af3c-142">Névtérkezelő, üzenetkezelési előállító vagy ügyfél</span><span class="sxs-lookup"><span data-stu-id="4af3c-142">Namespace Manager, Messaging Factory, and Client</span></span> |<span data-ttu-id="4af3c-143">ETW</span><span class="sxs-lookup"><span data-stu-id="4af3c-143">ETW</span></span> |
| <span data-ttu-id="4af3c-144">**[Service Fabric](#service-fabric)**</span><span class="sxs-lookup"><span data-stu-id="4af3c-144">**[Service Fabric](#service-fabric)**</span></span> |<span data-ttu-id="4af3c-145">Natív, az ügyfél része</span><span class="sxs-lookup"><span data-stu-id="4af3c-145">Native in client</span></span> |<span data-ttu-id="4af3c-146">Szoftveres</span><span class="sxs-lookup"><span data-stu-id="4af3c-146">Programmatic</span></span> |<span data-ttu-id="4af3c-147">Ügyfél</span><span class="sxs-lookup"><span data-stu-id="4af3c-147">Client</span></span> |<span data-ttu-id="4af3c-148">None</span><span class="sxs-lookup"><span data-stu-id="4af3c-148">None</span></span> | 
| <span data-ttu-id="4af3c-149">**[SQL Database with ADO.NET](#sql-database-using-adonet)**</span><span class="sxs-lookup"><span data-stu-id="4af3c-149">**[SQL Database with ADO.NET](#sql-database-using-adonet)**</span></span> |[<span data-ttu-id="4af3c-150">Polly</span><span class="sxs-lookup"><span data-stu-id="4af3c-150">Polly</span></span>](#transient-fault-handling-with-polly) |<span data-ttu-id="4af3c-151">Deklaratív és szoftveres</span><span class="sxs-lookup"><span data-stu-id="4af3c-151">Declarative and programmatic</span></span> |<span data-ttu-id="4af3c-152">Önálló utasítások vagy kódblokkok</span><span class="sxs-lookup"><span data-stu-id="4af3c-152">Single statements or blocks of code</span></span> |<span data-ttu-id="4af3c-153">Egyéni</span><span class="sxs-lookup"><span data-stu-id="4af3c-153">Custom</span></span> |
| <span data-ttu-id="4af3c-154">**[SQL Database with Entity Framework](#sql-database-using-entity-framework-6)**</span><span class="sxs-lookup"><span data-stu-id="4af3c-154">**[SQL Database with Entity Framework](#sql-database-using-entity-framework-6)**</span></span> |<span data-ttu-id="4af3c-155">Natív, az ügyfél része</span><span class="sxs-lookup"><span data-stu-id="4af3c-155">Native in client</span></span> |<span data-ttu-id="4af3c-156">Szoftveres</span><span class="sxs-lookup"><span data-stu-id="4af3c-156">Programmatic</span></span> |<span data-ttu-id="4af3c-157">Alkalmazástartományonként globális</span><span class="sxs-lookup"><span data-stu-id="4af3c-157">Global per AppDomain</span></span> |<span data-ttu-id="4af3c-158">None</span><span class="sxs-lookup"><span data-stu-id="4af3c-158">None</span></span> |
| <span data-ttu-id="4af3c-159">**[SQL Database with Entity Framework Core](#sql-database-using-entity-framework-core)**</span><span class="sxs-lookup"><span data-stu-id="4af3c-159">**[SQL Database with Entity Framework Core](#sql-database-using-entity-framework-core)**</span></span> |<span data-ttu-id="4af3c-160">Natív, az ügyfél része</span><span class="sxs-lookup"><span data-stu-id="4af3c-160">Native in client</span></span> |<span data-ttu-id="4af3c-161">Szoftveres</span><span class="sxs-lookup"><span data-stu-id="4af3c-161">Programmatic</span></span> |<span data-ttu-id="4af3c-162">Alkalmazástartományonként globális</span><span class="sxs-lookup"><span data-stu-id="4af3c-162">Global per AppDomain</span></span> |<span data-ttu-id="4af3c-163">None</span><span class="sxs-lookup"><span data-stu-id="4af3c-163">None</span></span> |
| <span data-ttu-id="4af3c-164">**[Tároló](#azure-storage)**</span><span class="sxs-lookup"><span data-stu-id="4af3c-164">**[Storage](#azure-storage)**</span></span> |<span data-ttu-id="4af3c-165">Natív, az ügyfél része</span><span class="sxs-lookup"><span data-stu-id="4af3c-165">Native in client</span></span> |<span data-ttu-id="4af3c-166">Szoftveres</span><span class="sxs-lookup"><span data-stu-id="4af3c-166">Programmatic</span></span> |<span data-ttu-id="4af3c-167">Ügyfél- és különálló műveletek</span><span class="sxs-lookup"><span data-stu-id="4af3c-167">Client and individual operations</span></span> |<span data-ttu-id="4af3c-168">TraceSource</span><span class="sxs-lookup"><span data-stu-id="4af3c-168">TraceSource</span></span> |

> [!NOTE]
> <span data-ttu-id="4af3c-169">Az Azure beépített a legtöbb ismételje meg a mechanizmusok, ott jelenleg nem lehet alkalmazni, ha a különböző típusú hiba vagy kivétel eltérő újrapróbálkozási szabályzatok.</span><span class="sxs-lookup"><span data-stu-id="4af3c-169">For most of the Azure built-in retry mechanisms, there is currently no way apply a different retry policy for different types of error or exception.</span></span> <span data-ttu-id="4af3c-170">Az optimális teljesítményt és a rendelkezésre állást biztosító szabályzatot kell konfigurálnia.</span><span class="sxs-lookup"><span data-stu-id="4af3c-170">You should configure a policy that provides the optimum average performance and availability.</span></span> <span data-ttu-id="4af3c-171">A szabályzat finomhangolásához elemezze a naplófájlokat, hogy megállapítsa, milyen típusú átmeneti hibák szoktak történni.</span><span class="sxs-lookup"><span data-stu-id="4af3c-171">One way to fine-tune the policy is to analyze log files to determine the type of transient faults that are occurring.</span></span> 

## <a name="azure-active-directory"></a><span data-ttu-id="4af3c-172">Azure Active Directory</span><span class="sxs-lookup"><span data-stu-id="4af3c-172">Azure Active Directory</span></span>
<span data-ttu-id="4af3c-173">Az Azure Active Directory egy átfogó, felhőalapú identitás- és hozzáférés-kezelő megoldás, amely ötvözi az alapvető címtárszolgáltatásokat, a fejlett identitáskezelést, a biztonsági szolgáltatásokat és az alkalmazáshozzáférés-felügyeletet.</span><span class="sxs-lookup"><span data-stu-id="4af3c-173">Azure Active Directory (Azure AD) is a comprehensive identity and access management cloud solution that combines core directory services, advanced identity governance, security, and application access management.</span></span> <span data-ttu-id="4af3c-174">Az Azure AD ezenkívül identitáskezelő platformot kínál a fejlesztőknek, hogy központi szabályzatokon és szabályokon alapuló hozzáférés-vezérléssel bővíthessék az alkalmazásaikat.</span><span class="sxs-lookup"><span data-stu-id="4af3c-174">Azure AD also offers developers an identity management platform to deliver access control to their applications, based on centralized policy and rules.</span></span>

> [!NOTE]
> <span data-ttu-id="4af3c-175">Felügyeltszolgáltatás-identitás végpontokon újrapróbálkozási útmutatóért lásd: [egy Azure virtuális gépek Felügyeltszolgáltatás-identitás (MSI) használata a token beszerzéséhez](/azure/active-directory/managed-service-identity/how-to-use-vm-token#error-handling).</span><span class="sxs-lookup"><span data-stu-id="4af3c-175">For retry guidance on Managed Service Identity endpoints, see [How to use an Azure VM Managed Service Identity (MSI) for token acquisition](/azure/active-directory/managed-service-identity/how-to-use-vm-token#error-handling).</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="4af3c-176">Újrapróbálkozási mechanizmus</span><span class="sxs-lookup"><span data-stu-id="4af3c-176">Retry mechanism</span></span>
<span data-ttu-id="4af3c-177">Az Azure Active Directory beépített újrapróbálkozási mechanizmussal rendelkezik az Active Directory Authentication Library (ADAL) részeként.</span><span class="sxs-lookup"><span data-stu-id="4af3c-177">There is a built-in retry mechanism for Azure Active Directory in the Active Directory Authentication Library (ADAL).</span></span> <span data-ttu-id="4af3c-178">A váratlan zárolások elkerülése érdekében azt javasoljuk, hogy külső kódtárak és alkalmazáskódok **ne** próbálkozhassanak újra sikertelen kapcsolódás esetén, és ezek újrapróbálkozásait az ADAL kezelje.</span><span class="sxs-lookup"><span data-stu-id="4af3c-178">To avoid unexpected lockouts, we recommend that third party libraries and application code do **not** retry failed connections, but allow ADAL to handle retries.</span></span> 

### <a name="retry-usage-guidance"></a><span data-ttu-id="4af3c-179">Újrapróbálkozásokra vonatkozó útmutató</span><span class="sxs-lookup"><span data-stu-id="4af3c-179">Retry usage guidance</span></span>
<span data-ttu-id="4af3c-180">Ügyeljen a következőkre az Azure Active Directory használata során:</span><span class="sxs-lookup"><span data-stu-id="4af3c-180">Consider the following guidelines when using Azure Active Directory:</span></span>

* <span data-ttu-id="4af3c-181">Amikor csak lehetséges, az ADAL-kódtárat és az újrapróbálkozások beépített támogatását használja.</span><span class="sxs-lookup"><span data-stu-id="4af3c-181">When possible, use the ADAL library and the built-in support for retries.</span></span>
* <span data-ttu-id="4af3c-182">Ha a REST API-t használ az Azure Active Directory, az eredménykód 429-es (túl sok kérés) vagy az 5xx tartományban hiba esetén próbálja megismételni a műveletet.</span><span class="sxs-lookup"><span data-stu-id="4af3c-182">If you are using the REST API for Azure Active Directory, retry the operation if the result code is 429 (Too Many Requests) or an error in the 5xx range.</span></span> <span data-ttu-id="4af3c-183">Más hibák esetében ne engedélyezze az újrapróbálkozást.</span><span class="sxs-lookup"><span data-stu-id="4af3c-183">Do not retry for any other errors.</span></span>
* <span data-ttu-id="4af3c-184">Az exponenciális visszatartási szabályzat használatát az Azure Active Directory Batch-forgatókönyvei esetében javasoljuk.</span><span class="sxs-lookup"><span data-stu-id="4af3c-184">An exponential back-off policy is recommended for use in batch scenarios with Azure Active Directory.</span></span>

<span data-ttu-id="4af3c-185">A következő kezdőbeállításokat javasoljuk az újrapróbálkozási műveletekhez.</span><span class="sxs-lookup"><span data-stu-id="4af3c-185">Consider starting with the following settings for retrying operations.</span></span> <span data-ttu-id="4af3c-186">Ezek általános célú beállítások, ezért javasoljuk, hogy monitorozza műveleteit, és finomhangolja az értékeket saját igényei szerint.</span><span class="sxs-lookup"><span data-stu-id="4af3c-186">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="4af3c-187">**Környezet**</span><span class="sxs-lookup"><span data-stu-id="4af3c-187">**Context**</span></span> | <span data-ttu-id="4af3c-188">**Mintacél E2E<br />max. késleltetése**</span><span class="sxs-lookup"><span data-stu-id="4af3c-188">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="4af3c-189">**Újrapróbálkozási stratégia**</span><span class="sxs-lookup"><span data-stu-id="4af3c-189">**Retry strategy**</span></span> | <span data-ttu-id="4af3c-190">**Beállítások**</span><span class="sxs-lookup"><span data-stu-id="4af3c-190">**Settings**</span></span> | <span data-ttu-id="4af3c-191">**Értékek**</span><span class="sxs-lookup"><span data-stu-id="4af3c-191">**Values**</span></span> | <span data-ttu-id="4af3c-192">**Működési elv**</span><span class="sxs-lookup"><span data-stu-id="4af3c-192">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="4af3c-193">Interaktív, felhasználói felület</span><span class="sxs-lookup"><span data-stu-id="4af3c-193">Interactive, UI,</span></span><br /><span data-ttu-id="4af3c-194">vagy előtér</span><span class="sxs-lookup"><span data-stu-id="4af3c-194">or foreground</span></span> |<span data-ttu-id="4af3c-195">2 másodperc</span><span class="sxs-lookup"><span data-stu-id="4af3c-195">2 sec</span></span> |<span data-ttu-id="4af3c-196">FixedInterval</span><span class="sxs-lookup"><span data-stu-id="4af3c-196">FixedInterval</span></span> |<span data-ttu-id="4af3c-197">Ismétlések száma</span><span class="sxs-lookup"><span data-stu-id="4af3c-197">Retry count</span></span><br /><span data-ttu-id="4af3c-198">Újrapróbálkozási időköz</span><span class="sxs-lookup"><span data-stu-id="4af3c-198">Retry interval</span></span><br /><span data-ttu-id="4af3c-199">Első gyors újrapróbálkozás</span><span class="sxs-lookup"><span data-stu-id="4af3c-199">First fast retry</span></span> |<span data-ttu-id="4af3c-200">3</span><span class="sxs-lookup"><span data-stu-id="4af3c-200">3</span></span><br /><span data-ttu-id="4af3c-201">500 ms</span><span class="sxs-lookup"><span data-stu-id="4af3c-201">500 ms</span></span><br /><span data-ttu-id="4af3c-202">true</span><span class="sxs-lookup"><span data-stu-id="4af3c-202">true</span></span> |<span data-ttu-id="4af3c-203">1. kísérlet – 0 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="4af3c-203">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="4af3c-204">2. kísérlet – 500 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="4af3c-204">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="4af3c-205">3. kísérlet – 500 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="4af3c-205">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="4af3c-206">Háttér vagy</span><span class="sxs-lookup"><span data-stu-id="4af3c-206">Background or</span></span><br /><span data-ttu-id="4af3c-207">kötegelt</span><span class="sxs-lookup"><span data-stu-id="4af3c-207">batch</span></span> |<span data-ttu-id="4af3c-208">60 másodperc</span><span class="sxs-lookup"><span data-stu-id="4af3c-208">60 sec</span></span> |<span data-ttu-id="4af3c-209">ExponentialBackoff</span><span class="sxs-lookup"><span data-stu-id="4af3c-209">ExponentialBackoff</span></span> |<span data-ttu-id="4af3c-210">Ismétlések száma</span><span class="sxs-lookup"><span data-stu-id="4af3c-210">Retry count</span></span><br /><span data-ttu-id="4af3c-211">Visszatartás (min.)</span><span class="sxs-lookup"><span data-stu-id="4af3c-211">Min back-off</span></span><br /><span data-ttu-id="4af3c-212">Visszatartás (max.)</span><span class="sxs-lookup"><span data-stu-id="4af3c-212">Max back-off</span></span><br /><span data-ttu-id="4af3c-213">Visszatartás (változás)</span><span class="sxs-lookup"><span data-stu-id="4af3c-213">Delta back-off</span></span><br /><span data-ttu-id="4af3c-214">Első gyors újrapróbálkozás</span><span class="sxs-lookup"><span data-stu-id="4af3c-214">First fast retry</span></span> |<span data-ttu-id="4af3c-215">5</span><span class="sxs-lookup"><span data-stu-id="4af3c-215">5</span></span><br /><span data-ttu-id="4af3c-216">0 másodperc</span><span class="sxs-lookup"><span data-stu-id="4af3c-216">0 sec</span></span><br /><span data-ttu-id="4af3c-217">60 másodperc</span><span class="sxs-lookup"><span data-stu-id="4af3c-217">60 sec</span></span><br /><span data-ttu-id="4af3c-218">2 másodperc</span><span class="sxs-lookup"><span data-stu-id="4af3c-218">2 sec</span></span><br /><span data-ttu-id="4af3c-219">false</span><span class="sxs-lookup"><span data-stu-id="4af3c-219">false</span></span> |<span data-ttu-id="4af3c-220">1. kísérlet – 0 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="4af3c-220">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="4af3c-221">2. kísérlet – kb. 2 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="4af3c-221">Attempt 2 - delay ~2 sec</span></span><br /><span data-ttu-id="4af3c-222">3. kísérlet – kb. 6 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="4af3c-222">Attempt 3 - delay ~6 sec</span></span><br /><span data-ttu-id="4af3c-223">4. kísérlet – kb. 14 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="4af3c-223">Attempt 4 - delay ~14 sec</span></span><br /><span data-ttu-id="4af3c-224">5. kísérlet – kb. 30 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="4af3c-224">Attempt 5 - delay ~30 sec</span></span> |

### <a name="more-information"></a><span data-ttu-id="4af3c-225">További információ</span><span class="sxs-lookup"><span data-stu-id="4af3c-225">More information</span></span>
* <span data-ttu-id="4af3c-226">[Az Azure Active Directory hitelesítési kódtárai][adal]</span><span class="sxs-lookup"><span data-stu-id="4af3c-226">[Azure Active Directory Authentication Libraries][adal]</span></span>

## <a name="cosmos-db"></a><span data-ttu-id="4af3c-227">Cosmos DB</span><span class="sxs-lookup"><span data-stu-id="4af3c-227">Cosmos DB</span></span>

<span data-ttu-id="4af3c-228">A Cosmos DB egy teljes körűen felügyelt, többmodelles adatbázis-szolgáltatás, amely támogatja a séma nélküli JSON-adatok használatát.</span><span class="sxs-lookup"><span data-stu-id="4af3c-228">Cosmos DB is a fully-managed multi-model database that supports schema-less JSON data.</span></span> <span data-ttu-id="4af3c-229">Teljesítménye konfigurálható és megbízható, natív JavaScript-tranzakciófeldolgozást kínál, és mivel felhőbeli felhasználásra készült, rugalmasan méretezhető.</span><span class="sxs-lookup"><span data-stu-id="4af3c-229">It offers configurable and reliable performance, native JavaScript transactional processing, and is built for the cloud with elastic scale.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="4af3c-230">Újrapróbálkozási mechanizmus</span><span class="sxs-lookup"><span data-stu-id="4af3c-230">Retry mechanism</span></span>
<span data-ttu-id="4af3c-231">A `DocumentClient` osztály automatikusan újrapróbálkozik a sikertelen kísérletekkel.</span><span class="sxs-lookup"><span data-stu-id="4af3c-231">The `DocumentClient` class automatically retries failed attempts.</span></span> <span data-ttu-id="4af3c-232">Az újrapróbálkozások számának és a maximális várakozási idő beállításához a [ConnectionPolicy.RetryOptions] konfigurálása szükséges.</span><span class="sxs-lookup"><span data-stu-id="4af3c-232">To set the number of retries and the maximum wait time, configure [ConnectionPolicy.RetryOptions].</span></span> <span data-ttu-id="4af3c-233">Az ügyfél által okozott kivételek vagy túlmutatnak az újrapróbálkozási szabályzaton, vagy nem átmeneti hibák.</span><span class="sxs-lookup"><span data-stu-id="4af3c-233">Exceptions that the client raises are either beyond the retry policy or are not transient errors.</span></span>

<span data-ttu-id="4af3c-234">Ha a Cosmos DB korlátozza az ügyfél kísérleteit, a 429-es HTTP-hibaüzenetet adja vissza.</span><span class="sxs-lookup"><span data-stu-id="4af3c-234">If Cosmos DB throttles the client, it returns an HTTP 429 error.</span></span> <span data-ttu-id="4af3c-235">Ellenőrizze a `DocumentClientException` állapotkódját.</span><span class="sxs-lookup"><span data-stu-id="4af3c-235">Check the status code in the `DocumentClientException`.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="4af3c-236">Szabályzatkonfiguráció</span><span class="sxs-lookup"><span data-stu-id="4af3c-236">Policy configuration</span></span>
<span data-ttu-id="4af3c-237">A következő táblázatban a `RetryOptions` osztály alapértelmezett beállításait tekintheti meg.</span><span class="sxs-lookup"><span data-stu-id="4af3c-237">The following table shows the default settings for the `RetryOptions` class.</span></span>

| <span data-ttu-id="4af3c-238">Beállítás</span><span class="sxs-lookup"><span data-stu-id="4af3c-238">Setting</span></span> | <span data-ttu-id="4af3c-239">Alapértelmezett érték</span><span class="sxs-lookup"><span data-stu-id="4af3c-239">Default value</span></span> | <span data-ttu-id="4af3c-240">Leírás</span><span class="sxs-lookup"><span data-stu-id="4af3c-240">Description</span></span> |
| --- | --- | --- |
| <span data-ttu-id="4af3c-241">MaxRetryAttemptsOnThrottledRequests</span><span class="sxs-lookup"><span data-stu-id="4af3c-241">MaxRetryAttemptsOnThrottledRequests</span></span> |<span data-ttu-id="4af3c-242">9</span><span class="sxs-lookup"><span data-stu-id="4af3c-242">9</span></span> |<span data-ttu-id="4af3c-243">Az újrapróbálkozások maximális száma abban az esetben, ha a Cosmos DB az ügyfélre alkalmazott korlátozása miatt sikertelen a kísérlet.</span><span class="sxs-lookup"><span data-stu-id="4af3c-243">The maximum number of retries if the request fails because Cosmos DB applied rate limiting on the client.</span></span> |
| <span data-ttu-id="4af3c-244">MaxRetryWaitTimeInSeconds</span><span class="sxs-lookup"><span data-stu-id="4af3c-244">MaxRetryWaitTimeInSeconds</span></span> |<span data-ttu-id="4af3c-245">30</span><span class="sxs-lookup"><span data-stu-id="4af3c-245">30</span></span> |<span data-ttu-id="4af3c-246">A maximális újrapróbálkozási idő másodpercben.</span><span class="sxs-lookup"><span data-stu-id="4af3c-246">The maximum retry time in seconds.</span></span> |

### <a name="example"></a><span data-ttu-id="4af3c-247">Példa</span><span class="sxs-lookup"><span data-stu-id="4af3c-247">Example</span></span>
```csharp
DocumentClient client = new DocumentClient(new Uri(endpoint), authKey); ;
var options = client.ConnectionPolicy.RetryOptions;
options.MaxRetryAttemptsOnThrottledRequests = 5;
options.MaxRetryWaitTimeInSeconds = 15;
```

### <a name="telemetry"></a><span data-ttu-id="4af3c-248">Telemetria</span><span class="sxs-lookup"><span data-stu-id="4af3c-248">Telemetry</span></span>
<span data-ttu-id="4af3c-249">Az újrapróbálkozási kísérletek strukturálatlan nyomkövetési üzenetekként lesznek naplózva a .NET **TraceSource** használatával.</span><span class="sxs-lookup"><span data-stu-id="4af3c-249">Retry attempts are logged as unstructured trace messages through a .NET **TraceSource**.</span></span> <span data-ttu-id="4af3c-250">Az események rögzítéséhez és megfelelő célnaplóba való írásához egy **TraceListener** osztályt kell konfigurálnia.</span><span class="sxs-lookup"><span data-stu-id="4af3c-250">You must configure a **TraceListener** to capture the events and write them to a suitable destination log.</span></span>

<span data-ttu-id="4af3c-251">Amennyiben például a következőt adja hozzá az App.config fájlhoz, a szövegfájlban a végrehajtható fájllal megegyező helyen jönnek létre a nyomkövetési adatok:</span><span class="sxs-lookup"><span data-stu-id="4af3c-251">For example, if you add the following to your App.config file, traces will be generated in a text file in the same location as the executable:</span></span>

```xml
<configuration>
  <system.diagnostics>
    <switches>
      <add name="SourceSwitch" value="Verbose"/>
    </switches>
    <sources>
      <source name="DocDBTrace" switchName="SourceSwitch" switchType="System.Diagnostics.SourceSwitch" >
        <listeners>
          <add name="MyTextListener" type="System.Diagnostics.TextWriterTraceListener" traceOutputOptions="DateTime,ProcessId,ThreadId" initializeData="CosmosDBTrace.txt"></add>
        </listeners>
      </source>
    </sources>
  </system.diagnostics>
</configuration>
```

## <a name="event-hubs"></a><span data-ttu-id="4af3c-252">Event Hubs</span><span class="sxs-lookup"><span data-stu-id="4af3c-252">Event Hubs</span></span>

<span data-ttu-id="4af3c-253">Az Azure Event Hubs egy rendkívül nagy kapacitású, telemetriai adatokat betöltő szolgáltatás, amely események millióinak adatait képes összegyűjteni, átalakítani és tárolni.</span><span class="sxs-lookup"><span data-stu-id="4af3c-253">Azure Event Hubs is a hyper-scale telemetry ingestion service that collects, transforms, and stores millions of events.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="4af3c-254">Újrapróbálkozási mechanizmus</span><span class="sxs-lookup"><span data-stu-id="4af3c-254">Retry mechanism</span></span>
<span data-ttu-id="4af3c-255">Az Azure Event Hubs Client Library újrapróbálkozási viselkedését az `EventHubClient` osztály `RetryPolicy` tulajdonsága vezérli.</span><span class="sxs-lookup"><span data-stu-id="4af3c-255">Retry behavior in the Azure Event Hubs Client Library is controlled by the `RetryPolicy` property on the `EventHubClient` class.</span></span> <span data-ttu-id="4af3c-256">Az alapértelmezett szabályzat exponenciális visszatartással végzi el az újrapróbálkozást, ha az Azure Event Hub egy átmeneti `EventHubsException` vagy egy `OperationCanceledException` választ ad.</span><span class="sxs-lookup"><span data-stu-id="4af3c-256">The default policy retries with exponential backoff when Azure Event Hub returns a transient `EventHubsException` or an `OperationCanceledException`.</span></span>

### <a name="example"></a><span data-ttu-id="4af3c-257">Példa</span><span class="sxs-lookup"><span data-stu-id="4af3c-257">Example</span></span>
```csharp
EventHubClient client = EventHubClient.CreateFromConnectionString("[event_hub_connection_string]");
client.RetryPolicy = RetryPolicy.Default;
```

### <a name="more-information"></a><span data-ttu-id="4af3c-258">További információ</span><span class="sxs-lookup"><span data-stu-id="4af3c-258">More information</span></span>
[<span data-ttu-id="4af3c-259">.NET standard ügyféloldali kódtár az Azure Event Hubshoz</span><span class="sxs-lookup"><span data-stu-id="4af3c-259"> .NET Standard client library for Azure Event Hubs</span></span>](https://github.com/Azure/azure-event-hubs-dotnet)

## <a name="azure-redis-cache"></a><span data-ttu-id="4af3c-260">Azure Redis Cache</span><span class="sxs-lookup"><span data-stu-id="4af3c-260">Azure Redis Cache</span></span>
<span data-ttu-id="4af3c-261">Az Azure Redis Cache gyors adathozzáférést és alacsony késést kínáló gyorsítótár-szolgáltatás, amely a népszerű, nyílt forráskódú Redis Cache-re épül.</span><span class="sxs-lookup"><span data-stu-id="4af3c-261">Azure Redis Cache is a fast data access and low latency cache service based on the popular open source Redis Cache.</span></span> <span data-ttu-id="4af3c-262">Biztonságos, a Microsoft felügyeli, és az Azure bármelyik alkalmazásából elérhető.</span><span class="sxs-lookup"><span data-stu-id="4af3c-262">It is secure, managed by Microsoft, and is accessible from any application in Azure.</span></span>

<span data-ttu-id="4af3c-263">Ebben az útmutatóban azt feltételezzük, hogy a StackExchange.Redis ügyfelet használja a gyorsítótár eléréséhez.</span><span class="sxs-lookup"><span data-stu-id="4af3c-263">The guidance in this section is based on using the StackExchange.Redis client to access the cache.</span></span> <span data-ttu-id="4af3c-264">A további alkalmas ügyfelek listája a [Redis webhelyén](http://redis.io/clients) tekinthető meg, ám ezeknek eltérő újrapróbálkozási mechanizmusai lehetnek.</span><span class="sxs-lookup"><span data-stu-id="4af3c-264">A list of other suitable clients can be found on the [Redis website](http://redis.io/clients), and these may have different retry mechanisms.</span></span>

<span data-ttu-id="4af3c-265">Vegye figyelembe, hogy a StackExchange.Redis ügyfél egyetlen kapcsolaton keresztül végez multiplexálást.</span><span class="sxs-lookup"><span data-stu-id="4af3c-265">Note that the StackExchange.Redis client uses multiplexing through a single connection.</span></span> <span data-ttu-id="4af3c-266">A javasolt felhasználás az, ha létrehozza az ügyfél egy példányát az alkalmazás indításakor, és ezt a példányt használja a gyorsítótár elérését célzó összes művelethez.</span><span class="sxs-lookup"><span data-stu-id="4af3c-266">The recommended usage is to create an instance of the client at application startup and use this instance for all operations against the cache.</span></span> <span data-ttu-id="4af3c-267">Így a gyorsítótárral való kapcsolat csak egyszer jön létre, ezért az ebben a szakaszban leírt összes útmutatás ezen első kapcsolat újrapróbálkozási szabályzatára vonatkozik, nem pedig a gyorsítótárhoz hozzáférő egyes műveletekre.</span><span class="sxs-lookup"><span data-stu-id="4af3c-267">For this reason, the connection to the cache is made only once, and so all of the guidance in this section is related to the retry policy for this initial connection—and not for each operation that accesses the cache.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="4af3c-268">Újrapróbálkozási mechanizmus</span><span class="sxs-lookup"><span data-stu-id="4af3c-268">Retry mechanism</span></span>
<span data-ttu-id="4af3c-269">A StackExchange.Redis ügyfél egy számos beállítással konfigurált kapcsolatkezelő osztályt használ. Néhány példa ezekre a beállításokra:</span><span class="sxs-lookup"><span data-stu-id="4af3c-269">The StackExchange.Redis client uses a connection manager class that is configured through a set of options, incuding:</span></span>

- <span data-ttu-id="4af3c-270">**ConnectRetry**.</span><span class="sxs-lookup"><span data-stu-id="4af3c-270">**ConnectRetry**.</span></span> <span data-ttu-id="4af3c-271">Ennyi alkalommal próbálkozik újra a gyorsítótárhoz való sikertelen kapcsolódás esetén.</span><span class="sxs-lookup"><span data-stu-id="4af3c-271">The number of times a failed connection to the cache will be retried.</span></span>
- <span data-ttu-id="4af3c-272">**ReconnectRetryPolicy**.</span><span class="sxs-lookup"><span data-stu-id="4af3c-272">**ReconnectRetryPolicy**.</span></span> <span data-ttu-id="4af3c-273">A használt újrapróbálkozási stratégia.</span><span class="sxs-lookup"><span data-stu-id="4af3c-273">The retry strategy to use.</span></span>
- <span data-ttu-id="4af3c-274">**ConnectTimeout**.</span><span class="sxs-lookup"><span data-stu-id="4af3c-274">**ConnectTimeout**.</span></span> <span data-ttu-id="4af3c-275">A maximális várakozási idő milliszekundumban.</span><span class="sxs-lookup"><span data-stu-id="4af3c-275">The maximum waiting time in milliseconds.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="4af3c-276">Szabályzatkonfiguráció</span><span class="sxs-lookup"><span data-stu-id="4af3c-276">Policy configuration</span></span>
<span data-ttu-id="4af3c-277">Az újrapróbálkozási szabályzat konfigurálása szoftveresen történik. Az ügyfél beállításait a gyorsítótárhoz való kapcsolódás előtt kell megadni.</span><span class="sxs-lookup"><span data-stu-id="4af3c-277">Retry policies are configured programmatically by setting the options for the client before connecting to the cache.</span></span> <span data-ttu-id="4af3c-278">Ehhez létre kell hozni a **ConfigurationOptions** osztály egy példányát, feltölteni adatokkal a tulajdonságait, majd továbbítani azt a **Connect** metódusnak.</span><span class="sxs-lookup"><span data-stu-id="4af3c-278">This can be done by creating an instance of the **ConfigurationOptions** class, populating its properties, and passing it to the **Connect** method.</span></span>

<span data-ttu-id="4af3c-279">A beépített osztályok támogatják a lineáris (állandó) késleltetést, illetve az exponenciális visszatartást véletlenszerű újrapróbálkozási időközökkel.</span><span class="sxs-lookup"><span data-stu-id="4af3c-279">The built-in classes support linear (constant) delay and exponential backoff with randomized retry intervals.</span></span> <span data-ttu-id="4af3c-280">Ezenkívül létrehozhat egyéni újrapróbálkozási szabályzatot az **IReconnectRetryPolicy** felület implementálásával.</span><span class="sxs-lookup"><span data-stu-id="4af3c-280">You can also create a custom retry policy by implementing the **IReconnectRetryPolicy** interface.</span></span>

<span data-ttu-id="4af3c-281">A következő példa exponenciális visszatartással konfigurálja az újrapróbálkozási stratégiát.</span><span class="sxs-lookup"><span data-stu-id="4af3c-281">The following example configures a retry strategy using exponential backoff.</span></span>

```csharp
var deltaBackOffInMilliseconds = TimeSpan.FromSeconds(5).Milliseconds;
var maxDeltaBackOffInMilliseconds = TimeSpan.FromSeconds(20).Milliseconds;
var options = new ConfigurationOptions
{
    EndPoints = {"localhost"},
    ConnectRetry = 3,
    ReconnectRetryPolicy = new ExponentialRetry(deltaBackOffInMilliseconds, maxDeltaBackOffInMilliseconds),
    ConnectTimeout = 2000
};
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

<span data-ttu-id="4af3c-282">Egy másik lehetőség, hogy a beállításokat sztringként adja meg és továbbítja a **Connect** metódusnak.</span><span class="sxs-lookup"><span data-stu-id="4af3c-282">Alternatively, you can specify the options as a string, and pass this to the **Connect** method.</span></span> <span data-ttu-id="4af3c-283">Vegye figyelembe, hogy a **ReconnectRetryPolicy** tulajdonság nem állítható be ezen a módon, csak a programkódon keresztül.</span><span class="sxs-lookup"><span data-stu-id="4af3c-283">Note that the **ReconnectRetryPolicy** property cannot be set this way, only through code.</span></span>

```csharp
var options = "localhost,connectRetry=3,connectTimeout=2000";
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

<span data-ttu-id="4af3c-284">Amikor a gyorsítótárhoz csatlakozik, közvetlenül is megadhat beállításokat.</span><span class="sxs-lookup"><span data-stu-id="4af3c-284">You can also specify options directly when you connect to the cache.</span></span>

```csharp
var conn = ConnectionMultiplexer.Connect("redis0:6380,redis1:6380,connectRetry=3");
```

<span data-ttu-id="4af3c-285">További információért tekintse meg [a Stack Exchange Redis konfigurálását](https://stackexchange.github.io/StackExchange.Redis/Configuration) a StackExchange.Redis dokumentációjában.</span><span class="sxs-lookup"><span data-stu-id="4af3c-285">For more information, see [Stack Exchange Redis Configuration](https://stackexchange.github.io/StackExchange.Redis/Configuration) in the StackExchange.Redis documentation.</span></span>

<span data-ttu-id="4af3c-286">A következő táblázatban a beépített újrapróbálkozási szabályzat alapértelmezett beállításai láthatók.</span><span class="sxs-lookup"><span data-stu-id="4af3c-286">The following table shows the default settings for the built-in retry policy.</span></span>

| <span data-ttu-id="4af3c-287">**Környezet**</span><span class="sxs-lookup"><span data-stu-id="4af3c-287">**Context**</span></span> | <span data-ttu-id="4af3c-288">**Beállítás**</span><span class="sxs-lookup"><span data-stu-id="4af3c-288">**Setting**</span></span> | <span data-ttu-id="4af3c-289">**Alapértelmezett érték**</span><span class="sxs-lookup"><span data-stu-id="4af3c-289">**Default value**</span></span><br /><span data-ttu-id="4af3c-290">(v 1.2.2)</span><span class="sxs-lookup"><span data-stu-id="4af3c-290">(v 1.2.2)</span></span> | <span data-ttu-id="4af3c-291">**Jelentés**</span><span class="sxs-lookup"><span data-stu-id="4af3c-291">**Meaning**</span></span> |
| --- | --- | --- | --- |
| <span data-ttu-id="4af3c-292">ConfigurationOptions</span><span class="sxs-lookup"><span data-stu-id="4af3c-292">ConfigurationOptions</span></span> |<span data-ttu-id="4af3c-293">ConnectRetry</span><span class="sxs-lookup"><span data-stu-id="4af3c-293">ConnectRetry</span></span><br /><br /><span data-ttu-id="4af3c-294">ConnectTimeout</span><span class="sxs-lookup"><span data-stu-id="4af3c-294">ConnectTimeout</span></span><br /><br /><span data-ttu-id="4af3c-295">SyncTimeout</span><span class="sxs-lookup"><span data-stu-id="4af3c-295">SyncTimeout</span></span><br /><br /><span data-ttu-id="4af3c-296">ReconnectRetryPolicy</span><span class="sxs-lookup"><span data-stu-id="4af3c-296">ReconnectRetryPolicy</span></span> |<span data-ttu-id="4af3c-297">3</span><span class="sxs-lookup"><span data-stu-id="4af3c-297">3</span></span><br /><br /><span data-ttu-id="4af3c-298">Legfeljebb 5000 ms, plusz SyncTimeout</span><span class="sxs-lookup"><span data-stu-id="4af3c-298">Maximum 5000 ms plus SyncTimeout</span></span><br /><span data-ttu-id="4af3c-299">1000</span><span class="sxs-lookup"><span data-stu-id="4af3c-299">1000</span></span><br /><br /><span data-ttu-id="4af3c-300">LinearRetry 5000 ms</span><span class="sxs-lookup"><span data-stu-id="4af3c-300">LinearRetry 5000 ms</span></span> |<span data-ttu-id="4af3c-301">Ennyi alkalommal kell megismételni a csatlakozási kísérletet az első csatlakozási művelet során.</span><span class="sxs-lookup"><span data-stu-id="4af3c-301">The number of times to repeat connect attempts during the initial connection operation.</span></span><br /><span data-ttu-id="4af3c-302">A csatlakozási műveletek időtúllépési értéke (ms).</span><span class="sxs-lookup"><span data-stu-id="4af3c-302">Timeout (ms) for connect operations.</span></span> <span data-ttu-id="4af3c-303">Nem az újrapróbálkozás kísérletek közötti késleltetést jelzi.</span><span class="sxs-lookup"><span data-stu-id="4af3c-303">Not a delay between retry attempts.</span></span><br /><span data-ttu-id="4af3c-304">A szinkron műveletek számára biztosított idő (ms).</span><span class="sxs-lookup"><span data-stu-id="4af3c-304">Time (ms) to allow for synchronous operations.</span></span><br /><br /><span data-ttu-id="4af3c-305">Újrapróbálkozás 5000 ms időközönként.</span><span class="sxs-lookup"><span data-stu-id="4af3c-305">Retry every 5000 ms.</span></span>|

> [!NOTE]
> <span data-ttu-id="4af3c-306">A szinkron műveletek esetében a `SyncTimeout` hozzájárulhat a végpontok közötti késéshez, de a túl alacsony érték gyakori időtúllépéseket eredményezhet.</span><span class="sxs-lookup"><span data-stu-id="4af3c-306">For synchronous operations, `SyncTimeout` can add to the end-to-end latency, but setting the value too low can cause excessive timeouts.</span></span> <span data-ttu-id="4af3c-307">További információért lásd [az Azure Redis Cache hibaelhárítását][redis-cache-troubleshoot].</span><span class="sxs-lookup"><span data-stu-id="4af3c-307">See [How to troubleshoot Azure Redis Cache][redis-cache-troubleshoot].</span></span> <span data-ttu-id="4af3c-308">Általában kerülje a szinkron műveletek használatát, és alkalmazzon inkább aszinkron műveleteket.</span><span class="sxs-lookup"><span data-stu-id="4af3c-308">In general, avoid using synchronous operations, and use asynchronous operations instead.</span></span> <span data-ttu-id="4af3c-309">További információért lásd a [folyamatokat és a multiplexereket](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/PipelinesMultiplexers.md).</span><span class="sxs-lookup"><span data-stu-id="4af3c-309">For more information see [Pipelines and Multiplexers](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/PipelinesMultiplexers.md).</span></span>
>
>

### <a name="retry-usage-guidance"></a><span data-ttu-id="4af3c-310">Újrapróbálkozásokra vonatkozó útmutató</span><span class="sxs-lookup"><span data-stu-id="4af3c-310">Retry usage guidance</span></span>
<span data-ttu-id="4af3c-311">Ügyeljen a következőkre az Azure Redis Cache használata során:</span><span class="sxs-lookup"><span data-stu-id="4af3c-311">Consider the following guidelines when using Azure Redis Cache:</span></span>

* <span data-ttu-id="4af3c-312">A StackExchange Redis ügyfél kezeli a saját újrapróbálkozásait, de csak amikor az alkalmazás első indításakor próbál kapcsolódni a gyorsítótárhoz.</span><span class="sxs-lookup"><span data-stu-id="4af3c-312">The StackExchange Redis client manages its own retries, but only when establishing a connection to the cache when the application first starts.</span></span> <span data-ttu-id="4af3c-313">Meghatározhatja a kapcsolati időtúllépés értékét, az újrapróbálkozási kísérletek számát, valamint a kapcsolat létrehozására tett ismételt próbálkozások között eltelt időt, de az újrapróbálkozási szabályzat nem vonatkozik a gyorsítótárra irányuló műveletekre.</span><span class="sxs-lookup"><span data-stu-id="4af3c-313">You can configure the connection timeout, the number of retry attempts, and the time between retries to establish this connection, but the retry policy does not apply to operations against the cache.</span></span>
* <span data-ttu-id="4af3c-314">Nagy számú újrapróbálkozási kísérlet helyett érdemes lehet inkább az eredeti adatforráshoz csatlakozni.</span><span class="sxs-lookup"><span data-stu-id="4af3c-314">Instead of using a large number of retry attempts, consider falling back by accessing the original data source instead.</span></span>

### <a name="telemetry"></a><span data-ttu-id="4af3c-315">Telemetria</span><span class="sxs-lookup"><span data-stu-id="4af3c-315">Telemetry</span></span>
<span data-ttu-id="4af3c-316">**TextWriter** használatával adatokat gyűjthet a kapcsolatokról (más műveletekről azonban nem).</span><span class="sxs-lookup"><span data-stu-id="4af3c-316">You can collect information about connections (but not other operations) using a **TextWriter**.</span></span>

```csharp
var writer = new StringWriter();
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

<span data-ttu-id="4af3c-317">Az alábbi példa azt mutatja be, hogy ez milyen kimenetet eredményez.</span><span class="sxs-lookup"><span data-stu-id="4af3c-317">An example of the output this generates is shown below.</span></span>

```text
localhost:6379,connectTimeout=2000,connectRetry=3
1 unique nodes specified
Requesting tie-break from localhost:6379 > __Booksleeve_TieBreak...
Allowing endpoints 00:00:02 to respond...
localhost:6379 faulted: SocketFailure on PING
localhost:6379 failed to nominate (Faulted)
> UnableToResolvePhysicalConnection on GET
No masters detected
localhost:6379: Standalone v2.0.0, master; keep-alive: 00:01:00; int: Connecting; sub: Connecting; not in use: DidNotRespond
localhost:6379: int ops=0, qu=0, qs=0, qc=1, wr=0, sync=1, socks=2; sub ops=0, qu=0, qs=0, qc=0, wr=0, socks=2
Circular op-count snapshot; int: 0 (0.00 ops/s; spans 10s); sub: 0 (0.00 ops/s; spans 10s)
Sync timeouts: 0; fire and forget: 0; last heartbeat: -1s ago
resetting failing connections to retry...
retrying; attempts left: 2...
...
```

### <a name="examples"></a><span data-ttu-id="4af3c-318">Példák</span><span class="sxs-lookup"><span data-stu-id="4af3c-318">Examples</span></span>
<span data-ttu-id="4af3c-319">A következő mintakód állandó (lineáris) késleltetést állít be az újrapróbálkozások között a StackExchange.Redis ügyfél inicializálásakor.</span><span class="sxs-lookup"><span data-stu-id="4af3c-319">The following code example configures a constant (linear) delay between retries when initializing the StackExchange.Redis client.</span></span> <span data-ttu-id="4af3c-320">Ez a példa azt mutatja be, hogyan állítható be a konfiguráció egy **ConfigurationOptions**-példánnyal.</span><span class="sxs-lookup"><span data-stu-id="4af3c-320">This example shows how to set the configuration using a **ConfigurationOptions** instance.</span></span>

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using StackExchange.Redis;

namespace RetryCodeSamples
{
    class CacheRedisCodeSamples
    {
        public async static Task Samples()
        {
            var writer = new StringWriter();
            {
                try
                {
                    var retryTimeInMilliseconds = TimeSpan.FromSeconds(4).Milliseconds; // delay between retries

                    // Using object-based configuration.
                    var options = new ConfigurationOptions
                                        {
                                            EndPoints = { "localhost" },
                                            ConnectRetry = 3,
                                            ReconnectRetryPolicy = new LinearRetry(retryTimeInMilliseconds)
                                        };
                    ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);

                    // Store a reference to the multiplexer for use in the application.
                }
                catch
                {
                    Console.WriteLine(writer.ToString());
                    throw;
                }
            }
        }
    }
}
```

<span data-ttu-id="4af3c-321">A következő példa sztringként adja meg a beállításokat, és így határozza meg a konfigurációt.</span><span class="sxs-lookup"><span data-stu-id="4af3c-321">The next example sets the configuration by specifying the options as a string.</span></span> <span data-ttu-id="4af3c-322">A kapcsolat időtúllépése a leghosszabb idő, amennyit a kapcsolat a gyorsítótárra várhat, nem pedig az újrapróbálkozási kísérletek közötti időköz.</span><span class="sxs-lookup"><span data-stu-id="4af3c-322">The connection timeout is the maximum period of time to wait for a connection to the cache, not the delay between retry attempts.</span></span> <span data-ttu-id="4af3c-323">Vegye figyelembe, hogy a **ReconnectRetryPolicy** tulajdonságot csak a programkódon keresztül lehet beállítani.</span><span class="sxs-lookup"><span data-stu-id="4af3c-323">Note that the **ReconnectRetryPolicy** property can only be set by code.</span></span>

```csharp
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using StackExchange.Redis;

namespace RetryCodeSamples
{
    class CacheRedisCodeSamples
    {
        public async static Task Samples()
        {
            var writer = new StringWriter();
            {
                try
                {
                    // Using string-based configuration.
                    var options = "localhost,connectRetry=3,connectTimeout=2000";
                    ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);

                    // Store a reference to the multiplexer for use in the application.
                }
                catch
                {
                    Console.WriteLine(writer.ToString());
                    throw;
                }
            }
        }
    }
}
```

<span data-ttu-id="4af3c-324">További példákért tekintse meg a projekt webhelyének a [konfigurációval](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Configuration.md#configuration) foglalkozó szakaszát.</span><span class="sxs-lookup"><span data-stu-id="4af3c-324">For more examples, see [Configuration](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Configuration.md#configuration) on the project website.</span></span>

### <a name="more-information"></a><span data-ttu-id="4af3c-325">További információ</span><span class="sxs-lookup"><span data-stu-id="4af3c-325">More information</span></span>
* [<span data-ttu-id="4af3c-326">Redis-webhely</span><span class="sxs-lookup"><span data-stu-id="4af3c-326">Redis website</span></span>](http://redis.io/)

## <a name="azure-search"></a><span data-ttu-id="4af3c-327">Azure Search</span><span class="sxs-lookup"><span data-stu-id="4af3c-327">Azure Search</span></span>
<span data-ttu-id="4af3c-328">Az Azure Search hatékony és kifinomult keresési lehetőségekkel egészíthet ki egy webhelyet vagy alkalmazást, gyorsan és könnyen pontosítja a keresési eredményeket, továbbá részletes és finomhangolt rangsorolási modelleket képes létrehozni.</span><span class="sxs-lookup"><span data-stu-id="4af3c-328">Azure Search can be used to add powerful and sophisticated search capabilities to a website or application, quickly and easily tune search results, and construct rich and fine-tuned ranking models.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="4af3c-329">Újrapróbálkozási mechanizmus</span><span class="sxs-lookup"><span data-stu-id="4af3c-329">Retry mechanism</span></span>
<span data-ttu-id="4af3c-330">Az Azure Search SDK újrapróbálkozási viselkedését a [SearchServiceClient] és a [SearchIndexClient] osztály `SetRetryPolicy` metódusa vezérli.</span><span class="sxs-lookup"><span data-stu-id="4af3c-330">Retry behavior in the Azure Search SDK is controlled by the `SetRetryPolicy` method on the [SearchServiceClient] and [SearchIndexClient] classes.</span></span> <span data-ttu-id="4af3c-331">Az alapértelmezett szabályzat exponenciális visszatartással végzi el az újrapróbálkozást, ha az Azure Search 5xx-es vagy 408-as (Kérés időtúllépése) választ ad vissza.</span><span class="sxs-lookup"><span data-stu-id="4af3c-331">The default policy retries with exponential backoff when Azure Search returns a 5xx or 408 (Request Timeout) response.</span></span>

### <a name="telemetry"></a><span data-ttu-id="4af3c-332">Telemetria</span><span class="sxs-lookup"><span data-stu-id="4af3c-332">Telemetry</span></span>
<span data-ttu-id="4af3c-333">Nyomkövetés ETW-vel vagy egyéni nyomkövetési szolgáltató regisztrálásával.</span><span class="sxs-lookup"><span data-stu-id="4af3c-333">Trace with ETW or by registering a custom trace provider.</span></span> <span data-ttu-id="4af3c-334">További információt az [AutoRest dokumentációjában][autorest] találhat.</span><span class="sxs-lookup"><span data-stu-id="4af3c-334">For more information, see the [AutoRest documentation][autorest].</span></span>

## <a name="service-bus"></a><span data-ttu-id="4af3c-335">Service Bus</span><span class="sxs-lookup"><span data-stu-id="4af3c-335">Service Bus</span></span>
<span data-ttu-id="4af3c-336">A Service Bus egy felhőalapú üzenetkezelési platform, amely skálázható és rugalmas módon biztosít lazán kapcsolódó üzenetváltásokat az alkalmazások összetevői számára, legyen szó felhőalapú vagy helyszíni megoldásról.</span><span class="sxs-lookup"><span data-stu-id="4af3c-336">Service Bus is a cloud messaging platform that provides loosely coupled message exchange with improved scale and resiliency for components of an application, whether hosted in the cloud or on-premises.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="4af3c-337">Újrapróbálkozási mechanizmus</span><span class="sxs-lookup"><span data-stu-id="4af3c-337">Retry mechanism</span></span>
<span data-ttu-id="4af3c-338">A Service Bus a [RetryPolicy](http://msdn.microsoft.com/library/microsoft.servicebus.retrypolicy.aspx) alaposztály implementációi alapján implementálja az újrapróbálkozásokat.</span><span class="sxs-lookup"><span data-stu-id="4af3c-338">Service Bus implements retries using implementations of the [RetryPolicy](http://msdn.microsoft.com/library/microsoft.servicebus.retrypolicy.aspx) base class.</span></span> <span data-ttu-id="4af3c-339">Az összes Service Bus-ügyfél elérhetővé tesz egy **RetryPolicy** tulajdonságot, amely beállítható a **RetryPolicy** alaposztály egyik implementációjaként.</span><span class="sxs-lookup"><span data-stu-id="4af3c-339">All of the Service Bus clients expose a **RetryPolicy** property that can be set to one of the implementations of the **RetryPolicy** base class.</span></span> <span data-ttu-id="4af3c-340">A beépített implementációk a következők:</span><span class="sxs-lookup"><span data-stu-id="4af3c-340">The built-in implementations are:</span></span>

* <span data-ttu-id="4af3c-341">A [RetryExponential osztály](http://msdn.microsoft.com/library/microsoft.servicebus.retryexponential.aspx).</span><span class="sxs-lookup"><span data-stu-id="4af3c-341">The [RetryExponential Class](http://msdn.microsoft.com/library/microsoft.servicebus.retryexponential.aspx).</span></span> <span data-ttu-id="4af3c-342">Ez elérhetővé teszi azokat a tulajdonságokat, amelyek a visszatartási időközöket, az újrapróbálkozások számát és a **TerminationTimeBuffer** tulajdonságot szabályozzák, amely korlátozza, hogy a művelet hányszor hajtható végre.</span><span class="sxs-lookup"><span data-stu-id="4af3c-342">This exposes properties that control the back-off interval, the retry count, and the **TerminationTimeBuffer** property that is used to limit the total time for the operation to complete.</span></span>
* <span data-ttu-id="4af3c-343">A [NoRetry osztály](http://msdn.microsoft.com/library/microsoft.servicebus.noretry.aspx).</span><span class="sxs-lookup"><span data-stu-id="4af3c-343">The [NoRetry Class](http://msdn.microsoft.com/library/microsoft.servicebus.noretry.aspx).</span></span> <span data-ttu-id="4af3c-344">Ezt akkor szokták használni, ha nincs szükség újrapróbálkozásra a Service Bus API-szintjén, például ha az újrapróbálkozásokat egy másik folyamat kezeli egy kötegelt vagy többlépéses művelet részeként.</span><span class="sxs-lookup"><span data-stu-id="4af3c-344">This is used when retries at the Service Bus API level are not required, such as when retries are managed by another process as part of a batch or multiple step operation.</span></span>

<span data-ttu-id="4af3c-345">Service Bus-műveletek számos kivételt, visszatérhet a [Service Bus-üzenetkezelés kivételei](/azure/service-bus-messaging/service-bus-messaging-exceptions).</span><span class="sxs-lookup"><span data-stu-id="4af3c-345">Service Bus actions can return a range of exceptions, as listed in [Service Bus messaging exceptions](/azure/service-bus-messaging/service-bus-messaging-exceptions).</span></span> <span data-ttu-id="4af3c-346">A lista azt is ismerteti, hogy ezek közül melyik utal arra, hogy a művelet újrapróbálható.</span><span class="sxs-lookup"><span data-stu-id="4af3c-346">The list provides information about which if these indicate that retrying the operation is appropriate.</span></span> <span data-ttu-id="4af3c-347">Például a **ServerBusyException** azt jelzi, hogy az ügyfélnek várnia kell egy ideig, majd ismét megpróbálkozni a művelettel.</span><span class="sxs-lookup"><span data-stu-id="4af3c-347">For example, a **ServerBusyException** indicates that the client should wait for a period of time, then retry the operation.</span></span> <span data-ttu-id="4af3c-348">A **ServerBusyException** jelentkezésekor a Service Bus eltérő módba vált, amelyben további 10 másodperc adódik a kiszámított újrapróbálkozási késleltetéshez.</span><span class="sxs-lookup"><span data-stu-id="4af3c-348">The occurrence of a **ServerBusyException** also causes Service Bus to switch to a different mode, in which an extra 10-second delay is added to the computed retry delays.</span></span> <span data-ttu-id="4af3c-349">Ebből a módból rövid időn belül automatikusan kilép.</span><span class="sxs-lookup"><span data-stu-id="4af3c-349">This mode is reset after a short period.</span></span>

<span data-ttu-id="4af3c-350">A Service Bus által visszaadott kivételek elérhetővé teszik az **IsTransient** tulajdonságot, amely jelzi, hogy az ügyfélnek érdemes-e újrapróbálkoznia a művelettel.</span><span class="sxs-lookup"><span data-stu-id="4af3c-350">The exceptions returned from Service Bus expose the **IsTransient** property that indicates if the client should retry the operation.</span></span> <span data-ttu-id="4af3c-351">A beépített **RetryExponential** szabályzat a **MessagingException** osztály (az összes Service Bus-kivétel alaposztálya) **IsTransient** tulajdonságára hagyatkozik.</span><span class="sxs-lookup"><span data-stu-id="4af3c-351">The built-in **RetryExponential** policy relies on the **IsTransient** property in the **MessagingException** class, which is the base class for all Service Bus exceptions.</span></span> <span data-ttu-id="4af3c-352">A **RetryPolicy** alaposztály egyéni implementációi esetében a kivételtípus és az **IsTransient** tulajdonság közös használatával pontosabban szabályozhatja az újrapróbálkozási műveleteket.</span><span class="sxs-lookup"><span data-stu-id="4af3c-352">If you create custom implementations of the **RetryPolicy** base class you could use a combination of the exception type and the **IsTransient** property to provide more fine-grained control over retry actions.</span></span> <span data-ttu-id="4af3c-353">Például észlelhet egy **QuotaExceededException**-kivételt, és utasítást adhat, hogy csak az üzenetsor kiürítése után próbálkozzon újra az üzenetküldéssel.</span><span class="sxs-lookup"><span data-stu-id="4af3c-353">For example, you could detect a **QuotaExceededException** and take action to drain the queue before retrying sending a message to it.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="4af3c-354">Szabályzatkonfiguráció</span><span class="sxs-lookup"><span data-stu-id="4af3c-354">Policy configuration</span></span>
<span data-ttu-id="4af3c-355">Az újrapróbálkozási szabályzatok beállítása szoftveresen történik, és beállítható alapértelmezett szabályzatként a **NamespaceManager** és a **MessagingFactory** számára, vagy egyenként, az egyes üzenetkezelési ügyfelek számára.</span><span class="sxs-lookup"><span data-stu-id="4af3c-355">Retry policies are set programmatically, and can be set as a default policy for a **NamespaceManager** and for a **MessagingFactory**, or individually for each messaging client.</span></span> <span data-ttu-id="4af3c-356">Az üzenetkezelési munkamenet alapértelmezett újrapróbálkozási szabályzatát a **NamespaceManager** **RetryPolicy** tulajdonságában állíthatja be.</span><span class="sxs-lookup"><span data-stu-id="4af3c-356">To set the default retry policy for a messaging session you set the **RetryPolicy** of the **NamespaceManager**.</span></span>

```csharp
namespaceManager.Settings.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                                maxBackoff: TimeSpan.FromSeconds(30),
                                                                maxRetryCount: 3);
```

<span data-ttu-id="4af3c-357">Az üzenetkezelési előállítóból létrehozott ügyelek alapértelmezett újrapróbálkozási szabályzatát a **MessagingFactory** **RetryPolicy** tulajdonságában állíthatja be.</span><span class="sxs-lookup"><span data-stu-id="4af3c-357">To set the default retry policy for all clients created from a messaging factory, you set the **RetryPolicy** of the **MessagingFactory**.</span></span>

```csharp
messagingFactory.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                    maxBackoff: TimeSpan.FromSeconds(30),
                                                    maxRetryCount: 3);
```

<span data-ttu-id="4af3c-358">Az üzenetkezelési ügyfél alapértelmezett újrapróbálkozási szabályzatának beállításához, illetve az alapértelmezett szabályzat felülbírálásához a szükséges szabályzatosztály példányának **RetryPolicy** tulajdonságát kell megadnia:</span><span class="sxs-lookup"><span data-stu-id="4af3c-358">To set the retry policy for a messaging client, or to override its default policy, you set its **RetryPolicy** property using an instance of the required policy class:</span></span>

```csharp
client.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                            maxBackoff: TimeSpan.FromSeconds(30),
                                            maxRetryCount: 3);
```

<span data-ttu-id="4af3c-359">Az újrapróbálkozási szabályzat nem állítható be az egyes műveletek szintjén.</span><span class="sxs-lookup"><span data-stu-id="4af3c-359">The retry policy cannot be set at the individual operation level.</span></span> <span data-ttu-id="4af3c-360">Az az üzenetkezelési ügyfél összes műveletére érvényes lesz.</span><span class="sxs-lookup"><span data-stu-id="4af3c-360">It applies to all operations for the messaging client.</span></span>
<span data-ttu-id="4af3c-361">A következő táblázatban a beépített újrapróbálkozási szabályzat alapértelmezett beállításai láthatók.</span><span class="sxs-lookup"><span data-stu-id="4af3c-361">The following table shows the default settings for the built-in retry policy.</span></span>

| <span data-ttu-id="4af3c-362">Beállítás</span><span class="sxs-lookup"><span data-stu-id="4af3c-362">Setting</span></span> | <span data-ttu-id="4af3c-363">Alapértelmezett érték</span><span class="sxs-lookup"><span data-stu-id="4af3c-363">Default value</span></span> | <span data-ttu-id="4af3c-364">Jelentés</span><span class="sxs-lookup"><span data-stu-id="4af3c-364">Meaning</span></span> |
|---------|---------------|---------|
| <span data-ttu-id="4af3c-365">Szabályzat</span><span class="sxs-lookup"><span data-stu-id="4af3c-365">Policy</span></span> | <span data-ttu-id="4af3c-366">Exponenciális</span><span class="sxs-lookup"><span data-stu-id="4af3c-366">Exponential</span></span> | <span data-ttu-id="4af3c-367">Exponenciális visszatartás.</span><span class="sxs-lookup"><span data-stu-id="4af3c-367">Exponential back-off.</span></span> |
| <span data-ttu-id="4af3c-368">MinimalBackoff</span><span class="sxs-lookup"><span data-stu-id="4af3c-368">MinimalBackoff</span></span> | <span data-ttu-id="4af3c-369">0</span><span class="sxs-lookup"><span data-stu-id="4af3c-369">0</span></span> | <span data-ttu-id="4af3c-370">Minimális visszatartási időköz.</span><span class="sxs-lookup"><span data-stu-id="4af3c-370">Minimum back-off interval.</span></span> <span data-ttu-id="4af3c-371">Ez hozzáadódik a deltaBackoff alapján kiszámított újrapróbálkozási időközhöz.</span><span class="sxs-lookup"><span data-stu-id="4af3c-371">This is added to the retry interval computed from deltaBackoff.</span></span> |
| <span data-ttu-id="4af3c-372">MaximumBackoff</span><span class="sxs-lookup"><span data-stu-id="4af3c-372">MaximumBackoff</span></span> | <span data-ttu-id="4af3c-373">30 másodperc</span><span class="sxs-lookup"><span data-stu-id="4af3c-373">30 seconds</span></span> | <span data-ttu-id="4af3c-374">Maximális visszatartási időköz.</span><span class="sxs-lookup"><span data-stu-id="4af3c-374">Maximum back-off interval.</span></span> <span data-ttu-id="4af3c-375">A MaximumBackoff akkor lép működésbe, ha a kiszámított újrapróbálkozási időköz nagyobb, mint a MaxBackoff értéke.</span><span class="sxs-lookup"><span data-stu-id="4af3c-375">MaximumBackoff is used if the computed retry interval is greater than MaxBackoff.</span></span> |
| <span data-ttu-id="4af3c-376">DeltaBackoff</span><span class="sxs-lookup"><span data-stu-id="4af3c-376">DeltaBackoff</span></span> | <span data-ttu-id="4af3c-377">3 másodperc</span><span class="sxs-lookup"><span data-stu-id="4af3c-377">3 seconds</span></span> | <span data-ttu-id="4af3c-378">Az újrapróbálkozások közötti visszatartási időköz.</span><span class="sxs-lookup"><span data-stu-id="4af3c-378">Back-off interval between retries.</span></span> <span data-ttu-id="4af3c-379">Az ezt követő újrapróbálkozási kísérletek esetében az időtartomány többszörösét fogja használni a rendszer.</span><span class="sxs-lookup"><span data-stu-id="4af3c-379">Multiples of this timespan will be used for subsequent retry attempts.</span></span> |
| <span data-ttu-id="4af3c-380">TimeBuffer</span><span class="sxs-lookup"><span data-stu-id="4af3c-380">TimeBuffer</span></span> | <span data-ttu-id="4af3c-381">5 másodperc</span><span class="sxs-lookup"><span data-stu-id="4af3c-381">5 seconds</span></span> | <span data-ttu-id="4af3c-382">Az újrapróbálkozáshoz tartozó leállítási időpuffer.</span><span class="sxs-lookup"><span data-stu-id="4af3c-382">The termination time buffer associated with the retry.</span></span> <span data-ttu-id="4af3c-383">A rendszer felhagy az újrapróbálkozással, ha a TimeBuffer értékben megadottnál kevesebb idő van hátra.</span><span class="sxs-lookup"><span data-stu-id="4af3c-383">Retry attempts will be abandoned if the remaining time is less than TimeBuffer.</span></span> |
| <span data-ttu-id="4af3c-384">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="4af3c-384">MaxRetryCount</span></span> | <span data-ttu-id="4af3c-385">10</span><span class="sxs-lookup"><span data-stu-id="4af3c-385">10</span></span> | <span data-ttu-id="4af3c-386">Az újrapróbálkozások maximális száma.</span><span class="sxs-lookup"><span data-stu-id="4af3c-386">The maximum number of retries.</span></span> |
| <span data-ttu-id="4af3c-387">ServerBusyBaseSleepTime</span><span class="sxs-lookup"><span data-stu-id="4af3c-387">ServerBusyBaseSleepTime</span></span> | <span data-ttu-id="4af3c-388">10 másodperc</span><span class="sxs-lookup"><span data-stu-id="4af3c-388">10 seconds</span></span> | <span data-ttu-id="4af3c-389">Ha az utolsó kivétel a **ServerBusyException** volt, ez az érték hozzáadódik a kiszámított újrapróbálkozási időközhöz.</span><span class="sxs-lookup"><span data-stu-id="4af3c-389">If the last exception encountered was **ServerBusyException**, this value will be added to the computed retry interval.</span></span> <span data-ttu-id="4af3c-390">Ez az érték nem módosítható.</span><span class="sxs-lookup"><span data-stu-id="4af3c-390">This value cannot be changed.</span></span> |

### <a name="retry-usage-guidance"></a><span data-ttu-id="4af3c-391">Újrapróbálkozásokra vonatkozó útmutató</span><span class="sxs-lookup"><span data-stu-id="4af3c-391">Retry usage guidance</span></span>
<span data-ttu-id="4af3c-392">Ügyeljen a következőkre a Service Bus használata során:</span><span class="sxs-lookup"><span data-stu-id="4af3c-392">Consider the following guidelines when using Service Bus:</span></span>

* <span data-ttu-id="4af3c-393">A beépített **RetryExponential** implementáció használatakor nincs szükség tartalékműveletek megvalósítására, mivel a szabályzat a „foglalt kiszolgáló” kivételekre reagálva automatikusan átvált a megfelelő újrapróbálkozási módra.</span><span class="sxs-lookup"><span data-stu-id="4af3c-393">When using the built-in **RetryExponential** implementation, do not implement a fallback operation as the policy reacts to Server Busy exceptions and automatically switches to an appropriate retry mode.</span></span>
* <span data-ttu-id="4af3c-394">A Service Bus támogatja a Párosított névterek nevű funkciót, amely automatikus feladatátvételt implementál, és az elsődleges névtér üzenetsorának hibájakor egy másik névtér tartalék üzenetsorára vált.</span><span class="sxs-lookup"><span data-stu-id="4af3c-394">Service Bus supports a feature called Paired Namespaces, which implements automatic failover to a backup queue in a separate namespace if the queue in the primary namespace fails.</span></span> <span data-ttu-id="4af3c-395">A másodlagos üzenetsor üzenetei továbbküldhetők az elsődleges üzenetsornak, miután az helyreállt.</span><span class="sxs-lookup"><span data-stu-id="4af3c-395">Messages from the secondary queue can be sent back to the primary queue when it recovers.</span></span> <span data-ttu-id="4af3c-396">Ez a funkció az átmeneti hibák kezelésére szolgál.</span><span class="sxs-lookup"><span data-stu-id="4af3c-396">This feature helps to address transient failures.</span></span> <span data-ttu-id="4af3c-397">További információért lásd [az aszinkron üzenetkezelési minták és a magas rendelkezésre állás](http://msdn.microsoft.com/library/azure/dn292562.aspx) ismertetését.</span><span class="sxs-lookup"><span data-stu-id="4af3c-397">For more information, see [Asynchronous Messaging Patterns and High Availability](http://msdn.microsoft.com/library/azure/dn292562.aspx).</span></span>

<span data-ttu-id="4af3c-398">A következő beállításokat javasoljuk az újrapróbálkozási műveletekhez.</span><span class="sxs-lookup"><span data-stu-id="4af3c-398">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="4af3c-399">Ezek általános célú beállítások, ezért javasoljuk, hogy monitorozza műveleteit, és finomhangolja az értékeket saját igényei szerint.</span><span class="sxs-lookup"><span data-stu-id="4af3c-399">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="4af3c-400">Környezet</span><span class="sxs-lookup"><span data-stu-id="4af3c-400">Context</span></span> | <span data-ttu-id="4af3c-401">Példa a maximális késésre</span><span class="sxs-lookup"><span data-stu-id="4af3c-401">Example maximum latency</span></span> | <span data-ttu-id="4af3c-402">Újrapróbálkozási szabályzat</span><span class="sxs-lookup"><span data-stu-id="4af3c-402">Retry policy</span></span> | <span data-ttu-id="4af3c-403">Beállítások</span><span class="sxs-lookup"><span data-stu-id="4af3c-403">Settings</span></span> | <span data-ttu-id="4af3c-404">Működés</span><span class="sxs-lookup"><span data-stu-id="4af3c-404">How it works</span></span> |
|---------|---------|---------|---------|---------|
| <span data-ttu-id="4af3c-405">Interaktív, felhasználói felület vagy előtér</span><span class="sxs-lookup"><span data-stu-id="4af3c-405">Interactive, UI, or foreground</span></span> | <span data-ttu-id="4af3c-406">2 másodperc\*</span><span class="sxs-lookup"><span data-stu-id="4af3c-406">2 seconds\*</span></span>  | <span data-ttu-id="4af3c-407">Exponenciális</span><span class="sxs-lookup"><span data-stu-id="4af3c-407">Exponential</span></span> | <span data-ttu-id="4af3c-408">MinimumBackoff = 0</span><span class="sxs-lookup"><span data-stu-id="4af3c-408">MinimumBackoff = 0</span></span> <br/> <span data-ttu-id="4af3c-409">MaximumBackoff = 30 mp.</span><span class="sxs-lookup"><span data-stu-id="4af3c-409">MaximumBackoff = 30 sec.</span></span> <br/> <span data-ttu-id="4af3c-410">DeltaBackoff = 300 ms</span><span class="sxs-lookup"><span data-stu-id="4af3c-410">DeltaBackoff = 300 msec.</span></span> <br/> <span data-ttu-id="4af3c-411">TimeBuffer = 300 ms</span><span class="sxs-lookup"><span data-stu-id="4af3c-411">TimeBuffer = 300 msec.</span></span> <br/> <span data-ttu-id="4af3c-412">MaxRetryCount = 2</span><span class="sxs-lookup"><span data-stu-id="4af3c-412">MaxRetryCount = 2</span></span> | <span data-ttu-id="4af3c-413">1. kísérlet: 0 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="4af3c-413">Attempt 1: Delay 0 sec.</span></span> <br/> <span data-ttu-id="4af3c-414">2. kísérlet: kb. 300 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="4af3c-414">Attempt 2: Delay ~300 msec.</span></span> <br/> <span data-ttu-id="4af3c-415">3. kísérlet: kb. 900 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="4af3c-415">Attempt 3: Delay ~900 msec.</span></span> |
| <span data-ttu-id="4af3c-416">Háttér vagy kötegelt</span><span class="sxs-lookup"><span data-stu-id="4af3c-416">Background or batch</span></span> | <span data-ttu-id="4af3c-417">30 másodperc</span><span class="sxs-lookup"><span data-stu-id="4af3c-417">30 seconds</span></span> | <span data-ttu-id="4af3c-418">Exponenciális</span><span class="sxs-lookup"><span data-stu-id="4af3c-418">Exponential</span></span> | <span data-ttu-id="4af3c-419">MinimumBackoff = 1</span><span class="sxs-lookup"><span data-stu-id="4af3c-419">MinimumBackoff = 1</span></span> <br/> <span data-ttu-id="4af3c-420">MaximumBackoff = 30 mp.</span><span class="sxs-lookup"><span data-stu-id="4af3c-420">MaximumBackoff = 30 sec.</span></span> <br/> <span data-ttu-id="4af3c-421">DeltaBackoff = 1,75 mp.</span><span class="sxs-lookup"><span data-stu-id="4af3c-421">DeltaBackoff = 1.75 sec.</span></span> <br/> <span data-ttu-id="4af3c-422">TimeBuffer = 5 mp.</span><span class="sxs-lookup"><span data-stu-id="4af3c-422">TimeBuffer = 5 sec.</span></span> <br/> <span data-ttu-id="4af3c-423">MaxRetryCount = 3</span><span class="sxs-lookup"><span data-stu-id="4af3c-423">MaxRetryCount = 3</span></span> | <span data-ttu-id="4af3c-424">1. kísérlet: kb. 1 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="4af3c-424">Attempt 1: Delay ~1 sec.</span></span> <br/> <span data-ttu-id="4af3c-425">2. kísérlet: kb. 3 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="4af3c-425">Attempt 2: Delay ~3 sec.</span></span> <br/> <span data-ttu-id="4af3c-426">3. kísérlet: kb. 6 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="4af3c-426">Attempt 3: Delay ~6 msec.</span></span> <br/> <span data-ttu-id="4af3c-427">4. kísérlet: kb. 13 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="4af3c-427">Attempt 4: Delay ~13 msec.</span></span> |

<span data-ttu-id="4af3c-428">\* Nem tartalmazza a további késleltetést, amely a „foglalt kiszolgáló” válasz esetén adódik az értékhez.</span><span class="sxs-lookup"><span data-stu-id="4af3c-428">\* Not including additional delay that is added if a Server Busy response is received.</span></span>

### <a name="telemetry"></a><span data-ttu-id="4af3c-429">Telemetria</span><span class="sxs-lookup"><span data-stu-id="4af3c-429">Telemetry</span></span>
<span data-ttu-id="4af3c-430">A Service Bus ETW-eseményként naplózza az újrapróbálkozásokat egy **EventSource** használatával.</span><span class="sxs-lookup"><span data-stu-id="4af3c-430">Service Bus logs retries as ETW events using an **EventSource**.</span></span> <span data-ttu-id="4af3c-431">Egy **EventListener** az eseményforráshoz csatolása szükséges, ha rögzíteni kívánja az eseményeket, és meg kívánja tekinteni azokat a Teljesítménynaplóban, vagy ha egy megfelelő célnaplóba írná azokat.</span><span class="sxs-lookup"><span data-stu-id="4af3c-431">You must attach an **EventListener** to the event source to capture the events and view them in Performance Viewer, or write them to a suitable destination log.</span></span> <span data-ttu-id="4af3c-432">Az újrapróbálkozási események formátuma a következő:</span><span class="sxs-lookup"><span data-stu-id="4af3c-432">The retry events are of the following form:</span></span>

```text
Microsoft-ServiceBus-Client/RetryPolicyIteration
ThreadID="14,500"
FormattedMessage="[TrackingId:] RetryExponential: Operation Get:https://retry-tests.servicebus.windows.net/TestQueue/?api-version=2014-05 at iteration 0 is retrying after 00:00:00.1000000 sleep because of Microsoft.ServiceBus.Messaging.MessagingCommunicationException: The remote name could not be resolved: 'retry-tests.servicebus.windows.net'.TrackingId:6a26f99c-dc6d-422e-8565-f89fdd0d4fe3, TimeStamp:9/5/2014 10:00:13 PM."
trackingId=""
policyType="RetryExponential"
operation="Get:https://retry-tests.servicebus.windows.net/TestQueue/?api-version=2014-05"
iteration="0"
iterationSleep="00:00:00.1000000"
lastExceptionType="Microsoft.ServiceBus.Messaging.MessagingCommunicationException"
exceptionMessage="The remote name could not be resolved: 'retry-tests.servicebus.windows.net'.TrackingId:6a26f99c-dc6d-422e-8565-f89fdd0d4fe3,TimeStamp:9/5/2014 10:00:13 PM"
```

### <a name="examples"></a><span data-ttu-id="4af3c-433">Példák</span><span class="sxs-lookup"><span data-stu-id="4af3c-433">Examples</span></span>
<span data-ttu-id="4af3c-434">A következő mintakód bemutatja, hogyan állíthatja be az újrapróbálkozási szabályzatot a következőkhöz:</span><span class="sxs-lookup"><span data-stu-id="4af3c-434">The following code example shows how to set the retry policy for:</span></span>

* <span data-ttu-id="4af3c-435">Egy névtérkezelő.</span><span class="sxs-lookup"><span data-stu-id="4af3c-435">A namespace manager.</span></span> <span data-ttu-id="4af3c-436">A szabályzat a kezelő összes műveletére vonatkozik, és nem bírálható felül az egyes műveletek esetében.</span><span class="sxs-lookup"><span data-stu-id="4af3c-436">The policy applies to all operations on that manager, and cannot be overridden for individual operations.</span></span>
* <span data-ttu-id="4af3c-437">Egy üzenetkezelési előállító.</span><span class="sxs-lookup"><span data-stu-id="4af3c-437">A messaging factory.</span></span> <span data-ttu-id="4af3c-438">A szabályzat az előállítóból létrehozott összes ügyfélre vonatkozik, és nem bírálható felül az egyes ügyfelek létrehozásakor.</span><span class="sxs-lookup"><span data-stu-id="4af3c-438">The policy applies to all clients created from that factory, and cannot be overridden when creating individual clients.</span></span>
* <span data-ttu-id="4af3c-439">Egy különálló üzenetkezelési ügyfél.</span><span class="sxs-lookup"><span data-stu-id="4af3c-439">An individual messaging client.</span></span> <span data-ttu-id="4af3c-440">Az ügyfél létrehozása után beállíthatja annak újrapróbálkozási szabályzatát.</span><span class="sxs-lookup"><span data-stu-id="4af3c-440">After a client has been created, you can set the retry policy for that client.</span></span> <span data-ttu-id="4af3c-441">A szabályzat az ügyfél összes műveletére vonatkozik.</span><span class="sxs-lookup"><span data-stu-id="4af3c-441">The policy applies to all operations on that client.</span></span>

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.ServiceBus;
using Microsoft.ServiceBus.Messaging;

namespace RetryCodeSamples
{
    class ServiceBusCodeSamples
    {
        private const string connectionString =
            @"Endpoint=sb://[my-namespace].servicebus.windows.net/;
                SharedAccessKeyName=RootManageSharedAccessKey;
                SharedAccessKey=C99..........Mk=";

        public async static Task Samples()
        {
            const string QueueName = "TestQueue";

            ServiceBusEnvironment.SystemConnectivity.Mode = ConnectivityMode.Http;

            var namespaceManager = NamespaceManager.CreateFromConnectionString(connectionString);

            // The namespace manager will have a default exponential policy with 10 retry attempts
            // and a 3 second delay delta.
            // Retry delays will be approximately 0 sec, 3 sec, 9 sec, 25 sec and the fixed 30 sec,
            // with an extra 10 sec added when receiving a ServiceBusyException.

            {
                // Set different values for the retry policy, used for all operations on the namespace manager.
                namespaceManager.Settings.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(0),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);

                // Policies cannot be specified on a per-operation basis.
                if (!await namespaceManager.QueueExistsAsync(QueueName))
                {
                    await namespaceManager.CreateQueueAsync(QueueName);
                }
            }

            var messagingFactory = MessagingFactory.Create(
                namespaceManager.Address, namespaceManager.Settings.TokenProvider);
            // The messaging factory will have a default exponential policy with 10 retry attempts
            // and a 3 second delay delta.
            // Retry delays will be approximately 0 sec, 3 sec, 9 sec, 25 sec and the fixed 30 sec,
            // with an extra 10 sec added when receiving a ServiceBusyException.

            {
                // Set different values for the retry policy, used for clients created from it.
                messagingFactory.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(1),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);


                // Policies cannot be specified on a per-operation basis.
                var session = await messagingFactory.AcceptMessageSessionAsync();
            }

            {
                var client = messagingFactory.CreateQueueClient(QueueName);
                // The client inherits the policy from the factory that created it.


                // Set different values for the retry policy on the client.
                client.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(0.1),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);

                // Policies cannot be specified on a per-operation basis.
                var session = await client.AcceptMessageSessionAsync();
            }
        }
    }
}
```

### <a name="more-information"></a><span data-ttu-id="4af3c-442">További információ</span><span class="sxs-lookup"><span data-stu-id="4af3c-442">More information</span></span>
* [<span data-ttu-id="4af3c-443">Aszinkron üzenetkezelési minták és magas rendelkezésre állás</span><span class="sxs-lookup"><span data-stu-id="4af3c-443">Asynchronous Messaging Patterns and High Availability</span></span>](http://msdn.microsoft.com/library/azure/dn292562.aspx)

## <a name="service-fabric"></a><span data-ttu-id="4af3c-444">Service Fabric</span><span class="sxs-lookup"><span data-stu-id="4af3c-444">Service Fabric</span></span>

<span data-ttu-id="4af3c-445">A megbízható szolgáltatások Service Fabric-fürtön belüli elosztásával a cikkben tárgyalt legtöbb átmeneti hiba elkerülhető.</span><span class="sxs-lookup"><span data-stu-id="4af3c-445">Distributing reliable services in a Service Fabric cluster guards against most of the potential transient faults discussed in this article.</span></span> <span data-ttu-id="4af3c-446">Azonban így is előfordulhatnak átmeneti hibák.</span><span class="sxs-lookup"><span data-stu-id="4af3c-446">Some transient faults are still possible, however.</span></span> <span data-ttu-id="4af3c-447">Előfordulhat például, hogy a kérés érkezésekor az elnevezési szolgáltatás egy útválasztási változtatást végez, ami kivételt eredményez.</span><span class="sxs-lookup"><span data-stu-id="4af3c-447">For example, the naming service might be in the middle of a routing change when it gets a request, causing it to throw an exception.</span></span> <span data-ttu-id="4af3c-448">Ugyanez a kérés 100 milliszekundummal később talán sikeres lett volna.</span><span class="sxs-lookup"><span data-stu-id="4af3c-448">If the same request comes 100 milliseconds later, it will probably succeed.</span></span>

<span data-ttu-id="4af3c-449">A Service Fabric az ehhez hasonló átmeneti hibák belső kezelését elvégzi.</span><span class="sxs-lookup"><span data-stu-id="4af3c-449">Internally, Service Fabric manages this kind of transient fault.</span></span> <span data-ttu-id="4af3c-450">A szolgáltatás beállítása közben konfigurálhat egyes beállításokat az `OperationRetrySettings` osztállyal.</span><span class="sxs-lookup"><span data-stu-id="4af3c-450">You can configure some settings by using the `OperationRetrySettings` class while setting up your services.</span></span>  <span data-ttu-id="4af3c-451">Az alábbi kód erre mutat egy példát.</span><span class="sxs-lookup"><span data-stu-id="4af3c-451">The following code shows an example.</span></span> <span data-ttu-id="4af3c-452">A legtöbb esetben ez nem szükséges, és az alapértelmezett beállítások tökéletesen megfelelnek.</span><span class="sxs-lookup"><span data-stu-id="4af3c-452">In most cases, this should not be necessary, and the default settings will be fine.</span></span>

```csharp
FabricTransportRemotingSettings transportSettings = new FabricTransportRemotingSettings
{
    OperationTimeout = TimeSpan.FromSeconds(30)
};

var retrySettings = new OperationRetrySettings(TimeSpan.FromSeconds(15), TimeSpan.FromSeconds(1), 5);

var clientFactory = new FabricTransportServiceRemotingClientFactory(transportSettings);

var serviceProxyFactory = new ServiceProxyFactory((c) => clientFactory, retrySettings);

var client = serviceProxyFactory.CreateServiceProxy<ISomeService>(
    new Uri("fabric:/SomeApp/SomeStatefulReliableService"),
    new ServicePartitionKey(0));
```

### <a name="more-information"></a><span data-ttu-id="4af3c-453">További információ</span><span class="sxs-lookup"><span data-stu-id="4af3c-453">More information</span></span>

* [<span data-ttu-id="4af3c-454">Távoli kivételkezelés</span><span class="sxs-lookup"><span data-stu-id="4af3c-454">Remote Exception Handling</span></span>](https://github.com/Microsoft/azure-docs/blob/master/articles/service-fabric/service-fabric-reliable-services-communication-remoting.md#remoting-exception-handling)

## <a name="sql-database-using-adonet"></a><span data-ttu-id="4af3c-455">SQL-adatbázishoz az ADO.NET használatával</span><span class="sxs-lookup"><span data-stu-id="4af3c-455">SQL Database using ADO.NET</span></span>
<span data-ttu-id="4af3c-456">Az SQL Database egy üzemeltetett SQL-adatbázis, amely különböző méretekben, normál (megosztott) és prémium (nem megosztott) szolgáltatásként is elérhető.</span><span class="sxs-lookup"><span data-stu-id="4af3c-456">SQL Database is a hosted SQL database available in a range of sizes and as both a standard (shared) and premium (non-shared) service.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="4af3c-457">Újrapróbálkozási mechanizmus</span><span class="sxs-lookup"><span data-stu-id="4af3c-457">Retry mechanism</span></span>
<span data-ttu-id="4af3c-458">Az SQL Database nem tartalmaz beépített támogatást az újrapróbálkozásokhoz, ha az ADO.NET használatával érik el.</span><span class="sxs-lookup"><span data-stu-id="4af3c-458">SQL Database has no built-in support for retries when accessed using ADO.NET.</span></span> <span data-ttu-id="4af3c-459">Ugyanakkor a kérések válaszkódjából megállapítható, hogy a kérés miért hiúsult meg.</span><span class="sxs-lookup"><span data-stu-id="4af3c-459">However, the return codes from requests can be used to determine why a request failed.</span></span> <span data-ttu-id="4af3c-460">További információt az SQL Database szabályozásáról [az Azure SQL Database erőforrás-korlátait](/azure/sql-database/sql-database-resource-limits) ismertető szakaszban talál.</span><span class="sxs-lookup"><span data-stu-id="4af3c-460">For more information about SQL Database throttling, see [Azure SQL Database resource limits](/azure/sql-database/sql-database-resource-limits).</span></span> <span data-ttu-id="4af3c-461">A kapcsolódó hibakódok listáját [az SQL Database ügyfélalkalmazásaiban felmerülő SQL-hibakódokat](/azure/sql-database/sql-database-develop-error-messages) ismertető szakaszban találja.</span><span class="sxs-lookup"><span data-stu-id="4af3c-461">For a list of relevant error codes, see [SQL error codes for SQL Database client applications](/azure/sql-database/sql-database-develop-error-messages).</span></span>

<span data-ttu-id="4af3c-462">A Polly kódtárt alkalmazva implementálhatja az újrapróbálkozást az SQL Database-ben.</span><span class="sxs-lookup"><span data-stu-id="4af3c-462">You can use the Polly library to implement retries for SQL Database.</span></span> <span data-ttu-id="4af3c-463">További információt az [átmeneti hibák a Polly használatával történő kezelését](#transient-fault-handling-with-polly) ismertető szakaszban talál.</span><span class="sxs-lookup"><span data-stu-id="4af3c-463">See [Transient fault handling with Polly](#transient-fault-handling-with-polly).</span></span>

### <a name="retry-usage-guidance"></a><span data-ttu-id="4af3c-464">Újrapróbálkozásokra vonatkozó útmutató</span><span class="sxs-lookup"><span data-stu-id="4af3c-464">Retry usage guidance</span></span>
<span data-ttu-id="4af3c-465">Ügyeljen a következőkre, amikor az ADO.NET használatával éri el az SQL Database-t:</span><span class="sxs-lookup"><span data-stu-id="4af3c-465">Consider the following guidelines when accessing SQL Database using ADO.NET:</span></span>

* <span data-ttu-id="4af3c-466">Válassza a megfelelő szolgáltatástípust (megosztott vagy prémium).</span><span class="sxs-lookup"><span data-stu-id="4af3c-466">Choose the appropriate service option (shared or premium).</span></span> <span data-ttu-id="4af3c-467">A megosztott példány esetében a szokottnál hosszabb csatlakozási késések fordulhatnak elő, valamint a kérések számának korlátozására lehet szükség, mivel a megosztott kiszolgálót más bérlők is használják.</span><span class="sxs-lookup"><span data-stu-id="4af3c-467">A shared instance may suffer longer than usual connection delays and throttling due to the usage by other tenants of the shared server.</span></span> <span data-ttu-id="4af3c-468">Ha kiszámíthatóbb teljesítményre és megbízhatóan alacsony késésű műveletekre van szükség, mindenképpen a prémium szolgáltatást érdemes választani.</span><span class="sxs-lookup"><span data-stu-id="4af3c-468">If more predictable performance and reliable low latency operations are required, consider choosing the premium option.</span></span>
* <span data-ttu-id="4af3c-469">Gondoskodjon arról, hogy az újrapróbálkozás a megfelelő szinten vagy hatókörrel legyen végrehajtva, amivel elkerülheti, hogy a nem idempotens műveletek miatt inkonzisztencia keletkezzen az adatokban.</span><span class="sxs-lookup"><span data-stu-id="4af3c-469">Ensure that you perform retries at the appropriate level or scope to avoid non-idempotent operations causing inconsistency in the data.</span></span> <span data-ttu-id="4af3c-470">Ideális esetben minden műveletnek idempotensnek kellene lennie, hogy inkonzisztencia veszélye nélkül lehessen ismételni azokat.</span><span class="sxs-lookup"><span data-stu-id="4af3c-470">Ideally, all operations should be idempotent so that they can be repeated without causing inconsistency.</span></span> <span data-ttu-id="4af3c-471">Ellenkező esetben az újrapróbálkozást olyan szinten vagy hatókörrel kell végrehajtani, hogy az összes kapcsolódó változtatást vissza lehessen vonni egy művelet meghiúsulásakor – például egy tranzakciós hatókörben.</span><span class="sxs-lookup"><span data-stu-id="4af3c-471">Where this is not the case, the retry should be performed at a level or scope that allows all related changes to be undone if one operation fails; for example, from within a transactional scope.</span></span> <span data-ttu-id="4af3c-472">További információt a [Cloud Service Fundamentals adatelérési réteg átmeneti hibák kezelésével](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx#Idempotent_Guarantee) foglalkozó részében talál.</span><span class="sxs-lookup"><span data-stu-id="4af3c-472">For more information, see [Cloud Service Fundamentals Data Access Layer – Transient Fault Handling](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx#Idempotent_Guarantee).</span></span>
* <span data-ttu-id="4af3c-473">Az Azure SQL Database esetében nem javasoljuk rögzített időközű stratégiák alkalmazását. Az interaktív forgatókönyvek kivételt képeznek, mivel csak néhány újrapróbálkozás történik nagyon rövid időközökkel.</span><span class="sxs-lookup"><span data-stu-id="4af3c-473">A fixed interval strategy is not recommended for use with Azure SQL Database except for interactive scenarios where there are only a few retries at very short intervals.</span></span> <span data-ttu-id="4af3c-474">Ehelyett a legtöbb esetben exponenciális visszatartási stratégia használata javasolt.</span><span class="sxs-lookup"><span data-stu-id="4af3c-474">Instead, consider using an exponential back-off strategy for the majority of scenarios.</span></span>
* <span data-ttu-id="4af3c-475">A kapcsolatok definiálásakor válasszon megfelelő értéket a kapcsolatok és a parancsok időtúllépéséhez.</span><span class="sxs-lookup"><span data-stu-id="4af3c-475">Choose a suitable value for the connection and command timeouts when defining connections.</span></span> <span data-ttu-id="4af3c-476">A túl alacsony időtúllépési érték miatt a kapcsolatok idő előtt szakadhatnak meg, ha az adatbázis leterhelt.</span><span class="sxs-lookup"><span data-stu-id="4af3c-476">Too short a timeout may result in premature failures of connections when the database is busy.</span></span> <span data-ttu-id="4af3c-477">A túl magas időtúllépési érték akadályozhatja az újrapróbálkozási logika megfelelő működését, mivel túl sokáig fog várni, mielőtt észlelné a sikertelen csatlakozást.</span><span class="sxs-lookup"><span data-stu-id="4af3c-477">Too long a timeout may prevent the retry logic working correctly by waiting too long before detecting a failed connection.</span></span> <span data-ttu-id="4af3c-478">Az időtúllépés értéke a végpontok közötti késés egyik összetevője. Gyakorlatilag hozzáadódik az újrapróbálkozási szabályzatban megadott újrapróbálkozási késleltetéshez minden egyes újrapróbálkozási kísérlet esetén.</span><span class="sxs-lookup"><span data-stu-id="4af3c-478">The value of the timeout is a component of the end-to-end latency; it is effectively added to the retry delay specified in the retry policy for every retry attempt.</span></span>
* <span data-ttu-id="4af3c-479">Megszakítja a kapcsolatot adott számú újrapróbálkozás után, akár egy exponenciális visszatartási újrapróbálkozási logika használata esetén is, és egy új kapcsolatot létesítve próbálja meg újra végrehajtani a műveletet.</span><span class="sxs-lookup"><span data-stu-id="4af3c-479">Close the connection after a certain number of retries, even when using an exponential back off retry logic, and retry the operation on a new connection.</span></span> <span data-ttu-id="4af3c-480">Ha ugyanazzal a művelettel többször próbálkozik újra ugyanazon a kapcsolaton keresztül, az önmagában is csatlakozási problémákat okozhat.</span><span class="sxs-lookup"><span data-stu-id="4af3c-480">Retrying the same operation multiple times on the same connection can be a factor that contributes to connection problems.</span></span> <span data-ttu-id="4af3c-481">Erre a technikára egy példát a [Cloud Service Fundamentals adatelérési réteg átmeneti hibák kezelésével](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx) foglalkozó részében találhat.</span><span class="sxs-lookup"><span data-stu-id="4af3c-481">For an example of this technique, see [Cloud Service Fundamentals Data Access Layer – Transient Fault Handling](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx).</span></span>
* <span data-ttu-id="4af3c-482">Ha kapcsolatkészletezést alkalmaz (ez az alapértelmezett beállítás), akkor előfordulhat, hogy a készletből ismét ugyanazt a kapcsolatot választja a rendszer, akár a kapcsolat lezárása és ismételt megnyitása után is.</span><span class="sxs-lookup"><span data-stu-id="4af3c-482">When connection pooling is in use (the default) there is a chance that the same connection will be chosen from the pool, even after closing and reopening a connection.</span></span> <span data-ttu-id="4af3c-483">Ebben az esetben az **SqlConnection** osztályból kell meghívni a **ClearPool** metódust, és jelezni, hogy a kapcsolat nem újrahasználható.</span><span class="sxs-lookup"><span data-stu-id="4af3c-483">If this is the case, a technique to resolve it is to call the **ClearPool** method of the **SqlConnection** class to mark the connection as not reusable.</span></span> <span data-ttu-id="4af3c-484">Ezt azonban csak abban az esetben javasoljuk, ha számos csatlakozási kísérlet meghiúsult, és csak akkor, ha az átmeneti hibák, például SQL-időtúllépések (-2-es hibakód) adott osztálya hibás kapcsolatokhoz kötődik.</span><span class="sxs-lookup"><span data-stu-id="4af3c-484">However, you should do this only after several connection attempts have failed, and only when encountering the specific class of transient failures such as SQL timeouts (error code -2) related to faulty connections.</span></span>
* <span data-ttu-id="4af3c-485">Amennyiben az adatelérési kód **TransactionScope**-példányként kezdeményezett tranzakciókat alkalmaz, az újrapróbálkozási logikának újra meg kell nyitnia a kapcsolatot, és új tranzakció-hatókört kell kezdeményeznie.</span><span class="sxs-lookup"><span data-stu-id="4af3c-485">If the data access code uses transactions initiated as **TransactionScope** instances, the retry logic should reopen the connection and initiate a new transaction scope.</span></span> <span data-ttu-id="4af3c-486">Ezért az újrapróbálható kódblokknak a tranzakció teljes hatókörét le kell fedne.</span><span class="sxs-lookup"><span data-stu-id="4af3c-486">For this reason, the retryable code block should encompass the entire scope of the transaction.</span></span>

<span data-ttu-id="4af3c-487">A következő beállításokat javasoljuk az újrapróbálkozási műveletekhez.</span><span class="sxs-lookup"><span data-stu-id="4af3c-487">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="4af3c-488">Ezek általános célú beállítások, ezért javasoljuk, hogy monitorozza műveleteit, és finomhangolja az értékeket saját igényei szerint.</span><span class="sxs-lookup"><span data-stu-id="4af3c-488">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="4af3c-489">**Környezet**</span><span class="sxs-lookup"><span data-stu-id="4af3c-489">**Context**</span></span> | <span data-ttu-id="4af3c-490">**Mintacél E2E<br />max. késleltetése**</span><span class="sxs-lookup"><span data-stu-id="4af3c-490">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="4af3c-491">**Újrapróbálkozási stratégia**</span><span class="sxs-lookup"><span data-stu-id="4af3c-491">**Retry strategy**</span></span> | <span data-ttu-id="4af3c-492">**Beállítások**</span><span class="sxs-lookup"><span data-stu-id="4af3c-492">**Settings**</span></span> | <span data-ttu-id="4af3c-493">**Értékek**</span><span class="sxs-lookup"><span data-stu-id="4af3c-493">**Values**</span></span> | <span data-ttu-id="4af3c-494">**Működési elv**</span><span class="sxs-lookup"><span data-stu-id="4af3c-494">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="4af3c-495">Interaktív, felhasználói felület</span><span class="sxs-lookup"><span data-stu-id="4af3c-495">Interactive, UI,</span></span><br /><span data-ttu-id="4af3c-496">vagy előtér</span><span class="sxs-lookup"><span data-stu-id="4af3c-496">or foreground</span></span> |<span data-ttu-id="4af3c-497">2 másodperc</span><span class="sxs-lookup"><span data-stu-id="4af3c-497">2 sec</span></span> |<span data-ttu-id="4af3c-498">FixedInterval</span><span class="sxs-lookup"><span data-stu-id="4af3c-498">FixedInterval</span></span> |<span data-ttu-id="4af3c-499">Ismétlések száma</span><span class="sxs-lookup"><span data-stu-id="4af3c-499">Retry count</span></span><br /><span data-ttu-id="4af3c-500">Újrapróbálkozási időköz</span><span class="sxs-lookup"><span data-stu-id="4af3c-500">Retry interval</span></span><br /><span data-ttu-id="4af3c-501">Első gyors újrapróbálkozás</span><span class="sxs-lookup"><span data-stu-id="4af3c-501">First fast retry</span></span> |<span data-ttu-id="4af3c-502">3</span><span class="sxs-lookup"><span data-stu-id="4af3c-502">3</span></span><br /><span data-ttu-id="4af3c-503">500 ms</span><span class="sxs-lookup"><span data-stu-id="4af3c-503">500 ms</span></span><br /><span data-ttu-id="4af3c-504">true</span><span class="sxs-lookup"><span data-stu-id="4af3c-504">true</span></span> |<span data-ttu-id="4af3c-505">1. kísérlet – 0 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="4af3c-505">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="4af3c-506">2. kísérlet – 500 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="4af3c-506">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="4af3c-507">3. kísérlet – 500 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="4af3c-507">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="4af3c-508">Háttér</span><span class="sxs-lookup"><span data-stu-id="4af3c-508">Background</span></span><br /><span data-ttu-id="4af3c-509">vagy kötegelt</span><span class="sxs-lookup"><span data-stu-id="4af3c-509">or batch</span></span> |<span data-ttu-id="4af3c-510">30 másodperc</span><span class="sxs-lookup"><span data-stu-id="4af3c-510">30 sec</span></span> |<span data-ttu-id="4af3c-511">ExponentialBackoff</span><span class="sxs-lookup"><span data-stu-id="4af3c-511">ExponentialBackoff</span></span> |<span data-ttu-id="4af3c-512">Ismétlések száma</span><span class="sxs-lookup"><span data-stu-id="4af3c-512">Retry count</span></span><br /><span data-ttu-id="4af3c-513">Visszatartás (min.)</span><span class="sxs-lookup"><span data-stu-id="4af3c-513">Min back-off</span></span><br /><span data-ttu-id="4af3c-514">Visszatartás (max.)</span><span class="sxs-lookup"><span data-stu-id="4af3c-514">Max back-off</span></span><br /><span data-ttu-id="4af3c-515">Visszatartás (változás)</span><span class="sxs-lookup"><span data-stu-id="4af3c-515">Delta back-off</span></span><br /><span data-ttu-id="4af3c-516">Első gyors újrapróbálkozás</span><span class="sxs-lookup"><span data-stu-id="4af3c-516">First fast retry</span></span> |<span data-ttu-id="4af3c-517">5</span><span class="sxs-lookup"><span data-stu-id="4af3c-517">5</span></span><br /><span data-ttu-id="4af3c-518">0 másodperc</span><span class="sxs-lookup"><span data-stu-id="4af3c-518">0 sec</span></span><br /><span data-ttu-id="4af3c-519">60 másodperc</span><span class="sxs-lookup"><span data-stu-id="4af3c-519">60 sec</span></span><br /><span data-ttu-id="4af3c-520">2 másodperc</span><span class="sxs-lookup"><span data-stu-id="4af3c-520">2 sec</span></span><br /><span data-ttu-id="4af3c-521">false</span><span class="sxs-lookup"><span data-stu-id="4af3c-521">false</span></span> |<span data-ttu-id="4af3c-522">1. kísérlet – 0 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="4af3c-522">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="4af3c-523">2. kísérlet – kb. 2 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="4af3c-523">Attempt 2 - delay ~2 sec</span></span><br /><span data-ttu-id="4af3c-524">3. kísérlet – kb. 6 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="4af3c-524">Attempt 3 - delay ~6 sec</span></span><br /><span data-ttu-id="4af3c-525">4. kísérlet – kb. 14 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="4af3c-525">Attempt 4 - delay ~14 sec</span></span><br /><span data-ttu-id="4af3c-526">5. kísérlet – kb. 30 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="4af3c-526">Attempt 5 - delay ~30 sec</span></span> |

> [!NOTE]
> <span data-ttu-id="4af3c-527">A végpontok közötti késés célértéke az alapértelmezett időtúllépési érték használatát feltételezi a szolgáltatáskapcsolatokhoz.</span><span class="sxs-lookup"><span data-stu-id="4af3c-527">The end-to-end latency targets assume the default timeout for connections to the service.</span></span> <span data-ttu-id="4af3c-528">Ha magasabb értéket ad meg a kapcsolatok időtúllépésére, a végpontok közötti késés minden újrapróbálkozási kísérlet esetében ennyivel lesz meghosszabbítva.</span><span class="sxs-lookup"><span data-stu-id="4af3c-528">If you specify longer connection timeouts, the end-to-end latency will be extended by this additional time for every retry attempt.</span></span>
>
>

### <a name="examples"></a><span data-ttu-id="4af3c-529">Példák</span><span class="sxs-lookup"><span data-stu-id="4af3c-529">Examples</span></span>
<span data-ttu-id="4af3c-530">Ebben a szakaszban azt mutatjuk be, hogy miként használhatja a Pollyt az Azure SQL Database elérésére a `Policy` osztályban konfigurált újrapróbálkozási szabályzatok segítségével.</span><span class="sxs-lookup"><span data-stu-id="4af3c-530">This section shows how you can use Polly to access Azure SQL Database using a set of retry policies configured in the `Policy` class.</span></span>

<span data-ttu-id="4af3c-531">A következő programkód bemutatja, hogyan bővítheti az `SqlCommand` osztályt, ha az exponenciális visszatartással hívja meg az `ExecuteAsync` metódust.</span><span class="sxs-lookup"><span data-stu-id="4af3c-531">The following code shows an extension method on the `SqlCommand` class that calls `ExecuteAsync` with exponential backoff.</span></span>

```csharp
public async static Task<SqlDataReader> ExecuteReaderWithRetryAsync(this SqlCommand command)
{
    GuardConnectionIsNotNull(command);

    var policy = Policy.Handle<Exception>().WaitAndRetryAsync(
        retryCount: 3, // Retry 3 times
        sleepDurationProvider: attempt => TimeSpan.FromMilliseconds(200 * Math.Pow(2, attempt - 1)), // Exponential backoff based on an initial 200ms delay.
        onRetry: (exception, attempt) => 
        {
            // Capture some info for logging/telemetry.  
            logger.LogWarn($"ExecuteReaderWithRetryAsync: Retry {attempt} due to {exception}.");
        });

    // Retry the following call according to the policy.
    await policy.ExecuteAsync<SqlDataReader>(async token =>
    {
        // This code is executed within the Policy 

        if (conn.State != System.Data.ConnectionState.Open) await conn.OpenAsync(token);
        return await command.ExecuteReaderAsync(System.Data.CommandBehavior.Default, token);

    }, cancellationToken);
}
```

<span data-ttu-id="4af3c-532">Ezt az aszinkron bővítési metódust a következőképpen lehet használni.</span><span class="sxs-lookup"><span data-stu-id="4af3c-532">This asynchronous extension method can be used as follows.</span></span>

```csharp
var sqlCommand = sqlConnection.CreateCommand();
sqlCommand.CommandText = "[some query]";

using (var reader = await sqlCommand.ExecuteReaderWithRetryAsync())
{
    // Do something with the values
}
```

### <a name="more-information"></a><span data-ttu-id="4af3c-533">További információ</span><span class="sxs-lookup"><span data-stu-id="4af3c-533">More information</span></span>
* [<span data-ttu-id="4af3c-534">Cloud Service Fundamentals adatelérési réteg – átmeneti hibák kezelése</span><span class="sxs-lookup"><span data-stu-id="4af3c-534">Cloud Service Fundamentals Data Access Layer – Transient Fault Handling</span></span>](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx)

<span data-ttu-id="4af3c-535">Ha további útmutatásra van szüksége az SQL Database minél jobb kihasználásához, olvassa el [az Azure SQL Database szolgáltatás teljesítményének és rugalmasságának útmutatóját](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx).</span><span class="sxs-lookup"><span data-stu-id="4af3c-535">For general guidance on getting the most from SQL Database, see [Azure SQL Database Performance and Elasticity Guide](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx).</span></span>

## <a name="sql-database-using-entity-framework-6"></a><span data-ttu-id="4af3c-536">SQL Database-hez az Entity Framework 6</span><span class="sxs-lookup"><span data-stu-id="4af3c-536">SQL Database using Entity Framework 6</span></span>
<span data-ttu-id="4af3c-537">Az SQL Database egy üzemeltetett SQL-adatbázis, amely különböző méretekben, normál (megosztott) és prémium (nem megosztott) szolgáltatásként is elérhető.</span><span class="sxs-lookup"><span data-stu-id="4af3c-537">SQL Database is a hosted SQL database available in a range of sizes and as both a standard (shared) and premium (non-shared) service.</span></span> <span data-ttu-id="4af3c-538">Az Entity Framework egy objektumrelációs leképező .NET-fejlesztők számára, amellyel tartományspecifikus objektumokat használva lehet relációs adatokkal dolgozni.</span><span class="sxs-lookup"><span data-stu-id="4af3c-538">Entity Framework is an object-relational mapper that enables .NET developers to work with relational data using domain-specific objects.</span></span> <span data-ttu-id="4af3c-539">Szükségtelenné teszi az adatelérési kód használatát, amelyet egyébként a fejlesztőknek kell megírniuk.</span><span class="sxs-lookup"><span data-stu-id="4af3c-539">It eliminates the need for most of the data-access code that developers usually need to write.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="4af3c-540">Újrapróbálkozási mechanizmus</span><span class="sxs-lookup"><span data-stu-id="4af3c-540">Retry mechanism</span></span>
<span data-ttu-id="4af3c-541">Az újrapróbálkozás akkor támogatott, ha az SQL Database-t az Entity Framework 6.0-s vagy újabb verziójával érik el, a [kapcsolat rugalmassága/újrapróbálkozási logika](http://msdn.microsoft.com/data/dn456835.aspx) mechanizmus segítségével.</span><span class="sxs-lookup"><span data-stu-id="4af3c-541">Retry support is provided when accessing SQL Database using Entity Framework 6.0 and higher through a mechanism called [Connection Resiliency / Retry Logic](http://msdn.microsoft.com/data/dn456835.aspx).</span></span> <span data-ttu-id="4af3c-542">Az újrapróbálkozási mechanizmus fő funkciói a következők:</span><span class="sxs-lookup"><span data-stu-id="4af3c-542">The main features of the retry mechanism are:</span></span>

* <span data-ttu-id="4af3c-543">Az elsődleges absztrakció az **IDbExecutionStrategy** felület.</span><span class="sxs-lookup"><span data-stu-id="4af3c-543">The primary abstraction is the **IDbExecutionStrategy** interface.</span></span> <span data-ttu-id="4af3c-544">Ez a felület:</span><span class="sxs-lookup"><span data-stu-id="4af3c-544">This interface:</span></span>
  * <span data-ttu-id="4af3c-545">Definiálja a szinkron és aszinkron **feldolgozási**\* metódusokat.</span><span class="sxs-lookup"><span data-stu-id="4af3c-545">Defines synchronous and asynchronous **Execute**\* methods.</span></span>
  * <span data-ttu-id="4af3c-546">Definiálja azokat az osztályokat, amelyek felhasználhatók közvetlenül, illetve konfigurálhatók alapértelmezett stratégiaként egy adatbázis-környezetben, leképezhetők egy szolgáltatónévre, vagy pedig leképezhetők egy szolgáltatónévre vagy kiszolgálónévre.</span><span class="sxs-lookup"><span data-stu-id="4af3c-546">Defines classes that can be used directly or can be configured on a database context as a default strategy, mapped to provider name, or mapped to a provider name and server name.</span></span> <span data-ttu-id="4af3c-547">Ha egy környezethez konfigurálják, az újrapróbálkozások az egyes adatbázis-műveletek szintjén történnek, amelyekből több is lehet egy adott környezeti művelet esetében.</span><span class="sxs-lookup"><span data-stu-id="4af3c-547">When configured on a context, retries occur at the level of individual database operations, of which there might be several for a given context operation.</span></span>
  * <span data-ttu-id="4af3c-548">Definiálja, hogy a sikertelen csatlakozást mikor kövesse újrapróbálkozás, és hogyan.</span><span class="sxs-lookup"><span data-stu-id="4af3c-548">Defines when to retry a failed connection, and how.</span></span>
* <span data-ttu-id="4af3c-549">Az **IDbExecutionStrategy** felület számos beépített implementálását tartalmazza:</span><span class="sxs-lookup"><span data-stu-id="4af3c-549">It includes several built-in implementations of the **IDbExecutionStrategy** interface:</span></span>
  * <span data-ttu-id="4af3c-550">Alapértelmezett – nincs újrapróbálkozás.</span><span class="sxs-lookup"><span data-stu-id="4af3c-550">Default - no retrying.</span></span>
  * <span data-ttu-id="4af3c-551">Az SQL Database alapértelmezett beállítása (automatikus) – nincs újrapróbálkozás, de megvizsgálja a kivételeket, és beburkolja azokat, az SQL Database stratégia használatát javasolva.</span><span class="sxs-lookup"><span data-stu-id="4af3c-551">Default for SQL Database (automatic) - no retrying, but inspects exceptions and wraps them with suggestion to use the SQL Database strategy.</span></span>
  * <span data-ttu-id="4af3c-552">Az SQL Database alapértelmezett beállítása – exponenciális (az alaposztálytól örökölt), plusz az SQL Database észlelési logikája.</span><span class="sxs-lookup"><span data-stu-id="4af3c-552">Default for SQL Database - exponential (inherited from base class) plus SQL Database detection logic.</span></span>
* <span data-ttu-id="4af3c-553">Véletlenszerűsítést tartalmazó exponenciális visszatartási stratégiát implementál.</span><span class="sxs-lookup"><span data-stu-id="4af3c-553">It implements an exponential back-off strategy that includes randomization.</span></span>
* <span data-ttu-id="4af3c-554">A beépített újrapróbálkozási osztályok állapotalapúak, és nem alkalmasak a többszálú futtatásra.</span><span class="sxs-lookup"><span data-stu-id="4af3c-554">The built-in retry classes are stateful and are not thread safe.</span></span> <span data-ttu-id="4af3c-555">Ugyanakkor az aktuális művelet befejezése után újra felhasználhatók.</span><span class="sxs-lookup"><span data-stu-id="4af3c-555">However, they can be reused after the current operation is completed.</span></span>
* <span data-ttu-id="4af3c-556">Amennyiben az újrapróbálkozások száma meghaladja a megadott értéket, a szolgáltatás új kivételbe burkolja az eredményeket.</span><span class="sxs-lookup"><span data-stu-id="4af3c-556">If the specified retry count is exceeded, the results are wrapped in a new exception.</span></span> <span data-ttu-id="4af3c-557">Nem rendezi buborékba az aktuális kivételt.</span><span class="sxs-lookup"><span data-stu-id="4af3c-557">It does not bubble up the current exception.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="4af3c-558">Szabályzatkonfiguráció</span><span class="sxs-lookup"><span data-stu-id="4af3c-558">Policy configuration</span></span>
<span data-ttu-id="4af3c-559">Az újrapróbálkozás akkor támogatott, ha az SQL Database-t az Entity Framework 6.0-s vagy újabb verziójával érik el.</span><span class="sxs-lookup"><span data-stu-id="4af3c-559">Retry support is provided when accessing SQL Database using Entity Framework 6.0 and higher.</span></span> <span data-ttu-id="4af3c-560">Az újrapróbálkozási szabályzatok konfigurálása szoftveresen történik.</span><span class="sxs-lookup"><span data-stu-id="4af3c-560">Retry policies are configured programmatically.</span></span> <span data-ttu-id="4af3c-561">A konfiguráció nem módosítható az egyes műveletek szintjén.</span><span class="sxs-lookup"><span data-stu-id="4af3c-561">The configuration cannot be changed on a per-operation basis.</span></span>

<span data-ttu-id="4af3c-562">Ha egy környezetfüggő stratégiát tesz alapértelmezetté, megad egy függvényt, amely igény szerint hoz létre egy új stratégiát.</span><span class="sxs-lookup"><span data-stu-id="4af3c-562">When configuring a strategy on the context as the default, you specify a function that creates a new strategy on demand.</span></span> <span data-ttu-id="4af3c-563">A következő kód azt mutatja be, hogyan hozható létre egy újrapróbálkozási konfigurációs osztályt, amely kibővíti a **DbConfiguration** alaposztályt.</span><span class="sxs-lookup"><span data-stu-id="4af3c-563">The following code shows how you can create a retry configuration class that extends the **DbConfiguration** base class.</span></span>

```csharp
public class BloggingContextConfiguration : DbConfiguration
{
  public BlogConfiguration()
  {
    // Set up the execution strategy for SQL Database (exponential) with 5 retries and 4 sec delay
    this.SetExecutionStrategy(
         "System.Data.SqlClient", () => new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(4)));
  }
}
```

<span data-ttu-id="4af3c-564">Az alkalmazás indításakor, a **DbConfiguration**-példány **SetConfiguration** metódusával teheti az alapértelmezett újrapróbálkozási stratégiává minden művelet számára.</span><span class="sxs-lookup"><span data-stu-id="4af3c-564">You can then specify this as the default retry strategy for all operations using the **SetConfiguration** method of the **DbConfiguration** instance when the application starts.</span></span> <span data-ttu-id="4af3c-565">Alapértelmezés szerint az EF automatikusan észleli és használatba veszi a konfigurációs osztályt.</span><span class="sxs-lookup"><span data-stu-id="4af3c-565">By default, EF will automatically discover and use the configuration class.</span></span>

```csharp
DbConfiguration.SetConfiguration(new BloggingContextConfiguration());
```

<span data-ttu-id="4af3c-566">Ha meg kíván adni egy újrapróbálkozási konfigurációs osztályt a környezethez, lássa el a környezeti osztályt egy **DbConfigurationType** attribútummal.</span><span class="sxs-lookup"><span data-stu-id="4af3c-566">You can specify the retry configuration class for a context by annotating the context class with a **DbConfigurationType** attribute.</span></span> <span data-ttu-id="4af3c-567">Amennyiben csak egy konfigurációs osztálya van, az EF azt fogja használni, nem szükséges külön jeleznie ezt.</span><span class="sxs-lookup"><span data-stu-id="4af3c-567">However, if you have only one configuration class, EF will use it without the need to annotate the context.</span></span>

```csharp
[DbConfigurationType(typeof(BloggingContextConfiguration))]
public class BloggingContext : DbContext
```

<span data-ttu-id="4af3c-568">Ha eltérő újrapróbálkozási stratégiára van szüksége adott műveletek esetében, vagy le kívánja tiltani az újrapróbálkozásokat egyes műveleteknél, létrehozhat egy konfigurációs osztályt, amely lehetővé teszi stratégiák felfüggesztését vagy cseréjét egy jelző a **CallContext** osztályban történő elhelyezésével.</span><span class="sxs-lookup"><span data-stu-id="4af3c-568">If you need to use different retry strategies for specific operations, or disable retries for specific operations, you can create a configuration class that allows you to suspend or swap strategies by setting a flag in the **CallContext**.</span></span> <span data-ttu-id="4af3c-569">A konfigurációs osztály ezt a jelzőt használva vált stratégiát vagy tiltja le a felhasználó által megadott stratégiát, majd az alapértelmezett stratégiát használja.</span><span class="sxs-lookup"><span data-stu-id="4af3c-569">The configuration class can use this flag to switch strategies, or disable the strategy you provide and use a default strategy.</span></span> <span data-ttu-id="4af3c-570">További információt a végrehajtási stratégiák újrapróbálkozásainak korlátozásait ismertető oldal [végrehajtási stratégia felfüggesztését](http://msdn.microsoft.com/dn307226#transactions_workarounds) ismertető szakaszában talál (EF6-os vagy újabb verzió).</span><span class="sxs-lookup"><span data-stu-id="4af3c-570">For more information, see [Suspend Execution Strategy](http://msdn.microsoft.com/dn307226#transactions_workarounds) in the page Limitations with Retrying Execution Strategies (EF6 onwards).</span></span>

<span data-ttu-id="4af3c-571">Bizonyos műveletek esetében egy másik lehetőség adott újrapróbálkozási stratégia használatára, ha létrehozza a kívánt stratégiaosztály egy példányát, és paramétereket használva ellátja a kívánt beállításokkal.</span><span class="sxs-lookup"><span data-stu-id="4af3c-571">Another technique for using specific retry strategies for individual operations is to create an instance of the required strategy class and supply the desired settings through parameters.</span></span> <span data-ttu-id="4af3c-572">Ezután meghívja annak **ExecuteAsync** metódusát.</span><span class="sxs-lookup"><span data-stu-id="4af3c-572">You then invoke its **ExecuteAsync** method.</span></span>

```csharp
var executionStrategy = new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(4));
var blogs = await executionStrategy.ExecuteAsync(
    async () =>
    {
        using (var db = new BloggingContext("Blogs"))
        {
            // Acquire some values asynchronously and return them
        }
    },
    new CancellationToken()
);
```

<span data-ttu-id="4af3c-573">A **DbConfiguration** osztály használatának legegyszerűbb módja, ha megkeresi ugyanazt a szerelvényt, amely a **DbContext** osztályban szerepel.</span><span class="sxs-lookup"><span data-stu-id="4af3c-573">The simplest way to use a **DbConfiguration** class is to locate it in the same assembly as the **DbContext** class.</span></span> <span data-ttu-id="4af3c-574">Ez azonban nem alkalmazható, ha különböző forgatókönyvekben van szükség ugyanarra a környezetre, például a különböző interaktív és háttérbeli újrapróbálkozási stratégiák esetében.</span><span class="sxs-lookup"><span data-stu-id="4af3c-574">However, this is not appropriate when the same context is required in different scenarios, such as different interactive and background retry strategies.</span></span> <span data-ttu-id="4af3c-575">Ha a különböző környezetek különálló alkalmazástartományokban futnak, igénybe veheti a konfigurációs osztályok a konfigurációs fájlban történő megadásának beépített lehetőségét, illetve beállíthatja azt közvetlenül is a kódban.</span><span class="sxs-lookup"><span data-stu-id="4af3c-575">If the different contexts execute in separate AppDomains, you can use the built-in support for specifying configuration classes in the configuration file or set it explicitly using code.</span></span> <span data-ttu-id="4af3c-576">Ha a különböző környezeteknek ugyanabban az alkalmazástartományban kell futniuk, egyéni megoldásra lesz szükség.</span><span class="sxs-lookup"><span data-stu-id="4af3c-576">If the different contexts must execute in the same AppDomain, a custom solution will be required.</span></span>

<span data-ttu-id="4af3c-577">További információt a [kódalapú konfigurálást (EF6-os vagy újabb verzió)](http://msdn.microsoft.com/data/jj680699.aspx) ismertető szakaszban találhat.</span><span class="sxs-lookup"><span data-stu-id="4af3c-577">For more information, see [Code-Based Configuration (EF6 onwards)](http://msdn.microsoft.com/data/jj680699.aspx).</span></span>

<span data-ttu-id="4af3c-578">A következő táblázatban a beépített újrapróbálkozási szabályzat alapértelmezett beállításai láthatók az EF6 használatakor.</span><span class="sxs-lookup"><span data-stu-id="4af3c-578">The following table shows the default settings for the built-in retry policy when using EF6.</span></span>

| <span data-ttu-id="4af3c-579">Beállítás</span><span class="sxs-lookup"><span data-stu-id="4af3c-579">Setting</span></span> | <span data-ttu-id="4af3c-580">Alapértelmezett érték</span><span class="sxs-lookup"><span data-stu-id="4af3c-580">Default value</span></span> | <span data-ttu-id="4af3c-581">Jelentés</span><span class="sxs-lookup"><span data-stu-id="4af3c-581">Meaning</span></span> |
|---------|---------------|---------|
| <span data-ttu-id="4af3c-582">Szabályzat</span><span class="sxs-lookup"><span data-stu-id="4af3c-582">Policy</span></span> | <span data-ttu-id="4af3c-583">Exponenciális</span><span class="sxs-lookup"><span data-stu-id="4af3c-583">Exponential</span></span> | <span data-ttu-id="4af3c-584">Exponenciális visszatartás.</span><span class="sxs-lookup"><span data-stu-id="4af3c-584">Exponential back-off.</span></span> |
| <span data-ttu-id="4af3c-585">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="4af3c-585">MaxRetryCount</span></span> | <span data-ttu-id="4af3c-586">5</span><span class="sxs-lookup"><span data-stu-id="4af3c-586">5</span></span> | <span data-ttu-id="4af3c-587">Az újrapróbálkozások maximális száma.</span><span class="sxs-lookup"><span data-stu-id="4af3c-587">The maximum number of retries.</span></span> |
| <span data-ttu-id="4af3c-588">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="4af3c-588">MaxDelay</span></span> | <span data-ttu-id="4af3c-589">30 másodperc</span><span class="sxs-lookup"><span data-stu-id="4af3c-589">30 seconds</span></span> | <span data-ttu-id="4af3c-590">Az újrapróbálkozások közötti maximális késleltetés.</span><span class="sxs-lookup"><span data-stu-id="4af3c-590">The maximum delay between retries.</span></span> <span data-ttu-id="4af3c-591">Az érték nem befolyásolja az egymást követő késleltetések kiszámítását.</span><span class="sxs-lookup"><span data-stu-id="4af3c-591">This value does not affect how the series of delays are computed.</span></span> <span data-ttu-id="4af3c-592">Csak a felső határt adja meg.</span><span class="sxs-lookup"><span data-stu-id="4af3c-592">It only defines an upper bound.</span></span> |
| <span data-ttu-id="4af3c-593">DefaultCoefficient</span><span class="sxs-lookup"><span data-stu-id="4af3c-593">DefaultCoefficient</span></span> | <span data-ttu-id="4af3c-594">1 másodperc</span><span class="sxs-lookup"><span data-stu-id="4af3c-594">1 second</span></span> | <span data-ttu-id="4af3c-595">Az exponenciális visszatartás számításának együtthatója.</span><span class="sxs-lookup"><span data-stu-id="4af3c-595">The coefficient for the exponential back-off computation.</span></span> <span data-ttu-id="4af3c-596">Ez az érték nem módosítható.</span><span class="sxs-lookup"><span data-stu-id="4af3c-596">This value cannot be changed.</span></span> |
| <span data-ttu-id="4af3c-597">DefaultRandomFactor</span><span class="sxs-lookup"><span data-stu-id="4af3c-597">DefaultRandomFactor</span></span> | <span data-ttu-id="4af3c-598">1.1</span><span class="sxs-lookup"><span data-stu-id="4af3c-598">1.1</span></span> | <span data-ttu-id="4af3c-599">A véletlenszerű késleltetés bejegyzésekhez való hozzáadására szolgáló szorzó.</span><span class="sxs-lookup"><span data-stu-id="4af3c-599">The multiplier used to add a random delay for each entry.</span></span> <span data-ttu-id="4af3c-600">Ez az érték nem módosítható.</span><span class="sxs-lookup"><span data-stu-id="4af3c-600">This value cannot be changed.</span></span> |
| <span data-ttu-id="4af3c-601">DefaultExponentialBase</span><span class="sxs-lookup"><span data-stu-id="4af3c-601">DefaultExponentialBase</span></span> | <span data-ttu-id="4af3c-602">2</span><span class="sxs-lookup"><span data-stu-id="4af3c-602">2</span></span> | <span data-ttu-id="4af3c-603">A következő késleltetés kiszámítására szolgáló szorzó.</span><span class="sxs-lookup"><span data-stu-id="4af3c-603">The multiplier used to calculate the next delay.</span></span> <span data-ttu-id="4af3c-604">Ez az érték nem módosítható.</span><span class="sxs-lookup"><span data-stu-id="4af3c-604">This value cannot be changed.</span></span> |

### <a name="retry-usage-guidance"></a><span data-ttu-id="4af3c-605">Újrapróbálkozásokra vonatkozó útmutató</span><span class="sxs-lookup"><span data-stu-id="4af3c-605">Retry usage guidance</span></span>
<span data-ttu-id="4af3c-606">Ügyeljen a következőkre, amikor az EF6 használatával éri el az SQL Database-t:</span><span class="sxs-lookup"><span data-stu-id="4af3c-606">Consider the following guidelines when accessing SQL Database using EF6:</span></span>

* <span data-ttu-id="4af3c-607">Válassza a megfelelő szolgáltatástípust (megosztott vagy prémium).</span><span class="sxs-lookup"><span data-stu-id="4af3c-607">Choose the appropriate service option (shared or premium).</span></span> <span data-ttu-id="4af3c-608">A megosztott példány esetében a szokottnál hosszabb csatlakozási késések fordulhatnak elő, valamint a kérések számának korlátozására lehet szükség, mivel a megosztott kiszolgálót más bérlők is használják.</span><span class="sxs-lookup"><span data-stu-id="4af3c-608">A shared instance may suffer longer than usual connection delays and throttling due to the usage by other tenants of the shared server.</span></span> <span data-ttu-id="4af3c-609">Ha kiszámítható teljesítményre és megbízhatóan alacsony késésű műveletekre van szükség, mindenképpen a prémium szolgáltatást érdemes választani.</span><span class="sxs-lookup"><span data-stu-id="4af3c-609">If predictable performance and reliable low latency operations are required, consider choosing the premium option.</span></span>
* <span data-ttu-id="4af3c-610">Az Azure SQL Database esetében nem javasoljuk rögzített időközű stratégiák alkalmazását.</span><span class="sxs-lookup"><span data-stu-id="4af3c-610">A fixed interval strategy is not recommended for use with Azure SQL Database.</span></span> <span data-ttu-id="4af3c-611">Használjon inkább egy exponenciális visszatartási stratégiát, mivel a szolgáltatás túlterhelődhet, és a hosszabb késleltetés több időt ad a helyreállításra.</span><span class="sxs-lookup"><span data-stu-id="4af3c-611">Instead, use an exponential back-off strategy because the service may be overloaded, and longer delays allow more time for it to recover.</span></span>
* <span data-ttu-id="4af3c-612">A kapcsolatok definiálásakor válasszon megfelelő értéket a kapcsolatok és a parancsok időtúllépéséhez.</span><span class="sxs-lookup"><span data-stu-id="4af3c-612">Choose a suitable value for the connection and command timeouts when defining connections.</span></span> <span data-ttu-id="4af3c-613">Az időtúllépés ideális értékét saját üzleti logikájának felépítése és tesztek alapján állapíthatja meg.</span><span class="sxs-lookup"><span data-stu-id="4af3c-613">Base the timeout on both your business logic design and through testing.</span></span> <span data-ttu-id="4af3c-614">A későbbiekben szükség lehet az érték módosítására, ahogy változik az adatmennyiség, vagy változnak az üzleti folyamatok.</span><span class="sxs-lookup"><span data-stu-id="4af3c-614">You may need to modify this value over time as the volumes of data or the business processes change.</span></span> <span data-ttu-id="4af3c-615">A túl alacsony időtúllépési érték miatt a kapcsolatok idő előtt szakadhatnak meg, ha az adatbázis leterhelt.</span><span class="sxs-lookup"><span data-stu-id="4af3c-615">Too short a timeout may result in premature failures of connections when the database is busy.</span></span> <span data-ttu-id="4af3c-616">A túl magas időtúllépési érték akadályozhatja az újrapróbálkozási logika megfelelő működését, mivel túl sokáig fog várni, mielőtt észlelné a sikertelen csatlakozást.</span><span class="sxs-lookup"><span data-stu-id="4af3c-616">Too long a timeout may prevent the retry logic working correctly by waiting too long before detecting a failed connection.</span></span> <span data-ttu-id="4af3c-617">Az időtúllépés értéke a végpontok közötti késés egyik összetevője. Bár nehéz előre megállapítani, hogy hány parancsot kell végrehajtani a környezet mentésekor.</span><span class="sxs-lookup"><span data-stu-id="4af3c-617">The value of the timeout is a component of the end-to-end latency, although you cannot easily determine how many commands will execute when saving the context.</span></span> <span data-ttu-id="4af3c-618">Az alapértelmezett időtúllépést a **DbContext** példány **CommandTimeout** tulajdonságában adhatja meg.</span><span class="sxs-lookup"><span data-stu-id="4af3c-618">You can change the default timeout by setting the **CommandTimeout** property of the **DbContext** instance.</span></span>
* <span data-ttu-id="4af3c-619">Az Entity Framework támogatja a konfigurációs fájlokban definiált újrapróbálkozási konfigurációk használatát.</span><span class="sxs-lookup"><span data-stu-id="4af3c-619">Entity Framework supports retry configurations defined in configuration files.</span></span> <span data-ttu-id="4af3c-620">Ugyanakkor a minél nagyobb rugalmasság érdekében azt javasoljuk, hogy a konfigurációt szoftveresen hozza létre az alkalmazásban.</span><span class="sxs-lookup"><span data-stu-id="4af3c-620">However, for maximum flexibility on Azure you should consider creating the configuration programmatically within the application.</span></span> <span data-ttu-id="4af3c-621">Az újrapróbálkozási szabályzatok konkrét paraméterei – például az újrapróbálkozások száma és az újrapróbálkozási időközök – a szolgáltatás konfigurációs fájljában tárolhatók, és a futtatás során felhasználhatók a megfelelő szabályzatok létrehozására.</span><span class="sxs-lookup"><span data-stu-id="4af3c-621">The specific parameters for the retry policies, such as the number of retries and the retry intervals, can be stored in the service configuration file and used at runtime to create the appropriate policies.</span></span> <span data-ttu-id="4af3c-622">Így a beállítások az alkalmazás újraindítása nélkül módosíthatók.</span><span class="sxs-lookup"><span data-stu-id="4af3c-622">This allows the settings to be changed without requiring the application to be restarted.</span></span>

<span data-ttu-id="4af3c-623">A következő kezdőbeállításokat javasoljuk az újrapróbálkozási műveletekhez.</span><span class="sxs-lookup"><span data-stu-id="4af3c-623">Consider starting with the following settings for retrying operations.</span></span> <span data-ttu-id="4af3c-624">Az újrapróbálkozási kísérletek közötti késleltetés nem adható meg (rögzített, és egy exponenciális sorozat részeként jön létre).</span><span class="sxs-lookup"><span data-stu-id="4af3c-624">You cannot specify the delay between retry attempts (it is fixed and generated as an exponential sequence).</span></span> <span data-ttu-id="4af3c-625">Ha nem hoz létre egyéni újrapróbálkozási stratégiát, csak a maximális értékeket adhatja meg, az itt látható módon.</span><span class="sxs-lookup"><span data-stu-id="4af3c-625">You can specify only the maximum values, as shown here; unless you create a custom retry strategy.</span></span> <span data-ttu-id="4af3c-626">Ezek általános célú beállítások, ezért javasoljuk, hogy monitorozza műveleteit, és finomhangolja az értékeket saját igényei szerint.</span><span class="sxs-lookup"><span data-stu-id="4af3c-626">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="4af3c-627">**Környezet**</span><span class="sxs-lookup"><span data-stu-id="4af3c-627">**Context**</span></span> | <span data-ttu-id="4af3c-628">**Mintacél E2E<br />max. késleltetése**</span><span class="sxs-lookup"><span data-stu-id="4af3c-628">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="4af3c-629">**Újrapróbálkozási szabályzat**</span><span class="sxs-lookup"><span data-stu-id="4af3c-629">**Retry policy**</span></span> | <span data-ttu-id="4af3c-630">**Beállítások**</span><span class="sxs-lookup"><span data-stu-id="4af3c-630">**Settings**</span></span> | <span data-ttu-id="4af3c-631">**Értékek**</span><span class="sxs-lookup"><span data-stu-id="4af3c-631">**Values**</span></span> | <span data-ttu-id="4af3c-632">**Működési elv**</span><span class="sxs-lookup"><span data-stu-id="4af3c-632">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="4af3c-633">Interaktív, felhasználói felület</span><span class="sxs-lookup"><span data-stu-id="4af3c-633">Interactive, UI,</span></span><br /><span data-ttu-id="4af3c-634">vagy előtér</span><span class="sxs-lookup"><span data-stu-id="4af3c-634">or foreground</span></span> |<span data-ttu-id="4af3c-635">2 másodperc</span><span class="sxs-lookup"><span data-stu-id="4af3c-635">2 seconds</span></span> |<span data-ttu-id="4af3c-636">Exponenciális</span><span class="sxs-lookup"><span data-stu-id="4af3c-636">Exponential</span></span> |<span data-ttu-id="4af3c-637">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="4af3c-637">MaxRetryCount</span></span><br /><span data-ttu-id="4af3c-638">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="4af3c-638">MaxDelay</span></span> |<span data-ttu-id="4af3c-639">3</span><span class="sxs-lookup"><span data-stu-id="4af3c-639">3</span></span><br /><span data-ttu-id="4af3c-640">750 ms</span><span class="sxs-lookup"><span data-stu-id="4af3c-640">750 ms</span></span> |<span data-ttu-id="4af3c-641">1. kísérlet – 0 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="4af3c-641">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="4af3c-642">2. kísérlet – 750 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="4af3c-642">Attempt 2 - delay 750 ms</span></span><br /><span data-ttu-id="4af3c-643">3. kísérlet – 750 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="4af3c-643">Attempt 3 – delay 750 ms</span></span> |
| <span data-ttu-id="4af3c-644">Háttér</span><span class="sxs-lookup"><span data-stu-id="4af3c-644">Background</span></span><br /> <span data-ttu-id="4af3c-645">vagy kötegelt</span><span class="sxs-lookup"><span data-stu-id="4af3c-645">or batch</span></span> |<span data-ttu-id="4af3c-646">30 másodperc</span><span class="sxs-lookup"><span data-stu-id="4af3c-646">30 seconds</span></span> |<span data-ttu-id="4af3c-647">Exponenciális</span><span class="sxs-lookup"><span data-stu-id="4af3c-647">Exponential</span></span> |<span data-ttu-id="4af3c-648">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="4af3c-648">MaxRetryCount</span></span><br /><span data-ttu-id="4af3c-649">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="4af3c-649">MaxDelay</span></span> |<span data-ttu-id="4af3c-650">5</span><span class="sxs-lookup"><span data-stu-id="4af3c-650">5</span></span><br /><span data-ttu-id="4af3c-651">12 másodperc</span><span class="sxs-lookup"><span data-stu-id="4af3c-651">12 seconds</span></span> |<span data-ttu-id="4af3c-652">1. kísérlet – 0 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="4af3c-652">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="4af3c-653">2. kísérlet – kb. 1 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="4af3c-653">Attempt 2 - delay ~1 sec</span></span><br /><span data-ttu-id="4af3c-654">3. kísérlet – kb. 3 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="4af3c-654">Attempt 3 - delay ~3 sec</span></span><br /><span data-ttu-id="4af3c-655">4. kísérlet – kb. 7 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="4af3c-655">Attempt 4 - delay ~7 sec</span></span><br /><span data-ttu-id="4af3c-656">5. kísérlet – 12 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="4af3c-656">Attempt 5 - delay 12 sec</span></span> |

> [!NOTE]
> <span data-ttu-id="4af3c-657">A végpontok közötti késés célértéke az alapértelmezett időtúllépési érték használatát feltételezi a szolgáltatáskapcsolatokhoz.</span><span class="sxs-lookup"><span data-stu-id="4af3c-657">The end-to-end latency targets assume the default timeout for connections to the service.</span></span> <span data-ttu-id="4af3c-658">Ha magasabb értéket ad meg a kapcsolatok időtúllépésére, a végpontok közötti késés minden újrapróbálkozási kísérlet esetében ennyivel lesz meghosszabbítva.</span><span class="sxs-lookup"><span data-stu-id="4af3c-658">If you specify longer connection timeouts, the end-to-end latency will be extended by this additional time for every retry attempt.</span></span>
>
>

### <a name="examples"></a><span data-ttu-id="4af3c-659">Példák</span><span class="sxs-lookup"><span data-stu-id="4af3c-659">Examples</span></span>
<span data-ttu-id="4af3c-660">A következő mintakód egy egyszerű adatelérési megoldást definiál, amely az Entity Frameworköt használja.</span><span class="sxs-lookup"><span data-stu-id="4af3c-660">The following code example defines a simple data access solution that uses Entity Framework.</span></span> <span data-ttu-id="4af3c-661">Egy adott újrapróbálkozási stratégiát állít be a **BlogConfiguration** osztály egy példányának definiálásával, amely a **DbConfiguration** osztályt bővíti ki.</span><span class="sxs-lookup"><span data-stu-id="4af3c-661">It sets a specific retry strategy by defining an instance of a class named **BlogConfiguration** that extends **DbConfiguration**.</span></span>

```csharp
using System;
using System.Collections.Generic;
using System.Data.Entity;
using System.Data.Entity.SqlServer;
using System.Threading.Tasks;

namespace RetryCodeSamples
{
    public class BlogConfiguration : DbConfiguration
    {
        public BlogConfiguration()
        {
            // Set up the execution strategy for SQL Database (exponential) with 5 retries and 12 sec delay.
            // These values could be loaded from configuration rather than being hard-coded.
            this.SetExecutionStrategy(
                    "System.Data.SqlClient", () => new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(12)));
        }
    }

    // Specify the configuration type if more than one has been defined.
    // [DbConfigurationType(typeof(BlogConfiguration))]
    public class BloggingContext : DbContext
    {
        // Definition of content goes here.
    }

    class EF6CodeSamples
    {
        public async static Task Samples()
        {
            // Execution strategy configured by DbConfiguration subclass, discovered automatically or
            // or explicitly indicated through configuration or with an attribute. Default is no retries.
            using (var db = new BloggingContext("Blogs"))
            {
                // Add, edit, delete blog items here, then:
                await db.SaveChangesAsync();
            }
        }
    }
}
```

<span data-ttu-id="4af3c-662">A [kapcsolat rugalmassága/újrapróbálkozási logika](http://msdn.microsoft.com/data/dn456835.aspx) mechanizmussal foglalkozó témakörben további példákat talál az Entity Framework újrapróbálkozási mechanizmusának használatára.</span><span class="sxs-lookup"><span data-stu-id="4af3c-662">More examples of using the Entity Framework retry mechanism can be found in [Connection Resiliency / Retry Logic](http://msdn.microsoft.com/data/dn456835.aspx).</span></span>

### <a name="more-information"></a><span data-ttu-id="4af3c-663">További információ</span><span class="sxs-lookup"><span data-stu-id="4af3c-663">More information</span></span>
* [<span data-ttu-id="4af3c-664">Az Azure SQL Database szolgáltatás teljesítményének és rugalmasságának útmutatója</span><span class="sxs-lookup"><span data-stu-id="4af3c-664">Azure SQL Database Performance and Elasticity Guide</span></span>](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx)

## <a name="sql-database-using-entity-framework-core"></a><span data-ttu-id="4af3c-665">SQL-adatbázisokhoz Entity Framework Core használatával</span><span class="sxs-lookup"><span data-stu-id="4af3c-665">SQL Database using Entity Framework Core</span></span>
<span data-ttu-id="4af3c-666">Az [Entity Framework Core](/ef/core/) egy objektumrelációs leképező .NET Core-fejlesztők számára, amellyel tartományspecifikus objektumokat használva lehet adatokkal dolgozni.</span><span class="sxs-lookup"><span data-stu-id="4af3c-666">[Entity Framework Core](/ef/core/) is an object-relational mapper that enables .NET Core developers to work with data using domain-specific objects.</span></span> <span data-ttu-id="4af3c-667">Szükségtelenné teszi az adatelérési kód használatát, amelyet egyébként a fejlesztőknek kell megírniuk.</span><span class="sxs-lookup"><span data-stu-id="4af3c-667">It eliminates the need for most of the data-access code that developers usually need to write.</span></span> <span data-ttu-id="4af3c-668">Az Entity Framework e verzióját az alapoktól építettük újra, ezért nem örökli meg automatikusan az EF6.x összes funkcióját.</span><span class="sxs-lookup"><span data-stu-id="4af3c-668">This version of Entity Framework was written from the ground up, and doesn't automatically inherit all the features from EF6.x.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="4af3c-669">Újrapróbálkozási mechanizmus</span><span class="sxs-lookup"><span data-stu-id="4af3c-669">Retry mechanism</span></span>
<span data-ttu-id="4af3c-670">Az újrapróbálkozás akkor támogatott, ha az SQL Database-t az Entity Framework Core-ral érik el, a [kapcsolat rugalmassága](/ef/core/miscellaneous/connection-resiliency) mechanizmus segítségével.</span><span class="sxs-lookup"><span data-stu-id="4af3c-670">Retry support is provided when accessing SQL Database using Entity Framework Core through a mechanism called [Connection Resiliency](/ef/core/miscellaneous/connection-resiliency).</span></span> <span data-ttu-id="4af3c-671">A kapcsolat rugalmassága először az EF Core 1.1.0-ban vált elérhetővé.</span><span class="sxs-lookup"><span data-stu-id="4af3c-671">Connection resiliency was introduced in EF Core 1.1.0.</span></span>

<span data-ttu-id="4af3c-672">Az elsődleges absztrakció az `IExecutionStrategy` felület.</span><span class="sxs-lookup"><span data-stu-id="4af3c-672">The primary abstraction is the `IExecutionStrategy` interface.</span></span> <span data-ttu-id="4af3c-673">Az SQL Server, és így az SQL Azure végrehajtási stratégiája felismeri a kivételtípusokat, amelyeknél elérhető az újrapróbálkozás, és észszerű alapértelmezett értékek vannak beállítva többek között az újrapróbálkozások maximális számánál és az újrapróbálkozások közötti késleltetésnél.</span><span class="sxs-lookup"><span data-stu-id="4af3c-673">The execution strategy for SQL Server, including SQL Azure, is aware of the exception types that can be retried and has sensible defaults for maximum retries, delay between retries, and so on.</span></span>

### <a name="examples"></a><span data-ttu-id="4af3c-674">Példák</span><span class="sxs-lookup"><span data-stu-id="4af3c-674">Examples</span></span>

<span data-ttu-id="4af3c-675">A következő kód lehetővé teszi az automatikus újrapróbálkozást a DbContext objektum konfigurálásakor, ami egy munkamenetet jelöl az adatbázisban.</span><span class="sxs-lookup"><span data-stu-id="4af3c-675">The following code enables automatic retries when configuring the DbContext object, which represents a session with the database.</span></span> 

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .UseSqlServer(
            @"Server=(localdb)\mssqllocaldb;Database=EFMiscellanous.ConnectionResiliency;Trusted_Connection=True;",
            options => options.EnableRetryOnFailure());
}
```

<span data-ttu-id="4af3c-676">A következő kód bemutatja, egy végrehajtási stratégia alkalmazásával hogyan hajtható végre egy tranzakció automatikus újrapróbálkozással.</span><span class="sxs-lookup"><span data-stu-id="4af3c-676">The following code shows how to execute a transaction with automatic retries, by using an execution strategy.</span></span> <span data-ttu-id="4af3c-677">A tranzakciót egy delegált definiálja.</span><span class="sxs-lookup"><span data-stu-id="4af3c-677">The transaction is defined in a delegate.</span></span> <span data-ttu-id="4af3c-678">Átmeneti hiba előfordulásakor a végrehajtási stratégia ismét meghívja a delegáltat.</span><span class="sxs-lookup"><span data-stu-id="4af3c-678">If a transient failure occurs, the execution strategy will invoke the delegate again.</span></span>

```csharp
using (var db = new BloggingContext())
{
    var strategy = db.Database.CreateExecutionStrategy();

    strategy.Execute(() =>
    {
        using (var transaction = db.Database.BeginTransaction())
        {
            db.Blogs.Add(new Blog { Url = "http://blogs.msdn.com/dotnet" });
            db.SaveChanges();

            db.Blogs.Add(new Blog { Url = "http://blogs.msdn.com/visualstudio" });
            db.SaveChanges();

            transaction.Commit();
        }
    });
}
```

## <a name="azure-storage"></a><span data-ttu-id="4af3c-679">Azure Storage</span><span class="sxs-lookup"><span data-stu-id="4af3c-679">Azure Storage</span></span>
<span data-ttu-id="4af3c-680">Az Azure Storage szolgáltatásai közé tartoznak a tábla- és blobtárolók, a fájl- és tárolási üzenetsorok.</span><span class="sxs-lookup"><span data-stu-id="4af3c-680">Azure storage services include table and blob storage, files, and storage queues.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="4af3c-681">Újrapróbálkozási mechanizmus</span><span class="sxs-lookup"><span data-stu-id="4af3c-681">Retry mechanism</span></span>
<span data-ttu-id="4af3c-682">Újrapróbálkozásokra az egyes REST-műveletek szintjén kerül sor, és az ügyfél-API implementálásának szerves részeit képezik.</span><span class="sxs-lookup"><span data-stu-id="4af3c-682">Retries occur at the individual REST operation level and are an integral part of the client API implementation.</span></span> <span data-ttu-id="4af3c-683">Az ügyfél Storage SDK-ja az [IExtendedRetryPolicy felületet](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.aspx) implementáló osztályokat alkalmaz.</span><span class="sxs-lookup"><span data-stu-id="4af3c-683">The client storage SDK uses classes that implement the [IExtendedRetryPolicy Interface](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.aspx).</span></span>

<span data-ttu-id="4af3c-684">A felület több módon implementálható.</span><span class="sxs-lookup"><span data-stu-id="4af3c-684">There are different implementations of the interface.</span></span> <span data-ttu-id="4af3c-685">A Storage-ügyfelek kifejezetten táblákra, blobokra és üzenetsorokra szabott szabályzatok közül választhatnak.</span><span class="sxs-lookup"><span data-stu-id="4af3c-685">Storage clients can choose from policies specifically designed for accessing tables, blobs, and queues.</span></span> <span data-ttu-id="4af3c-686">Minden implementáció különböző újrapróbálkozási stratégiát használ, amely alapvetően az újrapróbálkozási időközöket és egyéb részleteket határoz meg.</span><span class="sxs-lookup"><span data-stu-id="4af3c-686">Each implementation uses a different retry strategy that essentially defines the retry interval and other details.</span></span>

<span data-ttu-id="4af3c-687">A beépített osztályok támogatják a lineáris (állandó késleltetés) és az exponenciális (véletlenszerű újrapróbálkozási időközök) konfigurációkat.</span><span class="sxs-lookup"><span data-stu-id="4af3c-687">The built-in classes provide support for linear (constant delay) and exponential with randomization retry intervals.</span></span> <span data-ttu-id="4af3c-688">Nincs szükség újrapróbálkozási szabályzat beállítására, ha egy másik folyamat magasabb szinten kezeli az újrapróbálkozásokat.</span><span class="sxs-lookup"><span data-stu-id="4af3c-688">There is also a no retry policy for use when another process is handling retries at a higher level.</span></span> <span data-ttu-id="4af3c-689">Ugyanakkor implementálhat saját újrapróbálkozási osztályokat, ha a beépített osztályok nem felelnek meg valamilyen konkrét követelménynek.</span><span class="sxs-lookup"><span data-stu-id="4af3c-689">However, you can implement your own retry classes if you have specific requirements not provided by the built-in classes.</span></span>

<span data-ttu-id="4af3c-690">A váltakozó újrapróbálkozás a tárolási szolgáltatás elsődleges és másodlagos helye között vált, ha írásvédett georedundáns tárolót (RA-GRS) használ, és a kérés eredménye egy újrapróbálható hiba.</span><span class="sxs-lookup"><span data-stu-id="4af3c-690">Alternate retries switch between primary and secondary storage service location if you are using read access geo-redundant storage (RA-GRS) and the result of the request is a retryable error.</span></span> <span data-ttu-id="4af3c-691">További információt [az Azure Storage redundanciabeállításainál](http://msdn.microsoft.com/library/azure/dn727290.aspx) találhat.</span><span class="sxs-lookup"><span data-stu-id="4af3c-691">See [Azure Storage Redundancy Options](http://msdn.microsoft.com/library/azure/dn727290.aspx) for more information.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="4af3c-692">Szabályzatkonfiguráció</span><span class="sxs-lookup"><span data-stu-id="4af3c-692">Policy configuration</span></span>
<span data-ttu-id="4af3c-693">Az újrapróbálkozási szabályzatok konfigurálása szoftveresen történik.</span><span class="sxs-lookup"><span data-stu-id="4af3c-693">Retry policies are configured programmatically.</span></span> <span data-ttu-id="4af3c-694">Megszokott eljárás egy **TableRequestOptions**, **BlobRequestOptions**, **FileRequestOptions** vagy **QueueRequestOptions** példány létrehozása és adatokkal történő feltöltése.</span><span class="sxs-lookup"><span data-stu-id="4af3c-694">A typical procedure is to create and populate a **TableRequestOptions**, **BlobRequestOptions**, **FileRequestOptions**, or **QueueRequestOptions** instance.</span></span>

```csharp
TableRequestOptions interactiveRequestOption = new TableRequestOptions()
{
  RetryPolicy = new LinearRetry(TimeSpan.FromMilliseconds(500), 3),
  // For Read-access geo-redundant storage, use PrimaryThenSecondary.
  // Otherwise set this to PrimaryOnly.
  LocationMode = LocationMode.PrimaryThenSecondary,
  // Maximum execution time based on the business use case. 
  MaximumExecutionTime = TimeSpan.FromSeconds(2)
};
```

<span data-ttu-id="4af3c-695">Beállíthat egy kérésbeállítási példányt az ügyfélen, és ezt követően az ügyfelet érintő műveletek az így megadott kérésbeállításokat fogják használni.</span><span class="sxs-lookup"><span data-stu-id="4af3c-695">The request options instance can then be set on the client, and all operations with the client will use the specified request options.</span></span>

```csharp
client.DefaultRequestOptions = interactiveRequestOption;
var stats = await client.GetServiceStatsAsync();
```

<span data-ttu-id="4af3c-696">Az ügyfél kérésbeállításainak felülbírálásához a kérésbeállítás-osztály egy adatokkal feltöltött példányát kell megadni paraméterként a műveleti metódusoknál.</span><span class="sxs-lookup"><span data-stu-id="4af3c-696">You can override the client request options by passing a populated instance of the request options class as a parameter to operation methods.</span></span>

```csharp
var stats = await client.GetServiceStatsAsync(interactiveRequestOption, operationContext: null);
```

<span data-ttu-id="4af3c-697">Az **OperationContext** példányban határozhatja meg, hogy milyen kódot kell végrehajtani újrapróbálkozás esetén, illetve ha befejeződik a művelet.</span><span class="sxs-lookup"><span data-stu-id="4af3c-697">You use an **OperationContext** instance to specify the code to execute when a retry occurs and when an operation has completed.</span></span> <span data-ttu-id="4af3c-698">Ez a kód gyűjti össze a művelettel kapcsolatos adatokat, amelyek bekerülnek a naplókba és a telemetriába.</span><span class="sxs-lookup"><span data-stu-id="4af3c-698">This code can collect information about the operation for use in logs and telemetry.</span></span>

```csharp
// Set up notifications for an operation
var context = new OperationContext();
context.ClientRequestID = "some request id";
context.Retrying += (sender, args) =>
{
    /* Collect retry information */
};
context.RequestCompleted += (sender, args) =>
{
    /* Collect operation completion information */
};
var stats = await client.GetServiceStatsAsync(null, context);
```

<span data-ttu-id="4af3c-699">A kiterjesztett újrapróbálkozási szabályzatok jelzik, hogy egy adott hiba után szükség van-e újrapróbálkozásra, ezenkívül egy **RetryContext** objektumot adnak vissza, amely megmutatja az újrapróbálkozások számát, a legutóbbi kérés eredményét, illetve hogy a következő újrapróbálkozás az elsődleges vagy a másodlagos helyen fog történni (a részleteket lásd az alábbi táblázatban).</span><span class="sxs-lookup"><span data-stu-id="4af3c-699">In addition to indicating whether a failure is suitable for retry, the extended retry policies return a **RetryContext** object that indicates the number of retries, the results of the last request, whether the next retry will happen in the primary or secondary location (see table below for details).</span></span> <span data-ttu-id="4af3c-700">A **RetryContext** objektummal lehet meghatározni, hogy történjen-e újrapróbálkozás, és ha igen, mikor.</span><span class="sxs-lookup"><span data-stu-id="4af3c-700">The properties of the **RetryContext** object can be used to decide if and when to attempt a retry.</span></span> <span data-ttu-id="4af3c-701">További információért lásd az [IExtendedRetryPolicy.Evaluate metódus](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.evaluate.aspx) ismertetését.</span><span class="sxs-lookup"><span data-stu-id="4af3c-701">For more details, see [IExtendedRetryPolicy.Evaluate Method](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.evaluate.aspx).</span></span>

<span data-ttu-id="4af3c-702">A következő táblázatban a beépített újrapróbálkozási szabályzatok alapértelmezett beállításai tekinthetők meg.</span><span class="sxs-lookup"><span data-stu-id="4af3c-702">The following tables show the default settings for the built-in retry policies.</span></span>

<span data-ttu-id="4af3c-703">**Kérésbeállítások**</span><span class="sxs-lookup"><span data-stu-id="4af3c-703">**Request options**</span></span>

| <span data-ttu-id="4af3c-704">**Beállítás**</span><span class="sxs-lookup"><span data-stu-id="4af3c-704">**Setting**</span></span> | <span data-ttu-id="4af3c-705">**Alapértelmezett érték**</span><span class="sxs-lookup"><span data-stu-id="4af3c-705">**Default value**</span></span> | <span data-ttu-id="4af3c-706">**Jelentés**</span><span class="sxs-lookup"><span data-stu-id="4af3c-706">**Meaning**</span></span> |
| --- | --- | --- |
| <span data-ttu-id="4af3c-707">MaximumExecutionTime</span><span class="sxs-lookup"><span data-stu-id="4af3c-707">MaximumExecutionTime</span></span> | <span data-ttu-id="4af3c-708">None</span><span class="sxs-lookup"><span data-stu-id="4af3c-708">None</span></span> | <span data-ttu-id="4af3c-709">A kérés maximális végrehajtási ideje az esetleges újrapróbálkozási kísérletekkel együtt.</span><span class="sxs-lookup"><span data-stu-id="4af3c-709">Maximum execution time for the request, including all potential retry attempts.</span></span> <span data-ttu-id="4af3c-710">Ha nincs megadva, az, hogy mennyi ideig megengedett kérés esetében korlátlan.</span><span class="sxs-lookup"><span data-stu-id="4af3c-710">If it is not specified, then the amount of time that a request is permitted to take is unlimited.</span></span> <span data-ttu-id="4af3c-711">Más szóval a kérelem előfordulhat, hogy lefagy.</span><span class="sxs-lookup"><span data-stu-id="4af3c-711">In other words, the request might hang.</span></span> |
| <span data-ttu-id="4af3c-712">ServerTimeout</span><span class="sxs-lookup"><span data-stu-id="4af3c-712">ServerTimeout</span></span> | <span data-ttu-id="4af3c-713">None</span><span class="sxs-lookup"><span data-stu-id="4af3c-713">None</span></span> | <span data-ttu-id="4af3c-714">Kérés időtúllépési kerete a kiszolgálón (egész másodpercre kerekített érték).</span><span class="sxs-lookup"><span data-stu-id="4af3c-714">Server timeout interval for the request (value is rounded to seconds).</span></span> <span data-ttu-id="4af3c-715">Ha nincs megadva, a kiszolgálóhoz érkező összes kérés esetében az alapértelmezett értéket fogja használni.</span><span class="sxs-lookup"><span data-stu-id="4af3c-715">If not specified, it will use the default value for all requests to the server.</span></span> <span data-ttu-id="4af3c-716">Általában érdemes üresen hagyni ezt az értéket, hogy a kiszolgáló az alapértelmezett beállítást használja.</span><span class="sxs-lookup"><span data-stu-id="4af3c-716">Usually, the best option is to omit this setting so that the server default is used.</span></span> | 
| <span data-ttu-id="4af3c-717">LocationMode</span><span class="sxs-lookup"><span data-stu-id="4af3c-717">LocationMode</span></span> | <span data-ttu-id="4af3c-718">None</span><span class="sxs-lookup"><span data-stu-id="4af3c-718">None</span></span> | <span data-ttu-id="4af3c-719">Ha a tárfiókot az írásvédett georedundáns tároló (RA-GRS) replikációs beállítással hozza létre, a hely módja határozza meg, hogy melyik hely kapja meg a kérést.</span><span class="sxs-lookup"><span data-stu-id="4af3c-719">If the storage account is created with the Read access geo-redundant storage (RA-GRS) replication option, you can use the location mode to indicate which location should receive the request.</span></span> <span data-ttu-id="4af3c-720">A **PrimaryThenSecondary** érték esetében például a kérések először mindig az elsődleges helyre érkeznek.</span><span class="sxs-lookup"><span data-stu-id="4af3c-720">For example, if **PrimaryThenSecondary** is specified, requests are always sent to the primary location first.</span></span> <span data-ttu-id="4af3c-721">Ha a kérés sikertelen, ezután a másodlagos hely kapja meg azt.</span><span class="sxs-lookup"><span data-stu-id="4af3c-721">If a request fails, it is sent to the secondary location.</span></span> |
| <span data-ttu-id="4af3c-722">RetryPolicy</span><span class="sxs-lookup"><span data-stu-id="4af3c-722">RetryPolicy</span></span> | <span data-ttu-id="4af3c-723">ExponentialPolicy</span><span class="sxs-lookup"><span data-stu-id="4af3c-723">ExponentialPolicy</span></span> | <span data-ttu-id="4af3c-724">Az egyes beállítások részleteit az alábbiakban tekintheti meg.</span><span class="sxs-lookup"><span data-stu-id="4af3c-724">See below for details of each option.</span></span> |

<span data-ttu-id="4af3c-725">**Exponenciális szabályzat**</span><span class="sxs-lookup"><span data-stu-id="4af3c-725">**Exponential policy**</span></span> 

| <span data-ttu-id="4af3c-726">**Beállítás**</span><span class="sxs-lookup"><span data-stu-id="4af3c-726">**Setting**</span></span> | <span data-ttu-id="4af3c-727">**Alapértelmezett érték**</span><span class="sxs-lookup"><span data-stu-id="4af3c-727">**Default value**</span></span> | <span data-ttu-id="4af3c-728">**Jelentés**</span><span class="sxs-lookup"><span data-stu-id="4af3c-728">**Meaning**</span></span> |
| --- | --- | --- |
| <span data-ttu-id="4af3c-729">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="4af3c-729">maxAttempt</span></span> | <span data-ttu-id="4af3c-730">3</span><span class="sxs-lookup"><span data-stu-id="4af3c-730">3</span></span> | <span data-ttu-id="4af3c-731">Az újrapróbálkozási kísérletek száma.</span><span class="sxs-lookup"><span data-stu-id="4af3c-731">Number of retry attempts.</span></span> |
| <span data-ttu-id="4af3c-732">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="4af3c-732">deltaBackoff</span></span> | <span data-ttu-id="4af3c-733">4 másodperc</span><span class="sxs-lookup"><span data-stu-id="4af3c-733">4 seconds</span></span> | <span data-ttu-id="4af3c-734">Az újrapróbálkozások közötti visszatartási időköz.</span><span class="sxs-lookup"><span data-stu-id="4af3c-734">Back-off interval between retries.</span></span> <span data-ttu-id="4af3c-735">Az ezt követő újrapróbálkozási kísérletek esetében az időtartomány többszörösét (egy véletlenszerű elemet belevéve) fogja használni a rendszer.</span><span class="sxs-lookup"><span data-stu-id="4af3c-735">Multiples of this timespan, including a random element, will be used for subsequent retry attempts.</span></span> |
| <span data-ttu-id="4af3c-736">MinBackoff</span><span class="sxs-lookup"><span data-stu-id="4af3c-736">MinBackoff</span></span> | <span data-ttu-id="4af3c-737">3 másodperc</span><span class="sxs-lookup"><span data-stu-id="4af3c-737">3 seconds</span></span> | <span data-ttu-id="4af3c-738">Ez az érték hozzáadódik a deltaBackoff alapján kiszámított minden újrapróbálkozási időközhöz.</span><span class="sxs-lookup"><span data-stu-id="4af3c-738">Added to all retry intervals computed from deltaBackoff.</span></span> <span data-ttu-id="4af3c-739">Ez az érték nem módosítható.</span><span class="sxs-lookup"><span data-stu-id="4af3c-739">This value cannot be changed.</span></span>
| <span data-ttu-id="4af3c-740">MaxBackoff</span><span class="sxs-lookup"><span data-stu-id="4af3c-740">MaxBackoff</span></span> | <span data-ttu-id="4af3c-741">120 másodperc</span><span class="sxs-lookup"><span data-stu-id="4af3c-741">120 seconds</span></span> | <span data-ttu-id="4af3c-742">A MaxBackoff akkor lép működésbe, ha a kiszámított újrapróbálkozási időköz nagyobb, mint a MaxBackoff értéke.</span><span class="sxs-lookup"><span data-stu-id="4af3c-742">MaxBackoff is used if the computed retry interval is greater than MaxBackoff.</span></span> <span data-ttu-id="4af3c-743">Ez az érték nem módosítható.</span><span class="sxs-lookup"><span data-stu-id="4af3c-743">This value cannot be changed.</span></span> |

<span data-ttu-id="4af3c-744">**Lineáris szabályzat**</span><span class="sxs-lookup"><span data-stu-id="4af3c-744">**Linear policy**</span></span>

| <span data-ttu-id="4af3c-745">**Beállítás**</span><span class="sxs-lookup"><span data-stu-id="4af3c-745">**Setting**</span></span> | <span data-ttu-id="4af3c-746">**Alapértelmezett érték**</span><span class="sxs-lookup"><span data-stu-id="4af3c-746">**Default value**</span></span> | <span data-ttu-id="4af3c-747">**Jelentés**</span><span class="sxs-lookup"><span data-stu-id="4af3c-747">**Meaning**</span></span> |
| --- | --- | --- |
| <span data-ttu-id="4af3c-748">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="4af3c-748">maxAttempt</span></span> | <span data-ttu-id="4af3c-749">3</span><span class="sxs-lookup"><span data-stu-id="4af3c-749">3</span></span> | <span data-ttu-id="4af3c-750">Az újrapróbálkozási kísérletek száma.</span><span class="sxs-lookup"><span data-stu-id="4af3c-750">Number of retry attempts.</span></span> |
| <span data-ttu-id="4af3c-751">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="4af3c-751">deltaBackoff</span></span> | <span data-ttu-id="4af3c-752">30 másodperc</span><span class="sxs-lookup"><span data-stu-id="4af3c-752">30 seconds</span></span> | <span data-ttu-id="4af3c-753">Az újrapróbálkozások közötti visszatartási időköz.</span><span class="sxs-lookup"><span data-stu-id="4af3c-753">Back-off interval between retries.</span></span> |

### <a name="retry-usage-guidance"></a><span data-ttu-id="4af3c-754">Újrapróbálkozásokra vonatkozó útmutató</span><span class="sxs-lookup"><span data-stu-id="4af3c-754">Retry usage guidance</span></span>
<span data-ttu-id="4af3c-755">Ügyeljen a következőkre, amikor a Storage-ügyfél API-ján keresztül próbálja elérni az Azure Storage-szolgáltatásokat:</span><span class="sxs-lookup"><span data-stu-id="4af3c-755">Consider the following guidelines when accessing Azure storage services using the storage client API:</span></span>

* <span data-ttu-id="4af3c-756">Használja a Microsoft.WindowsAzure.Storage.RetryPolicies névtérben található beépített újrapróbálkozási szabályzatokat, amennyiben azok megfelelnek az elvárásainak.</span><span class="sxs-lookup"><span data-stu-id="4af3c-756">Use the built-in retry policies from the Microsoft.WindowsAzure.Storage.RetryPolicies namespace where they are appropriate for your requirements.</span></span> <span data-ttu-id="4af3c-757">A legtöbb esetben ezek a szabályzatok is elégségesek.</span><span class="sxs-lookup"><span data-stu-id="4af3c-757">In most cases, these policies will be sufficient.</span></span>
* <span data-ttu-id="4af3c-758">Az **ExponentialRetry** szabályzatot kötegelt műveletek, háttérfeladatok vagy nem interaktív forgatókönyvek esetében használja.</span><span class="sxs-lookup"><span data-stu-id="4af3c-758">Use the **ExponentialRetry** policy in batch operations, background tasks, or non-interactive scenarios.</span></span> <span data-ttu-id="4af3c-759">Ezekben az esetekben általában több idő áll rendelkezésre megvárni, amíg a szolgáltatás helyreáll, ami növeli a művelet sikeres teljesítésének esélyét.</span><span class="sxs-lookup"><span data-stu-id="4af3c-759">In these scenarios, you can typically allow more time for the service to recover—with a consequently increased chance of the operation eventually succeeding.</span></span>
* <span data-ttu-id="4af3c-760">Érdemes megadni a **RequestOptions** paraméter **MaximumExecutionTime** tulajdonságát a teljes végrehajtási idő korlátozásához, de az időtúllépési érték megadásakor vegye figyelembe a művelet típusát és méretét.</span><span class="sxs-lookup"><span data-stu-id="4af3c-760">Consider specifying the **MaximumExecutionTime** property of the **RequestOptions** parameter to limit the total execution time, but take into account the type and size of the operation when choosing a timeout value.</span></span>
* <span data-ttu-id="4af3c-761">Ha egyéni újrapróbálkozást implementál, ne hozzon létre burkolókat a Storage-ügyfél osztályai körül.</span><span class="sxs-lookup"><span data-stu-id="4af3c-761">If you need to implement a custom retry, avoid creating wrappers around the storage client classes.</span></span> <span data-ttu-id="4af3c-762">Inkább meglévő szabályzatokat bővítsen ki az **IExtendedRetryPolicy** felületen keresztül.</span><span class="sxs-lookup"><span data-stu-id="4af3c-762">Instead, use the capabilities to extend the existing policies through the **IExtendedRetryPolicy** interface.</span></span>
* <span data-ttu-id="4af3c-763">Amennyiben írásvédett georedundáns tárolót (RA-GRS) használ, a **LocationMode** segítségével beállíthatja, hogy az elsődleges hozzáférés sikertelensége esetén az újrapróbálkozási kísérletek a tároló másodlagos írásvédett példányához férjenek hozzá.</span><span class="sxs-lookup"><span data-stu-id="4af3c-763">If you are using read access geo-redundant storage (RA-GRS) you can use the **LocationMode** to specify that retry attempts will access the secondary read-only copy of the store should the primary access fail.</span></span> <span data-ttu-id="4af3c-764">Ebben az esetben azonban meg kell győződnie arról, hogy az alkalmazás esetlegesen elavult adatokkal is képes sikeresen működni (ha az elsődleges tároló replikálása még nem fejeződött be).</span><span class="sxs-lookup"><span data-stu-id="4af3c-764">However, when using this option you must ensure that your application can work successfully with data that may be stale if the replication from the primary store has not yet completed.</span></span>

<span data-ttu-id="4af3c-765">A következő beállításokat javasoljuk az újrapróbálkozási műveletekhez.</span><span class="sxs-lookup"><span data-stu-id="4af3c-765">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="4af3c-766">Ezek általános célú beállítások, ezért javasoljuk, hogy monitorozza műveleteit, és finomhangolja az értékeket saját igényei szerint.</span><span class="sxs-lookup"><span data-stu-id="4af3c-766">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>  

| <span data-ttu-id="4af3c-767">**Környezet**</span><span class="sxs-lookup"><span data-stu-id="4af3c-767">**Context**</span></span> | <span data-ttu-id="4af3c-768">**Mintacél E2E<br />max. késleltetése**</span><span class="sxs-lookup"><span data-stu-id="4af3c-768">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="4af3c-769">**Újrapróbálkozási szabályzat**</span><span class="sxs-lookup"><span data-stu-id="4af3c-769">**Retry policy**</span></span> | <span data-ttu-id="4af3c-770">**Beállítások**</span><span class="sxs-lookup"><span data-stu-id="4af3c-770">**Settings**</span></span> | <span data-ttu-id="4af3c-771">**Értékek**</span><span class="sxs-lookup"><span data-stu-id="4af3c-771">**Values**</span></span> | <span data-ttu-id="4af3c-772">**Működési elv**</span><span class="sxs-lookup"><span data-stu-id="4af3c-772">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="4af3c-773">Interaktív, felhasználói felület</span><span class="sxs-lookup"><span data-stu-id="4af3c-773">Interactive, UI,</span></span><br /><span data-ttu-id="4af3c-774">vagy előtér</span><span class="sxs-lookup"><span data-stu-id="4af3c-774">or foreground</span></span> |<span data-ttu-id="4af3c-775">2 másodperc</span><span class="sxs-lookup"><span data-stu-id="4af3c-775">2 seconds</span></span> |<span data-ttu-id="4af3c-776">Lineáris</span><span class="sxs-lookup"><span data-stu-id="4af3c-776">Linear</span></span> |<span data-ttu-id="4af3c-777">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="4af3c-777">maxAttempt</span></span><br /><span data-ttu-id="4af3c-778">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="4af3c-778">deltaBackoff</span></span> |<span data-ttu-id="4af3c-779">3</span><span class="sxs-lookup"><span data-stu-id="4af3c-779">3</span></span><br /><span data-ttu-id="4af3c-780">500 ms</span><span class="sxs-lookup"><span data-stu-id="4af3c-780">500 ms</span></span> |<span data-ttu-id="4af3c-781">1. kísérlet – 500 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="4af3c-781">Attempt 1 - delay 500 ms</span></span><br /><span data-ttu-id="4af3c-782">2. kísérlet – 500 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="4af3c-782">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="4af3c-783">3. kísérlet – 500 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="4af3c-783">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="4af3c-784">Háttér</span><span class="sxs-lookup"><span data-stu-id="4af3c-784">Background</span></span><br /><span data-ttu-id="4af3c-785">vagy kötegelt</span><span class="sxs-lookup"><span data-stu-id="4af3c-785">or batch</span></span> |<span data-ttu-id="4af3c-786">30 másodperc</span><span class="sxs-lookup"><span data-stu-id="4af3c-786">30 seconds</span></span> |<span data-ttu-id="4af3c-787">Exponenciális</span><span class="sxs-lookup"><span data-stu-id="4af3c-787">Exponential</span></span> |<span data-ttu-id="4af3c-788">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="4af3c-788">maxAttempt</span></span><br /><span data-ttu-id="4af3c-789">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="4af3c-789">deltaBackoff</span></span> |<span data-ttu-id="4af3c-790">5</span><span class="sxs-lookup"><span data-stu-id="4af3c-790">5</span></span><br /><span data-ttu-id="4af3c-791">4 másodperc</span><span class="sxs-lookup"><span data-stu-id="4af3c-791">4 seconds</span></span> |<span data-ttu-id="4af3c-792">1. kísérlet – kb. 3 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="4af3c-792">Attempt 1 - delay ~3 sec</span></span><br /><span data-ttu-id="4af3c-793">2. kísérlet – kb. 7 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="4af3c-793">Attempt 2 - delay ~7 sec</span></span><br /><span data-ttu-id="4af3c-794">3. kísérlet – kb. 15 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="4af3c-794">Attempt 3 - delay ~15 sec</span></span> |

### <a name="telemetry"></a><span data-ttu-id="4af3c-795">Telemetria</span><span class="sxs-lookup"><span data-stu-id="4af3c-795">Telemetry</span></span>
<span data-ttu-id="4af3c-796">Az újrapróbálkozási kísérletek a **TraceSource** osztályban lesznek naplózva.</span><span class="sxs-lookup"><span data-stu-id="4af3c-796">Retry attempts are logged to a **TraceSource**.</span></span> <span data-ttu-id="4af3c-797">Az események rögzítéséhez és megfelelő célnaplóba való írásához egy **TraceListener** osztályt kell konfigurálnia.</span><span class="sxs-lookup"><span data-stu-id="4af3c-797">You must configure a **TraceListener** to capture the events and write them to a suitable destination log.</span></span> <span data-ttu-id="4af3c-798">Használja a **TextWriterTraceListener** vagy az **XmlWriterTraceListener** osztályt az adatok a naplófájlba írásához. Az **EventLogTraceListener** osztállyal a Windows eseménynaplóba írhat, míg az **EventProviderTraceListener** osztállyal a nyomkövetési adatokat rögzítheti az ETW alrendszerben.</span><span class="sxs-lookup"><span data-stu-id="4af3c-798">You can use the **TextWriterTraceListener** or **XmlWriterTraceListener** to write the data to a log file, the **EventLogTraceListener** to write to the Windows Event Log, or the **EventProviderTraceListener** to write trace data to the ETW subsystem.</span></span> <span data-ttu-id="4af3c-799">Beállíthatja, hogy a puffer ürítése automatikusan megtörténjen, valamint a naplózott események részletességét is (például: Hiba, Figyelmeztetés, Tájékoztató és Részletes).</span><span class="sxs-lookup"><span data-stu-id="4af3c-799">You can also configure auto-flushing of the buffer, and the verbosity of events that will be logged (for example, Error, Warning, Informational, and Verbose).</span></span> <span data-ttu-id="4af3c-800">További információt a [.NET-es Storage-ügyfélkódtárral történő ügyféloldali naplózást](http://msdn.microsoft.com/library/azure/dn782839.aspx) ismertető szakaszban találhat.</span><span class="sxs-lookup"><span data-stu-id="4af3c-800">For more information, see [Client-side Logging with the .NET Storage Client Library](http://msdn.microsoft.com/library/azure/dn782839.aspx).</span></span>

<span data-ttu-id="4af3c-801">A műveletek kaphatnak egy **OperationContext**-példányt, amely közzétesz egy **Retrying** eseményt, amelyhez egyéni telemetrialogika csatolható.</span><span class="sxs-lookup"><span data-stu-id="4af3c-801">Operations can receive an **OperationContext** instance, which exposes a **Retrying** event that can be used to attach custom telemetry logic.</span></span> <span data-ttu-id="4af3c-802">További információ: [OperationContext.Retrying Event](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.operationcontext.retrying.aspx).</span><span class="sxs-lookup"><span data-stu-id="4af3c-802">For more information, see [OperationContext.Retrying Event](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.operationcontext.retrying.aspx).</span></span>

### <a name="examples"></a><span data-ttu-id="4af3c-803">Példák</span><span class="sxs-lookup"><span data-stu-id="4af3c-803">Examples</span></span>
<span data-ttu-id="4af3c-804">A következő mintakód azt mutatja be, hogyan lehet két **TableRequestOptions**-példányt létrehozni eltérő újrapróbálkozási beállításokkal. Az egyik az interaktív, a másik pedig a háttérkérések kezelésére szolgál.</span><span class="sxs-lookup"><span data-stu-id="4af3c-804">The following code example shows how to create two **TableRequestOptions** instances with different retry settings; one for interactive requests and one for background requests.</span></span> <span data-ttu-id="4af3c-805">A példa ezután az ügyfélre alkalmazza ezt a két újrapróbálkozási szabályzatot, így azok minden kérésre vonatkozni fognak, ezenkívül az interaktív stratégiát egy adott kéréshez köti, így az felülbírálja az ügyfél alapértelmezett beállításait.</span><span class="sxs-lookup"><span data-stu-id="4af3c-805">The example then sets these two retry policies on the client so that they apply for all requests, and also sets the interactive strategy on a specific request so that it overrides the default settings applied to the client.</span></span>

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.RetryPolicies;
using Microsoft.WindowsAzure.Storage.Table;

namespace RetryCodeSamples
{
    class AzureStorageCodeSamples
    {
        private const string connectionString = "UseDevelopmentStorage=true";

        public async static Task Samples()
        {
            var storageAccount = CloudStorageAccount.Parse(connectionString);

            TableRequestOptions interactiveRequestOption = new TableRequestOptions()
            {
                RetryPolicy = new LinearRetry(TimeSpan.FromMilliseconds(500), 3),
                // For Read-access geo-redundant storage, use PrimaryThenSecondary.
                // Otherwise set this to PrimaryOnly.
                LocationMode = LocationMode.PrimaryThenSecondary,
                // Maximum execution time based on the business use case. 
                MaximumExecutionTime = TimeSpan.FromSeconds(2)
            };

            TableRequestOptions backgroundRequestOption = new TableRequestOptions()
            {
                // Client has a default exponential retry policy with 4 sec delay and 3 retry attempts
                // Retry delays will be approximately 3 sec, 7 sec, and 15 sec
                MaximumExecutionTime = TimeSpan.FromSeconds(30),
                // PrimaryThenSecondary in case of Read-access geo-redundant storage, else set this to PrimaryOnly
                LocationMode = LocationMode.PrimaryThenSecondary
            };

            var client = storageAccount.CreateCloudTableClient();
            // Client has a default exponential retry policy with 4 sec delay and 3 retry attempts
            // Retry delays will be approximately 3 sec, 7 sec, and 15 sec
            // ServerTimeout and MaximumExecutionTime are not set

            {
                // Set properties for the client (used on all requests unless overridden)
                // Different exponential policy parameters for background scenarios
                client.DefaultRequestOptions = backgroundRequestOption;
                // Linear policy for interactive scenarios
                client.DefaultRequestOptions = interactiveRequestOption;
            }

            {
                // set properties for a specific request
                var stats = await client.GetServiceStatsAsync(interactiveRequestOption, operationContext: null);
            }

            {
                // Set up notifications for an operation
                var context = new OperationContext();
                context.ClientRequestID = "some request id";
                context.Retrying += (sender, args) =>
                {
                    /* Collect retry information */
                };
                context.RequestCompleted += (sender, args) =>
                {
                    /* Collect operation completion information */
                };
                var stats = await client.GetServiceStatsAsync(null, context);
            }
        }
    }
}
```

### <a name="more-information"></a><span data-ttu-id="4af3c-806">További információ</span><span class="sxs-lookup"><span data-stu-id="4af3c-806">More information</span></span>
* [<span data-ttu-id="4af3c-807">Az Azure Storage ügyféloldali kódtárhoz javasolt újrapróbálkozási szabályzatok</span><span class="sxs-lookup"><span data-stu-id="4af3c-807">Azure Storage Client Library Retry Policy Recommendations</span></span>](https://azure.microsoft.com/blog/2014/05/22/azure-storage-client-library-retry-policy-recommendations/)
* [<span data-ttu-id="4af3c-808">Storage ügyféloldali kódtár 2.0 – újrapróbálkozási szabályzatok implementálása</span><span class="sxs-lookup"><span data-stu-id="4af3c-808">Storage Client Library 2.0 – Implementing Retry Policies</span></span>](http://gauravmantri.com/2012/12/30/storage-client-library-2-0-implementing-retry-policies/)

## <a name="general-rest-and-retry-guidelines"></a><span data-ttu-id="4af3c-809">Általános REST- és újrapróbálkozási irányelvek</span><span class="sxs-lookup"><span data-stu-id="4af3c-809">General REST and retry guidelines</span></span>
<span data-ttu-id="4af3c-810">Vegye figyelembe a következőket, amikor az Azure-t vagy külső szolgáltatást próbál elérni:</span><span class="sxs-lookup"><span data-stu-id="4af3c-810">Consider the following when accessing Azure or third party services:</span></span>

* <span data-ttu-id="4af3c-811">Alkalmazzon rendszerszintű megközelítést az újrapróbálkozások kezelésére, például egy újrafelhasználható kód formájában, hogy minden ügyfél és megoldás esetében ugyanaz a módszertan érvényesüljön.</span><span class="sxs-lookup"><span data-stu-id="4af3c-811">Use a systematic approach to managing retries, perhaps as reusable code, so that you can apply a consistent methodology across all clients and all solutions.</span></span>
* <span data-ttu-id="4af3c-812">Fontolja meg, mint például egy újrapróbálkozási keretrendszert használni [Polly] [ polly] kezelheti az újrapróbálkozásokat, ha a célszolgáltatás vagy -ügyfél nem rendelkezik beépített újrapróbálkozási mechanizmus.</span><span class="sxs-lookup"><span data-stu-id="4af3c-812">Consider using a retry framework such as [Polly][polly] to manage retries if the target service or client has no built-in retry mechanism.</span></span> <span data-ttu-id="4af3c-813">Így konzisztens újrapróbálkozási viselkedés valósítható meg, és megfelelő alapértelmezett újrapróbálkozási stratégiát tehet elérhetővé a célszolgáltatás számára.</span><span class="sxs-lookup"><span data-stu-id="4af3c-813">This will help you implement a consistent retry behavior, and it may provide a suitable default retry strategy for the target service.</span></span> <span data-ttu-id="4af3c-814">Ugyanakkor szükség lehet egyéni újrapróbálkozási kód kialakítására a nem szabványos viselkedésű szolgáltatások esetében, amelyek nem kivételek alapján állapítják meg az átmeneti hibákat, illetve ha **Retry-Response** válasszal kívánja kezelni az újrapróbálkozási viselkedést.</span><span class="sxs-lookup"><span data-stu-id="4af3c-814">However, you may need to create custom retry code for services that have non-standard behavior, that do not rely on exceptions to indicate transient failures, or if you want to use a **Retry-Response** reply to manage retry behavior.</span></span>
* <span data-ttu-id="4af3c-815">Az átmeneti hibák észlelési logikája a REST-hívások végrehajtására ténylegesen használt ügyfél-API-tól függ.</span><span class="sxs-lookup"><span data-stu-id="4af3c-815">The transient detection logic will depend on the actual client API you use to invoke the REST calls.</span></span> <span data-ttu-id="4af3c-816">Egyes ügyfelek, például az újabb **HttpClient** osztály, nem váltanak ki kivételeket, ha a kérés befejeződik, de nem sikert jelző HTTP-állapotkódot ad vissza.</span><span class="sxs-lookup"><span data-stu-id="4af3c-816">Some clients, such as the newer **HttpClient** class, will not throw exceptions for completed requests with a non-success HTTP status code.</span></span> 
* <span data-ttu-id="4af3c-817">A szolgáltatás által visszaadott HTTP-állapotkód segíthet megállapítani, hogy a hiba átmeneti jellegű-e.</span><span class="sxs-lookup"><span data-stu-id="4af3c-817">The HTTP status code returned from the service can help to indicate whether the failure is transient.</span></span> <span data-ttu-id="4af3c-818">Lehet, hogy az állapotkód vagy az azzal egyenértékű kivételtípus megismeréséhez meg kell vizsgálnia az ügyfél vagy az újrapróbálkozási keretrendszer által előállított kivételeket.</span><span class="sxs-lookup"><span data-stu-id="4af3c-818">You may need to examine the exceptions generated by a client or the retry framework to access the status code or to determine the equivalent exception type.</span></span> <span data-ttu-id="4af3c-819">A következő HTTP-kódok általában azt jelzik, hogy érdemes újrapróbálkozni:</span><span class="sxs-lookup"><span data-stu-id="4af3c-819">The following HTTP codes typically indicate that a retry is appropriate:</span></span>
  * <span data-ttu-id="4af3c-820">408 Kérés időtúllépése</span><span class="sxs-lookup"><span data-stu-id="4af3c-820">408 Request Timeout</span></span>
  * <span data-ttu-id="4af3c-821">429 túl sok kérelem</span><span class="sxs-lookup"><span data-stu-id="4af3c-821">429 Too Many Requests</span></span>
  * <span data-ttu-id="4af3c-822">500 Belső kiszolgálóhiba</span><span class="sxs-lookup"><span data-stu-id="4af3c-822">500 Internal Server Error</span></span>
  * <span data-ttu-id="4af3c-823">502 Hibás átjáró</span><span class="sxs-lookup"><span data-stu-id="4af3c-823">502 Bad Gateway</span></span>
  * <span data-ttu-id="4af3c-824">503 A szolgáltatás nem érhető el</span><span class="sxs-lookup"><span data-stu-id="4af3c-824">503 Service Unavailable</span></span>
  * <span data-ttu-id="4af3c-825">504 Időtúllépés az átjárón</span><span class="sxs-lookup"><span data-stu-id="4af3c-825">504 Gateway Timeout</span></span>
* <span data-ttu-id="4af3c-826">Amennyiben kivételekre épül az újrapróbálkozási logikája, a következők általában a csatlakozást meghiúsító átmeneti hibát jeleznek:</span><span class="sxs-lookup"><span data-stu-id="4af3c-826">If you base your retry logic on exceptions, the following typically indicate a transient failure where no connection could be established:</span></span>
  * <span data-ttu-id="4af3c-827">WebExceptionStatus.ConnectionClosed</span><span class="sxs-lookup"><span data-stu-id="4af3c-827">WebExceptionStatus.ConnectionClosed</span></span>
  * <span data-ttu-id="4af3c-828">WebExceptionStatus.ConnectFailure</span><span class="sxs-lookup"><span data-stu-id="4af3c-828">WebExceptionStatus.ConnectFailure</span></span>
  * <span data-ttu-id="4af3c-829">WebExceptionStatus.Timeout</span><span class="sxs-lookup"><span data-stu-id="4af3c-829">WebExceptionStatus.Timeout</span></span>
  * <span data-ttu-id="4af3c-830">WebExceptionStatus.RequestCanceled</span><span class="sxs-lookup"><span data-stu-id="4af3c-830">WebExceptionStatus.RequestCanceled</span></span>
* <span data-ttu-id="4af3c-831">Amennyiben egy szolgáltatás állapota „Nem érhető el”, előfordulhat, hogy a szolgáltatás a megfelelő késleltetést jelzi, mielőtt újrapróbálkozna a **Retry-After** válaszfejlécben vagy egy másik egyéni fejlécben.</span><span class="sxs-lookup"><span data-stu-id="4af3c-831">In the case of a service unavailable status, the service might indicate the appropriate delay before retrying in the **Retry-After** response header or a different custom header.</span></span> <span data-ttu-id="4af3c-832">A szolgáltatások további információkat küldhetnek egyéni fejlécként, vagy a válasz tartalmába beágyazva.</span><span class="sxs-lookup"><span data-stu-id="4af3c-832">Services might also send additional information as custom headers, or embedded in the content of the response.</span></span> 
* <span data-ttu-id="4af3c-833">Ne próbálkozzon újra, ha az állapotkód ügyfélhibát jelez (4xx-es tartomány). E szabály alól csak a 408-as Kérés időtúllépése képez kivételt.</span><span class="sxs-lookup"><span data-stu-id="4af3c-833">Do not retry for status codes representing client errors (errors in the 4xx range) except for a 408 Request Timeout.</span></span>
* <span data-ttu-id="4af3c-834">Tesztelje le alaposan az újrapróbálkozási stratégiáit és mechanizmusait különböző feltételek mellett, például különböző hálózati állapotok és rendszerterhelések esetén.</span><span class="sxs-lookup"><span data-stu-id="4af3c-834">Thoroughly test your retry strategies and mechanisms under a range of conditions, such as different network states and varying system loadings.</span></span>

### <a name="retry-strategies"></a><span data-ttu-id="4af3c-835">Újrapróbálkozási stratégiák</span><span class="sxs-lookup"><span data-stu-id="4af3c-835">Retry strategies</span></span>
<span data-ttu-id="4af3c-836">A következők az újrapróbálkozási stratégiák időközeinek jellemző típusai:</span><span class="sxs-lookup"><span data-stu-id="4af3c-836">The following are the typical types of retry strategy intervals:</span></span>

* <span data-ttu-id="4af3c-837">**Exponenciális**.</span><span class="sxs-lookup"><span data-stu-id="4af3c-837">**Exponential**.</span></span> <span data-ttu-id="4af3c-838">Egy újrapróbálkozási szabályzatot, amely végrehajtja a megadott számú újrapróbálkozást, véletlenszerű exponenciális visszatartási megközelítéssel megállapítsa az újrapróbálkozások között.</span><span class="sxs-lookup"><span data-stu-id="4af3c-838">A retry policy that performs a specified number of retries, using a randomized exponential back off approach to determine the interval between retries.</span></span> <span data-ttu-id="4af3c-839">Példa:</span><span class="sxs-lookup"><span data-stu-id="4af3c-839">For example:</span></span>

    ```csharp
    var random = new Random();

    var delta = (int)((Math.Pow(2.0, currentRetryCount) - 1.0) *
                random.Next((int)(this.deltaBackoff.TotalMilliseconds * 0.8),
                (int)(this.deltaBackoff.TotalMilliseconds * 1.2)));
    var interval = (int)Math.Min(checked(this.minBackoff.TotalMilliseconds + delta),
                    this.maxBackoff.TotalMilliseconds);
    retryInterval = TimeSpan.FromMilliseconds(interval);
    ```

* <span data-ttu-id="4af3c-840">**Növekményes**.</span><span class="sxs-lookup"><span data-stu-id="4af3c-840">**Incremental**.</span></span> <span data-ttu-id="4af3c-841">Egy újrapróbálkozási stratégia adott számú újrapróbálkozási kísérletet és az újrapróbálkozások között eltelt időt.</span><span class="sxs-lookup"><span data-stu-id="4af3c-841">A retry strategy with a specified number of retry attempts and an incremental time interval between retries.</span></span> <span data-ttu-id="4af3c-842">Példa:</span><span class="sxs-lookup"><span data-stu-id="4af3c-842">For example:</span></span>

    ```csharp
    retryInterval = TimeSpan.FromMilliseconds(this.initialInterval.TotalMilliseconds +
                    (this.increment.TotalMilliseconds * currentRetryCount));
    ```

* <span data-ttu-id="4af3c-843">**Lineáris**.</span><span class="sxs-lookup"><span data-stu-id="4af3c-843">**LinearRetry**.</span></span> <span data-ttu-id="4af3c-844">Ez az újrapróbálkozási szabályzat adott számú újrapróbálkozást, egy meghatározott végző rögzített újrapróbálkozások között eltelt időt.</span><span class="sxs-lookup"><span data-stu-id="4af3c-844">A retry policy that performs a specified number of retries, using a specified fixed time interval between retries.</span></span> <span data-ttu-id="4af3c-845">Példa:</span><span class="sxs-lookup"><span data-stu-id="4af3c-845">For example:</span></span>

    ```csharp
    retryInterval = this.deltaBackoff;
    ```

### <a name="transient-fault-handling-with-polly"></a><span data-ttu-id="4af3c-846">Átmeneti hibák kezelése a Polly használatával</span><span class="sxs-lookup"><span data-stu-id="4af3c-846">Transient fault handling with Polly</span></span>
<span data-ttu-id="4af3c-847">[Polly] [ polly] van egy kódtár lehetővé teszi az újrapróbálkozások és [áramkör-megszakító] [ circuit-breaker] stratégiák.</span><span class="sxs-lookup"><span data-stu-id="4af3c-847">[Polly][polly] is a library to programatically handle retries and [circuit breaker][circuit-breaker] strategies.</span></span> <span data-ttu-id="4af3c-848">A Polly projekt a [.NET Foundation][dotnet-foundation] keretében érhető el.</span><span class="sxs-lookup"><span data-stu-id="4af3c-848">The Polly project is a member of the [.NET Foundation][dotnet-foundation].</span></span> <span data-ttu-id="4af3c-849">A Polly olyan szolgáltatások számára kínál alternatívát, amelyekben az ügyfél nem támogatja natív módon az újrapróbálkozásokat. Megszünteti az olyan egyéni újrapróbálkozási kódok írásának szükségét, amelyeket egyébként nehéz lenne megfelelően implementálni.</span><span class="sxs-lookup"><span data-stu-id="4af3c-849">For services where the client does not natively support retries, Polly is a valid alternative and avoids the need to write custom retry code, which can be hard to implement correctly.</span></span> <span data-ttu-id="4af3c-850">A Polly ezenkívül módot ad a hibák nyomon követésére, így naplózhatóvá teszi az újrapróbálkozásokat.</span><span class="sxs-lookup"><span data-stu-id="4af3c-850">Polly also provides a way to trace errors when they occur, so that you can log retries.</span></span>


<!-- links -->

[adal]: /azure/active-directory/develop/active-directory-authentication-libraries
[autorest]: https://github.com/Azure/autorest/tree/master/docs
[circuit-breaker]: ../patterns/circuit-breaker.md
[ConnectionPolicy.RetryOptions]: https://msdn.microsoft.com/library/azure/microsoft.azure.documents.client.connectionpolicy.retryoptions.aspx
[dotnet-foundation]: https://dotnetfoundation.org/
[polly]: http://www.thepollyproject.org
[redis-cache-troubleshoot]: /azure/redis-cache/cache-how-to-troubleshoot
[SearchIndexClient]: https://msdn.microsoft.com/library/azure/microsoft.azure.search.searchindexclient.aspx
[SearchServiceClient]: https://msdn.microsoft.com/library/microsoft.azure.search.searchserviceclient.aspx


### <a name="more-information"></a><span data-ttu-id="4af3c-854">További információ</span><span class="sxs-lookup"><span data-stu-id="4af3c-854">More information</span></span>
* [<span data-ttu-id="4af3c-855">Kapcsolat rugalmassága</span><span class="sxs-lookup"><span data-stu-id="4af3c-855">Connection Resiliency</span></span>](/ef/core/miscellaneous/connection-resiliency)
* [<span data-ttu-id="4af3c-856">Adatpontok – EF Core 1.1</span><span class="sxs-lookup"><span data-stu-id="4af3c-856">Data Points - EF Core 1.1</span></span>](https://msdn.microsoft.com/magazine/mt745093.aspx)


