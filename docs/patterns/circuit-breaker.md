---
title: "Áramköri megszakító"
description: "Kezeli olyan hárítsa el a távoli szolgáltatás vagy az erőforrás való csatlakozáskor a változó időt is igénybe vehet."
keywords: "Kialakítási mintája"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories: resiliency
ms.openlocfilehash: ce110d0bbda600575d328895f2feca5aa253479d
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="circuit-breaker-pattern"></a><span data-ttu-id="dd42f-104">Áramköri megszakító minta</span><span class="sxs-lookup"><span data-stu-id="dd42f-104">Circuit Breaker pattern</span></span>

<span data-ttu-id="dd42f-105">Kezeli olyan helyreállítani, a változó időt is igénybe vehet a távoli szolgáltatás vagy az erőforrás történő csatlakozás során.</span><span class="sxs-lookup"><span data-stu-id="dd42f-105">Handle faults that might take a variable amount of time to recover from, when connecting to a remote service or resource.</span></span> <span data-ttu-id="dd42f-106">Ez javítja a stabilitás és a rugalmasságot az alkalmazás.</span><span class="sxs-lookup"><span data-stu-id="dd42f-106">This can improve the stability and resiliency of an application.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="dd42f-107">A környezetben, és probléma</span><span class="sxs-lookup"><span data-stu-id="dd42f-107">Context and problem</span></span>

<span data-ttu-id="dd42f-108">Elosztott környezetben a távoli erőforrásokhoz és szolgáltatásokhoz hívások megakadályozhatják átmeneti hibák, például lassú hálózati kapcsolatoknál, időtúllépések vagy az erőforrásokat, amelyek túljegyzett vagy átmenetileg nem érhető el.</span><span class="sxs-lookup"><span data-stu-id="dd42f-108">In a distributed environment, calls to remote resources and services can fail due to transient faults, such as slow network connections, timeouts, or the resources being overcommitted or temporarily unavailable.</span></span> <span data-ttu-id="dd42f-109">Ezek a hibák általában javítsa magukat egy rövid időn belül, és a hatékony felhő alkalmazás stratégia használatával kezelendő kell készíteni a [újrapróbálkozási mintát][retry-pattern].</span><span class="sxs-lookup"><span data-stu-id="dd42f-109">These faults typically correct themselves after a short period of time, and a robust cloud application should be prepared to handle them by using a strategy such as the [Retry pattern][retry-pattern].</span></span>

<span data-ttu-id="dd42f-110">Azonban is lehet olyan helyzetekben, ahol hibák vannak a nem várt esemény miatt, és, hogy sokkal hosszabb ideig is tarthat megoldásával kapcsolatban.</span><span class="sxs-lookup"><span data-stu-id="dd42f-110">However, there can also be situations where faults are due to unanticipated events, and that might take much longer to fix.</span></span> <span data-ttu-id="dd42f-111">Ezek a hibák között szolgáltatás teljes hiba súlyossága a egy részleges a kapcsolat megszakadása lehet.</span><span class="sxs-lookup"><span data-stu-id="dd42f-111">These faults can range in severity from a partial loss of connectivity to the complete failure of a service.</span></span> <span data-ttu-id="dd42f-112">Ezekben a helyzetekben előfordulhat, hogy az alkalmazás számára, amely nem valószínű, hogy a sikeres művelet folyamatosan újra értelmetlen, és ehelyett az alkalmazás kell gyorsan fogadja el, hogy a művelet sikertelen volt, és ennek megfelelően kezeli ezt a hibát.</span><span class="sxs-lookup"><span data-stu-id="dd42f-112">In these situations it might be pointless for an application to continually retry an operation that is unlikely to succeed, and instead the application should quickly accept that the operation has failed and handle this failure accordingly.</span></span>

<span data-ttu-id="dd42f-113">Emellett, ha a szolgáltatás foglalt, a rendszer egy része sikertelen vezethet kaszkádolt hibák.</span><span class="sxs-lookup"><span data-stu-id="dd42f-113">Additionally, if a service is very busy, failure in one part of the system might lead to cascading failures.</span></span> <span data-ttu-id="dd42f-114">Például egy művelet, amely meghívja a szolgáltatást sikerült konfigurálni egy időtúllépési bevezetéséhez, és válaszadás egy hibaüzenet, ha a szolgáltatás nem válaszol az ez idő alatt.</span><span class="sxs-lookup"><span data-stu-id="dd42f-114">For example, an operation that invokes a service could be configured to implement a timeout, and reply with a failure message if the service fails to respond within this period.</span></span> <span data-ttu-id="dd42f-115">Ezt a stratégiát azonban sok egyidejű kérés művelethez az időkorlát eléréséig letiltása esetén.</span><span class="sxs-lookup"><span data-stu-id="dd42f-115">However, this strategy could cause many concurrent requests to the same operation to be blocked until the timeout period expires.</span></span> <span data-ttu-id="dd42f-116">Ezek a blokkolt kérelmek előfordulhat, hogy tartsa a kritikus rendszer-erőforrások, például a memória, a szálak, a Helyadatbázis-kapcsolatot és a stb.</span><span class="sxs-lookup"><span data-stu-id="dd42f-116">These blocked requests might hold critical system resources such as memory, threads, database connections, and so on.</span></span> <span data-ttu-id="dd42f-117">Ezért ezeket az erőforrásokat sikerült válnak kimerül, más szeretné használni, ugyanazokat az erőforrásokat a rendszer valószínűleg független részeinek hibája okozza.</span><span class="sxs-lookup"><span data-stu-id="dd42f-117">Consequently, these resources could become exhausted, causing failure of other possibly unrelated parts of the system that need to use the same resources.</span></span> <span data-ttu-id="dd42f-118">Ebben az esetben a művelet nem sikerül azonnal, és csak meghívására történt kísérlet a szolgáltatás valószínűleg sikeres lesz esetén előnyösebb lenne.</span><span class="sxs-lookup"><span data-stu-id="dd42f-118">In these situations, it would be preferable for the operation to fail immediately, and only attempt to invoke the service if it's likely to succeed.</span></span> <span data-ttu-id="dd42f-119">Vegye figyelembe, hogy rövidebb időkorlátnak súgó a probléma, de az időtúllépés nem lehet, hogy a művelet sikertelen lesz az esetek többségében rövid akkor is, ha a a szolgáltatásnak küldött kérelemben volna végül sikeres lehet.</span><span class="sxs-lookup"><span data-stu-id="dd42f-119">Note that setting a shorter timeout might help to resolve this problem, but the timeout shouldn't be so short that the operation fails most of the time, even if the request to the service would eventually succeed.</span></span>

## <a name="solution"></a><span data-ttu-id="dd42f-120">Megoldás</span><span class="sxs-lookup"><span data-stu-id="dd42f-120">Solution</span></span>

<span data-ttu-id="dd42f-121">Az áramköri megszakító mintát, Michael Nygard által a könyv popularized [kiadás azt!](https://pragprog.com/book/mnee/release-it), megakadályozhatja, hogy az alkalmazás a ismételten valószínűleg hiba fog előfordulni egy művelet végrehajtása közben.</span><span class="sxs-lookup"><span data-stu-id="dd42f-121">The Circuit Breaker pattern, popularized by Michael Nygard in his book, [Release It!](https://pragprog.com/book/mnee/release-it), can prevent an application from repeatedly trying to execute an operation that's likely to fail.</span></span> <span data-ttu-id="dd42f-122">Várakozás nélkül lehetővé téve a rögzített vagy általi lefoglalását CPU-ciklusok kell, amíg azt meghatározza, hogy a tartalék hosszú hiba tartó.</span><span class="sxs-lookup"><span data-stu-id="dd42f-122">Allowing it to continue without waiting for the fault to be fixed or wasting CPU cycles while it determines that the fault is long lasting.</span></span> <span data-ttu-id="dd42f-123">Az áramköri megszakító mintát is lehetővé teszi a az alkalmazás észleléséhez, hogy megoldódott-e a hibát.</span><span class="sxs-lookup"><span data-stu-id="dd42f-123">The Circuit Breaker pattern also enables an application to detect whether the fault has been resolved.</span></span> <span data-ttu-id="dd42f-124">Ha a probléma úgy tűnik, hogy a javított, az alkalmazás megpróbálja meghívni a műveletet.</span><span class="sxs-lookup"><span data-stu-id="dd42f-124">If the problem appears to have been fixed, the application can try to invoke the operation.</span></span>

> <span data-ttu-id="dd42f-125">Az áramköri megszakító mintát célja az újrapróbálkozási mintát eltér.</span><span class="sxs-lookup"><span data-stu-id="dd42f-125">The purpose of the Circuit Breaker pattern is different than the Retry pattern.</span></span> <span data-ttu-id="dd42f-126">Az újrapróbálkozási mintát lehetővé teszi, hogy egy alkalmazást, majd ismételje meg a várt értéket, amely akkor lesz sikeres a művelet.</span><span class="sxs-lookup"><span data-stu-id="dd42f-126">The Retry pattern enables an application to retry an operation in the expectation that it'll succeed.</span></span> <span data-ttu-id="dd42f-127">Az áramköri megszakító mintát megakadályozza, hogy az alkalmazás, amely valószínűleg hiba fog előfordulni művelet végrehajtása a.</span><span class="sxs-lookup"><span data-stu-id="dd42f-127">The Circuit Breaker pattern prevents an application from performing an operation that is likely to fail.</span></span> <span data-ttu-id="dd42f-128">Egy alkalmazás ezen két minták kombinálhatja újrapróbálkozási minta meghívni egy áramköri megszakító keresztül művelet használatával.</span><span class="sxs-lookup"><span data-stu-id="dd42f-128">An application can combine these two patterns by using the Retry pattern to invoke an operation through a circuit breaker.</span></span> <span data-ttu-id="dd42f-129">Azonban az újrapróbálkozási logika legyen az áramköri megszakító által visszaadott kivételek-és nagybetűket, és abandon újrapróbálkozások, ha az áramköri megszakító azt jelzi, hogy a hibát nem átmeneti.</span><span class="sxs-lookup"><span data-stu-id="dd42f-129">However, the retry logic should be sensitive to any exceptions returned by the circuit breaker and abandon retry attempts if the circuit breaker indicates that a fault is not transient.</span></span>

<span data-ttu-id="dd42f-130">Egy áramköri megszakító proxyként viselkedik, előfordulhat, hogy eleget nem tevő műveletekhez.</span><span class="sxs-lookup"><span data-stu-id="dd42f-130">A circuit breaker acts as a proxy for operations that might fail.</span></span> <span data-ttu-id="dd42f-131">A proxy kell történt, és ezen információk használatával határozza meg, hogy a művelet folytatásához engedélyezze a legújabb hibák számának figyelése, vagy egyszerűen küldött azonnal kivételt.</span><span class="sxs-lookup"><span data-stu-id="dd42f-131">The proxy should monitor the number of recent failures that have occurred, and use this information to decide whether to allow the operation to proceed, or simply return an exception immediately.</span></span>

<span data-ttu-id="dd42f-132">A proxy, a következő állapotokkal egy elektromos áramköri megszakító működésére nézve, amelyek egy állapotgép valósítható meg:</span><span class="sxs-lookup"><span data-stu-id="dd42f-132">The proxy can be implemented as a state machine with the following states that mimic the functionality of an electrical circuit breaker:</span></span>

- <span data-ttu-id="dd42f-133">**Lezárt**: az alkalmazás a kérelem annak biztosítására, hogy a műveletet.</span><span class="sxs-lookup"><span data-stu-id="dd42f-133">**Closed**: The request from the application is routed to the operation.</span></span> <span data-ttu-id="dd42f-134">A proxy fenntartja a száma a legutóbbi hibákat, és ha a művelet hívása sikertelen a proxy növeli a számláló értéke.</span><span class="sxs-lookup"><span data-stu-id="dd42f-134">The proxy maintains a count of the number of recent failures, and if the call to the operation is unsuccessful the proxy increments this count.</span></span> <span data-ttu-id="dd42f-135">A legújabb hibák száma meghaladja a megadott küszöbértéket egy adott időtartamon belül, ha a proxy elhelyezi a **nyitott** állapota.</span><span class="sxs-lookup"><span data-stu-id="dd42f-135">If the number of recent failures exceeds a specified threshold within a given time period, the proxy is placed into the **Open** state.</span></span> <span data-ttu-id="dd42f-136">Ezen a ponton a proxy elindul egy időtúllépési számlálót, és idő letelte után a proxy elhelyezi a **félig nyitott** állapotát.</span><span class="sxs-lookup"><span data-stu-id="dd42f-136">At this point the proxy starts a timeout timer, and when this timer expires the proxy is placed into the **Half-Open** state.</span></span>

    > <span data-ttu-id="dd42f-137">Az időtúllépés időzítő célja próbálja megoldani a problémát, mielőtt engedélyezné az alkalmazást újra a műveletet a hibát okozó rendszer ideje.</span><span class="sxs-lookup"><span data-stu-id="dd42f-137">The purpose of the timeout timer is to give the system time to fix the problem that caused the failure before allowing the application to try to perform the operation again.</span></span>

- <span data-ttu-id="dd42f-138">**Nyissa meg**: A kérelmet az alkalmazás azonnal leáll, és kivételt küld vissza az alkalmazást.</span><span class="sxs-lookup"><span data-stu-id="dd42f-138">**Open**: The request from the application fails immediately and an exception is returned to the application.</span></span>

- <span data-ttu-id="dd42f-139">**Váltakozó nyitott**: az alkalmazás kérelmeinek korlátozott számú engedélyezettek továbbítja, és meghívja a műveletet.</span><span class="sxs-lookup"><span data-stu-id="dd42f-139">**Half-Open**: A limited number of requests from the application are allowed to pass through and invoke the operation.</span></span> <span data-ttu-id="dd42f-140">Ha ezek a kérelmek sikeres, feltételezzük, hogy a hiba, amely korábban következtében a hiba kijavítása, és az áramköri megszakító vált a **lezárva** állapota (a sikertelen kísérletek számlálója alaphelyzetbe állítása).</span><span class="sxs-lookup"><span data-stu-id="dd42f-140">If these requests are successful, it's assumed that the fault that was previously causing the failure has been fixed and the circuit breaker switches to the **Closed** state (the failure counter is reset).</span></span> <span data-ttu-id="dd42f-141">Ha a kérelmet, sikertelen, az áramköri megszakító azt feltételezi, hogy a hiba továbbra is jelen-e, hogy a rendszer visszatér a **nyitott** állapot és az időtúllépés időzítő ahhoz, hogy megkapja a további időt állítsa helyre a hibát a rendszer újraindul.</span><span class="sxs-lookup"><span data-stu-id="dd42f-141">If any request fails, the circuit breaker assumes that the fault is still present so it reverts back to the **Open** state and restarts the timeout timer to give the system a further period of time to recover from the failure.</span></span>

    > <span data-ttu-id="dd42f-142">A **félig nyitott** állapota akkor hasznos, ezáltal megakadályozhatja, hogy a helyreállítási szolgáltatás által érintett hirtelen túlterhelve.</span><span class="sxs-lookup"><span data-stu-id="dd42f-142">The **Half-Open** state is useful to prevent a recovering service from suddenly being flooded with requests.</span></span> <span data-ttu-id="dd42f-143">Szolgáltatás állítja helyre, akkor előfordulhat, hogy lehet támogatja a korlátozott mennyiségű kérést, amíg a helyreállítás, de a folyamatban lévő helyreállítás alatt a munkahelyi áramlik sok túllépi az időkorlátot a szolgáltatás, vagy végezzen ismét feladat.</span><span class="sxs-lookup"><span data-stu-id="dd42f-143">As a service recovers, it might be able to support a limited volume of requests until the recovery is complete, but while recovery is in progress a flood of work can cause the service to time out or fail again.</span></span>

![Áramköri megszakító állapotok](./_images/circuit-breaker-diagram.png)

<span data-ttu-id="dd42f-145">Az ábrán a sikertelen kísérletek számlálója használják a **lezárva** állapota időpontokat.</span><span class="sxs-lookup"><span data-stu-id="dd42f-145">In the figure, the failure counter used by the **Closed** state is time based.</span></span> <span data-ttu-id="dd42f-146">Automatikusan visszaáll a rendszeres időközönként.</span><span class="sxs-lookup"><span data-stu-id="dd42f-146">It's automatically reset at periodic intervals.</span></span> <span data-ttu-id="dd42f-147">Ez segít megakadályozni az áramköri megszakító belépjen a **nyitott** állapot, ha azt észlel alkalmi hibák.</span><span class="sxs-lookup"><span data-stu-id="dd42f-147">This helps to prevent the circuit breaker from entering the **Open** state if it experiences occasional failures.</span></span> <span data-ttu-id="dd42f-148">A hiba küszöbértékét, amely utazás közben az áramköri megszakító be a **nyitott** állapot csak akkor érhető el, amikor adott számú hiba történt egy adott időszakban.</span><span class="sxs-lookup"><span data-stu-id="dd42f-148">The failure threshold that trips the circuit breaker into the **Open** state is only reached when a specified number of failures have occurred during a specified interval.</span></span> <span data-ttu-id="dd42f-149">A számláló által használt a **félig nyitott** állapotát rögzíti a hívási a művelet sikeres próbálkozások számát.</span><span class="sxs-lookup"><span data-stu-id="dd42f-149">The counter used by the **Half-Open** state records the number of successful attempts to invoke the operation.</span></span> <span data-ttu-id="dd42f-150">Az áramköri megszakító visszatér a **lezárva** állapot után a megadott számú egymást követő műveletek meghívása sikeresek voltak.</span><span class="sxs-lookup"><span data-stu-id="dd42f-150">The circuit breaker reverts to the **Closed** state after a specified number of consecutive operation invocations have been successful.</span></span> <span data-ttu-id="dd42f-151">Bármely hívás sikertelen lesz, ha beírja-e az áramköri megszakító a **nyissa meg** azonnal állapotát, és a sikeres a számláló a következő alkalommal, akkor visszaáll a **félig nyitott** állapotát.</span><span class="sxs-lookup"><span data-stu-id="dd42f-151">If any invocation fails, the circuit breaker enters the **Open** state immediately and the success counter will be reset the next time it enters the **Half-Open** state.</span></span>

> <span data-ttu-id="dd42f-152">Hogyan állítja helyre a rendszer kezeli kívülről, valószínűleg visszaállítása vagy újraindítása sikertelen összetevő vagy a hálózati kapcsolat javítása.</span><span class="sxs-lookup"><span data-stu-id="dd42f-152">How the system recovers is handled externally, possibly by restoring or restarting a failed component or repairing a network connection.</span></span>

<span data-ttu-id="dd42f-153">Az áramköri megszakító mintát stabilitását biztosít, miközben a rendszer egy történt hiba után állítja helyre, és minimálisra csökkenti a teljesítményre gyakorolt hatása.</span><span class="sxs-lookup"><span data-stu-id="dd42f-153">The Circuit Breaker pattern provides stability while the system recovers from a failure and minimizes the impact on performance.</span></span> <span data-ttu-id="dd42f-154">Ez a Súgó gombra a rendszer a válaszidő karbantartása gyorsan visszautasítja a kérelmet, amely valószínűleg hiba fog előfordulni művelet, hanem a Várakozás a művelet időtúllépés, vagy soha nem adja vissza.</span><span class="sxs-lookup"><span data-stu-id="dd42f-154">It can help to maintain the response time of the system by quickly rejecting a request for an operation that's likely to fail, rather than waiting for the operation to time out, or never return.</span></span> <span data-ttu-id="dd42f-155">Az áramköri megszakító kivált egy eseményt minden alkalommal, amikor az állapot változik, ha ezt az információt a részét a rendszer az áramköri megszakító által védett állapotának figyelésére, illetve riasztást a rendszergazda, ha egy áramköri megszakító utazás közben történő használható-e a **megnyitása** állapotát.</span><span class="sxs-lookup"><span data-stu-id="dd42f-155">If the circuit breaker raises an event each time it changes state, this information can be used to monitor the health of the part of the system protected by the circuit breaker, or to alert an administrator when a circuit breaker trips to the **Open** state.</span></span>

<span data-ttu-id="dd42f-156">A minta személyre szabható legyen, és ezeket szerint, a lehetséges hiba.</span><span class="sxs-lookup"><span data-stu-id="dd42f-156">The pattern is customizable and can be adapted according to the type of the possible failure.</span></span> <span data-ttu-id="dd42f-157">Például egy növekvő időtúllépés időzítő egy áramköri megszakító alkalmazhat.</span><span class="sxs-lookup"><span data-stu-id="dd42f-157">For example, you can apply an increasing timeout timer to a circuit breaker.</span></span> <span data-ttu-id="dd42f-158">Az áramköri megszakító a sikerült elhelyezni a **nyitott** néhány másodpercen belül kezdetben állapotát, és ezután Ha a hiba nem sikerült megoldani az időtúllépési néhány percet, és így tovább.</span><span class="sxs-lookup"><span data-stu-id="dd42f-158">You could place the circuit breaker in the **Open** state for a few seconds initially, and then if the failure hasn't been resolved increase the timeout to a few minutes, and so on.</span></span> <span data-ttu-id="dd42f-159">Egyes esetekben nem pedig a **nyitott** állapot adatszolgáltató hiba és kivételt, ez például akkor lehet hasznos, amely elsősorban az alkalmazás az alapértelmezett értéket.</span><span class="sxs-lookup"><span data-stu-id="dd42f-159">In some cases, rather than the **Open** state returning failure and raising an exception, it could be useful to return a default value that is meaningful to the application.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="dd42f-160">Problémákat és szempontok</span><span class="sxs-lookup"><span data-stu-id="dd42f-160">Issues and considerations</span></span>

<span data-ttu-id="dd42f-161">Ez a kialakítás megvalósítása meghatározásakor a következő szempontokat kell figyelembe vennie:</span><span class="sxs-lookup"><span data-stu-id="dd42f-161">You should consider the following points when deciding how to implement this pattern:</span></span>

<span data-ttu-id="dd42f-162">**Kivételkezelés**.</span><span class="sxs-lookup"><span data-stu-id="dd42f-162">**Exception Handling**.</span></span> <span data-ttu-id="dd42f-163">Egy alkalmazás egy áramköri megszakító keresztül művelet meghívása kezelni a kivételek jelenik meg, ha a művelet nem érhető el kell készíteni.</span><span class="sxs-lookup"><span data-stu-id="dd42f-163">An application invoking an operation through a circuit breaker must be prepared to handle the exceptions raised if the operation is unavailable.</span></span> <span data-ttu-id="dd42f-164">Kivételek kezelése lesz az adott alkalmazás.</span><span class="sxs-lookup"><span data-stu-id="dd42f-164">The way exceptions are handled will be application specific.</span></span> <span data-ttu-id="dd42f-165">Például egy alkalmazás sikerült ideiglenesen csökkentheti a funkciókat, végrehajthatja ugyanezt a feladatot, vagy az beszerzése ugyanazokat az adatokat, a másik művelet meghívása vagy jelentse a kivétel a felhasználó és kérje meg, hogy később próbálja meg újból.</span><span class="sxs-lookup"><span data-stu-id="dd42f-165">For example, an application could temporarily degrade its functionality, invoke an alternative operation to try to perform the same task or obtain the same data, or report the exception to the user and ask them to try again later.</span></span>

<span data-ttu-id="dd42f-166">**Kivételek típusú**.</span><span class="sxs-lookup"><span data-stu-id="dd42f-166">**Types of Exceptions**.</span></span> <span data-ttu-id="dd42f-167">A kérelem sikertelen lehet, amelyek jelezhetik a többinél szigorúbb típusú hiba számos oka.</span><span class="sxs-lookup"><span data-stu-id="dd42f-167">A request might fail for many reasons, some of which might indicate a more severe type of failure than others.</span></span> <span data-ttu-id="dd42f-168">Például egy kérelem meghiúsulhat, mert a távoli szolgáltatás összeomlott és helyreállítása, néhány percet vesz igénybe, vagy mert a szolgáltatás ideiglenesen túlterhelt alatt időtúllépés miatt.</span><span class="sxs-lookup"><span data-stu-id="dd42f-168">For example, a request might fail because a remote service has crashed and will take several minutes to recover, or because of a timeout due to the service being temporarily overloaded.</span></span> <span data-ttu-id="dd42f-169">Egy áramköri megszakító megvizsgálhatja a kivételeket, és állítsa be ezeket a kivételeket természetétől függően a stratégia típusú lehet.</span><span class="sxs-lookup"><span data-stu-id="dd42f-169">A circuit breaker might be able to examine the types of exceptions that occur and adjust its strategy depending on the nature of these exceptions.</span></span> <span data-ttu-id="dd42f-170">Például akkor lehet szükség időtúllépési kivételek az áramköri megszakító való trip nagyobb számú a **nyitott** állapot, a hiba oka, hogy a szolgáltatás nem volt teljesen elérhető száma képest.</span><span class="sxs-lookup"><span data-stu-id="dd42f-170">For example, it might require a larger number of timeout exceptions to trip the circuit breaker to the **Open** state compared to the number of failures due to the service being completely unavailable.</span></span>

<span data-ttu-id="dd42f-171">**Naplózás**.</span><span class="sxs-lookup"><span data-stu-id="dd42f-171">**Logging**.</span></span> <span data-ttu-id="dd42f-172">Egy áramköri megszakító kell naplófájl összes sikertelen kérések (és valószínűleg sikeres kérelmek) ahhoz, hogy a rendszergazda a művelet állapotának figyelése.</span><span class="sxs-lookup"><span data-stu-id="dd42f-172">A circuit breaker should log all failed requests (and possibly successful requests) to enable an administrator to monitor the health of the operation.</span></span>

<span data-ttu-id="dd42f-173">**Helyreállíthatósága**.</span><span class="sxs-lookup"><span data-stu-id="dd42f-173">**Recoverability**.</span></span> <span data-ttu-id="dd42f-174">Az áramköri megszakító az általa védett művelet valószínűleg helyreállítási szerkezet megfelelően konfigurálni kell.</span><span class="sxs-lookup"><span data-stu-id="dd42f-174">You should configure the circuit breaker to match the likely recovery pattern of the operation it's protecting.</span></span> <span data-ttu-id="dd42f-175">Például, ha az áramköri megszakító marad a **nyitott** állapot hosszú ideig felvethet kivételek akkor is, ha a hiba okát lett feloldva.</span><span class="sxs-lookup"><span data-stu-id="dd42f-175">For example, if the circuit breaker remains in the **Open** state for a long period, it could raise exceptions even if the reason for the failure has been resolved.</span></span> <span data-ttu-id="dd42f-176">Ehhez hasonlóan a áramköri megszakító ingadozik, és csökkenti az alkalmazások válaszidejét, ha vált a **nyissa meg** állapotát a **félig nyitott** túl gyorsan állapot.</span><span class="sxs-lookup"><span data-stu-id="dd42f-176">Similarly, a circuit breaker could fluctuate and reduce the response times of applications if it switches from the **Open** state to the **Half-Open** state too quickly.</span></span>

<span data-ttu-id="dd42f-177">**Tesztelése sikertelen műveletek**.</span><span class="sxs-lookup"><span data-stu-id="dd42f-177">**Testing Failed Operations**.</span></span> <span data-ttu-id="dd42f-178">Az a **nyissa meg** állapota, nem pedig egy időzítő segítségével határozza meg, váltson át a **félig nyitott** állapotba kerül, a áramköri megszakító helyette rendszeresen ping paranccsal elérhető a távoli szolgáltatás vagy az erőforrás annak meghatározásához, hogy rendelkezik ismét elérhetővé válik.</span><span class="sxs-lookup"><span data-stu-id="dd42f-178">In the **Open** state, rather than using a timer to determine when to switch to the **Half-Open** state, a circuit breaker can instead periodically ping the remote service or resource to determine whether it's become available again.</span></span> <span data-ttu-id="dd42f-179">A ping művelet, amely korábban nem sikerült meghívni kísérlet formájában is beletelhet, vagy azt egy különleges műveletet a távoli kifejezetten a teszt a szolgáltatás által biztosított szerint használhat a [állapotfigyelő végpont Figyelési mintát](health-endpoint-monitoring.md).</span><span class="sxs-lookup"><span data-stu-id="dd42f-179">This ping could take the form of an attempt to invoke an operation that had previously failed, or it could use a special operation provided by the remote service specifically for testing the health of the service, as described by the [Health Endpoint Monitoring pattern](health-endpoint-monitoring.md).</span></span>

<span data-ttu-id="dd42f-180">**Kézi felülbírálás**.</span><span class="sxs-lookup"><span data-stu-id="dd42f-180">**Manual Override**.</span></span> <span data-ttu-id="dd42f-181">A rendszer, ha a helyreállítási a sikertelen művelet ideje rendkívül változó hogy a rendszer kézi alaphelyzetbe állítása, amely lehetővé teszi a rendszergazdának, zárja be a áramköri megszakító (és alaphelyzetbe állítja a sikertelen kísérletek számlálója) beállítás számára előnyös.</span><span class="sxs-lookup"><span data-stu-id="dd42f-181">In a system where the recovery time for a failing operation is extremely variable, it's beneficial to provide a manual reset option that enables an administrator to close a circuit breaker (and reset the failure counter).</span></span> <span data-ttu-id="dd42f-182">Hasonlóképpen, a rendszergazda kényszerítheti egy áramköri megszakító be a **nyissa meg** állapotban (Újraindítás a időtúllépés időzítő) Ha a művelet védi az áramköri megszakító: átmenetileg nem érhető el.</span><span class="sxs-lookup"><span data-stu-id="dd42f-182">Similarly, an administrator could force a circuit breaker into the **Open** state (and restart the timeout timer) if the operation protected by the circuit breaker is temporarily unavailable.</span></span>

<span data-ttu-id="dd42f-183">**Párhuzamossági**.</span><span class="sxs-lookup"><span data-stu-id="dd42f-183">**Concurrency**.</span></span> <span data-ttu-id="dd42f-184">Az azonos áramköri megszakító nagyszámú egyidejű alkalmazáspéldányra fért hozzá.</span><span class="sxs-lookup"><span data-stu-id="dd42f-184">The same circuit breaker could be accessed by a large number of concurrent instances of an application.</span></span> <span data-ttu-id="dd42f-185">Végrehajtása ne egyidejű-kérések blokkolása, vagy adja hozzá túlzott terhelés minden olyan művelet hívására.</span><span class="sxs-lookup"><span data-stu-id="dd42f-185">The implementation shouldn't block concurrent requests or add excessive overhead to each call to an operation.</span></span>

<span data-ttu-id="dd42f-186">**Erőforrás megkülönböztetési**.</span><span class="sxs-lookup"><span data-stu-id="dd42f-186">**Resource Differentiation**.</span></span> <span data-ttu-id="dd42f-187">Ügyeljen arra, hogy ha egy egyetlen áramköri megszakító használ egy típusú erőforrás fennáll-e több alapul szolgáló független is.</span><span class="sxs-lookup"><span data-stu-id="dd42f-187">Be careful when using a single circuit breaker for one type of resource if there might be multiple underlying independent providers.</span></span> <span data-ttu-id="dd42f-188">Például a tárolóban, amely több szilánkok tartalmazza, egy shard lehet teljes elérhető amíg egy másik ütközik egy ideiglenes probléma.</span><span class="sxs-lookup"><span data-stu-id="dd42f-188">For example, in a data store that contains multiple shards, one shard might be fully accessible while another is experiencing a temporary issue.</span></span> <span data-ttu-id="dd42f-189">A következő használati helyzetekben a hibaválaszok egyesítve lesznek, ha egy alkalmazás előfordulhat, hogy próbáljon meg hozzáférni bizonyos szilánkok akkor is, ha a hiba nagyon valószínű, amíg más szilánkok elérésére blokkolhatja, annak ellenére, hogy valószínűleg sikeres.</span><span class="sxs-lookup"><span data-stu-id="dd42f-189">If the error responses in these scenarios are merged, an application might try to access some shards even when failure is highly likely, while access to other shards might be blocked even though it's likely to succeed.</span></span>

<span data-ttu-id="dd42f-190">**Gyorsított áramkör legfrissebb**.</span><span class="sxs-lookup"><span data-stu-id="dd42f-190">**Accelerated Circuit Breaking**.</span></span> <span data-ttu-id="dd42f-191">Néha hibaválaszt út azonnal, és minimális időtartama tripped marad az áramköri megszakító elegendő információt tartalmazhat.</span><span class="sxs-lookup"><span data-stu-id="dd42f-191">Sometimes a failure response can contain enough information for the circuit breaker to trip immediately and stay tripped for a minimum amount of time.</span></span> <span data-ttu-id="dd42f-192">Például egy megosztott erőforráson, amely túl van terhelve válaszát hiba oka lehet, hogy egy azonnali újrapróbálkozási nem ajánlott, és, hogy az alkalmazás kell helyette próbálkozzon újra néhány perc múlva.</span><span class="sxs-lookup"><span data-stu-id="dd42f-192">For example, the error response from a shared resource that's overloaded could indicate that an immediate retry isn't recommended and that the application should instead try again in a few minutes.</span></span>

> [!NOTE]
> <span data-ttu-id="dd42f-193">A szolgáltatás adhat vissza HTTP 429 (túl sok kérelem) Ha az ügyfél szabályozás, vagy a HTTP 503-as (a szolgáltatás nem érhető el), ha a szolgáltatás már nem érhető el.</span><span class="sxs-lookup"><span data-stu-id="dd42f-193">A service can return HTTP 429 (Too Many Requests) if it is throttling the client, or HTTP 503 (Service Unavailable) if the service is not currently available.</span></span> <span data-ttu-id="dd42f-194">A válasz tartalmazhatnak további adatokat, például az a késleltetés várható időtartama.</span><span class="sxs-lookup"><span data-stu-id="dd42f-194">The response can include additional information, such as the anticipated duration of the delay.</span></span>

<span data-ttu-id="dd42f-195">**Visszajátszását sikertelen kérelmek**.</span><span class="sxs-lookup"><span data-stu-id="dd42f-195">**Replaying Failed Requests**.</span></span> <span data-ttu-id="dd42f-196">Az a **nyitott** állapotában ahelyett, hogy egyszerűen sikertelen gyorsan, egy áramköri megszakító sikerült is jegyezze fel az egyes kérelmek naplóba részleteit, és gondoskodik ezeket a kérelmeket a rendszer kell játssza, amikor a távoli erőforrás vagy a szolgáltatás elérhetővé válik.</span><span class="sxs-lookup"><span data-stu-id="dd42f-196">In the **Open** state, rather than simply failing quickly, a circuit breaker could also record the details of each request to a journal and arrange for these requests to be replayed when the remote resource or service becomes available.</span></span>

<span data-ttu-id="dd42f-197">**Külső szolgáltatások nem megfelelő időtúllépése**.</span><span class="sxs-lookup"><span data-stu-id="dd42f-197">**Inappropriate Timeouts on External Services**.</span></span> <span data-ttu-id="dd42f-198">Előfordulhat, hogy egy áramköri megszakító nem teljes védelme érdekében az alkalmazások külső szolgáltatások egy hosszabb időkorlát konfigurált teljesítő műveletek.</span><span class="sxs-lookup"><span data-stu-id="dd42f-198">A circuit breaker might not be able to fully protect applications from operations that fail in external services that are configured with a lengthy timeout period.</span></span> <span data-ttu-id="dd42f-199">Az időtúllépési érték túl hosszú, ha a szál fut egy áramköri megszakító blokkolhatja hosszú időn keresztül a áramköri megszakító azt jelzi, hogy sikerült-e a művelet előtt.</span><span class="sxs-lookup"><span data-stu-id="dd42f-199">If the timeout is too long, a thread running a circuit breaker might be blocked for an extended period before the circuit breaker indicates that the operation has failed.</span></span> <span data-ttu-id="dd42f-200">Megadott idő példányok is megpróbálja meghívni az áramköri megszakító szolgáltatásba, és szálak előtt minden jelentős számú lefoglalják számos egyéb alkalmazás sikeres lesz.</span><span class="sxs-lookup"><span data-stu-id="dd42f-200">In this time, many other application instances might also try to invoke the service through the circuit breaker and tie up a significant number of threads before they all fail.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="dd42f-201">Mikor érdemes használni ezt a mintát</span><span class="sxs-lookup"><span data-stu-id="dd42f-201">When to use this pattern</span></span>

<span data-ttu-id="dd42f-202">Ez a minta használata:</span><span class="sxs-lookup"><span data-stu-id="dd42f-202">Use this pattern:</span></span>

- <span data-ttu-id="dd42f-203">Megakadályozhatja, hogy egy alkalmazás adott távoli szolgáltatás, vagy egy megosztott erőforrás elérésére, ha ez a művelet nagyon valószínű, sikertelen lesz.</span><span class="sxs-lookup"><span data-stu-id="dd42f-203">To prevent an application from trying to invoke a remote service or access a shared resource if this operation is highly likely to fail.</span></span>

<span data-ttu-id="dd42f-204">Ez a minta nem ajánlott:</span><span class="sxs-lookup"><span data-stu-id="dd42f-204">This pattern isn't recommended:</span></span>

- <span data-ttu-id="dd42f-205">Az alkalmazások, például a memória adatszerkezet helyi titkos erőforrásainak kezelése elérésére.</span><span class="sxs-lookup"><span data-stu-id="dd42f-205">For handling access to local private resources in an application, such as in-memory data structure.</span></span> <span data-ttu-id="dd42f-206">Ebben a környezetben egy áramköri megszakító használatával volna hozzá protokollterhelés a rendszer.</span><span class="sxs-lookup"><span data-stu-id="dd42f-206">In this environment, using a circuit breaker would add overhead to your system.</span></span>
- <span data-ttu-id="dd42f-207">Az alkalmazások üzleti logikát a kivételek kezelése helyett.</span><span class="sxs-lookup"><span data-stu-id="dd42f-207">As a substitute for handling exceptions in the business logic of your applications.</span></span>

## <a name="example"></a><span data-ttu-id="dd42f-208">Példa</span><span class="sxs-lookup"><span data-stu-id="dd42f-208">Example</span></span>

<span data-ttu-id="dd42f-209">A webalkalmazás a lapok számos feltöltött származó adatok egy külső szolgáltatás.</span><span class="sxs-lookup"><span data-stu-id="dd42f-209">In a web application, several of the pages are populated with data retrieved from an external service.</span></span> <span data-ttu-id="dd42f-210">A rendszer megvalósítja a minimális gyorsítótárazást, ha ezeken a lapokon a legtöbb találatok miatt a szolgáltatás oda-vissza.</span><span class="sxs-lookup"><span data-stu-id="dd42f-210">If the system implements minimal caching, most hits to these pages will cause a round trip to the service.</span></span> <span data-ttu-id="dd42f-211">A szolgáltatás a webes alkalmazás közötti kapcsolatok sikerült kell konfigurálni a határidőn (jellemzően 60 másodperc), és ha a szolgáltatás ennyi időn belül nem válaszol az egyes weblapon logikai feltételezik, hogy a szolgáltatás nem érhető el, és kivételt jelez.</span><span class="sxs-lookup"><span data-stu-id="dd42f-211">Connections from the web application to the service could be configured with a timeout period (typically 60 seconds), and if the service doesn't respond in this time the logic in each web page will assume that the service is unavailable and throw an exception.</span></span>

<span data-ttu-id="dd42f-212">Azonban ha a szolgáltatás leáll és a rendszer túlzottan megnő, felhasználók sikerült kényszeríthető, várjon, amíg egy kivétel előtt 60 másodperc.</span><span class="sxs-lookup"><span data-stu-id="dd42f-212">However, if the service fails and the system is very busy, users could be forced to wait for up to 60 seconds before an exception occurs.</span></span> <span data-ttu-id="dd42f-213">Végül erőforrások, például a memória, a kapcsolatok és a szálak sikerült kell kimerül, akadályozza meg, hogy más felhasználók csatlakozzanak a rendszer akkor is, ha nem érik lapok, amelyek az adatok beolvasása a szolgáltatástól.</span><span class="sxs-lookup"><span data-stu-id="dd42f-213">Eventually resources such as memory, connections, and threads could be exhausted, preventing other users from connecting to the system, even if they aren't accessing pages that retrieve data from the service.</span></span>

<span data-ttu-id="dd42f-214">A rendszer további kiszolgálók hozzáadásával és végrehajtásával, terheléselosztás előfordulhat, hogy erőforrások válnak kimerül, de azt nem megoldani a problémát, mert a felhasználói kérelmek továbbra is lefagyottnak késleltetés és a webkiszolgálók végül továbbra is futtathat a skálázás erőforrások.</span><span class="sxs-lookup"><span data-stu-id="dd42f-214">Scaling the system by adding further web servers and implementing load balancing might delay when resources become exhausted, but it won't resolve the issue because user requests will still be unresponsive and all web servers could still eventually run out of resources.</span></span>

<span data-ttu-id="dd42f-215">A logika, amely csatlakozik a szolgáltatáshoz, és lekéri az adatokat egy áramköri megszakító az alkalmazásburkoló segíthet a probléma megoldásához, és a szolgáltatás hibája több elegantly kezelését.</span><span class="sxs-lookup"><span data-stu-id="dd42f-215">Wrapping the logic that connects to the service and retrieves the data in a circuit breaker could help to solve this problem and handle the service failure more elegantly.</span></span> <span data-ttu-id="dd42f-216">Felhasználói kérelmek továbbra is sikertelen lesz, de és gyorsabban fog-e sikertelen, és az erőforrások nem tiltható le.</span><span class="sxs-lookup"><span data-stu-id="dd42f-216">User requests will still fail, but they'll fail more quickly and the resources won't be blocked.</span></span>

<span data-ttu-id="dd42f-217">A `CircuitBreaker` osztály egy olyan objektum, amely megvalósítja az áramköri megszakító vonatkozó állapotadatokat megőrzi a `ICircuitBreakerStateStore` felülete az alábbi kódban látható.</span><span class="sxs-lookup"><span data-stu-id="dd42f-217">The `CircuitBreaker` class maintains state information about a circuit breaker in an object that implements the `ICircuitBreakerStateStore` interface shown in the following code.</span></span>

```csharp
interface ICircuitBreakerStateStore
{
  CircuitBreakerStateEnum State { get; }

  Exception LastException { get; }

  DateTime LastStateChangedDateUtc { get; }

  void Trip(Exception ex);

  void Reset();

  void HalfOpen();

  bool IsClosed { get; }
}
```

<span data-ttu-id="dd42f-218">A `State` tulajdonság jelzi az áramköri megszakító aktuális állapotát, és akkor **nyitott**, **HalfOpen**, vagy **lezárva** a általdefiniált`CircuitBreakerStateEnum`enumerálása.</span><span class="sxs-lookup"><span data-stu-id="dd42f-218">The `State` property indicates the current state of the circuit breaker, and will be either **Open**, **HalfOpen**, or **Closed** as defined by the `CircuitBreakerStateEnum` enumeration.</span></span> <span data-ttu-id="dd42f-219">A `IsClosed` tulajdonság értéke true, ha az áramköri megszakító le van zárva, de hamis, ha megnyitott vagy félig nyitva kell.</span><span class="sxs-lookup"><span data-stu-id="dd42f-219">The `IsClosed` property should be true if the circuit breaker is closed, but false if it's open or half open.</span></span> <span data-ttu-id="dd42f-220">A `Trip` metódus a nyitott állapotba vált, az áramköri megszakító állapotát, és rögzíti a kivételt, ami miatt a módosítás állapotú, és a dátum és időpont, amikor a kivétel történt.</span><span class="sxs-lookup"><span data-stu-id="dd42f-220">The `Trip` method switches the state of the circuit breaker to the open state and records the exception that caused the change in state, together with the date and time that the exception occurred.</span></span> <span data-ttu-id="dd42f-221">A `LastException` és a `LastStateChangedDateUtc` tulajdonságok ezeket az információkat adnak vissza.</span><span class="sxs-lookup"><span data-stu-id="dd42f-221">The `LastException` and the `LastStateChangedDateUtc` properties return this information.</span></span> <span data-ttu-id="dd42f-222">A `Reset` metódus bezárja az áramköri megszakító és a `HalfOpen` módszert állítja be az áramköri megszakító fele megnyitásához.</span><span class="sxs-lookup"><span data-stu-id="dd42f-222">The `Reset` method closes the circuit breaker, and the `HalfOpen` method sets the circuit breaker to half open.</span></span>

<span data-ttu-id="dd42f-223">A `InMemoryCircuitBreakerStateStore` példában osztály tartalmaz egy megvalósítása a `ICircuitBreakerStateStore` felületet.</span><span class="sxs-lookup"><span data-stu-id="dd42f-223">The `InMemoryCircuitBreakerStateStore` class in the example contains an implementation of the `ICircuitBreakerStateStore` interface.</span></span> <span data-ttu-id="dd42f-224">A `CircuitBreaker` osztály hoz létre a megszakító állapot tárolásához az osztály egy példányát.</span><span class="sxs-lookup"><span data-stu-id="dd42f-224">The `CircuitBreaker` class creates an instance of this class to hold the state of the circuit breaker.</span></span>

<span data-ttu-id="dd42f-225">A `ExecuteAction` metódust a `CircuitBreaker` osztály becsomagolja művelet, mint a megadott egy `Action` delegálni.</span><span class="sxs-lookup"><span data-stu-id="dd42f-225">The `ExecuteAction` method in the `CircuitBreaker` class wraps an operation, specified as an `Action` delegate.</span></span> <span data-ttu-id="dd42f-226">Ha az áramköri megszakító le van zárva, `ExecuteAction` meghívja a `Action` delegálni.</span><span class="sxs-lookup"><span data-stu-id="dd42f-226">If the circuit breaker is closed, `ExecuteAction` invokes the `Action` delegate.</span></span> <span data-ttu-id="dd42f-227">A művelet sikertelen lesz, ha meghívja-e egy kivételkezelőbe `TrackException`, amely megnyitásához áramköri megszakító állapotúra állítja.</span><span class="sxs-lookup"><span data-stu-id="dd42f-227">If the operation fails, an exception handler calls `TrackException`, which sets the circuit breaker state to open.</span></span> <span data-ttu-id="dd42f-228">Az alábbi példakód mutatja be a folyamatot.</span><span class="sxs-lookup"><span data-stu-id="dd42f-228">The following code example highlights this flow.</span></span>

```csharp
public class CircuitBreaker
{
  private readonly ICircuitBreakerStateStore stateStore =
    CircuitBreakerStateStoreFactory.GetCircuitBreakerStateStore();

  private readonly object halfOpenSyncObject = new object ();
  ...
  public bool IsClosed { get { return stateStore.IsClosed; } }

  public bool IsOpen { get { return !IsClosed; } }

  public void ExecuteAction(Action action)
  {
    ...
    if (IsOpen)
    {
      // The circuit breaker is Open.
      ... (see code sample below for details)
    }

    // The circuit breaker is Closed, execute the action.
    try
    {
      action();
    }
    catch (Exception ex)
    {
      // If an exception still occurs here, simply
      // retrip the breaker immediately.
      this.TrackException(ex);

      // Throw the exception so that the caller can tell
      // the type of exception that was thrown.
      throw;
    }
  }

  private void TrackException(Exception ex)
  {
    // For simplicity in this example, open the circuit breaker on the first exception.
    // In reality this would be more complex. A certain type of exception, such as one
    // that indicates a service is offline, might trip the circuit breaker immediately.
    // Alternatively it might count exceptions locally or across multiple instances and
    // use this value over time, or the exception/success ratio based on the exception
    // types, to open the circuit breaker.
    this.stateStore.Trip(ex);
  }
}
```

<span data-ttu-id="dd42f-229">A következő példa bemutatja a (nincs megadva az előző példából) futtatott kód is, ha az áramköri megszakító nem lezárja.</span><span class="sxs-lookup"><span data-stu-id="dd42f-229">The following example shows the code (omitted from the previous example) that is executed if the circuit breaker isn't closed.</span></span> <span data-ttu-id="dd42f-230">Először ellenőrzi, ha az áramköri megszakító van nyitva, a helyi által megadott időtartamnál hosszabb ideig `OpenToHalfOpenWaitTime` mező mellett a `CircuitBreaker` osztály.</span><span class="sxs-lookup"><span data-stu-id="dd42f-230">It first checks if the circuit breaker has been open for a period longer than the time specified by the local `OpenToHalfOpenWaitTime` field in the `CircuitBreaker` class.</span></span> <span data-ttu-id="dd42f-231">Ha ez a helyzet, a `ExecuteAction` metódus állítja be az áramköri megszakító váltakozó megnyitásához, majd megpróbálja végrehajtani a műveletet, amelyet a `Action` delegálni.</span><span class="sxs-lookup"><span data-stu-id="dd42f-231">If this is the case, the `ExecuteAction` method sets the circuit breaker to half open, then tries to perform the operation specified by the `Action` delegate.</span></span>

<span data-ttu-id="dd42f-232">Ha a művelet sikeres, az áramköri megszakító lezárt állapotában lesz visszaállítva.</span><span class="sxs-lookup"><span data-stu-id="dd42f-232">If the operation is successful, the circuit breaker is reset to the closed state.</span></span> <span data-ttu-id="dd42f-233">A művelet sikertelen lesz, ha vissza a nyitott állapotban) és időpontot (a kivétel történt a frissül, hogy az áramköri megszakító további időtartamra újra a művelet végrehajtása előtt megvárja a azokat kioldják.</span><span class="sxs-lookup"><span data-stu-id="dd42f-233">If the operation fails, it is tripped back to the open state and the time the exception occurred is updated so that the circuit breaker will wait for a further period before trying to perform the operation again.</span></span>

<span data-ttu-id="dd42f-234">Ha az áramköri megszakító csak van nyitva egy rövid ideig kevesebb, mint `OpenToHalfOpenWaitTime` érték, a `ExecuteAction` metódus egyszerűen jelez a `CircuitBreakerOpenException` kivétel és a hibát, ami miatt az áramköri megszakító való áttéréssel megnyitott állapotát jelzi.</span><span class="sxs-lookup"><span data-stu-id="dd42f-234">If the circuit breaker has only been open for a short time, less than the `OpenToHalfOpenWaitTime` value, the `ExecuteAction` method simply throws a `CircuitBreakerOpenException` exception and returns the error that caused the circuit breaker to transition to the open state.</span></span>

<span data-ttu-id="dd42f-235">Emellett használ egy zároláshoz az áramköri megszakító megakadályozása egyidejű hívás a művelet végrehajtása, amíg félig nyitva.</span><span class="sxs-lookup"><span data-stu-id="dd42f-235">Additionally, it uses a lock to prevent the circuit breaker from trying to perform concurrent calls to the operation while it's half open.</span></span> <span data-ttu-id="dd42f-236">Egy párhuzamos próbáltak meghívni a művelet automatikusan elvégezhető, mintha az áramköri megszakító meg volt nyitva, és azt fogja sikertelen, és a kivételt, a későbbiekben olvashat.</span><span class="sxs-lookup"><span data-stu-id="dd42f-236">A concurrent attempt to invoke the operation will be handled as if the circuit breaker was open, and it'll fail with an exception as described later.</span></span>

```csharp
    ...
    if (IsOpen)
    {
      // The circuit breaker is Open. Check if the Open timeout has expired.
      // If it has, set the state to HalfOpen. Another approach might be to
      // check for the HalfOpen state that had be set by some other operation.
      if (stateStore.LastStateChangedDateUtc + OpenToHalfOpenWaitTime < DateTime.UtcNow)
      {
        // The Open timeout has expired. Allow one operation to execute. Note that, in
        // this example, the circuit breaker is set to HalfOpen after being
        // in the Open state for some period of time. An alternative would be to set
        // this using some other approach such as a timer, test method, manually, and
        // so on, and check the state here to determine how to handle execution
        // of the action.
        // Limit the number of threads to be executed when the breaker is HalfOpen.
        // An alternative would be to use a more complex approach to determine which
        // threads or how many are allowed to execute, or to execute a simple test
        // method instead.
        bool lockTaken = false;
        try
        {
          Monitor.TryEnter(halfOpenSyncObject, ref lockTaken)
          if (lockTaken)
          {
            // Set the circuit breaker state to HalfOpen.
            stateStore.HalfOpen();

            // Attempt the operation.
            action();

            // If this action succeeds, reset the state and allow other operations.
            // In reality, instead of immediately returning to the Closed state, a counter
            // here would record the number of successful operations and return the
            // circuit breaker to the Closed state only after a specified number succeed.
            this.stateStore.Reset();
            return;
          }
          catch (Exception ex)
          {
            // If there's still an exception, trip the breaker again immediately.
            this.stateStore.Trip(ex);

            // Throw the exception so that the caller knows which exception occurred.
            throw;
          }
          finally
          {
            if (lockTaken)
            {
              Monitor.Exit(halfOpenSyncObject);
            }
          }
        }
      }
      // The Open timeout hasn't yet expired. Throw a CircuitBreakerOpen exception to
      // inform the caller that the call was not actually attempted,
      // and return the most recent exception received.
      throw new CircuitBreakerOpenException(stateStore.LastException);
    }
    ...
```

<span data-ttu-id="dd42f-237">Használatához a `CircuitBreaker` objektum védelméhez egy műveletet, az alkalmazás létrehoz egy új a `CircuitBreaker` osztályhoz, és meghívja a `ExecuteAction` metódus, a művelet lehet elvégezni, mivel a paraméter megadásával.</span><span class="sxs-lookup"><span data-stu-id="dd42f-237">To use a `CircuitBreaker` object to protect an operation, an application creates an instance of the `CircuitBreaker` class and invokes the `ExecuteAction` method, specifying the operation to be performed as the parameter.</span></span> <span data-ttu-id="dd42f-238">Az alkalmazás tényleges kell készíteni a `CircuitBreakerOpenException` kivétel, ha a művelet sikertelen, mert az áramköri megszakító meg nyitva.</span><span class="sxs-lookup"><span data-stu-id="dd42f-238">The application should be prepared to catch the `CircuitBreakerOpenException` exception if the operation fails because the circuit breaker is open.</span></span> <span data-ttu-id="dd42f-239">A következő kód egy példát mutat be:</span><span class="sxs-lookup"><span data-stu-id="dd42f-239">The following code shows an example:</span></span>

```csharp
var breaker = new CircuitBreaker();

try
{
  breaker.ExecuteAction(() =>
  {
    // Operation protected by the circuit breaker.
    ...
  });
}
catch (CircuitBreakerOpenException ex)
{
  // Perform some different action when the breaker is open.
  // Last exception details are in the inner exception.
  ...
}
catch (Exception ex)
{
  ...
}
```

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="dd42f-240">Útmutató és a kapcsolódó minták</span><span class="sxs-lookup"><span data-stu-id="dd42f-240">Related patterns and guidance</span></span>

<span data-ttu-id="dd42f-241">A következő minták is hasznosak lehetnek ebben a mintában végrehajtása során:</span><span class="sxs-lookup"><span data-stu-id="dd42f-241">The following patterns might also be useful when implementing this pattern:</span></span>

- <span data-ttu-id="dd42f-242">[Ismételje meg a minta][retry-pattern].</span><span class="sxs-lookup"><span data-stu-id="dd42f-242">[Retry Pattern][retry-pattern].</span></span> <span data-ttu-id="dd42f-243">Ismerteti, hogyan alkalmazás is kezelni a várható ideiglenes hibák csatlakozik egy szolgáltatás vagy a hálózati erőforráshoz transzparens módon megpróbálásával korábban sikertelen műveletet.</span><span class="sxs-lookup"><span data-stu-id="dd42f-243">Describes how an application can handle anticipated temporary failures when it tries to connect to a service or network resource by transparently retrying an operation that has previously failed.</span></span>

- <span data-ttu-id="dd42f-244">[Minta figyelési állapotfigyelő végpont](health-endpoint-monitoring.md).</span><span class="sxs-lookup"><span data-stu-id="dd42f-244">[Health Endpoint Monitoring Pattern](health-endpoint-monitoring.md).</span></span> <span data-ttu-id="dd42f-245">Előfordulhat, hogy egy áramköri megszakító tudja egy kérést küld a végpont által a szolgáltatás a szolgáltatás teszteléséhez.</span><span class="sxs-lookup"><span data-stu-id="dd42f-245">A circuit breaker might be able to test the health of a service by sending a request to an endpoint exposed by the service.</span></span> <span data-ttu-id="dd42f-246">A szolgáltatás Megadja, hogy annak állapotát kell visszaadnia.</span><span class="sxs-lookup"><span data-stu-id="dd42f-246">The service should return information indicating its status.</span></span>


[retry-pattern]: ./retry.md
