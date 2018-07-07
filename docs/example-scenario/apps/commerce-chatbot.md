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
# <a name="conversational-azure-chatbot-for-hotel-reservations"></a>Természetes nyelvi Azure csevegőrobot szállodai foglalások számára

Ebben a példaforgatókönyvben olyan vállalatok, amelyek a természetes nyelvi csevegőrobot kell integrálható alkalmazásokat alkalmazható. Ebben a megoldásban a C# csevegőrobot egy Szálloda láncot, amely lehetővé teszi az ügyfelek számára, hogy ellenőrizze a rendelkezésre állás és a egy webes vagy mobilalkalmazás segítségével könyv vendéglátóipar szolgál.

Példaforgatókönyvek közé tartozik, így az ügyfelek számára Szálloda rendelkezésre állás és a könyv termek megtekintése egy étterem elvihető menü tekintse át és helyezze el egy food sorrendben, vagy keresse meg és fényképek nyomatok. Hagyományosan vállalkozások hire kész, és ezek ügyfél-kérésekre való válaszolás ügyfél szolgáltatási ügynökök kell, és a felhasználónak kell várja meg, hogy egy munkatárs elérhető segítséget nyújt.

Azure-szolgáltatások például a Bot Service és Language Understanding vagy Speech API szolgáltatás segítségével, a vállalatok végrehajtásában, ügyfelek és feldolgozzák a megrendeléseket vagy automatizált és méretezhető robotokat rendelkező foglalásokat.

## <a name="potential-use-cases"></a>A lehetséges alkalmazási helyzetek

Fontolja meg a megoldást a következő esetekben használja:

* Éttermek elvihető menüben, és a sorrend food megtekintése
* Szálloda elérhetőségének ellenőrzése és szobát
* Rendelkezésre álló fényképek keresse meg és nyomatrendelés

## <a name="architecture"></a>Architektúra

![Az Azure-összetevőket a természetes nyelvi csevegőrobot részt architektúrájának áttekintése][architecture]

Ez a megoldás, amely úgy működik, mint a számára egy Szálloda recepciószolgálata természetes nyelvi robotprogramok ismerteti. Az adatok a következőképpen folyamatok a megoldáson keresztül:

1. Az ügyfél és a egy mobile csevegőrobot, vagy a web app fér hozzá.
2. Az Azure Active Directory B2C (üzleti 2 ügyfél), a felhasználó hitelesítését.
3. A Bot Service használata, a felhasználói kérelmek Szálloda rendelkezésre állási kapcsolatos információkat.
4. A cognitive Services dolgozza fel a természetes nyelvi kérés tudni, hogy az ügyfél-kommunikációt.
5. Miután a felhasználó nem elégedett az eredménnyel, a bot hozzáadása, vagy frissíti az ügyfél foglalását egy SQL Database-ben.
6. Az Application Insights futásidejű telemetriai adatokat a fejlesztési és üzemeltetési csapat a robot teljesítményének javítása és használatának érdekében a folyamat során gyűjt.

### <a name="components"></a>Összetevők

* [Az Azure Active Directory] [ aad-docs] van a Microsoft több-bérlős felhőalapú címtár- és identitáskezelési szolgáltatás. Az Azure AD B2C összekötő így azonosíthatja a magánszemélyek külső azonosítókat, például Google, Facebook vagy a Microsoft-Account támogatja.
* [App Service-ben] [ appservice-docs] hozhat létre és üzemeltethet webalkalmazásokat az Ön által választott programozási nyelven infrastruktúra kezelése nélkül is.
* [Bot Service] [ botservice-docs] elkészítheti, tesztelheti, telepítheti és felügyelheti a robotokat intelligens eszközöket biztosít.
* [A cognitive Services] [ cognitive-docs] lehetővé teszi, hogy intelligens algoritmusok segítségével jelenik meg, hallgassa meg, beszéljenek, megértéséhez és a felhasználó szükségletek értelmezése a kommunikáció természetes módszereivel keresztül.
* [Az SQL Database] [ sqldatabase-docs] van egy teljes körűen felügyelt relációs adatbázis-szolgáltatás, amely az SQL Server-motorhoz.
* [Az Application Insights] [ appinsights-docs] egy bővíthető alkalmazásteljesítmény-felügyeleti (APM) szolgáltatás, amely lehetővé teszi az alkalmazások, például a csevegőrobot teljesítményének figyelése.

### <a name="alternatives"></a>Alternatív megoldások

* [Microsoft Speech API] [ speech-api] hogyan kommunikáljanak az ügyfelek a robot módosításához is használható.
* [A QnA Maker] [ qna-maker] feltárhatja, hogy gyorsan ismeretek hozzáadása a robot a szolgáltatásban tárolt részben strukturált tartalmat, például egy Gyakori is használható.
* [Fordítói szöveg] [ translator] egy szolgáltatás, amely akkor érdemes lehet könnyen többnyelvű támogatás hozzáadása a robot.

## <a name="considerations"></a>Megfontolandó szempontok

### <a name="availability"></a>Rendelkezésre állás

Ez a megoldás az Azure SQL Database ügyfél foglalások tárolásához. Az SQL Database szolgáltatás zóna redundáns adatbázisainak, feladatátvételi csoportok és georeplikáció. További információkért lásd: [Azure SQL Database a rendelkezésre állás][sqlavailability-docs].

Méretezhetőség témaköröket talál a [rendelkezésre állási ellenőrzőlista] [ availability] a az Azure Architecture Centert.

### <a name="scalability"></a>Méretezhetőség

A megoldás az Azure App Service-ben. Az App Service automatikusan skálázhatja a robot futtató példányok száma. Ez a funkció lehetővé teszi a webalkalmazás és csevegőrobot kereslet kielégítésére. Az automatikus méretezésben további információkért lásd: [automatikus méretezés ajánlott eljárásai] [ autoscaling] architektúra közepén.

Méretezhetőség témaköröket talál a [méretezési ellenőrzőlista] [ scalability] a az Azure Architecture Centert.

### <a name="security"></a>Biztonság

Ez a megoldás Azure Active Directory B2C-t használ felhasználók hitelesítésére (üzleti 2 fogyasztói). Az AAD B2C-vel a csevegőrobot bármely ügyfél bizalmas fiókadatok vagy a hitelesítő adatok nem tárolja. További információkért lásd: [Azure Active Directory B2C – áttekintés][aadb2c-docs].

Az Azure SQL Database-ben tárolt adatok titkosítása transzparens adattitkosítás (TDE). SQL-adatbázis is kínál, amely titkosítja az adatok lekérdezése és feldolgozása során mindig titkosítja. További információ az SQL Database biztonsági: [Azure SQL Database biztonsági és megfelelőségi][sqlsecurity-docs].

Általános megoldások biztonságos, tekintse át a [Azure Security dokumentációja][security].

### <a name="resiliency"></a>Rugalmasság

Ez a megoldás az Azure SQL Database ügyfél foglalások tárolásához. Az SQL Database szolgáltatás zóna zónaredundáns adatbázisok, a feladatátvételi csoportok, a georeplikáció és az automatikus biztonsági mentést. Ezek a funkciók lehetővé teszik az alkalmazás egy karbantartási esemény vagy szolgáltatáskimaradás esetén is működőképes marad. További információkért lásd: [Azure SQL Database a rendelkezésre állás][sqlavailability-docs].

Az alkalmazás állapotának figyeléséhez, ez a megoldás az Application Insights. Az Application Insights riasztást, és reagálni, amely hatással lenne, a felhasználói élmény és a csevegőrobot rendelkezésre állása teljesítménybeli problémák. További információkért lásd: [Mi az Application Insights?][appinsights-docs]

Rugalmas megoldások tervezésével kapcsolatos általános útmutatásért lásd: [rugalmas alkalmazások tervezése az Azure][resiliency].

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

Ez a megoldás segítségével megismerheti a legtöbb összpontosított területek három összetevőből áll:

* [Infrastruktúra-összetevőket](#deploy-infrastructure-components). Egy Azure Resource Manager-sablon használatával helyezhet üzembe egy App Service, webalkalmazás, az Application Insights, tárfiók, és az SQL Server és adatbázis alapvető infrastruktúra összetevők.
* [Webes alkalmazás Csevegőrobot](#deploy-web-app-chatbot). A robot a Bot Service és Language Understanding és Intelligent Services (LUIS) alkalmazás üzembe helyezéséhez az Azure CLI használatával.
* [Mintaalkalmazás a C# csevegőrobot](#deploy-chatbot-c-application-code). Tekintse át a minta Szálloda foglalás C# alkalmazás kódja és a egy robot, az Azure-ban való üzembe helyezése a Visual Studio használatával.

**Előfeltételek.** Meglévő Azure-fiókkal kell rendelkeznie. Ha nem rendelkezik Azure-előfizetéssel, mindössze néhány perc alatt létrehozhat egy [ingyenes fiókot](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) a virtuális gép létrehozásának megkezdése előtt.

### <a name="deploy-infrastructure-components"></a>Infrastruktúra-összetevők telepítése

Az infrastruktúra-összetevőket az Azure Resource Manager-sablon üzembe helyezéséhez hajtsa végre az alábbi lépéseket.

1. Kattintson a **üzembe helyezés az Azure** gombra:<br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fcommerce-chatbot.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. Várja meg a sablon üzembe helyezéséhez az Azure Portal megnyitásához, majd kövesse az alábbi lépéseket:
   * Válassza ki a **új létrehozása** erőforrás csoportra, majd adjon meg egy nevet például *myCommerceChatBotInfrastructure* a szövegmezőben.
   * Válassza ki a régiót, a **hely** legördülő listából.
   * Adjon meg egy felhasználónevet és a biztonságos jelszó az SQL Server-rendszergazdai fiók.
   * Tekintse át a használati feltételeket, majd ellenőrizze **elfogadom a feltételeket és a fenti feltételeket**.
   * Válassza ki a **beszerzési** gombra.

A központi telepítés befejezése néhány percet vesz igénybe.

### <a name="deploy-web-app-chatbot"></a>Webalkalmazás csevegőrobot üzembe helyezése

Az Azure CLI segítségével a csevegőrobot hozhat létre. A következő példa telepíti a CLI-bővítményt a Bot Service, létrehoz egy erőforráscsoportot, majd üzembe helyezi a robot, az Application Insights. Amikor a rendszer kéri, Microsoft-fiókja hitelesítéséhez, és lehetővé téve a robot regisztrálja magát a Bot Service és Language Understanding és Intelligent Services (LUIS) alkalmazást.

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

### <a name="deploy-chatbot-c-application-code"></a>Csevegőrobot C# alkalmazás kód üzembe helyezése

A minta C#-alkalmazás a Githubon érhető el: 

* [Kereskedelmi Bot C#-minta](https://github.com/Microsoft/AzureBotServices-scenarios/tree/master/CSharp/Commerce/src)

A mintaalkalmazás az Azure Active Directory authentication összetevői és a Language Understanding és Intelligent Services (LUIS) összetevő integráció a Cognitive Services. Az alkalmazás létrehozása és a megoldás üzembe helyezése a Visual Studio használatához. További tájékoztatást az AAD B2C-vel és a LUIS-alkalmazás konfigurálása a GitHub-tárház dokumentációjában található.

## <a name="pricing"></a>Díjszabás

Ez a megoldás költségének megismeréséhez, a szolgáltatások mindegyike a költségkalkulátor az előre konfigurált. Tekintse meg, hogyan díjszabását szeretné módosítani az adott használati esetekhez, módosítsa a megfelelő változókat egyezik a várt forgalomhoz.

Adtunk meg három példa költség profilok feldolgozni a csevegőrobot várt üzenetek mennyisége alapján:

* [Kis][small-pricing]: Ez havonta < 10 000 üzenetek feldolgozására utal.
* [Közepes][medium-pricing]: Ez havi 500 000 < üzenetek feldolgozására utal.
* [Nagy][large-pricing]: Ez havonta < 10 millió üzenetek feldolgozására utal.

## <a name="related-resources"></a>Kapcsolódó erőforrások

Az interaktív oktatóanyagok az Azure Bot Service kihasználva készletének, tekintse meg a [oktatóanyag csomópont] [ botservice-docs] dokumentáció.

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