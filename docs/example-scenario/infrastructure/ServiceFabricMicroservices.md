---
title: A Service Fabric használata érdekében a monolitikus alkalmazások
description: A nagy, monolitikus alkalmazások mikroszolgáltatásokra decomposing forgatókönyv
author: timomta
ms.date: 09/20/2018
ms.openlocfilehash: 38249a6db591652d207d76d5b07c30c816ca2be9
ms.sourcegitcommit: 36398937420b9a6a2c45759b90847a1819df4451
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 09/25/2018
ms.locfileid: "47175607"
---
# <a name="using-service-fabric-to-break-up-monolithic-applications"></a>A Service Fabric használata érdekében a monolitikus alkalmazások

Ebben a példában a forgatókönyvben azt végig egy megközelítés a [Service Fabric](/azure/service-fabric/service-fabric-overview) decomposing meg nehézkessé monolitikus alkalmazáshoz tartozó platform.  Itt azt gondolja át, egy IIS/ASP.Net webhely decomposing egy alkalmazásba, iteratív megközelítés mikroszolgáltatásokból álló, több felügyelhető.

Helyezi át a monolitikus a mikroszolgáltatási architektúrát, hogy érheti el a következő előnyökkel jár:

- Módosítsa a kód egy kis méretű, érthető egysége, és csak az adott egység üzembe helyezése
- Kód egységek mindegyikhez perc vagy annál kisebb üzembe helyezéséhez
- Ha hiba van a kis egység, csak az adott egység nem működik tovább, nem a teljes alkalmazáshoz
- A kód kis egységeket is egyszerűen és discretely szétoszthatók több fejlesztői csapatok
- Új fejlesztők is gyorsan és egyszerűen bonyolultnak diszkrét funkcióját a egység

Egy kiszolgálóból álló farmra nagy IIS-alkalmazás használatban van ebben a példában, de az iteratív idősorfelbontási és üzemeltetése bármilyen nagy is használható. Míg ez a megoldás használja a Windows, a Service Fabric is futtatható Linux rendszeren, az alapul szolgáló operációs rendszer. Ezt futtathatja a helyszínen, az Azure-ban, vagy a felhőbeli szolgáltató a kiválasztott virtuális gép csomópontján.

## <a name="related-use-cases"></a>Kapcsolódó alkalmazási helyzetek

Ebben a forgatókönyvben a következő nagy monolitikus webalkalmazások tapasztaló szervezetekre vonatkozó:

- Az egész helynek felosztása kisebb kódmódosítások hibák
- A hely teljes kiadási szükségessége miatt több napig tart kiadások
- Nehézségekbe bevezetési és hosszú ramp regisztrálásához új fejlesztők vagy fejlesztőcsapatok miatt az összetett programkód alap igénylő tudni, hogy egy személy legfeljebb is alkalmazható megőrzése

## <a name="architecture"></a>Architektúra

A Service Fabricet használja, a üzemeltetési platformot, hogy sikerült alakítsa nagy IIS-webhely gyűjteménye, mikroszolgáltatások, ahogy az alábbi:

![Teljes architektúra ábrája](./media/service-fabric-microservices/service-fabric-complete.png)

A fenti képen hogy egy nagy, az IIS-alkalmazás minden része szétbontása:

- Egy böngészőben a bejövő kérelmeket, fogadó útválasztást vagy a gateway szolgáltatás elemzi őket meghatározni, melyik szolgáltatás kezelje őket, majd továbbítja a kérést erre a szolgáltatásra.
- Négy ASP.Net Core-alkalmazásokat is hivatalosan virtuális könyvtárak (könyvtárak) alatt az ASP.Net-alkalmazások futtatásához egy IIS-webhely. Az alkalmazások saját független mikroszolgáltatásokra elkülönülnek. Milyen hatása, hogy megváltozott, a rendszerverzióval ellátott és a frissített külön-külön lehet. Ebben a példában azt rewrote mindegyik alkalmazás használata a .net Core és az ASP.Net Core. Ezek készültek, mint "[Reliable Services](/azure/service-fabric/service-fabric-reliable-services-introduction)" natív módon hozzáféréssel rendelkezzenek a teljes Service Fabric platformot képességeket és előnyöket (kommunikációs szolgáltatások, rendszerállapot-jelentések, értesítések, stb.).
- Egy Windows-szolgáltatás, neve "Indexelő szolgáltatás", helyezni egy Windows-tárolót, hogy már nincs közvetlen módosítást hajt végre az alapjául szolgáló kiszolgáló beállításjegyzékében, de futtatható önálló, majd egyetlen egységként minden függőségével együtt telepíteni.
- Egy archív szolgáltatás, amely csak egy végrehajtható fájl, amely egy ütemezés szerint fut, és végrehajt néhány feladatot a helyek. Vannak tárolva egy önálló végrehajtható fájl, közvetlenül, mert azt határozza meg, mit kell tennie módosítás nélkül hajtja végre, és nem érdemes módosítani a befektetés.

## <a name="considerations"></a>Megfontolandó szempontok

Az első ellenőrzést, hogy kisebb bits kód, amely képes megosztani ki a monolit, meghívhatja a monolit mikroszolgáltatásokra azonosításához megkezdéséhez. Iteratív, idővel a monolit van osztva, a fejlesztők könnyen megérthetik, módosítása és gyors üzembe helyezés alacsony kockázat ilyen mikroszolgáltatásokból álló egy gyűjteményt.

A Service Fabric lett választva, mert mindegyik mikroszolgáltatás a különféle formában futó támogathatja. Például előfordulhat, hogy utasításkészletet önálló végrehajtható fájlok, új kis webhelyek, új kis API-kat, és a tárolóalapú szolgáltatásokat, stb. A Service Fabric kombinálhatja ezeket egyetlen fürtöt alakzatot minden szolgáltatástípus.

A végső, elkülönített alkalmazás eléréséhez egy iteratív megközelítéssel használtuk. Hogy lépései IIS/ASP.Net nagyméretű webhelyek kiszolgálóból álló farmra. A kiszolgálófarm egyetlen csomópont van ábra alább. Az eredeti webhely a több virtuális könyvtárak (könyvtárak) tartalmaz, egy további Windows-szolgáltatás, a hely hívásokat, és a egy végrehajtható fájl, amely bizonyos rendszeres archiválása helykarbantartás.

![Monolitikus architektúra ábrája](./media/service-fabric-microservices/service-fabric-monolith.png)

A fejlesztés első példányát, az IIS-hely és a könyvtárak helyezni egy [Windows tároló](/azure/service-fabric/service-fabric-containers-overview). Ez lehetővé teszi a hely maradjon működési, de szorosan nincs kötve a mögöttes kiszolgáló-csomópontot az operációs rendszer. A tároló futtatásához és az alapul szolgáló Service Fabric-csomópont által előkészített, de a csomópont nem kell rendelkeznie minden olyan állapotban, hogy a hely (beállításjegyzékbeli bejegyzéseket, fájlokat, stb.) függ. Azok az elemek találhatók a tárolóban. Azt is be van jelölve az Indexelő szolgáltatás egy Windows-tároló okok miatt. A tárolók üzembe helyezett, a rendszerverzióval ellátott és a méretezett egymástól függetlenül lehet. Végül azt üzemeltetett az archív szolgáltatás egy egyszerű [önálló exe](/azure/service-fabric/service-fabric-guest-executables-introduction) egy önálló exe egy speciális, mert.

Az alábbi képen látható, hogyan a nagyméretű webhely most részlegesen bontjuk független egységekbe, és készen áll a kell szétbontása több idő lehetővé teszik.

![Részlegesen szétbontása architektúra ábrája](./media/service-fabric-microservices/service-fabric-midway.png)

További fejlesztési összpontosít elválasztása a egyetlen nagy Default Web site tároló fent szerepel. Mindegyik VDir ASP.Net alkalmazás egyszerre a tároló egyik távolítva, és az ASP.Net Core legelterjedtebb [a reliable services](/azure/service-fabric/service-fabric-reliable-services-introduction).

A könyvtárak mindegyike lett faktorált ki, ha az alapértelmezett webhely kiírt egy ASP.Net Core reliable Services, amely elfogadja a bejövő böngésző kéréseket, és továbbítja őket a megfelelő ASP.Net-alkalmazáshoz.

### <a name="availability-scalability-and-security"></a>Rendelkezésre állási, méretezhetőségi és biztonsági

A Service fabric [képes a különféle formában előforduló mikroszolgáltatások](/azure/service-fabric/service-fabric-choose-framework) miközben gondoskodik a hívások közötti gyors és egyszerű ugyanazon a fürtön. A Service fabric egy [hibatűrő](/azure/service-fabric/service-fabric-availability-services), önjavítási fürt, amely futtathat tárolókat, végrehajtható fájlok, és még rendelkezik egy natív API-t közvetlenül (a "Reliable Services" a fent említett), a mikroszolgáltatások írásához. A platform lehetővé teszi a működés közbeni frissítések és az egyes mikroszolgáltatások verziókezelését. Beállíthatja, hogy a több vagy kevesebb bármely adott mikroszolgáltatás annak érdekében, hogy a Service Fabric-fürt elosztva az platform [méretezési](/azure/service-fabric/service-fabric-concepts-scalability) be vagy ki csak a mikroszolgáltatás-alapú van szüksége.

A Service Fabric egy, fürt virtuális (vagy fizikai) csomópontokhoz, amelyek rendelkeznek a hálózati, tárolási és az operációs rendszer infrastruktúrájának épül. Rendelkezik mint ilyen, felügyeleti csoportja, a karbantartással és a figyelési feladatok.

Is érdemes figyelembe venni, cégirányítási és a fürt ellenőrzése. Nem érdemes személyek önkényesen üzembe az éles adatbázis-kiszolgáló, adatbázisok, mint sem lenne szeretné a Service Fabric-fürtöt néhány felügyelet nélküli alkalmazások telepítése.

Service Fabric, amely képes a számos különböző üzemeltetési [alkalmazási](/azure/service-fabric/service-fabric-application-scenarios), tekintse meg, melyeket alkalmazni a forgatókönyvéhez hosszabb időt is igénybe.

## <a name="pricing"></a>Díjszabás

Az Azure-ban üzemeltetett Service Fabric-fürtön a legnagyobb részét a költségek számát és méretét a csomópontok a fürtben. Az Azure lehetővé teszi mikroszolgáltatásokból álló, az alapul szolgáló csomópont mérete határozza meg a fürt létrehozása gyors és egyszerű, de a számítási díjak a csomópont méretét a csomópontok számát megszorozva alapulnak.

Egyéb költség kevésbé költséges összetevői a tárolási díjakat minden csomópont virtuális lemezek és hálózati i/o kimenő forgalom (például hálózati forgalom Azure-ból a felhasználó böngészője) Azure-ból.

A képet kapjon a költség, hoztunk létre egy példa néhány alapértelmezett értékeinek használata foglalásiegység-méret, hálózati és tárolási: vessen egy pillantást a [díjkalkulátor](https://azure.com/e/52dea096e5844d5495a7b22a9b2ccdde). Nyugodtan frissítse az értékeket, az alapértelmezett kalkulátor való a helyzetnek megfelelő.

## <a name="next-steps"></a>További lépések

Ismerkedjen meg a platform a hosszabb időt is igénybe a [dokumentáció](/azure/service-fabric/service-fabric-overview) és különböző több áttekintésével [alkalmazási](/azure/service-fabric/service-fabric-application-scenarios) a Service Fabrichez. A dokumentáció, megtudhatja, milyen egy fürt tartalmaz, mi a futtatásához, szoftverarchitektúrára és karbantartási.

Egy meglévő .NET-alkalmazás a Service Fabric bemutató megtekintéséhez üzembe helyezése a Service Fabric [rövid](/azure/service-fabric/service-fabric-quickstart-dotnet).

Az aktuális alkalmazás szempontjából, gondolja át a különböző funkciók kezdődik. Válasszon egyet ezek közül, és úgy gondolja, hogy hogyan elkülönítheti a függvény csak a teljes keresztül. Egyszerre csak egy különálló, érthető, darab végrehajtani.

## <a name="related-resources"></a>Kapcsolódó erőforrások

- [Mikroszolgáltatások létrehozása az Azure-ban](/azure/architecture/microservices/)
- [A Service Fabric áttekintése](/azure/service-fabric/service-fabric-overview)
- [Service Fabric programozási modell](/azure/service-fabric/service-fabric-choose-framework)
- [Service Fabric rendelkezésre állása](/azure/service-fabric/service-fabric-availability-services)
- [A Service Fabric méretezése](/azure/service-fabric/service-fabric-concepts-scalability)
- [Üzemeltetési tárolók Service Fabricben](/azure/service-fabric/service-fabric-containers-overview)
- [A Service Fabricben üzemeltetési önálló végrehajtható fájlok](/azure/service-fabric/service-fabric-guest-executables-introduction)
- [A Service Fabric natív Reliable Services](/azure/service-fabric/service-fabric-reliable-services-introduction)
- [Service Fabric-alkalmazás forgatókönyvek](/azure/service-fabric/service-fabric-application-scenarios)