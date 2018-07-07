---
title: Természetes nyelvi Azure csevegőrobot szállodai foglalások számára
description: Bevált megoldást, amellyel a természetes nyelvi csevegőrobot kereskedelmi alkalmazások az Azure Bot Service, a Cognitive Services és a LUIS, Azure SQL Database és az Application Insights.
author: iainfoulds
ms.date: 07/05/2018
ms.openlocfilehash: 85bdc3194961bbbd8d89db34e5c56e4baa8d8599
ms.sourcegitcommit: 5d99b195388b7cabba383c49a81390ac48f86e8a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 07/06/2018
ms.locfileid: "37891516"
---
# <a name="conversational-azure-chatbot-for-hotel-reservations"></a><span data-ttu-id="4849d-103">Természetes nyelvi Azure csevegőrobot szállodai foglalások számára</span><span class="sxs-lookup"><span data-stu-id="4849d-103">Conversational Azure chatbot for hotel reservations</span></span>

<span data-ttu-id="4849d-104">Ebben a példaforgatókönyvben olyan vállalatok, amelyek a természetes nyelvi csevegőrobot kell integrálható alkalmazásokat alkalmazható.</span><span class="sxs-lookup"><span data-stu-id="4849d-104">This example scenario is applicable to businesses that need integrate a conversational chatbot into applications.</span></span> <span data-ttu-id="4849d-105">Ebben a megoldásban a C# csevegőrobot egy Szálloda láncot, amely lehetővé teszi az ügyfelek számára, hogy ellenőrizze a rendelkezésre állás és a egy webes vagy mobilalkalmazás segítségével könyv vendéglátóipar szolgál.</span><span class="sxs-lookup"><span data-stu-id="4849d-105">In this solution, a C# chatbot is used for a hotel chain that allows customers to check availability and book accommodation through a web or mobile application.</span></span>

<span data-ttu-id="4849d-106">Példaforgatókönyvek közé tartozik, így az ügyfelek számára Szálloda rendelkezésre állás és a könyv termek megtekintése egy étterem elvihető menü tekintse át és helyezze el egy food sorrendben, vagy keresse meg és fényképek nyomatok.</span><span class="sxs-lookup"><span data-stu-id="4849d-106">Example scenarios include providing a way for customers to view hotel availability and book rooms, review a restaurant take-out menu and place a food order, or search for and order prints of photographs.</span></span> <span data-ttu-id="4849d-107">Hagyományosan vállalkozások hire kész, és ezek ügyfél-kérésekre való válaszolás ügyfél szolgáltatási ügynökök kell, és a felhasználónak kell várja meg, hogy egy munkatárs elérhető segítséget nyújt.</span><span class="sxs-lookup"><span data-stu-id="4849d-107">Traditionally, businesses would need to hire and train customer service agents to respond to these customer requests, and customers would have to wait until a representative is available to provide assistance.</span></span>

<span data-ttu-id="4849d-108">Azure-szolgáltatások például a Bot Service és Language Understanding vagy Speech API szolgáltatás segítségével, a vállalatok végrehajtásában, ügyfelek és feldolgozzák a megrendeléseket vagy automatizált és méretezhető robotokat rendelkező foglalásokat.</span><span class="sxs-lookup"><span data-stu-id="4849d-108">By using Azure services such as the Bot Service and Language Understanding or Speech API services, companies can assist customers and process orders or reservations with automated, scalable bots.</span></span>

## <a name="potential-use-cases"></a><span data-ttu-id="4849d-109">A lehetséges alkalmazási helyzetek</span><span class="sxs-lookup"><span data-stu-id="4849d-109">Potential use cases</span></span>

<span data-ttu-id="4849d-110">Fontolja meg a megoldást a következő esetekben használja:</span><span class="sxs-lookup"><span data-stu-id="4849d-110">Consider this solution for the following use cases:</span></span>

* <span data-ttu-id="4849d-111">Éttermek elvihető menüben, és a sorrend food megtekintése</span><span class="sxs-lookup"><span data-stu-id="4849d-111">View restaurant take-out menu and order food</span></span>
* <span data-ttu-id="4849d-112">Szálloda elérhetőségének ellenőrzése és szobát</span><span class="sxs-lookup"><span data-stu-id="4849d-112">Check hotel availability and reserve a room</span></span>
* <span data-ttu-id="4849d-113">Rendelkezésre álló fényképek keresse meg és nyomatrendelés</span><span class="sxs-lookup"><span data-stu-id="4849d-113">Search available photos and order prints</span></span>

## <a name="architecture"></a><span data-ttu-id="4849d-114">Architektúra</span><span class="sxs-lookup"><span data-stu-id="4849d-114">Architecture</span></span>

![Az Azure-összetevőket a természetes nyelvi csevegőrobot részt architektúrájának áttekintése][architecture]

<span data-ttu-id="4849d-116">Ez a megoldás, amely úgy működik, mint a számára egy Szálloda recepciószolgálata természetes nyelvi robotprogramok ismerteti.</span><span class="sxs-lookup"><span data-stu-id="4849d-116">This solution covers a conversational bot that functions as a concierge for a hotel.</span></span> <span data-ttu-id="4849d-117">Az adatok a következőképpen folyamatok a megoldáson keresztül:</span><span class="sxs-lookup"><span data-stu-id="4849d-117">The data flows through the solution as follows:</span></span>

1. <span data-ttu-id="4849d-118">Az ügyfél és a egy mobile csevegőrobot, vagy a web app fér hozzá.</span><span class="sxs-lookup"><span data-stu-id="4849d-118">The customer accesses the chatbot with a mobile or web app.</span></span>
2. <span data-ttu-id="4849d-119">Az Azure Active Directory B2C (üzleti 2 ügyfél), a felhasználó hitelesítését.</span><span class="sxs-lookup"><span data-stu-id="4849d-119">Using Azure Active Directory B2C (Business 2 Customer), the user is authenticated.</span></span>
3. <span data-ttu-id="4849d-120">A Bot Service használata, a felhasználói kérelmek Szálloda rendelkezésre állási kapcsolatos információkat.</span><span class="sxs-lookup"><span data-stu-id="4849d-120">Interacting with the Bot Service, the user requests information about hotel availability.</span></span>
4. <span data-ttu-id="4849d-121">A cognitive Services dolgozza fel a természetes nyelvi kérés tudni, hogy az ügyfél-kommunikációt.</span><span class="sxs-lookup"><span data-stu-id="4849d-121">Cognitive Services processes the natural language request to understand the customer communication.</span></span>
5. <span data-ttu-id="4849d-122">Miután a felhasználó nem elégedett az eredménnyel, a bot hozzáadása, vagy frissíti az ügyfél foglalását egy SQL Database-ben.</span><span class="sxs-lookup"><span data-stu-id="4849d-122">After the user is happy with the results, the bot adds or updates the customer’s reservation in a SQL Database.</span></span>
6. <span data-ttu-id="4849d-123">Az Application Insights futásidejű telemetriai adatokat a fejlesztési és üzemeltetési csapat a robot teljesítményének javítása és használatának érdekében a folyamat során gyűjt.</span><span class="sxs-lookup"><span data-stu-id="4849d-123">Application Insights gathers runtime telemetry throughout the process to help the DevOps team with bot performance and usage.</span></span>

### <a name="components"></a><span data-ttu-id="4849d-124">Összetevők</span><span class="sxs-lookup"><span data-stu-id="4849d-124">Components</span></span>

* <span data-ttu-id="4849d-125">[Az Azure Active Directory] [ aad-docs] van a Microsoft több-bérlős felhőalapú címtár- és identitáskezelési szolgáltatás.</span><span class="sxs-lookup"><span data-stu-id="4849d-125">[Azure Active Directory][aad-docs] is Microsoft’s multi-tenant cloud-based directory and identity management service.</span></span> <span data-ttu-id="4849d-126">Az Azure AD B2C összekötő így azonosíthatja a magánszemélyek külső azonosítókat, például Google, Facebook vagy a Microsoft-Account támogatja.</span><span class="sxs-lookup"><span data-stu-id="4849d-126">Azure AD supports a B2C connector allowing you to identify individuals using external IDs such as Google, Facebook, or a Microsoft Account.</span></span>
* <span data-ttu-id="4849d-127">[App Service-ben] [ appservice-docs] hozhat létre és üzemeltethet webalkalmazásokat az Ön által választott programozási nyelven infrastruktúra kezelése nélkül is.</span><span class="sxs-lookup"><span data-stu-id="4849d-127">[App Service][appservice-docs] enables you to build and host web applications in the programming language of your choice without managing infrastructure.</span></span>
* <span data-ttu-id="4849d-128">[Bot Service] [ botservice-docs] elkészítheti, tesztelheti, telepítheti és felügyelheti a robotokat intelligens eszközöket biztosít.</span><span class="sxs-lookup"><span data-stu-id="4849d-128">[Bot Service][botservice-docs] provides tools to build, test, deploy, and manage intelligent bots.</span></span>
* <span data-ttu-id="4849d-129">[A cognitive Services] [ cognitive-docs] lehetővé teszi, hogy intelligens algoritmusok segítségével jelenik meg, hallgassa meg, beszéljenek, megértéséhez és a felhasználó szükségletek értelmezése a kommunikáció természetes módszereivel keresztül.</span><span class="sxs-lookup"><span data-stu-id="4849d-129">[Cognitive Services][cognitive-docs] lets you use intelligent algorithms to see, hear, speak, understand and interpret your user needs through natural methods of communication.</span></span>
* <span data-ttu-id="4849d-130">[Az SQL Database] [ sqldatabase-docs] van egy teljes körűen felügyelt relációs adatbázis-szolgáltatás, amely az SQL Server-motorhoz.</span><span class="sxs-lookup"><span data-stu-id="4849d-130">[SQL Database][sqldatabase-docs] is a fully managed relational cloud database service that provides SQL Server engine compatibility.</span></span>
* <span data-ttu-id="4849d-131">[Az Application Insights] [ appinsights-docs] egy bővíthető alkalmazásteljesítmény-felügyeleti (APM) szolgáltatás, amely lehetővé teszi az alkalmazások, például a csevegőrobot teljesítményének figyelése.</span><span class="sxs-lookup"><span data-stu-id="4849d-131">[Application Insights][appinsights-docs] is an extensible Application Performance Management (APM) service that lets you monitor the performance of applications, such as your chatbot.</span></span>

### <a name="alternatives"></a><span data-ttu-id="4849d-132">Alternatív megoldások</span><span class="sxs-lookup"><span data-stu-id="4849d-132">Alternatives</span></span>

* <span data-ttu-id="4849d-133">[Microsoft Speech API] [ speech-api] hogyan kommunikáljanak az ügyfelek a robot módosításához is használható.</span><span class="sxs-lookup"><span data-stu-id="4849d-133">[Microsoft Speech API][speech-api] can be used to change how customers interface with your bot.</span></span>
* <span data-ttu-id="4849d-134">[A QnA Maker] [ qna-maker] feltárhatja, hogy gyorsan ismeretek hozzáadása a robot a szolgáltatásban tárolt részben strukturált tartalmat, például egy Gyakori is használható.</span><span class="sxs-lookup"><span data-stu-id="4849d-134">[QnA Maker][qna-maker] can be used as to quickly add knowledge to your bot from semi-structured content like an FAQ.</span></span>
* <span data-ttu-id="4849d-135">[Fordítói szöveg] [ translator] egy szolgáltatás, amely akkor érdemes lehet könnyen többnyelvű támogatás hozzáadása a robot.</span><span class="sxs-lookup"><span data-stu-id="4849d-135">[Translator Text][translator] is a service that you might consider to easily add multi-lingual support to your bot.</span></span>

## <a name="considerations"></a><span data-ttu-id="4849d-136">Megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="4849d-136">Considerations</span></span>

### <a name="availability"></a><span data-ttu-id="4849d-137">Rendelkezésre állás</span><span class="sxs-lookup"><span data-stu-id="4849d-137">Availability</span></span>

<span data-ttu-id="4849d-138">Ez a megoldás az Azure SQL Database ügyfél foglalások tárolásához.</span><span class="sxs-lookup"><span data-stu-id="4849d-138">This solution uses Azure SQL Database for storing customer reservations.</span></span> <span data-ttu-id="4849d-139">Az SQL Database szolgáltatás zóna redundáns adatbázisainak, feladatátvételi csoportok és georeplikáció.</span><span class="sxs-lookup"><span data-stu-id="4849d-139">SQL Database includes zone redundant databases, failover groups, and geo-replication.</span></span> <span data-ttu-id="4849d-140">További információkért lásd: [Azure SQL Database a rendelkezésre állás][sqlavailability-docs].</span><span class="sxs-lookup"><span data-stu-id="4849d-140">For more information, see [Azure SQL Database availability capabilities][sqlavailability-docs].</span></span>

<span data-ttu-id="4849d-141">Méretezhetőség témaköröket talál a [rendelkezésre állási ellenőrzőlista] [ availability] a az Azure Architecture Centert.</span><span class="sxs-lookup"><span data-stu-id="4849d-141">For other scalability topics, see the [availability checklist][availability] in the Azure Architecture Center.</span></span>

### <a name="scalability"></a><span data-ttu-id="4849d-142">Méretezhetőség</span><span class="sxs-lookup"><span data-stu-id="4849d-142">Scalability</span></span>

<span data-ttu-id="4849d-143">A megoldás az Azure App Service-ben.</span><span class="sxs-lookup"><span data-stu-id="4849d-143">This solution uses Azure App Service.</span></span> <span data-ttu-id="4849d-144">Az App Service automatikusan skálázhatja a robot futtató példányok száma.</span><span class="sxs-lookup"><span data-stu-id="4849d-144">With App Service, you can automatically scale the number of instances that run your bot.</span></span> <span data-ttu-id="4849d-145">Ez a funkció lehetővé teszi a webalkalmazás és csevegőrobot kereslet kielégítésére.</span><span class="sxs-lookup"><span data-stu-id="4849d-145">This functionality lets you keep up with customer demand for your web application and chatbot.</span></span> <span data-ttu-id="4849d-146">Az automatikus méretezésben további információkért lásd: [automatikus méretezés ajánlott eljárásai] [ autoscaling] architektúra közepén.</span><span class="sxs-lookup"><span data-stu-id="4849d-146">For more information on autoscale, see [Autoscaling best practices][autoscaling] in the architecture center.</span></span>

<span data-ttu-id="4849d-147">Méretezhetőség témaköröket talál a [méretezési ellenőrzőlista] [ scalability] a az Azure Architecture Centert.</span><span class="sxs-lookup"><span data-stu-id="4849d-147">For other scalability topics, see the [scalability checklist][scalability] in the Azure Architecture Center.</span></span>

### <a name="security"></a><span data-ttu-id="4849d-148">Biztonság</span><span class="sxs-lookup"><span data-stu-id="4849d-148">Security</span></span>

<span data-ttu-id="4849d-149">Ez a megoldás Azure Active Directory B2C-t használ felhasználók hitelesítésére (üzleti 2 fogyasztói).</span><span class="sxs-lookup"><span data-stu-id="4849d-149">This solution uses Azure Active Directory B2C (Business 2 Consumer) to authenticate users.</span></span> <span data-ttu-id="4849d-150">Az AAD B2C-vel a csevegőrobot bármely ügyfél bizalmas fiókadatok vagy a hitelesítő adatok nem tárolja.</span><span class="sxs-lookup"><span data-stu-id="4849d-150">With AAD B2C, your chatbot doesn't store any sensitive customer account information or credentials.</span></span> <span data-ttu-id="4849d-151">További információkért lásd: [Azure Active Directory B2C – áttekintés][aadb2c-docs].</span><span class="sxs-lookup"><span data-stu-id="4849d-151">For more information, see [Azure Active Directory B2C overview][aadb2c-docs].</span></span>

<span data-ttu-id="4849d-152">Az Azure SQL Database-ben tárolt adatok titkosítása transzparens adattitkosítás (TDE).</span><span class="sxs-lookup"><span data-stu-id="4849d-152">Information stored in Azure SQL Database is encrypted at rest with transparent data encryption (TDE).</span></span> <span data-ttu-id="4849d-153">SQL-adatbázis is kínál, amely titkosítja az adatok lekérdezése és feldolgozása során mindig titkosítja.</span><span class="sxs-lookup"><span data-stu-id="4849d-153">SQL Database also offers Always Encrypted which encrypts data during querying and processing.</span></span> <span data-ttu-id="4849d-154">További információ az SQL Database biztonsági: [Azure SQL Database biztonsági és megfelelőségi][sqlsecurity-docs].</span><span class="sxs-lookup"><span data-stu-id="4849d-154">For more information on SQL Database security, see [Azure SQL Database security and compliance][sqlsecurity-docs].</span></span>

<span data-ttu-id="4849d-155">Általános megoldások biztonságos, tekintse át a [Azure Security dokumentációja][security].</span><span class="sxs-lookup"><span data-stu-id="4849d-155">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="4849d-156">Rugalmasság</span><span class="sxs-lookup"><span data-stu-id="4849d-156">Resiliency</span></span>

<span data-ttu-id="4849d-157">Ez a megoldás az Azure SQL Database ügyfél foglalások tárolásához.</span><span class="sxs-lookup"><span data-stu-id="4849d-157">This solution uses Azure SQL Database for storing customer reservations.</span></span> <span data-ttu-id="4849d-158">Az SQL Database szolgáltatás zóna zónaredundáns adatbázisok, a feladatátvételi csoportok, a georeplikáció és az automatikus biztonsági mentést.</span><span class="sxs-lookup"><span data-stu-id="4849d-158">SQL Database includes zone redundant databases, failover groups, geo-replication, and automatic backups.</span></span> <span data-ttu-id="4849d-159">Ezek a funkciók lehetővé teszik az alkalmazás egy karbantartási esemény vagy szolgáltatáskimaradás esetén is működőképes marad.</span><span class="sxs-lookup"><span data-stu-id="4849d-159">These features allow your application to continue running in the event of a maintenance event or outage.</span></span> <span data-ttu-id="4849d-160">További információkért lásd: [Azure SQL Database a rendelkezésre állás][sqlavailability-docs].</span><span class="sxs-lookup"><span data-stu-id="4849d-160">For more information, see [Azure SQL Database availability capabilities][sqlavailability-docs].</span></span>

<span data-ttu-id="4849d-161">Az alkalmazás állapotának figyeléséhez, ez a megoldás az Application Insights.</span><span class="sxs-lookup"><span data-stu-id="4849d-161">To monitor the health of your application, this solution uses Application Insights.</span></span> <span data-ttu-id="4849d-162">Az Application Insights riasztást, és reagálni, amely hatással lenne, a felhasználói élmény és a csevegőrobot rendelkezésre állása teljesítménybeli problémák.</span><span class="sxs-lookup"><span data-stu-id="4849d-162">With Application Insights, you can generate alerts and respond to performance issues that would impact the customer experience and availability of the chatbot.</span></span> <span data-ttu-id="4849d-163">További információkért lásd: [Mi az Application Insights?][appinsights-docs]</span><span class="sxs-lookup"><span data-stu-id="4849d-163">For more information, see [What is Application Insights?][appinsights-docs]</span></span>

<span data-ttu-id="4849d-164">Rugalmas megoldások tervezésével kapcsolatos általános útmutatásért lásd: [rugalmas alkalmazások tervezése az Azure][resiliency].</span><span class="sxs-lookup"><span data-stu-id="4849d-164">For general guidance on designing resilient solutions, see [Designing resilient applications for Azure][resiliency].</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="4849d-165">A megoldás üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="4849d-165">Deploy the solution</span></span>

<span data-ttu-id="4849d-166">Ez a megoldás segítségével megismerheti a legtöbb összpontosított területek három összetevőből áll:</span><span class="sxs-lookup"><span data-stu-id="4849d-166">This solution is divided into three components for you to explore areas that you are most focused on:</span></span>

* <span data-ttu-id="4849d-167">[Infrastruktúra-összetevőket](#deploy-infrastructure-components).</span><span class="sxs-lookup"><span data-stu-id="4849d-167">[Infrastructure components](#deploy-infrastructure-components).</span></span> <span data-ttu-id="4849d-168">Egy Azure Resource Manager-sablon használatával helyezhet üzembe egy App Service, webalkalmazás, az Application Insights, tárfiók, és az SQL Server és adatbázis alapvető infrastruktúra összetevők.</span><span class="sxs-lookup"><span data-stu-id="4849d-168">Use an Azure Resource Manger template to deploy the core infrastructure components of an App Service, Web App, Application Insights, Storage account, and SQL Server and database.</span></span>
* <span data-ttu-id="4849d-169">[Webes alkalmazás Csevegőrobot](#deploy-web-app-chatbot).</span><span class="sxs-lookup"><span data-stu-id="4849d-169">[Web App Chatbot](#deploy-web-app-chatbot).</span></span> <span data-ttu-id="4849d-170">A robot a Bot Service és Language Understanding és Intelligent Services (LUIS) alkalmazás üzembe helyezéséhez az Azure CLI használatával.</span><span class="sxs-lookup"><span data-stu-id="4849d-170">Use the Azure CLI to deploy a bot with the Bot Service and Language Understanding and Intelligent Services (LUIS) app.</span></span>
* <span data-ttu-id="4849d-171">[Mintaalkalmazás a C# csevegőrobot](#deploy-chatbot-c-application-code).</span><span class="sxs-lookup"><span data-stu-id="4849d-171">[Sample C# chatbot application](#deploy-chatbot-c-application-code).</span></span> <span data-ttu-id="4849d-172">Tekintse át a minta Szálloda foglalás C# alkalmazás kódja és a egy robot, az Azure-ban való üzembe helyezése a Visual Studio használatával.</span><span class="sxs-lookup"><span data-stu-id="4849d-172">Use Visual Studio to review the sample hotel reservation C# application code and deploy to a bot in Azure.</span></span>

<span data-ttu-id="4849d-173">**Előfeltételek.**</span><span class="sxs-lookup"><span data-stu-id="4849d-173">**Prerequisites.**</span></span> <span data-ttu-id="4849d-174">Meglévő Azure-fiókkal kell rendelkeznie.</span><span class="sxs-lookup"><span data-stu-id="4849d-174">You must have an existing Azure account.</span></span> <span data-ttu-id="4849d-175">Ha nem rendelkezik Azure-előfizetéssel, mindössze néhány perc alatt létrehozhat egy [ingyenes fiókot](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) a virtuális gép létrehozásának megkezdése előtt.</span><span class="sxs-lookup"><span data-stu-id="4849d-175">If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.</span></span>

### <a name="deploy-infrastructure-components"></a><span data-ttu-id="4849d-176">Infrastruktúra-összetevők telepítése</span><span class="sxs-lookup"><span data-stu-id="4849d-176">Deploy infrastructure components</span></span>

<span data-ttu-id="4849d-177">Az infrastruktúra-összetevőket az Azure Resource Manager-sablon üzembe helyezéséhez hajtsa végre az alábbi lépéseket.</span><span class="sxs-lookup"><span data-stu-id="4849d-177">To deploy the infrastructure components with an Azure Resource Manager template, perform the following steps.</span></span>

1. <span data-ttu-id="4849d-178">Kattintson a **üzembe helyezés az Azure** gombra:</span><span class="sxs-lookup"><span data-stu-id="4849d-178">Click the **Deploy to Azure** button:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fcommerce-chatbot.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. <span data-ttu-id="4849d-179">Várja meg a sablon üzembe helyezéséhez az Azure Portal megnyitásához, majd kövesse az alábbi lépéseket:</span><span class="sxs-lookup"><span data-stu-id="4849d-179">Wait for the template deployment to open in the Azure portal, then complete the following steps:</span></span>
   * <span data-ttu-id="4849d-180">Válassza ki a **új létrehozása** erőforrás csoportra, majd adjon meg egy nevet például *myCommerceChatBotInfrastructure* a szövegmezőben.</span><span class="sxs-lookup"><span data-stu-id="4849d-180">Choose to **Create new** resource group, then provide a name such as *myCommerceChatBotInfrastructure* in the text box.</span></span>
   * <span data-ttu-id="4849d-181">Válassza ki a régiót, a **hely** legördülő listából.</span><span class="sxs-lookup"><span data-stu-id="4849d-181">Select a region from the **Location** drop-down box.</span></span>
   * <span data-ttu-id="4849d-182">Adjon meg egy felhasználónevet és a biztonságos jelszó az SQL Server-rendszergazdai fiók.</span><span class="sxs-lookup"><span data-stu-id="4849d-182">Provide a username and secure password for the SQL Server administrator account.</span></span>
   * <span data-ttu-id="4849d-183">Tekintse át a használati feltételeket, majd ellenőrizze **elfogadom a feltételeket és a fenti feltételeket**.</span><span class="sxs-lookup"><span data-stu-id="4849d-183">Review the terms and conditions, then check **I agree to the terms and conditions stated above**.</span></span>
   * <span data-ttu-id="4849d-184">Válassza ki a **beszerzési** gombra.</span><span class="sxs-lookup"><span data-stu-id="4849d-184">Select the **Purchase** button.</span></span>

<span data-ttu-id="4849d-185">A központi telepítés befejezése néhány percet vesz igénybe.</span><span class="sxs-lookup"><span data-stu-id="4849d-185">It takes a few minutes for the deployment to complete.</span></span>

### <a name="deploy-web-app-chatbot"></a><span data-ttu-id="4849d-186">Webalkalmazás csevegőrobot üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="4849d-186">Deploy Web App chatbot</span></span>

<span data-ttu-id="4849d-187">Az Azure CLI segítségével a csevegőrobot hozhat létre.</span><span class="sxs-lookup"><span data-stu-id="4849d-187">To create the chatbot, use the Azure CLI.</span></span> <span data-ttu-id="4849d-188">A következő példa telepíti a CLI-bővítményt a Bot Service, létrehoz egy erőforráscsoportot, majd üzembe helyezi a robot, az Application Insights.</span><span class="sxs-lookup"><span data-stu-id="4849d-188">The following example installs the CLI extension for Bot Service, creates a resource group, then deploys a bot that uses Application Insights.</span></span> <span data-ttu-id="4849d-189">Amikor a rendszer kéri, Microsoft-fiókja hitelesítéséhez, és lehetővé téve a robot regisztrálja magát a Bot Service és Language Understanding és Intelligent Services (LUIS) alkalmazást.</span><span class="sxs-lookup"><span data-stu-id="4849d-189">When prompted, authenticate your Microsoft account and allow the bot to register itself with the Bot Service and Language Understanding and Intelligent Services (LUIS) app.</span></span>

```azurecli-interactive
# Install the Azure CLI extension for the Bot Service
az extension add --name botservice --yes

# Create a resource group
az group create --name myCommerceChatbot --location eastus

# Create a Web App Chatbot that uses Application Insights
az bot create \
    --resource-group myCommerceChatbot \
    --name commerceChatbot \
    --location eastus \
    --kind webapp \
    --sku S1 \
    --insights eastus
```

### <a name="deploy-chatbot-c-application-code"></a><span data-ttu-id="4849d-190">Csevegőrobot C# alkalmazás kód üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="4849d-190">Deploy chatbot C# application code</span></span>

<span data-ttu-id="4849d-191">A minta C#-alkalmazás a Githubon érhető el:</span><span class="sxs-lookup"><span data-stu-id="4849d-191">A sample C# application is available on GitHub:</span></span> 

* [<span data-ttu-id="4849d-192">Kereskedelmi Bot C#-minta</span><span class="sxs-lookup"><span data-stu-id="4849d-192">Commerce Bot C# sample</span></span>](https://github.com/Microsoft/AzureBotServices-scenarios/tree/master/CSharp/Commerce/src)

<span data-ttu-id="4849d-193">A mintaalkalmazás az Azure Active Directory authentication összetevői és a Language Understanding és Intelligent Services (LUIS) összetevő integráció a Cognitive Services.</span><span class="sxs-lookup"><span data-stu-id="4849d-193">The sample application includes the Azure Active Directory authentication components and integration with the Language Understanding and Intelligent Services (LUIS) component of Cognitive Services.</span></span> <span data-ttu-id="4849d-194">Az alkalmazás létrehozása és a megoldás üzembe helyezése a Visual Studio használatához.</span><span class="sxs-lookup"><span data-stu-id="4849d-194">The application requires Visual Studio to build and deploy the solution.</span></span> <span data-ttu-id="4849d-195">További tájékoztatást az AAD B2C-vel és a LUIS-alkalmazás konfigurálása a GitHub-tárház dokumentációjában található.</span><span class="sxs-lookup"><span data-stu-id="4849d-195">Additional information on configuring AAD B2C and the LUIS app can be found in the GitHub repo documentation.</span></span>

## <a name="pricing"></a><span data-ttu-id="4849d-196">Díjszabás</span><span class="sxs-lookup"><span data-stu-id="4849d-196">Pricing</span></span>

<span data-ttu-id="4849d-197">Ez a megoldás költségének megismeréséhez, a szolgáltatások mindegyike a költségkalkulátor az előre konfigurált.</span><span class="sxs-lookup"><span data-stu-id="4849d-197">To explore the cost of running this solution, all of the services are pre-configured in the cost calculator.</span></span> <span data-ttu-id="4849d-198">Tekintse meg, hogyan díjszabását szeretné módosítani az adott használati esetekhez, módosítsa a megfelelő változókat egyezik a várt forgalomhoz.</span><span class="sxs-lookup"><span data-stu-id="4849d-198">To see how the pricing would change for your particular use case, change the appropriate variables to match your expected traffic.</span></span>

<span data-ttu-id="4849d-199">Adtunk meg három példa költség profilok feldolgozni a csevegőrobot várt üzenetek mennyisége alapján:</span><span class="sxs-lookup"><span data-stu-id="4849d-199">We have provided three sample cost profiles based on the amount of messages you expect your chatbot to process:</span></span>

* <span data-ttu-id="4849d-200">[Kis][small-pricing]: Ez havonta < 10 000 üzenetek feldolgozására utal.</span><span class="sxs-lookup"><span data-stu-id="4849d-200">[Small][small-pricing]: this correlates to processing < 10,000 messages per month.</span></span>
* <span data-ttu-id="4849d-201">[Közepes][medium-pricing]: Ez havi 500 000 < üzenetek feldolgozására utal.</span><span class="sxs-lookup"><span data-stu-id="4849d-201">[Medium][medium-pricing]: this correlates to processing < 500,000 messages per month.</span></span>
* <span data-ttu-id="4849d-202">[Nagy][large-pricing]: Ez havonta < 10 millió üzenetek feldolgozására utal.</span><span class="sxs-lookup"><span data-stu-id="4849d-202">[Large][large-pricing]: this correlates to processing < 10 million messages per month.</span></span>

## <a name="related-resources"></a><span data-ttu-id="4849d-203">Kapcsolódó erőforrások</span><span class="sxs-lookup"><span data-stu-id="4849d-203">Related Resources</span></span>

<span data-ttu-id="4849d-204">Az interaktív oktatóanyagok az Azure Bot Service kihasználva készletének, tekintse meg a [oktatóanyag csomópont] [ botservice-docs] dokumentáció.</span><span class="sxs-lookup"><span data-stu-id="4849d-204">For a set of guided tutorials on leveraging the Azure Bot Service, see the [tutorial node][botservice-docs] of the documentation.</span></span>

<!-- links -->
[aadb2c-docs]: /azure/active-directory-b2c/active-directory-b2c-overview
[aad-docs]: /azure/active-directory/
[appinsights-docs]: /azure/application-insights/app-insights-overview
[appservice-docs]: /azure/app-service/
[architecture]: ./media/commerce-chatbot/architecture-commerce-chatbot.png
[autoscaling]: ../../best-practices/auto-scaling.md
[availability]: ../../checklist/availability.md
[botservice-docs]: /azure/bot-service/
[cognitive-docs]: /azure/cognitive-services/
[resiliency]: ../../resiliency/index.md
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[security]: /azure/security/
[scalability]: ../../checklist/scalability.md
[sqlavailability-docs]: /azure/sql-database/sql-database-technical-overview#availability-capabilities
[sqldatabase-docs]: /azure/sql-database/
[sqlsecurity-docs]: /azure/sql-database/sql-database-technical-overview#advanced-security-and-compliance
[qna-maker]: /azure/cognitive-services/QnAMaker/Overview/overview
[speech-api]: /azure/cognitive-services/speech/home
[translator]: /azure/cognitive-services/translator/translator-info-overview

[small-pricing]: https://azure.com/e/dce05b6184904c50b38e1a8654f726b6
[medium-pricing]: https://azure.com/e/304d17106afc480dbc414f9726078a03
[large-pricing]: https://azure.com/e/8319dd5e5e3d4f118f9029e32a80e887