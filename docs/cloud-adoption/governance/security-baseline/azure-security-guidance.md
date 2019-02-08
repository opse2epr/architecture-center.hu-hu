---
title: 'CAF: Biztonsági útmutató az Azure-hoz'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Milyen biztonsági útmutatást nyújt a Microsoft?
author: BrianBlanchard
ms.openlocfilehash: 845b02422e2df63425e29acdfd9b43b744934c9f
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899046"
---
<!-- markdownlint-disable MD026 -->

# <a name="what-security-guidance-does-microsoft-provide"></a>Milyen biztonsági útmutatást nyújt a Microsoft?

## <a name="security-guidance-and-tools"></a>Biztonsági útmutatás és eszközök

A Microsoft bevezette a [Service megbízható platformmal](https://servicetrust.microsoft.com) és megfelelőségi Feladatkezelővel segít a következőre:

- Megfelelőségi felügyeleti kihívásokat
- Felelősséget a értekezlet szabályozási követelmények teljesítése
- Önkiszolgáló eseményeket és enterprise cloud service-felhasználás mintavételezését kockázatfelmérések folytatása

Ezek az eszközök segítségével a szervezetek összetett megfelelőségi kötelezettségeket és növeljék adatvédelmi funkciók kiválasztása és a Microsoft Cloud services használatával lettek kialakítva.

**Szolgáltatás megbízható platformmal (STP)** biztosít részletes információkat és eszközöket a Microsoft Cloud services, Azure, Office 365, Dynamics 365-és Windows igényeinek kielégítése érdekében. STP-nyújtani a szabályozási biztonsági, megfelelőségi és adatvédelmi információkat a Microsoft Cloud kapcsolódó. Fontos, ahol közzétesszük az adatokat és kockázatfelmérések önkiszolgáló felhőbeli szolgáltatások és eszközök végrehajtásához szükséges erőforrásokat. STP készült Azure-ban, a szabályozásoknak való megfelelőséget tevékenységek nyomon követéséhez többek között:

- **Megfelelőségi Manager**: Megfelelőségi Manager, a munkafolyamat-alapú kockázati frissítésfelmérő eszköz a Microsoft szolgáltatás megbízható platformon, lehetővé teszi, hogy nyomon követheti, hozzárendelése és ellenőrzése a Microsoft Cloud Services szolgáltatást, például az Office 365, Dynamics kapcsolatos a szervezet a jogszabályoknak való megfelelőség tevékenységeket 365 és az Azure. További részleteket a következő szakaszban találja.
- **Dokumentumok megbízható**: Jelenleg az alábbi három kategória az útmutatók, amelyekkel bőséges Microsoft Cloud; értékelhető erőforrás További tudnivalók a Microsoft operations a biztonsági, megfelelőségi és adatvédelmi; és reagálhat rájuk javítása az adatvédelmi funkciók segítségével. Ezek a következők:
- **Naplózási jelentések**: Naplózási jelentések legyen naprakész a legújabb adatvédelmi, biztonsági és megfelelőséggel kapcsolatos információkat a Microsoft Cloud services számára teszi lehetővé. Ez magában foglalja az ISO, SOC, FedRAMP és más naplózási jelentések, híd betűk, és független külső naplózások, a Microsoft Cloud services, Azure, Office 365, Dynamics 365 és más például kapcsolatos anyagok.
- **Data protection útmutatók**: Data protection útmutatókból megtudhatja hogyan a Microsoft Cloud services az adatok védelme, és hogyan kezelheti felhőbeli adatok biztonsági és megfelelőségi a szervezet számára. Ez magában foglalja a részletes tanulmányok, amelyek részleteket biztosítanak a Microsoft tervez és működik, a cloud services, gyakori kérdések, év vége a biztonság állapotát, behatolási teszteredmények és útmutatás nyújtása a hitelkockázat értékelése tartson, és növelheti az adatok jelentések védelmi funkciókat.
- **Az Azure biztonsági és megfelelőségi tervezet**: Tervezetek biztosítanak az erőforrások létrehozásához és bevezetéséhez felhőalapú alkalmazásokat, amelyek segítséget nyújtanak segítséget megfelel a szigorú előírásoknak és szabványoknak. A bármely más felhőszolgáltatónál több tanúsítvány akkor a kritikus fontosságú számítási feladatok üzembe helyezni az Azure, tervek, amelyek tartalmazzák a megbízhatósági:

- Útmutató és iparág-specifikus áttekintése
- Ügyfél feladatkörei mátrix
- A referenciaarchitektúrák a modelljei
- Ellenőrzés végrehajtása mátrixok
- Automation üzembe helyezéséhez referenciaarchitektúrák
- Adatvédelmi erőforrások – Data Protection hatás értékelések, Adattulajdonosi kérelmek (DSR-kérelmek) és adatok illetéktelen behatolás értesítési dokumentáció a saját felelősségre vonhatóság program támogatásához az általános adatvédelmi rendelet (GDPR) építsen be.

- **Ismerkedés az általános adatvédelmi rendelet**: Microsoft-termékek és szolgáltatások megkönnyíti a szervezetek számára általános adatvédelmi rendelet követelményeinek gyűjtése, vagy a személyes adatok feldolgozása közben. STP célja, hogy a képességekre vonatkozó információk a Microsoft-szolgáltatások, amelyekkel konkrét az általános adatvédelmi rendelet követelményeinek teljesítésére. A dokumentáció segít az általános adatvédelmi rendelet elszámoltathatóság és a műszaki és szervezési ismeretekkel. A Data Protection hatás értékelések, Adattulajdonosi kérelmek (DSR-kérelmek) és adatok illetéktelen behatolás értesítési dokumentáció építhet be a saját felelősségre vonhatóság program az általános adatvédelmi rendelet támogatásához.
  - **Adattulajdonosi kérelmek**: Az általános adatvédelmi rendelet jogosultságot személyek (vagy az Adattulajdonosok) bizonyos személyes adataik feldolgozásának kapcsolatban. Ez magában foglalja a jogot arra, hogy javítsa ki a hibás adatokat eredményez, adatok, törlésére vagy korlátozza a feldolgozást, valamint kapnak adataik és azok egy másik vezérlő adatokat továbbítson a kérés teljesítéséhez.
  - **Adatok illetéktelen behatolás**: Az általános adatvédelmi rendelet építjük értesítési követelményei adatkezelők és a feldolgozók megsértése esetén a személyes adatok. STP hogyan a Microsoft megpróbál adatszivárgás megakadályozása az elsőként, hogyan észleli a Microsoft az megsértése, és információt arról, hogy a Microsoft hogyan fog válaszolni megsértése esetén és értesíteni, mint egy adatok vezérlő biztosít.
  - **Data protection hatásvizsgálati**: A Microsoft segítségével végezze el az általános adatvédelmi rendelet Data Protection hatás értékelések tartományvezérlők. Az általános adatvédelmi rendelet biztosít az esetekben, ahol DPIAs el kell végezni, például az automatikus feldolgozása céljából profilkészítési és hasonló műveletek; a teljes listája feldolgozási nagy méretű speciális, a személyes adatok kategóriák és a egy nyilvánosan elérhető terület nagy méretű rendszeres figyelést.
  - **Egyéb erőforrások**: A fenti szakaszokban ismertetett eszközök útmutatás kiegészítéseként a STP más erőforrások, mint a biztonsági és megfelelőségi központjában regionális megfelelőség, a további erőforrások is biztosít, és a szolgáltatás megbízható platformmal kapcsolatos gyakori kérdések Megfelelőségi Manager, és az adatvédelmi/GDPR.
- **Regionális megfelelőségi**: STP biztosít számos megfelelőségi dokumentumok és útmutató a Microsoft online szolgáltatásaihoz, beleértve a Cseh Köztársaság, lengyel és Románia különböző régiókra vonatkozóan megfelelőségi követelmények teljesítése érdekében.

## <a name="unique-intelligent-insights"></a>Egyedi intelligens elemzésekkel

Növekedésével mennyisége és összetettsége biztonsági jelek, amely meghatározza, hogy ezeket a jeleket hihető fenyegetések-e, és ezután működő, veszi túl hosszú. A Microsoft kínál a biztonsági intelligencia segítségével gyorsan észlelheti és az elhárításban felhőszinten szállított páratlan technológiai spektrumunk kihasználtságának növelését.

## <a name="azure-threat-intelligence"></a>Az Azure fenyegetések felderítése

A Security Centerben elérhető fenyegetésfelderítési funkcióval az informatikai rendszergazdák azonosíthatják a környezetre leselkedő biztonsági fenyegetéseket. Például azt, ha egy adott számítógép egy botnet része. A számítógépek akkor válhatnak egy botnet csomópontjává, ha a támadók illetéktelenül kártevőket telepítenek a számítógépre, amelyek titokban csatlakoztatják a gépet a vezérlőrendszerhez. A fenyegetésfelderítés emellett az illegális kommunikációs csatornákról, például a dark webről érkező potenciális fenyegetéseket is azonosítani tudja.

A fenyegetésfelderítési képesség biztosításához a Security Center a Microsofton belül több forrásból származó adatokat használ. A Security Center segítségével ez azonosítja a környezetre leselkedő potenciális fenyegetéseket. A Threat intelligence panel három fő területet tartalmaz:

- Felderített fenyegetéstípusok
- Fenyegetés forrása
- Fenyegetésfelderítési térkép

## <a name="machine-learning-in-azure-security-center"></a>A Machine learning az Azure Security Centerben

Az Azure Security Center mélyen elemzi a webhelyen elérhető adatok számos különféle Microsoftos és partneri megoldásokat érhet el a nagyobb biztonságot nyújt segítséget. Ezek az adatok kihasználása a vállalat adattudományi és machine learning a fenyegetések megelőzését, észlelését és végül vizsgálat használja.

Széles körben az Azure Machine Learning segít a két eredményeket érhet el:

## <a name="next-generation-detection"></a>Következő generációs észlelése

A támadók egyre inkább automatizált és kifinomult. Az adatelemzés túl használnak. Fordított-mérnök védelmet, és a rendszer működésében mutációk támogatja. Azok a tevékenységek révén, a háttérzaj, és ismerje meg gyorsan a hibákat. Machine learning segítségével ezen fejlesztések válaszolni.

## <a name="simplified-security-baseline"></a>Egyszerűsített biztonsági alapterv

Hatékony biztonsági döntéseket nem könnyű. Felhasználói élmény és a szakértelem van szükség. Néhány nagy szervezetek ilyen szakértői személyzet rendelkezik, nincs sok vállalat. Az Azure Machine Learning segítségével az ügyfelek kihasználhatják a más szervezetek is megfordíthatja, amikor biztonsági döntések meghozatalában.

## <a name="behavioral-analytics"></a>Működés elemzése

A működés elemzése olyan módszer, amely megvizsgálja és összehasonlítja az adatokat az ismert minták gyűjteményével. Ezek a minták azonban nem csak egyszerű aláírások. A meghatározásuk hatalmas adatkészletekre alkalmazott összetett gépi tanulási algoritmusok használatával. Ezenkívül szakértő elemzők mélyrehatóan elemezték a kártékony működést a meghatározásukhoz. Az Azure Security Center a működéselemzéssel azonosíthatja a feltört erőforrásokat a virtuális gépek naplóinak, a virtuális hálózati eszközök naplóinak, a háló naplóinak, a összeomlási memóriaképek és a más forrásokból elemzése alapján.
