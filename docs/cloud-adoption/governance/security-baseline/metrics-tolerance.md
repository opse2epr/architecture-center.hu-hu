---
title: 'CAF: Biztonsági alapmértékekkel, a mutatók és a szervezet kockázattűrési határát'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Biztonsági alapmértékekkel, a mutatók és a szervezet kockázattűrési határát
author: BrianBlanchard
ms.openlocfilehash: 30deafca59b2e09c78432ad3b59d328fb27a1e2c
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298430"
---
# <a name="security-baseline-metrics-indicators-and-risk-tolerance"></a>Biztonsági alapmértékekkel, a mutatók és a szervezet kockázattűrési határát

Ebből a cikkből segítséget nyújt az üzleti szervezet kockázattűrési határát számszerűsítik a biztonsági alapkonfiguráció vonatkozik. Metrikák és a mutatók segítségével hozhat létre egy üzleti esetet, befektetés végzett biztonsági alapterv tantárgy stabilizálódó.

## <a name="metrics"></a>Mérőszámok

Alapvető biztonsági általában összpontosít a magánfelhők számára a potenciális biztonsági réseket azonosítása. A kockázati részeként elemzési adatok gyűjtéséhez szeretné kapcsolatos meghatározni, hogy mekkora kockázatot a biztonsági környezet, arcfelismerés, és mennyire fontos befektetési biztonsági alapterv cégirányítási, hogy a tervezett felhőalapú környezetekben.

Mivel minden munkahely, eltérő biztonsági környezetben és a követelményeket és a biztonsági adatok különböző lehetséges forrásai. A következő példák hasznos metrikák mérik a kockázattűrése az alapvető biztonsági területen belül összegyűjtendő:

- **Az adatbesorolás**. Felhőben tárolt adatokat és szolgáltatásokat, amelyek nem a következők szerint a szervezet adatvédelmi, megfelelőségi és üzleti hatás szabványok a száma.
- **Bizalmas adatok üzletek száma** -tárolási végpontok vagy meg kell védeni, és a bizalmas adatokat tartalmazó adatbázisok számát.
- **A titkosítatlan adatok üzletek száma** – nem titkosított bizalmas adatok üzletek száma.
- **Támadási felület**. Hány teljes adatforrások, szolgáltatások és alkalmazások felhőben üzemeltetett lesz. Ezek az adatforrások százaléka bizalmasként van? Ezek az alkalmazások és szolgáltatások hány százaléka azok alapvető fontosságú?
- **Szabványokkal szabályozott**. A biztonsági csapat által meghatározott biztonsági előírások száma.
- **Lefedett erőforrások**. Üzembe helyezett eszközök biztonsági szabványok alá esnek.
- **Általános szabványoknak való megfelelés**. Biztonsági szabványok betartásának megfelelőségi aránya.
- **Súlyosság szerint támadások**. Hány koordinált kísérletek akadályozza a felhőben üzemeltetett szolgáltatások, például keresztül elosztott szolgáltatásmegtagadásos (DDoS) támadások, az infrastruktúra does élmény? Mi az a méretét és az ilyen jellegű támadások súlyossága?
- **Kártevők elleni védekezés**. Százalékos üzembe helyezett virtuális gépek (VM), hogy minden szükséges kártevőirtó, tűzfal vagy más biztonsági szoftver telepítve van.
- **Késési javítási**. Mennyi ideig, lett, mivel a virtuális gépek kellett volna a alkalmazni az operációs rendszer és szoftverek javítások.
- **Biztonsági állapot javaslatok**. Biztonsági szoftver javaslatok állapota standards üzembe helyezett erőforrások súlyosság szerint rendezve megoldásának száma.

## <a name="risk-tolerance-indicators"></a>Kockázati tolerancia mutatók

Cloud platform szolgáltatásai lehetővé teszik az alapvető biztonsági beállítások konfigurálása a további alapos tervezés nélkül a kisméretű környezetet csapatok alapkészletével adja meg. Ennek eredményeképpen kisebb fejlesztési/tesztelési vagy kísérleti első számítási feladatok, amelyek nem tartalmaznak bizalmas adatok viszonylag alacsony kockázat képviselik, és valószínűleg nem kell annyit átszállítást megakadályozzák a formális alapvető biztonsági házirend. Azonban, amint a fontos adatok vagy az alapvető fontosságú funkcióit helyezik át a felhőbe, biztonsági kockázatokat növelését, miközben ezek a kockázatok szintű lecsökken rövid idő alatt. Ahogyan több adatait és funkcióit a felhő telepít, annál valószínűbb kell a biztonsági alapkonfiguráció fegyelem megnövekedett befektetést.

A felhő bevezetésének korai szakaszában, az informatikai biztonsági csoport és az üzleti résztvevőkkel azonosításához együttműködve [üzleti kockázatai](business-risks.md) biztonsággal kapcsolatos, majd vizsgálja meg, az elfogadható referenciakonfigurációjához tartozó biztonsági szervezet kockázattűrési határát. Ez a szakasza a CAF példákat talál, de a részletes kockázatokat és a vállalat vagy a központi telepítések alapterveit eltérő lehet.

Ha már van egy alapkonfiguráció elfogadhatatlan mértékű növekedést az azonosított kockázatok a minimális referenciaalapokhoz képest történő létrehozásához. Ezek referenciaalapokhoz képest történő eseményindítók a szerepét, amikor szüksége van arra kockázatok csökkentése érdekében. Az alábbiakban néhány példa a biztonsági mérőszámokat, például a fent tárgyalt hogyan igazolja az alapvető biztonsági fegyelem megnövekedett befektetést.

- **Alapvető fontosságú számítási feladatokhoz eseményindító**. A vállalat üzleti szempontból alapvető fontosságú számítási feladatokat helyeznek üzembe a felhőben kell beruházni a biztonsági alapkonfiguráció fegyelem szolgáltatás vagy a bizalmas adatok esetleges kimaradásának elkerülése érdekében.
- **Védett adatok eseményindító**. A vállalat bizalmas, titkos vagy jogszabályi szempontok vonatkoznak más módon kell besorolni felhőbeli üzemeltetéséhez. Győződjön meg arról, hogy ezek az adatok nem adatvesztés, kitettség vagy ellopják, egy biztonsági alapterv fegyelem van szükségük.
- **Külső támadásokkal eseményindító**. Egy vállalat, amely során a hálózati infrastruktúra X időtől havonta elleni súlyos támadások lép hasznos lenne az az alapvető biztonsági fegyelem.  
- **Szabványok megfelelőségi eseményindító**. A vállalat több, mint X %-a biztonsági szabványoknak való megfelelés-erőforrásokat kell fektethet be a biztonsági alapkonfiguráció fegyelem szabványok következetesen alkalmazzák az informatikai infrastruktúrában biztosításához.
- **Felhőbeli hagyatéki mérete eseményindító**. Egy üzemeltetési vállalat több mint X alkalmazások, szolgáltatások és adatok források száma. Nagyméretű felhőbeli üzembe az alapvető biztonsági területen győződjön meg arról, hogy általános támadási felület megfelelően védett-e illetéktelen hozzáféréstől és más külső fenyegetésekkel szemben, befektetés is kihasználhatják.
- **Biztonsági szoftverfrissítési megfelelőség eseményindító**. Egy vállalati, ahol kevesebb, mint X üzembe helyezett virtuális gépek % van-e az összes szükséges biztonsági szoftver telepítve. A biztonsági alapkonfiguráció fegyelem segítségével győződjön meg arról, telepített szoftverek konzisztens az összes szoftver.
- **Az eseményindító javítás**. Egy vállalat, ahol üzembe helyezett virtuális gépek vagy szolgáltatások, ahol az operációs rendszer vagy a szoftver javítások nem lettek végrehajtva az utolsó X nap száma. A biztonsági alapkonfiguráció fegyelem annak biztosítása érdekében javítási mindig naprakész marad szükséges ütemezés belül használható.
- **A fókuszban lévő biztonsági**. Egyes vállalatok lesz erős biztonsági és az adatok bizalmas követelmények még a teszt- és kísérleti számítási feladatokhoz. Ezek a vállalatok biztonsági alapterv tantárgy befektetni a telepítések megkezdése előtt kell.

A pontos mérőszámokat és eseményindítók összevetheti a szervezet kockázattűrési határát, és a szintet, befektetés az alapvető biztonsági fegyelem használja a szervezet specifikus, de hasznos bázi. a felhő Cégirányítási csapaton belül leírásáért lásd a fenti példák szolgáljon.  

## <a name="next-steps"></a>További lépések

Használatával a [Cloud Management-sablon](./template.md), dokumentum-mérőszámok és hibatűrő mutatók, amelyek a jelenlegi cloud bevezetési tervet.

Building a kockázatokat és a hibatűrést, szabályozó és biztonsági alapterv szabályzat betartása egy folyamatot hoz létre.

> [!div class="nextstepaction"]
> [Szabályzat megfelelőségi folyamat létrehozása](compliance-processes.md)
