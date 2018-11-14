---
title: Beszélgető csevegőrobot szállodai foglalásokhoz az Azure-ban
description: Beszélgető csevegőrobotot hozhat létre kereskedelmi alkalmazásokhoz az Azure Bot Service segítségével.
author: iainfoulds
ms.date: 07/05/2018
ms.openlocfilehash: a922a75d621672fcac95296b1d99112d68c91107
ms.sourcegitcommit: 0a31fad9b68d54e2858314ca5fe6cba6c6b95ae4
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/13/2018
ms.locfileid: "51610770"
---
# <a name="conversational-chatbot-for-hotel-reservations-on-azure"></a>Beszélgető csevegőrobot szállodai foglalásokhoz az Azure-ban

Ebben a példaforgatókönyvben olyan vállalatok, amelyek a természetes nyelvi csevegőrobot beépíthetik kell alkalmazható. Ebben a forgatókönyvben egy C# csevegőrobot egy Szálloda láncot, amely lehetővé teszi az ügyfelek számára, hogy ellenőrizze a rendelkezésre állás és a egy webes vagy mobilalkalmazás segítségével könyv vendéglátóipar szolgál.

A lehetséges használati módjai többek között, így az ügyfelek számára Szálloda rendelkezésre állás és a könyv termek megtekintése egy étterem elvihető menü tekintse át és helyezze el egy food sorrendben, vagy keresse meg és fényképek nyomatok. Hagyományosan vállalkozások hire kész, és ezek ügyfél-kérésekre való válaszolás ügyfél szolgáltatási ügynökök kell, és a felhasználónak kell várja meg, hogy egy munkatárs elérhető segítséget nyújt.

Azure-szolgáltatások például a Bot Service és Language Understanding vagy Speech API szolgáltatás segítségével, a vállalatok végrehajtásában, ügyfelek és feldolgozzák a megrendeléseket vagy automatizált és méretezhető robotokat rendelkező foglalásokat.

## <a name="relevant-use-cases"></a>Alkalmazási helyzetek

Egyéb alkalmazási helyzetek a következők:

* Egy étterem elvihető menü megtekintése és élelmiszer rendezése
* Szálloda rendelkezésre állásának ellenőrzése és a egy hely foglalása
* Keresés a rendelkezésre álló fényképek, és rendelése

## <a name="architecture"></a>Architektúra

![Az Azure-összetevőket a természetes nyelvi csevegőrobot részt architektúrájának áttekintése][architecture]

Ebben a forgatókönyvben a természetes nyelvi robot, amely úgy működik, mint a számára egy Szálloda recepciószolgálata ismerteti. A áramlanak keresztül az adatok a forgatókönyv a következő:

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
* [A cognitive Services] [ cognitive-docs] lehetővé teszi, hogy intelligens algoritmusok segítségével jelenik meg, hallgassa meg beszél, megértésében, valamint a felhasználói szükségletek értelmezése a kommunikáció természetes módszereivel keresztül.
* [Az SQL Database] [ sqldatabase-docs] van egy teljes körűen felügyelt relációs adatbázis-szolgáltatás, amely az SQL Server-motorhoz.
* [Az Application Insights] [ appinsights-docs] egy bővíthető alkalmazásteljesítmény-felügyeleti (APM) szolgáltatás, amely lehetővé teszi az alkalmazások, például a csevegőrobot teljesítményének figyelése.

### <a name="alternatives"></a>Alternatív megoldások

* [Microsoft Speech API] [ speech-api] hogyan kommunikáljanak az ügyfelek a robot módosításához is használható.
* [A QnA Maker] [ qna-maker] feltárhatja, hogy gyorsan ismeretek hozzáadása a robot a szolgáltatásban tárolt részben strukturált tartalmat, például egy Gyakori is használható.
* [Fordítói szöveg] [ translator] egy szolgáltatás, amely akkor érdemes lehet könnyen többnyelvű támogatás hozzáadása a robot.

## <a name="considerations"></a>Megfontolandó szempontok

### <a name="availability"></a>Rendelkezésre állás

Ebben a forgatókönyvben az Azure SQL Database ügyfél foglalások tárolásához. Az SQL Database szolgáltatás zóna redundáns adatbázisainak, feladatátvételi csoportok és georeplikáció. További információkért lásd: [Azure SQL Database a rendelkezésre állás][sqlavailability-docs].

Rendelkezésre állási témaköröket talál a [rendelkezésre állási ellenőrzőlista] [ availability] a az Azure Architecture Centert.

### <a name="scalability"></a>Méretezhetőség

Ebben a példában az Azure App Service-ben. Az App Service automatikusan skálázhatja a robot futtató példányok száma. Ez a funkció lehetővé teszi a webalkalmazás és csevegőrobot kereslet kielégítésére. További információ az automatikus méretezésben: [automatikus méretezés ajánlott eljárásai] [ autoscaling] a az Azure Architecture Centert.

Méretezhetőség témaköröket talál a [méretezési ellenőrzőlista] [ scalability] a az Azure Architecture Centert.

### <a name="security"></a>Biztonság

Ez a forgatókönyv Azure Active Directory B2C-t használ felhasználók hitelesítésére (üzleti 2 fogyasztói). Az AAD B2C-vel a csevegőrobot bármely ügyfél bizalmas fiókadatok vagy a hitelesítő adatok nem tárolja. További információkért lásd: [Azure Active Directory B2C – áttekintés][aadb2c-docs].

Az Azure SQL Database-ben tárolt adatok titkosítása transzparens adattitkosítás (TDE). SQL-adatbázis is kínál, amely titkosítja az adatok lekérdezése és feldolgozása során mindig titkosítja. További információ az SQL Database biztonsági: [Azure SQL Database biztonsági és megfelelőségi][sqlsecurity-docs].

Általános megoldások biztonságos, tekintse át a [Azure Security dokumentációja][security].

### <a name="resiliency"></a>Rugalmasság

Ebben a forgatókönyvben az Azure SQL Database ügyfél foglalások tárolásához. Az SQL Database szolgáltatás zóna zónaredundáns adatbázisok, a feladatátvételi csoportok, a georeplikáció és az automatikus biztonsági mentést. Ezek a funkciók lehetővé teszik az alkalmazás marad, ha van egy karbantartási esemény, illetve leállás. További információkért lásd: [Azure SQL Database a rendelkezésre állás][sqlavailability-docs].

Az alkalmazás állapotának figyelése, az ebben a forgatókönyvben az Application Insights. Az Application Insights riasztást, és reagálni, amely hatással lenne, a felhasználói élmény és a csevegőrobot rendelkezésre állása teljesítménybeli problémák. További információkért lásd: [Mi az Application Insights?][appinsights-docs]

Rugalmas megoldások tervezésével kapcsolatos általános útmutatásért lásd: [rugalmas alkalmazások tervezése az Azure][resiliency].

## <a name="deploy-the-scenario"></a>A forgatókönyv megvalósításához

Ez a forgatókönyv segítségével megismerheti a legtöbb összpontosított területek három összetevőből áll:

* [Infrastruktúra-összetevőket](#deploy-infrastructure-components). Egy Azure Resource Manager-sablon használatával helyezhet üzembe egy App Service, webalkalmazás, az Application Insights, tárfiók, és az SQL Server és adatbázis alapvető infrastruktúra összetevők.
* [Webes alkalmazás Csevegőrobot](#deploy-web-app-chatbot). A robot a Bot Service és Language Understanding és Intelligent Services (LUIS) alkalmazás üzembe helyezéséhez az Azure CLI használatával.
* [Mintaalkalmazás a C# csevegőrobot](#deploy-chatbot-c-application-code). Tekintse át a minta Szálloda foglalás C# alkalmazás kódja és a egy robot, az Azure-ban való üzembe helyezése a Visual Studio használatával.

**Előfeltételek.** Meglévő Azure-fiókkal kell rendelkeznie. Ha nem rendelkezik Azure-előfizetéssel, mindössze néhány perc alatt létrehozhat egy [ingyenes fiókot](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) a virtuális gép létrehozásának megkezdése előtt.

### <a name="deploy-infrastructure-components"></a>Infrastruktúra-összetevők telepítése

Az infrastruktúra-összetevőket a Resource Manager-sablon üzembe helyezéséhez hajtsa végre az alábbi lépéseket.

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

A mintaalkalmazás az Azure Active Directory authentication összetevői és a Language Understanding és Intelligent Services (LUIS) összetevő integráció a Cognitive Services. Az alkalmazás létrehozása és üzembe helyezése a forgatókönyvet a Visual Studio használatához. További tájékoztatást az AAD B2C-vel és a LUIS-alkalmazás konfigurálása a GitHub-tárház dokumentációjában található.

## <a name="pricing"></a>Díjszabás

Ebben a forgatókönyvben költségének megismeréséhez, a szolgáltatások mindegyike a költségkalkulátor az előre konfigurált. Tekintse meg, hogyan díjszabását szeretné módosítani az adott használati esetekhez, módosítsa a megfelelő változókat egyezik a várt forgalomhoz.

Adtunk meg három példa költség profilok feldolgozni a csevegőrobot várt üzenetek száma alapján:

* [Kis][small-pricing]: havonta < 10 000 üzenetek feldolgozására utal. a díjszabási példa.
* [Közepes][medium-pricing]: a díjszabási Példa havi 500 000 < üzenetek feldolgozására utal.
* [Nagy][large-pricing]: a díjszabási példa havonta < 10 millió üzenetek feldolgozására utal.

## <a name="related-resources"></a>Kapcsolódó források (lehet, hogy a cikkek angol nyelvűek)

Az Azure Bot Service az interaktív oktatóanyagok készletének, tekintse meg a [oktatóanyag szakasz] [ botservice-docs] dokumentáció.


<!-- links -->
[aadb2c-docs]: /azure/active-directory-b2c/active-directory-b2c-overview
[aad-docs]: /azure/active-directory/
[appinsights-docs]: /azure/application-insights/app-insights-overview
[appservice-docs]: /azure/app-service/
[architecture]: ./media/architecture-commerce-chatbot.png
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