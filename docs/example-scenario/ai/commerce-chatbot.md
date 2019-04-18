---
title: Beszélgető csevegőrobot szállodai foglalásokhoz
titleSuffix: Azure Example Scenarios
description: Beszélgető csevegőrobotot hozhat létre kereskedelmi alkalmazásokhoz az Azure Bot Service segítségével.
author: iainfoulds
ms.date: 07/05/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
social_image_url: /azure/architecture/example-scenario/ai/media/architecture-commerce-chatbot.png
ms.openlocfilehash: c4859cb0e43603991e4f8e6a0311a28537f29f1a
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/17/2019
ms.locfileid: "59640260"
---
# <a name="conversational-chatbot-for-hotel-reservations-on-azure"></a><span data-ttu-id="4e092-103">Beszélgető csevegőrobot szállodai foglalásokhoz az Azure-ban</span><span class="sxs-lookup"><span data-stu-id="4e092-103">Conversational chatbot for hotel reservations on Azure</span></span>

<span data-ttu-id="4e092-104">Ebben a példaforgatókönyvben olyan vállalatok, amelyek a természetes nyelvi csevegőrobot beépíthetik kell alkalmazható.</span><span class="sxs-lookup"><span data-stu-id="4e092-104">This example scenario is applicable to businesses that need to integrate a conversational chatbot into applications.</span></span> <span data-ttu-id="4e092-105">Ebben a forgatókönyvben egy C# csevegőrobot egy Szálloda láncot, amely lehetővé teszi az ügyfelek számára, hogy ellenőrizze a rendelkezésre állás és a egy webes vagy mobilalkalmazás segítségével könyv vendéglátóipar szolgál.</span><span class="sxs-lookup"><span data-stu-id="4e092-105">In this scenario, a C# chatbot is used for a hotel chain that allows customers to check availability and book accommodation through a web or mobile application.</span></span>

<span data-ttu-id="4e092-106">A lehetséges használati módjai többek között, így az ügyfelek számára Szálloda rendelkezésre állás és a könyv termek megtekintése egy étterem elvihető menü tekintse át és helyezze el egy food sorrendben, vagy keresse meg és fényképek nyomatok.</span><span class="sxs-lookup"><span data-stu-id="4e092-106">Potential uses include providing a way for customers to view hotel availability and book rooms, review a restaurant take-out menu and place a food order, or search for and order prints of photographs.</span></span> <span data-ttu-id="4e092-107">Hagyományosan vállalkozások hire kész, és ezek ügyfél-kérésekre való válaszolás ügyfél szolgáltatási ügynökök kell, és a felhasználónak kell várja meg, hogy egy munkatárs elérhető segítséget nyújt.</span><span class="sxs-lookup"><span data-stu-id="4e092-107">Traditionally, businesses would need to hire and train customer service agents to respond to these customer requests, and customers would have to wait until a representative is available to provide assistance.</span></span>

<span data-ttu-id="4e092-108">Azure-szolgáltatások például a Bot Service és Language Understanding vagy Speech API szolgáltatás segítségével, a vállalatok végrehajtásában, ügyfelek és feldolgozzák a megrendeléseket vagy automatizált és méretezhető robotokat rendelkező foglalásokat.</span><span class="sxs-lookup"><span data-stu-id="4e092-108">By using Azure services such as the Bot Service and Language Understanding or Speech API services, companies can assist customers and process orders or reservations with automated, scalable bots.</span></span>

## <a name="relevant-use-cases"></a><span data-ttu-id="4e092-109">Alkalmazási helyzetek</span><span class="sxs-lookup"><span data-stu-id="4e092-109">Relevant use cases</span></span>

<span data-ttu-id="4e092-110">Egyéb alkalmazási helyzetek a következők:</span><span class="sxs-lookup"><span data-stu-id="4e092-110">Other relevant use cases include:</span></span>

- <span data-ttu-id="4e092-111">Egy étterem elvihető menü megtekintése és élelmiszer rendezése</span><span class="sxs-lookup"><span data-stu-id="4e092-111">Viewing a restaurant take-out menu and ordering food</span></span>
- <span data-ttu-id="4e092-112">Szálloda rendelkezésre állásának ellenőrzése és a egy hely foglalása</span><span class="sxs-lookup"><span data-stu-id="4e092-112">Checking hotel availability and reserving a room</span></span>
- <span data-ttu-id="4e092-113">Keresés a rendelkezésre álló fényképek, és rendelése</span><span class="sxs-lookup"><span data-stu-id="4e092-113">Searching available photos and ordering prints</span></span>

## <a name="architecture"></a><span data-ttu-id="4e092-114">Architektúra</span><span class="sxs-lookup"><span data-stu-id="4e092-114">Architecture</span></span>

![Az Azure-összetevőket a természetes nyelvi csevegőrobot részt architektúrájának áttekintése][architecture]

<span data-ttu-id="4e092-116">Ebben a forgatókönyvben a természetes nyelvi robot, amely úgy működik, mint a számára egy Szálloda recepciószolgálata ismerteti.</span><span class="sxs-lookup"><span data-stu-id="4e092-116">This scenario covers a conversational bot that functions as a concierge for a hotel.</span></span> <span data-ttu-id="4e092-117">A áramlanak keresztül az adatok a forgatókönyv a következő:</span><span class="sxs-lookup"><span data-stu-id="4e092-117">The data flows through the scenario as follows:</span></span>

1. <span data-ttu-id="4e092-118">Az ügyfél és a egy mobile csevegőrobot, vagy a web app fér hozzá.</span><span class="sxs-lookup"><span data-stu-id="4e092-118">The customer accesses the chatbot with a mobile or web app.</span></span>
2. <span data-ttu-id="4e092-119">Az Azure Active Directory B2C (üzleti 2 ügyfél), a felhasználó hitelesítését.</span><span class="sxs-lookup"><span data-stu-id="4e092-119">Using Azure Active Directory B2C (Business 2 Customer), the user is authenticated.</span></span>
3. <span data-ttu-id="4e092-120">A Bot Service használata, a felhasználói kérelmek Szálloda rendelkezésre állási kapcsolatos információkat.</span><span class="sxs-lookup"><span data-stu-id="4e092-120">Interacting with the Bot Service, the user requests information about hotel availability.</span></span>
4. <span data-ttu-id="4e092-121">A cognitive Services dolgozza fel a természetes nyelvi kérés tudni, hogy az ügyfél-kommunikációt.</span><span class="sxs-lookup"><span data-stu-id="4e092-121">Cognitive Services processes the natural language request to understand the customer communication.</span></span>
5. <span data-ttu-id="4e092-122">Miután a felhasználó nem elégedett az eredménnyel, a bot hozzáadása, vagy frissíti az ügyfél foglalását egy SQL Database-ben.</span><span class="sxs-lookup"><span data-stu-id="4e092-122">After the user is happy with the results, the bot adds or updates the customer’s reservation in a SQL Database.</span></span>
6. <span data-ttu-id="4e092-123">Az Application Insights futásidejű telemetriai adatokat a fejlesztési és üzemeltetési csapat a robot teljesítményének javítása és használatának érdekében a folyamat során gyűjt.</span><span class="sxs-lookup"><span data-stu-id="4e092-123">Application Insights gathers runtime telemetry throughout the process to help the DevOps team with bot performance and usage.</span></span>

### <a name="components"></a><span data-ttu-id="4e092-124">Összetevők</span><span class="sxs-lookup"><span data-stu-id="4e092-124">Components</span></span>

- <span data-ttu-id="4e092-125">[Az Azure Active Directory] [ aad-docs] van a Microsoft több-bérlős felhőalapú címtár- és identitáskezelési szolgáltatás.</span><span class="sxs-lookup"><span data-stu-id="4e092-125">[Azure Active Directory][aad-docs] is Microsoft’s multi-tenant cloud-based directory and identity management service.</span></span> <span data-ttu-id="4e092-126">Az Azure AD B2C összekötő így azonosíthatja a magánszemélyek külső azonosítókat, például Google, Facebook vagy a Microsoft-Account támogatja.</span><span class="sxs-lookup"><span data-stu-id="4e092-126">Azure AD supports a B2C connector allowing you to identify individuals using external IDs such as Google, Facebook, or a Microsoft Account.</span></span>
- <span data-ttu-id="4e092-127">[App Service-ben] [ appservice-docs] hozhat létre és üzemeltethet webalkalmazásokat az Ön által választott programozási nyelven infrastruktúra kezelése nélkül is.</span><span class="sxs-lookup"><span data-stu-id="4e092-127">[App Service][appservice-docs] enables you to build and host web applications in the programming language of your choice without managing infrastructure.</span></span>
- <span data-ttu-id="4e092-128">[Bot Service] [ botservice-docs] elkészítheti, tesztelheti, telepítheti és felügyelheti a robotokat intelligens eszközöket biztosít.</span><span class="sxs-lookup"><span data-stu-id="4e092-128">[Bot Service][botservice-docs] provides tools to build, test, deploy, and manage intelligent bots.</span></span>
- <span data-ttu-id="4e092-129">[A cognitive Services] [ cognitive-docs] lehetővé teszi, hogy intelligens algoritmusok segítségével jelenik meg, hallgassa meg beszél, megértésében, valamint a felhasználói szükségletek értelmezése a kommunikáció természetes módszereivel keresztül.</span><span class="sxs-lookup"><span data-stu-id="4e092-129">[Cognitive Services][cognitive-docs] lets you use intelligent algorithms to see, hear, speak, understand, and interpret your user needs through natural methods of communication.</span></span>
- <span data-ttu-id="4e092-130">[Az SQL Database] [ sqldatabase-docs] van egy teljes körűen felügyelt relációs adatbázis-szolgáltatás, amely az SQL Server-motorhoz.</span><span class="sxs-lookup"><span data-stu-id="4e092-130">[SQL Database][sqldatabase-docs] is a fully managed relational cloud database service that provides SQL Server engine compatibility.</span></span>
- <span data-ttu-id="4e092-131">[Az Application Insights] [ appinsights-docs] egy bővíthető alkalmazásteljesítmény-felügyeleti (APM) szolgáltatás, amely lehetővé teszi az alkalmazások, például a csevegőrobot teljesítményének figyelése.</span><span class="sxs-lookup"><span data-stu-id="4e092-131">[Application Insights][appinsights-docs] is an extensible Application Performance Management (APM) service that lets you monitor the performance of applications, such as your chatbot.</span></span>

### <a name="alternatives"></a><span data-ttu-id="4e092-132">Alternatív megoldások</span><span class="sxs-lookup"><span data-stu-id="4e092-132">Alternatives</span></span>

- <span data-ttu-id="4e092-133">[Microsoft Speech API] [ speech-api] hogyan kommunikáljanak az ügyfelek a robot módosításához is használható.</span><span class="sxs-lookup"><span data-stu-id="4e092-133">[Microsoft Speech API][speech-api] can be used to change how customers interface with your bot.</span></span>
- <span data-ttu-id="4e092-134">[A QnA Maker] [ qna-maker] feltárhatja, hogy gyorsan ismeretek hozzáadása a robot a szolgáltatásban tárolt részben strukturált tartalmat, például egy Gyakori is használható.</span><span class="sxs-lookup"><span data-stu-id="4e092-134">[QnA Maker][qna-maker] can be used as to quickly add knowledge to your bot from semi-structured content like an FAQ.</span></span>
- <span data-ttu-id="4e092-135">[Fordítói szöveg] [ translator] egy szolgáltatás, amely akkor érdemes lehet könnyen többnyelvű támogatás hozzáadása a robot.</span><span class="sxs-lookup"><span data-stu-id="4e092-135">[Translator Text][translator] is a service that you might consider to easily add multi-lingual support to your bot.</span></span>

## <a name="considerations"></a><span data-ttu-id="4e092-136">Megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="4e092-136">Considerations</span></span>

### <a name="availability"></a><span data-ttu-id="4e092-137">Rendelkezésre állás</span><span class="sxs-lookup"><span data-stu-id="4e092-137">Availability</span></span>

<span data-ttu-id="4e092-138">Ebben a forgatókönyvben az Azure SQL Database ügyfél foglalások tárolásához.</span><span class="sxs-lookup"><span data-stu-id="4e092-138">This scenario uses Azure SQL Database for storing customer reservations.</span></span> <span data-ttu-id="4e092-139">Az SQL Database szolgáltatás zóna redundáns adatbázisainak, feladatátvételi csoportok és georeplikáció.</span><span class="sxs-lookup"><span data-stu-id="4e092-139">SQL Database includes zone redundant databases, failover groups, and geo-replication.</span></span> <span data-ttu-id="4e092-140">További információkért lásd: [Azure SQL Database a rendelkezésre állás][sqlavailability-docs].</span><span class="sxs-lookup"><span data-stu-id="4e092-140">For more information, see [Azure SQL Database availability capabilities][sqlavailability-docs].</span></span>

### <a name="scalability"></a><span data-ttu-id="4e092-141">Méretezhetőség</span><span class="sxs-lookup"><span data-stu-id="4e092-141">Scalability</span></span>

<span data-ttu-id="4e092-142">Ebben a példában az Azure App Service-ben.</span><span class="sxs-lookup"><span data-stu-id="4e092-142">This scenario uses Azure App Service.</span></span> <span data-ttu-id="4e092-143">Az App Service automatikusan skálázhatja a robot futtató példányok száma.</span><span class="sxs-lookup"><span data-stu-id="4e092-143">With App Service, you can automatically scale the number of instances that run your bot.</span></span> <span data-ttu-id="4e092-144">Ez a funkció lehetővé teszi a webalkalmazás és csevegőrobot kereslet kielégítésére.</span><span class="sxs-lookup"><span data-stu-id="4e092-144">This functionality lets you keep up with customer demand for your web application and chatbot.</span></span> <span data-ttu-id="4e092-145">További információ az automatikus méretezésben: [automatikus méretezés ajánlott eljárásai] [ autoscaling] a az Azure Architecture Centert.</span><span class="sxs-lookup"><span data-stu-id="4e092-145">For more information on autoscale, see [Autoscaling best practices][autoscaling] in the Azure Architecture Center.</span></span>

<span data-ttu-id="4e092-146">Méretezhetőség témaköröket talál a [méretezési ellenőrzőlista] [ scalability] a az Azure Architecture Centert.</span><span class="sxs-lookup"><span data-stu-id="4e092-146">For other scalability topics, see the [scalability checklist][scalability] in the Azure Architecture Center.</span></span>

### <a name="security"></a><span data-ttu-id="4e092-147">Biztonság</span><span class="sxs-lookup"><span data-stu-id="4e092-147">Security</span></span>

<span data-ttu-id="4e092-148">Ez a forgatókönyv Azure Active Directory B2C-t használ felhasználók hitelesítésére (üzleti 2 fogyasztói).</span><span class="sxs-lookup"><span data-stu-id="4e092-148">This scenario uses Azure Active Directory B2C (Business 2 Consumer) to authenticate users.</span></span> <span data-ttu-id="4e092-149">Az AAD B2C-vel a csevegőrobot bármely ügyfél bizalmas fiókadatok vagy a hitelesítő adatok nem tárolja.</span><span class="sxs-lookup"><span data-stu-id="4e092-149">With AAD B2C, your chatbot doesn't store any sensitive customer account information or credentials.</span></span> <span data-ttu-id="4e092-150">További információkért lásd: [Azure Active Directory B2C – áttekintés][aadb2c-docs].</span><span class="sxs-lookup"><span data-stu-id="4e092-150">For more information, see [Azure Active Directory B2C overview][aadb2c-docs].</span></span>

<span data-ttu-id="4e092-151">Az Azure SQL Database-ben tárolt adatok titkosítása transzparens adattitkosítás (TDE).</span><span class="sxs-lookup"><span data-stu-id="4e092-151">Information stored in Azure SQL Database is encrypted at rest with transparent data encryption (TDE).</span></span> <span data-ttu-id="4e092-152">SQL-adatbázis is kínál, amely titkosítja az adatok lekérdezése és feldolgozása során mindig titkosítja.</span><span class="sxs-lookup"><span data-stu-id="4e092-152">SQL Database also offers Always Encrypted which encrypts data during querying and processing.</span></span> <span data-ttu-id="4e092-153">További információ az SQL Database biztonsági: [Azure SQL Database biztonsági és megfelelőségi][sqlsecurity-docs].</span><span class="sxs-lookup"><span data-stu-id="4e092-153">For more information on SQL Database security, see [Azure SQL Database security and compliance][sqlsecurity-docs].</span></span>

<span data-ttu-id="4e092-154">Általános megoldások biztonságos, tekintse át a [Azure Security dokumentációja][security].</span><span class="sxs-lookup"><span data-stu-id="4e092-154">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="4e092-155">Rugalmasság</span><span class="sxs-lookup"><span data-stu-id="4e092-155">Resiliency</span></span>

<span data-ttu-id="4e092-156">Ebben a forgatókönyvben az Azure SQL Database ügyfél foglalások tárolásához.</span><span class="sxs-lookup"><span data-stu-id="4e092-156">This scenario uses Azure SQL Database for storing customer reservations.</span></span> <span data-ttu-id="4e092-157">Az SQL Database szolgáltatás zóna zónaredundáns adatbázisok, a feladatátvételi csoportok, a georeplikáció és az automatikus biztonsági mentést.</span><span class="sxs-lookup"><span data-stu-id="4e092-157">SQL Database includes zone redundant databases, failover groups, geo-replication, and automatic backups.</span></span> <span data-ttu-id="4e092-158">Ezek a funkciók lehetővé teszik az alkalmazás marad, ha van egy karbantartási esemény, illetve leállás.</span><span class="sxs-lookup"><span data-stu-id="4e092-158">These features allow your application to continue running if there is a maintenance event or outage.</span></span> <span data-ttu-id="4e092-159">További információkért lásd: [Azure SQL Database a rendelkezésre állás][sqlavailability-docs].</span><span class="sxs-lookup"><span data-stu-id="4e092-159">For more information, see [Azure SQL Database availability capabilities][sqlavailability-docs].</span></span>

<span data-ttu-id="4e092-160">Az alkalmazás állapotának figyelése, az ebben a forgatókönyvben az Application Insights.</span><span class="sxs-lookup"><span data-stu-id="4e092-160">To monitor the health of your application, this scenario uses Application Insights.</span></span> <span data-ttu-id="4e092-161">Az Application Insights riasztást, és reagálni, amely hatással lenne, a felhasználói élmény és a csevegőrobot rendelkezésre állása teljesítménybeli problémák.</span><span class="sxs-lookup"><span data-stu-id="4e092-161">With Application Insights, you can generate alerts and respond to performance issues that would impact the customer experience and availability of the chatbot.</span></span> <span data-ttu-id="4e092-162">További információkért lásd: [Mi az Application Insights?][appinsights-docs]</span><span class="sxs-lookup"><span data-stu-id="4e092-162">For more information, see [What is Application Insights?][appinsights-docs]</span></span>

<span data-ttu-id="4e092-163">Rugalmasság témaköröket talál [megbízható Azure-alkalmazások tervezése](../../reliability/index.md).</span><span class="sxs-lookup"><span data-stu-id="4e092-163">For other resiliency topics, see [Designing reliable Azure applications](../../reliability/index.md).</span></span>

## <a name="deploy-the-scenario"></a><span data-ttu-id="4e092-164">A forgatókönyv megvalósításához</span><span class="sxs-lookup"><span data-stu-id="4e092-164">Deploy the scenario</span></span>

<span data-ttu-id="4e092-165">Ez a forgatókönyv segítségével megismerheti a legtöbb összpontosított területek három összetevőből áll:</span><span class="sxs-lookup"><span data-stu-id="4e092-165">This scenario is divided into three components for you to explore areas that you are most focused on:</span></span>

- <span data-ttu-id="4e092-166">[Infrastruktúra-összetevőket](#walk-through).</span><span class="sxs-lookup"><span data-stu-id="4e092-166">[Infrastructure components](#walk-through).</span></span> <span data-ttu-id="4e092-167">Egy Azure Resource Manager-sablon használatával helyezhet üzembe egy App Service, webalkalmazás, az Application Insights, tárfiók, és az SQL Server és adatbázis alapvető infrastruktúra összetevők.</span><span class="sxs-lookup"><span data-stu-id="4e092-167">Use an Azure Resource Manger template to deploy the core infrastructure components of an App Service, Web App, Application Insights, Storage account, and SQL Server and database.</span></span>
- <span data-ttu-id="4e092-168">[Webes alkalmazás Csevegőrobot](#deploy-web-app-chatbot).</span><span class="sxs-lookup"><span data-stu-id="4e092-168">[Web App Chatbot](#deploy-web-app-chatbot).</span></span> <span data-ttu-id="4e092-169">A robot a Bot Service és Language Understanding és Intelligent Services (LUIS) alkalmazás üzembe helyezéséhez az Azure CLI használatával.</span><span class="sxs-lookup"><span data-stu-id="4e092-169">Use the Azure CLI to deploy a bot with the Bot Service and Language Understanding and Intelligent Services (LUIS) app.</span></span>
- <span data-ttu-id="4e092-170">[Mintaalkalmazás a C# csevegőrobot](#deploy-chatbot-c-application-code).</span><span class="sxs-lookup"><span data-stu-id="4e092-170">[Sample C# chatbot application](#deploy-chatbot-c-application-code).</span></span> <span data-ttu-id="4e092-171">Tekintse át a minta Szálloda foglalás C# alkalmazás kódja és a egy robot, az Azure-ban való üzembe helyezése a Visual Studio használatával.</span><span class="sxs-lookup"><span data-stu-id="4e092-171">Use Visual Studio to review the sample hotel reservation C# application code and deploy to a bot in Azure.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="4e092-172">Előfeltételek</span><span class="sxs-lookup"><span data-stu-id="4e092-172">Prerequisites</span></span>

<span data-ttu-id="4e092-173">Meglévő Azure-fiókkal kell rendelkeznie.</span><span class="sxs-lookup"><span data-stu-id="4e092-173">You must have an existing Azure account.</span></span> <span data-ttu-id="4e092-174">Ha nem rendelkezik Azure-előfizetéssel, mindössze néhány perc alatt létrehozhat egy [ingyenes fiókot](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) a virtuális gép létrehozásának megkezdése előtt.</span><span class="sxs-lookup"><span data-stu-id="4e092-174">If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.</span></span>

### <a name="walk-through"></a><span data-ttu-id="4e092-175">Az útmutatóban</span><span class="sxs-lookup"><span data-stu-id="4e092-175">Walk-through</span></span>

<span data-ttu-id="4e092-176">Az infrastruktúra-összetevőket a Resource Manager-sablon üzembe helyezéséhez hajtsa végre az alábbi lépéseket.</span><span class="sxs-lookup"><span data-stu-id="4e092-176">To deploy the infrastructure components with a Resource Manager template, perform the following steps.</span></span>

<!-- markdownlint-disable MD033 -->

1. <span data-ttu-id="4e092-177">Kattintson a **üzembe helyezés az Azure** gombra:</span><span class="sxs-lookup"><span data-stu-id="4e092-177">Click the **Deploy to Azure** button:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fcommerce-chatbot.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. <span data-ttu-id="4e092-178">Várja meg a sablon üzembe helyezéséhez az Azure Portal megnyitásához, majd kövesse az alábbi lépéseket:</span><span class="sxs-lookup"><span data-stu-id="4e092-178">Wait for the template deployment to open in the Azure portal, then complete the following steps:</span></span>
   - <span data-ttu-id="4e092-179">Válassza ki a **új létrehozása** erőforrás csoportra, majd adjon meg egy nevet például *myCommerceChatBotInfrastructure* a szövegmezőben.</span><span class="sxs-lookup"><span data-stu-id="4e092-179">Choose to **Create new** resource group, then provide a name such as *myCommerceChatBotInfrastructure* in the text box.</span></span>
   - <span data-ttu-id="4e092-180">Válassza ki a régiót, a **hely** legördülő listából.</span><span class="sxs-lookup"><span data-stu-id="4e092-180">Select a region from the **Location** drop-down box.</span></span>
   - <span data-ttu-id="4e092-181">Adjon meg egy felhasználónevet és a biztonságos jelszó az SQL Server-rendszergazdai fiók.</span><span class="sxs-lookup"><span data-stu-id="4e092-181">Provide a username and secure password for the SQL Server administrator account.</span></span>
   - <span data-ttu-id="4e092-182">Tekintse át a használati feltételeket, majd ellenőrizze **elfogadom a feltételeket és a fenti feltételeket**.</span><span class="sxs-lookup"><span data-stu-id="4e092-182">Review the terms and conditions, then check **I agree to the terms and conditions stated above**.</span></span>
   - <span data-ttu-id="4e092-183">Válassza ki a **beszerzési** gombra.</span><span class="sxs-lookup"><span data-stu-id="4e092-183">Select the **Purchase** button.</span></span>

<!-- markdownlint-enable MD033 -->

<span data-ttu-id="4e092-184">A központi telepítés befejezése néhány percet vesz igénybe.</span><span class="sxs-lookup"><span data-stu-id="4e092-184">It takes a few minutes for the deployment to complete.</span></span>

### <a name="deploy-web-app-chatbot"></a><span data-ttu-id="4e092-185">Webalkalmazás csevegőrobot üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="4e092-185">Deploy Web App chatbot</span></span>

<span data-ttu-id="4e092-186">Az Azure CLI segítségével a csevegőrobot hozhat létre.</span><span class="sxs-lookup"><span data-stu-id="4e092-186">To create the chatbot, use the Azure CLI.</span></span> <span data-ttu-id="4e092-187">A következő példa telepíti a CLI-bővítményt a Bot Service, létrehoz egy erőforráscsoportot, majd üzembe helyezi a robot, az Application Insights.</span><span class="sxs-lookup"><span data-stu-id="4e092-187">The following example installs the CLI extension for Bot Service, creates a resource group, then deploys a bot that uses Application Insights.</span></span> <span data-ttu-id="4e092-188">Amikor a rendszer kéri, Microsoft-fiókja hitelesítéséhez, és lehetővé téve a robot regisztrálja magát a Bot Service és Language Understanding és Intelligent Services (LUIS) alkalmazást.</span><span class="sxs-lookup"><span data-stu-id="4e092-188">When prompted, authenticate your Microsoft account and allow the bot to register itself with the Bot Service and Language Understanding and Intelligent Services (LUIS) app.</span></span>

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

### <a name="deploy-chatbot-c-application-code"></a><span data-ttu-id="4e092-189">Csevegőrobot C# alkalmazás kód üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="4e092-189">Deploy chatbot C# application code</span></span>

<span data-ttu-id="4e092-190">A minta C#-alkalmazás a Githubon érhető el:</span><span class="sxs-lookup"><span data-stu-id="4e092-190">A sample C# application is available on GitHub:</span></span>

- [<span data-ttu-id="4e092-191">Kereskedelmi Bot C#-minta</span><span class="sxs-lookup"><span data-stu-id="4e092-191">Commerce Bot C# sample</span></span>](https://github.com/Microsoft/AzureBotServices-scenarios/tree/master/CSharp/Commerce/src)

<span data-ttu-id="4e092-192">A mintaalkalmazás az Azure Active Directory authentication összetevői és a Language Understanding és Intelligent Services (LUIS) összetevő integráció a Cognitive Services.</span><span class="sxs-lookup"><span data-stu-id="4e092-192">The sample application includes the Azure Active Directory authentication components and integration with the Language Understanding and Intelligent Services (LUIS) component of Cognitive Services.</span></span> <span data-ttu-id="4e092-193">Az alkalmazás létrehozása és üzembe helyezése a forgatókönyvet a Visual Studio használatához.</span><span class="sxs-lookup"><span data-stu-id="4e092-193">The application requires Visual Studio to build and deploy the scenario.</span></span> <span data-ttu-id="4e092-194">További tájékoztatást az AAD B2C-vel és a LUIS-alkalmazás konfigurálása a GitHub-tárház dokumentációjában található.</span><span class="sxs-lookup"><span data-stu-id="4e092-194">Additional information on configuring AAD B2C and the LUIS app can be found in the GitHub repo documentation.</span></span>

## <a name="pricing"></a><span data-ttu-id="4e092-195">Díjszabás</span><span class="sxs-lookup"><span data-stu-id="4e092-195">Pricing</span></span>

<span data-ttu-id="4e092-196">Ebben a forgatókönyvben költségének megismeréséhez, a szolgáltatások mindegyike a költségkalkulátor az előre konfigurált.</span><span class="sxs-lookup"><span data-stu-id="4e092-196">To explore the cost of running this scenario, all of the services are pre-configured in the cost calculator.</span></span> <span data-ttu-id="4e092-197">Tekintse meg, hogyan díjszabását szeretné módosítani az adott használati esetekhez, módosítsa a megfelelő változókat egyezik a várt forgalomhoz.</span><span class="sxs-lookup"><span data-stu-id="4e092-197">To see how the pricing would change for your particular use case, change the appropriate variables to match your expected traffic.</span></span>

<span data-ttu-id="4e092-198">Adtunk meg három példa költség profilok feldolgozni a csevegőrobot várt üzenetek száma alapján:</span><span class="sxs-lookup"><span data-stu-id="4e092-198">We have provided three sample cost profiles based on the number of messages you expect your chatbot to process:</span></span>

- <span data-ttu-id="4e092-199">[Kis][small-pricing]: havonta < 10 000 üzenetek feldolgozására utal. a díjszabási példa.</span><span class="sxs-lookup"><span data-stu-id="4e092-199">[Small][small-pricing]: this pricing example correlates to processing < 10,000 messages per month.</span></span>
- <span data-ttu-id="4e092-200">[Közepes][medium-pricing]: a díjszabási Példa havi 500 000 < üzenetek feldolgozására utal.</span><span class="sxs-lookup"><span data-stu-id="4e092-200">[Medium][medium-pricing]: this pricing example correlates to processing < 500,000 messages per month.</span></span>
- <span data-ttu-id="4e092-201">[Nagy][large-pricing]: a díjszabási példa havonta < 10 millió üzenetek feldolgozására utal.</span><span class="sxs-lookup"><span data-stu-id="4e092-201">[Large][large-pricing]: this pricing example correlates to processing < 10 million messages per month.</span></span>

## <a name="related-resources"></a><span data-ttu-id="4e092-202">Kapcsolódó források (lehet, hogy a cikkek angol nyelvűek)</span><span class="sxs-lookup"><span data-stu-id="4e092-202">Related resources</span></span>

<span data-ttu-id="4e092-203">Az Azure Bot Service az interaktív oktatóanyagok készletének, tekintse meg a [oktatóanyag szakasz] [ botservice-docs] dokumentáció.</span><span class="sxs-lookup"><span data-stu-id="4e092-203">For a set of guided tutorials for the Azure Bot Service, see the [tutorial section][botservice-docs] of the documentation.</span></span>

<!-- links -->

[aadb2c-docs]: /azure/active-directory-b2c/active-directory-b2c-overview
[aad-docs]: /azure/active-directory/
[appinsights-docs]: /azure/application-insights/app-insights-overview
[appservice-docs]: /azure/app-service/
[architecture]: ./media/architecture-commerce-chatbot.png
[autoscaling]: ../../best-practices/auto-scaling.md
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