---
title: Szolgáltatásspecifikus útmutató az újrapróbálkozási mechanizmushoz
description: Szolgáltatásspecifikus útmutató az újrapróbálkozási mechanizmus beállításához.
author: dragon119
ms.date: 07/13/2016
pnp.series.title: Best Practices
ms.openlocfilehash: f02843f179671da04bc2f09326b58075b432ba95
ms.sourcegitcommit: 85334ab0ccb072dac80de78aa82bcfa0f0044d3f
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 06/11/2018
ms.locfileid: "35253077"
---
# <a name="retry-guidance-for-specific-services"></a><span data-ttu-id="340b9-103">Újrapróbálkozási útmutatás adott szolgáltatásoknál</span><span class="sxs-lookup"><span data-stu-id="340b9-103">Retry guidance for specific services</span></span>

<span data-ttu-id="340b9-104">A legtöbb Azure-szolgáltatás és ügyfél-SDK tartalmaz egy újrapróbálkozási mechanizmust.</span><span class="sxs-lookup"><span data-stu-id="340b9-104">Most Azure services and client SDKs include a retry mechanism.</span></span> <span data-ttu-id="340b9-105">Ezek azonban eltérőek lehetnek, mert minden szolgáltatásnak eltérő jellemzői és követelményei vannak, így mindegyik újrapróbálkozási mechanizmus az adott szolgáltatásra van szabva.</span><span class="sxs-lookup"><span data-stu-id="340b9-105">However, these differ because each service has different characteristics and requirements, and so each retry mechanism is tuned to a specific service.</span></span> <span data-ttu-id="340b9-106">Ez az útmutató összefoglalást ad az Azure-szolgáltatások többségére jellemző újrapróbálkozási funkciókról, továbbá segítséget nyújt a szolgáltatás újrapróbálkozási mechanizmusának használatához, módosításához vagy kibővítéséhez.</span><span class="sxs-lookup"><span data-stu-id="340b9-106">This guide summarizes the retry mechanism features for the majority of Azure services, and includes information to help you use, adapt, or extend the retry mechanism for that service.</span></span>

<span data-ttu-id="340b9-107">Általános útmutatást az átmeneti hibák kezeléséhez, valamint a szolgáltatásokat és erőforrásokat célzó kapcsolatok és műveletek újrapróbálásához az [újrapróbálkozási útmutatásban](./transient-faults.md) talál.</span><span class="sxs-lookup"><span data-stu-id="340b9-107">For general guidance on handling transient faults, and retrying connections and operations against services and resources, see [Retry guidance](./transient-faults.md).</span></span>

<span data-ttu-id="340b9-108">A következő táblázat az útmutatóban érintett Azure-szolgáltatások újrapróbálkozási funkcióiról nyújt áttekintést.</span><span class="sxs-lookup"><span data-stu-id="340b9-108">The following table summarizes the retry features for the Azure services described in this guidance.</span></span>

| <span data-ttu-id="340b9-109">**Szolgáltatás**</span><span class="sxs-lookup"><span data-stu-id="340b9-109">**Service**</span></span> | <span data-ttu-id="340b9-110">**Újrapróbálkozási képességek**</span><span class="sxs-lookup"><span data-stu-id="340b9-110">**Retry capabilities**</span></span> | <span data-ttu-id="340b9-111">**Szabályzatkonfiguráció**</span><span class="sxs-lookup"><span data-stu-id="340b9-111">**Policy configuration**</span></span> | <span data-ttu-id="340b9-112">**Hatókör**</span><span class="sxs-lookup"><span data-stu-id="340b9-112">**Scope**</span></span> | <span data-ttu-id="340b9-113">**Telemetriafunkciók**</span><span class="sxs-lookup"><span data-stu-id="340b9-113">**Telemetry features**</span></span> |
| --- | --- | --- | --- | --- |
| <span data-ttu-id="340b9-114">**[Azure Active Directory](#azure-active-directory)**</span><span class="sxs-lookup"><span data-stu-id="340b9-114">**[Azure Active Directory](#azure-active-directory)**</span></span> |<span data-ttu-id="340b9-115">Natív, az ADAL-kódtár része</span><span class="sxs-lookup"><span data-stu-id="340b9-115">Native in ADAL library</span></span> |<span data-ttu-id="340b9-116">Beágyazva az ADAL-kódtárba</span><span class="sxs-lookup"><span data-stu-id="340b9-116">Embeded into ADAL library</span></span> |<span data-ttu-id="340b9-117">Belső</span><span class="sxs-lookup"><span data-stu-id="340b9-117">Internal</span></span> |<span data-ttu-id="340b9-118">None</span><span class="sxs-lookup"><span data-stu-id="340b9-118">None</span></span> |
| <span data-ttu-id="340b9-119">**[Cosmos DB](#cosmos-db)**</span><span class="sxs-lookup"><span data-stu-id="340b9-119">**[Cosmos DB](#cosmos-db)**</span></span> |<span data-ttu-id="340b9-120">Natív, a szolgáltatás része</span><span class="sxs-lookup"><span data-stu-id="340b9-120">Native in service</span></span> |<span data-ttu-id="340b9-121">Nem konfigurálható</span><span class="sxs-lookup"><span data-stu-id="340b9-121">Non-configurable</span></span> |<span data-ttu-id="340b9-122">Globális</span><span class="sxs-lookup"><span data-stu-id="340b9-122">Global</span></span> |<span data-ttu-id="340b9-123">TraceSource</span><span class="sxs-lookup"><span data-stu-id="340b9-123">TraceSource</span></span> |
| <span data-ttu-id="340b9-124">**[Az Event Hubs](#azure-event-hubs)**</span><span class="sxs-lookup"><span data-stu-id="340b9-124">**[Event Hubs](#azure-event-hubs)**</span></span> |<span data-ttu-id="340b9-125">Natív, az ügyfél része</span><span class="sxs-lookup"><span data-stu-id="340b9-125">Native in client</span></span> |<span data-ttu-id="340b9-126">Szoftveres</span><span class="sxs-lookup"><span data-stu-id="340b9-126">Programmatic</span></span> |<span data-ttu-id="340b9-127">Ügyfél</span><span class="sxs-lookup"><span data-stu-id="340b9-127">Client</span></span> |<span data-ttu-id="340b9-128">None</span><span class="sxs-lookup"><span data-stu-id="340b9-128">None</span></span> |
| <span data-ttu-id="340b9-129">**[Redis gyorsítótár](#azure-redis-cache)**</span><span class="sxs-lookup"><span data-stu-id="340b9-129">**[Redis Cache](#azure-redis-cache)**</span></span> |<span data-ttu-id="340b9-130">Natív, az ügyfél része</span><span class="sxs-lookup"><span data-stu-id="340b9-130">Native in client</span></span> |<span data-ttu-id="340b9-131">Szoftveres</span><span class="sxs-lookup"><span data-stu-id="340b9-131">Programmatic</span></span> |<span data-ttu-id="340b9-132">Ügyfél</span><span class="sxs-lookup"><span data-stu-id="340b9-132">Client</span></span> |<span data-ttu-id="340b9-133">TextWriter</span><span class="sxs-lookup"><span data-stu-id="340b9-133">TextWriter</span></span> |
| <span data-ttu-id="340b9-134">**[Keresés](#azure-search)**</span><span class="sxs-lookup"><span data-stu-id="340b9-134">**[Search](#azure-search)**</span></span> |<span data-ttu-id="340b9-135">Natív, az ügyfél része</span><span class="sxs-lookup"><span data-stu-id="340b9-135">Native in client</span></span> |<span data-ttu-id="340b9-136">Szoftveres</span><span class="sxs-lookup"><span data-stu-id="340b9-136">Programmatic</span></span> |<span data-ttu-id="340b9-137">Ügyfél</span><span class="sxs-lookup"><span data-stu-id="340b9-137">Client</span></span> |<span data-ttu-id="340b9-138">ETW vagy egyéni</span><span class="sxs-lookup"><span data-stu-id="340b9-138">ETW or Custom</span></span> |
| <span data-ttu-id="340b9-139">**[Service Bus](#service-bus)**</span><span class="sxs-lookup"><span data-stu-id="340b9-139">**[Service Bus](#service-bus)**</span></span> |<span data-ttu-id="340b9-140">Natív, az ügyfél része</span><span class="sxs-lookup"><span data-stu-id="340b9-140">Native in client</span></span> |<span data-ttu-id="340b9-141">Szoftveres</span><span class="sxs-lookup"><span data-stu-id="340b9-141">Programmatic</span></span> |<span data-ttu-id="340b9-142">Névtérkezelő, üzenetkezelési előállító vagy ügyfél</span><span class="sxs-lookup"><span data-stu-id="340b9-142">Namespace Manager, Messaging Factory, and Client</span></span> |<span data-ttu-id="340b9-143">ETW</span><span class="sxs-lookup"><span data-stu-id="340b9-143">ETW</span></span> |
| <span data-ttu-id="340b9-144">**[Service Fabric](#service-fabric)**</span><span class="sxs-lookup"><span data-stu-id="340b9-144">**[Service Fabric](#service-fabric)**</span></span> |<span data-ttu-id="340b9-145">Natív, az ügyfél része</span><span class="sxs-lookup"><span data-stu-id="340b9-145">Native in client</span></span> |<span data-ttu-id="340b9-146">Szoftveres</span><span class="sxs-lookup"><span data-stu-id="340b9-146">Programmatic</span></span> |<span data-ttu-id="340b9-147">Ügyfél</span><span class="sxs-lookup"><span data-stu-id="340b9-147">Client</span></span> |<span data-ttu-id="340b9-148">None</span><span class="sxs-lookup"><span data-stu-id="340b9-148">None</span></span> | 
| <span data-ttu-id="340b9-149">**[SQL Database with ADO.NET](#sql-database-using-adonet)**</span><span class="sxs-lookup"><span data-stu-id="340b9-149">**[SQL Database with ADO.NET](#sql-database-using-adonet)**</span></span> |[<span data-ttu-id="340b9-150">Polly</span><span class="sxs-lookup"><span data-stu-id="340b9-150">Polly</span></span>](#transient-fault-handling-with-polly) |<span data-ttu-id="340b9-151">Deklaratív és szoftveres</span><span class="sxs-lookup"><span data-stu-id="340b9-151">Declarative and programmatic</span></span> |<span data-ttu-id="340b9-152">Önálló utasítások vagy kódblokkok</span><span class="sxs-lookup"><span data-stu-id="340b9-152">Single statements or blocks of code</span></span> |<span data-ttu-id="340b9-153">Egyéni</span><span class="sxs-lookup"><span data-stu-id="340b9-153">Custom</span></span> |
| <span data-ttu-id="340b9-154">**[SQL Database with Entity Framework](#sql-database-using-entity-framework-6)**</span><span class="sxs-lookup"><span data-stu-id="340b9-154">**[SQL Database with Entity Framework](#sql-database-using-entity-framework-6)**</span></span> |<span data-ttu-id="340b9-155">Natív, az ügyfél része</span><span class="sxs-lookup"><span data-stu-id="340b9-155">Native in client</span></span> |<span data-ttu-id="340b9-156">Szoftveres</span><span class="sxs-lookup"><span data-stu-id="340b9-156">Programmatic</span></span> |<span data-ttu-id="340b9-157">Alkalmazástartományonként globális</span><span class="sxs-lookup"><span data-stu-id="340b9-157">Global per AppDomain</span></span> |<span data-ttu-id="340b9-158">None</span><span class="sxs-lookup"><span data-stu-id="340b9-158">None</span></span> |
| <span data-ttu-id="340b9-159">**[SQL Database with Entity Framework Core](#sql-database-using-entity-framework-core)**</span><span class="sxs-lookup"><span data-stu-id="340b9-159">**[SQL Database with Entity Framework Core](#sql-database-using-entity-framework-core)**</span></span> |<span data-ttu-id="340b9-160">Natív, az ügyfél része</span><span class="sxs-lookup"><span data-stu-id="340b9-160">Native in client</span></span> |<span data-ttu-id="340b9-161">Szoftveres</span><span class="sxs-lookup"><span data-stu-id="340b9-161">Programmatic</span></span> |<span data-ttu-id="340b9-162">Alkalmazástartományonként globális</span><span class="sxs-lookup"><span data-stu-id="340b9-162">Global per AppDomain</span></span> |<span data-ttu-id="340b9-163">None</span><span class="sxs-lookup"><span data-stu-id="340b9-163">None</span></span> |
| <span data-ttu-id="340b9-164">**[Tárolás](#azure-storage)**</span><span class="sxs-lookup"><span data-stu-id="340b9-164">**[Storage](#azure-storage)**</span></span> |<span data-ttu-id="340b9-165">Natív, az ügyfél része</span><span class="sxs-lookup"><span data-stu-id="340b9-165">Native in client</span></span> |<span data-ttu-id="340b9-166">Szoftveres</span><span class="sxs-lookup"><span data-stu-id="340b9-166">Programmatic</span></span> |<span data-ttu-id="340b9-167">Ügyfél- és különálló műveletek</span><span class="sxs-lookup"><span data-stu-id="340b9-167">Client and individual operations</span></span> |<span data-ttu-id="340b9-168">TraceSource</span><span class="sxs-lookup"><span data-stu-id="340b9-168">TraceSource</span></span> |

> [!NOTE]
> <span data-ttu-id="340b9-169">Az Azure beépített része, majd ismételje meg mechanizmusok, nincs jelenleg nem tudja alkalmazni a különböző típusú hiba vagy kivétel különböző újrapróbálkozási házirendje.</span><span class="sxs-lookup"><span data-stu-id="340b9-169">For most of the Azure built-in retry mechanisms, there is currently no way apply a different retry policy for different types of error or exception.</span></span> <span data-ttu-id="340b9-170">Konfigurálnia kell egy házirendet, amely az optimális átlagos teljesítményt és rendelkezésre állást biztosít.</span><span class="sxs-lookup"><span data-stu-id="340b9-170">You should configure a policy that provides the optimum average performance and availability.</span></span> <span data-ttu-id="340b9-171">A szabályzat finomhangolásához elemezze a naplófájlokat, hogy megállapítsa, milyen típusú átmeneti hibák szoktak történni.</span><span class="sxs-lookup"><span data-stu-id="340b9-171">One way to fine-tune the policy is to analyze log files to determine the type of transient faults that are occurring.</span></span> 

## <a name="azure-active-directory"></a><span data-ttu-id="340b9-172">Azure Active Directory</span><span class="sxs-lookup"><span data-stu-id="340b9-172">Azure Active Directory</span></span>
<span data-ttu-id="340b9-173">Az Azure Active Directory egy átfogó, felhőalapú identitás- és hozzáférés-kezelő megoldás, amely ötvözi az alapvető címtárszolgáltatásokat, a fejlett identitáskezelést, a biztonsági szolgáltatásokat és az alkalmazáshozzáférés-felügyeletet.</span><span class="sxs-lookup"><span data-stu-id="340b9-173">Azure Active Directory (Azure AD) is a comprehensive identity and access management cloud solution that combines core directory services, advanced identity governance, security, and application access management.</span></span> <span data-ttu-id="340b9-174">Az Azure AD ezenkívül identitáskezelő platformot kínál a fejlesztőknek, hogy központi szabályzatokon és szabályokon alapuló hozzáférés-vezérléssel bővíthessék az alkalmazásaikat.</span><span class="sxs-lookup"><span data-stu-id="340b9-174">Azure AD also offers developers an identity management platform to deliver access control to their applications, based on centralized policy and rules.</span></span>

> [!NOTE]
> <span data-ttu-id="340b9-175">Felügyelt Szolgáltatásidentitás végpontokon újrapróbálkozási útmutatóért lásd: [egy Azure virtuális gép felügyelt szolgáltatás Identity (MSI) használata a token beszerzési](/azure/active-directory/managed-service-identity/how-to-use-vm-token#error-handling).</span><span class="sxs-lookup"><span data-stu-id="340b9-175">For retry guidance on Managed Service Identity endpoints, see [How to use an Azure VM Managed Service Identity (MSI) for token acquisition](/azure/active-directory/managed-service-identity/how-to-use-vm-token#error-handling).</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="340b9-176">Újrapróbálkozási mechanizmus</span><span class="sxs-lookup"><span data-stu-id="340b9-176">Retry mechanism</span></span>
<span data-ttu-id="340b9-177">Az Azure Active Directory beépített újrapróbálkozási mechanizmussal rendelkezik az Active Directory Authentication Library (ADAL) részeként.</span><span class="sxs-lookup"><span data-stu-id="340b9-177">There is a built-in retry mechanism for Azure Active Directory in the Active Directory Authentication Library (ADAL).</span></span> <span data-ttu-id="340b9-178">A váratlan zárolások elkerülése érdekében azt javasoljuk, hogy külső kódtárak és alkalmazáskódok **ne** próbálkozhassanak újra sikertelen kapcsolódás esetén, és ezek újrapróbálkozásait az ADAL kezelje.</span><span class="sxs-lookup"><span data-stu-id="340b9-178">To avoid unexpected lockouts, we recommend that third party libraries and application code do **not** retry failed connections, but allow ADAL to handle retries.</span></span> 

### <a name="retry-usage-guidance"></a><span data-ttu-id="340b9-179">Újrapróbálkozásokra vonatkozó útmutató</span><span class="sxs-lookup"><span data-stu-id="340b9-179">Retry usage guidance</span></span>
<span data-ttu-id="340b9-180">Ügyeljen a következőkre az Azure Active Directory használata során:</span><span class="sxs-lookup"><span data-stu-id="340b9-180">Consider the following guidelines when using Azure Active Directory:</span></span>

* <span data-ttu-id="340b9-181">Amikor csak lehetséges, az ADAL-kódtárat és az újrapróbálkozások beépített támogatását használja.</span><span class="sxs-lookup"><span data-stu-id="340b9-181">When possible, use the ADAL library and the built-in support for retries.</span></span>
* <span data-ttu-id="340b9-182">Ha a REST API-t használ az Azure Active Directory, a eredménykódja 429 (túl sok kérelem) vagy a 5xx tartomány hiba esetén próbálja megismételni a műveletet.</span><span class="sxs-lookup"><span data-stu-id="340b9-182">If you are using the REST API for Azure Active Directory, retry the operation if the result code is 429 (Too Many Requests) or an error in the 5xx range.</span></span> <span data-ttu-id="340b9-183">Más hibák esetében ne engedélyezze az újrapróbálkozást.</span><span class="sxs-lookup"><span data-stu-id="340b9-183">Do not retry for any other errors.</span></span>
* <span data-ttu-id="340b9-184">Az exponenciális visszatartási szabályzat használatát az Azure Active Directory Batch-forgatókönyvei esetében javasoljuk.</span><span class="sxs-lookup"><span data-stu-id="340b9-184">An exponential back-off policy is recommended for use in batch scenarios with Azure Active Directory.</span></span>

<span data-ttu-id="340b9-185">A következő kezdőbeállításokat javasoljuk az újrapróbálkozási műveletekhez.</span><span class="sxs-lookup"><span data-stu-id="340b9-185">Consider starting with the following settings for retrying operations.</span></span> <span data-ttu-id="340b9-186">Ezek általános célú beállítások, ezért javasoljuk, hogy monitorozza műveleteit, és finomhangolja az értékeket saját igényei szerint.</span><span class="sxs-lookup"><span data-stu-id="340b9-186">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="340b9-187">**Környezet**</span><span class="sxs-lookup"><span data-stu-id="340b9-187">**Context**</span></span> | <span data-ttu-id="340b9-188">**Mintacél E2E<br />max. késleltetése**</span><span class="sxs-lookup"><span data-stu-id="340b9-188">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="340b9-189">**Újrapróbálkozási stratégia**</span><span class="sxs-lookup"><span data-stu-id="340b9-189">**Retry strategy**</span></span> | <span data-ttu-id="340b9-190">**Beállítások**</span><span class="sxs-lookup"><span data-stu-id="340b9-190">**Settings**</span></span> | <span data-ttu-id="340b9-191">**Értékek**</span><span class="sxs-lookup"><span data-stu-id="340b9-191">**Values**</span></span> | <span data-ttu-id="340b9-192">**Működési elv**</span><span class="sxs-lookup"><span data-stu-id="340b9-192">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="340b9-193">Interaktív, felhasználói felület</span><span class="sxs-lookup"><span data-stu-id="340b9-193">Interactive, UI,</span></span><br /><span data-ttu-id="340b9-194">vagy előtér</span><span class="sxs-lookup"><span data-stu-id="340b9-194">or foreground</span></span> |<span data-ttu-id="340b9-195">2 másodperc</span><span class="sxs-lookup"><span data-stu-id="340b9-195">2 sec</span></span> |<span data-ttu-id="340b9-196">FixedInterval</span><span class="sxs-lookup"><span data-stu-id="340b9-196">FixedInterval</span></span> |<span data-ttu-id="340b9-197">Ismétlések száma</span><span class="sxs-lookup"><span data-stu-id="340b9-197">Retry count</span></span><br /><span data-ttu-id="340b9-198">Újrapróbálkozási időköz</span><span class="sxs-lookup"><span data-stu-id="340b9-198">Retry interval</span></span><br /><span data-ttu-id="340b9-199">Első gyors újrapróbálkozás</span><span class="sxs-lookup"><span data-stu-id="340b9-199">First fast retry</span></span> |<span data-ttu-id="340b9-200">3</span><span class="sxs-lookup"><span data-stu-id="340b9-200">3</span></span><br /><span data-ttu-id="340b9-201">500 ms</span><span class="sxs-lookup"><span data-stu-id="340b9-201">500 ms</span></span><br /><span data-ttu-id="340b9-202">true</span><span class="sxs-lookup"><span data-stu-id="340b9-202">true</span></span> |<span data-ttu-id="340b9-203">1. kísérlet – 0 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="340b9-203">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="340b9-204">2. kísérlet – 500 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="340b9-204">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="340b9-205">3. kísérlet – 500 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="340b9-205">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="340b9-206">Háttér vagy</span><span class="sxs-lookup"><span data-stu-id="340b9-206">Background or</span></span><br /><span data-ttu-id="340b9-207">kötegelt</span><span class="sxs-lookup"><span data-stu-id="340b9-207">batch</span></span> |<span data-ttu-id="340b9-208">60 másodperc</span><span class="sxs-lookup"><span data-stu-id="340b9-208">60 sec</span></span> |<span data-ttu-id="340b9-209">ExponentialBackoff</span><span class="sxs-lookup"><span data-stu-id="340b9-209">ExponentialBackoff</span></span> |<span data-ttu-id="340b9-210">Ismétlések száma</span><span class="sxs-lookup"><span data-stu-id="340b9-210">Retry count</span></span><br /><span data-ttu-id="340b9-211">Visszatartás (min.)</span><span class="sxs-lookup"><span data-stu-id="340b9-211">Min back-off</span></span><br /><span data-ttu-id="340b9-212">Visszatartás (max.)</span><span class="sxs-lookup"><span data-stu-id="340b9-212">Max back-off</span></span><br /><span data-ttu-id="340b9-213">Visszatartás (változás)</span><span class="sxs-lookup"><span data-stu-id="340b9-213">Delta back-off</span></span><br /><span data-ttu-id="340b9-214">Első gyors újrapróbálkozás</span><span class="sxs-lookup"><span data-stu-id="340b9-214">First fast retry</span></span> |<span data-ttu-id="340b9-215">5</span><span class="sxs-lookup"><span data-stu-id="340b9-215">5</span></span><br /><span data-ttu-id="340b9-216">0 másodperc</span><span class="sxs-lookup"><span data-stu-id="340b9-216">0 sec</span></span><br /><span data-ttu-id="340b9-217">60 másodperc</span><span class="sxs-lookup"><span data-stu-id="340b9-217">60 sec</span></span><br /><span data-ttu-id="340b9-218">2 másodperc</span><span class="sxs-lookup"><span data-stu-id="340b9-218">2 sec</span></span><br /><span data-ttu-id="340b9-219">false</span><span class="sxs-lookup"><span data-stu-id="340b9-219">false</span></span> |<span data-ttu-id="340b9-220">1. kísérlet – 0 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="340b9-220">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="340b9-221">2. kísérlet – kb. 2 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="340b9-221">Attempt 2 - delay ~2 sec</span></span><br /><span data-ttu-id="340b9-222">3. kísérlet – kb. 6 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="340b9-222">Attempt 3 - delay ~6 sec</span></span><br /><span data-ttu-id="340b9-223">4. kísérlet – kb. 14 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="340b9-223">Attempt 4 - delay ~14 sec</span></span><br /><span data-ttu-id="340b9-224">5. kísérlet – kb. 30 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="340b9-224">Attempt 5 - delay ~30 sec</span></span> |

### <a name="more-information"></a><span data-ttu-id="340b9-225">További információ</span><span class="sxs-lookup"><span data-stu-id="340b9-225">More information</span></span>
* <span data-ttu-id="340b9-226">[Az Azure Active Directory hitelesítési kódtárai][adal]</span><span class="sxs-lookup"><span data-stu-id="340b9-226">[Azure Active Directory Authentication Libraries][adal]</span></span>

## <a name="cosmos-db"></a><span data-ttu-id="340b9-227">Cosmos DB</span><span class="sxs-lookup"><span data-stu-id="340b9-227">Cosmos DB</span></span>

<span data-ttu-id="340b9-228">A Cosmos DB egy teljes körűen felügyelt, többmodelles adatbázis-szolgáltatás, amely támogatja a séma nélküli JSON-adatok használatát.</span><span class="sxs-lookup"><span data-stu-id="340b9-228">Cosmos DB is a fully-managed multi-model database that supports schema-less JSON data.</span></span> <span data-ttu-id="340b9-229">Teljesítménye konfigurálható és megbízható, natív JavaScript-tranzakciófeldolgozást kínál, és mivel felhőbeli felhasználásra készült, rugalmasan méretezhető.</span><span class="sxs-lookup"><span data-stu-id="340b9-229">It offers configurable and reliable performance, native JavaScript transactional processing, and is built for the cloud with elastic scale.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="340b9-230">Újrapróbálkozási mechanizmus</span><span class="sxs-lookup"><span data-stu-id="340b9-230">Retry mechanism</span></span>
<span data-ttu-id="340b9-231">A `DocumentClient` osztály automatikusan újrapróbálkozik a sikertelen kísérletekkel.</span><span class="sxs-lookup"><span data-stu-id="340b9-231">The `DocumentClient` class automatically retries failed attempts.</span></span> <span data-ttu-id="340b9-232">Az újrapróbálkozások számának és a maximális várakozási idő beállításához a [ConnectionPolicy.RetryOptions] konfigurálása szükséges.</span><span class="sxs-lookup"><span data-stu-id="340b9-232">To set the number of retries and the maximum wait time, configure [ConnectionPolicy.RetryOptions].</span></span> <span data-ttu-id="340b9-233">Az ügyfél által okozott kivételek vagy túlmutatnak az újrapróbálkozási szabályzaton, vagy nem átmeneti hibák.</span><span class="sxs-lookup"><span data-stu-id="340b9-233">Exceptions that the client raises are either beyond the retry policy or are not transient errors.</span></span>

<span data-ttu-id="340b9-234">Ha a Cosmos DB korlátozza az ügyfél kísérleteit, a 429-es HTTP-hibaüzenetet adja vissza.</span><span class="sxs-lookup"><span data-stu-id="340b9-234">If Cosmos DB throttles the client, it returns an HTTP 429 error.</span></span> <span data-ttu-id="340b9-235">Ellenőrizze a `DocumentClientException` állapotkódját.</span><span class="sxs-lookup"><span data-stu-id="340b9-235">Check the status code in the `DocumentClientException`.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="340b9-236">Szabályzatkonfiguráció</span><span class="sxs-lookup"><span data-stu-id="340b9-236">Policy configuration</span></span>
<span data-ttu-id="340b9-237">A következő táblázatban a `RetryOptions` osztály alapértelmezett beállításait tekintheti meg.</span><span class="sxs-lookup"><span data-stu-id="340b9-237">The following table shows the default settings for the `RetryOptions` class.</span></span>

| <span data-ttu-id="340b9-238">Beállítás</span><span class="sxs-lookup"><span data-stu-id="340b9-238">Setting</span></span> | <span data-ttu-id="340b9-239">Alapértelmezett érték</span><span class="sxs-lookup"><span data-stu-id="340b9-239">Default value</span></span> | <span data-ttu-id="340b9-240">Leírás</span><span class="sxs-lookup"><span data-stu-id="340b9-240">Description</span></span> |
| --- | --- | --- |
| <span data-ttu-id="340b9-241">MaxRetryAttemptsOnThrottledRequests</span><span class="sxs-lookup"><span data-stu-id="340b9-241">MaxRetryAttemptsOnThrottledRequests</span></span> |<span data-ttu-id="340b9-242">9</span><span class="sxs-lookup"><span data-stu-id="340b9-242">9</span></span> |<span data-ttu-id="340b9-243">Az újrapróbálkozások maximális száma abban az esetben, ha a Cosmos DB az ügyfélre alkalmazott korlátozása miatt sikertelen a kísérlet.</span><span class="sxs-lookup"><span data-stu-id="340b9-243">The maximum number of retries if the request fails because Cosmos DB applied rate limiting on the client.</span></span> |
| <span data-ttu-id="340b9-244">MaxRetryWaitTimeInSeconds</span><span class="sxs-lookup"><span data-stu-id="340b9-244">MaxRetryWaitTimeInSeconds</span></span> |<span data-ttu-id="340b9-245">30</span><span class="sxs-lookup"><span data-stu-id="340b9-245">30</span></span> |<span data-ttu-id="340b9-246">A maximális újrapróbálkozási idő másodpercben.</span><span class="sxs-lookup"><span data-stu-id="340b9-246">The maximum retry time in seconds.</span></span> |

### <a name="example"></a><span data-ttu-id="340b9-247">Példa</span><span class="sxs-lookup"><span data-stu-id="340b9-247">Example</span></span>
```csharp
DocumentClient client = new DocumentClient(new Uri(endpoint), authKey); ;
var options = client.ConnectionPolicy.RetryOptions;
options.MaxRetryAttemptsOnThrottledRequests = 5;
options.MaxRetryWaitTimeInSeconds = 15;
```

### <a name="telemetry"></a><span data-ttu-id="340b9-248">Telemetria</span><span class="sxs-lookup"><span data-stu-id="340b9-248">Telemetry</span></span>
<span data-ttu-id="340b9-249">Az újrapróbálkozási kísérletek strukturálatlan nyomkövetési üzenetekként lesznek naplózva a .NET **TraceSource** használatával.</span><span class="sxs-lookup"><span data-stu-id="340b9-249">Retry attempts are logged as unstructured trace messages through a .NET **TraceSource**.</span></span> <span data-ttu-id="340b9-250">Az események rögzítéséhez és megfelelő célnaplóba való írásához egy **TraceListener** osztályt kell konfigurálnia.</span><span class="sxs-lookup"><span data-stu-id="340b9-250">You must configure a **TraceListener** to capture the events and write them to a suitable destination log.</span></span>

<span data-ttu-id="340b9-251">Amennyiben például a következőt adja hozzá az App.config fájlhoz, a szövegfájlban a végrehajtható fájllal megegyező helyen jönnek létre a nyomkövetési adatok:</span><span class="sxs-lookup"><span data-stu-id="340b9-251">For example, if you add the following to your App.config file, traces will be generated in a text file in the same location as the executable:</span></span>

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

## <a name="event-hubs"></a><span data-ttu-id="340b9-252">Event Hubs</span><span class="sxs-lookup"><span data-stu-id="340b9-252">Event Hubs</span></span>

<span data-ttu-id="340b9-253">Az Azure Event Hubs egy rendkívül nagy kapacitású, telemetriai adatokat betöltő szolgáltatás, amely események millióinak adatait képes összegyűjteni, átalakítani és tárolni.</span><span class="sxs-lookup"><span data-stu-id="340b9-253">Azure Event Hubs is a hyper-scale telemetry ingestion service that collects, transforms, and stores millions of events.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="340b9-254">Újrapróbálkozási mechanizmus</span><span class="sxs-lookup"><span data-stu-id="340b9-254">Retry mechanism</span></span>
<span data-ttu-id="340b9-255">Az Azure Event Hubs Client Library újrapróbálkozási viselkedését az `EventHubClient` osztály `RetryPolicy` tulajdonsága vezérli.</span><span class="sxs-lookup"><span data-stu-id="340b9-255">Retry behavior in the Azure Event Hubs Client Library is controlled by the `RetryPolicy` property on the `EventHubClient` class.</span></span> <span data-ttu-id="340b9-256">Az alapértelmezett szabályzat exponenciális visszatartással végzi el az újrapróbálkozást, ha az Azure Event Hub egy átmeneti `EventHubsException` vagy egy `OperationCanceledException` választ ad.</span><span class="sxs-lookup"><span data-stu-id="340b9-256">The default policy retries with exponential backoff when Azure Event Hub returns a transient `EventHubsException` or an `OperationCanceledException`.</span></span>

### <a name="example"></a><span data-ttu-id="340b9-257">Példa</span><span class="sxs-lookup"><span data-stu-id="340b9-257">Example</span></span>
```csharp
EventHubClient client = EventHubClient.CreateFromConnectionString("[event_hub_connection_string]");
client.RetryPolicy = RetryPolicy.Default;
```

### <a name="more-information"></a><span data-ttu-id="340b9-258">További információ</span><span class="sxs-lookup"><span data-stu-id="340b9-258">More information</span></span>
[<span data-ttu-id="340b9-259">.NET standard ügyféloldali kódtár az Azure Event Hubshoz</span><span class="sxs-lookup"><span data-stu-id="340b9-259"> .NET Standard client library for Azure Event Hubs</span></span>](https://github.com/Azure/azure-event-hubs-dotnet)

## <a name="azure-redis-cache"></a><span data-ttu-id="340b9-260">Azure Redis Cache</span><span class="sxs-lookup"><span data-stu-id="340b9-260">Azure Redis Cache</span></span>
<span data-ttu-id="340b9-261">Az Azure Redis Cache gyors adathozzáférést és alacsony késést kínáló gyorsítótár-szolgáltatás, amely a népszerű, nyílt forráskódú Redis Cache-re épül.</span><span class="sxs-lookup"><span data-stu-id="340b9-261">Azure Redis Cache is a fast data access and low latency cache service based on the popular open source Redis Cache.</span></span> <span data-ttu-id="340b9-262">Biztonságos, a Microsoft felügyeli, és az Azure bármelyik alkalmazásából elérhető.</span><span class="sxs-lookup"><span data-stu-id="340b9-262">It is secure, managed by Microsoft, and is accessible from any application in Azure.</span></span>

<span data-ttu-id="340b9-263">Ebben az útmutatóban azt feltételezzük, hogy a StackExchange.Redis ügyfelet használja a gyorsítótár eléréséhez.</span><span class="sxs-lookup"><span data-stu-id="340b9-263">The guidance in this section is based on using the StackExchange.Redis client to access the cache.</span></span> <span data-ttu-id="340b9-264">A további alkalmas ügyfelek listája a [Redis webhelyén](http://redis.io/clients) tekinthető meg, ám ezeknek eltérő újrapróbálkozási mechanizmusai lehetnek.</span><span class="sxs-lookup"><span data-stu-id="340b9-264">A list of other suitable clients can be found on the [Redis website](http://redis.io/clients), and these may have different retry mechanisms.</span></span>

<span data-ttu-id="340b9-265">Vegye figyelembe, hogy a StackExchange.Redis ügyfél egyetlen kapcsolaton keresztül végez multiplexálást.</span><span class="sxs-lookup"><span data-stu-id="340b9-265">Note that the StackExchange.Redis client uses multiplexing through a single connection.</span></span> <span data-ttu-id="340b9-266">A javasolt felhasználás az, ha létrehozza az ügyfél egy példányát az alkalmazás indításakor, és ezt a példányt használja a gyorsítótár elérését célzó összes művelethez.</span><span class="sxs-lookup"><span data-stu-id="340b9-266">The recommended usage is to create an instance of the client at application startup and use this instance for all operations against the cache.</span></span> <span data-ttu-id="340b9-267">Így a gyorsítótárral való kapcsolat csak egyszer jön létre, ezért az ebben a szakaszban leírt összes útmutatás ezen első kapcsolat újrapróbálkozási szabályzatára vonatkozik, nem pedig a gyorsítótárhoz hozzáférő egyes műveletekre.</span><span class="sxs-lookup"><span data-stu-id="340b9-267">For this reason, the connection to the cache is made only once, and so all of the guidance in this section is related to the retry policy for this initial connection—and not for each operation that accesses the cache.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="340b9-268">Újrapróbálkozási mechanizmus</span><span class="sxs-lookup"><span data-stu-id="340b9-268">Retry mechanism</span></span>
<span data-ttu-id="340b9-269">A StackExchange.Redis ügyfél egy számos beállítással konfigurált kapcsolatkezelő osztályt használ. Néhány példa ezekre a beállításokra:</span><span class="sxs-lookup"><span data-stu-id="340b9-269">The StackExchange.Redis client uses a connection manager class that is configured through a set of options, incuding:</span></span>

- <span data-ttu-id="340b9-270">**ConnectRetry**.</span><span class="sxs-lookup"><span data-stu-id="340b9-270">**ConnectRetry**.</span></span> <span data-ttu-id="340b9-271">Ennyi alkalommal próbálkozik újra a gyorsítótárhoz való sikertelen kapcsolódás esetén.</span><span class="sxs-lookup"><span data-stu-id="340b9-271">The number of times a failed connection to the cache will be retried.</span></span>
- <span data-ttu-id="340b9-272">**ReconnectRetryPolicy**.</span><span class="sxs-lookup"><span data-stu-id="340b9-272">**ReconnectRetryPolicy**.</span></span> <span data-ttu-id="340b9-273">A használt újrapróbálkozási stratégia.</span><span class="sxs-lookup"><span data-stu-id="340b9-273">The retry strategy to use.</span></span>
- <span data-ttu-id="340b9-274">**ConnectTimeout**.</span><span class="sxs-lookup"><span data-stu-id="340b9-274">**ConnectTimeout**.</span></span> <span data-ttu-id="340b9-275">A maximális várakozási idő milliszekundumban.</span><span class="sxs-lookup"><span data-stu-id="340b9-275">The maximum waiting time in milliseconds.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="340b9-276">Szabályzatkonfiguráció</span><span class="sxs-lookup"><span data-stu-id="340b9-276">Policy configuration</span></span>
<span data-ttu-id="340b9-277">Az újrapróbálkozási szabályzat konfigurálása szoftveresen történik. Az ügyfél beállításait a gyorsítótárhoz való kapcsolódás előtt kell megadni.</span><span class="sxs-lookup"><span data-stu-id="340b9-277">Retry policies are configured programmatically by setting the options for the client before connecting to the cache.</span></span> <span data-ttu-id="340b9-278">Ehhez létre kell hozni a **ConfigurationOptions** osztály egy példányát, feltölteni adatokkal a tulajdonságait, majd továbbítani azt a **Connect** metódusnak.</span><span class="sxs-lookup"><span data-stu-id="340b9-278">This can be done by creating an instance of the **ConfigurationOptions** class, populating its properties, and passing it to the **Connect** method.</span></span>

<span data-ttu-id="340b9-279">A beépített osztályok támogatják a lineáris (állandó) késleltetést, illetve az exponenciális visszatartást véletlenszerű újrapróbálkozási időközökkel.</span><span class="sxs-lookup"><span data-stu-id="340b9-279">The built-in classes support linear (constant) delay and exponential backoff with randomized retry intervals.</span></span> <span data-ttu-id="340b9-280">Ezenkívül létrehozhat egyéni újrapróbálkozási szabályzatot az **IReconnectRetryPolicy** felület implementálásával.</span><span class="sxs-lookup"><span data-stu-id="340b9-280">You can also create a custom retry policy by implementing the **IReconnectRetryPolicy** interface.</span></span>

<span data-ttu-id="340b9-281">A következő példa exponenciális visszatartással konfigurálja az újrapróbálkozási stratégiát.</span><span class="sxs-lookup"><span data-stu-id="340b9-281">The following example configures a retry strategy using exponential backoff.</span></span>

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

<span data-ttu-id="340b9-282">Egy másik lehetőség, hogy a beállításokat sztringként adja meg és továbbítja a **Connect** metódusnak.</span><span class="sxs-lookup"><span data-stu-id="340b9-282">Alternatively, you can specify the options as a string, and pass this to the **Connect** method.</span></span> <span data-ttu-id="340b9-283">Vegye figyelembe, hogy a **ReconnectRetryPolicy** tulajdonság nem állítható be ezen a módon, csak a programkódon keresztül.</span><span class="sxs-lookup"><span data-stu-id="340b9-283">Note that the **ReconnectRetryPolicy** property cannot be set this way, only through code.</span></span>

```csharp
var options = "localhost,connectRetry=3,connectTimeout=2000";
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

<span data-ttu-id="340b9-284">Amikor a gyorsítótárhoz csatlakozik, közvetlenül is megadhat beállításokat.</span><span class="sxs-lookup"><span data-stu-id="340b9-284">You can also specify options directly when you connect to the cache.</span></span>

```csharp
var conn = ConnectionMultiplexer.Connect("redis0:6380,redis1:6380,connectRetry=3");
```

<span data-ttu-id="340b9-285">További információért tekintse meg [a Stack Exchange Redis konfigurálását](https://stackexchange.github.io/StackExchange.Redis/Configuration) a StackExchange.Redis dokumentációjában.</span><span class="sxs-lookup"><span data-stu-id="340b9-285">For more information, see [Stack Exchange Redis Configuration](https://stackexchange.github.io/StackExchange.Redis/Configuration) in the StackExchange.Redis documentation.</span></span>

<span data-ttu-id="340b9-286">A következő táblázatban a beépített újrapróbálkozási szabályzat alapértelmezett beállításai láthatók.</span><span class="sxs-lookup"><span data-stu-id="340b9-286">The following table shows the default settings for the built-in retry policy.</span></span>

| <span data-ttu-id="340b9-287">**Környezet**</span><span class="sxs-lookup"><span data-stu-id="340b9-287">**Context**</span></span> | <span data-ttu-id="340b9-288">**Beállítás**</span><span class="sxs-lookup"><span data-stu-id="340b9-288">**Setting**</span></span> | <span data-ttu-id="340b9-289">**Alapértelmezett érték**</span><span class="sxs-lookup"><span data-stu-id="340b9-289">**Default value**</span></span><br /><span data-ttu-id="340b9-290">(v 1.2.2)</span><span class="sxs-lookup"><span data-stu-id="340b9-290">(v 1.2.2)</span></span> | <span data-ttu-id="340b9-291">**Jelentés**</span><span class="sxs-lookup"><span data-stu-id="340b9-291">**Meaning**</span></span> |
| --- | --- | --- | --- |
| <span data-ttu-id="340b9-292">ConfigurationOptions</span><span class="sxs-lookup"><span data-stu-id="340b9-292">ConfigurationOptions</span></span> |<span data-ttu-id="340b9-293">ConnectRetry</span><span class="sxs-lookup"><span data-stu-id="340b9-293">ConnectRetry</span></span><br /><br /><span data-ttu-id="340b9-294">ConnectTimeout</span><span class="sxs-lookup"><span data-stu-id="340b9-294">ConnectTimeout</span></span><br /><br /><span data-ttu-id="340b9-295">SyncTimeout</span><span class="sxs-lookup"><span data-stu-id="340b9-295">SyncTimeout</span></span><br /><br /><span data-ttu-id="340b9-296">ReconnectRetryPolicy</span><span class="sxs-lookup"><span data-stu-id="340b9-296">ReconnectRetryPolicy</span></span> |<span data-ttu-id="340b9-297">3</span><span class="sxs-lookup"><span data-stu-id="340b9-297">3</span></span><br /><br /><span data-ttu-id="340b9-298">Legfeljebb 5000 ms, plusz SyncTimeout</span><span class="sxs-lookup"><span data-stu-id="340b9-298">Maximum 5000 ms plus SyncTimeout</span></span><br /><span data-ttu-id="340b9-299">1000</span><span class="sxs-lookup"><span data-stu-id="340b9-299">1000</span></span><br /><br /><span data-ttu-id="340b9-300">LinearRetry 5000 ms</span><span class="sxs-lookup"><span data-stu-id="340b9-300">LinearRetry 5000 ms</span></span> |<span data-ttu-id="340b9-301">Ennyi alkalommal kell megismételni a csatlakozási kísérletet az első csatlakozási művelet során.</span><span class="sxs-lookup"><span data-stu-id="340b9-301">The number of times to repeat connect attempts during the initial connection operation.</span></span><br /><span data-ttu-id="340b9-302">A csatlakozási műveletek időtúllépési értéke (ms).</span><span class="sxs-lookup"><span data-stu-id="340b9-302">Timeout (ms) for connect operations.</span></span> <span data-ttu-id="340b9-303">Nem az újrapróbálkozás kísérletek közötti késleltetést jelzi.</span><span class="sxs-lookup"><span data-stu-id="340b9-303">Not a delay between retry attempts.</span></span><br /><span data-ttu-id="340b9-304">A szinkron műveletek számára biztosított idő (ms).</span><span class="sxs-lookup"><span data-stu-id="340b9-304">Time (ms) to allow for synchronous operations.</span></span><br /><br /><span data-ttu-id="340b9-305">Újrapróbálkozás 5000 ms időközönként.</span><span class="sxs-lookup"><span data-stu-id="340b9-305">Retry every 5000 ms.</span></span>|

> [!NOTE]
> <span data-ttu-id="340b9-306">A szinkron műveletek esetében a `SyncTimeout` hozzájárulhat a végpontok közötti késéshez, de a túl alacsony érték gyakori időtúllépéseket eredményezhet.</span><span class="sxs-lookup"><span data-stu-id="340b9-306">For synchronous operations, `SyncTimeout` can add to the end-to-end latency, but setting the value too low can cause excessive timeouts.</span></span> <span data-ttu-id="340b9-307">További információért lásd [az Azure Redis Cache hibaelhárítását][redis-cache-troubleshoot].</span><span class="sxs-lookup"><span data-stu-id="340b9-307">See [How to troubleshoot Azure Redis Cache][redis-cache-troubleshoot].</span></span> <span data-ttu-id="340b9-308">Általában kerülje a szinkron műveletek használatát, és alkalmazzon inkább aszinkron műveleteket.</span><span class="sxs-lookup"><span data-stu-id="340b9-308">In general, avoid using synchronous operations, and use asynchronous operations instead.</span></span> <span data-ttu-id="340b9-309">További információért lásd a [folyamatokat és a multiplexereket](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/PipelinesMultiplexers.md).</span><span class="sxs-lookup"><span data-stu-id="340b9-309">For more information see [Pipelines and Multiplexers](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/PipelinesMultiplexers.md).</span></span>
>
>

### <a name="retry-usage-guidance"></a><span data-ttu-id="340b9-310">Újrapróbálkozásokra vonatkozó útmutató</span><span class="sxs-lookup"><span data-stu-id="340b9-310">Retry usage guidance</span></span>
<span data-ttu-id="340b9-311">Ügyeljen a következőkre az Azure Redis Cache használata során:</span><span class="sxs-lookup"><span data-stu-id="340b9-311">Consider the following guidelines when using Azure Redis Cache:</span></span>

* <span data-ttu-id="340b9-312">A StackExchange Redis ügyfél kezeli a saját újrapróbálkozásait, de csak amikor az alkalmazás első indításakor próbál kapcsolódni a gyorsítótárhoz.</span><span class="sxs-lookup"><span data-stu-id="340b9-312">The StackExchange Redis client manages its own retries, but only when establishing a connection to the cache when the application first starts.</span></span> <span data-ttu-id="340b9-313">Meghatározhatja a kapcsolati időtúllépés értékét, az újrapróbálkozási kísérletek számát, valamint a kapcsolat létrehozására tett ismételt próbálkozások között eltelt időt, de az újrapróbálkozási szabályzat nem vonatkozik a gyorsítótárra irányuló műveletekre.</span><span class="sxs-lookup"><span data-stu-id="340b9-313">You can configure the connection timeout, the number of retry attempts, and the time between retries to establish this connection, but the retry policy does not apply to operations against the cache.</span></span>
* <span data-ttu-id="340b9-314">Nagy számú újrapróbálkozási kísérlet helyett érdemes lehet inkább az eredeti adatforráshoz csatlakozni.</span><span class="sxs-lookup"><span data-stu-id="340b9-314">Instead of using a large number of retry attempts, consider falling back by accessing the original data source instead.</span></span>

### <a name="telemetry"></a><span data-ttu-id="340b9-315">Telemetria</span><span class="sxs-lookup"><span data-stu-id="340b9-315">Telemetry</span></span>
<span data-ttu-id="340b9-316">**TextWriter** használatával adatokat gyűjthet a kapcsolatokról (más műveletekről azonban nem).</span><span class="sxs-lookup"><span data-stu-id="340b9-316">You can collect information about connections (but not other operations) using a **TextWriter**.</span></span>

```csharp
var writer = new StringWriter();
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

<span data-ttu-id="340b9-317">Az alábbi példa azt mutatja be, hogy ez milyen kimenetet eredményez.</span><span class="sxs-lookup"><span data-stu-id="340b9-317">An example of the output this generates is shown below.</span></span>

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

### <a name="examples"></a><span data-ttu-id="340b9-318">Példák</span><span class="sxs-lookup"><span data-stu-id="340b9-318">Examples</span></span>
<span data-ttu-id="340b9-319">A következő mintakód állandó (lineáris) késleltetést állít be az újrapróbálkozások között a StackExchange.Redis ügyfél inicializálásakor.</span><span class="sxs-lookup"><span data-stu-id="340b9-319">The following code example configures a constant (linear) delay between retries when initializing the StackExchange.Redis client.</span></span> <span data-ttu-id="340b9-320">Ez a példa azt mutatja be, hogyan állítható be a konfiguráció egy **ConfigurationOptions**-példánnyal.</span><span class="sxs-lookup"><span data-stu-id="340b9-320">This example shows how to set the configuration using a **ConfigurationOptions** instance.</span></span>

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

<span data-ttu-id="340b9-321">A következő példa sztringként adja meg a beállításokat, és így határozza meg a konfigurációt.</span><span class="sxs-lookup"><span data-stu-id="340b9-321">The next example sets the configuration by specifying the options as a string.</span></span> <span data-ttu-id="340b9-322">A kapcsolat időtúllépése a leghosszabb idő, amennyit a kapcsolat a gyorsítótárra várhat, nem pedig az újrapróbálkozási kísérletek közötti időköz.</span><span class="sxs-lookup"><span data-stu-id="340b9-322">The connection timeout is the maximum period of time to wait for a connection to the cache, not the delay between retry attempts.</span></span> <span data-ttu-id="340b9-323">Vegye figyelembe, hogy a **ReconnectRetryPolicy** tulajdonságot csak a programkódon keresztül lehet beállítani.</span><span class="sxs-lookup"><span data-stu-id="340b9-323">Note that the **ReconnectRetryPolicy** property can only be set by code.</span></span>

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

<span data-ttu-id="340b9-324">További példákért tekintse meg a projekt webhelyének a [konfigurációval](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Configuration.md#configuration) foglalkozó szakaszát.</span><span class="sxs-lookup"><span data-stu-id="340b9-324">For more examples, see [Configuration](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Configuration.md#configuration) on the project website.</span></span>

### <a name="more-information"></a><span data-ttu-id="340b9-325">További információ</span><span class="sxs-lookup"><span data-stu-id="340b9-325">More information</span></span>
* [<span data-ttu-id="340b9-326">Redis-webhely</span><span class="sxs-lookup"><span data-stu-id="340b9-326">Redis website</span></span>](http://redis.io/)

## <a name="azure-search"></a><span data-ttu-id="340b9-327">Azure Search</span><span class="sxs-lookup"><span data-stu-id="340b9-327">Azure Search</span></span>
<span data-ttu-id="340b9-328">Az Azure Search hatékony és kifinomult keresési lehetőségekkel egészíthet ki egy webhelyet vagy alkalmazást, gyorsan és könnyen pontosítja a keresési eredményeket, továbbá részletes és finomhangolt rangsorolási modelleket képes létrehozni.</span><span class="sxs-lookup"><span data-stu-id="340b9-328">Azure Search can be used to add powerful and sophisticated search capabilities to a website or application, quickly and easily tune search results, and construct rich and fine-tuned ranking models.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="340b9-329">Újrapróbálkozási mechanizmus</span><span class="sxs-lookup"><span data-stu-id="340b9-329">Retry mechanism</span></span>
<span data-ttu-id="340b9-330">Az Azure Search SDK újrapróbálkozási viselkedését a [SearchServiceClient] és a [SearchIndexClient] osztály `SetRetryPolicy` metódusa vezérli.</span><span class="sxs-lookup"><span data-stu-id="340b9-330">Retry behavior in the Azure Search SDK is controlled by the `SetRetryPolicy` method on the [SearchServiceClient] and [SearchIndexClient] classes.</span></span> <span data-ttu-id="340b9-331">Az alapértelmezett szabályzat exponenciális visszatartással végzi el az újrapróbálkozást, ha az Azure Search 5xx-es vagy 408-as (Kérés időtúllépése) választ ad vissza.</span><span class="sxs-lookup"><span data-stu-id="340b9-331">The default policy retries with exponential backoff when Azure Search returns a 5xx or 408 (Request Timeout) response.</span></span>

### <a name="telemetry"></a><span data-ttu-id="340b9-332">Telemetria</span><span class="sxs-lookup"><span data-stu-id="340b9-332">Telemetry</span></span>
<span data-ttu-id="340b9-333">Nyomkövetés ETW-vel vagy egyéni nyomkövetési szolgáltató regisztrálásával.</span><span class="sxs-lookup"><span data-stu-id="340b9-333">Trace with ETW or by registering a custom trace provider.</span></span> <span data-ttu-id="340b9-334">További információt az [AutoRest dokumentációjában][autorest] találhat.</span><span class="sxs-lookup"><span data-stu-id="340b9-334">For more information, see the [AutoRest documentation][autorest].</span></span>

## <a name="service-bus"></a><span data-ttu-id="340b9-335">Service Bus</span><span class="sxs-lookup"><span data-stu-id="340b9-335">Service Bus</span></span>
<span data-ttu-id="340b9-336">A Service Bus egy felhőalapú üzenetkezelési platform, amely skálázható és rugalmas módon biztosít lazán kapcsolódó üzenetváltásokat az alkalmazások összetevői számára, legyen szó felhőalapú vagy helyszíni megoldásról.</span><span class="sxs-lookup"><span data-stu-id="340b9-336">Service Bus is a cloud messaging platform that provides loosely coupled message exchange with improved scale and resiliency for components of an application, whether hosted in the cloud or on-premises.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="340b9-337">Újrapróbálkozási mechanizmus</span><span class="sxs-lookup"><span data-stu-id="340b9-337">Retry mechanism</span></span>
<span data-ttu-id="340b9-338">A Service Bus a [RetryPolicy](http://msdn.microsoft.com/library/microsoft.servicebus.retrypolicy.aspx) alaposztály implementációi alapján implementálja az újrapróbálkozásokat.</span><span class="sxs-lookup"><span data-stu-id="340b9-338">Service Bus implements retries using implementations of the [RetryPolicy](http://msdn.microsoft.com/library/microsoft.servicebus.retrypolicy.aspx) base class.</span></span> <span data-ttu-id="340b9-339">Az összes Service Bus-ügyfél elérhetővé tesz egy **RetryPolicy** tulajdonságot, amely beállítható a **RetryPolicy** alaposztály egyik implementációjaként.</span><span class="sxs-lookup"><span data-stu-id="340b9-339">All of the Service Bus clients expose a **RetryPolicy** property that can be set to one of the implementations of the **RetryPolicy** base class.</span></span> <span data-ttu-id="340b9-340">A beépített implementációk a következők:</span><span class="sxs-lookup"><span data-stu-id="340b9-340">The built-in implementations are:</span></span>

* <span data-ttu-id="340b9-341">A [RetryExponential osztály](http://msdn.microsoft.com/library/microsoft.servicebus.retryexponential.aspx).</span><span class="sxs-lookup"><span data-stu-id="340b9-341">The [RetryExponential Class](http://msdn.microsoft.com/library/microsoft.servicebus.retryexponential.aspx).</span></span> <span data-ttu-id="340b9-342">Ez elérhetővé teszi azokat a tulajdonságokat, amelyek a visszatartási időközöket, az újrapróbálkozások számát és a **TerminationTimeBuffer** tulajdonságot szabályozzák, amely korlátozza, hogy a művelet hányszor hajtható végre.</span><span class="sxs-lookup"><span data-stu-id="340b9-342">This exposes properties that control the back-off interval, the retry count, and the **TerminationTimeBuffer** property that is used to limit the total time for the operation to complete.</span></span>
* <span data-ttu-id="340b9-343">A [NoRetry osztály](http://msdn.microsoft.com/library/microsoft.servicebus.noretry.aspx).</span><span class="sxs-lookup"><span data-stu-id="340b9-343">The [NoRetry Class](http://msdn.microsoft.com/library/microsoft.servicebus.noretry.aspx).</span></span> <span data-ttu-id="340b9-344">Ezt akkor szokták használni, ha nincs szükség újrapróbálkozásra a Service Bus API-szintjén, például ha az újrapróbálkozásokat egy másik folyamat kezeli egy kötegelt vagy többlépéses művelet részeként.</span><span class="sxs-lookup"><span data-stu-id="340b9-344">This is used when retries at the Service Bus API level are not required, such as when retries are managed by another process as part of a batch or multiple step operation.</span></span>

<span data-ttu-id="340b9-345">A Service Bus-műveletek számos kivételt adhatnak vissza. Ezek listáját [az üzenetkezelési kivételek függelékében](http://msdn.microsoft.com/library/hh418082.aspx) találja.</span><span class="sxs-lookup"><span data-stu-id="340b9-345">Service Bus actions can return a range of exceptions, as listed in [Appendix: Messaging Exceptions](http://msdn.microsoft.com/library/hh418082.aspx).</span></span> <span data-ttu-id="340b9-346">A lista azt is ismerteti, hogy ezek közül melyik utal arra, hogy a művelet újrapróbálható.</span><span class="sxs-lookup"><span data-stu-id="340b9-346">The list provides information about which if these indicate that retrying the operation is appropriate.</span></span> <span data-ttu-id="340b9-347">Például a [ServerBusyException](http://msdn.microsoft.com/library/microsoft.servicebus.messaging.serverbusyexception.aspx) azt jelzi, hogy az ügyfélnek várnia kell egy ideig, majd ismét megpróbálkozni a művelettel.</span><span class="sxs-lookup"><span data-stu-id="340b9-347">For example, a [ServerBusyException](http://msdn.microsoft.com/library/microsoft.servicebus.messaging.serverbusyexception.aspx) indicates that the client should wait for a period of time, then retry the operation.</span></span> <span data-ttu-id="340b9-348">A **ServerBusyException** jelentkezésekor a Service Bus eltérő módba vált, amelyben további 10 másodperc adódik a kiszámított újrapróbálkozási késleltetéshez.</span><span class="sxs-lookup"><span data-stu-id="340b9-348">The occurrence of a **ServerBusyException** also causes Service Bus to switch to a different mode, in which an extra 10-second delay is added to the computed retry delays.</span></span> <span data-ttu-id="340b9-349">Ebből a módból rövid időn belül automatikusan kilép.</span><span class="sxs-lookup"><span data-stu-id="340b9-349">This mode is reset after a short period.</span></span>

<span data-ttu-id="340b9-350">A Service Bus által visszaadott kivételek elérhetővé teszik az **IsTransient** tulajdonságot, amely jelzi, hogy az ügyfélnek érdemes-e újrapróbálkoznia a művelettel.</span><span class="sxs-lookup"><span data-stu-id="340b9-350">The exceptions returned from Service Bus expose the **IsTransient** property that indicates if the client should retry the operation.</span></span> <span data-ttu-id="340b9-351">A beépített **RetryExponential** szabályzat a **MessagingException** osztály (az összes Service Bus-kivétel alaposztálya) **IsTransient** tulajdonságára hagyatkozik.</span><span class="sxs-lookup"><span data-stu-id="340b9-351">The built-in **RetryExponential** policy relies on the **IsTransient** property in the **MessagingException** class, which is the base class for all Service Bus exceptions.</span></span> <span data-ttu-id="340b9-352">A **RetryPolicy** alaposztály egyéni implementációi esetében a kivételtípus és az **IsTransient** tulajdonság közös használatával pontosabban szabályozhatja az újrapróbálkozási műveleteket.</span><span class="sxs-lookup"><span data-stu-id="340b9-352">If you create custom implementations of the **RetryPolicy** base class you could use a combination of the exception type and the **IsTransient** property to provide more fine-grained control over retry actions.</span></span> <span data-ttu-id="340b9-353">Például észlelhet egy **QuotaExceededException**-kivételt, és utasítást adhat, hogy csak az üzenetsor kiürítése után próbálkozzon újra az üzenetküldéssel.</span><span class="sxs-lookup"><span data-stu-id="340b9-353">For example, you could detect a **QuotaExceededException** and take action to drain the queue before retrying sending a message to it.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="340b9-354">Szabályzatkonfiguráció</span><span class="sxs-lookup"><span data-stu-id="340b9-354">Policy configuration</span></span>
<span data-ttu-id="340b9-355">Az újrapróbálkozási szabályzatok beállítása szoftveresen történik, és beállítható alapértelmezett szabályzatként a **NamespaceManager** és a **MessagingFactory** számára, vagy egyenként, az egyes üzenetkezelési ügyfelek számára.</span><span class="sxs-lookup"><span data-stu-id="340b9-355">Retry policies are set programmatically, and can be set as a default policy for a **NamespaceManager** and for a **MessagingFactory**, or individually for each messaging client.</span></span> <span data-ttu-id="340b9-356">Az üzenetkezelési munkamenet alapértelmezett újrapróbálkozási szabályzatát a **NamespaceManager** **RetryPolicy** tulajdonságában állíthatja be.</span><span class="sxs-lookup"><span data-stu-id="340b9-356">To set the default retry policy for a messaging session you set the **RetryPolicy** of the **NamespaceManager**.</span></span>

```csharp
namespaceManager.Settings.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                                maxBackoff: TimeSpan.FromSeconds(30),
                                                                maxRetryCount: 3);
```

<span data-ttu-id="340b9-357">Az üzenetkezelési előállítóból létrehozott ügyelek alapértelmezett újrapróbálkozási szabályzatát a **MessagingFactory** **RetryPolicy** tulajdonságában állíthatja be.</span><span class="sxs-lookup"><span data-stu-id="340b9-357">To set the default retry policy for all clients created from a messaging factory, you set the **RetryPolicy** of the **MessagingFactory**.</span></span>

```csharp
messagingFactory.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                    maxBackoff: TimeSpan.FromSeconds(30),
                                                    maxRetryCount: 3);
```

<span data-ttu-id="340b9-358">Az üzenetkezelési ügyfél alapértelmezett újrapróbálkozási szabályzatának beállításához, illetve az alapértelmezett szabályzat felülbírálásához a szükséges szabályzatosztály példányának **RetryPolicy** tulajdonságát kell megadnia:</span><span class="sxs-lookup"><span data-stu-id="340b9-358">To set the retry policy for a messaging client, or to override its default policy, you set its **RetryPolicy** property using an instance of the required policy class:</span></span>

```csharp
client.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                            maxBackoff: TimeSpan.FromSeconds(30),
                                            maxRetryCount: 3);
```

<span data-ttu-id="340b9-359">Az újrapróbálkozási szabályzat nem állítható be az egyes műveletek szintjén.</span><span class="sxs-lookup"><span data-stu-id="340b9-359">The retry policy cannot be set at the individual operation level.</span></span> <span data-ttu-id="340b9-360">Az az üzenetkezelési ügyfél összes műveletére érvényes lesz.</span><span class="sxs-lookup"><span data-stu-id="340b9-360">It applies to all operations for the messaging client.</span></span>
<span data-ttu-id="340b9-361">A következő táblázatban a beépített újrapróbálkozási szabályzat alapértelmezett beállításai láthatók.</span><span class="sxs-lookup"><span data-stu-id="340b9-361">The following table shows the default settings for the built-in retry policy.</span></span>

| <span data-ttu-id="340b9-362">Beállítás</span><span class="sxs-lookup"><span data-stu-id="340b9-362">Setting</span></span> | <span data-ttu-id="340b9-363">Alapértelmezett érték</span><span class="sxs-lookup"><span data-stu-id="340b9-363">Default value</span></span> | <span data-ttu-id="340b9-364">Jelentés</span><span class="sxs-lookup"><span data-stu-id="340b9-364">Meaning</span></span> |
|---------|---------------|---------|
| <span data-ttu-id="340b9-365">Szabályzat</span><span class="sxs-lookup"><span data-stu-id="340b9-365">Policy</span></span> | <span data-ttu-id="340b9-366">Exponenciális</span><span class="sxs-lookup"><span data-stu-id="340b9-366">Exponential</span></span> | <span data-ttu-id="340b9-367">Exponenciális visszatartás.</span><span class="sxs-lookup"><span data-stu-id="340b9-367">Exponential back-off.</span></span> |
| <span data-ttu-id="340b9-368">MinimalBackoff</span><span class="sxs-lookup"><span data-stu-id="340b9-368">MinimalBackoff</span></span> | <span data-ttu-id="340b9-369">0</span><span class="sxs-lookup"><span data-stu-id="340b9-369">0</span></span> | <span data-ttu-id="340b9-370">Minimális visszatartási időköz.</span><span class="sxs-lookup"><span data-stu-id="340b9-370">Minimum back-off interval.</span></span> <span data-ttu-id="340b9-371">Ez hozzáadódik a deltaBackoff alapján kiszámított újrapróbálkozási időközhöz.</span><span class="sxs-lookup"><span data-stu-id="340b9-371">This is added to the retry interval computed from deltaBackoff.</span></span> |
| <span data-ttu-id="340b9-372">MaximumBackoff</span><span class="sxs-lookup"><span data-stu-id="340b9-372">MaximumBackoff</span></span> | <span data-ttu-id="340b9-373">30 másodperc</span><span class="sxs-lookup"><span data-stu-id="340b9-373">30 seconds</span></span> | <span data-ttu-id="340b9-374">Maximális visszatartási időköz.</span><span class="sxs-lookup"><span data-stu-id="340b9-374">Maximum back-off interval.</span></span> <span data-ttu-id="340b9-375">A MaximumBackoff akkor lép működésbe, ha a kiszámított újrapróbálkozási időköz nagyobb, mint a MaxBackoff értéke.</span><span class="sxs-lookup"><span data-stu-id="340b9-375">MaximumBackoff is used if the computed retry interval is greater than MaxBackoff.</span></span> |
| <span data-ttu-id="340b9-376">DeltaBackoff</span><span class="sxs-lookup"><span data-stu-id="340b9-376">DeltaBackoff</span></span> | <span data-ttu-id="340b9-377">3 másodperc</span><span class="sxs-lookup"><span data-stu-id="340b9-377">3 seconds</span></span> | <span data-ttu-id="340b9-378">Az újrapróbálkozások közötti visszatartási időköz.</span><span class="sxs-lookup"><span data-stu-id="340b9-378">Back-off interval between retries.</span></span> <span data-ttu-id="340b9-379">Az ezt követő újrapróbálkozási kísérletek esetében az időtartomány többszörösét fogja használni a rendszer.</span><span class="sxs-lookup"><span data-stu-id="340b9-379">Multiples of this timespan will be used for subsequent retry attempts.</span></span> |
| <span data-ttu-id="340b9-380">TimeBuffer</span><span class="sxs-lookup"><span data-stu-id="340b9-380">TimeBuffer</span></span> | <span data-ttu-id="340b9-381">5 másodperc</span><span class="sxs-lookup"><span data-stu-id="340b9-381">5 seconds</span></span> | <span data-ttu-id="340b9-382">Az újrapróbálkozáshoz tartozó leállítási időpuffer.</span><span class="sxs-lookup"><span data-stu-id="340b9-382">The termination time buffer associated with the retry.</span></span> <span data-ttu-id="340b9-383">A rendszer felhagy az újrapróbálkozással, ha a TimeBuffer értékben megadottnál kevesebb idő van hátra.</span><span class="sxs-lookup"><span data-stu-id="340b9-383">Retry attempts will be abandoned if the remaining time is less than TimeBuffer.</span></span> |
| <span data-ttu-id="340b9-384">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="340b9-384">MaxRetryCount</span></span> | <span data-ttu-id="340b9-385">10</span><span class="sxs-lookup"><span data-stu-id="340b9-385">10</span></span> | <span data-ttu-id="340b9-386">Az újrapróbálkozások maximális száma.</span><span class="sxs-lookup"><span data-stu-id="340b9-386">The maximum number of retries.</span></span> |
| <span data-ttu-id="340b9-387">ServerBusyBaseSleepTime</span><span class="sxs-lookup"><span data-stu-id="340b9-387">ServerBusyBaseSleepTime</span></span> | <span data-ttu-id="340b9-388">10 másodperc</span><span class="sxs-lookup"><span data-stu-id="340b9-388">10 seconds</span></span> | <span data-ttu-id="340b9-389">Ha az utolsó kivétel a **ServerBusyException** volt, ez az érték hozzáadódik a kiszámított újrapróbálkozási időközhöz.</span><span class="sxs-lookup"><span data-stu-id="340b9-389">If the last exception encountered was **ServerBusyException**, this value will be added to the computed retry interval.</span></span> <span data-ttu-id="340b9-390">Ez az érték nem módosítható.</span><span class="sxs-lookup"><span data-stu-id="340b9-390">This value cannot be changed.</span></span> |

### <a name="retry-usage-guidance"></a><span data-ttu-id="340b9-391">Újrapróbálkozásokra vonatkozó útmutató</span><span class="sxs-lookup"><span data-stu-id="340b9-391">Retry usage guidance</span></span>
<span data-ttu-id="340b9-392">Ügyeljen a következőkre a Service Bus használata során:</span><span class="sxs-lookup"><span data-stu-id="340b9-392">Consider the following guidelines when using Service Bus:</span></span>

* <span data-ttu-id="340b9-393">A beépített **RetryExponential** implementáció használatakor nincs szükség tartalékműveletek megvalósítására, mivel a szabályzat a „foglalt kiszolgáló” kivételekre reagálva automatikusan átvált a megfelelő újrapróbálkozási módra.</span><span class="sxs-lookup"><span data-stu-id="340b9-393">When using the built-in **RetryExponential** implementation, do not implement a fallback operation as the policy reacts to Server Busy exceptions and automatically switches to an appropriate retry mode.</span></span>
* <span data-ttu-id="340b9-394">A Service Bus támogatja a Párosított névterek nevű funkciót, amely automatikus feladatátvételt implementál, és az elsődleges névtér üzenetsorának hibájakor egy másik névtér tartalék üzenetsorára vált.</span><span class="sxs-lookup"><span data-stu-id="340b9-394">Service Bus supports a feature called Paired Namespaces, which implements automatic failover to a backup queue in a separate namespace if the queue in the primary namespace fails.</span></span> <span data-ttu-id="340b9-395">A másodlagos üzenetsor üzenetei továbbküldhetők az elsődleges üzenetsornak, miután az helyreállt.</span><span class="sxs-lookup"><span data-stu-id="340b9-395">Messages from the secondary queue can be sent back to the primary queue when it recovers.</span></span> <span data-ttu-id="340b9-396">Ez a funkció az átmeneti hibák kezelésére szolgál.</span><span class="sxs-lookup"><span data-stu-id="340b9-396">This feature helps to address transient failures.</span></span> <span data-ttu-id="340b9-397">További információért lásd [az aszinkron üzenetkezelési minták és a magas rendelkezésre állás](http://msdn.microsoft.com/library/azure/dn292562.aspx) ismertetését.</span><span class="sxs-lookup"><span data-stu-id="340b9-397">For more information, see [Asynchronous Messaging Patterns and High Availability](http://msdn.microsoft.com/library/azure/dn292562.aspx).</span></span>

<span data-ttu-id="340b9-398">A következő beállításokat javasoljuk az újrapróbálkozási műveletekhez.</span><span class="sxs-lookup"><span data-stu-id="340b9-398">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="340b9-399">Ezek általános célú beállítások, ezért javasoljuk, hogy monitorozza műveleteit, és finomhangolja az értékeket saját igényei szerint.</span><span class="sxs-lookup"><span data-stu-id="340b9-399">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="340b9-400">Környezet</span><span class="sxs-lookup"><span data-stu-id="340b9-400">Context</span></span> | <span data-ttu-id="340b9-401">Példa a maximális késésre</span><span class="sxs-lookup"><span data-stu-id="340b9-401">Example maximum latency</span></span> | <span data-ttu-id="340b9-402">Újrapróbálkozási szabályzat</span><span class="sxs-lookup"><span data-stu-id="340b9-402">Retry policy</span></span> | <span data-ttu-id="340b9-403">Beállítások</span><span class="sxs-lookup"><span data-stu-id="340b9-403">Settings</span></span> | <span data-ttu-id="340b9-404">Működés</span><span class="sxs-lookup"><span data-stu-id="340b9-404">How it works</span></span> |
|---------|---------|---------|---------|---------|
| <span data-ttu-id="340b9-405">Interaktív, felhasználói felület vagy előtér</span><span class="sxs-lookup"><span data-stu-id="340b9-405">Interactive, UI, or foreground</span></span> | <span data-ttu-id="340b9-406">2 másodperc\*</span><span class="sxs-lookup"><span data-stu-id="340b9-406">2 seconds\*</span></span>  | <span data-ttu-id="340b9-407">Exponenciális</span><span class="sxs-lookup"><span data-stu-id="340b9-407">Exponential</span></span> | <span data-ttu-id="340b9-408">MinimumBackoff = 0</span><span class="sxs-lookup"><span data-stu-id="340b9-408">MinimumBackoff = 0</span></span> <br/> <span data-ttu-id="340b9-409">MaximumBackoff = 30 mp.</span><span class="sxs-lookup"><span data-stu-id="340b9-409">MaximumBackoff = 30 sec.</span></span> <br/> <span data-ttu-id="340b9-410">DeltaBackoff = 300 ms</span><span class="sxs-lookup"><span data-stu-id="340b9-410">DeltaBackoff = 300 msec.</span></span> <br/> <span data-ttu-id="340b9-411">TimeBuffer = 300 ms</span><span class="sxs-lookup"><span data-stu-id="340b9-411">TimeBuffer = 300 msec.</span></span> <br/> <span data-ttu-id="340b9-412">MaxRetryCount = 2</span><span class="sxs-lookup"><span data-stu-id="340b9-412">MaxRetryCount = 2</span></span> | <span data-ttu-id="340b9-413">1. kísérlet: 0 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="340b9-413">Attempt 1: Delay 0 sec.</span></span> <br/> <span data-ttu-id="340b9-414">2. kísérlet: kb. 300 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="340b9-414">Attempt 2: Delay ~300 msec.</span></span> <br/> <span data-ttu-id="340b9-415">3. kísérlet: kb. 900 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="340b9-415">Attempt 3: Delay ~900 msec.</span></span> |
| <span data-ttu-id="340b9-416">Háttér vagy kötegelt</span><span class="sxs-lookup"><span data-stu-id="340b9-416">Background or batch</span></span> | <span data-ttu-id="340b9-417">30 másodperc</span><span class="sxs-lookup"><span data-stu-id="340b9-417">30 seconds</span></span> | <span data-ttu-id="340b9-418">Exponenciális</span><span class="sxs-lookup"><span data-stu-id="340b9-418">Exponential</span></span> | <span data-ttu-id="340b9-419">MinimumBackoff = 1</span><span class="sxs-lookup"><span data-stu-id="340b9-419">MinimumBackoff = 1</span></span> <br/> <span data-ttu-id="340b9-420">MaximumBackoff = 30 mp.</span><span class="sxs-lookup"><span data-stu-id="340b9-420">MaximumBackoff = 30 sec.</span></span> <br/> <span data-ttu-id="340b9-421">DeltaBackoff = 1,75 mp.</span><span class="sxs-lookup"><span data-stu-id="340b9-421">DeltaBackoff = 1.75 sec.</span></span> <br/> <span data-ttu-id="340b9-422">TimeBuffer = 5 mp.</span><span class="sxs-lookup"><span data-stu-id="340b9-422">TimeBuffer = 5 sec.</span></span> <br/> <span data-ttu-id="340b9-423">MaxRetryCount = 3</span><span class="sxs-lookup"><span data-stu-id="340b9-423">MaxRetryCount = 3</span></span> | <span data-ttu-id="340b9-424">1. kísérlet: kb. 1 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="340b9-424">Attempt 1: Delay ~1 sec.</span></span> <br/> <span data-ttu-id="340b9-425">2. kísérlet: kb. 3 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="340b9-425">Attempt 2: Delay ~3 sec.</span></span> <br/> <span data-ttu-id="340b9-426">3. kísérlet: kb. 6 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="340b9-426">Attempt 3: Delay ~6 msec.</span></span> <br/> <span data-ttu-id="340b9-427">4. kísérlet: kb. 13 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="340b9-427">Attempt 4: Delay ~13 msec.</span></span> |

<span data-ttu-id="340b9-428">\* Nem tartalmazza a további késleltetést, amely a „foglalt kiszolgáló” válasz esetén adódik az értékhez.</span><span class="sxs-lookup"><span data-stu-id="340b9-428">\* Not including additional delay that is added if a Server Busy response is received.</span></span>

### <a name="telemetry"></a><span data-ttu-id="340b9-429">Telemetria</span><span class="sxs-lookup"><span data-stu-id="340b9-429">Telemetry</span></span>
<span data-ttu-id="340b9-430">A Service Bus ETW-eseményként naplózza az újrapróbálkozásokat egy **EventSource** használatával.</span><span class="sxs-lookup"><span data-stu-id="340b9-430">Service Bus logs retries as ETW events using an **EventSource**.</span></span> <span data-ttu-id="340b9-431">Egy **EventListener** az eseményforráshoz csatolása szükséges, ha rögzíteni kívánja az eseményeket, és meg kívánja tekinteni azokat a Teljesítménynaplóban, vagy ha egy megfelelő célnaplóba írná azokat.</span><span class="sxs-lookup"><span data-stu-id="340b9-431">You must attach an **EventListener** to the event source to capture the events and view them in Performance Viewer, or write them to a suitable destination log.</span></span> <span data-ttu-id="340b9-432">Ehhez a [szemantikus naplózási alkalmazásblokk](http://msdn.microsoft.com/library/dn775006.aspx) használatát javasoljuk.</span><span class="sxs-lookup"><span data-stu-id="340b9-432">You could use the [Semantic Logging Application Block](http://msdn.microsoft.com/library/dn775006.aspx) to do this.</span></span> <span data-ttu-id="340b9-433">Az újrapróbálkozási események formátuma a következő:</span><span class="sxs-lookup"><span data-stu-id="340b9-433">The retry events are of the following form:</span></span>

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

### <a name="examples"></a><span data-ttu-id="340b9-434">Példák</span><span class="sxs-lookup"><span data-stu-id="340b9-434">Examples</span></span>
<span data-ttu-id="340b9-435">A következő mintakód bemutatja, hogyan állíthatja be az újrapróbálkozási szabályzatot a következőkhöz:</span><span class="sxs-lookup"><span data-stu-id="340b9-435">The following code example shows how to set the retry policy for:</span></span>

* <span data-ttu-id="340b9-436">Egy névtérkezelő.</span><span class="sxs-lookup"><span data-stu-id="340b9-436">A namespace manager.</span></span> <span data-ttu-id="340b9-437">A szabályzat a kezelő összes műveletére vonatkozik, és nem bírálható felül az egyes műveletek esetében.</span><span class="sxs-lookup"><span data-stu-id="340b9-437">The policy applies to all operations on that manager, and cannot be overridden for individual operations.</span></span>
* <span data-ttu-id="340b9-438">Egy üzenetkezelési előállító.</span><span class="sxs-lookup"><span data-stu-id="340b9-438">A messaging factory.</span></span> <span data-ttu-id="340b9-439">A szabályzat az előállítóból létrehozott összes ügyfélre vonatkozik, és nem bírálható felül az egyes ügyfelek létrehozásakor.</span><span class="sxs-lookup"><span data-stu-id="340b9-439">The policy applies to all clients created from that factory, and cannot be overridden when creating individual clients.</span></span>
* <span data-ttu-id="340b9-440">Egy különálló üzenetkezelési ügyfél.</span><span class="sxs-lookup"><span data-stu-id="340b9-440">An individual messaging client.</span></span> <span data-ttu-id="340b9-441">Az ügyfél létrehozása után beállíthatja annak újrapróbálkozási szabályzatát.</span><span class="sxs-lookup"><span data-stu-id="340b9-441">After a client has been created, you can set the retry policy for that client.</span></span> <span data-ttu-id="340b9-442">A szabályzat az ügyfél összes műveletére vonatkozik.</span><span class="sxs-lookup"><span data-stu-id="340b9-442">The policy applies to all operations on that client.</span></span>

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

### <a name="more-information"></a><span data-ttu-id="340b9-443">További információ</span><span class="sxs-lookup"><span data-stu-id="340b9-443">More information</span></span>
* [<span data-ttu-id="340b9-444">Aszinkron üzenetkezelési minták és magas rendelkezésre állás</span><span class="sxs-lookup"><span data-stu-id="340b9-444">Asynchronous Messaging Patterns and High Availability</span></span>](http://msdn.microsoft.com/library/azure/dn292562.aspx)

## <a name="service-fabric"></a><span data-ttu-id="340b9-445">Service Fabric</span><span class="sxs-lookup"><span data-stu-id="340b9-445">Service Fabric</span></span>

<span data-ttu-id="340b9-446">A megbízható szolgáltatások Service Fabric-fürtön belüli elosztásával a cikkben tárgyalt legtöbb átmeneti hiba elkerülhető.</span><span class="sxs-lookup"><span data-stu-id="340b9-446">Distributing reliable services in a Service Fabric cluster guards against most of the potential transient faults discussed in this article.</span></span> <span data-ttu-id="340b9-447">Azonban így is előfordulhatnak átmeneti hibák.</span><span class="sxs-lookup"><span data-stu-id="340b9-447">Some transient faults are still possible, however.</span></span> <span data-ttu-id="340b9-448">Előfordulhat például, hogy a kérés érkezésekor az elnevezési szolgáltatás egy útválasztási változtatást végez, ami kivételt eredményez.</span><span class="sxs-lookup"><span data-stu-id="340b9-448">For example, the naming service might be in the middle of a routing change when it gets a request, causing it to throw an exception.</span></span> <span data-ttu-id="340b9-449">Ugyanez a kérés 100 milliszekundummal később talán sikeres lett volna.</span><span class="sxs-lookup"><span data-stu-id="340b9-449">If the same request comes 100 milliseconds later, it will probably succeed.</span></span>

<span data-ttu-id="340b9-450">A Service Fabric az ehhez hasonló átmeneti hibák belső kezelését elvégzi.</span><span class="sxs-lookup"><span data-stu-id="340b9-450">Internally, Service Fabric manages this kind of transient fault.</span></span> <span data-ttu-id="340b9-451">A szolgáltatás beállítása közben konfigurálhat egyes beállításokat az `OperationRetrySettings` osztállyal.</span><span class="sxs-lookup"><span data-stu-id="340b9-451">You can configure some settings by using the `OperationRetrySettings` class while setting up your services.</span></span>  <span data-ttu-id="340b9-452">Az alábbi kód erre mutat egy példát.</span><span class="sxs-lookup"><span data-stu-id="340b9-452">The following code shows an example.</span></span> <span data-ttu-id="340b9-453">A legtöbb esetben ez nem szükséges, és az alapértelmezett beállítások tökéletesen megfelelnek.</span><span class="sxs-lookup"><span data-stu-id="340b9-453">In most cases, this should not be necessary, and the default settings will be fine.</span></span>

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

### <a name="more-information"></a><span data-ttu-id="340b9-454">További információ</span><span class="sxs-lookup"><span data-stu-id="340b9-454">More information</span></span>

* [<span data-ttu-id="340b9-455">Távoli kivételkezelés</span><span class="sxs-lookup"><span data-stu-id="340b9-455">Remote Exception Handling</span></span>](https://github.com/Microsoft/azure-docs/blob/master/articles/service-fabric/service-fabric-reliable-services-communication-remoting.md#remoting-exception-handling)

## <a name="sql-database-using-adonet"></a><span data-ttu-id="340b9-456">Az ADO.NET használatával SQL-adatbázis</span><span class="sxs-lookup"><span data-stu-id="340b9-456">SQL Database using ADO.NET</span></span>
<span data-ttu-id="340b9-457">Az SQL Database egy üzemeltetett SQL-adatbázis, amely különböző méretekben, normál (megosztott) és prémium (nem megosztott) szolgáltatásként is elérhető.</span><span class="sxs-lookup"><span data-stu-id="340b9-457">SQL Database is a hosted SQL database available in a range of sizes and as both a standard (shared) and premium (non-shared) service.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="340b9-458">Újrapróbálkozási mechanizmus</span><span class="sxs-lookup"><span data-stu-id="340b9-458">Retry mechanism</span></span>
<span data-ttu-id="340b9-459">Az SQL Database nem tartalmaz beépített támogatást az újrapróbálkozásokhoz, ha az ADO.NET használatával érik el.</span><span class="sxs-lookup"><span data-stu-id="340b9-459">SQL Database has no built-in support for retries when accessed using ADO.NET.</span></span> <span data-ttu-id="340b9-460">Ugyanakkor a kérések válaszkódjából megállapítható, hogy a kérés miért hiúsult meg.</span><span class="sxs-lookup"><span data-stu-id="340b9-460">However, the return codes from requests can be used to determine why a request failed.</span></span> <span data-ttu-id="340b9-461">További információt az SQL Database szabályozásáról [az Azure SQL Database erőforrás-korlátait](/azure/sql-database/sql-database-resource-limits) ismertető szakaszban talál.</span><span class="sxs-lookup"><span data-stu-id="340b9-461">For more information about SQL Database throttling, see [Azure SQL Database resource limits](/azure/sql-database/sql-database-resource-limits).</span></span> <span data-ttu-id="340b9-462">A kapcsolódó hibakódok listáját [az SQL Database ügyfélalkalmazásaiban felmerülő SQL-hibakódokat](/azure/sql-database/sql-database-develop-error-messages) ismertető szakaszban találja.</span><span class="sxs-lookup"><span data-stu-id="340b9-462">For a list of relevant error codes, see [SQL error codes for SQL Database client applications](/azure/sql-database/sql-database-develop-error-messages).</span></span>

<span data-ttu-id="340b9-463">A Polly kódtárt alkalmazva implementálhatja az újrapróbálkozást az SQL Database-ben.</span><span class="sxs-lookup"><span data-stu-id="340b9-463">You can use the Polly library to implement retries for SQL Database.</span></span> <span data-ttu-id="340b9-464">További információt az [átmeneti hibák a Polly használatával történő kezelését](#transient-fault-handling-with-polly) ismertető szakaszban talál.</span><span class="sxs-lookup"><span data-stu-id="340b9-464">See [Transient fault handling with Polly](#transient-fault-handling-with-polly).</span></span>

### <a name="retry-usage-guidance"></a><span data-ttu-id="340b9-465">Újrapróbálkozásokra vonatkozó útmutató</span><span class="sxs-lookup"><span data-stu-id="340b9-465">Retry usage guidance</span></span>
<span data-ttu-id="340b9-466">Ügyeljen a következőkre, amikor az ADO.NET használatával éri el az SQL Database-t:</span><span class="sxs-lookup"><span data-stu-id="340b9-466">Consider the following guidelines when accessing SQL Database using ADO.NET:</span></span>

* <span data-ttu-id="340b9-467">Válassza a megfelelő szolgáltatástípust (megosztott vagy prémium).</span><span class="sxs-lookup"><span data-stu-id="340b9-467">Choose the appropriate service option (shared or premium).</span></span> <span data-ttu-id="340b9-468">A megosztott példány esetében a szokottnál hosszabb csatlakozási késések fordulhatnak elő, valamint a kérések számának korlátozására lehet szükség, mivel a megosztott kiszolgálót más bérlők is használják.</span><span class="sxs-lookup"><span data-stu-id="340b9-468">A shared instance may suffer longer than usual connection delays and throttling due to the usage by other tenants of the shared server.</span></span> <span data-ttu-id="340b9-469">Ha kiszámíthatóbb teljesítményre és megbízhatóan alacsony késésű műveletekre van szükség, mindenképpen a prémium szolgáltatást érdemes választani.</span><span class="sxs-lookup"><span data-stu-id="340b9-469">If more predictable performance and reliable low latency operations are required, consider choosing the premium option.</span></span>
* <span data-ttu-id="340b9-470">Gondoskodjon arról, hogy az újrapróbálkozás a megfelelő szinten vagy hatókörrel legyen végrehajtva, amivel elkerülheti, hogy a nem idempotens műveletek miatt inkonzisztencia keletkezzen az adatokban.</span><span class="sxs-lookup"><span data-stu-id="340b9-470">Ensure that you perform retries at the appropriate level or scope to avoid non-idempotent operations causing inconsistency in the data.</span></span> <span data-ttu-id="340b9-471">Ideális esetben minden műveletnek idempotensnek kellene lennie, hogy inkonzisztencia veszélye nélkül lehessen ismételni azokat.</span><span class="sxs-lookup"><span data-stu-id="340b9-471">Ideally, all operations should be idempotent so that they can be repeated without causing inconsistency.</span></span> <span data-ttu-id="340b9-472">Ellenkező esetben az újrapróbálkozást olyan szinten vagy hatókörrel kell végrehajtani, hogy az összes kapcsolódó változtatást vissza lehessen vonni egy művelet meghiúsulásakor – például egy tranzakciós hatókörben.</span><span class="sxs-lookup"><span data-stu-id="340b9-472">Where this is not the case, the retry should be performed at a level or scope that allows all related changes to be undone if one operation fails; for example, from within a transactional scope.</span></span> <span data-ttu-id="340b9-473">További információt a [Cloud Service Fundamentals adatelérési réteg átmeneti hibák kezelésével](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx#Idempotent_Guarantee) foglalkozó részében talál.</span><span class="sxs-lookup"><span data-stu-id="340b9-473">For more information, see [Cloud Service Fundamentals Data Access Layer – Transient Fault Handling](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx#Idempotent_Guarantee).</span></span>
* <span data-ttu-id="340b9-474">Az Azure SQL Database esetében nem javasoljuk rögzített időközű stratégiák alkalmazását. Az interaktív forgatókönyvek kivételt képeznek, mivel csak néhány újrapróbálkozás történik nagyon rövid időközökkel.</span><span class="sxs-lookup"><span data-stu-id="340b9-474">A fixed interval strategy is not recommended for use with Azure SQL Database except for interactive scenarios where there are only a few retries at very short intervals.</span></span> <span data-ttu-id="340b9-475">Ehelyett a legtöbb esetben exponenciális visszatartási stratégia használata javasolt.</span><span class="sxs-lookup"><span data-stu-id="340b9-475">Instead, consider using an exponential back-off strategy for the majority of scenarios.</span></span>
* <span data-ttu-id="340b9-476">A kapcsolatok definiálásakor válasszon megfelelő értéket a kapcsolatok és a parancsok időtúllépéséhez.</span><span class="sxs-lookup"><span data-stu-id="340b9-476">Choose a suitable value for the connection and command timeouts when defining connections.</span></span> <span data-ttu-id="340b9-477">A túl alacsony időtúllépési érték miatt a kapcsolatok idő előtt szakadhatnak meg, ha az adatbázis leterhelt.</span><span class="sxs-lookup"><span data-stu-id="340b9-477">Too short a timeout may result in premature failures of connections when the database is busy.</span></span> <span data-ttu-id="340b9-478">A túl magas időtúllépési érték akadályozhatja az újrapróbálkozási logika megfelelő működését, mivel túl sokáig fog várni, mielőtt észlelné a sikertelen csatlakozást.</span><span class="sxs-lookup"><span data-stu-id="340b9-478">Too long a timeout may prevent the retry logic working correctly by waiting too long before detecting a failed connection.</span></span> <span data-ttu-id="340b9-479">Az időtúllépés értéke a végpontok közötti késés egyik összetevője. Gyakorlatilag hozzáadódik az újrapróbálkozási szabályzatban megadott újrapróbálkozási késleltetéshez minden egyes újrapróbálkozási kísérlet esetén.</span><span class="sxs-lookup"><span data-stu-id="340b9-479">The value of the timeout is a component of the end-to-end latency; it is effectively added to the retry delay specified in the retry policy for every retry attempt.</span></span>
* <span data-ttu-id="340b9-480">Megszakítja a kapcsolatot adott számú újrapróbálkozás után, akár egy exponenciális visszatartási újrapróbálkozási logika használata esetén is, és egy új kapcsolatot létesítve próbálja meg újra végrehajtani a műveletet.</span><span class="sxs-lookup"><span data-stu-id="340b9-480">Close the connection after a certain number of retries, even when using an exponential back off retry logic, and retry the operation on a new connection.</span></span> <span data-ttu-id="340b9-481">Ha ugyanazzal a művelettel többször próbálkozik újra ugyanazon a kapcsolaton keresztül, az önmagában is csatlakozási problémákat okozhat.</span><span class="sxs-lookup"><span data-stu-id="340b9-481">Retrying the same operation multiple times on the same connection can be a factor that contributes to connection problems.</span></span> <span data-ttu-id="340b9-482">Erre a technikára egy példát a [Cloud Service Fundamentals adatelérési réteg átmeneti hibák kezelésével](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx) foglalkozó részében találhat.</span><span class="sxs-lookup"><span data-stu-id="340b9-482">For an example of this technique, see [Cloud Service Fundamentals Data Access Layer – Transient Fault Handling](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx).</span></span>
* <span data-ttu-id="340b9-483">Ha kapcsolatkészletezést alkalmaz (ez az alapértelmezett beállítás), akkor előfordulhat, hogy a készletből ismét ugyanazt a kapcsolatot választja a rendszer, akár a kapcsolat lezárása és ismételt megnyitása után is.</span><span class="sxs-lookup"><span data-stu-id="340b9-483">When connection pooling is in use (the default) there is a chance that the same connection will be chosen from the pool, even after closing and reopening a connection.</span></span> <span data-ttu-id="340b9-484">Ebben az esetben az **SqlConnection** osztályból kell meghívni a **ClearPool** metódust, és jelezni, hogy a kapcsolat nem újrahasználható.</span><span class="sxs-lookup"><span data-stu-id="340b9-484">If this is the case, a technique to resolve it is to call the **ClearPool** method of the **SqlConnection** class to mark the connection as not reusable.</span></span> <span data-ttu-id="340b9-485">Ezt azonban csak abban az esetben javasoljuk, ha számos csatlakozási kísérlet meghiúsult, és csak akkor, ha az átmeneti hibák, például SQL-időtúllépések (-2-es hibakód) adott osztálya hibás kapcsolatokhoz kötődik.</span><span class="sxs-lookup"><span data-stu-id="340b9-485">However, you should do this only after several connection attempts have failed, and only when encountering the specific class of transient failures such as SQL timeouts (error code -2) related to faulty connections.</span></span>
* <span data-ttu-id="340b9-486">Amennyiben az adatelérési kód **TransactionScope**-példányként kezdeményezett tranzakciókat alkalmaz, az újrapróbálkozási logikának újra meg kell nyitnia a kapcsolatot, és új tranzakció-hatókört kell kezdeményeznie.</span><span class="sxs-lookup"><span data-stu-id="340b9-486">If the data access code uses transactions initiated as **TransactionScope** instances, the retry logic should reopen the connection and initiate a new transaction scope.</span></span> <span data-ttu-id="340b9-487">Ezért az újrapróbálható kódblokknak a tranzakció teljes hatókörét le kell fedne.</span><span class="sxs-lookup"><span data-stu-id="340b9-487">For this reason, the retryable code block should encompass the entire scope of the transaction.</span></span>

<span data-ttu-id="340b9-488">A következő beállításokat javasoljuk az újrapróbálkozási műveletekhez.</span><span class="sxs-lookup"><span data-stu-id="340b9-488">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="340b9-489">Ezek általános célú beállítások, ezért javasoljuk, hogy monitorozza műveleteit, és finomhangolja az értékeket saját igényei szerint.</span><span class="sxs-lookup"><span data-stu-id="340b9-489">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="340b9-490">**Környezet**</span><span class="sxs-lookup"><span data-stu-id="340b9-490">**Context**</span></span> | <span data-ttu-id="340b9-491">**Mintacél E2E<br />max. késleltetése**</span><span class="sxs-lookup"><span data-stu-id="340b9-491">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="340b9-492">**Újrapróbálkozási stratégia**</span><span class="sxs-lookup"><span data-stu-id="340b9-492">**Retry strategy**</span></span> | <span data-ttu-id="340b9-493">**Beállítások**</span><span class="sxs-lookup"><span data-stu-id="340b9-493">**Settings**</span></span> | <span data-ttu-id="340b9-494">**Értékek**</span><span class="sxs-lookup"><span data-stu-id="340b9-494">**Values**</span></span> | <span data-ttu-id="340b9-495">**Működési elv**</span><span class="sxs-lookup"><span data-stu-id="340b9-495">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="340b9-496">Interaktív, felhasználói felület</span><span class="sxs-lookup"><span data-stu-id="340b9-496">Interactive, UI,</span></span><br /><span data-ttu-id="340b9-497">vagy előtér</span><span class="sxs-lookup"><span data-stu-id="340b9-497">or foreground</span></span> |<span data-ttu-id="340b9-498">2 másodperc</span><span class="sxs-lookup"><span data-stu-id="340b9-498">2 sec</span></span> |<span data-ttu-id="340b9-499">FixedInterval</span><span class="sxs-lookup"><span data-stu-id="340b9-499">FixedInterval</span></span> |<span data-ttu-id="340b9-500">Ismétlések száma</span><span class="sxs-lookup"><span data-stu-id="340b9-500">Retry count</span></span><br /><span data-ttu-id="340b9-501">Újrapróbálkozási időköz</span><span class="sxs-lookup"><span data-stu-id="340b9-501">Retry interval</span></span><br /><span data-ttu-id="340b9-502">Első gyors újrapróbálkozás</span><span class="sxs-lookup"><span data-stu-id="340b9-502">First fast retry</span></span> |<span data-ttu-id="340b9-503">3</span><span class="sxs-lookup"><span data-stu-id="340b9-503">3</span></span><br /><span data-ttu-id="340b9-504">500 ms</span><span class="sxs-lookup"><span data-stu-id="340b9-504">500 ms</span></span><br /><span data-ttu-id="340b9-505">true</span><span class="sxs-lookup"><span data-stu-id="340b9-505">true</span></span> |<span data-ttu-id="340b9-506">1. kísérlet – 0 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="340b9-506">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="340b9-507">2. kísérlet – 500 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="340b9-507">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="340b9-508">3. kísérlet – 500 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="340b9-508">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="340b9-509">Háttér</span><span class="sxs-lookup"><span data-stu-id="340b9-509">Background</span></span><br /><span data-ttu-id="340b9-510">vagy kötegelt</span><span class="sxs-lookup"><span data-stu-id="340b9-510">or batch</span></span> |<span data-ttu-id="340b9-511">30 másodperc</span><span class="sxs-lookup"><span data-stu-id="340b9-511">30 sec</span></span> |<span data-ttu-id="340b9-512">ExponentialBackoff</span><span class="sxs-lookup"><span data-stu-id="340b9-512">ExponentialBackoff</span></span> |<span data-ttu-id="340b9-513">Ismétlések száma</span><span class="sxs-lookup"><span data-stu-id="340b9-513">Retry count</span></span><br /><span data-ttu-id="340b9-514">Visszatartás (min.)</span><span class="sxs-lookup"><span data-stu-id="340b9-514">Min back-off</span></span><br /><span data-ttu-id="340b9-515">Visszatartás (max.)</span><span class="sxs-lookup"><span data-stu-id="340b9-515">Max back-off</span></span><br /><span data-ttu-id="340b9-516">Visszatartás (változás)</span><span class="sxs-lookup"><span data-stu-id="340b9-516">Delta back-off</span></span><br /><span data-ttu-id="340b9-517">Első gyors újrapróbálkozás</span><span class="sxs-lookup"><span data-stu-id="340b9-517">First fast retry</span></span> |<span data-ttu-id="340b9-518">5</span><span class="sxs-lookup"><span data-stu-id="340b9-518">5</span></span><br /><span data-ttu-id="340b9-519">0 másodperc</span><span class="sxs-lookup"><span data-stu-id="340b9-519">0 sec</span></span><br /><span data-ttu-id="340b9-520">60 másodperc</span><span class="sxs-lookup"><span data-stu-id="340b9-520">60 sec</span></span><br /><span data-ttu-id="340b9-521">2 másodperc</span><span class="sxs-lookup"><span data-stu-id="340b9-521">2 sec</span></span><br /><span data-ttu-id="340b9-522">false</span><span class="sxs-lookup"><span data-stu-id="340b9-522">false</span></span> |<span data-ttu-id="340b9-523">1. kísérlet – 0 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="340b9-523">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="340b9-524">2. kísérlet – kb. 2 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="340b9-524">Attempt 2 - delay ~2 sec</span></span><br /><span data-ttu-id="340b9-525">3. kísérlet – kb. 6 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="340b9-525">Attempt 3 - delay ~6 sec</span></span><br /><span data-ttu-id="340b9-526">4. kísérlet – kb. 14 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="340b9-526">Attempt 4 - delay ~14 sec</span></span><br /><span data-ttu-id="340b9-527">5. kísérlet – kb. 30 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="340b9-527">Attempt 5 - delay ~30 sec</span></span> |

> [!NOTE]
> <span data-ttu-id="340b9-528">A végpontok közötti késés célértéke az alapértelmezett időtúllépési érték használatát feltételezi a szolgáltatáskapcsolatokhoz.</span><span class="sxs-lookup"><span data-stu-id="340b9-528">The end-to-end latency targets assume the default timeout for connections to the service.</span></span> <span data-ttu-id="340b9-529">Ha magasabb értéket ad meg a kapcsolatok időtúllépésére, a végpontok közötti késés minden újrapróbálkozási kísérlet esetében ennyivel lesz meghosszabbítva.</span><span class="sxs-lookup"><span data-stu-id="340b9-529">If you specify longer connection timeouts, the end-to-end latency will be extended by this additional time for every retry attempt.</span></span>
>
>

### <a name="examples"></a><span data-ttu-id="340b9-530">Példák</span><span class="sxs-lookup"><span data-stu-id="340b9-530">Examples</span></span>
<span data-ttu-id="340b9-531">Ebben a szakaszban azt mutatjuk be, hogy miként használhatja a Pollyt az Azure SQL Database elérésére a `Policy` osztályban konfigurált újrapróbálkozási szabályzatok segítségével.</span><span class="sxs-lookup"><span data-stu-id="340b9-531">This section shows how you can use Polly to access Azure SQL Database using a set of retry policies configured in the `Policy` class.</span></span>

<span data-ttu-id="340b9-532">A következő programkód bemutatja, hogyan bővítheti az `SqlCommand` osztályt, ha az exponenciális visszatartással hívja meg az `ExecuteAsync` metódust.</span><span class="sxs-lookup"><span data-stu-id="340b9-532">The following code shows an extension method on the `SqlCommand` class that calls `ExecuteAsync` with exponential backoff.</span></span>

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

<span data-ttu-id="340b9-533">Ezt az aszinkron bővítési metódust a következőképpen lehet használni.</span><span class="sxs-lookup"><span data-stu-id="340b9-533">This asynchronous extension method can be used as follows.</span></span>

```csharp
var sqlCommand = sqlConnection.CreateCommand();
sqlCommand.CommandText = "[some query]";

using (var reader = await sqlCommand.ExecuteReaderWithRetryAsync())
{
    // Do something with the values
}
```

### <a name="more-information"></a><span data-ttu-id="340b9-534">További információ</span><span class="sxs-lookup"><span data-stu-id="340b9-534">More information</span></span>
* [<span data-ttu-id="340b9-535">Cloud Service Fundamentals adatelérési réteg – átmeneti hibák kezelése</span><span class="sxs-lookup"><span data-stu-id="340b9-535">Cloud Service Fundamentals Data Access Layer – Transient Fault Handling</span></span>](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx)

<span data-ttu-id="340b9-536">Ha további útmutatásra van szüksége az SQL Database minél jobb kihasználásához, olvassa el [az Azure SQL Database szolgáltatás teljesítményének és rugalmasságának útmutatóját](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx).</span><span class="sxs-lookup"><span data-stu-id="340b9-536">For general guidance on getting the most from SQL Database, see [Azure SQL Database Performance and Elasticity Guide](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx).</span></span>

## <a name="sql-database-using-entity-framework-6"></a><span data-ttu-id="340b9-537">SQL-adatbázis használata az Entity Framework 6</span><span class="sxs-lookup"><span data-stu-id="340b9-537">SQL Database using Entity Framework 6</span></span>
<span data-ttu-id="340b9-538">Az SQL Database egy üzemeltetett SQL-adatbázis, amely különböző méretekben, normál (megosztott) és prémium (nem megosztott) szolgáltatásként is elérhető.</span><span class="sxs-lookup"><span data-stu-id="340b9-538">SQL Database is a hosted SQL database available in a range of sizes and as both a standard (shared) and premium (non-shared) service.</span></span> <span data-ttu-id="340b9-539">Az Entity Framework egy objektumrelációs leképező .NET-fejlesztők számára, amellyel tartományspecifikus objektumokat használva lehet relációs adatokkal dolgozni.</span><span class="sxs-lookup"><span data-stu-id="340b9-539">Entity Framework is an object-relational mapper that enables .NET developers to work with relational data using domain-specific objects.</span></span> <span data-ttu-id="340b9-540">Szükségtelenné teszi az adatelérési kód használatát, amelyet egyébként a fejlesztőknek kell megírniuk.</span><span class="sxs-lookup"><span data-stu-id="340b9-540">It eliminates the need for most of the data-access code that developers usually need to write.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="340b9-541">Újrapróbálkozási mechanizmus</span><span class="sxs-lookup"><span data-stu-id="340b9-541">Retry mechanism</span></span>
<span data-ttu-id="340b9-542">Az újrapróbálkozás akkor támogatott, ha az SQL Database-t az Entity Framework 6.0-s vagy újabb verziójával érik el, a [kapcsolat rugalmassága/újrapróbálkozási logika](http://msdn.microsoft.com/data/dn456835.aspx) mechanizmus segítségével.</span><span class="sxs-lookup"><span data-stu-id="340b9-542">Retry support is provided when accessing SQL Database using Entity Framework 6.0 and higher through a mechanism called [Connection Resiliency / Retry Logic](http://msdn.microsoft.com/data/dn456835.aspx).</span></span> <span data-ttu-id="340b9-543">Az újrapróbálkozási mechanizmus fő funkciói a következők:</span><span class="sxs-lookup"><span data-stu-id="340b9-543">The main features of the retry mechanism are:</span></span>

* <span data-ttu-id="340b9-544">Az elsődleges absztrakció az **IDbExecutionStrategy** felület.</span><span class="sxs-lookup"><span data-stu-id="340b9-544">The primary abstraction is the **IDbExecutionStrategy** interface.</span></span> <span data-ttu-id="340b9-545">Ez a felület:</span><span class="sxs-lookup"><span data-stu-id="340b9-545">This interface:</span></span>
  * <span data-ttu-id="340b9-546">Definiálja a szinkron és aszinkron **feldolgozási**\* metódusokat.</span><span class="sxs-lookup"><span data-stu-id="340b9-546">Defines synchronous and asynchronous **Execute**\* methods.</span></span>
  * <span data-ttu-id="340b9-547">Definiálja azokat az osztályokat, amelyek felhasználhatók közvetlenül, illetve konfigurálhatók alapértelmezett stratégiaként egy adatbázis-környezetben, leképezhetők egy szolgáltatónévre, vagy pedig leképezhetők egy szolgáltatónévre vagy kiszolgálónévre.</span><span class="sxs-lookup"><span data-stu-id="340b9-547">Defines classes that can be used directly or can be configured on a database context as a default strategy, mapped to provider name, or mapped to a provider name and server name.</span></span> <span data-ttu-id="340b9-548">Ha egy környezethez konfigurálják, az újrapróbálkozások az egyes adatbázis-műveletek szintjén történnek, amelyekből több is lehet egy adott környezeti művelet esetében.</span><span class="sxs-lookup"><span data-stu-id="340b9-548">When configured on a context, retries occur at the level of individual database operations, of which there might be several for a given context operation.</span></span>
  * <span data-ttu-id="340b9-549">Definiálja, hogy a sikertelen csatlakozást mikor kövesse újrapróbálkozás, és hogyan.</span><span class="sxs-lookup"><span data-stu-id="340b9-549">Defines when to retry a failed connection, and how.</span></span>
* <span data-ttu-id="340b9-550">Az **IDbExecutionStrategy** felület számos beépített implementálását tartalmazza:</span><span class="sxs-lookup"><span data-stu-id="340b9-550">It includes several built-in implementations of the **IDbExecutionStrategy** interface:</span></span>
  * <span data-ttu-id="340b9-551">Alapértelmezett – nincs újrapróbálkozás.</span><span class="sxs-lookup"><span data-stu-id="340b9-551">Default - no retrying.</span></span>
  * <span data-ttu-id="340b9-552">Az SQL Database alapértelmezett beállítása (automatikus) – nincs újrapróbálkozás, de megvizsgálja a kivételeket, és beburkolja azokat, az SQL Database stratégia használatát javasolva.</span><span class="sxs-lookup"><span data-stu-id="340b9-552">Default for SQL Database (automatic) - no retrying, but inspects exceptions and wraps them with suggestion to use the SQL Database strategy.</span></span>
  * <span data-ttu-id="340b9-553">Az SQL Database alapértelmezett beállítása – exponenciális (az alaposztálytól örökölt), plusz az SQL Database észlelési logikája.</span><span class="sxs-lookup"><span data-stu-id="340b9-553">Default for SQL Database - exponential (inherited from base class) plus SQL Database detection logic.</span></span>
* <span data-ttu-id="340b9-554">Véletlenszerűsítést tartalmazó exponenciális visszatartási stratégiát implementál.</span><span class="sxs-lookup"><span data-stu-id="340b9-554">It implements an exponential back-off strategy that includes randomization.</span></span>
* <span data-ttu-id="340b9-555">A beépített újrapróbálkozási osztályok állapotalapúak, és nem alkalmasak a többszálú futtatásra.</span><span class="sxs-lookup"><span data-stu-id="340b9-555">The built-in retry classes are stateful and are not thread safe.</span></span> <span data-ttu-id="340b9-556">Ugyanakkor az aktuális művelet befejezése után újra felhasználhatók.</span><span class="sxs-lookup"><span data-stu-id="340b9-556">However, they can be reused after the current operation is completed.</span></span>
* <span data-ttu-id="340b9-557">Amennyiben az újrapróbálkozások száma meghaladja a megadott értéket, a szolgáltatás új kivételbe burkolja az eredményeket.</span><span class="sxs-lookup"><span data-stu-id="340b9-557">If the specified retry count is exceeded, the results are wrapped in a new exception.</span></span> <span data-ttu-id="340b9-558">Nem rendezi buborékba az aktuális kivételt.</span><span class="sxs-lookup"><span data-stu-id="340b9-558">It does not bubble up the current exception.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="340b9-559">Szabályzatkonfiguráció</span><span class="sxs-lookup"><span data-stu-id="340b9-559">Policy configuration</span></span>
<span data-ttu-id="340b9-560">Az újrapróbálkozás akkor támogatott, ha az SQL Database-t az Entity Framework 6.0-s vagy újabb verziójával érik el.</span><span class="sxs-lookup"><span data-stu-id="340b9-560">Retry support is provided when accessing SQL Database using Entity Framework 6.0 and higher.</span></span> <span data-ttu-id="340b9-561">Az újrapróbálkozási szabályzatok konfigurálása szoftveresen történik.</span><span class="sxs-lookup"><span data-stu-id="340b9-561">Retry policies are configured programmatically.</span></span> <span data-ttu-id="340b9-562">A konfiguráció nem módosítható az egyes műveletek szintjén.</span><span class="sxs-lookup"><span data-stu-id="340b9-562">The configuration cannot be changed on a per-operation basis.</span></span>

<span data-ttu-id="340b9-563">Ha egy környezetfüggő stratégiát tesz alapértelmezetté, megad egy függvényt, amely igény szerint hoz létre egy új stratégiát.</span><span class="sxs-lookup"><span data-stu-id="340b9-563">When configuring a strategy on the context as the default, you specify a function that creates a new strategy on demand.</span></span> <span data-ttu-id="340b9-564">A következő kód azt mutatja be, hogyan hozható létre egy újrapróbálkozási konfigurációs osztályt, amely kibővíti a **DbConfiguration** alaposztályt.</span><span class="sxs-lookup"><span data-stu-id="340b9-564">The following code shows how you can create a retry configuration class that extends the **DbConfiguration** base class.</span></span>

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

<span data-ttu-id="340b9-565">Az alkalmazás indításakor, a **DbConfiguration**-példány **SetConfiguration** metódusával teheti az alapértelmezett újrapróbálkozási stratégiává minden művelet számára.</span><span class="sxs-lookup"><span data-stu-id="340b9-565">You can then specify this as the default retry strategy for all operations using the **SetConfiguration** method of the **DbConfiguration** instance when the application starts.</span></span> <span data-ttu-id="340b9-566">Alapértelmezés szerint az EF automatikusan észleli és használatba veszi a konfigurációs osztályt.</span><span class="sxs-lookup"><span data-stu-id="340b9-566">By default, EF will automatically discover and use the configuration class.</span></span>

```csharp
DbConfiguration.SetConfiguration(new BloggingContextConfiguration());
```

<span data-ttu-id="340b9-567">Ha meg kíván adni egy újrapróbálkozási konfigurációs osztályt a környezethez, lássa el a környezeti osztályt egy **DbConfigurationType** attribútummal.</span><span class="sxs-lookup"><span data-stu-id="340b9-567">You can specify the retry configuration class for a context by annotating the context class with a **DbConfigurationType** attribute.</span></span> <span data-ttu-id="340b9-568">Amennyiben csak egy konfigurációs osztálya van, az EF azt fogja használni, nem szükséges külön jeleznie ezt.</span><span class="sxs-lookup"><span data-stu-id="340b9-568">However, if you have only one configuration class, EF will use it without the need to annotate the context.</span></span>

```csharp
[DbConfigurationType(typeof(BloggingContextConfiguration))]
public class BloggingContext : DbContext
```

<span data-ttu-id="340b9-569">Ha eltérő újrapróbálkozási stratégiára van szüksége adott műveletek esetében, vagy le kívánja tiltani az újrapróbálkozásokat egyes műveleteknél, létrehozhat egy konfigurációs osztályt, amely lehetővé teszi stratégiák felfüggesztését vagy cseréjét egy jelző a **CallContext** osztályban történő elhelyezésével.</span><span class="sxs-lookup"><span data-stu-id="340b9-569">If you need to use different retry strategies for specific operations, or disable retries for specific operations, you can create a configuration class that allows you to suspend or swap strategies by setting a flag in the **CallContext**.</span></span> <span data-ttu-id="340b9-570">A konfigurációs osztály ezt a jelzőt használva vált stratégiát vagy tiltja le a felhasználó által megadott stratégiát, majd az alapértelmezett stratégiát használja.</span><span class="sxs-lookup"><span data-stu-id="340b9-570">The configuration class can use this flag to switch strategies, or disable the strategy you provide and use a default strategy.</span></span> <span data-ttu-id="340b9-571">További információt a végrehajtási stratégiák újrapróbálkozásainak korlátozásait ismertető oldal [végrehajtási stratégia felfüggesztését](http://msdn.microsoft.com/dn307226#transactions_workarounds) ismertető szakaszában talál (EF6-os vagy újabb verzió).</span><span class="sxs-lookup"><span data-stu-id="340b9-571">For more information, see [Suspend Execution Strategy](http://msdn.microsoft.com/dn307226#transactions_workarounds) in the page Limitations with Retrying Execution Strategies (EF6 onwards).</span></span>

<span data-ttu-id="340b9-572">Bizonyos műveletek esetében egy másik lehetőség adott újrapróbálkozási stratégia használatára, ha létrehozza a kívánt stratégiaosztály egy példányát, és paramétereket használva ellátja a kívánt beállításokkal.</span><span class="sxs-lookup"><span data-stu-id="340b9-572">Another technique for using specific retry strategies for individual operations is to create an instance of the required strategy class and supply the desired settings through parameters.</span></span> <span data-ttu-id="340b9-573">Ezután meghívja annak **ExecuteAsync** metódusát.</span><span class="sxs-lookup"><span data-stu-id="340b9-573">You then invoke its **ExecuteAsync** method.</span></span>

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

<span data-ttu-id="340b9-574">A **DbConfiguration** osztály használatának legegyszerűbb módja, ha megkeresi ugyanazt a szerelvényt, amely a **DbContext** osztályban szerepel.</span><span class="sxs-lookup"><span data-stu-id="340b9-574">The simplest way to use a **DbConfiguration** class is to locate it in the same assembly as the **DbContext** class.</span></span> <span data-ttu-id="340b9-575">Ez azonban nem alkalmazható, ha különböző forgatókönyvekben van szükség ugyanarra a környezetre, például a különböző interaktív és háttérbeli újrapróbálkozási stratégiák esetében.</span><span class="sxs-lookup"><span data-stu-id="340b9-575">However, this is not appropriate when the same context is required in different scenarios, such as different interactive and background retry strategies.</span></span> <span data-ttu-id="340b9-576">Ha a különböző környezetek különálló alkalmazástartományokban futnak, igénybe veheti a konfigurációs osztályok a konfigurációs fájlban történő megadásának beépített lehetőségét, illetve beállíthatja azt közvetlenül is a kódban.</span><span class="sxs-lookup"><span data-stu-id="340b9-576">If the different contexts execute in separate AppDomains, you can use the built-in support for specifying configuration classes in the configuration file or set it explicitly using code.</span></span> <span data-ttu-id="340b9-577">Ha a különböző környezeteknek ugyanabban az alkalmazástartományban kell futniuk, egyéni megoldásra lesz szükség.</span><span class="sxs-lookup"><span data-stu-id="340b9-577">If the different contexts must execute in the same AppDomain, a custom solution will be required.</span></span>

<span data-ttu-id="340b9-578">További információt a [kódalapú konfigurálást (EF6-os vagy újabb verzió)](http://msdn.microsoft.com/data/jj680699.aspx) ismertető szakaszban találhat.</span><span class="sxs-lookup"><span data-stu-id="340b9-578">For more information, see [Code-Based Configuration (EF6 onwards)](http://msdn.microsoft.com/data/jj680699.aspx).</span></span>

<span data-ttu-id="340b9-579">A következő táblázatban a beépített újrapróbálkozási szabályzat alapértelmezett beállításai láthatók az EF6 használatakor.</span><span class="sxs-lookup"><span data-stu-id="340b9-579">The following table shows the default settings for the built-in retry policy when using EF6.</span></span>

| <span data-ttu-id="340b9-580">Beállítás</span><span class="sxs-lookup"><span data-stu-id="340b9-580">Setting</span></span> | <span data-ttu-id="340b9-581">Alapértelmezett érték</span><span class="sxs-lookup"><span data-stu-id="340b9-581">Default value</span></span> | <span data-ttu-id="340b9-582">Jelentés</span><span class="sxs-lookup"><span data-stu-id="340b9-582">Meaning</span></span> |
|---------|---------------|---------|
| <span data-ttu-id="340b9-583">Szabályzat</span><span class="sxs-lookup"><span data-stu-id="340b9-583">Policy</span></span> | <span data-ttu-id="340b9-584">Exponenciális</span><span class="sxs-lookup"><span data-stu-id="340b9-584">Exponential</span></span> | <span data-ttu-id="340b9-585">Exponenciális visszatartás.</span><span class="sxs-lookup"><span data-stu-id="340b9-585">Exponential back-off.</span></span> |
| <span data-ttu-id="340b9-586">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="340b9-586">MaxRetryCount</span></span> | <span data-ttu-id="340b9-587">5</span><span class="sxs-lookup"><span data-stu-id="340b9-587">5</span></span> | <span data-ttu-id="340b9-588">Az újrapróbálkozások maximális száma.</span><span class="sxs-lookup"><span data-stu-id="340b9-588">The maximum number of retries.</span></span> |
| <span data-ttu-id="340b9-589">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="340b9-589">MaxDelay</span></span> | <span data-ttu-id="340b9-590">30 másodperc</span><span class="sxs-lookup"><span data-stu-id="340b9-590">30 seconds</span></span> | <span data-ttu-id="340b9-591">Az újrapróbálkozások közötti maximális késleltetés.</span><span class="sxs-lookup"><span data-stu-id="340b9-591">The maximum delay between retries.</span></span> <span data-ttu-id="340b9-592">Az érték nem befolyásolja az egymást követő késleltetések kiszámítását.</span><span class="sxs-lookup"><span data-stu-id="340b9-592">This value does not affect how the series of delays are computed.</span></span> <span data-ttu-id="340b9-593">Csak a felső határt adja meg.</span><span class="sxs-lookup"><span data-stu-id="340b9-593">It only defines an upper bound.</span></span> |
| <span data-ttu-id="340b9-594">DefaultCoefficient</span><span class="sxs-lookup"><span data-stu-id="340b9-594">DefaultCoefficient</span></span> | <span data-ttu-id="340b9-595">1 másodperc</span><span class="sxs-lookup"><span data-stu-id="340b9-595">1 second</span></span> | <span data-ttu-id="340b9-596">Az exponenciális visszatartás számításának együtthatója.</span><span class="sxs-lookup"><span data-stu-id="340b9-596">The coefficient for the exponential back-off computation.</span></span> <span data-ttu-id="340b9-597">Ez az érték nem módosítható.</span><span class="sxs-lookup"><span data-stu-id="340b9-597">This value cannot be changed.</span></span> |
| <span data-ttu-id="340b9-598">DefaultRandomFactor</span><span class="sxs-lookup"><span data-stu-id="340b9-598">DefaultRandomFactor</span></span> | <span data-ttu-id="340b9-599">1.1</span><span class="sxs-lookup"><span data-stu-id="340b9-599">1.1</span></span> | <span data-ttu-id="340b9-600">A véletlenszerű késleltetés bejegyzésekhez való hozzáadására szolgáló szorzó.</span><span class="sxs-lookup"><span data-stu-id="340b9-600">The multiplier used to add a random delay for each entry.</span></span> <span data-ttu-id="340b9-601">Ez az érték nem módosítható.</span><span class="sxs-lookup"><span data-stu-id="340b9-601">This value cannot be changed.</span></span> |
| <span data-ttu-id="340b9-602">DefaultExponentialBase</span><span class="sxs-lookup"><span data-stu-id="340b9-602">DefaultExponentialBase</span></span> | <span data-ttu-id="340b9-603">2</span><span class="sxs-lookup"><span data-stu-id="340b9-603">2</span></span> | <span data-ttu-id="340b9-604">A következő késleltetés kiszámítására szolgáló szorzó.</span><span class="sxs-lookup"><span data-stu-id="340b9-604">The multiplier used to calculate the next delay.</span></span> <span data-ttu-id="340b9-605">Ez az érték nem módosítható.</span><span class="sxs-lookup"><span data-stu-id="340b9-605">This value cannot be changed.</span></span> |

### <a name="retry-usage-guidance"></a><span data-ttu-id="340b9-606">Újrapróbálkozásokra vonatkozó útmutató</span><span class="sxs-lookup"><span data-stu-id="340b9-606">Retry usage guidance</span></span>
<span data-ttu-id="340b9-607">Ügyeljen a következőkre, amikor az EF6 használatával éri el az SQL Database-t:</span><span class="sxs-lookup"><span data-stu-id="340b9-607">Consider the following guidelines when accessing SQL Database using EF6:</span></span>

* <span data-ttu-id="340b9-608">Válassza a megfelelő szolgáltatástípust (megosztott vagy prémium).</span><span class="sxs-lookup"><span data-stu-id="340b9-608">Choose the appropriate service option (shared or premium).</span></span> <span data-ttu-id="340b9-609">A megosztott példány esetében a szokottnál hosszabb csatlakozási késések fordulhatnak elő, valamint a kérések számának korlátozására lehet szükség, mivel a megosztott kiszolgálót más bérlők is használják.</span><span class="sxs-lookup"><span data-stu-id="340b9-609">A shared instance may suffer longer than usual connection delays and throttling due to the usage by other tenants of the shared server.</span></span> <span data-ttu-id="340b9-610">Ha kiszámítható teljesítményre és megbízhatóan alacsony késésű műveletekre van szükség, mindenképpen a prémium szolgáltatást érdemes választani.</span><span class="sxs-lookup"><span data-stu-id="340b9-610">If predictable performance and reliable low latency operations are required, consider choosing the premium option.</span></span>
* <span data-ttu-id="340b9-611">Az Azure SQL Database esetében nem javasoljuk rögzített időközű stratégiák alkalmazását.</span><span class="sxs-lookup"><span data-stu-id="340b9-611">A fixed interval strategy is not recommended for use with Azure SQL Database.</span></span> <span data-ttu-id="340b9-612">Használjon inkább egy exponenciális visszatartási stratégiát, mivel a szolgáltatás túlterhelődhet, és a hosszabb késleltetés több időt ad a helyreállításra.</span><span class="sxs-lookup"><span data-stu-id="340b9-612">Instead, use an exponential back-off strategy because the service may be overloaded, and longer delays allow more time for it to recover.</span></span>
* <span data-ttu-id="340b9-613">A kapcsolatok definiálásakor válasszon megfelelő értéket a kapcsolatok és a parancsok időtúllépéséhez.</span><span class="sxs-lookup"><span data-stu-id="340b9-613">Choose a suitable value for the connection and command timeouts when defining connections.</span></span> <span data-ttu-id="340b9-614">Az időtúllépés ideális értékét saját üzleti logikájának felépítése és tesztek alapján állapíthatja meg.</span><span class="sxs-lookup"><span data-stu-id="340b9-614">Base the timeout on both your business logic design and through testing.</span></span> <span data-ttu-id="340b9-615">A későbbiekben szükség lehet az érték módosítására, ahogy változik az adatmennyiség, vagy változnak az üzleti folyamatok.</span><span class="sxs-lookup"><span data-stu-id="340b9-615">You may need to modify this value over time as the volumes of data or the business processes change.</span></span> <span data-ttu-id="340b9-616">A túl alacsony időtúllépési érték miatt a kapcsolatok idő előtt szakadhatnak meg, ha az adatbázis leterhelt.</span><span class="sxs-lookup"><span data-stu-id="340b9-616">Too short a timeout may result in premature failures of connections when the database is busy.</span></span> <span data-ttu-id="340b9-617">A túl magas időtúllépési érték akadályozhatja az újrapróbálkozási logika megfelelő működését, mivel túl sokáig fog várni, mielőtt észlelné a sikertelen csatlakozást.</span><span class="sxs-lookup"><span data-stu-id="340b9-617">Too long a timeout may prevent the retry logic working correctly by waiting too long before detecting a failed connection.</span></span> <span data-ttu-id="340b9-618">Az időtúllépés értéke a végpontok közötti késés egyik összetevője. Bár nehéz előre megállapítani, hogy hány parancsot kell végrehajtani a környezet mentésekor.</span><span class="sxs-lookup"><span data-stu-id="340b9-618">The value of the timeout is a component of the end-to-end latency, although you cannot easily determine how many commands will execute when saving the context.</span></span> <span data-ttu-id="340b9-619">Az alapértelmezett időtúllépést a **DbContext** példány **CommandTimeout** tulajdonságában adhatja meg.</span><span class="sxs-lookup"><span data-stu-id="340b9-619">You can change the default timeout by setting the **CommandTimeout** property of the **DbContext** instance.</span></span>
* <span data-ttu-id="340b9-620">Az Entity Framework támogatja a konfigurációs fájlokban definiált újrapróbálkozási konfigurációk használatát.</span><span class="sxs-lookup"><span data-stu-id="340b9-620">Entity Framework supports retry configurations defined in configuration files.</span></span> <span data-ttu-id="340b9-621">Ugyanakkor a minél nagyobb rugalmasság érdekében azt javasoljuk, hogy a konfigurációt szoftveresen hozza létre az alkalmazásban.</span><span class="sxs-lookup"><span data-stu-id="340b9-621">However, for maximum flexibility on Azure you should consider creating the configuration programmatically within the application.</span></span> <span data-ttu-id="340b9-622">Az újrapróbálkozási szabályzatok konkrét paraméterei – például az újrapróbálkozások száma és az újrapróbálkozási időközök – a szolgáltatás konfigurációs fájljában tárolhatók, és a futtatás során felhasználhatók a megfelelő szabályzatok létrehozására.</span><span class="sxs-lookup"><span data-stu-id="340b9-622">The specific parameters for the retry policies, such as the number of retries and the retry intervals, can be stored in the service configuration file and used at runtime to create the appropriate policies.</span></span> <span data-ttu-id="340b9-623">Így a beállítások az alkalmazás újraindítása nélkül módosíthatók.</span><span class="sxs-lookup"><span data-stu-id="340b9-623">This allows the settings to be changed without requiring the application to be restarted.</span></span>

<span data-ttu-id="340b9-624">A következő kezdőbeállításokat javasoljuk az újrapróbálkozási műveletekhez.</span><span class="sxs-lookup"><span data-stu-id="340b9-624">Consider starting with the following settings for retrying operations.</span></span> <span data-ttu-id="340b9-625">Az újrapróbálkozási kísérletek közötti késleltetés nem adható meg (rögzített, és egy exponenciális sorozat részeként jön létre).</span><span class="sxs-lookup"><span data-stu-id="340b9-625">You cannot specify the delay between retry attempts (it is fixed and generated as an exponential sequence).</span></span> <span data-ttu-id="340b9-626">Ha nem hoz létre egyéni újrapróbálkozási stratégiát, csak a maximális értékeket adhatja meg, az itt látható módon.</span><span class="sxs-lookup"><span data-stu-id="340b9-626">You can specify only the maximum values, as shown here; unless you create a custom retry strategy.</span></span> <span data-ttu-id="340b9-627">Ezek általános célú beállítások, ezért javasoljuk, hogy monitorozza műveleteit, és finomhangolja az értékeket saját igényei szerint.</span><span class="sxs-lookup"><span data-stu-id="340b9-627">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="340b9-628">**Környezet**</span><span class="sxs-lookup"><span data-stu-id="340b9-628">**Context**</span></span> | <span data-ttu-id="340b9-629">**Mintacél E2E<br />max. késleltetése**</span><span class="sxs-lookup"><span data-stu-id="340b9-629">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="340b9-630">**Újrapróbálkozási szabályzat**</span><span class="sxs-lookup"><span data-stu-id="340b9-630">**Retry policy**</span></span> | <span data-ttu-id="340b9-631">**Beállítások**</span><span class="sxs-lookup"><span data-stu-id="340b9-631">**Settings**</span></span> | <span data-ttu-id="340b9-632">**Értékek**</span><span class="sxs-lookup"><span data-stu-id="340b9-632">**Values**</span></span> | <span data-ttu-id="340b9-633">**Működési elv**</span><span class="sxs-lookup"><span data-stu-id="340b9-633">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="340b9-634">Interaktív, felhasználói felület</span><span class="sxs-lookup"><span data-stu-id="340b9-634">Interactive, UI,</span></span><br /><span data-ttu-id="340b9-635">vagy előtér</span><span class="sxs-lookup"><span data-stu-id="340b9-635">or foreground</span></span> |<span data-ttu-id="340b9-636">2 másodperc</span><span class="sxs-lookup"><span data-stu-id="340b9-636">2 seconds</span></span> |<span data-ttu-id="340b9-637">Exponenciális</span><span class="sxs-lookup"><span data-stu-id="340b9-637">Exponential</span></span> |<span data-ttu-id="340b9-638">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="340b9-638">MaxRetryCount</span></span><br /><span data-ttu-id="340b9-639">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="340b9-639">MaxDelay</span></span> |<span data-ttu-id="340b9-640">3</span><span class="sxs-lookup"><span data-stu-id="340b9-640">3</span></span><br /><span data-ttu-id="340b9-641">750 ms</span><span class="sxs-lookup"><span data-stu-id="340b9-641">750 ms</span></span> |<span data-ttu-id="340b9-642">1. kísérlet – 0 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="340b9-642">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="340b9-643">2. kísérlet – 750 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="340b9-643">Attempt 2 - delay 750 ms</span></span><br /><span data-ttu-id="340b9-644">3. kísérlet – 750 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="340b9-644">Attempt 3 – delay 750 ms</span></span> |
| <span data-ttu-id="340b9-645">Háttér</span><span class="sxs-lookup"><span data-stu-id="340b9-645">Background</span></span><br /> <span data-ttu-id="340b9-646">vagy kötegelt</span><span class="sxs-lookup"><span data-stu-id="340b9-646">or batch</span></span> |<span data-ttu-id="340b9-647">30 másodperc</span><span class="sxs-lookup"><span data-stu-id="340b9-647">30 seconds</span></span> |<span data-ttu-id="340b9-648">Exponenciális</span><span class="sxs-lookup"><span data-stu-id="340b9-648">Exponential</span></span> |<span data-ttu-id="340b9-649">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="340b9-649">MaxRetryCount</span></span><br /><span data-ttu-id="340b9-650">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="340b9-650">MaxDelay</span></span> |<span data-ttu-id="340b9-651">5</span><span class="sxs-lookup"><span data-stu-id="340b9-651">5</span></span><br /><span data-ttu-id="340b9-652">12 másodperc</span><span class="sxs-lookup"><span data-stu-id="340b9-652">12 seconds</span></span> |<span data-ttu-id="340b9-653">1. kísérlet – 0 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="340b9-653">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="340b9-654">2. kísérlet – kb. 1 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="340b9-654">Attempt 2 - delay ~1 sec</span></span><br /><span data-ttu-id="340b9-655">3. kísérlet – kb. 3 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="340b9-655">Attempt 3 - delay ~3 sec</span></span><br /><span data-ttu-id="340b9-656">4. kísérlet – kb. 7 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="340b9-656">Attempt 4 - delay ~7 sec</span></span><br /><span data-ttu-id="340b9-657">5. kísérlet – 12 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="340b9-657">Attempt 5 - delay 12 sec</span></span> |

> [!NOTE]
> <span data-ttu-id="340b9-658">A végpontok közötti késés célértéke az alapértelmezett időtúllépési érték használatát feltételezi a szolgáltatáskapcsolatokhoz.</span><span class="sxs-lookup"><span data-stu-id="340b9-658">The end-to-end latency targets assume the default timeout for connections to the service.</span></span> <span data-ttu-id="340b9-659">Ha magasabb értéket ad meg a kapcsolatok időtúllépésére, a végpontok közötti késés minden újrapróbálkozási kísérlet esetében ennyivel lesz meghosszabbítva.</span><span class="sxs-lookup"><span data-stu-id="340b9-659">If you specify longer connection timeouts, the end-to-end latency will be extended by this additional time for every retry attempt.</span></span>
>
>

### <a name="examples"></a><span data-ttu-id="340b9-660">Példák</span><span class="sxs-lookup"><span data-stu-id="340b9-660">Examples</span></span>
<span data-ttu-id="340b9-661">A következő mintakód egy egyszerű adatelérési megoldást definiál, amely az Entity Frameworköt használja.</span><span class="sxs-lookup"><span data-stu-id="340b9-661">The following code example defines a simple data access solution that uses Entity Framework.</span></span> <span data-ttu-id="340b9-662">Egy adott újrapróbálkozási stratégiát állít be a **BlogConfiguration** osztály egy példányának definiálásával, amely a **DbConfiguration** osztályt bővíti ki.</span><span class="sxs-lookup"><span data-stu-id="340b9-662">It sets a specific retry strategy by defining an instance of a class named **BlogConfiguration** that extends **DbConfiguration**.</span></span>

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

<span data-ttu-id="340b9-663">A [kapcsolat rugalmassága/újrapróbálkozási logika](http://msdn.microsoft.com/data/dn456835.aspx) mechanizmussal foglalkozó témakörben további példákat talál az Entity Framework újrapróbálkozási mechanizmusának használatára.</span><span class="sxs-lookup"><span data-stu-id="340b9-663">More examples of using the Entity Framework retry mechanism can be found in [Connection Resiliency / Retry Logic](http://msdn.microsoft.com/data/dn456835.aspx).</span></span>

### <a name="more-information"></a><span data-ttu-id="340b9-664">További információ</span><span class="sxs-lookup"><span data-stu-id="340b9-664">More information</span></span>
* [<span data-ttu-id="340b9-665">Az Azure SQL Database szolgáltatás teljesítményének és rugalmasságának útmutatója</span><span class="sxs-lookup"><span data-stu-id="340b9-665">Azure SQL Database Performance and Elasticity Guide</span></span>](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx)

## <a name="sql-database-using-entity-framework-core"></a><span data-ttu-id="340b9-666">SQL-adatbázis használata az Entity Framework Core</span><span class="sxs-lookup"><span data-stu-id="340b9-666">SQL Database using Entity Framework Core</span></span>
<span data-ttu-id="340b9-667">Az [Entity Framework Core](/ef/core/) egy objektumrelációs leképező .NET Core-fejlesztők számára, amellyel tartományspecifikus objektumokat használva lehet adatokkal dolgozni.</span><span class="sxs-lookup"><span data-stu-id="340b9-667">[Entity Framework Core](/ef/core/) is an object-relational mapper that enables .NET Core developers to work with data using domain-specific objects.</span></span> <span data-ttu-id="340b9-668">Szükségtelenné teszi az adatelérési kód használatát, amelyet egyébként a fejlesztőknek kell megírniuk.</span><span class="sxs-lookup"><span data-stu-id="340b9-668">It eliminates the need for most of the data-access code that developers usually need to write.</span></span> <span data-ttu-id="340b9-669">Az Entity Framework e verzióját az alapoktól építettük újra, ezért nem örökli meg automatikusan az EF6.x összes funkcióját.</span><span class="sxs-lookup"><span data-stu-id="340b9-669">This version of Entity Framework was written from the ground up, and doesn't automatically inherit all the features from EF6.x.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="340b9-670">Újrapróbálkozási mechanizmus</span><span class="sxs-lookup"><span data-stu-id="340b9-670">Retry mechanism</span></span>
<span data-ttu-id="340b9-671">Az újrapróbálkozás akkor támogatott, ha az SQL Database-t az Entity Framework Core-ral érik el, a [kapcsolat rugalmassága](/ef/core/miscellaneous/connection-resiliency) mechanizmus segítségével.</span><span class="sxs-lookup"><span data-stu-id="340b9-671">Retry support is provided when accessing SQL Database using Entity Framework Core through a mechanism called [Connection Resiliency](/ef/core/miscellaneous/connection-resiliency).</span></span> <span data-ttu-id="340b9-672">A kapcsolat rugalmassága először az EF Core 1.1.0-ban vált elérhetővé.</span><span class="sxs-lookup"><span data-stu-id="340b9-672">Connection resiliency was introduced in EF Core 1.1.0.</span></span>

<span data-ttu-id="340b9-673">Az elsődleges absztrakció az `IExecutionStrategy` felület.</span><span class="sxs-lookup"><span data-stu-id="340b9-673">The primary abstraction is the `IExecutionStrategy` interface.</span></span> <span data-ttu-id="340b9-674">Az SQL Server, és így az SQL Azure végrehajtási stratégiája felismeri a kivételtípusokat, amelyeknél elérhető az újrapróbálkozás, és észszerű alapértelmezett értékek vannak beállítva többek között az újrapróbálkozások maximális számánál és az újrapróbálkozások közötti késleltetésnél.</span><span class="sxs-lookup"><span data-stu-id="340b9-674">The execution strategy for SQL Server, including SQL Azure, is aware of the exception types that can be retried and has sensible defaults for maximum retries, delay between retries, and so on.</span></span>

### <a name="examples"></a><span data-ttu-id="340b9-675">Példák</span><span class="sxs-lookup"><span data-stu-id="340b9-675">Examples</span></span>

<span data-ttu-id="340b9-676">A következő kód lehetővé teszi az automatikus újrapróbálkozást a DbContext objektum konfigurálásakor, ami egy munkamenetet jelöl az adatbázisban.</span><span class="sxs-lookup"><span data-stu-id="340b9-676">The following code enables automatic retries when configuring the DbContext object, which represents a session with the database.</span></span> 

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .UseSqlServer(
            @"Server=(localdb)\mssqllocaldb;Database=EFMiscellanous.ConnectionResiliency;Trusted_Connection=True;",
            options => options.EnableRetryOnFailure());
}
```

<span data-ttu-id="340b9-677">A következő kód bemutatja, egy végrehajtási stratégia alkalmazásával hogyan hajtható végre egy tranzakció automatikus újrapróbálkozással.</span><span class="sxs-lookup"><span data-stu-id="340b9-677">The following code shows how to execute a transaction with automatic retries, by using an execution strategy.</span></span> <span data-ttu-id="340b9-678">A tranzakciót egy delegált definiálja.</span><span class="sxs-lookup"><span data-stu-id="340b9-678">The transaction is defined in a delegate.</span></span> <span data-ttu-id="340b9-679">Átmeneti hiba előfordulásakor a végrehajtási stratégia ismét meghívja a delegáltat.</span><span class="sxs-lookup"><span data-stu-id="340b9-679">If a transient failure occurs, the execution strategy will invoke the delegate again.</span></span>

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

## <a name="azure-storage"></a><span data-ttu-id="340b9-680">Azure Storage</span><span class="sxs-lookup"><span data-stu-id="340b9-680">Azure Storage</span></span>
<span data-ttu-id="340b9-681">Az Azure Storage szolgáltatásai közé tartoznak a tábla- és blobtárolók, a fájl- és tárolási üzenetsorok.</span><span class="sxs-lookup"><span data-stu-id="340b9-681">Azure storage services include table and blob storage, files, and storage queues.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="340b9-682">Újrapróbálkozási mechanizmus</span><span class="sxs-lookup"><span data-stu-id="340b9-682">Retry mechanism</span></span>
<span data-ttu-id="340b9-683">Újrapróbálkozásokra az egyes REST-műveletek szintjén kerül sor, és az ügyfél-API implementálásának szerves részeit képezik.</span><span class="sxs-lookup"><span data-stu-id="340b9-683">Retries occur at the individual REST operation level and are an integral part of the client API implementation.</span></span> <span data-ttu-id="340b9-684">Az ügyfél Storage SDK-ja az [IExtendedRetryPolicy felületet](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.aspx) implementáló osztályokat alkalmaz.</span><span class="sxs-lookup"><span data-stu-id="340b9-684">The client storage SDK uses classes that implement the [IExtendedRetryPolicy Interface](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.aspx).</span></span>

<span data-ttu-id="340b9-685">A felület több módon implementálható.</span><span class="sxs-lookup"><span data-stu-id="340b9-685">There are different implementations of the interface.</span></span> <span data-ttu-id="340b9-686">A Storage-ügyfelek kifejezetten táblákra, blobokra és üzenetsorokra szabott szabályzatok közül választhatnak.</span><span class="sxs-lookup"><span data-stu-id="340b9-686">Storage clients can choose from policies specifically designed for accessing tables, blobs, and queues.</span></span> <span data-ttu-id="340b9-687">Minden implementáció különböző újrapróbálkozási stratégiát használ, amely alapvetően az újrapróbálkozási időközöket és egyéb részleteket határoz meg.</span><span class="sxs-lookup"><span data-stu-id="340b9-687">Each implementation uses a different retry strategy that essentially defines the retry interval and other details.</span></span>

<span data-ttu-id="340b9-688">A beépített osztályok támogatják a lineáris (állandó késleltetés) és az exponenciális (véletlenszerű újrapróbálkozási időközök) konfigurációkat.</span><span class="sxs-lookup"><span data-stu-id="340b9-688">The built-in classes provide support for linear (constant delay) and exponential with randomization retry intervals.</span></span> <span data-ttu-id="340b9-689">Nincs szükség újrapróbálkozási szabályzat beállítására, ha egy másik folyamat magasabb szinten kezeli az újrapróbálkozásokat.</span><span class="sxs-lookup"><span data-stu-id="340b9-689">There is also a no retry policy for use when another process is handling retries at a higher level.</span></span> <span data-ttu-id="340b9-690">Ugyanakkor implementálhat saját újrapróbálkozási osztályokat, ha a beépített osztályok nem felelnek meg valamilyen konkrét követelménynek.</span><span class="sxs-lookup"><span data-stu-id="340b9-690">However, you can implement your own retry classes if you have specific requirements not provided by the built-in classes.</span></span>

<span data-ttu-id="340b9-691">A váltakozó újrapróbálkozás a tárolási szolgáltatás elsődleges és másodlagos helye között vált, ha írásvédett georedundáns tárolót (RA-GRS) használ, és a kérés eredménye egy újrapróbálható hiba.</span><span class="sxs-lookup"><span data-stu-id="340b9-691">Alternate retries switch between primary and secondary storage service location if you are using read access geo-redundant storage (RA-GRS) and the result of the request is a retryable error.</span></span> <span data-ttu-id="340b9-692">További információt [az Azure Storage redundanciabeállításainál](http://msdn.microsoft.com/library/azure/dn727290.aspx) találhat.</span><span class="sxs-lookup"><span data-stu-id="340b9-692">See [Azure Storage Redundancy Options](http://msdn.microsoft.com/library/azure/dn727290.aspx) for more information.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="340b9-693">Szabályzatkonfiguráció</span><span class="sxs-lookup"><span data-stu-id="340b9-693">Policy configuration</span></span>
<span data-ttu-id="340b9-694">Az újrapróbálkozási szabályzatok konfigurálása szoftveresen történik.</span><span class="sxs-lookup"><span data-stu-id="340b9-694">Retry policies are configured programmatically.</span></span> <span data-ttu-id="340b9-695">Megszokott eljárás egy **TableRequestOptions**, **BlobRequestOptions**, **FileRequestOptions** vagy **QueueRequestOptions** példány létrehozása és adatokkal történő feltöltése.</span><span class="sxs-lookup"><span data-stu-id="340b9-695">A typical procedure is to create and populate a **TableRequestOptions**, **BlobRequestOptions**, **FileRequestOptions**, or **QueueRequestOptions** instance.</span></span>

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

<span data-ttu-id="340b9-696">Beállíthat egy kérésbeállítási példányt az ügyfélen, és ezt követően az ügyfelet érintő műveletek az így megadott kérésbeállításokat fogják használni.</span><span class="sxs-lookup"><span data-stu-id="340b9-696">The request options instance can then be set on the client, and all operations with the client will use the specified request options.</span></span>

```csharp
client.DefaultRequestOptions = interactiveRequestOption;
var stats = await client.GetServiceStatsAsync();
```

<span data-ttu-id="340b9-697">Az ügyfél kérésbeállításainak felülbírálásához a kérésbeállítás-osztály egy adatokkal feltöltött példányát kell megadni paraméterként a műveleti metódusoknál.</span><span class="sxs-lookup"><span data-stu-id="340b9-697">You can override the client request options by passing a populated instance of the request options class as a parameter to operation methods.</span></span>

```csharp
var stats = await client.GetServiceStatsAsync(interactiveRequestOption, operationContext: null);
```

<span data-ttu-id="340b9-698">Az **OperationContext** példányban határozhatja meg, hogy milyen kódot kell végrehajtani újrapróbálkozás esetén, illetve ha befejeződik a művelet.</span><span class="sxs-lookup"><span data-stu-id="340b9-698">You use an **OperationContext** instance to specify the code to execute when a retry occurs and when an operation has completed.</span></span> <span data-ttu-id="340b9-699">Ez a kód gyűjti össze a művelettel kapcsolatos adatokat, amelyek bekerülnek a naplókba és a telemetriába.</span><span class="sxs-lookup"><span data-stu-id="340b9-699">This code can collect information about the operation for use in logs and telemetry.</span></span>

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

<span data-ttu-id="340b9-700">A kiterjesztett újrapróbálkozási szabályzatok jelzik, hogy egy adott hiba után szükség van-e újrapróbálkozásra, ezenkívül egy **RetryContext** objektumot adnak vissza, amely megmutatja az újrapróbálkozások számát, a legutóbbi kérés eredményét, illetve hogy a következő újrapróbálkozás az elsődleges vagy a másodlagos helyen fog történni (a részleteket lásd az alábbi táblázatban).</span><span class="sxs-lookup"><span data-stu-id="340b9-700">In addition to indicating whether a failure is suitable for retry, the extended retry policies return a **RetryContext** object that indicates the number of retries, the results of the last request, whether the next retry will happen in the primary or secondary location (see table below for details).</span></span> <span data-ttu-id="340b9-701">A **RetryContext** objektummal lehet meghatározni, hogy történjen-e újrapróbálkozás, és ha igen, mikor.</span><span class="sxs-lookup"><span data-stu-id="340b9-701">The properties of the **RetryContext** object can be used to decide if and when to attempt a retry.</span></span> <span data-ttu-id="340b9-702">További információért lásd az [IExtendedRetryPolicy.Evaluate metódus](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.evaluate.aspx) ismertetését.</span><span class="sxs-lookup"><span data-stu-id="340b9-702">For more details, see [IExtendedRetryPolicy.Evaluate Method](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.evaluate.aspx).</span></span>

<span data-ttu-id="340b9-703">A következő táblázatban a beépített újrapróbálkozási szabályzatok alapértelmezett beállításai tekinthetők meg.</span><span class="sxs-lookup"><span data-stu-id="340b9-703">The following tables show the default settings for the built-in retry policies.</span></span>

<span data-ttu-id="340b9-704">**Kérésbeállítások**</span><span class="sxs-lookup"><span data-stu-id="340b9-704">**Request options**</span></span>

| <span data-ttu-id="340b9-705">**Beállítás**</span><span class="sxs-lookup"><span data-stu-id="340b9-705">**Setting**</span></span> | <span data-ttu-id="340b9-706">**Alapértelmezett érték**</span><span class="sxs-lookup"><span data-stu-id="340b9-706">**Default value**</span></span> | <span data-ttu-id="340b9-707">**Jelentés**</span><span class="sxs-lookup"><span data-stu-id="340b9-707">**Meaning**</span></span> |
| --- | --- | --- |
| <span data-ttu-id="340b9-708">MaximumExecutionTime</span><span class="sxs-lookup"><span data-stu-id="340b9-708">MaximumExecutionTime</span></span> | <span data-ttu-id="340b9-709">None</span><span class="sxs-lookup"><span data-stu-id="340b9-709">None</span></span> | <span data-ttu-id="340b9-710">A kérés maximális végrehajtási ideje az esetleges újrapróbálkozási kísérletekkel együtt.</span><span class="sxs-lookup"><span data-stu-id="340b9-710">Maximum execution time for the request, including all potential retry attempts.</span></span> <span data-ttu-id="340b9-711">Ha nincs megadva, az, hogy mennyi ideig a kérelem megengedett nincs korlátozva.</span><span class="sxs-lookup"><span data-stu-id="340b9-711">If it is not specified, then the amount of time that a request is permitted to take is unlimited.</span></span> <span data-ttu-id="340b9-712">Más szóval a kérelem előfordulhat, hogy lefagy.</span><span class="sxs-lookup"><span data-stu-id="340b9-712">In other words, the request might hang.</span></span> |
| <span data-ttu-id="340b9-713">ServerTimeout</span><span class="sxs-lookup"><span data-stu-id="340b9-713">ServerTimeout</span></span> | <span data-ttu-id="340b9-714">None</span><span class="sxs-lookup"><span data-stu-id="340b9-714">None</span></span> | <span data-ttu-id="340b9-715">Kérés időtúllépési kerete a kiszolgálón (egész másodpercre kerekített érték).</span><span class="sxs-lookup"><span data-stu-id="340b9-715">Server timeout interval for the request (value is rounded to seconds).</span></span> <span data-ttu-id="340b9-716">Ha nincs megadva, a kiszolgálóhoz érkező összes kérés esetében az alapértelmezett értéket fogja használni.</span><span class="sxs-lookup"><span data-stu-id="340b9-716">If not specified, it will use the default value for all requests to the server.</span></span> <span data-ttu-id="340b9-717">Általában érdemes üresen hagyni ezt az értéket, hogy a kiszolgáló az alapértelmezett beállítást használja.</span><span class="sxs-lookup"><span data-stu-id="340b9-717">Usually, the best option is to omit this setting so that the server default is used.</span></span> | 
| <span data-ttu-id="340b9-718">LocationMode</span><span class="sxs-lookup"><span data-stu-id="340b9-718">LocationMode</span></span> | <span data-ttu-id="340b9-719">None</span><span class="sxs-lookup"><span data-stu-id="340b9-719">None</span></span> | <span data-ttu-id="340b9-720">Ha a tárfiókot az írásvédett georedundáns tároló (RA-GRS) replikációs beállítással hozza létre, a hely módja határozza meg, hogy melyik hely kapja meg a kérést.</span><span class="sxs-lookup"><span data-stu-id="340b9-720">If the storage account is created with the Read access geo-redundant storage (RA-GRS) replication option, you can use the location mode to indicate which location should receive the request.</span></span> <span data-ttu-id="340b9-721">A **PrimaryThenSecondary** érték esetében például a kérések először mindig az elsődleges helyre érkeznek.</span><span class="sxs-lookup"><span data-stu-id="340b9-721">For example, if **PrimaryThenSecondary** is specified, requests are always sent to the primary location first.</span></span> <span data-ttu-id="340b9-722">Ha a kérés sikertelen, ezután a másodlagos hely kapja meg azt.</span><span class="sxs-lookup"><span data-stu-id="340b9-722">If a request fails, it is sent to the secondary location.</span></span> |
| <span data-ttu-id="340b9-723">RetryPolicy</span><span class="sxs-lookup"><span data-stu-id="340b9-723">RetryPolicy</span></span> | <span data-ttu-id="340b9-724">ExponentialPolicy</span><span class="sxs-lookup"><span data-stu-id="340b9-724">ExponentialPolicy</span></span> | <span data-ttu-id="340b9-725">Az egyes beállítások részleteit az alábbiakban tekintheti meg.</span><span class="sxs-lookup"><span data-stu-id="340b9-725">See below for details of each option.</span></span> |

<span data-ttu-id="340b9-726">**Exponenciális szabályzat**</span><span class="sxs-lookup"><span data-stu-id="340b9-726">**Exponential policy**</span></span> 

| <span data-ttu-id="340b9-727">**Beállítás**</span><span class="sxs-lookup"><span data-stu-id="340b9-727">**Setting**</span></span> | <span data-ttu-id="340b9-728">**Alapértelmezett érték**</span><span class="sxs-lookup"><span data-stu-id="340b9-728">**Default value**</span></span> | <span data-ttu-id="340b9-729">**Jelentés**</span><span class="sxs-lookup"><span data-stu-id="340b9-729">**Meaning**</span></span> |
| --- | --- | --- |
| <span data-ttu-id="340b9-730">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="340b9-730">maxAttempt</span></span> | <span data-ttu-id="340b9-731">3</span><span class="sxs-lookup"><span data-stu-id="340b9-731">3</span></span> | <span data-ttu-id="340b9-732">Az újrapróbálkozási kísérletek száma.</span><span class="sxs-lookup"><span data-stu-id="340b9-732">Number of retry attempts.</span></span> |
| <span data-ttu-id="340b9-733">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="340b9-733">deltaBackoff</span></span> | <span data-ttu-id="340b9-734">4 másodperc</span><span class="sxs-lookup"><span data-stu-id="340b9-734">4 seconds</span></span> | <span data-ttu-id="340b9-735">Az újrapróbálkozások közötti visszatartási időköz.</span><span class="sxs-lookup"><span data-stu-id="340b9-735">Back-off interval between retries.</span></span> <span data-ttu-id="340b9-736">Az ezt követő újrapróbálkozási kísérletek esetében az időtartomány többszörösét (egy véletlenszerű elemet belevéve) fogja használni a rendszer.</span><span class="sxs-lookup"><span data-stu-id="340b9-736">Multiples of this timespan, including a random element, will be used for subsequent retry attempts.</span></span> |
| <span data-ttu-id="340b9-737">MinBackoff</span><span class="sxs-lookup"><span data-stu-id="340b9-737">MinBackoff</span></span> | <span data-ttu-id="340b9-738">3 másodperc</span><span class="sxs-lookup"><span data-stu-id="340b9-738">3 seconds</span></span> | <span data-ttu-id="340b9-739">Ez az érték hozzáadódik a deltaBackoff alapján kiszámított minden újrapróbálkozási időközhöz.</span><span class="sxs-lookup"><span data-stu-id="340b9-739">Added to all retry intervals computed from deltaBackoff.</span></span> <span data-ttu-id="340b9-740">Ez az érték nem módosítható.</span><span class="sxs-lookup"><span data-stu-id="340b9-740">This value cannot be changed.</span></span>
| <span data-ttu-id="340b9-741">MaxBackoff</span><span class="sxs-lookup"><span data-stu-id="340b9-741">MaxBackoff</span></span> | <span data-ttu-id="340b9-742">120 másodperc</span><span class="sxs-lookup"><span data-stu-id="340b9-742">120 seconds</span></span> | <span data-ttu-id="340b9-743">A MaxBackoff akkor lép működésbe, ha a kiszámított újrapróbálkozási időköz nagyobb, mint a MaxBackoff értéke.</span><span class="sxs-lookup"><span data-stu-id="340b9-743">MaxBackoff is used if the computed retry interval is greater than MaxBackoff.</span></span> <span data-ttu-id="340b9-744">Ez az érték nem módosítható.</span><span class="sxs-lookup"><span data-stu-id="340b9-744">This value cannot be changed.</span></span> |

<span data-ttu-id="340b9-745">**Lineáris szabályzat**</span><span class="sxs-lookup"><span data-stu-id="340b9-745">**Linear policy**</span></span>

| <span data-ttu-id="340b9-746">**Beállítás**</span><span class="sxs-lookup"><span data-stu-id="340b9-746">**Setting**</span></span> | <span data-ttu-id="340b9-747">**Alapértelmezett érték**</span><span class="sxs-lookup"><span data-stu-id="340b9-747">**Default value**</span></span> | <span data-ttu-id="340b9-748">**Jelentés**</span><span class="sxs-lookup"><span data-stu-id="340b9-748">**Meaning**</span></span> |
| --- | --- | --- |
| <span data-ttu-id="340b9-749">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="340b9-749">maxAttempt</span></span> | <span data-ttu-id="340b9-750">3</span><span class="sxs-lookup"><span data-stu-id="340b9-750">3</span></span> | <span data-ttu-id="340b9-751">Az újrapróbálkozási kísérletek száma.</span><span class="sxs-lookup"><span data-stu-id="340b9-751">Number of retry attempts.</span></span> |
| <span data-ttu-id="340b9-752">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="340b9-752">deltaBackoff</span></span> | <span data-ttu-id="340b9-753">30 másodperc</span><span class="sxs-lookup"><span data-stu-id="340b9-753">30 seconds</span></span> | <span data-ttu-id="340b9-754">Az újrapróbálkozások közötti visszatartási időköz.</span><span class="sxs-lookup"><span data-stu-id="340b9-754">Back-off interval between retries.</span></span> |

### <a name="retry-usage-guidance"></a><span data-ttu-id="340b9-755">Újrapróbálkozásokra vonatkozó útmutató</span><span class="sxs-lookup"><span data-stu-id="340b9-755">Retry usage guidance</span></span>
<span data-ttu-id="340b9-756">Ügyeljen a következőkre, amikor a Storage-ügyfél API-ján keresztül próbálja elérni az Azure Storage-szolgáltatásokat:</span><span class="sxs-lookup"><span data-stu-id="340b9-756">Consider the following guidelines when accessing Azure storage services using the storage client API:</span></span>

* <span data-ttu-id="340b9-757">Használja a Microsoft.WindowsAzure.Storage.RetryPolicies névtérben található beépített újrapróbálkozási szabályzatokat, amennyiben azok megfelelnek az elvárásainak.</span><span class="sxs-lookup"><span data-stu-id="340b9-757">Use the built-in retry policies from the Microsoft.WindowsAzure.Storage.RetryPolicies namespace where they are appropriate for your requirements.</span></span> <span data-ttu-id="340b9-758">A legtöbb esetben ezek a szabályzatok is elégségesek.</span><span class="sxs-lookup"><span data-stu-id="340b9-758">In most cases, these policies will be sufficient.</span></span>
* <span data-ttu-id="340b9-759">Az **ExponentialRetry** szabályzatot kötegelt műveletek, háttérfeladatok vagy nem interaktív forgatókönyvek esetében használja.</span><span class="sxs-lookup"><span data-stu-id="340b9-759">Use the **ExponentialRetry** policy in batch operations, background tasks, or non-interactive scenarios.</span></span> <span data-ttu-id="340b9-760">Ezekben az esetekben általában több idő áll rendelkezésre megvárni, amíg a szolgáltatás helyreáll, ami növeli a művelet sikeres teljesítésének esélyét.</span><span class="sxs-lookup"><span data-stu-id="340b9-760">In these scenarios, you can typically allow more time for the service to recover—with a consequently increased chance of the operation eventually succeeding.</span></span>
* <span data-ttu-id="340b9-761">Érdemes megadni a **RequestOptions** paraméter **MaximumExecutionTime** tulajdonságát a teljes végrehajtási idő korlátozásához, de az időtúllépési érték megadásakor vegye figyelembe a művelet típusát és méretét.</span><span class="sxs-lookup"><span data-stu-id="340b9-761">Consider specifying the **MaximumExecutionTime** property of the **RequestOptions** parameter to limit the total execution time, but take into account the type and size of the operation when choosing a timeout value.</span></span>
* <span data-ttu-id="340b9-762">Ha egyéni újrapróbálkozást implementál, ne hozzon létre burkolókat a Storage-ügyfél osztályai körül.</span><span class="sxs-lookup"><span data-stu-id="340b9-762">If you need to implement a custom retry, avoid creating wrappers around the storage client classes.</span></span> <span data-ttu-id="340b9-763">Inkább meglévő szabályzatokat bővítsen ki az **IExtendedRetryPolicy** felületen keresztül.</span><span class="sxs-lookup"><span data-stu-id="340b9-763">Instead, use the capabilities to extend the existing policies through the **IExtendedRetryPolicy** interface.</span></span>
* <span data-ttu-id="340b9-764">Amennyiben írásvédett georedundáns tárolót (RA-GRS) használ, a **LocationMode** segítségével beállíthatja, hogy az elsődleges hozzáférés sikertelensége esetén az újrapróbálkozási kísérletek a tároló másodlagos írásvédett példányához férjenek hozzá.</span><span class="sxs-lookup"><span data-stu-id="340b9-764">If you are using read access geo-redundant storage (RA-GRS) you can use the **LocationMode** to specify that retry attempts will access the secondary read-only copy of the store should the primary access fail.</span></span> <span data-ttu-id="340b9-765">Ebben az esetben azonban meg kell győződnie arról, hogy az alkalmazás esetlegesen elavult adatokkal is képes sikeresen működni (ha az elsődleges tároló replikálása még nem fejeződött be).</span><span class="sxs-lookup"><span data-stu-id="340b9-765">However, when using this option you must ensure that your application can work successfully with data that may be stale if the replication from the primary store has not yet completed.</span></span>

<span data-ttu-id="340b9-766">A következő beállításokat javasoljuk az újrapróbálkozási műveletekhez.</span><span class="sxs-lookup"><span data-stu-id="340b9-766">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="340b9-767">Ezek általános célú beállítások, ezért javasoljuk, hogy monitorozza műveleteit, és finomhangolja az értékeket saját igényei szerint.</span><span class="sxs-lookup"><span data-stu-id="340b9-767">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>  

| <span data-ttu-id="340b9-768">**Környezet**</span><span class="sxs-lookup"><span data-stu-id="340b9-768">**Context**</span></span> | <span data-ttu-id="340b9-769">**Mintacél E2E<br />max. késleltetése**</span><span class="sxs-lookup"><span data-stu-id="340b9-769">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="340b9-770">**Újrapróbálkozási szabályzat**</span><span class="sxs-lookup"><span data-stu-id="340b9-770">**Retry policy**</span></span> | <span data-ttu-id="340b9-771">**Beállítások**</span><span class="sxs-lookup"><span data-stu-id="340b9-771">**Settings**</span></span> | <span data-ttu-id="340b9-772">**Értékek**</span><span class="sxs-lookup"><span data-stu-id="340b9-772">**Values**</span></span> | <span data-ttu-id="340b9-773">**Működési elv**</span><span class="sxs-lookup"><span data-stu-id="340b9-773">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="340b9-774">Interaktív, felhasználói felület</span><span class="sxs-lookup"><span data-stu-id="340b9-774">Interactive, UI,</span></span><br /><span data-ttu-id="340b9-775">vagy előtér</span><span class="sxs-lookup"><span data-stu-id="340b9-775">or foreground</span></span> |<span data-ttu-id="340b9-776">2 másodperc</span><span class="sxs-lookup"><span data-stu-id="340b9-776">2 seconds</span></span> |<span data-ttu-id="340b9-777">Lineáris</span><span class="sxs-lookup"><span data-stu-id="340b9-777">Linear</span></span> |<span data-ttu-id="340b9-778">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="340b9-778">maxAttempt</span></span><br /><span data-ttu-id="340b9-779">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="340b9-779">deltaBackoff</span></span> |<span data-ttu-id="340b9-780">3</span><span class="sxs-lookup"><span data-stu-id="340b9-780">3</span></span><br /><span data-ttu-id="340b9-781">500 ms</span><span class="sxs-lookup"><span data-stu-id="340b9-781">500 ms</span></span> |<span data-ttu-id="340b9-782">1. kísérlet – 500 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="340b9-782">Attempt 1 - delay 500 ms</span></span><br /><span data-ttu-id="340b9-783">2. kísérlet – 500 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="340b9-783">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="340b9-784">3. kísérlet – 500 ms késleltetés</span><span class="sxs-lookup"><span data-stu-id="340b9-784">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="340b9-785">Háttér</span><span class="sxs-lookup"><span data-stu-id="340b9-785">Background</span></span><br /><span data-ttu-id="340b9-786">vagy kötegelt</span><span class="sxs-lookup"><span data-stu-id="340b9-786">or batch</span></span> |<span data-ttu-id="340b9-787">30 másodperc</span><span class="sxs-lookup"><span data-stu-id="340b9-787">30 seconds</span></span> |<span data-ttu-id="340b9-788">Exponenciális</span><span class="sxs-lookup"><span data-stu-id="340b9-788">Exponential</span></span> |<span data-ttu-id="340b9-789">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="340b9-789">maxAttempt</span></span><br /><span data-ttu-id="340b9-790">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="340b9-790">deltaBackoff</span></span> |<span data-ttu-id="340b9-791">5</span><span class="sxs-lookup"><span data-stu-id="340b9-791">5</span></span><br /><span data-ttu-id="340b9-792">4 másodperc</span><span class="sxs-lookup"><span data-stu-id="340b9-792">4 seconds</span></span> |<span data-ttu-id="340b9-793">1. kísérlet – kb. 3 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="340b9-793">Attempt 1 - delay ~3 sec</span></span><br /><span data-ttu-id="340b9-794">2. kísérlet – kb. 7 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="340b9-794">Attempt 2 - delay ~7 sec</span></span><br /><span data-ttu-id="340b9-795">3. kísérlet – kb. 15 mp. késleltetés</span><span class="sxs-lookup"><span data-stu-id="340b9-795">Attempt 3 - delay ~15 sec</span></span> |

### <a name="telemetry"></a><span data-ttu-id="340b9-796">Telemetria</span><span class="sxs-lookup"><span data-stu-id="340b9-796">Telemetry</span></span>
<span data-ttu-id="340b9-797">Az újrapróbálkozási kísérletek a **TraceSource** osztályban lesznek naplózva.</span><span class="sxs-lookup"><span data-stu-id="340b9-797">Retry attempts are logged to a **TraceSource**.</span></span> <span data-ttu-id="340b9-798">Az események rögzítéséhez és megfelelő célnaplóba való írásához egy **TraceListener** osztályt kell konfigurálnia.</span><span class="sxs-lookup"><span data-stu-id="340b9-798">You must configure a **TraceListener** to capture the events and write them to a suitable destination log.</span></span> <span data-ttu-id="340b9-799">Használja a **TextWriterTraceListener** vagy az **XmlWriterTraceListener** osztályt az adatok a naplófájlba írásához. Az **EventLogTraceListener** osztállyal a Windows eseménynaplóba írhat, míg az **EventProviderTraceListener** osztállyal a nyomkövetési adatokat rögzítheti az ETW alrendszerben.</span><span class="sxs-lookup"><span data-stu-id="340b9-799">You can use the **TextWriterTraceListener** or **XmlWriterTraceListener** to write the data to a log file, the **EventLogTraceListener** to write to the Windows Event Log, or the **EventProviderTraceListener** to write trace data to the ETW subsystem.</span></span> <span data-ttu-id="340b9-800">Beállíthatja, hogy a puffer ürítése automatikusan megtörténjen, valamint a naplózott események részletességét is (például: Hiba, Figyelmeztetés, Tájékoztató és Részletes).</span><span class="sxs-lookup"><span data-stu-id="340b9-800">You can also configure auto-flushing of the buffer, and the verbosity of events that will be logged (for example, Error, Warning, Informational, and Verbose).</span></span> <span data-ttu-id="340b9-801">További információt a [.NET-es Storage-ügyfélkódtárral történő ügyféloldali naplózást](http://msdn.microsoft.com/library/azure/dn782839.aspx) ismertető szakaszban találhat.</span><span class="sxs-lookup"><span data-stu-id="340b9-801">For more information, see [Client-side Logging with the .NET Storage Client Library](http://msdn.microsoft.com/library/azure/dn782839.aspx).</span></span>

<span data-ttu-id="340b9-802">A műveletek kaphatnak egy **OperationContext**-példányt, amely közzétesz egy **Retrying** eseményt, amelyhez egyéni telemetrialogika csatolható.</span><span class="sxs-lookup"><span data-stu-id="340b9-802">Operations can receive an **OperationContext** instance, which exposes a **Retrying** event that can be used to attach custom telemetry logic.</span></span> <span data-ttu-id="340b9-803">További információ: [OperationContext.Retrying Event](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.operationcontext.retrying.aspx).</span><span class="sxs-lookup"><span data-stu-id="340b9-803">For more information, see [OperationContext.Retrying Event](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.operationcontext.retrying.aspx).</span></span>

### <a name="examples"></a><span data-ttu-id="340b9-804">Példák</span><span class="sxs-lookup"><span data-stu-id="340b9-804">Examples</span></span>
<span data-ttu-id="340b9-805">A következő mintakód azt mutatja be, hogyan lehet két **TableRequestOptions**-példányt létrehozni eltérő újrapróbálkozási beállításokkal. Az egyik az interaktív, a másik pedig a háttérkérések kezelésére szolgál.</span><span class="sxs-lookup"><span data-stu-id="340b9-805">The following code example shows how to create two **TableRequestOptions** instances with different retry settings; one for interactive requests and one for background requests.</span></span> <span data-ttu-id="340b9-806">A példa ezután az ügyfélre alkalmazza ezt a két újrapróbálkozási szabályzatot, így azok minden kérésre vonatkozni fognak, ezenkívül az interaktív stratégiát egy adott kéréshez köti, így az felülbírálja az ügyfél alapértelmezett beállításait.</span><span class="sxs-lookup"><span data-stu-id="340b9-806">The example then sets these two retry policies on the client so that they apply for all requests, and also sets the interactive strategy on a specific request so that it overrides the default settings applied to the client.</span></span>

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

### <a name="more-information"></a><span data-ttu-id="340b9-807">További információ</span><span class="sxs-lookup"><span data-stu-id="340b9-807">More information</span></span>
* [<span data-ttu-id="340b9-808">Az Azure Storage ügyféloldali kódtárhoz javasolt újrapróbálkozási szabályzatok</span><span class="sxs-lookup"><span data-stu-id="340b9-808">Azure Storage Client Library Retry Policy Recommendations</span></span>](https://azure.microsoft.com/blog/2014/05/22/azure-storage-client-library-retry-policy-recommendations/)
* [<span data-ttu-id="340b9-809">Storage ügyféloldali kódtár 2.0 – újrapróbálkozási szabályzatok implementálása</span><span class="sxs-lookup"><span data-stu-id="340b9-809">Storage Client Library 2.0 – Implementing Retry Policies</span></span>](http://gauravmantri.com/2012/12/30/storage-client-library-2-0-implementing-retry-policies/)

## <a name="general-rest-and-retry-guidelines"></a><span data-ttu-id="340b9-810">Általános REST- és újrapróbálkozási irányelvek</span><span class="sxs-lookup"><span data-stu-id="340b9-810">General REST and retry guidelines</span></span>
<span data-ttu-id="340b9-811">Vegye figyelembe a következőket, amikor az Azure-t vagy külső szolgáltatást próbál elérni:</span><span class="sxs-lookup"><span data-stu-id="340b9-811">Consider the following when accessing Azure or third party services:</span></span>

* <span data-ttu-id="340b9-812">Alkalmazzon rendszerszintű megközelítést az újrapróbálkozások kezelésére, például egy újrafelhasználható kód formájában, hogy minden ügyfél és megoldás esetében ugyanaz a módszertan érvényesüljön.</span><span class="sxs-lookup"><span data-stu-id="340b9-812">Use a systematic approach to managing retries, perhaps as reusable code, so that you can apply a consistent methodology across all clients and all solutions.</span></span>
* <span data-ttu-id="340b9-813">Érdemes lehet egy újrapróbálkozási keretrendszert használni (például az átmenetihiba-kezelési alkalmazásblokkot) az újrapróbálkozások kezelésére, ha a célszolgáltatás vagy -ügyfél nem rendelkezik beépített újrapróbálkozási mechanizmussal.</span><span class="sxs-lookup"><span data-stu-id="340b9-813">Consider using a retry framework such as the Transient Fault Handling Application Block to manage retries if the target service or client has no built-in retry mechanism.</span></span> <span data-ttu-id="340b9-814">Így konzisztens újrapróbálkozási viselkedés valósítható meg, és megfelelő alapértelmezett újrapróbálkozási stratégiát tehet elérhetővé a célszolgáltatás számára.</span><span class="sxs-lookup"><span data-stu-id="340b9-814">This will help you implement a consistent retry behavior, and it may provide a suitable default retry strategy for the target service.</span></span> <span data-ttu-id="340b9-815">Ugyanakkor szükség lehet egyéni újrapróbálkozási kód kialakítására a nem szabványos viselkedésű szolgáltatások esetében, amelyek nem kivételek alapján állapítják meg az átmeneti hibákat, illetve ha **Retry-Response** válasszal kívánja kezelni az újrapróbálkozási viselkedést.</span><span class="sxs-lookup"><span data-stu-id="340b9-815">However, you may need to create custom retry code for services that have non-standard behavior, that do not rely on exceptions to indicate transient failures, or if you want to use a **Retry-Response** reply to manage retry behavior.</span></span>
* <span data-ttu-id="340b9-816">Az átmeneti hibák észlelési logikája a REST-hívások végrehajtására ténylegesen használt ügyfél-API-tól függ.</span><span class="sxs-lookup"><span data-stu-id="340b9-816">The transient detection logic will depend on the actual client API you use to invoke the REST calls.</span></span> <span data-ttu-id="340b9-817">Egyes ügyfelek, például az újabb **HttpClient** osztály, nem váltanak ki kivételeket, ha a kérés befejeződik, de nem sikert jelző HTTP-állapotkódot ad vissza.</span><span class="sxs-lookup"><span data-stu-id="340b9-817">Some clients, such as the newer **HttpClient** class, will not throw exceptions for completed requests with a non-success HTTP status code.</span></span> <span data-ttu-id="340b9-818">Ez növeli a teljesítményt, de megakadályozza az átmenetihiba-kezelési alkalmazásblokk használatát.</span><span class="sxs-lookup"><span data-stu-id="340b9-818">This improves performance but prevents the use of the Transient Fault Handling Application Block.</span></span> <span data-ttu-id="340b9-819">Ebben az esetben a hívást olyan kóddal kell beburkolni a REST API számára, amely kivételeket vált ki a nem sikert jelző HTTP-állapotkódok esetén, amelyet aztán feldolgozhat a blokk.</span><span class="sxs-lookup"><span data-stu-id="340b9-819">In this case you could wrap the call to the REST API with code that produces exceptions for non-success HTTP status codes, which can then be processed by the block.</span></span> <span data-ttu-id="340b9-820">Ezenkívül használhat egy másik mechanizmust is az újrapróbálkozások kezelésére.</span><span class="sxs-lookup"><span data-stu-id="340b9-820">Alternatively, you can use a different mechanism to drive the retries.</span></span>
* <span data-ttu-id="340b9-821">A szolgáltatás által visszaadott HTTP-állapotkód segíthet megállapítani, hogy a hiba átmeneti jellegű-e.</span><span class="sxs-lookup"><span data-stu-id="340b9-821">The HTTP status code returned from the service can help to indicate whether the failure is transient.</span></span> <span data-ttu-id="340b9-822">Lehet, hogy az állapotkód vagy az azzal egyenértékű kivételtípus megismeréséhez meg kell vizsgálnia az ügyfél vagy az újrapróbálkozási keretrendszer által előállított kivételeket.</span><span class="sxs-lookup"><span data-stu-id="340b9-822">You may need to examine the exceptions generated by a client or the retry framework to access the status code or to determine the equivalent exception type.</span></span> <span data-ttu-id="340b9-823">A következő HTTP-kódok általában azt jelzik, hogy érdemes újrapróbálkozni:</span><span class="sxs-lookup"><span data-stu-id="340b9-823">The following HTTP codes typically indicate that a retry is appropriate:</span></span>
  * <span data-ttu-id="340b9-824">408 Kérés időtúllépése</span><span class="sxs-lookup"><span data-stu-id="340b9-824">408 Request Timeout</span></span>
  * <span data-ttu-id="340b9-825">429 túl sok kérelem</span><span class="sxs-lookup"><span data-stu-id="340b9-825">429 Too Many Requests</span></span>
  * <span data-ttu-id="340b9-826">500 Belső kiszolgálóhiba</span><span class="sxs-lookup"><span data-stu-id="340b9-826">500 Internal Server Error</span></span>
  * <span data-ttu-id="340b9-827">502 Hibás átjáró</span><span class="sxs-lookup"><span data-stu-id="340b9-827">502 Bad Gateway</span></span>
  * <span data-ttu-id="340b9-828">503 A szolgáltatás nem érhető el</span><span class="sxs-lookup"><span data-stu-id="340b9-828">503 Service Unavailable</span></span>
  * <span data-ttu-id="340b9-829">504 Időtúllépés az átjárón</span><span class="sxs-lookup"><span data-stu-id="340b9-829">504 Gateway Timeout</span></span>
* <span data-ttu-id="340b9-830">Amennyiben kivételekre épül az újrapróbálkozási logikája, a következők általában a csatlakozást meghiúsító átmeneti hibát jeleznek:</span><span class="sxs-lookup"><span data-stu-id="340b9-830">If you base your retry logic on exceptions, the following typically indicate a transient failure where no connection could be established:</span></span>
  * <span data-ttu-id="340b9-831">WebExceptionStatus.ConnectionClosed</span><span class="sxs-lookup"><span data-stu-id="340b9-831">WebExceptionStatus.ConnectionClosed</span></span>
  * <span data-ttu-id="340b9-832">WebExceptionStatus.ConnectFailure</span><span class="sxs-lookup"><span data-stu-id="340b9-832">WebExceptionStatus.ConnectFailure</span></span>
  * <span data-ttu-id="340b9-833">WebExceptionStatus.Timeout</span><span class="sxs-lookup"><span data-stu-id="340b9-833">WebExceptionStatus.Timeout</span></span>
  * <span data-ttu-id="340b9-834">WebExceptionStatus.RequestCanceled</span><span class="sxs-lookup"><span data-stu-id="340b9-834">WebExceptionStatus.RequestCanceled</span></span>
* <span data-ttu-id="340b9-835">Amennyiben egy szolgáltatás állapota „Nem érhető el”, előfordulhat, hogy a szolgáltatás a megfelelő késleltetést jelzi, mielőtt újrapróbálkozna a **Retry-After** válaszfejlécben vagy egy másik egyéni fejlécben.</span><span class="sxs-lookup"><span data-stu-id="340b9-835">In the case of a service unavailable status, the service might indicate the appropriate delay before retrying in the **Retry-After** response header or a different custom header.</span></span> <span data-ttu-id="340b9-836">A szolgáltatások további információkat küldhetnek egyéni fejlécként, vagy a válasz tartalmába beágyazva.</span><span class="sxs-lookup"><span data-stu-id="340b9-836">Services might also send additional information as custom headers, or embedded in the content of the response.</span></span> <span data-ttu-id="340b9-837">Az átmenetihiba-kezelési alkalmazásblokk nem képes használni sem a szabványos, sem pedig az egyéni „retry-after”-fejléceket.</span><span class="sxs-lookup"><span data-stu-id="340b9-837">The Transient Fault Handling Application Block cannot use the standard or any custom “retry-after” headers.</span></span>
* <span data-ttu-id="340b9-838">Ne próbálkozzon újra, ha az állapotkód ügyfélhibát jelez (4xx-es tartomány). E szabály alól csak a 408-as Kérés időtúllépése képez kivételt.</span><span class="sxs-lookup"><span data-stu-id="340b9-838">Do not retry for status codes representing client errors (errors in the 4xx range) except for a 408 Request Timeout.</span></span>
* <span data-ttu-id="340b9-839">Tesztelje le alaposan az újrapróbálkozási stratégiáit és mechanizmusait különböző feltételek mellett, például különböző hálózati állapotok és rendszerterhelések esetén.</span><span class="sxs-lookup"><span data-stu-id="340b9-839">Thoroughly test your retry strategies and mechanisms under a range of conditions, such as different network states and varying system loadings.</span></span>

### <a name="retry-strategies"></a><span data-ttu-id="340b9-840">Újrapróbálkozási stratégiák</span><span class="sxs-lookup"><span data-stu-id="340b9-840">Retry strategies</span></span>
<span data-ttu-id="340b9-841">A következők az újrapróbálkozási stratégiák időközeinek jellemző típusai:</span><span class="sxs-lookup"><span data-stu-id="340b9-841">The following are the typical types of retry strategy intervals:</span></span>

* <span data-ttu-id="340b9-842">**Az exponenciális**.</span><span class="sxs-lookup"><span data-stu-id="340b9-842">**Exponential**.</span></span> <span data-ttu-id="340b9-843">A megadott számú ki megközelítés véletlenszerű exponenciális biztonsági használatával az intervallum újrapróbálkozások között, az újrapróbálkozásokat végző újrapróbálkozási házirendje.</span><span class="sxs-lookup"><span data-stu-id="340b9-843">A retry policy that performs a specified number of retries, using a randomized exponential back off approach to determine the interval between retries.</span></span> <span data-ttu-id="340b9-844">Példa:</span><span class="sxs-lookup"><span data-stu-id="340b9-844">For example:</span></span>

    ```csharp
    var random = new Random();

    var delta = (int)((Math.Pow(2.0, currentRetryCount) - 1.0) *
                random.Next((int)(this.deltaBackoff.TotalMilliseconds * 0.8),
                (int)(this.deltaBackoff.TotalMilliseconds * 1.2)));
    var interval = (int)Math.Min(checked(this.minBackoff.TotalMilliseconds + delta),
                    this.maxBackoff.TotalMilliseconds);
    retryInterval = TimeSpan.FromMilliseconds(interval);
    ```

* <span data-ttu-id="340b9-845">**Növekményes**.</span><span class="sxs-lookup"><span data-stu-id="340b9-845">**Incremental**.</span></span> <span data-ttu-id="340b9-846">A megadott számú újrapróbálkozások és a próbálkozások növekményes időközét újrapróbálkozási stratégiát.</span><span class="sxs-lookup"><span data-stu-id="340b9-846">A retry strategy with a specified number of retry attempts and an incremental time interval between retries.</span></span> <span data-ttu-id="340b9-847">Példa:</span><span class="sxs-lookup"><span data-stu-id="340b9-847">For example:</span></span>

    ```csharp
    retryInterval = TimeSpan.FromMilliseconds(this.initialInterval.TotalMilliseconds +
                    (this.increment.TotalMilliseconds * currentRetryCount));
    ```

* <span data-ttu-id="340b9-848">**LinearRetry**.</span><span class="sxs-lookup"><span data-stu-id="340b9-848">**LinearRetry**.</span></span> <span data-ttu-id="340b9-849">Egy megadott időpontban időköze újrapróbálkozások között, az újrapróbálkozásokat megadott számú végző újrapróbálkozási házirendje.</span><span class="sxs-lookup"><span data-stu-id="340b9-849">A retry policy that performs a specified number of retries, using a specified fixed time interval between retries.</span></span> <span data-ttu-id="340b9-850">Példa:</span><span class="sxs-lookup"><span data-stu-id="340b9-850">For example:</span></span>

    ```csharp
    retryInterval = this.deltaBackoff;
    ```

### <a name="transient-fault-handling-with-polly"></a><span data-ttu-id="340b9-851">Átmeneti hibák kezelése a Polly használatával</span><span class="sxs-lookup"><span data-stu-id="340b9-851">Transient fault handling with Polly</span></span>
<span data-ttu-id="340b9-852">[Polly] [ polly] egy függvénytár programozottan kezelni az újrapróbálkozások és [áramköri megszakító] [ circuit-breaker] stratégiák.</span><span class="sxs-lookup"><span data-stu-id="340b9-852">[Polly][polly] is a library to programatically handle retries and [circuit breaker][circuit-breaker] strategies.</span></span> <span data-ttu-id="340b9-853">A Polly projekt a [.NET Foundation][dotnet-foundation] keretében érhető el.</span><span class="sxs-lookup"><span data-stu-id="340b9-853">The Polly project is a member of the [.NET Foundation][dotnet-foundation].</span></span> <span data-ttu-id="340b9-854">A Polly olyan szolgáltatások számára kínál alternatívát, amelyekben az ügyfél nem támogatja natív módon az újrapróbálkozásokat. Megszünteti az olyan egyéni újrapróbálkozási kódok írásának szükségét, amelyeket egyébként nehéz lenne megfelelően implementálni.</span><span class="sxs-lookup"><span data-stu-id="340b9-854">For services where the client does not natively support retries, Polly is a valid alternative and avoids the need to write custom retry code, which can be hard to implement correctly.</span></span> <span data-ttu-id="340b9-855">A Polly ezenkívül módot ad a hibák nyomon követésére, így naplózhatóvá teszi az újrapróbálkozásokat.</span><span class="sxs-lookup"><span data-stu-id="340b9-855">Polly also provides a way to trace errors when they occur, so that you can log retries.</span></span>


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


### <a name="more-information"></a><span data-ttu-id="340b9-859">További információ</span><span class="sxs-lookup"><span data-stu-id="340b9-859">More information</span></span>
* [<span data-ttu-id="340b9-860">Kapcsolat rugalmassága</span><span class="sxs-lookup"><span data-stu-id="340b9-860">Connection Resiliency</span></span>](/ef/core/miscellaneous/connection-resiliency)
* [<span data-ttu-id="340b9-861">Adatpontok – EF Core 1.1</span><span class="sxs-lookup"><span data-stu-id="340b9-861">Data Points - EF Core 1.1</span></span>](https://msdn.microsoft.com/magazine/mt745093.aspx)


