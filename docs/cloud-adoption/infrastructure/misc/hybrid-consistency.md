---
title: 'CAF: Hibrid felhő konzisztencia'
description: A hibrid felhő konzisztencia megközelítés meghatározása
author: BrianBlanchard
ms.date: 12/27/2018
ms.openlocfilehash: 726ea56f52d68f6c9b0d1478d19a91c2b7f12fdd
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899466"
---
# <a name="create-hybrid-cloud-consistency"></a>Hibrid felhő konzisztencia

Ez a cikk végigvezeti a magas szintű megközelítések a hibrid felhő konzisztencia létrehozásához.

Hibrid üzembe helyezési modellt migrálás során is csökkentheti a kockázatokat, és hozzájárulnak a infrastruktúra zökkenőmentes átmenet. Felhőalapú platform a legnagyobb szintű rugalmasságot kínálnak, esetén, az üzleti folyamatokat. Számos szervezet is szívesen térjen át a felhőbe, teljes ellenőrzés a leginkább bizalmas adatokon inkább előnyben részesítése. Sajnos a helyszíni kiszolgálók nem teszi lehetővé az innováció, mivel a felhő költsége. Felhőalapú hibrid megoldás lehetővé teszi a használja ki mindkét világ előnyeit: A felhőbeli innovációk sebességétől és kényelmes a helyszíni felügyeleti.

## <a name="integrate-hybrid-cloud-consistency"></a>Hibrid felhő konzisztencia integrálása

Felhőalapú hibrid megoldás használata lehetővé teszi a szervezetek számára, hogy a számítási erőforrások skálázását. Is szükségtelenné teszi, hogy kezelni a rövid távú jelentkező nagy beruházási költségekkel jár. Ha változik az üzleti meghajtóhoz több bizalmas adatok és alkalmazások helyi erőforrásokat kell, azt, egyszerűbb, gyorsabb és olcsóbb a felhőbeli erőforrások megszüntetése. Fizetnie csak ezeket az erőforrásokat a szervezet ideiglenesen használ kellene vásárolnia és további erőforrások karbantartása helyett. Ez csökkenti a berendezés, előfordulhat, hogy hosszú időn keresztül tétlen marad. Hibrid felhőalapú számítástechnika egy "minden lehetséges világ legjobb" platform, felhőmegoldásból felhő-számítástechnika a rugalmasságot, méretezhetőséget és költséghatékonyságot; a legkisebb lehetséges kockázatát adatok felfedésének kockázatát.

![Hibrid felhő konzisztencia létrehozása több identitás, felügyeleti, biztonság, adatok, fejlesztési és DevOps](../../_images/hybrid-consistency.png)
*1. ábra. Hibrid felhő konzisztencia létrehozása több identitás, felügyeleti, biztonság, adatok, fejlesztési és DevOps*

Egy igazi hibrid felhőalapú megoldás négy összetevő jelentős előnnyel jár, többek között számos lehetőséget kínál, amelyek mindegyike kell megadnia:

- Közös identitással a helyszíni és felhőalapú alkalmazásokhoz: Ez javítja a felhasználói termelékenység csökkenését felhasználók egyszeri bejelentkezés (SSO), így az összes alkalmazásaikat. Emellett biztosítja a konzisztencia, mint az alkalmazások és felhasználók közötti felhőhöz vagy hálózati határok.
- Integrált felügyelet és biztonság a hibrid felhő: Ez biztosít összefüggő figyelése, kezelése és biztosítsa a környezetét, átláthatóbb és szabályozottabb engedélyezése lehetőséget.
- Az adatközpontból és a egy egységes platform: Adatok hordozhatóságának, zökkenőmentes hozzáférést biztosít a helyszíni kombinálva létrejön, és felhőbeli adatok szolgáltatásokat tisztább képet kaphat az összes adat a forrásból.
- Egységes fejlesztési és DevOps a felhő és a helyszíni adatközpontok között: Ez lehetővé teszi az alkalmazások áthelyezése szükséges, fejlesztés eredményességének növelése, javításához, mindkét helyen már ugyanabban a fejlesztési környezetben, a két környezet között.
  
Ezek az összetevők Azure szempontból közé:

- Az Azure Active Directory (Azure AD), amely együttműködik a helyszíni Azure AD közös identitást biztosíthat minden felhasználónak. Egyszeri bejelentkezés a helyszíni és a felhőn keresztül leegyszerűsíti a felhasználók biztonságosan férhetnek hozzá az alkalmazások és eszközök szükségük van. Rendszergazdai biztonsági és cégirányítási ellenőrzéseket kezelheti, így a felhasználók mire van szükségük, rugalmasan, állítsa be ezeket az engedélyeket, anélkül, hogy befolyásolná a felhasználói élmény.
- Az Azure mindkét felhőbe integrált felügyeleti és biztonsági szolgáltatásokat nyújt a és a helyszíni infrastruktúrát, beleértve az eszközök figyelését, konfigurálását és a hibrid felhők védelmének egy integrált választékát. A teljes körű módszerrel kifejezetten címek valós kihívást a mérlegeli egy hibrid felhőmegoldás szervezetek.
- Az Azure hybrid cloud gyakori eszközöket biztosít, amelyek zökkenőmentesen és hatékonyan, győződjön meg arról, hogy minden adat, a biztonságos hozzáférést. Konzisztens adatplatform létrehozása a Microsoft SQL Server Azure-adatszolgáltatások kombinálni. A konzisztens hibrid felhőalapú modell lehetővé teszi a felhasználóknak egyaránt üzemeltetési és elemzési adatokat, és az azonos szolgáltatásokkal a helyszínen és a felhőalapú adattárház, adatok elemzése és az adatvizualizációról működik.
- A Microsoft Azure cloud servicesben, kombinálva a Microsoft Azure Stack a helyszíni, egységesített fejlesztést és Devopsot adja meg. A felhőben és helyszíni konzisztencia azt jelenti, hogy a fejlesztési és üzemeltetési csapat alkalmazások készíthetők, amelyek mindkét környezetben futnak javítások és üzembe helyezhetik a megfelelő helyre. Sablonok között a hibrid megoldást is, ami tovább egyszerűsítheti a DevOps-folyamatokkal felhasználhatja.

## <a name="azure-stack-in-a-hybrid-cloud-environment"></a>Az Azure Stack egy hibrid felhőkörnyezetben

Microsoft Azure Stack az egyszerűsített fejlesztési, felügyeleti és biztonsági biztosítása érdekében, hogy az Azure nyilvános felhőbeli szolgáltatásaival összhangban van az egy hibrid felhőmegoldás, amely lehetővé teszi a cégek számára, hogy egységes Azure-szolgáltatások azok az adatközpontban. Az Azure Stack egy Azure-bővítmény, a helyszíni környezetből Azure-szolgáltatások futtatásához, és folytassa az Azure-felhő, amikor szükség.

Az Azure Stack üzembe helyezését és üzemeltetését is IaaS és PaaS ugyanazokkal az eszközökkel, és az Azure nyilvános felhő mint ugyanazt a felhasználói élményt kínál teszi lehetővé. Kezelés az Azure Stack, a webes felhasználói felület portálon keresztül vagy a PowerShell, egy egységes megjelenést és működést rendelkezik a rendszergazdák és a végfelhasználók számára az Azure-ral.

Az Azure és az Azure Stack zárolásának feloldása a ügyfél felé irányuló, mind belső üzleti alkalmazások, beleértve az új hibrid használati eseteket:

- **Peremhálózati és hálózat nélküli megoldásoknál**. Ügyfelek kezelheti késéssel és elérhetőséggel kapcsolatos követelmények helyileg az Azure Stackben adatok feldolgozását, és majd összesíti az Azure-ban történő közös alkalmazás-logikával további elemzéshez. Számos ügyfelünk olyan különböző környezetekben, beleértve a gyáraktól, hajózási részeként szerezhető be és adatbányászatot végezhessen az adataiban tengelyek ebben a forgatókönyvben edge iránt.
- **Felhőbeli alkalmazásokat, amelyek megfelelnek a különböző szabványok**. Ügyfelek fejlesszen és helyezzen üzembe alkalmazásokat az Azure-ban, a teljesen rugalmasan telepíthetők a helyszínen az Azure Stack használatával szabályozási vagy a házirend követelményeinek, a kódmódosítás nélkül. Globális naplózási, pénzügyi jelentések, kereskedelmi deviza, online játékokhoz és költségelszámolás szemléltető alkalmazás közé. Ügyfelek néha kíváncsi, üzembe helyezése az Azure-ban, az üzleti és technikai követelmények egyazon alkalmazás különböző példányaiban. Bár az Azure megfelel a legtöbb követelmények, Azure Stack kiegészítők az üzembe helyezés megközelítést, ahol szükséges.
- **Felhőalapú alkalmazás helyszíni modellje**. Ügyfelek használhatják az Azure web services, a tárolókat, kiszolgáló nélküli, és a mikroszolgáltatás-architektúrák frissítésére és meglévő alkalmazásaik vagy újak létrehozása. Egységes fejlesztési és üzemeltetési folyamatokat is használhat a felhőben és helyszíni Azure Stack Azure-ban. Nincs egy egyre bővülő érdekeltséget alkalmazások korszerűsítése, beleértve az alapvető fontosságú alkalmazáshoz.

Az Azure Stack két üzembe helyezési lehetőség érhető el:

- **Az Azure Stack integrált rendszerek**. Az Azure Stack integrált rendszerek a hardver és a Microsoft partnereinek évente megrendezett felhő saját innováció biztosító megoldások létrehozásának kialakított partnerség révén érhetők el felügyeleti vásárlásánál átgondolni. Azure Stack egy integrált rendszer, a hardver- és érhető el, mert a rugalmasságot és irányítást, megfelelő mennyiségű beolvasása közben továbbra is bevezetése az innováció a felhőben. Az Azure Stack integrált rendszerek tartománya az a 4 – 12 csomópontok méretét, és közösen a hardver partner és a Microsoft által támogatott. Azure Stack integrált rendszerek használatával engedélyezhető az új forgatókönyvek használhatók a termelési számítási feladatokhoz.
- **Az Azure Stack Development Kit**. A Microsoft Azure Stack Development Kit egy egy csomópontos üzembe helyezhető Azure Stacket, ami segítségével kipróbálni és megismerni az Azure stackről. A csomag egy fejlesztői környezet, amelyben API-k használatával is fejleszthet és eszközöket, amelyek megegyeznek az Azure-ral is használhatja. Az Azure Stack Development Kit nem javasolt éles környezetben használható.

## <a name="azure-stack-one-cloud-ecosystem"></a>Azure Stack One Cloud Ecosystem

Azure Stack kezdeményezések felgyorsíthatja a teljes Azure-ökoszisztéma kihasználva:

- Az Azure gondoskodik arról, hogy a legtöbb alkalmazás és szolgáltatás, az Azure certified fog működni az Azure Stacken. Több független szoftverszállítók &mdash; többek között a Bitnami, a Docker, a Kemp technológiák, a Pivotal Cloud Foundry-hoz, a Red Hat Enterprise Linux és a SUSE Linux &mdash; kiterjesztése megoldásaikat az Azure Stackhez.
- Kérheti, hogy az Azure Stack-i és a egy teljes körűen felügyelt szolgáltatásként működik. Számos partnerünk &mdash; Tieto, Yourhosting, Revera, Pulsant és NTT &mdash; fog felügyelt szolgáltatásajánlatok Azure és az Azure Stackben kis türelmet kérünk. Ezek a partnerek a Cloud Solution Provider (Felhőszolgáltatók) programon keresztül az Azure felügyelt szolgáltatások lett kidolgozását, és most kiterjesztése ajánlataik hibrid megoldásokat tartalmazza.
- Egy teljes Példa teljes körűen felügyelt felhőalapú hibrid megoldás, Avanade elkötelezett egy teljes körű ajánlatot, amely tartalmazza az átalakítási felhőszolgáltatások, szoftverek, infrastruktúra, telepítés és konfiguráció, és folyamatban lévő felügyelt szolgáltatások, az ügyfelek Az Azure Stack egyszerűen, mint az Azure-ral.
- Systems-rendszerintegrátorok (SI) segítségével, alkalmazások korszerűsítése kezdeményezések felgyorsíthatja az ügyfelek számára teljes körű Azure-megoldások fejlesztésével. Részletes Azure ismeretek, a tartomány és az iparág kapcsolatos és a folyamat tapasztalatainak (például DevOps) hoznak. Minden egyes Azure Stack-felhő lehetőséget nyújt a megoldás kialakításához, vezethet és befolyásolja a rendszer központi telepítése, testre szabhatja a csomagban foglalt funkciók és üzemeltetési tevékenységek nyújthat egy SI. Ez magában foglalja a SIs például Avanade, DXC, a Dell EMC szolgáltatások, InFront Consulting csoport, a HPE Pointnext és Pricewaterhouse Coopers (PwC).
